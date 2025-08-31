## 第三十二章。线程：线程取消

通常，多个线程并行执行，每个线程执行任务，直到决定通过调用 *pthread_exit()* 或从线程的启动函数返回来终止。

有时，取消线程是有用的；即，向它发送请求，要求它立即终止。例如，如果一组线程正在执行计算，而其中一个线程检测到需要其他线程终止的错误条件，那么就很有用。或者，图形界面驱动的应用程序可能会提供一个取消按钮，允许用户终止正在后台线程中执行的任务；在这种情况下，主线程（控制 GUI）需要告诉后台线程终止。

在本章中，我们将描述 POSIX 线程取消机制。

## 取消线程

*pthread_cancel()* 函数向指定的 *thread* 发送取消请求。

```
#include <pthread.h>

int `pthread_cancel` (pthread_t *thread*);
```

### 注意

成功时返回 0，错误时返回正的错误码

在发出取消请求后，*pthread_cancel()* 会立即返回；也就是说，它不会等待目标线程终止。

目标线程会发生什么，以及何时发生，取决于该线程的取消状态和类型，如下一节所述。

## 取消状态和类型

*pthread_setcancelstate()* 和 *pthread_setcanceltype()* 函数设置标志，使线程能够控制如何响应取消请求。

```
#include <pthread.h>

int `pthread_setcancelstate`(int *state*, int **oldstate*);
int `pthread_setcanceltype`(int *type*, int **oldtype*);
```

### 注意

成功时返回 0，错误时返回正的错误码

*pthread_setcancelstate()* 函数将调用线程的取消状态设置为 *state* 中给定的值。该参数有以下值之一：

`PTHREAD_CANCEL_DISABLE`

该线程不可取消。如果收到取消请求，则它保持挂起，直到取消状态被启用。

`PTHREAD_CANCEL_ENABLE`

该线程是可取消的。这是新创建线程的默认取消状态。

线程的先前取消状态会返回到 *oldstate* 所指向的位置。

### 注意

如果我们不关心之前的取消状态，Linux 允许将*oldstate*指定为 `NULL`。在许多其他实现中也是如此；然而，SUSv3 没有规定此功能，因此可移植的应用程序不能依赖它。我们应该始终为*oldstate*指定一个非 `NULL` 的值。

临时禁用取消（`PTHREAD_CANCEL_DISABLE`）在线程执行一段必须*全部*完成的代码时非常有用。

如果线程是可取消的（`PTHREAD_CANCEL_ENABLE`），则取消请求的处理由线程的取消类型决定，取消类型通过调用 *pthread_setcanceltype()* 中的 *type* 参数指定。该参数有以下值之一：

`PTHREAD_CANCEL_ASYNCHRONOUS`

线程可以在任何时候被取消（可能，但不一定，立即）。异步取消性很少有用，我们将在第 32.6 节中讨论它。

`PTHREAD_CANCEL_DEFERRED`

取消保持待处理状态，直到达到取消点（请参见下一节）。这是新创建线程中的默认取消性类型。我们将在后续章节中进一步讨论延迟取消性。

线程的先前可取消性类型将被返回到*oldtype*所指向的位置。

### 注意

与*pthread_setcancelstate() oldstate*参数一样，许多实现，包括 Linux，允许*oldtype*指定为`NULL`，如果我们不关心先前的可取消性类型。再次强调，SUSv3 并未指定此功能，因此可移植的应用程序不能依赖此功能。我们应该始终为*oldtype*指定一个非`NULL`值。

当线程调用*fork()*时，子线程继承调用线程的可取消性类型和状态。当线程调用*exec()*时，新程序的主线程的可取消性类型和状态会被重置为`PTHREAD_CANCEL_ENABLE`和`PTHREAD_CANCEL_DEFERRED`。

## 取消点

当启用并延迟取消时，取消请求仅在线程下次达到*取消点*时才会被处理。取消点是调用实现定义的一组函数之一。

SUSv3 规定，表 32-1 中显示的函数*必须*是取消点（如果它们由实现提供）。其中大多数是能够使线程阻塞无限期的函数。

表 32-1。SUSv3 要求作为取消点的函数

| *accept()**aio_suspend()**clock_nanosleep()**close()**connect()**creat()**fcntl(F_SETLKW)**fsync()**fdatasync()**getmsg()**getpmsg()**lockf(F_LOCK)**mq_receive()**mq_send()**mq_timedreceive()**mq_timedsend()**msgrcv()**msgsnd()**msync()* | *nanosleep()**open()**pause()**poll()**pread()**pselect()**pthread_cond_timedwait()**pthread_cond_wait()**pthread_join()**pthread_testcancel()**putmsg()**putpmsg()**pwrite()**read()**readv()**recv()**recvfrom()**recvmsg()**select()* | *sem_timedwait()**sem_wait()**send()**sendmsg()**sendto()**sigpause()**sigsuspend()**sigtimedwait()**sigwait()**sigwaitinfo()**sleep()**system()**tcdrain()**usleep()**wait()**waitid()**waitpid()**write()**writev()* |
| --- | --- | --- |

