## 第五十四章. POSIX 共享内存

在前面的章节中，我们探讨了两种允许不相关进程共享内存区域以进行进程间通信（IPC）的方法：System V 共享内存（第四十八章）和共享文件映射（共享文件映射）。这两种方法都有潜在的缺点：

+   System V 共享内存模型使用键值和标识符，这与标准 UNIX I/O 模型使用文件名和描述符不同。这种差异意味着我们需要一整套新的系统调用和命令来处理 System V 共享内存段。

+   使用共享文件映射进行 IPC 需要创建一个磁盘文件，即使我们不打算为共享区域提供持久的存储支持。除了需要创建文件的不便外，这种方法还会带来一定的文件 I/O 开销。

由于这些缺点，POSIX.1b 定义了一个新的共享内存 API：POSIX 共享内存，这也是本章的主题。

### 注意

POSIX 讨论共享内存 *对象*，而 System V 讨论共享内存 *段*。这些术语的差异是历史性的——这两个术语都用来指代进程间共享的内存区域。

## 概述

POSIX 共享内存允许我们在不需要创建相应映射文件的情况下，在不相关的进程之间共享映射区域。POSIX 共享内存在 Linux 2.4 内核及以后版本得到支持。

SUSv3 并未指定如何实现 POSIX 共享内存的具体细节。特别是，并没有要求使用（真实或虚拟的）文件系统来标识共享内存对象，尽管许多 UNIX 实现确实为此目的使用了文件系统。一些 UNIX 实现将共享内存对象的名称创建为标准文件系统中特殊位置的文件。Linux 使用专门的 *tmpfs* 文件系统（虚拟内存文件系统：*tmpfs*），该文件系统挂载在目录 `/dev/shm` 下。该文件系统具有内核持久性——其中包含的共享内存对象即使没有进程当前打开，也会保持存在，但如果系统关机，它们会丢失。

### 注意

系统上所有 POSIX 共享内存区域的总内存量受到底层 *tmpfs* 文件系统大小的限制。这个文件系统通常在启动时以某个默认大小（例如，256 MB）挂载。如果需要，超级用户可以通过使用命令 *mount -o remount,size=<num-bytes>* 重新挂载文件系统来更改其大小。

要使用 POSIX 共享内存对象，我们需要执行两个步骤：

1.  使用*shm_open()*函数以指定名称打开一个对象。（我们在第 51.1 节中描述了 POSIX 共享内存对象命名的规则。）*shm_open()*函数类似于*open()*系统调用。它可以创建一个新的共享内存对象，或者打开一个现有的对象。作为其函数结果，*shm_open()*返回一个文件描述符，指向该对象。

1.  将在前一步中获得的文件描述符传递给调用*mmap()*，并在*flags*参数中指定`MAP_SHARED`。这会将共享内存对象映射到进程的虚拟地址空间。与其他使用*mmap()*的情况一样，一旦我们映射了该对象，就可以关闭文件描述符而不影响映射。然而，我们可能需要保持文件描述符打开，以便在后续调用*fstat()*和*ftruncate()*时使用（参见创建共享内存对象）。

### 注意

对于 POSIX 共享内存，*shm_open()*与*mmap()*的关系类似于 System V 共享内存中*shmget()*与*shmat()*的关系。使用 POSIX 共享内存对象的两步过程（*shm_open()*加*mmap()*）的起源是历史性的。当 POSIX 委员会添加这一功能时，*mmap()*调用已经存在（[Stevens, 1999]）。实际上，我们所做的只是用*shm_open()*替换对*open()*的调用，不同之处在于使用*shm_open()*不需要在基于磁盘的文件系统中创建文件。

由于共享内存对象是通过文件描述符引用的，我们可以有效地利用 UNIX 系统中已经定义的各种文件描述符系统调用（例如，*ftruncate()*），而无需新的专用系统调用（这对于 System V 共享内存是必需的）。

## 创建共享内存对象

*shm_open()*函数创建并打开一个新的共享内存对象，或者打开一个现有的对象。*shm_open()*的参数类似于*open()*的参数。

