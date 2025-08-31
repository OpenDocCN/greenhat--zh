## 第六十章. 套接字：服务器设计

本章讨论了设计迭代型和并发型服务器的基本原理，并描述了 *inetd*，这是一个专门的守护进程，用于简化互联网服务器的创建。

## 迭代型和并发型服务器

使用套接字设计网络服务器的两种常见方式如下：

+   *迭代*：服务器一次处理一个客户端，完全处理该客户端的请求后，再处理下一个客户端。

+   *并发*：服务器设计用于同时处理多个客户端请求。

我们已经在 使用 FIFO 的客户端-服务器应用 中看到了一个使用 FIFO 的迭代型服务器示例，并在第 46.8 节中看到了一个使用 System V 消息队列的并发型服务器示例。

迭代型服务器通常只适用于那些客户端请求可以快速处理的场景，因为每个客户端必须等待所有前面的客户端请求处理完毕后才能被服务。使用迭代型服务器的典型场景是客户端和服务器之间交换单个请求和响应。

并发型服务器适用于需要较长处理时间来处理每个请求，或者客户端和服务器进行长时间的交互，不断交换消息的场景。在本章中，我们主要关注设计并发型服务器的传统方法（也是最简单的）：为每个新客户端创建一个新的子进程。每个服务器子进程执行所有必要的任务来服务单个客户端，然后终止。由于这些进程可以独立运行，因此可以同时处理多个客户端。主服务器进程（父进程）的主要任务是为每个新客户端创建一个新的子进程。（这种方法的变体是为每个客户端创建一个新的线程。）

在接下来的章节中，我们将展示使用互联网域套接字的迭代型和并发型服务器示例。这两个服务器实现了 *回显* 服务（RFC 862），这是一个基本服务，返回客户端发送的任何内容的副本。

## 一个迭代的 UDP *回显* 服务器

在本节和下一节中，我们将介绍 *回显* 服务的服务器。*回显* 服务在 UDP 和 TCP 端口 7 上运行。（由于这是一个保留端口，*回显* 服务器必须以超级用户权限运行。）

UDP *回显* 服务器持续读取数据报，并将每个数据报的副本返回给发送者。由于服务器一次只需要处理一个消息，迭代型服务器设计足够满足需求。服务器的头文件如 示例 60-1 所示。

示例 60-1. `id_echo_sv.c` 和 `id_echo_cl.c` 的头文件

```
`sockets/id_echo.h`
#include "inet_sockets.h"       /* Declares our socket functions */
#include "tlpi_hdr.h"

#define SERVICE "echo"          /* Name of UDP service */

#define BUF_SIZE 500            /* Maximum size of datagrams that can
                                   be read by client and server */
      `sockets/id_echo.h`
```

示例 60-2 展示了服务器的实现。请注意以下几点关于服务器实现的内容：

+   我们使用创建守护进程中的*becomeDaemon()*函数将服务器转变为守护进程。

+   为了简化该程序，我们采用了在互联网域套接字库中开发的互联网域套接字库。

+   如果服务器无法向客户端发送回复，它会使用*syslog()*记录一条消息。

### 注意

在实际应用中，我们可能会对使用*syslog()*写入的消息应用一些速率限制，既是为了防止攻击者填满系统日志，也是因为每次调用*syslog()*都比较昂贵，因为（默认情况下）*syslog()*又会调用*fsync()*。

示例 60-2。实现 UDP *echo*服务的迭代服务器

```
`sockets/id_echo_sv.c`
#include <syslog.h>
#include "id_echo.h"
#include "become_daemon.h"

int
main(int argc, char *argv[])
{
    int sfd;
    ssize_t numRead;
    socklen_t addrlen, len;
    struct sockaddr_storage claddr;
    char buf[BUF_SIZE];
    char addrStr[IS_ADDR_STR_LEN];

    if (becomeDaemon(0) == -1)
        errExit("becomeDaemon");

    sfd = inetBind(SERVICE, SOCK_DGRAM, &addrlen);
    if (sfd == -1) {
        syslog(LOG_ERR, "Could not create server socket (%s)", strerror(errno));
        exit(EXIT_FAILURE);
    }

    /* Receive datagrams and return copies to senders */

    for (;;) {
        len = sizeof(struct sockaddr_storage);
        numRead = recvfrom(sfd, buf, BUF_SIZE, 0,
                           (struct sockaddr *) &claddr, &len);
        if (numRead == -1)
            errExit("recvfrom");

        if (sendto(sfd, buf, numRead, 0, (struct sockaddr *) &claddr, len)
                    != numRead)
            syslog(LOG_WARNING, "Error echoing response to %s (%s)",
                    inetAddressStr((struct sockaddr *) &claddr, len,
                                   addrStr, IS_ADDR_STR_LEN),
                    strerror(errno));
    }
}
     `sockets/id_echo_sv.c`
```

