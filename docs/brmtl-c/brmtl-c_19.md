# 第十七章：模块化编程

![](img/chapterart.png)

到目前为止，我们一直在处理简单的小型单文件程序，如果你只是为了写书中的示例程序，这样是可以的。然而，在现实世界中，你可能会遇到包含超过 50 行代码的程序。

Linux 内核有 33,000 个文件和 2800 万行代码（而且这些数字还在不断增加，随着你阅读本文，它们会继续增加）。你不可能在没有将代码组织成*模块*的情况下处理这么大量的信息。

理想的模块是一个包含数据和函数集合的单一文件，它能够做好一件事，并且与其他模块的交互最小化。我们之前在本书中使用过 STM HAL 模块集合，包括包含`HAL_Init`函数的模块。它在内部做了大量的工作，但我们从未看到它。我们只看到了一个做一件事的简单模块：它初始化所有需要的硬件，使其能够工作。

## 简单模块

让我们创建一个使用两个文件的程序。主程序将被命名为*main.c*（见清单 17-1），并将调用*func.c*文件中的一个函数（见清单 17-2）。我们将使用一个 makefile（见清单 17-3）将这两个文件编译成一个程序。

**main.c**

```
/**
 * Demonstrate the use of extern.
 * @note: Oversimplifies things.
 */
#include <stdio.h>

1 extern void funct(void);        // An external function

int main()
{
    printf("In main()\n");
    funct();
    return (0);
}
```

清单 17-1：主程序

第一个需要注意的点是`funct`函数在*main.c*中的声明。`extern`关键字告诉 C 语言这个函数在另一个名为*func.c*的文件中定义。清单 17-2 包含了该文件。

**func.c**

```
/**
 * Demonstration of a function module
 */
#include <stdio.h>
/**
 * Demonstration function
 */
1 void funct(void)
{
    printf("In funct()\n");
}
```

清单 17-2：定义该函数的文件

`funct`函数在*func.c*文件中定义，并且清单 17-3 中的 makefile 处理编译。

```
main: main.c func.c
      gcc -g -Wall -Wextra -o main main.c func.c
```

*清单 17-3：simple/Makefile*

makefile 的第一行告诉`make`，如果*main.c*或*func.c*发生变化，目标`main`必须重新构建。第二行告诉`make`，当其中一个文件发生变化时，它应该编译两个文件并用它们来生成程序。

在这个例子中，我们将一个函数和一个主程序放在了两个不同的文件中。然后我们告诉`make`让编译器将它们组合成一个程序。这是模块化编程的一个过于简化的版本，但这些基本原则在更复杂的程序中也有应用，尤其是那些包含更多和更复杂模块的程序。

### 简单模块的问题

前面的例子有一些问题。第一个问题是相同的信息被重复了两次。

在*main.c*中，我们有以下内容：

```
extern void funct(void);        // An external function
```

在*func.c*中，我们有以下内容：

```
void funct(void)
```

这意味着如果我们更改一个文件，我们必须更改另一个文件。

更糟糕的是，*C 不检查跨文件的类型*，这意味着可能会有如下代码：

```
// File a.c
extern uint32_t flag;    // A flag
```

并且：

```
// File b.c
int16_t flag;        // A flag
```

在两个不同的文件中。假设文件 *a.c* 决定将 `flag` 设置为零。程序将会把 32 位的零存储到 `flag` 中，而 `flag` 在文件 *b.c* 中定义为只有 16 位长。实际发生的情况是，16 位将被存储到 `flag` 中，另 16 位将存储到其他地方。结果是，程序将发生意外的、令人惊讶的并且难以调试的问题。

在一个文件中可以声明一个变量为 `extern`，并在之后的地方不再使用 `extern` 声明。C 会检查确保 `extern` 定义中的类型与实际声明中的类型匹配：

```
#include <stdint.h>
extern uint32_t flag;     // A flag
int16_t flag;             // A flag
```

编译此代码将导致错误：

```
16.bad.c:3:9: error: conflicting types for 'flag'
 int16_t flag;  // A flag

16.bad.c:2:17: note: previous declaration of 'flag' was here
 extern uint32_t flag; // A flag
```

