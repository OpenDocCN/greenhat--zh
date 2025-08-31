## 第六十一章 套接字：高级主题

本章讨论与套接字编程相关的一些更高级的主题，包括以下内容：

+   流套接字上发生部分读取和写入的情况；

+   使用*shutdown()*来关闭两个已连接套接字之间的单向通道；

+   *recv()*和*send()* I/O 系统调用，它们提供了套接字特有的功能，这是*read()*和*write()*所不具备的；

+   *sendfile()*系统调用，在某些情况下用于高效地在套接字上传输数据；

+   TCP 协议的操作细节，目的是消除一些常见的误解，这些误解可能导致在编写使用 TCP 套接字的程序时出现错误；

+   使用*netstat*和*tcpdump*命令来监视和调试使用套接字的应用程序；

+   使用*getsockopt()*和*setsockopt()*系统调用来检索和修改影响套接字操作的选项。

我们还讨论了一些其他较小的主题，并以总结一些高级套接字功能作为本章的结束。

## 流套接字的部分读取和写入

当我们首次在第四章介绍*read()*和*write()*系统调用时，我们提到在某些情况下，它们可能传输的字节数少于请求的字节数。部分传输可以在流套接字上进行 I/O 时发生。我们现在考虑为什么它们会发生，并展示一对透明处理部分传输的函数。

如果套接字中可用的字节数少于*read()*调用请求的字节数，则可能发生部分读取。在这种情况下，*read()*仅返回可用的字节数。（这与我们在管道和 FIFO 的*read()*和*write()*语义和 write()语义")中看到的行为相同。）

如果缓冲区空间不足以传输所有请求的字节，并且以下之一为真，则可能发生部分写入：

+   信号处理程序在*write()*调用之后中断了操作（系统调用的中断与重启），该调用已经传输了部分请求的字节。

+   套接字在非阻塞模式下操作（`O_NONBLOCK`），并且只能传输部分请求的字节。

+   在仅传输了部分请求字节后发生了*异步错误*。这里所说的*异步错误*是指与应用程序使用套接字 API 调用无关的错误。例如，由于 TCP 连接出现问题，可能是由于对等应用程序崩溃所致。

在上述所有情况中，假设至少有 1 个字节可以传输，*write()* 将成功，并返回传输到输出缓冲区的字节数。

如果发生部分 I/O，例如，如果 *read()* 返回的字节数少于请求的字节数，或被信号处理程序中断的阻塞 *write()* 仅传输了部分请求的数据，那么有时重新启动系统调用以完成传输是有用的。在示例 61-1 和 writen() 的实现")中，我们提供了两个完成这一任务的函数：*readn()* 和 *writen()*。（这些函数的思路来源于[Stevens 等，2004]中的同名函数。）

```
#include "rdwrn.h"

ssize_t `readn`(int *fd*, void **buffer*, size_t *count*);
```

### 注意

返回读取的字节数，文件结束时返回 0，出错时返回 -1

```
ssize_t `writen`(int fd, void **buffer*, size_t *count*);
```

### 注意

返回写入的字节数，如果出错则返回 -1

*readn()* 和 *writen()* 函数与 *read()* 和 *write()* 函数具有相同的参数。它们通过使用循环重新启动这些系统调用，从而确保请求的字节数总是被传输（除非发生错误或 *read()* 检测到文件结束）。

示例 61-1. *readn()* 和 *writen()* 的实现

```
`sockets/rdwrn.c`
#include <unistd.h>
#include <errno.h>
#include "rdwrn.h"                      /* Declares readn() and writen() */

ssize_t
readn(int fd, void *buffer, size_t n)
{
    ssize_t numRead;                    /* # of bytes fetched by last read() */
    size_t totRead;                     /* Total # of bytes read so far */
    char *buf;

    buf = buffer;                       /* No pointer arithmetic on "void *" */
    for (totRead = 0; totRead < n; ) {
        numRead = read(fd, buf, n - totRead);

        if (numRead == 0)               /* EOF */
            return totRead;             /* May be 0 if this is first read() */
        if (numRead == -1) {
            if (errno == EINTR)
                continue;               /* Interrupted --> restart read() */
            else
                return -1;              /* Some other error */
        }
        totRead += numRead;
        buf += numRead;
    }
    return totRead;                     /* Must be 'n' bytes if we get here */
}

ssize_t
writen(int fd, const void *buffer, size_t n)
{
    ssize_t numWritten;                 /* # of bytes written by last write() */
    size_t totWritten;                  /* Total # of bytes written so far */
    const char *buf;

    buf = buffer;                       /* No pointer arithmetic on "void *" */
    for (totWritten = 0; totWritten < n; ) {
        numWritten = write(fd, buf, n - totWritten);

        if (numWritten <= 0) {
            if (numWritten == -1 && errno == EINTR)
                continue;               /* Interrupted --> restart write() */
            else
                return -1;              /* Some other error */
        }
        totWritten += numWritten;
        buf += numWritten;
    }
    return totWritten;                  /* Must be 'n' bytes if we get here */
}
      `sockets/rdwrn.c`
```

## *shutdown()* 系统调用

对套接字调用 *close()* 会关闭双向通信通道的两个端点。有时，关闭连接的一端是有用的，这样数据就可以只通过套接字单向传输。*shutdown()* 系统调用提供了这一功能。

```
#include <sys/socket.h>

int `shutdown`(int *sockfd*, int *how*);
```

### 注意

成功时返回 0，出错时返回 -1

*shutdown()* 系统调用根据 *how* 的值来关闭套接字 *sockfd* 的一个或两个通道，*how* 的值可以是以下之一：

`SHUT_RD`

关闭连接的读取端。后续的读取将返回文件结束标志（0）。数据仍然可以写入套接字。在 UNIX 域流套接字上执行 `SHUT_RD` 后，对端应用程序如果继续写入对端套接字，将会收到 `SIGPIPE` 信号，并且出现 `EPIPE` 错误。如在对 TCP 套接字调用 *shutdown()*")中讨论，`SHUT_RD` 对于 TCP 套接字并无实际意义。

`SHUT_WR`

关闭连接的写半部分。一旦对等应用程序读取完所有待处理数据，它将看到文件结束标志。后续写入本地套接字会触发`SIGPIPE`信号和`EPIPE`错误。对等方写入的数据仍然可以从套接字读取。换句话说，这个操作使我们能够向对等方发送文件结束标志，同时仍能读取对等方发送回我们的数据。`SHUT_WR`操作被*ssh*和*rsh*等程序使用（参见操作符号链接：*symlink()*和*readlink()*和 readlink()")，[Stevens, 1994]）。`SHUT_WR`操作是*shutdown()*最常见的用途，有时也被称为*套接字半关闭*。

`SHUT_RDWR`

关闭连接的读写两半。这等同于先执行`SHUT_RD`，然后执行`SHUT_WR`。

除了*how*参数的语义外，*shutdown()*与*close()*在另一个重要方面有所不同：无论是否存在其他文件描述符引用该套接字，*shutdown()*都会关闭套接字通道。（换句话说，*shutdown()*是在操作打开的文件描述，而不是文件描述符。请参见图 5-1，在独占创建文件一节。）例如，假设*sockfd*指向一个已连接的流套接字。如果我们按以下方式调用，那么连接将保持打开，并且我们仍然可以通过文件描述符*fd2*在该连接上执行 I/O 操作：

```
fd2 = dup(sockfd);
close(sockfd);
```

然而，如果我们按以下顺序进行调用，那么连接的两个通道都会关闭，且无法通过*fd2*进行 I/O 操作：

```
fd2 = dup(sockfd);
shutdown(sockfd, SHUT_RDWR);
```

如果在执行*fork()*时，套接字的文件描述符被复制，情况也类似。如果在*fork()*之后，一个进程在其副本上执行了`SHUT_RDWR`，那么另一个进程也将无法在其描述符上执行 I/O 操作。

请注意，即使*how*参数指定为`SHUT_RDWR`，*shutdown()*也不会关闭文件描述符。要关闭文件描述符，我们还需要额外调用*close()*。

#### 示例程序

示例 61-2 演示了*shutdown()*的`SHUT_WR`操作。该程序是*回声*服务的 TCP 客户端。（我们在第 60.3 节中介绍了*回声*服务的 TCP 服务器。）为了简化实现，我们使用了互联网域套接字库中显示的函数。

### 注意

在某些 Linux 发行版中，*echo* 服务默认未启用，因此在运行 示例 61-2 前，我们必须先启用它。通常，该服务由 *inetd(8)* 守护进程（The *inetd* (Internet Superserver) Daemon Daemon")）内部实现。为了启用 *echo* 服务，我们必须编辑文件 `/etc/inetd.conf`，取消注释与 UDP 和 TCP *echo* 服务相关的两行（参见 示例 60-5，以及 The `/etc/inetd.conf` 文件），然后向 *inetd* 守护进程发送 `SIGHUP` 信号。

许多发行版提供了更现代的 *xinetd(8)* 代替 *inetd(8)*。有关如何在 *xinetd* 中进行等效更改的信息，请参考 *xinetd* 文档。

作为其唯一的命令行参数，程序接受运行 *echo* 服务器的主机名称。客户端执行 *fork()*，产生父进程和子进程。

客户端父进程将标准输入的内容写入套接字，以便 *echo* 服务器可以读取。当父进程检测到标准输入的文件结束时，它使用 *shutdown()* 来关闭其套接字的写入半部分。这会导致 *echo* 服务器看到文件结束，此时它关闭其套接字（这反过来会导致客户端子进程看到文件结束）。然后父进程终止。

客户端子进程从套接字读取 *echo* 服务器的响应，并将响应回显到标准输出。当它在套接字上看到文件结束时，子进程终止。

以下显示了运行此程序时我们看到的示例：

```
$ `cat > tell-tale-heart.txt`                           *Create a file for testing*
`It is impossible to say how the idea entered my brain;`
`but once conceived, it haunted me day and night.`
*Type Control-D*
$ `./is_echo_cl tekapo < tell-tale-heart.txt`
It is impossible to say how the idea entered my brain;
but once conceived, it haunted me day and night.
```

示例 61-2. *echo* 服务的客户端

