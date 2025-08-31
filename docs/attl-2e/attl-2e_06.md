## 第六章：使用 Automake 自动生成 Makefile

*如果你理解了，事情就是它们本来的样子；如果你不理解，事情就是它们本来的样子。

—禅宗格言*

![Image](img/common.jpg)

在 Autoconf 开始走向成功后不久，David MacKenzie 开始着手开发一个用于自动生成 GNU 项目 makefile 的新工具：Automake。在*GNU 编码标准（GCS）*的早期开发中，MacKenzie 意识到，因为*GCS*对项目产品的构建、测试和安装的要求非常具体，很多 GNU 项目的 makefile 实际上是模板内容。Automake 利用这一点，使得维护者的工作更加轻松，并且让用户的体验更加一致。

MacKenzie 在 Automake 上的工作持续了近一年，直到 1994 年 11 月左右结束。一年后，即 1995 年 11 月，Tom Tromey（来自 Red Hat 和 Cygnus）接管了 Automake 项目，并在其发展中发挥了重要作用。尽管 MacKenzie 最初用 Bourne shell 脚本编写了 Automake 的版本，Tromey 完全用 Perl 重写了这个工具，并在接下来的五年中继续维护和增强 Automake。

到 2000 年底，Alexandre Duret-Lutz 几乎接管了 Automake 项目的维护工作。他作为项目负责人一直持续到大约 2007 年中期，此时 Ralf Wildenhues^(1)接手了项目，偶尔有 Akim Demaille 和 Jim Meyering 的参与。从 2012 年到 2017 年初，Stefano Lattarini 在为 Google 瑞士分部工作期间负责 Automake 的维护。现任维护者是 Mathieu Lirzin，他是法国波尔多大学计算机科学硕士生。

我所见到的大多数关于 Autotools 的抱怨最终都与 Automake 有关。原因很简单：Automake 为构建系统提供了最高级别的抽象，并且对使用它的项目强加了一个相当严格的结构。Automake 的语法简洁——实际上，它是简练的，几乎到了极致。一个 Automake 语句代表了*大量*功能。但是，一旦你理解了它，你就能在短时间内（也就是几分钟，而不是几个小时或几天）建立起一个相对完整、复杂且功能正确的构建系统。

在本章中，我将为你提供一些关于 Automake 内部工作原理的见解。通过这些见解，你将不仅对 Automake 能为你做什么感到熟悉，而且会开始在其自动化不足的领域进行扩展。

### 开始正式工作

让我们面对现实吧——正确编写 Makefile 通常很难。正如人们所说，魔鬼藏在细节中。考虑在我们继续改进 Jupiter 项目的构建系统时，对项目目录结构中的文件进行以下更改。让我们从清理工作区开始。你可以使用 `make distclean` 来完成这项工作，或者如果你是从 GitHub 仓库工作区构建的，也可以使用 `git clean` 命令的某种形式：^(2)

Git 标签 6.0

```
   $ git clean -xfd
   --snip--
➊ $ rm bootstrap.sh Makefile.in src/Makefile.in
➋ $ echo "SUBDIRS = src" > Makefile.am
➌ $ echo "bin_PROGRAMS = jupiter
   > jupiter_SOURCES = main.c" > src/Makefile.am
➍ $ touch NEWS README AUTHORS ChangeLog
   $ ls -1
   AUTHORS
   ChangeLog
   configure.ac
   Makefile.am
   NEWS
   README
   src
   $
```

➊ 处的 `rm` 命令删除了我们手动编写的 *Makefile.in* 模板和我们为确保所有支持脚本和文件被复制到项目根目录中而编写的 `bootstrap.sh` 脚本。由于我们正在将 Jupiter 升级为正式的 Automake，因此不再需要这个脚本。（为了简洁起见，我在 ➋ 和 ➌ 使用了 `echo` 语句来写入新的 *Makefile.am* 文件；如果你愿意，可以使用文本编辑器。）

**注意**

*在 ➌ 处的行末有一个硬回车符。Shell 会继续接受输入，直到引号关闭为止。*

我在 ➍ 使用了 `touch` 命令来创建项目根目录中新的、空的 *NEWS*、*README*、*AUTHORS* 和 *ChangeLog* 文件。（*INSTALL* 和 *COPYING* 文件是通过 `autoreconf -i` 添加的。）这些文件是 *GCS* 对所有 GNU 项目所要求的。尽管它们对于非 GNU 项目并非必需，但它们已经成为开源世界中的一种惯例；用户已经习惯了这些文件的存在。^(3)

**注意**

*GCS* 覆盖了这些文件的格式和内容。第 6.7 和 6.8 节分别讲解了 *NEWS* 和 *ChangeLog* 文件，第 7.3 节则涵盖了 *README*、*INSTALL* 和 *COPYING* 文件。*AUTHORS* 文件是一个列出需要给与归属的人员（姓名和可选的电子邮件地址）清单。^(4)

维护 *ChangeLog* 文件可能有点痛苦——特别是因为你已经在为你的仓库提交添加提交信息时做过一次了。为了简化这个过程，考虑使用一个 shell 脚本在你做新版本发布之前，将仓库日志抓取到 *ChangeLog* 中。网络上有现成的脚本可供使用；例如，*gnulib*（见 第十三章）提供了 `gitlog-to-changelog` 脚本，可以用来在发布之前将 git 仓库的日志信息导入 *ChangeLog* 中。

#### *在 configure.ac 中启用 Automake*

为了在构建系统中启用 Automake，我向 *configure.ac* 添加了一行代码：在 `AC_INIT` 和 `AC_CONFIG_SRCDIR` 之间调用 `AM_INIT_AUTOMAKE`，如 列表 6-1 所示。

```
#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.
AC_PREREQ([2.69])
AC_INIT([Jupiter], [1.0], [jupiter-bugs@example.org])
AM_INIT_AUTOMAKE
AC_CONFIG_SRCDIR([src/main.c])
--snip--
```

*列表 6-1：向 configure.ac 添加 Automake 功能*

如果你的项目已经使用 Autoconf 进行了配置，这将是启用 Automake 的*唯一*必要行，前提是配置文件`configure.ac`有效。`AM_INIT_AUTOMAKE`宏接受一个可选参数：一个由空格分隔的选项标签列表，可以将这些标签传递给此宏，以修改 Automake 的通用行为。有关每个选项的详细描述，请参阅*GNU Automake 手册*的第十七章。^(5) 但是，我将在这里指出一些最有用的选项。

gnits, gnu, foreign

这些选项设置 Automake 的严格性检查。默认值为`gnu`。`gnits`选项使 Automake 变得比原来更加挑剔，而`foreign`选项则稍微放宽一些——使用`foreign`时，你不需要像 GNU 项目那样强制要求*INSTALL*、*README*和*ChangeLog*文件。

check-news

`check-news`选项会导致如果项目的当前版本（来自*configure.ac*）没有出现在*NEWS*文件的前几行中，`make dist`命令失败。

dist-bzip2, dist-lzip, dist-xz, dist-shar, dist-zip, dist-tarZ

你可以使用`dist-*`选项来更改默认的分发包类型。默认情况下，`make dist`会生成一个*.tar.gz*文件，但开发者常常希望分发例如*.tar.xz*格式的包。这些选项使得更改变得非常简单。（即使没有`dist-xz`选项，你也可以通过使用`make dist-xz`来覆盖当前的默认设置，但如果你总是希望构建*.xz*包，使用该选项会更简单。）

readme-alpha

`readme-alpha`选项会在项目的 Alpha 版本发布期间临时更改构建和分发过程的行为。使用此选项会自动分发项目根目录中的名为*README-alpha*的文件。使用此选项还会更改项目的版本控制方案。

-W category, --warnings=category

`-W` *`category`* 和 `--warnings=`*`category`* 选项表示项目希望使用 Automake 并启用各种警告类别。可以使用多个这样的选项，每个选项可以有不同的类别标签。请参考*GNU Automake 手册*，查找有效类别的列表。

parallel-tests

`parallel-tests`功能允许在执行`check`目标时并行执行检查，以便在多处理器机器上利用并行执行。

subdir-objects

`subdir-objects`选项在你打算引用当前目录以外的目录中的源代码时是必需的。使用此选项会导致 Automake 生成`make`命令，使得目标文件和中间文件与源文件生成在同一目录下。有关此选项的更多信息，请参阅“非递归 Automake”部分，见第 175 页。

version

*`version`* 选项实际上是一个占位符，用于表示此项目接受的最低版本的 Automake 版本号。例如，如果传入 `1.11` 作为选项标记，如果 Automake 的版本低于 1.11，则在处理 *configure.ac* 时会失败。如果你打算使用只有较新版本的 Automake 才支持的功能，这会非常有用。

现在，已经有了新的 *Makefile.am* 文件，并且在 *configure.ac* 中启用了 Automake，接下来我们可以运行 `autoreconf` 并使用 `-i` 选项，以便为我们的项目添加 Automake 可能需要的任何新工具文件：

