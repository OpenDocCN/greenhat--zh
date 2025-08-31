## 第五十二章 POSIX 消息队列

本章描述了 POSIX 消息队列，它允许进程以消息的形式交换数据。POSIX 消息队列与 System V 消息队列类似，数据以完整的消息单位交换。然而，它们也有一些显著的不同之处：

+   POSIX 消息队列是引用计数的。标记为删除的队列仅在所有当前使用该队列的进程关闭后才会被删除。

+   每个 System V 消息都有一个整数类型，可以通过 *msgrcv()* 函数以多种方式选择消息。与此不同，POSIX 消息具有相关的优先级，消息总是严格按优先级顺序排队（因此也按优先级顺序接收）。

+   POSIX 消息队列提供了一项功能，允许进程在队列中有消息时异步通知。

POSIX 消息队列是 Linux 中相对较新的功能。所需的实现支持是在内核 2.6.6 中添加的（此外，还要求 *glibc* 2.3.4 或更高版本）。

### 注意

POSIX 消息队列支持是一个可选的内核组件，通过 `CONFIG_POSIX_MQUEUE` 选项进行配置。

## 概述

POSIX 消息队列 API 中的主要功能如下：

+   *mq_open()* 函数用于创建一个新的消息队列或打开一个现有的队列，并返回一个消息队列描述符，以供后续调用使用。

+   *mq_send()* 函数将消息写入队列。

+   *mq_receive()* 函数从队列中读取一条消息。

+   *mq_close()* 函数关闭进程之前打开的消息队列。

+   *mq_unlink()* 函数移除消息队列的名称，并在所有进程关闭该队列后标记该队列为删除。

上述函数的目的相当明确。此外，POSIX 消息队列 API 还有一些独特的功能：

+   每个消息队列都有一组相关的属性。部分属性可以在使用 *mq_open()* 创建或打开队列时设置。提供了两个函数来检索和更改队列属性：*mq_getattr()* 和 *mq_setattr()*。

+   *mq_notify()* 函数允许进程注册以接收来自队列的消息通知。注册后，进程将通过信号传递或在单独线程中调用函数的方式通知消息的可用性。

## 打开、关闭和取消链接消息队列

在本节中，我们将查看用于打开、关闭和删除消息队列的函数。

#### 打开消息队列

*mq_open()* 函数用于创建一个新的消息队列或打开一个现有的队列。

```
#include <fcntl.h>            /* Defines O_* constants */
#include <sys/stat.h>         /* Defines mode constants */
#include <mqueue.h>

mqd_t `mq_open`(const char **name*, int *oflag*, ...
              /* mode_t *mode*, struct mq_attr **attr* */);
```

### 注意

在成功时返回消息队列描述符，出错时返回 *(mqd_t)* -1。

*name* 参数标识消息队列，并根据第 51.1 节中给定的规则进行指定。

*oflag* 参数是一个位掩码，控制着 *mq_open()* 操作的各个方面。可以包含在此掩码中的值已在表 52-1 oflag 参数的位值")中总结。

表 52-1. *mq_open() oflag* 参数的位值

| Flag | 描述 |
| --- | --- |
| `O_CREAT` | 如果队列尚不存在，则创建队列 |
| `O_EXCL` | 与 `O_CREAT` 一起，独占地创建队列 |
| `O_RDONLY` | 仅用于读取 |
| `O_WRONLY` | 仅用于写入 |
| `O_RDWR` | 以读写模式打开 |
| `O_NONBLOCK` | 以非阻塞模式打开 |

*oflag* 参数的一个用途是确定我们是打开一个已存在的队列，还是创建并打开一个新的队列。如果 *oflag* 不包含 `O_CREAT`，则表示我们正在打开一个已存在的队列。如果 *oflag* 包含 `O_CREAT`，则如果给定的 *name* 对应的队列尚不存在，就会创建一个新的空队列。如果 *oflag* 同时指定了 `O_CREAT` 和 `O_EXCL`，并且给定的 *name* 已经存在队列，那么 *mq_open()* 会失败。

*oflag* 参数还指示调用进程对消息队列的访问方式，具体通过指定以下三种值之一：`O_RDONLY`、`O_WRONLY` 或 `O_RDWR`。

剩余的标志值 `O_NONBLOCK` 会使队列以非阻塞模式打开。如果后续对 *mq_receive()* 或 *mq_send()* 的调用无法在不阻塞的情况下执行，则该调用会立即因错误 `EAGAIN` 失败。

如果 *mq_open()* 用于打开一个已存在的消息队列，则该调用只需要两个参数。然而，如果在 *flags* 中指定了 `O_CREAT`，则需要另外两个参数：*mode* 和 *attr*。（如果由 *name* 指定的队列已经存在，这两个参数会被忽略。）这两个参数的用途如下：

+   *mode* 参数是一个位掩码，指定新消息队列的权限。可以指定的位值与文件的权限相同（表 15-4，见常规文件的权限），并且与 *open()* 一样，*mode* 中的值会与进程的 umask 进行掩码处理（进程文件模式创建掩码：*umask()*")）。为了从队列中读取数据（*mq_receive()*），必须授予相应用户类别的读取权限；为了向队列中写入数据（*mq_send()*），则需要写入权限。

+   *attr* 参数是一个 *mq_attr* 结构，指定新消息队列的属性。如果 *attr* 为 `NULL`，则队列将使用实现定义的默认属性创建。我们将在第 52.4 节描述 *mq_attr* 结构。

成功完成时，*mq_open()* 返回一个 *消息队列描述符*，该值为 *mqd_t* 类型，用于在后续的调用中引用此打开的消息队列。SUSv3 对该数据类型的唯一要求是，它不能是数组；即，它必须是一个可以在赋值语句中使用或作为函数参数按值传递的类型。（在 Linux 上，*mqd_t* 是 *int*，但例如在 Solaris 上，它被定义为 *void**。）

示例 52-2 中提供了使用 *mq_open()* 的示例。

#### *fork()*, *exec()* 和进程终止对消息队列描述符的影响

在 *fork()* 过程中，子进程会收到父进程的消息队列描述符的副本，这些描述符指向相同的打开消息队列描述符。（我们在第 52.3 节中解释了消息队列描述符。）子进程不会继承父进程的任何消息通知注册。

当一个进程执行 *exec()* 或终止时，它所有打开的消息队列描述符都会被关闭。由于关闭了消息队列描述符，进程在相应队列上的所有消息通知注册都会被取消。

#### 关闭消息队列

*mq_close()* 函数关闭消息队列描述符 *mqdes*。

