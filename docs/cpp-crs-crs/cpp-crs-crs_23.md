## 第二十三章：NETWORK PROGRAMMING WITH BOOST ASIO**

*任何在使用电脑时迷失时间的人，都知道那种做梦的倾向、实现梦想的冲动，以及错过午餐的习惯。*

—蒂姆·伯纳斯-李*

![Image](img/common.jpg)

Boost Asio 是一个用于低级 I/O 编程的库。在本章中，你将了解 Boost Asio 的基本网络功能，它使程序能够轻松高效地与网络资源进行交互。不幸的是，从 C++17 开始，标准库中并没有包含网络编程库。因此，Boost Asio 在许多具有网络组件的 C++ 程序中发挥着核心作用。

尽管 Boost Asio 是 C++ 开发者在想要将跨平台、高性能 I/O 融入程序时的主要选择，但它是一个出了名的复杂库。这种复杂性与对低级网络编程的不熟悉相结合，可能会让新手感到过于压倒。如果你觉得本章晦涩难懂，或者如果你不需要关于网络编程的信息，你可以跳过这一章。

**注意**

*Boost Asio 还包含用于与串口、流和一些操作系统特定对象进行 I/O 的功能。事实上，这个名称来源于“异步 I/O”这个短语。欲了解更多信息，请参阅 Boost Asio 文档。*

### Boost Asio 编程模型

在 Boost 编程模型中，一个 *I/O 上下文对象* 抽象了处理异步数据处理的操作系统接口。这个对象是 *I/O 对象* 的注册表，I/O 对象会发起异步操作。每个对象都知道其对应的服务，而上下文对象则在其中进行调解。

**注意**

*所有 Boost Asio 类都出现在 <boost/asio.hpp> 方便的头文件中。*

Boost Asio 定义了一个单一的服务对象，`boost::asio::io_context`。它的构造函数接受一个可选的整数参数，称为*并发提示*，它表示 `io_context` 应该允许并发运行的线程数量。例如，在一台八核机器上，你可以按如下方式构造一个 `io_context`：

```
boost::asio::io_context io_context{ 8 };
```

你将把相同的 `io_context` 对象传递给你的 I/O 对象的构造函数。一旦你设置好所有 I/O 对象，你将调用 `io_context` 上的 `run` 方法，它会阻塞直到所有待处理的 I/O 操作完成。

最简单的 I/O 对象之一是 `boost::asio::steady_timer`，你可以用它来安排任务。它的构造函数接受一个 `io_context` 对象和一个可选的 `std::chrono::time_point` 或 `std::chrono_duration`。例如，下面的代码构造了一个三秒钟后过期的 `steady_timer`：

```
boost::asio::steady_timer timer{
  io_context, std::chrono::steady_clock::now() + std::chrono::seconds{ 3 }
};
```

您可以使用阻塞或非阻塞调用等待定时器。要阻塞当前线程，您使用定时器的`wait`方法。其结果与使用“Chrono”中学到的`std::this_thread::sleep_for`基本相似，您可以在第 387 页找到相关内容。要进行异步等待，您使用定时器的`async_wait`方法。这接受一个称为*回调*的函数对象。操作系统将在线程唤醒时调用该函数对象。由于现代操作系统带来的复杂性，这可能是由于定时器到期或其他原因。

一旦定时器到期，您可以创建另一个定时器，如果您想进行额外的等待。如果您等待一个已经到期的定时器，它将立即返回。这可能不是您打算做的事情，所以确保只在未到期的定时器上等待。

要检查定时器是否已经到期，函数对象必须接受一个`boost::system::error_code`。`error_code`类是一个表示操作系统特定错误的简单类。它隐式转换为`bool`（如果表示错误条件则为`true`；否则为`false`）。如果回调的`error_code`评估为`false`，则定时器已经到期。

一旦您使用`async_wait`排队了一个异步操作，您将在您的`io_context`对象上调用`run`方法，因为此方法会阻塞，直到所有异步操作完成。

清单 20-1 演示了如何构建和使用用于阻塞和非阻塞等待的定时器。

```
#include <iostream>
#include <boost/asio.hpp>
#include <chrono>

boost::asio::steady_timer make_timer(boost::asio::io_context& io_context) { ➊
  return boost::asio::steady_timer{
          io_context,
          std::chrono::steady_clock::now() + std::chrono::seconds{ 3 }
  };
}

int main() {
  boost::asio::io_context io_context; ➋

  auto timer1 = make_timer(io_context); ➌
  std::cout << "entering steady_timer::wait\n";
  timer1.wait(); ➍
  std::cout << "exited steady_timer::wait\n";

  auto timer2 = make_timer(io_context); ➎
  std::cout << "entering steady_timer::async_wait\n";
  timer2.async_wait([] (const boost::system::error_code& error) { ➏
    if (!error) std::cout << "<<callback function>>\n";
  });
  std::cout << "exited steady_timer::async_wait\n";
  std::cout << "entering io_context::run\n";
  io_context.run(); ➐
  std::cout << "exited io_context::run\n";
}
-----------------------------------------------------------------------
entering steady_timer::wait
exited steady_timer::wait
entering steady_timer::async_wait
exited steady_timer::async_wait
entering io_context::run
<<callback function>>
exited io_context::run
```

*清单 20-1：使用`boost::asio::steady_timer`进行同步和异步等待的程序*

您定义`make_timer`函数来构建在三秒后到期的`steady_timer`。在`main`中，您初始化程序的`io_context`，并从`make_timer`构造第一个定时器。当您在此定时器上调用`wait`时，线程将在三秒后继续。接下来，您使用`make_timer`构造另一个定时器，然后使用在定时器到期时打印`<<callback_function>>`的 lambda 调用`async_wait`。最后，您在您的`io_context`上调用`run`以开始处理操作。

### 使用 Asio 进行网络编程

Boost Asio 包含用于在几个重要网络协议上执行基于网络的 I/O 的设施。现在您已经了解了`io_context`的基本用法以及如何排队异步 I/O 操作，您可以探索如何执行更复杂的 I/O 操作。在本节中，您将扩展对等待定时器的了解，并使用 Boost Asio 的网络 I/O 设施。通过本章结束时，您将知道如何构建可以在网络上通信的程序。

#### *互联网协议套件*

**互联网协议（IP）**是跨网络传输数据的主要协议。每个参与者在 IP 网络中被称为*主机*，每个主机都会获得一个 IP 地址用于标识自己。IP 地址有两种版本：IPv4 和 IPv6。IPv4 地址是 32 位，IPv6 地址是 128 位。

**互联网控制消息协议（ICMP）**被网络设备用于发送支持 IP 网络运行的信息。ping 和 traceroute 程序使用 ICMP 消息来查询网络。通常，最终用户应用程序不需要直接与 ICMP 交互。

要在 IP 网络中发送数据，通常使用传输控制协议（TCP）或用户数据报协议（UDP）。一般来说，当你需要确保数据到达目的地时，使用 TCP；当你需要确保数据快速传输时，使用 UDP。TCP 是一个面向连接的协议，接收方会确认它已收到目标消息。UDP 是一个简单的无连接协议，没有内建的可靠性。

**注意**

*你可能会想知道在 TCP/UDP 的上下文中，“连接”是什么意思，或者觉得“无连接”协议似乎很荒谬。这里的连接指的是在网络中的两个参与者之间建立一个通道，以保证消息的传输和顺序。这些参与者通过握手建立连接，并且有一种机制相互通知，表示它们想要关闭连接。而在无连接协议中，参与者直接向另一个参与者发送数据包，而不先建立通道。*

