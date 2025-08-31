## 第五章：更有趣的 Autoconf：配置用户选项

*希望不是确信某事会顺利发展，而是确信某事有意义，无论结果如何。*

—瓦茨拉夫·哈维尔*，《打破和平》

![图片](img/common.jpg)

在第四章中，我们讨论了 Autoconf 的基本内容——如何启动一个新的或现有的项目，以及如何理解一些*configure.ac*文件的基本方面。在本章中，我们将介绍一些更复杂的 Autoconf 宏。我们将从讨论如何将自定义变量替换到模板文件（例如*Makefile.in*）中开始，并介绍如何在配置脚本中定义我们自己的预处理器定义。在本章中，我们将继续通过添加重要的检查和测试来开发 Jupiter 项目的功能。我们将涵盖至关重要的`AC_OUTPUT`宏，最后将讨论如何应用在*configure.ac*文件中指定的用户定义的项目配置选项。

除此之外，我将介绍一种分析技巧，可以帮助你解读宏的内部工作原理。以稍微复杂的`AC_CHECK_PROG`宏为例，我将向你展示一些了解其工作原理的方法。

### 替换和定义

我们将通过讨论 Autoconf 套件中三个最重要的宏开始本章内容：`AC_SUBST`和`AC_DEFINE`，以及后者的兄弟宏`AC_DEFINE_UNQUOTED`。

这些宏提供了配置过程与构建和执行过程之间的主要通信机制。*替换*到生成文件中的值为构建过程提供配置信息，而在预处理器变量中定义的值则在构建时为编译器提供配置信息，并在运行时为构建的程序和库提供配置信息。因此，深入了解`AC_SUBST`和`AC_DEFINE`是非常值得的。

#### *AC_SUBST*

你可以使用`AC_SUBST`扩展 Autoconf 中作为核心部分的变量替换功能。每个与替换变量相关的 Autoconf 宏，最终都会调用这个宏，从现有的 Shell 变量中创建替换变量。有时这些 Shell 变量来自环境变量；有时，较高级的宏会在调用`AC_SUBST`之前，作为其功能的一部分设置这些 Shell 变量。这个宏的签名相对简单（注意，该原型中的方括号表示可选参数，而不是 Autoconf 的引用）：

```
AC_SUBST(shell_var[, value])
```

**注意**

*如果在调用 M4 宏时选择省略任何尾随的可选参数，你也可以省略尾随的逗号。^(1)然而，如果你省略了中间部分的任何参数，必须为缺失的参数提供逗号作为占位符。*

第一个参数，*`shell_var`*，表示一个 shell 变量，其值将被替换到通过 `config.status` 从模板生成的所有文件中。可选的第二个参数是赋给该变量的值。如果没有指定，shell 变量的当前值将被使用，无论它是继承的还是由某些之前的 shell 代码设置的。

替代变量将与 shell 变量具有相同的名称，只是在模板文件中，它将被 `@` 符号括起来。因此，一个名为 `my_var` 的 shell 变量将变成替代变量引用 `@my_var@`，你可以在任何模板文件中使用它。

在 *configure.ac* 中调用 `AC_SUBST` 不应具有条件性；也就是说，它们不应在类似 `if`-`then`-`else` 结构的条件 shell 语句中被调用。当你仔细考虑 `AC_SUBST` 的目的时，这一点变得清晰：你已经将替代变量引用硬编码到模板文件中，因此你最好对每个变量无条件地使用 `AC_SUBST`，否则输出文件将保留变量引用，而不是应该被替换的值。

#### *AC_DEFINE*

`AC_DEFINE` 和 `AC_DEFINE_UNQUOTED` 宏定义 C 预处理器宏，这些宏可以是简单的宏或类似函数的宏。这些宏要么在 *config.h.in* 模板中定义（如果你使用 `AC_CONFIG_HEADERS`），要么通过 *Makefile.in* 模板中的 `@DEFS@` 替代变量传递给编译器命令行。回想一下，如果你没有自己编写 *config.h.in*，`autoheader` 会根据你在 *configure.ac* 文件中调用这些宏的情况自动生成它。

这两个宏名称实际上代表四个不同的 Autoconf 宏。以下是它们的原型：

```
AC_DEFINE(variable, value[, description])
AC_DEFINE(variable)
AC_DEFINE_UNQUOTED(variable, value[, description])
AC_DEFINE_UNQUOTED(variable)
```

这些宏的正常版本与 `UNQUOTED` 版本的区别在于，正常版本原样使用指定的值作为预处理器宏的值。`UNQUOTED` 版本对 *`value`* 参数进行 shell 扩展，并使用结果作为预处理器宏的值。因此，如果值包含你希望 `configure` 扩展的 shell 变量，应该使用 `AC_DEFINE_UNQUOTED`。（在头文件中将 C 预处理器宏设置为未扩展的 shell 变量没有意义，因为 C 编译器或预处理器在编译源代码时都不知道该如何处理它。）

单参数版本和多参数版本的区别在于预处理器宏的定义方式。单参数版本仅保证宏在预处理器命名空间中被*定义*，而多参数版本确保宏以特定的值被定义。

可选的第三个参数，*`description`*，告诉`autoheader`在*config.h.in*模板中为此宏添加注释。（如果你不使用`autoheader`，传递描述就没有意义——因此，它是可选的。）如果你希望定义一个没有值的预处理器宏并提供*`description`*，你应该使用这些宏的多参数版本，但将`value`参数留空。另一种选择是使用`AH_TEMPLATE`——一个特定于`autoheader`的宏——它在给定*`description`*但不需要*`value`*时，与`AC_DEFINE`做相同的事情。

### 检查编译器

`AC_PROG_CC`宏确保用户系统中有一个有效的 C 语言编译器。以下是这个宏的原型：

```
AC_PROG_CC([compiler-search-list])
```

如果你的代码需要特定类型或品牌的 C 编译器，你可以在这个参数中传递一个由空格分隔的程序名称列表。例如，如果你使用`AC_PROG_CC([cc cl gcc])`，该宏会扩展为在 shell 代码中搜索`cc`、`cl`和`gcc`，按此顺序进行。通常，可选的参数会被省略，允许宏找到用户系统中最佳的编译器选项。

你会记得在“通过`autoscan`更快开始”中提到的内容，位于第 95 页，当`autoscan`在目录树中发现 C 源文件时，它会在 Jupiter 的*configure.scan*文件中插入一个无参数调用该宏的命令。列表 5-1 复现了生成的*configure.scan*文件中的相关部分。

```
--snip--
# Checks for programs.
AC_PROG_CC
AC_PROG_INSTALL
--snip--
```

*列表 5-1:* configure.scan：*检查编译器和其他程序*

**注意**

*如果 Jupiter 目录树中的源文件后缀是* .cc, .cxx, *或* .C *（这些都是常见的 C++源文件扩展名），*`autoscan`*会改为插入对*`AC_PROG_CXX`*的调用。*

`AC_PROG_CC`宏在系统搜索路径中查找`gcc`，然后是`cc`。如果它没有找到任何一个，它会继续查找其他 C 编译器。当它找到一个兼容的编译器时，宏会设置一个已知的变量`CC`，并根据需要提供移植性选项，除非用户已经在环境变量或`configure`命令行中设置了`CC`。

`AC_PROG_CC`宏还定义了以下 Autoconf 替换变量，其中一些你可能会认作是*用户变量*（在表格 3-2 中的第 71 页列出）：

+   `@CC@`（编译器的完整路径）

+   `@CFLAGS@`（例如，`-g -O2`用于`gcc`）

+   `@CPPFLAGS@`（默认空）

+   `@EXEEXT@`（例如，*.exe*）

+   `@OBJEXT@`（例如，*o*）^(2)

`AC_PROG_CC` 配置这些替换变量，但除非你在你的*Makefile.in*模板中使用它们，否则你只是浪费时间运行`./configure`。方便的是，我们已经在我们的*Makefile.in*模板中使用了它们，因为在 Jupiter 项目的早期，我们将它们添加到了我们的编译命令行中，并为`CFLAGS`添加了一个默认值，用户可以在`make`命令行中覆盖它。

唯一剩下的事情是确保`config.status`为这些变量引用进行替换。列表 5-2 显示了*src*目录*Makefile.in*模板的相关部分以及使这一切发生所需的更改。

