

## 第十六章：16 独立汇编语言程序



![](img/opener.jpg)

到目前为止，本书依赖于一个 C/C++ 主程序来调用用汇编语言编写的示例代码。虽然这可能是现实世界中使用汇编语言的最大案例，但也可以编写独立的汇编程序（不需要 C/C++ 主程序）。在本章中，你将学习如何编写这样的独立程序。

就本书而言，*独立汇编语言程序*指的是汇编语言代码包含一个实际的 *main* 程序（而不是 asmMain，它只是 C++ 程序调用的一个函数）。这样的程序不进行任何 C/C++ stdlib 调用；唯一的外部调用是操作系统 API 函数调用。

> 注意

*一些读者可能会将“独立程序”一词理解为汇编语言程序不进行任何外部函数调用，甚至不调用操作系统，而是在应用程序内部以硬件级别处理所有 I/O。这个定义适用于嵌入式系统，但不是本书中的定义。*

从技术上讲，你的汇编代码总是会被 C/C++ 程序调用。这是因为操作系统本身是用 C/C++ 编写的，只有极少量的汇编代码。当操作系统将控制权转交给你的汇编代码时，这与 C/C++ 主程序调用你的汇编代码没有太大区别。然而，“纯”汇编应用程序有一些明显的优势：你不需要携带 C/C++ 库代码和应用程序运行时系统，因此你的程序可以更小，而且不会与 C/C++ 公共名称发生外部命名冲突。

本章介绍了适用于 macOS 和 Linux（包括 Pi OS）的操作系统系统调用。它首先解释了如何在代码中保持可移植性，因为系统调用在不同操作系统之间并不具有可移植性。接下来介绍了这两个操作系统的系统调用概念。在讨论了用于调用操作系统 API 函数的 svc（超级调用）指令之后，提供了两个示例：一个独立的“Hello, world!”应用程序和一个文件 I/O 应用程序。最后指出，macOS 不鼓励直接进行系统调用，而是希望你通过 C 库函数调用与操作系统进行交互。

### 16.1 系统调用的可移植性问题

尽管到目前为止，本书中的大多数示例程序在 macOS 和 Linux 之间是可移植的，但系统 API 调用因操作系统而异。前几章中的代码通过调用 C/C++ stdlib 函数来处理操作系统的低级细节，忽略了这个问题，但本章中的示例代码则进行了操作系统特定的调用。因此，可移植性不会自动发生。你有四种方法来处理这个问题：

+   忽略可移植性，仅为 macOS 或 Linux 编写给定的示例程序。通常，在编写特定于操作系统的代码时，我会采用这种方法。

+   编写两个（不可移植的）版本的相同程序：一个适用于 Linux，一个适用于 macOS。

+   编写一个单一的程序，使用条件汇编根据需要包含操作系统特定的代码。

+   创建两个封装文件，一个包含 macOS 版本的操作系统调用，另一个包含 Linux 版本的操作系统调用，并将适当的封装文件与主（便携式）代码一起包含。

使用哪种机制取决于你的应用。如果你不打算编写能够跨操作系统工作的便携式汇编代码（编写汇编应用时最常见的情况），你将采用第一种方法，仅为你所针对的操作系统编写代码。

如果你希望你的汇编应用在 macOS 和 Linux 上运行，你的选择将取决于应用的规模。如果应用相对较小，编写两个操作系统特定的变种并不难（尽管维护可能是一个问题，因为你需要维护两个独立版本的应用）。如果应用程序较大，或者你预计会频繁升级和维护它，第三种或第四种方法可能更好。使用条件汇编来处理操作系统特定问题的单一应用程序通常比两个独立的应用程序更容易维护和扩展，而使用封装代码使得维护每个特定操作系统的代码更加容易。

还有第五种方法：将所有操作系统相关的代码用 C/C++编写，并调用处理与操作系统无关的功能的汇编函数。这就是本书中所有示例程序的编写方式。

不言而喻，本章中的代码没有使用*build*脚本来编译/汇编示例应用程序。*build*脚本假设使用*c.cpp*主程序（而本章的重点就是停止使用该代码）。因此，本章中的每个示例程序都包含一个 makefile，用于构建代码。

### 16.2 独立代码与系统调用

本书中的第一个示例程序是第 5 页上的清单 1-1，它是一个独立程序。为了讨论的需要，下面是经过几处修改后的清单 16-1。

```
// Listing16-1.S
//
// Comments consist of all text from a //
// sequence to the end of the line.
// The .text directive tells MASM that the
// statements following this directive go in
// the section of memory reserved for machine
// instructions (code).

       .text

// Here is the main function.
// (This example assumes that the
// assembly language program is a
// stand-alone program with its own
// main function.)
//
// Under macOS, the main program
// must have the name _main
// beginning with an underscore.
// Linux systems generally don't
// require the underscore.
//
// The .global _main statement
// makes the _main procedure's name
// visible outside this source file
// (needed by the linker to produce
// an executable).

 ❶ .global _main  // This is the macOS entry point.
     ❷ .global main   // This is the Linux entry point name.

// The .align 2 statement tells the
// assembler to align the following code
// on a 4-byte boundary (required by the
// ARM CPU). The 2 operand specifies
// 2 raised to this power (2), which
// is 4.

       .align 2

// Here's the actual main program. It
// consists of a single ret (return)
// instruction that simply returns
// control to the operating system.

❶ _main:
❷ main:
    ❸ ret
```

与清单 1-1 中的代码相比，我对这段代码进行了两处修改。在❶和❷的位置，我引入了一个新符号，main（和 _main）。这是因为 Linux 要求主程序命名为 main，而 macOS 要求命名为 _main。如果你尝试在 Linux 上编译清单 1-1，你会得到类似“未定义的`main`引用”这样的错误消息。我干脆在源文件中同时包含这两个符号，而不是处理条件汇编（或编写两个独立的清单 16-1 版本）。Linux 大多忽略 _main 符号，macOS 忽略 main 符号；因此，该程序能够在任一操作系统上顺利编译。

清单 16-1 由一条指令组成：ret ❸。进入时，LR 寄存器包含一个返回地址，将控制转回操作系统。因此，这个程序（如果你真的执行它）会立即返回到操作系统。

尽管通过 ret 指令返回操作系统有效（特别是在使用 GCC 构建此代码时），但这不是返回 Linux 或 macOS 的标准方式。相反，应用程序应该调用 exit() API 函数。要调用系统 API 函数，程序必须将函数号加载到寄存器中，将适当的参数加载到参数寄存器（X0 至 X7）中，然后执行超级用户（操作系统）调用指令 svc #OSint，其中 OSint 对于 Linux 是 0，对于 macOS 是 0x80。

> 注意

*实际上，macOS 似乎忽略了 svc 指令后面的立即数常量。许多在线示例使用值 0 作为 svc 操作数（个人实验也证明它有效）。然而，macOS 源代码似乎使用 0x80 作为常量，因此我建议在 macOS 下使用该值。*

在 Linux 中，您将系统调用号加载到 X8 寄存器中，而在 macOS 中，您将其加载到 X16 寄存器中。我在 *aoaa.inc* 中添加了以下语句来处理此问题：

```
#if isMacOS

    // Under macOS, the system call number
    // goes into X16:

    #define svcReg x16
    #define OSint  0x80

#else

    // Under Linux, the system call number
    // is passed in X8:

    #define svcReg x8
    #define OSint  0

#endif
```

在 Linux 和 macOS 下，exit 函数期望在 X0 寄存器中接收一个整数参数，该参数保存程序的返回码（如果程序运行时没有发生错误，通常为 0）。剩下的唯一问题是，“exit() 的系统调用号是多少？”在 Linux 下，代码是 93，而在 macOS 下是 1（我将在第 16.3 节“svc 接口与操作系统可移植性”中讨论我是如何确定这些魔法数字的，详情请见下一页）。列表 16-2 提供了一个非常简单的汇编应用程序，它立即返回到操作系统，您可以为 macOS 或 Linux 编译它。

```
// Listing16-2.S
//
// Simple shell program that calls exit()

      ❶ #include    "aoaa.inc"

        // Specify OS-dependent return code:

      ❷ #ifdef      isMacOS
        #define     exitCode 1
        #else
        #define     exitCode 93
        #endif

        .text
        .global     _main
        .global     main
        .align      2

_main:
main:
      ❸ mov         x0, #0  // Return success.
      ❹ mov         svcReg, exitCode
      ❺ svc         #OSint
```

列表 16-2 包含 *aoaa.inc* ❶ 以便在命令行中未定义操作系统符号（Linux 或 Darwin）时生成错误（*aoaa.inc* 将其转换为 isLinux 或 isMacOS），并获取 OSint 和 svcReg 常量。

该程序使用条件汇编来生成适用于 macOS 或 Linux 的不同代码，将常量 exitCode 设置为操作系统的退出函数号 ❷。该函数将 0 ❸ 加载到 X0 寄存器中，表示成功并返回。然后，它将 exitCode 函数号加载到操作系统的函数号参数寄存器 ❹（在 Linux 中为 X8，在 macOS 中为 X16，如 *aoaa.inc* 中定义，参见前面的示例）。最后，代码发出超级用户调用指令，调用操作系统 ❺。由于此调用的性质，svc 指令永远不会将控制返回给程序，因此无需 ret 指令。

