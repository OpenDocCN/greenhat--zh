## 第二十四章：**编写应用程序**

*对于一群没有毛发的猿人，我们实际上已经发明了一些相当了不起的东西。*

—Ernest Cline*,《玩家一号》

![图片](img/common.jpg)

本章包含了一些重要的主题，通过教授构建真实世界应用程序的基础知识，帮助你更好地理解 C++的实际应用。首先讨论 C++内置的程序支持，允许你与应用生命周期进行交互。接着，你将学习 Boost ProgramOptions，这是一个非常优秀的开发控制台应用程序的库，它提供了接受用户输入的功能，省去了你重新发明轮子的麻烦。此外，你还将学习一些关于预处理器和编译器的特殊主题，这些内容你在构建源代码超过一个文件的应用程序时，可能会遇到。

### 程序支持

有时你的程序需要与操作环境的应用生命周期进行交互。本节涵盖了三类主要的交互：

+   处理程序终止和清理

+   与环境的通信

+   管理操作系统信号

为了帮助说明本节中的各种功能，你将使用清单 21-1 作为框架。它使用了一个改进版的类模拟，类比于清单 4-5 中的`Tracer`类，来自第四章，帮助跟踪在各种程序终止场景中哪些对象被清理。

```
#include <iostream>
#include <string>

struct Tracer { ➊
  Tracer(std::string name_in)
    : name{ std::move(name_in) } {
    std::cout << name << " constructed.\n";
  }
  ~Tracer() {
    std::cout << name << " destructed.\n";
  }
private:
  const std::string name;
};

Tracer static_tracer{ "static Tracer" }; ➋

void run() { ➌
  std::cout << "Entering run()\n";
  // ...
  std::cout << "Exiting run()\n";
}

int main() {
  std::cout << "Entering main()\n"; ➍
  Tracer local_tracer{ "local Tracer" }; ➎
  thread_local Tracer thread_local_tracer{ "thread_local Tracer" }; ➏
  const auto* dynamic_tracer = new Tracer{ "dynamic Tracer" }; ➐
  run(); ➑
  delete dynamic_tracer; ➒
  std::cout << "Exiting main()\n"; ➓
}
-----------------------------------------------------------------------
static Tracer constructed. ➋
Entering main() ➍
local Tracer constructed. ➎
thread_local Tracer constructed. ➏
dynamic Tracer constructed. ➐
Entering run() ➑
Exiting run() ➑
dynamic Tracer destructed. ➒
Exiting main() ➓
local Tracer destructed. ➎
thread_local Tracer destructed. ➏
static Tracer destructed. ➋
```

*清单 21-1：一个用于调查程序终止和清理功能的框架*

首先，你声明了一个`Tracer`类，它接受一个任意的`std::string`标签，并在`Tracer`对象构造和析构时向 stdout 报告 ➊。接着，你声明了一个具有静态存储持续时间的`Tracer` ➋。`run`函数报告程序进入和退出时的情况 ➌。中间部分是一个单独的注释，你将在后续的部分中用其他代码替换。在`main`中，你进行一次声明 ➍；初始化具有局部 ➎、线程局部 ➏ 和动态 ➐ 存储持续时间的`Tracer`对象；并调用`run` ➑。然后，你删除动态的`Tracer`对象 ➒，并宣布即将从`main`返回 ➓。

**警告**

*如果清单 21-1 中的输出让你感到惊讶，请在继续之前复习一下第 89 页中的“对象的存储持续时间”！*

#### *处理程序终止和清理*

`<cstdlib>`头文件包含了若干用于管理程序终止和资源清理的函数。程序终止函数可以分为两个大类：

+   那些导致程序终止的交互

+   注册回调函数，当程序终止即将发生时

##### 使用 std::atexit 的终止回调

要注册一个在程序正常终止时调用的函数，你可以使用`std::atexit`函数。你可以注册多个函数，它们将按注册的逆序被调用。回调函数不接受任何参数，并且返回`void`。如果`std::atexit`成功注册了一个函数，它将返回一个非零值；否则，返回零。

示例 21-2 展示了你可以注册一个`atexit`回调，并且它将在预期的时刻被调用。

```
#include <cstdlib>
#include <iostream>
#include <string>

struct Tracer {
--snip--
};

Tracer static_tracer{ "static Tracer" };

 void run() {
  std::cout << "Registering a callback\n"; ➊
  std::atexit([] { std::cout << "***std::atexit callback executing***\n"; }); ➋
  std::cout << "Callback registered\n"; ➌
}

int main() {
--snip--
}
-----------------------------------------------------------------------
static Tracer constructed.
Entering main()
local Tracer constructed.
thread_local Tracer constructed.
dynamic Tracer constructed.
Registering a callback
Callback registered ➌
dynamic Tracer destructed.
Exiting main()
local Tracer destructed.
thread_local Tracer destructed.
***std::atexit callback executing*** ➋
static Tracer destructed.
```

*示例 21-2：注册一个`atexit`回调*

在`run`中，你宣布即将注册一个回调 ➊，你注册了一个 ➋，然后你宣布即将从`run`返回 ➌。在输出中，你可以清楚地看到回调发生在你从`main`返回后，并且所有非静态对象都已销毁。

编写回调函数时，有两个重要的注意事项：

+   你不能从回调函数中抛出未捕获的异常。这样会导致调用`std::terminate`。

+   你需要非常小心与程序中的非静态对象交互。`atexit`回调函数在`main`返回后执行，因此除非特别小心保持它们的存活，否则所有局部、线程局部和动态对象将在此时被销毁。