```
#include <fcntl.h>            /* Defines O_* constants */
#include <sys/stat.h>         /* Defines mode constants */
#include <sys/mman.h>

int `shm_open`(const char **name*, int *oflag*, mode_t *mode*);
```

### 注意

成功时返回文件描述符，出错时返回-1

*name*参数标识要创建或打开的共享内存对象。*oflag*参数是一个位掩码，用于修改调用的行为。可以包括在此掩码中的值总结在表 54-1 oflag 参数的位值")中。

表 54-1. *shm_open()* oflag 参数的位值

| 标志 | 描述 |
| --- | --- |
| `O_CREAT` | 如果对象尚不存在，则创建对象 |
| `O_EXCL` | 与`O_CREAT`一起使用，独占创建对象 |
| `O_RDONLY` | 以只读方式打开 |
| `O_RDWR` | 以读写方式打开 |
| `O_TRUNC` | 将对象截断为零长度 |

*oflag* 参数的一个目的，是确定我们是打开一个已存在的共享内存对象，还是创建并打开一个新对象。如果 *oflag* 不包含 `O_CREAT`，则我们是在打开一个现有的对象。如果指定了 `O_CREAT`，则如果对象不存在，则会创建该对象。将 `O_EXCL` 与 `O_CREAT` 一起指定，表示请求确保调用者是该对象的创建者；如果对象已经存在，将会发生错误（`EEXIST`）。

*oflag* 参数还通过指定 `O_RDONLY` 或 `O_RDWR` 中的一个值，指示调用进程将如何访问共享内存对象。

剩余的标志值 `O_TRUNC` 会使成功打开一个现有的共享内存对象时，将该对象的长度截断为零。

### 注意

在 Linux 上，即使是只读打开，截断操作也会发生。然而，SUSv3 指出，在只读打开时使用 `O_TRUNC` 的结果是未定义的，因此我们无法在此情况下依赖特定的行为。

当创建一个新的共享内存对象时，它的所有权和组所有权来自调用 *shm_open()* 的进程的有效用户和组 ID，并且对象的权限根据 *mode* 位掩码参数中提供的值来设置。*mode* 的位值与文件的位值相同（表 15-4，在常规文件权限中）。与 *open()* 系统调用一样，*mode* 中的权限掩码会根据进程的 umask 进行屏蔽（进程文件模式创建掩码：*umask()*")）。与 *open()* 不同，调用 *shm_open()* 时始终需要 *mode* 参数；如果我们不是创建新对象，则该参数应指定为 0。

关闭时执行标志（`FD_CLOEXEC`，文件描述符和 *exec()*")）会在 *shm_open()* 返回的文件描述符上设置，因此当进程执行 *exec()* 时，文件描述符会被自动关闭。（这与映射在执行 *exec()* 时被解除映射是一致的。）

当创建一个新的共享内存对象时，它最初的长度为零。这意味着，在创建新的共享内存对象后，我们通常会调用 *ftruncate()* （截断文件：*truncate()* 和 *ftruncate()* 和 ftruncate()")）来设置对象的大小，然后再调用 *mmap()*。在调用 *mmap()* 后，我们还可以使用 *ftruncate()* 来扩展或缩小共享内存对象，具体操作需要参考边界情况中讨论的内容。

当共享内存对象被扩展时，新增加的字节会自动初始化为 0。

在任何时候，我们都可以对由*shm_open()*返回的文件描述符应用*fstat()*（检索文件信息：*stat()*")），以获取一个*stat*结构，其中的字段包含共享内存对象的信息，包括其大小*(st_size)*、权限*(st_mode)*、所有者*(st_uid)*和组*(st_gid)*。（这些是 SUSv3 要求*fstat()*在*stat*结构中设置的唯一字段，尽管 Linux 还会在时间字段中返回有意义的信息，以及在其余字段中返回一些其他不太有用的信息。）

可以分别使用*fchmod()*和*fchown()*来更改共享内存对象的权限和所有权。

#### 示例程序