```
$ autoreconf -i
configure.ac:11: installing './compile'
configure.ac:6: installing './install-sh'
configure.ac:6: installing './missing'
Makefile.am: installing './INSTALL'
Makefile.am: installing './COPYING' using GNU General Public License v3 file
Makefile.am:     Consider adding the COPYING file to the version control system
Makefile.am:     for your code, to avoid questions about which license your
project uses src/Makefile.am: installing './depcomp'
$
$ ls -1p
aclocal.m4
AUTHORS
autom4te.cache/
ChangeLog
compile
config.h.in
configure
configure.ac
COPYING
depcomp
INSTALL
install-sh
Makefile.am
Makefile.in
missing
NEWS
README
src/
$
```

将 `AM_INIT_AUTOMAKE` 宏添加到 *configure.ac* 中，会导致 `autoreconf -i` 现在执行 `automake -i`，这将包括一些额外的工具文件：*aclocal.m4*、*install-sh*、*compile*、*missing* 和 *depcomp*。此外，Automake 现在会从 *Makefile.am* 生成 *Makefile.in*。

我在第二章中提到过 *aclocal.m4*，在第四章中提到过 `install-sh`。`missing` 脚本是一个小的辅助工具脚本，当命令行上指定的工具不可用时，它会打印一个格式化的错误信息。其实没有必要了解更多细节；如果你感兴趣，可以在项目目录中执行 `./missing --help`。

我们稍后会谈到 `depcomp` 脚本，但在此我想提一下 `compile` 脚本的目的。这个脚本是一些旧编译器的包装器，它们不理解同时使用 `-c` 和 `-o` 命令行选项。当你使用特定产品的标志（我们稍后会讨论）时，Automake 必须生成代码，这些代码可能会多次编译源文件，每次编译使用不同的标志。因此，它必须为每组标志命名不同的目标文件。`compile` 脚本简化了这个过程。

Automake 还会添加默认的 *INSTALL* 和 *COPYING* 文本文件，这些文件包含与 GNU 项目相关的模板文本。你可以根据需要修改这些文件以适应你的项目。我发现默认的 *INSTALL* 文件内容对于与 Autotools 构建项目相关的通用指令非常有用，但在将其提交到我的代码库之前，我喜欢在文件顶部添加一些项目特定的信息。Automake 的 `-i` 选项在项目中已经包含这些文本文件时不会覆盖它们，因此，一旦通过 `autoreconf -i` 添加了这些文件，你可以根据需要修改这些默认文件。

*COPYING* 文件包含 GPL 许可证的文本，可能适用于或不适用于你的项目。如果你的项目是根据 GPL 许可发布的，只需保留该文本不变。如果你是根据其他许可证（如 BSD、MIT 或 Apache Commons 许可证）发布的，请将默认文本替换为适合该许可证的文本。^(6)

**注意**

*你只需要在新检出的工作区或新创建的项目中使用一次*`-i`*选项。添加缺失的工具文件后，除非你向*configure.ac*中添加某些宏，否则你可以在以后调用*`autoreconf`*时省略*`-i`*选项，添加的宏可能会导致使用*`-i`*选项以添加更多缺失的文件。我们将在后面的章节中看到这类情况。*

上述命令创建了一个基于 Automake 的构建系统，包含了我们在原始*Makefile.in*模板中编写的所有内容（除了稍后会提到的`check`功能之外），但是这个系统在功能上更加完整，且符合*GCS*的规范。查看生成的*Makefile.in*模板，我们可以看到 Automake 为我们做了大量工作。生成的顶层*Makefile.in*模板几乎有 24KB，而原来的手工编写的 makefile 只有几百字节。

一个 Automake 构建系统支持以下重要的`make`目标（从 Automake 生成的*Makefile*派生）：

| `all` | `check` | `clean` | `ctags` |
| --- | --- | --- | --- |
| `dist` | `dist-bzip2` | `dist-gzip` | `dist-lzip` |
| `dist-shar` | `dist-tarZ` | `dist-xz` | `dist-zip` |
| `distcheck` | `distclean` | `distdir` | `dvi` |
| `html` | `info` | `install` | `install-data` |
| `install-dvi` | `install-exec` | `install-html` | `install-info` |
| `install-pdf` | `install-ps` | `install-strip` | `installcheck` |
| `installdirs` | `maintainer-clean` | `mostlyclean` | `pdf` |
| `ps` | `tags` | `ininstall` |  |

如你所见，这远远超出了我们在手动编写的*Makefile.in*模板中能提供的功能。Automake 将这些基础功能写入每个使用它的项目中。

#### *一个隐藏的好处：自动依赖关系跟踪*

在《依赖规则》一节中（见第 46 页），我们讨论了`make`依赖规则。这些规则是我们在 makefile 中定义的，以便`make`能够意识到 C 语言源文件与包含的头文件之间的隐含关系。Automake 花费了大量精力来确保你不需要为它能够理解的语言（如 C、C++和 Fortran）编写这些依赖规则。这是对于包含多个源文件的项目来说非常重要的一个特性。

手动为数十或数百个源文件编写依赖规则既繁琐又容易出错。事实上，这已经成为一个问题，以至于编译器作者经常提供一种机制，使编译器能够根据其对源文件和语言的内部知识自动编写这些规则。GNU 编译器等支持一系列`-M`选项（`-M`、`-MM`、`-MF`、`-MG`等）。这些选项告诉编译器为指定的源文件生成一个`make`依赖规则。（其中一些选项可以在正常的编译命令行中使用，因此可以在源文件被编译时生成依赖规则。）

这些选项中最简单的是基本的 `-M` 选项，它使编译器为指定的源文件在 `stdout` 上生成一个依赖关系规则，然后终止。这个规则可以被捕获到一个文件中，随后由 makefile 包含，从而将该规则中的依赖关系信息纳入到 `make` 构建的有向图中。

但是，当系统中的本地编译器不提供依赖关系生成选项，或者它们与编译过程无法配合工作时，会发生什么情况呢？在这种情况下，Automake 提供了一个名为*depcomp*的包装脚本，该脚本会执行两次编译：第一次生成依赖关系信息，第二次编译源文件。当编译器缺少生成*任何*依赖关系信息的选项时，可以使用另一个工具递归地确定哪些头文件会影响给定的源文件。在没有这些选项可用的系统上，自动依赖关系生成将失败。

**注意**

*关于依赖关系生成编译器选项的更详细描述，请参阅第 529 页的“项目 10：使用生成的源代码”。关于 Automake 依赖关系管理的更多内容，请参阅* GNU Automake 手册的相关章节。

现在是时候咬紧牙关，尝试一下了。和上一章的构建系统一样，运行 `autoreconf`（由于我们之前运行过 `autoreconf -i`，所以这是可选的，但没有害处），接着运行 `./configure` 和 `make`。

```
$ autoreconf
$ ./configure
--snip--
$ make
make  all-recursive
make[1]: Entering directory '/.../jupiter'
Making all in src
make[2]: Entering directory '/.../jupiter/src'
gcc -DHAVE_CONFIG_H -I. -I..         -g -O2 -MT main.o -MD -MP -MF .deps/main.Tpo -c -o main.o main.c
mv -f .deps/main.Tpo .deps/main.Po
gcc  -g -O2   -o jupiter main.o  -lpthread
make[2]: Leaving directory '/.../jupiter/src'
make[2]: Entering directory '/.../jupiter'
make[2]: Leaving directory '/.../jupiter'
make[1]: Leaving directory '/.../jupiter'
$
```

如果不尝试我们已经熟悉的其他 `make` 目标，你是无法真正体会到 Automake 所做的工作。自己尝试 `install`、`dist` 和 `distcheck` 目标，以确认你在删除手写的 *Makefile.in* 模板之后，仍然拥有之前的所有功能。

**注意**

*`check`* 目标目前作为一个无操作的目标存在，但在我们能将测试加回来之前，我们需要更详细地研究 Automake 构造。当我们深入到这一部分时，你会发现它甚至比我们最初编写的代码更简单。

### Makefile.am 文件实际上包含了什么？

在第四章中，我们讨论了 Autoconf 如何将一个包含 M4 宏的 shell 脚本作为输入，并生成同样的 shell 脚本，其中这些宏得到了完全展开。同样，Automake 将一个包含 Automake 命令的 makefile 作为输入。正如 Autoconf 的输入文件仅仅是增强版的 shell 脚本一样，Automake 的*Makefile.am* 文件也不过是标准的 makefile，只是额外包含了 Automake 特有的语法。

Autoconf 和 Automake 之间的一个显著区别是，Autoconf 输出的唯一文本是输入文件中现有的 shell 脚本，以及嵌入的 M4 宏展开后产生的任何附加 shell 脚本。另一方面，Automake 假设所有的 makefile 都应该包含一个最小的基础设施，用于支持*GCS*，除了你指定的任何目标和变量之外。

