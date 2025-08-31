# 第十九章：调优服务质量

![image](img/common01.jpg)

理想情况下，我们的应用程序应使用最小或高度可预测的处理、内存、存储和网络资源。然而，在现实世界中，应用程序是“突发性的”，其负载变化由用户需求、大量数据或复杂处理驱动。在 Kubernetes 集群中，应用组件动态部署到集群中不同的节点上，如果负载在这些节点间分布不均，可能会造成性能瓶颈。

从应用架构的角度来看，越是将应用组件做得小巧且可扩展，我们就能越均匀地分配负载到集群中。不幸的是，性能问题并不总是能够通过水平扩展来解决。在本章中，我们将探讨如何使用资源规格来向集群提供有关如何调度我们的 Pod 的提示，目的是使应用性能更加可预测。

### 实现可预测性

在日常语言中，“实时”一词通常指某些迅速且持续发生的事情。但在计算机科学中，我们区分“实时”和“实时快速”，它们甚至被认为是对立的。这是因为可预测性的重要性。

实时处理指的是需要跟上现实世界某些活动的处理。它可以是任何需要跟上传感器数据输入并保持最新电子飞行显示的飞机驾驶舱软件，也可以是需要及时接收并解码每一帧视频以便显示的视频流应用程序。在实时系统中，至关重要的是我们能够保证处理“足够快”，以跟上现实世界的需求。

只要“足够快”就好。处理速度不需要快过现实世界，因为应用程序没有其他事情可做。但即便是一个处理速度慢于现实世界的时间间隔，也意味着我们落后于输入或输出，导致观影者的不满——甚至可能导致飞机坠毁。

因此，实时系统中的主要目标是可预测性。资源是根据系统可能遇到的最坏情况进行分配的，我们愿意提供比实际需要更多的处理能力，以确保在最坏情况下有足够的余地。实际上，要求这类系统在最大预期负载下，即使在可用处理和内存资源上，也要保持低于 50%的利用率是很常见的。

但尽管响应性始终很重要，大多数应用程序并不在实时环境中运行，而这种额外的资源余量是昂贵的。出于这个原因，大多数系统试图在可预测性和效率之间找到平衡，这意味着我们通常愿意容忍应用组件略微的性能下降，只要它是暂时的。

### 服务质量类别

为了帮助我们平衡集群中容器的可预测性和效率，Kubernetes 将 Pods 分配到三种不同的服务质量类别：`BestEffort`、`Burstable` 和 `Guaranteed`。从某种意义上讲，我们可以将这些类别看作是描述性的。`BestEffort` 用于我们没有提供任何资源要求时，它只能尽最大努力为 Pod 提供足够的资源。`Burstable` 用于 Pod 可能超过其资源请求的情况。`Guaranteed` 用于我们提供一致的资源要求，并且期望 Pod 始终保持在这些要求内。因为这些类别是描述性的，并且仅基于容器在 Pod 中指定的资源要求，因此没有办法手动指定 Pod 的 QoS。

QoS 类别有两种使用方式。首先，属于同一 QoS 类别的 Pods 会被分组，以便进行 Linux 控制组（cgroups）配置。正如我们在第三章中看到的，cgroups 用于控制一组进程的资源使用，特别是处理能力和内存，因此，Pod 的 cgroup 会影响其在系统负载较高时的处理时间优先级。其次，如果节点因内存资源不足需要开始逐出 Pods，QoS 类别会影响哪些 Pods 会首先被逐出。

#### BestEffort

最简单的情况是我们声明一个没有 `limits` 的 Pod。在这种情况下，Pod 被分配到 `BestEffort` 类别。让我们创建一个示例 Pod 来探索这意味着什么。

**注意**

