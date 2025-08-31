# 第五章 推动极限

在本章中，你会发现一些通常不需要的技巧，但有时它们会非常有用。例如，有时通过在 C 语言或甚至 Guile 中创建新函数来扩展 GNU `make` 的语言是非常有用的。本章将展示如何做到这一点以及更多内容。

# 做算术

GNU `make` 没有内建的算术功能。但我们可以为整数的加法、减法、乘法和除法创建函数。还可以为整数比较（例如大于、不等于）创建函数。这些函数完全通过 GNU `make` 的内建列表和字符串处理函数实现：`$(subst)`、`$(filter)`、`$(filter-out)`、`$(words)`、`$(wordlist)`、`$(call)`、`$(foreach)` 和 `$(if)`。在定义完我们的算术函数后，我们将实现一个简单的计算器。

要创建一个算术库，我们首先需要一个数字的表示法。表示一个数字的简单方法是使用包含该数量项的列表。例如，对于算术库，数字就是由字母 `x` 组成的列表。因此，数字 5 表示为 `x x x x x`。

给定这种表示法，我们可以使用 `$(words)` 函数将内部形式（所有 `x`）转换为人类可读的形式。例如，以下代码将输出 5：

```
five := x x x x x

all: ; @echo $(words $(five))
```

让我们创建一个用户自定义函数 `decode`，将 `x` 表示法转换为数字：

```
decode = $(words $1)
```

在 Makefile 中使用 `decode`，我们需要使用 GNU `make` 的 `$(call)` 函数，它可以通过一组参数调用用户自定义的函数：

```
five := x x x x x

all: ; @echo $(call decode,$(five))
```

参数将存储在名为 `$1`、`$2`、`$3` 等临时变量中。在 `decode` 中，它接受一个参数——即要解码的数字——我们只需使用 `$1`。

## 加法与减法

现在我们已经有了表示法，可以定义加法、增量（加 1）和减量（减 1）的函数：

```
plus = $1 $2
increment = x $1
decrement = $(wordlist 2,$(words $1),$1)
```

`plus` 函数将其两个参数组成一个列表；通过连接字符串就能实现带有 `x` 表示法的加法运算。`increment` 函数向其参数中添加一个单独的 `x`。`decrement` 函数通过从索引 2 开始请求整个由 `x` 组成的字符串，去除其参数中的第一个 `x`。例如，以下代码将输出 11：

```
two := x x
three := x x x
four := x x x x
five := x x x x x
six := x x x x x x

all: ; @echo $(call decode,$(call plus,$(five),$(six)))
```

注意在 `decode` 调用中的 `plus` 函数的嵌套调用，以便我们输出数字 11，而不是由 11 个 `x` 组成的列表。

我们可以创建另一个简单的函数 `double`，它将参数翻倍：

```
double = $1 $1
```

实现减法比加法更具挑战性。但在我们进行减法实现之前，先来实现 `max` 和 `min` 函数：

```
max = $(subst xx,x,$(join $1,$2))
min = $(subst xx,x,$(filter xx,$(join $1,$2)))
```

`max` 函数使用了两个 GNU `make` 内建函数：`$(join)` 和 `$(subst)`。`$(join LIST1,LIST2)` 将两个列表作为参数，按顺序连接两个列表：将 `LIST1` 的第一个元素与 `LIST2` 的第一个元素连接，依此类推。如果一个列表比另一个长，剩余的项将直接附加到列表的末尾。

`$(subst FROM,TO,LIST)`遍历一个列表，并将匹配`FROM`模式的元素替换为`TO`值。为了了解`max`如何工作，考虑在计算`$(call max,$(five),$(six))`时发生的事件顺序：

```
$(call max,$(five),$(six))
→  $(call max,x x x x x,x x x x x x)
→  $(subst xx,x,$(join x x x x x,x x x x x x))
→  $(subst xx,x,xx xx xx xx xx x)
→  x x x x x x
```

首先，`$(join)`将包含五个`x`的列表与包含六个`x`的列表连接，结果是一个包含六个元素的列表，其中前五个是`xx`。然后，`$(subst)`将前五个`xx`转换为`x`。最终结果是六个`x`，这是最大值。

为了实现`min`，我们使用类似的技巧，但我们只保留`xx`并丢弃`x`：

```
$(call min,$(five),$(six))
→  $(call min,x x x x x,x x x x x x)
→  $(subst xx,x,$(filter xx,$(join x x x x x,x x x x x x)))
→  $(subst xx,x,$(filter xx,xx xx xx xx xx x))
→  $(subst xx,x,xx xx xx xx xx)
→  x x x x x
```

`xx`表示两个列表可能连接的位置。较短的列表只会包含`xx`。`$(filter PATTERN,LIST)`函数会遍历列表并移除与模式不匹配的元素。

类似的模式适用于减法操作：

```
subtract = $(if $(call gte,$1,$2),          \
       $(filter-out xx,$(join $1,$2)),      \
       $(warning Subtraction underflow))
```

暂时忽略定义中的`$(warning)`和`$(if)`部分，专注于`$(filter-out)`。`$(filter-out)`是`$(filter)`的反操作：它从列表中移除与模式匹配的元素。例如，我们可以看到这里的`$(filter-out)`实现了减法操作：

```
$(filter-out xx,$(join $(six),$(five)))
→  $(filter-out xx,$(join x x x x x x,x x x x x))
→  $(filter-out xx,xx xx xx xx xx x)
→  x
```

不幸的是，如果将五和六的位置反转，这种方法也会奏效，因此我们首先需要检查第一个参数是否大于或等于第二个参数。在`subtract`定义中，特殊函数`gte`（*大于或等于*）将在第一个参数大于第二个参数时返回一个非空字符串。我们使用`gte`来决定是否进行减法操作或使用`$(warning)`输出警告消息。

`gte`函数是通过两个其他函数来实现的，分别用于*大于*（`gt`）和*等于*（`eq`）：

```
gt = $(filter-out $(words $2),$(words $(call max,$1,$2)))
eq = $(filter $(words $1),$(words $2))
gte = $(call gt,$1,$2)$(call eq,$1,$2)
```

如果`gt`或`eq`返回非空字符串，`gte`将返回一个非空字符串。

