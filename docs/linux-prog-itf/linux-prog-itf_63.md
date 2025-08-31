## 第六十三章 替代 I/O 模型

本章讨论了三种替代传统文件 I/O 模型的方法，这些方法在本书中展示的大多数程序中都有应用：

+   I/O 复用（*select()* 和 *poll()* 系统调用）；

+   信号驱动 I/O；

+   特定于 Linux 的*epoll* API。

## 概述

到目前为止，本书中展示的大多数程序都采用了一个 I/O 模型，其中一个进程一次只能对一个文件描述符进行 I/O 操作，每个 I/O 系统调用会阻塞，直到数据被传输。例如，在从管道中读取数据时，如果管道中没有数据，*read()*调用通常会阻塞；而*write()*调用则会在管道中没有足够空间存储待写入的数据时阻塞。在执行其他各种类型的文件（包括 FIFO 和套接字）的 I/O 操作时，也会发生类似的行为。

### 注解

磁盘文件是一个特殊的情况。如第十三章所述，内核使用缓冲区缓存来加速磁盘 I/O 请求。因此，向磁盘写入*write()*调用会在请求的数据传输到内核缓冲区缓存后立即返回，而不是等到数据写入磁盘（除非在打开文件时指定了`O_SYNC`标志）。相应地，*read()*会将数据从缓冲区缓存传输到用户缓冲区，如果所需数据不在缓冲区缓存中，内核会将进程挂起，直到完成磁盘读取。

传统的阻塞 I/O 模型对于许多应用程序来说是足够的，但并非所有情况都适用。特别是一些应用程序需要能够做到以下一项或两项：

+   检查文件描述符上是否可以进行 I/O 操作，如果不行，则不阻塞。

+   监控多个文件描述符，看看是否可以对其中任何一个进行 I/O 操作。

我们已经遇到过两种可以部分解决这些需求的技术：非阻塞 I/O 和使用多个进程或线程。

我们在非阻塞 I/O 和非阻塞 I/O 中详细描述了非阻塞 I/O。如果我们通过启用`O_NONBLOCK`打开文件状态标志将文件描述符设置为非阻塞模式，那么一个无法立即完成的 I/O 系统调用会返回错误，而不是阻塞。非阻塞 I/O 可以用于管道、FIFO、套接字、终端、伪终端和其他一些类型的设备。

非阻塞 I/O 允许我们定期检查（“轮询”）某个文件描述符是否可以执行 I/O 操作。例如，我们可以将一个输入文件描述符设置为非阻塞，然后定期进行非阻塞读取。如果我们需要监视多个文件描述符，那么可以将它们都设置为非阻塞，并依次轮询每个文件描述符。然而，以这种方式进行轮询通常是不可取的。如果轮询的频率太低，那么应用程序响应 I/O 事件的延迟可能会过长；另一方面，紧密的轮询循环会浪费 CPU 时间。

### 注意

在本章中，我们使用 *poll* 一词有两种不同的含义。其中一种是作为 I/O 多路复用系统调用的名称，即 *poll()*。另一种意思是“执行一个非阻塞的文件描述符状态检查”。

如果我们不希望进程在执行文件描述符的 I/O 操作时被阻塞，我们可以创建一个新的进程来执行 I/O 操作。父进程可以继续执行其他任务，而子进程则阻塞直到 I/O 操作完成。如果我们需要处理多个文件描述符的 I/O 操作，我们可以为每个描述符创建一个子进程。此方法的问题是代价高昂且复杂。创建和维护进程会对系统造成负担，通常，子进程需要使用某种形式的进程间通信（IPC）来通知父进程 I/O 操作的状态。

使用多个线程而不是进程对资源的需求较低，但线程之间可能仍然需要相互传递有关 I/O 操作状态的信息，且编程可能会很复杂，尤其是在使用线程池来最小化处理大量同时连接的线程数量时。（线程特别有用的一个场景是，当应用程序需要调用一个执行阻塞 I/O 的第三方库时。在这种情况下，应用程序可以通过在单独的线程中进行库调用来避免阻塞。）

由于非阻塞 I/O 和使用多个线程或进程的局限性，以下替代方案之一通常是更为可取的：

+   *I/O 多路复用* 允许进程同时监控多个文件描述符，以确定是否可以在其中任何一个上执行 I/O 操作。*select()* 和 *poll()* 系统调用执行 I/O 多路复用。

+   *信号驱动的 I/O* 是一种技术，其中进程请求内核在输入可用或可以在指定的文件描述符上写入数据时发送信号给它。进程可以继续执行其他操作，并在通过接收到信号时被通知 I/O 操作已可进行。在监视大量文件描述符时，信号驱动的 I/O 比 *select()* 和 *poll()* 提供显著更好的性能。

+   *epoll* API 是一个 Linux 特有的功能，首次出现在 Linux 2.6 中。与 I/O 多路复用 API 类似，*epoll* API 允许进程监视多个文件描述符，查看是否可以在其中任何一个上执行 I/O。与信号驱动 I/O 类似，*epoll* API 在监视大量文件描述符时提供了更好的性能。

### 注意

在本章的剩余部分，我们将通常从进程的角度框定上述技术的讨论。然而，这些技术也可以应用于多线程应用程序。

实际上，I/O 多路复用、信号驱动 I/O 和 *epoll* 都是实现相同结果的方法——同时监视一个或多个文件描述符，以查看它们是否*准备好*执行 I/O（准确地说，是查看是否可以在不阻塞的情况下执行 I/O 系统调用）。文件描述符进入就绪状态的转换是由某种类型的 I/O *事件*触发的，例如输入的到来、套接字连接的完成，或者在 TCP 将排队的数据传输到套接字对端后，先前满的套接字发送缓冲区中空间的可用性。监视多个文件描述符在某些应用中非常有用，例如网络服务器必须同时监视多个客户端套接字，或必须同时监视来自终端和管道或套接字的输入的应用。

请注意，这些技术都不执行 I/O 操作。它们只是告诉我们某个文件描述符已经就绪。然后必须使用其他系统调用来实际执行 I/O。

### 注意

本章没有描述的一种 I/O 模型是 POSIX 异步 I/O（AIO）。POSIX AIO 允许进程将 I/O 操作排队到一个文件，然后在操作完成时得到通知。POSIX AIO 的优势在于初始的 I/O 调用会立即返回，因此进程不会被挂起等待数据传输到内核或操作完成。这使得进程能够在执行 I/O 的同时执行其他任务（这可能包括排队进一步的 I/O 请求）。对于某些类型的应用，POSIX AIO 可以提供有用的性能优势。目前，Linux 在 *glibc* 中提供了基于线程的 POSIX AIO 实现。在写作时，正在进行中的工作是提供内核中的 POSIX AIO 实现，这应该能提供更好的扩展性能。POSIX AIO 在 [Gallmeister, 1995]、[Robbins & Robbins, 2003] 和 *aio(7)* 手册页面中有描述。

#### 选择哪种技术？

在本章中，我们将考虑选择某种技术而非其他技术的原因。与此同时，我们总结了几个要点：

+   *select()* 和 *poll()* 系统调用是长期存在的接口，在 UNIX 系统上已存在多年。与其他技术相比，它们的主要优点是可移植性。它们的主要缺点是，当监控大量（数百或数千）文件描述符时，扩展性较差。

+   *epoll* API 的主要优点是它允许应用程序高效地监控大量文件描述符。它的主要缺点是它是一个 Linux 特有的 API。

    ### 注意

    其他一些 UNIX 实现提供了类似于 *epoll* 的（非标准）机制。例如，Solaris 提供了特殊的 `/dev/poll` 文件（详见 Solaris *poll(7d)* 手册页），一些 BSD 系统提供了 *kqueue* API（该 API 提供了比 *epoll* 更通用的监控功能）。[Stevens 等人，2004] 简要描述了这两种机制；关于 *kqueue* 的更详细讨论可以在 [Lemon, 2001] 中找到。

+   与 *epoll* 类似，信号驱动 I/O 也允许应用程序高效地监控大量文件描述符。然而，*epoll* 提供了比信号驱动 I/O 更多的优点：

    +   我们避免了处理信号的复杂性。

    +   我们可以指定希望执行的监控类型（例如，准备好读取或准备好写入）。

    +   我们可以选择级别触发或边缘触发通知（详见哪种技术？）。

    此外，充分利用信号驱动 I/O 需要使用不可移植的 Linux 特有功能，如果我们这样做，信号驱动 I/O 的可移植性与 *epoll* 一样差。

因为 *select()* 和 *poll()* 更具可移植性，而信号驱动 I/O 和 *epoll* 提供了更好的性能，对于某些应用程序，编写一个抽象的软件层来监控文件描述符事件是值得的。有了这样的层，便携式程序可以在提供 *epoll*（或类似 API）的系统上使用它，并在其他系统上回退到使用 *select()* 或 *poll()*。

### 注意

