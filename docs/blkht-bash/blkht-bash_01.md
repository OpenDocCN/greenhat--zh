

# 第一章：1 BASH 基础



![](img/opener.jpg)

*Bash* 是一种命令语言解释器，提供了一个环境，用户可以在其中执行命令和运行应用程序。作为渗透测试人员和安全从业者，我们经常编写 bash 脚本来自动化各种任务，因此 bash 是黑客的必备工具。在本章中，你将设置你的 bash 开发环境，探索未来脚本中要使用的有用 Linux 命令，并学习该语言语法的基础知识，包括变量、数组、流、参数和操作符。

## 环境设置

在你开始学习 bash 之前，你需要在终端中运行 bash shell 并且有一个文本编辑器。你可以按照本节中的说明，在任何主要操作系统上访问这些工具。

> 注意

*从第四章开始，你将使用 Kali Linux 来运行 bash 命令并完成黑客实验室。如果你现在想设置 Kali，请参考第三章中的步骤。*

### 访问 Bash Shell

如果你使用的是 Linux 或 macOS，bash 应该已经可用。在 Linux 上，按 ALT-CTRL-T 打开终端应用程序。在 macOS 上，你可以通过导航到系统 Dock 上的 Launchpad 图标找到终端。

Kali 和 macOS 默认使用 Z Shell，因此当你打开新的终端窗口时，你需要输入 exec bash 来切换到 bash shell，然后再运行命令。如果你希望将默认 shell 改为 bash，以后不必手动切换 shell，可以使用 chsh -s /bin/bash 命令。

