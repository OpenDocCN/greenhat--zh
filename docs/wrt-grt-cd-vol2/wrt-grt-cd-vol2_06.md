# 第六章：**常量与高级语言**

![image](img/common01.jpg)

一些程序员可能没有意识到，但许多 CPU 在机器代码级别并不会将常量和变量数据视为相同。大多数 CPU 提供一种特殊的*立即寻址模式*，允许语言翻译器将常量值直接嵌入机器指令中，而不是将其存储在内存位置并作为变量进行访问。然而，CPU 表示常量数据的能力在不同 CPU 之间有所不同，事实上，甚至不同类型的数据也有差异。通过了解 CPU 如何在机器代码级别处理常量数据，你可以选择合适的方式在高级语言（HLL）源代码中表示常量，从而生成更小、更快的可执行程序。为此，本章将讨论以下主题：

+   如何正确使用字面常量来提高程序的效率

+   字面常量和显式常量之间的区别

+   编译器如何处理编译时常量表达式，以减少程序大小并避免运行时计算

+   编译时常量与存储在内存中的只读数据之间的区别

+   编译器如何表示非整数常量，如枚举数据类型、布尔数据类型、浮点常量和字符串常量

+   编译器如何表示复合数据类型常量，如数组常量和记录/结构常量

当你读完本章时，你应该清楚了解各种常量如何影响编译器生成的机器代码的效率。

**注意**

*如果你已经读过* WGC1，*你可能只想浏览一下本章，完整性考虑它重复了一些第六章和第七章中的信息。*

### 6.1 字面常量与程序效率

高级编程语言和大多数现代 CPU 允许你在几乎任何可以合法读取内存变量值的地方指定常量值。考虑以下 Visual Basic 和 HLA 语句，它们将常量 `1000` 赋值给变量 `i`：

```

			i = 1000

mov( 1000, i );
```

80x86 与大多数 CPU 一样，实际上将常量值 1,000 直接编码到机器指令中。这提供了一种紧凑且高效的方式在机器级别处理常量。因此，以这种方式使用字面常量的语句通常比那些将常量值赋给变量再在代码中引用该变量的语句更高效。考虑以下 Visual Basic 代码序列：

```

			oneThousand = 1000
    .
    .
    .
x = x + oneThousand 'Using "oneThousand" rather than
                    ' a literal constant.
y = y + 1000        'Using a literal constant.
```

现在考虑你可能为这两个语句编写的 80x86 汇编代码。对于第一个语句，我们必须使用两条指令，因为我们不能将一个内存位置的值直接加到另一个内存位置上：

```

			mov( oneThousand, eax ); // x = x + oneThousand
add( eax, x );
```

但我们可以将常量添加到内存位置，因此第二条 Visual Basic 语句会转化为一个单独的机器指令：

```
add( 1000, y ); // y = y + 1000
```

如你所见，使用字面常量而不是变量更为高效。然而，这并不是说每个处理器在使用字面常量时都更高效，或者每个 CPU 在无论常量值如何时都更高效。一些非常老旧的 CPU 根本不支持将字面常量嵌入到机器指令中；而许多 RISC 处理器，如 ARM，只有在较小的 8 位、12 位或 16 位常量的情况下才支持此操作。^(1) 即便是那些允许加载任何整数常量的 CPU，也可能不支持字面浮点常量——例如广泛使用的 80x86 处理器就是一个例子。很少有 CPU 提供将大数据结构（如数组、记录或字符串）编码为机器指令一部分的能力。例如，考虑以下 C 代码：

```

			#include <stdlib.h>
#include <stdio.h>
int main( int argc, char **argv, char **envp )
{
  int i,j,k;

  i = 1;
  j = 16000;
  k = 100000;
  printf( "%d, %d, %d\n", i, j, k );

}
```

通过 GCC 编译器将其编译为 PowerPC 汇编代码的过程如下所示（已编辑以去除不相关的代码）：

```

			L1$pb:
    mflr r31
    stw r3,120(r30)
    stw r4,124(r30)
    stw r5,128(r30)

; The following two instructions copy the value 1 into the variable "i"

    li r0,1
    stw r0,64(r30)

; The following two instructions copy the value 16,000 into the variable "j"

    li r0,16000
    stw r0,68(r30)

; It takes three instructions to copy the value 100,000 into variable "k"

    lis r0,0x1
    ori r0,r0,34464
    stw r0,72(r30)

; The following code sets up and calls the printf function:

    addis r3,r31,ha16(LC0-L1$pb)
    la r3,lo16(LC0-L1$pb)(r3)
    lwz r4,64(r30)
    lwz r5,68(r30)
    lwz r6,72(r30)
    bl L_printf$stub
    mr r3,r0
    lwz r1,0(r1)
    lwz r0,8(r1)
    mtlr r0
    lmw r30,-8(r1)
    blr
```

PowerPC CPU 在单个指令中仅允许 16 位立即数常量。为了将更大的值加载到寄存器中，程序必须首先使用`lis`指令将 32 位寄存器的高 16 位（HO）加载，然后使用`ori`指令将低 16 位（LO）合并进来。这些指令的具体操作并不太重要。值得注意的是，编译器对于大常量发出三条指令，而对于较小的常量只发出两条指令。因此，在 PowerPC 上使用 16 位常量值会生成更短且更快速的机器代码。

通过 GCC 编译器将此 C 代码编译为 ARMv7 汇编代码的过程如下所示（已编辑以去除不相关的代码）：

