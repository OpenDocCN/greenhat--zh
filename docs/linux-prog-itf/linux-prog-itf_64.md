## 第六十四章. 伪终端

*伪终端*是一个提供 IPC 通道的虚拟设备。在通道的一端是一个期望连接到终端设备的程序。另一端是一个通过使用通道发送输入并读取输出来驱动终端导向程序的程序。

本章描述了伪终端的使用，展示了它们如何在诸如终端仿真器、*script(1)* 程序以及提供网络登录服务的程序如*ssh*等应用中发挥作用。

## 概述

图 64-1 展示了伪终端帮助我们解决的一个问题：如何使一个主机上的用户能够在通过网络连接的另一台主机上操作终端导向程序（例如，*vi*）？

如图所示，通过允许网络上的通信，套接字提供了解决这个问题所需的部分机制。然而，我们不能将终端导向程序的标准输入、输出和错误直接连接到套接字。这是因为终端导向程序期望连接到一个终端——以便执行在第三十四章和第六十二章中描述的终端操作。这些操作包括将终端置于非规范模式、开关回显功能以及设置终端前台进程组。如果程序尝试在套接字上执行这些操作，相关的系统调用将会失败。

此外，终端导向程序期望终端驱动程序执行某些输入输出处理。例如，在规范模式下，当终端驱动程序看到行首的文件结束字符（通常是*Control-D*）时，它会使得下一次*read()*调用返回无数据。

最后，终端导向程序必须有一个控制终端。这使得程序能够通过打开`/dev/tty`获得控制终端的文件描述符，同时也使得生成与作业控制和终端相关的信号（例如，`SIGTSTP`、`SIGTTIN`和`SIGINT`）成为可能。

从这个描述中，应该可以清楚地看出终端导向程序的定义非常广泛。它涵盖了我们通常会在交互式终端会话中运行的各种程序。

![问题：如何在网络上操作终端导向程序？](img/64-1_PTY-problem.png.jpg)图 64-1. 问题：如何在网络上操作终端导向程序？

#### 伪终端主从设备

伪终端提供了创建到终端导向程序的网络连接的缺失环节。伪终端是一对连接的虚拟设备：*伪终端主设备*和*伪终端从设备*，有时统称为*伪终端对*。伪终端对提供了一种 IPC 通道，类似于双向管道——两个进程可以打开主设备和从设备，然后通过伪终端在任一方向上传输数据。

关于伪终端的关键点是，从设备看起来就像一个标准终端。所有可以应用于终端设备的操作也可以应用于伪终端从设备。有些操作对于伪终端来说没有意义（例如，设置终端线速或校验位），但这没关系，因为伪终端从设备会默默忽略它们。

#### 程序如何使用伪终端

图 64-2 展示了两个程序如何典型地使用伪终端。（该图中的*pty*是*伪终端*的常用缩写，我们在本章的各种图示和函数名称中也使用此缩写。）终端导向程序的标准输入、输出和错误连接到伪终端从设备，该设备也成为该程序的控制终端。在伪终端的另一侧，驱动程序充当用户的代理，向终端导向程序提供输入并读取该程序的输出。

![通过伪终端进行通信的两个程序](img/64-2_PTY-pty-scale90.png.jpg)图 64-2。两个程序通过伪终端进行通信

通常，驱动程序同时从另一个 I/O 通道读取和写入数据。它充当中继，双向传输数据，介于伪终端和另一个程序之间。为了实现这一点，驱动程序必须同时监控来自任一方向的输入。通常，使用 I/O 多路复用（*select()*或*poll()*），或者使用一对进程或线程来执行每个方向的数据传输。

使用伪终端的应用程序通常按以下方式进行：

1.  驱动程序打开伪终端主设备。

1.  驱动程序调用*fork()*来创建一个子进程。子进程执行以下步骤：

    1.  调用*setsid()*以启动一个新会话，子进程成为该会话的会话领导（参见第 34.3 节）。此步骤还使子进程失去其控制终端。

    1.  打开与主设备对应的伪终端从设备。由于子进程是会话领导，并且没有控制终端，伪终端从设备成为子进程的控制终端。

    1.  使用*dup()*（或类似方法）复制从设备的文件描述符到标准输入、输出和错误。

    1.  调用*exec()*来启动要连接到伪终端从设备的面向终端的程序。

这时，两个程序现在可以通过伪终端进行通信。驱动程序写入主设备的任何内容都会作为输入出现在从设备上的面向终端程序中，面向终端程序写入从设备的任何内容都会被主设备上的驱动程序读取。我们将在 64.5 节中进一步讨论伪终端 I/O 的细节。

### 注意

伪终端也可以用来连接任意一对进程（即，不一定是父进程和子进程）。所需要的只是打开伪终端主设备的进程通知另一个进程对应从设备的名称，可能是通过写入文件或使用其他 IPC 机制传输该名称。（当我们像上述那样使用*fork()*时，子进程会自动继承父进程的足够信息，以便确定从设备的名称。）

到目前为止，我们对伪终端使用的讨论还比较抽象。图 64-3 展示了一个具体的例子：*ssh*的伪终端使用，*ssh*是一种允许用户在通过网络连接的远程系统上安全运行登录会话的应用程序。（实际上，这个图表结合了图 64-1 和图 64-2 中的信息。）在远程主机上，伪终端主设备的驱动程序是*ssh*服务器（*sshd*），连接到伪终端从设备的面向终端的程序是登录 shell。*ssh*服务器是通过套接字将伪终端连接到*ssh*客户端的“粘合剂”。一旦所有登录的细节完成，*ssh*服务器和客户端的主要功能就是在本地主机的用户终端与远程主机的 shell 之间双向传递字符。

### 注意

我们省略了对*ssh*客户端和服务器的许多细节描述。例如，这些程序会加密在网络中任何方向上传输的数据。我们展示了一个位于远程主机的单个*ssh*服务器进程，但实际上，*ssh*服务器是一个并发的网络服务器。它变成一个守护进程，并创建一个被动的 TCP 套接字来监听来自*ssh*客户端的传入连接。对于每个连接，主*ssh*服务器会分叉出一个子进程，处理单个客户端登录会话的所有细节。（我们在图 64-3 中将这个子进程称为*ssh*服务器。）除了上述伪终端设置的细节外，*ssh*服务器子进程还会进行用户认证，更新远程主机上的登录帐户文件（如第四十章中所描述），然后执行登录 shell。