为了说明这一点，在 Jupiter 项目的根目录中创建一个 *temp* 目录，并向其中添加一个空的 *Makefile.am* 文件。接下来，用文本编辑器将这个新的 *Makefile.am* 添加到项目的 *configure.ac* 文件中，并从顶层的 *Makefile.am* 文件中引用它，如下所示：

```
   $ mkdir temp
   $ touch temp/Makefile.am
➊ $ echo "SUBDIRS = src temp" > Makefile.am
   $ vi configure.ac
   --snip--
   AC_CONFIG_FILES([Makefile
                    src/Makefile
                 ➋ temp/Makefile])
   --snip--
   $ autoreconf
   $ ./configure
   --snip--
   $ ls -1sh temp
   total 24K
➌ 12K Makefile
   0 Makefile.am
➍ 12K Makefile.in
   $
```

我在 ➊ 处使用了一个 `echo` 语句，重写了一个新的顶层 *Makefile.am* 文件，其中 `SUBDIRS` 同时引用 *src* 和 *temp*。我用文本编辑器将 *temp/Makefile* 添加到 Autoconf 将从模板生成的 makefile 列表中（➋）。如您所见，每个 makefile 中生成了一些 Automake 认为不可或缺的支持代码。即使是一个空的 *Makefile.am* 文件，也会生成一个 12KB 的 *Makefile.in* 模板（➍），然后 `configure` 从中生成一个类似大小的 *Makefile*（➌）。^(7)

由于 `make` 工具使用一套相当严格的规则来处理 makefile，Automake 对你额外的 `make` 代码有一些灵活的处理。以下是一些具体内容：

+   在 *Makefile.am* 文件中定义的 `make` 变量被放置在生成的 *Makefile.in* 模板的顶部，紧跟在任何 Automake 生成的变量定义之后。

+   在 *Makefile.am* 文件中指定的 `make` 规则被放置在生成的 *Makefile.in* 模板的末尾，紧跟在任何 Automake 生成的规则之后。

+   大多数由 `config.status` 替换的 Autoconf 变量被转换为 `make` 变量，并初始化为那些替换变量。

`make` 工具不关心规则之间的位置关系，因为它会在处理任何规则之前，将每条规则读取到一个内部数据库中。变量也类似，只要它们在使用之前被定义。为了避免任何变量绑定问题，Automake 会将所有变量按定义顺序放在输出文件的顶部。

### 分析我们的新构建系统

现在让我们来看一下我们在这两个简单的 *Makefile.am* 文件中放了什么，从顶层的 *Makefile.am* 文件开始（如列表 6-2 所示）。

```
SUBDIRS = src
```

*列表 6-2:* Makefile.am: *顶层* Makefile.am *文件仅包含一个子目录引用。*

这一行文本告诉 Automake 我们项目的几个信息：

+   一个或多个子目录包含要处理的 makefile，除了这个文件之外。^(8)

+   此空格分隔的目录列表应按指定的顺序处理。

+   此列表中的目录应为所有主要目标递归处理。

+   除非另有说明，否则此列表中的目录应视为项目分发的一部分。

和大多数 Automake 构造一样，`SUBDIRS`只是一个`make`变量，对于 Automake 有特殊的含义。`SUBDIRS`变量可用于处理具有任意复杂目录结构的*Makefile.am*文件，目录列表可以包含任何相对目录引用（不仅仅是直接的子目录）。可以说，`SUBDIRS`就像是在使用递归构建系统时，将 makefile 连接在项目目录层次结构中的“粘合剂”。

Automake 生成递归的`make`规则，这些规则在处理`SUBDIRS`列表中指定的目录后，隐式地处理当前目录，但有时需要在其他某些或所有目录之前构建当前目录。可以通过在`SUBDIRS`列表中的任何位置引用当前目录（使用点符号）来更改默认的构建顺序。例如，要在*src*目录之前构建顶层目录，可以按如下方式修改列表 6-2 中的`SUBDIRS`变量：

```
SUBDIRS = . src
```

现在让我们转到*src*目录中的*Makefile.am*文件，如列表 6-3 所示。

```
bin_PROGRAMS = jupiter
jupiter_SOURCES = main.c
```

*列表 6-3：* src/Makefile.am：*这个* Makefile.am *文件的初始版本只包含两行*

第一行是*产品列表变量*的规范，第二行是*产品源变量*的规范。

#### *产品列表变量*

产品在*Makefile.am*文件中通过*产品列表变量（PLV）*进行指定，像`SUBDIRS`一样，PLV 是`make`变量的一类，对 Automake 具有特殊意义。以下模板显示了 PLV 的常见格式：

```
[modifier-list]prefix_PRIMARY = product1 product2 ... productN
```

列表 6-3 中第一行的 PLV 名称由两部分组成：*前缀*（*bin*）和*主元素*（`PROGRAMS`），由下划线（`_`）分隔。该变量的值是由此*Makefile.am*文件生成的产品的一个以空格分隔的列表。

##### 安装位置前缀

列表 6-3 中显示的产品列表变量的*bin*部分是一个*安装位置前缀*的示例。*GCS*定义了许多常见的安装位置，大多数在表 3-1 中列出，位于第 65 页。然而，任何以`dir`结尾且值为文件系统位置的`make`变量，都是有效的安装位置变量，并且可以作为 Automake PLV 中的前缀使用。

在 PLV 前缀中引用安装位置变量时，应省略变量名称中的`dir`部分。例如，在列表 6-3 中，当`$(bindir)` `make`变量用作安装位置前缀时，只需称其为`bin`。

Automake 还识别四个以特殊 `pkg` 前缀开头的安装位置变量：`pkglibdir`、`pkgincludedir`、`pkgdatadir` 和 `pkglibexecdir`。这些 `pkg` 版本的标准 `libdir`、`includedir`、`datadir` 和 `libexecdir` 变量表示列出的产品应安装在这些位置的子目录中，子目录名称与软件包相同。例如，在 Jupiter 项目中，带有 `lib` 前缀的 PLV 中列出的产品将安装到 `$(libdir)` 中，而带有 `pkglib` 前缀的 PLV 中列出的产品将安装到 `$(libdir)`*/jupiter* 中。

由于 Automake 从所有以 `dir` 结尾的 `make` 变量中推导出有效的安装位置和前缀列表，因此您可以提供自己的 PLV 前缀，指向自定义的安装位置。要将一组 XML 文件安装到系统数据目录中的 *xml* 目录，您可以在 清单 6-4 中的 *Makefile.am* 文件中使用代码。

```
xmldir = $(datadir)/xml
xml_DATA = file1.xml file2.xml file3.xml ...
```

*清单 6-4：指定自定义安装目录*

安装位置变量将包含由 Automake 生成的 makefile 或您在 *Makefile.am* 文件中定义的默认值，但用户总是可以在 `configure` 或 `make` 命令行中覆盖这些默认值。如果您不希望在特定构建过程中安装某些产品，请在命令行中的安装位置变量中指定空值；Automake 生成的规则将确保不安装目标目录中的产品。例如，要仅为一个软件包安装文档和共享数据文件，您可以输入 `make bindir='' libdir='' install`。^(9)

##### 与安装无关的前缀

某些前缀与安装位置无关。例如，`noinst`、`check` 和 `EXTRA` 分别用于表示不安装的产品、仅用于测试的产品或可选构建的产品。以下是关于这三个前缀的更多信息：

noinst

表示列出的产品应该构建，但不需要安装。例如，一个所谓的静态 *便利库* 可能会作为中间产品构建，然后在构建过程的其他阶段中用于构建最终产品。`noinst` 前缀告诉 Automake 不应安装该产品，仅构建静态库。（毕竟，构建一个不会安装的共享库是没有意义的。）

检查

表示仅为测试目的构建的产品，因此不需要安装。在 PLV 中以 `check` 为前缀列出的产品仅在用户输入 `make check` 时才会构建。

EXTRA

用于列出有条件构建的程序。Automake 要求所有源文件都必须在*Makefile.am*文件中静态指定，而不是在构建过程中计算或推导，以便它能够生成一个适用于任何可能命令行的*Makefile.in*模板。然而，项目维护者可以选择允许某些产品根据传递给`configure`脚本的配置选项有条件地构建。如果产品在由`configure`脚本生成的变量中列出，它们也应该在*Makefile.am*文件中的 PLV 中列出，并以`EXTRA`为前缀。这个概念在清单 6-5 和 6-6 中有所说明。

```
AC_INIT(...)
--snip--
optional_programs=
AC_SUBST([optional_programs])
--snip--
if test "x$(build_opt_prog)" = xyes; then
➊ optional_programs=$(optional_programs) optprog
fi
--snip--
```

*清单 6-5：在*configure.ac 中定义的有条件构建的程序，存储在一个 Shell 变量中

```
➋ EXTRA_PROGRAMS = optprog
➌ bin_PROGRAMS = myprog $(optional_programs)
```