关于第二个问题，假设我们想要在多个文件中使用我们的外部函数 `funct`。我们是否希望在每个文件中都添加一个 `extern` 声明？那样的话，`funct` 的定义将在多个地方重复出现（而且不会被编译器检查）。

解决方法是创建一个头文件来保存 `extern` 定义。清单 17-4 包含了这个文件。

**func.h**

```
#ifndef __FUNC_H__
#define __FUNC_H__
1 extern void funct(void);
#endif // __FUNC_H__
```

清单 17-4：定义函数的文件

除了函数定义 1，这个文件还包含了 *双重包含保护*。`#findef`/`#endif` 对可以防止程序出现类似以下的情况：

```
#include "func.h"
#include "func.h"
```

这将导致 *func.h* 中的定义被定义两次，这对于 `extern` 声明并不是问题，但如果涉及到多个 `#define` 实例，编译器会感到困扰。

这个例子看起来有些傻，因为在实际程序中，问题并不是那么显而易见。你可能会遇到类似以下的情况：

```
#include "database.h"
#include "service.h"
```

但是 *database.h* 文件包含了 *direct_io.h* 文件，后者又包含了 *func.h*，而 *service.h* 文件包含了 *network.h* 文件，后者也包含了 *func.h*。你会发现 *func.h* 被包含了两次，尽管你是通过绕远路去做的。

这些示例中 `#include` 语句的格式略有变化，不再是：

```
#include <file.h>
```

它是这样的：

```
#include "file.h"
```

引号表示要包含的文件是用户生成的文件。编译器将会在当前目录中查找它，而不是通过系统文件进行查找。

清单 17-5 包含了改进版的 *main.c*，它使用包含文件来引入 `extern` 声明。

**good/main.c**

```
/**
 * Demonstrate the use of extern.
 */
#include <stdio.h>
 #include "func.h"

int main()
{
    printf("In main()\n");
    funct();
    return (0);
}
```

清单 17-5：改进版 *main.c*

清单 17-6 包含改进版的 *func.c*，该文件包括 *func.h*。`extern` 在 *func.h* 中定义的函数实际上并不需要用来编译 *func.c*，但是通过引入它们，我们可以确保 `extern` 与实际的函数声明相匹配。

**good/func.c**

```
/**
 * Demonstration of a function module
 */
#include <stdio.h>
#include "func.h"
/**
 * Demonstration function
 */
void funct(void)
{
    printf("In funct()\n");
}
```

清单 17-6：改进版 *func.c*

通过将 *func.h* 文件包含两次，我们解决了 `extern` 与实际声明不匹配时可能发生的问题。在 *func.c* 中包含它可以让编译器检查函数定义，而在 *main.c* 中包含它则为我们提供了函数的定义。

### 构建模块

这个程序的 makefile 也发生了变化（见 Listing 17-7）。

```
CFLAGS = -g -Wall -Wextra

OBJS = main.o func.o

1 main: $(OBJS)
        gcc -g -Wall -Wextra -o main $(OBJS)

main.o: main.c func.h

func.o: func.c func.h
```

Listing 17-7: 改进版*Makefile*

第一行定义了一个名为`CFLAGS`的宏，这是编译 C 程序时使用的特定名称。下一行定义了另一个名为`OBJS`的宏（这个名称没有特殊含义），它包含了我们用来生成程序的对象列表。在这个例子中，我们将编译*main.c*为*main.o*目标文件，并将*func.c*编译为*func.o*目标文件。

我们在这里使用宏是为了避免在下一个规则 1 中重复写出列表，这个规则告诉`make`从*main.c*和*func.h*创建*main.o*。然而，这个规则后面并没有跟着一个规则来告诉`make`*如何*做。当`make`没有规则时，它会回退到内置规则的列表中。当我们从*.c*文件创建*.o*（或.*obj*）文件时，那个内置规则是：

```
$(CC) $(CFLAGS) -c `file.c`
```

其中`CC`是包含 C 编译器名称的宏（在这里是`cc`，它是`gcc`的别名）。

这个例子展示了一个简单的模块化程序，但当程序有更多模块时，设计模式同样适用。