**警告**

*你可以使用 std::atexit 注册至少 32 个函数，尽管确切的限制由实现定义。*

##### 使用 std::exit 退出

在本书中，你一直通过从`main`返回来终止程序。在某些情况下，比如多线程程序中，你可能希望以其他方式优雅地退出程序，尽管你应该避免引入相关的复杂性。你可以使用`std::exit`函数，它接受一个整数`int`作为程序的退出代码。它将执行以下清理步骤：

1.  与当前线程关联的线程局部对象和静态对象将被销毁。任何`atexit`回调函数将被调用。

1.  所有的 stdin、stdout 和 stderr 都会被刷新。

1.  任何临时文件都会被删除。

1.  程序会将给定的状态码报告给操作环境，之后操作环境会恢复控制。

示例 21-3 通过注册一个`atexit`回调并在`run`内部调用`exit`，展示了`std::exit`的行为。

```
#include <cstdlib>
#include <iostream>
#include <string>

struct Tracer {
--snip--
};

Tracer static_tracer{ "static Tracer" };

void run() {
  std::cout << "Registering a callback\n"; ➊
  std::atexit([] { std::cout << "***std::atexit callback executing***\n"; }); ➋
  std::cout << "Callback registered\n"; ➌
  std::exit(0); ➍
}

int main() {
--snip--
}
-----------------------------------------------------------------------
static Tracer constructed.
Entering main()
local Tracer constructed.
thread_local Tracer constructed.
dynamic Tracer constructed.
Registering a callback ➊
Callback registered ➌
thread_local Tracer destructed.
***std::atexit callback executing*** ➍
static Tracer destructed.
```

*示例 21-3：调用`std::exit`*

在`run`中，你宣布正在注册一个回调 ➊，你通过`atexit`注册了一个回调 ➋，你宣布完成注册 ➌，然后你使用零作为参数调用`exit` ➍。将示例 21-3 的程序输出与示例 21-2 的输出进行比较。请注意，以下几行没有出现：

```
dynamic Tracer destructed.
Exiting main()
local Tracer destructed.
```

根据`std::exit`的规则，调用栈上的局部变量不会被清理。当然，因为程序从`run`中没有返回到`main`，所以`delete`也不会被调用。哎呀。

这个例子突出了一个重要的考虑因素：你不应该使用`std::exit`来处理正常的程序执行。这里提到它是为了完整性，因为你可能会在早期的 C++代码中看到它。

**注意**

*`<cstdlib>`头文件还包括一个`std::quick_exit`，它会调用你用`std::at_quick_exit`注册的回调，`std::at_quick_exit`的接口类似于`std::atexit`。主要的区别在于，`at_quick_exit`回调不会执行，除非你显式地调用`quick_exit`，而`atexit`回调在程序即将退出时总会执行。*

##### std::abort

要结束一个程序，你也可以使用`std::abort`来实现这一目标。这个函数接受一个整数值的状态码，并立即将其返回给操作环境。没有对象的析构函数被调用，也没有`std::atexit`回调被触发。清单 21-4 展示了如何使用`std::abort`。

```
#include <cstdlib>
#include <iostream>
#include <string>

struct Tracer {
--snip--
};

Tracer static_tracer{ "static Tracer" };

void run() {
  std::cout << "Registering a callback\n"; ➊
  std::atexit([] { std::cout << "***std::atexit callback executing***\n"; }); ➋
  std::cout << "Callback registered\n"; ➌
  std::abort(); ➍
}

int main() {
  --snip--
}
-----------------------------------------------------------------------
static Tracer constructed.
Entering main()
local Tracer constructed.
thread_local Tracer constructed.
dynamic Tracer constructed.
Registering a callback
Callback registered
```

*清单 21-4：调用`std::abort`*

在`run`中，你再次声明你正在注册一个回调 ➊，你用`atexit`注册一个回调并宣布注册完成 ➌。这一次，你改为调用`abort` ➍。注意，在宣布完成回调注册 ➊ 后，没有输出打印出来。程序没有清理任何对象，且你的`atexit`回调没有被调用。

正如你想象的那样，`std::abort`并没有太多典型的使用场景。你最可能遇到的一个场景是`std::terminate`的默认行为，当同时有两个异常发生时，它会被调用。

#### *与环境的通信*

有时候，你可能希望启动另一个进程。例如，Google 的 Chrome 浏览器会启动多个进程来服务一个浏览器会话。这通过依赖操作系统的进程模型来增强一些安全性和鲁棒性。例如，Web 应用和插件通常会运行在独立的进程中，这样如果它们崩溃，整个浏览器就不会崩溃。此外，通过将浏览器的渲染引擎运行在一个独立的进程中，任何安全漏洞也变得更难被利用，因为 Google 将该进程的权限限制在所谓的沙盒环境中。

##### std::system

你可以使用位于`<cstdlib>`头文件中的`std::system`函数来启动一个独立的进程，它接受一个 C 风格的字符串作为要执行的命令，并返回一个`int`，对应于命令的返回码。实际行为依赖于操作环境。例如，在 Windows 机器上，该函数会调用*cmd.exe*，而在 Linux 机器上会调用*/bin/sh*。该函数在命令执行时会阻塞。

清单 21-5 展示了如何使用`std::system`来 ping 一个远程主机。（如果你不是使用类似 Unix 的操作系统，你需要将`command`的内容更新为适合你操作系统的命令。）

