# 第十二章：预处理器

![](img/chapterart.png)

基本的 C 编译器具有许多强大的功能，但有些事情它就是做不到。为了克服这些限制，语言中增加了一个预处理器。预处理器主要是一个*宏处理器*，它是一个用其他文本替换文本的程序，但它也可以根据某些条件包含或排除文本并执行其他操作。这个概念是让一个程序（预处理器）完成一个小而简单的文本编辑任务，然后将其输入到真正的编译器中。由于这两个步骤（以及其他几个步骤）是隐藏在`gcc`命令后面的，你几乎不会去考虑它们，但它们的确存在。

例如，让我们看看以下代码：

```
#define SIZE 20    // Size of the array
int array[SIZE];   // The array
`--snip--`
    for (unsigned int i = 0; i < SIZE; ++i) {
```

当`SIZE`被定义为`20`时，预处理器实际上会对`SIZE`进行全局搜索并替换为`20`。

我们与 STM 微处理器一起使用的 HAL 库在几个方面广泛使用了预处理器。首先，头文件包含每个可读取和可设置的处理器位的`#define`，而且这些位相当多。其次，STMicroelectronics 并不只生产一种芯片；它生产各种各样的芯片。与其拥有 20 个不同的头文件来包含 20 个芯片的信息，不如使用一种叫做*条件编译*的过程，只编译需要的头文件部分。

## 简单宏

让我们从简单的宏开始。一个*宏*基本上是一个模式（在此例中是`SIZE`），它被替换成其他内容（在此例中是`20`）。`#define`预处理指令用来定义这个模式和替换内容：

**size.c**

```
#define SIZE 20
The size is SIZE
```

这不是一个 C 程序。预处理器可以处理任何内容，包括纯英文文本。让我们使用`-E`标志将其传递给预处理器，这个标志告诉`gcc`仅通过预处理器处理程序并停止：

```
$ **gcc -E size.c**
```

以下是预处理后的结果：

```
# 1 "size.c"
# 1 "<built-in>"
# 1 "<command-line>"
# 31 "<command-line>"
# 1 "/usr/include/stdc-predef.h" 1 3 4
# 32 "<command-line>" 2
1 # 1 "size.c"

2 The size is 20
```

以井号（`#`）开头的行叫做*行标记*。它们由一个井号、行号和文件名（以及一些其他信息）组成。由于预处理器可能会添加或删除行，没有它们，编译器无法知道它在原始输入文件中的位置。

很多事情发生在第一行处理之前，但最终我们会到达第二次出现的位置 1，并且输出 2 显示`SIZE`已经被替换为定义的值。

预处理器按字面意思处理事物，这可能会让你陷入麻烦，正如这里所示：

**square.c**

```
#include <stdio.h>

1 #define SIDE 10 + 2   // Size + margin

int main()
{
  2 printf("Area %d\n", SIDE * SIDE);
    return (0);
} 
```

这个例子计算一个正方形的面积。它包含了一些边距，因此正方形的边长定义为 1。为了计算面积，我们将边长相乘并打印结果 2。然而，这个程序包含一个 bug：`SIZE`不是`12`，而是`10 + 2`。预处理器只是一个简单的文本编辑器，它不理解 C 语法或算术。

通过将程序传递给预处理器，我们可以看到我们在哪里犯了错误：

**square.i**

```
# 5 "square.c"
int main()
{
    printf("Area %d\n", 10 + 2 * 10 + 2);
    return (0);
}
```

如前所述，预处理器并不理解 C 语言。当我们使用以下语句时，它会将`SIZE`定义为字面意义上的`10 + 2`，而不是`12`：

```
#define SIDE 10 + 2   // Size + margin
```

正如你所看到的，`12 * 12`与`10 + 2 * 10 + 2`是不同的数字。

当使用`#define`来定义比简单数字更复杂的常量时，我们将整个表达式用括号括起来，如下所示：

```
#define SIDE (10 + 2)   // Size + margin
```

遵循这种风格规则可以避免在替换后因操作顺序不确定而导致的错误结果。

为了避免在`#define`的目的是在一个地方设置或计算一个值并在程序中使用时出现宏计算错误，建议使用`const`，在可能的情况下，`const`应优于`#define`。这里是一个例子：