`eq`函数有点让人费解。它计算其两个参数中元素的数量，将一个参数视为模式，另一个作为列表，并使用`$(filter)`来判断它们是否相同。以下是一个它们相等的例子：

```
$(call eq,$(five),$(five))
→  $(call eq,x x x x x,x x x x x)
→  $(filter $(words x x x x x),$(words x x x x x))
→  $(filter 5,5)
→  5
```

`eq`函数将两个`$(five)`都转换为由五个`x`组成的列表。然后，这些列表都通过`$(words)`转换为数字 5。将这两个 5 传入`$(filter)`。由于`$(filter)`的两个参数相同，结果是 5，而 5 不是空字符串，因此它被解释为*真*。

当它们不相等时，发生了以下情况：

```
$(call eq,$(five),$(six))
→  $(call eq,x x x x x,x x x x x x)
→  $(filter $(words x x x x x),$(words x x x x x x))
→  $(filter 5,6)
```

这与`$(call eq,$(five),$(five))`的过程类似，只不过用`$(six)`替换了其中一个`$(five)`。由于`$(filter 5,6)`是一个空字符串，结果为假。

因此，`$(filter)`函数充当了一种字符串相等运算符；在我们的例子中，两个字符串分别是两个数字字符串的长度。`gt`函数的实现方式类似：如果第一个数字字符串的长度不等于两个数字字符串中的最大值，它返回一个非空字符串。下面是一个例子：

```
$(call gt,$(six),$(five))
→  $(call gt,x x x x x x,x x x x x)
→  $(filter-out $(words x x x x x),
   $(words $(call max,x x x x x x,x x x x x)))
→  $(filter-out $(words x x x x x),$(words x x x x x x))
→  $(filter-out 5,6)
→  6
```

`gt` 函数的工作方式与 `eq`（前面描述的）类似，但使用 `$(filter-out)` 而不是 `$(filter)`。它将两个 `x` 表示的数字转换为数字，但使用 `$(filter-out)` 比较它们中的第一个与两者的最大值。当第一个数字大于第二个时，两个不同的数字会传递给 `$(filter-out)`。由于它们不同，`$(filter-out)` 会返回一个非空字符串，表示真。

这里有一个例子，其中第一个数字小于第二个：

```
$(call gt,$(five),$(six))
→  $(call gt,x x x x x,x x x x x x)
→  $(filter-out $(words x x x x x x),
   $(words $(call max,x x x x x x,x x x x x)))
→  $(filter-out $(words x x x x x x),$(words x x x x x x))
→  $(filter-out 6,6)
```

这里，因为两个数字的 `max` 与第二个数字相同（因为它是最大的），所以 `$(filter-out)` 被传入相同的数字并返回一个空字符串，表示假。

类似地，我们可以定义 *不等于* (`ne`)，*小于* (`lt`)，和 *小于或等于* (`lte`) 操作符：

```
lt = $(filter-out $(words $1),$(words $(call max,$1,$2)))
ne = $(filter-out $(words $1),$(words $2))
lte = $(call lt,$1,$2)$(call eq,$1,$2)
```

`lte` 是通过 `lt` 和 `eq` 定义的。因为非空字符串意味着 *真*，所以 `lte` 只会将 `lt` 和 `eq` 返回的值连接起来；如果其中任何一个返回真，那么 `lte` 就返回真。

## 乘法和除法

在我们定义了另外三个函数：`multiply`、`divide` 和 `encode` 后，我们将拥有一个非常强大的算术包。`encode` 是将一个整数转化为一串 `x` 字符的方式；我们将把它留到最后，并实现我们的计算器。

乘法使用 `$(foreach VAR,LIST,DO)` 函数。它将名为 `VAR` 的变量设置为 `LIST` 中的每个元素，并执行 `DO` 中指定的操作。因此，乘法很容易实现：

```
multiply = $(foreach a,$1,$2)
```

`multiply` 将其第二个参数与第一个参数中有多少个 `x` 字符拼接在一起。例如：

```
$(call multiply,$(two),$(three))
→  $(call multiply,x x,x x x)
→  $(foreach a,x x,x x x)
→  x x x x x x
```

`divide` 是其中最复杂的函数，因为它需要递归：

```
divide = $(if $(call gte,$1,$2),             \
    x $(call divide,$(call subtract,$1,$2),$2),)
```

如果第一个参数小于第二个，除法将返回 `0`，因为 `$(if)` 的 `ELSE` 部分是空的（见结尾的 `,)`）。如果可以进行除法，`divide` 会通过从第一个参数中反复减去第二个参数来工作，使用 `subtract` 函数。每次减去时，它会添加一个 `x` 并再次调用 `divide`。以下是一个例子：

```
$(call divide,$(three),$(two))
→  $(call divide,x x x,x x)
→  $(if $(call gte,x x x,x x),
   x $(call divide,$(call subtract,x x x,x x),x x),)

→  x $(call divide,$(call subtract,x x x,x x),x x)
→  x $(call divide,x,x x)
→  x $(if $(call gte,x,x x),
   x $(call divide,$(call subtract,x,x x),x x),)

→  x
```

首先，`gte` 返回一个非空字符串，因此会发生递归。接下来，`gte` 返回一个空字符串，因此不会再发生递归。

我们可以通过在除以 2 的特殊情况下避免递归；我们定义 `halve` 函数，它是 `double` 的反操作：

```
halve = $(subst xx,x,      \
   $(filter-out xy x y,    \
     $(join $1,$(foreach a,$1,y x))))
```

到现在为止，你已经看过 `halve` 中使用的所有函数。通过一个例子，假设 `$(call halve,$(five))`，来查看它是如何工作的。

唯一需要注意的事情是将用户输入的数字转换成一串 `x` 字符。`encode` 函数通过从预定义的 `x` 字符串中删除一个子串来完成这项工作：

```
16 := x x x x x x x x x x x x x x x x
input_int := $(foreach a,$(16),      \
       $(foreach b,$(16),            \
        $(foreach c,$(16),$(16)))))

encode = $(wordlist 1,$1,$(input_int))
```

