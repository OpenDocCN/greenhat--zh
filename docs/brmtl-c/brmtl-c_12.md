# 第十一章：链接器

![](img/chapterart.png)

本章极为详细地探讨了链接过程的工作原理。链接器的工作是将构成程序的所有目标文件组合在一起。链接器必须准确知道设备的内存布局，以便能够将程序装入内存。它还负责将一个文件中的外部符号与另一个文件中的实际定义连接起来。这个过程称为*链接符号*。

正是链接器知道事物的具体位置。在拥有数 GB 内存的大型系统中，这没什么大不了的，但在具有 16KB RAM 的微控制器上，了解每个字节的用途非常重要。

让我们来看一个典型问题，看看更好地理解链接器如何帮助解决问题。假设你在现场有一个系统发生崩溃。当它崩溃时，它会打印出一个堆栈跟踪，显示出导致问题的调用堆栈（见示例 11-1）。

```
**#0  0x0000000000001136 in ?? ()**
**#1  0x0000000000001150 in ?? ()**
#2  0x0000000000001165 in ?? ()
#3  0x000000000000117a in ?? ()
#4  0x00007ffff7de50b3 in __libc_main (main=0x555555555168) at ../csu/libc-start.c:308
#5  0x000000000000106e in ?? ()
```

示例 11-1：一个示例堆栈跟踪

这告诉你故障发生在地址为`0x0000000000001136`的函数中。

由于你没有使用绝对地址编写程序，因此函数的名称对你来说更有用。链接器映射就是在这种情况下发挥作用的。

示例 11-2 显示了该程序映射的一个片段。

```
.text          0x0000000000001129       0x58 /tmp/cctwz0VM.o
               **0x0000000000001129                three**
 **0x000000000000113e                two**
               0x0000000000001153                one
               0x0000000000001168                main
```

示例 11-2：来自示例 11-1 程序映射的片段

我们在示例 11-1 中的`0x1136`处中止了。在示例 11-2 中，函数`three`从`0x1129`开始，一直到下一个函数`0x113e`。实际上，我们已经进入了函数`three`的 13 个字节，所以我们离函数的开始位置很近。

示例 11-1 显示了函数`three`是由位于地址`0x1150`的某个地方调用的。示例 11-2 显示函数`two`从`0x113e`到`0x1153`，因此它调用了`three`。通过类似的分析，我们可以看出`two`是由`one`调用的，而`one`又是由`main`调用的。

## 链接器的工作

链接器的工作是将组成程序的目标文件组合在一起，形成一个单一的程序文件。目标文件包含代码和数据，这些代码和数据按名称组织成不同的部分。（部分的实际名称依赖于编译器。高级程序员甚至可以自定义部分名称。）

目标文件中的部分没有固定地址。它们被称为*可重定位*的，这意味着它们几乎可以放置在任何地方，但链接器会将它们放置在内存中的特定位置。

ARM 芯片包含两种类型的内存：随机存取内存（RAM）和闪存。RAM 用于存储变量。这种类型内存的问题之一是，当电源关闭时，所有数据都会丢失。闪存，在实际应用中，类似于只读存储器。（如果你在 I/O 系统上非常聪明，当然也可以写入它。）闪存中的数据不会在系统断电时丢失。

链接器从所有对象文件中提取数据，并将其打包进 RAM 中。然后，它将剩余的 RAM 分配给堆栈和堆。代码和只读数据存放在闪存中。这个描述有些过于简化，但我们会在本章后面详细讨论这些细节。

链接器的最终任务是写出一个映射文件，告诉你它将每个部分放在哪里。为什么我们关心链接器把东西放在哪里？毕竟，最重要的事情是程序被加载到内存中。然而，在现场调试时，我们需要知道各个部分的位置。此外，有时我们可能需要定义特定的内存区域或为系统附加额外的内存芯片。

然后是一个重要原因：固件升级。据说硬件人员必须第一次就把硬件做对。软件人员唯一需要做对的事情就是固件升级。但如何使用正在运行的软件来替换正在运行的软件呢？更重要的是，如何做到这一点而不会把系统弄成“砖头”？（*砖头*是指固件升级失败后，系统变得和砖头一样毫无用处。）这涉及一些复杂的编程，我将在本章末尾解释。

## 编译和链接的内存模型

*内存模型*描述了系统中内存的配置方式。基本上，内存被划分为命名的区域。C 标准、对象文件和 ARM 芯片使用不同的名称来描述它们的内存。更糟糕的是，还可以通过 C 语言扩展来定义自定义名称。链接器必须知道如何处理这些自定义区域。

### 理想的 C 模型

理想情况下，C 程序中的所有内容都会放入标准部分之一：`text`、`data`或`bss`。

只读指令和只读数据存放在`text`部分。这里，`main`的代码和文本字符串（只读）都放在`text`部分：

```
int main() {
   doIt("this goes in text too");
   return();
}
```

已初始化的数据（已初始化的全局变量）存放在`data`部分：

```
int anExample = 5;
```

未初始化的数据（未初始化的全局变量）存放在`bss`部分：

```
int uninitialized;
```

从技术上讲，`bss`根据标准是未初始化的。然而，在我见过的每一个 C 编程系统的实现中，`bss`部分都会被初始化为零。

这些部分的数据在编译时分配。C 编译器会输出一个对象文件，表示：“我需要这么多`text`，这里是内容。我需要这么多`data`，这里是内容。我需要这么多`bss`，但是没有指定内容。”

