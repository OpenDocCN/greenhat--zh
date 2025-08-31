## 第十九章：流**

*要么写些值得阅读的东西，要么做些值得书写的事情。

—本杰明·富兰克林*

![Image](img/common.jpg)

本章介绍流这一主要概念，它使你能够使用一个通用框架连接来自任何源的输入和任何目标的输出。你将了解构成该通用框架的基本元素的类、几个内置功能，并学习如何将流集成到用户定义的类型中。

### 流

*流* 模拟 *数据流*。在流中，数据在对象之间流动，这些对象可以对数据执行任意处理。当你使用流时，输出是进入流的数据，输入是流中出来的数据。这些术语反映了用户视角下的流。

在 C++ 中，流是执行输入输出（I/O）的主要机制。无论数据源或目标是什么，你都可以使用流作为连接输入和输出的通用语言。STL 使用类继承来编码不同流类型之间的关系。这些层次结构中的主要类型有：

+   `<ostream>`头文件中的`std::basic_ostream`类模板代表输出设备

+   `<istream>`头文件中的`std::basic_istream`类模板代表输入设备

+   `<iostream>`头文件中的`std::basic_iostream`类模板代表同时具有输入输出功能的设备

这三种流类型都需要两个模板参数。第一个对应流的底层数据类型，第二个对应特征类型。

本节从用户的角度介绍流，而不是从库实现者的角度。你将了解流的接口，并知道如何使用 STL 内置的流支持与标准 I/O、文件和字符串进行交互。如果你必须实现一种新的流（例如，为新的库或框架），你将需要一份 ISO C++ 17 标准、一些工作示例以及大量的咖啡。I/O 很复杂，你会看到这种复杂性在流实现的内部结构中有所体现。幸运的是，设计良好的流类会将这些复杂性隐藏起来，使得用户不必直接面对。

#### *流类*

所有用户交互的 STL 流类都来源于`basic_istream`、`basic_ostream`，或者通过`basic_iostream`同时继承这两者。声明每种类型的头文件还为这些模板提供了`char`和`wchar_t`的特化，如表 16-1 所示。这些广泛使用的特化在处理人类语言数据的输入输出时尤其有用。

**表 16-1：** 主要流模板的模板特化

| **模板** | **参数** | **特化** | **头文件** |
| --- | --- | --- | --- |
| `basic_istream` | `char` | `istream` | `<istream>` |
| `basic_ostream` | `char` | `ostream` | `<ostream>` |
| `basic_iostream` | `char` | `iostream` | `<iostream>` |
| `basic_istream` | `wchar_t` | `wistream` | `<istream>` |
| `basic_ostream` | `wchar_t` | `wostream` | `<ostream>` |
| `basic_iostream` | `wchar_t` | `wiostream` | `<iostream>` |

表 16-1 中的对象是你可以在程序中使用的抽象，你可以利用它们编写通用代码。你想写一个将输出日志记录到任意源的函数吗？如果是，你可以接受一个 `ostream` 引用参数，而不需要处理所有那些令人头疼的实现细节。（稍后在“输出文件流”部分第 542 页，你将学到如何实现这一点。）

通常，你可能需要与用户（或程序的执行环境）进行 I/O 操作。全局流对象提供了一个方便的基于流的封装，供你操作。

##### 全局流对象

STL 在 `<iostream>` 头文件中提供了几个 *全局流对象*，它们封装了输入、输出和错误流 stdin、stdout 和 stderr。这些实现定义的标准流是你程序与其执行环境之间的预连接通道。例如，在桌面环境中，stdin 通常绑定到键盘，stdout 和 stderr 则绑定到控制台。

**注意**

*回想一下，在 第一部分中，你看到过广泛使用 `printf` 向 stdout 写入数据。*

表 16-2 列出了全局流对象，所有这些对象都位于 `std` 命名空间中。

**表 16-2：** 全局流对象

| **对象** | **类型** | **目的** |
| --- | --- | --- |
| `cout``wcout` | `ostream``wostream` | 输出，如屏幕 |
| `cin``wcin` | `istream``wistream` | 输入，如键盘 |
| `cerr``wcerr` | `ostream``wostream` | 错误输出（无缓冲） |
| `clog``wclog` | `ostream``wostream` | 错误输出（有缓冲） |

那么如何使用这些对象呢？流类支持的操作可以分为两类：

**格式化操作** 可能会在执行 I/O 之前对输入参数进行一些预处理

**未格式化操作** 直接执行 I/O 操作

接下来的部分会依次解释这些类别。

##### 格式化操作

所有格式化 I/O 都通过两个函数传递：*标准流操作符*，`operator<<` 和 `operator>>`。你会认出这些是来自“逻辑运算符”部分的左移和右移操作符第 182 页。有些令人困惑的是，流重载了左移和右移操作符，赋予它们完全不同的功能。表达式 `i << 5` 的语义完全依赖于 `i` 的类型。如果 `i` 是一个整数类型，这个表达式的意思是 *取* i *并将其按左移五个二进制位*。如果 `i` 不是一个整数类型，它意味着 *将值 5 写入* i。虽然这种符号冲突很不幸，但在实际应用中并不会造成太大问题。只需要注意你使用的类型，并且充分测试你的代码。

输出流重载了`operator<<`，它被称为*输出操作符*或*插入器*。`basic_ostream`类模板为所有基本类型（除了`void`和`nullptr_t`）及一些 STL 容器（如`basic_string`、`complex`和`bitset`）重载了输出操作符。作为`ostream`的用户，你无需担心这些重载如何将对象转换为可读输出。

清单 16-1 展示了如何使用输出操作符将各种类型写入`cout`。

```
#include <iostream>
#include <string>
#include <bitset>

using namespace std;

int main() {
  bitset<8> s{ "01110011" };
  string str("Crying zeros and I'm hearing ");
  size_t num{ 111 };
  cout << s; ➊
  cout << '\n'; ➋
  cout << str; ➌
  cout << num; ➍
  cout << "s\n"; ➎
}
-----------------------------------------------------------------------
01110011 ➊➋
Crying zeros and I'm hearing 111s ➌➍➎
```

*清单 16-1：使用`cout`和`operator<<`写入标准输出*

你使用输出`操作符`<<`将`bitset` ➊、`char` ➋、`string` ➌、`size_t` ➍和一个以空字符终止的字符串文字 ➎通过`cout`写入标准输出。尽管你向控制台输出了五种不同类型的数据，但你无需处理序列化问题。（考虑如果使用`printf`来得到类似的输出，你将不得不跳过多少障碍。）

标准流操作符的一个非常棒的特点是，它们通常会返回对流的引用。从概念上讲，重载通常是这样定义的：

```
ostream& operator<<(ostream&, char);
```

这意味着你可以将输出操作符链接在一起。通过这种技巧，你可以重构清单 16-1，使得`cout`只出现一次，正如清单 16-2 所示。

```
#include <iostream>
#include <string>
#include <bitset>

using namespace std;