```
#include <mqueue.h>

int `mq_close`(mqd_t *mqdes*);
```

### 注意

成功时返回 0，出错时返回 -1

如果调用进程通过 *mqdes* 在队列上注册了消息通知（消息通知），那么通知注册会自动移除，之后其他进程可以注册该队列的消息通知。

当进程终止或调用 *exec()* 时，消息队列描述符会被自动关闭。与文件描述符一样，我们应当显式关闭不再需要的消息队列描述符，以防止进程耗尽消息队列描述符。

与文件的 *close()* 类似，关闭消息队列并不会删除它。为了删除消息队列，我们需要使用 *mq_unlink()*，它是 *unlink()* 的消息队列类比。

#### 移除消息队列

*mq_unlink()* 函数移除由 *name* 标识的消息队列，并将该队列标记为在所有进程停止使用后销毁（如果所有打开队列的进程已经关闭该队列，可能会立即销毁）。

```
#include <mqueue.h>

int `mq_unlink`(const char **name*);
```

### 注意

成功时返回 0，出错时返回 -1

示例 52-1 断开 POSIX 消息队列") 演示了如何使用 *mq_unlink()*。

示例 52-1：使用 *mq_unlink()* 断开 POSIX 消息队列

```
`pmsg/pmsg_unlink.c`
#include <mqueue.h>
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    if (argc != 2 || strcmp(argv[1], "--help") == 0)
        usageErr("%s mq-name\n", argv[0]);

    if (mq_unlink(argv[1]) == -1)
        errExit("mq_unlink");
    exit(EXIT_SUCCESS);
}
    `pmsg/pmsg_unlink.c`
```

## 描述符与消息队列之间的关系

消息队列描述符和打开的消息队列之间的关系类似于文件描述符和打开文件之间的关系（参见图 5-2，以及文件描述符与打开文件之间的关系）。消息队列描述符是每个进程的句柄，指向系统范围内打开消息队列描述符表中的一个条目，而该条目又指向一个消息队列对象。这个关系如图 52-1 所示。

### 注意

在 Linux 上，POSIX 消息队列作为虚拟文件系统中的 i 节点实现，消息队列描述符和打开的消息队列描述符分别作为文件描述符和打开文件描述符实现。然而，这些是实现细节，并非 SUSv3 的要求，在其他一些 UNIX 实现中并不成立。尽管如此，我们将在 Linux 特定功能一节回到这一点，因为 Linux 提供了一些由此实现所支持的非标准功能。

![POSIX 消息队列的内核数据结构关系](img/52-1_PMSG-mq-model.png.jpg)图 52-1. POSIX 消息队列的内核数据结构关系

图 52-1 帮助澄清了消息队列描述符使用的一些细节（这些都类似于文件描述符的使用）：

+   打开的消息队列描述符有一组相关的标志。SUSv3 只指定了一个这样的标志 `O_NONBLOCK`，它决定了 I/O 是否为非阻塞模式。

+   两个进程可以持有指向同一个打开的消息队列描述符的消息队列描述符（图中描述符 *x*）。这可以发生在一个进程打开消息队列后调用 *fork()*。这些描述符共享 `O_NONBLOCK` 标志的状态。

+   两个进程可以持有指向不同消息队列描述符的打开的消息队列描述符，而这些描述符却指向同一个消息队列（例如，进程 A 中的描述符 *z* 和进程 B 中的描述符 *y* 都指向 `/mq-r`）。这是因为两个进程各自使用 *mq_open()* 打开了同一个队列。

## 消息队列属性

*mq_open()*、*mq_getattr()* 和 *mq_setattr()* 函数都允许传入一个指向 *mq_attr* 结构的指针。这个结构在 `<mqueue.h>` 中定义如下：

```
struct mq_attr {
    long mq_flags;        /* Message queue description flags: 0 or
                             O_NONBLOCK [mq_getattr(), mq_setattr()] */
    long mq_maxmsg;       /* Maximum number of messages on queue
                             [mq_open(), mq_getattr()] */
    long mq_msgsize;      /* Maximum message size (in bytes)
                             [mq_open(), mq_getattr()] */
    long mq_curmsgs;      /* Number of messages currently in queue
                             [mq_getattr()] */
};
```

在我们详细查看 *mq_attr* 结构之前，值得注意以下几点：

+   每个函数只使用其中的一部分字段。每个函数使用的字段在上面结构定义的注释中有所指示。

+   该结构包含有关打开的消息队列描述符（*mq_flags*）的信息，这些描述符与消息描述符相关联，还包含与该描述符关联的队列的信息（*mq_maxmsg*，*mq_msgsize*，*mq_curmsgs*）。

+   一些字段包含在使用 *mq_open()* 创建队列时固定的信息（*mq_maxmsg* 和 *mq_msgsize*）；其他字段返回有关当前消息队列描述符（*mq_flags*）或消息队列（*mq_curmsgs*）的状态信息。

#### 在队列创建期间设置消息队列属性

当我们使用 *mq_open()* 创建一个消息队列时，以下 *mq_attr* 字段决定了队列的属性：

+   *mq_maxmsg* 字段定义了使用 *mq_send()* 放置在队列中的消息数量的上限。该值必须大于 0。

+   *mq_msgsize* 字段定义了可以放置在队列中的每条消息的大小上限。该值必须大于 0。

这两个值一起允许内核确定该消息队列可能需要的最大内存量。

*mq_maxmsg* 和 *mq_msgsize* 属性在消息队列创建时被固定；它们之后不能更改。在 消息队列限制一节中，我们描述了两个 `/proc` 文件，它们对可以为 *mq_maxmsg* 和 *mq_msgsize* 属性指定的值设置了系统范围的限制。

示例 52-2 中的程序提供了一个命令行界面来使用 *mq_open()* 函数，并展示了如何将 *mq_attr* 结构与 *mq_open()* 一起使用。

有两个命令行选项允许指定消息队列属性：* -m * 用于 *mq_maxmsg*，* -s * 用于 *mq_msgsize*。如果提供了这些选项中的任意一个，将一个非 `NULL` 的 *attrp* 参数传递给 *mq_open()*。如果只在命令行中指定了 *-m* 或 *-s* 中的一个选项，将为 *attrp* 指向的 *mq_attr* 结构体字段分配一些默认值。如果两个选项都没有提供，则在调用 *mq_open()* 时，*attrp* 将被指定为 `NULL`，这将导致队列使用实现定义的默认值来创建队列属性。

示例 52-2. 创建一个 POSIX 消息队列