![How ssh uses a pseudoterminal](img/64-3_PTY-ssh-login.png.jpg)图 64-3. *ssh*如何使用伪终端

在某些情况下，多个进程可能连接到伪终端的从端。我们的*ssh*示例说明了这一点。对于从端的会话领导者是一个 shell，它创建进程组来执行远程用户输入的命令。所有这些进程都将伪终端从端作为它们的控制终端。与传统终端一样，这些进程组中的一个可以是伪终端从端的前台进程组，只有这个进程组被允许从从端读取（如果已设置`TOSTOP`位），并且（如果设置了`TOSTOP`位）写入到从端。

#### 伪终端的应用

伪终端也用于许多除了网络服务以外的应用程序。以下是一些例子：

+   *expect(1)*程序使用伪终端允许一个交互式的终端导向程序通过脚本文件驱动。

+   终端仿真器，如*xterm*，使用伪终端来提供与终端窗口相关的终端功能。

+   *screen(1)*程序使用伪终端来在多个进程（例如多个 shell 会话）之间复用一个物理终端（或终端窗口）。

+   伪终端用于*script(1)*程序，该程序记录在 shell 会话期间发生的所有输入和输出。

+   有时，伪终端可以绕过*stdio*函数在将输出写入磁盘文件或管道时执行的默认块缓冲区，而使用终端输出所用的行缓冲区。（我们将在习题 64-7 中进一步讨论这个问题。）

#### System V (UNIX 98) 和 BSD 伪终端

BSD 和 System V 提供了不同的接口来查找和打开伪终端对的两端。BSD 伪终端实现历史上更为人所知，因为它与许多基于套接字的网络应用一起使用。出于兼容性原因，许多 UNIX 实现最终支持了这两种风格的伪终端。

System V 接口的使用方式比 BSD 接口更简单，且 SUSv3 伪终端规范基于 System V 接口。（伪终端规范首次出现在 SUSv1 中。）出于历史原因，在 Linux 系统中，这种类型的伪终端通常被称为 *UNIX 98* 伪终端，尽管 UNIX 98 标准（即 SUSv2）要求伪终端必须基于 STREAMS，而 Linux 对伪终端的实现并不是基于 STREAMS 的。（SUSv3 不要求基于 STREAMS 的实现。）

早期版本的 Linux 仅支持 BSD 风格的伪终端，但从内核 2.2 开始，Linux 同时支持这两种类型的伪终端。本章我们主要关注 UNIX 98 伪终端。有关 BSD 伪终端的差异，请参见第 64.8 节。

## UNIX 98 伪终端

一步一步地，我们将开发一个函数 *ptyFork()*，它完成创建如图 64-2 所示的设置的大部分工作。然后，我们将使用这个函数来实现 *script(1)* 程序。在此之前，我们先来看一下与 UNIX 98 伪终端相关的各种库函数：

+   *posix_openpt()* 函数打开一个未使用的伪终端主设备，返回一个文件描述符，用于后续调用中引用该设备。

+   *grantpt()* 函数更改与伪终端主设备对应的从设备的所有权和权限。

+   *unlockpt()* 函数解锁与伪终端主设备对应的从设备，使得从设备可以被打开。

+   *ptsname()* 函数返回与伪终端主设备对应的从设备的名称。然后可以使用 *open()* 打开该从设备。

### 打开未使用的主设备：*posix_openpt()*

*posix_openpt()* 函数查找并打开一个未使用的伪终端主设备，返回一个文件描述符，可以在后续中用来引用该设备。

```
#define _XOPEN_SOURCE 600
#include <stdlib.h>
#include <fcntl.h>

int `posix_openpt`(int *flags*);
```

### 注意

成功时返回文件描述符，出错时返回 -1

*flags* 参数是通过按位或操作将以下常量中的零个或多个组合在一起构造的：

`O_RDWR`

打开设备用于读写。通常情况下，我们总是会在 *flags* 中包含这个常量。

`O_NOCTTY`

不要让这个终端成为进程的控制终端。在 Linux 中，伪终端主设备无法成为进程的控制终端，无论在调用*posix_openpt()*时是否指定了`O_NOCTTY`标志。（这是有道理的，因为伪终端主设备并不是真正的终端；它是与从设备连接的终端的另一端。）然而，在一些实现中，如果我们希望防止进程因打开伪终端主设备而获得控制终端，则需要使用`O_NOCTTY`。

和*open()*一样，*posix_openpt()*使用最低可用的文件描述符来打开伪终端主设备。

调用*posix_openpt()*还会在`/dev/pts`目录中创建一个对应的伪终端从设备文件。当我们在下面描述*ptsname()*函数时，会进一步介绍该文件。

*posix_openpt()*函数是 SUSv3 中新增加的，由 POSIX 委员会发明。在原始的 System V 伪终端实现中，通过打开*伪终端主设备克隆设备*`/dev/ptmx`来获取一个可用的伪终端主设备。打开这个虚拟设备会自动定位并打开下一个未使用的伪终端主设备，并返回其文件描述符。Linux 提供了这个设备，其中*posix_openpt()*实现如下：

```
int
posix_openpt(int flags)
{
    return open("/dev/ptmx", flags);
}
```

#### UNIX 98 伪终端数量限制

因为每对使用中的伪终端都会消耗少量不可交换的内核内存，内核会对系统中的 UNIX 98 伪终端对数量进行限制。在 2.6.3 版本之前的内核中，该限制由内核配置选项（`CONFIG_UNIX98_PTYS`）控制。该选项的默认值为 256，但我们可以将限制更改为 0 到 2048 之间的任何值。

从 Linux 2.6.4 版本开始，`CONFIG_UNIX98_PTYS`内核配置选项被弃用，改为采用更灵活的方法。相反，伪终端的数量限制由 Linux 特定的`/proc/sys/kernel/pty/max`文件中的值定义。该文件的默认值为 4096，且可以设置为最大 1,048,576。一个相关的只读文件`/proc/sys/kernel/pty/nr`显示当前正在使用的 UNIX 98 伪终端数量。

### 更改从设备所有权和权限：*grantpt()*

SUSv3 规范要求使用*grantpt()*来更改与文件描述符*mfd*引用的伪终端主设备对应的从设备的所有权和权限。在 Linux 上，实际上调用*grantpt()*并不是必要的。然而，在某些实现中，必须使用*grantpt()*，因此便携式应用程序在调用*posix_openpt()*之后应当调用它。

