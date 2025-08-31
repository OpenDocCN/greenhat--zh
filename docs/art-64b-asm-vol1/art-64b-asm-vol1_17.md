# 第十五章：管理复杂项目

![](img/chapterart.png)

大多数汇编语言源文件并不是独立的程序。它们是多个源文件的组成部分，可能用不同的语言编写，编译并链接在一起，形成复杂的应用程序。*大型编程*是软件工程师用来描述处理大型软件项目开发的过程、方法和工具的术语。

虽然每个人对什么是*大型*程序有自己的理解，*单独编译*是支持大型编程的流行技术之一。使用单独编译，你首先将大型源文件拆分成易于管理的部分。然后你将这些单独的文件编译成目标代码模块。最后，你将目标模块链接在一起，形成一个完整的程序。如果你需要对某个模块进行小的修改，你只需要重新组装那个模块；不需要重新组装整个程序。一旦你调试并测试了代码的大部分，当你对程序的其他部分进行小的修改时，继续组装相同的代码就是浪费时间。想象一下，在一台快速的 PC 上，你只改动了一行代码，却要等 20 或 30 分钟才能重新组装程序！

以下章节描述了 MASM 提供的单独编译工具，以及如何有效地在程序中使用这些工具，以实现模块化和减少开发时间。

## 15.1 `include` 指令

当源文件中遇到 `include` 指令时，它会在 `include` 指令的位置将指定的文件合并到编译中。`include` 指令的语法是

```
include `filename`
```

其中 `filename` 是一个有效的文件名。根据约定，MASM 的 include 文件具有 *.inc*（include）后缀，但任何包含 MASM 汇编语言源代码的文件都可以正常使用。被包含的文件可以在汇编过程中再次包含其他文件。

单独使用 `include` 指令并不能实现单独编译。你*可以*使用 `include` 指令将一个大型源文件拆分成多个模块，并在编译时将这些模块合并在一起。下面的示例会在程序编译时包括 *print.inc* 和 *getTitle.inc* 文件：

```
include  print.inc
include  getTitle.inc
```

现在你的程序将受益于模块化。可惜，你并不会节省任何开发时间。`include` 指令在编译时将源文件插入到 `include` 指令的位置，就像你自己手动输入这些代码一样。MASM 仍然需要编译代码，而这需要时间。如果你在汇编过程中包含了大量源文件（例如一个庞大的库），编译过程可能需要*永远*。

一般来说，你*不*应该使用`include`指令来包含前面示例中展示的源代码^(1)。相反，你应该使用`include`指令将一组通用的常量、类型、外部过程声明以及其他类似的项目插入程序中。通常，汇编语言的包含文件*不*包含任何机器代码（宏外的部分；详细信息请参见第十三章）。以这种方式使用`include`文件的目的，在你看到外部声明如何工作的之后，会变得更加清晰。

## 15.2 忽略重复的包含操作

当你开始开发复杂的模块和库时，你最终会发现一个大问题：一些头文件需要包含其他头文件。其实，这并不是什么大问题，但问题出现在当一个头文件包含另一个头文件，而第二个头文件又包含另一个，第三个头文件又包含另一个……最后那个头文件又包含第一个头文件时。现在*这*就是一个大问题，因为它会在编译器中产生一个无限循环，并导致 MASM 抱怨重复的符号定义。毕竟，第一次读取头文件时，它会处理该文件中的所有声明；第二次读取时，它会将这些符号视为重复符号。

忽略重复包含的标准技巧，C/C++程序员非常熟悉，就是使用条件汇编让 MASM 忽略包含文件的内容。（请参见第十三章中的“条件汇编（编译时决策）”）诀窍是将一个`ifndef`（*如果未定义*）语句放在包含文件的所有语句周围。你将包含文件的文件名作为`ifndef`操作数，使用下划线替换点（或其他任何未定义的符号）。然后，在`ifndef`语句之后，立即定义该符号（通常使用数值等式并将该符号赋值为常数 0）。以下是这个`ifndef`用法的一个示例：

```
 ifndef  myinclude_inc   ; Filename: myinclude.inc
myinclude_inc =       0

`Put all the source code lines for the include file here`

; The following statement should be the last non-blank line
; in the source file:

              endif  ; myinclude_inc
```

在第二次包含时，MASM 会直接跳过包含文件的内容（包括任何`include`指令），这样就避免了无限循环和所有的重复符号定义。

## 15.3 汇编单元和外部指令

*汇编单元*是一个源文件及其直接或间接包含的任何文件的集合。汇编单元在汇编后会生成一个单独的*.obj*文件。微软链接器将多个目标文件（由 MASM 或其他编译器生成，如 MSVC）结合成一个单独的可执行单元（*.exe*文件）。本节的主要目的（实际上，这一整章的目的）是描述这些汇编单元（*.obj*文件）在链接过程中如何相互传递链接信息。汇编单元是创建汇编语言模块化程序的基础。

要使用 MASM 的汇编单元功能，你必须创建至少两个源文件。一个文件包含第二个文件使用的变量和过程。第二个文件使用这些变量和过程，但不知道它们是如何实现的。

与其使用 `include` 指令来创建模块化程序（因为每次汇编主程序时，MASM 都必须重新编译无错误的代码，浪费时间），不如预先汇编调试好的模块并将目标代码模块链接在一起，这样的解决方案要好得多。这正是 `public`、`extern` 和 `externdef` 指令所允许你做的事情。

从技术上讲，本书到目前为止出现的所有程序都是单独汇编的模块（这些模块恰好与 C/C++ 主程序链接，而不是与其他汇编语言模块链接）。名为 `asmMain` 的汇编语言主程序只是一个与 C++ 兼容的函数，通用的 *c.cpp* 程序从其主程序中调用了这个函数。考虑 第二章 中 Listing 2-1 的 `asmMain` 函数体：

