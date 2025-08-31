## 第二十一章：信号：信号处理程序

本章继续描述在上一章中开始的信号部分。它重点讨论信号处理程序，并扩展了第 20.4 节中开始的讨论。我们考虑的主题包括以下内容：

+   如何设计信号处理程序，这需要讨论可重入性和异步信号安全函数；

+   执行信号处理程序的正常返回的替代方法，特别是为此目的使用非本地跳转；

+   在备用栈上处理信号；

+   使用 *sigaction()* `SA_SIGINFO` 标志，允许信号处理程序获取有关触发其调用的信号的更多详细信息；

+   如何通过信号处理程序中断一个阻塞的系统调用，以及如何在需要时重新启动该调用。

## 设计信号处理程序

通常，编写简单的信号处理程序是更可取的。这样做的一个重要原因是减少产生竞争条件的风险。信号处理程序的两种常见设计如下：

+   信号处理程序设置一个全局标志并退出。主程序定期检查此标志，如果标志被设置，则采取适当的行动。（如果主程序无法执行这样的定期检查，因为它需要监视一个或多个文件描述符以查看是否可以进行 I/O 操作，那么信号处理程序也可以向一个专用管道写入一个字节，该管道的读取端包含在主程序监控的文件描述符中。我们在自管道技巧中展示了这个技术的一个示例。）

+   信号处理程序执行某种类型的清理，然后终止进程，或使用非本地跳转（从信号处理程序执行非本地跳转）来展开栈并将控制权返回到主程序中的预定位置。

在接下来的章节中，我们将探讨这些想法，以及在信号处理程序设计中重要的其他概念。

### 信号不是排队的（重访）

在信号掩码（阻塞信号传递）中，我们提到在执行信号处理程序时，信号的传递会被阻塞（除非我们在 *sigaction()* 中指定 `SA_NODEFER` 标志）。如果在处理程序执行期间再次生成该信号，则该信号将被标记为待处理，并在处理程序返回时传递。我们还已经提到，信号不会排队。如果在处理程序执行期间生成多次信号，则它仍然会被标记为待处理，并且只会在稍后传递一次。

信号可能以这种方式“消失”，这对我们设计信号处理器有重要影响。首先，我们无法可靠地计算信号生成的次数。此外，我们可能需要编写信号处理器代码，以应对可能发生多次与该信号对应的事件的情况。我们将在《为 SIGCHLD 建立处理器》中看到一个例子。

### 可重入函数与异步信号安全函数

并非所有的系统调用和库函数都可以在信号处理器中安全地调用。要理解原因，需要解释两个概念：可重入函数和异步信号安全函数。

#### 可重入函数和非可重入函数

要解释什么是可重入函数，我们需要首先区分单线程程序和多线程程序。经典的 UNIX 程序只有一个*执行线程*：CPU 处理程序中单一逻辑执行流的指令。在多线程程序中，同一个进程内有多个独立的并发逻辑执行流。

在第二十九章中，我们将看到如何显式地创建包含多个执行线程的程序。然而，多线程执行的概念对于使用信号处理器的程序也同样重要。因为信号处理器可能在程序的任何时刻异步中断程序的执行，主程序和信号处理器实际上形成了同一个进程内两个独立的（尽管不是并发的）执行线程。

如果一个函数能够被同一进程中的多个执行线程同时安全执行，则称该函数为*可重入*。在这个上下文中，“安全”意味着无论其他执行线程的状态如何，该函数都能达到预期的结果。

### 注意

SUSv3 对可重入函数的定义是：“当两个或多个线程调用时，其效果保证就像这些线程一个接一个按不确定顺序执行该函数，即使实际执行是交替进行的。”

如果一个函数更新了全局或静态数据结构，那么它可能是*不可重入*的。（一个仅使用局部变量的函数是保证可重入的。）如果两个函数调用（即两个线程同时执行）试图同时更新相同的全局变量或数据结构，那么这些更新很可能会相互干扰并产生错误结果。例如，假设一个线程正在更新一个链表数据结构以添加一个新的列表项，而另一个线程也试图更新同一个链表。由于向列表中添加新项需要更新多个指针，如果另一个线程中断这些步骤并更新相同的指针，就会导致混乱。

这样的可能性实际上在标准 C 库中很常见。例如，我们在《*malloc()* 和 *free()* 的实现》 and free()")中已经提到，*malloc()* 和 *free()* 会维护一个已经释放的内存块的链表，这些内存块可以从堆中重新分配。如果主程序中的 *malloc()* 调用被一个信号处理程序中也调用 *malloc()* 的中断，那么这个链表可能会被破坏。因此，*malloc()* 系列函数，以及使用它们的其他库函数，是不可重入的。

其他一些库函数之所以不可重入，是因为它们通过静态分配的内存返回信息。这类函数的例子（本书中其他地方有描述）包括 *crypt()*、*getpwnam()*、*gethostbyname()* 和 *getservbyname()*。如果信号处理程序也使用这些函数中的某个函数，那么它将覆盖先前从主程序中调用相同函数返回的信息（反之亦然）。

如果一个函数使用静态数据结构进行内部管理，它也可能是不可重入的。最明显的这类函数是 *stdio* 库的成员（*printf()*、*scanf()* 等），它们会更新用于缓冲输入输出的内部数据结构。因此，当在信号处理程序中使用 *printf()* 时，如果该处理程序在主程序执行 *printf()* 或其他 *stdio* 函数调用的过程中被中断，我们有时会看到奇怪的输出，甚至程序崩溃或数据损坏。

即使我们没有使用不可重入的库函数，重入问题仍然可能相关。如果信号处理程序更新了程序员定义的全局数据结构，而这些数据结构也在主程序中被更新，那么我们可以说信号处理程序在主程序中的作用是不可重入的。

如果一个函数是不可重入的，那么它的手册页通常会明确或隐含地指出这一点。特别要注意那些说明该函数使用或返回静态分配变量信息的语句。

#### 示例程序

示例 21-1 和信号处理程序中调用一个非重入函数") 展示了 *crypt()* 函数的非重入特性（密码加密和用户认证）。作为命令行参数，该程序接受两个字符串。程序执行以下步骤：

1.  调用 *crypt()* 对第一个命令行参数中的字符串进行加密，并使用 *strdup()* 将该字符串复制到一个单独的缓冲区中。

1.  为 `SIGINT`（通过键入 *Control-C* 生成）设置一个处理程序。该处理程序调用 *crypt()* 对第二个命令行参数中的字符串进行加密。

1.  进入一个无限的 `for` 循环，使用 *crypt()* 对第一个命令行参数中的字符串进行加密，并检查返回的字符串是否与第 1 步中保存的字符串相同。

在没有信号的情况下，第 3 步中的字符串始终匹配。然而，如果 `SIGINT` 信号到达并且信号处理程序的执行在 `for` 循环中执行 *crypt()* 调用之后，但在检查字符串是否匹配之前中断了主程序，那么主程序将报告不匹配。当我们运行该程序时，会看到如下结果：

```
$ `./non_reentrant abc def`
*Repeatedly type Control-C to generate* SIGINT
Mismatch on call 109871 (mismatch=1 handled=1)
Mismatch on call 128061 (mismatch=2 handled=2)
*Many lines of output removed*
Mismatch on call 727935 (mismatch=149 handled=156)
Mismatch on call 729547 (mismatch=150 handled=157)
*Type Control-\ to generate* SIGQUIT
Quit (core dumped)
```

比较上述输出中的 *mismatch* 和 *handled* 值，我们看到在大多数调用信号处理程序的情况下，它会覆盖在 *crypt()* 调用和 *main()* 中字符串比较之间的静态分配缓冲区。

