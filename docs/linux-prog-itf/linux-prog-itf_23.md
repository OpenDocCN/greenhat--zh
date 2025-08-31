## 第二十三章 定时器和睡眠

定时器允许进程安排在未来某个时间发生的通知。睡眠允许进程（或线程）暂停执行一段时间。本章描述了用于设置定时器和睡眠的接口。它包括以下主题：

+   用于设置间隔定时器的经典 UNIX API（*setitimer()*和*alarm()*），用于在指定时间过后通知进程；

+   允许进程在指定间隔内休眠的 API；

+   POSIX.1b 时钟和定时器 API；以及

+   Linux 特有的*timerfd*设施，允许创建定时器，其到期可以从文件描述符读取。

## 间隔定时器

*setitimer()*系统调用建立一个*interval timer*，即一个在未来某个时刻到期，并（可选）在此后定期到期的定时器。

```
#include <sys/time.h>

int `setitimer`(int *which*, const struct itimerval **new_value*,
              struct itimerval **old_value*);
```

### 注意

成功时返回 0，出错时返回-1

使用*setitimer()*，进程可以通过将*which*指定为以下之一来建立三种不同类型的定时器：

`ITIMER_REAL`

创建一个以实时时间（即墙钟时间）倒计时的定时器。当定时器到期时，会为进程生成一个`SIGALRM`信号。

`ITIMER_VIRTUAL`

创建一个以进程虚拟时间（即用户模式 CPU 时间）倒计时的定时器。当定时器到期时，会为进程生成一个`SIGVTALRM`信号。

`ITIMER_PROF`

创建一个*profiling*定时器。一个 profiling 定时器以进程时间（即用户模式和内核模式的 CPU 时间之和）倒计时。当定时器到期时，会为进程生成一个`SIGPROF`信号。

所有定时器信号的默认处置是终止进程。除非这是期望的结果，否则我们必须为定时器发送的信号建立一个处理程序。

*new_value*和*old_value*参数是指向*itimerval*结构的指针，定义如下：

```
struct itimerval {
    struct timeval it_interval;     /* Interval for periodic timer */
    struct timeval it_value;        /* Current value (time until
                                       next expiration) */
};
```

*itimerval*结构中的每个字段又是一个*timeval*类型的结构，包含秒和微秒字段：

```
struct timeval {
    time_t      tv_sec;             /* Seconds */
    suseconds_t tv_usec;            /* Microseconds (long int) */
};
```

*it_value*子结构的*new_value*参数指定定时器到期的延迟时间。*it_interval*子结构指定这是否是一个周期性定时器。如果*it_interval*的两个字段都设置为 0，那么定时器只会到期一次，在*it_value*指定的时间到期。如果*it_interval*的一个或两个字段为非零，则每次定时器到期后，定时器会重置，并在指定的时间间隔后再次到期。

每个进程只能有三种定时器中的一种。如果我们第二次调用*setitimer()*，它将改变与*which*对应的任何现有定时器的特性。如果我们调用*setitimer()*时，*new_value.it_value*的两个字段都设置为 0，那么任何现有定时器将被禁用。

如果*old_value*不是`NULL`，则它指向一个*itimerval*结构，用于返回定时器的上一个值。如果*old_value.it_value*的两个字段都为 0，则定时器之前已被禁用。如果*old_value.it_interval*的两个字段都为 0，则之前的定时器设置为仅在*old_value.it_value*指定的时间到达时过期一次。检索定时器的先前设置在我们希望在新定时器过期后恢复设置时非常有用。如果我们不关心定时器的先前值，可以将*old_value*指定为`NULL`。

随着定时器的推进，它会从初始值(*it_value*)倒计时直到 0。当定时器达到 0 时，相应的信号会发送到进程，然后，如果间隔(*it_interval*)非零，定时器值(*it_value*)会重新加载，并重新开始倒计时直到 0。

在任何时候，我们都可以使用*getitimer()*来检索定时器的当前状态，以查看距离下次过期还剩多少时间。

```
#include <sys/time.h>

int `getitimer`(int *which*, struct itimerval **curr_value*);
```

### 注意

成功返回 0，出错返回-1

*getitimer()*系统调用返回由*which*指定的定时器的当前状态，存储在*curr_value*指向的缓冲区中。这与通过*setitimer()*的*old_value*参数返回的信息完全相同，区别在于我们无需更改定时器设置就可以检索信息。*curr_value.it_value*子结构返回定时器下次过期前剩余的时间。随着定时器倒计时，这个值会发生变化，如果在设置定时器时指定了非零的*it_interval*值，当定时器过期时该值会被重置。*curr_value.it_interval*子结构返回此定时器的间隔；此值保持不变，直到后续调用*setitimer()*。

使用*setitimer()*（以及稍后讨论的*alarm()*）建立的定时器在*exec()*调用后会被保留，但不会被*fork()*创建的子进程继承。

### 注意

SUSv4 标记*getitimer()*和*setitimer()*为过时，指出首选使用 POSIX 定时器 API（POSIX 间隔定时器）。

#### 示例程序

示例 23-1 演示了*setitimer()*和*getitimer()*的使用。该程序执行以下步骤：

+   为`SIGALRM`信号建立处理程序 ![](img/U003.png)。

+   使用命令行参数中提供的值设置一个实时（`ITIMER_REAL`）定时器的值和间隔字段 ![](img/U004.png)。如果这些参数缺失，程序将设置一个仅在 2 秒后过期一次的定时器。

+   执行一个连续的循环 ![](img/U005.png)，消耗 CPU 时间，并定期调用函数*displayTimes()* ![](img/U001.png)，该函数显示程序开始以来的经过实际时间以及当前`ITIMER_REAL`定时器的状态。

每次定时器到期时，`SIGALRM` 处理程序会被调用，并设置一个全局标志 *gotAlarm* ![](img/U002.png)。每当此标志被设置时，主程序中的循环会调用 *displayTimes()* 以显示处理程序何时被调用以及定时器的状态 ![](img/U006.png)。 （我们设计信号处理程序时这样做是为了避免在处理程序中调用非异步信号安全函数，原因请参见 可重入与异步信号安全函数）。如果定时器的间隔为零，则程序在收到第一个信号时退出；否则，它会在终止之前捕获最多三个信号 ![](img/U007.png)。

当我们运行 示例 23-1 中的程序时，我们会看到以下内容：

```
$ `./real_timer 1 800000 1 0`         *Initial value 1.8 seconds, interval 1 second*
       Elapsed   Value  Interval
START:    0.00
Main:     0.50    1.30    1.00      *Timer counts down until expiration*
Main:     1.00    0.80    1.00
Main:     1.50    0.30    1.00
ALARM:    1.80    1.00    1.00      *On expiration, timer is reloaded from interval*
Main:     2.00    0.80    1.00
Main:     2.50    0.30    1.00
ALARM:    2.80    1.00    1.00
Main:     3.00    0.80    1.00
Main:     3.50    0.30    1.00
ALARM:    3.80    1.00    1.00
That's all folks
```

示例 23-1. 使用实时定时器

```
`timers/real_timer.c`
    #include <signal.h>
    #include <sys/time.h>
    #include <time.h>
    #include "tlpi_hdr.h"

    static volatile sig_atomic_t gotAlarm = 0;
                            /* Set nonzero on receipt of SIGALRM */

    /* Retrieve and display the real time, and (if 'includeTimer' is
       TRUE) the current value and interval for the ITIMER_REAL timer */

    static void
 displayTimes(const char *msg, Boolean includeTimer)
    {
        struct itimerval itv;
        static struct timeval start;
        struct timeval curr;
        static int callNum = 0;             /* Number of calls to this function */

        if (callNum == 0)                   /* Initialize elapsed time meter */
            if (gettimeofday(&start, NULL) == -1)
                errExit("gettimeofday");

        if (callNum % 20 == 0)              /* Print header every 20 lines */
            printf("       Elapsed   Value Interval\n");

            if (gettimeofday(&curr, NULL) == -1)
            errExit("gettimeofday");
        printf("%-7s %6.2f", msg, curr.tv_sec - start.tv_sec +
                            (curr.tv_usec - start.tv_usec) / 1000000.0);

        if (includeTimer) {
            if (getitimer(ITIMER_REAL, &itv) == -1)
                errExit("getitimer");
            printf("  %6.2f  %6.2f",
                    itv.it_value.tv_sec + itv.it_value.tv_usec / 1000000.0,
                    itv.it_interval.tv_sec + itv.it_interval.tv_usec / 1000000.0);
        }

        printf("\n");
        callNum++;
    }

    static void
    sigalrmHandler(int sig)
    {
     gotAlarm = 1;
    }

    int
    main(int argc, char *argv[])
    {
        struct itimerval itv;
        clock_t prevClock;
        int maxSigs;                /* Number of signals to catch before exiting */
        int sigCnt;                 /* Number of signals so far caught */
        struct sigaction sa;

        if (argc > 1 && strcmp(argv[1], "--help") == 0)
            usageErr("%s [secs [usecs [int-secs [int-usecs]]]]\n", argv[0]);

        sigCnt = 0;

        sigemptyset(&sa.sa_mask);
        sa.sa_flags = 0;
        sa.sa_handler = sigalrmHandler;
     if (sigaction(SIGALRM, &sa, NULL) == -1)
            errExit("sigaction");

         /* Set timer from the command-line arguments */

        itv.it_value.tv_sec = (argc > 1) ? getLong(argv[1], 0, "secs") : 2;
        itv.it_value.tv_usec = (argc > 2) ? getLong(argv[2], 0, "usecs") : 0;
        itv.it_interval.tv_sec = (argc > 3) ? getLong(argv[3], 0, "int-secs") : 0;
        itv.it_interval.tv_usec = (argc > 4) ? getLong(argv[4], 0, "int-usecs") : 0;

       /* Exit after 3 signals, or on first signal if interval is 0 */

        maxSigs = (itv.it_interval.tv_sec == 0 &&
                    itv.it_interval.tv_usec == 0) ? 1 : 3;

            displayTimes("START:", FALSE);
     if (setitimer(ITIMER_REAL, &itv, 0) == -1)
            errExit("setitimer");

        prevClock = clock();
        sigCnt = 0;

     for (;;) {

            /* Inner loop consumes at least 0.5 seconds CPU time */

            while (((clock() - prevClock) * 10 / CLOCKS_PER_SEC) < 5) {
             if (gotAlarm) {                     /* Did we get a signal? */
                    gotAlarm = 0;
                    displayTimes("ALARM:", TRUE);

                    sigCnt++;
                 if (sigCnt >= maxSigs) {
                        printf("That's all folks\n");
                        exit(EXIT_SUCCESS);
                    }
                }
            }

            prevClock = clock();
            displayTimes("Main: ", TRUE);
        }
    }
         `timers/real_timer.c`
```

