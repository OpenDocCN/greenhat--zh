## 第五十章 虚拟内存操作

本章讨论了执行操作的各种系统调用，这些操作作用于进程的虚拟地址空间：

+   *mprotect()* 系统调用更改虚拟内存区域的保护。

+   *mlock()* 和 *mlockall()* 系统调用将虚拟内存区域锁定到物理内存中，从而防止其被交换出去。

+   *mincore()* 系统调用允许进程确定虚拟内存区域中的页面是否驻留在物理内存中。

+   *madvise()* 系统调用允许进程向内核建议其未来使用虚拟内存区域的模式。

其中一些系统调用特别用于与共享内存区域结合使用（第四十八章、第四十九章 和 第五十四章），但它们也可以应用于任何进程的虚拟内存区域。

### 注意

本章所描述的技术实际上与进程间通信（IPC）无关，但我们将它们包括在本书的这一部分，因为它们有时与共享内存一起使用。

## 更改内存保护：*mprotect()*

*mprotect()* 系统调用更改从 *addr* 开始、持续 *length* 字节的虚拟内存页的保护。

```
#include <sys/mman.h>

int `mprotect`(void **addr*, size_t *length*, int *prot*);
```

### 注意

成功时返回 0，错误时返回 -1。

*addr* 中给定的值必须是系统页面大小的倍数（由 *sysconf(_SC_PAGESIZE)* 返回）。(SUSv3 指定 *addr* 必须按页面对齐。SUSv4 表示实现 *可以* 要求此参数按页面对齐。) 因为保护是针对整个页面设置的，所以 *length* 实际上会四舍五入到系统页面大小的下一个倍数。

*prot* 参数是一个位掩码，指定该内存区域的新保护。它必须指定为 `PROT_NONE` 或通过对 `PROT_READ`、`PROT_WRITE` 和 `PROT_EXEC` 之一或多个进行 OR 运算得到的组合。这些值的含义与 *mmap()* 中相同（见表 49-2，以及创建映射：*mmap()*")）。

如果进程尝试以违反内存保护的方式访问内存区域，内核会为该进程生成一个 `SIGSEGV` 信号。

*mprotect()* 的一个用途是改变通过 *mmap()* 调用时设置的映射内存区域的保护，如 示例 50-1 修改内存保护") 中所示。该程序创建了一个匿名映射，初始时所有访问都被拒绝（`PROT_NONE`）。然后程序将该区域的保护更改为可读加可写。在更改前后，程序使用 *system()* 函数执行一个 shell 命令，显示来自 `/proc/`*PID*`/maps` 文件中与映射区域对应的行，从而可以看到内存保护的变化。（我们本可以直接解析 `/proc/self/maps` 来获取映射信息，但我们使用 *system()* 调用是因为它能使程序更简短。）当我们运行这个程序时，输出如下：

```
$ `./t_mprotect`
Before mprotect()
b7cde000-b7dde000 ---s 00000000 00:04 18258    /dev/zero (deleted)
After mprotect()
b7cde000-b7dde000 rw-s 00000000 00:04 18258    /dev/zero (deleted)
```

从最后一行输出中，我们可以看到 *mprotect()* 已经将内存区域的权限更改为 `PROT_READ | PROT_WRITE`。（关于 shell 输出中 `/dev/zero` 后出现的 `(deleted)` 字符串的解释，请参见第 48.5 节。）

示例 50-1. 使用 *mprotect()* 修改内存保护

```
`vmem/t_mprotect.c`
#define _BSD_SOURCE         /* Get MAP_ANONYMOUS definition from <sys/mman.h> */
#include <sys/mman.h>
#include "tlpi_hdr.h"

#define LEN (1024 * 1024)

#define SHELL_FMT "cat /proc/%ld/maps | grep zero"
#define CMD_SIZE (sizeof(SHELL_FMT) + 20)
                            /* Allow extra space for integer string */

int
main(int argc, char *argv[])
{
    char cmd[CMD_SIZE];
    char *addr;

    /* Create an anonymous mapping with all access denied */

    addr = mmap(NULL, LEN, PROT_NONE, MAP_SHARED | MAP_ANONYMOUS, -1, 0);
    if (addr == MAP_FAILED)
        errExit("mmap");

    /* Display line from /proc/self/maps corresponding to mapping */

    printf("Before mprotect()\n");
    snprintf(cmd, CMD_SIZE, SHELL_FMT, (long) getpid());
    system(cmd);

    /* Change protection on memory to allow read and write access */

    if (mprotect(addr, LEN, PROT_READ | PROT_WRITE) == -1)
        errExit("mprotect");

    printf("After mprotect()\n");
    system(cmd);                /* Review protection via /proc/self/maps */

    exit(EXIT_SUCCESS);
}
      `vmem/t_mprotect.c`
```

