# 第二章. Makefile 调试

本章介绍了一些在调试 makefile 时可能有用的技巧。由于缺乏内置的调试工具，再加上追踪 `make` 中变量的复杂性，这使得理解为什么某个目标被（或更常见的是没有）构建变得非常具有挑战性。

本章中的第一个配方展示了你可以添加到 makefile 中的最有用的一行；它相当于在代码中插入一个用于调试的打印语句。

# 打印 Makefile 变量的值

如果你曾经查看过一个 makefile，你会意识到 makefile 变量（通常简称为变量）是任何 `make` 过程的骨干。变量通常定义了哪些文件将被编译、传递给编译器的命令行参数，甚至编译器的位置。如果你曾经试图调试一个 makefile，你知道自己问的第一个问题是，“变量 `X` 的值是什么？”

GNU `make` 没有内置调试器，也不像 Perl 或 Python 这样的脚本语言那样提供交互式的功能。那么，如何找出一个变量的值呢？

看看 示例 2-1 中展示的简单 makefile，它仅设置了几个变量：

示例 2-1. 一个设置各种变量的简单 makefile

```
X=$(YS) hate $(ZS)
Y=dog
YS=$(Y)$(S)
Z=cat
ZS=$(Z)$(S)
S=s

all:
```

`X` 的值是什么？

这个 makefile 的小巧和简洁使得追踪所有变量的赋值变得可行，但即便如此，要得出 `X` 的值是 `dogs hate cats` 还是需要一些工作。如果是一个拥有成千上万行的 makefile，充分利用 GNU `make` 的变量和函数，要弄清楚一个变量的值确实可能会非常繁琐。幸运的是，这里有一个小小的 `make` 配方，它可以为你完成所有的工作：

```
print-%: ; @echo $* = $($*)
```

现在，你可以使用以下命令来查找变量 `X` 的值：

```
$ **make print-X**
```

由于没有为 `print-X` 目标定义显式规则，`make` 会查找模式规则，找到 `print-%`（`%` 起到通配符的作用），并运行相关联的命令。这个命令使用 `$*`，一个特殊的变量，包含与规则中的 `%` 匹配的值，来打印变量的名称，然后使用 `$($*)` 来获取其值。这在 makefile 中是一个非常有用的技巧，因为它允许计算变量的名称。在这种情况下，要打印的变量名称来自另一个变量 `$*`。

下面是如何使用这个规则来打印在 示例 2-1 中定义的变量的值：

```
$ **make print-X**
X = dogs hate cats
$ **make print-YS**
YS = dogs
$ **make print-S**
S = s
```

有时了解一个变量是*如何*被定义的非常有用。`make`有一个`$origin`函数，它返回一个字符串，包含变量的*类型*——即变量是如何定义的，是否在 makefile 中、命令行中，或者在环境中定义。修改`print-%`以同时输出来源信息也很简单：

```
print-%: ; @echo $* = '$($*)' from $(origin $*)
```

现在我们看到`YS`是在 makefile 中定义的：

```
$ **make print-YS**
YS = 'dogs' from file
```

如果我们在命令行中覆盖了`YS`的值，我们将看到：

```
$ **make print-YS YS=fleas**
YS = 'fleas' from command line
```

由于`YS`是在`make`命令行中设置的，因此它的`$(origin)`现在是`command line`，而不再是`file`。

# 打印每个 Makefile 变量

上一部分展示了如何通过特殊规则打印单个 makefile 变量的值。那么，如果你想打印 makefile 中定义的所有变量呢？

幸运的是，GNU `make` 3.80 引入了几个新功能，使得通过单个规则打印 makefile 中定义的所有变量的值变得可行。

再次考虑示例 2-1。它设置了五个变量：`X`、`Y`、`Z`、`S`、`YS`和`ZS`。向示例中添加以下行会创建一个名为`printvars`的目标，该目标将打印 makefile 中定义的所有变量，如示例 2-2 所示。

示例 2-2. 打印所有变量的目标

```
.PHONY: printvars
printvars:
→ @$(foreach V,$(sort $(.VARIABLES)),            \
→ $(if $(filter-out environ% default automatic,  \
→ $(origin $V)),$(info $V=$($V) ($(value $V)))))
```

在我们仔细查看它是如何工作的之前，像示例 2-3 中的变量")所示那样先自己尝试一下。

示例 2-3. 使用`printvars`打印的示例 2-1 中的所有变量

```
$ **make printvars**
MAKEFILE_LIST= Makefile helper.mak ( Makefile helper.mak)
MAKEFLAGS= ()
S=s (s)
SHELL=/bin/sh (/bin/sh)
X=dogs hate cats ($(YS) hate $(ZS))
Y=dog (dog)
YS=dogs ($(Y)$(S))
Z=cat (cat)
ZS=cats ($(Z)$(S))
```

注意到`make`引入了三个额外的变量，这些变量并未明确在文件中定义——`MAKEFILE_LIST`、`MAKEFLAGS`和`SHELL`——但其他变量都在 makefile 中定义。每一行显示了变量的名称、其完全替换的值以及定义的方式。

当我们将打印变量的长且复杂的行重新格式化后，它会变得更容易理解：

```
   $(foreach V,$(sort $(.VARIABLES)),
     $(if
➊    $(filter-out environment% default automatic,$(origin $V)),
       $(info $V=$($V) ($(value $V)))
     )
   )
```

`.VARIABLES`变量是 GNU `make` 3.80 中的一个新特性：它的值是一个包含 makefile 中定义的所有变量名称的列表。首先，代码将其排序：`$(sort $(.VARIABLES))`。然后，它逐个变量名地遍历排序后的列表，并将`V`设置为每个名称：`$(foreach V,$(sort (.VARIABLES)),...)`。

对于每个变量名，循环会决定是否打印该变量，或者忽略它，这取决于该变量是如何定义的。如果它是内建变量，如`$@`或`$(CC)`，或者来自环境变量，它不应该被打印。这个决定由➊处的条件表达式做出。它首先通过调用`$(origin $V)`来确定变量`$V`是如何定义的。此调用返回一个字符串，描述了该变量的定义方式：`environment`表示环境变量，`file`表示在 makefile 中定义的变量，`default`表示`make`定义的内容。`$(filter-out)`语句表示，如果`$(origin)`的结果匹配任何模式`environment%`、`default`或`automatic`（对于`make`的自动变量，如`$@`、`$<`等，`$(origin)`会返回`automatic`），则返回空字符串；否则，保持原样。这意味着，`$(if)`的条件只有在变量是在 makefile 中定义或在命令行上设置时才为真。

如果`$(if)`的条件为真，那么`$(info $V=$($V) ($(value $V)))`会输出一条包含变量名、其完全展开的值以及其定义值的消息。`$(value)`函数是 GNU `make` 3.80 中的另一个新特性；它输出变量的值而不进行展开。在示例 2-3 中，`$(YS)`将返回值`dogs`，但是`$(value YS)`将返回`$(Y)$(S)`。也就是说，`$(value YS)`展示的是`YS`的定义方式，而不是其最终值。这是一个非常有用的调试特性。

# 跟踪变量值

随着 makefile 的增大，可能会很难找出一个变量的使用位置。尤其是因为 GNU `make`的递归变量：一个变量的使用可能被隐藏在 makefile 中某个其他变量定义的深处。本条食谱展示了如何跟踪各个变量在使用时的情况。

在这个例子中，我们将使用示例 2-4 中的 makefile（这些行已被编号，便于后续参考）。

示例 2-4. 用于跟踪的示例 makefile

```
 1 X=$(YS) hate $(ZS)
 2 Y=dog
 3 YS=$(Y)$(S)
 4 Z=cat
 5 ZS=$(Z)$(S)
 6 S=s
 7
 8 all: $(YS) $(ZS)
 9 all: ; @echo $(X)
10
11 $(YS): ; @echo $(Y) $(Y)
12 $(ZS): ; @echo $(Z) $(Z)
```

运行时，这个 makefile 会打印：

```
dog dog
cat cat
dogs hate cats
```

如示例 2-4 所示，makefile 包含了许多递归定义的变量，并且在规则定义和命令中使用了它们。

## 跟踪变量的使用

如果你跟踪示例 2-4，你会看到变量`$(Y)`在第 8、9 和 11 行被使用，并且在第 12 行出现了两次。变量使用的频率真是惊人！原因是，`make`仅在需要时（即当变量被使用并展开时）获取递归展开变量的值（比如在示例 2-4 中的`YS`），而递归展开的变量通常是深度嵌套的。

