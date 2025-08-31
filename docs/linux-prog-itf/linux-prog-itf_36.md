## 第三十六章：进程资源

每个进程都会消耗系统资源，如内存和 CPU 时间。本章介绍与资源相关的系统调用。我们从 *getrusage()* 系统调用开始，它允许进程监控自身或其子进程使用的资源。接下来，我们将讨论 *setrlimit()* 和 *getrlimit()* 系统调用，这些调用可以用于更改和获取调用进程对各种资源的使用限制。

## 进程资源使用

*getrusage()* 系统调用检索调用进程或其所有子进程使用的各种系统资源的统计信息。

```
#include <sys/resource.h>

int `getrusage`(int *who*, struct rusage **res_usage*);
```

### 注意

成功时返回 0，出错时返回 -1。

*who* 参数指定要检索资源使用信息的进程。它有以下几种值：

`RUSAGE_SELF`

返回调用进程的信息。

`RUSAGE_CHILDREN`

返回有关调用进程所有已终止并等待的子进程的信息。

`RUSAGE_THREAD`（自 Linux 2.6.26 起）

返回调用线程的信息。此值为 Linux 特定。

*res_usage* 参数是指向 *rusage* 类型结构体的指针，定义如 示例 36-1 所示。

示例 36-1. *rusage* 结构定义

```
struct rusage {
    struct timeval ru_utime;      /* User CPU time used */
    struct timeval ru_stime;      /* System CPU time used */
    long           ru_maxrss;     /* Maximum size of resident set (kilobytes)
                                     [used since Linux 2.6.32] */
    long           ru_ixrss;      /* Integral (shared) text memory size
                                     (kilobyte-seconds) [unused] */
    long           ru_idrss;      /* Integral (unshared) data memory used
                                     (kilobyte-seconds) [unused] */
    long           ru_isrss;      /* Integral (unshared) stack memory used
                                     (kilobyte-seconds) [unused] */
    long           ru_minflt;     /* Soft page faults (I/O not required) */
    long           ru_majflt;     /* Hard page faults (I/O required) */
    long           ru_nswap;      /* Swaps out of physical memory [unused] */
    long           ru_inblock;    /* Block input operations via file
                                     system [used since Linux 2.6.22] */
    long           ru_oublock;    /* Block output operations via file
                                     system [used since Linux 2.6.22] */
    long           ru_msgsnd;     /* IPC messages sent [unused] */
    long           ru_msgrcv;     /* IPC messages received [unused] */
    long           ru_nsignals;   /* Signals received [unused] */
    long           ru_nvcsw;      /* Voluntary context switches (process
                                     relinquished CPU before its time slice
                                     expired) [used since Linux 2.6] */
    long          ru_nivcsw;      /* Involuntary context switches (higher
                                     priority process became runnable or time
                                     slice ran out) [used since Linux 2.6] */
};
```

如 示例 36-1 中的注释所示，在 Linux 上，许多 *rusage* 结构中的字段未被 *getrusage()*（或 *wait3()* 和 *wait4()*）填写，或者仅在更新的内核版本中才会填写。在 Linux 上未使用的某些字段在其他 UNIX 实现中会被使用。Linux 提供这些字段，以便将来实现时，*rusage* 结构不需要修改，从而避免破坏现有的应用程序二进制文件。

### 注意

虽然 *getrusage()* 出现在大多数 UNIX 实现中，但它在 SUSv3 中仅有弱定义（只指定了 *ru_utime* 和 *ru_stime* 字段）。部分原因是 *rusage* 结构中的许多信息的含义依赖于具体实现。

*ru_utime* 和 *ru_stime* 字段是 *timeval* 类型的结构体（日历时间），分别返回进程在用户模式和内核模式下消耗的 CPU 时间的秒数和微秒数。（通过 *times()* 系统调用可以检索到类似的信息，详见第 10.7 节。）

### 注意

Linux 特定的 `/proc/`*PID*`/stat` 文件暴露了系统上所有进程的一些资源使用信息（CPU 时间和页面错误）。有关详细信息，请参见 *proc(5)* 手册页。