```
; Here is the "asmMain" function.

        public  asmMain
asmMain proc
         .
         .
         .
asmMain endp
```

每个包含 `asmMain` 函数的程序中都包含了 `public asmMain` 语句，而没有任何定义或解释。好了，现在是时候解决这个遗漏了。

MASM 源文件中的普通符号是该源文件*私有*的，其他源文件无法访问这些符号（当然，前提是这些源文件没有直接包含包含这些私有符号的文件）。也就是说，源文件中大多数符号的*作用域*仅限于该源文件中的代码行（以及它包含的任何文件）。`public` 指令告诉 MASM 将指定符号设置为全局符号——在链接阶段，其他汇编单元可以访问它。通过本书示例程序中的 `public asmMain` 语句，这些示例程序将 `asmMain` 符号设置为包含它们的源文件的全局符号，以便 *c.cpp* 程序可以调用 `asmMain` 函数。

仅仅将符号设置为公共符号不足以在另一个源文件中使用该符号。想要使用该符号的源文件还必须将该符号声明为*外部*符号。这会通知链接器，当包含外部声明的文件使用该符号时，链接器必须修补该公共符号的地址。例如，*c.cpp* 源文件在以下代码行中将 `asmMain` 符号定义为外部符号（顺便提一下，这个声明还定义了外部符号 `getTitle` 和 `readLine`）：

```
// extern "C" namespace prevents
// "name mangling" by the C++
// compiler.

extern "C"
{
 // asmMain is the assembly language
    // code's "main program":

    void asmMain(void);

    // getTitle returns a pointer to a
    // string of characters from the
    // assembly code that specifies the
    // title of that program (which makes
    // this program generic and usable
    // with a large number of sample
    // programs in "The Art of 64-Bit
    // Assembly").

    char *getTitle(void);

    // C++ function that the assembly
    // language program can call:

    int readLine(char *dest, int maxLen);

};
```

请注意，在这个示例中，`readLine` 是一个在 *c.cpp* 源文件中定义的 C++ 函数。C/C++ 没有显式的公共声明。相反，如果你为一个源文件中的函数提供源代码，并且声明该函数为外部函数，C/C++ 会通过外部声明自动将该符号设置为公共符号。

MASM 实际上有两个外部符号声明指令：`extern`和`externdef`。^(2)这两个指令的语法是：

```
extern    `symbol`:`type`  {`optional_list_of_symbol:type_pairs`}
externdef `symbol`:`type`  {`optional_list_of_symbol:type_pairs`}
```

其中，`symbol`是你想要从另一个汇编单元中使用的标识符，而`type`是该符号的数据类型。数据类型可以是以下任何一种：

+   `proc`，表示该符号是一个过程（函数）名称或语句标签

+   任意 MASM 内建数据类型（例如`byte`、`word`、`dword`、`qword`、`oword`等）

+   任意用户自定义数据类型（例如结构体名称）

+   `abs`，表示一个常量值

`abs`类型并不是用来声明通用外部常量（例如`someConst = 0`）。像这样的纯常量声明通常会出现在头文件（即包含文件）中，本节稍后会描述这一点。相反，`abs`类型通常保留给基于对象模块中代码偏移量的常量。例如，如果你在一个汇编单元中有以下代码，

```
 public someLen
someStr   byte   "abcdefg"
someLen   =      $-someStr
```

`someLen`的类型，在`extern`声明中，将是`abs`。

这两个指令使用逗号分隔的列表来允许多个符号声明；例如：

```
extern p:proc, b:byte, d:dword, a:abs
```

然而，我认为，如果将每个声明限制为单个符号，你的程序会更易于阅读。

当你在程序中放置`extern`指令时，MASM 会将该声明视为任何其他符号声明。如果符号已存在，MASM 会生成符号重定义错误。通常，应该将所有外部声明放在源文件的开始部分，以避免作用域或前向引用问题。由于`public`指令实际上并不定义符号，因此`public`指令的位置并不像`extern`指令那么关键。有些程序员将所有公共声明放在源文件的开头；其他程序员则将公共声明放在符号定义之前（如我在大多数相同程序中对`asmMain`符号所做的那样）。这两种位置都可以。

## 15.4 MASM 中的头文件

由于一个源文件中的公共符号可以被多个汇编单元使用，因此会出现一个小问题：你必须在所有使用该符号的文件中复制`extern`指令。对于少量符号来说，这不是什么大问题。然而，随着外部符号数量的增加，跨多个源文件维护这些外部符号会变得繁琐。MASM 的解决方案与 C/C++相同：头文件。

*头文件*是包含多个汇编单元间共有的外部（以及其他）声明的包含文件。之所以叫做*头文件*，是因为通常会在使用它们的源文件的开始部分（*头部*）插入包含语句。这实际上是 MASM 中包含文件的主要用途：包含外部（以及其他）公共声明。

## 15.5 `externdef`指令

当你开始使用包含大量库模块（汇编单元）的头文件时，你会很快发现`extern`指令存在一个大问题。通常，你会为一大套库函数创建一个头文件，每个函数可能会出现在自己的汇编单元中。有些库函数可能会使用同一*库模块*（一组目标文件）中的其他函数；因此，该特定库函数的源文件可能会想要包含库的头文件，以便引用其他库函数的外部名称。

不幸的是，如果头文件包含当前源文件中函数的外部定义，则会发生符号重新定义错误：

```
; header.inc
           ifndef   header_inc
header_inc =        0

           extern  func1:proc
           extern  func2:proc

           endif   ; header_inc
```

以下源文件的汇编会产生错误，因为`func1`已经在*header.inc*头文件中定义：

```
; func1.asm

           include header.inc

           .code

func1      proc
             .
             .
             .
           call func2
             .
             .
             .
func1      endp
           end
```

C/C++不会遇到这个问题，因为`external`关键字既作为公共声明，也作为外部声明。

