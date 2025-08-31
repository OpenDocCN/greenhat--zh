## 第四十八章. System V 共享内存

本章描述了 System V 共享内存。共享内存允许两个或更多进程共享相同的物理内存区域（通常称为*段*）。由于共享内存段成为进程用户空间内存的一部分，因此不需要内核干预进行进程间通信。所需的唯一操作是一个进程将数据复制到共享内存中；这些数据会立即对所有共享相同段的其他进程可用。与管道或消息队列等技术相比，这种方法提供了快速的进程间通信，因为在管道或消息队列中，发送进程需要将数据从用户空间的缓冲区复制到内核内存中，而接收进程则需要执行反向复制。（每个进程还需要执行系统调用来执行复制操作。）

另一方面，使用共享内存的进程间通信（IPC）不经过内核调度，这意味着通常需要某种同步方法，以防止进程同时访问共享内存（例如，两个进程同时进行更新，或一个进程在另一个进程更新共享内存时从中读取数据）。System V 信号量是进行此类同步的自然方法。其他方法，如 POSIX 信号量（第五十三章") 和文件锁（第五十五章")，也是可行的。

### 注

在*mmap()*术语中，内存区域是在某个地址上*映射*的，而在 System V 术语中，共享内存段是在某个地址上*附加*的。这些术语是等效的，术语差异源于这两个 API 的不同起源。

## 概述

为了使用共享内存段，通常需要执行以下步骤：

+   调用*shmget()*来创建一个新的共享内存段，或获取现有段的标识符（即，其他进程创建的段）。此调用返回一个共享内存标识符，以供后续调用使用。

+   使用*shmat()*来*附加*共享内存段；即使该段成为调用进程虚拟内存的一部分。

+   此时，共享内存段可以像程序中的任何其他内存一样进行处理。为了引用共享内存，程序使用*shmat()*调用返回的*addr*值，该值是进程虚拟地址空间中指向共享内存段开始处的指针。

+   调用*shmdt()*来分离共享内存段。调用此函数后，进程将无法再引用共享内存。此步骤是可选的，并且在进程终止时会自动执行。

+   调用*shmctl()*来删除共享内存段。该段只有在所有当前附加的进程都已分离它之后才会被销毁。只有一个进程需要执行此步骤。

## 创建或打开共享内存段

*shmget()*系统调用创建一个新的共享内存段或获取现有段的标识符。新创建的共享内存段的内容会初始化为 0。

```
#include <sys/types.h>        /* For portability */
#include <sys/shm.h>

int `shmget`(key_t *key*, size_t *size*, int *shmflg*);
```

### 注意

成功时返回共享内存段标识符，出错时返回-1。

*key* 参数是使用 IPC Keys 中描述的某种方法生成的密钥（即，通常是`IPC_PRIVATE`值或*ftok()*返回的密钥）。

当我们使用*shmget()*创建一个新的共享内存段时，*size*指定一个正整数，表示段的期望大小，以字节为单位。内核以系统页面大小的倍数分配共享内存，因此*size*实际上会向上舍入为系统页面大小的下一个倍数。如果我们使用*shmget()*获取一个现有段的标识符，则*size*对段没有影响，但它必须小于或等于该段的大小。

*shmflg*参数执行与其他 IPC *get*调用相同的任务，指定新共享内存段的权限（表 15-4，见常规文件的权限），或者检查现有段的权限。此外，零个或多个以下标志可以与*shmflg*通过 OR（|）运算结合，以控制*shmget()*的操作：

`IPC_CREAT`

如果指定的*key*没有对应的共享内存段，则创建一个新的共享内存段。

`IPC_EXCL`

如果也指定了`IPC_CREAT`，且指定的*key*已经存在，则返回错误`EEXIST`。

上述标志在 45.1 节中有更详细的描述。此外，Linux 还允许以下非标准标志：

`SHM_HUGETLB`（自 Linux 2.6 起）

一个特权 (`CAP_IPC_LOCK`) 进程可以使用此标志来创建一个使用*大页*的共享内存段。大页是许多现代硬件架构提供的一项功能，旨在通过使用非常大的页面大小来管理内存。（例如，x86-32 允许使用 4-MB 页面作为 4-kB 页面的替代。）在具有大量内存的系统上，且应用程序需要大块内存时，使用大页可以减少硬件内存管理单元的转换后备缓冲区（TLB）中所需的条目数。这是有益的，因为 TLB 中的条目通常是稀缺资源。有关更多信息，请参见内核源文件`Documentation/vm/hugetlbpage.txt`。

`SHM_NORESERVE`（自 Linux 2.6.15 起）

此标志在*shmget()*中与`MAP_NORESERVE`标志在*mmap()*中的作用相同。参见 49.9 节。

成功时，*shmget()*返回新创建或现有共享内存段的标识符。

## 使用共享内存

*shmat()*系统调用将由*shmid*标识的共享内存段附加到调用进程的虚拟地址空间。

```
#include <sys/types.h>        /* For portability */
#include <sys/shm.h>

void *`shmat`(int *shmid*, const void **shmaddr*, int *shmflg*);
```

### 注意

成功时返回附加的共享内存地址，出错时返回 *(void *)* -1。

*shmaddr* 参数和 `SHM_RND` 位在 *shmflg* 位掩码参数中的设置控制着段如何附加：

