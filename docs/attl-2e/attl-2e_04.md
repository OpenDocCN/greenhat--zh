## 第四章：使用 AUTOCONF 配置你的项目

*来吧，我的朋友们，‘寻找一个新的世界’还不算太晚。  

——阿尔弗雷德·丁尼生，《尤利西斯》*

![Image](img/common.jpg)

Autoconf 项目有着悠久的历史，始于 1992 年，当时 David McKenzie 在自愿为自由软件基金会工作时，正在寻找一种简化创建复杂配置脚本的方式，以支持当时每天添加到 GNU 项目的目标平台。与此同时，他还在马里兰大学帕克分校攻读计算机科学学士学位。

在 McKenzie 最初的 Autoconf 工作之后，他一直是该项目的重要贡献者，直到 1996 年，Ben Elliston 接手了项目的维护。从那时起，维护者和主要贡献者包括 Akim Demaille、Jim Meyering、Alexandre Oliva、Tom Tromey、Lars J. Aas（*autom4te* 这个名字的发明者之一）等人，Mo DeJong、Steven G. Johnson、Matthew D. Langston、Paval Roskin 和 Paul Eggert（贡献者名单更长，详情见 Autoconf 的 *AUTHORS* 文件）。

今天的维护者，Eric Blake，自 2012 年起开始为 Autoconf 做出重要贡献。从那时起，他一直担任该项目的维护者，并为 Red Hat 工作。由于 Automake 和 Libtool 本质上是 Autoconf 框架的附加组件，因此花一些时间专注于在没有 Automake 和 Libtool 的情况下使用 Autoconf 是有益的。这将提供一些洞察，帮助理解 Autoconf 的操作方式，因为许多由 Automake 隐藏的工具细节将得以显现。

在 Automake 出现之前，Autoconf 是单独使用的。事实上，许多遗留的开源项目从未从 Autoconf 过渡到完整的 GNU Autotools 套件。因此，在较旧的开源项目中，找到一个名为 *configure.in* 的文件（这是原始的 Autoconf 命名约定），以及手写的 *Makefile.in* 模板，并不罕见。

在本章中，我将向你展示如何为现有项目添加 Autoconf 构建系统。我将在本章的大部分时间里讲解 Autoconf 的基础功能，在 第五章中，我将详细介绍一些更复杂的 Autoconf 宏是如何工作的以及如何正确使用它们。在整个过程中，我们将继续使用 Jupiter 项目作为示例。

### Autoconf 配置脚本

`autoconf` 程序的输入是带有宏调用的 Bourne shell 脚本。输入数据流还必须包括所有引用宏的定义——包括 Autoconf 提供的宏和你自己编写的宏。

在 Autoconf 中使用的宏语言叫做 *M4*。（这个名字的含义是 *M，外加 4 个字母*，或者是 *宏* 这个词。^(1)) `m4` 工具是一个通用的宏语言处理器，最初由 Brian Kernighan 和 Dennis Ritchie 于 1977 年编写。

虽然你可能不熟悉它，但你可以在今天使用的每个 Unix 和 Linux 变体（以及其他系统）中找到某种形式的 M4。这个工具的普及性是 Autoconf 使用它的主要原因，因为 Autoconf 的原始设计目标就是能够在所有系统上运行，而不需要添加复杂的工具链和实用程序集。^(2)

Autoconf 依赖于相对较少的工具：Bourne shell、M4 和 Perl 解释器。它生成的配置脚本和 Makefile 依赖于一组不同的工具，包括 Bourne shell、`grep`、`ls`以及`sed`或`awk`。^(3)

**注意**

*不要将 Autotools 的要求与它们生成的脚本和 Makefile 的要求混淆。Autotools 是维护工具，而生成的脚本和 Makefile 是最终用户工具。我们可以合理地预期开发系统中安装的功能比最终用户系统中更多。*

配置脚本确保最终用户的构建环境已正确配置，以构建你的项目。该脚本检查已安装的工具、实用程序、库和头文件，以及这些资源中的特定功能。Autoconf 与其他项目配置框架的不同之处在于，Autoconf 的测试还确保这些资源能够被你的项目正确使用。你看，不仅仅是你的用户在他们的系统上正确安装了*libxyz.so*及其公共头文件，更重要的是，他们的文件版本是否兼容。Autoconf 在这类测试上非常严格。它通过为每个功能编译和链接一个小的测试程序，确保最终用户的环境符合项目要求——如果你愿意，它就像是一个示范例子，做的事情与项目源代码在更大范围内所做的相同。

*难道我不能仅仅通过在库路径中搜索文件名来确保* libxyz.2.1.0.so *已安装吗？* 这个问题的答案是有争议的。在一些合法的情况下，库和工具会悄无声息地更新。有时，项目所依赖的特定功能是以安全错误修复或库功能增强的形式添加的，在这种情况下，供应商甚至不需要更新版本号。但通常很难判断你拥有的是版本 2.1.0.r1 还是版本 2.1.0.r2，除非你查看文件大小或调用库函数来确保它按预期工作。

此外，供应商经常会将新产品中的错误修复和功能回移到旧平台，而不更新版本号。因此，仅通过查看版本号，你无法判断库是否支持在该版本库发布后*新增*的功能。

然而，不依赖库版本号的最重要原因是，它们并不代表库的特定营销版本。如我们在第八章中将讨论的，库版本号表示的是特定平台上的二进制接口特性。这意味着，同一功能集的库版本号可能在不同平台之间有所不同。因此，除非进行编译和链接到该库，否则你可能无法判断某个库是否具备你的项目所需的功能。

最后，有几个重要情况是，在不同的系统上，完全不同的库提供相同的功能。例如，你可能在一个系统中找到 *libtermcap* 提供光标操作功能，在另一个系统中找到 *libncurses*，而在另一个系统中找到 *libcurses*。但你不需要知道所有这些边缘情况，因为当你的项目在用户的系统上由于这些差异而无法构建时，用户会告诉你。

当报告此类 bug 时，你该怎么办？你可以使用 Autoconf 的 `AC_SEARCH_LIBS` 宏来测试多个库是否具备相同的功能。只需将一个库添加到搜索列表中，完成即可。由于这个修复非常简单，发现问题的用户很可能会直接发送一个补丁到你的 *configure.ac* 文件。

由于 Autoconf 测试是用 shell 脚本编写的，你在测试操作的方式上有很大的灵活性。你可以编写一个仅检查用户系统中常见位置是否存在某个库或工具的测试，但这绕过了 Autoconf 的一些重要特性。幸运的是，Autoconf 提供了数十个符合其特性测试哲学的宏。你应当仔细研究并使用可用宏的列表，而不是编写自己的宏，因为它们专门设计来确保所需功能在尽可能多的系统和平台上可用。

### 最简洁的 `configure.ac` 文件

`autoconf` 的输入文件叫做 *configure.ac*。最简单的 *configure.ac* 文件只有两行，如清单 4-1 所示。

```
AC_INIT([Jupiter], [1.0])
AC_OUTPUT
```

*清单 4-1：最简单的 configure.ac 文件*

对于新接触 Autoconf 的人来说，这两行看起来像是几个函数调用，可能是某种晦涩的编程语言的语法。不要让它们的外观把你吓到——这些是 M4 宏调用。这些宏定义在与 autoconf 软件包一起分发的文件中。例如，你可以在 Autoconf 的安装目录中的 *general.m4* 文件中找到 `AC_INIT` 的定义（通常是 */usr/(local/)share/autoconf/autoconf*）。`AC_OUTPUT` 的定义在同一目录下的 *status.m4* 中。

### 将 M4 与 C 预处理器进行比较

M4 宏在许多方面类似于在 C 语言源文件中定义的 C 预处理器（CPP）宏。C 预处理器也是一种文本替换工具，这并不奇怪：M4 和 C 预处理器是由 Kernighan 和 Ritchie 在差不多同一时期设计和编写的。

Autoconf 使用方括号将宏参数括起来作为引用机制。引用仅在宏调用的上下文可能导致歧义，而宏处理器可能错误地解决这种歧义时才需要。我们将在第十六章中详细讨论 M4 的引用。现在，只需在每个参数周围使用方括号，以确保生成预期的宏展开。

与 CPP 宏一样，你可以定义 M4 宏来接受以逗号分隔并括在括号中的参数列表。与 CPP 中通过*预处理器指令*定义宏：`#define` *`name`*`(`*`args`*`)` *`expansion`*不同，在 M4 中，宏是通过内建宏定义的：`define(`*`name`*`,` *`expansion`*`)`。另一个显著的区别是，在 CPP 中，宏定义中指定的参数是必需的^(4)，而在 M4 中，参数化宏的参数是可选的，调用者可以简单地省略它们。如果没有传递参数，你也可以省略括号。传递给 M4 宏的额外参数会被忽略。最后，M4 不允许宏调用中的宏名和开括号之间有空格。

### M4 宏的性质

