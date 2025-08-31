

## 第十五章：15 管理复杂项目



![](img/opener.jpg)

大多数汇编语言源文件并不是独立的程序。通常，你必须调用各种标准库或其他例程，而这些例程并未在你的主程序中定义，因为试图将这样的代码写入你的应用程序中会工作量过大（而且是糟糕的编程实践）。

例如，ARM 并没有提供像读、写或放置等用于 I/O 操作的机器指令。本书中的函数包含了成千上万行的源代码来完成这些操作。对于小程序，使用单个源文件没有问题，但对于大型程序来说，这就变得很繁琐。如果你必须将成千上万行代码合并到你的简单程序中，这将使编程变得相当困难，而且编译速度也会变得很慢。此外，一旦你调试并测试了程序的一个大型部分，在对程序的其他部分进行小改动时，还需要重新组装这部分代码，简直是浪费时间。试想一下，在一台快速的 PC 上，仅仅做了一行代码的修改后，你可能要等 20 或 30 分钟才能重新组装程序！

*大型编程*是软件工程师用来描述减少大型软件项目开发时间的过程、方法和工具的术语。虽然每个人对“庞大”的定义不同，*分离编译*是支持大型编程的更流行技术之一。首先，你将大型源文件拆分成可管理的块。然后你将单独的文件编译成目标代码模块。最后，你将目标模块链接在一起形成完整的程序。如果你需要对其中一个模块进行小改动，你只需要重新组装那个模块，而不是整个程序。

本章描述了 Gas 和操作系统提供的分离编译工具，以及如何有效地在程序中使用这些工具。

### 15.1 .include 指令

.include 指令在源文件中出现时，会将程序输入从当前文件切换到 .include 指令操作数字字段中指定的文件。第 1.5 节，“*aoaa.inc* 包含文件”，在 第 10 页 中描述了 .include 指令，它允许你将来自不同源文件的代码包含到当前汇编中，从而构建包含公共常量、类型、源代码和其他 Gas 项目的文本文件。正如该节中所提到的，.include 指令的语法是：

```
.include "`filename`"
```

其中，文件名必须是一个有效的文件名。

根据本书的约定，Gas 包含文件有 *.inc*（包含）后缀。然而，Gas 并不要求包含文件必须有这个后缀；任何包含 Gas 汇编语言源代码的文件名都可以使用。Gas 会在 .include 指令的位置将指定的文件合并到编译过程中。你可以在包含的文件中嵌套 .include 语句；也就是说，在汇编过程中被包含进另一个文件的文件可以再包含第三个文件。

单独使用 .include 指令并不会提供独立编译。你*可以*使用 .include 将一个大源文件拆分成多个模块，在编译时将这些模块合并。以下示例将在编译程序时包含 *print.inc* 和 *getTitle.inc* 文件：

```
.include  "print.inc"
.include  "getTitle.inc"
```

现在你的程序将从模块化中受益。遗憾的是，你并不会节省任何开发时间。 .include 指令会在编译时将源文件插入到 .include 的位置，正如你亲自输入了这些代码一样。Gas 仍然需要编译代码，而这需要时间。如果你要将大量源文件（比如一个庞大的库）包含到你的汇编中，编译过程可能需要*永远*。

通常情况下，你*不应该*使用 .include 指令来包含源代码，如前面的示例所示，因为这样做的代码无法利用独立编译。相反，应该使用 .include 指令将一组公共常量、类型、外部过程声明及其他类似项目插入到程序中。通常，汇编语言的包含文件*不包含*任何机器代码（宏之外；有关详细信息，请参见第十三章）。以这种方式使用 .include 文件的目的将在你看到外部声明如何工作后变得更加清晰（请参见下一页第 15.3 节，“汇编单元与外部指令”）。

如果你的汇编语言源文件有 *.S* 后缀，你也可以使用 #include "filename" 指令来包含源文件。这通常更可取，因为你可以在这样的包含文件中使用 CPP 指令（而在标准 .include 文件中无法使用）。本章的其余部分假定使用 #include 指令，而不是 .include。

### 15.2 忽略重复的包含操作

当你开始开发复杂的模块和库时，最终你会发现一个大问题：有些头文件需要包含其他头文件。技术上来说，这本身并没有问题，但问题出现在一个头文件包含了另一个头文件，而那个第二个头文件又包含了另一个头文件，依此类推，最终导致最后一个头文件又包含了第一个头文件。

头文件间接包含自身有两个问题。首先，这会在编译器中创建一个无限循环。编译器会不断地重复包含这些文件，直到内存耗尽或发生其他错误。其次，当 Gas 第二次包含头文件时，它会开始对重复的符号定义进行强烈抱怨。毕竟，第一次读取头文件时，它处理了文件中的所有声明；第二次读取时，它将所有这些符号视为重复符号。

解决递归包含文件的标准技术，是使用条件汇编让 Gas 忽略包含文件的内容，这一点 C/C++程序员十分熟悉。（请参见第十三章，了解有关 CPP 和 Gas CTL 的条件汇编讨论。）技巧是将一个#ifdef（如果已定义）语句放在包含文件的所有语句周围。指定一个未定义的符号作为#ifdef 操作数（我通常使用包含文件的文件名，将句点替换为下划线）。然后，在#ifdef 语句之后立即定义该符号；通常使用数字等式，并将符号赋值为常量 1。以下是该#ifdef 用法的一个示例：

```
#ifdef  myinclude_inc   // Filename: myinclude.inc
#define myinclude_inc 1

 `Put all the source code lines for the include file here.`

// The following statement should be the last nonblank line
// in the source file:

#endif  // myinclude_inc
```

如果你尝试第二次包含*myinclude.inc*，#ifdef 指令将导致 Gas（实际上是 CPP）跳过所有文本，直到对应的#endif 指令，从而避免重复定义错误。

### 15.3 汇编单元与外部指令

*汇编单元*是源文件及其直接或间接包含的任何文件的集合。一个汇编单元在汇编后生成一个单独的*.o*（目标）文件。链接器将多个目标文件（由 Gas 或其他编译器，如 GCC 生成）合并为一个可执行文件。本节的主要目的，实际上也是整章的目的，是描述这些汇编单元（*.o*文件）在链接过程中如何相互传递链接信息。汇编单元是创建汇编语言模块化程序的基础。

要使用 Gas 的汇编单元功能，你必须创建至少两个源文件。一个文件包含第二个文件使用的一组变量和过程。第二个文件使用这些变量和过程，而无需知道它们是如何实现的。