#### 一个更简单的定时器接口：*alarm()*

*alarm()* 系统调用提供了一个简单的接口，用于建立一个只会到期一次、不重复的实时定时器。（历史上，*alarm()* 是 UNIX 原始的定时器设置 API。）

```
#include <unistd.h>

unsigned int `alarm`(unsigned int *seconds*);
```

### 注意

总是成功，返回任何先前设置的定时器剩余的秒数，如果没有设置定时器则返回 0

*seconds* 参数指定定时器到期的未来秒数。在那个时间点，`SIGALRM` 信号会被发送给调用进程。

使用 *alarm()* 设置定时器会覆盖任何之前设置的定时器。我们可以通过调用 *alarm(0)* 来禁用现有的定时器。

*alarm()* 的返回值给出我们任何先前设置的定时器到期前剩余的秒数，如果没有设置定时器，则返回 0。

在第 23.3 节中展示了使用 *alarm()* 的一个示例。

### 注意

在本书的某些后续示例程序中，我们使用 *alarm()* 启动定时器，而不设置相应的 `SIGALRM` 处理程序，这是一种确保进程被终止的技术，前提是进程没有被其他方式终止。

#### *setitimer()* 和 *alarm()* 之间的交互

在 Linux 上，*alarm()* 和 *setitimer()* 共享同一个每进程的实时定时器，这意味着使用这些函数之一设置定时器会改变之前由任何一个函数设置的定时器。在其他 UNIX 实现中可能并非如此（即，这些函数可能控制独立的定时器）。SUSv3 明确没有指定 *setitimer()* 和 *alarm()* 之间的交互，以及这些函数与 低分辨率睡眠：*sleep()*") 函数之间的交互。为了最大限度地提高可移植性，我们应确保我们的应用程序仅使用 *setitimer()* 或 *alarm()* 中的一个来设置实时定时器。

## 定时器的调度和准确性

根据系统负载和进程调度的情况，某个进程可能要等到定时器实际到期后一小段时间（通常是几秒的一小部分）才会被调度执行。尽管如此，由*setitimer()*建立的周期定时器，或本章后续描述的其他接口，仍将保持规律。例如，如果一个实时定时器被设置为每 2 秒到期，那么尽管个别定时器事件的触发可能会受到上述延迟的影响，但后续定时器到期的调度仍将严格在下一个 2 秒的间隔内进行。换句话说，间隔定时器不会出现累积误差。

尽管*setitimer()*使用的*timeval*结构支持微秒级精度，但定时器的精度传统上受到软件时钟频率的限制（参见软件时钟（Jiffies））。如果定时器值与软件时钟的粒度没有完全匹配，定时器值将被四舍五入。例如，如果我们指定一个间隔定时器，每 19,100 微秒触发一次（即大约 19 毫秒），假设时钟粒度为 4 毫秒，那么实际的定时器将每 20 毫秒触发一次。

#### 高分辨率定时器

在现代 Linux 内核中，之前关于定时器分辨率受限于软件时钟频率的说法不再成立。从 2.6.21 内核版本开始，Linux 可选地支持高分辨率定时器。如果启用了此支持（通过`CONFIG_HIGH_RES_TIMERS`内核配置选项），那么我们在本章中描述的各种定时器和睡眠接口的精度将不再受内核时钟的大小限制。相反，这些调用的精度可以达到底层硬件所允许的程度。在现代硬件上，精度通常可达到微秒级。

### 注意

可以通过检查*clock_getres()*返回的时钟分辨率来确定高分辨率定时器的可用性，相关内容可以参见获取时钟值：*clock_gettime()*")。

## 设置阻塞操作的超时

实时定时器的一个用途是对阻塞系统调用的最大阻塞时间设定上限。例如，我们可能希望在用户在一定时间内没有输入一行内容的情况下取消从终端的*read()*调用。我们可以按如下方式实现：

1.  调用*sigaction()*建立`SIGALRM`的处理程序，省略`SA_RESTART`标志，这样系统调用就不会被重新启动（参见系统调用的中断与重启）。

1.  调用*alarm()*或*setitimer()*来设置一个定时器，指定我们希望系统调用阻塞的最大时间限制。

1.  执行阻塞的系统调用。

1.  系统调用返回后，再次调用*alarm()*或*setitimer()*来禁用定时器（以防系统调用在定时器过期之前完成）。

1.  检查阻塞系统调用是否失败，并将*errno*设置为`EINTR`（被中断的系统调用）。

示例 23-2")演示了使用*alarm()*来设置定时器的*read()*技术。

示例 23-2. 执行带超时的*read()*

```
`timers/timed_read.c`
#include <signal.h>
#include "tlpi_hdr.h"

#define BUF_SIZE 200

static void     /* SIGALRM handler: interrupts blocked system call */
handler(int sig)
{
    printf("Caught signal\n");          /* UNSAFE (see Section 21.1.2) */
}

int
main(int argc, char *argv[])
{
    struct sigaction sa;
    char buf[BUF_SIZE];
    ssize_t numRead;
    int savedErrno;

    if (argc > 1 && strcmp(argv[1], "--help") == 0)
        usageErr("%s [num-secs [restart-flag]]\n", argv[0]);

    /* Set up handler for SIGALRM. Allow system calls to be interrupted,
       unless second command-line argument was supplied. */

    sa.sa_flags = (argc > 2) ? SA_RESTART : 0;
    sigemptyset(&sa.sa_mask);
    sa.sa_handler = handler;
    if (sigaction(SIGALRM, &sa, NULL) == -1)
        errExit("sigaction");

    alarm((argc > 1) ? getInt(argv[1], GN_NONNEG, "num-secs") : 10);

    numRead = read(STDIN_FILENO, buf, BUF_SIZE - 1);

    savedErrno = errno;                 /* In case alarm() changes it */
    alarm(0);                           /* Ensure timer is turned off */
    errno = savedErrno;

    /* Determine result of read() */

    if (numRead == -1) {
        if (errno == EINTR)
            printf("Read timed out\n");
        else
            errMsg("read");
    } else {
        printf("Successful read (%ld bytes): %.*s",
                (long) numRead, (int) numRead, buf);
    }

    exit(EXIT_SUCCESS);
}
     `timers/timed_read.c`
```

请注意，示例 23-2")中的程序存在一个理论上的竞态条件。如果定时器在调用*alarm()*之后，但在*read()*调用开始之前过期，则*read()*调用不会被信号处理程序中断。由于在这种情况下使用的超时值通常较大（至少几秒钟），因此这种情况发生的可能性很小，因此在实际操作中，这是一种可行的技术。[Stevens & Rago, 2005]提出了一种使用*longjmp()*的替代技术。处理 I/O 系统调用的另一种替代方案是使用*select()*或*poll()*系统调用的超时功能（第六十三章），它们还有一个优点，就是能够同时等待多个描述符的 I/O。

## 为固定时间间隔挂起执行（睡眠）

有时，我们希望将进程执行挂起一定的固定时间。虽然可以通过组合使用*sigsuspend()*和前面描述的定时器函数来实现，但使用其中一个睡眠函数会更为简单。

### 低分辨率睡眠：*sleep()*

*sleep()*函数将调用进程的执行挂起，直到指定的*seconds*参数秒数或者信号被捕获（从而中断调用）。

```
#include <unistd.h>

unsigned int `sleep`(unsigned int *seconds*);
```

### 注意

正常完成时返回 0，或者在被提前终止时返回未睡眠的秒数

如果睡眠完成，*sleep()*返回 0。如果睡眠被信号中断，*sleep()*返回剩余的（未睡眠的）秒数。与*alarm()*和*setitimer()*设置的定时器一样，系统负载可能意味着进程只有在*sleep()*调用完成后的某个（通常较短的）时间才会被重新调度。