在这里，我们限制了最多输入到 65536。我们可以通过改变 `input_int` 中的 `x` 数量来解决这个问题。一旦我们得到了编码中的数字，只有可用的内存限制了我们可以处理的整数大小。

## 使用我们的算术库：一个计算器

为了真正展示这个库，下面是一个完全用 GNU `make` 函数编写的逆波兰表示法计算器实现：

```
stack :=

push = $(eval stack := $$1 $(stack))
pop = $(word 1,$(stack))$(eval stack := $(wordlist 2,$(words $(stack)),$(stack)))
pope = $(call encode,$(call pop))
pushd = $(call push,$(call decode,$1))
comma := ,
calculate = $(foreach t,$(subst $(comma), ,$1),$(call handle,$t))$(stack)
seq = $(filter $1,$2)
handle = $(call pushd,                            \
    $(if $(call seq,+,$1),                        \
      $(call plus,$(call pope),$(call pope)),     \
      $(if $(call seq,-,$1),                      \
      $(call subtract,$(call pope),$(call pope)), \
        $(if $(call seq,*,$1),                    \
     $(call multiply,$(call pope),$(call pope)),  \
        $(if $(call seq,/,$1),                    \
       $(call divide,$(call pope),$(call pope)),  \
           $(call encode,$1))))))

.PHONY: calc
calc: ; @echo $(call calculate,$(calc))
```

操作符和数字被传递到 GNU `make` 中的 `calc` 变量中，且通过逗号分隔。例如：

```
$ **make calc="3,1,-,3,21,5,*,+,/"**
54
```

显然，这不是 GNU `make` 的设计初衷，但它展示了 GNU `make` 函数的强大功能。下面是完整的注释版 makefile：

```
# input_int consists of 65536 x's built from the 16 x's in 16

16 := x x x x x x x x x x x x x x x x
input_int := $(foreach a,$(16),$(foreach b,$(16),$(foreach c,$(16),$(16)))))

# decode turns a number in x's representation into an integer for human
# consumption

decode = $(words $1)

# encode takes an integer and returns the appropriate x's
# representation of the number by chopping $1 x's from the start of
# input_int

encode = $(wordlist 1,$1,$(input_int))

# plus adds its two arguments, subtract subtracts its second argument
# from its first if and only if this would not result in a negative result

plus = $1 $2

subtract = $(if $(call gte,$1,$2),     \
       $(filter-out xx,$(join $1,$2)), \
       $(warning Subtraction underflow))

# multiply multiplies its two arguments and divide divides its first
# argument by its second

multiply = $(foreach a,$1,$2)
divide = $(if $(call gte,$1,$2),x $(call divide,$(call subtract,$1,$2),$2),)

# max returns the maximum of its arguments and min the minimum

max = $(subst xx,x,$(join $1,$2))
min = $(subst xx,x,$(filter xx,$(join $1,$2)))

# The following operators return a non-empty string if their result is true:
#
# gt First argument is greater than second argument
# gte First argument is greater than or equal to second argument
# lt First argument is less than second argument

# lte First argument is less than or equal to second argument
# eq First argument is numerically equal to the second argument
# ne First argument is not numerically equal to the second argument

gt = $(filter-out $(words $2),$(words $(call max,$1,$2)))
lt = $(filter-out $(words $1),$(words $(call max,$1,$2)))
eq = $(filter $(words $1),$(words $2))
ne = $(filter-out $(words $1),$(words $2))
gte = $(call gt,$1,$2)$(call eq,$1,$2)
lte = $(call lt,$1,$2)$(call eq,$1,$2)

# increment adds 1 to its argument, decrement subtracts 1\. Note that
# decrement does not range check and hence will not underflow, but
# will incorrectly say that 0 - 1 = 0

increment = $1 x
decrement = $(wordlist 2,$(words $1),$1)

# double doubles its argument, and halve halves it

double = $1 $1
halve = $(subst xx,x,$(filter-out xy x y,$(join $1,$(foreach a,$1,y x))))

# This code implements a Reverse Polish Notation calculator by
# transforming a comma-separated list of operators (+ - * /) and
# numbers stored in the calc variable into the appropriate calls to
# the arithmetic functions defined in this makefile.

# This is the current stack of numbers entered into the calculator. The push
# function puts an item onto the top of the stack (the start of the list), and
# pop removes the top item.

stack :=

push = $(eval stack := $$1 $(stack))
pop = $(word 1,$(stack))$(eval stack := $(wordlist 2,$(words $(stack)),$(stack)))

# pope pops a number off the stack and encodes it
# and pushd pushes a number onto the stack after decoding

pope = $(call encode,$(call pop))
pushd = $(call push,$(call decode,$1))

# calculate runs through the input numbers and operations and either
# pushes a number on the stack or pops two numbers off and does a
# calculation followed by pushing the result back. When calculate is
# finished, there will be one item on the stack, which is the result.

comma := ,
calculate=$(foreach t,$(subst $(comma), ,$1),$(call handle,$t))$(stack)

# seq is a string equality operator that returns true (a non-empty
# string) if the two strings are equal

seq = $(filter $1,$2)

# handle is used by calculate to handle a single token. If it's an
# operator, the appropriate operator function is called; if it's a
# number, it is pushed.

handle = $(call pushd,                            \
      $(if $(call seq,+,$1),                      \
        $(call plus,$(call pope),$(call pope)),   \
        $(if $(call seq,-,$1),                    \
      $(call subtract,$(call pope),$(call pope)), \
          $(if $(call seq,*,$1),                  \
      $(call multiply,$(call pope),$(call pope)), \
            $(if $(call seq,/,$1),                \
       $(call divide,$(call pope),$(call pope)),  \
              $(call encode,$1))))))

.PHONY: calc
calc: ; @echo $(call calculate,$(calc))
```

你将在第六章中更详细地了解这些技术，当你学习 GNU Make 标准库时。

# 制作 XML 材料清单

使用标准的 GNU `make` 输出，很难回答构建了什么以及为什么。这一节介绍了一种简单的技术，可以让 GNU `make` 创建一个包含*材料清单（BOM）*的 XML 文件。BOM 包含了 makefile 构建的所有文件的名称，并且通过嵌套结构显示每个文件的先决条件。

## 示例 Makefile 和 BOM