*getrusage()*返回的*rusage*结构体中的`RUSAGE_CHILDREN`操作包含了调用进程所有后代的资源使用统计信息。例如，如果我们有三个进程，分别为父进程、子进程和孙进程，那么当子进程对孙进程执行*wait()*时，孙进程的资源使用值会被加到子进程的`RUSAGE_CHILDREN`值中；当父进程对子进程执行*wait()*时，子进程和孙进程的资源使用值都会被加到父进程的`RUSAGE_CHILDREN`值中。相反，如果子进程没有对孙进程执行*wait()*，那么孙进程的资源使用值不会被记录在父进程的`RUSAGE_CHILDREN`值中。

对于`RUSAGE_CHILDREN`操作，*ru_maxrss*字段返回调用进程所有后代进程中的最大常驻集大小（而不是所有后代的总和）。

### 注意

SUSv3 规定，如果`SIGCHLD`被忽略（这样子进程的子进程就不会变成僵尸进程，不能被等待），那么子进程的统计信息不应该被加入到`RUSAGE_CHILDREN`返回的值中。然而，正如在忽略死去的子进程中所提到的，在 2.6.9 版本之前的内核中，Linux 偏离了这个要求——如果`SIGCHLD`被忽略，那么死去的子进程的资源使用值*会*被包含在返回的`RUSAGE_CHILDREN`值中。

## 进程资源限制

每个进程都有一组资源限制，用于限制进程可能消耗的各种系统资源。例如，如果我们担心某个程序可能消耗过多资源，可以在执行任意程序之前对进程设置资源限制。我们可以使用*ulimit*内建命令（在 C shell 中为*limit*）来设置 shell 的资源限制。这些限制会被 shell 创建的进程继承，用于执行用户命令。

### 注意

从内核 2.6.24 版本起，Linux 特有的`/proc/`*PID*`/limits`文件可以用于查看任何进程的所有资源限制。该文件由相应进程的实际用户 ID 所有，并且其权限只允许该用户 ID（或特权进程）读取。

*getrlimit()*和*setrlimit()*系统调用允许进程获取和修改其资源限制。

```
#include <sys/resource.h>

int `getrlimit`(int *resource*, struct rlimit **rlim*);
int `setrlimit`(int *resource*, const struct rlimit **rlim*);
```

### 注意

两者成功时返回 0，出错时返回-1。

*resource*参数标识要获取或更改的资源限制。*rlim*参数用于返回资源限制值`(getrlimit())`或指定新的资源限制值`(setrlimit())`，并且它是指向一个包含两个字段的结构体的指针：

```
struct rlimit {
    rlim_t rlim_cur;        /* Soft limit (actual process limit) */
    rlim_t rlim_max;        /* Hard limit (ceiling for rlim_cur) */
};
```

这些字段对应于资源的两个相关限制：*软限制*（*rlim_cur*）和*硬限制*（*rlim_max*）。（*rlim_t*数据类型是整数类型。）软限制控制进程可以消耗的资源量。进程可以将软限制调整为从 0 到硬限制的任何值。对于大多数资源，硬限制的唯一目的是为软限制提供上限。特权进程（`CAP_SYS_RESOURCE`）可以在任一方向调整硬限制（只要其值始终大于软限制），但非特权进程只能将硬限制调整为较低的值（不可逆）。在*rlim_cur*或*rlim_max*中的值`RLIM_INFINITY`表示无限（资源没有限制），无论是通过*getrlimit()*获取还是通过*setrlimit()*设置。

在大多数情况下，资源限制适用于特权进程和非特权进程。它们会被通过*fork()*创建的子进程继承，并在*exec()*调用过程中保留。

*getrlimit()*和*setrlimit()*的*resource*参数可以指定的值总结在表 36-1 和 setrlimit()的资源值")中，并在第 36.3 节中详细描述。

尽管资源限制是每个进程的属性，但在某些情况下，限制不仅针对该进程消耗的相应资源进行衡量，还要衡量所有具有相同实际用户 ID 的进程所消耗资源的总和。`RLIMIT_NPROC`限制是一个很好的例子，它限制了可创建的进程数量。仅对进程自身创建的子进程数量应用该限制是无效的，因为进程创建的每个子进程也能创建更多的子进程，依此类推。因此，限制是针对所有具有相同实际用户 ID 的进程的数量进行衡量的。然而，请注意，资源限制仅在设置了限制的进程中检查（即进程本身及其后代，后代继承限制）。如果另一个由相同实际用户 ID 拥有的进程未设置该限制（即，限制为无限）或设置了不同的限制，则该进程创建子进程的能力将根据其设置的限制进行检查。