```
const unsigned int SIDE = 10 + 2;       // This works.
```

这个规则的主要原因是`const`修饰符是 C 语言的一部分，编译器会计算分配给`const`变量的表达式，所以`SIDE`实际上是`12`。

当 C 语言最初设计时，并没有`const`修饰符，因此每个人都必须使用`#define`语句，这也是为什么即使`const`已经使用了一段时间，`#define`依然如此广泛使用的原因。

### 带参数的宏

*带参数的* *宏* 允许我们为宏提供参数。这里是一个例子：

```
#define DOUBLE(x) (2 * (x))
`--snip--`
    printf("Twice %d is %d\n", 32, DOUBLE(32);
```

在这种情况下，我们不需要在展开时将括号括起来。我们可以按如下方式编写宏：

```
#define DOUBLE_BAD(x) (2 * x)
```

为什么这样不好呢？想想看当我们使用这个宏与一个表达式时会发生什么：

```
 value = DOUBLE_BAD(1 + 2)
```

风格规则是为带参数的宏的参数加上括号。如果没有括号，`DOUBLE(1+2)`将展开成如下内容：

```
DOUBLE(1+2) = (2 * 1 + 2) = 4   // Wrong
```

使用括号后，我们得到如下结果：

```
DOUBLE(1+2) = (2 * (1 + 2)) = 6
```

我们已经有一个规则，规定除了独立的一行外不要使用`++`或`--`。让我们看看当我们违反这个规则，使用带参数的宏时会发生什么：

```
#define CUBE(x) ((x) * (x) * (x))

    int x = 5;

    int y = CUBE(x++);
```

执行完这段代码后，`x`的值是多少？是`8`而不是预期的`6`。更糟的是，`y`的值可以是任何值，因为在 C 语言中，当混合使用乘法（`*`）和递增（`++`）操作时，执行顺序规则是不明确的。

如果你要编写这样的代码，考虑使用`inline`函数，它会用函数体替代函数调用：

```
static inline int CUBE_INLINE(const int x) {
    return (x * x * x);
}
```

即使使用以下语句，它也能正常工作：

```
y = CUBE_INLINE(x++);
```

但是，再次强调，你不应该编写这样的代码。相反，应该像这样编写代码：

```
x++;
y = CUBE_INLINE(x);
```

尽可能使用`inline`函数代替带参数的宏。因为`inline`函数是 C 语言的一部分，编译器可以确保它们被正确使用（与预处理器不同，预处理器只是盲目地替换文本）。

### 代码宏

到目前为止，我们一直在编写宏来定义常量和简单表达式。我们也可以使用`#define`来定义代码。这里是一个例子：

```
#define FOR_EACH_VALUE for (unsigned int i = 0; i < VALUE_SIZE; ++i)
`--snip--`
    int sum = 0;
    FOR_EACH_VALUE
        sum += value[i]
```

然而，这段代码存在一些问题。首先，变量`i`的来源不明确。我们还隐去了递增它的部分，这也是这种宏很少见的原因。

一个更常见的宏是模拟短函数的宏。让我们定义一个叫`DIE`的宏，它输出一条消息，然后终止程序：

```
// Defined badly
#define DIE(why)              \
    printf("Die: %s\n", why); \
    exit(99);
```

我们使用反斜杠（`\`）来将宏扩展到多行。我们可以像这样使用这个宏：

```
void functionYetToBeImplemented(void) {
    DIE("Function has not been written yet");
}
```

在这种情况下，它有效，这更多是运气而非设计的结果。问题是`DIE`看起来像一个函数，因此我们可以将其视为函数。让我们把它放入`if`语句中：

```
// Problem code
if (index < 0)
    DIE("Illegal index");
```

为了理解为什么这是个问题，让我们看看这段代码的展开结果：

```
if (index < 0)
   printf("Die %s\n", "Illegal index");
   exit(99); 
```

这是正确缩进后的代码：

```
if (index < 0)
    printf("Die %s\n", "Illegal index");