`size`命令显示程序在每个部分使用了多少空间。

```
$ **size example.o**
   text   data    bss    dec    hex    filename
    481      4      4    489    1e9    example.o
```

对象文件使用了 481 字节的`text`，4 字节的`data`，以及另外 4 字节的`bss`。这三者总共占用 489 字节，或在十六进制下为 1e9。

理想的 C 模型还有两个其他内存区块。然而，这些区块并不是由编译器分配的，而是由链接器分配的。它们分别是 *栈* 和 *堆*。栈用于局部变量，并在调用过程时动态分配。堆是一个可以动态分配和释放的内存池（有关堆的更多内容，请参见第十三章）。

编译器会将我们的变量定义分配到内存区块中。这些区块使用的命名空间与理想的 C 内存区块名称不同。在某些情况下，名称相似，而在其他情况下，名称完全不同。不同的编译器，甚至同一编译器的不同版本，可能会为这些区块使用不同的名称。

清单 11-3 显示了一个包含我们讨论的所有类型数据的程序。

```
/**
 * A program to demonstrate various types of variable
 * storage, so we can see what the linker does with them
 */
int uninitializedGlobal;   // An uninitialized global (section bss)
int initializedGlobal = 1; // An initialized global (section data)
int initializedToZero = 0; // An initialized global (section bss)

// aString -- initialized variable (section bss)
// "A string." -- constant (section text)
const char* aString = "A string."; // String (pointing to ready-only data)
static int uninitializedModule;    // An uninitialized module-only symbol
                                   // (section bss)
static int initializedModule = 2;  // An initialized module-only symbol
                                   // (section data)

int main()
{
    int uninitializedLocal;      // A local variable (section stack)
    int initializedLocal = 1234; // An initialized local (section stack)

 static int uninitializedStatic;      // "Uninitialized" static (section bss)
    static int initializedStatic = 5678; // Initialized static (section data)

    while (1)
        continue; // Not much logic here
}
```

清单 11-3：数据类型示例

让我们看看我们的 GNU GCC 编译器如何处理 清单 11-3 中的示例程序——具体来说，它是如何为不同类型的变量和数据分配内存的。

首先，这是来自 清单 11-3 的 `initializedGlobal`：

```
int initializedGlobal = 1; // An initialized global (section data)

  16                            .global initializedGlobal
  17                            .data
  18                            .align  2
  21                    initializedGlobal:
  22 0000 01000000              .word   1
```

`.global` 指令告诉汇编器这是一个全局符号，其他目标文件可以引用它。`.data` 指令告诉汇编器接下来的内容应该放入 `.data` 区块。到目前为止，我们遵循了理想的 C 内存模型命名规范。

`.align` 指令告诉汇编器接下来的数据应当按 4 字节对齐。（地址的最后两位必须为零，因此使用 `.align 2`。）最后，有 `initializedGlobal` 标签和 `.word 1` 数据。

当一个变量被初始化为零（清单 11-3 中的 `initializedToZero`）时，我们会看到略有不同的代码：

```
int initializedToZero = 0; // An initialized global (section bss)

  23                            .global initializedToZero
  24                            .bss
  25                            .align  2
  28                    initializedToZero:
  29 0000 00000000              .space  4
```

在这里，编译器使用 `.bss` 指令将变量放入 `bss` 区块。它还使用 `.space` 指令而不是 `.word`，告诉汇编器该变量占用 4 字节空间，并将这些字节初始化为零。

现在让我们处理一个未初始化的全局变量（清单 11-3 中的 `uninitializedGlobal`）：

```
int uninitializedGlobal; // An uninitialized global (section bss)

  15                            .comm   uninitializedGlobal,4,4
```

`.comm` 区块告诉汇编器定义一个 4 字节长且按 4 字节对齐的符号。该符号会被放入名为 `COMMON` 的内存区块中。在这种情况下，区块名称并不遵循理想的 C 内存模型命名规范。

在 清单 11-3 中定义 `aString` 的语句同时定义了一个字符串常量（`"A string."`）。字符串常量是只读的，而指针（`aString`）是读写的。以下是生成的代码：

```
const char* aString = "A string."; // String (pointing to read-only data)

  30                            .global aString
  31                            .section        .rodata
  32                            .align  2
  33                    .LC0:
  34 0000 41207374              .ascii  "A string.\000"
  34      72696E67
  34      2E00
  35                            .data
  36                            .align  2
  39                    aString:
  40 0004 00000000              .word   .LC0
```

首先，编译器必须为 `"A string."` 生成常量。它为该常量生成一个内部名称（`.LC0`），并通过 `.ascii` 汇编指令生成该常量的内容。`.section .rodata` 指令将常量放入名为 `.rodata` 的链接器区块中。（理想的 C 内存模型将其称为 `text`。）

现在我们来看看变量本身的定义，`aString`。`.data`指令将其放入`data`区域。由于它是一个指针，因此它被初始化为字符串的地址（即`.LC0`）。

最后的主要区域是包含代码的区域。理想的 C 语言内存模型将其称为`text`。以下是`main`函数开始的汇编代码：

```
int main()

  52                            .section        .text.main,"ax",%progbits
  53                            .align  1
  54                            .global main
  60                    main:
  67 0000 80B5                  push    {r7, lr}
```

这个区域的名称是`text.main`。在这种情况下，编译器决定将`text`前缀和模块名（`main`）组合成区域名。

我们已经覆盖了编译器知道的主要内存区域，接下来让我们看看由其他类型声明生成的代码。`static`关键字用于任何过程之外时，表示该变量只能在当前模块内使用。