在我们接下来描述每个资源限制时，我们将指出那些根据所有具有相同实际用户 ID 的进程所消耗的资源来衡量的限制。如果没有特别说明，那么资源限制仅针对进程自身消耗的资源进行衡量。

### 注意

请注意，在许多情况下，获取和设置资源限制的 shell 命令（*bash* 和 Korn shell 中的 *ulimit*，以及 C shell 中的 *limit*）使用的单位与 *getrlimit()* 和 *setrlimit()* 中使用的单位不同。例如，shell 命令通常以千字节为单位表示各种内存段的大小限制。

表 36-1. *getrlimit()* 和 *setrlimit()* 的资源值

| *resource* | 限制 | SUSv3 |
| --- | --- | --- |
| `RLIMIT_AS` | 进程虚拟内存大小（字节） | • |
| `RLIMIT_CORE` | 核心文件大小（字节） | • |
| `RLIMIT_CPU` | CPU 时间（秒） | • |
| `RLIMIT_DATA` | 进程数据段（字节） | • |
| `RLIMIT_FSIZE` | 文件大小（字节） | • |
| `RLIMIT_MEMLOCK` | 锁定的内存（字节） |   |
| `RLIMIT_MSGQUEUE` | 为实际用户 ID 分配的 POSIX 消息队列字节数（自 Linux 2.6.8 起） |   |
| `RLIMIT_NICE` | Nice 值（自 Linux 2.6.12 起） |   |
| `RLIMIT_NOFILE` | 最大文件描述符数量加一 | • |
| `RLIMIT_NPROC` | 实际用户 ID 的进程数量 |   |
| `RLIMIT_RSS` | 常驻集大小（字节；未实现） |   |
| `RLIMIT_RTPRIO` | 实时调度优先级（自 Linux 2.6.12 起） |   |
| `RLIMIT_RTTIME` | 实时 CPU 时间（微秒；自 Linux 2.6.25 起） |   |
| `RLIMIT_SIGPENDING` | 实际用户 ID 的已排队信号数量（自 Linux 2.6.8 起） |   |
| `RLIMIT_STACK` | 栈段大小（字节） | • |

#### 示例程序

在深入了解每个资源限制的具体情况之前，我们先来看一个资源限制使用的简单示例。示例 36-2 定义了 *printRlimit()* 函数，该函数显示一条消息，并显示指定资源的软限制和硬限制。

### 注意

*rlim_t* 数据类型通常与 *off_t* 以相同方式表示，用于处理 `RLIMIT_FSIZE`（文件大小资源限制）的表示。因此，当打印 *rlim_t* 值时（如在 示例 36-2 中所示），我们将其强制转换为 *long long* 类型，并使用 `%lld` *printf()* 格式说明符，如在 大文件 I/O 中所述。

示例 36-3 中的程序调用 *setrlimit()* 设置用户可以创建的进程数量的软限制和硬限制（`RLIMIT_NPROC`），使用 示例 36-2 的 *printRlimit()* 函数在更改前后显示限制，然后创建尽可能多的进程。当我们运行这个程序时，将软限制设置为 30，硬限制设置为 100，我们看到以下内容：

```
$ `./rlimit_nproc 30 100`
Initial maximum process limits:  soft=1024; hard=1024
New maximum process limits:      soft=30; hard=100
Child 1 (PID=15674) started
Child 2 (PID=15675) started
Child 3 (PID=15676) started
Child 4 (PID=15677) started
ERROR [EAGAIN Resource temporarily unavailable] fork
```

在这个例子中，程序只成功创建了 4 个新进程，因为该用户已经有 26 个进程在运行。

示例 36-2. 显示进程资源限制

