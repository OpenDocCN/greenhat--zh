## 第三十章。线程：线程同步

在本章中，我们将介绍两个线程可以用来同步操作的工具：互斥锁和条件变量。互斥锁允许线程同步访问共享资源，举例来说，避免一个线程在另一个线程正在修改共享变量时同时尝试访问该变量。条件变量则执行一个互补任务：它们允许线程相互通知共享变量（或其他共享资源）已发生状态变化。

## 保护共享变量访问：互斥锁

线程的主要优点之一是它们可以通过全局变量共享信息。然而，这种简单的共享是有代价的：我们必须确保多个线程不会同时尝试修改同一个变量，或者一个线程在另一个线程正在修改该变量时不会尝试读取变量的值。术语*临界区*用于指代访问共享资源的代码段，并且该代码段的执行应该是*原子*的；也就是说，其执行不应被其他同时访问同一共享资源的线程中断。

示例 30-1 提供了一个简单的例子，说明当共享资源没有原子访问时可能会发生的问题。该程序创建了两个线程，每个线程执行相同的函数。该函数执行一个循环，不断地递增全局变量*glob*，通过将*glob*的值复制到局部变量*loc*，递增*loc*，再将*loc*的值复制回*glob*。（由于*loc*是一个自动变量，在每个线程的栈上分配，因此每个线程都有自己的该变量副本。）循环的迭代次数由传递给程序的命令行参数决定，或者如果没有提供参数，则使用默认值。

示例 30-1. 两个线程错误地递增全局变量

```
`threads/thread_incr.c`
#include <pthread.h>
#include "tlpi_hdr.h"

static int glob = 0;

static void *                   /* Loop ’arg’ times incrementing ’glob’ */
threadFunc(void *arg)
{
    int loops = *((int *) arg);
    int loc, j;

    for (j = 0; j < loops; j++) {
        loc = glob;
        loc++;
        glob = loc;
    }

    return NULL;
}

int
main(int argc, char *argv[])
{
    pthread_t t1, t2;
    int loops, s;

    loops = (argc > 1) ? getInt(argv[1], GN_GT_0, "num-loops") : 10000000;

    s = pthread_create(&t1, NULL, threadFunc, &loops);
    if (s != 0)
        errExitEN(s, "pthread_create");
    s = pthread_create(&t2, NULL, threadFunc, &loops);
    if (s != 0)
        errExitEN(s, "pthread_create");

    s = pthread_join(t1, NULL);
    if (s != 0)
        errExitEN(s, "pthread_join");
    s = pthread_join(t2, NULL);
    if (s != 0)
        errExitEN(s, "pthread_join");

    printf("glob = %d\n", glob);
    exit(EXIT_SUCCESS);
}
     `threads/thread_incr.c`
```

![两个线程在没有同步的情况下递增全局变量](img/30-1_THREADS-A2-incr-scale90.png.jpg)图 30-1. 两个线程在没有同步的情况下递增全局变量

当我们运行示例 30-1，并指定每个线程应将变量递增 1000 次时，一切看起来都很正常：

```
$ `./thread_incr 1000`
glob = 2000
```

然而，可能发生的情况是，第一个线程在第二个线程开始之前就已经完成了所有工作并终止了。当我们要求两个线程做更多的工作时，结果则大不相同：

```
$ `./thread_incr 10000000`
glob = 16517656
```

在这一系列操作的末尾，*glob*的值应该是 2000 万。这里的问题来自于如下的执行顺序（见图 30-1，上面）：

1.  线程 1 将*glob*的当前值获取到它的局部变量*loc*中。假设当前*glob*的值是 2000。

1.  线程 1 的调度时间片用完，线程 2 开始执行。

1.  线程 2 执行多个循环，在每个循环中，它将*glob*的当前值获取到它的局部变量*loc*中，增加*loc*的值，并将结果赋回*glob*。在第一个循环中，从*glob*获取到的值将是 2000。假设当线程 2 的时间片用完时，*glob*的值已经增加到 3000。

1.  线程 1 接收另一个时间片并继续执行它之前的操作。它之前（步骤 1）将*glob*的值（2000）复制到了*loc*，现在它增加*loc*并将结果（2001）赋回*glob*。此时，线程 2 所做的递增操作的效果就丢失了。

如果我们多次运行示例 30-1，使用相同的命令行参数，我们会发现*glob*的打印值剧烈波动：

```
$ `./thread_incr 10000000`
glob = 10880429
$ `./thread_incr 10000000`
glob = 13493953
```

这种非确定性行为是内核 CPU 调度决策的变动性的结果。在复杂的程序中，这种非确定性行为意味着这些错误可能很少发生，难以重现，因此也很难发现。

似乎我们可以通过用一个单一语句替换示例 30-1 中`for`循环内部的三条语句来消除这个问题：

```
glob++;             /* or: ++glob; */
```

然而，在许多硬件架构中（例如 RISC 架构），编译器仍然需要将这个单一语句转换为机器码，其步骤相当于*threadFunc()*循环内部的三条语句。换句话说，尽管它看起来很简单，即使是 C 语言的自增操作符也可能不是原子的，它可能会表现出我们上面描述的行为。

为了避免线程尝试更新共享变量时可能出现的问题，我们必须使用*互斥锁*（*mutex*，即*mutual exclusion*的缩写）来确保一次只有一个线程能够访问该变量。更一般地说，互斥锁可以用于确保对任何共享资源的原子访问，但保护共享变量是最常见的用途。

互斥量有两种状态：*锁定*和*解锁*。在任何时刻，最多只有一个线程可以持有互斥量的锁。尝试锁定已被锁定的互斥量，要么会阻塞，要么会因所用方法的不同而返回错误。