```

			.LC0:
    .ascii  "i=%d, j=%d, k=%d\012\000"
    .text
    .align  2
    .global main
    .type   main, %function
main:
    @ args = 0, pretend = 0, frame = 24
    @ frame_needed = 1, uses_anonymous_args = 0
    stmfd   sp!, {fp, lr}
    add fp, sp, #4
    sub sp, sp, #24
    str r0, [fp, #-24]
    str r1, [fp, #-28]

; Store 1 into 'i' variable:

    mov r3, #1
    str r3, [fp, #-8]
@ Store 16000 into 'j' variable:

    mov r3, #16000
    str r3, [fp, #-12]

@ Store 100,000 (constant appears in memory) into 'k' variable:

    ldr r3, .L3
    str r3, [fp, #-16]

@ Fetch the values and print them:

    ldr r0, .L3+4
    ldr r1, [fp, #-8]
    ldr r2, [fp, #-12]
    ldr r3, [fp, #-16]
    bl  printf
    mov r3, #0
    mov r0, r3
    sub sp, fp, #4
    @ sp needed
    ldmfd   sp!, {fp, pc}
.L4:

@ constant value for k appears in memory:

    .align  2
.L3:
    .word    100000
    .word   .LC0
```

ARM CPU 在单个指令中仅允许 16 位立即数常量。为了将更大的值加载到寄存器中，编译器将常量放置到内存位置，并从内存中加载常量。

即便像 80x86 这样的 CISC 处理器通常可以在单条指令中编码任何整数常量（最多 32 位），这并不意味着程序的效率与程序中使用的常量大小无关。CISC 处理器通常对具有大或小立即操作数的机器指令使用不同的编码方式，从而使程序在处理较小常量时能使用更少的内存。例如，考虑以下两个 80x86/HLA 机器指令：

```

			add( 5, ebx );
add( 500_000, ebx );
```

在 80x86 上，汇编器可以用 3 个字节来编码第一条指令：2 个字节用于操作码和寻址模式信息，1 个字节用于保存小的立即常量`5`。另一方面，第二条指令需要 6 个字节来编码：2 个字节用于操作码和寻址模式信息，4 个字节用于保存常量`500_000`。显然，第二条指令更大，在某些情况下，它甚至可能运行得稍微慢一些。

### 6.2 绑定时间

常量究竟是什么？显然，从高级语言（HLL）的角度来看，常量是某种值不变的实体（即保持恒定）。然而，定义中还有更多内容。例如，考虑以下 Pascal 常量声明：

```

			const someConstant:integer = 5;
```

在此声明后的代码中，^(2) 你可以用名字 someConstant 替代值 `5`。但在此声明之前呢？在这个声明所属的作用域之外呢？显然，someConstant 的值会随着编译器处理这个声明而变化。所以，常量的“值不变”这一概念在这里并不完全适用。

这里真正关注的不是程序将值与 someConstant 关联的*位置*，而是*时间*。*绑定* 是创建某个对象的属性（如名字、值和作用域）之间关联的技术术语。例如，之前的 Pascal 示例将值 `5` 绑定到名字 someConstant。*绑定时间*——即绑定（关联）发生的时间——可以在多个不同的时刻发生：

+   *在语言定义时。* 这指的是语言设计者定义语言的时间。许多语言中的常量 `true` 和 `false` 就是很好的例子。

+   *在编译期间。* 本节中的 Pascal someConstant 声明就是一个很好的例子。

+   *在链接阶段。* 一个示例可能是指定程序中对象代码（机器指令）大小的常量。程序不能在链接阶段之前计算这个大小，因为链接器会将所有对象代码模块提取并合并在一起。

+   *程序加载（到内存）时。* 一个好的加载时绑定示例是将内存中某个对象的地址（例如变量或机器指令）与某个指针常量关联起来。在许多系统中，操作系统在加载代码到内存时会进行重定位，因此程序只能在加载后确定绝对内存地址。

+   *程序执行期间。* 有些绑定只能在程序运行时发生。例如，当你将某个（计算出的）算术表达式的值赋给一个变量时，值与变量的绑定发生在执行期间。

*动态绑定* 是指在程序执行期间发生的绑定。*静态绑定* 是指在其他任何时间发生的绑定。第七章将再次讨论绑定（参见《什么是变量？》在第 180 页）。

### 6.3 字面常量与显式常量

*显式常量* 是与符号名称相关联（即绑定到符号名称）的常量值。语言翻译器可以在源代码中每次出现该名称时直接替换该值，从而生成易于阅读和维护的程序。正确使用显式常量是专业编写代码的良好标志。

在许多编程语言中，声明显式常量非常简单：

+   Pascal 程序员使用 `const` 区域。

+   HLA 程序员可以使用 `const` 或 `val` 声明区块。

+   C/C++ 程序员可以使用 `#define` 宏功能。

这个 Pascal 代码片段展示了在程序中正确使用显式常量的例子：

```

			const
    maxIndex = 9;

var
    a :array[0..maxIndex] of integer;
        .
        .
        .
    for i := 0 to maxIndex do
        a[i] := 0;
```

这段代码比使用文字常量的代码更易读和维护。通过更改此程序中的单一语句（`maxIndex` 常量声明）并重新编译源文件，你可以轻松设置元素的数量，并且程序将继续正常运行。

由于编译器将文字常量替换为显式常量的符号名称，因此使用显式常量不会带来性能损失。鉴于它们能够在不损失效率的情况下提高程序的可读性，显式常量是优秀代码的重要组成部分。请使用它们。

### 6.4 常量表达式

许多编译器支持使用 *常量表达式*，即那些可以在编译时计算的表达式。常量表达式的组成值在编译时就已知，因此编译器可以在编译时计算表达式并替代其值，而不是在运行时计算它。像显式常量一样，常量表达式使你能够编写更易于阅读和维护的代码，而不会带来任何运行时效率损失。

例如，考虑以下 C 代码：

```

			#define smArraySize 128
#define bigArraySize (smArraySize*8)
      .
      .
      .
char name[ smArraySize ];
int  values[ bigArraySize ];
```

这两个数组声明扩展为以下内容：

```

			char name[ 128 ];
int  values[ (smArraySize * 8) ];
```

C 预处理器进一步将其扩展为：