## 什么构成好的模块

以下列表列出了制作好模块的一些规则：

+   每个模块应该有一个与模块同名的头文件。该文件应包含该模块中公共类型、变量和函数的定义（并且没有其他内容）。

+   每个模块应包含自己的头文件，以便 C 可以检查头文件和实现是否匹配。

+   模块应包含用于共同目的的代码，并且它们应该向外界暴露最少的信息。它们通过`extern`声明暴露的信息是全局的（程序中的所有部分都可以看到），正如下一节所述，这有时会成为一个问题。

## 命名空间

C 语言的一个问题是它没有命名空间。例如，在 C++中，你可以告诉编译器某个模块中的所有符号都属于`db`命名空间，这样你就可以创建一个模块，其中的条目如`insert`、`delete`和`query`，在其他人看来分别就是`db::insert`、`db::delete`和`db::query`。

在 C 中，如果你定义了一个名为`Init`的公共函数，其他人不能在任何模块中再定义一个名为`Init`的函数。如果发生这种情况，链接器会抱怨重复的符号。由于可能有多个项目需要初始化，这可能会成为一个问题。

大多数程序员通过为每个公共函数、类型或变量添加模块前缀来解决这个问题。你可以在 Nucleo 项目中自动添加的 HAL 库中看到这一点。例如，如 Listing 17-8 所示，所有操作 UART 的函数都以`UART_`前缀开头。

```
HAL_StatusTypeDef UART_CheckIdleState(UART_HandleTypeDef *huart);
HAL_StatusTypeDef UART_SetConfig(UART_HandleTypeDef *huart);
HAL_StatusTypeDef UART_Transmit_IT(UART_HandleTypeDef *huart);
HAL_StatusTypeDef UART_EndTransmit_IT(UART_HandleTypeDef *huart);
HAL_StatusTypeDef UART_Receive_IT(UART_HandleTypeDef *huart);
```

Listing 17-8: *stm32f0xx_hal_uart.h*的摘录

这里的关键是 HAL 库中的公共符号以 `HAL_` 开头，这使得我们可以轻松判断一个函数是否属于该库。它还确保你不会不小心使用已经被 HAL 库占用的名称。

## 库

当程序中包含的文件少于 20 个时，列出每个文件还不算太麻烦。但一旦文件数量超过 20 个，就会显得有些繁琐，不过还是能管理的，直到数量变得非常庞大。我们一直在编写的主机程序使用的是标准 C 库函数。C 库有超过 1,600 个文件。幸运的是，我们在编译程序时不需要列出所有这些文件。

标准 C 库是一个名为 *libc.a* 的文件，在程序链接时会自动加载。这个库是由多个目标文件组成的，采用简单的归档格式（因此有 *.a* 后缀）。

让我们创建一个包含多个模块的库，用来计算不同类型数字的平方。列表 17-9 展示了一个计算浮点数平方的函数。

**square_float.c**

```
#include "square_float.h"

/**
 * Square a floating-point number.
 *
 * @param number Number to square
 * @returns The square of the number
 */
float square_float(const float number) {
    return (number * number);
}
```

列表 17-9：一个用于计算浮点数平方的函数

列表 17-10 是该模块的头文件。

**square_float.h**

```
#ifndef __SQUARE_FLOAT_H__
#define __SQUARE_FLOAT_H__
extern float square_float(const float number);
#endif // __SQUARE_FLOAT_H__
```

列表 17-10：*square_float.c* 模块的头文件

列表 17-11 定义了一个用于计算整数平方的函数。

**square_int.c**

```
#include "square_int.h"

/**
 * Square an integer.
 *
 * @param number Number to square
 * @returns The square of the number
 */
int square_int(const int number) {
    return (number * number);
}
```

列表 17-11：一个用于计算整数平方的函数

列表 17-12 定义了它的头文件。

**square_int.h**

```
#ifndef __SQUARE_INT_H__
#define __SQUARE_INT_H__
extern int square_int(const int number);
#endif // __SQUARE_INT_H__
```

列表 17-12：*square_int.c* 的头文件