跟踪一个变量通过示例 2-4 中的简单 Makefile 已经够麻烦了，但要跟踪一个真实的 Makefile 几乎是不可能的。幸运的是，可以通过以下代码让`make`为你完成这项工作，应该将其添加到要跟踪的 Makefile 的开始部分（它只会在显式调用时使用）：

```
ifdef TRACE
.PHONY: _trace _value
_trace: ; @$(MAKE) --no-print-directory TRACE= \
      $(TRACE)='$$(warning TRACE $(TRACE))$(shell $(MAKE) TRACE=$(TRACE) _value)'
_value: ; @echo '$(value $(TRACE))'
endif
```

在我们深入了解它是如何工作的之前，先看一个例子，展示如何在我们的示例 Makefile 中跟踪`Y`的值。要使用跟踪器，只需告诉`make`运行`trace`目标，方法是将`TRACE`变量设置为你要跟踪的变量的名称。跟踪变量`Y`的方式如下：

```
$ **make TRACE=Y**
Makefile:8: TRACE Y
Makefile:11: TRACE Y
Makefile:12: TRACE Y
Makefile:12: TRACE Y
dog dog
cat cat
Makefile:9: TRACE Y
dogs hate cats
```

从`TRACE`输出中，你可以看到`Y`首次出现在第 8 行的`all`目标定义中，该目标通过`$(YS)`引用了`Y`；然后在第 11 行，`cats`目标的定义中，也使用了`$(YS)`；接着在第 12 行出现两次，直接引用了`$(Y)`；最后，在第 9 行通过`$(X)`使用了`$(YS)`，而`$(YS)`又引用了`$(Y)`。

同样，我们也可以使用这个跟踪器来查找`$(S)`被使用的位置：

```
$ **make TRACE=S**
Makefile:8: TRACE S
Makefile:8: TRACE S
Makefile:11: TRACE S
Makefile:12: TRACE S
dog dog
cat cat
Makefile:9: TRACE S
Makefile:9: TRACE S
dogs hate cats
```

输出显示，`S`首先在第 8 行使用了两次（`all`目标使用了`XS`和`YS`，这两者都使用了`S`）。然后，`S`再次出现在第 4 行（因为使用了`YS`）和第 12 行（因为使用了`XS`）。最后，`S`在第 9 行被使用了两次，当`X`被回显时，因为`X`被`XS`和`YS`使用，这两者都使用了`S`。

## 变量跟踪器的工作原理

GNU `make`有一个特殊的`$(warning)`函数，可以将警告信息输出到`STDERR`并返回空字符串。从高级别来看，我们的跟踪代码将要跟踪的变量的值更改为包含一个`$(warning)`消息。每次变量展开时，警告都会被打印，而每当`make`输出警告信息时，它会打印使用中的 Makefile 名称和行号。

例如，假设`Y`的定义从

```
Y=dog
```

更改为

```
Y=$(warning TRACE Y)dog
```

然后，每当`$(Y)`被展开时，就会生成一个警告，并且`$(Y)`的值会是`dog`。由于`$(warning)`不返回任何值，因此`Y`的值不受影响。

要添加这个`$(warning)`调用，跟踪器代码首先获取要跟踪的变量的未扩展值，然后将其与适当的`$(warning)`一起预先处理，最后使用经过特殊修改的变量值运行所需的`make`。它使用`$(value)`函数，正如你在示例 2-2 中看到的那样，`$(value)`可以让你获取变量的未扩展值。

这里是跟踪器如何工作的详细说明。如果`TRACE`已定义，`make`将处理跟踪器定义的代码块。在这种情况下，由于`_trace`是第一个遇到的目标，它将是默认运行的规则。`_trace`规则包含一个单一且复杂的命令：

```
@$(MAKE) --no-print-directory TRACE= \
       $(TRACE)='$$(warning TRACE $(TRACE))$(shell $(MAKE) TRACE=$(TRACE) _value)'
```

在命令的右侧是一个`$(shell)`调用，它会使用不同的目标重新运行 makefile。例如，如果我们在跟踪`YS`，这个`$(shell)`调用会执行以下命令：

```
make TRACE=YS _value
```

这将运行`_value`规则，定义如下：

```
_value: ; @echo '$(value $(TRACE))'
```

因为`TRACE`已被设置为`YS`，所以这个规则只是回显`YS`的定义，即字面量字符串`$(Y)$(S)`。因此，`$(shell)`最终会评估为这个值。

那个`$(shell)`调用实际上是在一个命令行变量定义中（通常称为*命令行覆盖*）：

```
$(TRACE)='$$(warning TRACE $(TRACE))$(shell $(MAKE)TRACE=$(TRACE) _value)'
```

这添加了`$(warning)`，用于输出`TRACE X`消息。请注意，定义的变量名是一个计算值：它的名称包含在`$(TRACE)`中。当跟踪`YS`时，这个定义变成了：

```
YS='$(warning TRACE YS)$(Y)$(S)'
```

单引号用于防止 shell 看到`$`符号。双`$$`用于防止`make`看到`$`。在这两种情况下，都会发生变量扩展（无论是在`make`中还是由 shell 进行），我们希望延迟任何变量扩展，直到`YS`实际使用时。

最后，`_trace`规则会递归地运行`make`：

```
make TRACE= YS='$(warning TRACE YS)$(Y)$(S)'
```

`TRACE`的值被重置为空字符串，因为这个递归调用的`make`应该运行真实的规则，而不是跟踪规则。此外，它覆盖了`YS`的值。回想一下，命令行上定义的变量会覆盖 makefile 中的定义：即使`YS`在 makefile 中有定义，带有`warning`的命令行定义才是被使用的。现在，每次扩展`YS`时，都会打印一个警告。

请注意，这种技术不适用于目标特定的变量。`make`允许你按示例 2-5 的方式定义一个目标特定的变量：

示例 2-5. 定义目标特定变量

```
all: FOO=foo
all: a
all: ; @echo $(FOO)

a: ; @echo $(FOO)
```

变量`FOO`将在构建`all`规则和`all`的任何前提条件中具有值`foo`。在示例 2-5 中的 makefile 会打印`foo`两次，因为`FOO`在`all`和`a`规则中都被定义。跟踪器无法获取`FOO`的值，实际上会导致该 makefile 行为不正确。

跟踪器的工作原理是通过重新定义被跟踪的变量，如前所述。由于这发生在规则定义之外，跟踪器无法获取目标特定变量的值。例如，在示例 2-5 中，`FOO`仅在运行`all`或`a`规则时定义。跟踪器无法获取其值。在该 makefile 上使用跟踪器跟踪`FOO`会导致错误的行为：

```
$ **make TRACE=FOO**
Makefile:10: TRACE FOO
Makefile:8: TRACE FOO
```

这应该输出`foo`两次（一次是`all`规则，一次是`a`规则），但跟踪器已经重新定义了`FOO`并搞乱了它的值。不要将这个跟踪器用于目标特定变量。

`$(warning)`函数将其输出发送到`STDERR`，这使得可以将正常的`make`输出与跟踪输出分开。只需将`STDERR`重定向到跟踪日志文件。以下是一个示例：

```
$ **make TRACE=S 2> trace.log**
dog dog
cat cat
dogs hate cats
```

这个命令将把正常的`make`输出写到命令行，同时将跟踪输出重定向到*trace.log*。

# 跟踪规则执行

在 GNU `make` 4.0 之前，没有内建的方式来跟踪 makefile 目标的执行顺序。GNU `make` 4.0 增加了`--trace`选项，我在 GNU make 4.0 跟踪中有详细讲解，但如果你需要使用更早版本的`make`，有其他方法来跟踪 makefile 是很有用的。这里展示的技术适用于 GNU `make` 4.0 及更早版本。

### 注意

*如果你曾经盯着一份晦涩的日志输出，心里想：“是什么规则导致了这个输出？”或者“`foo`规则的输出在哪里？”那么这一节正是为你准备的。说实话，谁没有好奇过 GNU `make`的日志文件输出意味着什么呢？*

## 示例

本节使用以下示例 makefile：

```
.PHONY: all
all: foo.o bar

bar: ; @touch $@
```