*清单 6-6：在* Makefile.am 中使用`EXTRA`前缀有条件地定义产品

在清单 6-5 中的➊处，`optprog`被附加到一个名为`optional_programs`的 Autoconf 替代变量中。在清单 6-6 中的➋处，`EXTRA_PROGRAMS`变量列出了`optprog`，作为一个可能构建或不构建的产品，取决于最终用户的配置选择，这些配置决定了➌处的`$(optional_programs)`是否为空或包含`optprog`。

尽管在*configure.ac*和*Makefile.am*中都指定`optprog`看起来可能是冗余的，但 Automake 需要在`EXTRA_PROGRAMS`中提供该信息，因为它无法尝试解释在*configure.ac*中定义的`$(optional_programs)`的可能值。因此，在这个示例中将`optprog`添加到`EXTRA_PROGRAMS`中，告诉 Automake 生成规则来构建它，即使`$(optional_programs)`在某次构建中不包含`optprog`。

##### 主要产品

*主要产品*就像产品类别，它们代表可能由构建系统生成的产品类型。一个主要产品定义了构建、测试、安装和执行特定类别产品所需的一组步骤。例如，程序和库使用不同的编译器和链接器命令来构建，Java 类需要虚拟机来执行，而 Python 程序需要解释器。某些产品类别，如脚本、数据和头文件，没有构建、测试或执行语义——只有安装语义。

支持的主要产品列表定义了可以由 Automake 构建系统自动构建的产品类别集合。Automake 构建系统仍然可以构建其他产品类别，但维护者必须在项目的*Makefile.am*文件中显式地定义`make`规则。

彻底理解 Automake 的主要产品是正确使用 Automake 的关键。目前支持的主要产品的完整列表如下。

PROGRAMS

当在 PLV 中使用`PROGRAMS`主要产品时，Automake 会生成使用编译器和链接器来构建列出产品的二进制可执行程序的`make`规则。

LIBRARIES/LTLIBRARIES

使用 `LIBRARIES` 主项会导致 Automake 生成规则，使用系统编译器和库管理器构建静态库（库文件）。`LTLIBRARIES` 主项做同样的事情，但生成的规则还会构建 Libtool 共享库，并通过 `libtool` 脚本执行这些工具（以及链接器）。 （我将在第七章和第八章中详细讨论 Libtool 包。) Automake 限制了 `LIBRARIES` 和 `LTLIBRARIES` 主项的安装位置：它们只能安装在 `$(libdir)` 和 `$(pkglibdir)` 中。

LISP

`LISP` 主项主要用于管理 Emacs Lisp 程序的构建。因此，它期望引用一个 *.el* 文件列表。您可以在 Automake 手册第 10.1 节中找到有关使用该主项的详细信息。

PYTHON

Python 是一种解释型语言；`python` 解释器逐行将 Python 脚本转换为 Python 字节码，并在转换的同时执行它，因此（像 shell 脚本一样）Python 源文件可以直接执行。使用 `PYTHON` 主项告诉 Automake 生成规则，将 Python 源文件 (*.py*) 预编译为标准 (*.pyc*) 和优化 (*.pyo*) 字节编译版本，使用 `py-compile` 工具进行编译。由于 Python 源代码通常是解释执行的，这种编译发生在安装时，而不是在构建时。

JAVA

Java 是一个虚拟机平台；使用 `JAVA` 主项会告诉 Automake 生成规则，使用 `javac` 编译器将 Java 源文件 (*.java*) 转换为 Java 类文件 (*.class*)。虽然这个过程是正确的，但并不完整。Java 程序（有实际意义的程序）通常包含多个类文件，通常以 *.jar* 或 *.war* 文件打包，这些文件可能还包含多个附带的文本文件。`JAVA` 主项是有用的，但仅此而已。（我将在《使用 Autotools 构建 Java 源代码》一章中详细讨论如何使用——以及扩展——`JAVA` 主项，详情请见 第 408 页。）

脚本

在这个上下文中，*脚本* 指任何解释型文本文件——无论是 shell、Perl、Python、Tcl/Tk、JavaScript、Ruby、PHP、Icon、Rexx 还是其他任何类型的文件。Automake 允许为 `SCRIPTS` 主项设置受限的安装位置，包括 `$(bindir)`、`$(sbindir)`、`$(libexecdir)` 和 `$(pkgdatadir)`。虽然 Automake 不会生成构建脚本的规则，但它也不假设脚本是项目中的静态文件。脚本通常由 *Makefile.am* 文件中的手写规则生成，有时通过使用 `sed` 或 `awk` 工具处理输入文件。因此，脚本不会自动分发。如果您项目中有一个静态脚本，并希望 Automake 将其添加到您的分发归档中，则应像“PLV 和 PSV 修饰符”中所讨论的那样，在 `SCRIPTS` 主项前添加 `dist` 修饰符，详情请参见 第 161 页。

数据

任意数据文件可以通过 PLV 中的 `DATA` 主项进行安装。Automake 允许 `DATA` 主项的限制安装位置，包括 `$(datadir)`、`$(sysconfdir)`、`$(sharedstatedir)`、`$(localstatedir)` 和 `$(pkgdatadir)`。数据文件不会自动分发，因此如果你的项目包含静态数据文件，请在 `DATA` 主项上使用 `dist` 修饰符，如在 第 161 页的《PLV 和 PSV 修饰符》中所讨论的那样。

HEADERS

头文件是一种源文件形式。如果不是因为某些头文件已经被安装，它们本可以直接列在产品源代码中。包含已安装库产品公共接口的头文件会被安装到 `$(includedir)` 或由 `$(pkgincludedir)` 定义的包特定子目录中，因此此类已安装头文件的最常见 PLV 是 `include_HEADERS` 和 `pkginclude_HEADERS` 变量。像其他源文件一样，头文件会自动分发。如果你有一个生成的头文件，请使用 `nodist` 修饰符与 `HEADERS` 主项一起使用，具体如在 第 161 页的《PLV 和 PSV 修饰符》中所讨论的那样。

MANS

*Man 页面* 是包含 `troff` 标记的 UTF-8 文本文件，用户查看时由 `man` 渲染。Man 页面可以通过 `man_MANS` 或 `man`*`N`*`_MANS` 产品列表变量安装，其中 *`N`* 代表介于 0 到 9 之间的单数字节或字母 *l*（用于数学库主题）或 *n*（用于 Tcl/Tk 主题）。`man_MANS` PLV 中的文件应具有表示其所属 man 部分的数字扩展名，从而指示其目标目录。`man`*`N`*`_MANS` PLV 中的文件可以使用数字扩展名或 *.man* 扩展名命名，并将在 `make install` 安装时重命名为相关的数字扩展名。项目的 man 页面默认不进行分发，因为 man 页面通常是生成的，因此你应该使用 `dist` 修饰符，如在 第 161 页的《PLV 和 PSV 修饰符》中所讨论的那样。

TEXINFOS

在 Linux 或 Unix 文档中，Texinfo^(10)是 GNU 项目的首选格式。`makeinfo`工具接受 Texinfo 源文件（*.texinfo*、*.txi*或*.texi*），并渲染包含 UTF-8 文本的 info 文件（*.info*），这些文本用 Texinfo 标记注释，`info`工具将其渲染为格式化文本供用户使用。与 Texinfo 源一起使用的最常见的产品列表变量是`info_TEXINFOS`。使用此 PLV 会导致 Automake 生成构建*.info*、*.dvi*、*.ps*和*.html*文档文件的规则。然而，只有*.info*文件会在`make all`时构建，并通过`make install`安装。为了构建和安装其他类型的文件，必须在`make`命令行中明确指定`dvi`、`ps`、`pdf`、`html`、`install-dvi`、`install-ps`、`install-pdf`和`install-html`目标。由于许多 Linux 发行版默认未安装`makeinfo`工具，生成的*.info*文件会自动添加到分发归档中，以便最终用户不必去寻找`makeinfo`。

#### *产品源变量*

清单 6-3 中的第二行是一个 Automake *产品源变量*（*PSV*）的示例。PSV 符合以下模板：

```
[modifier-list]product_SOURCES = file1 file2 ... fileN
```

和 PLV 一样，PSV 由多个部分组成：产品名称（此处为`jupiter`）和`SOURCES`标签。PSV 的值是一个由空格分隔的源文件列表，这些文件用于构建*`product`*。在清单 6-3 的第二行中，PSV 的值是用于构建`jupiter`程序的源文件列表。最终，Automake 将这些文件添加到生成的*Makefile.in*模板中的各种`make`规则依赖列表和命令中。

只有`make`变量中允许的字符（字母、数字、@符号和下划线）才允许出现在 PSV 的`product`标签中。因此，Automake 会对 PLV 中列出的产品名称进行转换，以呈现关联 PSV 中使用的*`product`*标签。Automake 将非法字符转换为下划线，如清单 6-7 所示。

```
➊ lib_LIBRARIES = libc++.a
➋ libc___a_SOURCES = ...
```