```

			char name[ 128 ];
int  values[ (128 * 8) ];
```

尽管 C 语言定义支持常量表达式，但并非所有语言都支持此特性，因此你需要查看特定编译器的语言参考手册。例如，Pascal 语言定义并未提及常量表达式。一些 Pascal 实现支持它们，但其他一些则不支持。

现代优化编译器能够在编译时计算算术表达式中的常量子表达式（称为 *常量折叠*；详见 第 63 页的“常见编译器优化”），从而节省了在运行时计算固定值的开销。考虑以下 Pascal 代码：

```

			var
    i   :integer;
            .
            .
            .
    i := j + (5*2-3);
```

任何一个合格的 Pascal 实现都能够识别子表达式 `5*2–3` 是一个常量表达式，在编译期间计算这个表达式的值（`7`），并在编译时用该值替代。换句话说，一个优秀的 Pascal 编译器通常会生成等同于以下语句的机器代码：

```
i := j + 7;
```

如果你的编译器完全支持常量表达式，你可以利用这个特性来编写更好的源代码。可能看起来有些矛盾，但在程序的某些地方写出完整的表达式，有时会使那部分代码更容易阅读和理解；阅读代码的人可以准确看到你如何计算一个值，而不必弄清楚你是如何得出某个“魔法”数字的。例如，在发票或工时单的计算中，表达式`5*2–3`可能比字面常量`7`更能描述“两个工人工作五小时，减去三小时工时”的计算过程。

以下示例 C 代码及 GCC 编译器生成的 PowerPC 输出展示了常量表达式优化的实际情况：

```

			#include <stdio.h>
int main( int argc, char **argv, char **envp )
{
  int j;

  j = argc+2*5+1;
  printf( "%d %d\n", j, argc );
}
```

以下是 GCC 的输出（PowerPC 汇编语言）：

```

			_main:
    mflr r0
    mr r4,r3            // Register r3 holds the ARGC value upon entry
    bcl 20,31,L1$pb
L1$pb:
    mr r5,r4            // R5 now contains the ARGC value.
    mflr r10
    addi r4,r4,11       // R4 contains argc+ 2*5+1
                        // (i.e., argc+11)
    mtlr r0             // Code that calls the printf function.
    addis r3,r10,ha16(LC0-L1$pb)
    la r3,lo16(LC0-L1$pb)(r3)
    b L_printf$stub
```

如你所见，GCC 将常量表达式`2*5+1`替换为了常量`11`。

使你的代码更具可读性无疑是做得很好的事情，也是编写优秀代码的一个重要部分。然而，请记住，一些编译器可能不支持常量表达式的使用，而是会生成代码，在运行时计算常量值。显然，这会影响你程序的大小和执行速度。了解你的编译器能够做什么，将帮助你决定是否使用常量表达式，或者为了提高效率而以牺牲可读性为代价预计算表达式。

### 6.5 显式常量与只读内存对象

C/C++程序员可能注意到，上一节没有讨论 C/C++ `const`声明的使用。这是因为你在 C/C++ `const`语句中声明的符号名（以下简称*符号*）不一定是显式常量。也就是说，C/C++并不总是在源文件中每次出现符号时都替换它的值。相反，C/C++编译器可能将该`const`值存储在内存中，然后像引用静态（只读）变量一样引用该`const`对象。这样，`const`对象和静态变量之间唯一的区别是，C/C++编译器不允许你在运行时给`const`赋值。

C/C++有时会将你在`const`语句中声明的常量当作静态变量处理，这是有充分理由的——它允许你在函数内创建局部常量，且这些常量的值可以在每次函数执行时变化（尽管在函数执行过程中，值保持不变）。这就是为什么你不能总是在 C/C++的`const`中使用这样的“常量”，并期望 C/C++编译器预计算它的值。

大多数 C++编译器会接受这个：

```

			const int arraySize = 128;
      .
      .
      .
int anArray[ arraySize ];
```

然而，他们不会接受这个序列：

```

			const int arraySizes[2] = {128,256}; // This is legal
const int arraySize = arraySizes[0]; // This is also legal

int array[ arraySize ]; // This is not legal
```

`arraySize` 和 `arraySizes` 都是常量。然而，C++ 编译器不允许你使用 `arraySizes` 常量，或者基于它的任何内容作为数组边界。这是因为 `arraySizes[0]` 实际上是一个运行时内存位置，因此 `arraySize` 也必须是一个运行时内存位置。理论上，你可能认为编译器会足够智能，能够推断出 `arraySize` 在编译时是可以计算的，并将其值（`128`）直接替代。然而，C++ 语言并不允许这么做。

### 6.6 Swift let 语句

在 Swift 编程语言中，你可以使用 `let` 语句来创建常量。例如：

```

			let someConstant = 5
```

然而，该值在运行时绑定到常量的名称（也就是说，这是一个动态绑定）。赋值运算符（`=`）右侧的表达式不一定是常量表达式；它可以是包含变量和其他非常量组件的任意表达式。每次程序执行此语句时（例如在循环中），程序可能会为 `someConstant` 绑定一个不同的值。

Swift 的 `let` 语句并不真正定义常量，至少不像传统意义上的常量那样；它允许你创建“只写”变量。换句话说，在你使用 `let` 语句定义的符号的作用域内，你只能初始化该名称一次。请注意，如果你离开并重新进入该名称的作用域，值会被销毁（在退出作用域时），并且你可以在重新进入作用域时为该名称绑定一个新的（可能不同的）值。与 C++ 中的 `const int` 声明不同，`let` 语句不允许你在只读内存中为对象分配存储空间。

### 6.7 枚举类型

