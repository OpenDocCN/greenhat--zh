## **可扩展性与架构**

![image](img/common01.jpg)

迟早，你的开发过程将不得不考虑弹性和可扩展性。一个应用程序的可扩展性、并发性和并行性在很大程度上取决于其最初的架构和设计。正如我们在本章中所看到的，有些范式——例如多线程——在 Python 中并不适用，而其他技术，例如面向服务的架构，则效果更好。

完整地涵盖可扩展性需要一本书，而事实上已经有很多书对此进行了详细讨论。本章涵盖了可扩展性的基本原则，即使你不打算构建有百万用户的应用程序。

### **Python 中的多线程及其限制**

默认情况下，Python 进程只在一个线程上运行，这个线程被称为*主线程*。这个线程在单个处理器上执行代码。*多线程*是一种编程技术，允许代码通过同时运行多个线程在单个 Python 进程内并发执行。这是我们在 Python 中引入并发的主要机制。如果计算机配备了多个处理器，你甚至可以使用*并行 ism*，在多个处理器上并行运行线程，以加快代码执行速度。

多线程最常用的场景（虽然不总是适用）是：

+   你需要在不停止主线程执行的情况下运行后台或 I/O 导向的任务。例如，图形用户界面的主循环正在忙着等待一个事件（例如用户点击或键盘输入），但代码还需要执行其他任务。

+   你需要将工作负载分配到多个 CPU 上。

第一个场景是多线程的一个很好的通用案例。虽然在这种情况下实现多线程会引入额外的复杂性，但控制多线程是可管理的，除非 CPU 工作负载非常密集，否则性能通常不会受到影响。当工作负载是 I/O 密集型时，使用并发的性能提升更为显著，特别是在 I/O 延迟较高时：你等待读取或写入的时间越长，做其他事情的好处就越大。

在第二种情况下，你可能希望为每个新请求启动一个新线程，而不是逐个处理它们。这看起来像是多线程的一个好用例。然而，如果你像这样分散工作负载，你将会遇到 Python 的*全局解释器锁（GIL）*，它是每次 CPython 执行字节码时必须获得的锁。这个锁意味着在任何时候只有一个线程可以控制 Python 解释器。这个规则最初是为了防止竞争条件而引入的，但不幸的是，它意味着如果你试图通过让应用程序运行多个线程来扩展它，你将始终受到这个全局锁的限制。

因此，虽然使用线程看似是理想的解决方案，但大多数在多个线程中运行请求的应用程序难以达到 150% 的 CPU 使用率，或者相当于 1.5 个核心的使用率。大多数计算机有 4 或 8 个核心，服务器则提供 24 或 48 个核心，但 GIL 阻止了 Python 使用全部的 CPU。目前有一些移除 GIL 的计划，但这个努力非常复杂，因为它需要在性能和向后兼容性之间做出权衡。

尽管 CPython 是 Python 语言中最常用的实现，但也有其他实现没有 GIL。例如，*Jython* 可以高效地并行运行多个线程。不幸的是，像 Jython 这样的项目由于其自身的性质总是滞后于 CPython，因此并不是真正有用的目标；创新发生在 CPython 中，其他实现只是在追随 CPython 的步伐。

所以，让我们回顾一下现在所知道的两个用例，并找出一个更好的解决方案：

+   当你需要运行后台任务时，*可以*使用多线程，但更简单的解决方案是围绕事件循环构建应用程序。有很多 Python 模块可以实现这一点，现在的标准是`asyncio`。还有一些框架，如 Twisted，也是围绕这个概念构建的。最先进的框架会让你通过信号、定时器和文件描述符活动来访问事件——我们将在本章的“事件驱动架构”中讨论这个问题，详见第 181 页。

+   当你需要分担工作负载时，使用多个进程是最有效的方法。我们将在下一节中探讨这种技术。

