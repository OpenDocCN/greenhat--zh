# 第四章：陷阱与问题

在本章中，你将学习如何应对随着项目规模扩大，makefile 维护者所面临的问题。那些在小型 makefile 中看似简单的任务，在大型的、可能是递归的 `make` 进程中会变得更加困难。随着 makefile 变得更加复杂，容易遇到一些边缘情况的问题，或者 GNU `make` 的行为难以理解的情况。

在这里，你将看到解决“递归 `make` 问题”的完整方案，如何克服 GNU `make` 处理包含空格的文件名的问题，如何处理跨平台的文件路径等等。

# GNU make 注意事项：ifndef 和 ?=

检查变量是否已定义的两种方式 `ifndef` 和 `?=` 很容易让人迷惑，因为它们做的是相似的事情，但一个名字具有误导性。`ifndef` 并不真正检查变量是否已定义，它只检查变量是否为空，而 `?=` 则根据变量是否已定义来做决定。

比较以下两种在 makefile 中有条件设置变量 `FOO` 的方式：

```
ifndef FOO
FOO=New Value
endif
```

和

```
FOO ?= New Value
```

它们看起来应该做相同的事情，实际上它们差不多。

## `?=` 的作用

GNU `make` 中的 `?=` 运算符将其左侧的变量设置为右侧的值，前提是左侧的变量尚未定义。例如：

```
FOO ?= New Value
```

这个 makefile 将 `FOO` 设置为 `New Value`。

但以下内容则不返回此值：

```
FOO=Old Value
FOO ?= New Value
```

即使 `FOO` 最初为空，这个也不会返回：

```
FOO=
FOO ?= New Value
```

实际上，`?=` 与以下 makefile 是相同的，它使用 GNU `make $(origin)` 函数来判断变量是否未定义：

```
ifeq ($(origin FOO),undefined)
FOO = New Value
endif
```

`$(origin FOO)` 将返回一个字符串，显示 `FOO` 是否以及如何定义。如果 `FOO` 未定义，则 `$(origin FOO)` 的值为 `undefined`。

请注意，使用 `?=` 定义的变量会像使用 `=` 运算符定义的变量一样进行展开。它们在使用时会展开，但在定义时不会展开，就像普通的 GNU `make` 变量一样。

## `ifndef` 的作用

如前所述，`ifndef` 测试变量是否为空，但并不检查变量是否已定义。`ifndef` 意味着 *如果变量未定义或已定义但为空*。因此，以下内容：

```
ifndef FOO
FOO=New Value
endif
```

如果 `FOO` 未定义或 `FOO` 为空，则会将 `FOO` 设置为 `New Value`。因此，`ifndef` 可以重写为：

```
ifeq ($(FOO),)
FOO=New Value
endif
```

因为未定义的变量在读取时总是被视为具有空值。

# $(shell) 和 := 一起使用

本节中的建议通常通过适当放置冒号来加速 makefile 的执行。要理解一个冒号如何带来如此大的变化，你需要了解 GNU `make` 的 `$(shell)` 函数以及 `=` 和 `:=` 之间的区别。

## $(shell) 解释

`$(shell)` 是 GNU `make` 中与 shell 中反引号（`` ` ``）操作符相对应的函数。它执行一个命令，将结果展平（把所有空白字符，包括换行符，转换为空格），并返回最终的字符串。

例如，如果你想将`date`命令的输出存入一个名为`NOW`的变量，你可以这样写：

```
NOW = $(shell date)
```

如果你想统计当前目录中的文件数量，并将该数量存入`FILE_COUNT`，可以这样做：

```
FILE_COUNT = $(shell ls | wc -l )
```

因为`$(shell)`会将输出展平，获取当前目录中所有文件的名称并将其存入一个变量，以下方法有效：

```
FILES = $(shell ls)
```

文件之间的换行符被替换为一个空格，使得`FILES`成为一个用空格分隔的文件名列表。

常见的做法是执行`pwd`命令，将当前工作目录存入一个变量（在此例中为`CWD`）：

```
CWD = $(shell pwd)
```

我们稍后会查看`pwd`命令，考虑如何优化一个示例 makefile，避免重复多次获取当前工作目录。

## `=`和`:=`的区别

百分之九十九的情况下，你会看到在 makefile 中使用`=`形式的变量定义，像这样：

```
   FOO = foo
   BAR = bar
   FOOBAR = $(FOO) $(BAR)

   all: $(FOOBAR)
➊ $(FOOBAR):
   →  @echo $@ $(FOOBAR)

   FOO = fooey
   BAR = barney
```

在这里，变量`FOO`、`BAR`和`FOOBAR`是*递归展开*的变量。这意味着，当需要一个变量的值时，任何它引用的变量都会在此时展开。例如，如果需要`$(FOOBAR)`的值，GNU `make`会获取`$(FOO)`和`$(BAR)`的值，将它们合并并在中间加上空格，最终返回`foo bar`。通过必要的多级变量展开会在变量使用时完成。

在这个 makefile 中，`FOOBAR`有两个不同的值。运行它会输出：

```
$ **make**
foo fooey barney
bar fooey barney
```

`FOOBAR`的值用于定义`all`规则的先决条件列表，并被展开为`foo bar`；同样的事情也发生在下一个规则➊中，该规则定义了`foo`和`bar`的规则。

但是当规则被*执行*时，在`echo`中使用的`FOOBAR`的值会产生`fooey barney`。（你可以通过查看`$@`的值来验证，在规则定义时`FOOBAR`的值是`foo bar`，`$@`是正在构建的目标，规则执行时可以查看它的值）。

请记住以下两种情况：

+   当在 makefile 中定义规则时，变量会评估为*那个时刻*在 makefile 中的值。

+   在配方中使用的变量（即在命令中）会有最终的值：无论变量在 makefile 的末尾时是什么值。

如果将`FOOBAR`的定义改为使用`:=`而不是`=`，运行 makefile 将产生完全不同的结果：

```
$ **make**
foo foo bar
bar foo bar
```

现在`FOOBAR`在所有地方都有相同的值。这是因为`:=`强制在 makefile 解析时立刻展开定义的右侧内容。GNU `make`没有将`$(FOO) $(BAR)`作为`FOOBAR`的定义，而是存储了`$(FOO) $(BAR)`的展开结果，在那个时刻就是`foo bar`。即使后来在 makefile 中重新定义了`FOO`和`BAR`，也不影响结果；`FOOBAR`已经被展开并设置为固定字符串。GNU `make`将这种方式定义的变量称为*简单展开*。

一旦一个变量变为简单展开变量，它就会保持这种状态，除非通过`=`操作符重新定义。这意味着当文本追加到一个简单展开变量时，它会在添加到变量之前进行展开。

例如，这个：

```
FOO=foo
BAR=bar
BAZ=baz
FOOBAR := $(FOO) $(BAR)
FOOBAR += $(BAZ)
BAZ=bazzy
```

导致`FOOBAR`变为`foo bar baz`。如果使用`=`而不是`:=`，当`$(BAZ)`被追加时，它不会被展开，结果是`FOOBAR`将变为`foo baz bazzy`。

## `=`的隐性成本

看看这个示例的 makefile：

```
CWD = $(shell pwd)
SRC_DIR=$(CWD)/src/
OBJ_DIR=$(CWD)/obj/
OBJS = $(OBJ_DIR)foo.o $(OBJ_DIR)bar.o $(OBJ_DIR)baz.o

$(OBJ_DIR)%.o: $(SRC_DIR)%.c ; @echo Make $@ from $<

