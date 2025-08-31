# 第一章 基础回顾

本章涵盖的内容可能被视为基本的 GNU `make`知识，但我们将重点讲解常见的误解功能，并澄清一些 GNU `make`中的混淆部分。它还涵盖了 GNU `make`版本 3.79.1、3.81、3.82 和 4.0 之间的差异。如果你使用的是 3.79.1 之前的版本，建议你升级。

本章绝不是官方 GNU `make`手册（自由软件基金会，2004 年）的替代品。我强烈推荐拥有一本该手册的副本。你也可以在* [`www.gnu.org/make/manual`](http://www.gnu.org/make/manual) *找到手册。

# 将环境变量导入 GNU `make`

在 GNU `make`启动时，任何在环境中设置的变量都将在 makefile 内作为 GNU `make`变量可用。例如，考虑以下简单的 makefile：

```
$(info $(FOO))
```

如果在运行 GNU `make`时，环境变量`FOO`被设置为`foo`，那么这个 makefile 将输出`foo`，从而验证`FOO`确实在 makefile 内被设置为`foo`。你可以通过使用 GNU `make`的`$(origin)`函数来发现`FOO`是从哪里得到这个值的。尝试将下面的内容添加到 makefile 中（新部分用**加粗**表示）：

```
$(info $(FOO) **$(origin FOO)**)
```

如果变量`FOO`在环境中定义，并自动导入到 GNU `make`中，`$(origin FOO)`将返回`environment`。当你运行 makefile 时，它应该输出`foo environment`。

可以在 makefile 中覆盖从环境导入的变量。只需设置其值：

```
**FOO=bar**
$(info $(FOO) $(origin FOO))
```

这将输出`bar file`。请注意，`$(origin FOO)`的值从`environment`变为`file`，表明该变量的值是在 makefile 内设置的。

可以通过在 GNU `make`的命令行中指定`-e`（或`--environment-overrides`）选项，来防止 makefile 中的定义覆盖环境中的值。将`FOO`设置为`foo`并加上`-e`命令行选项来运行上述 makefile 时，会输出`foo environment override`。请注意，在这里`FOO`的值来自环境（`foo`），并且`$(origin FOO)`的输出已经变为`environment override`，告知我们该变量来自环境，尽管它在 makefile 中被重新定义。只有当变量定义被真正覆盖时，`override`这个词才会出现；如果变量在环境中定义且没有在 makefile 中重新定义，`$(origin)`函数将只返回`environment`（没有`override`）。

如果你关心的只是变量是否从环境中获取了值，那么使用`$(firstword $(origin VAR))`始终能保证返回字符串`environment`，如果变量`VAR`的值来自环境，无论是否指定`-e`选项。

假设你完全希望保证变量`FOO`的值来自 makefile，而不是来自环境。你可以使用`override`指令来做到这一点：

```
**override** FOO=bar
$(info $(FOO) $(origin FOO))
```

这将输出 `bar override`，无论环境中 `FOO` 的值是多少，或者你是否指定了 `-e` 命令行选项。注意，`$(origin)` 会告诉你这是一个通过返回 `override` 的覆盖。

设置变量的另一种方法是通过在 GNU `make` 的命令行中设置它。例如，将你的 makefile 恢复为以下内容：

```
FOO=bar
$(info $(FOO) $(origin FOO))
```

在命令行上运行 `FOO=foo make -e FOO=fooey` 将输出 `fooey command line`。此时 `$(origin FOO)` 返回了 `command line`。现在尝试将覆盖命令重新添加到 makefile 中：

```
**override**
FOO=bar $(info $(FOO) $(origin FOO))
```

如果你在命令行上运行相同的命令（`FOO=foo make -e FOO=fooey`），它现在输出 `bar override`。

迷茫了吗？有一个简单的规则可以帮助你理清楚这一切：`override` 指令优先于命令行，命令行优先于环境覆盖（`-e` 选项），环境覆盖优先于 makefile 中定义的变量，而 makefile 中定义的变量又优先于原始环境。或者，你总是可以使用 `$(origin)` 来找出发生了什么。

# 从外部设置变量

在 makefile 中设置可以通过命令行指定的选项是很常见的。例如，你可能希望改变正在执行的构建类型，或者指定一个目标架构，甚至是在 makefile 外部指定。

最常见的用例之一是一个调试选项，用于指定构建时是否应该创建可调试或发布代码。处理此问题的一种简单方法是使用一个名为 `BUILD_DEBUG` 的 makefile 变量，它在 makefile 中设置为 `yes`，并在构建发布版本时通过命令行覆盖。例如，makefile 可能在开始的地方有这一行：`BUILD_DEBUG := yes`。然后，`BUILD_DEBUG` 变量会在 makefile 的其他地方使用，以决定如何设置编译器的调试选项。因为在 makefile 中设置了 `BUILD_DEBUG := yes`，默认情况下会进行调试构建。然后，在发布时，可以从命令行覆盖此默认值：

```
$ **make BUILD_DEBUG=no**
```

接近发布时，可能会有冲动在 shell 启动脚本中（例如 `.cshrc` 或 `.bashrc`）将 `BUILD_DEBUG` 设置为 `no`，这样所有的构建都是发布版本，而不是调试版本。不幸的是，这种方法不起作用，因为 GNU `make` 会从环境中继承变量，而 makefile 中的变量会覆盖环境变量。

考虑这个简单的 makefile，它打印出 `BUILD_DEBUG` 的值，而该值在 makefile 开头被设置为 `yes`：

```
BUILD_DEBUG := yes
.PHONY: all
all: ; @echo BUILD_DEBUG is $(BUILD_DEBUG)
```

### 注意

*在这个例子中，`all` 目标相关的命令通过使用分号与目标名称放在同一行。另一种方法是：*

```
BUILD_DEBUG := yes
.PHONY: all
all:
→ @echo BUILD_DEBUG is $(BUILD_DEBUG)
```

*但这需要使用制表符来开始命令。当命令可以放在一行时，使用 GNU `make` 提供的分号格式会更清晰。*

现在尝试运行 makefile 三次：一次不设置选项，一次在 GNU `make` 的命令行上设置 `BUILD_DEBUG`，还有一次在环境中设置 `BUILD_DEBUG`：

```
$ **make**
BUILD_DEBUG is yes
$ **make BUILD_DEBUG=no**
BUILD_DEBUG is no
$ **export BUILD_DEBUG=no**
$ **make**
BUILD_DEBUG is yes
```

最后一行显示了在 makefile 中定义的变量覆盖环境中的值。但请注意，如果`BUILD_DEBUG`在 makefile 中根本没有定义，它将自动从环境中继承并导入到 makefile 中。

使用 GNU `make`工具可以通过`-e`选项来解决 makefile 中定义覆盖导入的环境变量的问题，该选项使环境变量优先。但这会影响*所有*变量。

```
$ **export BUILD_DEBUG=no**
$ **make**
BUILD_DEBUG is yes
$ **make -e**
BUILD_DEBUG is no
$ **make -e BUILD_DEBUG=maybe**
BUILD_DEBUG is maybe
```

需要记住的规则是：*命令行优先于 makefile，makefile 优先于环境*。在命令行中定义的变量优先于在 makefile 中定义的同名变量，而 makefile 中的变量又优先于环境中定义的同名变量。

可以有一个默认设置为`yes`的`BUILD_DEBUG`变量，它可以*在命令行或环境中*被覆盖。GNU `make`提供了两种方法来实现这一点，这两种方法都依赖于检查变量是否已经定义。

这里有一种方法。将原始 makefile 中的`BUILD_DEBUG`设置替换为：

```
ifndef BUILD_DEBUG
BUILD_DEBUG := yes
endif
```

如果`BUILD_DEBUG`尚未设置（这就是`ndef`的意思：*未定义*），它将被设置为`yes`；否则，它将保持不变。因为输入`ifndef SOME_VARIABLE`和`endif`有点笨重，GNU `make`提供了一个简便的方式来实现这一模式，即`?=`运算符：

```
BUILD_DEBUG ?= yes
.PHONY: all
all: ; @echo BUILD_DEBUG is $(BUILD_DEBUG)
```

`?=`运算符告诉 GNU `make`将`BUILD_DEBUG`设置为`yes`，除非它已经被定义，在这种情况下保持不变。重新运行测试得到：

```
$ **make**
BUILD_DEBUG is yes
$ **make BUILD_DEBUG=no**
BUILD_DEBUG is no
$ **export BUILD_DEBUG=no**
$ **make**
BUILD_DEBUG is no
```

这种技术提供了最终的灵活性。makefile 中的默认设置可以通过环境或命令行中的临时覆盖来覆盖：

```
$ **export BUILD_DEBUG=no**
$ **make BUILD_DEBUG=aardvark**
BUILD_DEBUG is aardvark
```

### 注意

*实际上，`ifndef`和`?=`在处理已定义但设置为空字符串的变量时有细微的区别。`ifndef`的意思是*如果未定义即使已定义*，而`?=`运算符将空的已定义变量视为已定义。这个差异在第四章中有更详细的讨论。*

# 命令使用的环境

GNU `make`在执行命令时（例如它执行的任何规则中的命令）使用的环境是 GNU `make`启动时的环境，再加上 makefile 中*导出的*任何变量—以及 GNU `make`自己添加的一些变量。

请考虑这个简单的 makefile：

```
FOO=bar

all: ; @echo FOO is $$FOO
```

首先，注意双`$`符号：它是一个转义的`$`，意味着 GNU `make`传递给 shell 的命令是`echo FOO is $FOO`。你可以使用双`$`来在 shell 中获得一个单一的`$`符号。

如果你在环境中没有定义`FOO`，运行这个 makefile 时，你会看到输出`FOO is`。因为 makefile 没有特别地将`FOO`导出到 GNU `make`用来运行命令的环境中，所以`FOO`的值没有被设置。因此，当 shell 执行`all`规则的`echo`命令时，`FOO`没有被定义。如果在运行 GNU `make`之前，环境中已经将`FOO`设置为`foo`，你将看到输出`FOO is bar`。这是因为`FOO`在 GNU `make`启动时已经存在于环境中，随后 makefile 中将`bar`的值赋给了它。

```
$ **export FOO=foo**
$ **make**
FOO is bar
```

如果你不确定`FOO`是否在环境中，但想确保它进入用于命令的环境，可以使用`export`指令。例如，你可以通过修改 makefile，确保`FOO`出现在子进程的环境中，如下所示：

```
**export** FOO=bar

all: ; @echo FOO is $$FOO
```

另外，你可以单独在一行写上`export FOO`。在这两种情况下，`FOO`将被导出到运行`all`规则命令的环境中。

你可以使用`unexport`将一个变量从环境中移除。为了确保`FOO`被排除在子进程的环境之外，无论它是否在父环境中设置，都可以运行以下命令：

```
FOO=bar
**unexport FOO**

all: ; @echo FOO is $$FOO
```

你会看到输出`FOO is`。

你可能会想知道，如果你`export`并`unexport`一个变量，会发生什么。答案是，最后一个指令会生效。

`export`指令也可以与目标特定变量一起使用，以便只修改某个特定规则的环境。例如：

```
export FOO=bar

all: export FOO=just for all

all: ; @echo FOO is $$FOO
```

makefile 将`FOO`设置为`just for all`用于`all`规则，而对其他任何规则则设置为`bar`。

注意，你不能使用目标特定的`unexport`从特定规则的环境中移除`FOO`。如果你写`all: unexport FOO`，你将得到一个错误。

GNU `make`还会向子进程环境中添加一些变量，特别是`MAKEFLAGS`、`MFLAGS`和`MAKELEVEL`。`MAKEFLAGS`和`MFLAGS`变量包含命令行上指定的标志：`MAKEFLAGS`包含用于 GNU `make`内部使用的格式化标志，而`MFLAGS`仅用于历史原因。在配方中不要使用`MAKEFLAGS`。如果确实需要，可以设置`MFLAGS`。`MAKELEVEL`变量包含递归`make`调用的深度，使用`$(MAKE)`，从零开始。有关这些变量的更多细节，请参阅 GNU `make`手册。

你也可以通过单独写一行`export`或指定`.EXPORT_ALL_VARIABLES:`来确保每个 makefile 变量都会被导出。但这些“扫射式”的方法可能不好，因为它们会把一些无用的—甚至可能有害的—变量填充到子进程环境中。

# $(shell)环境

你可能会期望`$(shell)`调用所使用的环境与规则命令执行时所使用的环境相同。事实上，并不是这样的。`$(shell)`使用的环境与 GNU `make`启动时的环境完全相同，没有任何变化。你可以通过以下 makefile 来验证这一点，该 makefile 从`$(shell)`调用和规则中获取`FOO`的值：

```
export FOO=bar

$(info $(shell printenv | grep FOO))

all: ; @printenv | grep FOO
```

这会输出：

```
$ **export FOO=foo**
$ **make**
FOO=foo
FOO=bar
```

无论你做什么，`$(shell)`都会获取父环境。

这是 GNU `make`中的一个错误（错误 #10593——详情请见 *[`savannah.gnu.org/bugs/?10593`](http://savannah.gnu.org/bugs/?10593)*）。之所以没有修复的部分原因是，显而易见的解决方案——在`$(shell)`中使用规则环境——会带来一个相当糟糕的后果。考虑这个 makefile：

```
export FOO=$(shell echo fooey)
all: ; @echo FOO is $$FOO
```

`all`规则中`FOO`的值是多少？要获取`all`环境中的`FOO`的值，必须展开`$(shell)`，这就需要获取`FOO`的值——这又需要展开`$(shell)`调用，依此类推，*无止境*。

面对这个问题，GNU `make`的开发者选择了简单的解决方法：他们根本没有修复这个错误。

鉴于这个错误目前不会消失，因此需要一个解决方法。幸运的是，大多数合适的 shell 都有一种方法可以内联设置环境变量。所以这一节中的第一个 makefile 可以改成：

```
export FOO=bar

$(info $(shell **FOO=$(FOO)** printenv | grep FOO))

all: ; @printenv | grep FOO
```

这将得到期望的结果：

```
$ **make**
FOO=bar
FOO=bar
```

它通过在`$(shell)`函数使用的 shell 中设置`FOO`的值，采用`FOO=$(FOO)`语法来实现。因为`$(shell)`的参数在执行前就会展开，所以变成了`FOO=bar`，其值来自于 makefile 中设置的`FOO`的值。

如果只需要一个额外的变量在环境中，这种方法效果很好。但如果需要很多变量，就有点麻烦了，因为在一行命令中设置多个 shell 变量会变得混乱。

一个更全面的解决方案是编写一个替代`$(shell)`命令的脚本，该命令*确实*导出变量。以下是一个函数`env_shell`，它正是做了这件事：

```
env_file = /tmp/env
env_shell = $(shell rm -f $(env_file))$(foreach V,$1,$(shell echo export
$V=$($V) >> $(env_file)))$(shell echo '$2' >> $(env_file))$(shell /bin/bash -e
$(env_file))
```

在我解释它是如何工作的之前，这里是如何在之前的 makefile 中使用它的方法。你只需将`$(shell)`更改为`$(call env_shell)`。`env_shell`的第一个参数是需要添加到环境中的变量列表，而第二个参数是要执行的命令。以下是更新后的 makefile，其中`FOO`已被导出：

```
export FOO=bar

$(info $(**call env_shell,FOO,printenv** | grep FOO))

all: ; @printenv | grep FOO
```

当你运行这个时，你将看到如下输出：

```
$ **make**
FOO=bar
FOO=bar
```

现在回到`env_shell`是如何工作的。首先，它创建一个 shell 脚本，将其第一个参数中的所有变量添加到环境中；然后，它执行第二个参数中的命令。默认情况下，shell 脚本存储在`env_file`变量指定的文件中（该变量之前设置为*/tmp/env*）。

*/tmp/env* 最终包含：

```
export FOO=bar
printenv | grep FOO
```

我们可以将对`env_shell`的调用分解为四个部分：

+   它通过`$(shell rm -f $(env_file))`删除*/tmp/env*。

+   它通过循环`$(foreach V,$1,$(shell echo export $V=$($V) >> $(env_file)))`添加包含每个变量定义的行，这些变量在第一个参数（`$1`）中指定。

+   它将实际的执行命令（位于第二个参数`$2`中）附加到`$(shell echo '$2' >> $(env_file))`。

+   它使用`-e`选项通过调用`shell`来运行*/tmp/env*：`$(shell /bin/bash -e $(env_file))`。

这不是一个完美的解决方案；如果 GNU `make`能自动决定应该放入环境中的内容，那就太好了。但这是一个可行的解决方案，直到 GNU `make`的开发者修复这个 bug。

# 目标特定变量和模式特定变量

每个 GNU `make`用户都熟悉 GNU `make`的变量。所有 GNU `make`用户都知道，变量本质上具有全局作用域。一旦它们在 makefile 中定义，就可以在 makefile 的任何地方使用。但有多少 GNU `make`用户熟悉 GNU `make`的局部作用域目标特定变量和模式特定变量呢？本节介绍了目标特定变量和模式特定变量，并展示了如何根据目标的名称或构建的目标来选择性地更改构建过程中的选项。

## 目标特定变量

示例 1-1 展示了一个简单的 makefile 示例，说明了 GNU `make`中全局作用域和局部作用域之间的区别：

示例 1-1。一个包含四个虚拟目标的 makefile 示例

```
   .PHONY: all foo bar baz

➊ VAR = global scope

   all: foo bar
   all: ; @echo In $@ VAR is $(VAR)

   foo: ; @echo In $@ VAR is $(VAR)

➋ bar: VAR = local scope
   bar: baz
   bar: ; @echo In $@ VAR is $(VAR)

   baz: ; @echo In $@ VAR is $(VAR)
```

该 makefile 有四个目标：`all`，`foo`，`bar`和`baz`。所有四个目标都是虚拟的；因为我们现在仅关注展示全局和局部作用域，这个 makefile 实际上并不生成任何文件。

`all`目标要求构建`foo`和`bar`，而`bar`依赖于`baz`。每个目标的命令做的事情相同——它们使用`shell echo`打印变量`VAR`的值。

`VAR`变量最初在➊处定义为`global scope`。这是`VAR`在 makefile 中任何地方的值——除非，当然，这个值被目标特定或模式特定变量覆盖。

为了说明局部作用域，`VAR`在➋处被重新定义为`local scope`，用于创建`bar`的规则。目标特定变量的定义与普通变量的定义完全相同：它使用相同的`=`,`:=`，`+=`，和`?=`操作符，但它前面会加上目标名称（及其冒号），用于定义该目标的变量。

如果你在这个 makefile 上运行 GNU `make`，你将看到示例 1-2 所示的输出。

示例 1-2。来自示例 1-1 的输出，显示了全局和局部作用域变量

```
$ **make**
In foo VAR is global scope
In baz VAR is local scope
In bar VAR is local scope
In all VAR is global scope
```

你可以清楚地看到，GNU `make`遵循其标准的深度优先、从左到右的搜索模式。首先它构建`foo`，因为它是`all`的第一个先决条件。然后构建`baz`，它是`bar`的先决条件，`all`的第二个先决条件。接着构建`bar`，最后构建`all`。

果然，在`bar`的规则中，`VAR`的值是`local scope`。因为在`all`或`foo`中没有`VAR`的局部定义，所以在这些规则中，`VAR`的值是`global scope`。

那`baz`怎么办呢？makefile 的输出显示`baz`中的`VAR`值为`local scope`，但实际上并没有为`baz`显式地定义针对特定目标的`VAR`。这是因为`baz`是`bar`的先决条件，因此它具有与`bar`相同的局部作用域变量。

针对特定目标的变量不仅适用于目标本身，还适用于该目标的所有先决条件，以及所有*它们*的先决条件，依此类推。针对特定目标的变量作用域是整个目标树，从定义该变量的目标开始。

请注意，由于`all`、`foo`、`bar`和`baz`的配方完全相同，因此可以将它们写在一行上，如下所示：

```
all foo bar baz: ; @echo In $@ VAR is $(VAR)
```

但在这一节中，我避免了使用多个目标，因为这有时会引起混淆（许多 GNU `make`用户认为这一行代表一个单一规则，将同时为`all`、`foo`、`bar`和`baz`执行，但实际上它是四个独立的规则）。

## 特定模式变量

特定模式的变量工作方式与针对特定目标的变量类似。不过，它们不是为某个目标定义的，而是为某个模式定义的，并应用于所有匹配该模式的目标。以下示例类似于示例 1-1，但已修改为包含特定模式的变量：

```
   .PHONY: all foo bar baz

   VAR = global scope

   all: foo bar
   all: ; @echo In $@ VAR is $(VAR)

   foo: ; @echo In $@ VAR is $(VAR)

   bar: VAR = local scope
   bar: baz
   bar: ; @echo In $@ VAR is $(VAR)

   baz: ; @echo In $@ VAR is $(VAR)

➊ f%: VAR = starts with f
```

最后一行 ➊ 将`VAR`的值设置为`starts with f`，适用于任何以`f`开头并跟着其他任何内容的目标（即`%`通配符）。(也可以使用多个目标来实现这一点，但现在先不讨论这个。)

现在，如果你运行`make`，你会看到如下输出：

```
$ **make**
In foo VAR is starts with f
In baz VAR is local scope
In bar VAR is local scope
In all VAR is global scope
```

这与示例 1-2 是相同的，只是`foo`规则中`VAR`的值已通过特定模式的定义设置为`starts with f`。

值得注意的是，这与 GNU `make`的模式规则无关。你可以使用特定模式的变量定义来更改常规规则中变量的值。你也可以在模式规则中使用它。

例如，假设一个 makefile 使用内建的`%.o: %.c`模式规则：

```
%.o: %.c
#  commands to execute (built-in):
→ $(COMPILE.c) $(OUTPUT_OPTION) $<
```

可以使用特定模式的变量为每个该规则构建的`.o`文件设置一个变量。下面是如何为每个`.o`文件将`-g`选项添加到`CFLAGS`中的方法：

```
%.o: CFLAGS += -g
```

在一个项目中，通常会有一个标准的规则来编译文件，而对于某些特定的文件或文件集合，可能需要稍微不同版本的规则，尽管这些文件最终使用的是相同的命令。例如，以下是一个 makefile，它使用模式规则构建两个子目录（`lib1` 和 `lib2`）中的所有 `.c` 文件：

```
   lib1_SRCS := $(wildcard lib1/*.c)
   lib2_SRCS := $(wildcard lib2/*.c)

   lib1_OBJS := $(lib1_SRCS:.c=.o)
   lib2_OBJS := $(lib2_SRCS:.c=.o)

   .PHONY: all
   all: $(lib1_OBJS) $(lib2_OBJS)

➊ %.o: %.c ; @$(COMPILE.C) -o $@ $<
```

首先，makefile 会将*lib1/*下所有的 `.c` 文件列出并存入 `lib1_SRCS` 变量，将*lib2/*下的 C 文件列出并存入 `lib2_SRCS`。然后，它使用替换引用将这些文件转换为目标文件列表，将 `.c` 文件转换为 `.o` 文件，并将结果存储在 `lib1_OBJS` 和 `lib2_OBJS` 中。最后一行的模式规则 ➊ 使用了 GNU `make` 的内建变量 `COMPILE.C` 来运行编译器，将 `.c` 文件编译成 `.o` 文件。makefile 会构建 `lib1_OBJS` 和 `lib2_OBJS` 中的所有目标文件，因为它们是 `all` 的前提条件。`lib1_OBJS` 和 `lib2_OBJS` 都包含了对应 `.c` 文件的 `.o` 文件列表。当 GNU `make` 搜索 `.o` 文件（即 `all` 的前提条件）时，它发现这些文件缺失，但可以使用 `%.o: %.c` 规则来构建它们。

如果所有的 `.c` 文件使用相同的编译选项，这样做是没问题的。但假设 `.c` 文件*lib1/special.c*需要 `-Wcomment` 选项来防止编译器因注释书写不规范而发出警告。显然，可以通过在 makefile 中添加 `CPPFLAGS += -Wcomment` 这一行来全局更改 `CPPFLAGS` 的值。但是，这样的修改会影响*每个*编译过程，这可能并不是你想要的效果。

幸运的是，你可以使用目标特定变量仅为该单个文件更改 `CPPFLAGS` 的值，如下所示：

```
lib1/special.o: CPPFLAGS += -Wcomment
```

这一行会只在创建*lib1/special.o*时更改 `CPPFLAGS` 的值。

假设一个子目录需要一个特殊的 `CPPFLAGS` 选项来最大化速度优化（例如 `gcc` 的 `-fast` 选项）。在这种情况下，使用特定模式的变量定义是最理想的：

```
lib1/%.o: CPPFLAGS += -fast
```

这样就能解决问题。所有在*lib1/*下构建的 `.o` 文件都将使用 `-fast` 命令行选项来编译。

# 版本检查

因为 GNU `make` 经常更新并不断添加新特性，了解当前运行的 GNU `make` 版本或是否支持某些特定功能非常重要。你可以通过两种方式来做到这一点：要么查看 `MAKE_VERSION` 变量，要么查看 `.FEATURES` 变量（该变量在 GNU `make` 3.81 版本中添加）。你也可以检查特定的功能，比如 `$(eval)`。

## MAKE_VERSION

`MAKE_VERSION` 变量包含正在处理该 makefile 的 GNU `make` 版本号。在这里是一个示例 makefile，它打印出 GNU `make` 的版本并停止执行：

```
.PHONY: all
all: ; @echo $(MAKE_VERSION)
```

下面是当 GNU `make` 3.80 解析这个 makefile 时生成的输出：

```
$ **make**
3.80
```

如果你想确定 GNU `make` 版本 3.80 或更高版本正在处理你的 Makefile，怎么办？如果假设版本号始终为 `X.YY.Z` 或 `X.YY` 形式，那么以下代码片段将在 `need` 中提到的版本小于或等于运行版本时，将 `ok` 变量设置为非空。

```
need := 3.80
ok := $(filter $(need),$(firstword $(sort $(MAKE_VERSION) $(need))))
```

如果 `ok` 不为空，则表示正在使用所需版本的 GNU `make` 或更高版本；如果为空，则表示版本过旧。该代码片段通过创建一个以空格分隔的 GNU `make` 运行版本列表（存储在 `MAKE_VERSION` 中）和所需版本（来自 `need`）来工作，并对该列表进行排序。假设运行的版本是 3.81，那么 `$(sort $(MAKE_VERSION) $(need))` 将是 `3.80 3.81`。该列表的 `$(firstword)` 是 `3.80`，因此 `$(filter)` 调用将保留 `3.80`，从而 `ok` 变量将非空。

现在假设运行版本是 3.79.1，那么 `$(sort $(MAKE_VERSION) $(need))` 将是 `3.79.1 3.80`，`$(firstword)` 将返回 `3.79.1`。`$(filter)` 调用将移除 `3.79.1`，因此 `ok` 将为空。

### 注意

*此代码片段在 GNU `make` 版本从 10.01 开始时将无法正确工作，因为它假设主版本号为单数位数。幸运的是，这还需要很长时间！*

## .FEATURES

GNU `make` 3.81 引入了 `.FEATURES` 默认变量，该变量包含一个支持特性的列表。在 GNU `make` 3.81 中，`.FEATURES` 列出了并支持七个特性：

+   ****`archives`****。使用 `archive(member)` 语法归档（`ar`）文件

+   ****`check-symlink`****。`-L` 和 `--check-symlink-times` 标志

+   ****`else-if`****。非嵌套形式 `else if X` 的 else 分支

+   ****`jobserver`****。使用作业服务器并行构建

+   ****`order-only`****。`order-only` 先决条件支持

+   ****`second-expansion`****。先决条件列表的双重展开

+   ****`target-specific`****。目标特定和模式特定变量

GNU `make` 3.82 添加并支持以下内容：

+   ****`oneshell`****。`.ONESHELL` 特殊目标

+   ****`shortest-stem`****。在选择匹配目标的模式规则时使用最短的词干选项

+   ****`undefine`****。`undefine` 指令

并且 GNU `make` 4.0 添加了以下内容：

+   ****`guile`****。如果 GNU `make` 是在支持 GNU Guile 的环境下构建的，那么该功能将会存在，并且 `$(guile)` 函数将被支持。

+   ****`load`****。支持加载动态对象以增强 GNU `make` 功能。

+   ****`output-sync`****。支持 `-O`（和 `--output-sync`）命令行选项。

你可以在 近期的 GNU make 版本：3.81，3.82 和 4.0 中找到更多关于这些以及其他许多特性的详细信息。

若要检查是否有特定功能可用，可以使用以下 `is_feature` 函数：如果请求的功能受支持，则返回 `T`，如果功能缺失，则返回空字符串：

```
is_feature = $(if $(filter $1,$(.FEATURES)),T)
```

例如，下面的 makefile 使用`is_feature`来回显`archives`特性是否可用：

```
.PHONY: all
all: ; @echo archives are $(if $(call is_feature,archives),,not )available
```

下面是使用 GNU `make` 3.81 时的输出：

```
$ **make**
archives are available
```

如果你想检查`.FEATURES`变量是否被支持，可以使用如 MAKE_VERSION 中所述的`MAKE_VERSION`，或者简单地展开`.FEATURES`并查看其是否为空。以下的 makefile 片段正是执行这一操作，如果`.FEATURES`变量存在并且包含任何特性，则将`has_features`设置为`T`（代表 true）：

```
has_features := $(if $(filter default,$(origin .FEATURES)),$(if $(.FEATURES),T))
```

该片段首先使用`$(origin)`来检查`.FEATURES`变量是否是默认变量；这样，如果有人在 makefile 中定义了`.FEATURES`，`has_features`就不会被误导。如果它是默认变量，第二个`$(if)`会检查`.FEATURES`是否为空。

## 检测`$(eval)`

`$(eval)`函数是一个强大的 GNU `make`特性，新增于版本 3.80。`$(eval)`的参数会被展开，然后解析，仿佛它是 makefile 的一部分，从而允许你在运行时修改 makefile。

如果你使用`$(eval)`，那么重要的是要检查该特性是否在读取你的 makefile 的 GNU `make`版本中可用。你可以使用前面提到的`MAKE_VERSION`来检查版本是否为 3.80。或者，你也可以使用以下代码片段，这段代码只有在`$(eval)`被实现时才会将`eval_available`设置为`T`：

```
$(eval eval_available := T)
```

如果`$(eval)`不可用，GNU `make`将寻找一个名为`eval eval_available := T`的变量并尝试获取其值。当然，这个变量并不存在，因此`eval_available`将被设置为空字符串。

你可以使用`eval_available`配合`ifneq`来生成一个致命错误，如果`$(eval)`没有被实现的话。

```
ifneq ($(eval_available),T)
$(error This makefile only works with a Make program that supports $$(eval))
endif
```

`eval_available`函数特别有用，如果你无法检查`MAKE_VERSION`，例如，如果你的 makefile 是通过非 GNU 的`make`工具运行的，如`clearmake`或`emake`。

# 使用布尔值

GNU `make`的`$(if)`函数和`ifdef`构造都将空字符串和未定义的变量视为 false，其他任何内容视为 true。但它们在评估参数时有细微的不同。

`$(if)`函数，也就是`$(if` *`X`*`,`*`if-part`*`,`*`else-part`*`)`，会在*`X`*不为空时展开*`if-part`*，否则展开*`else-part`*。在使用`$(if)`时，条件会被展开，并且*展开后的值*会被测试是否为空。以下代码片段报告了它走了*`else-part`*分支：

```
EMPTY =
VAR = $(EMPTY)
$(if $(VAR),$(info if-part),$(info else-part))
```

而接下来的片段则走了*`if-part`*分支，因为`HAS_A_VALUE`具有非空的值。

```
HAS_A_VALUE = I'm not empty
$(if $(HAS_A_VALUE),$(info if-part),$(info else-part))
```

`ifdef`构造的工作方式略有不同：它的参数是一个变量的*名称*，并不会进行展开：

```
ifdef VAR
if-part...
else
else-part...
endif
```

上述示例会在变量`VAR`不为空时执行*`if-part`*，而在`VAR`为空或未定义时执行*`else-part`*。

## 条件中的未定义变量

因为 GNU `make`将未定义的变量视为空值，`ifdef`实际上应该叫做`ifempty`——特别是因为它将已定义但为空的变量视为未定义。例如，以下代码片段报告`VAR`未定义：

```
VAR =
ifdef VAR
$(info VAR is defined)
else
$(info VAR is undefined)
endif
```

在实际的 makefile 中，这可能不是预期的结果。你可以通过`--warn-undefined-variables`命令行选项来请求未定义变量的警告。

`ifdef`的另一个细微差别是，它不会展开变量`VAR`。它只是检查`VAR`是否已被定义为非空值。以下代码片段报告`VAR`已定义，即使其完全展开后的值是空字符串：

```
EMPTY =
VAR = $(EMPTY)
ifdef VAR
$(info VAR is defined)
else
$(info VAR is not defined)
endif
```

GNU `make` 3.81 版本为`ifdef`引入了另一个变化：它的参数会被展开，从而可以计算出被测试的变量名。这对条件语句，如`ifdef VAR`没有影响，但允许你编写如下代码：

```
VAR_NAME = VAR
VAR = some value
ifdef $(VAR_NAME)
$(info VAR is defined)
else
$(info VAR is not defined)
endif
```

这与以下内容完全相同：

```
VAR = some value
ifdef VAR
$(info VAR is defined)
else
$(info VAR is not defined)
endif
```

在这两种情况下，`VAR`被检查是否为空，就像之前描述的那样，在两个输出中都会显示`VAR is defined`。

## 一致的真值

GNU `make`将任何非空字符串视为真。但如果你经常与真值和`$(if)`打交道，使用一个一致的真值可能会更方便。以下`make-truth`函数将任何非空字符串转为`T`：

```
make-truth = $(if $1,T)
```

请注意，我们可以去掉`$(if)`中的`else`部分，因为它是空的。在本书中，我会省略那些不必要的参数，而不是用多余的尾随逗号污染 makefile。但如果让你更舒服，你完全可以写`$(if $1,T,)`。

以下所有对`make-truth`的`call`都会返回`T`：

```
➊ $(call make-truth, )
   $(call make-truth,true)
   $(call make-truth,a b c)
```

即使是➊也返回`T`，因为通过`$(call)`调用的函数的参数在放入`$1`、`$2`等变量之前，并不会进行任何修改——甚至不会去除首尾的空格。因此，第二个参数是一个包含单个空格的字符串，而不是空字符串。

以下所有代码都会返回空字符串（表示假）：

```
➋ $(call make-truth,)
   EMPTY =
   $(call make-truth,$(EMPTY))
   VAR = $(EMPTY)
   $(call make-truth,$(VAR))
```

仔细观察➊和➋之间的区别：GNU `make`中的空格可能非常重要！

# 使用布尔值的逻辑操作

GNU `make`在 3.81 版本之前没有内建的逻辑运算符，直到那个版本才加入了`$(or)`和`$(and)`。然而，创建操作布尔值的用户自定义函数非常容易。这些函数通常使用 GNU `make`的`$(if)`函数来做决策。`$(if)`将任何非空字符串视为`'true'`，将空字符串视为`'false'`。

## 用户自定义逻辑运算符

让我们创建一个用户自定义的最简单逻辑运算符`or`。如果任意一个参数为真（即非空字符串），结果也应该是非空字符串。我们可以通过简单地连接参数来实现这一点：

```
or = $1$2
```

你可以在 一致的布尔值 中使用 `make-truth` 函数来清理 `or` 的结果，使其变为 `T`（真）或空字符串（假）：

```
or = $(call make-truth,$1$2)
```

或者，对于更简洁的版本，你只需要写：

```
or = $(if $1$2,T).
```

以下所有的返回 `T`：

```
$(call or, , )
$(call or,T,)
$(call or, ,)
$(call or,hello,goodbye my friend)
```

从 `or` 中返回假值的唯一方法是传入两个空的参数：

```
EMPTY=
$(call or,$(EMPTY),)
```

定义 `and` 稍微复杂一些，需要两次调用 `$(if)`：

```
and = $(if $1,$(if $2,T))
```

不需要将其包装在 `make-truth` 中，因为如果其参数非空，它总是返回 `T`，如果任一参数为空，则返回空字符串。

定义 `not` 只是一个简单的 `$(if)`：

```
not = $(if $1,,T)
```

在定义了 `and`、`or` 和 `not` 之后，你可以快速创建其他逻辑运算符：

```
nand = $(call not,$(call and,$1,$2)) nor = $(call not,$(call or,$1,$2))
xor = $(call and,$(call or,$1,$2),$(call not,$(call and,$1,$2)))
```

这些也有简化版本，只需要使用 `$(if)`：

```
nand = $(if $1,$(if $2,,T),T)
nor = $(if $1$2,,T)
xor = $(if $1,$(if $2,,T),$(if $2,T))
```

作为练习，试着编写一个 `xnor` 函数！

## 内建逻辑运算符（GNU make 3.81 及以后版本）

GNU `make` 3.81 及以后版本有内建的 `and` 和 `or` 函数，这些函数比之前定义的版本更快，因此在可能的情况下，最好使用这些内建函数。你应该测试 `and` 和 `or` 函数是否已存在，只有在它们不存在时才定义你自己的版本。

确定 `and` 和 `or` 是否已定义的最简单方法是尝试使用它们：

```
have_native_and := $(and T,T)
have_native_or := $(or T,T)
```

这些变量只有在内建的 `and` 和 `or` 函数存在时才会是 `T`。在 GNU `make` 3.81 之前的版本（或类似 `clearmake` 的模拟程序）中，`have_native_and` 和 `have_native_or` 将为空，因为 GNU `make` 找不到名为 `and` 或 `or` 的函数，也找不到名为 `and T`、`T` 或 `or T`、`T` 的变量！

你可以使用 `ifneq` 来检查这些调用的结果，并仅在必要时定义你自己的函数，像这样：

```
ifneq ($(have_native_and),T)
and = $(if $1,$(if $2,T))
endif
ifneq ($(have_native_or),T)
or = $(if $1$2,T)
endif

$(info This will be T: $(call and,T,T))
```

你可能会担心，你已经在各处写了 `$(call and,...)` 和 `$(call or,...)`，用 `call` 来调用你自己的逻辑运算符。你是不是需要将它们全部改成 `$(and)` 和 `$(or)`——去掉 `call` 来使用内建的运算符？

这是不必要的。GNU `make` 允许使用 `call` 关键字调用任何内建函数，因此 `$(and...)` 和 `$(call and,...)` 都会调用内建运算符。然而，相反的情况 *并不* 成立：无法通过编写 `$(foo arg1,arg2)` 来调用 *用户定义* 的函数 `foo`。你必须写成 `$(call foo,arg1,arg2)`。

因此，定义你自己的 `and` 和 `or` 函数，并在 GNU `make` 3.81 或更高版本下优雅地运行，只需要前面显示的几行来定义 `and` 和 `or`——不需要其他更改。

请注意，内建函数和用户定义的版本之间有一个重要区别。如果第一个参数完全决定了其真值，内建版本将不会评估第二个参数。例如，如果 `$a` 为假，则 `$(and $a,$b)` 不需要查看 `$b` 的值；如果 `$a` 为真，则 `$(or $a,$b)` 不需要查看 `$b` 的值。

如果您需要这种行为，则不能使用前面的用户定义版本，因为在执行`$(call)`函数时，所有参数都会被展开。替代方案是将`$(call and,X,Y)`替换为`$(if X,$(if Y,T))`，将`$(call or,X,Y)`替换为`$(if X,T,$(if Y,T))`。

# 命令检测

有时，在 makefile 中快速返回错误信息，如果构建系统中缺少特定软件会非常有用。例如，如果 makefile 需要`curl`程序，在解析时（即 make 加载 makefile 时）检查系统中是否存在`curl`会比在构建过程中才发现它不存在更为有用。

查找命令是否可用的最简单方法是使用`which`命令，并将其放在`$(shell)`调用中。如果命令不存在，则返回空字符串；如果命令存在，则返回命令的路径，这与`make`的*空字符串表示假，非空字符串表示真*逻辑非常契合。

例如，以下代码在`curl`存在时将`HAVE_CURL`设置为非空字符串：

```
HAVE_CURL := $(shell which curl)
```

然后，您可以使用`HAVE_CURL`来停止构建并在`curl`缺失时输出错误：

```
ifndef HAVE_CURL
$(error curl is missing)
endif
```

以下的`assert-command-present`函数将此逻辑封装为一个便捷的函数。调用`assert-command-present`并传入命令的名称，如果命令缺失，构建将立即退出并输出错误。以下示例使用`assert-command-present`检查`curl`和名为`curly`的命令是否存在：

```
assert-command-present = $(if $(shell which $1),,$(error '$1' missing and needed for this build))

$(call assert-command-present,curl)
$(call assert-command-present,curly)
```

如果在一个有`curl`但没有`curly`的系统上运行这段代码，会发生以下情况：

```
$ **make**
Makefile:4: *** 'curly' missing and needed for this build. Stop.
```

如果一个命令仅由某些构建目标使用，那么仅在相关目标下使用`assert-command-present`是有用的。以下的 makefile 将在`download`目标作为构建的一部分实际使用时，检查`curly`是否存在：

```
all: ; @echo Do all...

download: export _check = $(call assert-command-present,curly)
download: ; @echo Download stuff...
```

`download`目标的第一行设置了一个名为`_check`的目标特定变量，并将其导出为对`assert-command-present`调用的结果。这会导致`$(call)`仅在`download`作为构建的一部分时发生，因为当准备将其插入到配方的环境中时，`_check`的值会被展开。例如，`make all`不会检查`curly`是否存在：

```
$ **make**
Do all...
$ **make download**
Makefile:5: *** 'curly' missing and needed for this build. Stop.
```

请注意，这个 makefile 定义了一个名为`_`的变量，您可以通过`$(_)`甚至`$_`来访问它。使用下划线作为名称是一种表示该变量只是占位符，并且其值应该被忽略的方法。

# 延迟变量赋值

GNU `make`提供了两种定义变量的方式：简单的`:=`操作符和递归的`=`操作符。简单的`:=`操作符会立即评估右侧的值，并使用结果值来设置变量的值。例如：

```
BAR = before
FOO := $(BAR) the rain
BAR = after
```

这个代码片段会导致`FOO`的值为`before the rain`，因为当使用`:=`设置`FOO`时，`BAR`的值为`before`。

相比之下，

```
BAR = before
FOO = $(BAR) the rain
BAR = after
```

这导致`FOO`的值为`$(BAR) the rain`，而`$(FOO)`的值为`after the rain`。这是因为`=`定义了一个递归变量（可以包含其他变量引用的变量，使用`$()`或`${}`语法），其值在每次使用该变量时被确定。相比之下，使用`:=`定义的简单变量在定义时通过立即展开所有变量引用来确定一个固定的值。

简单变量具有明显的速度优势，因为它们是固定字符串，不需要每次使用时都进行展开。它们的使用可能有些棘手，因为 makefile 编写者常常假设变量可以按任意顺序设置，因为递归定义的变量（用`=`设置的变量）只有在使用时才会获得最终值。然而，简单变量通常比递归变量更快速访问，如果可能，我倾向于总是使用`:=`。

但是如果你能够兼顾两者的优点呢？一个变量，在首次使用时才会被设置，但它会被设定为一个固定值，且不会改变。如果变量的值需要大量计算，但最多只需要计算一次，甚至如果变量从未被使用则根本不计算，这将非常有用。这可以通过`$(eval)`函数实现。

考虑以下定义：

```
SHALIST = $(shell find . -name '*.c' | xargs shasum)
```

`SHALIST`变量将包含当前目录及所有子目录中每个`.c`文件的名称和 SHA1 加密哈希值。这个评估可能需要很长时间。而使用`=`定义`SHALIST`意味着每次使用`SHALIST`时都会发生这个昂贵的调用。如果使用多次，可能会显著降低 makefile 的执行速度。

另一方面，如果你使用`:=`定义`SHALIST`，`$(shell)`只会执行一次，但每次加载 makefile 时都会发生。如果`SHALIST`的值并不总是需要，比如在运行`make clean`时，这可能效率低下。

我们希望能够定义`SHALIST`，使得如果`SHALIST`从未使用，则`$(shell)`不会执行；而如果`SHALIST`被使用，则仅执行一次。下面是如何实现：

```
SHALIST = $(eval SHALIST := $(shell find . -name '*.c' | xargs shasum))$(SHALIST)
```

如果`$(SHALIST)`被评估，`$(eval SHALIST := $(shell find . -name '*.c' | xargs shasum))`部分将会被评估。因为这里使用了`:=`，它实际上会执行`$(shell)`并将`SHALIST`重新定义为该调用的结果。然后，GNU `make`会获取由`$(eval)`刚刚设置的`$(SHALIST)`的值。

你可以通过创建一个小的 makefile，使用`$(value)`函数（该函数显示变量的定义而不展开它）来查看`SHALIST`的值，而不对其进行评估：

```
   SHALIST = $(eval SHALIST := $(shell find . -name '*.c' | xargs
   shasum))$(SHALIST)

   $(info Before use SHALIST is: $(value SHALIST))
➊ $(info SHALIST is: $(SHALIST))
   $(info After use SHALIST is: $(value SHALIST))
```

使用目录中的一个`foo.c`文件运行该 makefile，结果会产生以下输出：

```
$ **make**
Before use SHALIST is: $(eval SHALIST := $(shell find . -name '*.c' | xargs
shasum))$(SHALIST)
SHALIST is: 3405ad0433933b9b489756cb3484698ac57ce821 ./foo.c
After use SHALIST is: 3405ad0433933b9b489756cb3484698ac57ce821 ./foo.c
```

显然，`SHALIST`的值自从第一次在➊使用时已经发生了变化。

# 简单的列表操作

在 GNU `make` 中，列表元素由空格分隔。例如，`peter paul and mary` 是一个包含四个元素的列表，`C:\Documents And Settings\Local User` 也是一个列表，包含四个元素。GNU `make` 提供了多个内置函数来操作列表：

+   ****`$(firstword)`****。获取列表中的第一个单词。

+   ****`$(words)`****。计算列表元素的数量。

+   ****`$(word)`****。提取指定索引的单词（从 1 开始计数）。

+   ****`$(wordlist)`****。从列表中提取一系列单词。

+   ****`$(foreach)`****。允许你遍历一个列表。

获取列表中的第一个元素很简单：

```
MY_LIST = a program for directed compilation
$(info The first word is $(firstword $(MY_LIST)))
```

那将输出 `The first word is a`。

你可以通过计算列表中单词的数量 *N* 来获取最后一个元素，然后取出第 *N* 个单词。这里有一个 `lastword` 函数，它返回列表中的最后一个单词：

```
➊ lastword = $(if $1,$(word $(words $1),$1))
   MY_LIST = a program for directed compilation
   $(info The last word is $(call lastword,$(MY_LIST)))
```

➊ 处的 `$(if)` 是必须的，因为如果列表为空，`$(words $1)` 将返回 `0`，而 `$(word 0,$1)` 会导致致命错误。前面的示例输出是 `The last word is compilation`。

### 注意

*GNU `make` 3.81 及更高版本内置了一个 `lastword` 函数，比之前的实现更快。*

剪去列表中的第一个单词只需返回从第二个元素到最后的子列表范围即可。GNU `make` 的内置 `$(wordlist` *`S`*`,`*`E`*`,`*`LIST`*`)` 函数返回 *`LIST`* 中从索引 *`S`* 开始，到索引 *`E`* 结束（包括 *`E`*）的元素范围：

```
notfirst = $(wordlist 2,$(words $1),$1)
MY_LIST = a program for directed compilation
$(info $(call notfirst,$(MY_LIST)))
```

你不需要担心前面示例中的空列表，因为 `$(wordlist)` 如果第二个参数不是有效的索引，也不会报错。那个示例的输出是 `program for directed compilation`。

剪去列表中的最后一个元素需要一些额外的思考，因为在 `make` 中没有简单的算术运算方法：不能直接写 `$(wordlist 1,$(words $1)–1, $1)`。相反，我们可以定义一个 `notlast` 函数，通过在列表开头添加一个虚拟元素，并使用 *原始* 列表的长度作为 `$(wordlist)` 的结束索引，从而剪掉最后一个元素。然后，因为我们添加了一个虚拟元素，我们需要记得通过将 `$(wordlist)` 的起始索引设置为 `2` 来去除它：

```
notlast = $(wordlist 2,$(words $1),dummy $1)
MY_LIST = a program for directed compilation
$(info $(call notlast,$(MY_LIST)))
```

这将输出 `a program for directed`。

# 用户定义的函数

本节介绍如何在 makefile 中定义 `make` 函数。在第五章，你将学习如何修改 GNU `make` 的源代码，使用 C 定义更复杂的函数。在前面的章节中，我们使用了很多用户定义的函数，现在我们将更详细地探讨这个话题。

## 基础知识

这是一个非常简单的 `make` 函数，它接受三个参数，并通过在这三个参数之间插入斜杠来生成日期：

```
make_date = $1/$2/$3
```

要使用 `make_date`，你可以像这样调用它：`$(call)`。

```
today := $(call make_date,5,5,2014)
```

结果是 `today` 包含 `5/5/2014`。

该函数使用了特殊变量`$1`、`$2`和`$3`，它们包含了在`$(call)`中指定的参数。没有参数数量的上限，但如果使用超过九个参数，则需要使用括号——也就是说，你不能写`$10`，而必须使用`$(10)`。如果函数调用时缺少某些参数，这些变量的内容将是未定义的，并被视为空字符串。

特殊参数`$0`包含函数的名称。在前面的例子中，`$0`是`make_date`。

由于函数本质上是引用一些由 GNU `make`自动创建和填充的特殊变量（如果你在任何参数变量（如`$1`等）上使用`$(origin)`函数，它们会被分类为`automatic`，就像`$@`一样），你可以使用 GNU `make`的内建函数来构建复杂的函数。

这是一个使用`$(subst)`函数将路径中的每个`/`转换为`\`的函数：

```
unix_to_dos = $(subst /,\,$1)
```

不必担心代码中`/`和`\`的使用。GNU `make`几乎不做转义处理，字面上的`\`大部分时间都代表一个实际的反斜杠字符。你将在第四章中了解到更多关于`make`如何处理转义的内容。

## 参数处理陷阱

`make`在处理`$(call)`时，会通过逗号分隔参数列表来设置变量`$1`、`$2`等。然后展开这些参数，确保这些变量在引用之前被完全展开。这就像`make`使用`:=`来设置它们一样。如果展开一个参数时有副作用，比如调用`$(shell)`，这个副作用会在`$(call)`执行时立即发生，即使该参数最终并未被调用的函数使用。

一个常见的问题是，如果参数中包含逗号，分割参数时可能会出错。例如，这里有一个简单的函数，它交换两个参数：

```
swap = $2 $1
```

如果你使用`$(call swap,first,argument,second)`，`make`没有办法知道第一个参数是想表示`first,argument`还是仅仅是`first`。它会假设后者，并最终返回`argument first`，而不是`second first,argument`。

你有两种方法来解决这个问题。首先，你可以简单地将第一个参数隐藏在一个变量中。因为`make`在分割参数之前不会展开这些参数，所以变量中的逗号不会引起任何混淆：

```
FIRST := first,argument
SWAPPED := $(call swap,$(FIRST),second)
```

另一种方法是创建一个仅包含逗号的简单变量，并使用它：

```
c := ,
SWAPPED := $(call swap,first$cargument,second)
```

或者甚至可以调用这个`,`变量并使用它（带括号）：

```
, := ,
SWAPPED := $(call swap,first$(,)argument,second)
```

正如我们将在第四章中看到的，给变量起一些巧妙的名字，如`,`，可能很有用，但也容易出错。

## 调用内建函数

你可以使用`$(call)`语法与`make`的内建函数一起使用。例如，你可以像这样调用`$(info)`：

```
$(call info,message)
```

这意味着你可以将任何函数名作为参数传递给用户定义的函数，并使用`$(call)`来调用它，而无需知道它是否是内置函数；因此，它允许你创建作用于函数的函数。例如，你可以创建经典的函数式编程中的`map`函数，该函数将一个函数应用于列表中的每个成员，并返回结果列表：

```
map = $(foreach a,$2,$(call $1,$a))
```

第一个参数是要调用的函数，第二个参数是要遍历的列表。以下是`map`的一个示例用法——遍历一个变量名列表，并打印每个变量的定义值和扩展值：

```
print_variable = $(info $1 ($(value $1) -> $($1)) )

print_variables = $(call map,print_variable,$1)
VAR1 = foo
VAR2 = $(VAR1)
VAR3 = $(VAR2) $(VAR1)

$(call print_variables,VAR1 VAR2 VAR3)
```

`print_variable`函数将变量名作为它的第一个也是唯一的参数，并返回一个由变量名、定义和其值组成的字符串。`print_variables`函数只是使用`map`将`print_variable`应用于一组变量列表。以下是 makefile 代码片段的输出结果：

```
$ **make**
VAR1 (foo -> foo) VAR2 ($(VAR1) -> foo) VAR3 ($(VAR2) $(VAR1) -> foo foo)
```

`make`中的函数也可以是递归的：函数可以调用`$(call)`自身。下面是一个递归实现的`reduce`函数，来自函数式编程，它接受两个参数：一个会被`reduce`调用的函数和一个待处理的列表。

```
reduce = $(if $(strip $2),$(call reduce,$1,$(wordlist 2,$(words $2),$2), \
$(call $1,$(firstword $2),$3)),$3)
```

第一个参数（函数）会反复使用两个参数进行调用：列表中的下一个元素是`reduce`的第二个参数，前一次调用该函数的结果是第一个参数。

要查看其工作原理，下面是一个`uniq`函数，用于从列表中删除重复项：

```
check_uniq = $(if $(filter $1,$2),$2,$2 $1)
uniq = $(call reduce,check_uniq,$1)
$(info $(call uniq,c b a a c c b a c b a))
```

这里的输出是`c b a`。之所以能这样工作，是因为`reduce`会使用输入列表中的每个成员调用`check_uniq`，并从`check_uniq`的结果构建一个新列表。`check_uniq`函数仅仅是判断一个元素是否存在于给定的列表中（使用内置的`filter`函数），如果不存在，则返回一个将该元素附加到列表后的新列表。

要查看其实际效果，下面是一个修改版，使用`$(info)`在每次调用`check_uniq`时输出传递给它的参数：

```
check_uniq = $(info check_uniq ($1) ($2))$(if $(filter $1,$2),$2,$2 $1)
uniq = $(call reduce,check_uniq,$1)
$(info $(call uniq,c b a a c c b a c b a))
```

以下是输出结果：

```
$ make
check_uniq (c) ()
check_uniq (b) ( c)
check_uniq (a) ( c b)
check_uniq (a) ( c b a)
check_uniq (c) ( c b a)
check_uniq (c) ( c b a)
check_uniq (b) ( c b a)
check_uniq (a) ( c b a)
check_uniq (c) ( c b a)
check_uniq (b) ( c b a)
check_uniq (a) ( c b a)
c b a
```

如果不需要保留顺序，那么使用内置的`$(sort)`函数会比这个用户定义的函数更快，因为它也会删除重复项。

# 最新的 GNU make 版本：3.81、3.82 和 4.0

GNU `make`的变化很慢，新版本（包括主版本和次版本）通常每隔几年才发布一次。由于发布周期较慢，因此常常会遇到旧版本的 GNU `make`，了解它们之间的差异非常有用。本节假设最常用的旧版本是 3.79.1（发布于 2000 年 6 月 23 日），并重点介绍了 3.81、3.82 和 4.0 版本中的主要变化。

## GNU make 3.81 中的新功能

GNU `make` 3.81 于 2006 年 4 月 1 日发布，比上一个版本（GNU `make` 3.80）晚了三年半，且新版本中加入了许多新特性：支持 OS/2、新的命令行选项、新的内建变量、新的条件语句和新函数。有关更改的完整列表，请参阅 GNU `make` 3.81 源代码分发包中的*NEWS*文件。

### .SECONDEXPANSION

用户使用 GNU `make`时常遇到的一个令人沮丧的问题是，自动变量只有在规则的命令被执行时才有效并被赋值；它们在规则定义部分是无效的。例如，不能写`foo: $@.c`来表示`foo`应该由`foo.c`生成，尽管当该规则的命令被执行时，`$@`的值会是`foo`。这令人沮丧，因为如果不必像这样重复自己就好了：

```
foo:foo.c
```

在 3.81 版本之前，GNU `make`支持在规则的前提条件列表中使用`$$@`（注意两个`$`符号）（该语法来自 SysV `make`）。例如，可以写`foo: $$@.c`，它等同于`foo: foo.c`。也就是说，`$$@`具有在规则命令中`$@`的值。要在 GNU `make` 3.81 及更高版本中获得此功能，必须在 makefile 中定义`.SECONDEXPANSION`。作为附加功能，GNU `make`支持在规则定义中使用所有标准的自动变量（尽管请注意，像`$$`这样的自动变量始终为空，因为它们无法在解析 makefile 时计算）。这发生的原因是，GNU `make`会对规则的前提条件列表进行两次扩展：第一次是在读取 makefile 时，第二次是在查找要构建的目标时。

你可以使用第二次扩展（second expansion）不仅仅是自动变量。用户定义的变量也可以被*第二次扩展*，它们最终会得到在 makefile 中定义的最后一个值。例如，你可以这样做：

```
.SECONDEXPANSION:

FOO = foo

all: $$(FOO)
all: ; @echo Making $@ from $?

bar: ; @echo Making $@

FOO = bar
```

这将产生以下输出：

```
$ **make**
Making bar
Making all from bar
```

当 makefile 被读取时，`all: $$(FOO)`会被扩展为`all: $(FOO)`。后来，当决定如何构建`all`时，`$(FOO)`被扩展为`bar`——也就是说，这是`FOO`在 makefile 解析结束时的值。请注意，如果你启用了`.SECONDEXPANSION`并且文件名中有`$`符号，那么`$`符号需要通过写`$$`来转义。

#### else

GNU `make` 3.81 中引入的另一个新特性是通过将条件和`else`写在同一行来支持非嵌套的`else`分支。例如，可以写：

```
ifdef FOO
$(info FOO defined)
else ifdef BAR
$(info BAR defined)
else
$(info BAR not defined)
endif
```

这种语法对任何使用过支持`else if`、`elseif`或`elsif`的语言的人来说都很熟悉。这是 GNU `make`将`else`和`if`写在同一行的方式。

之前，代码会像这样：

```
ifdef FOO
$(info FOO defined)
else
ifdef BAR
$(info BAR defined)
else
$(info BAR not defined)
endif
endif
```

这种写法比起带有非嵌套`else`分支的版本，要乱得多，且更难以阅读。

#### -L 命令行选项

命令行选项 `-L`（及其长形式 `--check-symlink-times`）使 `make` 考虑符号链接的修改时间以及符号链接所指向文件的修改时间，以便决定哪些文件需要重新编译。较新的修改时间将被视为文件的修改时间。这在构建使用符号链接指向不同版本源文件时非常有用，因为改变符号链接将更改修改时间，并强制重新构建。

### .INCLUDE_DIRS

`.INCLUDE_DIRS` 变量包含 `make` 在查找通过 `include` 指令包含的 makefile 时会搜索的目录列表。该变量由 GNU `make` 内置的标准目录列表设置，并可以通过 `-I` 命令行选项进行修改。尽管可以在实际的 makefile 中通过 `=` 或 `:=` 来改变 `.INCLUDE_DIRS` 的值，但这不会影响 GNU `make` 查找 makefile 的方式。

例如，在 Linux 上运行 `make -I /usr/foo` 并使用以下 makefile 输出 `/usr/foo /usr/local/include /usr/local/include /usr/include`：

```
$(info $(.INCLUDE_DIRS))
all: ; @true
```

### .FEATURES

`.FEATURES` 变量展开为 GNU `make` 支持的特性列表，可用于判断特定功能是否可用。在 Linux 上使用 GNU `make` 3.81 时，`.FEATURES` 的列表为 `target-specific order-only second-expansion else-if archives jobserver check-symlink`。这意味着 GNU `make` 3.81 支持特定目标和模式的变量，具有 orderonly 先决条件，支持第二次展开（`.SECONDEXPANSION`），支持 `else if` 非嵌套条件，支持 `ar` 文件，支持使用作业服务器进行并行编译，并支持用于检查符号链接的新 `-L` 命令行选项。

要测试特定功能是否可用，可以使用 `$(filter)`。例如：

```
has-order-only := $(filter order-only,$(.FEATURES))
```

这一行设置 `has-order-only` 为 true，前提是当前运行的 `make` 版本支持 order-only 先决条件。然而，这并不向后兼容；例如，在 GNU `make` 3.80 中，`.FEATURES` 会展开为一个空列表，表示即使特定目标变量可用，它们仍不可用。向后兼容的检查首先需要通过查看 `.FEATURES` 是否非空来判断它是否存在。

### .DEFAULT_GOAL

通常，如果命令行中未指定目标，`make` 将构建它在第一个解析的 makefile 中看到的第一个目标。可以通过在 makefile 中的任何位置设置 `.DEFAULT_GOAL` 变量来覆盖此行为。例如，以下 makefile 即使第一个目标是 `fail`，在没有命令行目标的情况下运行时，仍将构建 `all`：

```
fail: ; $(error wrong)
.DEFAULT_GOAL = all
all: ; $(info right)
```

`.DEFAULT_GOAL` 变量也可以读取当前的默认目标；如果设置为空（`.DEFAULT_GOAL :=`），`make` 将自动选择它遇到的下一个目标作为默认目标。

### MAKE_RESTARTS

`MAKE_RESTARTS` 变量表示 `make` 在执行 makefile *重建* 时重启的次数。GNU `make` 有一个特殊功能，允许 makefile 由 `make` 自动重建。这种重建发生在任何通过 `include` 引入的 makefile 中，以及最初启动的 makefile 和通过 `-f` 命令行选项设置的 makefile。`make` 会检查是否有规则来重建任何 makefile。如果找到，makefile 会像任何其他文件一样被重建，且 GNU `make` 会重启。

如果 GNU `make` 尚未重启，`MAKE_RESTARTS` 是空白，而不是 `0`。

#### 新函数

GNU `make` 3.81 还引入了多种内建函数：

+   ****`$(info` *`text`*`)`****。这个函数类似于现有的 `$(warning)` 函数，但它将展开后的 *`text`* 参数打印到 `STDOUT`，而不报告 makefile 和行号。例如，以下 makefile 会生成 `Hello, World!` 输出：

    ```
    $(info Hello, World!)
    all: ; @true
    ```

+   ****`$(lastword` *`LIST`*`)`****。该函数返回 GNU `make` 列表中的最后一个单词。之前可以通过写 `$(word $(words` *`LIST`*`),`*`LIST`*`)` 来实现，但 `$(lastword)` 更加高效。如果你使用 GNU Make Standard Library（GMSL），有一个名为 `last` 的函数，它与 `$(lastword)` 相同。如果你使用 GNU `make` 3.81 和 GMSL 1.0.6 或更高版本，`last` 会自动使用内建的 `lastword` 来提高速度。

+   ****`$(flavor` *`VAR`*`)`****。该函数返回变量的类型（如果是递归展开，则为 `recursive`；如果是简单展开，则为 `simple`）。例如，以下 makefile 会输出 `REC` 是递归的，`SIM` 是简单的：

    ```
    REC = foo
    SIM := foo
    $(info REC is $(flavor REC))
    $(info SIM is $(flavor SIM))

    all: ; @true
    ```

+   ****`$(or` *`arg1 arg2`* `...) 和 $(and)`****。`$(or)` 如果其任何一个参数非空，则返回非空字符串，而 `$(and)` 只有在所有参数都非空时才返回非空字符串。如果你使用 GMSL，`and` 和 `or` 函数是库的一部分。如果你使用 GNU `make` 3.81 和 GMSL 1.0.6 或更高版本，新的内建函数*不会*被 GMSL 版本覆盖，这意味着使用 GMSL 的 makefile 与 GNU `make` 3.81 版本完全向后和向前兼容。

+   ****`$(abspath DIR)`****。该函数返回相对于 GNU `make` 启动目录的 `DIR` 的绝对路径（考虑到任何 `-C` 命令行选项）。路径会解析所有的 `.` 和 `..` 元素，并删除重复的斜杠。请注意，GNU `make` 并不会检查路径是否*存在*；它只会解析路径元素以生成绝对路径。例如，以下 makefile 在我的机器上放在 */home/jgc* 中时，会输出 `/home/jgc/bar`：

    ```
    $(info $(abspath foo/./..//////bar))

    all: ; @true
    ```

+   ****`$(realpath DIR)`****。该函数返回与 `$(abspath DIR)` 相同的结果，除了会解析任何符号链接。例如，如果 `bar` 是指向 `over-here` 的符号链接，以下 makefile 会从 */home/jgc* 读取时返回 `/home/jgc/ over-here`：

    ```
    $(info $(realpath ../jgc/./bar))

    all: ; @true
    ```

## GNU make 3.82 中的新变化

GNU `make` 3.82 在 3.81 发布四年后推出，引入了许多新特性——以及一些向后不兼容的变化。

### 向后不兼容性

GNU `make` 3.82 的*NEWS*文件以七个向后不兼容的警告开始。以下是快速概述：

+   在 GNU `make`中，执行规则命令的 shell 是通过`-c`命令行选项调用的，该选项告诉 shell 从第一个非参数参数开始读取要执行的命令。例如，当执行以下小规则时，`make`实际上执行的是`execve("/bin/sh", ["/bin/sh", "-c", "echo \"hello\""], ...)`。要运行`echo "hello"`，`make`使用 shell`/bin/sh`并为其添加`-c`命令行选项。

    ```
    all: ; @echo "hello"
    ```

    但是，POSIX 标准在 2008 年修改了`make`的规定，要求必须在 shell 命令行中指定`-e`。GNU `make` 3.82 及更高版本的默认行为是不传递`-e`，除非指定了`.POSIX`特殊目标。任何在 makefile 中使用此目标的人需要注意这一变化。

+   `$?`自动变量包括所有导致重新构建的前提条件的名称，*即使它们不存在*。之前，任何不存在的前提条件不会被放入`$?`。

+   `$(wildcard)`函数一直返回一个已排序的文件列表，但这从未实际文档化。这个行为在 GNU `make` 3.82 中发生了变化，因此任何依赖于`$(wildcard)`的已排序列表的 makefile 都需要将其包裹在`$(sort)`的调用中；例如，执行`$(sort $(wildcard *.c))`以获取已排序的`.c`文件列表。

+   以前可以编写一个混合模式目标和显式目标的规则，像这样：

    ```
    myfile.out %.out: ; @echo Do stuff with $@
    ```

    这一直没有文档说明，并且在 GNU `make` 3.81 中被完全移除，因为这从未打算如此工作。现在它会导致错误信息。

+   不再可能有一个包含`=`符号的前提条件，即使使用`\`进行转义。例如，下面的写法不再有效：

    ```
    all: odd\=name

    odd%: ; @echo Make $@
    ```

    如果在目标或前提条件名称中需要一个等号，首先定义一个展开为`=`的变量，如下所示：

    ```
    eq := =

    all: odd$(eq)name
    odd%: ; @echo Make $@
    ```

+   在 GNU `make` 3.82 中，变量名不能包含空格。之前是可以这样做的：

    ```
    has space := variable with space in name
    $(info $(has space))
    ```

    如果需要一个包含空格的变量名，首先定义另一个只包含空格的变量，并按照以下方式使用它。但请注意，这种做法可能是危险的，且难以调试。

    ```
    sp :=
    sp +=
    has$(sp)space := variable with space in name

    $(info $(has space))
    ```

+   模式规则和模式特定变量应用的顺序曾经是按它们在 makefile 中出现的顺序。这个顺序在 GNU `make` 3.82 中发生了变化：它们现在按照“最短 stem”顺序应用。例如，下面的 makefile 展示了 GNU `make` 3.81 和 3.82 中不同模式规则的使用方法。

    ```
    all: output.o

    out%.o: ; @echo Using out%.o rule
    outp%.o: ; @echo Using outp%.o rule
    ```

    *stem*是模式中由`%`匹配的部分。在 GNU `make` 3.81 及更早版本中，`out%.o`规则可以匹配，因为它是首先定义的：

    ```
    $ make-3.81
    Using out%.o rule
    ```

    在 GNU `make` 3.82 及更高版本中，使用 `outp%.o` 规则，因为该规则的模板更短：

    ```
    $ make-3.82
    Using outp%.o rule
    ```

    对模式特定变量也会出现类似的行为。

### 新的命令行选项：`--eval`

新的 `--eval` 命令行选项会使 `make` 在解析 Makefile 之前，将其参数通过 `$(eval)` 进行处理。例如，如果你有这个 Makefile，并运行 `make --eval=FOO=bar`，你将看到输出 `FOO has value bar`。

```
all: ; @echo FOO has value $(FOO)
```

这是因为在解析 Makefile 之前，`FOO=bar` 这一行会被当作 Makefile 的第一行，并将 `FOO` 设置为 `bar`。

### 新的特殊变量：`.RECIPEPREFIX` 和 `.SHELLFLAGS`

GNU `make` 3.82 引入了两个新的特殊变量：

+   ****`.RECIPEPREFIX`****。GNU `make` 使用 `TAB` 字符作为规则中命令的有效空白字符。你可以通过 `.RECIPEPREFIX` 变量来更改此设置。（如果 `.RECIPEPREFIX` 是空字符串，则使用 `TAB`）。例如：

    ```
    .RECIPEPREFIX = >

    all:
    > @echo Making all
    ```

    此外，`.RECIPEPREFIX` 可以根据需要在 Makefile 中反复更改。

+   ****`.SHELLFLAGS`****。该变量包含在规则的命令运行时传递给 shell 的参数。默认情况下，它是 `-c`（如果在 Makefile 中指定了 `.POSIX:`，则为 `-ec`）。如果使用不同的 shell，可以读取或更改此值。

### `.ONESHELL` 目标

当规则的命令执行时，每行都会作为一个单独的 shell 调用发送到 shell 中。在 GNU `make` 3.82 中，引入了一个新的特殊目标 `.ONESHELL` 来改变这种行为。如果在 Makefile 中设置了 `.ONESHELL:`，则所有规则中的行将在同一个 shell 调用中执行。例如：

```
all:
→ @cd /tmp
→ @pwd
```

这不会输出 `/tmp`（除非 `make` 是在 */tmp* 目录下启动的），因为每行命令都会在单独的 shell 中执行。但使用 `.ONESHELL` 特殊目标时，两行命令会在同一个 shell 中执行，`pwd` 会输出 `/tmp`。

```
.ONESHELL:
all:
→ @cd /tmp
→ @pwd
```

### 使用 `private` 和 `undefine` 关键字来更改变量

目标特定变量通常为目标及其所有前提条件定义。但如果目标特定变量以 `private` 关键字为前缀，则该变量仅为该目标定义，*而不是*其前提条件。

在以下 Makefile 中，`DEBUG` 只在 `foo.o` 目标上设置为 `1`，因为它被标记为 `private:`。

```
DEBUG=0

foo.o: private DEBUG=1
foo.o: foo.c
→ @echo DEBUG is $(DEBUG) for $@

foo.c: foo.in
→ @echo DEBUG is $(DEBUG) for $@
```

GNU `make` 3.82 中的另一个新关键字是 `undefine`，它使得可以取消定义一个变量：

```
SPECIAL_FLAGS := xyz
$(info SPECIAL_FLAGS $(SPECIAL_FLAGS))
undefine SPECIAL_FLAGS
$(info SPECIAL_FLAGS $(SPECIAL_FLAGS))
```

你可以使用 `$(flavor)` 函数来检测空变量与未定义变量之间的区别。例如，以下输出 `simple`，然后输出 `undefined`：

```
EMPTY :=
$(info $(flavor EMPTY))
undefine EMPTY
$(info $(flavor EMPTY))
```

在 GNU `make` 3.82 之前的版本中，`define` 指令（用于定义多行变量）总是会创建一个递归定义的变量。例如，这里的 `COMMANDS` 将是一个递归变量，每次使用时都会展开：

```
FILE = foo.c

define COMMANDS
wc -l $(FILE)
shasum $(FILE)
endef
```

在 GNU 3.82 中，可以在 `define` 语句中的变量名后添加可选的 `=`、`:=` 或 `+=`。默认行为是每次都递归展开新变量；这与添加 `=` 相同。添加 `:=` 会创建一个简单变量，在定义时展开 `define` 的主体。添加 `+=` 会将多行追加到现有变量中。

以下 makefile 创建了一个名为 `COMMANDS` 的简单变量，然后向其中添加行：

```
   FILE = foo.c

   define COMMANDS :=
   wc -l $(FILE)
   shasum $(FILE)
   endef

   define COMMANDS +=
➊
   wc -c $(FILE)
   endef
   $(info $(COMMANDS))
```

注意➊处的额外空行。这是必要的，因为 `wc -c $(FILE)` 必须在 `shasum $(FILE)` 之后新的一行显示。如果没有它，`wc -c $(FILE)` 会被追加到 `shasum $(FILE)` 后面，并且没有换行符。

## GNU make 4.0 的新特性

GNU `make` 4.0 的发布引入了两个主要特性：与 GNU Guile 语言的集成，以及一个实验性选项，允许动态加载对象以在运行时扩展 `make` 的功能。此外，新的命令行选项对于调试特别有帮助。

### GNU Guile

GNU `make` 4.0 中最大的变化是新的 `$(guile)` 函数，其参数是用 GNU Guile 语言编写的代码。该代码被执行，并将返回值转换为字符串，该字符串将由 `$(guile)` 函数返回。

能够切换到另一种语言为 GNU `make` 添加了巨大的功能。以下是一个简单示例，使用 Guile 检查文件是否存在：

```
$(if $(guile (access? "foo.c" R_OK)),$(info foo.c exists))
```

使用 GNU Guile 内嵌在 GNU `make` 中的内容将在 第五章 中进一步详细介绍。

### 加载动态对象

本书中我们没有使用 `load` 操作符来定义 C 函数，但在 第五章 中解释了如何在 C 中定义函数和加载动态对象。

### 使用 `--output-sync` 同步输出

如果你使用递归的`make`或者使用作业服务器并行运行规则，`make`产生的输出可能会很难阅读，因为来自不同规则和子 `make` 的输出会交织在一起。

请考虑以下（稍微做过修改的）makefile：

```
all: one two three four

one two:
→ @echo $@ line start
→ @sleep 0.1s
→ @echo $@ line middle
→ @echo $@ line finish

three four:
→ @echo $@ line start
→ @sleep 0.2s
→ @echo $@ line middle
→ @echo $@ line finish
```

这个 makefile 包含四个目标：`one`、`two`、`three` 和 `four`。如果使用 `-j` 选项，目标将并行构建。为了模拟不同执行时间的命令，添加了两次 `sleep` 调用。

当使用 `-j4` 选项运行时，它会并行运行四个作业，输出可能如下所示：

```
$ **make -j4**
one line start
three line start
four line start
two line start
one line middle
two line middle
one line finish
two line finish
four line middle
three line middle
three line finish
four line finish
```

每个规则的输出行会混合在一起，使得很难辨别哪个输出属于哪个规则。指定 `-Otarget`（或 `--output-sync=target`）会使 `make` 跟踪哪些输出与哪个目标相关联，并且只在规则完成时刷新输出。现在每个目标的完整输出清晰可读：

```
$ **make -j4 -Otarget**
two line start
two line middle
two line finish
one line start
one line middle
one line finish
four line start
four line middle
four line finish
three line start
three line middle
three line finish
```

指定`--output-sync=recurse`可以处理递归子 make——即调用`$(MAKE)`的规则——通过缓存规则的所有输出，包括子 make 的输出，并一次性输出所有内容。这可以防止子 make 输出混合在一起，但可能导致`make`的输出出现长时间的暂停。

### --trace 命令行选项

你可以使用新的`--trace`选项来追踪 makefile 中规则的执行情况。当在`make`命令行中指定时，执行的每条规则的命令会与该规则的定义位置及其执行原因一起打印出来。

例如，这个简单的 makefile 有四个目标：

```
all: part-one part-two

part-one: part-three
→ @echo Make $@

part-two:
→ @echo Make $@

part-three:
→ @echo Make $@
```

使用`--trace`运行它：

```
$ **make --trace**
makefile:10: target 'part-three' does not exist
echo Make part-three
Make part-three
makefile:4: update target 'part-one' due to: part-three
echo Make part-one
Make part-one
makefile:7: target 'part-two' does not exist
echo Make part-two
Make part-two
```

这会显示每条规则为何被执行，它在 makefile 中的位置，以及执行了哪些命令。

### 新的赋值运算符：`!=`和`::=`

你可以使用`!=`运算符执行一个 shell 命令，并将命令的输出设置为变量，这与`$(shell)`类似。例如，下面的代码行使用`!=`获取当前的日期和时间并存入变量：

```
CURRENTLY != date
```

使用`!=`时需要注意一个重要的细节：结果变量是递归的，因此每次使用变量时它的值都会被展开。如果执行的命令（即`!=`的右侧部分）返回了`$`，`make`会将其解释为变量引用并展开。因此，最好使用`$(shell)`与`:=`，而不是使用`!=`。（这是为了兼容 BSD `make`，也可能会被添加到 POSIX 中。）

`::=`运算符与`:=`完全相同，且是为了 POSIX 兼容性而添加的。

### `$(file)`函数

你可以使用新的`$(file)`函数来创建或追加到一个文件。以下 makefile 使用`$(file)`在每次执行规则时创建一个文件并追加内容。它记录了 makefile 的执行日志：

```
LOG = make.log

$(file > $(LOG),Start)

all: part-one part-two

part-one: part-three
→ @$(file >> $(LOG),$@)
→ @echo Make $@

part-two:
→ @$(file >> $(LOG),$@)
→ @echo Make $@

part-three:
→ @$(file >> $(LOG),$@)
→ @echo Make $@
```

第一个`$(file)`使用`>`操作符创建日志文件，随后的`$(file)`调用使用`>>`将内容追加到日志中：

```
$ **make**
Make part-three
Make part-one
Make part-two
$ **cat make.log**
Start
part-three
part-one
part-two
```

很容易看出，`$(file)`函数是 GNU `make`的一个有用扩展。

## GNU make 4.1 的新特性

当前版本的 GNU `make`（在本文写作时）是 4.1 版本。该版本于 2014 年 10 月 5 日发布，包含了两个有用的改动以及大量的错误修复和小幅改进。

新增了`MAKE_TERMOUT`和`MAKE_TERMERR`变量。如果`make`认为`stdout`和`stderr`（分别）被发送到控制台，则这两个布尔值会被设置为 true（即它们不是空的）。

`$(file)`函数已被修改，可以打开一个文件而不往其中写入任何内容。如果没有提供文本参数，文件会被简单地打开然后关闭；你可以用这个方法通过`$(file > $(MY_FILE))`创建一个空文件。
