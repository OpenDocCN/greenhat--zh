## 第四章 文件 I/O：通用 I/O 模型

我们现在开始认真研究系统调用 API。文件是一个很好的起点，因为它们是 UNIX 哲学的核心。本章的重点是用于执行文件输入和输出的系统调用。

我们介绍了文件描述符的概念，接着探讨了构成所谓通用 I/O 模型的系统调用。这些是打开和关闭文件、读取和写入数据的系统调用。

我们重点讨论磁盘文件的 I/O。然而，这里讨论的许多内容对后续章节也很重要，因为相同的系统调用被用于执行各种类型文件的 I/O 操作，例如管道和终端。

第五章扩展了本章的讨论，提供了有关文件 I/O 的更多细节。文件 I/O 的另一个方面——缓冲——足够复杂，值得单独成章。第十三章讨论了内核和*stdio*库中的 I/O 缓冲。

## 概述

所有执行 I/O 的系统调用都是通过*文件描述符*来引用已打开的文件的，文件描述符是一个（通常较小的）非负整数。文件描述符用于引用所有类型的已打开文件，包括管道、FIFO、套接字、终端、设备和常规文件。每个进程都有自己的一组文件描述符。

按惯例，大多数程序期望能够使用表 4-1 中列出的三个标准文件描述符。这三个描述符由 shell 在程序启动之前代表程序打开。或者，更准确地说，程序继承了 shell 的文件描述符副本，并且 shell 通常总是保持这三个文件描述符处于打开状态。（在交互式 shell 中，这三个文件描述符通常指向 shell 正在运行的终端。）如果命令行中指定了 I/O 重定向，则 shell 确保在启动程序之前，文件描述符被适当修改。

表 4-1. 标准文件描述符

| 文件描述符 | 目的 | POSIX 名称 | *stdio*流 |
| --- | --- | --- | --- |
| 0 | 标准输入 | `STDIN_FILENO` | *stdin* |
| 1 | 标准输出 | `STDOUT_FILENO` | *stdout* |
| 2 | 标准错误 | `STDERR_FILENO` | *stderr* |

在程序中引用这些文件描述符时，我们可以使用数字（0、1 或 2），或者最好使用在`<unistd.h>`中定义的 POSIX 标准名称。

### 注意

尽管变量*stdin*、*stdout*和*stderr*最初指的是进程的标准输入、输出和错误，但可以通过使用*freopen()*库函数将它们更改为指向任何文件。作为操作的一部分，*freopen()*可能会改变重新打开的流所依赖的文件描述符。换句话说，在对*stdout*进行*freopen()*之后，例如，就不能再安全地假设底层文件描述符仍然是 1。

以下是执行文件 I/O 的四个关键系统调用（编程语言和软件包通常仅间接通过 I/O 库使用这些调用）：

+   *fd = open (pathname, flags, mode)* 打开由 *pathname* 标识的文件，并返回一个文件描述符，用于在后续调用中引用该打开的文件。如果文件不存在，*open()* 可能会根据 *flags* 参数的设置创建文件。*flags* 参数还指定文件是用于读取、写入还是两者兼有。*mode* 参数指定如果通过此调用创建文件时所赋予的权限。如果 *open()* 调用不是用于创建文件，则此参数会被忽略，可以省略。

+   *numread = read (fd, buffer, count)* 从由 *fd* 引用的打开文件中读取最多 *count* 字节，并将其存储在 *buffer* 中。*read()* 调用返回实际读取的字节数。如果无法读取更多字节（即遇到文件结束），*read()* 返回 0。

+   *numwritten = write (fd, buffer, count)* 从 *buffer* 中写入最多 *count* 字节到由 *fd* 引用的打开文件。*write()* 调用返回实际写入的字节数，这个数可能小于 *count*。

+   *status = close (fd)* 在所有 I/O 操作完成后被调用，用于释放文件描述符 *fd* 及其相关的内核资源。

在深入探讨这些系统调用的细节之前，我们先提供一个简短的演示，展示它们在 示例 4-1 中的用法。该程序是 *cp(1)* 命令的简化版。它将第一个命令行参数中指定的现有文件的内容复制到第二个命令行参数中指定的新文件中。

我们可以如下使用 示例 4-1 中的程序：

```
`$ ./copy oldfile newfile`
```

示例 4-1. 使用 I/O 系统调用

```
`fileio/copy.c`
#include <sys/stat.h>
#include <fcntl.h>
#include "tlpi_hdr.h"

#ifndef BUF_SIZE        /* Allow "cc -D" to override definition */
#define BUF_SIZE 1024
#endif

int
main(int argc, char *argv[])
{
    int inputFd, outputFd, openFlags;
    mode_t filePerms;
    ssize_t numRead;
    char buf[BUF_SIZE];

    if (argc != 3 || strcmp(argv[1], "--help") == 0)
        usageErr("%s old-file new-file\n", argv[0]);

    /* Open input and output files */

    inputFd = open(argv[1], O_RDONLY);
    if (inputFd == -1)
        errExit("opening file %s", argv[1]);

    openFlags = O_CREAT | O_WRONLY | O_TRUNC;
    filePerms = S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP |
                S_IROTH | S_IWOTH;      /* rw-rw-rw- */
    outputFd = open(argv[2], openFlags, filePerms);
    if (outputFd == -1)
        errExit("opening file %s", argv[2]);

    /* Transfer data until we encounter end of input or an error */

    while ((numRead = read(inputFd, buf, BUF_SIZE)) > 0)
        if (write(outputFd, buf, numRead) != numRead)
            fatal("couldn't write whole buffer");
    if (numRead == -1)
        errExit("read");

    if (close(inputFd) == -1)
        errExit("close input");
    if (close(outputFd) == -1)
        errExit("close output");

    exit(EXIT_SUCCESS);
}
      `fileio/copy.c`
```