int main() {
  bitset<8> s{ "01110011" };
  string str("Crying zeros and I'm hearing ");
  size_t num{ 111 };
  cout << s << '\n' << str << num << "s\n"; ➊
}
-----------------------------------------------------------------------
01110011
Crying zeros and I'm hearing 111s ➊
```

*清单 16-2：通过链式调用输出操作符重构清单 16-1*

由于每次调用`operator<<`都会返回一个对输出流（此处为`cout`）的引用，你只需将这些调用链接在一起，就能获得相同的输出 ➊。

输入流重载了`operator>>`，它被称为*输入操作符*或*提取器*。`basic_istream`类为所有与`basic_ostream`相同的类型提供了对应的输入操作符重载，同样作为用户，你也可以在很大程度上忽略反序列化的细节。

清单 16-3 展示了如何使用输入操作符从`cin`读取两个`double`对象和一个`string`，然后将推导出的数学运算结果输出到标准输出。

```
#include <iostream>
#include <string>

using namespace std;

int main() {
  double x, y;
  cout << "X: ";
  cin >> x; ➊
  cout << "Y: ";
  cin >> y; ➋

  string op;
  cout << "Operation: ";
  cin >> op; ➌
  if (op == "+") {
    cout << x + y; ➍
  } else if (op == "-") {
    cout << x - y; ➎
  } else if (op == "*") {
    cout << x * y; ➏
  } else if (op == "/") {
 cout << x / y; ➐
  } else {
    cout << "Unknown operation " << op; ➑
  }
}
```

*清单 16-3：一个原始计算器程序，使用`cin`和`operator<<`收集输入*

在这里，你收集了两个`double`类型的值`x` ➊和`y` ➋，接着是`string op` ➌，它编码了所需的运算类型。通过`if`语句，你可以输出指定运算的结果，如加法 ➍、减法 ➎、乘法 ➏和除法 ➐，或者告诉用户`op`是未知的 ➑。

要使用该程序，你需要按照指示在控制台中输入请求的值。一个换行符将会把输入（作为 stdin）传递给`cin`，如清单 16-4 所示。

```
X: 3959 ➊
Y: 6.283185 ➋
Operation: * ➌
24875.1 ➍
```

*清单 16-4：一个示例程序运行，计算地球的周长（以英里为单位），来自清单 16-3*

你输入了两个`double`对象：地球的半径，单位为英里，`3959` ➊ 和 2π，6.283185 ➋，并指定了乘法`*` ➌。结果是地球的周长，单位为英里 ➍。注意，对于整数值➊，你不需要提供小数点；流会智能地知道有一个隐式的小数点。

**注意**

*你可能会想，如果在示例 16-4 中输入一个非数字字符串作为`X` ➊ 或 `Y` ➋ 会发生什么。流进入错误状态，稍后你将在本章的“流状态”部分（第 530 页）了解这个问题。在错误状态下，流停止接受输入，程序将不再接受任何输入。*

##### 未格式化操作

当你在处理基于文本的流时，通常会想使用格式化操作符；然而，如果你在处理二进制数据或编写需要低级访问流的代码时，你需要了解未格式化操作。未格式化输入输出涉及很多细节。为了简洁起见，本节提供了相关方法的总结，如果你需要使用未格式化操作，请参考[input.output]。

`istream`类有许多未格式化的输入方法。这些方法在字节级别操作流，并在表 16-3 中进行了总结。在此表中，`is`是类型为`std::istream <T>`，`s`是`char*`，`n`是流大小，`pos`是位置类型，`d`是类型`T`的定界符。

**表 16-3：** `istream`的未格式化读取操作

| **方法** | **描述** |
| --- | --- |
| is.`get([`c`])` | 返回下一个字符，或者如果提供了字符引用 c，则写入该字符。 |
| is.`get(`s, n, `[`d`])`is.`getline(`s, n, `[`d`])` | 操作`get`将最多 n 个字符读取到缓冲区 s 中，遇到换行符时停止，若提供了 d，则在遇到 d 时停止。操作`getline`与之相同，唯一的区别是它还会读取换行符。两者都会将终止的空字符写入 s。你必须确保`s`有足够的空间。 |
| is.`read(`s, n`)`is.`readsome(`s, n`)` | 操作`read`将最多 n 个字符读取到缓冲区 s 中；遇到文件结尾时会报错。操作`readsome`与之相同，唯一的区别是它不把文件结尾视为错误。 |
| is.`gcount()` | 返回`is`上次未格式化读取操作所读取的字符数。 |
| is.`ignore()` | 提取并丢弃一个字符。 |
| is.`ignore(`n, `[`d`])` | 提取并丢弃最多 n 个字符。如果提供了 d，则在遇到 d 时停止。 |
| is.`peek()` | 返回下一个待读取的字符，但不提取它。 |
| is.`unget()` | 将最后提取的字符放回字符串中。 |
| is.`putback(`c`)` | 如果`c`是最后提取的字符，执行`unget`操作。否则，设置`badbit`。详见“流状态”部分。 |

输出流有相应的未格式化写入操作，它们在非常低的层次上操作流，如表 16-4 所总结。在该表中，`os` 是 `std::ostream <T>` 类型，`s` 是 `char*`，`n` 是流的大小。

**表 16-4：** `ostream` 的未格式化写入操作

| **方法** | **描述** |
| --- | --- |
| os.`put(`c`)` | 将 c 写入流 |
| os.`write(`s, n`)` | 将 n 个字符从 s 写入流 |
| os.`flush()` | 将所有缓冲数据写入底层设备 |

##### 基本类型的特殊格式化

所有基本类型，除了 `void` 和 `nullptr`，都重载了输入和输出操作符，但有些类型有特殊规则：

char **和** wchar_t 输入操作符会跳过空白字符来处理字符类型。

char* **和** wchar_t* 输入操作符首先跳过空白字符，然后读取字符串，直到遇到另一个空白字符或文件结尾（EOF）。必须为输入保留足够的空间。

void* 地址格式依赖于实现，输入和输出操作符也是如此。在桌面系统上，地址通常以十六进制字面量形式表示，如 32 位的 `0x01234567` 或 64 位的 `0x0123456789abcdef`。

bool 输入和输出操作符将布尔值视为数字：`true` 为 1，`false` 为 0。

**数字类型** 输入操作符要求输入必须以至少一个数字开头。格式不正确的输入数字会导致零值结果。

这些规则乍一看可能有些奇怪，但一旦习惯了，它们其实相当简单明了。

**注意**

*避免读取 C 风格字符串，因为你需要确保为输入数据分配了足够的空间。未进行充分检查会导致未定义行为，可能带来严重的安全漏洞。建议使用 `std::string` 替代。*

#### *流状态*

流的状态指示了输入/输出是否失败。每种流类型都暴露出常量静态成员，统称为它的*位标志*，这些标志指示流的可能状态：`goodbit`、`badbit`、`eofbit` 和 `failbit`。要判断流是否处于特定状态，可以调用返回`bool`值的成员函数，表示流是否处于对应状态。表 16-5 列出了这些成员函数、`true`结果对应的流状态以及该状态的含义。

**表 16-5：** 可能的流状态、它们的访问方法及其含义

| **方法** | **状态** | **含义** |
| --- | --- | --- |
| `good()` | `goodbit` | 流处于良好的工作状态。 |
| `eof()` | `eofbit` | 流遇到文件结尾（EOF）。 |
| `fail()` | `failbit` | 输入或输出操作失败，但流可能仍处于良好的工作状态。 |
| `bad()` | `badbit` | 发生了灾难性错误，流不处于良好状态。 |

**注意**

*要将流的状态重置为良好的工作状态，可以调用其 `clear()` 方法。*

流实现了隐式的布尔转换（`operator bool`），因此你可以简单直接地检查流是否处于良好的工作状态。例如，你可以使用一个简单的`while`循环逐词从 stdin 读取输入，直到遇到 EOF（或其他失败条件）。清单 16-5 展示了一个使用此技巧生成 stdin 单词计数的简单程序。

```
#include <iostream>
#include <string>