SUSv3 未明确规定*sleep()*与*alarm()*和*setitimer()*的可能交互。在 Linux 上，*sleep()*是通过调用*nanosleep()*来实现的（高分辨率睡眠：*nanosleep()*")），因此，*sleep()*与定时器函数之间没有交互。然而，在许多实现中，特别是较老的实现中，*sleep()*是通过*alarm()*和`SIGALRM`信号的处理程序来实现的。为了可移植性，我们应该避免将*sleep()*与*alarm()*和*setitimer()*混合使用。

### 高分辨率睡眠：*nanosleep()*

*nanosleep()*函数执行的任务与*sleep()*类似，但提供了许多优点，包括在指定睡眠间隔时具有更精细的分辨率。

```
#define _POSIX_C_SOURCE 199309
#include <time.h>

int `nanosleep`(const struct timespec **request*, struct timespec **remain*);
```

### 注意

成功完成睡眠时返回 0，发生错误或睡眠被中断时返回-1。

*request*参数指定睡眠间隔的持续时间，并且是指向如下结构的指针：

```
struct timespec {
    time_t tv_sec;         /* Seconds */
    long   tv_nsec;        /* Nanoseconds */
};
```

*tv_nsec*字段指定纳秒值。它必须是 0 到 999,999,999 之间的数字。

*nanosleep()*的另一个优点是 SUSv3 明确规定它不应通过信号来实现。这意味着，与*sleep()*的情况不同，我们可以在可移植性方面，将*nanosleep()*与*alarm()*或*setitimer()*的调用混合使用。

尽管*nanosleep()*的实现并未使用信号，但它仍可能被信号处理程序中断。在这种情况下，*nanosleep()*返回-1，并将*errno*设置为常见的`EINTR`，如果参数*remain*不为`NULL`，则指向的缓冲区返回剩余的未睡眠时间。如果需要，我们可以使用返回的值重新启动系统调用以完成睡眠。这在示例 23-3")中有所演示。作为命令行参数，该程序期望为*nanosleep()*提供秒和纳秒值。程序会反复循环，执行*nanosleep()*直到总睡眠时间过去。如果*nanosleep()*被`SIGINT`（由输入*Control-C*生成）信号的处理程序中断，则使用*remain*中返回的值重新启动调用。当我们运行该程序时，看到如下输出：

```
$ `./t_nanosleep 10 0`                      *Sleep for 10 seconds*
*Type Control-C*
Slept for:  1.853428 secs
Remaining:  8.146617000
*Type Control-C*
Slept for:  4.370860 secs
Remaining:  5.629800000
*Type Control-C*
Slept for:  6.193325 secs
Remaining:  3.807758000
Slept for: 10.008150 secs
Sleep complete
```

尽管*nanosleep()*在指定睡眠间隔时允许纳秒级精度，但睡眠间隔的准确性仍然受到软件时钟粒度的限制（软件时钟（Jiffies））。如果我们指定的间隔不是软件时钟的倍数，则该间隔会被向上舍入。

### 注意

如前所述，在支持高分辨率定时器的系统上，睡眠间隔的精度可以比软件时钟的粒度更细。

这种舍入行为意味着，如果信号接收频率很高，那么在示例 23-3")中的方法就会出现问题。问题在于，每次重新启动 *nanosleep()* 都会受到舍入误差的影响，因为返回的 *remain* 时间不太可能是软件时钟粒度的整数倍。因此，每次重新启动的 *nanosleep()* 会比前一次调用返回的 *remain* 时间更长。在信号传递频率极高（即与或更频繁于软件时钟粒度）时，进程可能永远无法完成其睡眠。在 Linux 2.6 中，可以通过使用带 `TIMER_ABSTIME` 选项的 *clock_nanosleep()* 来避免此问题。我们在改进的高分辨率睡眠：*clock_nanosleep()*")中描述了 *clock_nanosleep()*。

### 注意

在 Linux 2.4 及更早版本中，*nanosleep()* 的实现存在一个异常。假设一个进程正在执行 *nanosleep()* 调用，并被信号暂停。当该进程通过发送 `SIGCONT` 信号恢复时，*nanosleep()* 调用会按预期失败，并返回错误 `EINTR`。然而，如果程序随后重新启动 *nanosleep()* 调用，则进程在暂停状态下的时间*不会*被计算在睡眠间隔内，这样进程将比预期睡得更久。这个异常在 Linux 2.6 中被消除，在该版本中，*nanosleep()* 调用会在收到 `SIGCONT` 信号时自动恢复，并且睡眠状态的时间会被计算在睡眠间隔内。

示例 23-3：使用 *nanosleep()*

```
`timers/t_nanosleep.c`
#define _POSIX_C_SOURCE 199309
#include <sys/time.h>
#include <time.h>
#include <signal.h>
#include "tlpi_hdr.h"

static void
sigintHandler(int sig)
{
    return;                     /* Just interrupt nanosleep() */
}

int
main(int argc, char *argv[])
{
    struct timeval start, finish;
    struct timespec request, remain;
    struct sigaction sa;
    int s;

    if (argc != 3 || strcmp(argv[1], "--help") == 0)
        usageErr("%s secs nanosecs\n", argv[0]);

    request.tv_sec = getLong(argv[1], 0, "secs");
    request.tv_nsec = getLong(argv[2], 0, "nanosecs");

    /* Allow SIGINT handler to interrupt nanosleep() */

    sigemptyset(&sa.sa_mask);
    sa.sa_flags = 0;
    sa.sa_handler = sigintHandler;
    if (sigaction(SIGINT, &sa, NULL) == -1)
        errExit("sigaction");

    if (gettimeofday(&start, NULL) == -1)
        errExit("gettimeofday");

    for (;;) {
        s = nanosleep(&request, &remain);
        if (s == -1 && errno != EINTR)
            errExit("nanosleep");

        if (gettimeofday(&finish, NULL) == -1)
            errExit("gettimeofday");
        printf("Slept for: %9.6f secs\n", finish.tv_sec - start.tv_sec +
                        (finish.tv_usec - start.tv_usec) / 1000000.0);

        if (s == 0)
            break;                      /* nanosleep() completed */

        printf("Remaining: %2ld.%09ld\n", (long) remain.tv_sec,
                remain.tv_nsec);
        request = remain;               /* Next sleep is with remaining time */
    }

    printf("Sleep complete\n");
    exit(EXIT_SUCCESS);
}
      `timers/t_nanosleep.c`
```

## POSIX 时钟

POSIX 时钟（最初定义在 POSIX.1b 中）提供了一个 API，用于访问具有纳秒精度的时钟。纳秒时间值使用与 *nanosleep()* 相同的 *timespec* 结构表示（参见高分辨率睡眠：*nanosleep()*")）。

在 Linux 中，使用此 API 的程序必须使用 *-lrt* 选项编译，以便链接 *librt*（实时）库。

POSIX 时钟 API 的主要系统调用有 *clock_gettime()*，用于检索时钟的当前值；*clock_getres()*，用于返回时钟的分辨率；以及 *clock_settime()*，用于更新时钟。

### 获取时钟值：*clock_gettime()*

*clock_gettime()* 系统调用根据 *clockid* 指定的时钟返回时间。

```
#define _POSIX_C_SOURCE 199309
#include <time.h>

int `clock_gettime`(clockid_t *clockid*, struct timespec **tp*);
int `clock_getres`(clockid_t *clockid*, struct timespec **res*);
```

### 注意

两者成功时返回 0，出错时返回 -1。

时间值以 *tp* 所指向的 timespec 结构返回。尽管 *timespec* 结构提供纳秒级精度，但 *clock_gettime()* 返回的时间值的粒度可能比这更粗。*clock_getres()* 系统调用返回指向一个 *timespec* 结构的指针，该结构包含所指定的 *clockid* 时钟的分辨率。

*clockid_t* 数据类型是由 SUSv3 指定的，用于表示时钟标识符。表格 23-1 的第一列列出了可以为 *clockid* 指定的值。

表格 23-1. POSIX.1b 时钟类型

| 时钟 ID | 描述 |
| --- | --- |
| `CLOCK_REALTIME` | 可设置的全系统实时时钟 |
| `CLOCK_MONOTONIC` | 不可设置的单调时钟 |
| `CLOCK_PROCESS_CPUTIME_ID` | 每进程的 CPU 时间时钟（自 Linux 2.6.12 起） |
| `CLOCK_THREAD_CPUTIME_ID` | 每线程的 CPU 时间时钟（自 Linux 2.6.12 起） |

`CLOCK_REALTIME` 时钟是一个全系统的时钟，测量挂钟时间。与 `CLOCK_MONOTONIC` 时钟相比，该时钟的设置可以被更改。

SUSv3 规定，`CLOCK_MONOTONIC` 时钟测量自某个“未指定的过去时间点”以来的时间，该时间点在系统启动后不会改变。该时钟适用于不能受系统时钟的不连续变更影响的应用程序（例如，手动更改系统时间）。在 Linux 上，该时钟测量自系统启动以来的时间。

`CLOCK_PROCESS_CPUTIME_ID` 时钟测量调用进程消耗的用户和系统 CPU 时间。`CLOCK_THREAD_CPUTIME_ID` 时钟对进程中的单个线程执行类似任务。