*libevent* 库是一个软件层，提供了一个监控文件描述符事件的抽象。它已移植到多个 UNIX 系统。作为其底层机制，*libevent* 可以（透明地）使用本章中描述的任何技术：*select()*、*poll()*、信号驱动 I/O 或 *epoll*，以及 Solaris 特有的 `/dev/poll` 接口或 BSD 的 *kqueue* 接口。（因此，*libevent* 也作为如何使用这些技术的示例。）由 Niels Provos 编写，*libevent* 可在 [`monkey.org/~provos/libevent/`](http://monkey.org/~provos/libevent/) 获得。

### 级别触发与边缘触发通知

在详细讨论各种替代 I/O 机制之前，我们需要区分两种文件描述符就绪通知模型：

+   *等级触发通知*：如果能够执行 I/O 系统调用而不阻塞，则认为文件描述符已准备好。

+   *边缘触发通知*：如果自上次监控以来文件描述符上发生了 I/O 活动（例如，有新的输入），则会提供通知。

表 63-1 总结了 I/O 多路复用、信号驱动 I/O 和 *epoll* 所采用的通知模型。与其他两种 I/O 模型不同，*epoll* API 可以同时使用等级触发通知（默认）和边缘触发通知。

表 63-1. 使用等级触发和边缘触发通知模型

| I/O 模型 | 等级触发？ | 边缘触发？ |
| --- | --- | --- |
| *select()*, *poll()* | • |   |
| 信号驱动的 I/O |   | • |
| *epoll* | • | • |

这两种通知模型之间的差异将在本章过程中变得更加清晰。现在，我们先描述通知模型的选择如何影响程序设计。

当我们使用等级触发通知时，可以随时检查文件描述符的准备情况。这意味着，当我们确认文件描述符已准备好（例如，有可用输入）时，可以对该描述符执行某些 I/O 操作，然后重复监控操作，检查描述符是否仍然准备好（例如，仍然有更多输入可用），此时我们可以执行更多 I/O，依此类推。换句话说，由于等级触发模型允许我们随时重复进行 I/O 监控操作，因此每次收到文件描述符准备就绪的通知时，不需要尽可能多地执行 I/O 操作（例如，读取尽可能多的字节），甚至可以不执行任何 I/O 操作。

相比之下，当我们使用边缘触发通知时，只有在发生 I/O 事件时才会收到通知。在另一个 I/O 事件发生之前，我们不会收到进一步的通知。此外，当某个文件描述符的 I/O 事件被通知时，我们通常不知道可能进行多少 I/O 操作（例如，有多少字节可以读取）。因此，使用边缘触发通知的程序通常遵循以下规则进行设计：

+   在收到 I/O 事件通知后，程序应该在某个时刻对相应的文件描述符执行尽可能多的 I/O 操作（例如，尽可能多地读取字节）。如果程序未能做到这一点，那么它可能会错过执行某些 I/O 的机会，因为直到发生另一个 I/O 事件，它才会意识到需要对文件描述符进行操作。这可能导致程序中的虚假数据丢失或阻塞。我们说“在某个时刻”，因为有时在确认文件描述符已准备好后，立即执行所有 I/O 操作并不一定是理想的。问题在于，如果我们对一个文件描述符执行大量 I/O 操作，可能会使其他文件描述符得不到足够的关注。我们在描述 *epoll* 的边缘触发通知模型时，会更详细地考虑这一点，见 Edge-Triggered Notification。

+   如果程序使用循环来尽可能多地在文件描述符上执行 I/O 操作，而描述符被标记为阻塞，那么最终在没有更多 I/O 可执行时，I/O 系统调用将会阻塞。出于这个原因，每个监视的文件描述符通常会被设置为非阻塞模式，在收到 I/O 事件通知后，会重复执行 I/O 操作，直到相关的系统调用（例如，*read()* 或 *write()*）由于错误 `EAGAIN` 或 `EWOULDBLOCK` 而失败。

### 使用非阻塞 I/O 与其他 I/O 模型

非阻塞 I/O（`O_NONBLOCK` 标志）通常与本章描述的 I/O 模型一起使用。以下是一些此方式可能有用的示例：

+   如上一节所述，非阻塞 I/O 通常与提供边缘触发 I/O 事件通知的 I/O 模型一起使用。

+   如果多个进程（或线程）在同一打开的文件描述符上执行 I/O 操作，那么从某个特定进程的角度来看，文件描述符的就绪状态可能会在描述符被通知为就绪和随后的 I/O 调用之间发生变化。因此，一个阻塞的 I/O 调用可能会阻塞，进而阻止该进程监视其他文件描述符。（这种情况可能发生在我们本章描述的所有 I/O 模型中，无论它们是否使用了水平触发或边缘触发通知。）

+   即使在像 *select()* 或 *poll()* 这样的水平触发 API 告知我们流套接字的文件描述符已准备好写入后，如果我们在单个 *write()* 或 *send()* 中写入足够大的数据块，那么调用仍然会阻塞。

+   在少数情况下，像 *select()* 和 *poll()* 这样的水平触发 API 可能会返回虚假的就绪通知——它们可能错误地告知我们某个文件描述符已经准备就绪。这可能是由内核漏洞引起的，或者是某种不常见情境下的预期行为。

### 注意

[Stevens 等人，2004 年] 的第 16.6 节描述了在 BSD 系统上监听套接字出现虚假就绪通知的一个例子。如果客户端连接到服务器的监听套接字并且随后重置连接，服务器在这两个事件之间执行的 *select()* 会将监听套接字标记为可读，但在客户端重置之后执行的 *accept()* 将会阻塞。

## I/O 多路复用

I/O 多路复用允许我们同时监视多个文件描述符，以查看是否可以在其中任何一个上进行 I/O。我们可以通过两种系统调用来执行 I/O 多路复用，它们具有基本相同的功能。这两个系统调用中的第一个是 *select()*，它与 BSD 中的套接字 API 一起出现。历史上，这是这两个系统调用中使用更广泛的一个。另一个系统调用是 *poll()*，它出现在 System V 中。如今，*select()* 和 *poll()* 都是 SUSv3 的要求。

我们可以使用 *select()* 和 *poll()* 来监视常规文件、终端、伪终端、管道、FIFOs、套接字以及一些字符设备类型的文件描述符。这两个系统调用允许进程要么无限期阻塞，等待文件描述符变为就绪，要么指定一个超时时间。

### *select()* 系统调用

*select()* 系统调用会阻塞，直到一组文件描述符中的一个或多个变为就绪。

```
#include <sys/time.h>         /* For portability */
#include <sys/select.h>

int `select`(int *nfds*, fd_set **readfds*, fd_set **writefds*, fd_set **exceptfds*,
           struct timeval **timeout*);
```

### 注意

返回已就绪文件描述符的数量，超时时返回 0，错误时返回 -1

*nfds*、*readfds*、*writefds* 和 *exceptfds* 参数指定了 *select()* 要监视的文件描述符。*timeout* 参数可用于设置 *select()* 阻塞的最大时间。我们将在下面详细描述这些参数。

### 注意

在上面展示的 *select()* 原型中，我们包含了 `<sys/time.h>` 头文件，因为这是 SUSv2 中指定的头文件，而且一些 UNIX 实现要求包含此头文件。（在 Linux 上，`<sys/time.h>` 头文件是存在的，包含它并不会有任何不良影响。）

#### 文件描述符集合

*readfds*、*writefds* 和 *exceptfds* 参数是指向 *文件描述符集合* 的指针，使用 *fd_set* 数据类型表示。这些参数的使用方式如下：

+   *readfds* 是用于测试是否可以进行输入的文件描述符集合；

+   *writefds* 是用于测试是否可以进行输出的文件描述符集合；

+   *exceptfds* 是用于测试文件描述符集合，以检查是否发生了异常情况。

*异常情况* 这个术语通常被误解为表示文件描述符上发生了某种错误情况。实际上并非如此。在 Linux 中（其他 UNIX 实现也类似），异常情况仅在两种情况下发生：

+   当伪终端从属设备连接到处于数据包模式下的主设备时，会发生状态变化（参见第 64.5 节）。

+   带外数据通过流套接字接收（带外数据）。

通常，*fd_set*数据类型实现为位掩码。然而，我们无需了解细节，因为所有文件描述符集的操作都是通过四个宏完成的：`FD_ZERO()`、`FD_SET()`、`FD_CLR()`和`FD_ISSET()`。

```
#include <sys/select.h>

void `FD_ZERO`(fd_set **fdset*);
void `FD_SET`(int *fd*, fd_set **fdset*);
void `FD_CLR`(int *fd*, fd_set **fdset*);

int `FD_ISSET`(int *fd*, fd_set **fdset*);
```

### 注意

如果*fd*在*fdset*中，返回 true（1），否则返回 false（0）。

这些宏的操作方式如下：

+   `FD_ZERO()`将由*fdset*指向的集合初始化为空。

+   `FD_SET()`将文件描述符*fd*添加到由*fdset*指向的集合中。

+   `FD_CLR()`从由*fdset*指向的集合中移除文件描述符*fd*。

+   `FD_ISSET()`返回 true，如果文件描述符*fd*是由*fdset*指向的集合的成员。

文件描述符集有一个最大大小，由常量`FD_SETSIZE`定义。在 Linux 中，这个常量的值为 1024。（其他 UNIX 实现也有类似的值作为此限制。）

### 注意

尽管`FD_*`宏作用于用户空间数据结构，并且*select()*的内核实现可以处理更大尺寸的描述符集，*glibc*并没有提供一种简单的方式来修改`FD_SETSIZE`的定义。如果我们想要改变这个限制，就必须在*glibc*头文件中修改定义。然而，正如本章后面所描述的那样，如果我们需要监视大量描述符，那么使用*epoll*可能比使用*select()*更合适。

*readfds*、*writefds*和*exceptfds*参数都是值结果类型。在调用*select()*之前，这些参数指向的*fd_set*结构必须先初始化（使用`FD_ZERO()`和`FD_SET()`），以包含感兴趣的文件描述符集合。*select()*调用会修改这些结构，使得它们在返回时包含就绪的文件描述符集合。（由于这些结构会被调用修改，我们必须确保在循环中重复调用*select()*时重新初始化它们。）然后，可以使用`FD_ISSET()`检查这些结构。

如果我们对某类事件不感兴趣，那么可以将相应的*fd_set*参数指定为`NULL`。关于每种事件类型的具体含义，请参阅文件描述符何时准备好？。

*nfds*参数必须设置为比所有三个文件描述符集中的最大文件描述符号大 1。这个参数使得*select()*更加高效，因为内核可以知道不需要检查比这个值更高的文件描述符号是否是每个文件描述符集的一部分。

#### *timeout*参数

*timeout*参数控制*select()*的阻塞行为。它可以指定为`NULL`，在这种情况下，*select()*会无限期地阻塞，或者指定为指向*timeval*结构的指针：

```
struct timeval {
    time_t      tv_sec;         /* Seconds */
    suseconds_t tv_usec;        /* Microseconds (long int) */
};
```

如果 *timeout* 的两个字段都为 0，则 *select()* 不会阻塞；它会立即轮询指定的文件描述符，查看哪些已准备好，并立即返回。否则，*timeout* 指定了 *select()* 等待的最大时间。

尽管 *timeval* 结构提供微秒级精度，但调用的准确性受限于软件时钟的粒度（第 10.6 节）。SUSv3 规定，如果超时时间不是该粒度的精确倍数，则超时会向上舍入。

### 注释

SUSv3 要求最大允许的超时间隔至少为 31 天。大多数 UNIX 实现允许更高的限制。由于 Linux/x86-32 使用 32 位整数表示 *time_t* 类型，最大限制为多年。

当 *timeout* 为 `NULL`，或指向包含非零字段的结构时，*select()* 会阻塞，直到以下情况之一发生：

+   *readfds*、*writefds* 或 *exceptfds* 中指定的至少一个文件描述符变为就绪；

+   调用被信号处理程序中断；或

+   *timeout* 指定的时间已过去。

### 注释

在没有子秒精度的睡眠调用的旧版 UNIX 实现中（例如，*nanosleep()*），通过将 *nfds* 设置为 0，*readfds*、*writefds* 和 *exceptfds* 设置为 `NULL`，以及在 *timeout* 中指定所需的睡眠间隔，*select()* 被用来模拟这一功能。

在 Linux 上，如果 *select()* 返回，因为一个或多个文件描述符已变为就绪，并且 *timeout* 非 `NULL`，则 *select()* 会更新 *timeout* 所指向的结构体，以指示如果不发生超时，剩余的时间。然而，这种行为是特定实现的。SUSv3 也允许实现可能保持 *timeout* 所指向的结构体不变，且大多数其他 UNIX 实现**不会**修改该结构体。在循环中使用 *select()* 的可移植应用程序应始终确保在每次调用 *select()* 之前初始化 *timeout* 所指向的结构体，并在调用后忽略结构体中返回的信息。

SUSv3 规定，*timeout* 所指向的结构体只能在 *select()* 成功返回后进行修改。然而，在 Linux 上，如果 *select()* 被信号处理程序中断（导致其因错误 `EINTR` 失败），则该结构体会被修改，以指示直到超时发生时剩余的时间（即，像成功返回一样）。

### 注释

如果我们使用 Linux 特有的 *personality()* 系统调用设置包含 `STICKY_TIMEOUTS` 个性位的个性，则 *select()* 不会修改 *timeout* 所指向的结构体。

#### *select()* 的返回值

作为其函数结果，*select()* 返回以下之一：

+   返回值为 -1 表示发生了错误。可能的错误包括 `EBADF` 和 `EINTR`。`EBADF` 表示 `readfds`、*writefds* 或 *exceptfds* 中的一个文件描述符无效（例如，当前未打开）。`EINTR` 表示调用被信号处理程序中断。（如第 21.5 节所述，如果被信号处理程序中断，*select()* 永远不会自动重新启动。）

+   返回值为 0 表示在任何文件描述符变为准备好之前调用已经超时。在这种情况下，每个返回的文件描述符集将为空。

+   正返回值表示一个或多个文件描述符已经准备好。返回值是已准备好的描述符数量。在这种情况下，必须检查每个返回的文件描述符集（使用`FD_ISSET()`）以找出发生了哪些 I/O 事件。如果相同的文件描述符在 *readfds*、*writefds* 和 *exceptfds* 中被指定多次，并且它准备好了多个事件，则会被多次计数。换句话说，*select()* 返回的是在所有三个返回的集合中标记为准备好的文件描述符的总数。

#### 示例程序

示例 63-1 监控多个文件描述符") 中的程序演示了如何使用 *select()*。通过命令行参数，我们可以指定 *超时* 和我们希望监控的文件描述符。第一个命令行参数指定 *select()* 的 *超时*（单位为秒）。如果在此处指定了一个短横线（-），则表示 *select()* 被调用时的超时值为 `NULL`，即无限期阻塞。其余命令行参数指定了要监控的文件描述符的编号，后面跟着表示要检查的操作的字母。我们可以在这里指定的字母是 *r*（准备好读取）和 *w*（准备好写入）。

示例 63-1. 使用 *select()* 监控多个文件描述符

```
`altio/t_select.c`
#include <sys/time.h>
#include <sys/select.h>
#include "tlpi_hdr.h"

static void
usageError(const char *progName)
{
    fprintf(stderr, "Usage: %s {timeout|-} fd-num[rw]...\n", progName);
    fprintf(stderr, "    - means infinite timeout; \n");
    fprintf(stderr, "    r = monitor for read\n");
    fprintf(stderr, "    w = monitor for write\n\n");
    fprintf(stderr, "    e.g.: %s - 0rw 1w\n", progName);
    exit(EXIT_FAILURE);
}
int
main(int argc, char *argv[])
{
    fd_set readfds, writefds;
    int ready, nfds, fd, numRead, j;
    struct timeval timeout;
    struct timeval *pto;
    char buf[10];                       /* Large enough to hold "rw\0" */

    if (argc < 2 || strcmp(argv[1], "--help") == 0)
        usageError(argv[0]);

    /* Timeout for select() is specified in argv[1] */

    if (strcmp(argv[1], "-") == 0) {
        pto = NULL;                     /* Infinite timeout */
    } else {
        pto = &timeout;
        timeout.tv_sec = getLong(argv[1], 0, "timeout");
        timeout.tv_usec = 0;            /* No microseconds */
    }

    /* Process remaining arguments to build file descriptor sets */

    nfds = 0;
    FD_ZERO(&readfds);
    FD_ZERO(&writefds);

    for (j = 2; j < argc; j++) {
        numRead = sscanf(argv[j], "%d%2[rw]", &fd, buf);
        if (numRead != 2)
            usageError(argv[0]);
        if (fd >= FD_SETSIZE)
            cmdLineErr("file descriptor exceeds limit (%d)\n", FD_SETSIZE);

        if (fd >= nfds)
            nfds = fd + 1;              /* Record maximum fd + 1 */
        if (strchr(buf, 'r') != NULL)
            FD_SET(fd, &readfds);
        if (strchr(buf, 'w') != NULL)
            FD_SET(fd, &writefds);
    }

    /* We've built all of the arguments; now call select() */

    ready = select(nfds, &readfds, &writefds, NULL, pto);
                                        /* Ignore exceptional events */
    if (ready == -1)
        errExit("select");

    /* Display results of select() */

    printf("ready = %d\n", ready);
for (fd = 0; fd < nfds; fd++)
        printf("%d: %s%s\n", fd, FD_ISSET(fd, &readfds) ? "r" : "",
                FD_ISSET(fd, &writefds) ? "w" : "");

    if (pto != NULL)
        printf("timeout after select(): %ld.%03ld\n",
               (long) timeout.tv_sec, (long) timeout.tv_usec / 10000);
    exit(EXIT_SUCCESS);
}
           `altio/t_select.c`
```

在以下的 shell 会话日志中，我们演示了在示例 63-1 监控多个文件描述符")中使用该程序。在第一个示例中，我们请求监控文件描述符 0 的输入，设置 10 秒的 *超时*：

```
$ `./t_select 10 0r`
*Press Enter, so that a line of input is available on file descriptor 0*
ready = 1
0: r
timeout after select(): 8.003
$                                         *Next shell prompt is displayed*
```

上述输出显示 *select()* 确定有一个文件描述符已准备好。这个文件描述符是 0，它已准备好读取。我们还可以看到 *超时* 被修改了。最后一行输出仅显示 shell 的 `$` 提示符，是因为 *t_select* 程序没有读取使文件描述符 0 准备好的换行符，因此该字符被 shell 读取，shell 通过打印另一个提示符来响应。

在下一个示例中，我们再次监控文件描述符 0 的输入，但这次设置的 *超时* 为 0 秒：

```
$ `./t_select 0 0r`
ready = 0
timeout after select(): 0.000
```

*select()* 调用立即返回，并且没有发现任何文件描述符已经准备好。

在下一个示例中，我们监控两个文件描述符：描述符 0，用于查看是否有输入，和描述符 1，用于查看是否可以输出。在这种情况下，我们将*timeout*指定为`NULL`（第一个命令行参数是一个破折号），表示无限：

```
$ `./t_select - 0r 1w`
ready = 1
0:
1: w
```

*select()*调用立即返回，通知我们文件描述符 1 上的输出是可能的。

### *poll()*系统调用

*poll()*系统调用执行的任务与*select()*类似。这两个系统调用之间的主要区别在于我们如何指定要监控的文件描述符。使用*select()*时，我们提供三个集合，每个集合标记以指示感兴趣的文件描述符。使用*poll()*时，我们提供一个文件描述符列表，每个文件描述符都标记了感兴趣的事件集。

```
#include <poll.h>

int `poll`(struct pollfd *fds*[], nfds_t *nfds*, int *timeout*);
```

### 注意

返回准备就绪的文件描述符数，超时时返回 0，出错时返回-1

*fds*参数和*pollfd*数组（*nfds*）指定了*poll()*要监控的文件描述符。*timeout*参数可用于设置*poll()*阻塞的最大时间。我们将在下面详细描述这些参数。

#### *pollfd*数组

*fds*参数列出了*poll()*要监控的文件描述符。这个参数是一个*pollfd*结构体数组，定义如下：

```
struct pollfd {
    int   fd;               /* File descriptor */
    short events;           /* Requested events bit mask */
    short revents;          /* Returned events bit mask */
};
```

*nfds*参数指定*fds*数组中的项数。用于类型化*nfds*参数的*nfds_t*数据类型是无符号整数类型。

*pollfd*结构的*events*和*revents*字段是位掩码。调用者初始化*events*以指定要监控的文件描述符*fd*的事件。在从*poll()*返回时，*revents*被设置以指示此文件描述符上实际发生的事件。

表 63-2 列出了可能出现在*events*和*revents*字段中的位。表中的第一组位（`POLLIN`，`POLLRDNORM`，`POLLRDBAND`，`POLLPRI`和`POLLRDHUP`）与输入事件有关。接下来的位（`POLLOUT`，`POLLWRNORM`和`POLLWRBAND`）与输出事件有关。第三组位（`POLLERR`，`POLLHUP`和`POLLNVAL`）在*revents*字段中设置，用于返回有关文件描述符的附加信息。如果在*events*字段中指定，这三个位将被忽略。最后一个位（`POLLMSG`）在 Linux 中不被*poll()*使用。

### 注意

在提供 STREAMS 设备的 UNIX 实现中，`POLLMSG`表示包含`SIGPOLL`信号的消息已到达流的头部。`POLLMSG`在 Linux 中未使用，因为 Linux 不实现 STREAMS。

表 63-2. *pollfd*结构的*events*和*revents*字段的位掩码值

| 位 | *events*中输入？ | 在*revents*中返回？ | 描述 |
| --- | --- | --- | --- |
| `POLLIN` | • | • | 可以读取除高优先级数据之外的数据 |
| `POLLRDNORM` | • | • | 等同于`POLLIN` |
| `POLLRDBAND` | • | • | 可以读取优先数据（在 Linux 上未使用） |
| `POLLPRI` | • | • | 可以读取高优先级数据 |
| `POLLRDHUP` | • | • | 对等方套接字关闭 |
| `POLLOUT` | • | • | 可以写入正常数据 |
| `POLLWRNORM` | • | • | 等同于`POLLOUT` |
| `POLLWRBAND` | • | • | 可以写入优先数据 |
| `POLLERR` |   | • | 发生了错误 |
| `POLLHUP` |   | • | 发生了挂起 |
| `POLLNVAL` |   | • | 文件描述符未打开 |
| `POLLMSG` |   |   | 在 Linux 上未使用（在 SUSv3 中未指定） |

如果我们对特定文件描述符的事件不感兴趣，可以将*events*指定为 0。此外，指定负值给*fd*字段（例如，如果非零则取其相反值）会导致对应的*events*字段被忽略，并且*revents*字段始终返回为 0。可以使用这些技术（或许是暂时的）禁用单个文件描述符的监视，而不需要重新构建整个*fds*列表。

请注意以下有关 Linux 实现*poll()*的进一步要点：

+   尽管被定义为单独的位，`POLLIN`和`POLLRDNORM`是同义的。

+   尽管被定义为单独的位，`POLLOUT`和`POLLWRNORM`是同义的。

+   `POLLRDBAND`通常未使用；也就是说，它在*events*字段中被忽略，并且在*revents*中未被设置。

    ### 注

    唯一设置`POLLRDBAND`的地方是在实现（已废弃的）DECnet 网络协议的代码中。

+   尽管在某些情况下为套接字设置了`POLLWRBAND`，但它并未传达有用信息。（没有任何情况会在未设置`POLLOUT`和`POLLWRNORM`的情况下设置`POLLWRBAND`。）

    ### 注

    `POLLRDBAND`和`POLLWRBAND`在提供 System V STREAMS 的实现中有意义（而 Linux 不支持）。在 STREAMS 下，一条消息可以被分配一个非零优先级，并且这些消息会按优先级递减的顺序排队到接收者，位于普通（优先级 0）消息之前的一个带内。

+   必须定义`_XOPEN_SOURCE`特性测试宏，才能从`<poll.h>`中获取常量`POLLRDNORM`、`POLLRDBAND`、`POLLWRNORM`和`POLLWRBAND`的定义。

+   `POLLRDHUP`是一个 Linux 特定的标志，自内核 2.6.17 版本起可用。为了从`<poll.h>`中获得该定义，必须定义`_GNU_SOURCE`特性测试宏。

+   如果在执行*poll()*调用时，指定的文件描述符已关闭，则返回`POLLNVAL`。

总结以上要点，真实有意义的*poll()*标志是`POLLIN`、`POLLOUT`、`POLLPRI`、`POLLRDHUP`、`POLLHUP`和`POLLERR`。我们将在何时文件描述符准备好？中更详细地讨论这些标志的含义。

#### *timeout*参数

*timeout*参数决定了*poll()*的阻塞行为，如下所示：

+   如果 *timeout* 等于 -1，则阻塞直到 *fds* 数组中的某个文件描述符准备好（根据相应的 *events* 字段定义）或捕获到信号。

+   如果 *timeout* 等于 0，则不阻塞——只进行检查，看看哪些文件描述符已经准备好。

+   如果 *timeout* 大于 0，则最多阻塞 *timeout* 毫秒，直到 *fds* 中的某个文件描述符准备好，或捕获到信号。

与 *select()* 一样，*timeout* 的准确性受到软件时钟粒度的限制（见第 10.6 节），并且 SUSv3 规定，如果 *timeout* 不是时钟粒度的精确倍数，*timeout* 总是会向上舍入。

#### *poll()*的返回值

作为其函数结果，*poll()*返回以下之一：

+   返回值为 -1 表示发生了错误。一种可能的错误是 `EINTR`，表示调用被信号处理程序中断。（如第 21.5 节所述，*poll()* 在被信号处理程序中断时从不自动重新启动。）

+   返回值为 0 表示调用在任何文件描述符准备好之前已超时。

+   正值返回值表示一个或多个文件描述符已准备好。返回的值是 *fds* 数组中具有非零 *revents* 字段的 *pollfd* 结构体的数量。

### 注意

注意 *select()* 和 *poll()* 返回值的稍微不同含义。*select()* 系统调用会在文件描述符出现在多个返回的文件描述符集中时，多次计数该文件描述符。*poll()* 系统调用返回准备好的文件描述符的数量，且即使相应的 *revents* 字段中设置了多个位，文件描述符也只计数一次。

#### 示例程序

示例 63-2 来监控多个文件描述符") 提供了一个关于 *poll()* 使用的简单演示。该程序创建了多个管道（每个管道使用一对连续的文件描述符），向随机选择的管道的写端写入字节，然后执行 *poll()* 以查看哪些管道有数据可供读取。

以下的 shell 会话展示了运行此程序时我们所看到的示例。程序的命令行参数指定了应该创建十个管道，并且向三个随机选择的管道写入数据。

```
$ `./poll_pipes 10 3`
Writing to fd:   4 (read fd:   3)
Writing to fd:  14 (read fd:  13)
Writing to fd:  14 (read fd:  13)
poll() returned: 2
Readable:   3
Readable:  13
```

从上面的输出中，我们可以看到 *poll()* 发现有两个管道有数据可以读取。

示例 63-2. 使用 *poll()* 来监控多个文件描述符

```
`altio/poll_pipes.c`
#include <time.h>
#include <poll.h>
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    int numPipes, j, ready, randPipe, numWrites;
    int (*pfds)[2];                     /* File descriptors for all pipes */
    struct pollfd *pollFd;

    if (argc < 2 || strcmp(argv[1], "--help") == 0)
        usageErr("%s num-pipes [num-writes]\n", argv[0]);

    /* Allocate the arrays that we use. The arrays are sized according
       to the number of pipes specified on command line */

    numPipes = getInt(argv[1], GN_GT_0, "num-pipes");

    pfds = calloc(numPipes, sizeof(int [2]));
    if (pfds == NULL)
        errExit("malloc");
    pollFd = calloc(numPipes, sizeof(struct pollfd));
    if (pollFd == NULL)
        errExit("malloc");

    /* Create the number of pipes specified on command line */

    for (j = 0; j < numPipes; j++)
        if (pipe(pfds[j]) == -1)
            errExit("pipe %d", j);

    /* Perform specified number of writes to random pipes */

    numWrites = (argc > 2) ? getInt(argv[2], GN_GT_0, "num-writes") : 1;

    srandom((int) time(NULL));
    for (j = 0; j < numWrites; j++) {
        randPipe = random() % numPipes;
        printf("Writing to fd: %3d (read fd: %3d)\n",
                pfds[randPipe][1], pfds[randPipe][0]);
        if (write(pfds[randPipe][1], "a", 1) == -1)
            errExit("write %d", pfds[randPipe][1]);
    }

    /* Build the file descriptor list to be supplied to poll(). This list
       is set to contain the file descriptors for the read ends of all of
       the pipes. */

    for (j = 0; j < numPipes; j++) {
        pollFd[j].fd = pfds[j][0];
        pollFd[j].events = POLLIN;
    }

    ready = poll(pollFd, numPipes, -1);         /* Nonblocking */
    if (ready == -1)
        errExit("poll");

    printf("poll() returned: %d\n", ready);

    /* Check which pipes have data available for reading */

    for (j = 0; j < numPipes; j++)
        if (pollFd[j].revents & POLLIN)
            printf("Readable: %d %3d\n", j, pollFd[j].fd);

    exit(EXIT_SUCCESS);
}
     `altio/poll_pipes.c`
```

### 文件描述符何时准备好？

正确使用 *select()* 和 *poll()* 需要了解文件描述符指示为就绪的条件。SUSv3 规定，如果对 I/O 函数的调用不会阻塞（*无论该函数是否实际传输数据*），则认为文件描述符（`O_NONBLOCK` 未设置）是就绪的。关键点已被斜体标出：*select()* 和 *poll()* 告诉我们 I/O 操作是否不会阻塞，而不是它是否会成功传输数据。从这个角度来看，我们来看看这些系统调用如何在不同类型的文件描述符上运行。我们将这些信息展示在包含两列的表格中：

+   *select()* 列指示文件描述符是否被标记为可读（`r`）、可写（`w`）或具有异常条件（`x`）。

+   *poll()* 列指示在 *revents* 字段中返回的位。 在这些表中，我们省略了 `POLLRDNORM`、`POLLWRNORM`、`POLLRDBAND` 和 `POLLWRBAND` 的提及。虽然在某些情况下（如果在 *events* 中指定了它们），这些标志可能会在 *revents* 中返回，但它们传达的信息并没有超出 `POLLIN`、`POLLOUT`、`POLLHUP` 和 `POLLERR` 提供的信息。

#### 常规文件

引用常规文件的文件描述符始终会被 *select()* 标记为可读和可写，并且在 *poll()* 中返回 `POLLIN` 和 `POLLOUT` 设置在 *revents* 中，原因如下：

+   *read()* 操作将始终立即返回数据、文件结束符或错误（例如，文件没有以可读方式打开）。

+   *write()* 操作将始终立即传输数据或因某些错误而失败。

### 注意

SUSv3 规定，*select()* 也应该将常规文件的描述符标记为具有异常条件（尽管这对常规文件没有明显意义）。只有一些实现会这么做；Linux 是其中之一，没有这样做的实现。

#### 终端和伪终端

表 63-3 和 poll() 对终端和伪终端的指示") 总结了 *select()* 和 *poll()* 对终端和伪终端的行为（第六十四章）。

当伪终端对的一半关闭时，*poll()* 返回的另一半对的 *revents* 设置取决于实现。在 Linux 中，至少设置 `POLLHUP` 标志。然而，其他实现返回不同的标志来表示此事件——例如，`POLLHUP`、`POLLERR` 或 `POLLIN`。此外，在某些实现中，设置的标志取决于是主设备还是从设备被监视。

表 63-3. *select()* 和 *poll()* 对终端和伪终端的指示

| 条件或事件 | *select()* | *poll()* |
| --- | --- | --- |
| 输入可用 | `r` | `POLLIN` |
| 输出可能 | `w` | `POLLOUT` |
| 伪终端对关闭后 | `rw` | 见正文 |
| 伪终端主设备在数据包模式下检测从设备状态变化 | `x` | `POLLPRI` |

#### 管道和 FIFO

表 63-4 和 poll() 在管道或 FIFO 的读端指示") 总结了管道或 FIFO 的读端的详细信息。*管道中有数据？* 列指示管道中是否至少有 1 个字节的数据可供读取。在此表中，我们假设在 *poll()* 的 *events* 字段中指定了 `POLLIN`。

在某些其他 UNIX 实现中，如果管道的写端关闭，则 *poll()* 返回时不会设置 `POLLHUP`，而是设置 `POLLIN` 位（因为 *read()* 会立即返回文件结束标志）。便携式应用程序应检查是否设置了这两个位，以确定 *read()* 是否会阻塞。

表 63-5 和 poll() 在管道或 FIFO 的写端指示") 总结了管道写端的详细信息。在此表中，我们假设在 *poll()* 的 *events* 字段中指定了 `POLLOUT`。*是否有足够空间写入 PIPE_BUF 字节？* 列指示管道是否有空间原子性地写入 `PIPE_BUF` 字节而不阻塞。这是 Linux 判断管道是否准备好写入的标准。一些其他 UNIX 实现使用相同的标准；而其他一些则认为只要管道能够写入一个字节，就认为管道是可写的。（在 Linux 2.6.10 及更早版本中，管道的容量等于 `PIPE_BUF`。这意味着，如果管道中有一个字节数据，管道就被认为是不可写的。）

在某些其他 UNIX 实现中，如果管道的读端关闭，则 *poll()* 返回时不会设置 `POLLERR`，而是设置 `POLLOUT` 位或 `POLLHUP` 位。便携式应用程序需要检查是否设置了这些位中的任何一位，以确定 *write()* 是否会阻塞。

表 63-4. *select()* 和 *poll()* 在管道或 FIFO 的读端指示

| 条件或事件 | *select()* | *poll()* |
| --- | --- | --- |
| 管道中有数据？ | 写端打开？ |
| --- | --- |
| no | no | `r` | `POLLHUP` |
| yes | yes | `r` | `POLLIN` |
| yes | no | `r` | `POLLIN &#124; POLLHUP` |

表 63-5. *select()* 和 *poll()* 在管道或 FIFO 的写端指示

| 条件或事件 | *select()* | *poll()* |
| --- | --- | --- |
| 是否有足够空间写入 `PIPE_BUF` 字节？ | 读端打开？ |
| --- | --- |
| no | no | `w` | `POLLERR` |
| yes | yes | `w` | `POLLOUT` |
| yes | no | `w` | `POLLOUT &#124; POLLERR` |

#### 套接字

表 63-6 和 poll()的套接字指示")总结了*select()*和*poll()*在套接字上的行为。对于*poll()*列，我们假设*events*指定为`(POLLIN | POLLOUT | POLLPRI)`。对于*select()*列，我们假设正在测试文件描述符以查看是否可以输入、是否可以输出或是否发生了异常情况（即，文件描述符在传递给*select()*的所有三个集合中都有指定）。此表仅涵盖常见情况，并非所有可能的场景。

### 注意

Linux 的*poll()*行为与 UNIX 域套接字的*close()*操作后的行为与表 63-6 和 poll()的套接字指示")中的行为不同。除了其他标志外，*poll()*还会在*revents*中返回`POLLHUP`。

表 63-6. *select()*和*poll()*在套接字上的指示

| 条件或事件 | *select()* | *poll()* |
| --- | --- | --- |
| 输入可用 | `r` | `POLLIN` |
| 输出可能 | `w` | `POLLOUT` |
| 在监听套接字上建立的传入连接 | `r` | `POLLIN` |
| 接收到带外数据（仅 TCP） | `x` | `POLLPRI` |
| 流套接字的对等端关闭了连接或执行了*shutdown(SHUT_WR)* | `rw` | `POLLIN &#124; POLLOUT &#124; POLLRDHUP` |

Linux 特有的`POLLRDHUP`标志（自 Linux 2.6.17 开始提供）需要进一步解释。这个标志——实际上是`EPOLLRDHUP`的形式——主要用于*epoll* API 的边缘触发模式（第 63.4 节）。当流套接字连接的远程端关闭了连接的写入端时，将返回此标志。使用此标志可以让使用*epoll*边缘触发接口的应用程序使用更简单的代码来识别远程关闭。（另一种方法是应用程序注意到`POLLIN`标志已设置，然后执行*read()*，通过返回 0 来指示远程关闭。）

### *select()*与*poll()*比较

在本节中，我们考虑*select()*和*poll()*之间的一些相似性和差异。

#### 实现细节

在 Linux 内核中，*select()*和*poll()*都使用相同的一组内核内部的*poll*例程。这些*poll*例程与*poll()*系统调用本身是不同的。每个例程返回有关单个文件描述符准备情况的信息。这些准备情况的信息以位掩码的形式返回，其值对应于*poll()*系统调用中*revents*字段返回的位（表 63-2）。*poll()*系统调用的实现涉及为每个文件描述符调用内核的*poll*例程，并将结果信息放入相应的*revents*字段中。

实现*select()*时，使用一组宏将内核*poll*例程返回的信息转换为*select()*返回的相应事件类型：

```
#define POLLIN_SET  (POLLRDNORM | POLLRDBAND | POLLIN | POLLHUP | POLLERR)
                                     /* Ready for reading */
#define POLLOUT_SET (POLLWRBAND | POLLWRNORM | POLLOUT | POLLERR)
                                     /* Ready for writing */
#define POLLEX_SET  (POLLPRI)        /* Exceptional condition */
```

这些宏定义揭示了*select()*和*poll()*返回的信息之间的语义对应关系。（如果我们查看文件描述符何时准备好？中的*select()*和*poll()*列，我们会发现每个系统调用提供的指示与上述宏一致。）我们需要补充的唯一信息是，如果被监控的文件描述符在调用时被关闭，*poll()*会在*revents*字段中返回`POLLNVAL`，而*select()*会返回-1 并将*errno*设置为`EBADF`。

#### API 差异

以下是*select()*和*poll()* API 的一些差异：

+   使用*fd_set*数据类型为*select()*能够监控的文件描述符范围设定了一个上限（`FD_SETSIZE`）。在 Linux 中，默认情况下，这个上限为 1024，修改该值需要重新编译应用程序。相比之下，*poll()*对可以监控的文件描述符范围没有内在的限制。

+   由于*select()*的*fd_set*参数是值结果类型，我们必须在循环中反复调用*select()*时重新初始化它们。通过使用独立的*events*（输入）和*revents*（输出）字段，*poll()*避免了这一要求。

+   *select()*提供的*timeout*精度（微秒）大于*poll()*提供的精度（毫秒）。（不过，这两个系统调用的超时准确性都受软件时钟粒度的限制。）

+   如果被监控的文件描述符之一被关闭，*poll()*会通过相应*revents*字段中的`POLLNVAL`位告诉我们具体是哪一个。相比之下，*select()*只是返回-1，并将*errno*设置为`EBADF`，我们需要通过在对描述符进行 I/O 系统调用时检查错误来确定哪个文件描述符已关闭。然而，这通常不是一个重要的区别，因为应用程序通常可以跟踪它已关闭的文件描述符。

#### 可移植性

历史上，*select()*比*poll()*更广泛可用。如今，两个接口都已被 SUSv3 标准化，并在当代实现中广泛可用。然而，正如在文件描述符何时准备好？中所提到的，*poll()*在不同实现中的行为有所不同。

#### 性能

如果以下任一条件为真，则*poll()*和*select()*的性能相似：

+   监控的文件描述符范围较小（即，最大文件描述符号较低）。

+   正在监视大量文件描述符，但它们是密集排列的（即，大部分或所有从 0 到某个限制的文件描述符都在被监视）。

然而，如果要监视的文件描述符集是稀疏的，即最大文件描述符编号 N 很大，但在范围 0 到*N*之间只有一个或少数几个描述符被监视，*select()*和*poll()*的性能差异可能会明显不同。在这种情况下，*poll()*可能比*select()*表现得更好。我们可以通过考虑这两个系统调用的参数来理解其原因。对于*select()*，我们传递一个或多个文件描述符集以及一个整数*nfds*，该整数比每个集合中要检查的最大文件描述符大 1。无论我们监视的是范围 0 到*(nfds - 1)*中的所有文件描述符，还是仅监视描述符*(nfds - 1)*，*nfds*参数的值都是相同的。在这两种情况下，内核必须检查每个集合中的*nfds*个元素，以确切地检查哪些文件描述符需要被监视。相比之下，使用*poll()*时，我们仅指定我们感兴趣的文件描述符，内核只检查这些描述符。

### 注意

在 Linux 2.4 中，*poll()*和*select()*在稀疏描述符集下的性能差异相当显著。Linux 2.6 中的一些优化大大缩小了性能差距。

我们在《*epoll* 与 I/O 多路复用性能对比》中进一步讨论了*select()*和*poll()*的性能，那里我们将这些系统调用的性能与*epoll*进行了对比。

### *select()*和*poll()*的问题

*select()*和*poll()*系统调用是便携式的、历史悠久且广泛使用的多文件描述符就绪监视方法。然而，在监视大量文件描述符时，这些 API 存在一些问题：

+   在每次调用*select()*或*poll()*时，内核必须检查所有指定的文件描述符，看看它们是否准备好。当监视大量位于密集范围内的文件描述符时，这个操作所需的时间远远超过了接下来的两个操作所需的时间。

+   在每次调用*select()*或*poll()*时，程序必须向内核传递一个数据结构，描述所有需要监视的文件描述符，检查完描述符后，内核会返回一个修改过的版本的这个数据结构给程序。（此外，对于*select()*，我们必须在每次调用之前初始化这个数据结构。）对于*poll()*，数据结构的大小会随着被监视的文件描述符数量的增加而增大，当监视大量文件描述符时，从用户空间到内核空间再返回的复制任务会消耗相当多的 CPU 时间。而对于*select()*，数据结构的大小由`FD_SETSIZE`固定，不受监视的文件描述符数量的影响。

+   在调用*select()*或*poll()*之后，程序必须检查返回的数据结构的每个元素，以查看哪些文件描述符已经准备就绪。

上述几点的结果是，*select()*和*poll()*所需的 CPU 时间会随着被监视的文件描述符数量的增加而增加（有关更多细节，请参见性能对比：*epoll*与 I/O 多路复用）。这对监视大量文件描述符的程序来说，会带来问题。

*select()*和*poll()*的扩展性差是由这些 API 的一个简单限制引起的：通常，程序会重复调用来监视相同的一组文件描述符；然而，内核不会在连续的调用之间记住要监视的文件描述符列表。

信号驱动 I/O 和*epoll*（我们将在接下来的章节中讨论），都是允许内核记录进程感兴趣的文件描述符持久列表的机制。这样做消除了*select()*和*poll()*的性能扩展问题，提供了根据发生的 I/O 事件数量而非监视的文件描述符数量扩展的解决方案。因此，当监视大量文件描述符时，信号驱动 I/O 和*epoll*提供了更好的性能。

## 信号驱动 I/O

使用 I/O 多路复用时，进程会进行系统调用（*select()*或*poll()*），以检查某个文件描述符上是否可以进行 I/O 操作。而在信号驱动 I/O 中，进程请求内核在文件描述符上可以进行 I/O 操作时发送信号给它。进程可以在此期间进行其他活动，直到 I/O 操作可用，届时信号会发送到进程。要使用信号驱动 I/O，程序需要执行以下步骤：

1.  为由信号驱动的 I/O 机制发送的信号建立一个处理程序。默认情况下，这个通知信号是`SIGIO`。

1.  设置文件描述符的*所有者*——即在文件描述符上 I/O 操作可用时应接收信号的进程或进程组。通常，我们将调用进程设置为所有者。所有者通过*fcntl()* `F_SETOWN`操作来设置，格式如下：

    ```
    fcntl(fd, F_SETOWN, pid);
    ```

1.  通过设置`O_NONBLOCK`打开文件状态标志来启用非阻塞 I/O。

1.  通过启用`O_ASYNC`打开文件状态标志来启用信号驱动 I/O。这可以与前一步骤结合使用，因为它们都需要使用*fcntl()* `F_SETFL`操作（第 5.3 节），如下所示：

    ```
    flags = fcntl(fd, F_GETFL);                 /* Get current flags */
    fcntl(fd, F_SETFL, flags | O_ASYNC | O_NONBLOCK);
    ```

1.  调用进程现在可以执行其他任务。当 I/O 变得可用时，内核会为进程生成一个信号，并调用在步骤 1 中建立的信号处理程序。

1.  信号驱动 I/O 提供了边缘触发的通知（哪种技术？）。这意味着一旦进程被通知 I/O 可用，它应该尽可能地执行 I/O 操作（例如，读取尽可能多的字节）。假设文件描述符是非阻塞的，这意味着执行一个循环，进行 I/O 系统调用，直到调用因错误`EAGAIN`或`EWOULDBLOCK`而失败。

在 Linux 2.4 及更早版本中，可以通过文件描述符使用信号驱动 I/O，适用于套接字、终端、伪终端和某些其他类型的设备。Linux 2.6 还允许信号驱动 I/O 用于管道和 FIFO。从 Linux 2.6.25 开始，信号驱动 I/O 还可以用于*inotify*文件描述符。

在接下来的页面中，我们首先展示一个使用信号驱动 I/O 的示例，然后更详细地解释上述一些步骤。

### 注意

历史上，信号驱动 I/O 有时被称为*异步 I/O*，这也反映在与之关联的打开文件状态标志（`O_ASYNC`）的名称中。然而，现在，*异步 I/O*一词通常用于指代 POSIX AIO 规范所提供的功能类型。使用 POSIX AIO 时，进程请求内核执行 I/O 操作，内核*启动*操作，但立即将控制权返回给调用进程；进程随后会在 I/O 操作完成或发生错误时得到通知。

`O_ASYNC`在 POSIX.1g 中有所规定，但由于该标志的行为规范被认为不足，因此未包含在 SUSv3 中。

一些 UNIX 实现，尤其是较老的实现，没有定义用于*fcntl()*的`O_ASYNC`常量。相反，该常量被命名为`FASYNC`，并且*glibc*将此名称定义为`O_ASYNC`的同义词。

#### 示例程序

示例 63-3 提供了一个简单的信号驱动 I/O 使用例子。该程序执行了上述启用标准输入信号驱动 I/O 的步骤，然后将终端设置为 cbreak 模式（Cooked, Cbreak, 和 Raw 模式），使得输入可以逐个字符地读取。然后，程序进入无限循环，在等待输入可用时执行“工作”，即递增一个变量 *cnt*。每当输入变得可用时，`SIGIO` 处理程序会设置一个标志 *gotSigio*，主程序会监控这个标志。当主程序看到该标志被设置时，它会读取所有可用的输入字符，并打印出它们及当前的 *cnt* 值。如果输入中读取到哈希符号（`#`），程序将终止。

下面是我们运行该程序并输入字符 *x* 多次，然后输入哈希符号（`#`）时看到的例子：

```
$ `./demo_sigio`
cnt=37; read x
cnt=100; read x
cnt=159; read x
cnt=223; read x
cnt=288; read x
cnt=333; read #
```

示例 63-3. 使用信号驱动 I/O 在终端上

```
`altio/demo_sigio.c`
#include <signal.h>
#include <ctype.h>
#include <fcntl.h>
#include <termios.h>
#include "tty_functions.h"      /* Declaration of ttySetCbreak() */
#include "tlpi_hdr.h"

static volatile sig_atomic_t gotSigio = 0;
                                /* Set nonzero on receipt of SIGIO */

static void
sigioHandler(int sig)
{
    gotSigio = 1;
}

int
main(int argc, char *argv[])
{
    int flags, j, cnt;
    struct termios origTermios;
    char ch;
    struct sigaction sa;
    Boolean done;

    /* Establish handler for "I/O possible" signal */

    sigemptyset(&sa.sa_mask);
    sa.sa_flags = SA_RESTART;
    sa.sa_handler = sigioHandler;
    if (sigaction(SIGIO, &sa, NULL) == -1)
        errExit("sigaction");

    /* Set owner process that is to receive "I/O possible" signal */

    if (fcntl(STDIN_FILENO, F_SETOWN, getpid()) == -1)
        errExit("fcntl(F_SETOWN)");

    /* Enable "I/O possible" signaling and make I/O nonblocking
       for file descriptor */

    flags = fcntl(STDIN_FILENO, F_GETFL);
    if (fcntl(STDIN_FILENO, F_SETFL, flags | O_ASYNC | O_NONBLOCK) == -1)
        errExit("fcntl(F_SETFL)");

    /* Place terminal in cbreak mode */

    if (ttySetCbreak(STDIN_FILENO, &origTermios) == -1)
        errExit("ttySetCbreak");

    for (done = FALSE, cnt = 0; !done ; cnt++) {
        for (j = 0; j < 100000000; j++)
            continue;                   /* Slow main loop down a little */

        if (gotSigio) {                 /* Is input available? */

            /* Read all available input until error (probably EAGAIN)
               or EOF (not actually possible in cbreak mode) or a
               hash (#) character is read */

            while (read(STDIN_FILENO, &ch, 1) > 0 && !done) {
                printf("cnt=%d; read %c\n", cnt, ch);
                done = ch == '#';
            }

            gotSigio = 0;
        }
    }

    /* Restore original terminal settings */

    if (tcsetattr(STDIN_FILENO, TCSAFLUSH, &origTermios) == -1)
        errExit("tcsetattr");
    exit(EXIT_SUCCESS);
}
     `altio/demo_sigio.c`
```

#### 在启用信号驱动 I/O 之前，先建立信号处理程序

由于 `SIGIO` 的默认操作是终止进程，我们应该在启用文件描述符上的信号驱动 I/O 之前，启用 `SIGIO` 的处理程序。如果在建立 `SIGIO` 处理程序之前启用了信号驱动 I/O，那么就会有一个时间窗口，在这个时间窗口内，如果 I/O 变得可用，`SIGIO` 的传递会终止进程。

### 注意

在一些 UNIX 实现中，默认情况下会忽略 `SIGIO`。

#### 设置文件描述符所有者

我们使用 *fcntl()* 操作来设置文件描述符的所有者，操作形式如下：

```
fcntl(fd, F_SETOWN, pid);
```

我们可以指定在文件描述符上的 I/O 可用时，单个进程或进程组中的所有进程应接收到信号。如果*pid*是正数，则它被解释为进程 ID。如果*pid*是负数，则其绝对值指定一个进程组 ID。

### 注意

在较老的 UNIX 实现中，*ioctl()* 操作——`FIOSETOWN` 或 `SIOCSPGRP`——用于实现与 `F_SETOWN` 相同的效果。为了兼容，Linux 上也提供了这些 *ioctl()* 操作。

通常，*pid* 被指定为调用进程的进程 ID（以便将信号发送到打开文件描述符的进程）。然而，也可以指定其他进程或进程组（例如调用者的进程组），信号会发送到该目标，前提是符合第 20.5 节中描述的权限检查，其中发送信号的进程被视为执行 `F_SETOWN` 的进程。

*fcntl()* `F_GETOWN` 操作返回当文件描述符上 I/O 可用时，应该接收信号的进程或进程组的 ID：

```
id = fcntl(fd, F_GETOWN);
if (id == -1)
    errExit("fcntl");
```

这个调用返回一个负数，表示进程组 ID。

### 注意

在较老的 UNIX 实现中，与 `F_GETOWN` 对应的 *ioctl()* 操作是 `FIOGETOWN` 或 `SIOCGPGRP`。这两个 *ioctl()* 操作在 Linux 上也可以使用。

在某些 Linux 架构（尤其是 x86）上使用的系统调用约定存在一个限制：如果文件描述符由进程组 ID 小于 4096 的进程所有，那么在执行 *fcntl()* `F_GETOWN` 操作时，*glibc* 并不会将该 ID 返回为负值的函数结果，而是误将其解释为系统调用错误。因此，*fcntl()* 封装函数会返回 -1，*errno* 中包含（正值）进程组 ID。这是因为内核系统调用接口通过返回负的 *errno* 值作为函数结果来指示错误，在某些情况下需要区分这种错误结果和返回有效负值的成功调用。为了做出这个区分，*glibc* 将 -1 到 -4095 范围内的负系统调用返回值解释为表示错误，并将该（绝对）值复制到 *errno* 中，然后将 -1 作为函数结果返回给应用程序。这个技术通常足以处理少数可以返回有效负值的系统调用服务例程；但 *fcntl()* `F_GETOWN` 操作是唯一一个无法正确处理的实际情况。这个限制意味着使用进程组接收“ I/O 可能”信号的应用程序（这种情况不常见）不能可靠地使用 `F_GETOWN` 来发现哪个进程组拥有文件描述符。

### 注意

从 *glibc* 版本 2.11 开始，*fcntl()* 封装函数修复了进程组 ID 小于 4096 时的 `F_GETOWN` 问题。它通过在用户空间实现 `F_GETOWN`，使用由 Linux 2.6.32 及更高版本提供的 `F_GETOWN_EX` 操作（“I/O 可能”何时被信号化？）来解决此问题。

### “I/O 可能”何时被信号化？

现在我们来讨论何时为各种文件类型信号化“ I/O 可能”。

#### 终端和伪终端

对于终端和伪终端，每当有新输入可用时，即使之前的输入尚未读取，也会生成信号。如果终端发生文件结束条件，也会发出“输入可能”信号（但伪终端不会）。

终端没有“输出可能”信号，也没有终端断开信号。

从内核版本 2.4.19 开始，Linux 为伪终端的从端提供“输出可能”信号。每当伪终端主端的输入被消费时，都会生成此信号。

#### 管道和 FIFO

对于管道或 FIFO 的读端，在以下情况下会生成信号：

+   数据被写入管道（即使已存在未读的输入）。

+   管道的写端被关闭时。

对于管道或 FIFO 的写端，在以下情况下会生成信号：

+   从管道读取增加了管道中的空闲空间，使得现在可以写入 `PIPE_BUF` 字节而不阻塞。

+   管道的读端已关闭。

#### 套接字

信号驱动 I/O 适用于 UNIX 域和 Internet 域中的数据报套接字。在以下情况下会生成信号：

+   套接字上收到一个输入数据报（即使已经有未读的数据报等待读取）。

+   套接字上发生了异步错误。

信号驱动 I/O 适用于 UNIX 域和 Internet 域中的流套接字。在以下情况下会生成信号：

+   在一个监听套接字上接收到一个新的连接。

+   TCP *connect()* 请求完成；即，TCP 连接的主动端进入了 ESTABLISHED 状态，如 图 61-5（第 1272 页）所示。对于 UNIX 域套接字，不会触发类似的信号。

+   套接字上收到新的输入数据（即使之前已经有未读输入数据）。

+   对端使用 *shutdown()* 关闭连接的写入半部分，或者使用 *close()* 完全关闭其套接字。

+   套接字上可以进行输出操作（例如，套接字发送缓冲区中已腾出空间）。

+   套接字上发生了异步错误。

#### *inotify* 文件描述符

当 *inotify* 文件描述符变为可读时，会生成一个信号——即，当 *inotify* 文件描述符监控的文件之一发生事件时。

### 精细化信号驱动 I/O 的使用

在需要同时监控大量（即数千个）文件描述符的应用程序中——例如，某些类型的网络服务器——信号驱动 I/O 相比于 *select()* 和 *poll()*，可以提供显著的性能优势。信号驱动 I/O 提供了优越的性能，因为内核“记住”了需要监控的文件描述符列表，并且只有在这些描述符上发生 I/O 事件时才会向程序发送信号。因此，使用信号驱动 I/O 的程序性能是根据发生的 I/O 事件数量来扩展的，而不是根据被监控的文件描述符数量。

为了充分利用信号驱动 I/O，我们必须执行两个步骤：

+   使用 Linux 特定的 *fcntl()* 操作 `F_SETSIG`，指定一个实时信号，当文件描述符上有 I/O 可用时，应该传递该信号而不是 `SIGIO`。

+   在使用 *sigaction()* 建立实时信号的处理程序时，指定 `SA_SIGINFO` 标志（参见第 21.4 节）。

*fcntl()* `F_SETSIG` 操作指定一个替代信号，在文件描述符上有 I/O 可用时，应该传递该信号而不是 `SIGIO`：

```
if (fcntl(fd, F_SETSIG, sig) == -1)
    errExit("fcntl");
```

`F_GETSIG` 操作执行 `F_SETSIG` 的反向操作，检索当前为文件描述符设置的信号：

```
sig = fcntl(fd, F_GETSIG);
if (sig == -1)
    errExit("fcntl");
```

（为了从`<fcntl.h>`中获取`F_SETSIG`和`F_GETSIG`常量的定义，我们必须定义`_GNU_SOURCE`功能测试宏。）

使用`F_SETSIG`更改用于“ I/O 可能”通知的信号有两个目的，这两个目的都是必需的，如果我们正在监视多个文件描述符上的大量 I/O 事件：

+   默认的“I/O 可能”信号`SIGIO`是标准的、非排队信号之一。如果在`SIGIO`被阻塞时发生多个 I/O 事件——可能是因为`SIGIO`处理程序已经被调用——除第一个通知外，所有通知都会丢失。如果我们使用`F_SETSIG`指定一个实时信号作为“I/O 可能”信号，则可以排队多个通知。

+   如果信号的处理程序是通过在*sa.sa_flags*字段中指定`SA_SIGINFO`标志的*sigaction()*调用建立的，则*siginfo_t*结构体作为第二个参数传递给信号处理程序（第 21.4 节）。该结构体包含识别事件发生的文件描述符以及事件类型的字段。

请注意，必须同时使用*both* `F_SETSIG`和`SA_SIGINFO`，才能有效地将*siginfo_t*结构体传递给信号处理程序。

如果我们执行`F_SETSIG`操作，并将*sig*设置为 0，则我们将恢复默认行为：`SIGIO`被传递，并且不会向处理程序提供*siginfo_t*参数。

对于“ I/O 可能”事件，传递给信号处理程序的*siginfo_t*结构体中感兴趣的字段如下：

+   *si_signo*：导致调用处理程序的信号的编号。这个值与信号处理程序的第一个参数相同。

+   *si_fd*：发生 I/O 事件的文件描述符。

+   *si_code*：指示发生的事件类型的代码。该字段中可能出现的值及其一般描述如下所示：表 63-7。

+   *si_band*：一个位掩码，包含与*poll()*系统调用中*revents*字段返回的相同位。*si_code*中设置的值与*si_band*中的位掩码设置具有一一对应关系，如表 63-7 所示。

    表 63-7。*si_code*和*si_band*值在*siginfo_t*结构体中的“ I/O 可能”事件

    | *si_code* | *si_band*掩码值 | 描述 |
    | --- | --- | --- |
    | `POLL_IN` | `POLLIN &#124; POLLRDNORM` | 输入可用；文件结尾条件 |
    | `POLL_OUT` | `POLLOUT &#124; POLLWRNORM &#124; POLLWRBAND` | 输出可能 |
    | `POLL_MSG` | `POLLIN &#124; POLLRDNORM &#124; POLLMSG` | 输入消息可用（未使用） |
    | `POLL_ERR` | `POLLERR` | I/O 错误 |
    | `POLL_PRI` | `POLLPRI &#124; POLLRDNORM` | 高优先级输入可用 |
    | `POLL_HUP` | `POLLHUP &#124; POLLERR` | 挂起发生 |

在一个完全由输入驱动的应用程序中，我们可以进一步优化 `F_SETSIG` 的使用。我们可以阻止指定的“ I/O 可能”信号，而不是通过信号处理程序来监视 I/O 事件，然后通过调用 *sigwaitinfo()* 或 *sigtimedwait()*（第 22.10 节）接受排队的信号。这些系统调用返回一个 *siginfo_t* 结构，包含与通过 `SA_SIGINFO` 设置的信号处理程序传递的信息相同的内容。通过这种方式接受信号使我们回到了同步事件处理模型，但有一个优势，就是与使用 *select()* 或 *poll()* 相比，我们可以更高效地收到有关发生 I/O 事件的文件描述符的通知。

#### 处理信号队列溢出

我们在第 22.8 节中看到，实时信号队列的数量是有限制的。如果达到这个限制，内核将恢复使用默认的 `SIGIO` 信号来通知“ I/O 可能”。这会通知进程发生了信号队列溢出。当发生这种情况时，我们无法知道哪些文件描述符发生了 I/O 事件，因为 `SIGIO` 并未被排队。（此外，`SIGIO` 处理程序不会接收到 *siginfo_t* 参数，这意味着信号处理程序无法确定哪个文件描述符产生了信号。）

我们可以通过增加实时信号队列的数量限制来减少信号队列溢出的可能性，如第 22.8 节所述。然而，这并不能消除处理溢出可能性的必要性。一个使用 `F_SETSIG` 来建立实时信号作为“ I/O 可能”通知机制的合理设计应用程序，还必须为 `SIGIO` 建立一个处理程序。如果 `SIGIO` 被传递，则应用程序可以使用 *sigwaitinfo()* 来清空实时信号队列，并暂时恢复使用 *select()* 或 *poll()* 来获得所有发生 I/O 事件的文件描述符的完整列表。

#### 使用信号驱动的 I/O 与多线程应用

从内核版本 2.6.32 开始，Linux 提供了两个新的非标准 *fcntl()* 操作，可以用来设置“ I/O 可能”信号的目标：`F_SETOWN_EX` 和 `F_GETOWN_EX`。

`F_SETOWN_EX` 操作类似于 `F_SETOWN`，但除了允许指定目标为进程或进程组外，还允许指定一个线程作为“ I/O 可能”信号的目标。对于此操作，*fcntl()* 的第三个参数是指向以下结构的指针：

```
struct f_owner_ex {
    int   type;
    pid_t pid;
};
```

*type* 字段定义了 *pid* 字段的含义，并且它有以下几种值：

`F_OWNER_PGRP`

*pid* 字段指定了将成为“ I/O 可能”信号目标的进程组的 ID。与 `F_SETOWN` 不同，进程组 ID 被指定为正值。

`F_OWNER_PID`

*pid* 字段指定了将成为“ I/O 可能”信号目标的进程的 ID。

`F_OWNER_TID`

*pid* 字段指定将成为“ I/O 可用”信号目标的线程的 ID。*pid* 中指定的 ID 是通过 *clone()* 或 *gettid()* 返回的值。

`F_GETOWN_EX` 操作是 `F_SETOWN_EX` 操作的逆操作。它使用 *fcntl()* 的第三个参数所指向的 *f_owner_ex* 结构，返回由先前的 `F_SETOWN_EX` 操作定义的设置。

### 注意

由于 `F_SETOWN_EX` 和 `F_GETOWN_EX` 操作表示进程组 ID 为正值，因此 `F_GETOWN_EX` 在使用小于 4096 的进程组 ID 时不会遇到 `F_GETOWN` 中描述的问题。

## *epoll* API

与 I/O 多路复用系统调用和信号驱动 I/O 类似，Linux 的 *epoll*（事件轮询）API 用于监控多个文件描述符，查看它们是否准备好进行 I/O 操作。*epoll* API 的主要优势如下：

+   当监控大量文件描述符时，*epoll* 的性能比 *select()* 和 *poll()* 要好得多。

+   *epoll* API 支持触发模式为水平触发或边缘触发的通知。相比之下，*select()* 和 *poll()* 仅提供水平触发通知，而信号驱动 I/O 仅提供边缘触发通知。

*epoll* 和信号驱动 I/O 的性能类似。然而，*epoll* 相比信号驱动 I/O 有一些优势：

+   我们避免了信号处理的复杂性（例如，信号队列溢出）。

+   我们在指定希望执行的监控类型时具有更大的灵活性（例如，检查某个套接字的文件描述符是否准备好进行读取、写入或两者兼有）。

*epoll* API 是 Linux 特有的，且在 Linux 2.6 中新增。

*epoll* API 的核心数据结构是 *epoll 实例*，通过一个打开的文件描述符来引用该实例。这个文件描述符不用于 I/O 操作，而是作为内核数据结构的句柄，具有两个用途：

+   记录此进程声明希望监控的文件描述符列表——*兴趣列表*；以及

+   维护一个准备好进行 I/O 操作的文件描述符列表——*就绪列表*。

准备就绪列表的成员是兴趣列表的一个子集。

对于每个由 *epoll* 监控的文件描述符，我们可以指定一个位掩码，表示我们希望了解的事件。这些位掩码与 *poll()* 使用的位掩码非常相似。

*epoll* API 包含三个系统调用：

+   *epoll_create()* 系统调用创建一个 *epoll* 实例并返回一个引用该实例的文件描述符。

+   *epoll_ctl()* 系统调用用于操作与 *epoll* 实例相关联的兴趣列表。通过 *epoll_ctl()*，我们可以将一个新的文件描述符添加到列表中，移除现有描述符，或修改用于监控某个描述符的事件掩码。

+   *epoll_wait()* 系统调用返回与 *epoll* 实例关联的准备就绪列表中的项目。

### 创建 *epoll* 实例：*epoll_create()*

*epoll_create()* 系统调用创建一个新的 *epoll* 实例，其兴趣列表最初为空。

```
#include <sys/epoll.h>

int `epoll_create`(int *size*);
```

### 注意

成功时返回文件描述符，错误时返回 -1

*size* 参数指定我们期望通过 *epoll* 实例监视的文件描述符数量。该参数不是上限，而是向内核提供的提示，指示如何初始分配内部数据结构。（从 Linux 2.6.8 开始，*size* 参数必须大于零，但否则会被忽略，因为实现的变化意味着该信息已不再需要。）

作为其功能结果，*epoll_create()* 返回一个文件描述符，指向新的 *epoll* 实例。该文件描述符用于在其他 *epoll* 系统调用中引用 *epoll* 实例。当不再需要该文件描述符时，应按常规方式使用 *close()* 关闭。当所有引用 *epoll* 实例的文件描述符被关闭时，该实例将被销毁，并且其相关资源将被释放回系统。（由于调用 *fork()* 或使用 *dup()* 或类似方法进行描述符复制，多个文件描述符可能会引用同一个 *epoll* 实例。）

### 注意

从内核 2.6.27 开始，Linux 支持一个新的系统调用 *epoll_create1()*。该系统调用执行与 *epoll_create()* 相同的任务，但去除了过时的 *size* 参数，并添加了一个 flags 参数，用于修改系统调用的行为。目前支持的一个标志是 `EPOLL_CLOEXEC`，它使内核为新的文件描述符启用 close-on-exec 标志 `(FD_CLOEXEC)`。这个标志与 *open()* 的 `O_CLOEXEC` 标志类似，原因请参见 由 *open()* 返回的文件描述符号 返回的文件描述符号")。

### 修改 *epoll* 兴趣列表：*epoll_ctl()*

*epoll_ctl()* 系统调用修改由文件描述符 *epfd* 引用的 *epoll* 实例的兴趣列表。

```
#include <sys/epoll.h>

int `epoll_ctl`(int *epfd*, int *op*, int *fd*, struct epoll_event **ev*);
```

### 注意

成功时返回 0，错误时返回 -1

*fd* 参数标识兴趣列表中要修改设置的文件描述符。此参数可以是管道、FIFO、套接字、POSIX 消息队列、*inotify* 实例、终端、设备，甚至是另一个 *epoll* 描述符的文件描述符（即，我们可以构建一个监视描述符的层次结构）。然而，*fd* 不能是常规文件或目录的文件描述符（会导致错误 `EPERM`）。

*op* 参数指定要执行的操作，并具有以下值之一：

`EPOLL_CTL_ADD`

将文件描述符 *fd* 添加到 *epfd* 的兴趣列表中。我们希望监视 *fd* 的事件集合在 *ev* 指向的缓冲区中指定，如下所述。如果我们尝试添加一个已经在兴趣列表中的文件描述符，*epoll_ctl()* 会因错误 `EEXIST` 而失败。

`EPOLL_CTL_MOD`

修改文件描述符 *fd* 的事件设置，使用 *ev* 指向的缓冲区中指定的信息。如果我们尝试修改一个不在 *epfd* 兴趣列表中的文件描述符的设置，*epoll_ctl()* 会因错误 `ENOENT` 而失败。

`EPOLL_CTL_DEL`

从 *epfd* 的兴趣列表中移除文件描述符 *fd*。此操作忽略 *ev* 参数。如果我们尝试移除一个不在 *epfd* 兴趣列表中的文件描述符，*epoll_ctl()* 会因错误 `ENOENT` 而失败。关闭文件描述符会自动将其从所有包含它的 epoll 兴趣列表中移除。

*ev* 参数是指向类型为 *epoll_event* 的结构体的指针，其定义如下：

```
struct epoll_event {
    uint32_t     events;        /* epoll events (bit mask) */
    epoll_data_t data;          /* User data */
};
```

*epoll_event* 结构体的 *data* 字段的类型如下：

```
typedef union epoll_data {
    void        *ptr;           /* Pointer to user-defined data */
    int          fd;            /* File descriptor */
    uint32_t     u32;           /* 32-bit integer */
    uint64_t     u64;           /* 64-bit integer */
} epoll_data_t;
```

*ev* 参数指定文件描述符 *fd* 的设置，具体如下：

+   *events* 子字段是一个位掩码，指定我们感兴趣的文件描述符 *fd* 的事件集合。在下一节中，我们将详细说明可以在此字段中使用的位值。

+   *data* 子字段是一个联合体，其成员之一可以用来指定如果 *fd* 后来变为就绪状态时，通过 *epoll_wait()* 返回给调用进程的信息。

示例 63-4 和 epoll_ctl()") 展示了 *epoll_create()* 和 *epoll_ctl()* 的使用示例。

示例 63-4. 使用 *epoll_create()* 和 *epoll_ctl()*

```
int epfd;
    struct epoll_event ev;

    epfd = epoll_create(5);
    if (epfd == -1)
        errExit("epoll_create");

    ev.data.fd = fd;
    ev.events = EPOLLIN;
    if (epoll_ctl(epfd, EPOLL_CTL_ADD, fd, ev) == -1)
        errExit("epoll_ctl");
```

#### `max_user_watches` 限制

由于在 *epoll* 兴趣列表中注册的每个文件描述符都需要少量不可交换的内核内存，内核提供了一个接口，定义了每个用户可以在所有 *epoll* 兴趣列表中注册的文件描述符的总数的限制。此限制的值可以通过 `/proc/sys/fs/epoll` 目录中的一个 Linux 特有的文件 `max_user_watches` 查看和修改。此限制的默认值是根据可用的系统内存计算的（请参阅 *epoll(7)* 手册页）。

### 等待事件：*epoll_wait()*

*epoll_wait()* 系统调用返回有关文件描述符 *epfd* 所引用的 *epoll* 实例中就绪文件描述符的信息。单次 *epoll_wait()* 调用可以返回多个就绪文件描述符的信息。

```
#include <sys/epoll.h>

int `epoll_wait`(int *epfd*, struct epoll_event **evlist*, int
 *maxevents*, int *timeout*);
```

### 注意

返回就绪文件描述符的数量，超时时返回 0，出错时返回 -1。

有关就绪文件描述符的信息通过 *evlist* 指向的 *epoll_event* 结构体数组返回。（*epoll_event* 结构体在上一节中已描述。）*evlist* 数组由调用者分配，其包含的元素数量由 *maxevents* 指定。

数组*evlist*中的每一项返回关于单个就绪文件描述符的信息。*events*子字段返回该描述符上发生的事件的掩码。*data*子字段返回在我们使用*epoll_ctl()*注册对该文件描述符的兴趣时，指定的任何值。请注意，*data*字段是唯一能让我们找出与此事件相关联的文件描述符号的机制。因此，当我们调用*epoll_ctl()*将文件描述符添加到兴趣列表时，我们应该将*ev.data.fd*设置为文件描述符号（如示例 63-4 和 epoll_ctl()")所示），或者将*ev.data.ptr*设置为指向包含文件描述符号的结构体。

*timeout*参数决定了*epoll_wait()*的阻塞行为，如下所示：

+   如果*timeout*等于-1，阻塞直到一个事件发生在*epfd*的兴趣列表中的某个文件描述符上，或者直到捕获到信号。

+   如果*timeout*等于 0，执行非阻塞检查，查看*epfd*的兴趣列表中哪些事件当前可用。

+   如果*timeout*大于 0，则阻塞最多*timeout*毫秒，直到在*epfd*的兴趣列表中的某个文件描述符上发生事件，或者直到捕获到信号。

如果成功，*epoll_wait()*返回已放入数组*evlist*中的项目数，或者如果在*timeout*指定的时间间隔内没有文件描述符准备好，则返回 0。如果发生错误，*epoll_wait()*返回-1，并将*errno*设置为指示错误。

在多线程程序中，可能有一个线程使用*epoll_ctl()*将文件描述符添加到已由另一个线程的*epoll_wait()*监视的 epoll 实例的兴趣列表中。这些对兴趣列表的更改会立即生效，*epoll_wait()*调用将返回关于新添加的文件描述符的就绪信息。

#### *epoll*事件

我们在调用*epoll_ctl()*时可以指定的位值，以及*epoll_wait()*返回的*evlist[].events*字段中放置的值，见表 63-8。通过添加`E`前缀，大部分这些位与*poll()*中使用的相应事件位名称相同。（例外的是`EPOLLET`和`EPOLLONESHOT`，我们将在下面更详细地描述它们。）这种对应关系的原因是，当这些位作为输入指定给*epoll_ctl()*，或者通过*epoll_wait()*返回作为输出时，它们传达的含义与相应的*poll()*事件位完全相同。

表 63-8. *epoll 事件*字段的位掩码值

| 位 | 输入到*epoll_ctl()*? | *epoll_wait()*返回? | 描述 |
| --- | --- | --- | --- |
| `EPOLLIN` | • | • | 可以读取除高优先级数据外的其他数据 |
| `EPOLLPRI` | • | • | 可以读取高优先级数据 |
| `EPOLLRDHUP` | • | • | 对等套接字的关闭（自 Linux 2.6.17 起） |
| `EPOLLOUT` | • | • | 可以写入正常数据 |
| `EPOLLET` | • |   | 使用边缘触发事件通知 |
| `EPOLLONESHOT` | • |   | 事件通知后禁用监控 |
| `EPOLLERR` |   | • | 发生错误 |
| `EPOLLHUP` |   | • | 发生挂起 |

#### `EPOLLONESHOT`标志

默认情况下，一旦使用*epoll_ctl()*的`EPOLL_CTL_ADD`操作将文件描述符添加到*epoll*兴趣列表中，它将保持活跃状态（即，后续调用*epoll_wait()*时，会在文件描述符准备好时通知我们），直到我们明确使用*epoll_ctl()*的`EPOLL_CTL_DEL`操作将其从列表中移除。如果我们只希望收到一次特定文件描述符的通知，可以在传递给*epoll_ctl()*的*ev.events*值中指定`EPOLLONESHOT`标志（自 Linux 2.6.2 版本起提供）。如果指定了此标志，那么，在下次*epoll_wait()*调用通知我们对应的文件描述符已准备好之后，文件描述符将被标记为非活动状态，后续的*epoll_wait()*调用将不会再通知它的状态。如果需要，我们可以使用*epoll_ctl()*的`EPOLL_CTL_MOD`操作重新启用对该文件描述符的监控。（我们不能使用`EPOLL_CTL_ADD`操作来达到这个目的，因为非活动的文件描述符仍然是*epoll*兴趣列表的一部分。）

#### 示例程序

示例 63-5 演示了如何使用*epoll* API。作为命令行参数，程序期望一个或多个终端或 FIFO 的路径名。程序执行以下步骤：

+   创建一个*epoll*实例！[](figs/web/U001.png)。

+   打开命令行中指定的每个文件进行输入！[](figs/web/U002.png)，并将结果文件描述符添加到*epoll*实例的兴趣列表中！[](figs/web/U003.png)，同时指定要监控的事件集为`EPOLLIN`。

+   执行一个循环！[](figs/web/U004.png)，该循环调用*epoll_wait()*！[](figs/web/U005.png)来监控*epoll*实例的兴趣列表，并处理每次调用返回的事件。请注意以下几点：

    +   在*epoll_wait()*调用之后，程序会检查是否返回`EINTR`！[](figs/web/U006.png)，这种情况可能发生在程序在*epoll_wait()*调用的过程中被信号中断，随后通过`SIGCONT`信号恢复运行时。（参见第 21.5 节。）如果发生这种情况，程序将重新启动*epoll_wait()*调用。

    +   如果*epoll_wait()*调用成功，程序会使用一个进一步的循环检查*evlist*中的每个就绪项 ![](img/U007.png)。对于*evlist*中的每个项，程序会检查*events*字段，不仅检查是否存在`EPOLLIN` ![](img/U008.png)，还会检查是否存在`EPOLLHUP`和`EPOLLERR` ![](img/U009.png)。这些事件可能发生在 FIFO 的另一端被关闭或发生终端挂起时。如果返回了`EPOLLIN`，则程序会从相应的文件描述符读取一些输入并显示到标准输出上。否则，如果发生了`EPOLLHUP`或`EPOLLERR`，程序会关闭相应的文件描述符 ![](img/U010.png) 并减少打开文件的计数器（*numOpenFds*）。

    +   当所有打开的文件描述符都被关闭时（即当*numOpenFds*等于 0 时），循环终止。

以下 Shell 会话日志演示了示例 63-5 中程序的使用方法。我们使用两个终端窗口。在一个窗口中，我们使用示例 63-5 中的程序来监视两个 FIFO 的输入。（本程序对 FIFO 进行读取的每一次打开，只有在另一个进程已打开 FIFO 进行写入之后才能完成，正如第 44.7 节所述。）在另一个窗口中，我们运行*cat(1)*实例，将数据写入这些 FIFO。

```
`Terminal window 1`                   `Terminal window 2`
$ `mkfifo p q`
$ `./epoll_input p q`
                                    $ `cat > p`
Opened "p" on fd 4
                                    *Type Control-Z to suspend cat*
                                    [1]+  Stopped      cat >p
                                    $ `cat > q`
Opened "q" on fd 5
About to epoll_wait()
*Type Control-Z to suspend the epoll_input program*
[1]+  Stopped     ./epoll_input p q
```

上述内容中，我们暂停了监控程序，以便现在可以在两个 FIFO 上生成输入，并关闭其中一个的写入端：

```
`qqq`
                                    *Type Control-D to terminate “cat > q”*
                                    $ `fg %1`
                                    cat >p
                                    `ppp`
```

现在我们通过将监控程序带到前台来恢复它的运行，此时*epoll_wait()*返回两个事件：

```
$ `fg`
./epoll_input p q
About to epoll_wait()
Ready: 2
  fd=4; events: EPOLLIN
    read 4 bytes: ppp

  fd=5; events: EPOLLIN EPOLLHUP
    read 4 bytes: qqq

    closing fd 5
About to epoll_wait()
```

上述输出中的两个空行是由*cat*的实例读取后写入 FIFO，再由我们的监控程序读取并回显的换行符。

现在我们在第二个终端窗口中输入*Control-D*，以终止剩余的*cat*实例，这会导致*epoll_wait()*再次返回，这次只有一个事件：

```
*Type Control-D to terminate “cat >p”*
Ready: 1
  fd=4; events: EPOLLHUP
    closing fd 4
All file descriptors closed; bye
```

示例 63-5. 使用*epoll* API

```
`altio/epoll_input.c`
    #include <sys/epoll.h>
    #include <fcntl.h>
    #include "tlpi_hdr.h"

    #define MAX_BUF     1000        /* Maximum bytes fetched by a single read() */
    #define MAX_EVENTS     5        /* Maximum number of events to be returned from
                                       a single epoll_wait() call */

    int
    main(int argc, char *argv[])
    {
        int epfd, ready, fd, s, j, num0penFds;
        struct epoll_event ev;
        struct epoll_event evlist[MAX_EVENTS];
        char buf[MAX_BUF];

        if (argc < 2 || strcmp(argv[1], "--help") == 0)
            usageErr("%s file...\n", argv[0]);

    epfd = epoll_create(argc - 1);
        if (epfd == -1)
            errExit("epoll_create");

        /* Open each file on command line, and add it to the "interest
           list" for the epoll instance */

    for (j = 1; j < argc; j++) {
            fd = open(argv[j], O_RDONLY);
            if (fd == -1)
                errExit("open");
            printf("Opened \"%s\" on fd %d\n", argv[j], fd);

            ev.events = EPOLLIN;            /* Only interested in input events */
            ev.data.fd = fd;
        if (epoll_ctl(epfd, EPOLL_CTL_ADD, fd, &ev) == -1)
                errExit("epoll_ctl");
        }

        numOpenFds = argc - 1;

    while (numOpenFds > 0) {

            /* Fetch up to MAX_EVENTS items from the ready list */

            printf("About to epoll_wait()\n");
        ready = epoll_wait(epfd, evlist, MAX_EVENTS, -1);
            if (ready == -1) {
            if (errno == EINTR)
                    continue;               /* Restart if interrupted by signal */
                else
                    errExit("epoll_wait");
            }

                printf("Ready: %d\n", ready);

            /* Deal with returned list of events */

        for (j = 0; j < ready; j++) {
                printf("  fd=%d; events: %s%s%s\n", evlist[j].data.fd,
                        (evlist[j].events & EPOLLIN)  ? "EPOLLIN "  : "",
                        (evlist[j].events & EPOLLHUP) ? "EPOLLHUP " : "",
                        (evlist[j].events & EPOLLERR) ? "EPOLLERR " : "");

            if (evlist[j].events & EPOLLIN) {
                    s = read(evlist[j].data.fd, buf, MAX_BUF);
                    if (s == -1)
                        errExit("read");
                    printf("    read %d bytes: %.*s\n", s, s, buf);

            } else if (evlist[j].events & (EPOLLHUP | EPOLLERR)) {

                    /* If EPOLLIN and EPOLLHUP were both set, then there might
                       be more than MAX_BUF bytes to read. Therefore, we close
                       the file descriptor only if EPOLLIN was not set.
                       We'll read further bytes after the next epoll_wait(). */

                    printf("    closing fd %d\n", evlist[j].data.fd);
                if (close(evlist[j].data.fd) == -1)
                        errExit("close");
                    numOpenFds--;
                }
            }
        }

        printf("All file descriptors closed; bye\n");
        exit(EXIT_SUCCESS);
    }

          `altio/epoll_input.c`
```

### 更深入地了解*epoll*语义

我们现在来看一下打开文件、文件描述符和*epoll*交互的细微之处。为了讨论的需要，值得回顾一下图 5-2（第 95 页），该图展示了文件描述符、打开的文件描述符和系统范围的文件 i 节点表之间的关系。

当我们使用*epoll_create()*创建一个*epoll*实例时，内核会创建一个新的内存中的 i 节点和打开的文件描述符，并在调用进程中分配一个新的文件描述符，该文件描述符指向打开的文件描述符。*epoll*实例的兴趣列表与打开的文件描述符关联，而不是与*epoll*文件描述符关联。这有以下几个后果：

+   如果我们使用*dup()*（或类似方法）复制一个*epoll*文件描述符，那么该复制的描述符将指向与原始描述符相同的*epoll*兴趣列表和就绪列表。我们可以通过在调用*epoll_ctl()*时将任一文件描述符指定为*epfd*参数来修改兴趣列表。同样，我们可以通过在调用*epoll_wait()*时将任一文件描述符指定为*epfd*参数来从就绪列表中检索项目。

+   前述的情况在调用*fork()*后也适用。子进程继承父进程的*epoll*文件描述符的副本，并且这个副本描述符指向相同的*epoll*数据结构。

当我们执行*epoll_ctl()*的`EPOLL_CTL_ADD`操作时，内核会将一个项目添加到*epoll*兴趣列表中，记录被监视的文件描述符的编号以及对相应打开文件描述符的引用。为了*epoll_wait()*调用的目的，内核监视打开的文件描述符。这意味着我们需要完善之前的说法，即当文件描述符关闭时，它会自动从所有包含它的*epoll*兴趣列表中移除。完善后的说法是：只有当所有指向某个打开文件描述符的文件描述符都关闭时，打开文件描述符才会从*epoll*兴趣列表中移除。这意味着，如果我们创建指向一个打开文件的重复描述符——使用*dup()*（或类似方法）或*fork()*——那么只有在原始描述符和所有重复描述符都关闭之后，打开文件才会从*epoll*兴趣列表中移除。

这些语义可能导致一些初看起来令人惊讶的行为。假设我们执行示例 63-6 中显示的代码。在这段代码中，*epoll_wait()*调用会告诉我们文件描述符*fd1*已就绪（换句话说，*evlist[0].data.fd*将等于*fd1*），尽管*fd1*已经被关闭。这是因为仍然存在一个打开的文件描述符*fd2*，它指向*epoll*兴趣列表中包含的打开文件描述符。当两个进程持有指向相同打开文件描述符的重复描述符时（通常是通过*fork()*的结果），并且执行*epoll_wait()*的进程已关闭其文件描述符，但另一个进程仍然保持着重复描述符打开时，会发生类似的情况。

示例 63-6. 具有重复文件描述符的*epoll*语义

```
int epfd, fd1, fd2;
    struct epoll_event ev;
    struct epoll_event evlist[MAX_EVENTS];

    /* Omitted: code to open 'fd1' and create epoll file descriptor 'epfd' ... */

    ev.data.fd = fd1
    ev.events = EPOLLIN;
    if (epoll_ctl(epfd, EPOLL_CTL_ADD, fd1, ev) == -1)
        errExit("epoll_ctl");

    /* Suppose that 'fd1' now happens to become ready for input */

    fd2 = dup(fd1);
    close(fd1);
    ready = epoll_wait(epfd, evlist, MAX_EVENTS, -1);
    if (ready == -1)
        errExit("epoll_wait");
```

### *epoll*与 I/O 多路复用的性能

表 63-9, select(), and epoll for 100,000 monitoring operations")显示了在 Linux 2.6.25 上，使用*poll()*、*select()*和*epoll*监控从*0*到*N - 1*范围内的*N*个连续文件描述符时的结果。（该测试的安排是每次监控操作中，恰好一个随机选中的文件描述符变为就绪。）从表中可以看出，随着监控的文件描述符数量的增加，*poll()*和*select()*的性能表现较差。相比之下，*epoll*的性能几乎不会随着*N*的增大而下降。（*N*增大时性能的轻微下降，可能是由于测试系统达到了 CPU 缓存的限制。）

### 注意

在此测试中，`FD_SETSIZE`在*glibc*头文件中被更改为 16,384，以允许测试程序使用*select()*监控大量文件描述符。

表 63-9. *poll()*、*select()*和*epoll*在 100,000 次监控操作中的耗时

| 监控的描述符数量（*N*） | *poll()* CPU 时间（秒） | *select()* CPU 时间（秒） | *epoll* CPU 时间（秒） |
| --- | --- | --- | --- |
| `10` | `0.61` | `0.73` | `0.41` |
| `100` | `2.9` | `3.0` | `0.42` |
| `1000` | `35` | `35` | `0.53` |
| `10000` | `990` | `930` | `0.66` |

在问题与*select()*和*poll()* and poll()")中，我们已经看到为什么*select()*和*poll()*在监控大量文件描述符时性能较差。接下来我们来探讨为什么*epoll*表现更好的原因：

+   在每次调用*select()*或*poll()*时，内核必须检查调用中指定的所有文件描述符。相比之下，当我们使用*epoll_ctl()*标记一个描述符以进行监控时，内核会将这一事实记录在与底层打开文件描述符相关联的列表中，并且每当执行使文件描述符变为就绪的 I/O 操作时，内核会将一个条目添加到*epoll*描述符的就绪列表中。（对单个打开文件描述符的 I/O 事件可能导致与该描述符相关联的多个文件描述符变为就绪。）随后的*epoll_wait()*调用仅从就绪列表中提取条目。

+   每次我们调用*select()*或*poll()*时，我们都会向内核传递一个数据结构，用于标识所有需要监控的文件描述符，并且在返回时，内核会返回一个描述所有这些描述符就绪状态的数据结构。相比之下，使用*epoll*时，我们使用*epoll_ctl()*在*内核空间*构建一个数据结构，列出需要监控的文件描述符集合。一旦构建了这个数据结构，后续的每次*epoll_wait()*调用都不需要向内核传递任何关于文件描述符的信息，调用返回的信息仅涉及那些已经就绪的描述符。

### 注意

除了上述几点，对于*select()*，我们必须在每次调用之前初始化输入数据结构；对于*select()*和*poll()*，我们必须检查返回的数据结构，以找出哪些*N*文件描述符已准备就绪。然而，一些测试表明，除了系统调用监视*N*个描述符所需的时间外，其他步骤所需的时间是微不足道的。表 63-9、select()和 epoll 在 100,000 次监控操作中的耗时")中未包括检查步骤的时间。

非常粗略地说，对于大值的*N*（被监视的文件描述符数量），*select()*和*poll()*的性能随着*N*线性扩展。我们在表 63-9、select()和 epoll 在 100,000 次监控操作中的耗时")中的*N = 100*和*N = 1000*案例中开始看到这种行为。到*N = 10000*时，扩展性实际上已经变得比线性更差。

相比之下，*epoll*的扩展性（线性）与发生的 I/O 事件数量相关。因此，*epoll* API 在一个典型的服务器场景中尤其高效，该场景中服务器需要处理许多同时连接的客户端：在被监视的众多文件描述符中，大部分是空闲的，只有少数描述符已准备就绪。

### 边缘触发通知

默认情况下，*epoll*机制提供*电平触发*通知。这里指的是，*epoll*会告诉我们是否可以在文件描述符上执行 I/O 操作而不发生阻塞。这与*poll()*和*select()*提供的通知类型相同。

*epoll* API 还允许进行*边缘触发*通知——即，调用*epoll_wait()*告诉我们自上次调用*epoll_wait()*以来（如果没有上次调用，则自描述符打开以来）文件描述符是否发生了 I/O 活动。使用带有边缘触发通知的*epoll*语义上类似于信号驱动的 I/O，区别在于，如果发生多个 I/O 事件，*epoll*将它们合并为通过*epoll_wait()*返回的单个通知；而信号驱动 I/O 可能会生成多个信号。

要使用边缘触发通知，我们在调用*epoll_ctl()*时在*ev.events*中指定`EPOLLET`标志：

```
struct epoll_event ev;

ev.data.fd = fd
ev.events = EPOLLIN | EPOLLET;
if (epoll_ctl(epfd, EPOLL_CTL_ADD, fd, ev) == -1)
    errExit("epoll_ctl");
```

我们通过一个例子来说明*电平触发*和*边缘触发*的*epoll*通知之间的区别。假设我们使用*epoll*来监视一个套接字的输入（`EPOLLIN`），并且发生了以下步骤：

1.  输入数据到达套接字。

1.  我们执行*epoll_wait()*。无论是使用电平触发还是边缘触发通知，这次调用都会告诉我们套接字已准备就绪。

1.  我们进行第二次调用*epoll_wait()*。

如果我们使用的是基于级别的通知，那么第二次调用*epoll_wait()*将告知我们该套接字已准备好。如果我们使用的是基于边缘的通知，则第二次调用*epoll_wait()*会被阻塞，因为自上次调用*epoll_wait()*以来没有新的输入到达。

正如我们在中所指出的，基于边缘的通知通常与非阻塞文件描述符一起使用。因此，使用基于边缘的*epoll*通知的通用框架如下：

1.  将所有需要被监控的文件描述符设置为非阻塞模式。

1.  使用*epoll_ctl()*构建*epoll*兴趣列表。

1.  使用以下循环处理 I/O 事件：

    1.  使用*epoll_wait()*获取已准备好的描述符列表。

    1.  对于每个准备好的文件描述符，进行 I/O 操作，直到相关的系统调用（例如，*read()*, *write()*, *recv()*, *send()*, 或 *accept()*）返回错误 *EAGAIN* 或 `EWOULDBLOCK`。

#### 防止在使用基于边缘的通知时出现文件描述符饿死问题

假设我们正在使用基于边缘的通知监控多个文件描述符，并且一个准备好的文件描述符有大量（可能是无尽的）输入可用。如果在检测到该文件描述符准备好之后，我们试图通过非阻塞读取消耗所有输入，那么我们可能会导致其他文件描述符得不到关注（即可能需要很长时间才能再次检查它们的准备状态并对它们进行 I/O 操作）。解决这个问题的一种方法是让应用程序维护一个已通知为准备好的文件描述符列表，并执行一个循环，持续执行以下操作：

1.  使用*epoll_wait()*监控文件描述符，并将准备好的描述符添加到应用程序列表中。如果应用程序列表中已经注册了准备好的文件描述符，则该监控步骤的超时时间应该设置得较小或为 0，这样如果没有新的文件描述符准备好，应用程序可以迅速进入下一个步骤，并处理已知为准备好的文件描述符。

1.  对那些在应用程序列表中标记为准备好的文件描述符执行有限的 I/O 操作（可以采用轮询的方式循环遍历它们，而不是每次调用*epoll_wait()*后都从列表开头开始）。当相关的非阻塞 I/O 系统调用因 `EAGAIN` 或 `EWOULDBLOCK` 错误失败时，可以将文件描述符从应用程序列表中移除。

尽管这种方法需要额外的编程工作，但它除了防止文件描述符饿死问题外，还带来了其他好处。例如，我们可以在上述循环中包含其他步骤，比如处理定时器和使用*sigwaitinfo()*（或类似方法）接收信号。

在使用信号驱动的 I/O 时，也可能会出现饿死问题，因为它同样提供了一个边沿触发的通知机制。相比之下，饿死问题在采用水平触发通知机制的应用中并不一定适用。这是因为我们可以在使用水平触发通知的情况下，采用阻塞文件描述符，并使用一个循环不断检查描述符是否准备好，然后对准备好的描述符执行 *某些* I/O 操作，再次检查文件描述符是否准备好。

## 等待信号和文件描述符

有时，进程需要同时等待一组文件描述符中的某个文件描述符是否可以进行 I/O 操作，或者等待信号的传递。我们可能会尝试使用 *select()* 来执行这样的操作，如 示例 63-7") 中所示。

示例 63-7. 错误的解除信号阻塞方法与调用 *select()*

```
sig_atomic_t gotSig = 0;

void
handler(int sig)
{
    gotSig = 1;
}

int
main(int argc, char *argv[])
{
    struct sigaction sa;
    ...

    sa.sa_sigaction = handler;
    sigemptyset(&sa.sa_mask);
    sa.sa_flags = 0;
    if (sigaction(SIGUSR1, &sa, NULL) == -1)
        errExit("sigaction");

    /* What if the signal is delivered now? */

    ready = select(nfds, &readfds, NULL, NULL, NULL);
    if (ready > 0) {
        printf("%d file descriptors ready\n", ready);
    } else if (ready == -1 && errno == EINTR) {
        if (gotSig)
            printf("Got signal\n");
    } else {
        /* Some other error */
    }

    ...
}
```

该代码的问题在于，如果信号（在本例中为 `SIGUSR1`）在建立处理程序之后但在调用 *select()* 之前到达，那么 *select()* 调用仍然会被阻塞。（这是一种竞争条件。）接下来我们将查看解决此问题的一些方法。

### 注意

自版本 2.6.27 起，Linux 提供了一种新技术，可以同时等待信号和文件描述符：*signalfd* 机制，详见第 22.11 节。通过此机制，我们可以通过文件描述符接收信号，并使用 *select()*、*poll()* 或 *epoll_wait()* 监控该描述符（以及其他文件描述符）。

### *pselect()* 系统调用

*pselect()* 系统调用执行的任务与 *select()* 类似，主要的语义差异是一个额外的参数 *sigmask*，它指定在调用被阻塞期间要解除屏蔽的信号集。

```
#include <sys/select.h>

int `pselect`(int *nfds*, fd_set **readfds*, fd_set **writefds*, fd_set **exceptfds*,
            struct timespec **timeout*, const sigset_t **sigmask*);
```

### 注意

返回准备好的文件描述符数量，超时时返回 0，出错时返回 -1

更精确地说，假设我们有以下的 *pselect()* 调用：

```
ready = pselect(nfds, &readfds, &writefds, &exceptfds, timeout, &sigmask);
```

该调用相当于原子性地执行以下步骤：

```
sigset_t origmask;

sigprocmask(SIG_SETMASK, &sigmask, &origmask);
ready = select(nfds, &readfds, &writefds, &exceptfds, timeout);
sigprocmask(SIG_SETMASK, &origmask, NULL);        /* Restore signal mask */
```

使用 *pselect()*，我们可以将主程序主体的第一部分重写为 示例 63-7") 中的内容，如 示例 63-8") 所示。

