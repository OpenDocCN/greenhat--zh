## 第五十九章. 套接字：互联网域

在前几章中，我们已了解了通用套接字概念和 TCP/IP 协议套件，现在在本章中我们将开始探讨如何在 IPv4（`AF_INET`）和 IPv6（`AF_INET6`）域中进行套接字编程。

如在第五十八章中所述，互联网域套接字地址由 IP 地址和端口号组成。尽管计算机使用 IP 地址和端口号的二进制表示方式，但人类更擅长处理名称而非数字。因此，我们描述了使用名称识别主机计算机和端口的技术。我们还探讨了使用库函数获取特定主机名的 IP 地址和与特定服务名称对应的端口号的方法。关于主机名的讨论包括了域名系统（DNS）的描述，它实现了一个分布式数据库，用于将主机名映射到 IP 地址，反之亦然。

## 互联网域套接字

互联网域流套接字是基于 TCP 实现的。它们提供一个可靠的双向字节流通信通道。

互联网域数据报套接字是基于 UDP 实现的。UDP 套接字与其 UNIX 域对等物类似，但请注意以下区别：

+   UNIX 域数据报套接字是可靠的，但 UDP 套接字则不是——数据报可能会丢失、重复，或到达的顺序与发送顺序不同。

+   在 UNIX 域数据报套接字上发送数据时，如果接收套接字的数据队列已满，则会阻塞。相比之下，使用 UDP 时，如果传入的数据报会溢出接收方的队列，那么数据报会被悄无声息地丢弃。

## 网络字节顺序

IP 地址和端口号是整数值。当我们将这些值通过网络传输时，遇到的一个问题是，不同的硬件架构以不同的顺序存储多字节整数的字节。如图 59-1 所示，存储整数时先存储最重要字节（即存储在最低内存地址处）的架构被称为*大端序*；而先存储最不重要字节的架构则被称为*小端序*。（这些术语源自乔纳森·斯威夫特在 1726 年出版的讽刺小说《格列佛游记》，其中这些术语指的是在开煮蛋时从不同端打开蛋的对立政治派别。）最典型的小端序架构是 x86。（数字的 VAX 架构是另一个历史上重要的例子，因为 BSD 广泛用于该机器。）大多数其他架构都是大端序。一些硬件架构可以在两种格式之间切换。特定机器上使用的字节顺序被称为*主机字节顺序*。

![2 字节和 4 字节整数的大端和小端字节序](img/59-1_SOCKINET-byte-order.png.jpg)图 59-1：2 字节和 4 字节整数的大端和小端字节序

由于端口号和 IP 地址必须在网络上的所有主机之间传输并被理解，因此必须使用标准的字节顺序。这个字节顺序称为*网络字节序*，它正好是大端字节序。

