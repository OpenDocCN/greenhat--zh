## 第十二章 系统与进程信息

本章我们将探讨访问各种系统和进程信息的方法。本章的主要内容是讨论 `/proc` 文件系统。我们还将介绍 *uname()* 系统调用，用于检索各种系统标识符。

## `/proc` 文件系统

在旧版 UNIX 实现中，通常没有简便的方式来反向分析（或更改）内核的属性，以回答如下问题：

+   系统上运行着多少个进程，谁拥有它们？

+   一个进程打开了哪些文件？

+   当前哪些文件被锁定，哪些进程持有锁？

+   系统上正在使用哪些套接字？

一些旧版 UNIX 实现通过允许特权程序深入内核内存中的数据结构来解决这个问题。然而，这种方法存在各种问题。特别是，它需要对内核数据结构有专门的了解，并且这些结构可能会在不同的内核版本之间发生变化，这就需要依赖这些结构的程序进行重写。

为了便于访问内核信息，许多现代 UNIX 实现提供了 `/proc` 虚拟文件系统。这个文件系统位于 `/proc` 目录下，包含了各种暴露内核信息的文件，允许进程方便地读取这些信息，并在某些情况下，使用正常的文件 I/O 系统调用来修改它们。`/proc` 文件系统被称为虚拟文件系统，因为它包含的文件和子目录并不存储在磁盘上，而是由内核在进程访问时“动态”创建。

在本节中，我们概述了 `/proc` 文件系统。在后续章节中，我们将根据每个章节的主题描述特定的 `/proc` 文件。虽然许多 UNIX 实现提供了 `/proc` 文件系统，但 SUSv3 并未指定这一文件系统；本书中描述的细节是 Linux 特有的。

### 获取关于进程的信息：`/proc/`*PID*

对于系统中的每个进程，内核提供了一个对应的目录，命名为 `/proc/`*PID*，其中 *PID* 是进程的 ID。在这个目录中，包含了关于该进程的各种文件和子目录。例如，我们可以通过查看 `/proc/1` 目录下的文件来获取关于 *init* 进程的信息，*init* 进程的进程 ID 始终为 1。

每个 `/proc/`*PID* 目录下都有一个名为 `status` 的文件，提供关于该进程的各种信息：

```
$ `cat /proc/1/status`
Name:   init                            *Name of command run by this process*
State:  S (sleeping)                    *State of this process*
Tgid:   1                               *Thread group ID (traditional PID, getpid())*
Pid:    1                               *Actually, thread ID (gettid())*
PPid:   0                               *Parent process ID*
TracerPid:      0                       *PID of tracing process (0 if not traced)*
Uid:    0       0       0       0       *Real, effective, saved set, and FS UIDs*
Gid:    0       0       0       0       *Real, effective, saved set, and FS GIDs*
FDSize: 256
                             *# of file descriptor slots currently allocated*
Groups:                                 *Supplementary group IDs*
VmPeak:      852 kB                     *Peak virtual memory size*
VmSize:      724 kB                     *Current virtual memory size*
VmLck:         0 kB                     *Locked memory*
VmHWM:       288 kB                     *Peak resident set size*
VmRSS:       288 kB                     *Current resident set size*
VmData:      148 kB                     *Data segment size*
VmStk:        88 kB                     *Stack size*
VmExe:       484 kB                     *Text (executable code) size*
VmLib:         0 kB                     *Shared library code size*
VmPTE:        12 kB                     *Size of page table (since 2.6.10)*
Threads:        1                       *# of threads in this thread's thread group*
SigQ:   0/3067                          *Current/max. queued signals (since 2.6.12)*
SigPnd: 0000000000000000                *Signals pending for thread*
ShdPnd: 0000000000000000                *Signals pending for process (since 2.6)*
SigBlk: 0000000000000000                *Blocked signals*
SigIgn: fffffffe5770d8fc                *Ignored signals*
SigCgt: 00000000280b2603                *Caught signals*
CapInh: 0000000000000000                *Inheritable capabilities*
CapPrm: 00000000ffffffff                *Permitted capabilities*
CapEff: 00000000fffffeff                *Effective capabilities*
CapBnd: 00000000ffffffff                *Capability bounding set (since 2.6.26)*
Cpus_allowed:   1                       *CPUs allowed, mask (since 2.6.24)*
Cpus_allowed_list:      0               *Same as above, list format (since 2.6.26)*
Mems_allowed:   1                       *Memory nodes allowed, mask (since 2.6.24)*
Mems_allowed_list:      0               *Same as above, list format (since 2.6.26)*
voluntary_ctxt_switches:     6998       *Voluntary context switches (since 2.6.23)*
nonvoluntary_ctxt_switches:  107        *Involuntary context switches (since 2.6.23)*
Stack usage:    8 kB                    *Stack usage high-water mark (since 2.6.32)*
```