示例 54-1 提供了一个简单的示例，展示了如何使用*shm_open()*、*ftruncate()*和*mmap()*。该程序创建一个大小由命令行参数指定的共享内存对象，并将该对象映射到进程的虚拟地址空间中。（映射步骤是多余的，因为我们实际上并未对共享内存执行任何操作，但它有助于演示如何使用*mmap()*。）该程序允许使用命令行选项选择*shm_open()*调用的标志（`O_CREAT`和`O_EXCL`）。

在以下示例中，我们使用该程序创建一个 10,000 字节的共享内存对象，然后使用*ls*命令显示该对象在`/dev/shm`中的内容：

```
$ `./pshm_create -c /demo_shm 10000`
$ `ls -l /dev/shm`
total 0
-rw-------    1 mtk      users       10000 Jun 20 11:31 demo_shm
```

示例 54-1。创建 POSIX 共享内存对象

```
`pshm/pshm_create.c`
#include <sys/stat.h>
#include <fcntl.h>
#include <sys/mman.h>
#include "tlpi_hdr.h"

static void
usageError(const char *progName)
{
    fprintf(stderr, "Usage: %s [-cx] name size [octal-perms]\n", progName);
    fprintf(stderr, "    -c   Create shared memory (O_CREAT)\n");
    fprintf(stderr, "    -x   Create exclusively (O_EXCL)\n");
    exit(EXIT_FAILURE);
}

int
main(int argc, char *argv[])
{
    int flags, opt, fd;
    mode_t perms;
    size_t size;
    void *addr;

    flags = O_RDWR;
    while ((opt = getopt(argc, argv, "cx")) != -1) {
        switch (opt) {
        case 'c':   flags |= O_CREAT;           break;
        case 'x':   flags |= O_EXCL;            break;
        default:    usageError(argv[0]);
        }
    }

    if (optind + 1 >= argc)
        usageError(argv[0]);

    size = getLong(argv[optind + 1], GN_ANY_BASE, "size");
    perms = (argc <= optind + 2) ? (S_IRUSR | S_IWUSR) :
                getLong(argv[optind + 2], GN_BASE_8, "octal-perms");

    /* Create shared memory object and set its size */

    fd = shm_open(argv[optind], flags, perms);
    if (fd == -1)
        errExit("shm_open");

    if (ftruncate(fd, size) == -1)
        errExit("ftruncate");

    /* Map shared memory object */

    addr = mmap(NULL, size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    if (addr == MAP_FAILED)
        errExit("mmap");

    exit(EXIT_SUCCESS);
}
    `pshm/pshm_create.c`
```

## 使用共享内存对象

示例 54-2 和示例 54-3 演示了如何使用共享内存对象将数据从一个进程传输到另一个进程。示例 54-2 中的程序将第二个命令行参数中包含的字符串复制到第一个命令行参数指定的现有共享内存对象中。在映射该对象并执行复制操作之前，程序使用*ftruncate()*将共享内存对象的大小调整为与要复制的字符串长度相同。

示例 54-2。将数据复制到 POSIX 共享内存对象

```
`pshm/pshm_write.c`
#include <fcntl.h>
#include <sys/mman.h>
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    int fd;
    size_t len;                 /* Size of shared memory object */
    char *addr;

    if (argc != 3 || strcmp(argv[1], "--help") == 0)
        usageErr("%s shm-name string\n", argv[0]);

    fd = shm_open(argv[1], O_RDWR, 0);      /* Open existing object */
    if (fd == -1)
        errExit("shm_open");

    len = strlen(argv[2]);
    if (ftruncate(fd, len) == -1)           /* Resize object to hold string */
        errExit("ftruncate");
    printf("Resized to %ld bytes\n", (long) len);

    addr = mmap(NULL, len, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    if (addr == MAP_FAILED)
        errExit("mmap");

    if (close(fd) == -1)
        errExit("close");                   /* 'fd' is no longer needed */

    printf("copying %ld bytes\n", (long) len);
    memcpy(addr, argv[2], len);             /* Copy string to shared memory */
    exit(EXIT_SUCCESS);
}
    `pshm/pshm_write.c`
```

