## 第二十章：信号：基本概念

本章以及接下来的两章将讨论信号。虽然基本概念简单，但我们的讨论相当冗长，因为有许多细节需要涵盖。

本章涵盖以下主题：

+   各种不同的信号及其用途；

+   内核在什么情况下可能会为进程生成信号，以及一个进程可以使用哪些系统调用来向另一个进程发送信号；

+   默认情况下，进程如何响应信号，以及进程如何改变对信号的响应，特别是通过使用信号处理程序，这是一种在接收到信号时自动调用的程序员定义的函数；

+   使用进程信号屏蔽来阻塞信号，以及与之相关的待处理信号的概念；

+   进程如何挂起执行并等待信号的到达。

## 概念和概述

*信号*是通知进程某个事件发生的机制。信号有时被描述为*软件中断*。信号类似于硬件中断，它们中断了程序的正常执行流程；在大多数情况下，无法预测信号何时到达。

一个进程可以（如果它有适当的权限）向另一个进程发送信号。在这种用法中，信号可以用作同步技术，甚至作为进程间通信（IPC）的原始形式。进程也可以向自身发送信号。然而，许多发送给进程的信号通常来自内核。导致内核为进程生成信号的事件类型包括以下内容：

+   发生硬件异常，这意味着硬件检测到故障条件并将其通知给内核，内核随后向相关进程发送相应的信号。硬件异常的示例包括执行格式错误的机器语言指令、除以 0 或引用无法访问的内存区域。

+   用户输入了一个会生成信号的终端特殊字符。这些字符包括*中断*字符（通常是*Control-C*）和*挂起*字符（通常是*Control-Z*）。

+   发生了软件事件。例如，文件描述符上有了输入，终端窗口被调整大小，定时器触发，进程的 CPU 时间限制被超出，或者该进程的一个子进程终止。

每个信号都被定义为一个唯一的（小的）整数，从 1 开始按顺序排列。这些整数在`<signal.h>`中定义，并具有`SIGxxxx`格式的符号名称。由于每个信号使用的实际数字在不同的实现中有所不同，因此程序中总是使用这些符号名称。例如，当用户输入*中断*字符时，`SIGINT`（信号号 2）会被送达进程。

信号可以分为两大类。第一类是*传统的*或*标准的*信号，用于内核通知进程事件。在 Linux 系统中，标准信号的编号从 1 到 31。我们将在本章描述标准信号。另一类信号是*实时*信号，标准信号和实时信号的区别在 22.8 节中有详细描述。

信号被称为由某些事件*生成*。一旦生成，信号会稍后*送达*进程，进程将根据该信号采取相应的行动。在信号生成和送达之间，信号被称为*挂起*。

通常，当一个挂起的信号被送达时，它会在进程下一次调度运行时立即送达，或者如果进程已经在运行（例如进程向自己发送信号），则会立刻送达。然而，有时我们需要确保某段代码在信号传递过程中不被中断。为了做到这一点，我们可以将信号添加到进程的*信号屏蔽*中——这是一组当前*被阻塞*的信号。如果在信号被阻塞时生成了信号，它会保持挂起状态，直到以后解除阻塞（从信号屏蔽中移除）。各种系统调用允许进程添加或移除信号屏蔽中的信号。

在信号传递时，进程根据信号执行以下默认操作之一：

+   信号被*忽略*；即它被内核丢弃，对进程没有任何影响。（进程甚至不知道信号的发生。）

+   进程被*终止*（杀死）。这有时被称为*异常进程终止*，与使用*exit()*终止进程时的正常终止相对。

+   生成一个*核心转储文件*，并且进程被终止。核心转储文件包含进程的虚拟内存映像，可以加载到调试器中，以便检查进程终止时的状态。

+   进程被*停止*——进程的执行被暂停。

+   在进程被暂停后，执行被*恢复*。

程序可以通过改变信号到达时的处理方式，而不接受特定信号的默认处理方式。这被称为设置信号的*处置*。程序可以为信号设置以下几种处置方式：

+   应该执行*默认操作*。这对于撤销之前将信号处置更改为非默认行为时非常有用。

+   信号被*忽略*。对于默认行为是终止进程的信号，这种做法非常有用。

+   执行一个*信号处理程序*。

信号处理程序是由程序员编写的函数，用于响应信号的传递并执行适当的任务。例如，shell 有一个处理 `SIGINT` 信号的处理程序（该信号由 *中断* 字符 *Control-C* 生成），它使得 shell 停止当前操作并将控制权返回到主输入循环，从而使用户重新看到 shell 提示符。通知内核应调用某个处理程序函数通常被称为 *安装* 或 *建立* 信号处理程序。当信号处理程序在响应信号传递时被调用时，我们称该信号已被 *处理* 或同义地称为 *捕获*。

注意，不能将信号的处置设置为 *终止* 或 *生成核心转储*（除非其中之一是该信号的默认处置方式）。我们最接近的方法是安装一个处理程序，处理程序调用 *exit()* 或 *abort()* 函数。*abort()* 函数（异常终止进程：*abort()*")）为进程生成一个 `SIGABRT` 信号，导致进程生成核心转储并终止。

### 注意

Linux 特定的 `/proc/`*PID*`/status` 文件包含了各种位掩码字段，可以用来检查进程如何处理信号。这些位掩码以十六进制数字显示，最低有效位表示信号 1，左边的下一个位表示信号 2，依此类推。这些字段包括 *SigPnd*（每线程待处理信号）、*ShdPnd*（进程范围内的待处理信号；自 Linux 2.6 起）、*SigBlk*（被阻塞的信号）、*SigIgn*（被忽略的信号）和 *SigCgt*（已捕获的信号）。(我们将在第 33.2 节中描述多线程进程中信号的处理时，*SigPnd* 和 *ShdPnd* 字段之间的区别将变得更加清晰。) 使用 *ps(1)* 命令的各种选项也可以获得相同的信息。

信号在早期的 UNIX 实现中就已出现，但自其诞生以来经历了一些重大变化。在早期实现中，信号可能会丢失（即，在某些情况下不会传递给目标进程）。此外，尽管提供了阻止信号传递的设施，以便在执行关键代码时保护代码，但在某些情况下，这种阻止机制并不可靠。这些问题在 4.2BSD 中得到了修复，4.2BSD 提供了所谓的 *可靠信号*。（另一个 BSD 创新是增加了额外的信号以支持 shell 作业控制，我们将在第 34.7 节中描述这一点。）

System V 也为信号添加了可靠的语义，但采用了一种与 BSD 不兼容的模型。这些不兼容之处直到 POSIX.1-1990 标准发布时才得以解决，该标准采用了基于 BSD 模型的大部分可靠信号规范。

我们在 signal()的实现与可移植性的实现与可移植性")中考虑了可靠信号和不可靠信号的详细信息，并在早期信号 API（System V 和 BSD）中简要描述了旧的 BSD 和 System V 信号 API。

## 信号类型和默认动作

之前我们提到，标准信号在 Linux 中编号从 1 到 31。然而，Linux 的*signal(7)*手册页面列出了超过 31 个信号名称。超出的信号名称有多种解释方式。其中一些名称只是其他名称的同义词，为了与其他 UNIX 实现的源兼容而定义。其他名称则是定义了但未使用的。以下列表描述了各种信号：

`SIGABRT`

当进程调用*abort()*函数时，会发送此信号（异常终止进程：*abort()*")）。默认情况下，此信号会终止进程并生成核心转储。此操作实现了*abort()*调用的预期目的：生成用于调试的核心转储。

`SIGALRM`

内核在由*alarm()*或*setitimer()*调用设置的实时计时器到期时生成此信号。实时计时器是按照墙钟时间（即人类的时间概念）计数的计时器。更多详细信息，请参见第 23.1 节。

`SIGBUS`

此信号（“总线错误”）是为了表示某些类型的内存访问错误而生成的。例如，使用* mmap()*创建的内存映射时，如果尝试访问超出底层内存映射文件末尾的地址，就可能发生此类错误，具体内容请参见边界情况。

`SIGCHLD`

当一个子进程终止时（无论是通过调用*exit()*，还是由于被信号终止），内核会向父进程发送此信号（由内核发送）。当子进程被信号停止或恢复时，内核也会发送此信号。我们在第 26.3 节中详细讨论了`SIGCHLD`。

`SIGCLD`

这是`SIGCHLD`的同义词。

`SIGCONT`

当发送到一个停止的进程时，此信号会导致该进程恢复（即重新调度并在稍后的时间运行）。当接收到一个当前未停止的进程时，此信号默认会被忽略。进程可以捕获此信号，以便在恢复时执行某些操作。此信号在交付、处置和处理的特殊情况和作业控制中有更详细的介绍。

`SIGEMT`

