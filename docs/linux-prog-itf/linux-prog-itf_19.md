## 第十九章 监控文件事件

一些应用程序需要能够监控文件或目录，以确定被监控对象是否发生了事件。例如，一个图形化文件管理器需要能够确定当前显示的目录中是否有文件被添加或删除，或者一个守护进程可能需要监控其配置文件，以便了解文件是否已更改。

从内核 2.6.13 开始，Linux 提供了*inotify*机制，它允许应用程序监控文件事件。本章介绍了*inotify*的使用。

*inotify*机制取代了较旧的机制*dnotify*，后者提供了*inotify*部分功能的子集。本章末尾简要描述了*dnotify*，并重点解释了为何*inotify*更好。

*inotify*和*dnotify*机制是 Linux 特有的。（一些其他系统也提供类似的机制。例如，BSD 提供了*kqueue* API。）

### 注意

一些库提供了比*inotify*和*dnotify*更抽象、更具可移植性的 API。这些库的使用可能更适合某些应用程序。这些库中的一些在支持的系统上使用了*inotify*或*dnotify*。例如，FAM（文件更改监视器，[`oss.sgi.com/projects/fam/`](http://oss.sgi.com/projects/fam/)）和 Gamin（[`www.gnome.org/~veillard/gamin/`](http://www.gnome.org/~veillard/gamin/)）。

## 概述

使用*inotify* API 的关键步骤如下：

1.  应用程序使用*inotify_init()*来创建一个*inotify 实例*。此系统调用返回一个文件描述符，用于在后续操作中引用该*inotify 实例*。

1.  应用程序通过使用*inotify_add_watch()*将感兴趣的文件添加到先前创建的*inotify 实例*的监视列表中，从而告知内核哪些文件是感兴趣的。每个监视项由路径名和相关的位掩码组成。位掩码指定了要监控的路径名的事件集合。作为其函数结果，*inotify_add_watch()*返回一个*watch descriptor*，该描述符用于在后续操作中引用该监视项。（*inotify_rm_watch()*系统调用执行相反的任务，移除先前添加到*inotify 实例*中的监视项。）

1.  为了获得事件通知，应用程序对*inotify*文件描述符执行*read()*操作。每次成功的*read()*返回一个或多个*inotify_event*结构体，每个结构体包含有关通过此*inotify 实例*监控的路径名上发生的事件的信息。

1.  当应用程序完成监控时，它会关闭*inotify*文件描述符。这会自动移除所有与该*inotify 实例*关联的监视项。

*inotify*机制可用于监控文件或目录。当监控一个目录时，应用程序将收到该目录本身及其内部文件的事件通知。

*inotify*监视机制不是递归的。如果应用程序想要监视整个目录子树中的事件，它必须为树中的每个目录发出*inotify_add_watch()*调用。

可以使用*select()*、*poll()*、*epoll*以及从 Linux 2.6.25 开始的信号驱动 I/O 来监视*inotify*文件描述符。如果有事件可供读取，这些接口将表示*inotify*文件描述符是可读的。有关这些接口的更多细节，请参见第六十三章。

### 注意

*inotify*机制是一个可选的 Linux 内核组件，通过选项`CONFIG_INOTIFY`和`CONFIG_INOTIFY_USER`进行配置。

## *inotify* API

*inotify_init()*系统调用创建一个新的*inotify*实例。

```
#include <sys/inotify.h>

int `inotify_init`(void);
```

### 注意

成功时返回文件描述符，出错时返回-1

*inotify_init()*的函数结果返回一个文件描述符。这个文件描述符是后续操作中用于引用*inotify*实例的句柄。

### 注意

从内核 2.6.27 开始，Linux 支持一个新的非标准系统调用，*inotify_init1()*。该系统调用与*inotify_init()*执行相同的任务，但提供了一个额外的参数*flags*，可以用来修改系统调用的行为。支持两个标志。`IN_CLOEXEC`标志使内核为新文件描述符启用 close-on-exec 标志（`FD_CLOEXEC`）。此标志的作用与文件描述符号由*open()*返回返回的文件描述符号")中描述的*open()* `O_CLOEXEC`标志相同。`IN_NONBLOCK`标志使内核在底层的打开文件描述符上启用`O_NONBLOCK`标志，从而使以后的读取操作变为非阻塞。这可以避免额外调用*fcntl()*来实现相同的效果。

*inotify_add_watch()*系统调用将新的监视项添加到*inotify*实例的监视列表中，或者修改已存在的监视项，具体取决于文件描述符*fd*所引用的*inotify*实例。（参见图 19-1。）

```
#include <sys/inotify.h>

int `inotify_add_watch`(int *fd*, const char **pathname*, uint32_t *mask*);
```

### 注意

成功时返回监视描述符，出错时返回-1

![An inotify instance and associated kernel data structures](img/19-1_INOTIFY-inotify-instance-scale90.png.jpg)图 19-1. *inotify*实例及其关联的内核数据结构

*pathname*参数标识要创建或修改监视项的文件。调用者必须对该文件具有读取权限。（文件权限检查仅在*inotify_add_watch()*调用时执行一次。只要监视项继续存在，即使文件权限稍后发生更改，导致调用者不再拥有读取权限，调用者仍会继续接收文件通知。）

*mask*参数是一个位掩码，用于指定要监视的*pathname*的事件。我们将在稍后讨论可以在*mask*中指定的位值。

如果*pathname*之前没有被添加到*fd*的监视列表中，则*inotify_add_watch()*会在列表中创建一个新的监视项，并返回一个新的非负监视描述符，该描述符用于在后续操作中引用该监视项。此监视描述符对于该*inotify*实例是唯一的。

如果*pathname*之前已经被添加到*fd*的监视列表中，则*inotify_add_watch()*会修改现有监视项的*mask*，并返回该监视项的监视描述符。（此监视描述符将与最初添加*pathname*到此监视列表时返回的监视描述符相同。）我们将在下一节描述`IN_MASK_ADD`标志时进一步讨论*mask*如何被修改。

*inotify_rm_watch()*系统调用从由文件描述符*fd*引用的*inotify*实例中移除由*wd*指定的监视项。

```
#include <sys/inotify.h>

int `inotify_rm_watch`(int *fd*, int *wd*);
```

### 注意

成功时返回 0，错误时返回-1

*wd*参数是先前调用*inotify_add_watch()*返回的监视描述符。

移除监视会生成一个`IN_IGNORED`事件用于此监视描述符。我们将在稍后讨论这个事件。

## *inotify*事件

当我们使用*inotify_add_watch()*创建或修改一个监视项时，*mask*位掩码参数标识了要监视的给定*pathname*的事件。可以在*mask*中指定的事件位由表 19-1 的*输入*列指示。

表 19-1. *inotify*事件

| 位值 | 输入 | 输出 | 描述 |
| --- | --- | --- | --- |
| `IN_ACCESS` | • | • | 文件被访问（*read()*） |
| `IN_ATTRIB` | • | • | 文件元数据已更改 |
| `IN_CLOSE_WRITE` | • | • | 打开用于写入的文件被关闭 |
| `IN_CLOSE_NOWRITE` | • | • | 只读打开的文件被关闭 |
| `IN_CREATE` | • | • | 文件/目录在被监视的目录内创建 |
| `IN_DELETE` | • | • | 文件/目录从被监视的目录中删除 |
| `IN_DELETE_SELF` | • | • | 被监视的文件/目录本身被删除 |
| `IN_MODIFY` | • | • | 文件已被修改 |
| `IN_MOVE_SELF` | • | • | 被监视的文件/目录本身被移动 |
| `IN_MOVED_FROM` | • | • | 文件从被监视的目录中移动 |
| `IN_MOVED_TO` | • | • | 文件被移动到被监视的目录 |
| `IN_OPEN` | • | • | 文件被打开 |
| `IN_ALL_EVENTS` | • |   | 所有上述输入事件的简写 |
| `IN_MOVE` | • |   | `IN_MOVED_FROM &#124; IN_MOVED_TO`的简写 |
| `IN_CLOSE` | • |   | `IN_CLOSE_WRITE &#124; IN_CLOSE_NOWRITE`的简写 |
| `IN_DONT_FOLLOW` | • |   | 不要解引用符号链接（自 Linux 2.6.15 起） |
| `IN_MASK_ADD` | • |   | 将事件添加到当前*pathname*的监视掩码 |
| `IN_ONESHOT` | • |   | 仅监视*pathname*的一个事件 |
| `IN_ONLYDIR` | • |   | 如果*pathname*不是一个目录，则失败（自 Linux 2.6.15 起） |
| `IN_IGNORED` |   | • | 监视项被应用程序或内核移除 |
| `IN_ISDIR` |   | • | *name*中返回的文件名是一个目录 |
| `IN_Q_OVERFLOW` |   | • | 事件队列溢出 |
| `IN_UNMOUNT` |   | • | 包含对象的文件系统已被卸载 |

表 19-1 中大多数位的含义从它们的名称中可以看出。以下列表澄清了一些细节：

+   `IN_ATTRIB`事件发生在文件元数据（如权限、所有权、链接计数、扩展属性、用户 ID 或组 ID）发生更改时。

+   `IN_DELETE_SELF`事件发生在被监控的对象（即文件或目录）被删除时。`IN_DELETE`事件发生在监控的对象是目录，并且目录中包含的某个文件被删除时。

+   `IN_MOVE_SELF`事件发生在被监控的对象被重命名时。`IN_MOVED_FROM`和`IN_MOVED_TO`事件发生在对象在监控的目录内被重命名时。前一个事件发生在包含旧名称的目录，后一个事件发生在包含新名称的目录。

+   `IN_DONT_FOLLOW`、`IN_MASK_ADD`、`IN_ONESHOT`和`IN_ONLYDIR`位不指定要监控的事件。相反，它们控制*inotify_add_watch()*调用的操作。

+   `IN_DONT_FOLLOW`指定如果*pathname*是符号链接，则不应对其进行解引用。这允许应用程序监控符号链接，而不是它所指向的文件。

+   如果我们执行一个指定已被通过此*inotify*文件描述符监视的*pathname*的`inotify_add_watch()`调用，则默认情况下，给定的*mask*将替换此监视项的当前掩码。如果指定了`IN_MASK_ADD`，则当前掩码将通过与*mask*中给定的值进行或运算来修改。

+   `IN_ONESHOT`允许应用程序仅监控*pathname*的一个事件。该事件发生后，监视项会自动从监视列表中移除。

+   `IN_ONLYDIR`允许应用程序仅在*pathname*是目录时才进行监控。如果*pathname*不是一个目录，则`inotify_add_watch()`会因为错误`ENOTDIR`而失败。使用此标志可以防止在确保监控的是目录时可能发生的竞争条件。

## 读取*inotify*事件

在注册了监视项的情况下，应用程序可以通过使用*read()*从*inotify*文件描述符中读取事件来确定哪些事件已经发生。如果到目前为止没有事件发生，则*read()*会阻塞，直到事件发生（除非为文件描述符设置了`O_NONBLOCK`状态标志，在这种情况下，如果没有事件可用，*read()*会立即失败并返回错误`EAGAIN`）。

在事件发生后，每次 *read()* 调用返回一个缓冲区（参见 图 19-2），该缓冲区包含一个或多个如下类型的结构：

```
struct inotify_event {
    int      wd;          /* Watch descriptor on which event occurred */
    uint32_t mask;        /* Bits describing event that occurred */
    uint32_t cookie;      /* Cookie for related events (for rename()) */
    uint32_t len;         /* Size of 'name' field */
    char     name[];      /* Optional null-terminated filename */
};
```

![一个包含三个 inotify_event 结构的输入缓冲区](img/19-2_INOTIFY-inotify_event-buffer-scale90.png.jpg)图 19-2. 一个包含三个 *inotify_event* 结构的输入缓冲区

*wd* 字段告诉我们发生此事件的监视描述符。该字段包含先前调用 *inotify_add_watch()* 时返回的值。当应用程序通过相同的 *inotify* 文件描述符监视多个文件或目录时，*wd* 字段非常有用。它提供了一个链接，允许应用程序确定发生事件的特定文件或目录。（为此，应用程序必须维护一个书籍数据结构，将监视描述符与路径名关联起来。）

*mask* 字段返回一个描述事件的位掩码。*mask* 中可能出现的位范围通过 表 19-1 的*Out*列表示。请注意以下关于特定位的额外细节：

+   当移除监视时，会生成一个 `IN_IGNORED` 事件。这可能有两个原因：应用程序使用了 *inotify_rm_watch()* 调用显式移除了监视，或者监视由于被监控对象被删除或所在文件系统被卸载而被内核隐式移除。当使用 `IN_ONESHOT` 建立的监视被触发事件后自动移除时，不会生成 `IN_IGNORED` 事件。

+   如果事件的主题是一个目录，那么，除了其他一些位，`IN_ISDIR` 位还会在*mask*中被设置。

+   `IN_UNMOUNT` 事件通知应用程序，包含被监控对象的文件系统已经被卸载。该事件发生后，会发送一个包含 `IN_IGNORED` 位的后续事件。

+   我们在 队列限制和 `/proc` 文件中描述了 `IN_Q_OVERFLOW`，该部分讨论了关于排队的 *inotify* 事件的限制。

*cookie* 字段用于将相关事件关联在一起。目前，只有在文件被重命名时，该字段才会被使用。当文件重命名时，会为源目录生成一个 `IN_MOVED_FROM` 事件，然后为目标目录生成一个 `IN_MOVED_TO` 事件。（如果文件在同一目录中被赋予新名称，那么这两个事件会出现在同一目录中。）这两个事件将在它们的 *cookie* 字段中拥有相同的唯一值，从而使应用程序能够将它们关联起来。

当监控目录中的文件发生事件时，*name*字段用于返回一个空字符终止的字符串，标识该文件。如果事件发生在被监控对象本身上，则*name*字段不使用，而*len*字段将包含 0。

*len*字段表示实际分配给*name*字段的字节数。这个字段是必要的，因为在*name*中存储的字符串的结尾与*read()*返回的缓冲区中下一个*inotify_event*结构的开始之间，可能会有额外的填充字节（请参见图 19-2）。因此，单个*inotify*事件的长度是*sizeof(struct inotify_event) + len*。

如果传递给*read()*的缓冲区太小，无法容纳下一个*inotify_event*结构，则*read()*会因错误`EINVAL`而失败，以警告应用程序此事实。（在 2.6.21 之前的内核中，*read()*在这种情况下返回 0。使用`EINVAL`错误可以更清晰地表明发生了编程错误。）应用程序可以通过使用更大的缓冲区再次调用*read()*来响应。然而，可以通过确保缓冲区始终足够大，以至少容纳一个事件，从而完全避免这个问题：传递给*read()*的缓冲区应该至少为*(sizeof(struct inotify_event) + NAME_MAX + 1)*字节，其中`NAME_MAX`是文件名的最大长度，再加一个终止的空字节。

使用比最小值更大的缓冲区大小可以让应用程序通过一次*read()*高效地检索多个事件。从*inotify*文件描述符读取的*read()*将返回可用事件的数量和可以适配提供的缓冲区的事件数量中的最小值。

### 注意

调用*ioctl(fd, FIONREAD, &numbytes)*会返回当前可以从文件描述符*fd*所引用的*inotify*实例中读取的字节数。

从*inotify*文件描述符读取的事件形成一个有序的队列。因此，例如，可以保证在文件被重命名时，`IN_MOVED_FROM`事件会在`IN_MOVED_TO`事件之前被读取。

当将一个新事件添加到事件队列的末尾时，如果两个事件的*wd*、*mask*、*cookie*和*name*值相同，内核会将新事件与队列尾部的事件合并（因此新事件实际上并不会被排入队列）。这样做是因为许多应用程序不需要知道同一事件的重复实例，丢弃多余的事件减少了事件队列所需的（内核）内存。然而，这也意味着我们不能使用*inotify*来可靠地确定一个重复事件发生的次数或频率。

#### 示例程序

尽管前面的描述包含了很多细节，但 *inotify* API 实际上非常简单易用。示例 19-1 演示了 *inotify* 的使用。

示例 19-1. 使用 *inotify* API

```
`inotify/demo_inotify.c`
    #include <sys/inotify.h>
    #include <limits.h>
    #include "tlpi_hdr.h"

    static void             /* Display information from inotify_event structure */
    displayInotifyEvent(struct inotify_event *i)
    {
        printf("    wd =%2d; ", i->wd);
        if (i->cookie > 0)
            printf("cookie =%4d; ", i->cookie);

        printf("mask = ");
        if (i->mask & IN_ACCESS)        printf("IN_ACCESS ");
        if (i->mask & IN_ATTRIB)        printf("IN_ATTRIB ");
        if (i->mask & IN_CLOSE_NOWRITE) printf("IN_CLOSE_NOWRITE ");
        if (i->mask & IN_CLOSE_WRITE)   printf("IN_CLOSE_WRITE ");
        if (i->mask & IN_CREATE)        printf("IN_CREATE ");
        if (i->mask & IN_DELETE)        printf("IN_DELETE ");
        if (i->mask & IN_DELETE_SELF)   printf("IN_DELETE_SELF ");
        if (i->mask & IN_IGNORED)       printf("IN_IGNORED ");
        if (i->mask & IN_ISDIR)         printf("IN_ISDIR ");
        if (i->mask & IN_MODIFY)        printf("IN_MODIFY ");
        if (i->mask & IN_MOVE_SELF)     printf("IN_MOVE_SELF ");
        if (i->mask & IN_MOVED_FROM)    printf("IN_MOVED_FROM ");
        if (i->mask & IN_MOVED_TO)      printf("IN_MOVED_TO ");
        if (i->mask & IN_OPEN)          printf("IN_OPEN ");
        if (i->mask & IN_Q_OVERFLOW)    printf("IN_Q_OVERFLOW ");
        if (i->mask & IN_UNMOUNT)       printf("IN_UNMOUNT ");
        printf("\n");

        if (i->len > 0)
            printf("        name = %s\n", i->name);
    }

    #define BUF_LEN (10 * (sizeof(struct inotify_event) + NAME_MAX + 1))

    int
    main(int argc, char *argv[])
    {
        int inotifyFd, wd, j;
        char buf[BUF_LEN];
        ssize_t numRead;
        char *p;
        struct inotify_event *event;

        if (argc < 2 || strcmp(argv[1], "--help") == 0)
            usageErr("%s pathname... \n", argv[0]);

     inotifyFd = inotify_init();                 /* Create inotify instance */
        if (inotifyFd == -1)
            errExit("inotify_init");

        for (j = 1; j < argc; j++) {
         wd = inotify_add_watch(inotifyFd, argv[j], IN_ALL_EVENTS);
            if (wd == -1)
                errExit("inotify_add_watch");

            printf("Watching %s using wd %d\n", argv[j], wd);
        }

        for (;;) {                                  /* Read events forever */
         numRead = read(inotifyFd, buf, BUF_LEN);
            if (numRead == 0)
                fatal("read() from inotify fd returned 0!");

            if (numRead == -1)
                errExit("read");

            printf("Read %ld bytes from inotify fd\n", (long) numRead);

            /* Process all of the events in buffer returned by read() */

            for (p = buf; p < buf + numRead; ) {
                event = (struct inotify_event *) p;
             displayInotifyEvent(event);

                p += sizeof(struct inotify_event) + event->len;
            }
        }

        exit(EXIT_SUCCESS);
    }
         `inotify/demo_inotify.c`
```

程序在示例 19-1 中执行以下步骤：

+   使用 *inotify_init()* 创建一个 *inotify* 文件描述符 ![](img/U001.png)。

+   使用 *inotify_add_watch()* 为程序命令行参数中指定的每个文件添加一个监视项 ![](img/U002.png)。每个监视项都会监视所有可能的事件。

+   执行一个无限循环，其中：

    +   从 *inotify* 文件描述符读取一个事件缓冲区 ![](img/U003.png)。

    +   调用 *displayInotifyEvent()* 函数显示该缓冲区中每个 *inotify_event* 结构的内容 ![](img/U004.png)。

以下 shell 会话演示了示例 19-1 中程序的使用。我们启动一个后台运行的程序实例，监控两个目录：

```
$ `./demo_inotify dir1 dir2 &`
[1] 5386
Watching dir1 using wd 1
Watching dir2 using wd 2
```

然后我们执行生成事件的命令，在两个目录中创建一个文件，使用 *cat(1)*：

```
$`cat > dir1/aaa`
Read 64 bytes from inotify fd
    wd = 1; mask = IN_CREATE
        name = aaa
    wd = 1; mask = IN_OPEN
        name = aaa
```

上述由后台程序生成的输出显示，*read()* 获取了一个包含两个事件的缓冲区。我们继续为文件输入一些内容，然后输入终端的 *end-of-file* 字符：

```
`Hello world`
Read 32 bytes from inotify fd
    wd = 1; mask = IN_MODIFY
        name = aaa
*Type Control-D*
Read 32 bytes from inotify fd
    wd = 1; mask = IN_CLOSE_WRITE
        name = aaa
```

然后，我们将文件重命名到另一个监控的目录中。这会产生两个事件，一个是来自文件移动的源目录（监视描述符 1），另一个是目标目录（监视描述符 2）：

```
$ `mv dir1/aaa dir2/bbb`
Read 64 bytes from inotify fd
    wd = 1; cookie = 548; mask = IN_MOVED_FROM
        name = aaa
    wd = 2; cookie = 548; mask = IN_MOVED_TO
        name = bbb
```

这两个事件共享相同的*cookie*值，从而允许应用程序将它们关联起来。

当我们在监控的目录之一下创建一个子目录时，生成的事件的掩码中包含 `IN_ISDIR` 位，表明事件的主体是一个目录：

```
$ `mkdir dir2/ddd`
Read 32 bytes from inotify fd
    wd = 1; mask = IN_CREATE IN_ISDIR
        name = ddd
```

在此需要重复说明的是，*inotify* 监控并不是递归的。如果应用程序想要监控新创建的子目录中的事件，它需要再次调用 *inotify_add_watch()*，并指定子目录的路径名。

最后，我们移除其中一个被监控的目录：

```
$ `rmdir dir1`
Read 32 bytes from inotify fd
    wd = 1; mask = IN_DELETE_SELF
    wd = 1; mask = IN_IGNORED
```

最后一个事件 `IN_IGNORED` 被生成，以通知应用程序内核已将此监视项从监视列表中移除。

## 队列限制和 `/proc` 文件

排队 *inotify* 事件需要内核内存。因此，内核对 *inotify* 机制的操作设置了各种限制。超级用户可以通过 `/proc/sys/fs/inotify` 目录中的三个文件来配置这些限制：

`max_queued_events`

当调用 *inotify_init()* 时，该值用于设置新 *inotify* 实例上可以排队的事件数量的上限。如果达到该限制，则会生成一个 `IN_Q_OVERFLOW` 事件，且多余的事件会被丢弃。溢出事件的 *wd* 字段值为 -1。

`max_user_instances`

这是针对每个真实用户 ID 可以创建的 *inotify* 实例数量的限制。

`max_user_watches`

这是针对每个真实用户 ID 可以创建的监视项数量的限制。

这三个文件的典型默认值分别是 16,384、128 和 8192。

## 一个旧的文件事件监控系统：*dnotify*

Linux 提供了另一种监控文件事件的机制。这个机制叫做 *dnotify*，自内核 2.4 以来就已存在，但已被 *inotify* 取代。与 *inotify* 相比，*dnotify* 机制存在一些限制：

+   *dnotify* 机制通过向应用程序发送信号来通知事件。使用信号作为通知机制使应用程序设计变得复杂（信号的进程间通信）。这也使得在库中使用 *dnotify* 变得困难，因为调用程序可能会改变通知信号的处理方式。*inotify* 机制不使用信号。

+   *dnotify* 的监控单元是一个目录。当在该目录中的任何文件上执行操作时，应用程序会收到通知。相比之下，*inotify* 可以用于监控目录或单个文件。

+   为了监控一个目录，*dnotify* 要求应用程序为该目录打开一个文件描述符。使用文件描述符会导致两个问题。首先，因为文件系统忙碌，包含该目录的文件系统无法卸载。其次，因为每个目录都需要一个文件描述符，应用程序可能会消耗大量的文件描述符。而 *inotify* 不使用文件描述符，从而避免了这些问题。

+   *dnotify* 提供的文件事件信息比 *inotify* 提供的信息不够精确。当一个文件在监控的目录中发生变化时，*dnotify* 会告诉我们发生了事件，但不会告诉我们哪个文件参与了该事件。应用程序必须通过缓存目录内容的信息来确定这一点。此外，*inotify* 提供比 *dnotify* 更详细的事件类型信息。

+   在某些情况下，*dnotify* 无法提供可靠的文件事件通知。

有关 *dnotify* 的更多信息可以在 *fcntl(2)* 手册页中 `F_NOTIFY` 操作的描述中找到，也可以在内核源文件 `Documentation/dnotify.txt` 中查阅。

## 总结

Linux 特有的 *inotify* 机制允许应用程序在一组被监控的文件和目录上发生事件（文件被打开、关闭、创建、删除、修改、重命名等）时获取通知。*inotify* 机制取代了较旧的 *dnotify* 机制。

## 练习

1.  编写一个程序，记录在命令行参数指定的目录下的所有文件创建、删除和重命名操作。该程序应监控指定目录下所有子目录中的事件。为了获取所有这些子目录的列表，您需要使用 *nftw()* 函数（文件树遍历：*nftw()*")）。当树下添加新子目录或删除目录时，监控的子目录集合应相应更新。