下面是从清单 11-3 中创建`initializedModule`变量的代码：

```
static int initializedModule = 2; // An initialized module-only symbol
                                  // (section data)

  46                            .data
  47                            .align  2
  50                    initializedModule:
  51 0008 02000000              .word  2
```

它看起来与`initializedGlobal`非常相似，唯一的区别是缺少`.global`指令。

同样，来自清单 11-3 的`uninitializedModule`变量看起来与`uninitializedGlobal`非常相似，只不过我们再次缺少`.global`指令：

```
static int uninitializedModule; // An uninitialized module-only symbol
                                // (section bss)

  41                            .bss
  42                            .align  2
  43                    uninitializedModule:
  44 0004 00000000              .space  4
```

现在我们来讲解在过程内声明为`static`的变量。这些变量在编译时分配到主内存中，但它们的作用域仅限于它们定义的过程内。

让我们从清单 11-3 中的`uninitializedStatic`变量开始：

```
static int uninitializedStatic; // "Uninitialized" static (section bss)

  94                            .bss
  95                            .align  2
  96                    uninitializedStatic.4108:
  97 0008 00000000              .space  4
```

它看起来像任何未初始化的局部变量，只不过编译器将变量名从`uninitializedStatic`改为`uninitializedStatic.4108`。为什么？每个用大括号（`{}`）括起来的代码块可以有自己的`uninitializedStatic`变量。C 语言变量名的作用域局限于定义它的代码块。而汇编语言的作用域是整个文件，因此编译器通过在变量声明的末尾附加一个唯一的随机数来使得变量名唯一。

同样，`initializedStatic`变量看起来也与它的全局变量版本非常相似：

```
static int initializedStatic = 5678; // Initialized static (section data)

  88                            .data
  89                            .align  2
  92                    initializedStatic.4109:
  93 000c 2E160000              .word  5678
```

在这种情况下，`.global`缺失，并且通过添加后缀，变量名发生了变化。

### 非标准区域

我们已经讨论了 GNU 工具链生成的标准内存区域。STM32 芯片使用一个名为`.isr_vector`的自定义区域，它必须是写入闪存的第一个数据，因为 ARM 硬件使用这部分内存来处理中断和其他硬件相关功能。表 11-1，来自 STM32F030x4 手册，描述了中断向量。

表 11-1：中断向量文档（截断）

| **位置** | **优先级** | **优先级类型** | **缩写** | **描述** | **地址** |
| --- | --- | --- | --- | --- | --- |
| — | — | — | — | 保留 | `0x0000 0000` |
| — | –3 | 固定 | 重置 | 重置 | `0x0000 0004` |
| — | –2 | 固定 | NMI | 不可屏蔽中断。RCC 时钟安全系统（CSS）链接到 NMI 向量。 | `0x0000 0008` |
| — | –1 | 固定 | HardFault | 所有类型的故障 | `0x0000 000C` |
| — | 3 | 可设置 | SVCall | 通过 SWI 指令调用系统服务 | `0x0000 002C` |
| — | 5 | 可设置 | PendSV | 可挂起的系统服务请求 | `0x0000 0038` |
| — | 6 | 可设置 | SysTick | 系统滴答定时器 | `0x0000 003C` |
| 0 | 7 | 可设置 | WWDG | 窗口看门狗中断 | `0x0000 0040` |
| 1 |  |  | 保留 |  | `0x0000 0044` |
| 2 | 9 | 可设置 | RTC | RTC 中断（组合的 EXTI 线路 17、19 和 20） | `0x0000 0048` |

STM 固件文件 *startup_stm32f030x8.s*（汇编语言文件）包含定义此表的代码。以下是一个摘录：

```
131                       .section .isr_vector,"a",%progbits
134                    
135                    
136                    g_pfnVectors:
137 0000 00000000        .word  _estack
138 0004 00000000        .word  Reset_Handler
139 0008 00000000        .word  NMI_Handler
140 000c 00000000        .word  HardFault_Handler
```

第一行告诉链接器，该表将放在一个名为 `.isr_vector` 的段中。这个段是高度硬件特定的，定义非常精确，必须放在正确的位置，否则系统将无法正常工作。

该代码定义了一个名为 `g_pfnVectors` 的数组，其中包含以下内容：

+   初始栈的地址

+   复位处理程序的地址

+   不可屏蔽中断（NMI）处理程序的地址

+   其他中断向量，详见 表 11-1

我们将在下一节看到链接器如何处理这段代码。

## 链接过程

编译器和汇编器生成了一组目标文件，将代码和数据划分为以下几个部分：

1.  `text.` `<name>` 只读数据和代码

1.  `rodata` 只读数据

1.  `data` 已初始化数据

1.  `bss` 初始化为零的数据（与理想的 C 内存模型略有不同的定义）

1.  `COMMON` 未初始化数据

1.  `.isr_vector` 中断和复位处理程序，必须放在特定位置

链接器由名为 *LinkerScript.ld* 的脚本控制，该脚本是每个 STM32 工作台项目的一部分。脚本告诉链接器，系统的内存由两个部分组成：

1.  闪存，从 `0x8000000` 开始，长度 64KB

1.  RAM，从 `0x20000000` 开始，长度 8KB

链接器的工作是将目标文件中的数据通过以下步骤打包到内存中：

1.  将 `.isr_vector` 段放置在闪存的开头。

1.  将 `.text.*` 段中的所有数据放入闪存中。

