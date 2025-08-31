# 第十四章：限制与配额

![image](img/common01.jpg)

为了让我们的集群为应用程序提供一个可预测的环境，我们需要控制每个独立应用程序组件使用的资源。如果一个应用程序组件可以使用给定节点上所有的 CPU 或内存，Kubernetes 调度器将无法自信地将新 Pod 分配到节点，因为它无法知道每个节点的可用空间有多少。

在本章中，我们将探讨如何指定请求的资源和限制，确保容器获得所需的资源而不影响其他容器。我们将在运行时级别检查单个容器，以便我们可以看到 Kubernetes 如何配置我们在第一部分中看到的容器技术，足以满足容器的资源需求，同时避免容器超出其限制。

最后，我们将探讨如何使用基于角色的访问控制来管理配额，限制特定用户或应用程序可以请求的资源量，这将帮助我们了解如何以一种可靠支持多个独立应用程序或开发团队的方式管理集群。

### 请求与限制

Kubernetes 支持多种不同类型的资源，包括处理、内存、存储、网络带宽和特殊设备的使用，如图形处理单元（GPU）。我们将在本章后面讨论网络限制，但首先让我们从最常见的资源类型开始：处理和内存。

#### 处理和内存限制

处理和内存资源的规范有两个目的：调度和防止冲突。Kubernetes 为每个目的提供不同类型的资源规范。Pod 的容器在 Kubernetes 中消耗处理和内存资源，因此资源规范应用于这些地方。

在调度 Pods 时，Kubernetes 使用容器规范中的 `requests` 字段，将该字段的值在 Pod 中的所有容器中相加，并找到一个在处理和内存上都有足够余量的节点。通常，`requests` 字段设置为每个容器在 Pod 中的预期平均资源需求。

资源规范的第二个目的在于防止拒绝服务问题，其中一个容器占用了整个节点的资源，负面影响到其他容器。这要求在运行时执行容器资源的强制限制。Kubernetes 使用容器规范中的 `limits` 字段来实现这一目的，因此我们需要确保将 `limits` 字段设置得足够高，以便容器能够在不超出限制的情况下正确运行。

**性能调优**

请求应与预期的平均资源需求相匹配的想法，基于一个假设，即集群中各个容器的负载峰值是不可预测且不相关的，因此可以假设负载峰值会在不同时间发生。即便如此，仍然存在多个容器在同一节点上出现负载峰值时，导致该节点过载的风险。如果不同 Pod 之间的负载峰值是相关的，这种过载的风险就会增加。同时，如果我们为最坏情况配置`requests`，可能会导致集群过大，大部分时间都处于闲置状态。在第十九章中，我们探讨了 Kubernetes 为 Pod 提供的不同服务质量（QoS）类，并讨论了如何在性能保证和集群效率之间找到平衡。

清单 14-1 通过使用请求和限制的部署示例开始我们的检查。

*nginx-limit.yaml*

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
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
      nodeName: host01
```

*清单 14-1：带有限制的部署*

我们将使用这个部署来探索如何在容器运行时级别配置资源限制，因此我们使用`nodeName`字段确保容器最终运行在*host01*上。这会限制调度器放置 Pod 的位置，但调度器仍然会使用`requests`字段来确保有足够的资源。如果*host01*变得过于繁忙，调度器将拒绝调度该 Pod，这类似于我们在第十章中看到的情况。

`resources`字段是在单个容器级别定义的，允许我们为 Pod 中的每个容器指定单独的资源需求。对于这个容器，我们指定了`64Mi`的内存请求和`128Mi`的内存限制。后缀`Mi`表示我们使用的是 2 的幂次单位*兆二进制字节*（mebibytes），即 2 的 20 次方，而不是 10 的幂次单位*兆字节*（megabytes），后者的值略小，为 10 的 6 次方。

与此同时，使用`cpu`字段指定的处理请求和限制并不是基于任何绝对的处理单位，而是基于我们集群的合成*cpu 单位*。每个 cpu 单位大致对应一个虚拟 CPU 或核心。`m`后缀指定了*千分之一 cpu*，因此我们的`requests`值为`250m`，相当于四分之一核心，而`limit`为`500m`，相当于半个核心。

**注意**

*本书的示例代码库位于* [`github.com/book-of-kubernetes/examples`](https://github.com/book-of-kubernetes/examples)。*有关如何设置的详细信息，请参见第 xx 页的“运行示例”。*

让我们创建这个部署：

```
root@host01:~# kubectl apply -f /opt/nginx-limit.yaml 
deployment.apps/nginx created
```

Pod 将被分配到`host01`并启动：

```
root@host01:~# kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-56dbd744d9-vg5rj   1/1     Running   0          22m
```

然后`host01`将显示资源已分配给 Pod。

```
root@host01:~# kubectl describe node host01
Name:               host01
...
Non-terminated Pods:          (15 in total)
  Namespace Name                   CPU Requests CPU Limits Memory Requests Memory Limits Age
  --------- ----                   ------------ ---------- --------------- ------------- ---