当线程锁定互斥量时，它成为该互斥量的拥有者。只有互斥量的拥有者可以解锁该互斥量。这个属性改进了使用互斥量的代码结构，并且还允许对互斥量的实现进行一些优化。由于这个拥有权属性，*获取*和*释放*这两个术语有时与锁定和解锁互斥量同义使用。

通常，我们为每个共享资源（可能由多个相关变量组成）使用不同的互斥量，且每个线程都遵循以下协议来访问资源：

+   锁定共享资源的互斥量；

+   访问共享资源；以及

+   解锁互斥量；

如果多个线程尝试执行这段代码块（*临界区*），由于只有一个线程可以持有互斥量（其他线程被阻塞），意味着一次只能有一个线程进入该代码块，如图 30-2 所示。

![使用互斥量保护临界区](img/30-2_THREADS-A2-mutex-block-scale90.png.jpg)图 30-2. 使用互斥量保护临界区

最后，注意互斥量的锁定是建议性的，而非强制性的。我们的意思是，线程可以自由忽略互斥量的使用，直接访问相应的共享变量。为了安全地处理共享变量，所有线程必须在使用互斥量时协同工作，遵守其强制的锁定规则。

### 静态分配的互斥量

互斥量可以作为静态变量分配，或在运行时动态创建（例如，在通过*malloc()*分配的内存块中）。动态互斥量的创建稍微复杂一些，我们将推迟讨论，直到动态初始化互斥量。

互斥量是类型为*pthread_mutex_t*的变量。在使用互斥量之前，必须始终初始化它。对于静态分配的互斥量，我们可以通过将其赋值为`PTHREAD_MUTEX_INITIALIZER`来实现初始化，如以下示例所示：

```
pthread_mutex_t mtx = PTHREAD_MUTEX_INITIALIZER;
```

### 注意

根据 SUSv3 的规定，将我们在本节其余部分描述的操作应用于互斥量的*副本*会导致未定义的结果。互斥量操作应始终仅在已静态初始化的原始互斥量上执行，该原始互斥量可以通过`PTHREAD_MUTEX_INITIALIZER`进行静态初始化，或通过*pthread_mutex_init()*进行动态初始化（详见动态初始化互斥量）。

### 锁定与解锁互斥量

初始化后，互斥锁是解锁的。要锁定和解锁互斥锁，我们使用*pthread_mutex_lock()*和*pthread_mutex_unlock()*函数。

```
#include <pthread.h>

int `pthread_mutex_lock`(pthread_mutex_t **mutex*);
int `pthread_mutex_unlock`(pthread_mutex_t **mutex*);
```

### 注意

两者在成功时返回 0，出错时返回正的错误号

要锁定一个互斥锁，我们通过调用*pthread_mutex_lock()*来指定该互斥锁。如果互斥锁当前未被锁定，该调用会立即锁定互斥锁并返回。如果互斥锁当前已被另一个线程锁定，那么*pthread_mutex_lock()*会阻塞直到互斥锁被解锁，届时它会锁定该互斥锁并返回。

如果调用线程本身已经锁定了传递给*pthread_mutex_lock()*的互斥锁，那么对于默认类型的互斥锁，可能会出现两种实现定义的结果之一：线程死锁，尝试锁定它已经拥有的互斥锁时被阻塞，或者调用失败，返回错误`EDEADLK`。在 Linux 中，默认情况下线程会死锁。（当我们查看互斥锁类型时，我们会描述其他一些可能的行为。）

*pthread_mutex_unlock()*函数解锁由调用线程之前锁定的互斥锁。解锁一个未被锁定的互斥锁，或者解锁一个被另一个线程锁定的互斥锁是错误的。

如果多个线程在等待获取由调用*pthread_mutex_unlock()*解锁的互斥锁，那么哪个线程会成功获取该锁是无法确定的。

#### 示例程序

示例 30-2 是示例 30-1 程序的修改版。它使用互斥锁来保护对全局变量*glob*的访问。当我们使用类似之前的命令行运行此程序时，我们会看到*glob*始终被可靠地递增：

```
$ `./thread_incr_mutex 10000000`
glob = 20000000
```

示例 30-2. 使用互斥锁保护对全局变量的访问

```
`threads/thread_incr_mutex.c`
#include <pthread.h>
#include "tlpi_hdr.h"

static int glob = 0;
static pthread_mutex_t mtx = PTHREAD_MUTEX_INITIALIZER;

static void *                   /* Loop ’arg’ times incrementing ’glob’ */
threadFunc(void *arg)
{
    int loops = *((int *) arg);
    int loc, j, s;

    for (j = 0; j < loops; j++) {
        s = pthread_mutex_lock(&mtx);
        if (s != 0)
            errExitEN(s, "pthread_mutex_lock");

        loc = glob;
        loc++;
        glob = loc;

        s = pthread_mutex_unlock(&mtx);
        if (s != 0)
            errExitEN(s, "pthread_mutex_unlock");
    }

    return NULL;
}

int
main(int argc, char *argv[])
{
    pthread_t t1, t2;
    int loops, s;

    loops = (argc > 1) ? getInt(argv[1], GN_GT_0, "num-loops") : 10000000;

    s = pthread_create(&t1, NULL, threadFunc, &loops);
    if (s != 0)
        errExitEN(s, "pthread_create");
    s = pthread_create(&t2, NULL, threadFunc, &loops);
    if (s != 0)
        errExitEN(s, "pthread_create");

    s = pthread_join(t1, NULL);
    if (s != 0)
        errExitEN(s, "pthread_join");
    s = pthread_join(t2, NULL);
    if (s != 0)
        errExitEN(s, "pthread_join");

    printf("glob = %d\n", glob);
    exit(EXIT_SUCCESS);
}
     `threads/thread_incr_mutex.c`
```

#### *pthread_mutex_trylock()* 和 *pthread_mutex_timedlock()*