## I/O 的普遍性

UNIX I/O 模型的一个显著特征是 *I/O 的普遍性* 概念。这意味着相同的四个系统调用——*open()*, *read()*, *write()*, 和 *close()*——用于对所有类型的文件进行 I/O 操作，包括终端等设备。因此，如果我们仅使用这些系统调用编写程序，那么该程序将能在任何类型的文件上运行。例如，以下是程序在 示例 4-1 中的有效用法：

```
`$ ./copy test test.old`           *Copy a regular file*
`$ ./copy a.txt /dev/tty`          *Copy a regular file to this terminal*
`$ ./copy /dev/tty b.txt`          *Copy input from this terminal to a regular file*
`$ ./copy /dev/pts/16 /dev/tty`    *Copy input from another terminal*
```

I/O 的通用性通过确保每个文件系统和设备驱动程序实现相同的 I/O 系统调用来实现。由于与文件系统或设备相关的细节在内核中处理，我们在编写应用程序时通常可以忽略设备特定的因素。当需要访问文件系统或设备的特定功能时，程序可以使用 *ioctl()* 系统调用 (超出通用 I/O 模型的操作: *ioctl()*"))，它提供了对不属于通用 I/O 模型的功能的接口。

## 打开文件：*open()*

*open()* 系统调用可以打开一个已存在的文件，也可以创建并打开一个新文件。

```
#include <sys/stat.h>
#include <fcntl.h>

int `open`(const char **pathname*, int *flags*, ... /* mode_t *mode* */);
```

### 注意

成功时返回文件描述符，出错时返回 -1

要打开的文件由 *pathname* 参数指定。如果 *pathname* 是符号链接，它将被解除引用。成功时，*open()* 返回一个文件描述符，用于在后续的系统调用中引用该文件。如果发生错误，*open()* 返回 -1，并相应地设置 *errno*。

*flags* 参数是一个位掩码，指定文件的 *访问模式*，使用 表 4-2 中显示的常量之一。

### 注意

早期的 UNIX 实现使用数字 0、1 和 2 来代替 表 4-2 中显示的名称。大多数现代 UNIX 实现定义这些常量，使其具有这些值。因此，我们可以看到，`O_RDWR` 并不等同于 `O_RDONLY | O_WRONLY`；后一种组合是逻辑错误。

当 *open()* 用于创建新文件时，*mode* 位掩码参数指定文件的权限。（用于指定 *mode* 的 *mode_t* 数据类型是 SUSv3 中指定的整数类型。）如果 *open()* 调用没有指定 `O_CREAT`，则 *mode* 可以省略。

表 4-2. 文件访问模式

| 访问模式 | 描述 |
| --- | --- |
| `O_RDONLY` | 仅打开文件以供读取 |
| `O_WRONLY` | 仅打开文件以供写入 |
| `O_RDWR` | 打开文件以进行读写操作 |

我们在第 15.4 节中详细描述了文件权限。稍后我们将看到，实际设置的新文件权限不仅取决于 *mode* 参数，还取决于进程的 umask（进程文件模式创建掩码：*umask()*")）和（可选存在的）父目录的默认访问控制列表（默认 ACL 和文件创建）。在此之前，我们仅需注意，*mode* 参数可以指定为数字（通常是八进制）或更优地，通过将一个或多个位掩码常量按位 OR (`|`) 来指定，这些常量列在表 15-4 中，见文件权限。

示例 4-2* 使用示例") 展示了 *open()* 的使用示例，其中一些使用了我们稍后会介绍的附加 *flags* 位。

示例 4-2. *open()* 使用示例

```
/* Open existing file for reading */

    fd = open("startup", O_RDONLY);
    if (fd == -1)
        errExit("open");

    /* Open new or existing file for reading and writing, truncating to zero
       bytes; file permissions read+write for owner, nothing for all others */

    fd = open("myfile", O_RDWR | O_CREAT | O_TRUNC, S_IRUSR | S_IWUSR);
    if (fd == -1)
        errExit("open");

    /* Open new or existing file for writing; writes should always
       append to end of file */

    fd = open("w.log", O_WRONLY | O_CREAT | O_TRUNC | O_APPEND,
                       S_IRUSR | S_IWUSR);
    if (fd == -1)
        errExit("open");
```

#### *open()* 返回的文件描述符号

SUSv3 规范规定，如果 *open()* 成功，它会保证使用进程中最小的未使用文件描述符。我们可以利用这一特性确保文件通过特定的文件描述符打开。例如，下面的序列确保文件使用标准输入（文件描述符 0）打开。

```
if (close(STDIN_FILENO) == -1)      /* Close file descriptor 0 */
    errExit("close");

fd = open(pathname, O_RDONLY);
if (fd == -1)
    errExit("open");
```

由于文件描述符 0 未使用，*open()* 保证使用该描述符打开文件。在复制文件描述符一节中，我们讨论了如何使用 *dup2()* 和 *fcntl()* 来实现类似的结果，但它们提供了对使用的文件描述符的更灵活控制。在这一节中，我们还展示了一个示例，说明为何控制文件打开时所用的文件描述符会是有用的。

### *open() flags* 参数

在示例 4-2* 使用示例")中展示的某些 *open()* 调用中，我们在 *flags* 中除了文件访问模式外，还包括了其他位（`O_CREAT`、`O_TRUNC` 和 `O_APPEND`）。现在我们更详细地考虑 *flags* 参数。表 4-3* 的 *flags* 参数值") 总结了可以按位 OR (`|`) 在 *flags* 中的所有常量。最后一列指示了哪些常量在 SUSv3 或 SUSv4 中被标准化。