开发人员在使用多线程时应该三思而后行。例如，我曾经在几年前使用多线程来分配任务，在我写的 Debian 构建守护进程 rebuildd 中。虽然为每个正在运行的构建任务分配一个线程看起来很方便，但我很快就陷入了 Python 中线程并行性的陷阱。如果有机会重新开始，我会基于异步事件处理或多进程来构建，而不必担心 GIL。

多线程是复杂的，正确实现多线程应用程序很困难。你需要处理线程同步和锁定，这意味着有很多引入 bug 的机会。考虑到总体收益较小，最好在投入太多精力之前再三考虑。

### **多进程与多线程**

由于 GIL 阻止了多线程成为一个良好的可扩展性解决方案，因此可以考虑使用 Python 的*多进程*包提供的替代方案。该包提供了与多线程模块相同类型的接口，不同之处在于它启动的是新的*进程*（通过 `os.fork()`），而不是新的系统线程。

示例 11-1 展示了一个简单的示例，其中一百万个随机整数被求和八次，这个活动在八个线程中同时进行。

```py
import random
import threading
results = []
def compute():
    results.append(sum(
        [random.randint(1, 100) for i in range(1000000)]))
workers = [threading.Thread(target=compute) for x in range(8)] for worker in workers:
    worker.start()
for worker in workers:
    worker.join()
print("Results: %s" % results)
```

*示例 11-1：使用多线程实现并发活动*

在示例 11-1 中，我们使用`threading.Thread`类创建了八个线程，并将它们存储在`workers`数组中。这些线程将执行`compute()`函数。然后，它们使用`start()`方法开始执行。`join()`方法仅在线程执行完成后才会返回。此时，结果可以被打印出来。

运行此程序会返回以下结果：

```py
$ time python worker.py
Results: [50517927, 50496846, 50494093, 50503078, 50512047, 50482863,
50543387, 50511493]
python worker.py  13.04s user 2.11s system 129% cpu 11.662 total
```

这是在一个空闲的四核 CPU 上运行的，这意味着 Python 最多可能使用 400%的 CPU。然而，这些结果表明，即使有八个线程并行运行，CPU 使用率也明显未能达到这个数值。相反，CPU 使用率达到了 129%，仅占硬件能力的 32%（129/400）。

现在，让我们使用*多进程（multiprocessing）*重写这个实现。对于像这样简单的情况，切换到多进程非常直接，如示例 11-2 所示。

```py
import multiprocessing
import random

def compute(n):
    return sum(
        [random.randint(1, 100) for i in range(1000000)])

# Start 8 workers
pool = multiprocessing.Pool(processes=8)
print("Results: %s" % pool.map(compute, range(8)))
```

*示例 11-2：使用多进程实现并发活动*

`multiprocessing`模块提供了一个`Pool`对象，它接受一个参数，表示要启动的进程数。它的`map()`方法与原生的`map()`方法作用相同，只是不同的 Python 进程将负责执行`compute()`函数。

在相同条件下运行示例 11-2 程序，与示例 11-1 相比，得到以下结果：

```py
$ time python workermp.py
Results: [50495989, 50566997, 50474532, 50531418, 50522470, 50488087, 0498016, 50537899]
python workermp.py  16.53s user 0.12s system 363% cpu 4.581 total
```

多进程将执行时间减少了 60%。此外，我们已经能够消耗最多 363%的 CPU 功率，这超过了计算机 CPU 容量的 90%以上（363/400）。

每次你认为可以并行化某些工作时，几乎总是更好地依赖多进程，并将任务分叉到多个 CPU 核心上来分担负载。这对于非常短的执行时间不是一个好方案，因为`fork()`调用的成本太高，但对于更大的计算需求，它效果很好。

### **事件驱动架构**

*事件驱动编程*的特点是通过事件（如用户输入）来决定程序的控制流，它是组织程序流程的一个好方法。事件驱动程序会监听队列上发生的各种事件，并根据这些传入的事件作出反应。

假设你想构建一个应用程序，它监听套接字上的连接，然后处理接收到的连接。基本上，有三种方法可以解决这个问题：

