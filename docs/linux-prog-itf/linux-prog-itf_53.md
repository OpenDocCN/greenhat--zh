## 第五十三章。POSIX 信号量

本章描述了 POSIX 信号量，它允许进程和线程同步访问共享资源。在第四十七章中，我们描述了 System V 信号量，并假设读者熟悉本章开始时所介绍的一般信号量概念和使用信号量的理由。在本章过程中，我们将比较 POSIX 信号量和 System V 信号量，以澄清这两种信号量 API 在相同之处和不同之处。

## 概述

SUSv3 指定了两种类型的 POSIX 信号量：

+   *命名信号量*：这种信号量有一个名称。通过使用相同名称调用 *sem_open()*，不同的进程可以访问同一个信号量。

+   *无名信号量*：这种信号量没有名称，而是位于内存中的一个约定位置。无名信号量可以在进程之间或线程组之间共享。当进程之间共享时，信号量必须位于（System V、POSIX 或 *mmap()*）共享内存区域。当线程之间共享时，信号量可以位于线程共享的内存区域（例如堆或全局变量中）。

POSIX 信号量的操作方式类似于 System V 信号量；也就是说，POSIX 信号量是一个整数，其值不能低于 0。如果进程尝试将信号量的值减少到 0 以下，则根据使用的函数，调用要么阻塞，要么因操作当前无法完成而失败并返回错误。

一些系统未提供完整的 POSIX 信号量实现。一个典型的限制是，只支持无名线程共享信号量。Linux 2.4 就是这种情况；而在 Linux 2.6 及提供 NPTL 的 *glibc* 中，提供了完整的 POSIX 信号量实现。

### 注意

在带有 NPTL 的 Linux 2.6 上，信号量操作（增减）是通过 *futex(2)* 系统调用实现的。

## 命名信号量

要与命名信号量一起使用，我们使用以下函数：

+   *sem_open()* 函数打开或创建一个信号量，如果是通过该调用创建的信号量，则会初始化它，并返回一个句柄以供后续调用使用。

+   *sem_post(sem)* 和 *sem_wait(sem)* 函数分别增加和减少信号量的值。

+   *sem_getvalue()* 函数检索信号量的当前值。

+   *sem_close()* 函数移除调用进程与先前打开的信号量的关联。

+   *sem_unlink()* 函数移除信号量的名称，并在所有进程关闭该信号量后将其标记为删除。

SUSv3 并没有指定命名信号量的实现方式。某些 UNIX 实现将它们作为文件创建在标准文件系统的特定位置。在 Linux 中，它们作为小型 POSIX 共享内存对象创建，名称形式为`sem.`*name*，并存放在专用的*tmpfs*文件系统中（虚拟内存文件系统：*tmpfs*），挂载在`/dev/shm`目录下。该文件系统具有内核持久性——其中包含的信号量对象会持续存在，即使没有任何进程打开它们，但如果系统关闭，这些对象会丢失。

自 Linux 内核 2.6 起，支持命名信号量。

### 打开命名信号量

*sem_open()*函数用于创建并打开一个新的命名信号量，或者打开一个已有的信号量。

```
#include <fcntl.h>            /* Defines O_* constants */
#include <sys/stat.h>         /* Defines mode constants */
#include <semaphore.h>

sem_t *`sem_open`(const char **name*, int *oflag*, ...
                /* mode_t *mode*, unsigned int *value* */ );
```

### 注意

成功时返回指向信号量的指针，出错时返回`SEM_FAILED`

*name*参数标识信号量。它的指定规则见第 51.1 节。

*oflag*参数是一个位掩码，用于确定我们是打开一个已有的信号量，还是创建并打开一个新的信号量。如果*oflag*为 0，则我们正在访问一个已有的信号量。如果在*oflag*中指定了`O_CREAT`，则如果给定*name*的信号量不存在，就会创建一个新的信号量。如果*oflag*同时指定了`O_CREAT`和`O_EXCL`，且给定*name*的信号量已存在，则*sem_open()*会失败。

如果使用*sem_open()*来打开一个已有的信号量，调用只需要两个参数。然而，如果*flags*中指定了`O_CREAT`，则需要两个额外的参数：*mode*和*value*。（如果由*name*指定的信号量已经存在，则这两个参数会被忽略。）这些参数如下：