Pthreads API 提供了*pthread_mutex_lock()*函数的两个变体：*pthread_mutex_trylock()* 和 *pthread_mutex_timedlock()*。（有关这些函数的原型，请参见手册页。）

*pthread_mutex_trylock()*函数与*pthread_mutex_lock()*相同，不同之处在于如果互斥锁当前被锁定，*pthread_mutex_trylock()*会失败，返回错误`EBUSY`。

*pthread_mutex_timedlock()*函数与*pthread_mutex_lock()*相同，唯一的不同是调用者可以指定一个额外的参数*abstime*，该参数限定了线程在等待获取互斥锁时的最大等待时间。如果指定的*abstime*参数指定的时间间隔到期，且调用者未成为互斥锁的拥有者，那么*pthread_mutex_timedlock()*会返回错误`ETIMEDOUT`。

*pthread_mutex_trylock()* 和 *pthread_mutex_timedlock()* 函数的使用频率远低于 *pthread_mutex_lock()*。在大多数设计良好的应用中，一个线程应该只持有互斥锁短暂的时间，以便其他线程能够并行执行。这确保了被阻塞在互斥锁上的其他线程将很快获得互斥锁的锁定。使用 *pthread_mutex_trylock()* 定期轮询互斥锁以查看它是否可以被锁定的线程，存在因其他排队线程通过 *pthread_mutex_lock()* 依次获得互斥锁而无法访问互斥锁的风险。

### 互斥锁的性能

使用互斥锁的代价是多少？我们展示了两个不同版本的程序来递增一个共享变量：一个不使用互斥锁（示例 30-1），一个使用互斥锁（示例 30-2）。当我们在运行 Linux 2.6.31（带有 NPTL）的 x86-32 系统上运行这两个程序时，我们发现没有互斥锁的版本需要 0.35 秒来执行每个线程中的 1000 万次循环（并且产生错误的结果），而使用互斥锁的版本需要 3.1 秒。

起初，这似乎代价很高。但请考虑不使用互斥锁的版本执行的主循环（示例 30-1）。在该版本中，*threadFunc()* 函数执行一个 `for` 循环，递增一个循环控制变量，将该变量与另一个变量进行比较，执行两个赋值操作和另一个递增操作，然后跳回循环顶部。使用互斥锁的版本（示例 30-2）执行相同的步骤，并在每次循环时锁定和解锁互斥锁。换句话说，锁定和解锁互斥锁的成本大约是我们为第一个程序列出的操作成本的十倍不到。这相对较便宜。此外，在典型情况下，线程会花费更多时间做其他工作，并执行较少的互斥锁定和解锁操作，因此在大多数应用中，使用互斥锁的性能影响并不显著。

为了进一步说明这一点，在同一系统上运行一些简单的测试程序显示，使用 *fcntl()* 锁定和解锁文件区域的 2000 万次循环需要 44 秒，而使用 System V 信号量进行递增和递减的 2000 万次循环需要 28 秒 (第四十七章)。文件锁和信号量的问题在于，它们总是需要系统调用来执行锁定和解锁操作，每个系统调用都有一个小但显著的开销 (系统调用)。相比之下，互斥锁是通过原子机器语言操作（在所有线程可见的内存位置上执行）来实现的，只有在锁竞争的情况下才需要系统调用。

### 注意

在 Linux 中，互斥锁是通过 *futexes* 实现的（*futexes* 是 *fast user space mutexes* 的缩写），锁竞争通过 *futex()* 系统调用来处理。我们在本书中没有描述 futexes（它们不打算直接用于用户空间应用程序），但可以在 [Drepper, 2004 (a)] 中找到详细信息，该文献也描述了如何使用 futexes 实现互斥锁。[Franke 等人, 2002] 是一篇（现在已过时的）论文，由 futexes 的开发人员编写，描述了早期的 futex 实现，并研究了 futexes 带来的性能提升。

### 互斥锁死锁

有时，一个线程需要同时访问两个或多个不同的共享资源，每个资源都由一个独立的互斥锁来管理。当多个线程同时锁定相同的互斥锁集合时，可能会出现死锁情况。图 30-3 展示了一个死锁的例子，其中每个线程成功锁定一个互斥锁，然后尝试锁定另一个线程已经锁定的互斥锁。这两个线程将一直被阻塞，无法解锁。

![两个线程锁定两个互斥锁时的死锁](img/30-3_THREADS-A2-mutex-deadlock-scale90.png.jpg)图 30-3. 两个线程锁定两个互斥锁时的死锁

避免这种死锁的最简单方法是定义一个互斥锁层次结构。当线程可以锁定同一组互斥锁时，它们应该始终按相同的顺序来锁定它们。例如，在图 30-3 的场景中，如果两个线程始终按照 *mutex1* 然后是 *mutex2* 的顺序来锁定互斥锁，就能避免死锁的发生。有时，互斥锁之间有一个逻辑上显而易见的层次结构。然而，即使没有这样的结构，仍然可以设计出一个任意的层次顺序，供所有线程遵循。

一种较少使用的替代策略是“尝试，然后退回”。在这种策略中，线程使用 *pthread_mutex_lock()* 锁住第一个互斥锁，然后使用 *pthread_mutex_trylock()* 锁住其余的互斥锁。如果任何 *pthread_mutex_trylock()* 调用失败（返回 `EBUSY`），则线程释放所有互斥锁，然后再尝试，可能会在延迟一段时间后再次尝试。这种方法比锁层次结构效率低，因为可能需要多次迭代。另一方面，它更加灵活，因为它不需要严格的互斥锁层次结构。这个策略的示例在 [Butenhof, 1996] 中有所展示。

### 动态初始化互斥锁

静态初始化值 `PTHREAD_MUTEX_INITIALIZER` 仅能用于初始化具有默认属性的静态分配互斥锁。在所有其他情况下，我们必须使用 *pthread_mutex_init()* 动态初始化互斥锁。

