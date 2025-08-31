## 第六章：KERNEL GAMES**

![image](img/common01.jpg)

如果你是 Unix 管理的新手，*内核*这个词可能会让你感到害怕。毕竟，内核是计算机中的一些神秘部分，普通人不应该轻易触碰。在某些版本的 Unix 中，篡改内核是不可想象的。微软甚至不宣传它的操作系统有内核，这就像忽略了人类有大脑这一事实一样。^(1) 虽然高级用户可以通过各种方法访问内核，但这并不被广泛承认或鼓励。然而，在许多开源类 Unix 的世界中，干预内核是改变系统行为的一种非常可行且预期的方式。如果你被允许这样做，这可能也是调整其他操作系统的一个极好方式。

FreeBSD 的内核可以动态调节或随时更改，大多数系统性能的方面可以根据需要进行调整。我们将讨论内核的 sysctl 接口，以及如何使用它来更改正在运行的内核。

同时，内核的某些部分只能在系统启动的早期阶段进行修改。引导加载程序允许你在主机甚至没有找到其文件系统之前调整内核。

一些内核特性需要大量的重新配置。你可以为非常小的系统定制内核，或者为你正在运行的硬件精确调整一个内核。做到这一点的最佳方法是自己构建内核。

FreeBSD 有一个模块化的内核，这意味着可以将整个内核部分加载或卸载，从操作系统中打开或关闭整个子系统。这在如今可拆卸硬件时代非常有用，例如 PC 卡和 USB 设备。可加载的内核模块会影响性能、系统行为和硬件支持。

最后，我们将讨论如何调试你的内核，包括一些看起来可怕的错误信息，以及何时以及如何启动备用内核。

### 什么是内核？

你会听到很多关于内核的不同定义。许多定义完全令人困惑，有些技术上是正确的，但让初学者感到困惑，而其他的则是错误的。以下的定义并不完整，但对于大多数人来说，它足够简单易懂：*内核是硬件和软件之间的接口*。

内核让软件可以将数据写入磁盘驱动器和网络。当一个程序需要内存时，内核负责处理访问物理内存芯片和为该任务分配资源的所有底层细节。一旦你的 MP3 文件通过编解码器软件，内核将编解码器输出转换为你的特定声卡能够理解的零和一的流。当一个程序请求 CPU 时间时，内核会为其安排一个时间槽。简而言之，内核提供了程序访问硬件资源所需的所有软件接口。

虽然内核的工作定义起来很简单（至少在这种简化的方式下），但实际上它是一项复杂的任务。不同的程序期望内核提供不同的硬件接口，而不同类型的硬件以不同的方式提供接口。例如，FreeBSD 支持几打种类的以太网卡，每种卡都有自己的要求，内核必须处理这些要求。如果内核无法与网卡通信，系统就无法联网。不同的程序请求以不同的方式安排内存，如果你有一个请求内核不支持的内存安排方式的程序，那你就倒霉了。内核在启动序列中如何检查某些硬件，决定了硬件的行为方式，所以你必须控制这一点。有些设备以友好的方式自我标识，而有些则在你询问它们的功能时会锁死。

内核和 FreeBSD 附带的任何模块都是 */boot/kernel* 目录中的文件。第三方内核模块放在 */boot/modules* 中。系统中的其他文件不是内核的一部分。非内核文件统称为 *userland*，意味着它们是为用户准备的，即使它们使用内核的功能。

由于内核只是文件集合，你可以为特殊情况准备备用内核。在你自己构建内核的系统中，你会找到 */boot/kernel.old*，这是一个目录，包含了当前内核之前安装的内核。我习惯性地将与系统一起安装的内核复制到 */boot/kernel.install*。你也可以创建自己的特殊内核。FreeBSD 团队使得配置和安装内核尽可能简单。改变内核最简单且最有支持的方式是通过 sysctl 接口。

### 内核状态：sysctl

sysctl(8) 程序允许你查看内核使用的值，并在某些情况下设置它们。更复杂的是，这些值有时也被称为 *sysctl*。sysctl 接口是一个强大的功能，因为在许多情况下，它可以让你解决性能问题，而无需重建内核或重新配置应用程序。不幸的是，这种能力也让你有可能将正在运行的程序“踢倒”，并让你的用户非常不高兴。

sysctl(8) 程序处理所有的 sysctl 操作。在本书中，我将指出特定的 sysctl 如何改变系统行为，但首先，你需要一般性地了解 sysctl。开始时，先抓取你系统中所有对人类可见的 sysctl，并将它们保存到一个文件中，以便你可以轻松地进行研究。

```
# sysctl -o -a > sysctl.out
```

文件 *sysctl.out* 现在包含了数百个 sysctl 变量及其值，其中大部分看起来毫无意义。然而，你可以理解其中一些，而无需了解太多内容：

```
kern.hostname: storm
```

这个特定的 sysctl，叫做`kern.hostname`，其值为`storm`。奇怪的是，我运行该命令的系统的主机名也是*storm*，而且该 sysctl 提示这是内核为其运行的系统指定的名称。使用`-a`标志查看这些 sysctl。大多数 sysctl 都应该以这种方式读取，但有一些被称为*不透明 sysctl*，只能由用户态程序解读。使用`-o`标志显示不透明的 sysctl。

```
net.local.stream.pcblist: Format:S,xunpcb Length:5488 Dump:0x20000000000000001
1000000dec0adde...
```

我可以猜测变量`net.local.stream.pcblist`代表了网络栈中的某些内容，但我甚至无法猜测其值的含义。像`netstat(1)`这样的用户态程序从这些不透明的 sysctl 中获取信息。

#### *sysctl MIBs*

这些 sysctl 以树状格式组织，称为*管理信息库（MIB）*，包含多个大类，如 net（网络）、kern（内核）和 vm（虚拟内存）。表 6-1 列出了在运行 GENERIC 内核的系统上 sysctl MIB 树的根节点。

**表 6-1：** sysctl MIB 树的根节点

| **sysctl** | **功能** |
| --- | --- |
| `kern` | 核心内核功能和特性 |
| `vm` | 虚拟内存系统 |
| `vfs` | 文件系统 |
| `net` | 网络 |
| `debug` | 调试 |
| `hw` | 硬件 |
| `machdep` | 机器相关设置 |
| `user` | 用户态接口信息 |
| `p1003_1b` | POSIX 行为 |
| `kstat` | 内核统计信息 |
| `dev` | 设备特定信息 |
| `security` | 特定于安全的内核功能 |

这些类别每个还会进一步细分。例如，`net`类别，涵盖所有与网络相关的 sysctl，被分为 IP、ICMP、TCP 和 UDP 等子类别。管理信息库的概念在系统管理的其他部分也有应用，我们将在第二十一章看到，未来的职业生涯中你也会接触到。术语*sysctl MIB*和*sysctl*通常可以互换使用。每个类别的命名是通过将父类别和所有子类别串联在一起，形成一个唯一的变量名，例如：

```
--snip--
kern.maxfilesperproc: 11095
kern.maxprocperuid: 5547
kern.ipc.maxsockbuf: 262144
kern.ipc.sockbuf_waste_factor: 8
kern.ipc.max_linkhdr: 16
--snip--
```