+   每次建立新连接时，创建一个新进程，依赖像`multiprocessing`模块这样的工具。

+   每当建立一个新连接时，就启动一个新线程，依赖于类似 `threading` 模块的东西。

+   将这个新连接添加到你的事件循环中，并对它发生时将产生的事件做出反应。

确定现代计算机如何同时处理成千上万的连接被称为 *C10K 问题*。C10K 解决方案策略解释了如何使用事件循环来监听数百个事件源，远比每个连接使用一个线程的方式更具可扩展性。这并不意味着这两种技术不兼容，但确实意味着你通常可以用事件驱动机制替代多线程方式。

事件驱动架构使用事件循环：程序调用一个函数，直到接收到事件并准备好处理时才会阻塞执行。这个理念是，你的程序可以在等待输入输出完成时，继续忙于处理其他任务。最基本的事件是“数据准备好读取”和“数据准备好写入”。

在 Unix 中，用于构建此类事件循环的标准函数是系统调用 `select(2)` 或 `poll(2)`。这些函数期望接收一个文件描述符列表来进行监听，它们会在至少有一个文件描述符准备好进行读写操作时返回。

在 Python 中，我们可以通过 `select` 模块访问这些系统调用。利用这些调用构建事件驱动的系统非常简单，尽管这样做可能有点繁琐。Listing 11-3 展示了一个执行我们指定任务的事件驱动系统：监听套接字并处理它接收到的任何连接。

```py
import select
import socket

server = socket.socket(socket.AF_INET,
                       socket.SOCK_STREAM)
# Never block on read/write operations
server.setblocking(0)

# Bind the socket to the port
server.bind(('localhost', 10000))
server.listen(8)

while True:
    # select() returns 3 arrays containing the object (sockets, files...)

    # that are ready to be read, written to or raised an error
inputs,
outputs, excepts = select.select([server], [], [server])
    if server in inputs:
        connection, client_address = server.accept()
        connection.send("hello!\n")
```

*Listing 11-3：一个监听和处理连接的事件驱动程序*

在 Listing 11-3 中，创建了一个服务器套接字并将其设置为 *非阻塞*，意味着在该套接字上执行的任何读写操作都不会阻塞程序。如果程序在没有数据可读时尝试从套接字读取，套接字的 `recv()` 方法会引发 OSError，指示套接字尚未准备好。如果我们没有调用 `setblocking(0)`，套接字将保持在阻塞模式，而不会引发错误，这不是我们希望的结果。然后，套接字会绑定到一个端口，并以最多八个连接的回退队列进行监听。

主循环是通过 `select()` 构建的，它接收我们希望读取的文件描述符列表（此例中为套接字），我们希望写入的文件描述符列表（此例中没有），以及我们希望获取异常的文件描述符列表（此例中为套接字）。`select()` 函数会在其中一个被选中的文件描述符准备好读取、写入或引发异常时返回。返回值是符合请求条件的文件描述符列表。接下来，检查我们的套接字是否在准备读取的列表中，如果是，接受连接并发送消息。

### **其他选项与 asyncio**

另外，也有许多框架，如 Twisted 或 Tornado，它们提供了更为集成的此类功能；Twisted 在这方面多年来一直是事实上的标准。导出 Python 接口的 C 库，如 `libevent`、`libev` 或 `libuv`，也提供了非常高效的事件循环。

这些选项都解决了相同的问题。缺点是，尽管有多种选择，但大多数选项之间并不兼容。许多还采用 *回调机制*，这意味着在阅读代码时程序的流程并不十分清晰；你需要跳转到许多不同的地方才能通读程序。

另一种选择是使用 `gevent` 或 `greenlet` 库，它们避免使用回调。然而，它们的实现细节包括特定于 CPython x86 的代码和在运行时对标准函数的动态修改，这意味着你不愿意长期使用并维护使用这些库的代码。

