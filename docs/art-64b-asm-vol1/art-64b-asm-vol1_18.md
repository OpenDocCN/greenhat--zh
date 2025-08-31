# 第十六章：独立汇编语言程序

![](img/chapterart.png)

到目前为止，本书依赖于 C/C++ 主程序来调用用汇编语言编写的示例代码。尽管这可能是汇编语言在现实世界中的最大应用，但也可以在汇编语言中编写独立的代码（没有 C/C++ 主程序）。

在本章的上下文中，*独立汇编语言程序*指的是你编写的一个可执行的汇编程序，它不会直接链接到 C/C++ 程序中执行。没有 C/C++ 主程序调用你的汇编代码，你就不会拖带 C/C++ 库代码和运行时系统，因此你的程序会更小，也不会与 C/C++ 公共名称发生外部命名冲突。然而，你必须自己完成很多 C/C++ 库所做的工作，或者编写相应的汇编代码，或调用 Win32 API。

*Win32 API* 是一个裸金属接口，提供给 Windows 操作系统，提供了成千上万的函数，你可以从独立的汇编语言程序中调用——本章无法考虑所有这些函数。 本章为你提供了 Win32 应用程序的基本介绍（尤其是基于控制台的应用程序）。这些信息将帮助你开始在 Windows 下编写独立的汇编语言程序。

