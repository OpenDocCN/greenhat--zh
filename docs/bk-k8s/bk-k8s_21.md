# 第十八章：亲和性和设备

![image](img/common01.jpg)

理想化的应用程序展示了完全的简单性。它的设计简单，开发简单，部署简单。它的各个组件都是无状态的，因此很容易扩展以服务尽可能多的用户。每个服务端点都充当纯粹的函数，其输出仅由输入决定。应用程序处理的数据量合理，CPU 和内存需求适中，请求和响应容易适配到一个最多只有几千字节的 JSON 结构中。

当然，除了教程之外，理想化的应用程序是不存在的。现实世界中的应用程序会存储状态，包括长期的持久存储和可以快速访问的缓存。现实世界的应用程序有数据安全和授权方面的考虑，因此它们需要进行用户身份验证，记住用户是谁，并相应地限制访问权限。许多现实世界的应用程序还需要访问专用硬件，而不仅仅是使用理想化的 CPU、内存、存储和网络资源。

我们希望在 Kubernetes 集群上部署现实世界中的应用程序，而不仅仅是理想化的应用程序。这意味着我们需要做出明智的决策，关于如何部署那些让我们远离理想化世界的应用程序组件——在那个世界中，集群决定运行多少个容器实例以及如何调度它们。然而，我们不想创建一个过于僵化的应用架构，以至于失去集群的可扩展性和弹性。相反，我们希望在集群内工作，给集群一些提示，指导如何部署我们的应用组件，同时尽可能保持灵活性。在本章中，我们将探讨我们的应用组件如何在不失去 Kubernetes 优势的情况下，强制与其他组件或专用硬件之间形成一定的耦合。

### 亲和性与反亲和性

我们将首先看一下管理 Pods 调度的情况，这样我们可以优先或避免将多个容器部署在同一个节点上。例如，如果我们有两个消耗大量网络带宽并相互通信的容器，我们可能希望这两个容器一起运行在一个节点上，以减少延迟并避免拖慢集群中的其他部分。或者，如果我们希望确保一个高可用组件能够在集群中的一个节点丢失时依然存活，我们可能希望将 Pod 实例拆分，使它们尽可能在不同的集群节点上运行。

合并多个独立的容器到一个 Pod 规范中，是共置容器的一种方法。这对于两个进程完全相互依赖的情况是一个很好的解决方案。然而，这也失去了单独扩展实例的能力。例如，在一个由分布式存储支持的 Web 应用中，我们可能需要比存储进程更多的 Web 服务器进程实例。我们需要将这些应用组件放置在不同的 Pod 中，以便能够单独扩展它们。

在 第八章中，当我们想确保一个 Pod 在指定的节点上运行时，我们在 Pod 规范中添加了 `nodeName` 字段以覆盖调度器。这个方法对于示例来说是可以的，但对于实际应用，它会消除性能和可靠性所必需的扩展和故障转移功能。相反，我们将使用 Kubernetes 的 *亲和性* 概念，为调度器提供关于如何分配 Pod 的提示，而不强制任何 Pod 必须在特定节点上运行。

亲和性允许我们根据其他 Pods 的存在来限制 Pod 应该调度到哪里。让我们来看一个使用 `iperf3` 网络测试应用的例子。

**集群区域**

Pod 亲和性对于跨多个网络的大型集群最为有用。例如，我们可能会将 Kubernetes 集群部署到多个不同的数据中心，以消除单点故障。在这些情况下，我们会根据一个包含多个节点的区域来配置亲和性。在这里，我们只有一个小型示例集群，所以我们将把集群中的每个节点视为一个独立的区域。

#### 反亲和性

让我们从亲和性的反面开始：*反亲和性*。反亲和性会导致 Kubernetes 调度器避免将 Pods 共置在一起。在这种情况下，我们将创建一个有三个独立 `iperf3` 服务器 Pod 的 Deployment，但我们将使用反亲和性规则将这三个 Pod 分布到不同的节点上，使每个节点都有一个 Pod。

**注意**