为了测试服务器，我们使用示例 60-3 中显示的客户端程序。该程序也使用了在互联网域套接字库中开发的互联网域套接字库。作为第一个命令行参数，客户端程序期望接收到服务器所在主机的名称。客户端会执行一个循环，将剩余的每个命令行参数作为独立的数据报发送给服务器，并读取并打印每个由服务器返回的响应数据报。

示例 60-3。UDP *echo*服务的客户端

```
`sockets/id_echo_cl.c`
#include "id_echo.h"

int
main(int argc, char *argv[])
{
    int sfd, j;
    size_t len;
    ssize_t numRead;
    char buf[BUF_SIZE];

    if (argc < 2 || strcmp(argv[1], "--help") == 0)
        usageErr("%s: host msg...\n", argv[0]);

    /* Construct server address from first command-line argument */

    sfd = inetConnect(argv[1], SERVICE, SOCK_DGRAM);
    if (sfd == -1)
        fatal("Could not connect to server socket");

    /* Send remaining command-line arguments to server as separate datagrams */

    for (j = 2; j < argc; j++) {
        len = strlen(argv[j]);
        if (write(sfd, argv[j], len) != len)
            fatal("partial/failed write");

        numRead = read(sfd, buf, BUF_SIZE);
        if (numRead == -1)
            errExit("read");

        printf("[%ld bytes] %.*s\n", (long) numRead, (int) numRead, buf);
    }

    exit(EXIT_SUCCESS);
}
     `sockets/id_echo_cl.c`
```

以下是我们运行服务器和两个客户端实例时看到的例子：

```
$ `su`                                      *Need privilege to bind reserved port*
Password:
# `./id_echo_sv`                            *Server places itself in background*
# `exit`                                    *Cease to be superuser*
$ `./id_echo_cl localhost hello world`      *This client sends two datagrams*
[5 bytes] hello                           *Client prints responses from server*
[5 bytes] world
$ `./id_echo_cl localhost goodbye`          *This client sends one datagram*
[7 bytes] goodbye
```

## 一个并发的 TCP *echo*服务器

TCP *echo*服务也运行在 7 端口。TCP *echo*服务器接受一个连接后，开始持续循环，读取所有传输的数据并通过同一套接字将其返回给客户端。服务器继续读取直到检测到文件结束，此时会关闭其套接字（这样客户端如果仍在从套接字读取，会看到文件结束）。

由于客户端可能会向服务器发送无限量的数据（因此为客户端提供服务可能需要无限时间），因此并发服务器设计是合适的，以便多个客户端可以同时得到服务。服务器的实现见示例 60-4。我们将在第 61.2 节中展示该服务的客户端实现。关于该实现，请注意以下几点：

+   服务器通过调用第 37.2 节中显示的*becomeDaemon()*函数成为一个守护进程。

+   为了简化程序，我们使用了在示例 59-9（第 1228 页）中展示的互联网域套接字库。

+   由于服务器为每个客户端连接创建一个子进程，因此我们必须确保僵尸进程得到回收。我们在 `SIGCHLD` 处理程序中执行这项操作。

+   服务器的主体部分由一个 `for` 循环组成，该循环接受客户端连接，然后使用 *fork()* 创建一个子进程，子进程调用 *handleRequest()* 函数来处理该客户端。同时，父进程继续执行 `for` 循环，接受下一个客户端连接。

    ### 注意

    在实际应用中，我们可能会在服务器中包含一些代码，限制服务器能够创建的子进程数量，以防止攻击者通过使用该服务创建大量进程，从而使系统变得无法使用，类似于远程 fork bomb 攻击。我们可以通过在服务器中添加额外的代码，来统计当前正在执行的子进程数量（每次成功调用 *fork()* 后该数量递增，每次子进程被回收时在 `SIGCHLD` 处理程序中递减）。如果子进程数量达到了上限，我们可以暂时停止接受连接（或者，接受连接后立即关闭它们）。

