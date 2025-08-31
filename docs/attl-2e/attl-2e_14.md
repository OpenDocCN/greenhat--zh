## 第十四章：FLAIM：一个 AUTOTOOLS 示例

*阿布内尔叔叔说……一个人开始提着猫的尾巴回家，是在获得一种永远有用的知识。*

—马克·吐温，《汤姆·索亚在海外》

![Image](img/common.jpg)

到目前为止，在本书中，我已经带你快速浏览了 Autoconf、Automake 和 Libtool 的主要特性，以及与 Autotools 配合得好的其他工具。我尽力以一种既简洁易懂又便于记忆的方式进行解释——特别是如果你有时间和兴趣跟随我提供的示例进行实践。我始终认为，没有什么学习方式能比在实践中获得的学习更有效。

在本章及下一章中，我们将继续通过研究我用来将一个现有的、真实的开源项目从一个复杂的手写 makefile 转换为一个完整的 GNU Autotools 构建系统的过程，来学习更多关于 Autotools 的内容。我在这些章节中提供的示例说明了在转换过程中我做出的决策以及一些 Autotools 特性的具体应用，包括一些我在之前的章节中尚未展示的特性。这两章将通过展示解决实际问题的真实方案，完成我们对 Autotools 的学习。

我选择转换的项目叫做*FLAIM*，它代表着*灵活适应性信息管理*。

### 什么是 FLAIM？

FLAIM 是一个高可扩展的数据库管理库，使用 C++编写，并建立在名为 FLAIM 工具包的自身轻量级可移植性层之上。有些读者可能会认识到 FLAIM 是 Novell^(1) eDirectory 和 Novell GroupWise 服务器使用的数据库。FLAIM 起源于 1980 年代末的 WordPerfect，并在 1994 年 Novell 和 WordPerfect 合并时成为 Novell 软件组合的一部分。Novell eDirectory 使用 FLAIM 的一个衍生版本来管理包含超过十亿个对象的目录信息库，GroupWise 则使用一个更早的衍生版本来管理各种服务器端数据库。

