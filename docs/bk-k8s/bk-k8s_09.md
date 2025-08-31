# 第七章：将容器部署到 Kubernetes

![image](img/common01.jpg)

现在我们已准备好在工作中的 Kubernetes 集群上运行容器。由于 Kubernetes 提供声明式 API，我们将创建各种资源类型来运行它们，并且会监控集群以查看 Kubernetes 对每种资源类型的处理方式。

不同的容器有不同的使用场景。有些容器可能需要多个相同的实例，并具备自动扩缩容功能，以在负载下表现良好。其他容器可能仅用于执行一次性命令。还有一些容器可能需要固定的顺序，以便选择单个主实例，并提供受控的故障转移到副实例。Kubernetes 为这些使用场景提供了不同的 *控制器* 资源类型。我们将依次查看每个控制器，但我们将从最基本的资源——*Pod* 开始，它被所有这些使用场景所利用。

### Pods

Pod 是 Kubernetes 中最基本的资源，是我们运行容器的方式。每个 Pod 可以包含一个或多个容器。Pod 用于提供我们在第二章中看到的进程隔离。Linux 内核命名空间在 Pod 和容器级别得到应用：

mnt 挂载点：每个容器都有自己的根文件系统；其他挂载点对 Pod 中的所有容器都可用。

uts Unix 时间共享：在 Pod 级别进行隔离。

ipc 进程间通信：在 Pod 级别进行隔离。

pid 进程标识符：在容器级别进行隔离。

net 网络：在 Pod 级别进行隔离。

这种方式的最大优势是多个容器可以像同一虚拟主机上的进程一样工作，使用 `localhost` 地址进行通信，同时基于独立的容器镜像。

#### 部署 Pod

为了开始使用，让我们直接创建一个 Pod。与上一章中我们使用 `kubectl run` 自动生成 Pod 规格不同，这次我们将直接使用 YAML 文件进行指定，以便完全控制 Pod，并为以后使用控制器创建 Pods 做好准备，从而提供可扩展性和故障转移能力。

**注意**