上述输出来自内核 2.6.32。正如文件输出中附带的*since*注释所示，这个文件的格式随着时间的推移有所变化，在不同的内核版本中增加了新字段（在少数情况下，删除了字段）。（除了上面提到的 Linux 2.6 更改外，Linux 2.4 还增加了*Tgid*、*TracerPid*、*FDSize* 和 *Threads* 字段。）

由于该文件内容随时间发生变化，这提出了关于使用`/proc`文件的一个普遍问题：当这些文件包含多个条目时，我们应该采取防御性解析方式——在这种情况下，应该查找包含特定字符串（例如*PPid:*）的行进行匹配，而不是通过（逻辑的）行号处理文件。

表 12-1 列出了每个`/proc/`*PID*目录中发现的其他一些文件。

表 12-1. 每个`/proc/`*PID*目录中选定的文件

| 文件 | 描述（进程属性） |
| --- | --- |
| `cmdline` | 用`\0`分隔的命令行参数 |
| `cwd` | 指向当前工作目录的符号链接 |
| `environ` | 环境变量列表，格式为*NAME=value*对，使用`\0`分隔 |
| `exe` | 指向正在执行的文件的符号链接 |
| `fd` | 包含指向此进程打开文件的符号链接的目录 |
| `maps` | 内存映射 |
| `mem` | 进程虚拟内存（必须在 I/O 之前使用*lseek()*定位到有效偏移量） |
| `mounts` | 该进程的挂载点 |
| `root` | 指向根目录的符号链接 |
| `status` | 各种信息（例如，进程 ID、凭证、内存使用、信号等） |
| `task` | 包含进程中每个线程的一个子目录（Linux 2.6） |

#### `/proc/`*PID*`/fd`目录

`/proc/`*PID*`/fd`目录包含一个符号链接，指向该进程打开的每个文件描述符。每个符号链接的名称与描述符编号匹配；例如，`/proc/1968/1`是进程 1968 标准输出的符号链接。有关更多信息，请参见`/dev/fd`目录。

作为一种方便的方式，任何进程都可以通过符号链接`/proc/self`访问自己的`/proc/`*PID*目录。

#### 线程：`/proc/`*PID*`/task`目录

Linux 2.4 引入了线程组的概念，以正确支持 POSIX 线程模型。由于某些属性对于线程组中的线程是不同的，Linux 2.4 在`/proc/`*PID*目录下添加了一个`task`子目录。对于该进程中的每个线程，内核提供了一个名为`/proc/`*PID*`/task/`*TID*的子目录，其中*TID*是线程的线程 ID。（这个数字与通过线程中的*gettid()*调用返回的数字相同。）

每个`/proc/`*PID*`/task/`*TID*子目录下有一组与`/proc/`*PID*下的文件和目录完全相同的内容。由于线程共享许多属性，这些文件中的许多信息对于进程中的每个线程都是相同的。然而，在合适的情况下，这些文件会显示每个线程的不同信息。例如，在线程组的`/proc/`*PID*`/task/`*TID*`/status`文件中，*State*、*Pid*、*SigPnd*、*SigBlk*、*CapInh*、*CapPrm*、*CapEff*和*CapBnd*等字段可能对于每个线程是不同的。

### `/proc`下的系统信息

`/proc`下的各种文件和子目录提供了对系统范围内信息的访问。图 12-1 中显示了一些这些文件。

在图 12-1 中显示的许多文件在本书的其他地方有描述。表 12-2 总结了图 12-1 中所示的`/proc`子目录的一般用途。

表 12-2：选定的`/proc`子目录的用途

| 目录 | 此目录下文件公开的信息 |
| --- | --- |
| `/proc` | 各种系统信息 |
| `/proc/net` | 网络和套接字的状态信息 |
| `/proc/sys/fs` | 与文件系统相关的设置 |
| `/proc/sys/kernel` | 各种常规内核设置 |
| `/proc/sys/net` | 网络和套接字设置 |
| `/proc/sys/vm` | 内存管理设置 |
| `/proc/sysvipc` | 关于 System V IPC 对象的信息 |

### 访问 `/proc` 文件

`/proc`下的文件通常通过 shell 脚本访问（大多数包含多个值的`/proc`文件可以通过 Python 或 Perl 等脚本语言轻松解析）。例如，我们可以使用以下 shell 命令修改和查看`/proc`文件的内容：

```
# `echo 100000 > /proc/sys/kernel/pid_max`
# `cat /proc/sys/kernel/pid_max`
100000
```

