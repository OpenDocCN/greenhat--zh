## 第五十七章：套接字：UNIX 域

本章讲解了 UNIX 域套接字的使用，这些套接字允许同一主机系统上的进程之间进行通信。我们讨论了在 UNIX 域中使用流套接字和数据报套接字的方式。还介绍了使用文件权限控制对 UNIX 域套接字的访问，使用 *socketpair()* 创建一对连接的 UNIX 域套接字，以及 Linux 的抽象套接字命名空间。

## UNIX 域套接字地址：*struct sockaddr_un*

在 UNIX 域中，套接字地址的形式为路径名，特定于该域的套接字地址结构定义如下：

```
struct sockaddr_un {
    sa_family_t sun_family;         /* Always AF_UNIX */
    char sun_path[108];             /* Null-terminated socket pathname */
};
```

### 注意

*sockaddr_un* 结构体中的前缀 *sun_* 与 Sun Microsystems 无关，而是源自 *socket unix*。

SUSv3 没有指定 *sun_path* 字段的大小。早期的 BSD 实现使用 108 字节和 104 字节，而一个现代实现（HP-UX 11）使用 92 字节。可移植的应用程序应使用这个较小的值，并使用 *snprintf()* 或 *strncpy()* 来避免在写入该字段时出现缓冲区溢出。

为了将 UNIX 域套接字绑定到一个地址，我们初始化一个 *sockaddr_un* 结构体，然后将该结构体的（强制转换后的）指针作为 *bind()* 的 *addr* 参数，并将 *addrlen* 指定为该结构体的大小，如 示例 57-1 所示。

示例 57-1：绑定 UNIX 域套接字

```
const char *SOCKNAME = "/tmp/mysock";
    int sfd;
    struct sockaddr_un addr;

    sfd = socket(AF_UNIX, SOCK_STREAM, 0);            /* Create socket */
    if (sfd == -1)
        errExit("socket");

    memset(&addr, 0, sizeof(struct sockaddr_un));     /* Clear structure */
    addr.sun_family = AF_UNIX;                            /* UNIX domain address */
    strncpy(addr.sun_path, SOCKNAME, sizeof(addr.sun_path) - 1);

    if (bind(sfd, (struct sockaddr *) &addr, sizeof(struct sockaddr_un)) == -1)
        errExit("bind");
```

在 示例 57-1 中使用 *memset()* 函数确保所有结构体字段的值都为 0。（后续的 *strncpy()* 调用利用这一点，指定其最后一个参数为 *sun_path* 字段大小减一，以确保该字段始终有一个终止的空字节。）使用 *memset()* 清零整个结构体，而不是单独初始化各个字段，可以确保某些实现提供的非标准字段也被初始化为 0。

### 注意

衍生自 BSD 的 *bzero()* 函数是 *memset()* 的替代方法，用于将结构体内容清零。SUSv3 规定了 *bzero()* 和相关的 *bcopy()*（类似于 *memmove()*），但将这两个函数标记为遗留功能，指出应该优先使用 *memset()* 和 *memmove()*。SUSv4 删除了对 *bzero()* 和 *bcopy()* 的规定。

当用来绑定 UNIX 域套接字时，*bind()* 会在文件系统中创建一个条目。（因此，作为套接字路径名一部分的目录需要是可访问且可写的。）文件的所有权根据创建文件的常规规则来确定（新文件的所有权）。该文件被标记为一个套接字。当对该路径名应用 *stat()* 时，它会在 *stat* 结构的 *st_mode* 字段中的文件类型组件返回值 `S_IFSOCK`（获取文件信息：*stat()*")）。用 *ls -l* 列出时，UNIX 域套接字在第一列显示为类型 *s*，而 *ls -F* 会在套接字路径名后附加一个等号（=）。

### 注意

尽管 UNIX 域套接字是通过路径名进行标识的，但对这些套接字的 I/O 操作并不涉及底层设备的操作。

绑定 UNIX 域套接字时有几个要点需要注意：

+   我们不能将套接字绑定到已存在的路径名（*bind()* 会因错误 `EADDRINUSE` 失败）。

+   通常会将套接字绑定到一个绝对路径名，这样套接字就会在文件系统中驻留在一个固定地址。虽然使用相对路径名也是可能的，但不常见，因为它要求想要 *connect()* 到该套接字的应用程序知道执行 *bind()* 的应用程序的当前工作目录。

+   一个套接字只能绑定到一个路径名；反之，一个路径名也只能绑定到一个套接字。