+   *mode*参数是一个位掩码，用于指定新信号量的权限。位值与文件相同（表 15-4，常规文件的权限），并且与*open()*一样，*mode*中的值会与进程的 umask 进行掩码处理（进程文件模式创建掩码：*umask()*")）。SUSv3 并未为*oflag*指定任何访问模式标志（`O_RDONLY`、`O_WRONLY`和`O_RDWR`）。许多实现，包括 Linux，在打开信号量时默认使用`O_RDWR`访问模式，因为大多数使用信号量的应用程序必须同时使用*sem_post()*和*sem_wait()*，这涉及读取和修改信号量的值。这意味着我们应该确保每个需要访问信号量的用户类别——所有者、组和其他——都被授予读写权限。

+   *value*参数是一个无符号整数，指定分配给新信号量的初始值。信号量的创建和初始化是原子的。这避免了初始化 System V 信号量时所需的复杂性（信号量初始化）。

无论是创建新信号量还是打开现有信号量，*sem_open()*都会返回指向*sem_t*值的指针，我们在后续对信号量的操作中使用此指针。如果出错，*sem_open()*返回`SEM_FAILED`值。（在大多数实现中，`SEM_FAILED`被定义为*((sem_t *) 0)*或*((sem_t *) -1)*；Linux 定义为前者。）

SUSv3 规定，如果我们尝试对由*sem_open()*返回值指向的*sem_t*变量的*副本*执行操作（*sem_post()*, *sem_wait()*等），结果是未定义的。换句话说，以下对*sem2*的使用是不允许的：

```
sem_t *sp, sem2
sp = sem_open(...);
sem2 = *sp;
sem_wait(&sem2);
```

当通过*fork()*创建子进程时，子进程会继承父进程中打开的所有命名信号量的引用。在*fork()*之后，父进程和子进程可以使用这些信号量来同步它们的操作。

#### 示例程序

示例 53-1 打开或创建 POSIX 命名信号量")中的程序提供了一个简单的命令行界面来调用*sem_open()*函数。该程序的命令格式如*usageError()*函数中所示。

以下是一个示例的 Shell 会话日志，演示了该程序的使用。我们首先使用*umask*命令拒绝其他用户类的所有权限。然后，我们独占创建一个信号量并检查包含命名信号量的 Linux 特定虚拟目录的内容。

```
$ `umask 007`
$ `./psem_create -cx /demo 666`             *666 means read+write for all users*
$ `ls -l /dev/shm/sem.*`
-rw-rw----  1 mtk users 16 Jul  6 12:09 /dev/shm/sem.demo
```

*ls*命令的输出显示，进程的 umask 覆盖了用户类“其他”的读取和写入权限。

如果我们再试一次以相同的名称独占创建信号量，操作会失败，因为该名称已经存在。

```
$ `./psem_create -cx /demo 666`
ERROR [EEXIST File exists] sem_open       *Failed because of* O_EXCL
```

示例 53-1. 使用*sem_open()*打开或创建 POSIX 命名信号量

```
`psem/psem_create.c`
#include <semaphore.h>
#include <sys/stat.h>
#include <fcntl.h>
#include "tlpi_hdr.h"

static void
usageError(const char *progName)
{
    fprintf(stderr, "Usage: %s [-cx] name [octal-perms [value]]\n", progName);
    fprintf(stderr, "    -c   Create semaphore (O_CREAT)\n");
    fprintf(stderr, "    -x   Create exclusively (O_EXCL)\n");
    exit(EXIT_FAILURE);
}

int
main(int argc, char *argv[])
{
    int flags, opt;
    mode_t perms;
    unsigned int value;
    sem_t *sem;

    flags = 0;
    while ((opt = getopt(argc, argv, "cx")) != -1) {
        switch (opt) {
        case 'c':   flags |= O_CREAT;           break;
        case 'x':   flags |= O_EXCL;            break;
        default:    usageError(argv[0]);
        }
    }

    if (optind >= argc)
        usageError(argv[0]);

    /* Default permissions are rw-------; default semaphore initialization
       value is 0 */

    perms = (argc <= optind + 1) ? (S_IRUSR | S_IWUSR) :
                getInt(argv[optind + 1], GN_BASE_8, "octal-perms");
    value = (argc <= optind + 2) ? 0 : getInt(argv[optind + 2], 0, "value");

    sem = sem_open(argv[optind], flags, perms, value);
    if (sem == SEM_FAILED)
        errExit("sem_open");

    exit(EXIT_SUCCESS);
}
    `psem/psem_create.c`
```

