# 第二章：进程隔离

![image](img/common01.jpg)

容器建立在一系列技术的基础上，这些技术旨在隔离一个计算机程序与另一个程序，同时允许多个程序共享相同的 CPU、内存、存储和网络资源。容器利用 Linux 内核的基本能力，特别是命名空间（namespace），它们创建了进程标识符、用户、文件系统和网络接口的独立视图。容器运行时使用多种类型的命名空间，为每个容器提供系统的隔离视图。

在本章中，我们将考虑进程隔离的一些原因，并回顾 Linux 是如何历史性地实现进程隔离的。然后，我们将研究容器如何使用命名空间来提供隔离。我们将通过几种不同的容器运行时进行测试。最后，我们将使用 Linux 命令直接创建命名空间。

### 理解隔离

在运行一些容器并检查其隔离性之前，让我们先看看进程隔离的动机。我们还将考虑 Linux 中传统的进程隔离方式，以及这如何促成了容器所使用的隔离能力。

#### 为什么进程需要隔离

计算机的整体概念是它是一台通用机器，可以运行多种不同类型的程序。自计算机诞生以来，就有需要在多个程序之间共享同一台计算机的需求。最初，人们通过打孔卡片轮流提交程序，但随着计算机多任务处理的日益复杂，人们可以启动多个程序，而计算机会让它们看起来好像都在同一个 CPU 上同时运行。

当然，一旦某个资源需要共享，就需要确保共享是公平的，计算机程序也不例外。所以，尽管我们认为一个*进程*是一个独立的程序，拥有自己的 CPU 时间和内存空间，但有许多方式可能导致一个进程对另一个进程造成困扰，包括：

+   使用过多的 CPU、内存、存储或网络资源

+   覆盖另一个进程的内存或文件

+   从另一个进程中提取机密信息

+   向另一个进程发送错误数据，导致其行为异常

+   向另一个进程发送大量请求，使其停止响应

错误可能会导致进程意外地做出这些行为，但更大的问题是安全漏洞，允许恶意行为者利用一个进程对另一个进程造成问题。一个漏洞就足以在系统中造成重大问题，因此我们需要能够隔离进程的方式，以限制意外和故意行为带来的损害。

物理隔离是最好的——*气隔*系统常常被用于保护政府机密信息和安全关键系统——但这种方法对于许多应用来说也过于昂贵且不便。虚拟机可以在共享物理硬件的同时，提供隔离的外观，但虚拟机需要运行自己的操作系统、服务和虚拟设备，导致启动更慢，扩展性差。解决方案是运行常规进程，但利用进程隔离来降低影响其他进程的风险。

#### 文件权限与变更根目录

大多数进程隔离的工作集中在防止一个进程看到它不应该看到的内容。毕竟，如果一个进程甚至无法看到另一个进程，它就更难制造麻烦，无论是意外还是故意。Linux 传统上控制进程能够看到和做什么的方式为容器的思想提供了基础。

最基本的可见性控制之一是*文件系统权限*。Linux 为每个文件和目录关联一个所有者和一个组，并管理读、写和执行权限。这种基本的权限方案能够很好地确保用户文件的私密性，防止进程覆盖另一个进程的文件，并确保只有像 root 这样的特权用户才能安装新软件或修改关键的系统配置文件。

当然，这种权限方案依赖于我们确保每个进程以真实用户身份运行，并且用户位于适当的组中。通常，每个新服务安装都会为该服务创建一个专用用户。更好的是，这个*服务用户*可以配置为没有真实的登录 shell，这意味着该用户无法被利用登录系统。为了更清楚地说明这一点，我们来看一个示例。

**注意**