## 内存锁定：*mlock()* 和 *mlockall()*

在某些应用程序中，锁定进程虚拟内存的部分或全部内容是很有用的，这样可以保证它始终驻留在物理内存中。这样做的一个原因是为了提高性能。对锁定页面的访问可以保证不会因为页面错误而被延迟。这对于必须确保快速响应时间的应用程序非常有用。

锁定内存的另一个原因是安全性。如果包含敏感数据的虚拟内存页从未被交换出去，那么该页的副本将永远不会写入磁盘。如果该页被写入磁盘，理论上它可以在以后直接从磁盘设备读取。（攻击者可以故意制造这种情况，通过运行一个消耗大量内存的程序，从而迫使其他进程的内存被交换到磁盘。）即使在进程终止后，也可以从交换空间中读取信息，因为内核并不保证将交换空间中的数据清零。（通常，只有具有特权的进程才有权从交换设备读取数据。）

### 注意

笔记本电脑的挂起模式，以及一些桌面系统，会将系统的 RAM 内容保存到磁盘中，无论是否有内存锁定。

本节中，我们将讨论用于锁定和解锁进程虚拟内存部分或全部内容的系统调用。然而，在这之前，我们首先看看一个控制内存锁定的资源限制。

#### `RLIMIT_MEMLOCK` 资源限制

在具体资源限制的详细信息中，我们简要介绍了 `RLIMIT_MEMLOCK` 限制，它定义了进程可以锁定到内存中的字节数。现在我们更详细地讨论这个限制。

在 2.6.9 之前的 Linux 内核中，只有具有特权的进程（`CAP_IPC_LOCK`）可以锁定内存，而 `RLIMIT_MEMLOCK` 软资源限制对特权进程可以锁定的字节数设置了上限。

从 Linux 2.6.9 开始，内存锁定模型的变化允许非特权进程锁定少量内存。这对需要将少量敏感信息放入锁定内存中的应用程序非常有用，以确保这些信息永远不会写入磁盘的交换空间；例如，*gpg* 就是这样处理密码短语的。由于这些变化：

+   对于特权进程，可以锁定的内存量没有限制（即，`RLIMIT_MEMLOCK` 被忽略）；以及

+   现在，普通进程也能锁定内存，直到由`RLIMIT_MEMLOCK`定义的软限制为止。

`RLIMIT_MEMLOCK` 的软限制和硬限制的默认值都是 8 页（即 x86-32 上的 32,768 字节）。

`RLIMIT_MEMLOCK` 限制影响：

+   *mlock()* 和 *mlockall()*；

+   `mmap()`函数的`MAP_LOCKED`标志用于在创建内存映射时锁定内存映射，如附加的*mmap()*标志 Flags")中所述；

+   *shmctl()* `SHM_LOCK` 操作，用于锁定 System V 共享内存段，如第 48.7 节所述。

由于虚拟内存是以页为单位进行管理的，因此内存锁定作用于整个页。在执行限制检查时，`RLIMIT_MEMLOCK` 限制会向下舍入到系统页大小的最接近倍数。

尽管这个资源限制只有一个（软）值，但实际上它定义了两个独立的限制：

+   对于 *mlock()*、*mlockall()* 和 *mmap()* `MAP_LOCKED` 操作，`RLIMIT_MEMLOCK` 定义了每个进程的限制，限制了进程可以锁定的虚拟地址空间的字节数。

+   对于 *shmctl()* `SHM_LOCK` 操作，`RLIMIT_MEMLOCK` 定义了一个每用户限制，限制了该进程的实际用户 ID 可以锁定的共享内存段的字节数。当进程执行 *shmctl()* `SHM_LOCK` 操作时，内核会检查该进程的实际用户 ID 已经锁定的 System V 共享内存的总字节数。如果要锁定的内存段大小不会使总量超过该进程的 `RLIMIT_MEMLOCK` 限制，操作就会成功。

`RLIMIT_MEMLOCK` 对于 System V 共享内存具有不同语义的原因是，尽管没有任何进程附加到共享内存段，它仍然可以继续存在。（只有在执行显式的 *shmctl()* `IPC_RMID` 操作后，且所有进程都已将其从地址空间中分离时，才会被移除。）