它构建了两个文件：`foo.o`和`bar`。我们假设`foo.c`存在，这样`make`的内建规则就会创建`foo.o`；而`bar`是一个简单的规则，只是触碰`$@`。如果你第一次运行这个 makefile 的`make`，你会看到以下输出：

```
$ **make**
cc -c -o foo.o foo.c
```

该日志输出相当晦涩。没有看到`bar`的规则被执行的迹象（因为`touch $@`使用了`@`修饰符，这会阻止命令被打印）。也没有迹象表明是`foo.o`的规则生成了`cc`编译行。同样也没有显示`all`规则被使用的迹象。

当然，你可以使用`make -n`（它只会打印要执行的命令，而不会实际执行它们）来查看 GNU `make`将会执行的工作：

```
$ **make -n**
cc -c -o foo.o foo.c
touch bar
```

在这种情况下它是实用的，但通常 `make -n` 的输出可能像普通的日志文件一样晦涩，而且它没有提供将日志文件中的行与 makefile 中的行匹配的方法。

## SHELL 黑客

增强 GNU `make` 输出的一种简单方法是重新定义 `SHELL`，这是一个内置变量，包含 `make` 执行命令时要使用的 shell 的名称。大多数 shell 都有一个 `-x` 选项，可以使它们打印出每个即将执行的命令；因此，如果你通过在 makefile 中附加 `-x` 来修改 `SHELL`，它会导致每个命令在 makefile 执行时都被打印出来。

这是一个使用 GNU `make` 的 `+=` 操作符修改过的 makefile 示例，该操作符将 `-x` 附加到 `SHELL`：

```
SHELL += -x

.PHONY: all
all: foo.o bar

bar: ; @touch $@
```

在某些 shell 中，这可能不起作用（shell 可能期望接收单个选项单词）。在 GNU `make` 4.0 及更高版本中，一个名为 `.SHELLFLAGS` 的变量包含了 shell 的标志，并可以被设置来避免这个问题，而不需要修改 `SHELL`。

现在，makefile 输出显示 `touch bar` 是由 `bar` 规则生成的：

```
$ **make**
cc -c -o foo.o foo.c
+ cc -c -o foo.o foo.c
+ touch bar
```

`SHELL` 技术有一个缺点：它会使 `make` 变慢。如果 `SHELL` 保持不变，`make` 通常会避免使用 shell，前提是它知道可以直接执行命令——例如，对于简单的操作如编译和链接。但一旦在 makefile 中重新定义了 `SHELL`，`make` 就会始终使用 shell，从而导致变慢。

当然，这并不意味着这是一个糟糕的调试技巧：为了短暂的速度下降，获得额外的信息是一个非常小的代价。但是重新定义 `SHELL` 并不能帮助追踪日志文件中的行与 makefile 中的行之间的关系。幸运的是，通过更聪明地重新定义 `SHELL`，这是可以做到的。

## 更聪明的 SHELL 黑客

如果 `SHELL` 已被重新定义，`make` 会在执行每条规则的每一行之前扩展它的值。这意味着，如果 `SHELL` 的扩展输出信息，就可以在每条规则执行之前打印出信息。

正如你在追踪变量值中看到的，`$(warning)` 函数会帮助输出你选择的字符串，并附上 makefile 的名称和 `$(warning)` 所在行的行号。通过将 `$(warning)` 调用添加到 `SHELL` 中，每次 `SHELL` 扩展时都可以打印详细信息。以下代码片段就实现了这一点：

```
OLD_SHELL := $(SHELL)
SHELL = $(warning Building $@)$(OLD_SHELL)

.PHONY: all
all: foo.o bar

bar: ; @touch $@
```

第一行将 `SHELL` 的正常值捕获到一个名为 `OLD_SHELL` 的变量中。注意使用 `:=` 来获取 `SHELL` 的最终值，而不是它的定义。第二行定义了 `SHELL`，使其包括旧的 shell 值和一个会打印正在构建的目标名称的 `$(warning)`。

现在运行 GNU `make` 会输出非常有用的信息：

```
$ **make**
make: Building foo.o
cc -c -o foo.o foo.c
Makefile:7: Building bar
```

输出的第一行是在即将执行内建模式规则以生成`foo.o`时产生的。由于没有打印 makefile 或行号信息，我们知道这里使用了内建规则。然后，你会看到内建规则的实际输出（即`cc`命令）。接着是另一条来自`$(warning)`的输出，表明`bar`即将使用 makefile 中第 7 行的规则进行构建。

我们在添加到`SHELL`中的`$(warning)`语句中使用了`$@`，但没有什么能阻止我们使用其他自动变量。例如，在示例 2-6 中，我们使用了`$<`，它保存了构建目标的第一个前提条件，和`$?`，它保存了比目标更新的前提条件列表，并告诉我们为什么要构建该目标。

示例 2-6. 使用`SHELL`技巧

```
OLD_SHELL := $(SHELL)
SHELL = $(warning Building $@$(if $<, (from $<))$(if $?, ($? newer)))$(OLD_SHELL)

.PHONY: all
all: foo.o bar

bar: ; touch $@
```

这里`SHELL`被重新定义为输出三条信息：正在构建的目标的名称（`$@`），第一个前提条件的名称（`$<`，它被包裹在`$(if)`中，以便在没有前提条件时不打印任何内容），以及任何更新的前提条件的名称（`$?`）。

删除`foo.o`并在这个 makefile 上运行`make`，现在显示`foo.o`是从`foo.c`构建的，因为`foo.c`比`foo.o`更新（因为`foo.o`缺失了）：

```
$ **make**
make: Building foo.o (from foo.c) (foo.c newer)
cc -c -o foo.o foo.c
Makefile:7: Building bar
```

没有什么能阻止我们将这个`$(warning)`技巧与`-x`结合使用，以显示哪些规则被执行了以及执行了哪些命令，如示例 2-7 技巧与-x 结合")中所示。

示例 2-7. 将$(`warning`)技巧与`-x`结合

```
OLD_SHELL := $(SHELL)
SHELL = $(warning Building $@$(if $<, (from $<))$(if $?, ($? newer)))$(OLD_SHELL) -x

.PHONY: all
all: foo.o bar

bar: ; @touch $@
```

下面是示例 2-7 技巧与-x 结合")中 makefile 的完整输出。

```
$ **make**
make: Building foo.o (from foo.c) (foo.c newer)
cc -c -o foo.o foo.c
+ cc -c -o foo.o foo.c
Makefile:7: Building bar
+ touch bar
```

这假设在运行`make`时，`foo.c`比`foo.o`更新（或者`foo.o`缺失）。

## GNU `make` 4.0 跟踪

GNU `make` 4.0 增加了一个`--trace`命令行选项，可以用来跟踪规则执行。它提供的输出类似于示例 2-7 技巧与-x 结合")。下面是示例 2-6，去掉`SHELL`修改后的跟踪输出，使用 GNU `make` 4.0 时的结果：

```
$ **make --trace**
<builtin>: update target 'foo.o' due to: foo.c
cc    -c -o foo.o foo.c
Makefile:4: target 'bar' does not exist
touch bar
```

当使用`--trace`选项调用时，GNU `make` 4.0 会覆盖`@`修饰符（在前面的示例中用于抑制`touch bar`），就像`-n`和`--just-print`标志一样。

# Makefile 断言

大多数编程语言都有断言：当它们断言的值为真时不会执行任何操作，但如果不为真则会导致致命错误。它们通常作为运行时调试辅助工具，用于捕捉非常特殊的情况。C 语言中的典型断言可能看起来像`assert( foo != bar )`，如果`foo`和`bar`相同，则会导致致命错误。

不幸的是，GNU `make`没有任何内置的断言功能。但它们很容易通过现有的函数来创建，甚至在 GNU Make 标准库（GMSL）中定义了方便的断言函数。

GMSL 项目（在第六章中有介绍）提供了两个断言函数：`assert`和`assert_exists`。

## assert

如果`assert`的第一个参数为 false，它将输出一个致命错误。与`make`的`$(if)`函数一样，GMSL 将任何非空字符串视为 true，空字符串视为 false。因此，如果`assert`的参数是空字符串，断言将导致致命错误；`assert`的第二个参数将作为错误的一部分被打印出来。例如，这个 makefile 会立刻中断，因为`$(FOO)`和`$(BAR)`相同：

```
include gmsl

FOO := foo
BAR := foo

$(call assert,$(call sne,$(FOO),$(BAR)),FOO and BAR should not be equal)
```