```
#define _XOPEN_SOURCE 500
#include <stdlib.h>

int `grantpt`(int *mfd*);
```

### 注意

成功时返回 0，出错时返回-1

在需要*grantpt()*的系统中，这个函数会创建一个子进程，该子进程执行一个设置用户 ID 为*root*的程序。这个程序通常叫做*pt_chown*，它对伪终端从设备执行以下操作：

+   更改从设备的所有权，使其与调用进程的有效用户 ID 相同；

+   将从设备的组更改为*tty*；并且

+   更改从设备的权限，使得所有者具有读写权限，组具有写权限。

更改终端的组为*tty*并启用组写权限的原因是，*wall(1)*和*write(1)*程序是由*tty*组拥有的设置组 ID 程序。

在 Linux 上，伪终端从设备会自动按上述方式配置，这就是为什么不需要调用*grantpt()*（但仍然应该调用）的原因。

### 注意

由于可能会创建子进程，SUSv3 规定，如果调用程序已为`SIGCHLD`安装了处理程序，则*grantpt()*的行为未指定。

### 解锁从设备：*unlockpt()*

*unlockpt()*函数会移除与由文件描述符*mfd*引用的伪终端主设备对应的从设备上的内部锁。这个锁机制的目的是允许调用进程在其他进程能够打开它之前，执行伪终端从设备所需的初始化（例如，调用*grantpt()*）。

```
#define _XOPEN_SOURCE
#include <stdlib.h>

int `unlockpt`(int *mfd*);
```

### 注意

成功时返回 0，错误时返回-1

在伪终端从设备被*unlockpt()*解锁之前尝试打开它会失败，并返回错误`EIO`。

### 获取从设备的名称：*ptsname()*

*ptsname()*函数返回与文件描述符*mfd*引用的伪终端主设备对应的伪终端从设备名称。

```
#define _XOPEN_SOURCE
#include <stdlib.h>

char *`ptsname`(int *mfd*);
```

### 注意

成功时返回指向（可能是静态分配的）字符串的指针，出错时返回 NULL。

在 Linux 系统（与大多数实现相同）上，*ptsname()*返回一个形如`/dev/pts/`*nn*的名称，其中*nn*被一个唯一标识此伪终端从设备的数字替换。

用于返回从设备名称的缓冲区通常是静态分配的。因此，它会被后续对*ptsname()*的调用覆盖。

### 注意

GNU C 库提供了一个可重入的*ptsname()*模拟函数，形式为*ptsname_r(mfd, strbuf, buflen)*。然而，这个函数是非标准的，并且仅在少数其他 UNIX 实现中可用。必须定义`_GNU_SOURCE`功能测试宏，才能从`<stdlib.h>`中获取*ptsname_r()*的声明。

一旦我们使用*unlockpt()*解锁了从设备，就可以通过传统的*open()*系统调用来打开它。

### 注意

在使用 STREAMS 的 System V 衍生版本中，可能需要执行一些进一步的步骤（在打开从设备后，将 STREAMS 模块推送到从设备上）。可以在[Stevens & Rago, 2005]中找到如何执行这些步骤的示例。

## 打开主设备：*ptyMasterOpen()*

我们现在介绍一个函数，*ptyMasterOpen()*，它利用前面部分中描述的函数来打开一个伪终端主设备并获取相应伪终端从设备的名称。我们提供这个函数的原因有两个：

+   大多数程序以完全相同的方式执行这些步骤，因此将它们封装在一个函数中是很方便的。

+   我们的 *ptyMasterOpen()* 函数隐藏了所有与 UNIX 98 伪终端特定的细节。在第 64.8 节中，我们介绍了一个重新实现的函数，使用 BSD 风格的伪终端。我们在本章余下部分展示的所有代码都可以与这两种实现配合使用。

```
#include "pty_master_open.h"

int `ptyMasterOpen`(char **slaveName*, size_t *snLen*);
```

### 注意

成功时返回文件描述符，失败时返回 -1。

*ptyMasterOpen()* 函数打开一个未使用的伪终端主设备，在其上调用 *grantpt()* 和 *unlockpt()*，并将相应伪终端从设备的名称复制到 *slaveName* 指向的缓冲区中。调用者必须在参数 *snLen* 中指定此缓冲区中可用的空间大小。我们在示例 64-1* 的实现")中展示了此函数的实现。

### 注意

同样也可以省略 *slaveName* 和 *snLen* 参数，让 *ptyMasterOpen()* 的调用者直接调用 *ptsname()* 来获取伪终端从设备的名称。然而，我们使用 *slaveName* 和 *snLen* 参数，是因为 BSD 伪终端没有提供 *ptsname()* 函数的等效函数，我们为 BSD 风格伪终端实现的等效函数（见示例 64-4*")）封装了获取从设备名称的 BSD 技术。

示例 64-1. *ptyMasterOpen()* 实现

```
`pty/pty_master_open.c`
#define _XOPEN_SOURCE 600
#include <stdlib.h>
#include <fcntl.h>
#include "pty_master_open.h"            /* Declares ptyMasterOpen() */
#include "tlpi_hdr.h"

int
ptyMasterOpen(char *slaveName, size_t snLen)
{
    int masterFd, savedErrno;
    char *p;

    masterFd = posix_openpt(O_RDWR | O_NOCTTY); /* Open pty master */
    if (masterFd == -1)
        return -1;

    if (grantpt(masterFd) == -1) {              /* Grant access to slave pty */
        savedErrno = errno;
        close(masterFd);                        /* Might change 'errno' */
        errno = savedErrno;
        return -1;
    }

    if (unlockpt(masterFd) == -1) {             /* Unlock slave pty */
        savedErrno = errno;
        close(masterFd);                        /* Might change 'errno' */
        errno = savedErrno;
        return -1;
    }

    p = ptsname(masterFd);                      /* Get slave pty name */
    if (p == NULL) {
        savedErrno = errno;
        close(masterFd);                        /* Might change 'errno' */
        errno = savedErrno;
        return -1;
    }

    if (strlen(p) < snLen) {
        strncpy(slaveName, p, snLen);
    } else {                    /* Return an error if buffer too small */
        close(masterFd);
        errno = EOVERFLOW;
        return -1;
    }

    return masterFd;
}
      `pty/pty_master_open.c`
```