接下来，列表 17-13 是一个类似的函数，用于计算无符号整数的平方。

**square_unsigned.c**

```
#include "square_unsigned.h"

/**
 * Square an unsigned integer.
 *
 * @param number Number to square
 * @returns The square of the number
 */
unsigned int square_unsigned(const unsigned int number) {
    return (number * number);
}
```

列表 17-13：一个用于计算无符号整数平方的函数

列表 17-14 定义了该头文件。

**square_unsigned.h**

```
#ifndef __SQUARE_UNSIGNED_H__
#define __SQUARE_UNSIGNED_H__
extern unsigned int square_unsigned(const unsigned int number);
#endif // __SQUARE_UNSIGNED_H__
```

列表 17-14：*square_unsigned.c* 的头文件

我们将把这三个函数放入一个库中。如果用户想使用这个库，他们需要包含所有这些头文件。这将是一项繁重的工作。

为了简化操作，我们将为这个库创建一个名为 *square.h* 的头文件。这个文件整合了前面各个库组件（模块）的独立头文件。因此，使用这个库的人只需要包含 *square.h*（请参见 列表 17-15），而不需要包含一堆单独的头文件。

**square.h**

```
#ifndef __SQUARE_H__
#define __SQUARE_H__
#include "square_float.h"
#include "square_int.h"
#include "square_unsigned.h"
#endif // __SQUARE_H__
```

列表 17-15：库的头文件

我们现在遵循了每个程序文件一个头文件的风格规则，以及库接口应尽可能简单的风格规则。

接下来，让我们为库创建一个小的测试程序（请参见 列表 17-16）。

**square.c**

```
/**
 * Test the square library.
 */
#include <stdio.h>

#include "square.h"

int main()
{
   printf("5 squared is %d\n", square_int(5));
   printf("5.3 squared is %f\n", square_float(5.3));
   return (0);
}
```

列表 17-16：库的测试程序

请注意，我们并没有测试库中的所有成员（这一点稍后会很重要）。

现在我们已经有了库的源文件，接下来需要将它们转化为实际的库。如前所述，库是一个归档格式的目标文件集合，类似于一个 *.zip* 文件，只不过没有压缩。

在这种情况下，我们将通过 *square_float.o*、*square_int.o* 和 *square_unsigned.o* 文件创建 *libsquare.a* 文件（即库文件本身）。

`make` 程序非常智能，能够更新归档文件的组件。例如，*libsquare.a* 的一个组件是 *square_int.o*。以下规则将其作为库的一个组件：

```
libsquare.a(square_int.o): square_int.o
        ar crU libsquare.a square_int.o
```

第一行告诉 `make` 我们正在创建或更新 *libsquare.a* 库中的 *square_int.o* 组件。此组件依赖于 *square_int.o* 目标文件。

第二行是实际添加库的命令。`c` 选项告诉 `ar` 在归档文件不存在时创建它。`r` 使得 `ar` 在归档文件中创建或替换 *square_int.o* 组件。`U` 标志告诉 `ar` 以非确定性模式运行，这样会将文件的创建时间存储在归档文件中（我们将在本章后面讨论确定性模式和非确定性模式）。该命令之后是库的名称 (*libsquare.a*) 和要添加或替换的组件名称 (*square_int.o*)。链接器设置了命名规范，库文件名必须以 *lib* 开头，以 *.a* 结尾（更多关于命名规范的内容将在后面讨论）。

接下来，使用以下指令，我们告诉 `make` 应该用哪些组件来构成 *libsquare.a* 库：

```
libsquare.a: libsquare.a(square_int.o) \
        libsquare.a(square_float.o)

libsquare.a(square_unsigned.o)
        ranlib libsquare.a
```

前两行告诉 `make` 用哪些组件来创建 *libsquare.a*。第三行，`ranlib libsquare.a`，告诉 `make` 在安装完所有组件后运行名为 `ranlib` 的程序，生成归档文件的目录表。

### ranlib 和库链接

我们使用 `ranlib` 的原因是早期的链接器。假设我们有一个包含 *a.o*（定义了 `a_funct`）、*b.o*（定义了 `b_funct`）和 *c.o*（定义了 `c_funct`）的归档文件，而程序需要 *b.o* 中的某个函数。链接器将打开归档文件，并按顺序检查每个成员是否需要，决策过程如下：