```
`sockets/is_echo_cl.c`
#include "inet_sockets.h"
#include "tlpi_hdr.h"

#define BUF_SIZE 100

int
main(int argc, char *argv[])
{
    int sfd;
    ssize_t numRead;
    char buf[BUF_SIZE];

    if (argc != 2 || strcmp(argv[1], "--help") == 0)
        usageErr("%s host\n", argv[0]);

    sfd = inetConnect(argv[1], "echo", SOCK_STREAM);
    if (sfd == -1)
        errExit("inetConnect");

    switch (fork()) {
    case -1:
        errExit("fork");

    case 0:             /* Child: read server's response, echo on stdout */
        for (;;) {
            numRead = read(sfd, buf, BUF_SIZE);
            if (numRead <= 0)           /* Exit on EOF or error */
                break;
            printf("%.*s", (int) numRead, buf);
        }
        exit(EXIT_SUCCESS);

    default:            /* Parent: write contents of stdin to socket */
        for (;;) {
            numRead = read(STDIN_FILENO, buf, BUF_SIZE);
            if (numRead <= 0)           /* Exit loop on EOF or error */
                break;
            if (write(sfd, buf, numRead) != numRead)
                fatal("write() failed");
        }

        /* Close writing channel, so server sees EOF */

        if (shutdown(sfd, SHUT_WR) == -1)
            errExit("shutdown");
        exit(EXIT_SUCCESS);
    }
}
      `sockets/is_echo_cl.c`
```

## 套接字特定的 I/O 系统调用：*recv()* 和 *send()*

*recv()* 和 *send()* 系统调用在连接的套接字上执行 I/O 操作。它们提供了传统的 *read()* 和 *write()* 系统调用所不具备的套接字特定功能。

```
#include <sys/socket.h>

ssize_t `recv`(int *sockfd*, void **buffer*, size_t *length*, int *flags*);
```

### 注意

返回接收到的字节数，文件结束时返回 0，出错时返回 -1。

```
ssize_t `send`(int *sockfd*, const void **buffer*, size_t *length*, int *flags*);
```

### 注意

返回发送的字节数，出错时返回 -1。

*recv()* 和 *send()* 的返回值及前三个参数与 *read()* 和 *write()* 相同。最后一个参数 *flags* 是一个位掩码，修改 I/O 操作的行为。对于 *recv()*，可以在 *flags* 中进行或运算的位包括以下内容：

`MSG_DONTWAIT`

执行非阻塞的 *recv()*。如果没有数据可用，则不阻塞，而是立即返回错误 `EAGAIN`。我们可以通过使用 *fcntl()* 设置套接字的非阻塞模式（`O_NONBLOCK`）来获得相同的行为，区别在于 `MSG_DONTWAIT` 允许我们在每次调用时控制非阻塞行为。

`MSG_OOB`

在套接字上接收带外数据。我们在带外数据中简要描述了此功能。

`MSG_PEEK`

从套接字缓冲区中获取请求的字节的副本，但不将其实际从缓冲区中移除。数据可以通过另一次 *recv()* 或 *read()* 调用重新读取。

`MSG_WAITALL`

通常，*recv()* 调用返回请求的字节数（*length*）和套接字中实际可用字节数中较小的值。指定 `MSG_WAITALL` 标志会使系统调用阻塞，直到接收到 *length* 字节。然而，即使指定了此标志，如果发生以下情况，调用可能会返回比请求的字节数少的字节数：（a）捕获到信号；（b）流套接字的对端终止了连接；（c）遇到带外数据字节（带外数据）；（d）来自数据报套接字的接收消息少于 *length* 字节；或（e）套接字发生错误。（`MSG_WAITALL` 标志可以替代我们在示例 61-1 and writen()")中展示的 *readn()* 函数，区别在于我们的 *readn()* 函数会在信号处理程序中断时重新启动。）

上述所有标志在 SUSv3 中都有说明，除了 `MSG_DONTWAIT`，但它在某些其他 UNIX 实现中仍然可用。`MSG_WAITALL` 标志是后来的添加到套接字 API 中，在一些旧版本的实现中没有此标志。

对于 *send()*，*flags* 中可以进行按位或的位包括以下内容：

`MSG_DONTWAIT`

执行非阻塞 *send()*。如果数据无法立即传输（因为套接字发送缓冲区已满），则不会阻塞，而是返回错误 `EAGAIN`。与 *recv()* 一样，通过为套接字设置 `O_NONBLOCK` 标志也可以实现相同的效果。

`MSG_MORE`（自 Linux 2.4.4 起）

此标志与 TCP 套接字一起使用，达到与 `TCP_CORK` 套接字选项相同的效果（*sendfile()* 系统调用 System Call")），其区别在于它提供了按调用次数为单位的数据“塞栓”。自 Linux 2.6 起，此标志也可以与数据报套接字一起使用，其含义不同。在连续的 *send()* 或 *sendto()* 调用中指定 `MSG_MORE` 的数据被打包成一个单独的数据报，只有在不指定此标志的进一步调用时，数据才会被传输。（Linux 还提供了类似的 `UDP_CORK` 套接字选项，使得来自连续 *send()* 或 *sendto()* 调用的数据被累积成一个数据报，在禁用 `UDP_CORK` 时传输。）`MSG_MORE` 标志对 UNIX 域套接字没有影响。

`MSG_NOSIGNAL`

当在已连接的流套接字上发送数据时，如果连接的另一端已关闭，不要生成`SIGPIPE`信号。相反，*send()*调用会因错误`EPIPE`而失败。这与忽略`SIGPIPE`信号时获得的行为相同，区别在于`MSG_NOSIGNAL`标志按调用逐个控制行为。

`MSG_OOB`

在流套接字上发送带外数据。请参阅带外数据。

上述标志中，只有`MSG_OOB`由 SUSv3 指定。SUSv4 增加了`MSG_NOSIGNAL`的规范。`MSG_DONTWAIT`不是标准化的，但出现在一些其他的 UNIX 实现中。`MSG_MORE`是 Linux 特有的。*send(2)*和*recv(2)*手册页面描述了更多的标志，这里不再涉及。

## *sendfile()*系统调用

类似于 Web 服务器和文件服务器等应用程序经常需要通过（已连接的）套接字传输磁盘文件的未修改内容。实现这一功能的一种方法是使用以下形式的循环：

```
while ((n = read(diskfilefd, buf, BUZ_SIZE)) > 0)
    write(sockfd, buf, n);
```

对于许多应用程序，这样的循环是完全可以接受的。然而，如果我们通过套接字频繁传输大文件，这种技术效率低下。为了传输文件，我们必须使用两次系统调用（可能在循环内多次调用）：一次是将文件内容从内核缓冲区复制到用户空间，另一次是将用户空间的缓冲区复制回内核空间，以便通过套接字传输。该场景如图 61-1 左侧所示。如果应用程序在传输文件内容之前不对其进行处理，这种两步过程是浪费的。*sendfile()*系统调用旨在消除这种低效。当应用程序调用*sendfile()*时，文件内容会直接传输到套接字，而无需经过用户空间，如图 61-1 右侧所示。这被称为*零拷贝传输*。

![将文件内容传输到套接字](img/61-1_SOCKADV-sendfile.png.jpg)图 61-1。将文件内容传输到套接字

```
#include <sys/sendfile.h>

ssize_t `sendfile`(int *out_fd*, int *in_fd*, off_t **offset*, size_t *count*);
```

### 注意

返回传输的字节数，出错时返回-1

*sendfile()*系统调用将由描述符*in_fd*引用的文件中的字节传输到由描述符*out_fd*引用的文件中。*out_fd*描述符必须引用一个套接字。*in_fd*参数必须引用一个可以应用*mmap()*的文件；实际上，这通常意味着一个常规文件。这在某种程度上限制了*sendfile()*的使用。我们可以用它将数据从文件传输到套接字，但不能反向操作。而且，我们不能使用*sendfile()*将数据直接从一个套接字传输到另一个套接字。

### 注意

如果*sendfile()*可以用来在两个常规文件之间传输字节，也可以获得性能上的好处。在 Linux 2.4 及更早版本中，*out_fd*可以指向一个常规文件。一些底层实现的重构意味着这个功能在 2.6 内核中消失了。然而，这个功能可能会在未来的内核版本中恢复。

如果*offset*不是`NULL`，则它应指向一个*off_t*类型的值，该值指定从*in_fd*中传输字节的起始文件偏移量。这是一个值结果参数。返回时，它包含从*in_fd*中传输的最后一个字节之后的下一个字节的偏移量。在这种情况下，*sendfile()*不会更改*in_fd*的文件偏移量。

如果*offset*是`NULL`，则字节将从*in_fd*的当前文件偏移处开始传输，并且文件偏移量将更新以反映传输的字节数。

*count*参数指定要传输的字节数。如果在传输*count*字节之前遇到文件结尾，则仅传输可用的字节。成功时，*sendfile()*返回实际传输的字节数。

SUSv3 并未指定*sendfile()*。某些其他 UNIX 实现中提供了*sendfile()*的版本，但其参数列表通常与 Linux 版本不同。

### 注意

从 2.6.17 内核开始，Linux 提供了三个新的（非标准的）系统调用——*splice()*、*vmsplice()*和*tee()*——它们提供了*sendfile()*功能的超集。有关详细信息，请参阅手册页。

#### `TCP_CORK`套接字选项

为了进一步提高使用*sendfile()*的 TCP 应用程序的效率，有时使用 Linux 特有的`TCP_CORK`套接字选项是有益的。例如，考虑一个 web 服务器响应浏览器请求并传送网页的场景。web 服务器的响应由两部分组成：HTTP 头部，可能通过*write()*输出，然后是网页数据，可能通过*sendfile()*输出。在这种情况下，通常会传输*两个*TCP 段：头部在第一个（相对较小的）段中发送，然后页面数据在第二个段中发送。这种网络带宽的使用效率低下。它可能还会为发送和接收的 TCP 带来不必要的工作，因为在许多情况下，HTTP 头部和页面数据足够小，可以放入一个单独的 TCP 段中。`TCP_CORK`选项旨在解决这个低效问题。

当`TCP_CORK`选项在 TCP 套接字上启用时，所有后续的输出都会被缓冲到一个单一的 TCP 段中，直到达到段大小的上限、`TCP_CORK`选项被禁用、套接字被关闭，或从第一次写入被“塞住”字节的时刻起经过最多 200 毫秒。（这个超时确保了如果应用程序忘记禁用`TCP_CORK`选项，仍然能够传输已塞住的数据。）

