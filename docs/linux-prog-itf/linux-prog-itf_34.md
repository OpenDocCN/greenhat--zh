## 第三十四章：进程组、会话和作业控制

进程组和会话在进程之间形成了一个二级层次结构关系：进程组是相关进程的集合，会话是相关进程组的集合。每种情况下“相关”的含义将在本章中得到明确说明。

进程组和会话是为支持 Shell 作业控制而定义的抽象概念，它允许交互式用户在前台或后台运行命令。术语*作业*通常与*进程组*一词同义使用。

本章描述了进程组、会话和作业控制。

## 概述

*进程组*是一个或多个进程共享相同*进程组标识符*（PGID）的集合。进程组 ID 是与进程 ID 类型相同的数字（*pid_t*）。每个进程组都有一个*进程组领导进程*，即创建该组的进程，其进程 ID 成为该进程组的进程组 ID。新进程继承其父进程的进程组 ID。

进程组有一个*生命周期*，即从领导进程创建该组开始，到最后一个成员进程离开该组为止的时间段。进程可以通过终止或加入另一个进程组来离开当前的进程组。进程组的领导进程不一定是进程组中的最后一个成员。

*会话*是多个进程组的集合。进程的会话成员身份由其*会话标识符*（SID）决定，SID 和进程组 ID 一样，都是*pid_t* 类型的数字。*会话领导进程*是创建新会话的进程，其进程 ID 成为会话 ID。新进程继承其父进程的会话 ID。

会话中的所有进程共享一个*控制终端*。控制终端是在会话领导进程首次打开终端设备时建立的。一个终端最多只能是一个会话的控制终端。

在任何时候，会话中的一个进程组是终端的*前台进程组*，其他进程组则是*后台进程组*。只有前台进程组中的进程可以从控制终端读取输入。当用户在控制终端输入某个信号生成字符时，信号会发送给前台进程组中的所有成员。这些字符包括*中断*字符（通常是*Control-C*，生成`SIGINT`信号）；*退出*字符（通常是*Control-\*，生成`SIGQUIT`信号）；和*挂起*字符（通常是*Control-Z*，生成`SIGTSTP`信号）。

作为与控制终端建立连接（即打开终端）的结果，会话领导进程成为该终端的*控制进程*。成为控制进程的主要意义在于，如果发生终端断开，内核会向该进程发送`SIGHUP`信号。

### 注意

通过检查 Linux 特定的 `/proc/`*PID*`/stat` 文件，我们可以确定任何进程的进程组 ID 和会话 ID。我们还可以确定进程控制终端的设备 ID（以包含主次设备 ID 的单一十进制整数表示）以及该终端的控制进程的进程 ID。有关更多详细信息，请参阅*proc(5)*手册页。

会话和进程组的主要用途是用于 shell 作业控制。通过查看来自该领域的具体示例，有助于澄清这些概念。对于交互式登录，控制终端是用户登录的终端。登录 shell 成为会话的领导者和该终端的控制进程，并且也成为其进程组的唯一成员。每个从 shell 启动的命令或命令管道都会创建一个或多个进程，shell 将这些进程放入一个新的进程组中。（这些进程最初是该进程组的唯一成员，尽管它们创建的任何子进程也会成为该组的成员。）如果一个命令或管道以一个与符号（`&`）结束，它将作为一个后台进程组创建。否则，它将成为前台进程组。所有在登录会话期间创建的进程都属于同一个会话。

### 注意

在图形环境中，控制终端是一个伪终端，每个终端窗口都有一个独立的会话，窗口的启动 shell 是会话的领导者和该终端的控制进程。

进程组偶尔在作业控制以外的领域也有应用，因为它们具有两个有用的属性：父进程可以在特定进程组中的任何子进程上等待（*waitpid()* 系统调用 系统调用")），并且可以向进程组的所有成员发送信号（发送信号：*kill()*")）。

图 34-1 展示了由执行以下命令所产生的各个进程之间的进程组和会话关系：

```
$ `echo $$`                             *Display the PID of the shell*
400
$ `find / 2> /dev/null | wc -l &`       *Creates 2 processes in background group*
[1] 659
$ `sort < longlist | uniq -c`           *Creates 2 processes in foreground group*
```

此时，shell（*bash*）、*find*、*wc*、*sort* 和 *uniq* 都在运行。

![进程组、会话和控制终端之间的关系](img/34-1_PGSJC-overview.png.jpg)图 34-1. 进程组、会话和控制终端之间的关系

## 进程组

每个进程都有一个数字型的进程组 ID，用于定义它所属的进程组。一个新进程会继承其父进程的进程组 ID。一个进程可以使用*getpgrp()*来获取其进程组 ID。

```
#include <unistd.h>

pid_t `getpgrp`(void)
```

### 注意

始终成功返回调用进程的进程组 ID

如果 *getpgrp()* 返回的值与调用进程的进程 ID 匹配，则该进程是其进程组的领导者。

*setpgid()* 系统调用将进程 ID 为 *pid* 的进程的进程组更改为 *pgid* 中指定的值。

```
#include <unistd.h>

int `setpgid`(pid_t *pid*, pid_t *pgid*);
```

### 注意

成功时返回 0，出错时返回 -1

如果 *pid* 被指定为 0，则调用进程的进程组 ID 会被更改。如果 *pgid* 被指定为 0，则由 *pid* 指定的进程的进程组 ID 会与其进程 ID 相同。因此，以下 *setpgid()* 调用是等效的：

```
setpgid(0, 0);
setpgid(getpid(), 0);
setpgid(getpid(), getpid());
```

如果 *pid* 和 *pgid* 参数指定相同的进程（即 *pgid* 为 0 或与由 *pid* 指定的进程的进程 ID 匹配），则会创建一个新的进程组，并将指定的进程设为新组的领导者（即进程的进程组 ID 会与其进程 ID 相同）。如果这两个参数指定不同的值（即 *pgid* 不为 0 且与由 *pid* 指定的进程的进程 ID 不匹配），则 *setpgid()* 用于将进程从一个进程组移动到另一个进程组。

*setpgid()*（以及在 会话 中描述的 *setsid()*）的典型调用者是像 shell 和 *login(1)* 这样的程序。在 创建守护进程 中，我们将看到一个程序也会在成为守护进程的过程中调用 *setsid()*。

调用 *setpgid()* 时有若干限制：

+   *pid* 参数只能指定调用进程或其子进程之一。违反此规则会导致错误 `ESRCH`。

+   当将进程在进程组之间移动时，调用进程和 *pid* 所指定的进程（可能是同一个进程），以及目标进程组，必须都属于同一会话。违反此规则会导致错误 `EPERM`。

+   *pid* 参数不能指定会话领导进程。违反此规则会导致错误 `EPERM`。

+   在子进程执行 *exec()* 后，进程不能更改该子进程的进程组 ID。违反此规则会导致错误 `EACCES`。此约束的理由是，如果在程序开始执行后更改其进程组 ID，可能会混淆该程序。

#### 在作业控制 shell 中使用 *setpgid()*。

进程在其子进程执行 *exec()* 后不能更改该子进程的进程组 ID，这一限制影响了作业控制 shell 的编程，作业控制 shell 有以下要求：

+   作业中的所有进程（即一个命令或一个管道）必须放入一个单一的进程组中。（我们可以通过查看图 34-1 中*bash*创建的两个进程组，来看到期望的结果。）这一步骤允许 shell 使用*killpg()*（或等效的，用负的*pid*参数调用*kill()*）同时向进程组中的所有成员发送作业控制信号。当然，这一步必须在发送任何作业控制信号之前完成。

+   每个子进程必须在执行程序之前被转移到进程组中，因为程序本身对进程组 ID 的操作是无感知的。

对于作业中的每个进程，父进程或子进程都可以使用*setpgid()*来更改子进程的进程组 ID。然而，由于在*fork()*之后父子进程的调度是不可预测的（*fork()*后的竞争条件后的竞争条件")），我们不能依赖父进程在子进程执行*exec()*之前更改子进程的进程组 ID；也不能依赖子进程在父进程尝试发送任何作业控制信号之前更改其进程组 ID。（依赖这两种行为中的任何一种都会导致竞争条件。）因此，作业控制 shell 的编程方式是，父进程和子进程都在*fork()*之后立即调用*setpgid()*将子进程的进程组 ID 更改为相同的值，并且父进程会忽略*setpgid()*调用时出现的`EACCES`错误。换句话说，在作业控制 shell 中，我们会找到类似示例 34-1 所示的代码。

示例 34-1. 如何通过作业控制 shell 设置子进程的进程组 ID

```
pid_t childPid;
    pid_t pipelinePgid;         /* PGID to which processes in a pipeline
                                    are to be assigned */
    /* Other code */

    childPid = fork();
    switch (childPid) {
    case -1: /* fork() failed */
        /* Handle error */

    case 0: /* Child */
        if (setpgid(0, pipelinePgid) == -1)
            /* Handle error */
        /* Child carries on to exec the required program */

    default: /* Parent (shell) */
        if (setpgid(childPid, pipelinePgid) == -1 && errno != EACCES)
            /* Handle error */
        /* Parent carries on to do other things */
    }
```

情况比在示例 34-1 中展示的要复杂一些，因为在为管道创建进程时，父 shell 会记录管道中第一个进程的进程 ID，并将其作为该进程组中所有进程的进程组 ID（*pipelinePgid*）。

#### 其他（过时的）用于检索和修改进程组 ID 的接口

*getpgrp()*和*setpgid()*系统调用名称中不同的后缀值得解释。