```
`procres/print_rlimit.c`
#include <sys/resource.h>
#include "print_rlimit.h"           /* Declares function defined here */
#include "tlpi_hdr.h"

int                     /* Print 'msg' followed by limits for 'resource' */
printRlimit(const char *msg, int resource)
{
    struct rlimit rlim;

    if (getrlimit(resource, &rlim) == -1)
        return -1;

    printf("%s soft=", msg);
    if (rlim.rlim_cur == RLIM_INFINITY)
        printf("infinite");
#ifdef RLIM_SAVED_CUR               /* Not defined on some implementations */
    else if (rlim.rlim_cur == RLIM_SAVED_CUR)
        printf("unrepresentable");
#endif
    else
        printf("%lld", (long long) rlim.rlim_cur);

    printf("; hard=");
    if (rlim.rlim_max == RLIM_INFINITY)
        printf("infinite\n");
#ifdef RLIM_SAVED_MAX               /* Not defined on some implementations */
    else if (rlim.rlim_max == RLIM_SAVED_MAX)
        printf("unrepresentable");
#endif
    else
        printf("%lld\n", (long long) rlim.rlim_max);

    return 0;
}
     `procres/print_rlimit.c`
```

示例 36-3. 设置`RLIMIT_NPROC`资源限制

```
`procres/rlimit_nproc.c`
#include <sys/resource.h>
#include "print_rlimit.h"               /* Declaration of printRlimit() */
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    struct rlimit rl;
    int j;
    pid_t childPid;

    if (argc < 2 || argc > 3 || strcmp(argv[1], "--help") == 0)
        usageErr("%s soft-limit [hard-limit]\n", argv[0]);

    printRlimit("Initial maximum process limits: ", RLIMIT_NPROC);

    /* Set new process limits (hard == soft if not specified) */

    rl.rlim_cur = (argv[1][0] == 'i') ? RLIM_INFINITY :
                                getInt(argv[1], 0, "soft-limit");
    rl.rlim_max = (argc == 2) ? rl.rlim_cur :
                (argv[2][0] == 'i') ? RLIM_INFINITY :
                                getInt(argv[2], 0, "hard-limit");
    if (setrlimit(RLIMIT_NPROC, &rl) == -1)
        errExit("setrlimit");

    printRlimit("New maximum process limits:     ", RLIMIT_NPROC);

    /* Create as many children as possible */

    for (j = 1; ; j++) {
        switch (childPid = fork()) {
        case -1: errExit("fork");

        case 0: _exit(EXIT_SUCCESS);            /* Child */

        default:        /* Parent: display message about each new child
                           and let the resulting zombies accumulate */
            printf("Child %d (PID=%ld) started\n", j, (long) childPid);
            break;
        }
    }
}
     `procres/rlimit_nproc.c`
```

#### 无法表示的限制值

在一些编程环境中，*rlim_t*数据类型可能无法表示某个特定资源限制的完整值范围。比如在提供多个编程环境的系统中，*rlim_t*数据类型的大小可能有所不同。这种情况可能发生在系统中添加了一个大文件编译环境（即设置`_FILE_OFFSET_BITS`特性测试宏为 64，如大型文件的 I/O 所述），而系统中的*off_t*传统上是 32 位的（在每个环境中，*rlim_t*的大小将与*off_t*相同）。这会导致一种情况：一个小型*rlim_t*的程序在被一个 64 位*off_t*的程序执行后，可能会继承一个超过最大*rlim_t*值的资源限制（例如，文件大小限制）。

为了帮助可移植的应用程序处理资源限制无法表示的情况，SUSv3 规定了两个常量来表示无法表示的限制值：`RLIM_SAVED_CUR`和`RLIM_SAVED_MAX`。如果软资源限制无法在*rlim_t*中表示，那么*getrlimit()*会在*rlim_cur*字段中返回`RLIM_SAVED_CUR`。`RLIM_SAVED_MAX`在*rlim_max*字段中对无法表示的硬限制执行类似的功能。