在一般的 UNIX 系统中，此信号用于表示与实现相关的硬件错误。在 Linux 中，仅在 Sun SPARC 实现中使用此信号。后缀`EMT`来源于*模拟器陷阱*，这是 Digital PDP-11 上的汇编语言助记符。

`SIGFPE`

该信号由某些类型的算术错误生成，例如除以零。后缀`FPE`是*浮点异常*的缩写，尽管此信号也可以因整数算术错误而生成。此信号生成的精确细节取决于硬件架构和 CPU 控制寄存器的设置。例如，在 x86-32 架构上，整数除以零总是会产生`SIGFPE`，但浮点数除以零的处理则取决于是否启用了`FE_DIVBYZERO`异常。如果启用了此异常（使用*feenableexcept()*），则浮点数除以零会生成`SIGFPE`；否则，它会生成操作数的 IEEE 标准结果（浮点表示的无穷大）。有关详细信息，请参见*fenv(3)*手册页面和`<fenv.h>`。

`SIGHUP`

当终端断开连接（挂起）时，系统会向终端的控制进程发送此信号。我们在第 34.6 节中描述了控制进程的概念以及`SIGHUP`信号发送的各种情况。`SIGHUP`的第二种用途是与守护进程（例如*init*、*httpd*和*inetd*）一起使用。许多守护进程在收到`SIGHUP`信号时会通过重新初始化自己并重新读取配置文件来响应。系统管理员可以通过手动向守护进程发送`SIGHUP`，无论是通过显式的*kill*命令，还是通过执行一个执行相同操作的程序或脚本，来触发这些操作。

`SIGILL`

当一个进程尝试执行非法（即格式错误的）机器语言指令时，会向其发送此信号。

`SIGINFO`

在 Linux 中，此信号名称是`SIGPWR`的同义词。在 BSD 系统中，通过输入*Control-T*生成的`SIGINFO`信号用于获取前台进程组的状态信息。

`SIGINT`

当用户输入终端*中断*字符（通常是*Control-C*）时，终端驱动程序会将此信号发送给前台进程组。此信号的默认动作是终止进程。

`SIGIO`

使用*fcntl()*系统调用，可以安排在某些类型的打开文件描述符（如终端和套接字）上发生 I/O 事件（例如输入变得可用）时生成此信号。此功能在第 63.3 节中有进一步的描述。

`SIGIOT`

在 Linux 中，这是`SIGABRT`的同义词。在某些其他 UNIX 实现中，此信号表示实现定义的硬件故障。

`SIGKILL`

这是确定的*终止*信号。它无法被阻止、忽略或被处理程序捕获，因此始终会终止一个进程。

`SIGLOST`

这个信号名称在 Linux 上存在，但未被使用。在一些其他 UNIX 实现中，如果 NFS 客户端在恢复崩溃的远程 NFS 服务器后未能重新获取由本地进程持有的锁，则 NFS 客户端会向这些本地进程发送此信号。（此功能在 NFS 规范中未标准化。）

`SIGPIPE`

当一个进程尝试写入一个没有对应读取进程的管道、FIFO 或套接字时，会生成此信号。通常，这是因为读取进程已经关闭了该 IPC 通道的文件描述符。有关详细信息，请参见创建和使用管道。

`SIGPOLL`

这个信号源自 System V，在 Linux 上是`SIGIO`的同义词。

`SIGPROF`

当由*setitimer()*调用设置的性能计时器到期时，内核会生成此信号（Interval Timers）。性能计时器是用来计数进程使用的 CPU 时间的计时器。与虚拟计时器（见`SIGVTALRM`）不同，性能计时器计数在用户模式和内核模式下使用的 CPU 时间。

`SIGPWR`

这是*电源故障*信号。在具有不间断电源供应（UPS）的系统中，可以设置一个守护进程，监控电池备份电量，以应对电源故障。如果电池电量即将耗尽（经过一段时间的停电），则监控进程会向*init*进程发送`SIGPWR`信号，*init*进程将该信号解释为请求快速而有序地关闭系统。

`SIGQUIT`

当用户在键盘上输入*quit*字符（通常是*Control-\*），该信号会发送到前台进程组。默认情况下，这个信号会终止一个进程，并导致其生成核心转储（core dump），然后可以用来调试。以这种方式使用`SIGQUIT`在程序陷入无限循环或无法响应时非常有用。通过输入*Control-\*，然后使用*gdb*调试器加载生成的核心转储，并使用*backtrace*命令获取堆栈跟踪，我们可以找出程序代码的哪一部分正在执行。（[Matloff, 2008]描述了如何使用*gdb*。）

`SIGSEGV`

这个非常常见的信号是在程序做出无效内存引用时生成的。内存引用可能是无效的，因为引用的页面不存在（例如，它位于堆栈和堆之间的未映射区域），进程试图更新只读内存中的位置（例如，程序文本段或标记为只读的映射内存区域），或者进程试图在用户模式下访问内核内存的一部分（参见核心操作系统：内核）。在 C 语言中，这些事件通常是由于解引用包含错误地址（例如，未初始化的指针）或在函数调用中传递无效参数所导致。该信号的名称源自术语*段错误*。

`SIGSTKFLT`

在*signal(7)*中记录为“协处理器上的堆栈故障”，该信号已定义，但在 Linux 上未使用。

`SIGSTOP`

这是*强制停止*信号。它不能被阻止、忽略或通过处理程序捕获；因此，它总是会停止一个进程。

`SIGSYS`

如果一个进程发出了“错误”的系统调用，将会生成此信号。这意味着进程执行了一条被解释为系统调用陷阱的指令，但关联的系统调用编号无效（参见系统调用）。

`SIGTERM`

这是终止进程的标准信号，也是*kill*和*killall*命令默认发送的信号。用户有时会显式地向进程发送`SIGKILL`信号，例如通过*kill -KILL*或*kill -9*命令。然而，这通常是一个错误。设计良好的应用程序会为`SIGTERM`设置一个处理程序，使应用程序优雅地退出，提前清理临时文件并释放其他资源。使用`SIGKILL`终止进程会绕过`SIGTERM`处理程序。因此，我们应该始终首先尝试使用`SIGTERM`终止进程，并将`SIGKILL`保留为最后的手段，用于杀死那些未响应`SIGTERM`的失控进程。

`SIGTRAP`

该信号用于实现调试器断点和系统调用追踪，例如*strace(1)*所执行的操作（附录 A）。有关更多信息，请参阅*ptrace(2)*手册页。

`SIGTSTP`

这是作业控制的*停止*信号，当用户在键盘上输入*suspend*字符（通常是*Control-Z*）时，发送该信号以停止前台进程组。第三十四章详细描述了进程组（作业）和作业控制，以及程序何时和如何处理该信号的细节。该信号的名称源自“终端停止”。

`SIGTTIN`

在作业控制的 shell 中运行时，当后台进程组试图从终端进行*read()*操作时，终端驱动程序会向其发送此信号。此信号默认会停止进程。

`SIGTTOU`

这个信号与`SIGTTIN`的作用类似，但用于后台作业的终端输出。当在作业控制 Shell 下运行时，如果启用了`TOSTOP`（*终端输出停止*）选项（可能通过命令*stty tostop*启用），当终端驱动程序尝试向终端写入时，它会向后台进程组发送`SIGTTOU`（参见在 Shell 中使用作业控制）。此信号默认会停止进程。

`SIGUNUSED`

如名称所示，这个信号未被使用。在 Linux 2.4 及更高版本中，这个信号的名称与`SIGSYS`在许多架构上是同义的。换句话说，这个信号编号在这些架构上不再是未使用的，尽管为了向后兼容，信号名称仍然存在。

`SIGURG`

这个信号会发送到进程，表示套接字上存在*带外*（也称为*紧急*）数据（带外数据）。

`SIGUSR1`

这个信号和`SIGUSR2`可用于程序员定义的目的。内核从不为进程生成这些信号。进程可以使用这些信号来通知彼此事件或进行同步。在早期的 UNIX 实现中，这两个信号是唯一可以在应用程序中自由使用的信号。（实际上，进程可以向彼此发送任何信号，但如果内核也为进程生成了某个信号，这可能会导致混淆。）现代 UNIX 实现提供了一大组实时信号，这些信号也可用于程序员定义的目的（实时信号）。

`SIGUSR2`

参见`SIGUSR1`的描述。

`SIGVTALRM`

内核在虚拟定时器过期时生成此信号，虚拟定时器是通过调用*setitimer()*设置的（间隔定时器）。虚拟定时器是计算进程使用的用户模式 CPU 时间的定时器。

`SIGWINCH`

在窗口环境中，当终端窗口大小变化时，向前台进程组发送此信号（可能是由于用户手动调整大小，或者程序通过调用*ioctl()*调整大小，如在终端窗口大小中所述）。通过为此信号安装处理程序，程序（如*vi*和*less*）可以在窗口大小更改后知道重新绘制它们的输出。