起初，4.2BSD 提供了一个 *getprgp(pid)* 系统调用，返回由 *pid* 指定的进程的进程组 ID。在实践中，*pid* 总是用来指定调用进程。因此，POSIX 委员会认为这个调用比必要的更复杂，改为采用了不需要参数的 System V *getpgrp()* 调用，返回调用进程的进程组 ID。

为了更改进程组 ID，4.2BSD 提供了 *setpgrp(pid, pgid)* 调用，该调用的操作方式与 *setpgid()* 类似。主要区别在于 BSD 的 *setpgrp()* 可以将进程组 ID 设置为任何值。（我们之前提到过 *setpgid()* 不能将进程转移到不同会话中的进程组。）这导致了一些安全问题，并且在实现作业控制时比必要的更加灵活。因此，POSIX 委员会选择了一个更具限制性的函数，并命名为 *setpgid()*。

为了使问题更加复杂，SUSv3 规定了 *getpgid(pid)*，其语义与旧的 BSD *getpgrp()* 相同，并且还弱地规定了一个替代的 System V 衍生版 *setpgrp()*，不接受任何参数，大致相当于 *setpgid(0, 0)*。

尽管我们之前描述的 *setpgid()* 和 *getpgrp()* 系统调用足以实现 shell 作业控制，但 Linux 和大多数其他 UNIX 实现也提供了 *getpgid(pid)* 和 *setpgrp(void)*。为了向后兼容，许多 BSD 衍生实现继续提供 *setprgp(pid, pgid)* 作为 *setpgid(pid, pgid)* 的同义词。

如果我们在编译程序时显式定义了 `_BSD_SOURCE` 特性测试宏，那么 *glibc* 会提供基于 BSD 的 *setpgrp()* 和 *getpgrp()* 版本，而不是默认版本。

## 会话

会话是进程组的集合。进程的会话成员身份由其数字会话 ID 定义。一个新进程继承其父进程的会话 ID。*getsid()* 系统调用返回由 *pid* 指定的进程的会话 ID。

```
#define _XOPEN_SOURCE 500
#include <unistd.h>

pid_t `getsid`(pid_t *pid*);
```

### 注意

返回指定进程的会话 ID，或在错误时返回 *(pid_t)* -1

如果 *pid* 被指定为 0，*getsid()* 将返回调用进程的会话 ID。

### 注意

在一些 UNIX 实现（例如 HP-UX 11）中，*getsid()* 只能在进程与调用进程属于同一会话时才能用来检索进程的会话 ID。（SUSv3 允许这种可能性。）换句话说，这个调用仅通过其成功或失败（`EPERM`）来告知我们指定的进程是否与调用者处于同一会话。这种限制不适用于 Linux 或大多数其他实现。

如果调用进程不是进程组的领导者，*setsid()* 将创建一个新的会话。

```
#include <unistd.h>

pid_t `setsid`(void);
```

### 注意

返回新会话的会话 ID，或在错误时返回 *(pid_t)* -1

*setsid()* 系统调用创建一个新会话，具体如下：

+   调用进程成为新会话的领导者，并成为该会话内新进程组的领导者。调用进程的进程组 ID 和会话 ID 被设置为与其进程 ID 相同的值。

+   调用进程没有控制终端。任何之前存在的与控制终端的连接都会被断开。

如果调用进程是进程组领导者，*setsid()*将失败并返回错误`EPERM`。确保不发生这种情况的最简单方法是执行*fork()*并让父进程退出，而子进程继续调用*setsid()*。由于子进程继承了父进程的进程组 ID 并获得了自己的唯一进程 ID，因此它不能成为进程组领导者。

对进程组领导者不能调用*setsid()*的限制是必要的，因为如果没有此限制，进程组领导者将能够将自己置于另一个（新的）会话中，而进程组中的其他成员仍然保留在原始会话中。（不会创建新的进程组，因为按定义，进程组领导者的进程组 ID 已经与其进程 ID 相同。）这会违反严格的两级会话和进程组层次结构，其中进程组的所有成员必须属于同一个会话。

### 注意

当通过*fork()*创建新进程时，内核不仅确保它具有唯一的进程 ID，而且确保进程 ID 与任何现有进程的进程组 ID 或会话 ID 不匹配。因此，即使进程组或会话的领导者已经退出，新进程也不能重用领导者的进程 ID，从而意外成为现有会话或进程组的领导者。

示例 34-2 演示了如何使用*setsid()*来创建一个新会话。为了检查它不再拥有控制终端，程序尝试打开特殊文件`/dev/tty`（将在下一节描述）。当我们运行此程序时，会看到以下内容：

```
$ `ps -p $$ -o 'pid pgid sid command'`            *$$ is PID of shell*
  PID  PGID   SID COMMAND
12243 12243 12243 bash                          *PID, PGID, and SID of shell*
$ `./t_setsid`
$ PID=12352, PGID=12352, SID=12352
ERROR [ENXIO Device not configured] open /dev/tty
```

从输出中可以看出，进程成功地将自己放置在一个新的进程组中，并且该进程组属于一个新会话。由于此会话没有控制终端，*open()*调用失败。（在上面程序输出的倒数第二行中，我们看到一个与程序输出混合的 shell 提示符，因为 shell 注意到在*fork()*调用后父进程已经退出，因此在子进程完成之前打印出下一个提示符。）

示例 34-2. 创建新会话

```
`pgsjc/t_setsid.c`
#define _XOPEN_SOURCE 500
#include <unistd.h>
#include <fcntl.h>
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    if (fork() != 0)            /* Exit if parent, or on error */
        _exit(EXIT_SUCCESS);

    if (setsid() == -1)
        errExit("setsid");

    printf("PID=%ld, PGID=%ld, SID=%ld\n", (long) getpid(),
            (long) getpgrp(), (long) getsid(0));

    if (open("/dev/tty", O_RDWR) == -1)
        errExit("open /dev/tty");
    exit(EXIT_SUCCESS);
}
     `pgsjc/t_setsid.c`
```

## 控制终端和控制进程

会话中的所有进程可能有一个（单一的）控制终端。创建时，会话没有控制终端；控制终端是在会话领导者首次打开一个未作为任何会话的控制终端的终端时建立的，除非在调用*open()*时指定了`O_NOCTTY`标志。一个终端最多只能是一个会话的控制终端。

### 注意

SUSv3 指定了函数 *tcgetsid(int fd)*（在 `<termios.h>` 中声明），该函数返回与指定 *fd* 的控制终端关联的会话 ID。这个函数在 *glibc* 中提供（其通过 *ioctl()* 的 `TIOCGSID` 操作实现）。

控制终端是由 *fork()* 的子进程继承的，并且在 *exec()* 之后保持不变。

当会话领导者打开控制终端时，它同时也成为该终端的控制进程。如果随后发生终端断开连接，内核会向控制进程发送 `SIGHUP` 信号，通知它这一事件。我们在 `SIGHUP` 和控制进程终止 中会进一步详细说明这一点。

如果一个进程有控制终端，打开特殊文件 `/dev/tty` 会获取该终端的文件描述符。如果标准输入和输出被重定向，并且程序希望确保与控制终端通信，这是非常有用的。例如，密码加密与用户认证 中描述的 *getpass()* 函数就是为此目的打开 `/dev/tty`。如果进程没有控制终端，打开 `/dev/tty` 会因为错误 `ENXIO` 失败。

#### 移除进程与控制终端的关联

*ioctl(fd, TIOCNOTTY)* 操作可以用来移除进程与其控制终端的关联，控制终端通过文件描述符 *fd* 指定。调用此操作后，尝试打开 `/dev/tty` 将会失败。（虽然在 SUSv3 中没有明确规定，但 `TIOCNOTTY` 操作在大多数 UNIX 实现中都被支持。）

如果调用进程是终端的控制进程，那么与控制进程终止时的处理方式类似（参见 `SIGHUP` 和控制进程终止），以下步骤将发生：

1.  会话中的所有进程都会失去与控制终端的关联。

1.  控制终端失去与会话的关联，因此可以被另一个会话领导者作为控制进程获取。

1.  内核向前台进程组的所有成员发送 `SIGHUP` 信号（以及 `SIGCONT` 信号），以通知它们控制终端丢失。

#### 在 BSD 上建立控制终端

SUSv3 并没有明确指定会话如何获取控制终端，只是声明在打开终端时指定 `O_NOCTTY` 标志可以保证该终端不会成为会话的控制终端。我们在上面描述的 Linux 语义源自 System V。

在 BSD 系统中，打开终端时，即使指定了 `O_NOCTTY` 标志，终端也不会成为控制终端。相反，会话领导进程使用 *ioctl()* 的 `TIOCSCTTY` 操作显式地将文件描述符 *fd* 所指向的终端设为控制终端：

```
if (ioctl(fd, TIOCSCTTY) == -1)
    errExit("ioctl");
```

只有当会话还没有控制终端时，才能执行此操作。

`TIOCSCTTY` 操作在 Linux 上也可用，但在其他（非 BSD）实现中并不普遍。

#### 获取指向控制终端的路径名：*ctermid()*

*ctermid()* 函数返回一个路径名，指向控制终端。

```
#include <stdio.h>            /* Defines L_ctermid constant */

char *`ctermid`(char **ttyname*);
```

### 注意

返回指向包含控制终端路径名的字符串的指针，若路径名无法确定，则返回 `NULL`

*ctermid()* 函数通过两种方式返回控制终端的路径名：通过函数结果和通过 *ttyname* 指向的缓冲区。