如果所有可能的资源限制值都能在*rlim_t*中表示，那么 SUSv3 允许实现将`RLIM_SAVED_CUR`和`RLIM_SAVED_MAX`定义为与`RLIM_INFINITY`相同。这就是 Linux 上如何定义这些常量的方式，意味着所有可能的资源限制值都可以在*rlim_t*中表示。然而，在像 x86-32 这样的 32 位架构上并非如此。在这些架构中，在一个大文件编译环境中（即如大型文件的 I/O 中所述，将`_FILE_OFFSET_BITS`特性测试宏设置为 64），*glibc*对*rlim_t*的定义是 64 位的，但表示资源限制的内核数据类型是*unsigned long*，只有 32 位宽。当前版本的*glibc*对此问题的处理方式如下：如果一个使用`_FILE_OFFSET_BITS=64`编译的程序尝试设置一个大于 32 位*unsigned long*能表示的资源限制值，*glibc*包装函数*setrlimit()*会默默地将该值转换为`RLIM_INFINITY`。换句话说，请求的资源限制设置不会生效。

### 注意

因为在许多 x86-32 发行版中处理文件的工具通常使用`_FILE_OFFSET_BITS=64`进行编译，所以无法处理超过 32 位能表示的资源限制这一问题，不仅会影响到应用程序开发者，也会影响最终用户。

有人可能会争辩，*glibc setrlimit()* 包装器应该在请求的资源限制超过 32 位 *unsigned long* 的容量时给出错误。然而，根本问题是内核的限制，主文中描述的行为是 *glibc* 开发者处理该问题的方式。

## 特定资源限制的详细信息

在本节中，我们提供了有关 Linux 上每个资源限制的详细信息，并指出那些特定于 Linux 的限制。

#### `RLIMIT_AS`

`RLIMIT_AS` 限制指定了进程虚拟内存（地址空间）的最大大小，以字节为单位。尝试（*brk()*, *sbrk()*, *mmap()*, *mremap()*, 和 *shmat()*）超过此限制会失败，并返回 `ENOMEM` 错误。在实践中，程序最常遇到此限制的地方是在调用 *malloc* 包中的函数时，这些函数使用了 *sbrk()* 和 *mmap()*。遇到此限制时，栈的增长也可能失败，后果如下所述的 `RLIMIT_STACK`。

#### `RLIMIT_CORE`

`RLIMIT_CORE` 限制指定了在进程因某些信号终止时生成的核心转储文件的最大大小，以字节为单位（核心转储文件）。在达到此限制时，将停止生成核心转储文件。指定 0 限制会阻止创建核心转储文件，这在某些情况下很有用，因为核心转储文件可能非常大，而且最终用户通常不知道如何处理它们。禁用核心转储的另一个原因是出于安全考虑——防止程序内存的内容被转储到磁盘。如果 `RLIMIT_FSIZE` 限制低于此限制，则核心转储文件的大小将被限制为 `RLIMIT_FSIZE` 字节。

#### `RLIMIT_CPU`

`RLIMIT_CPU` 限制指定了进程可以使用的最大 CPU 时间（系统模式和用户模式下的时间，以秒为单位）。SUSv3 要求在软限制达到时发送 `SIGXCPU` 信号，但没有指定其他细节。（`SIGXCPU` 的默认操作是终止进程并生成核心转储。）可以为 `SIGXCPU` 设置一个处理程序，执行所需的任何处理后再返回控制权给主程序。此后，（在 Linux 上）每消耗一秒的 CPU 时间，都会发送一次 `SIGXCPU` 信号。如果进程继续执行，直到达到硬 CPU 限制，则内核会发送 `SIGKILL` 信号，这总是会终止进程。

UNIX 实现对于如何处理在处理 `SIGXCPU` 信号后仍继续消耗 CPU 时间的进程在细节上有所不同。大多数实现会继续定期发送 `SIGXCPU` 信号。如果目标是信号的可移植使用，我们应该编写应用程序，使其在第一次收到该信号时，执行所需的清理操作并终止。（或者，程序也可以在收到信号后更改资源限制。）

#### `RLIMIT_DATA`

`RLIMIT_DATA`限制指定进程数据段的最大大小，以字节为单位（包括已初始化数据、未初始化数据和堆段，详见进程内存布局）。尝试通过`sbrk()`和`brk()`扩展数据段（程序断点）超出该限制时，会失败并返回错误`ENOMEM`。与`RLIMIT_AS`类似，程序遇到此限制最常见的地方是在调用*malloc*包中的函数时。