```
`pmsg/pmsg_create.c`
#include <mqueue.h>
#include <sys/stat.h>
#include <fcntl.h>
#include "tlpi_hdr.h"

static void
usageError(const char *progName)
{
    fprintf(stderr, "Usage: %s [-cx] [-m maxmsg] [-s msgsize] mq-name "
            "[octal-perms]\n", progName);
    fprintf(stderr, "    -c          Create queue (O_CREAT)\n");
    fprintf(stderr, "    -m maxmsg   Set maximum # of messages\n");
    fprintf(stderr, "    -s msgsize  Set maximum message size\n");
    fprintf(stderr, "    -x          Create exclusively (O_EXCL)\n");
    exit(EXIT_FAILURE);
}

int
main(int argc, char *argv[])
{
    int flags, opt;
    mode_t perms;
    mqd_t mqd;
    struct mq_attr attr, *attrp;

    attrp = NULL;
    attr.mq_maxmsg = 50;
    attr.mq_msgsize = 2048;
    flags = O_RDWR;

    /* Parse command-line options */

    while ((opt = getopt(argc, argv, "cm:s:x")) != -1) {
        switch (opt) {
        case 'c':
            flags |= O_CREAT;
            break;

        case 'm':
            attr.mq_maxmsg = atoi(optarg);
            attrp = &attr;
            break;

        case 's':
            attr.mq_msgsize = atoi(optarg);
            attrp = &attr;
            break;

        case 'x':
            flags |= O_EXCL;
            break;

        default:
            usageError(argv[0]);
        }
    }

    if (optind >= argc)
        usageError(argv[0]);

    perms = (argc <= optind + 1) ? (S_IRUSR | S_IWUSR) :
                getInt(argv[optind + 1], GN_BASE_8, "octal-perms");

    mqd = mq_open(argv[optind], flags, perms, attrp);
    if (mqd == (mqd_t) -1)
        errExit("mq_open");

    exit(EXIT_SUCCESS);
}
    `pmsg/pmsg_create.c`
```

#### 检索消息队列属性

*mq_getattr()* 函数返回一个 *mq_attr* 结构体，其中包含关于消息队列描述符 *mqdes* 关联的消息队列的信息。

```
#include <mqueue.h>

int `mq_getattr`(mqd_t *mqdes*, struct mq_attr **attr*);
```

### 注意

成功时返回 0，错误时返回 -1

除了我们已经描述的 *mq_maxmsg* 和 *mq_msgsize* 字段外，以下字段也会在 *attr* 指向的结构体中返回：

*mq_flags*

这些是与描述符 *mqdes* 关联的打开的消息队列描述符的标志。仅指定了一个标志：`O_NONBLOCK`。这个标志由 *mq_open()* 的 *oflag* 参数初始化，并且可以通过 *mq_setattr()* 更改。

*mq_curmsgs*

这是当前队列中的消息数量。如果其他进程正在从队列中读取消息或向其写入消息，则在 *mq_getattr()* 返回时，此信息可能已经发生变化。

示例 52-3 中的程序使用 *mq_getattr()* 获取命令行参数中指定的消息队列的属性，然后将这些属性显示在标准输出上。

示例 52-3. 检索 POSIX 消息队列属性

```
`pmsg/pmsg_getattr.c`
#include <mqueue.h>
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    mqd_t mqd;
    struct mq_attr attr;

    if (argc != 2 || strcmp(argv[1], "--help") == 0)
        usageErr("%s mq-name\n", argv[0]);

    mqd = mq_open(argv[1], O_RDONLY);
    if (mqd == (mqd_t) -1)
        errExit("mq_open");

    if (mq_getattr(mqd, &attr) == -1)
        errExit("mq_getattr");

    printf("Maximum # of messages on queue:   %ld\n", attr.mq_maxmsg);
    printf("Maximum message size:             %ld\n", attr.mq_msgsize);
    printf("# of messages currently on queue: %ld\n", attr.mq_curmsgs);
    exit(EXIT_SUCCESS);
}
    `pmsg/pmsg_getattr.c`
```

在以下的 shell 会话中，我们使用示例 52-2 中的程序创建一个具有实现定义默认属性的消息队列（即，*mq_open()*的最后一个参数是`NULL`），然后使用示例 52-3 中的程序显示队列属性，以便我们能够查看 Linux 上的默认设置。

```
$ `./pmsg_create -cx /mq`
$ `./pmsg_getattr /mq`
Maximum # of messages on queue:   10
Maximum message size:             8192
# of messages currently on queue: 0
$ `./pmsg_unlink /mq`                             *Remove message queue*
```

从上述输出中，我们看到 Linux 上的默认值，*mq_maxmsg* 为 10，*mq_msgsize* 为 8192。

对于 *mq_maxmsg* 和 *mq_msgsize* 的实现定义默认值差异很大。便携式应用程序通常需要为这些属性选择明确的值，而不是依赖于默认值。

#### 修改消息队列属性

*mq_setattr()* 函数设置与消息队列描述符 *mqdes* 相关联的消息队列描述符的属性，并可选择性地返回有关消息队列的信息。

```
#include <mqueue.h>

int `mq_setattr`(mqd_t *mqdes*, const struct mq_attr **newattr*,
               struct mq_attr **oldattr*);
```

### 注意

成功时返回 0，出错时返回 -1

*mq_setattr()* 函数执行以下任务：

+   它使用 *mq_attr* 结构中由 *newattr* 指向的 *mq_flags* 字段来更改与描述符 *mqdes* 关联的消息队列描述符的标志。

+   如果 *oldattr* 非 `NULL`，它将返回一个包含先前消息队列描述符标志和消息队列属性的 *mq_attr* 结构（即，与 *mq_getattr()* 执行的任务相同）。

SUSv3 规定的唯一可以通过 *mq_setattr()* 更改的属性是 `O_NONBLOCK` 标志的状态。

考虑到某些实现可能会定义其他可修改的标志，或者 SUSv3 将来可能会添加新的标志，便携式应用程序应该使用 *mq_getattr()* 来检索 *mq_flags* 值，修改 `O_NONBLOCK` 位，然后调用 *mq_setattr()* 来更改 *mq_flags* 设置。举例来说，若要启用 `O_NONBLOCK`，我们可以这样做：

```
if (mq_getattr(mqd, &attr) == -1)
    errExit("mq_getattr");
attr.mq_flags |= O_NONBLOCK;
if (mq_setattr(mqd, &attr, NULL) == -1)
    errExit("mq_getattr");
```

## 交换消息

本节介绍了用于向队列发送消息和接收消息的函数。

### 发送消息

*mq_send()* 函数将由 *msg_ptr* 指向的缓冲区中的消息添加到由描述符 *mqdes* 引用的消息队列中。