如果 *ttyname* 不是 `NULL`，则它应为至少 `L_ctermid` 字节的缓冲区，路径名将被复制到此数组中。在这种情况下，函数的返回值也是指向该缓冲区的指针。如果 *ttyname* 是 `NULL`，*ctermid()* 会返回指向一个静态分配的缓冲区的指针，缓冲区包含路径名。当 *ttyname* 是 `NULL` 时，*ctermid()* 不是可重入的。

在 Linux 和其他 UNIX 实现中，*ctermid()* 通常返回字符串 `/dev/tty`。此函数的目的是简化向非 UNIX 系统的可移植性。

## 前台和后台进程组

控制终端维护前台进程组的概念。在一个会话中，只有一个进程组可以在某一时刻处于前台；该会话中的所有其他进程组为后台进程组。前台进程组是唯一可以自由读取和写入控制终端的进程组。当在控制终端上输入一个信号生成字符时，终端驱动程序会将相应的信号传递给前台进程组的成员。我们将在第 34.7 节中详细描述。

### 注意

理论上，可能出现会话没有前台进程组的情况。例如，如果前台进程组中的所有进程终止，而没有其他进程注意到并将自己移动到前台，就可能发生这种情况。在实践中，这种情况比较少见。通常，shell 是监视前台进程组状态的进程，它会在通过 *wait()* 注意到前台进程组已终止时，将自己重新调回前台。

*tcgetpgrp()* 和 *tcsetpgrp()* 函数分别用于检索和改变终端的进程组。这些函数主要由作业控制 shell 使用。

```
#include <unistd.h>

pid_t `tcgetpgrp`(int *fd*);
```

### 注意

返回终端前台进程组的进程组 ID，出错时返回 -1

```
int `tcsetpgrp`(int *fd*, pid_t *pgid*);
```

### 注意

成功时返回 0，出错时返回 -1

*tcgetpgrp()*函数返回由文件描述符*fd*所引用的终端的前台进程组 ID，该终端必须是调用进程的控制终端。

### 注意

如果此终端没有前台进程组，*tcgetpgrp()*将返回一个大于 1 的值，该值不匹配任何现有的进程组 ID。（这是 SUSv3 所规定的行为。）

*tcsetpgrp()*函数更改终端的前台进程组。如果调用进程有一个控制终端，并且文件描述符*fd*指向该终端，则*tcsetpgrp()*将终端的前台进程组设置为*pgid*中指定的值，该值必须与调用进程会话中的某个进程组 ID 匹配。

*tcgetpgrp()*和*tcsetpgrp()*函数在 SUSv3 中已标准化。在 Linux 中，和许多其他 UNIX 实现一样，这些函数是通过两个非标准化的*ioctl()*操作实现的：`TIOCGPGRP`和`TIOCSPGRP`。

## `SIGHUP`信号

当控制进程失去其终端连接时，内核会发送`SIGHUP`信号通知它这一事实。（还会发送`SIGCONT`信号，以确保如果进程之前因信号被停止，它能够重新启动。）通常，这可能在两种情况下发生：

+   当终端驱动程序检测到“断开连接”，表示调制解调器或终端线路信号丢失时。

+   当工作站上的终端窗口被关闭时。之所以发生这种情况，是因为与终端窗口关联的伪终端的主端口的最后一个打开的文件描述符被关闭了。

`SIGHUP`的默认行为是终止进程。如果控制进程处理或忽略该信号，则进一步尝试从终端读取将返回文件末尾。

### 注意

SUSv3 规定，如果既发生了终端断开连接，又存在某种条件导致*read()*函数返回`EIO`错误，则*read()*是否返回文件末尾或以`EIO`错误失败未作说明。便携式程序必须考虑到这两种可能性。我们将在实现作业控制和孤儿进程组（以及`SIGHUP`信号复审）中查看*read()*可能因`EIO`错误失败的情况。

向控制进程发送`SIGHUP`信号可能会引发一种链式反应，导致`SIGHUP`信号被发送到其他许多进程。这可能通过两种方式发生：

+   控制进程通常是一个 shell。shell 为`SIGHUP`信号建立一个处理程序，以便在终止之前，它可以向每个它创建的作业发送一个`SIGHUP`信号。该信号默认会终止这些作业，但如果这些作业捕获了该信号，则它们会因此得知 shell 的终结。

+   当终端的控制进程终止时，内核会将会话中的所有进程与控制终端解除关联，将控制终端与会话解除关联（以便它可以被另一个会话的会话领导者作为控制终端获取），并通过向前台进程组成员发送 `SIGHUP` 信号，告知它们其控制终端已丢失。

我们将在接下来的章节中详细讨论这两种情况。

### 注意

`SIGHUP` 信号还有其他用途。在 孤儿进程组（和 `SIGHUP` 重新讨论)") 中，我们将看到当进程组成为孤儿时会生成 `SIGHUP`。此外，手动发送 `SIGHUP` 通常用于触发守护进程重新初始化或重新读取其配置文件。（根据定义，守护进程没有控制终端，因此无法从内核接收 `SIGHUP`。）我们在第 37.4 节中描述了 `SIGHUP` 在守护进程中的应用。

### Shell 对 `SIGHUP` 的处理

在登录会话中，shell 通常是终端的控制进程。大多数 shell 都会在交互式运行时建立一个 `SIGHUP` 的处理程序。该处理程序会终止 shell，但在此之前，会向 shell 创建的每个进程组（包括前台和后台）发送 `SIGHUP` 信号。（`SIGHUP` 信号后可能会跟随一个 `SIGCONT` 信号，具体取决于 shell 和作业是否被暂停。）这些进程组中的进程如何响应 `SIGHUP` 是依赖于应用的；如果没有采取特别的行动，它们默认会被终止。

### 注意

一些作业控制 shell 也会在 shell 正常退出时（例如，当我们显式注销或在 shell 窗口中输入 *Control-D* 时）向暂停的后台作业发送 `SIGHUP`。这一点在 *bash* 和 Korn shell 中都有实现（在第一次注销尝试时会打印一条消息）。

*nohup(1)* 命令可用于使命令免受 `SIGHUP` 信号的影响——也就是说，以将 `SIGHUP` 的处理方式设置为 `SIG_IGN` 的方式启动它。*bash* 内置命令 *disown* 具有类似的功能，它将作业从 shell 的作业列表中移除，从而在 shell 终止时该作业不会收到 `SIGHUP`。

我们可以使用 示例 34-3 中的程序来演示当 shell 接收到 `SIGHUP` 时，它会将 `SIGHUP` 发送给它所创建的作业。该程序的主要任务是创建一个子进程，然后让父进程和子进程都暂停，以捕获 `SIGHUP` 并在接收到时显示一条消息。如果程序提供了一个可选的命令行参数（可以是任意字符串），则子进程将自己置于一个不同的进程组中（在同一个会话内）。这有助于展示即使进程与 shell 在同一个会话中，shell 也不会向它没有创建的进程组发送 `SIGHUP`。 （由于程序的最终 `for` 循环会永远循环，因此该程序使用 *alarm()* 来设置定时器以发送 `SIGALRM`。未处理的 `SIGALRM` 信号的到达会确保进程终止，除非该进程已被其他方式终止。）

示例 34-3. 捕获 `SIGHUP`

```
`pgsjc/catch_SIGHUP.c`
#define _XOPEN_SOURCE 500
#include <unistd.h>
#include <signal.h>
#include "tlpi_hdr.h"

static void
handler(int sig)
{
}
int
main(int argc, char *argv[])
{
    pid_t childPid;
    struct sigaction sa;

    setbuf(stdout, NULL);       /* Make stdout unbuffered */

    sigemptyset(&sa.sa_mask);
    sa.sa_flags = 0;
    sa.sa_handler = handler;
    if (sigaction(SIGHUP, &sa, NULL) == -1)
        errExit("sigaction");

    childPid = fork();
    if (childPid == -1)
        errExit("fork");

    if (childPid == 0 && argc > 1)
        if (setpgid(0, 0) == -1)        /* Move to new process group */
            errExit("setpgid");

    printf("PID=%ld; PPID=%ld; PGID=%ld; SID=%ld\n", (long) getpid(),
            (long) getppid(), (long) getpgrp(), (long) getsid(0));

    alarm(60);                 /* An unhandled SIGALRM ensures this process
                                  will die if nothing else terminates it */
    for(;;) {                  /* Wait for signals */
        pause();
        printf("%ld: caught SIGHUP\n", (long) getpid());
    }
}
     `pgsjc/catch_SIGHUP.c`
```

假设我们在终端窗口中输入以下命令来运行 示例 34-3 的两个实例，然后我们关闭终端窗口：

```
$ `echo $$`                                   *PID of shell is ID of session*
5533
$ `./catch_SIGHUP > samegroup.log 2>&1 &`
$ `./catch_SIGHUP x > diffgroup.log 2>&1`
```

第一个命令会创建两个进程，这些进程保持在 shell 创建的进程组中。第二个命令创建了一个子进程，将自己放入一个单独的进程组。

当我们查看 `samegroup.log` 时，我们看到它包含以下输出，表明该进程组中的两个成员都收到了 shell 发送的信号：

```
$ `cat samegroup.log`
PID=5612; PPID=5611; PGID=5611; SID=5533    *Child*
PID=5611; PPID=5533; PGID=5611; SID=5533    *Parent*
5611: caught SIGHUP
5612: caught SIGHUP
```

当我们检查 `diffgroup.log` 时，我们会看到以下输出，表明当 shell 接收到 `SIGHUP` 时，它没有向它没有创建的进程组发送信号：