示例 21-1. 从 *main()* 和信号处理程序中调用一个非重入函数

```
`signals/nonreentrant.c`
#define _XOPEN_SOURCE 600
#include <unistd.h>
#include <signal.h>
#include <string.h>
#include "tlpi_hdr.h"

static char *str2;              /* Set from argv[2] */
static int handled = 0;         /* Counts number of calls to handler */

static void
handler(int sig)
{

    crypt(str2, "xx");
    handled++;
}
int
main(int argc, char *argv[])
{
    char *cr1;
    int callNum, mismatch;
    struct sigaction sa;

    if (argc != 3)
        usageErr("%s str1 str2\n", argv[0]);

    str2 = argv[2];                      /* Make argv[2] available to handler */
    cr1 = strdup(crypt(argv[1], "xx"));  /* Copy statically allocated string
                                            to another buffer */
    if (cr1 == NULL)
        errExit("strdup");

    sigemptyset(&sa.sa_mask);
    sa.sa_flags = 0;
    sa.sa_handler = handler;
    if (sigaction(SIGINT, &sa, NULL) == -1)
        errExit("sigaction");

    /* Repeatedly call crypt() using argv[1]. If interrupted by a
       signal handler, then the static storage returned by crypt()
       will be overwritten by the results of encrypting argv[2], and
       strcmp() will detect a mismatch with the value in 'cr1'. */

    for (callNum = 1, mismatch = 0; ; callNum++) {
        if (strcmp(crypt(argv[1], "xx"), cr1) != 0) {
            mismatch++;
            printf("Mismatch on call %d (mismatch=%d handled=%d)\n",
                    callNum, mismatch, handled);
        }
    }
}
     `signals/nonreentrant.c`
```

#### 标准异步信号安全函数

*异步信号安全* 函数是指在从信号处理程序中调用时，实施保证其安全的函数。一个函数是异步信号安全的，要么因为它是可重入的，要么因为它不会被信号处理程序中断。

表 21-1 列出了各个标准要求的异步信号安全函数。在此表中，函数名称后没有 *v2* 或 *v3* 的函数是 POSIX.1-1990 中指定为异步信号安全的。SUSv2 将标记为 *v2* 的函数添加到列表中，而标记为 *v3* 的函数是 SUSv3 添加的。各个 UNIX 实现可能会使其他函数变为异步信号安全，但所有符合标准的 UNIX 实现必须确保至少这些函数是异步信号安全的（如果它们由实现提供；并非所有这些函数都在 Linux 上提供）。

SUSv4 对 表 21-1 做了以下更改：

+   以下函数已被移除：*fpathconf()*, *pathconf()*, 和 *sysconf()*。

+   以下函数被添加：*execl()*，*execv()*，*faccessat()*，*fchmodat()*，*fchownat()*，*fexecve()*，*fstatat()*，*futimens()*，*linkat()*，*mkdirat()*，*mkfifoat()*，*mknod()*，*mknodat()*，*openat()*，*readlinkat()*，*renameat()*，*symlinkat()*，*unlinkat()*，*utimensat()*，以及*utimes()*。

表 21-1. POSIX.1-1990、SUSv2 和 SUSv3 要求具备异步信号安全的函数

| *_Exit()(v3)**_exit()**abort() (v3)**accept() (v3)**access()**aio_error() (v2)**aio_return()(v2)**aio_suspend() (v2)**alarm()**bind()(v3)**cfgetispeed()**cfgetospeed()**cfsetispeed()**cfsetospeed()**chdir()**chmod()**chown()**clock_gettime() (v2)**close()**connect() (v3)**creat()**dup()**dup2()**execle()**execve()**fchmod() (v3)**fchown() (v3)**fcntl()**fdatasync() (v2)**fork()**fpathconf() (v2)**fstat()**fsync() (v2)**ftruncate() (v3)**getegid()**geteuid()**getgid()**getgroups()**getpeername() (v3)**getpgrp()* | *getpid()**getppid()**getsockname() (v3)**getsockopt() (v3)**getuid()**kill()**link()**listen() (v3)**lseek()**lstat() (v3)**mkdir()**mkfifo()**open()**pathconf()**pause()**pipe()**poll() (v3)**posix_trace_event() (v3)**pselect() (v3)**raise() (v2)**read()**readlink() (v3)**recv() (v3)**recvfrom() (v3)**recvmsg() (v3)**rename()**rmdir()**select() (v3)**sem_post() (v2)**send() (v3)**sendmsg() (v3)**sendto() (v3)**setgid()**setpgid()**setsid()**setsockopt() (v3)**setuid()**shutdown() (v3)**sigaction()**sigaddset()* | *sigdelset()**sigemptyset()**sigfillset()**sigismember()**signal() (v2)**sigpause() (v2)**sigpending()**sigprocmask()**sigqueue() (v2)**sigset() (v2)**sigsuspend()**sleep()**socket() (v3)**sockatmark() (v3)**socketpair() (v3)**stat()**symlink() (v3)**sysconf()**tcdrain()**tcflow()**tcflush()**tcgetattr()**tcgetpgrp()**tcsendbreak()**tcsetattr()**tcsetpgrp()**time()**timer_getoverrun() (v2)**timer_gettime() (v2)**timer_settime() (v2)**times()**umask()**uname()**unlink()**utime()**wait()**waitpid()**write()* |
| --- | --- | --- |

SUSv3 指出，所有未在表 21-1 中列出的函数都被认为与信号处理相关的不安全函数，但指出只有在信号处理程序的调用中断了不安全函数的执行，并且处理程序本身也调用了不安全函数时，函数才被认为是不安全的。换句话说，在编写信号处理程序时，我们有两种选择：

+   确保信号处理程序本身的代码是可重入的，并且仅调用异步信号安全的函数。

+   在执行主程序中调用不安全函数或与由信号处理程序更新的全局数据结构一起工作的代码时，阻止信号的传递。

第二种方法的问题在于，在一个复杂的程序中，确保信号处理程序在调用不安全函数时永远不会中断主程序可能会很困难。因此，上述规则通常简化为我们不能在信号处理程序中调用不安全函数的声明。

### 注意

如果我们设置相同的处理程序函数来处理多个不同的信号，或者使用`SA_NODEFER`标志来设置*sigaction()*，那么一个处理程序可能会中断自身。因此，如果处理程序更新全局（或静态）变量，即使这些变量没有被主程序使用，它也可能变得不可重入。

#### 在信号处理程序中使用*errno*

因为它们可能会更新*errno*，使用表 21-1 中列出的函数仍然可能使信号处理程序变得不可重入，因为它们可能会覆盖由主程序中调用的函数设置的*errno*值。解决方法是在进入使用表 21-1 中任何函数的信号处理程序时保存*errno*的值，并在从处理程序退出时恢复*errno*值，如下例所示：

```
void
handler(int sig)
{
    int savedErrno;

    savedErrno = errno;

    /* Now we can execute a function that might modify errno */

    errno = savedErrno;
}
```

#### 本书示例程序中不安全函数的使用

虽然*printf()*不是异步信号安全的，但我们在本书中的各种示例程序中仍在信号处理程序中使用它。我们这样做是因为*printf()*提供了一种简单而简洁的方式来演示信号处理程序已经被调用，并显示处理程序中相关变量的内容。出于类似的原因，我们偶尔会在信号处理程序中使用一些其他不安全的函数，包括其他*stdio*函数和*strsignal()*。

真实世界的应用应该避免在信号处理程序中调用非异步信号安全的函数。为了明确这一点，本书中的每个使用这些函数的信号处理程序都会标注一个注释，表明这种用法是不安全的：