Git 标签 5.0

```
--snip--
# VPATH-specific substitution variables
srcdir = @srcdir@
VPATH = @srcdir@

# Tool-specific substitution variables
CC = @CC@
CFLAGS = @CFLAGS@
CPPFLAGS = @CPPFLAGS@

all: jupiter

jupiter: main.c
        $(CC) $(CPPFLAGS) $(CFLAGS) -I. -I$(srcdir) -I.. -o $@ $(srcdir)/main.c
--snip--
```

*列表 5-2:* src/Makefile.in: *使用 Autoconf 编译器和标志替换变量*

### 检查其他程序

紧接在调用`AC_PROG_CC`之后（请参见列表 5-1）是调用`AC_PROG_INSTALL`。所有的`AC_PROG_*`宏都会设置（然后使用`AC_SUBST`进行替换）指向已定位工具的各种环境变量。`AC_PROG_INSTALL`为`install`工具做了相同的事情。要使用此检查，你需要在你的*Makefile.in*模板中使用相关的 Autoconf 替换变量，就像我们之前对`@CC@`、`@CFLAGS@`和`@CPPFLAGS@`所做的那样。列表 5-3 展示了这些更改。

Git 标签 5.1

```
--snip--
# Tool-specific substitution variables
CC = @CC@
CFLAGS = @CFLAGS@
CPPFLAGS = @CPPFLAGS@
INSTALL = @INSTALL@
INSTALL_DATA = @INSTALL_DATA@
INSTALL_PROGRAM = @INSTALL_PROGRAM@
INSTALL_SCRIPT = @INSTALL_SCRIPT@
--snip--
install:
        $(INSTALL) -d $(DESTDIR)$(bindir)
 $(INSTALL_PROGRAM) -m 0755 jupiter $(DESTDIR)$(bindir)
--snip--
```

*列表 5-3:* src/Makefile.in: *在你的 Makefile.in 模板中替换 `install` 实用程序*

`@INSTALL@`的值显然是已定位的安装程序的路径。`@INSTALL_DATA@`的值是`${INSTALL} -m 0644`。基于此，你可能会认为`@INSTALL_PROGRAM@`和`@INSTALL_SCRIPT@`的值会是`${INSTALL} -m 0755`之类的，但事实并非如此。这些值被简单地设置为`${INSTALL}`。^(3)

你可能还需要检查其他重要的实用程序，包括`lex`、`yacc`、`sed`和`awk`。如果你的程序需要这些工具中的一个或多个，你可以添加`AC_PROG_LEX`、`AC_PROG_YACC`、`AC_PROG_SED`或`AC_PROG_AWK`的调用。如果它在你项目的目录树中检测到带有*.yy*或*.ll*扩展名的文件，`autoscan`将会将`AC_PROG_YACC`和`AC_PROG_LEX`的调用添加到*configure.scan*中。

你可以使用这些更专业的宏检查大约十几种不同的程序。如果程序检查失败，生成的`configure`脚本将失败，并显示一条消息，指示无法找到所需的工具，并且在工具正确安装之前，构建无法继续。

程序和编译器检查会导致 `autoconf` 将特别命名的变量替换到模板文件中。你可以在 *GNU Autoconf Manual* 中找到每个宏的变量名称。你应该在 *Makefile.in* 模板中的命令中使用这些 `make` 变量来调用它们代表的工具。Autoconf 宏会根据用户系统上安装的工具设置这些变量的值，*如果用户尚未在环境中设置它们*。

这是 Autoconf 生成的 `configure` 脚本中的一个关键方面——用户可以 *始终* 通过在执行 `configure` 之前导出或设置适当的变量来覆盖 `configure` 对环境所做的任何更改。^(4)

例如，如果用户选择使用安装在主目录中的特定版本的 `bison` 构建，可以输入以下命令，以确保 `$(YACC)` 引用正确版本的 `bison`，并且 shell 代码 `AC_PROG_YACC` 生成的内容仅仅是将现有的 `YACC` 值替换为你 *Makefile.in* 模板中的 `@YACC@`：

```
$ cd jupiter
$ ./configure YACC="$HOME/bin/bison -y"
--snip--
```

**注意**

*将变量设置作为参数传递给*`configure`*的功能类似于在 shell 环境中通过命令行为*`configure`*进程设置变量（例如，*`YACC="$HOME/bin/bison -y" ./configure`*）。使用此示例中给出的语法的优点是，*`config.status --recheck`* 可以跟踪该值，并通过最初传递给它的选项正确地重新执行 *`configure`*。因此，你应该始终使用参数语法，而不是 shell 环境语法，来为 *`configure`* 设置变量。有关强制使用此语法的方法，请参阅 Autoconf 手册中 *`AC_ARG_VAR`* 的文档。

要检查某个程序是否存在，如果该程序未被这些更专用的宏覆盖，你可以使用通用的 `AC_CHECK_PROG` 宏，或编写你自己的特定用途宏（参见第十六章）。

这里需要记住的关键点如下：

+   `AC_PROG_*` 宏用于检查程序是否存在。

+   如果找到程序，则会创建一个替代变量。

+   你应该在 *Makefile.in* 模板中使用这些替代变量来执行相关的工具。

### Autoconf 中的一个常见问题

我们应借此机会解决开发者在使用 Autotools 时常遇到的一个问题。以下是 *GNU Autoconf Manual* 中给出的 `AC_CHECK_PROG` 的正式定义：

```
AC_CHECK_PROG(variable, prog-to-check-for, value-if-found,
    [value-if-not-found], [path], [reject])
```

检查程序`prog-to-check-for`是否存在于*`path`*中。如果找到，设置*`variable`*为*`value-if-found`*，否则设置为*`value-if-not-found`*，如果给定的话。即使*`reject`*（一个绝对路径文件名）在搜索路径中最先找到，也总是跳过它；在这种情况下，使用找到的不是*`reject`*的*`prog-to-check-for`*的绝对路径名来设置*`variable`*。如果*`variable`*已经设置，则不做任何操作。调用*`AC_SUBST`*为*`variable`*。此测试的结果可以通过设置*`variable`*变量或缓存变量*`ac_cv_prog_variable`*来覆盖。^(5)

这段话比较复杂，但仔细阅读后，你可以从中提取以下内容：

+   如果在系统搜索路径中找到*`prog-to-check-for`*，则*`variable`*被设置为*`value-if-found`*；否则，设置为*`value-if-not-found`*。

+   如果指定了*`reject`*（作为完整路径），并且它与在前一步中在系统搜索路径中找到的程序相同，则跳过它并继续搜索系统搜索路径中的下一个匹配程序。

+   如果*`reject`*首先在*`path`*中找到，然后找到另一个匹配项（不同于*`reject`*），则将*`variable`*设置为第二个（非*`reject`*）匹配项的绝对路径名。

+   如果用户已经在环境中设置了*`variable`*，则*`variable`*保持不变（从而允许用户在运行`configure`之前通过设置*`variable`*来覆盖检查）。

+   调用`AC_SUBST`以使*`variable`*成为 Autoconf 替代变量。

初读这段描述时，似乎存在冲突：在第一项中，我们看到如果在系统搜索路径中找到*`prog-to-check-for`*，*`variable`*将被设置为两个指定值之一。但随后我们在第三项中看到，如果首先找到并跳过*`reject`*，则*`variable`*将被设置为某个程序的完整路径。

发现`AC_CHECK_PROG`的真实功能就像读一个小的 shell 脚本一样简单。虽然你可以参考 Autoconf 的*programs.m4*宏文件中对`AC_CHECK_PROG`的定义，但那时你会离执行检查的实际 shell 代码有一层隔离。直接查看`AC_CHECK_PROG`生成的 shell 脚本岂不是更好？我们将使用 Jupiter 的*configure.ac*文件来玩这个概念。暂时根据列表 5-4 中突出显示的更改修改你的*configure.ac*文件。

```
--snip--
AC_PREREQ(2.69)
AC_INIT([Jupiter], [1.0], [jupiter-bugs@example.org])
AC_CONFIG_SRCDIR([src/main.c])
AC_CONFIG_HEADER([config.h])

# Checks for programs.
AC_PROG_CC
_DEBUG_START_
AC_CHECK_PROG([bash_var], [bash], [yes], [no],, [/usr/sbin/bash])
_DEBUG_END_
AC_PROG_INSTALL
--snip--
```

