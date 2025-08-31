# 第三章 构建与重建

了解何时以及为何重新构建目标和执行配方是使用 GNU `make` 的基础。对于简单的 makefile，很容易理解为什么会构建某个特定的目标文件，但对于现实世界中的 makefile，构建和重新构建变得更加复杂。此外，GNU `make` 的依赖关系可能会受到限制，因为文件只有在先决条件的修改时间晚于目标文件时才会更新。而且在大多数情况下，只有一个目标会由一个规则更新。

本章解释了在 GNU `make` 中处理依赖关系的高级技巧，包括在目标的配方发生变化时重新构建，在文件的校验和发生变化时重新构建，如何最好地实现递归 `make`，以及如何在一个规则中构建多个目标。

# 当 CPPFLAGS 变化时重新构建

本节展示了如何实现 GNU `make` 中一个重要的“缺失功能”：当目标的构建命令发生变化时，重新构建目标的能力。GNU `make` 在目标*过时*时重新构建目标；也就是说，当某些先决条件比目标本身更新时，它会重新构建。但是如果目标看起来是最新的（通过文件时间戳判断），但实际构建该目标的命令已经发生变化怎么办？

例如，当一个非调试构建之后跟着一个调试构建（可能是先运行 `make` 然后运行 `make DEBUG=1`）时会发生什么？除非构建结构设计得当，使得目标的名称依赖于是否为调试或非调试构建，否则不会发生任何变化。

GNU `make` 无法检测某些目标是否应该重新构建，因为它没有考虑到配方中的命令发生变化的情况。例如，如果 `DEBUG=1` 导致传递给编译器的标志发生变化，那么目标应该重新构建。

本节将教你如何通过几行 GNU `make` 代码实现这一目标。

## 示例 makefile

本节中使用的示例 makefile 来自 示例 3-1，用于演示*命令变化时重新构建*系统。为了使系统的操作更加清晰，我避免使用内建的 GNU `make` 规则，因此这个 makefile 并不像它本可以那么简单：

示例 3-1：用于演示命令变化时重新构建系统的示例 makefile。

```
all: foo.o bar.o

foo.o: foo.c
→ $(COMPILE.C) -DDEBUG=$(DEBUG) -o $@ $<

bar.o: bar.c
→ $(COMPILE.C) -o $@ $<
```

该 makefile 通过编译相应的 `.c` 文件来创建两个 `.o` 文件，`foo.o` 和 `bar.o`。编译使用内建变量 `COMPILE.C`（通常是系统中适用的编译器名称，后面跟着类似 `CPPFLAGS` 的变量以及使用 `$@` 和 `$<` 编译代码成目标文件）。

对 `$(DEBUG)` 的特定引用被转换为一个名为 `DEBUG` 的预处理器变量，使用编译器的 `-D` 选项。`foo.c` 和 `bar.c` 的内容已被省略，因为它们无关紧要。

这是在没有命令行选项的情况下运行`make`时发生的情况（这意味着`DEBUG`是未定义的）：

```
$ **make**
g++  -c -DDEBUG= -o foo.o foo.c
g++  -c -o bar.o bar.c
```

现在，`foo.o`和`bar.o`已经被创建，因此再次输入`make`不会做任何事情：

```
$ **make**
make: Nothing to be done for `all'.
```

输入`make DEBUG=1`也不会产生任何效果，即使如果使用`DEBUG`定义重建，`foo.o`文件很可能会有所不同（例如，它可能会包含由`#ifdef`控制的额外调试代码，`DEBUG`变量在源代码中被使用）：

```
$ **make DEBUG=1**
make: Nothing to be done for `all'.
```

下一节中的`signature`系统将修正这个问题，并且对 Makefile 维护者几乎不需要任何额外工作。

## 修改我们的示例 Makefile

为了解决前面一节中的问题，我们将使用一个辅助的 Makefile，名为`signature`。我们稍后会看看`signature`是如何工作的；首先，让我们看看如何修改示例 3-1 中的 Makefile 来使用它：

```
**include signature**

all: foo.o bar.o

foo.o: foo.c
→ **$(call do,$$(COMPILE.C) -DDEBUG=$$(DEBUG) -o $$@ $$<)**

bar.o: bar.c
→ **$(call do,$$(COMPILE.C) -o $$@ $$<)**

**-include foo.o.sig bar.o.sig**
```

文件做了三个更改：首先，`include signature`被添加到开头，这样处理更新*签名*的代码就被包含了。这些签名将捕获用于构建文件的命令，并在命令发生变化时用于重新构建。

其次，两个规则中的命令被包裹在`$(call do,...)`中，并且每个命令的`$`符号被用第二个`$`进行了转义。

第三，对于每个由`signature`管理的`.o`文件，都有一个对应的`.sig`文件的`include`。Makefile 的最后一行包括了`foo.o.sig`（对于`foo.o`）和`bar.o.sig`（对于`bar.o`）。请注意，使用了`-include`而不是单纯的`include`，以防`.sig`文件缺失（`-include`在要包含的文件不存在时不会产生错误）。

在你看到它是如何工作的之前，以下是一些它运行时的示例：

```
$ **make**
g++  -c -DDEBUG= -o foo.o foo.c
g++  -c -o bar.o bar.c
$ **make**
make: Nothing to be done for `all'.
```

首先，进行一次干净的构建（没有`.o`文件），然后重新运行`make`，以查看没有任何操作。

但是，现在在`make`命令行中将`DEBUG`设置为`1`会导致`foo.o`重新构建：

```
$ **make DEBUG=1**
g++  -c -DDEBUG=1 -o foo.o foo.c
```

这是因为它的*签名*（用于构建`foo.o`的实际命令）发生了变化。

当然，`bar.o`没有被重新构建，因为它确实是最新的（它的目标代码是新的，并且没有命令变化）。再次运行`make DEBUG=1`时，它会显示没有任何需要做的事情：

```
$ **make DEBUG=1**
make: Nothing to be done for `all'.
$ **make**
g++  -c -DDEBUG= -o foo.o foo.c
```

但是，只需输入`make`（回到非调试构建模式）就会再次重建`foo.o`，因为`DEBUG`现在未定义。

`signature`系统同样适用于递归变量中的变量。在 GNU `make`中，`COMPILE.C`实际上展开了`CPPFLAGS`以创建完整的编译器命令行。如果通过添加定义在 GNU `make`命令行中修改了`CPPFLAGS`，会发生以下情况：

```
$ **make CPPFLAGS+=-DFOO=foo**
g++ -DFOO=foo -c -DDEBUG= -o foo.o foo.c
g++ -DFOO=foo -c -o bar.o bar.c
```

由于`CPPFLAGS`发生了变化（并且`CPPFLAGS`是用于构建这两个目标文件的命令的一部分），所以`foo.o`和`bar.o`都被重新构建了。

当然，修改一个未被引用的变量并不会更新任何内容。例如：

```
$ **make**
g++  -c -DDEBUG= -o foo.o foo.c
g++  -c -o bar.o bar.c
$ **make SOMEVAR=42**
make: Nothing to be done for `all'.
```

这里我们从一个干净的构建开始，并重新定义了`SOMEVAR`。

## 签名如何工作

要理解`signature`是如何工作的，首先查看`.sig`文件。`.sig`文件是由`signature` makefile 中的规则自动生成的，适用于每个使用`$(call do,...)`形式的规则。

例如，下面是第一次干净构建运行后`foo.o.sig`文件的内容：

```
$(eval @ := foo.o)
$(eval % := )
$(eval < := foo.c)
$(eval ? := foo.o.force)
$(eval ^ := foo.c foo.o.force)
$(eval + := foo.c foo.o.force)
$(eval * := foo)

foo.o: foo.o.force

$(if $(call sne,$(COMPILE.C) -DDEBUG=$(DEBUG) -o $@ $<,\
g++ -c -DDEBUG= -o foo.o foo.c),$(shell touch foo.o.force))
```

前七行捕捉了在处理`foo.o`规则时定义的自动变量的状态。我们需要这些变量的值，以便将当前规则的命令（可能使用了自动变量）与上次运行该规则时的命令进行比较。