为了克服这个问题，MASM 引入了`externdef`指令。该指令类似于 C/C++中的`external`指令：当符号在源文件中不存在时，它表现得像一个`extern`指令，而当符号在源文件中定义时，它表现得像一个`public`指令。此外，同一符号的多个`externdef`声明可以出现在源文件中（尽管如果出现多个声明，它们应该指定相同的符号类型）。考虑修改后的*header.inc*头文件，使用`externdef`定义：

```
; header.inc
           ifndef     header_inc
header_inc =          0

 externdef  func1:proc
           externdef  func2:proc

           endif      ; header_inc
```

使用这个头文件，*func1.asm*汇编单元将会正确编译。

## 15.6 分离编译

很早在第十一章的“MASM 包含指令”中，我就开始将`print`和`getTitle`函数放入头文件中，这样我就可以在每个需要使用这些函数的源文件中简单地包含它们，而无需手动将这些函数复制粘贴到每个程序中。显然，这些是应该制作成汇编单元并与其他程序链接的好例子，而不是在汇编过程中被包含进来。

清单 15-1 是一个头文件，其中包含了必要的`print`和`getTitle`声明：^(3)

```
; aoalib.inc - Header file containing external function
;              definitions, constants, and other items used
;              by code in "The Art of 64-Bit Assembly."

            ifndef      aoalib_inc
aoalib_inc  equ         0

; Constant definitions:

; nl (newline constant):

nl          =           10

; SSE4.2 feature flags (in ECX):

SSE42       =       00180000h       ; Bits 19 and 20
AVXSupport  =       10000000h       ; Bit 28

; CPUID bits (EAX = 7, EBX register):

AVX2Support  =      20h             ; Bit 5 = AVX

**********************************************************

; External data declarations:

            externdef   ttlStr:byte

**********************************************************

; External function declarations:

            externdef   print:qword
            externdef   getTitle:proc

; Definition of C/C++ printf function that
; the print function will call (and some
; AoA sample programs call this directly,
; as well).

            externdef   printf:proc

            endif       ; aoalib_inc
```

清单 15-1：*aoalib.inc*头文件

清单 15-2 包含了在第十一章“MASM 包含指令”中使用的`print`函数，并将其转换为一个汇编单元。