```
#include <pthread.h>

int `pthread_mutex_init`(pthread_mutex_t **mutex*, const pthread_mutexattr_t **attr*);
```

### 注意

成功时返回 0，出错时返回一个正的错误号码

*mutex* 参数标识要初始化的互斥锁。*attr* 参数是指向一个已被初始化的 *pthread_mutexattr_t* 对象的指针，用来定义互斥锁的属性。（在下一节中，我们会详细讲解互斥锁属性。）如果 *attr* 被指定为 `NULL`，则互斥锁会被赋予各种默认属性。

SUSv3 规定，初始化一个已经初始化过的互斥锁会导致未定义的行为；我们不应这么做。

我们必须使用 *pthread_mutex_init()* 而不是静态初始化器的情况包括以下几种：

+   互斥锁是在堆上动态分配的。例如，假设我们创建了一个动态分配的结构体链表，每个结构体中都有一个包含互斥锁的 *pthread_mutex_t* 字段，用来保护对该结构体的访问。

+   互斥锁是一个分配在栈上的自动变量。

+   我们希望初始化一个具有默认属性之外的属性的静态分配互斥锁。

当一个自动或动态分配的互斥锁不再需要时，应该使用 *pthread_mutex_destroy()* 销毁它。（对于使用 `PTHREAD_MUTEX_INITIALIZER` 静态初始化的互斥锁，不需要调用 *pthread_mutex_destroy()*。）

```
#include <pthread.h>

int `pthread_mutex_destroy`(pthread_mutex_t **mutex*);
```

### 注意

成功时返回 0，出错时返回一个正的错误号码

只有在互斥锁处于解锁状态并且没有线程会再尝试锁住它时，销毁互斥锁才是安全的。如果互斥锁位于动态分配的内存区域中，则应在释放该内存区域之前销毁它。自动分配的互斥锁应在其宿主函数返回之前销毁。

一个已被 *pthread_mutex_destroy()* 销毁的互斥锁可以通过 *pthread_mutex_init()* 重新初始化。

### 互斥锁属性

如前所述，*pthread_mutex_init() attr* 参数可用于指定一个 *pthread_mutexattr_t* 对象，该对象定义了互斥锁的属性。可以使用各种 Pthreads 函数来初始化和获取 *pthread_mutexattr_t* 对象中的属性。我们不会深入讨论互斥锁属性的所有细节，也不会展示可以用来初始化 *pthread_mutexattr_t* 对象属性的各种函数的原型。然而，我们将描述一个可以为互斥锁设置的属性：其类型。

### 互斥锁类型

在前面的页面中，我们做了一些关于互斥锁行为的陈述：

+   单个线程不能两次锁定相同的互斥锁。

+   线程不能解锁它当前不拥有的互斥锁（即它没有锁定的互斥锁）。

+   线程不能解锁当前没有锁定的互斥锁。

在这些情况下发生的具体情况取决于互斥锁的*类型*。SUSv3 定义了以下互斥锁类型：

`PTHREAD_MUTEX_NORMAL`

这种类型的互斥锁不提供（自我）死锁检测。如果一个线程尝试锁定它已经锁定的互斥锁，就会发生死锁。解锁一个没有被锁定或被另一个线程锁定的互斥锁会产生未定义的结果。（在 Linux 上，这两种操作对这种类型的互斥锁会成功。）

`PTHREAD_MUTEX_ERRORCHECK`

所有操作都进行了错误检查。上述三个场景都会导致相关的 Pthreads 函数返回错误。这种类型的互斥锁通常比普通的互斥锁要慢，但可以作为调试工具，用于发现应用程序在哪里违反了有关互斥锁使用规则的规定。

`PTHREAD_MUTEX_RECURSIVE`

递归互斥锁维护一个锁定计数的概念。当一个线程首次获取互斥锁时，锁定计数被设置为 1。该线程每次执行锁操作时，锁定计数都会增加，每次解锁操作时，计数会减少。只有当锁定计数降为 0 时，互斥锁才会被释放（即允许其他线程获取）。解锁一个未锁定的互斥锁会失败，解锁一个当前被另一个线程锁定的互斥锁也会失败。

Linux 线程实现提供了每种互斥锁类型的非标准静态初始化器（例如，`PTHREAD_RECURSIVE_MUTEX_INITIALIZER_NP`），因此对于静态分配的互斥锁，不需要使用 *pthread_mutex_init()* 来初始化这些互斥锁类型。然而，可移植的应用程序应避免使用这些初始化器。

除了上面的互斥锁类型，SUSv3 还定义了 `PTHREAD_MUTEX_DEFAULT` 类型，这是如果我们使用 `PTHREAD_MUTEX_INITIALIZER` 或在调用 *pthread_mutex_init()* 时指定*attr*为 `NULL`，默认的互斥锁类型。此互斥锁类型的行为在本节开始时描述的所有三种情况中都是故意未定义的，这为互斥锁的高效实现提供了最大灵活性。在 Linux 上，`PTHREAD_MUTEX_DEFAULT` 互斥锁的行为类似于 `PTHREAD_MUTEX_NORMAL` 互斥锁。

示例 30-3 中展示的代码演示了如何设置互斥锁的类型，在本例中是创建一个*错误检查*互斥锁。

示例 30-3. 设置互斥锁类型

```
pthread_mutex_t mtx;
    pthread_mutexattr_t mtxAttr;
    int s, type;

    s = pthread_mutexattr_init(&mtxAttr);
    if (s != 0)
        errExitEN(s, "pthread_mutexattr_init");

    s = pthread_mutexattr_settype(&mtxAttr, PTHREAD_MUTEX_ERRORCHECK);
    if (s != 0)
        errExitEN(s, "pthread_mutexattr_settype");

    s = pthread_mutex_init(&mtx, &mtxAttr);
    if (s != 0)
        errExitEN(s, "pthread_mutex_init");

    s = pthread_mutexattr_destroy(&mtxAttr);        /* No longer needed */
    if (s != 0)
        errExitEN(s, "pthread_mutexattr_destroy");
```