`SIGXCPU`

当进程超过其 CPU 时间资源限制（`RLIMIT_CPU`，在具体资源限制的详细信息中描述）时，发送此信号。

`SIGXFSZ`

如果进程尝试通过 *write()* 或 *truncate()* 操作将文件大小增加到超过进程的文件大小资源限制（`RLIMIT_FSIZE`，在特定资源限制的详细信息中描述），则该信号会发送给该进程。

表格 20-1 汇总了有关 Linux 信号的一系列信息。请注意以下几点：

+   *信号编号* 列显示在各种硬件架构上分配给该信号的编号。除非另有说明，否则所有架构上的信号编号是相同的。架构间信号编号的差异会在括号中指明，并出现在 Sun SPARC 和 SPARC64 (S)、HP/Compaq/Digital Alpha (A)、MIPS (M) 以及 HP PA-RISC (P) 架构中。在此列中，*undef* 表示在指定架构上符号未定义。

+   *SUSv3* 列表示该信号是否在 SUSv3 中进行了标准化。

+   *默认行为* 列表示信号的默认操作：*term* 表示信号终止进程，*core* 表示进程生成核心转储文件并终止，*ignore* 表示忽略该信号，*stop* 表示信号停止进程，*cont* 表示信号恢复停止的进程。

### 注意

之前列出的某些信号未显示在表格 20-1 中：`SIGCLD`（`SIGCHLD`的同义词），`SIGINFO`（未使用），`SIGIOT`（`SIGABRT`的同义词），`SIGLOST`（未使用），以及 `SIGUNUSED`（在许多架构上是 `SIGSYS` 的同义词）。

表格 20-1. Linux 信号

| 名称 | 信号编号 | 描述 | SUSv3 | 默认行为 |
| --- | --- | --- | --- | --- |
| `SIGABRT` | 6 | 中止进程 | • | core |
| `SIGALRM` | 14 | 实时定时器超时 | • | term |
| `SIGBUS` | 7 (SAMP=10) | 内存访问错误 | • | core |
| `SIGCHLD` | 17 (SA=20, MP=18) | 子进程终止或停止 | • | ignore |
| `SIGCONT` | 18 (SA=19, M=25, P=26) | 如果停止，则继续 | • | cont |
| `SIGEMT` | undef (SAMP=7) | 硬件故障 |   | term |
| `SIGFPE` | 8 | 算术异常 | • | core |
| `SIGHUP` | 1 | 挂起 | • | term |
| `SIGILL` | 4 | 非法指令 | • | core |
| `SIGINT` | 2 | 终端中断 | • | term |
| `SIGIO`/`SIGPOLL` | 29 (SA=23, MP=22) | I/O 可用 | • | term |
| `SIGKILL` | 9 | 强制终止 | • | term |
| `SIGPIPE` | 13 | 管道破损 | • | term |
| `SIGPROF` | 27 (M=29, P=21) | 性能分析定时器超时 | • | term |
| `SIGPWR` | 30 (SA=29, MP=19) | 电源即将故障 |   | term |
| `SIGQUIT` | 3 | 终端退出 | • | core |
| `SIGSEGV` | 11 | 无效的内存引用 | • | core |
| `SIGSTKFLT` | 16 (SAM=undef, P=36) | 协处理器栈故障 |   | term |
| `SIGSTOP` | 19 (SA=17, M=23, P=24) | 确定停止 | • | stop |
| `SIGSYS` | 31 (SAMP=12) | 无效的系统调用 | • | core |
| `SIGTERM` | 15 | 终止进程 | • | term |
| `SIGTRAP` | 5 | 跟踪/断点陷阱 | • | 核心 |
| `SIGTSTP` | 20 (SA=18, M=24, P=25) | 终端停止 | • | 停止 |
| `SIGTTIN` | 21 (M=26, P=27) | 来自后台的终端读操作 | • | 停止 |
| `SIGTTOU` | 22 (M=27, P=28) | 来自后台的终端写操作 | • | 停止 |
| `SIGURG` | 23 (SA=16, M=21, P=29) | 套接字上有紧急数据 | • | 忽略 |
| `SIGUSR1` | 10 (SA=30, MP=16) | 用户定义信号 1 | • | 终止 |
| `SIGUSR2` | 12 (SA=31, MP=17) | 用户定义信号 2 | • | 终止 |
| `SIGVTALRM` | 26 (M=28, P=20) | 虚拟定时器到期 | • | 终止 |
| `SIGWINCH` | 28 (M=20, P=23) | 终端窗口大小更改 |   | 忽略 |
| `SIGXCPU` | 24 (M=30, P=33) | 超过 CPU 时间限制 | • | 核心 |
| `SIGXFSZ` | 25 (M=31, P=34) | 文件大小超限 | • | 核心 |

请注意以下关于某些信号默认行为的说明，详见 表 20-1：

+   在 Linux 2.2 中，信号 `SIGXCPU`、`SIGXFSZ`、`SIGSYS` 和 `SIGBUS` 的默认动作是终止进程并不产生核心转储。从内核 2.4 版本开始，Linux 遵循 SUSv3 的要求，导致这些信号会导致进程终止并生成核心转储。在其他一些 UNIX 实现中，`SIGXCPU` 和 `SIGXFSZ` 的处理方式与 Linux 2.2 相同。

+   `SIGPWR` 通常在这些其他 UNIX 实现中默认被忽略，其中它会出现。

+   `SIGIO` 在多个 UNIX 实现中默认被忽略（尤其是 BSD 衍生版本）。

+   尽管没有任何标准明确规定，`SIGEMT` 出现在大多数 UNIX 实现中。然而，在其他实现中，这个信号通常会导致终止并生成核心转储。

+   在 SUSv1 中，`SIGURG` 的默认动作是进程终止，这也是一些旧版本 UNIX 实现中的默认行为。SUSv2 采用了当前的规范（忽略）。

## 更改信号处理方式：*signal()*

UNIX 系统提供了两种更改信号处置的方法：*signal()*和*sigaction()*。*signal()*系统调用，如本节所述，是设置信号处置的原始 API，它提供了比*sigaction()*更简单的接口。另一方面，*sigaction()*提供了*signal()*无法提供的功能。此外，由于*signal()*在不同 UNIX 实现中的行为存在差异（*signal()*的实现与可移植性")），因此它不应在可移植程序中用于设置信号处理程序。由于这些可移植性问题，*sigaction()*是（强烈）推荐的 API，用于建立信号处理程序。在我们解释了更改信号处置：*sigaction()*")中的*sigaction()*的使用后，我们将在示例程序中始终使用该调用来建立信号处理程序。

### 注意

尽管在 Linux 手册页的第二部分中有文档说明，*signal()*实际上是在*glibc*中作为一个库函数实现的，该库函数是在*sigaction()*系统调用之上封装的。

```
#include <signal.h>

void ( *`signal`(int *sig*, void (**handler*)(int)) ) (int);
```

### 注意

如果成功，则返回先前的信号处置，出错时返回`SIG_ERR`

*signal()*的函数原型需要一些解码。第一个参数，*sig*，标识我们希望更改其处置的信号。第二个参数，*handler*，是当信号被传递时应该调用的函数的地址。该函数不返回任何值（*void*），并且接受一个整数参数。因此，信号处理程序具有以下通用形式：

```
void
handler(int sig)
{
    /* Code for the handler */
}
```

我们将在第 20.4 节中描述*sig*参数的用途，传递给处理程序函数。

*signal()*的返回值是信号的先前处置。像*handler*参数一样，这是一个指向返回空值并接受一个整数参数的函数的指针。换句话说，我们可以编写如下代码来暂时建立一个信号处理程序，然后将信号的处置重置为其之前的状态：

```
void (*oldHandler)(int);

oldHandler = signal(SIGINT, newHandler);
if (oldHandler == SIG_ERR)
    errExit("signal");

/* Do something else here. During this time, if SIGINT is
   delivered, newHandler will be used to handle the signal. */

if (signal(SIGINT, oldHandler) == SIG_ERR)
    errExit("signal");
```

### 注意

不可能使用*signal()*来检索信号的当前处置，而不同时改变该处置。要做到这一点，我们必须使用*sigaction()*。

我们可以通过使用以下类型定义为指向信号处理程序函数的指针，来使*signal()*的原型更易理解：

```
typedef void (*sighandler_t)(int);
```

这使我们能够将*signal()*的原型重写为如下形式：

```
sighandler_t signal(int sig, sighandler_t handler);
```

### 注意

如果定义了`_GNU_SOURCE`特性测试宏，那么*glibc*会在`<signal.h>`头文件中暴露非标准的*sighandler_t*数据类型。