如果你已经在 C 语言中编程多年，你无疑遇到过一些来自低层次黑暗区域的 C 预处理器宏。我说的就是那些真正邪恶的宏，它们展开后会生成一到两页的 C 代码。它们本应被写成 C 函数，但它们的作者要么过于担心性能，要么只是过于兴奋，结果现在轮到你来调试和维护它们了。但是，正如任何资深 C 程序员会告诉你的，使用宏而不是函数所带来的轻微性能提升，并不足以弥补你给维护者带来的调试麻烦。调试这样的宏可能是一场噩梦，因为宏生成的源代码通常无法通过符号调试器访问^(5)。

编写这种复杂的宏被 M4 程序员视为一种宏的极乐世界——它们越复杂、越功能强大，就越“酷”。在清单 4-1 中的两个 Autoconf 宏展开后会生成一个包含近 2400 行 Bourne-shell 脚本的文件，总大小超过 70KB！但你通过查看它们的定义是猜不到这一点的。它们都相当简短——每个只有几十行。这个明显差异的原因很简单：它们是以模块化的方式编写的，每个宏都扩展其他几个宏，后者又扩展其他几个宏，依此类推。

与编程人员被教导不滥用 C 预处理器的原因相同，广泛使用 M4 会给那些试图理解 Autoconf 的人带来相当大的困惑。这并不是说 Autoconf 不应该这样使用 M4；恰恰相反——这正是 M4 的领域。但也有一种观点认为，M4 对于 Autoconf 来说是一个不太好的选择，因为前面提到的宏问题。幸运的是，通常有效地使用 Autoconf 并不需要深入理解它附带的宏的内部工作原理。^(6)

### 执行 autoconf

运行 Autoconf 非常简单：只需在与*configure.ac*文件相同的目录中执行`autoconf`。虽然我可以为本章中的每个示例都执行这个操作，但我将使用`autoreconf`程序，而不是`autoconf`程序，因为运行`autoreconf`的效果与运行`autoconf`完全相同，只不过`autoreconf`在你开始向构建系统中添加 Automake 和 Libtool 功能时，也会正确地执行所有 Autotools。也就是说，它会根据*configure.ac*文件的内容按正确的顺序执行所有 Autotools。

`autoreconf`程序足够智能，只会按正确的顺序执行你需要的工具，并使用你想要的选项（有一个小小的限制，我会在稍后提到）。因此，运行`autoreconf`是执行 Autotools 工具链的推荐方法。

首先，将来自清单 4-1 的简单*configure.ac*文件添加到我们的项目目录中。当前的顶层目录仅包含一个*Makefile*和一个*src*目录，后者包含其自己的*Makefile*和一个*main.c*文件。一旦你将*configure.ac*添加到顶层目录中，运行`autoreconf`：

Git 标签 4.0

```
$ autoreconf
$
$ ls -1p
autom4te.cache/
configure
configure.ac
Makefile
src/
$
```

首先，注意到`autoreconf`默认是静默运行的。如果你想看到一些执行过程，可以使用`-v`或`--verbose`选项。如果你希望`autoreconf`以详细模式执行 Autotools，也可以在命令行中添加`-vv`。^(7)

接下来，注意到`autoconf`会创建一个名为*autom4te.cache*的目录。这是`autom4te`缓存目录。该缓存加速了在连续执行 Autotools 工具链中的实用工具时对*configure.ac*的访问。

将*configure.ac*通过`autoconf`处理的结果基本上是相同的文件（现在叫做`configure`），但是所有的宏都已经完全展开。你可以查看`configure`文件，但如果你立即无法理解其中的内容，也不必太惊讶。*configure.ac*文件已经通过 M4 宏展开转换成一个包含数千行复杂 Bourne shell 脚本的文本文件。

### 执行 configure

正如在“配置你的软件包”一节中讨论的，在 第 77 页，*GNU 编程标准* 指出手写的 `configure` 脚本应该生成另一个名为 `config.status` 的脚本，它的任务是从模板生成文件。毫不意外，这正是你在 Autoconf 生成的配置脚本中会找到的功能。这个脚本有两个主要任务：

+   执行请求的检查

+   生成并调用 `config.status`

`configure` 执行的检查结果会写入 `config.status`，以便作为 Autoconf 替换变量的替换文本，用于模板文件中（*Makefile.in*、*config.h.in* 等）。当你执行 `./configure` 时，它会告诉你它正在创建 `config.status`。它还会创建一个名为 *config.log* 的日志文件，包含一些重要的属性。我们来运行 `./configure`，然后看看我们的项目目录中新增加了什么：

```
$ ./configure
configure: creating ./config.status
$
$ ls -1p
autom4te.cache/
config.log
config.status
configure
configure.ac
Makefile
src/
$
```

我们可以看到，`configure` 确实生成了 `config.status` 和 *config.log* 文件。*config.log* 文件包含以下信息：*

** 用来调用 `configure` 的命令行（非常有用！）

+   `configure` 执行时的平台信息

+   `configure` 执行的核心测试信息

+   `configure` 中生成并调用 `config.status` 的行号

在日志文件的这一部分，`config.status` 接管生成日志信息，并添加以下内容：

+   用于调用 `config.status` 的命令行

在 `config.status` 从模板中生成所有文件后，它退出并将控制权返回给 `configure`，然后 `configure` 将以下信息附加到日志中：

+   `config.status` 用来执行任务的缓存变量

+   可能会在模板中替换的输出变量列表

+   `configure` 返回给 shell 的退出代码

当你调试 `configure` 脚本及其相关的 *configure.ac* 文件时，这些信息非常宝贵。

为什么 `configure` 不直接执行它写入 `config.status` 中的代码，而要经历生成第二个脚本并立即调用它的麻烦呢？有几个很好的理由。首先，执行检查和生成文件是概念上不同的操作，`make` 工具在将概念上不同的操作与独立的目标关联时效果最佳。第二个原因是，你可以单独执行 `config.status`，从而重新生成输出文件，而不需要再次执行那些繁琐的检查，这样可以节省时间。最后，`config.status` 会记住最初在 `configure` 命令行上使用的参数。因此，当 `make` 检测到需要更新构建系统时，它可以调用 `config.status` 重新执行 `configure`，使用最初指定的命令行选项。

### 执行 config.status

现在你已经了解了`configure`是如何工作的，你可能会想自己执行`config.status`。这正是 Autoconf 设计者和*GCS*的作者们的初衷，他们最初构思了这些设计目标。然而，将检查与模板处理分开更为重要的原因是，`make`规则可以使用`config.status`来重新生成 makefile，当`make`判断模板比对应的 makefile 更新时。

不要调用`configure`进行不必要的检查（因为你的环境没有改变——只是模板文件改变了），应该编写 makefile 规则来指示输出文件依赖于其模板。这些规则的命令会运行`config.status`，并将规则的目标作为参数传递。例如，如果你修改了其中一个*Makefile.in*模板，`make`会调用`config.status`来重新生成相应的*Makefile*，然后`make`会重新执行它最初的命令行——基本上是重新启动自己。^(8)

列表 4-2 显示了该*Makefile.in*模板的相关部分，包含了重新生成相应*Makefile*所需的规则。

```
Makefile: Makefile.in config.status
        ./config.status $@
```

*列表 4-2：一个规则，如果其模板发生变化，将导致`make`重新生成* Makefile*

这里触发规则的目标是`Makefile`。这个规则允许`make`在模板发生变化时从其模板重新生成源 makefile。它会在执行用户指定的目标或默认目标之前执行，如果没有指定特定目标的话。这个功能是`make`内建的——如果有一个目标为`Makefile`的规则，`make`总是会首先评估这个规则。

列表 4-2 中的规则表明，*Makefile*依赖于`config.status`和*Makefile.in*，因为如果`configure`更新了`config.status`，它可能会不同地生成*Makefile*。也许提供了不同的命令行选项，以便`configure`现在能够找到之前找不到的库和头文件。在这种情况下，Autoconf 替代变量可能会有不同的值。因此，如果*Makefile.in*或`config.status`中的任何一个被更新，*Makefile*应该重新生成。

由于`config.status`本身是一个生成的文件，因此可以推理出，你可以编写这样的规则，在需要时重新生成这个文件。在前面的例子基础上，列表 4-3 添加了所需的代码，以便在`configure`改变时重建`config.status`。

```
Makefile: Makefile.in config.status
        ./config.status $@

config.status: configure
        ./config.status --recheck
```

*列表 4-3：当`configure`改变时，重建`config.status`的规则*

由于`config.status`是`Makefile`目标的依赖项，`make`会寻找一个目标为`config.status`的规则，并在需要时执行其命令。

### 添加一些实际功能

我曾经建议过，您应该在 makefiles 中调用 `config.status` 来从模板生成这些 makefiles。清单 4-4 显示了实际执行此操作的 *configure.ac* 中的代码。这只是 清单 4-1 中两个原始行之间的一个额外宏调用。

Git 标签 4.1

```
AC_INIT([Jupiter],[1.0])
AC_CONFIG_FILES([Makefile src/Makefile])
AC_OUTPUT
```

*清单 4-4:* configure.ac: *使用 `AC_CONFIG_FILES` 宏*

该代码假设存在 *Makefile* 和 *src/Makefile* 的模板，分别称为 *Makefile.in* 和 *src/Makefile.in*。这些模板文件看起来与其 *Makefile* 对应文件完全相同，唯一的例外是：任何我希望 Autoconf 替换的文本，都标记为 Autoconf 替代变量，使用 `@`*`VARIABLE`*`@` 语法。

