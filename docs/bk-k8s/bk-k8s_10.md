# 第八章：覆盖网络

![image](img/common01.jpg)

当所有容器都在单个主机上时，容器网络已经足够复杂，正如我们在第四章中看到的那样。当我们扩展到一个包含多个节点的集群时，所有节点都运行容器时，复杂性会大幅增加。我们不仅需要为每个容器提供自己的虚拟网络设备，并管理 IP 地址，动态创建新的网络命名空间和设备，还需要确保一个节点上的容器能够与所有其他节点上的容器进行通信。

在本章中，我们将描述如何使用 *覆盖网络* 来提供跨 Kubernetes 集群所有节点的单一容器网络的表象。我们将考虑两种不同的方法来路由容器流量穿越主机网络，检查每种方法的网络配置和流量流向。最后，我们将探讨 Kubernetes 如何使用容器网络接口（CNI）标准将网络配置作为一个独立的插件，使其能够轻松切换到新的技术，并在需要时允许自定义解决方案。

### 集群网络

Kubernetes 集群的基本目标是将一组主机（物理机或虚拟机）视为一个单一的计算资源，可以根据需要分配以运行容器。从网络的角度来看，这意味着 Kubernetes 应该能够将 Pod 调度到任何节点，而不必担心与其他节点上的 Pods 的连接问题。这也意味着 Kubernetes 应该有一种方式，能够动态地为 Pods 分配 IP 地址，以支持集群范围的网络连接性。

正如我们将在本章中看到的，Kubernetes 使用插件设计来允许任何兼容的网络软件分配 IP 地址并提供跨节点的网络连接性。所有插件必须遵循几个重要的规则。首先，Pod 的 IP 地址应该来自一个单一的 IP 地址池，尽管这个池可以按节点细分。这意味着我们可以将所有 Pods 视为一个单一的平面网络，无论 Pods 运行在哪里。其次，流量应该是可路由的，以便所有 Pods 都能看到所有其他 Pods 和控制平面。

#### CNI 插件

插件通过 CNI 标准与 Kubernetes 集群进行通信，特别是与 `kubelet` 通信。CNI 规范了 `kubelet` 如何查找和调用 CNI 插件。当创建一个新的 Pod 时，`kubelet` 首先分配网络命名空间。然后它调用 CNI 插件，并为其提供网络命名空间的引用。CNI 插件向命名空间添加网络设备，分配 IP 地址，并将该 IP 地址返回给 `kubelet`。

让我们看看这个过程是如何工作的。为了做到这一点，本章的示例包括两种不同的环境和两种不同的 CNI 插件：Calico 和 WeaveNet。这两个插件都为 Pods 提供网络连接，但在跨节点网络方面有所不同。我们将从 Calico 环境开始。

**注意**

