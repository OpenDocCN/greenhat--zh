# 第六章。GNU Make 标准库

*GNU Make 标准库 (GMSL)* 是一个托管在 SourceForge 上的开源项目，由我发起，旨在收集 makefile 作者经常重复编写的常见函数。为了防止 makefile 编写者重复造轮子，GMSL 实现了常见的函数，例如反转列表、将字符串转为大写，或对列表的每个元素应用一个函数。

GMSL 包含了列表和字符串操作函数、完整的整数算术库，以及数据结构的函数。还包括 GNU `make` 对关联数组、集合和栈的实现，并提供内建的调试功能。

在本章中，你将学习如何在实际的 makefile 中使用 GMSL 函数。此外，你将看到 GMSL 函数的不同类别的完整参考。要查看 GMSL 的最新版本，请访问 *[`gmsl.sf.net/`](http://gmsl.sf.net/)*。

# 导入 GMSL

GMSL 实现为一对名为 `gmsl` 和 `__gmsl` 的 makefile。`__gmsl` 被 `gmsl` 导入，因此要在你的 makefile 中包含 GMSL，只需添加以下内容：

```
include gmsl
```

你可以在任意多个文件中执行此操作。为了防止多次定义和不必要的错误信息，GMSL 会自动检测是否已经包含过。

当然，GNU `make` 必须能够找到 `gmsl` 和 `__gmsl`。为了实现这一点，GNU `make` 默认会在多个地方查找 makefile，包括 `/usr/local/include`、`/usr/gnu/include/`、`/usr/include`、当前目录，以及任何通过 GNU `make -I`（或 `--include-dirL`）命令行选项指定的目录。

将 `gmsl` 和 `__gmsl` 放置在 `/usr/local/include` 是一个好地方，这样它们将对所有你的 makefile 可用。

如果 GNU `make` 无法找到 `gmsl` 或 `__gmsl`，你将看到常规的 GNU `make` 错误信息：

```
Makefile:1: gmsl: No such file or directory
```

GMSL 使用了一个小技巧，使得 `gmsl` 的位置完全灵活。由于 `gmsl` 使用 `include` 来查找 `__gmsl`，因此 `gmsl` makefile 需要知道在哪里找到 `__gmsl`。

假设 `gmsl` 存储在 `/foo` 中，并通过 `include /foo/gmsl` 来包含。为了使这个工作正常，而无需修改 `gmsl` 来硬编码 `__gmsl` 的位置，`gmsl` 会使用 `MAKEFILE_LIST` 来确定它的位置，然后将适当的路径添加到 `include __gmsl` 前面：

```
# Try to determine where this file is located. If the caller did
# include /foo/gmsl then extract the /foo/ so that __gmsl gets
# included transparently

__gmsl_root := $(word $(words $(MAKEFILE_LIST)),$(MAKEFILE_LIST))

# If there are any spaces in the path in __gmsl_root then give up

ifeq (1,$(words $(__gmsl_root)))
__gmsl_root := $(patsubst %gmsl,%,$(__gmsl_root))
else
__gmsl_root :=
endif

include $(__gmsl_root)__gmsl
```

如果你希望你的 makefile 具有位置独立性，这是一项非常有用的技巧。

# 调用 GMSL 函数

GMSL 中的函数实现为普通的 GNU `make` 函数声明。例如，函数 `last`（返回列表的最后一个元素）是这样声明的：

```
last = $(if $1,$(word $(words $1),$1))
```

该函数是通过 GNU `make` 的内建 `$(call)` 来调用的。例如，要返回列表 `1 2 3` 的最后一个元素，可以这样做：

```
$(call last,1 2 3)
```

这将返回 `3`。`$(call)` 展开其第一个参数中指定的变量（在这个例子中是 `last`），并将特殊的本地变量（`$1`、`$2`、`$3` 等）设置为传递给 `$(call)` 的参数。所以在这个例子中，`$1` 是 `1 2 3`。

GMSL 定义了布尔值`true`和`false`，它们只是变量，可以通过`$()`或`${}`访问：例如，`$(true)`或`${false}`。`false`是一个空字符串，`true`是字母`T`；这些定义对应于 GNU `make`中 true（非空字符串）和 false（空字符串）的概念。你可以在 GNU `make`的`$(if)`函数中或在预处理器`ifeq`中使用`true`和`false`：

```
$(if $(true),It's true!,Totally false)

ifeq ($(true),$(true))
--*snip*--
*endif*
```

这些例子是虚构的。你应该期望`$(true)`在`$(if)`中的返回值和`ifeq`中的第一个`$(true)`是来自函数调用的返回值，而不是常量值。

# 检查 GMSL 版本

GMSL 包含一个可以用来检查包含的版本是否与你使用的 GMSL 版本兼容的函数。函数`gmsl_compatible`检查包含的 GMSL 版本号是否大于或等于传入参数的版本号。

在撰写本文时，当前的 GMSL 版本是`v1.1.7`。要检查包含的 GMSL 是否至少是`v1.1.2`，请调用`gmsl_compatible`并传入一个包含三个元素的列表参数：`1 1 2`。

```
$(call gmsl_compatible,1 1 2)
```

这将返回`$(true)`，因为当前的 GMSL 版本是`v1.1.7`，大于`v1.1.2`。如果我们请求的是`v2.0.0`，我们将得到`$(false)`的响应：

```
$(call gmsl_compatible,2 0 0)
```

确保你使用正确版本的 GMSL 的一个简单方法是将对`gmsl_compatible`的调用包装在一个断言中：

```
$(call assert,$(call gmsl_compatible,1 0 0),Wrong GMSL version)
```

如果发现不兼容的 GMSL 版本，这将停止`make`进程并报错。

# 示例：现实世界中的 GMSL 使用

现在你已经设置好了 GMSL，让我们看一些例子。这些例子解决了现实世界中 makefile 必须处理的一些问题，比如不区分大小写的比较和在路径中搜索文件。

## 不区分大小写的比较

GMSL 包含两个函数，允许你创建一个简单的函数来进行不区分大小写的字符串比较：

```
ifcase = $(call seq,$(call lc,$1),$(call lc,$2))
```

它通过将两个参数转换为小写（使用 GMSL 的`lc`函数），然后调用`seq`（GMSL 的字符串相等函数）来检查它们是否相同。以下是使用`ifcase`的一种方式：

```
CPPFLAGS += $(if $(call ifcase,$(DEBUG),yes),-DDEBUG,)
```

在这里，它用于查看`DEBUG`变量是否已设置为`yes`；如果是，它会将`-DDEBUG`添加到`CPPFLAGS`中。

## 在路径中查找程序

这是一个搜索`PATH`中可执行文件的函数定义：

```
findpath = $(call first,$(call map,wildcard,$(call addsuffix,/$1,$(call split,:,$(PATH)))))
```

例如，`$(call findpath,cat)`将搜索`PATH`中第一个`cat`程序。它使用了 GMSL 中的三个函数：`first`、`map`和`split`。同时，它还使用了两个内置函数：`wildcard`和`addsuffix`。

调用`split`将`PATH`变量拆分为一个列表，并在冒号处进行分隔。然后调用内置的`addsuffix`函数，它将`/$1`添加到`PATH`的每个元素中。`$1`包含`findpath`的参数，即我们正在搜索的程序名称（在这种情况下是`cat`）。

然后调用 GMSL 的`map`函数，在每个可能的程序文件名上执行内建的`wildcard`。如果文件名中没有通配符字符，`wildcard`将返回文件名（如果存在）或空字符串。因此，`map`的作用是通过依次测试每个文件，找到`cat`在`PATH`上的位置（或多个位置）。

最后，调用 GMSL 函数`first`返回`map`函数返回的列表中的第一个元素（即`PATH`中所有`cat`程序的第一个位置）。

GMSL 的调试功能之一是能够追踪对 GMSL 函数的调用。通过将`GMSL_TRACE`设置为`1`，GMSL 会输出每个对 GMSL 函数的调用及其参数。例如：

```
Makefile:8: split(':', '/home/jgc/bin:/usr/local/bin:/usr/bin:/usr/X11R6/bin:/
bin:/usr/games:/opt/gnome/bin:/opt/kde3/bin:/usr/lib/java/jre/bin')
Makefile:8: map('wildcard',' /home/jgc/bin/make /usr/local/bin/make /usr/bin/
make /usr/X11R6/bin/make /bin/make /usr/games/make /opt/gnome/bin/make /opt/
kde3/bin/make /usr/lib/java/jre/bin/make')
Makefile:8: first(' /usr/bin/make')
```

在这里，我们使用启用了追踪功能的`findpath`函数搜索`cat`。

## 使用断言检查输入

通常，`makefile`是在指定构建目标的情况下执行的（或者假设在`makefile`的开始部分有一个`all`目标或类似目标）。此外，通常还会有影响构建的环境变量（例如调试选项、架构设置等）。检查这些变量是否已正确设置的快速方法是使用 GMSL 断言函数。

这是一个示例，检查`DEBUG`是否已设置为`yes`或`no`，`ARCH`是否包含`Linux`，我们是否在`OUTDIR`变量中指定了输出目录，并且该目录是否存在：

```
$(call assert,$(OUTDIR),Must set OUTDIR)
$(call assert_exists,$(OUTDIR),Must set OUTDIR)
$(call assert,$(if $(call seq,$(DEBUG),yes),$(true),$(call seq,$(DEBUG),no)),DEBUG must be yes or no)
$(call assert,$(call findstring,Linux,$(ARCH)),ARCH must be Linux)
```

如果断言函数的第一个参数是`$(false)`（即空字符串），它们会生成一个致命错误。

第一个断言检查`$(OUTDIR)`是否已设置。如果它有一个非空值，则断言通过；否则，会生成一个错误：

```
Makefile:3: *** GNU Make Standard Library: Assertion failure: Must set OUTDIR.
Stop.
```

第二个断言是`assert_exists`形式，用来检查它的第一个参数在文件系统中是否存在。在这个例子中，它检查`$(OUTDIR)`所指向的目录是否存在。它不检查该路径是否是一个目录。如果需要，还可以添加另一个断言来检查这一点，如下所示：

```
$(call assert,$(wildcard $(OUTDIR)/.),OUTDIR must be a directory)
```

这段代码检查`$(OUTDIR)`是否包含一个点 (**`.`**)。如果没有，`$(OUTDIR)`就不是一个目录，调用`wildcard`将返回一个空字符串，从而导致断言失败。

第三个断言检查`DEBUG`是否为`yes`或`no`，通过 GMSL 的`seq`函数来验证其值。最后，我们使用`findstring`断言`$(ARCH)`必须包含单词`Linux`（且`L`为大写）。

## `DEBUG`是否设置为 Y？

GMSL 有逻辑运算符`and`、`or`、`xor`、`nand`、`nor`、`xnor`和`not`，这些运算符与 GNU `make`的布尔值概念以及 GMSL 变量`$(true)`和`$(false)`一起工作。

你可以将 GNU `make`（和 GMSL）的布尔值与 GMSL 函数以及 GNU `make`的内建`$(if)`一起使用。GMSL 的逻辑运算符是为与`$(if)`和 GNU `make`预处理器`ifeq`指令配合使用而设计的。

假设一个 makefile 有一个调试选项，通过将 `DEBUG` 环境变量设置为 `Y` 来启用。使用 GMSL 函数 `seq`（字符串相等）和 `or` 运算符，你可以轻松确定是否需要调试：

```
include gmsl

debug_needed := $(call or,$(call seq,$(DEBUG),Y),$(call seq,$(DEBUG),y))
```

因为 GMSL 有一个小写函数（`lc`），你可以不用 `or` 就写出这个示例：

```
include gmsl

debug_needed := $(call seq,$(call lc,$(DEBUG)),y)
```

但是逻辑运算符 `or` 让我们可以更加宽容，接受 `YES` 和 `Y` 作为调试选项的值：

```
include gmsl

debug_needed := $(call or,$(call seq,$(call lc,$(DEBUG)),y),$(call seq,$(call lc,$(DEBUG)),yes))
```

`debug_needed` 函数对大小写不敏感。

## `DEBUG` 是否设置为 Y 或 N？

逻辑运算符的另一个可能用途是强制 makefile 的用户将 `DEBUG` 设置为 `Y` 或 `N`，从而避免如果他们忘记调试选项而引发的问题。GMSL 断言函数 `assert` 会在其参数不为真时输出致命错误。所以我们可以用它来断言 `DEBUG` 必须是 `Y` 或 `N`：

```
include gmsl

$(call assert,$(call or,$(call seq,$(DEBUG),Y),$(call seq,$(DEBUG),N)),DEBUG must be Y or N)
```

这里有一个示例：

```
$ **make DEBUG=Oui**
Makefile:1: *** GNU Make Standard Library: Assertion failure: DEBUG must be Y or N.
Stop.
```

如果用户犯了把 `DEBUG` 设置为 `Oui` 的错误，断言会产生这个错误。

## 在预处理器中使用逻辑运算符

因为 GNU `make` 的预处理器（它有 `ifeq`、`ifneq` 和 `ifdef` 指令）没有逻辑运算，所以很难编写复杂的语句。例如，要在 GNU `make` 中定义一个 makefile 的部分，当 `DEBUG` 被设置为 `Y` 或 `Yes` 时，你必须重复一段代码（糟糕！）或者写一个难以理解的语句：

```
ifeq ($(DEBUG),$(filter $(DEBUG),Y Yes))
--*snip*--
endif
```

这个方法通过使用 `$(DEBUG)` 的值来过滤列表 `Y Yes`，如果 `$(DEBUG)` 不是 `Y` 或 `Yes`，则返回空列表，如果是，则返回 `$(DEBUG)` 的值。然后 `ifeq` 会将结果值与 `$(DEBUG)` 进行比较。这种做法很丑陋、难以维护，并且包含一个微妙的 bug。（如果 `$(DEBUG)` 为空会发生什么？提示：空值等同于 `Y` 或 `Yes`。）修复这个 bug 需要做类似这样的事情：

```
ifeq (x$(DEBUG)x,$(filter x$(DEBUG)x,xYx xYesx))
--*snip*--
endif
```

GMSL 的 `or` 运算符使这一点变得更加清晰：

```
include gmsl

ifeq ($(true),$(call or,$(call seq,$(DEBUG),Y),$(call seq,$(DEBUG),Yes)))
--*snip*--
endif
```

这种方法更容易维护。它通过对两个 `seq` 调用进行 `or` 操作，并将结果与 `$(true)` 进行比较来工作。

## 从列表中移除重复项

GMSL 函数 `uniq` 从列表中移除重复项。GNU `make` 有一个内建的 `sort` 函数，可以对列表进行排序并移除重复项；`uniq` 移除重复项但不排序列表（如果列表顺序很重要，这可以很有用）。

例如，`$(sort c b a a c)` 会返回 `a b c`，而 `$(call uniq,c b a a c)` 会返回 `c b a`。

假设你需要通过移除重复项并保留顺序来简化 `PATH` 变量。`PATH` 通常是一个以冒号分隔的路径列表（如 `/usr/bin:/bin:/usr/local/bin:/bin`）。这里的 `simple-path` 是去除重复项并保留顺序后的 `PATH`：

```
include gmsl

simple-path := $(call merge,:,$(call uniq,$(call split,:,$(PATH))))
```

这使用了三个 GMSL 函数：`uniq`、`split`（它将字符串按照某个分隔符字符拆分成列表；在此例中是冒号）和 `merge`（它将列表合并成一个字符串，列表项之间用一个字符分隔；在此例中是冒号）。

## 自动递增版本号

当软件发布时，能够自动增加版本号是非常方便的。假设一个项目包含一个名为 `version.c` 的文件，其中包含当前版本号的字符串：

```
char * ver = "1.0.0";
```

理想的情况是，只需输入 `make major-release`、`make minor-release` 或 `make dot-release`，并让版本号的三个部分之一自动更新，`version.c` 文件也随之更改。

下面是如何实现的：

```
   VERSION_C := version.c
   VERSION := $(shell cat $(VERSION_C))

   space :=
   space +=

   PARTS := $(call split,",$(subst $(space),,$(VERSION)))

   VERSION_NUMBER := $(call split,.,$(word 2,$(PARTS)))
   MAJOR := $(word 1,$(VERSION_NUMBER))
   MINOR := $(word 2,$(VERSION_NUMBER))
   DOT := $(word 3,$(VERSION_NUMBER))
   major-release minor-release dot-release:
➊ → @$(eval increment_name := $(call uc,$(subst -release,,$@)))
➋ → @$(eval $(increment_name) := $(call inc,$($(increment_name))))
➌ → @echo 'char * ver = "$(MAJOR).$(MINOR).$(DOT)";' > $(VERSION_C)
```

`VERSION` 变量包含 `version.c` 文件的内容，类似于 `char * ver = "1.0.0";`。`PARTS` 变量是一个列表，通过先去除 `VERSION` 中的所有空白字符，再按双引号分割来创建的。这将 `VERSION` 分割成 `char*ver= 1.0.0 ;` 这个列表。

所以 `PARTS` 是一个包含三个元素的列表，第二个元素是当前的版本号，它被提取到 `VERSION_NUMBER` 中，并转化为一个包含三个元素的列表：`1 0 0`。

接下来，从 `VERSION_NUMBER` 中提取名为 `MAJOR`、`MINOR` 和 `DOT` 的变量。如果 `version.c` 中的版本号是 `1.2.3`，那么 `MAJOR` 将是 `1`，`MINOR` 将是 `2`，`DOT` 将是 `3`。

最后，定义了三个规则，用于主版本、次版本和修订版发布。这些规则使用一些 `$(eval)` 技巧，利用相同的规则体来更新主版本、次版本或修订版号，具体取决于命令行中指定的 `major-release`、`minor-release` 或 `dot-release`。

为了理解它是如何工作的，可以跟随 `make minor-release` 的过程，假设当前版本号是 `1.0.0`。

`$(eval increment_name := $(call uc,$(subst -release,,$@)))` ➊ 首先使用 `$(subst)` 去除目标名称中的 `-release`（因此 `minor-release` 就变成了 `minor`）。

然后它调用 GMSL 的 `uc` 函数（该函数将字符串转换为大写），将 `minor` 转换为 `MINOR`。它将其存储在名为 `increment-name` 的变量中。这里是关键部分：`increment-name` 将用作要增加的变量的名称（`MAJOR`、`MINOR` 或 `DOT` 之一）。

在 ➋，`$(eval $(increment_name) := $(call inc,$($(increment_name))))` 实际上执行了这个工作。它使用 GMSL 的 `inc` 函数来增加存储在名为 `increment-name` 的变量中的值（注意 `$($(increment-name))`，它用于查找另一个变量中存储的变量名的值），然后将该值设置为增加后的值。

最后，它创建一个新的 `version.c` 文件，其中包含新的版本号 ➌。例如：

```
$ **make -n major-release**
echo 'char * ver = "2.0.0";' > version.c
$ **make -n minor-release**
echo 'char * ver = "1.1.0";' > version.c
$ **make -n dot-release**
echo 'char * ver = "1.0.1";' > version.c
```

这是使用 `-n` 选项时，从版本 1.0.0 开始并要求不同可能的发布版本的结果。

# GMSL 参考

本节是 GNU Make 标准库版本 1.1.7 的完整参考，涵盖了 GMSL 逻辑运算符、整数函数、列表、字符串和集合操作函数、关联数组和命名堆栈。对于每一类 GMSL 函数，你将看到函数的简介，接着是一个快速参考部分，列出了参数和返回值。要查看最新版本的完整参考，请访问 GMSL 网站：[`gmsl.sf.net/`](http://gmsl.sf.net/)。

如果你对高级 GNU `make`编程感兴趣，值得研究 GMSL 的源代码（特别是`__gmsl`文件）。创建各个 GMSL 函数时使用的技巧通常在其他情况下也很有用。

## 逻辑运算符

GMSL 有布尔值`$(true)`，它是一个非空字符串，实际上设置为单个字符`T`，以及`$(false)`，它是一个空字符串。你可以使用以下运算符与这些变量或返回这些值的函数一起使用。

尽管这些函数在返回值上始终是`$(true)`或`$(false)`，但它们对任何表示*真*的非空字符串都比较宽容。例如：

```
$(call or,$(wildcard /tmp/foo),$(wildcard /tmp/bar))
```

这会测试两个文件`/tmp/foo`和`/tmp/bar`中的任意一个是否存在，使用了`$(wildcard)`和 GMSL 的`or`函数。执行`$(wildcard /tmp/foo)`会返回`/tmp/foo`（如果文件存在），或者返回空字符串（如果文件不存在）。因此，`$(wildcard /tmp/foo)`的输出可以直接传递给`or`，其中`/tmp/foo`会被解释为*真*，空字符串则为*假*。

如果你更喜欢只使用`$(true)`和`$(false)`这样的值，可以像这样定义一个`make-bool`函数：

```
make-bool = $(if $(strip $1),$(true),$(false))
```

这将把任何非空字符串（去除空格后）转换为`$(true)`，而把空字符串（或仅包含空格的字符串）留作`$(false)`。`make-bool`在函数返回值中可能包含空格时非常有用。

例如，下面是一个小的 GNU `make`变量，如果当前月份是 1 月，它的值为`$(true)`：

```
january-now := $(call make-bool,$(filter Jan,$(shell date)))
```

这会运行`date` shell 命令，提取单词`Jan`，并通过`make-bool`将其转换为真值。像这样使用`$(filter)`会把`date`的结果当作一个列表，然后过滤掉列表中任何不是`Jan`的词。这种技术在其他情况下也可以用来提取字符串的部分内容。

你可以创建一个通用函数来判断一个列表中是否包含某个词：

```
contains-word = $(call make-bool,$(filter $1,$2))
january-now := $(call contains-word,Jan,$(shell date))
```

使用`contains-word`，你可以重新定义`january-now`。

### not

GMSL 包含所有常见的逻辑运算符。最简单的是`not`函数，它对其参数进行逻辑取反：

```
**not**

Argument: A single boolean value
Returns:  $(true) if the boolean is $(false) and vice versa
```

例如，`$(call not,$(true))`返回`$(false)`。

### and

`and`函数仅在其两个参数都为真时返回`$(true)`：

```
**and**

Arguments: Two boolean values
Returns:   $(true) if both of the arguments are $(true)
```

例如，`$(call and,$(true),$(false))`返回`$(false)`。

### or

`or`函数在其任一参数为真时返回`$(true)`：

```
**or**

Arguments: Two boolean values
Returns:   $(true) if either of the arguments is $(true)
```

例如，`$(call or,$(true),$(false))`返回`$(true)`。

### xor

`xor`函数是*异或*：

```
**xor**

Arguments: Two boolean values
Returns:   $(true) if exactly one of the booleans is true
```

例如，`$(call xor,$(true),$(false))` 返回 `$(true)`。

### nand

`nand` 就是 *非与*：

```
**nand**

Arguments: Two boolean values
Returns:   Value of 'not and'
```

例如，`$(call nand,$(true),$(false))` 返回 `$(true)`，而 `$(call and,$(true),$(false))` 返回 `$(false)`。

### nor

`nor` 就是 *非或*：

```
**nor**

Arguments: Two boolean values
Returns:   Value of 'not or'
```

例如，`$(call nor,$(true),$(false))` 返回 `$(false)`，而 `$(call or,$(true),$(false))` 返回 `$(true)`。

### xnor

很少使用的 `xnor` 是 *非异或*：

```
**xnor**

Arguments: Two boolean values
Returns:   Value of 'not xor'
```

请注意，GMSL 逻辑函数 `and` 和 `or` 不是 *短路*；这两个函数的两个参数会在执行逻辑 `and` 或 `or` 前展开。GNU `make` 3.81 引入了内建的 `and` 和 `or` 函数，它们是短路的：它们首先评估第一个参数，然后决定是否有必要评估第二个参数。

## 整数算术函数

在第五章中，您已经看到如何通过将非负整数表示为 `x` 的列表，在 GNU `make` 中执行算术运算。例如，4 是 `x x x x`。GMSL 使用相同的整数表示法，并提供广泛的函数来进行整数计算。

算术库函数有两种形式：一种形式的函数接受整数作为参数，另一种形式接受编码参数（由调用 `int_encode` 创建的 `x`）。例如，有两个 `plus` 函数：`plus`（使用整数参数调用，返回整数）和 `int_plus`（使用编码参数调用，返回编码结果）。

`plus` 比 `int_plus` 慢，因为它的参数和结果必须在 `x` 格式和整数之间转换。如果您进行复杂的计算，请使用带有输入单一编码和输出单一解码的 `int_*` 形式。对于简单的计算，您可以使用直接形式。

### int_decode

`int_decode` 函数接受一个 `x` 表示法的数字并返回它表示的十进制整数：

```
**int_decode**

Arguments: 1: A number in x-representation
Returns:   The integer for human consumption that is represented
           by the string of x's
```

### int_encode

`int_encode` 是 `int_decode` 的逆运算：它接受一个十进制整数并返回 `x` 表示法：

```
**int_encode**

Arguments: 1: A number in human-readable integer form
Returns:   The integer encoded as a string of x's
```

### int_plus

`int_plus` 在 `x` 表示法中将两个数字相加，并返回它们的和，以 `x` 表示法返回：

```
**int_plus**

Arguments: 1: A number in x-representation
           2: Another number in x-representation
Returns:   The sum of the two numbers in x-representation
```

### plus

要加法十进制整数，请使用 `plus` 函数，它会在 `x` 表示法和整数之间转换，并调用 `int_plus`：

```
**plus** (wrapped version of int_plus)

Arguments: 1: An integer
           2: Another integer
Returns:   The sum of the two integers
```

### int_subtract

`int_subtract` 在 `x` 表示法中减去两个数字，并返回它们的差值，仍然以 `x` 表示法返回：

```
**int_subtract**

Arguments: 1: A number in x-representation
           2: Another number in x-representation
Returns:   The difference of the two numbers in x-representation,
           or outputs an error on a numeric underflow
```

如果差值小于 0（无法表示），则会发生错误。

### subtract

要减去十进制整数，请使用 `subtract` 函数，它会在 `x` 表示法和整数之间转换，并调用 `int_subtract`：

```
**subtract** (wrapped version of int_subtract)

Arguments: 1: An integer
           2: Another integer
Returns:   The difference of the two integers, or outputs an error on a
           numeric underflow
```

如果差值小于 0（无法表示），则会发生错误。

### int_multiply

`int_multiply` 在 `x` 表示法中乘以两个数字：

```
**int_multiply**

Arguments: 1: A number in x-representation
           2: Another number in x-representation
Returns:   The product of the two numbers in x-representation
```

### multiply

`multiply` 将两个十进制整数相乘并返回它们的积。它会自动在 `x` 表示法和整数之间转换，并调用 `int_multiply`：

```
**multiply** (wrapped version of int_multiply)

Arguments: 1: An integer
           2: Another integer
Returns:   The product of the two integers
```

### int_divide

`int_divide` 将一个数字除以另一个；两个数字都以 `x` 表示法表示，结果也是如此：

```
**int_divide**
Arguments: 1: A number in x-representation
           2: Another number in x-representation
Returns:   The result of integer division of argument 1 divided
           by argument 2 in x-representation
```

### divide

`divide` 函数调用 `int_divide` 来除以两个十进制整数，自动进行 `x` 表示法的转换：

```
**divide** (wrapped version of int_divide)

Arguments: 1: An integer
           2: Another integer
Returns:   The integer division of the first argument by the second
```

### int_max 和 int_min

`int_max` 和 `int_min` 分别返回两个数字中的最大值和最小值，结果是 `x` 表示法：

```
**int_max**, **int_min**

Arguments: 1: A number in x-representation
           2: Another number in x-representation
Returns:   The maximum or minimum of its arguments in x-representation
```

### max 和 min

`int_max` 和 `int_min` 的十进制整数等价物分别是 `max` 和 `min`；它们会自动转换为 `x` 表示法并从中转换：

```
**max**, **min**

Arguments: 1: An integer
           2: Another integer
Returns:   The maximum or minimum of its integer arguments
```

### int_inc

`int_inc` 是一个小的辅助函数，它仅仅将一个 `x` 表示法的数字加一：

```
**int_inc**

Arguments: 1: A number in x-representation
Returns:   The number incremented by 1 in x-representation
```

### inc

`inc` 函数将一个十进制整数加一：

```
**inc**

Arguments: 1: An integer
Returns:   The argument incremented by 1
```

### int_dec

`int_inc` 的反操作是 `int_dec`：它将一个数字减去一：

```
**int_dec**

Arguments: 1: A number in x-representation
Returns:   The number decremented by 1 in x-representation
```

### dec

`dec` 函数将一个十进制整数减一：

```
**dec**

Arguments: 1: An integer
Returns:   The argument decremented by 1
```

### int_double

`double` 和 `halve` 函数（以及它们的 `int_double` 和 `int_halve` 等价函数）是为了性能考虑而提供的。如果你需要乘以二或除以二，这些函数的执行速度会比乘法和除法更快。

`int_double` 会将整数乘以二：

```
**int_double**

Arguments: 1: A number in x-representation
Returns:   The number doubled (* 2) and returned in x-representation
```

### double

`double` 会将一个十进制整数乘以二：

```
**double**

Arguments: 1: An integer
Returns:   The integer times 2
```

它在内部将其转换为 `x` 表示法并调用 `int_double`。

### int_halve

你可以通过对一个 `x` 表示法的数字调用 `int_halve` 来执行整数除以二的操作：

```
**int_halve**

Arguments: 1: A number in x-representation
Returns:   The number halved (/ 2) and returned in x-representation
```

### halve

最后是 `halve`：

```
**halve**

Arguments: 1: An integer
Returns:   The integer divided by 2
```

这是 `int_halve` 的十进制整数等价物。

## 整数比较函数

所有的整数比较函数返回 `$(true)` 或 `$(false)`：

```
**int_gt**, **int_gte**, **int_lt**, **int_lte**, **int_eq**, **int_ne**

Arguments: Two x-representation numbers to be compared
Returns:   $(true) or $(false)

int_gt  First argument is greater than second argument
int_gte First argument is greater than or equal to second argument
int_lt  First argument is less than second argument
int_lte First argument is less than or equal to second argument
int_eq  First argument is numerically equal to the second argument
int_ne  First argument is not numerically equal to the second argument
```

这些函数可以与 GNU `make` 和 GMSL 函数一起使用，也可以与需要布尔值的指令一起使用（如 GMSL 逻辑运算符）。

但是你更可能使用这些比较函数的版本：

```
**gt**, **gte**, **lt**, **lte**, **eq**, **ne**

Arguments: Two integers to be compared
Returns:   $(true) or $(false)

int_gt  First argument is greater than second argument
int_gte First argument is greater than or equal to second argument
int_lt  First argument is less than second argument
int_lte First argument is less than or equal to second argument
int_eq  First argument is numerically equal to the second argument
int_ne  First argument is not numerically equal to the second argument
```

这些函数作用于十进制整数，而不是 GMSL 使用的内部 `x` 表示法。

## 杂项整数函数

大多数情况下，你不需要做任何复杂的 GNU `make` 算术运算，但是这里详细介绍的杂项函数用于基本转换和数字序列的生成。有时它们会很有用。

### sequence

你可以使用 `sequence` 函数来生成一个数字序列：

```
**sequence**

Arguments: 1: An integer
           2: An integer
Returns:   The sequence [arg1 arg2] if arg1 >= arg2 or [arg2 arg1] if arg2 > arg1
```

例如，`$(call sequence,10,15)` 将会得到列表 `10 11 12 13 14 15`。要创建一个递减的序列，你只需要反转 `sequence` 的参数。例如，`$(call sequence,15,10)` 将会得到列表 `15 14 13 12 11 10`。

### dec2hex、dec2bin 和 dec2oct

`dec2hex`、`dec2bin` 和 `dec2oct` 函数用于在十进制数字和十六进制、二进制、八进制之间进行转换：

```
**dec2hex**, **dec2bin**, **dec2oct**

Arguments: 1: An integer
Returns:   The decimal argument converted to hexadecimal, binary or octal
```

例如，`$(call dec2hex,42)` 会得到 `2a`。

没有用于填充前导零的选项。如果需要，可以使用 GMSL 字符串函数。例如，下面是一个填充版的 `dec2hex`，它接受两个参数：一个十进制数字要转换为十六进制，以及输出的位数：

```
__repeat = $(if $2,$(call $0,$1,$(call rest,$2),$1$3),$3)

repeat = $(call __repeat,$1,$(call int_encode,$2),)
```

这个通过定义一些辅助函数来实现。首先，`repeat`会创建一个由若干个相同字符串组成的字符串。例如，`$(call repeat,10,A)` 将返回 `AAAAAAAAAA`。

这个定义中发生了一些微妙的事情。`repeat`函数会用三个参数调用`__repeat`：`$1`是要重复的字符串，`$2`是重复`$1`的次数，`$3`在`$(call)`调用`repeat`时通过尾随逗号被设置为空字符串。`$0`变量包含当前函数的名称；在`__repeat`中，它将是`__repeat`。

`__repeat`函数是递归的，并且使用`$2`作为递归的终止条件。`repeat`函数将所需的重复次数转换为 GMSL 算术函数使用的`x`表示法，并将其传递给`__repeat`。例如，`$(call repeat,Hello,5)` 会变成 `$(call __repeat,Hello,x x x x x,)`，然后`__repeat`会每次从`$2`中去掉一个`x`，直到`$2`为空。

使用`repeat`函数后，我们只需要一种方法来将字符串填充到指定的字符数，并用填充字符来填充。`pad`函数实现了这个功能：

```
pad = $(call repeat,$1,$(call subtract,$2,$(call strlen,$3)))$3

paddeddec2hex = $(call pad,0,$2,$(call dec2hex,$1))
```

它的三个参数分别是填充字符、填充后的输出宽度（字符数）和要填充的字符串。例如，`$(call pad,0,4,2a)` 将返回 `002a`。由此，可以轻松地定义一个填充后的`dec2hex`。它接受两个参数：第一个是要转换为十六进制的十进制数字，第二个是填充到的字符数。

正如你所预期的那样，`$(call paddeddec2hex,42,8)` 返回 `0000002a`。

## 列表操作函数

在 GNU `make`和 GMSL 中，列表是由空格分隔的字符字符串。GNU `make`内建的对列表操作的函数和 GMSL 函数都将多个空格视为一个空格。所以，`1 2 3`和`1 2 3`是相同的。

我将在接下来的几节中详细解释一些列表操作函数。这些函数在使用上比其他函数更为复杂，通常在函数式语言中可用。

### 将函数应用到列表上，使用 map

当你使用 GNU `make`函数（无论是内建的还是自定义的）时，实际上你是在一个简单的函数式语言中编程。在函数式编程中，常常会有一个`map`函数，它会将一个函数应用于列表中的每个元素。GMSL 定义了`map`来做到这一点。例如：

```
SRCS := src/FOO.c src/SUBMODULE/bar.c src/foo.c
NORMALIZED := $(call uniq,$(call map,lc,$(SRCS)))
```

给定一个包含文件名（可能带有路径）的列表`SRCS`，这将确保所有文件名都转为小写，并应用`uniq`函数来获取一个唯一的源文件列表。

这使用了 GMSL 函数`lc`来将`SRCS`中的每个文件名转为小写。你可以将`map`函数与内建函数和用户自定义函数一起使用。在这里，`NORMALIZED`将会是`src/foo.c src/submodule/bar.c`。

`map`的另一个使用场景是获取每个源文件的大小：

```
size = $(firstword $(shell wc -c $1))

SOURCE_SIZES := $(call map,size,$(SRCS))
```

在这里我们定义了一个`size`函数，它使用`$(shell)`来调用`wc`，然后我们将其应用到`SRCS`中的每个文件。

这里的 `SOURCE_SIZES` 可能是类似 `1538 1481` 的内容，每个源文件对应一个元素。

### 创建一个 reduce 函数

在函数式编程语言中，另一个常见的函数是 `reduce`。`reduce` 对列表的连续元素应用一个接受两个参数的函数，并将该函数的返回值作为参数传递给下一个调用。GMSL 没有内置的 `reduce` 函数，但你可以很容易地定义一个：

```
reduce = $(if $2,$(call $0,$1,$(call rest,$2),$(call $1,$3,$(firstword $2))),$3)
```

### 使用 reduce 对数字列表求和

将 `reduce` 与 `plus` 函数结合使用，你可以轻松创建一个 GNU `make` 函数来对数字列表求和：

```
sum-list = $(call reduce,plus,$1,0)
```

`sum-list` 函数接受一个参数，即一个数字列表，并返回这些数字的总和。它将三个参数传递给 `reduce`：每个列表元素调用的函数名称（在此为 `plus`），数字列表，以及一个起始值（在此为 `0`）。

下面是它的工作原理。假设调用了 `$(call sum-list,1 2 3 4 5)`。接下来会依次调用 `plus` 函数：

```
$(call plus,1,0) which returns 1
$(call plus,1,2) which returns 3
$(call plus,3,3) which returns 6
$(call plus,6,4) which returns 10
$(call plus,10,5) which returns 15
```

第一次调用使用列表的第一个元素和起始值 `0`。每一次后续的调用使用列表中的下一个元素和上次调用 `plus` 函数的结果。

你可以将 `sum-list` 与 `SOURCE_SIZES` 变量结合使用，以获取源代码的总大小：

```
TOTAL_SIZE := $(call sum-list,$(SOURCE_SIZES))
```

在这种情况下，`TOTAL_SIZE` 会是 `3019`。

### 对一对列表映射函数

GMSL 为列表定义的另一个有趣的函数是 `pairmap`。它接受三个参数：两个列表（它们应该有相同的长度）和一个函数。该函数依次作用于每个列表的第一个元素、第二个元素、第三个元素，依此类推。

假设 `SRCS` 包含一个源文件列表。使用我们定义的 `size` 函数，结合 `map`，我们定义了 `SOURCE_SIZES`，它包含了每个源文件的大小列表。通过使用 `pairmap`，我们可以将这两个列表压缩在一起，输出每个文件的名称及其大小：

```
zip = $1:$2

SOURCES_WITH_SIZES := $(call pairmap,zip,$(SRCS),$(SOURCE_SIZES))
```

`zip` 函数依次作用于每个源文件名和文件大小，并生成一个用冒号分隔文件名和文件大小的字符串。使用我们在本节中的示例文件和大小，`SOURCES_WITH_SIZES` 可能会是 `src/foo.c:1538 src/submodule/bar.c:1481`。

### first

`first` 函数接收一个列表并返回其第一个元素：

```
**first**

Arguments: 1: A list
Returns:   Returns the first element of a list
```

请注意，`first` 与 GNU `make` 函数 `$(firstword)` 是相同的。

### last

`last` 函数返回列表的最后一个元素：

```
**last**

Arguments: 1: A list
Returns:   The last element of a list
```

GNU `make` 3.81 引入了 `$(lastword)`，它的工作方式与 `last` 相同。

### 其余部分

`rest` 函数几乎是 `first` 的相反。它返回列表中的所有元素，除了第一个元素：

```
**rest**

Arguments: 1: A list
Returns:   The list with the first element removed
```

### chop

要移除列表中的最后一个元素，请使用 `chop` 函数：

```
**chop**

Arguments: 1: A list
Returns:   The list with the last element removed
```

### map

`map` 函数遍历一个列表（它的第二个参数），并对每个列表元素调用一个函数（函数名在第一个参数中）。每次调用该函数时返回的值将组成一个列表，并返回该列表：

```
**map**
Arguments: 1: Name of function to $(call) for each element of list
           2: List to iterate over calling the function in 1
Returns:   The list after calling the function on each element
```

### pairmap

`pairmap` 类似于 `map`，但它遍历一对列表：

```
**pairmap**

Arguments: 1: Name of function to $(call) for each pair of elements
           2: List to iterate over calling the function in 1
           3: Second list to iterate over calling the function in 1
Returns:   The list after calling the function on each pair of elements
```

第一个参数中的函数被调用时，会传入两个参数：来自每个被迭代列表的一个元素。

### leq

`leq` 列表相等性测试函数会正确地为完全相同的列表返回 `$(true)`，即使它们仅因空格不同而有所差异：

```
**leq**

Arguments: 1: A list to compare against...
           2: ...this list
Returns:   $(true) if the two lists are identical
```

例如，`leq` 会认为 `1 2 3` 和 `1 2 3` 是相同的列表。

### lne

`lne` 是 `leq` 的反操作：当两个列表不相等时，它返回 `$(true)`：

```
**lne**

Arguments: 1: A list to compare against...
           2: ...this list
Returns:   $(true) if the two lists are different
```

### reverse

将列表 `reverse` 反转可能是有用的（特别是因为它可以作为输入传递给 `$(foreach)` 并反向迭代）。

```
**reverse**

Arguments: 1: A list to reverse
Returns:   The list with its elements in reverse order
```

### uniq

内建的 `$(sort)` 函数会去重列表，但它会在排序的同时进行去重。而 GMSL 的 `uniq` 函数则会去重列表，同时保留元素第一次出现的顺序：

```
**uniq**

Arguments: 1: A list to deduplicate
Returns:   The list with elements in the original order but without duplicates
```

例如，`$(call uniq,a c b a c b)` 将返回 `a c b`。

### length

要找出列表中的元素数量，可以调用 `length`：

```
**length**

Arguments: 1: A list
Returns:   The number of elements in the list
```

`length` 函数与 GNU `make $(words)` 函数相同。

## 字符串操作函数

字符串是由任何字符组成的序列，包括空格。字符串相等性（和字符串不等式）函数 `seq` 即使处理包含空格或仅由空格组成的字符串时也能正常工作。例如：

```
# space contains the space character

space :=
space +=

# tab contains a tab

tab :=→  # needed to protect the tab character

$(info $(call seq,White Space,White Space))
$(info $(call seq,White$(space)Space,White Space))
$(info $(call sne,White$(space)Space,White$(tab)Space))
$(info $(call seq,$(tab),$(tab)))
$(info $(call sne,$(tab),$(space)))
```

这将输出 `T` 五次，表示每次调用 `seq` 或 `sne` 都返回了 `$(true)`。

与列表操作函数类似，我将在接下来的部分详细介绍一些更复杂的函数。

### 将 CSV 数据拆分成 GNU make 列表

你可以使用 `split` 函数将 CSV 格式的值转换为 GNU `make` 列表。例如，以逗号为分隔符将 CSV 行分割成一个列表，然后可以从中提取各个项：

```
CSV_LINE := src/foo.c,gcc,-Wall

comma := ,
FIELDS := $(call split,$(comma),$(CSV_LINE))

$(info Compile '$(word 1,$(FIELDS))' using compiler '$(word 2,$(FIELDS))' with \
options '$(word 3,$(FIELDS))')
```

注意变量 `comma` 如何被定义为包含逗号字符，以便它可以在 `$(call)` 中传递给 `split` 函数。这个技巧在第一章中有讨论。

### 从目录列表创建 PATH

`merge` 函数的作用与 `split` 相反：它通过某个字符分隔列表项，将列表转化为一个字符串。例如，要将一个目录列表转换为适合 `PATH` 的格式（通常由冒号分隔），可以按如下方式定义 `list-to-path`：

```
DIRS := /usr/bin /usr/sbin /usr/local/bin /home/me/bin

list-to-path = $(call merge,:,$1)

$(info $(call list-to-path,$(DIRS)))
```

这将输出 `/usr/bin:/usr/sbin:/usr/local/bin:/home/me/bin`。

### 使用 `tr` 转换字符

最复杂的字符串函数是 `tr`，它的操作方式类似于 `tr` shell 程序。它将一个字符集中的每个字符转换为第二个列表中的相应字符。GMSL 为 `tr` 定义了一些常见的字符类。例如，它定义了名为 `[A-Z]` 和 `[a-z]` 的变量（是的，它们真的是这个名字），分别包含大写字母和小写字母。

我们可以使用 `tr` 创建一个函数，将其转换为黑客语言（leet-speak）：

```
leet = $(call tr,A E I O L T,4 3 1 0 1 7,$1)

$(info $(call leet,I AM AN ELITE GNU MAKE HAXOR))
```

这将输出 `1 4M 4N 31173 GNU M4K3 H4X0R`。

### seq

命名略显混乱的 `seq` 函数测试两个字符串是否相等：

```
**seq**

Arguments: 1: A string to compare against...
           2: ...this string
Returns:   $(true) if the two strings are identical
```

### sne

相反的字符串不等式可以通过 `sne` 来测试：

```
**sne**

Arguments: 1: A string to compare against...
           2: ...this string
Returns:   $(true) if the two strings are not the same
```

### streln

`length`函数获取列表的长度；对于字符串，等效的函数是`strlen`：

```
**strlen**

Arguments: 1: A string
Returns:   The length of the string
```

### substr

可以使用`substr`函数提取子字符串：

```
**substr**

Arguments: 1: A string
           2: Starting offset (first character is 1)
           3: Ending offset (inclusive)
Returns:   A substring
```

注意，在 GMSL 中，字符串从位置 1 开始，而不是 0。

### split

要将字符串分割成列表，可以使用`split`函数：

```
**split**

Arguments: 1: The character to split on
           2: A string to split
Returns:   A list separated by spaces at the split character in the
           first argument
```

注意，如果字符串包含空格，结果可能不符合预期。GNU `make`使用空格作为列表分隔符，使得同时处理空格和列表变得非常困难。有关 GNU `make`如何处理空格的更多信息，请参见第四章。

### merge

`merge`是`split`的相反操作。它接受一个列表，并在每个列表元素之间插入一个字符输出字符串：

```
**merge**

Arguments: 1: The character to put between fields
           2: A list to merge into a string
Returns:   A single string, list elements are separated by the character in
           the first argument
```

### tr

使用`tr`函数可以转换单个字符，它是创建`uc`和`lc`函数的构建块：

```
**tr**

Arguments: 1: The list of characters to translate from 
           2: The list of characters to translate to
           3: The text to translate
Returns:   The text after translating characters
```

### uc

`uc`对字母 a-z 进行简单的大写转换：

```
**uc**

Arguments: 1: Text to uppercase
Returns:   The text in uppercase
```

### lc

最后，我们有了`lc`：

```
**lc**

Arguments: 1: Text to lowercase
Returns:   The text in lowercase
```

该函数对字母 A-Z 进行简单的小写转换。

## 集合操作函数

集合通过排序去重的列表表示。要从列表中创建集合，可以使用`set_create`，或者从`empty_set`开始并使用`set_insert`插入各个元素。空集合由变量`empty_set`定义。

例如，一个 makefile 可以使用在创建目录中讨论的标记技术来跟踪它创建的所有目录：

```
MADE_DIRS := $(empty_set)

marker = $1.f
make_dir = $(eval $1.f: ; @$$(eval MADE_DIRS := $$(call      \
set_insert,$$(dir $$@),$$(MADE_DIRS))) mkdir -p $$(dir $$@); \
touch $$@)

all: $(call marker,/tmp/foo/) $(call marker,/tmp/bar/)
→  @echo Directories made: $(MADE_DIRS)

$(call make_dir,/tmp/foo/)
$(call make_dir,/tmp/bar/)
```

通过在`make_dir`函数（用于创建目录的规则）中调用`set_insert`，意味着变量`MADE_DIRS`将跟踪已创建的目录集合。

在一个真实的 makefile 中，可能会构建许多目录，使用集合是一种简单的方式来发现任何时刻哪些目录已经被构建。

注意，由于集合是作为 GNU `make`列表实现的，因此无法插入包含空格的项目。

### set_create

你可以通过使用`set_create`函数来创建一个集合：

```
**set_create**

Arguments: 1: A list of set elements
Returns:   The newly created set
```

它接受一个元素列表并将它们添加到集合中。集合本身会被返回。注意，集合元素不能包含空格。

### set_insert

一旦通过`set_create`创建了集合，可以使用`set_insert`向其中添加元素：

```
**set_insert**

Arguments: 1: A single element to add to a set
           2: A set
Returns:   The set with the element added
```

### set_remove

要从集合中移除一个元素，可以调用`set_remove`：

```
**set_remove**

Arguments: 1: A single element to remove from a set
           2: A set
Returns:   The set with the element removed
```

从集合中移除一个元素时，如果该元素不存在，则不会报错。

### set_is_member

要测试一个元素是否是集合的成员，可以调用`set_is_member`。它返回一个布尔值，指示该元素是否存在：

```
**set_is_member**

Arguments: 1: A single element
           2: A set
Returns:   $(true) if the element is in the set
```

### set_union

通过对两个集合调用`set_union`函数，你可以将两个集合合并。合并后的集合会被返回：

```
**set_union**

Arguments: 1: A set
           2: Another set
Returns:   The union of the two sets
```

### set_intersection

要确定两个集合的共同元素，可以使用`set_intersection`。它返回作为参数传入的两个集合中都存在的元素集合：

```
**set_intersection**

Arguments: 1: A set
           2: Another set
Returns:   The intersection of the two sets
```

### set_is_subset

有时，了解一个集合是否是另一个集合的子集是很有用的，可以通过调用`set_is_subset`来进行测试：

```
**set_is_subset**

Arguments: 1: A set
           2: Another set
Returns:   $(true) if the first set is a subset of the second
```

`set_is_subset`返回一个布尔值，指示第一个集合是否是第二个集合的子集。

### set_equal

要确定两个集合是否相等，请调用`set_equal`：

```
**set_equal**

Arguments: 1: A set
           2: Another set
Returns:   $(true) if the two sets are identical
```

`set_equal`返回`$(true)`，如果两个集合具有完全相同的元素。

## 关联数组

一个*关联数组*将一个键值（没有空格的字符串）映射到一个单一的值（任意字符串）。关联数组有时也被称为映射（maps）或哈希表（尽管那是一个实现细节，GMSL 的关联数组并不使用哈希）。

你可以使用关联数组作为*查找表*。例如：

```
C_FILES := $(wildcard *.c)

get-size = $(call first,$(shell wc -c $1))
$(foreach c,$(C_FILES),$(call set,c_files,$c,$(call get-size,$c)))

$(info All the C files: $(call keys,c_files))
$(info foo.c has size $(call get,c_files,foo.c))
```

这个小的 Makefile 获取当前目录中所有 `.c` 文件及其大小的列表，然后将文件名和大小之间建立关联数组映射。

`get-size`函数使用`wc`获取文件中的字节数。`C_FILES`变量包含当前目录中的所有`.c`文件。`$(foreach)`使用 GMSL 的`set`函数在名为`c_files`的关联数组中设置每个`.c`文件及其大小的映射。

以下是一个示例运行：

```
$ **make**
All the C files: bar.c foo.c foo.c
has size 551
```

第一行是所有`.c`文件的列表；它是通过`keys`函数获取关联数组中的所有键来打印的。第二行是通过使用`get`查找`foo.c`的长度来得到的。

### set

GMSL 会跟踪命名的关联数组，但不需要显式创建它们。只需调用`set`来添加元素到数组中，如果数组不存在，它会自动创建。请注意，数组的键不能包含空格。

```
**set**

Arguments: 1: Name of associative array
           2: The key value to associate
           3: The value associated with the key
Returns:   Nothing
```

### 获取

要从关联数组中检索项，请调用`get`。如果键不存在，`get`将返回一个空字符串。

```
**get**

Arguments: 1: Name of associative array
           2: The key to retrieve
Returns:   The value stored in the array for that key
```

### 键

`keys`函数返回关联数组中所有键的列表。你可以使用它和`$(foreach)`来遍历关联数组：

```
**keys**

Arguments: 1: Name of associative array
Returns:   A list of all defined keys in the array
```

### defined

要测试某个键是否存在于关联数组中，请调用`defined`：

```
**defined**

Arguments: 1: Name of associative array
           2: The key to test
Returns:   $(true) if the key is defined (i.e., not empty)
```

`defined`返回一个布尔值，表示键是否已定义。

## 命名栈

一个*栈*是一个有序的字符串列表（其中不包含空格）。在 GMSL 中，栈是内部存储的，并且它们有名称，像关联数组一样。例如：

```
traverse-tree = $(foreach d,$(patsubst %/.,%,$(wildcard $1/*/.)),  \
$(call push,dirs,$d)$(call traverse-tree,$d))

$(call traverse-tree,sources)

dump-tree = $(if $(call sne,$(call depth,dirs),0),$(call pop,dirs) \
$(call dump-tree))

$(info $(call dump-tree))
```

这个小的 Makefile 使用栈来跟踪目录树。

### traverse-tree

`traverse-tree`函数使用`$(wildcard)`函数查找其参数（存储在`$1`中）中的所有子目录，寻找始终存在于目录中的`.`文件。它使用`$(patsubst)`函数去除每个由`$(wildcard)`返回的值中的尾部`/.`，以获得完整的目录名。

在遍历该目录之前，它会将找到的目录推送到名为`dirs`的栈中。

### dump-tree

`dump-tree`函数会从`dirs`树中逐个弹出元素，直到没有剩余的元素（直到`depth`变为`0`）。

示例 6-1 展示了一个目录结构。

示例 6-1. 目录结构

```
$ **ls -R sources**
sources:
bar  foo

sources/bar:
barsub
sources/bar/barsub:

sources/foo:
subdir  subdir2

sources/foo/subdir:
subsubdir

sources/foo/subdir/subsubdir:

sources/foo/subdir2:
```

如果这个目录结构存在于`sources`下，Makefile 将输出：

```
sources/foo sources/foo/subdir2 sources/foo/subdir sources/foo/subdir/
subsubdir sources/bar sources/bar/barsub
```

如果希望以深度优先的方式遍历目录树，可以使用栈函数来定义 `dfs`，它会搜索目录树并构建包含目录的深度优先顺序的 `dirs` 栈：

```
__dfs = $(if $(call sne,$(call depth,work),0),$(call push,dirs,$(call    \
peek,work)$(foreach d,$(patsubst %/.,%,$(wildcard $(call                 \
pop,work)/*/.)),$(call push,work,$d)))$(call __dfs))

dfs = $(call push,work,$1)$(call __dfs)

$(call dfs,sources)

dump-tree = $(if $(call sne,$(call depth,dirs),0),$(call pop,dirs) $(call \
dump-tree))

$(info $(call dump-tree,dirs))
```

`dump-tree` 函数没有变化（它通过多次调用 `pop` 来输出栈中的所有内容）。但 `dfs` 函数是新的。它使用一个名为 `work` 的工作栈来跟踪待访问的目录。它首先将起始目录推送到 `work` 栈中，然后调用 `__dfs` 辅助函数。

实际工作由 `__dfs` 完成。它将当前目录推送到 `dirs` 栈中，将该目录的所有子目录推送到 `work` 栈中，然后递归。当 `work` 栈为空时，递归停止。

对于目录结构的输出，参见 示例 6-1 现在是：

```
sources/bar/barsub sources/bar sources/foo/subdir/subsubdir sources/foo/subdir
sources/foo/subdir2 sources/foo sources.
```

### 推送

任何使用过栈的人都对推入和弹出元素非常熟悉。GMSL 栈函数非常相似。要将元素添加到栈顶，调用 `push`：

```
**push**

Arguments: 1: Name of stack
           2: Value to push onto the top of the stack (must not contain
           a space)
Returns:   None
```

### 弹出

要获取栈顶元素，调用 `pop`：

```
**pop**

Arguments: 1: Name of stack
Returns:   Top element from the stack after removing it
```

### 查看

`peek` 函数的作用类似于 `pop`，但不会移除栈顶元素；它只返回该元素的值：

```
**peek**

Arguments: 1: Name of stack
Returns:   Top element from the stack without removing it
```

### 深度

最后，你可以调用 `depth`：

```
**depth**

Arguments: 1: Name of stack
Returns:   Number of items on the stack
```

`depth` 确定栈中当前有多少个元素。

## 函数记忆化

为了减少对慢速函数（如 `$(shell)`）的调用，提供了一个单一的记忆化函数。例如，假设一个 Makefile 需要知道各种文件的 MD5 值，并定义了一个 `md5` 函数。

```
md5 = $(shell md5sum $1)
```

这是一个相当昂贵的函数调用（因为 `md5sum` 执行时会消耗时间），因此希望每个文件只调用一次。`md5` 函数的记忆化版本如下所示：

```
md5once = $(call memoize,md5,$1)
```

它会对每个输入的文件名仅调用一次 `md5sum` 函数，并将返回的值内部记录，以便后续对相同文件名的 `md5once` 调用可以直接返回 MD5 值，而无需重新运行 `md5sum`。例如：

```
$(info $(call md5once,/etc/passwd))
$(info $(call md5once,/etc/passwd))
```

这会打印出 `/etc/passwd` 的 MD5 值两次，但仅执行一次 `md5sum`。

实际的 `memoize` 函数是使用 GMSL 关联数组函数定义的：

```
**memoize**

Arguments: 1: Name of function to memoize
           2: String argument for the function
Returns:   Result of $1 applied to $2 but only calls $1 once for each unique $2
```

## 杂项和调试功能

表 6-1 显示了 GMSL 定义的常量。

表 6-1. GMSL 常量

| 常量 | 值 | 目的 |
| --- | --- | --- |
| true | T | 布尔值 true |
| false | (一个空字符串) | 布尔值 false |
| gmsl_version | 1 1 7 | 当前 GMSL 版本号（主版本号、次版本号、修订号） |

你可以像访问普通 GNU `make` 变量一样，通过将它们包裹在 `$()` 或 `${}` 中来访问这些常量。

### gmsl_compatible

你已经在 检查 GMSL 版本 中了解了 `gmsl_compatible` 函数：

```
**gmsl_compatible**

Arguments: List containing the desired library version number (major minor
           revision)
Returns:   $(true) if the current version of the library is compatible
           with the requested version number, otherwise $(false)
```

在第一章中，你看到了一个使用模式规则和目标`print-%`输出变量值的示例。由于这是一个非常有用的规则，GMSL 定义了自己的`gmsl-print-%`目标，你可以用它来打印任何在包含 GMSL 的 makefile 中定义的变量的值。

例如：

```
include gmsl

FOO := foo bar baz
all:
```

### gmsl-print-%

`gmsl-print-%`可以用来打印任何 makefile 变量，包括 GMSL 内部的变量。例如，`make gmsl-print-gmsl_version`会打印当前的 GMSL 版本。

```
**gmsl-print-%** (target not a function)

Arguments: The % should be replaced by the name of a variable that you
           wish to print
Action:    Echoes the name of the variable that matches the % and its value
```

### assert

如 Makefile 断言中所讨论的，makefile 中的断言是很有用的。GMSL 提供了两个断言函数：`assert`和`assert_exists`。

```
**assert**

Arguments: 1: A boolean that must be true or the assertion will fail
           2: The message to print with the assertion
Returns:   None
```

### assert_exists

要断言某个文件或目录存在，GMSL 提供了`assert_exists`函数：

```
**assert_exists**

Arguments: 1: Name of file that must exist, if it is missing an assertion
           will be generated
Returns:   None
```

## 环境变量

表 6-2 显示了 GMSL 环境变量（或命令行覆盖项），这些变量控制着各种功能。

表 6-2. GMSL 环境变量

| 变量 | 目的 |
| --- | --- |
| GMSL_NO_WARNINGS | 如果设置了，防止 GMSL 输出警告信息。例如，算术函数可能会生成下溢警告。 |
| GMSL_NO_ERRORS | 如果设置了，防止 GMSL 生成致命错误：例如除零错误或断言失败都会被认为是致命的。 |
| GMSL_TRACE | 启用函数追踪。调用 GMSL 函数时，会追踪函数名和参数。有关 makefile 追踪的讨论，请参见追踪变量值。 |

这些环境变量都可以在环境中或命令行中设置。

例如，这个 makefile 包含一个总是失败的断言，导致`make`过程停止：

```
include gmsl

$(call assert,$(false),Always fail)

all:
```

设置`GMSL_NO_ERRORS`可以防止断言停止`make`过程。在这种情况下，`assert`的输出会被隐藏，`make`会正常继续：

```
$ **make**
Makefile:5: *** GNU Make Standard Library: Assertion failure: Always fail.
Stop.
$ **make GMSL_NO_ERRORS=1**
make: Nothing to be done for `all'.
```

在 makefile 中放置一些适当的 GMSL 断言可以产生很大的效果。通过检查 makefile 的前提条件（比如特定文件的存在，或者编译器的版本号），一个有责任心的 makefile 编写者可以在不迫使用户调试`make`中常常晦涩的输出信息的情况下，提醒用户潜在的问题。