除了 *sigmask* 参数外，*select()* 和 *pselect()* 在以下几个方面有所不同：

+   *timeout* 参数传递给 *pselect()* 时使用的是 *timespec* 结构体（高精度休眠：*nanosleep()*")），它允许以纳秒（而非微秒）精度指定超时时间。

+   SUSv3 明确指出，*pselect()* 在返回时不会修改 *timeout* 参数。

如果我们将*pselect()*的*sigmask*参数指定为`NULL`，则*pselect()*相当于*select()*（即，它不对进程信号屏蔽做任何操作），除了刚才提到的差异。

*pselect()*接口是 POSIX.1g 的发明，现已纳入 SUSv3。它并非所有 UNIX 实现都支持，并且仅在内核 2.6.16 中才添加到 Linux。

### 注意

之前，*pselect()*库函数由*glibc*提供，但该实现未能提供正确操作调用所需的原子性保证。只有*pselect()*的内核实现才能提供这种保证。

示例 63-8. 使用*pselect()*

```
sigset_t emptyset, blockset;
    struct sigaction sa;

    sigemptyset(&blockset);
    sigaddset(&blockset, SIGUSR1);

    if (sigprocmask(SIG_BLOCK, &blockset, NULL) == -1)
        errExit("sigprocmask");

    sa.sa_sigaction = handler;
    sigemptyset(&sa.sa_mask);
    sa.sa_flags = SA_RESTART;
    if (sigaction(SIGUSR1, &sa, NULL) == -1)
        errExit("sigaction");

    sigemptyset(&emptyset);
    ready = pselect(nfds, &readfds, NULL, NULL, NULL, &emptyset);
    if (ready == -1)
        errExit("pselect");
```