我们通过 *setsockopt()* 系统调用启用和禁用 `TCP_CORK` 选项（套接字选项）。以下代码（省略了错误检查）演示了如何在我们的假设 HTTP 服务器示例中使用 `TCP_CORK`：

```
int optval;

/* Enable TCP_CORK option on 'sockfd' - subsequent TCP output is corked
   until this option is disabled. */

optval = 1;
setsockopt(sockfd, IPPROTO_TCP, TCP_CORK, &optval, sizeof(optval));

write(sockfd, ...);                     /* Write HTTP headers */
sendfile(sockfd, ...);                  /* Send page data */

/* Disable TCP_CORK option on 'sockfd' - corked output is now transmitted
   in a single TCP segment. */

optval = 0
setsockopt(sockfd, IPPROTO_TCP, TCP_CORK, &optval, sizeof(optval));
```

我们可以通过在应用程序中构建一个单一的数据缓冲区，然后使用一次 *write()* 传输该缓冲区，来避免两个数据段被传输的可能性。（另外，我们也可以使用 *writev()* 将两个不同的缓冲区合并成一次输出操作。）然而，如果我们想要结合 *sendfile()* 的零拷贝效率，同时能够将一个头部包含在传输文件数据的第一个数据段中，那么我们需要使用 `TCP_CORK`。

### 注意

在 Socket-Specific I/O System Calls: *recv()* 和 *send()* and send()") 中，我们提到过 `MSG_MORE` 标志提供了类似于 `TCP_CORK` 的功能，但它是按每次系统调用来处理的。这不一定是一个优势。可以在套接字上设置 `TCP_CORK` 选项，然后执行一个程序，该程序在继承的文件描述符上执行输出，而不需要知道 `TCP_CORK` 选项。相比之下，使用 `MSG_MORE` 需要明确修改程序的源代码。

FreeBSD 提供了类似于 `TCP_CORK` 的选项，形式为 `TCP_NOPUSH`。

## 获取套接字地址

*getsockname()* 和 *getpeername()* 系统调用分别返回本地套接字绑定的本地地址和本地套接字所连接的对等套接字地址。

```
#include <sys/socket.h>

int `getsockname`(int *sockfd*, struct sockaddr **addr*, socklen_t **addrlen*);
int `getpeername`(int *sockfd*, struct sockaddr **addr*, socklen_t **addrlen*);
```

### 注意

成功时返回 0，出错时返回 -1

对于这两个调用，*sockfd* 是一个指向套接字的文件描述符，*addr* 是一个指向适当大小缓冲区的指针，该缓冲区用于返回包含套接字地址的结构体。该结构体的大小和类型取决于套接字的域。*addrlen* 参数是一个值结果参数。在调用之前，它应该被初始化为 *addr* 指向的缓冲区的长度；返回时，它包含实际写入该缓冲区的字节数。

*getsockname()* 函数返回套接字的地址族和套接字绑定的地址。如果套接字是由其他程序（例如 *inetd(8)*) 绑定的，且套接字文件描述符在 *exec()* 过程中被保留，则这非常有用。

调用 *getsockname()* 也很有用，如果我们想要确定内核在执行隐式绑定一个 Internet 域套接字时为套接字分配的临时端口号。内核会在以下情况下执行隐式绑定：

+   在对一个尚未通过 *bind()* 绑定地址的 TCP 套接字调用 *connect()* 或 *listen()* 后；

+   在对一个尚未绑定地址的 UDP 套接字进行第一次 *sendto()* 调用时；或者

+   在*bind()*调用后，端口号（*sin_port*）指定为 0。在这种情况下，*bind()*指定了套接字的 IP 地址，但内核会选择一个临时的端口号。

*getpeername()*系统调用返回流套接字连接的对等套接字的地址。这在 TCP 套接字中尤其有用，尤其是当服务器想要找出已建立连接的客户端的地址时。也可以在执行*accept()*调用时获得该信息；然而，如果服务器是由执行了*accept()*的程序（例如，*inetd*）启动的，则它会继承套接字文件描述符，但*accept()*返回的地址信息不再可用。

示例 61-3 和 getpeername()")演示了*getsockname()*和*getpeername()*的使用。此程序使用我们在示例 59-9 中定义的函数（在一个互联网域套接字库中），并执行以下步骤：

1.  使用我们的*inetListen()*函数创建一个监听套接字，*listenFd*，该套接字绑定到程序唯一命令行参数指定的通配符 IP 地址和端口。（端口可以通过数字指定，也可以通过服务名称指定。）*len*参数返回此套接字域的地址结构的长度。这个值会在稍后的*malloc()*调用中传递，用于分配一个缓冲区，该缓冲区用于从*getsockname()*和*getpeername()*调用中返回套接字地址。

1.  使用我们的*inetConnect()*函数创建第二个套接字，*connFd*，该套接字用于向第一步中创建的套接字发送连接请求。

1.  在监听套接字上调用*accept()*以创建一个第三个套接字，*acceptFd*，该套接字与前一步创建的套接字建立连接。

1.  使用*getsockname()*和*getpeername()*调用获取两个已连接套接字（*connFd*和*acceptFd*）的本地和对等地址。在每次调用之后，程序使用我们的*inetAddressStr()*函数将套接字地址转换为可打印的形式。

1.  稍等几秒钟，以便我们可以运行*netstat*来确认套接字地址信息。（我们在第 61.7 节中描述了*netstat*）

以下是一个示例程序运行的 Shell 会话日志：

```
$ `./socknames 55555 &`
getsockname(connFd):   (localhost, 32835)
getsockname(acceptFd): (localhost, 55555)
getpeername(connFd):   (localhost, 55555)
getpeername(acceptFd): (localhost, 32835)
[1] 8171
$ `netstat -a | egrep '(Address|55555)'`
Proto Recv-Q Send-Q Local Address    Foreign Address  State
tcp        0      0 *:55555          *:*              LISTEN
tcp        0      0 localhost:32835  localhost:55555  ESTABLISHED
tcp        0      0 localhost:55555  localhost:32835  ESTABLISHED
```

从上述输出中，我们可以看到已连接的套接字（*connFd*）已绑定到临时端口 32835\. *netstat*命令展示了程序创建的所有三个套接字的信息，并允许我们确认两个连接套接字的端口信息，这两个套接字处于已建立（ESTABLISHED）状态（在 TCP 状态机和状态转换图中描述）。

示例 61-3. 使用 *getsockname()* 和 *getpeername()*

```
`sockets/socknames.c`
#include "inet_sockets.h"               /* Declares our socket functions */
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    int listenFd, acceptFd, connFd;
    socklen_t len;                      /* Size of socket address buffer */
    void *addr;                         /* Buffer for socket address */
    char addrStr[IS_ADDR_STR_LEN];

    if (argc != 2 || strcmp(argv[1], "--help") == 0)
        usageErr("%s service\n", argv[0]);

    listenFd = inetListen(argv[1], 5, &len);
    if (listenFd == -1)
        errExit("inetListen");

    connFd = inetConnect(NULL, argv[1], SOCK_STREAM);
    if (connFd == -1)
        errExit("inetConnect");

    acceptFd = accept(listenFd, NULL, NULL);
    if (acceptFd == -1)
        errExit("accept");

    addr = malloc(len);
    if (addr == NULL)
        errExit("malloc");

    if (getsockname(connFd, addr, &len) == -1)
        errExit("getsockname");
    printf("getsockname(connFd):   %s\n",
            inetAddressStr(addr, len, addrStr, IS_ADDR_STR_LEN));
    if (getsockname(acceptFd, addr, &len) == -1)
        errExit("getsockname");
    printf("getsockname(acceptFd): %s\n",
            inetAddressStr(addr, len, addrStr, IS_ADDR_STR_LEN));

    if (getpeername(connFd, addr, &len) == -1)
        errExit("getpeername");
    printf("getpeername(connFd):   %s\n",
            inetAddressStr(addr, len, addrStr, IS_ADDR_STR_LEN));
    if (getpeername(acceptFd, addr, &len) == -1)
        errExit("getpeername");
    printf("getpeername(acceptFd): %s\n",
            inetAddressStr(addr, len, addrStr, IS_ADDR_STR_LEN));

    sleep(30);                          /* Give us time to run netstat(8) */
    exit(EXIT_SUCCESS);
}
     `sockets/socknames.c`
```

## 更深入了解 TCP

了解 TCP 操作的某些细节可以帮助我们调试使用 TCP 套接字的应用程序，并在某些情况下提高应用程序的效率。在接下来的部分，我们将探讨：

+   TCP 段的格式；

+   TCP 确认机制；

+   TCP 状态机；

+   TCP 连接的建立和终止；

+   TCP TIME_WAIT 状态。

### TCP 段格式

图 61-2 显示了 TCP 连接的端点之间交换的 TCP 段的格式。各字段的含义如下：

+   *源端口号*：这是发送方 TCP 的端口号。

+   *目标端口号*：这是目标 TCP 的端口号。

+   *序列号*：这是该段的序列号。它是该段中第一个数据字节相对于在此连接方向上传输的数据流中的偏移量，如 传输控制协议（TCP）") 中所述。

    ![TCP 段格式](img/61-2_SOCKADV-TCP-segment.png.jpg)图 61-2. TCP 段格式

+   *确认号*：如果 ACK 位（见下文）被设置，则此字段包含接收方期望从发送方接收的下一个字节的数据的序列号。

+   *头部长度*：这是头部的长度，单位为 32 位字。由于这是一个 4 位字段，总的头部长度最多可以是 60 字节（15 个字）。该字段使接收方 TCP 能够确定可变长度 *选项* 字段的长度以及 *数据* 的起始点。

+   *保留*：该字段由 4 位未使用的比特组成（必须设置为 0）。

+   *控制位*：此字段由 8 位组成，进一步指定该段的含义：

    +   *CWR*：*拥塞窗口减少*标志。

    +   *ECE*：*显式拥塞通知回显*标志。CWR 和 ECE 标志作为 TCP/IP 显式拥塞通知（ECN）算法的一部分。ECN 是 TCP/IP 中相对较新的添加项，描述详见 RFC 3168 和 [Floyd, 1994]。ECN 从 Linux 内核 2.4 版本起得到支持，并通过将非零值放入 Linux 特定的 `/proc/sys/net/ipv4/tcp_ecn` 文件中启用。

    +   *URG*：如果设置，则 *紧急指针* 字段包含有效信息。

    +   *ACK*：如果设置，则 *确认号* 字段包含有效信息（即该段确认了先前由对端发送的数据）。

    +   *PSH*：将所有接收到的数据推送到接收进程。此标志在 RFC 993 和 [Stevens, 1994] 中有所描述。

    +   *RST*：重置连接。此标志用于处理各种错误情况。

    +   *SYN*：同步序列号。带有此标志的段在连接建立期间交换，以允许两个 TCP 指定用于每个方向数据传输的初始序列号。

    +   *FIN*：由发送方使用，表示它已经完成数据发送。

    一个段中可以设置多个控制位（或根本没有设置），这使得一个段可以同时用于多个目的。例如，稍后我们将看到，带有 SYN 和 ACK 位的段将在 TCP 连接建立过程中交换。