+   如果 *shmaddr* 是 `NULL`，则段会附加在内核选择的合适地址处。这是附加段的首选方法。

+   如果 *shmaddr* 不是 `NULL`，并且没有设置 `SHM_RND`，则段将在 *shmaddr* 指定的地址附加，该地址必须是系统页面大小的倍数（否则会返回 `EINVAL` 错误）。

+   如果 *shmaddr* 不是 `NULL`，并且设置了 `SHM_RND`，那么段将在 *shmaddr* 提供的地址处附加，并向下舍入到常量 `SHMLBA` (*共享内存低边界地址*) 的最接近倍数。该常量等于系统页面大小的某个倍数。在某些架构上，为了提高 CPU 缓存性能并防止同一段的不同附加在 CPU 缓存中具有不一致的视图，必须在 `SHMLBA` 的倍数地址上附加段。

### 注意

在 x86 架构上，`SHMLBA`与系统页面大小相同，反映出在这些架构上不会出现这种缓存不一致的情况。

为 *shmaddr* 指定一个非`NULL`值（即上面列出的第二或第三个选项）并不推荐，原因如下：

+   它降低了应用程序的可移植性。在一种 UNIX 实现上有效的地址可能在另一种实现上无效。

+   如果尝试在某个特定地址附加共享内存段，而该地址已被占用，则附加会失败。例如，如果应用程序（可能在库函数内部）已经在该地址附加了另一个段或创建了内存映射，则可能会发生这种情况。

作为其函数结果，*shmat()* 返回共享内存段附加的地址。这个值可以像普通的 C 指针一样处理；该段看起来就像进程虚拟内存中的任何其他部分。通常，我们将 *shmat()* 的返回值赋给一个指向程序员定义的结构体的指针，以便将该结构体施加到该段上（例如，参见 示例 48-2）。

要附加一个只读访问的共享内存段，我们在 *shmflg* 中指定标志 `SHM_RDONLY`。尝试更新只读段的内容会导致段错误（`SIGSEGV`信号）。如果没有指定`SHM_RDONLY`，则内存既可以读取也可以修改。

要附加共享内存段，进程需要对该段具有读写权限，除非指定了`SHM_RDONLY`，在这种情况下，只需要读权限。

### 注意

在一个进程中，可以多次附加同一个共享内存段，甚至可以让一个附加为只读，另一个附加为读写。每个附加点的内存内容都是相同的，因为进程虚拟内存页表中的不同条目指向相同的物理内存页。

另一个可以在*shmflg*中指定的值是`SHM_REMAP`。在这种情况下，*shmaddr*必须是非`NULL`。该标志请求*shmat()*调用替换从*shmaddr*开始并持续共享内存段长度范围内的任何现有共享内存附加或内存映射。通常，如果我们尝试在已经使用的地址范围附加共享内存段，会导致`EINVAL`错误。`SHM_REMAP`是一个非标准的 Linux 扩展。

表 48-1 总结了可以在*shmat()*的*shmflg*参数中使用的常量。

当一个进程不再需要访问共享内存段时，它可以调用*shmdt()*将该段从其虚拟地址空间中分离。*shmaddr*参数用于标识要分离的内存段。它应该是先前调用*shmat()*时返回的值。

```
#include <sys/types.h>        /* For portability */
#include <sys/shm.h>

int `shmdt`(const void **shmaddr*);
```

### 注意

成功时返回 0，出错时返回-1

分离共享内存段与删除它不同。删除操作是通过*shmctl()*的`IPC_RMID`操作执行的，具体描述见第 48.7 节。

由*fork()*创建的子进程会继承父进程附加的共享内存段。因此，共享内存提供了一种父子进程之间进行 IPC 的简便方法。

在*exec()*过程中，所有附加的共享内存段都将被分离。进程终止时，共享内存段也会自动分离。

表 48-1. *shmflg* 位掩码值（适用于*shmat()*)

| 值 | 描述 |
| --- | --- |
| `SHM_RDONLY` | 以只读方式附加段 |
| `SHM_REMAP` | 替换*shmaddr*处的任何现有映射 |
| `SHM_RND` | 将*shmaddr*向下舍入为`SHMLBA`字节的倍数 |

## 示例：通过共享内存传输数据

现在我们来看一个使用 System V 共享内存和信号量的示例应用程序。该应用程序由两个程序组成：*writer*和*reader*。writer 从标准输入读取数据块，并将其复制（“写入”）到共享内存段中。reader 将共享内存段中的数据块复制（“读取”）到标准输出中。实际上，程序将共享内存当作管道使用。

这两个程序使用一对 System V 信号量，在二进制信号量协议中（实现二进制信号量协议中定义的*initSemAvailable()*, *initSemInUse()*, *reserveSem()*和*releaseSem()*函数）确保：

+   只有一个进程在任何时候访问共享内存段；并且

+   进程交替访问共享内存段（即，写入程序写入一些数据，然后读取程序读取这些数据，接着写入程序再次写入，以此类推）。

图 48-1 提供了这两个信号量使用的概览。请注意，写入程序初始化了这两个信号量，使得它可以是两个程序中第一个能够访问共享内存段的程序；也就是说，写入程序的信号量最初是可用的，而读取程序的信号量最初是正在使用中的。