这里列出了五个 sysctl，来自`kern`类别的中间部分。前两个直接位于`kern`标签下，除了它们与内核相关这一点外，并没有与其他值有明显的分组关系。剩余的三个都以`kern.ipc`开头，它们属于内核 sysctl 的 IPC（进程间通信）部分。如果你继续阅读你保存的 sysctl，你会发现一些 sysctl 变量有多个类别层级。

#### *sysctl 值和定义*

每个 MIB 都有一个值，表示内核使用的缓冲区、设置或特性。更改该值会改变内核的操作方式。例如，内核负责传输和接收数据包，但默认情况下不会将数据包从一个接口发送到另一个接口。你可以更改一个 sysctl 以允许这种转发，从而将主机变成一个路由器。

每个 sysctl 值可以是字符串、整数、二进制值或不透明类型。*字符串* 是任意长度的自由格式文本；*整数* 是普通的整数；*二进制* 值可以是 0（关闭）或 1（开启）；*不透明类型* 是机器代码片段，只有专门的程序才能解释。

许多 sysctl 值文档不全；没有一个单一的文档列出了所有可用的 sysctl MIB 及其功能。MIB 的文档通常出现在相应功能的 man 页面中，或者有时仅出现在源代码中。例如，`kern.securelevel` 的原始文档（在第九章讨论）在 security(7) 中。尽管近年来 sysctl 文档有所扩展，许多 MIB 仍然没有文档。

幸运的是，某些 MIB 有明显的含义。例如，正如我们在本章后面讨论的那样，如果你经常启动不同的内核，这是一个重要的 MIB：

```
kern.bootfile: /boot/kernel/kernel
```

如果你在调试一个问题，并且需要依次重启多个不同的内核，你可能会很容易忘记已经启动了哪个内核（不过，实际上这从未发生过在我身上）。因此，一个提醒可能会很有帮助。

了解一个 sysctl 的作用的一个简单方法是使用 `-d` 开关与完整的 MIB 结合使用。这会打印出该 sysctl 的简短描述：

```
# sysctl -d kern.maxfilesperproc
kern.maxfilesperproc: Maximum files allowed open per process
```

这个简短的定义告诉你，这个 sysctl 控制的正是你可能想象的内容。不幸的是，并不是所有的 sysctl 都提供 `-d` 参数的定义。虽然这个例子相对简单，但其他 MIB 可能更难以猜测。

#### *查看 sysctl 设置*

要查看 MIB 树中特定子树下所有可用的 MIB，可以使用 `sysctl` 命令并输入你想查看的树的名称。例如，要查看 `kern` 下的所有内容，输入以下命令：

```
# sysctl kern
kern.ostype: FreeBSD
kern.osrelease: 12.0-CURRENT
kern.osrevision: 199506
kern.version: FreeBSD 12.0-CURRENT #0 r322672: Fri Aug 18 16:31:34 EDT 2018
    root@storm:/usr/obj/usr/src/sys/GENERIC
--snip--
```

这个列表会持续很长时间。如果你刚开始熟悉 sysctl，你可能会用这个来查看可用项。要获取特定 sysctl 的确切值，请将完整的 MIB 名称作为参数传入：

```
# sysctl kern.securelevel
kern.securelevel: -1
```

MIB `kern.securelevel` 的整数值为 `-1`。我们将在第九章中讨论这个 sysctl 及其值的含义。

#### *更改 sysctl 设置*

有些 sysctl 是只读的。例如，看看硬件 MIB：

```
hw.model: Intel(R) Xeon(R) CPU E5-1620 v2 @ 3.70GHz
```

FreeBSD 项目尚未开发出通过软件设置将 Intel 硬件转换为 ARM64 硬件的技术，因此这个 sysctl 是只读的。如果你能够修改它，所有你会做的就是崩溃系统。FreeBSD 通过不允许你更改此值来保护你。尝试更改它不会造成任何损害，但你会收到警告。另一方面，考虑以下 MIB：

```
vfs.usermount: 0
```

这个 MIB 决定用户是否可以挂载可移动媒体，如 CDROM 和软盘驱动器，详见第十三章。更改此 MIB 不需要对内核进行大量调整或修改硬件；它只是一个内核内的权限设置。要更改此值，可以使用`sysctl(8)`命令、sysctl MIB、等号和期望的值：

```
# sysctl vfs.usermount=1
vfs.usermount: 0 -> 1
```

sysctl(8)程序会响应并显示 sysctl 名称、旧值和新值。这个 sysctl 现在已经更改。像这样可以动态调整的 sysctl 称为*运行时可调 sysctl*。

#### *自动设置 sysctl*

一旦你调整了内核设置以满足个人需求，你可能希望这些设置在重启后仍然生效。可以使用文件*/etc/sysctl.conf*来实现这一点。在这个文件中列出你希望设置的每个 sysctl 及其期望值。例如，要在启动时设置`vfs.usermount` sysctl，可以在*/etc/sysctl.conf*中添加如下内容：

```
vfs.usermount=1
```

### 内核环境

内核是由引导加载程序启动的一个程序。引导加载程序可以将环境变量传递给内核，从而创建*内核环境*。内核环境也是一个 MIB 树，类似于 sysctl 树。很多（但并非所有）环境变量最终会被映射为只读的 sysctl。

#### *查看内核环境*

使用 kenv(8)查看内核环境。为它提供一个内核环境变量的名称，可以只查看该变量，或者不带参数运行它以查看整个树状结构。

```
# kenv
LINES="24"
acpi.oem="SUPERM"
acpi.revision="2"
acpi.rsdp="0x000f04a0"
acpi.rsdt="0x7dff3028"
--snip--
```

这些变量看起来很像加载器变量，因为它们就是加载器变量。它们通常与初始硬件探测相关。如果你的串口使用了不寻常的内存地址，内核需要在尝试探测之前知道这个信息。

这些环境设置也被称为*启动时可调的 sysctl*，或*可调项*，通常与低级硬件设置相关。例如，当内核首次探测硬盘时，它必须决定是否使用基于标识的标签或基于 GPT ID 的标签。这个决定必须在内核访问硬盘之前做出，而且在不重启机器的情况下无法改变。

内核环境变量只能通过加载程序设置。你可以在启动时手动进行更改，或在*/boot/loader.conf*中设置它们，以便在下次启动时生效（见第四章）。

就像*sysctl.conf*一样，在*loader.conf*中设置可调值也可能会导致系统崩溃。好消息是，这些值可以轻松恢复。

**有太多可调项吗？**

不要将只能在启动时设置的 sysctl 值、可以动态调整的 sysctl 值和已配置为在启动时自动调整但可以动态设置的 sysctl 值混淆。记住，启动时可调的 sysctl 涉及低级内核功能，而运行时可调的 sysctl 涉及更高级的功能。让 sysctl 在启动时自行调整仅仅是保存你工作的一个例子——它并不改变该 sysctl 所属的类别。

#### *向设备驱动程序提供提示*

你可以使用环境变量来告知设备驱动程序所需的设置。你将通过阅读驱动程序的手册页和其他文档来了解这些设置。此外，许多古老的硬件要求内核以非常特定的 IRQ 和内存值来访问它们。如果你足够老，记得插卡即“上天祈祷”的“硬件配置”软盘和专门的总线主卡插槽，你知道我在说什么，甚至今天你可能还会在你的硬件柜里找到这样的系统。（如果你太年轻了，给我们这些老人买一杯酒，听听我们的噩梦故事。^(2)) 你可以告诉 FreeBSD 探测你指定的任何 IRQ 或内存地址上的硬件，这在你有一张已知配置的卡，而那张改变配置的软盘已经腐烂了多年前时非常有用。