我们可以指定以下值之一来替代作为*signal()*的*handler*参数指定一个函数的地址：

`SIG_DFL`

将信号的处理方式重置为默认值（表 20-1）。这对于撤销先前调用 *signal()* 更改信号处理方式的效果非常有用。

`SIG_IGN`

忽略信号。如果为此进程生成信号，内核会静默丢弃它。该进程甚至不知道信号已经发生。

成功调用 *signal()* 会返回信号的先前处理方式，这可能是一个先前安装的处理程序函数的地址，或者是常量 `SIG_DFL` 或 `SIG_IGN`。如果出错，*signal()* 会返回 `SIG_ERR`。

## 信号处理程序介绍

*信号处理程序*（也称为*信号捕捉器*）是一个函数，当指定的信号传递给进程时会被调用。本节中我们介绍信号处理程序的基本原理，然后在第二十一章中详细讨论。

信号处理程序的调用可能会在任何时候打断主程序的执行流；内核代表进程调用处理程序，当处理程序返回时，程序会从中断它的地方继续执行。此过程在图 20-1 中进行了说明。

![信号传递和处理程序执行](img/20-1_SIG-A-handling.png.jpg)图 20-1. 信号传递和处理程序执行

尽管信号处理程序几乎可以做任何事情，但一般来说，它们应该设计得尽可能简单。我们在第 21.1 节中扩展了这一点。

示例 20-1. 安装 `SIGINT` 的处理程序

```
`signals/ouch.c`
#include <signal.h>
#include "tlpi_hdr.h"

static void
sigHandler(int sig)
{
    printf("Ouch!\n");                  /* UNSAFE (see Section 21.1.2) */
}

int
main(int argc, char *argv[])
{
    int j;

    if (signal(SIGINT, sigHandler) == SIG_ERR)
        errExit("signal");

    for (j = 0; ; j++) {
        printf("%d\n", j);
        sleep(3);                       /* Loop slowly... */
    }
}
     `signals/ouch.c`
```

示例 20-1（在发送信号：*kill()*")中）展示了一个简单的信号处理程序函数示例和一个主程序，该程序将其作为 `SIGINT` 信号的处理程序。（当我们输入终端的*中断*字符，通常是*控制-C*时，终端驱动程序会生成此信号。）该处理程序仅打印一条消息并返回。

主程序不断循环。在每次迭代时，程序增加一个计数器并打印其值，然后程序休眠几秒钟。（为了以这种方式休眠，我们使用*sleep()*函数，它会暂停调用者的执行一段指定的时间。我们在低分辨率睡眠：*sleep()*")中描述了这个函数。）

当我们运行示例 20-1 中的程序时，我们会看到以下内容：

```
$ `./ouch`
0                         *Main program loops, displaying successive integers*
*Type Control-C*
Ouch!                     *Signal handler is executed, and returns*
1                         *Control has returned to main program*
2
*Type Control-C again*
Ouch!
3
*Type Control-\ (the terminal quit character)*
Quit (core dumped)
```

当内核调用信号处理程序时，它将导致调用的信号编号作为整数参数传递给处理程序。（这就是 示例 20-1 中的 *sig* 参数）。如果一个信号处理程序只捕获一种类型的信号，那么这个参数的作用就不大。然而，我们可以建立相同的处理程序来捕获不同类型的信号，并利用这个参数来确定是哪种信号导致了处理程序的调用。

这一点在 示例 20-2 中进行了说明，这是一个为 `SIGINT` 和 `SIGQUIT` 建立相同处理程序的程序。(`SIGQUIT` 是当我们输入终端的 *quit* 字符时，由终端驱动程序生成的信号，通常是 *Control-\*.) 处理程序的代码通过检查 *sig* 参数来区分这两个信号，并对每个信号采取不同的动作。在 *main()* 函数中，我们使用 *pause()*（在 等待信号：*pause()*") 中描述）来阻塞进程，直到捕获到信号。

以下是一个 shell 会话日志，演示了这个程序的使用：

```
$ `./intquit`
*Type Control-C*
Caught SIGINT (1)
*Type Control-C again*
Caught SIGINT (2)
*and again*
Caught SIGINT (3)
*Type Control-\*
Caught SIGQUIT - that's all folks!
```

在 示例 20-1 和 示例 20-2 中，我们使用 *printf()* 来显示信号处理程序的消息。由于我们在 可重入和异步信号安全函数 中讨论过的原因，现实中的应用程序通常不应从信号处理程序中调用 stdio 函数。然而，在各种示例程序中，我们仍然会从信号处理程序中调用 *printf()*，作为一种简单的方式来查看何时调用了处理程序。

示例 20-2. 为两个不同信号建立相同的处理程序

```
`signals/intquit.c`
#include <signal.h>
#include "tlpi_hdr.h"

static void
sigHandler(int sig)
{
    static int count = 0;

    /* UNSAFE: This handler uses non-async-signal-safe functions
       (printf(), exit(); see Section 21.1.2) */

    if (sig == SIGINT) {
        count++;
        printf("Caught SIGINT (%d)\n", count);
        return;                 /* Resume execution at point of interruption */
    }

    /* Must be SIGQUIT - print a message and terminate the process */

    printf("Caught SIGQUIT - that's all folks!\n");
    exit(EXIT_SUCCESS);
}

int
main(int argc, char *argv[])
{
    /* Establish same handler for SIGINT and SIGQUIT */

    if (signal(SIGINT, sigHandler) == SIG_ERR)
        errExit("signal");
    if (signal(SIGQUIT, sigHandler) == SIG_ERR)
        errExit("signal");

    for (;;)                    /* Loop forever, waiting for signals */
        pause();                /* Block until a signal is caught */
}
      `signals/intquit.c`
```

## 发送信号：*kill()*

一个进程可以使用 *kill()* 系统调用向另一个进程发送信号，这与 *kill* shell 命令类似。（之所以选择 *kill* 这个术语，是因为在早期 UNIX 实现中，大多数可用信号的默认动作是终止进程。）

```
#include <signal.h>

int `kill`(pid_t *pid*, int *sig*);
```

### 注意

成功时返回 0，错误时返回 -1

*pid* 参数标识要发送指定 *sig* 信号的一个或多个进程。四种不同的情况决定了如何解释 *pid*：

+   如果 *pid* 大于 0，信号将发送到由 *pid* 指定的进程 ID 的进程。

+   如果*pid*等于 0，信号会发送到与调用进程属于同一进程组的所有进程，包括调用进程本身。（SUSv3 规定，信号应发送到同一进程组的所有进程，但排除了“未指定的一组系统进程”，并对其余情况做出相同的规定。）

+   如果*pid*小于-1，信号会发送到进程组 ID 等于*pid*绝对值的所有进程。向进程组中的所有进程发送信号在 Shell 作业控制中有特定用途（作业控制）。

+   如果*pid*等于-1，信号会发送到调用进程有权限发送信号的所有进程，但不包括*init*（进程 ID 1）和调用进程。如果是特权进程发出此调用，则系统上的所有进程都会收到信号，除了这两个进程。出于显而易见的原因，这种方式发送的信号有时被称为*广播信号*。（SUSv3 并不要求排除调用进程接收信号；在这方面，Linux 遵循 BSD 的语义。）

如果没有进程与指定的*pid*匹配，*kill()*会失败，并将*errno*设置为`ESRCH`（“没有这样的进程”）。

进程需要适当的权限才能向另一个进程发送信号。权限规则如下：

+   一个特权（`CAP_KILL`）进程可以向任何进程发送信号。

+   *init*进程（进程 ID 1），以*root*的用户和组身份运行，是一个特例。它只能接收有安装处理程序的信号。这可以防止系统管理员不小心终止*init*进程，因为它对于系统的运行至关重要。

+   如果发送进程的真实用户 ID 或有效用户 ID 与接收进程的真实用户 ID 或保存的设置用户 ID 匹配，则一个非特权进程可以向另一个进程发送信号，如图 20-2 所示。此规则允许用户向他们启动的设置用户 ID 程序发送信号，而不管目标进程的当前有效用户 ID 设置如何。将目标进程的有效用户 ID 排除在检查之外，具有补充作用：它防止一个用户向另一个用户的进程发送信号，后者正在运行一个属于尝试发送信号的用户的设置用户 ID 程序。（SUSv3 强制执行图 20-2 中显示的规则，但 Linux 在 2.0 版本之前的内核中遵循了略有不同的规则，具体见*kill(2)*手册页。）

+   `SIGCONT` 信号有特殊处理。普通进程可以向同一会话中的任何其他进程发送此信号，而不需要检查用户 ID。这一规则允许作业控制 shell 重启停止的作业（进程组），即使作业中的进程已更改其用户 ID（即它们是使用检索和修改进程凭据中描述的系统调用来更改凭据的特权进程").

