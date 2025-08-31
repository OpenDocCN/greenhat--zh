# 第十一章：控制平面和访问控制

![图片](img/common01.jpg)

控制平面管理 Kubernetes 集群，存储应用程序的期望状态，监视当前状态以检测和恢复任何问题，调度新容器，并配置网络路由。在本章中，我们将仔细研究 API 服务器，这是控制平面的主要接口，也是检索任何状态和对整个集群进行更改的入口点。

虽然我们将重点放在 API 服务器上，但控制平面包括多个其他服务，每个服务都有各自的角色。其他控制平面服务作为 API 服务器的客户端，监视集群变化，并采取适当措施来更新集群的状态。以下列表描述了其他控制平面组件：

**调度程序** 将每个新 Pod 分配给一个节点。

**控制器管理器** 具有多种责任，包括为部署创建 Pod、监视节点并对故障做出反应。

**云控制器管理器** 这是一个可选组件，与底层云提供程序接口，检查节点并配置网络流量路由。

当我们演示 API 服务器的工作原理时，我们还将看到 Kubernetes 如何管理安全性，以确保只有授权的用户和服务可以查询集群并进行更改。像 Kubernetes 这样的容器编排环境的目的是为我们可能需要运行的任何类型的容器化应用程序提供平台，因此这种安全性至关重要，以确保集群仅按预期使用。

### API 服务器

尽管它在 Kubernetes 架构中的核心地位，API 服务器的目的很简单。它使用 HTTP 和表征状态转移（REST）暴露接口，用于执行集群中资源的基本创建、检索、更新和删除。它执行身份验证以识别客户端，授权以确保客户端对特定请求有权限，并验证以确保任何创建或更新的资源与相应的规范匹配。它还根据从客户端接收到的命令读取和写入数据存储。

然而，API 服务器并不负责实际更新集群的当前状态以匹配期望状态。这是其他控制平面和节点组件的责任。例如，如果客户端创建一个新的 Kubernetes 部署，API 服务器的工作仅仅是更新数据存储中的资源信息。然后，调度程序负责决定 Pod 将在哪里运行，分配给节点上的 kubelet 服务负责创建和监视容器，并配置网络以将流量路由到容器。

在这一章中，我们有一个由自动化脚本配置的三节点 Kubernetes 集群。三台节点都充当控制平面节点，因此有三份 API 服务器在运行。我们可以与其中任意一台进行通信，因为它们共享同一个后端数据库。API 服务器正在端口 6443 上监听安全 HTTP 连接，这是默认端口。

**注意**