all: $(OBJS)
→  @echo $? $(OBJS)
```

它将当前工作目录获取到`CWD`中，定义源目录和目标目录为`CWD`的子目录，定义一组对象（`foo.o`、`bar.o`和`baz.o`），将在`OBJ_DIR`中构建，设置一个模式规则，展示如何从`.c`文件构建`.o`文件，最后声明默认情况下，makefile 应构建所有对象并打印出那些过时的对象的列表（`$?`是过时规则的前提条件列表），以及所有对象的完整列表。

你可能会惊讶地发现，这个 makefile 最终只为了获取`CWD`值就进行了八次 shell 调用。试想一下，在一个包含成百上千个对象的真实 makefile 中，GNU `make`会进行多少次耗时的 shell 调用！

由于 makefile 使用了递归展开变量（即变量的值在使用时确定，而不是在定义时确定），因此会进行许多`$(shell)`调用：`OBJS`引用`OBJ_DIR`三次，每次都引用`CWD`；每次引用`OBJS`时，都会对`$(shell pwd)`进行三次调用。任何对`SRC_DIR`或`OBJ_DIR`的引用（例如，模式规则定义）都会导致另一次`$(shell pwd)`调用。

但一个快速的解决方法是将`CWD`的定义更改为通过插入`:`来展开，将`=`变为`:=`。因为工作目录在`make`过程中不会改变，所以我们可以安全地获取一次：

```
CWD := $(shell pwd)
```

现在，通过对 shell 进行一次调用来获取工作目录。在真实的 makefile 中，这可能是一个巨大的节省时间的办法。

因为在 makefile 中追踪变量的使用可能会很困难，你可以使用一个简单的技巧，使`make`打印出变量展开的确切位置。在`CWD`的定义中插入`$(warning Call to shell)`，使其定义变为如下：

```
CWD = **$(warning Call to shell)**$(shell pwd)
```

然后，当你运行`make`时，会得到以下输出：

```
$ **make**
makefile:8: Call to shell
makefile:8: Call to shell
makefile:10: Call to shell
makefile:10: Call to shell
makefile:10: Call to shell
Make /somedir/obj/foo.o from /somedir/src/foo.c
Make /somedir/obj/bar.o from /somedir/src/bar.c
Make /somedir/obj/baz.o from /somedir/src/baz.c
makefile:11: Call to shell
makefile:11: Call to shell
makefile:11: Call to shell
/somedir/obj/foo.o /somedir/obj/bar.o /somedir/obj/baz.o /somedir/obj/foo.o
/somedir/obj/bar.o /somedir/obj/baz.o
```

`$(warning)`不会改变`CWD`的值，但它会输出一条消息到`STDERR`。从输出中，你可以看到八次对 shell 的调用以及这些调用在 makefile 中是由哪几行引起的。

如果`CWD`是使用`:=`定义的，`$(warning)`技巧可以验证`CWD`仅被展开一次：

```
$ **make**
makefile:1: Call to shell
Make /somedir/obj/foo.o from /somedir/src/foo.c
Make /somedir/obj/bar.o from /somedir/src/bar.c
Make /somedir/obj/baz.o from /somedir/src/baz.c
/somedir/obj/foo.o /somedir/obj/bar.o /somedir/obj/baz.o /somedir/obj/foo.o
/somedir/obj/bar.o /somedir/obj/baz.o
```

一个快速的检查 makefile 是否使用了`=`和`$(shell)`这种耗时组合的方法是运行以下命令：

```
grep -n \$\(shell makefile | grep -v :=
```

这会打印出包含`$(shell)`且不包含`:=`的每一行的行号和详细信息。

# $(eval) 和变量缓存

在前面的章节中，你学习了如何使用 `:=` 来通过避免反复执行 `$(shell)` 来加速 makefile。不幸的是，重新修改 makefile 以使用 `:=` 可能会有问题，因为它们可能依赖于能够按照任何顺序定义变量。

在本节中，你将学习如何使用 GNU `make` 的 `$(eval)` 函数，在使用 `=` 扩展变量的递归优势的同时，获得类似于 `:=` 的速度提升。

## 关于 `$(eval)`

`$(eval)` 的参数会被扩展，然后像 makefile 中的部分内容一样解析。因此，在 `$(eval)` 中（它可能位于变量定义内），你可以编程式地定义变量、创建规则（显式或模式规则）、包含其他 makefile 等等。这是一个强大的函数。

这里是一个例子：

```
set = $(eval $1 := $2)

$(call set,FOO,BAR)
$(call set,A,B)
```

这导致 `FOO` 的值为 `BAR`，`A` 的值为 `B`。显然，这个例子可以在没有 `$(eval)` 的情况下实现，但很容易看出如何使用 `$(eval)` 对 makefile 中的定义进行编程式修改。

## 一个 `$(eval)` 副作用

`$(eval)` 的一种用途是创建副作用。例如，这里有一个变量，实际上是一个自动递增的计数器（它使用了 GMSL 的算术函数）：

```
include gmsl

c-value := 0
counter = $(c-value)$(eval c-value := $(call plus,$(c-value),1))
```

每次使用 `counter` 时，它的值都会增加 1。例如，以下 `$(info)` 函数序列会按顺序输出从 `0` 开始的数字：

```
$(info Starts at $(counter))
$(info Then it's $(counter))
$(info And then it's $(counter))
```

这里是输出结果：

```
$ **make**
Starts at 0
Then it's 1
And then it's 2
```

你可以使用像这样的简单副作用来找出 GNU `make` 重新评估某个变量的频率。你可能会对结果感到惊讶。例如，在构建 GNU `make` 时，它的 makefile 中的变量 `srcdir` 被访问了 48 次；`OBJEXT` 被访问了 189 次，而这只是一个非常小的项目。

GNU `make` 通过重复访问相同的字符串浪费时间来访问不变的变量。如果被访问的变量很长（例如长路径）或包含 `$(shell)` 调用或复杂的 GNU `make` 函数，那么变量处理的性能可能会影响 `make` 的整体运行时间。

如果你试图通过并行化 `make` 来最小化构建时间，或者开发人员正在运行只需要重新构建少数文件的增量构建，这一点尤其重要。在这两种情况下，GNU `make` 的长启动时间可能会非常低效。

## 缓存变量值

GNU `make` 确实提供了解决反复重新评估变量问题的方案：使用 `:=` 代替 `=`。使用 `:=` 定义的变量会将其值设置为一次性确定，右侧的表达式只会被评估一次，结果值被设置到变量中。使用 `:=` 可以使 makefile 解析更快，因为右侧只会被评估一次。但它确实引入了一些限制，因此很少使用。一个限制是它要求变量定义的顺序必须特定。例如，如果按如下顺序排列：

```
FOO := $(BAR)
BAR := bar
```

如果按照这种顺序排列，`FOO` 中的结果将与其它顺序下的值完全不同：

```
BAR := bar
FOO := $(BAR)
```

在第一个代码片段中，`FOO`是空的，而在第二个代码片段中，`FOO`是`bar`。

与以下内容的简洁性相比：

```
FOO = $(BAR)
BAR = bar
```

这里，`FOO` 的值是 `bar`。大多数 makefile 都是以这种方式编写的，只有非常用心（且注重速度）的 makefile 编写者才会使用 `:=`。

另一方面，几乎所有这些递归定义的变量在使用时只有一个值。复杂的递归定义变量的长时间求值对于 makefile 编写者来说是一种便利。

理想的解决方案是缓存变量值，以便保留`=`样式的灵活性，但变量只在首次计算时进行求值，从而提高速度。显然，这会导致灵活性略有丧失，因为变量不能取两个不同的值（这在 makefile 中有时是有用的）。但对于大多数用途来说，这会显著提升速度。

## 使用缓存的速度提升

请参见示例 makefile 在 示例 4-1 中。

示例 4-1。在这个 makefile 中，`FOO`和`C`被无意义地反复求值。

```
C := 1234567890 ABCDEFGHIJKLMNOPQRSTUVWXYZ
C += $C
C += $C
C += $C
C += $C
C += $C
C += $C
C += $C
C += $C
C += $C
C += $C
C += $C

FOO = $(subst 9,NINE,$C)$(subst 8,EIGHT,$C)$(subst 7,SEVEN,$C) \
$(subst 6,SIX,$C)$(subst 5,FIVE,$C)$(subst 4,FOUR,$C)          \
$(subst 3,THREE,$C)$(subst 2,TWO,$C)$(subst 1,ONE,$C)
_DUMMY := $(FOO)
--*snip*--

.PHONY: all
all:
```

它定义了一个变量 `C`，这是一个长字符串（实际上是 `1234567890` 重复 2,048 次，再加上字母表重复 2,048 次，最后加上空格，总共有 77,824 个字符）。在这里使用 `:=`，以便快速创建 `C`。`C` 旨在模拟在 makefile 中生成的长字符串（例如，带路径的源文件长列表）。

然后定义一个变量 `FOO`，使用内建的 `$(subst)` 函数来操作 `C`。`FOO` 模拟了 makefile 中的操作（例如，将文件名扩展名从 `.c` 改为 `.o`）。

最后，`$(FOO)`在小而实际的 makefile 中被求值 200 次，模拟了`FOO`的使用。这个 makefile 什么也不做；最后有一个虚拟的、空的 `all` 规则。

在我的笔记本上，使用 GNU `make` 3.81，这个 makefile 运行大约需要 3.1 秒。这是大量时间都花在反复操作 `C` 和 `FOO`，但并没有进行实际的构建。

使用来自 An $(eval) Side Effect Side Effect") 的 `counter` 技巧，你可以计算出在这个 makefile 中 `FOO` 和 `C` 被求值的次数。`FOO` 被求值了 200 次，而 `C` 被求值了 1600 次。令人惊讶的是，这些求值可以加起来非常快。

但 `C` 和 `FOO` 的值只需要计算一次，因为它们不会改变。假设你修改了 `FOO` 的定义，使用 `:=`：

```
FOO := $(subst 9,NINE,$C)$(subst 8,EIGHT,$C)$(subst 7,SEVEN,$C) \
$(subst 6,SIX,$C)$(subst 5,FIVE,$C)$(subst 4,FOUR,$C)           \
$(subst 3,THREE,$C)$(subst 2,TWO,$C)$(subst 1,ONE,$C)
```

这将运行时间降至 1.8 秒，`C` 被求值九次，而 `FOO` 只被求值一次。但当然，这需要使用 `:=`，并且会带来它的所有问题。

## 一个缓存函数

另一种缓存功能是这个简单的缓存方案：

```
cache = $(if $(cached-$1),,$(eval cached-$1 := 1)$(eval cache-$1 := $($1)))$(cache-$1)
```

首先，定义了一个名为`cache`的函数，它会在变量第一次被评估时自动缓存该变量的值，并在随后的每次尝试获取该值时从缓存中取出。

`cache`使用两个变量来存储变量的缓存值（在缓存变量`A`时，缓存值存储在`cache-A`中）以及该变量是否已被缓存（在缓存变量`A`时，*已缓存标志*是`cached-A`）。

首先，它检查变量是否已经缓存；如果缓存过，则`$(if)`什么也不做。如果没有缓存，则在第一次`$(eval)`中设置该变量的缓存标志，然后扩展变量的值（注意`$($1)`，它获取变量的名称并获取其值），并进行缓存。最后，`cache`返回缓存中的值。

要更新 makefile，只需将任何对变量的引用改为调用`cache`函数。例如，你可以通过简单的查找和替换，将示例 4-1 中的所有`$(FOO)`更改为`$(call cache,FOO)`。结果如示例 4-2 所示。

示例 4-2. 使用`cache`函数的示例 4-1 的修改版。

```
C := 1234567890 ABCDEFGHIJKLMNOPQRSTUVWXYZ
C += $C
C += $C
C += $C
C += $C
C += $C
C += $C
C += $C
C += $C
C += $C
C += $C
C += $C

FOO = $(subst 9,NINE,$C)$(subst 8,EIGHT,$C)$(subst 7,SEVEN,$C) \
$(subst 6,SIX,$C)$(subst 5,FIVE,$C)$(subst 4,FOUR,$C)          \
$(subst 3,THREE,$C)$(subst 2,TWO,$C)$(subst 1,ONE,$C)

_DUMMY := $(call cache,FOO)
--*snip*--

.PHONY: all
all:
```

在我的机器上运行此代码后，显示现在有一次访问`FOO`，仍然是九次访问`C`，并且运行时间为 2.4 秒。这不如`:=`版本（耗时 1.8 秒）快，但仍然快了 24%。在一个大的 makefile 中，这种技术可能会带来实际的差异。

## 总结

处理变量的最快方式是尽可能使用`:=`，但这需要小心和注意，最好只在新的 makefile 中进行（想象一下尝试回去重新设计一个已有的 makefile 来使用`:=`）。

如果你被`=`困住了，这里介绍的`cache`函数可以提供一个速度提升，尤其是对于进行增量短构建的开发者来说，这将是非常有用的。

如果只需要更改单个变量的定义，可以消除`cache`函数。例如，下面是将`FOO`的定义更改为神奇地从递归定义切换到简单定义的示例：

```
FOO = $(eval FOO := $(subst 9,NINE,$C)$(subst 8,EIGHT,$C)$(subst 7,SEVEN,$C) \
$(subst 6,SIX,$C)$(subst 5,FIVE,$C)$(subst 4,FOUR,$C)$(subst 3,THREE,$C)     \
$(subst 2,TWO,$C)$(subst 1,ONE,$C))$(value FOO)
```

第一次引用`$(FOO)`时，会触发`$(eval)`，将`FOO`从递归定义的变量变为简单定义（使用`:=`）。最后的`$(value FOO)`返回存储在`FOO`中的值，使得这个过程变得透明。

# 隐藏目标的问题

查看示例 4-3 中的 makefile：

示例 4-3。在这个 makefile 中，生成 `foo` 的规则也会生成 `foo.c`。

```
.PHONY: all
all: foo foo.o foo.c

foo:
→  touch $@ foo.c

%.o: %.c
→  touch $@
```

它包含了一个危险的陷阱，可能会导致 `make` 报告奇怪的错误，停止 `-n` 选项的正常工作，并阻止快速的并行 `make`。它甚至可能导致 GNU `make` 做错工作，并更新一个已经是最新的文件。

从表面看，这个 makefile 看起来很简单。如果你通过 GNU `make` 执行它，它会先构建 `foo`（生成文件 `foo` 和 `foo.c`），然后使用底部的模式从 `foo.c` 生成 `foo.o`。它最终会运行以下命令：

```
touch foo foo.c
touch foo.o
```

但其中有一个致命的缺陷。这个 makefile 中没有提到生成 `foo` 的规则实际上也生成了 `foo.c`。因此，`foo.c` 是一个*隐藏目标*，这是一个已经构建但 GNU `make` 不知道的文件，而隐藏目标会引发无数问题。

GNU `make` 在跟踪目标、需要构建的文件以及目标之间的依赖关系方面非常擅长。但 `make` 程序的表现好坏取决于其输入。如果你没有告诉 `make` 两个文件之间的关系，它不会自己发现这个关系，而且它会因为假设自己对文件及其关系拥有完美的了解而犯错误。

在这个例子中，`make` 之所以能工作，是因为它按从左到右的顺序构建 `all` 的先决条件。首先它遇到 `foo`，构建了它，并副作用地创建了 `foo.c`，然后再使用模式构建 `foo.o`。如果你改变 `all` 的先决条件的顺序，使得它不先构建 `foo`，构建就会失败。

隐藏目标至少有五个可怕的副作用。

## 如果隐藏目标缺失，会发生意外错误

假设 `foo` 存在，但 `foo.c` 和 `foo.o` 丢失：

```
$ **rm -f foo.c foo.o**
$ **touch foo**
$ **make**
No rule to make target `foo.c', needed by `foo.o'.
```

`make` 试图更新 `foo.o`，但因为它不知道如何生成 `foo.c`（因为它没有被列为任何规则的目标），调用 GNU `make` 会导致错误。

## -n 选项失效

GNU `make` 中有一个有用的 `-n` 调试选项，它会告诉 `make` 打印出它将要运行的命令，而不是实际运行它们：

```
$ **make -n** 
touch foo foo.c
No rule to make target `foo.c', needed by `foo.o'.
```

你已经看到，`make` 实际上会执行两个 `touch` 命令（`touch foo foo.c`，然后是 `touch foo.o`），但执行 `make -n`（没有 `foo*` 文件时）会导致错误。`make` 不知道生成 `foo` 的规则还会生成 `foo.c`，而且因为它没有实际运行 `touch` 命令，`foo.c` 就缺失了。因此，`-n` 不代表 `make` 实际会执行的命令，这使得它在调试时没有用处。

## 你无法并行化 make

GNU `make` 提供了一个方便的功能，允许它同时运行多个作业。如果构建中有多个编译任务，可以指定 `-j` 选项（后面跟一个数字，表示同时运行的作业数），以最大化 CPU 使用率并缩短构建时间。

不幸的是，一个隐藏的目标破坏了这个计划。以下是运行`make -j3`在我们的示例 makefile 中同时运行三个任务时的输出，参考自示例 4-3:

```
$ **make -j3**
touch foo foo.c
No rule to make target `foo.c', needed by `foo.o'.
Waiting for unfinished jobs....
```

GNU `make`尝试同时构建`foo`、`foo.o`和`foo.c`，并发现它不知道如何构建`foo.c`，因为它无法知道应该等待`foo`被构建。

## 如果隐藏目标被更新，make 会做错工作

假设`foo.c`文件在运行`make`时已经存在。因为`make`不知道`foo`的规则会影响到`foo.c`，它会被更新，即使它已经是最新的。在示例 4-2 中，`foo.c`被一个无害的`touch`操作修改，只有文件的时间戳被改变，但不同的操作可能会破坏或覆盖文件的内容：

```
$ **touch foo.c**
$ **rm -f foo foo.o**
$ **make**
touch foo foo.c
touch foo.o
```

`make`重建了`foo`，因为它缺失，并同时更新了`foo.c`，即使它显然是最新的。

## 你不能直接让 make 构建 foo.o

你希望输入`make foo.o`会导致 GNU `make`从`foo.c`构建`foo.o`，并在必要时构建`foo.c`。但是`make`不知道如何构建`foo.c`。当构建`foo`时，`foo.c`恰好被构建出来：

```
$ **rm -f foo.c**
$ **make foo.o**
No rule to make target `foo.c', needed by `foo.o'.
```

所以如果`foo.c`缺失，`make foo.o`会导致错误。

希望现在你已经相信隐藏目标是一个坏主意，并且可能会导致各种构建问题。

# GNU make 的转义规则

有时候你需要在 makefile 中插入特殊字符。也许你需要在`$(error)`消息中插入换行符、在`$(subst)`中插入空格字符，或者作为 GNU `make`函数的参数插入逗号。这三项简单的任务在 GNU `make`中可能会让人非常沮丧；本节将带你通过简单的语法，消除这些沮丧。

GNU `make`在包含命令的任何行开头使用制表符字符是一个著名的语言特性，但一些其他特殊字符也可能会让你困惑。GNU `make`处理`$`、`%`、`?`、`*`、`[`、`~`、`\`和`#`的方式都是特殊的。

## 处理`$`

每个 GNU `make`用户都熟悉`$`，它用于开始变量引用。你可以写`$(variable)`（带括号）或`${variable}`（带大括号）来获取`variable`的值，如果变量名是单个字符（如`a`），你可以省略括号，直接使用`$a`。

要获取字面量的`$`，你需要写`$$`。因此，要定义一个包含单个`$`符号的变量，你可以写：`dollar := $$`。

## 玩转`%`

转义`%`不像`$`那么简单，但只需要在三种情况中做转义，并且相同的规则适用于每种情况：在`vpath`指令中，在`$(patsubst)`中，以及在模式或静态模式规则中。

转义`%`的三个规则是：

+   你可以用一个单独的 `\` 字符来转义 `%`（也就是说，`\%` 就变成了字面量的 `%`）。

+   如果你需要在 `%` 前加一个字面量的 `\`（也就是说，你希望 `\` 不转义 `%`），则使用 `\` 进行转义（换句话说，`\\%` 变成了字面量的 `\` 后跟一个 `%` 字符，这个 `%` 将用于模式匹配）。

+   不用担心在模式的其他地方转义 `\`。它会被当作字面量处理。例如，`\hello` 就是 `\hello`。

## 通配符和路径

当符号 `?`、`*`、`[` 和 `]` 出现在文件名中时，它们会被特殊处理。一个包含以下内容的 makefile：

```
*.c:
→  @command
```

它实际上会搜索当前目录中的所有 `.c` 文件，并为每个文件定义一个规则。目标（以及 `include` 指令中提到的先决条件和文件）如果包含通配符字符，则会被 glob（文件系统被搜索，文件名与通配符字符匹配）。这些 glob 字符的意义与 Bourne shell 中相同。

`~` 字符在文件名中也有特殊处理，会被扩展为当前用户的主目录。

所有这些特殊的文件名字符都可以通过 `\` 来转义。例如：

```
\*.c:
→  @command
```

这个 makefile 为名为（字面上的）`*.c` 的文件定义了一个规则。

## 续行

除了转义功能外，你还可以在行尾使用 `\` 作为续行字符：

```
all:         \
prerequisite \
something else
→  @command
```

在这里，`all` 的规则有三个先决条件：`prerequisite`、`something` 和 `else`。

## 注释

你可以使用 `#` 字符来开始注释，也可以通过 `\` 转义将其变成字面量：

```
pound := \#
```

在这里，`$(pound)` 是一个单一字符：`#`。

## 我只想要一个换行符！

GNU `make` 尽最大努力将你与换行符隔离开。你不能转义换行符——没有特殊字符的语法（例如，你不能写 `\n`），即使是 `$(shell)` 函数也会从返回值中去掉换行符。

但是你可以使用 `define` 语法定义一个包含换行符的变量：

```
define newline

endef
```

请注意，这个定义包含了两行空白行，但使用 `$(newline)` 只会展开成一个换行符，这对于格式化错误消息非常有用：

```
$(error This is an error message$(newline)with two lines)
```

由于 GNU `make` 相当宽松的变量命名规则，可以定义一个名为 `\n` 的变量。所以，如果你喜欢保持熟悉的外观，可以这样做：

```
define \n

endef

$(error This is an error message $(\n)with two lines)
```

我们将在下一节中更详细地讨论特殊的变量名。

## 函数参数：空格和逗号

许多 GNU `make` 用户遇到的一个问题是处理 GNU `make` 函数参数中的空格和逗号。考虑以下 `$(subst)` 的用法：

```
spaces-to-commas = $(subst ,,,$1)
```

这需要三个由逗号分隔的参数：`from` 文本，`to` 文本，以及要更改的字符串。

它定义了一个名为 `spaces-to-commas` 的函数，用于将参数中的所有空格转换为逗号（这对于制作 CSV 文件可能很有用）。不幸的是，它由于两个原因无法正常工作：

+   `$(subst)`的第一个参数是一个空格。GNU `make`会去掉函数参数两端的所有空白字符。在这种情况下，第一个参数会被解释为空字符串。

+   第二个参数是一个逗号。GNU `make`无法区分用作参数分隔符的逗号和作为参数的逗号。此外，没有办法转义逗号。

如果你知道 GNU `make`在展开参数之前会进行空白字符的剥离和参数分隔，那么你可以绕过这两个问题。所以，如果我们能定义一个包含空格的变量和一个包含逗号的变量，我们可以写出如下的代码来达到预期效果：

```
spaces-to-commas = $(subst $(space),$(comma),$1)
```

定义一个包含逗号的变量很简单，如下所示：

```
comma := ,
```

但是空格有点复杂。你可以通过几种方式定义一个空格。一个方法是利用每次向变量添加内容（使用`+=`）时，都会在添加的文本前插入一个空格：

```
space :=
space +=
```

另一种方法是先定义一个不包含任何内容的变量，然后用它来围绕空格，以防空格被 GNU `make`剥离：

```
blank :=
space := $(blank) $(blank)
```

你也可以使用这个技巧将一个字面上的制表符字符放入变量中：

```
blank :=
tab := $(blank)→$(blank)
```

就像上一节中定义了`$(\n)`一样，定义特别命名的空格和逗号变量也是可能的。GNU `make`的规则足够宽松，允许我们这么做：

```
, := ,

blank :=
space := $(blank) $(blank)
$(space) := $(space)
```

第一行定义了一个名为`,`的变量（可以用`$(,)`甚至`$,`），其内容是一个逗号。

最后三行定义了一个名为`space`的变量，其内容是一个空格字符，然后用它来定义一个名为（没错，它的名字就是一个空格字符）的变量，该变量包含一个空格。

使用这个定义，你可以写`$( )`甚至`$`（在那个`$`后面有一个空格）来获得一个空格字符。请注意，这样做可能会在未来的`make`更新中引发问题，因此像这样玩弄技巧可能是危险的。如果你不喜欢冒险，最好使用名为`space`的变量，避免使用`$( )`。因为空白字符在 GNU `make`中是特殊的，通过像`$( )`这样的技巧将`make`的解析器推向极限可能会导致破坏。

使用这些定义，可以将`spaces-to-commas`函数写成：

```
spaces-to-commas = $(subst $( ),$(,),$1)
```

这个看起来很奇怪的定义通过`subst`将空格替换为逗号。它之所以有效，是因为`$( )`会被`subst`展开，并且本身就是一个空格。这个空格会成为第一个参数（即将被替换的字符串）。第二个参数是`$(,)`，当它被展开时，会变成一个逗号。结果是，`spaces-to-commas`将空格转化为逗号，而不会让 GNU `make`混淆空格和逗号字符。

## 《暮光之区》

可以像定义`$( )`和`$(\n)`这样的变量定义一样，进一步发展，定义像`=`、`#`或`:`这样的变量名。以下是一些其他有趣的变量定义：

```
# Define the $= or $(=) variable which has the value =
equals := =
$(equals) := =
# Define the $# or $(#) variable which has the value #
hash := \#
$(hash) := \#
# Define the $: or $(:) variable which has the value :
colon := :
$(colon) := :

# Define the $($$) variable which has the value $
dollar := $$
$(dollar) := $$
```

这些定义可能没有太大用处，但如果你想将 GNU `make`的语法推向极限，可以尝试以下方法：

```
+:=+
```

是的，这定义了一个名为 `+` 的变量，内容是一个 `+`。

# `$(wildcard)` 的问题

`$(wildcard)` 函数是 GNU `make` 的模式匹配函数。它是获取 makefile 中文件列表的一个有用方法，但它可能会表现得出乎意料。它并不总是提供与运行 `ls` 相同的结果。继续阅读，了解为什么会这样以及该怎么做。

## `$(wildcard)` 解释

你可以在 makefile 或规则中任何地方使用 `$(wildcard)` 来获取与一个或多个 *glob* 风格模式匹配的文件列表。例如，`$(wildcard *.foo)` 返回一个以 `.foo` 结尾的文件列表。回想一下，列表是一个字符串，其中元素之间用空格分隔，因此 `$(wildcard *.foo)` 可能返回 `a.foo b.foo c.foo`。（如果文件名中包含空格，返回的列表可能会看起来不正确，因为无法区分列表分隔符（空格）和文件名中的空格。）

你还可以传递一个模式列表给 `$(wildcard)`，因此 `$(wildcard *.foo *.bar)` 会返回所有以 `.foo` 或 `.bar` 结尾的文件。`$(wildcard)` 函数支持以下模式匹配操作符：`*`（匹配 0 或更多字符）、`?`（匹配 1 个字符）和 `[...]`（匹配字符，`[123]`，或字符范围，`[a-z]`）。

`$(wildcard)` 的另一个有用功能是，如果传给它的文件名不包含模式，它只是检查文件是否存在。如果文件存在，它返回文件名；否则，`$(wildcard)` 返回一个空字符串。因此，`$(wildcard)` 可以与 `$(if)` 结合使用，创建一个 `if-exists` 函数：

```
if-exists = $(if ($wildcard $1),$2,$3)
```

`if-exists` 有三个参数：要检查的文件名、文件存在时要执行的操作，以及文件不存在时要执行的操作。以下是其使用的一个简单示例：

```
$(info a.foo is $(call if-exists,a.foo,there,not there))
```

如果 `a.foo` 存在，它将打印 `a.foo is there`；如果不存在，它将打印 `a.foo is not there`。

## 意外的结果

以下每个示例使用两个变量来获取特定目录中以 `.foo` 结尾的文件列表：`WILDCARD_LIST` 和 `LS_LIST` 分别通过调用 `$(wildcard)` 和 `$(shell ls)` 来返回以 `.foo` 结尾的文件列表。变量 `DIRECTORY` 存储示例查找文件的目录；对于当前目录，`DIRECTORY` 保持为空。

起始的 makefile 如下所示：

```
WILDCARD_LIST = wildcard returned \'$(wildcard $(DIRECTORY)*.foo)\'
LS_LIST = ls returned \'$(shell ls $(DIRECTORY)*.foo)\'

.PHONY: all
all:
→  @echo $(WILDCARD_LIST)
→  @echo $(LS_LIST)
```

在当前目录中只有一个文件 `a.foo` 时，运行 GNU `make` 结果如下：

```
$ **touch a.foo**
$ **make**
wildcard returned 'a.foo'
ls returned 'a.foo'
```

现在扩展 makefile，使其通过 `touch` 创建一个名为 `b.foo` 的文件。这个 makefile 应该如下所示：示例 4-4 返回不同的结果。"):

示例 4-4。当你运行这个 makefile 时，`ls` 和 `$(wildcard)` 返回不同的结果。

```
WILDCARD_LIST = wildcard returned \'$(wildcard $(DIRECTORY)*.foo)\'
LS_LIST = ls returned \'$(shell ls $(DIRECTORY)*.foo)\'

.PHONY: all
all: b.foo
→ @echo $(WILDCARD_LIST)
→ @echo $(LS_LIST)

b.foo:
→ @touch $@
```

通过 GNU `make` 运行这个 makefile（仅有已存在的 `a.foo` 文件）会产生以下令人惊讶的输出：

```
$ **touch a.foo**
$ **make**
wildcard returned 'a.foo'
ls returned 'a.foo b.foo'
```

`ls` 返回正确的列表（因为`b.foo`在`all`规则执行时已经被创建），但`$(wildcard)`没有；`$(wildcard)`似乎显示的是`b.foo`创建之前的状态。

在子目录中使用`.foo`文件（而不是当前工作目录中的文件）会导致不同的输出，如示例 4-5 返回相同的结果。")所示。

示例 4-5. 这次，`ls` 和 `$(wildcard)` 返回相同的结果。

```
DIRECTORY=subdir/

.PHONY: all
all: $(DIRECTORY)b.foo
→  @echo $(WILDCARD_LIST)
→  @echo $(LS_LIST)

$(DIRECTORY)b.foo:
→  @touch $@
```

这里，Makefile 已更新，以便使用`DIRECTORY`变量来指定子目录`subdir`。有一个预先存在的文件`subdir/a.foo`，Makefile 将会创建`subdir/b.foo`。

运行这个 Makefile 会得到：

```
$ **touch subdir/a.foo**
$ **make**
wildcard returned 'subdir/a.foo subdir/b.foo'
ls returned 'subdir/a.foo subdir/b.foo'
```

在这里，`$(wildcard)`和`ls`都返回相同的结果，并且都显示了两个`.foo`文件的存在：`subdir/a.foo`，它在运行`make`之前就已经存在，以及`subdir/b.foo`，它是由 Makefile 创建的。

在我解释发生了什么之前，让我们看看最后一个 Makefile (示例 4-6 返回不同的结果。"))：

示例 4-6. 一个小的变化使得`ls`和`$(wildcard)`返回不同的结果。

```
DIRECTORY=subdir/

$(warning Preexisting file: $(WILDCARD_LIST))

.PHONY: all
all: $(DIRECTORY)b.foo
→  @echo $(WILDCARD_LIST)
→  @echo $(LS_LIST)
$(DIRECTORY)b.foo:
→  @touch $@
```

在这个 Makefile 中，使用了`$(warning)`来打印出子目录中已经存在的`.foo`文件列表。

以下是输出：

```
$ **touch subdir/a.foo**
$ **make**
makefile:6: Preexisting file: wildcard returned 'subdir/a.foo'
wildcard returned 'subdir/a.foo'
ls returned 'subdir/a.foo subdir/b.foo'
```

请注意，现在 GNU `make` 的行为看起来像是示例 4-4 返回不同的结果。")中的行为；即使`subdir/b.foo`文件已由 Makefile 创建，`$(wildcard)`仍然看不到它并未显示，尽管它已经被创建并且`ls`找到了它。

## 意外结果解释

我们得到意外且显然不一致的结果，因为 GNU `make` 包含它自己的目录条目缓存。`$(wildcard)`是从这个缓存中读取（而不是像`ls`那样直接从磁盘读取）来获取结果。了解何时填充缓存对于理解`$(wildcard)`返回的结果至关重要。

GNU `make` 只有在被迫时才会填充缓存（例如，当它需要读取目录条目以满足`$(wildcard)`或其他模式匹配请求时）。如果你知道 GNU `make` 只有在需要时才会填充缓存，那么就可以解释结果。

在示例 4-4 返回不同的结果。")中，GNU `make`在开始时会填充当前工作目录的缓存。因此，文件`b.foo`不会出现在`$(wildcard)`的输出中，因为它在缓存填充时并不存在。

在示例 4-5` 返回相同的结果。")中，GNU `make` 直到需要时才会填充来自 `subdir` 的缓存条目。这些条目首次被 `$(wildcard)` 需要，而 `$(wildcard)` 是在 `subdir/b.foo` 创建后执行的，因此 `subdir/b.foo` 会出现在 `$(wildcard)` 输出中。

在示例 4-6` 返回不同的结果。")中，`$(warning)` 在 Makefile 开始时触发并填充了缓存（因为它执行了 `$(wildcard)`），因此 `subdir/b.foo` 在那次 `make` 过程中没有出现在 `$(wildcard)` 的输出中。

预测缓存何时被填充非常困难。`$(wildcard)` 会填充缓存，但规则的目标或先决条件列表中使用像 `*` 这样的通配符操作符也会填充缓存。示例 4-7` 缓存可能很难理解。")是一个 Makefile，它构建了两个文件（`subdir/b.foo` 和 `subdir/c.foo`），并执行了几个 `$(wildcard)` 操作：

示例 4-7。 当 GNU `make` 填充缓存时，`$(wildcard)` 缓存可能很难理解。

```
DIRECTORY=subdir/

.PHONY: all
all: $(DIRECTORY)b.foo
→  @echo $(WILDCARD_LIST)
→  @echo $(LS_LIST)
$(DIRECTORY)b.foo: $(DIRECTORY)c.foo
→  @touch $@
→  @echo $(WILDCARD_LIST)
→  @echo $(LS_LIST)

$(DIRECTORY)c.foo:
→  @touch $@
```

输出可能会让你感到惊讶：

```
   $ **make**
   wildcard returned 'subdir/a.foo subdir/c.foo'
   ls returned 'subdir/a.foo subdir/c.foo'
➊ wildcard returned 'subdir/a.foo subdir/c.foo'
   ls returned 'subdir/a.foo subdir/b.foo subdir/c.foo'
```

即使第一个 `$(wildcard)` 已经在生成 `subdir/b.foo` 的规则中执行，并且在创建了 `subdir/b.foo` 后执行了 `touch`，但在 `$(wildcard)` 的输出中并没有提到 `subdir/b.foo` ➊。`ls` 的输出中也没有提到 `subdir/b.foo`。

原因在于，整个命令块在规则中的任何一行执行之前就已经扩展成其最终形式。因此，`$(wildcard)` 和 `$(shell ls)` 会在 `touch` 执行之前完成。

如果在使用 `-j` 开关并行执行 `make`，`$(wildcard)` 的输出会变得更加不可预测。在这种情况下，规则执行的确切顺序无法预测，因此 `$(wildcard)` 的输出可能变得更加不可预测。

我建议你：不要在规则中使用 `$(wildcard)`；只在解析时（在任何规则开始执行之前）在 Makefile 中使用 `$(wildcard)`。如果你将 `$(wildcard)` 的使用限制在解析时，你可以确保结果一致：`$(wildcard)` 将显示在 GNU `make` 执行之前的文件系统状态。

# 创建目录

现实世界中的 Makefile 黑客常遇到的一个问题是，在构建之前，或者至少在使用这些目录的命令运行之前，需要构建目录层次结构。最常见的情况是，Makefile 黑客希望确保将创建目标文件的目录已存在，并且他们希望这个过程自动化。本节将探讨在 GNU `make` 中实现目录创建的多种方法，并指出一个常见的陷阱。

## 一个示例 Makefile

以下 makefile 使用 GNU `make` 内建变量 `COMPILE.C` 从 `foo.c` 构建目标文件 `/out/foo.o`，通过运行编译器将 `.c` 文件转换为 `.o` 文件。

`foo.c` 和 makefile 在同一目录下，但 `foo.o` 会被放到 `/out/` 目录中：

```
.PHONY: all
all: /out/foo.o

/out/foo.o: foo.c
→  @$(COMPILE.C) -o $@ $<
```

这个示例在 `/out/` 存在的情况下工作良好。但如果它不存在，你会收到类似下面的编译错误：

```
$ **make**
Assembler messages:
FATAL: can't create /out/foo.o: No such file or directory
make: *** [/out/foo.o] Error 1
```

显然，你希望的是 makefile 在 `/out/` 不存在时能自动创建它。

## 不应该做的事

因为 GNU `make` 擅长创建不存在的东西，所以看起来很明显，应该将 `/out/` 作为 `/out/foo.o` 的前提条件，并为创建目录编写一个规则。这样，当我们需要构建 `/out/foo.o` 时，目录就会被创建。

示例 4-8 展示了修改后的 makefile，其中目录作为前提条件，并使用 `mkdir` 创建目录的规则。

示例 4-8. 这个 makefile 最终可能会做不必要的工作。

```
OUT = /out

.PHONY: all
all: $(OUT)/foo.o

$(OUT)/foo.o: foo.c $(OUT)/
→  @$(COMPILE.C) -o $@ $<

$(OUT)/:
→  mkdir -p $@
```

为了简化，输出目录的名称存储在一个名为 `OUT` 的变量中，并且 `mkdir` 命令使用 `-p` 选项，这样它就会一次性构建所有必要的父目录。在这个例子中，路径很简单：就是 `/out/`，但 `-p` 选项意味着 `mkdir` 可以一次性创建一条长路径的所有目录。

对于这个基础示例来说，这个方法工作良好，但存在一个重大问题。因为目录的时间戳通常在目录更新时（例如，文件被创建、删除或重命名时）会更新，所以这个 makefile 可能会做过多的工作。

例如，仅仅在 `/out/` 目录下创建另一个文件就会强制重新构建 `/out/foo.o`。在更复杂的示例中，这可能意味着许多目标文件会因为其他文件在同一目录下被重建而无故重建。

## 解决方案 1：在解析 makefile 时创建目录

在示例 4-8 中，解决问题的一个简单方法是，在解析 makefile 时直接创建目录。通过快速调用 `$(shell)` 可以实现：

```
OUT = /out

.PHONY: all
all: $(OUT)/foo.o

$(OUT)/foo.o: foo.c
→  @$(COMPILE.C) -o $@ $<

$(shell mkdir -p $(OUT))
```

在创建任何目标或运行任何命令之前，makefile 会被读取和解析。如果你在 makefile 中的某个位置放置 `$(shell mkdir -p $(OUT))`，GNU `make` 每次加载 makefile 时都会运行 `mkdir`。

一个可能的缺点是，如果需要创建多个目录，这个过程可能会比较慢。而且 GNU `make` 会做不必要的工作，因为每次运行 `make` 时，它都会尝试构建这些目录。某些用户也不喜欢这种方法，因为即使某些目录在 makefile 中的规则并未使用，所有目录还是会被创建。

通过首先测试目录是否存在，可以进行一些小的改进：

```
ifeq ($(wildcard $(OUT)/.),)
$(shell mkdir -p $(OUT))
endif
```

在这里，`$(wildcard)`与`/.`一起使用，以检查目录是否存在。如果目录缺失，`$(wildcard)`将返回一个空字符串，`$(shell)`将会被执行。

## 解决方案 2：仅在构建 all 时创建目录

一个相关的解决方案是仅在构建`all`时才创建目录。这意味着在每次解析 makefile 时，目录不会被创建（这可以避免在你输入`make clean`或`make depend`时进行不必要的工作）：

```
OUT = /out

.PHONY: all
all: make_directories $(OUT)/foo.o

$(OUT)/foo.o: foo.c
→  @$(COMPILE.C) -o $@ $<

.PHONY: make_directories
make_directories: $(OUT)/

$(OUT)/:
→  mkdir -p $@
```

这个解决方案有些杂乱，因为你必须将`make_directories`指定为任何目标的前提条件，该目标可能是在`make`后由用户指定的。如果不这样做，可能会遇到目录未创建的情况。你应该避免使用这种技术，特别是因为它会完全破坏并行构建。

## 解决方案 3：使用目录标记文件

如果你回头看看示例 4-8，你会注意到一个相当不错的特性：它只为特定目标构建所需的目录。在一个更复杂的例子中（有许多这样的目录需要构建），能够使用类似的解决方案会很好，同时避免目录时间戳变化导致的不断重建问题。

为此，你可以在目录中存储一个特殊的空文件，我称之为*标记*文件，并将其作为前提条件使用。因为它是一个普通文件，普通的 GNU `make`重建规则适用，并且其时间戳不会受到目录变化的影响。

如果你添加一个规则来构建标记文件（并确保其目录存在），你可以通过指定标记文件作为目录的代理，来指定目录作为前提条件。

```
OUT = /out
.PHONY: all
all: $(OUT)/foo.o

$(OUT)/foo.o: foo.c $(OUT)/.f
→  @$(COMPILE.C) -o $@ $<
$(OUT)/.f:
→  mkdir -p $(dir $@)
→  touch $@
```

注意，构建`$(OUT)/.f`的规则会在必要时创建目录，并触及`.f`文件。因为目标是一个文件（`.f`），它可以安全地作为`$(OUT)/foo.o`规则的前提条件。

`$(OUT)/.f`规则使用 GNU `make`函数`$(dir FILE)`来提取目标的目录部分（即`.f`文件的路径），并将该目录传递给`mkdir`。

唯一的缺点是，对于每个可能需要创建的目录中的目标构建规则，都必须指定`.f`文件。

为了简化使用，你可以创建函数，自动生成创建目录的规则，并计算`.f`文件的正确名称：

```
marker = $1.f
make_dir = $(eval $1.f: ; @mkdir -p $$(dir $$@) ; touch $$@)

OUT = /out
.PHONY: all
all: $(OUT)/foo.o

$(OUT)/foo.o: foo.c $(call marker,$(OUT))
→  @$(COMPILE.C) -o $@ $<

$(call make-dir,$(OUT))
```

在这里，`marker`和`make-dir`用于简化 makefile。

## 解决方案 4：使用仅顺序前提条件来创建目录

在 GNU `make` 3.80 及以后版本中，另一种解决方案是使用*仅顺序*前提条件。仅顺序前提条件在目标之前正常构建，但当前提条件发生变化时不会导致目标重新构建。通常情况下，当前提条件被重新构建时，目标也会被重新构建，因为 GNU `make` 假设目标依赖于前提条件。而仅顺序前提条件则不同：它们在目标之前被构建，但目标不会因为仅顺序前提条件的构建而更新。

这正是我们希望在示例 4-8 中的原始破损示例中实现的——确保目录按需重建，但不会在每次目录的时间戳更改时重新构建 `.o` 文件。

仅顺序的前提条件是指那些出现在竖线符号 `|` 后面的前提条件，且必须放在任何正常前提条件之后。

事实上，仅仅在示例 4-8 中的破损示例中添加这一字符，就能使其正确工作：

```
   OUT = /out

   .PHONY: all
   all: $(OUT)/foo.o
   $(OUT)/foo.o: foo.c | $(OUT)/
   →  @$(COMPILE.C) -o $@ $<

➊ $(OUT)/:
   →  mkdir -p $@
```

如果目录缺失，`$(OUT)/` ➊ 的规则将会被执行，但对目录的更改不会导致 `$(OUT)/foo.o` 被重新构建。

## 解决方案 5：使用模式规则、第二次展开和标记文件

在典型的 makefile 中（不是像书中这样的简单示例），目标通常通过模式规则构建，如下所示：

```
OUT = /out
.PHONY: all
all: $(OUT)/foo.o

$(OUT)/%.o: %.c
→  @$(COMPILE.C) -o $@ $<
```

但我们可以改变这个模式规则，通过使用标记文件自动构建目录。

在 GNU `make` 3.81 及以后版本中，有一个令人兴奋的功能叫做*第二次展开*（通过在 makefile 中指定 `.SECONDEXPANSION` 目标来启用）。通过第二次展开，任何规则的前提条件列表在规则被使用之前会进行第二次展开（第一次展开发生在读取 makefile 时）。通过用第二个 `$` 转义任何 `$` 符号，可以在前提条件列表中使用 GNU `make` 的自动变量（如 `$@`）。

使用每个目录的标记文件和第二次展开，你可以创建一个 makefile，通过在任何规则的前提条件列表中简单地添加一项内容，自动仅在必要时创建目录：

```
OUT = /tmp/out

.SECONDEXPANSION:

all: $(OUT)/foo.o

$(OUT)/%.o: %.c $$(@D)/.f
→  @$(COMPILE.C) -o $@ $<

%/.f:
→  mkdir -p $(dir $@)
→  touch $@

.PRECIOUS: %/.f
```

用于生成 `.o` 文件的模式规则有一个特殊的前提条件 `$$(@D)/.f`，它利用第二次展开功能来获取目标要构建的目录。它通过对 `$@` 应用 `D` 修饰符来实现这一点，`$@` 获取目标的目录（而 `$@` 本身获取目标的名称）。

该目录将在构建 `.f` 文件的过程中通过`%/.f`模式规则生成。请注意，`.f` 文件被标记为*珍贵*，以便 GNU `make` 不会删除它们。如果没有这一行，`.f` 文件会被视为无用的中间文件，并且在退出时会被 GNU `make` 清理掉。

## 解决方案 6：在规则中直接创建目录

也可以在需要目录的规则中创建目录；这称为“在规则中创建目录”。例如：

```
OUT = /out

.PHONY: all
all: $(OUT)/foo.o

$(OUT)/foo.o: foo.c
→  mkdir -p $(@D)
→  @$(COMPILE.C) -o $@ $<
```

在这里，我修改了`$(OUT)/foo.o`规则，使得每次都使用`-p`创建目录。只有当少数规则需要创建目录时，这种方式才有效。更新每条规则以添加`mkdir`是非常繁琐的，并且很容易遗漏某些规则。

# GNU make 遇到带空格的文件名

GNU `make`将空格字符视为列表分隔符；任何包含空格的字符串都可以视为由空格分隔的单词列表。这是 GNU `make`的基本原理，空格分隔的列表随处可见。不幸的是，当文件名包含空格时，这会带来问题。本节将探讨如何解决“文件名中的空格问题”。

## 一个示例 Makefile

假设你需要创建一个 makefile，处理两个名为`foo bar`和`bar baz`的文件，其中`foo bar`是从`bar baz`构建的。处理包含空格的文件名可能会很棘手。

在 makefile 中天真地编写的方式是这样的：

```
foo bar: bar baz
→ @echo Making $@ from $<
```

但这不起作用。GNU `make`无法区分文件名中空格是其一部分，还是仅仅是分隔符。实际上，天真地编写的 makefile 与以下内容完全相同：

```
foo: bar baz
→ @echo Making $@ from $<
bar: bar baz
→ @echo Making $@ from $<
```

将文件名用引号括起来也不起作用。如果你尝试这样做：

```
"foo bar": "bar baz"
→ @echo Making $@ from $<
```

GNU `make`认为你在谈论四个文件，分别是`"foo`、`bar"`、`"bar`和`baz"`。GNU `make`忽略了双引号，并像往常一样按空格拆分列表。

## 使用`\`转义空格

解决空格问题的一种方法是使用 GNU `make`的转义运算符`\`，你可以用它来转义敏感字符（如字面意义上的`#`，以防它开始注释，或者字面意义上的`%`，以防它被用作通配符）。

因此，对于包含空格的文件名的规则，使用`\`来转义空格。我们的示例 makefile 可以重写如下：

```
foo\ bar: bar\ baz
→ @echo Making $@ from $<
```

它将正确工作。`\`字符在解析 makefile 时被移除，因此实际的目标和前提条件名称正确地包含空格。这将在自动变量（如`$@`）中反映出来。

当`foo bar`需要更新时，简单的 makefile 会输出：

```
$ **make**
Making foo bar from bar baz
```

你还可以在 GNU `make`的`$(wildcard)`函数中使用相同的转义机制。要检查`foo bar`是否存在，可以使用`$(wildcard foo\ bar)`，GNU `make`将把`foo bar`作为一个单独的文件名，在文件系统中查找。

不幸的是，GNU `make`的其他处理空格分隔列表的函数并不尊重空格的转义。例如，`$(sort foo\ bar)`的输出是列表`bar foo\`，而不是你可能期待的`foo\ bar`。实际上，`$(wildcard)`是唯一一个尊重`\`字符来转义空格的 GNU `make`函数。

如果你必须处理包含目标列表的自动变量时，这会引发问题。考虑这个稍微复杂一些的例子：

```
foo\ bar: bar\ baz a\ b
→ @echo Making $@ from $<
```

现在 `foo bar` 有两个前提条件 `bar baz` 和 `a b`。在这种情况下，`$^`（所有前提条件的列表）的值是什么？它是 `bar baz a b`：转义符已经去掉，甚至如果没有去掉，只有 `$(wildcard)` 会处理 `\`，这意味着它将是无用的。从 GNU `make` 的角度来看，`$^` 是一个包含四个元素的列表。

查看自动变量的定义可以告诉我们哪些在文件名中存在空格时是安全使用的。表 4-1 显示了每个自动变量及其是否安全。

表 4-1. 自动变量的安全性

| 自动变量 | 它安全吗？ |
| --- | --- |
| `$@` | 是 |
| `$<` | 是 |
| `$%` | 是 |
| `$*` | 是 |
| `$?` | 否 |
| `$^` | 否 |
| `$+` | 否 |

那些本身就是列表的变量（`$?`、`$^` 和 `$+`）是不安全的，因为 GNU `make` 的列表是由空格分隔的；其他的则是安全的。

情况变得更糟了。即使表中的前四个自动变量是安全使用的，它们的修改版本（带有 `D` 和 `F` 后缀，用于提取相应自动变量的目录和文件名部分）也不安全。这是因为它们是通过 `dir` 和 `notdir` 函数定义的。

考虑这个示例 makefile：

```
/tmp/foo\ bar/baz: bar\ baz a\ b
→  @echo Making $@ from $<
```

`$@` 的值是 `/tmp/foo bar/baz`，如预期，但 `$(@D)` 的值是 `/tmp bar`（而不是 `/tmp/foo bar`），而 `$(@F)` 的值是 `foo baz`（而不是仅 `baz`）。

## 将空格转换为问号

另一种解决空格问题的方法是将空格转换为问号。这里是转换后的原始 makefile：

```
foo?bar: bar?baz
→  @echo Making $@ from $<
```

因为 GNU `make` 会对目标和前提条件的名称进行通配符匹配（并且会尊重其中的空格），所以这将会起作用。但结果是不可预测的。

如果 `foo bar` 存在，当这个 makefile 执行时，模式 `foo?bar` 将被转换为 `foo bar`，并且该值将被用于 `$@`。如果该文件在解析 makefile 时不存在，那么模式（因此 `$@`）将保持为 `foo?bar`。

另一个问题也存在：`?` 可能匹配到除了空格以外的其他字符。例如，如果系统上有一个名为 `foombar` 的文件，makefile 可能会错误地处理错误的文件。

为了绕过这个问题，Robert Mecklenburg 在 *Managing Projects with GNU Make, 3rd edition*（O'Reilly, 2004）中定义了两个函数来自动添加和删除空格。`sq` 函数将每个空格转化为问号（`sq` 表示空格变问号）；`qs` 函数做相反的操作（它将每个问号转换为空格）。以下是更新后的 makefile，使用了两个函数（`sq` 和 `qs`）来添加和删除问号。除非某个文件名中包含问号，否则该方法有效，但需要将所有文件名的使用都包装在 `sq` 和 `qs` 的调用中。

```
sp :=
sp +=
qs = $(subst ?,$(sp),$1)
sq = $(subst $(sp),?,$1)

$(call sq,foo bar): $(call sq,bar baz)
→ @echo Making $(call qs,$@) from $(call qs,$<)
```

无论哪种方式，由于我们仍然不能确定自动变量中是否会包含问号，因此使用基于列表的自动变量或任何 GNU `make`列表函数仍然是不可能的。

## 我的建议

由于 GNU `make`在处理文件名中的空格时存在困难，应该怎么办呢？以下是我的建议：

**如果可能，重命名文件以避免空格。**

然而，这对许多人来说是不可能的，因为文件名中的空格可能是由第三方添加的。

**使用 8.3 文件名。**

如果你在使用 Windows，可能可以使用短的 8.3 文件名，这样你仍然可以在磁盘上使用空格，但在 makefile 中避免使用它们。

**使用`\`进行转义。**

如果你需要空格，可以使用`\`进行转义，这会得到一致的结果。只要确保避免使用在表 4-1 中列为不安全的自动变量。

如果你使用`\`进行转义，并且需要处理包含空格的文件名列表，最好的做法是将空格替换为其他字符，然后再将其恢复。

例如，下面代码中的`s+`和`+s`函数将转义空格替换为`+`符号，再将其恢复。然后，你可以安全地使用所有 GNU `make`函数来处理文件名列表。只要确保在规则中使用这些名称之前，移除`+`符号即可。

```
space :=
space +=

s+ = $(subst \$(space),+,$1)
+s = $(subst +,\$(space),$1)
```

下面是一个示例，演示如何使用它们将带有转义空格的源文件列表转换为目标文件列表，然后将这些目标文件用作定义`all`规则的前提条件：

```
SRCS := a\ b.c c\ d.c e\ f.c
SRCS := $(call s+,$(SRCS))

OBJS := $(SRCS:.c=.o)

all: $(call +s,$(OBJS))
```

源文件存储在`SRCS`中，其中的文件名空格已被转义。因此，`SRCS`包含三个文件，分别是`a b.c`、`c d.c`和`e f.c`。GNU `make`使用`\`转义来保留每个文件名中的转义空格。将`SRCS`转换为`OBJS`中的目标文件列表时，通常使用`.c=.o`来替换每个`.c`扩展名为`.o`，但首先通过`s+`函数修改`SRCS`，将转义的空格变为`+`符号。结果，GNU `make`将看到`SRCS`作为包含三个元素的列表，分别是`a+b.c`、`c+d.c`和`e+f.c`，并且扩展名更改将正确执行。当稍后在 makefile 中使用`OBJS`时，`+`符号将通过调用`+s`函数被还原为转义空格。

# 路径处理

Makefile 的创建者通常需要操作文件系统路径，但 GNU `make`提供的路径操作函数很少。而跨平台的`make`由于路径语法差异而变得困难。本节将介绍如何在 GNU `make`中操作路径，并在跨平台的复杂环境中导航。

## 目标名称匹配

看下面这个示例 makefile，假设`../foo`文件丢失了。这个 makefile 能成功创建它吗？

```
.PHONY: all
all: ../foo

.././foo:
→ touch $@
```

如果你使用 GNU `make`运行这个 makefile，你可能会惊讶地看到以下错误：

```
$ **make**
make: *** No rule to make target `../foo', needed by `all'. Stop.
```

如果你将 makefile 改成这样：

```
.PHONY: all
all: ../foo

./../foo:
→ touch $@
```

你会发现它按预期工作，并执行`touch ../foo`。

第一个 makefile 会失败，因为 GNU `make` 不对目标名称进行路径处理，所以它会将 `../foo` 和 `.././foo` 视为两个不同的目标，导致无法将它们关联起来。第二个 makefile 则工作正常，因为我在前述句子中说谎了。实际上，GNU `make` 确实会做一点路径处理：它会去掉目标名称前面的 `./`。因此，在第二个 makefile 中，两个目标都是 `../foo`，并按预期工作。

GNU `make` 目标的普遍规则是，它们被视为字面字符串，不会以任何方式进行解释。因此，当你在 makefile 中引用目标时，确保使用相同的字符串是非常重要的。

## 处理路径列表

需要再次强调的是，GNU `make` 列表仅仅是字符串，其中任何空格都被视为列表分隔符。因此，不推荐路径中有空格，因为这会导致无法使用许多 GNU `make` 的内建函数，且路径中的空格会给目标带来问题。

例如，假设目标是`/tmp/sub directory/target`，我们可以为它写出如下规则：

```
/tmp/sub directory/target:
→ @do stuff
```

GNU `make` 实际上会将其解释为两个规则，一个针对 `/tmp/sub`，另一个针对 `directory/target`，就像你写的是这样：

```
/tmp/sub:
→ @do stuff
directory/target:
→ @do stuff
```

你可以通过使用 `\` 来转义空格来解决这个问题，但 GNU `make` 对这个转义的支持不太好（它仅在目标名称和 `$(wildcard`) 函数中有效）。

除非必须使用空格，否则避免在目标名称中使用空格。

## `VPATH` 和 `vpath` 中的路径列表

另一个在 GNU `make` 中出现路径列表的地方是指定 `VPATH` 或 `vpath` 指令，用来指定 GNU `make` 查找前提条件的位置。例如，可以设置 `VPATH` 来在一系列 `:` 或空格分隔的路径中查找源文件：

```
VPATH = ../src:../thirdparty/src /src

vpath %c ../src:../thirdparty/src /src
```

GNU `make` 会在冒号或空格处正确地拆分路径。在 Windows 系统上，GNU `make` 的原生构建使用 `;` 作为 `VPATH`（和 `vpath`）的路径分隔符，因为 `:` 被用于驱动器字母。在 Windows 上，GNU `make` 实际上会智能地在冒号处拆分路径，除非它看起来像一个驱动器字母（一个字母后跟一个冒号）。这种驱动器字母的智能会在路径中有单个字母的目录名时造成问题：在这种情况下，必须使用 `;` 作为路径分隔符。否则，GNU `make` 会认为它是一个驱动器：

```
VPATH = ../src;../thirdparty/src /src

vpath %c ../src;../thirdparty/src /src
```

在 POSIX 和 Windows 系统中，路径中的空格是 `VPATH` 和 `vpath` 中的分隔符。所以，使用空格是跨平台 makefile 的最佳选择。

## 使用 / 或 \

在 POSIX 系统中，`/` 是路径分隔符，而在 Windows 系统中是 `\`。在 makefile 中常见的路径构建方式如下：

```
SRCDIR := src
MODULE_DIR := module_1

MODULE_SRCS := $(SRCDIR)/$(MODULE_DIR)
```

理想情况下，应该删除 POSIX-only 的 `/`，并用一个能够与不同分隔符兼容的东西来替代它。一个方法是定义一个名为 `/` 的变量（GNU `make` 允许几乎任何东西作为变量名），并用它代替 `/`：

```
/ := /

SRCDIR := src
MODULE_DIR := module_1

MODULE_SRCS := $(SRCDIR)$/$(MODULE_DIR)
```

如果这让你感到不舒服，可以简单地称它为 `SEP`：

```
SEP := /

SRCDIR := src
MODULE_DIR := module_1

MODULE_SRCS := $(SRCDIR)$(SEP)$(MODULE_DIR)
```

现在，当你切换到 Windows 时，只需将 `/`（或 `SEP`）重新定义为 `\`。由于 GNU `make` 将 `\` 解释为行继续符且无法转义，因此很难将字面上的 `\` 作为变量值分配，因此这里使用 `$(strip)` 来定义它。

```
/ := $(strip \)

SRCDIR := src
MODULE_DIR := module_1

MODULE_SRCS := $(SRCDIR)$/$(MODULE_DIR)
```

但是，请注意，GNU `make` 的 Windows 版本也会接受 `/` 作为路径分隔符，因此像 `c:/src` 这样的路径是合法的。使用这些路径可以简化 makefile，但在将它们传递给期望使用 `\` 分隔路径的本地 Windows 工具时需要小心。如果需要这样做，可以改用以下方法：

```
forward-to-backward = $(subst /,\,$1)
```

这个简单的函数将把正斜杠路径转换为反斜杠路径。

## Windows 特殊情况：不区分大小写但保持大小写

在 POSIX 系统上，文件名是区分大小写的；而在 Windows 上则不是。在 Windows 上，`File`、`file` 和 `FILE` 是同一个文件。但 Windows 的一个特殊之处在于，第一次访问文件时，会记录并保留所使用的具体大小写。因此，如果我们 `touch File`，它会显示为 `File`，但可以作为 `FILE`、`file` 或其他任何大小写组合来访问。

默认情况下，GNU `make` 执行区分大小写的目标比较，因此以下 makefile 的行为可能并不是你预期的：

```
.PHONY: all
all: File

file:
→ @touch $@
```

如此使用时，这个文件会导致错误，但你可以在 Windows 上编译 GNU `make` 以进行不区分大小写的比较（使用 `HAVE_CASE_INSENSITIVE_FS` 构建选项）。

当 makefile 中指定的目标也出现在通配符搜索中时，这个特殊情况更容易发生，因为操作系统可能会返回与 makefile 中使用的大小写不同的文件名。目标名称可能会在大小写上有所不同，这可能导致意外的 `No rule to make` 错误。

## 内置路径函数和变量

你可以使用内置的 `CURDIR` 来确定当前工作目录。请注意，`CURDIR` 会跟随符号链接。如果你在 `/foo` 目录下，但 `/foo` 实际上是指向 `/somewhere/foo` 的符号链接，那么 `CURDIR` 会报告目录为 `/somewhere/foo`。如果你需要不跟随符号链接的目录名称，可以使用环境变量 `PWD` 的值：

```
CURRENT_DIRECTORY := $(PWD)
```

但一定要在 makefile 的其他部分更改 `PWD` 之前获取其值：它可以像从环境导入的任何其他变量一样被修改。

你还可以使用 GNU `make` 3.80 引入的 `MAKEFILE_LIST` 变量来找到当前 makefile 存储的目录。在 makefile 开头，可以通过以下方式提取其目录：

```
CURRENT_MAKEFILE := $(word $(words $(MAKEFILE_LIST)),$(MAKEFILE_LIST))
MAKEFILE_DIRECTORY := $(dir $(CURRENT_MAKEFILE))
```

GNU `make` 提供了用于拆分路径为组件的函数：`dir`、`notdir`、`basename` 和 `suffix`。

假设文件名 `/foo/bar/source.c` 存储在变量 `FILE` 中。你可以使用 `dir`、`notdir`、`basename` 和 `suffix` 函数来提取目录、文件名和后缀。所以，要获取目录，例如，使用 `$(dir $(FILE))`。表 4-2 显示了这些函数及其结果。

表 4-2. `dir`、`notdir`、`basename` 和 `suffix` 的结果

| 函数 | 结果 |
| --- | --- |
| `dir` | `/foo/bar/` |
| `notdir` | `source.c` |
| `basename` | `source` |
| `suffix` | `.c` |

可以看到，目录部分、非目录部分、后缀（或扩展名）以及去除后缀的非目录部分都已被提取。这四个函数使得文件名操作变得简单。如果没有指定目录，GNU `make` 使用当前目录（`./`）。例如，假设 `FILE` 只是 `source.c`。表 4-3 显示了每个函数的结果。

表 4-3. 没有指定目录时，`dir`、`notdir`、`basename` 和 `suffix` 的结果

| 函数 | 结果 |
| --- | --- |
| `dir` | `./` |
| `notdir` | `source.c` |
| `basename` | `source` |
| `suffix` | `.c` |

因为这些函数通常与 GNU `make` 的自动变量（如 `$@`）一起使用，所以 GNU `make` 提供了一种修饰符语法。向任何自动变量添加 `D` 或 `F` 等价于在其上调用 `$(dir)` 或 `$(notdir)`。例如，`$(@D)` 等价于 `$(dir $@)`，而 `$(@F)` 与 `$(notdir $@)` 相同。

## 3.81 版本中的有用函数：abspath 和 realpath

`realpath` 是 GNU `make` 对 C 库 `realpath` 函数的封装，它移除 `./`、解析 `../`、删除重复的 `/` 并跟随符号链接。`realpath` 的参数必须存在于文件系统中。`realpath` 返回的路径是绝对路径。如果路径不存在，该函数将返回一个空字符串。

例如，你可以像这样找到当前目录的完整路径：`current := $(realpath ./)`。

`abspath` 类似，但不会跟随符号链接，并且其参数不必指向现有的文件或目录。

# 乌斯曼定律

`make clean` 并不一定会清理干净。这就是乌斯曼定律（以我一位聪明的同事命名，他花了几个月时间与真实世界的 makefile 一起工作）。`make clean` 的目的是恢复到一个可以从头开始重新构建的状态，但往往并不会。继续阅读以了解为什么。

## 人为因素

OpenSSL makefile 中的 `clean` 规则如下所示：

```
clean:
→  rm -f *.o *.obj lib tags core .pure .nfs* *.old *.bak fluff $(EXE)
```

注意，它是一个长长的、明显由人工维护的目录、模式和文件名列表，这些都需要被删除才能回到干净的状态。人工维护意味着人工错误。假设有人添加了一个规则，创建一个带有固定名称的临时文件。这个临时文件应该添加到 `clean` 规则中，但它很可能不会被添加。

乌斯曼定律再次出现。

## 糟糕的命名

这是许多自动生成的 Makefile 中找到的一个代码片段：

```
mostlyclean::
→ rm -f *.o

clean:: mostlyclean
→ -$(LIBTOOL) --mode=clean rm -f $(program) $(programs)
→ rm -f $(library).a squeeze *.bad *.dvi *.lj

extraclean::
→ rm -f *.aux *.bak *.bbl *.blg *.dvi *.log *.pl *.tfm *.vf *.vpl
→ rm -f *.*pk *.*gf *.mpx *.i *.s *~ *.orig *.rej *\#*
→ rm -f CONTENTS.tex a.out core mfput.* texput.* mpout.*
```

在这个例子中，三种 `clean` 似乎有不同的清洁程度：`mostlyclean`、`clean` 和 `extraclean`。

`mostlyclean` 只是删除从源代码编译的目标文件。`clean` 做到这一点，并删除生成的库和一些其他文件。你可能会认为 `extraclean` 会删除比其他两个更多的文件，但它实际上删除的是一组不同的文件。我还见过包含 `reallyclean`、`veryclean`、`deepclean`，甚至 `partiallyclean` 规则的 Makefile！

当你从命名中无法判断到底是做什么时，它很容易导致未来潜在的问题。

乌斯曼定律再次出现。

## 静默失败

这是另一个有时能工作的 Makefile 代码片段：

```
clean:
→ @-rm *.o &> /dev/null
```

`@` 表示命令不会被回显。`-` 表示忽略返回的任何错误，并且所有输出都会被重定向到 `/dev/null`，使其不可见。由于 `rm` 命令上没有 `-f` 选项，任何失败（例如权限问题）将完全不被注意到。

乌斯曼定律再次出现。

## 递归清理

许多 Makefile 是递归的，因此 `make clean` 也必须是递归的，因此你会看到如下模式：

```
SUBDIRS = library executable

.PHONY: clean
clean:
→ for dir in $(SUBDIRS); do \
→ $(MAKE) -C $$dir clean; \
→ done
```

这样的问题在于，它意味着 `make clean` 必须在 `SUBDIR` 中的每个目录中都能正确工作，从而增加了出错的机会。

乌斯曼定律再次出现。

# GNU make 并行化的陷阱与好处

许多构建过程需要运行数小时，因此构建管理人员通常输入 `make` 命令后就回家睡觉。GNU `make` 解决这个问题的方法是并行执行：一个简单的命令行选项，指示 GNU `make` 使用 Makefile 中的依赖信息并行运行任务，确保按正确的顺序执行。

然而，在实践中，GNU `make` 的并行执行受到一个严重限制，即几乎所有 Makefile 都假设它们的规则将按顺序执行。在编写 Makefile 时，Makefile 的作者很少会*考虑并行性*。这会导致隐藏的陷阱，导致构建失败并显示致命错误，或者更糟糕的是，构建“成功”但在 GNU `make` 以并行模式运行时生成不正确的二进制文件。

本节将探讨 GNU `make` 并行化的陷阱以及如何绕过它们，以获得最大程度的并行性。

## 使用 -j（或 -jobs）

要以并行模式启动 GNU `make`，可以在命令行上指定 `-j` 或 `--jobs` 选项。该选项的参数是 GNU `make` 将并行运行的最大进程数。

例如，输入`make --jobs=4`允许 GNU `make`同时运行最多四个子进程，这样理论上可以获得 4 倍的加速。然而，理论上的时间被 Makefile 中的限制严重限制。要计算最大实际加速，您可以使用阿姆达尔定律（在阿姆达尔定律与并行化的极限中有讲解）。

在并行 GNU `make`中发现的一个简单但令人讨厌的问题是，由于作业不再按顺序执行（顺序取决于作业的执行时机），因此 GNU `make`的输出将根据作业执行的实际顺序随机排序。

幸运的是，这个问题在 GNU `make` 4.0 中通过`--output-sync`选项得到了处理，具体内容在第一章中有描述。

考虑 示例 4-9 中的例子：

示例 4-9. 一个简单的 Makefile 来说明并行构建

```
.PHONY: all
all: t5 t4 t1
→ @echo Making $@

t1: t3 t2
→ touch $@

t2:
→ cp t3 $@

t3:
→ touch $@

t4:
→ touch $@

t5:
→ touch $@
```

它构建了五个目标：`t1`、`t2`、`t3`、`t4` 和 `t5`。除了 `t2`（它是从 `t3` 复制的）之外，其他目标都只是简单地进行了触碰。

通过标准的 GNU `make`运行 示例 4-9，没有并行选项的情况下，会输出如下：

```
$ **make**
touch t5
touch t4
touch t3
cp t3 t2
touch t1
Making all
```

执行顺序每次都会相同，因为 GNU `make`会遵循先处理前置条件的深度优先顺序，并从左到右执行。注意，左到右的执行顺序（例如在`all`规则中，`t5`在`t4`之前构建，`t4`在`t1`之前构建）是 POSIX `make`标准的一部分。

现在如果以并行模式运行`make`，显然`t5`、`t4`和`t1`可以同时运行，因为它们之间没有依赖关系。同样，`t3`和`t2`也不相互依赖，因此它们可以并行执行。

并行运行 示例 4-9 的输出可能是：

```
$ **make --jobs=16**
touch t4
touch t5
touch t3
cp t3 t2
touch t1
Making all
```

或者甚至是：

```
$ **make --jobs=16**
touch t3
cp t3 t2
touch t4
touch t1
touch t5
Making all
```

这使得任何检查日志文件以检测构建问题的过程（例如比较日志文件）变得困难。不幸的是，在没有`--output-sync`选项的情况下，GNU `make` 没有简单的解决方案，因此除非你升级到 GNU `make` 4.0，否则只能忍受这种情况。

## 缺少的依赖项

示例 4-9 中的例子有一个额外的问题。作者在编写 Makefile 时陷入了经典的从左到右的陷阱，因此在并行执行时，可能会发生以下情况：

```
$ **make --jobs=16**
touch t5
touch t4
cp t3 t2
cp: cannot stat `t3': No such file or directory
make: *** [t2] Error 1
```

原因是，当并行运行时，构建` t2 `的规则可能会先于构建` t3 `的规则，而` t2 `需要` t3 `已经被构建。这在串行情况下没有发生，因为存在从左到右的假设：构建` t1 `的规则是` t1: t3 t2 `，这意味着` t3 `会在` t2 `之前构建。

但是，在 makefile 中并没有明确声明` t3 `必须在` t2 `之前构建。解决方法很简单：只需在 makefile 中添加` t2: t3 `即可。

这是一个简单的例子，展示了当 makefile 并行运行时缺失或隐式（从左到右的）依赖关系所带来的真实问题。如果 makefile 在并行运行时出错，值得立即检查是否缺少依赖关系，因为这些问题非常常见。

## 隐藏的临时文件问题

GNU `make`在并行运行时的另一种可能出错的方式是多个规则使用相同的临时文件。考虑示例 4-10 中的 makefile 例子：

示例 4-10. 一个隐藏的临时文件破坏并行构建

```
TMP_FILE := /tmp/scratch_file

.PHONY: all
all: t

t: t1 t2
→ cat t1 t2 > $@

t1:
→ echo Output from $@ > $(TMP_FILE)
→ cat $(TMP_FILE) > $@

t2:
→ echo Output from $@ > $(TMP_FILE)
→ cat $(TMP_FILE) > $@
```

在没有并行选项的情况下，GNU `make`会生成以下输出：

```
$ **make**
echo Output from t1 > /tmp/scratch_file
cat /tmp/scratch_file > t1
echo Output from t2 > /tmp/scratch_file
cat /tmp/scratch_file > t2
cat t1 t2 > t
```

并且`t`文件包含：

```
Output from t1
Output from t2
```

但在并行运行时，示例 4-10 会给出以下输出：

```
$ make --jobs=2
echo Output from t1 > /tmp/scratch_file
echo Output from t2 > /tmp/scratch_file
cat /tmp/scratch_file > t1
cat /tmp/scratch_file > t2
cat t1 t2 > t
```

现在`t`包含：

```
Output from t2
Output from t2
```

这是因为` t1 `和` t2 `之间没有依赖关系（因为它们都不需要对方的输出），所以它们可以并行运行。在输出中，你可以看到它们是并行运行的，但两个规则的输出是交错的。由于两个`echo`语句先运行，`t2`覆盖了` t1 `的输出，因此临时文件（由两个规则共享）在最终执行`cat`到` t1 `时具有错误的值，导致`t`的值错误。

这个例子看起来可能有些牵强，但在实际的 makefile 中，当并行运行时，会发生相同的情况，导致构建失败或生成错误的二进制文件。例如，`yacc`程序会生成名为`y.tab.c`和`y.tab.h`的临时文件。如果在同一目录下同时运行多个`yacc`，错误的文件可能会被错误的进程使用。

对于示例 4-10 中的 makefile，一种简单的解决方案是将`TMP_FILE`的定义改为`TMP_FILE = /tmp/scratch_file.$@`，这样其名称就会依赖于正在构建的目标。现在并行运行将如下所示：

```
$ **make --jobs=2**
echo Output from t1 > /tmp/scratch_file.t1
echo Output from t2 > /tmp/scratch_file.t2
cat /tmp/scratch_file.t1 > t1
cat /tmp/scratch_file.t2 > t2
cat t1 t2 > t
```

一个相关的问题是，当 makefile 中的多个任务写入共享文件时，即使它们从不读取该文件（例如，它们可能写入日志文件），为了写入访问而锁定文件也会导致竞争的任务停滞，从而降低并行构建的整体性能。

请参考示例 4-11 中的 makefile 示例，它使用`lockfile`命令锁定一个文件并模拟写锁。尽管文件被锁定，但每个任务会等待若干秒：

示例 4-11。锁定共享文件可能会锁住并行构建，使其以串行方式运行。

```
LOCK_FILE := lock.me

.PHONY: all
all: t1 t2
→ @echo done.

t1:
→ @lockfile $(LOCK_FILE)
→ @sleep 10
→ @rm -f $(LOCK_FILE)
→ @echo Finished $@
t2:
→ @lockfile $(LOCK_FILE)
→ @sleep 20
→ @rm -f $(LOCK_FILE)
→ @echo Finished $@
```

在串行构建中运行示例 4-11 大约需要 30 秒：

```
$ **time make**
Finished t1
Finished t2
done.
make 0.01s user 0.01s system 0% cpu 30.034 total
```

但是即使`t1`和`t2`应该能够并行运行，它在并行中也并不更快：

```
$ **time make -j4**
Finished t1
Finished t2
done.
make -j4 0.01s user 0.02s system 0% cpu 36.812 total
```

实际上，这会更慢，因为`lockfile`检测锁可用性的方法。正如您想象的那样，写锁文件可能会导致在本应支持并行的 makefile 中出现类似的延迟。

与文件锁定问题相关的是有关归档文件（`ar`文件）的风险。如果多个`ar`进程同时在同一归档文件上运行，归档文件可能会被损坏。在并行构建中，必须对归档更新进行锁定；否则，您需要防止依赖项同时在同一文件上运行多个`ar`命令。

防止并行问题的一种方法是在 makefile 中指定`.NOTPARALLEL`。如果看到该标志，整个`make`执行将以串行方式运行，`-j`或`--jobs`命令行选项将被忽略。`.NOTPARALLEL`是一个非常直接的工具，因为它会影响整个 GNU `make`的调用，但在递归的`make`情境下，它可能会很有用，例如在使用不支持并行的第三方 makefile 时。

## 正确的递归 make 做法

GNU `make`足够智能，能够在子 make 之间共享并行性，只要使用`$(MAKE)`的 makefile 在调用子 make 时小心处理。GNU `make`具有跨平台的消息传递机制（Windows 支持在 GNU `make` 4.0 中加入），使得子 make 能够使用通过`-j`或`--jobs`指定的所有可用任务，通过管道在`make`进程之间传递令牌。

唯一需要注意的地方是，您必须以一种实际允许子 make 并行运行的方式编写 makefile。经典的递归`make`样式使用 shell `for`循环处理每个子 make，不允许一次运行多个子 make。例如：

```
SUBDIRS := foo bar baz

.PHONY: all
all:
→ for d in $(SUBDIRS);     \
→ do                       \
→ $(MAKE) –directory=$$d;  \
→ done
```

这个代码有一个大问题：如果子 make 失败，`make`看起来会像是成功了。虽然可以修复这个问题，但修复方案越来越复杂：其他方法会更好。

在并行模式下运行时，`all` 规则会遍历每个子目录，并等待其 `$(MAKE)` 完成。尽管这些子 make 可以并行运行，但整体的 `make` 并不能并行，这意味着加速比不理想。例如，如果 `bar` 目录中的 `make` 一次只能运行四个作业，那么在一台 16 核心机器上运行也不会比在 4 核心机器上更快。

解决方案是移除 `for` 循环，并为每个目录替换为一个单独的规则：

```
SUBDIRS := foo bar baz

.PHONY: all $(SUBDIRS)
all: $(SUBDIRS)

$(SUBDIRS):
→ $(MAKE) --directory=$@
```

每个目录被认为是一个虚拟目标，因为目录本身并不会被实际构建。

现在每个目录都可以在其他目录运行的同时执行，并且并行度达到了最大；甚至可能存在目录间的依赖关系，导致某些子 make 在其他子 make 之前运行。当某个子 make 必须在另一个子 make 之前运行时，目录依赖关系非常有用。

## 阿姆达尔定律与并行化的极限

此外，项目中的并行化也存在实际限制。查看示例 4-12：

示例 4-12. 使用 `sleep` 模拟需要时间完成的作业的 makefile

```
.PHONY: all
all: t
→ @echo done
t: t1 t2 t3 t4 t5 t6 t7 t8 t9 t10 t11 t12
→ @sleep 10
→ @echo Made $@

t1:
→ @sleep 11
→ @echo Made $@

t2:
→ @sleep 4
→ @echo Made $@

t3: t5
→ @sleep 7
→ @echo Made $@

t4:
→ @sleep 9
→ @echo Made $@

t5: t8
→ @sleep 10
→ @echo Made $@

t6:
→ @sleep 2
→ @echo Made $@

t7:
→ @sleep 12
→ @echo Made $@

t8:
→ @sleep 3
→ @echo Made $@

t9: t10
→ @sleep 4
→ @echo Made $@

t10:
→ @sleep 6
→ @echo Made $@

t11: t12
→ @sleep 1
→ @echo Made $@

t12:
→ @sleep 9
→ @echo Made $@
```

在串行模式下运行时，完成整个过程大约需要 88 秒：

```
$ **time make**
Made t1
Made t2
Made t8
Made t5
Made t3
Made t4
Made t6
Made t7
Made t10
Made t9
Made t12
Made t11
Made t
done
make 0.04s user 0.03s system 0% cpu 1:28.68 total
```

假设可以根据需要提供任意数量的 CPU，最大加速比是多少？逐步分析 makefile，你会发现 `t` 的构建时间为 10 秒，其他所有任务必须在此之前完成。`t1`、`t2`、`t4`、`t6` 和 `t7` 都是独立的，其中最长的一个需要 12 秒。`t3` 等待 `t5`，而 `t5` 又依赖于 `t8`：这条链总共需要 20 秒。`t9` 依赖于 `t10`，需要 10 秒，`t11` 依赖于 `t12`，也需要 10 秒。

因此，这次构建的最长串行部分是 `t`、`t3`、`t5`、`t8` 的顺序，总共需要 30 秒。这次构建的速度永远不会超过 30 秒（或者说是比串行的 88 秒快 2.93 倍）。需要多少处理器才能达到这种加速？

通常，最大加速比由阿姆达尔定律控制：如果 `F` 是无法并行化的构建部分的比例，`N` 是可用处理器的数量，那么最大加速比为 `1 / ( F + ( 1 - F ) / N )`。

在示例 4-12 中，34% 的构建无法并行化。表 4-4 显示了应用阿姆达尔定律的结果：

表 4-4. 基于处理器数量的最大加速比

| 处理器数量 | 最大加速比 |
| --- | --- |
| 2 | 1.49x |
| 3 | 1.79x |
| 4 | 1.98x |
| 5 | 2.12x |
| 6 | 2.22x |
| 7 | 2.30x |
| 8 | 2.37x |
| 9 | 2.42x |
| 10 | 2.46x |
| 11 | 2.50x |
| 12 | 2.53x |

对于这个小型构建，Amdahl 定律预测的最大加速比在大约八个处理器时达到平台期。实际的加速平台期进一步受到限制，因为构建中只有 13 个可能的任务。

查看构建的结构，我们可以看到最多使用八个处理器，因为五个任务可以并行运行且没有任何依赖关系：`t1`、`t2`、`t4`、`t6` 和 `t7`。然后三个小任务链可以各自使用一个处理器：`t3`、`t5` 和 `t8`；`t9` 和 `t10`；以及 `t11` 和 `t12`。构建 `t` 时可以重用其中一个空闲的处理器，因为到那时所有的处理器都将处于空闲状态。

Amdahl 定律在构建时间上的实际影响，通常出现在具有链接步骤的语言中，比如 C 和 C++。通常，所有目标文件在链接步骤之前就已构建完毕，然后会进行一个单独的（通常非常大的）链接过程。这个链接过程通常无法并行化，成为构建并行化的限制因素。

# 使 `$(wildcard)` 递归

内建的 `$(wildcard)` 函数不是递归的：它仅在单个目录中查找文件。你可以在 `$(wildcard)` 中使用多个通配符模式，并利用这些模式查找子目录中的文件。例如，`$(wildcard */*.c)` 可以查找当前目录下所有子目录中的 `.c` 文件。但如果你需要在任意目录树中进行查找，则没有内建的方法来实现。

幸运的是，创建一个递归版本的 `$(wildcard)` 非常简单，如下所示：

```
rwildcard=$(foreach d,$(wildcard $1*),$(call rwildcard,$d/,$2) $(filter $(subst *,%,$2),$d))
```

`rwildcard` 函数接受两个参数：第一个是开始查找的目录（此参数可以为空，表示从当前目录开始查找），第二个是要在每个目录中查找的文件的通配符模式。

例如，要查找当前目录（以及其子目录）下的所有 `.c` 文件，可以使用以下命令：

```
$(call rwildcard,,*.c)
```

或者要查找 `/tmp` 下的所有 `.c` 文件，可以使用以下命令：

```
$(call rwildcard,/tmp/,*.c)
```

`rwildcard` 也支持多个模式。例如：

```
$(call rwildcard,/src/,*.c *.h)
```

这将查找 `/src/` 下的所有 `.c` 和 `.h` 文件。

# 我在哪个 Makefile 中？

一个常见的问题是：有没有方法查找当前 makefile 的名称和路径？通常所说的“当前”是指 GNU `make` 当前正在解析的 makefile。没有内建的快捷方法来获取答案，但可以通过使用 GNU `make` 变量 `MAKEFILE_LIST` 来实现。

`MAKEFILE_LIST` 是当前加载或 `include` 的 makefile 列表。每次加载或 `include` 一个 makefile 时，`MAKEFILE_LIST` 会将其路径和名称添加到列表中。变量中的路径和名称是相对于当前工作目录的（即 GNU `make` 启动时所在的目录，或者通过 `-C` 或 `--directory` 选项更改的目录），但你可以通过 `CURDIR` 变量访问当前目录。

所以，使用这个方法，你可以定义一个 GNU `make` 函数（我们称之为 `where-am-i`），它将返回当前的 makefile（它使用 `$(word)` 从列表中获取最后一个 makefile 的名称）。

```
where-am-i = $(CURDIR)/$(word $(words $(MAKEFILE_LIST)),$(MAKEFILE_LIST))
```

然后，每当你想要查找当前 makefile 的完整路径时，只需在文件顶部写入以下内容：

```
THIS_MAKEFILE := $(call where-am-i)
```

这行代码放在文件顶部非常重要，因为任何 `include` 语句都会改变 `MAKEFILE_LIST` 的值，因此你希望在发生改变之前先获取当前 makefile 的位置。

示例 4-13 展示了一个使用 `where-am-i` 并从 `foo/` 子目录中包含另一个 makefile 的示例，而该 makefile 又包含了来自 `foo/bar/` 目录的 makefile。

示例 4-13. 一个可以确定其在文件系统中位置的 makefile

```
where-am-i = $(CURDIR)/$(word ($words $(MAKEFILE_LIST)),$(MAKEFILE_LIST)

include foo/makefile
```

`foo/makefile` 的内容见 示例 4-14。

示例 4-14. 一个由示例 4-13 包含的 makefile

```
THIS_MAKEFILE := $(call where-am-i)
$(warning $(THIS_MAKEFILE))

include foo/bar/makefile
```

`foo/bar/makefile` 的内容见 示例 4-15。

示例 4-15. 一个由示例 4-14 包含的 makefile

```
THIS_MAKEFILE := $(call where-am-i)
$(warning $(THIS_MAKEFILE))
```

将 示例 4-13、示例 4-14 和 示例 4-15 的三个 makefile 放在 `/tmp`（及其子目录）下，并运行 GNU `make`，会得到如下输出：

```
foo/makefile:2: /tmp/foo/makefile
foo/bar/makefile:2: /tmp/foo/bar/makefile
```

在本章中，我们探讨了 makefile 创建者和维护者在实际工作中常遇到的一些问题。在任何一个使用`make`的大型项目中，你很可能会遇到一个或多个（甚至全部！）这样的问题。