```
#include <cstdlib>
#include <iostream>
#include <string>

int main() {
  std::string command{ "ping -c 4 google.com" }; ➊
  const auto result = std::system(command.c_str()); ➋
  std::cout << "The command \'" << command
            << "\' returned " << result << "\n";
}
-----------------------------------------------------------------------
PING google.com (172.217.15.78): 56 data bytes
64 bytes from 172.217.15.78: icmp_seq=0 ttl=56 time=4.447 ms
64 bytes from 172.217.15.78: icmp_seq=1 ttl=56 time=12.162 ms
64 bytes from 172.217.15.78: icmp_seq=2 ttl=56 time=8.376 ms
64 bytes from 172.217.15.78: icmp_seq=3 ttl=56 time=10.813 ms

--- google.com ping statistics ---
4 packets transmitted, 4 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 4.447/8.950/12.162/2.932 ms
The command 'ping -c 4 google.com' returned 0 ➌
```

*清单 21-5：使用 `std::system` 调用 `ping` 工具（输出来自 macOS Mojave 版本 10.14。）*

首先，你初始化一个名为 `command` 的 `string`，其内容为 `ping -c 4 google.com` ➊。然后，你通过传递 `command` 的内容来调用 `std::system` ➋。这将导致操作系统调用 `ping` 命令并传递参数 `-c 4`（指定发送四次 ping）和地址 `google.com`。接着，你打印一个状态信息，报告 `std::system` 的返回值 ➌。

##### std::getenv

操作环境通常具有 *环境变量*，用户和开发人员可以设置这些变量，以帮助程序查找运行所需的重要信息。`<cstdlib>` 头文件包含了 `std::getenv` 函数，它接受一个 C 风格字符串作为参数，表示你想查找的环境变量的名称，并返回一个 C 风格字符串，包含对应变量的内容。如果未找到该变量，函数将返回 `nullptr`。

清单 21-6 说明了如何使用 `std::getenv` 获取 *路径变量*，该变量包含了包含重要可执行文件的目录列表。

```
#include <cstdlib>
#include <iostream>
#include <string>

int main() {
  std::string variable_name{ "PATH" }; ➊
  std::string result{ std::getenv(variable_name.c_str()) }; ➋
  std::cout << "The variable " << variable_name
            << " equals " << result << "\n"; ➌
}
-----------------------------------------------------------------------
The variable PATH equals /usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```

*清单 21-6：使用 `std::getenv` 获取路径变量（输出来自 macOS Mojave 版本 10.14。）*

首先，你初始化一个名为 `variable_name` 的 `string`，其内容为 `PATH` ➊。接下来，你将调用 `std::getenv` 获取 `PATH` 的结果，并将其存储在一个名为 `result` 的字符串中 ➋。然后，你将结果打印到标准输出 ➌。

#### *操作系统信号管理*

操作系统信号是异步通知，发送给进程，通知程序发生了某个事件。`<csignal>` 头文件包含了六个宏常量，代表操作系统发送给程序的不同信号（这些信号与操作系统无关）：

+   `SIGTERM` 表示终止请求。

+   `SIGSEGV` 表示无效的内存访问。

+   `SIGINT` 表示外部中断，例如键盘中断。

+   `SIGILL` 表示无效的程序镜像。

+   `SIGABRT` 表示异常终止条件，例如 `std::abort`。

+   `SIGFPE` 表示浮点错误，例如除以零。

要为这些信号注册处理程序，你可以使用 `<csignal>` 头文件中的 `std::signal` 函数。它接受一个 `int` 类型的参数，表示信号宏列表中的一个信号。第二个参数是一个函数指针（而不是函数对象！），指向一个接受 `int` 类型信号宏并返回 `void` 的函数。这个函数必须使用 C 链接（尽管大多数实现也允许 C++ 链接）。你将在本章后面学习 C 链接。现在，只需在你的函数定义前加上 `extern "C"`。请注意，由于中断的异步性质，任何对全局可变状态的访问都必须进行同步。

清单 21-7 包含一个等待键盘中断的程序。

```
#include <csignal>
#include <iostream>
#include <chrono>
#include <thread>
#include <atomic>

std::atomic_bool interrupted{}; ➊

extern "C" void handler(int signal) {
  std::cout << "Handler invoked with signal " << signal << ".\n"; ➋
  interrupted = true; ➌
}

int main() {
  using namespace std::chrono_literals;
  std::signal(SIGINT, handler); ➍
  while(!interrupted) { ➎
    std::cout << "Waiting..." << std::endl; ➏
    std::this_thread::sleep_for(1s);
  }
  std::cout << "Interrupted!\n"; ➐
}
-----------------------------------------------------------------------
Waiting...
Waiting...
Waiting...
Handler invoked with signal 2.
Interrupted! ➐
```

*清单 21-7：使用 `std::signal` 注册键盘中断*

你首先声明一个名为 `interrupted` 的 `atomic_bool`，用于存储程序是否收到键盘中断 ➊（它具有静态存储期，因为你不能在 `std::signal` 中使用函数对象，因此必须使用非成员函数来处理回调）。接下来，你声明一个回调处理程序，接受一个名为 `signal` 的 `int`，将其值打印到标准输出 ➋，并将 `interrupted` 设置为 true ➌。

在 `main` 中，你将 `SIGINT` 中断代码的信号处理程序设置为 `handler` ➍。在循环中，你通过打印消息 ➏ 并休眠一秒 ➐ 来等待程序被中断 ➎。程序一旦被中断，你将打印消息并从 `main` 返回 ➐。

**注意**

*通常，你可以通过按 CTRL-C 来引发现代操作系统中的键盘中断。*

### Boost ProgramOptions