## 状态变化的信号：条件变量

互斥锁防止多个线程同时访问共享变量。条件变量允许一个线程通知其他线程共享变量（或其他共享资源）状态的变化，并允许其他线程等待（阻塞）此类通知。

一个不使用条件变量的简单示例能够说明它们为何有用。假设我们有多个线程生产一些“结果单元”，这些单元由主线程消费，我们使用一个受互斥锁保护的变量*avail*来表示等待消费的生产单元数量：

```
static pthread_mutex_t mtx = PTHREAD_MUTEX_INITIALIZER;

static int avail = 0;
```

### 注意

本节中展示的代码段可以在本书源代码的 `threads/prod_no_condvar.c` 文件中找到。

在生产者线程中，我们会有如下代码：

```
/* Code to produce a unit omitted */

s = pthread_mutex_lock(&mtx);
if (s != 0)
    errExitEN(s, "pthread_mutex_lock");

avail++;    /* Let consumer know another unit is available */

s = pthread_mutex_unlock(&mtx);
if (s != 0)
    errExitEN(s, "pthread_mutex_unlock");
```

在主（消费者）线程中，我们可以使用以下代码：

```
for (;;) {
    s = pthread_mutex_lock(&mtx);
    if (s != 0)
        errExitEN(s, "pthread_mutex_lock");

    while (avail > 0) {         /* Consume all available units */
        /* Do something with produced unit */
        avail--;
    }

    s = pthread_mutex_unlock(&mtx);
    if (s != 0)
        errExitEN(s, "pthread_mutex_unlock");
}
```

上面的代码是有效的，但它浪费了 CPU 时间，因为主线程会不断循环，检查变量*avail*的状态。*条件变量*可以解决这个问题。它允许一个线程进入休眠（等待）状态，直到另一个线程通知（信号）它必须执行某些操作（即，某个“条件”已经发生，休眠线程现在必须响应）。

条件变量总是与互斥锁一起使用。互斥锁为访问共享变量提供互斥，而条件变量用于通知变量状态的变化。（此处使用的*信号*一词与第二十章到第二十二章中描述的信号无关；它是指*指示*的意思。）

### 静态分配条件变量

与互斥锁一样，条件变量可以静态或动态分配。我们将延后讨论动态分配条件变量，直到 Dynamically Allocated Condition Variables 章节，并在这里讨论静态分配的条件变量。

条件变量的类型是 *pthread_cond_t*。与互斥锁类似，条件变量在使用前必须初始化。对于静态分配的条件变量，可以通过将其赋值为 `PTHREAD_COND_INITIALIZER` 来初始化，如以下示例所示：

```
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
```

### 注意

根据 SUSv3，将我们在本节余下部分中描述的操作应用于条件变量的 *副本* 会导致未定义的结果。操作应始终仅在使用 `PTHREAD_COND_INITIALIZER` 静态初始化或使用 *pthread_cond_init()* 动态初始化的原始条件变量上执行（详见 动态分配的条件变量）。

### 条件变量的信号和等待

主要的条件变量操作是 *signal* 和 *wait*。信号操作是通知一个或多个等待线程共享变量的状态已经改变。等待操作是阻塞直到接收到这种通知的手段。

*pthread_cond_signal()* 和 *pthread_cond_broadcast()* 函数都向指定的条件变量 *cond* 发送信号。*pthread_cond_wait()* 函数阻塞一个线程，直到条件变量 *cond* 被信号唤醒。

```
#include <pthread.h>

int `pthread_cond_signal`(pthread_cond_t **cond*);
int `pthread_cond_broadcast`(pthread_cond_t **cond*);
int `pthread_cond_wait`(pthread_cond_t **cond*, pthread_mutex_t **mutex*);
```

### 注意

成功时返回 0，出错时返回正的错误号。

*pthread_cond_signal()* 和 *pthread_cond_broadcast()* 之间的区别在于，当多个线程在 *pthread_cond_wait()* 中被阻塞时发生的情况。使用 *pthread_cond_signal()* 时，我们仅确保至少一个被阻塞的线程会被唤醒；而使用 *pthread_cond_broadcast()* 时，所有被阻塞的线程都会被唤醒。

使用 *pthread_cond_broadcast()* 总是能产生正确的结果（因为所有线程应该编写代码来处理冗余和虚假唤醒），但是 *pthread_cond_signal()* 可能更高效。然而，*pthread_cond_signal()* 应仅在只需要唤醒一个等待线程来处理共享变量状态变化的情况使用，而且不在乎唤醒哪个等待线程。当所有等待线程都设计为执行完全相同的任务时，这种场景通常适用。在这些假设下，*pthread_cond_signal()* 可以比 *pthread_cond_broadcast()* 更高效，因为它避免了以下可能性：

1.  所有等待线程都被唤醒。

1.  一个线程首先被调度。该线程检查共享变量的状态（在关联互斥锁的保护下），并看到有工作需要完成。线程执行所需的工作，改变共享变量的状态以表明工作已完成，并解锁关联的互斥锁。

1.  剩下的每个线程依次锁定互斥量并检查共享变量的状态。然而，由于第一个线程所做的更改，这些线程发现没有工作可做，因此解锁互斥量并回到休眠状态（即再次调用 *pthread_cond_wait()*）。

相比之下，*pthread_cond_broadcast()* 处理的是等待线程需要执行不同任务的情况（在这种情况下，它们可能会有不同的条件变量谓词）。

条件变量不保存任何状态信息。它只是一个用于传递应用程序状态信息的机制。如果在条件变量被信号唤醒时没有线程在等待它，那么信号就会丢失。稍后等待条件变量的线程，只有当变量再次被信号唤醒时才会解除阻塞。