```
#include <mqueue.h>

int `mq_send`(mqd_t *mqdes*, const char **msg_ptr*, size_t *msg_len*,
            unsigned int *msg_prio*);
```

### 注意

成功时返回 0，出错时返回 -1。

*msg_len* 参数指定由 *msg_ptr* 指向的消息的长度。该值必须小于或等于队列的 *mq_msgsize* 属性，否则 *mq_send()* 会因错误 `EMSGSIZE` 而失败。零长度的消息是允许的。

每条消息都有一个非负整数的优先级，由 *msg_prio* 参数指定。消息在队列中按优先级降序排列（即，0 是最低优先级）。当新消息添加到队列时，它会排在任何其他相同优先级的消息之后。如果应用程序不需要使用消息优先级，则始终将 *msg_prio* 指定为 0 即可。

### 注意

正如本章开头所提到的，System V 消息的类型属性提供了不同的功能。System V 消息总是按 FIFO 顺序排队，但 *msgrcv()* 允许我们以各种方式选择消息：按 FIFO 顺序，按精确类型，或按小于或等于某个值的最大类型。

SUSv3 允许实现广告其消息优先级的上限，方法是定义常量 `MQ_PRIO_MAX` 或通过 *sysconf(_SC_MQ_PRIO_MAX)* 返回。SUSv3 要求此限制至少为 32 `(_POSIX_MQ_PRIO_MAX)`；即，至少提供从 0 到 31 的优先级。然而，实际实现中的范围差异很大。例如，在 Linux 上，此常量的值为 32,768；在 Solaris 上为 32；在 Tru64 上为 256。

如果消息队列已经满（即队列的 *mq_maxmsg* 限制已达到），则进一步调用 *mq_send()* 要么会阻塞直到队列中有空间可用，要么如果设置了 `O_NONBLOCK` 标志，则会立即因错误 `EAGAIN` 而失败。

示例 52-4 中的程序提供了一个命令行接口来调用 *mq_send()* 函数。我们将在下一节演示如何使用此程序。

示例 52-4. 向 POSIX 消息队列写入消息

```
`pmsg/pmsg_send.c`
#include <mqueue.h>
#include <fcntl.h>              /* For definition of O_NONBLOCK */
#include "tlpi_hdr.h"

static void
usageError(const char *progName)
{
    fprintf(stderr, "Usage: %s [-n] name msg [prio]\n", progName);
    fprintf(stderr, "    -n           Use O_NONBLOCK flag\n");
    exit(EXIT_FAILURE);
}

int
main(int argc, char *argv[])
{
    int flags, opt;
    mqd_t mqd;
    unsigned int prio;

    flags = O_WRONLY;
    while ((opt = getopt(argc, argv, "n")) != -1) {
        switch (opt) {
        case 'n':   flags |= O_NONBLOCK;        break;
        default:    usageError(argv[0]);
        }
    }

    if (optind + 1 >= argc)
        usageError(argv[0]);

    mqd = mq_open(argv[optind], flags);
    if (mqd == (mqd_t) -1)
        errExit("mq_open");

    prio = (argc > optind + 2) ? atoi(argv[optind + 2]) : 0;

    if (mq_send(mqd, argv[optind + 1], strlen(argv[optind + 1]), prio) == -1)
        errExit("mq_send");
    exit(EXIT_SUCCESS);
}
    `pmsg/pmsg_send.c`
```

### 接收消息

*mq_receive()* 函数从由 *mqdes* 引用的消息队列中移除优先级最高的最旧消息，并将该消息返回到由 *msg_ptr* 指向的缓冲区中。

```
#include <mqueue.h>

ssize_t `mq_receive`(mqd_t *mqdes*, char **msg_ptr*, size_t *msg_len*,
                   unsigned int **msg_prio*);
```

### 注意

成功时返回接收消息的字节数，出错时返回 -1。

*msg_len* 参数由调用者使用，用于指定由 *msg_ptr* 指向的缓冲区中可用的字节数。

无论消息的实际大小如何，*msg_len*（因此指向*msg_ptr*的缓冲区大小）必须大于或等于队列的*mq_msgsize*属性；否则，*mq_receive()*会因错误`EMSGSIZE`而失败。如果我们不知道队列的*mq_msgsize*属性值，可以使用*mq_getattr()*来获取。（在由协作进程组成的应用程序中，通常可以省略使用*mq_getattr()*，因为应用程序通常可以预先决定队列的*mq_msgsize*设置。）

如果*msg_prio*不是`NULL`，那么接收到的消息的优先级将被复制到*msg_prio*指向的位置。

如果消息队列当前为空，则*mq_receive()*要么阻塞直到有消息可用，要么，如果启用了`O_NONBLOCK`标志，立即失败并返回错误`EAGAIN`。（这与管道行为不同，管道在没有写入者时，读取者会看到文件结束标志。）

示例 52-5 中的程序提供了一个命令行界面来调用*mq_receive()*函数。该程序的命令格式如*usageError()*函数所示。

以下 Shell 会话演示了在示例 52-4 和示例 52-5 中使用程序的示例。我们首先创建一个消息队列，并发送几个不同优先级的消息：

```
$ `./pmsg_create -cx /mq`
$ `./pmsg_send /mq msg-a 5`
$ `./pmsg_send /mq msg-b 0`
$ `./pmsg_send /mq msg-c 10`
```

然后我们执行一系列命令来从队列中检索消息：

```
$ `./pmsg_receive /mq`
Read 5 bytes; priority = 10
msg-c
$ `./pmsg_receive /mq`
Read 5 bytes; priority = 5
msg-a
$ `./pmsg_receive /mq`
Read 5 bytes; priority = 0
msg-b
```

从上面的输出可以看到，消息是按优先级顺序检索的。

此时，队列已经为空。当我们执行另一个阻塞接收时，操作会阻塞：

```
$ `./pmsg_receive /mq`
*Blocks; we type Control-C to terminate the program*
```

另一方面，如果我们执行非阻塞接收，调用会立即返回并显示失败状态：

```
$ `./pmsg_receive -n /mq`
ERROR [EAGAIN/EWOULDBLOCK Resource temporarily unavailable] mq_receive
```

示例 52-5. 从 POSIX 消息队列读取消息