1.  将 `.rodata` 段放入闪存。

1.  将 `.data` 段放入 RAM 中，但 `.data` 段的初始化器需要放入闪存中（我们稍后会详细讨论）。

1.  将 `.bss` 段放入 RAM。

1.  最后，将 `COMMON` 段加载到 RAM 中。

`.data` 段是比较复杂的部分。考虑以下声明：

```
int initializedGlobal = 1234;
```

链接器为 `initializedGlobal` 在 RAM 中分配空间。初始化器（`1234`）放入闪存中。在启动时，初始化器会作为一个块复制到 RAM 中，以初始化 `.data` 段。

## 链接器定义的符号

在链接过程中，链接器会定义一些重要的符号，包括以下内容：

1.  `_sidata` 闪存中 `.data` 段初始化器的起始位置

1.  `_sdata` `.data` 段在 RAM 中的起始位置

1.  `_edata` `.data` 段在 RAM 中的结束位置

1.  `_sbss` `.bss` 和 `COMMON` 段在 RAM 中的起始位置

1.  `_ebss` `.bss` 和 `COMMON` 段在 RAM 中的结束位置

1.  `_estack` RAM 的最后地址

在复位时，*startup_stm32f030x8.S* 中的代码会执行，并完成以下步骤：

1.  使用 `_estack` 加载堆栈寄存器，堆栈将向下增长。

1.  将从 `_sdata` 到 `_edata` 之间的内存区域填充为从 `_sidata` 开始存储的初始化值。

1.  将 `_sbss` 和 `_ebss` 之间的内存清零。

1.  调用 `SystemInit` 函数来初始化 STM32 芯片。

1.  调用 `__libc_init_array` 函数来初始化 C 库。

1.  调用 `main`。

1.  永久循环。

## 重定位和链接目标文件

目标文件有两种类型：*绝对* 和 *可重定位*。绝对文件将所有内容定义为固定（绝对）地址。换句话说，符号 `main` 位于 `0x7B0`，且不能由链接器或其他任何工具设置为其他地址。

可重定位目标文件设计为其数据的位置可以变动（重定位）。例如，*main.c* 源文件会生成 *main.o* 目标文件。如果查看汇编列表，我们会看到符号 `main` 被定义在 `0000`：

```
52                  .section    .text.main,"ax",%progbits
`--snip--`
60                  main:
61                  .LFB0:
`--snip--`
67 0000 80B5            push    {r7, lr}
```

这个符号是相对于它所在的段（即 `text.main`）而言的。由于目标文件是可重定位的，`text.main` 可以位于内存中的任何地方。在这种情况下，链接器决定将其放置在闪存中的 `0x00000000080007b0` 位置。（我们通过链接器映射找到了这个值，接下来的章节会详细讨论。）由于 `main` 位于该段的开始位置，因此它被赋值为 `0x00000000080007b0`。

作为链接器过程的一部分，链接器会将可重定位目标文件分配到内存中的位置。最终结果是一个程序文件，每个目标文件都有绝对地址。

链接器还会将目标文件链接在一起。例如，*startup_stm32f030x8.S* 文件会调用 `main`。问题在于，这段代码并不知道 `main` 位于哪里。它在另一个模块（*main.o*）中定义，因此在链接时，链接器会看到 *startup_stm32f030x8.S* 需要知道 `main` 符号的定义位置，并会执行从 *startup_stm32f030x8.S* 中调用 `main` 到 `main` 的绝对地址（`0x7B0`）的链接操作。

库是以归档格式（类似 *.zip*，但不如其复杂）收集的目标文件（*.o*）。链接器脚本会告诉链接器包括 *libc.a*、*libm.a* 和 *libgcc.a* 库。例如，*libm.a* 库包含以下内容：

```
s_sin.o
s_tan.o
s_tanh.o
s_fpclassify.o
s_trunc.o
s_remquo.o
`--snip--`
```

在处理库时，链接器只会加载定义了你的程序所需符号的目标文件。例如，如果你的程序使用了 `sin` 函数，它将链接包含该函数定义的目标文件 *s_sin.o*。如果你没有使用 `sin` 函数，则链接器知道你不需要 *s_sin.o* 中的代码，因此不会将该文件链接进来。

## 链接器映射

当链接器将数据加载到程序中时，它会生成一个映射文件（*Debug/output.map*），其中包含有关我们代码和数据位置的信息。此映射文件非常完整，包含许多有用信息以及我们不关心的许多内容。例如，它告诉我们我们的内存配置是什么样的，显示处理器的各种类型和位置：

```
Memory Configuration

Name             Origin             Length             Attributes
FLASH            0x0000000008000000 0x0000000000010000 xr
RAM              0x0000000020000000 0x0000000000002000 xrw
*default*        0x0000000000000000 0xffffffffffffffff
```

在这种情况下，我们的芯片具有`FLASH`存储器，其具有设置了读取（`r`）和执行（`x`）属性。它从`0x8000000`开始，延伸至`0x10000`字节。`RAM`部分从`0x20000000`开始，仅延伸至`0x2000`字节。它可读（`r`）、可写（`w`）和可执行（`x`）。

如前所述，`.isr_vector`部分首先加载。链接器映射告诉我们它的位置：

```
.isr_vector     0x0000000008000000       0xc0
```

地址`0x8000000`是 Flash 的起始地址。硬件期望中断向量位于该地址，这是一个好消息。另一点信息是，此部分长度为`0xc0`字节。