### 关闭信号量

当一个进程打开一个命名信号量时，系统会记录进程与信号量之间的关联。*sem_close()*函数终止此关联（即关闭信号量），释放系统为该进程与信号量关联的任何资源，并减少引用该信号量的进程数量。

```
#include <semaphore.h>

int `sem_close`(sem_t **sem*);
```

### 注意

成功时返回 0，错误时返回-1

打开的命名信号量在进程终止时或进程执行*exec()*时会自动关闭。

关闭信号量并不会删除它。为此，我们需要使用*sem_unlink()*。

### 移除命名信号量

*sem_unlink()* 函数移除由 *name* 标识的信号量，并在所有进程停止使用它后标记该信号量为销毁状态（如果所有打开该信号量的进程已经关闭，它可能立即销毁）。

```
#include <semaphore.h>

int `sem_unlink`(const char **name*);
```

### 注意

成功时返回 0，错误时返回 -1

示例 53-2 来解除绑定 POSIX 命名信号量") 演示了如何使用 *sem_unlink()*。

示例 53-2. 使用 *sem_unlink()* 来解除绑定 POSIX 命名信号量

```
`psem/psem_unlink.c`
#include <semaphore.h>
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    if (argc != 2 || strcmp(argv[1], "--help") == 0)
        usageErr("%s sem-name\n", argv[0]);

    if (sem_unlink(argv[1]) == -1)
        errExit("sem_unlink");
    exit(EXIT_SUCCESS);
}
    `psem/psem_unlink.c`
```

## 信号量操作

与 System V 信号量一样，POSIX 信号量是一个整数，系统永远不会允许它小于 0。然而，POSIX 信号量操作与 System V 信号量操作在以下方面有所不同：

+   用于改变信号量值的函数—*sem_post()* 和 *sem_wait()*—每次只操作一个信号量。相比之下，System V 的 *semop()* 系统调用可以在一个信号量集合中操作多个信号量。

+   *sem_post()* 和 *sem_wait()* 函数将信号量的值分别增加和减少一个单位。相比之下，*semop()* 可以加减任意值。

+   没有类似于 System V 信号量提供的等待零操作（*semop()* 调用中 *sops.sem_op* 字段指定为 0）的功能。

从这个列表来看，可能会觉得 POSIX 信号量不如 System V 信号量强大。然而事实并非如此——我们用 System V 信号量可以做的任何事情，也能用 POSIX 信号量实现。在某些情况下，可能需要多一些编程工作，但在典型的场景下，使用 POSIX 信号量实际上需要更少的编程工作。（System V 信号量 API 比大多数应用需要的要复杂得多。）

### 等待信号量

*sem_wait()* 函数递减（减少 1）由 *sem* 引用的信号量的值。

```
#include <semaphore.h>

int `sem_wait`(sem_t **sem*);
```

### 注意

成功时返回 0，错误时返回 -1

如果信号量当前的值大于 0，*sem_wait()* 会立即返回。如果信号量的值当前为 0，*sem_wait()* 会阻塞直到信号量值升高到 0 以上；此时，信号量值会被递减，*sem_wait()* 返回。

如果一个被阻塞的 *sem_wait()* 调用被信号处理程序中断，那么它会以错误 `EINTR` 失败，无论在使用 *sigaction()* 设置信号处理程序时是否使用了 `SA_RESTART` 标志。（在某些其他 UNIX 实现中，`SA_RESTART` 会导致 *sem_wait()* 自动重启。）

示例 53-3 来递减一个 POSIX 信号量") 中的程序提供了一个命令行接口来使用 *sem_wait()* 函数。我们稍后会展示如何使用这个程序。