接下来是`foo.o: foo.o.force`这一行。这表示如果`foo.o.force`更新了，`foo.o`必须重新构建。正是这一行导致了`foo.o`在命令变化时被重建，下一行则会在命令发生变化时更新`foo.o.force`。

长的`$(if)`语句使用 GMSL 的`sne`（字符串不等）函数，将`foo.o`的当前命令（通过展开它们）与上次展开时的值进行比较。如果命令发生变化，则会调用`$(shell touch foo.o.force)`。

由于`.sig`文件在解析 makefile 时被处理（它们只是使用`include`读取的 makefile），所有`.force`文件将在任何规则运行之前被更新。因此，这个小小的`.sig`文件在命令发生变化时，完成了强制重建目标文件的工作。

`.sig`文件由`signature`创建：

```
include gmsl

last_target :=

dump_var = \$$(eval $1 := $($1))

define new_rule
@echo "$(call map,dump_var,@ % < ? ^ + *)" > $S
@$(if $(wildcard $F),,touch $F)
@echo $@: $F >> $S
endef
define do
$(eval S := $@.sig)$(eval F := $@.force)$(eval C := $(strip $1))
$(if $(call sne,$@,$(last_target)),$(call new_rule),$(eval last_target := $@))
@echo "S(subst ",\",$(subst $$,\$$,$$(if $$(call sne,$(strip $1),$C),$$(shell touch $F))))" >> $S
$C
endef
```

`signature`包含了 GMSL，并定义了用于封装规则中命令的关键`do`变量。当调用`do`时，它会创建相应的`.sig`文件，包含所有自动变量的状态。

`do`调用的`new_rule`函数捕捉了自动变量。它使用 GMSL 的`map`函数，调用另一个函数（`dump_var`），对每个`@ % < ? ^ + *`进行处理。`new_rule`函数还确保创建了相应的`.force`文件。

此外，`do`会写出复杂的`$(if)`语句，包含当前规则的命令的未展开和已展开版本。然后，它会在最后实际执行这些命令（那就是`$C`）。

## 限制

签名系统有一些限制，可能会让不小心的人陷入困境。首先，如果规则中的命令包含任何副作用——例如，如果它们调用了`$(shell)`——如果假设副作用只发生一次，系统可能会出现异常。

第二，必须确保在任何`.sig`文件之前包含`signature`。

第三，如果编辑了 makefile 并且规则中的命令发生了变化，签名系统不会注意到这一点。如果发生这种情况，必须重新生成相应的目标，以便更新`.sig`文件。

尝试在`new_rule`定义的末尾添加以下一行：

```
@echo $F: Makefile >> $S
```

你可以通过将 makefile 作为每个 makefile 目标的前置条件来让签名系统在 makefile 发生变化时自动重新构建。这行代码是实现这一点的最简单方法。

# 当文件的校验和发生变化时重新构建

除了让 GNU `make` 在命令更改时重新构建目标外，另一种常见的技术是，当文件内容发生变化时重新构建，而不仅仅是文件的时间戳。

这种情况通常出现是因为生成的代码的时间戳，或者从源代码控制系统提取的代码的时间戳，比相关对象的时间戳要旧，因此 GNU `make` 不知道需要重新构建该对象。即使文件的内容与上次构建该对象时不同，仍然可能发生这种情况。

一个常见的场景是，某个工程师在本地机器上进行构建，重新构建所有对象文件，然后从源代码控制中获取最新版本的源文件。一些较旧的源代码控制系统将源文件的时间戳设置为文件提交到源代码控制时的时间戳；在这种情况下，新构建的对象文件的时间戳可能比（可能已经更改的）源代码文件还要新。

在本节中，你将学习一种简单的技巧，让 GNU `make` 在源文件内容发生变化时正确地执行（重新构建）。

## 一个示例 Makefile

示例 3-2 中的简单 makefile 使用内置规则从 `.c` 文件构建 `.o` 文件，生成对象文件 `foo.o`，方法是从 `foo.c` 和 `foo.h` 中构建：

示例 3-2. 一个简单的 makefile，从 `foo.c` 和 `foo.h` 构建 `foo.o`

```
.PHONY: all
all: foo.o

foo.o: foo.c foo.h
```

如果 `foo.c` 或 `foo.h` 比 `foo.o` 更新，则 `foo.o` 将被重新构建。

如果 `foo.h` 发生变化而未更新其时间戳，GNU `make` 将不会做任何操作。例如，如果 `foo.h` 从源代码控制中更新，可能会导致该 makefile 做出错误的操作。

为了解决这个问题，我们需要一种方法强制 GNU `make` 考虑文件的内容，而不是时间戳。因为 GNU `make` 仅能处理时间戳，所以我们需要修改 makefile，使文件的时间戳与文件内容相关联。

## 处理文件内容

检测文件变化的一个简单方法是使用消息摘要函数（如 MD5）来生成文件的摘要。因为文件的任何变化都会导致摘要变化，所以只需检查摘要即可检测文件内容的变化。

为了强制 GNU `make` 检查每个文件的内容，我们将为每个待测试的源代码文件关联一个扩展名为 `.md5` 的文件。每个 `.md5` 文件将包含相应源代码文件的 MD5 校验和。

在示例 3-2 中，源代码文件`foo.c`和`foo.h`将分别有相关的`.md5`文件`foo.c.md5`和`foo.h.md5`。为了生成 MD5 校验和，我们使用`md5sum`工具：它输出一个包含输入文件 MD5 校验和的十六进制字符串。

如果我们确保当相关文件的*校验和*变化时，`.md5`文件的*时间戳*也会变化，GNU `make`就可以检查`.md5`文件的时间戳，而不需要实际的源文件。

在我们的示例中，GNU `make`会检查`foo.c.md5`和`foo.h.md5`的时间戳，以确定是否需要重新构建`foo.o`。

## 修改后的 Makefile

下面是完整的 Makefile，它检查源文件的 MD5 校验和，以便当这些文件的内容（从而其校验和）发生变化时重新构建对象：

```
to-md5 = $1 $(addsuffix .md5,$1)

.PHONY: all
all: foo.o

foo.o: $(call to-md5,foo.c foo.h)

%.md5: FORCE
→ @$(if $(filter-out $(shell cat $@ 2>/dev/null),$(shell md5sum $*)),md5sum $* > $@)

FORCE:
```

首先注意到，`foo.o`的先决条件列表已从`foo.c foo.h`更改为`$(call to-md5,foo.c foo.h)`。在 Makefile 中定义的`to-md5`函数会在其参数的所有文件名后添加`.md5`后缀。

因此，展开后，该行会变成：

```
foo.o: foo.c foo.h foo.c.md5 foo.h.md5.
```

这告诉 GNU `make`如果`.md5`文件中的任何一个更新，或者`foo.c`或`foo.h`中的任何一个更新，都需要重新构建`foo.o`。

为了确保`.md5`文件始终包含正确的时间戳，它们会被重新构建。每个`.md5`文件由`%.md5: FORCE`规则重新生成。使用空规则`FORCE:`意味着每次都会检查`.md5`文件。这里使用`FORCE`的方式有点类似于使用`.PHONY`：如果没有名为`FORCE`的文件，GNU `make`会构建它（由于没有配方，所以什么都不做），然后 GNU `make`会认为`FORCE`比`%.md5`文件更新，并重新构建它。由于我们不能使用`.PHONY: %.md5`，因此我们改用这个`FORCE`技巧。

`%.md5: FORCE`规则的命令只有在`.md5`文件不存在或`.md5`文件中存储的校验和与相应文件的校验和不同的情况下，才会实际重新构建`.md5`文件，其工作原理如下：

1.  `$(shell md5sum $*)`对与`%.md5`的`%`部分匹配的文件进行校验和计算。例如，当这个规则用于生成`foo.h.md5`文件时，`%`匹配`foo.h`，然后`foo.h`会存储在`$*`中。

1.  `$(shell cat $@ 2>/dev/null)`获取当前`.md5`文件的内容（如果文件不存在则为空；请注意`2>/dev/null`表示忽略错误）。然后，`$(filter-out)`比较从`.md5`文件中获取的校验和和通过`md5sum`生成的校验和。如果它们相同，`$(filter-out)`将为空字符串。

