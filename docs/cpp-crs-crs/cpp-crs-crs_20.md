## 第二十章：文件系统**

*“所以，你是 UNIX 专家。”当时，兰迪仍然傻到会因为这种关注而感到受宠若惊，而他应该意识到这些话语其实是一种令人毛骨悚然的警告。*

—尼尔·斯蒂芬森*, 《密码锁》

![Image](img/common.jpg)

本章将教你如何使用 stdlib 的文件系统库对文件系统执行操作，如操作和检查文件、列举目录以及与文件流互操作。

stdlib 和 Boost 包含文件系统库。stdlib 的文件系统库起源于 Boost 的，因此它们在很大程度上是可以互换的。本章将重点介绍 stdlib 的实现。如果你有兴趣了解更多关于 Boost 的信息，请参考 Boost 文件系统文档。Boost 和 stdlib 的实现大致相同。

**注意**

*C++标准有一个将 Boost 库纳入标准的历史。这允许 C++社区在将新特性纳入 C++标准之前，先通过 Boost 获得这些特性的使用经验。*

### 文件系统概念

文件系统模型有几个重要概念。核心实体是文件。一个*文件*是一个支持输入输出并存储数据的文件系统对象。文件存在于名为*目录*的容器中，目录可以嵌套在其他目录中。为了简化，目录被视为文件。包含文件的目录称为该文件的*父目录*。

路径是一个字符串，用于标识特定的文件。路径以一个可选的*根名称*开始，这是一个特定于实现的字符串，例如 Windows 上的*C:*或*//localhost*，接着是一个可选的根目录，这是另一个特定于实现的字符串，例如类 Unix 系统上的`/`。路径的其余部分是由实现定义的分隔符分隔的目录序列。路径可以选择性地以非目录文件终止。路径可以包含特殊名称“`.`”和“`..`”，分别表示当前目录和父目录。

一个*硬链接*是一个目录条目，它为一个现有的文件分配了一个名称，*符号链接*（或*符号链接*）为一个路径（该路径可能存在，也可能不存在）分配一个名称。一个以另一个路径（通常是当前目录）为参考点的路径称为*相对路径*，而*规范路径*明确标识了文件的位置，不包含特殊名称“**.**”和“**..**”，且不包含任何符号链接。*绝对路径*是任何明确标识文件位置的路径。规范路径与绝对路径的一个主要区别是，规范路径不能包含特殊名称“**.**”和“**..**”。

**警告**

*如果目标平台不提供分层文件系统，stdlib 文件系统可能不可用。*

### std::filesystem::path

`std::filesystem::path` 是文件系统库中用于建模路径的类，你有许多构造路径的选项。也许最常见的两种方式是默认构造函数，它构造一个空路径，以及接受字符串类型的构造函数，它创建由字符串中的字符表示的路径。像所有其他文件系统类和函数一样，`path` 类位于 `<filesystem>` 头文件中。

在本节中，你将学习如何从 `string` 表示构造路径，将其分解为组成部分，并进行修改。在许多常见的系统和应用程序编程上下文中，你需要与文件进行交互。由于每个操作系统对文件系统的表示都是独特的，stdlib 的文件系统库提供了一个欢迎的抽象，使得你能够轻松编写跨平台代码。

#### *构造路径*

`path` 类支持与其他 `path` 对象以及与 `string` 对象进行比较，使用 `operator==`。但是，如果你只是想检查 `path` 是否为空，它提供了一个返回布尔值的 `empty` 方法。清单 17-1 展示了如何构造两个 `path`（一个为空，一个非空）并对其进行测试。

```
#include <string>
#include <filesystem>

TEST_CASE("std::filesystem::path supports == and .empty()") {
  std::filesystem::path empty_path; ➊
  std::filesystem::path shadow_path{ "/etc/shadow" }; ➋
  REQUIRE(empty_path.empty()); ➌
  REQUIRE(shadow_path == std::string{ "/etc/shadow" }); ➍
}
```

*清单 17-1：构造 `std::filesystem::path`*

你构造了两个路径：一个使用默认构造函数 ➊，另一个指向 `/etc/shadow` ➋。由于你使用了默认构造函数，`empty_path` 的 `empty` 方法返回 `true` ➌。`shadow_path` 等于一个包含 `/etc/shadow` 的 `string`，因为你使用相同的内容构造了它 ➍。

#### 分解路径

`path` 类包含一些分解方法，这些方法实际上是专门的字符串处理工具，允许你提取路径的各个组成部分，例如：

+   `root_name()` 返回根名称。

+   `root_directory()` 返回根目录。

+   `root_path()` 返回根路径。

+   `relative_path()` 返回相对于根路径的路径。

+   `parent_path()` 返回父路径。

+   `filename()` 返回文件名部分。

+   `stem()` 返回去除扩展名后的文件名。

+   `extension()` 返回扩展名。

清单 17-2 提供了这些方法返回的值，针对的是指向一个非常重要的 Windows 系统库 `kernel32.dll` 的路径。

```
#include <iostream>
#include <filesystem>

using namespace std;

int main() {
  const filesystem::path kernel32{ R"(C:\Windows\System32\kernel32.dll)" }; ➊
  cout << "Root name: " << kernel32.root_name() ➋
    << "\nRoot directory: " << kernel32.root_directory() ➌
    << "\nRoot path: " << kernel32.root_path() ➍
    << "\nRelative path: " << kernel32.relative_path() ➎
    << "\nParent path: " << kernel32.parent_path() ➏
    << "\nFilename: " << kernel32.filename() ➐
    << "\nStem: " << kernel32.stem() ➑
    << "\nExtension: " << kernel32.extension() ➒
    << endl;
}
-----------------------------------------------------------------------
Root name: "C:" ➋
Root directory: "\\" ➌
Root path: "C:\\" ➍
Relative path: "Windows\\System32\\kernel32.dll" ➎
Parent path: "C:\\Windows\\System32" ➏
Filename: "kernel32.dll" ➐
Stem: "kernel32" ➑
Extension: ".dll" ➒
```