示例 53-3. 使用 *sem_wait()* 来递减一个 POSIX 信号量

```
`psem/psem_wait.c`
#include <semaphore.h>
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    sem_t *sem;

    if (argc < 2 || strcmp(argv[1], "--help") == 0)
        usageErr("%s sem-name\n", argv[0]);

    sem = sem_open(argv[1], 0);
    if (sem == SEM_FAILED)
        errExit("sem_open");

    if (sem_wait(sem) == -1)
        errExit("sem_wait");

    printf("%ld sem_wait() succeeded\n", (long) getpid());
    exit(EXIT_SUCCESS);
}
    `psem/psem_wait.c`
```

*sem_trywait()* 函数是 *sem_wait()* 的非阻塞版本。

```
#include <semaphore.h>

int `sem_trywait`(sem_t **sem*);
```

### 注意

成功时返回 0，错误时返回 -1

如果递减操作无法立即执行，*sem_trywait()* 将因错误 `EAGAIN` 而失败。

*sem_timedwait()* 函数是 *sem_wait()* 的另一种变体。它允许调用者指定调用阻塞的时间限制。

```
#include <semaphore.h>

int `sem_timedwait`(sem_t **sem*, const struct timespec **abs_timeout*);
```

### 注意

成功时返回 0，出错时返回 -1

如果 *sem_timedwait()* 调用超时且无法递减信号量，则该调用将失败，并返回错误 `ETIMEDOUT`。

*abs_timeout* 参数是一个 *timespec* 结构体（高分辨率睡眠：*nanosleep()*")），它指定自纪元以来的绝对超时时间，单位为秒和纳秒。如果我们想执行相对超时，则必须使用 *clock_gettime()* 获取当前的 `CLOCK_REALTIME` 时钟值，并将所需的时间量加到该值上，生成一个适用于 *sem_timedwait()* 的 *timespec* 结构体。

*sem_timedwait()* 函数最初在 POSIX.1d（1999）中定义，并非所有 UNIX 实现都支持该函数。

### 发布信号量

*sem_post()* 函数将信号量 *sem* 的值递增（增加 1）。

```
#include <semaphore.h>

int `sem_post`(sem_t **sem*);
```

### 注意

成功时返回 0，出错时返回 -1

如果在 *sem_post()* 调用之前信号量的值为 0，并且其他进程（或线程）被阻塞等待递减信号量，那么该进程将被唤醒，且其 *sem_wait()* 调用将继续递减信号量。如果多个进程（或线程）在 *sem_wait()* 中被阻塞，则如果进程按照默认的轮询时间分配策略进行调度，则无法确定哪个进程将被唤醒并允许递减信号量。（像 System V 信号量一样，POSIX 信号量仅仅是一个同步机制，而不是排队机制。）

### 注意

SUSv3 规定，如果进程或线程在实时调度策略下执行，则将唤醒等待时间最长且优先级最高的进程或线程。

与 System V 信号量一样，递增 POSIX 信号量意味着释放一些共享资源，以供其他进程或线程使用。

示例 53-4 来递增 POSIX 信号量") 中的程序提供了一个命令行接口，用于调用 *sem_post()* 函数。我们稍后会演示如何使用这个程序。

示例 53-4. 使用 *sem_post()* 来递增 POSIX 信号量

```
`psem/psem_post.c`
#include <semaphore.h>
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    sem_t *sem;

    if (argc != 2)
        usageErr("%s sem-name\n", argv[0]);

    sem = sem_open(argv[1], 0);
    if (sem == SEM_FAILED)
        errExit("sem_open");

    if (sem_post(sem) == -1)
        errExit("sem_post");
    exit(EXIT_SUCCESS);
}
    `psem/psem_post.c`
```

### 获取信号量的当前值

*sem_getvalue()* 函数返回由 *sem* 引用的信号量的当前值，该值存储在 *sval* 所指向的 *int* 中。

```
#include <semaphore.h>

int `sem_getvalue`(sem_t **sem*, int **sval*);
```

### 注意

成功时返回 0，出错时返回 -1