*清单 6-7：非法的`make`变量字符在`product`标签中转换为下划线。*

在这里，Automake 将 PLV 中的*libc++.a*（在➊处）转换为 PSV 的`product`标签`libc___a`（即三个下划线），以在*Makefile.am*文件中找到关联的 PSV（在➋处）。你必须了解这些转换规则，以便能够编写与产品匹配的 PSV。

#### *PLV 和 PSV 修饰符*

之前定义的 PLV 和 PSV 模板中的`modifier-list`部分包含一组可选的修饰符。以下类似 BNF 的规则定义了这些模板中`modifier-list`元素的格式：

```
modifier-list = modifier_[modifier-list]
```

修饰符改变了它们前置的变量的正常行为。当前定义的前缀修饰符集包括`dist`、`nodist`、`nobase`和`notrans`。

`dist` 修饰符表示一组应该分发的文件（即应该包含在通过执行 `make dist` 时生成的分发包中）。例如，假设某些产品的源文件应该被分发，而某些不应该被分发，清单 6-8 中展示的变量可能在产品的 *Makefile.am* 文件中定义。

```
dist_myprog_SOURCES = file1.c file2.c
nodist_myprog_SOURCES = file3.c file4.c
```

*清单 6-8：在 `Makefile.am` 文件中使用 `dist` 和 `nodist` 修饰符*

Automake 通常会从 `HEADERS` PLV 中的头文件列表中去除相对路径信息。`nobase` 修饰符用于抑制从子目录中通过 *Makefile.am* 文件获取的安装头文件路径信息的去除。例如，查看 清单 6-9 中的 PLV 定义。

```
nobase_pkginclude_HEADERS = mylib.h sys/constants.h
```

*清单 6-9：在 `Makefile.am` 文件中使用 `nobase` PLV 修饰符*

在这一行中，我们可以看到 *mylib.h* 与 *Makefile.am* 位于同一目录下，而 *constants.h* 位于名为 *sys* 的子目录中。通常，两个文件都会通过 `pkginclude` 安装位置前缀安装到 `$(pkgincludedir)` 中。然而，由于我们使用了 `nobase` 修饰符，Automake 将保留第二个文件路径中的 *sys/* 部分进行安装，*constants.h* 将被安装到 `$(pkgincludedir)`*/sys* 中。当你希望安装（目标）目录结构与项目（源代码）目录结构相同，并且文件在安装过程中被复制时，这非常有用。

`notrans` 修饰符可以用于 man 页面 PLV，针对那些在安装过程中不应被转换的 man 页面（通常，Automake 会生成规则，将 man 页面扩展名从 *.man* 重命名为 *.N*，其中 *N* 为 *0*、*1*、...、*9*、*l*、*n*，当它们被安装时）。

你还可以使用 `EXTRA` 前缀作为修饰符。当与产品源变量（例如 `jupiter_SOURCES`）一起使用时，`EXTRA` 指定与 `jupiter` 产品直接关联的额外源文件，如 清单 6-10 中所示。

```
EXTRA_jupiter_SOURCES = possibly.c
```

*清单 6-10：与产品 `SOURCES` 变量一起使用 `EXTRA` 前缀*

在这里，*possibly.c* 是否被编译取决于在 *configure.ac* 中定义的一些条件。

### 单元测试：支持 `make check`

在 第三章，我们向 *src/Makefile* 中添加了代码，执行 `jupiter` 程序并检查当用户执行 `check` 目标时是否有正确的输出字符串。现在我们有足够的信息将我们的 `check` 目标测试重新添加到新的 Automake 构建系统中。我已经在 清单 6-11 中复制了 `check` 目标代码，以供接下来的讨论参考。

```
--snip--
check: all
        ./jupiter | grep "Hello from .*jupiter!"
        @echo "*** ALL TESTS PASSED ***"
--snip--
```

*清单 6-11：来自 第三章 的 `check` 目标*

幸运的是，Automake 对单元测试有很好的支持。为了将我们简单的 `grep` 测试重新添加到新的 Automake 生成的构建系统中，我们可以在 *src/Makefile.am* 的底部添加几行，如 Listing 6-12 所示。

Git 标签 6.1

```
   bin_PROGRAMS = jupiter
   jupiter_SOURCES = main.c

➊ check_SCRIPTS = greptest.sh
➋ TESTS = $(check_SCRIPTS)

   greptest.sh:

 echo './jupiter | grep "Hello from .*jupiter!"' > greptest.sh
         chmod +x greptest.sh

➌ CLEANFILES = greptest.sh
```

*Listing 6-12:* src/Makefile.am: *支持 `check` 目标所需的额外代码*

➊ 处的 `check_SCRIPTS` 行是一个 PLV，它指向一个在构建时生成的脚本。由于前缀是 `check`，我们知道此行列出的脚本只有在用户输入 `make check` 时才会被构建。然而，我们必须提供一个 `make` 规则来构建脚本，并在执行 `clean` 目标时删除该文件。我们在 ➌ 处使用 `CLEANFILES` 变量，来扩展 Automake 在执行 `make clean` 时删除的文件列表。

➋ 处的 `TESTS` 行是 Listing 6-12 中的重要部分，因为它指示在用户执行 `check` 目标时会执行哪些目标。（由于 `check_SCRIPTS` 变量包含了这些目标的完整列表，我在这里简单引用了它，作为实际的 `make` 变量。）请注意，在这个特定的例子中，`check_SCRIPTS` 是多余的，因为 Automake 会生成规则，确保在执行测试之前，所有在 `TESTS` 中列出的程序都已经构建完毕。然而，当需要在执行 `TESTS` 列表中的程序之前构建额外的帮助脚本或程序时，`check_*` PLV 就变得重要了。

这里可能不是很明显，但由于我们添加了第一个测试，在运行 `make check` 之前，我们需要重新执行 `autoreconf -i` 以添加一个新的实用程序脚本：*test-driver*。你可以在 Automake 文档中找到明确说明必须这样做的地方，但更简单的方法是让构建系统告诉你在缺少某些内容时，必须执行 `autoreconf (-i)`。为了让你感受一下这个过程，让我们先不运行 `autoreconf` 来试试：

```
$ make check
Making check in src
make[1]: Entering directory '/.../jupiter/src'
 cd .. && /bin/bash /.../jupiter/missing automake-1.15 --gnu src/Makefile
parallel-tests: error: required file './test-driver' not found
parallel-tests:   'automake --add-missing' can install 'test-driver'
Makefile:255: recipe for target 'Makefile.in' failed
make[1]: *** [Makefile.in] Error 1
make[1]: Leaving directory '/.../jupiter/src'
Makefile:352: recipe for target 'check-recursive' failed
make: *** [check-recursive] Error 1
$
```

现在让我们先运行 `autoreconf -i`：

```
$ autoreconf -i
parallel-tests: installing './test-driver'
$
$ make check
/bin/bash ./config.status --recheck
running CONFIG_SHELL=/bin/bash /bin/bash ./configure --no-create --no-recursion
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /bin/mkdir -p
--snip--
Making check in src
make[1]: Entering directory '/.../jupiter/src'
make  greptest.sh
make[2]: Entering directory '/.../jupiter/src'
echo './jupiter | grep "Hello from .*jupiter!"' > greptest.sh
chmod +x greptest.sh
make[2]: Leaving directory '/.../jupiter/src'
make  check-TESTS
make[2]: Entering directory '/.../jupiter/src'
make[3]: Entering directory '/.../jupiter/src'
PASS: greptest.sh
============================================================================
Testsuite summary for Jupiter 1.0
============================================================================
# TOTAL: 1
# PASS:  1
# SKIP:  0
# XFAIL: 0
# FAIL:  0
# XPASS: 0
# ERROR: 0
============================================================================
make[3]: Leaving directory '/.../jupiter/src'
make[2]: Leaving directory '/.../jupiter/src'
make[1]: Leaving directory '/.../jupiter/src'
make[1]: Entering directory '/.../jupiter'
make[1]: Leaving directory '/.../jupiter'
$
```

运行 `autoreconf -i` 后（并且注意到 `test-driver` 已经安装到我们的项目中），我们可以看到 `make check` 现在成功运行了。

请注意，在运行 `autoreconf -i` 后，我不需要手动调用 `configure`。构建系统通常足够智能，知道何时应该为你重新执行 `configure`。

### 使用便捷库减少复杂性

Jupiter 作为开源软件项目来说相当简单，因此为了突出 Automake 的一些关键特性，我们将稍微扩展它。我们将首先添加一个便捷库，然后修改 `jupiter` 来使用这个库。

*便利库*是一个静态库，仅在包含它的项目中使用。这种临时库通常用于当一个项目中的多个二进制文件需要集成相同的源代码时。我将把*main.c*中的代码移动到一个库源文件，并从`jupiter`的`main`例程中调用这个库中的函数。首先，从项目的顶级目录执行以下命令：

Git 标签 6.2

