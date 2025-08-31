# 第十七章：自定义资源和操作员

![image](img/common01.jpg)

我们已经看到，Kubernetes 集群中使用了许多不同的资源类型来运行容器工作负载、扩展它们、配置它们、路由网络流量并为它们提供存储。然而，Kubernetes 集群的一个最强大的功能是能够定义自定义资源类型，并将这些类型与我们已经看到的所有内置资源类型集成到集群中。

自定义资源定义使我们能够定义任何新的资源类型，并让集群跟踪相应的资源。我们可以利用这一能力为集群添加复杂的新行为，例如自动化部署一个高可用的数据库引擎，同时充分利用集群内置资源类型的所有现有功能以及集群控制平面的资源和状态管理。

在本章中，我们将看到自定义资源定义如何工作，以及我们如何利用它们部署 Kubernetes 操作员，从而扩展我们的集群以实现我们所需的任何额外行为。

### 自定义资源

在第六章中，我们讨论了 Kubernetes API 服务器如何提供声明式 API，其中主要操作是创建、读取、更新和删除集群中的资源。声明式 API 具有弹性的优势，因为集群可以跟踪资源的期望状态，并努力确保集群保持在该期望状态。然而，声明式 API 在扩展性方面也具有显著优势。API 服务器提供的操作足够通用，以至于将其扩展到任何类型的资源都很容易。

我们已经看到 Kubernetes 如何利用这种扩展性逐步更新其 API。Kubernetes 不仅能够随着时间的推移支持资源的新版本，还能够将具有新功能的全新资源添加到集群中，同时通过旧资源保持向后兼容性。我们在第七章中讨论了版本 2 的 HorizontalPodAutoscaler 的新功能，以及 Deployment 如何取代 ReplicationController。

我们确实能在使用*CustomResourceDefinitions*时看到这种扩展性的强大。CustomResourceDefinition，或简称 CRD，使我们能够动态地向集群添加任何新的资源类型。我们只需向 API 服务器提供新资源类型的名称和用于验证的规格，API 服务器就会立即允许我们创建、读取、更新和删除该新类型的资源。

CRD 非常有用并且被广泛使用。例如，已经部署到我们集群中的基础设施组件包括 CRD。

**注意**