因为`assert`不是一个内置函数——它是在 GMSL 的 makefile 中用户定义的——所以我们必须使用`$(call)`。

我们得到以下信息：

```
Makefile:5: *** GNU Make Standard Library: Assertion failure: FOO and BAR should
not be equal. Stop.
```

断言使用了另一个 GMSL 函数，`sne`，它比较两个字符串，如果它们不相等则返回 true，否则返回 false。

因为 true 仅意味着*非空字符串*，所以很容易断言一个变量已定义：

```
include gmsl

$(call assert,$(FOO),FOO is not defined)
```

你可以使用这个断言，例如检查用户是否设置了所有必要的命令行变量；如果`FOO`是 makefile 正确运行所必需的，而用户忘记在命令行中设置它，断言将会导致错误。

你甚至可以使用断言来强制某些命令行标志不被使用。这里有一个例子，防止用户设置`-i`，即忽略错误标志：

```
include gmsl

$(foreach o,$(MAKEFLAGS),$(call assert,$(call sne,-i,$o),You can't use the -i option))

ifneq ($(patsubst -%,-,$(firstword $(MAKEFLAGS))),-)
$(call assert,$(call sne,$(patsubst i%,i,$(patsubst %i,i,$(firstword \
$(MAKEFLAGS)))),i),You can't use the -i option)
endif
```

这个例子比前两个更复杂，因为`make`可以以两种方式将`-i`标志存储在`MAKEFLAGS`中：作为常见形式的`-i`标志，或作为`MAKEFLAGS`中第一个单词的字符块。也就是说，设置命令行标志`-i -k`会导致`MAKEFLAGS`的值为`ki`。所以循环中的第一个`assert`查找`-i`，第二个`assert`则在`MAKEFLAGS`的第一个单词中查找`i`。

## assert_exists

因为构建的成功依赖于所有必要文件的存在，GMSL 提供了一个专门设计的断言，用于在文件缺失时发出警告。`assert_exists`函数有一个参数：必须存在的文件名。例如，为了在 makefile 运行任何命令之前检查文件*foo.txt*是否存在，你可以在开头添加一个断言：

```
include gmsl

$(call assert_exists,foo.txt)
```

如果文件不存在，构建将停止：

```
Makefile:3: *** GNU Make Standard Library: Assertion failure: file 'foo.txt'
missing. Stop.
```

断言停止了构建，并且在 makefile 中显示了断言所在的行号——在本例中是第 3 行。

## assert_target_directory

在构建实际项目中的 makefile 时，一个常见问题是你必须在构建过程中或之前创建目录层次结构。你可以通过创建一个特殊的 `assert_target_directory` 变量，确保在每个规则执行之前每个目录都已存在，正如 示例 2-8 所示。

示例 2-8. 创建 `assert_target_directory` 变量

```
include gmsl

assert_target_directory = $(call assert,$(wildcard $(dir $@)),Target directory $(dir $@) missing)

foo/all: ; @$(call assert_target_directory)echo $@
```