除了 表 32-1 中的函数外，SUSv3 还指定了一个更大的函数组，实施可以将其定义为取消点。这些包括 *stdio* 函数、*dlopen* API、*syslog* API、*nftw()*、*popen()*、*semop()*、*unlink()* 和各种从系统文件（如 *utmp* 文件）中检索信息的函数。便携程序必须正确处理调用这些函数时线程可能被取消的情况。

SUSv3 指定，除了必须和可能是取消点的函数列表外，标准中其他的函数不能作为取消点（即，便携程序不需要处理调用这些其他函数时可能导致线程取消的情况）。

SUSv4 将 *openat()* 添加到必须作为取消点的函数列表中，并移除 *sigpause()*（它移到可以作为取消点的函数列表）和 *usleep()*（已从标准中删除）。

### 注意

实现可以标记标准中未指定的其他函数作为取消点。任何可能阻塞的函数（可能因为它可能访问文件）都可能成为取消点的候选函数。在 *glibc* 中，许多非标准函数因为这个原因被标记为取消点。

在接收到取消请求时，启用了取消性且取消被延迟的线程会在下次达到取消点时终止。如果线程没有被分离，则必须有其他线程在进程中与之连接，以防止它变成僵尸线程。当一个被取消的线程被连接时，*pthread_join()* 的第二个参数返回的是一个特殊的线程返回值：`PTHREAD_CANCELED`。

#### 示例程序

示例 32-1 取消线程")展示了一个简单的使用 *pthread_cancel()* 的例子。主程序创建一个执行无限循环的线程，该线程每秒睡眠一次并打印循环计数器的值。（该线程只有在收到取消请求或进程退出时才会终止。）与此同时，主程序睡眠 3 秒钟，然后向它创建的线程发送取消请求。当我们运行该程序时，看到以下输出：

```
$ `./t_pthread_cancel`
New thread started
Loop 1
Loop 2
Loop 3
Thread was canceled
```

示例 32-1. 使用*pthread_cancel()*取消线程

```
`threads/thread_cancel.c`
#include <pthread.h>
#include "tlpi_hdr.h"

static void *
threadFunc(void *arg)
{
    int j;
    printf("New thread started\n");     /* May be a cancellation point */
    for (j = 1; ; j++) {
        printf("Loop %d\n", j);         /* May be a cancellation point */
        sleep(1);                       /* A cancellation point */
    }

    /* NOTREACHED */
    return NULL;
}

int
main(int argc, char *argv[])
{
    pthread_t thr;
    int s;
    void *res;

    s = pthread_create(&thr, NULL, threadFunc, NULL);
    if (s != 0)
        errExitEN(s, "pthread_create");

    sleep(3);                           /* Allow new thread to run a while */

    s = pthread_cancel(thr);
    if (s != 0)
        errExitEN(s, "pthread_cancel");

    s = pthread_join(thr, &res);
    if (s != 0)
        errExitEN(s, "pthread_join");

    if (res == PTHREAD_CANCELED)
        printf("Thread was canceled\n");
    else
        printf("Thread was not canceled (should not happen!)\n");

    exit(EXIT_SUCCESS);
}
      `threads/thread_cancel.c`
```

## 测试线程取消

在示例 32-1 取消线程")中，*main()*创建的线程接受了取消请求，因为它执行了一个取消点的函数（*sleep()*是一个取消点；*printf()*可能也是一个取消点）。然而，假设一个线程执行一个不包含取消点的循环（例如一个计算密集型循环）。在这种情况下，线程将永远不会响应取消请求。

*pthread_testcancel()*的目的是作为一个取消点。当此函数被调用时，如果有挂起的取消请求，调用线程将被终止。

```
#include <pthread.h>

void `pthread_testcancel`(void);
```

执行不包含取消点的代码的线程可以定期调用*pthread_testcancel()*，以确保及时响应另一个线程发送的取消请求。

## 清理处理程序

如果一个线程在到达取消点时被简单地终止，那么共享变量和 Pthreads 对象（例如互斥锁）可能会处于不一致的状态，可能会导致进程中的其余线程产生错误的结果、死锁或崩溃。为了解决这个问题，线程可以建立一个或多个*清理处理程序*——如果线程被取消，这些函数会自动执行。清理处理程序可以执行一些任务，比如在终止线程之前修改全局变量的值和解锁互斥锁。

每个线程可以有一个清理处理程序栈。当线程被取消时，清理处理程序按从栈顶开始的顺序执行；也就是说，最近建立的处理程序首先被调用，然后是下一个最近建立的，以此类推。当所有清理处理程序执行完毕后，线程终止。