#### `RLIMIT_FSIZE`

`RLIMIT_FSIZE`限制指定进程可以创建的文件的最大大小，以字节为单位。如果进程尝试将文件扩展到超出软限制的大小，它会收到`SIGXFSZ`信号，并且系统调用（例如，*write()* 或 *truncate()*) 将因错误`EFBIG`而失败。`SIGXFSZ`的默认动作是终止进程并生成核心转储。也可以捕获此信号，并将控制权返回给主程序。然而，任何进一步尝试扩展文件的操作都会触发相同的信号和错误。

#### `RLIMIT_MEMLOCK`

`RLIMIT_MEMLOCK`限制（源自 BSD；SUSv3 中没有，仅在 Linux 和 BSD 系统上可用）指定进程可以锁定到物理内存中的虚拟内存的最大字节数，以防止内存被换出。此限制影响*mlock()*和*mlockall()*系统调用，以及*mmap()*和*shmctl()*系统调用的锁定选项。我们在第 50.2 节中描述了详细信息。

如果在调用*mlockall()*时指定了`MCL_FUTURE`标志，则`RLIMIT_MEMLOCK`限制还可能导致之后调用*brk()*, *sbrk()*, *mmap()*或*mremap()*失败。

#### `RLIMIT_MSGQUEUE`

`RLIMIT_MSGQUEUE`限制（Linux 特有；自 Linux 2.6.8 开始）指定可以为调用进程的真实用户 ID 分配的 POSIX 消息队列的最大字节数。当使用*mq_open()*创建 POSIX 消息队列时，字节数会根据以下公式从此限制中扣除：

```
bytes = attr.mq_maxmsg * sizeof(struct msg_msg *) +
        attr.mq_maxmsg * attr.mq_msgsize;
```

在这个公式中，*attr*是作为第四个参数传递给*mq_open()*的*mq_attr*结构体。包含*sizeof(struct msg_msg *)*的加数确保用户不能排队无限数量的零长度消息。（*msg_msg*结构是内核内部使用的数据类型。）这是必要的，因为尽管零长度消息不包含任何数据，但它们确实会消耗一些系统内存用于账本管理开销。

`RLIMIT_MSGQUEUE`限制仅影响调用进程。其他属于该用户的进程不会受到影响，除非它们也设置此限制或继承了此限制。

#### `RLIMIT_NICE`

`RLIMIT_NICE`限制（特定于 Linux，适用于 Linux 2.6.12 及以后版本）指定可以使用*sched_setscheduler()*和*nice()*为进程设置的优先级值的上限。上限计算方式为*20 – rlim_cur*，其中*rlim_cur*是当前`RLIMIT_NICE`软资源限制的值。有关更多详细信息，请参阅进程优先级（Nice 值）。

#### `RLIMIT_NOFILE`

`RLIMIT_NOFILE`限制指定一个数字，比进程可以分配的最大文件描述符号大 1。尝试（例如，*open()*, *pipe()*, *socket()*, *accept()*, *shm_open()*, *dup()*, *dup2()*, *fcntl(F_DUPFD)*, 和 *epoll_create()*)分配超过此限制的描述符将失败。在大多数情况下，错误是`EMFILE`，但对于*dup2(fd, newfd)*，错误是`EBADF`；对于*fcntl(fd, F_DUPFD, newfd)*，如果*newfd*大于或等于限制，错误是`EINVAL`。

对`RLIMIT_NOFILE`限制的更改会反映在*sysconf(_SC_OPEN_MAX)*返回的值中。SUSv3 允许但不要求实现返回在更改`RLIMIT_NOFILE`限制前后对*sysconf(_SC_OPEN_MAX)*调用的不同值；其他实现可能在这一点上与 Linux 的行为不同。

### 注意

SUSv3 规定，如果应用程序将软限制或硬限制`RLIMIT_NOFILE`设置为小于或等于当前进程已打开的最高文件描述符的值，则可能会出现意外行为。

在 Linux 中，我们可以使用*readdir()*扫描`/proc/`*PID*`/fd`目录的内容，检查进程当前打开了哪些文件描述符，该目录包含每个当前已打开文件描述符的符号链接。