```
`pmsg/pmsg_receive.c`
#include <mqueue.h>
#include <fcntl.h>              /* For definition of O_NONBLOCK */
#include "tlpi_hdr.h"

static void
usageError(const char *progName)
{
    fprintf(stderr, "Usage: %s [-n] name\n", progName);
    fprintf(stderr, "    -n           Use O_NONBLOCK flag\n");
    exit(EXIT_FAILURE);
}

int
main(int argc, char *argv[])
{
    int flags, opt;
    mqd_t mqd;
    unsigned int prio;
    void *buffer;
    struct mq_attr attr;
    ssize_t numRead;

    flags = O_RDONLY;
    while ((opt = getopt(argc, argv, "n")) != -1) {
        switch (opt) {
        case 'n':   flags |= O_NONBLOCK;        break;
        default:    usageError(argv[0]);
        }
    }

    if (optind >= argc)
        usageError(argv[0]);

    mqd = mq_open(argv[optind], flags);
    if (mqd == (mqd_t) -1)
        errExit("mq_open");

    if (mq_getattr(mqd, &attr) == -1)
        errExit("mq_getattr");

    buffer = malloc(attr.mq_msgsize);
    if (buffer == NULL)
        errExit("malloc");

    numRead = mq_receive(mqd, buffer, attr.mq_msgsize, &prio);
    if (numRead == -1)
        errExit("mq_receive");

    printf("Read %ld bytes; priority = %u\n", (long) numRead, prio);
    if (write(STDOUT_FILENO, buffer, numRead) == -1)
        errExit("write");
    write(STDOUT_FILENO, "\n", 1);

    exit(EXIT_SUCCESS);
}
    `pmsg/pmsg_receive.c`
```

### 带有超时的消息发送和接收

*mq_timedsend()*和*mq_timedreceive()*函数与*mq_send()*和*mq_receive()*完全相同，不同之处在于，如果操作不能立即执行，且消息队列描述符上没有`O_NONBLOCK`标志，那么*abs_timeout*参数会指定调用阻塞的时间限制。

```
#include <mqueue.h>
#include <time.h>

int `mq_timedsend`(mqd_t *mqdes*, const char **msg_ptr*, size_t *msg_len*,
                 unsigned int *msg_prio*, const struct timespec **abs_timeout*);
```

### 注意

成功时返回 0，出错时返回-1

```
ssize_t `mq_timedreceive`(mqd_t *mqdes*, char **msg_ptr*, size_t *msg_len*,
                 unsigned int **msg_prio*, const struct timespec **abs_timeout*);
```

### 注意

成功时返回接收到的消息的字节数，出错时返回-1

*abs_timeout* 参数是一个 *timespec* 结构体（高精度睡眠：*nanosleep()*")），它指定了从纪元起的秒和纳秒的绝对超时时间。要执行相对超时，我们可以使用 *clock_gettime()* 获取 `CLOCK_REALTIME` 时钟的当前值，并将所需的时间量加到该值上，以生成一个适当初始化的 *timespec* 结构体。

如果 *mq_timedsend()* 或 *mq_timedreceive()* 调用超时，无法完成操作，则该调用失败，并返回错误 `ETIMEDOUT`。

在 Linux 上，将 *abs_timeout* 设置为 `NULL` 表示无限超时。然而，这种行为并未在 SUSv3 中指定，因此可移植的应用程序不能依赖它。

*mq_timedsend()* 和 *mq_timedreceive()* 函数最初来自 POSIX.1d (1999)，并非所有 UNIX 实现都支持它们。

## 消息通知

POSIX 消息队列与 System V 消息队列的一个区别是，POSIX 消息队列能够在队列之前为空时收到消息到达的异步通知（即，当队列从空转为非空时）。这一特性意味着，进程无需进行阻塞的 *mq_receive()* 调用或将消息队列描述符标记为非阻塞并定期执行 *mq_receive()* 调用（即“轮询”），而是可以请求在有消息到达时收到通知，之后继续执行其他任务，直到收到通知。进程可以选择通过信号或通过在单独线程中调用函数来接收通知。

### 注意

POSIX 消息队列的通知功能类似于我们在第 23.6 节中为 POSIX 定时器描述的通知机制。（这两个 API 都源自 POSIX.1b。）

*mq_notify()* 函数将调用进程注册为在描述符 *mqdes* 所引用的空队列上到达消息时接收通知。

```
#include <mqueue.h>

int `mq_notify`(mqd_t *mqdes*, const struct sigevent **notification*);
```

### 注意

成功时返回 0，出错时返回 -1

*notification* 参数指定了进程接收通知的机制。在详细介绍 *notification* 参数之前，我们先注意一些关于消息通知的要点：

+   在任何时候，只有一个进程（“注册进程”）可以被注册以接收特定消息队列的通知。如果该消息队列已经有一个进程注册，则进一步注册该队列的尝试会失败（*mq_notify()* 会返回错误 `EBUSY`）。

+   注册的进程仅在先前为空的队列上到达新消息时收到通知。如果队列在注册时已经包含消息，则仅在队列清空并且有新消息到达时才会收到通知。

+   在向注册进程发送一次通知后，注册会被移除，任何进程都可以重新注册以接收通知。换句话说，只要一个进程希望继续接收通知，它必须在每次通知后通过再次调用*mq_notify()*重新注册。

+   只有当其他进程没有在调用*mq_receive()*时被阻塞，注册的进程才会被通知。如果其他进程在*mq_receive()*中被阻塞，该进程将读取消息，而注册的进程将保持注册状态。

+   进程可以通过调用*mq_notify()*并将*notification*参数设为`NULL`，显式地取消注册自己作为消息通知的目标。

我们已经在创建计时器：*timer_create()*")中展示了用于指定*notification*参数的*sigevent*结构。这里，我们以简化的形式呈现该结构，仅显示与讨论*mq_notify()*相关的字段：

```
union sigval {
    int    sival_int;             /* Integer value for accompanying data */
    void  *sival_ptr;             /* Pointer value for accompanying data */
};

struct sigevent {
    int    sigev_notify;          /* Notification method */
    int    sigev_signo;           /* Notification signal for SIGEV_SIGNAL */
    union sigval sigev_value;     /* Value passed to signal handler or
                                     thread function */
    void (*sigev_notify_function) (union sigval);
                                  /* Thread notification function */
    void  *sigev_notify_attributes;   /* Really 'pthread_attr_t' */
};
```

该结构的*sigev_notify*字段被设置为以下值之一：

`SIGEV_NONE`

注册该进程以接收通知，但当先前空的队列中有消息到达时，不会实际通知进程。像往常一样，当新消息到达空队列时，注册会被移除。

`SIGEV_SIGNAL`