*清单 17-2：打印路径各种分解结果的程序*

你使用原始字符串字面量构造指向 kernel32 的路径，以避免需要转义反斜杠 ➊。你提取根名称 ➋、根目录 ➌ 和 kernel32 的根路径 ➍ 并将它们输出到标准输出。接下来，你提取相对路径，它显示的是相对于根路径 `C:\` 的路径 ➎。父路径是 `kernel32.dll` 的父路径，它只是包含该文件的目录 ➏。最后，你提取文件名 ➐、文件名主体 ➑ 和扩展名 ➒。

注意，你不需要在任何特定操作系统上运行示例 17-2。没有任何解析方法要求路径实际指向一个存在的文件。你只是提取路径内容的组成部分，而不是指向的文件。当然，不同的操作系统会产生不同的结果，特别是对于分隔符（例如，在 Linux 上是正斜杠）。

**注意**

*示例 17-2 演示了 `std::filesystem::path` 有一个 `operator<<`，它在路径的开头和结尾打印引号。在内部，它使用了 `<iomanip>` 头文件中的模板类 `std::quoted`，该类简化了带引号字符串的插入和提取。此外，记住在字符串字面量中必须转义反斜杠，这就是为什么你在源代码中看到路径中有两个反斜杠，而不是一个的原因。*

#### *修改路径*

除了解析方法外，`path` 还提供了几个 *修改器方法*，允许你修改路径的各种特征：

+   `clear()` 清空 `path`。

+   `make_preferred()` 将所有目录分隔符转换为实现首选的目录分隔符。例如，在 Windows 上，它将通用分隔符 `/` 转换为系统首选的反斜杠 `\`。

+   `remove_filename()` 移除路径中的文件名部分。

+   `replace_filename(p)` 用路径 p 替换 `path` 的文件名。

+   `replace_extension(p)` 用路径 p 替换 `path` 的扩展名。

+   `remove_extension()` 移除路径中的扩展名部分。

示例 17-3 演示了如何使用多个修改器方法操作路径。

```
#include <iostream>
#include <filesystem>

using namespace std;