+   我们不能使用 *open()* 打开一个套接字。

+   当套接字不再需要时，可以（且通常应该）使用 *unlink()*（或 *remove()*）删除其路径名条目。

在我们的大多数示例程序中，我们将 UNIX 域套接字绑定到 `/tmp` 目录中的路径名，因为该目录通常在每个系统上都存在且是可写的。这使得读者可以轻松运行这些程序，而无需先编辑套接字路径名。然而，请注意，这通常不是一个好的设计技巧。正如在执行文件操作和文件 I/O 时的陷阱中指出的那样，在像 `/tmp` 这样的公共可写目录中创建文件可能会导致各种安全漏洞。例如，通过在 `/tmp` 中创建一个与应用程序套接字使用的名称相同的路径名，我们可以创建一个简单的拒绝服务攻击。实际应用程序应将 *bind()* UNIX 域套接字绑定到适当安全的目录中的绝对路径名。

## UNIX 域中的流套接字

我们现在展示一个简单的客户端-服务器应用程序，它使用 UNIX 域中的流套接字。客户端程序（示例 57-4）连接到服务器，并利用该连接将数据从其标准输入传输到服务器。服务器程序（示例 57-3）接受客户端连接，并将客户端通过连接发送的所有数据传输到标准输出。该服务器是一个简单的*迭代式*服务器——它一次处理一个客户端，然后再处理下一个客户端。（我们在第六十章中会更详细地讨论服务器设计。）

示例 57-2 是这两个程序都使用的头文件。

示例 57-2. `us_xfr_sv.c` 和 `us_xfr_cl.c` 的头文件

```
`sockets/us_xfr.h`
#include <sys/un.h>
#include <sys/socket.h>
#include "tlpi_hdr.h"

#define SV_SOCK_PATH "/tmp/us_xfr"

#define BUF_SIZE 100
      `sockets/us_xfr.h`
```

在接下来的页面中，我们首先展示服务器和客户端的源代码，然后讨论这些程序的细节，并展示它们的使用示例。

示例 57-3. 一个简单的 UNIX 域流套接字服务器

```
`sockets/us_xfr_sv.c`
#include "us_xfr.h"

#define BACKLOG 5

int
main(int argc, char *argv[])
{
    struct sockaddr_un addr;
    int sfd, cfd;
    ssize_t numRead;
    char buf[BUF_SIZE];

    sfd = socket(AF_UNIX, SOCK_STREAM, 0);
    if (sfd == -1)
        errExit("socket");

    /* Construct server socket address, bind socket to it,
       and make this a listening socket */

    if (remove(SV_SOCK_PATH) == -1 && errno != ENOENT)
        errExit("remove-%s", SV_SOCK_PATH);

    memset(&addr, 0, sizeof(struct sockaddr_un));
    addr.sun_family = AF_UNIX;
    strncpy(addr.sun_path, SV_SOCK_PATH, sizeof(addr.sun_path) - 1);

    if (bind(sfd, (struct sockaddr *) &addr, sizeof(struct sockaddr_un)) == -1)
        errExit("bind");

    if (listen(sfd, BACKLOG) == -1)
        errExit("listen");

    for (;;) {          /* Handle client connections iteratively */

        /* Accept a connection. The connection is returned on a new
           socket, 'cfd'; the listening socket ('sfd') remains open
           and can be used to accept further connections. */

        cfd = accept(sfd, NULL, NULL);
        if (cfd == -1)
            errExit("accept");

        /* Transfer data from connected socket to stdout until EOF */

        while ((numRead = read(cfd, buf, BUF_SIZE)) > 0)
            if (write(STDOUT_FILENO, buf, numRead) != numRead)
                fatal("partial/failed write");

        if (numRead == -1)
            errExit("read");
        if (close(cfd) == -1)
            errMsg("close");
    }
}
     `sockets/us_xfr_sv.c`
```

示例 57-4. 一个简单的 UNIX 域流套接字客户端