```
; print.asm - Assembly unit containing the SSE/AVX dynamically
;             selectable print procedures.

            include aoalib.inc

            .data
            align   qword
print       qword   choosePrint     ; Pointer to print function

            .code

; print - "Quick" form of printf that allows the format string to
;         follow the call in the code stream. Supports up to five
;         additional parameters in RDX, R8, R9, R10, and R11.

; This function saves all the Microsoft ABI–volatile,
; parameter, and return result registers so that code
; can call it without worrying about any registers being
; modified (this code assumes that Windows ABI treats
; YMM6 to YMM15 as nonvolatile).

; Of course, this code assumes that AVX instructions are
; available on the CPU.

; Allows up to 5 arguments in:

;  RDX - Arg #1
;  R8  - Arg #2
;  R9  - Arg #3
;  R10 - Arg #4
;  R11 - Arg #5

; Note that you must pass floating-point values in
; these registers as well. The printf function
; expects real values in the integer registers. 

; There are two versions of this program, one that
; will run on CPUs without AVX capabilities (no YMM
; registers) and one that will run on CPUs that
; have AVX capabilities (YMM registers). The difference
; between the two is which registers they preserve
; (print_SSE preserves only XMM registers and will
; run properly on CPUs that don't have YMM register
; support; print_AVX will preserve the volatile YMM
; registers on CPUs with AVX support).

; On first call, determine if we support AVX instructions
; and set the "print" pointer to point at print_AVX or
; print_SSE:

choosePrint proc
            push    rax             ; Preserve registers that get
            push    rbx             ; tweaked by CPUID
            push    rcx
            push    rdx

            mov     eax, 1
            cpuid
            test    ecx, AVXSupport ; Test bit 28 for AVX
            jnz     doAVXPrint

            lea     rax, print_SSE  ; From now on, call
            mov     print, rax      ; print_SSE directly

; Return address must point at the format string
; following the call to this function! So we have
; to clean up the stack and JMP to print_SSE.

            pop     rdx
            pop     rcx
            pop     rbx
            pop     rax
            jmp     print_SSE

doAVXPrint: lea     rax, print_AVX  ; From now on, call
            mov     print, rax      ; print_AVX directly

; Return address must point at the format string
; following the call to this function! So we have
; to clean up the stack and JMP to print_AUX.

            pop     rdx
            pop     rcx
            pop     rbx
            pop     rax
            jmp     print_AVX

choosePrint endp

; Version of print that will preserve volatile
; AVX registers (YMM0 to YMM3):

thestr      byte "YMM4:%I64x", nl, 0
print_AVX   proc

; Preserve all the volatile registers
; (be nice to the assembly code that
; calls this procedure):

            push    rax
            push    rbx
            push    rcx
            push    rdx
            push    r8
            push    r9
            push    r10
            push    r11

; YMM0 to YMM7 are considered volatile, so preserve them:

            sub     rsp, 256
            vmovdqu ymmword ptr [rsp + 000], ymm0
            vmovdqu ymmword ptr [rsp + 032], ymm1
            vmovdqu ymmword ptr [rsp + 064], ymm2
            vmovdqu ymmword ptr [rsp + 096], ymm3
            vmovdqu ymmword ptr [rsp + 128], ymm4
            vmovdqu ymmword ptr [rsp + 160], ymm5
            vmovdqu ymmword ptr [rsp + 192], ymm6
            vmovdqu ymmword ptr [rsp + 224], ymm7

            push    rbp

returnAdrs  textequ <[rbp + 328]>

            mov     rbp, rsp
            sub     rsp, 256
            and     rsp, -16

; Format string (passed in RCX) is sitting at
; the location pointed at by the return address;
; load that into RCX:

            mov     rcx, returnAdrs

; To handle more than three arguments (four counting
; RCX), you must pass data on stack. However, to the
; print caller, the stack is unavailable, so use
; R10 and R11 as extra parameters (could be just
; junk in these registers, but pass them just
; in case).

 mov     [rsp + 32], r10
            mov     [rsp + 40], r11
            call    printf

; Need to modify the return address so
; that it points beyond the zero-terminating byte.
; Could use a fast strlen function for this, but
; printf is so slow it won't really save us anything.

            mov     rcx, returnAdrs
            dec     rcx
skipTo0:    inc     rcx
            cmp     byte ptr [rcx], 0
            jne     skipTo0
            inc     rcx
            mov     returnAdrs, rcx

            leave
            vmovdqu ymm0, ymmword ptr [rsp + 000]
            vmovdqu ymm1, ymmword ptr [rsp + 032]
            vmovdqu ymm2, ymmword ptr [rsp + 064]
            vmovdqu ymm3, ymmword ptr [rsp + 096]
            vmovdqu ymm4, ymmword ptr [rsp + 128]
            vmovdqu ymm5, ymmword ptr [rsp + 160]
            vmovdqu ymm6, ymmword ptr [rsp + 192]
            vmovdqu ymm7, ymmword ptr [rsp + 224]
            add     rsp, 256
            pop     r11
            pop     r10
            pop     r9
            pop     r8
            pop     rdx
            pop     rcx
            pop     rbx
            pop     rax
            ret
print_AVX   endp

; Version that will run on CPUs without
; AVX support and will preserve the
; volatile SSE registers (XMM0 to XMM3):

print_SSE   proc

; Preserve all the volatile registers
; (be nice to the assembly code that
; calls this procedure):

            push    rax
            push    rbx
            push    rcx
            push    rdx
            push    r8
            push    r9
 push    r10
            push    r11

; XMM0 to XMM3 are considered volatile, so preserve them:

            sub     rsp, 128
            movdqu  xmmword ptr [rsp + 00],  xmm0
            movdqu  xmmword ptr [rsp + 16],  xmm1
            movdqu  xmmword ptr [rsp + 32],  xmm2
            movdqu  xmmword ptr [rsp + 48],  xmm3
            movdqu  xmmword ptr [rsp + 64],  xmm4
            movdqu  xmmword ptr [rsp + 80],  xmm5
            movdqu  xmmword ptr [rsp + 96],  xmm6
            movdqu  xmmword ptr [rsp + 112], xmm7

            push    rbp

returnAdrs  textequ <[rbp + 200]>

            mov     rbp, rsp
            sub     rsp, 128
            and     rsp, -16

; Format string (passed in RCX) is sitting at
; the location pointed at by the return address;
; load that into RCX:

            mov     rcx, returnAdrs

; To handle more than three arguments (four counting
; RCX), you must pass data on stack. However, to the
; print caller, the stack is unavailable, so use
; R10 and R11 as extra parameters (could be just
; junk in these registers, but pass them just
; in case):

            mov     [rsp + 32], r10
            mov     [rsp + 40], r11
            call    printf

; Need to modify the return address so
; that it points beyond the zero-terminating byte.
; Could use a fast strlen function for this, but
; printf is so slow it won't really save us anything.

            mov     rcx, returnAdrs
            dec     rcx
skipTo0:    inc     rcx
            cmp     byte ptr [rcx], 0
            jne     skipTo0
            inc     rcx
            mov     returnAdrs, rcx

            leave
 movdqu  xmm0, xmmword ptr [rsp + 00] 
            movdqu  xmm1, xmmword ptr [rsp + 16] 
            movdqu  xmm2, xmmword ptr [rsp + 32] 
            movdqu  xmm3, xmmword ptr [rsp + 48] 
            movdqu  xmm4, xmmword ptr [rsp + 64] 
            movdqu  xmm5, xmmword ptr [rsp + 80] 
            movdqu  xmm6, xmmword ptr [rsp + 96] 
            movdqu  xmm7, xmmword ptr [rsp + 112] 
            add     rsp, 128
            pop     r11
            pop     r10
            pop     r9
            pop     r8
            pop     rdx
            pop     rcx
            pop     rbx
            pop     rax
            ret
print_SSE   endp            
            end
```

清单 15-2：出现在汇编单元中的`print`函数

为了完成迄今为止使用的所有常见*aoalib*函数，这里是清单 15-3。

```
; getTitle.asm - The getTitle function converted to
;                an assembly unit.

; Return program title to C++ program:

            include aoalib.inc

            .code
getTitle    proc
            lea     rax, ttlStr
            ret
getTitle    endp
            end
```

清单 15-3：作为汇编单元的`getTitle`函数

清单 15-4 是一个使用清单 15-2 和 15-3 中汇编单元的程序。

```
; Listing 15-4

; Demonstration of linking.

            include aoalib.inc

            .data
ttlStr      byte    "Listing 15-4", 0

***************************************************************

; Here is the "asmMain" function.

            .code
            public  asmMain
asmMain     proc
            push    rbx
            push    rsi
            push    rdi
            push    rbp
            mov     rbp, rsp
            sub     rsp, 56         ; Shadow storage

            call    print
            byte    "Assembly units linked", nl, 0

            leave
            pop     rdi
            pop     rsi
            pop     rbx
            ret     ; Returns to caller
asmMain     endp
            end
```

清单 15-4：一个使用`print`和`getTitle`汇编模块的主程序

那么如何构建和运行这个程序呢？不幸的是，本书到目前为止使用的*build.bat*批处理文件无法完成这个任务。这里有一个命令，它会将所有单元汇集并将它们链接在一起：

```
ml64 /c print.asm getTitle.asm listing15-4.asm
cl /EHa c.cpp print.obj getTitle.obj listing15-4.obj
```