*列表 5-4：首次尝试使用`AC_CHECK_PROG`*

现在执行`autoconf`，打开生成的`configure`脚本，并搜索`_DEBUG_START_`。

**注意**

*`_DEBUG_START_`* 和 *`_DEBUG_END_`* 字符串被称为栅栏。我将它们添加到 *configure.ac* 中，唯一目的是帮助我找到由 *`AC_CHECK_PROG`* 宏生成的 shell 代码的开始和结束位置。我特意选择这些名称，因为在生成的 *`configure`* 脚本中，你不太可能找到它们。^(6)

列表 5-5 显示了该宏生成的 `configure` 代码部分。

```
   --snip--
   _DEBUG_START_
➊ # Extract the first word of "bash" so it can be a program name with args.
   set dummy bash; ac_word=$2
   { $as_echo "$as_me:${as_lineno-$LINENO}: checking for $ac_word" >&5
   $as_echo_n "checking for $ac_word... " >&6; }
   if ${ac_cv_prog_bash_var+:} false; then :
     $as_echo_n "(cached) " >&6
   else
     if test -n "$bash_var"; then
     ac_cv_prog_bash_var="$bash_var" # Let the user override the test.
   else
     ac_prog_rejected=no
   as_save_IFS=$IFS; IFS=$PATH_SEPARATOR
   for as_dir in $PATH
   do
     IFS=$as_save_IFS
     test -z "$as_dir" && as_dir=.
       for ac_exec_ext in '' $ac_executable_extensions; do
     if as_fn_executable_p "$as_dir/$ac_word$ac_exec_ext"; then
     ➋ if test "$as_dir/$ac_word$ac_exec_ext" = "/usr/sbin/bash"; then
           ac_prog_rejected=yes
           continue
         fi
        ac_cv_prog_bash_var="yes"
        $as_echo "$as_me:${as_lineno-$LINENO}: found $as_dir/$ac_word$ac_exec_ext"
    >&5
        break 2
      fi
    done
      done
  IFS=$as_save_IFS

➌ if test $ac_prog_rejected = yes; then
    # We found a bogon in the path, so make sure we never use it.
    set dummy $ac_cv_prog_bash_var
    shift
    if test $# != 0; then
      # We chose a different compiler from the bogus one.
      # However, it has the same basename, so the bogon will be chosen
      # first if we set bash_var to just the basename; use the full file name.
      shift
      ac_cv_prog_bash_var="$as_dir/$ac_word${1+' '}$@"
    fi
  fi
    test -z "$ac_cv_prog_bash_var" && ac_cv_prog_bash_var="no"
  fi
  fi
  bash_var=$ac_cv_prog_bash_var
  if test -n "$bash_var"; then
    { $as_echo "$as_me:${as_lineno-$LINENO}: result: $bash_var" >&5
  $as_echo "$bash_var" >&6; }
  else
       { $as_echo "$as_me:${as_lineno-$LINENO}: result: no" >&5
  $as_echo "no" >&6; }
  fi

  _DEBUG_END_
  --snip--
```

*列表 5-5：由 `AC_CHECK_PROG` 生成的 `configure` 代码部分*

在此 shell 脚本的 ➊ 位置，开头的注释提示 `AC_CHECK_PROG` 具有一些未记录的功能。显然，你可以在 *`prog-to-check-for`* 参数中传递参数和程序名称。稍后，我们将查看一种可能需要这样做的情况。

在脚本的 ➋ 位置，你可以看到 *`reject`* 参数被添加进来，以便让 `configure` 搜索特定版本的工具。从 ➌ 位置的代码中，我们可以看到我们的 `bash_var` 变量可能有三种不同的值：如果请求的程序在搜索路径中未找到，则为空；如果找到了指定的程序，则为该程序；如果 *`reject`* 首先被找到，则为指定程序的完整路径。

*`reject`* 是在哪里使用的呢？例如，在安装了专有 Sun 工具的 Solaris 系统上，默认的 C 编译器通常是 Solaris C 编译器。但是某些软件可能需要使用 GNU C 编译器。作为维护者，我们不知道哪个编译器会在用户的搜索路径中首先找到。`AC_CHECK_PROG` 允许我们确保如果搜索路径中首先找到其他 C 编译器，`gcc` 会被使用，并且会提供完整的路径。

正如我之前提到的，M4 宏会意识到传递的参数是给定的、为空的，还是缺失的，并根据这些条件执行不同的操作。许多标准的 Autoconf 宏被编写成充分利用空的或未指定的可选参数，并在每种条件下生成完全不同的 shell 代码。Autoconf 宏还可以根据这些不同的条件优化生成的 shell 代码。

根据我们现在知道的内容，我们可能应该改为这样调用 `AC_CHECK_PROG`：

```
AC_CHECK_PROG([bash_shell],[bash -x],[bash -x],,,[/usr/sbin/bash])
```

你可以从这个例子中看到手册在技术上是准确的。如果没有指定*`reject`*，并且系统路径中找到了`bash`，那么`bash_shell`将被设置为`bash -x`。如果系统路径中*没有*找到`bash`，那么`bash_shell`将被设置为空字符串。另一方面，如果*`reject`* *被*指定，并且在路径中首先找到不想要的`bash`版本，那么`bash_shell`将被设置为路径中找到的*下一个*版本的完整路径，并带上最初指定的参数（`-x`）。宏之所以在这种情况下使用完整路径，是为了确保`configure`避免执行路径中首先找到的版本——*`reject`*。接下来的配置脚本可以使用`bash_shell`变量来运行所需的 Bash shell，只要它不为空。

**注意**

*如果你在自己的代码中跟着一起操作，别忘了从你的* configure.ac *文件中移除清单 5-4 中的临时代码*。

### 库和头文件检查

是否在项目中使用外部库是一个艰难的决定。一方面，你希望重用现有代码来提供所需的功能，而不是自己编写。重用是开源软件世界的一个标志。另一方面，你不想依赖那些在所有目标平台上可能不存在，或者可能需要进行大量移植才能使你需要的库在所需位置可用的功能。

偶尔，基于库的功能在不同平台之间可能会有所不同。尽管这些功能在本质上是等效的，但库的包名或 API 签名可能会有所不同。例如，POSIX 线程库（*pthread*）在功能上类似于许多本地线程库，但这些库的 API 通常会有一些微小的差异，而且它们的包名和库名几乎总是不同。假设我们尝试在一个不支持*pthread*的系统上构建一个多线程项目，考虑一下会发生什么；在这种情况下，你可能想在 Solaris 上使用*libthreads*库。

Autoconf 库选择宏允许生成的配置脚本智能地选择提供必要功能的库，即使这些库在不同平台之间的名称不同。为了说明 Autoconf 库选择宏的使用，我们将为 Jupiter 项目添加一些微不足道（而且相当牵强的）多线程功能，使得`jupiter`能够使用后台线程打印其信息。我们将使用*pthread* API 作为我们的基础线程模型。为了通过基于 Autoconf 的配置脚本实现这一点，我们需要将*pthread*库添加到我们的项目构建系统中。

**注意**

*正确使用多线程需要定义额外的替代变量，这些变量包含适当的标志、库和定义。*`AX_PTHREAD`* 宏会为你完成所有这些工作。你可以在 Autoconf 宏库网站上找到 *`AX_PTHREAD`* 的文档。^(7) 请参阅 第 384 页中的“正确使用线程”章节，了解如何使用 *`AX_PTHREAD`* 的示例。*

首先，让我们解决源代码的更改。我们将修改 *main.c*，使消息由一个辅助线程打印，正如在 列表 5-6 中所示。

Git 标签 5.2

```
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

static void * print_it(void * data)
{
    printf("Hello from %s!\n", (const char *)data);
    return 0;
}

int main(int argc, char * argv[])
{
    pthread_t tid;
    pthread_create(&tid, 0, print_it, argv[0]);
    pthread_join(tid, 0);
    return 0;
}
```

*列表 5-6:* src/main.c: *向 Jupiter 项目源代码中添加多线程*

这显然是一个荒谬的线程使用方式；然而，它 *确实* 是线程使用的典型形式。考虑一个假设的情况，其中后台线程执行一些长时间的计算，而 `main` 正在做其他事情，同时 `print_it` 也在工作。在多处理器机器上，以这种方式使用线程可以将程序的吞吐量翻倍。

