# 第十二章：容器运行时

![image](img/common01.jpg)

在上一章中，我们了解了控制平面如何管理和监控集群的状态。然而，实际上是容器运行时，特别是 `kubelet` 服务，负责创建、启动、停止和删除容器，以实际将集群带入所需状态。本章中，我们将探索 `kubelet` 如何在我们的集群中配置及其运作方式。

作为这一探索的一部分，我们将讨论 `kubelet` 如何在依赖于控制平面的同时，也承载着控制平面的功能。最后，我们将探讨 Kubernetes 集群中的节点维护，包括如何为维护关闭节点、可能导致节点无法正常工作的问题、当节点突然变得不可用时集群如何表现，以及节点在失去集群连接时的行为。

### 节点服务

将普通主机转化为 Kubernetes 节点的主要服务是 `kubelet`。由于它对 Kubernetes 集群的关键性，我们将详细探讨它是如何配置的，以及它的行为方式。

**CONTAINERD 和 CRI-O**

本章的示例提供了自动化脚本，用于通过两种容器运行时之一启动集群：`containerd` 和 CRI-O。我们将主要使用 `containerd` 安装，但也会简要介绍配置差异。CRI-O 集群的设置允许你尝试使用一个独立的容器运行时。它还展示了 `kubelet` 隐藏这一差异的事实，因为集群的其他配置不受容器运行时更改的影响。

我们在第六章中设置集群时，已将 `kubelet` 作为软件包安装到所有节点上，之后的各章中，自动化过程也同样为每个章节设置了它。

**注意**