## 使用伪终端连接进程：*ptyFork()*

我们现在准备实现一个函数，使用伪终端对建立两个进程之间的连接执行所有工作，如图 64-2 所示。*ptyFork()* 函数创建一个子进程，该进程通过伪终端对与父进程连接。

```
#include "pty_fork.h"

pid_t `ptyFork`(int **masterFd*, char **slaveName*, size_t *snLen*,
            const struct termios **slaveTermios*, const struct winsize **slaveWS*);
```

### 注意

在父进程中：成功时返回子进程的进程 ID，失败时返回 -1；在成功创建的子进程中：始终返回 0。

*ptyFork()* 的实现见示例 64-2* 的实现")。此函数执行以下步骤：

+   使用 *ptyMasterOpen()* 打开伪终端主设备（见示例 64-1* 的实现")） ![](img/U001.png)。

+   如果*slaveName*参数非`NULL`，则将伪终端从设备的名称复制到此缓冲区！[](figs/web/U002.png)。 （如果*slaveName*不为`NULL`，则它必须指向至少*snLen*字节的缓冲区。）调用者可以使用此名称更新登录帐户文件（第四十章），如果适用的话。更新登录帐户文件对于提供登录服务的应用程序是合适的——例如，*ssh*、*rlogin*和*telnet*。另一方面，像*script(1)*（第 64.6 节）这样的程序不更新登录帐户文件，因为它们不提供登录服务。

+   调用*fork()*创建子进程！[](figs/web/U003.png)。

+   在*fork()*之后，父进程所做的唯一事情就是确保将伪终端主设备的文件描述符返回给调用者，该文件描述符存储在*masterFd*指向的整数中！[](figs/web/U004.png)。

+   在*fork()*之后，子进程执行以下步骤：

    +   调用*setsid()*，创建一个新会话（第 34.3 节）！[](figs/web/U005.png)。子进程成为新会话的会话领导者，并失去其控制终端（如果它曾有的话）。

    +   关闭伪终端主设备的文件描述符，因为子进程不再需要它！[](figs/web/U006.png)。

    +   打开伪终端从设备！[](figs/web/U007.png)。由于子进程在前一步中失去了控制终端，这一步骤使得伪终端从设备成为子进程的控制终端。

    +   如果定义了`TIOCSCTTY`宏，则在伪终端从设备的文件描述符上执行`TIOCSCTTY` *ioctl()*操作！[](figs/web/U008.png)。这段代码使我们的*ptyFork()*函数能够在 BSD 平台上运行，在这些平台上，控制终端只能通过显式的`TIOCSCTTY`操作获得（参见第 34.4 节）。

    +   如果*slaveTermios*参数非`NULL`，则调用*tcsetattr()*，将伪终端从设备的终端属性设置为*termios*结构中由此参数指向的值！[](figs/web/U009.png)。该参数的使用便于某些交互式程序（例如，*script(1)*)，这些程序使用伪终端，并需要将从设备的属性设置为与程序运行所在终端的属性相同。

    +   如果*slaveWS*参数非`NULL`，则执行*ioctl()* `TIOCSWINSZ`操作，将伪终端从设备的窗口大小设置为*winsize*结构中由此参数指向的值！[](figs/web/U010.png)。此步骤与前一步执行的原因相同。

    +   使用*dup2()*将从端文件描述符复制为子进程的标准输入、输出和错误输出！[](figs/web/U011.png)。此时，子进程可以执行任何程序，而该程序可以使用标准文件描述符与伪终端进行通信。执行的程序可以执行所有通常由在常规终端上运行的程序执行的终端操作。

和*fork()*一样，*ptyFork()*在父进程中返回子进程的进程 ID，在子进程中返回 0，或者在出错时返回-1。

最终，通过*ptyFork()*创建的子进程将会终止。如果父进程没有同时终止，那么它必须等待子进程以消除产生的僵尸进程。然而，这一步通常可以省略，因为使用伪终端的应用程序通常设计为父进程在子进程结束时也会终止。

### 注意

BSD 衍生系统提供了两个相关的非标准函数用于与伪终端交互。其中第一个是*openpty()*，它打开一个伪终端对，返回主端和从端的文件描述符，选项上返回从端设备的名称，并可以根据类似于*slaveTermios*和*slaveWS*的参数设置终端属性和窗口大小。另一个函数是*forkpty()*，它与我们的*ptyFork()*相同，只是没有提供*snLen*参数的类似物。在 Linux 上，这两个函数由*glibc*提供，并在*openpty(3)*手册页中有文档说明。

示例 64-2. *ptyFork()*的实现

```
`pty/pty_fork.c`
    #include <fcntl.h>
    #include <termios.h>
    #include <sys/ioctl.h>
    #include "pty_master_open.h"
    #include "pty_fork.h"                   /* Declares ptyFork() */
    #include "tlpi_hdr.h"

    #define MAX_SNAME 1000

    pid_t
    ptyFork(int *masterFd, char *slaveName, size_t snLen,
            const struct termios *slaveTermios, const struct winsize *slaveWS)
    {
        int mfd, slaveFd, savedErrno;
        pid_t childPid;
        char slname[MAX_SNAME];

        mfd = ptyMasterOpen(slname, MAX_SNAME);
        if (mfd == -1)
            return -1;

    if (slaveName != NULL) {            /* Return slave name to caller */
            if (strlen(slname) < snLen) {
                strncpy(slaveName, slname, snLen);

            } else {                        /* 'slaveName' was too small */
                close(mfd);
                errno = EOVERFLOW;
                return -1;
            }
        }

    childPid = fork();

        if (childPid == -1) {               /* fork() failed */
            savedErrno = errno;             /* close() might change 'errno' */
            close(mfd);                     /* Don't leak file descriptors */
            errno = savedErrno;
            return -1;
        }

    if (childPid != 0) {                /* Parent */
            *masterFd = mfd;                /* Only parent gets master fd */
            return childPid;                /* Like parent of fork() */
        }

        /* Child falls through to here */

    if (setsid() == -1)                 /* Start a new session */
            err_exit("ptyFork:setsid");

    close(mfd);                         /* Not needed in child */

    slaveFd = open(slname, O_RDWR);     /* Becomes controlling tty */
        if (slaveFd == -1)
            err_exit("ptyFork:open-slave");

#ifdef TIOCSCTTY                        /* Acquire controlling tty on BSD */
        if (ioctl(slaveFd, TIOCSCTTY, 0) == -1)
            err_exit("ptyFork:ioctl-TIOCSCTTY");
    #endif

    if (slaveTermios != NULL)           /* Set slave tty attributes */
            if (tcsetattr(slaveFd, TCSANOW, slaveTermios) == -1)
                err_exit("ptyFork:tcsetattr");

    if (slaveWS != NULL)                /* Set slave tty window size */
            if (ioctl(slaveFd, TIOCSWINSZ, slaveWS) == -1)
                err_exit("ptyFork:ioctl-TIOCSWINSZ");

            /* Duplicate pty slave to be child's stdin, stdout, and stderr */

    if (dup2(slaveFd, STDIN_FILENO) != STDIN_FILENO)
            err_exit("ptyFork:dup2-STDIN_FILENO");
        if (dup2(slaveFd, STDOUT_FILENO) != STDOUT_FILENO)
            err_exit("ptyFork:dup2-STDOUT_FILENO");
        if (dup2(slaveFd, STDERR_FILENO) != STDERR_FILENO)
            err_exit("ptyFork:dup2-STDERR_FILENO");

        if (slaveFd > STDERR_FILENO)        /* Safety check */
            close(slaveFd);                 /* No longer need this fd */

        return 0;                           /* Like child of fork() */
    }
          `pty/pty_fork.c`
```