如果你真的很不幸运，可能会拥有一台内置软盘驱动器的机器。查看 */boot/device.hints*，可以找到配置此硬件的条目：

```
hint.fdc.0.at="isa"
hint.➊fdc.➋0.➌port=➍"0x3F0"
hint.fdc.0.irq=➎"6"
hint.fdc.0.drq=➏"2"
```

这些条目都是给 fdc(4)设备驱动程序的提示 ➊。该条目用于 fdc 设备编号零 ➋。如果启用此设备，启动时内核将探测位于内存地址（或端口 ➌）0x3F0 ➍、IRQ 6 ➎ 和 DRQ 2 ➏的卡。如果它发现具有这些特征的设备，就会将其分配给 fdc(4)驱动程序。如果该设备不是软盘驱动器，你将看到有趣的崩溃。^(3)

**测试启动时可调参数**

所有这些提示和启动时可调的 sysctl 都可以在启动加载器中使用，并可以在 OK 提示符下交互设置，如第四章中所讨论。你可以在不编辑 *loader.conf* 的情况下测试设置，找到合适的值后，再将其永久修改到文件中。

启动时可调参数和 sysctl 让你调整内核的行为，但内核模块允许你向正在运行的内核添加功能。

### 内核模块

内核模块是内核的一部分，可以在需要时启动或加载，在不使用时卸载。内核模块可以在插入硬件时加载，并随硬件一起移除。这极大地扩展了系统的灵活性。而且，若将所有可能的功能都编译到内核中，内核的体积会非常庞大。使用模块后，你可以拥有一个更小、更高效的内核，只有在需要时才加载那些不常用的功能。

就像默认的内核文件保存在*/boot/kernel/kernel*一样，内核模块也保存在*/boot/kernel/*目录下。查看该目录，您会看到数百个内核模块文件。每个内核模块的名称以*.ko*结尾。通常情况下，文件的名称与模块所包含的功能相对应。例如，文件*/boot/kernel/wlan.ko*处理 wlan(4)无线层。FreeBSD 需要这个模块来支持无线网络。

#### *查看已加载的模块*

kldstat(8)命令显示已加载到内核中的模块。

```
   # kldstat
   Id Refs Address           Size     Name
➊ 1   36 0xffffffff80200000 204c3e0  kernel
➋ 2    1 0xffffffff8224e000 3c14f0   zfs.ko
➌ 3    2 0xffffffff82610000 d5f8     opensolaris.ko
➍ 5    1 0xffffffff82821000 ac15     linprocfs.ko
   --snip--
```

这个桌面系统上已经加载了三个内核模块。第一个是内核本身➊；接着是支持 ZFS 的模块➋，以及 ZFS 所需的 OpenSolaris 内核功能模块➌。由于我在这个主机上试验 Linux 软件（参见第十七章），所以看到加载了 linprocfs(5)模块➍并不意外。

每个模块都包含一个或多个子模块，您可以通过使用`kldstat -v`来查看它们，但内核本身有数百个子模块——因此准备好面对大量输出。

#### *加载和卸载模块*

加载和卸载内核模块可以使用 kldload(8)和 kldunload(8)命令。例如，假设我在测试主机上实验 IPMI 功能。这需要 ipmi(4)内核模块。虽然我通常会使用*loader.conf*在启动时自动加载该模块，但我现在在实验室中。我使用`kldload`命令和包含该功能的内核模块或文件的名称：

```
# kldload /boot/kernel/ipmi.ko
```

如果我恰好记得模块的名称，我可以直接使用该名称。模块名称不需要后缀*.ko*。我恰好记得 IPMI 模块的名称。

```
# kldload ipmi
```

大多数情况下，我那颗脆弱的脑袋依赖于 shell 中的 tab 补全功能来提醒我模块的完整和正确名称。

实验完成后，我会卸载该模块。^(4)指定在 kldstat(8)中显示的内核模块名称。

```
# kldunload ipmi
```

任何正在使用中的模块，如每当使用 ZFS 时加载的*opensolaris.ko*模块，都不允许卸载。尝试卸载一个正在使用中的模块时，您将遇到类似下面的错误：

```
# kldunload opensolaris
kldunload: can't unload file: Device busy
```

系统管理员加载模块的频率远远高于卸载模块。卸载模块通常能正常工作，并且大多数情况下都能成功，但它也是导致系统崩溃最常见的方式之一。如果卸载模块时触发了崩溃，请按照第二十四章的指引提交错误报告。

#### *启动时加载模块*

使用*/boot/loader.conf*文件在启动时加载模块。默认的*loader.conf*包含了许多加载内核模块的示例，但语法始终是相同的。获取内核模块的名称，去掉后缀*.ko*，然后添加字符串`_load="YES"`。例如，要在启动时自动加载模块*/boot/kernel/procfs.ko*，可以在*loader.conf*中添加以下内容：

```
procfs_load="YES"
```

当然，最难的部分是知道加载哪个模块。简单的模块是设备驱动程序；如果你安装了一个新的网络卡或 SCSI 卡，而你的内核不支持它，你可以加载驱动程序模块，而不必重新配置内核。在这种情况下，你需要找出哪个驱动程序支持你的卡；man 页和 Google 会帮助你解决这个问题。在本书中，我会为解决特定问题提供具体的内核模块指引。

等一下——为什么 FreeBSD 需要你加载一个设备驱动来识别硬件，而它在启动时几乎可以识别所有硬件？这是个很好的问题！答案是，你可能已经构建了自己的定制内核，并移除了对你不使用的硬件的支持。你不知道如何构建内核？好吧，让我们现在就解决这个问题。

### 构建你自己的内核

最终，你会发现，仅仅使用 sysctl(8)和模块，你无法像希望的那样调整内核，你唯一的解决方法就是构建一个定制内核。这听起来比实际操作要难；我们说的并不是写代码——只是编辑一个文本文件并运行几个命令。如果你按照流程操作，它是完全安全的。如果你*不*按照流程操作，那么，就像在错误的车道上开车一样。（市中心。高峰时段。）不过，从一个坏的内核恢复过来也并不像想象中那么糟糕。

默认安装中附带的内核称为*GENERIC*。GENERIC 被配置为在各种硬件上运行，尽管不一定是最优的。GENERIC 可以在过去 15 年左右的大多数硬件上顺利启动，我在生产环境中经常使用它。当你定制内核时，可以为特定硬件添加支持，移除不需要的硬件支持，或者启用 GENERIC 中未包含的功能。

**不要重建内核**

曾几何时，构建内核被视为一种成长的仪式。但现在已经不是这种情况了。大多数系统管理员只在玩实验性特性或专用硬件时才需要重建内核。

#### *准备工作*

在构建内核之前，必须拥有内核源代码。如果你按照我在第三章中的建议操作，那么你已经准备好了。如果没有，你可以重新进入安装程序并加载内核源代码，或者从 FreeBSD 镜像站下载源代码，或者跳到第十八章使用 svnlite(1)。如果你不记得是否安装了源代码，可以查看你的*/usr/src*目录。如果其中包含一堆文件和目录，那么你已经有了内核源代码。

