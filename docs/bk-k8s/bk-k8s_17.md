# 第十五章：持久存储

![image](img/common01.jpg)

可扩展性和快速故障切换是容器化应用的巨大优势，且扩展、更新和替换没有持久存储的无状态容器要容易得多。因此，我们通常使用部署（Deployments）来创建一个或多个仅具有临时存储的 Pod 实例。

然而，即使我们有一个大多数组件都是无状态的应用架构，我们仍然需要一些持久存储来支持我们的应用。同时，我们不想失去将 Pod 部署到集群中任何节点的能力，也不希望在容器或节点故障时丢失持久存储的内容。

在本章中，我们将看到 Kubernetes 如何通过使用插件架构按需为 Pods 提供持久存储，该架构允许任何支持的分布式存储引擎作为后端存储。

### 存储类

Kubernetes 的存储插件架构高度灵活；它认识到一些集群可能根本不需要存储，而其他集群则需要多个存储插件来处理大量数据或低延迟存储。因此，`kubeadm` 在集群安装时不会立即设置存储；它是在安装后通过向集群添加*StorageClass*资源来配置的。

每个 StorageClass 都标识一个特定的存储插件，该插件将提供实际存储以及任何其他所需的参数。我们可以使用多个存储类来定义不同的插件或参数，甚至使用相同插件但不同参数的多个存储类，以便为不同的用途提供独立的服务类别。例如，一个集群可能提供内存存储、固态硬盘存储和传统的旋转磁盘存储，让应用程序选择最适合特定目的的存储类型。该集群可能为更昂贵且低延迟的存储提供较小的配额，同时为更适合不常访问数据的慢速存储提供较大的配额。

Kubernetes 内置了一组内部存储提供者。这包括支持流行云服务提供商（如 Amazon Web Services、Microsoft Azure 和 Google Container Engine）的存储驱动程序。然而，只要存储插件支持容器存储接口（CSI）这一已发布的标准，就可以轻松使用任何存储插件与存储提供商接口。

当然，为了与 CSI 兼容，存储提供者必须包含一些最低限度的功能，这些功能对于 Kubernetes 集群中的存储至关重要。最重要的功能包括动态存储管理（配置和解除配置）和动态存储附加（在集群中的任何节点上挂载存储）。这两个关键特性使得集群能够为任何请求存储的 Pod 分配存储，并在集群中的任何节点上调度该 Pod，如果现有节点失败或 Pod 被替换，还能在任何节点上启动具有相同存储的新 Pod。

#### 存储类定义

我们在第六章中部署的 Kubernetes 集群包含了 Longhorn 存储插件（请参阅“安装存储”章节 102 页）。自动化脚本已将其安装到集群中，并为后续各章做好了准备。部分安装工作创建了一个 DaemonSet，以确保 Longhorn 组件存在于每个节点上。该 DaemonSet 启动了多个 Longhorn 组件，并创建了一个 StorageClass 资源，告诉 Kubernetes 如何使用 Longhorn 为 Pod 配置存储。

**注意**