要创建这些文件，只需将现有的 *Makefile* 文件在顶层和 *src* 目录中重命名为 *Makefile.in*。这是将项目 *autoconfiscate* 的常见做法：

```
$ mv Makefile Makefile.in
$ mv src/Makefile src/Makefile.in
$
```

有了这些更改后，我们现在已经有效地在 Jupiter 中使用新的 *configure.ac* 文件来生成 makefiles。为了让它更有用，我们可以添加一些 Autoconf 替代变量来替换原始的默认值。在这些文件的顶部，我还添加了 Autoconf 替代变量 `@configure_input@`，并在注释符号后面添加。 清单 4-5 显示了在 *Makefile* 中生成的注释文本。

Git 标签 4.2

```
# Makefile. Generated from Makefile.in by configure.
--snip--
```

*清单 4-5:* Makefile: *从 Autoconf `@configure_input@` 变量生成的文本*

我还将之前示例中的 makefile 重新生成规则添加到了每个模板中，每个文件中略有不同的路径差异，以考虑到它们相对于构建目录中的 `config.status` 和 `configure` 的不同位置。

清单 4-6 和 4-7 突出了从 第三章 末尾部分到最终递归版本的 *Makefile* 和 *src/Makefile* 所需的更改。稍后我们将在讲解 Automake 时考虑编写这些文件的非递归版本——使用 Autoconf 和手写的 *Makefile.in* 模板的过程几乎与我们在 第三章 中使用 makefiles 所做的完全相同。^(9)

```
# @configure_input@

# Package-specific substitution variables
package = @PACKAGE_NAME@
version = @PACKAGE_VERSION@
tarname = @PACKAGE_TARNAME@
distdir = $(tarname)-$(version)

# Prefix-specific substitution variables
prefix = @prefix@
exec_prefix = @exec_prefix@
bindir = @bindir@

all clean check install uninstall jupiter:
        cd src && $(MAKE) $@
--snip--
$(distdir): FORCE
        mkdir -p $(distdir)/src
        cp configure.ac $(distdir)
        cp configure $(distdir)
        cp Makefile.in $(distdir)
        cp src/Makefile.in src/main.c $(distdir)/src

distcheck: $(distdir).tar.gz
        gzip -cd $(distdir).tar.gz | tar xvf -
        cd $(distdir) && ./configure
        cd $(distdir) && $(MAKE) all
        cd $(distdir) && $(MAKE) check
        cd $(distdir) && $(MAKE) DESTDIR=$${PWD}/_inst install
        cd $(distdir) && $(MAKE) DESTDIR=$${PWD}/_inst uninstall
        @remaining="`find $${PWD}/$(distdir)/_inst -type f | wc -l`"; \
        if test "$${remaining}" -ne 0; then \
          echo "*** $${remaining} file(s) remaining in stage directory!"; \
          exit 1; \
        fi
        cd $(distdir) && $(MAKE) clean
        rm -rf $(distdir)
        @echo "*** Package $(distdir).tar.gz is ready for distribution."
--snip--
FORCE:
        rm -f $(distdir).tar.gz
        rm -rf $(distdir)

Makefile: Makefile.in config.status
        ./config.status $@

config.status: configure
        ./config.status --recheck

.PHONY: FORCE all clean check dist distcheck install uninstall
```

*清单 4-6:* Makefile.in: *来自 第三章 的 Makefile 所需修改*

```
# @configure_input@

# Package-specific substitution variables
package = @PACKAGE_NAME@
version = @PACKAGE_VERSION@
tarname = @PACKAGE_TARNAME@
distdir = $(tarname)-$(version)

# Prefix-specific substitution variables
prefix = @prefix@
exec_prefix = @exec_prefix@
bindir = @bindir@

CFLAGS = -g -O0
--snip--
clean:
        rm -f jupiter
 Makefile: Makefile.in ../config.status
        cd .. && ./config.status src/$@

../config.status: ../configure
        cd .. && ./config.status --recheck

.PHONY: all clean check install uninstall
```

*清单 4-7:* src/Makefile.in: *来自 第三章 的 src/Makefile 所需修改*

我已经从顶层的 *Makefile.in* 中移除了 `export` 语句，并将所有的 `make` 变量（最初只在顶层 *Makefile* 中）复制到了 *src/Makefile.in* 中。由于 `config.status` 会生成这两个文件，因此我可以通过直接将这些变量的值替换到这两个文件中，从中获得显著的好处。这样做的主要优势是，我现在可以在任何子目录中运行 `make`，而无需担心那些原本由更高级别的 makefile 传递的未初始化变量。

由于 Autoconf 为这些`make`变量生成完整的值，你可能会想通过删除这些变量，只在文件中使用`@prefix@`来替代当前使用的`$(prefix)`，从而简化内容。保留`make`变量是有几个充分理由的。首先，我们将保留`make`变量的原始优点；最终用户可以继续在`make`命令行中替换他们自己的值。（即使 Autoconf 为这些变量设置了默认值，用户可能希望覆盖它们。）其次，对于像`$(distdir)`这样的变量，其值由多个变量引用组成，在一个地方构建这个名称并通过单个变量在其他地方使用，这样会显得更简洁。

我还稍微改变了一些分发目标中的命令。我不再分发 makefile，而是需要分发*Makefile.in*模板、新的`configure`脚本和*configure.ac*文件。^(10)

最终，我修改了`distcheck`目标的命令，使其在运行`make`之前先运行`configure`脚本。

### 从模板生成文件

请注意，你可以使用`AC_CONFIG_FILES`从同一目录中找到具有*.in*扩展名的同名文件生成*任何*文本文件。*.in*扩展名是`AC_CONFIG_FILES`的默认模板命名模式，但你可以覆盖这个默认行为。我稍后会详细讲解。

Autoconf 将`sed`或`awk`表达式生成到结果`configure`脚本中，然后将它们复制到`config.status`中。`config.status`脚本使用这些表达式在输入模板文件中执行字符串替换。

`sed`和`awk`都是处理文件流的文本处理工具。流编辑器的优点（*sed*这个名字是*stream editor*的缩写）是它能在字节流中替换文本模式。因此，`sed`和`awk`可以处理非常大的文件，因为它们不需要将整个输入文件加载到内存中进行处理。Autoconf 根据各种宏定义的变量列表构建`config.status`传递给`sed`或`awk`的表达式列表，其中许多宏我将在本章稍后更详细地讲解。重要的是要理解，Autoconf 替换变量是模板文件中生成输出文件时唯一会被替换的内容。

到目前为止，我几乎没有花费多少精力，就创建了一个基本的*configure.ac*文件。我现在可以执行`autoreconf`，然后执行`./configure`，接着是`make`，以便构建 Jupiter 项目。这个简单的三行*configure.ac*文件生成了一个完全可用的`configure`脚本，符合*GCS*所规定的正确配置脚本定义。

生成的配置脚本会运行各种系统检查，并生成一个 `config.status` 脚本，该脚本可以替换在此构建系统中指定的模板文件集中的许多替换变量。这仅仅是三行代码就实现了这么多功能。

### 添加 VPATH 构建功能

在 第三章 结束时，我提到过，我还没有讲解一个重要的概念——即 vpath 构建。*vpath 构建* 是一种使用 `make` 构造（`VPATH`）来在不同于源目录的目录中 `configure` 和构建项目的方法。如果你需要执行以下任何任务，这个概念很重要：

+   维护一个单独的调试配置

+   侧边比较不同的配置

+   在本地修改后，为补丁差异保留一个干净的源目录

+   从只读源目录进行构建

`VPATH` 关键字是 *虚拟搜索路径* 的缩写。一个 `VPATH` 语句包含一个由冒号分隔的路径列表，用于查找相对路径的依赖项，当它们在当前目录中无法找到时。换句话说，当 `make` 无法在当前目录中找到一个前置文件时，它会依次在 `VPATH` 语句中的每个路径中查找该文件。

使用 `VPATH` 向现有的 makefile 添加远程构建功能非常简单。清单 4-8 展示了在 makefile 中使用 `VPATH` 语句的一个示例。

```
VPATH = some/path:some/other/path:yet/another/path

program : src/main.c
        $(CC) ...
```

*清单 4-8：在 makefile 中使用 `VPATH` 的示例*

在这个（假设的）示例中，如果 `make` 在处理规则时无法在当前目录中找到 *src/main.c*，它将依次查找 *some/path/src/main.c*、*some/other/path/src/main.c*，最后查找 *yet/another/path/src/main.c*，然后因无法找到 *src/main.c* 而报错。

只需做几处简单的修改，我们就能完全支持 Jupiter 中的远程构建。清单 4-9 和 4-10 说明了对项目的两个 makefile 进行必要更改的方式。

Git 标签 4.3

```
--snip--
# Prefix-specific substitution variables
prefix = @prefix@
exec_prefix = @exec_prefix@
bindir = @bindir@

# VPATH-specific substitution variables
srcdir = @srcdir@
VPATH = @srcdir@
--snip--
$(distdir): FORCE
        mkdir -p $(distdir)/src
        cp $(srcdir)/configure.ac $(distdir)
        cp $(srcdir)/configure $(distdir)
        cp $(srcdir)/Makefile.in $(distdir)
        cp $(srcdir)/src/Makefile.in $(srcdir)/src/main.c $(distdir)/src
--snip--
```