现在，我们只需要一种方法来确定应该将哪些库添加到编译器（链接器）命令行中。如果我们不使用 Autoconf，我们只需将库添加到 makefile 中的链接器命令行中，正如在 列表 5-7 中所示。

```
program: main.c
        $(CC) ... -lpthread ...
```

*列表 5-7：手动将 pthread 库添加到编译命令行中*

相反，我们将使用 Autoconf 提供的 `AC_SEARCH_LIBS` 宏，这是基础的 `AC_CHECK_LIB` 宏的增强版。`AC_SEARCH_LIBS` 宏允许我们在一个库列表中测试所需的功能。如果某个指定的库中存在所需功能，合适的命令行选项会被添加到 `@LIBS@` 替代变量中，然后我们会在 *Makefile.in* 模板中使用该变量，将其添加到编译器（链接器）命令行中。以下是来自 *GNU Autoconf 手册* 的 `AC_SEARCH_LIBS` 宏的正式定义：

```
AC_SEARCH_LIBS(function, search-libs,
    [action-if-found], [action-if-not-found], [other-libraries])
```

如果库中还没有定义 *`function`*，则搜索一个定义该函数的库。这相当于首先调用 `AC_LINK_IFELSE([AC_LANG_CALL([],` `[`*`function`*`])])`，并且不使用任何库，然后对每个在 *`search-libs`* 中列出的库进行操作。

将 `-l`*`library`* 添加到 `LIBS` 中，以便为找到的第一个包含 *`function`* 的库添加链接，并运行 *`action-if-found`*。如果未找到 *`function`*，则运行 *`action-if-not-found`*。

如果与 *`library`* 链接时出现未解析的符号，而这些符号通过与额外库的链接可以解析，则将这些库作为 *`other-libraries`* 参数传递，多个库之间用空格分隔：例如 `-lXt -lX11`。否则，这个宏将无法检测到 *`function`* 是否存在，因为测试程序始终由于未解析的符号而无法链接。

这个测试的结果被缓存到`ac_cv_search` *`function`*变量中，如果*`function`*已经可用，则为`none required`，如果未找到包含*`function`*的库，则为`no`，否则为需要添加到`LIBS`前缀的`-l`*`library`*选项。^(8)

你能看出为什么生成的配置脚本如此庞大吗？当你在调用`AC_SEARCH_LIBS`时传递一个特定的函数，链接器命令行参数会被添加到一个名为`@LIBS@`的替代变量中。这些参数确保你将链接到包含传递函数的库。如果第二个参数中列出了多个库，并且由空格分隔，`configure`将确定这些库中哪些在用户的系统上可用，并使用最合适的一个。

示例 5-8 展示了如何在 Jupiter 的*configure.ac*文件中使用`AC_SEARCH_LIBS`来查找包含`pthread_create`函数的库。如果`AC_SEARCH_LIBS`没有在*pthread*库中找到`pthread_create`，它将不会向`@LIBS@`变量添加任何内容。

```
--snip--
# Checks for libraries.
AC_SEARCH_LIBS([pthread_create], [pthread])
--snip--
```

*示例 5-8:* configure.ac: *使用`AC_SEARCH_LIBS`检查系统上的*pthread*库*

正如我们在第七章中将详细讨论的那样，不同系统的库命名规则各不相同。例如，一些系统将库命名为*lib*basename*.so*，而其他系统使用*lib*basename*.sa*或*lib*basename*.a*。基于 Cygwin 的系统生成命名为*cig*basename*.dll*的库。`AC_SEARCH_LIBS`通过使用编译器计算库的实际名称来优雅地解决了这个问题；它通过尝试将一个小的测试程序与测试库中的请求函数链接来实现这一点。编译器命令行上只传递`-l`*`basename`*，这是 Unix 编译器之间的一个几乎通用的约定。

我们将不得不再次修改*src/Makefile.in*，以便正确使用现在已填充的`@LIBS@`变量，正如示例 5-9 所示。

```
--snip--
# Tool-specific substitution variables
CC = @CC@
LIBS = @LIBS@
CFLAGS = @CFLAGS@
CPPFLAGS = @CPPFLAGS@
--snip--
jupiter: main.c
        $(CC) $(CFLAGS) $(CPPFLAGS) -I. -I$(srcdir) -I..\
          -o $@ $(srcdir)/main.c $(LIBS)
--snip--
```

*示例 5-9:* src/Makefile.in: *使用`@LIBS@`替代变量*

**注意**

*我在编译器命令行上的源文件之后添加了*`$(LIBS)`*，因为链接器关心目标文件的顺序——它会按命令行上指定的顺序搜索文件中的必需函数。*

我希望*main.c*成为`jupiter`的主要目标代码来源，所以我会继续将其他目标文件，包括库，添加到这个文件之后的命令行中。

#### *这对吗，还是只是够好？*

到目前为止，我们已经确保我们的构建系统能在大多数系统上正确使用*pthread*。^(9) 如果我们的系统需要特定的库，该库的名称将被添加到`@LIBS@`变量中，然后在编译器命令行上使用。但我们还没有完成。

这个系统*通常*工作得很好，但在某些极端情况下可能会失败。因为我们希望提供卓越的用户体验，所以我们将把 Jupiter 的构建系统提升到一个新的水平。在此过程中，我们需要做出一个设计决策：如果`configure`未能在用户的系统上找到*pthread*库，我们是应该让构建过程失败，还是构建一个没有多线程的`jupiter`程序？

如果我们选择让构建失败，用户会注意到，因为构建会因错误信息而停止（尽管这个错误信息可能并不是很友好——编译或链接过程会因缺失头文件或未定义符号而出现难以理解的错误信息）。另一方面，如果我们选择构建一个单线程版本的`jupiter`，我们需要显示一些清晰的消息，说明程序正在构建时没有多线程功能，并解释原因。

一个潜在的问题是，有些用户的系统可能安装了*pthread*共享库，但没有安装*pthread.h*头文件——很可能是因为安装了*pthread*可执行文件（共享库）包，但没有安装开发者包。共享库通常与静态库和头文件分开打包，虽然可执行文件作为更高层次应用程序的依赖链的一部分安装，但开发者包通常由用户直接安装。^(10) 因此，Autoconf 提供了宏来测试库和头文件的存在。我们可以使用`AC_CHECK_HEADERS`宏来确保特定头文件的存在。

Autoconf 的检查非常彻底。它们通常不仅确保文件存在，而且确保文件是正确的，因为它们允许你指定关于文件的断言，然后宏会验证这些断言。`AC_CHECK_HEADERS`宏不仅仅是扫描文件系统寻找请求的头文件。像`AC_SEARCH_LIBS`一样，`AC_CHECK_HEADERS`宏会构建一个短小的测试程序，并将其编译，以确保编译器既能找到该文件，又能使用它。本质上，Autoconf 宏不仅仅是测试特定功能是否存在，而是测试这些功能所需的功能性。

`AC_CHECK_HEADERS`宏在*GNU Autoconf Manual*中的定义如下：

```
AC_CHECK_HEADERS(header-file..., [action-if-found],
    [action-if-not-found], [includes = 'AC_INCLUDES_DEFAULT'])
```

对于空格分隔的每个给定系统头文件*`header-file`*，如果它存在，则定义`HAVE_`*`header-file`*（全大写）。如果给定了*`action-if-found`*，它是找到某个头文件时执行的额外 Shell 代码。你可以给它一个`break`的值，在第一次匹配时跳出循环。如果给定了*`action-if-not-found`*，则在没有找到某个头文件时执行该代码。

*`includes`*的解释方式与`AC_CHECK_HEADER`相同，用于选择在测试头文件之前提供的一组预处理指令。^(11)

通常，`AC_CHECK_HEADERS` 只会在第一个参数中调用一个所需头文件的列表。其余参数是可选的，且不常使用，因为该宏在没有这些参数时通常已经能很好地工作。

我们将使用 `AC_CHECK_HEADERS` 在 *configure.ac* 中检查 *pthread.h* 头文件。如你所见，*configure.ac* 已经调用了 `AC_CHECK_HEADERS` 来查找 *stdlib.h*。`AC_CHECK_HEADERS` 接受一个文件名列表，所以我们只需将 *pthread.h* 添加到该列表中，文件名之间用空格分隔，如示例 5-10 所示。