#### *ppoll()*和*epoll_pwait()*系统调用

Linux 2.6.16 还添加了一个新的非标准系统调用*ppoll()*，它与*poll()*的关系类似于*pselect()*与*select()*的关系。同样，从内核 2.6.19 开始，Linux 还包括*epoll_pwait()*，为*epoll_wait()*提供了类似的扩展。有关详细信息，请参见*ppoll(2)*和*epoll_pwait(2)*手册页面。

### 自管道技巧

由于*pselect()*并未广泛实现，因此可移植的应用程序必须采用其他策略，以避免在同时等待信号和在一组文件描述符上调用*select()*时出现竞态条件。一个常见的解决方案是：

1.  创建一个管道，并将其读写端标记为非阻塞。

1.  除了监视所有其他感兴趣的文件描述符外，还应将管道的读端包括在传递给*select()*的*readfds*集合中。

1.  为感兴趣的信号安装一个处理程序。当此信号处理程序被调用时，它会向管道写入一个字节的数据。请注意以下几点：

    +   在第一步中，管道的写端被标记为非阻塞，以防止信号到达得过快，导致信号处理程序的多次调用填满管道，从而导致信号处理程序的*write()*（进程本身）被阻塞。（如果写入已满的管道失败也无关紧要，因为之前的写入已经表示信号的传递。）

    +   信号处理程序是在创建管道后安装的，以防止在管道创建之前信号被传递，从而引发竞态条件。

    +   在信号处理程序中使用*write()*是安全的，因为它是表 21-1 中列出的异步信号安全函数之一，参见标准异步信号安全函数。

