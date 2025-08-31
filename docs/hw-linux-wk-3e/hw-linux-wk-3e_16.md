# 第十六章：从 C 源代码编译软件的简介

![](img/chapterart.png)

大多数非专有的第三方 Unix 软件包以源代码形式提供，允许你构建和安装。这样做的一个原因是，Unix（以及 Linux 本身）有太多不同的版本和架构，难以为所有可能的平台组合分发二进制软件包。另一个至少同样重要的原因是，Unix 社区广泛分发源代码，鼓励用户为软件贡献错误修复和新功能，这也赋予了 *开源* 这个词意义。

你可以通过源代码获取 Linux 系统上几乎所有看到的内容——从内核和 C 库到网页浏览器。实际上，你甚至可以通过（重新）安装系统的一部分源代码来更新和增强你的整个系统。然而，除非你非常喜欢这个过程或者有其他原因，否则你可能*不应该*通过从源代码安装*所有*内容来更新你的机器。

Linux 发行版通常提供方便的方式来更新系统的核心部分，如 */bin* 中的程序，而发行版的一个特别重要的特点是它们通常能非常快速地修复安全问题。但不要指望你的发行版会为你提供所有内容。以下是你可能需要自己安装某些软件包的几个原因：

+   控制配置选项。

+   可以将软件安装到你喜欢的任何位置。你甚至可以安装同一个软件包的多个不同版本。

+   控制你安装的版本。不同的发行版并不总是与所有软件包的最新版本保持同步，尤其是软件包的附加组件（例如 Python 库）。

+   更好地理解某个软件包是如何工作的。

## 16.1 软件构建系统

在 Linux 上有许多编程环境，从传统的 C 语言到解释型脚本语言如 Python。每种环境通常都有至少一个独特的系统用于构建和安装软件包，除此之外还有 Linux 发行版提供的工具。

本章我们将介绍如何使用其中一个构建系统——由 GNU autotools 套件生成的配置脚本，来编译和安装 C 源代码。这个系统通常被认为是稳定的，许多基本的 Linux 工具都使用它。由于它是基于现有工具（如 `make`）构建的，因此当你看到它的实际操作后，你将能够将你的知识转移到其他构建系统上。

从 C 源代码安装软件包通常涉及以下步骤：

1.  解压源代码归档。

1.  配置软件包。

1.  运行 `make` 或其他构建命令来构建程序。

1.  运行 `make install` 或发行版特定的安装命令来安装软件包。

## 16.2 解压 C 源代码包

包的源代码分发通常是作为*.tar.gz*、*.tar.bz2*或*.tar.xz*文件提供的，你应该按照第 2.18 节中描述的方式解压该文件。然而，在解压之前，使用`tar tvf`或`tar ztvf`验证归档的内容，因为有些包不会在解压归档的目录中创建自己的子目录。

这样的输出意味着该包可能适合解压：

```
package-1.23/Makefile.in
package-1.23/README
package-1.23/main.c
package-1.23/bar.c
--`snip`--
```

然而，你可能会发现并非所有文件都位于一个公共目录中（就像前面的例子中的*package-1.23*）：

```
Makefile
README
main.c
--`snip`--
```

解压像这样的归档文件可能会在当前目录中留下很多杂乱的文件。为了避免这种情况，在解压归档内容之前，创建一个新目录并`cd`到该目录。

最后，注意包含绝对路径名的文件包，例如下面这样：

```
/etc/passwd
/etc/inetd.conf
```

你可能不会遇到类似这样的情况，但如果遇到，请从系统中删除该归档文件。它可能包含木马或其他恶意代码。

一旦你提取了源归档的内容，并且面前有一堆文件，试着对包的内容有个大致了解。特别是，查找名为*README*和*INSTALL*的文件。始终先查看任何*README*文件，因为它们通常包含包的描述、简短手册、安装提示和其他有用的信息。许多包还附带*INSTALL*文件，包含如何编译和安装该包的说明。特别注意特殊的编译器选项和定义。