exit(99); 
```

换句话说，它总是会退出，即使索引是正确的。

让我们看看是否可以通过在语句周围加上花括号（`{}`）来解决这个问题：

```
// Defined not as badly
#define DIE(why) {            \
    printf("Die: %s\n", why); \
    exit(99);                 \
}
```

现在在以下情况下它能正常工作：

```
// Problem code
if (index < 0)
    DIE("Illegal index");
```

然而，在这种情况下它不起作用：

```
if (index < 0)
    DIE("Illegal index");
else
    printf("Did not die\n");
```

这段代码会产生一个错误信息：`else without previous if`。然而，我们这里确实有一个`if`。让我们看看展开后的结果：

```
if (index < 0)
{
    printf("Die: %s\n", why); \
    exit(99);                 \
};                 // <=== Notice two characters here.
else
    print("Did not die\n");
```

这里的问题是，在`else`之前，C 语言要求一个以分号（`;`）结尾的语句，*或者* 一组被花括号（`{}`）包围的语句。它不知道如何处理一组以分号结尾、且被花括号包围的语句。

解决这个问题的方法是使用一个叫做`do`/`while`的 C 语言语句。它的样子是这样的：

```
do {
   // Statements
}
while (`condition`);
```

`do`后面的语句总是执行一次，然后只要`condition`为真，就会继续执行。虽然它是 C 语言标准的一部分，但我只在实际应用中见过两次，而且其中一次还是作为笑话的结尾。

然而，它用于代码宏的场景：

```
#define DIE(why)
do {            \
    printf("Die: %s\n", why); \
    exit(99);                 \
} while (0)
```

它能正常工作，因为我们可以在后面加一个分号：

```
if (index < 0)
    DIE("Illegal index");   // Note semicolon at the end of the statement.
else
    printf("Did not die\n");
```

这段代码展开后的结果是：

```
if (index < 0)
    do {
        printf("Die: %s\n", "Illegal index");
        exit(99);
    } while (0);
else
    printf("Did not die\n");
```

从语法上讲，`do`/`while`是一个单一语句，我们可以在它后面加一个分号而不会有问题。花括号（`printf`和`exit`）中的代码被安全地封装在`do`/`while`内部。花括号外的代码是一条语句，这正是我们想要的。现在编译器会接受这个代码宏。

## 条件编译

条件编译使我们能够在编译时改变代码内容。这个功能的经典用途是拥有一个调试版本和一个生产版本的程序。

`#ifdef`/`#endif`指令对会在定义了某个符号的情况下编译两个指令之间的代码。这里是一个例子：

```
int main()
{
#ifdef DEBUG
    printf("Debug version\n");
#endif // DEBUG
```

严格来说，`// DEBUG`注释并不是必需的，但请确保包含它，因为匹配`#ifdef`/`#endif`对非常困难。

如果你的程序看起来像这样：

```
#define DEBUG   // Debug version

int main()
{
#ifdef DEBUG
    printf("Debug version\n");
#endif // DEBUG
```

那么，预处理后的结果将是如下：

```
int main()
{
    printf("Debug version\n");
```

另一方面，如果你的程序看起来像这样：

```
//#define DEBUG         // Release version

int main()
{
#ifdef DEBUG
    printf("Debug version\n");
#endif // DEBUG
```

那么，预处理后的结果将是如下：

```
int main()
{
    // Nothing
```

由于`DEBUG`没有定义，代码没有生成。

一个问题是，所有的`#ifdef`语句会使得程序看起来很杂乱。考虑下面的代码：

```
int main()
{
#ifdef DEBUG
    printf("Debug version\n");
#endif // DEBUG

#ifdef DEBUG
    printf("Starting main loop\n");
#endif // DEBUG

    while (1) {
#ifdef DEBUG
        printf("Before process file \n");
#endif // DEBUG
        processFile();
#ifdef DEBUG
        printf("After process file \n");
#endif // DEBUG
```

我们可以用更少的代码做同样的事情：

```
#ifdef DEBUG
#define debug(msg) printf(msg)
#else // DEBUG
#define debug(msg) /* nothing */
#endif // DEBUG

int main()
{
    debug("Debug version\n");
    debug("Starting main loop\n");

    while (1) {
        debug("Before process file \n");
        processFile();
        debug("After process file \n");
```