1.  如果校验和发生了变化，规则将实际运行`md5sum $* > $@`，这将更新`.md5`文件的内容和时间戳。存储的校验和将在稍后再次运行 GNU `make`时用于检测，`.md5`文件的时间戳变化将导致相关的目标文件被重新构建。

## 操作中的黑客技术

为了查看这个黑客如何在其先决条件的校验和发生变化时更新目标文件，我们创建`foo.c`和`foo.h`文件并运行 GNU `make`：

```
$ **touch foo.c foo.h**
$ **ls**
foo.c foo.h makefile
$ **make**
cc -c -o foo.o foo.c
$ **ls**
foo.c foo.c.md5 foo.h foo.h.md5 foo.o makefile
```

GNU `make`生成目标文件`foo.o`和两个`.md5`文件，`foo.c.md5`和`foo.h.md5`。每个`.md5`文件包含该文件的校验和：

```
$ **cat foo.c.md5**
d41d8cd98f00b204e9800998ecf8427e foo.c
```

首先，我们验证所有内容是否都是最新的，然后验证更改`foo.c`或`foo.h`上的时间戳是否会导致重新构建`foo.o`：

```
$ **make**
make: Nothing to be done for `all'.
$ **touch foo.c**
$ **make**
cc  -c -o foo.o foo.c
$ **make**
make: Nothing to be done for `all'.
$ **touch foo.h**
$ **make**
cc -c -o foo.o foo.c
```

为了演示更改源文件的内容会导致重新构建`foo.o`，我们可以通过更改例如`foo.h`的内容并执行`touch foo.o`使`foo.o`比`foo.h`更新来作弊，这通常意味着`foo.o`不会被重新构建。

因此，我们知道`foo.o`比`foo.h`更新，但自从上次构建`foo.o`以来，`foo.h`的内容已经发生了变化：

```
$ **make**
make: Nothing to be done for `all'.
$ **cat foo.h.md5**
d41d8cd98f00b204e9800998ecf8427e foo.h
$ **cat >> foo.h**
// Add a comment
$ **touch foo.o**
$ **make**
cc  -c -o foo.o foo.c
$ **cat foo.h.md5**
65f8deea3518fcb38fd2371287729332 foo.h
```

你可以看到即使`foo.o`比所有相关的源文件都更新，它还是被重新构建了，而且`foo.h.md5`已更新为`foo.h`的新校验和。

## 改进代码

我们可以对代码做一些改进：第一个是优化。当文件的校验和发生变化时，更新规则会导致`.md5`文件实际上对同一个文件运行两次`md5sum`，结果相同。这是浪费时间。如果你使用的是 GNU `make` 3.80 或更高版本，可以将`md5sum $*`的输出存储在一个临时变量`CHECKSUM`中，并只使用该变量：

```
%.md5: FORCE
→ @$(eval CHECKSUM := $(shell md5sum $*))$(if $(filter-out \
$(shell cat $@ 2>/dev/null),$(CHECKSUM)),echo $(CHECKSUM) > $@)
```

第二个改进是让校验和对源文件中的空格变化不敏感。毕竟，如果两个开发人员对于正确缩进的意见不同，导致对象文件重建，而其他内容没有变化，那将是一种遗憾。

`md5sum`工具没有忽略空格的方法，但很容易将源文件通过`tr`过滤器去除空格，再交给`md5sum`计算校验和。（不过请注意，去除所有空格可能不是一个好主意，至少对于大多数语言而言不是。）

# 自动依赖关系生成

任何大于简单示例的项目都会面临依赖管理问题。随着工程师修改项目，依赖关系必须被生成并保持最新。GNU `make`并未提供处理这些问题的工具。GNU `make`提供的仅仅是一个机制，用于表达文件之间的关系，使用其熟悉的*`target`* `:` *`prerequisite1 prerequisite2`* `...`语法。

GNU `make`的依赖语法有缺陷，因为它不仅仅是一个先决条件列表：第一个先决条件具有特殊含义。`:`右边的任何东西都是先决条件，但第一个带有规则（即命令）的先决条件是特殊的：它是被自动变量`$<`赋值的先决条件，并且通常也是传递给编译器（或其他命令）以生成目标的先决条件。

`$<` 变量在另一个方面也很特殊。有时目标会有配方和其他规则来指定先决条件。例如，像下面这样并不罕见：

```
foo.o: foo.c
4 @compile -o $@ $<

foo.o: myheader.h string.h
```

`$<` 的值是通过具有配方的规则来设置的（在本例中将是 `foo.c`）。

看一下这个：

```
foo.o: foo.c header.h system.h
→ @echo Compiling $@ from $<...
```

它的输出是

```
$ **make**
Compiling foo.o from foo.c...
```

这里，如果 `foo.c`、`header.h` 或 `system.h` 改变，`foo.o` 会被重新构建，但规则也表明 `foo.o` 是由 `foo.c` 构建的。假设我们的例子是这样写的：

```
foo.o: foo.c
foo.o: header.h system.h
→ @echo Compiling $@ from $<...
```

输出将是：

```
$ **make**
Compiling foo.o from header.h...
```

这显然是错误的。

## 一个示例 Makefile

最大的问题是为大型项目生成所有表示所有依赖关系的规则。接下来的部分将使用以下构造的示例 makefile 作为起点：

```
.PHONY: all
all: foo.o bar.o baz.o

foo.o: foo.c foo.h common.h header.h
bar.o: bar.c bar.h common.h header.h ba.h
baz.o: baz.c baz.h common.h header.h ba.h
```

三个目标文件（`foo.o`、`bar.o` 和 `baz.o`）是从相应的 `.c` 文件（`foo.c`、`bar.c` 和 `baz.c`）构建的。每个 `.o` 文件依赖于不同的头文件，如 makefile 中最后三行所示。makefile 使用 GNU `make` 的内置规则，通过系统的编译器执行编译。

这里没有提到最终可执行文件的构建。原因是这个例子专注于处理源文件和目标文件之间的依赖关系；对象文件之间的关系通常更容易手动维护，因为它们较少且这些关系是产品设计的一部分。

## makedepend 和 make depend

因为手动维护任何真实的 makefile 是不可能的，许多项目使用广泛可用的 `makedepend` 程序。`makedepend` 会读取 C 和 C++ 文件，查看 `#include` 语句，打开被包含的文件，并自动构建依赖关系。将 `makedepend` 集成到项目中的一种基本方式是使用特殊的 `depend` 目标，如 示例 3-3 所示。

示例 3-3. 在你的 makefile 中使用 `makedepend`

```
.PHONY: all
all: foo.o bar.o baz.o

SRCS = foo.c bar.c baz.c

DEPENDS = dependencies.d
.PHONY: depend
depend:
→ @makedepend -f - $(SRCS) > $(DEPENDS)

-include $(DEPENDS)
```

执行 `make depend` 时，makefile 会执行 `depend` 规则，这会对源文件（在 `SRCS` 变量中定义）运行 `makedepend` 并将依赖关系输出到名为 `dependencies.d` 的文件中（由 `DEPENDS` 变量定义）。

makefile 在最后一行通过包含 `dependencies.d` 文件来添加依赖关系。`dependencies.d` 的内容如下：

```
# DO NOT DELETE

foo.o: foo.h header.h common.h
bar.o: bar.h header.h common.h ba.h
baz.o: baz.h header.h common.h ba.h
```

请注意，`makedepend` 并不会尝试定义目标文件（如 `foo.o`）与它所生成的源文件（`foo.c`）之间的关系。在这种情况下，GNU `make` 的标准规则会自动找到相关的 `.c` 文件。

## 自动化 makedepend 和移除 make depend

`make depend` 风格存在两个问题。运行 `make depend` 可能很慢，因为即使没有变化，仍然必须搜索每个源文件。此外，它是一个手动步骤：在每次执行 `make` 之前，用户必须先执行 `make depend` 来确保依赖关系是正确的。解决这些问题的办法是自动化。

示例 3-4 显示了来自 示例 3-3 的 makefile 另一个版本：

示例 3-4. 当需要时自动运行 `makedepend`