```
$ `cat diffgroup.log`
PID=5614; PPID=5613; PGID=5614; SID=5533    *Child*
PID=5613; PPID=5533; PGID=5613; SID=5533    *Parent*
5613: caught SIGHUP                         *Parent was signaled, but not child*
```

### `SIGHUP` 和控制进程的终止

如果由于终端断开连接而发送给控制进程的 `SIGHUP` 信号导致控制进程终止，则 `SIGHUP` 会发送给终端前台进程组的所有成员（参见 进程终止详情）。这种行为是控制进程终止的结果，而不是与 `SIGHUP` 信号特定相关的行为。如果控制进程由于任何原因终止，则前台进程组会收到 `SIGHUP` 信号。

### 注意

在 Linux 上，`SIGHUP` 信号后会跟随一个 `SIGCONT` 信号，以确保如果进程组之前由于信号而被停止，则该进程组会恢复。然而，SUSv3 没有指定这种行为，并且大多数其他 UNIX 实现不会在这种情况下发送 `SIGCONT`。

我们可以使用示例 34-4 中的程序来演示控制进程的终止导致`SIGHUP`信号发送到终端前台进程组的所有成员。此程序为每个命令行参数创建一个子进程 ![](img/U002.png)。如果相应的命令行参数是字母*d*，则子进程将自己放入自己的（不同的）进程组 ![](img/U003.png)；否则，子进程将与父进程保持在同一进程组内。（我们使用字母*s*来指定后者的行为，尽管除了*d*外，任何字母都可以使用。）然后，每个子进程会为`SIGHUP`信号建立一个处理程序 ![](img/U004.png)。为了确保它们在没有事件终止它们的情况下也能终止，父进程和子进程都调用*alarm()*设置一个定时器，在 60 秒后发送`SIGALRM`信号 ![](img/U005.png)。最后，所有进程（包括父进程）都会打印出它们的进程 ID 和进程组 ID ![](img/U006.png)，然后循环等待信号到达 ![](img/U007.png)。当信号到达时，处理程序打印出进程 ID 和信号编号 ![](img/U001.png)。

示例 34-4. 当终端断开连接时捕获`SIGHUP`

```
`pgsjc/disc_SIGHUP.c`
    #define _GNU_SOURCE     /* Get strsignal() declaration from <string.h> */
    #include <string.h>
    #include <signal.h>
    #include "tlpi_hdr.h"

    static void             /* Handler for SIGHUP */
    handler(int sig)
    {
    printf("PID %ld: caught signal %2d (%s)\n", (long) getpid(),
                sig, strsignal(sig));
                            /* UNSAFE (see Section 21.1.2) */
    }
        int
    main(int argc, char *argv[])
    {
        pid_t parentPid, childPid;
        int j;
        struct sigaction sa;

        if (argc < 2 || strcmp(argv[1], "--help") == 0)
            usageErr("%s {d|s}... [ > sig.log 2>&1 ]\n", argv[0]);

        setbuf(stdout, NULL);               /* Make stdout unbuffered */

        parentPid = getpid();
        printf("PID of parent process is:       %ld\n", (long) parentPid);
        printf("Foreground process group ID is: %ld\n",
                (long) tcgetpgrp(STDIN_FILENO));
    for (j = 1; j < argc; j++) {        /* Create child processes */
            childPid = fork();
            if (childPid == -1)
                errExit("fork");

            if (childPid == 0) {            /* If child... */
            if (argv[j][0] == 'd')     /* 'd' --> to different pgrp */
                    if (setpgid(0, 0) == -1)
                        errExit("setpgid");

                sigemptyset(&sa.sa_mask);
                sa.sa_flags = 0;
                sa.sa_handler = handler;
            if (sigaction(SIGHUP, &sa, NULL) == -1)
                    errExit("sigaction");
                break;                      /* Child exits loop */
            }
        }

        /* All processes fall through to here */
    alarm(60);          /* Ensure each process eventually terminates */

    printf("PID=%ld PGID=%ld\n", (long) getpid(), (long) getpgrp());
        for (;;)
        pause();        /* Wait for signals */
      }
          `pgsjc/disc_SIGHUP.c`
```

假设我们在一个终端窗口中运行示例 34-4 程序，命令如下：

```
$ `exec ./disc_SIGHUP d s s > sig.log 2>&1`
```

*exec*命令是一个内建命令，使得 shell 执行*exec()*，将自身替换为指定的程序。由于 shell 是终端的控制进程，因此我们的程序现在成为了控制进程，并将在终端窗口关闭时接收到`SIGHUP`信号。关闭终端窗口后，我们在`sig.log`文件中看到以下几行：

```
PID of parent process is:       12733
Foreground process group ID is: 12733
PID=12755 PGID=12755                *First child is in a different process group*
PID=12756 PGID=12733                *Remaining children are in same PG as parent*
PID=12757 PGID=12733
PID=12733 PGID=12733                *This is the parent process*
PID 12756: caught signal  1 (Hangup)
PID 12757: caught signal  1 (Hangup)
```

关闭终端窗口导致`SIGHUP`信号被发送到控制进程（父进程），因此父进程终止了。我们看到与父进程在同一进程组中的两个子进程（即终端的前台进程组）也都收到了`SIGHUP`信号。然而，位于单独（后台）进程组中的子进程并没有收到此信号。

## 作业控制

作业控制是 1980 年左右首次出现在 BSD 的 C shell 中的一种功能。作业控制允许用户同时执行多个命令（作业），其中一个在前台执行，其他的在后台执行。作业可以被停止和恢复，并可以在前台和后台之间移动，具体内容在以下段落中描述。

### 注意

在最初的 POSIX.1 标准中，作业控制的支持是可选的。后来，UNIX 标准将其作为强制要求。

在字符型傻瓜终端（物理终端设备，仅限显示 ASCII 字符）的时代，许多 shell 用户都知道如何使用 shell 作业控制命令。随着位图显示器运行 X 窗口系统的出现，了解 shell 作业控制的知识变得不太常见。然而，作业控制仍然是一个有用的功能。使用作业控制来管理多个同时执行的命令，比在多个窗口之间来回切换可能更快更简单。对于那些不熟悉作业控制的读者，我们将从简短的教程开始，介绍其使用方法。接着，我们将进一步探讨作业控制的实现细节，并考虑作业控制对应用设计的影响。

### 在 shell 中使用作业控制

当我们输入一个以符号`&` (`&`) 结尾的命令时，它会作为后台作业运行，如以下示例所示：

```
[1] 18932                           *Job 1: process running* *grep* *has PID 18932*
$ `sleep 60 &`
[2] 18934                           *Job 2: process running* *sleep* *has PID 18934*
```

每个被置于后台的作业都会被 shell 分配一个唯一的作业编号。该作业编号会在作业开始执行时以方括号的形式显示在后台，并且在通过各种作业控制命令操作或监控作业时也会显示。作业编号后面的数字是为了执行该命令而创建的进程的进程 ID，或者在管道的情况下，管道中最后一个进程的进程 ID。在接下来的命令中，作业可以通过 *%num* 这种表示法来引用，其中 *num* 是由 shell 分配给该作业的编号。

### 注意

在许多情况下，*%num* 参数可以省略，在这种情况下，默认使用 *当前* 作业。当前作业是最后一个在前台停止的作业（使用下面描述的 *suspend* 字符），或者如果没有这样的作业，则是最后一个在后台启动的作业。（不同的 shell 在如何确定哪个后台作业是当前作业方面有一些细微的差异。）此外，表示法 %% 或 %+ 指代当前作业，而表示法 %- 则指代 *之前的当前作业*。当前作业和之前的当前作业分别在 *jobs* 命令的输出中用加号（+）和减号（-）标记，我们接下来将对此进行描述。

*jobs* shell 内建命令列出所有后台作业：

```
$ `jobs`
[1]- Running        grep -r SIGHUP /usr/src/linux >x &
[2]+ Running        sleep 60 &
```

在这一点上，shell 是终端的前台进程。由于只有前台进程才能从控制终端读取输入并接收终端生成的信号，因此有时需要将后台作业移到前台。这可以通过使用 *fg* shell 内建命令来完成：

```
$ `fg %1`
grep -r SIGHUP /usr/src/linux >x
```

如这个示例所示，shell 每次将作业从前台移到后台时，都会重新显示该作业的命令行。下面，我们还会看到，每当作业在后台的状态发生变化时，shell 也会执行此操作。

当作业在前台运行时，我们可以使用终端的*suspend*字符（通常是*Control-Z*）来暂停它，这会将`SIGTSTP`信号发送给终端的前台进程组：

```
*Type Control-Z*
[1]+ Stopped        grep -r SIGHUP /usr/src/linux >x
```

在我们按下*Control-Z*后，shell 会显示已在后台暂停的命令。如果需要，我们可以使用*fg*命令将作业恢复到前台，或者使用*bg*命令将其恢复到后台。在这两种情况下，shell 都会通过发送`SIGCONT`信号来恢复暂停的作业。

```
$ `bg %1`
[1]+ grep -r SIGHUP /usr/src/linux >x &
```

我们可以通过向后台作业发送`SIGSTOP`信号来暂停它：

```
$ `kill -STOP %1`
[1]+ Stopped        grep -r SIGHUP /usr/src/linux >x
$ `jobs`
[1]+ Stopped        grep -r SIGHUP /usr/src/linux >x
[2]- Running        sleep 60 &
$ `bg %1`                             *Restart job in background*
[1]+ grep -r SIGHUP /usr/src/linux >x &
```

### 注意

Korn 和 C shell 提供了*stop*命令，作为*kill -stop*命令的简写。