*清单 4-9:* Makefile.in: *向顶层 makefile 添加 VPATH 构建功能*

```
--snip--
# Prefix-specific substitution variables
prefix = @prefix@
exec_prefix = @exec_prefix@
bindir = @bindir@

# VPATH-specific substitution variables
srcdir = @srcdir@
VPATH = @srcdir@
--snip--
jupiter: main.c
        $(CC) $(CPPFLAGS) $(CFLAGS) -o $@ $(srcdir)/main.c
--snip--
```

*清单 4-10:* src/Makefile.in: *向低级 makefile 添加 `VPATH` 构建功能*

就是这样，真的。当 `config.status` 生成一个文件时，它会将一个叫做 `@srcdir@` 的 Autoconf 替换变量替换为模板源目录的相对路径。在构建目录结构中的给定 *Makefile* 中，替换 `@srcdir@` 的值是源目录结构中包含相应 *Makefile.in* 模板的目录的相对路径。这里的概念是，对于远程构建目录中的每个 *Makefile*，`VPATH` 提供了一个相对路径，指向该构建目录的源代码所在的目录。

**注意**

*不要期望*`VPATH`* 在命令中有效。*`VPATH`* 只允许*`make`* 查找依赖项；因此，你只能期望*`VPATH`* 在规则中的目标和依赖列表中生效。你可以像我在 Listing 4-10 中所做的那样，在命令中使用*`$(srcdir)/`*作为文件系统对象的前缀，针对*`jupiter`* 目标规则。*

支持远程构建所需的更改在你的构建系统中总结如下：

+   将 `make` 变量 `srcdir` 设置为 `@srcdir@` 替代变量。

+   将 `VPATH` 变量设置为 `@srcdir@`。

+   在*命令中*，所有文件依赖项前缀都需要加上 `$(srcdir)/`。

**注意**

*不要在*`VPATH`* 语句中使用*`$(srcdir)`*，因为一些旧版本的*`make`* 在*`VPATH`* 语句中不会替换变量引用。*

如果源目录与构建目录相同，`@srcdir@` 替代变量会退化为一个点（*.*）。这意味着所有这些 `$(srcdir)`*/* 前缀都会简单地退化为 *./*，这无害。^(11)

一个简单的例子是展示这个功能如何工作的最简单方法。现在，Jupiter 在远程构建方面已经完全功能化，让我们试试看。在 Jupiter 项目目录中开始，创建一个名为*build*的子目录，然后进入该目录。使用相对路径执行 `configure` 脚本，然后列出当前目录的内容：

```
$ mkdir build
$ cd build
$ ../configure
configure: creating ./config.status
config.status: creating Makefile
config.status: creating src/Makefile
$
$ ls -1p
config.log
config.status
Makefile
src/
$
$ ls -1p src
Makefile
$
```

整个构建系统已由 `configure` 和 `config.status` 在*build* 子目录中构建完成。从*build* 目录内进入 `make` 以构建项目：

```
$ make
cd src && make all
make[1]: Entering directory '.../jupiter/build/src'
cc -g -O0 -o jupiter ../../src/main.c
make[1]: Leaving directory '.../jupiter/build/src'
$
$ ls -1p src
jupiter
Makefile
$
```

无论你身处何地，只要可以通过相对路径或绝对路径访问项目目录，你就可以从该位置进行远程构建。这仅仅是 Autoconf 在 Autoconf 生成的配置脚本中为你做的一件事。想象一下，在你自己编写的配置脚本中管理源目录的适当相对路径！

### 稍作休息

到目前为止，我已经展示了一个几乎完整的构建系统，包含了*GCS*中概述的几乎所有功能。Jupiter 的构建系统的特性都相对独立，并且容易理解。手动实现的最困难的功能是配置脚本。实际上，与使用 Autoconf 的简单性相比，手写配置脚本是如此繁琐，以至于我在 第三章 中完全跳过了手写版本。

尽管像我这里展示的那样使用 Autoconf 是非常简单的，但大多数人并不会像我展示的那样创建他们的构建系统。相反，他们会尝试复制另一个项目的构建系统，并对其进行调整，以使其在自己的项目中工作。后来，当他们开始一个新项目时，他们又会做同样的事情。这可能会导致问题，因为他们复制的代码从来没有打算以他们现在尝试的方式使用。

我曾见过一些项目，其中的 *configure.ac* 文件包含与所属项目无关的垃圾。这些残留的内容来自某个遗留项目，但维护者并不了解 Autoconf，因此无法正确地删除所有多余的文本。使用 Autotools 时，通常最好从小开始，根据需要添加内容，而不是从另一个功能齐全的构建系统中复制一个 *configure.ac* 文件，然后试图将其缩减到适合的大小或修改它以适应新项目。

我相信你一定觉得 Autoconf 还有很多东西需要学习，你说得对。我们将在本章剩余的部分研究最重要的 Autoconf 宏以及它们在 Jupiter 项目中的使用方式。但首先，让我们回过头来看看，是否可以通过使用 Autoconf 包中另一个工具，进一步简化 Autoconf 启动过程。

### 使用 autoscan 更快速的开始

创建一个（基本）完整的 *configure.ac* 文件的最简单方法是运行 `autoscan` 工具，它是 Autoconf 包的一部分。该工具会检查项目目录的内容，并使用现有的 makefile 和源文件生成一个 *configure.ac* 文件的基础（`autoscan` 将其命名为 *configure.scan*）。

让我们看看 `autoscan` 在 Jupiter 项目中表现如何。首先，我将清理掉我之前实验中留下的痕迹，然后我将在 *jupiter* 目录中运行 `autoscan`。

**注意**

*如果你正在使用本书附带的 git 仓库，你可以简单地运行 *`git clean -df`* 来删除所有未被 git 源控制管理的文件和目录。别忘了，如果你仍然在构建目录中，切换回父目录。*

请注意，我*并没有*删除我的原始 *configure.ac* 文件——我只是让 `autoscan` 告诉我如何改进它。在不到一秒钟的时间里，我在顶层目录下得到了一些新的文件：

```
   $ cd ..
   $ git clean -df
   $ autoscan
➊ configure.ac: warning: missing AC_CHECK_HEADERS([stdlib.h]) wanted by:
     src/main.c:2
   configure.ac: warning: missing AC_PREREQ wanted by: autoscan
   configure.ac: warning: missing AC_PROG_CC wanted by: src/main.c
   configure.ac: warning: missing AC_PROG_INSTALL wanted by: Makefile.in:18
   $
   $ ls -1p
   autom4te.cache/
   autoscan.log
   configure.ac
   configure.scan
   Makefile.in
   src/
   $
```

`autoscan` 工具会检查项目目录结构，并创建两个文件：*configure.scan* 和 *autoscan.log*。该项目可能已经为 Autotools 做了准备，也可能没有——这并不重要，因为 `autoscan` 是完全无破坏性的。它绝不会修改项目中任何现有的文件。

`autoscan` 工具会为它在现有的 *configure.ac* 文件中发现的每个问题生成一条警告消息。在这个例子中，`autoscan` 注意到 *configure.ac* 应该使用 Autoconf 提供的 `AC_CHECK_HEADERS`、`AC_PREREQ`、`AC_PROG_CC` 和 `AC_PROG_INSTALL` 宏。它是根据从现有的 *Makefile.in* 模板和 C 语言源文件中获取的信息做出这些假设的，正如你在警告语句后的评论中看到的那样，警告语句从 ➊ 开始。你可以通过检查 *autoscan.log* 文件来查看这些消息（更详细的信息）。

**注意**

*你从*`autoscan`* 接收到的通知和你的 configure.ac 文件的内容可能会根据你安装的 Autoconf 版本与我的略有不同。我系统中安装的是 GNU Autoconf 的 2.69 版本（截至本文写作时为最新版本）。如果你的 *`autoscan`* 版本较旧（或较新），你可能会看到一些小的差异。*

查看生成的 *configure.scan* 文件，我注意到 `autoscan` 向该文件添加的文本比我原始的 *configure.ac* 文件中的内容更多。查看后我确保理解所有内容后，我发现最简单的方法是用 *configure.scan* 文件覆盖 *configure.ac* 文件，然后更改一些特定于 Jupiter 的信息：

```
$ mv configure.scan configure.ac
$ cat configure.ac
#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.69])
AC_INIT([FULL-PACKAGE-NAME], [VERSION], [BUG-REPORT-ADDRESS])
AC_CONFIG_SRCDIR([src/main.c])
AC_CONFIG_HEADERS([config.h])

# Checks for programs.
AC_PROG_CC
AC_PROG_INSTALL

# Checks for libraries.

# Checks for header files.
AC_CHECK_HEADERS([stdlib.h])

# Checks for typedefs, structures, and compiler characteristics.

# Checks for library functions.
AC_CONFIG_FILES([Makefile
                 src/Makefile])
AC_OUTPUT
$
```

我的第一次修改涉及更改 Jupiter 的 `AC_INIT` 宏参数，如 示例 4-11 所示。

Git 标签 4.4

```
#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.69])
AC_INIT([Jupiter], [1.0], [jupiter-bugs@example.org])
AC_CONFIG_SRCDIR([src/main.c])
AC_CONFIG_HEADERS([config.h])
--snip--
```

*示例 4-11：* configure.ac: *调整 `autoscan` 生成的 `AC_INIT` 宏*

`autoscan` 工具为你做了大量的工作。*GNU Autoconf 手册*^(12) 中指出，在使用此文件之前，你应根据项目的需求修改该文件，但除了与 `AC_INIT` 相关的问题外，只有一些关键问题需要关注。我将依次讲解这些问题，但首先，我们需要处理一些行政细节。

在讨论 `autoscan` 时，我必须提到 `autoupdate`。如果你已经有了一个工作正常的 *configure.ac* 文件，并且你更新到更新版本的 Autoconf，你可以运行 `autoupdate` 来更新你现有的 *configure.ac* 文件，使其包含自旧版本 Autoconf 以来更改或新增的构造。

#### *所谓的 bootstrap.sh 脚本*

在 `autoreconf` 出现之前，维护者们会传递一个简短的 shell 脚本，通常命名为 `autogen.sh` 或 `bootstrap.sh`，该脚本会按照正确的顺序运行所需的所有 Autotools。这个脚本的推荐名称是 `bootstrap.sh`，因为 Autogen 是另一个 GNU 项目的名称。`bootstrap.sh` 脚本可以相当复杂，但为了处理缺失的 `install-sh` 脚本问题（请参见“Autoconf 中缺失的必需文件”），我将只添加一个简单的临时 `bootstrap.sh` 脚本到项目的根目录，如 示例 4-12 所示。

Git 标签 4.5

```
   #!/bin/sh
   autoreconf --install
➊ automake --add-missing --copy >/dev/null 2>&1
```

*示例 4-12：* bootstrap.sh: *一个临时的 bootstrap 脚本，用于执行所需的 Autotools*

Automake `--add-missing` 选项将所需的缺失工具脚本复制到项目中，`--copy` 选项表示应该创建真正的副本（否则，会创建指向安装目录中文件的符号链接，链接文件是与 Automake 包一起安装的）。^(13)

**注意**

*我们不需要看到执行 *`automake`* 时的警告，因此我在这个脚本的 ➊ 处将 *`stderr`* 和 *`stdout`* 流重定向到 /dev/null。在第六章中，我们将移除 *`bootstrap.sh`* 并简单地运行 *`autoreconf --install`*，但目前为止，这解决了我们缺失文件的问题。*

Autoconf 中缺少的必需文件

当我第一次尝试在 清单 4-11 中的 *configure.ac* 文件上执行 `autoreconf` 时，我发现了一个小问题，关于在没有 Automake 的情况下使用 Autoconf。当我运行 `configure` 脚本时，它因错误而失败：`configure: error: cannot find install-sh, install.sh, or shtool in "." "./.." "./../.."`。

Autoconf 旨在实现可移植性，但不幸的是，Unix 的 `install` 工具并不像它本可以那样可移植。从一个平台到另一个平台，安装功能的关键部分差异足够大，以至于会引发问题，因此 Autotools 提供了一个名为 `install-sh`（已弃用名称：`install.sh`）的 shell 脚本。该脚本作为系统自带的 `install` 工具的包装器，屏蔽了不同版本的 `install` 之间的重要差异。

`autoscan` 注意到我在 *src/Makefile.in* 模板中使用了 `install` 程序，因此它生成了 `AC_PROG_INSTALL` 宏的扩展。问题是 `configure` 无法在我的项目中找到 `install-sh` 包装脚本。

我推测缺失的文件是 Autoconf 包的一部分，只需要安装它即可。我还知道 `autoreconf` 接受一个命令行选项来将这些缺失的文件安装到项目目录中。`--install`（`-i`）选项由 `autoreconf` 支持，用于将特定于工具的选项传递给它调用的每个工具，以便安装缺失的文件。然而，当我尝试这个方法时，我发现文件依然丢失，因为 `autoconf` 不支持安装缺失文件的选项。

我本可以手动从 Automake 安装目录（通常是 */usr/(local/)share/automake-**）复制 `install-sh`，但为了寻找更自动化的解决方案，我尝试手动执行 `automake --add-missing --copy`。这个命令生成了大量警告，表明项目没有为 Automake 配置。然而，我现在可以看到 `install-sh` 已经被复制到了我的项目根目录，这正是我需要的。执行 `autoreconf --install` 并没有运行 `automake`，因为 *configure.ac* 没有为 Automake 设置。