表格 23-1 中的所有时钟都由 SUSv3 规定，但只有 `CLOCK_REALTIME` 是强制性的，并且在 UNIX 实现中得到广泛支持。

### 注意

Linux 2.6.28 在 表格 23-1 列出的时钟之外，新增了一种时钟类型 `CLOCK_MONOTONIC_RAW`。这是一种不可设置的时钟，类似于 `CLOCK_MONOTONIC`，但它提供对纯硬件时间的访问，且不受 NTP 调整的影响。该非标准时钟用于特定的时钟同步应用程序。

Linux 2.6.32 在 表格 23-1 列出的时钟之外，新增了两个时钟：`CLOCK_REALTIME_COARSE` 和 `CLOCK_MONOTONIC_COARSE`。这些时钟与 `CLOCK_REALTIME` 和 `CLOCK_MONOTONIC` 类似，但用于需要以最小成本获得低分辨率时间戳的应用程序。这些非标准时钟不会访问硬件时钟（某些硬件时钟源的访问可能非常昂贵），返回值的分辨率是 jiffy（软件时钟（Jiffies））。

### 设置时钟值：*clock_settime()*

*clock_settime()*系统调用将*clockid*指定的时钟设置为*tp*指向的缓冲区中的时间。

```
#define _POSIX_C_SOURCE 199309
#include <time.h>int `clock_settime`(clockid_t *clockid*,
 const struct timespec **tp*);
```

### 注意

成功时返回 0，出错时返回-1

如果*tp*指定的时间不是*clock_getres()*返回的时钟分辨率的倍数，时间将向下舍入。

具有特权的（`CAP_SYS_TIME`）进程可以设置`CLOCK_REALTIME`时钟。该时钟的初始值通常是自纪元以来的时间。表 23-1 中列出的其他时钟不可修改。

### 注意

根据 SUSv3 规范，某些实现可能允许设置`CLOCK_PROCESS_CPUTIME_ID`和`CLOCK_THREAD_CPUTIME_ID`时钟。截至写作时，这些时钟在 Linux 上是只读的。

### 获取特定进程或线程的时钟 ID

本节描述的函数允许我们获取测量特定进程或线程 CPU 时间的时钟 ID。我们可以将返回的时钟 ID 用于调用*clock_gettime()*，以便了解进程或线程消耗的 CPU 时间。

*clock_getcpuclockid()*函数返回指定*pid*进程的 CPU 时间时钟的标识符，该标识符保存在*clockid*指向的缓冲区中。

```
#include <time.h>

int `clock_getcpuclockid`(pid_t *pid*, clockid_t **clockid*);
```

### 注意

成功时返回 0，出错时返回正数错误码

如果*pid*为 0，*clock_getcpuclockid()*返回调用进程的 CPU 时间时钟的 ID。

*pthread_getcpuclockid()*函数是 POSIX 线程的*clock_getcpuclockid()*函数的对应函数。它返回测量调用进程中特定线程 CPU 时间的时钟标识符。

```
#include <pthread.h>
#include <time.h>

int `pthread_getcpuclockid`(pthread_t *thread*, clockid_t **clockid*);
```

### 注意

成功时返回 0，出错时返回正数错误码

*thread*参数是一个 POSIX 线程 ID，表示我们想要获取其 CPU 时间时钟 ID 的线程。时钟 ID 将保存在*clockid*指向的缓冲区中。

### 改进的高分辨率睡眠：*clock_nanosleep()*

与*nanosleep()*类似，Linux 特有的*clock_nanosleep()*系统调用会暂停调用进程，直到指定的时间间隔过去或收到信号。在本节中，我们描述了将*clock_nanosleep()*与*nanosleep()*区分开来的特点。

```
#include <time.h>

int `clock_nanosleep`(clockid_t *clockid*, *int flags*,
       const struct timespec **request*, struct timespec **remain*);
```

### 注意

成功完成睡眠时返回 0，出错或被中断时返回正数错误码

*request*和*remain*参数的作用与*nanosleep()*的相应参数类似。

默认情况下（即*flags*为 0 时），*request*中指定的睡眠间隔是相对的（如同*nanosleep()*）。然而，如果我们在*flags*中指定了`TIMER_ABSTIME`（请参见示例 23-4")中的示例），则*request*指定了一个由*clockid*所标识的时钟测量的绝对时间。这个特性在需要精确睡眠直到特定时间的应用中至关重要。如果我们试图获取当前时间，计算与目标时间的差值，并进行相对睡眠，那么可能会发生进程在这些步骤的中间被抢占，导致睡眠时间比预期的更长。

正如在高分辨率睡眠：*nanosleep()*")中所描述的，当一个进程使用循环来重新启动被信号处理程序中断的睡眠时，这种“过度睡眠”问题尤其明显。如果信号以较高的频率传递，那么由*nanosleep()*执行的相对睡眠可能会导致进程睡眠时间出现较大误差。我们可以通过首先调用*clock_gettime()*来获取当前时间，将所需的时间加到该时间上，然后使用`TIMER_ABSTIME`标志调用*clock_nanosleep()*（如果被信号处理程序中断，重新启动系统调用）来避免过度睡眠问题。

当指定了`TIMER_ABSTIME`标志时，*remain*参数将不被使用（它是多余的）。如果*clock_nanosleep()*调用被信号处理程序中断，则可以通过重复相同的*request*参数重新启动睡眠。

另一个使*clock_nanosleep()*与*nanosleep()*不同的特性是，我们可以选择用于测量睡眠间隔的时钟。我们在*clockid*中指定所需的时钟：`CLOCK_REALTIME`、`CLOCK_MONOTONIC`或`CLOCK_PROCESS_CPUTIME_ID`。有关这些时钟的描述，请参见表 23-1。

示例 23-4") 演示了如何使用*clock_nanosleep()*通过绝对时间值以`CLOCK_REALTIME`时钟来睡眠 20 秒。

示例 23-4. 使用*clock_nanosleep()*

```
struct timespec request;

    /* Retrieve current value of CLOCK_REALTIME clock */

    if (clock_gettime(CLOCK_REALTIME, &request) == -1)
        errExit("clock_gettime");

    request.tv_sec += 20;               /* Sleep for 20 seconds from now */

    s = clock_nanosleep(CLOCK_REALTIME, TIMER_ABSTIME, &request, NULL);
    if (s != 0) {
        if (s == EINTR)
            printf("Interrupted by signal handler\n");
        else
            errExitEN(s, "clock_nanosleep");
    }
```

## POSIX 间隔计时器

由*setitimer()*设置的经典 UNIX 间隔计时器存在若干限制：

+   我们只能设置每种类型的计时器一次，分别是`ITIMER_REAL`、`ITIMER_VIRTUAL`和`ITIMER_PROF`。

+   唯一通知计时器到期的方式是通过信号传递。而且，我们不能更改计时器到期时生成的信号。

+   如果间隔定时器在相应的信号被阻塞时多次到期，则信号处理程序仅被调用一次。换句话说，我们无法知道是否发生了 *timeroverrun*。

+   定时器的精度限制为微秒。然而，一些系统具有提供比这更高精度的硬件时钟，在这些系统上，能够访问这种更高精度的时间就显得尤为重要。

POSIX.1b 定义了一个 API 来解决这些限制，这个 API 在 Linux 2.6 中得到了实现。

### 注意

在较老的 Linux 系统中，通过基于线程的 *glibc* 实现提供了此 API 的不完整版本。然而，这种用户空间实现并未提供此处描述的所有功能。

POSIX 定时器 API 将定时器的生命周期分为以下几个步骤：

+   *timer_create()* 系统调用创建一个新的定时器，并定义当定时器到期时如何通知进程。

+   *timer_settime()* 系统调用启动（开始）或停止（停止）定时器。

+   *timer_delete()* 系统调用删除不再需要的定时器。

POSIX 定时器不会被 *fork()* 创建的子进程继承。它们会在 *exec()* 或进程终止时被禁用并删除。

在 Linux 上，使用 POSIX 定时器 API 的程序必须使用 *-lrt* 选项进行编译，以便链接到 *librt*（实时）库。

### 创建定时器：*timer_create()*

*timer_create()* 函数创建一个新的定时器，该定时器使用由 *clockid* 指定的时钟来计时。

```
#define _POSIX_C_SOURCE 199309
#include <signal.h>
#include <time.h>

int `timer_create`(clockid_t *clockid*, struct sigevent **evp*, timer_t **timerid*);
```

### 注意

成功时返回 0，出错时返回 -1

*clockid* 可以指定 表 23-1 中显示的任何值，或者由 *clock_getcpuclockid()* 或 *pthread_getcpuclockid()* 返回的 *clockid* 值。*timerid* 参数指向一个缓冲区，该缓冲区返回一个句柄，用于在后续系统调用中引用定时器。这个缓冲区的类型是 *timer_t*，它是 SUSv3 为表示定时器标识符而指定的数据类型。

*evp* 参数决定了当定时器到期时，程序如何被通知。它指向一个类型为 *sigevent* 的结构，定义如下：