2012 年，Guido Van Rossum 开始研究名为 *tulip* 的解决方案，并在 PEP 3156 中进行了文档说明（* [`www.python.org/dev/peps/pep-3156`](https://www.python.org/dev/peps/pep-3156) *）。该软件包的目标是提供一个标准的事件循环接口，能够与所有框架和库兼容，并具备互操作性。

此后，tulip 代码被重命名并合并到 Python 3.4 中，成为 `asyncio` 模块，并且现在已成为事实上的标准。并非所有库都与 `asyncio` 兼容，大多数现有的绑定需要重写。

从 Python 3.6 开始，`asyncio` 已经如此完全集成，以至于它拥有了自己的 `await` 和 `async` 关键字，使得使用起来非常简便。清单 11-4 展示了如何使用提供异步 HTTP 绑定的 `aiohttp` 库与 `asyncio` 一起并发运行多个网页检索。

```py
import aiohttp
import asyncio

async def get(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return response

loop = asyncio.get_event_loop()

coroutines = [get("http://example.com") for _ in range(8)]

results = loop.run_until_complete(asyncio.gather(*coroutines))

print("Results: %s" % results)
```

*清单 11-4：使用 aiohttp 并发地获取网页*

我们将 `get()` 函数定义为异步函数，因此它本质上是一个协程。`get()` 函数的两个步骤——连接和页面检索——被定义为异步操作，在准备好之前将控制权交还给调用者。这使得 `asyncio` 可以在任何时刻调度另一个协程。该模块在连接建立或页面准备好读取时会恢复协程的执行。这八个协程同时启动并交给事件循环，由 `asyncio` 负责高效调度它们。

`asyncio` 模块是编写异步代码和利用事件循环的一个优秀框架。它支持文件、套接字等，并且有许多第三方库支持各种协议。不要犹豫，尽管使用它！

### **面向服务的架构**

避开 Python 的扩展性缺陷可能看起来很棘手。然而，Python *确实*非常擅长实现 *面向服务的架构（SOA）*，这是一种软件设计风格，其中不同的组件通过通信协议提供一组服务。例如，OpenStack 在其所有组件中都使用 SOA 架构。这些组件使用 HTTP REST 与外部客户端（最终用户）通信，并且构建在高级消息队列协议（AMQP）之上的抽象远程过程调用（RPC）机制。

在你的开发环境中，了解在这些模块之间使用哪些通信渠道，主要是了解你将与谁进行通信。

当将服务暴露给外部世界时，首选的通道是 HTTP，特别是对于无状态设计，如 REST 风格（REpresentational State Transfer 风格）架构。这种架构使得服务的实现、扩展、部署和理解变得更加容易。

然而，在内部暴露和使用 API 时，HTTP 可能不是最佳协议。还有许多其他通信协议，甚至单独描述一种协议可能就会填满一本书。

在 Python 中，有很多用于构建 RPC 系统的库。Kombu 很有意思，因为它提供了一个基于许多后端的 RPC 机制，其中 AMQ 协议是主要的支持协议。它还支持 Redis、MongoDB、Beanstalk、Amazon SQS、CouchDB 或 ZooKeeper。

最终，通过使用这种松耦合的架构，你可以间接地获得大量的性能提升。如果我们认为每个模块都提供并暴露 API，那么我们可以运行多个守护进程，这些守护进程也可以暴露 API，从而让多个进程——因此也让多个 CPU——来处理工作负载。例如，*Apache httpd* 会创建一个新的 `worker`，该 `worker` 使用一个新的系统进程来处理新的连接；我们可以将连接分配给同一节点上运行的不同 `worker`。为了做到这一点，我们只需要一个系统来将工作分配给我们的各个 `workers`，而这个 API 就提供了这样的功能。每个模块都会是一个不同的 Python 进程，正如我们之前所看到的，这种方法比多线程更适合分散工作负载。你将能够在每个节点上启动多个 `workers`。即使无状态模块不是绝对必要的，在你有选择的情况下，你也应该偏向使用它们。

### **使用 ZeroMQ 的进程间通信**

正如我们刚刚讨论的，构建分布式系统时总是需要一个消息总线。你的进程需要相互通信以传递消息。`ZeroMQ` 是一个可以作为并发框架的套接字库。列表 11-5 实现了与 列表 11-1 中相同的 `worker`，但使用 `ZeroMQ` 作为分配工作和进程间通信的方式。

```py
   import multiprocessing
   import random
   import zmq

   def compute():
       return sum(
           [random.randint(1, 100) for i in range(1000000)])

   def worker():
       context = zmq.Context()
       work_receiver = context.socket(zmq.PULL)
       work_receiver.connect("tcp://0.0.0.0:5555")
       result_sender = context.socket(zmq.PUSH)
       result_sender.connect("tcp://0.0.0.0:5556")
       poller = zmq.Poller()
       poller.register(work_receiver, zmq.POLLIN)

       while True:
           socks = dict(poller.poll())
           if socks.get(work_receiver) == zmq.POLLIN:
               obj = work_receiver.recv_pyobj()
               result_sender.send_pyobj(obj())

   context = zmq.Context()
   # Build a channel to send work to be done
➊ work_sender = context.socket(zmq.PUSH)
   work_sender.bind("tcp://0.0.0.0:5555")
   # Build a channel to receive computed results
➋ result_receiver = context.socket(zmq.PULL)
   result_receiver.bind("tcp://0.0.0.0:5556")
   # Start 8 workers
   processes = []
   for x in range(8):
➌     p = multiprocessing.Process(target=worker)
       p.start()
       processes.append(p)
   # Send 8 jobs
   for x in range(8):
       work_sender.send_pyobj(compute)
   # Read 8 results

   results = []
   for x in range(8):
➍     results.append(result_receiver.recv_pyobj()) # Terminate all processes
   for p in processes:
       p.terminate()
   print("Results: %s" % results)
```

*列表 11-5：使用 ZeroMQ 的工作进程*

我们创建了两个套接字，一个用于发送函数`(work_sender)` ➊，另一个用于接收任务`(result_receiver)` ➋。每个由`multiprocessing.Process`启动的`worker` ➌都会创建自己的套接字，并将其连接到主进程。然后，`worker`执行传递给它的任何函数，并返回结果。主进程只需要通过发送套接字发送八个任务，并等待通过接收套接字返回八个结果 ➍。

正如你所看到的，`ZeroMQ`提供了一种构建通信通道的简便方法。我选择在这里使用 TCP 传输层来说明我们可以通过网络运行这个。需要注意的是，`ZeroMQ`还提供了一种进程间通信通道，它通过使用 Unix 套接字在本地工作（不涉及任何网络层）。显然，为了简洁明了，本例中基于`ZeroMQ`构建的通信协议非常简单，但不难想象在其基础上构建一个更复杂的通信层。同样，也很容易想象构建一个完全分布式的应用程序，通过像 ZeroMQ 或 AMQP 这样的网络消息总线进行通信。

请注意，诸如 HTTP、ZeroMQ 和 AMQP 之类的协议是语言无关的：你可以使用不同的语言和平台来实现系统的每个部分。虽然我们都认为 Python 是一个很好的语言，但其他团队可能有其他偏好，或者在解决某个问题的某个部分时，其他语言可能是更好的选择。

最终，使用传输总线将应用程序解耦成多个部分是一个不错的选择。通过这种方法，你可以构建可以从一台计算机分发到数千台计算机的同步和异步 API。它不会将你束缚于特定的技术或语言，因此你可以在正确的方向上发展一切。

### **总结**

Python 中的经验法则是仅在 I/O 密集型工作负载下使用线程，并在 CPU 密集型工作负载出现时尽早切换到多进程。将工作负载分布到更广泛的范围——例如在网络上构建分布式系统——需要外部库和协议。Python 支持这些内容，但它们是由外部提供的。