Autoconf 应该随 `install-sh` 一起提供，因为它提供了一个需要该脚本的宏，但那样的话 `autoconf` 还需要提供一个 `--add-missing` 命令行选项。不过，实际上有一个相当明显的解决方案。`install-sh` 脚本并不是 Autoconf 生成的任何代码所必需的。怎么会呢？Autoconf 并不生成任何 makefile 构造——它只是将变量替换到你的 *Makefile.in* 模板中。因此，Autoconf 没有理由抱怨缺少 `install-sh` 脚本。

#### *更新 Makefile.in*

让我们使 `bootstrap.sh` 可执行，然后执行它，看看最终会得到什么：

```
   $ chmod +x bootstrap.sh
   $ ./bootstrap.sh
   $ ls -1p
   autom4te.cache/
   bootstrap.sh
➊ config.h.in
   configure
   configure.ac
➋ install-sh
   Makefile.in
   src/
   $
```

从 ➊ 处的文件列表中，我们知道已经创建了 *config.h.in*，因此我们知道 `autoreconf` 已执行了 `autoheader`。我们还看到新创建的 `install-sh` 脚本在 ➋ 处，它是在我们执行 `bootstrap.sh` 中的 `automake` 时创建的。任何 Autotools 提供或生成的文件都应该复制到归档目录中，以便可以随发行的 tarball 一起打包。因此，我们将为这两个文件添加 `cp` 命令到顶层 *Makefile.in* 模板中的 `$(distdir)` 目标。请注意，我们不需要复制 `bootstrap.sh` 脚本，因为它完全是一个维护工具—用户不应该需要从 tarball 分发版中执行它。

列表 4-13 展示了对顶层 *Makefile.in* 模板中的 `$(distdir)` 目标所需的更改。

Git 标签 4.6

```
--snip--
$(distdir): FORCE
        mkdir -p $(distdir)/src
        cp $(srcdir)/configure.ac $(distdir)
        cp $(srcdir)/configure $(distdir)
        cp $(srcdir)/config.h.in $(distdir)
        cp $(srcdir)/install-sh $(distdir)
        cp $(srcdir)/Makefile.in $(distdir)
        cp $(srcdir)/src/Makefile.in $(distdir)/src
        cp $(srcdir)/src/main.c $(distdir)/src
--snip--
```

*列表 4-13:* Makefile.in：*在分发归档镜像目录中需要的附加文件*

如果你开始觉得这可能会变成一个维护问题，那么你是对的。我之前提到过 `$(distdir)` 目标维护起来很痛苦。幸运的是，`distcheck` 目标仍然存在并按预期工作。它会捕捉到这个问题，因为没有这些附加文件，从 tarball 构建的尝试将失败—如果构建失败，`distcheck` 目标肯定不会成功。当我们在 第六章 中讨论 Automake 时，我们会清理掉很多维护上的麻烦。

### 初始化和包信息

现在，让我们回到 列表 4-11 中的 *configure.ac* 文件内容（以及该列表之前的控制台示例）。第一部分包含 Autoconf 初始化宏。这些是所有项目所必需的。让我们单独考虑每个宏，因为它们都很重要。

#### *AC_PREREQ*

`AC_PREREQ` 宏仅仅定义了可以成功处理此 *configure.ac* 文件的最早版本的 Autoconf：

```
AC_PREREQ(version)
```

*GNU Autoconf 手册* 表示，`AC_PREREQ` 是唯一可以在 `AC_INIT` 之前使用的宏。这是因为，在处理任何其他可能依赖版本的宏之前，确保使用新版本的 Autoconf 是一个好做法。

#### *AC_INIT*

`AC_INIT`宏，顾名思义，用于初始化 Autoconf 系统。以下是它的原型，定义在*GNU Autoconf 手册*中：^(14)

```
AC_INIT(package, version, [bug-report], [tarname], [url])
```

它最多接受五个参数（`autoscan`仅生成带有前三个参数的调用）：*`package`*、*`version`*，以及可选的*`bug-report`*、*`tarname`*和*`url`*。*`package`*参数是包的名称。在你执行`make dist`时，它会以标准形式作为 Automake 生成的发布版本 tarball 名称的一部分。

**注意**

*Autoconf 在 tarball 名称中使用了包名称的标准化形式，因此，如果你愿意，可以在包名中使用大写字母。Automake 生成的 tarballs 默认命名为*`tarname`*-*`version`*.tar.gz，但*`tarname`*被设置为包名称（小写，所有标点符号转换为下划线）的标准化形式。选择包名称和版本字符串时请记住这一点。*