```
printf("Some message\n");           /* UNSAFE */
```

### 全局变量和*sig_atomic_t*数据类型

尽管存在重入问题，在主程序和信号处理程序之间共享全局变量可能是有用的。只要主程序正确处理信号处理程序可能随时更改全局变量的情况，这种做法是安全的。例如，一种常见的设计是让信号处理程序的唯一操作是设置一个全局标志。这个标志会被主程序定期检查，然后根据信号的传递采取适当的行动（并清除标志）。当全局变量通过信号处理程序以这种方式访问时，我们应始终使用`volatile`属性声明它们（见执行非局部跳转：*setjmp()* 和 *long jmp()* 和 long jmp()")），以防止编译器进行优化，导致变量被存储在寄存器中。

读取和写入全局变量可能涉及多条机器语言指令，并且信号处理程序可能在这些指令序列的中间中断主程序。（我们称访问该变量为*非原子性*的。）因此，C 语言标准和 SUSv3 规定了一种整数数据类型，*sig_atomic_t*，它保证对该类型的读取和写入是原子的。因此，共享在主程序和信号处理程序之间的全局标志变量应按如下方式声明：

```
volatile sig_atomic_t flag;
```

我们展示了在示例 22-5")和示例程序中使用*sig_atomic_t*数据类型的例子。

请注意，C 的自增（`++`）和自减（`--`）运算符不在*sig_atomic_t*提供的保证范围内。在某些硬件架构中，这些操作可能不是原子性的（有关更多细节，请参阅保护对共享变量的访问：互斥锁）。我们唯一被保证安全地允许对*sig_atomic_t*变量执行的操作是，在信号处理程序中设置它，并在主程序中检查它（或反之）。

C99 和 SUSv3 规定实现应定义两个常量（在`<stdint.h>`中），`SIG_ATOMIC_MIN`和`SIG_ATOMIC_MAX`，用于定义可以分配给*sig_atomic_t*类型变量的值范围。标准要求，如果*sig_atomic_t*表示为带符号值，则该范围至少应为-127 到 127；如果表示为无符号值，则应为 0 到 255。在 Linux 上，这两个常量等同于带符号 32 位整数的负数和正数极限。

## 其他终止信号处理程序的方法

到目前为止，我们所查看的所有信号处理程序通过返回到主程序来完成。然而，简单地从信号处理程序返回有时并不理想，或者在某些情况下，甚至没有用处。（当我们讨论硬件生成的信号时，在第 22.4 节中会看到一个返回信号处理程序无用的例子。）

还有各种其他终止信号处理程序的方法：

+   使用 *_exit()* 终止进程。在此之前，处理程序可以执行一些清理操作。请注意，我们不能使用 *exit()* 来终止信号处理程序，因为它不是在 表 21-1 中列出的安全函数之一。它不安全，因为它会在调用 *_exit()* 之前刷新 *stdio* 缓冲区，如第 25.1 节所述。

+   使用 *kill()* 或 *raise()* 发送一个终止进程的信号（即一个默认操作是终止进程的信号）。

+   从信号处理程序执行非局部跳转。

+   使用 *abort()* 函数通过核心转储终止进程。

这两种选项的最后两个将在以下章节中进一步详细描述。

### 从信号处理程序执行非局部跳转

执行非局部跳转：*setjmp()* 和 *longjmp()* 和 longjmp()") 描述了使用 *setjmp()* 和 *longjmp()* 从函数到其调用者之间执行非局部跳转。我们也可以在信号处理程序中使用这种技术。这提供了一种在硬件异常（例如内存访问错误）引发信号后进行恢复的方法，并且还允许我们捕获信号并将控制权返回到程序中的特定位置。例如，在接收到 `SIGINT` 信号（通常由输入 *Control-C* 生成）时，shell 执行非局部跳转，将控制权返回到其主输入循环（从而读取新命令）。

然而，使用标准的 *longjmp()* 函数从信号处理程序退出时存在一个问题。我们之前提到过，在进入信号处理程序时，内核会自动将调用信号以及 *act.sa_mask* 字段中指定的任何信号添加到进程信号屏蔽中，并在处理程序正常返回时将这些信号从屏蔽中移除。

如果我们使用 *longjmp()* 退出信号处理程序，信号屏蔽字会发生什么？答案取决于特定 UNIX 实现的血统。在 System V 下，*longjmp()* 不会恢复信号屏蔽字，因此在退出处理程序时，阻塞的信号不会被解除阻塞。Linux 遵循 System V 的行为。（这通常不是我们想要的，因为它会将导致调用处理程序的信号保持阻塞。）在 BSD 派生的实现中，*setjmp()* 将信号屏蔽字保存在其 *env* 参数中，并且 *longjmp()* 会恢复保存的信号屏蔽字。（BSD 派生的实现还提供了两个其他函数，*_setjmp()* 和 *_longjmp()*，它们遵循 System V 语义。）换句话说，我们不能在移植时使用 *longjmp()* 退出信号处理程序。

### 注意

如果我们在编译程序时定义了 `_BSD_SOURCE` 特性测试宏，那么 (*glibc*) *setjmp()* 将遵循 BSD 语义。

由于这两个主要 UNIX 变种之间的差异，POSIX.1-1990 选择不指定 *setjmp()* 和 *longjmp()* 处理信号屏蔽字的方式。相反，它定义了一对新函数 *sigsetjmp()* 和 *siglongjmp()*，这些函数在执行非局部跳转时提供对信号屏蔽字的显式控制。

```
#include <setjmp.h>

int `sigsetjmp`(sigjmp_buf *env*, int *savesigs*);
```

### 注意

初始调用时返回 0，通过 *siglongjmp()* 返回时返回非零值

```
void `siglongjmp`(sigjmp_buf *env*, int *val*);
```

*sigsetjmp()* 和 *siglongjmp()* 函数的操作类似于 *setjmp()* 和 *longjmp()*。唯一的不同在于 *env* 参数的类型（*sigjmp_buf* 替代了 *jmp_buf*）以及 *sigsetjmp()* 的额外 *savesigs* 参数。如果 *savesigs* 非零，则在调用 *sigsetjmp()* 时，当前进程的信号屏蔽字将保存在 *env* 中，并在稍后的 *siglongjmp()* 调用时通过相同的 *env* 参数恢复。如果 *savesigs* 为 0，则信号屏蔽字不会被保存和恢复。

*longjmp()* 和 *siglongjmp()* 函数没有列在 表 21-1 中。这是因为在执行非局部跳转后调用任何非异步信号安全的函数，存在与从信号处理程序中调用该函数相同的风险。此外，如果信号处理程序在主程序更新数据结构的过程中被中断，并且处理程序通过非局部跳转退出，那么未完成的更新可能会导致数据结构处于不一致的状态。一个可以帮助避免问题的技术是使用 *sigprocmask()* 暂时阻塞信号，以便在执行敏感更新时保护数据。

#### 示例程序

示例 21-2 演示了两种非局部跳转在信号屏蔽处理上的区别。该程序为 `SIGINT` 建立了一个信号处理程序。程序设计允许使用 *setjmp()* 加 *longjmp()* 或 *sigsetjmp()* 加 *siglongjmp()* 来退出信号处理程序，具体取决于程序是否在编译时定义了 `USE_SIGSETJMP` 宏。该程序在进入信号处理程序时以及非局部跳转将控制从处理程序转回主程序后，显示信号屏蔽的当前设置。

当我们构建程序，使得*longjmp()*用于退出信号处理程序时，这是我们运行程序时看到的输出：