Git 标签 5.3

```
--snip--
# Checks for header files.
AC_CHECK_HEADERS([stdlib.h pthread.h])
--snip--
```

*示例 5-10:* configure.ac: *将* pthread.h *添加到 `AC_CHECK_HEADERS` 宏中*

为了让尽可能多的人使用这个包，我们将使用双模式构建方法，这将允许我们在没有 *pthread* 库的情况下，至少提供 `jupiter` 程序的某种形式给用户。为了实现这一点，我们需要在 *src/main.c* 中添加一些条件预处理器语句，如示例 5-11 所示。

```
#include "config.h"

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

int main(int argc, char * argv[])
{
#if HAVE_PTHREAD_H
    pthread_t tid;
    pthread_create(&tid, 0, print_it, argv[0]);
    pthread_join(tid, 0);
#else
    print_it(argv[0]);
#endif
    return 0;
}
```

*示例 5-11:* src/main.c: *根据 pthread.h 的存在添加条件代码*

在这个版本的 *main.c* 中，我们添加了对头文件的条件检查。如果 `AC_CHECK_HEADERS` 生成的 shell 脚本找到了 *pthread.h* 头文件，`HAVE_PTHREAD_H` 宏将在用户的 *config.h* 文件中定义为值 `1`。如果 shell 脚本没有找到该头文件，原始的 `#undef` 语句将保留在 *config.h* 中被注释掉。因为我们依赖于这些定义，我们还需要在 *main.c* 的顶部包含 *config.h*。

如果你选择不在 *configure.ac* 中使用 `AC_CONFIG_HEADERS` 宏，那么 `@DEFS@` 将包含所有通过调用 `AC_DEFINE` 宏生成的定义。在这个例子中，我们使用了 `AC_CONFIG_HEADERS`，因此 *config.h.in* 将包含大部分这些定义，而 `@DEFS@` 只会包含 `HAVE_CONFIG_H`，而我们实际上并未使用它。^(12) *config.h.in* 模板方法显著缩短了编译器命令行（并且也使得在非 Autotools 平台上手动修改模板并拍摄快照变得简单）。示例 5-12 显示了对 *src/Makefile.in* 模板的必要更改。

```
--snip--
# Tool-related substitution variables
CC = @CC@
DEFS = @DEFS@
LIBS = @LIBS@
CFLAGS = @CFLAGS@
CPPFLAGS = @CPPFLAGS@
--snip--
jupiter: main.c
        $(CC) $(CFLAGS) $(DEFS) $(CPPFLAGS) -I. -I$(srcdir) -I..\
          -o $@ $(srcdir)/main.c $(LIBS)
--snip--
```

*示例 5-12:* src/Makefile.in: *在 src 级别的 Makefile 中添加 `@DEFS@` 的使用*

**注意**

*我在 *`$(DEFS)`* 前添加了 *`$(CPPFLAGS)`*，给最终用户提供了在命令行中覆盖我的任何政策决策的选项*。

我们现在拥有了所有需要的条件来构建 `jupiter` 程序。如果用户的系统已安装 *pthread* 功能，用户将自动构建一个使用多线程执行的 `jupiter` 版本；否则，他们只能选择串行执行。剩下的工作就是向 *configure.ac* 中添加一些代码，以便如果 `configure` 找不到 *pthread* 库，它将显示一条消息，指示将构建一个使用串行执行的程序。

现在，考虑一个不太可能的情况：用户已安装头文件，但没有安装库。例如，如果用户执行 `./configure` 时使用了 `CPPFLAGS=-I/usr/local/include`，但忽略了添加 `LDFLAGS=-L/usr/local/lib`，`configure` 会认为头文件可用，但库缺失。只需简单地跳过头文件检查（如果 `configure` 找不到库），就能轻松解决此问题。清单 5-13 显示了对 *configure.ac* 所做的必要更改。

Git 标签 5.4

```
--snip--
# Checks for libraries.
have_pthreads=no
AC_SEARCH_LIBS([pthread_create], [pthread], [have_pthreads=yes])

# Checks for header files.
AC_CHECK_HEADERS([stdlib.h])

if test "x${have_pthreads}" = xyes; then
    AC_CHECK_HEADERS([pthread.h], [], [have_pthreads=no])
fi

if test "x${have_pthreads}" = xno; then
    AC_MSG_WARN([
 ------------------------------------------
  Unable to find pthreads on this system.
  Building a single-threaded version.
  ------------------------------------------])
fi
--snip--
```

*清单 5-13:* configure.ac: *添加代码以指示在配置过程中多线程不可用*

现在，当我们运行 `./bootstrap.sh` 和 `./configure` 时，我们将看到一些额外的输出（在此处突出显示）：

```
$ ./bootstrap.sh
$ ./configure
checking for gcc... gcc
--snip--
checking for library containing pthread_create... -lpthread
--snip--
checking pthread.h usability... yes
checking pthread.h presence... yes
checking for pthread.h... yes
configure: creating ./config.status
config.status: creating Makefile
config.status: creating src/Makefile
config.status: creating config.h
$
```

例如，如果用户的系统缺少 *pthread.h* 头文件，他们将看到不同的输出。为了测试此情况，我们可以使用一个涉及 Autoconf 缓存变量的技巧。通过预设表示 *pthread.h* 头文件存在的缓存变量为 `no`，我们可以欺骗 `configure` 使其根本不去查找 *pthread.h*，因为如果缓存变量已经设置，它会认为搜索已经完成。让我们尝试一下：

```
$ ./configure ac_cv_header_pthread_h=no
checking for gcc... gcc
--snip--
checking for library containing pthread_create... -lpthread
--snip--
checking for pthread.h... (cached) no
configure: WARNING:
  ------------------------------------------
  Unable to find pthreads on this system.
  Building a single-threaded version.
  ------------------------------------------
configure: creating ./config.status
config.status: creating Makefile
config.status: creating src/Makefile
config.status: creating config.h
$
```

如果我们选择在找不到 *pthread.h* 头文件或 *pthread* 库时使构建失败，那么源代码将会更简单；无需进行条件编译。在这种情况下，我们可以将 *configure.ac* 更改为如下 清单 5-14 所示。

```
--snip--
# Checks for libraries.
have_pthreads=no
AC_SEARCH_LIBS([pthread_create], [pthread], [have_pthreads=yes])

# Checks for header files.
AC_CHECK_HEADERS([stdlib.h])

if test "x${have_pthreads}" = xyes; then
    AC_CHECK_HEADERS([pthread.h], [], [have_pthreads=no])
fi

if test "x${have_pthreads}" = xno; then
    AC_MSG_ERROR([
  ------------------------------------------
  The pthread library and header files are
  required to build jupiter. Stopping...
  Check 'config.log' for more information.
  ------------------------------------------])
fi
--snip--
```

*清单 5-14: 如果未找到 *pthread* 库，则使构建失败*

**注意**

*Autoconf 宏生成的 Shell 代码用于检查系统功能的存在，并根据这些测试设置变量。然而，作为维护者，你需要向 configure.ac 中添加 Shell 代码，以便根据结果变量的内容做出功能决策。*

#### *打印消息*

在前面的示例中，我们使用了几个 Autoconf 宏来在配置过程中显示消息：`AC_MSG_WARN` 和 `AC_MSG_ERROR`。以下是 Autoconf 提供的各种 `AC_MSG_*` 宏的原型：

```
AC_MSG_CHECKING(feature-description)
AC_MSG_RESULT(result-description)
AC_MSG_NOTICE(message)
AC_MSG_ERROR(error-description[, exit-status])
AC_MSG_FAILURE(error-description[, exit-status])
AC_MSG_WARN(problem-description)
```

`AC_MSG_CHECKING` 和 `AC_MSG_RESULT` 宏是配合使用的。`AC_MSG_CHECKING` 宏会打印一行，指示正在检查某个特性，但不会在该行的末尾打印换行符。当该特性在用户的机器上找到（或未找到）后，`AC_MSG_RESULT` 宏会打印结果，并在行的末尾添加换行符，完成由 `AC_MSG_CHECKING` 开始的行。*`result-description`* 文本应在 *`feature-description`* 消息的上下文中有意义。例如，消息 `Looking for a C compiler...` 可能以找到的编译器名称结束，或者以 `not found` 结束。

