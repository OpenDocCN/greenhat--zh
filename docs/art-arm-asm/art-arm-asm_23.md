

## 第十九章：C 安装和使用 GAS



![](img/opener.jpg)

要在你的计算机上编译 ARM 汇编语言源文件（包括本书中的文件），你需要安装 Gas 汇编器和其他工具。汇编器是 GNU C 编译器套件的一部分，因此如果你在系统上安装了该软件包，你也会获得汇编器。由于不同操作系统的安装步骤不同，本附录将帮助你在你的机器上找到并安装所需的文件。

在尝试安装任何软件之前，请使用以下命令检查该软件是否已安装：

```
as --version
gcc --help
```

如果这些命令打印出 Gas 和 GCC 的帮助信息，说明汇编器（和 GCC）已经在你的系统上安装好了，你可以继续进行。然而，如果其中任何一条命令提示文件未找到（或没有打印帮助信息），请跳到针对你操作系统的章节来安装 GCC 和 Gas。

## C.1 macOS

要在 Apple Silicon 机器（如 M*x* 类机器，甚至 iPad 或 iPhone）上编写汇编代码，你必须在 Mac 上安装 Apple 的 Xcode 开发平台。前往 Apple App Store，然后定位、下载并安装 Xcode。这是一个非常大的文件（多个 GB），下载需要一些时间。运行安装程序后，你应该能够编译本书中的汇编代码。

从技术上讲，当安装 Xcode 时，你实际上是在安装 Apple 的 LLVM Clang 编译器和 Clang 汇编器，而不是 GCC 和 Gas。然而，这些工具与 GCC 和 Gas 大多数是兼容的（并且完全能够处理本书中的所有源代码）。这两个汇编器有时会有语法差异，*aoaa.inc* 文件就是为了帮助弥合这些差异而创建的（如果你有兴趣查看一些差异，可以查看 *aoaa.inc* 的源代码）。本书中的各种评论也指出了 Gas 和 Clang 汇编器之间的差异（例如，参见第 3.8 节“获取内存对象的地址”，在 第 153 页有讨论 lea 宏）。

## C.2 Linux

许多 Linux 安装已经预装了 GCC 和 Gas。如果你使用的是 Debian/Ubuntu Linux 或 Raspberry Pi OS，在命令行中输入 gcc --version；如果 shell 没有报错而是打印出 GCC 的信息，说明一切就绪。

如果你确实需要安装 GCC 和 Gas，第一步与任何 Linux 安装一样（本书假设使用 Debian/Ubuntu 安装），是更新你的系统：

```
sudo apt update
```

请注意，这需要具有 sudo 权限的账户。

接下来，使用以下命令安装 *build-essential* 包：

```
sudo apt install build-essential
```

这将安装多个程序，包括 gcc、Gas、make 和 ld。通过以下命令验证系统是否已正确安装这些工具：

```
gcc --version
```

如果 GCC 已正确安装，这应会打印出其版本信息。

可选地，你可以通过以下命令安装 GCC 和其他工具的手册页：

```
sudo apt-get install manpages-dev
```

如果你使用的是其他版本的 Linux，请查阅你所在发行版的文档或在线搜索安装说明来安装 GCC。