```
union sigval {
    int   sival_int;              /* Integer value for accompanying data */
    void *sival_ptr;              /* Pointer value for accompanying data */
};

struct sigevent {
    int          sigev_notify;    /* Notification method */
    int          sigev_signo;     /* Timer expiration signal */
    union sigval sigev_value;     /* Value accompanying signal or
                                     passed to thread function */
    union {
        pid_t      _tid;          /* ID of thread to be signaled /
        struct {
            void (*_function) (union sigval);
                                  /* Thread notification function */
            void  *_attribute;    /* Really 'pthread_attr_t *' */
        } _sigev_thread;
    } _sigev_un;
};

#define sigev_notify_function    _sigev_un._sigev_thread._function
#define sigev_notify_attributes  _sigev_un._sigev_thread._attribute
#define sigev_notify_thread_id   _sigev_un._tid
```

该结构的 *sigev_notify* 字段被设置为 表 23-2 中所示的一个值。

表 23-2. *sigev_notify* 字段的值，用于 *sigevent* 结构

| *sigev_notify* 值 | 通知方式 | SUSv3 |
| --- | --- | --- |
| `SIGEV_NONE` | 不通知；使用 *timer_gettime()* 监控定时器 | • |
| `SIGEV_SIGNAL` | 向进程发送信号 *sigev_signo* | • |
| `SIGEV_THREAD` | 调用 *sigev_notify_function* 作为新线程的启动函数 | • |
| `SIGEV_THREAD_ID` | 向线程 *sigev_notify_thread_id* 发送信号 *sigev_signo* |   |

有关*sigev_notify*字段常量的更多详细信息，以及与每个常量值相关的*sigval*结构中的字段，详见以下内容：

`SIGEV_NONE`

不提供计时器到期的通知。进程仍然可以使用*timer_gettime()*监控计时器的进度。

`SIGEV_SIGNAL`

当计时器到期时，为进程生成在*sigev_signo*字段中指定的信号。*sigev_value*字段指定与信号一起传递的数据（一个整数或一个指针）（实时信号队列数的限制）。可以通过传递给该信号处理程序的*siginfo_t*结构的*si_value*字段或通过调用*sigwaitinfo()*或*sigtimedwait()*获取此数据。

`SIGEV_THREAD`

当计时器到期时，调用在*sigev_notify_function*字段中指定的函数。该函数的调用方式*就像*它是在一个新的线程中的启动函数一样。 “就像”这种表述来源于 SUSv3，并允许实现通过将每次通知发送到一个新的唯一线程或将通知按顺序发送到单一的新线程来生成周期性计时器的通知。*sigev_notify_attributes*字段可以指定为`NULL`，或者指定为一个指向*pthread_attr_t*结构的指针，该结构定义了线程的属性（线程属性）。*sigev_value*字段中指定的联合*sigval*值作为该函数的唯一参数传递。

`SIGEV_THREAD_ID`

这与`SIGEV_SIGNAL`类似，但信号被发送到线程 ID 与*sigev_notify_thread_id*匹配的线程。此线程必须与调用线程位于同一进程中。（使用`SIGEV_SIGNAL`通知时，信号会排队到整个进程，如果进程中有多个线程，则信号将被发送到该进程中任意选择的线程。）*sigev_notify_thread_id*字段可以设置为*clone()*返回的值或*gettid()*返回的值。`SIGEV_THREAD_ID`标志是为线程库使用而设计的。（它需要使用`CLONE_THREAD`选项的线程实现，详见示例程序。现代的 NPTL 线程实现使用`CLONE_THREAD`，但较旧的 LinuxThreads 线程实现不使用。）

上述所有常量都在 SUSv3 中定义，除了`SIGEV_THREAD_ID`，它是 Linux 特有的。

*evp*参数可以指定为`NULL`，这相当于将*sigev_notify*指定为`SIGEV_SIGNAL`，将*sigev_signo*指定为`SIGALRM`（这在其他系统上可能不同，因为 SUSv3 仅说“一个默认信号号”），并将*sigev_value.sival_int*指定为计时器 ID。

当前实现中，内核为每个使用*timer_create()*创建的 POSIX 定时器预分配一个排队的实时信号结构。此预分配的目的是确保至少有一个这样的结构可用于在定时器过期时排队信号。这意味着可以创建的 POSIX 定时器的数量受限于可排队的实时信号数量（请参阅实时信号）。

### 启动和关闭定时器：*timer_settime()*

一旦我们创建了定时器，就可以使用*timer_settime()*来启动（开始）或关闭（停止）它。

```
#define _POSIX_C_SOURCE 199309
#include <time.h>

int `timer_settime`(timer_t *timerid*, int *flags*, const struct itimerspec **value*,
                  struct itimerspec **old_value*);
```

### 注意

成功返回 0，失败返回-1。

*timer_settime()*的*timerid*参数是之前调用*timer_create()*返回的定时器句柄。

*value*和*old_value*参数类似于同名的*setitimer()*参数：*value*指定定时器的新设置，而*old_value*用于返回之前的定时器设置（请参阅下面关于*timer_gettime()*的描述）。如果我们不关心之前的设置，可以将*old_value*指定为`NULL`。*value*和*old_value*参数是指向*itimerspec*结构的指针，定义如下：

```
struct itimerspec {
    struct timespec it_interval;    /* Interval for periodic timer */
    struct timespec it_value;       /* First expiration */
};
```

*itimerspec*结构的每个字段本身都是一个*timespec*类型的结构，指定时间值，单位为秒和纳秒：

```
struct timespec {
    time_t tv_sec;                  /* Seconds */
    long   tv_nsec;                 /* Nanoseconds */
};
```

*it_value*字段指定定时器第一次过期的时间。如果*it_interval*的任一子字段非零，则这是一个周期性定时器，在*it_value*指定的初始过期后，将按照这些子字段中指定的频率过期。如果*it_interval*的两个子字段都为 0，则该定时器只会过期一次。

如果*flags*指定为 0，则*value.it_value*相对于调用*timer_settime()*时的时钟值进行解释（即类似于*setitimer()*）。如果*flags*指定为`TIMER_ABSTIME`，则*value.it_value*被解释为绝对时间（即从时钟的零点开始测量）。如果该时间已经过去，定时器将立即过期。

要启动定时器，我们需要调用*timer_settime()*，其中*value.it_value*的一个或两个子字段为非零。如果定时器之前已被启动，*timer_settime()*将替换之前的设置。

如果定时器的值和间隔不是相应时钟分辨率的整数倍（由*clock_getres()*返回），这些值会被四舍五入到下一个分辨率的整数倍。

每次定时器过期时，进程会使用在创建此定时器时定义的*timer_create()*调用中的方法进行通知。如果*it_interval*结构包含非零值，这些值将用于重新加载*it_value*结构。

要关闭定时器，我们需要调用*timer_settime()*，将*value.it_value*的两个字段都指定为 0。

### 获取定时器的当前值：*timer_gettime()*

*timer_gettime()*系统调用返回由*timerid*标识的 POSIX 定时器的间隔和剩余时间。

```
#define _POSIX_C_SOURCE 199309
#include <time.h>

int `timer_gettime`(timer_t *timerid*, struct itimerspec **curr_value*);
```

### 注意

成功时返回 0，出错时返回-1

定时器的间隔以及下次定时器过期之前的时间将以*itimerspec*结构的形式返回，该结构由*curr_value*指向。即使该定时器是使用`TIMER_ABSTIME`作为绝对定时器创建的，*curr_value.it_value*字段也会返回下次定时器过期之前的时间。

如果返回的*curr_value.it_value*结构的两个字段均为 0，则说明定时器当前处于解除状态。如果返回的*curr_value.it_interval*结构的两个字段均为 0，则说明定时器只会在*curr_value.it_value*中给定的时间点过期一次。

### 删除定时器：*timer_delete()*

每个 POSIX 定时器消耗少量系统资源。因此，当我们使用完定时器后，应通过使用*timer_delete()*来释放这些资源，从而移除定时器。

```
#define _POSIX_C_SOURCE 199309
#include <time.h>

int `timer_delete`(timer_t *timerid*);
```

### 注意

成功时返回 0，出错时返回-1

*timerid*参数是通过先前调用*timer_create()*返回的句柄。如果定时器已被启用，则在删除之前会自动解除启用。如果该定时器的过期信号尚未处理，则该信号仍处于挂起状态。（SUSv3 未指定此行为，因此其他 UNIX 实现可能表现不同。）定时器在进程终止时会自动删除。

### 通过信号通知

如果我们选择通过信号接收定时器通知，则可以通过信号处理程序接受该信号，或者通过调用*sigwaitinfo()*或*sigtimedwait()*来接收信号。这两种机制都允许接收进程获取一个*siginfo_t*结构（`SA_SIGINFO` 标志），该结构提供了有关信号的更多信息。（要在信号处理程序中利用此功能，我们在建立处理程序时指定`SA_SIGINFO`标志。）以下字段会在*siginfo_t*结构中设置：

+   *si_signo*：此字段包含由该定时器生成的信号。

+   *si_code*：此字段设置为`SI_TIMER`，表示此信号是由于 POSIX 定时器过期生成的。

+   *si_value*：此字段设置为在使用*timer_create()*创建定时器时，*evp.sigev_value*中提供的值。指定不同的*evp.sigev_value*值可以区分多个定时器的过期情况，这些定时器可能会发送相同的信号。