使用 TCP 和 UDP 时，网络设备通过*端口*彼此连接。端口是一个范围从 0 到 65,535（2 字节）的整数，指定在特定网络设备上运行的某个服务。通过这种方式，一台设备可以运行多个服务，每个服务可以单独寻址。当一台设备（称为*客户端*）与另一台设备（称为*服务器*）建立通信时，客户端指定它想连接的端口。当你将设备的 IP 地址与端口号配对时，结果就叫做*套接字*。

例如，一个 IP 地址为 10.10.10.100 的设备可以通过将一个 Web 服务器应用程序绑定到端口 80 来提供网页。这会在 10.10.10.100:80 上创建一个服务器套接字。接着，一个 IP 地址为 10.10.10.200 的设备启动一个 Web 浏览器，打开一个“随机高端口”，例如 55123。这会在 10.10.10.200:55123 上创建一个客户端套接字。然后，客户端通过在客户端套接字和服务器套接字之间创建 TCP 连接来连接到服务器。同时，其他许多进程可能在任何一台或两台设备上运行，并且有许多其他网络连接同时存在。

互联网分配号码管理局（IANA）维护着一个分配号码的列表，用于标准化某些类型的服务所使用的端口（该列表可以在 *[`www.iana.org/`](https://www.iana.org/)* 上找到）。表 20-1 提供了这个列表中的一些常用协议。

**表 20-1：** IANA 分配的知名协议

| **端口** | **TCP** | **UDP** | **关键词** | **描述** |
| --- | --- | --- | --- | --- |
| 7 | ✓ | ✓ | echo | 回显协议 |
| 13 | ✓ | ✓ | daytime | 日间协议 |
| 21 | ✓ |  | ftp | 文件传输协议 |
| 22 | ✓ |  | ssh | 安全外壳协议 |
| 23 | ✓ |  | telnet | Telnet 协议 |
| 25 | ✓ |  | smtp | 简单邮件传输协议 |
| 53 | ✓ | ✓ | domain | 域名系统 |
| 80 | ✓ |  | http | 超文本传输协议 |
| 110 | ✓ |  | pop3 | 邮局协议 |
| 123 |  | ✓ | ntp | 网络时间协议 |
| 143 | ✓ |  | imap | 互联网邮件访问协议 |
| 179 | ✓ |  | bgp | 边界网关协议 |
| 194 | ✓ |  | irc | 互联网中继聊天 |
| 443 | ✓ |  | https | 超文本传输协议（安全） |

Boost Asio 支持通过 ICMP、TCP 和 UDP 进行网络 I/O。为了简洁起见，本章仅讨论 TCP，因为这三种协议中所涉及的 Asio 类非常相似。

**注意**

*如果你不熟悉网络协议，*Charles M. Kozierok 的《TCP/IP 指南》是一本权威的参考书。*

#### *主机名解析*

当客户端想要连接到服务器时，它需要服务器的 IP 地址。在某些情况下，客户端可能已经有了这个信息。而在其他情况下，客户端可能只有一个服务名称。将服务名称转换为 IP 地址的过程称为 *主机名解析*。Boost Asio 包含 `boost::asio::ip::tcp::resolver` 类来执行主机名解析。要构造解析器，你只需要传递一个 `io_context` 实例作为唯一的构造参数，示例如下：

```
boost::asio::ip::tcp::resolver my_resolver{ my_io_context };
```

要执行主机名解析，你可以使用 `resolve` 方法，该方法接受至少两个 `string_view` 类型的参数：主机名和服务。你可以为服务提供一个关键字或端口号（有关一些示例关键字，请参阅 表 20-1）。`resolve` 方法返回一组 `boost::asio::ip::tcp::resolver::basic_resolver_entry` 对象，这些对象提供了几个有用的方法：

+   `endpoint` 获取 IP 地址和端口。

+   `host_name` 获取主机名。

+   `service_name` 获取与该端口关联的服务名称。

如果解析失败，`resolve` 会抛出一个 `boost::system::system_error`。或者，你可以传递一个 `boost::system::error_code` 引用，代替抛出异常，将错误信息传递给它。例如，示例 20-2 使用 Boost Asio 确定 No Starch Press 网站服务器的 IP 地址和端口。

```
#include <iostream>
#include <boost/asio.hpp>

int main() {
  boost::asio::io_context io_context; ➊
  boost::asio::ip::tcp::resolver resolver{ io_context }; ➋
  boost::system::error_code ec;
  for(auto&& result : resolver.resolve("www.nostarch.com", "http", ec)) { ➌
    std::cout << result.service_name() << " " ➍
              << result.host_name() << " " ➎
              << result.endpoint() ➏
              << std::endl;
  }
  if(ec) std::cout << "Error code: " << ec << std::endl; ➐
}
-----------------------------------------------------------------------
http [www.nostarch.com](http://www.nostarch.com) 104.20.209.3:80
http [www.nostarch.com](http://www.nostarch.com) 104.20.208.3:80
```

*示例 20-2：使用 Boost Asio 阻塞主机名解析*

**注意**

*你的结果可能会根据 No Starch Press 网站服务器在 IP 地址空间中的位置而有所不同。*

你初始化一个`io_context` ➊和一个`boost::asio::ip::tcp::resolver` ➋。在基于范围的`for`循环内，你迭代每个`result` ➌并提取`service_name` ➍、`host_name` ➎和`endpoint` ➏。如果`resolve`遇到错误，你将其打印到标准输出 ➐。

你可以使用`async_resolve`方法执行异步主机名解析。与`resolve`一样，你将主机名和服务作为前两个参数传递。此外，你提供一个回调函数对象，接受两个参数：`system_error_code`和一个`basic_resolver_entry`对象的范围。清单 20-3 展示了如何将清单 20-2 重构为使用异步主机名解析。

```
#include <iostream>
#include <boost/asio.hpp>

int main() {
  boost::asio::io_context io_context;
  boost::asio::ip::tcp::resolver resolver{ io_context };
  resolver.async_resolve("www.nostarch.com", "http", ➊
    [](boost::system::error_code ec, const auto& results) { ➋
      if (ec) { ➌
        std::cerr << "Error:" << ec << std::endl;
        return; ➍
      }
      for (auto&& result : results) { ➎
        std::cout << result.service_name() << " "
                  << result.host_name() << " "
                  << result.endpoint() << " "
                  << std::endl; ➏
      }
    }
  );
  io_context.run(); ➐
}
-----------------------------------------------------------------------
http [www.nostarch.com](http://www.nostarch.com) 104.20.209.3:80
http [www.nostarch.com](http://www.nostarch.com) 104.20.208.3:80
```

*清单 20-3：重构清单 20-2 以使用`async_resolve`*

设置与清单 20-2 相同，直到你在解析器上调用`async_resolve` ➊。你传递与之前相同的主机名和服务，但你添加了一个回调参数，该参数接受必需的参数 ➋。在回调 lambda 的主体中，你检查是否存在错误条件 ➌。若存在错误，你打印一个友好的错误信息并`return` ➍。在没有错误的情况下，你像之前一样迭代结果 ➎，打印`service_name`、`host_name`和`endpoint` ➏。与定时器一样，你需要在`io_context`上调用`run`，以便让异步操作有机会完成 ➐。

#### *连接中*

一旦通过主机名解析或自行构建的方式获取到端点范围，你就准备好进行连接了。

首先，你需要一个`boost::asio::ip::tcp::socket`，它是一个抽象操作系统底层套接字的类，用于在 Asio 中使用。套接字接受一个`io_context`作为参数。

第二步，你需要调用`boost::asio::connect`函数，该函数接受一个表示你想连接的端点的`socket`作为第一个参数，接受一个`endpoint`范围作为第二个参数。你可以提供一个`error_code`引用作为可选的第三个参数；否则，`connect`会在出现错误时抛出`system_error`异常。如果成功，`connect`会返回一个单一的`endpoint`，即成功连接的输入范围中的`endpoint`。此时，`socket`对象表示系统环境中的一个真实套接字。

清单 20-4 展示了如何连接到 No Starch Press 的网络服务器。

```
#include <iostream>
#include <boost/asio.hpp>

int main() {
  boost::asio::io_context io_context;
  boost::asio::ip::tcp::resolver resolver{ io_context }; ➊
  boost::asio::ip::tcp::socket socket{ io_context }; ➊
  try  {
    auto endpoints = resolver.resolve("www.nostarch.com", "http"); ➌
    const auto connected_endpoint = boost::asio::connect(socket, endpoints); ➍
    std::cout << connected_endpoint; ➎
  } catch(boost::system::system_error& se) {
    std::cerr << "Error: " << se.what() << std::endl; ➏
  }
}
-----------------------------------------------------------------------
104.20.209.3:80 ➎
```

*清单 20-4：连接到 No Starch 网站服务器*

你构建一个`resolver` ➊，如同在 Listing 20-3 中所示。此外，你使用相同的`io_context`初始化一个`socket` ➋。接下来，你调用`resolve`方法以获取与*【www.nostarch.com】(http://www.nostarch.com)*在端口 80 上关联的每个`endpoint` ➌。回想一下，每个`endpoint`都是一个 IP 地址和与所解析的主机对应的端口。在这种情况下，`resolve`使用域名系统确定*【www.nostarch.com】(http://www.nostarch.com)*在端口 80 上的 IP 地址是 104.20.209.3。然后，你使用`socket`和`endpoint`调用`connect` ➍，它返回`connect`成功连接的`endpoint` ➎。如果发生错误，`resolve`或`connect`将抛出异常，你将捕获该异常并将其打印到 stderr ➏。

你也可以使用`boost::asio::async_connect`进行异步连接，它接受与`connect`相同的两个参数：一个`socket`和一个`endpoint`范围。第三个参数是一个函数对象，充当回调，它必须接受一个`error_code`作为第一个参数，`endpoint`作为第二个参数。Listing 20-5 展示了如何进行异步连接。

```
#include <iostream>
#include <boost/asio.hpp>

int main() {
  boost::asio::io_context io_context;
 boost::asio::ip::tcp::resolver resolver{ io_context };
  boost::asio::ip::tcp::socket socket{ io_context };
  boost::asio::async_connect(socket, ➊
    resolver.resolve("www.nostarch.com", "http"), ➋
    [] (boost::system::error_code ec, const auto& endpoint){ ➌
      std::cout << endpoint; ➍
  });
  io_context.run(); ➎
}
-----------------------------------------------------------------------
104.20.209.3:80 ➍
```

*Listing 20-5: 异步连接到 No Starch web 服务器*

配置与 Listing 20-4 中的完全相同，只不过你将`connect`替换为`async_connect`，并传入相同的第一个➊和第二个➋参数。第三个参数是你的回调函数对象➌，在其中你将`endpoint`打印到 stdout ➍。像所有异步 Asio 程序一样，你需要对`io_context`调用`run` ➎。

#### *缓冲区*

Boost Asio 提供了几个缓冲区类。*缓冲区*（或*数据缓冲区*）是存储临时数据的内存。Boost Asio 缓冲区类形成了所有 I/O 操作的接口。在你进行任何网络连接操作之前，你需要一个用于读取和写入数据的接口。为此，你只需要三种缓冲区类型：

+   `boost::asio::const_buffer`持有一个缓冲区，一旦构造完成，就无法修改。

+   `boost::asio::mutable_buffer`持有一个可以在构造后修改的缓冲区。

+   `boost::asio::streambuf`持有一个基于`std::streambuf`的自动可调整大小的缓冲区。

所有三个缓冲区类提供了两个重要方法来访问其底层数据：`data`和`size`。

`mutable_buffer`和`const_buffer`类的`data`方法返回指向底层数据序列中第一个元素的指针，而它们的`size`方法返回该序列中元素的数量。这些元素是连续的。两个缓冲区都提供默认构造函数，初始化为空缓冲区，正如 Listing 20-6 所示。

```
#include <boost/asio.hpp>

TEST_CASE("const_buffer default constructor") {
  boost::asio::const_buffer cb; ➊
  REQUIRE(cb.size() == 0); ➋
}

TEST_CASE("mutable_buffer default constructor") {
 boost::asio::mutable_buffer mb; ➌
  REQUIRE(mb.size() == 0); ➍
}
```

*Listing 20-6: 默认构造`const_buffer`和`mutable_buffer`生成空缓冲区。*

使用默认构造函数➊➌，你构建了空的缓冲区，其`size`为零➋➍。

`mutable_buffer` 和 `const_buffer` 都提供接受 `void*` 和 `size_t` 的构造函数，这些构造函数对应于你要封装的数据。请注意，这些构造函数并不拥有指向的内存，因此*你必须确保该内存的存储周期至少与所构造的缓冲区的生命周期一样长*。这是一个设计决策，给你作为 Boost Asio 用户提供最大灵活性。不幸的是，它也可能导致一些棘手的错误。未能正确管理缓冲区及其指向的对象的生命周期将导致未定义行为。

列表 20-7 演示了如何使用基于指针的构造函数构造缓冲区。

```
#include <boost/asio.hpp>
#include <string>

TEST_CASE("const_buffer constructor") {
  boost::asio::const_buffer cb{ "Blessed are the cheesemakers.", 7 }; ➊

  REQUIRE(cb.size() == 7); ➋
  REQUIRE(*static_cast<const char*>(cb.data()) == 'B'); ➌
}

TEST_CASE("mutable_buffer constructor") {
  std::string proposition{ "Charity for an ex-leper?" };
  boost::asio::mutable_buffer mb{ proposition.data(), proposition.size() }; ➍

  REQUIRE(mb.data() == proposition.data()); ➎
  REQUIRE(mb.size() == proposition.size()); ➏
}
```

*列表 20-7：使用基于指针的构造函数构造 `const_buffer` 和 `mutable_buffer`*

在第一次测试中，你使用 C 风格字符串和固定长度 `7` ➊ 来构造一个 `const_buffer`。这个固定长度小于字符串字面量 `Blessed are the cheesemakers.` 的长度，因此这个缓冲区仅引用 `Blessed` 而不是整个字符串。这说明你可以选择数组的一个子集（就像你在“字符串视图”一节中学习的 `std::string_view`，在第 500 页）。得到的缓冲区大小为 `7` ➋，如果你将 `data` 指针转换为 `const char*`，你会发现它指向你的 C 风格字符串中的字符 `B` ➌。

在第二次测试中，你通过在缓冲区的构造函数中调用 `string` 的 `data` 和 `size` 成员来构造一个 `mutable_buffer` ➍。得到的缓冲区的 `data` ➎ 和 `size` ➏ 方法返回与原始 `string` 相同的数据。

`boost::asio::streambuf` 类接受两个可选的构造函数参数：一个 `size_t` 类型的最大大小和一个分配器。默认情况下，最大大小为 `std::numeric_limits<std::size_t>`，而分配器类似于标准库容器的默认分配器。`streambuf` 输入序列的初始大小始终为零，如列表 20-8 所示。

```
#include <boost/asio.hpp>

TEST_CASE("streambuf constructor") {
  boost::asio::streambuf sb; ➊
  REQUIRE(sb.size() == 0); ➋
}
```

*列表 20-8：默认构造 `streambuf`*

你默认构造了一个 `streambuf` ➊，当你调用它的 `size` 方法时，它返回 `0` ➋。

你可以将 `streambuf` 的指针传递给 `std::istream` 或 `std::ostream` 的构造函数。回想一下在“流类”一节中，第 524 页 提到过，这些是 `basic_istream` 和 `basic_ostream` 的特化版本，用于向底层同步或源暴露流操作。列表 20-9 演示了如何使用这些类向 `streambuf` 写入数据并随后读取数据。

```
TEST_CASE("streambuf input/output") {
  boost::asio::streambuf sb; ➊
  std::ostream os{ &sb }; ➋
  os << "Welease Wodger!"; ➌

  std::istream is{ &sb }; ➍
  std::string command; ➎
  is >> command; ➏

  REQUIRE(command == "Welease"); ➐
}
```

*列表 20-9：向 `streambuf` 写入数据并读取数据*

你再次构造一个空的 `streambuf` ➊，并将其地址传递给 `ostream` 的构造函数 ➋。然后，你将字符串 `Welease Wodger!` 写入 `ostream`，这会将字符串写入底层的 `streambuf` ➌。

接下来，你再次使用 `streambuf` 的地址来创建一个 `istream` ➍。然后，你创建一个 `string` ➎ 并将 `istream` 写入该 `string` ➏。回想一下在 第 529 页 中的“基本类型的特殊格式化”部分，该操作将跳过任何前导空格，然后读取接下来的字符串直到下一个空格。这会得到字符串的第一个单词 `Welease` ➐。

Boost Asio 还提供了方便的函数模板 `boost::asio::buffer`，该模板接受一个 `std::array` 或 `std::vector` 的 POD 元素，或者一个 `std::string`。例如，你可以使用以下构造方法来创建一个由 `std::string` 支持的 `mutable_buffer`，如 Listing 20-7 中所示：

```
std::string proposition{ "Charity for an ex-leper?" };
auto mb = boost::asio::buffer(proposition);
```

`buffer` 模板是特化过的，因此如果你提供一个 `const` 参数，它将返回一个 `const_buffer`。换句话说，要将 `proposition` 转换为 `const_buffer`，只需将其设置为 `const` 即可：

```
const std::string proposition{ "Charity for an ex-leper?" };
auto cb = boost::asio::buffer(proposition);
```

你现在已经创建了一个 `const_buffer cb`。

此外，你还可以创建一个动态缓冲区，这是一个由 `std::string` 或 `std::vector` 支持的动态可调整大小的缓冲区。你可以使用 `boost::asio::dynamic_buffer` 函数模板来创建该缓冲区，传入 `string` 或 `vector`，并根据情况返回 `boost::asio::dynamic_string_buffer` 或 `boost::asio::dynamic_vector_buffer`。例如，你可以使用以下构造方法创建一个动态缓冲区：

```
std::string proposition{ "Charity for an ex-leper?" };
auto db = boost::asio::dynamic_buffer(proposition);
```

尽管动态缓冲区是动态可调整大小的，但请记住，`vector` 和 `string` 类使用分配器，而分配操作可能相对较慢。因此，如果你知道要写入缓冲区的数据量，使用非动态缓冲区可能会带来更好的性能。像往常一样，测量和实验将帮助你决定采取哪种方法。

#### *使用缓冲区读取和写入数据*

通过掌握如何使用缓冲区存储和检索数据的知识，你可以学习如何从套接字中提取数据。你可以使用内置的 Boost Asio 函数将数据从活动的 `socket` 对象读取到缓冲区对象中。对于阻塞读取，Boost Asio 提供了三种函数：

+   `boost::asio::read` 尝试读取固定大小的数据块。

+   `boost::asio::read_at` 尝试从一个偏移位置开始读取固定大小的数据块。

+   `boost::asio::read_until` 尝试读取直到分隔符、正则表达式或任意谓词匹配为止。

这三种方法都将 `socket` 作为第一个参数，将缓冲区对象作为第二个参数。其余参数是可选的，具体取决于你使用的是哪种函数：

+   *完成条件* 是一个函数对象，它接受一个 `error_code` 和一个 `size_t` 参数。如果 Asio 函数遇到错误，`error_code` 将被设置，而 `size_t` 参数表示迄今为止已传输的字节数。该函数对象返回一个 `size_t`，对应剩余要传输的字节数，如果操作已完成，则返回 0。

+   *匹配条件* 是一个函数对象，接受由开始和结束迭代器指定的范围。它必须返回一个`std::pair`，其中第一个元素是指示下一个匹配尝试起始点的迭代器，第二个元素是`bool`，表示该范围是否包含匹配项。

+   `boost::system::error_code` 引用，函数将在遇到错误条件时设置此值。

表 20-2 列出了调用读取函数的多种方式。

**表 20-2：** `read`、`read_at` 和 `read_until` 的参数

| **调用** | **描述** |
| --- | --- |
| `read(`s, b, `[`cmp`], [`ec`])` | 从`socket` s 读取一定数量的数据到可变缓冲区 b，依据完成条件 cmp。如果遇到错误条件，则设置`error_code` ec；否则，抛出`system_error`。 |
| `read_at(`s, off, b, `[`cmp`], [`ec`])` | 从`socket` s 开始，按`size_t`偏移量 off，从某个位置读取一定数量的数据到可变缓冲区 b，依据完成条件 cmp。如果遇到错误条件，则设置`error_code` ec；否则，抛出`system_error`。 |
| `read_until(`s, b, x, `[`ec`])` | 从`socket` s 读取数据到可变缓冲区 b，直到满足由 x 表示的条件，x 可以是以下之一：`char`、`string_view`、`boost::regex`，或匹配条件。如果遇到错误条件，则设置`error_code` ec；否则，抛出`system_error`。 |

你也可以从缓冲区向活动的`socket`对象写入数据。对于阻塞式写入，Boost Asio 提供了两个函数：

+   `boost::asio::write` 尝试写入固定大小的数据块。

+   `boost::asio::write_at` 尝试从偏移量开始写入固定大小的数据块。

表 20-3 展示了如何调用这两个方法。它们的参数与读取方法的参数类似。

**表 20-3：** `write` 和 `write_at` 的参数

| **调用** | **描述** |
| --- | --- |
| `write(`s, b, `[`cmp`], [`ec`])` | 从`const`缓冲区 b，将一定数量的数据写入`socket` s，依据完成条件 cmp。如果遇到错误条件，则设置`error_code` ec；否则，抛出`system_error`。 |
| `write_at(`s, off, b, `[`cmp`], [`ec`])` | 从`const`缓冲区 b，按`size_t`偏移量 off 开始，将一定数量的数据写入`socket` s，依据完成条件 cmp。如果遇到错误条件，则设置`error_code` ec；否则，抛出`system_error`。 |

**注意**

*调用读取和写入函数有*很多*种排列方式。在将 Boost Asio 集成到代码中时，务必仔细阅读文档。*

#### *超文本传输协议 (HTTP)*

HTTP 是支撑 web 的 30 年历史的协议。尽管它是一个非常复杂的协议，涉及到网络的使用，但它的普遍性使它成为最相关的选择之一。在接下来的部分中，你将使用 Boost Asio 发出非常简单的 HTTP 请求。并不严格要求你对 HTTP 有扎实的基础，因此你可以在首次阅读时跳过这一部分。不过，这里提供的信息为下一部分中的示例增添了一些背景，并提供了进一步学习的参考资料。

HTTP 会话有两个参与方：客户端和服务器。HTTP 客户端通过 TCP 发送一个纯文本请求，其中包含一行或多行，由回车符和换行符（“CR-LF 换行符”）分隔。

第一行是请求行，其中包含三个标记：HTTP 方法、统一资源定位符（URL）和请求的 HTTP 版本。例如，如果客户端想要获取名为 *index.htm* 的文件，状态行可能是 *GET /index.htm HTTP/1.1*。

请求行之后紧接着的是一个或多个 *头部*，它们定义了 HTTP 事务的参数。每个头部包含一个键和值。键必须由字母数字字符和短横线组成。键和值之间用冒号和空格分隔。CR-LF 换行符标识头部的结束。以下头部在请求中尤为常见：

+   `Host` 指定请求的服务的域名。你可以选择性地包括端口。例如，`Host: [www.google.com](http://www.google.com)` 指定 *[www.google.com](http://www.google.com)* 作为请求服务的主机。

+   `Accept` 指定响应中可接受的媒体类型，以 MIME 格式表示。例如，`Accept: text/plain` 指定请求者可以处理纯文本。

+   `Accept-Language` 指定响应可接受的人类语言。例如，`Accept-Language: en-US` 指定请求者可以处理美式英语。

+   `Accept-Encoding` 指定响应可接受的编码方式。例如，`Accept-Encoding: identity` 指定请求者可以处理没有任何编码的内容。

+   `Connection` 指定当前连接的控制选项。例如，`Connection: close` 指定响应完成后将关闭连接。

你通过额外的 CR-LF 换行符来终止头部。对于某些类型的 HTTP 请求，你还会在头部之后包括一个主体。如果这样做，你还需要包含 `Content-Length` 和 `Content-Type` 头部。`Content-Length` 值指定请求主体的字节长度，而 `Content-Type` 值指定主体的 MIME 格式。

HTTP 响应的第一行是 *状态行*，其中包括响应的 HTTP 版本、状态码和原因短语。例如，状态行 `HTTP/1.1 200 OK` 表示请求成功（“OK”）。状态码始终是三位数字。首位数字表示状态码的类别：

**`1**`（信息性）** 请求已接收。

**`2**`（成功）** 请求已接收并被接受。

**`3**`（重定向）** 需要进一步操作。

**`4**`（客户端错误）** 请求有误。

**`5**`（服务器错误）** 请求似乎没问题，但服务器遇到内部错误。

在状态行之后，响应包含任意数量的头部，格式与请求相同。许多相同的请求头也常见于响应头中。例如，如果 HTTP 响应包含主体，响应头将包括 `Content-Length` 和 `Content-Type`。

如果你需要编写 HTTP 应用程序，绝对应该参考 Boost Beast 库，它提供高性能、低级的 HTTP 和 WebSocket 功能。它建立在 Asio 之上，并与其无缝协作。

**注意**

*有关 HTTP 及其安全性问题的优秀处理，请参考* 《The Tangled Web: A Guide to Securing Modern Web Applications》 *by Michal Zalewski。有关详细内容，请参考互联网工程任务组（IETF）的 RFCs 7230、7231、7232、7233、7234 和 7235。*

#### *实现一个简单的 Boost Asio HTTP 客户端*

在本节中，你将实现一个（非常）简单的 HTTP 客户端。你将构建一个 HTTP 请求，解析端点，连接到 Web 服务器，写入请求并读取响应。Listing 20-10 展示了一种可能的实现方式。

```
#include <boost/asio.hpp>
#include <iostream>
#include <istream>
#include <ostream>
#include <string>

std::string request(std::string host, boost::asio::io_context& io_context) { ➊
  std::stringstream request_stream;
  request_stream << "GET / HTTP/1.1\r\n"
                    "Host: " << host << "\r\n"
                    "Accept: text/html\r\n"
                    "Accept-Language: en-us\r\n"
                    "Accept-Encoding: identity\r\n"
                    "Connection: close\r\n\r\n";
  const auto request = request_stream.str(); ➋
  boost::asio::ip::tcp::resolver resolver{ io_context };
  const auto endpoints = resolver.resolve(host, "http"); ➌
  boost::asio::ip::tcp::socket socket{ io_context };
  const auto connected_endpoint = boost::asio::connect(socket, endpoints); ➍
  boost::asio::write(socket, boost::asio::buffer(request)); ➎
  std::string response;
  boost::system::error_code ec;
  boost::asio::read(socket, boost::asio::dynamic_buffer(response), ec); ➏
  if (ec && ec.value() != 2) throw boost::system::system_error{ ec }; ➐
  return response;
}

int main() {
  boost::asio::io_context io_context;
  try  {
    const auto response = request("www.arcyber.army.mil", io_context); ➑
    std::cout << response << "\n"; ➒
  } catch(boost::system::system_error& se) {
    std::cerr << "Error: " << se.what() << std::endl;
  }
}
-----------------------------------------------------------------------
HTTP/1.1 200 OK
Pragma: no-cache
Content-Type: text/html; charset=utf-8
X-UA-Compatible: IE=edge
pw_value: 3ce3af822980b849665e8c5400e1b45b
Access-Control-Allow-Origin: *
X-Powered-By:
Server:
X-ASPNET-VERSION:
X-FRAME-OPTIONS: SAMEORIGIN
Content-Length: 76199
Cache-Control: private, no-cache
Expires: Mon, 22 Oct 2018 14:21:09 GMT
Date: Mon, 22 Oct 2018 14:21:09 GMT
Connection: close
<!DOCTYPE html>
<html  lang="en-US">
<head id="Head">
--snip--
</body>
</html>
```

*Listing 20-10：完成对美国陆军网络指挥部 Web 服务器的简单请求*

你首先定义一个 `request` 函数，它接受一个 `host` 和一个 `io_context`，并返回一个 HTTP 响应 ➊。首先，你使用 `std::stringstream` 来构建一个包含 HTTP 请求的 `std::string` ➋。接着，你使用 `boost::asio::ip::tcp::resolver` 解析 `host` ➌，并将 `boost::asio::ip::tcp::socket` 连接到结果端点范围 ➍。（这与 Listing 20-4 中的方法相匹配。）

然后，你向你已连接的服务器发送 HTTP 请求。你使用 `boost::asio::write`，传入已连接的 `socket` 和你的 `request`。因为 write 接受 Asio 缓冲区，你使用 `boost::asio::buffer` 从你的请求（它是一个 `std::string`）创建一个 `mutable_buffer` ➎。

接下来，你从服务器读取 HTTP 响应。因为你事先不知道响应的长度，所以你创建一个名为 `response` 的 `std::string` 来接收响应。最终，你将使用它来支持一个动态缓冲区。为了简化，HTTP 请求包含一个 `Connection: close` 头部，它会导致服务器在发送响应后立即关闭连接。这将导致 Asio 返回一个“文件结束”错误代码（值为 2）。因为你预期这种行为，你声明一个 `boost::system::error_code` 来接收该错误。

接下来，你调用`boost::asio::read`，传入已连接的`socket`、一个将接收响应的动态缓冲区以及`error_condition` ➏。你使用`boost::asio_dynamic_buffer`从`response`构造动态缓冲区。`read`返回后，立即检查是否有其他类型的`error_condition`，例如文件结束错误（此时会抛出异常） ➐。否则，返回`response`。

在`main`函数中，你调用`request`函数，传入`www.arcyber.army.mil`主机和一个`io_context`对象 ➑。最后，你将响应打印到标准输出 ➒。

#### *异步读取与写入*

你也可以使用 Boost Asio 进行异步读写。相应的异步函数与它们的阻塞对应函数类似。对于异步读取，Boost Asio 提供了三个函数：

+   `boost::asio::async_read`尝试读取固定大小的数据块。

+   `boost::asio::async_read_at`尝试从一个偏移量开始读取固定大小的数据块。

+   `boost::asio::async_read_until`尝试读取直到遇到分隔符、正则表达式或任意条件为止。

Boost Asio 还提供了两个异步写入函数：

+   `boost::asio::async_write`尝试写入固定大小的数据块。

+   `boost::asio::async_write_at`尝试从一个偏移量开始写入固定大小的数据块。

这五个异步函数接受与它们的阻塞函数相同的参数，唯一不同的是它们的最后一个参数总是一个回调函数对象，该对象接受两个参数：一个`boost::system::error_code`表示函数是否遇到错误，以及一个`size_t`表示传输的字节数。对于异步的`write`函数，你需要判断 Asio 是否写入了整个负载。因为这些调用是异步的，所以你的线程在等待 I/O 完成时不会被阻塞。相反，操作系统会在 I/O 请求的某个部分完成时回调你的线程。

由于回调的第二个参数是一个`size_t`，表示已传输的字节数，你可以通过计算来确定是否还有数据需要写入。如果有，你必须通过传递剩余数据来调用另一个异步写入函数。

示例 20-11 包含了示例 20-10 的一个异步版本。请注意，使用异步函数稍微复杂一些，但它有一个一致的模式，通过回调和处理程序贯穿整个请求的生命周期。

```
#include <boost/asio.hpp>
#include <iostream>
#include <string>
#include <sstream>

using ResolveResult = boost::asio::ip::tcp::resolver::results_type;
using Endpoint = boost::asio::ip::tcp::endpoint;

struct Request {
  explicit Request(boost::asio::io_context& io_context, std::string host)
      : resolver{ io_context },
        socket{ io_context },
        host{ std::move(host) } { ➊
    std::stringstream request_stream;
    request_stream << "GET / HTTP/1.1\r\n"
                      "Host: " << this->host << "\r\n"
                      "Accept: text/plain\r\n"
                      "Accept-Language: en-us\r\n"
                      "Accept-Encoding: identity\r\n"
                      "Connection: close\r\n"
                      "User-Agent: C++ Crash Course Client\r\n\r\n";
    request = request_stream.str(); ➋
    resolver.async_resolve(this->host, "http",
       [this] (boost::system::error_code ec, const ResolveResult& results) {
         resolution_handler(ec, results); ➌
       });
  }
 void resolution_handler(boost::system::error_code ec,
                          const ResolveResult& results) {
    if (ec) { ➍
      std::cerr << "Error resolving " << host << ": " << ec << std::endl;
      return;
    }
    boost::asio::async_connect(socket, results,
            [this] (boost::system::error_code ec, const Endpoint& endpoint){
              connection_handler(ec, endpoint); ➎
            });
  }

  void connection_handler(boost::system::error_code ec,
                          const Endpoint& endpoint) { ➏
    if (ec) {
      std::cerr << "Error connecting to " << host << ": "
                << ec.message() << std::endl;
      return;
    }
    boost::asio::async_write(socket, boost::asio::buffer(request),
            [this] (boost::system::error_code ec, size_t transferred){
              write_handler(ec, transferred);
            });
  }

  void write_handler(boost::system::error_code ec, size_t transferred) { ➐
    if (ec) {
      std::cerr << "Error writing to " << host << ": " << ec.message()
                << std::endl;
    } else if (request.size() != transferred) {
      request.erase(0, transferred);
      boost::asio::async_write(socket, boost::asio::buffer(request),
                               [this] (boost::system::error_code ec,
                                       size_t transferred){
                                 write_handler(ec, transferred);
                               });
    } else {
      boost::asio::async_read(socket, boost::asio::dynamic_buffer(response),
                              [this] (boost::system::error_code ec,
                                      size_t transferred){
                                read_handler(ec, transferred);
                              });
    }
  }

  void read_handler(boost::system::error_code ec, size_t transferred) { ➑
    if (ec && ec.value() != 2)
      std::cerr << "Error reading from " << host << ": "
                << ec.message() << std::endl;
  }

  const std::string& get_response() const noexcept {
    return response;
  }
private:
 boost::asio::ip::tcp::resolver resolver;
  boost::asio::ip::tcp::socket socket;
  std::string request, response;
  const std::string host;
};

int main() {
  boost::asio::io_context io_context;
  Request request{ io_context, "www.arcyber.army.mil" }; ➒
  io_context.run(); ➓
  std::cout << request.get_response();
}
-----------------------------------------------------------------------
HTTP/1.1 200 OK
Pragma: no-cache
Content-Type: text/html; charset=utf-8
X-UA-Compatible: IE=edge
pw_value: 3ce3af822980b849665e8c5400e1b45b
Access-Control-Allow-Origin: *
X-Powered-By:
Server:
X-ASPNET-VERSION:
X-FRAME-OPTIONS: SAMEORIGIN
Content-Length: 76199
Cache-Control: private, no-cache
Expires: Mon, 22 Oct 2018 14:21:09 GMT
Date: Mon, 22 Oct 2018 14:21:09 GMT
Connection: close

<!DOCTYPE html>
<html  lang="en-US">
<head id="Head">
--snip--
</body>
</html>
```

*示例 20-11：一个示例 20-9 的异步重构*

首先你声明一个 `Request` 类来处理 Web 请求。它有一个构造函数，接受一个 `io_context` 和一个包含你要连接的主机的 `string` ➊。就像在 示例 20-9 中一样，你使用 `std::stringstream` 创建一个 HTTP GET 请求，并将结果 `string` 保存在 `request` 字段中 ➋。接下来，你使用 `async_resolve` 请求与所请求的 `host` 对应的端点。在回调函数中，你调用当前 `Request` 的 `resolution_handler` 方法 ➌。

`resolution_handler` 接收来自 `async_resolve` 的回调。它首先检查是否有错误条件，如果发现错误，则将错误输出到 stderr 并返回 ➍。如果 `async_resolve` 没有返回错误，`resolution_handler` 会使用 `results` 变量中包含的端点调用 `async_connect`。它还会传入当前 `Request` 的 `socket` 字段，`async_connect` 将在其中创建连接。最后，它会将一个连接回调作为第三个参数传递。在回调函数中，你调用当前请求的 `connection_handler` 方法 ➎。

`connection_handler` ➏ 的模式与 `resolution_handler` 方法类似。它检查是否存在错误条件，如果有，就将错误输出到 stderr 并返回；否则，它会通过调用 `async_write` 来继续处理请求，`async_write` 接受三个参数：活动的 `socket`、一个可变缓冲区包装的 `request` 和一个回调函数。回调函数将调用当前请求的 `write_handler` 方法。

你在这些处理函数中看到了模式吗？`write_handler` ➐ 会检查是否有错误，然后继续判断整个请求是否已经发送。如果没有，你仍然需要写入一些请求内容，因此你需要相应地调整 `request` 并再次调用 `async_write`。如果 `async_write` 已经将整个请求写入了 `socket`，那么就该读取响应了。为此，你调用 `async_read`，使用你的 `socket`、一个动态缓冲区来包装 `response` 字段，并传入一个回调函数，该函数会在当前请求上调用 `read_handler` 方法。

`read_handler` ➑ 首先检查是否有错误。由于你的请求使用了 `Connection: close` 头部，你预计会遇到文件结束错误（错误码为 2），就像在 示例 20-10 中一样，因此你忽略它。如果遇到其他类型的错误，你会将错误打印到 stderr 并返回。此时，你的请求已经完成。（呼，终于结束了。）

在 `main` 中，你声明了一个 `io_context` 并初始化一个 `Request` 对象，目标是 *[www.arcyber.army.mil](http://www.arcyber.army.mil)* ➒。由于你使用了异步函数，因此你在 `io_context` 上调用 `run` 方法 ➓。当 `io_context` 返回时，你就知道没有异步操作在等待，因此你将当前 `Request` 对象中的响应内容打印到标准输出（stdout）。

#### *服务*

在 Boost Asio 上构建一个服务器本质上与构建客户端类似。为了接受 TCP 连接，你使用 `boost::asio::ip::tcp::acceptor` 类，该类的构造函数唯一的参数是 `boost::asio::io_context` 对象。

使用阻塞方式接受 TCP 连接时，你使用 `acceptor` 对象的 `accept` 方法，该方法接收一个 `boost::asio::ip::tcp::socket` 引用，该引用将保存客户端的套接字，另有一个可选的 `boost::error_code` 引用，用来保存任何发生的错误条件。如果你没有提供 `boost::error_code`，且发生了错误，`accept` 会抛出一个 `boost::system_error` 异常。一旦 `accept` 返回且没有错误，你可以使用传入的 `socket` 来进行读写，使用之前在处理客户端时使用的相同读写方法。

例如，示例 20-12 演示了如何构建一个回显服务器，它接收一条消息并将其大写后发送回客户端。

```
#include <iostream>
#include <string>
#include <boost/asio.hpp>
#include <boost/algorithm/string/case_conv.hpp>

using namespace boost::asio;

void handle(ip::tcp::socket& socket) { ➊
  boost::system::error_code ec;
  std::string message;
  do {
    boost::asio::read_until(socket, dynamic_buffer(message), "\n"); ➋
    boost::algorithm::to_upper(message); ➌
    boost::asio::write(socket, buffer(message), ec); ➍
    if (message == "\n") return; ➎
    message.clear();
  } while(!ec); ➏
}

int main()  {
  try {
    io_context io_context;
    ip::tcp::acceptor acceptor{ io_context,
                                ip::tcp::endpoint(ip::tcp::v4(), 1895) }; ➐
    while (true) {
      ip::tcp::socket socket{ io_context };
      acceptor.accept(socket); ➑
      handle(socket); ➒
    }
  } catch (std::exception& e) {
    std::cerr << e.what() << std::endl;
  }
}
```

*示例 20-12：一个大写回显服务器*

你声明了一个接受 `socket` 引用的 `handle` 函数，该引用对应客户端，并处理来自客户端的消息 ➊。在一个 `do`-`while` 循环中，你从客户端读取一行文本到一个名为 `message` 的 `string` 变量中 ➋，然后使用 示例 15-31 中展示的 `to_upper` 函数将其转换为大写 ➌，并将其写回客户端 ➍。如果客户端发送了一个空行，你会退出 `handle` ➎；否则，如果没有发生错误条件，你会清空消息内容并继续循环 ➏。

在 `main` 中，你初始化了一个 `io_context` 和一个 `acceptor`，使程序绑定到 `localhost:1895` 套接字 ➐。在一个无限循环中，你创建一个 `socket` 并在 `acceptor` 上调用 `accept` ➑。只要没有抛出异常，`socket` 就代表了一个新的客户端，你可以将这个 `socket` 传递给 `handle` 来处理请求 ➒。

**注意**

*在示例 20-12 中，选择监听端口 1895。这个选择在技术上并不重要，只要你电脑上没有其他程序正在使用这个端口。然而，关于如何决定程序监听的端口，有一些指导原则。IANA 维护了一个注册端口的列表，地址是* [`www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.txt`](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.txt) *，你可能想避免使用其中的端口。另外，现代操作系统通常要求程序拥有提升的权限，才能绑定到端口值为 1023 或以下的* 系统端口。* 端口 1024 到 49151 通常不需要提升权限，称为* 用户端口。* 端口 49152 到 65535 是* 动态/私有端口，* 因为这些端口通常不会被 IANA 注册，因此使用它们一般是安全的。*

要与 Listing 20-12 中的服务器交互，你可以使用*GNU Netcat*，这是一个网络工具，允许你创建入站和出站的 TCP 和 UDP 连接，并读写数据。如果你使用的是类 Unix 系统，你可能已经安装了它。如果没有，请访问[*https://nmap.org/ncat/*](https://nmap.org/ncat/)。Listing 20-13 展示了一个连接到大写回显服务器的示例会话。

```
$ ncat localhost 1895 ➊
The 300 ➋
THE 300
This is Blasphemy! ➋
THIS IS BLASPHEMY!
This is madness! ➋
THIS IS MADNESS!
Madness...? ➋
MADNESS...?
This is Sparta! ➋
THIS IS SPARTA!
➌
Ncat: Broken pipe. ➍
```

*Listing 20-13：使用 Netcat 与大写回显服务器交互*

Netcat（`ncat`）需要两个参数：主机和端口 ➊。启动程序后，每次输入的行都会从服务器返回一个大写结果。当你将文本输入到标准输入（stdin）时，Netcat 将其发送到服务器 ➋，服务器将以大写形式响应。当你发送一个空行 ➌时，服务器终止套接字连接，你将看到`Broken pipe` ➍。

要使用异步方式接受连接，可以在`acceptor`上使用`async_accept`方法，该方法接受一个参数：一个回调对象，该对象接受`error_code`和`socket`。如果发生错误，`error_code`将包含错误信息；否则，`socket`代表成功连接的客户端。之后，你可以像在阻塞方式中一样使用这个套接字。

异步连接导向型服务器的常见模式是使用`std::enable_shared_from_this`模板，具体讨论可参见《高级模式》一章中的第 362 页。其思想是为每个连接创建一个会话对象的共享指针。当你在会话对象内注册读取和写入回调时，你会在回调对象中捕获一个指向`this`的共享指针，这样在 I/O 操作等待期间，会话对象依然存活。一旦没有 I/O 操作待处理，会话对象和所有共享指针一起销毁。Listing 20-14 展示了如何使用异步 I/O 重新实现大写回显服务器。

```
#include <iostream>
#include <string>
#include <boost/asio.hpp>
#include <boost/algorithm/string/case_conv.hpp>
#include <memory>
using namespace boost::asio;

struct Session : std::enable_shared_from_this<Session> {
  explicit Session(ip::tcp::socket socket) : socket{ std::move(socket) } { } ➊
  void read() {
    async_read_until(socket, dynamic_buffer(message), '\n', ➋
            [self=shared_from_this()] (boost::system::error_code ec,
                                       std::size_t length) {
              if (ec || self->message == "\n") return; ➌
              boost::algorithm::to_upper(self->message);
              self->write();
            });
  }
  void write() {
    async_write(socket, buffer(message), ➍
                [self=shared_from_this()] (boost::system::error_code ec,
                                           std::size_t length) {
                  if (ec) return; ➎
                  self->message.clear();
                  self->read();
                });
  }
private:
  ip::tcp::socket socket;
  std::string message;
};

void serve(ip::tcp::acceptor& acceptor) {
  acceptor.async_accept(&acceptor {
    serve(acceptor); ➐
    if (ec) return;
    auto session = std::make_shared<Session>(std::move(socket)); ➑
    session->read();
  });
}

int main()  {
  try {
    io_context io_context;
    ip::tcp::acceptor acceptor{ io_context,
                                ip::tcp::endpoint(ip::tcp::v4(), 1895) };
    serve(acceptor);
    io_context.run(); ➒
  } catch (std::exception& e) {
    std::cerr << e.what() << std::endl;
  }
}
```

*Listing 20-14：使用异步版本的 Listing 20-12*

首先，你需要定义一个`Session`类来管理连接。在构造函数中，你将对应连接客户端的`socket`的所有权转移过来，并将其存储为成员 ➊。

接下来，声明一个`read`方法，它会在`socket`上调用`async_read_until`，将数据读取到`dynamic_buffer`中，直到遇到下一个换行符`\n` ➋。回调对象使用`shared_from_this`方法将其捕获为`shared_ptr`。当回调被触发时，函数检查是否存在错误条件或空行，如果是，返回 ➌。否则，回调会将`message`转换为大写，并调用`write`方法。

`write`方法遵循与`read`方法类似的模式。它调用`async_read`，传入`socket`、`message`（现在为大写）和回调函数 ➍。在回调函数内，您检查是否存在错误条件，如果有则立即返回➎。否则，您知道 Asio 成功地将大写的`message`发送到了客户端，因此您调用`clear`方法来准备处理客户端的下一个消息。接着，您调用`read`方法，重新开始这个过程。

接下来，您定义一个接受`acceptor`对象的`serve`函数。在该函数内，您调用`async_accept`方法并传入一个回调函数来处理连接➏。回调函数首先使用`acceptor`再次调用`serve`，这样程序就可以立即处理新的连接➐。这就是使异步处理在服务器端如此强大的秘密所在：您可以同时处理多个连接，因为运行中的线程无需在处理另一个连接之前服务于一个客户端。接下来，您检查是否存在错误条件，如果有则退出；否则，您创建一个拥有新`Session`对象的`shared_ptr` ➑。该`Session`对象将拥有`acceptor`为您设置的`socket`。然后，您在新的`Session`对象上调用`read`方法，由于`shared_from_this`捕获，它会在`shared_ptr`中创建第二个引用。现在一切准备就绪！一旦由于客户端的空行或某些错误条件导致`read`和`write`周期结束，`shared_ptr`引用会归零，`Session`对象将被销毁。

最后，在`main`中，您构造一个`io_context`和一个`acceptor`，与示例 20-12 中的定义相同。然后，您将`acceptor`传递给`serve`函数以开始服务循环，并在`io_context`上调用`run`以启动异步操作的服务➒。

### 多线程 Boost Asio

为了使您的 Boost Asio 程序支持多线程，您可以简单地创建任务，调用`run`方法在您的`io_context`对象上运行。当然，这并不会让您的程序变得安全，所有在“共享与协调”章节中关于第 647 页的警告依然有效。示例 20-15 演示了如何根据示例 20-14 将您的服务器进行多线程处理。

```
#include <iostream>
#include <string>
#include <boost/asio.hpp>
#include <boost/algorithm/string/case_conv.hpp>
#include <memory>
#include <future>
struct Session : std::enable_shared_from_this<Session> {
--snip--
};

void serve(ip::tcp::acceptor& acceptor) {
--snip--
}

int main()  {
  const int n_threads{ 4 };
  boost::asio::io_context io_context{ n_threads };
  ip::tcp::acceptor acceptor{ io_context,
                              ip::tcp::endpoint(ip::tcp::v4(), 1895) }; ➊
  serve(acceptor); ➋

  std::vector<std::future<void>> futures;
  std::generate_n(std::back_inserter(futures), n_threads, ➌
                  [&io_context] {
                    return std::async(std::launch::async,
                                      [&io_context] { io_context.run(); }); ➍
                  });

  for(auto& future : futures) { ➎
    try {
      future.get(); ➏
    } catch (const std::exception& e) {
      std::cerr << e.what() << std::endl;
    }
  }
}
```

*示例 20-15：为您的异步回声服务器启用多线程*

您的`Session`和`serve`定义是相同的。在`main`中，您声明`n_threads`常量，表示您将用于服务的线程数，一个`io_context`对象，以及与示例 12-12 中相同参数的`acceptor`对象 ➊。接下来，您调用`serve`以开始`async_accept`循环 ➋。

或多或少，`main` 函数几乎与示例 12-12 相同。不同之处在于，你将为运行 `io_context` 分配多个线程，而不仅仅是一个。首先，你初始化一个 `vector` 来存储每个 `future`，对应于你将启动的任务。其次，你使用类似的方法，通过 `std::generate_n` 创建任务 ➌。作为生成函数对象，你传递一个 lambda，调用 `std::async` ➍。在 `std::async` 调用中，你传递执行策略 `std::launch::async` 和一个函数对象，该对象调用 `run` 来运行你的 `io_context`。

现在你已经为运行 `io_context` 分配了一些任务，Boost Asio 就开始运行了。你将希望等待所有异步操作完成，因此你需要对存储在 `futures` 中的每个 `future` 调用 `get` ➎。此循环完成后，每个 `Request` 都已完成，你准备好打印结果响应的摘要 ➏。

有时创建额外的线程并将它们分配给处理 I/O 是有意义的。通常，一个线程就足够了。你必须衡量这种优化（以及并发代码带来的相关困难）是否值得。

### 总结

本章介绍了 Boost Asio，一个用于低级 I/O 编程的库。你学习了如何在 Asio 中排队异步任务并提供线程池的基础知识，以及如何与其基本的网络功能进行交互。你编写了几个程序，包括一个使用同步和异步方法的简单 HTTP 客户端和一个回声服务器。

**练习**

**20-1.** 使用 Boost Asio 文档调查 UDP 类与本章中学习的 TCP 类的类似功能。将示例 20-14 中的大写回声服务器重写为一个 UDP 服务。

**20-2.** 使用 Boost Asio 文档调查 ICMP 类。编写一个程序，对给定子网中的所有主机进行 ping 测试，执行网络分析。调查 *Nmap*，一款免费的网络映射程序，网址为 *[`nmap.org/`](https://nmap.org/)*。

**20-3.** 调查 Boost Beast 文档。使用 Beast 重写示例 20-10 和 20-11。

**20-4.** 使用 Boost Beast 编写一个 HTTP 服务器，从目录中提供文件。有关帮助，请参考文档中提供的 Boost Beast 示例项目。

**进一步阅读**

+   *TCP/IP 指南*，作者：Charles M. Kozierok（No Starch Press，2005）

+   *错综复杂的网络：现代 Web 应用程序安全指南*，作者：Michal Zalewski（No Starch Press，2012）

+   *Boost C++ 库*（第二版），作者：Boris Schäling（XML Press，2014）

+   *Boost.Asio C++ 网络编程*（第二版），作者：Wisnu Anggoro 和 John Torjo（Packt，2015）