表 4-3. *open()* 的 *flags* 参数值

| 标志 | 目的 | SUS? |
| --- | --- | --- |
| `O_RDONLY` | 仅打开进行读取 | v3 |
| `O_WRONLY` | 仅打开进行写入 | v3 |
| `O_RDWR` | 允许读写 | v3 |
| `O_CLOEXEC` | 设置执行时关闭标志（自 Linux 2.6.23 起） | v4 |
| `O_CREAT` | 如果文件不存在则创建文件 | v3 |
| `O_DIRECT` | 文件 I/O 绕过缓冲区缓存 |   |
| `O_DIRECTORY` | 如果*pathname*不是目录，则失败 | v4 |
| `O_EXCL` | 与*O_CREAT*一起使用：排他性创建文件 | v3 |
| `O_LARGEFILE` | 用于 32 位系统打开大文件 |   |
| `O_NOATIME` | 不更新*read()*的文件最后访问时间（自 Linux 2.6.8 起） |   |
| `O_NOCTTY` | 不允许*pathname*成为控制终端 | v3 |
| `O_NOFOLLOW` | 不跟踪符号链接 | v4 |
| `O_TRUNC` | 将现有文件截断为零长度 | v3 |
| `O_APPEND` | 写入总是附加到文件末尾 | v3 |
| `O_ASYNC` | 当 I/O 操作可用时生成信号 |   |
| `O_DSYNC` | 提供同步 I/O 数据完整性（自 Linux 2.6.33 起） | v3 |
| `O_NONBLOCK` | 以非阻塞模式打开 | v3 |
| `O_SYNC` | 使文件写入同步 | v3 |

表格 4-3")中的常量分为以下几组：

+   *文件访问模式标志*：这些是之前描述的`O_RDONLY`、`O_WRONLY`和`O_RDWR`标志。在*flags*中只能指定其中一个值。访问模式可以使用*fcntl()*的`F_GETFL`操作来获取（打开文件状态标志）。

+   *文件创建标志*：这些是表格 4-3")中第二部分所示的标志。它们控制*open()*调用的各种行为，以及后续 I/O 操作的选项。这些标志不能被检索或更改。

+   *打开文件状态标志*：这些是表格 4-3")中其余的标志。可以使用*fcntl()*的`F_GETFL`和`F_SETFL`操作来获取和修改它们（打开文件状态标志）。这些标志有时也称为*文件状态标志*。

    ### 注意

    自内核版本 2.6.22 起，可以通过读取目录`/proc/`*PID*`/fdinfo`中的 Linux 特定文件来获取系统上任何进程的文件描述符信息。该目录中每个文件都代表进程的一个打开的文件描述符，文件名与描述符的编号相匹配。该文件中的*pos*字段显示当前文件偏移量（更改文件偏移量：*lseek()*")）。*flags*字段是一个八进制数字，显示文件访问模式标志和打开文件状态标志。（要解码此数字，我们需要查看 C 库头文件中这些标志的数值。）

以下是*flags*常量的详细信息：

`O_APPEND`

写入操作始终附加到文件末尾。我们将在第 5.1 节讨论这个标志的意义。

`O_ASYNC`

当文件描述符上变得可进行 I/O 操作时，生成信号。这种被称为*信号驱动 I/O*的功能仅适用于某些文件类型，例如终端、FIFO 和套接字。（`O_ASYNC`标志在 SUSv3 中未被指定；然而，它或其较老的同义词`FASYNC`在大多数 UNIX 实现中都有。）在 Linux 上，当调用*open()*时指定`O_ASYNC`标志没有效果。要启用信号驱动 I/O，我们必须改为使用*fcntl()*的`F_SETFL`操作设置此标志（打开文件状态标志）。 （其他几个 UNIX 实现的行为也类似。）有关`O_ASYNC`标志的更多信息，请参见信号驱动 I/O。

`O_CLOEXEC`（自 Linux 2.6.23 起）

为新的文件描述符启用关闭执行标志（`FD_CLOEXEC`）。我们在第 27.4 节描述了`FD_CLOEXEC`标志。使用`O_CLOEXEC`标志可以使程序避免执行额外的*fcntl()* `F_GETFD`和`F_SETFD`操作来设置关闭执行标志。在多线程程序中，这也是必需的，以避免使用后者技术时可能发生的竞争条件。这些竞争可能发生在一个线程打开文件描述符后，试图同时标记它为关闭执行，而另一个线程执行*fork()*并随后执行任意程序的*exec()*。 （假设第二个线程在第一个线程打开文件描述符并使用*fcntl()*设置关闭执行标志的时间之间成功执行了*fork()*和*exec()*。）这种竞争可能导致打开的文件描述符被意外传递给不安全的程序。（我们在第 5.1 节中将更多地讨论竞争条件。）