## 伪终端输入输出

伪终端对是一种类似于双向管道的结构。写入主端的任何内容会作为输入出现在从端，而写入从端的任何内容会作为输入出现在主端。

区分伪终端对和双向管道的关键点在于，从端像终端设备一样操作。从端按与普通控制终端相同的方式解释输入。例如，如果我们将*Control-C*字符（通常终端的*中断*字符）写入伪终端主端，从端将为其前台进程组生成一个`SIGINT`信号。就像常规终端一样，当伪终端从端以规范模式（默认模式）操作时，输入会逐行缓冲。换句话说，程序从伪终端从端读取时，只有在我们向伪终端主端写入换行符时，它才会看到（一行）输入。

像管道一样，伪终端有一个有限的容量。如果我们耗尽了这个容量，那么进一步的写入操作会被阻塞，直到伪终端另一端的进程消费了一些字节。

### 注意

在 Linux 上，伪终端的容量大约是每个方向 4 KB。

如果我们关闭所有指向伪终端主端的文件描述符，那么：

+   如果从设备有控制进程，则会向该进程发送`SIGHUP`信号（见第 34.6 节）。

+   从从属设备的*read()*返回文件结束符（0）。

+   向从设备的*write()*失败，错误为`EIO`。（在某些其他 UNIX 实现中，这种情况下*write()*会失败并返回错误`ENXIO`。）

如果我们关闭所有引用伪终端从设备的文件描述符，则：

+   从主设备的*read()*失败，并返回错误`EIO`。（在某些其他 UNIX 实现中，这种情况下*read()*会返回文件结束符。）

+   向主设备的*write()*成功，除非从设备的输入队列已满，在这种情况下，*write()*会阻塞。如果从设备随后被重新打开，则这些字节可以被读取。

UNIX 实现的行为在最后一种情况下差异很大。在某些 UNIX 实现中，*write()*会因错误`EIO`而失败。在其他实现中，*write()*会成功，但输出字节会被丢弃（即，如果重新打开从设备，则无法读取）。通常，这些差异不会造成问题。通常，主侧的进程会检测到从设备已经关闭，因为主设备的*read()*返回文件结束符或失败。此时，进程不会再向主设备执行写操作。

#### 数据包模式

*数据包模式*是一种机制，允许在伪终端主设备上运行的进程在以下与软件流控制相关的事件发生时，得到伪终端从设备的通知：

+   输入或输出队列被刷新；

+   终端输出被停止或重新启动（*Control-S/Control-Q*）；或

+   流控制已启用或禁用。

数据包模式有助于处理某些伪终端应用中的软件流控制，这些应用提供网络登录服务（例如，*telnet*和*login*）。

通过对引用伪终端主设备的文件描述符应用*ioctl()* `TIOCPKT`操作来启用数据包模式：

```
int arg;

arg = 1;                /* 1 == enable; 0 == disable */
if (ioctl(mfd, TIOCPKT, &arg) == -1)
    errExit("ioctl");
```

当数据包模式处于运行状态时，从伪终端主设备读取的内容要么是一个非零的控制字节，该字节是一个位掩码，指示从属设备上发生的状态变化，要么是一个 0 字节，后面跟着一个或多个写入伪终端从设备的数据字节。

当伪终端在数据包模式下发生状态变化时，*select()*会指示主设备发生了异常条件（即*exceptfds*参数），并且*poll()*会在*revents*字段中返回`POLLPRI`。（有关*select()*和*poll()*的描述，请参阅第六十三章。）

数据包模式在 SUSv3 中没有标准化，并且在其他 UNIX 实现中，一些细节有所不同。有关 Linux 中数据包模式的更多详细信息，包括用于指示状态变化的位掩码值，可以在*tty_ioctl(4)*手册页中找到。

## 实现*script(1)*

现在，我们准备实现标准*script(1)*程序的简单版本。该程序启动一个新的 shell 会话，并将该会话的所有输入和输出记录到一个文件中。本书中展示的大部分 shell 会话都是通过*script*记录的。

在正常的登录会话中，shell 直接连接到用户的终端。当我们运行*script*时，它将自己置于用户终端和 shell 之间，并使用伪终端对创建一个通信通道，连接自己和 shell（参见图 64-4）。shell 连接到伪终端从端，*script*进程连接到伪终端主端。*script*进程充当用户的代理，将输入内容写入伪终端主端，并从伪终端主端读取输出并写入用户终端。

此外，*script* 会生成一个输出文件（默认为`typescript`），该文件包含所有在伪终端主端上输出的字节副本。这意味着它不仅记录了 shell 会话生成的输出，还记录了输入内容。输入被记录是因为，像传统的终端设备一样，内核通过将输入字符复制到终端输出队列来回显输入（参见图 62-1，以及在检索和修改终端属性中）。然而，当禁用终端回显时（例如，读取密码的程序会这么做），伪终端从端的输入不会被复制到从端输出队列中，因此不会被复制到 script 输出文件中。