1.  将 *select()* 调用放入循环中，以便在被信号处理程序中断时重新启动。（以这种方式重新启动并非严格必要；它仅意味着我们可以通过检查 *readfds* 来检查信号是否到达，而不是检查 `EINTR` 错误返回。）

1.  在 *select()* 调用成功完成后，我们可以通过检查管道读端的文件描述符是否设置在 *readfds* 中来判断信号是否到达。

1.  每当信号到达时，读取管道中的所有字节。由于可能会有多个信号到达，使用一个循环，持续读取字节，直到（非阻塞）*read()*由于错误`EAGAIN`而失败。在清空管道后，执行针对信号交付所需的任何操作。

这种技术通常被称为*自管道技巧*，展示该技术的代码可以在示例 63-9 中找到。

这种技巧的变种同样可以在 *poll()* 和 *epoll_wait()* 中使用。

示例 63-9. 使用自管道技巧

```
*from* `altio/self_pipe.c`
static int pfd[2];                      /* File descriptors for pipe */

static void
handler(int sig)
{
    int savedErrno;                     /* In case we change 'errno' */

    savedErrno = errno;
    if (write(pfd[1], "x", 1) == -1 && errno != EAGAIN)
        errExit("write");
    errno = savedErrno;
}

int
main(int argc, char *argv[])
{
    fd_set readfds;
    int ready, nfds, flags;
    struct timeval timeout;
    struct timeval *pto;
    struct sigaction sa;
    char ch;

    /* ... Initialize 'timeout', 'readfds', and 'nfds' for select() */

    if (pipe(pfd) == -1)
        errExit("pipe");

    FD_SET(pfd[0], &readfds);           /* Add read end of pipe to 'readfds' */
    nfds = max(nfds, pfd[0] + 1);       /* And adjust 'nfds' if required */

    flags = fcntl(pfd[0], F_GETFL);
    if (flags == -1)
        errExit("fcntl-F_GETFL");
    flags |= O_NONBLOCK;                /* Make read end nonblocking */
    if (fcntl(pfd[0], F_SETFL, flags) == -1)
        errExit("fcntl-F_SETFL");

    flags = fcntl(pfd[1], F_GETFL);
    if (flags == -1)
        errExit("fcntl-F_GETFL");
    flags |= O_NONBLOCK;                /* Make write end nonblocking */
    if (fcntl(pfd[1], F_SETFL, flags) == -1)
        errExit("fcntl-F_SETFL");

    sigemptyset(&sa.sa_mask);
    sa.sa_flags = SA_RESTART;           /* Restart interrupted read()s */
    sa.sa_handler = handler;
    if (sigaction(SIGINT, &sa, NULL) == -1)
        errExit("sigaction");

    while ((ready = select(nfds, &readfds, NULL, NULL, pto)) == -1 &&
            errno == EINTR)
        continue;                       /* Restart if interrupted by signal */
    if (ready == -1)                    /* Unexpected error */
        errExit("select");

    if (FD_ISSET(pfd[0], &readfds)) {   /* Handler was called */
        printf("A signal was caught\n");

        for (;;) {                      /* Consume bytes from pipe */
            if (read(pfd[0], &ch, 1) == -1) {
                if (errno == EAGAIN)
                    break;              /* No more bytes */
                else
                    errExit("read");    /* Some other error */
            }

            /* Perform any actions that should be taken in response to signal */
        }
    }

    /* Examine file descriptor sets returned by select() to see
       which other file descriptors are ready */

}
      *from* `altio/self_pipe.c`
```