int main() {
  filesystem::path path{ R"(C:/Windows/System32/kernel32.dll)" };
  cout << path << endl; ➊

  path.make_preferred();
  cout << path << endl; ➋

  path.replace_filename("win32kfull.sys");
  cout << path << endl; ➌

  path.remove_filename();
  cout << path << endl; ➍

  path.clear();
  cout << "Is empty: " << boolalpha << path.empty() << endl; ➎
}
-----------------------------------------------------------------------
"C:/Windows/System32/kernel32.dll" ➊
"C:\\Windows\\System32\\kernel32.dll" ➋
"C:\\Windows\\System32\\win32kfull.sys" ➌
"C:\\Windows\\System32\\" ➍
Is empty: true ➎
```

*示例 17-3：使用修改器方法操作路径。（输出来自 Windows 10 x64 系统。）*

如在示例 17-2 中所示，你构造了一个指向 kernel32 的 `path`，尽管这个路径是非`const`的，因为你将要修改它 ➊。接下来，使用 `make_preferred` 将所有目录分隔符转换为系统首选的目录分隔符。示例 17-3 显示了来自 Windows 10 x64 系统的输出，因此它将斜杠 (`/`) 转换为反斜杠 (`\`) ➋。使用 `replace_filename`，你将文件名从 `kernel32.dll` 替换为 `win32kfull.sys` ➌。再次注意，由该路径描述的文件不需要在你的系统上实际存在；你只是操作路径。最后，使用 `remove_filename` 方法移除文件名 ➍，然后使用 `clear` 完全清空 `path` 的内容 ➎。

#### *文件系统路径方法总结*

表 17-1 包含了 `path` 的可用方法的部分列表。注意表中 `p`、`p1` 和 `p2` 是 `path` 对象，而 `s` 是 `stream`。

**表 17-1：** `std::filestystem::path` 操作总结

| **操作** | **备注** |
| --- | --- |
| `path{}` | 构造一个空路径。 |
| `Path{` s`, [`f`] }` | 从字符串类型 s 构造路径；f 是一个可选的 `path::format` 类型，默认为实现定义的路径格式。 |
| `Path{` p `}`p1 `=` p2 | 复制构造/赋值。 |
| `Path{ move(`p`) }`p1 `= move(`p2`)` | 移动构造/赋值。 |
| p`.assign(`s`)` | 将 p 赋值给 s，丢弃当前内容。 |
| p`.append(`s`)`p `/` s | 将 s 追加到 p 后，包含适当的分隔符 `path::preferred_separator`。 |
| p`.concat(`s`)`p `+` s | 将 s 追加到 p 后，不包括分隔符。 |
| p`.clear()` | 清除内容。 |
| p`.empty()` | 如果 p 为空，则返回 true。 |
| p`.make_preferred()` | 将所有目录分隔符转换为实现首选的目录分隔符。 |
| p`.remove_filename()` | 移除文件名部分。 |
| p1`.replace_filename(`p2`)` | 将 p1 的文件名替换为 p2 的文件名。 |
| p1`.replace_extension(`p2`)` | 将 p1 的扩展名替换为 p2 的扩展名。 |
| p`.root_name()` | 返回根名称。 |
| p`.root_directory()` | 返回根目录。 |
| p`.root_path()` | 返回根路径。 |
| p`.relative_path()` | 返回相对路径。 |
| p`.parent_path()` | 返回父路径。 |
| p`.filename()` | 返回文件名。 |
| p`.stem()` | 返回 stem 部分。 |
| p`.extension()` | 返回扩展名。 |
| p`.has_root_name()` | 如果 p 有根名称，则返回 true。 |
| p`.has_root_directory()` | 如果 p 有根目录，则返回 true。 |
| p`.has_root_path()` | 如果 p 有根路径，则返回 true。 |
| p`.has_relative_path()` | 如果 p 有相对路径，则返回 true。 |
| p`.has_parent_path()` | 如果 p 有父路径，则返回 true。 |
| p`.has_filename()` | 如果 p 有文件名，则返回 true。 |
| p`.has_stem()` | 如果 p 有 stem 部分，则返回 true。 |
| p`.has_extension()` | 如果 p 有扩展名，则返回 true。 |
| p`.c_str()`p`.native()` | 返回 p 的本地字符串表示。 |
| p`.begin()`p`.end()` | 顺序访问路径的元素，作为半开区间。 |
| s `<<` p | 将 p 写入 s。 |
| s `>>` p | 将 s 读入 p。 |
| p1`.swap(`p2`)``swap(`p1`,` p2`)` | 交换 p1 和 p2 中的每个元素。 |
| p1 `==` p2p1 `!=` p2p1 `>` p2p1 `>=` p2p1 `<` p2p1 `<=` p2 | 按字典顺序比较两个路径 p1 和 p2。 |

### 文件与目录

`path` 类是文件系统库的核心元素，但它的任何方法都不会与文件系统直接交互。相反，`<filesystem>` 头文件包含了非成员函数来执行这些操作。可以把 `path` 对象看作是声明你想与之交互的文件系统组件，而 `<filesystem>` 头文件则包含了执行这些操作的函数。

这些函数具有友好的错误处理接口，允许你将路径拆分成例如目录名、文件名和扩展名等部分。使用这些函数，你可以使用许多工具与环境中的文件进行交互，而无需使用特定操作系统的应用程序编程接口。

#### *错误处理*

与环境文件系统交互可能会导致错误，例如找不到文件、权限不足或不支持的操作。因此，文件系统库中每个与文件系统交互的非成员函数必须向调用者传达错误条件。这些非成员函数提供了两种选项：抛出异常或设置错误变量。

每个函数有两个重载版本：一个允许你传递一个 `std::system_error` 的引用，另一个则省略该参数。如果你提供引用，函数会将 `system_error` 设置为一个错误条件（如果发生错误）。如果不提供引用，函数将抛出一个 `std::filesystem::filesystem_error`（继承自 `std::system_error` 的异常类型）。

#### *路径组合函数*

作为使用 `path` 构造函数的替代方法，你可以构造各种类型的路径：

+   `absolute(`p, `[`ec`])` 返回一个绝对路径，指向与 p 相同的位置，但 `is_absolute()` 返回 true。

+   `canonical(`p, `[`ec`])` 返回一个规范路径，指向与 p 相同的位置。

+   `current_path([`ec`])` 返回当前路径。

+   `relative(`p, `[`base`], [`ec`])` 返回一个相对路径，其中 p 相对于 `base`。

+   `temp_directory_path([`ec`])` 返回一个用于临时文件的目录。结果保证是一个已存在的目录。

请注意，`current_path`支持重载，因此你可以设置当前目录（类似于 Posix 系统中的 cd 或 chdir）。只需提供一个路径参数，例如 `current_path(`p, `[`ec`])`。

清单 17-4 展示了这些函数的应用实例。

```
#include <filesystem>
#include <iostream>

using namespace std;