```
$ `make -s sigmask_longjmp`         *Default compilation causes*
 *setjmp()* *to be used*
$ `./sigmask_longjmp`
Signal mask at startup:
                <empty signal set>
Calling setjmp()
*Type Control-C to generate* SIGINT
Received signal 2 (Interrupt), signal mask is:
                2 (Interrupt)
After jump from handler, signal mask is:
                2 (Interrupt)
*(At this point, typing Control-C again has no effect, since* SIGINT *is blocked)*
*Type Control-\ to kill the program*
Quit
```

从程序输出中，我们可以看到，在从信号处理程序执行*longjmp()*后，信号屏蔽保持为进入信号处理程序时所设置的值。

### 注意

在上面的 shell 会话中，我们使用本书随附的源代码分发包中的 makefile 构建了程序。* -s* 选项告诉*make*不要回显它正在执行的命令。我们使用此选项来避免使会话日志杂乱无章。（[Mecklenburg, 2005] 提供了关于 GNU *make* 程序的描述。）

当我们编译相同的源文件，构建一个使用 *siglongjmp()* 来退出处理程序的可执行文件时，我们看到如下输出：

```
$ `make -s sigmask_siglongjmp`      *Compiles using* *cc -DUSE_SIGSETJMP*
$ `./sigmask_siglongjmp`
Signal mask at startup:
                <empty signal set>
Calling sigsetjmp()
*Type Control-C*
Received signal 2 (Interrupt), signal mask is:
                2 (Interrupt)
After jump from handler, signal mask is:
                <empty signal set>
```

此时，`SIGINT`没有被阻塞，因为 *siglongjmp()* 恢复了信号屏蔽到原始状态。接下来，我们再次输入 *Control-C*，以便信号处理程序再次被调用：

```
*Type Control-C*
Received signal 2 (Interrupt), signal mask is:
                2 (Interrupt)
After jump from handler, signal mask is:
                <empty signal set>
*Type Control-\ to kill the program*
Quit
```

从上述输出中，我们可以看到 *siglongjmp()* 将信号屏蔽恢复为在 *sigsetjmp()* 调用时的值（即一个空的信号集）。

示例 21-2 还演示了一种用于信号处理程序的有用技术，该处理程序执行非局部跳转。由于信号可以在任何时候产生，它实际上可能在目标跳转尚未由 *sigsetjmp()*（或 *setjmp()*) 设置之前就发生。为了防止这种情况（这会导致处理程序使用未初始化的 *env* 缓冲区执行非局部跳转），我们使用一个守卫变量 *canJump* 来指示 *env* 缓冲区是否已初始化。如果 *canJump* 为 false，则处理程序会简单地返回，而不是执行非局部跳转。另一种方法是安排程序代码，使得调用 *sigsetjmp()*（或 *setjmp()*）发生在建立信号处理程序之前。然而，在复杂程序中，确保这两个步骤按顺序执行可能比较困难，使用守卫变量可能会更简单。

请注意，在示例 21-2 中，使用`#ifdef`是以符合标准的方式编写程序的最简单方法。特别是，我们无法将`#ifdef`替换为以下运行时检查：

```
if (useSiglongjmp)
    s = sigsetjmp(senv, 1);
else
    s = setjmp(env);
if (s == 0)
    ...
```

这是不允许的，因为 SUSv3 不允许在赋值语句中使用*setjmp()*和*sigsetjmp()*（参见执行非本地跳转：*setjmp()*和*long jmp()*和 long jmp()")）。

示例 21-2. 从信号处理程序执行非本地跳转

```
`signals/sigmask_longjmp.c`
#define _GNU_SOURCE     /* Get strsignal() declaration from <string.h> */
#include <string.h>
#include <setjmp.h>
#include <signal.h>
#include "signal_functions.h"           /* Declaration of printSigMask() */
#include "tlpi_hdr.h"

static volatile sig_atomic_t canJump = 0;
                        /* Set to 1 once "env" buffer has been
                           initialized by [sig]setjmp() */
#ifdef USE_SIGSETJMP
static sigjmp_buf senv;
#else
static jmp_buf env;
#endif

static void
handler(int sig)
{
    /* UNSAFE: This handler uses non-async-signal-safe functions
       (printf(), strsignal(), printSigMask(); see Section 21.1.2) */

    printf("Received signal %d (%s), signal mask is:\n", sig,
            strsignal(sig));
    printSigMask(stdout, NULL);

    if (!canJump) {
        printf("'env' buffer not yet set, doing a simple return\n");
        return;
    }

#ifdef USE_SIGSETJMP
    siglongjmp(senv, 1);
#else
    longjmp(env, 1);
#endif
}

int
main(int argc, char *argv[])
{
    struct sigaction sa;

    printSigMask(stdout, "Signal mask at startup:\n");

    sigemptyset(&sa.sa_mask);
    sa.sa_flags = 0;
    sa.sa_handler = handler;
    if (sigaction(SIGINT, &sa, NULL) == -1)
        errExit("sigaction");

#ifdef USE_SIGSETJMP
    printf("Calling sigsetjmp()\n");
    if (sigsetjmp(senv, 1) == 0)
#else
    printf("Calling setjmp()\n");
    if (setjmp(env) == 0)
#endif
        canJump = 1;                    /* Executed after [sig]setjmp() */

    else                                /* Executed after [sig]longjmp() */
        printSigMask(stdout, "After jump from handler, signal mask is:\n" );

    for (;;)                            /* Wait for signals until killed */
        pause();
}
      `signals/sigmask_longjmp.c`
```

### 异常终止进程：*abort()*

*abort()*函数终止调用进程并生成核心转储。

```
#include <stdlib.h>

void `abort`(void);
```

*abort()*函数通过触发`SIGABRT`信号终止调用进程。`SIGABRT`的默认操作是生成核心转储文件并终止进程。然后可以在调试器中使用核心转储文件检查程序在*abort()*调用时的状态。

SUSv3 要求*abort()*覆盖阻塞或忽略`SIGABRT`信号的效果。此外，SUSv3 规定，除非进程通过一个不会返回的处理程序捕获信号，否则*abort()*必须终止进程。最后这一点值得深思。在终止信号处理程序的其他方法中描述的终止信号处理程序的方法中，相关的方法是使用非本地跳转退出处理程序。如果这样做，*abort()*的效果将被取消；否则，*abort()*始终会终止进程。在大多数实现中，终止是通过以下方式保证的：如果进程在第一次触发`SIGABRT`信号后仍未终止（即，处理程序捕获信号并返回，导致*abort()*的执行继续），则*abort()*会将`SIGABRT`的处理恢复为`SIG_DFL`，并触发第二次`SIGABRT`，从而保证终止进程。

如果*abort()*确实成功终止进程，它还会刷新并关闭*stdio*流。

在示例 3-3 中提供了*abort()*使用的示例，在解析数字命令行参数的函数中也有示例。

## 在备用栈上处理信号：*sigaltstack()*

通常，当信号处理程序被调用时，内核会在进程栈上为其创建一个栈帧。然而，如果进程试图将栈扩展到超过最大可能大小，这可能不可行。例如，这可能是由于栈增长过大，导致其遇到映射内存区域（虚拟内存中的共享内存位置）或向上增长的堆，或者达到 `RLIMIT_STACK` 资源限制（具体资源限制详情）。

当进程试图将其栈扩展到超过最大可能大小时，内核会为该进程生成 `SIGSEGV` 信号。然而，由于栈空间已耗尽，内核无法为程序可能已建立的任何 `SIGSEGV` 处理程序创建栈帧。因此，处理程序不会被调用，进程会被终止（`SIGSEGV` 的默认行为）。

如果我们需要确保在这些情况下处理 `SIGSEGV` 信号，可以执行以下操作：