这些命令将正确地编译所有源文件并将它们的目标代码链接在一起，生成可执行文件*c.exe*。

不幸的是，前面的命令失去了分离编译的一个主要优势。当你执行`ml64 /c print.asm getTitle.asm listing15-4.asm`命令时，它会编译所有的汇编源文件。记住，分离编译的一个主要原因是为了减少大项目的编译时间。虽然前面的命令有效，但它们并没有实现这个目标。

要分别编译这两个模块，你必须分别对它们运行 MASM。要分别编译这三个源文件，可以将`ml64`调用拆分成三个单独的命令：

```
ml64 /c print.asm
ml64 /c getTitle.asm
ml64 /c listing15-4.asm
cl /EHa c.cpp print.obj getTitle.obj listing15-4.obj
```

当然，这个顺序仍然会编译所有三个汇编源文件。然而，在第一次执行这些命令之后，你已经构建了*print.obj*和*getTitle.obj*文件。从此以后，只要你不更改*print.asm*或*getTitle.asm*源文件（并且不删除*print.obj*或*getTitle.obj*文件），你就可以通过使用这些命令来构建和运行 Listing 15-4 中的程序：

```
ml64 /c listing15-4.asm
cl /EHa c.cpp print.obj getTitle.obj listing15-4.obj
```

现在，你节省了编译*print.asm*和*getTitle.asm*文件所需的时间。

## 15.7 Makefile 简介

本书中使用的*build.bat*文件比逐个输入构建命令要方便得多。不幸的是，*build.bat*支持的构建机制实际上只适用于少数固定的源文件。虽然你可以轻松构造一个批处理文件来编译一个大型汇编项目中的所有文件，但运行该批处理文件时会重新汇编项目中的每一个源文件。虽然你可以使用复杂的命令行功能来避免一些这种情况，但有一种更简单的方法：makefile。

*makefile*是一种特殊语言的脚本（最早在 Unix 的早期版本中设计），它指定了如何基于某些条件执行一系列命令，这些命令由 make 程序执行。如果你已经安装了 MSVC 和 MASM 作为 Visual Studio 的一部分，那么你可能也已经安装了（作为同一过程的一部分）Microsoft 版本的 make：`nmake.exe`。^(4) 要使用`nmake.exe`，你可以在 Windows 命令行中按如下方式执行：

```
nmake `optional_arguments`
```

如果你在命令行中单独执行`nmake`（没有任何参数），`nmake.exe`将搜索名为*makefile*的文件，并尝试处理该文件中的命令。对于许多项目来说，这是非常方便的。你将把所有项目的源文件放在一个目录中（或该目录下的子目录中），并将一个名为*makefile*的单一 makefile 放在该目录中。通过切换到该目录并执行`nmake`（或`make`），你可以轻松构建项目。

如果您想使用不同于 *makefile* 的文件名，必须在文件名前加上 `/f` 选项，如下所示：

```
nmake /f mymake.mak
```

文件名不一定需要具有 *.mak* 扩展名。然而，当使用非 *makefile* 命名的 makefile 时，这是一个常见的约定。

`nmake` 程序确实提供了许多命令行选项，`/help` 将列出它们。请查阅 `nmake` 文档以了解其他命令行选项的描述（其中大多数是高级选项，对于大多数任务来说不必要）。

### 15.7.1 基本 Makefile 语法

makefile 是一个标准的 ASCII 文本文件，包含以下格式的一系列行（或该序列的多个出现）：

```
`target`: `dependencies`
    `commands`
```

`target``:` `dependencies` 行是可选的。`commands` 项是一个包含一个或多个命令行命令的列表，也是可选的。`target` 项，如果存在，必须从它所在的源行的第 1 列开始。`commands` 项必须在前面至少有一个空白字符（空格或制表符）（即，它们不能从源行的第 1 列开始）。考虑以下有效的 makefile：

```
c.exe:
  ml64 /c print.asm
  ml64 /c getTitle.asm
  ml64 /c listing15-4.asm
  cl /EHa c.cpp print.obj getTitle.obj listing15-4.obj
```

如果这些命令出现在名为 *makefile* 的文件中，并且您执行 `nmake`，那么 `nmake` 将像命令行解释器在批处理文件中出现这些命令时那样执行它们。

`target` 项是某种标识符或文件名。考虑以下 makefile：

```
executable:
  ml64 /c listing15-4.asm
  cl /EHa c.cpp print.obj getTitle.obj listing15-4.obj

library:
  ml64 /c print.asm
  ml64 /c getTitle.asm
```

这将构建命令分为两组：一组由 `executable` 标签指定，另一组由 `library` 标签指定。

如果您没有任何命令行选项运行 `nmake`，`nmake` 只会执行与 makefile 中第一个目标相关的命令。在这个例子中，如果您单独运行 `nmake`，它将汇编 *listing15-4.asm*、*print.asm* 和 *getTitle.asm*；编译 *c.cpp*；并尝试将生成的 *c.obj* 与 *print.obj*、*getTitle.obj* 和 *listing15-4.obj* 链接。这应该能够成功生成 *c.exe* 可执行文件。

要处理库目标之后的命令，请将目标名称作为 `nmake` 命令行参数指定：

```
nmake library
```

该 `nmake` 命令编译 *print.asm* 和 *getTitle.asm*。因此，如果您执行该命令一次（且以后不再更改 *print.asm* 或 *getTitle.asm*），只需执行 `nmake` 命令本身即可生成可执行文件（或者如果您希望明确说明正在构建可执行文件，可以使用 `nmake executable`）。

### 15.7.2 Make 依赖关系

尽管在命令行中指定要构建的目标非常有用，但随着项目的增大（包含许多源文件和库模块），始终跟踪哪些源文件需要重新编译可能会变得繁琐且容易出错；如果不小心，您可能会忘记在对某个不常用的库模块进行修改后重新编译它，并且困惑为何应用程序仍然失败。make 依赖选项可以让您自动化构建过程，帮助避免这些问题。