编写良好的程序通常使用一组名称来表示没有明确数值表示的现实世界量。例如，这样一组名称可能是各种显示技术，如 `crt`、`lcd`、`led` 和 `plasma`。尽管现实世界并未将数值与这些概念关联，但如果你希望在计算机系统中高效地表示它们，你必须将这些值编码为数字。每个符号的内部表示通常是任意的，只要我们分配的值是唯一的。许多计算机语言提供了 *枚举数据类型*，它自动将唯一的值与列表中的每个名称关联起来。通过在程序中使用枚举数据类型，你可以为数据分配有意义的名称，而不是使用类似 0、1、2 等“魔法数字”。

例如，在 C 语言的早期版本中，你会按照以下方式创建一系列唯一值的标识符：

```

			/*
   Define a set of symbols representing the
   different display technologies
*/

#define crt 0
#define lcd (crt+1)
#define led (lcd+1)
#define plasma (led+1)
```

通过分配连续的值，你可以确保每个值都是唯一的。这个方法的另一个优点是它对值进行了排序。也就是说，`crt` < `lcd` < `led` < `plasma`。不幸的是，这种方式创建显式常量既繁琐又容易出错。

幸运的是，大多数语言的枚举常量可以解决这个问题。 “枚举”意味着编号，这正是编译器所做的——它为每个常量编号，从而处理分配值给枚举常量的记录细节。

大多数现代编程语言都提供了声明枚举类型和常量的支持。以下是来自 C/C++、Pascal、Swift 和 HLA 的一些示例：

```

			enum displays {crt, lcd, led, plasma, oled };       // C++
type displays = (crt, lcd, led, plasma, oled );     // Pascal
type displays :enum{crt, lcd, led, plasma, oled };  // HLA
// Swift example:
enum Displays
{
    case crt
    case lcd
    case led
    case plasma
    case oled
}
```

这四个示例内部将 `0` 与 `crt` 关联，`1` 与 `lcd`，`2` 与 `led`，`3` 与 `plasma`，`4` 与 `oled`。同样，确切的内部表示无关紧要（只要每个值都是唯一的），因为该值的唯一目的是区分枚举对象。

大多数语言会为枚举列表中的符号分配*单调递增*的值（即每个后续值都大于所有前面的值）。因此，这些示例具有以下关系：

```
crt < lcd < led < plasma < oled
```

不要让这个给你留下这样一种印象：单个程序中出现的所有枚举常量都有唯一的内部表示。大多数编译器会将枚举列表中的第一个项分配值 `0`，第二个项分配值 `1`，以此类推。例如，考虑以下 Pascal 类型声明：

```

			type
    colors = (red, green, blue);
    fasteners = (bolt, nut, screw, rivet );
```

大多数 Pascal 编译器会将 `0` 作为 `red` 和 `bolt` 的内部表示；将 `1` 用于 `green` 和 `nut`；以此类推。在一些强制进行类型检查的语言（如 Pascal 和 Swift）中，通常不能在同一表达式中使用类型为 `colors` 和 `fasteners` 的符号。因此，这些符号共享相同的内部表示并不是问题，因为编译器的类型检查机制会防止任何可能的混淆。然而，一些语言（如 C/C++ 和汇编语言）并不提供强类型检查，因此可能会发生这种混淆。在这些语言中，避免混用不同类型的枚举常量是程序员的责任。

大多数编译器会分配 CPU 可以高效访问的最小内存单元来表示枚举类型。由于大多数枚举类型声明定义的符号少于 256 个，因此在能够高效访问字节数据的机器上，编译器通常会为任何具有枚举数据类型的变量分配一个字节。许多 RISC 机器上的编译器可以分配一个 32 位字（或更多），因为访问这些数据块更快。确切的表示方法依赖于语言和编译器/实现，因此请查阅编译器的参考手册以获取详细信息。

### 6.8 布尔常量

许多高级编程语言提供*布尔*或*逻辑*常量来表示`true`和`false`的值。因为布尔值只有两个可能的值，所以它们的表示只需要一个位。然而，由于大多数 CPU 不允许你分配单个位的存储空间，大多数编程语言使用整个字节甚至更大的对象来表示布尔值。那么，布尔对象中剩余的位会怎样呢？不幸的是，答案因语言而异。

许多语言将布尔数据类型视为枚举类型。例如，在 Pascal 中，布尔类型定义如下：

```

			type
    boolean = (false, true);
```

这种声明将内部值`0`与`false`关联，将`1`与`true`关联。这个关联具有一些理想的属性：

+   大多数布尔函数和运算符按预期工作——例如，(`true` 和 `true`) = `true`，(`true` 和 `false`) = `false`，等等。

+   当你比较这两个值时，`false`小于`true`——这是一个直观的结果。

不幸的是，将`0`与`false`，`1`与`true`关联并不总是最佳的解决方案。以下是一些原因：

+   某些布尔运算应用于位串时，不会产生预期的结果。例如，你可能期望（not `false`）等于`true`。然而，如果你将布尔变量存储在 8 位对象中，那么（not `false`）的结果是`$FF`，这不等于`true`（`1`）。

+   许多 CPU 提供指令，可以在操作后轻松检测`0`或非零；很少有 CPU 提供隐式的`1`检测。

许多语言，如 C、C++、C#和 Java，将`0`视为`false`，其他任何值视为`true`。这样做有几个优点：

+   提供简便的`0`和非零检查的 CPU 可以轻松测试布尔结果。

+   `0`/非零表示法无论布尔变量存储对象的大小如何，都有效。

不幸的是，这种方案也有一些缺点：

+   许多按位逻辑运算在应用于`0`和非零布尔值时会产生不正确的结果。例如，`$A5`（`true`/非零）与`$5A`（`true`/非零）进行与运算结果为`0`（`false`）。按逻辑与运算，`true`和`true`不应该产生`false`。类似地，（NOT `$A5`）结果是`$5A`。通常，你会期望（NOT `true`）应该产生`false`而不是`true`（`$5A`）。