int main() {
  std::string word; ➊
  size_t count{}; ➋
  while (std::cin >> word) ➌
    count++; ➍
  std::cout << "Discovered " << count << " words.\n"; ➎
}
```

*清单 16-5：一个从 stdin 读取并计数单词的程序*

你声明一个名为`word`的`string`类型变量来接收来自 stdin 的单词➊，并将`count`变量初始化为零➋。在`while`循环的布尔表达式中，你尝试将新的输入赋值给`word`➌。当成功时，你会增加`count`的值➍。一旦失败——例如，遇到 EOF——你就停止增加并打印最终的计数结果➎。

你可以尝试两种方法来测试清单 16-5。首先，你可以直接调用程序，输入一些文本，然后提供 EOF。如何发送 EOF 取决于你的操作系统。在 Windows 命令行中，你可以通过按 CTRL-Z 并回车来输入 EOF。在 Linux bash 或 OS X shell 中，你按 CTRL-D。清单 16-6 演示了如何从 Windows 命令行调用清单 16-5。

```
$ listing_16_5.exe ➊
Size matters not. Look at me. Judge me by my size, do you? Hmm? Hmm. And well
you should not. For my ally is the Force, and a powerful ally it is. Life
creates it, makes it grow. Its energy surrounds us and binds us. Luminous
beings are we, not this crude matter. You must feel the Force around you;
here, between you, me, the tree, the rock, everywhere, yes. ➋
^Z ➌
Discovered 70 words. ➍
```

*清单 16-6：通过在控制台输入来调用清单 16-5 中的程序*

首先，你调用你的程序➊。接着，输入一些任意文本，后跟换行符➋。然后发出 EOF。在 Windows 命令行中，命令行上会显示一些有些晦涩的序列**^Z**，此时你必须按回车键。这会导致`std::cin`进入`eofbit`状态，从而结束清单 16-5 中的`while`循环➌。程序显示你已将 70 个单词发送到 stdin ➍。

在 Linux 和 Mac 以及 Windows PowerShell 中，你有另一个选择。你可以将文本保存到一个文件中，比如*yoda.txt*，而不是直接在控制台中输入。诀窍是使用`cat`命令读取文本文件，然后使用管道操作符`|`将内容传递给你的程序。管道操作符将程序左侧的 stdout 传递到右侧程序的 stdin。以下命令演示了这一过程：

```
$ cat yoda.txt➊ |➋ ./listing_15_4➌
Discovered 70 words.
```

`cat`命令读取*yoda.txt*的内容➊。管道操作符➋将`cat`的 stdout 传递到`listing_15_4`的 stdin➌。由于`cat`在遇到*yoda.txt*的结尾时会发送 EOF，因此你无需手动输入 EOF。

有时你希望在出现某些故障位时，流会抛出异常。你可以通过流的`exceptions`方法轻松做到这一点，该方法接受一个参数，表示你希望抛出异常的位。如果你希望多个位抛出异常，只需使用布尔 OR (`|`)将它们连接起来。

示例 16-7 展示了如何重构 示例 16-5，以便用异常处理 `badbit`，并默认处理 `eofbit`/`failbit`。

```
#include <iostream>
#include <string>

using namespace std;

int main() {
  cin.exceptions(istream::badbit); ➊
  string word;
  size_t count{};
  try { ➋
    while(cin >> word) ➌
      count++;
    cout << "Discovered " << count << " words.\n"; ➍
  } catch (const std::exception& e) { ➎
    cerr << "Error occurred reading from stdin: " << e.what(); ➏
  }
}
```

*示例 16-7：重构 示例 16-5 来处理 `badbit` 异常*

程序通过调用 `std::cin` 上的异常方法开始 ➊。由于 `cin` 是一个 `istream`，你将 `istream::badbit` 作为 `exception` 参数传递，表示希望每当 `cin` 进入灾难性状态时抛出异常。为了处理可能出现的异常，你将现有代码包裹在一个 `try`-`catch` 块中 ➋，这样，如果 `cin` 在读取输入时设置了 `badbit` ➌，用户就不会收到关于词数的消息 ➍。相反，程序会捕获由此产生的异常 ➎ 并打印错误信息 ➏。

#### *缓冲与刷新*

许多 `ostream` 类模板在底层涉及操作系统调用，例如，写入控制台、文件或网络套接字。与其他函数调用相比，系统调用通常比较慢。为了避免每输出一个元素都调用一次系统调用，应用程序可以等待多个元素一起输出，从而提高性能。

排队行为被称为 *缓冲*。当流清空缓冲区并输出内容时，这被称为 *刷新*。通常，这种行为对用户是完全透明的，但有时你可能希望手动刷新 `ostream`。为此（以及其他任务），你可以使用操控符。

#### *操控符*

*操控符* 是一些特殊的对象，用于修改流的输入解释方式或格式化输出。操控符的存在是为了执行许多类型的流操作。例如，`std::ws` 修改一个 `istream`，跳过空白字符。以下是一些其他适用于 `ostream` 的操控符：

+   `std::flush` 会将任何缓冲区中的输出直接刷新到 `ostream`。

+   `std::ends` 发送一个空字节。

+   `std::endl` 类似于 `std::flush`，不过它会先发送一个换行符再进行刷新。

表 16-6 总结了 `<istream>` 和 `<ostream>` 头文件中的操控符。

**表 16-6：** `<istream>` 和 `<ostream>` 头文件中的四个操控符

| **操控符** | **类** | **行为** |
| --- | --- | --- |
| `ws` | `istream` | 跳过所有空白字符 |
| `flush` | `ostream` | 通过调用其 `flush` 方法将任何缓冲数据写入流 |
| `ends` | `ostream` | 发送一个空字节 |
| `endl` | `ostream` | 发送换行并刷新输出 |

例如，你可以将 示例 16-7 中的 ➍ 替换为以下内容：

```
cout << "Discovered " << count << " words." << endl;
```

这将打印一个换行符，并同时刷新输出。

**注意**

*作为一般规则，当程序在一段时间内已经完成向流输出文本时，使用 `std::endl`，当你知道程序很快会继续输出文本时，使用 `\n`。*

标准库提供了许多其他操作符，位于 `<ios>` 头文件中。例如，你可以确定 `ostream` 是以文本方式（`boolalpha`）还是数字方式（`noboolalpha`）表示布尔值；以八进制（`oct`）、十进制（`dec`）或十六进制（`hex`）表示整数值；以十进制表示浮点数（`fixed`）或科学记数法表示（`scientific`）。只需将其中一个操作符传递给 `ostream`，使用 `operator<<`，那么所有后续插入的相应类型的数据都会被操控（不仅仅是紧接着的一个操作数）。 

你还可以使用 `setw` 操作符设置流的宽度参数。流的宽度参数会根据流的不同产生不同的效果。例如，在 `std::cout` 中，`setw` 将固定分配给下一个输出对象的字符数。此外，对于浮点输出，`setprecision` 将设置随后的数字精度。 

示例 16-8 演示了这些操作符如何执行与各种 `printf` 格式说明符类似的功能。 

```
#include <iostream>
#include <iomanip>