如果你使用的是 Windows，你可以使用 Windows Subsystem for Linux (WSL)，它允许你运行 Linux 发行版并访问 bash 环境。官方的 Microsoft WSL 文档页面描述了如何安装它：*[`learn.microsoft.com/en-us/windows/wsl/install`](https://learn.microsoft.com/en-us/windows/wsl/install)*。

WSL 的替代方案是 *Cygwin*，它通过提供一组 Linux 实用工具和系统调用功能来模拟 Linux 环境。要安装 Cygwin，请访问 *[`www.cygwin.com/install.html`](https://www.cygwin.com/install.html)* 下载安装文件，然后按照安装向导进行操作。

Cygwin 默认安装在 *C:\cygwin64\* Windows 路径下。要执行你的 bash 脚本，请将脚本保存在包含你的用户名的目录下，如 *C:\cygwin64\home*。例如，如果你的用户名是 *david*，你应该将脚本保存在 *C:\cygwin64\home\david* 下。然后，在 Cygwin 终端中，你将能够切换到该 home 目录以运行脚本。

### 安装文本编辑器

要开始编写 bash 脚本，你需要一个文本编辑器，最好是内置语法高亮等实用功能的编辑器。你可以选择基于终端的文本编辑器和基于图形用户界面的文本编辑器。基于终端的文本编辑器（如 vi 或 GNU nano）非常有用，因为在渗透测试过程中，当你需要立即开发脚本时，它们可能是唯一可用的选项。

如果你更喜欢图形化文本编辑器，Sublime Text (*[`www.sublimetext.com`](https://www.sublimetext.com)*) 是一个你可以使用的选项。在 Sublime Text 中，你可以通过点击右下角的**Plain Text**，然后从下拉的语言列表中选择**Bash**来开启 bash 脚本的语法高亮功能。如果你使用的是其他文本编辑器，请参考其官方文档了解如何开启语法高亮功能。

## 探索 Shell

现在你已经有了一个功能齐全的 bash 环境，接下来就该学习一些基础知识了。尽管你将在文本编辑器中开发脚本，但你可能还会经常在终端中运行单个命令。这是因为你通常需要在将命令纳入脚本之前，先查看命令的运行方式和输出结果。让我们通过运行一些 bash 命令来开始。

首先，输入以下命令验证系统中是否有 bash：

```
$ **bash --version**
```

输出中的版本将取决于你运行的操作系统。

### 检查环境变量

在终端中运行时，bash 会在每次启动新会话时加载一组*环境变量*。程序可以使用这些环境变量执行各种任务，比如确定运行脚本的用户身份、用户的主目录位置以及默认的 shell。

要查看 bash 设置的环境变量列表，可以直接从 shell 中运行 env 命令（Listing 1-1）。

```
$ **env**

SHELL=/bin/bash
LANGUAGE=en_CA:en
DESKTOP_SESSION=ubuntu
PWD=/home/user
`--snip--`
```

Listing 1-1: 列出 bash 的环境变量

你可以使用 echo 命令读取单个环境变量，echo 会将文本输出到终端。例如，要打印用户设置的默认 shell，可以使用 SHELL 环境变量，并在变量前加上美元符号 ($)，并用大括号（{}）包围。这会使 bash 扩展该变量并显示其分配的值，如 Listing 1-2 所示。

```
$ **echo ${SHELL}**

/bin/bash 
```

Listing 1-2: 将环境变量打印到终端

以下是一些可用的默认环境变量：

BASH_VERSION 正在运行的 bash 版本

BASHPID 当前 bash 进程的进程标识符（PID）

GROUPS 当前用户所属的组列表

HOSTNAME 主机名

OSTYPE 操作系统类型

PWD 当前工作目录

RANDOM 从 0 到 32,767 的随机数

UID 当前用户的用户 ID（UID）

SHELL shell 的完整路径名

以下示例展示了如何检查这些环境变量中的一些值：

```
$ **echo ${RANDOM}**
8744

$ echo ${UID}
1000

$ **echo ${OSTYPE}**
linux-gnu 
```

这些命令分别生成一个随机数、输出当前用户的 ID，并显示操作系统类型。你可以在*[`www.gnu.org/software/bash/manual/html_node/Bash-Variables.html`](https://www.gnu.org/software/bash/manual/html_node/Bash-Variables.html)*找到完整的环境变量列表。

### 运行 Linux 命令

你在本书中编写的 bash 脚本将运行常见的 Linux 工具，所以如果你还不熟悉命令行导航和文件修改工具，如 cd、ls、chmod、mkdir 和 touch，尝试通过使用 man（手册）命令进行探索。你可以在任何 Linux 命令前加上它，打开一个基于终端的指南，解释该命令的用法和选项，如示例 1-3 所示。

```
$ **man ls**

NAME
      ls - list directory contents

SYNOPSIS
      ls [OPTION]... [FILE]...

DESCRIPTION
      List information about the FILEs (the current directory by default).
      Sort entries alphabetically if none of -cftuvSUX nor
      --sort is specified.

 Mandatory arguments to long options are mandatory for short options too.
      -a, --all
      do not ignore entries starting with .
`--snip--` 
```

示例 1-3：访问命令的手册页

Linux 命令可以在命令行上接受多种类型的输入。例如，你可以输入 ls 命令而不带任何参数，以查看文件和目录，或者传递参数，例如显示所有文件在一行中。

参数通过命令行传递，使用短格式或长格式的参数语法，这取决于正在使用的命令。*短格式*语法使用一个短横线（-）后跟一个或多个字符。以下示例使用 ls 命令列出文件和目录，采用短格式的参数语法：

```
$ **ls -l**
```

一些命令允许你通过将多个参数连接在一起或分别列出它们来传递多个参数：

```
$ **ls -la**
$ **ls -l -a** 
```

请注意，如果尝试用一个单一的短横线连接两个参数，某些命令可能会报错，因此请使用 man 命令了解允许的语法。

有些命令选项允许你使用*长格式*的参数语法，比如--help 命令用于列出可用的选项。长格式的参数语法是以双短横线（--）开头的：

```
$ **ls --help**
```

有时，为了方便，相同的命令参数支持短格式和长格式两种语法。例如，ls 支持参数-a（all），用于显示所有文件，包括隐藏文件。（以点号开头的文件在 Linux 中被视为隐藏文件。）然而，你也可以传递--all 参数，结果将是一样的：

```
$ **ls -a**
$ **ls --all** 
```

让我们执行一些简单的 Linux 命令，以便你可以看到每个命令提供的选项变化。首先，使用 mkdir 创建一个目录：

```
$ **mkdir directory1**
```

现在让我们使用 mkdir 创建两个目录：

```
$ **mkdir directory2 directory3**
```

接下来，使用 ps 命令列出进程，使用短格式的参数语法，先分别提供参数，然后一起提供：

```
$ **ps -e -f**
$ **ps -ef** 
```

最后，让我们使用 df 命令和长格式参数语法来显示可用的磁盘空间：

```
$ **df --human-readable**
```

在本书中，你将使用这些 Linux 命令编写脚本。

## Bash 脚本的元素

在本节中，你将学习 bash 脚本的构建模块。你将使用注释来记录脚本的功能，告诉 Linux 使用特定的解释器执行脚本，并为脚本添加样式以提高可读性。

Bash 没有官方的风格指南，但我们建议遵循 Google 的 Shell 风格指南（*[`google.github.io/styleguide/shellguide.html`](https://google.github.io/styleguide/shellguide.html)*），该指南概述了开发 bash 代码时应遵循的最佳实践。如果你在一个渗透测试团队中工作，并且有一个漏洞利用代码库，使用良好的代码风格实践将帮助你的团队进行维护。

### Shebang 行

每个脚本都应该以*shebang*行开始，这是一个字符序列，起始符号为井号和感叹号（#!），后面跟着脚本解释器的完整路径。列表 1-4 展示了一个典型 bash 脚本的 shebang 行示例。

```
#!/bin/bash
```

列表 1-4：Bash Shebang 行

Bash 解释器通常位于 */bin/bash*。如果你使用 Python 或 Ruby 编写脚本，那么你的 shebang 行会包括 Python 或 Ruby 解释器的完整路径。

你有时会遇到使用类似下面这种 Shebang 行的 bash 脚本：

```
#!/usr/bin/env bash
```

你可能想使用这个 shebang 行，因为它比列表 1-4 中的那个更具可移植性。一些 Linux 发行版将 bash 解释器放置在不同的系统位置，而这个 shebang 行将尝试找到该位置。这个方法在渗透测试中尤其有用，因为你可能不知道目标机器上 bash 解释器的具体位置。然而，为了简化起见，本书中将使用列表 1-4 中的 shebang 版本。

Shebang 行也可以接受可选的参数来改变脚本的执行方式。例如，你可以将特殊参数 -x 传递给你的 bash shebang，像这样：

```
#!/bin/bash -x
```

这个选项会将所有命令及其参数在执行时打印到终端。这对于调试脚本非常有用，尤其是在你开发脚本时。

另一个可选参数的例子是 -r：

```
#!/bin/bash -r
```

这个选项创建了一个*受限的 bash shell*，它限制了某些可能危险的命令，这些命令可能会例如导航到特定目录、修改敏感的环境变量，或者尝试从脚本内部关闭受限的 shell。

在 shebang 行中指定一个参数需要修改脚本，但你也可以通过使用以下语法将参数传递给 bash 解释器：

```
$ **bash -r myscript.sh**
```

无论你是在命令行上传递参数给 bash 解释器，还是在 shebang 行中传递，都不会有区别。命令行选项只是触发不同模式的一种更简便的方式。

### 注释

*注释*是脚本的一部分，bash 解释器不会将其视为代码，它们可以提高程序的可读性。试想一下，你写了一个很长的脚本，几年后需要修改其中的一些逻辑。如果你没有写注释来解释你做了什么，可能会发现很难记住每个部分的目的。

Bash 中的注释以井号 (#) 开始，如列表 1-5 所示。

```
#!/bin/bash

# This is my first script. 
```

列表 1-5：bash 脚本中的注释

除了 shebang 行外，所有以井号开头的行都被视为注释。如果你写了两次 shebang 行，bash 会认为第二行是注释。

要编写多行注释，请在每行前面加上井号，如列表 1-6 所示。

```
#!/bin/bash

# This is my first script!
# Bash scripting is fun... 
```

列表 1-6：多行注释

除了记录脚本的逻辑，注释还可以提供元数据，指示作者、脚本版本、联系问题的人等。这些注释通常出现在脚本的顶部部分，位于 shebang 行下方。

### 命令

脚本可以简短到只有两行：shebang 行和一条 Linux 命令。让我们写一个简单的脚本，将 Hello World! 打印到终端。打开文本编辑器并输入以下内容：

```
**#!/bin/bash**

**echo "Hello World!"** 
```

在这个示例中，我们使用 shebang 语句指定了我们选择的解释器 bash。然后我们使用 echo 命令将字符串 Hello World! 打印到屏幕上。

### 执行

要运行脚本，将文件保存为 *helloworld.sh*，然后打开终端并导航到脚本所在的目录。如果你将文件保存在主目录中，你应该运行列表 1-7 中显示的一组命令。

```
$ **cd ~**
$ **chmod u+x helloworld.sh**
$ **./helloworld.sh**

Hello World! 
```

列表 1-7：从主目录运行脚本

我们使用 cd 命令来切换目录。波浪号（~）代表当前运行用户的主目录。接下来，我们使用 chmod 设置文件拥有者（在这个例子中是我们自己）的可执行（u+x）权限。我们通过使用点斜杠符号（./）后跟脚本名称来运行脚本。点（.）表示当前目录，因此我们实际上是在告诉 bash 从当前工作目录运行*helloworld.sh*。

你也可以使用以下语法运行 bash 脚本：

```
$ **bash helloworld.sh**
```

因为我们指定了 bash 命令，脚本将使用 bash 解释器运行，并且不需要 shebang 行。另外，如果使用 bash 命令，脚本也不需要设置可执行权限（+x）。在后续章节中，你将更深入了解权限模型，并探讨它在渗透测试中查找配置错误的重要性。

### 调试

在编写 bash 脚本时，错误是不可避免的。幸运的是，调试脚本相当直观。一个简单的早期检查错误的方法是运行脚本时使用 -n 参数：

```
$ **bash -n script.sh**
```

这个参数将读取脚本中的命令，但不会执行它们，因此所有语法错误都会显示在屏幕上。你可以把 -n 当作一种干运行方法，用来测试语法的有效性。

你也可以使用 -x 参数开启详细模式，这样可以看到正在执行的命令，并帮助你在脚本实时执行时调试问题：

```
$ **bash -x script.sh**
```

如果你希望在脚本的某个特定位置开始调试，可以在脚本中包含 set 命令（列表 1-8）。

```
#!/bin/bash
set -x

`--snip--`

set +x 
```

清单 1-8：使用 set 调试脚本

你可以将 set 看作是一个开关，用于开启或关闭某个选项。在这个示例中，第一个命令设置调试模式（set -x），而最后一个命令（set +x）禁用调试模式。通过使用 set，你可以避免在脚本较大且包含特定问题区域时，在终端中生成大量无用信息。

## 基本语法

到目前为止，你已经编写了一个两行的脚本，向屏幕打印出消息 "Hello World!"。你还学会了如何运行和调试脚本。现在，你将学习一些 bash 语法，以便编写更有用的脚本。

最基本的 bash 脚本只是将 Linux 命令收集到一个文件中。例如，你可以编写一个脚本，在系统上创建资源，然后将这些资源的信息打印到屏幕上（清单 1-9）。

```
#!/bin/bash

# All this script does is create a directory, create a file
# within the directory, and then list the contents of the directory.

mkdir mydirectory
touch mydirectory/myfile
ls -l mydirectory 
```

清单 1-9：列出目录内容的 bash 脚本

在这个示例中，我们使用 mkdir 创建一个名为*mydirectory*的目录。接着，我们使用 touch 命令在该目录内创建一个名为*myfile*的文件。最后，我们运行 ls -l 命令列出*mydirectory*的内容。

脚本的输出如下所示：

```
`--snip--`
-rw-r--r-- 1 user user 0 Feb 16 13:37 myfile 
```

然而，这种逐行执行的策略可以在多个方面进行改进。首先，当命令运行时，bash 会等待命令完成后再继续执行下一行。如果包含一个长时间运行的命令（如文件下载或大文件复制），剩下的命令将在该命令完成之前无法执行。我们还没有实现任何检查机制来验证所有命令是否正确执行。你需要编写更智能的程序，以减少运行时的错误。

编写复杂程序通常需要使用变量、条件、循环和测试等功能。例如，如果我们希望修改这个脚本，使其在尝试创建新文件和目录之前检查磁盘是否有足够的空间，怎么办？或者如果我们能够检查目录和文件创建操作是否成功呢？本节和第二章将介绍你完成这些任务所需的语法元素。

### 变量

每种脚本语言都有变量。*变量*是我们分配给内存位置并存储值的名称；它们充当占位符或标签。我们可以直接为变量赋值，或者可以执行 bash 命令并将其输出存储为变量值，以供各种用途。

如果你曾接触过编程语言，你可能知道变量可以有不同的类型，如整数、字符串和数组。在 bash 中，变量是无类型的；它们都被视为字符字符串。尽管如此，你会看到 bash 允许你创建数组、访问数组元素，或执行算术运算，只要变量值仅由数字组成。

以下规则规范了 bash 变量的命名：

+   它们可以包含字母数字字符。

+   变量名不能以数字开头。

+   变量名可以包含下划线（_）。

+   变量名不能包含空格。

#### 变量赋值与访问

让我们来赋值一个变量。打开终端并在命令提示符下直接输入以下内容：

```
$ **book="black hat bash"**
```

我们创建了一个名为 book 的变量，并使用等号（=）将值“black hat bash”赋给它。现在我们可以在命令中使用这个变量。在以下示例中，我们使用 echo 命令将该变量打印到屏幕上：

```
$ **echo "This book's name is ${book}"**
This book's name is black hat bash 
```

在这里，我们通过在 echo 命令中使用${book}语法打印变量。这将扩展 book 变量的值。你也可以通过仅使用美元符号（$）后跟变量名来扩展变量：

```
$ **echo "This book's name is $book"**
```

使用${}语法可以使代码更不容易被误解，并帮助读者理解变量的开始和结束。

你还可以通过使用命令替换语法$()，将命令的输出赋值给变量，将所需的命令放在括号内。你将在 bash 编程中经常使用这种语法。尝试运行清单 1-10 中的命令。

```
$ **root_directory=$(ls -ld /)**
$ **echo "${root_directory}"**

drwxr-xr-x 1 user user 0 Feb 13 20:12 / 
```

清单 1-10：将命令输出赋值给变量

我们将命令`ls -ld /`的输出赋值给名为 root_directory 的变量，然后使用 echo 打印该命令的输出。在这个输出中，你可以看到我们能够获取有关根目录（/）的元数据，如其类型和权限、大小、用户和组所有者，以及最后修改的时间戳。

请注意，在创建变量时，赋值符号（=）两边不应有空格：

```
book = "this is an invalid variable assignment"
```

上述的变量赋值语法被认为是无效的。

#### 取消赋值变量

你可以通过使用 unset 命令来取消已赋值的变量，如清单 1-11 所示。

```
$ **book="Black Hat Bash"**
$ **unset book**
$ **echo "${book}"** 
```

清单 1-11：取消变量赋值

如果你在终端中执行这些命令，echo 命令执行后将不会显示任何输出。

#### 变量作用域

*全局*变量是整个程序都能访问的变量。但是，bash 中的变量也可以是*作用域*限定的，只能在特定的代码块内访问。这些*局部*变量是通过使用 local 关键字声明的。清单 1-12 中的脚本展示了局部和全局变量是如何工作的。

local_scope _variable.sh

```
#!/bin/bash

PUBLISHER="No Starch Press"

print_name(){
   local name
   name="Black Hat Bash"
   echo "${name} by ${PUBLISHER}"
}

print_name

echo "Variable ${name} will not be printed because it is a local variable." 
```

清单 1-12：访问全局和局部变量

我们将值“No Starch Press”赋给变量 PUBLISHER，然后创建一个名为 print_name()的函数。（你将在下一章学习函数。）在这个函数内，我们声明一个名为 name 的局部变量，并赋值为“Black Hat Bash”。然后我们调用 print_name()并尝试将 name 变量作为一句话的一部分通过 echo 打印出来。

脚本文件末尾的 echo 命令将导致空变量，因为 name 变量的作用域仅限于 print_name()函数，这意味着函数外部无法访问它。因此，它将直接返回一个空值。

> 注意

*本章中的脚本可通过以下链接获取*：[`github.com/dolevf/Black-Hat-Bash/blob/master/ch01`](https://github.com/dolevf/Black-Hat-Bash/blob/master/ch01)。

保存此脚本，记得使用 chmod 设置可执行权限，并使用以下命令运行它：

```
$ **./local_scope_variable.sh**

Black Hat Bash by No Starch Press

Variable  will not be printed here because it is a local variable 
```

如你所见，本地变量从未被打印。

### 算术运算符

*算术运算符*允许你对整数执行数学运算。表 1-1 显示了一些可用的算术运算符。完整的列表请参见 *[`tldp.org/LDP/abs/html/ops.html`](https://tldp.org/LDP/abs/html/ops.html)*。

表 1-1：算术运算符

| 运算符 | 描述 |
| --- | --- |
| + | 加法 |
| - | 减法 |
| * | 乘法 |
| / | 除法 |
| % | 取模 |
| += | 常量递增 |
| -= | 常量递减 |

你可以通过几种方式在 bash 中执行这些算术运算：使用 let 命令、使用双括号语法 $((expression))，或者使用 expr 命令。让我们来看每种方法的一个例子。

在清单 1-13 中，我们通过使用 let 命令执行了一个乘法运算。

```
$ **let result="4 * 5"**
$ **echo ${result}**

20 
```

清单 1-13：使用 let 进行算术运算

此命令接收一个变量名，并执行算术计算以求解其值。在清单 1-14 中，我们使用双括号语法执行了另一个乘法运算。

```
$ **result=$((5 * 5))**
$ **echo ${result}**

25 
```

清单 1-14：使用双括号语法进行算术运算

在这种情况下，我们在双括号内执行计算。最后，在清单 1-15 中，我们使用 expr 命令执行了加法运算。

```
$ **result=$(expr 5 + 505)**
$ **echo ${result}**

510 
```

清单 1-15：使用 expr 计算表达式

expr 命令用于计算表达式，这些表达式不一定是算术运算；例如，你可以用它来计算字符串的长度。使用 man expr 了解 expr 的更多功能。

### 数组

Bash 允许你创建一维数组。一个 *数组* 是一个由元素组成的集合，这些元素是有索引的。你可以通过使用它们的索引编号来访问这些元素，索引从零开始。在 bash 脚本中，每当你需要迭代多个字符串并对每个字符串运行相同的命令时，可能会使用数组。

清单 1-16 展示了如何在 bash 中创建一个数组。将此代码保存为名为 *array.sh* 的文件并执行。

```
#!/bin/bash

# Sets an array
IP_ADDRESSES=(192.168.1.1 192.168.1.2 192.168.1.3)

# Prints all elements in the array
echo "${IP_ADDRESSES[*]}"

# Prints only the first element in the array
echo "${IP_ADDRESSES[0]}" 
```

清单 1-16：创建和访问数组

此脚本使用名为 IP_ADDRESSES 的数组，其中包含三个互联网协议（IP）地址。第一个 echo 命令通过将 [*] 传递给变量名 IP_ADDRESSES（它保存了数组值）来打印数组中的所有元素。星号（*）表示每个数组元素。最后，另一个 echo 命令通过指定索引 0 来打印数组中的第一个元素。

运行此脚本应产生以下输出：

```
$ **chmod u+x array.sh**
$ **./array.sh**

192.168.1.1 192.168.1.2 192.168.1.3
192.168.1.1 
```

如你所见，我们能够让 bash 打印数组中的所有元素，也只打印第一个元素。

你也可以从数组中删除元素。列表 1-17 会删除数组中的 192.168.1.2。

```
IP_ADDRESSES=(192.168.1.1 192.168.1.2 192.168.1.3)

unset IP_ADDRESSES[1] 
```

列表 1-17：删除数组元素

你甚至可以用另一个值替换其中一个值。此代码将 192.168.1.1 替换为 192.168.1.10：

```
IP_ADDRESSES[0]="192.168.1.10"
```

当你需要迭代值并对其执行操作时，数组会特别有用，比如扫描的 IP 地址列表（或要发送钓鱼邮件的电子邮件地址列表）。

### 流

*流*是充当程序与其环境之间通信通道的文件。当你与程序交互时（无论是内建的 Linux 工具，如 ls 或 mkdir，还是你自己编写的程序），你都在与一个或多个流交互。Bash 有三个标准数据流，如表格 1-2 所示。

表 1-2：流

| 流名称 | 描述 | 文件描述符编号 |
| --- | --- | --- |
| 标准输入（stdin） | 输入到程序中的数据 | 0 |
| 标准输出（stdout） | 从程序输出的数据 | 1 |
| 标准错误（stderr） | 程序输出的错误信息 | 2 |

到目前为止，我们已经从终端运行了一些命令，并编写并执行了一个简单的脚本。生成的输出被发送到了*标准输出流（stdout）*，换句话说，就是你的终端屏幕。

脚本也可以接收命令作为输入。当脚本被设计为接收输入时，它会从*标准输入流（stdin）*中读取输入。最后，脚本可能会由于命令中的 bug 或语法错误而向屏幕显示错误信息。这些信息会被发送到*标准错误流（stderr）*。

为了说明流的概念，我们将使用 mkdir 命令来创建一些目录，然后使用 ls 来列出当前目录的内容。打开终端并执行以下命令：

```
$ **mkdir directory1 directory2 directory1**
mkdir: cannot create directory 'directory1': File exists

$ **ls -l**
total 1
drwxr-xr-x 1 user user   0 Feb 17 09:45 directory1
drwxr-xr-x 1 user user   0 Feb 17 09:45 directory2 
```

注意到 mkdir 生成了一个错误。这是因为我们在命令行中将目录名称*directory1*传递了两次。因此，当 mkdir 执行时，它创建了*directory1*和*directory2*，然后在第三个参数时失败，因为此时*directory1*已经被创建。这类错误会被发送到标准错误流。

接下来，我们执行 ls -l，这只是列出目录。ls 命令的结果成功执行，没有特定的错误，因此它被发送到标准输出流。

你将在我们介绍重定向时练习标准输入流的使用，具体内容在“重定向操作符”部分，见第 18 页。

### 控制操作符

Bash 中的*控制操作符*是执行控制功能的标记。表 1-3 概述了控制操作符。

表 1-3：Bash 控制操作符

| 操作符 | 描述 |
| --- | --- |
| & | 将命令发送到后台。 |
| && | 用作逻辑与。表达式中的第二个命令只有在第一个命令的结果为真时才会被执行。 |
| (和) | 用于命令分组。 |
| ; | 用作列表终结符。一个命令在终结符后将运行，在前一个命令完成后，无论它的返回值是否为 true。 |
| ;; | 结束一个 case 语句。 |
| &#124; | 将一个命令的输出重定向为另一个命令的输入。 |
| &#124;&#124; | 用作逻辑 OR。第二个命令将在第一个命令返回 false 时运行。 |

让我们来看一下这些控制操作符的实际应用。& 操作符将命令发送到后台。如果你有一个命令列表要运行，如列表 1-18 所示，将第一个命令发送到后台可以让 bash 继续执行下一行，即使前一个命令尚未完成其工作。

```
#!/bin/bash

# This script will send the sleep command to the background.
echo "Sleeping for 10 seconds..."
❶ sleep 10 &

# Creates a file
echo "Creating the file test123"
touch test123

# Deletes a file
echo "Deleting the file test123"
rm test123 
```

列表 1-18：将命令发送到后台，以便执行可以移动到下一行

长时间运行的命令通常会被发送到后台，以防止脚本挂起 ❶。当我们在第二章中讨论作业控制时，你将更深入地了解如何将命令发送到后台。

&& 操作符允许我们在两个命令之间执行 AND 操作。在以下示例中，只有第一个命令成功时，文件 *test123* 才会被创建：

```
touch test && touch test123
```

() 操作符允许我们将多个命令分组，以便在需要一起重定向时将它们视为一个整体：

```
(ls; ps)
```

这通常在你需要将多个命令的结果重定向到一个流时非常有用，正如接下来的“重定向操作符”中所示。

; 操作符允许我们运行多个命令，而不管它们的退出状态：

```
ls; ps; whoami
```

结果是，每个命令都会在前一个命令完成后一个接一个地执行。

|| 操作符允许我们通过 OR 操作将命令连接在一起：

```
lzl || echo "the lzl command failed"
```

在这个例子中，echo 命令只有在第一个命令失败时才会执行。

### 重定向操作符

我们之前提到的三个标准流可以从一个程序重定向到另一个程序。*重定向* 是将一个命令或脚本的输出作为另一个脚本或文件的输入，用于写入目的。表 1-4 描述了可用的重定向操作符。

表 1-4：重定向操作符

| 操作符 | 描述 |
| --- | --- |
| > | 将 stdout 重定向到一个文件 |
| >> | 将 stdout 重定向到一个文件，并将内容追加到现有内容中 |
| &> 或 >& | 将 stdout 和 stderr 重定向到文件 |
| &>> | 将 stdout 和 stderr 重定向到一个文件，并追加到现有内容中 |
| < | 将输入重定向到一个命令 |
| << | 被称为 here 文档或 heredoc，将多行输入重定向到一个命令 |
| &#124; | 将一个命令的输出重定向为另一个命令的输入 |

让我们练习使用重定向操作符，看看它们如何与标准流一起工作。> 操作符将标准输出流重定向到一个文件。任何在此字符之前的命令都会将其输出发送到指定的位置。直接在终端中运行以下命令：

```
$ **echo "Hello World!" > output.txt**
```

我们将标准输出流重定向到一个名为 *output.txt* 的文件中。要查看 *output.txt* 的内容，只需运行以下命令：

```
$ **cat output.txt**

Hello World! 
```

接下来，我们将使用 >> 操作符将一些内容附加到同一个文件的末尾（见 Listing 1-19）。

```
$ **echo "Goodbye!" >> output.txt**
$ **cat output.txt**

Hello World!
Goodbye! 
```

Listing 1-19：将内容附加到文件中

如果我们使用 > 而不是 >>，*output.txt* 的内容将被 Goodbye! 文本完全覆盖。

你可以通过使用 &> 将标准输出流和标准错误流都重定向到一个文件中。当你不希望将任何输出发送到屏幕，而是将所有内容保存到日志文件中（可能供以后分析）时，这非常有用：

```
$ **ls -l / &> stdout_and_stderr.txt**
```

要将标准输出流和标准错误流都附加到文件中，可以使用与符号后跟双箭头 (&>>)。

如果我们想将标准输出流发送到一个文件，而将标准错误流发送到另一个文件呢？使用流的文件描述符编号，也可以做到这一点：

```
$ **ls -l / 1> stdout.txt 2> stderr.txt**
```

有时，你可能会发现将标准错误流重定向到文件中非常有用，就像我们在这里所做的那样，这样你可以记录运行时发生的任何错误。下一个示例运行了一个不存在的命令 lzl。这应该会生成 bash 错误，并将其写入 *error.txt* 文件：

```
$ **lzl 2> error.txt**
$ **cat error.txt**

bash: lzl: command not found 
```

请注意，屏幕上看不到错误信息，因为 bash 将错误信息发送到了文件中。

接下来，我们使用标准输入流。在 shell 中运行 Listing 1-20 中的命令，将 *output.txt* 的内容作为输入提供给 cat 命令。

```
$ **cat < output.txt**

Hello World!
Goodbye! 
```

Listing 1-20：使用文件作为命令的输入

如果我们想将多行内容重定向到一个命令呢？此时，可以使用文档重定向（<<）来帮助完成这项操作（见 Listing 1-21）。

```
$ **cat << EOF**
 **Black Hat Bash**
 **by No Starch Press**
**EOF**

Black Hat Bash
by No Starch Press 
```

Listing 1-21：文档重定向

在这个示例中，我们将多行作为输入传递给一个命令。示例中的 EOF 起到了分隔符的作用，标记了输入的起始和结束位置。*文档重定向* 将输入当作独立文件处理，保留了换行符和空格。

*管道* 操作符 (|) 将一个命令的输出重定向，并将其作为另一个命令的输入。例如，我们可以在根目录下运行 ls 命令，然后使用另一个命令从中提取数据，如 Listing 1-22 所示。

```
$ **ls -l / | grep "bin"**

lrwxrwxrwx   1 root root          7 Mar 10 08:43 bin -> usr/bin
lrwxrwxrwx   1 root root          8 Mar 10 08:43 sbin -> usr/sbin 
```

Listing 1-22：将命令输出通过管道传递到另一个命令

我们使用 ls 打印根目录的内容到标准输出流中，然后使用管道将其作为输入传递给 grep 命令，后者会过滤掉包含 *bin* 的行。

### 位置参数

Bash 脚本可以接收 *位置参数*（也称为 *参数*），这些参数由命令行传递给脚本。参数尤其有用，例如，当你希望开发一个程序，使其根据其他程序或用户传递的输入改变行为时。参数还可以改变脚本的特性，比如输出格式或运行时的详细程度。

例如，假设你开发了一个漏洞并将其发送给几个同事，他们每个人将针对不同的 IP 地址使用它。你可以编写一个脚本，接受 IP 地址参数，然后根据该输入执行操作，避免在每次情况下都需要修改源代码。

一个 bash 脚本可以使用变量$1、$2 等来访问传递给它的命令行参数。数字代表参数输入的顺序。为了说明这一点，列表 1-23 中的脚本接受一个参数（IP 地址或域名），并使用 ping 工具对其执行 ping 测试。将此文件保存为*ping_with_arguments.sh*。

ping_with_arguments.sh

```
#!/bin/bash

# This script will ping any address provided as an argument.

SCRIPT_NAME="${0}"
TARGET="${1}"

echo "Running the script ${SCRIPT_NAME}..."
echo "Pinging the target: ${TARGET}..."
ping "${TARGET}" 
```

列表 1-23：一个接受命令行输入的脚本

这个脚本将第一个位置参数赋值给变量 TARGET。同时，还要注意，参数${0}被赋值给 SCRIPT_NAME 变量。这个参数包含脚本的名称（在本例中是*ping_with_arguments.sh*）。

要运行此脚本，请使用列表 1-24 中的命令。

```
$ **chmod u+x ping_with_arguments.sh**
$ **./ping_with_arguments.sh nostarch.com**

Running the script ping_with_arguments.sh...
Pinging the target nostarch.com...
PING nostarch.com (104.20.120.46) 56(84) bytes of data.

64 bytes from 104.20.120.46 (104.20.120.46): icmp_seq=1 ttl=57 time=6.89 ms
64 bytes from 104.20.120.46 (104.20.120.46): icmp_seq=2 ttl=57 time=4.16 ms
`--snip--` 
```

列表 1-24：将参数传递给脚本

这个脚本将执行针对命令行传递的域名*nostarch.com*的 ping 命令。该值被赋值给$1 变量；如果传递了另一个参数，它将被赋值给第二个变量$2。使用 CTRL-C 退出此脚本，因为在某些操作系统上，ping 命令可能会无限运行。

如果你想访问所有参数怎么办？你可以使用变量$@来实现。另外，使用$#，你可以获取传递的参数总数。列表 1-25 展示了这一过程是如何工作的。

```
#!/bin/bash

echo "The arguments are: $@"
echo "The total number of arguments is: $#" 
```

列表 1-25：检索所有参数及参数总数

将此脚本保存为名为*show_args.sh*的文件，并按如下方式运行：

```
$ **chmod u+x show_args.sh**
$ **./show_args.sh "hello" "world"**

The arguments are: hello world
The total number of arguments is: 2 
```

表格 1-5 总结了与位置参数相关的变量。

表格 1-5：与位置参数相关的特殊变量

| 变量 | 描述 |
| --- | --- |
| $0 | 脚本文件的名称 |
| $1, $2, $3, ... | 位置参数 |
| $# | 传递的定位参数的数量 |
| $* | 所有位置参数 |
| $@ | 所有位置参数，每个参数单独引用 |

当脚本使用"$*"（包括引号）时，bash 会将所有参数展开成一个单一的词。例如，以下示例将参数组合成一个词：

```
$ **./script.sh "1" "2" "3"**
1 2 3 
```

当脚本使用"$@"（包括引号）时，它会将参数展开成单独的词：

```
$ **./script.sh "1" "2" "3"**
1
2
3 
```

在大多数情况下，你会想使用"$@"，这样每个参数都会被当作一个独立的词来处理。

以下脚本演示了如何在 for 循环中使用这些特殊变量：

```
#!/bin/bash
# Change "$@" to "$*" to observe behavior.
for args in "$@"; do
    echo "${args}"
done 
```

### 输入提示

一些 bash 脚本在执行时不接受任何参数。然而，它们可能需要以交互方式向用户请求信息，并将响应输入到其运行时中。在这些情况下，我们可以使用 read 命令。在尝试安装软件时，经常会看到应用程序使用 *输入提示* 要求用户输入 *yes* 以继续或 *no* 以取消操作。

在 Listing 1-26 中的 bash 脚本中，我们要求用户输入他们的名和姓，然后将这些信息打印到标准输出流中。

input _prompting.sh

```
#!/bin/bash

# Takes input from the user and assigns it to variables
echo "What is your first name?"
read -r firstname

echo "What is your last name?"
read -r lastname

echo "Your first name is ${firstname} and your last name is ${lastname}" 
```

Listing 1-26: 向用户询问输入

将此脚本保存并运行为 *input_prompting.sh*：

```
$ **chmod u+x input_prompting.sh**
$ **./input_prompting.sh**

What is your first name?
John

What is your last name?
Doe

Your first name is John and your last name is Doe 
```

请注意，系统会提示您输入信息，然后将其打印出来。

### 退出码

Bash 命令返回 *退出码*，这些代码指示命令的执行是否成功。退出码范围在 0 到 255 之间，其中 0 表示成功，1 表示失败，126 表示找到命令但不可执行，127 表示未找到命令。任何其他数字的含义取决于使用的具体命令及其逻辑。

#### 检查退出码

要查看退出码的实际效果，请将 Listing 1-27 中的脚本保存到名为 *exit_codes.sh* 的文件中并运行它。

```
#!/bin/bash

# Experimenting with exit codes

ls -l > /dev/null
echo "The exit code of the ls command was: $?"

lzl 2> /dev/null
echo "The exit code of the non-existing lzl command was: $?" 
```

Listing 1-27: 使用退出码确定命令的成功

我们使用 echo 命令与特殊变量 $? 来返回执行命令 ls 和 lzl 的退出码。我们还将它们的标准输出和标准错误流重定向到文件 */dev/null*，这是一个特殊设备文件，用于丢弃发送到它的任何数据。当您希望静音命令时，可以将其输出重定向到此处。

您应该会看到如下输出：

```
$ **./exit_codes.sh**

The exit code of the ls command was: 0
The exit code of the non-existing lzl command was: 127 
```

我们收到两个不同的退出码，一个用于每个命令。第一个命令返回 0（成功），第二个返回 127（未找到命令）。

> 警告

*谨慎使用* /dev/null *。如果选择将输出重定向到其中，您可能会错过重要的错误信息。如果有疑问，请将标准输出和标准错误流重定向到专用日志文件中。*

要了解为什么要使用退出码，请想象一下，您正在尝试使用 bash 从互联网下载 1GB 文件。在尝试下载之前，先检查文件系统中是否已存在该文件可能是明智的选择。此外，您可能需要检查磁盘上是否有足够的空闲空间。通过运行命令并查看它们返回的退出码，您可以决定是否继续进行文件下载。

#### 设置脚本的退出码

您可以通过使用 exit 命令后跟代码编号来设置脚本的退出码，如 Listing 1-28 所示。

```
#!/bin/bash

# Sets the exit code of the script to be 223

echo "Exiting with exit code: 223"
exit 223 
```

Listing 1-28: 设置脚本的退出码

将此脚本保存为 *set_exit_code.sh* 并在命令行上运行它。然后使用特殊变量 $? 来查看它返回的退出码：

```
$ **chmod u+x set_exit_code.sh**
$ **./set_exit_code.sh**
Exiting with exit code: 223

**echo $?**
223 
```

你可以使用 `$?` 变量检查不仅是脚本的返回退出码，还可以检查单个命令的退出码：

```
$ **ps -ef**
$ **echo $?**

0 
```

退出码很重要；它们可以用于一系列相互调用的脚本或同一个脚本内，以控制代码执行的逻辑流程。

练习 1：记录你的姓名和日期

编写一个脚本，实现以下功能：

1. 接受命令行上的两个参数，并将它们赋值给变量。第一个参数应为你的名字，第二个应为你的姓氏。

2. 创建一个名为 *output.txt* 的新文件。

3. 使用 `date` 命令将当前日期写入 *output.txt*。（如果你能让 `date` 命令以 DD-MM-YYYY 格式输出日期，可以获得加分；使用 man date 了解如何实现。）

4. 将你的全名写入 *output.txt*。

5. 使用 `cp` 命令制作 *output.txt* 的备份副本，命名为 *backup.txt*。（如果你不确定命令的语法，可以使用 man cp 查阅帮助。）

6. 将 *output.txt* 文件的内容打印到标准输出流。

你可以在本书的 GitHub 仓库中找到一个示例解答，名为 *exercise_solution.sh*。

## 总结

在本章中，你在终端中运行了简单的 Linux 命令，并使用 man 学习了命令选项。你还学习了如何向脚本传递参数并从脚本中执行一系列命令。我们讲解了 bash 的基础知识，比如如何编写使用变量、数组、重定向、退出码和参数的基本程序。你还学会了如何提示用户输入任意信息，并将其作为脚本流程的一部分使用。