*本书的示例仓库在* [`github.com/book-of-kubernetes/examples`](https://github.com/book-of-kubernetes/examples)。*有关如何设置的详细信息，请参见“运行示例”章节 xx 页。*

示例 15-1 显示了 Longhorn 创建的 StorageClass。

```
root@host01:~# kubectl get storageclass
NAME      PROVISIONER         RECLAIMPOLICY  VOLUMEBINDINGMODE  ALLOWVOLUMEEXPANSION ...
longhorn  driver.longhorn.io  Delete         Immediate          true                 ...
```

*示例 15-1：Longhorn StorageClass*

这两个最重要的字段显示了 StorageClass 的名称和供应者。名称用于资源规格中，标识应该使用 Longhorn StorageClass 来配置请求的卷，而供应者则是`kubelet`内部用来与 Longhorn CSI 插件通信的。

#### CSI 插件内部实现

在继续配置卷并将其附加到 Pods 之前，我们先快速了解一下`kubelet`是如何查找并与 Longhorn CSI 插件通信的。请注意，`kubelet`作为服务直接运行在集群节点上；另一方面，所有 Longhorn 组件都被容器化。这意味着二者需要通过在主机文件系统上创建的 Unix 套接字来帮助它们进行通信，然后将该套接字挂载到 Longhorn 容器的文件系统中。Unix 套接字允许两个进程通过流式数据进行通信，类似于网络连接，但没有网络开销。

为了探讨这种通信如何工作，首先我们将列出在`host01`上运行的 Longhorn 容器：

```
root@host01:~# crictl ps --name 'longhorn.*|csi.*'
CONTAINER     ... STATE    NAME ...
c8347a513f71e ... Running  csi-provisioner ...
47f950a3e8dbf ... Running  csi-provisioner ...
3aad0fef7454e ... Running  longhorn-csi-plugin ...
9bfb61f786afa ... Running  csi-snapshotter ...
24a2994a264a1 ... Running  csi-snapshotter ...
7ee4c748b4c02 ... Running  csi-snapshotter ...
8d92886fdacda ... Running  csi-resizer ...
9868014407fe0 ... Running  csi-resizer ...
408d16181af51 ... Running  csi-attacher ...
0c6c341debb0c ... Running  longhorn-driver-deployer ...
ba328a9d0aaf2 ... Running  longhorn-manager ...
c39e5c4fee3bb ... Running  longhorn-ui ...
```

Longhorn 创建的容器名称以`longhorn`或`csi`开头，因此我们使用正则表达式和`crictl`来仅显示这些容器。

让我们获取`csi-attacher`容器的容器 ID，然后检查它，看看它挂载了哪些卷：

```
root@host01:~# CID=$(crictl ps -q --name csi-attacher)
root@host01:~# crictl inspect $CID
{
...
    "mounts": 
      {
        "containerPath": "/csi/",
 ➊ "hostPath": "/var/lib/kubelet/plugins/driver.longhorn.io",
        "propagation": "PROPAGATION_PRIVATE",
        "readonly": false,
        "selinuxRelabel": false
      }
...
      "envs": [
        {
          "key": "ADDRESS",
       ➋ "value": "/csi/csi.sock"
        },
...
}
```

`crictl inspect` 命令返回了容器的很多数据，但在这个示例中我们只展示了相关数据。我们可以看到这个 Longhorn 组件被指示连接到 */csi/csi.sock* ➋，这是容器内的 Unix 套接字挂载点，`kubelet` 用它与存储驱动进行通信。我们还可以看到容器内的 */csi* 实际上是 */var/lib/kubelet/plugins/driver.longhorn.io* ➊。`/var/lib/kubelet/plugins` 是 `kubelet` 查找存储插件的标准位置，当然，*driver.longhorn.io* 是 `provisioner` 字段的值，如 [Listing 15-1 中的 Longhorn StorageClass 所定义。

如果我们查看主机，能够确认这个 Unix 套接字存在：

```
root@host01:~# ls -l /var/lib/kubelet/plugins/driver.longhorn.io
total 0
srwxr-xr-x 1 root root 0 Feb 18 20:17 csi.sock
```

作为第一个字符的 `s` 表示这是一个 Unix 套接字。

### 持久卷

现在我们已经了解了 `kubelet` 如何与外部存储驱动通信，让我们看看如何请求分配存储并将其附加到 Pod。

#### Stateful Sets

在 Pod 中获取存储的最简单方式是使用 StatefulSet（第七章 中描述的一种资源）。像 Deployment 一样，StatefulSet 会创建多个 Pod，这些 Pod 可以分配到任何节点。然而，StatefulSet 还会创建持久存储，以及每个 Pod 和其存储之间的映射。如果某个 Pod 需要被替换，它将被替换为一个具有相同标识符和相同持久存储的新 Pod。

Listing 15-2 展示了一个示例 StatefulSet，它创建了两个带有持久存储的 PostgreSQL Pods。

*pgsql-set.yaml*

```
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 2
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
       ➊ value: "supersecret"
        - name: PGDATA
       ➋ value: /data/pgdata
        volumeMounts:
        - name: postgres-volume
       ➌ mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: postgres-volume
    spec:
      storageClassName: longhorn
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi
```

*Listing 15-2: PostgreSQL StatefulSet*

除了通过环境变量设置密码 ➊，我们还将 `PGDATA` 设置为 */data/pgdata* ➋，这告诉 PostgreSQL 数据库文件应该存储的位置。这与我们作为 StatefulSet 一部分声明的卷挂载相一致，因为那个持久卷将挂载到 */data* ➌。PostgreSQL 容器镜像文档建议将数据库文件配置在挂载点下的子目录中，以避免数据目录的所有权问题。

与 PostgreSQL Pod 的配置分开，我们为 StatefulSet 提供了 `volumeClaimTemplates` 字段。这个字段告诉 StatefulSet 我们希望如何配置持久存储。它包括 StorageClass 的名称和请求的大小，还包括 `ReadWriteOnce` 的 `accessMode`，我们稍后将探讨。StatefulSet 将使用此规范为每个 Pod 分配独立的存储。

如 第七章 中所提到的，这个 StatefulSet 通过 `serviceName` 字段引用了一个 Service，该 Service 用来为 Pods 创建域名。Service 的定义在同一个文件中，具体如下：

*pgsql-set.yaml*

```
---
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  clusterIP: None
  selector:
    app: postgres
```

将 `clusterIP` 字段设置为 `None` 会使其成为一个 *无头服务*，这意味着不会从服务 IP 范围分配 IP 地址，也不会为该 Service 配置 第九章 中描述的负载均衡。这个方法通常用于 StatefulSet。对于 StatefulSet，每个 Pod 都有自己独特的身份和独特的存储。由于服务负载均衡是随机选择目标，因此通常在 StatefulSet 中无效。相反，客户端需要明确选择一个 Pod 实例作为目标。

让我们创建 Service 和 StatefulSet：

```
root@host01:~# kubectl apply -f /opt/pgsql-set.yaml 
service/postgres created
statefulset.apps/postgres created
```

启动 Pods 需要一些时间，因为它们是顺序创建的，一个接一个。它们启动后，我们可以看到它们的名称：

```
root@host01:~# kubectl get pods
NAME         READY   STATUS    RESTARTS   AGE
postgres-0   1/1     Running   0          97s
postgres-1   1/1     Running   0          51s
```

让我们在容器内检查持久化存储：

```
root@host01:~# kubectl exec -ti postgres-0 -- /bin/sh
# findmnt /data
TARGET SOURCE                         FSTYPE OPTIONS
/data  /dev/longhorn/pvc-83becdac-... ext4   rw,relatime
# exit
```

如请求所示，我们看到一个已经挂载在 */data* 的 Longhorn 设备。即使节点失败或 Pod 升级，Kubernetes 仍会保留这个持久化存储。

这个 StatefulSet 还有两个重要的资源需要探索。第一个是我们创建的无头 Service：

```
root@host01:~# kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   54m
postgres     ClusterIP   None         <none>        <none>    19m
```

`postgres` Service 存在，但没有显示集群 IP 地址，因为我们创建它时是一个无头服务。然而，它为关联的 Pods 创建了 DNS 记录，因此我们可以使用它来连接特定的 PostgreSQL Pods，而无需知道 Pod 的 IP 地址。

我们需要使用集群 DNS 来进行查找。最简单的方法是从容器内进行：

```
root@host01:~# kubectl run -ti --image=alpine --restart=Never alpine
If you don't see a command prompt, try pressing enter.
/ #
```

这种形式的 `run` 命令保持在前台并为我们提供一个交互式终端。它还告诉 Kubernetes 在我们退出 shell 时不要尝试重启容器。

在这个容器内部，我们可以通过一个众所周知的名称来引用我们的任何 PostgreSQL Pod：

```
/ # ping -c 1 postgres-0.postgres.default.svc
PING postgres-0.postgres.default.svc (172.31.239.198): 56 data bytes
64 bytes from 172.31.239.198: seq=0 ttl=63 time=0.093 ms
...
/# ping -c 1 postgres-1.postgres.default.svc
PING postgres-1.postgres.default.svc (172.31.239.199): 56 data bytes
64 bytes from 172.31.239.199: seq=0 ttl=63 time=0.300 ms
...
# exit
```

命名约定与我们在 第九章 中看到的 Service 相同，但多了一个主机名前缀来表示 Pod 的名称；在这种情况下，可能是 `postgres-0` 或 `postgres-1`。

另一个重要的资源是 StatefulSet 自动创建的 *PersistentVolumeClaim*。PersistentVolumeClaim 实际上是通过 Longhorn StorageClass 分配存储的：

```
root@host01:~# kubectl get pvc
NAME                         STATUS   VOLUME      ...   CAPACITY   ...
postgres-volume-postgres-0   Bound    pvc-83becdac...   1Gi        ...
postgres-volume-postgres-1   Bound    pvc-0d850889...   1Gi        ...
```

我们用缩写 `pvc` 来代替其全称 `persistentvolumeclaim`。

StatefulSet 使用了 清单 15-2 中 `volumeClaimTemplates` 字段的数据来创建这两个 PersistentVolumeClaims。然而，如果我们删除 StatefulSet，PersistentVolumeClaims 会继续存在：

```
root@host01:~# kubectl delete -f /opt/pgsql-set.yaml 
service "postgres" deleted
statefulset.apps "postgres" deleted
root@host01:~# kubectl get pvc
NAME                         STATUS   VOLUME      ...   CAPACITY   ...
postgres-volume-postgres-0   Bound    pvc-83becdac...   1Gi        ...
postgres-volume-postgres-1   Bound    pvc-0d850889...   1Gi        ...
```

这可以保护我们免于意外删除持久化存储。如果我们再次创建 StatefulSet 并在卷声明模板中保持相同的名称，我们的新 Pods 会重新获得相同的存储。

**高可用 PostgreSQL**

我们已经部署了两个独立的 PostgreSQL 实例，每个实例都有自己的独立持久存储。然而，这只是部署高可用数据库的第一步。我们还需要将其中一个实例配置为主实例，另一个配置为备份实例，配置从主实例到备份实例的复制，以及配置故障切换。我们还需要配置客户端连接到主实例，并在发生故障时切换到新的主实例。幸运的是，我们无需自己进行这些配置。在 第十七章中，我们将看到如何利用自定义资源的强大功能，部署一个 Kubernetes Operator 来自动处理所有这些任务。

StatefulSet 是处理需要多个容器实例并且每个实例都需要独立存储的最佳方式。然而，我们也可以更直接地使用持久卷，这样能让我们对它们如何挂载到 Pod 中有更多控制。

#### 卷和声明

Kubernetes 有两种资源类型：*PersistentVolume* 和 PersistentVolumeClaim。PersistentVolumeClaim 表示对已分配存储的请求，而 PersistentVolume 则包含关于已分配存储的信息。在大多数情况下，这种区别并不重要，我们可以专注于 PersistentVolumeClaim。然而，在两种情况下，区别是很重要的：

+   管理员可以手动创建 PersistentVolume，并将这个 PersistentVolume 直接挂载到 Pod 中。

+   如果在按照 PersistentVolumeClaim 中指定的方式分配存储时出现问题，PersistentVolume 将不会被创建。

为了说明，我们首先从一个自动分配存储的 PersistentVolumeClaim 开始：

*pvc.yaml*

```
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nginx-storage
spec:
  storageClassName: longhorn
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
```

我们将这个 PersistentVolumeClaim 命名为 `nginx-storage`，因为我们接下来会用到它。这个 PersistentVolumeClaim 请求从 `longhorn` 存储类中获取 100MiB 的存储。当我们将这个 PersistentVolumeClaim 应用到集群时，Kubernetes 会调用 Longhorn 存储驱动并分配存储，过程中会创建一个 PersistentVolume：

```
root@host01:~# kubectl apply -f /opt/pvc.yaml 
persistentvolumeclaim/nginx-storage created
root@host01:~# kubectl get pv
NAME         ...  CAPACITY ... STATUS  CLAIM                               STORAGECLASS ...
pvc-0b50e5b4-...  1Gi      ... Bound   default/postgres-volume-postgres-1  longhorn     ...
pvc-ad092ba9-...  1Gi      ... Bound   default/postgres-volume-postgres-0  longhorn     ...
pvc-cb671684-...  100Mi    ... Bound   default/nginx-storage               longhorn     ...
```

缩写 `pv` 是 `persistentvolumes` 的简称。

即使没有 Pod 在使用这个存储，它仍然显示为 `Bound` 状态，因为有一个活动的 PersistentVolumeClaim 绑定了这个存储。

如果我们尝试创建一个没有匹配存储类的 PersistentVolumeClaim，集群将无法创建相应的 PersistentVolume：

*pvc-man.yaml*

```
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: manual
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
```

因为没有名为 `manual` 的 StorageClass，Kubernetes 无法自动创建这个存储：

```
root@host01:~# kubectl apply -f /opt/pvc-man.yaml 
persistentvolumeclaim/manual created
root@host01:~# kubectl get pvc
NAME                         STATUS    ... STORAGECLASS   AGE
manual                       Pending   ... manual         6s
...
root@host01:~# kubectl get pv
NAME                                       ...
pvc-0b50e5b4-9889-4c8d-a651-df78fa2bc764   ...
pvc-ad092ba9-cf30-4b7d-af01-ff02a5924db7   ...
pvc-cb671684-1719-4c33-9dd8-bcbbf24523b4   ...
```

我们的 PersistentVolumeClaim 处于 `Pending` 状态，并且没有相应的 PersistentVolume。然而，作为集群管理员，我们可以手动创建这个 PersistentVolume：

*pv.yaml*

```
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: manual
spec:
  claimRef:
    name: manual
    namespace: default
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 100Mi
  csi:
    driver: driver.longhorn.io
    volumeHandle: manual
```

以这种方式创建 PersistentVolume 时，我们需要指定所需的卷类型。在这种情况下，通过包含 `csi` 字段，我们将其标识为由 CSI 插件创建的卷。然后，我们指定要使用的 `driver` 并为 `volumeHandle` 提供唯一值。在 PersistentVolume 创建后，Kubernetes 会直接调用 Longhorn 存储驱动程序来分配存储。

我们通过以下方式创建 PersistentVolume：

```
root@host01:~# kubectl apply -f /opt/pv.yaml 
persistentvolume/manual created
```

因为我们为这个 PersistentVolume 指定了 `claimRef`，它将自动进入 `Bound` 状态：

```
root@host01:~# kubectl get pv manual
NAME     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   ...
manual   100Mi      RWO            Retain           Bound    ...
```

这将花费几秒钟，因此 PersistentVolume 可能会短暂地显示为 `Available`。

PersistentVolumeClaim 也会进入 `Bound` 状态：

```
root@host01:~# kubectl get pvc manual
NAME     STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
manual   Bound    manual   100Mi      RWO            manual         2m20s
```

对于管理员来说，手动创建 PersistentVolume 在某些特殊情况下非常有用，尤其是当应用程序需要特定存储时。然而，对于大多数持久存储，最好通过 StorageClass 和 PersistentVolumeClaim 或 StatefulSet 来自动化存储分配。

#### Deployments

既然我们已经直接创建了 PersistentVolumeClaim 并且有了相关的卷，我们就可以在 Deployment 中使用它。为了演示这一点，我们将展示如何使用持久存储来保存由 NGINX 网络服务器提供的 HTML 文件：

*nginx.yaml*

```
---
apiVersion: apps/v1
kind: Deployment
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
       ➊ - name: html
            mountPath: /usr/share/nginx/html
      volumes:
     ➋ - name: html
 persistentVolumeClaim:
            claimName: nginx-storage
```

将持久存储挂载到容器中需要两个步骤。首先，我们声明一个名为 `html` ➋ 的 `volume`，该卷引用我们创建的 PersistentVolumeClaim。这样，存储就可以在 Pod 中使用。接下来，我们声明一个 `volumeMount` ➊ 来指定这个特定的卷应该出现在容器的文件系统中的位置。将这两个步骤分开的好处是，我们可以在同一个 Pod 中的多个容器中挂载相同的卷，这使得我们能够在使用文件的情况下，即使这些进程来自不同的容器镜像，也能在进程之间共享数据。

这一功能允许一些有趣的用例。例如，假设我们正在构建一个包含一些静态内容的 web 应用程序。我们可能会部署一个 NGINX 网络服务器来提供这些内容，正如我们在这里所做的那样。同时，我们还需要一种更新内容的方法。我们可以通过在 Pod 中添加一个额外的容器，让它定期检查新内容，并更新与 NGINX 容器共享的持久卷。

让我们创建 NGINX Deployment，以便我们能够展示如何从持久存储中提供 HTML 文件。持久存储将会为空，因此最初不会有任何网络内容可供提供。让我们看看 NGINX 在这种情况下会如何表现：

```
root@host01:~# kubectl apply -f /opt/nginx.yaml 
deployment.apps/nginx created
```

一旦 NGINX 服务器启动并运行，我们需要获取它的 IP 地址，以便使用 `curl` 发出 HTTP 请求：

```
root@host01:~# IP=$(kubectl get po -l app=nginx -o jsonpath='{..podIP}')
root@host01:~# curl -v http://$IP
...
* Connected to 172.31.25.200 (172.31.25.200) port 80 (#0)
> GET / HTTP/1.1
...
< HTTP/1.1 403 Forbidden
```

在这种情况下，为了获取 IP 地址，我们使用 `kubectl` 的 `jsonpath` 输出格式，而不是使用 `jq` 来过滤 JSON 输出；`jsonpath` 提供了一个非常有用的语法，可以在 JSON 对象中进行搜索并提取单个唯一命名的字段（在这个例子中是 `podIP`）。我们也可以使用类似于在第八章中做的 `jq` 过滤器，但 `jq` 的递归语法更为复杂。

获取到 IP 地址后，我们使用 `curl` 来联系 NGINX。正如预期的那样，我们没有看到 HTML 响应，因为我们的持久存储是空的。然而，我们知道我们的卷已经正确挂载，因为在这种情况下，我们甚至没有看到默认的 NGINX 欢迎页面。

让我们复制一个 *index.html* 文件，以便给我们的 NGINX 服务器提供一些内容：

```
root@host01:~# POD=$(kubectl get po -l app=nginx -o jsonpath='{..metadata.name}')
root@host01:~# kubectl cp /opt/index.html $POD:/usr/share/nginx/html
```

首先，我们捕获由部署随机生成的 Pod 名称，然后使用 `kubectl cp` 将一个 HTML 文件复制进去。如果我们再次运行 `curl`，我们将看到一个更好的响应：

```
root@host01:~# curl -v http://$IP
...
* Connected to 172.31.239.210 (172.31.239.210) port 80 (#0)
> GET / HTTP/1.1
...
< HTTP/1.1 200 OK
...
<html>
  <head>
    <title>Hello, World</title>
  </head>
  <body>
    <h1>Hello, World!</h1>
  </body>
</html>
...
```

因为这是持久存储，所以即使我们删除并重新创建部署，这些 HTML 内容仍然可用。

然而，我们仍然有一个重要的问题需要解决。进行部署的主要原因之一是能够扩展到多个 Pod 实例。扩展这个部署是非常有意义的，因为我们可以有多个 Pod 实例来提供相同的 HTML 内容。不幸的是，目前扩展无法正常工作：

```
root@host01:~# kubectl scale --replicas=3 deployment/nginx
deployment.apps/nginx scaled
```

部署似乎已经扩展，但如果我们查看 Pod，我们会发现我们并没有真正拥有多个运行中的实例：

```
root@host01:~# kubectl get pods
NAME                    READY   STATUS              RESTARTS   AGE
...
nginx-db4f4d5d9-7q7rd   0/1     ContainerCreating   0          46s
nginx-db4f4d5d9-gbqxm   0/1     ContainerCreating   0          46s
nginx-db4f4d5d9-vrzr4   1/1     Running             0          10m
```

这两个新实例卡在了 `ContainerCreating` 状态。让我们检查其中一个 Pod，看看原因：

```
root@host01:~# kubectl describe pod/nginx-db4f4d5d9-7q7rd
Name:           nginx-db4f4d5d9-7q7rd
...
Status:         Pending
Events:
  Type     Reason              Age   From                     Message
  ----     ------              ----  ----                     -------
...
  Warning  FailedAttachVolume  110s  attachdetach-controller  Multi-Attach 
    error for volume "pvc-cb671684-1719-4c33-9dd8-bcbbf24523b4" Volume is 
    already used by pod(s) nginx-db4f4d5d9-vrzr4
```

我们创建的第一个 Pod 已经占用了该卷，其他 Pod 无法附加到它，因此它们卡在了 `Pending` 状态。更糟糕的是，这不仅阻止了扩展，还阻止了升级或对部署进行其他配置更改。如果我们更新部署配置，Kubernetes 会尝试在关闭任何旧的 Pod 之前使用新配置启动一个 Pod。新的 Pod 无法附加到卷，因此无法启动，这样旧的 Pod 就永远不会被清理，配置更改也永远不会生效。

我们可以通过几种方式强制更新 Pod。首先，每次我们做出更改时，可以手动删除并重新创建部署。其次，我们可以配置 Kubernetes 使用 `Recreate` 更新策略，在删除旧的 Pod 之前先删除它。我们将在第二十章中更详细地探讨更新策略选项。目前值得注意的是，这仍然无法让我们扩展部署。

如果我们想修复这个问题，以便能够扩展部署，我们需要允许多个 Pod 同时附加到持久卷。我们可以通过更改持久卷的访问模式来实现这一点。

#### 访问模式

Kubernetes 拒绝将多个 Pod 附加到同一个持久卷，因为我们将 PersistentVolumeClaim 配置为 `ReadWriteOnce` 的访问模式。另一种访问模式 `ReadWriteMany` 将允许所有 NGINX 服务器 Pod 同时挂载存储。只有一些存储驱动程序支持 `ReadWriteMany` 访问模式，因为它要求能够管理文件的同时更改，包括动态地将更改传递给集群中的所有节点。

Longhorn 确实支持 `ReadWriteMany`，因此创建一个具有 `ReadWriteMany` 访问模式的 PersistentVolumeClaim 是一个简单的变更：

*pvc-rwx.yaml*

```
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: storage
spec:
  storageClassName: longhorn
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Mi
```

不幸的是，我们无法修改现有的 PersistentVolumeClaim 来更改访问模式。并且在存储仍然被我们的 Deployment 使用时，无法删除 PersistentVolumeClaim。所以我们需要清理所有内容，然后重新部署：

```
root@host01:~# kubectl delete deploy/nginx pvc/storage
deployment.apps "nginx" deleted
persistentvolumeclaim "storage" deleted
root@host01:~# kubectl apply -f /opt/pvc-rwx.yaml 
persistentvolumeclaim/storage created
root@host01:~# kubectl apply -f /opt/nginx.yaml 
deployment.apps/nginx created
```

我们指定 `deploy/nginx` 和 `pvc/storage` 作为要删除的资源。这种标识资源的方式允许我们在同一个命令中操作两个资源。

大约一分钟后，新的 NGINX Pod 将开始运行：

```
root@host01:~# kubectl get pods
NAME                    READY   STATUS      RESTARTS   AGE
...
nginx-db4f4d5d9-6thzs   1/1     Running     0          44s
```

到这个时候，我们需要再次复制 HTML 内容，因为删除 PersistentVolumeClaim 会删除之前的存储：

```
root@host01:~# POD=$(kubectl get po -l app=nginx -o jsonpath='{..metadata.name}')
root@host01:~# kubectl cp /opt/index.html $POD:/usr/share/nginx/html
... no output ...
```

这一次，当我们扩展 NGINX 部署时，额外的两个 Pod 能够挂载存储并开始运行：

```
root@host01:~# kubectl scale --replicas=3 deploy nginx
deployment.apps/nginx scaled
root@host01:~# kubectl get po
NAME                    READY   STATUS      RESTARTS   AGE
...
nginx-db4f4d5d9-2j629   1/1     Running     0          23s
nginx-db4f4d5d9-6thzs   1/1     Running     0          5m19s
nginx-db4f4d5d9-7r5qj   1/1     Running     0          23s
```

所有三个 NGINX Pod 都在提供相同的内容，如果我们获取其中一个新 Pod 的 IP 地址并连接到它，就能看到这一点：

```
root@host01:~# IP=$(kubectl get po nginx-db4f4d5d9-2j629 -o jsonpath='{..podIP}')
root@host01:~# curl http://$IP
<html>
  <head>
    <title>Hello, World</title>
  </head>
  <body>
    <h1>Hello, World!</h1>
  </body>
</html>
```

此时，我们可以使用任何一个 NGINX Pod 来更新 HTML 内容，所有 Pod 都会提供新的内容。我们甚至可以使用一个单独的 CronJob，并配合一个动态更新内容的应用组件，NGINX 会很高兴地提供任何当前的文件。

### 最后的想法

持久存储是构建一个完全功能的应用程序的基本需求。在集群管理员配置了一个或多个存储类之后，应用程序开发人员可以轻松地将持久存储作为其应用部署的一部分动态请求。在大多数情况下，最好的方法是使用 StatefulSet，因为 Kubernetes 会自动为每个 Pod 分配独立的存储，并在故障转移和升级过程中保持 Pod 与存储之间的一对一关系。

与此同时，还有其他存储使用场景，比如多个 Pod 访问相同的存储。我们可以通过直接创建一个 PersistentVolumeClaim 资源，然后在像 Deployment 或 Job 这样的控制器中声明它作为一个卷，轻松处理这些场景。

虽然持久存储是让文件内容对容器可用的有效方式，但 Kubernetes 还有其他强大的资源类型，可以存储配置数据并将其传递给容器，作为环境变量或文件内容。在下一章中，我们将探索如何管理应用程序配置和机密。