using namespace std;

int main() {
  cout << "Gotham needs its " << boolalpha << true << " hero."; ➊
  cout << "\nMark it " << noboolalpha << false << "!"; ➋
  cout << "\nThere are " << 69 << "," << oct << 105 << " leaves in here."; ➌
  cout << "\nYabba " << hex << 3669732608 << "!"; ➍
  cout << "\nAvogadro's number: " << scientific << 6.0221415e-23; ➎
  cout << "\nthe Hogwarts platform: " << fixed << setprecision(2) << 9.750123; ➏
  cout << "\nAlways eliminate " << 3735929054; ➐
  cout << setw(4) << "\n"
       << 0x1 << "\n"
       << 0x10 << "\n"
       << 0x100 << "\n"
       << 0x1000 << endl; ➑
}
-----------------------------------------------------------------------
Gotham needs its true hero. ➊
Mark it 0! ➋
There are 69,151 leaves in here. ➌
Yabba dabbad00! ➍
Avogadro's Number: 6.022142e-23 ➎
the Hogwarts platform: 9.75 ➏
Always eliminate deadc0de ➐
1
10
100
1000 ➑
```

*示例 16-8：演示 `<iomanip>` 头文件中一些操作符的程序*

第一行的 `boolalpha` 操作符使布尔值以文本形式打印为 `true` 和 `false` ➊，而 `noboolalpha` 则使其以 1 和 0 形式打印 ➋。对于整数值，你可以使用 `oct` ➌ 打印为八进制，或使用 `hex` ➍ 打印为十六进制。对于浮点值，你可以使用 `scientific` ➎ 指定科学记数法，并且可以通过 `setprecision` 设置打印的数字精度，使用 `fixed` 指定十进制表示法 ➏。因为操作符应用于所有后续插入流中的对象，所以当你在程序结尾打印另一个整数值时，最后使用的整数操作符（`hex`）会被应用，因此你将得到一个十六进制表示 ➐。最后，你使用 `setw` 设置输出字段宽度为 4，然后打印一些整数值 ➑。

表 16-7 总结了常见操作符的示例。 

**表 16-7：** `<iomanip>` 头文件中可用的许多操作符 

| **操作符** | **行为** |
| --- | --- |
| `boolalpha` | 以文本形式表示布尔值，而非数字形式。 |
| `noboolalpha` | 以数字形式表示布尔值，而非文本形式。 |
| `oct` | 以八进制表示整数值。 |
| `dec` | 以十进制表示整数值。 |
| `hex` | 以十六进制表示整数值。 |
| `setw(n)` | 将流的宽度参数设置为 n。具体效果取决于流。 |
| `setprecision(p)` | 设置浮点数精度为 p。 |
| `fixed` | 以十进制表示浮点数。 |
| `scientific` | 以科学记数法表示浮点数。 |

**注意** 

*请参阅 Nicolai M. Josuttis 所著《C++ 标准库》第 2 版的 第十五章，或参考 [iostream.format]。*

#### *用户定义类型*

你可以通过实现某些非成员函数，使用户自定义类型与流兼容。要为`YourType`实现输出操作符，以下函数声明可以满足大多数用途：

```
ostream&➊ operator<<(ostream&➋ s, const YourType& m ➌);
```

在大多数情况下，你只需返回➊接收到的相同`ostream` ➋。如何将输出发送到`ostream`是由你决定的。但通常，这涉及访问`YourType`上的字段 ➌，可选地执行一些格式化和转换，然后使用输出操作符。例如，清单 16-9 展示了如何为`std::vector`实现输出操作符，以打印其大小、容量和元素。

```
#include <iostream>
#include <vector>
#include <string>

using namespace std;

template <typename T>
ostream& operator<<(ostream& s, vector<T> v) { ➊
  s << "Size: " << v.size()
    << "\nCapacity: " << v.capacity()
    << "\nElements:\n"; ➋
  for (const auto& element : v)
    s << "\t" << element << "\n"; ➌
  return s; ➍
}

int main() {
  const vector<string> characters {
    "Bobby Shaftoe",
    "Lawrence Waterhouse",
    "Gunter Bischoff",
    "Earl Comstock"
  }; ➎
  cout << characters << endl; ➏

  const vector<bool> bits { true, false, true, false }; ➐
  cout << boolalpha << bits << endl; ➑
}
-----------------------------------------------------------------------
Size: 4
Capacity: 4
Elements: ➋
 Bobby Shaftoe ➌
 Lawrence Waterhouse ➌
 Gunter Bischoff ➌
 Earl Comstock ➌

Size: 4
Capacity: 32
Elements: ➋
 true ➌
 false ➌
 true ➌
 false ➌
```

*清单 16-9：演示如何为`vector`实现输出操作符的程序*

首先，你定义一个自定义输出操作符作为模板，使用模板参数作为`std::vector`的模板参数➊。这样，你就可以将输出操作符应用于多种类型的`vector`（只要类型`T`也支持输出操作符）。输出的前三行显示`vector`的大小和容量，以及标题`Elements`，指示接下来是`vector`的元素➋。接下来的`for`循环遍历`vector`中的每个元素，将每个元素分别发送到`ostream`中➌。最后，返回流引用`s` ➍。

在`main`中，你初始化了一个名为`characters`的`vector`，其中包含四个字符串 ➎。借助你定义的输出操作符，你可以像处理基本类型一样，直接将`characters`发送到`cout` ➏。第二个示例使用了一个名为`bits`的`vector<bool>`，你也用四个元素初始化它 ➐，并打印到标准输出 ➑。注意，你使用了`boolalpha`操作符，这样当你定义的输出操作符运行时，`bool`元素会以文本形式打印 ➌。

你还可以提供用户自定义的输入操作符，其工作方式类似。一个简单的推论如下：

```
istream&➊ operator>>(istream&➋ s, YourType& m ➌);
```

与输出操作符类似，输入操作符通常返回➊接收到的相同流 ➋。然而，与输出操作符不同，`YourType`的引用通常不会是`const`，因为你希望使用流中的输入来修改相应的对象 ➌。

清单 16-10 演示了如何为`deque`指定输入操作符，使其将元素推送到容器中，直到插入失败（例如，遇到 EOF 字符）。

```
#include <iostream>
#include <deque>