```
.PHONY: all
all: foo.o bar.o baz.o

SRCS = foo.c bar.c baz.c

%.d : %.c
→ @makedepend -f - $< | sed 's,\($*\.o\)[ :]*,\1 $@ : ,g' > $@

-include $(SRCS:.c=.d)
```

这个版本仍然使用 `makedepend` 来生成依赖关系，但自动化了这个过程，并且只对已更改的源文件运行 `makedepend`。它通过将每个 `.c` 文件与一个 `.d` 文件关联来工作。例如，`foo.o`（由 `foo.c` 构建）有一个 `foo.d` 文件，其中仅包含 `foo.o` 的依赖关系行。

下面是 `foo.d` 的内容：

```
# DO NOT DELETE

foo.o foo.d : foo.h header.h common.h
```

请注意一项新增内容：这一行指定了何时重新构建 `foo.o`，但也指出在相同条件下应该重新构建 `foo.d`。如果任何与 `foo.o` 相关的源文件发生变化，`foo.d` 会被重新构建。`foo.c` 没有出现在这个列表中，因为它是作为重新构建 `.d` 文件的模式规则的一部分提到的（主 makefile 中的 `%.d : %.c` 规则意味着如果 `foo.c` 发生变化，`foo.d` 会被重新构建）。`foo.d` 是通过 `makedepend` 使用 示例 3-4 中显示的 `sed` 魔法添加到由 `makedepend` 创建的依赖关系行中的。

主 makefile 的最后一行包括所有 `.d` 文件：`$(SRCS:.c=.d)` 将 `SRCS` 变量中源文件的列表转换，将扩展名从 `.c` 更改为 `.d`。`include` 也告诉 GNU `make` 检查 `.d` 文件是否需要重新构建。