在构建新内核之前，你必须了解系统的硬件配置。这可能是一个棘手的问题；组件上的品牌名称不一定能描述设备的身份或功能。许多公司使用重新品牌化的通用组件——我记得有一家厂商发布了四款不同的网卡，但都使用了相同的型号名称，前三款甚至没有标注版本号。唯一区分它们的方法就是不断尝试不同的设备驱动程序，直到找到一个有效的。这个问题已经存在了几十年——许多不同的公司都生产 NE2000 兼容的网卡。盒子外面有厂商的名字，但卡片上的电路却标注了 *NE2000*。幸运的是，一些厂商为其驱动程序和硬件使用标准架构；你可以相当确定英特尔的网卡会被英特尔的设备驱动程序识别。

查看 FreeBSD 系统上硬件信息的最佳位置是文件 */var/run/dmesg.boot*，在第四章中有讨论。每一项条目代表内核中的硬件或软件特性。在为系统构建新内核时，随时保留该系统的 *dmesg.boot* 非常重要。

#### *总线和连接*

计算机中的每个设备都连接到其他某个设备。如果你仔细阅读 *dmesg.boot*，你可以看到这些连接链条。以下是编辑后的启动消息集，来做个演示：

```
➊ acpi0: <SUPERM SMCI--MB> on motherboard
➋ acpi0: Power Button (fixed)
➌ cpu0: <ACPI CPU> on acpi0
   cpu1: <ACPI CPU> on acpi0
➍ attimer0: <AT timer> port 0x40-0x43 irq 0 on acpi0
➎ pcib0: <ACPI Host-PCI bridge> port 0xcf8-0xcff on acpi0
➏ pci0: <ACPI PCI bus> on pcib0
```

这个系统上的第一个设备是 acpi0 ➊。你可能不知道它是什么，但你可以随时阅读 `man acpi` 来了解。(或者，如果你非得这么做，可以读完整章。) 在 acpi0 设备上有一个电源按钮 ➋。CPU ➌ 也连接到 acpi0 设备，还有一个计时设备 ➍。最终，我们找到了第一个 PCI 桥，pcib0 ➎，它连接到 acpi0 设备。第一个 PCI 总线 ➏ 也连接到 PCI 桥。

所以，你常见的 PCI 设备连接到一系列总线，这些总线再通过 PCI 桥与计算机的其他部分通信。你可以查看 *dmesg.boot* 并绘制系统上所有设备的树状图；虽然这不是必须的，但了解各设备的连接位置会让配置内核的成功几率大大提高。

如果你有疑问，可以使用 pciconf(8) 来查看系统上实际存在的设备。`pciconf -lv` 将列出系统上每一个 PCI 设备，无论当前内核是否为其找到驱动程序。

#### *备份你的工作内核*

一个坏的内核可能会使你的系统无法启动，因此你必须时刻保留一个好的内核。内核安装过程会将你之前的内核保留下来作为备份，存放在目录 */boot/kernel.old* 中。这对于能够回滚是很有用的，但我建议你更进一步。有关启动备用内核的详细信息，请参见第四章。

如果你没有保留一个已知的良好备份，可能会发生以下情况。如果你构建了一个新的内核，发现自己犯了一个小错误，必须重新构建它，而系统生成的备份内核实际上是你第一次制作的内核——那个包含小错误的内核。你的工作内核已经被删除。当你发现新的自定义内核也有相同的问题，或者甚至是更严重的错误时，你会深深后悔失去那个工作内核。

一个常见的保留已知良好内核的位置是*/boot/kernel.good*。像这样备份你的工作、可靠的内核：

```
# cp -a /boot/kernel /boot/kernel.good
```

如果你正在使用 ZFS，那么启动环境可能比复制更有意义（请参阅第十二章）。

不要害怕手头保留各种内核。磁盘空间比时间便宜。我知道一些人会将内核保存在以日期命名的目录中，以便在必要时回退到早期版本。许多人还会将当前版本的 GENERIC 内核保存在*/boot/kernel.GENERIC*中，用于测试和调试目的。拥有太多内核的唯一方法是把硬盘填满。

#### *配置文件格式*

FreeBSD 的内核通过文本文件进行配置。没有图形化工具或菜单驱动的系统来配置内核；这与 4.4 BSD 时的情况基本相同。如果你对文本配置文件不熟悉，构建内核就不适合你。

每个内核配置条目都位于单独的一行。你会看到一个标签来表示这是什么类型的条目，然后是条目的术语。许多条目还会有以井号标记的注释，类似于 FreeBSD 文件系统 FFS 的这个条目。

```
options         FFS                     # Berkeley Fast Filesystem
```

每个完整的内核配置文件由五种类型的条目组成：`cpu`、`ident`、`makeoptions`、`options`和`devices`。这些条目的存在与否决定了内核如何支持相关的功能或硬件：

cpu 这个标签表示该内核支持哪种类型的处理器。针对那些普通 PC 硬件的内核配置文件包括几个 CPU 条目，以涵盖如 486（I486_CPU）、奔腾（I586_CPU）和奔腾 Pro 到现代奔腾 4 CPU（I686_CPU）等处理器。而 amd64/EM64T 硬件的内核配置仅包括一个 CPU 类型——HAMMER，因为该架构只有一个 CPU 系列。虽然一个内核配置可以包括多个 CPU 类型，但它们必须是相似架构的；一个内核可以在 486 和奔腾 CPU 上运行，但你不能让一个内核同时在 Intel 兼容的处理器和 ARM 处理器上运行。

ident 每个内核都有一行`ident`，为内核提供一个名称。这就是 GENERIC 内核获得名称的方式；它是一个任意的文本字符串。

makeoptions 这个字符串为内核构建软件提供指令。最常见的选项是`DEBUG=-g`，它告诉编译器构建一个调试内核。调试内核帮助开发人员排查系统问题。

选项 这些是内核功能，不需要特定硬件。包括文件系统、网络协议和内核内调试器。

设备 也称为*设备驱动程序*，这些为内核提供了如何与特定设备通信的指令。如果你希望系统支持某个硬件，内核必须包括该硬件的设备驱动程序。有些设备条目，称为*伪设备*，并不与特定硬件绑定，而是支持整个硬件类别——例如以太网、随机数生成器或内存磁盘。你可能会问，伪设备与选项有什么区别。答案是，伪设备在至少某些方面表现得像设备，而选项没有类似设备的特征。例如，回环伪设备是一个仅连接到本地机器的网络接口。尽管它没有硬件支持，但软件可以连接到回环接口，并将网络流量发送到同一机器上的其他软件。

这是配置文件的另一个片段——涵盖 ATA 控制器的部分：

```
# ATA controllers
device          ahci     # AHCI-compatible SATA controllers
device          ata      # Legacy ATA/SATA controllers
device          mvs      # Marvell 88SX50XX/88SX60XX/88SX70XX/SoC SATA
device          siis     # SiliconImage SiI3124/SiI3132/SiI3531 SATA
```

这些设备是不同类型的 ATA 控制器。将这些条目与我们在*/var/run/dmesg.boot*中查看的几个 ATA 条目进行比较：