从技术上讲，#include 指令为您提供了创建此类模块化程序所需的所有功能。您可以创建多个模块，每个模块包含一个特定的例程，并通过使用#include 指令根据需要将这些模块包含到您的汇编语言程序中。然而，如果您使用这种方法，在编译时包含一个已调试的例程仍然会浪费时间，因为每当您汇编主程序时，Gas 都必须重新编译无错误的代码。一个更好的解决方案是预先汇编已调试的模块，并将目标代码模块链接在一起，使用 Gas 的.global 和.extern 指令，本节将介绍这些内容。

本书到目前为止出现的所有程序都是独立的模块，这些模块链接到一个 C/C++主程序，而不是另一个汇编语言模块。在每个程序中，汇编语言的“主程序”都命名为 asmMain，它实际上是一个与 C++兼容的函数，这个函数是由通用的*c.cpp*程序从其主程序中调用的。例如，考虑第 9 页中的 Listing 1-3 中的 asmMain 主体（适用于 Linux 和 Pi OS 系统）：

```
// Listing1-3.S
//
// A simple Gas module that contains
// an empty function to be called by
// the C++ code in Listing 1-2.

 .text

// Here is the asmMain function:

        .global asmMain
        .align  2    // Guarantee 4-byte alignment.
asmMain:

// Empty function just returns to C++ code:

        ret          // Returns to caller
```

这个.global asmMain 语句已经包含在每个拥有 asmMain 函数但没有定义或解释的程序中。现在是时候处理这个疏漏了。

Gas 源文件中的普通符号对该特定源文件是*私有*的，其他源文件无法访问（当然，除非这些源文件直接包含了包含这些私有符号的文件）。也就是说，大多数符号的*作用域*仅限于该特定源文件中的代码行以及它包含的任何文件。 .global 指令告诉 Gas 使指定符号对汇编单元*全局*，即在链接阶段，其他汇编单元也能访问这个符号。通过在本书中的示例程序中放置.global asmFunc 语句，这些示例程序使 asmMain 符号对包含它们的源文件全局化，以便*c.cpp*程序可以调用 asmMain 函数。

如您所记得，macOS 要求全局名称前面加上下划线前缀。这意味着，如果您希望该源文件在 macOS 下进行汇编，您应该使用.global _asmMain 和 _asmMain：。*aoaa.inc*头文件以可移植的方式解决了这个问题，但*Listing1-3.S*中的代码没有包含*aoaa.inc*。

仅仅将符号设为公共是无法在另一个源文件中使用该符号的。想要使用该符号的源文件还必须声明该符号为*外部*。这会通知链接器，在外部声明的文件使用该符号时，它将需要修补公共符号的地址。例如，*c.cpp*源文件在以下代码行中将 asmMain 符号声明为外部（顺便提一下，这个声明也定义了外部符号 getTitle）：

```
// extern "C" namespace prevents
// "name mangling" by the C++
// compiler.

extern "C"
{
    // Here's the external function,
    // written in assembly language,
    // that this program will call:

    void asmMain(void);
    int readLine(char *dest, int maxLen);
};
```

在这个例子中，readLine 实际上是一个在 *c.cpp* 源文件中定义的 C++ 函数。C/C++ 并没有明确的公共声明。相反，如果你在源文件中提供了一个函数的源代码，并声明该函数为外部符号，C/C++ 会通过外部声明自动将该符号设置为公共符号。

当你在程序中放置 `.extern` 指令时，Gas 会将该声明视为任何其他符号声明。如果该符号已存在，Gas 会生成符号重定义错误。通常，你应将所有外部声明放在源文件的开头，以避免作用域/前向引用问题。

从技术上讲，使用 `.extern` 指令是可选的，因为 Gas 假设你使用的任何符号如果在源文件中没有定义，都是外部符号。如果链接器在链接所有其他目标代码模块时未找到符号，它将报告实际的未定义符号。然而，显式地使用 `.extern` 定义外部符号是良好的编程风格，可以明确表达你的意图，方便其他人阅读你的源代码。

因为 `.global` 指令并不会真正定义符号，所以它的位置不像 `.extern` 指令那样重要。一些程序员将所有全局声明放在源文件的开头；也有一些程序员将全局声明放在符号定义之前（如我在大多数相同程序中对 asmMain 符号所做的那样）。这两种做法都可以。

因为一个源文件中的公共符号可以被多个汇编单元使用，所以会出现一个问题：你必须在所有使用该符号的文件中重复 `.extern` 指令。对于少数符号来说，这不是一个大问题。然而，随着外部符号数量的增加，在多个源文件中维护这些外部符号变得繁琐。

Gas 解决方案与 C/C++ 解决方案相同：*头文件*。这些文件只是包含外部（以及其他）声明的包含文件，这些声明在多个汇编单元中是共同的。头文件之所以得名，是因为包含它们的 `include` 语句通常出现在源文件的开头（“头”部分），用来将它们的代码注入到源文件中。这也恰好是 Gas 中包含文件的主要用途。

### 15.4 使用单独编译创建字符串库

第十四章提供了几个字符串处理函数的示例，附带宏和字符串结构体。这些函数和声明的问题在于，它们必须被剪切并粘贴到任何希望使用它们的源文件中。创建一个包含宏、结构体和外部符号定义的头文件，然后将各个函数编译成 *.o* 文件以便与那些希望使用这些函数的程序链接，显然是更好的选择。本节描述了如何为这些字符串函数创建可链接的目标模块。

字符串库的头文件是 *strings.inc*：

```
// strings.inc
//
// String function header file for the assembly
// language string format

#ifndef  strings_inc
#define strings_inc 1

// Assembly language string data structure:

            struct  string, -16
            dword   string.allocPtr // At offset -16
            word    string.maxlen   // At offset -8
            word    string.len      // At offset -4
            byte    string.chars    // At offset 0

            // Note: characters in string occupy offsets
            // 0 ... in this structure.

            ends    string

// str.buf
//
// Allocate storage for an empty string
// with the specified maximum size:

            .macro  str.buf strName, maxSize
            .align  4   // Align on 16-byte boundary.
            .dword  0   // NULL ptr for allocation ptr
            .word   \maxSize
            .word   0
\strName:   .space  ((\maxSize+16) & 0xFFFFFFF0), 0
            .endm

// str.literal
//
// Allocate storage for a string buffer and initialize
// it with a string literal:

            .macro  str.literal strName, strChars
            .align  4   // Align on 16-byte boundary.
            .dword  0   // NULL ptr for allocation ptr
            .word   len_\strName    // string.maxlen
            .word   len_\strName    // string.len

            // Emit the string data and compute the
            // string's length:

\strName:   .ascii  "\strChars"
len_\strName=       .-\strName
            .byte   0   // Zero-terminating byte

 // Ensure object is multiple of 16 bytes:

            .align  4
            .endm

// str.len
//
//          Return the length of the string pointed at by X0.
//          Returns length in X0

            .macro  str.len
            ldr     w0, [x0, #string.len]
            .endm

// External declarations:

            .extern str.cpy
            .extern str.cmp
            .extern str.substr
            .extern str.bufInit
            .extern str.alloc
            .extern str.free

// This would be a good place to include external
// declarations for any string functions you write.

#endif
```