*pthread_cond_timedwait()* 函数与 *pthread_cond_wait()* 相同，不同之处在于 *abstime* 参数指定了线程在等待条件变量被信号唤醒时的最大休眠时间。

```
#include <pthread.h>

int `pthread_cond_timedwait`(pthread_cond_t **cond*, pthread_mutex_t **mutex*,
                           const struct timespec **abstime*);
```

### 注

成功时返回 0，出错时返回一个正的错误号。

*abstime* 参数是一个 *timespec* 结构体（高分辨率睡眠：*nanosleep()*")），指定了自纪元（日历时间）以来的绝对时间，以秒和纳秒表示。如果 *abstime* 指定的时间间隔过期且条件变量没有被信号唤醒，则 *pthread_cond_timedwait()* 返回错误 `ETIMEDOUT`。

#### 在生产者-消费者示例中使用条件变量

让我们修改之前的示例以使用条件变量。我们全局变量以及关联的互斥量和条件变量的声明如下：

```
static pthread_mutex_t mtx = PTHREAD_MUTEX_INITIALIZER;
static pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

static int avail = 0;
```

### 注

本节中显示的代码段可以在本书的源代码分发文件 `threads/prod_condvar.c` 中找到。

生产者线程中的代码与之前相同，唯一不同的是我们添加了一个调用 *pthread_cond_signal()*：

```
s = pthread_mutex_lock(&mtx);
if (s != 0)
    errExitEN(s, "pthread_mutex_lock");

avail++;                /* Let consumer know another unit is available */

s = pthread_mutex_unlock(&mtx);
if (s != 0)
    errExitEN(s, "pthread_mutex_unlock");

s = pthread_cond_signal(&cond);         /* Wake sleeping consumer */
if (s != 0)
    errExitEN(s, "pthread_cond_signal");
```

在考虑消费者代码之前，我们需要更详细地解释 *pthread_cond_wait()*。我们之前提到过，条件变量总是与一个互斥量关联。这两个对象作为参数传递给 *pthread_cond_wait()*，它执行以下步骤：

+   解锁由 *mutex* 指定的互斥量；

+   阻塞调用线程，直到另一个线程发出条件变量 *cond* 的信号；并且

+   重新锁定 *mutex*。

*pthread_cond_wait()* 函数的设计目的是执行这些步骤，因为通常我们以以下方式访问共享变量：

```
s = pthread_mutex_lock(&mtx);
if (s != 0)
    errExitEN(s, "pthread_mutex_lock");

while (/* Check that shared variable is not in state we want */)
    pthread_cond_wait(&cond, &mtx);

/* Now shared variable is in desired state; do some work */

s = pthread_mutex_unlock(&mtx);
if (s != 0)
    errExitEN(s, "pthread_mutex_unlock");
```

（我们将在下一节解释为何将 *pthread_cond_wait()* 放在 `while` 循环中，而不是 `if` 语句中。）

在上述代码中，两个访问共享变量的操作都必须进行互斥锁保护，原因我们之前已经解释过。换句话说，互斥锁和条件变量之间有一个自然的关联：

1.  线程锁定互斥锁，为检查共享变量的状态做准备。

1.  共享变量的状态被检查。

1.  如果共享变量不处于期望的状态，那么线程必须解锁互斥锁（以便其他线程可以访问共享变量），然后再在条件变量上入睡。

1.  当线程因为条件变量被发出信号而重新唤醒时，必须再次锁定互斥锁，因为通常情况下，线程此时会立即访问共享变量。

*pthread_cond_wait()* 函数自动执行这两个步骤中的互斥锁解锁和锁定操作。在第三步中，释放互斥锁和在条件变量上阻塞是原子性执行的。换句话说，在调用 *pthread_cond_wait()* 的线程阻塞在条件变量之前，不可能有其他线程获取互斥锁并发出条件变量的信号。

### 注意

关于条件变量和互斥锁之间有自然关系的观察有一个推论：所有同时等待同一条件变量的线程在它们的 *pthread_cond_wait()*（或 *pthread_cond_timedwait()*）调用中必须指定相同的互斥锁。实际上，*pthread_cond_wait()* 调用在调用期间将条件变量动态绑定到一个唯一的互斥锁。SUSv3 提到，使用多个互斥锁进行并发的 *pthread_cond_wait()* 调用同一个条件变量的结果是未定义的。

将上述细节结合起来，我们可以修改主（消费者）线程来使用 *pthread_cond_wait()*，如下所示：

```
for (;;) {
    s = pthread_mutex_lock(&mtx);
    if (s != 0)
        errExitEN(s, "pthread_mutex_lock");

    while (avail == 0) {            /* Wait for something to consume */
        s = pthread_cond_wait(&cond, &mtx);
        if (s != 0)
            errExitEN(s, "pthread_cond_wait");
    }

    while (avail > 0) {             /* Consume all available units */
        /* Do something with produced unit */
        avail--;
    }

    s = pthread_mutex_unlock(&mtx);
    if (s != 0)
        errExitEN(s, "pthread_mutex_unlock");

    /* Perhaps do other work here that doesn’t require mutex lock */
}
```

我们最后对使用 *pthread_cond_signal()*（和 *pthread_cond_broadcast()*）做一个总结观察。在之前的生产者代码中，我们首先调用了 *pthread_mutex_unlock()*，然后调用了 *pthread_cond_signal()*；也就是说，我们先解锁了与共享变量相关的互斥锁，然后发出了相应的条件变量信号。我们也可以将这两个步骤的顺序反过来；SUSv3 允许它们以任意顺序执行。

### 注意