除了*README*和*INSTALL*文件，你还会找到其他文件，这些文件大致可以分为三类：

+   与`make`系统相关的文件，例如*Makefile*、*Makefile.in*、*configure*和*CMakeLists.txt*。一些非常旧的包附带一个 Makefile，你可能需要修改它，但大多数包使用配置工具，如 GNU autoconf 或 CMake。它们附带一个脚本或配置文件（例如*configure*或*CMakeLists.txt*），帮助你根据系统设置和配置选项从*Makefile.in*生成 Makefile。

+   以*.c*、*.h*或*.cc*结尾的源代码文件。C 源代码文件可能出现在包目录的任何位置。C++源代码文件通常具有*.cc*、*.C*或*.cxx*后缀。

+   以*.o*结尾的目标文件或二进制文件。通常，源代码分发包中没有目标文件，但在某些罕见情况下，当包的维护者无法发布某些源代码时，你可能会找到目标文件，这时你需要做一些特别的操作才能使用这些目标文件。在大多数情况下，源代码分发包中的目标文件（或二进制可执行文件）意味着包的组织不够规范，因此你应该运行`make clean`，以确保进行一次全新的编译。

## 16.3 GNU Autoconf

尽管 C 源代码通常是相当可移植的，但每个平台的差异使得大多数软件包无法使用单一的 Makefile 进行编译。对此问题的早期解决方案是为每个操作系统提供单独的 Makefile，或者提供一个易于修改的 Makefile。这个方法演变成了基于对构建软件包所使用系统的分析，自动生成 Makefile 的脚本。

GNU autoconf 是一个流行的自动生成 Makefile 的系统。使用该系统的包会附带名为*configure*、*Makefile.in*和*config.h.in*的文件。*.in*文件是模板；其目的是运行`configure`脚本来发现系统的特性，然后在*.in*文件中进行替换，从而生成真实的构建文件。对最终用户来说，这很简单；只需运行`configure`来从*Makefile.in*生成 Makefile：

```
$ **./configure**
```

在脚本检查你的系统是否满足前提条件时，你应该会看到很多诊断输出。如果一切顺利，`configure`将生成一个或多个 Makefile 和一个*config.h*文件，以及一个缓存文件（*config.cache*），以便它在以后不需要再次运行某些测试。

现在你可以运行`make`来编译软件包。成功的`configure`步骤并不意味着`make`步骤一定会成功，但成功的几率相当大。（有关解决配置和编译失败的提示，请参见第 16.6 节。）

让我们亲自体验一下这个过程。

### 16.3.1  一个 Autoconf 示例