*本书的示例代码库位于* [`github.com/book-of-kubernetes/examples`](https://github.com/book-of-kubernetes/examples)。 *有关如何设置的详细信息，请参见 第 xx 页中的“运行示例”。*

这是我们需要的 YAML 定义：

*ipf-server.yaml*

```
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: iperf-server
spec:
  replicas: 3
  selector:
    matchLabels:
      app: iperf-server
  template:
    metadata:
      labels:
        app: iperf-server
    spec:
   ➊ affinity:
        podAntiAffinity:
       ➋ requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - iperf-server
         ➌ topologyKey: "kubernetes.io/hostname"
 containers:
      - name: iperf
        image: bookofkubernetes/iperf3:stable
        env:
        - name: IPERF_SERVER
          value: "1"
```

这个 Deployment 资源是典型的，除了新的 `affinity` 部分 ➊。我们指定了一个基于 Deployment 用来管理其 Pods 的相同标签的反亲和性规则。通过这个规则，我们指定不希望将 Pod 调度到已经有 `app=iperf-server` 标签的区域。

`topologyKey` ➌ 指定了区域的大小。在这种情况下，集群中的每个节点都有不同的 `hostname` 标签，因此每个节点都被视为一个不同的区域。因此，反亲和性规则会阻止 `kube-scheduler` 在第一个 Pod 已经调度到某个节点后，再将第二个 Pod 调度到该节点。

最后，因为我们使用 `requiredDuringScheduling` ➋ 指定了规则，所以这是一个 *硬* 反亲和性规则，这意味着调度器不会调度 Pod，除非它能满足这个规则。如果规则不能满足，也可以使用 `preferredDuringScheduling` 并分配一个权重，给调度器提供提示，但不会阻止 Pod 调度。

**注意**

*topologyKey 可以基于应用于节点的任何标签。基于云的 Kubernetes 分发通常会根据节点的可用区自动为每个节点应用标签，这使得使用反亲和性在可用区之间分布 Pods 以实现冗余变得容易。*

让我们应用这个 Deployment 并查看结果：

```
root@host01:~# kubectl apply -f /opt/ipf-server.yaml 
deployment.apps/iperf-server created
```

一旦我们的 Pod 启动运行，我们会看到每个节点都被分配了一个 Pod：

```
root@host01:~# kubectl get po -o wide
NAME                            READY   STATUS    ... NODE     ...
iperf-server-7666fb76d8-7rz8j   1/1     Running   ... host01   ...
iperf-server-7666fb76d8-cljkh   1/1     Running   ... host02   ...
iperf-server-7666fb76d8-ktk92   1/1     Running   ... host03   ...
```

因为我们有三个节点和三个实例，这与使用 DaemonSet 本质上是相同的，但这种方法更加灵活，因为它不需要每个节点上都有实例。在大型集群中，我们可能只需要少量的 Pod 实例来满足服务需求。使用基于主机名的反亲和性与区域相结合，可以让我们在仍然将每个 Pod 分配到不同节点以提高可用性的同时，指定部署的正确规模。而且反亲和性也可以用于将 Pods 分布到其他类型的区域。

在继续之前，让我们创建一个 Service，供我们的 `iperf3` 客户端找到一个服务器实例。以下是 YAML 文件：

*ipf-svc.yaml*

```
---
kind: Service
apiVersion: v1
metadata:
  name: iperf-server
spec:
  selector:
    app: iperf-server
  ports:
  - protocol: TCP
    port: 5201
    targetPort: 5201
```

让我们将此应用于集群：

```
root@host01:~# kubectl apply -f /opt/ipf-svc.yaml 
service/iperf-server created
```

服务会启动所有三个 Pod：

```
root@host01:~# kubectl get ep iperf-server
NAME           ENDPOINTS                                                 ...
iperf-server   172.31.239.207:5201,172.31.25.214:5201,172.31.89.206:5201 ...
```

`ep` 是 `endpoints` 的缩写。每个 Service 都有一个相关联的 Endpoint 对象，用来记录当前接收流量的 Pods。

#### 亲和性

我们现在准备将 `iperf3` 客户端部署到这些服务器实例上。我们希望以相同的方式将客户端分配到每个节点，但我们需要确保每个客户端都部署到一个有服务器实例的节点上。为此，我们将使用亲和性和反亲和性规则：

*ipf-client.yaml*

```
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: iperf
spec:
  replicas: 3
  selector:
    matchLabels:
      app: iperf
  template:
    metadata:
      labels:
 app: iperf
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - iperf
            topologyKey: "kubernetes.io/hostname"
        ➊ podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - iperf-server
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: iperf
        image: bookofkubernetes/iperf3:stable
```

额外的 `podAffinity` 规则 ➊ 确保每个客户端实例只有在服务器实例已经存在的情况下才会部署到节点。亲和性规则中的字段与反亲和性规则相同。

让我们部署客户端实例：

```
root@host01:~# kubectl apply -f /opt/ipf-client.yaml 
deployment.apps/iperf created
```

在这些 Pods 运行后，我们可以看到它们已经分布到集群中的所有三个节点：

```
root@host01:~# kubectl get po -o wide
NAME                            READY   STATUS    ... NODE     ... 
iperf-c8d4566f-btppf            1/1     Running   ... host02   ... 
iperf-c8d4566f-s6rpn            1/1     Running   ... host03   ... 
iperf-c8d4566f-v9v8m            1/1     Running   ... host01   ... 
...
```

看起来我们已将`iperf3`客户端和服务器部署得能够使每个客户端连接到其本地的服务器实例，从而最大化客户端和服务器之间的带宽。然而，实际上并非如此。因为`iperf-server`服务配置了所有三个 Pods，每个客户端 Pod 都连接到一个随机的服务器。因此，我们的客户端可能无法正常工作。你可能会看到日志显示某个客户端能够连接到服务器，但也可能会看到客户端 Pods 处于`Error`或`CrashLoopBackOff`状态，并且有类似如下的日志输出：

```
root@host01:~# kubectl logs iperf-c8d4566f-v9v8m
iperf3: error - the server is busy running a test. try again later
iperf3 error - exiting
```

这表示某个客户端正在连接到已经有客户端连接的服务器，这意味着至少有两个客户端在使用同一个服务器。

### 服务流量路由

我们希望配置我们的客户端 Pods，使其能够访问我们部署的本地服务器 Pod，而不是不同节点上的服务器 Pod。让我们首先确认流量是否在所有三个服务器 Pods 之间随机路由。我们可以查看`kube-proxy`为该服务创建的`iptables`规则：

```
root@host01:~# iptables-save | grep iperf-server
...
-A KUBE-SVC-KN2SIRYEH2IFQNHK -m comment --comment "default/iperf-server" 
  -m statistic --mode random --probability 0.33333333349 -j KUBE-SEP-IGBNNG5F5VCPRRWI
-A KUBE-SVC-KN2SIRYEH2IFQNHK -m comment --comment "default/iperf-server" 
  -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-FDPADR4LUNHDJSPL
-A KUBE-SVC-KN2SIRYEH2IFQNHK -m comment --comment "default/iperf-server" 
  -j KUBE-SEP-TZDPKVKUEZYBFM3V
```

我们在*host01*上运行这个命令，看到有三条独立的`iptables`规则，并且目标是随机选择的。这意味着，*host01*上的`iperf3`客户端可能会被路由到任何一个服务器 Pod。

为了解决这个问题，我们需要更改我们服务的内部流量策略配置。默认情况下，策略是`Cluster`，表示集群中的所有 Pods 都是有效的目标。我们可以将策略更改为`Local`，这样就会限制服务仅路由到同一节点上的 Pods。

让我们修补服务来更改这个策略：

```
root@host01:~# kubectl patch svc iperf-server -p '{"spec":{"internalTrafficPolicy":"Local"}}'
service/iperf-server patched
```

更改立即生效，我们可以通过再次查看`iptables`规则来验证：

```
root@host01:~# iptables-save | grep iperf-server
...
-A KUBE-SVC-KN2SIRYEH2IFQNHK -m comment --comment "default/iperf-server" \
  -j KUBE-SEP-IGBNNG5F5VCPRRWI
```

这一次，只有一个可能的目标被配置在*host01*上，因为该服务只有一个本地 Pod 实例。

几分钟后，`iperf3`客户端现在显示出我们预期看到的输出：

```
root@host01:~# kubectl logs iperf-c8d4566f-btppf
Connecting to host iperf-server, port 5201
...
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec  8.67 GBytes  7.45 Gbits/sec  1250             sender
[  5]   0.00-10.00  sec  8.67 GBytes  7.45 Gbits/sec                  receiver
...
```

不仅所有客户端都能够连接到独特的服务器，而且由于网络连接是本地到每个节点的，性能始终很高。

在继续之前，让我们清理这些资源：

```
root@host01:~# kubectl delete svc/iperf-server deploy/iperf deploy/iperf-server
service "iperf-server" deleted
deployment.apps "iperf" deleted
deployment.apps "iperf-server" deleted
```

虽然`Local`内部流量策略有助于最大化客户端和服务器之间的带宽，但它也有一个主要的限制。如果某个节点没有健康的 Pod 实例，那么该节点上的客户端将根本无法访问服务，即使其他节点上有健康的实例。在使用这种设计模式时，至关重要的是还要配置一个就绪探针，如第十三章中所述，它不仅检查 Pod 本身，还检查其服务依赖性。这样，如果某个节点上的服务无法访问，该节点上的客户端也会报告自己为不健康，从而不会有流量路由到它。

我们所看到的亲和性和反亲和性功能使我们能够在不牺牲应用组件的可扩展性和弹性的前提下，向调度器提供提示。然而，尽管在应用架构中有紧密连接的组件时，使用这些功能可能很有诱惑力，但最好是让调度器无阻碍地工作，仅在实际的性能测试表明它能够带来显著差异时，才添加亲和性。

为了提高性能，服务路由是 Kubernetes 中的一个活跃开发领域。对于跨多个区域运行的集群，一种名为拓扑感知提示（Topology Aware Hints）的新功能，可以使 Kubernetes 将连接路由到离服务实例最近的地方，从而提高网络性能，同时在必要时允许跨区域流量。

### 硬件资源

亲和性和反亲和性允许我们控制 Pods 的调度位置，但应该仅在必要时使用。那么，对于某些 Pod 需要访问仅在某些节点上可用的专用硬件的情况该怎么办呢？例如，我们可能有需要图形处理单元（GPU）加速的处理任务，但为了降低成本，我们可能会限制集群中的 GPU 节点数量。在这种情况下，确保 Pod 被调度到正确的地方是绝对必要的。

和之前一样，我们可以通过 `nodeName` 将 Pod 直接绑定到某个节点。但集群中可能有多个节点具备所需的硬件，因此我们真正需要的是能够向 Kubernetes 说明需求，然后让调度器决定如何满足这个需求。

Kubernetes 提供了两种相关的方法来解决这一需求：设备插件和扩展资源。设备插件提供了最完整的功能，但插件本身必须存在于硬件设备上。同时，扩展资源可以用于任何硬件设备，但 Kubernetes 集群只会跟踪该资源的分配，而不实际管理其在容器中的可用性。

实现设备插件需要与 `kubelet` 紧密协作。类似于我们在第十五章中看到的存储插件架构，设备插件会向运行在节点上的 `kubelet` 实例注册自己，标识它管理的任何设备。Pod 标识它们所需的设备，设备管理器告诉 `kubelet` 如何在容器内使设备可用（通常是通过将设备从主机挂载到容器的文件系统中）。

由于我们是在一个虚拟化的示例集群中操作，因此没有专用硬件来演示设备插件，但扩展资源从分配的角度来看是相同的，因此我们仍然可以对整体方法有所了解。

首先，通过更新集群，指示某个节点具有示例扩展资源。我们通过修补节点的 `status` 来实现这一点。理想情况下，我们可以使用 `kubectl patch` 来执行此操作，但不幸的是，无法通过该命令更新资源的 `status`，因此我们只能使用 `curl` 直接调用 Kubernetes API。 */opt* 目录下有一个脚本可以简化此过程。清单 18-1 展示了相关部分。

*add-hw.sh*

```
#!/bin/bash
...
patch='
[
  {
    "op": "add", 
    "path": "/status/capacity/bookofkubernetes.com~1special-hw", 
    "value": "3"
  }
]
'
curl --cacert $ca --cert $cert --key $key \
  -H "Content-Type: application/json-patch+json" \
  -X PATCH -d "$patch" \
  https://192.168.61.10:6443/api/v1/nodes/host02/status
...
```

*清单 18-1：特殊硬件脚本*

该 `curl` 命令发送一个 JSON 补丁对象来更新节点的 `status` 字段，在 `capacity` 下添加一个名为 `bookofkubernetes.com/special-hw` 的条目。`~1` 起到斜杠字符的作用。

运行脚本以更新节点：

```
root@host01:~# /opt/add-hw.sh 
...
```

从 API 服务器返回的响应包括整个节点的资源。让我们再次确认我们关心的字段，以确保它已经应用：

```
root@host01:~# kubectl get node host02 -o json | jq .status.capacity
{
  "bookofkubernetes.com/special-hw": "3",
  "cpu": "2",
  "ephemeral-storage": "40593612Ki",
  "hugepages-2Mi": "0",
  "memory": "2035228Ki",
  "pods": "110"
}
```

扩展资源与节点的标准资源一起显示。现在，我们可以像请求标准资源一样请求该资源，正如我们在第十四章中看到的那样。

这是一个请求特殊硬件的 Pod：

*hw.yaml*

```
---
apiVersion: v1
kind: Pod
metadata:
  name: sleep
spec:
  containers:
  - name: sleep
    image: busybox
    command: ["/bin/sleep", "infinity"]
    resources:
      limits:
        bookofkubernetes.com/special-hw: 1
```

我们使用 `resources` 字段来指定对特殊硬件的需求。资源要么被分配，要么不分配；因此，`requests` 和 `limits` 之间没有区别，所以 Kubernetes 希望我们使用 `limits` 来指定。当我们将此应用到集群时，Kubernetes 调度器会确保该 Pod 运行在能够满足此要求的节点上：

```
root@host01:~# kubectl apply -f /opt/hw.yaml 
pod/sleep created
```

因此，Pod 最终被调度到 `host02`：

```
root@host01:~# kubectl get po -o wide
NAME    READY   STATUS    ... NODE     ...
sleep   1/1     Running   ... host02   ...
```

此外，节点状态现在反映了该扩展资源的分配：

```
root@host01:~# kubectl describe node host02
Name:               host02
...
Allocated resources:
...
  Resource                         Requests     Limits
  --------                         --------     ------
...
  bookofkubernetes.com/special-hw  1            1
...
```

当我们在清单 18-1 中添加扩展资源时，所指定的三台 `special-hw` 的可用数量，以及该资源分配给 Pod 的方式，都是任意的。扩展资源就像一个信号量，防止过多的用户同时使用同一资源，但如果我们真的有三个单独的特殊硬件设备在同一节点上运行，我们需要增加额外的处理来避免多个用户冲突。

如果我们根据指定的可用资源尝试过度分配，Pod 将无法调度。如果我们尝试添加另一个需要所有三个特殊硬件设备的 Pod，我们可以确认这一点：

*hw3.yaml*

```
---
apiVersion: v1
kind: Pod
metadata:
  name: sleep3
spec:
  containers:
  - name: sleep
    image: busybox
    command: ["/bin/sleep", "infinity"]
    resources:
 limits:
        bookofkubernetes.com/special-hw: 3
```

让我们尝试将这个 Pod 添加到集群中：

```
root@host01:~# kubectl apply -f /opt/hw3.yaml 
pod/sleep created
```

由于没有足够的特殊硬件设备可用，因此这个 Pod 保持在 Pending 状态：

```
root@host01:~# kubectl get po -o wide
NAME    READY   STATUS    ... NODE     ...
sleep   1/1     Running   ... host02   ...
sleep3  0/1     Pending   ... <none>   ...
```

Pod 将等待硬件可用。让我们删除原始的 Pod 以释放空间：

```
root@host01:~# kubectl delete pod sleep 
pod/sleep deleted
```

我们的新 Pod 现在将开始运行：

```
root@host01:~# kubectl get po -o wide
NAME    READY   STATUS    ... NODE     ...
sleep3  1/1     Running   ... host02   ...
```

和之前一样，Pod 被调度到 `host02`，这是由于特殊硬件的需求。

设备驱动程序从资源分配的角度来看是相同的。在这两种情况下，我们都使用`limits`字段来确定硬件要求。唯一的不同之处在于，我们不需要手动修补节点来记录资源，因为当设备驱动程序注册时，`kubelet`会自动更新节点的状态。此外，当容器创建时，`kubelet`会调用设备驱动程序来执行任何必要的硬件分配和配置。

### 最终思考

与理想应用程序不同，在现实世界中，我们通常需要处理紧密耦合的应用组件和对专用硬件的需求。至关重要的是，我们必须在不失去从将应用程序部署到 Kubernetes 集群中获得的灵活性和弹性的前提下，考虑这些应用程序的需求。在本章中，我们看到亲和性和设备驱动程序如何使我们能够向调度程序提供提示和资源要求，同时仍然允许它具有动态管理应用程序规模的灵活性。

调度并不是我们在考虑如何从现实世界应用程序中获得所需行为和性能时唯一需要关注的问题。在下一章中，我们将看到如何通过使用服务质量类来塑造我们 Pod 的处理和内存分配。
