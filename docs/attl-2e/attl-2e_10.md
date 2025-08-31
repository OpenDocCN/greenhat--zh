## 第十章：使用 PKG-CONFIG 查找构建依赖

*人们在尝试设计完全防傻的东西时常犯的一个错误，就是低估了彻头彻尾的傻瓜的创造力。

—道格拉斯·亚当斯*，《银河系漫游指南》

![Image](img/common.jpg)

假设你的项目依赖于一个叫做 *stpl* 的库——一个第三方库。你的构建系统如何确定 *libstpl.so* 在终端用户系统上的安装位置？*stpl* 的头文件在哪里？你是不是仅仅假设它们在 */usr/lib* 和 */usr/include* 中？如果你没有告诉 Autoconf 去别的地方找，这实际上就是 Autoconf 的工作方式，对于许多软件包来说，这也许没问题——将库和头文件安装到这些目录是一个常见的约定。

但如果它们没有安装在这些位置呢？或者它们可能是本地构建并安装到 */usr/local* 目录树中。当你在项目中使用 *stpl* 时，应该使用哪些编译器和链接器选项？这些问题从一开始就困扰着开发人员，而 Autotools 并没有真正解决这个问题。Autoconf 期望你使用的任何库都被安装到“标准位置”，也就是预处理器和链接器自动查找头文件和库的目录。如果用户有这个库，但它安装在了其他位置，Autotools 就期望终端用户知道如何解释配置失败。事实上，有几个充分的理由解释了为什么许多库不会安装在这些标准位置。

此外，Autoconf 并不容易找到那些根本没有安装的库。例如，你可能刚刚构建了一个库包，而你希望另一个包从第一个包的构建目录结构中获取头文件和库。这是 Autoconf 可以做到的，但它需要终端用户在 `configure` 命令行中设置像 `CPPFLAGS` 和 `LDFLAGS` 这样的变量。项目维护者可以通过提供用户指定的配置选项，利用 `AC_ARG_ENABLE` 和 `AC_ARG_WITH` 宏来简化这一过程，特别是针对他们预期不容易在标准位置找到的库。但是，如果你的项目使用了大量的第三方库，那么确定哪些库对于用户来说特别棘手，实际上只是猜测。而且，用户通常不是程序员；我们不能指望他们有足够的背景知识来知道应该为选项值使用什么，即使我们提供了命令行选项来解决问题依赖。贯穿本章，我将这些问题称为 *构建依赖问题*。

有一种工具通过使用软件设计中非常常见的策略——提供另一层间接性，优雅地解决了这些问题。它并不属于 GNU Autotools 的一部分。尽管如此，过去 20 年来它的使用已经变得如此广泛，以至于如果在任何讨论 Linux 构建系统的书籍中不描述 *pkg-config*，那将是一个疏忽。pkg-config 之所以如此有用，正是因为许多项目开始使用它——尤其是库项目，特别是那些安装到非标准位置的库项目。

在本章中，我们将讨论 pkg-config 的组件和功能，以及如何在你的项目中使用它。对于你自己的库项目，我们还将讨论如何在你的包安装到用户（或者我喜欢称他们为潜在贡献者）系统时，更新 pkg-config 数据库。

在我们开始之前，请允许我提前说明，本章中涉及到的很多文件系统对象可能在你的 Linux 版本中并不存在，或者可能以不同的形式或位置存在。这是我们所做的事情的本质——我们讨论的是包含库和头文件的包，这些文件可能在不同的 Linux 版本中位于不同的位置。我会尽量指出这些潜在的差异，以便在两套系统中出现不完全一致时，你不会感到过于惊讶。

### pkg-config 概述