*本书的示例仓库位于* [`github.com/book-of-kubernetes/examples`](https://github.com/book-of-kubernetes/examples)。*有关设置的详细信息，请参见第 xx 页中的“运行示例”。*

我们一直在使用`kubectl`与 API 服务器通信，创建和删除资源并获取状态，且`kubectl`一直通过端口 6443 使用安全 HTTP 与集群通信。它之所以这样做，是因为在集群初始化时，`kubeadm`将一个 Kubernetes 配置文件安装到*/etc/kubernetes*中。这个配置文件还包含认证信息，使我们能够读取集群状态并进行更改。

因为 API 服务器期待安全 HTTP 连接，我们可以使用`curl`直接与 Kubernetes API 通信。这将帮助我们更好地理解通信是如何工作的。我们从一个简单的`curl`命令开始：

```
root@host01:~# curl https://192.168.61.11:6443/
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.se/docs/sslcerts.html
...
```

这个错误信息表明`curl`不信任 API 服务器提供的证书。我们可以使用`curl`查看这个证书：

```
root@host01:~# curl -kv https://192.168.61.11:6443/
...
* Server certificate:
*  subject: CN=kube-apiserver
...
*  issuer: CN=kubernetes
...
```

`-k`选项告诉`curl`忽略任何证书问题，而`-v`选项则告诉`curl`提供更多的连接日志信息。

为了让`curl`信任这个证书，它需要信任`issuer`，因为 issuer 是证书的签署者。我们来从 Kubernetes 安装中提取证书，这样我们就可以将`curl`指向它：

```
root@host01:~# cp /etc/kubernetes/pki/ca.crt .
```

确保在文件名末尾加上`.`，将该文件复制到当前目录。我们这么做完全是为了方便后面命令的输入。

在使用证书之前，让我们先查看一下它：

```
root@host01:~# openssl x509 -in ca.crt -text
Certificate:
...
        Issuer: CN = kubernetes
...
        Subject: CN = kubernetes
```

`Issuer`和`Subject`是相同的，因此这是一个*自签名*证书。它是通过`kubeadm`在初始化集群时创建的。使用生成的证书使得`kubeadm`能够适应我们特定的集群网络配置，并且允许我们的集群拥有唯一的证书和密钥，而无需外部证书机构（CA）。然而，这意味着我们需要配置`kubectl`以信任此证书，以便在任何需要与 API 服务器通信的系统上使用。

现在我们可以告诉`curl`使用这个证书来验证 API 服务器：

```
root@host01:~# curl --cacert ca.crt https://192.168.61.11:6443/
{
...
  "status": "Failure",
  "message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
...
  "code": 403
}
```

现在我们已经为 `curl` 提供了正确的根证书，`curl` 可以验证 API 服务器证书，我们也能成功连接到 API 服务器。然而，API 服务器返回 403 错误，表示我们没有授权。这是因为目前我们没有为 `curl` 提供任何身份验证信息，导致 API 服务器将我们视为匿名用户。

最后一点：为了使此 `curl` 命令生效，我们需要选择性地使用主机名或 IP 地址。API 服务器监听所有网络接口，因此我们可以使用 `localhost` 或 `127.0.0.1` 连接它。然而，这些并未列在 `kube-apiserver` 证书中，且由于 `curl` 不会信任连接，因此无法用于安全的 HTTP。

### API 服务器身份验证

我们需要提供身份验证信息，API 服务器才会接受我们的请求，因此让我们了解一下 API 服务器的身份验证过程。身份验证是通过一组插件来处理的，每个插件会查看请求，以确定它是否能够识别客户端。第一个成功识别客户端的插件将身份信息提供给 API 服务器。然后，这个身份与授权一起使用，以确定客户端被允许执行的操作。

由于身份验证是基于插件的，因此可以根据需要使用多种不同的客户端身份验证方式。甚至可以在 API 服务器前添加一个代理，执行自定义身份验证逻辑，并通过 HTTP 头将用户的身份传递给 API 服务器。

对于我们的目的，我们将关注集群内部或集群设置过程中使用的三种主要身份验证插件：*客户端证书*、*启动令牌* 和 *服务账户*。

#### 客户端证书

如前所述，像 `curl` 这样的 HTTP 客户端通过将服务器的主机名与其证书进行比较，来验证服务器的身份，同时还会检查证书的签名是否与受信任的 CA 列表一致。除了检查服务器身份外，安全的 HTTP 还允许客户端向服务器提交证书。服务器将签名与其受信任的机构列表进行比对，然后使用证书的主题作为客户端的身份。

Kubernetes 广泛使用 HTTP 客户端证书身份验证，以便集群服务能够与 API 服务器进行身份验证。这包括控制平面组件以及每个节点上运行的 `kubelet` 服务。我们可以使用 `kubeadm` 列出控制平面使用的证书：

```
root@host01:~# kubeadm certs check-expiration
...
CERTIFICATE                ...  RESIDUAL TIME   CERTIFICATE AUTHORITY ...
admin.conf                 ...  363d                                  ...
apiserver                  ...  363d            ca                    ...
apiserver-etcd-client      ...  363d            etcd-ca               ...
apiserver-kubelet-client   ...  363d            ca                    ...
controller-manager.conf    ...  363d                                  ...
etcd-healthcheck-client    ...  363d            etcd-ca               ...
etcd-peer                  ...  363d            etcd-ca               ...
etcd-server                ...  363d            etcd-ca               ...
front-proxy-client         ...  363d            front-proxy-ca        ...
scheduler.conf             ...  363d                                  ...
...
```

`RESIDUAL TIME` 列显示证书过期前剩余的时间；默认情况下，它们在一年后过期。使用 `kubeadm certs renew` 来续订证书，传递证书的名称作为参数。

列表中的第一个项目 `admin.conf` 是我们在过去几章中用于验证自己身份的方式。在初始化过程中，`kubeadm` 创建了这个证书，并将其信息存储在 */etc/kubernetes/admin.conf* 文件中。我们运行的每个 `kubectl` 命令都在使用这个文件，因为我们的自动化脚本设置了 `KUBECONFIG` 环境变量：

```
root@host01:~# echo $KUBECONFIG
/etc/kubernetes/admin.conf
```

如果我们没有设置 `KUBECONFIG`，`kubectl` 将使用默认文件，即用户主目录下的 *.kube/config* 文件。

*admin.conf* 凭据旨在提供紧急访问集群的权限，绕过授权。在生产集群中，我们会避免直接使用这些凭据进行日常操作。相反，生产集群的最佳做法是为管理员和普通用户集成一个单独的身份管理器。对于我们的示例，由于没有单独的身份管理器，我们将为普通用户创建一个额外的证书。这种证书可能对在集群外部运行的自动化进程有用，但它无法与身份管理器集成。

我们可以使用 `kubeadm` 创建一个新的客户端证书：

```
root@host01:~# kubeadm kubeconfig user --client-name=me \
  --config /etc/kubernetes/kubeadm-init.yaml > kubeconfig
```

`kubeadm kubeconfig user` 命令请求 API 服务器生成一个新的客户端证书。由于这个证书是由集群的 CA 签名的，因此它可以用于身份验证。证书与连接 API 服务器所需的配置一起保存在 *kubeconfig* 文件中：

```
root@host01:~# cat kubeconfig
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: ...
    server: https://192.168.61.10:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: me
  name: me@kubernetes
current-context: me@kubernetes
kind: Config
preferences: {}
users:
- name: me
  user:
    client-certificate-data: ...
    client-key-data: ...
```

`clusters` 部分定义了连接到 API 服务器所需的信息，包括在我们高可用配置中，所有三个 API 服务器共享的负载均衡地址。`users` 部分定义了我们创建的新用户及其客户端证书。

到目前为止，我们成功创建了一个新用户，但尚未赋予该用户任何权限，因此我们用这些凭据的操作不会非常成功：

```
root@host01:~# KUBECONFIG=kubeconfig kubectl get pods
Error from server (Forbidden): pods is forbidden: User "me" cannot list 
  resource "pods" in API group "" in the namespace "default"
```

本章稍后我们将看到如何为这个用户授予权限。

#### 引导令牌

初始化一个分布式系统，如 Kubernetes 集群，是一项具有挑战性的任务。每个节点上运行的 `kubelet` 服务必须被添加到集群中。为此，`kubelet` 必须连接到 API 服务器并获取由集群的 CA 签名的客户端证书。然后，`kubelet` 服务使用该客户端证书进行集群认证。

证书的生成必须安全进行，以消除将恶意节点添加到集群的可能性，并防止恶意进程冒充真实节点。因此，API 服务器不能为任何请求加入集群的节点提供证书。相反，节点必须生成自己的私钥，向 API 服务器提交证书签名请求（CSR），并接收签名证书。

为了确保这个过程的安全性，我们需要确保一个节点被授权提交证书签名请求。但这个提交必须在节点获得用于更长期身份验证的客户端证书之前进行——我们面临一个先有鸡还是先有蛋的问题！Kubernetes 通过时间限制令牌来解决这个问题，这些令牌被称为*引导令牌*。引导令牌成为一个预共享的秘密，API 服务器和新节点都知道它。使这个令牌具有时间限制可以降低它暴露时对集群的风险。Kubernetes 控制器管理器负责在引导令牌过期时自动清理它们。

当我们初始化集群时，`kubeadm` 创建了一个引导令牌，但它配置为两小时后过期。如果我们在此之后需要将额外的节点加入集群，我们可以使用 `kubeadm` 生成一个新的引导令牌：

```
root@host01:~# TOKEN=$(kubeadm token create)
root@host01:~# echo $TOKEN
pqcnd6.4wawyqgkfaet06zm
```

这个令牌作为 Kubernetes *Secret* 被添加到`kube-system`命名空间。我们将在第十六章中更详细地讨论 Secrets。现在，我们只需要验证它是否存在：

```
root@host01:~# kubectl -n kube-system get secret
NAME                    TYPE                           DATA   AGE
...
bootstrap-token-pqcnd6  bootstrap.kubernetes.io/token  6      64s
...
```

我们可以使用这个令牌通过 HTTP Bearer 身份验证向 API 服务器发出请求。这意味着我们在 HTTP 头中提供令牌，头部的名称为`Authorization`，并以`Bearer`为前缀。当引导令牌认证插件看到该头并将提供的令牌与相应的密钥进行匹配时，它会验证我们的身份并允许我们访问 API。

出于安全原因，引导令牌仅能访问 API 服务器的证书签名请求功能，因此我们的令牌只能执行这个操作。

让我们使用引导令牌列出所有证书签名请求：

```
root@host01:~# curl --cacert ca.crt \
  -H "Authorization: Bearer $TOKEN" \
  https://192.168.61.11:6443/apis/certificates.k8s.io/v1/certificatesigningrequests
{
  "kind": "CertificateSigningRequestList",
 "apiVersion": "certificates.k8s.io/v1",
  "metadata": {
    "resourceVersion": "21241"
  },
  "items": [
...
  ]
}
```

了解引导令牌的工作原理非常重要，因为它们对于将节点添加到集群至关重要。然而，正如其名称所示，引导令牌实际上只有这个目的；通常不会用于正常的 API 服务器访问。对于正常的 API 服务器访问，尤其是在集群内部，我们需要一个*服务账户*。

#### 服务账户

在 Kubernetes 集群中运行的容器通常需要与 API 服务器进行通信。例如，在我们在第六章中部署的所有组件，包括 Calico 网络插件、Longhorn 存储驱动程序和指标服务器，都会与 API 服务器通信，以观察和修改集群状态。为了支持这一点，Kubernetes 会自动将凭证注入每个运行中的容器。

当然，出于安全原因，仅授予每个容器所需的 API 服务器权限非常重要，因此我们应该为每个应用或集群组件创建一个单独的 ServiceAccount。然后，将这些 ServiceAccount 的信息添加到 Deployment 或其他控制器中，以便 Kubernetes 会注入正确的凭据。在某些情况下，我们可能会为一个应用使用多个 ServiceAccount，限制每个应用组件只能访问其所需的权限。

除了为每个应用或组件使用单独的 ServiceAccount 外，最好为每个应用使用单独的命名空间。正如我们稍后将看到的，权限可以限制在单一命名空间内。让我们首先创建命名空间：

```
root@host01:~# kubectl create namespace sample
namespace/sample created
```

ServiceAccount 使用承载令牌，该令牌存储在 Kubernetes 创建 ServiceAccount 时自动生成的 Secret 中。让我们为本章中将要创建的 Deployment 创建一个 ServiceAccount：

*read-pods-sa.yaml*

```
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: read-pods
  namespace: sample
```

请注意，我们使用元数据将这个 ServiceAccount 放入我们刚刚创建的`sample`命名空间中。我们也可以使用 `-n` 标志与 `kubectl` 一起指定命名空间。我们将使用常规的 `kubectl apply` 来创建这个 ServiceAccount：

```
root@host01:~# kubectl apply -f /opt/read-pods-sa.yaml
serviceaccount/read-pods created
```

当创建 ServiceAccount 时，控制器管理器会检测到这一点，并自动创建一个包含凭证的 Secret：

```
root@host01:~# kubectl -n sample get serviceaccounts
NAME        SECRETS   AGE
default     1         27s
read-pods   1         8s
root@host01:~# kubectl -n sample get secrets
NAME                    TYPE                                  DATA   AGE
default-token-mzwpt     kubernetes.io/service-account-token   3      43s
read-pods-token-m4scq   kubernetes.io/service-account-token   3      25s
```

请注意，除了我们刚刚创建的`read-pods` ServiceAccount 外，还有一个已经存在的`default` ServiceAccount。这个账户是在创建命名空间时自动创建的；如果我们没有指定使用哪个 ServiceAccount 来为 Pod 提供服务，Kubernetes 将使用它。

新创建的 ServiceAccount 还没有任何权限。为了开始添加权限，我们需要了解一下 *基于角色的访问控制*（RBAC）。

### 基于角色的访问控制

在 API 服务器找到能够识别客户端的认证插件之后，它会使用该身份来判断客户端是否有权限执行所需的操作，这通过组合属于用户的角色列表来完成。角色可以直接与用户关联，也可以与用户所在的组关联。组成员资格是身份的一部分。例如，客户端证书可以通过在证书主题中包含组织字段来指定用户的组。

#### 角色和集群角色

每个角色都有一组权限。权限允许客户端对一种或多种资源类型执行一个或多个操作。

举个例子，让我们定义一个角色，授予客户端读取 Pod 状态的权限。我们有两个选择：可以创建一个*Role*或*ClusterRole*。Role 仅在单个命名空间内可见和可用，而 ClusterRole 在所有命名空间中都可见和可用。这一差异允许管理员在整个集群中定义通用角色，这些角色在新命名空间创建时立即可用，同时也允许为特定命名空间委派访问控制。

下面是 ClusterRole 的示例定义。该角色仅具备读取 Pods 数据的权限；不能修改 Pods 或访问其他集群信息：

*pod-reader.yaml*

```
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

因为这是一个集群范围的角色，将其分配给某个命名空间是没有意义的，因此我们不指定命名空间。

这个定义的关键部分是规则列表。每个 ClusterRole 或 Role 可以有任意数量的规则。每条规则都有一组`verbs`，定义了允许的操作。在此案例中，我们将`get`、`watch`和`list`作为动词，这意味着该角色允许读取 Pods，但不允许进行任何修改操作。

每条规则适用于一个或多个资源类型，这取决于`apiGroups`和`resources`的组合。每条规则为列出的`verbs`（动词）操作提供权限。在这种情况下，空字符串`""`用于引用默认的 API 组，即 Pods 所在的地方。如果我们想要同时包括 Deployments 和 StatefulSets，我们需要将规则定义如下：

```
- apiGroups: ["", "apps"]
  resources: ["pods", "deployments", "statefulsets"]
  verbs: ["get", "watch", "list"]
```

我们需要将`"apps"`添加到`apiGroups`字段中，因为 Deployment 和 StatefulSet 属于该组（在声明资源时，可以在`apiVersion`中识别）。当我们声明 Role 或 ClusterRole 时，API 服务器会接受`apiGroups`和`resources`字段中的任何字符串，无论该组合是否确实识别出任何资源类型，因此，必须注意资源属于哪个组。

让我们定义我们的`pod-reader` ClusterRole：

```
root@host01:~# kubectl apply -f /opt/pod-reader.yaml
clusterrole.rbac.authorization.k8s.io/pod-reader created
```

现在 ClusterRole 已经存在，我们可以应用它。为此，我们需要创建一个角色绑定。

#### 角色绑定和集群角色绑定

让我们将这个`pod-reader` ClusterRole 应用到我们之前创建的`read-pods` ServiceAccount。我们有两个选择：可以创建一个*RoleBinding*，它会将权限分配到特定的命名空间，或者创建一个*ClusterRoleBinding*，它会将权限分配到所有命名空间。这一特性非常有用，因为它意味着我们可以创建一个像`pod-reader`这样的 ClusterRole，并使其在整个集群中可见，但只在特定命名空间内创建绑定，以便用户和 ServiceAccount 仅能访问他们被允许访问的命名空间。这帮助我们应用之前提到的每个应用有一个命名空间的模式，同时确保非管理员用户无法接触到关键的基础设施组件，如在`kube-system`命名空间中运行的组件。

按照这个做法，我们将创建一个 RoleBinding，以便我们的 ServiceAccount 仅有权在 `sample` 命名空间中读取 Pods：

*read-pods-bind.yaml*

```
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: sample
subjects:
- kind: ServiceAccount
  name: read-pods
  namespace: sample
roleRef:
  kind: ClusterRole
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

不出所料，RoleBinding 将一个 Role 或 ClusterRole 与一个主体关联起来。RoleBinding 可以包含多个主体，因此我们可以通过单个绑定将相同的角色绑定到多个用户或组。

我们在元数据中定义了一个命名空间，并且在标识主体的地方也定义了命名空间。在这种情况下，两个地方都是 `sample`，因为我们希望授予 ServiceAccount 在其自己的命名空间中读取 Pod 状态的权限。然而，这两个命名空间也可以不同，以允许一个命名空间中的 ServiceAccount 在另一个命名空间中具有特定权限。当然，我们也可以使用 ClusterRoleBinding 来授予跨所有命名空间的权限。

现在我们可以创建 RoleBinding 了：

```
root@host01:~# kubectl apply -f /opt/read-pods-bind.yaml
rolebinding.rbac.authorization.k8s.io/read-pods created
```

我们现在已授予 `read-pods` ServiceAccount 在 `sample` 命名空间中读取 Pods 的权限。为了演示其工作原理，我们需要创建一个分配给 `read-pods` ServiceAccount 的 Pod。

#### 将 Service Account 分配给 Pods

要将 ServiceAccount 分配给 Pod，只需将 `serviceAccountName` 字段添加到 Pod 的 spec 中：

*read-pods-deploy.yaml*

```
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: read-pods
  namespace: sample
spec:
  replicas: 1
  selector:
    matchLabels:
      app: read-pods
  template:
    metadata:
      labels:
        app: read-pods
    spec:
      containers:
      - name: read-pods
        image: alpine
        command: ["/bin/sleep", "infinity"]
      serviceAccountName: read-pods
```

所标识的 ServiceAccount 必须存在于创建 Pod 的命名空间中。Kubernetes 会将 Pod 的容器注入 Service-Account 令牌，以便容器可以认证到 API 服务器。

让我们通过一个示例来展示这个过程，并说明授权是如何应用的。首先创建这个 Deployment：

```
root@host01:~# kubectl apply -f /opt/read-pods-deploy.yaml
deployment.apps/read-pods created
```

这会创建一个运行 `sleep` 的 Alpine 容器，我们可以将其用作 shell 命令的基础。

要进入 shell 提示符，我们首先获取 Pod 的生成名称，然后使用 `kubectl exec` 创建 shell：

```
root@host01:~# kubectl -n sample get pods
NAME                        READY   STATUS    RESTARTS   AGE
read-pods-9d5565548-fbwjb   1/1     Running   0          6s
root@host01:~# kubectl -n sample exec -ti read-pods-9d5565548-fbwjb -- /bin/sh
/ #
```

ServiceAccount 令牌挂载在目录 */run/secrets/kubernetes.io/serviceaccount* 中，因此切换到该目录并列出其内容：

```
/ # cd /run/secrets/kubernetes.io/serviceaccount
/run/secrets/kubernetes.io/serviceaccount # ls -l
total 0
lrwxrwxrwx    1 root     root  ...  ca.crt -> ..data/ca.crt
lrwxrwxrwx    1 root     root  ...  namespace -> ..data/namespace
lrwxrwxrwx    1 root     root  ...  token -> ..data/token
```

这些文件看起来像是奇怪的符号链接，但内容如预期所示。*ca.crt* 文件是集群的根证书，它用于信任与 API 服务器的连接。

让我们将令牌保存在一个变量中，以便使用：

```
/run/secrets/kubernetes.io/serviceaccount # TOKEN=$(cat token)
```

现在我们可以使用这个令牌与 `curl` 连接到 API 服务器。但首先，我们需要将 `curl` 安装到 Alpine 容器中：

```
default/run/secrets/kubernetes.io/serviceaccount # apk add curl
...
OK: 8 MiB in 19 packages
```

我们的 ServiceAccount 被允许对 Pods 执行 `get`、`list` 和 `watch` 操作。让我们列出 `sample` 命名空间中的所有 Pods：

```
/run/secrets/kubernetes.io/serviceaccount # curl --cacert ca.crt \
  -H "Authorization: Bearer $TOKEN" \
  https://kubernetes.default.svc/api/v1/namespaces/sample/pods
  "kind": "PodList",
  "apiVersion": "v1",
  "metadata": {
    "resourceVersion": "566610"
  },
  "items": [
    {
      "metadata": {
        "name": "read-pods-9d5565548-fbwjb",
...
  ]
}
```

与引导令牌一样，我们使用 HTTP Bearer 身份验证将 ServiceAccount 令牌传递给 API 服务器。由于我们在容器内部操作，我们可以使用标准地址 `kubernetes.default.svc` 来查找 API 服务器。这是可行的，因为 Kubernetes 集群始终在 `default` 命名空间中拥有一个服务，使用我们在第九章中看到的服务网络将流量路由到 API 服务器实例。

`curl`命令成功了，因为我们的 ServiceAccount 已绑定到我们创建的`pod-reader`角色。然而，RoleBinding 限定于`sample`命名空间，因此我们不能列出其他命名空间中的 Pods：

```
/run/secrets/kubernetes.io/serviceaccount # curl --cacert ca.crt \
  -H "Authorization: Bearer $TOKEN" \
  https://kubernetes.default.svc/api/v1/namespaces/kube-system/pods
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {
  },
  "status": "Failure",
  "message": "pods is forbidden: User 
    \"system:serviceaccount:default:read-pods\" cannot list resource 
    \"pods\" in API group \"\" in the namespace \"kube-system\"",
  "reason": "Forbidden",
  "details": {
    "kind": "pods"
  },
  "code": 403
}
```

我们可以使用错误信息来确认我们的 ServiceAccount 分配和身份验证按预期工作，因为 API 服务器将我们识别为`read-pods` ServiceAccount。然而，我们没有带有正确权限的 RoleBinding 来读取`kube-system`命名空间中的 Pods，因此请求被拒绝。

同样，由于我们仅有 Pods 的权限，我们不能列出我们的 Deployment，尽管它也位于`sample`命名空间中：

```
/run/secrets/kubernetes.io/serviceaccount # curl --cacert ca.crt \
  -H "Authorization: Bearer $TOKEN" \
  https://kubernetes.default.svc/apis/apps/v1/namespaces/sample/deploy
ments
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {
  },
  "status": "Failure",
  "message": "deploy.apps is forbidden: User 
    \"system:serviceaccount:default:read-pods\" cannot list resource 
    \"deploy\" in API group \"apps\" in the namespace \"sample\"",
  "reason": "Forbidden",
  "details": {
    "group": "apps",
    "kind": "deploy"
  },
 "code": 403
}
```

URL 的路径方案略有不同，从*/apis/apps/v1*而非*/api/v1*开始，这是因为 Deployments 位于`apps` API 组中，而不是默认的 API 组中。这个命令失败的原因类似，因为我们没有必要的权限来列出 Deployments。

我们已经完成了这个 shell 会话，接下来让我们退出它：

```
/run/secrets/kubernetes.io/serviceaccount # exit
```

不过，在我们结束 RBAC 话题之前，让我们展示一种简单的方法，为命名空间授予普通用户权限，而不允许任何管理员职能。

#### 将角色绑定到用户

为了授予普通用户权限，我们将利用一个现有的 ClusterRole，名为`edit`，它已经设置好为用户提供大多数资源类型的查看和编辑权限。

让我们快速查看一下`edit` ClusterRole，看看它有哪些权限：

```
root@host01:~# kubectl get clusterrole edit -o yaml
...
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
...
rules:
...
- apiGroups:
  - ""
  resources:
  - pods
  - pods/attach
  - pods/exec
  - pods/portforward
  - pods/proxy
  verbs:
  - create
  - delete
  - deletecollection
  - patch
  - update
...
```

完整的列表包含大量不同的规则，每个规则都有自己的一套权限。此示例中的子集仅展示了一个规则，用于为 Pods 提供编辑权限。

与 Pods 相关的某些命令，如`exec`，被单独列出，以便进行更细粒度的控制。例如，对于生产系统，允许某些人能够创建和删除 Pods 并查看日志，但不提供使用`exec`的权限可能会更有用，因为`exec`可能会被用来访问敏感的生产数据。

之前，我们创建了一个名为*me*的用户，并将客户端证书保存到名为*kubeconfig*的文件中。然而，我们还没有将任何角色绑定到该用户，因此该用户只有自动加入*system:authenticated*组时所拥有的非常有限的权限。

结果正如我们之前看到的那样，我们的普通用户甚至不能列出`default`命名空间中的 Pods。让我们将这个用户绑定到编辑角色。和之前一样，我们将使用常规的 RoleBinding，作用范围限定在`sample`命名空间，这样该用户将无法访问`kube-system`命名空间中的集群基础设施组件。

Listing 11-1 展示了我们需要的 RoleBinding。

*edit-bind.yaml*

```
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: editor
  namespace: sample
subjects:
- kind: User
  name: me
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole 
  name: edit
  apiGroup: rbac.authorization.k8s.io
```

*Listing 11-1: 将编辑角色绑定到用户*

现在我们应用这个 RoleBinding 来为用户添加权限：

```
root@host01:~# kubectl apply -f /opt/edit-bind.yaml
rolebinding.rbac.authorization.k8s.io/editor created
```

现在我们可以使用这个用户查看和修改 Pods、Deployments 和许多其他资源：

```
root@host01:~# KUBECONFIG=kubeconfig kubectl -n sample get pods
NAME                        READY   STATUS    RESTARTS   AGE
read-pods-9d5565548-fbwjb   1/1     Running   0          54m
root@host01:~# KUBECONFIG=kubeconfig kubectl delete -f /opt/read-pods-deploy.yaml
deployment.apps "read-pods" deleted
```

然而，由于我们使用的是 RoleBinding 而不是 ClusterRoleBinding，因此该用户无法查看其他命名空间：

```
root@host01:~# KUBECONFIG=kubeconfig kubectl get -n kube-system pods
Error from server (Forbidden): pods is forbidden: User "me" cannot list 
  resource "pods" in API group "" in the namespace "kube-system"
```

`kubectl` 显示的错误消息与 API 服务器 JSON 响应中的 `message` 字段形式相同。这并非巧合；`kubectl` 是 API 服务器 REST API 前的友好命令行界面。

### 最后的想法

API 服务器是 Kubernetes 控制平面中的一个核心组件。集群中的每个其他服务都会持续连接到 API 服务器，监视集群中的变化，以便采取适当的行动。用户也使用 API 服务器来部署和配置应用程序以及监控状态。在这一章中，我们看到了 API 服务器提供的底层 REST API，用于创建、检索、更新和删除资源。我们还看到了 API 服务器内置的广泛认证和授权功能，确保只有授权的用户和服务可以访问和修改集群状态。

在下一章中，我们将探讨集群基础设施的另一面：节点组件。我们将看到 `kubelet` 服务如何隐藏容器引擎之间的差异，以及它如何使用我们在第一部分中看到的容器功能来创建、启动和配置集群中的容器。