通过生成在*sigev_signo*字段中指定的信号来通知进程。*sigev_value*字段指定随信号一起传递的数据（队列中的实时信号数量限制）。此数据可以通过传递给信号处理程序的*siginfo_t*结构中的*si_value*字段，或者通过调用*sigwaitinfo()*或*sigtimedwait()*返回的数据进行检索。*siginfo_t*结构中的以下字段也会被填写：*si_code*，值为`SI_MESGQ`；*si_signo*，信号编号；*si_pid*，发送消息的进程的进程 ID；*si_uid*，发送消息的进程的真实用户 ID。（*si_pid*和*si_uid*字段在大多数其他实现中未设置。）

`SIGEV_THREAD`

通过调用在*sigev_notify_function*中指定的函数来通知进程，就像它是新线程中的启动函数一样。*sigev_notify_attributes*字段可以指定为`NULL`，或者作为指向定义线程属性的*pthread_attr_t*结构的指针（线程属性）。在*sigev_value*中指定的联合*sigval*值将作为此函数的参数传递。

### 通过信号接收通知

示例 52-6 提供了使用信号进行消息通知的示例。该程序执行以下步骤：

1.  以非阻塞模式打开命令行中指定的消息队列 ![](img/U001.png)，确定队列的*mq_msgsize*属性 ![](img/U002.png)，并为接收消息分配该大小的缓冲区 ![](img/U003.png)。

1.  阻塞通知信号（`SIGUSR1`）并为其设置处理程序 ![](img/U004.png)。

1.  初次调用*mq_notify()*以注册进程接收消息通知 ![](img/U005.png)。

1.  执行一个无限循环，执行以下步骤：

    1.  调用*sigsuspend()*，该函数解锁通知信号并等待信号的到来 ![](img/U006.png)。从此系统调用返回表示消息通知已经发生。此时，进程将不再注册消息通知。

    1.  调用*mq_notify()*重新注册该进程以接收消息通知 ![](img/U007.png)。

    1.  执行`while`循环，读取尽可能多的消息以清空队列 ![](img/U008.png)。

示例 52-6. 通过信号接收消息通知

```
`pmsg/mq_notify_sig.c`
    #include <signal.h>
    #include <mqueue.h>
    #include <fcntl.h>              /* For definition of O_NONBLOCK */
    #include "tlpi_hdr.h"

    #define NOTIFY_SIG SIGUSR1

    static void
    handler(int sig)
    {
        /* Just interrupt sigsuspend() */
    }

    int
    main(int argc, char *argv[])
    {
        struct sigevent sev;
        mqd_t mqd;
        struct mq_attr attr;
        void *buffer;
        ssize_t numRead;
        sigset_t blockMask, emptyMask;
        struct sigaction sa;

        if (argc != 2 || strcmp(argv[1], "--help") == 0)
            usageErr("%s mq-name\n", argv[0]);

    mqd = mq_open(argv[1], O_RDONLY | O_NONBLOCK);
        if (mqd == (mqd_t) -1)
            errExit("mq_open");

    if (mq_getattr(mqd, &attr) == -1)
            errExit("mq_getattr");

    buffer = malloc(attr.mq_msgsize);
        if (buffer == NULL)
            errExit("malloc");

    sigemptyset(&blockMask);
        sigaddset(&blockMask, NOTIFY_SIG);
        if (sigprocmask(SIG_BLOCK, &blockMask, NULL) == -1)
            errExit("sigprocmask");

            sigemptyset(&sa.sa_mask);
        sa.sa_flags = 0;
        sa.sa_handler = handler;
        if (sigaction(NOTIFY_SIG, &sa, NULL) == -1)
            errExit("sigaction");

    sev.sigev_notify = SIGEV_SIGNAL;
        sev.sigev_signo = NOTIFY_SIG;
        if (mq_notify(mqd, &sev) == -1)
            errExit("mq_notify");

        sigemptyset(&emptyMask);

        for (;;) {
        sigsuspend(&emptyMask);         /* Wait for notification signal */

        if (mq_notify(mqd, &sev) == -1)
                errExit("mq_notify");

        while ((numRead = mq_receive(mqd, buffer,
 attr.mq_msgsize, NULL)) >= 0)
                printf("Read %ld bytes\n", (long) numRead);

            if (errno != EAGAIN)            /* Unexpected error */
                errExit("mq_receive");
        }
    }
        `pmsg/mq_notify_sig.c`
```

Example 52-6 中的程序各个方面值得进一步评论：

+   我们阻塞通知信号并使用*sigsuspend()*等待它，而不是使用*pause()*，以防止程序在`for`循环中执行时错过已传递的信号（即，未阻塞等待信号）。如果发生这种情况，并且我们使用*pause()*等待信号，那么下次调用*pause()*时会被阻塞，即使信号已经传递。

+   我们以非阻塞模式打开队列，并且每当通知发生时，我们使用`while`循环读取队列中的所有消息。通过这种方式清空队列，确保当新消息到达时会生成进一步的通知。使用非阻塞模式意味着当我们清空队列时，`while`循环将终止（*mq_receive()*会因为错误`EAGAIN`而失败）。这种方法类似于使用带有边缘触发 I/O 通知的非阻塞 I/O，我们在 Which technique?中描述了该技术，并且出于类似的原因使用它。

+   在`for`循环中，我们必须在读取队列中的所有消息之前重新注册消息通知，*而不是*在读取后再进行注册。如果我们反转这些步骤，可能会出现以下情况：所有消息都被从队列中读取，`while`循环终止；另一条消息被放入队列；调用*mq_notify()*重新注册消息通知。此时，不会再生成进一步的通知信号，因为队列已经非空。因此，程序会在下一次调用*sigsuspend()*时永久阻塞。

### 通过线程接收通知

示例 52-7 提供了一个使用线程的消息通知示例。该程序与示例 52-6 中的程序共享一些设计特点：

+   当消息通知发生时，程序会在清空队列之前重新启用通知 ![](img/U002.png)。

+   使用非阻塞模式，以便在接收到通知后，我们能够在不阻塞的情况下完全清空队列 ![](img/U005.png)。

示例 52-7. 通过线程接收消息通知