+   当位串被当作二进制补码有符号整数值处理时，某些`true`的值可能小于零（例如，8 位值`$FF`等于`-1`）。因此，在某些情况下，`false`小于`true`的直观结果可能不正确。

除非你在汇编语言中工作（在这种情况下，你可以定义`true`和`false`的值），否则你必须接受高级语言（HLL）中表示布尔值的方案，正如它在语言参考手册中所解释的那样。

了解你的编程语言如何表示`true`和`false`，可以帮助你编写出生成更好机器代码的高级源代码。例如，假设你正在编写 C/C++代码。在这些语言中，`false`是`0`，`true`是其他任何值。考虑下面的 C 语言语句：

```

			int i, j, k;
      .
      .
      .
    i = j && k;
```

许多编译器为这个赋值语句生成的机器代码是非常糟糕的。它通常看起来像下面这样（Visual C++输出）：

```

			; Line 8
        cmp     DWORD PTR j$[rsp], 0
        je      SHORT $LN3@main
        cmp     DWORD PTR k$[rsp], 0
        je      SHORT $LN3@main
        mov     DWORD PTR tv74[rsp], 1
        jmp     SHORT $LN4@main
$LN3@main:
        mov     DWORD PTR tv74[rsp], 0
$LN4@main:
        mov     eax, DWORD PTR tv74[rsp]
        mov     DWORD PTR i$[rsp], eax
;
```

现在，假设你始终确保使用`0`表示`false`，使用`1`表示`true`（且不允许使用其他值）。在这种条件下，你可以将之前的语句写成这样：

```
i = j & k;  /* Notice the bitwise AND operator */
```

这是 Visual C++为前述语句生成的代码：

```

			; Line 8
        mov     eax, DWORD PTR k$[rsp]
        mov     ecx, DWORD PTR j$[rsp]
        and     ecx, eax
        mov     DWORD PTR i$[rsp], ecx
```

如你所见，这段代码显著更好。只要你始终使用`1`表示`true`，`0`表示`false`，你就可以使用按位与（`&`）和按位或（`|`）操作符代替逻辑运算符。^(3) 如前所述，使用按位取反操作符无法得到一致的结果；但是，你可以通过以下方式实现正确的逻辑非操作：

```
i = ~j & 1; /* "~" is C's bitwise not operator */
```

这个简短的代码片段会反转`j`中的所有位，然后清除除第 0 位以外的所有位。

关键是，你应该非常清楚你的编译器如何表示布尔常量。如果你有选择权（例如任何非零值），那么你可以为`true`和`false`选择适当的值，以帮助你的编译器生成更好的代码。

### 6.9 浮点常量

浮点常量在大多数计算机架构中是特殊情况。因为浮点表示可能会消耗大量位数，很少有 CPU 提供立即寻址模式来将任意常量加载到浮点寄存器中。即使是小的（32 位）浮点常量也是如此。即使是在许多 CISC 处理器上，如 80x86，也是如此。因此，编译器通常需要将浮点常量放置在内存中，然后让程序从内存中读取它们，就像它们是变量一样。例如，考虑以下 C 程序：

```

			#include <stdlib.h>
#include <stdio.h>
int main( int argc, char **argv, char **envp )
{
  static int j;
  static double i = 1.0;
  static double a[8] = {0,1,2,3,4,5,6,7};
  j = 0;
  a[j] = i+1.0;

}
```

现在考虑 GCC 为这个程序使用`-O2`选项生成的 PowerPC 代码：

```

			.lcomm _j.0,4,2
.data
// This is the variable i.
// As it is a static object, GCC emits the data directly
// for the variable in memory. Note that "1072693248" is
// the HO 32-bits of the double-precision floating-point
// value 1.0, 0 is the LO 32-bits of this value (in integer
// form).

    .align 3
_i.1:
    .long       1072693248
    .long       0

// Here is the "a" array. Each pair of double words below
// holds one element of the array. The funny integer values
// are the integer (bitwise) representation of the values
// 0.0, 1.0, 2.0, 3.0, ..., 7.0.

    .align 3
_a.2:
    .long       0
    .long       0
    .long       1072693248
    .long       0
    .long       1073741824
    .long       0
    .long       1074266112
    .long       0
    .long       1074790400
    .long       0
    .long       1075052544
    .long       0
    .long       1075314688
    .long       0
    .long       1075576832
    .long       0

// The following is a memory location that GCC uses to represent
// the literal constant 1.0\. Note that these 64 bits match the
// same value as a[1] in the _a.2 array. GCC uses this memory
// location whenever it needs the constant 1.0 in the program.

.literal8
    .align 3
LC0:
    .long       1072693248
    .long       0

// Here's the start of the main program:

.text
    .align 2
    .globl _main
_main:

// This code sets up the static pointer register (R10), used to
// access the static variables in this program.

    mflr r0
    bcl 20,31,L1$pb
L1$pb:
    mflr r10
    mtlr r0

    // Load floating-point register F13 with the value
    // in variable "i":

    addis r9,r10,ha16(_i.1-L1$pb)  // Point R9 at i
    li r0,0
    lfd f13,lo16(_i.1-L1$pb)(r9)   // Load F13 with i's value.

    // Load floating-point register F0 with the constant 1.0
    // (which is held in "variable" LC0:

    addis r9,r10,ha16(LC0-L1$pb)   // Load R9 with the
                                   //  address of LC0
    lfd f0,lo16(LC0-L1$pb)(r9)     // Load F0 with the value
                                   //  of LC0 (1.0).

    addis r9,r10,ha16(_j.0-L1$pb)  // Load R9 with j's address
    stw r0,lo16(_j.0-L1$pb)(r9)    // Store a zero into j.

    addis r9,r10,ha16(_a.2-L1$pb)  // Load a[j]'s address into R9

    fadd f13,f13,f0                // Compute i+1.0

    stfd f13,lo16(_a.2-L1$pb)(r9)  // Store sum into a[j]
    blr                            // Return to caller
```