示例 5-1 展示了一个示例 makefile。我们将查看其 BOM，然后反向追溯，以了解 BOM JSON 文件是如何生成的。

示例 5-1. 一个简单的 makefile 用来说明 BOM

```
all: foo bar
→  @echo Making $@

foo: baz
→  @echo Making $@

bar:
→  @echo Making $@

baz:
→  @echo Making $@
```

这个示例生成 `all`，由 `foo` 和 `bar` 构成。反过来，`foo` 是由 `baz` 构成的。在 GNU `make` 中运行这段代码，会产生如下输出：

```
$ **make**
Making baz
Making foo
Making bar
Making all
```

从输出中，无法识别构建的树形顺序或哪些文件依赖于哪些文件。在这种情况下，makefile 很小，手动追踪相对容易；但在实际的 makefile 中，手动追踪几乎是不可能的。

生成如示例 5-2 中所示的输出是很好的，它展示了构建了什么以及为什么：

示例 5-2. 一个展示示例 makefile 结构的 XML 文档

```
<rule target="all">
<prereq>
 <rule target="foo">
  <prereq>
   <rule target="baz" />
  </prereq>
 </rule>
 <rule target="bar" />
</prereq>
</rule>
```

在这里，每个由 makefile 执行的规则都添加了一个 `<rule>` 标签，并通过 `target` 属性给出了该规则构建的目标名称。如果规则有任何先决条件，在 `<rule>`/`</rule>` 标签对中，会用 `<prereq>`/`</prereq>` 标签包围一个先决条件规则的列表。

你可以看到 makefile 的结构通过标签的嵌套反映出来。将 XML 文档加载到 XML 编辑器中（或者直接加载到网页浏览器）可以让你自由地展开和收缩树形结构，以探索 makefile 的结构。

## 工作原理

要创建如示例 5-2 中所示的输出，示例 makefile 被修改，加入了一个使用标准 `include bom` 方法的特殊 `bom` makefile。加入后，我们可以通过运行 GNU `make` 使用命令行，如 `make bom-all` 来生成 XML 输出。

`bom-all`指示 GNU `make`从`all`目标开始构建 BOM。就像你输入了`make all`，但现在将创建一个 XML 文档。

默认情况下，XML 文档的名称与 makefile 相同，但附加`.xml`。如果示例 makefile 是`example.mk`，则创建的 XML 文档将被命名为`example.mk.xml`。

示例 5-3 显示了包含`bom` makefile 的内容：

示例 5-3. 创建 XML BOM 的`bom` makefile

```
➊ PARENT_MAKEFILE := $(word $(words $(MAKEFILE_LIST)),x $(MAKEFILE_LIST))
➋ bom-file := $(PARENT_MAKEFILE).xml

➌ bom-old-shell := $(SHELL)
➍ SHELL = $(bom-run)$(bom-old-shell)

   bom-%: %
➎ → @$(shell rm -f $(bom-file))$(call bom-dump,$*)
   bom-write = $(shell echo '$1' >> $(bom-file))
➏ bom-dump = $(if $(bom-prereq-$1),$(call bom-write,<rule target="$1">)      \
   $(call bom-write,<prereq>)$(foreach p,$(bom-prereq-$1),                    \
   $(call bom-dump,$p))$(call bom-write,</prereq>)$(call bom-write,</rule>),  \
   $(call bom-write,<rule target="$1" />))

➐ bom-run = $(if $@,$(eval bom-prereq-$@ := $^))
```

首先，我们通过提取包含`bom`的 makefile 的名称到`PARENT_MAKEFILE` ➊，将`.xml`附加到该名称上，并将结果存储在`bom-file` ➋中，从而确定 XML 文件的正确名称。

然后我们使用本书中多次出现的一个技巧：`SHELL`黑客。GNU `make`将在执行 makefile 中的每条规则时展开`$(SHELL)`的值。当`$(SHELL)`被展开时，每条规则的自动变量（如`$@`）已经被设置。因此，通过修改`SHELL`，我们可以在每条规则执行时为 makefile 中的每个规则执行某些任务。

在➌处，我们使用立即赋值（`:=`）将`SHELL`的原始值存储在`bom-old-shell`中，然后在➍处重新定义`SHELL`为`$(bom-run)`的展开值和原始 shell。因为`$(bom-run)`实际上展开为空字符串，所以其效果是`bom-run`在 makefile 的每条规则中展开，但实际使用的 shell 不受影响。

`bom-run`在➐处定义。它使用`$(eval)`存储当前正在构建的目标（`$(if)`确保`$@`已定义）与其前提条件之间的关系。例如，当构建`foo`时，将调用`bom-run`，并将`$@`设置为`foo`，`$^`（所有前提条件的列表）设置为`baz`。`bom-run`将`bom-prereq-foo`的值设置为`baz`。稍后，这些`bom-prereq-X`变量的值将用于打印 XML 树。

在➎处，我们定义了处理`bom-%`目标的模式规则。由于`bom-%`的前提条件是`%`，因此这个模式规则的效果是构建与`%`匹配的目标，然后构建`bom-%`。在我们的例子中，运行`make bom-all`会与这个模式规则匹配，先构建`all`，然后运行与`bom-%`关联的命令，`$*`设置为`all`。

`bom-%`的命令首先删除`bom-file`，然后从`$*`开始递归地转储 XML。在这个例子中，当用户执行`make bom-all`时，`bom-%`的命令会调用`bom-dump`，并传递参数`all`。

我们在➏处定义了`bom-dump`。它相当常规：它使用一个辅助函数`bom-write`将 XML 片段回显到`bom-file`，并为每个它正在转储的目标的前提条件中的每个目标调用自身。前提条件是从`bom-run`创建的`bom-prereq-X`变量中提取的。

## 注意事项

示例 5-3 中的技术有一些坑。一个坑是，这种技术最终可能会产生大量输出。这是因为它会打印任何目标下的整个树。如果一个目标在树中多次出现，那么即使是小项目，XML 的转储时间也会很长。

作为一种变通方法，我们可以修改 `bom-dump` 的定义，只为每个目标转储一次先决条件信息。这比示例 5-3 中的方法要快得多，并且可以通过像以下这样的脚本来处理，以帮助理解 `make` 的结构：