1.  分配一块内存区域，称为 *备用信号栈*，用于信号处理程序的栈帧。

1.  使用 *sigaltstack()* 系统调用通知内核存在备用信号栈。

1.  在建立信号处理程序时，指定 `SA_ONSTACK` 标志，以告诉内核此处理程序的栈帧应创建在备用栈上。

*sigaltstack()* 系统调用既建立备用信号栈，又返回有关已建立的备用信号栈的相关信息。

```
#include <signal.h>

int `sigaltstack`(const stack_t **sigstack*, stack_t **old_sigstack*);
```

### 注意

成功时返回 0，出错时返回 -1

*sigstack* 参数指向一个结构，指定新备用信号栈的位置和属性。*old_sigstack* 参数指向一个结构，用于返回有关先前建立的备用信号栈的信息（如果存在）。这两个参数中的任何一个都可以指定为 `NULL`。例如，我们可以通过指定 *sigstack* 参数为 `NULL`，在不更改现有备用信号栈的情况下获取相关信息。否则，这些参数中的每一个都指向以下类型的结构：

```
typedef struct {
    void  *ss_sp;         /* Starting address of alternate stack */
    int    ss_flags;      /* Flags: SS_ONSTACK, SS_DISABLE */
    size_t ss_size;       /* Size of alternate stack */
} stack_t;
```

*ss_sp* 和 *ss_size* 字段指定备用信号栈的大小和位置。在实际使用备用信号栈时，内核会自动将 *ss_sp* 中给定的值对齐到适合硬件架构的地址边界。

通常，备用信号栈要么是静态分配的，要么是动态分配在堆上的。SUSv3 指定常量 `SIGSTKSZ` 作为确定备用栈大小的典型值，`MINSIGSTKSZ` 作为调用信号处理程序所需的最小大小。在 Linux/x86-32 上，这些常量的值分别定义为 8192 和 2048。

内核不会调整备用信号栈的大小。如果栈溢出我们为其分配的空间，将会导致混乱（例如，栈外的变量被覆盖）。这通常不是一个问题——因为我们通常使用备用信号栈来处理标准栈溢出的特殊情况，通常只会为栈分配一两个帧。`SIGSEGV`处理程序的工作是执行一些清理工作并终止进程，或者使用非局部跳转解开标准栈。

*ss_flags*字段包含以下值之一：

`SS_ONSTACK`

如果在获取当前已建立的备用信号栈信息（*old_sigstack*）时设置此标志，表示进程当前正在备用信号栈上执行。在进程已经在备用信号栈上运行时，尝试建立新的备用信号栈将导致*sigaltstack()*返回错误（`EPERM`）。

`SS_DISABLE`

在*old_sigstack*中返回的这个标志表示当前没有建立备用信号栈。当在*sigstack*中指定时，它会禁用当前已建立的备用信号栈。

示例 21-3")展示了备用信号栈的建立和使用。在建立备用信号栈和`SIGSEGV`的处理程序后，这个程序调用一个无限递归的函数，导致栈溢出并向进程发送`SIGSEGV`信号。运行该程序时，我们看到如下输出：

```
$ `ulimit -s unlimited`
$ `./t_sigaltstack`
Top of standard stack is near 0xbffff6b8
Alternate stack is at          0x804a948-0x804cfff
Call    1 - top of stack near 0xbff0b3ac
Call    2 - top of stack near 0xbfe1714c
*Many intervening lines of output removed*
Call 2144 - top of stack near 0x4034120c
Call 2145 - top of stack near 0x4024cfac
Caught signal 11 (Segmentation fault)
Top of handler stack near      0x804c860
```

在这个 shell 会话中，使用*ulimit*命令移除了任何可能在 shell 中设置的`RLIMIT_STACK`资源限制。我们将在第 36.3 节解释这个资源限制。

示例 21-3. 使用*sigaltstack()*

```
`signals/t_sigaltstack.c`
#define _GNU_SOURCE          /* Get strsignal() declaration from <string.h> */
#include <string.h>
#include <signal.h>
#include "tlpi_hdr.h"

static void
sigsegvHandler(int sig)
{
    int x;

    /* UNSAFE: This handler uses non-async-signal-safe functions
       (printf(), strsignal(), fflush(); see Section 21.1.2) */

    printf("Caught signal %d (%s)\n", sig, strsignal(sig));
    printf("Top of handler stack near     %10p\n", (void *) &x);
    fflush(NULL);

    _exit(EXIT_FAILURE);                /* Can't return after SIGSEGV */
}

static void             /* A recursive function that overflows the stack */
overflowStack(int callNum)
{
    char a[100000];                     /* Make this stack frame large */

    printf("Call %4d - top of stack near %10p\n", callNum, &a[0]);
    overflowStack(callNum+1);
}

int
main(int argc, char *argv[])
{
    stack_t sigstack;
    struct sigaction sa;
    int j;

    printf("Top of standard stack is near %10p\n", (void *) &j);

    /* Allocate alternate stack and inform kernel of its existence */

    sigstack.ss_sp = malloc(SIGSTKSZ);
    if (sigstack.ss_sp == NULL)
        errExit("malloc");

    sigstack.ss_size = SIGSTKSZ;
    sigstack.ss_flags = 0;
    if (sigaltstack(&sigstack, NULL) == -1)
        errExit("sigaltstack");
    printf("Alternate stack is at         %10p-%p\n",
            sigstack.ss_sp, (char *) sbrk(0) - 1);

    sa.sa_handler = sigsegvHandler;     /* Establish handler for SIGSEGV */
    sigemptyset(&sa.sa_mask);
    sa.sa_flags = SA_ONSTACK;           /* Handler uses alternate stack */
    if (sigaction(SIGSEGV, &sa, NULL) == -1)
        errExit("sigaction");

    overflowStack(1);
}
      `signals/t_sigaltstack.c`
```

## `SA_SIGINFO` 标志

在使用*sigaction()*建立处理程序时设置`SA_SIGINFO`标志，可以使处理程序在信号传递时获取附加的信号信息。为了获取这些信息，我们必须按如下方式声明处理程序：

```
void handler(int sig, siginfo_t *siginfo, void *ucontext);
```

第一个参数，*sig*，是信号号码，类似于标准信号处理程序中的信号号码。第二个参数，*siginfo*，是一个结构体，用于提供有关信号的附加信息。我们将在下文描述这个结构。最后一个参数，*ucontext*，也将在下文描述。

由于上述信号处理程序与标准信号处理程序的原型不同，C 语言的类型规则意味着我们不能使用*sigaction*结构的*sa_handler*字段来指定处理程序的地址。相反，我们必须使用一个替代字段：*sa_sigaction*。换句话说，*sigaction*结构的定义比在改变信号处理：*sigaction*()")中显示的更为复杂。完整的结构定义如下：

```
struct sigaction {
    union {
        void (*sa_handler)(int);
        void (*sa_sigaction)(int, siginfo_t *, void *);
    } __sigaction_handler;
    sigset_t   sa_mask;
    int        sa_flags;
    void     (*sa_restorer)(void);
};
/* Following defines make the union fields look like simple fields
   in the parent structure */
#define sa_handler __sigaction_handler.sa_handler
#define sa_sigaction __sigaction_handler.sa_sigaction
```

*sigaction*结构使用联合体将*sa_sigaction*和*sa_handler*字段组合起来。（大多数其他 UNIX 实现也类似地使用联合体来实现这一目的。）使用联合体是可能的，因为在对*sigaction()*的特定调用中，仅需要其中一个字段。（然而，如果我们天真地期望能够独立设置*sa_handler*和*sa_sigaction*字段，可能会导致奇怪的错误，尤其是在我们在多个*sigaction()*调用中重用同一个*sigaction*结构来为不同的信号建立处理程序时。）