+   *窗口大小*：当接收方发送 ACK 以指示接收方有足够的空间接受多少字节的数据时，使用此字段。（这与在传输控制协议（TCP）")中简要描述的滑动窗口方案有关。）

+   *校验和*：这是一个 16 位的校验和，涵盖 TCP 头和 TCP 数据。

    ### 注意

    TCP 校验和不仅涵盖 TCP 头和数据，还包括通常称为 TCP *伪头部*的 12 个字节。伪头部包括以下内容：源和目的 IP 地址（各 4 个字节）；2 个字节指定 TCP 段的大小（该值是计算得出的，但不构成 IP 或 TCP 头的一部分）；1 个字节包含值 6，这是 TCP 在 TCP/IP 协议栈中的唯一协议号；以及 1 个包含 0 的填充字节（使伪头部的长度是 16 位的倍数）。将伪头部包括在校验和计算中是为了允许接收方的 TCP 再次检查传入的段是否到达正确的目的地（即，确保 IP 没有错误地接受指向其他主机的数据报，或者没有把应该传递给其他上层的包交给 TCP）。UDP 也以类似的方式计算其数据包头部的校验和，原因相似。有关伪头部的更多细节，请参见[Stevens, 1994]。

+   *紧急指针*：如果设置了 URG 控制位，则该字段指示在从发送方到接收方传输的数据流中所谓的紧急数据的位置。我们将在带外数据中简要讨论紧急数据。

+   *选项*：这是一个可变长度的字段，包含控制 TCP 连接操作的选项。

+   *数据*：此字段包含在此段中传输的用户数据。如果此段不包含任何数据（例如，仅为 ACK 段），则该字段的长度可能为 0。

### TCP 序列号和确认

通过 TCP 连接传输的每个字节都由 TCP 分配一个逻辑序列号。（连接中的每个流都有自己的序列号。）当一个报文段被传输时，它的 *序列号* 字段会设置为报文段中第一个数据字节在该方向数据流中的逻辑偏移量。这样接收方 TCP 就能按正确的顺序组装接收到的报文段，并在发送确认时指明哪些数据已经被接收。

为了实现可靠通信，TCP 使用正向确认；也就是说，当一个报文段成功接收时，接收方 TCP 会发送一个确认消息（即设置了 ACK 位的报文段）给发送方 TCP，如 图 61-3 所示。此消息的 *确认号* 字段被设置为指示接收方预期接收的下一个字节的数据的逻辑序列号。（换句话说，确认号字段中的值是它确认的报文段中最后一个字节的序列号加 1。）

![TCP 中的确认](img/61-3_SOCKADV-TCP-ACK.png.jpg)图 61-3. TCP 中的确认

当发送方 TCP 发送一个报文段时，它会设置一个定时器。如果在定时器到期之前没有收到确认，报文段将被重新发送。

### 注意

图 61-3 以及后续类似的图示旨在说明两个端点之间 TCP 报文段的交换。在阅读这些图示时，假设有一个隐式的时间维度，从上到下阅读时表示时间的流逝。

### TCP 状态机与状态转换图

维持一个 TCP 连接需要协调连接两端的 TCP。为了简化这项任务，TCP 端点被建模为一个 *状态机*。这意味着 TCP 可以处于一组固定的 *状态* 中，并且根据 *事件*（例如来自应用程序的系统调用或来自对端 TCP 的报文段到达）在状态之间转换。TCP 状态包括以下几种：

+   LISTEN：TCP 正在等待来自对端 TCP 的连接请求。

+   SYN_SENT：TCP 已代表应用程序发送了一个 SYN 请求，正在等待对端的回复，以完成连接。

+   SYN_RECV：TCP 在 LISTEN 状态下收到一个 SYN，并以 SYN/ACK 响应（即同时设置了 SYN 和 ACK 位的 TCP 报文段），现在等待接收方 TCP 的 ACK 来完成连接。

+   ESTABLISHED：与对端 TCP 的连接已建立完成。现在两个 TCP 之间可以在任意方向交换数据报文段。

+   FIN_WAIT1: 应用程序已关闭连接。TCP 向对端 TCP 发送了 FIN，以终止自己一方的连接，并在等待对方的 ACK。此状态及接下来的三个状态与执行主动关闭的应用程序相关，即第一个关闭连接一方的应用程序。

+   FIN_WAIT2: TCP 原本处于 FIN_WAIT1 状态，现在已收到来自对端 TCP 的 ACK。

+   CLOSING: TCP 原本在 FIN_WAIT1 状态下等待 ACK，但却收到了来自对端的 FIN，这表明对端同时试图执行主动关闭。（换句话说，两个 TCP 几乎在同一时间发送了 FIN 段。这是一个罕见的情况。）

+   TIME_WAIT: 在执行主动关闭后，TCP 已收到 FIN，表明对端 TCP 执行了被动关闭。此时，TCP 会在 TIME_WAIT 状态下等待一段固定时间，以确保 TCP 连接可靠终止，并确保任何过期的重复段在创建相同连接的新实例之前从网络中消失。（我们将在 TIME_WAIT 状态一节中详细解释 TIME_WAIT 状态。）当这段固定时间过后，连接关闭，相关的内核资源被释放。

+   CLOSE_WAIT: TCP 已收到来自对端 TCP 的 FIN。此状态及后续状态与执行被动关闭的应用程序相关，即第二个关闭连接的应用程序。

+   LAST_ACK: 应用程序执行了被动关闭，原本处于 CLOSE_WAIT 状态的 TCP 向对端 TCP 发送了 FIN，并在等待对方的确认。当收到这个 ACK 时，连接关闭，相关的内核资源被释放。

对上述状态，RFC 793 添加了一个额外的虚拟状态——CLOSED，表示没有连接的状态（即没有为描述 TCP 连接分配内核资源）。

### 注意

在上述列表中，我们使用的是 Linux 源代码中定义的 TCP 状态的拼写方式。这些拼写与 RFC 793 中的拼写略有不同。

图 61-4 显示了 TCP 的*状态转换图*。（该图基于 RFC 793 中的图示以及[Stevens 等，2004]的图示。）此图展示了 TCP 端点如何响应各种事件从一个状态转移到另一个状态。每个箭头表示一种可能的转换，并标注了触发该转换的事件。该标签可以是应用程序的动作（以粗体显示），也可以是*recv*字符串，表示接收到来自对端 TCP 的数据段。当 TCP 从一个状态转移到另一个状态时，它可能会向对端发送数据段，这一点通过转换上的*send*标签来表示。例如，从 ESTABLISHED 状态到 FIN_WAIT1 状态的转换箭头显示，触发事件是本地应用程序的*close()*，并且在转换过程中，TCP 向对端发送一个 FIN 数据段。

在图 61-4 中，客户端 TCP 的常见转换路径通过粗实线箭头表示，服务器 TCP 的常见转换路径通过粗虚线箭头表示。（其他箭头表示较少使用的路径。）通过查看这些路径上箭头的括号编号，我们可以看到两个 TCP 发送和接收的段是彼此的镜像。（在 ESTABLISHED 状态之后，如果是服务器执行主动关闭，则服务器 TCP 和客户端 TCP 的路径可能与所示路径相反。）

### 注意

