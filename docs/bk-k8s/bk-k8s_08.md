# 第六章：为什么 KUBERNETES 很重要

![image](img/common01.jpg)

容器使我们能够改变打包和部署应用组件的方式，但在集群中编排容器才能真正发挥容器化微服务架构的优势。正如第一章中所描述，现代应用架构的主要优点是可扩展性、可靠性和弹性，而这三大优点都需要像 Kubernetes 这样的容器编排环境，才能在多个服务器和网络中运行许多容器化的应用组件实例。

在本章中，我们将首先讨论在集群中跨多个服务器运行容器时出现的一些跨领域关注点。然后，我们将描述为解决这些关注点而设计的 Kubernetes 核心概念。在介绍完成后，我们将重点介绍本章的主要内容，即实际安装 Kubernetes 集群，包括重要的附加组件，如网络和存储。

### 在集群中运行容器

将我们的应用组件分布在多个服务器上的需求，对于现代应用架构来说并不新鲜。为了构建可扩展且可靠的应用，我们始终需要利用多个服务器来处理应用负载，并避免单点故障。我们现在将这些组件运行在容器中，并没有改变我们对多台服务器的需求；我们最终仍然使用 CPU，并且依赖硬件。

与此同时，容器编排环境带来了可能在其他类型的应用基础设施中不存在的挑战。当容器是我们构建系统时最小的单个模块时，我们最终得到的应用组件更加自包含，从我们的基础设施角度来看更具“封闭性”。这意味着，与静态的应用架构不同，在静态架构中我们预先选择将哪些应用组件分配到特定服务器上，而使用 Kubernetes 时，我们尽量使得任何容器都能在任何地方运行。

#### 跨领域关注点

在任何地方运行任何容器的能力最大化了我们的灵活性，但也增加了 Kubernetes 本身的复杂性。Kubernetes 无法提前知道将要求运行哪些容器，且容器工作负载随着新应用的部署或应用负载变化而不断变化。为了应对这一挑战，Kubernetes 需要考虑以下设计参数，这些参数适用于所有容器编排软件，无论运行什么容器：

**动态调度** 新的容器必须分配到服务器上，并且由于配置变化或故障，分配可能会发生变化。

**分布式状态** 整个集群必须保持关于容器运行状态和位置的信息，即使在硬件或网络故障时也要保持这一信息。

**多租户** 应该能够在单一集群中运行多个应用程序，同时实现安全性和可靠性的隔离。

**硬件隔离** 集群必须在云环境中运行，并且可以运行在各种类型的常规服务器上，实现容器与这些环境之间的隔离。

用来描述这些设计参数的最佳术语是 *横切关注点*，因为它们适用于我们可能需要部署的任何类型的容器化软件，甚至是 Kubernetes 基础架构本身。这些参数与我们在第一章中看到的容器编排需求一起工作，并最终推动 Kubernetes 架构和关键设计决策。

#### Kubernetes 概念

为了解决这些横切关注点，Kubernetes 架构允许任何东西在任何时候进出。这不仅包括部署到 Kubernetes 的容器化应用程序，还包括 Kubernetes 本身的基本软件组件，甚至包括底层硬件，如服务器、网络连接和存储。

##### 分离的控制平面

显然，要让 Kubernetes 成为容器编排环境，它需要能够运行容器。这个能力由一组叫做*节点*的工作机器提供。每个节点运行一个与底层容器运行时接口的 *kubelet* 服务，用于启动和监控容器。

Kubernetes 还有一组核心软件组件，负责管理工作节点及其容器，但这些软件组件与工作节点是分开部署的。这些核心 Kubernetes 软件组件统称为*控制平面*。由于控制平面与工作节点分离，工作节点可以运行控制平面，从而让 Kubernetes 核心软件组件受益于容器化。分离的控制平面也意味着 Kubernetes 本身具有微服务架构，允许对每个 Kubernetes 集群进行定制。例如，一个控制平面组件——*云控制器管理器*，仅在将 Kubernetes 部署到云提供商时使用，并且它会根据所使用的云提供商进行定制。这样的设计为应用容器和 Kubernetes 控制平面的其他部分提供了硬件隔离，同时仍然可以利用每个云提供商的特定功能。

##### 声明式 API

Kubernetes 控制平面的一个关键组件是*API 服务器*。API 服务器为集群控制和监视提供接口，其他集群用户和控制平面组件使用它。在定义 API 时，Kubernetes 可以选择*命令式*风格，其中每个 API 端点都是诸如“运行容器”或“分配存储”的命令。相反，API 是*声明式*的，提供诸如*创建*、*修补*、*获取*和*删除*的端点。这些命令的效果是从集群配置中创建、读取、更新和删除*资源*，每个资源的具体配置告诉 Kubernetes 我们希望集群执行什么操作。

这种声明式 API 对满足动态调度和分布式状态的横切关注点至关重要。因为声明式 API 只报告或更新集群配置，因此很容易对可能导致命令丢失的服务器或网络故障做出反应。考虑一个例子，即使在发出`apply`命令以更改集群配置后，API 服务器连接丢失。当连接恢复时，客户端只需查询集群配置，并确定命令是否成功接收。或者更简单地，客户端可以再次发出相同的`apply`命令，因为只要集群配置最终符合期望，Kubernetes 将尝试对实际集群进行“正确的操作”。这个核心原则被称为*幂等性*，意味着可以安全地多次发出相同的命令，因为它最多只会应用一次。

##### 自我修复

基于声明式 API，Kubernetes 被设计为*自我修复*。这意味着控制平面组件不断监视集群配置和实际集群状态，并尝试使它们保持一致。集群配置中的每个资源都有一个相关的状态和事件日志，反映了配置如何实际导致集群状态的变化。

配置和状态分离使得 Kubernetes 非常具有弹性。例如，表示容器的资源如果已经被调度并且实际在运行，则可能处于`Running`状态。如果 Kubernetes 控制平面与运行容器的服务器失去连接，它可以立即将状态设置为`Unknown`，然后努力重新建立连接或将节点视为失败并重新调度容器。