当一个后台作业最终完成时，shell 会在显示下一个 shell 提示符之前打印一条消息：

```
*Press Enter to see a further shell prompt*
[1]- Done           grep -r SIGHUP /usr/src/linux >x
[2]+ Done           sleep 60
$
```

只有前台作业中的进程可以从控制终端读取。这一限制防止了多个作业争夺终端输入。如果后台作业尝试从终端读取，它会收到一个`SIGTTIN`信号。`SIGTTIN`的默认行为是暂停该作业：

```
$ `cat > x.txt &`
[1] 18947
$
*Press Enter once more in order to see*
 *job state changes displayed prior to next shell prompt*
[1]+ Stopped        cat >x.txt
$
```

### 注意

在某些情况下，可能不需要按*Enter*键来查看前一个示例及一些后续示例中的作业状态变化。根据内核调度的决策，shell 可能在显示下一个 shell 提示符之前就收到关于后台作业状态变化的通知。

在此时，我们必须将作业带到前台（*fg*），并提供所需的输入。如果需要，我们可以通过先暂停作业再将其恢复到后台（*bg*）来继续执行作业。（当然，在这个特定的例子中，*cat*会立即被暂停，因为它会再次尝试从终端读取。）

默认情况下，后台作业被允许向控制终端输出。然而，如果终端设置了`TOSTOP`标志（*终端输出停止*，终端标志），那么后台作业试图执行终端输出时，会生成一个`SIGTTOU`信号。（我们可以使用*stty*命令设置`TOSTOP`标志，该命令在第 62.3 节中有详细说明。）与`SIGTTIN`一样，`SIGTTOU`信号也会暂停作业。

```
$ `stty tostop`                       *Enable* TOSTOP *flag for this terminal*
$ `date &`
[1] 19023
$
*Press Enter once more to see job state changes displayed prior to next shell prompt*
[1]+ Stopped        date
```

我们可以通过将作业带到前台来查看作业的输出：

```
$ `fg`
date
Tue Dec 28 16:20:51 CEST 2010
```

作业控制下作业的各种状态，以及用于在这些状态之间移动作业的 shell 命令和终端字符（以及随之产生的信号），总结在图 34-2 中。该图还包括一个概念上的*已终止*状态。这个状态可以通过向作业发送各种信号来实现，包括`SIGINT`和`SIGQUIT`，这些信号可以通过键盘生成。

![作业控制状态](img/34-2_PGSJC-job-control.png.jpg)图 34-2. 作业控制状态

### 实现作业控制

本节将探讨作业控制实现的各个方面，并以一个示例程序结束，该程序使作业控制的操作更加透明。

尽管在原始的 POSIX.1 标准中是可选的，但后来的标准，包括 SUSv3，要求实现支持作业控制。此支持要求如下：

+   实现必须提供特定的作业控制信号：`SIGTSTP`、`SIGSTOP`、`SIGCONT`、`SIGTTOU` 和 `SIGTTIN`。此外，`SIGCHLD` 信号（`SIGCHLD` 信号）也是必需的，因为它允许 shell（所有作业的父进程）在其子进程终止或停止时获知。

+   终端驱动程序必须支持生成作业控制信号，以便在输入特定字符或从后台作业执行某些终端 I/O 和其他终端操作（如下所述）时，能够将适当的信号（如图 34-2 所示）发送到相关的进程组。为了能够执行这些操作，终端驱动程序还必须记录与终端相关的会话 ID（控制进程）和前台进程组 ID（图 34-1）。

+   Shell 必须支持作业控制（大多数现代 shell 都支持）。这种支持以先前描述的命令形式提供，用于在前台和后台之间移动作业并监控作业的状态。这些命令中的某些会向作业发送信号（如图 34-2 所示）。此外，在执行将作业从*前台运行*状态移动到其他状态的操作时，shell 会调用 *tcsetpgrp()* 来调整终端驱动程序记录的前台进程组。

### 注意

在发送信号：*kill()*")中，我们看到，通常只有当发送进程的实际或有效用户 ID 与接收进程的实际用户 ID 或保存的设定用户 ID 匹配时，才能向进程发送信号。然而，`SIGCONT` 是这个规则的例外。内核允许进程（例如 shell）向同一会话中的任何进程发送 `SIGCONT`，而不考虑进程凭据。对 `SIGCONT` 规则的放宽是必要的，这样如果用户启动一个改变其凭据（特别是其实际用户 ID）的 set-user-ID 程序，即使该程序被停止，仍然可以通过 `SIGCONT` 恢复它。

#### `SIGTTIN` 和 `SIGTTOU` 信号

SUSv3 规范（并且 Linux 实现了）一些特殊情况，这些情况适用于生成背景作业的 `SIGTTIN` 和 `SIGTTOU` 信号：

+   如果进程当前阻塞或忽略`SIGTTIN`信号，则不会发送该信号。相反，从控制终端执行*read()*操作失败，并将*errno*设置为`EIO`。这样做的原因是，进程否则无法知道*read()*操作被拒绝。

+   即使终端的`TOSTOP`标志已设置，如果进程当前阻塞或忽略此信号，则不会发送`SIGTTOU`。相反，允许对控制终端执行*write()*操作（即，忽略`TOSTOP`标志）。

+   无论`TOSTOP`标志的设置如何，某些更改终端驱动程序数据结构的函数会导致如果后台进程尝试在其控制终端上应用这些函数时，生成`SIGTTOU`信号。这些函数包括*tcsetpgrp()*、*tcsetattr()*、*tcflush()*、*tcflow()*、*tcsendbreak()*和*tcdrain()*。（这些函数在第六十二章中有描述。）如果`SIGTTOU`信号被阻塞或忽略，这些调用将成功执行。

#### 示例程序：演示作业控制的操作

示例 34-5 中的程序允许我们看到 shell 如何将管道中的命令组织成一个作业（进程组）。该程序还允许我们监控发送的某些信号以及作业控制下对终端前台进程组设置的更改。该程序设计允许多个实例在管道中运行，如下所示的示例：

```
$ `./job_mon | ./job_mon | ./job_mon`
```

示例 34-5 中的程序执行以下步骤：

+   在启动时，程序为`SIGINT`、`SIGTSTP`和`SIGCONT`信号安装一个统一的处理程序 ![](img/U004.png)。该处理程序执行以下操作：

    +   显示终端的前台进程组 ![](img/U001.png)。为了避免输出重复的行，仅由进程组的领导者执行此操作。

    +   显示进程的 ID、进程在管道中的位置和接收到的信号 ![](img/U002.png)。

    +   如果处理程序捕获到`SIGTSTP`信号，它必须做一些额外的工作，因为在捕获时，这个信号并不会停止进程。因此，为了真正停止进程，处理程序会发送`SIGSTOP`信号 ![](img/U003.png)，该信号始终会停止进程。（我们在处理作业控制信号中详细阐述了如何处理`SIGTSTP`。）

+   如果程序是管道中的初始进程，它会打印所有进程产生的输出的标题 ![](img/U006.png)。为了测试它是否是管道中的初始（或最终）进程，程序使用 *isatty()* 函数（在终端识别中描述）来检查它的标准输入（或输出）是否是一个终端 ![](img/U005.png)。如果指定的文件描述符指向一个管道，*isatty()* 会返回 false（0）。

+   程序构建一个消息传递给其管道中的后继进程。这个消息是一个整数，表示此进程在管道中的位置。因此，对于初始进程，消息包含数字 1。如果程序是管道中的初始进程，消息被初始化为 0。如果它不是管道中的初始进程，程序会先从其前一个进程读取此消息 ![](img/U007.png)。程序在继续到下一步之前会增加消息值 ![](img/U008.png)。

+   无论它在管道中的位置如何，程序都会显示一行，包含它的管道位置、进程 ID、父进程 ID、进程组 ID 和会话 ID ![](img/U009.png)。

+   除非它是管道中的最后一个命令，否则程序会为其管道中的后续进程写一个整数消息 ![](img/U010.png)。

+   最后，程序永远循环，使用 *pause()* 等待信号 ![](img/U011.png)。

示例 34-5. 观察作业控制下的进程处理

```
`pgsjc/job_mon.c`
    #define _GNU_SOURCE     /* Get declaration of strsignal() from <string.h> */
    #include <string.h>
    #include <signal.h>
    #include <fcntl.h>
    #include "tlpi_hdr.h"

    static int cmdNum;              /* Our position in pipeline */

    static void                     /* Handler for various signals */
    handler(int sig)
    {
        /* UNSAFE: This handler uses non-async-signal-safe functions
           (fprintf(), strsignal(); see Section 21.1.2) */
        if (getpid() == getpgrp())          /* If process group leader */
          fprintf(stderr, "Terminal FG process group: %ld\n",
                    (long) tcgetpgrp(STDERR_FILENO));
    fprintf(stderr, "Process %ld (%d) received signal %d (%s)\n",
                    (long) getpid(), cmdNum, sig, strsignal(sig));

        /* If we catch SIGTSTP, it won't actually stop us. Therefore we
           raise SIGSTOP so we actually get stopped. */

    if (sig == SIGTSTP)
            raise(SIGSTOP);
    }

    int
    main(int argc, char *argv[])
    {
        struct sigaction sa;

        sigemptyset(&sa.sa_mask);
        sa.sa_flags = SA_RESTART;
        sa.sa_handler = handler;
    if (sigaction(SIGINT, &sa, NULL) == -1)
            errExit("sigaction");
        if (sigaction(SIGTSTP, &sa, NULL) == -1)
            errExit("sigaction");
        if (sigaction(SIGCONT, &sa, NULL) == -1)
            errExit("sigaction");

        /* If stdin is a terminal, this is the first process in pipeline:
           print a heading and initialize message to be sent down pipe */

    if (isatty(STDIN_FILENO)) {
            fprintf(stderr, "Terminal FG process group: %ld\n",
                    (long) tcgetpgrp(STDIN_FILENO));
        fprintf(stderr, "Command   PID  PPID  PGRP   SID\n");
            cmdNum = 0;

        } else {            /* Not first in pipeline, so read message from pipe */
        if (read(STDIN_FILENO, &cmdNum, sizeof(cmdNum)) <= 0)
                fatal("read got EOF or error");
        }

    cmdNum++;
    fprintf(stderr, "%4d    %5ld %5ld %5ld %5ld\n", cmdNum,
                (long) getpid(), (long) getppid(),
                (long) getpgrp(), (long) getsid(0));

        /* If not the last process, pass a message to the next process */

        if (!isatty(STDOUT_FILENO))   /* If not tty, then should be pipe */0
        if (write(STDOUT_FILENO, &cmdNum, sizeof(cmdNum)) == -1)
                errMsg("write");

    for(;;)             /* Wait for signals */
            pause();
    }

           `pgsjc/job_mon.c`
```