如果一个或多个进程（或线程）当前被阻塞，等待递减信号量的值，那么返回的值在 *sval* 中取决于实现。SUSv3 允许两种可能性：0 或一个负数，其绝对值是被 *sem_wait()* 阻塞的等待进程数。Linux 和其他一些实现采用前者；另外一些实现采用后者。

### 注意

尽管如果有阻塞的等待进程，返回负的 *sval* 很有用，特别是对于调试目的，但 SUSv3 并不要求这种行为，因为一些系统为了高效实现 POSIX 信号量所使用的技术并不（实际上无法）记录被阻塞的等待进程数量。

请注意，在 *sem_getvalue()* 返回时，*sval* 中返回的值可能已经过时。一个依赖于 *sem_getvalue()* 返回的信息在后续操作时不变的程序，将会面临检查时与使用时的竞争条件（小心信号与竞争条件）。

示例 53-5 to retrieve the value of a POSIX semaphore") 中的程序使用 *sem_getvalue()* 来检索命令行参数中指定的信号量的值，然后将该值显示在标准输出上。

示例 53-5. 使用 *sem_getvalue()* 检索 POSIX 信号量的值

```
`psem/psem_getvalue.c`
#include <semaphore.h>
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    int value;
    sem_t *sem;

    if (argc != 2)
        usageErr("%s sem-name\n", argv[0]);

    sem = sem_open(argv[1], 0);
    if (sem == SEM_FAILED)
        errExit("sem_open");

    if (sem_getvalue(sem, &value) == -1)
        errExit("sem_getvalue");

    printf("%d\n", value);
    exit(EXIT_SUCCESS);
}
    `psem/psem_getvalue.c`
```

#### 示例

以下 shell 会话日志演示了我们在本章中展示的程序的使用。我们首先创建一个初始值为零的信号量，然后启动一个后台程序，尝试递减该信号量：

```
$ `./psem_create -c /demo 600 0`
$ `./psem_wait /demo &`
[1] 31208
```

后台命令被阻塞，因为信号量当前的值为 0，因此无法递减。

然后我们检索信号量的值：

```
$ `./psem_getvalue /demo`
0
```

我们看到上面的值为 0。在一些其他实现中，我们可能会看到值为 -1，表示有一个进程正在等待该信号量。

然后我们执行一个命令来递增信号量。这将导致后台程序中的被阻塞的 *sem_wait()* 操作完成：

```
$ `./psem_post /demo`
$ 31208 sem_wait() succeeded
```

（上面的最后一行输出显示了 shell 提示符与后台作业的输出混合在一起。）

我们按下 *Enter* 键以查看下一个 shell 提示符，这也会导致 shell 报告已终止的后台作业，并随后对信号量执行进一步的操作：

```
*Press Enter*
[1]-  Done          ./psem_wait /demo
$ `./psem_post /demo` *Increment semaphore*
$ `./psem_getvalue /demo` *Retrieve semaphore value*
1
$ `./psem_unlink /demo` *We’re done with this semaphore*
```

## 无名信号量

无名信号量（也叫做 *基于内存的信号量*）是类型为 *sem_t* 的变量，这些变量存储在应用程序分配的内存中。通过将信号量放置在它们共享的内存区域中，使进程或线程能够使用它。

对无名信号量的操作使用与操作命名信号量相同的函数（*sem_wait()*、*sem_post()*、*sem_getvalue()* 等）。此外，还需要两个额外的函数：

+   *sem_init()* 函数初始化信号量，并告知系统该信号量是要在进程间共享还是在单个进程的线程间共享。

+   *sem_destroy(sem)* 函数用于销毁信号量。

这些函数不应与命名信号量一起使用。

#### 无名信号量与命名信号量

使用无名信号量可以避免为信号量创建名称的工作。这在以下情况下特别有用：

+   线程间共享的信号量不需要名称。将无名信号量作为共享（全局或堆）变量会自动使其对所有线程可访问。

+   在相关进程间共享的信号量不需要名称。如果父进程在共享内存区域（例如共享匿名映射）中分配了无名信号量，则子进程会自动继承该映射，因此信号量也会作为 *fork()* 操作的一部分被继承。