```
`pmsg/mq_notify_thread.c`
    #include <pthread.h>
    #include <mqueue.h>
    #include <fcntl.h>              /* For definition of O_NONBLOCK */
    #include "tlpi_hdr.h"

    static void notifySetup(mqd_t *mqdp);

    static void                     /* Thread notification function */
threadFunc(union sigval sv)
    {
        ssize_t numRead;
        mqd_t *mqdp;
        void *buffer;
        struct mq_attr attr;

        mqdp = sv.sival_ptr;

        if (mq_getattr(*mqdp, &attr) == -1)
            errExit("mq_getattr");

        buffer = malloc(attr.mq_msgsize);
        if (buffer == NULL)
            errExit("malloc");

    notifySetup(mqdp);

        while ((numRead = mq_receive(*mqdp, buffer, attr.mq_msgsize, NULL)) >= 0)
            printf("Read %ld bytes\n", (long) numRead);

        if (errno != EAGAIN)                        /* Unexpected error */
            errExit("mq_receive");

        free(buffer);
        pthread_exit(NULL);
    }

    static void
    notifySetup(mqd_t *mqdp)
    {
        struct sigevent sev;

        sev.sigev_notify = SIGEV_THREAD;            /* Notify via thread */
        sev.sigev_notify_function = threadFunc;
        sev.sigev_notify_attributes = NULL;
                /* Could be pointer to pthread_attr_t structure */
    sev.sigev_value.sival_ptr = mqdp;           /* Argument to threadFunc() */

        if (mq_notify(*mqdp, &sev) == -1)
            errExit("mq_notify");
    }

    int
    main(int argc, char *argv[])
    {
        mqd_t mqd;

        if (argc != 2 || strcmp(argv[1], "--help") == 0)
            usageErr("%s mq-name\n", argv[0]);

    mqd = mq_open(argv[1], O_RDONLY | O_NONBLOCK);
        if (mqd == (mqd_t) -1)
            errExit("mq_open");

    notifySetup(&mqd);
        pause();                    /* Wait for notifications via thread function */
    }

        `pmsg/mq_notify_thread.c`
```

注意以下几点，关于示例 52-7 中程序的设计：

+   程序通过线程请求通知，方法是在传递给*mq_notify()*的*sigevent*结构的*sigev_notify*字段中指定`SIGEV_THREAD`。线程的启动函数*threadFunc()*在*sigev_notify_function*字段中指定 ![](img/U003.png)。

+   启用消息通知后，主程序会无限期暂停 ![](img/U006.png)；定时器通知通过在单独线程中调用*threadFunc()*进行交付 ![](img/U001.png)。

+   我们本可以通过将消息队列描述符*mqd*设为全局变量，使其在*threadFunc()*中可见。然而，我们采用了不同的方式来说明另一种选择：我们将消息队列描述符的地址放入传递给*mq_notify()*的*sigev_value.sival_ptr*字段中！[](figs/web/U004.png)。当稍后调用*threadFunc()*时，这个地址会作为其参数传递。

### 注意

我们必须将指针分配给消息队列描述符的*sigev_value.sival_ptr*，而不是（某个类型转换版本的）描述符本身，因为，除了要求它不是数组类型外，SUSv3 并未保证用于表示*mqd_t*数据类型的类型的性质或大小。

## 特定于 Linux 的功能

Linux 实现的 POSIX 消息队列提供了一些未标准化但仍然有用的功能。

#### 通过命令行显示和删除消息队列对象

在第五十一章中，我们提到 POSIX IPC 对象作为虚拟文件系统中的文件实现，并且这些文件可以通过*ls*和*rm*列出和删除。为了对 POSIX 消息队列执行此操作，我们必须使用如下形式的命令来挂载消息队列文件系统：

```
# `mount -t mqueue` ``*`source`*`` *`target`*
```

*源*可以是任何名称（指定字符串*none*是典型的做法）。它的唯一意义在于，它会出现在`/proc/mounts`中，并且会被*mount*和*df*命令显示出来。*目标*是消息队列文件系统的挂载点。

以下 shell 会话展示了如何挂载消息队列文件系统并显示其内容。我们首先创建一个挂载点并进行挂载：

```
$ `su` *Privilege is required for* *mount*
Password:
# `mkdir /dev/mqueue`
# `mount -t mqueue none /dev/mqueue`
$ `exit`                                *Terminate**root* *shell session*
```

接下来，我们显示 `/proc/mounts` 中有关新挂载的记录，然后显示挂载目录的权限：

```
$ `cat /proc/mounts | grep mqueue`
none /dev/mqueue mqueue rw 0 0
$ `ls -ld /dev/mqueue`
drwxrwxrwt  2 root root 40 Jul 26 12:09 /dev/mqueue
```

从 *ls* 命令的输出中有一点需要注意，那就是消息队列文件系统会自动挂载，并为挂载目录设置了粘滞位。（我们从 *ls* 显示的其他执行权限字段中的 *t* 看出这一点。）这意味着，非特权进程只能取消链接它拥有的消息队列。

接下来，我们创建一个消息队列，使用*ls*命令查看它在文件系统中的可见性，然后删除该消息队列：

```
$ `./pmsg_create -c /newq`
$ `ls /dev/mqueue`
newq
$ `rm /dev/mqueue/newq`
```

#### 获取有关消息队列的信息

我们可以显示消息队列文件系统中文件的内容。这些虚拟文件中的每一个都包含与之关联的消息队列信息：

```
$ `./pmsg_create -c /mq` *Create a queue*
$ `./pmsg_send /mq abcdefg` *Write 7 bytes to the queue*
$ `cat /dev/mqueue/mq`
QSIZE:7       NOTIFY:0    SIGNO:0    NOTIFY_PID:0
```

`QSIZE` 字段是队列中数据的总字节数。其余字段与消息通知相关。如果 `NOTIFY_PID` 非零，则表示具有指定进程 ID 的进程已经注册了来自该队列的消息通知，其余字段提供有关通知类型的信息：

+   `NOTIFY` 是一个对应于 *sigev_notify* 常量的值：`SIGEV_SIGNAL` 为 0，`SIGEV_NONE` 为 1，或 `SIGEV_THREAD` 为 2。

+   如果通知方法是 `SIGEV_SIGNAL`，那么 `SIGNO` 字段表示为消息通知发送的信号。

以下 shell 会话展示了这些字段中出现的信息：

```
$ `./mq_notify_sig /mq &` *Notify using* SIGUSR1 *(signal 10 on x86)*
[1] 18158
$ `cat /dev/mqueue/mq`
QSIZE:7       NOTIFY:0    SIGNO:10   NOTIFY_PID:18158
$ `kill %1`
[1]   Terminated    ./mq_notify_sig /mq
$ `./mq_notify_thread /mq &` *Notify using a thread*
[2] 18160
$ `cat /dev/mqueue/mq`
QSIZE:7       NOTIFY:2    SIGNO:0    NOTIFY_PID:18160
```

#### 使用消息队列与替代 I/O 模型