`/proc`文件也可以通过程序使用常规的文件 I/O 系统调用访问。在访问这些文件时有一些限制：

+   某些`/proc`文件是只读的；也就是说，它们仅用于显示内核信息，不能用来修改这些信息。这适用于大多数`/proc/`*PID* 目录下的文件。

+   某些`/proc`文件仅能由文件所有者（或特权进程）读取。例如，`/proc/`*PID* 下的所有文件都归拥有对应进程的用户所有，并且某些文件（如 `/proc/`*PID*`/environ`）的读取权限仅授予文件所有者。

+   除了 `/proc/`*PID* 子目录中的文件外，`/proc`下的大多数文件归*root*所有，并且只有*root*可以修改那些可修改的文件。

![图 12-1：`/proc`下选定的文件和子目录](img/12-1_SYSINFO-procfs.png.jpg)图 12-1：`/proc`下选定的文件和子目录

#### 访问 `/proc/`*PID* 目录中的文件

`/proc/`*PID* 目录是易变的。当具有相应进程 ID 的进程创建时，这些目录就会生成，当进程终止时，目录也会消失。这意味着，如果我们确定某个 `/proc/`*PID* 目录存在，那么我们需要妥善处理在尝试打开该目录下的文件时，可能该进程已经终止，而相应的 `/proc/`*PID* 目录也已被删除的情况。

#### 示例程序

示例 12-1 演示了如何读取和修改 `/proc` 文件。此程序读取并显示 `/proc/sys/kernel/pid_max` 的内容。如果提供了命令行参数，程序将使用该值更新文件。这个文件（在 Linux 2.6 中新增）指定了进程 ID 的上限（进程 ID 和父进程 ID）。下面是使用此程序的示例：

```
$ `su`                            *Privilege is required to update* pid_max *file*
Password:
# `./procfs_pidmax 10000`
Old value: 32768
/proc/sys/kernel/pid_max now contains 10000
```

示例 12-1. 访问 `/proc/sys/kernel/pid_max`

```
`sysinfo/procfs_pidmax.c`
#include <fcntl.h>
#include "tlpi_hdr.h"

#define MAX_LINE 100

int
main(int argc, char *argv[])
{
    int fd;
    char line[MAX_LINE];
    ssize_t n;

    fd = open("/proc/sys/kernel/pid_max", (argc > 1) ? O_RDWR : O_RDONLY);
    if (fd == -1)
        errExit("open");

    n = read(fd, line, MAX_LINE);
    if (n == -1)
        errExit("read");

    if (argc > 1)
        printf("Old value: ");
    printf("%.*s", (int) n, line);

    if (argc > 1) {
        if (write(fd, argv[1], strlen(argv[1])) != strlen(argv[1]))
            fatal("write() failed");

        system("echo /proc/sys/kernel/pid_max now contains "
               "`cat /proc/sys/kernel/pid_max`");
    }

    exit(EXIT_SUCCESS);
}
     `sysinfo/procfs_pidmax.c`
```

## 系统标识：*uname()*

*uname()* 系统调用返回有关应用程序所在主机系统的一系列标识信息，这些信息存储在 *utsbuf* 指向的结构体中。

```
#include <sys/utsname.h>

int `uname`(struct utsname **utsbuf*);
```

### 注意

成功时返回 0，出错时返回 -1

*utsbuf* 参数是指向 *utsname* 结构体的指针，定义如下：

```
#define _UTSNAME_LENGTH 65

struct utsname {
    char sysname[_UTSNAME_LENGTH];      /* Implementation name */
    char nodename[_UTSNAME_LENGTH];     /* Node name on network */
    char release[_UTSNAME_LENGTH];      /* Implementation release level */
    char version[_UTSNAME_LENGTH];      /* Release version level */
    char machine[_UTSNAME_LENGTH];      /* Hardware on which system
                                           is running */
#ifdef _GNU_SOURCE                      /* Following is Linux-specific */
    char domainname[_UTSNAME_LENGTH];   /* NIS domain name of host */
#endif
};
```

SUSv3 指定了 *uname()*，但未定义 *utsname* 结构体各字段的长度，仅要求字符串以空字节结束。在 Linux 上，这些字段每个都是 65 字节长，包括终止的空字节空间。在某些 UNIX 实现中，这些字段较短；而在其他实现中（例如 Solaris），它们的长度可达到 257 字节。

*sysname*、*release*、*version* 和 *machine* 字段在 *utsname* 结构体中由内核自动设置。

### 注意

在 Linux 上，`/proc/sys/kernel` 目录中的三个文件提供了与 *utsname* 结构体中 *sysname*、*release* 和 *version* 字段相同的信息。这些只读文件分别是 `ostype`、`osrelease` 和 `version`。另一个文件 `/proc/version` 包含与这些文件相同的信息，还包括关于内核编译步骤的信息（即执行编译的用户的名字、进行编译的主机名称以及使用的 *gcc* 版本）。

