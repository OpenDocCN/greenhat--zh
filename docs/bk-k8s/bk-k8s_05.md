# 第四章：网络命名空间

![image](img/common01.jpg)

理解容器网络是构建基于容器化微服务的现代应用程序时面临的最大挑战。首先，即使没有引入容器，网络也是复杂的。仅仅从一台物理服务器发送一个简单的`ping`到另一台物理服务器，就涉及了多个抽象层。其次，容器引入了额外的复杂性，因为每个容器都有自己的一组虚拟网络设备，使其看起来像一个独立的机器。更重要的是，像 Kubernetes 这样的容器编排框架通过增加一个“覆盖”网络，使得容器即使运行在不同的主机上也能进行通信，从而增加了更多的复杂性。

在本章中，我们将详细了解容器网络是如何工作的。我们将查看容器的虚拟网络设备，包括每个网络设备如何分配一个可以访问主机的独立 IP 地址。我们还将看到，同一主机上的容器如何通过桥接设备连接到彼此，以及容器设备如何配置以路由流量。最后，我们将探讨如何使用地址转换，使容器能够连接到其他主机，而不会暴露容器网络内部结构到主机的网络上。

### 网络隔离

在第二章中，我们讨论了隔离对于系统可靠性的重要性，因为进程通常不能影响它们看不见的东西。这是容器网络隔离的重要原因之一。另一个原因是配置的简便性。要运行一个作为服务器的进程，比如一个 Web 服务器，我们需要选择一个或多个网络接口来监听该服务器，并且需要选择一个端口号来监听。我们不能让两个进程在同一个接口的相同端口上监听。

因此，作为服务器的进程通常会提供一种配置方式，让我们指定其监听连接的端口。然而，这仍然要求我们了解其他服务器的情况以及它们使用的端口，从而确保没有冲突。这对于像 Kubernetes 这样的容器编排框架来说几乎是不可能的，因为新的进程可以随时出现，来自不同的用户，并且可能需要监听任何端口。

解决这个问题的方法是为每个容器提供独立的虚拟网络接口。这样，容器中的进程可以选择任何它想要的端口——它将监听一个与另一个容器中进程不同的网络接口。我们来看一个简短的例子。

**注意**