## 总结

本章中，我们探讨了执行 I/O 的标准模型的各种替代方法：I/O 多路复用（*select()* 和 *poll()*）、信号驱动的 I/O 和 Linux 特有的 *epoll* API。所有这些机制都允许我们监视多个文件描述符，查看是否可以在其中任何一个上进行 I/O。没有一个机制实际上执行 I/O。相反，一旦我们确定某个文件描述符已准备好，我们就使用传统的 I/O 系统调用来执行 I/O。

*select()* 和 *poll()* I/O 多路复用调用同时监视多个文件描述符，查看是否可以在任何一个描述符上进行 I/O。使用这两个系统调用时，我们将待检查的文件描述符完整列表传递给内核，内核返回一个修改过的列表，指示哪些描述符已准备好。由于每次调用都传递并检查完整的文件描述符列表，这意味着在监视大量文件描述符时，*select()* 和 *poll()* 的性能较差。

信号驱动的 I/O 允许进程在文件描述符上可以进行 I/O 时接收信号。要启用信号驱动的 I/O，我们必须为 `SIGIO` 信号建立处理程序，设置接收信号的所有者进程，并通过设置 `O_ASYNC` 打开文件状态标志来启用信号生成。与 I/O 多路复用相比，这种机制在监视大量文件描述符时提供了显著的性能优势。Linux 允许我们更改用于通知的信号，如果我们使用实时信号，则可以将多个通知排队，信号处理程序可以使用其 *siginfo_t* 参数来确定生成信号的文件描述符和事件类型。