#### 锁定和解锁内存区域

进程可以使用 *mlock()* 和 *munlock()* 来锁定和解锁内存区域。

```
#include <sys/mman.h>

int `mlock`(void **addr*, size_t *length*);
int `munlock`(void **addr*, size_t *length*);
```

### 注意

成功时两者返回 0，错误时返回 -1

*mlock()* 系统调用会锁定从 *addr* 开始，长度为 *length* 字节的调用进程的虚拟地址范围的所有页面。与传递给其他几个内存相关系统调用的相应参数不同，*addr* 不需要按页面对齐：内核从 *addr* 以下的下一个页面边界开始锁定页面。然而，SUSv3 允许实现选择性地要求 *addr* 是系统页面大小的倍数，因此便携式应用程序应确保在调用 *mlock()* 和 *munlock()* 时满足此条件。

由于锁定是按整个页面单位进行的，因此被锁定区域的结束位置是大于 *length* 加 *addr* 的下一个页面边界。例如，在页面大小为 4096 字节的系统上，调用 *mlock(2000, 4000)* 将锁定从字节 0 到 8191。

### 注意

我们可以通过检查 Linux 特定的 `/proc/`*PID*`/status` 文件中的 `VmLck` 条目来找出进程当前锁定了多少内存。

在成功调用 *mlock()* 后，指定范围内的所有页面保证被锁定并驻留在物理内存中。如果没有足够的物理内存来锁定所有请求的页面，或者请求违反了 `RLIMIT_MEMLOCK` 软资源限制，*mlock()* 系统调用将失败。

我们在 示例 50-2 和 mincore()") 中展示了如何使用 *mlock()*。

*munlock()* 系统调用执行 *mlock()* 的相反操作，移除调用进程先前建立的内存锁。*addr* 和 *length* 参数的解释方式与 *mlock()* 相同。解锁一组页面并不能保证它们停止驻留在内存中：只有在其他进程的内存需求下，页面才会从 RAM 中移除。

除了显式使用 *munlock()*，在以下情况下内存锁定会自动移除：

+   在进程终止时；

+   如果锁定的页面通过 *munmap()* 被解除映射；

+   如果被锁定的页面通过 *mmap()* 的 `MAP_FIXED` 标志被覆盖。

#### 内存锁定的语义细节

在接下来的段落中，我们会说明一些内存锁定语义的细节。

内存锁定不会被 *fork()* 创建的子进程继承，并且不会跨 *exec()* 保留。

当多个进程共享一组页面（例如，`MAP_SHARED` 映射）时，只要至少有一个进程对这些页面保持内存锁定，这些页面就会保持在内存中。

内存锁定对于单个进程来说是不可嵌套的。如果一个进程在某个虚拟地址范围上重复调用 *mlock()*，则只会建立一个锁，并且这个锁会通过一次调用 *munlock()* 被移除。另一方面，如果我们使用 *mmap()* 在同一个进程中将相同的一组页面（即相同的文件）映射到多个不同的位置，并且锁定每个映射，那么这些页面将在所有映射被解锁之前一直保持在内存中。

内存锁定是按页面单位执行的，并且不能嵌套，这意味着独立地对同一虚拟页面上的不同数据结构应用 *mlock()* 和 *munlock()* 调用在逻辑上是不正确的。例如，假设我们在同一个虚拟内存页面中有两个数据结构，分别由指针 *p1* 和 *p2* 指向，并且我们执行以下调用：

```
mlock(*p1, len1);
mlock(*p2, len2);               /* Actually has no effect */
munlock(*p1, len1);
```

所有上述调用都会成功，但在这个序列的末尾，整个页面会被解锁；即，指向 *p2* 的数据结构不会被锁定在内存中。

请注意，*shmctl()* `SHM_LOCK` 操作（共享内存控制操作）的语义与 *mlock()* 和 *mlockall()* 不同，具体如下：

+   在执行 `SHM_LOCK` 操作后，页面只有在被后续访问调入内存时才会被锁定。相比之下，*mlock()* 和 *mlockall()* 会在调用返回之前将所有锁定的页面调入内存。

+   `SHM_LOCK` 操作设置的是共享内存段的属性，而不是进程的属性。（因此，`/proc/`*PID*`/status VmLck` 字段中的值不包括通过 `SHM_LOCK` 锁定的任何附加的 System V 共享内存段的大小。）这意味着，一旦页面被调入内存，即使所有进程都分离了共享内存段，这些页面仍然会保持在内存中。相比之下，使用 *mlock()*（或 *mlockall()*）将内存区域锁定后，只有在至少一个进程保持该区域的锁定时，该区域才会继续保持锁定状态。