*本书的示例仓库位于* [`github.com/book-of-kubernetes/examples`](https://github.com/book-of-kubernetes/examples)。 *有关设置的详细信息，请参阅第 xx 页中的“运行示例”。*

默认情况下，CNI 插件信息保存在 */etc/cni/net.d* 目录中。我们可以在该目录中查看 Calico 配置：

```
root@host01:~# ls /etc/cni/net.d
10-calico.conflist  calico-kubeconfig
```

文件*10-calico.conflist*包含实际的 Calico 配置。文件*calico-kubeconfig*由 Calico 组件用于与控制平面进行身份验证；它是基于在 Calico 安装过程中创建的服务账户生成的。配置文件名前缀为*10-*，因为`kubelet`会对它找到的任何配置文件进行排序，并使用第一个文件。

清单 8-1 显示了配置文件，该文件是 JSON 格式，指定了要使用的网络插件。

```
root@host01:~# cat /etc/cni/net.d/10-calico.conflist 
{
  "name": "k8s-pod-network",
  "cniVersion": "0.3.1",
  "plugins": [
    {
      "type": "calico",
...
    },
    {
      "type": "bandwidth",
      "capabilities": {"bandwidth": true}
    },
    {"type": "portmap", "snat": true, "capabilities": {"portMappings": true}}
  ]
}
```

*清单 8-1：Calico 配置*

最重要的字段是`type`；它指定了要运行的插件。在本例中，我们运行了三个插件：`calico`，用于处理 Pod 网络；`bandwidth`，可以用来配置网络限制；以及`portmap`，用于将容器端口暴露到主机网络。这两个插件通过`capabilities`字段告知`kubelet`它们的用途；因此，当`kubelet`调用它们时，它会传递相关的带宽和端口映射配置，以便插件可以进行必要的网络配置更改。

为了运行这些插件，`kubelet`需要知道它们的位置。实际插件可执行文件的默认位置是 */opt/cni/bin*，插件名称与`type`字段相匹配。

```
root@host01:~# ls /opt/cni/bin
bandwidth  calico-ipam  flannel      install   macvlan  sbr     vlan
bridge     dhcp         host-device  ipvlan    portmap  static
calico     firewall     host-local   loopback  ptp      tuning
```

在这里，我们看到一组常见的网络插件，它们是由`kubeadm`与我们的 Kubernetes 集群一起安装的。我们还看到了`calico`，它是由我们在集群初始化后安装的 Calico DaemonSet 添加到该目录中的。

#### Pod 网络

让我们查看一个示例 Pod，以便了解 CNI 插件如何配置 Pod 的网络命名空间。这个行为与我们在第四章中做的非常相似，通过将虚拟网络设备添加到网络命名空间中，来启用容器之间以及与主机网络之间的通信。

让我们创建一个基本的 Pod：

*pod.yaml*

```
---
apiVersion: v1
kind: Pod
metadata:
  name: pod
spec:
  containers:
  - name: pod
    image: busybox
    command: 
      - "sleep"
      - "infinity"
  nodeName: host01
```

我们添加了额外的字段`nodeName`，强制此 Pod 在`host01`上运行，这样更容易找到并检查其网络配置。

我们通过常规命令启动 Pod：

```
root@host01:~# kubectl apply -f /opt/pod.yaml
pod/pod created
```

接下来，检查它是否正在运行：

```
root@host01:~# kubectl get pods
NAME   READY   STATUS    RESTARTS   AGE
pod    1/1     Running   0          2m32s
```

它运行后，我们可以使用`crictl`捕获它的唯一 ID：

```
root@host01:~# POD_ID=$(crictl pods --name pod -q)
root@host01:~# echo $POD_ID
b7d2391320e07f97add7ccad2ad1a664393348f1dcb6f803f701318999ed0295
```

此时，使用 Pod ID，我们可以找到其网络命名空间。在清单 8-2 中，我们使用`jq`来提取我们想要的数据，就像在第四章中做的那样。然后我们将其赋值给一个变量。

```
root@host01:~# NETNS_PATH=$(crictl inspectp $POD_ID |
  jq -r '.info.runtimeSpec.linux.namespaces[]|select(.type=="network").path')
root@host01:~# echo $NETNS_PATH
/var/run/netns/cni-7cffed61-fb56-9be1-0548-4813d4a8f996
root@host01:~# NETNS=$(basename $NETNS_PATH)
root@host01:~# echo $NETNS
cni-7cffed61-fb56-9be1-0548-4813d4a8f996
```

*清单 8-2：网络命名空间*

现在，我们可以探索网络命名空间，查看 Calico 是如何为这个 Pod 设置 IP 地址和网络路由的。首先，正如预期的那样，这个网络命名空间是为我们的 Pod 使用的：

```
root@host01:~# ps $(ip netns pids $NETNS)
    PID TTY      STAT   TIME COMMAND
  35574 ?        Ss     0:00 /pause
  35638 ?        Ss     0:00 sleep infinity
```

我们可以看到预期中的两个进程。第一个是一个暂停容器，每当我们创建 Pod 时，它总是会被创建。这是一个永久容器，用于保持网络命名空间。第二个是我们运行`sleep`的 BusyBox 容器，正如我们在 Pod 的 YAML 文件中配置的那样。

现在，让我们看看配置好的网络接口：

```
root@host03:~# ip netns exec $NETNS ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN ...
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
3: ➊ eth0@if16: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 ... state UP ...
    link/ether 7a:9e:6c:e2:30:47 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet ➋ 172.31.239.205/32 brd 172.31.25.202 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::789e:6cff:fee2:3047/64 scope link 
       valid_lft forever preferred_lft forever
```

Calico 在网络命名空间➊中创建了网络设备`eth0@if16`，并为其分配了 IP 地址`172.31.239.205`➋。请注意，该 IP 地址的网络掩码是`/32`，这表示所有流量必须通过配置好的路由器。这与第四章中桥接容器网络的工作方式不同。这样配置是必要的，以便 Calico 通过网络策略提供防火墙功能。

该 Pod 所选的 IP 地址最终是由 Calico 决定的。Calico 的 IP 地址空间配置为`172.31.0.0/16`，用于 Pod 的 IP 地址分配。Calico 决定如何在节点之间划分该地址空间，并从分配给节点的范围内为每个 Pod 分配 IP 地址。然后，Calico 将此 IP 地址返回给`kubelet`，以便更新 Pod 的状态：

```
root@host01:~# kubectl get pods -o wide
NAME   READY   STATUS    RESTARTS   AGE   IP                NODE    ...
pod    1/1     Running   0          16m   172.31.239.205   host01   ...
```

当 Calico 在 Pod 中创建网络接口时，它是作为虚拟以太网（veth）对的一部分来创建的。veth 对充当一个虚拟网络线缆，创建一个到根命名空间中网络接口的连接，从而允许 Pod 外部的连接。清单 8-3 让我们看看 veth 对的两个部分。

```
root@host01:~# ip netns exec $NETNS ip link
...
3: eth0@if13: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue ... 
    link/ether 6e:4c:3a:41:d0:54 brd ff:ff:ff:ff:ff:ff link-netnsid 0
root@host01:~# ip link | grep -B 1 $NETNS
13: cali9381c30abed@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 ... 
    link/ether ee:ee:ee:ee:ee:ee ... link-netns cni-7cffed61-fb56-9be1-0548-4813d4a8f996
```

*清单 8-3：Calico veth 对*

第一个命令打印命名空间内的网络接口，而第二个命令打印主机上的接口。每个命令都包含字段`link-netns`，指向另一个接口的相应网络命名空间，显示这两个接口创建了 Pod 命名空间与根命名空间之间的链接。

### 跨节点网络

到目前为止，容器中虚拟网络设备的配置与第四章中的容器网络非常相似，当时并未安装 Kubernetes 集群。区别在于，网络插件配置不仅仅是为了连接单节点上的容器，而是为了连接在集群中任何地方运行的容器。

**为什么不使用 NAT？**

常规的容器网络确实提供与主机网络的连接。然而，正如我们所讨论的，它是通过网络地址转换（NAT）来实现的。这对于运行单个客户端应用程序的容器来说是可以的，因为连接跟踪使得 Linux 能够将服务器响应路由到原始容器中。但这对于需要充当服务器的容器就不适用了，而这正是 Kubernetes 集群的一个关键使用场景。

对于大多数使用 NAT 连接到更广泛网络的私有网络，端口转发用于从私有网络内部暴露特定服务。对于每个 Pod 中的每个容器来说，这并不是一个好的解决方案，因为我们很快就会用尽可分配的端口。网络插件最终确实使用 NAT，但仅仅是为了将作为客户端的容器连接到集群外部的网络。此外，我们将在第九章中看到端口转发的行为，它将是暴露服务到集群外部的可能方法之一。

跨节点网络的挑战在于，Pod 网络的 IP 地址范围与主机网络不同，因此主机网络不知道如何路由这些流量。网络插件有几种不同的方法来解决这个问题。我们将继续使用运行 Calico 的集群开始，然后展示使用 WeaveNet 的不同跨节点网络技术。

#### Calico 网络

Calico 使用第 3 层路由进行跨节点网络连接。这意味着它基于 IP 地址进行路由，在每个主机和 Pod 中配置 IP 路由表，以确保流量发送到正确的主机，然后到达正确的 Pod。因此，在主机级别，我们看到 Pod 的 IP 地址作为源地址和目标地址。由于 Calico 依赖于 Linux 的内建路由功能，我们不需要配置主机网络交换机来路由流量，但我们确实需要配置主机网络交换机上的任何安全控制，以允许 Pod 的 IP 地址跨网络传输。

为了探索 Calico 跨节点网络连接，最好有两个 Pods：一个在 `host01` 上，另一个在 `host02` 上。我们将使用这个资源文件：

*two-pods.yaml*

```
---
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
  - name: pod1
    image: busybox
    command: 
      - "sleep"
      - "infinity"
  nodeName: host01
---
apiVersion: v1
kind: Pod
metadata:
  name: pod2
spec:
  containers:
  - name: pod2
    image: busybox
    command: 
      - "sleep"
      - "infinity"
  nodeName: host02
```

和往常一样，这些文件已经通过自动化脚本加载到本章节的*/opt*目录中。

`---` 分隔符允许我们将两个不同的 Kubernetes 资源放在同一个文件中，以便我们可以一起管理它们。这两个 Pod 的唯一配置差异是它们各自有一个 `nodeName` 字段，以确保它们被分配到正确的节点。

让我们删除现有的 Pod，并用我们需要的两个 Pod 替换它：

```
root@host01:~# kubectl delete -f /opt/pod.yaml
pod "pod" deleted
root@host01:~# kubectl apply -f /opt/two-pods.yaml 
pod/pod1 created
pod/pod2 created
```

在这些 Pods 启动后，我们需要收集它们的 IP 地址：

```
root@host01:~# IP1=$(kubectl get po pod1 -o json | jq -r '.status.podIP')
root@host01:~# IP2=$(kubectl get po pod2 -o json | jq -r '.status.podIP')
root@host01:~# echo $IP1
172.31.239.216
root@host01:~# echo $IP2
172.31.89.197
```

我们能够使用简单的 `jq` 过滤器提取 Pod IP，因为我们的 `kubectl get` 命令保证只返回一个项目。如果我们没有过滤器地运行 `kubectl get`，或者使用可能匹配多个 Pods 的过滤器，JSON 输出将是一个列表，我们需要相应地修改 `jq` 过滤器。

让我们快速验证这两个 Pods 之间的连接性：

```
root@host01:~# kubectl exec -ti pod1 -- ping -c 3 $IP2
PING 172.31.89.197 (172.31.89.197): 56 data bytes
64 bytes from 172.31.89.197: seq=0 ttl=62 time=2.867 ms
64 bytes from 172.31.89.197: seq=1 ttl=62 time=0.916 ms
64 bytes from 172.31.89.197: seq=2 ttl=62 time=1.463 ms

--- 172.31.89.197 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.916/1.748/2.867 ms
```

`ping` 命令显示所有三个数据包成功到达，因此我们知道 Pods 可以跨节点通信。

如我们之前的示例所示，每个 Pod 都有一个网络接口，网络长度为`/32`，意味着所有流量必须经过路由器。例如，以下是`pod1`的 IP 配置和路由表：

```
root@host01:~# kubectl exec -ti pod1 -- ip addr
...
3: eth0@if17: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1450 qdisc noqueue 
    link/ether f2:ed:e8:04:00:cc brd ff:ff:ff:ff:ff:ff
    inet 172.31.239.216/32 brd 172.31.239.216 scope global eth0
...
root@host01:~# kubectl exec -ti pod1 -- ip route
default via 169.254.1.1 dev eth0 
169.254.1.1 dev eth0 scope link
```

根据此配置，当我们运行`ping`命令时，网络栈会识别目标 IP 不属于任何接口的本地网络。因此，它会在其地址解析协议（ARP）表中查找`169.254.1.1`以确定“下一跳”应该发送到哪里。如果我们尝试在容器或主机上找到一个具有`169.254.1.1`地址的接口，我们是无法成功的。Calico 并不会实际将该地址分配给某个接口，而是配置了“代理 ARP”，使得数据包通过 veth 对的`eth0`端发送。因此，容器内的 ARP 表中会有`169.254.1.1`的条目：

```
root@host01:~# kubectl exec -ti pod1 -- arp -n
? (169.254.1.1) at ee:ee:ee:ee:ee:ee [ether]  on eth0
...
```

如清单 8-3 所示，硬件地址`ee:ee:ee:ee:ee:ee`属于 veth 对的主机端，因此这足以将数据包从容器中取出并进入根网络命名空间。从那里，IP 路由接管。

Calico 已经配置了路由表，根据节点的目标 IP 地址范围将数据包发送到其他集群节点，并根据每个容器的 IP 地址将数据包发送到本地容器。我们可以在主机上的 IP 路由表中看到这个结果：

```
root@host01:~# ip route
...
172.31.25.192/26 via 192.168.61.13 dev enp0s8 proto 80 onlink 
172.31.89.192/26 via 192.168.61.12 dev enp0s8 proto 80 onlink 
172.31.239.216 dev calice0906292e2 scope link 
...
```

由于 ping 的目标地址位于`172.31.89.192/26`网络中，数据包现在被路由到`192.168.61.12`，即`host02`。

让我们查看`host02`上的路由表，以便跟随接下来的步骤：

```
root@host02:~# ip route
...
172.31.239.192/26 via 192.168.61.11 dev enp0s8 proto 80 onlink 
172.31.25.192/26 via 192.168.61.13 dev enp0s8 proto 80 onlink 
172.31.89.197 dev calibd2348b4f67 scope link 
...
```

如果你想自己运行这个命令，确保从`host02`运行。当我们的数据包到达`host02`时，它已经有了一个特定目标 IP 地址的路由，这个路由将数据包发送到附加在`pod2`网络命名空间的 veth 对中。

现在，ping 数据包已经到达，`pod2`内的网络栈会发送回一个回复。这个回复会通过相同的过程，到达`host02`的根网络命名空间。根据`host02`的路由表，它会被发送到`host01`，并使用`172.31.239.216`的路由表条目将数据包发送到适当的容器。

由于 Calico 使用的是第 3 层路由，主机网络可以看到实际的容器 IP 地址。我们可以使用`tcpdump`来确认这一点。为此，我们将切换回`host01`。

首先，让我们在后台启动`tcpdump`：

```
root@host01:~# tcpdump -n -w pings.pcap -i any icmp &
[1] 70949
tcpdump: listening on any ...
```

`-n`标志告诉`tcpdump`避免查找任何 IP 地址的主机名，这样可以节省时间。`-w pings.pcap`标志告诉`tcpdump`将数据写入文件*pings.pcap*；`-i any`标志告诉它监听所有网络接口；`icmp`过滤器告诉它仅监听 ICMP 流量；最后，`&`放在命令末尾表示将其放入后台。

*pcap* 文件扩展名非常重要，因为我们的 Ubuntu 主机系统只允许 `tcpdump` 读取具有该扩展名的文件。

现在，让我们再次运行 `ping`：

```
root@host01:~# kubectl exec -ti pod1 -- ping -c 3 $IP2
...
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.928/0.991/1.115 ms
```

ICMP 请求和回复已被收集，但它们在内存中被缓冲。

为了将它们转储到文件中，我们将关闭 `tcpdump`：

```
root@host01:~# killall tcpdump
12 packets captured
12 packets received by filter
0 packets dropped by kernel
```

有三个 ping，每个 ping 包括一个请求和一个回复。因此，我们可能期望有六个数据包，但事实上我们捕获了 12 个。为了理解原因，让我们打印出 `tcpdump` 收集到的数据包的详细信息：

```
root@host01:~# tcpdump -enr pings.pcap
reading from file pings.pcap, link-type LINUX_SLL (Linux cooked v1)
00:16:23...  In f2:ed:e8:04:00:cc ➊ ... 172.31.239.216 > 172.31.89.197: ICMP echo request ...
00:16:23... Out 08:00:27:b7:ef:ef ➋ ... 172.31.239.216 > 172.31.89.197: ICMP echo request ...
00:16:23...  In 08:00:27:fc:d2:36 ➌ ... 172.31.89.197 > 172.31.239.216: ICMP echo reply ...
00:16:23... Out ee:ee:ee:ee:ee:ee ➍ ... 172.31.89.197 > 172.31.239.216: ICMP echo reply ...
...
```

`tcpdump` 的 `-e` 标志打印硬件地址；否则，我们无法区分某些数据包。第一个硬件地址 ➊ 是 Pod 内部 `eth0` 的硬件地址。接下来是相同的数据包，但这次硬件地址是主机接口 ➋。然后我们看到回复，首先到达主机接口，并带有 `host02` 的硬件地址 ➌。最后，数据包被路由到对应我们 Pod 的 Calico 网络接口 ➍，我们的 `ping` 已经完成了往返。

我们现在完成了这两个 Pod，让我们删除它们：

```
root@host01:~# kubectl delete -f /opt/two-pods.yaml
pod "pod1" deleted
pod "pod2" deleted
```

对于 Kubernetes 集群来说，使用第三层路由是一个优雅的跨节点网络解决方案，因为它利用了 Linux 原生的路由和流量转发能力。然而，这意味着主机网络能看到 Pod 的 IP 地址，这可能需要安全规则的更改。例如，为了配合本书，在亚马逊网络服务（AWS）中自动设置虚拟机时，不仅配置了一个安全组以允许 Pod IP 地址空间内的所有流量，还关闭了虚拟机实例的“源/目标检查”。否则，底层 AWS 网络基础设施将拒绝传递具有意外 IP 地址的流量到我们集群的节点。

#### WeaveNet

第三层路由并不是跨节点网络的唯一解决方案。另一种选择是将容器数据包“封装”到明确从主机到主机发送的数据包中。这是流行的网络插件（如 Flannel 和 WeaveNet）采取的方法。我们将看一个 WeaveNet 的例子，但使用 Flannel 的流量看起来非常相似。

**注意**

基于 Calico 的较大集群也会使用封装技术来处理某些网络之间的流量。例如，在 AWS 中跨多个区域或可用区的集群可能需要配置 Calico 来使用封装，因为可能无法或不方便为跨区域或可用区的所有路由器配置必要的 Pod IP 路由。

因为在网络中可能会有一些定义的标准，所以有封装的标准也并不奇怪：虚拟可扩展局域网（VXLAN）。在 VXLAN 中，每个数据包都被包装在一个 UDP 数据报中并发送到目的地。

我们将使用相同的*two-pods.yaml*配置文件，在我们的 Kubernetes 集群中创建两个 Pod，这次使用的是本章示例中*weavenet*目录构建的集群。如同之前一样，我们最终会有一个 Pod 在`host01`，另一个 Pod 在`host02`：

```
root@host01:~# kubectl apply -f /opt/two-pods.yaml
pod/pod1 created
pod/pod2 created
```

让我们检查一下这些 Pod 是否正在运行，并且正确分配到它们各自的主机：

```
root@host01:~# kubectl get po -o wide
NAME   READY   STATUS    ... IP           NODE     ...
pod1   1/1     Running   ... 10.46.0.8    host01   ...
pod2   1/1     Running   ... 10.40.0.21   host02   ...
```

在这些 Pod 运行后，我们可以使用之前显示的相同命令来收集它们的 IP 地址：

```
root@host01:~# IP1=$(kubectl get po pod1 -o json | jq -r '.status.podIP')
root@host01:~# IP2=$(kubectl get po pod2 -o json | jq -r '.status.podIP')
root@host01:~# echo $IP1
10.46.0.8
root@host01:~# echo $IP2
10.40.0.21
```

请注意，分配的 IP 地址看起来与 Calico 示例完全不同。进一步探索显示地址和路由配置也有所不同，正如清单 8-4 中所示。

```
root@host01:~# kubectl exec -ti pod1 -- ip addr
...
25: eth0@if26: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1376 qdisc noqueue 
    link/ether e6:78:69:44:3d:a4 brd ff:ff:ff:ff:ff:ff
    inet 10.46.0.8/12 brd 10.47.255.255 scope global eth0
       valid_lft forever preferred_lft forever
...
root@host01:~# kubectl exec -ti pod1 -- ip route
default via 10.46.0.0 dev eth0 
10.32.0.0/12 dev eth0 scope link  src 10.46.0.8
```

*清单 8-4: WeaveNet 网络*

这一次，我们的 Pod 获得了一个大范围的`/12`网络中的 IP 地址，意味着单个网络中有超过一百万个可能的地址。在这种情况下，我们 Pod 的网络栈预计能够使用 ARP 直接识别网络上任何其他 Pod 的硬件地址，而不是像我们在 Calico 中看到的那样将流量路由到网关。

和之前一样，我们确实在这两个 Pod 之间建立了连接：

```
root@host01:~# kubectl exec -ti pod1 -- ping -c 3 $IP2
PING 10.40.0.21 (10.40.0.21): 56 data bytes
64 bytes from 10.40.0.21: seq=0 ttl=64 time=0.981 ms
64 bytes from 10.40.0.21: seq=1 ttl=64 time=0.963 ms
64 bytes from 10.40.0.21: seq=2 ttl=64 time=0.871 ms
--- 10.40.0.21 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.871/0.938/0.981 ms
```

现在我们已经运行了这个`ping`命令，我们应该期待`pod1`网络栈中的 ARP 表已经填充了`pod2`网络接口的硬件地址：

```
root@host01:~# kubectl exec -ti pod1 -- arp -n
? (10.40.0.21) at ba:75:e6:db:7c:c6 [ether]  on eth0
? (10.46.0.0) at 1a:72:78:64:36:c6 [ether]  on eth0
```

正如预期的那样，`pod1`有一个针对`pod2` IP 地址的 ARP 表项，对应于`pod2`内部的虚拟网络接口：

```
root@host01:~# kubectl exec -ti pod2 -- ip addr
...
53: eth0@if54: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1376 qdisc noqueue 
    link/ether ➊ ba:75:e6:db:7c:c6 brd ff:ff:ff:ff:ff:ff
    inet 10.40.0.21/12 brd 10.47.255.255 scope global eth0
       valid_lft forever preferred_lft forever
...
```

`pod1`的 ARP 表中的硬件地址与`pod2`虚拟网络设备的硬件地址匹配➊。为了实现这一点，WeaveNet 正在通过网络路由 ARP 请求，以便`pod2`的网络栈能够做出响应。

让我们看看跨节点的 ARP 和 ICMP 流量是如何传输的。首先，尽管 IP 地址管理可能不同，Calico 和 WeaveNet 之间的一个重要相似之处是，二者都使用 veth 对将容器连接到主机。如果你想深入探索这一点，可以使用清单 8-2 和清单 8-3 中的命令来确定`pod1`的网络命名空间，然后在`host01`上使用`ip addr`验证是否存在一个具有`link-netns`字段的`veth`设备，该字段对应于该网络命名空间。

出于我们的目的，因为我们之前已经看到过这个情况，我们假设流量是通过由 veth 对创建的虚拟网络线路传输的，并到达主机。从这里开始，我们追踪这两个 Pod 之间的 ICMP 流量。

如果我们使用与 Calico 相同的`tcpdump`捕获，我们将能够捕获到 ICMP 流量，但这只能帮助我们到达一定程度。让我们继续查看一下：

```
root@host01:~# tcpdump -w pings.pcap -i any icmp &
[1] 55999
tcpdump: listening on any, link-type LINUX_SLL (Linux cooked v1) ...
root@host01:~# kubectl exec -ti pod1 -- ping -c 3 $IP2
...
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.824/1.691/3.053 ms
root@host01:~# killall tcpdump
24 packets captured
24 packets received by filter
0 packets dropped by kernel
```

如同之前一样，我们在后台运行了`tcpdump`来捕获所有网络接口上的 ICMP 流量，运行了我们的`ping`命令，然后停止了`tcpdump`，让它写出捕获的包。这一次我们有 24 个数据包可以查看，但它们仍然不能讲述整个故事：

```
root@host01:~# tcpdump -enr pings.pcap
reading from file pings.pcap, link-type LINUX_SLL (Linux cooked v1)
16:22:08.211499   P e6:78:69:44:3d:a4 ... 10.46.0.8 > 10.40.0.21: ICMP echo request ...
16:22:08.211551 Out e6:78:69:44:3d:a4 ... 10.46.0.8 > 10.40.0.21: ICMP echo request ...
16:22:08.211553   P e6:78:69:44:3d:a4 ... 10.46.0.8 > 10.40.0.21: ICMP echo request ...
16:22:08.211745 Out e6:78:69:44:3d:a4 ... 10.46.0.8 > 10.40.0.21: ICMP echo request ...
16:22:08.212917   P ba:75:e6:db:7c:c6 ... 10.40.0.21 > 10.46.0.8: ICMP echo reply ...
16:22:08.213704 Out ba:75:e6:db:7c:c6 ... 10.40.0.21 > 10.46.0.8: ICMP echo reply ...
16:22:08.213708   P ba:75:e6:db:7c:c6 ... 10.40.0.21 > 10.46.0.8: ICMP echo reply ...
16:22:08.213724 Out ba:75:e6:db:7c:c6 ... 10.40.0.21 > 10.46.0.8: ICMP echo reply ...
...
```

这些行显示了一个单独的`ping`请求和回复的四个数据包，但硬件地址并没有发生变化。发生的情况是，这些 ICMP 数据包在网络接口之间被传递，且没有修改。然而，我们仍然没有看到实际在`host01`和`host02`之间传输的流量，因为我们从未看到任何与主机接口对应的硬件地址。

要查看主机级流量，我们需要告诉`tcpdump`捕获 UDP 流量，然后将其视为 VXLAN，这样可以使`tcpdump`识别出 ICMP 数据包的存在。

让我们重新开始捕获，这次查找 UDP 流量：

```
root@host01:~# tcpdump -w vxlan.pcap -i any udp &
...
root@host01:~# kubectl exec -ti pod1 -- ping -c 3 $IP2
...
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 1.139/1.364/1.545 ms
root@host01:~# killall tcpdump
22 packets captured
24 packets received by filter
0 packets dropped by kernel
```

这次我们将数据包数据保存在*vxlan.pcap*中。在这个例子中，`tcpdump`捕获了 22 个数据包。由于我们集群中有大量的跨 Pod 流量，而不仅仅是 ICMP 流量，您可能会看到不同的数量。

我们捕获的数据包覆盖了`host01`上的所有 UDP 流量，而不仅仅是我们的 ICMP 流量，因此在打印出清单 8-5 中显示的数据包时，我们需要进行选择。

```
root@host01:~# tcpdump -enr vxlan.pcap -T vxlan | grep -B 1 ICMP
reading from file vxlan.pcap, link-type LINUX_SLL (Linux cooked v1)
16:45:47.307949 Out 08:00:27:32:a0:28 ... 
  length 150: 192.168.61.11.50200 > 192.168.61.12.6784: VXLAN ...
e6:78:69:44:3d:a4 > ba:75:e6:db:7c:c6 ... 
  length 98: 10.46.0.8 > 10.40.0.21: ICMP echo request ...
16:45:47.308699  In 08:00:27:67:b9:da ... 
  length 150: 192.168.61.12.43489 > 192.168.61.11.6784: VXLAN ... 
ba:75:e6:db:7c:c6 > e6:78:69:44:3d:a4 ... 
  length 98: 10.40.0.21 > 10.46.0.8: ICMP echo reply ...
16:45:48.308240 Out 08:00:27:32:a0:28 ... 
  length 150: 192.168.61.11.50200 > 192.168.61.12.6784: VXLAN ... 
...
```

*清单 8-5：VXLAN 捕获*

`-T vxlan`标志告诉`tcpdump`将其看到的数据包数据视为 VXLAN 数据。这使得`tcpdump`可以深入查看并提取封装数据包中的数据，从而识别出那些被隐藏在内部的 ICMP 数据包。接着，我们使用`grep`和`-B 1`标志来查找这些 ICMP 数据包，并打印出它们之前的一行，以便查看 VXLAN 包装器。

这个捕获显示了主机的硬件地址，这表明我们已经成功捕获了在主机之间传输的流量。每个 ICMP 数据包都被封装在一个 UDP 数据报中，并通过主机网络发送。这些数据报的 IP 源和目标地址是主机网络的 IP 地址`192.168.61.11`和`192.168.61.12`，因此主机网络从未看到 Pod 的 IP 地址。然而，这些信息仍然存在于封装的 ICMP 数据包中，因此，当数据报到达目的地时，WeaveNet 能够将 ICMP 数据包发送到正确的目的地。

封装的优点是，我们所有的跨节点流量看起来就像主机之间的普通 UDP 数据报。通常，我们无需做任何额外的网络配置来允许这种流量。然而，我们也付出了代价。正如在清单 8-5 中看到的，每个 ICMP 数据包大小为 98 字节，但封装后的数据包为 150 字节。为了进行封装所需的包装器会产生网络开销，我们需要为每个发送的数据包支付这个开销。

请回顾一下清单 8-4 中的另一个结果。Pod 内部的虚拟网络接口的最大传输单元（MTU）为 1,376。这个值代表可以发送的最大数据包；任何更大的数据包必须被分段并在目的地重新组装。这个 1,376 的 MTU 远小于主机网络上的标准 1,500。Pod 接口上较小的 MTU 确保 Pod 的网络栈会进行必要的分段处理。这样，我们可以确保即使添加了封装层，主机层也不会超过 1,500。因此，如果你使用的是通过封装实现的网络插件，值得探索如何配置巨型帧，以便在主机网络上启用大于 1,500 的 MTU。

#### 选择网络插件

网络插件可以采用不同的方式来实现跨节点的网络连接。然而，正如工程学中的普遍规律，每种方法都有其权衡。第 3 层路由利用了 Linux 的原生功能，在使用网络带宽方面效率较高，但可能需要定制底层主机网络。通过 VXLAN 封装的方法适用于任何可以在主机之间发送 UDP 数据报的网络，但它会增加每个数据包的开销。

无论如何，我们的 Pods 都能满足其需求，即能够与集群中其他位置的 Pods 进行通信。实际上，配置工作和性能差异通常很小。因此，选择网络插件的最佳方式是从你的 Kubernetes 发行版推荐或默认安装的插件开始。如果你发现某些特定用例的性能无法满足要求，你可以基于实际网络流量而不是猜测，测试其他插件。

### 网络定制

某些场景可能需要比单一 Pod 网络连接跨所有集群节点更为复杂的集群网络。例如，一些受监管的行业要求某些数据（如安全审计日志）通过一个独立的网络传输。其他系统可能有专门的硬件，要求与该硬件交互的应用组件必须放置在特定的网络或虚拟局域网（VLAN）中。

网络插件架构的一个优势是 Kubernetes 集群能够容纳这些特定的网络场景。只要 Pods 有一个接口能够连接到集群的其他部分（并且能够从集群其他部分访问），Pods 就可以有额外的网络接口来提供专门的连接。

我们来看一个例子。我们将配置两个在同一节点上的 Pods，使它们拥有一个本地的仅主机网络，可以用于相互通信。由于是仅主机网络，它不提供与集群其他部分的连接，因此我们还将使用 Calico 为 Pods 提供集群网络。

由于需要配置 Calico 和我们的仅主机网络，我们将调用两个不同的 CNI 插件，它们将在 Pod 的网络命名空间中创建虚拟网络接口。如同我们在示例 8-1 中看到的那样，确实可以在一个配置文件中配置多个 CNI 插件。然而，`kubelet` 只期望其中一个 CNI 插件实际分配网络接口和 IP 地址。为了解决这个问题，我们将使用 Multus，一个设计用来调用多个插件的 CNI 插件，但会将其中一个插件视为主插件，用于向 `kubelet` 报告 IP 地址信息。Multus 还允许我们根据需要选择应用哪些 CNI 插件到每个 Pod。

我们将首先在本章的 `calico` 示例集群中安装 Multus：

```
root@host01:~# kubectl apply -f /opt/multus-daemonset.yaml
customresourcedefinition.../network-attachment-definitions... created
clusterrole.rbac.authorization.k8s.io/multus created
clusterrolebinding.rbac.authorization.k8s.io/multus created
serviceaccount/multus created
configmap/multus-cni-config created
daemonset.apps/kube-multus-ds created
```

正如文件名所示，这个 YAML 文件中的主要资源是一个 DaemonSet，它在每个主机上运行一个 Multus 容器。然而，这个文件还安装了其他几个资源，包括一个 *CustomResourceDefinition*。这个 CustomResourceDefinition 允许我们配置网络附加资源，告诉 Multus 在特定 Pod 中使用哪些 CNI 插件。

我们将在第十七章中详细查看 CustomResourceDefinitions。现在，在示例 8-6 中，我们将看到用于配置 Multus 的 NetworkAttachmentDefinition。

*netattach.yaml*

```
---
apiVersion: k8s.cni.cncf.io/v1
kind: NetworkAttachmentDefinition
metadata:
  name: macvlan-conf
spec:
  config: '{
      "cniVersion": "0.3.0",
      "type": "macvlan",
      "mode": "bridge",
      "ipam": {
        "type": "host-local",
        "subnet": "10.244.0.0/24",
        "rangeStart": "10.244.0.1",
        "rangeEnd": "10.244.0.254"
      }
    }'
```

*示例 8-6：网络附加*

`spec` 中的 `config` 字段看起来像一个 CNI 配置文件，这并不奇怪，因为 Multus 需要使用这些信息在我们要求将其添加到 Pod 时调用 `macvlan` CNI 插件。

我们需要将这个 NetworkAttachmentDefinition 添加到集群中：

```
root@host01:~# kubectl apply -f /opt/netattach.yaml 
networkattachmentdefinition.k8s.cni.cncf.io/macvlan-conf created
```

这个定义并不会立即影响任何 Pod；它只是为将来使用提供了 Multus 配置。

当然，要使用这个配置，必须调用 Multus。那么，当我们已经将 Calico 安装到这个集群时，如何实现这一点呢？答案就在*/etc/cni/net.d* 目录中，这个目录在 Multus DaemonSet 初始化时会修改我们集群中所有节点上的配置：

```
root@host01:~# ls /etc/cni/net.d
00-multus.conf  10-calico.conflist  calico-kubeconfig  multus.d
```

Multus 保留了现有的 Calico 配置文件，但添加了它自己的 *00-multus.conf* 配置文件和 *multus.d* 目录。由于 *00-multus.conf* 文件在字母排序中排在 *10-calico.conflist* 前面，`kubelet` 会在下次创建新 Pod 时开始使用它。

这是 *00-multus.conf*：

*00-multus.conf*

```
{
  "cniVersion": "0.3.1",
  "name": "multus-cni-network",
  "type": "multus",
  "capabilities": {
    "portMappings": true,
    "bandwidth": true
  },
  "kubeconfig": "/etc/cni/net.d/multus.d/multus.kubeconfig",
  "delegates": [
    {
      "name": "k8s-pod-network",
      "cniVersion": "0.3.1",
      "plugins": [
        {
          "type": "calico",
...
          }
        },
        {
          "type": "bandwidth",
...
        },
        {
          "type": "portmap",
...
        }
      ]
    }
  ]
}
```

`delegates` 字段来自 Multus 找到的 Calico 配置。这个字段用于确定 Multus 在每次调用时始终使用的默认 CNI 插件。顶层的 `capabilities` 字段是必须的，以确保 Multus 从 `kubelet` 获取所有正确的配置数据，以便能够调用 `portmap` 和 `bandwidth` 插件。

现在 Multus 已经完全设置好了，让我们用它向两个 Pod 添加一个仅主机网络。这些 Pod 的定义如下：

*local-pods.yaml*

```
---
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  annotations:
    k8s.v1.cni.cncf.io/networks: macvlan-conf
spec:
  containers:
  - name: pod1
    image: busybox
    command: 
      - "sleep"
      - "infinity"
  nodeName: host01
---
apiVersion: v1
kind: Pod
metadata:
  name: pod2
  annotations:
    k8s.v1.cni.cncf.io/networks: macvlan-conf
spec:
  containers:
  - name: pod2
    image: busybox
    command: 
      - "sleep"
      - "infinity"
  nodeName: host01
```

这一次，我们需要这两个 Pod 最终都在 `host01` 上运行，以便仅限主机的网络功能得以实现。此外，我们为每个 Pod 添加了 `k8s.v1.cni.cncf.io/networks` 注解。Multus 使用这个注解来识别应运行的额外 CNI 插件。`macvlan-conf` 这个名字与我们在 Listing 8-6 中的 NetworkAttachmentDefinition 中提供的名称匹配。

让我们创建这两个 Pod：

```
root@host01:~# kubectl apply -f /opt/local-pods.yaml
pod/pod1 created
pod/pod2 created
```

在这些 Pod 运行之后，我们可以检查它们是否各自有一个额外的网络接口：

```
root@host01:~# kubectl exec -ti pod1 -- ip addr
...
3: eth0@if12: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1450 qdisc noqueue 
    link/ether 9a:a1:db:ec:c7:91 brd ff:ff:ff:ff:ff:ff
    inet 172.31.239.198/32 brd 172.31.239.198 scope global eth0
       valid_lft forever preferred_lft forever
...
4: net1@if2: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue 
    link/ether 9e:4f:c4:47:40:07 brd ff:ff:ff:ff:ff:ff
    inet 10.244.0.2/24 brd 10.244.0.255 scope global net1
       valid_lft forever preferred_lft forever
...
root@host01:~# kubectl exec -ti pod2 -- ip addr
...
3: eth0@if13: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1450 qdisc noqueue 
    link/ether 52:08:99:a7:d2:bc brd ff:ff:ff:ff:ff:ff
    inet 172.31.239.199/32 brd 172.31.239.199 scope global eth0
       valid_lft forever preferred_lft forever
...
4: net1@if2: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue 
    link/ether a6:e5:01:82:81:82 brd ff:ff:ff:ff:ff:ff
    inet 10.244.0.3/24 brd 10.244.0.255 scope global net1
       valid_lft forever preferred_lft forever
...
```

`macvlan` CNI 插件已添加额外的 `net1` 网络接口，并使用我们在 NetworkAttachmentDefinition 中提供的 IP 地址管理配置。

这两个 Pod 现在可以通过以下接口相互通信：

```
root@host01:~# kubectl exec -ti pod1 -- ping -c 3 10.244.0.3
PING 10.244.0.3 (10.244.0.3): 56 data bytes
64 bytes from 10.244.0.3: seq=0 ttl=64 time=3.125 ms
64 bytes from 10.244.0.3: seq=1 ttl=64 time=0.192 ms
64 bytes from 10.244.0.3: seq=2 ttl=64 time=0.085 ms

--- 10.244.0.3 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.085/1.134/3.125 ms
```

这种通信通过 `macvlan` CNI 插件创建的桥接网络进行，而不是通过 Calico 进行。

请记住，我们在这里的目的仅仅是演示自定义网络，而无需集群主机外部的任何特定 VLAN 或复杂设置。对于实际的集群，这种仅限主机的网络价值有限，因为它限制了 Pod 的部署位置。在这种情况下，将两个容器放入同一个 Pod 可能更为可取，这样它们总是会一起调度，并可以使用 `localhost` 进行通信。

### 最后的思考

在这一章中，我们已经看了很多网络接口和流量流动。大多数情况下，了解集群中的每个 Pod 都会从 Pod 网络中分配一个 IP 地址，并且集群中的任何 Pod 都可以与任何其他 Pod 通信，且可以被访问，这就足够了。任何 Kubernetes 网络插件都可以提供这种功能，无论它们使用的是第 3 层路由、VXLAN 封装，还是两者兼而有之。

同时，集群中确实会发生网络问题，因此集群管理员和用户必须理解流量如何在主机之间流动，以及这些流量对主机网络的表现，以便调试交换机和主机配置问题，或者仅仅为了构建能够充分利用集群的应用程序。

我们还没有完成使 Kubernetes 集群完全功能所需的网络层。在下一章中，我们将探讨 Kubernetes 如何在 Pod 网络之上提供服务层，以提供负载均衡和自动故障切换，并结合 Ingress 网络层使容器服务在集群外部可访问。