类似于信号驱动的 I/O，*epoll* 在监视大量文件描述符时提供了优越的性能。*epoll*（以及信号驱动的 I/O）的性能优势来自于内核“记住”了进程正在监视的文件描述符列表（与*select()*和*poll()*不同，每次系统调用都必须再次告诉内核检查哪些文件描述符）。*epoll* API 相较于使用信号驱动的 I/O 有一些显著的优势：我们避免了处理信号的复杂性，并且可以指定需要监视的 I/O 事件类型（例如，输入或输出）。

在本章过程中，我们区分了水平触发和边缘触发的就绪通知。使用水平触发通知模型时，我们会被告知某个文件描述符是否可以进行 I/O 操作。相反，边缘触发通知则告诉我们自上次监视以来某个文件描述符是否发生了 I/O 活动。I/O 多路复用系统调用提供了水平触发通知模型；信号驱动的 I/O 类似于边缘触发模型；而*epoll*能够在这两种模型下工作（默认是水平触发）。边缘触发通知通常与非阻塞 I/O 一起使用。

本章最后，我们讨论了一个程序可能面临的问题，即在监视多个文件描述符的同时，如何等待信号的传递。解决这个问题的常见方法是所谓的自管道技巧，其中信号的处理程序向一个管道写入一个字节，该管道的读取端被包含在被监视的文件描述符集合中。SUSv3 指定了*pselect()*，它是*select()*的一个变种，提供了该问题的另一种解决方案。然而，*pselect()*并非所有 UNIX 实现中都可用。Linux 还提供了类似的（但非标准的）*ppoll()*和*epoll_pwait()*。