`main`符号定义在*src/main.o*中。它是`.text.main`段的一部分，并位于`0x0000000008000138`处：

```
.text.main     0x0000000008000138       0x60 src/main.o
                0x0000000008000138                main
```

它还包含一些代码（`0x60`字节，考虑到清单 11-3 只是一个无用的程序）。

我们还可以看到全局变量的位置。例如，这是`uninitializedGlobal`的位置：

```
 COMMON         0x0000000020000464        0x4 src/main.o
                0x0000000020000464        uninitializedGlobal
```

链接器映射提供了此程序中每个变量和函数的绝对地址。这有什么用呢？当我们在现场调试时（没有 JTAG 调试器），我们经常只有绝对地址，因此如果您的程序遇到致命错误并看到

```
FATAL ERROR: Address  0x0000000008000158
```

在调试控制台上，您会知道错误发生在`main`的`0x20`字节处。

我们一直在使用一个*外部*调试器与我们的 STM 板。该系统由运行调试器的主机计算机、一个 JTAG 调试探针和一个目标机组成。主机计算机上的调试器可以访问源代码和符号表（来自链接器）。当它检测到`0x8000158`处的错误时，它可以查看符号表，查看错误发生在程序的`0x20`字节处，找出错误发生的行，并在源文件中显示一个大红箭头指向错误发生的位置。

有些系统有*内部*调试器，在其中调试器和所有需要的文件都在目标系统上。一些内部调试器提供基于绝对地址转储内存的能力。这些调试器虽然小而愚笨，但在现场调试时非常有用。

假设您有这样一个调试器，并且需要知道`uninitializedGlobal`的值。愚笨的调试器不知道符号名称。它只基于地址转储内存，就这样。

另一方面，您确实知道符号名称。您拥有链接器映射，因此可以告诉调试器在`0x20000464`位置显示 4 字节值：

```
D> x/4 20000464
0x20000464:   1234    0x4D2
```

这种调试方式原始且困难，但在嵌入式系统中，有时它是唯一可以进行调试的方法。

也许你会想知道为什么我们不直接告诉调试器 `uninitializedGlobal` 的位置，这样会更简单。问题在于符号表占用了大量空间，而我们空间有限。而且，符号表存放在系统本身上是一个安全隐患。（黑客会很喜欢知道 `passwordCheckingFunction` 的地址！）

## 高级链接器使用

到目前为止，我们仅使用了默认设置的链接器。然而，在某些时候，你可能会想执行一些比默认设置更高级的功能。

### 闪存用于“永久”存储

默认的 C 内存模型存在一个问题，就是程序启动时所有数据都会被重置。在 STM32 中，这意味着重置设备会导致其丢失所有数据。假设你希望在重启之间保留一些配置信息。默认的设置无法实现这一点。我们该怎么做呢？

让我们从第九章的串口 “Hello World” 程序开始。我们将添加一个计数器，记录系统已经启动了多少次，然后将重置计数信息通过串口设备发送出去。

我们的设计很简单。我们将使用闪存的顶部 4KB 来存储配置信息。我们给它取了个富有创意的名字 `CONFIG`，并定义了一个新的内存区域 `.config`，在其中存放我们的重置变量。

这是完成这一操作的 C 代码：

```
static uint32_t resetCount __attribute__((section(.config)) = 0;
```

现在我们需要修改链接脚本，以处理我们的新区域。我们首先将闪存内存分为两个区域。第一个是我们之前讨论过的传统闪存内存。第二个，`CONFIG`，将存储我们的配置信息，这意味着我们需要编辑 *LinkerScript.ld* 并替换以下内容：

```
MEMORY
{
    FLASH (rx)     : ORIGIN = 0x8000000, LENGTH = 64K
    RAM (xrw)      : ORIGIN = 0x20000000, LENGTH = 8K
}
```

用这个：

```
MEMORY
{
    FLASH (rx)     : ORIGIN = 0x8000000,       LENGTH = 60K
    CONFIG (rw)    : ORIGIN = 0x8000000 + 60K, LENGTH = 4K
    RAM (xrw)      : ORIGIN = 0x20000000,      LENGTH = 8K
}
```

这将 `FLASH` 的大小减少 4KB，然后将这 4KB 用作名为 `CONFIG` 的内存区域。

闪存与普通内存不同，它只能在写入一次之后才能擦除。擦除必须一次进行一页。在 STM32 的情况下，这意味着我们的 `CONFIG` 区域必须至少有 1KB 长，并且必须是 1KB 的倍数。我们选择了 4KB，因为我们可能希望以后存储更多的配置信息。

现在我们需要告诉链接器将 `.config` 区域放入名为 `CONFIG` 的内存块中。这可以通过在 *LinkerScript.ld* 文件的 `SECTIONS` 部分添加以下内容来完成：

```
{
   . = ALIGN(4);
   *(.config*)
} >CONFIG
```

更改这个变量并不像简单地写下以下代码那样容易：

```
++resetCount;
```

编程芯片需要一系列的步骤。我们把所有步骤都放在了一个名为 `updateCounter` 的函数中，见 列表 11-4。

