# 第三章：资源限制

![image](img/common01.jpg)

我们在第二章中做的进程隔离工作非常重要，因为一个进程通常无法影响它看不见的东西。然而，我们的进程可以看到主机的 CPU、内存和网络，因此，进程有可能通过过度使用这些资源，导致其他进程无法正确运行，无法为其他进程留下足够的空间。在本章中，我们将看到如何保证进程仅使用其分配的 CPU、内存和网络资源，从而确保我们可以准确划分资源。这将在我们进行容器编排时有所帮助，因为它将为 Kubernetes 提供有关每个主机可用资源的确定性，从而在调度容器时进行决策。

CPU、内存和网络是重要的，但还有一个非常重要的共享资源：存储。然而，在像 Kubernetes 这样的容器编排环境中，存储是分布式的，限制需要在整个集群层面应用。因此，我们对存储的讨论必须等到我们在第十五章引入分布式存储时才开始。

### CPU 优先级

我们需要分别查看 CPU、内存和网络，因为应用限制的效果在每种情况下不同。让我们首先看看如何控制 CPU 使用。为了理解 CPU 限制，我们首先需要了解 Linux 内核是如何决定运行哪个进程以及运行多长时间的。在 Linux 内核中，*调度器*会维护一个所有进程的列表。它还会追踪哪些进程准备好运行，以及每个进程最近运行了多长时间。这使得它能够创建一个优先级列表，从而选择下一个要运行的进程。调度器的设计尽量公平（它被称为完全公平调度器）；因此，它会尽力给所有进程提供运行的机会。然而，它也接受外部输入，决定哪些进程比其他进程更为重要。这个优先级划分由两个部分组成：调度策略，以及在该策略下每个进程的优先级。

#### 实时和非实时策略

调度器支持几种不同的策略，但就我们的目的而言，我们可以将它们分为实时策略和非实时策略。术语*实时*意味着某些现实世界的事件对进程至关重要，并且需要在特定的最后期限前完成处理。如果进程在最后期限过后还没有完成处理，就会发生不良后果。例如，进程可能在从嵌入式硬件设备收集数据。在这种情况下，进程必须在硬件缓冲区溢出之前读取数据。实时进程通常不会非常占用 CPU，但当它需要 CPU 时，不能等待，因此所有处于实时策略下的进程都比任何处于非实时策略下的进程优先级更高。让我们通过一个示例 Linux 系统来探讨这个问题。

**注意**