以下的 Shell 会话演示了在示例 34-5 中使用程序的方法。我们首先显示 Shell 的进程 ID（它是会话的领导者，且是进程组的领导者，进程组中只有它一个成员），然后创建一个包含两个进程的后台作业：

```
$ `echo $$`                       *Show PID of the shell*
1204
$ `./job_mon | ./job_mon &`                   *Start a job containing 2 processes*
[1] 1227
Terminal FG process group: 1204
Command   PID  PPID  PGRP   SID
   1     1226  1204  1226  1204
   2     1227  1204  1226  1204
```

从上述输出可以看出，Shell 仍然是终端的前台进程。我们还可以看到新作业与 Shell 在同一个会话中，并且所有进程都在同一个进程组中。查看进程 ID，我们可以看到作业中的进程是按照命令行上给出的命令顺序创建的。（大多数 Shell 是这样做的，但某些 Shell 实现会以不同的顺序创建进程。）

我们继续，创建一个由三个进程组成的第二个后台作业：

```
$ `./job_mon | ./job_mon | ./job_mon &`
[2] 1230
Terminal FG process group: 1204
Command   PID  PPID  PGRP   SID
   1     1228  1204  1228  1204
   2     1229  1204  1228  1204
   3     1230  1204  1228  1204
```

我们看到 Shell 仍然是终端的前台进程组。我们还看到新作业的进程与 Shell 在同一个会话中，但在与第一个作业不同的进程组中。现在，我们将第二个作业带入前台，并向它发送 `SIGINT` 信号：

```
$ `fg`
./job_mon | ./job_mon | ./job_mon
*Type Control-C to generate* SIGINT *(signal2)*
Process 1230 (3) received signal 2 (Interrupt)
Process 1229 (2) received signal 2 (Interrupt)
Terminal FG process group: 1228
Process 1228 (1) received signal 2 (Interrupt)
```

从上述输出中，我们看到 `SIGINT` 信号已被发送到前台进程组中的所有进程。我们还看到该作业现在是终端的前台进程组。接下来，我们向该作业发送 `SIGTSTP` 信号：

```
*Type Control-Z to generate* SIGTSTP *(signal 20 on Linux/x86-32).*
Process 1230 (3) received signal 20 (Stopped)
Process 1229 (2) received signal 20 (Stopped)
Terminal FG process group: 1228
Process 1228 (1) received signal 20 (Stopped)

[2]+  Stopped       ./job_mon | ./job_mon | ./job_mon
```

现在，所有进程组的成员都已停止。输出表明进程组 1228 是前台作业。然而，在该作业停止后，虽然从输出中看不出来，shell 成为了前台进程组。

然后，我们使用 *bg* 命令重新启动作业，该命令将 `SIGCONT` 信号发送到作业中的进程：

```
$ `bg`                                        *Resume job in background*
[2]+ ./job_mon | ./job_mon | ./job_mon &
Process 1230 (3) received signal 18 (Continued)
Process 1229 (2) received signal 18 (Continued)
Terminal FG process group: 1204             *The shell is in the foreground*
Process 1228 (1) received signal 18 (Continued)
$ `kill %1 %2`                                *We’ve finished: clean up*
[1]-  Terminated    ./job_mon | ./job_mon
[2]+  Terminated    ./job_mon | ./job_mon | ./job_mon
```

### 处理作业控制信号

由于作业控制的操作对大多数应用程序是透明的，因此它们无需为处理作业控制信号采取特殊行动。一个例外是执行屏幕处理的程序，如 *vi* 和 *less*。这些程序控制文本在终端上的精确布局，并更改各种终端设置，包括允许终端按字符（而不是按行）逐次读取输入的设置。（我们在第六十二章中描述了各种终端设置。）

屏幕处理程序需要处理终端停止信号（`SIGTSTP`）。信号处理程序应将终端重置为标准（逐行）输入模式，并将光标放置在终端的左下角。当程序恢复时，它会将终端设置回程序所需的模式，检查终端窗口大小（此时用户可能已更改），并重新绘制屏幕以显示所需内容。

### 注意

当我们挂起或退出一个终端处理程序时，比如在 *xterm* 或其他终端仿真器上使用 *vi*，通常会看到终端被重绘为程序启动前显示的文本。终端仿真器通过捕捉两个字符序列来实现这一效果，这些程序在使用 *terminfo* 或 *termcap* 包时，必须在获取和释放终端布局控制时输出这两个序列。第一个序列称为 *smcup*（通常是 *Escape* 后跟 `[?1049h]`），它使终端仿真器切换到“备用”屏幕。第二个序列称为 *rmcup*（通常是 *Escape* 后跟 `[?1049l]`），它使终端仿真器恢复到默认屏幕，从而使终端恢复到屏幕处理程序控制终端之前的原始文本。

在处理`SIGTSTP`时，我们需要注意一些细节。我们已经在实现作业控制中提到过其中的第一个：如果捕获到`SIGTSTP`，它就不会执行默认的停止进程的操作。我们在示例 34-5 中通过让`SIGTSTP`的处理程序触发`SIGSTOP`信号来解决这个问题。由于`SIGSTOP`不能被捕获、阻塞或忽略，因此它保证立即停止进程。然而，这种做法并不完全正确。在等待状态值中，我们看到父进程可以使用`wait()`或`waitpid()`返回的等待状态值来判断是哪个信号导致其子进程停止。如果我们在`SIGTSTP`的处理程序中触发`SIGSTOP`信号，父进程会误认为子进程是被`SIGSTOP`信号停止的。

在这种情况下，正确的做法是让`SIGTSTP`的处理程序触发一个额外的`SIGTSTP`信号来停止进程，具体如下：

1.  处理程序将`SIGTSTP`的处理方式重置为默认值（`SIG_DFL`）。

1.  处理程序触发`SIGTSTP`。

1.  由于在进入处理程序时`SIGTSTP`信号已被阻塞（除非指定了`SA_NODEFER`标志），处理程序会解除阻塞该信号。此时，之前步骤中触发的挂起的`SIGTSTP`信号将执行其默认操作：进程会立即被挂起。

1.  稍后，当进程接收到`SIGCONT`时，将会恢复执行。此时，处理程序的执行继续进行。

1.  在返回之前，处理程序会重新阻塞`SIGTSTP`信号，并重新建立自己以处理下一个`SIGTSTP`信号的发生。

重新阻塞`SIGTSTP`信号的步骤是为了防止如果在处理程序重新建立自己之后，但在返回之前，又送来了另一个`SIGTSTP`信号时，处理程序会被递归调用。如*signal()*的实现与可移植性的实现与可移植性")中所指出的，信号处理程序的递归调用可能会导致堆栈溢出，尤其是在快速连续发送信号的情况下。阻塞信号还可以避免信号处理程序在重新建立自己后但在返回之前需要执行其他操作（例如保存或恢复全局变量的值）时出现问题。

#### 示例程序

示例 34-6 中的处理程序实现了上述步骤，正确处理了 `SIGTSTP` 信号。（我们在示例 62-4 中展示了另一个 `SIGTSTP` 信号处理的例子，在终端行速率（比特率）中也有提到。）在建立了 `SIGTSTP` 信号处理程序后，程序的*main()*函数进入一个循环，等待信号。以下是我们运行此程序时可能看到的输出：

```
$ `./handling_SIGTSTP`
*Type Control-Z, sending* SIGTSTP
Caught SIGTSTP                 *This message is printed by* SIGTSTP *handler*

[1]+  Stopped       ./handling_SIGTSTP
$ `fg`                           *Sends* SIGCONT
./handling_SIGTSTP
Exiting SIGTSTP handler        *Execution of handler continues; handler returns*
Main                           *pause() call in main() was interrupted by handler*
*Type Control-C to terminate the program*
```

在像 *vi* 这样的屏幕处理程序中，示例 34-6 中的信号处理程序内的*printf()*调用会被替换为修改终端模式并重新绘制终端显示的代码，正如上述所述。（由于需要避免调用非异步信号安全函数，可重入和异步信号安全函数中有描述，信号处理程序应该通过设置标志通知主程序重新绘制屏幕。）

请注意，`SIGTSTP` 信号处理程序可能会中断某些阻塞的系统调用（如系统调用的中断与重启中所述）。这一点在上述程序输出中得到了说明，具体表现为，在*pause()*调用被中断后，主程序打印出*Main*信息。

示例 34-6. 处理 `SIGTSTP`