*本书的示例仓库位于* [`github.com/book-of-kubernetes/examples`](https://github.com/book-of-kubernetes/examples)。*有关设置的详细信息，请参见第 xx 页中的“运行示例”部分。*

让我们来看看已经在我们的集群中注册的 CRD：

```
root@host01:~# kubectl get crds
NAME                                                  CREATED AT
...
clusterinformations.crd.projectcalico.org             ...
...
installations.operator.tigera.io                      ...
...
volumes.longhorn.io                                   ...
```

为了避免命名冲突，CRD 的名称必须包含一个组名，通常基于域名来确保唯一性。这个组名也用于为 API 服务器提供的 REST API 建立到该资源的路径。在这个例子中，我们看到 CRD 属于 `crd.projectcalico.org` 组和 `operator.tigera.io` 组，这两个组都由 Calico 使用。我们还看到一个属于 `longhorn.io` 组的 CRD，这个 CRD 是 Longhorn 使用的。

这些 CRD 允许 Calico 和 Longhorn 使用 Kubernetes API 将配置信息和状态信息记录在 `etcd` 中。CRD 还简化了自定义配置。例如，作为将 Calico 部署到集群的一部分，自动化创建了一个安装资源，对应于 `installations.operator.tigera.io` CRD：

*custom-resources.yaml*

```
---
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  calicoNetwork:
    ipPools:
    - blockSize: 26
      cidr: 172.31.0.0/16
...
```

这个配置是我们看到 Pods 获得 `172.31.0.0/16` 网络块中的 IP 地址的原因。这个 YAML 文件被自动放置在 */etc/kubernetes/components* 中，并作为 Calico 安装的一部分自动应用到集群。当部署时，Calico 会查询 API 服务器，查找此安装资源的实例，并相应地配置网络。

#### 创建 CRD

让我们通过创建自己的 CRD 来进一步探索 CRD。我们将使用列表 17-1 中提供的定义。

*crd.yaml*

```
 ---
 apiVersion: apiextensions.k8s.io/v1
 kind: CustomResourceDefinition
 metadata:
➊ name: samples.bookofkubernetes.com
 spec:
➋ group: bookofkubernetes.com
   versions:
  ➌ - name: v1
       served: true
       storage: true
       schema:
         openAPIV3Schema:
           type: object
           properties:
             spec:
               type: object
               properties:
                 value:
                   type: integer
➍ scope: Namespaced
 names:
➎ plural: samples
➏ singular: sample
➐ kind: Sample
     shortNames:
    ➑ - sam
```

*列表 17-1：示例 CRD*

这个定义包含多个重要部分。首先，定义了几种类型的名称。元数据 `name` 字段 ➊ 必须将资源的复数名称 ➎ 和组 ➋ 组合在一起。这些命名组件对于通过 API 进行访问也至关重要。

命名还包括 `kind` ➐，它在 YAML 文件中使用。这意味着当我们基于这个 CRD 创建特定资源时，我们将使用 `kind: Sample` 来标识它们。最后，我们需要定义如何在命令行中引用这个 CRD 的实例。这包括资源的完整名称，这在 `singular` ➏ 字段中指定，以及任何我们希望命令行识别的 `shortNames` ➑。

现在我们已经根据这个 CRD 为实例提供了所有必要的名称，接下来我们可以讨论 CRD 是如何被跟踪以及它包含了哪些数据。`scope` ➍ 字段告诉 Kubernetes 这个资源应该在 Namespace 级别进行跟踪，还是资源是集群范围的。命名空间资源会收到包含其所在命名空间的 API 路径，可以通过角色（Roles）和角色绑定（RoleBindings）在每个命名空间的基础上控制对命名空间资源的访问和修改权限，正如我们在第十一章中所看到的。

第三，`versions` 部分允许我们定义在基于此 CRD 创建资源时有效的实际内容。为了支持版本更新，可以有多个版本。每个版本都有一个 `schema`，声明哪些字段是有效的。在这个例子中，我们定义了一个 `spec` 字段，其中包含一个名为 `value` 的字段，并且我们声明这个字段的类型为整数。

这里有很多必需的配置，让我们回顾一下结果。这个 CRD 使我们能够告诉 Kubernetes 集群跟踪一种全新的资源类型——*Sample*。这个资源的每个实例（每个 Sample）都将属于一个命名空间，并且在 `value` 字段中包含一个整数。

让我们在集群中创建这个 CRD：

```
root@host01:~# kubectl apply -f /opt/crd.yaml
customresourcedefinition...k8s.io/samples.bookofkubernetes.com created
```

现在我们可以创建此类型的对象，并从集群中获取它们。以下是使用我们定义的 CRD 创建新示例的 YAML 定义示例：

*sample.yaml*

```
---
apiVersion: bookofkubernetes.com/v1
kind: Sample
metadata:
  namespace: default
  name: somedata
spec:
  value: 123
```

我们将 `apiVersion` 和 `kind` 与我们的 CRD 匹配，并确保 `spec` 与 schema 对应。这意味着我们必须提供一个名为 `value` 的字段，并且该字段的值必须是整数。

我们现在可以像创建其他资源一样，在集群中创建这个资源：

```
root@host01:~# kubectl apply -f /opt/somedata.yaml 
sample.bookofkubernetes.com/somedata created
```

现在有一个名为 `somedata` 的示例，它是 `default` 命名空间的一部分。

当我们在 Listing 17-1 中定义 CRD 时，我们为 Sample 资源指定了复数、单数和简短名称。我们可以使用这些名称中的任何一个来检索新资源：

```
root@host01:~# kubectl get samples
NAME       AGE
somedata   56s
root@host01:~# kubectl get sample
NAME       AGE
somedata   59s
root@host01:~# kubectl get sam
NAME       AGE
somedata   62s
```

通过仅声明我们的 CRD，我们就扩展了 Kubernetes 集群的行为，使其能够理解什么是 `samples`，并且我们可以在 API 中以及命令行工具中使用它。

这意味着 `kubectl describe` 也适用于 Samples。我们可以看到 Kubernetes 跟踪了与我们的新资源相关的其他数据，不仅仅是我们指定的数据：

```
root@host01:~# kubectl describe sample somedata
Name:         somedata
Namespace:    default
...
API Version:  bookofkubernetes.com/v1
Kind:         Sample
Metadata:
  Creation Timestamp:  ...
...
  Resource Version:  9386
  UID:               37cc58db-179f-40e6-a9bf-fbf6540aa689
Spec:
  Value:  123
Events:   <none>
```

这些附加数据，包括时间戳和资源版本控制，对于我们想要使用 CRD 中的数据是必不可少的。为了有效地使用我们的新资源，我们需要一个持续监控资源新实例或更新实例的软件组件，并根据情况采取相应的行动。我们将使用一个常规的 Kubernetes Deployment 来运行此组件，并与 Kubernetes API 服务器进行交互。

#### 观察 CRD

对于核心 Kubernetes 资源，控制平面组件通过与 API 服务器通信来采取正确的操作，当资源被创建、更新或删除时。例如，控制器管理器包括一个组件，监视服务和 Pod 的变化，使其能够更新每个服务的端点列表。然后，每个节点上的 `kube-proxy` 实例根据这些端点进行必要的网络路由更改，将流量发送到 Pods。

对于 CRD，API 服务器仅跟踪资源的创建、更新和删除。其他软件负责监视资源实例并采取正确的行动。为了方便监视资源，API 服务器提供了 `watch` 操作，通过 *长轮询* 保持连接打开，并在事件发生时持续推送事件。由于长轮询连接可能会随时中断，Kubernetes 跟踪的时间戳和资源版本数据将使我们能够在重新连接时检测到我们已经处理的集群变化。

我们可以直接从 `curl` 命令或 HTTP 客户端中使用 API 服务器的 `watch` 功能，但使用 Kubernetes 客户端库要容易得多。对于这个示例，我们将使用 Python 客户端库来演示如何监视我们的自定义资源。以下是我们将使用的 Python 脚本：

*watch.py*

```
   #!/usr/bin/env python3
   from kubernetes import client, config, watch
   import json, os, sys

   try:
  ➊ config.load_incluster_config()
   except:
     print("In cluster config failed, falling back to file", file=sys.stderr)
  ➋ config.load_kube_config()

➌ group = os.environ.get('WATCH_GROUP', 'bookofkubernetes.com')
   version = os.environ.get('WATCH_VERSION', 'v1')
   namespace = os.environ.get('WATCH_NAMESPACE', 'default')
   resource = os.environ.get('WATCH_RESOURCE', 'samples')
   api = client.CustomObjectsApi()

   w = watch.Watch()
➍ for event in w.stream(api.list_namespaced_custom_object,
          group=group, version=version, namespace=namespace, plural=resource):
➎ json.dump(event, sys.stdout, indent=2)
    sys.stdout.flush()
```

要连接到 API 服务器，我们需要加载集群配置。这包括 API 服务器的位置以及我们在第十一章中看到的认证信息。如果我们在 Kubernetes Pod 中运行容器，我们将自动获得这些信息，因此我们首先尝试加载集群内配置 ➊。然而，如果我们在 Kubernetes 集群外部，通常会使用 Kubernetes 配置文件作为备选方案 ➋。

在我们建立与 API 服务器的连接方式之后，我们使用自定义对象 API 和一个 watch 对象来流式传输与我们的自定义资源相关的事件 ➍。`stream()` 方法接受一个函数名和相关参数，这些参数我们已经从环境变量或默认值中加载 ➌。我们使用 `list_namespaced_custom_object` 函数，因为我们关心的是我们的自定义资源。Python 库中的所有 `list_*` 方法都设计用于与 `watch` 一起工作，以返回添加、更新和删除事件的流，而不仅仅是检索当前对象列表。当事件发生时，我们会将它们打印到控制台中，格式易于阅读 ➎。

我们将在 Kubernetes 部署中使用这个 Python 脚本。我已经构建并发布了一个容器镜像来运行它，所以这项任务非常简单。以下是部署定义：

*watch.yaml*

```
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: watch
spec:
  replicas: 1
  selector:
    matchLabels:
      app: watch
  template:
    metadata:
      labels:
        app: watch
    spec:
      containers:
      - name: watch
        image: bookofkubernetes/crdwatcher:stable
      serviceAccountName: watcher
```

此部署将运行一个 Python 脚本，监视 Sample CRD 实例上的事件。然而，在我们创建这个部署之前，我们需要确保我们的监视脚本有权限读取我们的自定义资源。默认的 ServiceAccount 权限最小，因此我们需要为此部署创建一个 ServiceAccount，并确保它有权限查看我们的 Sample 自定义资源。

我们本可以将一个自定义 Role 绑定到我们的 ServiceAccount 来实现这一点，但利用角色聚合将我们的 Sample 自定义资源添加到已经存在的 `view` ClusterRole 中会更加方便。这样，集群中任何拥有 `view` ClusterRole 的用户都将获得对我们 Sample 自定义资源的访问权限。

我们首先为我们的自定义资源定义一个新的 ClusterRole：

*sample-reader.yaml*

```
 ---
 apiVersion: rbac.authorization.k8s.io/v1
 kind: ClusterRole
 metadata:
   name: sample-reader
   labels:
  ➊ rbac.authorization.k8s.io/aggregate-to-view: "true"
 rules:
➋ - apiGroups: ["bookofkubernetes.com"]
    resources: ["samples"]
    verbs: ["get", "watch", "list"]
```

这个 ClusterRole 赋予了 `get`、`watch` 和 `list` 我们的 Sample 自定义资源 ➋ 的权限。我们还在元数据 ➊ 中添加了一个标签，向集群指示我们希望这些权限被聚合到 `view` ClusterRole 中。因此，我们不需要将 ServiceAccount 绑定到我们在这里定义的 `sample-reader` ClusterRole，而是可以将 ServiceAccount 绑定到通用的 `view` ClusterRole，从而为它提供对所有资源的只读访问权限。

我们还需要声明 ServiceAccount，并将其绑定到 `view` ClusterRole：

*sa.yaml*

```
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: watcher
  namespace: default
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: viewer
  namespace: default
subjects:
- kind: ServiceAccount
 name: watcher
  namespace: default
roleRef:
  kind: ClusterRole
  name: view
  apiGroup: rbac.authorization.k8s.io
```

我们使用 RoleBinding 来限制该 ServiceAccount 仅对 `default` 命名空间内的资源具有只读访问权限。RoleBinding 将 `watcher` ServiceAccount 绑定到通用的 `view` ClusterRole。由于我们指定的角色聚合，这个 ClusterRole 将可以访问我们的 Sample 自定义资源。

我们现在准备应用所有这些资源，包括我们的 Deployment：

```
root@host01:~# kubectl apply -f /opt/sample-reader.yaml 
clusterrole.rbac.authorization.k8s.io/sample-reader created
root@host01:~# kubectl apply -f /opt/sa.yaml
serviceaccount/watcher created
rolebinding.rbac.authorization.k8s.io/viewer created
root@host01:~# kubectl apply -f /opt/watch.yaml 
deployment.apps/watch created
```

不久之后，我们的监视器 Pod 将开始运行：

```
root@host01:~# kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
watch-69876b586b-jp25m   1/1     Running   0          47s
```

我们可以打印监视器的日志，以查看它从 API 服务器接收到的事件：

```
root@host01:~# kubectl logs watch-69876b586b-jp25m
{
  "type": "ADDED",
  "object": {
    "apiVersion": "bookofkubernetes.com/v1",
    "kind": "Sample",
    "metadata": {
...
      "creationTimestamp": "...",
...
      "name": "somedata",
      "namespace": "default",
      "resourceVersion": "9386",
      "uid": "37cc58db-179f-40e6-a9bf-fbf6540aa689"
 },
    "spec": {
      "value": 123
    }
  },
...
```

请注意，尽管我们在部署监视器之前就创建了 `somedata` Sample，但监视器 Pod 收到了这个 Sample 的 `ADDED` 事件。API 服务器能够确定我们的监视器还没有检索到这个对象，因此在连接时它会立即向我们发送一个事件，就像该对象是新创建的一样，这避免了我们本来需要处理的竞争条件。然而，注意如果客户端被重新启动，它将作为一个新客户端出现在 API 服务器上，并再次看到相同 Sample 的 `ADDED` 事件。因此，在我们实现处理自定义资源的逻辑时，必须确保逻辑是幂等的，以便我们能够多次处理相同的事件。

### 操作员

除了将事件记录到控制台之外，我们还会采取什么样的行动来响应自定义资源的创建、更新或删除呢？正如我们在检查自定义资源如何用于配置集群中 Calico 网络时所看到的，自定义资源的一个用途是配置集群基础设施组件，例如网络和存储。但另一个真正充分利用自定义资源的模式是 Kubernetes 的 *Operator*。

Kubernetes Operator 模式扩展了集群的行为，使得更容易部署和管理特定的应用程序组件。与直接使用 Kubernetes 资源集（如部署和服务）的标准集不同，我们只需创建特定于应用程序组件的自定义资源，操作器将为我们管理底层的 Kubernetes 资源。

让我们看一个示例，以说明 Kubernetes Operator 模式的强大。我们将在集群中添加一个 Postgres Operator，这将使我们能够通过添加单个自定义资源来部署高可用的 PostgreSQL 数据库到我们的集群。

我们的自动化已将所需文件暂存到 */etc/kubernetes/components* 并执行了一些初始设置，所以剩下的唯一步骤就是添加操作器。该操作器是一个普通的部署，将在我们选择的任何命名空间中运行。然后，它将监视自定义 `postgresql` 资源，并相应地创建 PostgreSQL 实例。

让我们部署该操作器：

```
root@host01:~# kubectl apply -f /etc/kubernetes/components/postgres-operator.yaml 
deployment.apps/postgres-operator created
```

这创建了操作器本身的部署，它创建一个单独的 Pod：

```
root@host01:~# kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
postgres-operator-5cdbff85d6-cclxf   1/1     Running   0          27s
...
```

Pod 与 API 服务器通信以创建定义 PostgreSQL 数据库所需的 CRD：

```
root@host01:~# kubectl get crd postgresqls.acid.zalan.do
NAME                        CREATED AT
postgresqls.acid.zalan.do   ...
```

尚未在集群中运行任何 PostgreSQL 实例，但我们可以通过基于该 CRD 创建自定义资源来轻松部署 PostgreSQL：

*pgsql.yaml*

```
---
apiVersion: "acid.zalan.do/v1"
kind: postgresql
metadata:
  name: pgsql-cluster
  namespace: default
spec:
  teamId: "pgsql"
  volume:
    size: 1Gi
    storageClass: longhorn
  numberOfInstances: 3
  users:
    dbuser:
    - superuser
    - createdb
  databases:
    defaultdb: dbuser
  postgresql:
    version: "14"
```

此自定义资源告诉 Postgres Operator 使用服务器版本 14 生成一个 PostgreSQL 数据库，具有三个实例（一个主实例和两个备份）。每个实例都将具有持久存储。主实例将配置为指定的用户和数据库。

Kubernetes Operator 模式的真正价值在于我们声明的 YAML 资源文件简短、简单且明确地与我们想要看到的 PostgreSQL 配置相关联。操作器的工作是将此信息转换为 StatefulSet、Services 和其他集群资源，以便操作此数据库。

我们像处理任何其他资源一样将此自定义资源应用于集群：

```
root@host01:~# kubectl apply -f /opt/pgsql.yaml 
postgresql.acid.zalan.do/pgsql-cluster created
```

我们应用后，Postgres Operator 将接收添加事件，并为 PostgreSQL 创建必要的集群资源：

```
root@host01:~# kubectl logs postgres-operator-5cdbff85d6-cclxf
... level=info msg="Spilo operator..."
...
... level=info msg="ADD event has been queued" 
  cluster-name=default/pgsql-cluster pkg=controller worker=0
... level=info msg="creating a new Postgres cluster" 
  cluster-name=default/pgsql-cluster pkg=controller worker=0
...
... level=info msg="statefulset 
  \"default/pgsql-cluster\" has been successfully created" 
  cluster-name=default/pgsql-cluster pkg=cluster worker=0
...
```

最终，将有一个 StatefulSet 和三个运行的 Pod（除了操作器本身仍在运行的 Pod）：

```
root@host01:~# kubectl get sts
NAME            READY   AGE
pgsql-cluster   3/3     2m39s
root@host01:~# kubectl get po
NAME                                 READY   STATUS    RESTARTS   AGE
pgsql-cluster-0                      1/1     Running   0          2m40s
pgsql-cluster-1                      1/1     Running   0          2m18s
pgsql-cluster-2                      1/1     Running   0          111s
postgres-operator-5cdbff85d6-cclxf   1/1     Running   0          4m6s
...
```

所有这些资源完全在集群上运行可能需要几分钟时间。

与我们在 第十五章 中创建的 PostgreSQL StatefulSet 不同，此 StatefulSet 中的所有实例均配置为高可用性，这可以通过检查每个 Pod 的日志来演示：

```
root@host01:~# kubectl logs pgsql-cluster-0
...
... INFO: Lock owner: None; I am pgsql-cluster-0
... INFO: trying to bootstrap a new cluster
...
... INFO: initialized a new cluster
...
... INFO: no action. I am (pgsql-cluster-0) the leader with the lock
root@host01:~# kubectl logs pgsql-cluster-1
...
... INFO: Lock owner: None; I am pgsql-cluster-1
... INFO: waiting for leader to bootstrap
... INFO: Lock owner: pgsql-cluster-0; I am pgsql-cluster-1
...
... INFO: no action. I am a secondary (pgsql-cluster-1) and following 
    a leader (pgsql-cluster-0)
```

如我们所见，第一个实例 `pgsql-cluster-0` 已将自己标识为领导者，而 `pgsql-cluster-1` 则配置为跟随者，将复制到领导者数据库的任何更新。

为了管理 PostgreSQL 的领导者和跟随者，并使数据库客户端能够访问领导者，操作器已创建了多个服务：

```
root@host01:~# kubectl get svc
NAME                   TYPE        CLUSTER-IP      ... PORT(S)    AGE
...
pgsql-cluster          ClusterIP   10.101.80.163   ... 5432/TCP   6m52s
pgsql-cluster-config   ClusterIP   None            ... <none>     6m21s
pgsql-cluster-repl     ClusterIP   10.96.13.186    ... 5432/TCP   6m52s
```

`pgsql-cluster` 服务只将流量路由到主节点；其他服务用于管理复制到备份实例。操作员会处理在主实例由于故障切换而发生变化时更新服务的任务。

要移除 PostgreSQL 数据库，我们只需要删除自定义资源，其余操作由 Postgres Operator 处理：

```
root@host01:~# kubectl delete -f /opt/pgsql.yaml 
postgresql.acid.zalan.do "pgsql-cluster" deleted
```

操作员会检测到删除操作并清理相关的 Kubernetes 集群资源：

```
root@host01:~# kubectl logs postgres-operator-5cdbff85d6-cclxf
...
... level=info msg="deletion of the cluster started" 
  cluster-name=default/pgsql-cluster pkg=controller worker=0
... level=info msg="DELETE event has been queued" 
  cluster-name=default/pgsql-cluster pkg=controller worker=0
...
... level=info msg="cluster has been deleted" 
  cluster-name=default/pgsql-cluster pkg=controller worker=0
```

Postgres Operator 现在已移除与该数据库集群相关的 StatefulSet、持久存储和其他资源。

我们能够轻松地部署和移除 PostgreSQL 数据库服务器，包括自动配置为高可用性配置的多个实例，这展示了 Kubernetes Operator 模式的强大。通过定义 CRD，常规的部署可以扩展我们的 Kubernetes 集群的行为。结果是无缝地增加了集群的新功能，并且与 Kubernetes 集群的内置功能完全集成。

### 最后的思考

CustomResourceDefinitions 和 Kubernetes Operators 为集群带来高级功能，但它们是通过构建在我们在本书中看到的基本 Kubernetes 集群功能之上的。Kubernetes API 服务器具有处理任何类型集群资源存储和检索的可扩展性。因此，我们能够动态定义新的资源类型，并让集群为我们管理这些资源。

我们在本书的第二部分中已经看到过这种模式。Kubernetes 本身是建立在我们在第一部分中看到的容器基本功能之上的，且它是通过将更基本的功能整合在一起来实现其更高级的功能的。通过理解这些基本功能的工作原理，我们能够更好地理解这些高级功能，即使它们的行为乍一看有点神奇。

我们现在已经了解了构建高质量、高性能应用程序所需掌握的 Kubernetes 关键能力。接下来，我们将关注在 Kubernetes 集群中运行应用时，如何提高应用的性能和弹性。