注意，我们使用了`#else`指令来告诉预处理器反转`#if`的判断逻辑。如果定义了`DEBUG`，则调用`debug`会被替换为调用`printf`；否则，它们将被替换为空白空间。在这种情况下，我们不需要`do`/`while`技巧，因为代码宏包含的是一个单独的函数调用（没有分号）。

另一个指令`#ifndef`在符号未定义时为真，其他情况下与`#ifdef`指令的用法相同。

## 符号的定义位置

我们可以通过三种方式定义符号：

1.  在程序内部通过`#define`

1.  从命令行

1.  预定义在预处理器内部

我们已经描述了在程序内部定义的符号，接下来我们来看看另外两种选项。

### 命令行符号

要在命令行中定义符号，请使用`-D`选项：

```
**$ gcc -Wall -Wextra -DDEBUG -o prog prog.c**
```

`-DDEBUG`参数定义了`DEBUG`符号，以便预处理器可以使用它。在这个例子中，它会在程序开始之前执行`#define DEBUG 1`。我们在前面的代码中使用了这个符号来控制是否编译`debug`语句。

除了符号之外，我们还需要手动添加到编译命令中，STM32 工作台会生成一个 makefile 来编译一个在命令行上定义了多个符号的程序。最重要的是通过`-DSTM32F030x8`选项定义的。*CMSIS/device/stm32f0xx.h*文件使用`STM32F030x8`符号来包含特定于板卡的文件：

```
#if defined(STM32F030x6)
  #include "stm32f030x6.h"
#elif defined(STM32F030x8)
  #include "stm32f030x8.h"
#elif defined(STM32F031x6)
  #include "stm32f031x6.h"
#elif defined(STM32F038xx)
```

STM 固件支持多种板卡，其中之一是 NUCLEO-F030R8。每个芯片的 I/O 设备位置不同。你不需要担心它们的位置，因为固件会使用前面的代码找到正确的位置。此文件的意思是：“如果我是一块 STM32F030x6 板卡，包含该板卡的头文件；如果我是一块 STM32F030x8 板卡，包含该板卡的头文件”，以此类推。

使用的指令是`#if`和`#elif`。`#if`用于测试后面的表达式是否为真（在这种情况下，测试`STM32F030x6`是否已定义）。如果为真，紧随其后的代码将被编译。`#elif`是`#else`和`#if`的组合，表示如果表达式不为真，则测试另一个表达式。另一个指令`defined`在符号已定义时为真。

### 预定义符号

最后，预处理器本身定义了多个符号，如`__VERSION__`（指定编译器版本）和`__linux`（在 Linux 系统中）。要查看系统中预定义的符号，可以使用以下命令：

```
$ **gcc -dM -E - < /dev/null**
```

`__cplusplus`符号仅在你编译 C++程序时定义。通常，你会在文件中看到类似如下的内容：

```
#ifdef   __cplusplus
extern "C"
{
#endif
```

这是 C++所需的一部分舞步，以便它可以使用 C 程序。现在可以忽略它。

## 包含文件

`#include`指令告诉预处理器将整个文件引入，仿佛它是原始文件的一部分。该指令有两种形式：

```
#include <file.h>
#include "file.h"
```

第一种形式引入系统头文件（即你使用的编译器或系统库附带的文件）。第二种形式引入你自己创建的文件。

头文件的一个问题是它们可能会被包含两次。如果发生这种情况，你会遇到很多重复定义的符号和其他问题。解决这个问题的方法是通过使用以下设计模式添加一个*哨兵*：

```
#ifndef __FILE_NAME_H__
#define __FILE_NAME_H__
// Body of the file
#endif __FILE_NAME_H__
```

第一次执行时，`__FILE_NAME_H__`符号（哨兵）没有被定义，因此整个头文件会被包含。这是好的，因为我们想要它被包含——*一次*。下次执行时，`__FILE_NAME_H__`已经定义，`#ifndef`会阻止其下方的代码被包含，直到文件末尾的`#endif`被执行。因此，尽管头文件被包含了两次，但文件的内容只会出现一次。