#### 进一步的信息

[Stevens 等人，2004] 描述了 I/O 多路复用和信号驱动的 I/O，特别强调了这些机制与套接字的结合使用。[Gammo 等人，2004] 是一篇比较*select()*、*poll()* 和 *epoll* 性能的论文。

一个特别有趣的在线资源是[`www.kegel.com/c10k.html`](http://www.kegel.com/c10k.html)。该网页由 Dan Kegel 编写，名为“C10K 问题”，探讨了为同时服务数万个客户端而设计的 Web 服务器开发者面临的问题。网页包含了大量相关信息的链接。

## 练习

1.  修改示例 63-2 监视多个文件描述符")中的程序`(poll_pipes.c)`，将其改为使用*select()*代替*poll()*。

1.  编写一个*echo*服务器（参见一个迭代的 UDP *echo*服务器和一个并发的 TCP *echo*服务器），能够同时处理 TCP 和 UDP 客户端。为此，服务器必须创建一个监听 TCP 套接字和一个 UDP 套接字，然后使用本章中描述的技术之一来监视这两个套接字。

1.  第 63.5 节指出，*select()*不能同时用于等待信号和文件描述符，并描述了使用信号处理程序和管道的解决方案。当程序需要等待一个文件描述符和一个 System V 消息队列的输入时，也会遇到类似的问题（因为 System V 消息队列不使用文件描述符）。一种解决方案是派生一个独立的子进程，将每条消息从队列复制到父进程监视的文件描述符之一的管道中。编写一个程序，使用此方案和*select()*同时监视来自终端和消息队列的输入。

1.  自管道技巧中关于自管道技巧描述的最后一步提到，程序应首先清空管道，然后再执行任何应对信号的操作。如果这两个子步骤的顺序被颠倒，会发生什么？

1.  修改示例 63-9（`self_pipe.c`）中的程序，改为使用*poll()*而不是*select()*。

1.  编写一个程序，使用*epoll_create()*创建一个*epoll*实例，然后立即使用*epoll_wait()*等待返回的文件描述符。当*epoll_wait()*被传递一个具有空兴趣列表的*epoll*文件描述符时，会发生什么？为什么这可能有用？

1.  假设我们有一个正在监视多个文件描述符的*epoll*文件描述符，这些文件描述符始终处于就绪状态。如果我们执行一系列*epoll_wait()*调用，其中*maxevents*远小于就绪文件描述符的数量（例如，*maxevents*为 1），并且在调用之间没有对就绪的描述符执行任何 I/O 操作，那么*epoll_wait()*在每次调用中返回哪个描述符？编写一个程序来确定答案。（在这个实验中，只需在*epoll_wait()*系统调用之间不执行任何 I/O 操作即可。）这种行为为什么可能有用？

1.  修改示例 63-3（`demo_sigio.c`）中的程序，改为使用实时信号代替`SIGIO`。修改信号处理程序，接受一个*siginfo_t*参数，并显示该结构体中*si_fd*和*si_code*字段的值。