*pthread_cleanup_push()*和*pthread_cleanup_pop()*函数分别在调用线程的清理处理程序栈上添加和移除处理程序。

```
#include <pthread.h>

void `pthread_cleanup_push`(void (**routine*)(void*), void **arg*);
void `pthread_cleanup_pop`(int *execute*);
```

*pthread_cleanup_push()*函数将*例程*中指定的函数地址添加到调用线程的清理处理程序栈的顶部。*例程*参数是指向具有以下形式的函数的指针：

```
void
routine(void *arg)
{
    /* Code to perform cleanup */
}
```

给定给*pthread_cleanup_push()*的*arg*值作为清理处理程序调用时的参数传递。这个参数被定义为*void **，但通过适当的类型转换，其他数据类型也可以传递到该参数中。

通常，只有在线程在执行特定代码段时被取消时，才需要清理操作。如果线程在没有被取消的情况下到达该代码段的末尾，那么清理操作就不再需要。因此，每次调用*pthread_cleanup_push()*时，都需要有相应的*pthread_cleanup_pop()*调用。此函数从清理处理程序的栈中移除最上面的函数。如果*execute*参数非零，则该处理程序也会被执行。如果我们希望执行清理操作，即使线程没有被取消，这也很方便。

尽管我们将*pthread_cleanup_push()*和*pthread_cleanup_pop()*描述为函数，但 SUSv3 允许它们作为宏实现，宏展开为包括打开（`{`）和关闭（`}`）大括号的语句序列。并非所有 UNIX 实现都是这样做的，但 Linux 和许多其他实现是如此。这意味着每次使用*pthread_cleanup_push()*时，必须在同一个词法块内配对一个对应的*pthread_cleanup_pop()*。（在采用这种方式的实现中，*pthread_cleanup_push()*和*pthread_cleanup_pop()*之间声明的变量将仅限于该词法范围内。）例如，写出如下代码是不正确的：

```
pthread_cleanup_push(func, arg);
...
if (cond) {
    pthread_cleanup_pop(0);
}
```

作为一种编码便利性，如果线程通过调用*pthread_exit()*终止，任何未弹出的清理处理程序也会自动执行（但如果只是简单的`return`，则不会执行）。

#### 示例程序

示例 32-2 中的程序提供了一个简单的清理处理程序使用示例。主程序创建了一个线程 ![](img/U008.png)，该线程的第一步是分配一块内存 ![](img/U003.png)，其位置存储在*buf*中，然后锁定互斥量*mtx* ![](img/U004.png)。由于线程可能被取消，因此它使用*pthread_cleanup_push()* ![](img/U005.png)安装一个清理处理程序，该处理程序在调用时会释放存储在*buf*中的内存并解锁互斥量 ![](img/U002.png)。

然后线程进入一个循环，等待条件变量*cond*被信号触发 ![](img/U006.png)。这个循环将在两种方式之一中终止，具体取决于程序是否提供了命令行参数：

+   如果没有提供命令行参数，则线程会被*main()*取消 ![](img/U009.png)。在这种情况下，取消将在调用*pthread_cond_wait()*时发生 ![](img/U006.png)，这是表 32-1 中列出的取消点之一。作为取消的一部分，使用*pthread_cleanup_push()*建立的清理处理程序会自动被调用。

+   如果提供了命令行参数，则条件变量在关联的全局变量*glob*首次设置为非零值后会被触发信号 ![](img/U010.png)。在这种情况下，线程会继续执行*pthread_cleanup_pop()* ![](img/U007.png)，该函数在给定非零参数时也会触发清理处理程序。

主程序与已终止的线程进行连接 ![](img/U011.png)，并报告线程是被取消还是正常终止。

示例 32-2. 使用清理处理程序