同时，使用声明式 API 和自愈方法具有重要的意义。由于 Kubernetes API 是声明式的，命令的“成功”响应仅意味着集群配置已更新。这并不意味着集群的实际状态已更新，因为可能需要一些时间才能实现请求的状态，或者可能存在一些问题，导致集群无法实现该状态。因此，我们不能仅仅因为创建了适当的资源，就假设集群正在运行我们期望的容器。相反，我们必须监视资源的状态，并查看事件日志，以诊断 Kubernetes 控制平面在使实际集群状态与我们指定的配置匹配时可能遇到的任何问题。

### 集群部署

在掌握了一些 Kubernetes 核心概念后，我们将使用 `kubeadm` Kubernetes 管理工具，在多台虚拟机上部署一个高度可用的 Kubernetes 集群。

**选择 Kubernetes 发行版**

与我们在第一章中使用特定的 Kubernetes 发行版不同，我们将使用通用的上游代码库部署一个“原生”的 Kubernetes 集群。这种方法给我们提供了最好的机会，能够跟随集群的部署过程，并将在接下来的几章中更容易深入探索集群。然而，当你准备好部署自己的 Kubernetes 集群时，特别是在生产环境中，考虑使用一个预构建的 Kubernetes 发行版，以便于管理并具备内建的安全性。云原生计算基金会（CNCF）发布了一套符合性测试，你可以用来确保你选择的 Kubernetes 发行版符合 Kubernetes 规范。

我们的 Kubernetes 集群将分布在四台虚拟机上，分别标记为 `host01` 到 `host04`。其中三台虚拟机，`host01` 到 `host03`，将运行控制平面组件，而第四台将仅作为工作节点。我们将使用三台控制平面节点，因为这是运行高度可用集群所需的最小数量。Kubernetes 使用投票机制提供故障转移，至少需要三台控制平面节点；这样，在网络故障发生时，集群能够检测到应该继续运行的节点。此外，为了使集群尽可能小，以便于我们在接下来的示例中使用，我们将配置 Kubernetes 在控制平面节点上运行常规容器，尽管我们通常会避免在生产集群中这么做。

**注意**