```
`pgsjc/handling_SIGTSTP.c`
#include <signal.h>
#include "tlpi_hdr.h"

static void                             /* Handler for SIGTSTP */
tstpHandler(int sig)
{
    sigset_t tstpMask, prevMask;
    int savedErrno;
    struct sigaction sa;

    savedErrno = errno;                 /* In case we change 'errno' here */

    printf("Caught SIGTSTP\n");         /* UNSAFE (see Section 21.1.2) */

    if (signal(SIGTSTP, SIG_DFL) == SIG_ERR)
        errExit("signal");              /* Set handling to default */

    raise(SIGTSTP);                     /* Generate a further SIGTSTP */

    /* Unblock SIGTSTP; the pending SIGTSTP immediately suspends the program */

    sigemptyset(&tstpMask);
    sigaddset(&tstpMask, SIGTSTP);
    if (sigprocmask(SIG_UNBLOCK, &tstpMask, &prevMask) == -1)
        errExit("sigprocmask");

    /* Execution resumes here after SIGCONT */

    if (sigprocmask(SIG_SETMASK, &prevMask, NULL) == -1)
        errExit("sigprocmask");         /* Reblock SIGTSTP */

    sigemptyset(&sa.sa_mask);           /* Reestablish handler */
    sa.sa_flags = SA_RESTART;
    sa.sa_handler = tstpHandler;
    if (sigaction(SIGTSTP, &sa, NULL) == -1)
        errExit("sigaction");

    printf("Exiting SIGTSTP handler\n");
    errno = savedErrno;
}

int
main(int argc, char *argv[])
{
    struct sigaction sa;
    /* Only establish handler for SIGTSTP if it is not being ignored */

    if (sigaction(SIGTSTP, NULL, &sa) == -1)
        errExit("sigaction");

    if (sa.sa_handler != SIG_IGN) {
        sigemptyset(&sa.sa_mask);
        sa.sa_flags = SA_RESTART;
        sa.sa_handler = tstpHandler;
        if (sigaction(SIGTSTP, &sa, NULL) == -1)
            errExit("sigaction");
    }

    for (;;) {                          /* Wait for signals */
        pause();
        printf("Main\n");
    }
}
      `pgsjc/handling_SIGTSTP.c`
```

#### 处理被忽略的作业控制信号和终端生成信号

示例 34-6 中的程序仅在未忽略`SIGTSTP`信号时才会建立信号处理程序。这是一个更一般的规则的实例，应用程序应当仅在信号未被事先忽略的情况下处理作业控制和终端生成的信号。在作业控制信号（`SIGTSTP`、`SIGTTIN` 和 `SIGTTOU`）的情况下，这可以防止应用程序尝试处理这些信号，特别是在它是从非作业控制的 shell（如传统 Bourne shell）启动时。在非作业控制的 shell 中，这些信号的处理方式被设置为 `SIG_IGN`；只有作业控制的 shell 才会将这些信号的处理方式设置为 `SIG_DFL`。

类似的说法也适用于从终端生成的其他信号：`SIGINT`、`SIGQUIT`和`SIGHUP`。对于`SIGINT`和`SIGQUIT`，原因在于当命令在非作业控制的 shell 中后台执行时，生成的进程并没有被放入单独的进程组中。相反，进程保持在与 shell 相同的组中，并且在执行命令之前，shell 会将`SIGINT`和`SIGQUIT`的处理设置为忽略。这确保了如果用户输入终端的*中断*或*退出*字符（这些字符应该仅影响名义上在前台的作业），进程不会被终止。如果进程随后撤销了 shell 对这些信号处理的操作，它就再次变得容易受到这些信号的影响。

如果通过 *nohup(1)* 执行命令，则会忽略`SIGHUP`信号。这样可以防止命令因为终端挂起而被终止。因此，如果命令被忽略，应用程序不应该尝试更改信号的处理方式。

### 孤儿进程组（以及`SIGHUP`的回顾）

在孤儿进程和僵尸进程中，我们看到，孤儿进程是指其父进程终止后被 *init*（进程 ID 1）收养的进程。在程序中，我们可以使用以下代码创建一个孤儿子进程：

```
if (fork() != 0)                /* Exit if parent (or on error) */
    exit(EXIT_SUCCESS);
```

假设我们在从 shell 执行的程序中包含这段代码。图 34-3 显示了父进程退出前后进程的状态。

在父进程终止后，图 34-3 中的子进程不仅是一个孤儿进程，还是*孤儿进程组*的一部分。SUSv3 定义一个进程组为孤儿进程组，如果“每个成员的父进程要么是该组的成员，要么不是该组会话的成员。”换句话说，只有当至少有一个成员的父进程在同一会话中，但在不同的进程组时，进程组才不是孤儿进程组。在图 34-3 中，包含子进程的进程组是孤儿进程组，因为子进程处于一个单独的进程组中，并且它的父进程（*init*）处于不同的会话中。

### 注意

根据定义，会话领导者处于孤儿进程组中。因为 *setsid()* 会在新会话中创建一个新的进程组，而会话领导者的父进程处于不同的会话中。

![孤儿进程组创建步骤](img/34-3_PGSJC-orphaned-pgrp.png.jpg)图 34-3. 孤儿进程组创建步骤

为了理解为什么孤儿进程组很重要，我们需要从外壳作业控制的角度来看待问题。请考虑以下基于图 34-3 的情境：

1.  在父进程退出之前，子进程被停止（可能是因为父进程向其发送了停止信号）。

1.  当父进程退出时，外壳会将父进程的进程组从其作业列表中移除。子进程被*init*收养，并成为终端的后台进程。包含子进程的进程组变成孤儿进程组。

1.  此时，没有进程通过*wait()*监控停止子进程的状态。

由于外壳没有创建子进程，因此它不知道子进程的存在，也不知道子进程与已故的父进程属于同一个进程组。此外，*init*进程只检查是否有终止的子进程，然后回收产生的僵尸进程。因此，停止的子进程可能会永远被搁置，因为没有其他进程知道应该发送`SIGCONT`信号来使其恢复执行。

即使孤儿进程组中的一个停止进程在不同会话中仍有存活的父进程，该父进程也无法保证能够向停止的子进程发送`SIGCONT`信号。进程可以向同一会话中的任何其他进程发送`SIGCONT`信号，但如果子进程在不同的会话中，则发送信号的正常规则适用（发送信号：*kill()*")），因此如果子进程是一个已经更改了其凭据的特权进程，父进程可能无法向子进程发送信号。

为了防止上述情况的发生，SUSv3 规定，如果一个进程组变成了孤儿进程组并且其中有任何停止的成员，那么该组的所有成员都会收到`SIGHUP`信号，告知它们已与会话断开连接，随后会发送`SIGCONT`信号，确保它们恢复执行。如果孤儿进程组中没有停止的成员，则不会发送任何信号。

进程组可能变成孤儿进程组，原因可能是同一会话中不同进程组中的最后一个父进程终止，或者是组内最后一个有父进程的进程终止（后者是图 34-3 中所示的情况）。无论是哪种情况，处理新成为孤儿的包含停止子进程的进程组的方式都是一样的。

### 注意

向包含已停止成员的新的孤儿进程组发送`SIGHUP`和`SIGCONT`信号是为了消除作业控制框架中的一个特定漏洞。如果另一个进程（具有适当权限）向其发送停止信号，已经成为孤儿的进程组中的成员仍然可能被停止。在这种情况下，进程将保持停止状态，直到某个进程（同样具有适当权限）向其发送`SIGCONT`信号。

当孤儿进程组的成员调用时，*tcsetpgrp()*函数（前台和后台进程组）会因错误`ENOTTY`而失败，并且对*tcsetattr()*、*tcflush()*、*tcflow()*、*tcsendbreak()*和*tcdrain()*函数的调用（这些函数在第六十二章中有描述）也会因错误`EIO`而失败。

#### 示例程序

示例 34-7 中的程序展示了我们刚才描述的孤儿进程的处理方式。在为`SIGHUP`和`SIGCONT`信号建立处理程序后！[](figs/web/U002.png)，该程序为每个命令行参数创建一个子进程！[](figs/web/U003.png)。然后每个子进程会停止自身（通过发送`SIGSTOP`信号）！[](figs/web/U004.png)，或者等待信号（使用*pause()*）！[](figs/web/U005.png)。子进程的行为由命令行参数是否以字母*s*（表示*stop*）开头来决定。（我们使用以字母*p*开头的命令行参数来指定调用*pause()*的反向操作，尽管可以使用除字母*s*以外的任何字符。）

在创建所有子进程后，父进程会睡眠几秒钟，以便给子进程一些时间进行初始化！[](figs/web/U006.png)。如创建新进程：*fork()*")中所述，使用*sleep()*这种方式虽然不完美，但有时是实现该结果的可行方法。然后父进程退出！[](figs/web/U007.png)，此时包含子进程的进程组变成了孤儿进程组。如果任何子进程因进程组变成孤儿而接收到信号，则会调用信号处理程序，并显示该子进程的进程 ID 和信号编号！[](figs/web/U001.png)。

示例 34-7. `SIGHUP` 和孤儿进程组