![普通进程发送信号所需的权限](img/20-2_SIG-A-perms-scale90.png.jpg)图 20-2. 普通进程发送信号所需的权限

如果进程没有权限向指定的*pid* 发送信号，则 *kill()* 调用失败，并将 *errno* 设置为 `EPERM`。当*pid* 指定一组进程（即*pid* 为负数）时，如果至少有一个进程能够接收信号，则 *kill()* 调用成功。

我们在示例 20-3 系统调用")中演示了如何使用 *kill()*。

## 检查进程是否存在

*kill()* 系统调用可以用于另一个目的。如果*sig* 参数指定为 0（所谓的*空信号*），则不会发送任何信号。相反，*kill()* 仅执行错误检查，以查看是否可以向进程发送信号。换句话说，我们可以使用空信号来测试某个特定进程 ID 的进程是否存在。如果发送空信号时出现错误`ESRCH`，则说明该进程不存在。如果调用失败并出现错误`EPERM`（意味着进程存在，但我们没有权限向其发送信号）或成功（意味着我们有权限向该进程发送信号），则说明该进程存在。

验证某个特定进程 ID 是否存在并不能保证该程序仍在运行。因为内核在进程出生和死亡时会回收进程 ID，随着时间的推移，相同的进程 ID 可能会指向一个不同的进程。此外，某个特定进程 ID 可能存在，但它是一个僵尸进程（即已终止的进程，但其父进程尚未执行*wait()* 来获取其终止状态，详见孤儿进程和僵尸进程）。

还可以使用其他各种技术来检查某个特定进程是否在运行，包括以下方法：

+   *wait()* 系统调用：这些调用在第二十六章中有描述。它们只能在被监控的进程是调用者的子进程时使用。

+   *信号量和独占文件锁*：如果被监控的进程持续持有一个信号量或文件锁，那么，如果我们能够获取该信号量或锁，就可以知道该进程已终止。我们在第四十七章和第五十三章中描述了信号量，在第五十五章中描述了文件锁。

+   *IPC 通道，如管道和 FIFO*：我们设置被监控的进程，使其在存活期间始终保持一个用于写入的文件描述符打开。与此同时，监控进程保持一个读取描述符打开，并且当通道的写入端关闭时（因为它看到文件结束），它就知道被监控进程已经终止。监控进程可以通过读取文件描述符或使用第六十三章中描述的技术来监视该描述符，从而确定这一点。

+   *`/proc/`PID 接口*：例如，如果存在进程 ID 为 12345 的进程，那么目录 `/proc/12345` 会存在，我们可以使用 *stat()* 等调用来检查这一点。

除了最后一种方法，所有这些技术都不受进程 ID 回收的影响。

示例 20-3 系统调用") 演示了 *kill()* 的使用。这个程序接受两个命令行参数，一个是信号编号，另一个是进程 ID，并使用 *kill()* 将信号发送到指定的进程。如果指定信号 0（空信号），程序会报告目标进程是否存在。

## 发送信号的其他方式：*raise()* 和 *killpg()*

有时，进程需要向自己发送信号。（我们在处理作业控制信号中看到过这个例子。）*raise()* 函数执行此任务。

```
#include <signal.h>

int `raise`(int *sig*);
```

### 注意

成功时返回 0，错误时返回非零值。

在单线程程序中，调用 *raise()* 等同于以下调用 *kill()*：

```
kill(getpid(), sig);
```

在支持线程的系统中，*raise(sig)* 的实现方式如下：

```
pthread_kill(pthread_self(), sig)
```

我们在发送信号到线程中描述了 *pthread_kill()* 函数，但现在可以简单地说，这个实现意味着信号将被发送到调用 *raise()* 的特定线程。相反，调用 *kill(getpid(), sig)* 会将信号发送到调用的 *进程*，该信号可能会被传递到进程中的任何线程。

### 注意

*raise()* 函数源自 C89。C 标准没有涵盖操作系统的细节，如进程 ID，但 *raise()* 可以在 C 标准中指定，因为它不需要引用进程 ID。

当一个进程使用 *raise()*（或 *kill()*）向自己发送信号时，信号会立即传递（即在 *raise()* 返回给调用者之前）。

请注意，*raise()* 在出错时会返回一个非零值（不一定是 -1）。*raise()* 唯一可能发生的错误是 `EINVAL`，因为 *sig* 无效。因此，在我们指定一个 `SIGxxxx` 常量时，通常不检查此函数的返回状态。

示例 20-3. 使用 *kill()* 系统调用

```
`signals/t_kill.c`
#include <signal.h>
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    int s, sig;

    if (argc != 3 || strcmp(argv[1], "--help") == 0)
        usageErr("%s sig-num pid\n", argv[0]);

    sig = getInt(argv[2], 0, "sig-num");

    s = kill(getLong(argv[1], 0, "pid"), sig);

    if (sig != 0) {
        if (s == -1)
            errExit("kill");

    } else {                    /* Null signal: process existence check */
        if (s == 0) {
            printf("Process exists and we can send it a signal\n");
        } else {
            if (errno == EPERM)
                printf("Process exists, but we don't have "
                       "permission to send it a signal\n");
            else if (errno == ESRCH)
                printf("Process does not exist\n");
            else
                errExit("kill");
        }
    }

    exit(EXIT_SUCCESS);
}
      `signals/t_kill.c`
```

*killpg()* 函数会向进程组中的所有成员发送信号。

```
#include <signal.h>

int `killpg`(pid_t *pgrp*, int *sig*);
```

### 注意

成功时返回 0，出错时返回 -1

调用 *killpg()* 等同于以下调用 *kill()*：

```
kill(-pgrp, sig);
```

如果 *pgrp* 被指定为 0，则信号会发送给与调用者处于同一进程组的所有进程。SUSv3 没有对这一点做出说明，但大多数 UNIX 实现将此情况与 Linux 解释相同。

## 显示信号描述

每个信号都有一个关联的可打印描述。这些描述列在 *sys_siglist* 数组中。例如，我们可以通过 *sys_siglist[SIGPIPE]* 来获取 `SIGPIPE`（管道破裂）的描述。然而，与直接使用 *sys_siglist* 数组相比，使用 *strsignal()* 函数更为推荐。

```
#define _BSD_SOURCE
#include <signal.h>

extern const char *const `sys_siglist`[];

#define _GNU_SOURCE
#include <string.h>

char *`strsignal`(int *sig*);
```

### 注意

返回指向信号描述字符串的指针

*strsignal()* 函数对 *sig* 参数执行边界检查，然后返回指向信号可打印描述的指针，或者如果信号号码无效，则返回指向错误字符串的指针。（在一些其他 UNIX 实现中，如果 *sig* 无效，*strsignal()* 返回 `NULL`。）

除了边界检查之外，*strsignal()* 相对于直接使用 *sys_siglist* 的另一个优点是，*strsignal()* 是受区域设置影响的（区域设置），因此信号描述会以本地语言显示。

使用 *strsignal()* 的示例见 示例 20-4。

*psignal()* 函数会在标准错误输出上显示其参数 *msg* 中给定的字符串，后跟冒号，然后是与 *sig* 对应的信号描述。像 *strsignal()* 一样，*psignal()* 是受区域设置影响的。

```
#include <signal.h>

void `psignal`(int *sig*, const char **msg*);
```

尽管 *psignal()*、*strsignal()* 和 *sys_siglist* 并未在 SUSv3 中标准化，但它们在许多 UNIX 实现中是可用的。（SUSv4 为 *psignal()* 和 *strsignal()* 添加了规范。）

## 信号集

许多与信号相关的系统调用需要能够表示一组不同的信号。例如，*sigaction()* 和 *sigprocmask()* 允许程序指定一组信号，这些信号会被进程阻塞，而 *sigpending()* 返回一个当前正在等待的信号组。（我们稍后会描述这些系统调用。）

多个信号使用一种名为*信号集*的数据结构表示，该数据结构由系统数据类型*sigset_t*提供。SUSv3 规定了一系列用于操作信号集的函数，我们现在将介绍这些函数。

### 注意

在 Linux 上，与大多数 UNIX 实现一样，*sigset_t*数据类型是一个位掩码。然而，SUSv3 并不要求这一点。信号集也可以使用其他类型的结构来表示。SUSv3 只要求*sigset_t*的类型是可赋值的。因此，它必须使用标量类型（例如整数）或 C 结构体（可能包含整数数组）来实现。

*sigemptyset()*函数初始化一个信号集，使其不包含任何成员。*sigfillset()*函数初始化一个信号集，使其包含所有信号（包括所有实时信号）。

```
#include <signal.h>

int `sigemptyset`(sigset_t **set*);
int `sigfillset`(sigset_t **set*);
```

### 注意

成功时返回 0，出错时返回-1