*nodename* 字段返回通过 *sethostname()* 系统调用设置的值（有关此系统调用的详细信息，请参见手册页）。通常，这个名字类似于系统 DNS 域名中的主机名前缀。

*domainname* 字段返回通过 *setdomainname()* 系统调用设置的值（有关此系统调用的详细信息，请参阅手册页）。这是主机的网络信息服务（NIS）域名（与主机的 DNS 域名不同）。

### 注意

*gethostname()* 系统调用，作为 *sethostname()* 的反向操作，用于获取系统主机名。系统主机名也可以通过 *hostname(1)* 命令和 Linux 特有的 `/proc/hostname` 文件进行查看和设置。

*getdomainname()* 系统调用，作为 *setdomainname()* 的反向操作，用于获取 NIS 域名。NIS 域名也可以通过 *domainname(1)* 命令和 Linux 特有的 `/proc/domainname` 文件进行查看和设置。

*sethostname()* 和 *setdomainname()* 系统调用在应用程序中很少使用。通常，主机名和 NIS 域名是在启动时由启动脚本设置的。

示例 12-2") 中的程序显示了 *uname()* 返回的信息。以下是运行此程序时可能看到的输出示例：

```
$ `./t_uname`
Node name:   tekapo
System name: Linux
Release:     2.6.30-default
Version:     #3 SMP Fri Jul 17 10:25:00 CEST 2009
Machine:     i686
Domain name:
```

示例 12-2. 使用 *uname()*

```
`sysinfo/t_uname.c`
#define _GNU_SOURCE
#include <sys/utsname.h>
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    struct utsname uts;

    if (uname(&uts) == -1)
        errExit("uname");

    printf("Node name:   %s\n", uts.nodename);
    printf("System name: %s\n", uts.sysname);
    printf("Release:     %s\n", uts.release);
    printf("Version:     %s\n", uts.version);
    printf("Machine:     %s\n", uts.machine);
#ifdef _GNU_SOURCE
    printf("Domain name: %s\n", uts.domainname);
#endif
    exit(EXIT_SUCCESS);
}
      `sysinfo/t_uname.c`
```

## 概述

`/proc` 文件系统将一系列内核信息暴露给应用程序。每个 `/proc/`*PID* 子目录包含文件和子目录，提供关于与 *PID* 匹配的进程的信息。`/proc` 下的各种其他文件和目录则暴露系统范围的信息，程序可以读取这些信息，并在某些情况下进行修改。

*uname()* 系统调用使我们能够发现 UNIX 实现和运行应用程序的机器类型。

#### 更多信息

关于 `/proc` 文件系统的更多信息可以在 *proc(5)* 手册页、内核源文件 `Documentation/filesystems/proc.txt` 和 `Documentation/sysctl` 目录中的各种文件中找到。

## 练习

1.  编写一个程序，列出所有由程序命令行参数中指定的用户运行的进程的进程 ID 和命令名。（你可能会在 示例 8-1 中找到 *userIdFromName()* 函数，在 示例程序 中可能会有用。）这可以通过检查系统中所有 `/proc/`*PID*`/status` 文件中的 *Name:* 和 *Uid:* 行来完成。遍历系统中所有的 `/proc/`*PID* 目录需要使用 *readdir(3)*，该函数在第 18.8 节中进行了描述。确保你的程序正确处理 `/proc/`*PID* 目录在程序确定该目录存在和尝试打开相应的 `/proc/`*PID*`/status` 文件之间消失的情况。

1.  编写一个程序，绘制一棵显示系统中所有进程的层次父子关系的树状图，一直到*init*。对于每个进程，程序应该显示进程 ID 和正在执行的命令。程序的输出应类似于*pstree(1)*产生的输出，尽管不需要那么复杂。可以通过检查系统中所有`/proc/`*PID*`/status`文件中的*PPid:*行来找到每个进程的父进程。要小心处理在扫描所有`/proc/`*PID*目录时，进程的父进程（以及其`/proc/`*PID*目录）可能消失的情况。

1.  编写一个程序，列出所有打开特定文件路径名的进程。这可以通过检查所有`/proc/`*PID*`/fd/*`符号链接的内容来实现。这将需要使用嵌套循环，利用*readdir(3)*扫描所有`/proc/`*PID*目录，然后扫描每个`/proc/`*PID*目录中的所有`/proc/`*PID/*`fd`条目的内容。要读取`/proc/`*PID*`/fd/`*n*符号链接的内容，需要使用*readlink()*，在第 18.5 节中描述。