```
`pgsjc/orphaned_pgrp_SIGHUP.c`
    #define _GNU_SOURCE     /* Get declaration of strsignal() from <string.h> */
    #include <string.h>
    #include <signal.h>
    #include "tlpi_hdr.h"

    static void             /* Signal handler */
    handler(int sig)
    {
    printf("PID=%ld: caught signal %d (%s)\n", (long) getpid(),
                sig, strsignal(sig));     /* UNSAFE (see Section 21.1.2) */
    }

    int
    main(int argc, char *argv[])
    {
        int j;
        struct sigaction sa;

        if (argc < 2 || strcmp(argv[1], "--help") == 0)
            usageErr("%s {s|p} ...\n", argv[0]);

        setbuf(stdout, NULL);               /* Make stdout unbuffered */

        sigemptyset(&sa.sa_mask);
        sa.sa_flags = 0;
        sa.sa_handler = handler;
    if (sigaction(SIGHUP, &sa, NULL) == -1)
            errExit("sigaction");
        if (sigaction(SIGCONT, &sa, NULL) == -1)
            errExit("sigaction");

        printf("parent: PID=%ld, PPID=%ld, PGID=%ld, SID=%ld\n",
                (long) getpid(), (long) getppid(),
                (long) getpgrp(), (long) getsid(0));

        /* Create one child for each command-line argument */

    for (j = 1; j < argc; j++) {
            switch (fork()) {
            case -1:
                errExit("fork");

            case 0:         /* Child */
                printf("child:  PID=%ld, PPID=%ld, PGID=%ld, SID=%ld\n",
                        (long) getpid(), (long) getppid(),
                        (long) getpgrp(), (long) getsid(0));

                if (argv[j][0] == 's') {    /* Stop via signal */
                    printf("PID=%ld stopping\n", (long) getpid());
                    raise(SIGSTOP);
                } else {                    /* Wait for signal */
                    alarm(60);              /* So we die if not SIGHUPed */
                    printf("PID=%ld pausing\n", (long) getpid());
                pause();
                }

                _exit(EXIT_SUCCESS);

            default:        /* Parent carries on round loop */
                break;
            }
        }

        /* Parent falls through to here after creating all children */

    sleep(3);                           /* Give children a chance to start */
        printf("parent exiting\n");
    exit(EXIT_SUCCESS);                 /* And orphan them and their group */
    }

         `pgsjc/orphaned_pgrp_SIGHUP.c`
```

以下是示例 34-7 中程序两次运行结果的 shell 会话日志：

```
$ `echo $$`                     *Display PID of shell, which is also the session ID*
4785
$ `./orphaned_pgrp_SIGHUP s p`
parent: PID=4827, PPID=4785, PGID=4827, SID=4785
child:  PID=4828, PPID=4827, PGID=4827, SID=4785
PID=4828 stopping
child:  PID=4829, PPID=4827, PGID=4827, SID=4785
PID=4829 pausing
parent exiting
$ PID=4828: caught signal 18 (Continued)
PID=4828: caught signal 1 (Hangup)
PID=4829: caught signal 18 (Continued)
PID=4829: caught signal 1 (Hangup)
*Press Enter to get another shell prompt*
$ `./orphaned_pgrp_SIGHUP p p`
parent: PID=4830, PPID=4785, PGID=4830, SID=4785
child:  PID=4831, PPID=4830, PGID=4830, SID=4785
PID=4831 pausing
child:  PID=4832, PPID=4830, PGID=4830, SID=4785
PID=4832 pausing
parent exiting
```

第一次运行创建了两个子进程，它们处于即将成为孤儿的进程组中：一个子进程停止自己，另一个暂停。（在这个运行中，Shell 提示符出现在子进程输出的中间，因为 Shell 注意到父进程已经退出。）如可以看到，父进程退出后，两个子进程都接收到`SIGCONT`和`SIGHUP`信号。在第二次运行中，创建了两个子进程，它们都没有停止自己，因此在父进程退出时没有发送任何信号。

#### 孤儿进程组与`SIGTSTP`、`SIGTTIN`和`SIGTTOU`信号

孤儿进程组还会影响`SIGTSTP`、`SIGTTIN`和`SIGTTOU`信号的传递语义。

在使用 Shell 中的作业控制中，我们看到如果后台进程尝试从控制终端*read()*，则会发送`SIGTTIN`信号；如果后台进程尝试向控制终端*write()*，并且终端的`TOSTOP`标志已设置，则会发送`SIGTTOU`信号。然而，向孤儿进程组发送这些信号是没有意义的，因为一旦停止，它将永远不会被恢复。因此，内核不发送`SIGTTIN`或`SIGTTOU`，而是使*read()*或*write()*失败，并返回错误`EIO`。

出于类似的原因，如果`SIGTSTP`、`SIGTTIN`或`SIGTTOU`的传递会停止一个孤儿进程组的成员，那么该信号将被静默丢弃。（如果信号正在被处理，则会传递给进程。）无论信号是如何发送的——例如，信号是由终端驱动程序生成，还是通过显式调用*kill()*发送——都会发生这种行为。

## 总结

会话和进程组（也称为作业）形成了一个两级层次的进程结构：会话是多个进程组的集合，进程组是多个进程的集合。会话领导者是创建会话的进程，使用*setsid()*来创建。类似地，进程组领导者是创建进程组的进程，使用*setpgid()*来创建。进程组中的所有成员共享相同的进程组 ID（与进程组领导者的进程 ID 相同），而构成会话的进程组中的所有进程共享相同的会话 ID（与会话领导者的进程 ID 相同）。每个会话可能有一个控制终端（`/dev/tty`），该终端在会话领导者打开终端设备时建立。打开控制终端还会使会话领导者成为该终端的控制进程。

会话和进程组是为了支持 shell 作业控制而定义的（尽管有时它们在应用程序中也有其他用途）。在作业控制下，shell 是会话领导者和控制终端的控制进程。由 shell 执行的每个作业（一个简单命令或管道）都会被创建为一个单独的进程组，shell 提供命令以在三种状态之间移动作业：在前台运行、在后台运行和在后台停止。

为了支持作业控制，终端驱动程序会维护一个控制终端的前台进程组（作业）的记录。当某些字符被输入时，终端驱动程序会向前台作业发送作业控制信号。这些信号会终止或停止前台作业。

终端前台作业的概念也被用来仲裁终端 I/O 请求。只有前台作业中的进程才能从控制终端读取数据。通过发送`SIGTTIN`信号，后台作业被阻止读取数据，默认操作是停止该作业。如果终端设置了`TOSTOP`，则通过发送`SIGTTOU`信号，后台作业也会被阻止向控制终端写入数据，默认操作是停止该作业。

当终端断开连接时，内核会向控制进程发送`SIGHUP`信号，以通知其发生了这种情况。这样的事件可能会引发连锁反应，导致`SIGHUP`信号被发送到许多其他进程。首先，如果控制进程是一个 shell（通常情况如此），那么在终止之前，shell 会向它所创建的每个进程组发送`SIGHUP`信号。其次，如果`SIGHUP`信号的传递导致控制进程终止，那么内核还会向控制终端的前台进程组中的所有成员发送`SIGHUP`信号。

一般来说，应用程序不需要关注作业控制信号。唯一的例外是当程序执行屏幕处理操作时。这类程序需要正确处理`SIGTSTP`信号，在进程挂起之前将终端属性重置为合理的值，并在应用程序恢复后，收到`SIGCONT`信号时恢复正确的（应用特定的）终端属性。

如果进程组的成员进程没有其他来自不同进程组的父进程，该进程组则被认为是孤儿进程组。孤儿进程组非常重要，因为没有外部进程可以同时监控该组中任何停止进程的状态，并且总是允许向这些停止的进程发送`SIGCONT`信号以重新启动它们。这样会导致这些停止的进程永远在系统中处于挂起状态。为避免这种可能性，当一个包含停止进程的进程组成为孤儿进程组时，该进程组的所有成员会收到一个`SIGHUP`信号，随后发送一个`SIGCONT`信号，通知它们已经成为孤儿进程组，并确保它们被重新启动。

#### 更多信息

第九章的[Stevens & Rago, 2005]涵盖了与本章类似的内容，并描述了登录过程中的步骤，以建立登录 Shell 的会话。*glibc* 手册中包含了关于作业控制及其在 Shell 中实现的详细说明。SUSv3 理由中有对会话、进程组和作业控制的广泛讨论。

## 练习

1.  假设父进程执行以下步骤：

    ```
    /* Call fork() to create a number of child processes, each of which
       remains in same process group as the parent */

    /* Sometime later... */
    signal(SIGUSR1, SIG_IGN);     /* Parent makes itself immune to SIGUSR1 */

    killpg(getpgrp(), SIGUSR1);   /* Send signal to children created earlier */
    ```

    这种应用设计可能会遇到什么问题？（考虑到 Shell 管道）该如何避免这个问题？

1.  编写一个程序验证父进程在子进程执行*exec()*之前可以更改其子进程的进程组 ID，但在子进程执行*exec()*之后则无法更改。

1.  编写一个程序验证从进程组领导者调用*setsid()*是否失败。

1.  修改示例 34-4（`disc_SIGHUP.c`）中的程序，验证如果控制进程在接收到`SIGHUP`后没有终止，那么内核不会将`SIGHUP`发送给前台进程组的成员。

1.  假设在示例 34-6 的信号处理程序中，将解除阻塞`SIGTSTP`信号的代码移动到处理程序的开始处。这样会产生什么潜在的竞态条件？

1.  编写一个程序验证当一个孤儿进程组中的进程尝试从控制终端*read()*时，*read()*会因为错误`EIO`而失败。

1.  编写一个程序验证，如果`SIGTTIN`、`SIGTTOU`或`SIGTSTP`信号发送到孤儿进程组的成员时，如果该信号会停止进程（即其处理方式为`SIG_DFL`），则该信号会被丢弃（即没有效果）；但是，如果该信号安装了处理程序，则信号会被传递。