```
/**
 * Update the resetCounter.
 *
 * In C this would be ++resetCount. Because we are dealing
 * with flash, this is a much more difficult operation.
 */
static HAL_StatusTypeDef updateCounter(void) {
  1 HAL_FLASH_Unlock(); // Allow flash to be modified.
 2 uint32_t newResetCount = resetCount + 1;  // Next value for reset count

    uint32_t pageError = 0;     // Error indication from the erase operation

    // Tell the flash system to erase resetCounter (and the rest of the page).
  3 FLASH_EraseInitTypeDef eraseInfo = {  
        .TypeErase = FLASH_TYPEERASE_PAGES,     // Going to erase one page
        .PageAddress = (uint32_t)&resetCount,   // The start of the page
        .NbPages = 1                            // One page to erase
    };

    // Erase the page and get the result.
  4 HAL_StatusTypeDef result = HAL_FLASHEx_Erase(&eraseInfo, &pageError);
    if (result != HAL_OK) {
        HAL_FLASH_Lock();
        return (result);
    }

    // Program the new reset counter into flash.
    result = 5 HAL_FLASH_Program(FLASH_TYPEPROGRAM_WORD,
            (uint32_t)&resetCount, newResetCount);

    HAL_FLASH_Lock();
    return (result);
}
```

列表 11-4：`updateCounter` 程序

STM32 芯片上的闪存是受保护的，因此我们需要通过调用 `HAL_FLASH_Unlock` 1 来解锁它。此函数将两个密码值写入闪存保护系统，以启用闪存的写入操作。然而，我们仍然不能直接将 `resetCount` 写入闪存，因此我们将 `resetCount`（一个闪存值）赋值给 `newResetCount`（一个常规变量）2，这样我们就可以对其进行递增。

在写入闪存之前，我们必须先擦除闪存，而擦除的最小单位是一个页面。我们首先需要初始化一个结构 3，指定要擦除的页面数量及地址，然后将其作为参数传递给 `HAL_FLASHEx_Erase` 来擦除内存 4。

现在，存储 `resetCount` 的内存已被清除，我们可以进行写入。不幸的是，我们有一个 32 位的值，而闪存一次只能写入 16 位，因此我们使用另一个 HAL 函数 `HAL_FLASH_Program` 5 来完成这项任务。

列表 11-5 显示了完整的程序。