```
$ mkdir common
$ touch common/jupcommon.h
$ cp src/main.c common/print.c
$ touch common/Makefile.am
$
```

现在将列表 6-13 和 6-14 中的高亮文本分别添加到新建的*common*目录中的*.h*和*.c*文件中。

```
int print_routine(const char * name);
```

*列表 6-13:* common/jupcommon.h: *这个文件的初始版本*

```
#include "config.h"

#include "jupcommon.h"

#include <stdio.h>
#include <stdlib.h>

#if HAVE_PTHREAD_H
# include <pthread.h>
#endif

static void * print_it(void * data)
{
    printf("Hello from %s!\n", (const char *)data);
    return 0;
}

int print_routine(const char * name)
{
#if ASYNC_EXEC
    pthread_t tid;
    pthread_create(&tid, 0, print_it, (void*)name);
    pthread_join(tid, 0);
#else
    print_it(name);
#endif
    return 0;
}
```

*列表 6-14:* common/print.c: *这个文件的初始版本*

如你所见，*print.c*仅仅是*main.c*的一个副本，经过了几个小的修改（在列表 6-14 中高亮显示）。首先，我将`main`重命名为`print_routine`，然后在包含*config.h*之后添加了对*jupcommon.h*头文件的包含。这个头文件将`print_routine`的原型提供给*src/main.c*，在那里它从`main`中被调用。

接下来，我们按列表 6-15 所示修改*src/main.c*，然后将列表 6-16 中的文本添加到*common/Makefile.am*中。

```
#include "config.h"

#include "jupcommon.h"

int main(int argc, char * argv[])
{
    return print_routine(argv[0]);
}
```

*列表 6-15:* src/main.c: *修改以让`main`调用新库*

**注意**

*将* config.h *包含在* src/main.c *的顶部可能显得有些奇怪，因为在那个源文件中似乎没有什么地方使用它。*GCS*建议遵循标准做法，将*config.h*包含在所有源文件的顶部，所有其他包含语句之前，以防其他包含的头文件中的某些内容使用了*config.h*中的定义。*我建议严格遵循这个做法*。

```
noinst_LIBRARIES = libjupcommon.a
libjupcommon_a_SOURCES = jupcommon.h print.c
```

*列表 6-16:* common/Makefile.am: *这个文件的初始版本*

让我们来看看这个新的*Makefile.am*文件。第一行指示了这个文件应该构建和安装哪些产品。`noinst`前缀表示这个库仅仅是为了方便在*common*目录中使用源代码。

我们正在创建一个名为*libjupcommon.a*的静态库，也叫做*归档*。归档类似于*.tar*文件，只包含目标文件（*.o*）。它们不能像共享库那样被执行或加载到进程地址空间中，但可以像目标文件一样被添加到链接命令行中。链接器足够聪明，能够识别出这些归档只是目标文件的集合。

**注意**

*链接器会将命令行中显式指定的每个目标文件添加到二进制产物中，但它们只从库中提取实际在链接的代码中引用的目标文件。因此，如果你链接到一个包含 97 个目标文件的静态库，但你只直接或间接调用其中两个文件的函数，那么只有这两个目标文件会被添加到你的程序中。相反，链接到 97 个原始目标文件则会将所有 97 个文件添加到你的程序中，无论你是否使用其中的任何功能*。

列表 6-16 中的第二行是一个产品源变量，包含与该库关联的源文件列表。^(11)

#### *产品选项变量*

现在我们需要向*src/Makefile.am*中添加一些额外的信息，以便生成的*Makefile*能够找到我们添加到*common*目录中的新库和头文件。让我们向现有的*Makefile.am*文件中再添加两行，如列表 6-17 所示。

```
   bin_PROGRAMS = jupiter
   jupiter_SOURCES = main.c
➊ jupiter_CPPFLAGS = -I$(top_srcdir)/common
➋ jupiter_LDADD = ../common/libjupcommon.a
   --snip--
```

*列表 6-17：* src/Makefile.am：*向* Makefile.am *文件中添加编译器和链接器指令*

像`jupiter_SOURCES`变量一样，这两个新变量是从程序名称派生的。这些*产品选项变量（POVs）*用于指定构建工具所使用的与产品相关的选项，这些工具从源代码构建产品。

`jupiter_CPPFLAGS`变量在➊处将产品特定的 C 预处理器标志添加到`jupiter`程序的所有源文件的编译器命令行中。`-I$(top_srcdir)/common`指令告诉 C 预处理器将`$(top_srcdir)`*/common*添加到其查找头文件引用的位置列表中。^(12)

`jupiter_LDADD`变量在➋处将库添加到`jupiter`程序的链接器命令行中。文件路径*../common/libjupcommon.a*仅仅是将一个目标文件添加到链接器命令行，以便该库中的代码能够成为最终程序的一部分。

**注意**

*你还可以使用*`$(top_builddir)`*代替../来引用此路径中*common*目录的位置。使用*`$(top_builddir)`*的附加优势在于，它使得将这个*Makefile.am*文件移动到另一个位置时无需修改它变得更加简单*。

向*`program`*`_LDADD`或*`library`*`_LIBADD`变量中添加一个库仅在该库作为你自己包的一部分构建时才需要。如果你正在将程序与已经安装在用户系统上的库进行链接，则在*configure.ac*中调用`AC_CHECK_LIB`或`AC_SEARCH_LIBS`将使生成的`configure`脚本通过`LIBS`变量将适当的引用添加到链接器命令行中。

Automake 支持的 POV 集合主要来源于 Table 3-2 中列出的标准用户变量的子集，见 第 71 页。你可以在 *GNU Autoconf Manual* 中找到程序和库选项变量的完整列表，但这里列出了一些重要的变量。

product_CPPFLAGS

使用 *`product`*`_CPPFLAGS` 将标志传递给 C 或 C++ 预处理器，添加到编译器命令行中。

product_CFLAGS

使用 *`product`*`_CFLAGS` 将 C 编译器标志传递到编译器命令行中。

product_CXXFLAGS

使用 *`product`*`_CXXFLAGS` 将 C++ 编译器标志传递到编译器命令行中。

product_LDFLAGS

使用 *`product`*`_LDFLAGS` 将全局和与顺序无关的共享库与程序链接器配置标志和选项传递给链接器，包括 `-static`、`-version-info`、`-release` 等。

program_LDADD

使用 *`program`*`_LDADD` 将 Libtool 对象（*.lo*）或库（*.la*）或非 Libtool 对象（*.o*）或档案（*.a*）添加到链接命令行中，进行程序链接。^(13)

library_LIBADD

使用 *`library`*`_LIBADD` 将非 Libtool 链接器对象和档案添加到非 Libtool 档案的 `ar` 工具命令行中。`ar` 工具将把命令行中提到的档案并入产品档案，因此你可以使用此变量将多个档案合并成一个。

ltlibrary_LIBADD

使用 *`ltlibrary`*`_LIBADD` 将 Libtool 链接器对象（*.lo*）和 Libtool 静态或共享库（*.la*）添加到 Libtool 静态或共享库中。

你可以使用此列表中的最后三个选项变量，将依赖顺序的静态和共享库引用传递给链接器。你也可以使用这些选项变量来传递 `-L` 和 `-l` 选项。以下是可接受的格式：`-L`*`libpath`*，`-l`*`libname`*，`[`*`relpath`*`/]`*`archive`*`.a`，`[`*`relpath`*`/]`*`objfile`*`.$(OBJEXT)`，`[`*`relpath`*`/]`*`ltobject`*`.lo`，以及 `[`*`relpath`*`/]`*`ltarchive`*`.la`。（请注意，术语 *`relpath`* 表示项目中的相对路径，可以是相对目录引用，使用点，或 `$(top_builddir)`。）

#### *每个 Makefile 的选项变量*

你通常会看到 Automake 变量 `AM_CPPFLAGS` 和 `AM_LDFLAGS` 被用在 *Makefile.am* 文件中。这些每个 Makefile 的形式用于当维护者想要将相同的标志集应用于 *Makefile.am* 文件中所有指定的产品时。^(14) 例如，如果你需要为 *Makefile.am* 文件中的所有产品设置一组预处理器标志，然后为特定产品（`prog1`）添加额外的标志，你可以使用 Listing 6-18 中展示的语句。

```
   AM_CFLAGS = ... some flags ...
   --snip--
➊ prog1_CFLAGS = $(AM_CFLAGS) ... more flags ...
   --snip--
```

*Listing 6-18: 同时使用每个产品和每个文件的标志*

每个产品变量的存在会覆盖 Automake 对每个 makefile 变量的使用，因此您需要在每个产品变量中引用每个 makefile 变量，以便让 makefile 变量影响该产品，如清单 6-18 中的 ➊ 所示。为了允许每个产品变量覆盖它们对应的 makefile 变量，最好首先引用 makefile 变量，然后再添加任何特定于产品的选项。

**注意**