在讨论如何更改 autoconf 的行为之前，让我们先看一个简单的示例，以便了解你可以期待什么。你将会在自己的主目录中安装 GNU coreutils 包（以确保不会破坏系统）。从[`ftp.gnu.org/gnu/coreutils/`](http://ftp.gnu.org/gnu/coreutils/)获取该包（最新版本通常是最好的），解压缩它，进入其目录并像下面这样进行配置：

```
$ ./**configure --prefix=$HOME/mycoreutils**
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
--`snip`--
config.status: executing po-directories commands
config.status: creating po/POTFILES
config.status: creating po/Makefile
```

现在运行`make`：

```
$ **make**
  GEN      lib/alloca.h
  GEN      lib/c++defs.h
--`snip`--
make[2]: Leaving directory '/home/juser/coreutils-8.32/gnulib-tests'
make[1]: Leaving directory '/home/juser/coreutils-8.32'
```

接下来，尝试运行你刚刚创建的其中一个可执行文件，例如*./src/ls*，并尝试运行`make check`以对软件包进行一系列测试。（这可能需要一些时间，但看看结果还是挺有趣的。）

最后，你准备好安装软件包了。首先使用`make -n`进行干运行，查看`make install`会做什么，而不实际执行安装：

```
$ **make -n install**
```

浏览输出内容，如果没有什么奇怪的地方（例如软件包安装到*mycoreutils*目录以外的位置），就可以正式安装了：

```
$ **make install**
```

现在，你应该在你的主目录下拥有一个名为*mycoreutils*的子目录，其中包含*bin*、*share*和其他子目录。查看一下*bin*中的一些程序（你刚刚构建了许多你在第二章学习的基础工具）。最后，由于你已经将*mycoreutils*目录配置为独立于你系统的其他部分，你可以完全删除它，而不必担心会造成损坏。

### 16.3.2  使用打包工具进行安装

在大多数发行版中，可以将新软件安装为一个包，之后可以使用发行版的打包工具进行维护。基于 Debian 的发行版，如 Ubuntu，可能是最容易的；你不是运行简单的`make install`，而是使用`checkinstall`工具来安装软件包，如下所示：

```
# checkinstall make install
```

运行此命令将显示与即将构建的包相关的设置，并提供更改它们的机会。安装时，`checkinstall`会跟踪所有要安装到系统上的文件，并将它们放入*.deb*文件中。然后你可以使用`dpkg`安装（或移除）新包。

创建 RPM 包稍微复杂一些，因为你必须首先为你的包创建一个目录树。你可以使用`rpmdev-setuptree`命令来完成这一步；完成后，你可以使用`rpmbuild`工具继续进行其余的步骤。最好根据在线教程进行此过程。

### 16.3.3  配置脚本选项

你刚刚看到的是`configure`脚本最有用的选项之一：使用`--prefix`来指定安装目录。默认情况下，自动生成的 Makefile 中的`install`目标使用*/usr/local*作为*prefix*—即，二进制程序放在*/usr/local/bin*，库文件放在*/usr/local/lib*，以此类推。你通常会想要更改这个前缀，方法如下：

```
$ **./configure --prefix=**`new_prefix`
```

大多数版本的`configure`都有一个`--help`选项，可以列出其他配置选项。不幸的是，列表通常很长，有时很难弄清楚哪些选项可能重要，因此这里列出了一些基本的选项：

1.  `--bindir=``directory` 将可执行文件安装到`directory`中。

1.  `--sbindir=``directory` 将系统可执行文件安装到`directory`中。

1.  `--libdir=``directory` 将库安装到`directory`。

1.  `--disable-shared` 防止构建共享库。根据库的不同，这可以避免后续出现麻烦（参见第 15.1.3 节）。

1.  `--with-``package``=``directory` 告诉`configure`，`package`位于`directory`中。当必要的库位于非标准位置时，这个选项非常有用。不幸的是，并非所有的`configure`脚本都支持这种类型的选项，而且确定确切的语法可能很困难。

### 16.3.4  环境变量

你可以通过环境变量影响`configure`，因为`configure`脚本会将这些变量传递给`make`。最重要的环境变量包括`CPPFLAGS`、`CFLAGS`和`LDFLAGS`。但需要注意的是，`configure`对环境变量非常挑剔。例如，你通常应该使用`CPPFLAGS`而不是`CFLAGS`来指定头文件目录，因为`configure`经常在独立于编译器的情况下运行预处理器。

在`bash`中，发送环境变量给`configure`的最简单方法是将变量赋值放在命令行的`./configure`之前。例如，要为预处理器定义一个`DEBUG`宏，可以使用以下命令：

```
**$ CPPFLAGS=-DDEBUG ./configure**
```

你还可以将一个变量作为选项传递给`configure`；例如：

```
**$ ./configure CPPFLAGS=-DDEBUG**
```

环境变量在`configure`不知道在哪里查找第三方包含文件和库时特别有用。例如，要让预处理器在`include_dir`中查找，运行以下命令：

```
$ **CPPFLAGS=-I**`include_dir` **./configure**
```

如第 15.2.6 节所示，要让链接器在`lib_dir`中查找，使用以下命令：

```
$ **LDFLAGS=-L**`lib_dir` **./configure**
```

如果`lib_dir`中包含共享库（参见第 15.1.3 节），上面的命令可能无法设置运行时动态链接器路径。在这种情况下，除了使用`-L`外，还需要使用`-rpath`链接器选项：

```
$ **LDFLAGS="-L**`lib_dir` **-Wl,-rpath=**`lib_dir`**" ./configure**
```

设置变量时要小心。一个小小的错误可能会让编译器出错，导致`configure`失败。例如，假设你忘记了`-`，如这里所示：

```
$ **CPPFLAGS=I**`include_dir` **./configure**
```

这将产生如下错误：

```
configure: error: C compiler cannot create executables
See 'config.log' for more details
```

从这个失败的尝试生成的*config.log*中查看，结果如下：

```
configure:5037: checking whether the C compiler works
configure:5059: gcc  Iinclude_dir  conftest.c  >&5
gcc: error: Iinclude_dir: No such file or directory
configure:5063: $? = 1
configure:5101: result: no
```

### 16.3.5  Autoconf 目标

一旦你让`configure`正常工作，你会发现它生成的 Makefile 除了标准的`all`和`install`之外，还有许多有用的目标：

1.  `make clean` 如第十五章所述，这个命令会删除所有目标文件、可执行文件和库文件。

1.  `make distclean` 这与`make clean`类似，除了它会删除所有自动生成的文件，包括 Makefile、*config.h*、*config.log*等。其目的是让源代码树在运行`make distclean`后看起来像是一个刚解压的分发包。

1.  `make check` 一些软件包附带一系列测试，用于验证编译后的程序是否正常工作；`make check`命令会运行这些测试。

1.  `make install-strip` 这与`make install`类似，除了它在安装时会去除可执行文件和库中的符号表和其他调试信息。去除调试信息的二进制文件占用更少的空间。

### 16.3.6  Autoconf 日志文件

如果在配置过程中出了问题，且原因不明显，你可以查看*config.log*以找到问题所在。不幸的是，*config.log*通常是一个非常大的文件，这会让你很难找到问题的具体源头。

在这种情况下，通常的做法是转到*config.log*的最末尾（例如，在`less`中输入大写的 G），然后向上翻页，直到你看到问题。然而，日志的末尾依然会有很多内容，因为`configure`会将整个环境输出在那里，包括输出变量、缓存变量和其他定义。因此，最好不要直接向上翻页，而是先到达文件末尾，再向后搜索一个字符串，比如`for more details`或其他靠近`configure`失败输出末尾的文本片段。（记住，你可以在`less`中用`?`命令进行反向搜索。）很有可能，错误就在你搜索到的内容上方。

### 16.3.7  pkg-config

系统中大量的第三方库意味着将它们都放在一个公共位置可能会显得杂乱无章。然而，使用单独的前缀安装每个库可能会导致构建需要这些第三方库的包时出现问题。例如，如果你想编译 OpenSSH，你需要 OpenSSL 库。你如何告诉 OpenSSH 配置过程 OpenSSL 库的位置以及需要哪些库？

现在，许多库不仅使用`pkg-config`程序来公开它们的头文件和库的位置，还指定编译和链接程序所需的确切标志。语法如下：

```
$ **pkg-config** `options package1 package2 ...`
```

例如，要查找流行压缩库所需的库，你可以运行以下命令：

```
$ **pkg-config --libs zlib**
```

输出应该类似于以下内容：

```
-lz
```

要查看`pkg-config`知道的所有库，包括每个库的简要描述，请运行以下命令：

```
$ **pkg-config --list-all**
```

#### pkg-config 如何工作

如果你深入了解幕后，你会发现`pkg-config`通过读取以*.pc*结尾的配置文件来查找包信息。例如，这是 Ubuntu 系统上 OpenSSL 套接字库的*openssl.pc*文件（位于*/usr/lib/x86_64-linux-gnu/pkgconfig*）：

```
prefix=/usr
exec_prefix=${prefix}
libdir=${exec_prefix}/lib/x86_64-linux-gnu
includedir=${prefix}/include

Name: OpenSSL
Description: Secure Sockets Layer and cryptography libraries and tools
Version: 1.1.1f
Requires: 
Libs: -L${libdir} -lssl -lcrypto
Libs.private: -ldl -lz
Cflags: -I${includedir} exec_prefix=${prefix}
```

你可以修改这个文件，例如，通过将`-Wl,-rpath=${libdir}`添加到库标志中，以设置运行时库搜索路径。然而，更大的问题是`pkg-config`最初是如何找到*.pc*文件的。默认情况下，`pkg-config`会在其安装前缀的*lib/pkgconfig*目录中查找。例如，使用*/usr/local*前缀安装的`pkg-config`会在*/usr/local/lib/pkgconfig*目录中查找。

#### 如何在非标准位置安装 pkg-config 文件

不幸的是，默认情况下，`pkg-config`不会读取其安装前缀外的任何*.pc*文件。这意味着位于非标准位置的*.pc*文件，如*/opt/openssl/lib/pkgconfig/openssl.pc*，将无法被任何标准的`pkg-config`安装所访问。有两种基本方法可以使*.pc*文件在`pkg-config`安装前缀外可用：

+   从实际的*.pc*文件创建符号链接（或复制）到中央*pkgconfig*目录。

+   设置`PKG_CONFIG_PATH`环境变量，将任何额外的*pkgconfig*目录包括在内。该策略在系统范围内效果不佳。

## 16.4 安装实践

知道*如何*构建和安装软件是好的，但知道*何时*和*在哪里*安装自己的包更有用。Linux 发行版尽量在安装时包含尽可能多的软件，因此你应该始终检查是否自己安装包会更好。自己安装的优点如下：

+   你可以自定义包的默认设置。

+   安装包时，你通常会更清楚如何使用它。

+   你可以控制你运行的版本。

+   备份自定义包更容易。

+   在网络中分发自安装的包更容易（前提是架构一致且安装位置相对隔离）。

以下是缺点：

+   如果你要安装的包已经安装在系统中，你可能会覆盖重要文件，导致问题。通过使用稍后会介绍的 */usr/local* 安装前缀可以避免这种情况。即使包没有安装在你的系统上，你也应该检查分发包是否可用。如果有，你需要记住这一点，以防你以后不小心安装了分发包。

+   这需要时间。

+   自定义包不会自动升级。分发包会保持大多数包的最新状态，且不需要你做太多工作。对于与网络交互的包，这是一个特别需要关注的问题，因为你希望确保始终拥有最新的安全更新。

+   如果你实际上并不使用这个包，那么你是在浪费时间。

+   存在错误配置包的潜在风险。

除非你正在构建一个非常自定义的系统，否则安装像本章早些时候构建的 coreutils 包（`ls`、`cat` 等）等包没什么意义。另一方面，如果你对像 Apache 这样的网络服务器有重要兴趣，最好的方式是自己安装这些服务器，从而获得完全的控制权。

### 16.4.1  安装位置

GNU autoconf 和许多其他包的默认前缀是 */usr/local*，这是本地安装软件的传统目录。操作系统升级时会忽略 */usr/local*，因此在操作系统升级过程中不会丢失那里安装的任何东西，对于小型本地软件安装，*/usr/local* 也足够了。唯一的问题是，如果你安装了大量自定义软件，可能会变得一团糟。成千上万的奇怪小文件可能会进入 */usr/local* 目录结构，你可能根本不知道这些文件来自哪里。

如果事情开始变得难以管理，你应该按照第 16.3.2 节中描述的方法创建你自己的包。

## 16.5 应用补丁

大多数软件源代码的更改作为开发者在线源代码版本的分支提供（例如 Git 仓库）。然而，偶尔你可能会得到一个需要应用于源代码的 *补丁*，用于修复错误或添加新功能。你也可能会看到 *diff* 这个术语作为补丁的同义词，因为 `diff` 程序生成补丁。

补丁的开始看起来像这样：

```
--- src/file.c.orig     2015-07-17 14:29:12.000000000 +0100
+++ src/file.c   2015-09-18 10:22:17.000000000 +0100
@@ -2,16 +2,12 @@
```

补丁通常包含对多个文件的更改。查找补丁中的三个连字符（`---`）以查看哪些文件有更改，并始终查看补丁的开头以确定所需的工作目录。注意，前面的示例提到的是 *src/file.c*。因此，在应用补丁之前，你应该切换到包含 *src* 的目录，而不是直接切换到 *src* 目录。

要应用补丁，运行`patch`命令：

```
$ **patch -p0 <** `patch_file`
```

如果一切顺利，`patch`会顺利退出，更新一组文件。然而，`patch`可能会问你这个问题：

```
File to patch: 
```

这通常意味着你不在正确的目录中，但也可能表示你的源代码与补丁中的源代码不匹配。在这种情况下，你可能就没有好运了。即使你能识别出一些文件需要打补丁，其他文件也无法正确更新，最终你将得到无法编译的源代码。

在某些情况下，你可能会遇到一个补丁，引用了类似这样的包版本：

```
--- package-3.42/src/file.c.orig     2015-07-17 14:29:12.000000000 +0100
+++ package-3.42/src/file.c   2015-09-18 10:22:17.000000000 +0100
```

如果你有一个稍微不同的版本号（或者只是更改了目录名称），你可以告诉`patch`去除路径中的前导部分。例如，假设你在包含*src*的目录中（如前所述）。要告诉`patch`忽略路径中的*package-3.42/*部分（即去除一个前导路径组件），使用`-p1`：

```
$ **patch -p1 <** `patch_file`
```

## 16.6 编译和安装故障排除

如果你理解编译器错误、编译器警告、链接器错误以及共享库问题的区别（如第十五章所述），你就不太会遇到在构建软件时出现的许多故障。此部分涵盖了一些常见问题。虽然在使用 autoconf 构建时不太可能遇到这些问题，但知道它们的表现方式总是有益的。

在讨论具体细节之前，确保你能读取某些类型的`make`输出。了解错误和被忽略的错误之间的区别非常重要。以下是一个需要调查的真实错误：

```
make: *** [`target`] Error 1
```

然而，一些 Makefile 知道某些错误条件可能会发生，但它们知道这些错误是无害的。你通常可以忽略类似的消息：

```
make: *** [`target`] Error 1 (ignored)
```

此外，GNU `make`在大型包中通常会多次调用自身，每次`make`实例的错误信息中会标记为`[``N``]`，其中`N`是一个数字。你通常可以通过查看编译器错误信息后*直接*跟随的`make`错误来快速找到问题。例如：

```
`compiler error message involving` file.c
make[3]: *** [file.o] Error 1
make[3]: Leaving directory '/home/src/package-5.0/src'
make[2]: *** [all] Error 2
make[2]: Leaving directory '/home/src/package-5.0/src'
make[1]: *** [all-recursive] Error 1
make[1]: Leaving directory '/home/src/package-5.0/'
make: *** [all] Error 2
```

这里的前三行提供了你需要的信息。问题集中在*file.c*，它位于*/home/src/package-5.0/src*。不幸的是，输出信息过多，可能很难找到重要的细节。学习如何过滤掉后续的`make`错误对于帮助你找到真正的原因非常有用。

### 16.6.1  具体错误

以下是你可能遇到的一些常见构建错误。

**问题**

1.  编译器错误信息：

    ```
    src.c:22: conflicting types for '`item`'
    /usr/include/`file`.h:47: previous declaration of '`item`'
    ```

**解释和修复**

1.  程序员在*src.c*的第 22 行错误地重新声明了`item`。你通常可以通过删除有问题的那一行（添加注释、`#ifdef`，或任何有效的方法）来修复此问题。

**问题**

1.  编译器错误信息：

    ```
    src.c:37: 'time_t' undeclared (first use this function)
    --`snip`--
    src.c:37: parse error before '...'
    ```

**解释和修复**

1.  程序员忘记了一个关键的头文件。手册页是查找缺失头文件的最佳方法。首先查看出错的行（在本例中是*src.c*中的第 37 行）。它可能是如下的变量声明：

    ```
    time_t v1;
    ```

    在程序中向前搜索`v1`，查看它在函数调用中的使用情况。例如：

    ```
    v1 = time(NULL);
    ```

    现在运行`man 2 time`或`man 3 time`，查找名为`time()`的系统和库调用。在这种情况下，第二部分手册页包含你需要的信息：

    ```
    SYNOPSIS
          #include <time.h>

          time_t time(time_t *t);
    ```

    这意味着`time()`需要`time.h`。在*src.c*的开头添加`#include <time.h>`并重新尝试。

**问题**

1.  编译器（预处理器）错误信息：

    ```
    src.c:4: `pkg`.h: No such file or directory
    (long list of errors follows)
    ```

**解释与修复**

1.  编译器对*src.c*运行了 C 预处理器，但无法找到*pkg.h*包含文件。源代码可能依赖于你需要安装的库，或者你可能只需要提供给编译器非标准的包含路径。通常，你只需要向 C 预处理器标志（`CPPFLAGS`）中添加`-I`包含路径选项。（请记住，你可能还需要一个`-L`链接器标志来配合包含文件一起使用。）

    如果看起来不是缺少库，可能是你正在尝试为一个此源代码不支持的操作系统进行编译。请检查 Makefile 和*README*文件，了解有关平台的详细信息。

    如果你使用的是基于 Debian 的发行版，尝试在头文件名上运行`apt-file`命令：

    ```
    $ `apt-file search` `pkg.h`
    ```

    这可能找到你需要的开发包。对于使用`yum`的发行版，你可以尝试改为使用以下命令：

    ```
    $ `yum provides */``pkg.h` 
    ```

**问题**

1.  `make`错误信息：

    ```
    make: `prog:` Command not found
    ```

**解释与修复**

1.  要构建该软件包，你的系统中需要安装`prog`。如果`prog`类似于`cc`、`gcc`或`ld`，说明你的系统没有安装开发工具。另一方面，如果你认为`prog`已经安装在系统上，尝试修改 Makefile，指定`prog`的完整路径名。

    在少数情况下，由于源代码配置不当，`make`构建了`prog`后立即使用`prog`，假设当前目录（`.`）在你的命令路径中。如果你的`$PATH`没有包含当前目录，你可以编辑 Makefile，将`prog`改为`./prog`。或者，你可以临时将`.`添加到你的路径中。

## 16.7 展望未来

我们仅触及了构建软件的基础知识。在你掌握了自己的构建方法后，尝试以下内容：

+   学习如何使用除 autoconf 以外的构建系统，如 CMake 和 SCons。

+   为你的软件设置构建环境。如果你在编写自己的软件，你需要选择一个构建系统并学习如何使用它。关于 GNU autoconf 打包，John Calcote 的《Autotools》第二版（No Starch Press, 2019）可以帮助你。

+   编译 Linux 内核。内核的构建系统与其他工具的构建系统完全不同。它拥有自己的配置系统，专门用于定制自己的内核和模块。尽管如此，过程相对直接，如果你理解引导加载程序是如何工作的，通常不会遇到任何问题。然而，在进行此操作时要小心；确保始终保留旧的内核，以防新内核无法启动。

+   探索特定发行版的源代码包。Linux 发行版维护自己版本的软件源代码，作为特殊的源代码包。有时你可以找到有用的补丁，扩展功能或修复其他未维护包中的问题。源代码包管理系统包括自动构建工具，如 Debian 的`debuild`和基于 RPM 的`mock`。

构建软件通常是学习编程和软件开发的第一步。你在本章和上一章中看到的工具揭开了系统软件来源的神秘面纱。向前迈出一步，查看源代码、进行修改并创建自己的软件并不困难。