大多数控制台应用程序接受命令行参数。正如你在《三种 `main` 重载》一节中学到的，在第 272 页，你可以定义 `main` 来接受参数 `argc` 和 `argv`，操作环境会分别用参数的数量和内容来填充它们。你总是可以手动解析这些参数并相应地修改程序的行为，但有一个更好的方法：Boost ProgramOptions 库是编写控制台应用程序的重要组成部分。

**注意**

*本节中介绍的所有 Boost ProgramOptions 类都可以在 `<boost/program_options.hpp>` 头文件中找到。*

你可能会想编写自己的参数解析代码，但 ProgramOptions 是一个更明智的选择，原因有四个：

1.  **它更加方便。** 一旦你学会了 ProgramOptions 的简洁声明式语法，你可以轻松地用几行代码描述相当复杂的控制台接口。

1.  **它轻松处理错误。** 当用户错误使用你的程序时，ProgramOptions 会告诉用户如何错误使用程序，而无需你做额外的工作。

1.  **它自动生成帮助提示。** 根据你的声明式标记，ProgramOptions 会为你创建格式良好、易于使用的文档。

1.  **它超越了命令行。** 如果你想从配置文件或环境变量中获取配置，转换命令行参数非常简单。

ProgramOptions 包含三个部分：

1.  **选项描述** 允许你指定允许的选项。

1.  **解析器组件** 从命令行、配置文件和环境变量中提取选项名称和值。

1.  **存储组件** 提供了访问已类型化选项的接口。

在接下来的子章节中，你将学习这些部分的内容。

#### *选项描述*

选项描述组件由三个主要类组成：

+   `boost::program_options::option_description` 描述一个单一选项。

+   `boost::program_options::value_semantic`知道单个选项的期望类型。

+   `boost::program_options::options_description`是一个容器，包含多个`option_description`类型的对象。

你构造一个`options_description`来指定程序选项的描述。可选地，你可以在构造函数中包含一个单独的字符串参数，描述你的程序。如果你包含它，它会在描述中打印出来，但不会对功能产生任何影响。接下来，你使用它的`add_options`方法，这会返回一个特殊类型的对象`boost::program_options::options_description_easy_init`。这个类有一个特殊的`operator()`，接受至少两个参数。

第一个参数是你想要添加的选项的名称。ProgramOptions 非常智能，因此你可以提供一个长名称和一个短名称，用逗号分隔。例如，如果你有一个名为`threads`的选项，ProgramOptions 会将命令行中的`--threads`参数绑定到这个选项。如果你将选项命名为`threads,t`，ProgramOptions 会将`--threads`或`-t`绑定到你的选项。

第二个参数是选项的描述。你可以使用`value_semantic`、C 风格字符串描述或两者的组合。因为`options_description_easy_init`从`operator()`返回对自身的引用，你可以将这些调用链式连接起来，形成程序选项的简洁表示。通常，你不会直接创建`value_semantic`对象，而是使用便捷的模板函数`boost::program_options::value`来生成它们。它接受一个单一的模板参数，对应于选项的期望类型。生成的指针指向一个具有将文本输入（例如来自命令行的输入）解析为期望类型的代码的对象。例如，要指定一个`int`类型的选项，你会调用`value<int>()`。

结果指向的对象将具有多个方法，允许你指定选项的附加信息。例如，你可以使用`default_value`方法来设置选项的默认值。例如，要指定一个`int`类型的选项默认值为 42，你可以使用以下结构：

```
value<int>()->default_value(42)
```

另一个常见的模式是可以接受多个标记的选项。这样的选项允许元素之间有空格，并且它们会被解析为一个单一的字符串。为了实现这一点，只需使用`multitoken`方法。例如，要指定一个选项可以接受多个`std::string`值，你可以使用以下结构：

```
value<std::string>()->multitoken()
```

如果你希望允许同一个选项的多个实例，你可以指定一个`std::vector`作为值，如下所示：

```
value<std::vector<std::string>>()
```

如果你有一个布尔选项，可以使用方便的函数`boost::program_options::bool_switch`，该函数接受一个指向`bool`的指针。如果用户包含相应的选项，函数将把指针指向的`bool`设置为 true。例如，以下构造将会把名为`flag`的`bool`设置为`true`，如果包含了相应的选项：

```
bool_switch(&flag)
```

`options_description`类支持`operator<<`，因此你可以轻松创建格式良好的帮助对话框，而无需额外的努力。清单 21-8 展示了如何使用 ProgramOptions 为名为*mgrep*的示例程序创建一个`program_options`对象。

```
#include <boost/program_options.hpp>
#include <iostream>
#include <string>

int main(int argc, char** argv) {
  using namespace boost::program_options;
  bool is_recursive{}, is_help{};

  options_description description{ "mgrep [options] pattern path1 path2 ..."
}; ➊
  description.add_options()
          ("help,h", bool_switch(&is_help), "display a help dialog") ➋
          ("threads,t", value<int>()->default_value(4),
                        "number of threads to use") ➌
          ("recursive,r", bool_switch(&is_recursive),
                          "search subdirectories recursively") ➍
          ("pattern", value<std::string>(), "pattern to search for") ➎
          ("paths", value<std::vector<std::string>>(), "path to search"); ➏
  std::cout << description; ➐
}
-----------------------------------------------------------------------
mgrep [options] pattern path1 path2 ...:
 -h [ --help ]             display a help dialog
 -t [ --threads ] arg (=4) number of threads to use
 -r [ --recursive ]        search subdirectories recursively
 --pattern arg             pattern to search for
 --path arg                path to search
```