内核对`RLIMIT_NOFILE`限制可以提高的值施加了上限。在 2.6.25 之前的内核中，这个上限是由内核常量`NR_OPEN`定义的硬编码值，其值为 1,048,576（要提高此上限，需要重新构建内核）。自 2.6.25 内核以来，该限制由 Linux 特有的`/proc/sys/fs/nr_open`文件中的值定义。该文件中的默认值是 1,048,576；超级用户可以修改此值。尝试将软限制或硬限制`RLIMIT_NOFILE`设置为高于此上限的值将返回错误`EPERM`。

系统还对所有进程可以打开的文件总数设置了系统范围的限制。可以通过 Linux 特有的 `/proc/sys/fs/file-max` 文件来检索和修改此限制。（参见 文件描述符与打开文件的关系，我们可以更精确地将 `file-max` 定义为系统范围内的打开文件描述符数量限制。）只有具有特权的 (`CAP_SYS_ADMIN`) 进程才能超出 `file-max` 限制。在非特权进程中，遇到 `file-max` 限制的系统调用会失败，并显示错误 `ENFILE`。

#### `RLIMIT_NPROC`

`RLIMIT_NPROC` 限制（源自 BSD；在 SUSv3 中不存在，仅在 Linux 和 BSD 中可用）指定调用进程的实际用户 ID 可以创建的最大进程数。超出此限制的尝试（*fork()*、*vfork()* 和 *clone()*) 会失败，并显示错误 `EAGAIN`。

`RLIMIT_NPROC` 限制仅影响调用进程。除非其他进程也设置或继承了此限制，否则属于该用户的其他进程不受此限制的影响。对于特权进程（`CAP_SYS_ADMIN` 或 `CAP_SYS_RESOURCE`），此限制不被强制执行。

### 注意

Linux 还对所有用户可以创建的进程数量设置了系统范围的限制。在 Linux 2.4 及以后版本中，可以使用 Linux 特有的 `/proc/sys/kernel/threads-max` 文件来检索和修改此限制。

精确地说，`RLIMIT_NPROC` 资源限制和 `threads-max` 文件实际上是对可以创建的线程数的限制，而不是进程数的限制。

设置 `RLIMIT_NPROC` 资源限制默认值的方式在不同的内核版本中有所不同。在 Linux 2.2 中，它是根据固定公式计算的。在 Linux 2.4 及以后版本中，它是根据可用的物理内存量使用公式计算的。

### 注意

SUSv3 并未指定 `RLIMIT_NPROC` 资源限制。SUSv3 规定的获取（但不更改）允许用户 ID 最大进程数的方法是通过调用 *sysconf(_SC_CHILD_MAX)*。此 *sysconf()* 调用在 Linux 上受支持，但在 2.6.23 之前的内核版本中，该调用并未返回准确的信息——它始终返回值 999。自 Linux 2.6.23（以及 *glibc* 2.4 及更高版本）起，该调用会正确报告限制（通过检查 `RLIMIT_NPROC` 资源限制的值）。

没有便捷的方式来发现特定用户 ID 已经创建了多少个进程。在 Linux 中，我们可以尝试扫描系统上所有 `/proc/`*PID*`/status` 文件，并检查 `Uid` 条目下的信息（该条目列出了四个进程用户 ID，顺序为：实际、有效、保存集和文件系统）来估算当前属于某个用户的进程数。然而要注意的是，在我们完成这样的扫描时，这些信息可能已经发生变化。

#### `RLIMIT_RSS`

`RLIMIT_RSS` 限制（源自 BSD；在 SUSv3 中缺失，但广泛可用）指定了进程常驻集中的最大页面数；即当前在物理内存中的虚拟内存页面总数。此限制在 Linux 上可用，但目前没有任何效果。

### 注意

在较早的 Linux 2.4 内核版本（包括 2.4.29）中，`RLIMIT_RSS` 确实会影响 *madvise()* `MADV_WILLNEED` 操作的行为（建议未来内存使用模式：*madvise()*")）。如果由于遇到 `RLIMIT_RSS` 限制而无法执行此操作，则会在 *errno* 中返回错误 `EIO`。