在调用*timer_create()*时，*evp.sigev_value.sival_ptr*通常会被赋值为同一调用中提供的*timerid*参数的地址（见示例 23-5）。这允许信号处理程序（或*sigwaitinfo()*调用）获取生成该信号的定时器的 ID。（或者，*evp.sigev_value.sival_ptr*也可以被赋值为包含*timerid*的结构的地址，该*timerid*是传递给*timer_create()*的。）

Linux 还在 *siginfo_t* 结构中提供了以下非标准字段：

+   *si_overrun*: 该字段包含此定时器的超限计数（请参见 定时器超限）。

    ### 注意

    Linux 还提供了另一个非标准字段：*si_timerid*。此字段包含一个标识符，系统内部使用它来标识定时器（与 *timer_create()* 返回的 ID 不同）。它对应用程序无用。

示例 23-5 演示了使用信号作为 POSIX 定时器的通知机制。

示例 23-5. 使用信号进行 POSIX 定时器通知

```
`timers/ptmr_sigev_signal.c`
    #define _POSIX_C_SOURCE 199309
    #include <signal.h>
    #include <time.h>
    #include "curr_time.h"                  /* Declares currTime() */
    #include "itimerspec_from_str.h"        /* Declares itimerspecFromStr() */
    #include "tlpi_hdr.h"

    #define TIMER_SIG SIGRTMAX              /* Our timer notification signal */

    static void
 handler(int sig, siginfo_t *si, void *uc)
    {
        timer_t *tidptr;

        tidptr = si->si_value.sival_ptr;

        /* UNSAFE: This handler uses non-async-signal-safe functions
           (printf(); see Section 21.1.2) */

            printf("[%s] Got signal %d\n", currTime("%T"), sig);
        printf("    *sival_ptr         = %ld\n", (long) *tidptr);
        printf("    timer_getoverrun() = %d\n", timer_getoverrun(*tidptr));
    }

    int
    main(int argc, char *argv[])
    {
        struct itimerspec ts;
        struct sigaction  sa;
        struct sigevent   sev;
        timer_t *tidlist;
        int j;

        if (argc < 2)
            usageErr("%s secs[/nsecs][:int-secs[/int-nsecs]]...\n", argv[0]);

        tidlist = calloc(argc - 1, sizeof(timer_t));
        if (tidlist == NULL)
            errExit("malloc");

        /* Establish handler for notification signal */

        sa.sa_flags = SA_SIGINFO;
        sa.sa_sigaction = handler;
        sigemptyset(&sa.sa_mask);
    if (sigaction(TIMER_SIG, &sa, NULL) == -1)
            errExit("sigaction");

        /* Create and start one timer for each command-line argument */

        sev.sigev_notify = SIGEV_SIGNAL;    /* Notify via signal */
        sev.sigev_signo = TIMER_SIG;        /* Notify using this signal */

        for (j = 0; j < argc - 1; j++) {
        itimerspecFromStr(argv[j + 1], &ts);

            sev.sigev_value.sival_ptr = &tidlist[j];
                    /* Allows handler to get ID of this timer */
        if (timer_create(CLOCK_REALTIME, &sev, &tidlist[j]) == -1)
                errExit("timer_create");
            printf("Timer ID: %ld (%s)\n", (long) tidlist[j], argv[j + 1]);
        if (timer_settime(tidlist[j], 0, &ts, NULL) == -1)
                errExit("timer_settime");
        }
    for (;;)                            /* Wait for incoming timer signals */
            pause();
    }
          `timers/ptmr_sigev_signal.c`
```

示例 23-5 中程序的每个命令行参数指定了定时器的初始值和间隔。这些参数的语法在程序的“使用”消息中进行了描述，并在下面的 shell 会话中进行了演示。此程序执行以下步骤：

+   为用于定时器通知的信号建立一个处理程序 ![](img/U002.png)。

+   对于每个命令行参数，创建 ![](img/U004.png) 并启动 ![](img/U005.png) 一个使用 `SIGEV_SIGNAL` 通知机制的 POSIX 定时器。我们用来将命令行参数转换为 *itimerspec* 结构的 *itimerspecFromStr()* 函数在 示例 23-6 中展示。

+   每次定时器到期时，*sev.sigev_signo* 中指定的信号将发送到进程。该信号的处理程序显示在 *sev.sigev_value.sival_ptr* 中提供的值（即定时器 ID，*tidlist[j]*）和定时器的超限值 ![](img/U001.png)。

+   创建并启动定时器后，通过执行一个反复调用 *pause()* 的循环来等待定时器到期！[](figs/web/U006.png)。

示例 23-6 展示了将 示例 23-5 中的每个命令行参数转换为相应的 *itimerspec* 结构的函数。此函数解释的字符串参数格式在列表顶部的注释中显示（并在下面的 shell 会话中进行了演示）。

示例 23-6. 将时间加间隔字符串转换为 *itimerspec* 值

```
`timers/itimerspec_from_str.c`
#define_POSIX_C_SOURCE 199309
#include <string.h>
#include <stdlib.h>
#include "itimerspec_from_str.h"        /* Declares function defined here */

/* Convert a string of the following form to an itimerspec structure:
   "value.sec[/value.nanosec][:interval.sec[/interval.nanosec]]".
   Optional components that are omitted cause 0 to be assigned to the
   corresponding structure fields. */

void
itimerspecFromStr(char *str, struct itimerspec *tsp)
{
    char *cptr, *sptr;

    cptr = strchr(str, ':');
    if (cptr != NULL)
        *cptr = '\0';

    sptr = strchr(str, '/');
    if (sptr != NULL)
        *sptr = '\0';

    tsp->it_value.tv_sec = atoi(str);
    tsp->it_value.tv_nsec = (sptr != NULL) ? atoi(sptr + 1) : 0;

    if (cptr == NULL) {
        tsp->it_interval.tv_sec = 0;
        tsp->it_interval.tv_nsec = 0;
    } else {
        sptr = strchr(cptr + 1, '/');
        if (sptr != NULL)
            *sptr = '\0';
        tsp->it_interval.tv_sec = atoi(cptr + 1);
        tsp->it_interval.tv_nsec = (sptr != NULL) ? atoi(sptr + 1) : 0;
    }
}
     `timers/itimerspec_from_str.c`
```

我们在示例 23-5 中的以下 Shell 会话中演示了该程序的使用，创建了一个初始定时器过期时间为 2 秒、间隔为 5 秒的单一定时器。

```
$ `./ptmr_sigev_signal 2:5`
Timer ID: 134524952 (2:5)
[15:54:56] Got signal 64                  SIGRTMAX *is signal 64 on this system*
    *sival_ptr         = 134524952        *sival_ptr* *points to the variable* *tid*
    timer_getoverrun() = 0
[15:55:01] Got signal 64
    *sival_ptr         = 134524952
    timer_getoverrun() = 0
*Type Control-Z to suspend the process*
[1]+  Stopped       ./ptmr_sigev_signal 2:5
```

在暂停程序后，我们暂停几秒钟，允许几个定时器过期事件发生，然后再恢复程序：

```
$ `fg`
./ptmr_sigev_signal 2:5
[15:55:34] Got signal 64
    *sival_ptr         = 134524952
    timer_getoverrun() = 5
*Type Control-C to kill the program*
```

程序输出的最后一行显示发生了五次定时器溢出，意味着自上次信号传递以来，定时器已过期六次。

### 定时器溢出

假设我们选择通过发送信号来接收定时器过期的通知（即，*sigev_notify* 设置为 `SIGEV_SIGNAL`）。进一步假设，在关联的信号被捕获或接收之前，定时器已经过期多次。这可能是因为进程在下一次调度之前有延迟。或者，可能是由于信号的传递被阻塞，阻塞可能是通过 *sigprocmask()* 显式进行的，也可能是在执行信号处理程序时隐式发生的。那么，我们怎么知道这些 *定时器溢出* 已经发生呢？

我们可能认为使用实时信号有助于解决这个问题，因为多个实时信号实例会排队。然而，这种方法最终证明是行不通的，因为有对可排队的实时信号数量的限制。因此，POSIX.1b 委员会决定采用不同的方法：如果我们选择通过信号接收定时器通知，那么即使使用实时信号，也永远不会排队多个信号实例。相反，在接收到信号后（无论是通过信号处理程序还是使用 *sigwaitinfo()*），我们可以获取 *定时器溢出计数*，即在信号生成和接收之间发生的额外定时器过期次数。例如，如果定时器自上次接收到信号以来已经过期三次，那么溢出计数为 2。

接收到定时器信号后，我们可以通过两种方式获取定时器溢出计数：

+   调用 *timer_getoverrun()*，我们将在下文中进行说明。这是 SUSv3 规范的获取溢出计数的方法。

+   使用信号返回的 *siginfo_t* 结构中的 *si_overrun* 字段的值。这种方法节省了 *timer_getoverrun()* 系统调用的开销，但它是一个非移植的 Linux 扩展。

每次接收到定时器信号时，定时器溢出计数会被重置。如果定时器自上次信号处理或接收以来仅过期了一次，则溢出计数为 0（即，没有溢出）。

```
#define _POSIX_C_SOURCE 199309
#include <time.h>

int `timer_getoverrun`(timer_t *timerid*);
```

### 注意

成功时返回定时器溢出计数，出错时返回-1