1.  查看未定义符号的列表（程序使用了 `b_funct`，所以它未定义）。

1.  打开归档文件。

1.  查看 *a.o*。它定义了需要的符号吗？没有。不要加载它。

1.  查看 *b.o*。它定义了需要的符号吗？是的。加载它。

1.  查看 *c.o*。它定义了需要的符号吗？没有。不要加载它。

现在假设 *b.o* 需要 `a_funct` 函数。链接器不会回过头重新检查归档文件，而是继续查看 *c.o*。由于 *c.o* 没有定义该符号，因此不会被加载。链接器会到达归档文件的末尾并中止，因为它没有找到满足 `a_funct` 需求的目标文件。

由于链接器的工作方式，有时你需要指定同一个库两次或三次。为了解决这个问题，归档文件中增加了目录表，以便组件可以随机顺序加载（因此有了 `ranlib` 的名字）。

现在加载组件的算法如下：

1.  查看未定义符号的列表（程序使用了`b_funct`，所以它是未定义的）。

1.  打开档案。

1.  我们是否有一个未定义的符号在目录中？

1.  如果有，加载它。

1.  重复此过程，直到我们没有更多可以由此库满足的符号。

这个过程解决了排序问题，因为目录使得一切都能被访问。

以下命令实际上将库与我们的程序链接在一起：

```
square: square.o libsquare.a
        $(CC) $(CFLAGS) -o square square.o -L. -lsquare
```

`-L.`选项告诉链接器在当前目录（` .`）中查找库文件。否则，它只会搜索系统库目录。库本身通过`-lsquare`指令来指定。链接器首先在当前目录（因为有`-L.`）查找名为*libsquare.a*的库，然后在系统目录中查找。

清单 17-17 显示了这个项目的完整 makefile。

```
CFLAGS=-g -Wall -Wextra

all: square

square: square.o libsquare.a
        $(CC) $(CFLAGS) -o square square.o -L. -lsquare

libsquare.a: libsquare.a(square_int.o) \
        libsquare.a(square_float.o) libsquare.a(square_unsigned.o)
        ranlib libsquare.a

libsquare.a(square_int.o): square_int.o
       ar crU libsquare.a square_int.o

libsquare.a(square_float.o): square_float.o
        ar crU libsquare.a square_float.o

libsquare.a(square_unsigned.o): square_unsigned.o
        ar crU libsquare.a square_unsigned.o

square_int.o: square_int.h square_int.c

square_float.o: square_float.h square_float.c

square_unsigned.o: square_unsigned.h square_unsigned.c

square.o: square_float.h square.h square_int.h square_unsigned.h square.c
```

*清单 17-17：完整的 makefile*

因为我们的测试程序没有调用`square_unsigned`，所以*square_unsigned.o*模块将不会被链接到我们的程序中。（为了演示链接器如何不链接不需要的目标文件，省略了对`square_unsigned`的测试。）

### 确定性库与非确定性库

理想情况下，如果你运行`make`命令，生成的二进制文件应该是相同的，无论命令在什么时候执行。出于这个原因，最初库文件并没有存储有关谁创建了组件或何时创建的信息。

然而，这给`make`程序带来了一些困难。如果档案不存储修改日期，`make`程序如何确定档案中的*square_int.o*版本是比你刚编译的版本更新还是更旧呢？

`ar`命令被修改以存储这些信息。因为这个功能破坏了传统功能，`ar`的维护者决定让存储这些信息成为可选项。如果你指定`D`选项，修改时间不会被存储，并且你将得到一个*确定性档案*（每次都是相同的二进制文件）。如果你指定`U`选项表示*非确定性*，你每次都会得到一个不同的二进制文件，但这是`make`程序更喜欢的。默认值是`D`，即传统格式。

## 弱符号

到目前为止，我们已经定义了总是被加载的具有函数和变量的模块。换句话说，如果一个模块定义了一个`doIt`函数，那就是唯一被加载的函数定义。GCC 和大多数其他编译器提供的 C 语言扩展允许使用*弱符号*。弱符号告诉链接器，“如果没有其他人定义这个符号，就使用我。”