#### `RLIMIT_RTPRIO`

`RLIMIT_RTPRIO` 限制（Linux 特有；自 Linux 2.6.12 起）指定了可以通过 *sched_setscheduler()* 和 *sched_setparam()* 为此进程设置的实时优先级的上限。有关详细信息，请参阅 修改和检索策略与优先级。

#### `RLIMIT_RTTIME`

`RLIMIT_RTTIME` 限制（Linux 特有；自 Linux 2.6.25 起）指定了在实时调度策略下，进程在不休眠（即不执行阻塞系统调用）的情况下，可以消耗的最大 CPU 时间（单位：微秒）。如果达到此限制，行为与 `RLIMIT_CPU` 相同：如果进程达到了软限制，则会向进程发送 `SIGXCPU` 信号，并且每消耗一秒额外的 CPU 时间，就会发送进一步的 `SIGXCPU` 信号。达到硬限制时，会发送 `SIGKILL` 信号。有关详细信息，请参阅 修改和检索策略与优先级。

#### `RLIMIT_SIGPENDING`

`RLIMIT_SIGPENDING` 限制（Linux 特有；自 Linux 2.6.8 起）指定了可以为调用进程的真实用户 ID 排队的最大信号数。超过此限制的尝试（*sigqueue()*）将失败，并返回错误 `EAGAIN`。

`RLIMIT_SIGPENDING` 限制只影响调用进程。除非其他进程也设置或继承了此限制，否则不会受到影响。

在最初的实现中，`RLIMIT_SIGPENDING` 限制的默认值为 1024。自内核 2.6.12 起，默认值已更改为与 `RLIMIT_NPROC` 的默认值相同。

为了检查 `RLIMIT_SIGPENDING` 限制，排队信号的计数包括实时信号和标准信号。（标准信号只能排队一次。）然而，这个限制仅在 *sigqueue()* 中执行。即使该限制指定的信号数量已经排队到属于该真实用户 ID 的进程，也仍然可以使用 *kill()* 排队每个尚未排队的信号（包括实时信号）。

从内核 2.6.12 起，Linux 特有的 `/proc/`*PID*`/status` 文件中的 `SigQ` 字段显示进程真实用户 ID 的当前和最大排队信号数量。

#### `RLIMIT_STACK`

`RLIMIT_STACK` 限制指定进程栈的最大大小，以字节为单位。尝试将栈增长超过此限制会导致为该进程生成 `SIGSEGV` 信号。由于栈已耗尽，捕获此信号的唯一方法是建立一个替代信号栈，具体方法见第 21.3 节。

### 注意

从 Linux 2.6.23 开始，`RLIMIT_STACK` 限制还决定了用于存储进程命令行参数和环境变量的空间大小。详情请参见 *execve(2)* 手册页。

## 摘要

进程会消耗各种系统资源。*getrusage()* 系统调用允许进程监控自身及其子进程消耗的某些资源。

*setrlimit()* 和 *getrlimit()* 系统调用允许进程设置和检索其对各种资源的消耗限制。每个资源限制有两个组成部分：软限制，它是内核在检查进程资源消耗时执行的限制，和硬限制，它作为软限制的上限。非特权进程可以将资源的软限制设置为从 0 到硬限制的任何值，但只能降低硬限制。特权进程可以对任一限制值进行更改，只要软限制小于或等于硬限制。如果进程遇到软限制，通常会通过接收到信号或系统调用失败来告知进程该限制已经被触及。

## 练习

1.  编写一个程序，演示 *getrusage()* `RUSAGE_CHILDREN` 标志仅检索已执行 *wait()* 调用的子进程的信息。（让程序创建一个消耗一些 CPU 时间的子进程，然后让父进程在调用 *wait()* 前后调用 *getrusage()*。）

1.  编写一个程序，执行一个命令并显示其资源使用情况。这类似于 *time(1)* 命令的功能。因此，我们可以如下使用这个程序：

    ```
    $ `./rusage` ``*`command arg...`*``
    ```

1.  编写程序，确定当一个进程消耗的某些资源已经超过通过 *setrlimit()* 调用指定的软限制时会发生什么。