`O_CREAT`

如果文件尚不存在，则会创建一个新的空文件。即使该文件仅用于读取，此标志也有效。如果我们指定`O_CREAT`，则必须在*open()*调用中提供*mode*参数；否则，新文件的权限将被设置为栈中的某个随机值。

`O_DIRECT`

允许文件 I/O 绕过缓冲区缓存。此功能在第 13.6 节中描述。必须定义`_GNU_SOURCE`特性测试宏，以使此常量定义在`<fcntl.h>`中可用。

`O_DIRECTORY`

如果*pathname*不是目录，则返回错误（*errno* 等于 `ENOTDIR`）。此标志是专门为实现*opendir()*（读取目录：*opendir()*和*readdir()*和 readdir()")）而设计的扩展。必须定义`_GNU_SOURCE`特性测试宏，以使此常量定义在`<fcntl.h>`中可用。

`O_DSYNC`（自 Linux 2.6.33 起）

根据同步 I/O 数据完整性完成的要求执行文件写入。有关内核 I/O 缓冲的讨论，请参见第 13.3 节。

`O_EXCL`

该标志与 `O_CREAT` 一起使用，表示如果文件已存在，则不应打开文件；相反，*open()* 应该失败，并将 *errno* 设置为 `EEXIST`。换句话说，该标志允许调用者确保它是创建文件的进程。检查文件是否存在和创建文件的操作是 *原子性* 执行的。我们在第 5.1 节中讨论了原子性的概念。当同时指定 `O_CREAT` 和 `O_EXCL` 时，如果 *pathname* 是符号链接，*open()* 会失败（错误为 `EEXIST`）。SUSv3 要求这种行为，以便特权应用程序可以在已知位置创建文件，而不会因为符号链接导致文件在不同位置（例如系统目录）创建，这可能带来安全隐患。

`O_LARGEFILE`

打开支持大文件的文件。该标志用于 32 位系统，以便处理大文件。尽管在 SUSv3 中没有规定，但在其他几个 UNIX 实现中可以找到 `O_LARGEFILE` 标志。在 Alpha 和 IA-64 等 64 位 Linux 实现中，该标志没有任何作用。有关更多信息，请参见大文件 I/O。

`O_NOATIME`（自 Linux 2.6.8 起）

读取该文件时，不更新文件的最后访问时间（在检索文件信息：*stat()*")中描述的 *st_atime* 字段）。要使用此标志，调用进程的有效用户 ID 必须与文件的所有者匹配，或者进程必须具有特权（`CAP_FOWNER`）；否则，*open()* 会因错误 `EPERM` 而失败。（实际上，对于非特权进程，当使用 `O_NOATIME` 标志打开文件时，必须匹配文件的用户 ID，而不是有效用户 ID，如第 9.5 节所述）。该标志是非标准的 Linux 扩展。为了从 `<fcntl.h>` 中暴露其定义，必须定义 `_GNU_SOURCE` 特性测试宏。`O_NOATIME` 标志旨在由索引和备份程序使用。使用该标志可以显著减少磁盘活动，因为读取文件内容并更新文件的 i-node 中的最后访问时间时，不需要反复在磁盘上进行寻道操作（即不需要来回读取磁盘）。

`O_NOCTTY`

如果正在打开的文件是一个终端设备，防止它成为控制终端。控制终端在第 34.4 节中讨论。如果打开的文件不是终端，这个标志没有效果。

`O_NOFOLLOW`

通常，*open()* 会取消引用 *pathname*，如果它是符号链接。然而，如果指定了 `O_NOFOLLOW` 标志，则如果 *pathname* 是符号链接，*open()* 将失败（并将 *errno* 设置为 `ELOOP`）。这个标志特别在特权程序中非常有用，确保 *open()* 不会取消引用符号链接。为了暴露这个标志的定义，必须定义 `_GNU_SOURCE` 特性测试宏。

`O_NONBLOCK`

以非阻塞模式打开文件。请参见第 5.9 节。

`O_SYNC`

以同步 I/O 模式打开文件。请参见第 13.3 节关于内核 I/O 缓冲的讨论。

`O_TRUNC`

如果文件已经存在并且是一个常规文件，则将其截断为零长度，销毁任何现有数据。在 Linux 上，无论文件是以读取还是写入模式打开，都会发生截断（在两种情况下，我们必须对文件具有写权限）。SUSv3 未指定 `O_RDONLY` 和 `O_TRUNC` 的组合，但大多数其他 UNIX 实现与 Linux 行为相同。

### *open()* 函数的错误

如果尝试打开文件时发生错误，*open()* 将返回 -1，*errno* 标识错误的原因。以下是一些可能发生的错误（除了上述描述 *flags* 参数时已提到的错误）：

`EACCES`

文件权限不允许调用进程以由 *flags* 指定的模式打开该文件。或者，由于目录权限，文件无法访问，或文件不存在且无法创建。

`EISDIR`

指定的文件是一个目录，且调用者尝试以写入模式打开它。这是不允许的。（另一方面，有时打开目录进行读取是有用的。我们在相对于目录文件描述符的操作中讨论了一个例子。）