可选的*`bug-report`*参数通常设置为一个电子邮件地址，但任何文本字符串都是有效的——一个接受项目 bug 报告的网页 URL 是常见的替代方式。一个名为`@PACKAGE_BUGREPORT@`的 Autoconf 替换变量会为它创建，并且该变量也会添加到*config.h.in*模板中，作为 C 预处理器定义。这样做的目的是让你在代码中使用这个变量，在适当的位置显示电子邮件地址或用于报告 bug 的 URL——可能是在用户请求帮助或版本信息时。

虽然*`version`*参数可以是你喜欢的任何内容，但有一些常用的开源软件规范可以让你更轻松地处理这个问题。最广泛使用的规范是传入*major.minor*（例如，1.2）。然而，并没有规定你不能使用*major.minor.revision*，这种方式也没有问题。生成的`VERSION`变量（Autoconf、shell 或`make`）在任何地方都不会被解析或分析——它们只是作为占位符，用于在不同位置替换文本。^(15) 所以如果你愿意，你甚至可以在这个宏中添加非数字文本，例如*0.15.alpha1*，这种做法有时也很有用。^(16)

**注意**

*另一方面，RPM 包管理器对版本字符串中的内容比较严格。为了兼容 RPM，你可能希望将版本字符串文本限制为仅包含字母数字字符和句点——不允许使用连字符或下划线。*

可选的*`url`*参数应该是你项目网站的 URL。它会显示在`configure --help`命令的帮助文本中。

Autoconf 根据`AC_INIT`的参数生成替换变量`@PACKAGE_NAME@`、`@PACKAGE_VERSION@`、`@PACKAGE_TARNAME@`、`@PACKAGE_STRING@`（包名称和版本信息的样式化连接）、`@PACKAGE_BUGREPORT@`和`@PACKAGE_URL@`。你可以在*Makefile.in*模板文件中使用这些变量中的任何一个或所有。

#### *AC_CONFIG_SRCDIR*

`AC_CONFIG_SRCDIR`宏是一个合理性检查。它的目的是确保生成的`configure`脚本知道它正在执行的目录实际上是项目目录。

更具体来说，`configure`需要能够找到它自己，因为它生成的代码需要执行自己，可能是从一个远程目录中执行。有许多方式可以不小心让`configure`找到其他的`configure`脚本。例如，用户可能会不小心为`configure`提供一个错误的`--srcdir`参数。`$0`这个 Shell 脚本参数最多也只是可靠，它可能包含的是 Shell 的名称，而不是脚本的名称，或者可能是`configure`在系统搜索路径中找到的，因此在命令行上没有指定路径信息。

`configure`脚本可以尝试在当前或父目录中查找，但它仍然需要一种方法来验证它找到的`configure`脚本是否确实是它自己。因此，`AC_CONFIG_SRCDIR`为`configure`提供了一个重要的提示，告诉它正在正确的位置查找。以下是`AC_CONFIG_SRCDIR`的原型：

```
AC_CONFIG_SRCDIR(unique-file-in-source-dir)
```

参数可以是任何源文件的路径（相对于项目的`configure`脚本）。你应该选择一个在你的项目中唯一的文件，以尽量减少`configure`被误导为其他项目的配置文件的可能性。我通常选择一个代表项目的文件，例如一个定义了项目特性的源文件。这样，即使我以后决定重新组织源代码，也不太可能因为文件重命名而丢失它。然而，在这种情况下，我们只有一个源文件，*main.c*，这使得遵循这个约定有点困难。无论如何，`autoconf`和`configure`都会告诉你和你的用户如果它找不到这个文件。

### 实例化宏

在深入讨论`AC_CONFIG_HEADERS`的细节之前，我想花点时间介绍一下 Autoconf 提供的文件生成框架。从一个高层次的角度来看，*configure.ac*中有四个主要的内容：

+   初始化

+   检查请求处理

+   文件实例化请求处理

+   生成`configure`脚本

我们已经覆盖了初始化——它并不复杂，尽管有一些宏你应该了解。更多信息可以查看*GNU Autoconf 手册*，查找`AC_COPYRIGHT`作为一个示例。现在让我们继续讨论文件实例化。

实际上有四个所谓的*实例化宏*：`AC_CONFIG_FILES`、`AC_CONFIG_HEADERS`、`AC_CONFIG_COMMANDS`和`AC_CONFIG_LINKS`。实例化宏接受标签或文件列表；`configure`将根据包含 Autoconf 替代变量的模板生成这些文件。

**注意**

*你可能需要在你的版本的 configure.scan 中将*`AC_CONFIG_HEADER`*（单数）更改为*`AC_CONFIG_HEADERS`*（复数）。单数版本是该宏的旧名称，旧宏比新宏功能少。^(17)*

这四个实例化宏具有一个有趣的共同签名。以下原型可用于表示它们中的每一个，适当的文本将替换宏名称中的*`XXX`*部分：

```
AC_CONFIG_XXXS(tag..., [commands], [init-cmds])
```

对于这四个宏中的每一个，标签参数的形式是*`OUT`*`[:`*`INLIST`*`]`，其中*`INLIST`*的形式是*`IN0`*`[:`*`IN1`*`:...:`*`INn`*`]`。通常，你会看到这些宏的调用只有一个参数，如下三个示例所示（请注意，这些示例表示宏*调用*，而不是*原型*，所以方括号实际上是 Autoconf 引号，而不是可选参数的指示）：

```
AC_CONFIG_HEADERS([config.h])
```

在这个示例中，*config.h*是前述规范中的*`OUT`*部分。*`INLIST`*的默认值是*`OUT`*部分并附加了*.in*。换句话说，前面的调用与以下内容完全等效：

```
AC_CONFIG_HEADERS([config.h:config.h.in])
```

这意味着`config.status`包含的 shell 代码将从*config.h.in*生成*config.h*，并在此过程中替换所有 Autoconf 变量。你还可以在*`INLIST`*部分提供一个输入文件列表。在这种情况下，*`INLIST`*中的文件将被连接起来形成结果*`OUT`*文件：

```
AC_CONFIG_HEADERS([config.h:cfg0:cfg1:cfg2])
```

在这里，`config.status`将通过连接*cfg0*、*cfg1*和*cfg2*（按此顺序），在替换所有 Autoconf 变量后生成*config.h*。*GNU Autoconf 手册*将这个完整的*`OUT`*`[:`*`INLIST`*`]`构造称为一个*标签*。

为什么不直接称它为*文件*呢？嗯，这个参数的主要用途是提供一种类似 makefile 目标名称的命令行目标名称。它还可以用作文件系统名称，如果相关宏生成文件的话，就像`AC_CONFIG_HEADERS`、`AC_CONFIG_FILES`和`AC_CONFIG_LINKS`那样。

但是`AC_CONFIG_COMMANDS`的独特之处在于它不会生成任何文件。相反，它运行用户在宏参数中指定的任意 shell 代码。因此，与其根据次要功能（文件生成）为这个第一个参数命名，*GNU Autoconf 手册*根据其主要用途，更一般地将其称为一个命令行*标签*，可以在`./config.status`命令行中指定，方式如下：

```
$ ./config.status config.h
```

这个命令将根据*configure.ac*中对`AC_CONFIG_HEADERS`宏的调用重新生成*config.h*文件。它*只会*重新生成*config.h*。

输入 `./config.status --help` 查看执行 `./config.status` 时可以使用的其他命令行选项：

```
   $ ./config.status --help
   `config.status' instantiates files and other configuration actions
   from templates according to the current configuration.    Unless the files
   and actions are specified as TAGs, all are instantiated by default.
➊ Usage: ./config.status [OPTION]... [TAG]...

    -h, --help       print this help, then exit
    -V, --version    print version number and configuration settings, then exit
     ➋ --config      print configuration, then exit
    -q, --quiet, --silent
                     do not print progress messages
    -d, --debug      don't remove temporary files
        --recheck    update config.status by reconfiguring in the same conditions
     ➌ --file=FILE[:TEMPLATE]
                     instantiate the configuration file FILE
        --header=FILE[:TEMPLATE]
                       instantiate the configuration header FILE

➍ Configuration files:
   Makefile src/Makefile

➎ Configuration headers:
   config.h

   Report bugs to <jupiter-bugs@example.org>.
   $