**注意**

*作为 configure.ac 的作者，您应尽力避免这两个宏调用之间出现额外的文本，因为如果这两组输出之间有不相关的文本，用户会很难跟踪*。

`AC_MSG_NOTICE` 和 `AC_MSG_WARN` 宏仅将字符串打印到屏幕上。`AC_MSG_WARN` 的前缀文本是 `configure: WARNING:`，而 `AC_MSG_NOTICE` 的前缀文本仅为 `configure:`。

`AC_MSG_ERROR` 和 `AC_MSG_FAILURE` 宏会生成错误消息，停止配置过程，并将错误代码返回给 shell。`AC_MSG_ERROR` 的前缀文本是 `configure: error:`。`AC_MSG_FAILURE` 宏会打印一条通知，指示错误发生的目录、用户指定的消息，然后是文本 `See 'config.log' for more details`。这些宏中的可选第二个参数 (*`exit-status`*) 允许维护者指定返回给 shell 的特定状态码。默认值是 `1`。

这些宏输出的文本消息会显示在 `stdout` 上，并发送到 *config.log* 文件中，因此使用这些宏很重要，而不是仅仅使用 shell 的 `echo` 或 `printf` 语句。

在这些宏的第一个参数中提供多行文本尤其重要，特别是在警告消息的情况下，警告仅表示构建在有限制的情况下继续进行。在大型配置过程中，快速构建机器上，一条单行警告信息可能会快速出现并被用户忽略。对于 `configure` 因错误终止的情况，这不是问题，因为用户很容易在输出的最后发现问题。^(13)

### 支持可选功能和包

我们已经讨论了如何处理 *pthread* 库存在与否的不同情况。但如果用户希望在安装了 *pthread* 库的情况下构建 `jupiter` 的单线程版本怎么办？我们当然不希望在 Jupiter 的 *README* 文件中添加一条提示，告诉用户重命名他们的 *pthread* 库！我们也不希望用户必须使用我们的 Autoconf 缓存变量技巧。

Autoconf 提供了两个用于处理可选功能和外部软件包的宏：`AC_ARG_ENABLE` 和 `AC_ARG_WITH`。它们的原型如下：

```
AC_ARG_WITH(package, help-string, [action-if-given], [action-if-not-given])
AC_ARG_ENABLE(feature, help-string, [action-if-given], [action-if-not-given])
```

与许多 Autoconf 宏一样，这两个宏仅用于设置一些环境变量：

AC_ARG_WITH `${withval}` 和 `${with_`package`}`

AC_ARG_ENABLE `${enableval}` 和 `${enable_`feature`}`

宏也可以以更复杂的形式使用，其中环境变量由 shell 脚本在宏的可选参数中使用。无论哪种情况，生成的变量必须在*configure.ac*中使用，否则执行检查就没有意义。

这些宏的设计目的是将选项 `--enable-feature[=yes|no]`（或 `--disable-feature`）和 `--with-package[=arg]`（或 `--without-package`）添加到生成的配置脚本的命令行界面，并在用户输入 `./configure --help` 时生成适当的帮助文本。如果用户提供了这些选项，宏将在脚本中设置前面的环境变量。（这些变量的值可能稍后在脚本中用于设置或清除各种预处理器定义或替代变量。）

`AC_ARG_WITH` 控制项目对可选外部软件包的使用，而 `AC_ARG_ENABLE` 控制可选软件功能的包含或排除。选择使用其中一个或另一个通常取决于你对正在考虑的软件的看法，有时只是个人偏好，因为这两个宏提供的功能集有些重叠。

例如，在 Jupiter 项目中，可以合理地认为 Jupiter 使用 *pthread* 代表使用了一个外部软件包，因此你会使用 `AC_ARG_WITH`。然而，也可以说 *异步处理* 是一个可以通过 `AC_ARG_ENABLE` 启用的软件功能。事实上，这两种说法都是正确的，选择使用哪个选项应该由对你所提供可选访问的功能或软件包的高层架构视角来决定。*pthread* 库不仅提供线程创建函数，还提供互斥锁和条件变量，这些都可以被一个不创建线程的库包使用。如果一个项目提供的库需要在多线程进程中以线程安全的方式工作，它很可能会使用 *pthread* 库中的互斥锁对象，但可能永远不会创建线程。因此，用户可以选择在配置时禁用异步执行功能，但项目仍然需要链接到 *pthread* 库以访问互斥锁功能。在这种情况下，指定 `--enable-async-exec` 比 `--with-pthreads` 更有意义。

一般来说，当用户需要在不同包或项目内部提供的不同功能实现之间进行选择时，你应该使用`AC_ARG_WITH`。例如，如果`jupiter`有某种原因需要加密文件，它可能会选择使用内部加密算法或外部加密库。默认配置可能使用内部算法，但该包可能允许用户通过命令行选项`--with-libcrypto`来覆盖默认值。谈到安全性，使用广为人知的库确实能帮助你的包获得社区的信任。

#### *为功能选项编写代码*

决定使用`AC_ARG_ENABLE`后，我们如何默认启用或禁用`async-exec`功能呢？这两种情况下如何在*configure.ac*中编码的区别仅限于帮助文本和传递给*`action-if-not-given`*参数的 shell 脚本。帮助文本描述了可用的选项和默认值，而 shell 脚本则指明了如果没有指定选项时希望发生的情况。（当然，如果指定了，我们不需要假设任何事情。）

假设我们决定将异步执行作为一个风险较大或实验性的功能，默认情况下希望禁用它。在这种情况下，我们可以将 Listing 5-15 中的代码添加到*configure.ac*中。

```
--snip--
AC_ARG_ENABLE([async-exec],
    [  --enable-async-exec     enableasync exec],
    [async_exec=${enableval}], [async_exec=no])
--snip--
```

*Listing 5-15: 默认情况下禁用的功能*

另一方面，如果我们决定异步执行对 Jupiter 来说是基本功能，那么我们可能应该像 Listing 5-16 中那样默认启用它。

```
--snip--
AC_ARG_ENABLE([async-exec],
    [  --disable-async-exec    disableasync exec],
    [async_exec=${enableval}], [async_exec=yes])
--snip--
```

*Listing 5-16: 默认情况下启用的功能*

现在问题是，我们是否在不管用户是否需要这个功能的情况下检查库和头文件，还是只有在启用`async-exec`功能时才检查？在这种情况下，这是一个偏好问题，因为我们仅为这个功能使用*pthread*库。（如果我们还因为非特定功能的原因使用它，那么无论如何都必须检查它。）

在需要即使功能禁用也需要库的情况下，我们会像前面的例子那样添加`AC_ARG_ENABLE`，并额外调用`AC_DEFINE`来为这个功能创建一个*config.h*定义。由于我们并不希望在库或头文件缺失时启用该功能——即使用户特别要求启用——我们还会添加一些 shell 代码，在库或头文件缺失时将该功能关闭，如 Listing 5-17 所示。

Git 标签 5.5

```
--snip--
# Checks for programs.
AC_PROG_CC
AC_PROG_INSTALL

# Checks for header files.
AC_CHECK_HEADERS([stdlib.h])

# Checks for command line options
AC_ARG_ENABLE([async-exec],
    [  --disable-async-exec    disable async execution feature],
    [async_exec=${enableval}], [async_exec=yes])

have_pthreads=no
AC_SEARCH_LIBS([pthread_create], [pthread], [have_pthreads=yes])

if test "x${have_pthreads}" = xyes; then
    AC_CHECK_HEADERS([pthread.h], [], [have_pthreads=no])
fi

if test "x${have_pthreads}" = xno; then
 ➊ if test "x${async_exec}" = xyes; then
        AC_MSG_WARN([
  ------------------------------------------
  Unable to find pthreads on this system.
  Building a single-threaded version.
  ------------------------------------------])
    fi
    async_exec=no
fi

if test "x${async_exec}" = xyes; then
    AC_DEFINE([ASYNC_EXEC], [1], [async execution enabled])
fi

# Checks for libraries.

# Checks for typedefs, structures, and compiler characteristics.
--snip--
```

*Listing 5-17:* configure.ac: *在配置期间正确管理可选功能*