`EMFILE`

已达到打开文件描述符的进程资源限制（`RLIMIT_NOFILE`，详见特定资源限制的详细信息）。

`ENFILE`

系统范围内的打开文件数限制已达到。

`ENOENT`

指定的文件不存在，并且未指定 `O_CREAT`，或者指定了 `O_CREAT`，但 *pathname* 中的某个目录不存在，或是指向不存在路径的符号链接（悬空链接）。

`EROFS`

指定的文件位于只读文件系统上，调用者尝试以写入模式打开它。

`ETXTBSY`

指定的文件是一个可执行文件（程序），且该程序当前正在执行。不能修改（即，不能以写入方式打开）与正在运行的程序相关联的可执行文件。（我们必须首先终止程序，才能修改可执行文件。）

当我们后续描述其他系统调用或库函数时，一般不会以上述方式列出可能出现的错误范围。（可以在每个系统调用或库函数的相应手册页中找到这样的列表。）我们在这里这么做有两个原因。其中之一是 *open()* 是我们详细描述的第一个系统调用，以上列表说明了一个系统调用或库函数可能因多种原因失败。第二，*open()* 失败的具体原因本身就形成了一个有趣的列表，展示了在访问文件时涉及的多个因素和检查。（上述列表不完整：有关 *open()* 失败的更多原因，请参阅 *open(2)* 手册页。）

### *creat()* 系统调用

在早期的 UNIX 实现中，*open()* 只有两个参数，且不能用于创建新文件。相反，*creat()* 系统调用用于创建并打开新文件。

```
#include <fcntl.h>

int `creat`(const char **pathname*, mode_t *mode*);
```

### 注意

返回文件描述符，出错时返回 -1

*creat()* 系统调用创建并打开一个具有给定 *pathname* 的新文件，或者如果该文件已经存在，则打开文件并将其截断为零长度。作为其功能结果，*creat()* 返回一个文件描述符，可以在后续的系统调用中使用。调用 *creat()* 相当于以下的 *open()* 调用：

```
fd = open(pathname, O_WRONLY | O_CREAT | O_TRUNC, mode);
```

由于 *open() flags* 参数提供了对文件打开方式的更大控制（例如，我们可以指定 `O_RDWR` 而不是 `O_WRONLY`），*creat()* 已经过时，尽管在较旧的程序中仍然可能看到它。

## 从文件读取：*read()*

*read()* 系统调用从由描述符 *fd* 引用的打开文件中读取数据。

```
#include <unistd.h>

ssize_t `read`(int *fd*, void **buffer*, size_t *count*);
```

### 注意

返回读取的字节数，遇到文件结尾（EOF）时返回 0，出错时返回 -1

*count* 参数指定要读取的最大字节数。（*size_t* 数据类型是一个无符号整数类型。）*buffer* 参数提供内存缓冲区的地址，用于存放输入数据。该缓冲区必须至少有 *count* 字节长。

### 注意

系统调用不会为用于返回信息的缓冲区分配内存。相反，我们必须传递一个指向先前分配的正确大小的内存缓冲区的指针。这与一些库函数不同，库函数会分配内存缓冲区以便返回信息给调用者。

成功调用 *read()* 时，返回实际读取的字节数，或者遇到文件结尾时返回 0。发生错误时，通常返回 -1。*ssize_t* 数据类型是一个带符号整数类型，用于存储字节数或 -1 错误指示。

调用*read()*时，读取的字节数可能少于请求的字节数。对于常规文件，通常的原因是文件接近末尾。

当*read()*应用于其他类型的文件时——例如管道、FIFO、套接字或终端——也有可能读取的字节数少于请求的字节数。例如，默认情况下，从终端的*read()*只会读取到下一个换行符（`\n`）为止。在接下来的章节中，我们将讨论其他文件类型时会涉及到这些情况。

使用*read()*从终端等设备输入一系列字符时，我们可能期望以下代码能够正常工作：

```
#define MAX_READ 20
char buffer[MAX_READ];

if (read(STDIN_FILENO, buffer, MAX_READ) == -1)
    errExit("read");
printf("The input data was: %s\n", buffer);
```

这段代码的输出可能会显得奇怪，因为它很可能包含除了实际输入的字符串之外的其他字符。这是因为*read()*并没有在字符串末尾放置一个终止的空字节，而*printf()*被要求打印这个字符串。经过短暂的思考后我们可以理解这一点，因为*read()*可以读取文件中的任何字节序列。在某些情况下，这些输入可能是文本，但在其他情况下，输入可能是二进制整数或二进制形式的 C 结构。*read()*无法区分这些情况，因此它不能遵循 C 语言中字符串以空字节终止的约定。如果输入缓冲区的末尾需要一个终止的空字节，我们必须显式地将其放在那里：

```
char buffer[MAX_READ + 1];
ssize_t numRead;

numRead = read(STDIN_FILENO, buffer, MAX_READ);
if (numRead == -1)
    errExit("read");

buffer[numRead] = '\0';
printf("The input data was: %s\n", buffer);
```

因为终止空字节需要一个字节的内存，所以*buffer*的大小必须至少比我们预计要读取的最大字符串大 1 个字节。

## 写入文件：*write()*

*write()*系统调用将数据写入一个打开的文件。