+   在每次调用 *fork()* 后，监听和连接套接字的文件描述符会在子进程中被复制（参见父子进程间的文件共享）。这意味着父进程和子进程都可以使用连接的套接字与客户端进行通信。然而，只有子进程需要执行这样的通信，因此父进程在 *fork()* 后会立即关闭连接套接字的文件描述符。（如果父进程不这样做，套接字将永远不会被关闭；此外，父进程最终会耗尽文件描述符。）由于子进程不接受新的连接，它会关闭它所复制的监听套接字的文件描述符。

+   每个子进程在处理完单个客户端后终止。

示例 60-4. 实现 TCP *回显* 服务的并发服务器

```
`sockets/is_echo_sv.c`
#include <signal.h>
#include <syslog.h>
#include <sys/wait.h>
#include "become_daemon.h"
#include "inet_sockets.h"       /* Declarations of inet*() socket functions */
#include "tlpi_hdr.h"

#define SERVICE "echo"          /* Name of TCP service */
#define BUF_SIZE 4096

static void             /* SIGCHLD handler to reap dead child processes */
grimReaper(int sig)
{
    int savedErrno;             /* Save 'errno' in case changed here */

    savedErrno = errno;
    while (waitpid(-1, NULL, WNOHANG) > 0)
        continue;
    errno = savedErrno;
}

/* Handle a client request: copy socket input back to socket */

static void
handleRequest(int cfd)
{
    char buf[BUF_SIZE];
    ssize_t numRead;

    while ((numRead = read(cfd, buf, BUF_SIZE)) > 0) {
        if (write(cfd, buf, numRead) != numRead) {
            syslog(LOG_ERR, "write() failed: %s", strerror(errno));
            exit(EXIT_FAILURE);
        }
    }

    if (numRead == -1) {
        syslog(LOG_ERR, "Error from read(): %s", strerror(errno));
        exit(EXIT_FAILURE);
    }
}

int
main(int argc, char *argv[])
{
    int lfd, cfd;               /* Listening and connected sockets */
    struct sigaction sa;

    if (becomeDaemon(0) == -1)
        errExit("becomeDaemon");

    sigemptyset(&sa.sa_mask);
    sa.sa_flags = SA_RESTART;
    sa.sa_handler = grimReaper;
    if (sigaction(SIGCHLD, &sa, NULL) == -1) {
        syslog(LOG_ERR, "Error from sigaction(): %s", strerror(errno));
        exit(EXIT_FAILURE);
    }

    lfd = inetListen(SERVICE, 10, NULL);
    if (lfd == -1) {
        syslog(LOG_ERR, "Could not create server socket (%s)", strerror(errno));
        exit(EXIT_FAILURE);
    }

    for (;;) {
        cfd = accept(lfd, NULL, NULL);  /* Wait for connection */
        if (cfd == -1) {
            syslog(LOG_ERR, "Failure in accept(): %s", strerror(errno));
            exit(EXIT_FAILURE);
        }

        /* Handle each client request in a new child process */

        switch (fork()) {
        case -1:
            syslog(LOG_ERR, "Can't create child (%s)", strerror(errno));
            close(cfd);                 /* Give up on this client */
            break;                      /* May be temporary; try next client */

        case 0:                         /* Child */
            close(lfd);                 /* Unneeded copy of listening socket */
            handleRequest(cfd);
            _exit(EXIT_SUCCESS);

        default:                        /* Parent */
            close(cfd);                 /* Unneeded copy of connected socket */
            break;                      /* Loop to accept next connection */
        }
    }
}
     `sockets/is_echo_sv.c`
```

## 其他并发服务器设计

前面章节中描述的传统并发服务器模型适用于许多需要通过 TCP 连接同时处理多个客户端的应用程序。然而，对于高负载的服务器（例如，每分钟处理成千上万请求的 Web 服务器），为每个客户端创建一个新的子进程（甚至是线程）会给服务器带来显著的负担（参见进程创建速度），因此需要采用替代设计。我们简要考虑一下这些替代方案。

#### 预派生和预线程化的服务器

预创建的进程和线程服务器在第三十章中有一些详细描述，出自[Stevens 等人，2004 年]。关键思想如下：

+   服务器不会为每个客户端创建新的子进程（或线程），而是在启动时预先创建固定数量的子进程（或线程）（即，在接收到任何客户端请求之前）。这些子进程构成了所谓的*服务器池*。