```

请注意，`config.status` 提供了有关项目的 `config.status` 文件的自定义帮助。它列出了我们可以在命令行中使用的配置文件 ➍ 和配置头文件 ➎，这些文件在用法中指定 `[TAG]...` 的位置 ➊。在这种情况下，`config.status` 只会实例化指定的对象。在命令的情况下，它将执行通过关联的 `AC_CONFIG_COMMANDS` 宏展开中传递的标签指定的命令集。

这些宏可以在 *configure.ac* 文件中多次使用。结果是累积的，我们可以在 *configure.ac* 中根据需要多次使用 `AC_CONFIG_FILES`。还需要注意的是，`config.status` 支持 `--file=` 选项（在 ➌ 位置）。当你在命令行中调用 `config.status` 时，唯一可以使用的标签是帮助文本中列出的可用配置文件、头文件、链接和命令。当你使用 `--file=` 选项执行 `config.status` 时，你是在告诉 `config.status` 生成一个新文件，该文件尚未与任何在 *configure.ac* 中找到的实例化宏调用关联。这个新文件是从一个关联的模板生成的，使用的是通过上次执行 `configure` 确定的配置选项和检查结果。例如，我可以通过以下方式执行 `config.status`（使用一个虚构的模板 *extra.in*）：

```
$ ./config.status --file=extra:extra.in
```

**注意**

*默认的模板名称是文件名加上 .in 后缀，因此这个调用可以在不使用 *`:extra.in`* 选项部分的情况下完成。我在这里添加它是为了更清晰地说明。*

最后，我想指出 `config.status` 的一个新特性——版本 2.65 的 Autoconf 添加了 `--config` 选项，在 ➋ 位置显示。使用此选项会显示传递给 `configure` 的显式配置选项。例如，假设我们以这种方式调用了 `./configure`：

```
$ ./configure --prefix=$HOME
```

当你使用新的 `--config` 选项时，`./config.status` 会显示以下内容：

```
$ ./config.status --config
'--prefix=/home/jcalcote'
```

**注意**

*较旧版本的 Autoconf 会生成一个 *`config.status`* 脚本，当使用 *`--version`* 选项时，它会显示这些信息，但它是更大一块文本的一部分。较新的 *`--config`* 选项使得查找和重用最初传递给 *`configure`* 脚本的配置选项变得更加容易。*

现在让我们回到第 102 页底部的实例化宏签名。我已经向你展示了 *`tag...`* 参数具有复杂的格式，但省略号表示它也代表多个标签，用空格分隔。你将在几乎所有 *configure.ac* 文件中看到的格式，如清单 4-14 所示。

```
AC_CONFIG_FILES([Makefile
                 src/Makefile
                 lib/Makefile
                 etc/proj.cfg])
```

*清单 4-14：在 `AC_CONFIG_FILES` 中指定多个标签（文件）*

这里的每一项都是一个标签规范，如果完全指定，应该类似于清单 4-15 中的调用。

```
AC_CONFIG_FILES([Makefile:Makefile.in
                 src/Makefile:src/Makefile.in
                 lib/Makefile:lib/Makefile.in
                 etc/proj.cfg:etc/proj.cfg.in])
```

*清单 4-15：在 `AC_CONFIG_FILES` 中完全指定多个标签*

回到实例化宏原型，有两个在这些宏中很少使用的可选参数：*`commands`* 和 *`init-cmds`*。*`commands`* 参数可用于指定一些任意的 shell 代码，这些代码将在 `config.status` 生成与标签相关的文件之前执行。通常不在文件生成实例化宏中使用此功能。你几乎总是会看到 *`commands`* 参数与 `AC_CONFIG_COMMANDS` 一起使用，因为默认情况下该宏不会生成任何文件，因为如果没有要执行的命令，调用此宏基本上是没有用的！^(18) 在这种情况下，*`tag`* 参数成为告诉 `config.status` 执行一组特定的 shell 命令的方法。

*`init-cmds`* 参数用于在 `config.status` 文件顶部初始化 shell 变量，变量值来自 *configure.ac* 和 `configure`。重要的是要记住，所有实例化宏的调用与 `config.status` 共享一个公共命名空间。因此，你应该尽量谨慎选择 shell 变量名，以减少它们与彼此之间以及与 Autoconf 生成的变量发生冲突的可能性。

旧有的谚语“画面胜于千言万语”在这里同样适用，因此让我们做一个小实验。创建一个包含 清单 4-16 内容的 *configure.ac* 文件的测试版本。你应该在一个单独的目录中执行此操作，因为在此实验中我们不依赖于 Jupiter 项目目录结构中的其他文件。

```
AC_INIT([test], [1.0])
AC_CONFIG_COMMANDS([abc],
                   [echo "Testing $mypkgname"],
                   [mypkgname=$PACKAGE_NAME])
AC_OUTPUT
```

*清单 4-16：实验 #1——使用 `AC_CONFIG_COMMANDS` 的简单* configure.ac *文件*

现在以不同的方式执行 `autoreconf`、`./configure` 和 `./config.status`，观察会发生什么：

```
   $ autoreconf
➊ $ ./configure
   configure: creating ./config.status
   config.status: executing abc commands
   Testing test
   $
➋ $ ./config.status
   config.status: executing abc commands
   Testing test
   $
➌ $ ./config.status --help
   'config.status' instantiates files from templates according to the current configuration.
   Usage: ./config.status [OPTIONS]... [FILE]...
   --snip--
   Configuration commands:
    abc

   Report bugs to <bug-autoconf@gnu.org>.
   $
➍ $ ./config.status abc
   config.status: executing abc commands
   Testing test
   $
```

如你在 ➊ 处看到的，执行 `./configure` 会导致 `config.status` 被执行且没有命令行选项。由于在 *configure.ac* 中没有指定检查，所以像我们在 ➋ 处做的那样手动执行 `./config.status`，几乎会产生相同的效果。查询 `config.status` 获取帮助（如我们在 ➌ 处所做的）表明 `abc` 是一个有效的标签；在命令行中执行带有该标签的 `./config.status`（如我们在 ➍ 处所做的）只是简单地运行关联的命令。

总结一下，关于实例化宏的关键点如下：

+   `config.status` 脚本从模板生成所有文件。

+   `configure` 脚本执行所有检查，然后执行 `./config.status`。

+   当你不带命令行选项执行 `./config.status` 时，它会根据最后一组检查结果生成文件。

+   你可以调用 `./config.status` 来执行任何实例化宏调用中指定的文件生成或命令集。

+   `config.status` 脚本可能会生成与 *configure.ac* 中任何标签未关联的文件，在这种情况下，它将基于最后一组检查结果替换变量。

#### *从模板生成头文件*

如你现在无疑已经得出结论，`AC_CONFIG_HEADERS` 宏允许你指定一个或多个头文件，让 `config.status` 从模板文件生成这些文件。配置头文件模板的格式非常具体。一个简短的示例见 列表 4-17。

```
/* Define as 1 if you have unistd.h. */
#undef HAVE_UNISTD_H
```

*列表 4-17：一个简短的头文件模板示例*

你可以在头文件模板中放置多条类似的语句，每行一条。当然，注释是可选的。让我们尝试另一个实验。创建一个新的 *configure.ac* 文件，如 列表 4-18 中所示。同样，你应该在一个隔离的目录中执行此操作。

```
AC_INIT([test], [1.0])
AC_CONFIG_HEADERS([config.h])
AC_CHECK_HEADERS([unistd.h foobar.h])
AC_OUTPUT
```

*列表 4-18：实验 #2——一个简单的* configure.ac *文件*

创建一个名为 *config.h.in* 的模板头文件，包含 列表 4-19 中的两行。

```
#undef HAVE_UNISTD_H
#undef HAVE_FOOBAR_H
```

*列表 4-19：实验 #2 继续——一个简单的* config.h.in *文件*

现在执行以下命令：

```
   $ autoconf
   $ ./configure
   checking for gcc... gcc
   --snip--
➊ checking for unistd.h... yes
   checking for unistd.h... (cached) yes
   checking foobar.h usability... no
   checking foobar.h presence... no
➋ checking for foobar.h... no
   configure: creating ./config.status
➌ config.status: creating config.h
   $
   $ cat config.h
   /* config.h.    Generated from config.h.in by configure.    */
   #define HAVE_UNISTD_H 1
➍ /* #undef HAVE_FOOBAR_H */
   $
```

你可以在 ➌ 看到，`config.status` 从我们编写的简单 *config.h.in* 模板生成了一个 *config.h* 文件。这个头文件的内容基于 `configure` 执行的检查。由于由 `AC_CHECK_HEADERS([unistd.h foobar.h])` 生成的 shell 代码能够在系统包含目录中找到 *unistd.h* 头文件（➊），因此相应的 `#undef` 语句被转换成了 `#define` 语句。当然，如你所见，系统包含目录中没有找到 *foobar.h* 头文件，正如在 ➋ 的 `./configure` 输出中所示；因此，它的定义被保留为注释，正如 ➍ 所示。

因此，你可以将 列表 4-20 中所示的代码添加到项目中适当的 C 语言源文件中。

```
#include "config.h"
#if HAVE_UNISTD_H
# include <unistd.h>
#endif
#if HAVE_FOOBAR_H
# include <foobar.h>
#endif
```

*列表 4-20：在 C 语言源文件中使用生成的 CPP 定义*

**注意**

*如今，`unistd.h` 头文件已经如此标准，以至于在 *`AC_CONFIG_HEADERS`* 中检查它其实不是必需的，但在这个示例中，它作为我确信在我的系统上存在的文件出现。*

#### *使用 autoheader 生成包含文件模板*

手动维护 *config.h.in* 模板比实际需要的麻烦。*config.h.in* 的格式非常严格——例如，`#undef` 行前后不能有空格，且你添加的 `#undef` 行必须使用 `#undef` 而不是 `#define`，主要是因为 `config.status` 只知道如何将 `#undef` 替换为 `#define`，或者注释掉包含 `#undef` 的行。^(19)