*timer_getoverrun()* 函数返回由其 *timerid* 参数指定的定时器的溢出计数。

*timer_getoverrun()*函数是 SUSv3 中规定的异步信号安全函数之一（见表 21-1, 以及信号处理程序中的*errno*使用），因此可以在信号处理程序中安全地调用。

### 通过线程进行通知

`SIGEV_THREAD`标志允许程序通过在单独的线程中调用函数来获取定时器到期的通知。理解此标志需要了解 POSIX 线程，我们将在稍后的第二十九章和第三十章中介绍这些内容。对于不熟悉 POSIX 线程的读者，建议在查看本节中的示例程序之前阅读这两章。

示例 23-7 演示了如何使用`SIGEV_THREAD`。该程序与示例 23-5 中的程序使用相同的命令行参数。该程序执行以下步骤：

+   对于每个命令行参数，程序会创建![](img/U006.png)，并启用![](img/U007.png)一个使用`SIGEV_THREAD`通知机制的 POSIX 定时器![](img/U003.png)。

+   每当该定时器到期时，*sev.sigev_notify_function*![](img/U004.png)指定的函数将在一个单独的线程中被调用。当该函数被调用时，它将接收*sev.sigev_value.sival_ptr*中指定的值作为参数。我们将定时器 ID 的地址(*tidlist[j]*)分配给此字段![](img/U005.png)，这样通知函数就可以获取导致其调用的定时器的 ID。

+   在创建并启用所有定时器后，主程序进入一个循环，等待定时器到期![](img/U008.png)。每次循环时，程序使用*pthread_cond_wait()*等待由处理定时器通知的线程发送的条件变量(*cond*)信号。

+   每当定时器到期时，*threadFunc()* 函数会被调用 ![](img/U001.png)。在打印一条消息后，它会增加全局变量 *expireCnt* 的值。为了考虑定时器溢出的可能性，*timer_getoverrun()* 返回的值也会被加到 *expireCnt* 中。（我们在 定时器溢出 中解释了定时器溢出的概念，并讨论了 `SIGEV_SIGNAL` 通知机制。定时器溢出也可能影响 `SIGEV_THREAD` 机制，因为一个定时器可能在通知函数被调用之前多次到期。）通知函数还会信号条件变量 *cond*，以便主程序知道该检查定时器是否到期 ![](img/U002.png)。

以下 shell 会话日志演示了 示例 23-7 中程序的使用。在此示例中，程序创建了两个定时器：一个初始到期时间为 5 秒，间隔为 5 秒；另一个初始到期时间为 10 秒，间隔为 10 秒。

```
$ `./ptmr_sigev_thread 5:5 10:10`
Timer ID: 134525024 (5:5)
Timer ID: 134525080 (10:10)
[13:06:22] Thread notify
    timer ID=134525024
    timer_getoverrun()=0
main(): count = 1
[13:06:27] Thread notify
    timer ID=134525080
    timer_getoverrun()=0
main(): count = 2
[13:06:27] Thread notify
    timer ID=134525024
    timer_getoverrun()=0
main(): count = 3
*Type Control-Z to suspend the program*
[1]+  Stopped       ./ptmr_sigev_thread 5:5 10:10
$ `fg`                                      *Resume execution*
./ptmr_sigev_thread 5:5 10:10
[13:06:45] Thread notify
    timer ID=134525024
    timer_getoverrun()=2                  *There were timer overruns*
main(): count = 6
[13:06:45] Thread notify
    timer ID=134525080
    timer_getoverrun()=0
main(): count = 7
*Type Control-C to kill the program*
```

示例 23-7. 使用线程函数进行 POSIX 定时器通知

```
`timers/ptmr_sigev_thread.c`
    #include <signal.h>
    #include <time.h>
    #include <pthread.h>
    #include "curr_time.h"              /* Declaration of currTime() */
    #include "tlpi_hdr.h"
    #include "itimerspec_from_str.h"    /* Declares itimerspecFromStr() */

    static pthread_mutex_t mtx = PTHREAD_MUTEX_INITIALIZER;
    static pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

    static int expireCnt = 0;           /* Number of expirations of all timers */

    static void                         /* Thread notification function */
 threadFunc(union sigval sv)
    {
        timer_t *tidptr;
        int s;

        tidptr = sv.sival_ptr;

        printf("[%s] Thread notify\n", currTime("%T"));
        printf("    timer ID=%ld\n", (long) *tidptr);
        printf("    timer_getoverrun()=%d\n", timer_getoverrun(*tidptr));

        /* Increment counter variable shared with main thread and signal
           condition variable to notify main thread of the change. */

        s = pthread_mutex_lock(&mtx);
        if (s != 0)
            errExitEN(s, "pthread_mutex_lock");

        expireCnt += 1 + timer_getoverrun(*tidptr);

        s = pthread_mutex_unlock(&mtx);
        if (s != 0)
            errExitEN(s, "pthread_mutex_unlock");
    s = pthread_cond_signal(&cond);
        if (s != 0)
            errExitEN(s, "pthread_cond_signal");
    }

    int
    main(int argc, char *argv[])
    {
        struct sigevent sev;
        struct itimerspec ts;
        timer_t *tidlist;
        int s, j;

            if (argc < 2)
            usageErr("%s secs[/nsecs][:int-secs[/int-nsecs]]...\n", argv[0]);

        tidlist = calloc(argc - 1, sizeof(timer_t));
        if (tidlist == NULL)
            errExit("malloc");
    sev.sigev_notify = SIGEV_THREAD;            /* Notify via thread */
    sev.sigev_notify_function = threadFunc;     /* Thread start function */
        sev.sigev_notify_attributes = NULL;
                /* Could be pointer to pthread_attr_t structure */

        /* Create and start one timer for each command-line argument */

        for (j = 0; j < argc - 1; j++) {
            itimerspecFromStr(argv[j + 1], &ts);
        sev.sigev_value.sival_ptr = &tidlist[j];
                    /* Passed as argument to threadFunc() */
        if (timer_create(CLOCK_REALTIME, &sev, &tidlist[j]) == -1)
                errExit("timer_create");
            printf("Timer ID: %ld (%s)\n", (long) tidlist[j], argv[j + 1]);
        if (timer_settime(tidlist[j], 0, &ts, NULL) == -1)
                errExit("timer_settime");
        }

        /* The main thread waits on a condition variable that is signaled
           on each invocation of the thread notification function. We
           print a message so that the user can see that this occurred. */

        s = pthread_mutex_lock(&mtx);
        if (s != 0)
            errExitEN(s, "pthread_mutex_lock");
    for (;;) {
            s = pthread_cond_wait(&cond, &mtx);
            if (s != 0)
                errExitEN(s, "pthread_cond_wait");
            printf("main(): expireCnt = %d\n", expireCnt);
        }
    }

          `timers/ptmr_sigev_thread.c`
```

## 通过文件描述符进行通知的定时器：*timerfd* API

从内核版本 2.6.25 开始，Linux 提供了另一种创建定时器的 API。Linux 特有的 *timerfd* API 创建一个定时器，其到期通知可以从文件描述符中读取。这是有用的，因为可以通过 *select()*、*poll()* 和 *epoll*（参见 第六十三章）监控该文件描述符与其他描述符一起使用。（对于本章讨论的其他定时器 API，需要一些额外的工作才能同时监控一个或多个定时器以及一组文件描述符。）

该 API 中三个新系统调用的操作类似于第 23.6 节中描述的 *timer_create()*、*timer_settime()* 和 *timer_gettime()* 系统调用的操作。

新系统调用中的第一个是 *timerfd_create()*，它创建一个新的定时器对象并返回一个指向该对象的文件描述符。

```
#include <sys/timerfd.h>

int `timerfd_create`(int *clockid*, int *flags*);
```

### 注意

成功时返回文件描述符，出错时返回-1

*clockid* 的值可以是 `CLOCK_REALTIME` 或 `CLOCK_MONOTONIC`（参见 表 23-1）。

在 *timerfd_create()* 的初始实现中，*flags* 参数预留供将来使用，必须指定为 0。然而，从 Linux 2.6.27 开始，支持两个标志：

`TFD_CLOEXEC`

为新的文件描述符设置关闭执行标志（`FD_CLOEXEC`）。此标志的作用与文件描述符号由 *open()* 返回 返回")中描述的 *open()* `O_CLOEXEC` 标志相同。

`TFD_NONBLOCK`

设置底层打开文件描述符的 `O_NONBLOCK` 标志，使得未来的读取操作为非阻塞模式。这避免了为实现相同结果而额外调用 *fcntl()*。

当我们使用完通过 *timerfd_create()* 创建的定时器后，应当 *close()* 关联的文件描述符，以便内核可以释放与定时器相关的资源。

*timerfd_settime()* 系统调用启动（开始）或停止（停止）由文件描述符 *fd* 引用的定时器。

```
#include <sys/timerfd.h>

int `timerfd_settime`(int *fd*, int *flags*, const struct itimerspec **new_value*,
                    struct itimerspec **old_value*);
```

### 注意

成功时返回 0，错误时返回 -1