应用程序的源代码由三个文件组成。其中第一个，示例 48-1，是一个被读取程序和写入程序共享的头文件。此头文件定义了我们用来声明指向共享内存段的指针的*shmseg*结构体。这样做使我们可以对共享内存段的字节施加结构。

![使用信号量确保对共享内存的独占、交替访问](img/48-1_SVSHM-shm_xfr.png.jpg)图 48-1. 使用信号量确保对共享内存的独占、交替访问示例 48-1. `svshm_xfr_writer.c` 和 `svshm_xfr_reader.c` 的头文件

```
`svshm/svshm_xfr.h`
#include <sys/types.h>
#include <sys/stat.h>
#include <sys/sem.h>
#include <sys/shm.h>
#include "binary_sems.h"        /* Declares our binary semaphore functions */
#include "tlpi_hdr.h"

#define SHM_KEY 0x1234          /* Key for shared memory segment */
#define SEM_KEY 0x5678          /* Key for semaphore set */

#define OBJ_PERMS (S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP)
                                /* Permissions for our IPC objects */

#define WRITE_SEM 0             /* Writer has access to shared memory */
#define READ_SEM 1              /* Reader has access to shared memory */

#ifndef BUF_SIZE                /* Allow "cc -D" to override definition */
#define BUF_SIZE 1024           /* Size of transfer buffer */
#endif

struct shmseg {                 /* Defines structure of shared memory segment */
    int cnt;                    /* Number of bytes used in 'buf' */
    char buf[BUF_SIZE];         /* Data being transferred */
};
      `svshm/svshm_xfr.h`
```

示例 48-2 是写入程序。此程序执行以下步骤：

+   创建一个包含写入程序和读取程序使用的两个信号量的集合，以确保它们交替访问共享内存段 ![](img/U001.png)。这些信号量被初始化，使得写入程序可以首先访问共享内存段。由于写入程序创建了信号量集合，它必须在读取程序之前启动。

+   创建共享内存段并将其附加到写入程序的虚拟地址空间，地址由系统选择 ![](img/U002.png)。

+   进入一个循环，将数据从标准输入传输到共享内存段 ![](img/U003.png)。在每次循环迭代中执行以下步骤：

    +   保留（递减）写入程序的信号量 ![](img/U004.png)。

    +   从标准输入读取数据到共享内存段 ![](img/U005.png)。

    +   释放（递增）读取程序的信号量 ![](img/U006.png)。

+   当标准输入没有更多数据时，循环终止 ![](img/U007.png)。在循环的最后一次迭代中，写入程序通过传递一个长度为 0 的数据块（*shmp -> cnt* 为 0）通知读取程序没有更多数据。

+   在退出循环时，写入程序再次保留其信号量，以便知道读取程序已经完成了对共享内存的最终访问 ![](img/U008.png)。然后，写入程序移除共享内存段和信号量集合 ![](img/U009.png)。

示例 48-3 是读取程序。它将共享内存段中的数据块传输到标准输出。读取程序执行以下步骤：

+   获取由写入程序创建的信号量集和共享内存段的 ID ![](img/U001.png)。

+   为只读访问附加共享内存段 ![](img/U002.png)。

+   进入一个循环，传输共享内存段中的数据 ![](img/U003.png)。在每次循环迭代中执行以下步骤：

    +   保留（减少）读取信号量 ![](img/U004.png)。

    +   检查 *shmp -> cnt* 是否为 0；如果是，则退出此循环 ![](img/U005.png)。

    +   将共享内存段中的数据块写入标准输出 ![](img/U006.png)。

    +   释放（增加）写入信号量 ![](img/U007.png)。

+   在退出循环后，分离共享内存段 ![](img/U008.png) 并释放写入信号量 ![](img/U009.png)，这样写入程序就可以移除 IPC 对象。

示例 48-2. 从 *stdin* 向 System V 共享内存段传输数据块

```
`svshm/svshm_xfr_writer.c`
    #include "semun.h"              /* Definition of semun union */
    #include "svshm_xfr.h"

    int
    main(int argc, char *argv[])
    {
        int semid, shmid, bytes, xfrs;
        struct shmseg *shmp;
        union semun dummy;

    semid = semget(SEM_KEY, 2, IPC_CREAT | OBJ_PERMS);
        if (semid == -1)
            errExit("semget");

            if (initSemAvailable(semid, WRITE_SEM) == -1)
            errExit("initSemAvailable");
        if (initSemInUse(semid, READ_SEM) == -1)
            errExit("initSemInUse");

    shmid = shmget(SHM_KEY, sizeof(struct shmseg), IPC_CREAT | OBJ_PERMS);
        if (shmid == -1)
            errExit("shmget");

        shmp = shmat(shmid, NULL, 0);
        if (shmp == (void *) -1)
            errExit("shmat");

        /* Transfer blocks of data from stdin to shared memory */

    for (xfrs = 0, bytes = 0; ; xfrs++, bytes += shmp->cnt) {

        if (reserveSem(semid, WRITE_SEM) == -1)         /* Wait for our turn */
                errExit("reserveSem");

        shmp->cnt = read(STDIN_FILENO, shmp->buf, BUF_SIZE);
            if (shmp->cnt == -1)
                errExit("read");

        if (releaseSem(semid, READ_SEM) == -1)          /* Give reader a turn */
                errExit("releaseSem");

            /* Have we reached EOF? We test this after giving the reader
               a turn so that it can see the 0 value in shmp->cnt. */

        if (shmp->cnt == 0)
                break;
        }

        /* Wait until reader has let us have one more turn. We then know
           reader has finished, and so we can delete the IPC objects. */

    if (reserveSem(semid, WRITE_SEM) == -1)
            errExit("reserveSem");

    if (semctl(semid, 0, IPC_RMID, dummy) == -1)
            errExit("semctl");
        if (shmdt(shmp) == -1)
            errExit("shmdt");
        if (shmctl(shmid, IPC_RMID, 0) == -1)
            errExit("shmctl");

        fprintf(stderr, "Sent %d bytes (%d xfrs)\n", bytes, xfrs);
        exit(EXIT_SUCCESS);
    }
          `svshm/svshm_xfr_writer.c`
```