在 makefile 中，一个或多个（以空格分隔的）依赖项可以跟随一个目标：

```
`target`: `dependency1` `dependency2` `dependency3` ...
```

依赖项可以是目标名称（出现在该 makefile 中的目标）或文件名。如果依赖项是一个目标名称（而不是文件名），`nmake`会执行与该目标相关联的命令。请考虑以下 makefile：

```
executable:
  ml64 /c listing15-4.asm
  cl /EHa c.cpp print.obj getTitle.obj listing15-4.obj

library:
  ml64 /c print.asm
  ml64 /c getTitle.asm

all: library executable
```

`all`目标依赖于`library`和`executable`目标，因此它会执行与这些目标相关联的命令（并按`library`、`executable`的顺序执行，这一点很重要，因为`library`目标文件必须在相关的目标模块链接到可执行程序之前构建）。`all`标识符是 makefile 中常见的目标，实际上，它通常是 makefile 中出现的第一个或第二个目标。

如果`target``:` `dependencies`行变得过长，导致无法读取（`nmake`并不特别关心行长问题），你可以通过在行末放置一个反斜杠字符（`\`）来将这一行拆分为多行。`nmake`程序会将以反斜杠结尾的源行与 makefile 中的下一行合并。

目标名称和依赖项也可以是文件名。将文件名指定为目标名称通常是为了告诉构建系统如何构建该特定文件。例如，我们可以将当前示例重写如下：

```
executable:
  ml64 /c listing15-4.asm
  cl /EHa c.cpp print.obj getTitle.obj listing15-4.obj

library: print.obj getTitle.obj

print.obj:
  ml64 /c print.asm

getTitle.obj:
  ml64 /c getTitle.asm

all: library executable
```

当依赖项与目标关联且目标为文件名时，你可以将`target``:` `dependencies`语句理解为“`target`依赖于`dependencies`”。在处理 make 命令时，`nmake`会比较指定为目标文件名和依赖文件名的文件的修改日期和时间戳。

如果目标的日期和时间早于*任何*依赖项（或者目标文件不存在），`nmake`会执行目标后的命令。如果目标文件的修改日期和时间比*所有*依赖文件的日期和时间都要晚（更新），`nmake`则不会执行命令。如果目标后面的某个依赖项本身是其他地方的目标，`nmake`会首先执行该命令（以查看它是否修改目标对象，改变其修改日期和时间，可能会导致`nmake`执行当前目标的命令）。如果目标或依赖项只是一个标签（而不是文件名），`nmake`会将其修改日期和时间视为比任何文件都要旧。

请考虑对运行中的`makefile`示例做如下修改：

```
c.exe: print.obj getTitle.obj listing15-4.obj
  cl /EHa c.cpp print.obj getTitle.obj listing15-4.obj

listing15-4.obj: listing15-4.asm
  ml64 /c listing15-4.asm

print.obj: print.asm
  ml64 /c print.asm

getTitle.obj: getTitle.asm
  ml64 /c getTitle.asm
```

注意，`all`和`library`目标已被移除（它们被认为是不必要的），而`executable`被更改为*c.exe*（最终的目标可执行文件）。

考虑 *c.exe* 目标。因为 *print.obj*、*getTitle.obj* 和 *listing15-4.obj* 都是目标（也是文件名），`nmake` 会首先执行这些目标。执行这些目标后，`nmake` 会比较 *c.exe* 的修改日期和时间与这三个目标文件的修改日期和时间。如果 *c.exe* 比其中任何一个目标文件都要旧，`nmake` 会执行 *c.exe* 目标行后面的命令（编译 *c.cpp* 并将其与目标文件链接）。如果 *c.exe* 比依赖的目标文件更新，`nmake` 将不会执行该命令。

对于每个依赖的目标文件，`nmake` 会按相同的过程递归执行，依次处理 *print.obj*、*getTitle.obj* 和 *listing15-4.obj* 目标。在处理 *c.exe* 目标时，`nmake` 会依次处理 *print.obj*、*getTitle.obj* 和 *listing15-4.obj* 目标（按这个顺序）。在每一种情况下，`nmake` 会比较 *.obj* 文件的修改日期和时间与对应的 *.asm* 文件。如果 *.obj* 文件比 *.asm* 文件更新，`nmake` 会返回处理 *c.exe* 目标，而不做任何操作；如果 *.obj* 文件比 *.asm* 文件旧（或不存在），`nmake` 会执行相应的 `ml64` 命令生成新的 *.obj* 文件。

如果 *c.exe* 比所有的 *.obj* 文件都更新（且它们都比 *.asm* 文件更新），执行 `nmake` 不会做任何事情（好吧，它会报告 *c.exe* 已经是最新的，但不会处理 makefile 中的任何命令）。如果任何文件是过时的（因为它们已被修改），这个 makefile 只会编译和链接必要的文件，以使 *c.exe* 更新。

到目前为止，makefile 缺少一个重要的依赖关系：所有的 *.asm* 文件都包含了 *aoalib.inc* 文件。对 *aoalib.inc* 的更改可能会导致这些 *.asm* 文件的重新编译。这个依赖关系已经添加到 Listing 15-5 中。这个列表还演示了如何通过在行首使用 `#` 字符来在 makefile 中包含注释。

```
# listing15-5.mak

# makefile for Listing 15-4.

listing15-4.exe:print.obj getTitle.obj listing15-4.obj
    cl /nologo /O2 /Zi /utf-8 /EHa /Felisting15-4.exe c.cpp \
            print.obj getTitle.obj listing15-4.obj

listing15-4.obj: listing15-4.asm aoalib.inc
  ml64 /nologo /c listing15-4.asm

print.obj: print.asm aoalib.inc
  ml64 /nologo /c print.asm

getTitle.obj: getTitle.asm aoalib.inc
  ml64 /nologo /c getTitle.asm
```