```
`sockets/us_xfr_cl.c`
#include "us_xfr.h"

int
main(int argc, char *argv[])
{
    struct sockaddr_un addr;
    int sfd;
    ssize_t numRead;
    char buf[BUF_SIZE];

    sfd = socket(AF_UNIX, SOCK_STREAM, 0);       /* Create client socket */
    if (sfd == -1)
        errExit("socket");

    /* Construct server address, and make the connection */

    memset(&addr, 0, sizeof(struct sockaddr_un));
    addr.sun_family = AF_UNIX;
    strncpy(addr.sun_path, SV_SOCK_PATH, sizeof(addr.sun_path) - 1);

    if (connect(sfd, (struct sockaddr *) &addr,
                sizeof(struct sockaddr_un)) == -1)
        errExit("connect");

    /* Copy stdin to socket */

    while ((numRead = read(STDIN_FILENO, buf, BUF_SIZE)) > 0)
        if (write(sfd, buf, numRead) != numRead)
            fatal("partial/failed write");

    if (numRead == -1)
        errExit("read");

    exit(EXIT_SUCCESS);         /* Closes our socket; server sees EOF */
}
     `sockets/us_xfr_cl.c`
```

服务器程序如示例 57-3 所示。服务器执行以下步骤：

+   创建一个套接字。

+   删除任何与我们希望绑定套接字的路径名相同的现有文件。

+   为服务器的套接字构建一个地址结构，将套接字绑定到该地址，并将其标记为监听套接字。

+   执行一个无限循环来处理传入的客户端请求。每次循环迭代执行以下步骤：

    +   接受一个连接，获取一个新的套接字，*cfd*，用于该连接。

    +   从已连接的套接字读取所有数据并将其写入标准输出。

    +   关闭已连接的套接字*cfd*。

服务器必须手动终止（例如，通过向其发送信号）。

客户端程序（示例 57-4）执行以下步骤：

+   创建一个套接字。

+   为服务器的套接字构建地址结构，并连接到该地址的套接字。

+   执行一个循环，将标准输入复制到套接字连接中。遇到标准输入的文件结束符时，客户端终止，结果是它的套接字被关闭，服务器在从连接另一端的套接字读取时看到文件结束符。

以下是一个展示如何使用这些程序的 shell 会话日志。我们首先在后台运行服务器：

```
$ `./us_xfr_sv > b &`
[1] 9866
$ `ls -lF /tmp/us_xfr`                        *Examine socket file with ls*
srwxr-xr-x    1 mtk      users         0 Jul 18 10:48 /tmp/us_xfr=
```

然后我们创建一个测试文件，用作客户端的输入，并运行客户端：

```
$ `cat *.c > a`
$ `./us_xfr_cl < a`                           *Client takes input from test file*
```

到此为止，子进程已经完成。现在我们也终止服务器，并检查服务器的输出是否与客户端的输入匹配：

```
$ `kill %1`                                   *Terminate server*
 [1]+  Terminated   ./us_xfr_sv >b          *Shell sees server’s termination*
$ `diff a b`
$
```

*diff* 命令不会产生任何输出，表示输入文件和输出文件是相同的。

请注意，在服务器终止后，套接字路径名仍然存在。这就是为什么服务器在调用 *bind()* 之前使用 *remove()* 删除任何已存在的套接字路径名的原因。（假设我们有适当的权限，此 *remove()* 调用将删除任何类型的文件，即使它不是套接字。）如果我们不这么做，*bind()* 调用将失败，如果之前的服务器调用已经创建了这个套接字路径名。

## UNIX 域中的数据报套接字

在我们提供的 数据报套接字的通用描述中，我们说明了使用数据报套接字的通信是不可靠的。这适用于通过网络传输的数据报。然而，对于 UNIX 域套接字，数据报传输是在内核中进行的，因此是可靠的。所有消息都会按顺序且不重复地交付。

#### UNIX 域数据报套接字的最大数据报大小

SUSv3 并没有规定通过 UNIX 域套接字发送的数据报的最大大小。在 Linux 上，我们可以发送相当大的数据报。限制由 `SO_SNDBUF` 套接字选项和各种 `/proc` 文件控制，如 *socket(7)* 手册页中所述。然而，一些其他的 UNIX 实现会施加较低的限制，例如 2048 字节。使用 UNIX 域数据报套接字的便携式应用程序应考虑对使用的数据报大小施加一个较低的上限。

#### 示例程序

示例 57-6 和 示例 57-7 展示了一个使用 UNIX 域数据报套接字的简单客户端-服务器应用程序。这两个程序都使用了 示例 57-5 中显示的头文件。

示例 57-5. `ud_ucase_sv.c` 和 `ud_ucase_cl.c` 使用的头文件

```
`sockets/ud_ucase.h`
#include <sys/un.h>
#include <sys/socket.h>
#include <ctype.h>
#include "tlpi_hdr.h"

#define BUF_SIZE 10             /* Maximum size of messages exchanged
                                   between client to server */

#define SV_SOCK_PATH "/tmp/ud_ucase"
     `sockets/ud_ucase.h`
```