必须使用*sigemptyset()*或*sigaddset()*来初始化信号集。这是因为 C 语言不会自动初始化变量，而静态变量初始化为 0 不能可靠地用于标记一个空的信号集，因为信号集可能会使用除位掩码之外的其他结构来实现。（出于同样的原因，使用*memset(3)*将信号集内容置零以标记其为空是不正确的。）

初始化后，可以使用*sigaddset()*将单个信号添加到集合中，使用*sigdelset()*将信号从集合中移除。

```
#include <signal.h>

int `sigaddset`(sigset_t **set*, int *sig*);
int `sigdelset`(sigset_t **set*, int *sig*);
```

### 注意

成功时返回 0，出错时返回-1

对于*sigaddset()*和*sigdelset()*，*sig*参数是一个信号编号。

*sigismember()*函数用于测试集合的成员资格。

```
#include <signal.h>

int `sigismember`(const sigset_t **set*, int *sig*);
```

### 注意

如果*sig*是*set*的成员，返回 1；否则返回 0

*sigismember()*函数如果*sig*是*set*的成员，返回 1（真）；否则返回 0（假）。

GNU C 库实现了三个非标准函数，这些函数执行的任务是标准信号集函数的补充。

```
#define _GNU_SOURCE
#include <signal.h>

int `sigandset`(sigset_t **dest*, sigset_t **left*, sigset_t **right*);
int `sigorset`(sigset_t **dest*, sigset_t **left*, sigset_t **right*);
```

### 注意

成功时返回 0，出错时返回-1

```
int `sigisemptyset`(const sigset_t **set*);
```

### 注意

如果*sig*为空，返回 1；否则返回 0

这些函数执行以下任务：

+   *sigandset()*将*left*和*right*集合的交集放入*dest*集合中；

+   *sigorset()*将*left*和*right*集合的并集放入*dest*集合中；并且

+   *sigisemptyset()*如果*set*不包含任何信号，则返回真。

#### 示例程序

使用本节中描述的函数，我们可以编写在示例 20-4 中展示的函数，这些函数在后续程序中被广泛使用。其中第一个，*printSigset()*，显示指定信号集中的信号成员。该函数使用`NSIG`常量，它在`<signal.h>`中定义，值比最高信号编号大 1。我们在一个循环中使用`NSIG`作为上限，测试所有信号编号是否属于某个集合。

### 注意

虽然`NSIG`在 SUSv3 中没有明确规定，但它在大多数 UNIX 实现中都有定义。然而，可能需要使用特定于实现的编译器选项来使其可见。例如，在 Linux 上，我们必须定义以下某个特性测试宏：`_BSD_SOURCE`、`_SVID_SOURCE`或`_GNU_SOURCE`。

*printSigMask()*和*printPendingSigs()*函数使用*printSigset()*分别显示进程信号屏蔽字和当前挂起的信号集合。*printSigMask()*和*printPendingSigs()*函数分别使用*sigprocmask()*和*sigpending()*系统调用。我们在信号屏蔽字（阻塞信号传递）")和挂起的信号中描述了*sigprocmask()*和*sigpending()*系统调用。

示例 20-4. 显示信号集的函数

```
`signals/signal_functions.c`
#define _GNU_SOURCE
#include <string.h>
#include <signal.h>
#include "signal_functions.h"           /* Declares functions defined here */
#include "tlpi_hdr.h"

/* NOTE: All of the following functions employ fprintf(), which
   is not async-signal-safe (see Section 21.1.2). As such, these
   functions are also not async-signal-safe (i.e., beware of
   indiscriminately calling them from signal handlers). */

void                    /* Print list of signals within a signal set */
printSigset(FILE *of, const char *prefix, const sigset_t *sigset)
{
    int sig, cnt;

    cnt = 0;
    for (sig = 1; sig < NSIG; sig++) {
        if (sigismember(sigset, sig)) {
            cnt++;
            fprintf(of, "%s%d (%s)\n", prefix, sig, strsignal(sig));
        }
    }

    if (cnt == 0)
        fprintf(of, "%s<empty signal set>\n", prefix);
}

int                     /* Print mask of blocked signals for this process */
printSigMask(FILE *of, const char *msg)
{
    sigset_t currMask;

    if (msg != NULL)
        fprintf(of, "%s", msg);

    if (sigprocmask(SIG_BLOCK, NULL, &currMask) == -1)
        return -1;

    printSigset(of, "\t\t", &currMask);

    return 0;
}

int                     /* Print signals currently pending for this process */
printPendingSigs(FILE *of, const char *msg)
{
    sigset_t pendingSigs;

    if (msg != NULL)
        fprintf(of, "%s", msg);

    if (sigpending(&pendingSigs) == -1)
        return -1;

    printSigset(of, "\t\t", &pendingSigs);

    return 0;
}      `signals/signal_functions.c`
```

## 信号屏蔽字（阻塞信号传递）

对于每个进程，内核维护一个*信号屏蔽字*—一个当前阻塞传递给该进程的信号集合。如果向进程发送一个被阻塞的信号，信号的传递会被延迟，直到它通过从进程信号屏蔽字中移除而被解锁。（在 UNIX 信号模型如何映射到线程中，我们将看到信号屏蔽字实际上是每个线程的属性，并且在多线程进程中，每个线程可以独立检查和修改其信号屏蔽字，方法是使用*pthread_sigmask()*函数。）

信号可以通过以下方式添加到信号屏蔽字中：

+   当信号处理程序被调用时，触发该调用的信号可以自动添加到信号屏蔽字中。是否发生这种情况取决于在使用*sigaction()*建立处理程序时所使用的标志。

+   当使用*sigaction()*建立信号处理程序时，可以指定在处理程序调用时需要阻塞的额外信号集。

+   可以随时使用*sigprocmask()*系统调用显式地将信号添加到信号屏蔽字中，或从信号屏蔽字中移除信号。

我们将延迟讨论前两个情况，直到我们在更改信号分配：*sigaction()*")中讨论*sigaction()*，现在讨论*sigprocmask()*。

```
#include <signal.h>

int `sigprocmask`(int *how*, const sigset_t **set*, sigset_t **oldset*);
```

### 注意

成功时返回 0，出错时返回-1

我们可以使用*sigprocmask()*来改变进程的信号屏蔽字，获取当前的屏蔽字，或两者兼顾。*how*参数决定了*sigprocmask()*对信号屏蔽字所做的更改：

`SIG_BLOCK`

指定在*set*指向的信号集中的信号将被添加到信号屏蔽字中。换句话说，信号屏蔽字将被设置为其当前值与*set*的并集。

`SIG_UNBLOCK`

指向*set*的信号集中的信号会从信号掩码中移除。解除对一个当前未被阻塞的信号的阻塞并不会导致错误返回。

`SIG_SETMASK`

指向*set*的信号集会被赋值到信号掩码中。

在每种情况下，如果*oldset*参数不为`NULL`，它指向一个*sigset_t*缓冲区，用于返回先前的信号掩码。

如果我们希望检索信号掩码而不做任何修改，那么可以为*set*参数指定`NULL`，在这种情况下，*how*参数会被忽略。

为了暂时阻止信号的传递，我们可以使用示例 20-5 中所示的一系列调用来阻塞信号，然后通过将信号掩码重置为之前的状态来解除阻塞。

示例 20-5. 暂时阻止信号传递

```
sigset_t blockSet, prevMask;

    /* Initialize a signal set to contain SIGINT */

    sigemptyset(&blockSet);
    sigaddset(&blockSet, SIGINT);

    /* Block SIGINT, save previous signal mask */

    if (sigprocmask(SIG_BLOCK, &blockSet, &prevMask) == -1)
        errExit("sigprocmask1");

    /* ... Code that should not be interrupted by SIGINT ... */

    /* Restore previous signal mask, unblocking SIGINT */

    if (sigprocmask(SIG_SETMASK, &prevMask, NULL) == -1)
        errExit("sigprocmask2");
```

SUSv3 规定，如果通过调用*sigprocmask()*解除阻塞任何待处理信号，那么至少有一个信号会在调用返回之前被传递。换句话说，如果我们解除阻塞一个待处理信号，它会立即传递给进程。

尝试阻塞`SIGKILL`和`SIGSTOP`会被默默忽略。如果我们尝试阻塞这些信号，*sigprocmask()*既不会遵从请求，也不会生成错误。这意味着我们可以使用以下代码阻塞所有信号，除了`SIGKILL`和`SIGSTOP`：

```
sigfillset(&blockSet);
if (sigprocmask(SIG_BLOCK, &blockSet, NULL) == -1)
    errExit("sigprocmask");
```

## 待处理信号

如果一个进程接收到一个它当前正在阻塞的信号，该信号会被添加到进程的待处理信号集中。当（如果）该信号后来被解除阻塞时，它会被传递给进程。要确定一个进程的待处理信号，我们可以调用*sigpending()*。