*本书的示例仓库位于* [`github.com/book-of-kubernetes/examples`](https://github.com/book-of-kubernetes/examples)。 *有关设置的详细信息，请参见第 xx 页的“运行示例”部分。*

Linux 的`ps`命令告诉我们每个进程适用的具体策略。在*host01*上运行此命令，以查看本章示例：

```
root@host01:~# ps -e -o pid,class,rtprio,ni,comm
 PID CLS RTPRIO  NI COMMAND
   1 TS       -   0 systemd
...
   6 TS       - -20 kworker/0:0H-kblockd
...
  11 FF      99   - migration/0
  12 FF      50   - idle_inject/0
...
  85 FF      99   - watchdogd
...
 484 RR      99   - multipathd
...
7967 TS       -   0 ps
```

`-o`标志为`ps`提供了一个自定义的输出字段列表，包括调度策略*class*（`CLS`）和两个数字优先级字段：`RTPRIO`和`NI`。

首先查看`CLS`字段，许多进程列出为`TS`，表示“时间共享”，这是默认的非实时策略。这包括我们自己运行的命令（例如我们运行的`ps`命令）以及重要的 Linux 系统进程，如`systemd`。然而，我们也看到具有`FF`策略（先进先出，FIFO）和`RR`策略（轮转）的进程。这些是实时进程，因此它们的优先级高于系统中所有非实时策略的进程。列表中的实时进程包括`watchdog`（用于检测系统死锁，因此可能需要抢占其他进程）和`multipathd`（用于监视设备更改，并且必须在其他进程有机会与设备通信之前配置设备）。

除了类之外，两个数字优先级字段还告诉我们进程在策略中的优先级。不出所料，`RTPRIO`字段表示“实时优先级”，仅适用于实时进程。`NI`字段是进程的“nice”级别，仅适用于非实时进程。由于历史原因，nice 级别从-20（最不友好，或最高优先级）到 19（最友好，最低优先级）。

#### 设置进程优先级

Linux 允许我们为启动的进程设置优先级。我们来尝试通过优先级控制 CPU 使用。我们将运行一个名为`stress`的程序，它旨在对我们的系统进行压力测试。我们将使用基于 CRI-O 的容器化版本的`stress`。

如之前所述，我们需要为 Pod 和容器定义 YAML 文件，以告诉`crictl`该运行什么。清单 3-1 中显示的 Pod YAML 与第二章中的 BusyBox 示例几乎相同，唯一不同的是名称：

*po-nolim.yaml*

```
---
metadata:
  name: stress
  namespace: crio
linux:
  security_context:
    namespace_options:
      network: 2
```

*清单 3-1：BusyBox Pod*

容器的 YAML 相比于 BusyBox 示例有更多的更改。除了使用不同的容器镜像，即已经安装了`stress`的镜像外，我们还需要向`stress`提供参数，告诉它只使用一个 CPU：

*co-nolim.yaml*

```
---
metadata:
  name: stress
image:
  image: docker.io/bookofkubernetes/stress:stable
args:
  - "--cpu"
  - "1"
  - "-v"
```

`host01`上已安装 CRI-O，因此只需几条命令即可启动这个容器。首先，我们将拉取镜像：

```
root@host01:/opt# crictl pull docker.io/bookofkubernetes/stress:stable
Image is up to date for docker.io/bookofkubernetes/stress...
```

然后，我们可以从镜像运行一个容器：

```
root@host01:~# cd /opt
root@host01:/opt# PUL_ID=$(crictl runp po-nolim.yaml)
root@host01:/opt# CUL_ID=$(crictl create $PUL_ID co-nolim.yaml po-nolim.yaml)
root@host01:/opt# crictl start $CUL_ID
...
root@host01:/opt# crictl ps
CONTAINER      IMAGE                                    ...
971e83927329e  docker.io/bookofkubernetes/stress:stable ...
```

`crictl ps`命令只是用来检查我们的容器是否按预期运行。

现在，`stress`程序已在我们的系统上运行，我们可以查看当前的优先级和 CPU 使用情况。我们想查看当前的 CPU 使用情况，因此我们将使用`top`：

```
root@host01:/opt# top -b -n 1 -p $(pgrep -d , stress)
top - 18:01:58 up  1:39,  1 user,  load average: 1.01, 0.40, 0.16
Tasks:   2 total,   1 running,   1 sleeping,   0 stopped,   0 zombie
%Cpu(s): 34.8 us, 0.0 sy, 0.0 ni, 65.2 id, 0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   1987.5 total,   1024.5 free,    195.8 used,    767.3 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.   1643.7 avail Mem 

  PID   USER  PR  NI  ...  %CPU  %MEM    TIME+ COMMAND
  13459 root  20   0  ... 100.0   0.2  0:29.78 stress-ng
  13435 root  20   0  ...   0.0   0.2  0:00.01 stress-ng
```

`pgrep`命令查找了`stress`的进程 ID（PID）；有两个 PID，因为`stress`为我们请求的 CPU 负载操作创建了一个独立的进程。这个 CPU 工作进程占用了一个 CPU 的 100%；幸运的是，我们的虚拟机有两个 CPU，所以它并没有超载。

我们以默认优先级启动了这个进程，因此它的 nice 值为`0`，如`NI`列所示。如果我们改变这个优先级会发生什么呢？让我们使用`renice`来找出答案：

```
root@host01:/opt# renice -n 19 -p $(pgrep -d ' ' stress)
13435 (process ID) old priority 0, new priority 19
13459 (process ID) old priority 0, new priority 19
```

之前使用的`ps`命令期望 PID 通过逗号分隔，而`renice`命令期望 PID 通过空格分隔；幸运的是，`pgrep`可以同时处理这两种情况。

我们已经成功地改变了进程的优先级：

```
root@host01:/opt# top -b -n 1 -p $(pgrep -d , stress)
top - 18:11:04 up  1:48,  1 user,  load average: 1.07, 0.95, 0.57
Tasks:   2 total,   1 running,   1 sleeping,   0 stopped,   0 zombie
%Cpu(s): 0.0 us, 0.0 sy, 28.6 ni, 71.4 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   1987.5 total,   1035.6 free,    182.2 used,    769.7 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.   1657.2 avail Mem 

  PID   USER  PR  NI  ...  %CPU  %MEM     TIME+ COMMAND
  13459 root  39  19  ... 100.0   0.2   9:35.50 stress-ng
  13435 root  39  19  ...   0.0   0.2   0:00.01 stress-ng
```

新的 nice 值是`19`，意味着我们的进程比之前的优先级低。然而，`stress`程序仍然占用了一个 CPU 的 100%！这是怎么回事？问题在于优先级只是一个相对的度量。如果没有其他程序需要 CPU（在这种情况下是这样），即使是低优先级的进程也可以尽可能多地使用 CPU。

这种安排可能看起来是我们想要的。毕竟，如果 CPU 可用，我们是不是希望我们的应用组件能够使用它？不幸的是，尽管这听起来很合理，但由于两个主要原因，它并不适合我们的容器化应用。首先，像 Kubernetes 这样的容器编排环境在容器能够被分配到任何具有足够资源的主机上时效果最佳。我们不可能了解 Kubernetes 集群中每个容器的相对优先级，特别是当我们考虑到一个 Kubernetes 集群可能是*多租户*的，即多个独立的应用或团队可能在同一个集群中使用时。第二，Kubernetes 如果没有某个容器将使用多少 CPU 的概念，就无法知道哪些主机已经满载，哪些主机还有空余空间。如果多个容器在同一台主机上同时变得繁忙，它们将争夺可用的 CPU 核心，整个主机将变慢，这是我们不希望发生的情况。

### Linux 控制组

正如我们在上一节看到的，进程优先级调整不会帮助像 Kubernetes 这样的容器编排环境了解在调度新容器时应该使用哪个主机，因为即使是低优先级进程在 CPU 空闲时也能获得大量的 CPU 时间。而且由于我们的 Kubernetes 集群可能是多租户的，集群不能仅仅依赖每个容器承诺只使用一定量的 CPU。首先，这样可能会导致一个进程负面影响到另一个进程，无论是恶意的还是意外的。其次，进程并不真正控制自己的调度；它们在 Linux 内核决定分配 CPU 时间时才获得 CPU 时间。我们需要一种不同的解决方案来控制 CPU 的使用。

为了找到答案，我们可以采用实时处理所使用的一种方法。正如我们在前一部分提到的，实时进程通常不需要大量计算，但当它需要 CPU 时，它需要立即获取。为了确保所有实时进程都能获得它们需要的 CPU，通常会为每个进程保留一部分 CPU 时间。即使我们的容器进程不是实时的，我们也可以使用相同的策略。如果我们能配置容器，使其只能使用分配的 CPU 时间片，Kubernetes 将能够计算每个主机上可用的空间，并能够将容器调度到有足够空间的主机上。

为了管理容器对 CPU 核心的使用，我们将使用 *控制组*。控制组（cgroups）是 Linux 内核的一个特性，用于管理进程资源的使用。每种资源类型，如 CPU、内存或块设备，都可以有一个与之关联的 cgroup 层级结构。进程进入 cgroup 后，内核会自动应用该组的控制。

cgroup 的创建和配置是通过一种特定的文件系统处理的，类似于 Linux 通过 */proc* 文件系统报告系统信息的方式。默认情况下，cgroup 的文件系统位于 */sys/fs/cgroup*：

```
root@host01:~# ls /sys/fs/cgroup
blkio        cpuacct  freezer  net_cls           perf_event  systemd
cpu          cpuset   hugetlb  net_cls,net_prio  pids        unified
cpu,cpuacct  devices  memory   net_prio          rdma
```

*/sys/fs/cgroup* 中的每一项都是可以限制的不同资源。如果我们查看其中一个目录，我们可以开始看到可以应用的控制。例如，对于*cpu*：

```
root@host01:~# cd /sys/fs/cgroup/cpu
root@host01:/sys/fs/cgroup/cpu# ls -F
cgroup.clone_children  cpuacct.stat               cpuacct.usage_user
cgroup.procs           cpuacct.usage              init.scope/
cgroup.sane_behavior   cpuacct.usage_all          notify_on_release
cpu.cfs_period_us      cpuacct.usage_percpu       release_agent
cpu.cfs_quota_us       cpuacct.usage_percpu_sys   system.slice/
cpu.shares             cpuacct.usage_percpu_user  tasks
cpu.stat               cpuacct.usage_sys          user.slice/
```

`ls` 命令上的 `-F` 标志会为目录添加斜杠字符，这使我们可以开始看到层级结构。每个子目录（*init.scope*、*system.slice* 和 *user.slice*）都是一个独立的 CPU cgroup，每个都有一组适用于该 cgroup 中进程的配置文件。

#### 使用 cgroups 的 CPU 配额

为了理解这个目录的内容，让我们看看如何使用 cgroups 来限制 `stress` 容器的 CPU 使用情况。我们将重新检查它的 CPU 使用情况：

```
root@host01:/sys/fs/cgroup/cpu# top -b -n 1 -p $(pgrep -d , stress)
top - 22:40:12 up 12 min,  1 user,  load average: 0.81, 0.35, 0.21
Tasks:   2 total,   1 running,   1 sleeping,   0 stopped,   0 zombie
%Cpu(s): 37.0 us, 0.0 sy, 0.0 ni, 63.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   1987.5 total,   1075.1 free,    179.4 used,    733.0 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.   1646.3 avail Mem 

  PID USER   PR  NI ...  %CPU  %MEM     TIME+ COMMAND
  5964 root  20  19 ...  100.0  0.2   1:19.72 stress-ng
  5932 root  20  19 ...  0.0    0.2   0:00.02 stress-ng
```

如果你仍然没有看到 `stress` 正在运行，使用本章前面提到的命令重新启动它。接下来，让我们探索 `stress` CPU 进程所在的 CPU cgroup。我们可以通过在 */sys/fs/cgroup/cpu* 层级中的文件内查找其 PID 来做到这一点：

```
root@host01:/sys/fs/cgroup/cpu# grep -R $(pgrep stress-ng-cpu)
system.slice/runc-050c.../cgroup.procs:5964
system.slice/runc-050c.../tasks:5964
```

`stress` 进程属于 *system.slice* 层级，并位于由 `runc` 创建的子目录中，`runc` 是 CRI-O 的内部组件之一。这非常方便，因为这意味着我们不需要创建自己的 cgroup 并将此进程移入其中。这也不是偶然的；正如我们稍后将看到的，CRI-O 支持对容器设置 CPU 限制，因此它自然需要为每个运行的容器创建一个 cgroup。实际上，cgroup 的名称是以容器 ID 命名的。

让我们进入容器 cgroup 的目录：

```
root@host01:/sys/fs/cgroup/cpu# cd system.slice/runc-${CUL_ID}.scope
```

我们使用之前保存的容器 ID 变量进入适当的目录。一旦进入该目录，我们可以看到它具有与*/sys/fs/cgroup/cpu*根目录相同的配置文件：

```
root@host01:/sys/fs/...07.scope# ls
cgroup.clone_children  cpu.uclamp.max        cpuacct.usage_percpu_sys
cgroup.procs           cpu.uclamp.min        cpuacct.usage_percpu_user
cpu.cfs_period_us      cpuacct.stat          cpuacct.usage_sys
cpu.cfs_quota_us       cpuacct.usage         cpuacct.usage_user
cpu.shares             cpuacct.usage_all     notify_on_release
cpu.stat               cpuacct.usage_percpu  tasks
```

*cgroup.procs*文件列出了这个控制组中的进程：

```
root@host01:/sys/fs/...07.scope# cat cgroup.procs
5932
5964
```

这个目录还有许多其他文件，但我们主要关心三个文件：

***cpu.shares*** 这个 cgroup 相对于同级 cgroup 所占的 CPU 份额

***cpu.cfs_period_us*** 一个周期的长度，以微秒为单位

***cpu.cfs_quota_us*** 一个周期内的 CPU 时间，以微秒为单位

我们将查看 Kubernetes 如何在第十四章中使用*cpu.shares*。现在，我们需要一种方法来控制我们的实例，避免它对系统造成过载。为此，我们将为这个容器设置一个绝对配额。首先，让我们查看*cpu.cfs_period_us*的值：

```
root@host01:/sys/fs/...07.scope# cat cpu.cfs_period_us
100000
```

该周期设置为 100,000 微秒，或者 0.1 秒。我们可以利用这个数字来计算应该设置什么样的配额，以限制`stress`容器能使用的 CPU 量。目前，没有设置配额：

```
root@host01:/sys/fs/...07.scope# cat cpu.cfs_quota_us
-1
```

我们只需更新*cpu.cfs_quota_us*文件即可设置配额：

```
root@host01:/sys/fs/...07.scope# echo "50000" > cpu.cfs_quota_us
```

这为该 cgroup 中的进程提供了每 100,000 微秒 50,000 微秒的 CPU 时间，平均分配为 50%的 CPU。进程会立即受到影响，正如我们可以确认的那样：

```
root@host01:/sys/fs/...07.scope# top -b -n 1 -p $(pgrep -d , stress)
top - 23:53:05 up  1:24,  1 user,  load average: 0.71, 0.93, 0.98
Tasks:   2 total,   1 running,   1 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us, 3.6 sy, 7.1 ni, 89.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   1987.5 total,   1064.9 free,    174.6 used,    748.0 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.   1663.9 avail Mem 

  PID USER   PR  NI  ...  %CPU  %MEM     TIME+ COMMAND
  5964 root  39  19  ...  50.0   0.2  73:45.68 stress-ng-cpu
  5932 root  39  19  ...   0.0   0.2   0:00.02 stress-ng
```

你的清单可能不会显示出精确的 50% CPU 使用率，因为`top`命令测量 CPU 使用情况的周期可能与内核的调度周期不完全对齐。但平均来看，我们的`stress`容器现在无法使用超过 50%的单个 CPU。

在继续之前，让我们先停止`stress`容器：

```
root@host01:/sys/fs/...07.scope# cd
root@host01:/opt# crictl stop $CUL_ID
...
root@host01:/opt# crictl rm $CUL_ID
...
root@host01:/opt# crictl stopp $PUL_ID
Stopped sandbox ...
root@host01:/opt# crictl rmp $PUL_ID
Removed sandbox ...
```

#### 使用 CRP-O 和 crictl 设置 CPU 配额

如果每次都需要在文件系统中找到 cgroup 位置并更新每个容器的 CPU 配额来控制 CPU 使用，这将是一件繁琐的事情。幸运的是，我们可以在`crictl`的 YAML 文件中指定配额，CRI-O 会为我们强制执行。让我们看看一个安装在*/opt*中的例子，当我们设置这个虚拟机时，配置也已安装。

Pod 配置与清单 3-1 只有略微的不同。我们添加了`cgroup_parent`设置，这样可以控制 CRI-O 创建 cgroup 的位置，这将使我们更容易找到 cgroup 并查看其配置：

*po-clim.yaml*

```
---
metadata:
  name: stress-clim
  namespace: crio
linux:
  cgroup_parent: pod.slice
  security_context:
    namespace_options:
      network: 2
```

容器配置是我们包含 CPU 限制的地方。我们的`stress1`容器将只分配 10%的 CPU：

*co-clim.yaml*

```
---
---
metadata:
  name: stress-clim
image:
  image: docker.io/bookofkubernetes/stress:stable
args:
  - "--cpu"
  - "1"
  - "-v"
linux:
  resources:
    cpu_period: 100000
    cpu_quota: 10000
```

`cpu_period`的值对应于文件*cpu.cfs_period_us*，并提供配额适用的周期长度。`cpu_quota`的值对应于文件*cpu.cfs_quota_us*。通过将配额除以周期，我们可以确定这将设置一个 10%的 CPU 限制。现在，让我们启动这个带有 CPU 限制的`stress`容器：

```
root@host01:~# cd /opt
root@host01:/opt# PCL_ID=$(crictl runp po-clim.yaml)
root@host01:/opt# CCL_ID=$(crictl create $PCL_ID co-clim.yaml po-clim.yaml)
root@host01:/opt# crictl start $CCL_ID
...
root@host01:/opt# crictl ps
CONTAINER      IMAGE                                    ...
ea8bccd711b86  docker.io/bookofkubernetes/stress:stable ...
```

我们的容器立即被限制为 10%的 CPU 使用：

```
root@host01:/opt# top -b -n 1 -p $(pgrep -d , stress)
top - 17:26:55 up 19 min,  1 user,  load average: 0.27, 0.16, 0.13
Tasks:   4 total,   2 running,   2 sleeping,   0 stopped,   0 zombie
%Cpu(s): 10.3 us, 0.0 sy, 0.0 ni, 89.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   1987.5 total,   1053.4 free,    189.3 used,    744.9 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.   1640.4 avail Mem 

  PID USER   PR  NI ... %CPU  %MEM     TIME+ COMMAND
  8349 root  20   0 ... 10.0   0.2   0:22.67 stress-ng
  8202 root  20   0 ...  0.0   0.2   0:00.02 stress-ng
```

如同我们之前的例子所示，显示的 CPU 使用率是在 `top` 运行期间的快照，因此可能不会完全匹配限制，但从长远来看，这个进程不会超过其分配的 CPU 使用量。

我们可以检查 cgroup 来确认 CRI-O 是否将其放置在我们指定的位置，并自动配置了 CPU 配额：

```
root@host01:/opt# cd /sys/fs/cgroup/cpu/pod.slice
root@host01:...pod.slice# cat crio-$CCL_ID.scope/cpu.cfs_quota_us
10000
```

CRI-O 为我们的容器创建了一个新的 cgroup 父级 *pod.slice*，在其中为容器创建了一个特定的 cgroup，并配置了它的 CPU 配额，而我们无需动手。

我们不再需要这个容器了，所以让我们把它移除：

```
root@host01:/sys/fs/cgroupcpu/pod.slice# cd
root@host01:~# crictl stop $CCL_ID
...
root@host01:~# crictl rm $CCL_ID
...
root@host01:~# crictl stopp $PCL_ID
Stopped sandbox ...
root@host01:~# crictl rmp $PCL_ID
Removed sandbox ...
```

使用这些命令，我们先停止容器，再删除容器，最后删除 Pod。

### 内存限制

内存是进程的另一个重要资源。如果系统没有足够的内存来满足请求，内存分配将失败。这通常会导致进程出现异常行为或完全失败。当然，大多数 Linux 系统使用 *交换空间* 将内存内容暂时写入磁盘，这使得系统内存看起来比实际更大，但也会降低系统性能。这个问题足够重要，以至于 Kubernetes 团队不鼓励在集群中启用交换空间。

此外，即使我们能够使用交换空间，我们也不希望某个进程占用所有常驻内存，从而使其他进程变得非常缓慢。因此，我们需要限制进程的内存使用，以便它们能够相互协作。我们还需要为内存使用设置一个明确的最大值，以便 Kubernetes 可以可靠地确保主机在调度新容器到主机之前有足够的可用内存。

Linux 系统与其他 Unix 变种一样，传统上需要处理多个共享稀缺资源的用户。因此，内核支持对系统资源的限制，包括 CPU、内存、子进程数量和打开文件数。我们可以通过命令行使用 `ulimit` 命令来设置这些限制。例如，一种限制类型是“虚拟内存”限制。它不仅包括进程在常驻内存中使用的 RAM，还包括它使用的任何交换空间。以下是一个限制虚拟内存的 `ulimit` 命令示例：

```
root@host01:~# ulimit -v 262144
```

`-v` 开关指定了虚拟内存的限制。参数以字节为单位，因此 262144 将对我们从这个 shell 会话启动的每个附加进程设置一个 256MiB 的虚拟内存限制。设置虚拟内存限制是一个总的限制；它可以确保进程不能通过交换空间绕过限制。我们可以通过将一些数据加载到内存中来验证限制是否已应用：

```
root@host01:~# cat /dev/zero | head -c 500m | tail
tail: memory exhausted
```

这个命令从 */dev/zero* 中读取数据，并尝试将它找到的前 500MiB 零字节保持在内存中。然而，当 `tail` 命令尝试分配更多空间来存放从 `head` 获取的零字节时，它因为达到限制而失败。

因此，Unix 限制使我们能够控制进程的内存使用，但由于一些原因，它们无法提供容器所需的所有功能。首先，Unix 限制只能应用于单个进程或整个用户。这两者都不能满足我们的需求，因为容器实际上是一个*进程组*。容器的初始进程可能会创建许多子进程，并且容器中的所有进程都需要在相同的限制下运行。同时，将限制应用于整个用户并不能真正帮助我们在像 Kubernetes 这样的容器编排环境中，因为从操作系统的角度来看，所有容器都属于同一个用户。其次，关于 CPU 限制，常规的 Unix 限制唯一能做的就是限制进程在被终止之前获得的最大 CPU 时间。这不是我们在共享 CPU 给长时间运行的进程时所需要的限制类型。

我们将不再使用传统的 Unix 限制，而是再次使用 cgroups，这次是为了限制进程可用的内存。我们将使用相同的 `stress` 容器镜像，这次包含一个尝试分配大量内存的子进程。

如果我们在启动 `stress` 容器后尝试应用内存限制，我们会发现内核不允许这么做，因为它已经占用了过多内存。因此，我们将立即在 YAML 配置中应用它。和之前一样，我们需要一个 Pod：

*po-mlim.yaml*

```
---
metadata:
  name: stress2
  namespace: crio
linux:
  cgroup_parent: pod.slice
  security_context:
    namespace_options:
      network: 2
```

这与我们用于 CPU 限制的 Pod 相同，但为了避免冲突，名称不同。就像我们之前做的那样，我们要求 CRI-O 将 cgroup 放入*pod.slice*，这样我们就可以轻松找到它。

我们还需要一个容器定义：

*co-mlim.yaml*

```
 ---
 ---
 metadata:
   name: stress2
 image:
   image: docker.io/bookofkubernetes/stress:stable
 args:
   - "--vm"
   - "1"
   - "--vm-bytes"
➊ - "512M"
   - "-v"
 linux:
   resources:
  ➋ memory_limit_in_bytes: 268435456
     cpu_period: 100000 
  ➌ cpu_quota: 10000
```

新的资源限制是 `memory_limit_in_bytes`，我们将其设置为 256MiB ➋。我们保持 CPU 配额 ➌，因为持续尝试分配内存将消耗大量 CPU。最后，在 `args` 部分，我们告诉 `stress` 尝试分配 512MB 的内存 ➊。

我们可以使用与之前相同的 `crictl` 命令运行它：

```
root@host01:~# cd /opt 
root@host01:/opt# PML_ID=$(crictl runp po-mlim.yaml)
root@host01:/opt# CML_ID=$(crictl create $PML_ID co-mlim.yaml po-mlim.yaml)
root@host01:/opt# crictl start $CML_ID
...
```

如果我们告诉 `crictl` 列出容器，所有情况看起来都正常：

```
root@host01:/opt# crictl ps
CONTAINER     IMAGE                                    ... STATE   ...
31025f098a6c9 docker.io/bookofkubernetes/stress:stable ... Running ...
```

这表明容器处于 `Running` 状态。然而，在背后，`stress` 正在努力分配内存。如果我们打印出来自 `stress` 容器的日志信息，我们就可以看到这一点：

```
root@host01:/opt# crictl logs $CML_ID
...
stress-ng: info:  [6] dispatching hogs: 1 vm
...
stress-ng: debug: [11] stress-ng-vm: started [11] (instance 0)
stress-ng: debug: [11] stress-ng-vm using method 'all'
stress-ng: debug: [11] stress-ng-vm: child died: signal 9 'SIGKILL' (instance 0)
stress-ng: debug: [11] stress-ng-vm: assuming killed by OOM killer, restarting again...
stress-ng: debug: [11] stress-ng-vm: child died: signal 9 'SIGKILL' (instance 0)
stress-ng: debug: [11] stress-ng-vm: assuming killed by OOM killer, restarting again...
```

Stress 报告说其内存分配进程正在被 “内存不足” 持续终止。

我们可以看到内核报告显示 `oom_reaper` 确实是导致进程被终止的原因：

```
root@host01:/opt# dmesg | grep -i oom_reaper | tail -n 1
[  696.651056] oom_reaper: reaped process 8756 (stress-ng-vm)...
```

`OOM killer` 是 Linux 在整个系统内存不足时使用的功能，它需要终止一个或多个进程来保护系统。在这种情况下，它通过发送 `SIGKILL` 信号终止进程，以确保 cgroup 在其内存限制下。`SIGKILL` 是一种信号，通知进程立即终止，且不进行任何清理。

**为什么使用 OOM Killer？**

当我们使用常规限制来控制内存时，超出限制会导致内存分配失败，但内核不会使用 OOM 杀手来终止我们的进程。为什么会有这种差异？答案在于容器的本质。当我们设计使用容器化微服务的可靠系统时，我们会发现，容器应该是快速启动和快速扩展的。这意味着应用中的每个单独容器本质上并不太重要。这也意味着，一个容器可能会被意外终止，通常不会引发太大关注。再加上不检查内存分配错误是最常见的 bug 之一，因此直接终止进程被认为是更安全的做法。

话虽如此，值得注意的是，确实可以为某个 cgroup 关闭 OOM 杀手。然而，与其让内存分配失败，效果是将进程暂停，直到该组中的其他进程释放内存。实际上，这样更糟，因为现在我们有一个既没有被正式终止，又没有执行任何有用操作的进程。

在继续之前，让我们先把这个不断失败的`stress`容器解脱出来：

```
root@host01:/opt# crictl stop $CML_ID
...
root@host01:/opt# crictl rm $CML_ID
...
root@host01:/opt# crictl stopp $PML_ID
Stopped sandbox ...
root@host01:/opt# crictl rmp $PML_ID
Removed sandbox ...
root@host01:/opt# cd
```

停止并移除容器和 Pod 可以防止`stress`容器浪费 CPU，不断尝试重启内存分配过程。

### 网络带宽限制

在本章中，我们从易于限制的资源转向了更难限制的资源。我们从 CPU 开始，内核完全负责哪个进程获得 CPU 时间以及在被抢占之前能获得多少时间。接着我们看了内存，内核没有能力强制进程放弃内存，但至少内核可以控制内存分配是否成功，或者它可以终止请求过多内存的进程。

现在我们开始讨论网络带宽，控制网络带宽比控制 CPU 或内存更为困难，原因有两个。首先，网络设备不像 CPU 或内存那样可以“合并”，因此我们需要在每个独立的网络设备层面上进行限制。其次，我们的系统实际上无法控制通过网络发送给它的数据；我们只能完全控制*出口*带宽，即通过特定网络设备发送的流量。

**正确的网络管理**

要实现一个完全可靠的集群，仅仅控制出站流量显然是不够的。一个下载大文件的进程将和一个上传大量数据的进程一样占用可用带宽。然而，我们实际上无法控制通过特定网络接口进入我们主机的流量，至少在主机层面上是无法控制的。如果我们真的想要管理网络带宽，我们需要在交换机或路由器上处理这类问题。例如，将物理网络划分为虚拟局域网（VLAN）是很常见的做法。一个 VLAN 可能是用于审计、日志记录以及供管理员登录使用的管理网络。我们还可能为重要的容器流量预留另一个 VLAN，或者使用流量整形确保重要数据包能够通过。只要我们在交换机上执行这种配置，通常可以允许剩余带宽以“最佳努力”方式传输。

虽然 Linux 确实为网络接口提供了一些 cgroup 功能，但这些仅有助于我们优先处理和分类网络流量。因此，与其使用 cgroups 来控制出站流量，我们将直接配置 Linux 内核的*流量控制*功能。我们将使用`iperf3`来测试网络性能，应用出站流量限制，然后再次进行测试。在本章的示例中，具有 IP 地址`192.168.61.12`的*host02*已自动设置并运行`iperf3`服务器，以便我们可以从*host01*向其发送数据。

让我们首先查看在没有限制的接口上可以获得的出站带宽：

```
root@host01:~# iperf3 -c 192.168.61.12
Connecting to host 192.168.61.12, port 5201
[  5] local 192.168.61.11 port 49044 connected to 192.168.61.12 port 5201
...
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec  2.18 GBytes  1.87 Gbits/sec  13184             sender
[  5]   0.00-10.00  sec  2.18 GBytes  1.87 Gbits/sec                  receiver
...
```

这个例子展示了千兆网速。根据你运行示例的方式，你可能会看到更低或更高的数值。现在我们有了基准，我们可以使用`tc`设置一个出站流量配额。你需要选择一个符合你带宽的配额；最有可能的是，设置 100Mb 的上限将会有效：

```
root@host01:~# IFACE=$(ip -o addr | grep 192.168.61.11 | awk '{print $2}')
root@host01:~# tc qdisc add dev $IFACE root tbf rate 100mbit \
  burst 256kbit latency 400ms
```

网络接口的名称在不同的系统上可能不同，因此我们使用`ip addr`来确定我们要控制的接口。然后，我们使用`tc`来实际应用限制。命令中的`token tbf`代表*令牌桶过滤器*。使用令牌桶过滤器时，每个数据包都会消耗令牌。桶会随着时间的推移不断地重新填充令牌，但如果桶在任何时候为空，数据包会被排队，直到有令牌可用。通过控制桶的大小和桶填充的速率，内核能够轻松地设置带宽限制。

现在我们已经对这个接口应用了限制，让我们通过再次运行完全相同的`iperf3`命令来查看其效果：

```
root@host01:~# iperf3 -c 192.168.61.12
Connecting to host 192.168.61.12, port 5201
[  5] local 192.168.61.11 port 49048 connected to 192.168.61.12 port 5201
...
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec   114 MBytes  95.7 Mbits/sec    0             sender
[  5]   0.00-10.01  sec   113 MBytes  94.5 Mbits/sec                  receiver
...
```

如预期所示，我们现在在这个接口上限速到 100Mbps。

当然，在这种情况下，我们限制了系统上每个人使用的网络接口的带宽。要正确使用这种功能来控制带宽使用，我们需要更精确地设定限制。然而，为了做到这一点，我们需要将一个进程隔离到其自己的一组网络接口中，这将是下一章的主题。

### 总结思考

确保一个进程不会给系统上的其他进程带来问题，包括确保它公平共享 CPU、内存和网络带宽等系统资源。在本章中，我们看到 Linux 提供了控制组（cgroups）来管理 CPU 和内存限制，以及管理网络接口的流量控制能力。当我们创建一个 Kubernetes 集群并部署容器到其中时，我们将看到 Kubernetes 如何利用这些底层 Linux 内核功能来确保容器被调度到具有足够资源的主机上，并确保这些主机上的容器行为良好。

我们已经介绍了容器运行时提供的一些最重要的进程隔离元素，但还有两种隔离类型我们尚未探讨：网络隔离和存储隔离。在下一章中，我们将看看 Linux 网络命名空间是如何被用来让每个容器看起来拥有自己的一组网络接口，包括独立的 IP 地址和端口。我们还将探讨这些单独容器接口的流量如何在我们的系统中流动，以便容器之间可以互相通信并与网络的其余部分进行通信。