...
  default   nginx-56dbd744d9-vg5rj 250m (12%)   500m (25%) 64M (3%)        128M (6%)     61s
...
```

即使我们的 NGINX web 服务器处于空闲状态，没有使用大量的处理或内存资源，这一点仍然成立：

```
root@host01:~# kubectl top pods
...
NAME                     CPU(cores)   MEMORY(bytes)   
nginx-56dbd744d9-vg5rj   0m           5Mi
```

类似于我们在第十二章中看到的，这个命令查询收集来自每个集群节点上运行的 `kubelet` 数据的度量插件。

#### Cgroup 强制执行

我们指定的处理和内存限制是通过使用 Linux 控制组（cgroup）功能来强制执行的，这在第三章中有描述。Kubernetes 在 */sys/fs/cgroup* 文件系统中的每个层级内管理自己的空间。例如，内存限制是在内存 cgroup 中配置的：

```
root@host01:~# ls -1F /sys/fs/cgroup/memory
...
kubepods.slice/
...
```

给定主机上的每个 Pod 在 *kubepods.slice* 树中都有一个目录。然而，找到特定 Pod 的目录需要一些工作，因为 Kubernetes 将 Pod 划分为不同的服务类别，并且 cgroup 目录的名称与 Pod 或其容器的 ID 不匹配。

为了避免我们在 */sys/fs/cgroup* 中四处查找，我们将使用本章自动化脚本安装的一个脚本：*/opt/cgroup-info*。这个脚本使用 `crictl` 查询容器运行时的 cgroup 路径，然后从该路径收集 CPU 和内存限制数据。脚本的最重要部分是这个收集路径的部分：

*cgroup-info*

```
#!/bin/bash
...
POD_ID=$(crictl pods --name ${POD} -q)
...
cgp_field='.info.config.linux.cgroup_parent'
CGP=$(crictl inspectp $POD_ID | jq -r "$cgp_field")

CPU=/sys/fs/cgroup/cpu/$CGP
MEM=/sys/fs/cgroup/memory/$CGP
...
```

`crictl pods` 命令收集 Pod 的 ID，然后与 `crictl inspectp` 和 `jq` 一起使用，以收集一个特定字段，称为 `cgroup_parent`。这个字段是为该 Pod 在每种资源类型中创建的 cgroup 子目录。

让我们使用我们的 NGINX Web 服务器运行这个脚本，看看 CPU 和内存限制是如何配置的：

```
root@host01:~# kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-56dbd744d9-vg5rj   1/1     Running   0          59m
root@host01:~# /opt/cgroup-info nginx-56dbd744d9-vg5rj

Container Runtime
-----------------
Pod ID: 54602befbd141a74316323b010fb38dae0c2b433cdbe12b5c4d626e6465c7315
Cgroup path: /kubepods.slice/...9f8f3dcf_6cca_49b8_a3df_d696ece01f59.slice

CPU Settings
------------
CPU Shares: 256
CPU Quota (us): 50000 per 100000