由于 PowerPC 处理器是一个 RISC CPU，GCC 为这个简单的代码序列生成的代码相当复杂。为了与 CISC 等效代码进行对比，请看下面的 80x86 的 HLA 代码；它是 C 代码逐行翻译的结果：

```

			program main;
static
    j:int32;
    i:real64 := 1.0;
    a:real64[8] := [0,1,2,3,4,5,6,7];

readonly
    OnePointZero : real64 := 1.0;

begin main;

    mov( 0, j );  // j=0;

    // push i onto the floating-point stack
 fld( i );

    // push the value 1.0 onto the floating-point stack

    fld( OnePointZero );

    // pop i and 1.0, add them, push sum onto the FP stack

    fadd();

    // use j as an index

    mov( j, ebx );

    // Pop item off FP stack and store into a[j].

    fstp( a[ ebx*8 ] );

end main;
```

这段代码比 PowerPC 代码更容易理解（这是 CISC 代码优于 RISC 代码的一个优势）。注意，和 PowerPC 一样，80x86 不支持大多数浮点操作数的立即寻址模式。因此，和 PowerPC 一样，你必须将常量`1.0`的副本放置在某个内存位置，并在需要使用`1.0`的值时访问该内存位置。^(4)

因为大多数现代 CPU 不支持对所有浮点常量使用立即寻址模式，所以在程序中使用这些常量等同于访问用这些常量初始化的变量。别忘了，如果你访问的内存位置不在数据缓存中，访问内存可能会非常慢。因此，使用浮点常量可能比访问适合寄存器的整数或其他常量值慢得多。

请注意，一些 CPU 确实允许将某些浮点立即常量编码为指令的操作码的一部分。例如，80x86 处理器有一个特殊的“加载零”指令，它将`0.0`加载到浮点栈中。ARM 处理器也提供了一条指令，允许将某些浮点常量加载到 CPU 浮点寄存器中（请参阅附录 C 在线中的“`vmov`指令”）。

在 32 位处理器上，CPU 通常可以使用整数寄存器和立即寻址模式执行简单的 32 位浮点运算。例如，你可以通过加载一个 32 位整数寄存器，将该数值的比特模式加载进去，然后将整数寄存器存储到浮点变量中，从而轻松地将一个 32 位单精度浮点值赋给变量。考虑以下代码：

```

			#include <stdlib.h>
#include <stdio.h>
int main( int argc, char **argv, char **envp )
{

  static float i;

  i = 1.0;

}
```

下面是 GCC 为此序列生成的 PowerPC 代码：

```

			.lcomm _i.0,4,2 // Allocate storage for float variable i

.text
    .align 2
    .globl _main
_main:

    // Set up the static data pointer in R10:

    mflr r0
    bcl 20,31,L1$pb
L1$pb:
    mflr r10
    mtlr r0

    // Load the address of i into R9:

    addis r9,r10,ha16(_i.0-L1$pb)

    // Load R0 with the floating-point representation of 1.0
    // (note that 1.0 is equal to 0x3f800000):

    lis r0,0x3f80 // Puts 0x3f80 in HO 16 bits, 0 in LO bits

    // Store 1.0 into variable i:

    stw r0,lo16(_i.0-L1$pb)(r9)

    // Return to whomever called this code:

    blr
```

作为 CISC 处理器，80x86 使得在汇编语言中执行此任务变得非常简单。下面是实现相同功能的 HLA 代码：

```

			program main;
static
    i:real32;
begin main;

    mov( $3f800_0000, i ); // i = 1.0;

end main;
```

将单精度浮点常量简单地赋值给浮点变量通常可以利用 CPU 的立即寻址模式，从而节省访问内存的开销（因为内存中的数据可能不在缓存中）。不幸的是，编译器并不总是利用这种技巧将浮点常量赋值给双精度变量。例如，PowerPC 或 ARM 上的 GCC 会退回到将常量保存在内存中，并在将常量赋值给浮点变量时复制该内存位置的值。

大多数优化编译器足够智能，能够在内存中维护它们创建的常量表。因此，如果你在源文件中多次引用常量`2.0`（或任何其他浮点常量），编译器只会为该常量分配一个内存对象。然而，请记住，这种优化仅在同一个源文件内有效。如果你在不同的源文件中引用相同的常量值，编译器可能会为该常量创建多个副本。

的确，拥有多个数据副本会浪费存储空间，但考虑到大多数现代系统的内存容量，这只是一个小问题。更大的问题是，程序通常以随机方式访问这些常量，因此它们很少驻留在缓存中，实际上，它们往往会将其他更常用的数据从缓存中逐出。

解决这个问题的一个方法是自己管理浮动点“常量”。因为就程序而言，这些常量实际上是变量，你可以负责这个过程，并将需要的浮动点常量放入已初始化的静态变量中。例如：

```

			#include <stdlib.h>
#include <stdio.h>

static double OnePointZero_c = 1.0;

int main( int argc, char **argv, char **envp )
{
  static double i;

  i = OnePointZero_c;
}
```

当然，在这个例子中，通过将浮动点常量处理为静态变量，你根本不会获得任何好处。然而，在更复杂的情况下，当你有多个浮动点常量时，你可以分析程序，确定哪些常量经常被访问，并将这些常量的变量放置在相邻的内存位置。由于大多数 CPU 处理引用的空间局部性的方式（参见*WGC1*），当你访问其中一个常量对象时，缓存行将被填充相邻对象的值。因此，当你在短时间内访问其他对象时，它们的值很可能已经在缓存中。自己管理这些常量的另一个优点是，你可以创建一个全局常量集合，可以在不同的编译单元（源文件）中引用，这样程序在访问某个常量时只会访问一个内存对象，而不是多个内存对象（每个编译单元一个）。编译器通常没有足够的智能来做出有关数据的这种决策。