这是使用`SA_SIGINFO`建立信号处理程序的一个示例：

```
struct sigaction act;

sigemptyset(&act.sa_mask);
act.sa_sigaction = handler;
act.sa_flags = SA_SIGINFO;

if (sigaction(SIGINT, &act, NULL) == -1)
    errExit("sigaction");
```

`SA_SIGINFO`标志的完整使用示例，请参见示例 22-3（第 462 页）和示例 23-5（第 500 页）。

#### *siginfo_t*结构

使用`SA_SIGINFO`建立的信号处理程序中作为第二个参数传递的*siginfo_t*结构具有以下形式：

```
typedef struct {
    int     si_signo;         /* Signal number */
    int     si_code;          /* Signal code */
    int     si_trapno;        /* Trap number for hardware-generated signal
                                 (unused on most architectures) */
    union sigval si_value;    /* Accompanying data from sigqueue() */
    pid_t   si_pid;           /* Process ID of sending process */
    uid_t   si_uid;           /* Real user ID of sender */
    int     si_errno;         /* Error number (generally unused) */
    void   *si_addr;          /* Address that generated signal

                                 (hardware-generated signals only) */

    int     si_overrun;       /* Overrun count (Linux 2.6, POSIX timers) */
    int     si_timerid;       /* (Kernel-internal) Timer ID
                                 (Linux 2.6, POSIX timers) */
    long    si_band;          /* Band event (SIGPOLL/SIGIO) */
    int     si_fd;            /* File descriptor (SIGPOLL/SIGIO) */
    int     si_status;        /* Exit status or signal (SIGCHLD) */
    clock_t si_utime;         /* User CPU time (SIGCHLD) */
    clock_t si_stime;         /* System CPU time (SIGCHLD) */
} siginfo_t;
```

必须定义`_POSIX_C_SOURCE`功能测试宏，并且其值必须大于或等于 199309，以便使*siginfo_t*结构的声明在`<signal.h>`中可见。

在 Linux 上，与大多数 UNIX 实现一样，*siginfo_t*结构中的许多字段被组合成一个联合体，因为并非所有字段都需要用于每个信号。（有关详细信息，请参见`<bits/siginfo.h>`。）

进入信号处理程序时，*siginfo_t*结构的字段设置如下：

*si_signo*

该字段对所有信号都设置。它包含导致调用处理程序的信号的编号——即与传递给处理程序的*sig*参数相同的值。

*si_code*

该字段对所有信号都设置。它包含一个代码，提供有关信号来源的进一步信息，如表 21-2 所示。

*si_value*

该字段包含通过*sigqueue()*发送的信号的附加数据。我们在排队实时信号的数量限制中描述了*sigqueue()*。

*si_pid*

对于通过*kill()*或*sigqueue()*发送的信号，该字段设置为发送进程的进程 ID。

*si_uid*

对于通过 *kill()* 或 *sigqueue()* 发送的信号，此字段设置为发送进程的实际用户 ID。系统提供发送进程的实际用户 ID，因为它比提供有效用户 ID 更具信息性。考虑发送信号时的权限规则，如发送信号：*kill()*")中所述：如果有效用户 ID 授予发送者发送信号的权限，那么该用户 ID 必须是 0（即具有特权的进程），或者与接收进程的实际用户 ID 或保存的设置用户 ID 相同。在这种情况下，对于接收者来说，知道发送者的实际用户 ID 可能会有帮助，因为它可能与有效用户 ID 不同（例如，如果发送者是设置用户 ID 的程序）。

*si_errno*

如果此字段设置为非零值，则它包含一个错误号（类似于 *errno*），用于标识信号的原因。此字段通常在 Linux 上未使用。

*si_addr*

此字段仅在硬件生成的 `SIGBUS`、`SIGSEGV`、`SIGILL` 和 `SIGFPE` 信号的情况下设置。对于 `SIGBUS` 和 `SIGSEGV` 信号，此字段包含导致无效内存引用的地址。对于 `SIGILL` 和 `SIGFPE` 信号，此字段包含导致信号的程序指令的地址。

以下字段为非标准 Linux 扩展，仅在 POSIX 定时器到期时生成的信号传递时设置（参见 POSIX 间隔定时器）：

*si_timerid*

此字段包含内核用于内部标识定时器的 ID。

*si_overrun*

此字段设置为定时器的溢出计数。

以下两个字段仅在发送 `SIGIO` 信号时设置（信号驱动 I/O）：

*si_band*

此字段包含与 I/O 事件关联的“带事件”值。（在 *glibc* 版本 2.3.2 之前，*si_band* 的类型为 *int*。）

*si_fd*

此字段包含与 I/O 事件关联的文件描述符的编号。此字段在 SUSv3 中没有定义，但在许多其他实现中存在。

以下字段仅在发送 `SIGCHLD` 信号时设置（`SIGCHLD` 信号）：

*si_status*

此字段包含子进程的退出状态（如果 *si_code* 为 `CLD_EXITED`），或者发送到子进程的信号的编号（即终止或暂停子进程的信号的编号，如等待状态值中所述）。

*si_utime*

此字段包含子进程使用的用户 CPU 时间。在 2.6 版本之前的内核中，以及 2.6.27 版本以来，使用系统时钟刻度（除以 *sysconf(_SC_CLK_TCK)*）。在 2.6 内核的 2.6.27 之前版本中，由于一个 bug，此字段报告以（用户可配置的）时钟滴答（参见 软件时钟（时钟滴答））为单位的时间。此字段在 SUSv3 中没有指定，但它在许多其他实现中存在。

*si_stime*

此字段包含子进程使用的系统 CPU 时间。请参见 *si_utime* 字段的描述。此字段在 SUSv3 中没有指定，但它在许多其他实现中存在。

*si_code* 字段提供有关信号来源的进一步信息，使用的值显示在 表 21-2 中。并非表中第二列中显示的所有信号特定值都出现在所有 UNIX 实现和硬件架构中（尤其是在四个硬件生成的信号 `SIGBUS`、`SIGSEGV`、`SIGILL` 和 `SIGFPE` 的情况下），尽管这些常量在 Linux 上都已定义，并且大多数出现在 SUSv3 中。

请注意以下关于 表 21-2 中显示的值的附加说明：

+   值 `SI_KERNEL` 和 `SI_SIGIO` 是 Linux 特有的。它们在 SUSv3 中没有指定，并且在其他 UNIX 实现中没有出现。

+   `SI_SIGIO` 仅在 Linux 2.2 中使用。从内核 2.4 版本起，Linux 使用表中显示的 `POLL_*` 常量。

### 注意

SUSv4 指定了 *psiginfo()* 函数，其作用与 *psignal()* 相似（参见 显示信号描述）。*psiginfo()* 函数接受两个参数：一个指向 *siginfo_t* 结构体的指针和一个消息字符串。它将在标准错误上打印消息字符串，接着打印 *siginfo_t* 结构体中描述的信号信息。*psiginfo()* 函数自 glibc 2.10 版本以来由 *glibc* 提供。*glibc* 实现打印信号描述、信号来源（由 *si_code* 字段指示），以及对于某些信号，*siginfo_t* 结构体中的其他字段。*psiginfo()* 函数在 SUSv4 中新增，并非所有系统都支持该函数。

表 21-2. 在 *siginfo_t* 结构体的 *si_code* 字段中返回的值