*清单 21-8：使用 Boost ProgramOptions 生成格式良好的帮助对话框*

首先，你使用自定义的使用字符串初始化一个`options_description`对象 ➊。接着，你调用`add_options`并开始添加选项：一个布尔标志，用于指示是否显示帮助对话框 ➋，一个`int`，用于指示使用多少线程 ➌，另一个布尔标志，用于指示是否以`递归`方式搜索子目录 ➍，一个`std::string`，用于指示在文件中搜索的`pattern` ➎，以及一个`std::string`值的列表，表示要搜索的`paths` ➏。然后，你将`description`写入标准输出 ➐。

假设你尚未实现的 mgrep 程序将始终需要`pattern`和`paths`参数。你可以将这些转换为*位置参数*，正如其名字所示，它们将根据位置分配参数。为此，你需要使用`boost::program_options::positional_options_description`类，该类不需要任何构造函数参数。你使用`add`方法，该方法接受两个参数：一个 C 风格字符串，表示你希望转换为位置参数的选项，以及一个`int`，表示你希望绑定的参数数量。你可以多次调用`add`来添加多个位置参数。但顺序很重要，位置参数将从左到右绑定，所以你第一次调用`add`时，绑定的是左侧的位置参数。对于最后一个位置参数，你可以使用数字`-1`来告诉 ProgramOptions 将所有剩余的元素绑定到相应的选项。

清单 21-9 提供了一段代码片段，你可以将其附加到清单 21-7 中的`main`函数，以添加位置参数。

```
  positional_options_description positional; ➊
  positional.add("pattern", 1); ➋
  positional.add("path", -1); ➌
```

*清单 21-9：将位置参数添加到清单 21-8 中*

你初始化了一个没有任何构造函数参数的`positional_options_description` ➊。接着，你调用`add`方法并传入参数`pattern`和`1`，这将把第一个位置参数绑定到*pattern*选项 ➋。你再次调用`add`方法，这次传入参数`path`和`-1` ➌，这将把剩余的位置参数绑定到*path*选项。

#### *解析选项*

现在你已经声明了程序如何接受选项，你可以解析用户输入。可以从环境变量、配置文件和命令行获取配置。为了简洁起见，本节只讨论最后一种情况。

**注意**

*有关如何从环境变量和配置文件获取配置信息，请参考 Boost ProgramOptions 文档，特别是教程部分。*

为了解析命令行输入，你使用 `boost::program_options::command_line_parser` 类，该类接受两个构造函数参数：一个 `int` 类型的参数对应于 *argc*，即命令行上的参数个数，另一个 `char**` 类型的参数对应于 *argv*，即命令行上参数的值（或内容）。该类提供了多个重要方法，你将使用这些方法来声明解析器如何解释用户输入。

首先，你将调用其 `options` 方法，该方法接受一个对应于你的 `options_description` 的参数。接下来，你将使用 `positional` 方法，该方法接受一个对应于你的 `positional_options_description` 的参数。最后，你将调用 `run` 方法，而不传递任何参数。这会导致解析器解析命令行输入并返回一个 `parsed_options` 对象。

清单 21-10 提供了一段代码，你可以将其追加到 `main` 中，放在 清单 21-8 之后，用于集成一个 `command_line_parser`。

```
command_line_parser parser{ argc, argv }; ➊
parser.options(description); ➋
parser.positional(positional); ➌
auto parsed_result = parser.run(); ➍
```

*清单 21-10：将 `command_line_parser` 添加到 清单 21-8*

你通过传递 `main` 中的参数 ➊ 来初始化一个名为 `parser` 的 `command_line_parser`。接着，你将 `options_description` 对象传递给 `options` 方法 ➋，并将 `positional_options_description` 传递给 `positional` 方法 ➌。然后你调用 `run` 方法生成 `parsed_options` 对象 ➍。

**警告**

*如果用户传入无法解析的输入，例如提供了不在你的描述中的选项，解析器将抛出一个继承自 `std::exception` 的异常。*

#### *存储和访问选项*

你将程序选项存储到 `boost::program_options::variables_map` 类中，该类的构造函数不接受任何参数。为了将解析后的选项放入 `variables_map`，你使用 `boost::program_options::store` 方法，该方法的第一个参数是一个 `parsed_options` 对象，第二个参数是一个 `variables_map` 对象。然后你调用 `boost::program_options::notify` 方法，该方法接受一个 `variables_map` 对象作为参数。此时，你的 `variables_map` 包含了用户指定的所有选项。

清单 21-11 提供了一段代码，你可以将其追加到 `main` 中，放在 清单 21-10 之后，用于将结果解析为 `variables_map`。

```
variables_map vm; ➊
store(parsed_result, vm); ➋
notify(vm); ➌
```

*清单 21-11：将结果存储到 `variables_map` 中*

首先声明一个 `variables_map` ➊。接下来，你将从列表 21-10 中传递你的 `parsed_result` 和新声明的 `variables_map` 给 `store` ➋。然后在你的 `variables_map` 上调用 `notify` ➌。

`variables_map` 类是一个关联容器，基本上类似于 `std::map<std::string, boost::any>`。要提取一个元素，你可以通过传递选项名称作为键，使用 `operator[]`。结果是一个 `boost::any`，因此你需要使用其 `as` 方法将其转换为正确的类型。（你在《`any`》一章中已经学习过 `boost::any`，参考 第 378 页。）使用 `empty` 方法检查任何可能为空的选项非常重要。如果你没有这样做并且强行转换 `any`，将会导致运行时错误。