*本书的示例代码库位于* [`github.com/book-of-kubernetes/examples`](https://github.com/book-of-kubernetes/examples)。*有关设置的详细信息，请参阅 第 xx 页的“运行示例”部分。*

从本章的说明开始，确保四台虚拟机都启动并运行，无论是在 Vagrant 中还是在 AWS 中。自动化配置将为所有四台机器设置`containerd`和`crictl`，因此我们无需手动配置。自动化配置脚本还将设置`kube-vip`或 AWS 网络负载均衡器，以提供所需的高可用性功能，如下文所述。

**注意**

*你可以使用本章示例提供的额外*provisioning*脚本自动安装 Kubernetes。有关说明，请参阅本章的 README 文件。*

你需要在每个虚拟机上运行命令，因此你可能希望为每个虚拟机打开一个终端标签。但是，第一系列命令需要在所有主机上运行，因此自动化脚本会在`host01`上设置一个名为`k8s-all`的命令来完成这项工作。你可以在*/usr/local/bin/k8s-all*中查看这个脚本的内容，或者查看本示例中*setup*目录下的*k8s* Ansible 角色。

#### 前提软件包

第一步是确保启用`br_netfilter`内核模块并设置为开机加载。Kubernetes 使用 Linux 防火墙的高级功能来处理跨集群的网络通信，因此我们需要这个模块。运行这两个命令：

```
root@host01:~# k8s-all modprobe br_netfilter
...
root@host01:~# k8s-all "echo 'br_netfilter' > /etc/modules-load.d/k8s.conf"
```

第一个命令确保模块已安装在当前运行的内核中，第二个命令将其添加到开机加载模块列表中。第二个命令中的稍微奇怪的引号确保在远程主机上发生 shell 重定向。

接下来，在清单 6-1 中，我们将设置一些 Linux 内核参数，以启用所需的高级网络功能，这些功能也用于通过`sysctl`命令跨集群进行网络通信：

```
root@host01:~# k8s-all sysctl -w net.ipv4.ip_forward=1 \
  net.bridge.bridge-nf-call-ip6tables=1 \
  net.bridge.bridge-nf-call-iptables=1
```

*清单 6-1：内核设置*

该命令启用以下 Linux 内核网络功能：

net.ipv4.ip_forward 将数据包从一个网络接口转发到另一个网络接口（例如，从容器的网络命名空间内的接口到主机网络）。

net.bridge.bridge-nf-call-ip6tables 通过`iptables`防火墙运行 IPv6 桥接流量。

net.bridge.bridge-nf-call-iptables 通过`iptables`防火墙运行 IPv4 桥接流量。

最后两项的需求将在第九章中变得更加明确，当我们讨论 Kubernetes 如何为服务提供网络时。

在清单 6-1 中的这些`sysctl`更改在重启后不会保持。自动化脚本会处理这些更改的持久化，因此如果你重启虚拟机，可以运行`extra` provisioning 脚本，或者重新运行这些命令。

我们现在已经完成了配置 Linux 内核以支持 Kubernetes 部署，准备好进行实际安装。首先，我们需要安装一些前提软件包：

```
root@host01:~# k8s-all apt install -y apt-transport-https \
  open-iscsi nfs-common
```

`apt-transport-https` 包确保 `apt` 能通过安全 HTTP 协议连接到仓库。其他两个软件包是我们在集群启动并运行后将安装的集群附加组件所需的。

#### Kubernetes 软件包

现在，我们可以添加 Kubernetes 仓库来安装将设置我们集群的 `kubeadm` 工具。首先，添加用于检查软件包签名的 GPG 密钥：

```
root@host01:~# k8s-all "curl -fsSL \
  https://packages.cloud.google.com/apt/doc/apt-key.gpg | \
  gpg --dearmor -o /usr/share/keyrings/google-cloud-keyring.gpg"
```

这条命令使用 `curl` 下载 GPG 密钥。然后它使用 `gpg` 重新格式化密钥，并将结果写入到 */usr/share/keyrings*。命令行标志 `fsSL` 将 `curl` 设置为一种更适合链式命令的模式，包括避免不必要的输出、跟随服务器重定向，并在出现问题时终止执行。

接下来，我们添加仓库配置：

```
root@host01:~# k8s-all "echo 'deb [arch=amd64' \
  'signed-by=/usr/share/keyrings/google-cloud-keyring.gpg]' \
  'https://apt.kubernetes.io/ kubernetes-xenial main' > \
  /etc/apt/sources.list.d/kubernetes.list"
```

和之前一样，引号非常重要，确保命令可以通过 SSH 正确传递到集群中的所有其他主机。命令将 `kubernetes-xenial` 配置为发行版；这个发行版用于任何版本的 Ubuntu，从较旧的 Ubuntu Xenial 开始。

在创建完这个新仓库之后，我们需要在所有主机上运行 `apt update` 来下载软件包列表：

```
root@host01:~# k8s-all apt update
...
```

现在我们可以使用 `apt` 安装所需的软件包：

```
root@host01:~# source /opt/k8sver
root@host01:~# k8s-all apt install -y kubelet=$K8SV kubeadm=$K8SV kubectl=$K8SV
```

`source` 命令加载一个带有变量的文件，用于安装特定版本的 Kubernetes。这个文件由自动化脚本创建，确保我们在所有章节中使用一致的 Kubernetes 版本。你可以更新自动化脚本来选择要安装的 Kubernetes 版本。

`apt` 命令安装以下三个软件包以及一些依赖项：

kubelet 服务用于所有工作节点，它与容器引擎接口，按控制平面的调度运行容器。

kubeadm 是我们用来安装 Kubernetes 并维护集群的管理工具。

kubectl 是我们用来检查 Kubernetes 集群并创建、删除资源的命令行客户端。

`kubelet` 包会立即启动其服务，但由于我们还没有安装控制平面，服务一开始会处于失败状态：

```
root@host01:~# systemctl status kubelet
  kubelet.service - kubelet: The Kubernetes Node Agent
...
   Main PID: 75368 (code=exited, status=1/FAILURE)
```

我们需要控制刚刚安装的软件包的版本，因为我们希望将集群的所有组件一起升级。为了防止不小心更新这些软件包，我们将把它们保持在当前版本：

```
root@host01:~# k8s-all apt-mark hold kubelet kubeadm kubectl
```

这条命令防止标准的 `apt full-upgrade` 命令更新这些软件包。相反，如果我们升级集群，我们需要通过使用 `apt install` 指定我们想要的确切版本。

#### 集群初始化

下一条命令 `kubeadm init` 初始化控制平面，并为所有节点提供 `kubelet` 工作节点服务配置。我们将在集群中的一个节点上运行 `kubeadm init`，然后在其他节点上使用 `kubeadm join` 将它们加入现有的集群。

要运行`kubeadm init`，我们首先创建一个 YAML 配置文件。这种方式有几个优点。它大大减少了我们需要记住的命令行标志数量，而且它让我们可以将集群配置保存在一个版本库中，从而对集群配置进行控制。然后，我们可以更新 YAML 文件并重新运行`kubeadm`来更改集群配置。

本章的自动化脚本已经在*/etc/kubernetes*中填充了一个 YAML 配置文件，所以它已准备好使用。以下是该文件的内容：

*kubeadm-init.yaml*

```
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: InitConfiguration
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: 1d8fb1.2875d52d62a3282d
  ttl: 2h0m0s
  usages:
  - signing
  - authentication
nodeRegistration:
  kubeletExtraArgs:
    node-ip: 192.168.61.11
 taints: []
localAPIEndpoint:
  advertiseAddress: 192.168.61.11
certificateKey: "5a7e07816958efb97635e9a66256adb1"
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
kubernetesVersion: 1.21.4
apiServer:
  extraArgs:
    service-node-port-range: 80-32767
networking:
  podSubnet: "172.31.0.0/16"
controlPlaneEndpoint: "192.168.61.10:6443"
---
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
serverTLSBootstrap: true
```

这个 YAML 文件有三个文档，通过破折号（`---`）分隔。第一个文档是专门用于初始化集群的，第二个文档包含更通用的配置，第三个文档用于提供跨所有节点的`kubelet`设置。我们来看一下每个配置项的用途：

apiVersion / kind 告诉 Kubernetes 每个 YAML 文档的用途，以便它能够验证内容。

bootstrapTokens 配置一个密钥，供其他节点用来加入集群。`token`应该在生产集群中保密。它会在两个小时后自动过期，所以如果我们以后想加入更多节点，需要重新生成一个。

nodeRegistration 配置项，用于传递给在`host01`上运行的`kubelet`服务。`node-ip`字段确保`kubelet`将正确的 IP 地址注册到 API 服务器，以便 API 服务器能够与之通信。`taints`字段确保常规容器可以被调度到控制平面节点上。

localAPIEndpoint API 服务器应该使用的本地 IP 地址。我们的虚拟机有多个 IP 地址，我们希望 API 服务器监听正确的网络。

certificateKey 配置一个密钥，供其他节点用来获取 API 服务器的证书。这个密钥是必须的，以便在我们高可用集群中的所有 API 服务器实例都可以使用相同的证书。在生产集群中要保密。

networking 集群中的所有容器都会从`podSubnet`中获取一个 IP 地址，无论它们运行在哪个主机上。稍后，我们将安装一个网络驱动程序，确保集群中所有主机上的容器能够互相通信。

controlPlaneEndpoint API 服务器的外部地址。对于高可用集群，这个 IP 地址需要能够访问到任何 API 服务器实例，而不仅仅是第一个实例。

serverTLSBootstrap 指示`kubelet`使用控制器管理器的证书授权机构来请求服务器证书。

`apiVersion` 和 `kind` 字段将在每个 Kubernetes YAML 文件中出现。`apiVersion` 字段定义了一组相关的 Kubernetes 资源，包括版本号。然后，`kind` 字段选择该组中的具体资源类型。这不仅允许 Kubernetes 项目和其他供应商随着时间的推移添加新的资源组，还允许在保持向后兼容的同时更新现有资源的规范。

**高可用集群**

`controlPlaneEndpoint` 字段用于配置高可用集群的最重要要求：一个可以访问所有 API 服务器的 IP 地址。我们需要在初始化集群时立即设置此 IP 地址，因为它用于生成客户端验证 API 服务器身份的证书。提供集群范围的 IP 地址的最佳方式取决于集群运行的位置；例如，在云环境中，使用提供商内建的能力（如 Amazon Web Services 中的弹性负载均衡器（ELB）或 Azure 负载均衡器）是最好的选择。

由于两种不同环境的特性，本书中的示例在使用 Vagrant 运行时使用 `kube-vip`，在使用 Amazon Web Services 运行时使用 ELB。示例文档中的顶层 *README.md* 文件包含更多细节。安装和配置会自动完成，因此无需进行其他配置。我们可以直接使用 `192.168.61.10:6443`，并期望流量能够到达运行在 `host01` 至 `host03` 上的任何 API 服务器实例。

因为我们已经准备好集群配置文件（YAML 文件），所以初始化集群的 `kubeadm init` 命令非常简单。我们只需要在 `host01` 上运行此命令：

```
root@host01:~# /usr/bin/kubeadm init \
  --config /etc/kubernetes/kubeadm-init.yaml --upload-certs
```

`--config` 选项指向我们之前查看过的 YAML 配置文件（*kubeadm-init.yaml*），而 `--upload-certs` 选项告诉 `kubeadm` 应该将 API 服务器的证书上传到集群的分布式存储中。其他控制平面节点随后可以在加入集群时下载这些证书，从而使所有 API 服务器实例使用相同的证书，这样客户端就会信任它们。这些证书是使用我们提供的 `certificateKey` 进行加密的，这意味着其他节点需要此密钥才能解密它们。

`kubeadm init` 命令在 `host01` 上初始化控制平面的组件。这些组件以容器的形式运行，并由 `kubelet` 服务进行管理，这使得它们容易升级。几个容器镜像将被下载，因此根据虚拟机的速度和网络连接的情况，此命令可能需要一段时间。

#### 将节点加入集群

`kubeadm init`命令会输出一个`kubeadm join`命令，我们可以用它将其他节点加入集群。然而，自动化脚本已经将配置文件预先放置到每个其他节点，确保它们以正确类型的节点加入。服务器`host02`和`host03`将作为额外的控制平面节点加入，而`host04`将仅作为工作节点加入。

这是`host02`的 YAML 配置文件，带有其特定设置：

*kubeadm-join.yaml（host02）*

```
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: JoinConfiguration
discovery:
  bootstrapToken:
    apiServerEndpoint: 192.168.61.10:6443
    token: 1d8fb1.2875d52d62a3282d
    unsafeSkipCAVerification: true
  timeout: 5m0s
nodeRegistration:
  kubeletExtraArgs:
    cgroup-driver: containerd
    node-ip: 192.168.61.12
  taints: []
  ignorePreflightErrors:
    - DirAvailable--etc-kubernetes-manifests
controlPlane:
  localAPIEndpoint:
    advertiseAddress: 192.168.61.12
  certificateKey: "5a7e07816958efb97635e9a66256adb1"
```

该资源的类型为`JoinConfiguration`，但大部分字段与`kubeadm-init.yaml`文件中的`InitConfiguration`相同。最重要的是，`token`和`certificateKey`与我们之前设置的秘密匹配，因此此节点将能够验证自己并解密 API 服务器证书。

一个不同之处是新增了`ignorePreflightErrors`。这个部分只有在我们安装`kube-vip`时出现，因为在这种情况下，我们需要将`kube-vip`的配置文件预先放置到*/etc/kubernetes/manifests*目录，并且需要告诉`kubeadm`该目录已经存在是可以的。

因为我们有这个 YAML 配置文件，`kubeadm join`命令很简单。在`host02`上运行它：

```
root@host02:~# /usr/bin/kubeadm join --config /etc/kubernetes/kubeadm-join.yaml
```

和之前一样，这个命令使用本节点上的`kubelet`服务以容器的方式运行控制平面组件，因此需要一些时间来下载容器镜像并启动容器。

当它完成时，在`host03`上运行完全相同的命令：

```
root@host03:~# /usr/bin/kubeadm join --config /etc/kubernetes/kubeadm-join.yaml
```

自动化脚本已经为每个主机设置了正确的 IP 地址，因此每个主机之间的配置差异已经考虑到。

当这个命令完成时，我们将创建一个高可用的 Kubernetes 集群，控制平面组件在三个独立的主机上运行。然而，我们还没有常规的工作节点。让我们解决这个问题。

我们将从将`host04`作为常规工作节点加入开始，并在`host04`上运行完全相同的`kubeadm join`命令，但 YAML 配置文件会有所不同。以下是该文件：

*kubeadm-join.yaml（host04）*

```
---
apiVersion: kubeadm.k8s.io/v1beta3
kind: JoinConfiguration
discovery:
  bootstrapToken:
    apiServerEndpoint: 192.168.61.10:6443
    token: 1d8fb1.2875d52d62a3282d
    unsafeSkipCAVerification: true
  timeout: 5m0s
nodeRegistration:
  kubeletExtraArgs:
    cgroup-driver: containerd
    node-ip: 192.168.61.14
  taints: []
```

这个 YAML 文件缺少`controlPlane`字段，因此`kubeadm`将其配置为常规工作节点，而非控制平面节点。

现在让我们将`host04`加入集群：

```
root@host04:~# /usr/bin/kubeadm join --config /etc/kubernetes/kubeadm-join.yaml
```

这个命令完成得稍微快一些，因为它不需要下载控制平面容器镜像并运行它们。我们现在有四个节点在集群中，可以通过在`host01`上运行`kubectl`来验证：

```
root@host01:~# export KUBECONFIG=/etc/kubernetes/admin.conf
root@host01:~# kubectl get nodes
NAME     STATUS     ROLES        ...
host01   NotReady   control-plane...
host02   NotReady   control-plane...
host03   NotReady   control-plane...
host04   NotReady   <none>       ...
```

第一个命令设置了一个环境变量，告诉`kubectl`使用哪个配置文件。`/etc/kubernetes/admin.conf`文件是在`kubeadm`初始化`host01`作为控制平面节点时自动创建的。该文件告诉`kubectl`使用哪个地址来访问 API 服务器，使用哪个证书来验证安全连接，以及如何进行身份验证。

当前四个节点应该报告状态为`NotReady`。让我们运行`kubectl describe`命令以获取节点详细信息：

```
root@host01:~# kubectl describe node host04
Name:               host04
...
Conditions:
  Type   Status ... Message
  ----   ------ ... -------
  Ready  False  ... container runtime network not ready...
...
```

我们还没有为我们的 Kubernetes 集群安装网络驱动程序，因此所有节点都报告为`NotReady`状态，这意味着它们不会接受常规的应用工作负载。Kubernetes 通过在节点配置中放置一个*污点*来传达这一点。污点限制了可以在节点上调度的内容。我们可以使用`kubectl`列出节点上的污点：

```
root@host01:~# kubectl get node -o json | \
  jq '.items[]|.metadata.name,.spec.taints[]'
"host01"
{
  "effect": "NoSchedule",
  "key": "node.kubernetes.io/not-ready"
}
"host02"
{
  "effect": "NoSchedule",
  "key": "node.kubernetes.io/not-ready"
}
"host03"
{
  "effect": "NoSchedule",
  "key": "node.kubernetes.io/not-ready"
}
"host04"
{
  "effect": "NoSchedule",
  "key": "node.kubernetes.io/not-ready"
}
```

我们选择`json`格式的输出，以便可以使用`jq`仅打印我们需要的信息。由于所有节点的状态都是`NotReady`，它们都有一个`not-ready`污点，并设置为`NoSchedule`，这会阻止 Kubernetes 调度器将容器调度到这些节点上。

通过在`kubeadm`配置中将`taints`指定为空数组，我们防止了三个控制平面节点有额外的控制平面污点。在生产集群中，这个污点将应用容器与控制平面容器隔离开，以确保安全，因此我们会保留这个污点。不过，在我们的示例集群中，这意味着我们需要多个额外的虚拟机作为工作节点，而我们并不希望这样。

命令`kubectl taint`允许我们手动移除`not-ready`污点，但正确的方法是安装一个网络驱动程序作为集群附加组件，这样节点将正确报告为`Ready`，从而使我们能够在其上运行容器。

### 安装集群附加组件

我们已经在四个单独的节点上安装了`kubelet`，并在其中三个节点上安装了控制平面，并将它们加入到我们的集群中。对于剩下的节点，我们将使用控制平面来安装集群附加组件。这些附加组件类似于我们部署的常规应用程序。它们由 Kubernetes 资源组成，并在容器中运行，但它们为集群提供我们应用程序所需的基本服务。

要让基础集群启动并运行，我们需要安装三种类型的附加组件：*网络驱动程序*、*存储驱动程序*和*入口控制器*。我们还将安装一个第四个可选附加组件，*度量服务器*。

#### 网络驱动程序

Kubernetes 网络基于容器网络接口（CNI）标准。任何人都可以通过实现这一标准为 Kubernetes 构建新的网络驱动程序，因此 Kubernetes 网络驱动程序有多种选择。我们将在第八章中演示不同的网络插件，但本书中的大多数集群都使用 Calico 网络驱动程序，因为它是许多 Kubernetes 平台的默认选择。

首先，下载 Calico 的主要 YAML 配置文件：

```
root@host01:~# cd /etc/kubernetes/components
root@host01:/etc/kubernetes/components# curl -L -O $calico_url
...
```

`-L`选项告诉`curl`跟随任何 HTTP 重定向，而`-O`选项则告诉`curl`将内容保存到文件中，文件名与 URL 中的文件名相同。`calico_url`环境变量的值在`k8s-ver`脚本中设置，该脚本还指定了 Kubernetes 的版本。这是非常重要的，因为 Calico 对我们运行的 Kubernetes 版本非常敏感，因此选择兼容的版本非常关键。

主要的 YAML 配置文件写入本地文件*tigera-operator.yaml*。这指的是初始安装是一个 Kubernetes Operator，之后它会创建所有其他集群资源来安装 Calico。我们将在第十七章中探讨 Operator。

除了这个主要的 YAML 配置文件，本章节的自动化脚本还添加了一个名为*custom-resources.yaml*的文件，为我们的示例集群提供了必要的配置。现在，我们可以告诉 Kubernetes API 服务器将这些文件中的所有资源应用到集群中：

```
root@host01:/etc/kubernetes/components# kubectl apply -f tigera-operator.yaml
...
root@host01:/etc/kubernetes/components# kubectl apply -f custom-resources.yaml
```

Kubernetes 需要几分钟来下载容器镜像并启动容器，之后 Calico 将在我们的集群中运行，节点应该报告为`Ready`状态：

```
root@host01:/etc/kubernetes/components# kubectl get nodes
NAME     STATUS   ROLES                ...
host01   Ready    control-plane,master ...
host02   Ready    control-plane,master ...
host03   Ready    control-plane,master ...
host04   Ready    <none>               ...
```

Calico 通过安装一个*DaemonSet*工作，这是一个 Kubernetes 资源，指示集群在每个节点上运行特定的容器或一组容器。Calico 容器随后为在该节点上运行的任何容器提供网络服务。然而，这引出了一个重要问题。当我们在集群中安装 Calico 时，所有节点都有一个污点，告诉 Kubernetes 不要在其上调度容器。那么，Calico 是如何在所有节点上运行容器的呢？答案是*容忍*。

容忍是应用于资源的配置设置，指示 Kubernetes 即使可能存在污点，也可以将该资源调度到节点上。Calico 在将其 DaemonSet 添加到集群时会指定一个容忍设置，正如我们通过`kubectl`所看到的：

```
root@host01:/etc/kubernetes/components# kubectl -n calico-system \
  get daemonsets -o json | \
  jq '.items[].spec.template.spec.tolerations[]'
{
  "key": "CriticalAddonsOnly",
  "operator": "Exists"
}
{
  "effect": "NoSchedule",
  "operator": "Exists"
}
{
  "effect": "NoExecute",
  "operator": "Exists"
}
```

`-n`选项选择`calico-system`*命名空间*。命名空间是 Kubernetes 用来将集群资源彼此隔离的一种方式，既出于安全原因，也为了避免命名冲突。此外，与之前一样，我们请求 JSON 输出，并使用`jq`选择我们感兴趣的字段。如果你想查看资源的完整配置，可以使用`-o=json`而不带`jq`，或者使用`-o=yaml`。

这个 DaemonSet 有三个容忍设置，第二个容忍设置提供了我们所需的行为。它告诉 Kubernetes 调度程序即使在节点上存在`NoSchedule`污点，也可以继续调度。这样，Calico 就可以在节点准备好之前启动，而一旦运行，它会将节点状态更改为`Ready`，从而可以调度正常的应用程序容器。控制平面组件也需要类似的容忍设置，才能在节点显示`Ready`之前运行。

#### 安装存储

集群节点已经准备好，所以如果我们部署一个常规应用，其容器将运行。然而，要求持久存储的应用将无法启动，因为集群还没有存储驱动程序。像网络驱动程序一样，Kubernetes 有多个存储驱动程序可供选择。容器存储接口（CSI）提供了存储驱动程序与 Kubernetes 配合使用所需满足的标准。我们将使用 Longhorn，这是一个来自 Rancher 的存储驱动程序；它安装简单，并且不需要额外的硬件支持，如额外的块设备或访问基于云的存储。

Longhorn 利用我们之前安装的 iSCSI 和 NFS 软件。它要求所有节点都启用了并正在运行 `iscsid` 服务，因此我们需要确保所有节点都满足这一要求：

```
root@host01:/etc/kubernetes/components# k8s-all systemctl enable --now iscsid
```

现在我们可以在集群上安装 Longhorn。安装 Longhorn 的过程与 Calico 很相似。首先下载 Longhorn 的 YAML 配置文件：

```
root@host01:/etc/kubernetes/components# curl -LO $longhorn_url
```

`longhorn_url` 环境变量同样由 `k8s-ver` 脚本设置，这让我们能够确保兼容性。

使用 `kubectl` 安装 Longhorn：

```
root@host01:/etc/kubernetes/components# kubectl apply -f longhorn.yaml
```

和之前一样，`kubectl apply` 确保 YAML 文件中的资源被应用到集群中，并根据需要创建或更新它们。`kubectl apply` 命令支持将 URL 作为资源的来源应用到集群，但对于这三次安装，我们运行一个单独的 `curl` 命令，因为方便拥有一个本地副本来应用到集群中的内容。

Longhorn 已经安装在集群上，我们将在本章接下来的内容中验证这一点。

#### 入口控制器

现在我们已经有了网络和存储，但目前的网络仅允许从我们集群内部访问容器。我们还需要一个将容器化应用暴露到集群外部的服务。最简单的方法是使用入口控制器。正如我们在第九章中所描述的，入口控制器监视 Kubernetes 集群中的 *Ingress* 资源并路由网络流量。

我们首先下载入口控制器的 YAML 配置文件：

```
root@host01:/etc/kubernetes/components# curl -Lo ingress-controller.yaml
  $ingress_url
```

和我们之前的例子一样，`ingress_url` 环境变量由 `k8s-ver` 脚本设置，以确保兼容性。在这种情况下，URL 以通用路径 *deploy.yaml* 结尾，因此我们使用 `-o` 为 `curl` 提供文件名，以明确说明下载的 YAML 文件的用途。

使用 `kubectl` 安装入口控制器：

```
root@host01:/etc/kubernetes/components# kubectl apply -f ingress-controller.yaml
```

这会创建很多资源，但主要有两个部分：一个实际执行 HTTP 流量路由的 NGINX Web 服务器，以及一个监视集群中 Ingress 资源变化并相应配置 NGINX 的组件。

还有一步我们需要完成。当前安装的 ingress 控制器尝试请求一个外部 IP 地址，以便允许外部流量访问它。由于我们运行的是一个没有外部 IP 地址访问权限的示例集群，因此此方法不可行。相反，我们将通过集群主机的端口转发来访问 ingress 控制器。目前，我们的 ingress 控制器已经配置为支持端口转发，但它使用的是一个随机端口。我们希望选择一个端口，以确保知道如何找到 ingress 控制器。同时，我们还将添加一个注释，以使该 ingress 控制器成为此集群的默认控制器。

为了应用端口更改，我们将为我们的 Kubernetes 集群提供一个额外的 YAML 配置文件，其中仅包含我们需要的更改。以下是该 YAML 文件：

*ingress-patch.yaml*

```
---
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  ports:
    - port: 80
      nodePort: 80
    - port: 443
      nodePort: 443
```

此文件指定了服务的名称和命名空间，以确保 Kubernetes 知道在哪些位置应用这些更改。它还指定了我们正在更新的 `port` 配置，以及用于端口转发的集群节点端口 `nodePort`。我们将在 第九章 中更详细地讨论 NodePort 服务类型和端口转发。

要修补该服务，我们使用 `kubectl patch` 命令：

```
root@host01:/etc/kubernetes/components# kubectl patch -n ingress-nginx \
  service/ingress-nginx-controller --patch-file ingress-patch.yaml
service/ingress-nginx-controller patched
```

要应用注释，请使用 `kubectl annotate` 命令：

```
root@host01:/etc/kubernetes/components# kubectl annotate -n ingress-nginx \
  ingressclass/nginx ingressclass.kubernetes.io/is-default-class="true"
ingressclass.networking.k8s.io/nginx annotated
```

Kubernetes 在我们进行每次更改时都会向每个资源报告更改情况，因此我们可以知道我们的更改已经应用。

#### 指标服务器

我们的最终附加组件是一个 *指标服务器*，它从我们的节点收集利用率指标，从而启用自动扩缩。为此，它需要连接到集群中的 `kubelet` 实例。出于安全考虑，它在连接到 `kubelet` 时需要验证 HTTP/S 证书。这就是为什么我们将 `kubelet` 配置为请求由控制器管理器签名的证书，而不是允许 `kubelet` 生成自签名证书的原因。

在设置过程中，`kubelet` 在每个节点上创建了一个证书请求，但这些请求并未自动批准。让我们查找这些请求：

```
root@host01:/etc/kubernetes/components# kubectl get csr
NAME      ... SIGNERNAME                                  ... CONDITION
csr-sgrwz ... kubernetes.io/kubelet-serving               ... Pending
csr-agwb6 ... kubernetes.io/kube-apiserver-client-kubelet ... Approved,Issued
csr-2kwwk ... kubernetes.io/kubelet-serving               ... Pending
csr-5496d ... kubernetes.io/kube-apiserver-client-kubelet ... Approved,Issued
csr-hm6lj ... kubernetes.io/kube-apiserver-client-kubelet ... Approved,Issued
csr-jbfmx ... kubernetes.io/kubelet-serving               ... Pending
csr-njjr7 ... kubernetes.io/kube-apiserver-client-kubelet ... Approved,Issued
csr-v7tcs ... kubernetes.io/kubelet-serving               ... Pending
csr-vr27n ... kubernetes.io/kubelet-serving               ... Pending
```

每个 `kubelet` 都有一个客户端证书，用于向 API 服务器进行身份验证；这些证书在引导过程中已自动批准。我们需要批准的请求是 `kubelet-serving` 证书请求，这些证书在客户端（如我们的指标服务器）连接到 `kubelet` 时使用。一旦请求被批准，控制器管理器就会签署证书。然后，`kubelet` 会收集该证书并开始使用它。

我们可以通过查询所有 `kubelet-serving` 请求的名称，并将这些名称传递给 `kubectl certificate approve`，一次性批准所有这些请求：

```
root@host01:/etc/kubernetes/components# kubectl certificate approve \$(kubectl
  get csr --field-selector spec.signerName=kubernetes.io/kubelet-serving -o name)
certificatesigningrequest.certificates.k8s.io/csr-sgrwz approved
...
```

我们现在可以通过下载并应用其 YAML 配置来安装我们的指标服务器：

```
root@host01:/etc/kubernetes/components# curl -Lo metrics-server.yaml \$metrics_url
root@host01:/etc/kubernetes/components# kubectl apply -f metrics-server.yaml
...
root@host01:/etc/kubernetes/components# cd
root@host01:~#
```

这个组件是我们需要安装的最后一个，因此我们可以离开这个目录。通过这些集群附加组件，我们现在拥有一个完整且高可用的 Kubernetes 集群。

### 探索集群

在将第一个应用程序部署到这个全新的 Kubernetes 集群之前，让我们先探索一下它上面正在运行的内容。我们在这里使用的命令将对以后调试我们自己的应用程序和一个运行不正常的集群时很有帮助。

我们将使用 `crictl`，这是我们在第一部分中用来探索运行容器的相同命令，来查看在 `host01` 上运行的容器：

```
root@host01:~# crictl ps
CONTAINER       ... STATE    NAME                       ...
25c63f29c1442   ... Running  longhorn-csi-plugin        ...
2ffdd044a81d8   ... Running  node-driver-registrar      ...
94468050de89c   ... Running  csi-provisioner            ...
119fbf417f1db   ... Running  csi-attacher               ...
e74c1a2a0c422   ... Running  kube-scheduler             ...
d1ad93cdbc686   ... Running  kube-controller-manager    ...
76266a522cc3d   ... Running  engine-image-ei-611d1496   ...
fc3cd1679e33e   ... Running  replica-manager            ...
48e792a973105   ... Running  engine-manager             ...
e658baebbc295   ... Running  longhorn-manager           ...
eb51d9ec0f2fc   ... Running  calico-kube-controllers    ...
53e7e3e4a3148   ... Running  calico-node                ...
772ac45ceb94e   ... Running  calico-typha               ...
4005370021f5f   ... Running  kube-proxy                 ...
26929cde3a264   ... Running  kube-apiserver             ...
9ea4c2f5af794   ... Running  etcd                       ...
```

控制平面节点非常忙碌，因为这个列表包括 Kubernetes 控制平面组件、Calico 组件和 Longhorn 组件。如果在所有节点上运行此命令，并且整理出各个容器在哪里运行以及其目的，这将会让人感到困惑。幸运的是，`kubectl` 提供了更清晰的视图，尽管知道我们可以深入到这些底层细节，准确查看在某个节点上运行的容器是什么，还是很有用的。

要使用 `kubectl` 探索集群，我们需要知道集群资源是如何组织到命名空间中的。如前所述，Kubernetes 命名空间提供安全性并避免名称冲突。为了确保幂等性，Kubernetes 需要每个资源都有一个唯一的名称。通过将资源划分到命名空间中，我们允许多个资源具有相同的名称，同时仍然使 API 服务器能够确切地知道我们指的是什么资源，这也支持多租户，这是我们的一个跨切面问题。

即使我们刚刚设置了集群，它已经填充了几个命名空间：

```
root@host01:~# kubectl get namespaces
NAME              STATUS   AGE
calico-system     Active   50m
default           Active   150m
kube-node-lease   Active   150m
kube-public       Active   150m
kube-system       Active   150m
longhorn-system   Active   16m
tigera-operator   Active   50m
```

当我们运行 `kubectl` 命令时，它们将应用于 `default` 命名空间，除非我们使用 `-n` 选项来指定不同的命名空间。

要查看哪些容器正在运行，我们可以使用 `kubectl` 获取 Pod 列表。我们将在第七章中更详细地查看 Kubernetes Pods。现在，只需知道 Pod 是一个或多个容器的集合，类似于我们在第一部分中使用 `crictl` 创建的 Pods。

如果我们尝试列出 `default` 命名空间中的 Pods，我们可以看到目前还没有任何 Pods：

```
root@host01:~# kubectl get pods
No resources found in default namespace.
```

到目前为止，当我们安装集群基础设施组件时，它们都被创建在其他命名空间中。这样，当我们配置普通用户帐户时，可以防止这些用户查看或编辑集群基础设施。Kubernetes 基础设施组件都被安装到了 `kube-system` 命名空间：

```
root@host01:~# kubectl -n kube-system get pods
NAME                             READY   STATUS    ...
coredns-558bd4d5db-7krwr         1/1     Running   ...
...
kube-apiserver-host01            1/1     Running   ...
...
```

我们在第十一章中讨论了控制平面组件。现在，让我们先探索其中一个控制平面 Pod——运行在 `host01` 上的 API 服务器。我们可以使用 `kubectl describe` 获取此 Pod 的所有详细信息：

```
root@host01:~# kubectl -n kube-system describe pod kube-apiserver-host01
Name:                 kube-apiserver-host01
Namespace:            kube-system
...
Node:                 host01/192.168.61.11
...
Status:               Running
Containers:
  kube-apiserver:
    Container ID:  containerd://26929cde3a264e...
...
```

命名空间和名称共同唯一标识这个 Pod。我们还可以看到 Pod 所在的节点、其状态以及关于实际容器的详细信息，包括一个容器 ID，我们可以使用 `crictl` 来找到底层 `containerd` 运行时中的容器。

让我们还验证一下 Calico 是否按预期部署到集群中：

```
root@host01:~# kubectl -n calico-system get pods
NAME                                       READY   STATUS    ...
calico-kube-controllers-7f58dbcbbd-ch7zt   1/1     Running   ...
calico-node-cp88k                          1/1     Running   ...
calico-node-dn4rj                          1/1     Running   ...
calico-node-xnkmg                          1/1     Running   ...
calico-node-zfscp                          1/1     Running   ...
calico-typha-68b99cd4bf-7lwss              1/1     Running   ...
calico-typha-68b99cd4bf-jjdts              1/1     Running   ...
calico-typha-68b99cd4bf-pjr6q              1/1     Running   ...
```

之前我们看到 Calico 安装了一个 DaemonSet 资源。Kubernetes 使用这个 DaemonSet 中的配置，自动为每个节点创建一个 `calico-node` Pod。像 Kubernetes 本身一样，Calico 也使用一个独立的控制平面来处理网络的整体配置，而其他 Pods 则提供该控制平面。

最后，我们将查看为 Longhorn 运行的容器：

```
root@host01:~# kubectl -n longhorn-system get pods
NAME                                       READY   STATUS    RESTARTS   AGE
engine-image-ei-611d1496-8q58f             1/1     Running   0          31m
...
longhorn-csi-plugin-8vkr6                  2/2     Running   0          31m
...
longhorn-manager-dl9sb                     1/1     Running   1          32m
...
```

与 Calico 类似，Longhorn 使用 DaemonSets，使其能够在每个节点上运行容器。这些容器为节点上的其他容器提供存储服务。Longhorn 还包括许多其他容器，它们作为控制平面运行，包括提供 Kubernetes 用来指示 Longhorn 在需要时创建存储的 CSI 实现。

我们花了很多精力来设置这个集群，因此，如果在本章结束时没有至少运行一个应用程序，那将是非常可惜的。在下一章中，我们将探讨多种不同的运行容器方式，但我们先在 Kubernetes 集群中快速运行一个简单的 NGINX 网络服务器：

```
root@host01:~# kubectl run nginx --image=nginx
pod/nginx created
```

这看起来像一个命令式的指令，但实际上，`kubectl` 正在使用我们指定的名称和容器镜像创建一个 Pod 资源，并将该资源应用于集群。让我们再次查看默认的命名空间：

```
root@host01:~# kubectl get pods -o wide
NAME    READY   STATUS    ... IP               NODE  ...
nginx   1/1     Running   ... 172.31.89.203   host02 ...
```

我们使用了 `-o wide` 来查看关于 Pod 的额外信息，包括其 IP 地址和调度位置，这些每次创建 Pod 时可能会不同。在这个例子中，Pod 被调度到了 `host02`，这表明我们成功地允许常规应用容器部署到我们的控制平面节点。IP 地址来自我们配置的 Pod CIDR，并由 Calico 自动分配。

Calico 还处理路由流量，以便我们可以从集群中的任何容器以及从主机网络访问 Pod。让我们验证这一点，从一个常规的 `ping` 开始：

```
root@host01:~# ping -c 1 172.31.89.203
PING 172.31.89.203 (172.31.89.203) 56(84) bytes of data.
64 bytes from 172.31.89.203: icmp_seq=1 ttl=63 time=0.848 ms

--- 172.31.89.203 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.848/0.848/0.848/0.000 ms
```

在此处替换为您 Pod 的 IP 地址。

我们还可以使用 `curl` 来验证 NGINX 网络服务器是否正常工作：

```
root@host01:~# curl http://172.31.89.203
...
<title>Welcome to nginx!</title>
...
```

Kubernetes 集群已经正常工作，准备好供我们部署应用。Kubernetes 将利用集群中的所有节点来负载均衡我们的应用，并在发生故障时提供弹性。

### 最后的思考

在这一章中，我们探讨了 Kubernetes 的架构，具备灵活性，允许集群组件随时加入或退出。这不仅适用于容器化应用程序，也适用于集群组件，包括控制平面微服务以及集群所使用的底层服务器和网络。我们成功地引导启动了一个集群，并动态地向其中添加了节点，配置这些节点接受特定类型的容器，然后使用 Kubernetes 集群本身动态地添加网络和存储驱动程序来运行和监控它们。最后，我们将第一个容器部署到 Kubernetes 集群，允许它自动将容器调度到可用节点上，并通过我们的网络驱动程序从主机网络访问该容器。

现在我们有了一个高可用的集群，接下来可以看看如何将应用程序部署到 Kubernetes。我们将探索一些关键的 Kubernetes 资源，这些资源是创建可扩展、可靠的应用程序所必需的。这个过程将为我们深入探索 Kubernetes 奠定基础，包括了解当我们的应用程序未按预期运行时发生了什么，以及如何调试应用程序或 Kubernetes 集群的问题。