```
atapci0: <Intel PIIX4 UDMA33 controller> port 0x1f0-0x1f7,0x3f6,0x170
-0x177,0x376,0xc160-0xc16f at device 1.1 on pci0
ata0: <ATA channel> at channel 0 on atapci0
ata1: <ATA channel> at channel 1 on atapci0
ada0 at ata0 bus 0 scbus0 target 0 lun 0
cd0 at ata1 bus 0 scbus1 target 0 lun 0
```

内核配置中有一个 ATA 总线，`device ata`。它是一个“传统”ATA 总线，不管“传统”这个词今天是什么意思。这里的 dmesg 片段以 atapci 设备开始，这是 ATA 与 PCI 连接的控制器。接下来有两个 ATA 总线，ata0 和 ata1。磁盘 ada0 在 ata0 上，而 CD 驱动器 cd0 在 ata1 上。

如果内核配置中没有`device ata`，内核将无法识别 ATA 总线。即使系统发现系统有一个 DVD 驱动器，内核也不知道如何与它进行信息交换。你的内核配置必须包含所有依赖于它们的驱动程序的中介设备。另一方面，如果你的系统没有 ATA RAID 驱动器、软盘驱动器或磁带驱动器，你可以从内核中移除这些设备驱动程序。

如果该主机有 AHCI、MVS 或 SIIS 控制器，那么这些设备名称将在 dmesg 中显示，而不是 ata。

#### *配置文件*

幸运的是，你通常不需要从头开始创建内核配置文件；相反，你是基于现有的配置文件进行构建的。从适合你硬件架构的 GENERIC 内核开始。它可以在*/sys/<arch>/conf*中找到——例如，i386 内核配置文件位于*/sys/i386/conf*，amd64 内核配置文件位于*/sys/amd64/conf*，依此类推。该目录包含多个文件，其中最重要的是*DEFAULTS*、*GENERIC*、*GENERIC.hints*、*MINIMAL*和*NOTES*：

***默认设置*** 这是为特定架构启用的选项和设备列表。这并不意味着你可以编译和运行*默认设置*，但如果你想通过添加设备来构建内核，它是一个起点。不过，使用*GENERIC*更为简单。

***GENERIC*** 这是标准内核的配置。它包含了启动并运行该架构的标准硬件所需的所有设置；这是安装程序使用的内核配置。

***GENERIC.hints*** 这是稍后会安装到*/boot/device.hints*的提示文件。此文件提供了旧硬件的配置信息。

***MINIMAL*** 此配置排除了任何可以从模块加载的内容。

***NOTES*** 这是该硬件平台的全面内核配置。每个平台特定的功能都包含在*NOTES*中。平台无关的内核功能请参见*/usr/src/sys/conf/NOTES*。

许多架构还有仅针对特定硬件的架构特定配置。i386 架构包括 PAE 内核配置，使你能够在 32 位系统上使用超过 4GB 的 RAM。arm 架构包括数十种配置，每种都对应 FreeBSD 支持的不同平台。

有时，你会找到一个完全符合你需求的内核配置。我想要最小的内核。*MINIMAL*内核看起来是一个不错的起点。让我们来构建它。

### 构建内核

一个基础的 FreeBSD 安装，加上操作系统源代码，包含了构建内核所需的所有基础设施。你所需要做的就是通过 KERNCONF 变量告诉系统要构建哪个内核配置。你可以在*/etc/src.conf*（或*/etc/make.conf*，如果你真的是老派做法）中设置 KERNCONF。

```
KERNCONF=MINIMAL
```

如果你正在尝试构建和运行不同的内核，最好在构建内核时在命令行中设置配置文件。使用`make buildkernel`命令来构建内核。

```
# cd /usr/src
# make KERNCONF=MINIMAL buildkernel
```

构建过程首先运行 config(8)来查找语法配置错误。如果 config(8)检测到问题，它会报告错误并停止。有些错误是非常明显的——例如，你可能不小心删除了对 Unix 文件系统（UFS）的支持，但却包含了从 UFS 启动的支持。一个依赖于另一个，config(8)会告诉你具体出了什么问题。其他一些错误消息则比较陌生和难以理解；那些最难以解决的错误可能像这样：

```
MINIMAL: unknown option "NET6"
```

NET6 是 IPv6 选项，不是吗？不，那是*I* NET6。显然有个傻瓜在文本编辑器中查看配置文件时不小心删除了一个字母。这个错误一目了然——只要你熟悉所有支持的内核选项。仔细阅读这些错误信息！

一旦 config(8)验证了配置，内核构建过程将在现代计算机上完成几分钟。构建成功后会显示类似这样的消息。

```
--------------------------------------------------------------
>>> Kernel build for MINIMAL completed on Tue Sep 12 14:27:08 EDT 2017
--------------------------------------------------------------
```

构建完内核后，进行安装。运行`make installkernel`会将当前内核移到*/boot/kernel.old*，并将新内核安装到*/boot/kernel*。安装内核的速度比构建内核要快得多。

**信任内核**

最终，你会到达一个阶段，信任自己的内核配置，并希望用一个命令构建并安装它。`make kernel`命令会构建并安装内核。真正的系统管理员会运行`make kernel && reboot`。

安装完成后，重启你的服务器并观察启动信息。如果一切正常，你会看到类似以下内容，显示正在运行的内核以及它的构建时间。

```
Copyright (c) 1992-2018 The FreeBSD Project.
Copyright (c) 1979, 1980, 1983, 1986, 1988, 1989, 1991, 1992, 1993, 1994
The Regents of the University of California. All rights reserved.
FreeBSD storm 12.0-CURRENT FreeBSD 12.0-CURRENT #0 r323136: Sat Sep  2 
21:46:53 EDT 2018     root@storm:/usr/obj/usr/src/sys/MINIMAL  amd64
--snip--
```

关键是，MINIMAL 内核无法启动所有硬件。它不能启动*大多数*硬件。而对于 MINIMAL 能够启动的硬件，它也无法启动该硬件上的大部分 FreeBSD 安装。

MINIMAL 将所有可以作为模块的东西都放在模块中。磁盘分区方法，无论是 GPT 还是 MBR，都可以作为模块。你必须通过*loader.conf*加载*geom_part_gpt.ko*或*geom_part_mbr.ko*才能启动 MINIMAL。文件系统也是模块，因此你必须加载它们。简而言之，你需要加载硬件和安装选择所需的每个模块。MINIMAL 是所有内核所需模块的一个很好的参考，也是设计自己内核的一个不错的起点，但不足以用于生产环境。

#### *启动备用内核*

那么，如果你的新内核无法工作，或者工作得很差怎么办？也许你忘记了某个设备驱动程序，或者不小心去掉了`INET`选项，导致无法访问互联网。有时它会在启动过程的早期就卡住，唯一能做的就是重启主机。别慌！你还保留着旧的内核，对吧？下面是该怎么做。

首先记录下错误信息。你需要研究这些信息，以找出你的新内核哪里出错了。^(5)不过，要修复这个错误，你需要启动一个工作内核，才能构建一个改进的内核。

在第四章中，我们讨论了启动备用内核的机制。我们将在这里讲解具体的操作步骤，但要查看一些有关加载器管理的详细内容，你可以回去查看前面的部分。现在，我们将专注于为什么需要启动备用内核，以及如何正确操作。