你从 *config.h.in* 中需要的大部分信息在 *configure.ac* 中都可以找到。幸运的是，`autoheader` 会根据 *configure.ac* 的内容为你生成一个格式正确的头文件模板，因此你通常不需要编写 *config.h.in* 模板。让我们回到命令提示符，进行一个最终实验。这个很简单——只需删除实验 #2 中的 *config.h.in* 模板，然后运行 `autoheader`，接着是 `autoconf`：

```
   $ rm config.h.in
   $ autoheader
   $ autoconf
   $ ./configure
   checking for gcc... gcc
   --snip--
   checking for unistd.h... yes
   checking for unistd.h... (cached) yes
   checking foobar.h usability... no
   checking foobar.h presence... no
   checking for foobar.h... no
   configure: creating ./config.status
   config.status: creating config.h
   $
➊ $ cat config.h
   /* config.h. Generated from config.h.in by configure.    */
   /* config.h.in. Generated from configure.ac by autoheader.    */
   /* Define to 1 if you have the <foobar.h> header file. */
   /* #undef HAVE_FOOBAR_H */
   --snip--
   /* Define to 1 if you have the <unistd.h> header file. */
   #define HAVE_UNISTD_H 1
   /* Define to the address where bug reports for this package should be sent. */
   #define PACKAGE_BUGREPORT ""
   /* Define to the full name of this package. */
   #define PACKAGE_NAME "test"
   /* Define to the full name and version of this package. */
   #define PACKAGE_STRING "test 1.0"
   /* Define to the one symbol short name of this package. */
   #define PACKAGE_TARNAME "test"
   /* Define to the version of this package. */
   #define PACKAGE_VERSION "1.0"
   /* Define to 1 if you have the ANSI C header files. */
   #define STDC_HEADERS 1
   $
```

**注意**

*再次，我鼓励你使用 *`autoreconf`*，如果它注意到 *`AC_CONFIG_HEADERS`* 在 configure.ac 中扩展，它将自动运行 *`autoheader`*。

如你所见，➊ 处 `cat` 命令的输出显示，`autoheader` 从 *configure.ac* 中派生出了一整套预处理器定义。

列表 4-21 展示了一个更实际的示例，说明如何使用生成的 *config.h* 文件来提高项目源代码的可移植性。在这个示例中，`AC_CONFIG_HEADERS` 宏的调用表示应该生成 *config.h*，而 `AC_CHECK_HEADERS` 的调用会使 `autoheader` 在 *config.h* 中插入一个定义。

```
AC_INIT([test], [1.0])
AC_CONFIG_HEADERS([config.h])
AC_CHECK_HEADERS([dlfcn.h])
AC_OUTPUT
```

*列表 4-21：使用 `AC_CONFIG_HEADERS` 的一个更实际的示例*

*config.h* 文件的目的是在你希望使用 C 预处理器在代码中测试已配置的选项时包含它。这个文件应当首先包含在源文件中，以便它可以影响后续系统头文件的包含。

**注意**

*`autoheader`* 生成的 *config.h.in* 模板不包含包含保护结构，因此你需要小心确保它不会在源文件中被包含多次。一个好的经验法则是始终将 *config.h* 作为每个 .c 源文件中第一个包含的头文件，并且不要在其他地方包含它。遵循这一规则将确保它永远不需要包含保护。

在项目中，通常每个 *.c* 文件都需要包含 *config.h*。在这种情况下，一个有趣的方法是使用 `gcc` 的 `-include` 选项，在编译器命令行中将其包含在每个编译的源文件顶部。这可以在 *configure.ac* 中通过将 `-include config.h` 添加到 `DEFS` 变量中来实现（当前该变量仅用于定义 `HAVE_CONFIG_H`——如果你更喜欢简洁的方法，可以改用 `CFLAGS`）。完成后，你可以假设 *config.h* 是每个翻译单元的一部分。

如果你的项目将库和头文件作为产品的一部分进行安装，千万不要犯将 *config.h* 包含在公共头文件中的错误。有关此主题的更详细信息，请参见 第 499 页 的“项 1：将私有细节排除在公共接口之外”。

使用示例 4-21 中的*configure.ac*文件，生成的`configure`脚本将创建一个*config.h*头文件，并根据编译时的判断来确定当前系统是否提供`dlfcn`接口。为了完成可移植性检查，你可以将示例 4-22 中的代码添加到你的项目源文件中，来使用动态加载器功能。

```
   #include "config.h"
➊ #if HAVE_DLFCN_H
   # include <dlfcn.h>
   #else
   # error Sorry, this code requires dlfcn.h.
   #endif
   --snip--
➋ #if HAVE_DLFCN_H
       handle = dlopen("/usr/lib/libwhatever.so", RTLD_NOW);
   #endif
   --snip--
```

*示例 4-22：检查动态加载器功能的示例源文件*

如果你已经有了包含*dlfcn.h*的代码，`autoscan`将会在*configure.ac*中生成一行，调用`AC_CHECK_HEADERS`，并在其参数列表中包含*dlfcn.h*，作为要检查的头文件之一。作为维护者，你的任务是在现有的*dlfcn.h*头文件的包含部分以及调用*dlfcn*接口函数的部分周围添加条件语句，标记为➊和➋。这是 Autoconf 可移植性支持的关键。

**注意**

*如果你选择在包含检查失败时“报错”，技术上你不需要预处理器条件语句围绕代码，但这样做会使读者明显看到哪些部分的源代码受到了条件包含的影响。*

你的项目可能更倾向于使用动态加载器功能，但在必要时也能没有它。也有可能你的项目确实需要一个动态加载器，在这种情况下，如果缺少关键功能，构建应该终止并报错（如此代码所示）。通常，这是一个可接受的临时解决方案，直到有人来为源代码添加更系统特定的动态加载器服务支持。

**注意**

*如果你必须在配置时就报错，最好在配置时而不是编译时处理。一般的做法是尽早退出。*

如前所述，`HAVE_CONFIG_H`是一个由 Autoconf 替换变量`@DEFS@`传递给编译器命令行的一系列定义的一部分。在`autoheader`和`AC_CONFIG_HEADERS`功能出现之前，Automake 将所有的编译器配置宏添加到`@DEFS@`变量中。如果你没有在*configure.ac*中使用`AC_CONFIG_HEADERS`，你仍然可以使用这种方法，但不推荐这样做——主要是因为大量的定义会导致编译器命令行非常长。

### 回到远程构建的话题

当我们结束这一章时，你会注意到我们已经走了一圈。我们一开始介绍了一些初步的信息，然后讨论了如何将远程构建添加到 Jupiter 中。现在我们将暂时回到这个话题，因为我还没有讲解如何让 C 预处理器正确地找到生成的*config.h*文件。

由于此文件是从模板生成的，因此它在构建目录结构中的相对位置将与其对应模板文件*config.h.in*在源代码目录结构中的位置相同。模板位于顶层*source*目录（除非你选择将其放在其他地方），因此生成的文件将位于顶层*build*目录。嗯，这很简单——它总是比生成的*src/Makefile*高一级。

在我们对头文件位置做出任何结论之前，先考虑一下头文件可能出现在项目中的位置。我们可能会在当前的构建目录中生成它们，作为构建过程的一部分。我们也可能将内部头文件添加到当前的源代码目录中。我们知道在顶层构建目录中有一个*config.h*文件。最后，我们还可能为我们的包提供的库接口头文件创建一个顶层*include*目录。这些不同的*include*目录的优先级顺序是什么？

我们在编译器命令行中放置*include 指令*（`-I`*`path`*选项）的顺序，就是它们被搜索的顺序，因此顺序应该基于与当前正在编译的源文件最相关的文件。因此，编译器命令行应首先包含当前构建目录（`.`）的`-I`*`path`*指令，然后是源代码目录[`$(srcdir)`]，接着是顶层构建目录（`..`），最后是我们的项目的*include*目录（如果有的话）。我们通过在编译器命令行中添加`-I`*`path`*选项来强制执行此顺序，如清单 4-23 所示。

Git 标签 4.7

```
--snip--
jupiter: main.c
        $(CC) $(CPPFLAGS) $(CFLAGS) -I. -I$(srcdir) -I.. -o $@ \
          $(srcdir)/main.c
--snip--
```

*清单 4-23：* src/Makefile.in: *添加正确的编译器包含指令*

现在我们知道了这一点，我们需要向我们在第 92 页上创建的列表中添加另一个远程构建的经验法则：

+   按照顺序添加当前构建目录、相关源代码目录和顶层构建目录（或者如果*config.h.in*位于其他地方，则是其他构建目录）的预处理器命令。

### 总结

在本章中，我们涵盖了一个完全功能的 GNU 项目构建系统几乎所有主要特性，包括编写*configure.ac*文件，Autoconf 从中生成一个完全功能的`configure`脚本。我们还涵盖了如何通过`VPATH`语句向 makefile 添加远程构建功能。

那还有什么呢？当然有！在下一章，我将继续向你展示如何使用 Autoconf 在用户运行`make`之前测试系统特性和功能。我们还将继续增强配置脚本，以便当我们完成时，用户将有更多选项，并且完全理解我们的包将如何在他们的系统上构建。