## 其他预处理器指令

一些小的预处理器指令也很有用，比如`#warning`、`#error`和`#pragma`。

`#warning`指令会在出现时显示编译器警告：

```
#ifndef PROCESSOR
#define PROCESSOR DEFAULT_PROCESSOR
#warning "No processor -- taking default"
#endif // PROCESSOR
```

相关的`#error`指令会发出错误，并停止程序的编译：

```
#ifndef RELEASE_VERSION
#error "No release version defined. It must be defined."
#endif // RELEASE_VERSION
```

`#pragma`指令定义了与编译器相关的控制。这里是一个例子：

```
// I wish they would fix this include file.
#pragma GCC diagnostic ignored "-Wmissing-prototypes"
#include "buggy.h"
#pragma GCC diagnostic warning "-Wmissing-prototypes"
```

这个 GCC 特定的`#pragma`会关闭缺失原型的警告，包含一个有问题的头文件，并重新打开警告。

## 预处理器技巧

预处理器是一个愚蠢的宏处理器，因此我们必须采用前面描述的一些样式规则，以避免出现问题。预处理器的强大功能还使我们能够执行一些有趣的技巧，来让我们的工作更加轻松。其中一个技巧是`enum`技巧，我们在第八章中讨论过。在这一节中，我们将讨论如何注释掉代码。

有时，我们需要禁用某些代码以进行测试。一种方法是注释掉代码。例如，假设审计过程有问题；我们可以禁用它，直到审计组修复问题为止。

这是原始代码：

```
int processFile(void) {
    readFile();
    connectToAuditServer();
    if (!audit()) {
        printf("ERROR: Audit failed\n");
        return;
    }
    crunchData();
    writeReport();
} 
```

这里是移除审计后的代码：

```
int processFile(void) {
    readFile();
//    connectToAuditServer();
//    if (!audit()) {
//        printf("ERROR: Audit failed\n");
//        return;
//    }
    crunchData();
    writeReport();
}
```

我们希望移除的每一行现在都以注释（`//`）标记开始。

然而，注释掉每一行是非常繁琐的。相反，我们可以使用条件编译来移除代码。我们所需要做的就是用`#ifdef UNDEF`和`#endif // UNDEF`语句将代码包围起来，像这样：

```
int processFile(void) {
    readFile();
#ifdef UNDEF
    connectToAuditServer();
    if (!audit()) {
        printf("ERROR: Audit failed\n");
        return;
    }
#endif // UNDEF
    crunchData();
    writeReport();
}
```

`#ifdef`/`#endif`块中的代码只有在定义了`UNDEF`时才会被编译，而没有理智的程序员会这么做。使用`#if 0` / `#endif`做同样的事情，而不依赖其他程序员的理智。

## 总结

C 预处理器是一个简单而强大的自动化文本编辑器。如果使用得当，它可以大大简化编程。它允许你定义简单的数值宏以及小的代码宏。（实际上，你也可以定义大的代码宏，但你真的不想那样做。）

它的一个主要特性是`#include`指令，它方便了模块之间接口的共享。此外，`#ifdef`功能使你能够通过条件编译编写一个具有多种功能的程序。

然而，你必须记住，预处理器并不理解 C 语法。因此，你必须记住一些样式规则和编程模式，才能有效地使用该系统。

尽管有很多限制和怪癖，预处理器在创建 C 程序时仍然是一个强大的工具。

## 编程问题

1.  编写一个宏来交换两个整数。

1.  高级：编写一个宏来交换任意类型的两个整数。（在做这件事之前，先阅读 GCC 的`typeof`关键字文档。）

1.  创建一个名为`islower(x)`的宏，如果`x`是小写字母，则返回 true。

1.  疯狂高级：弄清楚程序*zsmall.c*是如何工作的（[`www.cise.ufl.edu/~manuel/obfuscate/zsmall.hint`](https://www.cise.ufl.edu/~manuel/obfuscate/zsmall.hint)）。这个程序是模糊 C 竞赛的获奖作品（它获得了“最佳滥用预处理器”奖）。它所做的仅仅是打印一个素数列表，但所有的计算和循环都是通过预处理器完成的。