+   如果我们正在构建一个动态数据结构（例如二叉树），每个元素都需要一个关联的信号量，那么最简单的方法是在每个元素内分配一个无名信号量。为每个元素打开一个命名信号量将需要我们设计一种约定，用于为每个元素生成（唯一的）信号量名称，并管理这些名称（例如，在不再需要时取消链接它们）。

### 初始化一个无名信号量

*sem_init()* 函数将指向 *sem* 的无名信号量初始化为 *value* 指定的值。

```
#include <semaphore.h>

int `sem_init`(sem_t **sem*, int *pshared*, unsigned int *value*);
```

### 注意

成功时返回 0，错误时返回 -1。

*pshared* 参数指示信号量是否在线程间或进程间共享。

+   如果 *pshared* 为 0，则信号量将在调用进程的线程间共享。在这种情况下，*sem* 通常指定为全局变量或在堆上分配的变量的地址。线程共享的信号量具有进程持久性；当进程终止时，它将被销毁。

+   如果 *pshared* 非零，则信号量将在进程间共享。在这种情况下，*sem* 必须是指向共享内存区域（POSIX 共享内存对象、通过 *mmap()* 创建的共享映射或系统 V 共享内存段）中某个位置的地址。信号量的生命周期与其所在的共享内存的生命周期相同。（大多数这些技术创建的共享内存区域具有内核持久性，唯一的例外是共享匿名映射，它们只在至少有一个进程保持映射时才会持久存在。）由于通过 *fork()* 创建的子进程会继承父进程的内存映射，因此进程共享信号量会被 *fork()* 的子进程继承，父子进程可以使用这些信号量来同步它们的操作。

*pshared* 参数对于以下原因是必要的：

+   一些实现不支持进程共享信号量。在这些系统中，为*pshared*指定非零值会导致*sem_init()*返回错误。直到内核 2.6 和 NPTL 线程实现的出现，Linux 才开始支持无名进程共享信号量。（在较旧的 LinuxThreads 实现中，如果为*pshared*指定非零值，*sem_init()*会失败并返回`ENOSYS`错误。）

+   在支持进程共享和线程共享信号量的实现中，可能需要指定所需的共享类型，因为系统必须采取特殊措施来支持所请求的共享。提供这些信息还可能允许系统根据共享类型执行优化。

NPTL 的*sem_init()*实现忽略*pshared*，因为对于任何类型的共享，都不需要特殊操作。然而，可移植且具有前瞻性的应用程序应为*pshared*指定一个适当的值。

### 注意

SUSv3 对*sem_init()*的规范定义了失败时返回-1，但没有说明成功时的返回值。然而，大多数现代 UNIX 实现的手册页上记录了成功时返回 0。（一个显著的例外是 Solaris，其中返回值的描述类似于 SUSv3 的规范。然而，通过检查 OpenSolaris 的源代码可以发现，在该实现中，*sem_init()*在成功时确实返回 0。）SUSv4 修正了这一情况，明确规定*sem_init()*在成功时应返回 0。

无名信号量没有与之关联的权限设置（即，*sem_init()*没有类似*sem_open()*中*mode*参数的功能）。对无名信号量的访问受到底层共享内存区域赋予进程的权限控制。

SUSv3 规定，如果初始化一个已经初始化过的无名信号量，将导致未定义的行为。换句话说，我们必须设计我们的应用程序，确保只有一个进程或线程调用*sem_init()*来初始化信号量。

与命名信号量一样，SUSv3 规定，如果我们尝试对传递给*sem_init()*的*sem*参数所指向的*sem_t*变量的*副本*进行操作，结果是未定义的。操作应始终仅对“原始”信号量进行。

#### 示例程序

在锁定和解锁互斥量中，我们展示了一个程序（示例 30-2），该程序使用互斥量保护临界区，其中两个线程访问相同的全局变量。在示例 53-6 中的程序，通过使用一个无名的线程共享信号量来解决同样的问题。

示例 53-6：使用 POSIX 无名信号量保护对全局变量的访问