首先决定你想启动哪个内核。你的旧内核应该在*/boot*目录下；在本节中，我们假设你想启动*/boot/kernel.good*中的内核。重启并中断启动过程，进入启动菜单。第五个选项允许你选择其他内核。菜单会显示`loader.conf`中`kernels`选项列出的每个内核目录。默认列出的是`kernel`和`kernel.old`，我会添加`kernel.good`。

一旦你安装了另一个新内核，记住：现有的*/boot/kernel*会被复制到*/boot/kernel.old*，这样你的新内核就可以放在*/boot/kernel*中。如果那个内核无法启动，而你的新内核也无法启动，你将没有可用的内核。这种情况很糟糕。一定要确保手头有一个已知的工作内核。

### 自定义内核配置

也许提供的内核配置都不适合你，你需要的是其他的东西。FreeBSD 让你可以创建任何你想要的配置。但最简单的方式是修改现有的配置。你可以复制一个现有的文件或使用 `include` 选项。我们将从修改现有文件开始。确保你使用正确的架构目录，可能是 */sys/amd64/conf* 或 */sys/i386/conf*。

不要直接编辑配置目录中的任何文件。相反，复制 GENERIC 文件，并以你的机器或内核功能命名，然后编辑副本。在这个例子中，我正在构建一个支持 VirtualBox 系统的最小内核。我将 *GENERIC* 文件复制为名为 *VBOX* 的文件，并用我偏好的文本编辑器打开 *VBOX*。

#### *裁剪内核*

曾几何时，内存比今天贵得多，而且只有较小的容量。当系统只有 128MB 的内存时，你希望这些内存能用于工作，而不是用来存放无用的设备驱动程序。今天，当一台廉价的笔记本电脑以可怜的 64GB 内存艰难度日时，内核大小几乎变得无关紧要。

对大多数人来说，剥除内核中不必要的驱动和功能以减小其大小是浪费时间和精力，但我鼓励你至少做一次。这会教你如何构建内核，这样当你需要测试内核补丁或其他操作时，就不必在处理重新构建的问题时再学习内核构建的知识。当你开始在像 BeagleBone 或 Raspberry Pi 这样的小型主机上实验 FreeBSD 时，这也会有所帮助。

我想构建一个支持 VirtualBox 内核的内核。我在 VirtualBox 上启动一个正常工作的 FreeBSD 安装，以便查看 *dmesg.boot*。我将在 dmesg 和配置文件之间来回切换，注释掉不需要的条目。

##### CPU 类型

在大多数架构上，FreeBSD 只支持一种或两种类型的 CPU。amd64 平台只支持一种，HAMMER。i386 平台支持三种，但其中两种——486 和最初的奔腾——在嵌入式市场之外已经极为过时。

```
cpu             I486_CPU
cpu             I586_CPU
cpu             I686_CPU
```

你只需要包含你拥有的 CPU。如果你不确定硬件中的 CPU，检查 *dmesg.boot*。我有一台古老的笔记本，显示如下：

```
CPU: AMD Athlon(tm) 64 X2 Dual Core Processor 4200+ (2200.10-MHz 686-class CPU)
  Origin = "AuthenticAMD"  Id = 0x20fb1  Stepping = 1
  Features=0x178bfbff<FPU,VME,DE,PSE,TSC,MSR,PAE,MCE,CX8,APIC,SEP,MTRR,PGE,MCA,
CMOV,PAT,PSE36,CLFLUSH,MMX,FXSR,SSE,SSE2,HTT>
--snip--
```

如粗体所示，这是一个 686 类 CPU，这意味着我可以移除 I486_CPU 和 I586_CPU 语句，从而让我的内核更小。

##### 核心选项

在 CPU 类型配置条目之后，我们有一长串用于基本 FreeBSD 服务的选项，如 TCP/IP 和文件系统。普通系统不需要所有这些，但它们的存在提供了极大的灵活性。你还会遇到一些在你的环境中很少使用的选项，以及那些可以从自定义内核配置中移除的选项。我们不会讨论所有可能的内核选项，而是会具体介绍不同类型选项的示例。我会特别提到那些可以从互联网服务器中修剪掉的选项。LINT 文件、手册页以及你最喜欢的互联网搜索引擎可以帮助你了解其他选项。如果你对某个选项有疑问，保留它。或者禁用它，看看会有什么问题。

考虑以下与网络相关的选项：

```
options         INET                 # InterNETworking
options         INET6                # IPv6 communications protocols
options         IPSEC                # IP (v4/v6) security
options         IPSEC_SUPPORT        # Allow kldload of ipsec and tcpmd5
options         TCP_OFFLOAD          # TCP offload
options         TCP_HHOOK            # hhook(9) framework for TCP
options         SCTP                 # Stream Control Transmission Protocol
```

这些选项支持网络。INET 是传统的 TCP/IP，INET6 支持 IPv6。许多类似 Unix 的软件依赖于 TCP/IP，因此你肯定需要这两者。IPSEC 和 IPSEC_SUPPORT 让你使用 IPSec VPN 协议。我肯定不会在我的虚拟机上使用这些，所以我会将它们注释掉。

TCP_OFFLOAD 选项允许网络堆栈将 TCP/IP 计算卸载到网卡上。听起来不错，除了虚拟机上的 vnet(4) 网络接口并不执行此功能。砍掉它！

TCP_HHOOK 选项为你提供了一个方便的手册页供阅读。我会使用这个选项吗？或许吧。更重要的是，我不知道我正在运行的软件是否需要它。我会保留它。

SCTP 传输协议很不错，但对于在我笔记本电脑上运行的虚拟机来说完全没用。再见。

```
options         FFS             # Berkeley Fast Filesystem
options         SOFTUPDATES     # Enable FFS soft updates support
options         UFS_ACL         # Support for access control lists
options         UFS_DIRHASH     # Improve performance on big directories
options         UFS_GJOURNAL    # Enable gjournal-based UFS journaling
```

FFS 选项提供了标准的 FreeBSD 文件系统 UFS。即使是 ZFS 主机也需要 UFS 支持。保留它。其他选项都与 FFS 相关。我们在 第十一章 中详细讨论了 FFS 及其选项，比你想象的还要详细，但现在，先相信我，照做就行。

软更新确保即使系统不正确关闭时也能保证磁盘完整性。如在 acl(9) 中讨论的，UFS 访问控制列表允许你为文件授予非常详细的权限，而这些我在我的虚拟主机上不需要。砍掉它！

UFS_DIRHASH 启用目录哈希，使得包含成千上万个文件的目录更加高效。保留它。我将使用软更新日志，而不是 gjournaling，所以 UFS_GJOURNAL 可以去掉。

```
options         MD_ROOT                 # MD is a potential root device
```

这个选项——以及所有其他 _ROOT 选项——允许系统使用标准 UFS 或 ZFS 文件系统以外的其他磁盘设备作为根分区。安装程序使用内存设备（MD）作为根分区。如果你使用的是无盘系统（参见 第二十三章），你将需要一个 NFS 根分区。如果你在一台标准计算机系统上运行 FreeBSD，且配有硬盘、键盘等，你的内核不需要这些功能。

```
options         NFSCL                   # Network Filesystem Client
options         NFSD                    # Network Filesystem Server
options         NFSLOCKD                # Network Lock Manager
```

这两个选项支持网络文件系统（参见 第十三章）。这里的关键问题是，你需要 NFS 吗？如果需要，你是需要作为服务器还是客户端？我将保留这些选项。