```
#include <unistd.h>

ssize_t `write`(int fd, void **buffer*, size_t *count*);
```

### 注意

返回写入的字节数，出错时返回-1

*write()*的参数与*read()*类似：*buffer*是要写入的数据的地址；*count*是从*buffer*写入的字节数；*fd*是一个文件描述符，指向要写入数据的文件。

成功时，*write()*返回实际写入的字节数；这可能少于*count*。对于磁盘文件，可能导致*部分写入*的原因是磁盘已满，或进程的文件大小资源限制已达到。（相关限制在第 36.3 节中描述）

在对磁盘文件进行 I/O 操作时，*write()*成功返回并不保证数据已被写入磁盘，因为内核会对磁盘 I/O 进行缓冲，以减少磁盘活动并加速*write()*调用。我们将在第十三章中讨论这些细节。

## 关闭文件：*close()*

*close()*系统调用关闭一个打开的文件描述符，使其可以被进程重新使用。当进程终止时，它的所有打开文件描述符会自动关闭。

```
#include <unistd.h>

int `close`(int *fd*);
```

### 注意

成功时返回 0，出错时返回-1

通常，显式地关闭不需要的文件描述符是一个良好的实践，因为这样可以使我们的代码在后续修改中更具可读性和可靠性。此外，文件描述符是可消耗的资源，因此如果未能关闭文件描述符，可能会导致进程耗尽描述符。这在编写处理多个文件的长生命周期程序时尤为重要，例如 shell 或网络服务器。

就像其他系统调用一样，*close()* 调用应当配备错误检查代码，如下所示：

```
if (close(fd) == -1)
    errExit("close");
```

这可以捕捉到诸如尝试关闭未打开的文件描述符或重复关闭相同文件描述符等错误，并捕捉到某些文件系统在关闭操作中可能诊断出的错误条件。

### 注意

NFS（网络文件系统）提供了一个特定于文件系统的错误示例。如果发生 NFS 提交失败，意味着数据未能到达远程磁盘，那么该错误将作为 *close()* 调用中的失败被传递到应用程序。

## 更改文件偏移量：*lseek()*

对于每个打开的文件，内核记录一个 *文件偏移量*，有时也称为 *读写偏移量* 或 *指针*。这是文件中下一次 *read()* 或 *write()* 操作开始的位置。文件偏移量是相对于文件开始的字节位置表示的。文件的第一个字节位于偏移量 0 处。

文件偏移量在文件打开时设置为指向文件的开始，并会在每次随后的 *read()* 或 *write()* 调用时自动调整，使其指向刚刚读取或写入的字节之后的下一个字节。因此，连续的 *read()* 和 *write()* 调用会顺序地通过文件进行。

*lseek()* 系统调用会根据 *offset* 和 *whence* 指定的值，调整由文件描述符 *fd* 引用的打开文件的文件偏移量。

```
#include <unistd.h>

off_t `lseek`(int *fd*, off_t *offset*, int *whence*);
```

### 注意

如果成功，返回新的文件偏移量；如果发生错误，则返回 -1。

*offset* 参数指定一个字节数值（*off_t* 数据类型是一个有符号整数类型，依据 SUSv3 规范）。*whence* 参数指示 *offset* 解释的基准点，且其值必须是以下之一：

`SEEK_SET`

文件偏移量从文件开头起设置为 *offset* 字节。

`SEEK_CUR`

文件偏移量相对于当前的文件偏移量调整 *offset* 字节。

`SEEK_END`

文件偏移量设置为文件大小加上 *offset*。换句话说，*offset* 是相对于文件最后一个字节之后的下一个字节进行解释的。

图 4-1 的 whence 参数") 显示了 *whence* 参数的解释方式。

### 注意

在早期的 UNIX 实现中，使用的是整数 0、1 和 2，而不是主文中所示的 `SEEK_*` 常量。旧版本的 BSD 使用了不同的名称来表示这些值：`L_SET`、`L_INCR` 和 `L_XTND`。

![解释 *lseek()* 的 whence 参数](img/04-1_FILEIO-A-lseek-scale90.png.jpg)图 4-1. 解释 *lseek()* 的 *whence* 参数

如果 *whence* 是 `SEEK_CUR` 或 `SEEK_END`，*offset* 可以是负值或正值；对于 `SEEK_SET`，*offset* 必须是非负值。

成功的 *lseek()* 调用的返回值是新的文件偏移量。以下调用会在不改变文件偏移量的情况下，检索当前文件偏移量的位置：

```
curr = lseek(fd, 0, SEEK_CUR);
```

### 注意

一些 UNIX 实现（但不是 Linux）提供了非标准的 *tell(fd)* 函数，它与上述 *lseek()* 调用的作用相同。

以下是一些其他的 *lseek()* 调用示例，并附有注释，指示文件偏移量移动到的位置：

```
lseek(fd, 0, SEEK_SET);         /* Start of file */
lseek(fd, 0, SEEK_END);         /* Next byte after the end of the file */
lseek(fd, -1, SEEK_END);        /* Last byte of file */
lseek(fd, -10, SEEK_CUR);       /* Ten bytes prior to current location */
lseek(fd, 10000, SEEK_END);     /* 10001 bytes past last byte of file */
```

调用 *lseek()* 仅仅调整内核记录的与文件描述符相关联的文件偏移量。它不会导致任何物理设备访问。