我们正在将原有的库检查替换为新的命令行参数检查，这样不仅能检查用户未指定偏好的默认情况下的库，还带来了额外的好处。正如你所看到的，现有的大部分代码保持不变，只是添加了一些额外的脚本来处理用户的命令行选择。

**注意**

*在清单 5-17 中有些地方似乎有多余的空白或任意的缩进。这是故意的，因为它使得在运行`configure`时输出能够正确格式化。我们将在稍后通过添加额外的宏来修复其中一些问题*。

请注意，在➊处，我还添加了一个对`async_exec`变量中`yes`值的额外测试，因为这些文本真正属于功能测试，而不是*pthread*库测试。请记住，我们正在尝试在测试*pthread*功能和测试`async-exec`功能本身的需求之间创建逻辑分隔。

当然，现在我们还必须修改*src/main.c*，以使用新的定义，如清单 5-18 所示。

```
--snip--
#if HAVE_PTHREAD_H
# include <pthread.h>
#endif

static void * print_it(void * data)
{
    printf("Hello from %s!\n", (const char *)data);
    return 0;
}

int main(int argc, char * argv[])
{
#if ASYNC_EXEC
    pthread_t tid;
    pthread_create(&tid, 0, print_it, argv[0]);
    pthread_join(tid, 0);
#else
    print_it(argv[0]);
#endif
    return 0;
}
```

*清单 5-18:* src/main.c: *更改`async-exec`特定代码的条件*

请注意，我们保留了`HAVE_PTHREAD_H`检查以便在包含头文件时，可以便于以不同于此功能要求的方式使用*pthread.h*。

为了仅在启用该功能时检查库和头文件，我们将原始的检查代码包装在`async_exec`的测试中，如清单 5-19 所示。

Git 标签 5.6

```
--snip--
# Checks for command line options.
AC_ARG_ENABLE([async-exec],
  [    --disable-async-exec        disable async execution feature],
  [async_exec=${enableval}], [async_exec=yes])

if test "x${async_exec}" = xyes; then
    have_pthreads=no
    AC_SEARCH_LIBS([pthread_create], [pthread], [have_pthreads=yes])

    if test "x${have_pthreads}" = xyes; then
        AC_CHECK_HEADERS([pthread.h], [], [have_pthreads=no])
    fi

    if test "x${have_pthreads}" = xno; then
        AC_MSG_WARN([
  -----------------------------------------
  Unable to find pthreads on this system.
  Building a single-threaded version.
  -----------------------------------------])
        async_exec=no
    fi
fi

if test "x${async_exec}" = xyes; then
    AC_DEFINE([ASYNC_EXEC], 1, [async execution enabled])
fi
--snip--
```

*清单 5-19:* configure.ac: *仅在启用功能时检查库和头文件*

这次，我们将`async_exec`的测试从仅围绕消息语句移动到围绕整个头文件和库检查集合，这意味着如果用户禁用了`async_exec`功能，我们甚至不会查找*pthread*头文件和库。

#### *格式化帮助字符串*

我们将对清单 5-17 中`AC_ARG_ENABLE`的使用做最后的修改。请注意，在第二个参数中，方括号和参数文本开始之间正好有两个空格。你还会注意到，参数和描述之间的空格数量取决于参数文本的长度，因为描述文本应该与特定列对齐呈现。在清单 5-16 和 5-17 中，`--disable-async-exec`和描述之间有四个空格，但在清单 5-15 中，`--enable-async-exec`后面有五个空格，因为单词*enable*比*disable*少一个字符。

但是，如果 Autoconf 项目的维护者决定更改配置脚本的帮助文本格式怎么办？或者如果你修改了选项名称，但忘记调整帮助文本的缩进呢？

为了解决这些潜在问题，我们将使用一个名为`AS_HELP_STRING`的 Autoconf 助手宏，其原型如下：

```
AS_HELP_STRING(left-hand-side, right-hand-side,
    [indent-column = '26'], [wrap-column = '79'])
```

这个宏的唯一目的是抽象化关于在帮助文本中的各个位置应该嵌入多少空格的知识。要使用它，只需将`AC_ARG_ENABLE`中的第二个参数替换为`AS_HELP_STRING`的调用，如清单 5-20 所示。

Git 标签 5.7

```
--snip--
AC_ARG_ENABLE([async-exec],
    [AS_HELP_STRING([--disable-async-exec],
        [disable asynchronous execution @<:@default: no@:>@])],
    [async_exec=${enableval}], [async_exec=yes])
--snip--
```

*清单 5-20:* configure.ac: *使用`AS_HELP_STRING`*

**注意**

*关于清单 5-20 中围绕*`default: no`*的奇怪字符序列的详细信息，请参阅第 143 页的“Quadrigraphs”。*

### 检查类型和结构定义

现在，让我们考虑如何测试系统或编译器提供的类型和结构定义。在编写跨平台网络软件时，人们很快就会意识到，机器之间发送的数据需要以一种不依赖于特定 CPU 或操作系统架构的方式进行格式化。一些系统的本地整数大小是 32 位，而另一些则是 64 位。有些系统将整数值从最低有效字节到最高有效字节存储在内存和磁盘中，而另一些则相反。

让我们考虑一个例子。当使用 C 语言结构格式化网络消息时，你将遇到的第一个障碍是缺乏从一个平台到另一个平台具有相同大小的基本 C 语言类型。一个 32 位机器字大小的 CPU 可能会有一个 32 位的`int`和`unsigned`类型。C 语言中基本整数类型的大小是实现定义的。这是设计使然，目的是允许实现使用对每个平台最优的`char`、`short`、`int`和`long`的大小。

尽管这一语言特性对于优化设计为在单个平台上运行的软件非常有用，但在选择类型以便将数据*在平台之间*移动时却并不太有帮助。为了解决这个问题，工程师们尝试了从将网络数据作为字符串发送（如 XML 和 JSON）到发明自己的大小类型的各种方法。

为了弥补语言中的这一不足，C99 标准提供了大小类型`int`*`N`*`_t`和`uint`*`N`*`_t`，其中*`N`*可以是`8`、`16`、`32`或`64`。不幸的是，并不是所有今天的编译器都提供这些类型。（不足为奇的是，GNU C 已经领先一段时间，提供了通过包含*stdint.h*头文件来支持 C99 大小的类型。）

为了在某种程度上缓解这一痛苦，Autoconf 提供了宏来确定 C99 特定的标准化类型是否存在于用户的平台上，然后在它们不存在时进行定义。例如，你可以在*configure.ac*中添加一个`AC_TYPE_UINT16_T`的调用，以确保`uint16_t`在你的用户平台上存在，不管是作为*stdint.h*中的系统定义，还是作为更为普遍的非标准*inttypes.h*，或者作为 Autoconf 在*config.h*中的定义。

这些针对整数类型的编译器测试通常由配置脚本编写，作为一段类似于示例 5-21 中的 C 代码。

```
int main()
{
 ➊ static int test_array[1 - 2 * !((uint16_t) -1 >> (16 - 1) == 1)];
    test_array[0] = 0;
    return 0;
}
```

*示例 5-21：编译器检查 `uint16_t` 的正确实现*

你会注意到，在示例 5-21 中，重要的代码行在 ➊ 位置，即 `test_array` 的声明位置。Autoconf 依赖于一个事实，即所有 C 编译器在你尝试定义一个负大小的数组时都会生成错误。如果在该平台上 `uint16_t` 不是恰好为 16 位的无符号数据，则数组大小将是负数。

还要注意，示例中的带括号的表达式是一个编译时表达式。^(14) 是否能通过更简单的语法来实现，这谁也说不清，但这段代码在所有 Autoconf 支持的编译器中都能成功运行。只有在满足以下三个条件时，数组才会被定义为非负大小：

+   `uint16_t` 在其中一个包含的头文件中定义。

+   `uint16_t` 的大小恰好是 16 位。

+   `uint16_t` 在此平台上是无符号的。

按照示例 5-22 中显示的模式使用此宏提供的定义。即使在没有 *stdint.h* 或 *inttypes.h* 的系统上，Autoconf 也会在 *config.h* 中添加代码，如果系统的头文件没有提供 `uint16_t`，它将会定义该类型，这样你就可以在源代码中使用该类型，而无需额外的测试。

```
#include "config.h"

#if HAVE_STDINT_H
# include <stdint.h>
#elif HAVE_INTTYPES_H
# include <inttypes.h>
#endif
--snip--
uint16_t x;
--snip--
```