![脚本程序](img/64-4_PTY-script-scale90.png.jpg)图 64-4. *script* 程序

我们的*script*实现如示例 64-3")所示。该程序执行以下步骤：

+   获取程序运行时终端的属性和窗口大小 ![](img/U001.png)。这些信息会传递给后续调用的*ptyFork()*，用于设置伪终端从设备的相应值。

+   调用我们的*ptyFork()*函数（参见示例 64-2")），创建一个通过伪终端对连接的子进程 ![](img/U002.png)。

+   在*ptyFork()*调用后，子进程执行一个 shell ![](img/U004.png)。选择的 shell 由`SHELL`环境变量的设置决定 ![](img/U003.png)。如果没有设置`SHELL`变量或其值为空字符串，则子进程执行`/bin/sh`。

+   在*ptyFork()*调用之后，父进程执行以下步骤：

    +   打开输出脚本文件 ![](img/U005.png)。如果提供了命令行参数，则使用该参数作为脚本文件的名称。如果没有提供命令行参数，则使用默认名称`typescript`。

    +   将终端置于原始模式（使用*ttySetRaw()*函数，如示例 62-3、示例：设置原始模式和 cbreak 模式所示），这样所有输入字符都会直接传递给*script*程序，而不会被终端驱动程序修改 ![](img/U006.png)。*script*程序输出的字符也同样不会被终端驱动程序修改。

        ### 注意

        终端处于原始模式并不意味着未解析的原始控制字符会被传送到 shell，或者传送到伪终端从设备的前台进程组，或者该进程组的输出会直接传送到用户的终端。实际上，终端特殊字符的解析是在从设备内部进行的（除非从设备也被应用程序显式地设置为原始模式）。通过将用户的终端置于原始模式，我们防止了输入输出字符的*第二*次解析。

+   调用*atexit()*安装退出处理程序，在程序终止时将终端重置为原始模式 ![](img/U007.png)。

+   执行一个循环，在终端和伪终端主设备之间双向传输数据 ![](img/U008.png)。在每次循环迭代中，程序首先使用*select()*（*select()*系统调用系统调用")）监控终端和伪终端主设备的输入 ![](img/U009.png)。如果终端有可用输入，程序读取部分输入并将其写入伪终端主设备 ![](img/U010.png)。同样地，如果伪终端主设备有可用输入，程序读取部分输入并将其写入终端以及脚本文件 ![](img/U010.png)。该循环会持续执行，直到文件结束或在监视的文件描述符上检测到错误。

示例 64-3. 简单实现*script(1)*

```
`pty/script.c`
    #include <sys/stat.h>
    #include <fcntl.h>
    #include <libgen.h>
    #include <termios.h>
    #include <sys/select.h>
    #include "pty_fork.h"           /* Declaration of ptyFork() */
    #include "tty_functions.h"      /* Declaration of ttySetRaw() */
    #include "tlpi_hdr.h"

    #define BUF_SIZE 256
    #define MAX_SNAME 1000

    struct termios ttyOrig;

    static void             /* Reset terminal mode on program exit */
    ttyReset(void)
    {
        if (tcsetattr(STDIN_FILENO, TCSANOW, &ttyOrig) == -1)
            errExit("tcsetattr");
    }

    int
    main(int argc, char *argv[])
    {
        char slaveName[MAX_SNAME];
        char *shell;
        int masterFd, scriptFd;
        struct winsize ws;
        fd_set inFds;
        char buf[BUF_SIZE];
        ssize_t numRead;
        pid_t childPid;

    if (tcgetattr(STDIN_FILENO, &ttyOrig) == -1)
            errExit("tcgetattr");
        if (ioctl(STDIN_FILENO, TIOCGWINSZ, &ws) < 0)
            errExit("ioctl-TIOCGWINSZ");

    childPid = ptyFork(&masterFd, slaveName, MAX_SNAME, &ttyOrig, &ws);
        if (childPid == -1)
            errExit("ptyFork");

        if (childPid == 0) {        /* Child: execute a shell on pty slave */
        shell = getenv("SHELL");
            if (shell == NULL || *shell == '\0')
                shell = "/bin/sh";

        execlp(shell, shell, (char *) NULL);
            errExit("execlp");      /* If we get here, something went wrong */
        }

            /* Parent: relay data between terminal and pty master */

    scriptFd = open((argc > 1) ? argv[1] : "typescript",
                            O_WRONLY | O_CREAT | O_TRUNC,
                            S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP |
                                    S_IROTH | S_IWOTH);
        if (scriptFd == -1)
            errExit("open typescript");

    ttySetRaw(STDIN_FILENO, &ttyOrig);

    if (atexit(ttyReset) != 0)
            errExit("atexit");

    for (;;) {
            FD_ZERO(&inFds);
            FD_SET(STDIN_FILENO, &inFds);
            FD_SET(masterFd, &inFds);

        if (select(masterFd + 1, &inFds, NULL, NULL, NULL) == -1)
                errExit("select");

        if (FD_ISSET(STDIN_FILENO, &inFds)) {   /* stdin —> pty */
                numRead = read(STDIN_FILENO, buf, BUF_SIZE);
                if (numRead <= 0)
                    exit(EXIT_SUCCESS);

                if (write(masterFd, buf, numRead) != numRead)
                    fatal("partial/failed write (masterFd)");
            }

        if (FD_ISSET(masterFd, &inFds)) {       /* pty —> stdout+file */
                numRead = read(masterFd, buf, BUF_SIZE);
                if (numRead <= 0)
                    exit(EXIT_SUCCESS);

                if (write(STDOUT_FILENO, buf, numRead) != numRead)
                    fatal("partial/failed write (STDOUT_FILENO)");
                if (write(scriptFd, buf, numRead) != numRead)
                    fatal("partial/failed write (scriptFd)");
            }
        }
    }
         `pty/script.c`
```

在以下的 shell 会话中，我们演示了示例 64-3")中程序的使用。我们首先显示由*xterm*使用的伪终端的名称以及登录 shell 的进程 ID。这些信息在后续的 shell 会话中非常有用。

```
$ `tty`
/dev/pts/1
$ `echo $$`
7979
```

然后，我们启动一个*script*程序实例，它调用一个子 shell。再次，我们显示正在运行该 shell 的终端名称和 shell 的进程 ID：

```
$ `./script`
$ `tty`
/dev/pts/24                         *Pseudoterminal slave opened by* *script*
$ `echo $$`
29825                               *PID of subshell process started by* *script*
```