#### 锁定和解锁一个进程的所有内存

一个进程可以使用 *mlockall()* 和 *munlockall()* 来锁定和解锁其所有内存。

```
#include <sys/mman.h>

int `mlockall`(int *flags*);
int `munlockall`(void);
```

### 注意

两者在成功时返回 0，在出错时返回 -1

*mlockall()* 系统调用会根据通过按位“或”操作组合的以下常量之一或两者，锁定进程虚拟地址空间中当前映射的所有页面、未来映射的所有页面，或者两者：

`MCL_CURRENT`

锁定当前映射到调用进程虚拟地址空间中的所有页面。这包括程序文本、数据段、内存映射和栈所分配的所有页面。在成功调用并指定 `MCL_CURRENT` 标志后，调用进程的所有页面都可以保证驻留在内存中。此标志不会影响后续分配的页面；对于此类页面，我们必须使用 `MCL_FUTURE`。

`MCL_FUTURE`

锁定所有随后映射到调用进程虚拟地址空间的页面。这些页面可以是通过 *mmap()* 或 *shmat()* 映射的共享内存区域的一部分，或者是向上增长的堆或向下增长的栈的一部分。由于指定了 `MCL_FUTURE` 标志，后续的内存分配操作（例如 *mmap()*、*sbrk()* 或 *malloc()*) 可能会失败，或者栈增长可能会导致 `SIGSEGV` 信号，如果系统内存不足以分配给进程，或者遇到 `RLIMIT_MEMLOCK` 的软资源限制。

使用 *mlock()* 创建的内存锁的约束、生命周期和继承规则，同样适用于通过 *mlockall()* 创建的内存锁。

*munlockall()* 系统调用解锁调用进程的所有页面，并撤销任何先前的 *mlockall(MCL_FUTURE)* 调用的效果。与 *munlock()* 一样，解锁的页面不能保证会被此调用从 RAM 中移除。

### 注意

在 Linux 2.6.9 之前，调用 *munlockall()* 需要特权 (`CAP_IPC_LOCK`)，但 *munlock()* 在某些情况下不需要特权。自 Linux 2.6.9 起，已不再需要特权。

## 确定内存驻留：*mincore()*

*mincore()* 系统调用是内存锁定系统调用的补充。它报告虚拟地址范围内哪些页面当前驻留在 RAM 中，因此如果访问这些页面不会导致页面错误。

SUSv3 没有规范 *mincore()*。它在许多 UNIX 实现中可用，但并不是所有实现都支持。在 Linux 上，*mincore()* 从 2.4 内核开始可用。

```
#define _BSD_SOURCE           /* Or: #define _SVID_SOURCE */
#include <sys/mman.h>

int `mincore`(void **addr*, size_t *length*, unsigned char **vec*);
```

### 注意

成功时返回 0，出错时返回 -1

*mincore()* 系统调用返回关于从 *addr* 开始、长度为 *length* 字节的虚拟地址范围内页面的内存驻留信息。提供的 *addr* 地址必须是页面对齐的，并且由于返回的是整页信息，*length* 实际上会向上舍入到系统页面大小的下一个倍数。

内存驻留信息通过 *vec* 返回，*vec* 必须是一个大小为 *(length + PAGE_SIZE – 1) / PAGE_SIZE* 字节的数组。（在 Linux 上，*vec* 的类型是 *unsigned char **；在其他一些 UNIX 实现中，*vec* 的类型是 *char **。）如果相应的页面驻留在内存中，则每个字节的最低有效位被设置。其他位的设置在某些 UNIX 实现中是未定义的，因此可移植的应用程序应该只测试这一位。

*mincore()*返回的信息可能在调用发生时与检查*vec*元素时之间有所变化。唯一保证始终驻留在内存中的页面是那些通过*mlock()*或*mlockall()*锁定的页面。

### 注意

在 Linux 2.6.21 之前，存在一些实现问题，导致*mincore()*未能正确报告`MAP_PRIVATE`映射或非线性映射（通过*remap_file_pages()*建立）的内存驻留信息。