```
`psem/thread_incr_psem.c`
#include <semaphore.h>
#include <pthread.h>
#include "tlpi_hdr.h"

static int glob = 0;
static sem_t sem;

static void *                   /* Loop 'arg' times incrementing 'glob' */
threadFunc(void *arg)
{
    int loops = *((int *) arg);
    int loc, j;

    for (j = 0; j < loops; j++) {
        if (sem_wait(&sem) == -1)
            errExit("sem_wait");

        loc = glob;
        loc++;
        glob = loc;

        if (sem_post(&sem) == -1)
            errExit("sem_post");
    }

    return NULL;
}

int
main(int argc, char *argv[])
{
    pthread_t t1, t2;
    int loops, s;

    loops = (argc > 1) ? getInt(argv[1], GN_GT_0, "num-loops") : 10000000;

    /* Initialize a thread-shared mutex with the value 1 */

    if (sem_init(&sem, 0, 1) == -1)
        errExit("sem_init");

    /* Create two threads that increment 'glob' */

    s = pthread_create(&t1, NULL, threadFunc, &loops);
    if (s != 0)
        errExitEN(s, "pthread_create");
    s = pthread_create(&t2, NULL, threadFunc, &loops);
    if (s != 0)
        errExitEN(s, "pthread_create");

    /* Wait for threads to terminate */

    s = pthread_join(t1, NULL);
    if (s != 0)
        errExitEN(s, "pthread_join");
    s = pthread_join(t2, NULL);
    if (s != 0)
        errExitEN(s, "pthread_join");

    printf("glob = %d\n", glob);
    exit(EXIT_SUCCESS);
}
    `psem/thread_incr_psem.c`
```

### 销毁无名信号量

*sem_destroy()* 函数销毁信号量 *sem*，该信号量必须是之前使用 *sem_init()* 初始化的无名信号量。只有在没有进程或线程正在等待该信号量时，销毁信号量才是安全的。

```
#include <semaphore.h>

int `sem_destroy`(sem_t **sem*);
```

### 注意

成功时返回 0，出错时返回 -1

在使用 *sem_destroy()* 销毁无名信号量段后，可以使用 *sem_init()* 重新初始化它。

无名信号量应在其底层内存被释放之前销毁。例如，如果信号量是自动分配的变量，它应在宿主函数返回之前销毁。如果信号量位于 POSIX 共享内存区域中，则应在所有进程停止使用信号量后，并在共享内存对象使用 *shm_unlink()* 解除链接之前销毁它。

在某些实现中，省略对 *sem_destroy()* 的调用不会导致问题。然而，在其他实现中，未调用 *sem_destroy()* 可能会导致资源泄漏。为了避免此类问题，便携式应用程序应该调用 *sem_destroy()*。

## 与其他同步技术的比较

本节中，我们将 POSIX 信号量与另外两种同步技术进行比较：System V 信号量和互斥量（mutex）。

#### POSIX 信号量与 System V 信号量

POSIX 信号量和 System V 信号量都可以用来同步进程的操作。System V IPC 与 POSIX IPC 的比较列出了 POSIX IPC 相对于 System V IPC 的各种优势：POSIX IPC 接口更简单，并且与传统 UNIX 文件模型更一致，同时 POSIX IPC 对象是引用计数的，这简化了确定何时删除 IPC 对象的任务。这些一般优势同样适用于 POSIX（命名）信号量与 System V 信号量的具体情况。

POSIX 信号量相对于 System V 信号量有以下进一步优势：

+   POSIX 信号量接口比 System V 信号量接口简单得多。这种简化是在不损失功能性的前提下实现的。

+   POSIX 命名信号量消除了与 System V 信号量相关的初始化问题（信号量初始化）。

+   将 POSIX 无名信号量与动态分配的内存对象关联起来更为容易：信号量可以直接嵌入到对象内部。

+   在信号量争用激烈的场景中（即由于另一个进程将信号量设置为阻止操作立即进行的值，导致对信号量的操作频繁被阻塞），POSIX 信号量和 System V 信号量的性能相似。然而，在信号量争用较低的情况下（即信号量的值使得操作通常可以顺利进行而不会被阻塞），POSIX 信号量的性能明显优于 System V 信号量。（在作者测试的系统上，性能差异超过一个数量级；见练习 53-4。）POSIX 信号量在这种情况下表现更好的原因是它们的实现方式只有在发生争用时才需要系统调用，而 System V 信号量操作则总是需要系统调用，不论是否发生争用。