服务器程序（示例 57-6）首先创建一个套接字并将其绑定到一个已知地址。（在此之前，服务器会先删除与该地址匹配的路径名，以防路径名已存在。）然后服务器进入一个无限循环，使用 *recvfrom()* 从客户端接收数据报，将接收到的文本转换为大写，并通过 *recvfrom()* 获得的地址将转换后的文本返回给客户端。

客户端程序（示例 57-7）创建一个套接字并将其绑定到一个地址，以便服务器可以发送回复。通过在路径名中包含客户端的进程 ID，使客户端地址具有唯一性。然后客户端进入循环，将每个命令行参数作为单独的消息发送给服务器。在发送每个消息后，客户端读取服务器的响应并在标准输出上显示。

示例 57-6。一个简单的 UNIX 域数据报服务器

```
`sockets/ud_ucase_sv.c`
#include "ud_ucase.h"

int
main(int argc, char *argv[])
{
    struct sockaddr_un svaddr, claddr;
    int sfd, j;
    ssize_t numBytes;
    socklen_t len;
    char buf[BUF_SIZE];

    sfd = socket(AF_UNIX, SOCK_DGRAM, 0);       /* Create server socket */
    if (sfd == -1)
        errExit("socket");

    /* Construct well-known address and bind server socket to it */

    if (remove(SV_SOCK_PATH) == -1 && errno != ENOENT)
        errExit("remove-%s", SV_SOCK_PATH);

    memset(&svaddr, 0, sizeof(struct sockaddr_un));
    svaddr.sun_family = AF_UNIX;
    strncpy(svaddr.sun_path, SV_SOCK_PATH, sizeof(svaddr.sun_path) - 1);

    if (bind(sfd, (struct sockaddr *) &svaddr, sizeof(struct sockaddr_un)) == -1)
        errExit("bind");

    /* Receive messages, convert to uppercase, and return to client */

    for (;;) {
        len = sizeof(struct sockaddr_un);
        numBytes = recvfrom(sfd, buf, BUF_SIZE, 0,
                            (struct sockaddr *) &claddr, &len);
        if (numBytes == -1)
            errExit("recvfrom");

        printf("Server received %ld bytes from %s\n", (long) numBytes,
                claddr.sun_path);

        for (j = 0; j < numBytes; j++)
            buf[j] = toupper((unsigned char) buf[j]);

        if (sendto(sfd, buf, numBytes, 0, (struct sockaddr *) &claddr, len) !=
                numBytes)
            fatal("sendto");
    }
}
      `sockets/ud_ucase_sv.c`
```

示例 57-7。一个简单的 UNIX 域数据报客户端

```
`sockets/ud_ucase_cl.c`
#include "ud_ucase.h"

int
main(int argc, char *argv[])
{
    struct sockaddr_un svaddr, claddr;
    int sfd, j;
    size_t msgLen;
    ssize_t numBytes;
    char resp[BUF_SIZE];

    if (argc < 2 || strcmp(argv[1], "--help") == 0)
        usageErr("%s msg...\n", argv[0]);

    /* Create client socket; bind to unique pathname (based on PID) */

    sfd = socket(AF_UNIX, SOCK_DGRAM, 0);
    if (sfd == -1)
        errExit("socket");

    memset(&claddr, 0, sizeof(struct sockaddr_un));
    claddr.sun_family = AF_UNIX;
    snprintf(claddr.sun_path, sizeof(claddr.sun_path),
            "/tmp/ud_ucase_cl.%ld", (long) getpid());

    if (bind(sfd, (struct sockaddr *) &claddr, sizeof(struct sockaddr_un)) == -1)
        errExit("bind");

    /* Construct address of server */

    memset(&svaddr, 0, sizeof(struct sockaddr_un));
    svaddr.sun_family = AF_UNIX;
    strncpy(svaddr.sun_path, SV_SOCK_PATH, sizeof(svaddr.sun_path) - 1);

    /* Send messages to server; echo responses on stdout */

    for (j = 1; j < argc; j++) {
        msgLen = strlen(argv[j]);       /* May be longer than BUF_SIZE */
        if (sendto(sfd, argv[j], msgLen, 0, (struct sockaddr *) &svaddr,
                sizeof(struct sockaddr_un)) != msgLen)
            fatal("sendto");

        numBytes = recvfrom(sfd, resp, BUF_SIZE, 0, NULL, NULL);
        if (numBytes == -1)
            errExit("recvfrom");
        printf("Response %d: %.*s\n", j, (int) numBytes, resp);
    }

    remove(claddr.sun_path);            /* Remove client socket pathname */
    exit(EXIT_SUCCESS);
}
      `sockets/ud_ucase_cl.c`
```