*本书的示例代码库位于* [`github.com/book-of-kubernetes/examples`](https://github.com/book-of-kubernetes/examples)。*有关如何设置的详细信息，请参见第 xx 页中的“运行示例”。*

这是 Pod 的定义：

*best-effort.yaml*

```
---
apiVersion: v1
kind: Pod
metadata:
  name: best-effort
spec:
  containers:
  - name: best-effort
    image: busybox
    command: ["/bin/sleep", "infinity"]
  nodeName: host01
```

这个定义完全没有`resources`字段，但如果我们包含一个带有`requests`但没有`limits`的`resources`字段，QoS 类别会是一样的。

我们使用 `nodeName` 强制将该 Pod 部署到 `host01`，以便观察其资源使用配置。让我们将其应用到集群中：

```
root@host01:~# kubectl apply -f /opt/best-effort.yaml 
pod/best-effort created
```

在 Pod 启动后，我们可以查看它的详细信息，看到它已分配到 `BestEffort` QoS 类别：

```
root@host01:~# kubectl get po best-effort -o json | jq .status.qosClass
"BestEffort"
```

我们可以使用在第十四章中看到的 `cgroup-info` 脚本，查看 QoS 类别如何影响 Pod 中容器的 cgroup 配置：

```
root@host01:~# /opt/cgroup-info best-effort

Container Runtime
-----------------
Pod ID: 205...

Cgroup path: /kubepods.slice/kubepods-besteffort.slice/kubepods-...

CPU Settings
------------
CPU Shares: 2
CPU Quota (us): -1 per 100000

Memory Settings
---------------
Limit (bytes): 9223372036854771712
```

该 Pod 在 CPU 和内存使用上实际上没有限制。然而，Pod 的 cgroup 位于*kubepods-besteffort.slice*路径下，反映了它被分配到`BestEffort` QoS 类别中。这种分配直接影响了它的 CPU 优先级，正如我们在比较`BestEffort`类别和`Burstable`类别的`cpu.shares`时所看到的那样：

```
root@host01:~# cat /sys/fs/cgroup/cpu/kubepods.slice/kubepods-besteffort.slice/cpu.shares 
2
root@host01:~# cat /sys/fs/cgroup/cpu/kubepods.slice/kubepods-burstable.slice/cpu.shares 
1157
```

正如我们在第十四章中看到的，这些值是相对的，因此这一配置意味着，当系统的处理负载很高时，`Burstable` Pods 中的容器将被分配比`BestEffort` Pods 中容器超过 500 倍的处理器份额。这个值是基于已经在`BestEffort`和`Burstable` QoS 类别中的 Pod 数量，包括在*host01*上运行的各种集群基础设施组件，因此你可能会看到略有不同的值。

*kubepods.slice* cgroup 与用户和系统进程的 cgroup 处于同一级别，因此当系统负载较高时，它会获得与其他 cgroup 几乎相等的处理时间份额。基于在*kubepods.slice* cgroup 中识别到的*cpu.shares*，`BestEffort` Pods 相对于`Burstable` Pods，获得的处理器时间份额不到总份额的 1％，即使不考虑分配给`Guaranteed` Pods 的处理器时间。这意味着当系统负载高时，`BestEffort` Pods 几乎没有处理器时间，因此它们应该仅用于在集群空闲时运行的后台处理。此外，由于只有在未指定`limits`时才将 Pods 放置在`BestEffort`类别中，因此它们无法在具有限制配额的命名空间中创建。因此，我们的大多数应用程序 Pods 将位于其他两个 QoS 类别之一。

#### Burstable

如果 Pod 同时指定了`requests`和`limits`，并且这两个规格不同，则 Pod 会被放置在`Burstable`类别中。正如我们在第十四章中看到的，`requests`规格用于调度目的，而`limits`规格用于运行时强制执行。换句话说，这种情况下的 Pods 可以在其`requests`级别之上有“突发”的资源使用，但不能超过其`limits`。

让我们来看一个例子：

*burstable.yaml*

```
---
apiVersion: v1
kind: Pod
metadata:
  name: burstable
spec:
  containers:
  - name: burstable
    image: busybox
    command: ["/bin/sleep", "infinity"]
    resources:
      requests:
        memory: "64Mi"
        cpu: "50m"
      limits:
        memory: "128Mi"
        cpu: "100m"
  nodeName: host01
```

这个 Pod 定义提供了`requests`和`limits`资源要求，并且它们是不同的，因此我们可以预期这个 Pod 将被放置在`Burstable`类别中。

让我们将这个 Pod 应用到集群中：

```
root@host01:~# kubectl apply -f /opt/burstable.yaml 
pod/burstable created
```

接下来，让我们验证它是否已分配到`Burstable` QoS 类别：

```
root@host01:~# kubectl get po burstable -o json | jq .status.qosClass
"Burstable"
```

实际上，cgroup 配置遵循了我们指定的 QoS 类别和`limits`：

```
root@host01:~# /opt/cgroup-info burstable

Container Runtime
-----------------
Pod ID: 8d0...
Cgroup path: /kubepods.slice/kubepods-burstable.slice/kubepods-...

CPU Settings
------------
CPU Shares: 51
CPU Quota (us): 10000 per 100000

Memory Settings
---------------
Limit (bytes): 134217728
```

该`limits`为此 Pod 指定的值用于设置 CPU 限制和内存限制。此外，正如我们预期的，这个 Pod 的 cgroup 被放置在*kubepods-burstable.slice*中。

向`Burstable` QoS 类别添加另一个 Pod，导致 Kubernetes 重新平衡了处理器时间的分配：

```
root@host01:~# cat /sys/fs/cgroup/cpu/kubepods.slice/kubepods-besteffort.slice/cpu.shares 
2
root@host01:~# cat /sys/fs/cgroup/cpu/kubepods.slice/kubepods-burstable.slice/cpu.shares 
1413
```

结果是，`Burstable` QoS 类别下的 Pod 显示 *cpu.shares* 的值为 1413，而 `BestEffort` 类别下的 Pod 仍然显示 2。这意味着在负载下，`Burstable` 类别 Pod 的相对处理器份额是 700 比 1。再一次，你可能会看到略有不同的值，取决于 Kubernetes 为 `host01` 分配了多少基础设施 Pod。

因为 `Burstable` 类 Pod 是根据 `requests` 调度的，但 cgroup 运行时强制执行是基于 `limits` 的，所以节点的处理器和内存资源可能会超额分配。只要节点上的 Pod 彼此之间平衡，平均利用率与 `requests` 匹配，就没有问题。如果平均利用率超过了 `requests`，就会出现问题。在这种情况下，Pod 会看到其 CPU 被限速，如果内存变得紧张，可能会被驱逐，就像我们在第十章中看到的那样。

#### 保证类

如果我们希望提高 Pod 可用处理能力和内存的可预测性，可以通过设置相同的 `requests` 和 `limits` 来将 Pod 放入 `Guaranteed` QoS 类别。以下是一个示例：

*guaranteed.yaml*

```
---
apiVersion: v1
kind: Pod
metadata:
  name: guaranteed
spec:
  containers:
  - name: guaranteed
    image: busybox
    command: ["/bin/sleep", "infinity"]
    resources:
      limits:
        memory: "64Mi"
 cpu: "50m"
  nodeName: host01
```

在这个例子中，只有 `limits` 被指定，因为如果 `requests` 缺失，Kubernetes 会自动将 `requests` 设置为与 `limits` 匹配。

让我们将此应用于集群：

```
root@host01:~# kubectl apply -f /opt/guaranteed.yaml 
pod/guaranteed created
```

在 Pod 运行后，验证其 QoS 类别：

```
root@host01:~# kubectl get po guaranteed -o json | jq .status.qosClass
"Guaranteed"
```

cgroup 配置看起来有点不同：

```
root@host01:~# /opt/cgroup-info guaranteed

Container Runtime
-----------------
Pod ID: 146...
Cgroup path: /kubepods.slice/kubepods-...

CPU Settings
------------
CPU Shares: 51
CPU Quota (us): 5000 per 100000

Memory Settings
---------------
Limit (bytes): 67108864
```

与其将这些容器放入单独的目录中，`Guaranteed` QoS 类别下的容器直接放入 *kubepods.slice* 中。将它们放置在这个位置的效果是，当系统负载时，会优先考虑 `Guaranteed` 类 Pod 中的容器，因为这些容器按个别处理器份额接收 CPU 分配，而不是按类接收。

#### QoS 类别驱逐

`Guaranteed` QoS 类别 Pod 的优先处理也扩展到了 Pod 驱逐。如第三章中所述，cgroup 对内存限制的强制执行是由 OOM killer 处理的。当节点完全耗尽内存时，OOM killer 也会运行。为了帮助 OOM killer 选择要终止的容器，Kubernetes 会根据 Pod 的 QoS 类别设置 `oom_score_adj` 参数。此参数的值范围从 -1000 到 1000。数值越高，OOM killer 选择终止进程的可能性就越大。

`oom_score_adj` 值会为每个进程记录在 */proc* 中。自动化系统已添加一个名为 *oom-info* 的脚本，用于获取特定 Pod 的该值。让我们检查每个 QoS 类别下 Pod 的值：

```
root@host01:~# /opt/oom-info best-effort
OOM Score Adjustment: 1000
root@host01:~# /opt/oom-info burstable
OOM Score Adjustment: 968
root@host01:~# /opt/oom-info guaranteed
OOM Score Adjustment: -997
```

`BestEffort` QoS 类中的 Pods 具有最大调整值为 1000，因此它们会首先成为 OOM 杀手的目标。`Burstable` QoS 类中的 Pods 其得分是基于 `requests` 字段中指定的内存量计算的，作为节点总内存容量的百分比。因此，这个值对于每个 Pod 都会有所不同，但始终介于 2 和 999 之间。因此，`Burstable` QoS 类中的 Pods 在 OOM 杀手的优先级中始终排在第二位。与此同时，`Guaranteed` QoS 类中的 Pods 被设置为接近最小值，在本例中为 -997，因此它们会尽可能避免被 OOM 杀手终止。

当然，正如 第三章 中提到的，OOM 杀手会立即终止一个进程，因此它是一种极端的措施。当节点上的内存不足但尚未耗尽时，Kubernetes 会尝试驱逐 Pods 以回收内存。这个驱逐过程也根据 QoS 类进行优先级排序。`BestEffort` 类中的 Pods 和使用超过其 `requests` 值的 `Burstable` 类 Pods（高使用 `Burstable`）是最先被驱逐的，其次是使用低于其 `requests` 值的 `Burstable` 类 Pods（低使用 `Burstable`）和 `Guaranteed` 类中的 Pods。

在继续之前，让我们做一些清理：

```
root@host01:~# kubectl delete po/best-effort po/burstable po/guaranteed
pod "best-effort" deleted
pod "burstable" deleted
pod "guaranteed" deleted
```

现在我们可以在本章稍后再看一下 Pod 优先级时从头开始。

#### 选择 QoS 类

鉴于处理时间和驱逐优先级的这一优先顺序，可能会想将所有 Pods 都放在 `Guaranteed` QoS 类中。对于某些应用组件来说，这是一个可行的策略。如 第七章 所述，我们可以配置一个 HorizontalPodAutoscaler，当现有实例消耗了它们分配资源的显著比例时，自动创建新的 Pod 实例。这意味着我们可以为 Deployment 中的 Pods 请求一个合理的 `limits` 值，并允许集群在这些 Pods 接近限制时自动扩展 Deployment。如果集群运行在云环境中，我们甚至可以将自动扩展扩展到节点级别，在负载高时动态创建新的集群节点，在集群空闲时减少节点数量。

仅使用`Guaranteed` Pod 配合自动扩展听起来不错，但这假设我们的应用组件是容易扩展的。它也只有在我们的应用负载由许多小请求组成时才有效，这样负载增加主要意味着我们正在处理来自更多用户的类似大小的请求。如果我们的应用组件周期性地处理大或复杂的请求，我们必须为这些组件设置`limits`，以应对最坏情况。考虑到`Guaranteed` QoS 类中的 Pod 具有`requests`等于`limits`，我们的集群需要足够的资源来处理这个最坏情况，否则我们甚至无法调度我们的 Pod。这将导致集群在没有达到最大负载时大部分处于空闲状态。同样，如果我们有扩展性限制，如依赖于专业硬件，我们可能会对可以为某个组件创建的 Pod 数量有自然限制，从而迫使每个 Pod 拥有更多资源来处理其在整体负载中的份额。

因此，平衡使用`Guaranteed`和`Burstable` QoS 类对我们的 Pod 来说是有意义的。任何负载稳定，或者可以通过水平扩展来满足额外需求的 Pod，应该使用`Guaranteed`类。那些更难以扩展，或者需要处理大负载和小负载混合的 Pod，应该使用`Burstable`类。这些 Pod 应该根据其平均利用率来指定`requests`，并根据其最坏情况来指定`limits`。以这种方式指定资源需求，将确保集群的预期性能边际可以通过简单地将分配的资源与集群容量进行比较来进行监控。最后，如果一个大请求导致多个应用组件同时以最坏情况的利用率运行，那么可能值得进行性能测试，并探索反亲和性，如第十八章所述，以避免过载单个节点。

### Pod 优先级

除了使用提示帮助 Kubernetes 集群理解在系统高度负载时如何管理 Pods，还可以直接告诉集群为某些 Pods 分配比其他 Pods 更高的优先级。在 Pod 驱逐时，这种更高的优先级适用，因为 Pods 会根据其 QoS 类内的优先级顺序被驱逐。它在调度时也适用，因为 Kubernetes 调度器会在必要时驱逐 Pod，以便调度一个优先级更高的 Pod。

Pod 优先级是一个简单的数字字段；数字越大，优先级越高。大于十亿的数字保留给关键系统 Pod。为了为 Pod 分配优先级，我们必须首先创建一个*PriorityClass*资源。以下是一个示例：

*essential.yaml*

```
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: essential
value: 999999
```

让我们将其应用到集群中：

```
root@host01:~# kubectl apply -f /opt/essential.yaml 
priorityclass.scheduling.k8s.io/essential created
```

现在这个 PriorityClass 已经定义完毕，我们可以将其应用到 Pods。不过，首先让我们创建大量低优先级的 Pods，通过这些 Pods，我们可以看到 Pods 被抢占。我们将使用这个 Deployment：

```
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: lots
spec:
  replicas: 1000
  selector:
    matchLabels:
      app: lots
  template:
    metadata:
      labels:
        app: lots
    spec:
      containers:
      - name: sleep
        image: busybox
        command: ["/bin/sleep", "infinity"]
        resources:
          limits:
            memory: "64Mi"
            cpu: "250m"
```

这是一个基本的 Deployment，运行 `sleep`，并且没有请求太多内存或 CPU，但它将 `replicas` 设置为 `1000`，所以我们要求 Kubernetes 集群创建 1,000 个 Pods。示例集群的规模不足以部署 1,000 个 Pods，因为我们没有足够的资源来满足这个规格，而且每个节点默认最多只能调度 110 个 Pods。不过，还是让我们将它应用到集群中，如清单 19-1 所示，调度器会创建尽可能多的 Pods：

```
root@host01:~# kubectl apply -f /opt/lots.yaml 
deployment.apps/lots created
```

*清单 19-1：部署大量 Pods*

让我们描述一下 Deployment，看看情况如何：

```
root@host01:~# kubectl describe deploy lots
Name:                   lots
Namespace:              default
...
Replicas:               1000 desired ... | 7 available | 993 unavailable
...
```

由于集群基础设施组件已经运行了一些 Pods，我们的示例集群仅能容纳七个 Pods。不幸的是，这就是我们能得到的所有 Pods：

```
root@host01:~# kubectl describe node host01
Name:               host01
  (Total limits may be over 100 percent, i.e., overcommitted.)
Allocated resources:
...
  Resource           Requests     Limits
  --------           --------     ------
  cpu             ➊ 1898m (94%)  768m (38%)
  memory             292Mi (15%)  192Mi (10%)
  ephemeral-storage  0 (0%)       0 (0%)
  hugepages-2Mi      0 (0%)       0 (0%)
...
```

`host01` 的数据表明，我们已经分配了 94% 的可用 CPU ➊。但是我们的每个 Pod 请求 250 毫核心，所以没有足够的容量来调度另一个 Pod 到这个节点。其他两个节点也处于类似情况，没有足够的 CPU 容量来调度更多 Pods。不过，集群的运行状况非常良好。理论上，我们已经分配了所有的处理能力，但那些容器仅仅在运行 `sleep`，因此它们实际上并没有使用很多 CPU。

同时，重要的是要记住，`requests` 字段用于调度，因此尽管我们有一些基础设施 `BestEffort` Pods，它们指定了 `requests` 但没有 `limits`，而且我们这个节点上有足够的 `Limits` 容量，但我们依然没有空间调度新的 Pods。只有 `Limits` 可以超配，`Requests` 不能。

由于我们没有更多的 CPU 来分配给 Pods，Deployment 中剩余的 Pods 都卡在了 Pending 状态：

```
root@host01:~# kubectl get po | grep -c Pending
993
```

这 993 个 Pods 都有默认的 pod 优先级 0。因此，当我们使用 `essential` PriorityClass 创建一个新 Pod 时，它将排到调度队列的前面。不仅如此，集群还会根据需要驱逐 Pods，以便让它能够被调度。

这是 Pod 定义：

*needed.yaml*

```
---
apiVersion: v1
kind: Pod
metadata:
  name: needed
spec:
  containers:
  - name: needed
    image: busybox
    command: ["/bin/sleep", "infinity"]
    resources:
      limits:
        memory: "64Mi"
        cpu: "250m"
  priorityClassName: essential
```

这里的关键区别是 `priorityClassName` 的指定，它与我们创建的 PriorityClass 匹配。让我们将其应用到集群中：

```
root@host01:~# kubectl apply -f /opt/needed.yaml 
pod/needed created
```

集群需要一些时间来驱逐另一个 Pod，以便为这个 Pod 调度，但大约一分钟后它将开始运行：

```
root@host01:~# kubectl get po needed
NAME     READY   STATUS    RESTARTS   AGE
needed   1/1     Running   0          36s
```

为了让这一切发生，我们在清单 19-1 中创建的 `lots` Deployment 中的一个 Pod 必须被驱逐：

```
root@host01:~# kubectl describe deploy lots
Name:                   lots
Namespace:              default
CreationTimestamp:      Fri, 01 Apr 2022 19:20:52 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=lots
Replicas:               1000 desired ... | ➊ 6 available | 994 unavailable
```

现在在部署中只剩下六个 Pod ➊，因为有一个 Pod 被驱逐。值得注意的是，处于`Guaranteed` QoS 类别并没有防止该 Pod 被驱逐。`Guaranteed` QoS 类别在节点资源使用导致的驱逐中有优先权，但在调度器为更高优先级的 Pod 找到空间时，不能阻止驱逐。

当然，指定 Pod 的更高优先级，从而驱逐其他 Pod 的能力是非常强大的，应该谨慎使用。普通用户没有能力创建新的 PriorityClass，管理员可以为给定的命名空间应用配额，以限制 PriorityClass 的使用，实质上限制普通用户创建高优先级的 Pod。

### 最后的思考

将应用部署到 Kubernetes 上，使其既高效又可靠，需要理解应用架构以及每个组件的正常负载和最坏情况下的负载。Kubernetes QoS 类别允许我们塑造 Pod 部署到节点的方式，以在资源使用的可预测性和效率之间实现平衡。此外，QoS 类别和 Pod 优先级都可以为 Kubernetes 集群提供提示，以便在集群负载过高时，部署的应用能够优雅降级。

在下一章，我们将整合如何最好地利用 Kubernetes 集群的特性来部署高性能、具韧性的应用的想法。我们还将探讨如何监控这些应用，并自动响应行为变化。