```
/**
 * @brief Write the number of times the system reset to the serial device.
 */
#include <stdbool.h>
#include "stm32f0xx_nucleo.h"
#include "stm32f0xx.h"

const char message1[] = "This system has been reset ";   // Part 1 of message
const char message2[] = " times\r\n";                    // Part 2 of message
const char many[] = "many";         // The word many
// Number of times reset has been performed
uint32_t resetCount __attribute__((section(".config.keep"))) = 0;
int current; // The character in the message we are sending

UART_HandleTypeDef uartHandle;      // UART initialization

/**
  * @brief This function is executed in case of error occurrence.
  *
  * All it does is blink the LED.
  */
void Error_Handler(void)
{
    /* Turn ED3 on. */
    HAL_GPIO_WritePin(LED2_GPIO_PORT, LED2_PIN, GPIO_PIN_SET);

    while (true)
    {
    // Toggle the state of LED2.
        HAL_GPIO_TogglePin(LED2_GPIO_PORT, LED2_PIN);
        HAL_Delay(1000);        // Wait one second.
    }
}
/**
 * Send character to the UART.
 *
 * @param ch The character to send
 */
void myPutchar(const char ch)
{
    // This line gets and saves the value of UART_FLAG_TXE at call
    // time. This value changes, so if you stop the program on the "if"
    // line below, the value will be set to zero because it goes away
    // faster than you can look at it.
    int result __attribute__((unused)) =
        (uartHandle.Instance->ISR & UART_FLAG_TXE);

    // Block until the transmit empty (TXE) flag is set.
    while ((uartHandle.Instance->ISR & UART_FLAG_TXE) == 0)
        continue;

    uartHandle.Instance->TDR = ch;     // Send character to the UART.
}

/**
 * Send string to the UART.
 *
 * @param msg Message to send
 */
static void myPuts(const char* const msg)
{
    for (unsigned int i = 0; msg[i] != '\0'; ++i) {
        myPutchar(msg[i]);
    }
}

/**
 * Initialize LED2 (so we can blink red for error).
 */
void led2_Init(void)
{
    // LED clock initialization
    LED2_GPIO_CLK_ENABLE();

    GPIO_InitTypeDef GPIO_LedInit;      // Initialization for the LED
    // Initialize LED.
    GPIO_LedInit.Pin = LED2_PIN;
    GPIO_LedInit.Mode = GPIO_MODE_OUTPUT_PP;
    GPIO_LedInit.Pull = GPIO_PULLUP;
    GPIO_LedInit.Speed = GPIO_SPEED_FREQ_HIGH;
    HAL_GPIO_Init(LED2_GPIO_PORT, &GPIO_LedInit);
}

/**
 * Initialize UART2 for output.
 */
void uart2_Init(void)
{
    // UART initialization
    // UART2 -- one connected to ST-LINK USB
    uartHandle.Instance = USART2;
    uartHandle.Init.BaudRate = 9600;                    // Speed 9600
    uartHandle.Init.WordLength = UART_WORDLENGTH_8B;    // 8 bits/character
    uartHandle.Init.StopBits = UART_STOPBITS_1;         // One stop bit
    uartHandle.Init.Parity = UART_PARITY_NONE;          // No parity
    uartHandle.Init.Mode = UART_MODE_TX_RX;             // Transmit & receive
    uartHandle.Init.HwFlowCtl = UART_HWCONTROL_NONE;    // No hw control

    // Oversample the incoming stream.
    uartHandle.Init.OverSampling = UART_OVERSAMPLING_16;

    // Do not use one-bit sampling.
    uartHandle.Init.OneBitSampling = UART_ONE_BIT_SAMPLE_DISABLE;

    // Nothing advanced
    uartHandle.AdvancedInit.AdvFeatureInit = UART_ADVFEATURE_NO_INIT;
    /*
     * For those of you connecting a terminal emulator, the above parameters
     * translate to 9600,8,N,1.
     */

    if (HAL_UART_Init(&uartHandle) != HAL_OK)
    {
        Error_Handler();
    }
}
/**
 * Update the resetCounter.
 *
 * In C, this would be ++resetCounter. Because we are dealing
 * with flash, this is a much more difficult operation.
 */
static HAL_StatusTypeDef updateCounter(void) {
    HAL_FLASH_Unlock(); // Allow flash to be modified.
    uint32_t newResetCount = resetCount + 1;    // Next value for reset count

    uint32_t pageError = 0;     // Error indication from the erase operation
    // Tell the flash system to erase resetCounter (and the rest of the page).
    FLASH_EraseInitTypeDef eraseInfo = {
        .TypeErase = FLASH_TYPEERASE_PAGES,     // Going to erase 1 page
        .PageAddress = (uint32_t)&resetCount,   // The start of the page
        .NbPages = 1                            // One page to erase
    };

    // Erase the page and get the result.
    HAL_StatusTypeDef result = HAL_FLASHEx_Erase(&eraseInfo, &pageError);
    if (result != HAL_OK) {
        HAL_FLASH_Lock();
        return (result);
    }

    // Program the new reset counter into flash.
    result = HAL_FLASH_Program(FLASH_TYPEPROGRAM_WORD,
            (uint32_t)&resetCount, newResetCount);

    HAL_FLASH_Lock();
    return (result);
}

int main(void)
{
    HAL_Init(); // Initialize hardware.
    led2_Init();
    uart2_Init();

    myPuts(message1);

    HAL_StatusTypeDef status = updateCounter();

    switch (status) {
        case HAL_FLASH_ERROR_NONE:
            // Nothing, this is correct.
            break;
        case HAL_FLASH_ERROR_PROG:
            myPuts("HAL_FLASH_ERROR_PROG");
            break;
        case HAL_FLASH_ERROR_WRP:
            myPuts("HAL_FLASH_ERROR_WRP");
            break;
        default:
            myPuts("**unknown error code**");
            break;
    }
    // A copout to avoid writing an integer to an ASCII function
 if (resetCount < 10)
        myPutchar('0'+ resetCount);
    else
        myPuts("many");

    myPuts(message2);

    for (;;) {
        continue;       // Do nothing.
    }
}

/**
 * Magic function that's called by the HAL layer to actually
 * initialize the UART. In this case, we need to put the UART pins in
 * alternate mode so they act as UART pins and not like GPIO pins.
 *
 * @note: Only works for UART2, the one connected to the USB serial
 * converter
 *
 * @param uart The UART information
 */
void HAL_UART_MspInit(UART_HandleTypeDef* uart)
{
    GPIO_InitTypeDef GPIO_InitStruct;
    if(uart->Instance == USART2)
    {
        /* Peripheral clock enable */
        __HAL_RCC_USART2_CLK_ENABLE();

        /*
         * USART2 GPIO Configuration
         * PA2     ------> USART2_TX
         * PA3     ------> USART2_RX
         */
        GPIO_InitStruct.Pin = GPIO_PIN_2|GPIO_PIN_3;
        GPIO_InitStruct.Mode = GPIO_MODE_AF_PP;
        GPIO_InitStruct.Pull = GPIO_NOPULL;
        GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
        // Alternate function -- that of UART
        GPIO_InitStruct.Alternate = GPIO_AF1_USART2;
        HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);
    }

}

/**
 * Magic function called by HAL layer to de-initialize the
 * UART hardware. Something we never do, but we put this
 * in here for the sake of completeness.
 *
 * @note: Only works for UART2, the one connected to the USB serial
 * converter
 *
 * @param uart The UART information
 */
void HAL_UART_MspDeInit(UART_HandleTypeDef* uart)
{
    if(uart->Instance == USART2)
    {
        /* Peripheral clock disable */
        __HAL_RCC_USART2_CLK_DISABLE();

        /*
         * USART2 GPIO Configuration
         * PA2     ------> USART2_TX
         * PA3     ------> USART2_RX
         */
        HAL_GPIO_DeInit(GPIOA, GPIO_PIN_2|GPIO_PIN_3);
    }
}
```

列表 11-5：重置计数程序

### 多个配置项

假设我们想在闪存中保持多个配置变量。问题在于，闪存并不是普通内存。当你将一个值存储到闪存变量中后，除非擦除包含该变量的整个内存页面，否则无法更改该值。

当每页存储一个变量时（这种做法非常浪费），这能正常工作，但如果我们想在内存中存储多个配置变量并更新其中一个怎么办呢？这需要一点工作。下面是处理过程：

1.  将所有配置变量保存在 RAM 中。

1.  在 RAM 中更新你需要更改的值。

1.  擦除闪存中的所有配置变量。（擦除闪存页面。）

1.  将 RAM 版本复制回闪存。

列表 11-6 显示了在 `.config` 部分声明配置结构并更新 `struct` 中值的代码框架。