Pkg-config 是一个开源软件工具，由 *[freedesktop.org](http://freedesktop.org)* 项目维护。pkg-config 网站是 [freedesktop.org](http://freedesktop.org) 项目网站的一部分。pkg-config 项目是将 `gnome-config` 脚本——gnome 构建系统的一部分——转变为一个更通用工具的努力结果，而且它似乎得到了广泛应用。pkg-config 的灵感也来自于 `gtk-config` 程序。

关于 PKG-CONFIG 克隆的说明

[freedesktop.org](http://freedesktop.org) 的 pkg-config 项目已经存在很长时间，并且拥有一定数量的忠实用户。因此，现在有其他项目提供类似的功能——通常是完全相同的——与 pkg-config 项目类似。其中一个就是 pkgconf 项目（当你要求 `yum` 包管理器为你安装 pkg-config 时，Red Hat 的 Fedora Linux 似乎更偏向使用它）。

不要将这两者混淆。pkgconf 项目是原始 pkg-config 项目的现代克隆，声称具有更高的效率——就我而言，它很可能确实能够兑现这些声明。不管怎样，本章是关于 pkg-config 的。如果你发现某个项目提供类似于 pkg-config 的功能，并且你喜欢它，那么请随意使用它。我的目标是教你关于 pkg-config 的知识。如果这些内容帮助你理解如何使用 pkgconf 或其他 pkg-config 克隆项目，那你从本书中得到的正是我希望传达的内容。

然而，话虽如此，我不能涵盖不同 pkg-config 克隆之间所有细微的差异。如果你想跟随我的示例，但你的 Linux 版本无法为你安装原版 pkg-config 包，你总是可以通过浏览器访问*[`www.freedesktop.org/wiki/Software/pkg-config`](https://www.freedesktop.org/wiki/Software/pkg-config)*，下载并安装原版 pkg-config 项目的 0.29.2 版本。

使用 pkg-config 非常简单，只需调用`pkg-config`命令行工具，并使用显示所需数据的选项。当你寻找库和头文件时，“所需数据”包括包的版本信息以及指定库和头文件位置的编译器和链接器选项。例如，要获取访问库头文件所需的 C 预处理器标志，只需在`pkg-config`命令行中指定`--cflags`选项，适用于该包的编译器选项就会显示在`stdout`中。这个输出可以捕捉并根据需要附加到你的配置脚本和 makefile 中的编译器命令行。

也许你会想，既然 Autoconf 已经提供了`AC_CHECK_LIB`和`AC_SEARCH_LIBS`宏，为什么我们还需要像`pkg-config`这样的工具呢？首先，正如我之前提到的，Autoconf 的宏只会在“标准位置”查找库。你可以通过将搜索路径预加载到`CPPFLAGS`（使用`-I`选项）和`LDFLAGS`（使用`-L`选项）来欺骗这些宏，使其查找其他地方。然而，pkg-config 的设计目的是帮助你找到那些可能安装在只有用户的 pkg-config 安装才知道的位置的库；pkg-config 最棒的一点是，它知道如何在最终用户系统上找到那些用户自己都不知道的库和头文件。pkg-config 还可以告诉你的构建系统，在用户尝试静态链接到你的库时，需要哪些额外的依赖项。因此，它有效地将这些细节从用户那里隐藏开来，这就是我们所追求的用户体验。

然而，在使用 pkg-config 与 Autotools 时，有一些注意事项。在本书的第一版中，我曾建议使用与 pkg-config 一起发布的`PKG_CHECK_MODULES`附加 M4 宏来与 Autoconf 配合使用，这是一种很好的方法。但随着多年来的发现，我已修改了对此问题的看法，因为我发现，在一些相当常见的情况下，使用这个宏可能会带来比解决问题更多的问题。此外，`pkg-config`在 Shell 脚本中直接使用非常简单，因此用不透明的 M4 宏将其包装起来几乎没有意义。本章我们将更加详细地讨论这一主题，但我想先为接下来的示例展示一种使用模式。

### 深入探讨

让我们先来看一下`pkg-config`命令的`--help`选项的输出：

```
$ pkg-config --help
Usage:
  pkg-config [OPTION...]
--snip--
Application Options:
--snip--
  --modversion                               output version for package
--snip--
  --libs                                     output all linker flags
  --static                                   output linker flags for static linking
--snip--
  --libs-only-l                              output -l flags
  --libs-only-other                          output other libs (e.g. -pthread)
  --libs-only-L                              output -L flags
  --cflags                                   output all pre-processor and compiler flags
  --cflags-only-I                            output -I flags
  --cflags-only-other                        output cflags not covered by the cflags-only-I option
  --variable=NAME                            get the value of variable named NAME
  --define-variable=NAME=VALUE               set variable NAME to VALUE
  --exists                                   return 0 if the module(s) exist
  --print-variables                          output list of variables defined by the module
  --uninstalled                              return 0 if the uninstalled version of one or more
                                             module(s) or their dependencies will be used
  --atleast-version=VERSION                  return 0 if the module is at least version VERSION
  --exact-version=VERSION                    return 0 if the module is at exactly version VERSION
  --max-version=VERSION                      return 0 if the module is at no newer than version
                                             VERSION
  --list-all                                 list all known packages
  --debug                                    show verbose debug information
  --print-errors                             show verbose information about missing or conflicting
                                             packages (default unless --exists or
                                             --atleast/exact/max-version given on the command line)
--snip--
$
```

我这里只展示了我认为最有用的选项。还有十几个其他的选项，但这些是我们在 *configure.ac* 文件中会经常使用的选项。（我已经将较长的描述行进行了换行，因为 pkg-config 似乎认为每个人都有一个 300 列的显示器。）

让我们首先列出系统中 pkg-config 已知的所有模块。以下是我系统上的一些示例：

```
$ pkg-config --list-all
--snip--
systemd                        systemd - systemd System and Service Manager
fontutil                       FontUtil - Font utilities dirs
usbutils                       usbutils - USB device database
bash-completion                bash-completion - programmable completion for the bash shell
libcurl                        libcurl - Library to transfer files with ftp, http, etc.
--snip--
notify-python                  notify-python - Python bindings for libnotify
nemo-python                    Nemo-Python - Nemo-Python Components
libcrypto                      OpenSSL-libcrypto - OpenSSL cryptography library
libgdiplus                     libgdiplus - GDI+ implementation
shared-mime-info               shared-mime-info - Freedesktop common MIME database
libssl                         OpenSSL-libssl - Secure Sockets Layer and cryptography libraries
xbitmaps                       X bitmaps - Bitmaps that are shared between X applications
--snip--
xkbcomp                        xkbcomp - XKB keymap compiler
dbus-python                    dbus-python - Python bindings for D-Bus
$
```

pkg-config 通过让包的安装过程更新 pkg-config 数据库来“了解”一个包，这个数据库实际上只是一个知名目录，`pkg-config` 会检查该目录来解决查询。数据库条目只是以 *.pc* 扩展名结尾的纯文本文件。因此，让 pkg-config 在安装过程中了解你的库项目，其实并没有比生成并安装一个文本文件更难，Autoconf 可以帮助我们生成这个文件，稍后我们将看到这一点。

`pkg-config` 工具会在多个目录中查找这些文件。我们可以通过调用它并使用 `--debug` 选项，将输出（发送到 `stderr`）通过 `grep` 管道来查看它查找的目录和搜索顺序，方法如下：

```
$ pkg-config --debug |& grep directory
Cannot open directory #1 '/usr/local/lib/x86_64-linux-gnu/pkgconfig' in package search path: No
such file or directory
Cannot open directory #2 '/usr/local/lib/pkgconfig' in package search path: No such file or
directory
Cannot open directory #3 '/usr/local/share/pkgconfig' in package search path: No such file or
directory
Scanning directory #4 '/usr/lib/x86_64-linux-gnu/pkgconfig'
Scanning directory #5 '/usr/lib/pkgconfig'
Scanning directory #6 '/usr/share/pkgconfig'
$
```

`pkg-config` 在我的系统上尝试查找的前三个目录不存在。这些目录都位于 */usr/local* 目录树中。我在这个系统上并没有构建和安装很多包，因此没有将任何 *.pc* 文件安装到 */usr/local* 目录树中。

从输出中可以清楚地看到，*.pc* 文件必须安装到以下六个目录中的一个：*/usr(/local)/lib/x86_64-linux-gnu/pkgconfig*、*/usr(/local)/lib/pkgconfig* 或 */usr(/local)/share/pkgconfig*。当你仔细思考时，你会认识到这些路径本质上就是 pkg-config 的 `${libdir}`*/**pkgconfig* 和 `${datadir}`*/pkgconfig* 目录，如果说 pkg-config 在安装时不需要在 */usr* 和 */usr/local* 之间做选择的话。早期，这些确实就是 pkg-config 的库和数据安装路径，但不久之后，项目开发者就意识到 pkg-config 安装的位置与其应该在用户系统上搜索 *.pc* 文件的位置并无太大关系——这些文件可能分布在很多地方，取决于用户在系统上安装包的位置，而不是 pkg-config 安装的位置。

那么，如何处理安装到自定义位置的包或尚未安装的包呢？pkg-config 对这些情况也有解决方案。`PKG_CONFIG_PATH` 环境变量可以将用户指定的路径添加到 `pkg-config` 用来搜索数据文件的默认搜索路径之前。随着我们在 *configure.ac* 中介绍更多使用 `pkg-config` 命令的细节，我们将学习如何使用这一功能。

### 编写 pkg-config 元数据文件

如前所述，pkg-config 的*.pc*文件仅仅是简短的文本文件，它描述了构建和链接过程中的关键方面，供使用这些依赖包组件的消费端构建过程使用。

让我们看一下我系统上的一个示例*.pc*文件——它是*libssl*库的文件，属于 OpenSSL 包的一部分。首先，我们需要找到它：

```
$ pkg-config --variable pcfiledir libssl
/usr/lib/x86_64-linux-gnu/pkgconfig
$
```

`--variable`选项允许你查询变量的值，`pcfiledir`是 pkg-config 为每个*.pc*文件定义的一个变量。我将在本章后面介绍预定义变量的完整列表。`pcfiledir`变量显示了`pkg-config`发现的文件当前所在位置。这一变量的好处在于，它也可以在你的*.pc*文件中使用，提供一种类似重定位的机制。如果你的库和包含文件路径都相对于`${pcfiledir}`在*.pc*文件中定义，你可以随意移动文件（只要你将它定位的库和头文件也移动到相同的相对位置）。

我在清单 10-1 中提供了我的*libssl.pc*文件的完整内容。

```
➊ prefix=/usr
   exec_prefix=${prefix}
   libdir=${exec_prefix}/lib/x86_64-linux-gnu
   includedir=${prefix}/include

➋ Name: OpenSSL-libssl
   Description: Secure Sockets Layer and cryptography libraries
   Version: 1.0.2g
   Requires.private: libcrypto
   Libs: -L${libdir} -lssl
   Libs.private: -ldl
   Cflags: -I${includedir}
```

*清单 10-1:* libssl.pc：*一个示例* .pc *文件*

*.pc*文件包含两种类型的实体：变量定义（以➊开始），它们可以使用类似 Bourne shell 的语法引用其他变量；以及键值对标签（以➋开始），它们定义了`pkg-config`可以返回的关于已安装包的数据类型。这些文件可以包含 pkg-config 规范所需的任何内容，也可以只是包含最少的信息。除了这些实体，*.pc*文件还可以包含注释——任何以井号（`#`）标记开头的文本。虽然这些类型的实体可以混合出现，但通常的约定是将变量定义放在最上面，接着是键值对。

变量的外观和作用类似于 shell 变量；定义格式为变量名，后跟等号（`=`），再后跟值。即使值包含空格，你也无需对其进行引号处理。

Pkg-config 提供了一些预定义的变量，可以在*.pc*文件中使用，也可以通过命令行访问（正如我们之前所见）。表 10-1 显示了这些变量。

**表 10-1:** pkg-config 识别的预定义变量

| **变量** | **描述** |
| --- | --- |
| `pc_path` | `pkg-config`用来查找*.pc*文件的默认搜索路径 |
| `pcfiledir` | *.pc*文件的安装位置 |
| `pc_sysrootdir` | 用户设置的系统根目录，默认值为`/*` |
| `pc_top_builddir` | 执行`pkg-config`时用户的顶级构建目录的位置 |

在你查看足够多的*.pc*文件后，你可能会开始想，像`prefix`、`exec_prefix`、`includedir`、`libdir`和`datadir`这些变量是否对 pkg-config 有特殊的意义。它们没有；它们只是用来相对定义这些路径，以减少重复。

键值对格式为一个知名关键词，后跟冒号（`:`）字符，然后是构成值部分的文本。值可以引用变量；引用未定义的变量仅会展开为空。这些值中也不需要使用引号。

键值对的键是知名且有文档记录的，尽管在文件中放置未知键不会影响`pkg-config`使用文件中其余数据的能力。表 10-2 中显示的键是知名的：

**表 10-2：** pkg-config 识别的键值对中的知名键

| **键** | **描述** |
| --- | --- |
| `Name` | 库或包的可读名称。 |
| `Description` | 包的简短可读描述。 |
| `URL` | 与包关联的 URL——可能是包的下载站点。 |
| `Version` | 包的版本字符串。 |
| `Requires` | 本包所需的包列表；可以指定特定版本。 |
| `Requires.private` | 本包所需的私有包列表。 |
| `Conflicts` | 可选字段，描述本包与哪些包存在冲突。 |
| `Cflags` | 应该与此包一起使用的编译器标志。 |
| `Libs` | 应该与此包一起使用的链接器标志。 |
| `Libs.private` | 本包所需的私有库的链接器标志。 |

#### *信息字段*

为了让`pkg-config --exists`命令返回零到 shell，你至少需要指定`Name`、`Description`和`Version`。为了完整起见，考虑在项目有 URL 时也提供一个 URL。

如果你不确定为什么某个特定的`pkg-config`命令没有按预期工作，可以使用`--print-errors`选项。在`pkg-config`通常会默默返回一个 shell 代码的地方，`--print-errors`会显示非零 shell 代码的原因：

```
$ cat test.pc
prefix=/usr
libdir=${prefix}/lib dir

Name: test
#Description: a test pc file
Version: 1.0.0
$
$ pkg-config --exists --print-errors test.pc
Package 'test' has no Description: field
$ echo $?
1
$
```

**注意**

*`--validate`选项也会为已安装和未安装的*.pc*文件提供此信息。*

一个明显的疏漏是缺乏显示属于某个包的名称和描述信息的选项。描述信息在使用`--list-all`选项时会显示；然而，即使在该列表中显示的包名称实际上也是*.pc*文件的基础名称，而不是文件中`Name`字段的值。尽管如此，正如前面所提到的，这三个字段——`Name`、`Description`和`Version`——是必需的；否则，`pkg-config`认为该包不存在。

#### *功能字段*

`Version` 字段的类别跨越了信息性到功能性，因为有一些 `pkg-config` 命令行选项可以利用该字段的值，为配置脚本提供有关包的数据。`Requires`、`Requires.private`、`Cflags`、`Libs` 和 `Libs.private` 字段也为配置脚本和 makefile 提供机器可读的信息。`Cflags`、`Libs` 和 `Libs.private` 直接提供了 C 编译器和链接器的命令行选项。通过使用不同的 `pkg-config` 命令行选项，可以访问这些工具命令行中要添加的选项。

虽然 pkg-config 在概念上很简单，但一些细节如果你没有足够的实践，可能会有些难以捉摸。接下来我们将更详细地讲解这些字段。

信息性字段是为人类阅读而设计的。例如，可以使用 `--modversion` 选项显示包的版本：

```
$ pkg-config --modversion libssl
1.0.2g
$
```

**注意**

*不要将 `--version` 选项与 *`--modversion`* 选项混淆。如果你混淆了，它会默默返回 pkg-config 的版本，无论你在 *`--version`* 后指定什么模块。*

然而，`Version` 字段也可以用来向配置脚本指示一个包的版本是否满足要求：

```
$ pkg-config --atleast-version 1.0.2 libssl && echo "libssl is good enough"
libssl is good enough
$ pkg-config --exists "libssl >= 2.0" || echo "nope - too old :("
nope - too old :(
$
```

**注意**

*库版本检查与 Autoconf 的一般哲学相悖，后者检查的是所需功能而不是库的特定版本，因为某些发行版提供商会将功能回移植到旧版本的库，以便在不升级库的情况下在其发行版的目标版本上使用该功能（主要是为了方便，因为较新版本的库有时会带来新的依赖要求，这些要求可能会传播到多个级别）。这些示例仅用于向你展示 pkg-config 提供的功能的可能性。*

在功能字段中，有些比其他的更为直观。我们将逐一讲解每个字段，从较简单的开始。`Cflags` 字段可能是最简单的理解，它只是提供了包含路径的附加和其他选项给 C 预处理器和编译器。这两个工具的所有选项都汇聚在这一字段中，但 `pkg-config` 提供了命令行选项，用于返回字段值的部分内容：

```
$ pkg-config --cflags xorg-wacom
-I/usr/include/xorg
$ pkg-config --cflags-only-other xorg-wacom
$
```

**注意**

*这里需要注意的重要事项是，*`Cflags`* 字段包含的是编译器命令行选项，而不是编译器命令行选项的部分。例如，要为你的库定义包含路径，确保 *`Cflags`* 中包含 *`-I`* 标志和路径，就像在编译器命令行中那样。*

影响 `Cflags` 字段输出的其他选项包括 `--cflags-only-I` 和 `--cflags-only-other`。如你所见，`pkg-config` 区分了 `-I` 选项和其他选项；如果你指定了 `--cflags-only-I`，你只会看到 *.pc* 文件中的 `-I` 选项。

`Libs`字段提供了设置`-L`、`-l`和其他面向链接器的选项的位置。例如，如果你的包提供了*stpl*库，即*libstpl.so*，你需要在`Libs`字段中添加`-L`*`/installed/lib/path`*和`-lstpl`选项。Pkg-config 的`--libs`选项返回完整的值，并且与`Cflags`一样，有一些单独的选项（`--libs-only-l`，`--libs-only-L`，和`--libs-only-other`），它们将返回`Libs`选项的子集。

比较难理解的是`Libs.private`字段的使用。该字段的文档说明它是“该包所需但不暴露给应用程序的库”。然而，实际上，虽然这些是构建包发布的库所需的库，它们也是包的消费者在静态链接到该包的库时所需的库。^(1)事实上，使用`pkg-config`的`--static`命令行选项，配合`--libs`（或类似选项）选项，将会显示`Libs`和`Libs.private`字段选项的组合。这是因为，静态链接库时，在链接阶段，所有从静态库中直接链接的代码所需的符号都必须被链接。

这是一个重要的概念，理解它的工作原理是正确编写项目的*.pc*文件的关键。从终端用户的角度考虑：他们想要编译某个项目，并且希望将其与你的库静态链接（当然，我们还必须假设你的项目会构建并安装一个静态版本的库）。为了做到这一点，编译器和链接器命令行上需要哪些选项和库，*除了那些在链接到动态库时已经要求的选项*，才能成功完成这个任务？这个问题的答案将告诉你在你项目的*.pc*文件中`Libs.private`字段应填入什么内容。

既然这些话题已经讲完，我们可以正式讨论`Requires`和`Requires.private`字段。这些字段中的值是其他 pkg-config 包的名称，并可以选择性地指定版本。如果你的包依赖于另一个由 pkg-config 管理的特定版本的包，只需要在`Requires`字段中指定该包，前提是它的`Cflags`和`Libs`字段的值是用户的构建过程需要的，以便使用你包的共享库；如果它的`Cflags`和`Libs.private`字段的值是用户构建静态库时需要的，则应将该包指定在`Requires.private`字段中。

通过理解`Requires`和`Requires.private`字段，我们现在可以看到，pkg-config 包所需的附加选项，通常会放在`Cflags`和`Libs`或`Libs.private`字段中，其实不需要放在这些字段中，因为你可以简单地通过名称（以及版本或版本范围）在`Requires`或`Requires.private`中引用该包。Pkg-config 会递归地查找并根据需要合并所有包字段中的选项。

如果你包所需的包不是由 pkg-config 管理的，你必须将通常会出现在*.pc*文件中的选项添加到你自己的`Cflags`、`Libs`和`Libs.private`字段中。

在`Requires`和`Requires.private`字段中使用的版本规范与 RPM 版本规范相同。你可以使用`>`、`>=`、`=`、`<=`或`<`。遗憾的是，这些字段只允许一个给定库的实例，这意味着你无法对所需包的版本同时应用上下界。清单 10-2 提供了一个使用版本范围的做法示例。

```
--snip--
Name: music
Description: A library for managing music files
Version: 1.0.0
Requires: chooser >= 1.0.1, player < 3.0
--snip--
```

*清单 10-2：在`Requires`中指定版本和版本范围*

`Requires`字段表示这里需要两个库：*chooser*和*player*。*chooser*的版本必须是 1.0.1 或更高，而*player*的版本必须低于 3.0。^(2)

最后，`Conflicts`字段只是让你作为包的作者定义与包冲突的其他包，字段的格式与`Requires`和`Requires.private`相同。对于这个字段，你可以多次提供相同的包，以便定义与包冲突的特定版本范围。

当你完成*.pc*文件的编写后，可以使用`--validate`选项来验证它：

```
$ pkg-config --validate test.pc
$
```

**注意**

*你可以使用任何提供字段信息的*`pkg-config`*选项，这些信息可以来源于已安装的*.pc*文件（只需使用文件的基本名称），也可以来源于未安装的*.pc*文件（通过指定文件的完整名称），正如这个示例所示。*

如果`pkg-config`能够检测到任何错误，它们会被显示出来。如果什么都没有显示，那就说明`pkg-config`至少能够正确解析你的文件，并且一些基本检查通过了。

### 使用 Autoconf 生成*.pc*文件

现在你已经理解了*.pc*文件的基本结构，我们来考虑一下如何利用配置脚本生成的配置数据来生成*.pc*文件。考虑 pkg-config 提供的信息类型，其中大部分是路径信息，而配置脚本的目的是管理所有这些路径，包括构建产品的安装位置。

例如，用户可能会在`configure`命令行中指定安装前缀。该前缀决定了包的包含文件和库在用户安装该包时将被放置在系统的哪个位置。*.pc*文件最好能够知道这些位置，而且，如果我们能提供一个自动更新该文件的构建系统，使其反映用户在`configure`命令行中指定的前缀路径，那将是非常方便的。

#### *从 pc.in 模板生成 pc 文件*

为了实现这一点，我们不会直接编写*.pc*文件。相反，我们将为 Autoconf 编写*.pc.in*模板文件，并将`prefix`变量的值设置为`@prefix@`，以便在`configure`将*.pc.in*模板转换为可安装的*.pc*文件时，替换该引用为实际的配置前缀。

我们还可以将`Version`字段的值设置为`@PACKAGE_VERSION@`，该值由你传递给 Autoconf `AC_INIT`宏的值定义，位于*configure.ac*中。为了方便实验，在一个空目录中创建一个*configure.ac*文件，如示例 10-3 所示。

```
AC_INIT([test],[3.1])
AC_OUTPUT([test.pc])
```

*示例 10-3:* configure.ac: *生成* test.pc *文件，源自* test.pc.in

现在，在同一目录下创建一个*test.pc.in*文件，如示例 10-4 所示。

```
➊ prefix=@prefix@
   libdir=${prefix}/lib/test
   includedir=${prefix}/include/test

   Name: test
   Description: A test .pc file
➋ Version: @PACKAGE_VERSION@

   CFlags: -I${includedir} -std=c11
   Libs: -L${libdir} -ltest
```

*示例 10-4:* test.pc.in: *一个* .pc *模板文件*

在这里，我们在➊和➋处指定了`prefix`和`Version`字段的值，作为 Autoconf 替换变量引用。

生成文件并检查结果：

```
   $ autoreconf -i
   $ ./configure --prefix=$HOME/test
   configure: creating ./config.status
   config.status: creating test.pc
   $
   $ cat test.pc
➊ prefix=/home/jcalcote/test
   libdir=${prefix}/lib/test
   includedir=${prefix}/include/test

   Name: test
   Description: A test .pc file
➋ Version: 3.1

   CFlags: -I${includedir} -std=c11
   Libs: -L${libdir} -ltest
   $
   $ pkg-config --cflags test.pc
   -std=c11 -I/home/jcalcote/test/include/test
   $
```

正如你在控制台输出的➊和➋处看到的，生成的*test.pc*文件中的 Autoconf 变量引用被替换为这些 Autoconf 变量的值。

#### *使用 make 生成.pc 文件*

使用 Autoconf 从模板生成*.pc*文件的缺点是，它限制了用户在运行`make`时更改前缀选择的能力。这个小问题可以通过编写*Makefile.am*规则来生成*.pc*文件来解决。请按照示例 10-5 中所示，修改之前实验中的*configure.ac*文件。

```
AC_INIT([test],[3.1])
AM_INIT_AUTOMAKE([foreign])
AC_OUTPUT([Makefile])
```

*示例 10-5:* configure.ac: *修改示例 10-3 以使用* `make` *生成* test.pc

接下来，添加一个*Makefile.am*文件，如示例 10-6 所示。

```
EXTRA_DIST = test.pc
%.pc : %.pc.in
        sed -e 's|[@]prefix@|$(prefix)|g'\
          -e 's|[@]PACKAGE_VERSION@|$(PACKAGE_VERSION)|' $< >$@
```

*示例 10-6:* Makefile.am: *添加* `make` *规则来生成* test.pc

清单 10-6 中的关键功能是封装在模式规则中，使用一个简单的 `sed` 命令将 *.pc.in* 文件转换为 *.pc* 文件。这个 `sed` 命令中唯一不同的部分是，在要替换的变量的前导 @ 符号周围使用了方括号。`sed` 将这些方括号视为多余的正则表达式语法，但它们对 Autoconf 的作用是阻止它将该序列解释为替换变量的开头字符。我们不希望 Autoconf 替换这个变量，而是希望 `sed` 在 *test.pc.in* 文件中查找这个序列。另一种解决方法是自己制定变量替换格式，但需要注意，这种语法在 Autotools 社区中用于此目的是相当常见的。

**注意**

*模式规则是特定于 GNU *`make`* 的，因此不可移植。最近，Automake 邮件列表中有一些讨论，提到是否放宽要求生成可移植 *`make`* 语法的限制，而仅要求使用 GNU *`make`*，因为 GNU *`make`* 目前已经广泛移植。*

在这个示例中，我已将 *test.pc* 添加到 Automake 的 `EXTRA_DIST` 变量中，以便在执行 `make dist` 或 `distclean` 时构建它，但你也可以将 *test.pc* 作为任何目标的前提条件添加到你的 *Makefile.am* 文件中，以便在构建的该阶段使其可用（如果需要的话）。我们来试试：

```
$ autoreconf -i
configure.ac:2: installing './install-sh'
configure.ac:2: installing './missing'
$
$ ./configure
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /bin/mkdir -p
checking for gawk... gawk
checking whether make sets $(MAKE)... yes
checking whether make supports nested variables... yes
checking that generated files are newer than configure... done
configure: creating ./config.status
config.status: creating Makefile
$
$ make prefix=/usr dist
make    dist-gzip am__post_remove_distdir='@:'
make[1]: Entering directory '/home/jcalcote/dev/book/autotools2e/book/test'
sed -e 's|[@]prefix@|/usr|g'\
            -e 's|[@]PACKAGE_VERSION@|3.1|' test.pc.in >test.pc
--snip--
$
$ cat test.pc
prefix=/usr
--snip--
```

请注意，我在 `make` 命令行中添加了 `prefix=/usr`；因此，*test.pc* 是使用该值在 `prefix` 变量中生成的。

### 卸载的 .pc 文件

我在本章开头提到过，pkg-config 也能够处理解析未安装的库和头文件的引用。这里所说的 *未安装*，指的是已经构建但未安装的产品；它们仍然保留在另一个项目的构建输出目录中。现在我们来看看如何实现这一点。

要使用它，用户需要将 `PKG_CONFIG_PATH` 设置为指向包含所需软件包的 *-uninstalled* 变体 *.pc* 文件的目录。这里所说的“*-uninstalled* 变体”是指一个名为 *test.pc* 的 *.pc* 文件，会有一个名为 *test-uninstalled.pc* 的 *-uninstalled* 变体。该 *-uninstalled* 变体并未安装在 pkg-config 数据库目录中，而是仍然保留在用户已构建的第三方依赖项的项目源目录中。以下是一个示例：

```
$ ./configure PKG_CONFIG_PATH=$HOME/required/pkg
--snip--
$
```

**注意**

*我在这里遵循的是 Autoconf 推荐的做法，即将环境变量作为参数传递给 *`configure`*。在环境中设置变量或在 *`configure`* 前的同一命令行中设置它也可以，但不推荐这样做，因为 *`configure`* 对这些通过其他方式设置的变量了解较少。*

假设*`$HOME`**/required/pkg*是所需包解压和构建的目录，并且假设该目录中包含该包的（可能生成的）*.pc*文件，并且该目录中有*-uninstalled*版本，则该文件将通过执行`pkg-config`工具时引用所需包名称的方式进行访问，这些执行来自我们`configure`脚本中的引用。

显然，你不希望安装任何带有*-uninstalled*后缀的*.pc*文件变体——它们仅设计用于在构建目录中以这种方式使用。或许不那么明显的是，带有*-uninstalled*后缀的*.pc*文件并不包含与已安装版本相同的所有选项。简而言之，它们的区别在于路径选项。*-uninstalled*版本应包含相对于头文件源位置和库构建位置的绝对路径，以便当选项传递给消费者工具时，它们能够在这些路径中找到产品（头文件和库）。

让我们试试。编辑你在示例 10-3 中创建的*configure.ac*文件，使其与示例 10-7 中显示的文件相同。

```
AC_INIT([test],[3.1])
AC_OUTPUT([test.pc test-uninstalled.pc])
```

*示例 10-7:* configure.ac: *生成* -uninstalled *版本的* test.pc

绝对路径可以通过使用适当的 Autoconf 替换变量来推导，例如在示例 10-8 中所示的`@abs_top_srcdir@`和`@abs_top_builddir@`。

```
➊ libdir=@abs_top_builddir@/lib/test
➋ includedir=@abs_top_srcdir@/include/test

   Name: test
   Description: A test .pc file
➌ Version: @PACKAGE_VERSION@

   CFlags: -I${includedir} -std=c11
   Libs: -L${libdir} -ltest
```

*示例 10-8:* test-uninstalled.pc: *test.pc.in 的* -uninstalled *版本*

这是来自示例 10-4 的*.pc*文件的*-uninstalled*版本。我已经删除了`prefix`变量，因为在这种情况下它已经没有意义。我已将`${prefix}`引用替换为在`libdir` pkg-config 变量中使用`@abs_top_builddir@`，在`includedir` pkg-config 变量中使用`@abs_top_srcdir@`，如图➊和➋所示。让我们试试：

```
$ autoreconf
$ ./configure
configure: creating ./config.status
config.status: creating test.pc
config.status: creating test-uninstalled.pc
$ pkg-config --cflags test.pc
-std=c11 -I/home/jcalcote/dev/book/autotools2e/book/temp/include/test
$
```

你可能会问，为什么这比在`configure`命令行中直接设置`CFLAGS`（或`CPPFLAGS`）和`LDFLAGS`要容易得多。嗯，一方面，记住`PKG_CONFIG_PATH`比记住所有可能需要的单个工具变量更容易。另一个原因是，这些选项被封装在最能理解它们的地方——即由所需软件包的作者编写的*.pc*文件中。最后，如果这些选项发生变化，你必须相应地改变你使用的单个变量，但`PKG_CONFIG_PATH`将保持不变。pkg-config 提供的额外间接层次将所有细节隐藏在你和你的高级用户以及贡献者之外。

### 在 configure.ac 中使用 pkg-config

我们已经看到 *.pc* 文件是如何构建的。现在，让我们来看看如何在 *configure.ac* 中使用这个功能。如前一节所述，`--cflags` 选项提供了编译器所需的 `Cflags` 字段，以便编译此包。让我们用之前看到的 *libssl.pc* 文件来试一试。我在 清单 10-1 中的相关部分已在 清单 10-9 中重现。

```
prefix=/usr
--snip--
includedir=${prefix}/include
--snip--
Cflags: -I${includedir}
```

*清单 10-9:* libssl.pc: *此* .pc *文件的相关部分*

当我们对这个 *.pc* 文件使用 `--cflags` 选项时，我们现在明白应该看到一个 `-I` 编译器命令行选项。

```
$ pkg-config --cflags libssl

$
```

然而，什么也没有打印出来。嗯，我们做错什么了吗？*libssl.pc* 文件告诉我们，如果我们将变量展开，我们应该看到类似于 `-I/usr/include` 的内容，对吧？实际上，`pkg-config` 正在做它应该做的事情——它正在打印出找到 *libssl* 头文件所需的*附加命令行选项*。我们不需要告诉编译器关于 */usr**/include* 目录的事情，因为这是一个标准位置，`pkg-config` 知道这一点，并会自动省略这类选项。^(3)

让我们试试一个 `Cflags` 值包含非标准包含位置的 *.pc* 文件。请注意，我在这里使用 `pkg-config` 本身来查找其数据库目录的路径，因为在不同的 Linux 发行版上，这个路径是不同的，用来查找 *xorg-wacom.pc* 文件：

```
$ cat $(pkg-config --variable pcfiledir xorg-wacom)/xorg-wacom.pc
sdkdir=/usr/include/xorg

Name: xorg-wacom
Description: X.Org Wacom Tablet driver.
Version: 0.32.0
Cflags: -I${sdkdir}
$
$ pkg-config --cflags xorg-wacom
-I/usr/include/xorg
$
```

由于 */usr/include/xorg* 不是一个标准的包含路径，因此会显示该路径的 `-I` 选项。^(4) 这意味着你可以在你的 *.pc* 文件中完整地记录包的需求，而不必担心在消费者的编译器和链接器命令行中添加冗余的无用定义。

那么，我们如何使用这个输出呢？其实没有什么比一个小的 shell 脚本更难的了，正如在 清单 10-10 中所示。

```
--snip--
LIBSSL_CFLAGS=$(pkg-config --cflags libssl)
--snip--
```

*清单 10-10：使用 `pkg-config` 填充 `CFLAGS` 在* configure.ac

使用美元括号符号（dollar-parens notation）可以将此 `pkg-config` 命令的输出捕获到 `LIBSSL_CFLAGS` 环境变量中。

**注意**

*当然，你可以使用反引号（backticks）来代替我在 清单 10-10 中使用的美元括号符号来实现相同的目标。反引号格式较旧，并且略微具有更好的可移植性，但它的缺点是无法轻松嵌套。例如，你不能像 *`$(pkg-config --cflags $(cat libssl-pc-file.txt))`* 这样使用反引号，而不进行大量的转义魔法。*

链接器选项的访问方式类似：

```
$ pkg-config --libs libssl
-lssl
$
```

回到 列表 10-1 中提到的 *libssl.pc* 文件，我们确实可以看到 `Libs` 行包含了 `-lssl`。同时，正如我们刚刚发现的，`-L` 选项，指向一个标准的链接器位置，*/usr/lib/x86_64-linux-gnu*，被自动省略。我们可以按照 列表 10-11 中展示的方式，将其添加到我们的 *configure.ac* 文件中。

```
--snip--
LIBSSL_LIBS=$(pkg-config --libs libssl)
--snip--
```

*列表 10-11：在 `pkg-config` 中填充 configure.ac 中的 `LIBS`*

让我们将所有内容结合起来，填充编译 *libssl* 头文件并与 *libssl* 链接所需的所有变量。列表 10-12 展示了这可能是如何实现的。

```
--snip--
if pkg-config --atleast-version=1.0.2 libssl; then
  LIBSSL_CFLAGS=$(pkg-config --cflags libssl)
  LIBSSL_LIBS=$(pkg-config --libs libssl)
else
  m4_fatal([Requires libssl v1.0.2 or higher])
fi
--snip--
CFLAGS="${CFLAGS} ${LIBSSL_CFLAGS}"
LIBS="${LIBS} ${LIBSSL_LIBS}"
--snip--
```

*列表 10-12：在 `pkg-config` 中使用 libssl 访问 configure.ac*

难道还有比这更简单或更易读的方法吗？我怀疑。让我们看一个更多的例子——即静态链接到 *libssl*，这也要求（私下）链接 *libcrypto*：

```
   $ cat $(pkg-config --variable pcfiledir libssl)/libssl.pc
   prefix=/usr
   exec_prefix=${prefix}
   libdir=${exec_prefix}/lib/x86_64-linux-gnu
   includedir=${prefix}/include

   Name: OpenSSL-libssl
   Description: Secure Sockets Layer and cryptography libraries
   Version: 1.0.2g
➊ Requires.private: libcrypto
   Libs: -L${libdir} -lssl
➋ Libs.private: -ldl
   Cflags: -I${includedir}
   $
   $ cat $(pkg-config --variable pcfiledir libcrypto)/libcrypto.pc
   prefix=/usr
   exec_prefix=${prefix}
   libdir=${exec_prefix}/lib/x86_64-linux-gnu
   includedir=${prefix}/include

   Name: OpenSSL-libcrypto
   Description: OpenSSL cryptography library
   Version: 1.0.2g
   Requires:
   Libs: -L${libdir} -lcrypto
➌ Libs.private: -ldl
   Cflags: -I${includedir}
   $
   $ pkg-config --static --libs libssl
➍ -lssl -ldl -lcrypto -ldl
   $
```

正如你在此控制台示例中看到的 ➊，*libssl* 私下要求由 pkg-config 管理的 *libcrypto* 包，这意味着链接到 *libssl* 共享库时不需要在链接命令行中添加 `-lcrypto`，但静态链接时确实需要这个额外的库选项。我们还可以在 ➋ 看到，*libssl* 私下要求一个非 pkg-config 管理的库 *libdl.so*。

**注意**

*你可能会发现你的* libssl.pc *和* libcrypto.pc *文件的内容与我的有所不同，这取决于你所使用的 Linux 发行版和你安装的 openssl 版本。别担心这些差异——你的系统和你的 * .pc * 文件上的一切都会正常工作。这个例子中最重要的是理解我所解释的概念。*

进入 *libcrypto.pc* 文件，我们在 ➌ 看到 *libcrypto* 还私下要求 *libdl.so*。

在 ➍ 处，值得注意的是，pkg-config “足够智能” 能理解链接器的库排序要求，并将 `-ldl` 设置在输出行中，排在 `-lssl` 和 `-lcrypto` 之后。^(5) 我们人类有时很难手动完成这些事情。幸运的是，当一个工具能够处理一切，且不需要我们担心它是如何完成的时，真是太好了。最终，我想强调的是，pkg-config 将选项的控制权牢牢掌握在最有可能理解这些选项如何指定和排序的人手中——我们依赖项的维护者。

### pkg-config Autoconf 宏

正如我在本章开始时提到的，pkg-config 还提供了一组 Autoconf 扩展宏，存放在一个名为 *pkg.m4* 的文件中，并安装在 */usr**(/local)/share/aclocal* 目录下，这也是 `autoconf` 查找 *.m4* 文件的位置，包含了你可以在 *configure.ac* 文件中使用的 Autoconf 标准宏。

为什么我在例子中没有使用这些宏？嗯，避免使用这些宏有几个原因，一个显而易见，另一个则更为微妙——甚至有些狡猾。显而易见的原因是，直接在 *configure.ac* 中使用 `pkg-config` 工具在 shell 脚本中是多么容易。为什么要试图将它封装在 M4 宏中呢？

至于第二个原因，回顾之前的讨论，`autoconf` 的输入是包含 *configure.ac* 文件内容的一个数据流，以及所有宏定义，这些宏定义允许 M4 将所有宏调用展开为 shell 脚本。这些宏定义成为输入流的一部分，因为 `autoconf` 在读取你的 *configure.ac* 文件之前，首先会读取 */usr**(/local)/share/aclocal* 目录中的所有 *.m4* 文件。换句话说，`autoconf` 并不知道一个必需的 *.m4* 文件缺失。它仅仅期望在安装路径的 *aclocal* 目录中的 *.m4* 文件中找到 *configure.ac* 所需要的所有宏定义。因此，`autoconf` 无法告诉你输入流中是否缺少某个宏定义。它只是没有意识到 `PKG_CHECK_MODULES` 是一个宏，因此没有将它展开成有效的 shell 脚本。所有这些都发生在你运行 `autoconf`（或 `autoreconf`）时。当你接着尝试运行 `configure` 时，它会失败，并显示与实际问题相差甚远的错误信息，你仅凭这些信息无法知道它们的含义。

一幅图胜过千言万语，所以让我们尝试一个快速实验。在一个空目录中创建一个 *configure.ac* 文件，如 列表 10-13 所示。

```
AC_INIT([test],[1.0])
AN_UNDEFINED_MACRO()
```

*列表 10-13：一个* configure.ac *文件，其中有一个未知的* 宏 *扩展*

现在执行 `autoconf`，然后是 `./configure`：

```
$ autoconf
$ ./configure
./configure: line 1675: syntax error: unexpected end of file
$
```

注意，当 `autoconf` 将 *configure.ac* 转换为 *configure* 时，你不会收到任何错误。这完全是合理的，因为 `m4` 作为一个基于文本的宏处理器，只会尝试解释数据流中已知的宏。其他所有内容都会直接传递到输出流，就好像它是实际的 shell 脚本一样。

当我们运行 `configure` 时，得到了一个关于意外文件结束的加密错误（在第 1675 行……来自一个只有两行的 *configure.ac* 文件）。实际上发生的情况是，你不小心开始定义了一个名为 `AN_UNDEFINED_MACRO` 的 shell 函数，但没有在大括号中提供该函数的主体。shell 认为这不对劲，并用它一贯简洁的方式告诉了你。

如果我们省略了 `AN_UNDEFINED_MACRO` 后面的圆括号，shell 会给出更具信息性的错误：

```
$ cat configure.ac
AC_INIT([test],[1.0])
AN_UNDEFINED_MACRO
$ ./configure
./configure: line 1674: AN_UNDEFINED_MACRO: command not found
$
```

至少这次，shell 告诉了我们问题项的名称，给了我们机会去 *configure.ac* 中查找它，并可能弄清楚出了什么问题。^(6)

关键是，当你*认为*自己在使用 pkg-config 宏，但 `autoconf` 在查找常规宏目录时没有找到 *pkg.m4* 时，就会发生这种情况。其实并没有太多启发。在我个人的谦虚意见中，你最好跳过那几百行不透明的宏代码，直接在 *configure.ac* 文件中使用 `pkg-config`。

然而，`autoconf` 无法找到已安装的 *pkg.m4* 文件的原因却颇具启发性。有一个常见的原因是，你从你的发行版的包仓库中安装了 *pkg-config* 包（或者在操作系统安装时它被自动安装了），使用了 `yum` 或 `apt`。但你从 GNU 网站下载、构建并安装了 Autoconf，因为你的发行版版本的 Autoconf 版本落后了四个版本，而你需要最新的版本。那么，pkg-config 的安装过程将 *pkg.m4* 安装到了哪里？（提示：*/usr/share/aclocal*。）`autoconf` 从哪里获取宏文件？（提示：*/usr/*local*/share/aclocal*。）当然，你可以通过将 */usr/share/aclocal/pkg.m4* 复制到 */usr/local/share/aclocal* 来轻松解决这个问题，一旦你遇到这个问题一两次，你就再也不会被它困住了。但是你的高级用户和贡献者将不得不经历同样的过程——或者你也可以直接告诉他们都买本书，去读 第十章。

### 总结

在本章中，我们讨论了将 Autoconf 与 pkg-config 一起使用的好处，如何从 Autoconf 模板生成 *.pc* 文件，如何从 *configure.ac* 文件中使用 `pkg-config`，以及 pkg-config 特性的一些细微差别。

你可以在官方的 pkg-config 网站上阅读一些关于正确使用 *pkg-config* 包的更多内容，网址是 *[`www.freedesktop.org/wiki/Software/pkg-config`](https://www.freedesktop.org/wiki/Software/pkg-config)*。Dan Nicholson 在他个人页面上写了一篇简明易懂的教程，介绍如何使用 pkg-config，网址是 [freedesktop.org](http://freedesktop.org) (*[`people.freedesktop.org/~dbn/pkg-config-guide.html`](http://people.freedesktop.org/~dbn/pkg-config-guide.html)*)。这个页面也可以通过 pkg-config 网站上的链接访问。

*pkg-config* 手册页提供了有关如何正确使用 `pkg-config` 的更多信息，但老实说，除了由一些有胆识的个人撰写的博客文章外，几乎没有其他更多的资料。幸运的是，关于 pkg-config 其实没有太多需要搞明白的东西。它写得很好，文档也很完善（就软件而言），只有少数几个小问题，我已经在这里尝试覆盖了这些问题。