列表 21-12 展示了如何从 `variables_map` 中检索值。

```
if (is_help) std::cout << "Is help.\n"; ➊
if (is_recursive) std::cout << "Is recursive.\n"; ➋
std::cout << "Threads: " << vm["threads"].as<int>() << "\n"; ➌
if (!vm["pattern"].empty()) { ➍
  std::cout << "Pattern: " << vm["pattern"].as<std::string>() << "\n"; ➎
} else {
  std::cout << "Empty pattern.\n";
}
if (!vm["path"].empty()) { ➏
  std::cout << "Paths:\n";
  for(const auto& path : vm["path"].as<std::vector<std::string>>()) ➐
    std::cout << "\t" << path << "\n";
} else {
  std::cout << "Empty path.\n";
}
```

*列表 21-12：从 `variables_map` 中检索值*

由于你使用 `bool_switch` 值来处理 `help` 和 `recursive` 选项，你只需直接使用这些布尔值来判断用户是否请求了这两个选项 ➊➋。由于 `threads` 有默认值，你无需确认它是否为空，因此可以直接使用 `as<int>` 提取其值 ➌。对于那些没有默认值的选项，例如 `pattern`，你首先检查它是否为空 ➍。如果这些选项不为空，你可以使用 `as<std::string>` 提取它们的值 ➎。对 `path` 选项也做相同的操作 ➏，它允许你使用 `as<std::vector<std::string>>` 提取用户提供的集合 ➐。

#### *将一切结合起来*

现在你已经具备了组装基于 ProgramOptions 的应用所需的所有知识。列表 21-13 展示了一种将之前的代码片段结合在一起的方法。

```
#include <boost/program_options.hpp>
#include <iostream>
#include <string>

int main(int argc, char** argv) {
  using namespace boost::program_options;
  bool is_recursive{}, is_help{};

  options_description description{ "mgrep [options] pattern path1 path2 ..." };
  description.add_options()
          ("help,h", bool_switch(&is_help), "display a help dialog")
          ("threads,t", value<int>()->default_value(4),
                        "number of threads to use")
          ("recursive,r", bool_switch(&is_recursive),
                         "search subdirectories recursively")
          ("pattern", value<std::string>(), "pattern to search for")
          ("path", value<std::vector<std::string>>(), "path to search");

  positional_options_description positional;
  positional.add("pattern", 1);
  positional.add("path", -1);

  command_line_parser parser{ argc, argv };
  parser.options(description);
  parser.positional(positional);

  variables_map vm;
  try {
    auto parsed_result = parser.run(); ➊
    store(parsed_result, vm);
    notify(vm);
  } catch (const std::exception& e) {
    std::cerr << e.what() << "\n";
    return -1;
  }

  if (is_help) { ➋
    std::cout << description;
    return 0;
  }
  if (vm["pattern"].empty()) { ➌
    std::cerr << "You must provide a pattern.\n";
    return -1;
  }
  if (vm["path"].empty()) { ➍
    std::cerr << "You must provide at least one path.\n";
    return -1;
  }
  const auto threads = vm["threads"].as<int>();
  const auto& pattern = vm["pattern"].as<std::string>();
  const auto& paths = vm["path"].as<std::vector<std::string>>();
  // Continue program here ... ➎
  std::cout << "Ok." << std::endl;
}
```

*列表 21-13：使用之前的代码片段的完整命令行参数解析应用*

与之前的代码片段不同的是，你将调用解析器的 `run` 函数封装在一个 `try`-`catch` 块中，以减轻用户提供的错误输入 ➊。如果他们确实提供了错误输入，你只需捕获异常，打印错误到 stderr，并 `return`。

一旦你声明并存储了程序选项，像在列表 21-8 到 21-12 中的示例一样，你首先检查用户是否请求了帮助提示 ➋。如果请求了，你只需打印用法并退出，因为无需进行任何进一步的检查。接下来，你进行一些错误检查，确保用户提供了模式 ➌ 和至少一个路径 ➍。如果没有，你将打印一个错误并显示程序的正确用法后退出；否则，你可以继续编写程序 ➎。

列表 21-14 展示了你的程序的各种输出，程序已被编译成二进制文件 mgrep。

```
$ ./mgrep ➊
You must provide a pattern.
$ ./mgrep needle ➋
You must provide at least one path.
$ ./mgrep --supercharge needle haystack1.txt haystack2.txt ➌
unrecognised option '--supercharge'
$ ./mgrep --help ➍
mgrep [options] pattern path1 path2 ...:
  -h [ --help ]             display a help dialog
  -t [ --threads ] arg (=4) number of threads to use
  -r [ --recursive ]        search subdirectories recursively
  --pattern arg             pattern to search for
  --path arg                path to search
$ ./mgrep needle haystack1.txt haystack2.txt haystack3.txt ➎
Ok.
$ ./mgrep --recursive needle haystack1.txt ➏
Ok.
$ ./mgrep -rt 10 needle haystack1.txt haystack2.txt ➐
Ok.
```

*列表 21-14：来自 列表 21-13 中程序的各种调用和输出*

前三次调用由于不同的原因返回错误：你没有提供模式 ➊，你没有提供路径 ➋，或者你提供了一个无法识别的选项 ➌。

在下一次调用中，由于你提供了`--help`选项 ➍，你将获得友好的帮助对话框。最后三次调用正确解析，因为它们都包含模式和至少一个路径。第一次调用没有任何选项 ➎，第二次调用使用了长选项语法 ➏，第三次调用使用了短选项语法 ➐。

### 编译中的特殊话题