```
options         MSDOSFS              # MSDOS filesystem
options         CD9660               # ISO 9660 filesystem
options         PROCFS               # Process filesystem (requires PSEUDOFS)
options         PSEUDOFS             # Pseudo-filesystem framework
```

这些选项支持间歇性使用的文件系统，如 FAT、CD、进程文件系统和伪文件系统框架。我们在第十三章中讨论了许多这些文件系统，但它们都可以作为内核模块使用。删除它们。

```
options         COMPAT_FREEBSD32        # Compatible with i386 binaries
options         COMPAT_FREEBSD4         # Compatible with FreeBSD4
options         COMPAT_FREEBSD5         # Compatible with FreeBSD5
options         COMPAT_FREEBSD6         # Compatible with FreeBSD6
--snip--
```

这些兼容性选项允许你的系统运行为较旧版本的 FreeBSD 或者基于较旧版本的 FreeBSD 内核假设构建的软件。如果你是从头开始安装系统，可能不需要兼容 FreeBSD 4、5 或 6，但有相当一部分软件需要兼容 32 位的 FreeBSD。保留 `COMPAT_FREEBSD32` 选项，否则你的系统 *会* 崩溃。

```
options         SCSI_DELAY=5000         # Delay (in ms) before probing SCSI
```

`SCSI_DELAY` 选项指定 FreeBSD 在找到你的 SCSI 控制器后等待多少毫秒再进行探测，给它们一点时间来启动并自我识别到 SCSI 总线上。如果你没有 SCSI 硬件，可以删除这一行。

```
options         SYSVSHM                 # SYSV-style shared memory
options         SYSVMSG                 # SYSV-style message queues
options         SYSVSEM                 # SYSV-style semaphores
```

这些选项启用 System-V 风格的共享内存和进程间通信。许多数据库程序使用此功能。

##### 多处理器

以下条目启用 i386 内核中的对称多处理（SMP）：

```
options         SMP                     # Symmetric MultiProcessor Kernel
options         DEVICE_NUMA             # I/O Device Affinity
options         EARLY_AP_STARTUP
```

这些选项可能不会有害，但如果你知道你的系统运行在单核板卡上，可能是非常旧的系统或使用嵌入式硬件，你可以删除这些选项。

##### 设备驱动

在所有选项之后，你会找到设备驱动条目，这些条目被分组得相当合理。为了缩小内核的大小，你需要删除所有主机不使用的部分——但究竟是什么是主机不使用的呢？可以在 *dmesg.boot* 中搜索每个设备驱动。

第一个设备条目是总线，例如 `device pci` 和 `device acpi`。除非你系统中确实没有这种类型的总线，否则请保留这些条目。

接下来，我们进入大多数人认为的真正的设备驱动部分——软盘驱动器、SCSI 控制器、RAID 控制器等的条目。如果你的目标是减少内核的大小，这是一个很好的修剪位置；删除所有不具备的硬件设备驱动。你还会找到一些设备驱动条目，如键盘、显卡、USB 端口等的条目。你几乎肯定不想删除这些。

网络卡设备驱动部分相当长，看起来很像 SCSI 和 IDE 部分。如果你近期不打算更换网络卡，可以删除所有不使用的网络卡驱动。

我们不会在这里列出所有设备驱动，因为从这样一个列表中学到的东西很少，除了展示 FreeBSD 在我编写本节时支持的硬件。请查看你当前运行的 FreeBSD 版本的发布说明，以了解它支持的硬件。

你还会看到一大部分虚拟化驱动程序。最常用的虚拟接口基于 VirtIO，但你也会看到专门为 Xen、Hyper-V 和 VMware 设计的驱动程序。内核只需要为其运行的平台提供虚拟化驱动程序。真实硬件的内核不需要任何虚拟化驱动程序，即使主机上会运行虚拟机。

##### 伪设备

在 GENERIC 内核配置的底部，你会找到一组选项伪设备。顾名思义，这些完全是由软件创建的。以下是一些常用的伪设备。

```
device          loop            # Network loopback
```

回环设备允许系统通过网络套接字和网络协议与自身进行通信。我们将在下一章详细讨论网络连接。你可能会惊讶于有多少程序使用回环设备，因此不要删除它。

```
device          random                  # Entropy device
device          padlock_rng             # VIA Padlock RNG
device          rdrand_rng              # Intel Bull Mountain RNGdevice          
```

这些设备提供伪随机数，供加密操作和一些关键任务应用程序（如游戏）使用。有些设备需要底层芯片组的支持。FreeBSD 支持多种随机源，透明地将它们聚合到随机设备*/dev/random*和*/dev/urandom*中。

```
device          ether           # Ethernet support
```

以太网具有许多设备特性，对于 FreeBSD 来说，将其视为一个设备是最简单的。除非你想学习，否则无需更改此设置。

```
device          vlan                    # 802.1Q VLAN support
device          tun                     # Packet tunnel
device          gif                     # IPv6 and IPv4 tunneling
```

这些设备支持像 VLAN 和各种隧道这样的网络功能。

```
device          md              # Memory "disks"
```

内存磁盘允许你将文件存储在内存中。这对于非常快速的临时数据存储非常有用，正如我们将在第十三章中学习的那样。对于大多数（但不是所有）互联网服务器，内存磁盘是浪费内存的。你也可以使用内存磁盘来挂载和访问磁盘映像。如果你不使用内存磁盘，可以将其从内核中移除。

##### 可移动硬件

GENERIC 内核支持几种不同类型的可移动硬件。如果你的笔记本电脑是由包含两个连续数字 9 或 0 的年份生产的，它可能有 Cardbus 甚至 PCMCIA 卡。否则，你不需要在内核中支持这些。FreeBSD 支持热插拔的 PCI 卡，但如果你没有这些卡，就将这些驱动程序丢弃。

##### 包含配置文件

你的内核二进制文件可能与其构建所在的机器分开。我建议使用 INCLUDE_CONFIG_FILE 选项，将内核配置文件复制到已编译的内核中。这样你会失去任何注释，但至少你会保留此内核中的选项和设备，并且如果需要，可以复制它。sysctl `kern.conftxt`包含内核配置。

一旦你有了修剪过的内核，尝试构建它。你的第一个内核配置肯定会出错。

#### *内核构建故障排除*

如果你的内核构建失败，第一步是查看输出的最后几行。这些错误中的一些可能非常难以理解，但其他的会很直白。需要记住的重要一点是，错误信息中说“在 *某目录* 停止”的并没有用；有用的错误信息会出现在它们之前。我们在《寻求帮助》中讨论了如何解决这些问题，见第 11 页：拿着错误信息去搜索引擎搜一搜。编译错误通常是由于配置错误引起的。

幸运的是，FreeBSD 强制要求在安装任何东西之前编译完整的内核。构建失败不会损坏已安装的系统。然而，它将给你一个机会来测试我们在第一章中提到的故障排除技巧。

最常见的错误类型是 `make buildkernel` 阶段失败。它可能看起来像这样：

```
--snip--
linking kernel.full
vesa.o: In function `vesa_unload':
/usr/src/sys/dev/fb/vesa.c:1952: undefined reference to ➊ `vesa_unload_ioctl'
vesa.o: In function `vesa_configure':
/usr/src/sys/dev/fb/vesa.c:1169: undefined reference to ➋ `vesa_load_ioctl'
*** Error code 1
--snip--
```