*本书的示例仓库在* [`github.com/book-of-kubernetes/examples`](https://github.com/book-of-kubernetes/examples)。*有关设置详细信息，请参见“运行示例”部分，位于第 xx 页。*

本章的自动化脚本执行完整的集群安装，包含三个节点，运行控制平面和常规应用，提供最小的高可用集群用于测试。自动化还会创建一些 Kubernetes 资源的 YAML 文件。以下是一个基本的 YAML 资源，用于创建运行 NGINX 的 Pod：

*nginx-pod.yaml*

```
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
```

Pod 是 *核心* Kubernetes API 的一部分，因此我们只需为 `apiVersion` 指定 `v1` 的版本号。指定 `Pod` 作为 `kind` 可以告诉 Kubernetes 我们在 API 组中创建的资源类型。我们将在所有 Kubernetes 资源中看到这些字段。

`metadata`字段有许多用途。对于 Pod，我们只需要提供一个必需的字段——`name`。我们没有在 metadata 中指定`namespace`，因此默认情况下，这个 Pod 将被放入`default`命名空间。

剩下的字段`spec`告诉 Kubernetes 运行此 Pod 所需的一切。目前，我们提供的是最基本的信息，即要运行的容器列表，但还有许多其他选项可供选择。在这种情况下，我们只有一个容器，因此我们只提供 Kubernetes 应该使用的容器名称和镜像。

让我们将这个 Pod 添加到集群中。自动化将文件添加到了*/opt*，因此我们可以在`host01`上按如下方式操作：

```
root@host01:~# kubectl apply -f /opt/nginx-pod.yaml
pod/nginx created
```

在清单 7-1 中，我们可以查看 Pod 的状态。

```
root@host01:~# kubectl get pods -o wide
NAME    READY   STATUS    RESTARTS   AGE     IP               NODE   ...
nginx   1/1     Running   0          2m26s   172.31.25.202   host03 ...
```

*清单 7-1：NGINX 状态*

在 Pod 显示为`Running`之前可能需要一些时间，特别是如果你刚刚设置了 Kubernetes 集群，它仍在忙于部署核心组件。不断尝试这个`kubectl`命令以检查状态。

为了避免多次输入`kubectl`命令，你也可以使用`watch`。`watch`命令是观察集群随时间变化的一个好方法。只需在命令前加上`watch`，它将每两秒钟自动执行一次。

我们在命令中添加了`-o wide`选项，以查看此 Pod 的 IP 地址和节点分配。Kubernetes 会为我们管理这些信息。在这种情况下，Pod 被调度到了`host03`，所以我们需要去那里查看正在运行的容器：

```
root@host03:~# crictl pods --name nginx
POD ID         CREATED         STATE  NAME   NAMESPACE  ...
9f1d6e0207d7e  19 minutes ago  Ready  nginx  default    ...
```

在 NGINX Pod 所在的主机上运行此命令。

如果我们收集了 Pod ID，我们还可以看到容器：

```
root@host03:~# POD_ID=$(crictl pods -q --name nginx)
root@host03:~# crictl ps --pod $POD_ID
CONTAINER      IMAGE          CREATED         STATE    NAME   ...
9da09b3671418  4cdc5dd7eaadf  20 minutes ago  Running  nginx  ...
```

这个输出看起来非常类似于清单 7-1 中的`kubectl get`命令输出，这并不令人惊讶，因为我们的集群是通过在该节点上运行的`kubelet`服务获取这些信息的，而`kubelet`服务又使用与`crictl`相同的容器运行时接口（CRI）API 与容器引擎进行通信。

#### Pod 详情和日志

使用`crictl`与底层容器引擎一起探查集群中运行的容器非常有价值，但它确实要求我们连接到运行该容器的特定主机。大多数时候，我们可以通过使用`kubectl`命令连接到集群的 API 服务器，从任何地方检查 Pod，从而避免这一点。让我们回到`host01`，进一步探查 NGINX Pod。

在第六章中，我们看到如何使用`kubectl describe`来查看集群节点的状态和事件日志。我们可以使用相同的命令查看其他 Kubernetes 资源的状态和配置详情。以下是我们 NGINX Pod 的事件日志：

```
 root@host01:~# kubectl describe pod nginx
 Name:         nginx
 Namespace: ➊ default 
 ...
 Containers:
   nginx:
     Container ID:   containerd://9da09b3671418...
 ...
➋ Type    Reason     Age   From               Message
   ----    ------     ----  ----               -------
   Normal  Scheduled  22m   default-scheduler  Successfully assigned ...
   Normal  Pulling    22m   kubelet            Pulling image "nginx"
   Normal  Pulled     21m   kubelet            Successfully pulled image ...
   Normal  Created    21m   kubelet            Created container nginx
   Normal  Started    21m   kubelet            Started container nginx
```

我们可以使用`kubectl describe`查看许多不同的 Kubernetes 资源，因此我们首先告诉`kubectl`我们关注的是一个 Pod，并提供 Pod 的名称。因为我们没有指定命名空间，Kubernetes 将默认在`default`命名空间中查找该 Pod ➊。

**注意**

*我们在本书中的大多数示例使用默认命名空间，以减少输入，但使用多个命名空间来将应用分开是一个好习惯，这样可以避免命名冲突并管理访问控制。我们将在 第十一章 中更详细地讨论命名空间。*

`kubectl describe` 命令的输出提供了事件日志 ➋，这是在启动容器遇到问题时，第一个需要查看的地方。

Kubernetes 在部署容器时需要经过几个步骤。首先，它需要将容器调度到一个节点上，这要求该节点可用且具备足够的资源。然后，控制权转交给该节点上的`kubelet`，它需要与容器引擎交互，拉取镜像，创建容器并启动它。

容器启动后，`kubelet` 会收集标准输出和标准错误。我们可以使用 `kubectl logs` 命令查看这些输出：

```
root@host01:~# kubectl logs nginx
...
2021/07/13 22:37:03 [notice] 1#1: start worker processes
2021/07/13 22:37:03 [notice] 1#1: start worker process 33
2021/07/13 22:37:03 [notice] 1#1: start worker process 34
```

`kubectl logs` 命令始终指向一个 Pod，因为 Pod 是运行容器的基本资源，而我们的 Pod 只有一个容器，所以我们只需要将 Pod 的名称作为一个参数传递给 `kubectl logs`。和之前一样，Kubernetes 会在 `default` 命名空间中查找，因为我们没有指定命名空间。

即使容器已经退出，容器输出仍然可用，因此如果容器被拉取并成功启动后崩溃，`kubectl logs` 命令是查看日志的地方。当然，我们希望容器打印出一条日志消息，解释为何崩溃。在 第十章 中，我们将讨论如果容器无法启动且没有日志消息时该怎么办。

我们已经完成了 NGINX Pod 的操作，现在让我们清理它：

```
root@host01:~# kubectl delete -f /opt/nginx-pod.yaml
pod "nginx" deleted
```

我们可以使用相同的 YAML 配置文件删除 Pod，这在我们将多个 Kubernetes 资源定义在同一个文件中时非常方便，因为一个命令就能删除所有资源。`kubectl` 命令使用文件中定义的每个资源的名称来执行删除操作。

### 部署

要运行一个容器，我们需要一个 Pod，但这并不意味着我们通常希望直接创建 Pod。当我们直接创建 Pod 时，我们无法获得 Kubernetes 提供的可扩展性和故障转移功能，因为 Kubernetes 只会运行 Pod 的一个实例。这个 Pod 只会在创建时分配给一个节点，即使该节点发生故障，也不会重新分配。

为了获得可扩展性和故障转移，我们需要创建一个控制器来管理 Pod。我们将介绍多种可以运行 Pods 的控制器，但让我们先从最常见的 *Deployment* 开始。

#### 创建一个 Deployment

Deployment 管理一个或多个 *完全相同* 的 Kubernetes Pods。当我们创建一个 Deployment 时，我们提供一个 Pod 模板。Deployment 然后借助 *ReplicaSet* 创建与该模板匹配的 Pods。

**DEPLOYMENTS 和 REPLICASETS**

Kubernetes 随着时间的发展，逐步演化了其控制器资源。第一种类型的控制器，*ReplicationController*，仅提供了基本功能。它被 ReplicaSet 所取代，后者在识别要管理的 Pod 方面进行了改进。

替换 ReplicationControllers 为 ReplicaSets 的部分原因是 ReplicationControllers 变得越来越复杂，使得代码难以维护。新的方法将控制器的责任分拆给 ReplicaSets 和 Deployments。ReplicaSets 负责基本的 Pod 管理，包括监控 Pod 状态和执行故障切换。Deployments 则负责跟踪由于配置更改或容器镜像更新而导致的 Pod 模板的变化。Deployments 和 ReplicaSets 共同工作，但 Deployment 会创建自己的 ReplicaSet，因此我们通常只需要与 Deployments 交互。出于这个原因，我通常使用*Deployment*这个术语泛指 ReplicaSet 提供的功能，例如监控 Pod 并提供所请求的副本数量。

这是我们将用来创建 NGINX Deployment 的 YAML 文件：

*nginx-deploy.yaml*

```
---
 kind: Deployment
 apiVersion: apps/v1 
 metadata:
➊ name: nginx 
 spec:
   replicas: 3 
   selector: 
     matchLabels:
       app: nginx
   template:
     metadata:
    ➋ labels:
         app: nginx
  ➌ spec:   
       containers:
       - name: nginx
         image: nginx
      ➍ resources:
           requests:
             cpu: "100m"
```

Deployments 位于`apps` API 组中，因此我们为`apiVersion`指定`apps/v1`。像每个 Kubernetes 资源一样，我们需要提供一个唯一的名称 ➊，以便将这个 Deployment 与我们可能创建的其他 Deployment 区分开来。

Deployment 规格包含几个重要字段，我们来详细看看它们。`replicas`字段告诉 Kubernetes 我们想要多少个相同的 Pod 实例。Kubernetes 将努力保持这数量的 Pod 在运行。下一个字段，`selector`，用于使 Deployment 能够找到它的 Pod。`matchLabels`的内容必须与`template.metadata.labels`字段 ➋中的内容完全匹配，否则 Kubernetes 将拒绝该 Deployment。

最后，`template.spec` ➌的内容将作为此 Deployment 创建的任何 Pod 的`spec`。这里的字段可以包括我们为 Pod 提供的任何配置。此配置与我们之前查看的*nginx-pod.yaml*相匹配，不同之处在于我们添加了一个 CPU 资源请求 ➍，以便以后可以配置自动扩缩容。

让我们从这个 YAML 资源文件创建我们的 Deployment：

```
root@host01:~# kubectl apply -f /opt/nginx-deploy.yaml
deployment.apps/nginx created
```

我们可以使用`kubectl get`跟踪 Deployment 的状态：

```
root@host01:~# kubectl get deployment nginx
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   3/3     3            3           4s
```

当 Deployment 完全启动时，它将报告已准备好并可用的三个副本，这意味着我们现在有三个由这个 Deployment 管理的独立 NGINX Pod：

```
root@host01:~# kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6799fc88d8-6vn44   1/1     Running   0          18s
nginx-6799fc88d8-dcwx5   1/1     Running   0          18s
nginx-6799fc88d8-sh8qs   1/1     Running   0          18s
```

每个 Pod 的名称以 Deployment 的名称开头。Kubernetes 会添加一些随机字符来构建 ReplicaSet 的名称，然后再加上更多随机字符，以确保每个 Pod 都有唯一的名称。我们不需要直接创建或管理 ReplicaSet，但可以使用`kubectl get`来查看它：

```
root@host01:~# kubectl get replicasets
NAME               DESIRED   CURRENT   READY   AGE
nginx-6799fc88d8   3         3         3       30s
```

尽管我们通常只与 Deployments 交互，但了解 ReplicaSet 仍然很重要，因为在创建 Pod 时遇到的一些特定错误只会在 ReplicaSet 事件日志中报告。

`nginx` 前缀在 ReplicaSet 和 Pod 名称中纯粹是为了方便。Deployment 不使用名称来与 Pods 匹配。相反，它使用选择器来匹配 Pod 上的标签。如果我们在其中一个 Pod 上运行`kubectl describe`，就能看到这些标签：

```
root@host01:~# kubectl describe pod nginx-6799fc88d8-6vn44
Name:         nginx-6799fc88d8-6vn44
Namespace:    default
...
Labels:       app=nginx
...
```

这与 Deployment 的选择器匹配：

```
root@host01:~# kubectl describe deployment nginx
Name:                   nginx
Namespace:              default
...
Selector:               app=nginx
...
```

Deployment 查询 API 服务器以识别与其选择器匹配的 Pods。而 Deployment 使用程序化 API，下面的`kubectl get`命令生成了类似的 API 服务器查询，给我们一个了解其工作原理的机会：

```
root@host01:~# kubectl get all -l app=nginx
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6799fc88d8-6vn44   1/1     Running   0          69s
nginx-6799fc88d8-dcwx5   1/1     Running   0          69s
nginx-6799fc88d8-sh8qs   1/1     Running   0          69s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-6799fc88d8   3         3         3       69s
```

在这种情况下，使用`kubectl get all`可以列出多种不同类型的资源，只要它们与选择器匹配。因此，我们不仅能看到三个 Pods，还能看到 Deployment 为管理这些 Pods 而创建的 ReplicaSet。

看起来可能有些奇怪，Deployment 使用选择器而不是仅仅跟踪它创建的 Pods。然而，这种设计使得 Kubernetes 更容易自我修复。在任何时候，Kubernetes 节点可能会掉线，或者我们可能会遇到网络分割，期间某些控制节点与集群失去连接。如果一个节点重新上线，或者集群在网络分割后需要重新组合，Kubernetes 必须能够查看所有运行中的 Pods 的当前状态，并找出需要进行哪些更改以实现所需的状态。这可能意味着，当由于节点断开连接导致 Deployment 启动了一个额外的 Pod 时，在该节点重新连接时，Deployment 需要关闭一个 Pod，以便集群能够保持适当数量的副本。使用选择器避免了 Deployment 需要记住它曾创建过的所有 Pods，即使是那些在失败节点上的 Pods。

#### 监控与扩展

因为 Deployment 正在监视它的 Pods，以确保我们有正确数量的副本，所以我们可以删除一个 Pod，它会被自动重新创建：

```
root@host01:~# kubectl delete pod nginx-6799fc88d8-6vn44
pod "nginx-6799fc88d8-6vn44" deleted
root@host01:~# kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6799fc88d8-dcwx5   1/1     Running   0          3m52s
nginx-6799fc88d8-dtddk   1/1     Running   0        ➊ 14s
nginx-6799fc88d8-sh8qs   1/1     Running   0          3m52s
```

一旦旧的 Pod 被删除，Deployment 就会创建一个新的 Pod ➊。类似地，如果我们更改 Deployment 的副本数量，Pods 会自动更新。让我们再添加一个副本：

```
root@host01:~# kubectl scale --replicas=4 deployment nginx
deployment.apps/nginx scaled
root@host01:~# kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6799fc88d8-dcwx5   1/1     Running   0          8m22s
nginx-6799fc88d8-dtddk   1/1     Running   0          4m44s
nginx-6799fc88d8-kk7r6   1/1     Running   0        ➊ 5s 
nginx-6799fc88d8-sh8qs   1/1     Running   0          8m22s
```

第一个命令将副本数量设置为四个。因此，Kubernetes 需要启动一个新的相同 Pod 来满足我们请求的数量 ➊。我们可以通过更新 YAML 文件并重新运行`kubectl apply`来扩展 Deployment，或者我们可以使用`kubectl scale`命令直接编辑 Deployment。无论哪种方式，这都是一种声明式方法；我们在更新 Deployment 的资源声明；然后，Kubernetes 会更新集群的实际状态以使其匹配。

同样，缩小 Deployment 会导致 Pods 被自动删除：

```
root@host01:~# kubectl scale --replicas=2 deployment nginx
deployment.apps/nginx scaled
root@host01:~# kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6799fc88d8-dcwx5   1/1     Running   0          10m
nginx-6799fc88d8-sh8qs   1/1     Running   0          10m
```

当我们缩小时，Kubernetes 会选择两个 Pods 进行终止。这些 Pods 需要一些时间来完成关闭，届时我们只会有两个 NGINX Pods 在运行。

#### 自动扩展

对于正在接收用户真实请求的应用程序，我们会选择必要的副本数量以提供高质量的应用程序，同时在可能的情况下缩减副本数量，以减少应用程序使用的资源。当然，我们的应用程序负载是不断变化的，持续监控应用程序的每个组件并独立地进行缩放会很繁琐。相反，我们可以让集群为我们执行监控和缩放工作，使用 *HorizontalPodAutoscaler*。这里的 *horizontal* 术语仅指自动缩放器可以更新由控制器管理的同一 Pod 的副本数量。

要配置自动缩放，我们创建一个新的资源，引用我们的部署。然后，集群监控 Pod 使用的资源，并根据需要重新配置部署。我们可以使用 `kubectl autoscale` 命令将 HorizontalPodAutoscaler 添加到我们的部署中，但使用 YAML 资源文件可以将自动缩放配置保持在版本控制下，这样更好。以下是 YAML 文件：

*nginx-scaler.yaml*

```
   ---
➊ apiVersion: autoscaling/v2
   kind: HorizontalPodAutoscaler
   metadata:
     name: nginx
     labels:
       app: nginx
   spec:
  ➋ scaleTargetRef:
       apiVersion: apps/v1
       kind: Deployment
       name: nginx
  ➌ minReplicas: 1
     maxReplicas: 10
     metrics:
       - type: Resource
         resource:
           name: cpu
           target:
             type: Utilization
             averageUtilization: ➍ 50
```

在 `metadata` 字段中，我们添加了标签 `app: nginx`。这不会改变资源的行为；其唯一目的是确保如果我们在 `kubectl get` 命令中使用 `app=nginx` 标签选择器时，这个资源能够显示出来。通过一致的元数据标记应用程序组件的这种方式是一个好习惯，有助于他人理解哪些资源是相关的，并且使调试更容易。

这个 YAML 配置使用了版本 2 的自动缩放器配置 ➊。提供新的 API 资源组版本是 Kubernetes 在不失去任何向后兼容性的情况下支持未来功能的方式。通常，在最终配置发布之前，会发布 alpha 和 beta 版本的资源组，并且 beta 版本与最终版本之间至少有一个版本重叠，以支持无缝升级。

自动缩放器的版本 2 支持多个资源。每个资源用于计算对所需 Pod 数量的投票，最大数值将胜出。支持多个资源需要改变 YAML 布局，这是 Kubernetes 维护者创建新资源版本的常见原因。

我们使用 NGINX 部署 ➋ 的 API 资源组、类型和名称将其指定为自动缩放器的目标，这些足以唯一标识 Kubernetes 集群中的任何资源。然后，我们告诉自动缩放器监控属于该部署的 Pod 的 CPU 使用率 ➍。自动缩放器将努力保持 Pod 的平均 CPU 使用率接近 50%，并根据需要进行扩展或缩减。然而，副本数将永远不会超过我们指定的范围 ➌。

让我们使用此配置创建自动缩放器：

```
root@host01:~# kubectl apply -f /opt/nginx-scaler.yaml
horizontalpodautoscaler.autoscaling/nginx created
```

我们可以查询集群，查看它是否已被创建：

```
root@host01:~# kubectl get hpa
NAME    REFERENCE          TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
nginx   Deployment/nginx   0%/50%    1         10        3          96s
```

输出显示了自动伸缩器的目标引用、当前和期望的资源利用率，以及副本的最大值、最小值和当前值。

我们使用 `hpa` 作为 `horizontalpodautoscaler` 的缩写。Kubernetes 允许我们使用单数或复数名称，并为大多数资源提供缩写，以节省输入。例如，我们可以输入 `deploy` 来代替 `deployment`，甚至可以输入 `po` 来代替 `pods`。每一个额外的击键都很重要！

自动伸缩器使用 `kubelet` 已经从容器引擎收集的 CPU 利用率数据。这个数据由我们作为集群附加组件安装的指标服务器集中管理。如果没有这个集群附加组件，就不会有利用率数据，自动伸缩器也不会对部署进行任何更改。在这种情况下，因为我们实际上没有使用我们的 NGINX 服务器实例，它们没有消耗任何 CPU，部署被缩减到一个 Pod，即我们指定的最小值：

```
root@host01:~# kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6799fc88d8-dcwx5   1/1     Running   0          15m
```

自动伸缩器已计算出只需要一个 Pod，并已将部署调整到匹配的规模。然后，部署选择一个 Pod 终止，以达到所需的规模。

为了准确性，自动伸缩器不会使用最近才启动的 Pod 的 CPU 数据，并且它有逻辑来防止过于频繁地进行缩放，因此如果你快速完成了这些示例，你可能需要等待几分钟才能看到其缩放。

我们将在 第十四章 中更详细地探讨 Kubernetes 资源利用率指标。

### 其他控制器

部署是最通用和最常用的控制器，但 Kubernetes 还有其他一些有用的选项。在本节中，我们将探讨 *Job* 和 *CronJob*、*StatefulSets* 以及 *DaemonSets*。

#### 作业和定时作业

部署非常适合应用组件，因为我们通常希望一个或多个实例持续运行。然而，对于需要运行命令的情况，无论是一次性运行还是按计划运行，我们可以使用 Job。主要区别在于，部署确保任何停止运行的容器都会重启，而 Job 可以检查主进程的退出代码，仅当退出代码非零时才会重启，表示失败。

作业定义与部署非常相似：

*sleep-job.yaml*

```
---
apiVersion: batch/v1
kind: Job
metadata:
  name: sleep
spec:
  template:
    spec:
      containers:
      - name: sleep
        image: busybox
        command: 
          - "/bin/sleep"
          - "30"
      restartPolicy: OnFailure
```

`restartPolicy` 可以设置为 `OnFailure`，此时容器将在退出代码为非零时重启，或者设置为 `Never`，此时无论退出代码是什么，容器退出后作业将完成。

我们可以创建并查看作业及其创建的 Pod：

```
root@host01:~# kubectl apply -f /opt/sleep-job.yaml
job.batch/sleep created
root@host01:~# kubectl get job
NAME    COMPLETIONS   DURATION   AGE
sleep   0/1           3s         3s
root@host01:~# kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
...
sleep-fgcnz              1/1     Running   0          10s
```

作业根据 YAML 文件中提供的规范创建了一个 Pod。作业反映出 `0/1` 的完成状态，因为它正在等待其 Pod 成功退出。

当 Pod 运行了 30 秒后，它以零代码退出，表示成功，并且作业和 Pod 状态相应更新：

```
root@host01:~# kubectl get jobs
NAME    COMPLETIONS   DURATION   AGE
sleep   1/1           31s        40s
root@host01:~# kubectl get pods
NAME                     READY   STATUS      RESTARTS   AGE
nginx-65db7cf9c9-2wcng   1/1     Running     0          31m
sleep-fgcnz              0/1     Completed   0          43s
```

Pod 仍然可用，这意味着我们可以查看其日志（如果需要的话），但它显示为 `Completed` 状态，因此 Kubernetes 不会尝试重新启动已退出的容器。

CronJob 是一种按照计划创建 Jobs 的控制器。例如，我们可以设置我们的 sleep Job 每天运行一次：

*sleep-cronjob.yaml*

```
 ---
 apiVersion: batch/v1
 kind: CronJob
 metadata:
   name: sleep
 spec:
➊ schedule: "0 3 * * *"
➋ jobTemplate: 
   spec:
     template:
       spec:
         containers:
           - name: sleep
             image: busybox
             command: 
               - "/bin/sleep"
               - "30"
         restartPolicy: OnFailure
```

Job 规格的全部内容都嵌入在 `jobTemplate` 字段 ➋ 中。然后，我们添加一个遵循 Unix `cron` 命令标准格式的 `schedule` ➊。在这个例子中，`0 3 * * *` 表示 Job 应该在每天凌晨 3 点创建。

Kubernetes 的设计原则之一是任何东西都可能随时发生故障。对于 CronJob，如果集群在 Job 计划执行的时间遇到问题，Job 可能不会按计划执行，或者可能会被执行两次。这意味着你应该小心编写幂等的 Job，以便它们能够处理缺失或重复的调度。

如果我们创建这个 CronJob

```
root@host01:~# kubectl apply -f /opt/sleep-cronjob.yaml 
cronjob.batch/sleep created
```

它现在已存在于集群中，但不会立即创建 Job 或 Pod：

```
root@host01:~# kubectl get jobs
NAME    COMPLETIONS   DURATION   AGE
sleep   1/1           31s        2m32s
root@host01:~# kubectl get pods
NAME                     READY   STATUS      RESTARTS   AGE
nginx-65db7cf9c9-2wcng   1/1     Running     0          33m
sleep-fgcnz              0/1     Completed   0          2m23s
```

相反，每当 CronJob 的调度被触发时，它将创建一个新的 Job。

#### StatefulSets

到目前为止，我们已经看过一些创建相同 Pods 的控制器。对于 Deployments 和 Jobs，我们并不在乎哪个 Pod 是哪个，或者它部署在哪里，只要我们在正确的时间运行足够的实例。然而，这并不总是符合我们所需的行为。例如，尽管 Deployment 可以创建具有持久存储的 Pods，但存储必须是为每个新的 Pod 创建一个全新的存储，或者同一个存储必须在所有 Pods 之间共享。这与“主从”架构（例如数据库）并不完全匹配。对于这些情况，我们希望将特定的存储附加到特定的 Pods 上。

同时，由于 Pod 可能因硬件故障或升级而来去变化，我们需要一种方法来管理 Pod 的替换，以确保每个 Pod 都附加到正确的存储上。这就是 *StatefulSet* 的目的。StatefulSet 通过编号（从零开始）标识每个 Pod，并为每个 Pod 分配相应的持久存储。当 Pod 必须被替换时，新 Pod 会被分配相同的数字标识符，并附加到相同的存储上。Pods 可以查看它们的主机名来确定其标识符，因此 StatefulSet 对于需要固定主实例的情况以及动态选择主实例的情况都很有用。

在接下来的几章中，我们将深入探讨 Kubernetes StatefulSets 的更多细节，包括持久存储和服务。对于这一章，我们将查看 StatefulSet 的一个基本示例，然后在引入其他重要概念时进一步扩展。

对于这个简单的示例，让我们创建两个 Pods，并展示它们如何获得独特的存储，这些存储即使 Pod 被替换也会保持不变。我们将使用这个 YAML 资源：

*sleep-set.yaml*

```
 ---
 apiVersion: apps/v1
 kind: StatefulSet
 metadata:
    name: sleep
 spec:
➊ serviceName: sleep 
   replicas: 2
   selector:
     matchLabels:
       app: sleep
   template:
     metadata:
       labels:
         app: sleep
     spec:
       containers:
         - name: sleep
           image: busybox
           command: 
             - "/bin/sleep"
             - "3600"
        ➋ volumeMounts: 
             - name: sleep-volume
               mountPath: /storagedir
➌ volumeClaimTemplates: 
     - metadata:
         name: sleep-volume
       spec:
         storageClassName: longhorn
         accessModes:
           - ReadWriteOnce
         resources:
           requests:
             storage: 10Mi
```

与 Deployment 或 Job 相比，这里有一些重要的不同之处。首先，我们必须声明一个 `serviceName`，将这个 StatefulSet 绑定到 Kubernetes Service ➊。这个连接用于为每个 Pod 创建一个 DNS（域名服务）条目。我们还必须提供一个模板，供 StatefulSet 用来请求持久化存储 ➌，然后告诉 Kubernetes 在我们的容器中挂载该存储 ➋。

实际的 *sleep-set.yaml* 文件是自动化脚本安装的，其中包含 `sleep` 服务定义。我们将在第九章中详细讲解服务。

让我们创建 `sleep` StatefulSet：

```
root@host01:~# kubectl apply -f /opt/sleep-set.yaml
```

StatefulSet 创建了两个 Pods：

```
root@host01:~# kubectl get statefulsets
NAME    READY   AGE
sleep   2/2     1m14s
root@host01:~# kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
...
sleep-0   1/1     Running   0          57s
sleep-1   1/1     Running   0          32s
```

每个 Pod 的持久化存储是全新的，因此它开始时是空的。让我们创建一些内容。最简单的方法是通过容器内部，使用 `kubectl exec` 命令，它允许我们在容器内运行命令，类似于 `crictl`。`kubectl exec` 命令无论容器在哪个主机上运行都能工作，即使我们从集群外部连接到 Kubernetes API 服务器也是如此。

让我们将每个容器的主机名写入文件并打印出来，以便验证它是否成功：

```
root@host01:~# kubectl exec sleep-0 -- /bin/sh -c \
  'hostname > /storagedir/myhost'
root@host01:~# kubectl exec sleep-0 -- /bin/cat /storagedir/myhost
sleep-0
root@host01:~# kubectl exec sleep-1 -- /bin/sh -c \
  'hostname > /storagedir/myhost'
root@host01:~# kubectl exec sleep-1 -- /bin/cat /storagedir/myhost
sleep-1
```

现在我们的每个 Pod 在其持久化存储中都有独特的内容。让我们删除其中一个 Pod，并验证它的替代 Pod 是否继承了前一个 Pod 的存储：

```
root@host01:~# kubectl delete pod sleep-0
pod "sleep-0" deleted
root@host01:~# kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
...
sleep-0   1/1     Running   0          28s
sleep-1   1/1     Running   0          8m18s
root@host01:~# kubectl exec sleep-0 -- /bin/cat /storagedir/myhost
sleep-0
```

删除 `sleep-0` 后，我们看到一个新 Pod 被创建，且其名称与之前不同，这与 Deployment 不同，因为 Deployment 为每个新 Pod 生成一个随机名称。此外，对于这个新 Pod，我们之前创建的文件仍然存在，因为 StatefulSet 在删除旧 Pod 时，将相同的持久存储附加到了它创建的新 Pod 上。

#### Daemon Sets

*DaemonSet* 控制器类似于 StatefulSet，DaemonSet 也运行特定数量的 Pods，每个 Pod 都有独特的身份。然而，DaemonSet 每个节点只运行一个 Pod，这主要对集群的控制平面和附加组件（如网络或存储插件）非常有用。

我们的集群已经安装了多个 DaemonSets，因此让我们来看一下已经在运行的 `calico-node` DaemonSet，它在每个节点上运行，为该节点上的所有容器提供网络配置。

`calico-node` DaemonSet 位于 `calico-system` 命名空间，因此我们将指定该命名空间来请求有关 DaemonSet 的信息：

```
root@host01:~# kubectl -n calico-system get daemonsets
NAME          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   ...
calico-node   3         3         3       3            3           ...
```

我们的集群有三个节点，因此 `calico-node` DaemonSet 创建了三个实例。以下是该 DaemonSet 的 YAML 格式配置：

```
root@host01:~# kubectl -n calico-system get daemonset calico-node -o yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
...
  name: calico-node
  namespace: calico-system
...
spec:
...
  selector:
    matchLabels:
      k8s-app: calico-node
...
```

`-o yaml` 参数用于 `kubectl get` 命令，输出一个或多个资源的配置和状态，格式为 YAML，这样我们可以详细检查 Kubernetes 资源。

这个 DaemonSet 的选择器期望标签 `k8s-app` 设置为 `calico-node`。我们可以使用它来仅显示此 DaemonSet 创建的 Pods：

```
root@host01:~# kubectl -n calico-system get pods \
  -l k8s-app=calico-node -o wide
NAME                READY   STATUS   ... NODE   ...
calico-node-h9kjh   1/1     Running  ... host01 ...
calico-node-rcfk7   1/1     Running  ... host03 ...
calico-node-wj876   1/1     Running  ... host02 ...
```

DaemonSet 已创建了三个 Pods，每个 Pod 都被分配到了我们集群中的一个节点。如果我们向集群中添加更多节点，DaemonSet 也会在新节点上调度一个 Pod。

### 最后的思考

本章从普通集群用户的角度探讨了 Kubernetes，创建控制器进而创建带有容器的 Pods。掌握这些控制器资源类型的核心知识对于构建我们的应用程序至关重要。与此同时，重要的是要记住，Kubernetes 使用了我们在第一部分中探讨过的容器技术。

容器技术的一个关键方面是能够将容器隔离在不同的网络命名空间中。在 Kubernetes 集群中运行容器增加了网络方面的额外要求，因为我们现在需要连接运行在不同集群节点上的容器。在下一章中，我们将考虑多种方法来实现这一目标，并探讨覆盖网络。