在 Linux 实现中，消息队列描述符实际上是一个文件描述符。我们可以通过 I/O 多路复用系统调用（*select()* 和 *poll()*）或者 *epoll* API 来监控这个文件描述符。（更多关于这些 API 的细节，请参见第六十三章）这使我们能够避免在尝试等待来自消息队列和文件描述符的输入时，遇到与 System V 消息队列相关的难题（参见 System V 消息队列的缺点）。然而，这个特性是非标准的；SUSv3 并没有要求消息队列描述符必须实现为文件描述符。

## 消息队列限制

SUSv3 定义了两个 POSIX 消息队列的限制：

`MQ_PRIO_MAX`

我们在发送消息中描述了这个限制，它定义了消息的最大优先级。

`MQ_OPEN_MAX`

实现可以定义此限制，以表示进程可以保持打开的消息队列的最大数量。SUSv3 要求此限制至少为`_POSIX_MQ_OPEN_MAX`（8）。Linux 不定义此限制。相反，因为 Linux 将消息队列描述符实现为文件描述符（Linux 特定功能），适用的限制是文件描述符的限制。（换句话说，在 Linux 上，文件描述符和消息队列描述符的数量的每进程和系统范围限制实际上适用于文件描述符和消息队列描述符的总和。）有关适用限制的详细信息，请参见第 36.3 节中对`RLIMIT_NOFILE`资源限制的讨论。

除了上述 SUSv3 指定的限制外，Linux 还提供了一些`/proc`文件，用于查看和（在有权限的情况下）更改控制 POSIX 消息队列使用的限制。以下三个文件位于目录`/proc/sys/fs/mqueue`中：

`msg_max`

此限制指定了新消息队列的*mq_maxmsg*属性的上限（即在使用*mq_open()*创建队列时，*attr.mq_maxmsg*的上限）。该限制的默认值为 10。最小值为 1（在 Linux 2.6.28 之前的内核中为 10）。最大值由内核常量`HARD_MSGMAX`定义。该常量的值计算为（131,072 / *sizeof(void *)*），在 Linux/x86-32 上计算为 32,768。当特权进程（`CAP_SYS_RESOURCE`）调用*mq_open()*时，`msg_max`限制会被忽略，但`HARD_MSGMAX`仍然作为*attr.mq_maxmsg*的上限。

`msgsize_max`

此限制指定了未特权进程创建的新消息队列的*mq_msgsize*属性的上限（即在使用*mq_open()*创建队列时，*attr.mq_msgsize*的上限）。该限制的默认值为 8192。最小值为 128（在 Linux 2.6.28 之前的内核中为 8192）。最大值为 1,048,576（在 Linux 2.6.28 之前的内核中为`INT_MAX`）。当特权进程（`CAP_SYS_RESOURCE`）调用*mq_open()*时，此限制将被忽略。

`queues_max`

这是一个系统范围的限制，指定可以创建的消息队列的最大数量。一旦达到此限制，只有特权进程（`CAP_SYS_RESOURCE`）可以创建新的队列。该限制的默认值为 256。可以将其更改为 0 到`INT_MAX`范围内的任何值。

Linux 还提供了`RLIMIT_MSGQUEUE`资源限制，可以用于为调用进程的真实用户 ID 所属的所有消息队列所占用的空间设置上限。有关详细信息，请参见特定资源限制的详细信息。

## POSIX 与 System V 消息队列的比较

System V IPC 和 POSIX IPC 比较 列出了 POSIX IPC 接口相对于 System V IPC 接口的各种优点：POSIX IPC 接口更简洁，并且与传统的 UNIX 文件模型更一致，且 POSIX IPC 对象是引用计数的，这简化了确定何时删除对象的任务。这些一般的优点也适用于 POSIX 消息队列。

POSIX 消息队列相比于 System V 消息队列也有以下具体优点：

+   消息通知功能允许（单个）进程在消息到达先前为空的队列时，通过信号或线程实例化异步通知。

+   在 Linux（但不是其他 UNIX 实现）上，可以使用 *poll()*、*select()* 和 *epoll()* 来监视 POSIX 消息队列。System V 消息队列不提供此功能。

然而，POSIX 消息队列相比于 System V 消息队列也有一些缺点：

+   POSIX 消息队列的可移植性较差。这个问题甚至在 Linux 系统之间也存在，因为只有自内核 2.6.6 版本开始才提供消息队列支持。

+   通过类型选择 System V 消息的功能，提供了比 POSIX 消息的严格优先级排序更大的灵活性。

### 注意

POSIX 消息队列在 UNIX 系统上的实现方式差异较大。有些系统提供用户空间的实现，且在至少一个这样的实现（Solaris 10）中，*mq_open()* 手册页面明确指出，该实现不能被认为是安全的。在 Linux 上，选择内核实现消息队列的动机之一是因为认为无法提供一个安全的用户空间实现。

## 总结

POSIX 消息队列允许进程以消息的形式交换数据。每条消息都有一个关联的整数优先级，消息按优先级顺序排队（因此也按顺序接收）。

POSIX 消息队列相比于 System V 消息队列有一些优点，尤其是它们是引用计数的，并且进程可以异步通知消息到达空队列。然而，POSIX 消息队列比 System V 消息队列的可移植性差。

#### 进一步信息

[Stevens, 1999] 提供了一个 POSIX 消息队列的替代展示，并展示了使用内存映射文件的用户空间实现。POSIX 消息队列也在 [Gallmeister, 1995] 中进行了详细描述。

## 练习

1.  修改 示例 52-5 (`pmsg_receive.c`) 程序，使其接受一个超时（相对秒数）作为命令行参数，并使用 *mq_timedreceive()* 代替 *mq_receive()*。

1.  将 A Client-Server Application Using FIFOs 中的序列号客户端-服务器应用程序重新编码，改用 POSIX 消息队列。

1.  重写 A File-Server Application Using Message Queues 中的文件服务器应用程序，改用 POSIX 消息队列替代 System V 消息队列。

1.  编写一个简单的聊天程序（类似于*talk(1)*，但没有*curses*界面），使用 POSIX 消息队列。

1.  修改示例 52-6 中的程序（`mq_notify_sig.c`），以演示由*mq_notify()*建立的消息通知仅发生一次。可以通过删除`for`循环中的*mq_notify()*调用来实现。

1.  替换示例 52-6 中的信号处理程序（`mq_notify_sig.c`）为使用*sigwaitinfo()*。从*sigwaitinfo()*返回后，显示返回的*siginfo_t*结构中的值。程序如何在*sigwaitinfo()*返回的*siginfo_t*结构中获取消息队列描述符？

1.  在示例 52-7 中，*buffer*能否作为全局变量，只在主程序中分配一次内存？请解释你的答案。