```
bom-%: %
→  @$(shell rm -f $(bom-file))$(call bom-write,<bom>)$(call bom-dump,$*)$(call bom-write,</bom>)

bom-write = $(shell echo '$1' >> $(bom-file))

bom-dump = $(if $(bom-prereq-$1),$(call bom-write,<rule target="$1">) \
$(call bom-write,<prereq>)$(foreach p,$(bom-prereq-$1),               \
$(call bom-write,<rule target="$p" />))$(call bom-write,</prereq>)    \
$(call bom-write,</rule>),$(call bom-write,<rule target="$1" />))     \
$(foreach p,$(bom-prereq-$1),$(call bom-dump,$p))$(eval bom-prereq-$1 := )
```

在示例 5-1 中的示例 makefile，XML 文档现在如下所示：

```
<bom>
<rule target="all">
 <prereq>
  <rule target="foo" />
  <rule target="bar" />
 </prereq>
</rule>
<rule target="foo">
 <prereq>
  <rule target="baz" />
 </prereq>
</rule>
<rule target="baz" />
<rule target="bar" />
</bom>
```

另一个坑是，如果 makefile 包含没有命令的规则，这些规则会导致示例 5-3 中的技术输出的树中断。例如，如果示例 makefile 如下：

```
all: foo bar
→ @echo Making $@

foo: baz

bar:
→ @echo Making $@

baz:
→ @echo Making $@
```

生成的 XML 完全不会提到 `baz`，因为 `foo` 的规则没有任何命令。因此 `SHELL` 不会被展开，黑客方法无法生效。以下是这种情况下的 XML：

```
<bom>
<rule target="all">
 <prereq>
  <rule target="foo" />
  <rule target="bar" />
 </prereq>
</rule>
<rule target="foo" />
<rule target="bar" />
</bom>
```

作为一种变通方法，我们可以修改 `foo: baz`，并为其添加一个无用的命令：

```
foo: baz ; @true
```

现在将生成正确的结果。

# 高级用户定义函数

在第一章中，我们讨论了在 GNU `make` 中创建用户定义的函数。现在我们将深入了解 GNU `make` 源代码，看看如何通过编写 C 代码来增强 GNU `make`，使其支持我们自己的内建函数。

首先，我们从自由软件基金会获取 GNU `make` 源代码。对于这一部分，我使用的是 GNU `make` 3.81。对于 GNU `make` 3.82 或 4.0，变化不大。

下载 `make-3.81.tar.gz`，然后使用 `gunzip` 和 `untar` 解压，再使用标准的 `configure` 和 `make` 构建 GNU `make`：

```
$ **cd make-3.81**
$ **./configure**
$ **make**
```

完成之后，我们得到一个在同一目录下工作的 GNU `make`。

## 开始修改 GNU make

能够知道自己正在运行哪个版本的 GNU `make` 非常方便。因此，作为第一次修改，我们将更改打印版本信息时显示的消息。默认信息如下：

```
$ **./make -v**
GNU Make 3.81
Copyright (C) 2006 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.
There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE.

This program built for i386-apple-darwin9.2.0
```

正如你所看到的，我正在使用 Mac（该字符串会根据你使用的机器有所不同），并且使用的是 GNU `make` 版本 3.81。

让我们更改该消息，使其在版本号后打印 `(with jgc's modifications)`。为此，我们需要在文本编辑器中打开 `main.c` 文件，找到 `print_version` 函数（位于第 2,922 行），它看起来像这样：

```
/* Print version information. */

static void
print_version (void)
{
static int printed_version = 0;

char *precede = print_data_base_flag ? "# " : "";

if (printed_version)
 /* Do it only once. */
 return;

/* Print this untranslated. The coding standards recommend translating the
  (C) to the copyright symbol, but this string is going to change every
  year, and none of the rest of it should be translated (including the
  word "Copyright", so it hardly seems worth it. */

printf ("%sGNU Make %s\n\
%sCopyright (C) 2006 Free Software Foundation, Inc.\n",
    precede, version_string, precede);

printf (_("%sThis is free software; see the source for copying conditions.\n\
%sThere is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A\n\
%sPARTICULAR PURPOSE.\n"),
     precede, precede, precede);

if (!remote_description || *remote_description == '\0')
 printf (_("\n%sThis program built for %s\n"), precede, make_host);
else
 printf (_("\n%sThis program built for %s (%s)\n"),
     precede, make_host, remote_description);

printed_version = 1;

/* Flush stdout so the user doesn't have to wait to see the
  version information while things are thought about. */
fflush (stdout);
}
```

`print_version` 中的第一个 `printf` 是打印版本号的地方。我们可以像这样修改它：

```
printf ("%sGNU Make %s (with jgc's modifications)\n\
%sCopyright (C) 2006 Free Software Foundation, Inc.\n",
    precede, version_string, precede);
```

保存文件后，再次运行 `make`。现在输入 **`make -v`**：

```
$ **./make -v**
GNU Make 3.81 (with jgc's modifications)
Copyright (C) 2006 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.
There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE.

This program built for i386-apple-darwin9.2.0
```

现在我们知道我们正在使用哪个版本。

## 内置函数的结构

GNU `make` 的内置函数在 `function.c` 文件中定义。为了开始理解这个文件如何工作，可以查看 GNU `make` 知道的函数表。这个表叫做 `function_table_init[]`，位于第 2,046 行。由于这个表比较大，我删除了其中的一些行：

```
static struct function_table_entry function_table_init[] =
{
/* Name/size */          /* MIN MAX EXP? Function */
{ STRING_SIZE_TUPLE("abspath"),    0, 1, 1, func_abspath},
{ STRING_SIZE_TUPLE("addprefix"),   2, 2, 1,
func_addsuffix_addprefix},
{ STRING_SIZE_TUPLE("addsuffix"),   2, 2, 1,
func_addsuffix_addprefix},
{ STRING_SIZE_TUPLE("basename"),   0, 1, 1, func_basename_dir},
{ STRING_SIZE_TUPLE("dir"),      0, 1, 1, func_basename_dir},
--*snip*--

{ STRING_SIZE_TUPLE("value"),     0, 1, 1, func_value},
{ STRING_SIZE_TUPLE("eval"),     0, 1, 1, func_eval},
#ifdef EXPERIMENTAL
{ STRING_SIZE_TUPLE("eq"),      2, 2, 1, func_eq},
{ STRING_SIZE_TUPLE("not"),      0, 1, 1, func_not},
#endif
};
```