示例 48-3. 从 System V 共享内存段向 *stdout* 传输数据块

```
`svshm/svshm_xfr_reader.c`
    #include "svshm_xfr.h"

    int
    main(int argc, char *argv[])
    {
        int semid, shmid, xfrs, bytes;
        struct shmseg *shmp;

        /* Get IDs for semaphore set and shared memory created by writer */

    semid = semget(SEM_KEY, 0, 0);
        if (semid == -1)
            errExit("semget");

        shmid  = shmget(SHM_KEY, 0, 0);
        if (shmid == -1)
            errExit("shmget");

    shmp = shmat(shmid, NULL, SHM_RDONLY);
        if (shmp == (void *) -1)
            errExit("shmat");

        /* Transfer blocks of data from shared memory to stdout */

    for (xfrs = 0, bytes = 0; ; xfrs++) {

        if (reserveSem(semid, READ_SEM) == -1)          /* Wait for our turn */
                errExit("reserveSem");

        if (shmp->cnt == 0)                    /* Writer encountered EOF */
                break;
            bytes += shmp->cnt;

        if (write(STDOUT_FILENO, shmp->buf, shmp->cnt) != shmp->cnt)
                fatal("partial/failed write");

        if (releaseSem(semid, WRITE_SEM) == -1)         /* Give writer a turn */
                errExit("releaseSem");
        }

    if (shmdt(shmp) == -1)
            errExit("shmdt");

        /* Give writer one more turn, so it can clean up */

    if (releaseSem(semid, WRITE_SEM) == -1)
            errExit("releaseSem");

        fprintf(stderr, "Received %d bytes (%d xfrs)\n", bytes, xfrs);
        exit(EXIT_SUCCESS);
    }
          `svshm/svshm_xfr_reader.c`
```

以下 shell 会话演示了在示例 48-2 和示例 46-9 中使用程序的方法。我们调用写入程序，使用文件`/etc/services`作为输入，然后调用读取程序，将其输出重定向到另一个文件：

```
$ `wc -c /etc/services`                               *Display size of test file*
764360 /etc/services
$ `./svshm_xfr_writer < /etc/services &`
[1] 9403
$ `./svshm_xfr_reader > out.txt`
Received 764360 bytes (747 xfrs)                    *Message from reader*
Sent 764360 bytes (747 xfrs)                        *Message from writer*
[1]+  Done              ./svshm_xfr_writer < /etc/services
$ `diff /etc/services out.txt`
$
```

*diff* 命令没有产生输出，表示读取程序生成的输出文件与写入程序使用的输入文件内容相同。

## 共享内存在虚拟内存中的位置

在进程的内存布局中，我们考虑了进程在虚拟内存中的各个部分的布局。重新审视这个话题对于附加 System V 共享内存段是非常有用的。如果我们遵循推荐的方法，允许内核选择共享内存段附加的位置，那么（在 x86-32 架构上）内存布局将如图 48-2 所示，内存段将被附加在向上增长的堆和向下增长的栈之间的未分配空间中。为了给堆和栈增长留出空间，共享内存段从虚拟地址`0x40000000`开始附加。映射映射（第四十九章）和共享库（第四十一章和第四十二章）也会被放置在这个区域。（共享内存映射和内存段放置的默认位置会有所不同，具体取决于内核版本以及进程的`RLIMIT_STACK`资源限制设置。）

### 注意

地址`0x40000000`被定义为内核常量`TASK_UNMAPPED_BASE`。通过定义这个常量为不同的值并重建内核，可以改变这个地址。

如果我们采用不推荐的方法，在调用*shmat()*（或*mmap()*）时显式指定一个地址，则共享内存段（或内存映射）可以被放置在`TASK_UNMAPPED_BASE`以下的地址。

使用 Linux 特有的`/proc/`*PID*`/maps`文件，我们可以看到程序映射的共享内存段和共享库的位置，如下面的 shell 会话所示。

### 注意

从内核 2.6.14 开始，Linux 还提供了`/proc/`*PID*`/smaps`文件，它展示了关于每个进程映射的内存消耗的更多信息。详细信息，请参阅*proc(5)*手册页。

![共享内存、内存映射和共享库的位置（x86-32）](img/48-2_SVSHM-shm-layout.png.jpg)图 48-2. 共享内存、内存映射和共享库的位置（x86-32）

在下面的 shell 会话中，我们使用了三种在本章中未展示的程序，但它们已在本书源代码分发的`svshm`子目录中提供。这些程序执行以下任务：