*用户变量，例如*`CFLAGS`*，是为最终用户保留的，配置脚本或 makefile 不应设置这些变量。Automake 会始终将它们附加到适当的工具命令行，从而允许用户覆盖 makefile 中指定的选项*。

### 构建新库

接下来，我们需要编辑顶级 *Makefile.am* 文件中的 `SUBDIRS` 变量，以包括我们刚刚添加的新的 *common* 目录。我们还需要将 *common* 目录中生成的新 makefile 添加到 *configure.ac* 中 `AC_CONFIG_FILES` 宏调用生成的文件列表中。这些更改如清单 6-19 和清单 6-20 所示。

```
SUBDIRS = common src
```

*清单 6-19:* Makefile.am: *将公共目录添加到 `SUBDIRS` 变量*

```
--snip--
AC_CONFIG_FILES([Makefile
                 common/Makefile
                 src/Makefile])
--snip--
```

*清单 6-20:* configure.ac: *将* common/Makefile *添加到 `AC_CONFIG_FILES` 宏*

这是我们到目前为止所做的最大一组更改，但我们正在重新组织整个应用程序，因此这是合理的。让我们尝试一下更新后的构建系统。将 `-i` 选项添加到 `autoreconf` 命令行中，以便在这些增强之后安装可能需要的任何额外缺失文件。经过这么多更改后，我喜欢从一个干净的环境开始，因此可以先执行 `make distclean`，或者如果你是从 Git 仓库的工作区运行，则使用某种形式的 `git clean` 命令。

```
   $ make distclean
   --snip--
   $ autoreconf -i
   configure.ac:11: installing './compile'
   configure.ac:6: installing './install-sh'
   configure.ac:6: installing './missing'
   Makefile.am: installing './INSTALL'
   Makefile.am: installing './COPYING' using GNU General Public License v3 file
   Makefile.am:     Consider adding the COPYING file to the version control
   system
   Makefile.am:     for your code, to avoid questions about which license your
   project uses
➊ common/Makefile.am:1: error: library used but 'RANLIB' is undefined
   common/Makefile.am:1:  The usual way to define 'RANLIB' is to add 'AC_PROG_
   RANLIB'
   common/Makefile.am:1:  to 'configure.ac' and run 'autoconf' again.
   common/Makefile.am: installing './depcomp'
   parallel-tests: installing './test-driver'
   autoreconf: automake failed with exit status: 1
   $
```

看来我们还没有完全完成。由于我们向构建系统中添加了一种新的实体——静态库，`automake`（通过 `autoreconf`）在 ➊ 告诉我们，我们需要向 *configure.ac* 文件中添加一个新的宏 `AC_PROG_RANLIB`。^(15)

如清单 6-21 所示，将此宏添加到 *configure.ac* 文件中。

```
--snip--
# Checks for programs.
AC_PROG_CC
AC_PROG_INSTALL
AC_PROG_RANLIB
--snip--
```

*清单 6-21:* configure.ac: *添加 `AC_PROG_RANLIB`*

最后，再次输入 `autoreconf -i`。添加 `-i` 确保如果我们在 *configure.ac* 中添加的新功能需要安装任何额外的文件，`autoreconf` 会执行安装。

```
$ autoreconf -i
$
```

没有更多的抱怨；一切顺利。

### 什么应该包含在发布包中？

Automake 通常会自动确定在使用`make dist`创建的分发包中应该包含哪些文件，因为它非常清楚每个文件在构建过程中的作用。为此，Automake 需要知道每个用于构建产品的源文件以及每个安装的文件和产品。这意味着，当然，所有文件必须在某个时刻通过一个或多个 PLV 和 PSV 变量来指定。^(16)

Automake 的 `EXTRA_DIST` 变量包含一个以空格分隔的文件和目录列表，这些文件和目录应当在执行 `dist` 目标时添加到分发包中。例如：

```
EXTRA_DIST = windows
```

你可以使用 `EXTRA_DIST` 变量将一个源代码目录添加到分发包中，而 Automake 并不会自动添加它——例如，一个特定于 Windows 的目录。

**注意**

*在这种情况下*，windows *是一个目录，而不是文件。Automake 会自动递归地将该目录中的每个文件添加到分发包中；这可能包括一些你其实并不想要的文件，比如隐藏的* .svn *或* .CVS *状态目录。请参阅 第 389 页上的“Automake -hook 和 -local 规则”，了解如何解决这个问题。*

关于实用工具脚本

Autotools 已经在我们的项目目录结构根目录下添加了几个文件：`compile`、`depcomp`、`install-sh` 和 `missing`。因为 `configure` 或生成的 *Makefile* 会在构建过程中不同的阶段执行这些脚本，最终用户将需要它们；然而，我们只能从 Autotools 获取这些脚本，我们不希望用户必须安装 Autotools。因此，这些脚本会自动添加到分发包中。

那么，你是否应该将它们提交到源代码仓库中呢？这个问题的答案是有争议的，但通常我推荐你不要这样做。任何需要创建分发包的维护者应该已经安装了 Autotools，并且应该从一个仓库工作区中进行工作。因此，这些维护者也会运行 `autoreconf -i`（可能与 `--force` 选项*）一起使用，以确保他们拥有最新的 Autotools 提供的实用工具脚本。如果你将它们提交到仓库中，这只会增加它们随着时间推移变得过时的可能性。它还会导致你的仓库修订历史中出现不必要的波动，因为贡献者在使用不同版本的 Autotools 生成的文件之间反复切换。

我也将这种观点扩展到`configure`脚本。有些人认为将工具和`configure`脚本检查到项目代码库中是有益的，因为这可以确保如果有人从工作区检出了项目，他们可以从工作区构建项目，而无需安装 Autotools。然而，我个人的观点是，开发者和维护者应该预期安装这些工具。偶尔，最终用户可能需要从工作区构建项目，但这应该是例外情况，而不是典型的情况，在这些特殊情况下，用户应愿意承担维护者的角色和要求。

* 小心使用`--force`选项；它也会覆盖文本文件，如*INSTALL*，这些文件可能已根据项目修改，且与 Autotools 随附的默认文本文件不同。

### 维护者模式

有时，分发源文件的时间戳会比用户系统时钟的当前时间更晚。无论原因如何，这种不一致性会使`make`感到困惑，导致它认为每个源文件都过时了，需要重新构建。因此，它会重新执行 Autotools，以尝试更新`configure`和*Makefile.in*模板。但是作为维护者，我们并不真正期望我们的用户安装 Autotools——或者至少没有安装我们系统中最新版本的 Autotools。

这就是 Automake 的*维护者模式*的作用。默认情况下，Automake 会在 makefile 中添加规则，重新生成模板文件、配置脚本和从维护者源文件（如*Makefile.am*和*configure.ac*）以及 Lex 和 Yacc 输入文件生成的源文件。然而，我们可以在*configure.ac*中使用 Automake 的`AM_MAINTAINER_MODE`宏来禁用这些维护者级别的`make`规则的默认生成。

对于那些希望这些规则在构建系统发生变化后保持其构建系统适当更新的维护者，`AM_MAINTAINER_MODE`宏提供了一个`configure`脚本命令行选项（`--enable-maintainer-mode`），它告诉`configure`生成包含必要规则和命令以执行 Autotools 的*Makefile.in*模板。

维护者必须意识到在他们的项目中使用`AM_MAINTAINER_MODE`。在运行`configure`时，他们需要使用这个命令行选项，以生成完整的构建系统，这样当源代码被修改时，能够正确地重建由 Autotools 生成的文件。

**注意**

*我还建议在项目的*INSTALL*或*README*文件中提及使用维护者模式，以便最终用户在修改 Autotools 源文件时不会感到意外，且这些修改没有效果。*

虽然 Automake 的维护者模式有其优势，但你应该知道，也有各种反对使用它的观点。大多数观点集中在 `make` 规则不应被故意限制的想法上，因为这样做会生成一个在某些情况下总是失败的构建系统。不过，我要声明的是，后来的 Autotools 版本在告知你缺少所需工具时做得要好得多。事实上，这正是 `missing` 脚本的作用。大多数工具调用都被封装在 `missing` 脚本中，脚本会相当清晰地告诉你缺少什么，以及如何安装它。

使用这个宏时的另一个重要考虑因素是，你现在已经将测试矩阵中的行数翻倍了，因为每个构建选项都有两种模式——一种假设 Autotools 已经安装，另一种假设没有安装。如果你决定使用该宏默认禁用维护者模式以供最终用户使用，请牢记这些要点。

### 打破噪音

基于 Autotools 的构建系统产生的噪音量一直是 Automake 邮件列表上最具争议的话题之一。一方喜欢安静的构建，只显示重要的信息，如警告和错误。另一方则认为有价值的信息通常嵌入在所谓的“噪音”中，因此所有信息都很重要，应该显示出来。偶尔，新的 Autotools 开发者会发帖询问如何减少 `make` 显示的信息量。这几乎总会引发一场激烈的辩论，持续好几天，几乎上百封邮件。老手们只是笑笑，常开玩笑说“有人又把开关打开了”。