```
`threads/thread_cleanup.c`
    #include <pthread.h>
    #include "tlpi_hdr.h"

    static pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
    static pthread_mutex_t mtx = PTHREAD_MUTEX_INITIALIZER;
    static int glob = 0;                    /* Predicate variable */

    static void     /* Free memory pointed to by 'arg' and unlock mutex */
    cleanupHandler(void *arg)
    {
        int s;

        printf("cleanup: freeing block at %p\n", arg);
    free(arg);

        printf("cleanup: unlocking mutex\n");
    s = pthread_mutex_unlock(&mtx);
        if (s != 0)
            errExitEN(s, "pthread_mutex_unlock");
    }

    static void *
    threadFunc(void *arg)
    {
        int s;
        void *buf = NULL;                   /* Buffer allocated by thread */

    buf = malloc(0x10000);              /* Not a cancellation point */
        printf("thread:  allocated memory at %p\n", buf);

    s = pthread_mutex_lock(&mtx);       /* Not a cancellation point */
        if (s != 0)
            errExitEN(s, "pthread_mutex_lock");

    pthread_cleanup_push(cleanupHandler, buf);

        while (glob == 0) {
        s = pthread_cond_wait(&cond, &mtx);    /* A cancellation point */
            if (s != 0)
                errExitEN(s, "pthread_cond_wait");
        }

        printf("thread:  condition wait loop completed\n");
    pthread_cleanup_pop(1);             /* Executes cleanup handler */
        return NULL;
    }

    int
    main(int argc, char *argv[])
    {
        pthread_t thr;
        void *res;
        int s;

        s = pthread_create(&thr, NULL, threadFunc, NULL);
        if (s != 0)
            errExitEN(s, "pthread_create");

        sleep(2);                   /* Give thread a chance to get started */

        if (argc == 1) {            /* Cancel thread */
            printf("main:    about to cancel thread\n");
        s = pthread_cancel(thr);
            if (s != 0)
                errExitEN(s, "pthread_cancel");

        } else {                    /* Signal condition variable */
            printf("main:    about to signal condition variable\n");
            glob = 1;
        s = pthread_cond_signal(&cond);
            if (s != 0)
                errExitEN(s, "pthread_cond_signal");
        }

    s = pthread_join(thr, &res);
        if (s != 0)
            errExitEN(s, "pthread_join");
        if (res == PTHREAD_CANCELED)
            printf("main:    thread was canceled\n");
        else
            printf("main:    thread terminated normally\n");

        exit(EXIT_SUCCESS);
    }
          `threads/thread_cleanup.c`
```

如果我们在没有任何命令行参数的情况下调用示例 32-2 中的程序，则*main()*调用*pthread_cancel()*，清理处理程序会自动被调用，我们会看到以下情况：

```
$ .`/thread_cleanup`
thread:  allocated memory at 0x804b050
main:    about to cancel thread
cleanup: freeing block at 0x804b050
cleanup: unlocking mutex
main:    thread was canceled
```

如果我们在命令行中提供参数，则*main()*将*glob*设置为 1 并触发条件变量，清理处理程序由*pthread_cleanup_pop()*调用，我们可以看到以下情况：

```
$ `./thread_cleanup s`
thread:  allocated memory at 0x804b050
main:    about to signal condition variable
thread:  condition wait loop completed
cleanup: freeing block at 0x804b050
cleanup: unlocking mutex
main:    thread terminated normally
```

## 异步取消

当线程被设置为异步可取消（取消类型`PTHREAD_CANCEL_ASYNCHRONOUS`）时，它可以在任何时候被取消（即在任何机器语言指令处）；取消的传递不会等到线程下次到达取消点才进行。

异步取消的问题在于，尽管清理处理程序仍然会被调用，但处理程序无法确定线程的状态。在示例 32-2 中使用的是延迟取消类型，线程只能在执行调用*pthread_cond_wait()*时被取消，因为它是唯一的取消点。到那时，我们知道*buf*已经初始化并指向一块已分配的内存，且互斥锁*mtx*已被加锁。然而，对于异步取消，线程可能在任何点被取消；例如，在*malloc()*调用之前、*malloc()*调用和加锁互斥锁之间，或者在加锁互斥锁之后。清理处理程序无法知道取消发生的位置，也无法确定需要哪些清理步骤。此外，线程甚至可能在*malloc()*调用期间被取消，之后可能会导致混乱（*malloc()* 和 *free()* 的实现 和 free() 的实现")）。

一般而言，一个可异步取消的线程不能分配任何资源，也不能获取任何互斥锁、信号量或锁。这排除了使用许多库函数，包括大多数 Pthreads 函数。（SUSv3 对*pthread_cancel()*、*pthread_setcancelstate()*和*pthread_setcanceltype()*有例外，这些函数被明确要求是*异步取消安全*的；也就是说，必须使这些函数能够安全地从一个可异步取消的线程中调用。）换句话说，异步取消的有用场景非常少。一个这样的场景是取消一个处于计算密集型循环中的线程。

## 总结

*pthread_cancel()*函数允许一个线程向另一个线程发送取消请求，这是一种请求目标线程终止的请求。

目标线程如何响应该请求由其取消状态和类型决定。如果取消状态当前设置为禁用，则该请求将保持挂起，直到取消状态设置为启用。如果启用了取消性，则取消类型决定目标线程何时响应该请求。如果类型是延迟的，则取消发生在线程下次调用 SUSv3 指定的一个或多个取消点函数时。如果类型是异步的，则可以在任何时候发生取消（但这种情况很少有用）。

线程可以建立一个清理处理程序的堆栈，这些程序是程序员定义的函数，当线程被取消时，它们会自动调用以执行清理工作（例如，恢复共享变量的状态或解锁互斥锁）。

#### 进一步信息

请参阅总结中列出的进一步信息来源。