以下 shell 会话日志演示了服务器和客户端程序的使用：

```
$ `./ud_ucase_sv &`
[1] 20113
$ `./ud_ucase_cl hello world`                 *Send 2 messages to server*
Server received 5 bytes from /tmp/ud_ucase_cl.20150
Response 1: HELLO
Server received 5 bytes from /tmp/ud_ucase_cl.20150
Response 2: WORLD
$ `./ud_ucase_cl 'long message'`              *Send 1 longer message to server*
Server received 10 bytes from /tmp/ud_ucase_cl.20151
Response 1: LONG MESSA
$ `kill %1`                                   *Terminate server*
```

客户端程序的第二次调用旨在演示，当*recvfrom()*调用指定一个比消息大小短的*length*（`BUF_SIZE`，在示例 57-5 中定义，值为 10）时，消息会被静默截断。我们可以看到发生了这种截断，因为服务器打印了一个消息，表示它接收到了 10 个字节，而客户端发送的消息包含 12 个字节。

## UNIX 域套接字权限

套接字文件的所有权和权限决定了哪些进程可以与该套接字进行通信：

+   要连接到 UNIX 域流套接字，必须对套接字文件具有写权限。

+   要向 UNIX 域数据报套接字发送数据报，必须对套接字文件具有写权限。

此外，需要在套接字路径名中的每个目录上具有执行（搜索）权限。

默认情况下，套接字由*bind()*创建，所有权限都授予所有者（用户）、组和其他人。为了更改此设置，我们可以在调用*bind()*之前使用*umask()*来禁用我们不希望授予的权限。

有些系统忽略了套接字文件的权限（SUSv3 允许这种情况）。因此，我们无法便捷地使用套接字文件的权限来控制对套接字的访问，尽管我们可以便捷地使用托管目录的权限来实现这一目的。

## 创建一个连接的套接字对：*socketpair()*

有时，单个进程创建一对套接字并将它们连接在一起是有用的。这可以通过两次调用*socket()*，一次调用*bind()*，然后调用*listen()*、*connect()*和*accept()*（对于流套接字），或者调用*connect()*（对于数据报套接字）来完成。*socketpair()*系统调用提供了这种操作的简写。

```
#include <sys/socket.h>

int `socketpair`(int *domain*, int *type*, int *protocol*, int *sockfd*[2]);
```

### 注意

成功时返回 0，出错时返回-1

该*socketpair()*系统调用只能在 UNIX 域中使用；即，*域*必须指定为`AF_UNIX`。（此限制适用于大多数实现，并且是合乎逻辑的，因为套接字对是在单个主机系统上创建的。）套接字*类型*可以指定为`SOCK_DGRAM`或`SOCK_STREAM`。*协议*参数必须指定为 0。*sockfd*数组返回指向两个连接套接字的文件描述符。

指定*类型*为`SOCK_STREAM`会创建一个双向管道的等效物（也称为*流管道*）。每个套接字既可以用于读取，也可以用于写入，并且在两个套接字之间，每个方向都有独立的数据通道流动。（在 BSD 派生的实现中，*pipe()*被实现为对*socketpair()*的调用。）

通常，套接字对的使用方式类似于管道。调用*socketpair()*之后，进程通过*fork()*创建一个子进程。子进程继承了父进程的文件描述符副本，包括指向套接字对的描述符。因此，父子进程可以使用套接字对进行进程间通信（IPC）。

使用*socketpair()*的一个方式与手动创建一对连接套接字的区别在于，这些套接字不绑定任何地址。这有助于避免一类安全漏洞，因为这些套接字对其他进程不可见。

### 注意

从内核 2.6.27 开始，Linux 为*类型*参数提供了第二种用途，允许两个非标准标志与套接字类型进行“或”操作。`SOCK_CLOEXEC`标志会导致内核为两个新的文件描述符启用 close-on-exec 标志（`FD_CLOEXEC`）。该标志的用途与*open()*的`O_CLOEXEC`返回的文件描述符号")标志相同。`SOCK_NONBLOCK`标志会导致内核为两个底层打开的文件描述符设置`O_NONBLOCK`标志，从而使未来对该套接字的 I/O 操作变为非阻塞。这省去了额外调用*fcntl()*以达到相同结果的需求。