*str.cpy* 函数的源文件在 *str.cpy.S* 中：

```
// str.cpy.S
//
// A str.cpy string copy function

            #include    "aoaa.inc"
          ❶ #include    "strings.inc"

            .code

///////////////////////////////////////////////////////////
//
// str.cpy
//
// Copies the data from one string variable to another
//
// On entry:
//
// X0- Pointer to source string (string struct variable)
// X1- Pointer to destination string
//
// On exit:
//
// Carry flag clear if no errors, carry is set if
// the source string will not fit in the destination.

 ❷ proc    str.cpy, public

            locals  str_cpy
            qword   str_cpy.saveV0
            qword   str_cpy.saveX2X3
            dword   str_cpy.saveX4
            byte    str_cpy.stkSpace,64
            endl    str_cpy

            enter   str_cpy.size

            // Preserve X2 ... X4 and V0:

            str     q0,     [fp, #str_cpy.saveV0]
            stp     x2, x3, [fp, #str_cpy.saveX2X3]
            str     x4,     [fp, #str_cpy.saveX4]

            // Ensure the source will fit in the destination
            // string object:

            ldr     w4, [x0, #string.len]
            ldr     w3, [x1, #string.maxlen]
            cmp     w4, w3
            bhi     str.cpy.done    // Note: carry is set.

            // Set the length of the destination string
            // to the length of the source string.

            str     w4, [x1, #string.len]

            // X4 contains the number of characters to copy;
            // while this is greater than 16, copy 16 bytes
            // at a time from source to dest:

            mov     x2, x0  // Preserve X0 and X1.
            mov     x3, x1

cpy16:      ldr     q0, [x2], #16
            str     q0, [x3], #16
            subs    w4, w4, #16
            bhi     cpy16

// At this point, you have fewer than 16 bytes to copy. If
// W4 is not 0, just copy 16 remaining bytes (you know,
// because of the string data structure, that if you have at
// least 1 byte left to copy, you can safely copy
// 16 bytes):

            beq     setZByte    // Skip if 0 bytes.

            ldr     q0, [x2]
            str     q0, [x3]

// Need to add a zero-terminating byte to the end of
// the string. Note that maxlen does not include the
// 0 byte, so it's always safe to append the 0
// byte to the end of the string.

setZByte:   ldr     w4,  [x0, #string.len]
            strb    wzr, [x1, w4, uxtw]

            adds    wzr, wzr, wzr   // Clears the carry

str.cpy.done:
            ldr     q0,     [fp, #str_cpy.saveV0]
            ldp     x2, x3, [fp, #str_cpy.saveX2X3]
            ldr     x4,     [fp, #str_cpy.saveX4]
            leave
            endp    str.cpy
```

*str.cpy.S* 源文件是通过包含 *strings.inc* 头文件 ❶ 并从 第十四章第 819 页 的列表 14-2 中剪切粘贴 str.cpy 函数 ❷ 创建的。注意 proc 宏后的 public 参数。这会导致 proc 宏为 str.cpy 符号发出 .global 指令，使得该函数可以被其他源文件访问。