using namespace std;

template <typename T>
istream& operator>>(istream& s, deque<T>& t) { ➊
  T element; ➋
  while (s >> element) ➌
    t.emplace_back(move(element)); ➍
  return s; ➎
}

int main() {
  cout << "Give me numbers: "; ➏
  deque<int> numbers;
  cin >> numbers; ➐
  int sum{};
  cout << "Cumulative sum:\n";
  for(const auto& element : numbers) {
    sum += element;
    cout << sum << "\n"; ➑
  }
}
-----------------------------------------------------------------------
Give me numbers: ➏ 1 2 3 4 5 ➐
Cumulative sum:
1  ➑
3  ➑
6  ➑
10 ➑
15 ➑
```

*清单 16-10：演示如何为`deque`实现输入操作符的程序*

你的用户定义的输入运算符是一个函数模板，因此你可以接受任何支持输入运算符的 `deque` 类型 ➊。首先，你构造一个 `T` 类型的元素，以便从 `istream` 中存储输入 ➋。接下来，你使用熟悉的 `while` 结构从 `istream` 接受输入，直到输入操作失败 ➌。（回想一下“流状态”一节，流可能因多种原因进入失败状态，包括到达文件末尾或遇到 I/O 错误。）每次插入后，你将结果 `move` 到 `deque` 的 `emplace_back` 中，以避免不必要的拷贝 ➍。插入完成后，你只需返回 `istream` 引用 ➎。

在 `main` 中，你提示用户输入数字 ➏，然后使用插入运算符对新初始化的 `deque` 执行插入操作，将元素从标准输入流插入。在本示例程序的运行中，你输入了数字 1 到 5 ➐。为了增加趣味性，你通过保持一个累积和并对每个元素进行迭代，打印每次迭代的结果 ➑。

**注意**

*前面的示例是简单的用户定义输入和输出运算符的实现。你可能希望在生产代码中扩展这些实现。例如，这些实现仅适用于 `ostream` 类，这意味着它们无法与任何非 `char` 序列一起使用。*

#### *字符串流*

*字符串流类* 提供了从字符序列中读取和写入的功能。这些类在多个场合都非常有用。输入字符串尤其有用，如果你想将字符串数据解析为不同类型。因为你可以使用输入运算符，所以所有标准的操作符功能都可以使用。输出字符串非常适合从可变长度的输入中构建字符串。

##### 输出字符串流

*输出字符串流* 为字符序列提供输出流语义，它们都从 `<sstream>` 头文件中的类模板 `std::basic_ostringstream` 派生，并提供以下特化：

```
using ostringstream = basic_ostringstream<char>;
using wostringstream = basic_ostringstream<wchar_t>;
```

输出字符串流支持与 `ostream` 相同的所有功能。每当你向字符串流发送输入时，流会将这些输入存储到内部缓冲区中。你可以将其视为与 `string` 的 `append` 操作在功能上等效（除了字符串流可能更高效）。

输出字符串流还支持 `str()` 方法，它有两种操作模式。如果没有传递参数，`str` 返回内部缓冲区的副本作为 `basic_string`（因此 `ostringstream` 返回 `string`；`wostringstream` 返回 `wstring`）。如果传递了一个 `basic_string` 参数，字符串流将用该参数的内容替换其缓冲区的当前内容。清单 16-11 演示了如何使用 `ostringstream`，将字符数据发送到其中，构建一个 `string`，重置其内容并重复此过程。

```
#include <string>
#include <sstream>

TEST_CASE("ostringstream produces strings with str") {
  std::ostringstream ss; ➊
  ss << "By Grabthar's hammer, ";
  ss << "by the suns of Worvan. ";

  ss << "You shall be avenged."; ➋
  const auto lazarus = ss.str(); ➌

  ss.str("I am Groot."); ➍
  const auto groot = ss.str(); ➎

  REQUIRE(lazarus == "By Grabthar's hammer, by the suns"
                     " of Worvan. You shall be avenged.");
  REQUIRE(groot == "I am Groot.");
}
```

*清单 16-11：使用 `ostringstream` 构建字符串*

在声明一个`ostringstream` ➊之后，你像使用其他任何`ostream`一样使用它，利用输出操作符发送三个独立的字符序列 ➋。接下来，你调用不带参数的`str`，它生成一个名为`lazarus`的`string` ➌。然后你使用带有字符串字面量`I am Groot` ➍调用`str`，这会替换`ostringstream`的内容 ➎。

**注意**

*回忆一下，在“C 风格字符串”部分，第 45 页提到过，你可以将多个字符串字面量放在连续的行中，编译器会将它们视为一个字符串。这完全是为了源代码格式化的目的。*

##### 输入字符串流

*输入字符串流*为字符序列提供输入流语义，它们都继承自`<sstream>`头文件中的类模板`std::basic_istringstream`，该类提供了以下特化：

```
using istringstream = basic_istringstream<char>;
using wistringstream = basic_istringstream<wchar_t>;
```

这些特化类似于`basic_ostringstream`。你可以通过传递一个适当特化的`basic_string`（对于`istringstream`是`string`，对于`wistringstream`是`wstring`）来构造输入字符串流。列表 16-12 演示了通过构造一个包含三个数字的字符串输入流，并使用输入操作符提取它们。（回忆一下在“格式化操作”中提到的内容，关于第 525 页，空白符是字符串数据的适当分隔符。）

```
TEST_CASE("istringstream supports construction from a string") {
  std::string numbers("1 2.23606 2"); ➊
  std::istringstream ss{ numbers }; ➋
  int a;
  float b, c, d;
  ss >> a; ➌
  ss >> b; ➍
  ss >> c;
  REQUIRE(a == 1);
  REQUIRE(b == Approx(2.23606));
  REQUIRE(c == Approx(2));
  REQUIRE_FALSE(ss >> d); ➎
}
```

*列表 16-12：使用`string`构建`istringstream`对象并提取数值类型*

你从字面量`1 2.23606 2` ➊构建一个`string`，并将其传入名为`ss` ➋的`istringstream`构造函数。这使得你可以像处理任何其他输入流一样，使用输入操作符解析出`int`对象 ➌和`float`对象 ➍。当你耗尽流并且输出操作符失败时，`ss`会转换为`false` ➎。

##### 支持输入和输出的字符串流

此外，如果你需要一个支持输入和输出操作的字符串流，可以使用`basic_stringstream`，它具有以下特化：

```
using stringstream = basic_stringstream<char>;
using wstringstream = basic_stringstream<wchar_t>;
```

该类支持输入和输出操作符、`str`方法以及从字符串构造的功能。列表 16-13 演示了如何使用输入和输出操作符的组合从字符串中提取标记。

```
TEST_CASE("stringstream supports all string stream operations") {
  std::stringstream ss;
  ss << "Zed's DEAD"; ➊

  std::string who;
  ss >> who; ➋
  int what;
  ss >> std::hex >> what; ➌

  REQUIRE(who == "Zed's");
  REQUIRE(what == 0xdead);
}
```

*列表 16-13：使用`stringstream`进行输入和输出*

你创建了一个`stringstream`，并使用输出操作符发送`Zed's DEAD` ➊。接下来，你使用输入操作符从`stringstream`中解析出`Zed's` ➋。因为`DEAD`是一个有效的十六进制整数，所以你使用输入操作符和`std::hex`操纵符将其提取为`int` ➌。

**注意**

*所有字符串流都是可移动的。*

##### 字符串流操作总结

表 16-8 提供了 `basic_stringstream` 操作的部分列表。在此表中，`ss, ss1` 和 `ss2` 类型为 `std::basic_stringstream<T>`；`s` 为 `std::basic_string<``T``>`；`obj` 为格式化对象；`pos` 为位置类型；`dir` 为 `std::ios_base::seekdir`；`flg` 为 `std::ios_base::iostate`。

**表 16-8：** `std::basic_stringstream` 操作的部分列表

| **操作** | **备注** |
| --- | --- |
| `basic_stringstream<`T`>``{ [`s`], [`om`] }` | 执行新构造的字符串流的花括号初始化。默认为空字符串 s 和 `in&#124;out` 打开模式 om。 |
| `basic_stringstream<`T`>``{ move(`ss`) }` | 获取 ss 的内部缓冲区所有权。 |
| `~basic_stringstream` | 析构内部缓冲区。 |
| ss.`rdbuf()` | 返回原始字符串设备对象。 |
| ss.`str()` | 获取字符串设备对象的内容。 |
| ss.`str(`s`)` | 将字符串设备对象的内容设置为 s。 |
| ss `>>` obj | 从字符串流中提取格式化数据。 |
| ss `<<` obj | 将格式化数据插入到字符串流中。 |
| ss.`tellg()` | 返回输入位置索引。 |
| ss.`seekg(`pos`)`ss.`seekg(`pos, dir`)` | 设置输入位置指示符。 |
| ss.`flush()` | 同步底层设备。 |
| ss.`good()`ss.`eof()`ss.`bad()`!ss | 检查字符串流的位状态。 |
| ss.`exceptions(`flg`)` | 配置字符串流，在 flg 中的某一位被设置时抛出异常。 |
| ss1.`swap(`ss2`)``swap(`ss1, ss2`)` | 交换 ss1 和 ss2 的每个元素。 |