[Butenhof, 1996]指出，在某些实现中，先解锁互斥锁再发出条件变量信号的性能可能优于逆序执行这些步骤。如果互斥锁只有在条件变量被信号后才解锁，那么执行*pthread_cond_wait()*的线程可能会在互斥锁仍然被锁住时被唤醒，然后在发现互斥锁被锁住时立即重新进入睡眠状态。这会导致两个多余的上下文切换。一些实现通过采用称为*等待变形*的技术来消除这个问题，该技术将信号线程从条件变量等待队列移动到互斥锁等待队列，而无需执行上下文切换，前提是互斥锁已被锁住。

### 测试条件变量的谓词

每个条件变量都有一个与之关联的谓词，该谓词涉及一个或多个共享变量。例如，在前面章节中的代码段中，与*cond*相关联的谓词是*(avail == 0)*。该代码段展示了一个通用设计原则：*pthread_cond_wait()*调用必须由`while`循环来控制，而不是`if`语句。这是因为，从*pthread_cond_wait()*返回后，谓词的状态无法保证；因此，我们应该立即重新检查谓词，并在其不处于期望状态时继续睡眠。

我们不能对从*pthread_cond_wait()*返回后的谓词状态做出任何假设，原因如下：

+   *其他线程可能会先被唤醒*。可能有多个线程在等待获取与条件变量相关联的互斥锁。即使发出信号的线程已将谓词设置为期望状态，仍然可能是另一个线程先获取了互斥锁并改变了相关共享变量的状态，从而改变了谓词的状态。

+   *设计“松散”谓词可能更简单*。有时，基于指示*可能性*而非*确定性*的条件变量设计应用程序会更容易。换句话说，信号条件变量意味着“可能有某些事情”需要信号线程去做，而不是“有某些事情”需要做。使用这种方法，条件变量可以基于谓词状态的近似值来发出信号，信号线程可以通过重新检查谓词来确认是否真的有事情要做。

+   *虚假唤醒可能会发生*。在某些实现中，一个线程在等待条件变量时，可能会被唤醒，尽管没有其他线程实际发出条件变量信号。这种虚假唤醒是某些多处理器系统中为了高效实现而需要的技术的（罕见）结果，并且在 SUSv3 中明确允许。

### 示例程序：加入任何已终止的线程

我们之前提到过，*pthread_join()*只能与特定线程连接。它没有提供与*任何*已终止线程连接的机制。现在，我们将展示如何使用条件变量来绕过这个限制。

程序在示例 30-4 中为每个命令行参数创建一个线程。每个线程休眠时间为对应命令行参数指定的秒数，然后终止。休眠时间是我们模拟线程执行任务的手段。

程序维护一组全局变量，记录所有已创建线程的信息。对于每个线程，全球*thread*数组中的一个元素记录线程的 ID（*tid*字段）和其当前状态（*state*字段）。*state*字段有以下几种值：`TS_ALIVE`，表示线程处于活动状态；`TS_TERMINATED`，表示线程已终止但尚未连接；或`TS_JOINED`，表示线程已终止并且已连接。

每当一个线程终止时，它会将值`TS_TERMINATED`分配给*thread*数组中该线程元素的*state*字段，增加一个全局计数器，记录已终止但尚未连接的线程(*numUnjoined*)，并触发条件变量*threadDied*。

主线程使用一个循环，持续等待条件变量*threadDied*。每当*threadDied*被触发且存在尚未连接的已终止线程时，主线程扫描*thread*数组，寻找*state*字段设置为`TS_TERMINATED`的元素。对于处于该状态的每个线程，调用*pthread_join()*，使用*thread*数组中相应的*tid*字段，然后将*state*设置为`TS_JOINED`。当所有由主线程创建的线程都已终止——即全局变量*numLive*为 0 时，主循环终止。

以下 shell 会话日志演示了示例 30-4 中程序的使用：

```
$ `./thread_multijoin 1 1 2 3 3`              *Create 5 threads*
Thread 0 terminating
Thread 1 terminating
Reaped thread 0 (numLive=4)
Reaped thread 1 (numLive=3)
Thread 2 terminating
Reaped thread 2 (numLive=2)
Thread 3 terminating
Thread 4 terminating
Reaped thread 3 (numLive=1)
Reaped thread 4 (numLive=0)
```

最后，注意尽管示例程序中的线程是以可连接的方式创建，并且在终止时通过*pthread_join()*立即回收，但我们不需要采用这种方法来了解线程的终止状态。我们本可以让线程为分离线程，移除使用*pthread_join()*，并仅使用*thread*数组（和相关的全局变量）来记录每个线程的终止情况。

示例 30-4. 一个可以与任何已终止线程连接的主线程