| 信号 | *si_code* 值 | 信号来源 |
| --- | --- | --- |
| 任意 | `SI_ASYNCIO` | 异步 I/O（AIO）操作完成 |
|   | `SI_KERNEL` | 由内核发送（例如，来自终端驱动程序的信号） |
|   | `SI_MESGQ` | POSIX 消息队列上的消息到达（自 Linux 2.6.6 起） |
|   | `SI_QUEUE` | 通过 *sigqueue()* 发送的实时信号来自用户进程 |
|   | `SI_SIGIO` | `SIGIO` 信号（仅适用于 Linux 2.2） |
|   | `SI_TIMER` | POSIX（实时）定时器到期 |
|   | `SI_TKILL` | 通过 *tkill()* 或 *tgkill()* 终止的用户进程（自 Linux 2.4.19 起） |
|   | `SI_USER` | 通过 *kill()* 或 *raise()* 终止的用户进程 |
| `SIGBUS` | `BUS_ADRALN` | 无效的地址对齐 |
|   | `BUS_ADRERR` | 不存在的物理地址 |
|   | `BUS_MCEERR_AO` | 硬件内存错误；操作可选（自 Linux 2.6.32 起） |
|   | `BUS_MCEERR_AR` | 硬件内存错误；操作要求（自 Linux 2.6.32 起） |
|   | `BUS_OBJERR` | 对象特定的硬件错误 |
| `SIGCHLD` | `CLD_CONTINUED` | 子进程被 `SIGCONT` 信号继续执行（自 Linux 2.6.9 起） |
|   | `CLD_DUMPED` | 子进程异常终止，并生成核心转储 |
|   | `CLD_EXITED` | 子进程退出 |
|   | `CLD_KILLED` | 子进程异常终止，无核心转储 |
|   | `CLD_STOPPED` | 子进程已停止 |
|   | `CLD_TRAPPED` | 被跟踪的子进程已停止 |
| `SIGFPE` | `FPE_FLTDIV` | 浮点除以零 |
|   | `FPE_FLTINV` | 无效的浮点操作 |
|   | `FPE_FLTOVF` | 浮点溢出 |
|   | `FPE_FLTRES` | 浮点不精确结果 |
|   | `FPE_FLTUND` | 浮点下溢 |
|   | `FPE_INTDIV` | 整数除以零 |
|   | `FPE_INTOVF` | 整数溢出 |
|   | `FPE_SUB` | 下标越界 |
| `SIGILL` | `ILL_BADSTK` | 内部堆栈错误 |
|   | `ILL_COPROC` | 协处理器错误 |
|   | `ILL_ILLADR` | 非法寻址模式 |
|   | `ILL_ILLOPC` | 非法操作码 |
|   | `ILL_ILLOPN` | 非法操作数 |
|   | `ILL_ILLTRP` | 非法陷阱 |
|   | `ILL_PRVOPC` | 特权操作码 |
|   | `ILL_PRVREG` | 特权寄存器 |
| `SIGPOLL` | `POLL_ERR` | I/O 错误 |
| `SIGIO` | `POLL_HUP` | 设备断开连接 |
|   | `POLL_IN` | 输入数据可用 |
|   | `POLL_MSG` | 输入消息可用 |
|   | `POLL_OUT` | 输出缓冲区可用 |
|   | `POLL_PRI` | 高优先级输入可用 |
| `SIGSEGV` | `SEGV_ACCERR` | 对映射对象的权限无效 |
|   | `SEGV_MAPERR` | 地址未映射到对象 |
| `SIGTRAP` | `TRAP_BRANCH` | 进程分支陷阱 |
|   | `TRAP_BRKPT` | 进程断点 |
|   | `TRAP_HWBKPT` | 硬件断点/监视点 |
|   | `TRAP_TRACE` | 进程跟踪陷阱 |

#### *ucontext* 参数