```
struct config {
    char name[16];    // Name of the unit
    uint16_t sensors[10]; // The type of sensor connected to each input
    uint32_t reportTime;  // Seconds between reports
    // ... Lots of other stuff
};
struct config theConfig __attribute__((section ".config")); // The configuration

static void updateReportTime(const uint32_t newReportTime) {

    // <Prepare flash>

    struct config currentConfig = config;
    currentConfig.reportTime = newReportTime;

 // <Erase flash>
    writeFlash(&config, &currentConfig, sizeof(currentConfig));

    // <Lock flash>
}
```

列表 11-6：更新闪存中的配置

闪存有许多问题。如前所述，第一个问题是必须擦除整个页面才能写入一个字。这需要时间来写入页面到闪存，并且在写入过程中系统可能会断电或重启。如果发生这种情况，写入将不完整，且你的配置数据将会损坏。

解决此问题的方法是拥有两个配置区，一个主配置区和一个备份配置区，每个配置区都包含一个校验和。程序首先尝试读取主配置，如果校验和不正确，则读取第二个配置。因为每次只写入一个配置，所以你可以相当确定主配置或备份配置中的一个是正确的。

另一个关于闪存的问题是，它存在*内存磨损*。你只能进行有限次数的编程/擦除周期，之后内存会变得损坏。根据使用的闪存类型，这个周期可以在 100,000 到 1,000,000 次之间。所以，使用闪存存储一个预期每月更改一次的配置是可以的。但如果用它来存储每秒更改几次的内容，闪存很快就会磨损。

也有一些方法可以绕过闪存的限制进行编程。你还可以为你的系统添加外部内存芯片，这些芯片没有闪存的设计限制。

### 现场定制示例

假设我们在一家制造报警器的公司工作。这些报警器会发送给报警服务公司，由它们在最终用户的现场进行安装。现在，如果 Joe 的报警公司和钓具店安装的报警面板在启动时显示的是 Acme 报警制造商的标志，他会不高兴的。Joe 注重品牌，他希望显示自己的标志，这意味着我们需要给客户提供一种定制标志的方法。我们可以为标志预留一块内存空间：

```
MEMORY
{
    FLASH (rx)     : ORIGIN = 0x8000000,       LENGTH = 52K
    LOGO (r)       : ORIGIN = 0x8000000 + 52K, LENGTH = 8K
    CONFIG (rw)    : ORIGIN = 0x8000000 + 60K, LENGTH = 4K
    RAM (xrw)      : ORIGIN = 0x20000000,      LENGTH = 8K
}
```

现在问题是，我们如何将标志导入系统？我们可以在工厂进行编程，但这意味着每次我们发货时，都需要有人打开盒子，插入设备，编程标志，然后再放回盒子里，这是一项昂贵的操作。

但是，我们可以让客户自己做这件事。我们可以给他们一根电缆和一些软件，让他们自己编程设置标志。我们可以将这个功能作为一项特性来出售，允许客户在需要时更新设备的标志。

编程可以通过我们用来将代码加载到闪存的相同硬件和软件来完成，或者我们可以编写一个板载程序，从串行线获取数据并将其编程到`LOGO`内存中。

更换标志是一个简单的定制操作。而且，如果更换过程出错，坏的标志不会影响系统的正常运行。然而，更换固件则是另外一回事。

### 固件升级

在运行软件的同时升级软件是有点棘手的，但有几种方法可以做到这一点。最简单的方法之一是将闪存划分为三个部分：

1.  引导加载程序

1.  程序部分 1

1.  程序部分 2

引导加载程序是一个非常小的程序，永远不会被升级。它的工作相对简单，希望我们能第一次就做对。程序部分包含程序的完整版本，它们还包含程序版本号和校验和。

引导加载程序的任务是决定应该使用哪个程序部分。它会验证两个部分的校验和，然后根据以下计算决定使用哪个部分：

```
if ((bad checksum1) and (good checksum2)) use section2
if ((good checksum1) and (bad checksum2)) use section1
if (both good) use the section with the highest version number
if (both bad) blink the emergency light; we're bricked
```

这是总体思路，但我们跳过了一些记录步骤。例如，`.isr_vector`部分中的中断表需要进行修改，以确保所有的中断都能正确地指向相应的位置。

## 总结

内存是有限资源，尤其是在进行嵌入式编程时。你需要确切知道你的内存位置以及如何最大限度地利用它。

链接器的工作是将你的程序的各个部分连接起来，生成一个可以加载到内存中的程序。对于简单的程序，默认配置效果很好。然而，随着你深入更高级的系统，你将需要更精确地控制有限内存资源的使用，因此理解链接器对于成为一名有效的嵌入式程序员至关重要。

## 编程问题

1.  修改配置程序（Listing 11-6），使得`CONFIG`段不再从页面边界开始。会发生什么？

1.  修改配置程序，使其打印一个完整的数字，而不是打印一个单一的重置数字。

1.  链接器脚本定义了多个符号，用于指示内存区域的起始和结束。检查链接器脚本或链接器映射，以找到定义文本区域起始和结束的符号。使用这些符号，打印文本区域的大小。使用`arm-none-eabi-size`命令验证你的结果。

1.  使用相同的技术打印分配的栈空间量。

1.  高级：打印剩余的栈空间。这需要使用`asm`关键字将栈寄存器的当前值读取到变量中。

1.  了解二进制文件中的内容非常有用，GNU 工具链提供了多个程序来实现这一点。检查以下命令的文档：

    1.  `objdump`，用于转储目标文件信息

    1.  `nm`，用于列出文件中的符号

    1.  `ar`，用于创建库或从中提取信息和文件

    1.  `readelf`，用于显示 elf（程序）文件的信息