```
#include <signal.h>

int `sigpending`(sigset_t **set*);
```

### 注意

成功返回 0，错误返回-1

*sigpending()*系统调用会返回调用进程的待处理信号集，存储在*set*指向的*sigset_t*结构中。然后我们可以使用第 20.9 节中描述的*sigismember()*函数来检查*set*。

如果我们更改一个待处理信号的处置方式，那么当该信号后续被解除阻塞时，它会根据新的处置方式来处理。虽然这种方法不常用，但其中一个应用是通过将信号的处置方式设置为`SIG_IGN`来防止待处理信号的传递，或者如果该信号的默认行为是*忽略*，则设置为`SIG_DFL`。这样，信号就会从进程的待处理信号集中移除，从而不再传递。

## 信号不会排队

待处理信号集只是一个掩码；它仅表示某个信号是否已经发生，但不表示信号发生的次数。换句话说，如果在信号被阻塞期间同一个信号被多次生成，它会被记录在待处理信号集中，之后仅会传递一次。（标准信号和实时信号之间的一个区别是，实时信号是排队的，如第 22.8 节所讨论的。）

示例 20-6 和示例 20-7 展示了可以用来观察信号不会排队的两个程序。示例 20-6 中的程序最多接受四个命令行参数，具体如下：

```
$ `./sig_sender` ``*`PID num-sigs sig-num [sig-num-2]`*``
```

第一个参数是程序应该向其发送信号的进程 ID。第二个参数指定要发送给目标进程的信号数量。第三个参数指定要发送给目标进程的信号编号。如果第四个参数提供了一个信号编号，那么程序会在发送前面参数指定的信号后，发送该信号编号的一个实例。在下面的示例 Shell 会话中，我们使用这个最终参数向目标进程发送`SIGINT`信号；稍后发送此信号的目的将变得清晰。

示例 20-6. 发送多个信号

```
`signals/sig_sender.c`
#include <signal.h>
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    int numSigs, sig, j;
    pid_t pid;

    if (argc < 4 || strcmp(argv[1], "--help") == 0)
        usageErr("%s pid num-sigs sig-num [sig-num-2]\n", argv[0]);
    pid = getLong(argv[1], 0, "PID");
    numSigs = getInt(argv[2], GN_GT_0, "num-sigs");
    sig = getInt(argv[3], 0, "sig-num");

    /* Send signals to receiver */

    printf("%s: sending signal %d to process %ld %d times\n",
            argv[0], sig, (long) pid, numSigs);

    for (j = 0; j < numSigs; j++)
        if (kill(pid, sig) == -1)
            errExit("kill");

    /* If a fourth command-line argument was specified, send that signal */

    if (argc > 4)
        if (kill(pid, getInt(argv[4], 0, "sig-num-2")) == -1)
            errExit("kill");

    printf("%s: exiting\n", argv[0]);
    exit(EXIT_SUCCESS);
}
     `signals/sig_sender.c`
```

在示例 20-7 中展示的程序旨在捕获并报告由示例 20-6 中的程序发送的信号统计信息。此程序执行以下步骤：

+   程序设置了一个处理程序来捕获所有信号 ![](img/U002.png)。 （无法捕获`SIGKILL`和`SIGSTOP`信号，但我们忽略尝试为这些信号建立处理程序时发生的错误。）对于大多数类型的信号，处理程序 ![](img/U001.png) 仅使用数组对信号进行计数。如果接收到`SIGINT`，处理程序会设置一个标志（*gotSigint*），使得程序退出其主循环（下面描述的`while`循环）。 （我们将在全局变量和*sig_atomic_t*数据类型中解释`volatile`限定符和用于声明*gotSigint*变量的*sig_atomic_t*数据类型的使用。）

+   如果程序接收到命令行参数，那么程序会在该参数指定的秒数内屏蔽所有信号，随后在解除屏蔽信号之前，显示挂起信号的集合 ![](img/U003.png)。这允许我们在程序开始执行下一步之前向其发送信号。

+   程序执行一个`while`循环，消耗 CPU 时间，直到*gotSigint* 被设置为 ![](img/U004.png)。(等待信号：*pause()* 和 使用屏蔽等待信号：*sigsuspend()* 描述了使用 *pause()* 和 *sigsuspend()*，它们是等待信号到达时更节省 CPU 的方法。)

+   退出`while`循环后，程序会显示所有接收到的信号的计数 ![](img/U005.png)。

我们首先使用这两个程序来说明，阻塞的信号只有在生成多次的情况下也只会被传递一次。我们通过为接收方指定一个睡眠间隔，并在睡眠间隔完成之前发送所有信号来实现这一点。

```
$ `./sig_receiver 15 &`                     *Receiver blocks signals for 15 secs*
[1] 5368
./sig_receiver: PID is 5368
./sig_receiver: sleeping for 15 seconds
$ `./sig_sender 5368 1000000 10 2`          *Send* SIGUSR1  *signals*, *plus a* SIGINT
./sig_sender: sending signal 10 to process 5368 1000000 times
./sig_sender: exiting
./sig_receiver: pending signals are:
                2 (Interrupt)
                10 (User defined signal 1)
./sig_receiver: signal 10 caught 1 time
[1]+  Done                    ./sig_receiver 15
```

发送程序的命令行参数指定了`SIGUSR1`和`SIGINT`信号，它们分别是 Linux/x86 上的信号 10 和 2。

从上面的输出可以看出，尽管发送了百万个信号，但只有一个信号被传递给接收方。

即使进程没有阻塞信号，它也可能接收到的信号数量少于发送给它的信号数量。如果信号发送得太快，以至于它们在接收进程被内核调度执行之前就到达，那么多个信号将只在进程的待处理信号集里记录一次。如果我们在没有命令行参数的情况下执行示例 20-7（这样它不会阻塞信号和睡眠），我们会看到以下结果：

```
$ `./sig_receiver &`
[1] 5393
./sig_receiver: PID is 5393
$ `./sig_sender 5393 1000000 10 2`
./sig_sender: sending signal 10 to process 5393 1000000 times
./sig_sender: exiting
./sig_receiver: signal 10 caught 52 times
[1]+  Done                    ./sig_receiver
```

在发送的百万个信号中，只有 52 个被接收进程捕获。（被捕获的信号的准确数量会根据内核调度算法的决定变化。）出现这种情况的原因是，每次发送程序被调度执行时，它会向接收方发送多个信号。然而，只有一个信号会被标记为待处理，并且在接收方有机会运行时被传递。

示例 20-7. 捕获和计数信号

```
`signals/sig_receiver.c`
    #define _GNU_SOURCE
    #include <signal.h>
    #include "signal_functions.h"           /* Declaration of printSigset() */
    #include "tlpi_hdr.h"

    static int sigCnt[NSIG];                /* Counts deliveries of each signal */
    static volatile sig_atomic_t gotSigint = 0;
                                            /* Set nonzero if SIGINT is delivered */

    static void
 handler(int sig)
    {
            if (sig == SIGINT)
            gotSigint = 1;
        else
            sigCnt[sig]++;
    }

    int
    main(int argc, char *argv[])
    {
        int n, numSecs;
        sigset_t pendingMask, blockingMask, emptyMask;

        printf("%s: PID is %ld\n", argv[0], (long) getpid());

     for (n = 1; n < NSIG; n++)          /* Same handler for all signals */
            (void) signal(n, handler);      /* Ignore errors */

        /* If a sleep time was specified, temporarily block all signals,
           sleep (while another process sends us signals), and then
           display the mask of pending signals and unblock all signals */

     if (argc > 1) {
            numSecs = getInt(argv[1], GN_GT_0, NULL);

            sigfillset(&blockingMask);
            if (sigprocmask(SIG_SETMASK, &blockingMask, NULL) == -1)
                errExit("sigprocmask");

            printf("%s: sleeping for %d seconds\n", argv[0], numSecs);
            sleep(numSecs);

            if (sigpending(&pendingMask) == -1)
                errExit("sigpending");

            printf("%s: pending signals are: \n", argv[0]);
            printSigset(stdout, "\t\t", &pendingMask);

            sigemptyset(&emptyMask);        /* Unblock all signals */
            if (sigprocmask(SIG_SETMASK, &emptyMask, NULL) == -1)
                errExit("sigprocmask");
        }

     while (!gotSigint)                  /* Loop until SIGINT caught */
            continue;

     for (n = 1; n < NSIG; n++)          /* Display number of signals received */
            if (sigCnt[n] != 0)
                printf("%s: signal %d caught %d time%s\n", argv[0], n,
                        sigCnt[n], (sigCnt[n] == 1) ? "" : "s");

        exit(EXIT_SUCCESS);
    }
         `signals/sig_receiver.c`
```