示例 50-2 和 mincore()")演示了*mlock()*和*mincore()*的使用。在使用*mmap()*分配和映射内存区域之后，该程序使用*mlock()*来锁定整个区域或定期锁定页面组。（程序的每个命令行参数以页面为单位表达；程序将这些参数转换为字节，以便用于调用*mmap()*、*mlock()*和*mincore()*。）在调用*mlock()*之前和之后，程序使用*mincore()*来获取关于该区域页面内存驻留状态的信息，并将这些信息以图形方式显示。

示例 50-2. 使用*mlock()*和*mincore()*

```
`vmem/memlock.c`
#define _BSD_SOURCE     /* Get mincore() declaration and MAP_ANONYMOUS
                           definition from <sys/mman.h> */
#include <sys/mman.h>
#include "tlpi_hdr.h"

/* Display residency of pages in range [addr .. (addr + length - 1)] */

static void
displayMincore(char *addr, size_t length)
{
    unsigned char *vec;
    long pageSize, numPages, j;

    pageSize = sysconf(_SC_PAGESIZE);

    numPages = (length + pageSize - 1) / pageSize;
    vec = malloc(numPages);
    if (vec == NULL)
        errExit("malloc");

    if (mincore(addr, length, vec) == -1)
        errExit("mincore");

    for (j = 0; j < numPages; j++) {
        if (j % 64 == 0)
            printf("%s%10p: ", (j == 0) ? "" : "\n", addr + (j * pageSize));
        printf("%c", (vec[j] & 1) ? '*' : '.');
    }
    printf("\n");

    free(vec);
}

int
main(int argc, char *argv[])
{
    char *addr;
    size_t len, lockLen;
    long pageSize, stepSize, j;

    if (argc != 4 || strcmp(argv[1], "--help") == 0)
        usageErr("%s num-pages lock-page-step lock-page-len\n", argv[0]);

    pageSize = sysconf(_SC_PAGESIZE);
    if (pageSize == -1)
        errExit("sysconf(_SC_PAGESIZE)");

    len =      getInt(argv[1], GN_GT_0, "num-pages") * pageSize;
    stepSize = getInt(argv[2], GN_GT_0, "lock-page-step") * pageSize;
    lockLen =  getInt(argv[3], GN_GT_0, "lock-page-len") * pageSize;

    addr = mmap(NULL, len, PROT_READ, MAP_SHARED | MAP_ANONYMOUS, -1, 0);
    if (addr == MAP_FAILED)
        errExit("mmap");

    printf("Allocated %ld (%#lx) bytes starting at %p\n",
            (long) len, (unsigned long) len, addr);

    printf("Before mlock:\n");
    displayMincore(addr, len);

    /* Lock pages specified by command line arguments into memory */

    for (j = 0; j + lockLen <= len; j += stepSize)
        if (mlock(addr + j, lockLen) == -1)
            errExit("mlock");

    printf("After mlock:\n");
    displayMincore(addr, len);

    exit(EXIT_SUCCESS);
}
      `vmem/memlock.c`
```

以下 shell 会话展示了示例 50-2 和 mincore()")程序的示例运行。在这个示例中，我们分配了 32 个页面，并在每组 8 个页面中锁定 3 个连续的页面：

```
$ `su`                                        *Assume privilege*
Password:
# `./memlock 32 8 3`
Allocated 131072 (0x20000) bytes starting at 0x4014a000
Before mlock:
0x4014a000: ................................
After mlock:
0x4014a000: ***.....***.....***.....***.....
```

在程序输出中，点表示未驻留在内存中的页面，星号表示驻留在内存中的页面。从最后一行输出中可以看到，在每组 8 个页面中，有 3 个页面驻留在内存中。

在这个示例中，我们假设拥有超级用户权限，以便程序可以使用*mlock()*。但在 Linux 2.6.9 及以后的版本中，如果要锁定的内存量在`RLIMIT_MEMLOCK`软资源限制范围内，则不需要此权限。

## 建议未来的内存使用模式：*madvise()*

*madvise()*系统调用的作用是通过告知内核调用进程在以*addr*为起始位置、持续*length*字节范围内的页面可能使用情况，从而提高应用程序的性能。内核可以利用这些信息来提高在文件映射上的 I/O 效率，该文件映射支撑着这些页面。（关于文件映射的讨论，请参见文件映射。）在 Linux 中，*madvise()*自 2.4 版本内核起可用。

```
#define _BSD_SOURCE
#include <sys/mman.h>

int `madvise`(void **addr*, size_t *length*, int *advice*);
```

### 注意

成功时返回 0，出错时返回-1