每一行都定义了一个单独的函数，并包含五个信息项：函数的名称、函数必须具有的最小参数数量、最大参数数量（指定最小值非零且最大值为零，意味着该函数可以有无限数量的参数）、参数是否需要展开，以及实际执行函数的 C 函数的名称。

例如，下面是 `findstring` 函数的定义：

```
{ STRING_SIZE_TUPLE("findstring"), 2, 2, 1, func_findstring},
```

`findstring` 至少需要两个参数，最多两个参数，并且在调用 C 函数`func_findstring`之前，参数应当被展开。`func_findstring`（位于 `function.c` 文件的第 819 行）完成实际的工作：

```
static char*
func_findstring (char *o, char **argv, const char *funcname UNUSED)
{
/* Find the first occurrence of the first string in the second. */
if (strstr (argv[1], argv[0]) != 0)
 o = variable_buffer_output (o, argv[0], strlen (argv[0]));

return o;
}
```

实现 GNU `make` 内置函数的 C 函数有三个参数：`o`（指向一个缓冲区的指针，函数的输出将写入该缓冲区），`argv`（函数的参数，作为一个以 null 结尾的字符串数组），以及 `funcname`（一个包含函数名称的字符串；大多数函数不需要这个，但如果一个 C 函数处理多个 GNU `make` 函数时，它会很有用）。

你可以看到，`func_findstring` 仅使用标准 C 库中的 `strstr` 函数来查找第二个参数（在 `argv[1]` 中）是否出现在第一个参数中（在 `argv[0]` 中）。

`func_findstring` 使用了一个方便的 GNU `make` C 函数，名为 `variable_buffer_output`（定义在 `expand.c` 文件的第 57 行）。`variable_buffer_output` 将一个字符串复制到 GNU `make` 函数的输出缓冲区 `o` 中。第一个参数应该是输出缓冲区，第二个是要复制的字符串，最后一个是要复制的字符串的长度。

`func_findstring` 要么将第一个参数的全部内容复制到输出缓冲区 `o` 中（如果 `strstr` 成功），要么不改变 `o`（因此，`o` 将保持为空，因为它在调用 `func_findstring` 之前被初始化为空字符串）。

有了这些信息，我们就有足够的基础开始编写我们自己的 GNU `make` 函数了。

## 反转一个字符串

在 GNU `make` 中没有直接的方法来反转一个字符串，但编写一个 C 函数来实现这一点并将其插入到 GNU `make` 中是很容易的。

首先，我们将把 `reverse` 的定义添加到 GNU `make` 知道的函数列表中。`reverse` 将有一个必须展开的单一参数，并且将调用一个名为 `func_reverse` 的 C 函数。

下面是需要添加到 `function_table_init[]` 中的条目：

```
{ STRING_SIZE_TUPLE("reverse"), 1, 1, 1, func_reverse},
```

现在我们可以定义`func_reverse`，它通过交换字符来反转`argv[0]`中的字符串，并更新输出缓冲区`o`，如示例 5-4 所示：

示例 5-4. 使用 C 定义 GNU make 函数

```
static char*
func_reverse(char *o, char **argv, const char *funcname UNUSED)
{
int len = strlen(argv[0]);
if (len > 0) {
 char * p = argv[0];
 int left = 0;
 int right = len - 1;
 while (left < right) {
  char temp = *(p + left);
  *(p + left) = *(p + right);
  *(p + right) = temp;
  left++;
  right--;
 }

 o = variable_buffer_output(o, p, len);
}

return o;
}
```

这个函数通过从字符串的起始和结尾同时遍历，并交换字符对，直到`left`和`right`在中间相遇。

为了测试它，我们可以编写一个简单的 makefile，尝试三种情况：一个空字符串、一个长度为偶数的字符串，以及一个长度为奇数的字符串，所有这些都调用新的内建函数`reverse`：

```
EMPTY :=

$(info Empty string: [$(reverse $(EMPTY))]);

EVEN := 1234
$(info Even length string: [$(reverse $(EVEN))]);

ODD := ABCDE
$(info Odd length string: [$(reverse $(ODD))]);
```

输出显示它正常工作：

```
$ **./make**
Empty string: []
Even length string: [4321]
Odd length string: [EDCBA]
```

使用 C 语言编写可以访问完整的 C 标准库函数；因此，你可以创建的 GNU `make`内建函数仅受你的想象力限制。

# GNU make 4.0 可加载对象

将`reverse`函数添加到 GNU `make`中相当复杂，因为我们需要修改 GNU `make`的源代码。但是使用 GNU `make` 4.0 或更高版本，你可以在不更改源代码的情况下将 C 函数添加到 GNU `make`中。GNU `make` 4.0 添加了一个`load`指令，允许你加载一个包含用 C 语言编写的 GNU `make`函数的共享对象。

你可以将示例 5-4 中的`reverse`函数转换为可加载的 GNU `make`对象，只需将其保存在名为`reverse.c`的文件中，并做一些小修改。下面是完整的`reverse.c`文件：

```
   #include <string.h>
   #include <gnumake.h>

➊ int plugin_is_GPL_compatible;

   char* func_reverse(const char *nm, unsigned int argc, char **argv)
   {
     int len = strlen(argv[0]);
     if (len > 0) {
➋     char * p = gmk_alloc(len+1);
       *(p+len) = '\0';
       int i;
       for (i = 0; i < len; i++) {
         *(p+i) = *(argv[0]+len-i-1);
       }
       return p;
     }

     return NULL;
   }

   int reverse_gmk_setup()
   {
➌   gmk_add_function("reverse", func_reverse, 1, 1, 1);
     return 1;
   }
```

通过在➌处调用`gmk_add_function`，将`reverse`函数添加到 GNU `make`中。然后，`reverse`函数就可以像任何其他 GNU `make`内建函数一样使用。字符串的实际反转由`func_reverse`处理，它在➋处调用 GNU `make`的 API 函数`gmk_alloc`为新字符串分配空间。