以下是构建列表 16-2 中程序的 makefile：

```
# Listing16-2.mak
#
# makefile to build the Listing16-2 file

unamestr=`uname`

Listing16-2:
    g++ -D$(unamestr) Listing16-2.S -o Listing16-2

clean:
    rm -f Listing16-2.o
    rm -f Listing16-2
```

要构建并运行此程序，请在 shell 程序中输入以下命令：

```
% make -f Listing16-2.mak
g++ -D`uname` Listing16-2.S -o Listing16-2
% ./Listing16-2
```

程序按预期返回，而没有产生任何输出。

### 16.3 svc 接口与操作系统可移植性

macOS 和 Linux 都使用主管调用指令（svc）向操作系统发出 API 调用。然而，两个操作系统之间的实际调用顺序差异很大。本节将阐明它们在支持的功能（API）方面的区别，特别是在调用号、参数和错误处理方面。

尽管这两个操作系统都是基于 Unix 的（并共享许多符合 POSIX 标准的功能），每个操作系统都有自己的一套特定的操作系统函数，这些函数在另一个系统中可能没有对应的功能。即使是常见的（例如 POSIX）函数，也可能期望不同的参数并产生不同的返回结果，这意味着在编写能够在这两个操作系统之间移植的汇编代码时，必须特别小心。这是一个很好的例子，说明使用包装器来本地化操作系统系统调用有助于提高代码的可移植性和可维护性。

#### 16.3.1 调用号

如前所述，函数调用号在不同操作系统之间有所不同，传递调用号的位置也不同（Linux 使用 X8，macOS 使用 X16）。通过使用#define（或.reqdirective）来克服寄存器位置问题相对容易。然而，函数调用号的值完全依赖于操作系统。

*sys*/*syscall.h*文件是一个头文件，包含了所有系统 API 调用号的定义。（即使它是一个 C 头文件，你也可以在汇编语言源文件中包含它。）当你安装 C 编译器（如 GCC 或 Clang）时，这个文件通常会安装到你的系统中，并通常位于编译器默认的包含路径中。有关更多详细信息，请参见 GCC 或 Xcode 文档。

虽然#包括<sys/syscall.h>在 Linux 和 macOS 上都能工作，但实际的定义可能出现在编译器目录树中的其他文件中，*sys*/*syscall.h*中会有适当的#include 指令来指向实际文件。

下面是 macOS 机器上*sys/syscall.h*文件中的几行内容：

```
#ifdef __APPLE_API_PRIVATE
#define SYS_syscall        0
#define SYS_exit           1
#define SYS_fork           2
#define SYS_read           3
#define SYS_write          4
#define SYS_open           5
#define SYS_close          6
#define SYS_wait4          7
   .
   .
   .
```

该代码中的#ifdef 语句是一个警告，Apple 认为 svc API 接口是未记录的并且是私有的，正如下一页“在 macOS 下使用 svc”框中所讨论的那样。

在我的 macOS 系统上，我使用 Unix 的 find 命令定位*sys*/*syscall.h*，它深埋在 Xcode 目录路径*/Library/Developer/CommandLineTools/SDK/ ...* 中，但你的情况可能不同。

在 Debian Linux 下，#include <sys/syscall.h>包括了*/usr/include/asm-generic/unistd.h*（如果该文件不在此位置，可以再次使用 Unix 的 find 命令）。以下是该文件中的几行内容，按与 macOS *syscall.h*文件中语句的顺序排列：

```
#define __NR_exit 93
__SYSCALL(__NR_exit, sys_exit)
#define __NR_read 63
__SYSCALL(__NR_read, sys_read)
#define __NR_write 64
__SYSCALL(__NR_write, sys_write)
#define __NR_openat 56
__SYSCALL(__NR_openat, sys_openat)
#define __NR_close 57
__SYSCALL(__NR_close, sys_close)
   .
   .
   .
```

如你所见，两个文件中的函数名、常量名和函数调用号并不一致。例如，Linux 通常更倾向于使用 openat() 而不是 open() 函数。幸运的是，macOS 也提供了 openat()，因此在两个操作系统上使用相同的函数是可能的。然而，macOS 和 Linux 对相同函数的符号名称却有很大不同，这意味着包含 *sys/syscall.h* 不是一个可移植的解决方案。你仍然需要提供自己的本地名称，将其映射到对应的 Linux 和 macOS 名称（建议：使用两个 syscall 封装 *.S* 文件，一个用于 Mac，一个用于 Linux，来解决这些问题）。