我们将在第 5.4 节中描述文件偏移量、文件描述符和打开文件之间关系的更多细节。

我们不能将 *lseek()* 应用于所有类型的文件。将 *lseek()* 应用于管道、FIFO、套接字或终端是不允许的；此时 *lseek()* 会失败，且 *errno* 被设置为 `ESPIPE`。另一方面，对于某些设备，应用 *lseek()* 是合理的。例如，能够在磁盘或磁带设备上定位到指定的位置。

### 注意

*lseek()* 名称中的 *l* 源自于 *offset* 参数和返回值最初都被定义为 *long* 类型。早期的 UNIX 实现提供了一个 *seek()* 系统调用，该系统调用将这些值定义为 *int* 类型。

#### 文件孔

如果程序尝试在文件末尾之后寻址，然后执行 I/O 操作，会发生什么？调用 *read()* 将返回 0，表示文件结束。令人有些惊讶的是，实际上可以在文件末尾之后的任意位置写入字节。

在文件末尾和新写入的字节之间的空间被称为 *文件孔*。从编程的角度来看，孔中的字节是存在的，从孔中读取将返回包含 0（空字节）的字节缓冲区。

然而，文件孔不会占用任何磁盘空间。文件系统不会为一个孔分配磁盘块，直到在某个以后时刻，数据被写入其中。文件孔的主要优势是，稀疏填充的文件比需要实际分配磁盘块的零字节文件消耗的磁盘空间要少。核心转储文件（Core Dump Files）是包含大孔的文件的常见示例。

### 注意

文件空洞不占用磁盘空间的说法需要稍作说明。在大多数文件系统中，文件空间是按块的单位分配的（文件系统）。块的大小取决于文件系统，但通常是 1024、2048 或 4096 字节。如果空洞的边缘位于块内，而不是在块的边界上，则会分配一个完整的块来存储块的另一部分数据，空洞对应的部分则会填充为零字节。

大多数本地 UNIX 文件系统支持文件空洞的概念，但许多非本地文件系统（例如，微软的 VFAT）不支持。在不支持空洞的文件系统上，会显式地向文件写入空字节。

存在空洞意味着文件的名义大小可能大于其实际使用的磁盘存储空间（在某些情况下，可能大得多）。将字节写入文件的空洞中会减少空闲磁盘空间，因为内核会分配块来填补空洞，即使文件的大小没有变化。这样的情况虽然不常见，但仍然需要注意。

### 注意

SUSv3 指定了一个函数，*posix_fallocate(fd, offset, len)*，它确保为由描述符*fd*引用的磁盘文件中由*offset*和*len*指定的字节范围分配磁盘空间。这使得应用程序可以确保稍后的*write()*不会因为磁盘空间耗尽而失败（否则如果文件中的空洞被填充，或者其他应用程序占用了磁盘空间，可能会发生这种情况）。历史上，*glibc*实现此函数的方式是通过向指定范围的每个块写入一个 0 字节来达到预期效果。自 2.6.23 版本起，Linux 提供了*fallocate()*系统调用，这为确保分配必要空间提供了一种更高效的方法，并且当该系统调用可用时，*glibc posix_fallocate()*实现会利用它。

I 节点描述了文件中空洞是如何表示的，检索文件信息：*stat()*")描述了*stat()*系统调用，它可以告诉我们文件的当前大小，以及实际分配给文件的块数。

#### 示例程序

示例 4-3, write(), and lseek()")演示了*read()*、*write()*和*lseek()*的结合使用。此程序的第一个命令行参数是要打开的文件名，其余参数指定对该文件执行的 I/O 操作。每个操作由一个字母后跟一个相关的值（没有空格分隔）组成：

+   `s`*偏移量*：从文件开始位置寻址到字节*偏移量*。

+   `r`*length*: 从文件的当前文件偏移量开始，读取 *length* 字节，并以文本形式显示。

+   `R`*length*: 从文件的当前文件偏移量开始，读取 *length* 字节，并以十六进制形式显示。

+   `w`*str*: 在当前文件偏移量处写入 *str* 中指定的字符字符串。

示例 4-3. 演示 *read()*, *write()* 和 *lseek()*

```
`fileio/seek_io.c`
#include <sys/stat.h>
#include <fcntl.h>
#include <ctype.h>
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    size_t len;
    off_t offset;
    int fd, ap, j;
    char *buf;
    ssize_t numRead, numWritten;

    if (argc < 3 || strcmp(argv[1], "--help") == 0)
        usageErr("%s file {r<length>|R<length>|w<string>|s<offset>}...\n",
                 argv[0]);

    fd = open(argv[1], O_RDWR | O_CREAT,
                S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP |
                S_IROTH | S_IWOTH);                     /* rw-rw-rw- */
    if (fd == -1)
        errExit("open");

    for (ap = 2; ap < argc; ap++) {
        switch (argv[ap][0]) {
        case 'r':   /* Display bytes at current offset, as text */
        case 'R':   /* Display bytes at current offset, in hex */
            len = getLong(&argv[ap][1], GN_ANY_BASE, argv[ap]);

            buf = malloc(len);
            if (buf == NULL)
                errExit("malloc");

            numRead = read(fd, buf, len);
            if (numRead == -1)
                errExit("read");

            if (numRead == 0) {
                printf("%s: end-of-file\n", argv[ap]);
            } else {
                printf("%s: ", argv[ap]);
                for (j = 0; j < numRead; j++) {
                    if (argv[ap][0] == 'r')
                        printf("%c", isprint((unsigned char) buf[j]) ?
                                                buf[j] : '?');
                    else
                        printf("%02x ", (unsigned int) buf[j]);
                }
                printf("\n");
            }

            free(buf);
            break;

        case 'w':   /* Write string at current offset */
            numWritten = write(fd, &argv[ap][1], strlen(&argv[ap][1]));
            if (numWritten == -1)
                errExit("write");
            printf("%s: wrote %ld bytes\n", argv[ap], (long) numWritten);
            break;

        case 's':   /* Change file offset */
            offset = getLong(&argv[ap][1], GN_ANY_BASE, argv[ap]);
            if (lseek(fd, offset, SEEK_SET) == -1)
                errExit("lseek");
            printf("%s: seek succeeded\n", argv[ap]);
            break;

        default:
            cmdLineErr("Argument must start with [rRws]: %s\n", argv[ap]);
        }
    }

    exit(EXIT_SUCCESS);
}
      `fileio/seek_io.c`
```