现在我们使用*ps(1)*显示关于两个 shell 和运行*script*的进程的信息，然后终止*script*启动的 shell：

```
$ `ps -p 7979 -p 29825 -C script -o "pid ppid sid tty cmd"`
  PID  PPID   SID TT       CMD
 7979  7972  7979 pts/1    /bin/bash
29824  7979  7979 pts/1    ./script
29825 29824 29825 pts/24   /bin/bash
$ `exit`
```

*ps(1)*的输出显示了登录 shell、运行*script*的进程以及*script*启动的子 shell 之间的父子关系。

到此，我们已返回登录 shell。显示`typescript`文件的内容，展示了*script*运行期间生成的所有输入和输出记录：

```
$ `cat typescript`
$ tty
/dev/pts/24
$ echo $$
29825
$ ps -p 7979 -p 29825 -C script -o "pid ppid sid tty cmd"
  PID  PPID   SID TT       CMD
 7979  7972  7979 pts/1    /bin/bash
29824  7979  7979 pts/1    ./script
29825 29824 29825 pts/24   /bin/bash
$ exit
```

## 终端属性和窗口大小

主设备和从设备共享终端属性（*termios*）和窗口大小（*winsize*）结构。（这两种结构在第六十二章中有描述。）这意味着运行在伪终端主设备上的程序可以通过对主设备文件描述符应用*tcsetattr()*和*ioctl()*来更改伪终端从设备的这些属性。

改变终端属性在*script*程序中可能是有用的一个例子。假设我们在终端仿真器窗口中运行*script*，并改变窗口的大小。在这种情况下，终端仿真器程序将通知内核相应终端设备大小的变化，但此变化不会反映在伪终端从设备的单独内核记录中（见图 64-4）。因此，在伪终端从设备上运行的屏幕导向程序（如*vi*）会产生混乱的输出，因为它们对终端窗口大小的理解与终端的实际大小不同。我们可以通过以下方法解决这个问题：

1.  在*script*父进程中安装一个`SIGWINCH`处理程序，以便当终端窗口大小发生变化时，它能够接收到信号。

1.  当*script*父进程接收到`SIGWINCH`信号时，它使用*ioctl()* `TIOCGWINSZ`操作来获取与其标准输入关联的终端窗口的*winsize*结构。然后，它使用这个结构在*ioctl()* `TIOCSWINSZ`操作中设置伪终端主设备的窗口大小。

1.  如果新的伪终端窗口大小与旧的大小不同，内核会为伪终端从设备的前台进程组生成一个`SIGWINCH`信号。像*vi*这样的屏幕处理程序被设计成捕捉该信号，并执行一个*ioctl()* `TIOCGWINSZ`操作来更新它们对终端窗口大小的理解。

我们在第 62.9 节中描述了终端窗口大小以及*ioctl()* `TIOCGWINSZ`和`TIOCSWINSZ`操作的详细信息。

## BSD 伪终端

本章大部分内容集中在 UNIX 98 伪终端上，因为这是 SUSv3 中标准化的伪终端样式，因此应在所有新程序中使用。然而，在旧版应用程序中或将程序从其他 UNIX 实现移植到 Linux 时，我们有时会遇到 BSD 伪终端。因此，我们现在考虑 BSD 伪终端的细节。

### 注意

在 Linux 中，BSD 伪终端的使用已被弃用。从 Linux 2.6.4 开始，BSD 伪终端支持成为一个可选的内核组件，可以通过 `CONFIG_LEGACY_PTYS` 选项进行配置。

BSD 伪终端与 UNIX 98 伪终端的区别仅在于如何找到并打开伪终端主设备和从设备的细节。一旦主设备和从设备被打开，BSD 伪终端与 UNIX 98 伪终端的操作方式相同。

使用 UNIX 98 伪终端时，我们通过调用 *posix_openpt()* 获取一个未使用的伪终端主设备，该函数打开 `/dev/ptmx`，即伪终端主设备克隆设备。然后，我们使用 *ptsname()* 获取对应的伪终端从设备的名称。相比之下，使用 BSD 伪终端时，主设备和从设备对是在 `/dev` 目录下预创建的条目。每个主设备的名称形式为 `/dev/pty`*xy*，其中 *x* 被 `[p-za-e]` 范围内的一个字母替换，*y* 被 `[0-9a-f]` 范围内的一个字母替换。与特定伪终端主设备对应的从设备名称形式为 `/dev/tty`*xy*。因此，例如，设备 `/dev/ptyp0` 和 `/dev/ttyp0` 构成一对 BSD 伪终端。

### 注意

UNIX 实现提供的 BSD 伪终端对的数量和名称有所不同，有些实现默认仅提供 32 对。大多数实现至少提供 32 个主设备，其名称在 `/dev/pty[pq][0-9a-f]` 范围内，并附带相应的从设备。

为了找到一个未使用的伪终端对，我们执行一个循环，逐个尝试打开每个主设备，直到其中一个成功打开。在执行此循环时，我们可能会遇到调用 *open()* 时的两个错误：

+   如果给定的主设备名称不存在，*open()* 会返回错误 `ENOENT`。通常，这意味着我们已经遍历了系统上所有伪终端主设备名称，而没有找到空闲设备（即，没有列出上述完整的设备范围）。

+   如果主设备正在使用中，*open()* 会返回错误 `EIO`。我们可以忽略这个错误，并尝试下一个设备。

### 注意

在 HP-UX 11 中，尝试打开一个正在使用中的 BSD 伪终端主设备时，*open()* 会失败并返回错误 `EBUSY`。

一旦找到一个可用的主设备，我们可以通过将主设备名称中的 `pty` 替换为 `tty` 来获得对应从设备的名称。然后，我们可以使用 *open()* 打开从设备。

### 注意

在 BSD 伪终端中，没有类似*grantpt()*的函数来更改伪终端从设备的所有权和权限。如果需要这样做，必须明确调用*chown()*（只有在特权程序中才能实现）和*chmod()*，或者编写一个设置用户 ID 的程序（如*pt_chown*），为非特权程序执行此任务。

示例 64-4")展示了使用 BSD 伪终端重新实现第 64.3 节的*ptyMasterOpen()*函数。替换这个实现即可使我们的*script*程序（第 64.6 节）在 BSD 伪终端上工作。