+   服务器池中的每个子进程一次处理一个客户端，但在处理完客户端后，子进程不会终止，而是获取下一个需要服务的客户端，并继续服务，依此类推。

使用上述技术需要在服务器应用程序中进行一些细致的管理。服务器池应该足够大，以确保能够响应客户端请求。这意味着服务器父进程必须监控空闲子进程的数量，并在高峰负载时增加池的大小，以便始终有足够的子进程可以立即服务新的客户端。如果负载减少，服务器池的大小应减少，因为系统中存在过多的进程会降低整体系统性能。

此外，服务器池中的子进程必须遵循某些协议，以便它们能够独占地选择各个客户端连接。在大多数 UNIX 实现（包括 Linux）中，足以让每个子进程在监听描述符上阻塞在*accept()*调用中。换句话说，服务器父进程在创建任何子进程之前，首先创建监听套接字，而每个子进程在*fork()*时会继承该套接字的文件描述符。当一个新的客户端连接到达时，只有一个子进程会完成*accept()*调用。然而，由于在某些旧的实现中，*accept()*并不是一个原子系统调用，因此该调用可能需要通过一些互斥技术（例如，文件锁）来确保只有一个子进程在同一时刻执行该调用（[Stevens 等人，2004 年]）。

### 注意

也有一些替代方法，避免让服务器池中的所有子进程执行*accept()*调用。如果服务器池由独立的进程组成，服务器父进程可以执行*accept()*调用，然后将包含新连接的文件描述符传递给池中一个空闲的进程，使用我们在传递文件描述符中简要描述的技术。如果服务器池由线程组成，主线程可以执行*accept()*调用，然后通知一个空闲的服务器线程，新客户端已连接到该描述符。

#### 处理来自单个进程的多个客户端

在某些情况下，我们可以设计一个单一服务器进程来处理多个客户端。为此，我们必须采用其中一种 I/O 模型（I/O 复用、信号驱动 I/O 或*epoll*），这些模型允许单个进程同时监控多个文件描述符上的 I/O 事件。这些模型在第六十三章中有所描述。

在单一服务器设计中，服务器进程必须承担一些通常由内核处理的调度任务。在每个客户端都有一个服务器进程的解决方案中，我们可以依赖内核来确保每个服务器进程（因此也包括每个客户端）能够公平地访问服务器主机的资源。但当我们使用单一服务器进程来处理多个客户端时，服务器必须做一些工作，确保一个或少数客户端不会垄断对服务器的访问，导致其他客户端无法获得资源。我们将在边缘触发通知中对此做进一步讨论。

#### 使用服务器集群

处理高客户端负载的其他方法涉及使用多个服务器系统——即*服务器集群*。

构建服务器集群的最简单方法之一（某些 Web 服务器采用）是*DNS 轮询负载共享*（或*负载分配*），在这种方式中，区域的权威名称服务器将相同的域名映射到多个 IP 地址（即，多个服务器共享同一个域名）。连续的 DNS 请求返回这些 IP 地址，并按轮询顺序进行。关于 DNS 轮询负载共享的更多信息可以在[Albitz & Liu, 2006]中找到。

轮询 DNS 的优点是成本低，且易于设置。然而，它也有一些缺点。一个执行迭代解析的 DNS 服务器可能会缓存其结果（参见域名系统（DNS）")），导致未来对该域名的查询返回相同的 IP 地址，而不是权威 DNS 服务器生成的轮询顺序。此外，轮询 DNS 没有内建机制来确保良好的负载均衡（不同客户端可能会对服务器施加不同负载）或确保高可用性（如果某个服务器死掉或它运行的服务器应用崩溃怎么办？）。另一个我们可能需要考虑的问题——这是许多采用多个服务器机器的设计面临的挑战——是确保*服务器亲和性*；也就是说，确保来自同一客户端的一系列请求都指向同一台服务器，这样服务器维护的关于该客户端的状态信息才能保持准确。

一个更灵活但也更复杂的解决方案是*服务器负载均衡*。在这种情况下，一个负载均衡服务器将传入的客户端请求路由到服务器农场中的某个成员。（为了确保高可用性，可能会有一个备用服务器，在主要负载均衡服务器崩溃时接管。）这消除了与远程 DNS 缓存相关的问题，因为服务器农场向外界呈现的是单一的 IP 地址（即负载均衡服务器的地址）。负载均衡服务器采用算法来测量或估计服务器负载（可能基于服务器农场成员提供的度量标准），并智能地将负载分配到服务器农场的各个成员。负载均衡服务器还会自动检测服务器农场成员的故障（以及如果需求增加时，添加新服务器）。最后，负载均衡服务器还可能支持服务器亲和性。有关服务器负载均衡的更多信息，请参阅[Kopparapu, 2002]。