*本书的示例仓库在* [`github.com/book-of-kubernetes/examples`](https://github.com/book-of-kubernetes/examples)。*有关设置的详细信息，请参见第 xx 页中的“运行示例”。*

Linux 的`rsyslogd`服务提供日志服务，因此需要写入*/var/log*中的文件，但不应该有权限读取或写入该目录中的所有文件。文件权限用于控制这一点，示例如下：

```
   root@host01:~# ps -ef | grep rsyslogd | grep -v grep
➊ syslog  698  1  0 Mar05 ?   00:00:04 /usr/sbin/rsyslogd -n -iNONE
   root@host01:~# su syslog
➋ This account is currently not available.
   root@host01:~# ls -l /var/log/auth.log
➌ -rw-r----- 1 syslog adm 18396 Mar  6 01:27 /var/log/auth.log
   root@host01:~# ls -ld /var/log/private
➍ drwx------ 2 root root 4096 Mar  5 21:04 /var/log/private
```

*syslog*用户 ➊ 专门用于运行`rsyslogd`，并且出于安全原因，该用户被配置为没有登录 shell ➋。由于`rsyslogd`需要能够写入*auth.log*，因此赋予了写权限，如文件模式输出中所示 ➌。管理员组（*adm*）的成员对该文件具有只读权限。

文件模式中的初始 `d` ➍ 表示这是一个目录。接下来的 `rwx` 表示 root 用户具有读、写和执行权限。其余的破折号表示 root 组的成员或其他系统用户没有权限，因此我们可以推断出 `rsyslogd` 进程无法查看此目录的内容。

权限控制很重要，但它并不能完全满足我们对进程隔离的需求。一个原因是，它不足以防止*特权升级*，即一个脆弱的进程和系统可能让恶意行为者获得 root 权限。为了解决这个问题，一些 Linux 服务通过在文件系统的隔离部分中运行来进一步加强安全。这种方法被称为 `chroot`，即“更改根目录”。在 `chroot` 环境中运行需要一些配置，正如我们在这个示例中看到的那样：

```
   root@host01:~# mkdir /tmp/newroot
   root@host01:~# ➊ cp --parents /bin/bash /bin/ls /tmp/newroot
   root@host01:~# cp --parents /lib64/ld-linux-x86-64.so.2 \
  ➋ $(ldd /bin/bash /bin/ls | grep '=>' | awk '{print $3}') /tmp/newroot
   ...
   root@host01:~# ➌ chroot /tmp/newroot /bin/bash
   bash-5.0# ls -l /bin
   total 1296
➍ -rwxr-xr-x 1 0 0 1183448 Mar  6 02:15 bash
   -rwxr-xr-x 1 0 0  142144 Mar  6 02:15 ls
   bash-5.0# exit
   exit
```

首先，我们需要将所有打算运行的可执行文件复制到容器中 ➊。我们还需要将这些可执行文件使用的所有共享库复制进来，我们通过 `ldd | grep | awk` 命令来指定这些库 ➋。当二进制文件和库都被复制进容器后，我们可以使用 `chroot` 命令 ➌ 进入隔离环境。只有我们复制进来的文件是可见的 ➍。

#### 容器隔离

对于有经验的 Linux 系统管理员来说，文件权限和更改根目录是基础知识。然而，这些概念也为容器的工作原理提供了基础。尽管正在运行的容器看起来像是一个完全独立的系统，拥有自己的主机名、网络、进程和文件系统（正如我们在第一章中看到的那样），它实际上只是一个普通的 Linux 进程，利用隔离机制而不是虚拟机。

一个容器具有多种隔离方式，包括一些我们之前未曾见过的关键隔离类型：

+   挂载的文件系统

+   主机名和域名

+   进程间通信

+   进程标识符

+   网络设备

这些不同类型的隔离机制共同作用，使得一个进程或一组进程看起来像是一个完全独立的系统。尽管这些进程仍然共享内核和物理硬件，但这种隔离机制大大确保了它们不会对其他进程造成困扰，特别是当我们正确配置容器，以控制它们可用的 CPU、内存、存储和网络资源时。

### 容器平台和容器运行时

指定在隔离文件系统中运行进程所需的所有二进制文件、库和配置文件会很繁琐。幸运的是，正如我们在第一章中看到的那样，*容器镜像*已经预先打包了所需的可执行文件和库。通过使用 Docker，我们能够轻松下载并在容器中运行 NGINX。Docker 是一个*容器平台*的例子，提供了不仅是运行容器的能力，还有容器存储、网络和安全性。

在背后，现代版本的 Docker 使用`containerd`作为*容器运行时*，也被称为*容器引擎*。容器运行时提供了在容器中运行进程的底层功能。

为了进一步探索隔离性，让我们实验使用两种不同的容器运行时，从现有镜像启动容器，然后检查容器中进程如何与系统的其他部分隔离。

#### 安装 containerd

我们将在第二部分中使用`containerd`来支持我们的 Kubernetes 集群，因此让我们首先安装并直接与这个运行时交互。直接与`containerd`交互也将有助于我们探索进程隔离。

你可以通过使用本章示例提供的*额外*配置脚本跳过安装命令。请参阅本章的 README 文件以获取说明。

尽管`containerd`可以在标准的 Ubuntu 软件包库中找到，我们还是会从官方的 Docker 软件包库安装，以确保我们获得最新的稳定版本。为此，我们需要让 Apt 支持 HTTP/S 协议，因此我们首先需要进行此设置：

```
root@host01:~# apt update
...
root@host01:~# apt -y install apt-transport-https
...
```

现在让我们添加包注册表并进行安装：

```
root@host01:~# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
root@host01:~# echo "deb [arch=amd64" \
  "signed-by=/usr/share/keyrings/docker-archive-keyring.gpg]" \
  "https://download.docker.com/linux/ubuntu focal stable" > \
  /etc/apt/sources.list.d/docker.list
root@host01:~# apt update && apt install -y containerd.io
...
root@host01:~# ctr images ls
REF TYPE DIGEST SIZE PLATFORMS LABELS
```

最后的命令只是确保包已正确安装，服务正在运行，并且`ctr`命令可以正常工作。我们没有看到任何镜像，因为我们还没有安装任何镜像。

容器运行时是底层库。通常不会直接使用它们，而是由更高层的容器平台或编排环境（例如 Docker 或 Kubernetes）使用。这意味着它们会将大量精力放在高质量的应用程序编程接口（API）上，但不会在命令行工具上花费太多精力，尽管这些工具仍然是测试所必需的。幸运的是，`containerd`提供了我们将用于实验的`ctr`工具。

#### 使用 containerd

我们最初的`containerd`命令显示尚未下载任何镜像。让我们下载一个小镜像，用于运行容器。我们将使用*BusyBox*，这是一个包含 shell 和基本 Linux 工具的小型容器镜像。为了下载镜像，我们使用`pull`命令：

```
root@host01:~# ctr image pull docker.io/library/busybox:latest
...
root@host01:~# ctr images ls
REF                              ...
docker.io/library/busybox:latest ...
```

我们的镜像列表不再为空。让我们从这个镜像运行一个容器：

```
root@host01:~# ctr run -t --rm docker.io/library/busybox:latest v1
/ #
```

这看起来与使用 Docker 类似。我们使用`-t`来为这个容器创建一个 TTY，以便与其交互，并使用`--rm`告诉`containerd`在主进程停止时删除容器。然而，有一些重要的区别需要注意。当我们在第一章中使用 Docker 时，我们并没有担心在运行容器之前拉取镜像，我们可以使用像`nginx`或`rockylinux:8`这样的简化名称。`ctr`工具要求我们指定*docker.io/library/busybox:latest*，即镜像的完整路径，包括注册表主机名和标签。另外，我们需要先拉取镜像，因为运行时不会自动为我们做这件事。

现在我们进入这个容器，可以看到它具有隔离的网络栈和进程空间：

```
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
        valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
        valid_lft forever preferred_lft forever
/ # ps -ef
PID   USER     TIME  COMMAND
    1 root      0:00 sh
    8 root      0:00 ps -ef
/ #
```

在容器内部，我们看到一个回环网络接口。我们还看到我们的 shell 进程和我们运行的 `ps` 命令。就容器中的进程而言，我们正在一个没有其他进程运行或在网络上监听的独立系统上运行。

**为什么没有桥接接口？**

如果你曾经使用过 Docker，可能会惊讶地发现这个容器只有一个回环接口。容器平台的默认网络配置通常还提供一个附加的接口，该接口连接到一个桥接。这样，容器之间可以互相看到，并且容器可以通过网络地址转换（NAT）使用主机接口访问外部网络。

在这种情况下，我们直接与一个较低级别的容器运行时进行交互。这个容器运行时仅处理镜像管理和容器运行。如果我们需要一个桥接接口和互联网连接，就需要自己提供（我们在第四章中正是这么做的）。

我们已经说明了如何与 `containerd` 运行时交互以运行容器，并且在容器内部，我们与系统的其他部分是隔离的。这个隔离是如何工作的呢？为了找出答案，我们将继续让容器运行并从主机系统进行调查。

#### 介绍 Linux 命名空间

和其他容器运行时一样，`containerd` 使用名为 *命名空间* 的 Linux 内核特性来隔离容器中的进程。如前所述，进程隔离的主要工作是确保进程看不到它不该看到的东西。运行在命名空间中的进程只能看到特定系统资源的有限视图。

尽管容器化看起来像是新技术，但 Linux 命名空间已经存在了多年。随着时间的推移，添加了更多类型的命名空间。我们可以使用 `lsns` 命令找出与我们的容器相关联的命名空间，但首先我们需要知道容器内 shell 进程在主机上的进程 ID（PID）。在保持容器运行的同时，打开另一个终端标签或窗口。（有关更多信息，请参见第 xx 页中的“运行示例”）。然后，使用 `ctr` 列出正在运行的容器：

```
root@host01:~# ctr task ls
TASK    PID      STATUS    
v1      18088    RUNNING
```

让我们使用 `ps` 来验证我们是否得到了正确的 PID。当你自己运行这些命令时，请务必使用列出中显示的 PID：

```
root@host01:~# ps -ef | grep 18088 | grep -v grep
root       18088   18067  0 18:46 pts/0    00:00:00 sh
root@host01:~# ps -ef | grep 18067 | grep -v grep
root       18067       1  0 18:46 ?        00:00:00 
  /usr/bin/containerd-shim-runc-v2 -namespace default -id v1 -address 
  /run/containerd/containerd.sock
root       18088   18067  0 18:46 pts/0    00:00:00 sh
```

正如预期的那样，这个 PID 的父进程是 `containerd`。接下来，让我们使用 `lsns` 列出 `containerd` 创建的命名空间，以隔离这个进程：

```
root@host01:~# lsns | grep 18088
4026532180 mnt         1 18088 root            sh
4026532181 uts         1 18088 root            sh
4026532182 ipc         1 18088 root            sh
4026532183 pid         1 18088 root            sh
4026532185 net         1 18088 root            sh
```

在这里，`containerd` 使用五种不同类型的命名空间来完全隔离在 `busybox` 容器中运行的进程：

mnt 挂载点

uts Unix 时间共享（主机名和网络域）

ipc 进程间通信（例如，共享内存）

pid 进程标识符（以及正在运行的进程列表）

net 网络（包括接口、路由表和防火墙）

最后，我们通过在该容器中运行`exit`来关闭 BusyBox 容器（第一个终端窗口）：

```
/ # exit
```

该命令将返回常规的 Shell 提示符，使我们准备好进行下一组示例。

#### CRI-O 中的容器和命名空间

除了`containerd`，Kubernetes 还支持其他容器运行时。根据你使用的 Kubernetes 发行版，你可能会发现容器运行时不同。例如，Red Hat OpenShift 使用*CRI-O*，这是另一种容器运行时。CRI-O 还被 Podman、Buildah 和 Skopeo 工具套件使用，它们是 Red Hat 8 及相关系统上管理容器的标准方式。

让我们使用 CRI-O 运行相同的容器镜像，以便更好地了解容器运行时如何彼此不同，但也展示它们如何利用相同的 Linux 内核功能进行进程隔离。

你可以通过使用本章示例中提供的*额外*预配脚本跳过这些安装命令。有关说明，请参阅本章的 README 文件。

OpenSUSE Kubic 项目为各种 Linux 发行版（包括 Ubuntu）提供 CRI-O 的存储库，因此我们将从那里安装。具体的 URL 取决于我们要安装的 CRI-O 版本，且 URL 较长并且难以输入，因此自动化会安装一个脚本来配置一些有用的环境变量。在继续之前，我们需要加载该脚本：

```
root@host01:~# source /opt/crio-ver
```

我们现在可以使用环境变量来设置 CRI-O 存储库并安装 CRI-O：

```
root@host01:~# echo "deb $REPO/$OS/ /" > /etc/apt/sources.list.d/kubic.list
root@host01:~# echo "deb $REPO:/cri-o:/$VERSION/$OS/ /" \
  > /etc/apt/sources.list.d/kubic.cri-o.list
root@host01:~# curl -L $REPO/$OS/Release.key | apt-key add -
...
OK
root@host01:~# apt update && apt install -y cri-o cri-o-runc
...
root@host01:~# systemctl enable crio && systemctl start crio
...
root@host01:~# curl -L -o /tmp/crictl.tar.gz $CRICTL_URL
...
root@host01:~# tar -C /usr/local/bin -xvzf /tmp/crictl.tar.gz
crictl
root@host01:~# rm -f /tmp/crictl.tar.gz
```

我们首先通过向*/etc/apt/sources.list.d*添加文件来将 CRI-O 添加到`apt`的存储库列表中。然后我们使用`apt`安装 CRI-O 软件包。安装 CRI-O 后，我们使用`systemd`启用并启动其服务。

与`containerd`不同，CRI-O 没有附带我们可以用来进行测试的命令行工具，因此最后一条命令安装了`crictl`，它是 Kubernetes 项目的一部分，旨在测试任何与容器运行时接口（CRI）标准兼容的容器运行时。CRI 是 Kubernetes 本身用于与容器运行时通信的编程 API。

因为`crictl`与任何支持 CRI 的容器运行时兼容，所以需要配置它以连接到 CRI-O。CRI-O 已安装了一个配置文件*/etc/crictl.yaml*来配置`crictl`：

*crictl.yaml*

```
runtime-endpoint: unix:///var/run/crio/crio.sock
image-endpoint: unix:///var/run/crio/crio.sock
...
```

这个配置告诉`crictl`连接到 CRÍ-O 的套接字。

要创建和运行容器，`crictl`命令要求我们提供 JSON 或 YAML 文件格式的定义文件。本章的自动化脚本已将两个`crictl`定义文件添加到*/opt*目录。第一个文件，如清单 2-1 所示，创建一个 Pod：

*pod.yaml*

```
---
metadata:
  name: busybox
  namespace: crio
linux:
  security_context:
    namespace_options:
      network: 2
```

*清单 2-1：CRI-O Pod 定义*

与我们在 第一章 中看到的 Kubernetes Pod 类似，Pod 是一组在同一隔离空间中运行的一个或多个容器。在我们的案例中，我们只需要一个容器在 Pod 中，第二个文件，见 示例 2-2，定义了 CRI-O 应该启动的容器进程。我们提供一个名称（`busybox`）和命名空间（`crio`）来区分这个 Pod 和其他 Pod。否则，我们只需提供网络配置。CRI-O 期望使用容器网络接口（CNI）插件来配置网络命名空间。我们将在 第八章 中讨论 CNI 插件，因此现在我们将使用 `network: 2` 告诉 CRI-O 不要创建单独的网络命名空间，而是使用主机网络：

*container.yaml*

```
---
metadata:
  name: busybox
image:
  image: docker.io/library/busybox:latest
args:
  - "/bin/sleep"
  - "36000"
```

*示例 2-2：CRI-O 容器定义*

再次使用 BusyBox 是因为它体积小，运行快速且轻量。然而，由于 `crictl` 会在后台创建此容器而没有终端，我们需要指定 */bin/sleep* 作为容器内要运行的命令；否则，容器会立即终止，因为 shell 会发现它没有 TTY。

在运行容器之前，我们首先需要拉取镜像：

```
root@host01:~# crictl pull docker.io/library/busybox:latest
Image is up to date for docker.io/library/busybox@sha256:...
```

然后，我们将 *pod.yaml* 和 *container.yaml* 文件提供给 `crictl`，以创建并启动我们的 BusyBox 容器：

```
root@host01:~# cd /opt
root@host01:~# POD_ID=$(crictl runp pod.yaml)
root@host01:~# crictl pods
POD ID              CREATED                  STATE ...
3bf297ace44b5       Less than a second ago   Ready ...
root@host01:~# CONTAINER_ID=$(crictl create $POD_ID container.yaml pod.yaml)
root@host01:~# crictl start $CONTAINER_ID
91394a7f37e3da3a557782ed6d6eb2cf8c23e5b3dd4e2febd415bba071d10734
root@host01:~# crictl ps
CONTAINER           ... STATE
91394a7f37e3d       ... Running
```

我们捕获了 Pod 的唯一标识符和容器的标识符，分别保存在 `POD_ID` 和 `CONTAINER_ID` 变量中，以便在这里和接下来的命令中使用。

在查看 CRI-O 创建的 Linux 命名空间之前，让我们通过使用 `crictl exec` 命令在容器内部启动一个新的 shell 进程来查看 `busybox` 容器的内部：

```
root@host01:~# crictl exec -ti $CONTAINER_ID /bin/sh
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
...
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel qlen 1000
...
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel qlen 1000
...
/ # ps -ef
PID   USER     TIME  COMMAND
    1 root      0:00 /pause
    7 root      0:00 /bin/sleep 36000
    13 root      0:00 /bin/sh
    20 root      0:00 ps -ef
/ # exit
```

这个在 CRI-O 中运行的 BusyBox 容器与在 `containerd` 中运行的 BusyBox 看起来有些不同。首先，因为我们将 Pod 配置为 `network: 2`，所以容器可以看到与常规进程相同的网络设备。其次，我们看到了一些额外的进程。当我们在 第十二章讨论 Kubernetes 下的容器运行时时，我们会看到 PID 为 1 的 `pause` 进程。另一个额外的进程是 `sleep`，我们将其作为此容器的入口点。

CRI-O 也使用 Linux 命名空间来进行进程隔离，正如我们从检查容器进程和列出命名空间中看到的那样：

```
root@host01:~# PID=$(crictl inspect $CONTAINER_ID | jq '.info.pid')
root@host01:~# ps -ef | grep $PID | grep -v grep
root       23906   23894  0 20:15 ?        00:00:00 /bin/sleep 36000
root@host01:/opt# ps -ef | grep 23894 | grep -v grep
root       23894       1  0 20:15 ?        00:00:00 /usr/bin/conmon ...
root       23906   23894  0 20:15 ?        00:00:00 /bin/sleep 36000
```

`crictl inspect` 命令提供了大量关于容器的信息，但目前我们只需要 PID。由于 `crictl` 返回 JSON 格式的输出，我们可以使用 `jq` 从 `info` 结构中提取 `pid` 字段并将其保存到一个名为 `PID` 的环境变量中。尝试运行 crictl inspect $CONTAINER_ID 来查看完整信息。

使用我们发现的 PID，我们可以看到我们的`sleep`命令。然后，我们可以使用其父 PID 来验证它是由`conmon`（一个 CRI-O 工具）管理的。接下来，让我们看看 CRI-O 创建的命名空间。由于 CRI-O 中进程的命名空间分配更为复杂，我们将列出 Linux 系统上的所有命名空间，并挑选出与容器相关的命名空间：

```
root@host01:~# lsns
        NS TYPE   NPROCS   PID USER            COMMAND
...
4026532183 uts         2 23867 root            /pause
4026532184 ipc         2 23867 root            /pause
4026532185 mnt         1 23867 root            /pause
4026532186 pid         2 23867 root            /pause
4026532187 mnt         1 23906 root            /bin/sleep 36000
...
```

在这里，我们只看到四种类型的命名空间。因为我们告诉 CRI-O 允许容器访问主机的网络命名空间，所以它不需要创建`net`命名空间。此外，在 CRI-O 中，大多数命名空间与`pause`命令关联（尽管有些命名空间被多个进程共享，正如我们通过`NPROCS`列看到的）。有两个`mnt`命名空间，因为每个 Pod 中的独立容器会得到一组不同的挂载点，具体原因我们将在第五章中讨论。

### 在命名空间中直接运行进程

在容器中运行进程时，最棘手的任务之一是处理作为 PID 1 所带来的责任。为了更好地理解这一点，我们不会让容器运行时为我们创建命名空间。而是直接与 Linux 内核通信，手动在命名空间中运行进程。我们将使用命令行，虽然容器运行时使用 Linux 内核 API，但结果是相同的。

因为命名空间是 Linux 内核的特性，所以无需安装或配置其他内容。我们只需在启动进程时使用`unshare`命令：

```
root@host01:~# unshare -f -p --mount-proc -- /bin/sh -c /bin/bash
```

`unshare`命令在不同命名空间下运行一个程序。通过添加`-p`，我们指定需要一个新的 PID 命名空间。选项`--mount-proc`与此配合，添加一个新的挂载命名空间，并确保`/proc`被正确地重新挂载，以便进程看到正确的进程信息。否则，进程仍然可以看到系统中其他进程的信息。最后，`--`后面的内容指示要运行的命令。

因为这是一个隔离的进程命名空间，它无法看到该命名空间外的进程列表：

```
root@host01:~# ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 22:21 pts/0    00:00:00 /bin/sh -c /bin/bash
root           2       1  0 22:21 pts/0    00:00:00 /bin/bash
root           9       2  0 22:22 pts/0    00:00:00 ps -ef
```

获取这个命名空间的 ID，以便我们在列表中识别它：

```
root@host01:~# ls -l /proc/self/ns/pid
lrwxrwxrwx 1 root root 0 Mar  6 22:22 /proc/self/ns/pid -> 'pid:[4026532190]'
```

现在，从另一个终端窗口列出所有命名空间，并查找与我们的隔离 shell 相关的命名空间：

```
root@host01:~# lsns
        NS TYPE NPROCS PID   USER COMMAND
...
4026532189 mnt  3      12110 root unshare -f -p ...
4026532190 pid  2      12111 root /bin/sh -c /bin/bash
...
root@host01:~# exit
```

我们看到一个与之前看到的匹配的`pid`命名空间。此外，我们还看到了一个`mnt`命名空间。这个命名空间确保我们的 shell 看到`/proc`中的正确信息。

因为`pid`命名空间是由`sh`命令拥有的，当我们在命名空间内运行`ps`时，`sh`命令的 PID 为 1。这意味着`sh`负责正确管理其子进程（如`bash`）。例如，`sh`负责向其子进程发送信号，确保它们正确终止。记住这一点很重要，因为这是在运行容器时常见的问题，可能导致僵尸进程或清理已停止容器时的其他问题。

幸运的是，`sh` 很好地处理了它的管理任务，我们可以看到，当我们向它发送 `kill` 信号时，它会将该信号传递给它的子进程。从第二个终端窗口运行此命令，位于命名空间之外：

```
root@host01:~# kill -9 12111
```

在第一个窗口中，你会看到以下输出：

```
root@host01:~# Killed
```

这表明 `bash` 收到了 `kill` 信号并正确终止。

### 最后的思考

虽然容器创建了一个完全独立的系统的表象，但其实现方式与虚拟机完全不同。相反，这个过程类似于传统的进程隔离方式，例如用户权限和独立的文件系统。容器运行时使用命名空间，这是 Linux 内核内置的功能，可实现各种类型的进程隔离。在本章中，我们研究了 `containerd` 和 CRI-O 容器运行时如何使用多种类型的 Linux 命名空间，为每个容器提供对其他进程、网络设备和文件系统的独立视图。命名空间的使用防止了容器中的进程看到并干扰其他进程。

同时，容器中的进程仍然共享相同的 CPU、内存和网络。一个使用过多资源的进程会阻止其他进程正常运行。然而，命名空间无法解决这个问题。为了防止这个问题，我们需要关注资源限制——这是下一章的主题。