*str.cmp.S*、*str.substr.S*、*str.alloc.S*、*str.free.S* 和 *str.bufInit.S* 源文件是以类似方式从它们对应的函数（在 第十四章 中）创建的。我不会在这里包含这些源文件，因为它们是冗余的并且占用太多空间，但你可以在在线源文件中找到副本，网址是 *[`<wbr>artofarm<wbr>.randallhyde<wbr>.com`](https://artofarm.randallhyde.com)*。

如果你尝试使用常规构建命令来组装这些模块中的任何一个，系统会报错，提示缺少符号。这是因为这些模块不是独立的汇编语言程序。在接下来的部分，我会描述如何正确构建这些库模块；与此同时，这里有一些简单的命令可以在不出错的情况下组装这些文件（尽管会有警告）：

```
./build -c str.cpy
./build -c str.cmp
./build -c str.substr
./build -c str.bufInit
./build -c str.alloc
./build -c str.free
```

这将组装文件而不运行链接器（-c 表示*仅编译*），分别生成文件 *str.cpy.o*、*str.cmp.o*、*str.substr.o*、*str.bufInit.o*、*str.alloc.o* 和 *str.free.o*。当然，下一个问题是如何将这些文件与应用程序链接。列表 15-1 是从 第十四章 中各种 asmMain 函数的合并版本，它们调用了 str.cpy、str.cmp 和 str.substr 函数。

```
// Listing15-1.S
//
// A program that calls various string functions

            #include    "aoaa.inc"
 #include    "strings.inc"

///////////////////////////////////////////////////////////

            .data

            str.buf     destination, 256
            str.literal src,    "String to copy"
            str.literal left,   "some string"
            str.literal right1, "some string"
            str.literal right2, "some string."
            str.literal right3, "some string"

            str.buf     smallDest, 32
            str.literal dest,   "Initial destination string"

//                             1111111111222222222233333
//                   01234567890123456789012345678901234
str.literal source, "Hello there, world! How's it going?"

fmtStr:     .asciz      "source='%s', destination='%s'\n"
ltFmtStr:   .asciz      "Left ('%s') is less than right ('%s')\n"
gtFmtStr:   .asciz      "Left ('%s') is greater than right ('%s')\n"
eqFmtStr:   .asciz      "Left ('%s') is equal to right ('%s')\n"

successStr: .asciz      "substr('%s', %2d, %3d)= '%s'\n"
failureStr: .asciz      "substr('%s', %2d, %3d) failed\n"

///////////////////////////////////////////////////////////

            .code
ttlStr:     wastr  "Listing15-1"

// Standard getTitle function
// Returns pointer to program name in X0

            proc    getTitle, public
            lea     x0, ttlStr
            ret
            endp    getTitle

///////////////////////////////////////////////////////////
//
// prtResult
//
// Utility function to print the result of a string
// comparison:

            proc    prtResult

            mov     x2, x1
            mov     x1, x0
            mstr    x1, [sp]
            mstr    x2, [sp, #8]
 beq     strsEQ
            bhi     strGT

            // Must be LT at this point.

            lea     x0, ltFmtStr
            b       printf

strsEQ:     lea     x0, eqFmtStr
            b       printf

strGT:      lea     x0, gtFmtStr
            b       printf

            endp    prtResult

///////////////////////////////////////////////////////////
//
// testSubstr
//
// Utility function to test call to str.substr
//
// On entry:
// X0, X1, X2, X3 -- str.substr parameters

            proc    testSubstr

            locals  testSS
            byte    testSS.stkspace, 64
            endl    testSS

            enter   testSS.size

            lea     x5, successStr
            bl      str.substr
            bcc     success
            lea     x5, failureStr

success:
            mov     x4, x3
            mov     x3, x2
            mov     x2, x1
            mov     x1, x0
            mov     x0, x5
            mstr    x1, [sp]
            mstr    x2, [sp, #8]
            mstr    x3, [sp, #16]
            mstr    x4, [sp, #24]
            bl      printf
            leave
            endp    testSubstr

///////////////////////////////////////////////////////////
//
// Main program to test the code:

            proc    asmMain, public

            locals  lcl
            byte    stkSpace, 64
            endl    lcl

            enter   lcl.size      // Reserve space for locals.

            lea     x0, src
            lea     x1, destination
            bl      str.cpy

            mov     x2, x1
            mov     x1, x0
            lea     x0, fmtStr
            mstr    x1, [sp]
            mstr    x2, [sp, #8]
            bl      printf

            lea     x0, left
            lea     x1, right1
            bl      str.cmp
            bl      prtResult

            lea     x0, left
            lea     x1, right2
            bl      str.cmp
            bl      prtResult

            lea     x0, left
            lea     x1, right3
            bl      str.cmp
            bl      prtResult

            lea     x0, source
            mov     x1, #0
            mov     x2, #11
            lea     x3, dest
            bl      testSubstr

            lea     x0, source
            mov     x1, #20
            mov     x2, #15
            lea     x3, dest
            bl      testSubstr

            lea     x0, source
            mov     x1, #20
            mov     x2, #20
 lea     x3, dest
            bl      testSubstr

            lea     x0, source
            mov     x1, #40
            mov     x2, #20
            lea     x3, dest
            bl      testSubstr

            lea     x0, source
            mov     x1, #0
            mov     x2, #100
            lea     x3, smallDest
            bl      testSubstr

AllDone:    leave
            endp    asmMain
```

如果你尝试使用以下命令来构建这个程序

```
./build Listing15-1
```

系统会抱怨它无法在提供的目标文件中找到符号 str.cpy、str.cmp 和 str.substr。不幸的是，*build* shell 脚本不支持在多个目标模块之间链接（除了 *c.cpp* 和指定文件的目标文件）。因此，你必须指定一个明确的 g++ 命令来处理所有文件：

```
g++ -DisMacOS c.cpp Listing15-1.S str.cpy.o str.cmp.o str.substr.o -o Listing15-1
```

在 Linux 或 Pi OS（而不是 macOS）下编译代码时，命令行参数 -DisMacOS 应该更改为 -DisLinux。正如你在第 1.10.1 节“在多个操作系统下汇编程序”中从 第 36 页 回忆的那样，*build* shell 脚本会确定操作系统并发出一个 g++ 命令行定义（-Dxxxx 选项），以使操作系统对汇编源文件（特别是 *aoaa.inc* 头文件）可知。由于这个 g++ 命令会尝试汇编 *Listing15-1.S* 源文件（其中包含 *aoaa.inc*），因此命令行必须包含 isMacOS 或 isLinux 的定义，否则汇编将失败。

这个 g++ 命令将编译 *c.cpp*，汇编 *Listing15-1.S*，并将它们的目标文件与 *str.cpy.o*、*str.cmp.o* 和 *str.substr.o* 目标文件链接在一起。当然，这假设你已经汇编了 *str.*.S* 源文件，并且它们的目标文件位于当前目录。Listing 15-1 中的示例程序并没有调用 str.alloc、str.free 或 str.bufInit 函数，因此无需将它们各自的目标代码文件链接进来，尽管这么做不会产生错误。

这是构建所有这些文件并生成和运行 Listing 15-1 可执行文件所需的完整命令集：

```
% g++ -c -DisMacOS str.cpy.S
% g++ -c -DisMacOS str.cmp.S
% g++ -c -DisMacOS str.substr.S
% g++ -DisMacOS c.cpp Listing15-1.S str.cpy.o str.cmp.o str.substr.o -o Listing15-1
% ./Listing15-1
Calling Listing15-1:
source='String to copy', destination='String to copy'
Left ('some string') is equal to right ('some string')
Left ('some string') is less than right ('some string.')
Left ('some string') is greater than right ('some string')
substr('Hello there, world! How's it going?',  0,  11)= 'Hello there'
substr('Hello there, world! How's it going?', 20,  15)= 'How's it going?'
substr('Hello there, world! How's it going?', 20,  20)= 'How's it going?'
substr('Hello there, world! How's it going?', 40,  20)= ''
substr('Hello there, world! How's it going?',  0, 100) failed
listing15-1 terminated
```

诚然，为了编译和链接一个简单的源文件，这需要大量的输入。你可以通过将所有命令放入一个文本文件并作为 shell 脚本执行（类似于*build*脚本）来解决这个问题，但有一个更好的方法：makefile。

### 15.5 引入 Makefile

本书中使用的*build*文件比手动命令要方便得多，用于构建上一节中的示例。不幸的是，*build* 支持的构建机制仅适用于少数固定的源文件。虽然你可以轻松地构建一个 shell 脚本来编译大型汇编项目中的所有文件，但这在很大程度上违背了使用单独汇编文件的目的，因为运行脚本文件会重新汇编项目中的每个源文件。虽然你可以使用复杂的命令行功能来避免部分问题，但使用 makefile 会更加简便。

*makefile* 是一种脚本语言（最早在 Unix 发布时设计）用于指定如何基于某些条件执行一系列命令。最简单的形式下，makefile 可以完全像 shell 脚本一样运行；你可以在文本文件中列出一系列命令，并让 make 程序执行它们。当然，如果你仅仅这样做，那么使用 shell 脚本并不会带来额外的好处；如果你打算使用 makefile，应该利用 make 的特性。

make 程序是一个可执行文件，就像 Gas (as) 或 GCC 一样。由于 make 并不是 Linux 或 macOS 系统的一部分，因此在使用之前，你必须先获取 make 程序。幸运的是，大多数 Linux 和 macOS 发行版都预装了 make（如果你能运行 GCC，当然也能运行 make）。你可以通过命令行执行它，像这样：

```
make `optionalArguments`
```

如果你在命令行执行 make 时没有任何参数，make 会查找一个名为 *Makefile* 的文件，并尝试处理该文件中的命令。对于许多项目来说，这非常方便。如果你将所有源文件放在一个目录中（可能包含子目录），并将一个名为 *Makefile* 的单一 makefile 放在其中，那么你只需进入该目录并执行 make，就可以在最小的麻烦下构建项目。

如果你愿意，可以使用与*Makefile*不同的文件名。不过，必须注意的是，使用命令行时，你必须在文件名之前加上 make -f 选项，像这样：

```
make -f mymake.mak
```

你不需要给文件名加上 *.mak* 扩展名，但这是使用自定义名称的 makefile 时的常见约定。

make 程序提供了许多命令行选项，你可以使用 --help 列出常用选项。你可以在线查阅 make 文档（或在命令行中输入 man make）以了解其他命令行选项的说明，但它们大多数是高级选项，通常在大多数任务中不需要。

当然，要实际使用 make，你需要创建 makefile。以下小节描述了 make 脚本语言及一些常见的 makefile 约定。

#### 15.5.1 基本 Makefile 语法

makefile 是一个标准的 ASCII 文本文件，包含如下格式的多行（或多次出现此序列）：

```
`target`: `dependencies`
    `commands`
```

代码中的所有组件——目标、依赖项和命令——都是可选的。目标项是某种标识符或文件名，如果存在，必须从源行的第 1 列开始。依赖项项是目标正确构建所依赖的文件名列表。命令项是一个或多个命令行命令的列表，命令前必须至少有一个制表符。

考虑以下 makefile，它构建了一组字符串库函数（注意，每个 g++ 命令前都有一个制表符）：

```
all:
    g++ -c -DisMacOS str.cpy.S
    g++ -c -DisMacOS str.cmp.S
    g++ -c -DisMacOS str.substr.S
    g++ -DisMacOS c.cpp Listing15-1.S str.cpy.o str.cmp.o \
         str.substr.o -o Listing15-1
```

如果这些命令出现在一个名为 *Makefile* 的文件中，并且你执行 make，它们将像命令行解释器那样执行，假如它们出现在 shell 脚本中。

考虑对前一个 makefile 的以下修改：

```
executable:
  g++ -c -DisMacOS Listing15-1.S
  g++ -DisMacOS c.cpp Listing15-1.o str.cpy.o str.cmp.o str.substr.o -o Listing15-1

library:
  g++ -c -DisMacOS str.cpy.S
  g++ -c -DisMacOS str.cmp.S
  g++ -c -DisMacOS str.substr.S
```

这将构建命令分成两组：一组由可执行文件标签指定，另一组由库标签指定。

如果你运行 make 而不指定任何命令行选项，它只会执行文件中第一个目标后面的命令。因此，在这个例子中，如果你单独运行 make，它将汇编 *Listing15-1.S*，编译 *c.cpp*，并尝试将（生成的）*c.obj* 与 *str.cpy.o*、*str.cmp.o*、*str.substr.o* 和 *Listing15-1.o* 链接在一起。假设你之前已经编译过字符串函数，这应该能够成功生成 Listing15-1 可执行文件（无需重新编译字符串函数）。

为了让 make 处理库目标之后的命令，你必须将目标名称作为 make 命令行参数指定：

```
make library
```

这个 make 命令编译 *str.cpy.S*、*str.cmp.S* 和 *str.substr.S*。如果你执行一次这个命令（并且此后不再更改字符串函数），只需单独执行 make 命令即可生成可执行文件。如果你想明确表示自己在构建可执行文件，也可以使用 make executable。

在命令行上指定你想要构建的目标是非常有用的。然而，随着项目变大，源文件和库模块增多，时刻跟踪哪些源文件需要重新编译可能会变得繁琐且容易出错。如果你不小心，在修改了某个不常见的库模块之后，可能会忘记重新编译它，然后不明白为什么应用程序仍然失败。make 的依赖选项通过让你自动化构建过程，帮助你避免这些问题。

在 makefile 中，目标后可以跟一个或多个由空格分隔的依赖关系：

```
target: `dependency1` `dependency2` `dependency3` ...
```

依赖关系可以是目标名称（出现在该 makefile 中的目标）或文件名。如果依赖关系是目标名称（且不是文件名），make 将执行与该目标相关的命令。考虑以下 makefile（如果在 Linux 或 Pi OS 下编译，请确保将这个例子中的-DisMacOS 命令行选项更改为-DisLinux）：

```
executable:
  g++-c -DisMacOS Listing15-1.S
  g++-DisMacOS c.cpp Listing15-1.o str.cpy.o str.cmp.o str.substr.o -o Listing15-1

library:
  g++ -c -DisMacOS str.cpy.S
  g++-c -DisMacOS str.cmp.S
  g++-c -DisMacOS str.substr.S

all: library executable
```

这个代码中的 all 目标没有任何与之相关的命令。相反，all 目标依赖于库和可执行目标，因此它将执行与这些目标相关的命令，从库开始。这是因为在将关联的对象模块链接成可执行程序之前，必须先构建库的目标文件。all 标识符是 makefile 中常见的目标。事实上，它通常是 makefile 中出现的第一个或第二个目标。

如果一个目标：依赖关系行变得过长，导致难以阅读（make 不太关心行的长度），你可以通过在行的末尾加上反斜杠字符（\）来将该行分为多行。make 程序会将以反斜杠结尾的源行与 makefile 中的下一行合并。反斜杠必须是行的最后一个字符；行尾不允许有空白字符（制表符和空格）。

目标名称和依赖关系也可以是文件名。将文件名指定为目标名称通常是为了告诉 make 系统如何构建该特定文件。例如，你可以将当前的示例重写如下：

```
executable:
  g++ -c -DisMacOS Listing15-1.1
  g++ -DisMacOS c.cpp Listing15-1.o str.cpy.o str.cmp.o str.substr.o -o Listing15-1

library: str.cpy.o str.cmp.o str.substr.o

str.cpy.o:
  g++ -c -DisMacOS str.cpy.S

str.cmp.o:
  g++ -c -DisMacOS str.cmp.S

str.substr.o:
  g++ -c -DisMacOS str.substr.S

all: library executable
```

当依赖关系与一个文件名目标相关联时，你可以将目标：依赖关系语句理解为“目标依赖于依赖关系”。在处理命令时，make 会比较作为目标和依赖关系文件名指定的文件的修改日期/时间戳。

如果目标的日期/时间比*任何*依赖项都要旧（或者目标文件不存在），make 将执行目标后面的命令。如果目标文件的修改日期/时间比*所有*依赖文件都要新，make 将不会执行命令。如果目标后的某个依赖项本身是其他地方的目标，make 将首先执行那个命令（以查看它是否修改了目标对象，改变其修改日期/时间，可能导致 make 执行当前目标的命令）。如果目标或依赖项只是一个标签（而不是文件名），make 将把它的修改日期/时间视为比任何文件都旧。

考虑以下对正在运行的 makefile 示例的修改：

```
Listing15-1:Listing15-1.o str.cpy.o str.cmp.o str.substr.o
  gcc -DisMacOS c.cpp Listing15-1.o str.cpy.o str.cmp.o str.substr.o -o Listing15-1

Listing15-1.o:
  g++ -c -DisMacOS Listing15-1.S

str.cpy.o:
  g++ -c -DisMacOS str.cpy.S

str.cmp.o:
  g++ -c -DisMacOS str.cmp.S

str.substr.o:
  g++ -c -DisMacOS str.substr.S
```

这段代码删除了所有和库的目标，因为它们被证明是不必要的，并将可执行文件更改为 Listing15-1，即最终的目标可执行文件。

因为 str.cpy.o、str.cmp.o、str.substr.o 和 Listing15-1.o 都是目标（以及文件名），make 将首先处理这些目标。之后，make 会将 Listing15-1 的修改日期/时间与这四个目标文件的修改日期/时间进行比较。如果 Listing15-1 比任何一个目标文件旧，make 将执行 Listing15-1 目标行之后的命令（编译*c.cpp*并将其与目标文件链接）。如果 Listing15-1 比其依赖的目标文件新，make 将不执行该命令。

对于所有依赖的目标文件，在处理 Listing15-1 目标时，同样的过程会递归发生。处理 Listing15-1 目标时，make 还会处理 str.cpy.o、str.cmp.o、str.substr.o 和 Listing15-1.o 目标（按此顺序）。在每种情况下，make 会将*.o*文件的修改日期/时间与相应的*.S*文件进行比较。如果*.o*文件的修改日期/时间比*.S*文件新，make 将返回并继续处理 Listing15-1 目标，而不执行任何操作；如果*.o*文件比*.S*文件旧（或不存在），make 将执行相应的 g++命令生成新的*.o*文件。

如果 Listing15-1 比所有*.o*文件更新（并且它们都比*.S*文件更新），那么执行 make 时只会报告 Listing15-1 是最新的，但不会执行 makefile 中的任何命令。如果任何文件是过时的（因为它们已被修改），这个 makefile 将只编译和链接必要的文件，以使 Listing15-1 保持最新。

到目前为止，makefile 有一个相当严重的缺陷：缺少一个重要的依赖项。由于所有的*.S*文件都包含*aoaa.inc*文件，因此*aoaa.inc*的更改可能会要求重新编译这些*.S*文件。Listing 15-2 在*Listing15-2.mak* makefile 中添加了这个依赖，并演示了如何在 makefile 中通过在行首使用#字符来添加注释。

```
# Listing15-2.mak
#
# makefile for Listing15-1

Listing15-1:Listing15-1.o str.cpy.o str.cmp.o str.substr.o
  gcc -DisMacOS c.cpp Listing15-1.o str.cpy.o str.cmp.o str.substr.o -o Listing15-1

Listing15-1.o:aoaa.inc Listing15-1.S
  gcc -c -DisMacOS Listing15-1.S

str.cpy.o:aoaa.inc str.cpy.S
  gcc -c -DisMacOS str.cpy.S

str.cmp.o:aoaa.inc str.cmp.S
  gcc -c -DisMacOS str.cmp.S

str.substr.o:aoaa.inc str.substr.S
  gcc -c -DisMacOS str.substr.S
```

这是执行 make（在 macOS 下）的一个示例：

```
% make -f Listing15-2.mak
gcc -c -DisMacOS Listing15-1.S
gcc -c -DisMacOS str.cpy.S
gcc -c -DisMacOS str.cmp.S
gcc -c -DisMacOS str.substr.S
gcc -DisMacOS c.cpp Listing15-1.o str.cpy.o str.cmp.o str.substr.o -o Listing15-1
```

在 Linux 或 Pi OS 下执行此命令时，请不要忘记将所有 `-DisMacOS` 命令行选项改为 `-DisLinux`，并确保所有命令在第一列有一个制表符。如果希望能够自动为任何操作系统编译代码，只需复制 *构建* 脚本中的代码，该脚本设置了带有适当命令行选项的 shell 变量，如列表 15-3 所示。

```
# Listing15-3.mak
#
# makefile for Listing15-1 with dependencies that will
# automatically set up the define for the OS

❶ unamestr=`uname`

Listing15-1:Listing15-1.o str.cpy.o str.cmp.o str.substr.o
    gcc -D$(unamestr) c.cpp Listing15-1.o str.cpy.o str.cmp.o \
        str.substr.o -o Listing15-1

Listing15-1.o:aoaa.inc Listing15-1.S
  ❷ gcc -c -D$(unamestr) Listing15-1.S

str.cpy.o:aoaa.inc str.cpy.S
    gcc -c -D$(unamestr) str.cpy.S

str.cmp.o:aoaa.inc str.cmp.S
    gcc -c -D$(unamestr) str.cmp.S

str.substr.o:aoaa.inc str.substr.S
    gcc -c -D$(unamestr) str.substr.S
```

第一个语句 ❶ 是一个 makefile *宏*（或 *变量*）的示例。OS 命令 `uname` 将显示操作系统（内核）名称。在 Linux 系统下，这将被替换为字符串 `Linux`，而在 macOS 系统下则为字符串 `Darwin`（macOS 内核的内部名称）。

Makefile 宏使用延迟执行。这意味着宏 `unamestr` 实际上包含了文本 `uname`，而 `uname` 命令将在 make 程序展开 `unamestr` 宏时执行。make 程序将展开 `-D$(unamestr)` 命令行选项，生成 `-D` `uname` ❷。反引号（`）告诉 make 执行该命令并用命令打印的文本替换它：操作系统内核名称。

唯一的问题是 `uname` 命令会打印出 Linux 或 Darwin，因此 `-D` 命令定义了这两个符号之一。*构建* 脚本将这些字符串转换为 `isMacOS` 和 `isLinux`。我最初这样做是因为在基于 Linux 的汇编语言程序中，符号 `Linux` 可能会出现。不幸的是，这种符号翻译技巧在 makefile 中无法生效，因此我修改了 *aoaa.inc*，使其既接受 `Linux` 和 `Darwin`，也接受 `inLinux` 和 `inMacOS`。我修改了 *aoaa.inc* 来进行翻译，并在使用这些符号时取消定义 `Linux` 或 `Darwin`：

```
// Makefiles define the symbols Darwin (for macOS)
// and Linux (for Linux) rather than isMacOS and
// isLinux. Deal with that here:

#ifdef Darwin
    #define isMacOS (1)
    #undef isLinux
    #undef Darwin
#endif
#ifdef Linux
    #define isLinux (1)
    #undef isMacOS
    #undef Linux
#endif
```

这是执行 `make` 命令以构建列表 15-3 中代码的过程（假设没有已经创建的目标文件）：

```
% make -f Listing15-3.mak
g++ -c -D`uname` Listing15-1.S
g++ -c -D`uname` str.cpy.S
g++ -c -D`uname` str.cmp.S
g++ -c -D`uname` str.substr.S
g++ -D`uname` c.cpp Listing15-1.o str.cpy.o str.cmp.o \
    str.substr.o -o Listing15-1
```

注意，`-D` `uname` 会根据操作系统的不同翻译为 `-DLinux` 或 `-DDarwin`。

#### 15.5.2 Make Clean 和 Touch

在大多数专业制作的 makefile 中，你会发现一个常见的目标是 `clean`，它删除一组适当的文件，以便下次执行 makefile 时强制重新构建整个系统。该命令通常会删除与项目相关的所有 `*.o` 文件和可执行文件。

列表 15-4 提供了一个针对列表 15-3 中 makefile 的示例清理目标。

```
# Listing15-4.mak
#
# makefile for listing15-1 with dependencies that will
# automatically set up the define for the OS
#
# Demonstrates the clean target

unamestr=`uname`

Listing15-1:Listing15-1.o str.cpy.o str.cmp.o str.substr.o
    gcc -D$(unamestr) c.cpp listing15-1.o str.cpy.o str.cmp.o \
        str.substr.o -o Listing15-1

Listing15-1.o:aoaa.inc Listing15-1.S
    gcc -c -D$(unamestr) Listing15-1.S
str.cpy.o:aoaa.inc str.cpy.S
    gcc -c -D$(unamestr) str.cpy.S

str.cmp.o:aoaa.inc str.cmp.S
    gcc -c -D$(unamestr) str.cmp.S

str.substr.o:aoaa.inc str.substr.S
    gcc -c -D$(unamestr) str.substr.S
clean:
    rm str.cpy.o
    rm str.cmp.o
    rm str.substr.o
    rm Listing15-1.o
    rm c.o
    rm Listing15-1
```

执行命令

```
% make -f Listing15-4.mak clean
```

将删除与项目相关的所有可执行文件和目标文件。

要强制重新编译单个文件（而不手动编辑和修改它），可以使用 Unix 工具 `touch`。该程序接受一个文件名作为参数，并更新文件的修改日期/时间（不会修改文件内容）。例如，在使用列表 15-4 中的 makefile 构建 *Listing15-1.S* 后，如果执行以下命令

```
touch Listing15-1.S
```

然后重新执行列表 15-4 中的 makefile，make 将重新汇编 *Listing15-1.S* 中的代码，重新编译 *c.cpp*，并生成一个新的可执行文件。

### 15.6 使用归档程序生成库文件

许多常见的项目重用开发者很久以前创建的代码，或者来自开发者组织外部的代码。这些代码库相对*静态*：在使用它们的项目开发过程中，它们很少发生变化。特别地，通常不会将库的构建包含在某个特定项目的 makefile 中。一个特定的项目可能会在 makefile 中列出库文件作为依赖项，但假设这些库文件已经在其他地方构建，并作为整体提供给项目。

更重要的是，库与一组目标代码文件之间有一个主要的区别：打包。当你在处理大量独立的目标文件时，尤其是当你需要处理大量的库目标文件时，事情变得麻烦。一个库可能包含几十、几百甚至几千个目标文件。列出所有这些目标文件（或者仅列出一个项目使用的文件）是一个繁琐的工作，可能会导致一致性错误。

解决这个问题的常见方法是将目标文件组合成一个单独的包（文件），称为*库文件*。在 Linux 和 macOS 下，库文件通常有一个*.a*后缀（其中*a*代表*归档*）。对于许多项目，你会得到一个库文件，它将特定的库模块打包在一起。你将这个文件提供给链接器，当构建程序时，链接器会自动从库中提取它需要的目标模块。这是一个重要的点：在构建可执行文件时包含一个库并不会自动将库中的所有代码插入到可执行文件中。链接器足够智能，能够仅提取它需要的目标文件，忽略它不使用的目标文件（记住，库只是一个包含大量目标文件的包）。

如何创建一个库文件？简短的回答是：“通过使用归档程序（ar）。”以下是它的基本语法

```
ar rcs `libname.a` `list-of-.o-files`
```

其中，libname.a 是你想要生成的库文件的名称，而 list-of-.o-files 是你想要收集到库中的目标文件名称列表（以空格分隔）。例如，以下命令将 *print.o* 和 *getTitle.o* 文件合并成一个库模块（*aoaalib.a*）：

```
ar rcs aoaalib.a getTitle.o print.o
```

rcs 组件实际上是一系列三个命令选项。r 选项告诉命令替换归档中已有的（如果有）目标文件；c 选项表示创建归档（如果你是将目标文件添加到现有归档文件中，通常不需要指定此选项）；s 选项表示为归档文件添加索引，或者如果索引已存在，则更新索引。（有关更多 ar 命令行选项，请参阅第 15.9 节“更多信息”，见第 887 页。）

一旦你有了一个库模块，你可以像指定目标文件一样在链接器（或 ld 或 gcc）的命令行中指定它。例如，如果你构建了一个*strings.a*库模块来保存*str.cpy.o*、*str.cmp.o*、*str.substr.o*、*str.bufInit.o*、*str.free.o*和*str.alloc.o*目标文件，并且你想要将*strings.a*与 Listing 15-1 中的程序链接，你可以使用以下命令：

```
g++ -DisMacOS c.cpp Listing15-1.S strings.a -o Listing15-1
```

Listing 15-5 是一个 makefile 的示例，它将构建*strings.a*库文件。

```
# Listing15-5.mak
#
# makefile to build the string.a library file

unamestr=`uname`

strings.a:str.cpy.o str.cmp.o str.substr.o str.bufInit.o \
            str.alloc.o str.free.o
    ar rcs strings.a str.cpy.o str.cmp.o str.substr.o \
        str.bufInit.o str.alloc.o str.free.o

str.cpy.o:aoaa.inc str.cpy.S
    g++ -c -D$(unamestr) str.cpy.S

str.cmp.o:aoaa.inc str.cmp.S
    g++ -c -D$(unamestr) str.cmp.S

str.substr.o:aoaa.inc str.substr.S
    g++ -c -D$(unamestr) str.substr.S

str.bufInit.o:aoaa.inc str.bufInit.S
    g++ -c -D$(unamestr) str.bufInit.S

str.free.o:aoaa.inc str.free.S
    g++ -c -D$(unamestr) str.free.S

str.alloc.o:aoaa.inc str.alloc.S
    g++ -c -D$(unamestr) str.alloc.S

 clean:
    rm -f strings.a
    rm -f str.cpy.o
    rm -f str.cmp.o
    rm -f str.substr.o
    rm -f str.bufInit.o
    rm -f str.alloc.o
    rm -f str.free.o
```

Listing 15-6 修改了 Listing 15-5 的 makefile，通过使用*strings.a*库模块来构建代码。

```
# Listing15-6.mak
#
# makefile that uses the string.a library file

unamestr=`uname`

Listing15-1:Listing15-1.o strings.a
    g++ -D$(unamestr) c.cpp Listing15-1.o strings.a -o Listing15-1

Listing15-1.o:aoaa.inc Listing15-1.S
    g++ -c -D$(unamestr) Listing15-1.S

lib:
    rm -f strings.a
    rm -f str.*.o
    make -f Listing15-5.mak

clean:
    rm -f Listing15-1
    rm -f c.o
    rm -f Listing15-1.o
```

请注意，clean 命令不会删除库文件。如果你想进行干净的库构建，只需在运行 make 时指定 lib 命令行选项：

```
make -f Listing15-6.mak lib
```

一般来说，你应该独立构建库代码和应用程序代码。大多数时候，库是预构建的，你不需要重新构建它。然而，*strings.a*必须是应用程序的依赖项，因为如果库发生变化，你很可能也需要重新构建应用程序。

另一个在处理库文件时有用的 Unix 工具是 nm（names）。nm 工具会列出库模块中找到的所有全局名称。例如，命令

```
nm strings.a
```

列出了在*strings.a*库文件中找到的所有（全局）符号（它比较长，我这里不提供打印输出）。

### 15.7 管理目标文件对程序大小的影响

程序中的链接基本单元是目标文件。当将目标文件组合成可执行文件时，链接器会将单个目标文件中的所有数据合并到最终的可执行文件中。即使主程序没有调用目标模块中的所有函数（直接或间接）或没有使用该目标文件中的所有数据，这一点仍然成立。如果你把 100 个例程放入一个汇编语言源文件中，并将其编译成目标模块，那么链接器将在最终的可执行文件中包含所有 100 个例程的代码，即使你只使用其中的一个。

为了避免这种情况，你可以将这 100 个例程拆分成 100 个单独的目标模块，并将生成的 100 个目标文件合并成一个库文件。当链接器处理这个库文件时，它会挑选出包含程序使用的函数的单个目标文件，并只将该文件合并到最终的可执行文件中。

通常，这比将一个包含 100 个函数的单一目标文件链接到程序要高效得多。然而，在某些情况下，将多个函数合并到一个目标文件中是有充分理由的。首先，考虑一下链接器将目标文件合并到可执行文件时会发生什么。为了确保正确的对齐，每当链接器从目标文件中提取一个段/区段（例如 .code 区段）时，它会添加足够的填充，以确保该区段中的数据按照该区段指定的对齐边界进行对齐。大多数区段有一个默认的 16 字节区段对齐。这意味着链接器将把从目标文件中链接的每个区段对齐到 16 字节边界上。

通常，这并不是一个大问题，特别是当你的过程很大时。然而，如果这 100 个过程都非常简短（每个只有几个字节），你就会浪费大量空间。确实，在现代计算机上，几百字节的浪费空间并不是大问题。不过，将其中一些过程合并成一个目标模块（即使你并不调用所有这些过程）来填补一些浪费的空间，可能会更实用。寻找自然配对的元素或有依赖关系的元素，例如 alloc 和 free。然而，别做得过头。一旦你超过了对齐边界，无论是因为填充浪费空间，还是因为你包含了永远不会被调用的代码，你都会浪费空间。

### 15.8 接下来的内容

如果你用汇编语言编写大型应用程序，你将需要将源代码分解成多个模块，并自动化从这些模块构建应用程序。本章首先讨论了 Gas 在模块之间共享外部和公共符号的机制。接着介绍了用于从多个源文件构建应用程序的 make 工具，然后讲解了如何通过链接器和归档工具构建库模块。

一个大型库代码的来源是操作系统内核（如 macOS、Linux 或 Pi OS）。然而，不要将操作系统库函数链接到你的应用程序中；当应用程序运行时，这些代码已经存在于内存中。调用操作系统函数时，你需要使用操作系统 API 调用序列。下一章将讨论如何在 Linux（Pi OS）和 macOS（Darwin）内核中调用操作系统函数。

### 15.9 更多信息

+   有关 makefile 的信息，请访问以下网站：

    +   Computer Hope: *[`www.computerhope.com/unix/umake.htm`](https://www.computerhope.com/unix/umake.htm)*

    +   GNU make: *[`www.gnu.org/software/make/`](https://www.gnu.org/software/make/)*

    +   Wikipedia: *[`en.wikipedia.org/wiki/Make_(software)`](https://en.wikipedia.org/wiki/Make_(software))*

+   另外，查看以下关于 make 的书籍：

    +   罗伯特·梅克伦堡，*《用 GNU Make 管理项目：GNU Make 构建一切的威力》*，第三版（O'Reilly Media，2004 年）。你也可以在线访问本书，网址是 *[`<wbr>www<wbr>.oreilly<wbr>.com<wbr>/openbook<wbr>/make3<wbr>/book<wbr>/index<wbr>.csp`](https://www.oreilly.com/openbook/make3/book/index.csp)*。

    +   约翰·格雷厄姆-卡明，*《GNU Make 书籍》*，第一版（No Starch Press，2015 年）

    +   安德鲁·奥拉姆和史蒂夫·塔尔博特，*《用 Make 管理项目》*（O'Reilly & Associates，2004 年）

+   请参阅 *[`<wbr>man7<wbr>.org<wbr>/linux<wbr>/man<wbr>-pages<wbr>/man1<wbr>/ar<wbr>.1<wbr>.html`](https://man7.org/linux/man-pages/man1/ar.1.html)* 以获取 ar 命令行选项的完整列表。你也可以输入 ar --help 或 man ar 获取在线帮助。