+   `svshm_create.c`程序创建一个共享内存段。这个程序接受与我们为消息队列（示例 46-1"), 在创建或打开消息队列）和信号量提供的相同的命令行选项，但增加了一个额外的参数，用于指定段的大小。

+   `svshm_attach.c`程序根据其命令行参数附加由共享内存标识符指定的共享内存段。每个参数都是由共享内存标识符和附加地址组成的冒号分隔的数字对。为附加地址指定 0 意味着系统应选择地址。程序显示内存实际附加的地址。为了提供信息，程序还显示了 SHMLBA 常量的值以及运行程序的进程 ID。

+   `svshm_rm.c`程序删除由其命令行参数指定的共享内存段。

我们通过创建两个共享内存段（分别为 100 kB 和 3200 kB 大小）开始 shell 会话：

```
$ `./svshm_create -p 102400`
9633796
$ `./svshm_create -p 3276800`
9666565
$ `./svshm_create -p 102400`
1015817
$ `./svshm_create -p 3276800`
1048586
```

然后我们启动一个程序，将这两个段附加到内核选择的地址：

```
$ `./svshm_attach 9633796:0 9666565:0`
SHMLBA = 4096 (0x1000), PID = 9903
1: 9633796:0 ==> 0xb7f0d000
2: 9666565:0 ==> 0xb7bed000
Sleeping 5 seconds
```

上面的输出显示了段附加的地址。在程序完成休眠之前，我们暂停它，然后检查对应的`/proc/`*PID*`/maps`文件的内容：

```
*Type Control-Z to suspend program*
[1]+  Stopped           ./svshm_attach 9633796:0 9666565:0
$ `cat /proc/9903/maps`
```

`cat`命令产生的输出如示例 48-4 所示。

示例 48-4. `/proc/`*PID*`/maps`的内容示例

```
$ `cat /proc/9903/maps`

    08048000-0804a000 r-xp 00000000 08:05 5526989  /home/mtk/svshm_attach
    0804a000-0804b000 r--p 00001000 08:05 5526989  /home/mtk/svshm_attach
    0804b000-0804c000 rw-p 00002000 08:05 5526989  /home/mtk/svshm_attach
 b7bed000-b7f0d000 rw-s 00000000 00:09 9666565  /SYSV00000000 (deleted)
    b7f0d000-b7f26000 rw-s 00000000 00:09 9633796  /SYSV00000000 (deleted)
    b7f26000-b7f27000 rw-p b7f26000 00:00 0
 b7f27000-b8064000 r-xp 00000000 08:06 122031   /lib/libc-2.8.so
    b8064000-b8066000 r--p 0013d000 08:06 122031   /lib/libc-2.8.so
    b8066000-b8067000 rw-p 0013f000 08:06 122031   /lib/libc-2.8.so
    b8067000-b806b000 rw-p b8067000 00:00 0
    b8082000-b8083000 rw-p b8082000 00:00 0
 b8083000-b809e000 r-xp 00000000 08:06 122125   /lib/ld-2.8.so
    b809e000-b809f000 r--p 0001a000 08:06 122125   /lib/ld-2.8.so
    b809f000-b80a0000 rw-p 0001b000 08:06 122125   /lib/ld-2.8.so
 bfd8a000-bfda0000 rw-p bffea000 00:00 0        [stack]
 ffffe000-fffff000 r-xp 00000000 00:00 0        [vdso]
```

在示例 48-4 中显示的`/proc/`*PID*`/maps`输出中，我们可以看到以下内容：

+   三行对应主程序`shm_attach`。这些行对应于程序的文本段和数据段！[](figs/web/U001.png)。其中第二行是一个只读页面，包含程序使用的字符串常量。

+   附带的 System V 共享内存段有两行！[](figs/web/U002.png)。

+   对应于两个共享库段的行。其中一个是标准 C 库（`libc`-*版本*.`so`）！[](figs/web/U003.png)。另一个是动态链接器（`ld`-*版本*.`so`），我们在使用共享库中描述过！[](figs/web/U004.png)。

+   一行标记为`[stack]`。这对应于进程栈！[](figs/web/U005.png)。