通过在每个规则或模式规则的配方开始处插入 `$(call assert_target_directory)`，`make` 会自动检查目标文件将被写入的目录是否存在。例如，如果 *foo/* 不存在，那么 示例 2-8 中的 makefile 会出现以下错误：

```
Makefile:6: *** GNU Make Standard Library: Assertion failure: Target directory
foo/ missing. Stop.
```

错误信息会给出出错的 makefile 名称和出错的行号，方便迅速找到问题所在。

最后一个技巧是，可以通过两行修改使 makefile 在检查每个规则时检查缺失的目录。与其在每个规则中添加 `$(call assert_target_directory)`，不如重新定义 `SHELL` 变量，使其包含 `$(call assert_target_directory)`。这样做会稍微影响性能，但对于追踪某个深层嵌套 makefile 中缺失的目录非常有用：

```
include gmsl

assert_target_directory = $(call assert,$(wildcard $(dir $@)),Target directory $(dir $@) missing)

OLD_SHELL := $(SHELL)
SHELL = $(call assert_target_directory)$(OLD_SHELL)

foo/all: ; @echo $@
```

`make` 扩展 `SHELL` 的值，从而对每个执行的规则都调用 `assert_target_directory`。这一简单的修改意味着每个规则都会检查目标目录是否存在。

新的 `SHELL` 值包括对 `assert_target_directory` 的调用，该函数始终返回空字符串，后面跟着存储在 `OLD_SHELL` 中的旧 `SHELL` 值。注意 `OLD_SHELL` 是使用 `:=` 定义的，以确保 `SHELL` 不会引用自身——`OLD_SHELL` 包含 `SHELL` 在运行时的值，可以安全地用来重新定义 `SHELL`。如果 `OLD_SHELL` 使用 `=` 定义，`make` 会因为循环引用而失败：`SHELL` 会引用 `OLD_SHELL`，而 `OLD_SHELL` 又会引用 `SHELL`，如此反复。

`assert_target_directory` 函数通过调用内建的 `$(wildcard)` 函数，并传入当前目标应写入的目录名来工作。`$(wildcard)` 函数简单地检查目录是否存在，如果存在，则返回目录名；如果目录缺失，则返回空字符串。目标由自动变量 `$@` 定义，目录部分则通过 `$(dir)` 提取。

# 一个交互式 GNU make 调试器

尽管 GNU `make` 很受欢迎，但调试功能少之又少。GNU `make` 有一个 `-d` 选项，能够输出关于构建的广泛调试信息（但不一定有用），还有一个 `-p` 选项，会打印出 GNU `make` 的内部规则和变量数据库。本节展示了如何仅使用 GNU `make` 的内部函数和 shell `read` 命令来构建一个交互式调试器。

调试器有断点功能，能够在断点被触发时输出有关规则的信息，并允许交互式查询变量值和定义。

## 调试器实战

在你了解调试器如何工作之前，先来看一下如何使用它。调试器和这些示例都假设你正在使用 GNU `make` 3.80 或更高版本。示例 2-9 展示了一个示例 makefile，它从先决条件 `foo` 和 `bar` 构建 `all`。

示例 2-9. 使用 `__BREAKPOINT` 变量设置断点

```
MYVAR1 = hello
MYVAR2 = $(MYVAR1) everyone
all: MYVAR3 = $(MYVAR2)
all: foo bar
→ $(__BREAKPOINT)
→ @echo Finally making $@
foo bar:
→ @echo Building $@
```

为了演示如何使用调试器，断点被设置在 `all` 规则中，通过在规则的配方开始处插入一行仅包含变量 `__BREAKPOINT`。当规则执行时，`$(__BREAKPOINT)` 会展开，导致调试器中断执行，并在 `all` 规则即将运行时提示，如 示例 2-9 所示。

当执行这个 makefile 且没有名为 `all`、`foo` 或 `bar` 的文件时，会发生以下情况：

```
$ **make**
Building foo
Building bar
Makefile:51: GNU Make Debugger Break
Makefile:51: - Building 'all' from 'foo bar'
Makefile:51: - First prerequisite is 'foo'
Makefile:51: - Prerequisites 'foo bar' are newer than 'all'
1>
```

首先，你会看到执行 `foo` 和 `bar` 规则时的输出（即 `Building foo` 和 `Building bar` 行），接着进入调试器。调试器的断点会显示触发断点的行和所在的 makefile。在这个例子中，断点发生在 makefile 的第 51 行。（它是第 51 行，因为在 示例 2-9 中没有显示的是所有实际使调试器工作的 GNU `make` 变量。）

调试器还会输出正在构建的规则的信息。在这里，你可以看到 `all` 是从 `foo` 和 `bar` 构建的，第一个先决条件是 `foo`。这一点很重要，因为第一个先决条件会存储在 GNU `make` 的 `$<` 自动变量中。（`$<` 通常作为编译时的源代码文件名使用。）调试器还会显示为什么 `all` 规则会运行：因为 `foo` 和 `bar` 都比 `all` 更新（因为它们刚刚由各自的规则构建）。

最后，调试器会提示 `1>` 输入命令。在自动继续执行 makefile 之前，调试器会接受 32 个命令。数字 `1` 表示这是第一个命令；一旦达到 `32>`，调试器会自动继续执行。首先，你可以通过输入 `h` 来请求帮助：

```
1< **h**
Makefile:51: c       continue
Makefile:51: q       quit
Makefile:51: v VAR   print value of $(VAR)
Makefile:51: o VAR   print origin of $(VAR)
Makefile:51: d VAR   print definition of $(VAR)
2>
```

调试器提供了两种停止调试的方法：输入 `c` 会继续正常执行 makefile；输入 `q` 会退出 `make`。三个调试器命令 `v`、`o` 和 `d` 允许用户通过询问变量的值、来源（它是在哪里定义的）或定义来查询 GNU `make` 变量。例如，在 示例 2-9 中，makefile 包含两个变量——`MYVAR1` 和 `MYVAR2`——以及一个特定于 `all` 规则的变量：`MYVAR3`。第一步是向调试器询问这些变量的值：

```
2> **v MYVAR1**
Makefile:55: MYVAR1 has value 'hello'
3> **v MYVAR2**
Makefile:55: MYVAR2 has value 'hello everyone'
4> **v MYVAR3**
Makefile:55: MYVAR3 has value 'hello everyone'
5>
```

如果不清楚 `MYVAR3` 是如何获得其值的，你可以向调试器询问它的定义：

```
5> **d MYVAR3**
Makefile:55: MYVAR3 is defined as '$(MYVAR2)'
6>
```

这显示了 `MYVAR3` 被定义为 `$(MYVAR2)`。接下来的显而易见的步骤是找出 `MYVAR2` 是如何定义的（以及 `MYVAR1`）：

```
6> **d MYVAR2**
Makefile:55: MYVAR2 is defined as '$(MYVAR1) everyone' 7
> **d MYVAR1**
Makefile:55: MYVAR1 is defined as 'hello'
8>
```

如果不清楚 `MYVAR1` 的值是如何获得的，`o` 命令将显示它的来源：

```
8> **o MYVAR1**
Makefile:55: MYVAR1 came from file
9>
```

这意味着 `MYVAR1` 在一个 makefile 中被定义。相比之下：

```
$ **make MYVAR1=Hello**
1> **v MYVAR1**
Makefile:55: MYVAR1 has value 'Hello'
2> **o MYVAR1**
Makefile:55: MYVAR1 came from command line
3>
```

如果用户在命令行上覆盖了 `MYVAR1` 的值（例如，运行 `make MYVAR1=Hello`），则 `o` 命令会反映这一点。

## 模式中的断点

除了在正常规则中设置断点，你还可以在模式中设置断点。每次使用该模式规则时，断点都会被触发。例如：

```
all: foo.x bar.x

%.x: FOO = foo
%.x: %.y
→ $(__BREAKPOINT)
→ @echo Building $@ from $<...

foo.y:
bar.y:
```

在这里，`all` 是由 `foo.x` 和 `bar.x` 构建的，这需要通过 `%.x: %.y` 规则从 `foo.y` 和 `bar.y` 构建它们。一个断点被插入到模式规则中，调试器会中断两次：一次是 `foo.x`，一次是 `bar.x`：

```
$ **make**
Makefile:66: GNU Make Debugger Break
Makefile:66: - Building 'foo.x' from 'foo.y'
Makefile:66: - First prerequisite is 'foo.y'
Makefile:66: - Prerequisites 'foo.y' are newer than 'foo.x'
1> **c**
Building foo.x from foo.y...
Makefile:66: GNU Make Debugger Break
Makefile:66: - Building 'bar.x' from 'bar.y'
Makefile:66: - First prerequisite is 'bar.y'
Makefile:66: - Prerequisites 'bar.y' are newer than 'bar.x'
1> **c**
Building bar.x from bar.y...
```

即使是特定于模式的变量也能正常工作：

```
$ **make**
Makefile:67: GNU Make Debugger Break
Makefile:67: - Building 'foo.x' from 'foo.y'
Makefile:67: - First prerequisite is 'foo.y'
Makefile:67: - Prerequisites 'foo.y' are newer than 'foo.x'
1> **v FOO**
Makefile:67: FOO has value 'foo'
2>
```

`%.x` 具有一个特定于模式的变量 `FOO`，其值为 `foo`；调试器的 `v` 命令可以在模式规则的断点处访问它。

## Makefile 中的断点

此外，如果需要，你也可以直接在 makefile 中插入断点。makefile 的解析将在断点处暂停，以便你检查 makefile 中当前变量的状态。例如，在每次定义 `FOO` 后插入一个断点，你可以看到它的值如何变化：

```
FOO = foo
$(__BREAKPOINT)
FOO = bar
$(__BREAKPOINT)
```

下面是一个示例运行：

```
$ **make**
Makefile:76: GNU Make Debugger Break
1> **v FOO**
Makefile:76: FOO has value 'foo'
2> **c**
Makefile:78: GNU Make Debugger Break
1> **v FOO**
Makefile:78: FOO has value 'bar'
2>
```

这两个单独的断点会被激活（每次设置 `FOO` 后）。使用调试器的 `v` 命令可以显示在每个断点处 `FOO` 的值如何变化。

## 调试器内部实现

调试器调用了在 GMSL 中定义的函数（你可以在 第六章 中了解更多关于 GMSL 的信息）。调试器的第一行包括 GMSL 函数：

```
include gmsl

__LOOP := 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32
```

调试器使用`__PROMPT`变量来输出`n>`并读取一个带有单一参数的命令。`__PROMPT`使用`read` shell 命令将命令和参数读取到 shell 变量`$CMD`和`$ARG`中，然后返回一个包含两个元素的列表：第一个元素是命令，第二个元素是参数。展开`__PROMPT`会提示并返回一个单一的命令和参数对：

```
__PROMPT = $(shell read -p "$(__HISTORY)> " CMD ARG ; echo $$CMD $$ARG)
```

你使用`__BREAK`变量来获取并处理单一命令。首先，它将`__PROMPT`的结果存储在`__INPUT`中，然后调用`__DEBUG`函数（处理调试器命令），并传入两个参数：由`__PROMPT`在`__INPUT`中返回的命令及其参数。

```
__BREAK = $(eval __INPUT := $(__PROMPT))       \
          $(call __DEBUG,                      \
              $(word 1,$(__INPUT)),            \
              $(word 2,$(__INPUT)))
```

`__DEBUG`函数处理调试器的核心部分。`__DEBUG`接受一个单字符命令作为第一个参数`$1`，以及命令的可选参数`$2`。`$1`存储在变量`__c`中，`$2`存储在`__a`中。然后，`__DEBUG`检查`__c`是否为支持的调试器命令之一（`c`、`q`、`v`、`d`、`o`或`h`）；如果不是，`$(warning)`将输出错误消息。

`__DEBUG`由一组嵌套的`$(if)`语句组成，使用 GMSL 的`seq`函数来判断`__c`是否为有效的调试器命令。如果是，`$(if)`的第一个参数将被展开；如果不是，接下来的`$(if)`将被检查。例如，`v`命令（用于输出变量的值）是这样处理的：

```
$(if $(call seq,$(__c),v),$(warning $(__a) has value '$($(__a))'), ... next if ... )
```

如果`__c`命令是`v`，则使用`$(warning)`输出由`__a`命名的变量的值（`$($(__a))`输出存储在`__a`中的变量名对应的值）。

当`__DEBUG`完成时，它返回`$(true)`或`$(false)`（空字符串）。`$(true)`表示调试器应停止提示命令并继续执行（`q`命令通过调用 GNU `make`的`$(error)`函数来引发致命错误，从而停止`make`）：

```
__DEBUG = $(eval __c = $(strip $1))                      \
          $(eval __a = $(strip $2))                      \
          $(if $(call seq,$(__c),c),                     \
           $(true),                                      \
           $(if $(call seq,$(__c),q),                    \
            $(error Debugger terminated build),          \
            $(if $(call seq,$(__c),v),                   \
             $(warning $(__a) has value '$($(__a))'),    \
             $(if $(call seq,$(__c),d),                  \
     $(warning $(__a) is defined as '$(value $(__a))'),  \
               $(if $(call seq,$(__c),o),                \
     $(warning $(__a) came from $(origin $(__a))),       \
                $(if $(call seq,$(__c),h),               \
                 $(warning c       continue)             \
                 $(warning q       quit)                 \
             $(warning v VAR print value of $$(VAR))     \
             $(warning o VAR print origin of $$(VAR))    \
        $(warning d VAR print definition of $$(VAR)),    \
        $(warning Unknown command '$(__c)')))))))
```

最后，我们来看看`__BREAKPOINT`的定义（这是我们在示例 2-9 中使用的断点变量）。它首先输出一个包含信息的横幅（稍后你将看到`__BANNER`的作用）；然后它通过调用`__BREAK`来循环询问命令。循环会在`__LOOP`中没有更多项时结束（这里定义了 32 个命令的限制），或者当调用`__BREAK`返回`$(true)`时结束：

```
__BREAKPOINT = $(__BANNER)                              \
               $(eval __TERMINATE := $(false))          \
               $(foreach __HISTORY,                     \
                   $(__LOOP),                           \
                   $(if $(__TERMINATE),,                \
                      $(eval __TERMINATE := $(__BREAK))))
```

`__BANNER`显示调试器已在断点处停止，并通过检查 GNU `make`自动变量，能够提供有关当前正在构建的规则的信息：

```
__BANNER = $(warning GNU Make Debugger Break)           \
           $(if $^,                                     \
              $(warning - Building '$@' from '$^'),     \
              $(warning - Building '$@'))               \
       $(if $<,$(warning - First prerequisite is '$<')) \
           $(if $%,$(warning - Archive target is '$%')) \
           $(if $?,$(warning - Prerequisites '$?' are newer than '$@'))
```

这是完整的调试器代码：

```
__LOOP := 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32

__PROMPT = $(shell read -p "$(__HISTORY)> " CMD ARG ; echo $$CMD $$ARG)

__DEBUG = $(eval __c = $(strip $1))                     \
          $(eval __a = $(strip $2))                     \
          $(if $(call seq,$(__c),c),                    \
           $(true),                                     \
           $(if $(call seq,$(__c),q),                   \
            $(error Debugger terminated build),         \
            $(if $(call seq,$(__c),v),                  \
             $(warning $(__a) has value '$($(__a))'),   \
             $(if $(call seq,$(__c),d),                 \
     $(warning $(__a) is defined as '$(value $(__a))'), \
               $(if $(call seq,$(__c),o),               \
     $(warning $(__a) came from $(origin $(__a))),      \
               $(if $(call seq,$(__c),h),               \
                $(warning c       continue)             \
                $(warning q       quit)                 \
             $(warning v VAR print value of $$(VAR))    \
             $(warning o VAR print origin of $$(VAR))   \
        $(warning d VAR print definition of $$(VAR)),   \
        $(warning Unknown command '$(__c)')))))))

__BREAK = $(eval __INPUT := $(__PROMPT))                \
          $(call __DEBUG,                               \
              $(word 1,$(__INPUT)),                     \
              $(word 2,$(__INPUT)))

__BANNER = $(warning GNU Make Debugger Break)           \
           $(if $^,                                     \
              $(warning - Building '$@' from '$^'),     \
              $(warning - Building '$@'))               \
       $(if $<,$(warning - First prerequisite is '$<')) \
           $(if $%,$(warning - Archive target is '$%')) \
           $(if $?,$(warning - Prerequisites '$?' are newer than '$@'))
__BREAKPOINT = $(__BANNER)                              \
               $(eval __TERMINATE := $(false))          \
               $(foreach __HISTORY,                     \
               $(__LOOP),                               \
               $(if $(__TERMINATE),,                    \
                  $(eval __TERMINATE := $(__BREAK))))
```

要查看最新版本，请访问 GNU `make` 调试器开源项目，地址是*[`gmd.sf.net/`](http://gmd.sf.net/)*。

# GNU make 调试器中的动态断点

前一节展示了如何完全使用 GNU `make` 编写一个调试器。但它只有静态（硬编码）断点。本节展示了如何通过添加动态断点来增强调试器。这使得可以根据文件的名称（在 GNU `make` 语言中，称为 *目标*）设置和移除断点，而这个目标是 makefile 将要构建的。

不再需要在 makefile 中插入 `$(__BREAKPOINT)` 字符串。输入一个简单的设置断点命令就能产生相同的效果。另一个按键操作则列出当前生效的所有断点。

本节展示了新断点的使用以及如何编写代码。新代码完全使用 GNU `make` 的变量语言编写，并使用 GMSL 设置函数（详见第六章）来维护当前断点列表。

要激活断点，需要一些 GNU `make` 的魔法，但首先让我们看一个例子。

## 动态断点的实际应用

在了解调试器如何工作之前，先看看如何使用它。调试器和这些示例都假设你使用的是 GNU `make` 3.80 或更高版本。

这是一个示例 makefile，它从前置条件 `foo` 和 `bar` 构建 `all`。

```
include gmd

MYVAR1 = hello
MYVAR2 = $(MYVAR1) everyone

all: MYVAR3 = $(MYVAR2)
all: foo bar
all: ; @echo Finally making $@
foo bar: ; @echo Building $@

$(__BREAKPOINT)
```

为了说明调试器的使用，通过在 makefile 末尾插入一行仅包含变量 `$(__BREAKPOINT)` 的代码来设置断点。`$(__BREAKPOINT)` 会在 makefile 解析完毕时展开，导致调试器在任何规则执行前中断执行并提示输入。（调试器通过在开始处的 `include gmd` 命令引入。你可以从 GMD 网站获取 GMD 文件，地址是 *[`gmd.sf.net/`](http://gmd.sf.net/)*；所有代码都是开源的。）

当执行这个 makefile 且没有名为 `all`、`foo` 或 `bar` 的现有文件时，会发生以下情况：

```
$ **make**
Makefile:11: GNU Make Debugger Break
1> **h**
Makefile:11: c:     continue
Makefile:11: q:     quit
Makefile:11: v VAR: print value of $(VAR)
Makefile:11: o VAR: print origin of $(VAR)
Makefile:11: d VAR: print definition of $(VAR)
Makefile:11: b TAR: set a breakpoint on target TAR
Makefile:11: r TAR: unset breakpoint on target TAR
Makefile:11: l: list all target breakpoints
2>
```

调试器会立即中断并等待输入。首先要做的是输入 `h` 查看帮助文本以及三个新命令：`b`（设置断点），`r`（移除断点），和 `l`（列出当前断点）。

然后在 makefile 中设置两个断点：一个是当 `foo` 被构建时，另一个是为 `all` 设置的。（如果你回顾一下调试器实战，你会看到你也可以通过修改 makefile 来实现这一点，但这些新断点可以在运行时动态设置。）

设置断点后，使用 `l` 命令来验证它们是否已设置：

```
2> **b foo**
Makefile:11: Breakpoint set on `foo'
3> **b all**
Makefile:11: Breakpoint set on `all'
4> **l**
Makefile:11: Current target breakpoints: `all' `foo'
5>
```

通过输入 `c` 继续执行时，`foo` 断点会立即触发。`foo` 是 makefile 将构建的第一个目标（接着是 `bar`，最后是 `all`）。该断点表明 `foo` 的规则位于第 9 行：

```
5> **c**
Makefile:9: GNU Make Debugger Break
Makefile:9: - Building 'foo'
1>
```

继续执行时，首先会显示生成 `bar` 时的输出，然后触发 `all` 断点。

```
1> **c**
Building foo
Building bar
Makefile:7: GNU Make Debugger Break
Makefile:7: - Building 'all' from 'foo bar'
Makefile:7: - First prerequisite is 'foo'
Makefile:7: - Prerequisites 'foo bar' are newer than 'all'
1>
```

`all` 断点打印出的信息比 `foo` 断点多得多，因为 `all` 具有前置条件。

## 简单部分

为了将断点函数添加到 GNU `make` 调试器中，首先修改了处理键盘输入的调试器代码，以识别`b`、`r`和`l`命令，并调用用户定义的 GNU `make` 函数`__BP_SET`、`__BP_UNSET`和`__BP_LIST`。

定义断点的目标只是一个 GMSL 目标名称集合。最初，没有断点，因此该集合（称为`__BREAKPOINTS`）是空的：

```
__BREAKPOINTS := $(empty_set)
```

设置和删除断点只需调用 GMSL 函数`set_insert`和`set_remove`来向`__BREAKPOINTS`中添加或移除元素：

```
__BP_SET = $(eval __BREAKPOINTS := $(call set_insert,$1,$(__BREAKPOINTS))) \
          $(warning Breakpoint set on `$1')

__BP_UNSET = $(if $(call set_is_member,$1,$(__BREAKPOINTS)),               \
        $(eval __BREAKPOINTS := $(call set_remove,$1,$(__BREAKPOINTS)))    \
        $(warning Breakpoint on `$1' removed),                             \
        $(warning Breakpoint on `$1' not found))
```

两个函数都使用 GNU `make $(eval)` 函数来更改`__BREAKPOINTS`的值。`$(eval FOO)`将其参数`FOO`当作文本在解析 makefile 时进行求值：这意味着在运行时你可以更改变量值或定义新的规则。

`__BP_UNSET`使用 GMSL 函数`set_is_member`来判断要移除的断点是否已定义，并在用户尝试移除一个不存在的断点时（这可能是由于用户输入错误）输出一个有用的消息。

列出当前断点仅仅是输出存储在`__BREAKPOINTS`中的集合内容。因为该集合只是一个没有重复元素的列表，所以`__BP_LIST`将其值传递给 GNU `make` 函数`$(addprefix)`和`$(addsuffix)`，以便在目标名称周围加上引号：

```
__BP_LIST = $(if $(__BREAKPOINTS),                      \
             $(warning Current target breakpoints:      \
       $(addsuffix ',$(addprefix `,$(__BREAKPOINTS)))), \
             $(warning No target breakpoints set))
```

`__BP_LIST`使用 GNU `make $(if)` 函数来选择：如果有断点，则列出断点；如果`__BREAKPOINTS`集合为空，则显示`No target breakpoints set`。

## 诀窍

要让 GNU `make` 进入调试器，它必须展开`__BREAKPOINT`变量，该变量输出有关断点的信息并提示输入命令。但为了实现这一点，我们需要一种方法来检查每次规则即将运行时定义了哪些断点。如果我们能做到这一点，那么`make`就可以在必要时展开`$(__BREAKPOINT)`，导致`make`在断点处停止。

幸运的是，通过修改内置的`SHELL`变量，可以让`make`展开`__BREAKPOINT`。

每次命令准备在规则内部运行时，`SHELL`变量也会被展开。这使得它非常适合检查断点。以下是 GNU `make` 调试器中实际使用 `SHELL` 处理断点的代码：

```
__BP_OLD_SHELL := $(SHELL)
__BP_NEW_SHELL = $(if $(call seq,$(__BP_FLAG),$@), \
                 $(call $1,),                      \
                 $(__BP_CHECK))$(__BP_OLD_SHELL)
SHELL = $(call __BP_NEW_SHELL,$1)
```

首先，`SHELL`的实际值存储在`__BP_OLD_SHELL`中（请注意，GNU `make :=` 操作符用于捕获`SHELL`的值，而不是定义）。然后，`SHELL`被重新定义为调用`__BP_NEW_SHELL`变量。

`__BP_NEW_SHELL`是执行有趣工作的地方。它的最后部分是`$(__BP_OLD_SHELL)`，即原始`SHELL`变量的值。毕竟，一旦检查完断点，GNU `make`需要使用原始的`shell`来实际运行命令。在此之前，还有一个相当复杂的`$(if)`。集中精力看一下对`$(__BP_CHECK)`的调用。这个变量实际上会检查是否应该执行断点。它是这样定义的：

```
__BP_CHECK = $(if $(call set_is_member,$@,        \
              $(__BREAKPOINTS)),                  \
              $(eval __BP_FLAG := $@)             \
              $(eval __IGNORE := $(call SHELL,    \
                                 __BREAKPOINT)))
__BP_FLAG :=
```

`__BP_CHECK`检查当前正在构建的目标（存储在标准 GNU `make`自动变量`$@`中）是否存在于断点列表中。它通过使用 GMSL 函数`set_is_member`来完成这一检查。如果目标存在，它会做两件事：设置一个名为`__BP_FLAG`的内部变量，将其值设为已激活断点的目标，并继续执行`$(call)`某个变量，并通过将结果存储在`__IGNORE`变量中来丢弃它。这么做是为了确保`__BP_CHECK`的返回值始终为空；毕竟它是用于定义`SHELL`的，而`SHELL`最终需要只是要执行的`shell`的名称。

有经验的 GNU `make`用户可能会抓耳挠腮，想弄清楚那个奇怪的语法`$(call SHELL,__BREAKPOINT)`。这时就涉及到一些 GNU `make`的火箭科学。

## 火箭科学

与其写`$(call SHELL,__BREAKPOINT)`，人们更容易写`$(__BREAKPOINT)`来激活断点。但这样并不起作用。

这么做会导致致命的 GNU `make`错误。追溯变量链从`__BP_CHECK`开始，就会发现它已经展开，因为`SHELL`正在展开（因为规则即将运行）。跟进到`__BREAKPOINT`，会有一个令人吃惊的结果：调用`$(shell)`（在 GMD 代码中的加法与减法或者前一节中可以看到），这会导致`SHELL`展开。

危险，威尔·罗宾逊！`SHELL`是通过`SHELL`定义的，这会导致 GNU `make`发现递归并放弃。`$(call SHELL,__BREAKPOINT)`语法让我们可以玩火。每次在 GNU `make`中使用`$(call)`调用一个变量时，用于检查递归的标志会被禁用。因此，执行`$(call SHELL,__BREAKPOINT)`意味着`SHELL`的递归标志被关闭（避免了错误），并且`SHELL`的定义会调用`__BP_NEW_SHELL`并传入一个参数。该参数是词汇`__BREAKPOINT`。`__BP_NEW_SHELL`会检查`__BP_FLAG`是否与`$@`的值相同（通过 GMSL 的`seq`函数进行检查），然后继续执行`$(call)`第一个参数（即`__BREAKPOINT`）；断点被触发，提示符出现。

当执行 `$(shell)` 并且 `SHELL` 被再次展开时，可能看起来会发生可怕的无限递归。两个因素阻止了这种情况：`__BP_FLAG` 与 `$@` 保持一致（因此 `__BP_CHECK` 不会再次被调用），并且这次 `SHELL` 没有参数（`$1` 的值为空），所以 `$(call $1,)` 什么也不做，递归停止。

# `remake` 简介

`remake` 项目（*[`bashdb.sourceforge.net/remake/`](http://bashdb.sourceforge.net/remake/)*) 是一个基于 GNU `make` 的分支，它通过修改 GNU `make` 源代码集成了一个完整的调试器。`remake` 从 GNU `make` 3.82 分支而来，目前版本为 3.82+dbg-0.9。

## 仅打印和跟踪

为了说明 `remake` 的操作，我们使用 示例 2-10，这是一个示例 makefile：

示例 2-10. 一个简单的 makefile 用于说明 remake

```
.PHONY: all
all: foo bar baz

foo: bar
→ @touch $@
bar:
→ @touch $@

baz: bam
→ @touch $@

bam:
→ @touch $@
```

运行标准的 GNU `make -n`（或 `--just-print`）选项来执行这个 makefile 会产生以下输出：

```
$ **make -n**
touch bar
touch foo
touch bam
touch baz
```

但是 `remake` 为每个规则提供了 makefile 和行号信息。这些信息显示了目标（`$@` 的值）和要执行的命令：

```
$ remake -n
##>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Makefile:8: bar
touch bar
##<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
##>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Makefile:5: foo
touch foo
##<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
##>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Makefile:14: bam
touch bam
##<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
##>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Makefile:11: baz
touch baz
##<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
```

当然，你必须运行任何实际的 makefile 来理解其执行过程。`remake` 提供了一个方便的跟踪选项 `-x`，它在运行 makefile 的同时输出有关为何构建目标的信息，并显示执行的命令及其输出：

```
$ **remake -x**
Reading makefiles...
Updating goal targets....
Makefile:2  File `all' does not exist.
  Makefile:4 File `foo' does not exist.
    Makefile:7 File `bar' does not exist.
    Makefile:7 Must remake target `bar'.
##>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Makefile:8: bar
touch bar
##<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
+ touch bar
   Makefile:7 Successfully remade target file `bar'.
 Makefile:4 Must remake target `foo'.
##>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Makefile:5: foo
touch foo
##<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
+ touch foo
 Makefile:4 Successfully remade target file `foo'.
  Makefile:10 File `baz' does not exist.
    Makefile:13 File `bam' does not exist.
   Makefile:13 Must remake target `bam'.
##>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Makefile:14: bam
touch bam
##<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
+ touch bam
   Makefile:13 Successfully remade target file `bam'.
 Makefile:10 Must remake target `baz'.
##>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Makefile:11: baz
touch baz
##<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
+ touch baz

Makefile:10 Successfully remade target file `baz'.
Makefile:2 Must remake target `all'. Is a phony target.
Makefile:2 Successfully remade target file `all'.
```

跟踪选项在发生错误时非常有用。这里是当一个不存在的选项 `-z` 被添加到 `touch` 命令中以构建目标 `bar` 时的输出：

```
$ **remake -x**
Reading makefiles...
Updating goal targets....
Makefile:2 File `all' does not exist.
  Makefile:4 File `foo' does not exist.
    Makefile:7 File `bar' does not exist.
   Makefile:7 Must remake target `bar'.
##>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Makefile:8: bar
touch -z bar
##<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
+ touch -z bar
touch: invalid option -- 'z'
Try `touch --help' for more information.
Makefile:8: *** [bar] Error 1

#0 bar at Makefile:8
#1 foo at Makefile:4
#2 all at Makefile:2
Command-line arguments:
       "-x"
```

输出的最底部是调用栈，显示依赖于 `bar` 构建成功的目标列表，以及 `touch` 生成的错误、执行的实际命令和在 makefile 中的位置。

## 调试

因为 `remake` 包含一个交互式调试器，你可以用它来调试 `touch` 问题。运行 `remake` 时使用 `-X` 选项（大写的 `X` 用于调试器；小写的 `x` 用于跟踪），调试器会在第一个目标构建时断点：

```
$ **remake -X**
GNU Make 3.82+dbg0.9
Built for x86_64-unknown-linux-gnu
Copyright (C) 2010 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Reading makefiles...
Updating makefiles....
Updating goal targets....
  Makefile:2 File `all' does not exist.
-> (Makefile:4)
foo: bar
remake<0>
```

所以，第一个断点是在 makefile 的第 2 行，它显示第一个目标是 `all`（并显示完整的前提条件列表）。输入 `h` 会提供完整的帮助信息：

```
remake<0> **h**
  Command                Short Name  Aliases
  ---------------------- ----------  ---------
  break [TARGET|LINENUM] [all|run|prereq|end]* (b) L
  cd DIR                          (C)
  comment TEXT                    (#)
  continue [TARGET [all|run|prereq|end]*] (c)
  delete breakpoint numbers..     (d)
  down [AMOUNT]                   (D)
  edit                            (e)
  eval STRING                     (E)
  expand STRING                   (x)
  finish [AMOUNT]                 (F)
  frame N                         (f)
  help [COMMAND]                  (h) ?, ??
  info [SUBCOMMAND]               (i)
  list [TARGET|LINE-NUMBER]       (l)
  next [AMOUNT]                   (n)
  print {VARIABLE [attrs...]}     (p)
  pwd                             (P)
  quit [exit-status]              (q) exit, return
  run [ARGS]                      (R) restart
  set OPTION {on|off|toggle}
  set variable VARIABLE VALUE     (=)
  setq VARIABLE VALUE             (")
  shell STRING                    (!) !!
  show [SUBCOMMAND]               (S)
  source FILENAME                 (<)
  skip                            (k)
  step [AMOUNT]                   (s)
  target [TARGET-NAME] [info1 [info2...]] (t)
  up [AMOUNT]                     (u)
  where                           (T) backtrace, bt
  write [TARGET [FILENAME]]       (w)
```

因为 `touch` 问题发生在 `make` 执行的后期（在 `bar` 规则中），所以只需继续执行，通过 `s` 单步调试：

```
remake<1> **s**
    Makefile:4 File `foo' does not exist.
-> (Makefile:7)
bar:
remake<2> **s**
      Makefile:7 File `bar' does not exist.
      Makefile:7 Must remake target `bar'.
Invoking recipe from Makefile:8 to update target `bar'.
##>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
touch -z bar
##<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<
++ (Makefile:7)
bar
remake<3> **s**
touch: invalid option -- 'z'
Try 'touch --help' for more information.
Makefile:7: *** [bar] Error 1

#0 bar at Makefile:7
#1 foo at Makefile:4
#2 all at Makefile:2

***Entering debugger because we encountered a fatal error.
** Exiting the debugger will exit make with exit code 1.
!! (Makefile:7)
bar
remake<4>
```

在调试器中，你可以修复 makefile 中的错误，然后输入`R`来重新启动构建：

```
remake<4> **R**
Changing directory to /home/jgc and restarting...
GNU Make 3.82+dbg0.9
Built for x86_64-unknown-linux-gnu
Copyright (C) 2010 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Reading makefiles...
Updating makefiles....
Updating goal targets....
  Makefile:2 File `all' does not exist.
-> (Makefile:4)
foo: bar
remake<0> **c**
```

现在，一切正常工作。

## 目标、宏值和展开

当在调试器中停止时，可以查询 makefile 中目标的信息，例如变量值（扩展和未扩展的）和命令。例如，在示例 2-10 中，当停在断点时，可以通过使用 `target` 命令查找 `remake` 关于 `all` 目标的所有信息：

```
$ **remake -X**
GNU Make 3.82+dbg0.9
Built for x86_64-unknown-linux-gnu
Copyright (C) 2010 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Reading makefiles...
Updating makefiles....
Updating goal targets....
  /home/jgc/src/thirdparty/remake-3.82+dbg0.9/Makefile:2 File `all' does not exist.
-> (/home/jgc/src/thirdparty/remake-3.82+dbg0.9/Makefile:4)
foo: bar
remake<0> **target all**
all: foo bar baz
# Phony target (prerequisite of .PHONY).
# Implicit rule search has not been done.
# Implicit/static pattern stem: `'
# File does not exist.
# File has not been updated.
# Commands not yet started.
# automatic
# @ := all
# automatic
# % :=
# automatic
# * :=
# automatic
# + := foo bar baz
# automatic
# | :=
# automatic
# < := all
# automatic
# ^ := foo bar baz
# automatic
# ? :=
remake<1>
```

`remake` 显示 `all` 是一个虚拟目标，并打印出将为此规则设置的自动变量信息。没有任何限制可以查询当前目标：

```
remake<1> **target foo**
foo: bar
#  Implicit rule search has not been done.
#  Implicit/static pattern stem: `'
#  File does not exist.
#  File has not been updated.
#  Commands not yet started.
#  automatic
# @ := foo
# automatic
# % :=
# automatic
# * :=
# automatic
# + := bar
# automatic
# | :=
# automatic
# < := bar
# automatic
# ^ := bar
# automatic
# ? :=
#  commands to execute (from `Makefile', line 5):
    @touch $@

remake<2>
```

因为目标 `foo` 有命令，它们会列在底部（以及在哪里找到它们的 makefile）。要查看命令的扩展形式，请使用 `target` 命令的 `expand` 修饰符：

```
remake<2> **target foo expand**
foo:
#  commands to execute (from `Makefile', line 5):
   @touch foo

remake<3>
```

要获取关于变量的信息，我们使用方便的 `print` 和 `expand` 命令：`print` 给出变量的定义，而 `expand` 给出扩展后的值。以下是如何查找内置的 `COMPILE.c` 变量的定义（它包含用于编译 `.c` 文件的命令）：

```
remake<4> **print COMPILE.c**
(origin default) COMPILE.c = $(CC) $(CFLAGS) $(CPPFLAGS) $(TARGET_ARCH) -c
```

要查看扩展后的值，`expand`它：

```
remake<7> **expand COMPILE.c**
(origin default) COMPILE.c := cc -c
```

`remake` 也可以使用 `set`（它扩展一个字符串并将变量设置为该值）和 `setq`（它将变量设置为一个未扩展的字符串）来设置变量的值。例如，将 `CC` 从 `cc` 改为 `gcc` 会改变 `make` 使用的 C 编译器：

```
remake<7> **expand COMPILE.c**
(origin default) COMPILE.c := cc    -c
remake<8> **print CC**
(origin default) CC = cc
remake<9> **setq CC gcc**
Variable CC now has value 'gcc'
remake<10> **print CC**
(origin debugger) CC = gcc
remake<11> **expand COMPILE.c**
(origin default) COMPILE.c := gcc   -c
remake<12>
```

`remake` 是一个非常有用的工具，可以添加到你的 `make` 工具包中。你不需要每天使用它，但当你遇到棘手问题时，如果你没有使用 GNU `make` 4.0 中新增的任何特性，将 `make` 切换到 `remake` 会毫不费力。