GNU `make` 将检查是否有规则来重新构建包含的 makefile（在这种情况下是 `.d` 文件），如有必要重新构建它们（按照 makefile 中指定的依赖关系），然后重新启动。这个 makefile 重新构建功能（*[`www.gnu.org/software/make/manual/html_node/Remaking-Makefiles.html`](http://www.gnu.org/software/make/manual/html_node/Remaking-Makefiles.html)*) 意味着只需键入 `make` 就会做正确的事：它会重新构建需要重新构建的任何依赖文件，但仅在源文件发生变化时。然后，GNU `make` 将根据新的依赖关系进行构建。

## 让已删除的文件从依赖关系中消失

不幸的是，如果源文件被删除，我们的 makefile 会因致命错误而中断。如果 `header.h` 不再需要，所有对它的引用都会从 `.c` 文件中移除，文件会从磁盘上删除，运行 `make` 会产生以下错误：

```
$ **make**
No rule to make target `header.h', needed by `foo.d'.
```

这是因为 `header.h` 仍然在 `foo.d` 中作为 `foo.d` 的前提条件被提到；因此，`foo.d` 无法重新构建。你可以通过让 `foo.d` 的生成更加智能来修复这个问题：

```
# DO NOT DELETE

foo.d : $(wildcard foo.h header.h common.h)
foo.o : foo.h header.h common.h
```

新的 `foo.d` 分别包含了 `foo.o` 和 `foo.d` 的依赖关系。`foo.d` 的依赖关系被包裹在调用 GNU `make` 的 `$(wildcard)` 函数中。

这是更新后的 makefile，它通过新一轮的 `makedepend` 调用和一个 `sed` 行来创建修改后的 `.d` 文件：

```
.PHONY: all
all: foo.o bar.o baz.o

SRCS = foo.c bar.c baz.c

%.d : %.c
→ @makedepend -f - $< | sed 's,\($*\.o\)[ :]*\(.*\),$@ : $$\(wildcard \2\)\n\1 : \2,g' > $@

-include $(SRCS:.c=.d)
```

现在移除头文件不会破坏`make`：当解析`foo.d`时，`foo.d`的依赖行会通过`$(wildcard)`处理。当文件名中没有通配符如`*`或`?`时，`$(wildcard)`充当一个简单的存在性过滤器，将所有不存在的文件从列表中移除。所以如果`header.h`被移除，`foo.d`的第一行将相当于以下内容：

```
foo.d : foo.h common.h
```

`make`会正常工作。这个示例 makefile 现在在添加`.c`文件时可以正常工作（用户只需更新`SRCS`，新的`.d`文件会自动创建），在删除`.c`文件时（用户更新`SRCS`，旧的`.d`文件会被忽略），在添加头文件时（因为这需要修改现有的`.c`或`.h`文件，`.d`文件会重新生成），以及在删除头文件时（`$(wildcard)`隐藏了删除操作，`.d`文件会重新生成）。

一个可能的优化是通过将生成`.d`文件的规则与生成`.o`文件的规则合并，来避免 GNU `make`重启的需要：

```
.PHONY: all
all: foo.o bar.o baz.o

SRCS = foo.c bar.c baz.c

%.o : %.c
→ @makedepend -f - $< | sed 's,\($*\.o\)[ :]*\(.*\),$@ : $$\(wildcard \2\)\n\1 : \2,g' > $*.d
→ @$(COMPILE.c) -o $@ $<

-include $(SRCS:.c=.d)
```

因为只有在`.o`文件需要更新时，`.d`文件才会更新（当任何`.o`文件的源文件发生变化时，两者都会更新），所以可以在编译的同时进行`makedepend`。

这个规则使用了`$*`，这是另一个 GNU `make`变量。`$*`是模式`%.c`中与`%`匹配的部分。如果这个规则正在从`foo.c`构建`foo.o`，那么`$*`就是`foo`。`$*`生成`makedepend`写入的`.d`文件的名称。

这个版本不使用 GNU `make`的 makefile 重建系统。没有创建`.d`文件的规则（它们是作为创建`.o`文件的副作用生成的），因此 GNU `make`不需要重启。这提供了准确性和速度的最佳结合。

通常，创建多个文件的规则是一个不好的主意，因为 GNU `make`无法找到由其他操作副作用创建的文件的规则。在这种情况下，这种行为是我们想要的：我们希望将`.d`文件的创建隐藏起来，以免 GNU `make`尝试生成它们并导致重启。

Tom Tromey 提出了一个类似的想法，没有使用`$(wildcard)`技巧。你可以在 GNU `make`的维护者 Paul Smith 的网站上找到更多关于构建依赖文件的信息，网址是*[`make.mad-scientist.net/papers/advanced-auto-dependency-generation/`](http://make.mad-scientist.net/papers/advanced-auto-dependency-generation/)*。

## 摆脱 makedepend

此外，如果使用的是 GNU `gcc`、`llvm`、`clang`或类似的编译器，可以完全省略`makedepend`。

`-MD`选项在编译的同时完成`makedepend`的工作：

```
.PHONY: all
all: foo.o bar.o baz.o

SRCS = foo.c bar.c baz.c

%.o : %.c
→ @$(COMPILE.c) -MD -o $@ $<
→ @sed -i 's,\($*\.o\)[ :]*\(.*\),$@ : $$\(wildcard \2\)\n\1 : \2,g' $*.d

-include $(SRCS:.c=.d)
```

例如，`foo.o`的编译步骤会从`foo.c`生成`foo.d`。然后，会对`foo.d`运行`sed`，为`foo.d`添加包含`$(wildcard)`的额外行。

## 使用 gcc -MP

`gcc`还有一个`-MP`选项，它试图通过创建空规则来“构建”缺失的文件，从而解决消失文件的问题。例如，可以完全消除`sed`魔法，使用`-MP`选项代替`-MD`：

```
.PHONY: all
all: foo.o bar.o baz.o

SRCS = foo.c bar.c baz.c

%.o : %.c
→ @$(COMPILE.c) -MP -o $@ $<

-include $(SRCS:.c=.d)
```

`foo.d`文件将如下所示：

```
foo.o : foo.h header.h common.h 
foo.h :
header.h :
common.h :
```

举例来说，如果`foo.h`被删除，`make`不会报错，因为它会找到空规则（`foo.h :`）来构建它，从而避免了缺失文件的错误。然而，每次构建`foo.o`时，更新`foo.d`文件是至关重要的。如果没有更新，`foo.d`中仍会包含`foo.h`作为前提条件，而每次运行`make`时，`foo.o`都会重新构建，因为`make`会尝试用空规则来构建`foo.h`（从而强制构建`foo.o`）。

# GNU make 中的原子规则

GNU `make`物理学的一个基本法则是每个规则只构建一个文件（称为*目标*）。这个规则是有例外的（我们将在本节后面看到），但无论如何，对于任何正常的 GNU `make`规则，像

```
a: b c
→ @command
```

左侧`:`的地方只有一个文件名被提及。这个文件名会被放入`$@`自动变量中。预计`command`会实际更新该文件。

本节解释了如果一个命令更新多个文件该怎么办，并且如何表达这一点，以便 GNU `make`知道有多个文件被更新并且正确地执行。

## 不该做的事

假设有一个命令可以在一个步骤中通过相同的前提条件构建两个文件（`a`和`b`）。在这一节中，这个命令通过`touch a b`来模拟，但实际上它可能比这复杂得多。

示例 3-5 展示了不该做的事情：

示例 3-5. 不该做的事

```
.PHONY: all
all: a b

a b: c d
→ touch a b
```

乍一看，示例 3-5 看起来是正确的；它似乎说明了`a`和`b`是通过一个命令从`c`和`d`构建的。如果你在`make`中运行它，你可能会得到类似这样的输出（尤其是在你使用`-j`选项来进行并行构建时）：

```
$ **make**
touch a b
touch a b
```

命令被运行了两次。在这种情况下这是无害的，但对于一个真正执行工作的命令，运行两次几乎肯定是不对的。此外，如果你使用`-j`选项进行并行构建，命令可能会同时多次运行。

原因在于 GNU `make`实际上是这样解释 makefile 的：

```
.PHONY: all
all: a b

a: c d
→ touch a b

b: c d
→ touch a b
```

有两个独立的规则（一个声明它构建`a`，另一个声明它构建`b`），这两个规则都构建`a`和`b`。

## 使用模式规则

GNU `make`确实有一种方式可以在单个规则中构建多个目标，使用模式规则。模式规则可以拥有任意数量的目标模式，仍然被视为一个规则。

举例来说：

```
%.foo %.bar %.baz:
→ command
```

这意味着具有`.foo`、`.bar`和`.baz`扩展名的文件（当然还有与`%`匹配的相同前缀）将在一次`command`调用中构建。

假设 makefile 像这样：

```
.PHONY: all
all: a.foo a.bar a.baz

%.foo %.bar %.baz:
→ command
```

然后，`command`只会被调用一次。事实上，仅指定一个目标并运行模式规则就足够了：

```
.PHONY: all
all: a.foo

%.foo %.bar %.baz:
→ command
```

这非常有用。例如：

```
$(OUT)/%.lib $(OUT)/%.dll: $(VERSION_RESOURCE)
→ link /nologo /dll /fixed:no /incremental:no  \
   /map:'$(call to_dos,$(basename $@).map)'    \
   /out:'$(call to_dos,$(basename $@).dll)'    \
   /implib:'$(call to_dos,$(basename $@).lib)' \
        $(LOADLIBES) $(LDLIBS)                 \
   /pdb:'$(basename $@).pdb'                   \
   /machine:x86                                \
   $^
```

这是一个实际的规则，来自一个真实的 makefile，它一次性构建了`.lib`及其相关的`.dll`。

当然，如果文件的名称中没有共同部分，使用模式规则将不起作用。它在本节开始时的简单示例中无法使用，但有一种替代方法。

## 使用哨兵文件

一个可能的替代方法是引入一个单独的文件，用来指示多目标规则中的任何目标是否已经构建。创建一个单一的“指示”文件将多个文件转化为一个文件，而 GNU `make`可以理解单一文件。以下是示例 3-5 的重写版本：

```
.PHONY: all
all: a b

a b: .sentinel
→ @:

.sentinel: c d
→ touch a b
→ touch .sentinel
```

构建`a`和`b`的规则只能运行一次，因为只指定了一个前提条件（`.sentinel`）。如果`c`或`d`较新，`.sentinel`会被重新构建（从而`a`和`b`也会被重新构建）。如果 makefile 请求`a`或`b`中的任何一个，它们会通过`.sentinel`文件重新构建。

`a b`规则中的有趣`@:`命令只是意味着有构建`a`和`b`的命令，但它们什么也不做。

使这一过程透明化会很不错。这就是`atomic`函数的作用。`atomic`函数会根据要构建的目标名称自动设置哨兵文件，并创建必要的规则：

```
sp :=
sp +=
sentinel = .sentinel.$(subst $(sp),_,$(subst /,_,$1))
atomic = $(eval $1: $(call sentinel,$1) ; @:)$(call sentinel,$1): $2 ; touch $$@

.PHONY: all
all: a b

$(call atomic,a b,c d)
→ touch a b
```

我们所做的只是将原来的`a b : c d`规则替换为对`atomic`的调用。第一个参数是需要原子化构建的目标列表；第二个参数是前提条件列表。

`atomic`使用`sentinel`函数创建一个唯一的哨兵文件名（对于目标`a b`，哨兵文件名为`.sentinel.a_b`），然后设置必要的规则。

在这个 makefile 中展开`atomic`就相当于这样做：

```
.PHONY: all
all: a b

a b: .sentinel.a_b ; @:

.sentinel.a_b: c d ; touch $@
→ touch a b
```

这种技术有一个缺陷。如果删除了`a`或`b`，你还必须删除相关的哨兵文件，否则文件不会重新构建。

为了解决这个问题，你可以让 makefile 在必要时删除哨兵文件，通过检查是否有任何正在构建的目标丢失。以下是更新后的代码：

```
sp :=
sp +=
sentinel = .sentinel.$(subst $(sp),_,$(subst /,_,$1))
atomic = $(eval $1: $(call sentinel,$1) ; @:)$(call sentinel,$1):  \
$2 ; touch $$@ $(foreach t,$1,$(if $(wildcard $t),,$(shell rm -f   \
$(call sentinel,$1))))

.PHONY: all
all: a b

$(call atomic,a b,c d)
→ touch a b
```

现在`atomic`遍历这些目标。如果有任何目标丢失（通过`$(wildcard)`检测），则会删除哨兵文件。

# 无痛非递归 make

一旦 makefile 项目达到一定规模（通常是当它依赖于子项目时），构建管理者就不可避免地会写出包含对`$(MAKE)`调用的规则。而就在这时，构建管理者创建了递归的`make`：一个执行整个`make`过程的`make`。这样做非常诱人，因为从概念上讲，递归`make`很简单：如果你需要构建一个子项目，只需进入其目录并通过`$(MAKE)`运行`make`。

但是它有一个主要的缺陷：一旦启动了一个单独的 `make` 进程，所有关于依赖的信息都会丢失。父 `make` 并不知道子项目的 `make` 是否真的需要执行，所以它每次都必须运行，这可能会很慢。解决这个问题并不容易，但一旦实现，非递归 `make` 是非常强大的。

对使用非递归 `make` 的一个常见反对意见是，使用递归 `make` 时，可以在源代码树的任何地方输入 `make`。这样做通常会构建在该层次的 makefile 中定义的对象（如果 makefile 递归，还可能构建下面的对象）。

非递归的 `make` 系统（基于 `include` 语句而非 `make` 调用）通常无法提供这种灵活性，而 GNU `make` 必须在顶层目录中运行。尽管非递归的 GNU `make` 通常更高效（从顶层目录运行应该很快），但能够为开发者提供与递归 `make` 系统相同的功能是很重要的。

本节概述了一种非递归的 GNU `make` 系统模式，它支持递归的 GNU `make` 系统中常见的 `make` 随处可用的风格。在一个目录中输入 `make` 会构建该目录及以下的所有内容，但没有递归的 `$(MAKE)` 调用。运行的唯一一个 `make` 知道跨项目和子项目的所有依赖关系，并且能够高效地构建。

## 一个简单的递归 `make`

想象一个包含以下子目录的项目：

```
/src/
/src/library/
/src/executable/
```

`/src/` 是顶层目录，在这里你可以输入 `make` 来进行完整的构建。在 `/src/` 目录下有一个 `library/` 目录，它从源文件 `lib1.c` 和 `lib2.c` 构建一个名为 `lib.a` 的库：

```
/src/library/lib1.c
/src/library/lib2.c
```

`/src/executable/` 目录从两个源文件（`foo.c` 和 `bar.c`）构建一个名为 `exec` 的可执行文件，并与库 `lib.a` 链接：

```
/src/executable/foo.c
/src/executable/bar.c
```

经典的递归 `make` 解决方案是在每个子目录中放置一个 makefile。每个 makefile 包含构建该目录对象的规则，而顶层的 makefile 会递归进入每个子目录。以下是一个递归 makefile (`/src/makefile`) 的内容：

```
SUBDIRS = library executable

.PHONY: all
all:
→ for dir in $(SUBDIRS); do \
→ $(MAKE) -C $$dir;         \
→ done
```

这会依次进入每个目录，并运行 `make` 来先构建库，再构建可执行文件。可执行文件和库之间的依赖关系（即库需要先于可执行文件构建）在 `SUBDIRS` 中指定的目录顺序中是隐式的。

下面是使用 `for` 循环和每个目录的虚假目标来改进的一个例子：

```
SUBDIRS = library executable

.PHONY: $(SUBDIRS)
$(SUBDIRS):
→ $(MAKE) -C $@

.PHONY: all
all: $(SUBDIRS)

executable: library
```

你需要解开 `all` 规则中的循环，为每个子目录创建独立的规则，并明确指定 `executable` 和 `library` 之间的依赖关系。这段代码更清晰，但它仍然是递归的，每个子目录都有单独的 `make` 调用。

## 一个灵活的非递归 `make` 系统

当转向非递归 `make` 时，理想的顶级 makefile 应该像 示例 3-6 这样。

示例 3-6：一个小型非递归 makefile

```
SUBDIRS = library executable

include $(addsuffix /makefile,$(SUBDIRS))
```

这仅仅是说要包含每个子目录中的 makefile。诀窍是如何使其工作！在你看到如何做之前，这里是 `library` 和 `executable` 子目录中 makefile 内容的框架：

```
# /src/library/Makefile

include root.mak
include top.mak

SRCS := lib1.c lib2.c
BINARY := lib
BINARY_EXT := $(_LIBEXT)
include bottom.mak
```

和

```
# /src/executable/Makefile

include root.mak
include top.mak
SRCS := foo.c foo.c
BINARY := exec
BINARY_EXT := $(_EXEEXT)

include bottom.mak
```

每个 makefile 都指定了要构建的源文件（在 `SRCS` 变量中）、最终链接的二进制文件的名称（在 `BINARY` 变量中）和二进制文件的类型（使用 `BINARY_EXT` 变量，该变量由特殊变量 `_LIBEXT` 和 `_EXEEXT` 设置）。

这两个 makefile 都 `include` 了位于 `/src/` 目录中的公共 makefile `root.mak`、`top.mak` 和 `bottom.mak`。

因为包含的 `.mak` makefile 不在子目录中，所以 GNU `make` 需要去寻找它们。要在 `/src` 中找到 `.mak` 文件，可以这样做：

```
$ **make -I /src**
```

在这里，你使用 `-I` 命令行选项将目录添加到 `include` 搜索路径中。

要求用户在 `make` 命令行中添加任何内容是令人遗憾的。为了避免这种情况，你可以创建一个简单的方法，自动向上遍历源代码树以找到 `.mak` 文件。以下是 `/src/library` 的实际 makefile：

```
sp :=
sp +=
_walk = $(if $1,$(wildcard /$(subst $(sp),/,$1)/$2) $(call _walk,$(wordlist 2,$(words $1),x $1),$2))
_find = $(firstword $(call _walk,$(strip $(subst /, ,$1)),$2))
_ROOT := $(patsubst %/root.mak,%,$(call _find,$(CURDIR),root.mak))

include $(_ROOT)/root.mak
include $(_ROOT)/top.mak

SRCS := lib1.c lib2.c
BINARY := lib
BINARY_EXT := $(_LIBEXT)

include $(_ROOT)/bottom.mak
```

`_find` 函数从 `$1` 中指定的目录开始向上遍历目录树，查找名为 `$2` 的文件。实际的查找是通过调用 `_walk` 函数实现的，该函数沿着树向上遍历，找到 `$1` 中每个逐渐缩短的路径中 `$2` 文件的每个实例。

makefile 开头的代码块找到 `root.mak` 的位置，它与 `top.mak` 和 `bottom.mak` 在同一目录下（即 `/src`），并将该目录保存在 `_ROOT` 中。

然后，makefile 可以使用 `$(_ROOT)/` 来 `include` `root.mak`、`top.mak` 和 `bottom.mak` makefile，而无需输入除 `make` 之外的任何内容。

以下是第一个包含的 makefile (`root.mak`) 的内容：

```
_push = $(eval _save$1 := $(MAKEFILE_LIST))
_pop = $(eval MAKEFILE_LIST := $(_save$1))
_INCLUDE = $(call _push,$1)$(eval include $(_ROOT)/$1/Makefile)$(call _pop,$1)
DEPENDS_ON = $(call _INCLUDE,$1)
DEPENDS_ON_NO_BUILD = $(eval _NO_RULES := T)$(call _INCLUDE,$1)$(eval _NO_RULES :=)
```

目前，忽略其内容，回到这些函数在查看模块之间的依赖关系时的作用。实际工作从 `top.mak` 开始：

```
_OUTTOP ?= /tmp/out

.PHONY: all
all:

_MAKEFILES := $(filter %/Makefile,$(MAKEFILE_LIST))
_INCLUDED_FROM := $(patsubst $(_ROOT)/%,%,$(if $(_MAKEFILES), \
$(patsubst %/Makefile,%,$(word $(words $(_MAKEFILES)),$(_MAKEFILES)))))
ifeq ($(_INCLUDED_FROM),)
_MODULE := $(patsubst $(_ROOT)/%,%,$(CURDIR))
else
_MODULE := $(_INCLUDED_FROM)
endif
_MODULE_PATH := $(_ROOT)/$(_MODULE)
_MODULE_NAME := $(subst /,_,$(_MODULE))
$(_MODULE_NAME)_OUTPUT := $(_OUTTOP)/$(_MODULE)

_OBJEXT := .o
_LIBEXT := .a
_EXEEXT :=
```

`_OUTTOP` 变量定义了所有二进制输出（目标文件等）将被放置的顶级目录。这里它的默认值是 `/tmp/out`，并且它是用 `?=` 定义的，因此可以在命令行中覆盖。

接下来，`top.mak` 设置 GNU `make` 的默认目标为经典的 `all`。这里它没有依赖项，但之后会为每个将要构建的模块添加依赖项。

之后，多个变量会将`_MODULE_PATH`设置为正在构建的模块目录的完整路径。例如，在构建`library`模块时，`_MODULE_PATH`将是`/src/library`。设置这个变量是复杂的，因为确定模块目录必须独立于执行 GNU `make`的目录（这样库文件既可以从顶层目录构建，适用于`make all`，也可以从单独的`library`目录构建，适用于单个开发者构建，甚至可以将库文件作为依赖项包含在另一个模块中）。

`_MODULE_NAME`只是相对于树根路径的路径，其中`/`被替换为`_`。在示例 3-5 中，这两个模块有`_MODULE_NAME`：`library`和`executable`。但是如果`library`有一个包含名为`sublibrary`的模块的子目录，那么它的`_MODULE_NAME`将是`library_sublibrary`。

`_MODULE_NAME`还用于创建`$(_MODULE_NAME)_OUTPUT`特殊变量，它的名称是基于`_MODULE_NAME`计算得出的。所以对于`library`模块，创建了变量`library_OUTPUT`，它包含将`library`的目标文件写入的目录的完整路径。输出路径是基于`_OUTTOP`和相对于正在构建模块的路径。因此，`/tmp/out`目录结构会镜像源代码目录结构。

最后，设置了一些用于文件名扩展名的标准定义。这里使用的是 Linux 系统的定义，但这些定义可以很容易地更改为不使用`.o`作为目标文件或`.a`作为库文件的系统（例如 Windows）。

`bottom.mak`使用这些变量来设置实际构建模块的规则：

```
$(_MODULE_NAME)_OBJS := $(addsuffix $(_OBJEXT),$(addprefix \
$($(_MODULE_NAME)_OUTPUT)/,$(basename $(SRCS)))) $(DEPS)
$(_MODULE_NAME)_BINARY := $($(_MODULE_NAME)_OUTPUT)/$(BINARY)$(BINARY_EXT)

ifneq ($(_NO_RULES),T)
ifneq ($($(_MODULE_NAME)_DEFINED),T)
all: $($(_MODULE_NAME)_BINARY)

.PHONY: $(_MODULE_NAME)
$(_MODULE_NAME): $($(_MODULE_NAME)_BINARY)
_IGNORE := $(shell mkdir -p $($(_MODULE_NAME)_OUTPUT))

_CLEAN := clean-$(_MODULE_NAME)
.PHONY: clean $(_CLEAN)
clean: $(_CLEAN)
$(_CLEAN):
→ rm -rf $($(patsubst clean-%,%,$@)_OUTPUT)

$($(_MODULE_NAME)_OUTPUT)/%.o: $(_MODULE_PATH)/%.c
→ @$(COMPILE.c) -o '$@' '$<'
$($(_MODULE_NAME)_OUTPUT)/$(BINARY).a: $($(_MODULE_NAME)_OBJS)
→ @$(AR) r '$@' $^
→ @ranlib '$@'
$($(_MODULE_NAME)_OUTPUT)/$(BINARY)$(_EXEEXT): $($(_MODULE_NAME)_OBJS)
→ @$(LINK.cpp) $^ -o'$@'

$(_MODULE_NAME)_DEFINED := T
endif
endif
```

`bottom.mak`首先设置两个带有计算名称的变量：`$(_MODULE_NAME)_OBJS`（它是从`SRCS`变量通过转换扩展名计算得出的模块目标文件列表）和`$(_MODULE_NAME)_BINARY`（它是模块创建的二进制文件的名称；通常是正在构建的库文件或可执行文件）。

我们包含了`DEPS`变量，因此`$(_MODULE_NAME)_OBJS`变量也包括该模块需要但不构建的任何目标文件。稍后你将看到如何使用这个变量在库和可执行文件之间定义依赖关系。

接下来，如果该模块的规则尚未设置（由`$(_MODULE_NAME)_DEFINED`变量控制），并且未被`_NO_RULES`变量明确禁用，则定义构建该模块的实际规则。

在这个示例中，展示了 Linux 的规则。这是你为其他操作系统更改此示例的地方。

`all`包含当前的二进制文件，来自`$(_MODULE_NAME)_BINARY`，它作为前提条件添加到模块中，这样在执行完整构建时该模块会被构建。接着有一个规则将模块名与模块二进制文件关联，这样在顶层执行`make library`时，只会构建库文件。

然后是一个通用的`clean`规则和一个模块特定的清理规则（对于`library`模块，有一个叫做`clean-library`的规则，仅清理它的对象文件）。`clean`通过简单的`rm -rf`实现，因为所有的输出文件都被组织在`_OUTTOP`的特定子目录中。

接下来，使用`$(shell)`来设置模块输出文件的目录。最后，特定的规则将该模块输出目录中的目标文件与该模块源代码目录中的源文件关联起来。

在建立了所有这些基础设施之后，我们终于可以查看`executable`目录中的 makefile 了：

```
sp :=
sp +=
_walk = $(if $1,$(wildcard /$(subst $(sp),/,$1)/$2) $(call _walk,$(wordlist 2,$(words $1),x $1),$2))
_find = $(firstword $(call _walk,$(strip $(subst /, ,$1)),$2))
_ROOT := $(patsubst %/root.mak,%,$(call _find,$(CURDIR),root.mak))

include $(_ROOT)/root.mak

$(call DEPENDS_ON,library)

include $(_ROOT)/top.mak

SRCS := foo.c bar.c
BINARY := exec
BINARY_EXT := $(_EXEEXT)
DEPS := $(library_BINARY)

include $(_ROOT)/bottom.mak
```

这看起来很像库的 makefile，但也有一些不同之处。因为可执行文件需要库，所以`DEPS`行指定了可执行文件依赖于库所创建的二进制文件。由于每个模块有独特的对象和二进制文件变量，所以可以通过引用`$(library_BINARY)`来轻松定义这种依赖关系，它会展开为由库模块创建的库文件的完整路径。

为了确保`$(library_BINARY)`被定义，需要包含来自`library`目录的 makefile。`root.mak`文件提供了两个使这一过程变得简单的函数：`DEPENDS_ON`和`DEPENDS_ON_NO_BUILD`。

`DEPENDS_ON_NO_BUILD`仅设置指定模块的变量，以便在 makefile 中使用。如果在`executable`的 makefile 中使用该函数，库文件（`lib.a`）必须存在，才能使可执行文件成功构建。另一方面，`DEPENDS_ON`在这里用于确保在必要时构建`library`。

`DEPENDS_ON_NO_BUILD`提供了类似经典递归构建的功能，虽然它不知道如何构建该库，但却依赖于它。`DEPENDS_ON`更加灵活，因为在没有递归的情况下，你可以指定关系，并确保代码能够被构建。

## 使用非递归 make 系统

非递归的`make`系统提供了很大的灵活性。以下是一些例子，展示了非递归的`make`系统与递归的`make`系统一样灵活（甚至更灵活！）。

从顶层构建所有内容很简单，只需运行`make`（在这些示例中，我们使用命令`make -n`，这样命令就会清晰地显示出来）：

```
$ **cd /src**
$ **make -n**
cc  -c -o '/tmp/out/library/lib1.o' '/home/jgc/doc/nonrecursive/library/lib1.c'
cc  -c -o '/tmp/out/library/lib2.o' '/home/jgc/doc/nonrecursive/library/lib2.c'
ar r '/tmp/out/library/lib.a' /tmp/out/library/lib1.o /tmp/out/library/lib2.o
ranlib '/tmp/out/library/lib.a'
cc  -c -o '/tmp/out/executable/foo.o' '/home/jgc/doc/nonrecursive/executable/foo.c'
cc  -c -o '/tmp/out/executable/bar.o' '/home/jgc/doc/nonrecursive/executable/bar.c'
g++ /tmp/out/executable/foo.o /tmp/out/executable/bar.o /tmp/out/library/lib.a -o'/tmp/out/
executable/exec'
```

清理所有内容也很简单：

```
$ **cd /src**
$ **make -n clean**
rm -rf /tmp/out/library
rm -rf /tmp/out/executable
```

从顶层目录开始，可以请求构建或清理任何单独的模块：

```
$ **cd /src**
$ **make -n clean-library**
rm -rf /tmp/out/library
$ **make -n library**
cc  -c -o '/tmp/out/library/lib1.o' '/home/jgc/doc/nonrecursive/library/lib1.c'
cc  -c -o '/tmp/out/library/lib2.o' '/home/jgc/doc/nonrecursive/library/lib2.c'
ar r '/tmp/out/library/lib.a' /tmp/out/library/lib1.o /tmp/out/library/lib2.o
ranlib '/tmp/out/library/lib.a'
```

如果我们要求构建`executable`模块，由于依赖关系，`library`模块也会同时被构建：

```
$ **cd /src**
$ **make -n executable**
cc -c -o '/tmp/out/executable/foo.o' '/home/jgc/doc/nonrecursive/executable/foo.c'
cc -c -o '/tmp/out/executable/bar.o' '/home/jgc/doc/nonrecursive/executable/bar.c'
cc -c -o '/tmp/out/library/lib1.o' '/home/jgc/doc/nonrecursive/library/lib1.c'
cc -c -o '/tmp/out/library/lib2.o' '/home/jgc/doc/nonrecursive/library/lib2.c'
ar r '/tmp/out/library/lib.a' /tmp/out/library/lib1.o /tmp/out/library/lib2.o
ranlib '/tmp/out/library/lib.a'
g++ /tmp/out/executable/foo.o /tmp/out/executable/bar.o /tmp/out/library/lib.a -o'/tmp/out/
executable/exec'
```

好的，关于顶层就讲到这里。如果我们进入`library`模块，就可以像构建或清理其他模块一样轻松地操作：

```
$ **cd /src/library**
$ **make -n clean**
rm -rf /tmp/out/library
$ **make -n**
cc  -c -o '/tmp/out/library/lib1.o' '/home/jgc/doc/nonrecursive/library/lib1.c'
cc  -c -o '/tmp/out/library/lib2.o' '/home/jgc/doc/nonrecursive/library/lib2.c'
ar r '/tmp/out/library/lib.a' /tmp/out/library/lib1.o /tmp/out/library/lib2.o
ranlib '/tmp/out/library/lib.a'
```

当然，在`executable`目录中这样做也会构建`library`：

```
$ **cd /src/executable**
$ **make -n**
cc  -c -o '/tmp/out/library/lib1.o' '/home/jgc/doc/nonrecursive/library/lib1.c'
cc  -c -o '/tmp/out/library/lib2.o' '/home/jgc/doc/nonrecursive/library/lib2.c'
ar r '/tmp/out/library/lib.a' /tmp/out/library/lib1.o /tmp/out/library/lib2.o
ranlib '/tmp/out/library/lib.a'
cc  -c -o '/tmp/out/executable/foo.o' '/home/jgc/doc/nonrecursive/executable/foo.c'
cc  -c -o '/tmp/out/executable/bar.o' '/home/jgc/doc/nonrecursive/executable/bar.c'
g++   /tmp/out/executable/foo.o /tmp/out/executable/bar.o /tmp/out/library/lib.a -o'/tmp/out/
executable/exec'
```

## 那么，子模块如何处理呢？

假设源代码树实际上是

```
/src/
/src/library/
/src/library/sublibrary
/src/executable/
```

其中在`library`下还有一个额外的`sublibrary`，该`sublibrary`使用以下 makefile 从`slib1.c`和`slib2.c`构建`slib.a`：

```
sp :=
sp +=
_walk = $(if $1,$(wildcard /$(subst $(sp),/,$1)/$2) $(call _walk,$(wordlist 2,$(words $1),x $1),$2))
_find = $(firstword $(call _walk,$(strip $(subst /, ,$1)),$2))
_ROOT := $(patsubst %/root.mak,%,$(call _find,$(CURDIR),root.mak))

include $(_ROOT)/root.mak
include $(_ROOT)/top.mak

SRCS := slib1.c slib2.c
BINARY := slib
BINARY_EXT := $(_LIBEXT)

include $(_ROOT)/bottom.mak
```

指定`library`依赖`sublibrary`非常简单，只需在`library`目录中的 makefile 里添加一个`DEPENDS_ON`调用：

```
sp :=
sp +=
_walk = $(if $1,$(wildcard /$(subst $(sp),/,$1)/$2) $(call _walk,$(wordlist 2,$(words $1),x $1),$2))
_find = $(firstword $(call _walk,$(strip $(subst /, ,$1)),$2))
_ROOT := $(patsubst %/root.mak,%,$(call _find,$(CURDIR),root.mak))

include $(_ROOT)/root.mak

$(call DEPENDS_ON,library/sublibrary)

include $(_ROOT)/top.mak

SRCS := lib1.c lib2.c
BINARY := lib
BINARY_EXT := $(_LIBEXT)

include $(_ROOT)/bottom.mak
```

在这个示例中，没有`DEPS`行，因此`library`在对象级别上并不依赖于`sublibrary`。我们只是声明`sublibrary`是`library`的一个子模块，当`library`构建时，`sublibrary`也需要被构建。

回顾并重复前面的示例，我们可以看到`sublibrary`已经成功地包含在`library`的构建中（并且自动包含在`executable`的构建中）。

这是从头开始的完整构建，接下来是一个`clean`操作：

```
$ **cd /src**
$ **make -n**
cc  -c -o '/tmp/out/library/sublibrary/slib1.o' '/home/jgc/doc/nonrecursive/library/sublibrary/
slib1.c'
cc  -c -o '/tmp/out/library/sublibrary/slib2.o' '/home/jgc/doc/nonrecursive/library/sublibrary/
slib2.c'
ar r '/tmp/out/library/sublibrary/slib.a' /tmp/out/library/sublibrary/slib1.o /tmp/out/library/
sublibrary/slib2.o
ranlib '/tmp/out/library/sublibrary/slib.a'
cc  -c -o '/tmp/out/library/lib1.o' '/home/jgc/doc/nonrecursive/library/lib1.c'
cc  -c -o '/tmp/out/library/lib2.o' '/home/jgc/doc/nonrecursive/library/lib2.c'
ar r '/tmp/out/library/lib.a' /tmp/out/library/lib1.o /tmp/out/library/lib2.o
ranlib '/tmp/out/library/lib.a'
cc  -c -o '/tmp/out/executable/foo.o' '/home/jgc/doc/nonrecursive/executable/foo.c'
cc  -c -o '/tmp/out/executable/bar.o' '/home/jgc/doc/nonrecursive/executable/bar.c'
g++   /tmp/out/executable/foo.o /tmp/out/executable/bar.o /tmp/out/library/lib.a -o'/tmp/out/
executable/exec'
$ **make -n clean**
rm -rf /tmp/out/library/sublibrary
rm -rf /tmp/out/library
rm -rf /tmp/out/executable
```

在这里，我们要求构建`sublibrary`：

```
$ **cd /src**
$ **make -n clean-library_sublibrary**
rm -rf /tmp/out/library/sublibrary
$ **make -n library_sublibrary**
cc -c -o '/tmp/out/library/sublibrary/slib1.o' '/home/jgc/doc/nonrecursive/library/sublibrary/
slib1.c'
cc -c -o '/tmp/out/library/sublibrary/slib2.o' '/home/jgc/doc/nonrecursive/library/sublibrary/
slib2.c'
ar r '/tmp/out/library/sublibrary/slib.a' /tmp/out/library/sublibrary/slib1.o /tmp/out/library/
sublibrary/slib2.o
ranlib '/tmp/out/library/sublibrary/slib.a'
```

如果我们要求构建`executable`模块，那么`library`会同时被构建（并且`sublibrary`也会被构建），因为有这个依赖关系：

```
$ **cd /src/executable**
$ **make -n executable**
cc  -c -o '/tmp/out/library/sublibrary/slib1.o' '/home/jgc/doc/nonrecursive/library/sublibrary/
slib1.c'
cc  -c -o '/tmp/out/library/sublibrary/slib2.o' '/home/jgc/doc/nonrecursive/library/sublibrary/
slib2.c'
ar r '/tmp/out/library/sublibrary/slib.a' /tmp/out/library/sublibrary/slib1.o /tmp/out/library/
sublibrary/slib2.o
ranlib '/tmp/out/library/sublibrary/slib.a'
cc  -c -o '/tmp/out/library/lib1.o' '/home/jgc/doc/nonrecursive/library/lib1.c'
cc  -c -o '/tmp/out/library/lib2.o' '/home/jgc/doc/nonrecursive/library/lib2.c'
ar r '/tmp/out/library/lib.a' /tmp/out/library/lib1.o /tmp/out/library/lib2.o
ranlib '/tmp/out/library/lib.a'
cc  -c -o '/tmp/out/executable/foo.o' '/home/jgc/doc/nonrecursive/executable/foo.c'
cc  -c -o '/tmp/out/executable/bar.o' '/home/jgc/doc/nonrecursive/executable/bar.c'
g++   /tmp/out/executable/foo.o /tmp/out/executable/bar.o /tmp/out/library/lib.a -o'/tmp/out/
executable/exec'
```

尽管这种非递归的系统比递归`make`更复杂，但它非常灵活。它允许模块之间的单独二进制文件之间存在依赖关系，而递归`make`无法做到这一点，并且它允许在不失去“去任何目录并输入`make`”这一工程师熟知的理念的情况下实现这一点。

GNU `make`功能非常强大（这也是它多年仍然存在的部分原因），但当项目变得庞大时，makefile 可能变得难以管理。通过本章所学的内容，你现在可以简化 makefile，解决 GNU `make`的不足，使大型项目变得更加简单和可靠。