*本书的示例仓库位于* [`github.com/book-of-kubernetes/examples`](https://github.com/book-of-kubernetes/examples)。*有关设置的详细信息，请参见第 xx 页中的“运行示例”。*

我们将运行两个实例的 NGINX Web 服务器；每个实例将在端口 80 上监听。与以前一样，我们将使用 CRI-O 和 `crictl`，但我们将使用一个脚本来减少输入：

```
root@host01:~# cd /opt
root@host01:/opt# source nginx.sh
...
```

在 *nginx.sh* 之前的 `source` 很重要；它确保脚本以一种方式运行，使得它设置的环境变量在我们的 shell 中对未来的命令可用。在 *nginx.sh* 中是我们在前几章中使用过的常规命令 `crictl runp`、`crictl create` 和 `crictl start`。YAML 文件也与我们以前看到的示例非常相似；唯一的区别是我们使用了安装有 NGINX 的容器镜像。

让我们验证我们有两个 NGINX 服务器正在运行：

```
root@host01:/opt# crictl ps
CONTAINER      IMAGE            ... NAME    ...
ae341010886ae  .../nginx:latest ... nginx2  ...
6a95800b16f15  .../nginx:latest ... nginx1  ...
```

我们还可以验证两个 NGINX 服务器都在监听端口 80，这是 Web 服务器的标准端口：

```
root@host01:/opt# crictl exec $N1C_ID cat /proc/net/tcp
  sl  local_address ...
   0: 00000000:0050 ...
root@host01:/opt# crictl exec $N2C_ID cat /proc/net/tcp
  sl  local_address ...
   0: 00000000:0050 ...
```

通过打印 */proc/net/tcp* 我们查看开放的端口，因为我们需要在 NGINX 容器内运行这个命令，而我们没有标准的 Linux 命令，如 `netstat` 或 `ss`。正如我们在 第二章 中看到的，在容器中，我们有一个单独的 `mnt` 命名空间为每个容器提供单独的文件系统，因此只有在该单独文件系统中可用的可执行文件才能在该命名空间中运行。

在这两种情况下显示的端口是 `0050`，这是十六进制中的端口 80 在十进制中的表示。如果这两个进程在没有网络隔离的同一系统上运行，它们都无法同时监听端口 80，但在这种情况下，这两个 NGINX 实例有单独的网络接口。为了进一步探索这一点，让我们启动一个新的 BusyBox 容器：

```
root@host01:/opt# source busybox.sh
...
```

现在除了我们的两个 NGINX 容器外，BusyBox 也在运行：

```
root@host01:/opt# crictl ps
CONTAINER      IMAGE              ... NAME    ...
189dd26766d26  .../busybox:latest ... busybox ...
ae341010886ae  .../nginx:latest   ... nginx2  ...
6a95800b16f15  .../nginx:latest   ... nginx1  ...
```

让我们在容器内部启动一个 shell：

```
root@host01:/opt# crictl exec -ti $B1C_ID /bin/sh
/ #
```

列表 4-1 显示了容器的网络设备和地址。

```
/ # ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue ...
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
        valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
        valid_lft forever preferred_lft forever
3: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue 
    link/ether 9a:7c:73:2f:f7:1a brd ff:ff:ff:ff:ff:ff
    inet 10.85.0.4/16 brd 10.85.255.255 scope global eth0
        valid_lft forever preferred_lft forever
    inet6 fe80::987c:73ff:fe2f:f71a/64 scope link 
        valid_lft forever preferred_lft forever
```

*列表 4-1: BusyBox 网络*

忽略标准的环回设备，我们看到一个网络设备，其 IP 地址为 `10.85.0.4`。这与主机的 IP 地址 `192.168.61.11` 根本不对应；它在完全不同的网络上。由于我们的容器位于单独的网络上，我们可能不希望能够从容器内部 `ping` 底层主机系统，但这是有效的，正如 列表 4-2 所示。

```
/ # ping -c 1 192.168.61.11
PING 192.168.61.11 (192.168.61.11): 56 data bytes
64 bytes from 192.168.61.11: seq=0 ttl=64 time=7.471 ms

--- 192.168.61.11 ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 7.471/7.471/7.471 ms
```

*列表 4-2: BusyBox ping 测试*

要使流量从我们的容器到主机网络，路由表中必须有一个条目来实现这一点。正如 列表 4-3 所示，我们可以使用 `ip` 命令验证这一点。

```
/ # ip route
default via 10.85.0.1 dev eth0 
10.85.0.0/16 dev eth0 scope link  src 10.85.0.4
```

*列表 4-3: BusyBox 路由*

预期地，存在一个默认路由。当我们发送 `ping` 时，我们的 BusyBox 容器连接到 `10.85.0.1`，然后有能力将 `ping` 转发直至到达 `192.168.61.11`。

我们将保持这三个容器继续运行以进一步探索它们，但让我们退出 BusyBox shell 返回主机：

```
/ # exit
```

从容器内部查看网络可以解释为什么我们的两个 NGINX 服务器都能监听 80 端口。如前所述，只有一个进程能够监听特定接口的端口，但当然，如果每个 NGINX 服务器都有一个单独的网络接口，就不会发生冲突。

### 网络命名空间

CRI-O 使用 Linux 网络命名空间来创建这种隔离。在第二章中，我们简要地探讨了网络命名空间；在本章中，我们将更详细地讨论它们。

首先，让我们使用`lsns`命令列出 CRI-O 为我们的容器创建的网络命名空间：

```
root@host01:/opt# lsns -t net
        NS TYPE NPROCS   PID USER    NETNSID NSFS                   COMMAND
4026531992 net     114     1 root unassigned                        /sbin/init
4026532196 net       4  5801 root          0 /run/netns/ab8be6e6... /pause
4026532272 net       4  5937 root          1 /run/netns/8ffe0394... /pause
4026532334 net       2  6122 root          2 /run/netns/686d71d9... /pause
```

除了用于所有不在容器中的进程的根网络命名空间外，我们还看到三个网络命名空间，每个命名空间对应一个我们创建的 Pod。

当我们使用 CRI-O 与`crictl`时，网络命名空间实际上属于 Pod。这里列出的`pause`进程存在的目的是为了让命名空间在 Pod 内的容器进出时能够持续存在。

在上一个示例中，有四个网络命名空间。第一个是我们主机启动时创建的根命名空间。其他三个是为我们启动的每个容器创建的：两个 NGINX 容器和一个 BusyBox 容器。

#### 检查网络命名空间

为了了解网络命名空间是如何工作的并进行操作，我们将使用`ip netns`命令列出网络命名空间：

```
root@host01:/opt# ip netns list
7c185da0-04e2-4321-b2eb-da18ceb5fcf6 (id: 2)
d26ca6c6-d524-4ae2-b9b7-5489c3db92ce (id: 1)
38bbb724-3420-46f0-bb50-9a150a9f0889 (id: 0)
```

这个命令会在不同的配置位置查找网络命名空间，因此只列出了三个容器命名空间。

我们希望获取我们 BusyBox 容器的网络命名空间。它是三个列出的命名空间之一，我们可以猜测它是标记为`(id: 2)`的那个，因为我们最后创建了它，但我们也可以使用`crictl`和`jq`来提取我们需要的信息：

```
root@host01:/opt# NETNS_PATH=$(crictl inspectp $B1P_ID |
  jq -r '.info.runtimeSpec.linux.namespaces[]|select(.type=="network").path')
root@host01:/opt# echo $NETNS_PATH
/var/run/netns/7c185da0-04e2-4321-b2eb-da18ceb5fcf6
root@host01:/opt# NETNS=$(basename $NETNS_PATH)
root@host01:/opt# echo $NETNS
7c185da0-04e2-4321-b2eb-da18ceb5fcf6
```

如果单独运行`crictl inspectp $B1P_ID`，你将看到关于 BusyBox Pod 的大量信息。在所有这些信息中，我们只需要关于网络命名空间的信息，因此我们使用`jq`分三步提取这些信息。首先，它会深入到 JSON 数据中，提取与此 Pod 相关的所有命名空间。然后，它只选择具有`type`字段为`network`的命名空间。最后，它提取该命名空间的`path`字段，并将其存储在环境变量`NETNS_PATH`中。

`crictl`返回的值是网络命名空间在*/var/run*下的完整路径。对于接下来的命令，我们只需要命名空间的值，所以我们使用`basename`来去掉路径部分。此外，由于如果将这些信息分配给环境变量会更易于使用，我们这么做了，然后使用`echo`打印出该值，以便我们确认一切正常。

当然，对于交互式调试，您通常可以仅滚动浏览整个`crictl inspectp`（用于 Pods）和`crictl inspect`（用于容器）的内容，并选择您想要的值。但是使用`jq`提取数据的这种方法在脚本编写或减少手动扫描输出量方面非常有用。

现在我们从`crictl`中提取了 BusyBox 的网络命名空间，让我们看看分配给该命名空间的进程有哪些：

```
root@host01:/opt# ps --pid $(ip netns pids $NETNS)
PID TTY      STAT   TIME COMMAND
5800 ?        Ss     0:00 /pause
5839 ?        Ss     0:00 /bin/sleep 36000
```

如果我们只运行`ip netns pids $NETNS`，我们将得到一个进程 ID（PID）列表，但没有额外的信息。我们将该输出发送到`ps --pid`，这样我们就可以看到命令的名称。正如预期的那样，我们看到了我们在运行 BusyBox 容器时指定的`pause`进程和`sleep`进程。

在上一节中，我们使用`crictl exec`在容器内运行了一个 Shell，这使我们能够看到该网络命名空间中可用的网络接口。现在我们知道了网络命名空间的 ID，我们可以使用`ip netns exec`从网络命名空间内单独运行命令。使用`ip netns exec`非常强大，因为它不仅限于网络命令，还可以是任何进程，比如 Web 服务器。但请注意，这与完全在容器内部运行不同，因为我们没有进入容器的任何其他命名空间（例如用于进程隔离的`pid`命名空间）。

接下来，让我们在 BusyBox 网络命名空间内尝试`ip addr`命令：

```
root@host01:/opt# ip netns exec $NETNS ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue ...
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
        valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
        valid_lft forever preferred_lft forever
3: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue ...
    link/ether 9a:7c:73:2f:f7:1a brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.85.0.4/16 brd 10.85.255.255 scope global eth0
        valid_lft forever preferred_lft forever
    inet6 fe80::987c:73ff:fe2f:f71a/64 scope link 
        valid_lft forever preferred_lft forever
```

这里看到的网络设备和 IP 地址列表与我们在清单 4-1 内部运行 BusyBox 容器时看到的内容相匹配。 CRI-O 正在创建这些网络设备并将它们放置在网络命名空间中。（当我们查看第八章节关于 Kubernetes 网络时，我们将看到 CRI-O 是如何配置执行容器网络的。）现在，让我们看看如何创建自己的设备和网络命名空间以进行网络隔离。这也将向我们展示在容器网络出现问题时如何进行调试。

#### 创建网络命名空间

我们可以用一个命令创建一个网络命名空间：

```
root@host01:/opt# ip netns add myns
```

这个新的命名空间立即出现在列表中：

```
root@host01:/opt# ip netns list
myns
7c185da0-04e2-4321-b2eb-da18ceb5fcf6 (id: 2)
d26ca6c6-d524-4ae2-b9b7-5489c3db92ce (id: 1)
38bbb724-3420-46f0-bb50-9a150a9f0889 (id: 0)
```

这个命名空间目前还不是很有用；它有一个回环接口，但没有其他内容：

```
root@host01:/opt# ip netns exec myns ip addr
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

此外，即使是回环接口也是关闭的，因此无法使用。让我们快速修复它：

```
root@host01:/opt# ip netns exec myns ip link set dev lo up
root@host01:/opt# ip netns exec myns ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue ...
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
```

回环接口现在已经启动，并且具有`127.0.0.1`的典型 IP 地址。现在，在这个网络命名空间中基本的回环`ping`将会起作用：

```
root@host01:/opt# ip netns exec myns ping -c 1 127.0.0.1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.035 ms

--- 127.0.0.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.035/0.035/0.035/0.000 ms
```

`ping`回环网络接口的能力是任何网络堆栈的有用初步测试，因为它显示了发送和接收数据包的能力。因此，我们现在在新的网络命名空间中拥有一个基本的工作网络堆栈，但它仍然不是特别有用，因为回环接口本身无法与系统上的其他任何东西通信。我们需要在此网络命名空间中添加另一个网络设备，以便与主机和其他网络建立连接。

为此，我们将创建一个*虚拟以太网*（veth）设备。你可以将 veth 视为一根虚拟网络电缆。像网络电缆一样，它有两个端口，任何从一个端口进入的东西都会从另一个端口出来。因此，通常使用术语*veth 对*。

我们从一个创建 veth 对的命令开始：

```
root@host01:/opt# ip link add myveth-host type veth \
                  peer myveth-myns netns myns
```

该命令做了三件事：

1.  创建一个名为`myveth-host`的 veth 设备

1.  创建一个名为`myveth-myns`的 veth 设备

1.  将设备`myveth-myns`放置到网络命名空间`myns`中

veth 对的主机端出现在主机的常规网络设备列表中：

```
root@host01:/opt# ip addr
...
8: myveth-host@if2: <BROADCAST,MULTICAST> mtu 1500 ... state DOWN ...
    link/ether fe:7a:5d:86:00:d9 brd ff:ff:ff:ff:ff:ff link-netns myns
```

该输出显示了`myveth-host`，并且它连接到了网络命名空间`myns`中的设备。

如果你自己运行此命令并查看主机网络设备的完整列表，你会注意到每个容器网络命名空间都有额外的`veth`设备。这些设备是 CRI-O 在我们部署 NGINX 和 BusyBox 时创建的。

同样，我们可以看到我们的`myns`网络命名空间有了一个新的网络接口：

```
root@host01:/opt# ip netns exec myns ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue ...
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: myveth-myns@if8: <BROADCAST,MULTICAST> mtu 1500 ... state DOWN ...
    link/ether 26:0f:64:a8:37:1f brd ff:ff:ff:ff:ff:ff link-netnsid 0
```

如之前所述，这个接口当前是关闭的。我们需要启动 veth 对的两端，才能开始通信。我们还需要为`myveth-myns`端分配一个 IP 地址，以使其能够通信：

```
root@host01:/opt# ip netns exec myns ip addr add 10.85.0.254/16 \
                  dev myveth-myns
root@host01:/opt# ip netns exec myns ip link set dev myveth-myns up
root@host01:/opt# ip link set dev myveth-host up
```

一个快速检查确认我们已经成功配置了 IP 地址并启动了网络：

```
root@host01:/opt# ip netns exec myns ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue ...
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: myveth-myns@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> ... state UP ...
    link/ether 26:0f:64:a8:37:1f brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.85.0.254/16 scope global myveth-myns
       valid_lft forever preferred_lft forever
    inet6 fe80::240f:64ff:fea8:371f/64 scope link 
       valid_lft forever preferred_lft forever
```

除了回环接口，我们现在还看到一个具有 IP 地址`10.85.0.254`的附加接口。如果我们尝试`ping`这个新的 IP 地址，会发生什么呢？事实证明，我们确实可以`ping`它，但只能在网络命名空间内部进行：

```
   root@host01:/opt# ip netns exec myns ping -c 1 10.85.0.254
   PING 10.85.0.254 (10.85.0.254) 56(84) bytes of data.
   64 bytes from 10.85.0.254: icmp_seq=1 ttl=64 time=0.030 ms

   --- 10.85.0.254 ping statistics ---
➊ 1 packets transmitted, 1 received, 0% packet loss, time 0ms
   rtt min/avg/max/mdev = 0.030/0.030/0.030/0.000 ms
   root@host01:/opt# ping -c 1 10.85.0.254
   PING 10.85.0.254 (10.85.0.254) 56(84) bytes of data.
   From 10.85.0.1 icmp_seq=1 Destination Host Unreachable

   --- 10.85.0.254 ping statistics ---
➋ 1 packets transmitted, 0 received, +1 errors, 100% packet loss, time 0ms
```

第一个`ping`命令通过`ip netns exec`运行，以便在网络命名空间内运行，显示了成功的响应➊。然而，第二个`ping`命令没有通过`ip netns exec`运行，显示没有接收到数据包➋。问题在于，我们已经成功创建了一个网络命名空间中的网络接口，并且 veth 对的另一端在主机网络上，但我们没有在主机上连接相应的网络设备，因此没有主机网络接口可以与网络命名空间中的接口通信。

与此同时，当我们从 BusyBox 容器中运行`ping`测试时，在清单 4-2 中，我们能够顺利地`ping`主机。显然，CRI-O 在创建容器时为我们进行了更多配置。让我们在下一节中探讨这一点。

### 桥接接口

veth 对的主机端目前没有连接到任何设备，因此我们还不能与外界进行通信也就不足为奇。为了修复这个问题，让我们来看一下 CRI-O 创建的其中一个 veth 对：

```
root@host01:/opt# ip addr
...
7: veth062abfa6@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> ... master cni0 ...
    link/ether fe:6b:21:9b:d0:d2 brd ff:ff:ff:ff:ff:ff link-netns ...
    inet6 fe80::fc6b:21ff:fe9b:d0d2/64 scope link 
       valid_lft forever preferred_lft forever
...
```

与我们创建的接口不同，这个接口指定了 `master cni0`，表明它属于一个 *网络桥接器*。网络桥接器用于将多个接口连接在一起。你可以将它视为一个以太网交换机，因为它根据接口的媒体访问控制（MAC）地址来路由流量。

我们可以在主机的网络设备列表中看到桥接器 `cni0`：

```
root@host01:/opt# ip addr
...
4: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue ...
    link/ether 8e:0c:1c:7d:94:75 brd ff:ff:ff:ff:ff:ff
    inet 10.85.0.1/16 brd 10.85.255.255 scope global cni0
       valid_lft forever preferred_lft forever
    inet6 fe80::8c0c:1cff:fe7d:9475/64 scope link 
 valid_lft forever preferred_lft forever
...
```

这个桥接器比典型的以太网交换机更智能，它提供了一些防火墙和路由功能。它的 IP 地址也是 `10.85.0.1`。这个 IP 地址与我们在 Listing 4-3 中看到的 BusyBox 容器默认路由相同，因此我们已经开始解开 BusyBox 容器能够与其网络外部主机通信的谜团。

#### 向桥接器添加接口

要检查桥接器并向其添加设备，我们将使用 `brctl` 命令。首先，让我们检查桥接器：

```
root@host01:/opt# brctl show
bridge name     bridge id               STP enabled     interfaces
cni0            8000.8e0c1c7d9475       no              veth062abfa6
                                                        veth43ab68cd
                                                        vetha251c619
```

桥接器 `cni0` 上有三个接口，分别对应我们运行的三个容器的 veth 对的主机端（两个 NGINX 和一个 BusyBox）。我们可以利用这个现有的桥接器来为我们创建的网络命名空间设置网络连接：

```
root@host01:/opt# brctl addif cni0 myveth-host
root@host01:/opt# brctl show
bridge name     bridge id               STP enabled     interfaces
cni0            8000.8e0c1c7d9475       no              myveth-host
                                                        veth062abfa6
                                                        veth43ab68cd
                                                        vetha251c619
```

现在我们 veth 对的主机端已连接到桥接器，这意味着我们现在可以从主机使用 `ping` 命令测试与命名空间的连接：

```
   root@host01:/opt# ping -c 1 10.85.0.254
   PING 10.85.0.254 (10.85.0.254) 56(84) bytes of data.
   64 bytes from 10.85.0.254: icmp_seq=1 ttl=64 time=0.194 ms

   --- 10.85.0.254 ping statistics ---
➊ 1 packets transmitted, 1 received, 0% packet loss, time 0ms
   rtt min/avg/max/mdev = 0.194/0.194/0.194/0.000 ms
```

数据包被接收 ➊ 的事实表明我们已经建立了一个有效的连接。我们应该为它的成功感到高兴，但如果我们真的想理解这一点，我们不能仅仅满足于说“我们可以从主机 ping 这个接口”。我们需要更具体地了解流量是如何流动的。

#### 跟踪流量

让我们实际跟踪一下这个流量，看看在我们运行 `ping` 命令时发生了什么。我们将使用 `tcpdump` 打印流量。首先，让我们在后台启动一个 `ping` 命令，以便产生一些流量来跟踪：

```
root@host01:/opt# ping 10.85.0.254 >/dev/null 2>&1 &
...
```

我们将输出发送到 */dev/null*，以免它干扰到我们的会话。现在，让我们使用 `tcpdump` 来查看流量：

```
root@host01:/opt# timeout 1s tcpdump -i any -n icmp
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on any, link-type LINUX_SLL (Linux cooked v1), ...
17:37:33.204863 IP 10.85.0.1 > 10.85.0.254: ICMP echo request, ...
17:37:33.204894 IP 10.85.0.1 > 10.85.0.254: ICMP echo request, ...
17:37:33.204936 IP 10.85.0.254 > 10.85.0.1: ICMP echo reply, ...
17:37:33.204936 IP 10.85.0.254 > 10.85.0.1: ICMP echo reply, ...

4 packets captured
4 packets received by filter
0 packets dropped by kernel
root@host01:/opt# killall ping
```

我们使用 `timeout` 来防止 `tcpdump` 无限运行，之后我们还会使用 `killall` 来停止 `ping` 命令并终止其在后台的运行。

输出显示 `ping` 来自桥接接口，该接口的 IP 地址是 `10.85.0.1`。这是因为主机的路由表设置：

```
root@host01:/opt# ip route
...
10.85.0.0/16 dev cni0 proto kernel scope link src 10.85.0.1 
192.168.61.0/24 dev enp0s8 proto kernel scope link src 192.168.61.11
```

当 CRI-O 创建了桥接器并配置了其 IP 地址时，它还设置了一条路由，确保所有目标为 `10.85.0.0/16` 网络的流量（即从 `10.85.0.0` 到 `10.85.255.255` 的所有流量）都会通过 `cni0`。这足以让 `ping` 命令知道如何发送数据包，桥接器处理剩下的工作。

事实上，`ping`来自`10.85.0.1`网桥接口而不是`192.168.61.11`主机接口，实际上有很大的区别，我们可以通过尝试从命名空间向主机网络运行`ping`来看到这一点。让我们尝试从命名空间内部向主机网络进行`ping`：

```
root@host01:/opt# ip netns exec myns ping -c 1 192.168.61.11
ping: connect: Network is unreachable
```

这里的问题是我们的网络命名空间中的接口不知道如何到达主机网络。网桥是可用的，并愿意将流量路由到主机网络，但我们尚未配置必要的路由来使用它。让我们现在来做这个：

```
root@host01:/opt# ip netns exec myns ip route add default via 10.85.0.1
```

现在`ping`命令有效了：

```
root@host01:/opt# ip netns exec myns ping -c 1 192.168.61.11
PING 192.168.61.11 (192.168.61.11) 56(84) bytes of data.
64 bytes from 192.168.61.11: icmp_seq=1 ttl=64 time=0.097 ms

--- 192.168.61.11 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.097/0.097/0.097/0.000 ms
```

这说明了在调试网络问题时需要记住的一个重要规则：很容易就会对网络流量的实际发送和接收情况下结论。往往需要使用跟踪工具来查看实际的流量情况，没有什么可以替代这一点。

**主机上的 IP 地址**

此方法并不是唯一能实现从主机到网络命名空间的连接性的方法。我们还可以直接为 veth 对的主机端分配 IP 地址。然而，即使这样做可以使主机能够与我们的网络命名空间通信，但它不会提供多个网络命名空间之间进行通信的方法。使用桥接接口，正如 CRI-O 所做的那样，能够在主机上互连所有容器，使它们看起来都在同一个网络上。

这也解释了为什么我们没有给 veth 对的主机端分配 IP 地址。在使用网桥时，只有网桥接口会获得 IP 地址。添加到网桥的接口不会获得 IP 地址。

在进行最后一次更改后，看起来我们已经匹配了我们的容器的网络配置，但我们仍然缺少与`host01`之外的更广泛网络通信的能力。我们可以通过尝试从我们的网络命名空间向`host02`进行`ping`来演示这一点，`host02`位于与`host01`相同的内部网络上，并具有 IP 地址`192.168.61.12`。如果我们尝试从我们的 BusyBox 容器进行`ping`，它会成功：

```
root@host01:/opt# crictl exec $B1C_ID ping -c 1 192.168.61.12
PING 192.168.61.12 (192.168.61.12): 56 data bytes
64 bytes from 192.168.61.12: seq=0 ttl=63 time=0.816 ms

--- 192.168.61.12 ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 0.816/0.816/0.816 ms
```

`ping`输出报告收到一个数据包。但是，如果我们尝试使用我们创建的网络命名空间执行相同的命令，它却不起作用：

```
root@host01:/opt# ip netns exec myns ping -c 1 192.168.61.12
PING 192.168.61.12 (192.168.61.12) 56(84) bytes of data.

--- 192.168.61.12 ping statistics ---
1 packets transmitted, 0 received, 100% packet loss, time 0ms
```

此命令报告未接收到任何数据包。

实际上，我们应该感到惊讶的是，我们的 BusyBox 容器的`ping`命令确实有效了。毕竟，`host02`并不知道任何关于 BusyBox 容器、`cni0`网桥接口或容器所在的`10.85.0.0/16`网络的信息。`host02`如何可能与我们的 BusyBox 容器进行`ping`通信？要理解这一点，我们需要看看网络伪装。

### 伪装

*伪装*，也称为网络地址转换（NAT），在网络中每天都会使用。例如，大多数家庭连接到互联网时，只有一个可以从互联网访问的 IP 地址，但家庭网络内的许多设备也需要连接互联网。路由器的工作就是让所有来自该网络的流量看起来都是从单一 IP 地址发出的。它通过重写出站流量的*源*IP 地址，并跟踪所有出站连接，以便它可以重写任何回复的*目标*IP 地址来实现这一点。

**注意**

*我们在这里讨论的 NAT 类型在技术上被称为源 NAT（SNAT）。不过不要过于纠结于这个名称；为了让它正常工作，任何回复数据包必须将其目标地址重新写入。这里的“源”一词意味着，当发起新连接时，源地址是被重写的。*

伪装听起来正是我们需要的，用于将运行在`10.85.0.0/16`网络中的容器连接到主机网络`192.168.61.0/24`，实际上它确实是这样工作的。当我们从 BusyBox 容器发送 ping 时，源 IP 地址被重写，使得 ping 看起来是来自`host01`的 IP `192.168.61.11`。当`host02`回应时，它将回复发送到`192.168.61.11`，但是目标地址被重写，最终实际上是发送到了 BusyBox 容器。

让我们追踪一下`ping`流量，直到整个过程完成，以便演示：

```
root@host01:/opt# crictl exec $B1C_ID ping 192.168.61.12 >/dev/null 2>&1 &
[1] 6335
root@host01:/opt# timeout 1s tcpdump -i any -n icmp
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on any, link-type LINUX_SLL (Linux cooked v1)...
18:53:44.310789 IP 10.85.0.4 ➊ > 192.168.61.12: ICMP echo request, id 12, seq 17...
18:53:44.310789 IP 10.85.0.4 > 192.168.61.12: ICMP echo request, id 12, seq 17...
18:53:44.310876 IP 192.168.61.11 ➋ > 192.168.61.12: ICMP echo request, id 12, seq 17...
18:53:44.311619 IP 192.168.61.12 > 192.168.61.11: ICMP echo reply, ➌ id 12, seq 17...
18:53:44.311648 IP 192.168.61.12 > 10.85.0.4: ➍ ICMP echo reply, id 12, seq 17...
18:53:44.311656 IP 192.168.61.12 > 10.85.0.4: ICMP echo reply, id 12, seq 17...

6 packets captured
6 packets received by filter
0 packets dropped by kernel
root@host01:/opt# killall ping
```

当`ping`从我们的 BusyBox 容器中发起时，它的源 IP 地址是`10.85.0.4` ➊。这个地址被重写，使得`ping`看起来是来自主机 IP `192.168.61.11` ➋。当然，`host02`知道如何响应来自该地址的`ping`，所以`ping`得到了回复 ➌。此时，伪装的另一部分开始生效，目标地址被重写为`10.85.0.4` ➍。最终，BusyBox 容器能够向一个独立主机发送数据包并接收到回复。

为了完成我们网络命名空间的设置，我们需要一个类似的规则来伪装来自`10.85.0.254`的流量。我们可以从使用`iptables`查看 CRI-O 创建的规则开始，看看它在配置容器时做了什么：

```
root@host01:/opt# iptables -t nat -n -L
...
Chain POSTROUTING (policy ACCEPT)
target                        prot opt source    destination ...
CNI-f82910b3a7e28baf6aedc0d3  all  --  10.85.0.2 anywhere    ...
CNI-7f8aa3d8a4f621b186149f43  all  --  10.85.0.3 anywhere    ...
CNI-48ad69d30fe932fda9ea71d2  all  --  10.85.0.4 anywhere    ...

Chain CNI-48ad69d30fe932fda9ea71d2 (1 references)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             10.85.0.0/16 ...
MASQUERADE all  --  anywhere             !224.0.0.0/4 ...
...
```

伪装从连接发起时开始；在这种情况下，当流量的源地址位于`10.85.0.0/16`网络时就会开始。为此，使用了`POSTROUTING`链，因为它会处理所有出站流量。`POSTROUTING`链中有一条针对每个容器的规则；每条规则都会调用该容器的`CNI`链。

为了简洁起见，只展示了三个`CNI`链中的一个。其他两个是相同的。`CNI`链首先会接受所有本地容器网络的流量，因此这些流量不会被伪装。然后，它为所有流量设置伪装（除了`224.0.0.0/4`，这是无法伪装的多播流量，因为无法正确路由回复）。

这个配置中缺少的是来自`10.85.0.254`的流量的匹配设置，`10.85.0.254`是我们在网络命名空间中分配给接口的 IP 地址。让我们添加这个设置。首先，在`nat`表中创建一个新的链：

```
root@host01:/opt# iptables -t nat -N chain-myns
```

接下来，添加一个规则来接受本地网络的所有流量：

```
root@host01:/opt# iptables -t nat -A chain-myns -d 10.85.0.0/16 -j ACCEPT
```

现在，所有剩余的流量（除了组播）应该都被伪装：

```
root@host01:/opt# iptables -t nat -A chain-myns \
                  ! -d 224.0.0.0/4 -j MASQUERADE
```

最后，告诉`iptables`对于来自`10.85.0.254`的任何流量使用这个链：

```
root@host01:/opt# iptables -t nat -A POSTROUTING -s 10.85.0.254 -j chain-myns
```

我们可以通过重新列出规则来验证我们是否正确完成了所有操作：

```
root@host01:/opt# iptables -t nat -n -L
...
Chain POSTROUTING (policy ACCEPT)
target      prot opt source               destination
chain-myns  all  --  10.85.0.254          anywhere            
...
Chain chain-myns (1 references)
target      prot opt source               destination         
ACCEPT      all  --  anywhere             10.85.0.0/16        
MASQUERADE  all  --  anywhere             !224.0.0.0/4
```

看起来我们已经得到了所需的配置，因为这个配置与我们为 BusyBox 容器配置虚拟网络设备的方式相匹配。为了确认，让我们再次尝试对`host02`进行`ping`：

```
root@host01:/opt# ip netns exec myns ping -c 1 192.168.61.12
PING 192.168.61.12 (192.168.61.12) 56(84) bytes of data.
64 bytes from 192.168.61.12: icmp_seq=1 ttl=63 time=0.843 ms

--- 192.168.61.12 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.843/0.843/0.843/0.000 ms
```

成功！我们已经完全复制了 CRI-O 为我们的容器提供的网络隔离和连接性。

### 最后的想法

容器网络在运行容器时看起来 deceptively 简单。每个容器都被提供一组自己的网络设备，避免了端口冲突的问题，也减少了一个容器对另一个容器的影响。然而，正如我们在本章中所看到的，这种“简单”的网络隔离需要一些复杂的配置，不仅仅是隔离，还需要实现容器之间以及容器与其他网络之间的连接性。在第二部分中，当我们正确引入 Kubernetes 后，我们将回到容器网络，并展示当我们需要连接在不同主机上运行的容器并在多个容器实例之间负载均衡流量时，复杂性如何增加。

目前，在我们进入 Kubernetes 之前，还有一个关键主题需要处理，就是容器存储的工作原理。我们需要理解容器存储是如何工作的，包括当启动一个新的容器时，作为基础文件系统使用的容器镜像，以及正在运行的容器使用的临时存储。在下一章，我们将探讨容器存储是如何简化应用部署的，以及如何通过使用分层文件系统来节省存储空间并提高效率。