### 6.10 字符串常量

像浮动点常量一样，字符串常量也无法被大多数编译器高效处理（即使它们是字面值常量或显式常量）。理解何时应该使用显式常量，何时应将其替换为内存引用，可以帮助你指导编译器生成更好的机器代码。例如，大多数 CPU 无法将字符串常量编码为指令的一部分。使用显式字符串常量实际上可能使程序的效率降低。考虑以下 C 代码：

```

			#define strConst "A string constant"
        .
        .
        .
    printf( "string: %s\n", strConst );
        .
        .
        .
    sptr = strConst;
        .
        .
        .
    result = strcmp( s, strConst );
        .
        .
        .
```

编译器（实际上是 C 预处理器）将宏`strConst`展开为字符串字面值`"A string constant"`，每当标识符`strConst`出现在源文件中时，所以这段代码实际上等价于：

```

			    .
    .
    .
printf( "string: %s\n", "A string constant" );
    .
    .
    .
sptr = "A string constant";
    .
    .
    .
result = strcmp( s, "A string constant" );
```

这段代码的问题在于相同的字符串常量在程序的不同位置出现。在 C/C++中，编译器将字符串常量放入内存并替换为指向该字符串的指针。一个没有优化的编译器可能会在内存中创建三份相同的字符串副本，这会浪费空间，因为这三份数据是完全相同的。（记住，我们这里说的是*常量*字符串。）

编译器开发者几十年前发现了这个问题，并修改了编译器以跟踪给定源文件中的字符串。如果一个程序多次使用相同的字符串字面常量，编译器不会为第二个副本的字符串分配存储空间，而是直接使用第一个字符串的地址。这种优化（常量折叠）可以减少代码的大小，特别是当相同的字符串出现在源文件的多个地方时。

不幸的是，常量折叠并不总是正常工作。一个问题是，许多旧的 C 程序将字符串字面常量分配给字符指针变量，然后继续修改该字面字符串中的字符。例如：

```

			sptr = "A String Constant";
    .
    .
    .
*(sptr+2) = 's';
    .
    .
    .
/* The following displays "string: 'a string Constant'" */

printf( "string: '%s'\n", sptr );
    .
    .
    .
/* This prints "a string Constant"! */

printf( "A String Constant" );
```

重用相同字符串常量的编译器会失败，如果用户将数据存储到字符串对象中，就像这段代码演示的那样。虽然这是不良的编程实践，但在旧的 C 程序中，这种情况足够频繁，以至于编译器供应商无法为多个副本的相同字符串字面量使用相同的存储空间。即使编译器供应商将字符串字面常量放入只读内存以防止这个问题，仍然会出现其他语义问题。这引出了如下的 C/C++ 代码：

```

			sptr1 = "A String Constant";
sptr2 = "A String Constant";
s1EQs2 = sptr1 == sptr2;
```

执行完这段指令序列后，`s1EQs2` 会包含 `true`（`1`）还是 `false`（`0`）？在 C 编译器没有良好优化器的早期程序中，这段语句会让 `s1EQs2` 为 `false`。这是因为编译器创建了两个不同的字符串副本，并将这些字符串放置在内存的不同地址（因此程序分配给 `sptr1` 和 `sptr2` 的地址会不同）。在一个后来的编译器中，如果编译器仅保留字符串数据的单一副本，这段代码序列会使 `s1EQs2` 为 `true`，因为 `sptr1` 和 `sptr2` 会指向相同的内存地址。无论字符串数据是否出现在受保护的内存中，这种差异都存在。

为了解决这个难题，许多编译器供应商提供了一个编译器选项，允许程序员决定编译器是应生成每个字符串的单一副本，还是为每个字符串的出现生成单独的副本。如果你不向字符串字面常量写入数据或比较它们的地址，可以选择这个选项来减少程序的大小。如果你有旧代码需要单独的字符串数据副本（希望你不会再写需要这种方式的新代码），你可以启用此选项。

不幸的是，许多程序员完全没有意识到这个选项，且一些编译器的默认条件通常是创建字符串数据的多个副本。如果你正在使用 C/C++ 或其他通过字符数据指针操作字符串的语言，检查编译器是否提供合并相同字符串的选项，如果有的话，启用该功能。

如果你的 C/C++ 编译器没有提供这个字符串合并优化，你可以手动实现它。为此，只需在程序中创建一个`char`数组变量，并用字符串的地址进行初始化。然后，像使用常量一样在整个程序中使用该数组变量的名称。例如：

```

			char strconst[] = "A String Constant";
        .
        .
        .
    sptr = strconst;
        .
        .
        .
    printf( strconst );
        .
        .
        .
    if( strcmp( string, strconst ) == 0 )
    {
        .
        .
        .
    }
```

这段代码将只在内存中保持一个字符串字面量常量的副本，即使编译器并不直接支持该优化。事实上，即使你的编译器直接支持此优化，仍然有几个很好的理由让你使用这个技巧，而不是依赖编译器为你完成这个工作。

+   将来你可能需要将代码移植到一个不支持此优化的不同编译器。

+   通过手动处理优化，你就不必担心这个问题了。

+   通过使用指针变量而非字符串字面量常量，你可以在程序控制下轻松更改该指针所指向的字符串。

+   将来你可能需要修改程序，以便在程序控制下切换（自然）语言。

+   你可以在多个文件之间轻松共享字符串。