示例 64-4. 使用 BSD 伪终端实现*ptyMasterOpen()*

```
`pty/pty_master_open_bsd.c`
#include <fcntl.h>
#include "pty_master_open.h"            /* Declares ptyMasterOpen() */
#include "tlpi_hdr.h"

#define PTYM_PREFIX     "/dev/pty"
#define PTYS_PREFIX     "/dev/tty"
#define PTY_PREFIX_LEN  (sizeof(PTYM_PREFIX) - 1)
#define PTY_NAME_LEN    (PTY_PREFIX_LEN + sizeof("XY"))
#define X_RANGE         "pqrstuvwxyzabcde"
#define Y_RANGE         "0123456789abcdef"

int
ptyMasterOpen(char *slaveName, size_t snLen)
{
    int masterFd, n;
    char *x, *y;
    char masterName[PTY_NAME_LEN];

    if (PTY_NAME_LEN > snLen) {
        errno = EOVERFLOW;
        return -1;
    }

    memset(masterName, 0, PTY_NAME_LEN);
    strncpy(masterName, PTYM_PREFIX, PTY_PREFIX_LEN);

    for (x = X_RANGE; *x != '\0'; x++) {
        masterName[PTY_PREFIX_LEN] = *x;

        for (y = Y_RANGE; *y != '\0'; y++) {
            masterName[PTY_PREFIX_LEN + 1] = *y;

            masterFd = open(masterName, O_RDWR);

            if (masterFd == -1) {
                if (errno == ENOENT)    /* No such file */
                    return -1;          /* Probably no more pty devices */
                else                    /* Other error (e.g., pty busy) */
                    continue;

            } else {            /* Return slave name corresponding to master */
                n = snprintf(slaveName, snLen, "%s%c%c", PTYS_PREFIX, *x, *y);
                if (n >= snLen) {
                    errno = EOVERFLOW;
                    return -1;
                } else if (n == -1) {
                    return -1;
                }

                return masterFd;
            }
        }
    }

    return -1;                  /* Tried all ptys without success */
}
     `pty/pty_master_open_bsd.c`
```

## 总结

伪终端对由一个连接的主设备和从设备组成。两个设备共同提供了一个双向 IPC 通道。伪终端的好处在于，在从设备端，我们可以连接一个由打开主设备的程序驱动的面向终端的程序。伪终端从设备的行为就像一个常规终端一样。可以应用于常规终端的所有操作也可以应用于从设备，且从主设备传输到从设备的输入与在常规终端上键盘输入的解释方式相同。

伪终端的一个常见用途是在提供网络登录服务的应用程序中。然而，伪终端也被广泛应用于许多其他程序中，如终端仿真器和*script(1)*程序。

在 System V 和 BSD 中出现了不同的伪终端 API。Linux 同时支持这两种 API，但 System V API 构成了 SUSv3 标准化的伪终端 API 的基础。

## 练习

1.  当用户在运行示例 64-3 实现")程序时，输入结束符字符（通常是*Control-D*），*script*父进程和子 shell 进程是以什么顺序终止的？为什么？

1.  对示例 64-3 实现")中的程序（`script.c`）做以下修改：

    1.  标准*script(1)*程序在输出文件的开始和结束处添加显示脚本开始和结束时间的行。请添加此功能。

    1.  添加代码来处理如第 64.7 节所述的终端窗口大小变化。你可以参考示例 62-5 中的程序（`demo_SIGWINCH.c`）来测试此功能。

1.  修改示例 64-3 的简单实现")（`script.c`）中的程序，将*select()*的使用替换为一对进程：一个用于处理从终端到伪终端主设备的数据传输，另一个用于处理反方向的数据传输。

1.  修改示例 64-3 的简单实现")（`script.c`）中的程序，添加一个带时间戳的记录功能。每次程序向`typescript`文件写入字符串时，它还应该将带时间戳的字符串写入第二个文件（比如`typescript.timed`）。写入第二个文件的记录可能具有以下通用形式：

    ```
    <timestamp> <space> <string> <newline>
    ```

    *时间戳*应以文本形式记录，表示自脚本会话开始以来的毫秒数。以文本形式记录时间戳的优点是生成的文件具有可读性。在*string*内，真实的换行符需要转义。一种可能的方式是将换行符记录为 2 个字符的序列`\n`，将反斜杠记录为`\\`。

    编写第二个程序`script_replay.c`，该程序读取带时间戳的脚本文件，并以最初写入的相同速率将其内容显示到标准输出。结合这两个程序，提供了一个简单的 shell 会话日志记录和回放功能。

1.  实现客户端和服务器程序，提供一个简单的*telnet*风格的远程登录功能。设计服务器以并发处理客户端（第 60.1 节）。图 64-3 显示了为每个客户端登录所需建立的设置。该图未显示的是父服务器进程，它处理来自客户端的传入套接字连接并创建服务器子进程来处理每个连接。请注意，所有身份验证用户和启动登录 shell 的工作都可以通过让*ptyFork()*创建的（孙）子进程执行*login(1)*来处理。

1.  在上一练习中开发的程序中添加代码，以便在登录会话开始和结束时更新登录帐户文件（第四十章）。

1.  假设我们执行一个长期运行的程序，该程序缓慢生成的输出被重定向到文件或管道，如以下示例所示：

    ```
    $ `longrunner | grep str`
    ```

    上述场景的一个问题是，默认情况下，*stdio*包仅在*stdio*缓冲区被填满时才刷新标准输出。这意味着*longrunner*程序的输出将以长时间间隔分隔的突发形式出现。解决此问题的一种方法是编写一个程序，执行以下操作：

    1.  创建一个伪终端。

    1.  执行命令行参数中指定的程序，并将标准文件描述符连接到伪终端从设备。

    1.  从伪终端主机读取输出并立即写入标准输出（`STDOUT_FILENO`，文件描述符 1），同时，从终端读取输入并写入伪终端主机，以便 execed 程序可以读取。

        这样的程序，我们称之为 *unbuffer*，其使用方法如下：

        ```
        $ `./unbuffer longrunner | grep str`
        ```

        编写 *unbuffer* 程序。（这个程序的大部分代码将与示例 64-3 的简单实现")的代码类似。）

1.  编写一个实现脚本语言的程序，该语言可以用来以非交互模式驱动 *vi*。由于 *vi* 期望从终端运行，该程序将需要使用伪终端。