## Linux 抽象套接字命名空间

所谓的*抽象命名空间*是一个特定于 Linux 的功能，它允许我们将 UNIX 域套接字绑定到一个名称，而无需该名称在文件系统中创建。这提供了一些潜在的优势：

+   我们不需要担心与文件系统中现有名称可能发生的冲突。

+   我们不需要在使用完套接字后取消链接套接字路径名。抽象名称会在套接字关闭时自动删除。

+   我们不需要为套接字创建文件系统路径名。这在*chroot*环境中可能非常有用，或者当我们没有对文件系统的写入权限时。

要创建抽象绑定，我们将 *sun_path* 字段的第一个字节指定为空字节（`\0`）。这区分了抽象套接字名称和传统 UNIX 域套接字路径名，后者由一个或多个非空字节组成，且以空字节结尾。抽象套接字的名称由 *sun_path* 中剩余的字节（包括任何空字节）定义，直到地址结构的大小所定义的长度（即 *addrlen - sizeof(sa_family_t)*）为止。

示例 57-8 展示了抽象套接字绑定的创建。

示例 57-8. 创建抽象套接字绑定

```
*from* `sockets/us_abstract_bind.c`
    struct sockaddr_un addr;

    memset(&addr, 0, sizeof(struct sockaddr_un));  /* Clear address structure */
    addr.sun_family = AF_UNIX;                     /* UNIX domain address */

    /* addr.sun_path[0] has already been set to 0 by memset() */

    str = "xyz";         /* Abstract name is "\0xyz" */
    strncpy(&addr.sun_path[1], str, strlen (str));

    sockfd = socket(AF_UNIX, SOCK_STREAM, 0);
    if (sockfd == -1)
        errExit("socket");

    if (bind(sockfd, (struct sockaddr *) &addr,
            sizeof(sa_family_t) + strlen(str) + 1) == -1)
        errExit("bind");
                                                  *from* `sockets/us_abstract_bind.c`
```

使用初始空字节来区分抽象套接字名称和传统套接字名称可能会带来一种不同寻常的后果。假设变量*name*恰好指向一个零长度的字符串，并且我们尝试将一个 UNIX 域套接字绑定到如下初始化的*sun_path*：

```
strncpy(addr.sun_path, name, sizeof(addr.sun_path) - 1);
```

在 Linux 上，我们会不经意地创建一个抽象套接字绑定。然而，这样的代码序列可能是无意的（即，一个 bug）。在其他 UNIX 实现中，后续的 *bind()* 调用将失败。

## 总结

UNIX 域套接字允许同一主机上的应用程序之间进行通信。UNIX 域支持流式套接字和数据报套接字。

UNIX 域套接字通过文件系统中的路径名进行标识。可以使用文件权限来控制对 UNIX 域套接字的访问。

*socketpair()* 系统调用创建一对连接的 UNIX 域套接字。这避免了多个系统调用来创建、绑定和连接套接字。套接字对通常像管道一样使用：一个进程创建套接字对，然后通过分叉（fork）创建一个子进程，子进程继承指向套接字的描述符。然后，两个进程可以通过套接字对进行通信。

Linux 特有的抽象套接字命名空间允许我们将 UNIX 域套接字绑定到一个在文件系统中不存在的名称。

#### 进一步的信息

请参考进一步的信息中列出的资料来源。

## 练习

1.  在 UNIX 域中的数据报套接字中，我们提到 UNIX 域数据报套接字是可靠的。编写程序证明，如果发送方比接收方读取数据报的速度更快，那么发送方最终会被阻塞，并且会一直被阻塞，直到接收方读取一些待处理的数据报。

1.  将示例 57-3（`us_xfr_sv.c`）和示例 57-4（`us_xfr_cl.c`）中的程序重写为使用 Linux 特有的抽象套接字命名空间(The Linux Abstract Socket Namespace)。

1.  使用 UNIX 域流套接字重新实现 A Client-Server Application Using FIFOs 中的序列号服务器和客户端。

1.  假设我们创建了两个绑定到路径`/somepath/a`和`/somepath/b`的 UNIX 域数据报套接字，并将套接字`/somepath/a`连接到`/somepath/b`。如果我们创建第三个数据报套接字并尝试通过该套接字向`/somepath/a`发送(*sendto()*)一个数据报，会发生什么？编写一个程序来确定答案。如果你可以访问其他 UNIX 系统，在这些系统上测试程序，看看答案是否不同。