## 改变信号处置：*sigaction()*

*sigaction()* 系统调用是设置信号处置的 *signal()* 的替代方案。尽管 *sigaction()* 的使用比 *signal()* 略复杂，但它提供了更大的灵活性。特别是，*sigaction()* 允许我们在不改变信号处置的情况下检索它，并且可以设置控制信号处理程序调用时具体发生什么的各种属性。此外，正如我们在信号的实现与可移植性中将详细阐述的那样，*sigaction()* 在建立信号处理程序时比 *signal()* 更具可移植性。

```
#include <signal.h>

int `sigaction`(int *sig*, const struct sigaction **act*, struct sigaction **oldact*);
```

### 注意

成功时返回 0，出错时返回 -1

*sig* 参数标识了我们想要检索或更改其处理方式的信号。此参数可以是任何信号，除了 `SIGKILL` 或 `SIGSTOP`。

*act* 参数是指向一个结构体的指针，该结构体指定了信号的新处理方式。如果我们只关心获取信号的当前处理方式，那么可以为此参数指定 `NULL`。*oldact* 参数是指向同一类型结构体的指针，用于返回信号先前处理方式的信息。如果我们不关心这些信息，那么可以为此参数指定 `NULL`。*act* 和 *oldact* 所指向的结构体类型如下：

```
struct sigaction {
    void   (*sa_handler)(int);    /* Address of handler */
    sigset_t sa_mask;             /* Signals blocked during handler
                                     invocation */
    int      sa_flags;            /* Flags controlling handler invocation */
    void   (*sa_restorer)(void);  /* Not for application use */
};
```

### 注意

*sigaction* 结构体实际上比这里所示的更为复杂。我们将在第 21.4 节中进一步讨论细节。

*sa_handler* 字段对应于传递给 *signal()* 的 *handler* 参数。它指定了信号处理程序的地址，或者是常量 `SIG_IGN` 或 `SIG_DFL`。*sa_mask* 和 *sa_flags* 字段，稍后我们将讨论，只有在 *sa_handler* 是信号处理程序地址时才会被解释——也就是说，当其值不是 `SIG_IGN` 或 `SIG_DFL` 时。剩余字段 *sa_restorer* 并不打算在应用程序中使用（并且在 SUSv3 中未指定）。

### 注意

*sa_restorer* 字段在内部使用，确保在信号处理程序完成后，调用特殊目的的 *sigreturn()* 系统调用，以恢复进程的执行上下文，使其能够继续执行在被信号处理程序中断的位置。可以在 *glibc* 源文件 `sysdeps/unix/sysv/linux/i386/sigaction.c` 中找到这种用法的示例。

*sa_mask* 字段定义了一组信号，在调用由 *sa_handler* 定义的处理程序时，这些信号会被阻塞。当信号处理程序被调用时，任何当前不在进程信号屏蔽中的信号都会自动添加到该屏蔽中。这些信号会在信号处理程序返回时自动从进程信号屏蔽中移除。*sa_mask* 字段允许我们指定一组信号，这些信号在执行该处理程序时不允许中断执行。此外，导致调用信号处理程序的信号也会自动添加到进程信号屏蔽中。这意味着，如果在信号处理程序执行期间同一信号的第二次实例到达，信号处理程序不会递归中断自己。由于被阻塞的信号不会被排队，如果在信号处理程序执行期间这些信号反复生成，它们将在稍后（只）被送达一次。

*sa_flags* 字段是一个位掩码，用于指定控制信号处理方式的各种选项。以下位可以在此字段中进行按位或 (`|`) 操作：

`SA_NOCLDSTOP`

如果*sig*是`SIGCHLD`，当子进程因接收到信号而被停止或恢复时，不生成此信号。参见为停止的子进程投递`SIGCHLD`。

`SA_NOCLDWAIT`

（自 Linux 2.6 起）如果*sig*是`SIGCHLD`，在子进程终止时，不将其转变为僵尸进程。更多细节，请参见忽略死掉的子进程。

`SA_NODEFER`

当捕获到此信号时，在处理程序执行时，不要自动将其添加到进程的信号掩码中。`SA_NOMASK`这个名称是`SA_NODEFER`的历史同义词，但后者更为推荐，因为它已在 SUSv3 中标准化。

`SA_ONSTACK`

使用*sigaltstack()*安装的备用栈调用此信号的处理程序。参见第 21.3 节。

`SA_RESETHAND`

当捕获到此信号时，在调用处理程序之前，将其处置方式重置为默认值（即，`SIG_DFL`）。(默认情况下，信号处理程序保持有效，直到通过再次调用*sigaction()*显式取消)。`SA_ONESHOT`这个名称是`SA_RESETHAND`的历史同义词，但后者更为推荐，因为它已在 SUSv3 中标准化。

`SA_RESTART`

自动重启被此信号处理程序中断的系统调用。请参见第 21.5 节。

`SA_SIGINFO`

使用附加参数调用信号处理程序，提供关于信号的更多信息。我们将在第 21.4 节中描述此标志。

以上所有选项都在 SUSv3 中进行了规定。

使用*sigaction()*的一个示例见于示例 21-1 和信号处理程序同时调用非重入函数")。

## 等待信号：*pause()*

调用*pause()*会暂停进程的执行，直到该调用被信号处理程序中断（或者直到未处理的信号终止进程）。

```
#include <unistd.h>

int `pause`(void);
```

### 注意

始终返回-1，且将*errno*设置为`EINTR`

当信号被处理时，*pause()*会被中断，并始终返回-1，且将*errno*设置为`EINTR`。(我们在第 21.5 节中更详细地讨论了`EINTR`错误。)

使用*pause()*的一个示例见于示例 20-2。

在使用掩码等待信号：*sigsuspend()*")、同步等待信号和通过文件描述符获取信号中，我们探讨了程序在等待信号时暂停执行的其他方式。

## 总结

信号是一种通知，表示某种事件已发生，并且可以由内核、另一个进程或进程本身发送给一个进程。信号有多种标准类型，每种类型都有一个唯一的编号和用途。

信号传递通常是异步的，这意味着信号中断进程执行的时刻是不可预测的。在某些情况下（例如硬件生成的信号），信号是同步传递的，这意味着信号在程序执行的某个特定时刻可预测且可重复地传递。

默认情况下，一个信号要么被忽略，要么终止一个进程（有无核心转储），要么停止一个运行中的进程，或者重新启动一个已停止的进程。特定的默认行为取决于信号类型。或者，程序可以使用*signal()*或*sigaction()*显式忽略一个信号，或者建立一个程序员定义的信号处理函数，该函数在信号到达时被调用。出于可移植性的考虑，最好使用*sigaction()*来建立信号处理程序。

一个进程（具有适当权限）可以使用*kill()*向另一个进程发送信号。发送空信号（0）是一种确定特定进程 ID 是否正在使用的方法。

每个进程都有一个信号屏蔽字，它是当前被阻塞的信号集合。可以使用*sigprocmask()*向信号屏蔽字中添加或移除信号。

如果信号在被阻塞时接收，它将保持挂起状态，直到解锁。标准信号不能排队；也就是说，信号只能标记为挂起（并因此稍后传递）一次。一个进程可以使用*sigpending()*系统调用检索一个信号集（表示多个不同信号的数据结构），以标识它挂起的信号。

*sigaction()*系统调用在设置信号的处理方式时提供比*signal()*更多的控制和灵活性。首先，我们可以指定一组额外的信号，当处理程序被调用时将被阻塞。此外，还可以使用各种标志来控制信号处理程序调用时发生的操作。例如，有些标志可以选择较旧的不可靠信号语义（即不阻塞导致处理程序调用的信号，并且在调用处理程序之前将信号的处理方式重置为默认值）。

使用*pause()*，一个进程可以暂停执行，直到接收到信号。

#### 更多信息

[Bovet & Cesati, 2005] 和 [Maxwell, 1999] 提供了 Linux 中信号实现的背景资料。[Goodheart & Cox, 1994] 详细介绍了 System V Release 4 上信号的实现。GNU C 库手册（在线提供，网址为 [`www.gnu.org/`](http://www.gnu.org/)）包含了关于信号的广泛描述。

## 练习

1.  正如在更改信号处理：*signal()*")中所述，*sigaction()*在设置信号处理程序时比*signal()*更具可移植性。在示例 20-7(`sig_receiver.c`)的程序中，将*signal()*替换为*sigaction()*。

1.  编写一个程序，演示当待处理信号的处理方式被更改为`SIG_IGN`时，程序永远不会看到（捕获）该信号。

1.  编写程序，验证在使用*sigaction()*设置信号处理程序时，`SA_RESETHAND`和`SA_NODEFER`标志的作用。

1.  使用*sigaction()*实现系统调用的中断与重启中描述的*siginterrupt()*函数。