要在你的汇编程序中使用 Win32 API，你需要从 [`www.masm32.com/`](https://www.masm32.com/) 下载 MASM32 库包。^(1) 本章中的大多数示例假设 MASM32 64 位包含文件已经在你的系统的 *C:\masm32* 子目录中。

## 16.1 独立的 Hello World

在向你展示一些 Windows 独立汇编语言编程的奇迹之前，也许最好的起点是从头开始：一个独立的“Hello, world!”程序（清单 16-1）。

```
; Listing 16-1.asm

; A stand-alone assembly language version of 
; the ubiquitous "Hello, world!" program.

; Link in the Windows Win32 API:

            includelib kernel32.lib

; Here are the two Windows functions we will need
; to send "Hello, world!" to the standard console device:

            extrn __imp_GetStdHandle:proc
            extrn __imp_WriteFile:proc

            .code
hwStr       byte    "Hello World!"
hwLen       =       $-hwStr

; This is the honest-to-goodness assembly language
; main program:

main        proc

; On entry, stack is aligned at 8 mod 16\. Setting aside
; 8 bytes for "bytesWritten" ensures that calls in main have
; their stack aligned to 16 bytes (8 mod 16 inside function),
; as required by the Windows API (which __imp_GetStdHandle and
; __imp_WriteFile use. They are written in C/C++).

            lea     rbx, hwStr
            sub     rsp, 8
            mov     rdi, rsp      ; Hold # of bytes written here

; Note: must set aside 32 bytes (20h) for shadow registers for
; parameters (just do this once for all functions). 
; Also, WriteFile has a 5th argument (which is NULL), 
; so we must set aside 8 bytes to hold that pointer (and
; initialize it to zero). Finally, stack must always be 
; 16-byte-aligned, so reserve another 8 bytes of storage
; to ensure this.

            sub     rsp, 030h  ; Shadow storage for args

; Handle = GetStdHandle(-11);
; Single argument passed in ECX.
; Handle returned in RAX.

            mov     rcx, -11                     ; STD_OUTPUT
            call    qword ptr __imp_GetStdHandle ; Returns handle
                                                 ; in RAX

; WriteFile(handle, "Hello World!", 12, &bytesWritten, NULL);
; Zero out (set to NULL) "lpOverlapped" argument:

            xor     rcx, rcx
            mov     [rsp + 4 * 8], rcx

            mov     r9, rdi    ; Address of "bytesWritten" in R9
            mov     r8d, hwLen ; Length of string to write in R8D
            lea     rdx, hwStr ; Ptr to string data in RDX
            mov     rcx, rax   ; File handle passed in RCX
            call    qword ptr __imp_WriteFile

; Clean up stack and return:

            add     rsp, 38h
            ret
main        endp
            end
```

清单 16-1：独立的“Hello, world!”程序

`__imp_``GetStdHandle` 和 `__imp_``WriteFile` 过程是 Windows 内的函数（它们是所谓的 Win32 API 的一部分，尽管这是执行的 64 位代码）。`__imp_GetStdHandle` 过程，在传入（虽然是魔法般的）数字 -11 作为参数时，返回标准输出设备的句柄。使用这个句柄，调用 `__imp_WriteFile` 将把输出发送到标准输出设备（控制台）。要构建并运行此程序，使用以下命令：

```
ml64 listing16-1.asm /link /subsystem:console /entry:main
```

MASM 的`/link`命令行选项告诉它，接下来的命令（直到行末）将被传递给链接器。`/subsystem:console`（链接器）命令行选项告诉链接器这个程序是一个控制台应用程序（也就是说，它将在命令行窗口中运行）。`/entry:main`链接器选项将主程序的名称传递给链接器。链接器将这个地址存储在可执行文件中的一个特殊位置，以便 Windows 在将可执行文件加载到内存后确定主程序的起始地址。

## 16.2 头文件与 Windows 接口

在 Listing 16-1 的“Hello, world!”示例的开始部分，你会注意到以下几行：

```
includelib kernel32.lib

; Here are the two Windows functions we will need
; to send "Hello, world!" to the standard console device:

extrn __imp_GetStdHandle:proc
extrn __imp_WriteFile:proc
```

*kernel32.lib*库文件包含了许多 Win32 API 函数的对象模块定义，包括`__imp_GetStdHandle`和`__imp_WriteFile`过程。为所有 Win32 API 函数在你的汇编语言程序中插入`extrn`指令是一个巨大的工作量。处理这些函数定义的正确方式是将它们包含在一个头文件（包含文件）中，然后在你编写的每个使用 Win32 API 函数的应用程序中都包含这个文件。

坏消息是，创建一个合适的头文件集合是一个庞大的任务。好消息是，已经有人为你做了所有这些工作：MASM32 头文件。Listing 16-2 是 Listing 16-1 的重做版，使用 MASM32 64 位包含文件来获取 Win32 外部声明。请注意，我们通过包含文件*listing16-2.inc*来引入 MASM32，而不是直接使用它。稍后会详细解释。

```
; Listing 16-2

            include    listing16-2.inc
            includelib kernel32.lib               ; File I/O library

; Include just the files we need from masm64rt.inc:

;           include \masm32\include64\masm64rt.inc
;           OPTION DOTNAME                        ; Required for macro files
;           option casemap:none                   ; Case sensitive
;           include \masm32\include64\win64.inc
;           include \masm32\macros64\macros64.inc
;           include \masm32\include64\kernel32.inc

            .data
bytesWrtn   qword   ?
hwStr       byte    "Listing 16-2", 0ah, "Hello, World!", 0
hwLen       =       sizeof hwStr

            .code

**********************************************************

; Here is the "asmMain" function.

            public  asmMain
asmMain     proc
            push    rbx
            push    rsi
            push    rdi
            push    r15
            push    rbp
            mov     rbp, rsp
            sub     rsp, 56            ; Shadow storage
            and     rsp, -16

            mov     rcx, -11           ; STD_OUTPUT
            call    __imp_GetStdHandle ; Returns handle

            xor     rcx, rcx
            mov     bytesWrtn, rcx

            lea     r9, bytesWrtn      ; Address of "bytesWritten" in R9
            mov     r8d, hwLen         ; Length of string to write in R8D 
            lea     rdx, hwStr         ; Ptr to string data in RDX
            mov     rcx, rax           ; File handle passed in RCX
            call    __imp_WriteFile

allDone:    leave
            pop     r15
            pop     rdi
            pop     rsi
            pop     rbx
            ret     ; Returns to caller
asmMain     endp
            end
```

这是*listing16-2.inc*包含文件：

```
; listing16-2.inc

; Header file entries extracted from MASM32 header
; files (placed here rather than including the 
; full MASM32 headers to avoid namespace pollution
; and speed up assemblies).

PPROC           TYPEDEF PTR PROC        ; For include file prototypes

externdef __imp_GetStdHandle:PPROC
externdef __imp_WriteFile:PPROC
```

Listing 16-2: 使用 MASM32 64 位包含文件

这是构建命令和示例输出：

```
C:\>**ml64 /nologo listing16-2.asm kernel32.lib /link /nologo /subsystem:console /entry:asmMain**
 Assembling: listing16-2.asm

C:\>**listing16-2**
Listing 16-2
Hello, World!
```

MASM32 包含文件

```
include \masm32\include64\masm64rt.inc
```

包含了 MASM32 64 位系统中的其他数百个包含文件。将这个包含指令加入到你的程序中，能够为你的应用程序提供对大量 Win32 API 函数、数据声明和其他资源（如 MASM32 宏）的访问。

然而，当你组装源文件时，计算机会暂停一会儿。这是因为那个单一的包含指令在组装过程中将成千上万行代码包含到程序中。如果你知道哪个头文件包含你需要使用的实际声明，你可以通过只包含必要的文件来加速编译过程（就像在*listing16-2.asm*中使用 MASM32 64 位包含文件那样）。

将 *masm64rt.inc* 引入到你的程序中还存在一个问题：*命名空间污染*。MASM32 包含文件会将成千上万的符号引入到你的程序中，因此有可能你想使用的符号已经在 MASM32 包含文件中被定义了（并且可能是用于与你想要的用途不同的目的）。如果你有一个 *file grep* 工具，这是一个搜索目录中文件并递归查找子目录中特定字符串的程序，你可以轻松找到你想在文件中使用的符号的所有出现位置，并将该符号的定义复制到你自己的源文件中（或者更好的是，复制到你专门为此目的创建的头文件中）。本章使用这种方法来处理许多示例程序。

## 16.3 Win32 API 和 Windows ABI

Win32 API 函数都遵循 Windows ABI 调用约定。这意味着对这些函数的调用可以修改所有易失寄存器（RAX、RCX、RDX、R8、R9、R10、R11 和 XMM0 到 XMM5），但必须保留非易失寄存器（这里没有列出的其他寄存器）。此外，API 调用通过 RDX、RCX、R8、R9（以及 XMM0 到 XMM3）传递参数，然后是栈；在进行 API 调用之前，栈必须进行 16 字节对齐。有关更多详细信息，请参见本书中关于 Windows ABI 的讨论。

## 16.4 构建独立的控制台应用程序

看一下前面章节中的（简化版）构建命令：^(2)

```
ml64 listing16-2.asm /link /subsystem:console /entry:asmMain
```

`/subsystem:console` 选项告诉链接器，除了可能创建的 GUI 窗口外，系统还必须为应用程序创建一个特殊窗口以显示控制台信息。如果你从 Windows 命令行运行该程序，它将使用已经打开的 *cmd.exe* 程序的控制台窗口。

## 16.5 构建独立的 GUI 应用程序

要创建一个纯 Windows GUI 应用程序而不打开控制台窗口，可以指定 `/subsystem:windows` 而不是 `/subsystem:console`。Listing 16-3 中的简单对话框应用程序是一个特别简单的 Windows 应用程序示例。它显示一个简单的对话框，然后在用户点击对话框中的确定按钮时退出。

```
; Listing 16-3

; Dialog box demonstration.

            include    listing16-3.inc
            includelib user32.lib

          ; include \masm32\include64\masm64rt.inc

            .data

msg         byte    "Dialog Box Demonstration",0
DBTitle     byte    "Dialog Box Title", 0

            .code

**********************************************************

; Here is the "asmMain" function.

            public  asmMain
asmMain     proc
            push    rbp
            mov     rbp, rsp
            sub     rsp, 56         ; Shadow storage
            and     rsp, -16

            xor     rcx, rcx        ; HWin = NULL
            lea     rdx, msg        ; Message to display
            lea     r8, DBTitle     ; Dialog box title
            mov     r9d, MB_OK      ; Has an "OK" button
            call    MessageBox

allDone:    leave
            ret     ; Returns to caller
asmMain     endp
            end
```

Listing 16-3：一个简单的对话框应用程序

这是 *listing16-3.inc* 包含文件：

```
; listing16-3.inc

; Header file entries extracted from MASM32 header
; files (placed here rather than including the 
; full MASM32 headers to avoid namespace pollution
; and speed up assemblies).

PPROC           TYPEDEF PTR PROC        ; For include file prototypes

MB_OK                                equ 0h

externdef __imp_MessageBoxA:PPROC
MessageBox equ <__imp_MessageBoxA>
```

以下是 Listing 16-3 中程序的构建命令：

```
C:\>**ml64 listing16-3.asm /link /subsystem:windows /entry:asmMain**
```

图 16-1 显示了 Listing 16-3 的运行时输出。

file:///Users/DisPater/Desktop/Hyde501089/Production/IndesignFiles/image_fi/501089c16/f16001.tiff

图 16-1：示例对话框输出

## 16.6 简要了解 MessageBox Windows API 函数

虽然在汇编语言中创建 GUI 应用程序超出了本书的范围，但 `MessageBox` 函数足够实用（即使在控制台应用程序中）值得特别提及。

`MessageBox` 函数有四个参数：

1.  RCX 窗口句柄。通常是 NULL（0），表示消息框是一个独立的对话框，未与任何特定窗口关联。

1.  RDX 消息指针。RDX 包含一个指向零终止字符串的指针，该字符串将在消息框的正文中显示。

1.  R8 窗口标题。R8 包含一个指向零终止字符串的指针，该字符串显示在消息框窗口的标题栏中。

1.  R9D 消息框类型。这是一个整数值，指定消息框中出现的按钮类型和其他图标。典型的值有：`MB_OK`、`MB_OKCANCEL`、`MB_ABORTRETRYIGNORE`、`MB_YESNOCANCEL`、`MB_YESNO` 和 `MB_RETRYCANCEL`。

`MessageBox` 函数返回一个整数值到 RAX，表示用户按下的按钮（如果指定了 `MB_OK`，那么当用户点击“确定”按钮时，消息框返回的就是这个值）。

## 16.7 Windows 文件 I/O

本书中大多数示例代码缺少一个关于文件 I/O 的讨论。尽管你可以轻松地使用 C 标准库函数来打开、读取、写入和关闭文件，但在本章中，使用文件 I/O 作为示例，涵盖这个缺失的细节似乎是合适的。

Win32 API 提供了许多有用的*文件 I/O*函数：读取和写入文件数据。本节描述了这些函数中的一小部分：

1.  `CreateFileA` 一个函数（尽管它的名字是这样），你用它来打开现有文件或创建新文件。

1.  `WriteFile` 一个函数，用来将数据写入文件。

1.  `ReadFile` 一个函数，用来从文件中读取数据。

1.  `CloseHandle` 一个函数，关闭文件并将任何缓存数据刷新到存储设备。

1.  `GetStdHandle` 一个你已经见过的函数，它返回标准输入或输出设备（标准输入、标准输出或标准错误）的句柄。

1.  `GetLastError` 一个函数，你可以用它来检索 Windows 错误代码，如果在执行这些函数中的任何一个时发生错误。

清单 16-4 演示了这些函数的使用，并创建了一些有用的过程来调用这些函数。请注意，这段代码相当长，因此我已将其拆分成更小的块，并在每个部分前面加上了个别的解释。

Win32 文件 I/O 函数都属于 *kernel32.lib* 库模块。因此，清单 16-4 使用 `includelib kernel32.lib` 语句，在构建阶段自动链接此库。为了加快汇编速度并减少命名空间污染，本程序并没有自动包含所有的 MASM32 等式文件（通过 `include \masm32\include64\masm64rt.inc` 语句）。相反，我从 MASM32 头文件中收集了所有必要的等式和其他定义，并将它们放在 *listing16-4.inc* 头文件中（稍后在本章中会看到）。最后，程序还包含了 *aoalib.inc* 头文件，只是为了使用该文件中定义的一些常量（如 `cr` 和 `nl`）：

```
; Listing 16-4 

; File I/O demonstration.

            include    listing16-4.inc
            include    aoalib.inc   ; To get some constants
            includelib kernel32.lib ; File I/O library

            .const
prompt      byte    "Enter (text) filename:", 0
badOpenMsg  byte    "Could not open file", cr, nl, 0

            .data

inHandle    dword   ?
inputLn     byte    256 dup (0)

fileBuffer  byte    4096 dup (0)
```

以下代码围绕每个文件 I/O 函数构建了 *包装代码*，以保留易失性寄存器值。这些函数使用以下宏定义来保存和恢复寄存器值：

```
 .code

rcxSave     textequ <[rbp - 8]>
rdxSave     textequ <[rbp - 16]>
r8Save      textequ <[rbp - 24]>
r9Save      textequ <[rbp - 32]>
r10Save     textequ <[rbp - 40]>
r11Save     textequ <[rbp - 48]>
xmm0Save    textequ <[rbp - 64]>
xmm1Save    textequ <[rbp - 80]>
xmm2Save    textequ <[rbp - 96]>
xmm3Save    textequ <[rbp - 112]>
xmm4Save    textequ <[rbp - 128]>
xmm5Save    textequ <[rbp - 144]>
var1        textequ <[rbp - 160]>

mkActRec    macro
            push    rbp
            mov     rbp, rsp
            sub     rsp, 256        ; Includes shadow storage
            and     rsp, -16        ; Align to 16 bytes
            mov     rcxSave, rcx
            mov     rdxSave, rdx
            mov     r8Save, r8
            mov     r9Save, r9
            mov     r10Save, r10
            mov     r11Save, r11
            movdqu  xmm0Save, xmm0
            movdqu  xmm1Save, xmm1
 movdqu  xmm2Save, xmm2
            movdqu  xmm3Save, xmm3
            movdqu  xmm4Save, xmm4
            movdqu  xmm5Save, xmm5
            endm

rstrActRec  macro
            mov     rcx, rcxSave
            mov     rdx, rdxSave
            mov     r8, r8Save 
            mov     r9, r9Save 
            mov     r10, r10Save
            mov     r11, r11Save
            movdqu  xmm0, xmm0Save
            movdqu  xmm1, xmm1Save
            movdqu  xmm2, xmm2Save
            movdqu  xmm3, xmm3Save
            movdqu  xmm4, xmm4Save
            movdqu  xmm5, xmm5Save
            leave
            endm
```

清单 16-4 中出现的第一个函数是 `getStdOutHandle`。这是一个包装函数，封装了 `__imp_GetStdHandle`，用于保留易失性寄存器并显式请求标准输出设备句柄。该函数返回标准输出设备句柄，保存在 RAX 寄存器中。在 `getStdOutHandle` 后面是类似的函数，用于获取标准错误句柄和标准输入句柄：

```
; getStdOutHandle - Returns stdout handle in RAX:

getStdOutHandle proc
                mkActRec
                mov     rcx, STD_OUTPUT_HANDLE
                call    __imp_GetStdHandle  ; Returns handle
                rstrActRec
                ret
getStdOutHandle endp

; getStdErrHandle - Returns stderr handle in RAX:

getStdErrHandle proc
                mkActRec
                mov     rcx, STD_ERROR_HANDLE
                call    __imp_GetStdHandle  ; Returns handle
                rstrActRec
                ret
getStdErrHandle endp

; getStdInHandle - Returns stdin handle in RAX:

getStdInHandle proc
               mkActRec
               mov     rcx, STD_INPUT_HANDLE
               call    __imp_GetStdHandle   ; Returns handle
               rstrActRec
               ret
getStdInHandle endp
```

现在考虑 `write` 函数的包装代码：

```
; write - Write data to a file handle.

; RAX - File handle.
; RSI - Pointer to buffer to write.
; RCX - Length of buffer to write.

; Returns:

; RAX - Number of bytes actually written
;       or -1 if there was an error.

write       proc
            mkActRec

            mov     rdx, rsi        ; Buffer address
            mov     r8, rcx         ; Buffer length
            lea     r9, var1        ; bytesWritten
            mov     rcx, rax        ; Handle
            xor     r10, r10        ; lpOverlapped is passed
            mov     [rsp+4*8], r10  ; on the stack
            call    __imp_WriteFile
            test    rax, rax        ; See if error
            mov     rax, var1       ; bytesWritten
            jnz     rtnBytsWrtn     ; If RAX was not zero
            mov     rax, -1         ; Return error status

rtnBytsWrtn:
            rstrActRec
            ret
write       endp
```

`write` 函数将数据从内存缓冲区写入由文件句柄指定的输出文件（如果你希望将数据写入控制台，它也可以是标准输出或标准错误句柄）。`write` 函数期望以下参数数据：

1.  RAX 文件句柄，指定写入目标。这通常是通过 `open` 或 `openNew` 函数（在程序稍后的部分）或 `getStdOutHandle` 和 `getStdErrHandle` 函数获得的句柄。

1.  RSI 包含要写入文件的数据的缓冲区地址。

1.  RCX 写入文件的数据字节数（来自缓冲区）。

此函数不遵循 Windows ABI 调用约定。虽然没有官方的 *汇编语言调用约定*，但许多汇编语言程序员倾向于使用 x86-64 字符串指令使用的相同寄存器。例如，源数据（缓冲区）通过 RSI（源索引寄存器）传递，计数（缓冲区大小）参数出现在 RCX 寄存器中。`write` 过程将数据移动到适当位置，以供调用 `__imp_WriteFile`（并设置额外的参数）。

`__imp_WriteFile` 函数是实际的 Win32 API 写入函数（技术上，`__imp_WriteFile` 是指向该函数的指针；调用指令是通过此指针的间接调用）。`__imp_WriteFile` 具有以下参数：

1.  RCX 文件句柄。

1.  RDX 缓冲区地址。

1.  R8 缓冲区大小（实际上是 R8D 中的 32 位）。

1.  R9 地址，指向一个 DWORD 变量，用于接收写入文件的字节数；如果写操作成功，该值将等于缓冲区大小。

1.  [rsp + 32] `lpOverlapped`值；将其设置为 NULL（0）。根据 Windows ABI，调用者通过栈传递第四个参数之后的所有参数，为前四个参数留出空间（影子参数）。

从`__imp_WriteFile`返回时，如果写入成功，RAX 包含非零值（true）；如果出现错误，RAX 包含零（false）。如果发生错误，可以调用 Win32 的`GetLastError`函数来获取错误代码。

请注意，`write`函数将写入文件的字节数返回在 RAX 寄存器中。如果发生错误，`write`在 RAX 寄存器中返回`-1`。

接下来是`puts`和`newLn`函数：

```
; puts - Outputs a zero-terminated string to standard output device.

; RSI - Address of string to print to standard output.

            .data
stdOutHnd   qword   0
hasSOHndl   byte    0

            .code
puts        proc
            push    rax
            push    rcx
            cmp     hasSOHndl, 0
            jne     hasHandle

 call    getStdOutHandle
            mov     stdOutHnd, rax
            mov     hasSOHndl, 1

; Compute the length of the string:

hasHandle:  mov     rcx, -1
lenLp:      inc     rcx
            cmp     byte ptr [rsi][rcx * 1], 0
            jne     lenLp

            mov     rax, stdOutHnd
            call    write

            pop     rcx
            pop     rax
            ret
puts        endp

; newLn - Outputs a newline sequence to the standard output device:

newlnSeq    byte    cr, nl

newLn       proc
            push    rax
            push    rcx
            push    rsi
            cmp     hasSOHndl, 0
            jne     hasHandle

            call    getStdOutHandle
            mov     stdOutHnd, rax
            mov     hasSOHndl, 1

hasHandle:  lea     rsi, newlnSeq
            mov     rcx, 2
            mov     rax, stdOutHnd
            call    write

            pop     rsi
            pop     rcx
            pop     rax
            ret
newLn       endp
```

`puts`和`newLn`过程将字符串写入标准输出设备。`puts`函数写入一个以零终止的字符串，其地址通过 RSI 寄存器传递。`newLn`函数写入一个换行序列（回车和换行符）到标准输出设备。

这两个函数有一个小优化：它们只调用`getStdOutHandle`一次来获取标准输出设备句柄。在第一次调用这两个函数中的任何一个时，它们调用`getStdOutHandle`并缓存结果（在`stdOutHnd`变量中），并设置标志（`hasSOHndl`），指示缓存的值有效。之后，这些函数使用缓存值，而不是不断调用`getStdOutHandle`来检索标准输出设备句柄。

`write`函数需要一个缓冲区长度；它不适用于以零终止的字符串。因此，`puts`函数在调用`write`之前必须显式确定零终止字符串的长度。`newLn`函数不需要这样做，因为它知道回车换行序列的长度（两个字符）。

在清单 16-4 中的下一个函数是`read`函数的包装器：

```
; read - Read data from a file handle.

; EAX - File handle.
; RDI - Pointer to buffer receive data.
; ECX - Length of data to read.

; Returns:

; RAX - Number of bytes actually read
;       or -1 if there was an error.

read        proc
            mkActRec

            mov     rdx, rdi        ; Buffer address
            mov     r8, rcx         ; Buffer length
            lea     r9, var1        ; bytesRead
            mov     rcx, rax        ; Handle
            xor     r10, r10        ; lpOverlapped is passed
            mov     [rsp+4*8], r10  ; on the stack
            call    __imp_ReadFile
            test    rax, rax        ; See if error
            mov     rax, var1       ; bytesRead
            jnz     rtnBytsRead     ; If RAX was not zero
            mov     rax, -1         ; Return error status

rtnBytsRead:
            rstrActRec
            ret
read        endp
```

`read`函数是`write`函数的输入对应函数。参数相似（但请注意，`read`使用 RDI 作为*目标地址*来传递缓冲区参数）：

1.  RAX 文件句柄。

1.  RDI 目标缓冲区，用于存储从文件读取的数据。

1.  RCX 从文件中读取的字节数。

`read`函数是对 Win32 API `__imp_ReadFile`函数的包装，具有以下参数：

1.  RCX 文件句柄。

1.  RDX 文件缓冲区地址。

1.  R8 要读取的字节数。

1.  R9 地址，指向一个 DWORD 变量，用于接收实际读取的字节数。

1.  [rsp + 32] 重叠操作；应为 NULL（0）。根据 Windows ABI，调用者通过栈传递第四个参数之后的所有参数，为前四个参数留出空间（影子参数）。

`read` 函数如果在读取操作期间发生错误，会在 RAX 中返回 `-1`。否则，它返回实际从文件中读取的字节数。如果读取操作到达文件结尾（EOF），此值可能会小于请求的读取量。返回值为 `0` 通常表示已到达文件末尾（EOF）。

`open` 函数用于打开一个现有的文件进行读取、写入或两者兼有。它是 Windows `CreateFileA` API 调用的封装函数：

```
; open - Open existing file for reading or writing.

; RSI - Pointer to filename string (zero-terminated).
; RAX - File access flags.
;       (GENERIC_READ, GENERIC_WRITE, or
;       "GENERIC_READ + GENERIC_WRITE")

; Returns:

; RAX - Handle of open file (or INVALID_HANDLE_VALUE if there
;       was an error opening the file).

open        proc
            mkActRec

            mov     rcx, rsi               ; Filename
            mov     rdx, rax               ; Read and write access
            xor     r8, r8                 ; Exclusive access
            xor     r9, r9                 ; No special security
            mov     r10, OPEN_EXISTING     ; Open an existing file
            mov     [rsp + 4 * 8], r10     
            mov     r10, FILE_ATTRIBUTE_NORMAL
            mov     [rsp + 5 * 8], r10
            mov     [rsp + 6 * 8], r9      ; NULL template file
            call    __imp_CreateFileA
            rstrActRec
            ret
open        endp
```

`open` 过程有两个参数：

1.  RSI 是指向包含要打开文件的文件名的零终止字符串的指针。

1.  RAX 是一组文件访问标志。通常是常量 `GENERIC_READ`（用于打开文件以进行读取）、`GENERIC_WRITE`（用于打开文件以进行写入）或 `GENERIC_READ + GENERIC_WRITE`（用于同时打开文件进行读取和写入）。

`open` 函数在设置好适当的参数后调用 Windows `CreateFileA` 函数。`CreateFileA` 中的 `A` 后缀代表 *ASCII*。这个函数期望调用者传递一个 ASCII 文件名。另一个函数 `CreateFileW` 则期望传递 Unicode 文件名，且编码为 UTF-16。Windows 内部使用 Unicode 文件名；当调用 `CreateFileA` 时，它会将 ASCII 文件名转换为 Unicode，然后调用 `CreateFileW`。`open` 函数坚持使用 ASCII 字符。

`CreateFileA` 函数具有以下参数：

1.  RCX 是指向零终止的（ASCII）字符串，包含要打开文件的文件名。

1.  RDX 读取和写入访问标志（`GENERIC_READ` 和 `GENERIC_WRITE`）。

1.  R8 共享模式标志（`0` 表示独占访问）。控制当前进程打开文件时，是否允许其他进程访问该文件。可能的标志值有 `FILE_SHARE_READ`、`FILE_SHARE_WRITE` 和 `FILE_SHARE_DELETE`（或它们的组合）。

1.  R9 是指向安全描述符的指针。`open` 函数没有指定任何特殊的安全性，它只是将 NULL (0) 作为该参数传递。

1.  [rsp + 32] 该参数包含创建处置标志。`open` 函数打开一个现有的文件，因此它传递 `OPEN_EXISTING`。其他可能的值有 `CREATE_ALWAYS`、`CREATE_NEW`、`OPEN_ALWAYS`、`OPEN_EXISTING` 或 `TRUNCATE_EXISTING`。`OPEN_EXISTING` 要求文件必须存在，否则会返回打开错误。作为第五个参数，该值通过堆栈传递（在第五个 64 位位置）。

1.  [rsp + 40] 该参数包含文件属性。此函数仅使用 `FILE_ATTRIBUTE_NORMAL` 属性（例如，不是只读的）。

1.  [rsp + 48] 该参数是指向文件模板句柄的指针。`open` 函数不使用文件模板，因此它在该参数中传递 NULL (0)。

`open` 函数返回一个文件句柄，该句柄存储在 RAX 寄存器中。如果发生错误，函数会在 RAX 中返回 `INVALID_HANDLE_VALUE`。

`openNew` 函数也是对 `CreateFileA` 函数的封装：

```
; openNew - Creates a new file and opens it for writing.

; RSI - Pointer to filename string (zero-terminated).

; Returns:

; RAX - Handle of open file (or INVALID_HANDLE_VALUE if there
;       was an error opening the file).

openNew     proc
            mkActRec

            mov     rcx, rsi                         ; Filename
            mov     rdx, GENERIC_WRITE+GENERIC_WRITE ; Access
 xor     r8, r8                           ; Exclusive access
            xor     r9, r9                           ; No security
            mov     r10, CREATE_ALWAYS               ; Open a new file
            mov     [rsp + 4 * 8], r10 
            mov     r10, FILE_ATTRIBUTE_NORMAL
            mov     [rsp + 5 * 8], r10
            mov     [rsp + 6 * 8], r9                ; NULL template
            call    __imp_CreateFileA
            rstrActRec
            ret
openNew     endp
```

`openNew`在磁盘上创建一个新的（空的）文件。如果文件之前已存在，`openNew`会在打开新文件之前删除它。这个函数与前面的`open`函数几乎相同，只有以下两个区别：

+   调用者不通过 RAX 寄存器传递文件访问标志。文件访问始终假定为`GENERIC_WRITE`。

+   该函数传递`CREATE_ALWAYS`创建方式标志给`CreateFileA`，而不是`OPEN_EXISTING`。

`closeHandle`函数是对 Windows `CloseHandle`函数的一个简单封装。你将要关闭的文件句柄传递给 RAX 寄存器。该函数如果发生错误，则返回 RAX 中的`0`，如果文件关闭操作成功，则返回一个非零文件句柄。这个封装函数的唯一目的是在调用 Windows `CloseHandle`函数时保留所有易失性寄存器：

```
; closeHandle - Closes a file specified by a file handle.

; RAX - Handle of file to close.

closeHandle proc
            mkActRec

            call    __imp_CloseHandle

            rstrActRec
            ret
closeHandle endp
```

尽管该程序没有显式地使用`getLastError`，但它确实提供了一个封装`getLastError`函数的函数（只是为了展示它是如何写的）。每当此程序中的 Windows 函数返回错误指示时，你必须调用`getLastError`来获取实际的错误代码。该函数没有输入参数。它返回在 RAX 寄存器中生成的最后一个 Windows 错误代码。

在函数返回错误指示后，立即调用`getLastError`非常重要。如果在错误和错误代码检索之间调用了其他 Windows 函数，这些中介调用将重置最后的错误代码值。

和`closeHandle`函数一样，`getLastError`过程是对 Windows `GetLastError`函数的一个非常简单的封装，它在调用过程中保留了易失性寄存器的值：

```
; getLastError - Returns the error code of the last Windows error.

; Returns:

; RAX - Error code.

getLastError proc
             mkActRec
             call   __imp_GetLastError
             rstrActRec
             ret
getLastError endp
```

`stdin_read`是对`read`函数的一个简单封装函数，它从标准输入设备读取数据（而不是从另一个设备上的文件读取数据）：

```
; stdin_read - Reads data from the standard input.

; RDI - Buffer to receive data.
; RCX - Buffer count (note that data input will
;       stop on a newline character if that
;       comes along before RCX characters have
;       been read).

; Returns:

; RAX - -1 if error, bytes read if successful.

stdin_read  proc
            .data
hasStdInHnd byte    0
stdInHnd    qword   0
            .code
            mkActRec
            cmp     hasStdInHnd, 0
            jne     hasHandle

            call    getStdInHandle
            mov     stdInHnd, rax
            mov     hasStdInHnd, 1

hasHandle:  mov     rax, stdInHnd   ; Handle
            call    read

            rstrActRec
            ret
stdin_read  endp
```

`stdin_read`类似于`puts`（和`newLn`）过程，因为它在第一次调用时缓存了标准输入句柄，并在随后的调用中使用该缓存值。需要注意的是，`stdin_read`不会（直接）保留易失性寄存器。该函数没有直接调用任何 Windows 函数，因此不需要保留易失性寄存器（`stdin_read`调用了`read`函数，后者会保留易失性寄存器）。`stdin_read`函数有以下参数：

1.  RDI 指向目标缓冲区，该缓冲区将接收从标准输入设备读取的字符。

1.  RCX 缓冲区大小（最大读取字节数）。

此函数返回实际读取的字节数，存储在 RAX 寄存器中。这个值可能小于 RCX 中传递的值。如果用户按下回车键，该函数会立即返回。此函数不会为从标准输入设备读取的字符串添加零终止符。请使用 RAX 寄存器中的值来确定字符串的长度。如果该函数因为用户在标准输入设备上按下回车键而返回，那么该回车符将出现在缓冲区中。

`stdin_getc` 函数从标准输入设备读取一个字符，并将该字符返回到 AL 寄存器：

```
; stdin_getc - Reads a single character from the standard input.
;              Returns character in AL register.

stdin_getc  proc
            push    rdi
            push    rcx
            sub     rsp, 8

            mov     rdi, rsp
            mov     rcx, 1
            call    stdin_read
            test    eax, eax        ; Error on read?
            jz      getcErr
            movzx   rax, byte ptr [rsp]

getcErr:    add     rsp, 8
            pop     rcx
            pop     rdi 
            ret
stdin_getc  endp
```

`readLn` 函数从标准输入设备读取一串字符，并将其放入调用者指定的缓冲区。参数如下：

1.  RDI 缓冲区的地址。

1.  RCX 最大缓冲区大小。（`readLn` 允许用户输入最多 RCX - 1 个字符。）

此函数将在用户输入的字符串末尾添加一个零终止字节。此外，它会去除行末的回车符（或换行符或换行符）。它将字符数返回在 RAX 寄存器中（不包括回车键）：

```
; readLn - Reads a line of text from the user.
;          Automatically processes backspace characters
;          (deleting previous characters, as appropriate).
;          Line returned from function is zero-terminated
;          and does not include the ENTER key code (carriage
;          return) or line feed.

; RDI - Buffer to place line of text read from user.
; RCX - Maximum buffer length.

; Returns:

; RAX - Number of characters read from the user
;       (does not include ENTER key).

readLn      proc
            push    rbx

            xor     rbx, rbx           ; Character count
            test    rcx, rcx           ; Allowable buffer is 0?
            je      exitRdLn
            dec     rcx                ; Leave room for 0 byte
readLp:
            call    stdin_getc         ; Read 1 char from stdin
            test    eax, eax           ; Treat error like ENTER
            jz      lineDone
            cmp     al, cr             ; Check for ENTER key
            je      lineDone
            cmp     al, nl             ; Check for newline code
            je      lineDone
            cmp     al, bs             ; Handle backspace character
            jne     addChar

; If a backspace character came along, remove the previous
; character from the input buffer (assuming there is a
; previous character).

            test    rbx, rbx           ; Ignore BS character if no
            jz      readLp             ; chars in the buffer
            dec     rbx
            jmp     readLp

; If a normal character (that we return to the caller),
; then add the character to the buffer if there is
; room for it (ignore the character if the buffer is full).

addChar:    cmp     ebx, ecx           ; See if we're at the
            jae     readLp             ; end of the buffer
            mov     [rdi][rbx * 1], al ; Save char to buffer
 inc     rbx
            jmp     readLp

; When the user presses ENTER (or the line feed) key
; during input, come down here and zero-terminate the string.

lineDone:   mov     byte ptr [rdi][rbx * 1], 0 

exitRdLn:   mov     rax, rbx        ; Return char cnt in RAX
            pop     rbx
            ret
readLn      endp
```

这是 列表 16-4 的主程序，它从用户处读取文件名，打开该文件，读取文件数据，并将数据显示到标准输出设备：

```
**********************************************************

; Here is the "asmMain" function.

            public  asmMain
asmMain     proc
            push    rbx
            push    rsi
            push    rdi
            push    rbp
            mov     rbp, rsp
            sub     rsp, 64         ; Shadow storage
            and     rsp, -16

; Get a filename from the user:

            lea     rsi, prompt
            call    puts

            lea     rdi, inputLn
            mov     rcx, lengthof inputLn
            call    readLn

; Open the file, read its contents, and display
; the contents to the standard output device:

            lea     rsi, inputLn
            mov     rax, GENERIC_READ
            call    open

            cmp     eax, INVALID_HANDLE_VALUE
            je      badOpen

            mov     inHandle, eax

; Read the file 4096 bytes at a time:

readLoop:   mov     eax, inHandle
            lea     rdi, fileBuffer
            mov     ecx, lengthof fileBuffer
            call    read
            test    eax, eax        ; EOF?
            jz      allDone
            mov     rcx, rax        ; Bytes to write

            call    getStdOutHandle
            lea     rsi, fileBuffer
            call    write
            jmp     readLoop

badOpen:    lea     rsi, badOpenMsg
            call    puts

allDone:    mov     eax, inHandle
            call    closeHandle

            leave
            pop     rdi
            pop     rsi
            pop     rbx
            ret     ; Returns to caller
asmMain     endp
            end
```

列表 16-4：文件 I/O 演示程序

这是 列表 16-4 的构建命令和示例输出：

```
C:\>**nmake /nologo /f listing16-4.mak**
        ml64 /nologo listing16-4.asm  /link /subsystem:console /entry:asmMain
 Assembling: listing16-4.asm
Microsoft (R) Incremental Linker Version 14.15.26730.0
Copyright (C) Microsoft Corporation.  All rights reserved.

/OUT:listing16-4.exe
listing16-4.obj
/subsystem:console
/entry:asmMain

C:\>**listing16-4**
Enter (text) filename:listing16-4.mak
listing16-4.exe: listing16-4.obj listing16-4.asm
        ml64 /nologo listing16-4.asm \
                /link /subsystem:console /entry:asmMain
```

这是 *listing16-4.inc* 包含文件：

```
; listing16-4.inc

; Header file entries extracted from MASM32 header
; files (placed here rather than including the 
; entire set of MASM32 headers to avoid namespace 
; pollution and speed up assemblies).

STD_INPUT_HANDLE                     equ -10
STD_OUTPUT_HANDLE                    equ -11
STD_ERROR_HANDLE                     equ -12
CREATE_NEW                           equ 1
CREATE_ALWAYS                        equ 2
OPEN_EXISTING                        equ 3
OPEN_ALWAYS                          equ 4
FILE_ATTRIBUTE_READONLY              equ 1h
FILE_ATTRIBUTE_HIDDEN                equ 2h
FILE_ATTRIBUTE_SYSTEM                equ 4h
FILE_ATTRIBUTE_DIRECTORY             equ 10h
FILE_ATTRIBUTE_ARCHIVE               equ 20h
FILE_ATTRIBUTE_NORMAL                equ 80h
FILE_ATTRIBUTE_TEMPORARY             equ 100h
FILE_ATTRIBUTE_COMPRESSED            equ 800h
FILE_SHARE_READ                      equ 1h
FILE_SHARE_WRITE                     equ 2h
GENERIC_READ                         equ 80000000h
GENERIC_WRITE                        equ 40000000h
GENERIC_EXECUTE                      equ 20000000h
GENERIC_ALL                          equ 10000000h
INVALID_HANDLE_VALUE                 equ -1

PPROC           TYPEDEF PTR PROC        ; For include file prototypes

externdef __imp_GetStdHandle:PPROC
externdef __imp_WriteFile:PPROC
externdef __imp_ReadFile:PPROC
externdef __imp_CreateFileA:PPROC
externdef __imp_CloseHandle:PPROC
externdef __imp_GetLastError:PPROC
```

这是 *listing16-4.mak* makefile 文件：

```
listing16-4.exe: listing16-4.obj listing16-4.asm
    ml64 /nologo listing16-4.asm \
        /link /subsystem:console /entry:asmMain
```

## 16.8 Windows 应用程序

本章仅展示了在 Windows 下编写纯汇编语言应用程序时可能实现的一些功能。*kernel32.lib* 库提供了数百个可供调用的函数，涵盖了多个不同的主题领域，如操作文件系统（例如，删除文件、查找目录中的文件名、切换目录）、创建线程并进行同步、处理环境字符串、分配和释放内存、操作 Windows 注册表、使程序暂停一定时间、等待事件发生等等。

*kernel32.lib* 库只是 Win32 API 中的一个库。*gdi32.lib* 库包含了创建在 Windows 下运行的 GUI 应用程序所需的大部分函数。创建此类应用程序远超本书的范围，但如果你想创建独立的 Windows GUI 应用程序，你需要深入了解这个库。以下的“获取更多信息”部分提供了互联网资源链接，如果你有兴趣用汇编语言创建独立的 Windows GUI 应用程序，可以参考。

## 16.9 获取更多信息

如果你想编写在 Windows 上运行的独立 64 位汇编语言程序，你的第一站应该是 [`www.masm32.com/`](https://www.masm32.com/)。虽然这个网站主要致力于创建在 Windows 上运行的 32 位汇编语言程序，但它也为 64 位程序员提供了大量的信息。更重要的是，这个网站包含了你需要从 64 位汇编语言程序访问 Win32 API 的头文件。

如果你打算认真编写基于 Win32 API 的 Windows 汇编语言应用程序，Charles Petzold 的 *Programming Windows*（第五版，Microsoft，1998 年）是一本绝对必要购买的书。这本书已经很老了（不要购买新版的 C# 和 XAML 版本），你可能需要购买二手书。它是为 C 程序员（而非汇编程序员）编写的，但如果你了解 Windows ABI（你现在应该已经知道了），将所有的 C 调用翻译成汇编语言并不难。尽管关于 Win32 API 的很多信息可以在网上找到（例如在 MASM32 网站上），但将所有信息集成在一本（非常大的！）书中是必不可少的。

网络上另一个关于 Win32 API 调用的好资源是软件分析师 Geoff Chappell 的 Win32 编程页面（[`www.geoffchappell.com/studies/windows/win32/`](https://www.geoffchappell.com/studies/windows/win32/)）。

Iczelion 教程是编写 x86 汇编语言 Windows 程序的最初标准。尽管它们最初是为 32 位 x86 汇编语言编写的，但已经有多个将该代码翻译成 64 位汇编语言的版本，例如：[`masm32.com/board/index.php?topic=4190.0/`](http://masm32.com/board/index.php?topic=4190.0/)。

HLA 标准库和示例（可以在 [`www.randallhyde.com/`](https://www.randallhyde.com/) 找到）包含了大量的 Windows 代码和 API 函数调用。尽管这些代码都是 32 位的，但将它们转换为 64 位的 MASM 代码非常容易。

## 16.10 自我测试

1.  告诉 MASM 你正在构建控制台应用程序的链接器命令行选项是什么？

1.  你应该访问哪个网站来获取 Win32 编程信息？

1.  将 *\masm32\include64\masm64rt.inc* 包含在所有汇编语言源文件中的主要缺点是什么？

1.  哪个链接器命令行选项允许你指定汇编语言主程序的名称？

1.  允许你弹出对话框的 Win32 API 函数的名称是什么？

1.  什么是包装代码？

1.  你将使用哪个 Win32 API 函数来打开一个现有文件？

1.  你使用哪个 Win32 API 函数来检索最后的 Windows 错误代码？