本节解释了几个重要的预处理器特性，帮助你理解双重包含问题（将在下一小节中描述）以及如何解决该问题。你将学习如何通过使用编译器标志优化代码的不同选项。此外，你还将了解如何使用特殊的语言关键字，使链接器能够与 C 语言互操作。

#### *重新审视预处理器*

预处理器是一个在编译之前对源代码进行简单转换的程序。你通过预处理器指令给预处理器指令。所有预处理器指令都以井号（`#`）开始。回顾一下“编译器工具链”部分，在第 5 页中，`#include`是一个预处理器指令，它告诉预处理器将相应的头文件内容直接复制并粘贴到源代码中。

预处理器还支持其他指令。最常见的是*宏*，它是一个已赋予名称的代码片段。每当你在 C++代码中使用该名称时，预处理器会将该名称替换为宏的内容。

两种不同类型的宏是类对象宏和函数宏。你可以使用以下语法来声明类对象宏：

```
#define <NAME> <CODE>
```

其中，NAME 是宏的名称，`CODE`是用来替换该名称的代码。例如，清单 21-15 展示了如何将字符串字面量定义为宏。

```
#include <cstdio>
#define MESSAGE "LOL" ➊

int main(){
  printf(MESSAGE); ➋
}
-----------------------------------------------------------------------
LOL
```

*清单 21-15：一个包含类对象宏的 C++程序*

你定义了宏`MESSAGE`，它对应的代码是`"LOL"` ➊。接下来，你将`MESSAGE`宏作为格式字符串传递给`printf` ➋。在预处理器完成对清单 21-15 的处理后，它会呈现为编译器看到的清单 21-16。

```
#include <cstdio>

int main(){
  printf("LOL");
}
```

*清单 21-16：预处理清单 21-15 的结果*

预处理器在这里无非是一个复制粘贴工具。宏消失了，剩下的就是一个简单的程序，它将`LOL`打印到控制台。

**注意**

*如果你想检查预处理器所做的工作，编译器通常有一个标志，允许你仅执行预处理步骤，从而限制编译。这样，编译器会输出每个翻译单元的预处理源文件。在 GCC、Clang 和 MSVC 等编译器中，你可以使用`-E`标志。*

类似函数的宏就像对象宏，只不过它可以在标识符后接受一系列参数：

```
#define <NAME>(<PARAMETERS>) <CODE>
```

你可以在代码中使用这些参数，允许用户自定义宏的行为。列表 21-17 包含了类似函数的宏`SAY_LOL_WITH`。

```
#include <cstdio>
#define SAY_LOL_WITH(fn) fn("LOL") ➊

int main() {
  SAY_LOL_WITH(printf); ➋
}
```

*列表 21-17：一个具有类似函数宏的 C++程序*

`SAY_LOL_WITH`宏接受一个名为`fn`的单一参数 ➊。预处理器将宏粘贴到表达式`fn("LOL")`中。当它评估`SAY_LOL_WITH`时，预处理器会将`printf`粘贴到表达式中 ➋，从而生成一个类似于列表 21-16 的翻译单元。

##### 条件编译

预处理器还提供了*条件编译*功能，这是一种基本的`if`-`else`逻辑。条件编译有几种变体，但你最有可能遇到的是列表 21-18 中所示的形式。

```
#ifndef MY_MACRO ➊
// Segment 1 ➋
#else
// Segment 2 ➌
#endif
```

*列表 21-18：一个带有条件编译的 C++程序*

如果在预处理器评估`#ifndef` ➊时，`MY_MACRO`未定义，列表 21-18 会缩减为`// Segment 1` ➋表示的代码。如果`MY_MACRO`已经`#defined`，列表 21-18 会评估为`// Segment 2` ➌表示的代码。`#else`是可选的。

##### 双重包含

除了使用`#include`，你应该尽量少用预处理器。预处理器非常原始，如果你过度依赖它，会导致很难调试的错误。这一点通过`#include`可以看出，它只是一个简单的复制粘贴命令。

由于你只能定义一个符号一次（这个规则被称为*单一定义规则*），因此必须确保你的头文件不会试图重新定义符号。最容易犯这个错误的方式是重复包含相同的头文件，这就是*双重包含问题*。

避免双重包含问题的常见方法是使用条件编译来制作*包含保护*。包含保护检测头文件是否已经被包含过。如果已经包含过，它会通过条件编译来清空该头文件。列表 21-19 展示了如何为头文件添加包含保护。

```
// step_function.h
#ifndef STEP_FUNCTION_H ➊
int step_function(int x);
#define STEP_FUNCTION_H ➋
#endif
```

*列表 21-19：一个更新了包含保护的`step_function.h`*

当预处理器第一次在源文件中包含`step_function.h`时，宏`STEP_FUNCTION_H`尚未定义，因此`#ifndef` ➊会包含直到`#endif`之间的代码。在这段代码中，你会`#define`宏`STEP_FUNCTION_H` ➋。这样，如果预处理器再次包含`step_function.h`，`#ifndef STEP_FUNCTION_H`会返回假值，不会生成任何代码。

包含保护符号非常普遍，以至于大多数现代工具链都支持 `#pragma once` 这种特殊语法。如果其中一个支持的预处理器看到这一行，它将像头文件有包含保护一样处理。这减少了不少繁琐的步骤。使用这种结构，你可以将示例 21-19 重构为示例 21-20。

```
#pragma once ➊
int step_function(int x);
```

*示例 21-20：更新了 `#pragma once` 的 `step_function.h`*