```
`threads/thread_multijoin.c`
#include <pthread.h>
#include "tlpi_hdr.h"

static pthread_cond_t threadDied = PTHREAD_COND_INITIALIZER;
static pthread_mutex_t threadMutex = PTHREAD_MUTEX_INITIALIZER;
                /* Protects all of the following global variables */

static int totThreads = 0;      /* Total number of threads created */
static int numLive = 0;         /* Total number of threads still alive or
                                   terminated but not yet joined */
static int numUnjoined = 0;     /* Number of terminated threads that
                                   have not yet been joined */
enum tstate {                   /* Thread states */
    TS_ALIVE,                   /* Thread is alive */
    TS_TERMINATED,              /* Thread terminated, not yet joined */
    TS_JOINED                   /* Thread terminated, and joined */
};

static struct {                 /* Info about each thread */
    pthread_t tid;              /* ID of this thread */
    enum tstate state;          /* Thread state (TS_* constants above) */
    int sleepTime;              /* Number seconds to live before terminating */
} *thread;

static void *                   /* Start function for thread */
threadFunc(void *arg)
{
    int idx = *((int *) arg);
    int s;

    sleep(thread[idx].sleepTime);       /* Simulate doing some work */
    printf("Thread %d terminating\n", idx);

    s = pthread_mutex_lock(&threadMutex);
    if (s != 0)
        errExitEN(s, "pthread_mutex_lock");

    numUnjoined++;
    thread[idx].state = TS_TERMINATED;

    s = pthread_mutex_unlock(&threadMutex);
    if (s != 0)
        errExitEN(s, "pthread_mutex_unlock");
    s = pthread_cond_signal(&threadDied);
    if (s != 0)
        errExitEN(s, "pthread_cond_signal");

    return NULL;
}

int
main(int argc, char *argv[])
{
    int s, idx;

    if (argc < 2 || strcmp(argv[1], "--help") == 0)
        usageErr("%s nsecs...\n", argv[0]);

    thread = calloc(argc - 1, sizeof(*thread));
    if (thread == NULL)
        errExit("calloc");

    /* Create all threads */

    for (idx = 0; idx < argc - 1; idx++) {
        thread[idx].sleepTime = getInt(argv[idx + 1], GN_NONNEG, NULL);
        thread[idx].state = TS_ALIVE;
        s = pthread_create(&thread[idx].tid, NULL, threadFunc, &idx);
        if (s != 0)
            errExitEN(s, "pthread_create");
    }

    totThreads = argc - 1;
    numLive = totThreads;

    /* Join with terminated threads */

    while (numLive > 0) {
        s = pthread_mutex_lock(&threadMutex);
        if (s != 0)
            errExitEN(s, "pthread_mutex_lock");

        while (numUnjoined == 0) {
            s = pthread_cond_wait(&threadDied, &threadMutex);
            if (s != 0)
                errExitEN(s, "pthread_cond_wait");
        }

        for (idx = 0; idx < totThreads; idx++) {
            if (thread[idx].state == TS_TERMINATED){
                s = pthread_join(thread[idx].tid, NULL);
                if (s != 0)
                    errExitEN(s, "pthread_join");

                thread[idx].state = TS_JOINED;
                numLive--;
                numUnjoined--;

                printf("Reaped thread %d (numLive=%d)\n", idx, numLive);
            }
        }

        s = pthread_mutex_unlock(&threadMutex);
        if (s != 0)
            errExitEN(s, "pthread_mutex_unlock");
    }

    exit(EXIT_SUCCESS);
}
      `threads/thread_multijoin.c`
```

### 动态分配的条件变量

*pthread_cond_init()*函数用于动态初始化条件变量。我们需要使用*pthread_cond_init()*的情境类似于使用*pthread_mutex_init()*动态初始化互斥锁的情境（参见动态初始化互斥锁）；也就是说，我们必须使用*pthread_cond_init()*来初始化自动和动态分配的条件变量，并初始化具有非默认属性的静态分配条件变量。

```
#include <pthread.h>

int `pthread_cond_init`(pthread_cond_t **cond*, const pthread_condattr_t **attr*);
```

### 注意

成功时返回 0，出错时返回正的错误号

*cond*参数标识要初始化的条件变量。与互斥锁一样，我们可以指定一个*attr*参数，该参数是之前初始化的，用于确定条件变量的属性。可以使用各种 Pthreads 函数来初始化*attr*指向的*pthread_condattr_t*对象中的属性。如果*attr*为`NULL`，则为条件变量分配一组默认属性。

SUSv3 规定，初始化已经初始化过的条件变量会导致未定义行为；我们不应这样做。

当自动或动态分配的条件变量不再需要时，应使用*pthread_cond_destroy()*销毁它。对于通过`PTHREAD_COND_INITIALIZER`静态初始化的条件变量，无需调用*pthread_cond_destroy()*。

```
#include <pthread.h>

int `pthread_cond_destroy`(pthread_cond_t **cond*);
```

### 注意

成功时返回 0，出错时返回正的错误号

只有在没有线程等待条件变量时，销毁该条件变量才是安全的。如果条件变量位于动态分配的内存区域中，则应该在释放该内存区域之前销毁该条件变量。自动分配的条件变量应在其宿主函数返回之前销毁。

已通过*pthread_cond_destroy()*销毁的条件变量可以通过*pthread_cond_init()*重新初始化。

## 总结

线程提供的更大共享带来了代价。线程化应用程序必须使用同步原语，如互斥锁和条件变量，以协调对共享变量的访问。互斥锁提供对共享变量的独占访问。条件变量允许一个或多个线程等待通知，直到另一个线程改变共享变量的状态。

#### 更多信息

请参考总结中列出的更多信息来源。

## 练习

1.  修改示例 30-1 程序（`thread_incr.c`），使线程的启动函数中的每个循环输出*glob*的当前值以及一个唯一标识该线程的标识符。线程的唯一标识符可以作为参数传递给用于创建线程的*pthread_create()*调用。对于这个程序，这要求改变线程启动函数的参数，使其成为指向包含唯一标识符和循环限制值的结构的指针。运行该程序，将输出重定向到文件中，然后检查文件，查看当内核调度器在两个线程之间交替执行时，*glob*发生了什么变化。

1.  实现一组线程安全的函数，用于更新和搜索一个不平衡的二叉树。这个库应包括以下形式的函数（具有明显的功能）：

    ```
    initialize(tree);
    add(tree, char *key, void *value);
    delete(tree, char *key)
    Boolean lookup(char *key, void **value)
    ```

    在上述原型中，*树*是一种指向树根的结构（你需要为此目的定义一个合适的结构）。树的每个元素都包含一个键值对。你还需要为每个元素定义一个结构，包含一个互斥量来保护该元素，以确保每次只有一个线程能够访问它。*initialize()*、*add()*和*lookup()*函数相对容易实现。*delete()*操作则需要稍微更多的工作。

    ### 注意

    不再需要维护平衡树，极大简化了实现中的锁定要求，但也带来了某些输入模式可能导致树性能较差的风险。维护平衡树需要在*add()*和*delete()*操作期间在子树之间移动节点，这需要更复杂的锁定策略。