#### *文件流*

*文件流类* 提供了读取和写入字符序列的功能。文件流类的结构遵循字符串流类的结构。文件流类模板可用于输入、输出或二者兼有。

文件流类提供了相较于使用原生系统调用操作文件内容的以下主要优势：

+   您将获得常规流接口，这些接口提供了丰富的功能，用于格式化和操作输出。

+   文件流类是文件的 RAII 包装器，这意味着不可能泄露资源，例如文件。

+   文件流类支持移动语义，因此您可以精确控制文件的作用范围。

##### 使用流打开文件

您可以选择两种方式使用文件流打开文件。第一种方法是 `open` 方法，它接受 `const char* filename` 和一个可选的 `std::ios_base::openmode` 位掩码参数。`openmode` 参数可以是 表 16-9 中列出的多种值组合之一。

**表 16-9：** 可能的流状态、其访问方法及含义

| **标志** (**in** `std::ios`) | **文件** | **含义** |
| --- | --- | --- |
| `in` | 必须存在 | 读取 |
| `out` | 如果不存在则创建 | 删除文件，然后写入 |
| `app` | 如果不存在则创建 | 追加 |
| `in&#124;out` | 必须存在 | 从开头读写 |
| `in | app` | 如果文件不存在则创建 | 在文件末尾更新 |
| `out | app` | 如果文件不存在则创建 | 追加模式 |
| `out | trunc` | 如果文件不存在则创建 | 清空文件后进行读写 |
| `in | out | app` | 如果文件不存在则创建 | 在文件末尾更新 |
| `in | out | trunc` | 如果文件不存在则创建 | 清空文件后进行读写 |

此外，你可以将`binary`标志添加到这些组合中的任何一个，以使文件处于*二进制模式*。在二进制模式下，流不会转换特殊字符序列，如行结束符（例如，Windows 上的回车符加换行符）或 EOF。

指定要打开的文件的第二种方法是使用流的构造函数。每个文件流提供一个构造函数，接受与`open`方法相同的参数。所有文件流类都是对它们所拥有的文件句柄的 RAII 封装，因此当文件流对象析构时，文件会自动清理。你也可以手动调用`close`方法，该方法不接受任何参数。如果你知道文件操作已经完成，但你的代码结构使得文件流类对象不会立即析构，那么你可能想手动调用这个方法。

文件流也有默认构造函数，这些构造函数不会打开任何文件。要检查文件是否已打开，可以调用`is_open`方法，该方法不接受任何参数，返回一个布尔值。

##### 输出文件流

*输出文件流*提供字符序列的输出流语义，它们都从`std::basic_ofstream`类模板派生，该模板定义在`<fstream>`头文件中，并提供以下特化：

```
using ofstream = basic_ofstream<char>;
using wofstream = basic_ofstream<wchar_t>;
```

默认的`basic_ofstream`构造函数不会打开文件，而非默认构造函数的第二个可选参数默认设置为`ios::out`。

每当你向文件流发送输入时，流会将数据写入相应的文件。清单 16-14 展示了如何使用`ofstream`将简单的消息写入文本文件。

```
#include <fstream>

using namespace std;

int main() {
  ofstream file{ "lunchtime.txt", ios::out|ios::app }; ➊
  file << "Time is an illusion." << endl; ➋
  file << "Lunch time, " << 2 << "x so." << endl; ➌
}
-----------------------------------------------------------------------
lunchtime.txt:
Time is an illusion. ➋
Lunch time, 2x so. ➌
```

*清单 16-14：一个打开文件 lunchtime.txt 并向其中追加消息的程序。（输出对应程序执行一次后 lunchtime.txt 的内容。）*

你初始化了一个名为`file`的`ofstream`对象，使用路径`lunchtime.txt`和标志`out`与`app` ➊。因为这个标志组合是追加输出，所以你通过输出运算符发送到此文件流的数据会被追加到文件末尾。如预期，文件包含你通过输出运算符传递的消息 ➋➌。

得益于`ios::app`标志，如果*lunchtime.txt*文件存在，程序会将输出追加到该文件。例如，如果你再次运行程序，输出将是：