*new_value* 参数指定定时器的新设置。*old_value* 参数可以用于返回定时器的先前设置（详细信息见下面的 *timerfd_gettime()* 描述）。如果我们不关心先前的设置，可以将 *old_value* 指定为 `NULL`。这两个参数都是 *itimerspec* 结构体，使用方式与 *timer_settime()* 相同（详见 定时器的启动和停止：*timer_settime()*")）。

*flags* 参数类似于 *timer_settime()* 的对应参数。它可以为 0，表示 *new_value.it_value* 相对于调用 *timerfd_settime()* 的时间进行解释，或者可以为 `TFD_TIMER_ABSTIME`，表示 *new_value.it_value* 被解释为绝对时间（即从时钟的零点开始计算）。

*timerfd_gettime()* 系统调用返回由文件描述符 *fd* 标识的定时器的间隔和剩余时间。

```
#include <sys/timerfd.h>

int `timerfd_gettime`(int *fd*, struct itimerspec **curr_value*);
```

### 注意

成功时返回 0，错误时返回 -1

与 *timer_gettime()* 一样，定时器的间隔和下次过期时间会返回在 *itimerspec* 结构体中，该结构体由 *curr_value* 指向。*curr_value.it_value* 字段返回下次定时器过期的时间，即使该定时器是通过 `TFD_TIMER_ABSTIME` 作为绝对定时器建立的。如果返回的 *curr_value.it_value* 结构体的两个字段都为 0，则表示定时器当前已被解除。如果返回的 *curr_value.it_interval* 结构体的两个字段都为 0，则定时器只会过期一次，时间为 *curr_value.it_value* 中给定的时间。

#### *timerfd* 与 *fork()* 和 *exec()* 的交互

在 *fork()* 过程中，子进程会继承 *timerfd_create()* 创建的文件描述符的副本。这些文件描述符指向与父进程中相应描述符相同的定时器对象，并且定时器过期事件可以在任一进程中读取。

由 *timerfd_create()* 创建的文件描述符在 *exec()* 跨进程调用时会被保留（除非描述符被标记为 close-on-exec，如 文件描述符和 *exec()*") 中所述），并且定时器在 *exec()* 后仍会继续生成定时器过期事件。

#### 从 *timerfd* 文件描述符读取

一旦使用*timerfd_settime()*启动了定时器，我们可以使用*read()*从关联的文件描述符中读取定时器过期事件的信息。为此，传递给*read()*的缓冲区必须足够大，以容纳一个无符号的 8 字节整数（*uint64_t*）。

如果自从使用*timerfd_settime()*最后一次修改定时器设置或最后一次执行*read()*以来发生了一个或多个过期事件，那么*read()*会立即返回，返回的缓冲区包含已经发生的过期事件数。如果没有定时器过期事件发生，*read()*则会阻塞，直到下一次过期事件发生。也可以使用*fcntl()*的`F_SETFL`操作（打开文件状态标志）为文件描述符设置`O_NONBLOCK`标志，这样读取操作就变为非阻塞模式，并在没有定时器过期事件时返回`EAGAIN`错误。

如前所述，*timerfd*文件描述符可以通过*select()*、*poll()*和*epoll*进行监控。如果定时器已过期，则文件描述符会被标记为可读。

#### 示例程序

示例 23-8 演示了如何使用*timerfd* API。这个程序接受两个命令行参数。第一个参数是必需的，指定定时器的初始时间和间隔。（该参数通过示例 23-6 中展示的*itimerspecFromStr()*函数进行解释。）第二个参数是可选的，指定程序在终止之前应等待的定时器最大过期次数；该参数的默认值是 1。

程序使用*timerfd_create()*创建一个定时器，并使用*timerfd_settime()*进行启动。然后，它进入循环，从文件描述符中读取过期通知，直到达到指定的过期次数。在每次*read()*之后，程序会显示自定时器启动以来经过的时间、已读取的过期次数以及到目前为止的总过期次数。

在以下的 shell 会话日志中，命令行参数指定了一个初始值为 1 秒、间隔为 1 秒且最大过期次数为 100 的定时器。

```
$ `./demo_timerfd 1:1 100`
1.000: expirations read: 1; total=1
2.000: expirations read: 1; total=2
3.000: expirations read: 1; total=3
*Type Control-Z to suspend program in background for a few seconds*
[1]+  Stopped           ./demo_timerfd 1:1 100
$ `fg`                                      *Resume program in foreground*
./demo_timerfd 1:1 100
14.205: expirations read: 11; total=14    *Multiple expirations since last* *read()*
15.000: expirations read: 1; total=15
16.000: expirations read: 1; total=16
*Type Control-C to terminate the program*
```

从上面的输出可以看出，当程序在后台挂起时，发生了多个定时器过期事件，并且这些过期事件在程序恢复执行后的第一次*read()*中全部返回。

示例 23-8. 使用*timerfd* API

```
`timers/demo_timerfd.c`
#include <sys/timerfd.h>
#include <time.h>
#include <stdint.h>                     /* Definition of uint64_t */
#include "itimerspec_from_str.h"        /* Declares itimerspecFromStr() */
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    struct itimerspec ts;
    struct timespec start, now;
    int maxExp, fd, secs, nanosecs;
    uint64_t numExp, totalExp;
    ssize_t s;

    if (argc < 2 || strcmp(argv[1], "--help") == 0)
        usageErr("%s secs[/nsecs][:int-secs[/int-nsecs]] [max-exp]\n", argv[0]);

    itimerspecFromStr(argv[1], &ts);
    maxExp = (argc > 2) ? getInt(argv[2], GN_GT_0, "max-exp") : 1;

    fd = timerfd_create(CLOCK_REALTIME, 0);
    if (fd == -1)
        errExit("timerfd_create");

    if (timerfd_settime(fd, 0, &ts, NULL) == -1)
        errExit("timerfd_settime");

    if (clock_gettime(CLOCK_MONOTONIC, &start) == -1)
        errExit("clock_gettime");

    for (totalExp = 0; totalExp < maxExp;) {

        /* Read number of expirations on the timer, and then display
           time elapsed since timer was started, followed by number
           of expirations read and total expirations so far. */

        s = read(fd, &numExp, sizeof(uint64_t));
        if (s != sizeof(uint64_t))
            errExit("read");

        totalExp += numExp;

        if (clock_gettime(CLOCK_MONOTONIC, &now) == -1)
            errExit("clock_gettime");

        secs = now.tv_sec - start.tv_sec;
        nanosecs = now.tv_nsec - start.tv_nsec;
        if (nanosecs < 0) {
            secs--;
            nanosecs += 1000000000;
        }

        printf("%d.%03d: expirations read: %llu; total=%llu\n",
                secs, (nanosecs + 500000) / 1000000,
                (unsigned long long) numExp, (unsigned long long) totalExp);
    }

    exit(EXIT_SUCCESS);
}
     `timers/demo_timerfd.c`
```

## 总结

进程可以使用*setitimer()*或*alarm()*设置定时器，以便在经过指定的真实时间或进程时间后接收到信号。定时器的一种用途是设置系统调用的阻塞时间上限。

需要在指定的实际时间间隔内暂停执行的应用程序可以使用多种休眠函数来实现这一目的。

Linux 2.6 实现了 POSIX.1b 扩展，定义了一个用于高精度时钟和定时器的 API。POSIX.1b 定时器相比传统的 (*setitimer()*) UNIX 定时器提供了许多优点。我们可以：创建多个定时器；选择定时器过期时发送的信号；获取定时器溢出次数，以确定定时器是否自上次过期通知以来已过期多次；并选择通过执行线程函数而不是发送信号来接收定时器通知。

Linux 特定的 *timerfd* API 提供了一组用于创建定时器的接口，这些接口类似于 POSIX 定时器 API，但允许通过文件描述符读取定时器通知。可以使用 *select()*、*poll()* 和 *epoll* 来监视该文件描述符。

#### 进一步的信息

根据各个函数的原理，SUSv3 提供了关于本章描述的（标准）定时器和休眠接口的有用说明。[Gallmeister, 1995] 讨论了 POSIX.1b 时钟和定时器。

## 练习

1.  尽管 *alarm()* 是作为 Linux 内核中的系统调用实现的，但这是多余的。使用 *setitimer()* 来实现 *alarm()*。

1.  尝试在后台运行程序 Example 23-3") (`t_nanosleep.c`)，并使用以下命令向后台进程发送尽可能多的 `SIGINT` 信号，同时设置 60 秒的休眠间隔：

    ```
    $ `while true; do kill -INT` ``*`pid`*```; done`

    ```

    你应该观察到程序的休眠时间比预期的要长。将 *nanosleep()* 替换为 *clock_gettime()*（使用 `CLOCK_REALTIME` 时钟）和 *clock_nanosleep()*（使用 `TIMER_ABSTIME` 标志）。(此练习需要 Linux 2.6。) 使用修改后的程序重复测试，并解释差异。

1.  编写一个程序，展示如果 *timer_create()* 的 *evp* 参数指定为 `NULL`，则等同于指定 *evp* 为指向一个 *sigevent* 结构体的指针，其中 *sigev_notify* 设置为 `SIGEV_SIGNAL`，*sigev_signo* 设置为 `SIGALRM`，并且 *si_value.sival_int* 设置为定时器 ID。

1.  修改程序 Example 23-5 (`ptmr_sigev_signal.c`)，使用 *sigwaitinfo()* 代替信号处理程序。