➊处是一个特殊的、未使用的变量`plugin_is_GPL_compatible`，它在任何可加载模块中都是必需的。

要使用新的`reverse`函数，你需要将`reverse.c`文件编译成`.so`文件并加载到 GNU `make`中：

```
   all:
   --*snip*--
   load reverse.so
➍ reverse.so: reverse.c ; @$(CC) -shared -fPIC -o $@ $<
```

`load`指令加载`.so`文件，规则➍从`.c`文件构建`.so`文件。如果在 GNU `make`遇到`load`指令时`.so`文件缺失，GNU `make`会根据规则构建该文件并重新启动，从头解析 makefile。

加载后，你可以如下使用`reverse`：

```
A_PALINDROME := $(reverse saippuakivikauppias)
```

注意，使用`$(call)`并非必要。`reverse`函数就像任何其他内建的 GNU `make`函数一样。

# 在 GNU make 中使用 Guile

GNU `make` 4.0 引入了一个重要的变化，即`$(guile)`函数。这个函数的参数被发送到内置的 Guile 语言并由它执行。（GNU Guile 是 Scheme 的一个实现，而 Scheme 本身就是 Lisp。）`$(guile)`的返回值是执行后的 Guile 代码返回的值，经过转换成 GNU `make`能识别的类型。严格来说，GNU `make`没有数据类型（所有东西都是字符串），尽管它有时会把字符串当作其他类型（例如，包含空格的字符串在许多函数中会被当作列表）。

这是如何使用`$(guile)`和 Guile 函数`reverse`反转一个列表的方法：

```
   NAMES := liesl friedrich louisa kurt brigitta marta gretl

➊ $(info $(guile (reverse '($(NAMES)))))
```

执行时，这个 Makefile 将输出：

```
$ make
gretl marta brigitta kurt louisa friedrich liesl
```

值得深入研究➊，看看会发生什么，因为有几个微妙的细节。`$(guile)`的参数首先由 GNU `make`展开，所以➊变成了：

```
$(info $(guile (reverse '(liesl friedrich louisa kurt brigitta marta gretl))))
```

所以要执行的 Guile 代码是`(reverse '(liesl friedrich louisa kurt brigitta marta gretl))`。GNU `make`变量`$(NAMES)`已经被扩展成名字列表，并通过用`'(...)`将其包装变成了 Guile 列表。因为 Guile 有数据类型，你必须使用正确的语法：在这种情况下，你需要用圆括号括住一个列表，并用单引号引起来，告诉 Guile 这是一个字面意义的列表（而不是函数调用）。

Guile 的`reverse`函数反转这个列表并返回反转后的列表。GNU `make`然后将 Guile 列表转换为 GNU `make`列表（一个包含空格的字符串）。最后，`$(info)`显示该列表。

由于 Guile 是一种功能强大的语言，因此可以创建更复杂的函数。例如，这里有一个名为`file-exists`的 GNU `make`函数，它使用 Guile 的`access?`函数来判断一个文件是否存在。它返回一个布尔值，将 Guile 的`#t`/`#f`（真/假）值通过转换成 GNU `make`的布尔值（非空字符串表示真，空字符串表示假）：

```
file-exists = $(guile (access? "$1" R_OK))
```

注意参数`$1`周围的双引号。Guile 需要知道文件名实际上是一个字符串。

你可以通过在 Makefile 中使用 Guile 的`http-get`函数从网络下载数据来构建一个更复杂的例子：

```
define setup
(use-modules (web uri))
(use-modules (web client))
(use-modules (ice-9 receive))
endef

$(guile $(setup))

UA := "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_0) \
AppleWebKit/537.36 (KHTML, like Gecko) Chrome/40.0.2214.115 \
Safari/537.36"

define get-url
(receive (headers body)
  (http-get
    (string->uri "$1")
    #:headers '((User-Agent . $(UA))))
  body)
endef
utc-time = $(guile $(call get-url,http://www.timeapi.org/utc/now))

$(info $(utc-time))
```

在这里，`http-get`从一个 Web 服务中获取当前的 UTC 时间，该服务以字符串形式在 HTTP 响应的正文中返回时间。