*本书的示例代码仓库在* [`github.com/book-of-kubernetes/examples`](https://github.com/book-of-kubernetes/examples)。*详细设置方法请见“运行示例”部分，第 xx 页。*

`kubelet` 包还包含一个系统服务。我们的操作系统使用 `systemd` 来运行服务，因此我们可以使用 `systemctl` 获取服务信息：

```
root@host01:~# systemctl status kubelet
  kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; ...
    Drop-In: /etc/systemd/system/kubelet.service.d
               10-kubeadm.conf
     Active: active (running) since ...
```

第一次启动 `kubelet` 时，它没有加入集群所需的配置。当我们运行 `kubeadm` 时，它创建了前面输出中显示的文件 *10-kubeadm.conf*。该文件通过设置命令行参数来为集群配置 `kubelet` 服务。

清单 12-1 展示了传递给 `kubelet` 服务的命令行参数。

```
root@host01:~# strings /proc/$(pgrep kubelet)/cmdline
/usr/bin/kubelet
--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf
--kubeconfig=/etc/kubernetes/kubelet.conf
--config=/var/lib/kubelet/config.yaml
--container-runtime=remote
--container-runtime-endpoint=/run/containerd/containerd.sock
--node-ip=192.168.61.11
--pod-infra-container-image=k8s.gcr.io/pause:3.4.1
```

*清单 12-1：Kubelet 命令行*

`pgrep kubelet`嵌入命令输出`kubelet`服务的进程 ID。我们接着使用该 ID 通过*/proc* Linux 虚拟文件系统打印进程的命令行。我们使用`strings`来打印该文件，而不是`cat`，因为每个单独的命令行参数都是以空字符结尾，`strings`会将其转换为良好的多行显示格式。

`kubelet`服务需要三个主要的配置选项组：*集群配置*、*容器运行时配置*和*网络配置*。

#### Kubelet 集群配置

集群配置选项告诉`kubelet`如何与集群通信以及如何进行身份验证。当`kubelet`第一次启动时，它使用清单 12-1 中显示的`bootstrap-kubeconfig`来查找集群，验证服务器证书，并使用我们在第十一章中讨论的引导令牌进行身份验证。这个引导令牌用于提交此新节点的证书签名请求（CSR）。然后，`kubelet`从 API 服务器下载签名的客户端证书，并将其存储在*/etc/kubernetes/kubelet.conf*中，这是由`kubeconfig`选项指定的位置。此*kubelet.conf*文件遵循与配置`kubectl`与 API 服务器通信相同的格式，正如我们在第十一章中看到的那样。在*kubelet.conf*写入后，引导文件会被删除。

在清单 12-1 中指定的*/var/lib/kubelet/config.yaml*文件也包含了重要的配置内容。为了从`kubelet`拉取度量信息，我们需要为其配置自己的服务器证书，而不仅仅是客户端证书，并且需要配置它如何验证自己的客户端。以下是由`kubeadm`创建的配置文件中的相关内容：

```
root@host01:~# cat /var/lib/kubelet/config.yaml
...
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
...
```

`authentication`部分告诉`kubelet`不允许匿名请求，但允许 webhook 承载令牌以及由集群证书颁发机构签名的任何客户端证书。我们为度量服务器安装的 YAML 资源文件包括一个 ServiceAccount，该账户在其部署中使用，因此它会自动注入凭证，供其用来向`kubelet`实例进行身份验证，正如我们在第十一章中看到的那样。

#### Kubelet 容器运行时配置

容器运行时配置选项告诉`kubelet`如何连接到容器运行时，以便`kubelet`能够管理本地机器上的容器。由于`kubelet`期望运行时支持容器运行时接口（CRI）标准，因此只需要几个设置，如清单 12-1 所示。

第一个关键设置是 `container-runtime`，可以设置为 `remote` 或 `docker`。Kubernetes 诞生于 Docker 引擎与 `containerd` 运行时分离之前，因此它对 Docker 有遗留支持，使用 *shim* 来模拟标准的 CRI 接口。因为我们直接使用 `containerd`，而不是通过 Docker shim 或 Docker 引擎，所以我们将其设置为 `remote`。

接下来，我们使用 `container-runtime-endpoint` 设置指定容器运行时的路径。此情况下的值是 */run/containerd/containerd.sock*。`kubelet` 连接到这个 Unix 套接字以发送 CRI 请求并接收状态。

`container-runtime-endpoint` 命令行设置是切换集群在 `containerd` 和 CRI-O 之间所需的唯一差异。此外，当节点初始化时，`kubeadm` 会自动检测到它，因此自动化脚本中的唯一差异是在安装 Kubernetes 之前安装 CRI-O，而不是 `containerd`。如果我们查看 CRI-O 集群中 `kubelet` 的命令行选项，我们会看到命令行选项中只有一个变化：

```
root@host01:~# strings /proc/$(pgrep kubelet)/cmdline
...
--container-runtime-endpoint=/var/run/crio/crio.sock
...
```

剩下的命令行选项与我们的 `containerd` 集群相同。

最后，我们还有一个与容器运行时相关的设置：`pod-infra-container-image`。此设置指定 Pod 基础设施镜像。我们在第二章中以 `pause` 进程的形式看到了这个镜像，它是为我们的容器创建的 Linux 命名空间的所有者。在这种情况下，这个 `pause` 进程将来自容器镜像 `k8s.gcr.io/pause:3.4.1`。

拥有一个单独的容器来管理 Pod 中容器之间共享的命名空间是非常方便的。因为 `pause` 进程实际上什么都不做，它非常可靠，不容易崩溃，所以即使 Pod 中的其他容器意外终止，它也能继续管理这些共享的命名空间。

`pause` 镜像大约有 300KB，如我们在其中一个节点上运行 `crictl` 所见：

```
root@host01:~# crictl images
IMAGE             TAG                 IMAGE ID            SIZE
,,,
k8s.gcr.io/pause  3.4.1               0f8457a4c2eca       301kB
...
```

此外，`pause` 进程几乎不占用 CPU，因此每个 Pod 为每个节点增加一个额外进程对节点的影响很小。

#### Kubelet 网络配置

网络配置帮助 `kubelet` 将其集成到集群中，并将 Pods 集成到整体的集群网络中。正如我们在第八章中看到的，实际的 Pod 网络设置是由网络插件执行的，但 `kubelet` 也有几个重要的角色。

我们的`kubelet`命令行包括一个与网络配置相关的选项：`node-ip`。这是一个可选标志，如果没有提供，`kubelet`将尝试确定它应该使用的 IP 地址与 API 服务器进行通信。然而，直接指定该标志是有用的，因为它可以确保我们的集群在节点有多个网络接口的情况下正常工作（例如本书示例中的 Vagrant 配置，其中使用一个单独的内部网络进行集群通信）。

除了这一行命令行选项外，`kubeadm`还将两个重要的网络设置放入*/var/lib/kubelet/config.yaml*：

```
root@host01:~# cat /var/lib/kubelet/config.yaml
...
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
...
```

这些设置用于将*/etc/resolv.conf*文件提供给所有容器。`clusterDNS`条目提供了该 DNS 服务器的 IP 地址，而`clusterDomain`条目提供了一个默认的搜索域，以便我们区分集群内部的主机名和外部网络上的主机名。

让我们快速查看这些值是如何提供给 Pod 的。我们将从创建一个 Pod 开始：

```
root@host01:~# kubectl apply -f /opt/pod.yaml 
pod/debug created
```

几秒钟后，当 Pod 正在运行时，我们可以获取一个 shell：

```
root@host01:~# kubectl exec -ti debug -- /bin/sh
/ #
```

请注意，*/etc/resolv.conf*是我们容器中单独挂载的文件：

```
/ # mount | grep resolv
/dev/sda1 on /etc/resolv.conf type ext4 ...
```

其内容反映了`kubelet`的配置：

```
/ # cat /etc/resolv.conf 
search default.svc.cluster.local svc.cluster.local cluster.local 
nameserver 10.96.0.10
options ndots:5
```

这个 DNS 配置指向 Kubernetes 集群核心组件的一部分 DNS 服务器，使得我们在第九章中看到的服务查找成为可能。根据你网络中的 DNS 配置，你可能会在`search`列表中看到其他项目，而不仅仅是这里显示的内容。

同时请注意，*/run/secrets/kubernetes.io/serviceaccount*也是我们容器中单独挂载的目录。这个目录包含了我们在第十一章中看到的 ServiceAccount 信息，用于在容器内与 API 服务器进行身份验证：

```
/ # mount | grep run
tmpfs on /run/secrets/kubernetes.io/serviceaccount type tmpfs (ro,relatime)
```

在这种情况下，挂载的目录是`tmpfs`类型，因为`kubelet`已经创建了一个内存文件系统来存储认证信息。

让我们通过退出 shell 会话并删除 Pod 来结束操作（我们不再需要它）：

```
/ # exit
root@host01:~# kubectl delete pod debug
```

这次清理将使得接下来的 Pod 列表更加清晰，因为我们将查看当一个节点停止工作时集群的反应。在此之前，我们还有一个关键的谜题需要解决：`kubelet`如何同时托管控制平面并依赖于它。

### 静态 Pods

创建我们的集群时，我们遇到了一种“先鸡还是先蛋”的问题。我们希望`kubelet`能够管理控制平面组件作为 Pods，因为这样可以更容易地监控、维护和更新控制平面组件。然而，`kubelet`依赖于控制平面来决定运行哪些容器。解决方案是让`kubelet`支持静态 Pod 定义，它从文件系统中拉取并在建立控制平面连接之前自动运行。

这个静态 Pod 配置在*/var/lib/kubelet/config.yaml*中处理：

```
root@host01:~# cat /var/lib/kubelet/config.yaml 
...
staticPodPath: /etc/kubernetes/manifests
...
```

如果我们查看 */etc/kubernetes/manifests*，我们会看到多个 YAML 文件。这些文件是由 `kubeadm` 放置的，定义了运行此节点控制平面组件所必需的 Pods：

```
root@host01:~# ls -1 /etc/kubernetes/manifests
etcd.yaml
kube-apiserver.yaml
kube-controller-manager.yaml
kube-scheduler.yaml
```

正如预期的那样，我们看到每个我们在第十一章讨论过的三个关键控制平面服务都有一个 YAML 文件。我们还看到一个 `etcd` 的 Pod 定义，`etcd` 是存储集群状态并帮助选举领导者以确保我们集群高可用的组件。我们将在第十六章中更详细地了解 `etcd`。

这些文件中的每一个都包含一个 Pod 定义，类似于我们已经看到的：

```
root@host01:~# cat /etc/kubernetes/manifests/kube-apiserver.yaml 
apiVersion: v1
kind: Pod
metadata:
...
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
...
```

`kubelet` 服务持续监控此目录中的任何变化，并相应地更新对应的静态 Pod，这使得 `kubeadm` 能够在不中断的情况下，按滚动方式升级集群的控制平面。

集群附加组件如 Calico 和 Longhorn 也可以使用这个目录运行，但它们使用 DaemonSet 来确保集群在每个节点上运行一个 Pod。这是有道理的，因为 DaemonSet 可以一次性管理整个集群，确保所有节点之间的配置一致。

这个静态 Pod 目录在我们的三个控制平面节点 *host01* 到 *host03* 与我们的“普通”节点 *host04* 上有所不同。为了将 *host04* 设为普通节点，`kubeadm` 会在 */etc/kubernetes/manifests* 中省略控制平面的静态 Pod 文件：

```
root@host04:~# ls -1 /etc/kubernetes/manifests
root@host04:~#
```

请注意，这个命令是在 *host04* 上执行的，它是我们集群中唯一的普通节点。

### 节点维护

控制平面中的控制器管理器组件持续监控节点，以确保它们仍然连接且健康。`kubelet` 服务负责报告节点信息，包括节点内存使用、磁盘使用和与底层容器运行时的连接。如果一个节点变得不健康，控制平面将把 Pods 移动到其他节点，以维持部署的预期规模，并且在节点恢复健康之前，不会向该节点调度任何新的 Pods。

#### 节点排空与隔离

如果我们知道需要对某个节点进行维护，比如重启，我们可以告诉集群将 Pods 从该节点迁移，并将该节点标记为不可调度。我们通过使用 `kubectl drain` 命令来实现这一点。

举个例子，我们创建一个有八个 Pods 的 Deployment，这样每个节点很可能会获得一个 Pod：

```
root@host01:~# kubectl apply -f /opt/deploy.yaml 
deployment.apps/debug created
```

如果我们给足够的启动时间，我们可以看到 Pods 被分配到各个节点上：

```
root@host01:~# kubectl get pods -o wide
NAME                     READY   STATUS    ... NODE   ...
debug-8677494fdd-7znxn   1/1     Running   ... host02 ...  
debug-8677494fdd-9dgvd   1/1     Running   ... host03 ...  
debug-8677494fdd-hv6mt   1/1     Running   ... host04 ...  
debug-8677494fdd-ntqjp   1/1     Running   ... host02 ...  
debug-8677494fdd-pfw5n   1/1     Running   ... host03 ...  
debug-8677494fdd-qbhmn   1/1     Running   ... host02 ...  
debug-8677494fdd-qp9zv   1/1     Running   ... host03 ...  
debug-8677494fdd-xt8dm   1/1     Running   ... host03 ...
```

为了最小化我们的测试集群大小，我们的普通节点 `host04` 资源较少，因此在这个例子中它只获得一个 Pod。但这足以看到当我们关闭节点时会发生什么。这个过程有一定的随机性，所以如果你没有看到任何 Pods 分配到 `host04`，你可以删除 Deployment 重新尝试，或者像我们在下一个例子中那样缩小后再放大。

要关闭节点，我们使用 `kubectl drain` 命令：

```
root@host01:~# kubectl drain --ignore-daemonsets host04
node/host04 cordoned
WARNING: ignoring DaemonSet-managed Pods: ...
...
pod/debug-8677494fdd-hv6mt evicted
node/host04 evicted
```

我们需要提供`--ignore-daemonsets`选项，因为我们所有的节点都运行着 Calico 和 Longhorn DaemonSets，当然，这些 Pod 无法迁移到其他节点。

驱逐过程会花费一些时间。完成后，我们可以看到部署在另一个节点上创建了一个 Pod，这样我们的 Pod 数量保持在八个：

```
root@host01:~# kubectl get pods -o wide
NAME                     READY   STATUS    ... NODE     ...
debug-8677494fdd-7znxn   1/1     Running   ... host02   ...
debug-8677494fdd-9dgvd   1/1     Running   ... host03   ...
debug-8677494fdd-ntqjp   1/1     Running   ... host02   ...
debug-8677494fdd-pfw5n   1/1     Running   ... host03   ...
debug-8677494fdd-qbhmn   1/1     Running   ... host02   ...
debug-8677494fdd-qfnml   1/1     Running   ... host01   ...
debug-8677494fdd-qp9zv   1/1     Running   ... host03   ...
debug-8677494fdd-xt8dm   1/1     Running   ... host03   ...
```

此外，节点已被*隔离*，因此将不会再有 Pod 被调度到该节点上：

```
root@host01:~# kubectl get nodes
NAME     STATUS                     ROLES        ...
host01   Ready                      control-plane...
host02   Ready                      control-plane...
host03   Ready                      control-plane...
host04   Ready,SchedulingDisabled   <none>       ...
```

此时，停止`kubelet`或容器运行时、重启节点，甚至完全从 Kubernetes 中删除该节点都是安全的：

```
root@host01:~# kubectl delete node host04
node "host04" deleted
```

该删除操作会从集群的存储中移除节点信息，但由于该节点仍然拥有有效的客户端证书和所有配置，简单地重启`host04`上的`kubelet`服务将把它重新加入集群。首先让我们重启`kubelet`：

```
root@host04:~# systemctl restart kubelet
```

请确保在`host04`上执行此操作。接下来，在`host01`上，如果我们等待`host04`上的`kubelet`完成上次运行的清理并重新初始化，我们会看到它重新出现在节点列表中：

```
root@host01:~# kubectl get nodes
NAME     STATUS   ROLES        ...
host01   Ready    control-plane...
host02   Ready    control-plane...
host03   Ready    control-plane...
host04   Ready    <none>       ...
```

注意，隔离已经被移除，`host04`不再显示包含`SchedulingDisabled`的状态。这是移除隔离的一种方式，另一种方式是直接使用`kubectl uncordon`命令。

#### 不健康节点

如果节点因内存不足或磁盘空间等资源限制变得不健康，Kubernetes 还会自动将 Pod 迁移到其他节点。让我们模拟`host04`上内存不足的情况，以便观察这一过程。

首先，我们需要重置`debug`部署的规模，以确保新的 Pod 被分配到`host04`上：

```
root@host01:~# kubectl scale deployment debug --replicas=1
deployment.apps/debug scaled
root@host01:~# kubectl scale deployment debug --replicas=12
deployment.apps/debug scaled
```

我们首先将部署的规模缩减到最小，然后再将其扩大。这样，我们有更多机会将至少一个 Pod 调度到`host04`上。一旦 Pod 有机会稳定下来，我们会看到`host04`上再次有 Pod：

```
root@host01:~# kubectl get pods -o wide
NAME                     READY   STATUS    ... NODE     ...
...
debug-8677494fdd-j7cth   1/1     Running   ... host04   ...
debug-8677494fdd-jlj4v   1/1     Running   ... host04   ...
...
```

我们可以使用`kubectl top`来检查当前节点的统计信息：

```
root@host01:~# kubectl top nodes
NAME     CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
host01   503m         25%    1239Mi          65%       
host02   518m         25%    1346Mi          71%       
host03   534m         26%    1382Mi          73%       
host04   288m         14%    542Mi           29%
```

`host04`总共有 2GB 内存，目前已使用超过 500MiB。默认情况下，当剩余内存少于 100MiB 时，`kubelet`会驱逐 Pod。我们可以尝试使用节点上的内存直到低于这个默认阈值，但这很冒险，因为大量使用内存可能会导致节点行为不正常。相反，我们可以更新驱逐限制。为此，我们将向*/var/lib/kubelet/config.yaml*添加几行，然后重启`kubelet`。

这是我们将添加到`kubelet`配置文件中的额外配置：

*node-evict.yaml*

```
evictionHard:
  memory.available: "1900Mi"
```

这会告诉`kubelet`，如果可用内存少于 1,900MiB，它将开始驱逐 Pod。在我们的示例集群中的节点上，这将立即发生。让我们应用这一更改：

```
root@host04:~# cat /opt/node-evict.yaml >> /var/lib/kubelet/config.yaml
root@host04:~# systemctl restart kubelet
```

请确保在`host04`上执行这些命令。第一条命令会向`kubelet`配置文件中添加额外的行。第二条命令会重启`kubelet`，以便它加载这个更改。

如果我们检查`host04`的节点状态，它似乎仍然是就绪的：

```
root@host01:~# kubectl get nodes
NAME     STATUS   ROLES        ...
host01   Ready    control-plane...
host02   Ready    control-plane...
host03   Ready    control-plane...
host04   Ready    <none>       ...
```

然而，节点的事件日志清楚地显示了发生了什么：

```
root@host01:~# kubectl describe node host04
Name:               host04
...
  Normal   NodeHasInsufficientMemory  6m31s                ...
  Warning  EvictionThresholdMet       7s (x14 over 6m39s)  ...
```

节点开始驱逐 Pod，并且集群会根据需要在其他节点上自动创建新的 Pod 以保持所需的规模：

```
root@host01:~# kubectl get pods -o wide
NAME                     READY   STATUS        ... NODE     ...
debug-8677494fdd-4274k   1/1     Running       ... host01   ...
debug-8677494fdd-4pnzb   1/1     Running       ... host01   ...
debug-8677494fdd-5nw6n   1/1     Running       ... host01   ...
debug-8677494fdd-7kbp8   1/1     Running       ... host03   ...
debug-8677494fdd-dsnp5   1/1     Running       ... host03   ...
debug-8677494fdd-hgdbc   1/1     Running       ... host01   ...
debug-8677494fdd-j7cth   1/1     Running       ... host04   ...
debug-8677494fdd-jlj4v   0/1     OutOfmemory   ... host04   ...
debug-8677494fdd-lft7h   1/1     Running       ... host01   ...
debug-8677494fdd-mnk6r   1/1     Running       ... host01   ...
debug-8677494fdd-pc8q8   1/1     Running       ... host01   ...
debug-8677494fdd-sr2kw   0/1     OutOfmemory   ... host04   ...
debug-8677494fdd-tgpb2   1/1     Running       ... host03   ...
debug-8677494fdd-vnjks   0/1     OutOfmemory   ... host04   ...
debug-8677494fdd-xn8t8   1/1     Running       ... host02   ...
```

分配给`host04`的 Pod 显示为`OutOfMemory`，它们已被其他节点上的 Pod 替换。这些 Pod 在节点上被停止，但不像前面我们排空节点的情况，这些 Pod 不会自动终止。即使节点从低内存状态恢复，这些 Pod 仍将显示在 Pod 列表中，处于`OutOfMemory`状态，直到重新启动`kubelet`。

#### 节点不可达

我们还有一个案例要讨论。在我们之前的两个示例中，`kubelet`可以与控制平面通信以更新其状态，使控制平面能够相应地采取行动。但是如果出现网络问题或突然断电，并且节点失去与集群的连接而无法报告正在关闭，会发生什么情况呢？在这种情况下，集群将记录节点状态为未知，并在超时后开始将 Pod 转移到其他节点。

让我们模拟一下这种情况。我们将从恢复`host04`到正常工作状态开始：

```
root@host04:~# sed -i '/^evictionHard/,+2d' /var/lib/kubelet/config.yaml 
root@host04:~# systemctl restart kubelet
```

确保在`host04`上运行这些命令。第一个命令删除我们添加到`kubelet`配置中的两行，而第二个命令重新启动`kubelet`以应用更改。现在我们可以再次调整我们的部署，以便重新分配：

```
root@host01:~# kubectl scale deployment debug --replicas=1
root@host01:~# kubectl scale deployment debug --replicas=12
```

与之前一样，在运行这些命令后，请等待几分钟以使 Pod 稳定下来。然后，使用`kubectl get pods -o wide`来验证至少有一个 Pod 分配到了`host04`。

现在我们准备强制断开`host04`与集群的连接。我们将通过添加防火墙规则来执行此操作：

```
root@host04:~# iptables -I INPUT -s 192.168.61.10 -j DROP
root@host04:~# iptables -I OUTPUT -d 192.168.61.10 -j DROP
```

确保在`host04`上运行此命令。第一个命令告诉防火墙丢弃所有来自 IP 地址`192.168.61.10`的流量，这是所有三个控制平面节点共享的高可用 IP 地址。第二个命令告诉防火墙丢弃所有发送到同一 IP 地址的流量。

大约一分钟后，`host04`将显示为`NotReady`状态：

```
root@host01:~# kubectl get nodes
NAME     STATUS     ROLES        ...
host01   Ready      control-plane...
host02   Ready      control-plane...
host03   Ready      control-plane...
host04   NotReady   <none>       ...
```

如果等待几分钟，`host04`上的 Pod 将显示为`Terminating`，因为集群放弃了这些 Pod 并将它们转移到其他节点：

```
root@host01:~# kubectl get pods -o wide
NAME                     READY   STATUS        ... NODE     ...
debug-8677494fdd-2wrn2   1/1     Running       ... host01   ...
debug-8677494fdd-4lz48   1/1     Running       ... host02   ...
debug-8677494fdd-78874   1/1     Running       ... host01   ...
debug-8677494fdd-7f8fw   1/1     Running       ... host01   ...
debug-8677494fdd-9vb5m   1/1     Running       ... host03   ...
debug-8677494fdd-b7vj6   1/1     Running       ... host03   ...
debug-8677494fdd-c2c4v   1/1     Terminating   ... host04   ...
debug-8677494fdd-c8tzv   1/1     Running       ... host03   ...
debug-8677494fdd-d2r6b   1/1     Terminating   ... host04   ...
debug-8677494fdd-d5t6b   1/1     Running       ... host01   ...
debug-8677494fdd-j7cth   1/1     Terminating   ... host04   ...
debug-8677494fdd-jjfsl   1/1     Terminating   ... host04   ...
debug-8677494fdd-nqb8z   1/1     Running       ... host03   ...
debug-8677494fdd-sskd5   1/1     Running       ... host02   ...
debug-8677494fdd-wz6c6   1/1     Terminating   ... host04   ...
debug-8677494fdd-x5b4w   1/1     Running       ... host02   ...
debug-8677494fdd-zfbml   1/1     Running       ... host01   ...
```

然而，因为`host04`上的`kubelet`无法连接到控制平面，它不知道它应该关闭其 Pod。如果我们检查在`host04`上运行哪些容器，我们仍然会看到多个容器在运行：

```
root@host04:~# crictl ps
CONTAINER           IMAGE          ...  STATE      NAME  ...
2129a1cb00607       16ea53ea7c652  ...  Running    debug ...
cfd7fd6142321       16ea53ea7c652  ...  Running    debug ...
0289ffa5c816d       16ea53ea7c652  ...  Running    debug ...
fb2d297d11efb       16ea53ea7c652  ...  Running    debug ...
...
```

不仅 Pods 仍在运行，而且由于我们切断连接的方式，它们仍然能够与集群的其余部分进行通信。这一点非常重要。Kubernetes 会尽力运行请求的实例数量并响应错误，但它只能基于它所拥有的信息来执行此操作。在这种情况下，由于`host04`上的`kubelet`无法与控制平面通信，Kubernetes 无法知道 Pods 仍然在运行。在为像 Kubernetes 集群这样的分布式系统构建应用时，你应该认识到某些类型的错误可能会导致意想不到的结果，比如部分网络连接或与指定实例数量不同的情况。在包含滚动更新的更高级的应用架构中，这甚至可能导致旧版本的应用组件意外运行。确保构建能够应对这些意外行为的具有弹性的应用。

### 最后的思考

最终，要拥有一个 Kubernetes 集群，我们需要能够运行容器的节点，这意味着需要连接到控制平面和容器运行时的`kubelet`实例。在本章中，我们检查了如何配置`kubelet`以及当节点离开或加入集群时，集群的行为——无论是故意的还是由于故障。

本章的一个关键主题是 Kubernetes 如何在节点出现问题时仍然保持指定数量的 Pods 运行。在下一章中，我们将看到如何将监控扩展到容器内部及其进程，确保进程按预期运行。我们将看到如何指定探针以允许 Kubernetes 监控容器，以及当容器不健康时集群如何响应。