一个使用弱符号的例子是 STM 中断表。你*必须*为每个可能的中断定义要调用的函数；硬件要求这样做。因此，你必须为那些从不发生的中断编写中断处理程序。由于该函数永远不会被调用，这应该让事情变得简单。

然而，STM 固件的设计理念是，尽管禁用中断的中断路由*应该*永远不会被调用，但这并不意味着它们*永远*不会被调用。STM 固件为所有会让系统崩溃的中断定义了中断处理程序。如果它们真的被调用，您的系统会停止，您有机会使用调试器进行分析并尝试找出原因。

唯一的方式使默认的中断处理程序被调用是如果您开启了中断并且没有提供自己的中断处理程序。在这种情况下，默认的中断处理程序会知道出了问题，并且静静地等待您来找出问题所在。

STM 的 USART2 中断处理程序是`USART2_IRQHandler`，它的定义如下：

```
void USART2_IRQHandler(void) {
    while(true)
        continue;
}
```

然而，如果我们自己定义了该函数，固件库中的`sub2`函数将消失，尽管同一模块中的其他约 40 个中断函数仍然会被加载。

让我们通过自己的代码来看这个过程。在清单 17-18 和 17-19 中，我们有`sub1`和`sub2`，而`sub2`被定义了两次（一次在*main.c*中，另一次在*sub.c*中）。当链接器查看这两个文件时，它会说：“这里有两个`sub2`函数。我应该抛出错误吗？不，一个是弱符号，我可以把它丢弃。”在*main.c*中的`sub2`将被链接，而在*sub.c*中的`sub2`则不会。

首先让我们定义一个主程序，它的任务是调用我们的两个子例程（参见清单 17-18）。

**main.c**

```
#include "sub.h"

int main()
{
    sub1();
    sub2();
    return (0);
}
```

清单 17-18：调用两个子例程的主程序

在清单 17-19 中，我们通过 GCC 扩展告诉编译器`sub2`在*sub.c*中是弱符号。

**sub.c**

```
#include "sub.h"

void sub2(void) __attribute__((weak));

void sub1(void) {}
void sub2(void) {} 
```

清单 17-19：告诉编译器`sub2`是弱符号

接下来，我们需要一个头文件，因此让我们创建一个（参见清单 17-20）。

**sub.h**

```
#ifndef __SUB_H__
#define __SUB_H__
extern void sub1(void);
extern void sub2(void);
#endif // __SUB_H__
```

清单 17-20：头文件

最后，我们在清单 17-21 中定义了我们自己的`sub2`函数。

**sub2.c**

```
#include <stdio.h>
#include "sub.h"

void sub2(void) {
    printf("The non-weak sub2\n");
}
```

清单 17-21：定义`sub2`函数

如果我们链接*main.c*和*sub.c*，弱符号`sub2`将被链接进来。如果我们链接*main.c*、*sub.c*和*sub2.c*，则会使用在*sub2.c*中定义的非弱版本。

这对于像中断例程这样的情况非常有用，您必须定义一个，无论是否使用它。这让您能够提供一个回退或默认版本。

## 总结

模块使您能够将大型程序拆分成可管理的单元。良好的设计意味着大型程序不需要包含庞大的部分。多个模块可以组织成一个库。库的优势在于它可以包含大量的专用模块，链接器只会链接需要的模块。

良好的编程就是组织信息，而模块和库使您能够将庞大的程序乱象整理成可管理的小单元。

## 编程问题

1.  编写一个库来计算几何形状的面积（`rectangle_area`、`triangle_area` 等）。每个函数应在自己的目标文件中，所有面积计算函数应合并成一个单独的库。编写一个主程序对这些函数进行单元测试。

1.  重写前面章节中创建的一个串行输出程序，使所有与 UART 相关的代码都在一个独立的模块中。

1.  测试一下发生了什么，当：

    1.  你定义了两个弱符号和一个强符号。

    1.  你定义了两个弱符号，没有强符号。

    1.  你定义了两个强符号。
