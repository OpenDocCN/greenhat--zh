# 第十六章：配置与机密

![image](img/common01.jpg)

任何高质量的应用程序都设计为可以在运行时注入关键配置项，而不是将其嵌入源代码中。当我们将应用程序组件迁移到容器时，我们需要一种方法来告诉容器运行时需要注入哪些配置信息，以确保我们的应用程序组件按预期行为运行。

Kubernetes 提供了两种主要的资源类型用于注入这些配置信息：ConfigMap 和 Secret。这两种资源在功能上非常相似，但有些许不同的使用场景。

### 注入配置

当我们在第一部分中查看容器运行时时，我们看到可以将环境变量传递给我们的容器。当然，由于 Kubernetes 为我们管理容器运行时，我们首先需要将这些信息传递给 Kubernetes，然后 Kubernetes 再将其传递给容器运行时。

**注意**

*本书的示例代码库位于* [`github.com/book-of-kubernetes/examples`](https://github.com/book-of-kubernetes/examples)。*有关设置的详细信息，请参阅第 xx 页中的“运行示例”。*

对于简单的配置注入，我们可以直接从 Pod 规范中提供环境变量。当我们在第十章创建 PostgreSQL 服务器时，就看到了一个类似的 Pod 示例。下面是一个 PostgreSQL 部署示例，其中的 Pod 规范包含了类似的配置：

*pgsql.yaml*

```
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres
        env:
        - name: POSTGRES_PASSWORD
          value: "supersecret"
```

当我们直接在 Deployment 中提供环境变量时，这些环境变量会直接存储在 YAML 文件中，以及该 Deployment 的集群配置中。以这种方式嵌入环境变量有两个重要问题。首先，我们降低了灵活性，因为我们无法在不更改 Deployment YAML 文件的情况下指定环境变量的新值。其次，密码以明文形式直接显示在 Deployment YAML 文件中。YAML 文件通常会被检查到源代码管理中，因此我们很难充分保护密码。

**GITOPS**

定义 Kubernetes 资源的 YAML 文件之所以经常被提交到源代码管理，是因为这是管理应用程序部署的最佳方式。GitOps 是一种最佳实践，通过这种方式，所有配置都保存在 Git 仓库中。这包括集群配置、额外的基础设施组件，如负载均衡器、入口控制器和存储插件，以及构建、组合和部署应用程序所需的所有信息。GitOps 提供了集群配置变更的日志，避免了随着时间推移可能发生的配置漂移，并确保开发、测试和生产环境之间的一致性。不仅如此，像 FluxCD 和 ArgoCD 这样的 GitOps 工具可以用来监控 Git 仓库的变化，并自动拉取最新配置来更新集群。

首先，我们来看一下如何将配置移出 Deployment；然后我们再考虑如何最好地保护密码。

#### 配置外部化

将配置嵌入到 Deployment 中会使资源定义变得不那么可重用。例如，如果我们想为应用程序的测试版本和生产版本部署 PostgreSQL 服务器，重用相同的 Deployment 可以避免重复，并防止两个版本之间的配置漂移。然而，出于安全考虑，我们不希望在这两个环境中使用相同的密码。

更好的做法是通过将配置存储在单独的资源中并从 Deployment 中引用它来实现配置外部化。为此，Kubernetes 提供了 *ConfigMap* 资源。ConfigMap 指定了一组键值对，可以在指定 Pod 时引用。例如，我们可以这样定义 PostgreSQL 配置：

*pgsql-cm.yaml*

```
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: pgsql
data:
  POSTGRES_PASSWORD: "supersecret"
```

通过将这些配置信息存储在 ConfigMap 中，它不再是 Deployment YAML 文件或 Deployment 集群配置的一部分。

在我们定义好 ConfigMap 后，可以在我们的 Deployment 中引用它，如 示例 16-1 中所示。

*pgsql-ext-cfg.yaml*

```
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres
        envFrom:
        - configMapRef:
            name: pgsql
```

*示例 16-1：带 ConfigMap 的 PostgreSQL*

在 `env` 字段的位置，我们有一个 `envFrom` 字段，用于指定一个或多个 ConfigMap，作为容器的环境变量。ConfigMap 中的所有键值对将成为环境变量。

这与直接在 Deployment 中指定一个或多个环境变量具有相同的效果，但我们的 Deployment 规范现在是可重用的。Deployment 将在其自己的 Namespace 中查找已识别的 ConfigMap，因此我们可以在不同的 Namespaces 中从相同的规范创建多个 Deployments，每个都可以有不同的配置。

这种通过使用命名空间隔离来防止命名冲突的方法，结合我们在第十一章中看到的命名空间范围的安全控制和我们在第十四章中看到的命名空间范围的配额，使得单个集群可以被多个不同的团队用于不同的目的，这个概念被称为*多租户*。

让我们创建这个部署并查看 Kubernetes 如何注入配置。首先，让我们创建实际的部署：

```
root@host01:~# kubectl apply -f /opt/pgsql-ext-cfg.yaml 
deployment.apps/postgres created
```

这个命令成功完成，因为部署已经在集群中创建，但 Kubernetes 无法启动任何 Pod，因为缺少 ConfigMap：

```
root@host01:~# kubectl get pods
NAME                       READY  STATUS                      RESTARTS  AGE
postgres-6bf595fcbc-s8dqz  0/1    CreateContainerConfigError  0         53s
```

如果我们现在创建 ConfigMap，我们会看到 Pod 被创建：

```
root@host01:~# kubectl apply -f /opt/pgsql-cm.yaml 
configmap/pgsql created
root@host01:~# kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
postgres-6bf595fcbc-s8dqz   1/1     Running   0          2m41s
```

Kubernetes 可能需要一分钟左右的时间来确定 ConfigMap 是否可用并启动 Pod。一旦 Pod 启动，我们可以验证环境变量是否根据 ConfigMap 中的数据被注入：

```
root@host01:~# kubectl exec -ti postgres-6bf595fcbc-s8dqz -- /bin/sh -c env
...
POSTGRES_PASSWORD=supersecret
...
```

`env` 命令会打印出与进程关联的所有环境变量。因为 Kubernetes 向我们的 `/bin/sh` 进程提供了与主 PostgreSQL 进程相同的环境变量，所以我们知道环境变量已经按预期设置。然而，值得注意的是，即使我们可以随时更改 ConfigMap，这样做也不会导致部署更新其 Pods；应用程序不会自动获取任何环境变量的变化。相反，我们需要对部署进行一些配置更改，促使它创建新的 Pods。

尽管配置已经被外部化，但我们仍然没有保护它。接下来我们来做这个操作。

#### 保护机密

在保护机密数据时，思考保护措施的性质非常重要。例如，我们可能需要保护我们的应用程序用来连接数据库的身份验证信息。然而，鉴于应用程序本身需要这些信息才能建立连接，任何能够检查应用程序内部细节的人都会能够提取这些凭证。

正如我们在第十一章中看到的，Kubernetes 对每种资源类型提供细粒度的访问控制。为了保护机密数据，Kubernetes 提供了一个单独的资源类型，*Secret*。通过这种方式，只有那些需要访问的用户才能访问机密数据，这一原则被称为*最小权限*。

Secret 资源类型的另一个优点是，它对所有数据使用 base64 编码，并在数据提供给 Pod 时自动解码，这简化了二进制数据的存储。

**加密机密数据**

默认情况下，存储在 Secret 中的数据是 base64 编码的，但没有加密。可以加密密钥数据，且在生产集群中这样做是一个良好的实践，但请记住，数据必须解密才能提供给 Pod。因此，任何能够控制某个命名空间中 Pod 存在的人都能访问 Secret 数据，任何能够访问底层容器运行时的集群管理员也能访问。这一点即便是 Secret 数据在存储时进行了加密也同样成立。适当的访问控制对于保持集群的安全至关重要。

Secret 的定义几乎与 ConfigMap 的定义完全相同：

*pgsql-secret.yaml*

```
---
kind: Secret
apiVersion: v1
metadata:
  name: pgsql
stringData:
  POSTGRES_PASSWORD: "supersecret"
```

唯一明显的区别是 Secret 的资源类型，而不是 ConfigMap。然而，也有一个微妙的差别。当我们定义这个 Secret 时，我们将键值对放置在一个名为 `stringData` 的字段中，而不是仅仅使用 `data`。这告诉 Kubernetes 我们提供的是未编码的字符串。当 Kubernetes 创建 Secret 时，它会为我们编码这些字符串：

```
root@host01:~# kubectl apply -f /opt/pgsql-secret.yaml 
secret/pgsql created
root@host01:~# kubectl get secret pgsql -o json | jq .data
{
  "POSTGRES_PASSWORD": "c3VwZXJzZWNyZXQ="
}
```

即使我们使用字段 `stringData` 并提供了未编码的字符串来指定数据，实际的 Secret 仍然使用字段 `data` 并使用 base64 编码存储值。我们也可以自己进行 base64 编码。在这种情况下，我们直接将值放入 `data` 字段：

*pgsql-secret-2.yaml*

```
---
kind: Secret
apiVersion: v1
metadata:
  name: pgsql
data:
  POSTGRES_PASSWORD: c3VwZXJzZWNyZXQ=
```

这种方法对于定义 Secret 的二进制内容是必要的，以便我们能够将该二进制内容作为 YAML 资源定义的一部分提供。

我们在 Deployment 定义中使用 Secret 的方式与使用 ConfigMap 完全相同：

*pgsql-ext-sec.yaml*

```
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres
        envFrom:
        - secretRef:
            name: pgsql
```

唯一的变化是用 `secretRef` 代替了 `configMapRef`。

为了测试这个，我们可以应用这个新的 Deployment 配置：

```
root@host01:~# kubectl apply -f /opt/pgsql-ext-sec.yaml 
deployment.apps/postgres configured
```

从我们的 Pod 的角度来看，行为完全相同。Kubernetes 处理 base64 解码，使解码后的值对我们的 Pod 可见：

```
root@host01:~# kubectl get pods
NAME                        READY   STATUS        RESTARTS   AGE
postgres-6bf595fcbc-s8dqz   1/1     Terminating   0          12m
postgres-794ff85bbf-xzz49   1/1     Running       0          26s
root@host01:~# kubectl exec -ti postgres-794ff85bbf-xzz49 -- /bin/sh -c env
...
POSTGRES_PASSWORD=supersecret
...
```

如之前所示，我们使用 `env` 命令来验证 `POSTGRES_PASSWORD` 环境变量是否按预期设置。无论我们是直接指定环境变量，还是使用 ConfigMap 或 Secret，Pod 都会看到相同的行为。

在继续之前，让我们删除这个 Deployment：

```
root@host01:~# kubectl delete deploy postgres
deployment.apps "postgres" deleted
```

使用 ConfigMap 和 Secret，我们可以将应用程序的环境变量配置外部化，从而使我们的 Deployment 规范可重用，并便于对密钥数据进行精细化访问控制。

### 注入文件

当然，环境变量并不是我们常见的唯一配置应用程序的方式。我们还需要一种方式来提供配置文件。我们可以使用我们已经看到的相同的 ConfigMap 和 Secret 资源来实现。

以这种方式注入的任何文件都会覆盖容器镜像中存在的文件，这意味着我们可以为容器镜像提供一个合理的默认配置，然后通过每次运行容器来覆盖该配置。这大大简化了容器镜像的重用。

能够在 ConfigMap 中指定文件内容，然后将其挂载到容器中，立即对配置文件非常有用，但我们也可以利用它更新我们在第十五章中展示的 NGINX web 服务器示例。正如我们将看到的，通过这个版本，我们可以仅使用 Kubernetes 资源的 YAML 文件来声明 HTML 内容，而无需通过控制台命令将内容复制到 PersistentVolume 中。

第一步是定义一个包含我们想要提供的 HTML 内容的 ConfigMap：

*nginx-cm.yaml*

```
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx
data:
  index.html: |
    <html>
      <head>
        <title>Hello, World</title>
      </head>
      <body>
        <h1>Hello, World from a ConfigMap!</h1>
      </body>
    </html>
```

键值对中的关键部分用于指定所需的文件名，在这种情况下是*index.html*。为了便于阅读，我们使用管道字符(`|`)来开始 YAML 多行字符串。只要后续行保持缩进，或者直到 YAML 文件结束，这个字符串就会继续。我们可以通过添加更多的键来定义多个文件。

在我们在清单 16-1 中看到的部署中，我们将 ConfigMap 指定为环境变量的来源。在这里，我们将它指定为卷挂载的来源：

*nginx-deploy.yaml*

```
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        volumeMounts:
        - name: nginx-files
          mountPath: /usr/share/nginx/html
      volumes:
        - name: nginx-files
          configMap:
            name: nginx
```

这个卷定义看起来与我们在第十五章中看到的类似。和之前一样，卷规范分为两部分。`volume`字段指定了卷的来源，在这种情况下是 ConfigMap。`volumeMounts`让我们指定容器中文件应该挂载到的路径。除了使我们能够在 Pod 中的多个容器之间使用相同的卷外，这还意味着我们在挂载持久卷和将配置作为文件挂载到容器文件系统时可以共享相同的语法。

我们先创建 ConfigMap，然后启动这个部署：

```
root@host01:~# kubectl apply -f /opt/nginx-cm.yaml 
configmap/nginx created
root@host01:~# kubectl apply -f /opt/nginx-deploy.yaml
deployment.apps/nginx created
```

在 Pod 运行后，我们可以看到文件内容符合预期，NGINX 正在服务我们的 HTML 文件：

```
root@host01:~# IP=$(kubectl get po -l app=nginx -o jsonpath='{..podIP}')
root@host01:~# curl http://$IP
<html>
  <head>
    <title>Hello, World</title>
  </head>
  <body>
    <h1>Hello, World from a ConfigMap!</h1>
  </body>
</html>
```

输出看起来与我们在第十五章中看到的相似，当时我们将 HTML 内容提供为 PersistentVolume，但我们能够避免附加 PersistentVolume 并将内容复制到其中的工作。实际上，这两种方法都有其价值，因为维护一个包含大量数据的 ConfigMap 会显得笨重。

为了让 ConfigMap 的内容作为文件出现在目录中，Kubernetes 会将 ConfigMap 的内容写到主机文件系统中，然后将该目录从主机挂载到容器中。这意味着特定目录会作为`mount`命令在容器内输出的一部分显示：

```
root@host01:~# kubectl exec -ti nginx-58bc54b5cd-4lbkq -- /bin/mount
...
/dev/sda1 on /usr/share/nginx/html type ext4 (ro,relatime)
...
```

`mount`命令报告显示，目录*/usr/share/nginx/html*是一个来自主机主硬盘*/dev/sda1*的单独挂载路径。

我们已经完成了 NGINX 的部署，接下来删除它：

```
root@host01:~# kubectl delete deploy nginx
deployment.apps "nginx" deleted
```

接下来，让我们看看 ConfigMap 和 Secret 信息在典型的 Kubernetes 集群中是如何存储的，这样我们就可以看到`kubelet`从哪里获取这些内容。

### 集群配置仓库

虽然可以选择不同的配置仓库来运行 Kubernetes 集群，但大多数 Kubernetes 集群使用`etcd`作为所有集群配置数据的后端存储。这不仅包括 ConfigMap 和 Secret 存储，还包括所有其他集群资源和当前集群状态。Kubernetes 还使用`etcd`来在多个 API 服务器的高可用配置下选举领导者。

尽管`etcd`通常是稳定和可靠的，但节点故障可能导致`etcd`集群无法重新建立并选举出领导者。我们展示`etcd`的目的不仅仅是为了查看配置数据如何存储，还旨在提供一些有价值的背景信息，帮助管理员在需要调试时理解这一重要的集群组件。

对于我们所有的示例集群，`etcd`与 API 服务器安装在同一节点上，这在小型集群中是很常见的。在大型集群中，将`etcd`运行在独立的节点上，使其可以与 Kubernetes 控制平面分开扩展，这也是常见的做法。

为了探索`etcd`后端存储的内容，我们将使用`etcdctl`，这是一个为控制和排查`etcd`问题而设计的命令行客户端。

#### 使用`etcdctl`

我们需要告诉`etcdctl`我们的`etcd`服务器实例位于何处，以及如何进行认证。为了认证，我们将使用与 API 服务器相同的客户端证书。

为了方便起见，我们可以设置`etcdctl`将读取的环境变量，这样我们就不必在每个命令中通过命令行传递这些值。

这里是我们需要的环境变量：

*etcd-env*

```
export ETCDCTL_API=3
export ETCDCTL_CACERT=/etc/kubernetes/pki/etcd/ca.crt
export ETCDCTL_CERT=/etc/kubernetes/pki/apiserver-etcd-client.crt
export ETCDCTL_KEY=/etc/kubernetes/pki/apiserver-etcd-client.key
export ETCDCTL_ENDPOINTS=https://192.168.61.11:2379
```

这些变量配置`etcdctl`如下：

ETCDCTL_API 使用`etcd` API 的版本 3。对于近期的`etcd`版本，仅支持版本 3。

ETCDCTL_CACERT 使用提供的证书授权验证`etcd`主机。

ETCDCTL_CERT 使用此证书认证到`etcd`。

ETCDCTL_KEY 使用这个私钥认证到`etcd`。

ETCDCTL_ENDPOINTS 通过此 URL 连接到`etcd`。尽管`etcd`运行在所有三个节点上，我们只需要与其中一个节点进行通信。

在我们的示例中，这些环境变量方便地存储在*/opt*中的一个脚本中，以便我们加载它们并用于后续命令：

```
root@host01:~# source /opt/etcd-env
```

现在我们可以使用`etcdctl`命令来检查集群及其存储的配置数据。我们先从仅列出集群成员开始：

```
root@host01:~# etcdctl member list
45a2b6125030fdde, started, host02, https://192.168.61.12:2380, https://192.168.61.12:2379
91007aab9448ce27, started, host03, https://192.168.61.13:2380, https://192.168.61.13:2379
bf7b9991d532ba78, started, host01, https://192.168.61.11:2380, https://192.168.61.11:2379
```

如预期的那样，每个控制平面节点都有一个`etcd`实例。对于高可用配置，我们需要至少运行三个实例，并且需要大多数实例运行才能保证集群的健康。这个`etcdctl`命令是判断集群是否有故障节点的第一步。

只要集群保持健康，我们就可以存储和检索数据。在`etcd`中，信息是以键值对的形式存储的。键是作为路径在层次结构中指定的。我们可以列出有内容的路径：

```
root@host01:~# etcdctl get / --prefix --keys-only
...
/registry/configmaps/default/nginx
/registry/configmaps/default/pgsql
...
/registry/secrets/default/pgsql
...
```

`--prefix` 标志告诉 `etcdctl` 获取所有以 `/` 开头的键，而 `--keys-only` 确保我们只打印出键，防止数据过载。然而，仍然会返回大量信息，包括我们在本书中描述的所有 Kubernetes 资源类型。还包括我们刚刚创建的 ConfigMaps 和 Secrets。

#### 解密 `etcd` 中的数据

我们通常可以依赖 Kubernetes 将正确的配置信息存储在 `etcd` 中，并且可以依赖 `kubectl` 查看当前的集群配置。然而，了解底层数据存储的工作原理是很有用的，以防我们需要在集群故障或异常状态时检查配置。

为了节省存储空间和带宽，`etcd` 和 Kubernetes 都使用 `protobuf` 库，这是一个语言中立的二进制数据格式。由于我们正在使用 `etcdctl` 从 `etcd` 获取数据，我们可以要求它以 JSON 格式返回数据；然而，JSON 数据将包含一个嵌入的 `protobuf` 结构，其中包含 Kubernetes 的数据，因此我们还需要解码它。

让我们首先检查 `etcd` 中 Kubernetes Secret 的 JSON 格式。我们将通过 `jq` 进行格式化输出：

```
root@host01:~# etcdctl -w json get /registry/secrets/default/pgsql | jq
{
 "header": {
...
  },
  "kvs": [
    {
      "key": "L3JlZ2lzdHJ5L3NlY3JldHMvZGVmYXVsdC9wZ3NxbA==",
      "create_revision": 14585,
      "mod_revision": 14585,
      "version": 1,
      "value": "azhzAAoMCgJ2MRIGU2..."
    }
  ],
  "count": 1
}
```

`kvs` 字段包含 Kubernetes 为此 Secret 存储的键值对。该键的值是一个简单的 base64 编码字符串：

```
root@host01:~# echo $(etcdctl -w json get /registry/secrets/default/pgsql \
| jq -r '.kvs[0].key' | base64 -d)
/registry/secrets/default/pgsql
```

我们使用 `jq` 提取键的值，并以原始格式（无引号）返回，然后使用 `base64` 解码该字符串。

当然，这个键值对中有趣的部分是值，因为它包含了实际的 Kubernetes Secret。尽管该值也是 base64 编码的，但我们需要做更多的解开处理才能访问其信息。

在解码 base64 值后，我们将得到一个 `protobuf` 消息。然而，它有一个 Kubernetes 使用的魔术前缀，以允许未来存储格式的更改。如果我们查看解码值的前几个字节，就可以看到该前缀：

```
root@host01:~# etcdctl -w json get /registry/secrets/default/pgsql \
| jq -r '.kvs[0].value' | base64 -d | head --bytes=10 | xxd
00000000: 6b38 7300 0a0c 0a02 7631                 k8s.....v1
```

我们使用 `head` 获取解码值的前 10 个字节，然后使用 `xxd` 查看十六进制转储。前几个字节是 `k8s`，后跟一个 ASCII 空字符。从第 5 字节开始的其余数据是实际的 `protobuf` 消息。

让我们再运行一个命令，使用 `protoc` 工具实际解码 `protobuf` 消息：

```
root@host01:~# etcdctl -w json get /registry/secrets/default/pgsql \
| jq -r '.kvs[0].value' | base64 -d | tail --bytes=+5 | protoc --decode_raw
1 {
  1: "v1"
  2: "Secret"
}
2 {
  1 {
    1: "pgsql"
    2: ""
    3: "default"
    4: ""
...
  }
  2 {
    1: "POSTGRES_PASSWORD"
    2: "supersecret"
  }
  3: "Opaque"
}
...
```

`protoc` 工具主要用于生成源代码来读取和写入 `protobuf` 消息，但它在消息解码方面也非常有用。正如我们所看到的，在 `protobuf` 消息中包含了 Kubernetes 为此 Secret 存储的所有数据，包括资源版本和类型、资源名称和命名空间，以及数据。这说明，如前所述，访问 Kubernetes 运行的主机就可以访问集群中的所有密钥数据。即使我们将 Kubernetes 配置为在存储到 `etcd` 之前加密数据，密钥本身也需要以未加密的形式存储在 `etcd` 中，以便 API 服务器可以使用它们。

### 最后的思考

通过为 Pods 提供环境变量或文件的能力，ConfigMaps 和 Secrets 使我们能够将容器的配置外部化，这使得我们可以在各种应用程序中重用 Kubernetes 资源定义，例如 Deployments 和容器镜像。

同时，我们需要意识到 Kubernetes 是如何存储这些配置数据的，以及它是如何将这些数据提供给容器的。任何拥有正确角色的人都可以使用`kubectl`访问配置数据；任何可以访问运行容器的主机的人都可以从容器运行时访问这些数据；任何拥有正确认证信息的人都可以直接从`etcd`中访问它。对于生产集群，确保这些机制的安全性至关重要。

到目前为止，我们已经看到 Kubernetes 如何在 `etcd` 中存储内建的集群资源数据，但 Kubernetes 也可以存储我们可能选择声明的任何自定义资源数据。在下一章中，我们将探讨自定义资源定义如何使我们能够通过运维工具在 Kubernetes 集群中添加新的行为。