Memory Settings
---------------
Limit (bytes): 134217728
```

我们首先收集 Pod 的名称，然后用它来收集 cgroup 信息。请注意，这只有在 Pod 运行在 `host01` 上时才有效；该脚本适用于任何 Pod，但必须从该 Pod 运行所在的主机上执行。

对于 CPU 配置，有两个关键数据。配额是硬限制；它意味着在任何给定的 100,000 微秒期间，这个 Pod 只能使用 50,000 微秒的处理器时间。这个值对应于清单 14-1 中指定的 `500m` CPU 限制（回想一下，`500m` 限制相当于半个核心）。

除了这个硬限制之外，我们在清单 14-1 中指定的 CPU 请求字段已经用于配置 CPU 配额。正如我们在第三章中看到的，这个字段按相对方式配置 CPU 使用率。因为它是相对于相邻目录中的值的，所以没有单位，因此 Kubernetes 以每个核心等于 1,024 为基础计算 CPU 配额。我们指定了 `250m` 的 CPU 请求，因此这相当于 256。

CPU 配额并没有对 CPU 使用设定任何限制，因此如果系统空闲，Pod 可以使用其硬性限制范围内的所有处理能力。然而，随着系统变得繁忙，CPU 配额决定了每个 Pod 相对于同一服务类中的其他 Pod 分配的处理能力。这有助于确保如果系统超载，所有 Pod 将根据其 CPU 请求公平地降级。

最后，对于内存，只有一个相关的值。我们指定了 `128Mi` 的内存限制，相当于 128MiB。正如我们在第三章中看到的，如果我们的容器尝试超过此限制，它将被终止。因此，至关重要的是要么配置应用程序使其不会超过此值，要么了解应用程序在负载下的表现，以选择最佳限制。

一个进程实际使用的内存量最终取决于该进程本身，这意味着内存请求值除了在初始使用时确保有足够的内存来调度 Pod 外没有其他作用。因此，我们在 cgroup 配置中看不到 `64Mi` 的内存请求值被使用。

资源分配在 cgroup 中的反映方式让我们了解到关于集群性能的重要信息。因为 `requests` 用于调度，而 `limits` 用于运行时强制执行，所以一个节点可能会过度分配处理能力和内存。如果容器的 `limit` 大于 `requests`，并且容器始终在其 `requests` 之上运行，这可能会导致节点上的容器出现性能问题。我们将在第十九章中更详细地讨论这一点。

我们已经完成了 NGINX 部署，现在让我们将其删除：

```
root@host01:~# kubectl delete -f /opt/nginx-limit.yaml 
deployment.apps "nginx" deleted
```

到目前为止，容器运行时可以强制执行我们所看到的限制。然而，集群必须强制执行其他类型的限制，如网络。

#### 网络限制

理想情况下，我们的应用程序将设计为中等程度地需要用于互相通信的带宽，并且我们的集群将有足够的带宽来满足所有容器的需求。然而，如果确实有一个容器试图占用超过其份额的网络带宽，我们需要一种方法来限制它。

因为网络设备是通过插件配置的，我们需要一个插件来管理带宽。幸运的是，`bandwidth` 插件是与我们的 Kubernetes 集群一起安装的标准 CNI 插件的一部分。此外，正如我们在第八章中看到的，默认的 CNI 配置启用了 `bandwidth` 插件：

```
root@host01:~# cat /etc/cni/net.d/10-calico.conflist 
{
  "name": "k8s-pod-network",
  "cniVersion": "0.3.1",
  "plugins": [
...
    {
      "type": "bandwidth",
      "capabilities": {"bandwidth": true}
    },
...
  ]
```

结果是，`kubelet` 在每次创建新 Pod 时都会调用 `bandwidth` 插件。如果 Pod 配置了带宽限制，插件将利用我们在第三章中看到的 Linux 内核的流量控制功能，确保 Pod 的虚拟网络设备不会超过指定的限制。

让我们来看一个例子。首先，我们部署一个 `iperf3` 服务器来监听客户端连接：

*iperf-server.yaml*

```
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: iperf-server
spec:
  replicas: 1
  selector:
    matchLabels:
      app: iperf-server
  template:
    metadata:
      labels:
        app: iperf-server
    spec:
      containers:
      - name: iperf
        image: bookofkubernetes/iperf3:stable
        env:
        - name: IPERF_SERVER
          value: "1"
        resources: ...
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

除了 Deployment，我们还创建了一个 Service。这样，我们的 `iperf3` 客户端就可以通过其知名名称 `iperf-server` 找到服务器。我们指定了端口 5201，这是 `iperf3` 的默认端口。

让我们部署这个服务器：

```
root@host01:~# kubectl apply -f /opt/iperf-server.yaml 
deployment.apps/iperf-server created
service/iperf-server created
```

让我们运行一个不应用任何带宽限制的 `iperf3` 客户端。这将让我们了解在没有任何流量控制的情况下，集群网络的速度。以下是客户端定义：

*iperf.yaml*

```
---
kind: Pod
apiVersion: v1
metadata:
  name: iperf
spec:
  containers:
  - name: iperf
    image: bookofkubernetes/iperf3:stable
    resources: ...
```

通常，`iperf3` 客户端模式下会运行一次然后终止。这个镜像有一个脚本会重复运行 `iperf3`，每次运行之间休眠一分钟。让我们启动一个客户端 Pod：

```
root@host01:~# kubectl apply -f /opt/iperf.yaml 
pod/iperf created
```

Pod 启动需要几秒钟，之后初次运行将需要 10 秒钟。大约 30 秒后，Pod 日志将显示结果：

```
root@host01:~# kubectl logs iperf
Connecting to host iperf-server, port 5201
[  5] local 172.31.89.200 port 54346 connected to 10.96.0.192 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec   152 MBytes  1.28 Gbits/sec  225    281 KBytes       
[  5]   1.00-2.00   sec   154 MBytes  1.29 Gbits/sec  153    268 KBytes       
[  5]   2.00-3.00   sec   163 MBytes  1.37 Gbits/sec  230    325 KBytes       
[  5]   3.00-4.00   sec   171 MBytes  1.44 Gbits/sec  254    243 KBytes       
[  5]   4.00-5.00   sec   171 MBytes  1.44 Gbits/sec  191    319 KBytes       
[  5]   5.00-6.00   sec   174 MBytes  1.46 Gbits/sec  230    302 KBytes       
[  5]   6.00-7.00   sec   180 MBytes  1.51 Gbits/sec  199    221 KBytes       
[  5]   7.00-8.01   sec   151 MBytes  1.26 Gbits/sec  159    270 KBytes       
[  5]   8.01-9.00   sec   160 MBytes  1.36 Gbits/sec  145    298 KBytes       
[  5]   9.00-10.00  sec   147 MBytes  1.23 Gbits/sec  230    276 KBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec  1.59 GBytes  1.36 Gbits/sec  2016             sender
[  5]   0.00-10.00  sec  1.59 GBytes  1.36 Gbits/sec                  receiver

iperf Done.
```

在这种情况下，我们看到客户端和服务器之间的传输速率为 `1.36 GBits/sec`。根据您的集群部署情况以及客户端和服务器是否位于同一主机上，您的结果可能会有所不同。

在继续之前，我们将关闭现有的客户端，以防它干扰我们的下一个测试：

```
root@host01:~# kubectl delete pod iperf
pod "iperf" deleted
```

显然，在运行时，`iperf3` 尝试尽可能多地使用网络带宽。这对于测试应用程序来说没问题，但对于 Kubernetes 集群中的应用组件来说，这种行为并不太礼貌。为了限制其带宽，我们将在 Pod 定义中添加一个注解：

*iperf-limit.yaml*

```
 ---
 kind: Pod
 apiVersion: v1
 metadata:
   name: iperf-limit
➊ annotations:
     kubernetes.io/ingress-bandwidth: 1M
     kubernetes.io/egress-bandwidth: 1M
 spec:
   containers:
 - name: iperf
     image: bookofkubernetes/iperf3:stable
     resources: ...
   nodeName: host01
```

我们希望检查如何将限制应用到网络设备上，如果这个 Pod 最终在 `host01` 上，检查会更容易，所以我们相应地设置了 `nodeName`。否则，这个 Pod 定义中唯一的变化是 Pod 元数据中的 `annotations` 部分 ➊。我们为 ingress 和 egress 设置了 `1M` 的值，相当于对 Pod 设置了 1Mb 的带宽限制。当这个 Pod 被调度时，`kubelet` 会获取这些注解，并将指定的带宽限制发送给带宽插件，以便它可以相应地配置 Linux 流量整形。

让我们创建这个 Pod 并查看它的实际操作：

```
root@host01:~# kubectl apply -f /opt/iperf-limit.yaml 
pod/iperf-limit created
```

和之前一样，我们等待足够的时间让客户端完成一次与服务器的测试，然后打印日志：

```
root@host01:~# kubectl logs iperf-limit
Connecting to host iperf-server, port 5201
[  5] local 172.31.239.224 port 45680 connected to 10.96.0.192 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.01   sec  22.7 MBytes 190 Mbits/sec    0   1.37 KBytes       
[  5]   1.01-2.01   sec  0.00 Bytes  0.00 bits/sec    0    633 KBytes       
[  5]   2.01-3.00   sec  0.00 Bytes  0.00 bits/sec    0    639 KBytes       
[  5]   3.00-4.00   sec  0.00 Bytes  0.00 bits/sec    0    646 KBytes       
[  5]   4.00-5.00   sec  0.00 Bytes  0.00 bits/sec    0    653 KBytes       
[  5]   5.00-6.00   sec  1.25 MBytes 10.5 Mbits/sec   0    658 KBytes       
[  5]   6.00-7.00   sec  0.00 Bytes  0.00 bits/sec    0    658 KBytes       
[  5]   7.00-8.00   sec  0.00 Bytes  0.00 bits/sec    0    658 KBytes       
[  5]   8.00-9.00   sec  0.00 Bytes  0.00 bits/sec    0    658 KBytes       
[  5]   9.00-10.00  sec  0.00 Bytes  0.00 bits/sec    0    658 KBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec  24.0 MBytes  20.1 Mbits/sec    0             sender
[  5]   0.00-10.10  sec  20.7 MBytes  17.2 Mbits/sec                  receiver

iperf Done.
```

变化是显著的，因为该 Pod 的速度受限于我们在没有限制的客户端上看到的速度的一小部分。然而，由于流量整形基于令牌桶过滤器，短时间内流量控制并不精确，因此我们看到的比特率大约为 20Mb 而不是 1Mb。要了解原因，让我们看看实际的流量整形配置。

`bandwidth` 插件将令牌桶过滤器应用于为 Pod 创建的虚拟以太网（veth）对的主机端，因此我们可以通过显示主机接口的流量控制配置来查看它：

```
root@host01:~# tc qdisc show
...
qdisc tbf 1: dev calid43b03f2e06 ... rate 1Mbit burst 21474835b lat 4123.2s 
...
```

`rate`和`burst`的组合展示了为什么我们的 Pod 能够在 10 秒的测试运行中达到 20Mb。由于`burst`值，Pod 能够立即发送大量数据，但代价是花费了几秒钟的时间，无法发送或接收任何数据。在一个更长的时间间隔内，我们会看到平均为 1Mbps 的带宽，但我们仍然会看到这种爆发式的行为。

在继续之前，让我们清理客户端和服务器：

```
root@host01:~# kubectl delete -f /opt/iperf-server.yaml 
deployment.apps "iperf-server" deleted
service "iperf-server" deleted
root@host01:~# kubectl delete -f /opt/iperf-limit.yaml
pod "iperf-limit" deleted
```

管理 Pod 的带宽是有用的，但正如我们所见，带宽限制可能表现为 Pod 视角中的间歇性连接。因此，这种流量整形应该被视为无法配置自身带宽使用的容器的最后手段。

### 配额

限制（Limits）允许我们的 Kubernetes 集群确保每个节点拥有足够的资源来支持其分配的 Pod。然而，如果我们希望集群能够可靠地托管多个应用程序，我们需要一种方法来控制任何一个应用程序可以请求的资源数量。

为了实现这一点，我们将使用配额（quotas）。配额是基于命名空间（Namespaces）分配的，它们指定了在该命名空间内可以分配的最大资源量。这不仅包括 CPU 和内存等基本资源，还包括如 GPU 等专用集群资源。我们甚至可以使用配额来指定在给定命名空间内可以创建的特定对象类型的最大数量，比如部署（Deployment）、服务（Service）或定时任务（CronJob）。

由于配额是基于命名空间分配的，它们需要与我们在第十一章中描述的访问控制结合使用，以确保特定用户受我们创建的配额约束。这意味着创建命名空间和应用配额通常由集群管理员处理。

让我们为我们的部署创建一个示例命名空间：

```
root@host01:~# kubectl create namespace sample
namespace/sample created
```

现在，让我们创建一个*ResourceQuota*资源类型，以便为命名空间应用配额：

*quota.yaml*

```
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: sample-quota
  namespace: sample
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 256Mi
    limits.cpu: "2"
    limits.memory: 512Mi
```

这个资源定义了 CPU 和内存的配额，适用于请求（requests）和限制（limits）。单位与 Listing 14-1 中部署（Deployment）规范中的限制相同。

让我们将此配额应用到`sample`命名空间：

```
root@host01:~# kubectl apply -f /opt/quota.yaml
resourcequota/sample-quota created
```

我们可以看到这个配额已经成功应用：

```
root@host01:~# kubectl describe namespace sample
Name:         sample
Labels:       kubernetes.io/metadata.name=sample
Annotations:  <none>
Status:       Active

Resource Quotas
  Name:            sample-quota
  Resource         Used  Hard
  --------         ---   ---
  limits.cpu       0     2
  limits.memory    0     512Mi
  requests.cpu     0     1
  requests.memory  0     256Mi
...
```

即使这个配额会应用于所有尝试在命名空间中创建 Pod 的用户，包括集群管理员，考虑到管理员总是可以创建新的命名空间来绕过配额，使用普通用户更为现实。因此，我们还将创建一个用户：

```
root@host01:~# kubeadm kubeconfig user --client-name=me \
  --config /etc/kubernetes/kubeadm-init.yaml > kubeconfig
```

如同我们在第十一章中所做的那样，我们将把`edit`角色绑定到该用户，以提供在`sample`命名空间中创建和编辑资源的权限。我们将使用在 Listing 11-1 中看到的相同 RoleBinding：

```
root@host01:~# kubectl apply -f /opt/edit-bind.yaml
rolebinding.rbac.authorization.k8s.io/editor created
```

现在我们的用户已设置完成，让我们设置`KUBECONFIG`环境变量，以便未来的`kubectl`命令将以我们的正常用户身份执行：

```
root@host01:~# export KUBECONFIG=kubeconfig
```

首先，我们可以验证普通用户所拥有的 `edit` 角色并不允许对命名空间中的配额进行更改，这很合理——配额是管理员职能：

```
root@host01:~# kubectl delete -n sample resourcequota sample-quota
Error from server (Forbidden): resourcequotas "sample-quota" is forbidden: 
User "me" cannot delete resource "resourcequotas" in API group "" in the 
namespace "sample"
```

现在我们可以在 `sample` 命名空间中创建一些 Pods 来测试配额。首先，让我们尝试创建一个没有限制的 Pod：

```
root@host01:~# kubectl run -n sample nginx --image=nginx
Error from server (Forbidden): pods "nginx" is forbidden: failed quota: 
sample-quota: must specify limits.cpu,limits.memory...
```

因为我们的命名空间有配额，我们不再允许创建没有指定限制的 Pods。

在清单 14-2 中，我们再次尝试，这次使用了一个指定资源限制的部署，该部署为它创建的 Pods 设置了资源限制。

*sleep.yaml*

```
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: sleep
  namespace: sample
spec:
  replicas: 1
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
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "512m"
```

*清单 14-2：带有限制的部署*

现在我们可以将其应用到集群中：

```
root@host01:~# kubectl apply -n sample -f /opt/sleep.yaml 
deployment.apps/sleep created
```

这是成功的，因为我们指定了必要的请求和限制字段，并且没有超过配额。此外，Pod 以我们指定的限制启动：

```
root@host01:~# kubectl get -n sample pods
NAME                     READY   STATUS    RESTARTS   AGE
sleep-688dc46d95-wtppg   1/1     Running   0          72s
```

然而，我们可以看到，我们现在正在使用配额中的资源：

```
root@host01:~# kubectl describe namespace sample
Name:         sample
Labels:       kubernetes.io/metadata.name=sample
Annotations:  <none>
Status:       Active

Resource Quotas
  Name:            sample-quota
  Resource         Used   Hard
  --------         ---    ---
  limits.cpu       512m   2
  limits.memory    128Mi  512Mi
  requests.cpu     250m   1
  requests.memory  64Mi   256Mi
...
```

这将限制我们扩展该部署的能力。让我们来说明一下：

```
root@host01:~# kubectl scale -n sample deployment sleep --replicas=12
deployment.apps/sleep scaled
root@host01:~# kubectl get -n sample pods
NAME                     READY   STATUS    RESTARTS   AGE
sleep-688dc46d95-trnbl   1/1     Running   0          6s
sleep-688dc46d95-vzfsx   1/1     Running   0          6s
sleep-688dc46d95-wtppg   1/1     Running   0          3m13s
```

我们请求了 12 个副本，但我们只看到有三个在运行。如果我们描述这个部署，就会看到一个问题：

```
root@host01:~# kubectl describe -n sample deployment sleep
Name:      sleep
Namespace: sample
...
Replicas:   12 desired | 3 updated | 3 total | 3 available | 9 unavailable
...
Conditions:
  Type             Status  Reason
  ----             ------  ------
  Progressing      True    NewReplicaSetAvailable
  Available        False   MinimumReplicasUnavailable
  ReplicaFailure   True    FailedCreate
OldReplicaSets:    <none>
NewReplicaSet:     sleep-688dc46d95 (3/12 replicas created)
...
```

现在命名空间报告说，我们已经消耗了足够的配额，无法为另一个 Pod 分配所需的资源：

```
root@host01:~# kubectl describe namespace sample
Name:         sample
...
Resource Quotas
  Name:            sample-quota
  Resource         Used   Hard
  --------         ---    ---
  limits.cpu       1536m  2
  limits.memory    384Mi  512Mi
  requests.cpu     750m   1
  requests.memory  192Mi  256Mi
...
```

我们的 Pods 正在运行 `sleep`，因此我们知道它们几乎不使用任何 CPU 或内存。然而，Kubernetes 是基于我们指定的配额来计算配额利用率，而不是 Pod 实际使用的资源。这一点至关重要，因为进程在变得繁忙时可能会使用更多的 CPU 或分配更多的内存，而 Kubernetes 需要确保为集群的其他部分留出足够的资源，以保证其正常运行。

### 最终思考

为了让我们的容器化应用程序更可靠，我们需要确保一个应用组件不会占用过多资源，从而有效地使集群中其他容器“饿死”。Kubernetes 能够利用底层容器运行时和 Linux 内核的资源限制功能，将每个容器限制在其已分配的资源范围内。这一做法确保了容器在集群节点上的调度更加可靠，并确保即使集群负载较重，集群资源的分配也能公平共享。

在本章中，我们已经了解了如何为我们的部署指定资源需求，以及如何为命名空间应用配额，从而有效地将集群中的所有节点视为一个大型可用资源池。在下一章，我们将探讨这一原理如何扩展到存储方面，看看如何动态地为 Pods 分配存储，无论它们被调度到哪里。