在*addr*中指定的值必须是页面对齐的，并且*length*实际上会向上舍入到系统页面大小的下一个倍数。*advice*参数是以下选项之一：

`MADV_NORMAL`

这是默认行为。页面以集群的形式传输（系统页面大小的小倍数）。这会导致一些预读和延迟读取。

`MADV_RANDOM`

该区域内的页面将被随机访问，因此预读取没有任何好处。因此，内核应在每次读取时获取最小量的数据。

`MADV_SEQUENTIAL`

该范围内的页面将按顺序访问一次。因此，内核可以积极地预读取，并且页面在访问后可以迅速被释放。

`MADV_WILLNEED`

在该区域内预读取页面，为未来访问做准备。`MADV_WILLNEED` 操作的效果类似于 Linux 特有的 *readahead()* 系统调用和 *posix_fadvise()* 中的 `POSIX_FADV_WILLNEED` 操作。

`MADV_DONTNEED`

调用进程不再需要该区域中的页面驻留在内存中。此标志的具体效果在不同的 UNIX 实现中有所不同。我们首先注意到 Linux 上的行为。对于 `MAP_PRIVATE` 区域，映射的页面会被显式丢弃，这意味着对页面的修改会丢失。虚拟内存地址范围仍然可以访问，但每次访问页面时都会触发页面错误，重新初始化页面，要么用其映射的文件内容，要么在匿名映射的情况下用零填充。这可以作为显式重新初始化 `MAP_PRIVATE` 区域内容的一种方式。对于 `MAP_SHARED` 区域，内核在某些情况下*可能*会丢弃已修改的页面，具体取决于架构（在 x86 上不会发生这种行为）。其他一些 UNIX 实现也表现得与 Linux 相同。然而，在某些 UNIX 实现中，`MADV_DONTNEED` 只是通知内核，如果有必要，指定的页面可以被换出。可移植的应用程序不应依赖于 Linux 对 `MADV_DONTNEED` 的破坏性语义。

### 注意

Linux 2.6.16 增加了三个新的非标准*建议*值：`MADV_DONTFORK`、`MADV_DOFORK` 和 `MADV_REMOVE`。Linux 2.6.32 和 2.6.33 增加了另外四个非标准*建议*值：`MADV_HWPOISON`、`MADV_SOFT_OFFLINE`、`MADV_MERGEABLE` 和 `MADV_UNMERGEABLE`。这些值在特殊情况下使用，并在 *madvise(2)* 手册页中进行了描述。

大多数 UNIX 实现提供了一个 *madvise()* 的版本，通常至少允许上述描述的 *建议* 常量。然而，SUSv3 将这个 API 标准化为一个不同的名称 *posix_madvise()*，并将相应的 *建议* 常量的前缀加上 `POSIX_` 字符串。因此，这些常量为 `POSIX_MADV_NORMAL`、`POSIX_MADV_RANDOM`、`POSIX_MADV_SEQUENTIAL`、`POSIX_MADV_WILLNEED` 和 `POSIX_MADV_DONTNEED`。这个替代接口在 *glibc*（版本 2.2 及更高版本）中通过调用 *madvise()* 实现，但并非所有 UNIX 实现都支持。

### 注意

SUSv3 规定 *posix_madvise()* 不应影响程序的语义。然而，在 *glibc* 2.7 之前的版本中，`POSIX_MADV_DONTNEED` 操作是通过 *madvise()* `MADV_DONTNEED` 实现的，这会影响程序的语义，正如前文所述。从 *glibc* 2.7 开始，*posix_madvise()* 封装函数实现 `POSIX_MADV_DONTNEED` 为不做任何操作，因此不会影响程序的语义。

## 总结

在本章中，我们讨论了可以在进程的虚拟内存上执行的各种操作：

+   *mprotect()* 系统调用用于更改虚拟内存区域的保护状态。

+   *mlock()* 和 *mlockall()* 系统调用分别将进程的部分或全部虚拟地址空间锁定到物理内存中。

+   *mincore()* 系统调用报告虚拟内存区域中哪些页面当前驻留在物理内存中。

+   *madvise()* 系统调用和 *posix_madvise()* 函数允许一个进程向内核建议该进程预计的内存使用模式。

## 练习

1.  通过编写一个程序来验证 `RLIMIT_MEMLOCK` 资源限制的效果，该程序设置此限制的值并尝试锁定超出限制的更多内存。

1.  编写一个程序，验证 *madvise()* `MADV_DONTNEED` 操作在可写的 `MAP_PRIVATE` 映射中的运行效果。