示例 54-3 中的程序会在标准输出上显示其命令行参数中指定的现有共享内存对象中的字符串。在调用*shm_open()*之后，程序使用*fstat()*来确定共享内存的大小，并使用该大小在调用*mmap()*时映射对象，并在*write()*调用中打印字符串。

示例 54-3. 从 POSIX 共享内存对象中复制数据

```
`pshm/pshm_read.c`
#include <fcntl.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    int fd;
    char *addr;
    struct stat sb;

    if (argc != 2 || strcmp(argv[1], "--help") == 0)
        usageErr("%s shm-name\n", argv[0]);

    fd = shm_open(argv[1], O_RDONLY, 0);    /* Open existing object */
    if (fd == -1)
        errExit("shm_open");

    /* Use shared memory object size as length argument for mmap()
       and as number of bytes to write() */

    if (fstat(fd, &sb) == -1)
        errExit("fstat");

    addr = mmap(NULL, sb.st_size, PROT_READ, MAP_SHARED, fd, 0);
    if (addr == MAP_FAILED)
        errExit("mmap");

    if (close(fd) == -1);                   /* 'fd' is no longer needed */
        errExit("close");

    write(STDOUT_FILENO, addr, sb.st_size);
    printf("\n");
    exit(EXIT_SUCCESS);
}
    `pshm/pshm_read.c`
```

以下 shell 会话演示了示例 54-2 和示例 54-3 中的程序的使用。我们首先使用示例 54-1 中的程序创建一个零长度的共享内存对象。

```
$ `./pshm_create -c /demo_shm 0`
$ `ls -l /dev/shm`                        *Check the size of object*
total 4
-rw-------    1 mtk    users    0 Jun 21 13:33 demo_shm
```

然后我们使用示例 54-2 中的程序将字符串复制到共享内存对象中：

```
$ `./pshm_write /demo_shm 'hello'`
$ `ls -l /dev/shm`                        *Check that object has changed in size*
total 4
-rw-------    1 mtk    users    5 Jun 21 13:33 demo_shm
```

从输出中我们可以看到，程序调整了共享内存对象的大小，使其足够大以容纳指定的字符串。

最后，我们使用示例 54-3 中的程序显示共享内存对象中的字符串：

```
$ `./pshm_read /demo_shm`
hello
```

应用程序通常需要使用某种同步技术来允许进程协调对共享内存的访问。在这里显示的示例 shell 会话中，协调是由用户一个接一个地运行程序提供的。通常，应用程序会使用同步原语（例如信号量）来协调对共享内存对象的访问。

## 移除共享内存对象

SUSv3 要求 POSIX 共享内存对象至少具有内核持久性；也就是说，它们会一直存在，直到被显式删除或系统重启。当共享内存对象不再需要时，应使用*shm_unlink()*进行删除。

```
#include <sys/mman.h>

int `shm_unlink`(const char **name*);
```

### 注意

成功时返回 0，出错时返回-1

*shm_unlink()*函数移除由*name*指定的共享内存对象。删除共享内存对象不会影响对象的现有映射（映射会继续有效，直到相应的进程调用*munmap()*或终止），但会防止进一步的*shm_open()*调用打开该对象。一旦所有进程都取消映射该对象，该对象就会被删除，并且其内容会丢失。

示例 54-4 解除链接一个 POSIX 共享内存对象")中的程序使用*shm_unlink()*来移除程序命令行参数中指定的共享内存对象。

示例 54-4. 使用*shm_unlink()*解除链接一个 POSIX 共享内存对象

```
`pshm/pshm_unlink.c`
#include <fcntl.h>
#include <sys/mman.h>
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    if (argc != 2 || strcmp(argv[1], "--help") == 0)
        usageErr("%s shm-name\n", argv[0]);
    if (shm_unlink(argv[1]) == -1)
        errExit("shm_unlink");
    exit(EXIT_SUCCESS);
}
    `pshm/pshm_unlink.c`
```