`utc-time`变量包含当前的 UTC 时间。它通过使用存储在`get-url`变量中的 Guile 代码从*[`www.timeapi.org/utc/now/`](http://www.timeapi.org/utc/now/)*获取时间。`get-url`中的 Guile 代码使用`http-get`函数来获取网页的头部和正文，并只返回正文。

注意如何使用 GNU `make define`指令来创建大量的 Guile 代码块。如果 Guile 代码变得庞大，可以这样做：

`$(guile (load "myfunctions.scm"))`

这是你如何将 Guile 代码存储在文件中并加载它。

# 自文档化的 Makefile

遇到一个新的 makefile，许多人会问：“这个 makefile 做什么？”或者“我需要了解哪些重要的目标？”对于任何规模较大的 makefile，回答这些问题可能会很困难。在本节中，我将介绍一个简单的 GNU `make` 技巧，你可以使用它使 makefile 自动生成文档，并自动输出帮助信息。

在我向你展示它是如何工作的之前，这里有一个小示例。这个 makefile 有三个目标，创建者认为你需要了解：`all`、`clean` 和 `package`。他们通过为每个目标附加一些额外信息来记录这个 makefile：

```
include help-system.mk

all: $(call print-help,all,Build all modules in Banana Wumpus system)
→ ...commands for building all ...

clean: $(call print-help,clean,Remove all object and library files)
→ ...commands for doing a clean ...

package: $(call print-help,package,Package application-must run all target first)
→ ...commands for doing package step ...
```

对于每个需要文档化的目标，makefile 维护者添加了对用户自定义函数 `print-help` 的调用，传递了两个参数：目标的名称和该目标的简短描述。对 `print-help` 的调用不会干扰规则的前提条件定义，因为它始终返回（或扩展为）空字符串。

输入 `make` 和这个 makefile 会输出：

```
$ **make**
Type 'make help' to get help
```

输入 `make help` 会显示：

```
$ **make help**
Makefile:11: all -- Build all modules in Banana Wumpus system
Makefile:17: clean -- Remove all object and library files
Makefile:23: package -- Package application-must run all target first
```

`make` 会自动打印出有趣的目标名称，并包括关于它们所做的事情的解释，以及 makefile 中你可以找到更多目标相关命令的信息所在的行号。

有趣的工作是由包含的 makefile `help-system.mak` 完成的。`help-system.mak` 首先定义了用户自定义函数 `print-help`。`print-help` 是为每个需要文档化的目标调用的函数：

```
define print-help
$(if $(need-help),$(warning $1 -- $2))
endef
```

`print-help` 使用 GNU `make` 的 `$(warning)` 函数，根据传递给它的两个参数输出相应的消息。第一个参数（存储在 `$1` 中）是目标的名称，第二个参数（在 `$2` 中）是帮助文本；它们通过 `--` 分隔。`$(warning)` 将消息写入控制台并返回一个空字符串；因此，你可以安全地在规则的前提条件列表中使用 `print-help`。

`print-help` 通过检查 `need-help` 变量来决定是否需要打印任何消息。如果用户在 `make` 命令行中指定了 `help`，则 `need-help` 变量的值为字符串 `help`，否则为空字符串。在任何情况下，`print-help` 的扩展值都是空字符串。`need-help` 通过检查内置变量 `MAKECMDGOALS` 来判断用户是否在命令行中输入了 `help`，`MAKECMDGOALS` 是一个空格分隔的目标列表，列出了命令行中指定的所有目标。`need-help` 会过滤掉任何不匹配文本 `help` 的目标，因此，如果 `MAKECMDGOALS` 中包含 `help`，`need-help` 为字符串 `help`，否则为空。

```
need-help := $(filter help,$(MAKECMDGOALS))
```

`need-help` 和 `print-help` 的定义是我们所需要的，当命令行中输入 `help` 时，`make` 会打印出每个目标的帮助信息。`help-system.mak` 的其余部分会在用户只输入 `make` 时打印出消息 `Type 'make help' to get help`。

它为 makefile 定义了一个默认目标，名为 `help`，如果命令行中没有指定其他目标，则会运行此目标：

```
help: ; @echo $(if $(need-help),,Type \'$(MAKE)$(dash-f) help\' to get help)
```

如果用户请求了`help`（通过`need-help`变量确定），此规则将不输出任何内容；否则，它将输出包含`make`程序名称（存储在`$(MAKE)`中）以及加载 makefile 所需的适当参数的消息。这最后一部分是微妙的。

如果包含`help-system.mak`的 makefile 文件名仅为`Makefile`（或`makefile`或`GNUmakefile`），那么 GNU `make`会自动查找它，输入`make help`即可获得帮助。如果不是这种情况，则需要使用`-f`参数指定实际的 makefile 文件名。

该规则使用名为`dash-f`的变量来输出正确的命令行。如果使用了默认的 makefile 文件名，则`dash-f`不包含任何内容；否则，它包含`-f`，后跟正确的 makefile 文件名：

```
dash-f := $(if $(filter-out Makefile makefile GNUmakefile, \
$(parent-makefile)), -f $(parent-makefile))
```

`dash-f`查看名为`parent-makefile`的变量的值，该变量包含了包含`help-system.mak`的 makefile 的文件名。如果它不是标准名称，`dash-f`会返回带有`-f`选项的父 makefile 文件名。

`parent-makefile`是通过查看`MAKEFILE_LIST`来确定的。`MAKEFILE_LIST`是一个按顺序列出已读取的所有 makefile 的列表。`help-system.mak`首先确定它自己的文件名：

```
this-makefile := $(call last-element,$(MAKEFILE_LIST))
```

然后它通过从`MAKEFILE_LIST`中移除`this-makefile`（即`help-system.mak`）来获取所有其他包含的 makefile 的列表：

```
other-makefiles := $(filter-out $(this-makefile),$(MAKEFILE_LIST))
```

`other-makefiles`的最后一个元素将是`help-system.mak`的父级：

```
parent-makefile := $(call last-element,$(other-makefiles))
```

你可以使用`last-element`函数获取以空格分隔的列表中的最后一个元素：

```
define last-element
$(word $(words $1),$1)
endef
```

`last-element`通过使用`$(words)`获取单词计数并返回相应的单词，来返回列表中的最后一个单词。由于 GNU `make`的列表是从位置 1 开始计数的，`$(words LIST)`表示最后一个元素的索引。

## 使用`print-help`文档化 Makefiles

使用`print-help`文档化 makefile 非常简单。只需将相关的`$(call print-help,target,description)`添加到每个要文档化的目标的前置条件列表中。如果将调用添加到用于该目标的命令旁边，帮助系统不仅会打印帮助，还会自动将用户指向 makefile 中查看更多信息的地方。

由于目标的描述实际上是目标定义的一部分，而不是单独的帮助列表，因此保持文档的更新也很容易。

## 完整的 help-system.mak

最后，这是完整的`help_system.mak`文件：

```
help: ; @echo $(if $(need-help),,Type \'$(MAKE)$(dash-f) help\' to get help)

need-help := $(filter help,$(MAKECMDGOALS))

define print-help
$(if $(need-help),$(warning $1 -- $2))
endef

define last-element
$(word $(words $1),$1)
endef

this-makefile := $(call last-element,$(MAKEFILE_LIST))
other-makefiles := $(filter-out $(this-makefile),$(MAKEFILE_LIST))
parent-makefile := $(call last-element,$(other-makefiles))

dash-f := $(if $(filter-out Makefile makefile GNUmakefile, \
$(parent-makefile)), -f $(parent-makefile))
```

只需`include help-system.mak`，即可在需要文档的 makefile 中开始使用该系统。

在第六章中，我们将介绍一个有用的资源——GMSL 项目。创建 GNU `make`内置函数很简单，但它确实带来了一个维护问题：下一次更新 GNU `make`时，我们需要将更改移植到新版本。如果我们能够在不修改源代码的情况下使用 GNU `make`内置功能完成需求，那么 makefile 将会更具可移植性。GMSL 提供了大量额外的功能，而无需修改 GNU `make`的源代码。