图 61-4 并未显示 TCP 状态机的所有可能转换；它仅展示了主要感兴趣的那些。更详细的 TCP 状态转换图可以在[`www.cl.cam.ac.uk/~pes20/Netsem/poster.pdf`](http://www.cl.cam.ac.uk/~pes20/Netsem/poster.pdf)找到。

### TCP 连接建立

在套接字 API 层，通过以下步骤连接两个流套接字（参见图 56-1，在监听传入连接：*listen()*")）：

1.  服务器调用*listen()*执行套接字的被动打开，然后调用*accept()*，该调用会阻塞，直到建立连接。

1.  客户端调用*connect()*来执行套接字的主动打开，以便建立与服务器的被动套接字的连接。

TCP 执行建立连接的步骤如图 61-5 所示。这些步骤通常被称为*三次握手*，因为在两个 TCP 之间会传递三个数据段。步骤如下：

1.  *connect()* 函数使客户端 TCP 向服务器 TCP 发送一个 SYN 数据段。这个数据段通知服务器 TCP 客户端 TCP 的初始序列号（图中标记为 *M*）。这一信息是必要的，因为序列号并不是从 0 开始的，正如在 传输控制协议 (TCP)") 中所提到的那样。

1.  服务器 TCP 必须同时确认客户端 TCP 的 SYN 数据段，并通知客户端 TCP 其自己的初始序列号（图中标记为 *N*）。(需要两个序列号，因为流套接字是双向的。) 服务器 TCP 可以通过返回一个包含 SYN 和 ACK 控制位的单个数据段来完成这两个操作。（我们说 ACK 是 *附加* 在 SYN 上的。）

1.  客户端 TCP 向服务器 TCP 发送 ACK 数据段以确认服务器 TCP 的 SYN 数据段。

![TCP 状态转换图](img/61-4_SOCKADV-TCP-STD.png.jpg)图 61-4. TCP 状态转换图

### 注意

在三次握手的前两步中交换的 SYN 数据段，可能包含 TCP 头部的 *options* 字段中的信息，这些信息用于确定连接的各种参数。有关详细信息，请参见 [Stevens 等，2004]、[Stevens，1994] 和 [Wright & Stevens，1995]。

图 61-5 中尖括号内的标签（例如，<LISTEN>）表示连接两端 TCP 的状态。

SYN 标志占用了连接的序列号空间中的一个字节。这样做是必要的，因为该标志必须能够明确地被确认，因为带有此标志的数据段可能还包含数据字节。这就是为什么我们在 图 61-5 中显示对 *SYN M* 数据段的确认为 *ACK M+1* 的原因。

![TCP 连接建立的三次握手](img/61-5_SOCKADV-TCP-connection-establishment.png.jpg)图 61-5. TCP 连接建立的三次握手

### TCP 连接终止

关闭 TCP 连接通常按以下方式进行：

1.  连接一端的应用程序执行 *close()*。通常情况下，但不一定是客户端，会执行此操作。我们称这个应用程序执行了 *主动关闭*。

1.  之后，连接另一端的应用程序（服务器）也执行 *close()* 操作。这被称为 *被动关闭*。

图 61-6 显示了底层 TCP 执行的相应步骤（在此假设是客户端执行主动关闭）。这些步骤如下：

1.  客户端执行主动关闭，导致客户端 TCP 向服务器 TCP 发送 FIN。

1.  在收到 FIN 后，服务器 TCP 会响应一个 ACK。之后，服务器若尝试从套接字上*read()*，则会返回文件结束符（即 0）。

1.  当服务器稍后关闭连接的一端时，服务器 TCP 会向客户端 TCP 发送一个 FIN。

1.  客户端 TCP 用 ACK 回应，以确认服务器的 FIN。

与 SYN 标志类似，出于相同的原因，FIN 标志会占用连接序列号空间中的一个字节。这就是为什么我们在图 61-6 中将*FIN M*段的确认表示为*ACK M+1*。

![TCP 连接终止](img/61-6_SOCKADV-TCP-disconnect.png.jpg)图 61-6. TCP 连接终止

### 在 TCP 套接字上调用*shutdown()*

前一节讨论假设了全双工关闭；即，应用程序使用*close()*关闭 TCP 套接字的发送和接收通道。正如在系统调用*shutdown()*")中所指出的，我们可以使用*shutdown()*仅关闭连接的一个通道（半双工关闭）。本节指出了在 TCP 套接字上使用*shutdown()*的一些具体细节。

将*how*指定为`SHUT_WR`或`SHUT_RDWR`会启动 TCP 连接终止序列（即主动关闭），该过程在 TCP 连接终止中进行了描述，无论是否有其他文件描述符引用该套接字。一旦该序列被启动，本地 TCP 进入 FIN_WAIT1 状态，然后进入 FIN_WAIT2 状态，而对端 TCP 进入 CLOSE_WAIT 状态（图 61-6）。如果*how*指定为`SHUT_WR`，由于套接字文件描述符保持有效且连接的读取半部分仍然开放，因此对端可以继续向我们发送数据。

`SHUT_RD`操作在 TCP 套接字上无法有效使用。这是因为大多数 TCP 实现并未提供`SHUT_RD`预期的行为，且`SHUT_RD`的效果在不同的实现中有所不同。在 Linux 及其他一些实现中，在执行`SHUT_RD`（并且读取完所有待处理数据）之后，*read()*会返回文件结束符，这与第 61.2 节中对`SHUT_RD`的描述一致。然而，如果对端应用随后在其套接字上写入数据，则仍然可以在本地套接字上读取该数据。

在某些其他实现中（例如 BSD 系统），`SHUT_RD` 确实会导致后续对 *read()* 的调用始终返回 0。然而，在这些实现中，如果对端继续对套接字执行 *write()* 操作，则数据通道最终会填满，直到对端的进一步（阻塞） *write()* 调用被阻塞。（对于 UNIX 域流套接字，如果对端在本地套接字执行了 `SHUT_RD` 后继续向其套接字写入数据，它将收到 `SIGPIPE` 信号，并且出现 `EPIPE` 错误。）

总结来说，应避免在可移植的 TCP 应用程序中使用 `SHUT_RD`。

### TIME_WAIT 状态

TCP 的 TIME_WAIT 状态是网络编程中常见的一个困惑来源。通过查看 图 61-4，我们可以看到执行主动关闭的 TCP 会经历该状态。TIME_WAIT 状态存在有两个目的：

+   实现可靠的连接终止；并且

+   允许旧的重复段在网络中过期，以便它们不会被新连接的实例接受。

TIME_WAIT 状态与其他状态不同，因为导致从该状态（转至 CLOSED）过渡的事件是超时。该超时的持续时间为两倍的 MSL（2MSL），其中 MSL（*最大段生命周期*）是指 TCP 段在网络中假定的最大生命周期。

### 注意

IP 头部中的 8 位生存时间（TTL）字段确保所有 IP 数据包在经过固定数量的跃点（路由器跳数）并且没有到达目的地时，最终会被丢弃。MSL 是一个估计值，表示一个 IP 数据包超出 TTL 限制所需的最大时间。由于 TTL 使用 8 位表示，最大允许的跃点数为 255。通常，IP 数据包所需的跃点数远低于此值。数据包可能会遇到此限制，原因是某些类型的路由器异常（例如路由器配置问题）导致数据包在网络中形成环路，直到超过 TTL 限制。

BSD 套接字实现假定 MSL 的值为 30 秒，而 Linux 遵循 BSD 标准。因此，在 Linux 上，TIME_WAIT 状态的生命周期为 60 秒。然而，RFC 1122 推荐将 MSL 的值设为 2 分钟，遵循此推荐的实现中，TIME_WAIT 状态的持续时间可以达到 4 分钟。

我们可以通过查看图 61-6 来理解 TIME_WAIT 状态的第一个目的——确保可靠的连接终止。在该图中，我们可以看到，TCP 连接的终止通常会交换四个段。这些段中最后一个是从执行主动关闭的 TCP 发送给执行被动关闭的 TCP 的 ACK。假设这个 ACK 在网络中丢失。如果发生这种情况，执行被动关闭的 TCP 最终会重新传输其 FIN。让执行主动关闭的 TCP 保持在 TIME_WAIT 状态一段固定时间，可以确保它能够在这种情况下重新发送最终的 ACK。如果执行主动关闭的 TCP 不再存在，那么——因为它没有连接的状态信息——TCP 协议会对重新传输的 FIN 发送一个 RST（复位）段给执行被动关闭的 TCP，而这个 RST 会被解释为一个错误。（这也解释了为什么 TIME_WAIT 状态的持续时间是*两倍*的 MSL：一个 MSL 用于最终的 ACK 到达对端 TCP，再加一个 MSL，以防必须发送进一步的 FIN。）

### 注意

对于执行被动关闭的 TCP 来说，不需要 TIME_WAIT 状态的等效机制，因为它是连接终止中最后交换的发起者。在发送完 FIN 后，该 TCP 会等待来自对端的 ACK，并在定时器在收到 ACK 之前过期时重新传输 FIN。

要理解 TIME_WAIT 状态的第二个目的——确保网络中旧的重复段过期——我们必须记住，TCP 使用的重传算法可能会生成重复的段，并且根据路由决策，这些重复的段可能在连接关闭后到达。例如，假设我们有一个 TCP 连接，连接的两个套接字地址分别是 `204.152.189.116` 端口 21（FTP 端口）和 `200.0.0.1` 端口 50,000。假设这个连接已经关闭，随后建立了一个新连接，使用完全相同的 IP 地址和端口。这被称为连接的新实例。在这种情况下，TCP 必须确保旧的重复段不会在新实例中被当作有效数据接受。为此，TCP 会通过防止在 TIME_WAIT 状态的端点存在时建立新的连接实例来实现这一点。

在线论坛上一个常见的问题是如何禁用 TIME_WAIT 状态，因为当重新启动的服务器尝试将套接字绑定到处于 TIME_WAIT 状态的地址时，可能会导致 `EADDRINUSE` 错误（“地址已在使用”）。虽然有方法可以做到这一点（参见 [Stevens 等，2004]），也有方法可以终结处于此状态的 TCP（即使 TIME_WAIT 状态提前终止，参见 [Snader，2000]），但应该避免这样做，因为它会破坏 TIME_WAIT 状态所提供的可靠性保证。在 The *SO_REUSEADDR* Socket Option 中，我们将探讨如何使用 `SO_REUSEADDR` 套接字选项，它可以避免 `EADDRINUSE` 错误的常见原因，同时仍然允许 TIME_WAIT 提供其可靠性保证。

## 监控套接字：*netstat*

*netstat* 程序显示系统上互联网和 UNIX 域套接字的状态。它是编写套接字应用程序时的一个有用的调试工具。大多数 UNIX 实现都提供了 *netstat* 的版本，尽管在命令行参数的语法上不同的实现存在一些差异。

默认情况下，在没有命令行选项的情况下执行 *netstat* 时，会显示 UNIX 和互联网域中已连接的套接字的信息。我们可以使用多个命令行选项来更改显示的信息。部分选项列在 表格 61-1 中。

表格 61-1. *netstat* 命令的选项

| 选项 | 描述 |
| --- | --- |
| `-a` | 显示所有套接字的信息，包括监听套接字 |
| `-e` | 显示扩展信息（包括套接字所有者的用户 ID） |
| `-c` | 持续重新显示套接字信息（每秒更新一次） |
| `-l` | 仅显示监听套接字的信息 |
| `-n` | 以数字形式显示 IP 地址、端口号和用户名 |
| `-p` | 显示套接字所属程序的进程 ID 和名称 |
| `--inet` | 显示互联网域套接字的信息 |
| `--tcp` | 显示互联网域 TCP（流式）套接字的信息 |
| `--udp` | 显示互联网域 UDP（数据报）套接字的信息 |
| `--unix` | 显示 UNIX 域套接字的信息 |

这是使用 *netstat* 列出系统上所有互联网域套接字时的简化输出示例：

```
$ `netstat -a --inet`
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address    Foreign Address  State
tcp        0      0 *:50000          *:*              LISTEN
tcp        0      0 *:55000          *:*              LISTEN
tcp        0      0 localhost:smtp   *:*              LISTEN
tcp        0      0 localhost:32776  localhost:58000  TIME_WAIT
tcp    34767      0 localhost:55000  localhost:32773  ESTABLISHED
tcp        0 115680 localhost:32773  localhost:55000  ESTABLISHED
udp        0      0 localhost:61000  localhost:60000  ESTABLISHED
udp      684      0 *:60000          *:*
```

对于每个互联网域套接字，我们会看到以下信息：

+   `Proto`：这是套接字协议—例如，`tcp` 或 `udp`。

+   `Recv-Q`：这是本地应用程序尚未读取的套接字接收缓冲区中的字节数。对于 UDP 套接字，此字段不仅计算数据，还包括 UDP 头部和其他元数据中的字节。

+   `发送队列`：这是套接字发送缓冲区中排队等待传输的字节数。与 `接收队列` 字段一样，对于 UDP 套接字，这个字段包括 UDP 头和其他元数据中的字节数。

+   `本地地址`：这是套接字绑定的地址，格式为 *主机-IP 地址:端口*。默认情况下，地址的两个部分都会显示为名称，除非数字值无法解析为相应的主机和服务名称。如果地址的主机部分为星号（`*`），则表示通配符 IP 地址。

+   `外部地址`：这是与此套接字绑定的对等套接字的地址。字符串 `*:*` 表示没有对等地址。

+   `状态`：这是套接字的当前状态。对于 TCP 套接字，此状态是 TCP 状态机与状态转换图 中描述的状态之一。

欲了解更多详细信息，请参阅 *netstat(8)* 手册页面。

`/proc/net` 目录下的各种 Linux 特定文件允许程序读取与 *netstat* 显示的相同信息。这些文件分别命名为 `tcp`、`udp`、`tcp6`、`udp6` 和 `unix`，用途显而易见。欲了解更多详细信息，请参阅 *proc(5)* 手册页面。

## 使用 *tcpdump* 监控 TCP 流量

*tcpdump* 程序是一个有用的调试工具，它允许超级用户监控实时网络上的互联网流量，生成类似于图表的实时文本信息，如 图 61-3。尽管其名称为 *tcpdump*，但它可以用于显示所有类型网络包的流量（例如 TCP 段、UDP 数据报和 ICMP 包）。对于每个网络包，*tcpdump* 显示诸如时间戳、源和目标 IP 地址以及其他协议特定的细节信息。可以根据协议类型、源和目标 IP 地址及端口号以及其他标准来选择要监控的包。有关详细信息，请参阅 *tcpdump* 手册页面。

### 注意

*wireshark*（前身为 *ethereal*；[`www.wireshark.org`](http://www.wireshark.org)）程序执行的任务与 *tcpdump* 相似，但通过图形界面显示流量信息。

对于每个 TCP 段，*tcpdump* 会显示如下形式的一行：

```
*src* > *dst*: *flags data-seqno ack window urg* <*options*>
```

这些字段的含义如下：

+   *src*：这是源 IP 地址和端口。

+   *dst*：这是目标 IP 地址和端口。

+   *flags*：该字段包含零个或多个字母，每个字母对应 TCP 段格式 中描述的一个 TCP 控制位：S（SYN）、F（FIN）、P（PSH）、R（RST）、E（ECE）和 C（CWR）。

+   *data-seqno*：这是该数据包中所包含字节的序列号空间的范围。

    ### 注意

    默认情况下，序列号范围是相对于监视数据流的第一个字节显示的。*tcpdump -S*选项使得序列号以绝对格式显示。

+   *ack*：这是一个形式为“`ack` *num*”的字符串，表示从连接的另一个方向预期接收的下一个字节的序列号。

+   *window*：这是一个形式为“`win` *num*”的字符串，表示此连接在反方向上可用于传输的接收缓冲区空间的字节数。

+   *urg*：这是一个形式为“`urg` *num*”的字符串，表示该报文段在指定偏移量处包含紧急数据。

+   *options*：这个字符串描述了报文段中包含的任何 TCP 选项。

*src*、*dst*和*flags*字段始终会出现。其余字段仅在适当时显示。

下面的 Shell 会话展示了如何使用*tcpdump*监控客户端（运行在`pukaki`主机上）与服务器（运行在`tekapo`主机上）之间的流量。在这个 Shell 会话中，我们使用了两个使输出更简洁的*tcpdump*选项。* -t *选项禁止显示时间戳信息。* -N *选项使得主机名显示时不带有限定的域名。此外，为了简洁起见，并且因为我们没有描述 TCP 选项的细节，我们已从*tcpdump*输出的行中删除了*options*字段。

服务器在端口 55555 上运行，因此我们的*tcpdump*命令选择了该端口的流量。输出显示了在连接建立过程中交换的三个报文段：

```
$ `tcpdump -t -N 'port 55555'`
IP pukaki.60391 > tekapo.55555: S 3412991013:3412991013(0) win 5840
IP tekapo.55555 > pukaki.60391: S 1149562427:1149562427(0) ack 3412991014 win 5792
IP pukaki.60391 > tekapo.55555: . ack 1 win 5840
```

这三个报文段是为了三次握手而交换的 SYN、SYN/ACK 和 ACK 报文段（参见图 61-5）。

在以下输出中，客户端发送给服务器两条消息，分别包含 16 字节和 32 字节，而服务器分别以 4 字节的消息做出响应：

```
IP pukaki.60391 > tekapo.55555: P 1:17(16) ack 1 win 5840
IP tekapo.55555 > pukaki.60391: . ack 17 win 1448
IP tekapo.55555 > pukaki.60391: P 1:5(4) ack 17 win 1448
IP pukaki.60391 > tekapo.55555: . ack 5 win 5840
IP pukaki.60391 > tekapo.55555: P 17:49(32) ack 5 win 5840
IP tekapo.55555 > pukaki.60391: . ack 49 win 1448
IP tekapo.55555 > pukaki.60391: P 5:9(4) ack 49 win 1448
IP pukaki.60391 > tekapo.55555: . ack 9 win 5840
```

对于每个数据段，我们可以看到反方向发送的 ACK。

最后，我们展示了在连接终止过程中交换的报文段（首先，客户端关闭连接的一端，然后服务器关闭另一端）：

```
IP pukaki.60391 > tekapo.55555: F 49:49(0) ack 9 win 5840
IP tekapo.55555 > pukaki.60391: . ack 50 win 1448
IP tekapo.55555 > pukaki.60391: F 9:9(0) ack 50 win 1448
IP pukaki.60391 > tekapo.55555: . ack 10 win 5840
```

上述输出显示了在连接终止过程中交换的四个报文段（参见图 61-6）。

## 套接字选项

套接字选项影响套接字操作的各个特性。在本书中，我们仅描述了几种可用的套接字选项。关于大多数标准套接字选项的详细讨论请参见[Stevens 等人，2004]。有关更多 Linux 特定细节，请参阅*tcp(7)*、*udp(7)*、*ip(7)*、*socket(7)*和*unix(7)*手册页。

*setsockopt()*和*getsockopt()*系统调用用于设置和获取套接字选项。

```
#include <sys/socket.h>

int `getsockopt`(int *sockfd*, int *level*, int *optname*, void **optval*,
               socklen_t **optlen*);
int `setsockopt`(int *sockfd*, int *level*, int *optname*, const void **optval*,
               socklen_t *optlen*);
```

### 注意

两者在成功时返回 0，出错时返回-1。

对于*setsockopt()*和*getsockopt()*，*sockfd*是一个指向套接字的文件描述符。

*level*参数指定套接字选项适用的协议——例如，IP 或 TCP。对于本书中描述的大多数套接字选项，*level*被设置为`SOL_SOCKET`，表示该选项适用于套接字 API 层。

*optname*参数标识我们希望设置或检索其值的选项。*optval*参数是一个指向缓冲区的指针，用于指定或返回选项值；根据选项，该参数是指向整数或结构的指针。

*optlen*参数指定了*optval*指向的缓冲区的大小（以字节为单位）。对于*setsockopt()*，该参数按值传递。对于*getsockopt()*，*optlen*是一个值-结果参数。在调用之前，我们将其初始化为*optval*指向的缓冲区的大小；返回时，它被设置为实际写入该缓冲区的字节数。

如《*accept()*跨越标志和选项的继承》*跨越标志和选项的继承》")中详细描述，*accept()*调用返回的套接字文件描述符会继承监听套接字的可设置套接字选项值。

套接字选项与一个打开的文件描述符相关联（参见图 5-2，在文件描述符与打开文件之间的关系中）。这意味着通过*dup()*（或类似操作）或*fork()*复制的文件描述符共享相同的套接字选项集。

一个简单的套接字选项示例是`SO_TYPE`，它可以用来查看套接字的类型，示例如下：

```
int optval;
socklen_t optlen;

optlen = sizeof(optval);
if (getsockopt(sfd, SOL_SOCKET, SO_TYPE, &optval, &optlen) == -1)
    errExit("getsockopt");
```

在此调用后，*optval*包含套接字类型——例如，`SOCK_STREAM`或`SOCK_DGRAM`。在一个通过*exec()*继承了套接字文件描述符的程序中使用此调用非常有用——例如，由*inetd*启动的程序——因为该程序可能不知道它继承了哪种类型的套接字。

`SO_TYPE`是一个只读套接字选项的示例。无法使用*setsockopt()*更改套接字的类型。

## *SO_REUSEADDR*套接字选项

`SO_REUSEADDR`套接字选项有多种用途（详见第七章，[Stevens 等，2004]）。我们只关心其中一个常见用法：避免在 TCP 服务器重新启动并尝试将套接字绑定到当前有关联 TCP 的端口时出现`EADDRINUSE`（“地址已在使用”）错误。通常发生这种情况的有两种情况：

+   之前对连接到客户端的服务器的调用执行了主动关闭操作，可能是通过调用*close()*，也可能是由于崩溃（例如，它被信号杀死）。这会导致一个 TCP 端点保持在 TIME_WAIT 状态，直到 2MSL 超时到期。

+   之前对服务器的调用创建了一个子进程来处理与客户端的连接。稍后，服务器终止，而子进程继续为客户端提供服务，从而保持使用服务器著名端口的 TCP 端点。

在这两种情况下，未完成的 TCP 端点无法接受新的连接。然而，在这两种情况下，默认情况下，大多数 TCP 实现都会阻止将新的监听套接字绑定到服务器的著名端口。

### 注意

`EADDRINUSE`错误通常不会出现在客户端，因为它们通常使用一个临时端口，该端口不会是当前在 TIME_WAIT 状态的端口之一。然而，如果客户端绑定到特定端口号，它也可能会遇到此错误。

为了理解`SO_REUSEADDR`套接字选项的操作，可以帮助回顾我们之前关于流套接字的电话类比（流套接字）。就像电话通话（我们忽略了电话会议的概念），TCP 套接字连接是通过一对连接端点的*组合*来标识的。*accept()*的操作类似于电话接线员在内部公司总机上执行的任务（“服务器”）。当外部电话到达时，接线员将其转接到公司内部的某个电话（“新套接字”）。从外部的角度来看，无法识别这个内部电话。当多个外部电话通过总机处理时，区分它们的唯一方法是通过外部来电者的号码和总机号码的组合。（考虑到整个电话网络中将会有多个公司总机时，后者是必要的。）类似地，每次我们在监听套接字上接受一个套接字连接时，都会创建一个新的套接字。所有这些套接字都与监听套接字使用相同的本地地址。区分它们的唯一方法是通过它们与不同对等套接字的连接。

换句话说，一个已连接的 TCP 套接字是通过以下形式的四元组（即四个值的组合）来标识的：

```
{ local-IP-address, local-port, foreign-IP-address, foreign-port }
```

TCP 规范要求每个这样的四元组是唯一的；也就是说，只有一个对应的连接实例（“电话通话”）可以存在。问题是，大多数实现（包括 Linux）强制执行更严格的约束：如果主机上存在具有匹配本地端口的任何 TCP 连接实例，则无法重用本地端口（即在调用*bind()*时指定）。即使 TCP 无法接受新的连接，也会强制执行这一规则，就像本节开头描述的场景一样。

启用`SO_REUSEADDR`套接字选项放宽了此约束，使其更接近 TCP 要求。默认情况下，此选项的值为 0，意味着它是禁用的。我们通过在绑定套接字之前赋予它一个非零值来启用此选项，如示例 61-4 所示。

设置`SO_REUSEADDR`选项意味着即使在本节开头描述的两种情况中的任何一种中，另一个 TCP 已经绑定到相同的端口，我们仍然可以将套接字绑定到本地端口。大多数 TCP 服务器应该启用此选项。我们已经在示例 59-6（第 1221 页）和示例 59-9（第 1228 页）中看到了一些使用此选项的例子。

示例 61-4. 设置`SO_REUSEADDR`套接字选项

```
int sockfd, optval;

    sockfd = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd == -1)
        errExit("socket");

    optval = 1;
    if (setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, &optval,
            sizeof(optval)) == -1)
        errExit("socket");

    if (bind(sockfd, &addr, addrlen) == -1)
        errExit("bind");
    if (listen(sockfd, backlog) == -1)
        errExit("listen");
```

## 标志和选项在*accept()*中的继承

各种标志和设置可以与打开的文件描述符和文件描述符关联（文件描述符与打开文件的关系）。此外，正如在套接字选项中所描述的，可以为套接字设置各种选项。如果这些标志和选项设置在监听套接字上，它们会被*accept()*返回的新套接字继承吗？我们在这里描述了详细信息。

在 Linux 上，以下属性不会被*accept()*返回的新文件描述符继承：

+   与打开的文件描述符相关的状态标志—这些标志可以通过*fcntl()*的`F_SETFL`操作进行更改（打开文件状态标志）。其中包括如`O_NONBLOCK`和`O_ASYNC`等标志。

+   文件描述符标志—这些标志可以通过*fcntl()*的`F_SETFD`操作进行更改。唯一的此类标志是执行时关闭标志（`FD_CLOEXEC`，在文件描述符和*exec()*")中有描述）。

+   *fcntl()*中的`F_SETOWN`（所有者进程 ID）和`F_SETSIG`（生成信号）与信号驱动 I/O 相关的文件描述符属性（信号驱动 I/O）。

另一方面，*accept()*返回的新描述符会继承大多数可以使用*setsockopt()*设置的套接字选项的副本（套接字选项）。

SUSv3 对这里描述的细节保持沉默，以及由*accept()*返回的新连接套接字的继承规则在 UNIX 实现中有所不同。 特别是，在某些 UNIX 实现中，如果在侦听套接字上设置了诸如`O_NONBLOCK`和`O_ASYNC`等开放文件状态标志，则它们将被*accept()*返回的新套接字继承。 为了可移植性，可能需要显式地重置*accept()*返回的套接字上的这些属性。

## TCP 与 UDP

鉴于 TCP 提供数据的可靠传输，而 UDP 则不提供，一个显而易见的问题是，“为什么要使用 UDP 呢？” 这个问题的答案在第二十二章中有详细讨论，见于[Stevens 等人，2004 年]。 在这里，我们总结了一些可能导致我们选择 UDP 而不是 TCP 的要点：

+   UDP 服务器可以从多个客户端接收（和回复）数据报，而无需为每个客户端创建和终止连接（即，使用 UDP 传输单个消息的开销低于使用 TCP 时所需的开销）。

+   对于简单的请求-响应通信，UDP 可能比 TCP 更快，因为它不需要建立和终止连接。 附录 A 提到，在最佳情况下，使用 TCP 的时间是

    ```
    2 * RTT + SPT
    ```

    在这个公式中，RTT 是往返时间（发送请求并接收响应所需的时间），而 SPT 是服务器处理请求所花费的时间。 （在广域网上，与 RTT 相比，SPT 值可能很小。）对于 UDP，单个请求-响应通信的最佳情况是

    ```
    RTT + SPT
    ```

    这比 TCP 所需的时间少一个 RTT。 由于在大范围（即跨洲际）距离或许多中间路由器之间分隔的主机之间的 RTT 通常是几十分之一秒，这种差异可以使 UDP 对某些类型的请求-响应通信具有吸引力。 DNS 是一个使用 UDP 的好例子，原因在于使用 UDP 允许在服务器之间的每个方向传输单个数据包来执行名称查找。

+   UDP 套接字允许广播和多播。 *广播* 允许发送者向连接到网络的所有主机的相同目的端口传输数据报。 *多播* 类似，但允许将数据报发送到指定的一组主机。 更多细节请参见第二十一章和第二十二章，见于[Stevens 等人，2004 年]。

+   某些类型的应用程序（例如，视频和音频流传输）在没有 TCP 提供的可靠性的情况下也能正常工作。另一方面，TCP 尝试从丢失的数据段恢复后的延迟可能导致不可接受的传输延迟。（流媒体传输的延迟可能比传输流的短暂丢失更糟。）因此，这些应用程序可能更倾向于使用 UDP，并采用特定于应用的恢复策略来应对偶尔的数据包丢失。

使用 UDP 的应用程序，如果仍然需要可靠性，必须自行实现可靠性功能。通常，这至少需要序列号、确认、丢失数据包的重传和重复检测。如何实现这一点的示例可以参见[Stevens et al., 2004]。然而，如果还需要更高级的功能，如流量控制和拥塞控制，那么最好使用 TCP。尝试在 UDP 上实现所有这些功能是复杂的，即使实现得很好，结果也不太可能比 TCP 表现更好。

## 高级功能

UNIX 和互联网域套接字还有许多其他特性，我们在本书中没有详细介绍。我们在本节中总结了一些这些特性。有关详细信息，请参见[Stevens et al., 2004]。

### 带外数据

带外数据是流套接字的一项功能，它允许发送方将传输的数据标记为高优先级；即接收方可以在不需要读取流中所有中间数据的情况下获得带外数据可用的通知。这个功能在如*telnet、rlogin*和*ftp*等程序中使用，使得可以中止之前传输的命令。带外数据通过在调用*send()*和*recv()*时使用`MSG_OOB`标志来发送和接收。当套接字接收到带外数据可用的通知时，内核会为套接字的拥有者（通常是使用该套接字的进程）生成`SIGURG`信号，这是由*fcntl()*的`F_SETOWN`操作设置的。

在与 TCP 套接字一起使用时，每次最多只能标记 1 字节的数据为带外数据。如果发送方在接收方处理完前一个字节之前传输了额外的带外数据字节，那么前一个带外字节的指示将丢失。

### 注意

TCP 对带外数据限制为单个字节，这是套接字 API 的通用带外模型与其使用 TCP 的*紧急模式*的具体实现之间不匹配的结果。我们在 TCP 段格式中提到过 TCP 的紧急模式。TCP 通过设置 TCP 头中的 URG 位并将紧急指针字段指向紧急数据，来指示带外（紧急）数据的存在。然而，TCP 无法指示紧急数据序列的长度，因此紧急数据被认为由单个字节组成。

关于 TCP 带外数据的更多信息可以在 RFC 793 中找到。

在一些 UNIX 实现中，UNIX 域流套接字支持带外数据。Linux 不支持此功能。

目前不推荐使用带外数据，在某些情况下可能不可靠（参见[Gont & Yourtchenko, 2009]）。一种替代方案是保持一对流套接字用于通信。其中一个用于正常通信，另一个用于高优先级通信。应用程序可以使用在第六十三章中描述的技术之一来监控这两个通道。这种方法允许传输多个字节的优先级数据。此外，它可以与任何通信域中的流套接字一起使用（例如，UNIX 域套接字）。

### *sendmsg()* 和 *recvmsg()* 系统调用

*sendmsg()* 和 *recvmsg()* 系统调用是套接字 I/O 系统调用中最通用的。*sendmsg()* 系统调用可以执行*write()*、*send()*和*sendto()*所做的所有操作；*recvmsg()* 系统调用可以执行*read()*、*recv()*和*recvfrom()*所做的所有操作。此外，这些调用还允许以下操作：

+   我们可以执行散列-聚集 I/O，类似于*readv()*和*writev()*（散列-聚集 I/O: *readv()* 和 *writev()* 和 writev()")）。当我们使用*sendmsg()*在数据报套接字上执行聚集输出时（或在连接的 UDP 套接字上使用*writev()*），会生成一个数据报。相反，*recvmsg()*（和*readv()*）可以用于在数据报套接字上执行散列输入，将单个数据报的字节分散到多个用户空间缓冲区中。

+   我们可以传输包含特定领域的*附加数据*（也称为控制信息）的消息。附加数据可以通过流套接字和数据报套接字传递。我们在下面描述了一些附加数据的示例。

### 注意

Linux 2.6.33 新增了一个系统调用，*recvmmsg()*。这个系统调用与*recvmsg()*类似，但允许在一次系统调用中接收多个数据报文。这减少了在处理高网络流量的应用程序中系统调用的开销。一个类似的*sendmmsg()*系统调用可能会在未来的内核版本中添加。

### 传递文件描述符

使用*sendmsg()*和*recvmsg()*，我们可以通过 UNIX 域套接字将包含文件描述符的辅助数据从一个进程传递到同一主机上的另一个进程。任何类型的文件描述符都可以通过这种方式传递——例如，通过调用*open()*或*pipe()*获得的文件描述符。一个与套接字更相关的示例是，主服务器可以在 TCP 监听套接字上接受客户端连接，并将该描述符传递给服务器子进程池中的一个成员（其他并发服务器设计），该子进程随后会响应客户端请求。

尽管这种技术通常被称为传递文件描述符，但实际上在两个进程之间传递的是对同一打开文件描述的引用（图 5-2，在文件描述符与打开文件之间的关系中）。接收进程中使用的文件描述符号通常与发送方使用的文件描述符号不同。

### 注意

在本书的源代码分发包中的`sockets`子目录下，文件`scm_rights_send.c`和`scm_rights_recv.c`提供了传递文件描述符的示例。

### 接收发送方凭证

使用辅助数据的另一个示例是通过 UNIX 域套接字接收发送方凭证。这些凭证包括发送进程的用户 ID、组 ID 和进程 ID。发送方可以指定其用户 ID 和组 ID 作为对应的真实 ID、有效 ID 或保存 ID。这使得接收进程能够验证同一主机上的发送方。更多详细信息，请参见*socket(7)*和*unix(7)*手册页。

与传递文件凭证不同，传递发送方凭证在 SUSv3 中并未指定。除了 Linux 外，这一功能在一些现代 BSD 系统中得到了实现（其中凭证结构包含的信息比 Linux 中的更多），但在其他一些 UNIX 实现中则较为罕见。有关 FreeBSD 上凭证传递的详细信息，请参见[Stevens 等人，2004]。

在 Linux 中，如果特权进程具有相应的`CAP_SETUID`、`CAP_SETGID`和`CAP_SYS_ADMIN`能力，它可以伪造作为凭证传递的用户 ID、组 ID 和进程 ID。 

### 注意

一个传递凭证的示例可以在源代码分发包中的 `sockets` 子目录下的 `scm_cred_send.c` 和 `scm_cred_recv.c` 文件中找到。

### 顺序数据包套接字

顺序数据包套接字结合了流套接字和数据报套接字的特性：

+   与流套接字类似，顺序数据包套接字是面向连接的。连接的建立方式与流套接字相同，使用 *bind()*、*listen()*、*accept()* 和 *connect()*。

+   像数据报套接字一样，消息边界被保留。从顺序数据包套接字的 *read()* 操作中读取时，将返回一个完整的消息（如对等方所写）。如果消息的长度超过了调用者提供的缓冲区，则多余的字节会被丢弃。

+   像流套接字一样，顺序数据包套接字与数据报套接字不同，通信是可靠的。消息将按顺序、无错误、不重复地传递给对等应用程序，并保证到达（假设没有系统或应用程序崩溃，或网络中断）。

通过调用 *socket()* 并将 *type* 参数指定为 `SOCK_SEQPACKET` 来创建顺序数据包套接字。

从历史上看，Linux 和大多数 UNIX 实现一样，既不支持 UNIX 域也不支持互联网域中的顺序数据包套接字。然而，从内核 2.6.4 开始，Linux 支持 UNIX 域套接字中的 `SOCK_SEQPACKET`。

在互联网领域，UDP 和 TCP 协议不支持 `SOCK_SEQPACKET`，但 SCTP 协议（将在下一节中介绍）支持。

本书中没有展示顺序数据包套接字的使用示例，但除了保留消息边界外，它们的使用方式与流套接字非常相似。

### SCTP 和 DCCP 传输层协议

SCTP 和 DCCP 是两种较新的传输层协议，未来可能会变得越来越常见。

*流控制传输协议*（SCTP，[`www.sctp.org/`](http://www.sctp.org/)）最初是为了支持电话信令而设计的，但它也是通用的。像 TCP 一样，SCTP 提供可靠的、双向的、面向连接的传输。与 TCP 不同，SCTP 保留了消息边界。SCTP 的一个显著特征是支持多流，可以在单个连接上使用多个逻辑数据流。

SCTP 在 [Stewart & Xie, 2001]、[Stevens et al., 2004] 和 RFC 4960、3257、3286 中有描述。

自 Linux 内核 2.6 版本起，SCTP 可在 Linux 上使用。有关此实现的更多信息，请访问 [`lksctp.sourceforge.net/`](http://lksctp.sourceforge.net/)。

在前面描述套接字 API 的章节中，我们将互联网域的流套接字与 TCP 等同。然而，SCTP 提供了一种替代协议来实现流套接字，可以通过以下调用创建：

```
socket(AF_INET, SOCK_STREAM, IPPROTO_SCTP);
```

从内核 2.6.14 开始，Linux 支持一种新的数据报协议——*数据报拥塞控制协议*（DCCP）。像 TCP 一样，DCCP 提供了拥塞控制（避免需要在应用层实现拥塞控制），防止快速传输者压垮网络。（我们在讲解 TCP 时已解释了拥塞控制，参见 传输控制协议 (TCP)")。）然而，与 TCP 不同（但像 UDP 一样），DCCP 不提供可靠或按顺序交付的保证，因此允许不需要这些特性的应用程序避免由此产生的延迟。关于 DCCP 的信息可以在 [`www.read.cs.ucla.edu/dccp/`](http://www.read.cs.ucla.edu/dccp/) 和 RFC 4336、4340 中找到。

## 总结

在不同的情况下，执行流套接字的 I/O 操作时可能会发生部分读取和写入。我们展示了两个函数的实现，*readn()* 和 *writen()*，可以确保完整的缓冲区数据被读取或写入。

*shutdown()* 系统调用提供了更精确的连接终止控制。通过使用 *shutdown()*，我们可以强制关闭双向通信流的任意一半或两半，无论是否存在其他指向该套接字的打开文件描述符。

与 *read()* 和 *write()* 类似，*recv()* 和 *send()* 可用于在套接字上执行 I/O 操作，但这些调用提供了一个额外的参数 *flags*，用于控制套接字特定的 I/O 功能。

*sendfile()* 系统调用允许我们高效地将文件内容复制到套接字。这种效率是因为我们不需要像调用 *read()* 和 *write()* 时那样，将文件数据复制到用户内存和从用户内存复制回来。

*getsockname()* 和 *getpeername()* 系统调用分别用于检索套接字绑定的本地地址和与该套接字连接的对端地址。

我们考虑了 TCP 操作的一些细节，包括 TCP 状态、TCP 状态转换图以及 TCP 连接的建立和终止。在这部分讨论中，我们了解了为什么 TIME_WAIT 状态是 TCP 可靠性保证的重要组成部分。尽管这个状态在重启服务器时可能导致“地址已在使用”错误，但我们后来发现可以使用 `SO_REUSEADDR` 套接字选项来避免此错误，同时仍然允许 TIME_WAIT 状态发挥其预期作用。

*netstat* 和 *tcpdump* 命令是监控和调试使用套接字的应用程序的有用工具。

*getsockopt()* 和 *setsockopt()* 系统调用用于检索和修改影响套接字操作的选项。

在 Linux 上，当一个新的套接字由 *accept()* 创建时，它不会继承监听套接字的打开文件状态标志、文件描述符标志或与信号驱动 I/O 相关的文件描述符属性。然而，它确实会继承套接字选项的设置。我们注意到 SUSv3 对这些细节保持沉默，而这些细节在不同的实现中有所不同。

虽然 UDP 没有提供 TCP 的可靠性保证，但我们看到仍然有一些原因表明，UDP 在某些应用中可能更为优选。

最后，我们概述了几项套接字编程的高级功能，但在本书中并未详细描述。

#### 进一步信息

请参阅进一步信息中列出的其他信息来源。

## 练习

1.  假设 示例 61-2 (`is_echo_cl.c`) 中的程序被修改，改为使用一个进程，首先将标准输入复制到套接字，然后读取服务器的响应，而不是使用 *fork()* 创建两个并发操作的进程。那么，运行此客户端时可能会出现什么问题？（请查看 图 58-8，选择一个 UDP 数据报大小以避免 IP 分片。）

1.  使用 *socketpair()* 来实现 *pipe()*。使用 *shutdown()* 确保结果管道是单向的。

1.  使用*read()*、*write()* 和 *lseek()* 实现 *sendfile()* 的替代方案。

1.  编写一个程序，使用*getsockname()* 来展示，如果我们在 TCP 套接字上调用 *listen()* 而没有先调用 *bind()*，套接字将被分配一个临时端口号。

1.  编写一个客户端和一个服务器，允许客户端在服务器主机上执行任意的 shell 命令。（如果在这个应用程序中没有实现任何安全机制，应该确保服务器在一个用户帐户下运行，该帐户在受到恶意用户调用时不会造成任何损害。）客户端应该使用两个命令行参数执行：

    ```
    $ `./is_shell_cl` ``*`server-host`*```'```*`some-shell-command`*```'`

    ```

    连接到服务器后，客户端将给定的命令发送到服务器，然后使用 *shutdown()* 关闭套接字的写入端，这样服务器就会看到文件结束符。服务器应在一个单独的子进程中处理每个传入的连接（即并发设计）。对于每个传入的连接，服务器应从套接字中读取命令（直到文件结束符），然后执行一个 shell 来执行该命令。以下是几个提示：

    +   请参阅实现 *system()*")中的 *system()* 实现示例，以了解如何执行 shell 命令。

    +   通过使用*dup2()*将套接字复制到标准输出和标准错误，execed 命令将自动写入套接字。

1.  带外数据提到，带外数据的替代方法是创建两个客户端和服务器之间的套接字连接：一个用于正常数据，另一个用于优先数据。编写客户端和服务器程序来实现这个框架。以下是一些提示：

    +   服务器需要某种方法来知道哪两个套接字属于同一个客户端。一种方法是让客户端首先使用临时端口（即绑定到端口 0）创建一个监听套接字。在获取到其监听套接字的临时端口号（使用*getsockname()*）后，客户端将其“正常”套接字连接到服务器的监听套接字，并发送包含客户端监听套接字端口号的消息。然后，客户端等待服务器使用客户端的监听套接字在相反方向建立连接，以获取“优先级”套接字。（服务器可以在*accept()*正常连接时获取客户端的 IP 地址。）

    +   实现某种类型的安全机制，防止恶意进程尝试连接到客户端的监听套接字。为此，客户端可以通过正常套接字向服务器发送一个 cookie（即某种唯一消息）。然后，服务器通过优先级套接字返回该 cookie，以便客户端进行验证。

    +   为了实验从客户端到服务器传输正常数据和优先数据，你需要编写服务器代码，通过*select()*或*poll()*对来自两个套接字的输入进行多路复用（如在 I/O 多路复用中所描述的）。