*示例 5-22：正确使用 Autoconf 的 `uint16_t` 定义的源代码*

Autoconf 提供了几十种类型检查，如 `AC_TYPE_UINT16_T`，详情请参见《*GNU Autoconf 手册*》的第 5.9 节。此外，通用类型检查宏 `AC_CHECK_TYPES` 允许你指定一个逗号分隔的、你项目需要的可疑类型列表。

**注意**

*这个列表使用逗号分隔，因为某些定义（比如 *`struct fooble`*）可能包含空格。由于它们是用逗号分隔的，因此如果列出多个类型，必须使用 Autoconf 的方括号引号围绕这个参数。*

这是 `AC_CHECK_TYPES` 的正式声明：

```
AC_CHECK_TYPES(types, [action-if-found], [action-if-not-found],
    [includes = 'AC_INCLUDES_DEFAULT'])
```

如果你没有在最后一个参数中指定头文件列表，则在编译器测试中将使用默认头文件，通过宏 `AC_INCLUDES_DEFAULT` 来实现，宏扩展为示例 5-23 中显示的文本。

```
#include <stdio.h>
#ifdef HAVE_SYS_TYPES_H
# include <sys/types.h>
#endif
#ifdef HAVE_SYS_STAT_H
# include <sys/stat.h>
#endif
#ifdef STDC_HEADERS
# include <stdlib.h>
# include <stddef.h>
#else
# ifdef HAVE_STDLIB_H
#  include <stdlib.h>
# endif
#endif
#ifdef HAVE_STRING_H
# if !defined STDC_HEADERS && defined HAVE_MEMORY_H
#  include <memory.h>
# endif
# include <string.h>
#endif
#ifdef HAVE_STRINGS_H
# include <strings.h>
#endif
#ifdef HAVE_INTTYPES_H
# include <inttypes.h>
#endif
#ifdef HAVE_STDINT_H
# include <stdint.h>
#endif
#ifdef HAVE_UNISTD_H
# include <unistd.h>
#endif
```

*示例 5-23：Autoconf 版本 2.69 中 `AC_INCLUDES_DEFAULT` 的定义*

如果你知道你的类型没有在这些头文件之一中定义，那么你应该指定一个或多个需要包含在测试中的头文件，如示例 5-24 所示。这个列表首先包括默认的头文件，然后是额外的头文件（通常仍然需要一些默认头文件）。

```
   AC_CHECK_TYPES([struct doodah], [], [], [
➊ AC_INCLUDES_DEFAULT
   #include<doodah.h>
   #include<doodahday.h>])
```

*示例 5-24：在检查 `struct doodah` 时使用非默认的 `include` 集*

请注意在清单 5-24 中的➊，我将宏的最后一个参数分成了三行写在*configure.ac*文件中，并且没有缩进。这个参数的文本会原样包含在测试源文件中，因此你需要确保你放入这个参数中的内容在你使用的编程语言中是有效的代码。

**注意**

*与测试相关的问题通常是开发者在使用 Autoconf 时抱怨的类型。当你遇到这种语法问题时，请检查*config.log*文件，里面包含了所有失败测试的完整源代码，包括在编译测试时生成的编译器输出。这些信息通常能提供解决问题的线索*。

### AC_OUTPUT 宏

最后，我们来到了`AC_OUTPUT`宏，它在`configure`中展开为 shell 代码，根据前面宏展开中指定的数据生成`config.status`脚本。所有其他宏必须在`AC_OUTPUT`展开之前使用，否则它们对你生成的`configure`脚本将没有多大价值。（额外的 shell 脚本可以在`AC_OUTPUT`之后放入*configure.ac*中，但它不会影响`config.status`执行的配置或文件生成。）

考虑在`AC_OUTPUT`后添加 shell 的`echo`或`printf`语句，告诉用户构建系统是如何根据指定的命令行选项进行配置的。你还可以使用这些语句告诉用户`make`的其他有用目标。例如，我们可以在 Jupiter 的*configure.ac*文件中，在`AC_OUTPUT`*之后*添加代码，如清单 5-25 所示。

Git 标签 5.8

```
--snip--
AC_OUTPUT

cat << EOF
-------------------------------------------------

${PACKAGE_NAME} Version ${PACKAGE_VERSION}

Prefix: '${prefix}'.
Compiler: '${CC} ${CFLAGS} ${CPPFLAGS}'

Package features:
  Async Execution: ${async_exec}

Now type 'make @<:@<target>@:>@'
  where the optional <target> is:
    all                - build all binaries
    install            - install everything

--------------------------------------------------
EOF
```

*清单 5-25：* configure.ac: *将配置摘要文本添加到`configure`的输出中*

在*configure.ac*文件的末尾添加这样的输出是一个方便的项目功能，因为它可以让用户一目了然地看到配置过程中发生了什么。由于像`async_exec`这样的变量会根据配置设置为`yes`或`no`，用户可以看到请求的配置是否真正生效。

**注意**

*Autoconf 版本 2.62（及之后版本）比早期版本更好地解读用户关于方括号使用的意图。在过去，你可能需要使用四重符号强制 Autoconf 显示一个方括号，但现在你可以直接使用字符本身。大多数发生的问题是由于没有正确引用参数。这种增强的功能主要来自 Autoconf 库宏的增强，它们可能接受带方括号字符的参数。为了确保方括号在你自己的*configure.ac*代码中不会被误解，你应该阅读“引用规则”中的 M4 双重引号部分，详见第 438 页*。

**四重符号**

在清单 5-25 中，围绕单词`<target>`的那些有趣的字符序列被称为*四字符序列*，简称*四字符组*。它们与转义序列的作用相同，但四字符组比转义字符或转义序列更可靠，因为它们永远不会受到歧义的影响。

序列`@<:@`是方括号字符的四字符序列，而`@:>@`是闭合方括号字符的四字符序列。这些四字符组将*始终*由`autom4te`输出为字面意义上的方括号字符。这发生在 M4 处理完文件后，因此没有机会将它们误解为 Autoconf 的引号字符。

如果你有兴趣更详细地研究四字符组，请查阅*GNU Autoconf 手册*的第八部分。

### 摘要

在本章中，我们介绍了许多项目的*configure.ac*文件中发现的一些更高级的结构。我们从生成替代变量所需的宏开始。我称这些为“高级”宏，因为许多更高级别的 Autoconf 宏在内部使用`AC_SUBST`和`AC_DEFINE`，使它们对你来说有些透明。然而，了解这些宏有助于你理解 Autoconf 的工作原理，并为你学习编写自己的宏提供必要的背景信息。

我们讲解了编译器和其他工具的检查，以及在用户系统上检查一些不太常见的数据类型和结构。本章中的示例旨在帮助你理解 Autoconf 类型和结构定义检查宏的正确使用方法，以及其他相关宏。

我们还研究了一种调试复杂 Autoconf 宏使用的技巧：在*configure.ac*中的宏调用周围使用栅栏，快速定位`configure`中生成的相关文本。我们查看了库和头文件的检查，并审视了这些 Autoconf 宏正确使用的一些细节。我们详细探讨了构建一个健壮且用户友好的配置过程，包括向 Autoconf 生成的`configure`脚本添加项目特定的命令行选项。

最后，我们讨论了`AC_OUTPUT`宏在*configure.ac*中的正确位置，以及添加一些总结生成的 Shell 代码，旨在帮助用户了解在其系统上配置你的项目时发生了什么。

从第四章和第五章中要带走的一个重要 Autoconf 概念，在第四章的开头就已明确指出：Autoconf 从你写入 *configure.ac* 的 shell 源代码生成 shell 脚本。这意味着，只要你理解所调用宏的正确用法，你就能对最终生成的配置脚本拥有 *完全* 的控制权。事实上，你可以在 *configure.ac* 中做任何你想做的事情。Autoconf 宏的存在，仅仅是为了让你选择的操作更加一致，并且更容易编写。你越少依赖 Autoconf 宏来执行配置任务，你的用户在配置过程中与其他开源项目相比，就越不一致。

下一章将暂时离开 Autoconf，我们将关注 GNU Automake，这是一个 Autotools 工具链的附加组件，它抽象化了为软件项目创建功能强大的 makefile 的许多细节。