## 共享内存 API 之间的比较

到目前为止，我们已经考虑了多种不同的技术，用于在无关进程之间共享内存区域：

+   System V 共享内存（第四十八章）；

+   共享文件映射（共享文件映射）；以及

+   POSIX 共享内存对象（本章的主题）。

### 注意

本节中提到的许多观点同样适用于共享匿名映射（匿名映射），这些映射用于在通过*fork()*相关的进程之间共享内存。

有许多点适用于所有这些技术：

+   它们提供快速的进程间通信（IPC），而且应用程序通常必须使用信号量（或其他同步原语）来同步对共享区域的访问。

+   一旦共享内存区域被映射到进程的虚拟地址空间中，它就像进程内存空间中的任何其他部分一样。

+   系统以类似的方式将共享内存区域放置在进程的虚拟地址空间中。在第 48.5 节中描述 System V 共享内存时我们概述了这种放置方式。Linux 特定的`/proc/`*PID*`/maps`文件列出了所有类型共享内存区域的信息。

+   假设我们不尝试在固定地址映射共享内存区域，我们应该确保对该区域位置的所有引用都是以偏移量计算的（而不是指针），因为该区域可能在不同进程中位于不同的虚拟地址（在共享内存中存储指针）。

+   在第五十章中描述的对虚拟内存区域操作的函数可以应用于使用这些技术创建的共享内存区域。

这些共享内存技术之间也有一些显著的区别：

+   共享文件映射的内容与底层映射文件同步，这意味着存储在共享内存区域中的数据可以在系统重启后持续存在。

+   System V 和 POSIX 共享内存使用不同的机制来标识和引用共享内存对象。System V 使用自己的一套密钥和标识符，这与标准 UNIX I/O 模型不兼容，并且需要单独的系统调用（例如 *shmctl()*）和命令（*ipcs* 和 *ipcrm*）。相比之下，POSIX 共享内存使用名称和文件描述符，因此可以使用多种现有的 UNIX 系统调用（例如 *fstat()* 和 *fchmod()*）来检查和操作共享内存对象。

+   System V 共享内存段的大小在创建时是固定的（通过 *shmget()*）。相比之下，对于由文件或 POSIX 共享内存对象支持的映射，我们可以使用 *ftruncate()* 来调整底层对象的大小，然后通过 *munmap()* 和 *mmap()*（或 Linux 特有的 *mremap()*）重新创建映射。

+   历史上，System V 共享内存比 *mmap()* 和 POSIX 共享内存更为广泛，但现在大多数 UNIX 实现都提供了所有这些技术。

除了最后一个关于可移植性的点外，上述列出的差异是有利于共享文件映射和 POSIX 共享内存对象的优势。因此，在新的应用程序中，可能会选择这些接口而非 System V 共享内存。我们选择哪个，取决于是否需要持久化的后端存储。共享文件映射提供了这种存储；POSIX 共享内存对象则可以避免在不需要后端存储时使用磁盘文件的开销。

## 总结

POSIX 共享内存对象用于在不创建底层磁盘文件的情况下，在不相关进程之间共享内存区域。为此，我们用 *shm_open()* 替代通常在 *mmap()* 之前调用的 *open()*。*shm_open()* 调用会在基于内存的文件系统中创建一个文件，并且我们可以使用传统的文件描述符系统调用来对这个虚拟文件执行各种操作。特别是，必须使用 *ftruncate()* 来设置共享内存对象的大小，因为它最初的长度为零。

我们现在已经描述了三种在不相关进程之间共享内存区域的技术：System V 共享内存、共享文件映射和 POSIX 共享内存对象。这三种技术有一些相似之处，也存在一些重要的差异，除非涉及可移植性问题，否则这些差异更有利于共享文件映射和 POSIX 共享内存对象。

## 练习

1.  将 示例 48-2 (`svshm_xfr_writer.c`) 和 示例 48-3 (`svshm_xfr_reader.c`) 中的程序重写，使用 POSIX 共享内存对象替代 System V 共享内存。