在本章后面，我们将介绍各种函数，它们将主机名（例如，[www.kernel.org](http://www.kernel.org)）和服务名（例如，*http*）转换为相应的数字形式。这些函数通常返回网络字节序中的整数，这些整数可以直接复制到套接字地址结构的相关字段中。

然而，我们有时直接使用整数常量来表示 IP 地址和端口号。例如，我们可能会选择将端口号硬编码到程序中，作为程序的命令行参数指定端口号，或者在指定 IPv4 地址时使用`INADDR_ANY`和`INADDR_LOOPBACK`等常量。这些值在 C 语言中按照主机机器的约定表示，因此它们是主机字节序。我们必须在将这些值存储到套接字地址结构之前，将它们转换为网络字节序。

*htons()，htonl()，ntohs()，ntohl()* 函数被定义（通常作为宏），用于在主机字节序和网络字节序之间进行整数的双向转换。

```
#include <arpa/inet.h>

uint16_t `htons`(uint16_t *host_uint16*);
```

### 注意

返回*host_uint16*转换为网络字节序

```
uint32_t `htonl`(uint32_t *host_uint32*);
```

### 注意

返回*host_uint32*转换为网络字节序

```
uint16_t `ntohs`(uint16_t *net_uint16*);
```

### 注意

返回*net_uint16*转换为主机字节序

```
uint32_t `ntohl`(uint32_t *net_uint32*);
```

### 注意

返回*net_uint32*转换为主机字节序

在早期，这些函数的原型如下：

```
unsigned long htonl(unsigned long hostlong);
```

这揭示了函数名称的来源——在这种情况下是*host to network long*。在大多数早期实现套接字的系统中，短整型是 16 位，长整型是 32 位。但在现代系统中这一点已不再成立（至少对于长整型而言），因此上面给出的原型提供了更精确的类型定义，尽管函数名没有变化。*uint16_t* 和 *uint32_t* 数据类型分别是 16 位和 32 位无符号整数。

严格来说，这四个函数的使用仅在主机字节序与网络字节序不同的系统上是必要的。然而，应该始终使用这些函数，以便程序能够在不同的硬件架构上移植。在主机字节序与网络字节序相同的系统上，这些函数仅返回未更改的参数。

## 数据表示

在编写网络程序时，我们需要意识到不同的计算机架构使用不同的约定来表示各种数据类型。我们已经提到，整数类型可以以大端序或小端序形式存储。还有其他可能的差异。例如，C 语言中的*long*数据类型在某些系统上可能是 32 位，而在其他系统上是 64 位。当我们考虑结构体时，问题进一步复杂化，因为不同的实现会使用不同的规则将结构体的字段对齐到主机系统的地址边界，从而在字段之间留下不同数量的填充字节。

由于数据表示的差异，跨异构系统之间通过网络交换数据的应用程序必须采用某种共同的编码约定。发送方必须按照此约定对数据进行编码，而接收方则根据相同的约定进行解码。将数据转化为标准格式以便通过网络传输的过程称为*封送处理*。存在各种封送处理标准，例如 XDR（外部数据表示，描述见 RFC 1014）、ASN.1-BER（抽象语法表示法 1，[`www.asn1.org/`](http://www.asn1.org/)）、CORBA 和 XML。通常，这些标准为每种数据类型定义固定的格式（例如，定义字节顺序和使用的位数）。除了按照要求的格式进行编码外，每个数据项还会附加标识其类型（以及可能的长度）的额外字段。

然而，通常采用比封送处理更简单的方法：将所有传输的数据编码为文本形式，数据项通过指定字符（通常是换行符）分隔。此方法的一个优点是我们可以使用*telnet*来调试应用程序。为此，我们使用以下命令：

```
$ `telnet` ``*`host port`*``
```

然后我们可以输入要传输到应用程序的文本行，并查看应用程序发送的响应。我们在客户端-服务器示例（流套接字）中演示了此技术。

### 注意

跨异构系统的表示差异所带来的问题不仅适用于网络中的数据传输，还适用于这些系统之间的任何数据交换机制。例如，在不同异构系统之间传输磁盘或磁带文件时，我们也面临相同的问题。网络编程只是我们当前最常遇到这一问题的编程场景。

如果我们将通过流套接字传输的数据编码为以换行符分隔的文本，则可以方便地定义一个像*readLine()*这样的函数，示例见示例 59-1。

```
#include "read_line.h"

ssize_t `readLine`(int *fd*, void **buffer*, size_t *n*);
```

### 注意

返回复制到*缓冲区*中的字节数（不包括终止的空字节），如果是文件结尾则返回 0，出错时返回-1

*readLine()*函数从由文件描述符参数*fd*引用的文件中读取字节，直到遇到换行符为止。输入的字节序列将返回到*buffer*指向的位置，*buffer*必须指向至少*n*字节的内存区域。返回的字符串总是以 null 终止；因此，最多会返回*(n - 1)*个实际数据字节。成功时，*readLine()*返回存入*buffer*的字节数；终止空字节不包括在此计数中。

示例 59-1。逐行读取数据

```
`sockets/read_line.c`
#include <unistd.h>
#include <errno.h>
#include "read_line.h"                  /* Declaration of readLine() */

ssize_t
readLine(int fd, void *buffer, size_t n)
{
    ssize_t numRead;                    /* # of bytes fetched by last read() */
    size_t totRead;                     /* Total bytes read so far */
    char *buf;
    char ch;

    if (n <= 0 || buffer == NULL) {
        errno = EINVAL;
        return -1;
    }

    buf = buffer;                       /* No pointer arithmetic on "void *" */

    totRead = 0;
    for (;;) {
        numRead = read(fd, &ch, 1);

        if (numRead == -1) {
            if (errno == EINTR)        /* Interrupted --> restart read() */
                continue;
            else
                return -1;              /* Some other error */

        } else if (numRead == 0) {      /* EOF */
            if (totRead == 0)           /* No bytes read; return 0 */
                return 0;
            else                        /* Some bytes read; add '\0' */
                break;

        } else {                        /* 'numRead' must be 1 if we get here */
            if (totRead < n - 1) {      /* Discard > (n - 1) bytes */
                totRead++;
                *buf++ = ch;
            }

            if (ch == '\n')
                break;
        }
    }

    *buf = '\0';
    return totRead;
}
     `sockets/read_line.c`
```

如果在遇到换行符之前读取的字节数大于或等于*(n - 1)*，则*readLine()*函数会丢弃多余的字节（包括换行符）。如果在前*(n - 1)*个字节内读取到了换行符，则它会包含在返回的字符串中。（因此，我们可以通过检查返回的*buffer*中的换行符是否位于终止空字节之前，来确定是否丢弃了字节。）我们采取这种方法是为了确保依赖于按行处理输入的应用协议，不会将长行误处理为多行。这可能会破坏协议，因为两端的应用程序将会不同步。另一种方法是让*readLine()*只读取足够的字节以填充提供的缓冲区，将剩余的字节（直到下一个换行符）留给下一次调用*readLine()*。在这种情况下，*readLine()*的调用者需要处理可能读取到部分行的情况。

在客户端-服务器示例（流套接字）中，我们在示例程序中使用了*readLine()*函数。

## 互联网套接字地址

有两种类型的互联网域套接字地址：IPv4 和 IPv6。

#### IPv4 套接字地址：*struct sockaddr_in*

IPv4 套接字地址存储在*sockaddr_in*结构中，定义在`<netinet/in.h>`中，具体如下：

```
struct in_addr {                    /* IPv4 4-byte address */
    in_addr_t s_addr;               /* Unsigned 32-bit integer */
};

struct sockaddr_in {                /* IPv4 socket address */
    sa_family_t    sin_family;      /* Address family (AF_INET) */
    in_port_t      sin_port;        /* Port number */
    struct in_addr sin_addr;        /* IPv4 address */
    unsigned char  __pad[X];        /* Pad to size of 'sockaddr'
                                       structure (16 bytes) */
};
```

在通用套接字地址结构：*struct sockaddr*中，我们看到通用的*sockaddr*结构开始时有一个字段，用来标识套接字域。这对应于*sockaddr_in*结构中的*sin_family*字段，该字段始终设置为`AF_INET`。*sin_port*和*sin_addr*字段分别是端口号和 IP 地址，都是网络字节顺序。*in_port_t*和*in_addr_t*数据类型分别是无符号整数类型，长度为 16 位和 32 位。

#### IPv6 套接字地址：*struct sockaddr_in6*

与 IPv4 地址类似，IPv6 套接字地址包含 IP 地址和端口号。不同之处在于，IPv6 地址是 128 位而不是 32 位。IPv6 套接字地址存储在*sockaddr_in6*结构中，定义在`<netinet/in.h>`中，具体如下：

```
struct in6_addr {                   /* IPv6 address structure */
    uint8_t s6_addr[16];            /* 16 bytes == 128 bits */
};
struct sockaddr_in6 {               /* IPv6 socket address */
    sa_family_t sin6_family;        /* Address family (AF_INET6) */
    in_port_t   sin6_port;          /* Port number */
    uint32_t    sin6_flowinfo;      /* IPv6 flow information */
    struct in6_addr sin6_addr;      /* IPv6 address */
    uint32_t    sin6_scope_id;      /* Scope ID (new in kernel 2.4) */
};
```

*sin_family* 字段被设置为 `AF_INET6`。*sin6_port* 和 *sin6_addr* 字段分别是端口号和 IP 地址。（*uint8_t* 数据类型，用于定义 *in6_addr* 结构体的字节，是一个 8 位无符号整数。）其余字段，*sin6_flowinfo* 和 *sin6_scope_id*，超出了本书的讨论范围；就我们的目的而言，它们始终被设置为 0。*sockaddr_in6* 结构体中的所有字段都采用网络字节顺序。

### 注意

IPv6 地址在 RFC 4291 中进行了描述。有关 IPv6 流量控制（*sin6_flowinfo*）的信息可以在附录 A 中找到，此外，RFC 2460 和 3697 也包含相关信息。RFC 3493 和 4007 提供了关于 *sin6_scope_id* 的信息。

IPv6 有与 IPv4 通配符地址和回环地址等效的地址。然而，它们的使用较为复杂，因为 IPv6 地址是存储在数组中的（而不是使用标量类型）。我们使用 IPv6 通配符地址（`0::0`）来说明这一点。这个地址的常量 `IN6ADDR_ANY_INIT` 定义如下：

```
#define IN6ADDR_ANY_INIT { { 0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0 } }
```

### 注意

在 Linux 上，某些头文件的细节与本节中的描述有所不同。特别是，*in6_addr* 结构体包含一个联合体定义，将 128 位的 IPv6 地址分成 16 个字节，八个 2 字节整数，或者四个 32 字节整数。由于存在这个定义，*glibc* 对 `IN6ADDR_ANY_INIT` 常量的定义实际上包含比主文本中显示的更多的嵌套大括号。

我们可以在伴随变量声明的初始化器中使用 `IN6ADDR_ANY_INIT` 常量，但不能在赋值语句的右侧使用它，因为 C 语法不允许在赋值中使用结构常量。相反，我们必须使用一个预定义的变量 *in6addr_any*，该变量由 C 库按照如下方式初始化：

```
const struct in6_addr in6addr_any = IN6ADDR_ANY_INIT;
```

因此，我们可以使用通配符地址初始化 IPv6 套接字地址结构，如下所示：

```
struct sockaddr_in6 addr;

memset(&addr, 0, sizeof(struct sockaddr_in6));
addr.sin6_family = AF_INET6;
addr.sin6_addr = in6addr_any;
addr.sin6_port = htons(SOME_PORT_NUM);
```

对应的 IPv6 回环地址 (`::1`) 的常量和变量是 `IN6ADDR_LOOPBACK_INIT` 和 *in6addr_loopback*。

与 IPv4 地址不同，IPv6 常量和变量初始化器采用网络字节顺序。但正如上面代码所示，我们仍然需要确保端口号采用网络字节顺序。

如果 IPv4 和 IPv6 在同一主机上共存，它们共享相同的端口号空间。这意味着，如果某个应用程序将 IPv6 套接字绑定到 TCP 端口 2000（使用 IPv6 通配符地址），那么 IPv4 TCP 套接字就不能绑定到相同的端口。（TCP/IP 实现确保其他主机上的套接字能够与此套接字进行通信，无论这些主机运行的是 IPv4 还是 IPv6。）

#### *sockaddr_storage* 结构体

随着 IPv6 套接字 API 的推出，引入了新的通用 *sockaddr_storage* 结构。该结构被定义为足够大，以容纳任何类型的套接字地址（即，任何类型的套接字地址结构都可以被强制转换并存储在其中）。特别地，该结构使我们能够透明地存储 IPv4 或 IPv6 套接字地址，从而消除了代码中的 IP 版本依赖性。*sockaddr_storage* 结构在 Linux 中的定义如下：

```
#define __ss_aligntype uint32_t         /* On 32-bit architectures */
struct sockaddr_storage {
    sa_family_t ss_family;
    __ss_aligntype __ss_align;          /* Force alignment */
    char __ss_padding[SS_PADSIZE];      /* Pad to 128 bytes */
};
```

## 主机和服务转换函数概述

计算机将 IP 地址和端口号表示为二进制。然而，人类更容易记住名字而不是数字。使用符号名称还提供了一种有用的间接性；即使底层的数字值发生变化，用户和程序仍然可以继续使用相同的名称。

*主机名* 是连接到网络的系统的符号标识符（可能有多个 IP 地址）。*服务名称* 是端口号的符号表示。

以下是表示主机地址和端口的方法：

+   主机地址可以表示为二进制值、符号主机名或表示格式（IPv4 的点分十进制或 IPv6 的十六进制字符串）。

+   端口可以表示为二进制值或符号服务名称。

提供了各种库函数，用于在这些格式之间进行转换。本节简要总结了这些函数。接下来的章节将详细描述现代 API（*inet_ntop()*、*inet_pton()*、*getaddrinfo()*、*getnameinfo()* 等）。在过时的主机和服务转换 API 中，我们简要讨论了过时的 API（*inet_aton()*、*inet_ntoa()*、*gethostbyname()*、*getservbyname()* 等）。

#### 在二进制和人类可读格式之间转换 IPv4 地址

*inet_aton()* 和 *inet_ntoa()* 函数将点分十进制表示法中的 IPv4 地址转换为二进制格式，并反之亦然。我们主要描述这些函数，因为它们出现在历史代码中。如今，它们已经过时。现代程序如果需要进行此类转换，应使用我们接下来描述的函数。

#### 在二进制和人类可读格式之间转换 IPv4 和 IPv6 地址

*inet_pton()* 和 *inet_ntop()* 函数类似于 *inet_aton()* 和 *inet_ntoa()*，但不同之处在于它们还处理 IPv6 地址。它们将二进制的 IPv4 和 IPv6 地址转换为并从 *表示* 格式转换——即点分十进制或十六进制字符串表示法。

由于人类处理名字比处理数字更为轻松，我们通常在程序中仅偶尔使用这些函数。*inet_ntop()* 的一个用途是生成可打印的 IP 地址表示形式，以供日志记录使用。有时，使用此函数而不是将 IP 地址转换（“解析”）为主机名是更可取的，原因如下：

+   解析 IP 地址到主机名可能涉及到向 DNS 服务器发送一个可能耗时的请求。

+   在某些情况下，可能没有 DNS（PTR）记录将 IP 地址映射到相应的主机名。

我们在 《*inet_pton()* 和 *inet_ntop()* 函数 and inet_ntop() Functions")](ch59.html#the_inet_underscore_pton_open_parenthesi) 中描述了这些函数，*getaddrinfo()* 和 *getnameinfo()* 在它们之前，它们执行二进制表示和相应符号名称之间的转换，主要是因为它们提供了一个更简单的 API。这使我们能够快速展示一些使用互联网域套接字的工作示例。

#### 将主机和服务名称转换为二进制形式（已废弃）

*gethostbyname()* 函数返回与主机名对应的二进制 IP 地址，而 *getservbyname()* 函数返回与服务名称对应的端口号。反向转换由 *gethostbyaddr()* 和 *getservbyport()* 完成。我们描述这些函数是因为它们在现有代码中广泛使用。然而，它们现在已被废弃。（SUSv3 标记这些函数为废弃，SUSv4 删除了它们的规格。）新代码应使用 *getaddrinfo()* 和 *getnameinfo()* 函数（接下来将描述）来进行这些转换。

#### 将主机和服务名称转换为二进制形式（现代）

*getaddrinfo()* 函数是 *gethostbyname()* 和 *getservbyname()* 的现代替代者。给定一个主机名和一个服务名，*getaddrinfo()* 返回一个包含相应二进制 IP 地址和端口号的结构集合。与 *gethostbyname()* 不同，*getaddrinfo()* 透明地处理 IPv4 和 IPv6 地址。因此，我们可以使用它编写不依赖于所使用 IP 版本的程序。所有新代码应使用 *getaddrinfo()* 将主机名和服务名转换为二进制表示。

*getnameinfo()* 函数执行反向转换，将 IP 地址和端口号转换为相应的主机名和服务名。

我们还可以使用 *getaddrinfo()* 和 *getnameinfo()* 将二进制 IP 地址转换为表示格式，或将其从表示格式转换回来。

在 协议独立的主机和服务转换 中讨论的 *getaddrinfo()* 和 *getnameinfo()*，需要附带描述 DNS（域名系统 (DNS)")) 和 `/etc/services` 文件（`/etc/services` 文件）。DNS 允许合作的服务器维护一个分布式数据库，将二进制 IP 地址映射到主机名，反之亦然。像 DNS 这样的系统的存在对于互联网的运作至关重要，因为集中管理庞大的互联网主机名集合是不可能的。`/etc/services` 文件将端口号映射到符号服务名称。

## *inet_pton()* 和 *inet_ntop()* 函数

*inet_pton()* 和 *inet_ntop()* 函数允许将 IPv4 和 IPv6 地址在二进制形式和点分十进制或十六进制字符串表示法之间进行转换。

```
#include <arpa/inet.h>

int `inet_pton`(int *domain*, const char **src_str*, void **addrptr*);
```

### 注意

成功转换时返回 1，如果 *src_str* 不是表示形式格式，则返回 0，或者在错误时返回 -1。

```
const char *`inet_ntop`(int *domain*, const void **addrptr*,
 char **dst_str*, size_t *len*);
```

### 注意

成功时返回指向 *dst_str* 的指针，错误时返回 NULL。

这些函数名称中的 *p* 代表“表示形式”（presentation），而 n 代表“网络”（network）。表示形式是人类可读的字符串，例如以下示例：

+   `204.152.189.116`（IPv4 点分十进制地址）；

+   `::1`（IPv6 冒号分隔的十六进制地址）；或者

+   `::FFFF:204.152.189.116`（IPv4 映射的 IPv6 地址）。

*inet_pton()* 函数将 *src_str* 中包含的表示字符串转换为网络字节顺序的二进制 IP 地址。*domain* 参数应指定为 `AF_INET` 或 `AF_INET6`。转换后的地址将放置在 *addrptr* 指向的结构体中，*addrptr* 应该指向一个 *in_addr* 或 *in6_addr* 结构体，这取决于 *domain* 中指定的值。

*inet_ntop()* 函数执行反向转换。同样，*domain* 应该指定为 `AF_INET` 或 `AF_INET6`，*addrptr* 应该指向我们希望转换的 *in_addr* 或 *in6_addr* 结构体。转换后的以空字符终止的字符串将被放置在 *dst_str* 指向的缓冲区中。*len* 参数必须指定该缓冲区的大小。成功时，*inet_ntop()* 返回 *dst_str*。如果 *len* 太小，则 *inet_ntop()* 返回 `NULL`，并将 *errno* 设置为 `ENOSPC`。

为了正确设置 *dst_str* 指向的缓冲区大小，我们可以使用 `<netinet/in.h>` 中定义的两个常量。这些常量表示 IPv4 和 IPv6 地址的表示字符串的最大长度（包括终止的空字节）：

```
#define INET_ADDRSTRLEN  16     /* Maximum IPv4 dotted-decimal string */
#define INET6_ADDRSTRLEN 46     /* Maximum IPv6 hexadecimal string */
```

我们将在下一节提供 *inet_pton()* 和 *inet_ntop()* 使用的示例。

## 客户端-服务器示例（数据报套接字）

在本节中，我们将 UNIX 域中的数据报套接字中展示的大小写转换服务器和客户端程序修改为使用`AF_INET6`域中的数据报套接字。由于其结构与早期的程序类似，我们将以最少的注释来展示这些程序。新程序的主要区别在于 IPv6 套接字地址结构的声明和初始化，我们在第 59.4 节中对此进行了描述。

客户端和服务器都使用了示例 59-2 中展示的头文件。该头文件定义了服务器的端口号以及客户端和服务器可以交换的最大消息大小。

示例 59-2. `i6d_ucase_sv.c`和`i6d_ucase_cl.c`使用的头文件

```
`sockets/i6d_ucase.h`
#include <netinet/in.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <ctype.h>
#include "tlpi_hdr.h"

#define BUF_SIZE 10                     /* Maximum size of messages exchanged
                                           between client and server */

#define PORT_NUM 50002                  /* Server port number */
     `sockets/i6d_ucase.h`
```

示例 59-3 展示了服务器程序。服务器使用*inet_ntop()*函数将客户端的主机地址（通过*recvfrom()*调用获得）转换为可打印的形式。

在示例 59-4 中显示的客户端程序，相比早期的 UNIX 域版本(示例 57-7, 在示例程序中)，有两个显著的修改。第一个区别是客户端将其初始的命令行参数解释为服务器的 IPv6 地址。（剩余的命令行参数作为单独的数据报传递给服务器。）客户端使用*inet_pton()*将服务器地址转换为二进制形式。另一个区别是客户端没有将其套接字绑定到地址。如端口号中所述，如果一个互联网域套接字没有绑定到地址，内核会将套接字绑定到主机系统的一个临时端口。我们可以在以下的 Shell 会话日志中观察到这一点，在该日志中，我们在同一主机上运行了服务器和客户端：

```
$ `./i6d_ucase_sv &`
[1] 31047
$ `./i6d_ucase_cl ::1 ciao`                     *Send to server on local host*
Server received 4 bytes from (::1, 32770)
Response 1: CIAO
```

从上述输出中，我们看到服务器的*recvfrom()*调用能够获得客户端套接字的地址，包括临时端口号，尽管客户端没有进行*bind()*操作。

示例 59-3. 使用数据报套接字的 IPv6 大小写转换服务器

```
`sockets/i6d_ucase_sv.c`
#include "i6d_ucase.h"

int
main(int argc, char *argv[])
{
    struct sockaddr_in6 svaddr, claddr;
    int sfd, j;
    ssize_t numBytes;
    socklen_t len;
    char buf[BUF_SIZE];
    char claddrStr[INET6_ADDRSTRLEN];

    sfd = socket(AF_INET6, SOCK_DGRAM, 0);
    if (sfd == -1)
        errExit("socket");

    memset(&svaddr, 0, sizeof(struct sockaddr_in6));
    svaddr.sin6_family = AF_INET6;
    svaddr.sin6_addr = in6addr_any;                    /* Wildcard address */
    svaddr.sin6_port = htons(PORT_NUM);

    if (bind(sfd, (struct sockaddr *) &svaddr,
                sizeof(struct sockaddr_in6)) == -1)
        errExit("bind");

    /* Receive messages, convert to uppercase, and return to client */

    for (;;) {
        len = sizeof(struct sockaddr_in6);
        numBytes = recvfrom(sfd, buf, BUF_SIZE, 0,
                            (struct sockaddr *) &claddr, &len);
        if (numBytes == -1)
            errExit("recvfrom");

        if (inet_ntop(AF_INET6, &claddr.sin6_addr, claddrStr,
                    INET6_ADDRSTRLEN) == NULL)
            printf("Couldn't convert client address to string\n");
        else
            printf("Server received %ld bytes from (%s, %u)\n",
                    (long) numBytes, claddrStr, ntohs(claddr.sin6_port));

        for (j = 0; j < numBytes; j++)
            buf[j] = toupper((unsigned char) buf[j]);

        if (sendto(sfd, buf, numBytes, 0, (struct sockaddr *) &claddr, len) !=
                numBytes)
            fatal("sendto");
    }
}
      `sockets/i6d_ucase_sv.c`
```

示例 59-4. 使用数据报套接字的 IPv6 大小写转换客户端

```
`sockets/i6d_ucase_cl.c`
#include "i6d_ucase.h"

int
main(int argc, char *argv[])
{
    struct sockaddr_in6 svaddr;
    int sfd, j;
    size_t msgLen;
    ssize_t numBytes;
    char resp[BUF_SIZE];

    if (argc < 3 || strcmp(argv[1], "--help") == 0)
        usageErr("%s host-address msg...\n", argv[0]);

    sfd = socket(AF_INET6, SOCK_DGRAM, 0);      /* Create client socket */
    if (sfd == -1)
        errExit("socket");

    memset(&svaddr, 0, sizeof(struct sockaddr_in6));
    svaddr.sin6_family = AF_INET6;
    svaddr.sin6_port = htons(PORT_NUM);
    if (inet_pton(AF_INET6, argv[1], &svaddr.sin6_addr) <= 0)
        fatal("inet_pton failed for address '%s'", argv[1]);

    /* Send messages to server; echo responses on stdout */

    for (j = 2; j < argc; j++) {
        msgLen = strlen(argv[j]);
        if (sendto(sfd, argv[j], msgLen, 0, (struct sockaddr *) &svaddr,
                    sizeof(struct sockaddr_in6)) != msgLen)
            fatal("sendto");

        numBytes = recvfrom(sfd, resp, BUF_SIZE, 0, NULL, NULL);
        if (numBytes == -1)
            errExit("recvfrom");

        printf("Response %d: %.*s\n", j - 1, (int) numBytes, resp);
    }

    exit(EXIT_SUCCESS);
}
      `sockets/i6d_ucase_cl.c`
```

## 域名系统（DNS）

在协议无关的主机与服务转换中，我们描述了 *getaddrinfo()*，它获取与主机名对应的 IP 地址，以及 *getnameinfo()*，它执行反向操作。然而，在查看这些函数之前，我们将先解释如何使用 DNS 来维护主机名与 IP 地址之间的映射。

在 DNS 出现之前，主机名与 IP 地址之间的映射是定义在一个手动维护的本地文件`/etc/hosts`中的，该文件包含如下形式的记录：

```
# IP-address    canonical hostname      [aliases]
127.0.0.1       localhost
```

*gethostbyname()* 函数（*getaddrinfo()* 的前身）通过搜索此文件来获取 IP 地址，寻找与规范主机名（即主机的官方或主名称）或其中一个（可选的、空格分隔的）别名匹配的记录。

然而，`/etc/hosts` 方案的扩展性较差，随着网络中主机数量的增加（例如互联网，拥有数百万主机），这种方法变得不可行。

DNS 的设计就是为了解决这个问题。DNS 的关键思想如下：

+   主机名被组织成一个层次化的命名空间（图 59-2）。DNS 层次结构中的每个*节点*都有一个*标签*（名称），该名称最多可以包含 63 个字符。层次结构的根节点是一个没有名称的节点，称为“匿名根节点”。

+   一个节点的*域名*由从该节点到根节点的所有名称组成，名称之间用句点（`.`）分隔。例如，`google.com` 是节点 `google` 的域名。

+   *完全限定域名*（FQDN），例如 [www.kernel.org](http://www.kernel.org)。，在层级结构中标识一个主机。完全限定域名的特点是以句点结尾，尽管在许多上下文中，句点可能会被省略。

+   没有单一的组织或系统管理整个层级结构。相反，存在一个 DNS 服务器层级，每个服务器管理树的一个分支（*区域*）。通常，每个区域有一个*主名称服务器*，以及一个或多个*从名称服务器*（有时也称为*二级主名称服务器*），它们在主名称服务器崩溃时提供备份。区域本身也可以被划分为多个单独管理的较小区域。当在区域内添加主机，或更改主机名到 IP 地址的映射时，负责相应本地名称服务器的管理员会更新该服务器上的名称数据库。（层级结构中其他任何名称服务器数据库无需手动更改。）

    ### 注意

    在 Linux 上使用的 DNS 服务器实现是广泛使用的伯克利互联网名称域（BIND）实现，*named(8)*，由*互联网系统协会*（[`www.isc.org/`](http://www.isc.org/)）维护。该守护进程的操作由文件`/etc/named.conf`控制（请参阅*named.conf(5)*手册页）。关于 DNS 和 BIND 的关键参考资料是[Albitz & Liu, 2006]。有关 DNS 的信息也可以在[Stevens, 1994]的第十四章、[Stevens et al., 2004]的第十一章以及[Comer, 2000]的第二十四章中找到。

+   当一个程序调用*getaddrinfo()*来*解析*（即获取域名的 IP 地址）一个域名时，*getaddrinfo()*会使用一组库函数（*解析库*），这些函数与本地 DNS 服务器进行通信。如果该服务器无法提供所需的信息，它会与层级中的其他 DNS 服务器通信以获取该信息。有时，这个解析过程可能需要相当长的时间，而 DNS 服务器使用缓存技术来避免对频繁查询的域名进行不必要的通信。

使用上述方法使得 DNS 能够应对大型命名空间，并且不需要对名称进行集中管理。

![DNS 层次结构的一个子集](img/59-2_SOCKINET-DNS-hierarchy-scale90.png.jpg)图 59-2. DNS 层次结构的一个子集

#### 递归和迭代解析请求

DNS 解析请求分为两类：*递归*和*迭代*。在递归请求中，请求方要求服务器处理整个解析任务，包括与其他 DNS 服务器的通信任务（如有必要）。当本地主机上的应用程序调用*getaddrinfo()*时，该函数会向本地 DNS 服务器发起递归请求。如果本地 DNS 服务器没有足够的信息进行解析，它会迭代地解析域名。

我们通过一个示例来解释迭代解析。假设本地 DNS 服务器被请求解析名称[www.otago.ac.nz](http://www.otago.ac.nz)。为此，它首先与一个小范围的*根名称服务器*进行通信，每个 DNS 服务器都必须知道这些服务器。（我们可以通过命令*dig . NS*或访问[`www.root-servers.org/`](http://www.root-servers.org/)获取这些服务器的列表。）给定名称[www.otago.ac.nz](http://www.otago.ac.nz)，根名称服务器将本地 DNS 服务器引导到其中一个 nz DNS 服务器。本地 DNS 服务器随后使用名称[www.otago.ac.nz](http://www.otago.ac.nz)查询`nz`服务器，并收到一个响应，将其指向 ac.nz 服务器。本地 DNS 服务器接着用名称[www.otago.ac.nz](http://www.otago.ac.nz)查询 ac.nz 服务器，并被指引到 otago.ac.nz 服务器。最后，本地 DNS 服务器用名称[www.otago.ac.nz](http://www.otago.ac.nz)查询`otago.ac.nz`服务器，并获得所需的 IP 地址。

如果我们向*gethostbyname()*提供一个不完整的域名，解析器会在解析之前尝试将其补全。域名如何补全的规则定义在`/etc/resolv.conf`文件中（请参阅*resolv.conf(5)*手册页）。默认情况下，解析器至少会尝试使用本地主机的域名进行补全。例如，如果我们在机器`oghma.otago.ac.nz`上登录并输入命令*ssh octavo*，生成的 DNS 查询将是`octavo.otago.ac.nz`。

#### 顶级域名

匿名根节点下方的节点构成了所谓的*顶级域名*（TLD）。（在这些节点下方是*二级域名*，依此类推。）TLD 分为两类：*通用*和*国家*。

历史上，有七个*通用*TLD，大多数可以视为国际性的。在图 59-2 中，我们展示了四个最初的通用 TLD。其他三个是`int`、`mil`和`gov`；后两者保留给美国。近年来，添加了许多新的通用 TLD（例如，`info`、`name`和`museum`）。

每个国家都有一个对应的*国家*（或*地理*）TLD（标准化为 ISO 3166-1），其名称由两个字符组成。在图 59-2 中，我们展示了其中的一些：`de`（德国，*Deutschland*）、`eu`（欧洲联盟的超国家地理 TLD）、`nz`（新西兰）和`us`（美国）。几个国家将其 TLD 划分为一组类似于通用域名的二级域名。例如，新西兰有`ac.nz`（学术机构）、`co.nz`（商业）和`govt.nz`（政府）。

## `/etc/services` 文件

如端口号中所述，知名端口号由 IANA 集中注册。每个端口都有一个对应的*服务名称*。由于服务号是集中管理的，且比 IP 地址更稳定，因此通常不需要类似 DNS 服务器的等效机制。相反，端口号和服务名称被记录在`/etc/services`文件中。*getaddrinfo()*和*getnameinfo()*函数使用该文件中的信息将服务名称转换为端口号，反之亦然。

`/etc/services`文件由包含三列的行组成，如下例所示：

```
# Service name  port/protocol  [aliases]
echo            7/tcp          Echo     # echo service
echo            7/udp          Echo
ssh             22/tcp                  # Secure Shell
ssh             22/udp
telnet          23/tcp                  # Telnet
telnet          23/udp
smtp            25/tcp                  # Simple Mail Transfer Protocol
smtp            25/udp
domain          53/tcp                  # Domain Name Server
domain          53/udp
http            80/tcp                  # Hypertext Transfer Protocol
http            80/udp
ntp             123/tcp                 # Network Time Protocol
ntp             123/udp
login           513/tcp                 # rlogin(1)
who             513/udp                 # rwho(1)
shell           514/tcp                 # rsh(1)
syslog          514/udp                 # syslog
```

*协议*通常为`tcp`或`udp`。可选的（空格分隔的）*别名*指定服务的替代名称。除上述内容外，行中还可以包含以`#`字符开头的注释。

如前所述，给定的端口号对于 UDP 和 TCP 表示不同的实体，但 IANA 政策为服务分配了这两个端口号，即使该服务仅使用其中一个协议。例如，*telnet*、*ssh*、HTTP 和 SMTP 都使用 TCP，但也为这些服务分配了相应的 UDP 端口。反之，NTP 仅使用 UDP，但 TCP 端口 123 也分配给该服务。在某些情况下，服务同时使用 UDP 和 TCP；DNS 和*echo*就是此类服务的例子。最后，也有少数情况，UDP 和 TCP 端口号相同但分配给不同的服务；例如，*rsh*使用 TCP 端口 514，而*syslog*守护进程（使用*syslog*记录消息和错误）使用 UDP 端口 514。这是因为这些端口号在采用现行 IANA 政策之前就已被分配。

### 注意

`/etc/services`文件仅是名称到数字映射的记录。它不是一种预留机制：端口号出现在`/etc/services`中并不能保证该端口号实际可供某个服务绑定。

## 协议独立的主机和服务转换

*getaddrinfo()*函数将主机和服务名称转换为 IP 地址和端口号。它在 POSIX.1g 中被定义为（可重入）替代过时的*gethostbyname()*和*getservbyname()*函数。（用*getaddrinfo()*替代*gethostbyname()*的使用，允许我们消除程序中的 IPv4 与 IPv6 依赖性。）

*getnameinfo()*函数是*getaddrinfo()*的逆操作。它将套接字地址结构（无论是 IPv4 还是 IPv6）转换为包含对应主机和服务名称的字符串。该函数是（可重入）过时的*gethostbyaddr()*和*getservbyport()*函数的等效物。

### 注意

第十一章 中的 [Stevens 等，2004] 详细描述了 *getaddrinfo()* 和 *getnameinfo()*，并提供了这些函数的实现。这些函数也在 RFC 3493 中进行了描述。

### *getaddrinfo()* 函数

给定一个主机名和一个服务名，*getaddrinfo()* 返回一个套接字地址结构体列表，每个结构体包含一个 IP 地址和端口号。

```
#include <sys/socket.h>
#include <netdb.h>

int `getaddrinfo`(const char **host*, const char **service*,
                const struct addrinfo **hints*, struct addrinfo ***result*);
```

### 注意

成功时返回 0，出错时返回非零值

作为输入，*getaddrinfo()* 接受 *host*、*service* 和 *hints* 这三个参数。*host* 参数包含主机名或数字地址字符串，表示为 IPv4 点分十进制表示法或 IPv6 十六进制字符串表示法。（更准确地说，*getaddrinfo()* 接受 IPv4 数字字符串，以 *inet_aton()* 和 *inet_ntoa()* 函数 和 inet_ntoa() 函数") 中描述的更一般的数字与点的表示法。）*service* 参数包含服务名或十进制端口号。*hints* 参数指向一个 *addrinfo* 结构体，该结构体指定了进一步选择通过 *result* 返回的套接字地址结构体的标准。我们将在下面更详细地描述 *hints* 参数。

作为输出，*getaddrinfo()* 动态分配一个 *addrinfo* 结构体的链表，并将 *result* 设置为指向该链表的开始位置。这些 *addrinfo* 结构体中的每一个都包含一个指向对应于 *host* 和 *service* 的套接字地址结构的指针（图 59-3 分配并返回的结构体")）。*addrinfo* 结构体的形式如下：

```
struct addrinfo {
    int    ai_flags;            /* Input flags (AI_* constants) */
    int    ai_family;           /* Address family */
    int    ai_socktype;         /* Type: SOCK_STREAM, SOCK_DGRAM */
    int    ai_protocol;         /* Socket protocol */
    size_t ai_addrlen;          /* Size of structure pointed to by ai_addr */
    char  *ai_canonname;        /* Canonical name of host */
    struct sockaddr *ai_addr;   /* Pointer to socket address structure */
    struct addrinfo *ai_next;   /* Next structure in linked list */
};
```

*result* 参数返回一个结构体列表，而不是单一的结构体，因为可能存在多个主机和服务组合与 *host*、*service* 和 *hints* 中指定的标准相对应。例如，对于一个有多个网络接口的主机，可能会返回多个地址结构。此外，如果 *hints.ai_socktype* 被指定为 0，那么两个结构体可能会被返回——一个用于 `SOCK_DGRAM` 套接字，另一个用于 `SOCK_STREAM` 套接字——如果给定的 *service* 同时适用于 UDP 和 TCP。

每个通过*result*返回的*addrinfo*结构的字段描述了相关套接字地址结构的属性。*ai_family*字段被设置为`AF_INET`或`AF_INET6`，告知我们套接字地址结构的类型。*ai_socktype*字段被设置为`SOCK_STREAM`或`SOCK_DGRAM`，指示该地址结构是用于 TCP 还是 UDP 服务。*ai_protocol*字段返回适用于地址族和套接字类型的协议值。（这三个字段*ai_family*、*ai_socktype*和*ai_protocol*提供了在调用*socket()*创建此地址的套接字时所需的参数值。）*ai_addrlen*字段给出了由*ai_addr*指向的套接字地址结构的大小（以字节为单位）。*in_addr*字段指向套接字地址结构（IPv4 的*in_addr*结构或 IPv6 的*in6_addr*结构）。*ai_flags*字段未使用（它用于*hints*参数）。*ai_canonname*字段仅在第一个*addrinfo*结构中使用，且只有在*hints.ai_flags*中使用了`AI_CANONNAME`标志时才会使用，如下所述。

与*gethostbyname()*类似，*getaddrinfo()*可能需要向 DNS 服务器发送请求，而此请求可能需要一些时间才能完成。对于*getnameinfo()*也是如此，我们在《*getnameinfo()*函数》 Function")中进行了描述。

我们在客户端-服务器示例（流套接字）")中演示了*getaddrinfo()*的使用。

![getaddrinfo()分配并返回的结构](img/59-3_SOCKINET-addrinfo-list-scale90.png.jpg)图 59-3. *getaddrinfo()*分配并返回的结构

#### *hints*参数

*hints*参数指定了选择*getaddrinfo()*返回的套接字地址结构的进一步标准。在作为*hints*参数使用时，只有*ai_flags*、*ai_family*、*ai_socktype*和*ai_protocol*字段可以设置。其他字段未使用，应该根据需要初始化为 0 或`NULL`。

*hints.ai_family*字段选择返回的套接字地址结构的域。它可以指定为`AF_INET`或`AF_INET6`（如果实现支持，也可以是其他`AF_*`常量）。如果我们希望返回所有类型的套接字地址结构，可以为此字段指定值`AF_UNSPEC`。

*hints.ai_socktype*字段指定返回的地址结构将用于的套接字类型。如果我们将此字段指定为`SOCK_DGRAM`，则会执行 UDP 服务的查找，并通过*result*返回相应的套接字地址结构。如果我们指定为`SOCK_STREAM`，则会执行 TCP 服务的查找。如果*hints.ai_socktype*指定为 0，则接受任何套接字类型。

*hints.ai_protocol*字段选择返回地址结构的套接字协议。对于我们的用途，该字段始终指定为 0，这意味着调用者将接受任何协议。

*hints.ai_flags*字段是一个位掩码，用于修改*getaddrinfo()*的行为。此字段通过对以下值进行按位或操作来形成：

`AI_ADDRCONFIG`

仅在本地系统配置了至少一个 IPv4 地址（不包括 IPv4 回环地址）时，返回 IPv4 地址；仅在本地系统配置了至少一个 IPv6 地址（不包括 IPv6 回环地址）时，返回 IPv6 地址。

`AI_ALL`

请参阅下面关于`AI_V4MAPPED`的描述。

`AI_CANONNAME`

如果*host*不是`NULL`，则返回指向包含主机规范名称的空终止字符串的指针。该指针在通过*result*返回的第一个*addrinfo*结构的*ai_canonname*字段指向的缓冲区中返回。

`AI_NUMERICHOST`

强制将*host*解释为数字地址字符串。这用于防止在不必要的情况下进行名称解析，因为名称解析可能会耗时。

`AI_NUMERICSERV`

将*service*解释为数字端口号。此标志防止调用任何名称解析服务，当*service*是数字字符串时，这些服务是不需要的。

`AI_PASSIVE`

返回适用于被动打开（即监听套接字）的套接字地址结构。在这种情况下，*host*应为`NULL`，由*result*返回的套接字地址结构中的 IP 地址部分将包含通配符 IP 地址（即`INADDR_ANY`或`IN6ADDR_ANY_INIT`）。如果未设置此标志，则通过*result*返回的地址结构将适用于*connect()*和*sendto()*；如果*host*为`NULL`，则返回的套接字地址结构中的 IP 地址将设置为回环 IP 地址（根据域名，可能是`INADDR_LOOPBACK`或`IN6ADDR_LOOPBACK_INIT`）。

`AI_V4MAPPED`

如果在*hints*的*ai_family*字段中指定了`AF_INET6`，则如果找不到匹配的 IPv6 地址，应在*result*中返回 IPv4 映射的 IPv6 地址结构。如果同时指定了`AI_ALL`和`AI_V4MAPPED`，则在*result*中返回 IPv6 和 IPv4 地址结构，IPv4 地址将作为 IPv4 映射的 IPv6 地址结构返回。

如上所述，对于`AI_PASSIVE`，*host*可以指定为`NULL`。还可以将*service*指定为`NULL`，在这种情况下，返回地址结构中的端口号将设置为 0（即我们只关心将主机名解析为地址）。但是，不允许将*host*和*service*都指定为`NULL`。

如果我们不需要在 *hints* 中指定任何上述选择条件，则 *hints* 可以指定为 `NULL`，在这种情况下，*ai_socktype* 和 *ai_protocol* 默认为 0，*ai_flags* 默认为 `(AI_V4MAPPED | AI_ADDRCONFIG)`，*ai_family* 默认为 `AF_UNSPEC`。（*glibc* 实现故意偏离了 SUSv3，SUSv3 中规定，如果 *hints* 为 `NULL`，则 *ai_flags* 默认为 0。）

### 释放 *addrinfo* 列表： *freeaddrinfo()*

*getaddrinfo()* 函数动态分配内存给所有由 *result* 引用的结构体（图 59-3 分配并返回的结构")）。因此，调用者必须在这些结构体不再需要时释放它们。提供了 *freeaddrinfo()* 函数，以便通过一步操作方便地执行释放。

```
#include <sys/socket.h>
#include <netdb.h>

void `freeaddrinfo`(struct addrinfo **result*);
```

如果我们希望保留其中一个 *addrinfo* 结构体或其关联的套接字地址结构体的副本，那么在调用 *freeaddrinfo()* 之前，我们必须复制这些结构体。

### 错误诊断： *gai_strerror()*

出现错误时，*getaddrinfo()* 返回 表 59-1 和 getnameinfo() 的错误返回") 中所示的非零错误代码之一。

表 59-1. *getaddrinfo()* 和 *getnameinfo()* 的错误返回

| 错误常量 | 描述 |
| --- | --- |
| `EAI_ADDRFAMILY` | 在 *hints.ai_family* 中没有 *host* 的地址（SUSv3 中没有定义，但大多数实现中有定义；仅适用于 *getaddrinfo()*） |
| `EAI_AGAIN` | 名称解析暂时失败（稍后再试） |
| `EAI_BADFLAGS` | 在 *hints.ai_flags* 中指定了无效的标志 |
| `EAI_FAIL` | 访问名称服务器时发生无法恢复的故障 |
| `EAI_FAMILY` | 在 *hints.ai_family* 中指定的地址族不受支持 |
| `EAI_MEMORY` | 内存分配失败 |
| `EAI_NODATA` | *host* 没有关联的地址（SUSv3 中没有定义，但大多数实现中有定义；仅适用于 *getaddrinfo()*） |
| `EAI_NONAME` | 未知的 *host* 或 *service*，或者 *host* 和 *service* 都为 `NULL`，或者指定了 `AI_NUMERICSERV` 但 *service* 没有指向数字字符串 |
| `EAI_OVERFLOW` | 参数缓冲区溢出 |
| `EAI_SERVICE` | 指定的 *service* 不支持 *hints.ai_socktype*（仅适用于 *getaddrinfo()*） |
| `EAI_SOCKTYPE` | 指定的 *hints.ai_socktype* 不受支持（仅适用于 *getaddrinfo()*） |
| `EAI_SYSTEM` | 系统错误，返回值为 *errno* |

给定 表 59-1 和 getnameinfo() 的错误返回") 中的某个错误代码，*gai_strerror()* 函数会返回描述该错误的字符串。 （该字符串通常比 表 59-1 和 getnameinfo() 的错误返回") 中的描述更简洁。）

```
#include <netdb.h>

const char *`gai_strerror`(int *errcode*);
```

### 注意

返回指向包含错误信息的字符串的指针

我们可以使用由*gai_strerror()*返回的字符串作为应用程序显示的错误消息的一部分。

### *getnameinfo()*函数

*getnameinfo()*函数是*getaddrinfo()*的反操作。给定一个套接字地址结构（无论是 IPv4 还是 IPv6），它返回包含相应主机和服务名称的字符串，或者如果无法解析名称，则返回数字等效项。

```
#include <sys/socket.h>
#include <netdb.h>

int `getnameinfo`(const struct sockaddr **addr*, socklen_t *addrlen*, char **host*,
                size_t *hostlen*, char **service*, size_t *servlen*, int *flags*);
```

### 注意

成功时返回 0，出错时返回非零值。

*addr*参数是指向要转换的套接字地址结构的指针。该结构的长度由*addrlen*给出。通常，*addr*和*addrlen*的值是通过调用*accept()*、*recvfrom()*、*getsockname()*或*getpeername()*获得的。

返回的主机和服务名称作为以空字符结尾的字符串返回，存储在*host*和*service*指向的缓冲区中。这些缓冲区必须由调用者分配，并且其大小必须通过*hostlen*和*servlen*传递。`<netdb.h>`头文件定义了两个常量来帮助确定这些缓冲区的大小。`NI_MAXHOST`表示返回的主机名字符串的最大大小（以字节为单位）。它被定义为 1025。`NI_MAXSERV`表示返回的服务名字符串的最大大小（以字节为单位）。它被定义为 32。这两个常量在 SUSv3 中未指定，但在所有提供*getnameinfo()*的 UNIX 实现中都已定义。（自*glibc* 2.8 以来，我们必须定义其中一个特性文本宏`_BSD_SOURCE`、`_SVID_SOURCE`或`_GNU_SOURCE`，以获取`NI_MAXHOST`和`NI_MAXSERV`的定义。）

如果我们不关心获取主机名，可以将*host*指定为`NULL`，并将*hostlen*指定为 0。同样，如果我们不需要服务名称，可以将*service*指定为`NULL`，并将*servlen*指定为 0。然而，*host*和*service*中至少一个必须非`NULL`（并且相应的长度参数必须非零）。

最后的参数，*flags*，是一个位掩码，控制*getnameinfo()*的行为。以下常量可以通过按位或运算组合在一起形成该位掩码：

`NI_DGRAM`

默认情况下，*getnameinfo()*返回与*流*套接字（即 TCP）服务对应的名称。通常，这不会产生影响，因为正如在《/etc/services 文件》中所提到的，服务名称通常在对应的 TCP 和 UDP 端口之间相同。然而，在少数名称不同的情况下，`NI_DGRAM`标志会强制返回数据报套接字（即 UDP）服务的名称。

`NI_NAMEREQD`

默认情况下，如果无法解析主机名，则在*host*中返回数字地址字符串。如果指定了`NI_NAMEREQD`标志，则返回错误（`EAI_NONAME`）。

`NI_NOFQDN`

默认情况下，返回主机的完全限定域名。指定`NI_NOFQDN`标志时，若这是本地网络上的主机，则仅返回名称的第一部分（即主机名）。

`NI_NUMERICHOST`

强制返回一个数字地址字符串作为*host*。如果我们希望避免可能耗时的 DNS 服务器调用，这非常有用。

`NI_NUMERICSERV`

强制返回一个十进制端口号字符串作为*service*。在已知端口号不对应服务名称的情况下（例如，如果它是内核分配给套接字的临时端口号），这非常有用，我们希望避免不必要地搜索`/etc/services`带来的低效。

如果成功，*getnameinfo()* 返回 0。如果出错，它将返回表格 59-1 和 getnameinfo() 的错误返回")中列出的非零错误代码之一。

## 客户端-服务器示例（流套接字）

现在我们有足够的信息来查看一个使用 TCP 套接字的简单客户端-服务器应用程序。该应用程序执行的任务与 A Client-Server Application Using FIFOs 中介绍的 FIFO 客户端-服务器应用程序执行的任务相同：为客户端分配唯一的序列号（或序列号范围）。

为了处理服务器和客户端主机上整数可能以不同格式表示的情况，我们将所有传输的整数编码为以换行符结束的字符串，并使用我们的*readLine()*函数（示例 59-1）来读取这些字符串。

#### 公共头文件

服务器和客户端都包含示例 59-5 中显示的头文件。此文件包含其他头文件，并定义应用程序将使用的 TCP 端口号。

#### 服务器程序

示例 59-6 中显示的服务器程序执行以下步骤：

+   将服务器的序列号初始化为 1，或为可选命令行参数中提供的值 ![](img/U001.png)。

+   忽略`SIGPIPE`信号 ![](img/U002.png)。这防止服务器在尝试写入已关闭对等端的套接字时收到`SIGPIPE`信号；相反，*write()* 会因错误`EPIPE`而失败。

+   调用*getaddrinfo()* ![](img/U004.png)以获取一组用于 TCP 套接字的套接字地址结构，该套接字使用端口号`PORT_NUM`。（与使用硬编码端口号不同，我们通常会使用服务名称。）我们指定`AI_PASSIVE`标志 ![](img/U003.png)，以便生成的套接字将绑定到通配符地址（IP 地址）。因此，如果服务器在一个多宿主主机上运行，它可以接受发送到主机任何网络地址的连接请求。

+   进入一个循环，逐个处理前一步返回的套接字地址结构 ![](img/U005.png)。当程序找到一个可以成功创建并绑定套接字的地址结构时，循环终止 ![](img/U007.png)。

+   为在前一步创建的套接字设置`SO_REUSEADDR`选项 ![](img/U006.png)。我们将在《*SO_REUSEADDR* 套接字选项》中讨论此选项，届时我们会提到，TCP 服务器通常应在其监听套接字上设置此选项。

+   将套接字标记为监听套接字 ![](img/U008.png)。

+   启动一个无限`for`循环 ![](img/U009.png)，逐个处理客户端请求（第六十章）。每个客户端的请求在接受下一个客户端的请求之前会被处理。对于每个客户端，服务器执行以下步骤：

    +   接受一个新的连接 ![](img/U010.png)。服务器为*accept()*的第二个和第三个参数传递非`NULL`指针，以便获取客户端的地址。服务器在标准输出上显示客户端的地址（IP 地址加端口号） ![](img/U011.png)。

    +   读取客户端的消息 ![](img/U012.png)，该消息是一个以换行符结尾的字符串，指定客户端想要多少个序列号。服务器将此字符串转换为整数，并将其存储在变量*reqLen*中 ![](img/U013.png)。

    +   将当前序列号值(*seqNum*)发送回客户端，并将其编码为以换行符结尾的字符串 ![](img/U014.png)。客户端可以假设它已分配了从*seqNum*到*(seqNum + reqLen - 1)*范围内的所有序列号。

    +   通过将*reqLen*加到*seqNum*来更新服务器的序列号值 ![](img/U015.png)。

示例 59-5. `is_seqnum_sv.c`和`is_seqnum_cl.c`使用的头文件

```
`sockets/is_seqnum.h`
#include <netinet/in.h>
#include <sys/socket.h>
#include <signal.h>
#include "read_line.h"          /* Declaration of readLine() */
#include "tlpi_hdr.h"

#define PORT_NUM "50000"        /* Port number for server */

#define INT_LEN 30              /* Size of string able to hold largest
                                   integer (including terminating '\n') */xs
     `sockets/is_seqnum.h`
```

示例 59-6. 使用流套接字与客户端通信的迭代服务器

```
`sockets/is_seqnum_sv.c`
    #define _BSD_SOURCE             /* To get definitions of NI_MAXHOST and
                                       NI_MAXSERV from <netdb.h> */
    #include <netdb.h>
    #include "is_seqnum.h"

    #define BACKLOG 50

    int
    main(int argc, char *argv[])
    {
        uint32_t seqNum;
        char reqLenStr[INT_LEN];            /* Length of requested sequence */
        char seqNumStr[INT_LEN];            /* Start of granted sequence */
        struct sockaddr_storage claddr;
        int lfd, cfd, optval, reqLen;
        socklen_t addrlen;
        struct addrinfo hints;
        struct addrinfo *result, *rp;
    #define ADDRSTRLEN (NI_MAXHOST + NI_MAXSERV + 10)
        char addrStr[ADDRSTRLEN];
        char host[NI_MAXHOST];
        char service[NI_MAXSERV];

        if (argc > 1 && strcmp(argv[1], "--help") == 0)
            usageErr("%s [init-seq-num]\n", argv[0]);
    seqNum = (argc > 1) ? getInt(argv[1], 0, "init-seq-num") : 0;
    if (signal(SIGPIPE, SIG_IGN) == SIG_ERR)
            errExit("signal");

        /* Call getaddrinfo() to obtain a list of addresses that
           we can try binding to */

        memset(&hints, 0, sizeof(struct addrinfo));
        hints.ai_canonname = NULL;
        hints.ai_addr = NULL;
        hints.ai_next = NULL;
        hints.ai_socktype = SOCK_STREAM;
        hints.ai_family = AF_UNSPEC;        /* Allows IPv4 or IPv6 */
    hints.ai_flags = AI_PASSIVE | AI_NUMERICSERV;
                            /* Wildcard IP address; service name is numeric */
    if (getaddrinfo(NULL, PORT_NUM, &hints, &result) != 0)
            errExit("getaddrinfo");

        /* Walk through returned list until we find an address structure
           that can be used to successfully create and bind a socket */

        optval = 1;
    for (rp = result; rp != NULL; rp = rp->ai_next) {
            lfd = socket(rp->ai_family, rp->ai_socktype, rp->ai_protocol);
            if (lfd == -1)
                continue;                   /* On error, try next address */
            if (setsockopt(lfd, SOL_SOCKET, SO_REUSEADDR, &optval, sizeof(optval))
                     == -1)
                 errExit("setsockopt");
        if (bind(lfd, rp->ai_addr, rp->ai_addrlen) == 0)
                break;                      /* Success */

            /* bind() failed: close this socket and try next address */

            close(lfd);
        }

        if (rp == NULL)
            fatal("Could not bind socket to any address");
    if (listen(lfd, BACKLOG) == -1)
            errExit("listen");

        freeaddrinfo(result);
    for (;;) {                 /* Handle clients iteratively */

            /* Accept a client connection, obtaining client's address */

            addrlen = sizeof(struct sockaddr_storage);
        cfd = accept(lfd, (struct sockaddr *) &claddr, &addrlen);
            if (cfd == -1) {
                errMsg("accept");
                continue;
            }
        if (getnameinfo((struct sockaddr *) &claddr, addrlen,
                        host, NI_MAXHOST, service, NI_MAXSERV, 0) == 0)
                snprintf(addrStr, ADDRSTRLEN, "(%s, %s)", host, service);
            else
                snprintf(addrStr, ADDRSTRLEN, "(?UNKNOWN?)");
            printf("Connection from %s\n", addrStr);

            /* Read client request, send sequence number back */
        if (readLine(cfd, reqLenStr, INT_LEN) <= 0) {
                close(cfd);
                continue;                   /* Failed read; skip request */
            }
        reqLen = atoi(reqLenStr);
            if (reqLen <= 0) {              /* Watch for misbehaving clients */
                close(cfd);
                continue;                   /* Bad request; skip it */
            }
        snprintf(seqNumStr, INT_LEN, "%d\n", seqNum);
            if (write(cfd, &seqNumStr, strlen(seqNumStr)) != strlen(seqNumStr))
                fprintf(stderr, "Error on write");

            seqNum += reqLen;               /* Update sequence number */

            if (close(cfd) == -1)           /* Close connection */
                errMsg("close");
        }
    }
          `sockets/is_seqnum_sv.c`
```

#### 客户端程序

客户端程序显示在示例 59-7 中。该程序接受两个参数。第一个参数是服务器运行所在主机的名称，这是必需的。可选的第二个参数是客户端期望的序列长度，默认长度为 1。客户端执行以下步骤：

+   调用*getaddrinfo()*获取一组适用于连接到指定主机上的 TCP 服务器的套接字地址结构！[](figs/web/U001.png)。对于端口号，客户端指定`PORT_NUM`。

+   输入一个循环！[](figs/web/U002.png)，该循环遍历上一步返回的套接字地址结构，直到客户端找到一个可以成功创建！[](figs/web/U003.png)并连接！[](figs/web/U004.png)套接字到服务器的结构。由于客户端尚未绑定其套接字，*connect()*调用导致内核为套接字分配一个临时端口。

+   发送一个整数，指定客户端期望的序列的长度！[](figs/web/U005.png)。该整数作为以换行符终止的字符串发送。

+   读取服务器返回的序列号（同样是一个以换行符终止的字符串）！[](figs/web/U006.png)，并将其打印到标准输出！[](figs/web/U007.png)。

当我们在同一主机上运行服务器和客户端时，我们看到以下情况：

```
$ `./is_seqnum_sv &`
[1] 4075
$ `./is_seqnum_cl localhost`              *Client 1: requests 1 sequence number*
Connection from (localhost, 33273)      *Server displays client address + port*
Sequence number: 0                      *Client displays returned sequence number*
$ `./is_seqnum_cl localhost 10`           *Client 2: requests 10 sequence numbers*
Connection from (localhost, 33274)
Sequence number: 1
$ `./is_seqnum_cl localhost`              *Client 3: requests 1 sequence number*
Connection from (localhost, 33275)
Sequence number: 11
```

接下来，我们演示如何使用*telnet*调试此应用程序：

```
$ `telnet localhost 50000`                *Our server uses this port number*
                                        *Empty line printed by* *telnet*
Trying 127.0..0.1...
Connection from (localhost, 33276)
Connected to localhost.
Escape character is '^]'.
`1`                                       *Enter length of requested sequence*
12                                      *telnet* *displays sequence number and*
Connection closed by foreign host.      *detects that server closed connection*
```

### 注意

在 shell 会话日志中，我们看到内核按顺序循环遍历临时端口号。(其他实现也表现出类似的行为。) 在 Linux 上，这种行为是为了优化内核本地套接字绑定表中的哈希查找。当达到这些端口号的上限时，内核会从范围的低端开始重新分配一个可用的端口号（该范围由 Linux 特有的`/proc/sys/net/ipv4/ip_local_port_range`文件定义）。

示例 59-7. 一个使用流套接字的客户端

```
`sockets/is_seqnum_cl.c`
    #include <netdb.h>
    #include "is_seqnum.h"

    int
    main(int argc, char *argv[])
    {
        char *reqLenStr;                    /* Requested length of sequence */
        char seqNumStr[INT_LEN];            /* Start of granted sequence */
        int cfd;
        ssize_t numRead;
        struct addrinfo hints;
        struct addrinfo *result, *rp;

        if (argc < 2 || strcmp(argv[1], "--help") == 0)
            usageErr("%s server-host [sequence-len]\n", argv[0]);

        /* Call getaddrinfo() to obtain a list of addresses that
           we can try connecting to */

        memset(&hints, 0, sizeof(struct addrinfo));
        hints.ai_canonname = NULL;
        hints.ai_addr = NULL;
        hints.ai_next = NULL;
        hints.ai_family = AF_UNSPEC;                /* Allows IPv4 or IPv6 */
        hints.ai_socktype = SOCK_STREAM;
        hints.ai_flags = AI_NUMERICSERV;
    if (getaddrinfo(argv[1], PORT_NUM, &hints, &result) != 0)
            errExit("getaddrinfo");

        /* Walk through returned list until we find an address structure
           that can be used to successfully connect a socket */
    for (rp = result; rp != NULL; rp = rp->ai_next) {
        cfd = socket(rp->ai_family, rp->ai_socktype, rp->ai_protocol);
            if (cfd == -1)
                continue;                           /* On error, try next address */
        if (connect(cfd, rp->ai_addr, rp->ai_addrlen) != -1)
                break;                              /* Success */

                /* Connect failed: close this socket and try next address */

            close(cfd);
        }

        if (rp == NULL)
            fatal("Could not connect socket to any address");

        freeaddrinfo(result);

        /* Send requested sequence length, with terminating newline */
    reqLenStr = (argc > 2) ? argv[2] : "1";
        if (write(cfd, reqLenStr, strlen(reqLenStr)) !=  strlen(reqLenStr))
            fatal("Partial/failed write (reqLenStr)");
        if (write(cfd, "\n", 1) != 1)
            fatal("Partial/failed write (newline)");

        /* Read and display sequence number returned by server */
    numRead = readLine(cfd, seqNumStr, INT_LEN);
        if (numRead == -1)
            errExit("readLine");
        if (numRead == 0)
            fatal("Unexpected EOF from server");
    printf("Sequence number: %s", seqNumStr);           /* Includes '\n' */

        exit(EXIT_SUCCESS);                                 /* Closes 'cfd' */
    }
          `sockets/is_seqnum_cl.c`
```

## 一个互联网域套接字库

在本节中，我们使用协议无关的主机和服务转换中介绍的功能，来实现一个用于执行常见任务的函数库，这些任务通常适用于互联网域套接字。 (该函数库抽象了客户端-服务器示例（流套接字）示例程序中展示的许多步骤。) 由于这些函数使用了协议无关的*getaddrinfo()*和*getnameinfo()*函数，因此它们可以同时用于 IPv4 和 IPv6。示例 59-8 展示了声明这些函数的头文件。

该库中的许多函数具有相似的参数：

+   *host* 参数是一个字符串，包含主机名或数字地址（IPv4 点分十进制或 IPv6 十六进制字符串表示法）。另外，*host* 也可以指定为 `NULL` 指针，表示使用回环 IP 地址。

+   *service* 参数是一个服务名称或端口号，指定为十进制字符串。

+   *type* 参数是套接字类型，可以指定为 `SOCK_STREAM` 或 `SOCK_DGRAM`。

示例 59-8. `inet_sockets.c` 的头文件

```
`sockets/inet_sockets.h`
#ifndef INET_SOCKETS_H
#define INET_SOCKETS_H          /* Prevent accidental double inclusion */

#include <sys/socket.h>
#include <netdb.h>

int inetConnect(const char *host, const char *service, int type);

int inetListen(const char *service, int backlog, socklen_t *addrlen);

int inetBind(const char *service, int type, socklen_t *addrlen);

char *inetAddressStr(const struct sockaddr *addr, socklen_t addrlen,
                char *addrStr, int addrStrLen);

#define IS_ADDR_STR_LEN 4096
                        /* Suggested length for string buffer that caller
                           should pass to inetAddressStr(). Must be greater
                           than (NI_MAXHOST + NI_MAXSERV + 4) */
#endif
      `sockets/inet_sockets.h`
```

*inetConnect()* 函数创建一个具有指定套接字 *type* 的套接字，并将其连接到由 *host* 和 *service* 指定的地址。此函数设计用于需要将其套接字连接到服务器套接字的 TCP 或 UDP 客户端。

```
#include "inet_sockets.h"

int `inetConnect`(const char **host*, const char **service*, int *type*);
```

### 注意

成功时返回文件描述符，错误时返回 -1

新套接字的文件描述符作为函数结果返回。

*inetListen()* 函数创建一个监听流 (`SOCK_STREAM`) 套接字，并绑定到 *service* 指定的 TCP 端口上的通配符 IP 地址。此函数设计供 TCP 服务器使用。

```
#include "inet_sockets.h"

int `inetListen`(const char **service*, int *backlog*, socklen_t **addrlen*);
```

### 注意

成功时返回文件描述符，错误时返回 -1

新套接字的文件描述符作为函数结果返回。

*backlog* 参数指定允许的挂起连接的最大数量（类似于 *listen()*）。

如果 *addrlen* 被指定为非 `NULL` 指针，则它指向的位置将用于返回与返回的文件描述符对应的套接字地址结构的大小。这个值使我们能够分配适当大小的套接字地址缓冲区，传递给后续的 *accept()* 调用，以便获取连接客户端的地址。

*inetBind()* 函数创建一个指定 *type* 的套接字，并将其绑定到由 *service* 和 *type* 指定的端口上的通配符 IP 地址。（套接字 *type* 表示这是一个 TCP 还是 UDP 服务。）此函数主要设计用于 UDP 服务器和客户端，创建一个绑定到特定地址的套接字。

```
#include "inet_sockets.h"

int `inetBind`(const char **service*, int *type*, socklen_t **addrlen*);
```

### 注意

成功时返回文件描述符，错误时返回 -1

新套接字的文件描述符作为函数结果返回。

与 *inetListen()* 类似，*inetBind()* 会返回一个与该套接字关联的套接字地址结构的长度，这个长度存储在 *addrlen* 指向的位置。如果我们想分配一个缓冲区并将其传递给 *recvfrom()* 以获取发送数据报的套接字地址，这个值就非常有用。（*inetListen()* 和 *inetBind()* 所需的许多步骤是相同的，这些步骤在库中由一个函数 *inetPassiveSocket()* 实现。）

*inetAddressStr()* 函数将互联网套接字地址转换为可打印的形式。

```
#include "inet_sockets.h"

char *`inetAddressStr`(const struct sockaddr **addr*, socklen_t *addrlen*,
                     char **addrStr*, int *addrStrLen*);
```

### 注意

返回指向 *addrStr* 的指针，*addrStr* 是一个包含主机和服务名称的字符串。

给定一个套接字地址结构 *addr*，其长度由 *addrlen* 指定，*inetAddressStr()* 返回一个以空字符终止的字符串，包含相应的主机名和端口号，格式如下：

```
(hostname, port-number)
```

字符串被返回在*addrStr*指向的缓冲区中。调用者必须在*addrStrLen*中指定此缓冲区的大小。如果返回的字符串超过了(*addrStrLen - 1*)字节，则会被截断。常量`IS_ADDR_STR_LEN`定义了*addrStr*缓冲区的建议大小，应该足够大以容纳所有可能的返回字符串。作为其函数结果，*inetAddressStr()*返回*addrStr*。

本节中描述的函数实现见示例 59-9。

示例 59-9. 一个互联网域套接字库

```
`sockets/inet_sockets.c`
#define _BSD_SOURCE             /* To get NI_MAXHOST and NI_MAXSERV
                                   definitions from <netdb.h> */
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <netdb.h>
#include "inet_sockets.h"       /* Declares functions defined here */
#include "tlpi_hdr.h"

int
inetConnect(const char *host, const char *service, int type)
{
    struct addrinfo hints;
    struct addrinfo *result, *rp;
    int sfd, s;

    memset(&hints, 0, sizeof(struct addrinfo));
    hints.ai_canonname = NULL;
    hints.ai_addr = NULL;
    hints.ai_next = NULL;
    hints.ai_family = AF_UNSPEC;        /* Allows IPv4 or IPv6 */
    hints.ai_socktype = type;

    s = getaddrinfo(host, service, &hints, &result);
    if (s != 0) {
        errno = ENOSYS;
        return -1;
    }

    /* Walk through returned list until we find an address structure
       that can be used to successfully connect a socket */

    for (rp = result; rp != NULL; rp = rp->ai_next) {
        sfd = socket(rp->ai_family, rp->ai_socktype, rp->ai_protocol);
        if (sfd == -1)
            continue;                   /* On error, try next address */

        if (connect(sfd, rp->ai_addr, rp->ai_addrlen) != -1)
            break;                      /* Success */

        /* Connect failed: close this socket and try next address */

        close(sfd);
    }

    freeaddrinfo(result);

    return (rp == NULL) ? -1 : sfd;
}

static int              /* Public interfaces: inetBind() and inetListen() */
inetPassiveSocket(const char *service, int type, socklen_t *addrlen,
                  Boolean doListen, int backlog)
{
    struct addrinfo hints;
    struct addrinfo *result, *rp;
    int sfd, optval, s;

    memset(&hints, 0, sizeof(struct addrinfo));
    hints.ai_canonname = NULL;
    hints.ai_addr = NULL;
    hints.ai_next = NULL;
    hints.ai_socktype = type;
    hints.ai_family = AF_UNSPEC;        /* Allows IPv4 or IPv6 */
    hints.ai_flags = AI_PASSIVE;        /* Use wildcard IP address */

    s = getaddrinfo(NULL, service, &hints, &result);
    if (s != 0)
        return -1;

    /* Walk through returned list until we find an address structure
       that can be used to successfully create and bind a socket */

    optval = 1;
    for (rp = result; rp != NULL; rp = rp->ai_next) {
        sfd = socket(rp->ai_family, rp->ai_socktype, rp->ai_protocol);
        if (sfd == -1)
            continue;                   /* On error, try next address */

        if (doListen) {
            if (setsockopt(sfd, SOL_SOCKET, SO_REUSEADDR, &optval,
                    sizeof(optval)) == -1) {
                close(sfd);
                freeaddrinfo(result);
                return -1;
            }
        }

        if (bind(sfd, rp->ai_addr, rp->ai_addrlen) == 0)
            break;                      /* Success */

        /* bind() failed: close this socket and try next address */

        close(sfd);
    }

    if (rp != NULL && doListen) {
        if (listen(sfd, backlog) == -1) {
            freeaddrinfo(result);
            return -1;
        }
    }

    if (rp != NULL && addrlen != NULL)
        *addrlen =  rp->ai_addrlen;     /* Return address structure size */
    freeaddrinfo(result);

    return (rp == NULL) ? -1 : sfd;
}

int
inetListen(const char *service, int backlog, socklen_t *addrlen)
{
    return inetPassiveSocket(service, SOCK_STREAM, addrlen, TRUE, backlog);
}

int
inetBind(const char *service, int type, socklen_t *addrlen)
{
    return inetPassiveSocket(service, type, addrlen, FALSE, 0);
}

char *
inetAddressStr(const struct sockaddr *addr, socklen_t addrlen,
               char *addrStr, int addrStrLen)
{
    char host[NI_MAXHOST], service[NI_MAXSERV];

    if (getnameinfo(addr, addrlen, host, NI_MAXHOST,
                    service, NI_MAXSERV, NI_NUMERICSERV) == 0)
        snprintf(addrStr, addrStrLen, "(%s, %s)", host, service);
    else
        snprintf(addrStr, addrStrLen, "(?UNKNOWN?)");

    addrStr[addrStrLen - 1] = '\0';     /* Ensure result is null-terminated */
    return addrStr;
}
      `sockets/inet_sockets.c`
```

## 主机和服务转换的过时 API

在以下各节中，我们将描述用于将主机名和服务名在二进制格式和表示格式之间转换的旧函数，这些函数现在已经过时。尽管新的程序应该使用本章前面描述的现代函数进行这些转换，但了解这些过时的函数仍然有用，因为我们可能会在旧代码中遇到它们。

### *inet_aton()*和*inet_ntoa()*函数

*inet_aton()*和*inet_ntoa()*函数在点分十进制表示法和二进制形式（网络字节顺序）之间转换 IPv4 地址。这些函数如今已被*inet_pton()*和*inet_ntop()*所取代。

*inet_aton*（“ASCII 到网络”）函数将*str*指向的点分十进制字符串转换为网络字节顺序中的 IPv4 地址，并返回该地址，存储在*addr*指向的*in_addr*结构中。

```
#include <arpa/inet.h>

int `inet_aton`(const char **str*, struct in_addr **addr*);
```

### 注意

如果*str*是有效的点分十进制地址，则返回 1（真），否则在出错时返回 0（假）。

*inet_aton()*函数在转换成功时返回 1，如果*str*无效则返回 0。

传递给*inet_aton()*的字符串的数字组件不必是十进制的。它们可以是八进制（以 0 开头）或十六进制（以 0x 或 0X 开头）。此外，*inet_aton()*支持简写形式，允许使用少于四个数字组件来指定地址。（有关详细信息，请参见*inet(3)*手册页。）术语*numbers-and-dots notation*用于指代使用这些特性的更通用的地址字符串。

SUSv3 未规定*inet_aton()*。然而，大多数实现都提供此函数。在 Linux 上，我们必须定义其中一个特性测试宏`_BSD_SOURCE`、`_SVID_SOURCE`或`_GNU_SOURCE`，才能从`<arpa/inet.h>`中获取*inet_aton()*的声明。

*inet_ntoa*（“网络到 ASCII”）函数执行*inet_aton()*的反向操作。

```
#include <arpa/inet.h>

char *`inet_ntoa`(struct in_addr *addr*);
```

### 注意

返回指向（静态分配的）点分十进制字符串版本的*addr*的指针。

给定一个*in_addr*结构（一个 32 位 IPv4 地址，采用网络字节顺序），*inet_ntoa()*返回一个指向包含该地址的点分十进制表示法的（静态分配的）字符串的指针。

由于*inet_ntoa()*返回的字符串是静态分配的，因此会被连续调用覆盖。

### *gethostbyname()* 和 *gethostbyaddr()* 函数

*gethostbyname()* 和 *gethostbyaddr()* 函数允许在主机名和 IP 地址之间进行转换。这些函数如今已被 *getaddrinfo()* 和 *getnameinfo()* 所淘汰。

```
#include <netdb.h>

extern int `h_errno`;

struct hostent *`gethostbyname`(const char **name*);
struct hostent *`gethostbyaddr`(const char **addr*, socklen_t *len*, int *type*);
```

### 注意

两者在成功时返回指向（静态分配的）*hostent* 结构体的指针，出错时返回 NULL。

*gethostbyname()* 函数解析给定的 *name* 主机名，返回指向一个静态分配的 *hostent* 结构体的指针，该结构体包含关于该主机名的信息。该结构体的形式如下：

```
struct hostent {
    char  *h_name;              /* Official (canonical) name of host */
    char **h_aliases;           /* NULL-terminated array of pointers
                                   to alias strings */
    int    h_addrtype;          /* Address type (AF_INET or AF_INET6) */
    int    h_length;            /* Length (in bytes) of addresses pointed
                                   to by h_addr_list (4 bytes for AF_INET,
                                   16 bytes for AF_INET6) */
    char **h_addr_list;         /* NULL-terminated array of pointers to
                                   host IP addresses (in_addr or in6_addr
                                   structures) in network byte order */
};

#define h_addr  h_addr_list[0]
```

*h_name* 字段返回主机的正式名称，作为一个以空字符终止的字符串。*h_aliases* 字段指向一个指针数组，数组中的指针指向包含该主机名别名（替代名称）的以空字符终止的字符串。

*h_addr_list* 字段是指向该主机的 IP 地址结构体的指针数组。（多宿主主机有多个地址。）此列表由 *in_addr* 或 *in6_addr* 结构体组成。我们可以通过 *h_addrtype* 字段来确定这些结构体的类型，该字段包含 `AF_INET` 或 `AF_INET6`，通过 *h_length* 字段来确定它们的长度。*h_addr* 定义是为了与早期实现（例如 4.2BSD）保持向后兼容，早期实现只在 *hostent* 结构体中返回一个地址。有些现有代码依赖于此名称（因此不支持多宿主主机）。

在现代版本的 *gethostbyname()* 中，*name* 还可以指定为一个数值 IP 地址字符串；即 IPv4 的数字点分表示法或 IPv6 的十六进制字符串表示法。在这种情况下，系统不会执行查找操作；相反，*name* 会被复制到 *hostent* 结构体的 *h_name* 字段中，*h_addr_list* 则被设置为 *name* 的二进制等效值。

*gethostbyaddr()* 函数执行 *gethostbyname()* 的反向操作。给定一个二进制 IP 地址，它返回一个 *hostent* 结构体，包含该地址主机的相关信息。

在出错时（例如无法解析某个名称），*gethostbyname()* 和 *gethostbyaddr()* 都返回一个 `NULL` 指针并设置全局变量 *h_errno*。顾名思义，该变量类似于 *errno*（此变量中可能的值在 *gethostbyname(3)* 手册页中有描述），而 *herror()* 和 *hstrerror()* 函数类似于 *perror()* 和 *strerror()*。

*herror()* 函数会在标准错误输出显示 *str* 中给定的字符串，后跟一个冒号（`:`），然后是当前错误在 *h_errno* 中的消息。或者，我们可以使用 *hstrerror()* 获取一个指向与 *err* 中指定的错误值对应的字符串的指针。

```
#define _BSD_SOURCE           /* Or _SVID_SOURCE or _GNU_SOURCE */
#include <netdb.h>

void `herror`(const char **str*);

const char *`hstrerror`(int *err*);
Returns pointer to *h_errno* error string corresponding to *err*
```

示例 59-10 获取主机信息") 演示了 *gethostbyname()* 的使用。该程序显示了命令行中列出的每个主机的 *hostent* 信息。以下 shell 会话演示了该程序的使用：

```
$ `./t_gethostbyname www.jambit.com`
Canonical name: jamjam1.jambit.com
        alias(es):      www.jambit.com
        address type:   AF_INET
        address(es):    62.245.207.90
```

示例 59-10. 使用 *gethostbyname()* 获取主机信息

```
`sockets/t_gethostbyname.c`
#define _BSD_SOURCE     /* To get hstrerror() declaration from <netdb.h> */
#include <netdb.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    struct hostent *h;
    char **pp;
    char str[INET6_ADDRSTRLEN];

    for (argv++; *argv != NULL; argv++) {
        h = gethostbyname(*argv);
        if (h == NULL) {
            fprintf(stderr, "gethostbyname() failed for '%s': %s\n",
                    *argv, hstrerror(h_errno));
            continue;
        }

        printf("Canonical name: %s\n", h->h_name);

        printf("        alias(es):     ");
        for (pp = h->h_aliases; *pp != NULL; pp++)
            printf(" %s", *pp);
        printf("\n");

        printf("        address type:   %s\n",
                (h->h_addrtype == AF_INET) ? "AF_INET" :
                (h->h_addrtype == AF_INET6) ? "AF_INET6" : "???");

        if (h->h_addrtype == AF_INET || h->h_addrtype == AF_INET6) {
            printf("        address(es):   ");
            for (pp = h->h_addr_list; *pp != NULL; pp++)
                printf(" %s", inet_ntop(h->h_addrtype, *pp,
                                        str, INET6_ADDRSTRLEN));
            printf("\n");
        }
    }

    exit(EXIT_SUCCESS);
}
     `sockets/t_gethostbyname.c`
```

### *getservbyname()* 和 *getservbyport()* 函数

*getservbyname()* 和 *getservbyport()* 函数从 `/etc/services` 文件中检索记录（`/etc/services` 文件）。这些函数如今已经被 *getaddrinfo()* 和 *getnameinfo()* 所淘汰。

```
#include <netdb.h>

struct servent *`getservbyname`(const char **name*, const char **proto*);
struct servent *`getservbyport`(int *port*, const char **proto*);
```

### 注意

两者都在成功时返回指向（静态分配的）*servent* 结构的指针，未找到或发生错误时返回 `NULL`。

*getservbyname()* 函数查找服务名称（或其别名）与 *name* 匹配且协议与 *proto* 匹配的记录。*proto* 参数是一个字符串，如 *tcp* 或 *udp*，也可以是 `NULL`。如果 *proto* 被指定为 `NULL`，则返回任何服务名称与 *name* 匹配的记录。（通常这是足够的，因为在 `/etc/services` 文件中，如果存在相同名称的 UDP 和 TCP 记录，它们通常具有相同的端口号。）如果找到匹配的记录，*getservbyname()* 将返回指向以下类型的静态分配结构的指针：

```
struct servent {
    char  *s_name;          /* Official service name */
    char **s_aliases;       /* Pointers to aliases (NULL-terminated) */
    int    s_port;          /* Port number (in network byte order) */
    char  *s_proto;         /* Protocol */
};
```

通常，我们仅调用 *getservbyname()* 来获取端口号，该端口号将返回在 *s_port* 字段中。

*getservbyport()* 函数执行 *getservbyname()* 的反向操作。它返回一个包含来自 `/etc/services` 记录的信息的 *servent* 记录，该记录的端口号与 *port* 匹配，协议与 *proto* 匹配。同样，我们可以将 *proto* 指定为 `NULL`，在这种情况下，调用将返回任何端口号与 *port* 中指定的端口号匹配的记录。（在上述少数情况下，可能不会返回期望的结果，因为相同的端口号在 UDP 和 TCP 中可能对应不同的服务名称。）

### 注意

本书源代码分发中的 `files/t_getservbyname.c` 文件中提供了一个使用 *getservbyname()* 函数的示例。

## UNIX 与互联网域套接字

在编写通过网络通信的应用程序时，我们必须使用互联网域套接字。然而，当使用套接字在同一系统上的应用程序之间进行通信时，我们可以选择使用互联网域套接字或 UNIX 域套接字。在这种情况下，我们应该使用哪个域，为什么？

仅使用互联网域套接字编写应用程序通常是最简单的方法，因为它可以在单个主机和网络之间工作。然而，我们可能会选择使用 UNIX 域套接字，原因有以下几点：

+   在某些实现中，UNIX 域套接字比互联网域套接字更快。

+   我们可以使用目录（以及在 Linux 上的文件）权限来控制对 UNIX 域套接字的访问，以便只有具有指定用户或组 ID 的应用程序才能连接到监听流套接字或向数据报套接字发送数据报。这提供了一种简单的客户端身份验证方法。对于互联网域套接字，如果我们希望验证客户端，则需要做更多的工作。

+   使用 UNIX 域套接字，我们可以传递打开的文件描述符和发送者凭据，如传递文件描述符中所总结的。

## 进一步的信息

关于 TCP/IP 和套接字 API，有大量的印刷和在线资源：

+   网络编程的关键书籍是[Stevens 等人, 2004]。[Snader, 2000] 提供了一些关于套接字编程的有用指南。

+   [Stevens, 1994] 和 [Wright & Stevens, 1995] 详细描述了 TCP/IP。[Comer, 2000]、[Comer & Stevens, 1999]、[Comer & Stevens, 2000]、[Kozierok, 2005] 和 [Goralksi, 2009] 也提供了对相同内容的很好的介绍。

+   [Tanenbaum, 2002] 提供了计算机网络的一般背景。

+   [Herbert, 2004] 描述了 Linux 2.6 TCP/IP 堆栈的细节。

+   GNU C 库手册（在线地址：[`www.gnu.org/`](http://www.gnu.org/)）详细讨论了套接字 API。

+   IBM 红皮书《*TCP/IP 教程与技术概述*》提供了关于网络概念、TCP/IP 内部结构、套接字 API 及相关主题的详细内容。它可以从 [`www.redbooks.ibm.com/`](http://www.redbooks.ibm.com/) 免费下载。

+   [Gont, 2008] 和 [Gont, 2009b] 提供了关于 IPv4 和 TCP 的安全评估。

+   Usenet 新 sgroup *comp.protocols.tcp-ip* 专门讨论与 TCP/IP 网络协议相关的问题。

+   [Sarolahti & Kuznetsov, 2002] 描述了 Linux TCP 实现中的拥塞控制和其他细节。

+   有关 Linux 特定信息，请参阅以下手册页面：*socket(7)*、*ip(7)*、*raw(7)*、*tcp(7)*、*udp(7)* 和 *packet(7)*。

+   另请参见第 58.7 节中的 RFC 列表。

## 总结

Internet 域套接字允许不同主机上的应用程序通过 TCP/IP 网络进行通信。一个 Internet 域套接字地址由 IP 地址和端口号组成。在 IPv4 中，IP 地址是一个 32 位的数字；在 IPv6 中，它是一个 128 位的数字。Internet 域数据报套接字通过 UDP 操作，提供无连接、不可靠的面向消息的通信；Internet 域流套接字通过 TCP 操作，提供可靠的、双向的字节流通信通道，供两个连接的应用程序之间使用。

不同的计算机架构使用不同的约定来表示数据类型。例如，整数可能以小端或大端形式存储，不同的计算机可能使用不同的字节数来表示数字类型，如 *int* 或 *long*。这些差异意味着在通过网络传输数据时，我们需要采用一些与架构无关的表示方法。我们提到，已经存在多种标准化的解决方案来应对这个问题，并且描述了一种许多应用程序使用的简单解决方法：将所有传输的数据以文本形式编码，并使用指定字符（通常是换行符）分隔字段。

我们研究了一些可以用来在 IP 地址的（数字）字符串表示（IPv4 的点分十进制和 IPv6 的十六进制字符串）与其二进制等价物之间转换的函数。然而，通常更推荐使用主机名和服务名，而不是数字，因为名字更容易记住，并且即使对应的数字发生变化，依然可以使用。我们研究了各种将主机名和服务名转换为其数字等价物及反向转换的函数。用于将主机名和服务名转换为套接字地址的现代函数是*getaddrinfo()*，但在现有代码中常见的是历史函数*gethostbyname()*和*getservbyname()*。

对主机名转换的考虑引发了我们对 DNS 的讨论，DNS 实现了一个分布式数据库，用于提供层级目录服务。DNS 的优势在于数据库管理并不集中化。相反，本地区域管理员更新他们负责的数据库层级部分的更改，DNS 服务器之间相互通信以解析主机名。

## 练习

1.  当读取大量数据时，如示例 59-1 中显示的*readLine()*函数效率较低，因为每读取一个字符都需要系统调用。一个更高效的接口是将一块字符读取到缓冲区中，然后从缓冲区中一次提取一行数据。这样的接口可能由两个函数组成。第一个函数，可能被称为*readLineBufInit(fd*, *&rlbuf)*，初始化由*rlbuf*指向的记账数据结构。这个结构包括一个数据缓冲区的空间、缓冲区的大小和指向该缓冲区中下一个“未读”字符的指针。它还包括一个参数*fd*给定的文件描述符的副本。第二个函数*readLineBuf(&rlbuf)*从与*rlbuf*相关联的缓冲区返回下一行数据。如果需要，这个函数会从保存在*rlbuf*中的文件描述符中读取更多数据块。实现这两个函数。修改示例 59-6 (`is_seqnum_sv.c`) 和示例 59-7 (`is_seqnum_cl.c`)中的程序，使用这两个函数。

1.  修改示例 59-6（`is_seqnum_sv.c`）和示例 59-7（`is_seqnum_cl.c`）中的程序，以使用示例 59-9（`inet_sockets.c`）中提供的*inetListen()*和*inetConnect()*函数。

1.  编写一个 UNIX 域套接字库，API 类似于互联网域套接字库中展示的 Internet 域套接字库。重写示例 57-3 中的程序（`us_xfr_sv.c`，见 UNIX 域中的流套接字）和示例 57-4 中的程序（`us_xfr_cl.c`，见 UNIX 域中的流套接字），以使用该库。

1.  编写一个网络服务器，用于存储名称-值对。该服务器应允许客户端添加、删除、修改和检索名称。编写一个或多个客户端程序来测试服务器。可选地，实现某种安全机制，允许仅允许创建名称的客户端删除该名称或修改与其关联的值。

1.  假设我们创建两个 Internet 域数据报套接字，绑定到特定地址，并将第一个套接字连接到第二个套接字。如果我们创建第三个数据报套接字，并尝试通过该套接字向第一个套接字发送(*sendto()*)一个数据报，结果会怎样？编写程序来确定答案。