事情的真相是，双方都有其合理的观点。GNU 项目本身就是关于选项的，因此 Automake 的维护者们增加了一个功能，允许你可选地将静默规则提供给用户。**静默规则**在 Automake 的 makefile 中并不是真正的静默，它们只是比传统的 Automake 生成的规则稍微安静一些。

静默规则不会显示整个编译器或链接器的命令行，而是显示一行简短的信息，指示正在处理该工具的工具和文件名。`make` 生成的输出仍然会显示，用户可以知道当前正在处理哪个目录和目标。这是启用了静默规则后的 Jupiter 构建输出（首先执行 `make clean` 确保有东西被构建）：

```
$ make clean
--snip--
$ configure --enable-silent-rules
--snip--
$ make
make  all-recursive
make[1]: Entering directory '/.../jupiter'
Making all in common
make[2]: Entering directory '/.../jupiter/common'
  CC       print.o
  AR       libjupcommon.a
ar: `u' modifier ignored since `D' is the default (see `U')
make[2]: Leaving directory '/.../jupiter/common'
Making all in src
make[2]: Entering directory '/.../jupiter/src'
  CC       jupiter-main.o
  CCLD     jupiter
make[2]: Leaving directory '/.../jupiter/src'
make[2]: Entering directory '/.../jupiter'
make[2]: Leaving directory '/.../jupiter'
make[1]: Leaving directory '/.../jupiter'
$
```

如你所见，使用静默规则对 Jupiter 并没有太大影响——Jupiter 的构建系统花费大量时间在目录之间切换，实际构建的时间非常短。然而，在包含数百个源文件的项目中，你会看到一长串 `CC` *`filename`*`.o` 行，有时还会显示 `make` 切换目录或链接器正在构建产品的信息——编译器警告往往会引起注意。例如，输出中的 `ar` 警告如果没有静默规则的话，可能会被忽视。^(17)

静默规则默认是禁用的。要在 Automake 生成的 *Makefile.am* 模板中默认启用静默规则，可以在 *configure.ac* 中调用 `AM_SILENT_RULES` 宏，并传递 `yes` 参数。

无论如何，用户始终可以通过在 `configure` 命令行中使用 `--enable-silent-rules` 或 `--disable-silent-rules` 来设置构建的默认详细程度。然后，构建将根据配置的默认值以及用户是否在 `make` 命令行中指定 `V=0` 或 `V=1` 来决定是“静默”构建还是正常构建。

**注意**

*`configure`* 选项并不是必须的——静默规则的实际调用最终是由生成的 makefile 中的 *`V`* 变量控制的。*configure* 选项仅仅设置 *`V`* 的默认值。

对于较小的项目，我发现 Automake 的静默规则比起在 `make` 命令行中简单地将 `stdout` 重定向到 */dev/null* 更不实用，可以按以下方式操作：

```
$ make >/dev/null
ar: `u' modifier ignored since `D' is the default (see `U')
$
```

正如本示例所示，警告和错误仍然会显示在 `stderr` 上，通常会提供足够的信息，帮助你确定问题所在（尽管在此例中并未显示）。在这种情况下，无警告构建才是真正的静默构建。你应当定期使用这种技巧来清理源代码中的编译器警告。静默规则非常有用，因为警告在构建输出中很突出。

### 非递归 Automake

现在我们已经将手写的 *Makefile.in* 模板转换为 Automake 的 *Makefile.am* 文件，让我们看看如何将这个递归构建系统转换为非递归构建系统。在前面的章节中，我们看到使用 `make` 的 `include` 指令有助于将 makefile 分割为各个子目录负责的部分。然而，在 Automake 中，将所有内容放入顶层的 *Makefile.am* 文件更加简单，因为内容非常简短，我们可以一眼看出整个构建系统。如果需要进一步划分责任，只需简单的注释即可。

关键点是，和我们之前提到的一样，参考内容时要假设 `make` 是从顶层目录运行的（再说一次，它确实是）。

清单 6-22 包含了顶层 *Makefile.am* 文件的全部内容——这是我们在此转换中唯一使用的 makefile。

Git 标签 6.3

```
noinst_LIBRARIES = common/libjupcommon.a
common_libjupcommon_a_SOURCES = common/jupcommon.h common/print.c

bin_PROGRAMS = src/jupiter
src_jupiter_SOURCES = src/main.c
src_jupiter_CPPFLAGS = -I$(top_srcdir)/common
src_jupiter_LDADD = common/libjupcommon.a

check_SCRIPTS = src/greptest.sh
TESTS = $(check_SCRIPTS)

src/greptest.sh:
        echo './src/jupiter | grep "Hello from .*jupiter!"' > src/greptest.sh
        chmod +x src/greptest.sh

CLEANFILES = src/greptest.sh
```

*清单 6-22:* Makefile.am: *Jupiter 的非递归 Automake 实现*

如你所见，我已经将`SUBDIRS`变量从顶级*Makefile.am*文件中替换为每个目录中*Makefile.am*文件的完整内容。这些目录是由该变量引用的。我接着为每个输入对象和产品引用添加了适当的相对路径信息，以便源文件可以从顶级目录访问，而这些文件实际上位于各自的子目录中，并且产品也能最终被放到正确的位置——与其源输入文件一起（或者至少在源树外构建时，放到正确的对应目录）。我已在顶级文件中粘贴了每个子目录*Makefile.am*文件的更改部分。

请注意，`common_`或`src_`被加到了产品源变量的前面，因为这些前缀现在确实是产品名称的一部分。最终，这些名称用于创建`make`目标，这些目标的定义不仅依赖于名称，还依赖于它们的位置。通常，位置是当前目录，因此目录部分通常会被省略。对于我们的非递归构建，产品现在会生成到除当前目录以外的位置，因此必须明确指出这些位置。和产品名称中的其他特殊字符一样，目录分隔的斜杠在 PSV 中会变成下划线。

我们还需要添加一个 Automake 选项，并从*configure.ac*中移除多余的*Makefile*引用，如清单 6-23 所示。

```
#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.69])
AC_INIT([Jupiter], [1.0], [jupiter-bugs@example.org])
AM_INIT_AUTOMAKE([subdir-objects])
AC_CONFIG_SRCDIR([src/main.c])
AC_CONFIG_HEADERS([config.h])
--snip--
# Checks for library functions.

AC_CONFIG_FILES([Makefile])
AC_OUTPUT

cat << EOF
--snip--
```

*清单 6-23:* configure.ac: *移除非递归构建中的额外 Makefile 引用*

Automake 的`subdir-objects`选项是必要的，它告诉 Automake 你打算访问来自其他目录的源文件，而这些源文件并非位于当前目录。它还需要声明你希望目标文件和其他中间产品生成到与源文件相同的目录中（或者生成到正确的外部构建对应目录）。这个选项不仅对于非递归构建是必须的，还适用于任何需要在源文件所在目录外构建一个或多个源文件的情况。如果省略了这个选项，构建通常仍然会工作，但你会看到两个效果：`autoreconf`（或`automake`）会生成警告，提示你应该使用这个选项；而目标文件会被错误地放在不正确的目录下。如果你恰好在不同目录中有多个同名的源文件，那么第二个目标文件会覆盖第一个，这通常会导致链接错误，因为链接器找不到第一个被覆盖的目标中的符号。

最后，我们可以简单地删除*common*和*src*目录中的*Makefile.am*文件：

```
$ rm common/Makefile.am src/Makefile.am
$
```

### 总结

在本章中，我们讨论了如何使用已经为 Autoconf 做过准备的项目来为 Automake 做准备。（较新的项目通常同时为 Autoconf 和 Automake 做准备。）

我们讨论了如何使用`SUBDIRS`变量将*Makefile.am*文件连接起来，以及围绕产品列表、产品源和产品选项变量的相关概念。除了产品列表变量外，我还介绍了 Automake 的主要概念——这也是 Automake 的核心概念。最后，我讲解了如何使用`EXTRA_DIST`将附加文件添加到分发包中，使用`AM_MAINTAINER_MODE`宏确保用户不需要安装 Autotools，如何转换为非递归的 Automake 构建系统，以及如何使用 Automake 的静默规则。

通过这一切，我们用简短、简洁的*Makefile.am*文件替换了手写的*Makefile.in*模板，这些文件提供了显著更多的功能。我希望这个练习能让你开始意识到使用 Automake 而不是手写 makefile 的好处。

在第七章和第八章中，我们将研究如何将 Libtool 添加到 Jupiter 项目中。在第九章中，我们将通过深入探讨 Autoconf 的便携式测试框架——autotest，完成对 Autotools 的介绍。接着，在第十章至第十三章中，我们将暂时从 Autotools 中抽离，处理一些重要的旁支话题，但我们将在第十四章和第十五章中回归，带着“Autotool 化”一个真实项目，同时探索 Automake 的其他几个重要方面。