以下 shell 会话日志演示了 示例 4-3, write(), and lseek()") 中程序的使用，显示了我们尝试从文件孔中读取字节时发生的情况：

```
$ `touch tfile`                       *Create new, empty file*
$ `./seek_io tfile s100000` ``*`wabc`*``      *Seek to offset 100,000, write “abc”*
s100000: seek succeeded
wabc: wrote 3 bytes
$ `ls -l tfile`                       *Check size of file*
-rw-r--r--    1 mtk    users   100003 Feb 10 10:35 tfile
$ `./seek_io tfile s10000 R5`         *Seek to offset 10,000, read 5 bytes from hole*
s10000: seek succeeded
R5: 00 00 00 00 00                  *Bytes in the hole contain 0*
```

## 通用 I/O 模型之外的操作：*ioctl()*

*ioctl()* 系统调用是一个通用机制，用于执行不符合本章前面描述的通用 I/O 模型的文件和设备操作。

```
#include <sys/ioctl.h>

int `ioctl`(int *fd*, int *request*, ... /* *argp* */);
```

### 注意

成功时返回的值取决于 *request*，或在出错时返回 -1。

*fd* 参数是一个打开的文件描述符，表示将要执行 *request* 指定的控制操作的设备或文件。特定设备的头文件定义了可以在 *request* 参数中传递的常量。

如标准 C 中的省略号（`...`）表示法所示，*ioctl()* 的第三个参数，我们标记为 *argp*，可以是任何类型。*request* 参数的值使 *ioctl()* 能够确定在 *argp* 中预期的值类型。通常，*argp* 是指向整数或结构的指针；在某些情况下，它未使用。

我们将在后面的章节中看到 *ioctl()* 的多个使用案例（例如，参见 I-node 标志 (*ext2* 扩展文件属性)"))。

### 注意

SUSv3 对 *ioctl()* 的唯一规定是控制 STREAMS 设备的操作。（STREAMS 是一个 System V 特性，主流 Linux 内核不支持该特性，尽管已经开发出一些附加实现。）本书中描述的其他 *ioctl()* 操作在 SUSv3 中没有具体规定。然而，*ioctl()* 调用自 UNIX 系统早期版本以来就已存在，因此我们描述的几个 *ioctl()* 操作在许多其他 UNIX 实现中都有提供。在描述每个 *ioctl()* 操作时，我们会指出可移植性问题。

## 总结

为了对常规文件进行 I/O 操作，我们必须首先使用 *open()* 获取文件描述符。然后，使用 *read()* 和 *write()* 进行 I/O 操作。完成所有 I/O 操作后，我们应该使用 *close()* 释放文件描述符及其相关资源。这些系统调用可以用于对所有类型的文件进行 I/O 操作。

所有文件类型和设备驱动程序实现相同的 I/O 接口，这使得 I/O 具有通用性，这意味着一个程序通常可以与任何类型的文件一起使用，而无需编写特定于文件类型的代码。

对于每个打开的文件，内核维护一个文件偏移量，确定下一个读取或写入将发生的位置。文件偏移量会在读取和写入时隐式更新。通过使用 *lseek()*，我们可以显式地将文件偏移量重新定位到文件中的任何位置，甚至超过文件末尾。在文件的末尾写入数据会在文件中创建一个空洞。从文件空洞中读取数据会返回包含零的字节。

*ioctl()* 系统调用是一个涵盖设备和文件操作的通用方法，适用于那些不符合标准文件 I/O 模型的操作。

## 练习

1.  *tee* 命令读取标准输入直到文件结尾，并将输入的副本写入标准输出和命令行参数中指定的文件。（我们在第 44.7 节讨论 FIFO 时会展示此命令的使用示例。）使用 I/O 系统调用实现 *tee*。默认情况下，*tee* 会覆盖任何已存在的文件。实现 -*a* 命令行选项（*tee -a file*），该选项会使 *tee* 在文件已存在的情况下将文本追加到文件末尾。（参见 附录 B，其中描述了 *getopt()* 函数，能够解析命令行选项。）

1.  编写一个类似 *cp* 的程序，当用于复制包含空洞（空字节序列）的常规文件时，也会在目标文件中创建相应的空洞。