int main() {
  try {
    const auto temp_path = filesystem::temp_directory_path(); ➊
    const auto relative = filesystem::relative(temp_path); ➋
    cout << boolalpha
      << "Temporary directory path: " << temp_path ➌
      << "\nTemporary directory absolute: " << temp_path.is_absolute() ➍
      << "\nCurrent path: " << filesystem::current_path() ➎
      << "\nTemporary directory's relative path: " << relative ➏
      << "\nRelative directory absolute: " << relative.is_absolute() ➐
      << "\nChanging current directory to temp.";
    filesystem::current_path(temp_path); ➑
    cout << "\nCurrent directory: " << filesystem::current_path(); ➒
  } catch(const exception& e) {
    cerr << "Error: " << e.what(); ➓
  }
}
-----------------------------------------------------------------------
Temporary directory path: "C:\\Users\\lospi\\AppData\\Local\\Temp\\" ➌
Temporary directory absolute: true ➍
Current path: "c:\\Users\\lospi\\Desktop" ➎
Temporary directory's relative path: "..\\AppData\\Local\\Temp" ➏
Relative directory absolute: false ➐
Changing current directory to temp. ➑
Current directory: "C:\\Users\\lospi\\AppData\\Local\\Temp" ➒
```

*清单 17-4：一个使用多个路径组合函数的程序。（输出来自 Windows 10 x64 系统。）*

你可以使用 `temp_directory_path` 构造路径，它返回系统的临时文件目录 ➊，然后使用 `relative` 确定其相对路径 ➋。打印临时路径 ➌ 后，`is_absolute` 说明该路径是绝对路径 ➍。接着，打印当前路径 ➎ 以及临时目录相对于当前路径的路径 ➏。由于这是相对路径，`is_absolute` 返回 `false` ➐。一旦你将路径更改为临时路径 ➑，然后打印当前目录 ➒。当然，你的输出可能与 清单 17-4 中的输出不同，如果系统不支持某些操作，甚至可能会出现 `exception` ➓。（回想一下章节开始时的警告：C++ 标准允许某些环境可能不支持文件系统库中的部分或全部功能。）

#### *检查文件类型*

你可以使用以下函数检查文件的属性：

+   `is_block_file(`p, `[`ec`])` 用于判断 p 是否是 *块文件*，这是一种在某些操作系统中使用的特殊文件（例如，Linux 中的块设备，允许你以固定大小的块传输随机可访问的数据）。

+   `is_character_file(`p, `[`ec`])` 用于判断 p 是否是 *字符文件*，这是一种在某些操作系统中使用的特殊文件（例如，Linux 中的字符设备，允许你发送和接收单个字符）。

+   `is_regular_file(`p, `[`ec`])` 用于判断 p 是否是常规文件。

+   `is_symlink(`p, `[`ec`])` 用于判断 p 是否是符号链接，它是指向另一个文件或目录的引用。

+   `is_empty(`p, `[`ec`])` 用于判断 p 是否是一个空文件或空目录。

+   `is_directory(`p, `[`ec`])` 用于判断 p 是否是一个目录。

+   `is_fifo(`p, `[`ec`])` 用于判断 p 是否是 *命名管道*，这是一种在许多操作系统中使用的特殊进程间通信机制。

+   `is_socket(`p, `[`ec`])` 用于判断 p 是否是 *套接字*，这也是许多操作系统中使用的另一种特殊进程间通信机制。

+   `is_other(`p, `[`ec`])` 用于判断 p 是否是除常规文件、目录或符号链接之外的某种文件。

Listing 17-5 使用 `is_directory` 和 `is_regular_file` 来检查四个不同的路径。

```
#include <iostream>
#include <filesystem>

using namespace std;

void describe(const filesystem::path& p) { ➊
  cout << boolalpha << "Path: " << p << endl;
  try {
    cout << "Is directory: " << filesystem::is_directory(p) << endl; ➋
 cout << "Is regular file: " << filesystem::is_regular_file(p) << endl; ➌
  } catch (const exception& e) {
    cerr << "Exception: " << e.what() << endl;
  }
}