你会看到几页的错误代码 1 消息，但实际的错误出现在它们之前。

我们的内核中的某行代码需要`vesa_unload_ioctl` ➊ 和 `vesa_load_ioctl` ➋ 这两个函数，但提供这些功能的设备或选项并没有包含在内核中。尝试通过互联网搜索这些错误，看看是否有这些函数的手册页。如果都不行，再去查找源代码。

```
# cd /usr/src/sys
# grep -R vesa_unload_ioctl *
dev/fb/vesa.h:int vesa_unload_ioctl(void);
dev/fb/vesa.c:  if ((error = vesa_unload_ioctl()) == 0) {
dev/syscons/scvesactl.c:vesa_unload_ioctl(void)
```

等等——在 GENERIC 配置文件中不是有提到“syscons”驱动程序吗？

```
# syscons is the default console driver, resembling an SCO console
#device          sc
#options         SC_PIXEL_MODE           # add support for the raster text mode
```

我已经注释掉了 sc(4) 驱动程序。把它重新添加回去，再试一次。

还有更多“正确”的方法来弄清楚哪些内核设备需要哪些设备。最终的方法归结为“阅读并理解源代码”。对于我们大多数人来说，试错、研究，再继续试错，反而是更快速的方式。

### 包含、排除与扩展内核

现在你可以编译内核了，接下来让我们来点花样，看看如何使用包含、各种 *no* 配置和 *NOTES* 文件。

#### *备注*

FreeBSD 的内核包含了许多在 GENERIC 中没有的功能。这些特殊功能通常是为特定系统或某些特殊网络的边缘案例设计的。你可以在每个平台的内核配置目录下找到包含硬件特定功能的完整列表——例如，*/sys/amd64/conf/NOTES*。而那些硬件无关的内核功能——即适用于 FreeBSD 支持的所有平台的功能——可以在 */sys/conf/NOTES* 中找到。如果你的硬件在 GENERIC 内核中似乎没有完全得到支持，看看 *NOTES* 文件。虽然其中一些功能较为晦涩，但如果你有这些硬件，你会感激它们的。让我们来看看 *NOTES* 中的一个典型条目：

```
# Direct Rendering modules for 3D acceleration.
device          drm             # DRM core module required by DRM drivers
device          mach64drm       # ATI Rage Pro, Rage Mobility P/M, Rage XL
device          mgadrm          # AGP Matrox G200, G400, G450, G550
device          r128drm         # ATI Rage 128
device          savagedrm       # S3 Savage3D, Savage4
device          sisdrm          # SiS 300/305, 540, 630
device          tdfxdrm         # 3dfx Voodoo 3/4/5 and Banshee
device          viadrm          # VIA
options         DRM_DEBUG       # Include debug printfs (slow)
```

你在桌面上使用这些显卡吗？也许你想要一个包含适当设备驱动程序的自定义内核。

如果 *NOTES* 文件列出了每个设备的所有功能，为什么不直接将其作为内核的基础呢？首先，这样的内核会使用比 GENERIC 内核更多的内存。虽然即使是小型现代计算机也有足够的内存来运行 GENERIC，而如果内核变得比 GENERIC 大十倍，却没有相应的功能增加，人们就会感到烦恼。此外，许多选项是互斥的。你会发现有些选项允许你决定内核如何调度进程。例如，内核一次只能使用一个调度程序，而每个调度程序的影响遍及整个内核。将所有调度程序同时添加到内核中，会增加代码复杂性并降低稳定性。

我每发布一个或两个版本，就会重点查看 *NOTES*，以便寻找一些有趣的新特性。

#### *包含和排除*

FreeBSD 的内核配置有两个有趣的功能，可以使维护内核变得更容易：`no` 选项和 `include` 功能。

`include` 功能让你将一个独立的文件引入到内核配置中。例如，如果你有一个可以描述为“GENERIC 加上几个额外功能”的内核配置，你可以使用 `include` 语句将 GENERIC 内核配置包括进来：

```
include GENERIC
```

所以，如果你想构建一个既具备 GENERIC 所有功能，又支持 VIA 3d 芯片的 DRM 功能的内核，你可以创建一个完全由以下内容组成的有效内核配置：

```
ident        VIADRM
include      GENERIC
options      drm
options      viadrm
```

你可能会认为这实际上比将 GENERIC 复制到一个新文件并进行编辑更麻烦，而你是对的。那么，为什么还要做这个呢？最大的原因是，随着 FreeBSD 的升级，GENERIC 配置可能会发生变化。FreeBSD 12.1 中的 GENERIC 与 12.0 中的略有不同。你的新配置对于这两个版本都是有效的，并且在这两种情况下都可以正当描述为“GENERIC 加上我的选项”。

这种方法很适合包括内容，但对于从内核中删除内容并不太有效。与其为每个新的 FreeBSD 版本手动重建内核，你可以使用 `include` 语句，但通过 `nodevice` 和 `nooptions` 关键字排除不需要的条目。使用 `nodevice` 移除不需要的设备驱动，而 `nooptions` 则禁用不需要的选项。

查看 -current 机器上的 GENERIC-NODEBUG 内核配置。它与 GENERIC 配置相同，只是禁用了所有调试功能。

```
include GENERIC

ident   GENERIC-NODEBUG

nooptions       INVARIANTS
nooptions       INVARIANT_SUPPORT
nooptions       WITNESS
nooptions       WITNESS_SKIPSPIN
nooptions       BUF_TRACKING
nooptions       DEADLKRES
nooptions       FULL_BUF_TRACKING
```

我们首先包括了 GENERIC 内核配置。不过，这个内核自我标识为 GENERIC-NODEBUG。接下来的七个 `nooptions` 语句关闭了 FreeBSD-current 的标准调试选项。开发人员使用 GENERIC-NODEBUG 内核来查看内核调试器是否引起了问题。如果带调试的内核在运行时出现崩溃，而没有调试的内核则不会崩溃，那么调试代码看起来就会显得可疑。

#### *跳过模块*

如果你已经费心构建了一个自定义内核，你可能清楚地知道主机需要哪些内核模块。如果你从不打算使用它们，为什么还要构建这么多内核模块呢？你可以使用`MODULES_OVERRIDE`选项关闭模块的构建。将`MODULES_OVERRIDE`设置为你想要构建和安装的模块列表。

```
# make MODULES_OVERRIDE='' kernel
```

也许你想构建大多数模块，但你有理由讨厌某个特定的模块。使用`WITHOUT_MODULES`将其从构建中排除。在这里，我排除了 vmm 模块，因为我甚至不想有在 VirtualBox 上运行 bhyve(8)的诱惑。从这里开始，虚拟化层的数量会迅速增多，最终我可能会感叹为什么我的笔记本变慢了。

```
# make KERNCONF=VBOX WITHOUT_MODULES=vmm kernel
```

选择性构建模块，结合自定义内核，可以让你将系统锁定到非常小的框架中。当你发现缺少某个你从未想过会需要的功能时，你就会明白这些框架有多小。如果你必须构建一个内核，保守地保留一些功能是明智的。

现在，你的本地机器已经调优到你想要的精确状态，接下来我们来考虑互联网的其余部分。