然而，与 System V 信号量相比，POSIX 信号量也有以下缺点：

+   POSIX 信号量的可移植性稍差。（在 Linux 上，命名信号量从内核 2.6 开始才得到支持。）

+   POSIX 信号量没有提供类似于 System V 信号量的撤销特性。（然而，正如我们在信号量撤销值中提到的，这个特性在某些情况下可能并不有用。）

#### POSIX 信号量与 Pthreads 互斥锁

POSIX 信号量和 Pthreads 互斥锁都可以用来同步同一进程内线程的操作，且它们的性能相似。然而，互斥锁通常更为优选，因为互斥锁的所有权特性有助于代码结构的规范化（只有锁定互斥锁的线程才能解锁它）。相比之下，一个线程可以递增另一个线程递减的信号量。这种灵活性可能导致不良的同步设计。（因此，信号量有时被称为并发编程中的“goto”。）

在多线程应用程序中，有一种情况是无法使用互斥锁（mutexes）的，因此信号量可能更为可取。由于它是异步信号安全的（参见表 21-1，在标准异步信号安全函数中），`*sem_post()*`函数可以在信号处理程序内使用，以便与另一个线程同步。这对于互斥锁来说是不可能的，因为操作互斥锁的 Pthreads 函数不是异步信号安全的。然而，由于通常更推荐通过使用*sigwaitinfo()*（或类似函数）来处理异步信号，而不是使用信号处理程序（参见理智地处理异步信号），因此信号量在这方面相较于互斥锁的优势通常不需要。

## 信号量限制

SUSv3 定义了两个适用于信号量的限制：

`SEM_NSEMS_MAX`

这是一个进程可以拥有的 POSIX 信号量的最大数量。SUSv3 要求此限制至少为 256。对于 Linux，POSIX 信号量的数量实际上仅受可用内存的限制。

`SEM_VALUE_MAX`

这是一个 POSIX 信号量可以达到的最大值。信号量可以取从 0 到此限制之间的任意值。SUSv3 要求此限制至少为 32,767；而 Linux 实现允许的最大值为 `INT_MAX`（在 Linux/x86-32 上为 2,147,483,647）。

## 摘要

POSIX 信号量允许进程或线程同步其操作。POSIX 信号量有两种类型：命名信号量和未命名信号量。命名信号量通过名称进行标识，可以被任何具有打开信号量权限的进程共享。未命名信号量没有名称，但进程或线程可以通过将其放置在它们共享的内存区域中来共享该信号量（例如，在用于进程共享的 POSIX 共享内存对象中，或在用于线程共享的全局变量中）。

POSIX 信号量接口比 System V 信号量接口更简单。信号量是单独分配和操作的，等待（wait）和发布（post）操作将信号量的值调整为 1。

POSIX 信号量相较于 System V 信号量具有一些优势，但它们的可移植性稍差。在多线程应用程序中的同步，通常更推荐使用互斥锁（mutexes）而非信号量。

#### 进一步信息

[Stevens, 1999] 提供了 POSIX 信号量的另一种呈现方式，并展示了使用其他 IPC 机制（如 FIFO、内存映射文件和 System V 信号量）的用户空间实现。[Butenhof, 1996] 描述了在多线程应用程序中使用 POSIX 信号量的方法。

## 练习

1.  将示例 48-2 和示例 48-3(示例：通过共享内存传输数据)中的程序重写为一个多线程应用程序，两个线程通过一个全局缓冲区相互传递数据，并使用 POSIX 信号量进行同步。

1.  修改示例 53-3 来递减 POSIX 信号量")中的程序(`psem_wait.c`)，将*sem_wait()*替换为*sem_timedwait()*。程序应该接受一个额外的命令行参数，指定一个（相对的）秒数作为*sem_timedwait()*调用的超时。

1.  设计一种使用 System V 信号量实现 POSIX 信号量的方法。

1.  在与其他同步技术的比较中，我们指出在信号量未发生竞争的情况下，POSIX 信号量比 System V 信号量表现得更好。编写两个程序（每种信号量类型一个）来验证这一点。每个程序应该简单地增加和减少一个信号量指定次数。比较这两个程序所需的时间。