这个字符串优化讨论假设你的编程语言通过引用操作字符串（即，通过使用指向实际字符串数据的指针）。虽然对于 C/C++ 程序来说这确实是事实，但并非所有语言都如此。支持字符串的 Pascal 实现（如 Free Pascal）通常是通过值而非通过引用来操作字符串。每当你将一个字符串值赋给一个字符串变量时，编译器会复制字符串数据，并将该副本放入为字符串变量保留的存储空间中。这个复制过程可能会很耗费资源，如果你的程序从不修改字符串变量中的数据，那么这种复制就是不必要的。更糟糕的是，如果（Pascal）程序将字符串字面量赋给字符串变量，程序将会有两个字符串副本在内存中（一个是字符串字面量常量，另一个是程序为字符串变量所做的副本）。如果程序以后再也不修改这个字符串（这并不罕见），它将浪费内存，通过保留两个字符串副本来维护一个本可以只保留一个副本的字符串。这些原因（空间和速度）可能就是 Borland 在创建 Delphi 4.0 时采用了更复杂的字符串格式，而放弃了早期版本 Delphi 中的字符串格式的原因。^(5)

Swift 也将字符串视为值对象。这意味着，在最坏的情况下，每当你将一个字符串字面量赋值给字符串变量时，Swift 会复制该字符串字面量。然而，Swift 实现了一种名为*按需复制*的优化。每当你将一个字符串对象赋给另一个，Swift 只会复制一个指针。因此，如果多个字符串被赋值相同的值，Swift 会为所有副本在内存中使用相同的字符串数据。当你修改字符串的某个部分时，Swift 会在修改之前先复制字符串（因此称为“按需复制”），以确保引用原始字符串数据的其他字符串对象不会受到该修改的影响。

### 6.11 复合数据类型常量

许多语言除了字符串外，还支持其他复合常量类型（如数组、结构体/记录和集合）。通常，这些语言使用这些常量在程序执行前静态初始化变量。例如，考虑以下 C/C++代码：

```
static int arrayOfInts[8] = {1,2,3,4,5,6,7,8};
```

注意，`arrayOfInts` 不是一个常量。相反，它是构成数组常量的初始化器——即`{1,2,3,4,5,6,7,8}`。在可执行文件中，大多数 C 编译器只是在与`arrayOfInts`关联的地址上叠加这八个整数值。

例如，下面是 GCC 为这个变量输出的内容：

```

			LC0:          // LC0 is the internal label associated
              //  with arrayOfInts
    .long       1
    .long       2
    .long       3
    .long       4
    .long       5
    .long       6
    .long       7
    .long       8
```

假设`arrayOfInts`是 C 中的静态对象，那么存储常量数据不会占用额外的空间。

然而，如果你正在初始化的变量不是静态分配的对象，规则就会发生变化。考虑以下简短的 C 代码序列：

```

			int f()
{
  int arrayOfInts[8] = {1,2,3,4,5,6,7,8};
    .
    .
    .
} // end f
```

在这个例子中，`arrayOfInts`是一个*自动*变量，这意味着每次程序调用函数`f()`时，程序都会在栈上为该变量分配存储空间。因此，编译器不能仅仅在程序加载到内存时使用常量数据来初始化数组。`arrayOfInts`对象实际上可能在每次激活函数时位于不同的地址。为了遵循 C 编程语言的语义，编译器必须复制数组常量，并在程序调用该函数时将该常量数据物理复制到`arrayOfInts`变量中。以这种方式使用数组常量会消耗额外的空间（用于存储数组常量的副本）和额外的时间（用于复制数据）。有时，算法的语义要求每次新激活函数`f()`时都要获取数据的新副本。然而，你需要认识到什么时候这是必要的（以及什么时候额外的空间和时间是值得的），而不是无谓地浪费内存和 CPU 周期。

如果你的程序没有修改数组的数据，你可以使用一个静态对象，编译器可以在加载程序到内存时初始化该对象一次：

```

			int f()
{
  static int arrayOfInts[8] = {1,2,3,4,5,6,7,8};
    .
    .
    .
} // end f
```

C/C++语言也支持结构体常量。当初始化自动变量时，我们看到的数组的空间和速度问题，同样适用于结构体常量。

Embarcadero 的 Delphi 编程语言也支持结构化常量，尽管这里的“常量”一词有些误导。Embarcadero 称它们为 *类型化常量*，你可以在 Delphi 的 `const` 部分这样声明：

```

			const
    ary: array[0..7] of integer = (1,2,3,4,5,6,7,8);
```

尽管声明出现在 Delphi 的 `const` 部分，Delphi 实际上将其视为变量声明。这是一个不太理想的设计选择，但对于想要创建结构化常量的程序员来说，这种机制是可行的。与本节中的 C/C++ 示例一样，重要的是要记住，示例中的常量实际上是 `(1,2,3,4,5,6,7,8)` 对象，而不是 `ary` 变量。

Delphi（以及大多数现代 Pascal，如 Free Pascal）也支持其他几种复合常量类型。例如，集合常量就是一个很好的例子。每当你创建一个对象集合时，Pascal 编译器通常会用集合数据的幂集（位图）表示来初始化某个内存位置。每当你在程序中引用该集合常量时，Pascal 编译器会生成一个指向集合常量数据的内存引用。

Swift 也支持复合数据类型常量，如数组、元组、字典、结构体/类以及其他数据类型。例如，以下 `let` 语句创建了一个包含八个元素的数组常量：

```

			let someArray = [1,2,3,4,11,12,13,14]
```

### 6.12 常量不变

理论上，绑定到常量的值是不会改变的（Swift 中的 `let` 语句是一个明显的例外）。在现代系统中，将常量放入内存的编译器通常会将它们放置在写保护内存区域中，以便在发生意外写入时强制引发异常。当然，很少有程序仅使用只读（或一次写入）对象来编写。大多数程序都需要能够改变它们操作的对象（*变量*）的值。这是下一章的内容。

### 6.13 更多信息

Duntemann, Jeff. *逐步学习汇编语言*. 第 3 版. 印第安纳波利斯：Wiley，2009 年。

Hyde, Randall. *汇编语言的艺术*. 第 2 版. 旧金山：No Starch Press，2010 年。

——. *编写高质量代码，第 1 卷：理解机器*. 第 2 版. 旧金山：No Starch Press，2020 年。