+   包含标签`[vdso]`的行 ![](img/U006.png)。这是*linux-gate*虚拟动态共享对象（DSO）的条目。该条目仅在 2.6.12 及以后的内核中出现。有关此条目的更多信息，请参见[`www.trilithium.com/johan/2005/08/linux-gate/`](http://www.trilithium.com/johan/2005/08/linux-gate/)。

`/proc/`*PID*`/maps`中每行显示的以下列，从左到右：

1.  一对由连字符分隔的数字，表示内存段映射的虚拟地址范围（以十六进制表示）。这两个数字中的第二个表示内存段末尾*之后*的下一个字节的地址。

1.  该内存段的保护和标志。前三个字母表示该段的保护方式：读取（`r`）、写入（`w`）和执行（`x`）。如果这些字母中的任何一个被一个连字符（`-`）替代，则表示相应的保护已被禁用。最后一个字母表示该内存段的映射标志；它可以是私有（`p`）的，也可以是共享（`s`）的。有关这些标志的解释，请参见第 49.2 节中对`MAP_PRIVATE`和`MAP_SHARED`标志的描述。（System V 共享内存段始终标记为共享。）

1.  对应映射文件中内存段的十六进制偏移量（以字节为单位）。当我们描述*mmap()*系统调用时，当前列和接下来的两列的含义会变得更加清晰，请参见第四十九章。对于 System V 共享内存段，偏移量始终为 0。

1.  对应映射文件所在设备的设备编号（主设备号和次设备号）。

1.  映射文件的 i-node 号，或者对于 System V 共享内存段，这是该段的标识符。

1.  与该内存段相关的文件名或其他标识标签。对于 System V 共享内存段，这个标签由字符串`SYSV`与该段的*shmget()键*（以十六进制表示）连接而成。在此示例中，`SYSV`后跟零，因为我们使用键`IPC_PRIVATE`（其值为 0）创建了这些段。`SYSV`字段后面的字符串`(deleted)`是 System V 共享内存段实现中的副产品。这些段作为映射文件在一个隐形的*tmpfs*文件系统中创建（参见虚拟内存文件系统：*tmpfs*），然后被取消链接。共享匿名内存映射的实现方式相同。（我们在第四十九章中描述了映射文件和共享匿名内存映射。）

## 在共享内存中存储指针

每个进程可能使用不同的共享库和内存映射，并且可能附加不同的共享内存段。因此，如果我们遵循推荐的做法，让内核选择共享内存段附加的位置，则该段可能会在每个进程中附加到不同的地址。因此，当我们在共享内存段中存储指向该段内其他地址的引用时，应该使用（相对的）偏移量，而不是（绝对的）指针。

例如，假设我们有一个共享内存段，其起始地址由 *baseaddr* 指向（即，*baseaddr* 是 *shmat()* 返回的值）。此外，在 *p* 指向的位置，我们想要存储一个指向与 *target* 指向的相同位置的指针，如图 48-3 所示。如果我们要在该段内构建一个链表或二叉树，这种操作就是典型的。设置 **p** 的常见 C 习惯用法如下：

```
*p = target;                    /* Place pointer in *p (WRONG!) */
```

![在共享内存段中使用指针](img/48-3_SVSHM-pointers-scale90.png.jpg)图 48-3. 在共享内存段中使用指针

这段代码的问题在于，当共享内存段在另一个进程中附加时，*target* 指向的位置可能位于不同的虚拟地址，这意味着在该进程中存储在 **p** 处的值是没有意义的。正确的方法是在 **p** 处存储一个偏移量，如下所示：

```
*p = (target - baseaddr);       /* Place offset in *p */
```

当解引用这些指针时，我们需要反转上述步骤：

```
target = baseaddr + *p;         /* Interpret offset */
```

在这里，我们假设在每个进程中，*baseaddr* 指向共享内存段的起始位置（即，它是每个进程中 *shmat()* 返回的值）。在此假设下，偏移量值将被正确解释，无论共享内存段在进程的虚拟地址空间中附加到哪里。

或者，如果我们将一组固定大小的结构链接在一起，我们可以将共享内存段（或其一部分）转换为数组，然后使用索引号作为从一个结构到另一个结构的“指针”。

## 共享内存控制操作

*shmctl()* 系统调用对由 *shmid* 标识的共享内存段执行一系列控制操作。

```
#include <sys/types.h>        /* For portability */
#include <sys/shm.h>

int `shmctl`(int *shmid*, int *cmd*, struct shmid_ds **buf*);
```

### 注意

成功时返回 0，出错时返回 -1。

*cmd* 参数指定要执行的控制操作。`IPC_STAT` 和 `IPC_SET` 操作（见下文）需要 *buf* 参数，对于其他操作，则应将其指定为 `NULL`。

在本节的其余部分，我们将描述可以为 *cmd* 指定的各种操作。

#### 通用控制操作

以下操作与其他类型的 System V IPC 对象相同。有关这些操作的详细信息，包括调用进程所需的权限和特权，请参见第 45.3 节。

`IPC_RMID`

标记共享内存段及其关联的 *shmid_ds* 数据结构以进行删除。如果当前没有进程附加到该段，删除将立即执行；否则，该段将在所有进程都已从其分离后被移除（即，当 *shmid_ds* 数据结构中的 *shm_nattch* 字段值降为 0 时）。在某些应用中，我们可以确保通过在所有进程使用 *shmat()* 将其附加到虚拟地址空间后立即标记为删除，从而确保共享内存段在应用程序终止时被整洁地清除。这类似于在打开文件后立即取消链接。

### 注意

在 Linux 中，如果共享内存段已通过 `IPC_RMID` 标记为删除，但由于某些进程仍然附加着它，因此尚未被移除，那么另一个进程仍然可以附加该段。然而，这种行为并不具备可移植性：大多数 UNIX 实现会阻止新的进程附加到已标记为删除的内存段。（SUSv3 对此场景下应该发生的行为没有说明。）一些 Linux 应用程序已经依赖于这种行为，这就是 Linux 没有进行修改以匹配其他 UNIX 实现的原因。

`IPC_STAT`

将与此共享内存段关联的 *shmid_ds* 数据结构的副本放置在 *buf* 指向的缓冲区中。（我们在第 48.8 节中描述了这个数据结构。）

`IPC_SET`

使用 *buf* 指向的缓冲区中的值更新与此共享内存段关联的 *shmid_ds* 数据结构中的选定字段。

#### 锁定和解锁共享内存

共享内存段可以被锁定在 RAM 中，这样它就永远不会被交换出去。这样可以提高性能，因为一旦每一页内存段被载入内存，应用程序就能保证在访问该页时不会因为页面错误而被延迟。有两个 *shmctl()* 锁定操作：

+   `SHM_LOCK` 操作将共享内存段锁定到内存中。

+   `SHM_UNLOCK` 操作解锁共享内存段，允许它被交换出去。

这些操作没有在 SUSv3 中规定，并且并非所有 UNIX 实现都提供这些功能。

在 2.6.10 版本之前的 Linux 中，只有具有特权的（`CAP_IPC_LOCK`）进程才能将共享内存段锁定到内存中。从 Linux 2.6.10 起，如果进程的有效用户 ID 与段的所有者或创建者的用户 ID 匹配，并且（在 `SHM_LOCK` 的情况下）进程的 `RLIMIT_MEMLOCK` 资源限制足够高，则非特权进程也可以锁定和解锁共享内存段。有关详细信息，请参见 内存锁定：*mlock()* 和 *mlockall()* 和 mlockall()")。

锁定共享内存段并不保证在*shmctl()*调用完成时，内存段的所有页面都会驻留在内存中。实际上，非驻留页面仅在通过附加共享内存段的进程随后引用时，才会逐一被锁定到内存中。一旦页面被加载到内存中，它们会保持驻留状态，直到随后被解锁，即使所有进程都已从地址空间中分离该段。（换句话说，`SHM_LOCK`操作设置的是共享内存段的属性，而不是调用进程的属性。）

### 注意

所谓的*加载到内存*是指当进程引用非驻留页面时，会发生页面错误。在此时，如果该页面在交换区中，则会重新加载到内存中。如果该页面是第一次被引用，则在交换文件中不存在相应的页面。因此，内核会分配一个新的物理内存页面，并调整进程的页面表和共享内存段的记账数据结构。

另一种内存锁定方法，语义略有不同，是使用*mlock()*，我们将在第 50.2 节中描述。

## 共享内存相关数据结构

每个共享内存段都有一个关联的*shmid_ds*数据结构，格式如下：

```
struct shmid_ds {
    struct ipc_perm shm_perm;   /* Ownership and permissions */
    size_t   shm_segsz;         /* Size of segment in bytes */
    time_t   shm_atime;         /* Time of last shmat() */
    time_t   shm_dtime;         /* Time of last shmdt() */
    time_t   shm_ctime;         /* Time of last change */
    pid_t    shm_cpid;          /* PID of creator */
    pid_t    shm_lpid;          /* PID of last shmat() / shmdt() */
    shmatt_t shm_nattch;        /* Number of currently attached processes */
};
```

SUSv3 要求显示所有这些字段。其他一些 UNIX 实现则在*shmid_ds*结构中包含额外的非标准字段。

*shmid_ds*结构的字段会被各种共享内存系统调用隐式更新，并且*shm_perm*字段的某些子字段可以通过*shmctl()* `IPC_SET`操作显式更新。详细信息如下：

*shm_perm*

当创建共享内存段时，该子结构的字段会按照第 45.3 节所述进行初始化。*uid*、*gid*以及（*mode*字段的低 9 位）可以通过`IPC_SET`进行更新。除了常见的权限位外，*shm_perm.mode*字段还包含两个只读位掩码标志。第一个标志，`SHM_DEST`（销毁），指示该段是否在所有进程从其地址空间分离后标记为删除（通过*shmctl()* `IPC_RMID`操作）。另一个标志，`SHM_LOCKED`，指示该段是否已被锁定到物理内存中（通过*shmctl()* `SHM_LOCK`操作）。这两个标志在 SUSv3 中没有标准化，且仅在少数其他 UNIX 实现中出现，在某些情况下名称不同。

*shm_segsz*

在创建共享内存段时，此字段被设置为请求的段大小（以字节为单位，即*shmget()*调用中指定的*size*参数的值）。正如在创建或打开共享内存段中所述，共享内存是以页面为单位分配的，因此段的实际大小可能大于此值。

*shm_atime*

该字段在共享内存段创建时设置为 0，每当进程附加该内存段时（*shmat()*），它会设置为当前时间。该字段以及 *shmid_ds* 结构中的其他时间戳字段被定义为 *time_t* 类型，并以自纪元以来的秒数表示时间。

*shm_dtime*

该字段在共享内存段创建时设置为 0，每当一个进程分离该内存段时（*shmdt()*），它会设置为当前时间。

*shm_ctime*

该字段在段创建时设置为当前时间，并在每次成功的 `IPC_SET` 操作时更新。

*shm_cpid*

该字段设置为创建该段的进程的进程 ID，该进程通过 *shmget()* 创建内存段。

*shm_lpid*

该字段在共享内存段创建时设置为 0，然后在每次成功调用 *shmat()* 或 *shmdt()* 时，设置为调用进程的进程 ID。

*shm_nattch*

该字段记录当前有多少进程附加了该内存段。当段创建时初始化为 0，每当成功调用 *shmat()* 时增加，每当成功调用 *shmdt()* 时减少。用于定义此字段的 *shmatt_t* 数据类型是一个无符号整数类型，SUSv3 要求其至少与 *unsigned short* 类型一样大。（在 Linux 上，这个类型被定义为 *unsigned long*。）

## 共享内存限制

大多数 UNIX 实现对 System V 共享内存施加了各种限制。以下是 Linux 共享内存限制的列表。限制所涉及的系统调用和达到限制时产生的错误会在括号中注明。

`SHMMNI`

这是一个系统范围的限制，用于限制可以创建的共享内存标识符的数量（换句话说，就是共享内存段的数量）。(*shmget()*, `ENOSPC`)

`SHMMIN`

这是共享内存段的最小大小（以字节为单位）。此限制的定义值为 1（不可更改）。然而，实际的有效限制是系统页面大小。(*shmget()*, `EINVAL`)

`SHMMAX`

这是共享内存段的最大大小（以字节为单位）。`SHMMAX` 的实际上限取决于可用的内存和交换空间。(*shmget()*, `EINVAL`)

`SHMALL`

这是一个系统范围的限制，用于限制共享内存的总页面数。大多数其他 UNIX 实现不提供这个限制。`SHMALL` 的实际上限取决于可用的内存和交换空间。(*shmget()*, `ENOSPC`)

其他一些 UNIX 实现也施加了以下限制（在 Linux 上未实现）：

`SHMSEG`

这是每个进程的限制，用于限制每个进程可以附加的共享内存段的数量。

在系统启动时，共享内存的限制设置为默认值。（这些默认值可能会随内核版本的不同而有所变化，一些发行版的内核可能设置了与标准内核不同的默认值。）在 Linux 中，某些限制可以通过`/proc`文件系统中的文件查看或更改。表 48-2 列出了与每个限制对应的`/proc`文件。以下是我们在一台 x86-32 系统上使用 Linux 2.6.31 时看到的默认限制：

```
$ `cd /proc/sys/kernel`
$ `cat shmmni`
4096
$ `cat shmmax`
33554432
$ `cat shmall`
2097152
```

Linux 特有的*shmctl()* `IPC_INFO`操作检索一个类型为*shminfo*的结构，包含各个共享内存限制的值：

```
struct shminfo buf;

shmctl(0, IPC_INFO, (struct shmid_ds *) &buf);
```

一个与 Linux 特定相关的操作，`SHM_INFO`，检索一个类型为*shm_info*的结构，包含关于共享内存对象实际使用资源的信息。`SHM_INFO`的使用示例如在本书的源代码分发包中的`svshm/svshm_info.c`文件中提供。

关于`IPC_INFO`、`SHM_INFO`、以及*shminfo*和*shm_info*结构的详细信息，可以在*shmctl(2)*手册页中找到。

表 48-2. 系统 V 共享内存限制

| 限制 | 上限值（x86-32） | `/proc/sys/kernel`中的对应文件 |
| --- | --- | --- |
| `SHMMNI` | `32768` (`IPCMNI`) | `shmmni` |
| `SHMMAX` | 取决于可用内存 | `shmmax` |
| `SHMALL` | 取决于可用内存 | `shmall` |

## 总结

共享内存允许两个或多个进程共享相同的内存页面。在通过共享内存交换数据时，不需要内核的干预。一旦一个进程将数据复制到共享内存段中，其他进程可以立即看到这些数据。共享内存提供了快速的进程间通信（IPC），尽管这种速度优势在某种程度上被必须使用某种同步技术（例如 System V 信号量）来同步对共享内存访问的需求所抵消。

附加共享内存段时，推荐的方法是允许内核选择该段在进程虚拟地址空间中附加的地址。这意味着该段可能在不同的进程中位于不同的虚拟地址。因此，任何对该段内地址的引用都应保持为相对偏移量，而不是绝对指针。

#### 进一步的信息

Linux 的内存管理方案以及共享内存实现的一些细节描述可以参考[Bovet & Cesati, 2005]。

## 练习

1.  用事件标志替换示例 48-2（`svshm_xfr_writer.c`）和示例 48-3（`svshm_xfr_reader.c`）中的二进制信号量使用（练习 47-5）。

1.  解释为什么如果将`for`循环修改如下，示例 48-3 中的程序会错误地报告传输的字节数：

    ```
    for (xfrs = 0, bytes = 0; shmp->cnt != 0; xfrs++, bytes += shmp->cnt) {
        reserveSem(semid, READ_SEM);            /* Wait for our turn */

        if (write(STDOUT_FILENO, shmp->buf, shmp->cnt) != shmp->cnt)
            fatal("write");

        releaseSem(semid, WRITE_SEM);           /* Give writer a turn */
    }
    ```

1.  尝试编译示例 48-2（`svshm_xfr_writer.c`）和示例 48-3（`svshm_xfr_reader.c`）中的程序，并使用不同的缓冲区大小（由常量`BUF_SIZE`定义）来交换数据。对每个缓冲区大小进行`svshm_xfr_reader.c`的执行时间测量。

1.  编写一个程序，显示与共享内存段关联的*shmid_ds*数据结构（共享内存关联数据结构）。该段的标识符应作为命令行参数指定。（参见示例 47-3，在监视信号量集中，它执行类似的任务用于 System V 信号量。）

1.  编写一个目录服务，使用共享内存段发布名称-值对。你需要提供一个 API，允许调用者创建新名称、修改现有名称、删除现有名称以及检索与名称关联的值。使用信号量确保执行更新共享内存段的进程对该段有独占访问权限。

1.  编写一个程序（类似于示例 46-6，参见使用消息队列的客户端-服务器编程），该程序使用*shmctl()* `SHM_INFO`和`SHM_STAT`操作来获取并显示系统中所有共享内存段的列表。