int main() {
  filesystem::path win_path{ R"(C:/Windows/System32/kernel32.dll)" };
  describe(win_path); ➍
  win_path.remove_filename();
  describe(win_path); ➎

  filesystem::path nix_path{ R"(/bin/bash)" };
  describe(nix_path); ➏
  nix_path.remove_filename();
  describe(nix_path); ➐
}
```

*Listing 17-5：一个使用 `is_directory` 和 `is_regular_file` 检查四个典型的 Windows 和 Linux 路径的程序。*

在一台 Windows 10 x64 机器上，运行 Listing 17-5 程序输出了以下结果：

```
Path: "C:/Windows/System32/kernel32.dll" ➍
Is directory: false ➍
Is regular file: true ➍
Path: "C:/Windows/System32/" ➎
Is directory: true ➎
Is regular file: false ➎
Path: "/bin/bash" ➏
Is directory: false ➏
Is regular file: false ➏
Path: "/bin/" ➐
Is directory: false ➐
Is regular file: false ➐
```

在一台 Ubuntu 18.04 x64 机器上，运行 Listing 17-5 程序输出了以下结果：

```
Path: "C:/Windows/System32/kernel32.dll" ➍
Is directory: false ➍
Is regular file: false ➍
Path: "C:/Windows/System32/" ➎
Is directory: false ➎
Is regular file: false ➎
Path: "/bin/bash" ➏
Is directory: false ➏
Is regular file: true ➏
Path: "/bin/" ➐
Is directory: true ➐
Is regular file: false ➐
```

首先，你定义了 `describe` 函数，它接受一个单一的 `path` ➊ 参数。打印路径后，你还会打印该路径是否是一个目录 ➋ 或常规文件 ➌。在 `main` 中，你传递了四个不同的路径给 `describe`：

+   `C:/Windows/System32/kernel32.dll` ➍

+   `C:/Windows/System32/` ➎

+   `/bin/bash` ➏

+   `/bin/` ➐

注意，结果是操作系统特定的。

#### *检查文件和目录*

你可以使用以下函数检查各种文件系统属性：

+   `current_path([`p`], [`ec`])`，如果提供了 p，则将程序的当前路径设置为 p；否则，它返回程序的当前路径。

+   `exists(`p, `[`ec`])` 返回文件或目录是否存在于 p。

+   `equivalent(`p1, p2, `[`ec`])` 返回 p1 和 p2 是否指向同一个文件或目录。

+   `file_size(`p, `[`ec`])` 返回位于 p 的常规文件的字节大小。

+   `hard_link_count(`p, `[`ec`])` 返回 p 的硬链接数量。

+   `last_write_time(`p, `[`t`] [`ec`])`，如果提供了 t`ec``t`，则将 p 的最后修改时间设置为 t；否则，它会返回 p 的最后修改时间。（t 是一个 `std::chrono::time_point`。）

+   `permissions(`p, prm, `[`ec`])` 设置 p 的权限。 prm 是 `std::filesystem::perms` 类型，这是一个基于 POSIX 权限位模型的枚举类。（参考 [fs.enum.perms]。）

+   `read_symlink(`p, `[`ec`])` 返回符号链接 p 的目标。

+   `space(`p, `[`ec`])` 返回文件系统 p 占用的空间信息，形式为 `std::filesystem::space_info`。该 POD 包含三个字段：容量（总大小）、`free`（可用空间）和 `available`（可供非特权进程使用的可用空间）。所有字段都是无符号整数类型，以字节为单位度量。

+   `status(`p, `[`ec`])` 返回文件或目录 p 的类型和属性，形式为 `std::filesystem::file_status`。该类包含一个 `type` 方法，该方法不接受任何参数，返回一个 `std::filesystem::file_type` 类型的对象，这个枚举类包含描述文件类型的值，如 `not_found`、`regular`、`directory` 等。`symlink file_status` 类还提供一个 `permissions` 方法，不接受任何参数，返回一个 `std::filesystem::perms` 类型的对象。（详细信息参考 [fs.class.file_status]。）

+   `symlink_status(`p, `[`ec`])` 返回状态，不跟随符号链接。

如果你熟悉类似 Unix 的操作系统，肯定多次使用过 `ls`（“列出”命令）来列举文件和目录。在类似 DOS 的操作系统（包括 Windows）中，你可以使用类似的 `dir` 命令。稍后你将在本章中（在 Listing 17-7）使用这些函数来构建自己的简单列出程序。

现在你已经知道如何检查文件和目录，让我们来看一下如何操作路径所指向的文件和目录。

#### *操作文件和目录*

此外，文件系统库包含许多操作文件和目录的方法：

+   `copy(`p1, p2, `[`opt`], [`ec`])` 将文件或目录从 p1 复制到 p2。你可以提供一个 `std::filesystem::copy_options` `opt` 来自定义 `copy_file` 的行为。这个 `enum` 类可以接受多个值，包括 none（如果目标已存在则报告错误）、`skip_existing`（保留现有文件）、`overwrite_existing`（覆盖现有文件）和 `update_existing`（如果 p1 较新则覆盖）。 （详细信息参考 **[**fs.enum.copy.opts**]**。）

+   `copy_file(`p1, p2, `[`opt`], [`ec`])` 类似于 copy，除了如果 p1 不是常规文件时，它会生成错误。

+   `copy_file(`p1, `p2`, [`opt`], [`ec`])` 类似于 copy，除了如果 p1 不是常规文件时，它会生成错误。

+   `create_directory(`p`, [`ec`])` 创建目录 p。

+   `create_directories(`p`, [`ec`])` 类似于递归调用 `create_directory`，因此，如果嵌套路径包含不存在的父目录，使用这种形式。

+   `create_hard_link(`tgt, `lnk`, [`ec`])` 在 lnk 处创建指向 tgt 的硬链接。

+   `create_symlink(`tgt, `lnk`, [`ec`])` 在 lnk 处创建指向 tgt 的符号链接。

+   `create_directory_symlink``(`tgt`,` lnk`, [`ec`])` 应用于目录，而不是`create_symlink`。

+   `remove``(`p`, [`ec`])` 删除文件或空目录 p（不跟随符号链接）。

+   `remove_all``(`p`, [`ec`])` 递归删除文件或目录 p（不跟随符号链接）。

+   `rename``(`p1`,` p2`, [`ec`])` 将 p1 重命名为 p2。

+   `resize_file``(`p`,` new_size`, [`ec`])` 将 p（如果是常规文件）调整为 new_size。如果该操作增加了文件大小，新的空间将被零填充。否则，操作会从文件末尾裁剪 p。

你可以创建一个程序，使用这些方法中的几种来复制、调整大小和删除文件。列表 17-6 通过定义一个打印文件大小和修改时间的函数来说明这一点。在`main`函数中，程序创建并修改了两个`path`对象，并在每次修改后调用该函数。

```
#include <iostream>
#include <filesystem>

using namespace std;
using namespace std::filesystem;
using namespace std::chrono;
 void write_info(const path& p) {
  if (!exists(p)) { ➊
    cout << p << " does not exist." << endl;
    return;
  }
  const auto last_write = last_write_time(p).time_since_epoch();
  const auto in_hours = duration_cast<hours>(last_write).count();
  cout << p << "\t" << in_hours << "\t" << file_size(p) << "\n"; ➋
}

int main() {
  const path win_path{ R"(C:/Windows/System32/kernel32.dll)" }; ➌
  const auto reamde_path = temp_directory_path() / "REAMDE"; ➍
  try {
    write_info(win_path); ➎
    write_info(reamde_path); ➏

    cout << "Copying " << win_path.filename()
         << " to " << reamde_path.filename() << "\n";
    copy_file(win_path, reamde_path);
    write_info(reamde_path); ➐

    cout << "Resizing " << reamde_path.filename() << "\n";
    resize_file(reamde_path, 1024);
    write_info(reamde_path); ➑

    cout << "Removing " << reamde_path.filename() << "\n";
    remove(reamde_path);
    write_info(reamde_path); ➒
  } catch(const exception& e) {
    cerr << "Exception: " << e.what() << endl;
  }
}
-----------------------------------------------------------------------
"C:/Windows/System32/kernel32.dll"      3657767 720632 ➎
"C:\\Users\\lospi\\AppData\\Local\\Temp\\REAMDE" does not exist. ➏
Copying "kernel32.dll" to "REAMDE"
"C:\\Users\\lospi\\AppData\\Local\\Temp\\REAMDE"        3657767 720632 ➐
Resizing "REAMDE"
"C:\\Users\\lospi\\AppData\\Local\\Temp\\REAMDE"        3659294 1024 ➑
Removing "REAMDE"
"C:\\Users\\lospi\\AppData\\Local\\Temp\\REAMDE" does not exist. ➒
```

*列表 17-6：一个示例程序，展示了几种与文件系统交互的方法。（输出来自 Windows 10 x64 系统。）*

`write_info`函数接受一个`path`参数。你检查该路径是否存在 ➊，如果不存在，则打印错误信息并立即返回。如果路径存在，打印消息显示其最后的修改时间（自纪元以来的小时数）和文件大小 ➋。

在`main`中，你创建了一个指向`kernel32.dll`的路径`win_path` ➌ 和一个指向文件系统临时文件目录中不存在的文件`REAMDE`的路径`reamde_path` ➍。（回顾表 17-1，你可以使用`operator/`连接两个路径对象。）在`try`-`catch`块中，你对这两个路径调用`write_info` ➎➏。（如果你使用的是非 Windows 机器，输出会有所不同。你可以将`win_path`修改为系统中存在的文件来继续操作。）

接下来，你将`win_path`处的文件复制到`reamde_path`，并在其上调用`write_info` ➐。注意，与之前的情况 ➏ 相比，`reamde_path`处的文件存在，且它的最后写入时间和文件大小与`kernel32.dll`相同。

然后，你将`reamde_path`处的文件大小调整为 1024 字节，并调用`write_info` ➑。注意，最后的写入时间从 3657767 增加到 3659294，文件大小从 720632 减少到 1024。

最后，你删除`reamde_path`处的文件并调用`write_info` ➒，它告诉你该文件已经不存在。

**注意**

*文件系统如何在后台调整文件大小因操作系统不同而异，超出了本书的范围。但你可以从概念上理解调整大小操作，类似于对`std::vector`的`resize`操作。操作系统会丢弃文件末尾不适合新大小的数据。*

### 目录迭代器

文件系统库提供了两个类用于迭代目录中的元素：`std::filesystem::directory_iterator`和`std::filesystem::recursive_directory_iterator`。`directory_iterator`不会进入子目录，而`recursive_directory_iterator`会。 本节介绍了`directory_iterator`，但是`recursive_directory_iterator`是一个可以替换的实现，并支持所有以下操作。

#### *构造*

`directory_iterator`的默认构造函数会生成结束迭代器。（回忆一下，输入结束迭代器表示输入范围已经用尽。）另一个构造函数接受路径，它表示你想要枚举的目录。可选地，你可以提供`std::filesystem::directory_options`，它是一个`enum`类位掩码，包含以下常量：

+   `none`指示迭代器跳过目录符号链接。如果迭代器遇到权限拒绝，则会产生错误。

+   `follow_directory_symlink`跟随符号链接。

+   `skip_permission_denied`如果迭代器遇到权限拒绝，会跳过目录。

此外，你还可以提供一个`std::error_code`，像所有其他接受`error_code`的文件系统库函数一样，如果在构造过程中发生错误，它会设置此参数，而不是抛出异常。

表 17-2 总结了构造`directory_iterator`的这些选项。请注意，表中的`p`是`path`，`d`是`directory`，`op`是`directory_options`，`ec`是`error_code`。

**表 17-2：** `std::filesystem::directory_iterator`操作总结

| 操作 | 备注 |
| --- | --- |
| `directory_iterator{}` | 构造结束迭代器。 |
| `directory_iterator{` p`, [`op`], [`ec`] }` | 构造一个指向目录 p 的目录迭代器。参数 op 默认为`none`。如果提供，ec 会接收错误条件，而不是抛出异常。 |
| `directory_iterator {` d `}`d1 `=` d2 | 复制构造/赋值。 |
| `directory_iterator { move(`d`) }`d1 `= move(`d2`)` | 移动构造/赋值。 |

#### 目录条目

输入迭代器`directory_iterator`和`recursive_directory_iterator`会为它们遇到的每个条目生成一个`std::filesystem::directory_entry`元素。`directory_entry`类存储一个`path`，以及一些关于该`path`的属性，这些属性通过方法公开。表 17-3 列出了这些方法。请注意，表中的`de`是一个`directory_entry`。 

**表 17-3：** `std::filesystem::directory_entry`操作总结

| 操作 | 描述 |
| --- | --- |
| de`.path()` | 返回引用的路径。 |
| de`.exists()` | 如果引用的路径在文件系统中存在，则返回`true`。 |
| de`.is_block_file()` | 如果引用的路径是块设备，则返回`true`。 |
| de`.is_character_file()` | 如果引用的路径是字符设备，则返回`true`。 |
| de`.is_directory()` | 如果引用的路径是一个目录，则返回`true`。 |
| de`.is_fifo()` | 如果引用路径是命名管道，则返回`true`。 |
| de`.is_regular_file()` | 如果引用路径是常规文件，则返回`true`。 |
| de`.is_socket()` | 如果引用路径是套接字，则返回`true`。 |
| de`.is_symlink()` | 如果引用路径是符号链接，则返回`true` |
| de`.is_other()` | 如果引用路径是其他类型，则返回`true`。 |
| de`.file_size()` | 返回引用路径的大小。 |
| de`.hard_link_count()` | 返回引用路径的硬链接数量。 |
| de`.last_write_time([`t`])` | 如果提供了`t`，则设置引用路径的最后修改时间；否则，返回最后修改时间。 |
| de`.status()` de`.symlink_status()` | 返回引用路径的`std::filesystem::file_status`。 |

你可以使用`directory_iterator`和表 17-3 中的多个操作，创建一个简单的目录列出程序，正如 Listing 17-7 所展示的那样。

```
#include <iostream>
#include <filesystem>
#include <iomanip>

using namespace std;
using namespace std::filesystem;
using namespace std::chrono;

void describe(const directory_entry& entry) { ➊
  try {
    if (entry.is_directory()) { ➋
      cout << "           *";
    } else {
      cout << setw(12) << entry.file_size();
    }
    const auto lw_time =
      duration_cast<seconds>(entry.last_write_time().time_since_epoch());
    cout << setw(12) << lw_time.count()
      << " " << entry.path().filename().string()
      << "\n"; ➌
  } catch (const exception& e) {
    cout << "Error accessing " << entry.path().string()
         << ": " << e.what() << endl; ➍
  }
}

int main(int argc, const char** argv) {
  if (argc != 2) {
    cerr << "Usage: listdir PATH";
    return -1; ➎
  }
  const path sys_path{ argv[1] }; ➏
  cout << "Size         Last Write  Name\n";
  cout << "------------ ----------- ------------\n"; ➐
  for (const auto& entry : directory_iterator{ sys_path }) ➑
    describe(entry); ➒
}
-----------------------------------------------------------------------
> listdir c:\Windows
Size         Last Write  Name
------------ ----------- ------------
 * 13177963504 addins
 * 13171360979 appcompat
--snip--
 * 13173551028 WinSxS
 316640 13167963236 WMSysPr9.prx
 11264 13167963259 write.exe
```

*Listing 17-7：一个使用`std::filesystem::directory_iterator`列举给定目录的文件和目录的程序。（输出来自 Windows 10 x64 系统。）*

**注意**

*你应该将程序的名称从`listdir`修改为与你的编译器输出相匹配的任何值。*

首先定义一个`describe`函数，该函数接受一个`path`引用 ➊，用于检查路径是否为目录 ➋，并为目录打印星号，为文件打印相应的大小。接下来，确定该条目自纪元以来的最后修改时间（以秒为单位），并将其与条目关联的文件名一起打印 ➌。如果发生任何异常，打印错误信息并返回 ➍。

在`main`函数中，首先检查用户是否使用单个参数调用了程序，如果没有，则返回一个负数 ➎。接下来，使用单个参数 ➏ 构造路径，打印一些华丽的输出头部 ➐，遍历目录中的每个`entry` ➑，并将其传递给`describe` ➒。

#### *递归目录迭代*

`recursive_directory_iterator` 是 `directory_iterator` 的替代品，支持相同的所有操作，但会列举子目录。你可以结合使用这些迭代器，构建一个计算给定目录中文件和子目录的大小和数量的程序。Listing 17-8 展示了如何实现。

```
#include <iostream>
#include <filesystem>

using namespace std;
using namespace std::filesystem;

struct Attributes {
  Attributes& operator+=(const Attributes& other) {
    this->size_bytes += other.size_bytes;
    this->n_directories += other.n_directories;
    this->n_files += other.n_files;
    return *this;
  }
  size_t size_bytes;
  size_t n_directories;
  size_t n_files;
}; ➊

void print_line(const Attributes& attributes, string_view path) {
  cout << setw(14) << attributes.size_bytes
       << setw(7) << attributes.n_files
       << setw(7) << attributes.n_directories
       << " " << path << "\n"; ➋
}

Attributes explore(const directory_entry& directory) {
  Attributes attributes{};
  for(const auto& entry : recursive_directory_iterator{ directory.path() }) { ➌
      if (entry.is_directory()) {
        attributes.n_directories++; ➍
      } else {
        attributes.n_files++;
 attributes.size_bytes += entry.file_size(); ➎
      }
  }
  return attributes;
}

int main(int argc, const char** argv) {
  if (argc != 2) {
    cerr << "Usage: treedir PATH";
    return -1; ➏
  }
  const path sys_path{ argv[1] };
  cout << "Size           Files  Dirs   Name\n";
  cout << "-------------- ------ ------ ------------\n";
  Attributes root_attributes{};
  for (const auto& entry : directory_iterator{ sys_path }) { ➐
    try {
      if (entry.is_directory()) {
        const auto attributes = explore(entry); ➑
        root_attributes += attributes;
        print_line(attributes, entry.path().string());
        root_attributes.n_directories++;
      } else {
        root_attributes.n_files++;
        error_code ec;
        root_attributes.size_bytes += entry.file_size(ec); ➒
        if (ec) cerr << "Error reading file size: "
                     << entry.path().string() << endl;
      }
    } catch(const exception&) {
    }
  }
  print_line(root_attributes, argv[1]); ➓
}
-----------------------------------------------------------------------
> treedir C:\Windows
Size         Files  Dirs Name
------------ ----- ----- ------------
 802      1      0 C:\Windows\addins
 8267330      9      5 C:\Windows\apppatch
--snip--
 11396916465  73383  20480 C:\Windows\WinSxS
 21038460348 110950  26513 C:\Windows ➓
```

*Listing 17-8：一个使用`std::filesystem::recursive_directory_iterator`列出给定路径子目录中文件数量和总大小的程序。（输出来自 Windows 10 x64 系统。）*

**注意**

*你应该将程序的名称从`treedir`修改为与你的编译器输出相匹配的任何值。*

在声明用于存储会计数据的`Attributes`类➊后，你定义了一个`print_line`函数，它以用户友好的方式展示`Attributes`实例，并附带路径字符串➋。接下来，你定义了一个`explore`函数，它接受一个`directory_entry`引用并递归地遍历它➌。如果结果的`entry`是一个目录，你会增加目录计数器➍；否则，你会增加文件计数和总大小➎。

在`main`函数中，你检查程序是否确实传入了两个参数。如果没有，你会返回错误代码 -1 ➏。你使用（非递归的）`directory_iterator`枚举`sys_path`所指的目标路径中的内容➐。如果一个`entry`是目录，你会调用`explore`来确定其属性➑，然后将其打印到控制台。你还会增加`root_attributes`中的`n_directories`成员来进行统计。如果`entry`不是目录，你会相应地增加`root_attributes`中的`n_files`和`size_bytes`成员➒。

完成遍历所有`sys_path`子元素后，你会打印`root_attributes`作为最后一行输出➓。例如，清单 17-8 中的最后一行输出显示该特定 Windows 目录包含 110,950 个文件，占用 21,038,460,348 字节（约 21GB）和 26,513 个子目录。

### fstream 互操作性

除了字符串类型外，你还可以使用`std::filesystem::path`或`std::filesystem::directory_entry`来构造文件流（`basic_ifstream`、`basic_ofstream` 或 `basic_fstream`）。

例如，你可以遍历一个目录并构造一个`ifstream`来读取你遇到的每个文件。清单 17-9 展示了如何检查每个 Windows 可执行文件（如 *.sys*、*.dll*、*.exe* 等）开头的魔术 `MZ` 字节，并报告任何违反此规则的文件。

```
#include <iostream>
#include <fstream>
#include <filesystem>
#include <unordered_set>

using namespace std;
using namespace std::filesystem;

int main(int argc, const char** argv) {
  if (argc != 2) {
    cerr << "Usage: pecheck PATH";
    return -1; ➊
  }
  const unordered_set<string> pe_extensions{
    ".acm", ".ax",  ".cpl", ".dll", ".drv",
    ".efi", ".exe", ".mui", ".ocx", ".scr",
    ".sys", ".tsp"
  }; ➋
  const path sys_path{ argv[1] };
  cout << "Searching " << sys_path << " recursively.\n";
  size_t n_searched{};
  auto iterator = recursive_directory_iterator{ sys_path,
                                 directory_options::skip_permission_denied }; ➌
  for (const auto& entry : iterator) { ➍
    try {
 if (!entry.is_regular_file()) continue;
      const auto& extension = entry.path().extension().string();
      const auto is_pe = pe_extensions.find(extension) != pe_extensions.end();
      if (!is_pe) continue; ➎
      ifstream file{ entry.path() }; ➏
      char first{}, second{};
      if (file) file >> first;
      if (file) file >> second; ➐
      if (first != 'M' || second != 'Z')
        cout << "Invalid PE found: " << entry.path().string() << "\n"; ➑
      ++n_searched;
    } catch(const exception& e) {
      cerr << "Error reading " << entry.path().string()
           << ": " << e.what() << endl;
    }
  }
  cout << "Searched " << n_searched << " PEs for magic bytes." << endl; ➒
}
----------------------------------------------------------------------
listing_17_9.exe c:\Windows\System32
Searching "c:\\Windows\\System32" recursively.
Searched 8231 PEs for magic bytes.
```

*清单 17-9：搜索 Windows System32 目录中的 Windows 便携式可执行文件*

在`main`函数中，你检查是否正好传入了两个参数，并根据情况返回相应的错误代码➊。你构建了一个`unordered_set`，其中包含与便携式可执行文件相关的所有扩展名➋，这些扩展名将用于检查文件扩展名。你使用带有`directory_options::skip_permission_denied`选项的`recursive_directory_iterator`来枚举指定路径中的所有文件➌。你遍历每个条目➍，跳过所有不是常规文件的条目，并通过尝试在`pe_extensions`中`find`该条目来判断该条目是否为便携式可执行文件。如果条目没有这种扩展名，你就跳过该文件➎。

要打开文件，只需将`entry`的路径传递给`ifstream`的构造函数➏。然后使用得到的输入文件流将文件的前两个字节读入`first`和`second`➐。如果这两个字符不是`MZ`，则向控制台打印一条消息➑。无论如何，都要增加一个名为`n_searched`的计数器。在用完目录迭代器后，你需要打印一个包含`n_searched`的消息给用户，然后从`main`返回➒。

### 总结

在本章中，你学习了 stdlib 文件系统功能，包括路径、文件、目录和错误处理。这些功能使你能够编写与环境中文件交互的跨平台代码。本章的内容以一些重要的操作、目录迭代器和文件流的互操作性为结尾。

**习题**

**17-1.** 实现一个程序，接受两个参数：一个路径和一个扩展名。该程序应该递归地搜索给定路径，并打印任何具有指定扩展名的文件。

**17-2.** 改进 Listing 17-8 中的程序，使其可以接受一个可选的第二个参数。如果第一个参数以连字符（`-`）开头，程序将读取紧跟连字符后面的所有连续字母，并将每个字母解析为一个选项。第二个参数则变成搜索的路径。如果选项列表中包含*R*，则执行递归目录操作。否则，不使用递归目录迭代器。

**17-3.** 请参阅*dir*或*ls*命令的文档，并在你改进版的 Listing 17-8 中实现尽可能多的选项。

**进一步阅读**

+   *Windows NT 文件系统内部结构：开发者指南*，作者：Rajeev Nagar（O'Reilly，1997）

+   *Boost C++库*（第二版），作者：Boris Schäling（XML Press，2014）

+   *Linux 编程接口：Linux 和 UNIX 系统编程手册*，作者：Michael Kerrisk（No Starch Press，2010）