## *inetd*（Internet Superserver）守护进程

如果我们查看`/etc/services`的内容，会看到字面上列出了数百个不同的服务。这意味着一个系统理论上可以运行大量的服务器进程。然而，这些服务器大多数情况下通常什么也不做，只是在等待偶尔的连接请求或数据报文。尽管如此，这些服务器进程仍然会占用内核进程表中的插槽，并消耗一定的内存和交换空间，从而给系统带来负担。

*inetd*守护进程旨在消除运行大量不常用的服务器的需求。使用*inetd*提供了两个主要的好处：

+   与其为每个服务运行单独的守护进程，不如让一个单独的进程——*inetd*守护进程——监控一组指定的套接字端口，并根据需要启动其他服务器。这样，系统上运行的进程数量得以减少。

+   由于*inetd*执行了启动时所有网络服务器通常需要的几个步骤，因此它简化了由*inetd*启动的服务器的编程。

由于它管理一系列服务，根据需要调用其他服务器，*inetd*有时被称为*互联网超级服务器*。

### 注意

在某些 Linux 发行版中提供了*inetd*的扩展版本——*xinetd*。*xinetd*增加了许多安全增强功能。有关*xinetd*的信息可以在[`www.xinetd.org/`](http://www.xinetd.org/)找到。

#### *inetd*守护进程的操作

*inetd*守护进程通常在系统启动时启动。在成为守护进程后（创建守护进程），*inetd*执行以下步骤：

1.  对于其配置文件中指定的每个服务，`/etc/inetd.conf` 中的 *inetd* 会创建一个适当类型的套接字（即流套接字或数据报套接字），并将其绑定到指定的端口。每个 TCP 套接字还会被标记为允许通过 *listen()* 调用接收传入连接。

1.  使用 *select()* 系统调用（*select()* 系统调用 系统调用")），*inetd* 监控在前一步中创建的所有套接字，等待数据报或传入的连接请求。

1.  *select()* 调用会阻塞，直到一个 UDP 套接字有可读的数据报，或一个连接请求在 TCP 套接字上接收到。在 TCP 连接的情况下，*inetd* 在继续执行下一步之前会执行 *accept()* 来接受该连接。

1.  为了启动为此套接字指定的服务器，*inetd()* 调用 *fork()* 创建一个新进程，然后执行 *exec()* 启动服务器程序。在执行 *exec()* 之前，子进程执行以下步骤：

    1.  关闭从父进程继承的所有文件描述符，除了用于接收 UDP 数据报或已接受的 TCP 连接的套接字文件描述符。

    1.  使用 复制文件描述符 中描述的技巧，将套接字文件描述符复制到文件描述符 0、1 和 2，并关闭套接字文件描述符本身（因为它不再需要）。在此步骤之后，执行的服务器可以通过这三个标准文件描述符在套接字上进行通信。

    1.  可选地，为被执行的服务器设置用户和组 ID，设置值来自 `/etc/inetd.conf` 文件。

1.  如果在步骤 3 中在 TCP 套接字上接受了连接，*inetd* 会关闭已连接的套接字（因为它只在执行的服务器中需要）。

1.  *inetd* 服务器返回到步骤 2。

#### `/etc/inetd.conf` 文件

*inetd* 守护进程的操作由配置文件控制，通常是 `/etc/inetd.conf`。该文件中的每一行描述了一个由 *inetd* 处理的服务。示例 60-5 显示了一个 Linux 发行版附带的 `/etc/inetd.conf` 文件中的一些示例条目。

示例 60-5. `/etc/inetd.conf` 文件中的示例行

```
# echo  stream  tcp  nowait  root    internal
# echo  dgram   udp  wait    root    internal
ftp     stream  tcp  nowait  root    /usr/sbin/tcpd   in.ftpd
telnet  stream  tcp  nowait  root    /usr/sbin/tcpd   in.telnetd
login   stream  tcp  nowait  root    /usr/sbin/tcpd   in.rlogind
```

示例 60-5 的前两行被初始的 `#` 字符注释掉；我们现在展示它们，因为我们将很快提到 *echo* 服务。

`/etc/inetd.conf` 文件的每一行由以下字段组成，字段之间由空白字符分隔：

+   *服务名称*：此项指定来自 `/etc/services` 文件的服务名称。与 *协议* 字段一起使用，用来查找 `/etc/services`，从中确定 *inetd* 应该监控该服务的端口号。

+   *套接字类型*：此项指定此服务使用的套接字类型——例如，`stream` 或 `dgram`。

+   *协议*：此项指定此套接字将使用的协议。该字段可以包含文件 `/etc/protocols` 中列出的任何互联网协议（在 *protocols(5)* 手册页中有文档），但几乎每个服务都指定 `tcp`（用于 TCP）或 `udp`（用于 UDP）。

+   *标志*：此字段包含 `wait` 或 `nowait`。此字段指定由 *inetd* 执行的服务器是否（临时）接管该服务的套接字管理。如果执行的服务器管理套接字，则此字段指定为 `wait`。这会导致 *inetd* 从它通过 *select()* 监视的文件描述符集移除该套接字，直到执行的服务器退出（*inetd* 通过处理 `SIGCHLD` 来检测这一点）。我们将在下面详细说明此字段。

+   *登录名*：此字段包含来自 `/etc/passwd` 的用户名，后面可选地跟一个点（`.`）和来自 `/etc/group` 的组名。这些确定了执行的服务器运行时的用户和组 ID。（由于 *inetd* 以 *root* 的有效用户 ID 运行，因此它的子进程也具有特权，进而可以使用 *setuid()* 和 *setgid()* 系统调用来更改进程的凭据（如果需要的话）。）

+   *服务器程序*：此项指定要执行的服务器程序的路径名。

+   *服务器程序参数*：此字段指定一个或多个参数，参数之间用空格分隔，当执行服务器程序时作为参数列表使用。这些参数中的第一个对应于执行程序中的 *argv[0]*，因此通常与 *服务器程序* 名称的基本名称部分相同。下一个参数对应于 *argv[1]*，以此类推。

### 注意

在 示例 60-5 中，对于 *ftp*、*telnet* 和 *login* 服务，我们看到服务器程序和参数的设置与之前描述的有所不同。这三项服务都导致 *inetd* 调用相同的程序，*tcpd(8)*（TCP 守护进程包装器），它在执行适当的程序之前，先进行一些日志记录和访问控制检查，这取决于作为第一个服务器程序参数指定的值（该值可以通过 *argv[0]* 提供给 *tcpd*）。有关 *tcpd* 的更多信息，请参阅 *tcpd(8)* 手册页以及 [Mann & Mitchell, 2003]。

由 *inetd* 调用的流式套接字（TCP）服务器通常设计为只处理单个客户端连接，然后终止，将进一步监听连接的任务留给 *inetd*。对于这样的服务器，*标志* 应指定为 `nowait`。（如果执行的服务器需要接受连接，则应指定为 `wait`，在这种情况下，*inetd* 不接受连接，而是将 *监听* 套接字的文件描述符作为描述符 0 传递给执行的服务器。）

对于大多数 UDP 服务器，应该将*flags*字段指定为`wait`。通过*inetd*调用的 UDP 服务器通常设计为读取并处理套接字上的所有未处理数据报，然后终止。（这通常需要某种类型的超时机制来读取套接字，以便在指定的时间间隔内没有新数据报到达时，服务器终止。）通过指定`wait`，我们可以防止*inetd*守护进程同时尝试对套接字执行*select()*，这样会导致*inetd*与 UDP 服务器争抢检查数据报，如果它赢得了竞争，就会启动另一个 UDP 服务器实例。

### 注意

由于*SUSv3*没有规定*inetd*的操作和其配置文件的格式，因此在`/etc/inetd.conf`字段中可以指定的值存在一些（通常很小的）差异。大多数版本的*inetd*至少提供了我们在正文中描述的语法。有关更多详细信息，请参见*inetd.conf(8)*手册页。

作为一种效率措施，*inetd*自己实现了一些简单的服务，而不是执行单独的服务器来执行任务。UDP 和 TCP *echo*服务就是*inetd*实现的服务示例。对于这些服务，相应`/etc/inetd.conf`记录中的*server program*字段被指定为`internal`，而*server program arguments*被省略。（在示例 60-5 中的示例行中，我们看到*echo*服务条目被注释掉了。要启用*echo*服务，我们需要删除这些行开头的`#`字符。）

每当我们更改`/etc/inetd.conf`文件时，需要向*inetd*发送`SIGHUP`信号，要求它重新读取该文件：

```
# `killall -HUP inetd`
```

#### 示例：通过*inetd*调用 TCP *echo*服务

我们之前提到过，*inetd*简化了服务器的编程，特别是并发（通常是 TCP）服务器。它通过代表被调用的服务器执行以下步骤来实现这一点：

1.  执行所有与套接字相关的初始化，调用*socket()*、*bind()*和（对于 TCP 服务器）*listen()*。

1.  对于 TCP 服务，执行*accept()*以建立新的连接。

1.  创建一个新进程来处理传入的 UDP 数据报或 TCP 连接。该进程会自动设置为守护进程。*inetd*程序通过*fork()*执行所有进程创建的细节，并通过`SIGCHLD`信号处理程序回收死去的子进程。

1.  复制 UDP 套接字或连接的 TCP 套接字的文件描述符到文件描述符 0、1 和 2，并关闭所有其他文件描述符（因为在被 execed 的服务器中它们不会被使用）。

1.  执行服务器程序。

（在上述步骤的描述中，我们假设`/etc/inetd.conf`中服务条目的*flags*字段通常为 TCP 服务指定`nowait`，为 UDP 服务指定`wait`。）

作为*inetd*如何简化 TCP 服务编程的示例，在示例 60-6 中，我们展示了通过*inetd*调用的等效 TCP*回显*服务器，源自示例 60-4。由于*inetd*执行了上述所有步骤，服务器剩下的部分仅是由子进程执行的代码，用于处理客户端请求，该代码可以通过文件描述符 0（`STDIN_FILENO`）读取。

如果服务器位于目录`/bin`（例如），那么我们需要在`/etc/inetd.conf`中创建以下条目，以便让*inetd*调用该服务器：

```
echo stream tcp nowait root /bin/is_echo_inetd_sv is_echo_inetd_sv
```

示例 60-6. 通过*inetd*调用的 TCP*回显*服务器

```
`sockets/is_echo_inetd_sv.c`
#include <syslog.h>
#include "tlpi_hdr.h"

#define BUF_SIZE 4096

int
main(int argc, char *argv[])
{
    char buf[BUF_SIZE];
    ssize_t numRead;

    while ((numRead = read(STDIN_FILENO, buf, BUF_SIZE)) > 0) {
        if (write(STDOUT_FILENO, buf, numRead) != numRead) {
            syslog(LOG_ERR, "write() failed: %s", strerror(errno));
            exit(EXIT_FAILURE);
        }
    }

    if (numRead == -1) {
        syslog(LOG_ERR, "Error from read(): %s", strerror(errno));
        exit(EXIT_FAILURE);
    }

    exit(EXIT_SUCCESS);
}
     `sockets/is_echo_inetd_sv.c`
```

## 总结

迭代服务器一次处理一个客户端，完全处理该客户端的请求后，才会继续处理下一个客户端。并发服务器则能同时处理多个客户端。在高负载场景下，传统的并发服务器设计每个客户端创建一个新的子进程（或线程），可能表现得不够好，我们概述了多种其他方法来并发处理大量客户端。

互联网超级服务器守护进程*inetd*监控多个套接字，并在接收到 UDP 数据报或 TCP 连接时启动相应的服务器。使用*inetd*可以通过最小化系统中的网络服务器进程数量来减少系统负载，并且简化了服务器进程的编程，因为它执行了服务器所需的大部分初始化步骤。

#### 进一步信息

请参考进一步信息中列出的更多信息来源。

## 练习

1.  向示例 60-4（`is_echo_sv.c`）中的程序添加代码，以限制同时执行的子进程数量。

1.  有时候，可能需要编写一个套接字服务器，以便它可以直接从命令行调用，也可以通过*inetd*间接调用。在这种情况下，使用一个命令行选项来区分这两种情况。修改示例 60-4 中的程序，使得如果给定了*-i*命令行选项，它假设是通过*inetd*调用，并在连接的套接字上处理单个客户端，该套接字由*inetd*通过`STDIN_FILENO`提供。如果没有提供*-i*选项，则程序可以假设它是从命令行调用的，并以通常的方式运行。（这个修改只需要添加几行代码。）修改`/etc/inetd.conf`，以便为*echo*服务调用这个程序。