```
Time is an illusion.
Lunch time, 2x so.
Time is an illusion.
Lunch time, 2x so.
```

程序的第二次迭代将相同的短语添加到了文件末尾。

##### 输入文件流

*输入文件流*提供字符序列的输入流语义，它们都从`std::basic_ifstream`类模板派生，该模板定义在`<fstream>`头文件中，并提供以下特化：

```
using ifstream = basic_ifstream<char>;
using wifstream = basic_ifstream<wchar_t>;
```

默认的`basic_ifstream`构造函数不会打开文件，而非默认构造函数的第二个可选参数默认为`ios::in`。

每当你从文件流中读取数据时，流会从相应的文件中读取数据。考虑下面的示例文件，*numbers.txt*：

```
-54
203
9000
0
99
-789
400
```

列表 16-15 包含了一个程序，使用`ifstream`从包含整数的文本文件中读取数据并返回最大值。输出与调用程序并传递*numbers.txt*文件路径相对应。

```
#include <iostream>
#include <fstream>
#include <limits>

using namespace std;

int main() {
  ifstream file{ "numbers.txt" }; ➊
  auto maximum = numeric_limits<int>::min(); ➋
  int value;
  while (file >> value) ➌
    maximum = maximum < value ? value : maximum; ➍
  cout << "Maximum found was " << maximum << endl; ➎
}
-----------------------------------------------------------------------
Maximum found was 9000 ➎
```

*列表 16-15：一个读取文本文件* numbers.txt *并打印其最大整数的程序*

你首先初始化一个`istream`来打开*numbers.txt*文本文件 ➊。接着，使用`int`类型的最小值初始化最大值变量 ➋。通过典型的输入流和`while`循环组合 ➌，你遍历文件中的每个整数，在找到更大值时更新最大值 ➍。一旦文件流无法再解析任何整数，你就将结果打印到标准输出 ➎。

##### 处理失败

与其他流一样，文件流默默失败。如果你使用文件流构造函数打开文件，你必须检查`is_open`方法来确定流是否成功打开了文件。这个设计与大多数其他标准库对象不同，后者通过异常来强制执行不变量。很难说为什么库实现者选择了这种方法，但事实是，你可以相对容易地选择基于异常的方法。

你可以创建自己的工厂函数，用异常处理文件打开失败。列表 16-16 展示了如何实现一个名为`open`的`ifstream`工厂。

```
#include <fstream>
#include <string>

using namespace std;

ifstream➊ open(const char* path➋, ios_base::openmode mode = ios_base::in➌) {
  ifstream file{ path, mode }; ➍
  if(!file.is_open()) { ➎
    string err{ "Unable to open file " };
    err.append(path);
    throw runtime_error{ err }; ➏
  }
  file.exceptions(ifstream::badbit);
  return file; ➐
}
```

*列表 16-16：一个工厂函数，用于生成处理异常而非默默失败的 `ifstream`*

你的工厂函数返回一个`ifstream` ➊，并接受与文件流构造函数（以及`open`方法）相同的参数：文件`path` ➋和`openmode` ➌。你将这两个参数传递给`ifstream`的构造函数 ➍，然后判断文件是否成功打开 ➎。若未成功，你抛出一个`runtime_error` ➏；若成功，你告诉结果`ifstream`在未来每当其`badbit`被设置时抛出异常 ➐。

##### 文件流操作概述

表 16-10 提供了`basic_fstream`操作的部分列表。在这个表格中，`fs, fs1`和`fs2`是`std:: basic_fstream <T>`类型；`p`是一个 C 风格字符串，`std::string`或`std::filesystem::path`；`om`是`std::ios_base::openmode`；`s`是`std::basic_string<``T``>`；`obj`是一个格式化对象；`pos`是一个位置类型；`dir`是`std::ios_base::seekdir`；`flg`是`std::ios_base::iostate`。

**表 16-10：** `std::basic_fstream` 操作的部分列表

| **操作** | **备注** |
| --- | --- |
| `basic_fstream<`T`>``{ [`p`], [`om`] }` | 对新构建的文件流进行花括号初始化。如果提供了 p，则尝试在路径 p 打开文件。默认情况下不打开，并且使用`in | out`打开模式。 |
| `basic_fstream<`T`>``{ move(`fs`) }` | 获取 fs 的内部缓冲区的所有权。 |
| `~basic_fstream` | 析构内部缓冲区。 |
| fs.`rdbuf()` | 返回原始字符串设备对象。 |
| fs.`str()` | 获取文件设备对象的内容。 |
| fs.`str(`s`)` | 将文件设备对象的内容放入 s 中。 |
|  fs `>>` obj  | 从文件流中提取格式化数据。 |
| fs `<<` obj | 将格式化数据插入到文件流中。 |
| fs.`tellg()` | 返回输入位置索引。 |
| fs.`seekg(`pos`)`fs.`seekg(`pos, dir`)` | 设置输入位置指示器。 |
| fs.`flush()` | 同步底层设备。 |
| fs.`good()`fs.`eof()`fs.`bad()``!`fs | 检查文件流的状态位。 |
| fs.`exceptions(`flg`)` | 配置文件流，在 flg 中的某一位被设置时抛出异常。 |
| fs1.`swap(`fs2`)``swap(`fs1, fs2`)` | 交换 fs1 中的每个元素与 fs2 中的一个元素。 |

#### *流缓冲区*

流不会直接读写数据。背后，它们使用流缓冲区类。从高层次来看，*流缓冲区类* 是模板类，负责发送或提取字符。除非你计划实现自己的流库，否则实现细节不重要，但需要知道它们在多个上下文中存在。你通过使用流的`rdbuf`方法来获取流缓冲区，这是所有流都提供的。

##### 向 sdout 写文件

有时你只想将输入文件流的内容直接写入输出流。为此，你可以从文件流中提取流缓冲区指针，并将其传递给输出操作符。例如，你可以使用`cout`以如下方式将文件内容输出到 stdout：

```
cout << my_ifstream.rdbuf()
```

就这么简单。

##### 输出流缓冲迭代器

*输出流缓冲迭代器* 是模板类，暴露了一个输出迭代器接口，将写入操作转换为底层流缓冲区的输出操作。换句话说，这些是适配器，允许你像使用输出迭代器一样使用输出流。

要构造输出流缓冲迭代器，可以使用`ostreambuf_iterator`模板类（在`<iterator>`头文件中）。它的构造函数接受一个输出流参数和一个对应于构造函数参数模板参数（字符类型）的单一模板参数。示例 16-17 展示了如何从`cout`构造一个输出流缓冲迭代器。

```
#include <iostream>
#include <iterator>

using namespace std;

int main() {
  ostreambuf_iterator<char> itr{ cout }; ➊
  *itr = 'H'; ➋
  ++itr; ➌
  *itr = 'i'; ➍
}
-----------------------------------------------------------------------
H➋i➍
```

*示例 16-17：使用`ostreambuf_iterator`类将消息`Hi`写入 stdout*