列表 15-5: 用于构建 Listing 15-4 的 makefile

这是使用 Listing 15-5 中的 makefile 来构建 Listing 15-4 程序的 `nmake` 命令：

```
C:\>**nmake /f listing15-5.mak**

Microsoft (R) Program Maintenance Utility Version 14.15.26730.0
Copyright (C) Microsoft Corporation.  All rights reserved.

 ml64 /nologo /c print.asm
 Assembling: print.asm
        ml64 /nologo /c getTitle.asm
 Assembling: getTitle.asm
        ml64 /nologo /c listing15-4.asm
 Assembling: listing15-4.asm
        cl /nologo /O2 /Zi /utf-8 /EHa /Felisting15-4.exe c.cpp  print.obj getTitle.obj listing15-4.obj
c.cpp

C:\>**listing15-4**
Calling Listing 15-4:
Assembly units linked
Listing 15-4 terminated
```

### 15.7.3 Make Clean 和 Touch

在大多数专业制作的 makefile 中，你会找到一个常见的目标 `clean`。`clean` 目标会删除一组适当的文件，以便下次执行 makefile 时强制重新构建整个系统。这个命令通常会删除与项目相关的所有 *.obj* 和 *.exe* 文件。Listing 15-6 提供了 Listing 15-5 中的 `clean` 目标。

```
# listing15-6.mak

# makefile for Listing 15-4.

listing15-4.exe:print.obj getTitle.obj listing15-4.obj
    cl /nologo /O2 /Zi /utf-8 /EHa /Felisting15-4.exe c.cpp \
            print.obj getTitle.obj listing15-4.obj

listing15-4.obj: listing15-4.asm aoalib.inc
    ml64 /nologo /c listing15-4.asm

print.obj: print.asm aoalib.inc
    ml64 /nologo /c print.asm

getTitle.obj: getTitle.asm aoalib.inc
    ml64 /nologo /c getTitle.asm

clean:
    del getTitle.obj
    del print.obj
    del listing15-4.obj
    del c.obj
    del listing15-4.ilk
    del listing15-4.pdb
    del vc140.pdb
    del listing15-4.exe

# Alternative clean (if you like living dangerously):

# clean:
#   del *.obj
#   del *.ilk
#   del *.pdb
#   del *.exe
```

列表 15-6: 一个 `clean` 目标示例

这是一个示例的清理和重建操作：

```
C:\>**nmake /f listing15-6.mak clean**

Microsoft (R) Program Maintenance Utility Version 14.15.26730.0
Copyright (C) Microsoft Corporation.  All rights reserved.

        del getTitle.obj
        del print.obj
        del listing15-4.obj
        del c.obj
        del listing15-4.ilk
        del listing15-4.pdb
        del listing15-4.exe

C:\>**nmake /f listing15-6.mak**

Microsoft (R) Program Maintenance Utility Version 14.15.26730.0
Copyright (C) Microsoft Corporation.  All rights reserved.

        ml64 /nologo /c print.asm
 Assembling: print.asm
        ml64 /nologo /c getTitle.asm
 Assembling: getTitle.asm
        ml64 /nologo /c listing15-4.asm
 Assembling: listing15-4.asm
        cl /nologo /O2 /Zi /utf-8 /EHa /Felisting15-4.exe c.cpp
           print.obj getTitle.obj listing15-4.obj
c.cpp
```

如果你想强制重新编译一个文件（而不需要手动编辑和修改它），一个 Unix 工具会派上用场：`touch`。`touch`程序接受一个文件名作为参数，然后更新文件的修改日期和时间（而不对文件本身进行修改）。例如，在使用 Listing 15-6 中的 makefile 构建 Listing 15-4 之后，如果你执行命令

```
touch listing15-4.asm
```

然后再次执行 Listing 15-6 中的 makefile，它会重新组装 Listing 15-4 中的代码，重新编译*c.cpp*，并生成一个新的可执行文件。

不幸的是，虽然`touch`是一个标准的 Unix 应用程序，并且在每个 Unix 和 Linux 发行版中都会附带，但它不是 Windows 的标准应用程序^(5)。幸运的是，你可以很容易地在互联网上找到适用于 Windows 的`touch`版本。这也是一个相对简单的程序，可以自行编写。

## 15.8 Microsoft 链接器和库代码

许多常见的项目会重用开发人员早期创建的代码（或者使用来自开发者组织外部的代码）。这些代码库相对来说是*静态的*：在使用这些库代码的项目开发过程中，它们很少发生变化。特别地，通常不会将库的构建过程纳入特定项目的 makefile 中。一个特定项目可能会在 makefile 中将库文件列为依赖项，但假设库文件是在其他地方构建的，并作为整体提供给项目。除此之外，库和一组目标代码文件之间还存在一个主要的区别：打包。

在处理大量单独的目标文件时，尤其是当你在处理真正的库目标文件集时，会变得很麻烦。一个库可能包含几十、几百甚至上千个目标文件。列出所有这些目标文件（甚至仅仅是项目使用的文件）是一项繁重的工作，并且可能导致一致性错误。解决这个问题的常见方法是将各种目标文件组合成一个单独的包（文件），称为*库文件*。在 Windows 下，库文件通常具有*.lib*后缀。

对于许多项目，你会获得一个库（*.lib*）文件，它将特定的库模块打包在一起。你在构建程序时将这个文件提供给链接器，链接器会自动从库中挑选出它需要的目标模块。这是一个重要的要点：在构建可执行文件时包含一个库，并不会自动将该库中的所有代码插入到可执行文件中。链接器足够智能，能够只提取它需要的目标文件，并忽略它不使用的目标文件（记住，库只是一个包含大量目标文件的包）。

那么问题是，“如何创建一个库文件？”简短的回答是，“通过使用 Microsoft Library Manager 程序（*lib.exe*）。”`lib`程序的基本语法是