传递给使用 `SA_SIGINFO` 标志设置的处理程序的最终参数，*ucontext*，是指向 *ucontext_t* 类型结构体的指针（定义在 `<ucontext.h>` 中）。(SUSv3 使用 *void* 指针作为此参数，因为它未指定该参数的任何细节。）该结构提供所谓的用户上下文信息，描述了信号处理程序调用前的进程状态，包括先前的进程信号屏蔽和保存的寄存器值（例如程序计数器和堆栈指针）。这些信息在信号处理程序中很少使用，因此我们不再进一步讨论。

### 注释

*ucontext_t*结构的另一个用途是与*getcontext()*、*makecontext()*、*setcontext()*和*swapcontext()*函数一起使用，这些函数允许进程检索、创建、更改和交换执行上下文，分别。 （这些操作有些像*setjmp()*和*longjmp()*，但更通用。）这些函数可用于实现协程，其中进程的执行线程在两个（或更多）函数之间交替。 SUSv3 指定了这些函数，但将其标记为过时。 SUSv4 删除了这些规范，并建议应用程序应重新编写以使用 POSIX 线程。 *glibc*手册提供了关于这些函数的更多信息。

## 系统调用的中断和重新启动

考虑以下情景：

1.  我们为某些信号建立了处理程序。

1.  我们进行阻塞系统调用，例如从终端设备读取*read()*，该调用会一直阻塞，直到输入被提供。

1.  当系统调用被阻塞时，传递了我们为其建立处理程序的信号，并调用了其信号处理程序。

信号处理程序返回后会发生什么？默认情况下，系统调用失败，返回错误`EINTR`（“中断函数”）。这可以是一个有用的特性。在阻塞操作设置超时中，我们将看到如何使用计时器（导致传递`SIGALRM`信号）为诸如*read()*之类的阻塞系统调用设置超时。

通常情况下，我们希望继续执行被中断的系统调用。为了做到这一点，我们可以使用以下代码来在信号处理程序中手动重新启动系统调用：

```
while ((cnt = read(fd, buf, BUF_SIZE)) == -1 && errno == EINTR)
    continue;                 /* Do nothing loop body */

if (cnt == -1)                /* read() failed with other than EINTR */
    errExit("read");
```

如果我们经常编写如上所示的代码，定义以下宏可能会很有用：

```
#define NO_EINTR(stmt) while ((stmt) == -1 && errno == EINTR);
```

使用此宏，我们可以将之前的*read()*调用重写为以下形式：

```
NO_EINTR(cnt = read(fd, buf, BUF_SIZE));

if (cnt == -1)                /* read() failed with other than EINTR */
    errExit("read");
```

### 注意

GNU C 库在`<unistd.h>`中提供了与我们的`NO_EINTR()`宏相同目的的（非标准）宏。如果定义了`_GNU_SOURCE`特性测试宏，则会提供名为`TEMP_FAILURE_RETRY()`的宏。

即使我们使用类似`NO_EINTR()`的宏，由于信号处理程序必须在每个阻塞系统调用中添加代码（假设我们要在每种情况下重新启动调用），中断系统调用可能会很不方便。相反，我们可以在使用*sigaction()*建立信号处理程序时指定`SA_RESTART`标志，以便内核代表进程自动重新启动系统调用。这意味着我们不需要为这些系统调用处理可能的`EINTR`错误返回。

`SA_RESTART`标志是每个信号的设置。换句话说，我们可以允许一些信号的处理程序中断阻塞系统调用，而其他信号则允许自动重新启动系统调用。

#### 对于有效的`SA_RESTART`系统调用（及库函数）

不幸的是，并非所有阻塞系统调用在指定了 `SA_RESTART` 后都会自动重新启动。部分原因是历史原因：

+   系统调用的重新启动功能是在 4.2BSD 中引入的，涵盖了对 *wait()* 和 *waitpid()* 的中断调用，以及以下 I/O 系统调用：*read()*、*readv()*、*write()*、*writev()* 和阻塞 *ioctl()* 操作。这些 I/O 系统调用是可中断的，因此仅当操作“慢”设备时，才会通过 `SA_RESTART` 自动重新启动。慢设备包括终端、管道、FIFO 和套接字。在这些文件类型上，各种 I/O 操作可能会阻塞。（相比之下，磁盘文件不属于慢设备类别，因为磁盘 I/O 操作通常可以通过缓存区快速完成。如果需要磁盘 I/O，内核会将进程挂起，直到 I/O 完成。）

+   许多其他阻塞系统调用源自 System V，最初并未提供系统调用的重新启动功能。

在 Linux 上，如果通过 `SA_RESTART` 标志使用信号处理程序中断，则以下阻塞系统调用（以及其上层的库函数）会自动重新启动：

+   用于等待子进程的系统调用（等待子进程）：*wait()*、*waitpid()*、*wait3()*、*wait4()* 和 *waitid()*。

+   I/O 系统调用 *read()*、*readv()*、*write()*、*writev()* 和 *ioctl()* 在应用于“慢”设备时。如果数据在信号传递时已经部分传输，输入输出系统调用将被中断，但会返回成功状态：一个整数，表示成功传输的字节数。

+   *open()* 系统调用，在可能发生阻塞的情况下（例如，打开 FIFO 时，如 FIFOs 中描述的那样）。

+   与套接字一起使用的各种系统调用：*accept()*、*accept4()*、*connect()*、*send()*、*sendmsg()*、*sendto()*、*recv()*、*recvfrom()* 和 *recvmsg()*。 （在 Linux 上，如果使用 *setsockopt()* 设置了套接字超时，则这些系统调用不会自动重新启动。详情请参见 *signal(7)* 手册页。）

+   用于 POSIX 消息队列上进行 I/O 的系统调用：*mq_receive()*、*mq_timedreceive()*、*mq_send()* 和 *mq_timedsend()*。

+   用于放置文件锁的系统调用和库函数：*flock()*、*fcntl()* 和 *lockf()*。

+   Linux 特有的 *futex()* 系统调用的 `FUTEX_WAIT` 操作。

+   *sem_wait()* 和 *sem_timedwait()* 函数用于递减一个 POSIX 信号量。 （在某些 UNIX 实现中，如果指定了`SA_RESTART`标志，*sem_wait()* 将会重新启动。）

+   用于同步 POSIX 线程的函数：*pthread_mutex_lock()*、*pthread_mutex_trylock()*、*pthread_mutex_timedlock()*、*pthread_cond_wait()* 和 *pthread_cond_timedwait()*。

在 2.6.22 版本之前的内核中，*futex()*、*sem_wait()*和*sem_timedwait()*在中断时总是返回`EINTR`错误，无论`SA_RESTART`标志的设置如何。

以下阻塞系统调用（以及建立在系统调用之上的库函数）即使指定了`SA_RESTART`，也不会自动重新启动：

+   *poll()*、*ppoll()*、*select()*和*pselect()* I/O 多路复用调用。 (SUSv3 明确指出，*select()*和*pselect()*在被信号处理程序中断时的行为未指定，无论`SA_RESTART`的设置如何。)

+   Linux 特有的*epoll_wait()*和*epoll_pwait()*系统调用。

+   Linux 特有的*io_getevents()*系统调用。

+   用于 System V 消息队列和信号量的阻塞系统调用：*semop()*、*semtimedop()*、*msgrcv()*和*msgsnd()*。 (尽管 System V 最初未提供自动重启系统调用的功能，但在某些 UNIX 实现中，如果指定了`SA_RESTART`标志，这些系统调用*会*被重启。)

+   从*inotify*文件描述符的*read()*操作。

+   旨在暂停程序执行一段指定时间的系统调用和库函数：*sleep()*、*nanosleep()*和*clock_nanosleep()*。

+   专门设计用于等待信号传递的系统调用：*pause()*、*sigsuspend()*、*sigtimedwait()*和*sigwaitinfo()*。

#### 修改`SA_RESTART`标志以处理信号

*siginterrupt()*函数改变与信号相关的`SA_RESTART`设置。

```
#include <signal.h>

int `siginterrupt`(int *sig*, int *flag*);
```

### 注意

成功时返回 0，出错时返回-1

如果*flag*为真（1），则信号*sig*的处理程序会中断阻塞系统调用。如果*flag*为假（0），则在执行*sig*的处理程序后，阻塞系统调用会重新启动。

*siginterrupt()*函数通过使用*sigaction()*获取信号当前处理方式的副本，修改返回的*oldact*结构中的`SA_RESTART`标志，然后再次调用*sigaction()*来更新信号的处理方式。

SUSv4 标记*siginterrupt()*为废弃，推荐改用*sigaction()*来完成此任务。

#### 某些 Linux 系统调用在未处理的停止信号下可能会生成`EINTR`错误。

在 Linux 中，某些阻塞系统调用即使没有信号处理程序，也可能返回`EINTR`错误。如果系统调用被阻塞且进程被信号（`SIGSTOP`、`SIGTSTP`、`SIGTTIN`或`SIGTTOU`）停止，然后通过`SIGCONT`信号恢复执行，就可能发生这种情况。

以下系统调用和函数展示了此行为：*epoll_pwait()*、*epoll_wait()*、*inotify*文件描述符的*read()*、*semop()*、*semtimedop()*、*sigtimedwait()*和*sigwaitinfo()*。

在 2.6.24 版本之前的内核中，*poll()*也表现出这种行为，*sem_wait()*、*sem_timedwait()*、futex(FUTEX_WAIT*)在 2.6.22 版本之前的内核中也存在类似行为，2.6.9 版本之前的内核中，*msgrcv()*和*msgsnd()*以及 Linux 2.4 及更早版本中的*nanosleep()*也有类似行为。

在 Linux 2.4 及更早版本中，*sleep()*也可以以这种方式中断，但它不会返回错误，而是返回剩余未睡眠的秒数。

这种行为的结果是，如果我们的程序有可能被信号停止并重新启动，那么我们可能需要在程序中加入代码，以便在程序没有安装停止信号的处理程序时，也能重新启动这些系统调用。

## 总结

在本章中，我们考虑了影响信号处理程序操作和设计的多种因素。

由于信号不被排队处理，信号处理程序有时必须编写成处理多次相同类型事件的可能性，即使只有一个信号被传递。重入性问题影响我们如何更新全局变量，并限制我们可以安全调用的函数集合。

信号处理程序可以以多种方式终止，而不是返回，包括调用*exit()*，通过发送信号终止进程（*kill()*, *raise()*, 或 *abort()*），或者执行非局部跳转。使用*sigsetjmp()*和*siglongjmp()*提供了程序对非局部跳转时进程信号掩码处理的显式控制。

我们可以使用*sigaltstack()*为进程定义一个备用信号栈。这是一个内存区域，在调用信号处理程序时，会替代标准的进程栈。备用信号栈在标准栈因过度增长而耗尽时特别有用（此时内核会向进程发送`SIGSEGV`信号）。

*sigaction()* `SA_SIGINFO`标志允许我们建立一个信号处理程序，接收关于信号的附加信息。此信息通过一个*siginfo_t*结构提供，其地址作为参数传递给信号处理程序。

当信号处理程序中断一个阻塞的系统调用时，系统调用会失败并返回错误`EINTR`。我们可以利用这一行为，例如在一个阻塞的系统调用上设置定时器。如果需要，中断的系统调用可以手动重新启动。或者，使用带有*sigaction()* `SA_RESTART`标志的信号处理程序可以导致许多（但不是所有）系统调用自动重新启动。

#### 更多信息

请参阅总结中列出的来源。

## 练习

1.  实现*abort()*。