不幸的是，*sys/syscall.h* 头文件没有提供各个函数的参数列表。你可以在 *[`<wbr>arm<wbr>.syscall<wbr>.sh`](https://arm.syscall.sh)* 找到 Linux 的参数信息。例如，考虑 exit() 函数的条目：

```
name  reference   x8    x0               x1 x2 x3 x4 x5
exit  man/ cs/    5D    int error_code   -- -- -- -- --
```

这一行告诉你，X8 必须包含 0x5D（93），而 X0 必须包含退出代码（error_code）。Linux 系统调用最多有六个参数（X0 到 X5），但 exit() 只使用其中一个（除了函数调用号在 X8 中）。

在 macOS 上，你必须使用 macOS 的调用号（exit 的调用号为 1），并将该调用号加载到 X16 中。参数通常对于等效的 macOS 函数相同，但当然受到 Linux 与 macOS ABI 差异的影响（请参阅第 16.8 节，“更多信息”，在 930 页 中查看系统调用的集合）。

#### 16.3.2 API 参数

一般来说，所有系统调用都有明确定义的名称和参数列表，你可以通过搜索函数名称或使用命令行的 man 命令在线查找。例如，openat() 调用具有以下参数（来自 Linux 手册页）：

```
int openat(int dfd, const char *pathname, int flags, mode_t mode);
```

openat 函数的代码（在 Linux 中为 57，在 macOS 中为 463）放在 X8（Linux）或 X16（macOS）寄存器中，目录描述符放在 X0 中，指向文件名（路径名）的指针放在 X1 中，标志参数传入 X2（一个可选的模式参数可以传入 X3）。你需要根据 ARM ABI 选择参数寄存器。

为了创建在 macOS 和 Linux 之间可移植的代码，你可以在源文件的开头使用以下条件汇编，根据操作系统选择常量：

```
 #include "aoaa.inc"
    #include <sys/syscall.h>

#if isMacOS

    #define  sys_openat SYS_openat
    #define  sys_read SYS_read
    #define  sys_close SYS_close

#elif isLinux

    #define  sys_openat __NR_openat
    #define  sys_read __NR_read
    #define  sys_close __NR_close

#endif
```

从此之后，你可以在任一操作系统上使用 sys_* 符号。当然，如果你不需要在两个操作系统之间的可移植性，你可以简单地包含 *sys/syscall.h* 并根据你的操作系统选择适当使用 SYS_* 或 _NR_* 符号。

#### 16.3.3 API 错误处理

macOS 和 Linux API 调用之间的另一个重大区别在于它们返回错误指示的方式。对于 macOS API 调用，错误状态通过进位标志返回（C = 1 表示错误，C = 0 表示没有错误）。如果进位标志被设置，macOS 会在 X0 寄存器中返回一个错误代码。而 Linux 则是在 X0 中返回–1 来表示错误；然后你必须从 errno 变量中获取实际的错误代码（例如，参见第 358 页中的第 7-2 号清单）。

在可移植代码中处理错误返回值可能会存在问题。一种解决方案是使用一组包装函数，以操作系统特定的方式处理每个操作系统中的错误。我选择创建一个小宏，将错误返回状态转换为 macOS 和 Linux 下通用的值：

```
 .macro  checkError

            #if     isMacOS

            // If macOS, convert the error code to be
            // compatible with Linux (carry set is
            // error flag and X0 is error code):

          ❶ bcc     0f
          ❷ neg     x0, x0

            #elif   isLinux

            // If Linux, fetch the errno error code
            // (if return value is -1), negate it,
            // and return that as the error code:

          ❸ cmp     x0, #-1
            bne     0f
          ❹ getErrno
            neg     x0, x0

            #endif
0:
            .endm
```

在 macOS 下，如果进位标志未设置 ❶，则此宏不做任何操作（没有错误）。如果发生错误（进位标志设置），宏会取反 X0 中的值 ❷（目前是一个正的错误代码）。

在 Linux 中，错误通过在 X0 中返回–1 来表示，在这种情况下，代码必须从 errno 变量中检索实际的错误代码。如果 API 函数返回–1 ❸，代码将获取 errno 的值 ❹（这是一个正数），并对其取反。

这个宏假设 API 函数在没有错误时会在 X0 中返回一个非负值，因此如果发生错误，它会返回实际错误代码的相反值。这将提供一组一致的符号值，你可以在任何操作系统下进行测试。

尽管 checkError 宏生成了一组可移植的错误代码，但不要假设这两个操作系统在任何给定情况下会产生完全相同的错误代码。在相同的情况下，它们更可能产生略有不同的错误代码。至少，你应该能够处理任何 macOS 或 Linux 手册页中列出的给定 API 函数的错误返回代码（这再次证明了使用包装函数来处理可移植代码中的错误代码的必要性）。

你可以从*errno.h*文件（或它可能包含的其他文件）中提取适当的定义；这将允许你在汇编源代码中引用类似 EPERM 或 EACCES 的 Unix 兼容常量名。别忘了，checkError 宏会取反错误代码，因此你需要与取反后的*errno.h*常量进行比较（例如，-EPERM 或-EACCES）。

### 16.4 独立的“Hello, World!”程序

按惯例，编写独立汇编程序时第一个“真实”的程序是“Hello, world!”在 Linux 和 macOS 下，你可以使用系统的 write()函数将字符串写入标准输出设备，如第 16-3 号清单所示。

```
// Listing16-3.S
//
// A stand-alone "Hello, world!" program

        #include    "aoaa.inc"
      ❶ #include    <sys/syscall.h>

        // Specify OS-dependent return code:

        #if         isMacOS

      ❷ #define     exitCode    SYS_exit
        #define     sys_write   SYS_write

        #else

      ❸ #define     exitCode    __NR_exit
        #define     sys_write   __NR_write

        #endif

        .data
hwStr:  .asciz      "Hello, world!\n"
hwSize  =           .-hwStr

 .text
       .global      _main
       .global      main
       .align       2

_main:
main:

      ❹ mov         x0, #1          // stdout file handle
        lea         x1, hwStr       // String to print
        mov         x2, #hwSize     // Num chars to print
        mov         svcReg, #sys_write
        svc         #OSint          // Call OS to print str.

      ❺ mov         svcReg, #exitCode
        mov         x0, #0
        svc         #OSint          // Quit program.
```

在清单 16-3 中，`#include` 语句加载了操作系统特定的常量名称，用于 API 函数调用的值 ❶。该代码定义了 macOS 的系统调用常量 ❷。由于 `write` 符号已经在 *aoaa.inc* 头文件中定义（即 C 标准库的 `write()` 函数），我使用了 `sys_write` 来避免命名空间污染。同样，代码还定义了 Linux 系统调用常量 ❸。

调用系统 API `write()` 函数会打印 "Hello, world!" ❹。这个调用期望将字符串的指针放入 X1 寄存器，字符串的长度放入 X2 寄存器，以及文件描述符的值放入 X0 寄存器。对于 stdout 设备，文件描述符是 1。最后，我包含了通常的程序终止代码 ❺。

请注意，C 标准库的 `write()` 函数不过是直接调用 Linux `write()` API 函数的外观代码。如果我们愿意与 C 代码链接，我们也可以通过调用 `write()` 来实现相同的功能，但这样做会违背本章的目的。

以下是将构建清单 16-3 中程序的 makefile：

```
# Listing16-3.mak
#
# makefile to build the Listing16-3 file

unamestr=`uname`

Listing16-3:
    g++ -D$(unamestr) Listing16-3.S -o Listing16-3

clean:
    rm -f Listing16-3.o
    rm -f Listing16-3
```

以下是构建和运行程序的命令，以及示例输出：

```
% make -f Listing16-3.mak clean
rm -f Listing16-3.o
rm -f Listing16-3
% make -f Listing16-3.mak
g++ -D`uname` Listing16-3.S -o Listing16-3
% ./Listing16-3
Hello, world!
```

请注意，程序并没有打印“调用清单 16-3”或“清单 16-3 终止”。这些输出是由 *c.cpp* 中的 main() 函数产生的，而这段代码并没有使用它。

### 16.5 一个示例文件 I/O 程序

文件 I/O 在本书中至今一直缺席。尽管使用 C 标准库中的 fopen、fclose 和 fprintf 等函数可以轻松实现读写文件数据，但 Linux 和 macOS 的 API 提供了许多有用的函数（C 标准库就是基于这些函数构建的），可以用于此目的。本节将介绍其中的一些函数：

open    打开（或创建）一个文件，用于读取、写入或追加。

read    从打开的文件中读取数据。

write    将数据写入一个打开的文件。

close    关闭一个打开的文件。

出于本示例的目的，我将实现这些调用为一个名为 *files* 的库，包含三个源模块：

***volatile.S***    一对用于保存和恢复所有易失性寄存器的工具函数。

***stdio.S***    一组 I/O 例程，用于将数据写入 stdout 设备并从 stdin 设备读取数据（控制台 I/O）。

***files.S***    一组用于打开、读取、写入和关闭文件的例程。

我将这些文件放在一个 *files* 子目录中，并提供了一个 *files.mak* makefile，它将汇编这些文件并将它们放入一个 *file.a* 归档文件中。以下是该 makefile：

```
# files.mak
#
# makefile to build the files library

unamestr=`uname`

files.a:files.o stdio.o volatile.o
    ar rcs files.a files.o stdio.o volatile.o
  ❶ cp files.a ..

files.o:files.S files.inc ../aoaa.inc
    g++ -c -D$(unamestr) files.S

stdio.o:stdio.S files.inc ../aoaa.inc
    g++ -c -D$(unamestr) stdio.S

volatile.o:volatile.S files.inc ../aoaa.inc
    g++ -c -D$(unamestr) volatile.S

clean:
    rm -f files.o
    rm -f volatile.o
    rm -f stdio.o
    rm -f files.a
```

在这个 makefile 成功构建源文件（并将它们组合成 *file.a* 归档文件）后 ❶，它将 *file.a* 复制到父目录中，应用程序会在那里使用 *files.a*。

在讨论文件库的源文件之前，我将首先介绍 *files.inc* 头文件，因为它包含了库和应用程序源代码都将使用的定义：

```
// files.inc
//
// Header file that holds the files library
// globals and constants

            #include "../aoaa.inc"  // Get isMacOS and isLinux.
❶ #if isMacOS
#define __APPLE_API_PRIVATE
#endif
            #include        <sys/syscall.h>

            #if     isMacOS

❷ sys_Read    =       SYS_read
sys_Write   =       SYS_write
sys_Open    =       SYS_openat
sys_Close   =       SYS_close
AT_FDCWD    =       -2

#define O_CREAT     00000200

            #else

❸ sys_Read    =       __NR_read
sys_Write   =       __NR_write
sys_Open    =       __NR_openat
sys_Close   =       __NR_close
AT_FDCWD    =       -100

#define O_CREAT     00000100

            #endif

// Handles for the stdio files:

❹ stdin       =       0
stdout      =       1
stderr      =       2

// Other useful constants:

cr          =       0xd     // Carriage return (ENTER)
lf          =       0xa     // Line feed/newline char
bs          =       0x8     // Backspace

// Note the following are octal (base 8) constants!
// (Leading 0 indicates octal in Gas.)
//
// These constants were copied from fcntl.h.

❺ #define S_IRWXU  (00700)
#define S_RDWR   (00666)
#define S_IRUSR  (00400)
#define S_IWUSR  (00200)
#define S_IXUSR  (00100)
#define S_IRWXG  (00070)
#define S_IRGRP  (00040)
#define S_IWGRP  (00020)
#define S_IXGRP  (00010)
#define S_IRWXO  (00007)
#define S_IROTH  (00004)
#define S_IWOTH  (00002)
#define S_IXOTH  (00001)
#define S_ISUID  (0004000)
#define S_ISGID  (0002000)
#define S_ISVTX  (0001000)

#define O_RDONLY    00000000
#define O_WRONLY    00000001
#define O_RDWR      00000002
#define O_EXCL      00000200
#define O_NOCTTY    00000400
#define O_TRUNC     00001000
#define O_APPEND    00002000
#define O_NONBLOCK  00004000
#define O_DSYNC     00010000
#define FASYNC      00020000
#define O_DIRECT    00040000
#define O_LARGEFILE 00100000
#define O_DIRECTORY 00200000
#define O_NOFOLLOW  00400000
#define O_NOATIME   01000000
#define O_CLOEXEC   02000000

// Macro to test an error return
// value from an OS API call:

          ❻ .macro  file.checkError

            #if     isMacOS

 // If macOS, convert the error code to be
            // compatible with Linux (carry set is
            // error flag, and X0 is error code):

            bcc     0f
            neg     x0, x0

            #elif   isLinux

            // If Linux, fetch the errno error code
            // (if return value is -1), negate it,
            // and return that as the error code:

            cmp     x0, #-1
            bne     0f
            getErrno
            neg     x0, x0

            #endif
0:
            .endm

          ❼ .extern saveVolatile
            .extern restoreVolatile

            .extern file.write
            .extern file.read
            .extern file.open
            .extern file.openNew
            .extern file.close

            .extern stdout.puts
            .extern stdout.newLn

            .extern stdin.read
            .extern stdin.getc
            .extern stdin.readln
```

如前所述，macOS 的 SYS_* 符号出现在 #ifdef 块内，如果没有定义符号 __APPLE_API_PRIVATE，则会隐藏这些定义。因此，在 macOS 下包含 sys/syscall.h 头文件时，*files.inc* 需要定义符号 __APPLE_API_PRIVATE，这样所有的 SYS_* 标签才会被 CPP 处理 ❶。

*files.inc* 头文件随后定义了各种符号，这些符号的值因操作系统而异（特别是 API 函数调用编号） ❷ ❸。这个条件汇编块还定义了 O_CREAT 符号，它在两个操作系统中是不同的。

接下来，头文件定义了在库源代码和与库链接的应用程序中都将使用的各种常量 ❹。stdin、stdout 和 stderr 常量分别是标准输入设备、标准输出设备和标准错误（输出）设备的 Unix 文件描述符值。库使用 cr、lf 和 bs 作为 ASCII 字符代码常量。

接着，我插入了从 *fcntl.h* ❺ 中提取的几个 #define 语句（这是另一个包含有用 API 常量定义的 C/C++ 头文件；通常你会在与 *syscall.h* 相同的目录中找到它）。这些常量在使用 openat() 函数创建新文件时使用（你需要将这些常量提供给模式参数）。与 *errno.h* 类似，你不能简单地包含 *fcntl.h*，因为 Gas 无法处理其中出现的 C/C++ 语句。

如前所述，库在 svc 指令之后使用 file.checkError 宏 ❻ 来检查错误返回结果。最后，代码包含了所有在 *files.a* 库 ❼ 中出现的函数的外部定义。

#### 16.5.1 volatiles.S 函数

*volatiles.S* 源文件包含了两个保存和恢复所有易失性寄存器的函数 saveVolatile 和 restoreVolatile：

```
// volatiles.S
//
// saveVolatile and restoreVolatile functions used
// to preserve volatile registers

            #include    "../aoaa.inc"
            #include    "files.inc"

            .code
            .align  2

// saveVolatile
//
// A procedure that will save all the volatile
// registers at the location pointed at by FP

            proc    saveVolatile, public
            stp     x0,  x1,  [fp], #16
            stp     x2,  x3,  [fp], #16
            stp     x4,  x5,  [fp], #16
            stp     x6,  x7,  [fp], #16
            stp     x8,  x9,  [fp], #16
            stp     x10, x11, [fp], #16
            stp     x12, x13, [fp], #16
            stp     x14, x15, [fp], #16
            stp     q0,  q1,  [fp], #32
            stp     q2,  q3,  [fp], #32
            stp     q4,  q5,  [fp], #32
            stp     q6,  q7,  [fp], #32
            stp     q8,  q9,  [fp], #32
            stp     q10, q11, [fp], #32
            stp     q12, q13, [fp], #32
            stp     q14, q15, [fp], #32
 ret
            endp    saveVolatile

// restoreVolatile
//
// A procedure that will restore all the volatile
// registers from the location pointed at by FP

            proc    restoreVolatile, public
            ldp     x0,  x1,  [fp], #16
            ldp     x2,  x3,  [fp], #16
            ldp     x4,  x5,  [fp], #16
            ldp     x6,  x7,  [fp], #16
            ldp     x8,  x9,  [fp], #16
            ldp     x10, x11, [fp], #16
            ldp     x12, x13, [fp], #16
            ldp     x14, x15, [fp], #16
            ldp     q0,  q1,  [fp], #32
            ldp     q2,  q3,  [fp], #32
            ldp     q4,  q5,  [fp], #32
            ldp     q6,  q7,  [fp], #32
            ldp     q8,  q9,  [fp], #32
            ldp     q10, q11, [fp], #32
            ldp     q12, q13, [fp], #32
            ldp     q14, q15, [fp], #32
            ret
            endp    restoreVolatile
```

这些函数仅仅是将寄存器存储到 FP 寄存器所持有的地址处的连续位置。调用者有责任在调用 saveVolatile 或 restoreVolatile 之前保存 FP 寄存器，并将其加载为 volatile_save 结构的地址。如你所见，*volatiles.S* 中的代码并不会保存 FP 寄存器中的值。

saveVolatile 和 restoreVolatile 的目的是克服操作系统 API 调用可能修改易失性寄存器集的问题。在汇编语言编程中，除非明确地在寄存器中返回结果，否则保持寄存器的值是良好的编程风格。*volatiles.S* 函数使你能够在调用会破坏易失性寄存器的低级 API 函数时，仍然遵循这一编程风格。

这些函数的一个缺点是，你永远无法知道给定的 API 函数可能修改哪些易变寄存器，因此你必须保存所有的易变寄存器，即使 API 函数只改变其中的少数几个。不幸的是，这会给代码带来低效；读写内存并不特别快速。然而，在汇编语言代码中不必担心易变寄存器的保存，还是值得这点小小的效率损失的。（而且，文件 I/O 通常本身就是一个相对较慢的过程，所以如果你频繁调用文件 I/O 函数，保存和恢复寄存器的开销可能在运行时间中所占的比例非常小。）

*aoaa.inc*头文件包含了以下结构，用来定义 saveVolatile 保存的寄存器布局，并由 restoreVolatile 加载：

```
struct  volatile_save
qword   volatile_save.x0x1
qword   volatile_save.x2x3
qword   volatile_save.x4x5
qword   volatile_save.x6x7
qword   volatile_save.x8x9
qword   volatile_save.x10x11
qword   volatile_save.x12x13
qword   volatile_save.x14x15
qword   volatile_save.v0
qword   volatile_save.v1
qword   volatile_save.v2
qword   volatile_save.v3
qword   volatile_save.v4
qword   volatile_save.v5
qword   volatile_save.v6
qword   volatile_save.v7
qword   volatile_save.v8
qword   volatile_save.v9
qword   volatile_save.v10
qword   volatile_save.v11
qword   volatile_save.v12
qword   volatile_save.v13
qword   volatile_save.v14
qword   volatile_save.v15
ends    volatile_save
```

由于这个结构体相当大，saveVolatile 和 restoreVolatile 不涉及单独的字段。某些成员的偏移量过大，无法在 32 位加载指令的寻址模式偏移字段中编码。不过，这些结构体确实记录了 saveVolatile 和 restoreVolatile 放置数据的位置。

#### 16.5.2 files.S 文件 I/O 函数

*files.S*源文件包含了库中的文件 I/O 函数。由于这个文件相当长，我会将它分成几部分，依次进行讨论。（我不会包含你传递给这些函数的参数值；这些参数在线上有很好的文档，或者你可以使用 Unix 的 man 命令来查询 read()、write()、open()、openat()和 close()函数。）

大多数的*files.S*函数是外观代码——也就是说，它们存在的目的是改变另一个函数（在本例中是操作系统 API 函数）的环境或参数。这些函数会保存易变寄存器，这样调用者就不需要担心它们的保存问题；在少数情况下（如 open 调用），它们会为调用者自动设置某些默认参数；或者在发生错误时，它们会修改返回码，以便在不同的操作系统中产生一致的结果。file.write 函数演示了如何提供统一的接口（跨操作系统），保存易变寄存器，并返回一致的错误码：

```
// files.S
//
// File I/O functions:

            #include    "../aoaa.inc"
            #include    "files.inc"

            .code
            .align  2

// file.write
//
// Write data to a file handle.
//
// X0- File handle
// X1- Pointer to buffer to write
// X2- Length of buffer to write
//
// Returns:
//
// X0- Number of bytes actually written
//     or -1 if there was an error

            proc    file.write, public

            locals  fw_locals
            qword   fw_locals.saveX0
          ❶ byte    fw_locals.volSave, volatile_save.size
            byte    fw_locals.stkspace, 64
          ❷ dword   fw_locals.fpSave
            endl    fw_locals

            enter   fw_locals.size

            // Preserve all the volatile registers because
            // the OS API write function might modify them.
            //
            // Note: Because fw_locals.volSave is at the
            // bottom of the activation record, SP just
            // happens to be pointing at it right now.
            // Use it to temporarily save FP so you can
            // pass the address of fw_locals.volSave to
            // saveVolatile in the FP register.

          ❸ str     fp, [sp]    // fw_locals.fpSave
            add     fp, fp, #fw_locals.volSave
            bl      saveVolatile
            ldr     fp, [sp]    // Restore FP.

            // Okay, now do the write operation (note that
            // the sys_Write arguments are already sitting
 // in X0, X1, and X2 upon entry into this
            // function):

          ❹ mov     svcReg, #sys_Write
            svc     #OSint

            // Check for error return code:

          ❺ file.checkError

            // Restore the volatile registers, except
            // X0 (because we return the function
            // result in X0):

          ❻ str     x0, [fp, #fw_locals.saveX0] // Return value.
            str     fp, [sp]    // fw_locals.fpSave
            add     fp, fp, #fw_locals.volSave
            bl      restoreVolatile
            ldr     fp, [sp]    // Restore FP.
            ldr     x0, [fp, #fw_locals.saveX0]
            leave
            endp    file.write
```

在激活记录中，file.write 为易变寄存器保存区域❶和特殊变量 fw_locals.fpSave❷保留了空间。代码会使用这个变量在调用 saveVolatile 和 restoreVolatile 时保存 FP 寄存器。注意，fw_locals.fpSave 出现在激活记录的最后，因此当 file.write 构建激活记录时，它将位于栈顶。这是一个临时变量，当系统调用使用栈顶的空间时（假设它们这样做），该变量将不再使用。

接下来，file.write 会将所有的易失性寄存器保存到易失性保存区（fw_locals.volSave） ❸。因为 saveVolatile 期望 FP 指向保存区，所以这段代码将 FP 保存到栈顶（这恰好是 fw_locals.fpSave 变量的位置），然后将 FP 加载为 fw_locals.volSave 结构体的地址，调用 saveVolatile，并在返回时恢复 FP。

请注意，代码不能通过使用 [FP, #fw_locals.fpSave] 寄存器寻址模式来引用 fw_locals.fpSave 变量。首先，激活记录的大小太大，fw_locals.fpSave 的偏移量无法编码进 32 位指令。其次，FP 在从 saveVolatile 返回时并不指向激活记录，因此 [FP, #fw_locals.fpSave] 寄存器寻址模式会引用错误的位置（即使偏移量没有太大问题）。

然后，代码实际调用了 API 函数 ❹。这段代码几乎可以说是微不足道的，因为 write() 函数所需要的所有参数已经在相应的寄存器中，它们已经通过这些寄存器传递给了 file.write。

代码会检查是否发生了错误，并在写入操作过程中如果发生错误，将 X0 中的值进行调整 ❺。file.write 函数会恢复先前保存在 fw_locals.volSave 中的寄存器 ❻。这段代码和解释几乎与 ❸ 中的相同，唯一的不同是：X0 中的值。因为这段代码将函数结果返回在 X0 中，并且 restoreVolatile 会将 X0 恢复到原来的值，所以这段代码必须在调用 restoreVolatile 之前保存和恢复 X0。由于变量 fw_locals.saveX0 在激活记录中出现在 fw_locals.volSave 之前，所以在使用 [FP, #fw_locals.saveX0] 寄存器寻址模式时不会有偏移量过大的问题；只有出现在 fw_locals.volSave 之后的变量，其偏移量会太大，无法在该寻址模式下使用。

file.read 函数几乎与 file.write 函数相同：

```
// files.S (cont.)
//
// file.read
//
// Read data from a file handle.
//
// X0- File handle
// X1- Pointer to buffer receive data
// X2- Length of data to read
//
// Returns:
//
// X0- Number of bytes actually read
//     or negative value if there was an error

            proc    file.read, public

            locals  fr_locals
            qword   fr_locals.saveX0
            byte    fr_locals.volSave, volatile_save.size
            byte    fr_locals.stkspace, 64
            dword   fr_locals.fpSave
            endl    fr_locals

            enter   fr_locals.size

            // Preserve all the volatile registers because
            // the OS API read function might modify them.
            //
            // Note: Because fr_locals.volSave is at the
            // bottom of the activation record, SP just
            // happens to be pointing at it right now.
            // Use it to temporarily save FP so we can
            // pass the address of fr_locals.volSave to
            // saveVolatile in the FP register.

            str     fp, [sp]    // fr_locals.fpSave
            add     fp, fp, #fr_locals.volSave
            bl      saveVolatile
            ldr     fp, [sp]    // Restore FP.

            // Okay, now do the read operation (note that
            // the sys_Read arguments are already sitting
 // in X0, X1, and X2 upon entry into this
            // function):

            mov     svcReg, #sys_Read
            svc     #OSint

            // Check for error return code:

            file.checkError

            // Restore the volatile registers, except
            // X0 (because we return the function
            // result in X0):

            str     x0, [fp, #fr_locals.saveX0] // Return value.
            str     fp, [sp]    // fr_locals.fpSave
            add     fp, fp, #fr_locals.volSave
            bl      restoreVolatile
            ldr     fp, [sp]    // Restore FP.
            ldr     x0, [fp, #fr_locals.saveX0]
            leave
            endp    file.read
```

file.read 和 file.write 唯一的真正区别在于 svcReg 寄存器中加载的函数号。

接下来，*files.S* 提供了 file.open 函数的代码：

```
// files.S (cont.)
//
// file.open
//
// Open existing file for reading or writing.
//
// X0- Pointer to pathname string (zero-terminated)
// X1- File access flags
//     (O_RDONLY, O_WRONLY, or O_RDWR)
//
// Returns:
//
// X0- Handle of open file (or negative value if there
//     was an error opening the file)

            proc    file.open, public

            locals  fo_locals
            qword   fo_locals.saveX0
            byte    fo_locals.volSave, volatile_save.size
            byte    fo_locals.stkspace, 64
            dword   fo_locals.fpSave
            endl    fo_locals

            enter   fo_locals.size

 // Preserve all the volatile registers because
            // the OS API open function might modify them:

            str     fp, [sp]    // fo_locals.fpSave
            add     fp, fp, #fo_locals.volSave
            bl      saveVolatile
            ldr     fp, [sp]    // Restore FP.

            // Call the OS API open function:

          ❶ mov     svcReg, #sys_Open
            mov     x2, x1
            mov     x1, x0
            mov     x0, #AT_FDCWD
            mov     x3, #S_RDWR     // Mode, usually ignored
            svc     #OSint

            // Check for error return code:

            file.checkError

            // Restore the volatile registers, except
            // X0 (because we return the function
            // result in X0):

            str     x0, [fp, #fo_locals.saveX0] // Return value.
            str     fp, [sp]    // fo_locals.fpSave
            add     fp, fp, #fo_locals.volSave
            bl      restoreVolatile
            ldr     fp, [sp]    // Restore FP.
            ldr     x0, [fp, #fo_locals.saveX0]
            leave
            endp    file.open
```

file.open 函数与 file.write 和 file.read 函数相同，唯一不同的是调用了操作系统的 API 函数 ❶。file.open 并不是调用 API 的 open() 函数，而是调用了 openat() 函数，这是 open() 函数的一个更现代的版本。以下是这两个函数的 C/C++ 原型：

```
int open(const char *pathname, int flags);
int openat(int dirfd, const char *pathname, int flags);
```

openat() 函数有一个额外的参数 int dirfd。这使得问题变得复杂，因为 file.open 期望的参数与 open() 函数相同；因此，在进入 file.open 时，参数位于错误的寄存器中，无法直接调用 openat()。

这可以通过将 X1 移至 X2，X0 移至 X1，然后将 X0 加载为 AT_FDCWD 值来修复，从而使 openat()函数表现得像 open()函数一样❶。open()和 openat()函数各有一个可选的第三个或第四个参数（分别），允许你在创建新文件时设置权限。file.open 函数用于打开已存在的文件，因此在调用它时通常不需要指定那个额外的参数。然而，万一调用者在 X1 中指定了 O_CREAT，代码就会将 X3 设置为一个合理的值（为所有人设置读写权限）。

file.openNew 函数是 file.open 的一个变体，用于创建新文件：

```
// files.S (cont.)
//
// file.openNew
//
// Creates a new file and opens it for writing
//
// X0- Pointer to filename string (zero-terminated)
//
// Returns:
//
// X0- Handle of open file (or negative if there
//     was an error creating the file)

            proc    file.openNew, public

            locals  fon_locals
            qword   fon_locals.saveX0
            byte    fon_locals.volSave, volatile_save.size
            byte    fon_locals.stkspace, 64
            dword   fon_locals.fpSave
            endl    fon_locals

            enter   fon_locals.size

            // Preserve all the volatile registers because
            // the OS API open function might modify them:

            str     fp, [sp]    // fon_locals.fpSave
            add     fp, fp, #fon_locals.volSave
            bl      saveVolatile
            ldr     fp, [sp]    // Restore FP.

            // Call the OS API open function:

            mov     svcReg, #sys_Open
            mov     x2, #O_CREAT+O_WRONLY+O_EXCL
            mov     x1, x0
            mov     x0, #AT_FDCWD
            mov     x3, #S_RDWR // User/Group has RW perms.
            svc     #OSint

            // Check for error return code:

            file.checkError

            // Restore the volatile registers, except
            // X0 (because we return the function
 // result in X0):

            str     x0, [fp, #fon_locals.saveX0] // Return value.
            str     fp, [sp]    // w_locals.fpSave
            add     fp, fp, #fon_locals.volSave
            bl      restoreVolatile
            ldr     fp, [sp]    // Restore FP.
            ldr     x0, [fp, #fon_locals.saveX0]
            leave
            endp    file.openNew
```

file.openNew 和 file.open 之间唯一的区别是，file.openNew 只期望一个参数（X0 中的路径名），并为调用 openat()自动提供标志值（O_CREAT+O_WRONLY+O_EXCL）。

file.close 函数是*files.S*源文件中的最后一个文件 I/O 函数：

```
// files.S (cont.)
//
// file.close
//
// Closes a file specified by a file handle
//
// X0- Handle of file to close

            proc    file.close, public

            locals  fc_locals
            qword   fc_locals.saveX0
            byte    fc_locals.volSave, volatile_save.size
            byte    fc_locals.stkspace, 64
            dword   fc_locals.fpSave
            endl    fc_locals

            enter   fc_locals.size

            // Preserve all the volatile registers because
            // the OS API open function might modify them:

            str     fp, [sp]    // fc_locals.fpSave
            add     fp, fp, #fc_locals.volSave
            bl      saveVolatile
            ldr     fp, [sp]    // Restore FP.

            // Call the OS API close function (handle is
            // already in X0):

            mov     svcReg, #sys_Close
            svc     #OSint

            // Check for error return code:

            file.checkError

 // Restore the volatile registers, except
            // X0 (because we return the function
            // result in X0):

            str     x0, [fp, #fc_locals.saveX0] // Return value.
            str     fp, [sp]    // w_locals.fpSave
            add     fp, fp, #fc_locals.volSave
            bl      restoreVolatile
            ldr     fp, [sp]    // Restore FP.
            ldr     x0, [fp, #fc_locals.saveX0]
            leave
            endp    file.close
```

file.close 函数期望在 X0 中传入文件描述符（由成功调用 file.open 或 file.openNew 返回），并将该描述符传递给 API 的 close()函数。否则，它的形式与 file.read 和 file.write 函数类似。

#### 16.5.3 stdio.S 函数

文件库中的最后一个源文件是*stdio.S*文件。这个模块包含了你可以用来在控制台（标准 I/O）设备上读写字符串的函数。由于它的大小，我将这个源文件拆分成了更易于消化的部分。

首先，stdout.puts 函数将一个（以零结束的）字符串写入标准输出设备（通常是显示控制台）：

```
// stdio.S
//
// Standard input and standard output functions:

            #include    "../aoaa.inc"
            #include    "files.inc"
            #include    <sys/syscall.h>

            .code
            .align  2

// stdout.puts
//
// Outputs a zero-terminated string to standard output device
//
// X0- Address of string to print to standard output

            proc    stdout.puts, public

            locals  lcl_puts
          ❶ qword   lcl_puts.saveX0X1
            dword   lcl_puts.saveX2
            byte    lcl_puts.stkSpace, 64
            endl    lcl_puts

            enter   lcl_puts.size

 stp     x0, x1, [fp, #lcl_puts.saveX0X1]
            str     x2,     [fp, #lcl_puts.saveX2]

            mov     x1, x0

// Compute the length of the string:

❷ lenLp:      ldrb    w2, [x1], #1
            cbnz    w2, lenLp
            sub     x2, x1, x0  // Compute length

            // Call file_write to print the string:

          ❸ mov     x1, x0
            mov     x0, #stdout
            bl      file.write

            // Return to caller:

            ldr     x2,     [fp, #lcl_puts.saveX2]
            ldp     x0, x1, [fp, #lcl_puts.saveX0X1]
            leave
            endp    stdout.puts
```

请注意，这段代码并没有保存所有的易失性寄存器❶，因为 stdout.puts 函数并没有直接调用可能修改寄存器的操作系统 API 函数。因此，这个函数只保留它实际使用的寄存器。

这个函数将调用 file.write 将字符串写入标准输出设备。file.write 函数需要三个参数：文件描述符（stdout 常量对于描述符值非常合适）、数据的地址（字符串）以及长度值。虽然 stdout.puts 函数在 X0 中具有字符串的地址，但没有长度参数。因此，这段代码计算了地址在 X0 中的以零结束的字符串的长度❷。

> 注意

*一次扫描一个字节来计算字符串长度是一个简陋的方式，但我在这里使用它，因为它比其他方法更简单。如果这真的让你感到困扰，你可以链接 C 标准库* strlen() *函数。不过，请记住，进行系统调用并在屏幕上绘制所有这些像素来打印字符串的速度比这个字符串长度计算慢得多，因此使用更快的字符串长度计算代码并不会节省多少时间。*

一旦计算出长度，stdout.put 函数会调用 file.write 将字符串实际打印到标准输出设备 ❸。在恢复这段代码修改过的几个寄存器后，函数返回。

从技术上讲，file.write 可能返回一个错误代码（stdout.puts 会忽略这个错误并且不会将其返回给调用者）。然而，这种错误的可能性较低，因此这段代码忽略了错误。一个可能的问题是，如果标准输出被重定向到磁盘文件，而写入磁盘时出现问题，那么这个 bug 就值得关注。如果这个例程要进入生产代码，应该解决这个问题；我选择在这里不处理此问题，以保持代码的简洁（而且发生这种情况的概率极低）。

stdout.newLn 函数与 stdout.puts 相同，只是它会向标准输出设备写入一个固定的字符串（换行符）：

```
// stdio.S (cont.)
//
// stdout.newLn
//
// Outputs a newline sequence to the standard output device:

stdout.nl:  .ascii  "\n"
nl.len      =       .-stdout.nl
            .byte   0
            .align  2

            proc    stdout.newLn, public
            locals  lcl_nl
            qword   lcl_nl.saveX0X1
            dword   lcl_nl.saveX2
            byte    lcl_nl.stkSpace, 64
            endl    lcl_nl

            enter   lcl_nl.size
            stp     x0, x1, [fp, #lcl_nl.saveX0X1]
            str     x2,     [fp, #lcl_nl.saveX2]

            lea     x1, stdout.nl
            mov     x2, #nl.len
            mov     x0, stdout
            bl      file.write

            ldr     x2,     [fp, #lcl_nl.saveX2]
            ldp     x0, x1, [fp, #lcl_nl.saveX0X1]
            leave
            endp    stdout.newLn
```

stdin.read 函数是 stdout.write 的输入补充：

```
// stdio.S (cont.)
//
// stdin.read
//
// Reads data from the standard input
//
// X0- Buffer to receive data
// X1- Buffer count (note that data input will
//     stop on a newline character if that
//     comes along before X1 characters have
//     been read)
//
// Returns:
//
// X0- Negative value if error, bytes read if successful

             proc    stdin.read, public
             locals  sr_locals
             qword   sr_locals.saveX1X2
             byte    sr_locals.stkspace, 64
             dword   sr_locals.fpSave
             endl    sr_locals

             enter   sr_locals.size
             stp     x1, x2, [fp, #sr_locals.saveX1X2]

             // Call the OS API read function:

           ❶ mov     svcReg, #sys_Read
             mov     x2, x1
             mov     x1, x0
             mov     x0, #stdin
             svc     #OSint

             file.checkError

             ldp     x1, x2, [fp, #sr_locals.saveX1X2]
             leave
             endp    stdin.read
```

当你传递给 stdin.read 一个缓冲区的地址和大小时，它会从标准输入设备（通常是键盘）读取指定数量的字符，并将这些字符放入缓冲区。当它读取到指定数量的字符或者遇到换行符（换行符）时，读取会停止。

这段看似简单的代码的关键在于，在调用操作系统的 read() 函数之前，需要先移动地址和字节数 ❶。这是因为在调用 read() 时，缓冲区的地址需要放入 X1 和 X2 寄存器中，而函数必须将标准输入文件描述符加载到 X0 寄存器中。

stdin.getc 函数是 stdin.read 的一个字符版本：

```
// stdio.S (cont.)
//
// stdin_getc
//
// Read a single character from the standard input.
// Returns character in X0 register

            proc    stdin.getc, public
            locals  sgc_locals
            qword   sgc_locals.saveX1X2
          ❶ byte    sgc_buf, 16
            byte    sgc_locals.stkspace, 64
            endl    sgc_locals

 enter   sgc_locals.size
            stp     x1, x2, [fp, #sgc_locals.saveX1X2]

            // Initialize return value to all 0s:

          ❷ str     xzr, [fp, #sgc_buf]

            // Call the OS API read function to read
            // a single character:

            mov     svcReg, #sys_Read
            mov     x0, #stdin
          ❸ add     x1, fp, #sgc_buf
            mov     x2, #1
            svc     #OSint

          ❹ file.checkError
            cmp     x0, #0
            bpl     noError

            // If there was an error, return the
            // error code in X0 rather than a char:

            str     x0, [fp, #sgc_buf]

noError:
            ldp     x1, x2, [fp, #sgc_locals.saveX1X2]
          ❺ ldr     x0, [fp, #sgc_buf]
            leave

            endp    stdin.getc
```

stdin.getc 函数返回它在 X0 中读取的字符，而不是将其放入缓冲区。由于调用 API 的 read() 函数需要一个缓冲区，这个函数必须为缓冲区预留存储空间 ❶。从技术上讲，缓冲区只需要 8 个字符长，但这个函数预留了 16 字节，只是为了帮助保持栈的 16 字节对齐。该代码将缓冲区的前 8 个字节初始化为 0 ❷。函数实际上会返回所有 8 个字节（即使读取操作只会将一个字节存储到缓冲区）。该函数计算缓冲区的地址并将其传递给 API 的 read() 函数 ❸。

如果某种情况下，调用 API 的 read() 函数返回错误，代码将把负的错误返回代码存储在缓冲区的前 8 个字节中 ❹。在返回之前，stdin.getc 会将缓冲区开头的 8 个字节加载到 X0 寄存器中，并返回该值（这可以是一个字符加上七个 0，或者一个 UTF-8 值，或者是 8 字节的负错误代码） ❺。

> 注意

*stdin.get* 函数并不是从键盘读取单个字符并立即返回给调用者。相反，操作系统会从键盘读取整行文本，并返回该行的第一个字符。对 *stdin.get* 的后续调用将从该操作系统内部缓冲区读取剩余字符。这是标准的 Unix 行为，而非该函数的特定功能。

*stdio.S* 文件中的最后一个函数是 stdin.readln：

```
// stdio.S (cont.)
//
// stdin.readln
//
// Reads a line of text from the user.
// Automatically processes backspace characters
// (deleting previous characters, as appropriate).
// Line returned from function is zero-terminated
// and does not include the ENTER key code (carriage
// return) or line feed.
//
// X0- Buffer to place line of text read from user
// X1- Maximum buffer length
//
// Returns:
//
// X0- Number of characters read from the user
//     (does not include ENTER key)

            proc    stdin.readln, public
            locals  srl_locals
            qword   srl_locals.saveX1X2
            dword   srl_locals.saveX3
            byte    srl_buf, 16
            byte    srl_locals.stkspace, 64
            endl    srl_locals

            enter   srl_locals.size
            stp     x1, x2, [fp, #srl_locals.saveX1X2]
            str     x3,     [fp, #srl_locals.saveX3]

            mov     x3, x0          // Buf ptr in X3
            mov     x2, #0          // Character count
            cbz     x1, exitRdLn    // Bail if zero chars.

            sub     x1, x1, #1      // Leave room for 0 byte.
readLp:
          ❶ bl      stdin.getc      // Read 1 char from stdin.

            cmp     w0, wzr         // Check for error.
            bmi     exitRdLn

          ❷ cmp     w0, #cr         // Check for newline code.
            beq     lineDone

            cmp     w0, #lf         // Check for newline code.
            beq     lineDone

 ❸ cmp     w0, #bs         // Handle backspace character.
            bne     addChar

// If a backspace character came along, remove the previous
// character from the input buffer (assuming there is a
// previous character):

            cmp     x2, #0          // Ignore BS character if no
            beq     readLp          // chars in the buffer.
            sub     x2, x2, #1
            b.al    readLp

// If a normal character (that we return to the caller),
// add the character to the buffer if there is room
// for it (ignore the character if the buffer is full):

❹ addChar:    cmp     x2, x1          // See if you're at the
            bhs     readLp          // end of the buffer.
            strb    w0, [x3, x2]    // Save char to buffer.
            add     x2, x2, #1
            b.al    readLp

// When the user presses the ENTER key (or line feed)
// during input, come down here and zero-terminate the string:

lineDone:
          ❺ strb    wzr, [x3, x2]

exitRdLn:   mov     x0, x2          // Return char cnt in X0.
            ldp     x1, x2, [fp, #srl_locals.saveX1X2]
            ldr     x3,     [fp, #srl_locals.saveX3]
            leave
            endp    stdin.readln
```

这个函数主要用于交互式使用，它会从键盘读取一行文本并进行适度编辑（处理退格符），将这些字符放入缓冲区。从许多方面来看，它的工作方式就像 stdin.read，只是按下 BACKSPACE 键会删除输入缓冲区中的一个字符，而不是将退格符 ASCII 代码作为缓冲区中的字符返回。

这个函数重复调用 stdin.getc❶，每次读取一个字符。如果 stdin.getc 返回错误（负返回值），该函数会立即返回，将错误代码传递给调用者。

代码通过将字符与回车符或换行符（换行符）❷的 ASCII 代码进行比较，检查输入行是否完成。如果字符与这两者中的任何一个匹配，代码会退出循环。

然后，stdin.readln 函数会检查是否有退格符❸。如果是退格符，该函数将从输入缓冲区删除之前的字符（如果有的话）。如果字符不是退格符，代码会跳转到❹，在这里它将该字符追加到缓冲区的末尾。

当函数在输入流中找到回车符或换行符时，它会将控制权转移到❺，在那里它会将字符串终止并返回实际读取的字符数到 X0 寄存器中。

除了处理退格符，读取文本行时使用 stdin.readln 和简单调用 stdin.read 之间还有两个额外的区别。首先，stdin.readln 会将读取到缓冲区的字符串终止。其次，stdin.readln 不会将换行符（或回车符）放入缓冲区。

#### 16.5.4 文件 I/O 演示应用

清单 16-4 中的简单应用演示了 *file.a* 库的使用。

```
// Listing16-4.S
//
// File I/O demonstration:

            #include    "aoaa.inc"
            #include    "files/files.inc"
            #include    <sys/syscall.h>

            #if isMacOS

// Map main to "_main" as macOS requires
// underscores in front of global names
// (inherited from C code, anyway).

#define main _main
sys_Exit    =   SYS_exit

            #else

sys_Exit    =   __NR_exit

            #endif

            .data

// Buffer to hold line of text read from user:

inputLn:    .space  256, (0)
inputLn.len =       .-inputLn

// Buffer to hold data read from a file:

fileBuffer: .space  4096, (0)
fileBuffer.len =    .-fileBuffer

// Prompt the user for a filename:

prompt:     .ascii  "Enter (text) filename:"
prompt.len  =       .-prompt
            .byte   0

// Error message string:

badOpenMsg: wastr   "Could not open file\n"

OpenMsg:    wastr   "Opening file: "

            .code

// Here is the asmMain function:

            proc    main, public
            locals  am
            dword   am.inHandle
            byte    am_stkSpace, 64
            endl    am

            enter   am.size

// Get a filename from the user:

          ❶ lea     x0, prompt
            bl      stdout.puts

            lea     x0, inputLn
            mov     x1, #inputLn.len
            bl      stdin.readln
            cmp     x0, #0
            bmi     badOpen

            lea     x0, OpenMsg
            bl      stdout.puts
            lea     x0, inputLn
            bl      stdout.puts
            bl      stdout.newLn

// Open the file, read its contents, and display
// the contents to the standard output device:

          ❷ lea     x0, inputLn
            mov     x1, #O_RDONLY
            bl      file.open
            cmp     x0, xzr
            ble     badOpen

            str     x0, [fp, #am.inHandle]

// Read the file 4,096 bytes at a time:

readLoop:
          ❸ ldr     x0, [fp, #am.inHandle]
            lea     x1, fileBuffer
            mov     x2, fileBuffer.len
            bl      file.read

 // Quit if there was an error or
            // file.read read 0 bytes:

            cmp     x0, xzr
            ble     allDone

            // Write the data just read to the
            // stdout device:

          ❹ mov     x2, x0        // Bytes to write
            lea     x1, fileBuffer
            mov     x0, #stdout
            bl      file.write
            b.al    readLoop

badOpen:    lea     x0, badOpenMsg
            bl      stdout.puts

allDone:
          ❺ ldr     x0, [fp, #am.inHandle]
            bl      file.close

            // Return error code 0 to the OS:

            mov     svcReg, #sys_Exit
            mov     x0, #0
            svc     #OSint
            endp    main
```

该程序首先提示用户输入一个文件名❶。它从用户那里读取这个文件名，然后将文件名回显到显示屏上。程序打开该文件并保存由文件 .open 返回的文件句柄❷。如果在打开文件时发生错误，程序会跳转到 badOpen 标签，打印错误信息并退出。

接下来，程序会不断地读取（最多）4,096 字节的块，直到文件末尾被读取完（或发生其他错误）❸。从文件读取时，file.read 函数会读取完整的 4,096 字节，忽略所有换行符（仅在从标准输入读取时才会在换行符处停止）。如果该函数从输入中读取到 0 字节，表示已到达文件末尾，循环会退出。

然后，代码将读取的字节写入标准输出设备 ❹，并使用 file.read 的返回值作为调用 file.write 时的字节计数。这是因为从文件读取的最后一块字节可能不是 4,096 字节；如果读取的字节少于 4,096 字节，下一次读取将返回 0 字节，操作就会完成。一旦程序完成，它会关闭文件并退出 ❺>。

这是将在清单 16-4 中构建程序的 makefile：

```
# Listing16-4.mak
#
# makefile to build the Listing16-4.S file

unamestr=`uname`

Listing16-4:Listing16-4.S aoaa.inc files/files.inc files.a
    cd files; make -f files.mak; cd ..
    g++ -D$(unamestr) -o Listing16-4 Listing16-4.S files.a

clean:
    rm -f Listing16-4.o
    rm -f Listing16-4
    rm -f file.a
    cd files; make -f files.mak clean; cd ..
```

这是一个示例构建操作和程序执行：

```
% make -f Listing16-4.mak clean
rm -f Listing16-4.o
rm -f Listing16-4
rm -f file.a
cd files; make -f files.mak clean; cd ..
rm -f files.o
rm -f volatile.o
rm -f stdio.o
rm -f files.a
% make -f Listing16-4.mak
cd files; make -f files.mak; cd ..
g++ -c -D`uname` files.S
g++ -c -D`uname` stdio.S
g++ -c -D`uname` volatile.S
ar rcs files.a files.o stdio.o volatile.o
cp files.a ..
g++ -D`uname` -o Listing16-4 Listing16-4.S files.a
% ./Listing16-4
Enter (text) filename:Listing16-4.mak
Opening file: Listing16-4.mak
# listing16-4.mak
#
# makefile to build the Listing16-4.S file.

unamestr=`uname`

Listing16-4:Listing16-4.S aoaa.inc files/files.inc files.a
    cd files; make -f files.mak; cd ..
    g++ -D$(unamestr) -o Listing16-4 Listing16-4.S files.a

clean:
    rm -f Listing16-4.o
    rm -f Listing16-4
    rm -f file.a
    cd files; make -f files.mak clean; cd ..
```

我使用了*Listing16-4.mak*文本文件作为这次程序运行的输入。

### 16.6 在 macOS 下调用系统库函数

正如我之前提到的，苹果公司对直接通过 svc 指令调用 macOS 内核的应用程序持反对态度。公司声称，正确的调用方式是通过苹果提供的 C 库代码。本章展示了低级调用，因为这本章的目的就是这个；如果你是编写 C 库代码的人（或者是类似的与操作系统交互的库代码），你需要了解这些信息。然而，如果我不向你展示苹果推荐的与 macOS 接口的方式，我就失职了。

我创建了一个变体的*files.a*库，存储在在线源码集中的*files-macOS*目录下，链接了内核的 read()、write()、open()和 close()函数。为了避免冗余，我没有在本章中打印所有代码，但我将在这里列出 file.write 函数，给你一个关于修改简单性的概念：

```
// file.write
//
// Write data to a file handle.
//
// X0- File handle
// X1- Pointer to buffer to write
// X2- Length of buffer to write
//
// Returns:
//
// X0- Number of bytes actually written
//     or -1 if there was an error

            proc    file.write, public
            locals  fw_locals
            qword   fw_locals.saveX0
            byte    fw_locals.volSave, volatile_save.size
            byte    fw_locals.stkspace, 64
            dword   fw_locals.fpSave
            endl    fw_locals

            enter   fw_locals.size

            // Preserve all the volatile registers because
            // the OS API write function might modify them.
            //
            // Note: because fw_locals.volSave is at the
            // bottom of the activation record, SP just
            // happens to be pointing at it right now.
            // Use it to temporarily save FP so you can
            // pass the address of w_locals.volSave to
            // saveVolatile in the FP register.

            str     fp, [sp]    // fw_locals.fpSave
            add     fp, fp, #fw_locals.volSave
            bl      saveVolatile
            ldr     fp, [sp]    // Restore FP.

 // Okay, now do the write operation (note that
            // the write arguments are already sitting
            // in X0, X1, and X2 upon entry into this
            // function):

          ❶ bl      _write

            // Check for error return code:

            file.checkError

            // Restore the volatile registers, except
            // X0 (because we return the function
            // result in X0):

            str     x0, [fp, #fw_locals.saveX0] // Return value.
            str     fp, [sp]    // w_locals.fpSave
            add     fp, fp, #fw_locals.volSave
            bl      restoreVolatile
            ldr     fp, [sp]    // Restore FP.
            ldr     x0, [fp, #fw_locals.saveX0]
            leave
            endp    file.write
```

这个版本的 file.write 与原始*files.a*库中的版本之间唯一的区别是，我将 svc 指令序列替换为对 _write()函数的调用 ❶。

新的*files.a*库还包括了对*files.inc*头文件的一些更改。最重要的更改是对 file.checkError 宏的修改：

```
 .macro  file.checkError

            cmp     x0, #-1
            bne     0f
            getErrno
            neg     x0, x0
0:
            .endm
```

macOS 的 _write()函数在发生错误时返回-1，因为 C 代码无法测试进位标志。因此，我修改了 file.checkError 函数，以便像 Linux 那样处理错误。

我必须先构建*files-macOS*库（以创建一个新的*file.a*版本，替换掉直接进行操作系统调用的版本），然后使用*files-macOS*中的*file.a*库创建*Listing16-4.S*。程序的运行与前一节中的原始文件 I/O 示例相同。

理论上，你可以使用相同的方法在 Linux 上操作，这样可以创建在这两种操作系统之间稍微更具可移植性的代码。然而，Linux 下的 svc API 接口已经明确定义并且有文档，所以没有理由不直接调用 API 函数。### 16.7 在没有 GCC 的情况下创建汇编应用程序

在本章中，我继续使用 GCC 来汇编和链接汇编语言文件。这是因为本章中的大多数示例代码都包含*aoaa.inc*，而这个文件依赖于 CPP。你可能对这种方法有所怀疑，认为 GCC 可能会偷偷把一些 C 代码引入到你的程序中。你是对的：即使你用 GCC 构建一个“纯”汇编语言程序，它也会链接一些代码，在程序执行之前设置环境（这样，如果你确实调用任何 C 库代码，环境就已经为其做好了准备）。

通常，这类额外的代码影响不大——它只执行一次，执行速度较快，并且占用的空间也不多。然而，如果你是一个绝对的纯粹主义者，并且只希望执行你自己编写的代码，你可以通过一些额外的工作做到这一点。你只是不能使用*aoaa.inc*，而且你必须专门为 macOS 或 Linux 编写不可移植的代码。

清单 16-5 是为 Linux 编写的“纯”汇编语言程序。

```
❶ // Listing16-5.s
//
// A truly stand-alone "Hello, world!" program
// written for Linux

        .text
      ❷ .global     _start
        .align      2
hwStr:  .asciz      "Hello, world!\n"
hwSize  =           .-hwStr
        .align      2

❸ _start:

        mov         x0, #1          // stdout file handle
        adr         x1, hwStr       // String to print
        mov         x2, #hwSize     // Num chars to print
        mov         X8, #64         // __NR_write
        svc         #0              // Call OS to print str.

        mov         X8, #93         // __NR_exit
        mov         x0, #0
        svc         #0              // Quit program.
```

请注意，这个文件名必须有小写的*.s*后缀 ❶；你不会使用 GCC 进行编译，因此不会在这个代码中使用 CPP。在 Linux 下，默认的程序入口点名为 _start。因此，这段代码将 _start 声明为全局符号 ❷，并使用 _start 作为程序的入口点 ❸。在本章早些时候的示例中，我可以使用 main（或 _main），因为 GCC 链接的 C 代码提供了 _start 标签并将控制权转交给 main（或 _main）；然而，由于我们放弃了 GCC 生成的代码，我们必须显式地提供 _start 标签。

要汇编、链接并运行这个程序，请使用以下 Linux 命令：

```
as -o Listing16-5.o Listing16-5.s
ld -s -o Listing16-5 Listing16-5.o
./Listing16-5
Hello, world!
```

要使这个程序在 macOS 下运行，你必须首先修改源代码，以使用适当的 macOS API 常量，如清单 16-6 所示。

```
// Listing16-6.s
//
// A truly stand-alone "Hello, world!" program
// written for macOS

        .text
        .global     _start
        .global     _main    // Later versions of macOS require this name.
        .align      2
hwStr:  .asciz      "Hello, world!\n"
hwSize  =           .-hwStr
        .align      2

_start:
_main:

        mov         x0, #1          // stdout file handle
        adr         x1, hwStr       // String to print
        mov         x2, #hwSize     // Num chars to print
        mov         X16, #4         // SYS_write
        svc         #0x80           // Call OS to print str.

        mov         X16, #1         // SYS_exit
        mov         x0, #0
        svc         #0x80           // Quit program.

        svc         #0              // Quit program.
```

汇编代码与 Linux 类似（请注意，程序名后缀也是小写的*.s*）：

```
as -arch arm64 -o Listing16-6.o Listing16-6.s
```

然而，链接程序稍微复杂一些：

```
ld -macos_version_min 12.3.0 -o HelloWorld Listing16-6.o \
 -lSystem -syslibroot `xcrun -sdk macosx --show-sdk-path` -arch arm64
```

在 macOS 下构建纯汇编文件时，最好使用 makefile；每次想要构建应用程序时手动输入这些命令是相当繁琐的！

如你所见，链接器（ld）命令仍然会链接一些 C 代码（libSystem）。我知道的没有其他方法可以避免这一点，这也是我非常乐意让 GCC 为我做这些工作的原因。

> 注意

*Apple 并不是开玩笑，当它警告你不要编写这样的代码时。在我第一次编写这个“Hello, World！”程序和本章审阅之间的时间里，Apple 对其系统进行了更改，导致程序无法运行。特别是，现在链接器期望程序的名称是* _main *而不是* _start *，并且* ld *的命令行有一些微妙的变化。故事的寓意是：坚持使用 GCC（Clang）来为你做所有这些工作。*

### 16.8 更多信息

+   你可以在*[`<wbr>github<wbr>.com<wbr>/torvalds<wbr>/linux<wbr>/blob<wbr>/v4<wbr>.17<wbr>/include<wbr>/uapi<wbr>/asm<wbr>-generic<wbr>/unistd<wbr>.h`](https://github.com/torvalds/linux/blob/v4.17/include/uapi/asm-generic/unistd.h)*上找到 Linux 的系统调用编号。

+   你可以在*[`<wbr>github<wbr>.com<wbr>/opensource<wbr>-apple<wbr>/xnu<wbr>/blob<wbr>/master<wbr>/bsd<wbr>/kern<wbr>/syscalls<wbr>.master`](https://github.com/opensource-apple/xnu/blob/master/bsd/kern/syscalls.master)*或*[`<wbr>opensource<wbr>.apple<wbr>.com<wbr>/source<wbr>/xnu<wbr>/xnu<wbr>-1504<wbr>.3<wbr>.12<wbr>/bsd<wbr>/kern<wbr>/syscalls<wbr>.master`](https://opensource.apple.com/source/xnu/xnu-1504.3.12/bsd/kern/syscalls.master)*上找到 macOS 的系统调用编号。