在这里，你从`cout`构造一个输出流缓冲区迭代器 ➊，然后像通常的输出操作符那样进行写操作：赋值 ➋，递增 ➌，赋值 ➍，以此类推。结果是逐字符输出到标准输出(stdout)。（回顾“输出迭代器”中关于输出操作符的处理方法，见第 464 页。）

##### 输入流缓冲区迭代器

*输入流缓冲区迭代器*是模板类，暴露出一个输入迭代器接口，将读取操作转换为对底层流缓冲区的读取操作。这与输出流缓冲区迭代器完全类似。

要构造一个输入流缓冲区迭代器，使用`istreambuf_iterator`模板类，该类位于`<iterator>`头文件中。与`ostreambuf_iterator`不同，它接受一个流缓冲区参数，因此你必须在要适配的输入流上调用`rdbuf()`。这个参数是可选的：`istreambuf_iterator`的默认构造函数对应于输入迭代器的范围结束迭代器。例如，清单 16-18 展示了如何使用`string`的基于范围的构造函数从`std::cin`构造一个字符串。

```
#include <iostream>
#include <iterator>
#include <string>

using namespace std;

int main() {
  istreambuf_iterator<char> cin_itr{ cin.rdbuf() } ➊, end{} ➋;
  cout << "What is your name? "; ➌
  const string name{ cin_itr, end }; ➍
  cout << "\nGoodbye, " << name; ➎
}
-----------------------------------------------------------------------
What is your name? ➌josh ➍
Goodbye, josh➎
```

*清单 16-18：使用输入流缓冲区迭代器从`cin`构造一个字符串*

你从`cin`的流缓冲区构造一个`istreambuf_iterator` ➊，以及范围结束迭代器 ➋。向程序的用户发送提示 ➌ 后，你使用`string name`的基于范围的构造函数 ➍ 构造该字符串。当用户输入内容（以 EOF 结束）时，字符串的构造函数会复制输入内容。然后，你使用他们的`name`向用户告别 ➎。（回顾“流状态”部分，见第 530 页，不同操作系统向控制台发送 EOF 的方法有所不同。）

#### *随机访问*

有时你可能需要对流进行随机访问（特别是文件流）。输入和输出操作符显然不支持这种用例，因此`basic_istream`和`basic_ostream`提供了单独的随机访问方法。这些方法跟踪光标或位置，也就是流中当前字符的索引。位置指示输入流将读取的下一个字节或输出流将写入的下一个字节。

对于输入流，你可以使用`tellg`和`seekg`两种方法。`tellg`方法不接受参数，返回当前位置。`seekg`方法允许你设置光标位置，并且有两个重载。第一个选项是提供一个`pos_type`位置参数，用于设置读取位置。第二个选项是提供一个`off_type`偏移量参数，以及一个`ios_base::seekdir`方向参数。`pos_type`和`off_type`由`basic_istream`或`basic_ostream`的模板参数决定，但通常它们会转换为整数类型。`seekdir`类型有以下三种值：

+   `ios_base::beg`指定位置参数相对于起始位置。

+   `ios_base::cur`指定位置参数相对于当前位置。

+   `ios_base::end`指定位置参数是相对于文件末尾的。

对于输出流，您可以使用两个方法`tellp`和`seekp`。它们大致与输入流的`tellg`和`seekg`方法类似：`p`代表 put，`g`代表 get。

考虑一个文件*introspection.txt*，其内容如下：

```
The problem with introspection is that it has no end.
```

清单 16-19 展示了如何使用随机访问方法来重置文件游标。

```
#include <fstream>
#include <exception>
#include <iostream>

using namespace std;

ifstream open(const char* path, ios_base::openmode mode = ios_base::in) { ➊
--snip--
}

int main() {
  try {
    auto intro = open("introspection.txt"); ➋
    cout << "Contents: " << intro.rdbuf() << endl; ➌
    intro.seekg(0); ➍
    cout << "Contents after seekg(0): " << intro.rdbuf() << endl; ➎
    intro.seekg(-4, ios_base::end); ➏
    cout << "tellg() after seekg(-4, ios_base::end): "
                                                    << intro.tellg() << endl; ➐
    cout << "Contents after seekg(-4, ios_base::end): "
                                                    << intro.rdbuf() << endl; ➑
  }
  catch (const exception& e) {
    cerr << e.what();
  }
}
-----------------------------------------------------------------------
Contents: The problem with introspection is that it has no end. ➌
Contents after seekg(0): The problem with introspection is that it has no end. ➎
tellg() after seekg(-4, ios_base::end): 49 ➐
Contents after seekg(-4, ios_base::end): end. ➑
```

*清单 16-19：使用随机访问方法读取文本文件中任意字符的程序*

使用清单 16-16 中的工厂函数 ➊，您打开文本文件*introspection.txt* ➋。接下来，使用`rdbuf`方法 ➌将内容打印到 stdout，重置游标到文件的第一个字符 ➍，然后再次打印内容。请注意，这两次输出是相同的（因为文件没有变化） ➎。然后，您使用`seekg`的相对偏移重载来导航到文件末尾前第四个字符 ➏。使用`tellg`，您会发现这是第 49 个字符（以零为基础的索引） ➐。当您将输入文件打印到 stdout 时，输出只有`end.`，因为这些是文件中的最后四个字符 ➑。

**注意**

*Boost 提供了一个 IOStream 库，具有 std 库所没有的丰富附加功能，包括内存映射文件 I/O、压缩和过滤等功能。*

### 总结

在本章中，您了解了流，这是提供执行 I/O 的公共抽象的主要概念。您还了解了文件作为 I/O 的主要源和目标。您首先了解了 stdlib 中的基本流类，以及如何执行格式化和非格式化操作、检查流状态和处理异常错误。您了解了操作符和如何将流整合到用户定义的类型、字符串流和文件流中。本章的高潮是流缓冲区迭代器，它使您能够将流适配为迭代器。

**练习**

**16-1.** 实现一个输出操作符，打印“扩展示例：刹车”中的`AutoBrake`信息（参见第 283 页）。包括车辆当前的碰撞阈值和速度。

**16-2.** 编写一个程序，接受 stdin 中的输出，将其大写，并将结果写入 stdout。

**16-3.** 阅读 Boost IOStream 的介绍文档。

**16-4.** 编写一个程序，接受一个文件路径，打开文件并打印有关文件内容的摘要信息，包括单词计数、平均单词长度和字符的直方图。

**进一步阅读**

+   *《标准 C++ IOStreams 和区域设置：高级程序员指南与参考》*，作者：Angelika Langer（Addison-Wesley Professional，2000）

+   *ISO 国际标准 ISO/IEC（2017）— C++编程语言*（国际标准化组织；瑞士日内瓦；* [`isocpp.org/std/the-standard/`](https://isocpp.org/std/the-standard/) *）

+   *Boost C++库*，第二版，由 Boris Schäling 编写（XML Press，2014 年）