你所做的只是用 `#pragma once` ➊ 开始了头文件，这是首选方法。一般来说，每个头文件应以 `#pragma once` 开始。

#### *编译器优化*

现代编译器可以对代码执行复杂的变换，以提高运行时性能并减少二进制文件的大小。这些变换被称为*优化*，它们对程序员有一定成本。优化必然会增加编译时间。此外，优化后的代码通常比非优化代码更难调试，因为优化器通常会消除和重新排列指令。简而言之，在编程时通常希望关闭优化，但在测试和生产时则应开启。因此，编译器通常提供几种优化选项。表 21-1 描述了一个这样的例子——GCC 8.3 中的优化选项，尽管这些标志在主要编译器中都比较常见。

**表 21-1：** GCC 8.3 优化选项

| **标志** | **描述** |
| --- | --- |
| `-O0 (默认)` | 通过关闭优化来减少编译时间，提供良好的调试体验，但运行时性能较差。 |
| `-O` 或 `-O1` | 执行大多数可用的优化，但省略那些可能会消耗大量（编译）时间的优化。 |
| `-O2` | 执行 `-O1` 的所有优化，并加上几乎所有不会大幅增加二进制文件大小的优化。编译时间可能比 `-O1` 要长得多。 |
| `-O3` | 执行 `-O2` 的所有优化，并加上许多可能大幅增加二进制文件大小的优化。再次强调，这比 `-O1` 和 `-O2` 的编译时间要长。 |
| `-Os` | 类似于 `-O2` 进行优化，但优先考虑减少二进制文件大小。你可以将其（大致）看作是 `-O3` 的对立面，`-O3` 优先考虑性能，而可能增加二进制文件大小。所有不会增加二进制文件大小的 `-O2` 优化都会执行。 |
| `-Ofast` | 启用所有 `-O3` 优化，并且包含一些可能违反标准合规性的危险优化。请注意。 |
| `-Og` | 启用不会降低调试体验的优化。提供合理优化、快速编译和易于调试的良好平衡。 |

一般来说，除非有充分理由进行更改，否则应使用 `-O2` 进行生产环境的二进制文件编译。调试时使用 `-Og`。

#### *与 C 的链接*

您可以允许 C 代码通过*语言链接*来引用您程序中的函数和变量。语言链接指示编译器生成具有特定格式的符号，以便于其他目标语言。例如，要允许 C 程序使用您的函数，您只需在代码中添加`extern "C"`语言链接。

请参考清单 21-21 中的`sum.h`头文件，它为`sum`生成了一个 C 兼容的符号。

```
 // sum.h
#pragma once
extern "C" int sum(const int* x, int len);
```

*清单 21-21：使`sum`函数对 C 链接器可用的头文件*

现在，编译器将生成 C 链接器可以使用的对象。要在 C 代码中使用此函数，您只需像往常一样声明`sum`函数：

```
int sum(const int* x, size_t len);
```

然后指示您的 C 链接器包含 C++目标文件。

**注意**

*根据 C++标准，* pragma *是一种向编译器提供超出源代码中嵌入信息的额外信息的方法。这些信息由实现定义，因此编译器不要求以任何方式使用 pragma 指定的信息。* pragma *是希腊语词根，意思是“事实”。*

您还可以反向互操作：通过将 C 编译器生成的目标文件提供给链接器，将 C 编译器输出用于 C++程序中。

假设 C 编译器生成了一个等效于`sum`的函数。您可以使用`sum.h`头文件进行编译，并且链接器可以毫无问题地使用目标文件，这要归功于语言链接。

如果您有多个外部函数，可以使用大括号`{}`，正如清单 21-22 所示。

```
// sum.h
#pragma once

extern "C" {
  int sum_int(const int* x, int len);
  double sum_double(const double* x, int len);
--snip--
}
```

*清单 21-22：重构了清单 21-21，其中包含多个带有`extern`修饰符的函数。*

`sum_int`和`sum_double`函数将具有 C 语言链接。

**注意**

*您还可以通过 Boost Python 实现 C++和 Python 之间的互操作。详情请参阅 Boost 文档。*

### 总结

在本章中，您首先了解了支持程序功能，它们允许您与应用程序生命周期进行交互。接下来，您探索了 Boost ProgramOptions，它使您能够使用声明式语法轻松地接受用户输入。然后，您研究了一些编译中的精选主题，这些主题在扩展 C++应用程序开发时非常有帮助。

**练习**

**21-1.** 在清单 20-12 中的异步大写回显服务器中添加优雅的键盘中断处理。添加一个具有静态存储持续时间的关闭开关，供会话对象和接受器在排队更多异步 I/O 之前检查。

**21-2.** 在清单 20-10 中的异步 HTTP 客户端中添加程序选项。它应该接受主机选项（例如*[www.nostarch.com](http://www.nostarch.com)*)和一个或多个资源（例如*/index.htm*）。它应该为每个资源创建一个单独的请求。

**21-3.** 在第 21-2 题的程序中添加一个选项，接受一个目录，在该目录下写入所有 HTTP 响应。从每个主机/资源组合中派生出文件名。

**21-4.** 实现 mgrep 程序。它应该包含你在第二部分中学到的许多库。研究 Boost 算法中的 Boyer-Moore 查找算法（在 `<boost/algorithm/searching/boyer_moore.hpp>` 头文件中）。使用 std::async 启动任务，并确定一种方法来协调任务之间的工作。

**进一步阅读**

+   *Boost C++ 库*，第二版，作者：Boris Schäling（XML Press，2014）

+   *C++ API 设计*，作者：Martin Reddy（Morgan Kaufmann，2011）