```
lib /out:`libname.lib` `list_of_.obj_files`
```

其中`libname.lib`是你要生成的库文件的名称，`list_of_.obj_files`是你要合并到库中的（以空格分隔的）目标文件列表。例如，如果你想将*print.obj*和*getTitle.obj*文件合并成一个库模块（*aoalib.lib*），可以使用以下命令：

```
lib /out:aoalib.lib getTitle.obj print.obj
```

一旦你有了一个库模块，你可以像指定目标文件一样，在链接器（或`ml64`或`cl`）命令行中指定它。例如，要将*aoalib.lib*模块与 Listing 15-4 中的程序链接，你可以使用以下命令：

```
cl /EHa /Felisting15-4.exe c.cpp listing15-4.obj aoalib.lib
```

`lib`程序支持多种命令行选项。你可以通过使用以下命令获取这些选项的列表：

```
lib /?
```

请参阅在线的 Microsoft 文档，了解各种命令的描述。最有用的选项之一可能是

```
lib /list `lib_filename.lib`
```

其中`lib_filename.lib`表示库文件名。这将打印该库模块中包含的目标文件列表。例如，`lib /list aoalib.lib`会输出如下内容：

```
C:\>**lib /list aoalib.lib**
Microsoft (R) Library Manager Version 14.15.26730.0
Copyright (C) Microsoft Corporation.  All rights reserved.

getTitle.obj
print.obj
```

MASM 提供了一条特殊指令`includelib`，允许你指定要包含的库。此指令的语法为

```
includelib `lib_filename.lib`
```

其中`lib_filename.lib`是你要包含的库文件的名称。此指令在 MASM 生成的目标文件中嵌入一条命令，将该库文件名传递给链接器。链接器将在处理包含`includelib`指令的目标模块时自动加载库文件。

这一操作与手动将库文件名指定给链接器（通过命令行）是相同的。你是否偏好将`includelib`指令放在 MASM 源文件中，或是在链接器（或`ml64`/`cl`）命令行中包含库名称，取决于你自己。根据我的经验，大多数汇编语言程序员（尤其是在编写独立的汇编语言程序时）更喜欢使用`includelib`指令。

## 15.9 目标文件和库对程序大小的影响

程序中的基本链接单元是目标文件。在将目标文件组合成可执行文件时，Microsoft 链接器将把单个目标文件中的所有数据合并到最终的可执行文件中。即使主程序没有直接或间接调用该目标模块中的所有函数，或没有使用该目标文件中的所有数据，这也是成立的。所以，如果你将 100 个例程放入一个单独的汇编语言源文件并将它们编译成一个目标模块，链接器会将这 100 个例程的代码全部包含到你的最终可执行文件中，即使你只使用其中的一个例程。

如果你想避免这种情况，你应该将这 100 个例程拆分成 100 个独立的目标模块，并将这 100 个目标文件组合成一个单一的库。当微软的链接器处理这个库文件时，它将选择包含程序使用的函数的单个目标文件，并仅将该目标文件合并到最终的可执行文件中。*通常*，这种方式比将一个包含 100 个函数的目标文件链接进来更高效。

上一句话中的关键词是*通常*。事实上，将多个函数合并成一个目标文件是有一些合理原因的。首先，考虑当链接器将目标文件合并到可执行文件中时会发生什么。为了确保正确的对齐，每当链接器从目标文件中获取一个部分或段（例如，`.code`段）时，它会添加足够的填充，以确保该段中的数据对齐到指定的对齐边界。大多数段的默认对齐为 16 字节，因此链接器会将它链接的每个目标文件中的段对齐到 16 字节边界。通常，这并不算太糟糕，特别是当你的过程较大时。然而，假设你创建的这 100 个过程都是非常短小的（每个只有几字节）。那么你就会浪费很多空间。

当然，在现代计算机上，几百字节的浪费空间并不会造成太大影响。然而，结合这些过程中的几个到一个单独的目标模块（即使你并不调用所有的）来填补一些浪费的空间可能更为实际。不过，不要过度操作；一旦你超出了对齐边界，不管是因为填充浪费了空间，还是因为你包含了从未被调用的代码，最终你还是在浪费空间。

## 15.10 更多信息

虽然这是一本较旧的书，涵盖的是 MASM 6 版本，*《Waite Group 的微软宏汇编语言宝典》* 由纳巴乔提·巴尔卡提和本书作者编写（Sams，1992 年），它详细讨论了 MASM 的外部指令（`extern`，`externdef`，和`public`）和包含文件。

你也可以在网上找到 MASM 6 手册（最后发布的版本）。

欲了解更多关于 makefile 的信息，请参考以下资源：

+   维基百科：[`en.wikipedia.org/wiki/Make_(software)`](https://en.wikipedia.org/wiki/Make_(software))

+   *使用 GNU Make 管理项目*，第三版，作者：罗伯特·梅克伦堡（O'Reilly Media，2004 年）

+   *《GNU Make 书》* 由约翰·格雷厄姆-卡明（No Starch Press，2015 年）

## 15.11 测试自己

1.  你会使用什么语句来防止递归包含文件？

1.  什么是汇编单元？

1.  你会使用什么指令来告诉 MASM 一个符号是全局的，并且在当前源文件外可见？

1.  你会使用什么指令来告诉 MASM 使用另一个目标模块中的全局符号？

1.  哪个指令可以防止在汇编源文件中定义外部符号时出现重复符号错误？

1.  你会使用什么外部数据类型声明来访问外部常量符号？

1.  你会使用什么外部数据类型声明来访问外部过程？

1.  微软的 make 程序叫什么名字？

1.  基本的 makefile 语法是什么？

1.  什么是 makefile 依赖的文件？

1.  makefile 中的 `clean` 命令通常做什么？

1.  什么是库文件？