Novell 在 2006 年将 FLAIM 源代码作为开源项目发布，并采用 GNU 较宽松公共许可证（LGPL）版本 2^(2)。FLAIM 项目目前托管在[SourceForge.net](http://SourceForge.net)上，经过了 25 年的开发和在各种 WordPerfect 和 Novell 产品及项目中的巩固与完善。^(3)

### 为什么选择 FLAIM？

虽然 FLAIM 远不是一个主流的开源软件项目，但它具有若干特质，使其成为展示如何将项目转换为使用 Autotools 的完美示例。首先，FLAIM 当前是使用一个手写的 GNU Makefile 构建的，该 Makefile 包含超过 2000 行复杂的 make 脚本。FLAIM 的 Makefile 包含许多 GNU Make 特有的构造，因此只能使用 GNU Make 来处理这个 Makefile。多个（但几乎相同的）Makefile 被用来构建 *flaim*、*xflaim* 和 *flaimsql* 数据库库，以及 FLAIM 工具包（*ftk*），并在 Linux、各种 Unix 版本、Windows 和 NetWare 上构建几个实用程序和示例程序。

现有的 FLAIM 构建系统支持多种不同版本的 Unix 操作系统，包括 AIX、Solaris 和 HP-UX，以及 Apple 的 macOS。它还支持这些系统上的多种编译器。这些特性使得 FLAIM 成为该示例转换项目的理想选择，因为我可以向你展示如何在新的 *configure.ac* 文件中处理操作系统和工具集的差异。

现有的构建系统还包含许多标准 Automake 目标的规则，例如分发 tar 包。此外，它还提供了构建二进制安装包的规则，以及为能够构建和安装 RPM 包的系统提供的 RPM 规则。它甚至提供了构建 Doxygen^(4) 描述文件的目标，之后用于生成源代码文档。我将用几段话向你展示如何将这些类型的目标添加到 Automake 提供的基础设施中。

FLAIM 工具包是一个可移植性库，第三方项目可以独立地将其集成并使用。我们可以使用该工具包展示 Autoconf 如何将独立的子项目作为可选子目录管理在一个项目中。如果用户的构建机器上已经安装了 FLAIM 工具包，他们可以使用已安装的版本，或者选择覆盖为本地副本。另一方面，如果未安装工具包，则默认使用本地子目录版的工具包。

FLAIM 项目还提供了用于构建 Java 和 C# 语言绑定的代码，因此我将稍微探讨这些晦涩的领域。我不会深入讲解如何构建 Java 或 C# 应用程序，但我会介绍如何编写生成 Java 和 C# 程序及语言绑定库的 *Makefile.am* 文件。

FLAIM 项目很好地利用了单元测试。这些测试作为独立程序构建，可以在没有命令行选项的情况下运行，因此我可以轻松向你展示如何使用 Automake 的简单测试框架将实际的单元测试添加到新的 FLAIM Autotools 构建系统中。

FLAIM 项目及其原始构建系统采用了相当模块化的目录布局，使其转换为 Autotools 模块化构建系统变得相当简单。对目录树进行一次简单的 `diff` 工具比较就足够了。

### 后勤

当本书的第一版在 2010 年发布时，FLAIM 刚刚作为一个开源项目在[SourceForge.net](http://SourceForge.net)上发布，并使用 Subversion 管理其源代码库。从那时起，FLAIM 项目基本上变得不活跃。我所知道的没有人正在积极使用该代码库。由于我是唯一剩下的源代码维护者，我为 FLAIM 创建了一个 GitHub 仓库，专门用于本书第二版的第十四章和第十五章。您可以在 GitHub 的 FLAIM 项目下的 NSP-Autotools 区域找到这个仓库。^(5) 我已经更新了本章中的信息，以便与 FLAIM 在 git 仓库中的存储方式相关。

本章的源代码库与前几章的源代码库风格略有不同。我对 FLAIM [SourceForge.net](http://SourceForge.net) 项目所做的原始 Autotools 构建系统更改被埋藏在几十个无关的更改之下，并与之交织在一起。与其花费数小时将这些更改分离开来，以便为您提供 FLAIM 代码库的前后快照，不如直接选择将最终的 FLAIM 代码和其 Autotools 构建系统提交到 GitHub 项目中。^(6)

不要对 FLAIM 当前的活动状态感到灰心——它仍然为我们提供了在现实世界项目中学习 Autotools 构建系统技术的各种机会。

### 初步观察

首先，我要说明将 FLAIM 从 GNU makefile 转换为 Autotools 构建系统并不是一个简单的项目。这花费了我几周的时间，其中大部分时间都用来确定具体需要构建什么以及如何构建——换句话说，就是分析遗留的构建系统。我花费的另一大部分时间则是在转换那些位于 Autotools 功能边缘的方面。例如，我花了*更多*时间来转换用于构建 C#语言绑定的构建系统规则，而不是转换用于构建核心 C++库的规则。

这个转换项目的第一步是分析 FLAIM 现有的目录结构和构建系统。哪些组件实际上被构建，哪些组件依赖于其他组件？是否可以单独构建、分发和消费各个组件？这些组件级别的关系非常重要，因为它们通常决定了你如何布局项目的目录结构。

FLAIM 项目实际上是在其代码库中由多个小项目组成的一个大项目。这里有三个独立且不同的数据库产品：*flaim*、*xflaim* 和 *flaimsql*。flaim 子项目是最初的 FLAIM 数据库库，用于 eDirectory 和 GroupWise。xflaim 项目是为 Novell 内部项目开发的分层 XML 数据库；它针对基于路径的节点访问进行了优化。flaimsql 项目是 FLAIM 数据库之上的 SQL 层。它被写作一个独立的库，目的是优化 FLAIM 的底层 API 以支持 SQL 访问。这个项目是一个实验，坦白说，它还没有完全完成（但它可以编译）。

关键在于，这三种数据库库彼此独立且无关，没有相互依赖。由于它们可以彼此独立使用，因此实际上可以作为独立的发行版进行发布。你可以将它们看作是各自独立的开源项目。那么，这将成为我的主要目标之一：允许 FLAIM 开源项目轻松拆分为多个可以独立管理的开源项目。

FLAIM 工具包也是一个独立的项目。尽管它专门为 FLAIM 数据库库量身定制，仅提供数据库管理系统所需的系统服务抽象，但它完全依赖于自身，因此可以轻松地作为其他项目中可移植性的基础，而不带任何不必要的数据库负担。^(7)

原始 FLAIM 项目在其代码库中的布局如下：

```
$ tree -d --charset=ascii FLAIM
FLAIM
|-- flaim
|   |-- debian
|   |-- docs
|   |-- sample
|   |-- src
|   `-- util
|-- ftk
|   |-- debian
|   |-- src
|   `-- util
|-- sql
|   `-- src
--snip--
`-- xflaim
    |-- csharp
    --snip--
    |-- java
    --snip--
    |-- sample
    --snip--
    |-- src
    `-- util
--snip--
```

整个目录结构相当广泛，并且在某些地方稍显深入，包括由传统构建系统构建的显著实用程序、测试以及其他此类二进制文件。在深入到这个层级结构的过程中，我不得不停下来思考是否值得转换那个额外的实用程序或层。（如果我没有这么做，本章的长度会翻倍，实用性却会减半。）为此，我决定转换以下内容：

+   数据库库

+   单元和库接口测试

+   各种*util*目录中找到的实用程序和其他此类高级程序

+   在 *xflaim* 库中找到的 Java 和 C# 语言绑定

我还会转换 C# 单元测试，但不会涉及 Java 单元测试，因为我已经在使用 Automake 的 `JAVA` 主文件转换 Java 语言绑定。由于 Automake 对 C# 没有支持，我不得不自己提供一切，因此我将转换整个 C# 代码库。这将提供一个编写完全不受支持的 Automake 产品类代码的示例。

### 入门

如前所述，我的第一个真正的设计决策是如何将原始的 FLAIM 项目组织成子项目。结果表明，现有的目录布局几乎是完美的。我在顶层的*flaim*目录中创建了一个主*configure.ac*文件，该目录就在版本库的根目录下。这个最上层的*configure.ac*文件充当每个四个下级项目（ftk、flaim、flaimsql 和 xflaim）的 Autoconf 控制文件。

我通过将工具包视为一个纯外部依赖项来管理 FLAIM 工具包的数据库库依赖关系，该依赖项由`make`变量`FTKINC`和`FTKLIB`定义。我有条件地定义了这些变量，以指向多个不同的来源，包括已安装的库，甚至是用户指定的配置脚本选项中的位置。

#### *添加 configure.ac 文件*

在以下的目录布局中，我使用了注释列来指示每个*configure.ac*文件的位置。这些文件代表一个可能被打包并独立分发的项目。

```
$ tree -d --charset=ascii FLAIM
FLAIM                           configure.ac (flaim-projects)
|-- flaim                       configure.ac (flaim)
|   |-- debian
|   |-- docs
|   |-- sample
|   |-- src
|   `-- util
|-- ftk                         configure.ac (ftk)
|   |-- debian
|   |-- src
|   `-- util
|-- sql                         configure.ac (flaimsql)
|   `-- src
--snip--
`-- xflaim                      configure.ac (xflaim)
    |-- csharp
    --snip--
    |-- java
    --snip--
    |-- sample
    --snip--
    |-- src
    `-- util
--snip--
```

我的下一个任务是创建这些*configure.ac*文件。顶层文件非常简单，因此我手动创建了它。与项目相关的文件更复杂，因此我让`autoscan`工具为我完成大部分工作。Listing 14-1 展示了顶层的*configure.ac*文件。

```
   #                                               -*- Autoconf -*-
   # Process this file with autoconf to produce a configure script.

   AC_PREREQ([2.69])
➊ AC_INIT([flaim-projects], [1.0])
➋ AM_INIT_AUTOMAKE([-Wall -Werror foreign])
➌ AM_PROG_AR
➍ LT_PREREQ([2.4])
   LT_INIT([dlopen])

➎ AC_CONFIG_MACRO_DIRS([m4])
➏ AC_CONFIG_SUBDIRS([ftk flaim sql xflaim])
   AC_CONFIG_FILES([Makefile])
   AC_OUTPUT
```

*Listing 14-1*: configure.ac: *Umbrella 项目的 Autoconf 输入文件*

这个*configure.ac*文件简短而简单，因为它没有做太多工作；尽管如此，这里有一些新的重要概念。我在➊处发明了`flaim-projects`这个名称和版本号`1.0`。除非项目目录结构发生重大变化，或者维护者决定发布一个包含所有子项目的完整捆绑包，否则这些内容不太可能改变。

**注意**

*对于你自己的项目，考虑使用`AC_INIT`宏的可选第三个参数。你可以在此处添加电子邮件或网址，指示用户可以在哪里提交 bug 报告。此参数的内容会显示在*configure*输出中。*

像这样的 Umbrella 项目中最重要的方面是➏处的`AC_CONFIG_SUBDIRS`宏，这是本书中我尚未介绍的。该参数是一个以空格分隔的子项目列表，每个子项目都是一个完全符合*GCS*标准的独立项目。以下是该宏的原型：

```
AC_CONFIG_SUBDIRS(dir1[ dir2 ... dirN])
```

它允许维护者以与 Automake `SUBDIRS` 配置单个项目中的目录层次结构类似的方式设置项目层次结构。

因为这四个子项目包含所有实际的构建功能，所以这个*configure.ac*文件只是充当一个控制文件，将所有指定的配置选项传递给宏参数中给出的顺序中的每个子项目。必须首先构建 FLAIM 工具包项目，因为其他项目依赖于它。

##### Umbrella 项目中的 Automake

Automake 通常要求在顶层项目目录中存在几个文本文件，包括*AUTHORS*、*COPYING*、*INSTALL*、*NEWS*、*README*和*ChangeLog*文件。最好在总项目中不必处理这些文件。实现这一目标的一种方法是干脆不在总项目中使用 Automake。我要么得为这个目录编写自己的*Makefile.in*模板，要么只用一次 Automake 来生成一个*Makefile.in*模板，然后将其与`automake --add-missing`（或`autoreconf -i`）添加的*install-sh*和*missing*脚本一起提交到仓库中，作为项目的一部分。一旦这些文件到位，我就可以从主*configure.ac*文件中删除`AM_INIT_AUTOMAKE`。

另一种选择是保留 Automake，并在`AM_INIT_AUTOMAKE`的宏可选参数中使用`foreign`选项（这是我在 ➋ 所做的）。这个参数包含一串以空格分隔的选项，告诉 Automake 如何替代特定的 Automake 命令行选项。当`automake`解析*configure.ac*文件时，它会记录下这些选项并启用它们，就像它们是从命令行传递的一样。`foreign`选项告诉 Automake，该项目不会完全遵循 GNU 标准，因此 Automake 不会要求常见的 GNU 项目文本文件。

我选择了两种方法中的后者，因为我可能会在某个时候想修改下属项目的列表，而不希望每次都手动调整生成的*Makefile.in*模板。我还在这个列表中传递了`-Wall`和`-Werror`选项，这表明 Automake 应该启用所有 Automake 特有的警告并将其报告为错误。这些选项与用户的编译环境无关——仅与 Automake 处理相关。

##### 为什么要添加 Libtool 宏？

为什么在 ➍ 包含这些昂贵的 Libtool 宏？好吧，尽管我在总项目中没有使用 Libtool，但低级项目期望包含项目提供所有必要的脚本，而`LT_INIT`宏提供了*ltmain.sh*脚本。如果你在总项目中没有初始化 Libtool，像`autoreconf`这样的工具（它实际上会在*父*目录中查找，来判断当前项目是否为子项目）会因为找不到当前项目的*configure.ac*文件所需要的脚本而失败。

例如，`autoreconf`期望在 ftk 项目的顶级目录中找到名为*../ltmain.sh*的文件。注意这里对父目录的引用：`autoreconf`通过检查父目录发现，ftk 实际上是一个更大项目的子项目。为了避免多次安装所有辅助脚本，Autotools 生成代码，查找项目父目录中的脚本。这是为了减少将这些脚本安装到多项目包中的副本数量。^(8) 如果我在主项目中不使用`LT_INIT`，则无法在子项目中成功运行`autoreconf`，因为*ltmain.sh*脚本将不会出现在项目的父目录中。

##### 添加宏子目录

➎处的`AC_CONFIG_MACRO_DIRS`宏表示一个子目录的名称，`aclocal`工具可以在该目录中找到所有特定于项目的 M4 宏文件。以下是原型：

```
AC_CONFIG_MACRO_DIRS(macro-dir)
```

本目录中的*.m4*宏文件最终通过`m4_include`语句引用，在`aclocal`生成的*aclocal.m4*文件中，`autoconf`会读取该文件。这个宏用一个包含单独宏或较小宏集合的目录替换了原来的*acinclude.m4*文件，每个宏都定义在各自的*.m4*文件中。^(9)

我通过`AC_CONFIG_MACRO_DIRS`参数指明，所有要添加到*aclocal.m4*中的本地宏文件都位于一个名为*m4*的子目录中。作为附带功能，当执行`autoreconf -i`时，然后执行带有各自*add-missing*选项的必要 Autotools 工具时，这些工具会注意到*configure.ac*中使用了此宏，并将任何缺少的系统宏文件添加到*m4*目录中。

我选择在这里使用`AC_CONFIG_MACRO_DIRS`的原因是，如果没有以这种方式启用宏目录选项，Libtool 将不会将其附加的宏文件添加到项目中。相反，它会抱怨应该将这些文件添加到*acinclude.m4*中。^(10)

由于这是一个相对复杂的项目，我希望 Autotools 为我完成这项工作，因此决定使用这个宏目录功能。未来的 Autotools 版本可能会要求使用这种形式，因为它被认为是将宏文件添加到*aclocal.m4*的更现代的方式，而不是使用单一的用户生成的*acinclude.m4*文件。

关于这个宏的最后一个思考：如果你在 Autoconf 手册中查找它，你是找不到的——至少目前找不到，因为它不是一个 Autoconf 宏，而是一个 Automake 宏。它的前缀是 `AC_`，因为最初的设计目标是未来某个版本的 Autoconf 会接管这个宏。它比它的单一前身功能更强大，后者在 Autoconf 手册中有文档，但在 Automake 出现之前，这个功能并不需要。事实上，我有相当可靠的消息来源（预发布的 Autoconf *ChangeLog*），表示当 Autoconf 2.70 发布时，这个宏的所有权将会转移。

我们在这里尚未讨论的一个项目是 ➌ 位置的 `AM_PROG_AR` 宏。这是一个较新的 Automake 宏。本书的第一版没有使用它。当我更新 Autotools 时，突然 `autoreconf` 报告需要它，所以我加上了它，问题就解决了。Autoconf 手册简单地说明，如果你想使用具有特殊接口（如 Microsoft `lib`）的归档器（`ar`），你需要它。事实上，真正发出抱怨的是 Libtool，它似乎习惯性地抱怨没有包含它认为你应该使用的其他 Autotools 特性。我添加了它以消除警告。

#### *顶层 Makefile.am 文件*

关于总项目的唯一其他要点是顶层 *Makefile.am* 文件，如清单 14-2 所示。

```
➊ ACLOCAL_AMFLAGS = -I m4

➋ EXTRA_DIST = README.W32 tools win32

➌ SUBDIRS = ftk flaim sql xflaim

➍ rpms srcrpm:
          for dir in $(SUBDIRS); do \
            (cd $$dir && $(MAKE) $(AM_MAKEFLAGS) $@) || exit 1; \
          done
  .PHONY: rpms srcrpm
```

*清单 14-2*: Makefile.am: *总项目 Automake 输入文件*

根据 Automake 文档，在 ➊ 位置定义的 `ACLOCAL_AMFLAGS` 变量应该在任何使用 `AC_CONFIG_MACRO_DIR`（单数）作为 *configure.ac* 文件中的配置项的项目的顶层 *Makefile.am* 文件中定义。此行指定的标志告诉 `aclocal` 在执行时应该在哪里查找宏文件，这些规则是在 *Makefile.am* 中定义的。此选项的格式类似于 C 编译器命令行的 `-I` 指令；你也可以指定其他 `aclocal` 命令行选项。

当使用旧版 `AC_CONFIG_MACRO_DIR` 时，此变量曾经是必需的，但随着新版 `AC_CONFIG_MACRO_DIRS` 的出现，你不再需要此变量，因为它生成的代码使 Automake 能够理解应该传递给 `aclocal` 的选项。不幸的是，当 Libtool 看到你在 *Makefile.am* 文件中使用宏目录而没有这个变量时，它还是会在 `autoreconf` 时发出警告。我希望当 Autoconf 接管了这个新宏（当然，还需要 Libtool 的后续版本发布）时，这个噪音会消失。

Autotools 在两个不相关的地方使用这个变量。第一个是在生成的 `make` 规则中，用于根据各种输入源更新 *aclocal.m4* 文件。此规则及其支持的变量定义见于清单 14-3，这是从 Autotools 生成的 Makefile 中复制的代码片段。

```
ACLOCAL_M4 = $(top_srcdir)/aclocal.m4
ACLOCAL=${SHELL} .../flaim-ch8-10/missing --run aclocal-1.10
ACLOCAL_AMFLAGS = -I m4
$(ACLOCAL_M4): $(am__aclocal_m4_deps)
        cd $(srcdir) && $(ACLOCAL) $(ACLOCAL_AMFLAGS)
```

*清单 14-3：用于更新* aclocal.m4 *的`make` 规则和所使用的变量，它们来自各种依赖项*

`ACLOCAL_AMFLAGS` 定义也在执行 `autoreconf` 时使用，`autoreconf` 会扫描顶级 *Makefile.am* 文件中的此定义，并直接将该值传递给命令行中的 `aclocal`。请注意，`autoreconf` 不会对这个字符串进行变量扩展，因此如果你在文本中添加了 shell 或 `make` 变量引用，它们在 `autoreconf` 执行 `aclocal` 时将不会被扩展。

返回到清单 14-2，我在 ➋ 使用了 `EXTRA_DIST` 变量，以确保几个额外的顶级文件能够被分发——这些文件和目录是特定于 Windows 构建系统的。对于总体项目来说，这并不是至关重要的，因为我不打算在这个层次上创建分发包，但我喜欢做到完整。

➌ 处的 `SUBDIRS` 变量重复了 *configure.ac* 文件中 `AC_CONFIG_SUBDIRS` 宏的信息。我尝试创建一个 shell 替代变量，并用 `AC_SUBST` 导出它，但没有成功——当我运行 `autoreconf` 时，出现了一个错误，提示我应该在 `AC_CONFIG_SUBDIRS` 宏参数中使用字面量。

`rpms` 和 `srcrpm` 目标在 ➍ 处允许最终用户为基于 RPM 的 Linux 系统构建 RPM 包。这个规则中的 shell 命令只是将用户指定的目标和变量依次传递给每个低级项目，就像我们在第三章、第四章和第五章中用手写的 makefile 和 *Makefile.in* 模板所做的那样。

在以这种方式将控制权传递给低级 makefile 时，你应该努力遵循这个模式。传递 `AM_MAKEFLAGS` 的扩展使得低级 makefile 能够访问当前或父级 makefile 中定义的相同 `make` 标志。然而，你可以为这种递归的 `make` 代码添加更多功能。要查看 Automake 如何将控制权传递给低级 makefile 以处理它自己的目标，可以打开一个 Automake 生成的 *Makefile.in* 模板，并搜索文本 "`$(am__recursive_targets):`"。该目标下的代码准确地显示了 Automake 是如何做的。虽然初看起来很复杂，但这段代码实际上只执行了两项额外的任务。首先，它确保 `make -k` 的继续错误功能能够正常工作。其次，它确保如果 `SUBDIRS` 变量中包含当前目录（`.`），则会正确处理。

这让我想到了关于这段代码的最后一点：如果你选择以这种方式编写自己的递归目标（稍后我们将在讨论 FLAIM 构建系统转换时看到其他示例），你应该避免在`SUBDIRS`变量中使用点，或者增强 shell 代码以处理这种特殊情况。如果不这样做，用户在尝试构建这些目标时，可能会陷入无休止的递归循环。有关此主题的更广泛讨论，请参见第 505 页的“条目 2：实现递归扩展目标”。

#### FLAIM 子项目

我使用`autoscan`为 ftk 项目生成了一个起点。`autoscan`工具在查找信息时有些挑剔。如果你的项目没有一个名为*Makefile*的 makefile，或者如果你的项目已经包含了一个 Autoconf 的*Makefile.in*模板，`autoscan`将不会将任何关于所需库的信息添加到*configure.scan*输出文件中。它无法通过任何其他方式来确定这些信息，除非查看你旧的构建系统，而它只有在条件完全合适的情况下才会这么做。

鉴于 ftk 项目遗留的 makefile 的复杂性，我对`autoscan`解析它以获取库信息的能力印象深刻。示例 14-4 展示了结果中*configure.scan*文件的一部分。

```
--snip--
AC_PREREQ([2.69])
AC_INIT(FULL-PACKAGE-NAME, VERSION, BUG-REPORT-ADDRESS)
AC_CONFIG_SRCDIR([src/ftktext.cpp])
AC_CONFIG_HEADERS([config.h])

# Checks for programs.
AC_PROG_CXX
AC_PROG_CC
AC_PROG_INSTALL

# Checks for libraries.
# FIXME: Replace `main' with a function in `-lc':
AC_CHECK_LIB([c], [main])
# FIXME: Replace `main' with a function in...
AC_CHECK_LIB([crypto], [main])
--snip--
AC_CONFIG_FILES([Makefile])
AC_OUTPUT
```

*示例 14-4：在 ftk 项目目录结构上运行`autoscan`时输出的一部分*

#### *FLAIM 工具包 configure.ac 文件*

在修改并重命名这个*configure.scan*文件后，结果生成的*configure.ac*文件包含了许多新的构造，接下来几节我将讨论这些内容。为了便于讨论，我将这个文件分成了两部分，第一部分显示在示例 14-5 中。

```
   #                                               -*- Autoconf -*-
   # Process this file with autoconf to produce a configure script.
   AC_PREREQ([2.69])
➊ AC_INIT([FLAIMTK],[1.2],[flaim-users@lists.sourceforge.net])
➋ AM_INIT_AUTOMAKE([-Wall -Werror])
   AM_PROG_AR
   LT_PREREQ([2.4])
   LT_INIT([dlopen])

➌ AC_LANG([C++])

➍ AC_CONFIG_MACRO_DIRS([m4])
➎ AC_CONFIG_SRCDIR([src/flaimtk.h])
   AC_CONFIG_HEADERS([config.h])

   # Checks for programs.
   AC_PROG_CXX
   AC_PROG_INSTALL

   # Checks for optional programs.
➏ FLM_PROG_TRY_DOXYGEN

   # Configure options: --enable-debug[=no].
➐ AC_ARG_ENABLE([debug],
     [AS_HELP_STRING([--enable-debug],
       [enable debug code (default is no)])],
     [debug="$withval"], [debug=no])

   # Configure option: --enable-openssl[=no].
   AC_ARG_ENABLE([openssl],
     [AS_HELP_STRING([--enable-openssl],
       [enable the use of openssl (default is no)])],
     [openssl="$withval"], [openssl=no])

   # Create Automake conditional based on the DOXYGEN variable
➑ AM_CONDITIONAL([HAVE_DOXYGEN], [test -n "$DOXYGEN"])
   #AM_COND_IF([HAVE_DOXYGEN], [AC_CONFIG_FILES([docs/doxyfile])])
➒ AS_IF([test -n "$DOXYGEN"], [AC_CONFIG_FILES([docs/doxyfile])])
--snip--
```

*示例 14-5*：ftk/configure.ac：*ftk 项目的 configure.ac 文件的前半部分*

在➊处，你会看到我为`autoscan`在`AC_INIT`宏中留下的占位符替换了真实值。在➋处，我添加了对`AM_INIT_AUTOMAKE`、`LT_PREREQ`和`LT_INIT`的调用，在➍处，我添加了对`AC_CONFIG_MACRO_DIRS`的调用。（暂时忽略`AM_PROG_AR`宏——我稍后会在本章中解释它。）

**注意**

*这次我没有在*`AM_INIT_AUTOMAKE`*中使用*`foreign`*关键字。由于这是一个真正的开源项目，FLAIM 的开发者（或者至少应该）希望拥有这些文件。我使用了*`touch`*命令来创建 GNU 项目文本文件的空版本，^(11)，除了*COPYING*和*INSTALL*，*这两个文件是*`autoreconf`*添加的。*

新的结构出现在 ➌ 处，它是 `AC_LANG` 宏，表示 Autoconf 在生成 `configure` 中的编译测试时应使用的编程语言（从而确定编译器）。我传递了 `C++` 作为参数，这样 Autoconf 就会通过 `CXX` 变量使用 C++ 编译器来编译这些测试，而不是通过 `CC` 变量使用默认的 C 编译器。然后，我删除了 `AC_PROG_CC` 宏调用，因为该项目的源代码完全是用 C++ 编写的。

我将 ➎ 处的 `AC_CONFIG_SRCDIR` 文件参数改成了一个对我来说更合适的参数，而不是 `autoscan` 随机选择的那个。

`FLM_PROG_TRY_DOXYGEN` 宏在 ➏ 处是我编写的自定义宏。下面是它的原型：

```
FLM_PROG_TRY_DOXYGEN([quiet])
```

我将在 第十六章 中详细讲解这个宏是如何工作的。目前只需要知道它管理一个名为 `DOXYGEN` 的宝贵变量。如果该变量已经设置，这个宏什么也不做；如果该变量没有设置，它会扫描系统搜索路径，查找 `doxygen` 程序，如果找到了，就将该变量设置为程序名称。我会在介绍 xflaim 项目时解释 Autoconf 的宝贵变量。

在 ➐ 处，我使用 `AC_ARG_ENABLE` 向 `configure` 的命令行解析器添加了几个配置选项。当我们讨论其他使用这些宏定义的变量的新结构时，我会更全面地讨论这些调用的细节。

##### Automake 配置特性

Automake 提供了我在 ➑ 处使用的 `AM_CONDITIONAL` 宏；它的原型如下：

```
AM_CONDITIONAL(variable, condition)
```

*`variable`* 参数是一个 Automake 条件名称，你可以在 *Makefile.am* 文件中使用它来测试相关条件。*`condition`* 参数是一个 *shell 条件*——一段 shell 脚本，可以用作 shell `if-then` 语句中的条件。事实上，这正是该宏内部如何使用 *`condition`* 参数的方式，所以它必须格式化为一个正确的 `if-then` 语句 *条件* 表达式：

```
if condition; then...
```

`AM_CONDITIONAL` 宏总是定义两个 Autoconf 替换变量，分别是 *`variable`*`_TRUE` 和 *`variable`*`_FALSE`。如果 *`condition`* 为真，*`variable`*`_TRUE` 为空，而 *`variable`*`_FALSE` 被定义为一个井号（`#`），表示在 makefile 中注释的开始。如果 *`condition`* 为假，这两个替换变量的定义会被反转；也就是说，*`variable`*`_FALSE` 为空，而 *`variable`*`_TRUE` 变成了井号。Automake 使用这些变量有条件地注释掉你在 Automake 条件语句中定义的 makefile 脚本部分。

这一实例的`AM_CONDITIONAL`定义了条件名`HAVE_DOXYGEN`，您可以在项目的*Makefile.am*文件中使用它，根据是否能够成功执行`doxygen`（通过`DOXYGEN`变量）有条件地执行某些操作。在*Makefile.am*中的条件为真时，`make`脚本中的任何行都以`@`*`variable`*`_TRUE@`为前缀，在 Automake 生成的*Makefile.in*模板中。相反，任何在 Automake 条件测试为假时找到的行都以`@`*`variable`*`_FALSE@`为前缀。当`config.status`从*Makefile.in*生成*Makefile*时，这些行根据条件的真假被注释掉（以井号为前缀）或不被注释掉。

使用`AM_CONDITIONAL`有一个注意事项：您不能在*configure.ac*文件中有条件地调用它（例如，在 shell 的`if-then-else`语句中）。您不能有条件地定义替换变量——您可以根据指定的条件不同地定义它们的内容，但这些变量本身在 Autoconf 创建`configure`脚本时要么已定义，要么未定义。由于 Automake 生成的模板文件是在用户执行`configure`之前很久就创建的，因此 Automake 必须能够依赖这些变量的存在，无论它们是如何定义的。

在`configure`脚本中，您可能希望根据 Automake 条件的值执行其他 Autoconf 操作。这时，位于➒的（已注释的）Automake 提供的`AM_COND_IF`宏就发挥作用了。^(12) 它的原型如下：

```
AM_COND_IF(conditional-variable, [if-true], [if-false])
```

如果*`conditional-variable`*在先前调用`AM_CONDITIONAL`时被定义为真，则执行*`if-true`* shell 脚本（包括任何 Autoconf 宏调用）。否则，执行*`if-false`* shell 脚本。

现在假设，举个例子，您希望有条件地构建项目目录结构的一部分——例如，基于 Automake 条件`HAVE_DOXYGEN`构建*xflaim/docs/doxygen*目录。也许您在*Makefile.am*文件中的 Automake 条件语句内将该子目录附加到`SUBDIRS`变量中（我实际上正在做这件事，正如您将在第 388 页的“FLAIM 工具包 Makefile.am 文件”一节中看到的）。由于如果条件为假，`make`不会构建项目目录结构的这一部分，因此在配置过程中没有理由让`config.status`处理该目录中的*doxyfile.in*模板。因此，您可以在*configure.ac*文件中使用列出 14-6 中显示的代码。

```
--snip--
AM_CONDITIONAL([HAVE_DOXYGEN], [test -n "$DOXYGEN"])
AM_COND_IF([HAVE_DOXYGEN], [AC_CONFIG_FILES([docs/doxyfile])])
#AS_IF([test -n "$DOXYGEN"], [AC_CONFIG_FILES([docs/doxyfile])])
--snip--
```

*列出 14-6*: ftk/configure.ac: *使用`AM_COND_IF`有条件地配置模板*

在此代码存在的情况下，如果用户系统上未安装`doxygen`，`configure`将根本不会处理*docs*目录中的*doxyfile.in*模板。

**注意**

*docs/Makefile.in*模板不应该包含在这里，因为*`dist`*目标必须能够在执行诸如*`all`*和*`clean`*等构建目标时，处理项目中的所有目录——无论它们是否是有条件构建的。因此，你不应在 configure.ac 中有条件地处理*Makefile.in*模板。*然而，你当然可以有条件地处理其他类型的模板。*

在➒行之后的那一行是使用*M4sh*（一种内置于 Autoconf 的宏库，旨在简化编写可移植的 Bourne shell 脚本）的替代方法来完成相同的事情。这里是原型：

```
AS_IF(test1, [run-if-true], ..., [run-if-false])
```

在第二个和最后一个参数之间省略的可选参数是*`test`*`N`和*`run-if-true`*参数的配对。最终，这个宏的工作方式就像一个`if-then-elif...`的 Shell 语句，带有用户指定数量的`elif`条件。

Listing 14-7 展示了 ftk 的*configure.ac*文件的后半部分。

```
   --snip--
   # Configure for large files, even in 32-bit environments
➊ AC_SYS_LARGEFILE
   # Check for pthreads
➋ AX_PTHREAD(
     [AC_DEFINE([HAVE_PTHREAD], [1],
       [Define if you have POSIX threads libraries and header files.])
     LIBS="$PTHREAD_LIBS $LIBS"
     CFLAGS="$CFLAGS $PTHREAD_CFLAGS"
     CXXFLAGS="$CXXFLAGS $PTHREAD_CXXFLAGS"])

➌ # Checks for libraries.
   AC_SEARCH_LIBS([initscr], [ncurses])
   AC_CHECK_HEADER([curses.h],,[echo "*** Error: curses.h not found - install
   curses devel package."; exit 1])
   AC_CHECK_LIB([rt], [aio_suspend])
   AS_IF([test "x$openssl" = xyes],
   ➍ [AC_DEFINE([FLM_OPENSSL], [1], [Define to use openssl])
      AC_CHECK_LIB([ssl], [SSL_new])
      AC_CHECK_LIB([crypto], [CRYPTO_add])
      AC_CHECK_LIB([dl], [dlopen])
      AC_CHECK_LIB([z], [gzopen])])

➎ # Checks for header files.
   AC_HEADER_RESOLV
   AC_CHECK_HEADERS([arpa/inet.h fcntl.h limits.h malloc.h netdb.h netinet/in.h
   stddef.h stdlib.h string.h strings.h sys/mount.h sys/param.h sys/socket.h sys/
   statfs.h sys/statvfs.h sys/time.h sys/vfs.h unistd.h utime.h])

   # Checks for typedefs, structures, and compiler characteristics.
   AC_CHECK_HEADER_STDBOOL
   AC_C_INLINE
   AC_TYPE_INT32_T
   AC_TYPE_MODE_T
   AC_TYPE_PID_T
   AC_TYPE_SIZE_T
   AC_CHECK_MEMBERS([struct stat.st_blksize])
   AC_TYPE_UINT16_T
   AC_TYPE_UINT32_T
   AC_TYPE_UINT8_T

   # Checks for library functions.
   AC_FUNC_LSTAT_FOLLOWS_SLASHED_SYMLINK
   AC_FUNC_MALLOC
   AC_FUNC_MKTIME
   AC_CHECK_FUNCS([atexit fdatasync ftruncate getcwd gethostbyaddr gethostbyname
   gethostname gethrtime gettimeofday inet_ntoa localtime_r memmove memset mkdir
   pstat_getdynamic realpath rmdir select socket strchr strrchr strstr])

   # Configure DEBUG source code, if requested.
➏ AS_IF([test "x$debug" = xyes],
     [AC_DEFINE([FLM_DEBUG], [1], [Define to enable FLAIM debug features])])

➐ --snip--

➑ AC_CONFIG_FILES([Makefile
                      docs/Makefile
                      obs/Makefile
 obs/flaimtk.spec
                      src/Makefile
                      util/Makefile
                      src/libflaimtk.pc])

   AC_OUTPUT

   # Fix broken libtool
   sed 's/link_all_deplibs=no/link_all_deplibs=yes/' libtool >libtool.tmp && \
     mv libtool.tmp libtool

➒ cat <<EOF

     FLAIM toolkit ($PACKAGE_NAME) version $PACKAGE_VERSION
     Prefix.........: $prefix
     Debug Build....: $debug
     Using OpenSSL..: $openssl
     C++ Compiler...: $CXX $CXXFLAGS $CPPFLAGS
     Linker.........: $LD $LDFLAGS $LIBS
     Doxygen........: ${DOXYGEN:-NONE}

   EOF
```

*Listing 14-7*: ftk/configure.ac: *ftk 项目的* configure.ac *文件的后半部分*

在➊处，我调用了`AC_SYS_LARGEFILE`宏。如果用户使用的是 32 位系统，该宏会确保向*C 预处理器定义（以及可能的编译器选项）*中添加适当的定义，从而强制使用 64 位文件寻址（也称为*大文件*）。有了这些变量，C 库中的大地址感知文件 I/O 函数就可以在项目源代码中使用。作为一个数据库系统，FLAIM 非常重视这个特性。

近几年，32 位通用计算机系统已经不再那么流行，因为像英特尔和微软这样的公司在关于未来版本产品的声明中表示，将不再支持 32 位地址空间。然而，由于市场压力，数百万现有的 32 位系统使得它们稍微放缓了这一言辞，回归了更务实的视角。尽管如此，32 位 PC 将在不久的将来退出历史舞台。即便如此，Linux 仍将继续在 32 位系统上运行，因为许多嵌入式系统仍然从使用更小、更节能的 32 位微处理器中获得显著的好处。

##### 正确处理线程

在➋处有另一个新的构造体，`AX_PTHREAD`。在 Jupiter 项目中，我仅通过`-lpthread`链接器标志将`jupiter`程序与*pthreads*库链接起来。但坦白说，这不是使用*pthreads*的正确方式。

在存在多个执行线程的情况下，必须配置许多标准 C 库函数以使其以线程安全的方式运行。你可以通过确保一个或多个预处理器定义在所有标准库头文件被编译到程序中时可见来实现这一点。这些 C 预处理器定义必须在编译器命令行上定义，并且在编译器供应商之间没有标准化。

一些供应商为构建单线程和多线程程序提供完全不同的标准库，因为将线程安全性添加到库中会在某种程度上降低性能。编译器供应商（正确地）认为，通过为这些目的提供不同版本的标准库，他们是在帮你一个忙。在这种情况下，必须告诉链接器使用正确的运行时库。

不幸的是，每个供应商都有自己实现多线程的方式，从编译器选项到库名称再到预处理器定义。但这个问题有一个合理的解决方案：GNU Autoconf Archive^(13)提供了一个名为`AX_PTHREAD`的宏，它检查用户的编译器，并为各种平台提供正确的标志和选项。

这个宏非常简单易用：

```
AX_PTHREAD(action-if-found[, action-if-not-found])
```

它设置了几个环境变量，包括`PTHREAD_CFLAGS`、`PTHREAD_CXXFLAGS`和`PTHREAD_LIBS`。调用者需要通过向*`action-if-found`*参数添加 Shell 代码来正确使用这些变量。如果你的整个项目都是多线程的，事情就更简单了：你只需要将这些变量附加到标准的`CFLAGS`、`CXXFLAGS`和`LIBS`变量中，或者在这些变量内使用它们。FLAIM 项目的代码库完全是多线程的，所以我选择了这样做。

如果你检查*ftk/m4*目录中的*ax_pthread.m4*文件，你可能会期待看到一个大的`case`语句，用于设置已知平台和编译器组合的选项，但那并不是 Autoconf 的方式。

相反，该宏包含了一个已知的*pthread*编译器选项长列表，生成的`configure`脚本使用主机编译器依次用这些选项编译一个小的*pthreads*程序。编译器识别并能正确构建测试程序的标志将被添加到`PTHREAD_CFLAGS`和`PTHREAD_CXXFLAGS`变量中。通过这种方式，`AX_PTHREAD`即使在未来编译器选项发生重大变化时，也有很大可能继续正常工作——这就是 Autoconf 的方式。

##### 获取恰到好处的库

我删除了每个`AC_CHECK_LIB`宏调用上方的*FIXME*注释（见 Listing 14-4 中的*configure.scan*，page 378），这些注释位于 Listing 14-7 的➌位置。我开始用实际的库函数名称替换这些宏中的主要占位符，但随后我开始怀疑这些库是否都真的必要。我对`autoscan`的能力并不太担心，而是更关注原始 makefile 的真实性。在手写构建系统中，我偶尔会注意到作者会将一组库名称从一个 makefile 复制到另一个，直到程序可以在没有丢失符号的情况下构建成功。^(14)

我没有盲目地继续这种趋势，而是选择简单地注释掉所有 `AC_CHECK_LIB` 的调用，看看在构建过程中能走多远，然后根据需要逐个添加它们，以解决缺失的符号。除非你的项目需要消耗数百个库，否则这只需要几分钟的额外时间。我喜欢只链接对我的项目必要的库；这不仅能加速链接过程，而且在严格执行时，能为项目提供很好的文档。

*configure.scan* 文件包含了 14 个 `AC_CHECK_LIB` 的调用。事实证明，我的 64 位 Linux 系统上的 FLAIM 工具包只需要其中的三个——*pthread*、*ncurses* 和 *rt*——所以我删除了剩下的条目，并用 *ncurses* 和 *rt* 库中的实际函数替换了占位符参数。回头看，这个策略似乎非常成功，因为我从 14 个库减少到了 2 个。第三个库是 POSIX 线程（*pthreads*）库，通过我在前一节中讨论的 `AX_PTHREAD` 宏来添加。

我还将 *ncurses* 的 `AC_CHECK_LIB` 调用转换为 `AC_SEARCH_LIBS`，因为我怀疑未来的 FLAIM 平台可能会使用不同的库名称来实现 *curses* 功能。我希望为这些平台的构建系统做好准备，搜索额外的库。*ncurses* 库在大多数平台上是可选的，因此我添加了 `AC_CHECK_HEADER` 宏来检查 *curses.h*，并在 *`action-if-not-found`*（第三个）参数中显示用户应安装 *curses-development* 包的消息，同时以错误退出配置过程。这个规则是在配置阶段尽早发现问题，而不是在编译阶段。

##### 维护者定义的命令行选项

接下来的四个库在 ➍ 处的 Autoconf 条件语句中进行了检查。这个语句基于最终用户使用 `--enable-openssl` 命令行参数，`AC_ARG_ENABLE` 提供了该参数（请参见 Listing 14-5 中的 ➐，在 第 379 页）。

我在这里使用了 `AS_IF`，而不是 shell 的 `if-then` 语句，因为如果条件语句中的任何宏调用需要额外的宏来展开才能正常工作，`AS_IF` 将确保这些依赖项首先在条件语句外部展开。`AS_IF` 宏不仅是 *M4sh* 库的一部分，也是 Autoconf 自动依赖框架的一部分（该框架在《Autoconf 与 M4》一书的 第 439 页中详细讨论）。

在这种情况下，`openssl` 变量根据 `AC_ARG_ENABLE` 提供的默认值以及最终用户的命令行选项被定义为 `yes` 或 `no`。

`AC_DEFINE`宏，在`AS_IF`的第一个参数中调用，确保 C 预处理器变量`FLM_OPENSSL`在*config.h*头文件中被定义。接着，`AC_CHECK_LIB`宏确保将`-lssl`、`-lcrypto`、`-ldl`和`-lz`字符串添加到`LIBS`变量中，但仅在`openssl`变量被设置为`yes`时才会添加。我们并不希望强制要求用户安装这些库，除非他们请求了需要这些库的功能。

你可以在处理维护者定义的命令行选项（如`--enable-openssl`）时变得尽可能复杂。但要小心：某些级别的自动化可能会让用户感到惊讶。例如，自动启用该选项，因为你的检查发现 OpenSSL 库已安装并且可以访问，这可能会让人感到有些不安。

我把所有头文件和库函数检查都留在了➎位置，正如`autoscan`所规定的，因为通过源代码进行简单的文本扫描来查找头文件和函数名称可能相当准确。

然而，请注意，`autoscan`并没有将 ftk 源代码中使用的*所有*头文件都放入`AC_CHECK_HEADERS`参数中。`autoscan`工具的算法简单但有效：它会添加所有源代码中条件性包含的头文件。这种方法假设任何你条件性包含的头文件可能会由于移植性问题，在不同平台上以不同的方式包含。虽然这种方法通常是正确的，但并不总是正确的，所以你应该查看每个添加的头文件，找到源代码中的条件性包含，并更智能地评估是否应该将其添加到*configure.ac*中的`AC_CHECK_HEADERS`。

在这个项目中，一个很好的例子是*stdlib.h*的条件性包含。事实上，*stdlib.h*会在 Windows 构建时包含，也会在 Unix 构建时包含。但它不会在 NetWare 构建时包含。无论如何，`AC_CHECK_HEADERS`中并不需要检查它，有两个原因。首先，它在平台间已经得到广泛标准化；其次，这个构建系统是专为 Unix 系统设计的。^(15) 重点是，你应该仔细检查`autoscan`为你做了什么，以确定是否应该在你的项目中执行此操作。

在➏位置，我们看到基于`debug`变量内容的条件性（`AS_IF`）使用了`AC_DEFINE`。这是另一个环境变量，根据传递给`configure`的命令行参数的结果条件性地定义。`--enable-debug`选项将`debug`变量设置为`yes`，这最终启用了*config.h*中的`FLM_DEBUG` C 预处理器定义。`FLM_OPENSSL`和`FLM_DEBUG`已经在 FLAIM 项目源代码中使用。以这种方式使用`AC_DEFINE`允许最终用户决定哪些功能被编译到库中。

我在 ➐ 处省略了一大块涉及编译器和工具优化的代码，这些代码将在下一章中呈现。这个代码在所有项目的 *configure.ac* 文件中都是相同的。

最后，我将对位于 ➑ 处的 *docs*、*obs*、*src* 和 *util* 目录中的 makefile 以及 *obs/flaimtk.spec* 和 *src/libflaimtk.pc* 文件添加了对 `AC_CONFIG_FILES` 宏调用的引用，并在底部附近添加了我常用的 `cat` 语句，作为对我的配置状态的可视化验证。现在，只需忽略 `cat` 语句上方的 `sed` 命令。我将在《传递依赖》章节中详细介绍它，见 第 401 页。

#### *FLAIM 工具包的 Makefile.am 文件*

如果我们暂时忽略 Doxygen 和 RPM 特定目标的命令，*ftk/Makefile.am* 文件相对简单。Listing 14-8 显示了整个文件内容。

```
   ACLOCAL_AMFLAGS = -I m4

   EXTRA_DIST = GNUMakefile README.W32 debian netware win32

➊ if HAVE_DOXYGEN
     DOXYDIR = docs
   endif

   SUBDIRS = src util obs $(DOXYDIR)

➋ doc_DATA = AUTHORS ChangeLog COPYING INSTALL NEWS README

   RPM = rpm

➌ rpms srcrpm: dist
          (cd obs && $(MAKE) $(AM_MAKEFLAGS) $@) || exit 1
           rpmarch=`$(RPM) --showrc | \
             grep "^build arch" | sed 's/\(.*: \)\(.*\)/\2/'`; \
           test -z "obs/$$rpmarch" || \
             ( mv obs/$$rpmarch/* . && rm -rf obs/$$rpmarch )
           rm -rf obs/$(distdir)

➍ #dist-hook:
   #        rm -rf `find $(distdir) -name .svn`

   .PHONY: srcrpm rpms
```

*Listing 14-8*: ftk/Makefile.am: *FLAIM 工具包顶层 makefile 的全部内容*

在这个文件中，你会发现常见的 `ACLOCAL_AMFLAGS`、`EXTRA_DIST` 和 `SUBDIRS` 变量定义，但你也可以看到在 ➊ 处使用了 Automake 条件。`if` 语句允许我将另一个目录（*docs*）添加到 `SUBDIRS` 列表中，但前提是 `configure` 检测到 `doxygen` 程序可用。我在这里使用了一个单独的变量（`DOXYDIR`），但 Automake 条件也可以直接包围一个语句，通过 Automake `+=` 运算符将目录名（`doc`）追加到 `SUBDIRS` 变量中。

**注意**

*不要将 Automake 条件与 GNU Make 条件混淆，后者使用关键字 *`ifeq`*、*`ifneq`*、*`ifdef`* 和 *`ifndef`*。如果你尝试在*Makefile.am*中使用一个 Automake 条件，而没有在*configure.ac*中添加相应的 *`AM_CONDITIONAL`* 语句，Automake 会对此发出警告。正确使用这个结构时，Automake 会在 *`make`* 看到它之前，将其转换为 *`make`* 可以理解的内容。*

另一个新的结构（至少在顶层 *Makefile.am* 文件中）是使用了 ➋ 处的 `doc_DATA` 变量。FLAIM 工具包在其顶层目录中提供了一些额外的文档文件，我希望将它们安装上。通过在 `DATA` 主要部分使用 `doc` 前缀，我告诉 Automake 希望将这些文件作为数据文件安装到 `$(docdir)` 目录，默认情况下最终解析到 `$(prefix)`*/share/doc* 目录中。

在 `DATA` 变量中提到的文件，如果它们没有 Automake 的特殊含义，默认不会自动分发（即不会被加入到分发的 tar 包中），因此你需要手动分发它们，方法是将它们添加到 `EXTRA_DIST` 变量中列出的文件中。

**注意**

*我不需要在 *`EXTRA_DIST`* 中列出标准的 GNU 项目文本文件，因为它们总是会自动分发。然而，我确实需要在 *`doc_DATA`* 变量中提到这些文件，因为 Automake 不假设你想要安装哪些文件。*

我将推迟讨论 ➌ 处的 RPM 目标，直到下一章。

##### Automake -hook 和 -local 规则

Automake 识别两种类型的集成扩展，我称之为 `-local` 目标和 `-hook` 目标。Automake 识别并遵循 `-local` 扩展，适用于以下标准目标：

| `all` | `install-data` | `installcheck` |
| --- | --- | --- |
| `check` | `install-dvi` | `installdirs` |
| `clean` | `install-exec` | `maintainer-clean` |
| `distclean` | `install-html` | `mostlyclean` |
| `dvi` | `install-info` | `pdf` |
| `html` | `install-pdf` | `ps` |
| `info` | `install-ps` | `uninstall` |

在你的 *Makefile.am* 文件中将 `-local` 添加到这些目标中的任何一个，将导致相关命令在标准目标 *之前* 执行。Automake 通过为标准目标生成规则来实现这一点，使得 `-local` 版本成为其依赖项之一（如果存在的话）。^(16) 在《整理你的房间》（见 第 404 页）中，我将展示一个使用 `clean-local` 目标的示例。

`-hook` 目标稍有不同，它们在相应的标准目标执行 *之后* 执行。^(17) Automake 通过在标准目标命令列表的末尾添加另一个命令来实现这一点。该命令仅执行 `$(MAKE)` 在包含的 makefile 上，将 `-hook` 目标作为命令行目标。因此，`-hook` 目标以递归方式在标准目标命令的末尾执行。

以下标准的 Automake 目标支持 `-hook` 版本：

| `dist` | `install-data` | `uninstall` |
| --- | --- | --- |
| `distcheck` | `install-exec` |  |

Automake 会自动将所有现有的 `-local` 和 `-hook` 目标添加到生成的 makefile 中的 `.PHONY` 规则中。

在本书的第一版中，我在 *Makefile.am* 中使用了 `dist-hook` 目标（现在已被注释掉）来调整分发目录，调整发生在构建后，但在 `make` 从其内容构建分发档案之前。`rm` 命令删除了由于我将整个目录添加到 `EXTRA_DIST` 变量中而成为分发目录一部分的多余文件和目录。当你将目录名称添加到 `EXTRA_DIST` 中时（在本例中为 *debian*、*netware* 和 *win32*），这些目录中的所有内容都会被添加到分发中——甚至包括隐藏的仓库控制文件和目录。^(18)

清单 14-9 是生成的 *Makefile* 的一部分，展示了 Automake 如何将 `dist-hook` 融入到最终的 makefile 中。相关部分已被高亮显示。

```
--snip--
distdir: $(DISTFILES)
        ... # copy files into distdir
        $(MAKE) $(AM_MAKEFLAGS) top_distdir="$(top_distdir)" \
            distdir="$(distdir)" dist-hook
        ... # change attributes of files in distdir
--snip--
dist dist-all: distdir
        tardir=$(distdir) && $(am__tar) | GZIP=$(GZIP_ENV) gzip -c \
          >$(distdir).tar.gz
        $(am__remove_distdir)
--snip--
.PHONY: ... dist-hook ...
--snip--
dist-hook:
        rm -rf `find $(distdir) -name .svn`
--snip--
```

*清单 14-9：定义 `dist-hook` 目标的结果，在* ftk/Makefile.am

**注意**

*不要害怕深入生成的 makefile，查看 Automake 如何处理你的代码。虽然*`make`*命令中有相当多的丑陋 shell 代码，但大多数可以忽略。你通常更关心 Automake 生成的*`make`*规则，且很容易将它们分离出来。*

#### *设计 ftk/src/Makefile.am 文件*

我现在需要在 FLAIM 工具包项目的*src*和*utils*目录中创建*Makefile.am*文件。在创建这些文件时，我希望确保所有原有功能都得以保留。基本上，这包括：

+   正确构建 ftk 共享和静态库

+   正确指定所有已安装文件的安装位置

+   正确设置 ftk 共享库的版本信息

+   确保所有剩余的未使用文件都被分发

+   确保使用平台特定的编译器选项

在清单 14-10 中展示的模板应该涵盖大多数这些要点，因此我将在所有 FLAIM 库项目中使用它，并根据每个库的需求进行适当的增减。

```
EXTRA_DIST = ...

lib_LTLIBRARIES = ...
include_HEADERS = ...

xxxxx_la_SOURCES = ...
xxxxx_la_LDFLAGS = -version-info x:y:z
```

*清单 14-10：src 和 utils 目录的框架* Makefile.am *文件*

原始的*GNUMakefile*告诉我库的名称是*libftk.so*。这个名字在 Linux 上并不好，因为大多数三字母的库名已经被占用了。因此，我做出了一个决定，将*ftk*库重命名为*flaimtk*。

清单 14-11 显示了最终的*ftk/src/Makefile.am*文件的大部分内容。

```
➊ EXTRA_DIST = ftknlm.h

➋ lib_LTLIBRARIES = libflaimtk.la
➌ include_HEADERS = flaimtk.h

➍ pkgconfigdir = $(libdir)/pkgconfig
   pkgconfig_DATA = libflaimtk.pc

➎ libflaimtk_la_SOURCES = \
   ftkarg.cpp \
   ftkbtree.cpp \
   ftkcmem.cpp \
   ftkcoll.cpp \
   --snip--
   ftksys.h \
   ftkunix.cpp \
   ftkwin.cpp \
   ftkxml.cpp

➏ libflaimtk_la_LDFLAGS = -version-info 0:0:0
```

*清单 14-11*：ftk/src/Makefile.am：*整个文件内容，减去一些几十个源文件*

我将 Libtool 库名称*libflaimtk.la*添加到`lib_LTLIBRARIES`列表中的➋，并将清单 14-10 中其余宏的*`xxxxx`*部分更改为`libflaimtk`。我本可以手动输入所有源文件，但在阅读原始 makefile 时，我注意到它使用了 GNU `make`的函数宏`$(wildcard src/*.cpp)`来从*src*目录的内容构建文件列表。这告诉我，*src*目录中的所有*.cpp*文件都是库所需的（或者至少会被使用）。为了将文件列表添加到*Makefile.am*中，我使用了一个简单的 shell 命令，将其拼接到*Makefile.am*文件的末尾（假设我在*ftk/src*目录中）：

```
$ printf '%s \\\n' *.cpp >> Makefile.am
```

这让我在*ftk/src/Makefile.am*的底部得到了一个单列、以反斜杠结束、按字母顺序排列的所有*.cpp*文件列表。

**注意**

*不要忘记在*`printf`*参数周围加上单引号，这对于防止在生成列表时，第一个反斜杠对 shell 作为转义字符的解释是必要的。不管怎么引用，*`printf`*都会正确理解并解析*`\n`*字符。*

我将列表移到`libflaimtk_la_SOURCES`行下方➎，在等号后添加了一个反斜杠字符，并删除了最后一个文件后的反斜杠。另一种格式化技巧是简单地在每大约 70 个字符后用反斜杠和换行符包裹一行，但我更倾向于将每个文件放在单独的一行，特别是在转换过程的早期，这样我可以根据需要轻松地从列表中提取文件或添加文件。将文件放在单独的行上还能带来一个好处，即在查看拉取请求和其他`diff`风格的输出时，源文件列表更容易进行对比。

我必须手动检查*src*目录中的每个头文件，以确定它在项目中的使用情况。这里只有四个头文件，结果发现，FLAIM 工具包在 Unix 和 Linux 平台上唯一没有使用的是*ftknlm.h*，它是专门用于 NetWare 构建的。我将这个文件添加到➊位置的`EXTRA_DIST`列表中，以便进行分发；仅仅因为构建不使用它并不意味着用户不需要或不希望使用它。^(19)

（重新命名后的）*flaimtk.h*文件是唯一的公共头文件，因此我将它移入了➌位置的`include_HEADERS`列表。其他两个文件在库的构建过程中是内部使用的，因此我将它们保留在`libflaimtk_la_SOURCES`列表中。如果这是我自己的项目，我会将*flaimtk.h*移动到项目根目录下的*include*目录中，但记住，我的一个目标是尽量限制对目录结构和源代码的更改。移动这个头文件是一个哲学性的决定，我决定将其留给维护者来做出决定。^(20)

最后，我在原始 makefile 中注意到，*ftk*库的最后一个版本发布了 4.0 的接口版本。然而，由于我将库的名称从*libftk*更改为*libflaimtk*，我将此值重置为 0.0，因为它是一个不同的库。我在➏位置的`libflaimtk_la_LDFLAGS`变量中的`-version-info`选项里将*`x`*`:`*`y`*`:`*`z`*替换为`0:0:0`。

**注意**

*`0:0:0`的版本字符串是默认值，因此我本可以完全移除该参数并达到相同的效果。然而，包含它可以为新开发者提供一些关于如何在未来更改接口版本的见解。*

我在➍位置添加了`pkgconfigdir`和`pkgconfig_DATA`变量，以便为此项目提供支持安装 pkg-config 元数据文件的功能。有关 pkg-config 系统的更多信息，请参见第十章。

#### *继续处理 ftk/util 目录*

正确设计*Makefile.am*用于*util*目录时，需要再次检查原始的 makefile，以便支持更多的产品。快速浏览*ftk/util*目录后发现，那里只有一个源文件：*ftktest.cpp*。这看起来像是一个针对*ftk*库的测试程序，但我知道 FLAIM 的开发者在多种场景中都在使用它，除了仅仅用于测试构建。所以在这里，我需要做出一个设计决策：我应该把它作为一个普通程序还是作为一个检查程序来构建？

*检查程序*只有在执行`make check`时才会被构建，并且它们永远不会被安装。如果我希望`ftktest`作为常规程序构建，但不安装，我必须在程序列表变量中使用`noinst`前缀，而不是通常的`bin`前缀。

在任何情况下，我可能都希望将`ftktest`添加到`make check`期间执行的测试列表中，所以这里有两个问题：（1）我是否希望在`make check`期间自动运行`ftktest`，（2）我是否希望安装`ftktest`程序。考虑到 FLAIM 工具包是一个成熟的产品，我选择在`make check`期间构建`ftktest`并保持它未安装。

列表 14-12 展示了我的最终*ftk/util/Makefile.am*文件。

```
FTK_INCLUDE = -I$(top_srcdir)/src
FTK_LTLIB = ../src/libflaimtk.la

check_PROGRAMS = ftktest
ftktest_SOURCES = ftktest.cpp
ftktest_CPPFLAGS = $(FTK_INCLUDE)
ftktest_LDADD = $(FTK_LTLIB)

TESTS = ftktest
```

*列表 14-12*：ftk/util/Makefile.am：*此文件的最终内容*

我希望到现在为止你已经理解了`TESTS`和`check_PROGRAMS`之间的关系。坦率地说，`check_PROGRAMS`中列出的文件与`TESTS`中列出的文件之间**没有**任何关系。`check`目标只是确保在执行`TESTS`程序和脚本之前，`check_PROGRAMS`已经被构建。`TESTS`可以指任何不需要命令行参数即可执行的内容。这种职责分离使得系统非常简洁且灵活。

这就是 FLAIM 工具包库和工具的全部内容。我不知道你怎么想，但我宁愿维护这一小组简短的文件，而不愿意去维护一个单一的 2,200 行的 makefile！

### 设计 XFLAIM 构建系统

现在我完成了 FLAIM 工具包的工作，接下来我将进入 xflaim 项目。我选择从 xflaim 开始，而不是 flaim，因为它提供了最多的构建功能，可以转换为 Autotools，包括 Java 和 C#语言绑定（这些内容我将在下一章详细讨论）。在完成 xflaim 之后，覆盖其余的数据库项目就显得多余了，因为它们的过程是相同的，甚至更简单一些。不过，你可以在本书的 GitHub 仓库中找到其他构建系统文件。

我再次使用`autoscan`生成了*configure.ac*文件。在每个独立项目中使用`autoscan`非常重要，因为每个项目的源代码不同，因此会导致不同的宏被写入每个*configure.scan*文件中。^(21) 接着，我使用了在 FLAIM 工具包中使用的相同技术来创建 xflaim 的*configure.ac*文件。

#### *XFLAIM 的 configure.ac 文件*

在手动修改生成的*configure.scan*文件并将其重命名为*configure.ac*之后，我发现它在许多方面与工具包的*configure.ac*文件类似。它相当长，因此我只会展示列表 14-13 中的最重要区别。

```
   --snip--
➊ # Checks for optional programs.
   FLM_PROG_TRY_CSC
   FLM_PROG_TRY_CSVM
   FLM_PROG_TRY_JNI
   FLM_PROG_TRY_JAVADOC
   FLM_PROG_TRY_DOXYGEN

➋ # Configure variables: FTKLIB and FTKINC.
   AC_ARG_VAR([FTKLIB], [The PATH wherein libflaimtk.la can be found.])
   AC_ARG_VAR([FTKINC], [The PATH wherein flaimtk.h can be found.])
   --snip--
➌ # Ensure that both or neither is specified.
   if (test -n "$FTKLIB" && test -z "$FTKINC") || \
      (test -n "$FTKINC" && test -z "$FTKLIB"); then
     AC_MSG_ERROR([Specify both FTK library and include paths, or neither.])
   fi

   # Not specified? Check for FTK in standard places.
   if test -z "$FTKLIB"; then
   ➍ # Check for FLAIM toolkit as a sub-project.
      if test -d "$srcdir/ftk"; then
        AC_CONFIG_SUBDIRS([ftk])
        FTKINC='$(top_srcdir)/ftk/src'
        FTKLIB='$(top_builddir)/ftk/src'
      else
   ➎ # Check for FLAIM toolkit as a superproject.
        if test -d "$srcdir/../ftk"; then
          FTKINC='$(top_srcdir)/../ftk/src'
          FTKLIB='$(top_builddir)/../ftk/src'
        fi
      fi
    fi

➏ # Still empty? Check for *installed* FLAIM toolkit.
   if test -z "$FTKLIB"; then
     AC_CHECK_LIB([flaimtk], [ftkFastChecksum],
       [AC_CHECK_HEADERS([flaimtk.h])
          LIBS="-lflaimtk $LIBS"],
       [AC_MSG_ERROR([No FLAIM toolkit found. Terminating.])])
   fi

➐ # AC_SUBST command line variables from FTKLIB and FTKINC.
   if test -n "$FTKLIB"; then
     AC_SUBST([FTK_LTLIB], ["$FTKLIB/libflaimtk.la"])
     AC_SUBST([FTK_INCLUDE], ["-I$FTKINC"])
   fi

➑ # Automake conditionals
   AM_CONDITIONAL([HAVE_JAVA], [test "x$flm_prog_have_jni" = xyes])
   AM_CONDITIONAL([HAVE_CSHARP], [test -n "$CSC"])
   AM_CONDITIONAL([HAVE_DOXYGEN], [test -n "$DOXYGEN"])
   #AM_COND_IF([HAVE_DOXYGEN], [AC_CONFIG_FILES([docs/doxygen/doxyfile])])
   AS_IF([test -n "$DOXYGEN"], [AC_CONFIG_FILES([docs/doxygen/doxyfile])])
   --snip--
   AC_OUTPUT
   # Fix broken libtool
   sed 's/link_all_deplibs=no/link_all_deplibs=yes/' libtool >libtool.tmp && \
     mv libtool.tmp libtool

   cat <<EOF

     ($PACKAGE_NAME) version $PACKAGE_VERSION
     Prefix.........: $prefix
     Debug Build....: $debug
     C++ Compiler...: $CXX $CXXFLAGS $CPPFLAGS
     Linker.........: $LD $LDFLAGS $LIBS
     FTK Library....: ${FTKLIB:-INSTALLED}
     FTK Include....: ${FTKINC:-INSTALLED}
     CSharp Compiler: ${CSC:-NONE} $CSCFLAGS
     CSharp VM......: ${CSVM:-NONE}
     Java Compiler..: ${JAVAC:-NONE} $JAVACFLAGS
     JavaH Utility..: ${JAVAH:-NONE} $JAVAHFLAGS
     Jar Utility....: ${JAR:-NONE} $JARFLAGS
     Javadoc Utility: ${JAVADOC:-NONE}
     Doxygen........: ${DOXYGEN:-NONE}

   EOF
```

*列表 14-13*：xflaim/configure.ac：*这个 Autoconf 输入文件中最重要的部分*

首先，注意到我在➊处发明了几个更多的`FLM_PROG_TRY_*`宏。在这里，我正在检查以下程序的存在：C#编译器、C#虚拟机、Java 编译器、JNI 头文件和存根生成器、Javadoc 生成工具、Java 归档工具和 Doxygen。我为这些检查编写了单独的宏文件，并将它们添加到我的*xflaim/m4*目录中。

与工具包中使用的`FLM_PROG_TRY_DOXYGEN`宏一样，这些宏中的每一个都会尝试定位相关程序，但如果找不到程序，它们不会导致配置过程失败。我希望在这些程序可用时能使用它们，但不希望要求用户必须拥有它们才能构建基础库。

你会在➋处找到一个新宏，`AC_ARG_VAR`。像`AC_ARG_ENABLE`和`AC_ARG_WITH`宏一样，`AC_ARG_VAR`允许项目维护者扩展`configure`脚本的命令行接口。然而，这个宏不同之处在于，它将一个公共变量，而不是命令行选项，添加到`configure`关心的公共变量列表中。在这个例子中，我添加了两个公共变量，`FTKINC`和`FTKLIB`。它们将在`configure`的帮助文本中出现在“一些影响环境变量”部分下。*GNU Autoconf 手册*将这些变量称为*珍贵的*。我的所有`FLM_PROG_TRY_*`宏都在内部使用`AC_ARG_VAR`宏，使相关变量既是公共的，也是珍贵的。^(22)

**注意**

*从* ➋ *到* ➐ *的代码行可以在 GitHub 仓库的* xflaim/m4/flm_ftk_search.m4 *中找到*。*到第十五章结束时，本章中的列表和 GitHub 仓库中的文件之间的所有差异都已解决。*

从➌开始的大段代码实际上使用这些变量来设置构建系统中使用的其他变量。用户可以在环境中设置这些公共变量，或者可以通过这种方式在`configure`的命令行上指定它们：

```
$ ./configure FTKINC="$HOME/dev/ftk/include" ...
```

首先，我会检查`FTKINC`和`FTKLIB`变量是否都指定了，或者都没有指定。如果只指定了其中一个，我必须因为出错而失败。用户不能只告诉我在哪里找到*半*个工具包；我需要同时拥有头文件和库。^(23) 如果这两个变量都没有指定，我会在➍通过查找名为*ftk*的子目录来寻找它们。如果找到了，我将使用`AC_CONFIG_SUBDIRS`宏将该目录配置为一个由 Autoconf 处理的子项目。^(24) 我还会将这两个变量设置为指向 ftk 子项目中适当的相对位置。

如果没有找到*ftk*子目录，我会在父目录的➎查找。如果在那里找到了，我会相应地设置变量。这次，我不需要将找到的*ftk*目录配置为子项目，因为我假设 xflaim 项目本身是上层项目的一个子项目。如果我没有找到*ftk*，无论是作为子项目还是同级项目，我会在➏使用标准的`AC_CHECK_LIB`和`AC_CHECK_HEADERS`宏来检查用户的主机是否安装了工具包库。在这种情况下，我只需要将`-lflaimtk`添加到`LIBS`变量中。在这种情况下，头文件将位于标准位置：通常是*/usr(/local)/include*。`AC_CHECK_LIB`的可选第三个参数的默认功能会自动将库引用添加到`LIBS`变量中，但由于我覆盖了这个默认功能，我必须手动将工具包库引用添加到`LIBS`中。

如果找不到库，我会放弃并显示一个错误信息，提示 xflaim 无法在没有 FLAIM 工具包的情况下构建。然而，在通过所有这些检查后，如果`FTKLIB`变量不再为空，我会在➐使用`AC_SUBST`来发布`FTK_INCLUDE`和`FTK_LTLIB`变量，这些变量包含适用于预处理器和链接器命令行选项的`FTKINC`和`FTKLIB`的衍生值。

**注意**

*第十六章将➌到➑之间的大段代码转换成一个名为*`FLM_FTK_SEARCH`*的自定义 M4 宏。*

➑处的剩余代码以类似于我在 ftk 项目中处理 Doxygen 的方式，调用`AM_CONDITIONAL`来处理 Java、C#和 Doxygen。这些宏配置为生成警告信息，指出如果找不到 Java 或 C#工具，xflaim 项目的这些部分将不会被构建，但无论如何，我允许构建继续进行。

#### *创建 xflaim/src/Makefile.am 文件*

我跳过了*xflaim/Makefile.am*文件，因为它与*ftk/Makefile.am*几乎相同。相反，我们将继续讨论*xflaim/src/Makefile.am*，这是我按照与*ftk/src*版本相同的设计原则编写的。它看起来与 ftk 的对应文件非常相似，唯一的例外是：根据原始构建系统的 makefile，Java 本地接口（JNI）和 C#本地语言绑定源文件直接编译并链接到*xflaim*共享库中。

这并不是一种不常见的做法，而且非常有用，因为它避免了为这些语言专门构建额外的库对象。基本上，*xflaim*共享库导出了这些语言的本地接口，然后这些接口被相应的本地封装器使用。^(25)

我现在暂时忽略这些语言绑定，但稍后当我完成整个 xflaim 项目时，我会重新关注如何将它们正确地集成到库中。除去这个例外，清单 14-14 中展示的*Makefile.am*文件与其 ftk 对应文件几乎一模一样。

```
if HAVE_JAVA
  JAVADIR = java
  JNI_LIBADD = java/libxfjni.la
endif

if HAVE_CSHARP
  CSDIR = cs
  CSI_LIBADD = cs/libxfcsi.la
endif

SUBDIRS = $(JAVADIR) $(CSDIR)

pkgconfigdir = $(libdir)/pkgconfig
pkgconfig_DATA = libxflaim.pc

lib_LTLIBRARIES = libxflaim.la
include_HEADERS = xflaim.h

libxflaim_la_SOURCES = \
 btreeinfo.cpp \
  f_btpool.cpp \
  f_btpool.h \
  --snip--
  rfl.h \
  scache.cpp \
  translog.cpp

libxflaim_la_CPPFLAGS = $(FTK_INCLUDE)
libxflaim_la_LIBADD = $(JNI_LIBADD) $(CSI_LIBADD) $(FTK_LTLIB)
libxflaim_la_LDFLAGS = -version-info 3:2:0
```

*清单 14-14*：xflaim/src/Makefile.am：*xflaim 项目 src 目录的 Automake 输入文件*

我根据*configure.ac*中相应 Automake 条件语句定义了`SUBDIRS`变量的内容。当执行`make all`时，`SUBDIRS`变量会根据条件递归到*java*和*cs*子目录中。但当执行`make dist`时，一个隐藏的`DIST_SUBDIRS`变量（由 Automake 根据`SUBDIRS`变量的*所有可能内容*创建）引用了无论是条件性还是无条件附加到`SUBDIRS`中的所有目录。^(26)

**注意**

*库接口版本信息是从原始 makefile 中提取的。*

#### *转向 xflaim/util 目录*

xflaim 的*util*目录稍微复杂一些。根据原始 makefile，它生成了几个实用程序以及一个被这些工具使用的便捷库。

找出哪些源文件属于哪些工具，哪些根本没有被使用，稍微有些困难。*xflaim/util*目录中的几个文件并没有被任何工具使用。我们是否需要分发这些额外的源文件？我选择分发它们，因为它们已经被原始构建系统分发，并将它们添加到`EXTRA_DIST`列表中使得后来的人可以明显看出它们没有被使用。

清单 14-15 展示了*xflaim/util/Makefile.am*文件的一部分；缺失的部分是冗余的。

```
   EXTRA_DIST = dbdiff.cpp dbdiff.h domedit.cpp diffbackups.cpp xmlfiles

   XFLAIM_INCLUDE = -I$(top_srcdir)/src
   XFLAIM_LDADD = ../src/libxflaim.la

➊ AM_CPPFLAGS = $(XFLAIM_INCLUDE) $(FTK_INCLUDE)
   LDADD = libutil.la $(XFLAIM_LDADD)

 ## Utility Convenience Library

   noinst_LTLIBRARIES = libutil.la

   libutil_la_SOURCES = \
    flm_dlst.cpp \
    flm_dlst.h \
    flm_lutl.cpp \
    flm_lutl.h \
    sharutil.cpp \
    sharutil.h

   ## Utility Programs

   bin_PROGRAMS = xflmcheckdb xflmrebuild xflmview xflmdbshell

   xflmcheckdb_SOURCES = checkdb.cpp
   xflmrebuild_SOURCES = rebuild.cpp

   xflmview_SOURCES = \
    viewblk.cpp \
    view.cpp \
    --snip--
    viewmenu.cpp \
    viewsrch.cpp

   xflmdbshell_SOURCES = \
    domedit.h \
    fdomedt.cpp \
    fshell.cpp \
    fshell.h \
    xshell.cpp

   ## Check Programs

   check_PROGRAMS = \
    ut_basictest \
    ut_binarytest \
    --snip--
    ut_xpathtest \
    ut_xpathtest2

➋ check_DATA = copy-xml-files.stamp
   check_HEADERS = flmunittest.h

   ut_basictest_SOURCES = flmunittest.cpp basictestsrv.cpp
➌ --snip--
   ut_xpathtest2_SOURCES = flmunittest.cpp xpathtest2srv.cpp

   ## Unit Tests

   TESTS = \
    ut_basictest \
    --snip--
    ut_xpathtest2

   ## Miscellaneous rules required by Check Programs

➍ copy-xml-files.stamp:
           cp $(srcdir)/xmlfiles/*.xml .
           echo Timestamp > $@

➎ clean-local:
           rm -rf ix2.*
           rm -rf bld.*
           rm -rf tst.bak
           rm -f *.xml
           rm -f copy-xml-files.stamp
```

*清单 14-15*：xflaim/util/Makefile.am：*xflaim 项目的* util *目录 Automake 输入文件*

在这个例子中，您可以通过省略的部分看到，我省略了多个长文件列表和产品清单。这个 Makefile 构建了 22 个单元测试，但由于它们几乎完全相同，除了命名差异和构建源文件的不同，我只保留了其中两个的描述（见 ➌）。

我在 ➊ 处定义了文件全局的 `AM_CPPFLAGS` 和 `LDADD` 变量，用于将 `XFLAIM` 和 `FTK` 的头文件和库文件与这个 *Makefile.am* 文件中列出的每个项目关联起来。这样，我就不需要显式地将这些信息附加到每个产品上。

##### 传递性依赖

然而，请注意，`AM_CPPFLAGS` 变量同时使用了 `XFLAIM_INCLUDE` 和 `FTK_INCLUDE` 变量——xflaim 工具显然需要来自两组头文件的信息。那么，为什么 `LDADD` 变量没有引用 *ftk* 库呢？这是因为 Libtool 为您管理传递性依赖，并且它以非常便捷的方式做到这一点，因为某些系统没有本地的机制来管理传递性依赖。由于我通过 `XFLAIM_LDADD` 引用了 *libxflaim.la*，并且 *libxflaim.la* 将 *libflaimtk.la* 列为依赖项，Libtool 能够在工具程序的链接器命令行中为我提供传递性引用。

为了更清楚地了解这一点，请查看 *libxflaim.la* 的内容（在您的构建目录下的 *xflaim/src* 文件夹中——您需要先构建该项目；运行 `autoreconf -i; ./configure && make`）。您会在文件中间附近找到几行，内容看起来非常像 Listing 14-16 中的内容。

```
--snip--
# Libraries that this one depends upon.
dependency_libs=' .../flaim/build/ftk/src/libflaimtk.la -lrt -lncurses'
--snip--
```

*Listing 14-16：xflaim/src* 中的部分 *libxflaim.la* 文件内容，显示了依赖库。

这里列出了 *libflaimtk.la* 的路径信息，这样我们就不需要在 xflaim 工具的 `LDADD` 语句中显式指定它。^(27)

像 Libtool 一样，GNU 链接器和 Linux 加载器也能管理传递性依赖（TD）。这通过在适当的链接器命令行选项使用时，让 `ld` 将这些间接依赖合并到它生成的 ELF 二进制文件中来实现。Libtool 的机制依赖于对 *.la* 文件层级结构的递归搜索，而 Linux 的本地机制则是在构建时递归地搜索库层级，并将所有必需的库引用直接嵌入到构建的程序或库中。加载器随后在加载时看到并使用这些引用。使用这种本地 TD 管理的一个好处是，如果库在包的较新版本中更新，加载器将立即开始引用新库更新后的二级符号，而基于该库构建的项目将立即开始使用新版本的传递性依赖。

最近，一些发行版供应商决定在他们的平台上利用这个特性。问题在于，Libtool 的 TD 管理减少了使用`ld`（以及系统加载器）内部 TD 管理的优势——它在某种程度上妨碍了这一点。为了解决这个问题，这些供应商决定发布一个修改版的 Libtool，其中其 TD 管理功能被有效地禁用。结果是，你现在必须在链接器（`libtool`）命令行上明确指定所有直接和间接库，或者修改你的构建系统以使用非便携的本地 TD 管理链接器选项。

由于并非所有平台都支持本地 TD 管理，而 Libtool 的基于文本文件的方法完全是可移植的，我们通常会在没有本地 TD 管理系统的系统上依赖 Libtool 来正确处理间接依赖，尤其是在链接程序和库时。当你使用“被发行版破坏”的 Libtool 包来构建设计为利用 Libtool TD 管理功能的项目时，你的构建将在链接阶段失败，并出现“缺少 DSO”（动态共享对象）消息。^(28)

*configure.ac*中的`sed`命令会在*libtool*脚本中搜索`link_all_deplibs=no`文本，并将其替换为`link_all_deplibs=yes`。它出现了两次，`sed`命令会替换这两个实例。`AC_OUTPUT`执行`config.status`，它会在项目目录中生成*libtool*脚本，因此`sed`命令必须跟随`AC_OUTPUT`才能生效。

**注意**

*即使在没有出现问题的系统上使用这个*`sed`*命令也不会有任何不良影响——*`sed`* 只会在你的*libtool*脚本中找不到任何内容来替换。不过要注意的是，如果你的软件包被某个使用内部 TD 管理的 Linux 供应商选中进行分发，他们很可能会把这类命令“修补”掉。*

当然，另一种选择是完全放弃使用自动传递依赖管理，通过在每个程序或库的链接器命令行上明确指定你知道需要的所有链接依赖项来实现。实际上，Pkg-config 已经为你做了这件事，因此如果你能够依赖 pkg-config 来满足所有的库管理需求，那么你的项目就不会受到这个问题的影响。这可以通过在 flam、xflaim 和 flaimsql 项目中手动添加`$(FTK_LTLIB)`到`LDADD`变量中来实现，具体可以参见 Listing 14-15 中的➊。

可以尝试通过注释掉*configure.ac*中的`sed`命令来进行测试，然后重新构建项目。假设你在一个修改过 Libtool 的平台上构建项目，那么在 flam 和 xflaim 项目尝试仅通过*libflaim.la*和*libxflaim.la*链接它们的工具时，构建将会失败。为了让它重新工作，按照之前提到的方式更新`LDADD`变量。

##### 时间戳目标

在创建这个 Makefile 时，我遇到了一个没有预料到的小问题。至少有一个单元测试似乎要求在执行测试的目录中存在一些 XML 数据文件。测试失败了，当我深入调查时，我注意到它在尝试打开这些文件时失败了。稍微四处看看，我发现了*xflaim/util/xmldata*目录，其中包含了几十个 XML 文件。

我需要将这些文件复制到构建层次结构中的*xflaim/util*目录，然后才能运行单元测试。我知道以 `check` 为前缀的产品在执行 `TESTS` 之前会先构建，所以我想到我可以在 ➋ 处在 `check_DATA PLV` 中列出这些文件。`check_DATA` 变量引用一个名为*copy-xml-files.stamp*的文件，它是一个特殊类型的文件目标，称为*stamp*目标。它的目的是用一个单一的代表文件替代一组未指定的文件，或者一个非基于文件的操作。这个时间戳文件用于向构建系统指示所有 XML 数据文件已经被复制到*util*目录中。Automake 在它自己生成的规则中经常使用时间戳文件。

生成时间戳文件的规则 ➍ 也将 XML 数据文件复制到测试执行目录中。`echo` 语句仅仅创建一个名为*copy-xml-files.stamp*的文件，文件中包含一个单词：*Timestamp*。这个文件可以包含任何内容（甚至什么都不包含）。这里的重要点是文件存在并且有一个时间和日期与之相关联。`make` 工具利用这些信息来确定是否需要执行复制操作。在这种情况下，由于*copy-xml-files.stamp*没有依赖项，它的存在仅仅意味着 `make` 已经完成了操作。删除时间戳文件可以让 `make` 在下一次构建时执行复制操作。

这是一种介于真实基于文件的规则和伪目标之间的混合体。伪目标总是会被执行——它们不是实际的文件，因此 `make` 无法根据文件属性确定是否应该执行相关操作。基于文件的规则的时间戳可以与它们的依赖列表进行检查，以确定是否需要重新执行它们。像这样的时间戳规则只有在时间戳文件丢失时才会执行，因为没有依赖项可以与目标的时间和日期进行比较。^(29)

##### 清理你的房间

所有放置在构建目录中的文件都应该在用户输入`make clean`时被清理掉。由于我将 XML 数据文件放入了构建目录，因此我也需要清理它们。列在`DATA`变量中的文件不会自动清理，因为`DATA`文件不一定是生成的。有时，`DATA`主变量用于列出需要安装的静态项目文件。我“创建”了一些 XML 文件和一个标记文件，所以在`make clean`时需要删除它们。为此，我在➎处添加了`clean-local`目标，以及其相关的`rm`命令。

**注意**

*当删除从源树复制到构建树相应位置的文件时要小心——你可能会不小心删除源文件，尤其是当在源树内构建时。你可以在*`make`*命令中将*`$(srcdir)`*与“*`.`*”进行比较，看看用户是否在源树中构建。*

还有另一种方法可以确保在执行`clean`目标时，使用你自己的`make`规则创建的文件能够被清理掉。你可以定义`CLEANFILES`变量，包含一个以空格分隔的文件列表（或通配符规范），以供删除。我在这个例子中使用了`clean-local`目标，因为`CLEANFILES`变量有一个限制：它不能删除目录，只能删除文件。每个删除通配符文件规范的`rm`命令都涉及到至少一个目录。我将在第十五章中展示如何正确使用`CLEANFILES`。

无论你的单元测试清理得多么干净，你仍然可能需要编写`clean`规则来尝试清理中间的测试文件。这样，你的 makefile 将清理被中断的测试和调试运行留下的残余文件。^(30) 请记住，用户可能在源目录中构建。尽量让你的通配符尽可能具体，以免不小心删除源文件。

我在这里使用了 Automake 支持的`clean-local`目标，作为扩展`clean`目标的一种方式。如果存在，`clean-local`目标会作为依赖项（因此会在`clean`目标之前执行）。列表 14-17 展示了来自 Automake 生成的*Makefile.in*模板的相应代码，您可以看到这个基础设施是如何连接起来的。关键部分已被突出显示。

```
   --snip--
   clean: clean-am
➊ clean-am: clean-binPROGRAMS clean-checkPROGRAMS \
     clean-generic clean-libtool clean-local \
     clean-noinstLTLIBRARIES mostlyclean-am
   --snip--
➋ .PHONY: ... clean-local...
   --snip--
   clean-local:
           rm -rf ix2.*
           rm -rf bld.*
           rm -rf tst.bak
           rm -f *.xml
           rm -f copy-xml-files.stamp
   --snip--
```

*列表 14-17*：xflaim/util/Makefile.in：*从* xflaim/util/Makefile.am *生成的清理规则*

Automake 注意到我在*Makefile.am*中有一个名为`clean-local`的目标，因此它在➊处将`clean-local`添加到`clean-am`的依赖列表中，然后在➋处将其添加到`.PHONY`变量中。如果我没有编写`clean-local`目标，这些引用就不会出现在生成的*Makefile*中。

### 总结

好吧，这些就是基础知识。如果你跟随并理解了本章中的内容，那么你应该能够将几乎任何项目转换为使用基于 Autotools 的构建系统。有关此处涉及主题的更多细节，请参考 Autotools 手册。通常，知道一个概念的名称，以便你可以轻松地在手册或在线搜索中找到它，价值非常大。

在第十五章中，我将介绍将此项目转换的更奇特方面，包括构建 Java 和 C#代码的细节，添加特定编译器的优化标志和命令行选项，甚至使用用户定义的`make`目标在你的*Makefile.am*文件中构建 RPM 包。
