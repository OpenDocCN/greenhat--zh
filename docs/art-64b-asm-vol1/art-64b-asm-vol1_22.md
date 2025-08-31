# 第十九章：C

安装和使用 Visual Studio

![](img/chapterart.png)

本书使用的 Microsoft 宏汇编器（MASM）、Microsoft C++ 编译器、Microsoft 链接器及其他工具，都可以在 Microsoft Visual Studio 包中找到。在写这篇文章时，你可以在 [`visualstudio.microsoft.com/vs/community/`](https://visualstudio.microsoft.com/vs/community/) 下载 Windows 版本的 Visual Studio Community 版。当然，网址会随时间变化。通过网络搜索 *Microsoft Visual Studio download* 应该能引导你到合适的页面。

## C.1 安装 Visual Studio Community

下载 Visual Studio Community 版后，运行安装程序。由于 Microsoft 以其即使在发生小幅更新时也会完全更改程序的用户界面而闻名，因此本附录不提供逐步的操作指导。这里提供的任何指引在你尝试运行时可能已经过时。不过，最重要的是确保你下载并安装 Microsoft Visual C++ 桌面工具。

## C.2 为 MASM 创建命令行提示符

为了使用 Microsoft Visual C++（MSVC）编译器和 MASM，我们需要通过使用 Visual Studio 提供的批处理文件来初始化环境，然后保持命令行解释器（CLI）打开，以便我们可以构建和运行程序。我们有两个选择：使用 Visual Studio 安装程序创建的环境，或者创建一个自定义环境。

在写这篇文章时，Visual Studio 2019 安装程序创建了各种命令行界面（CLI）环境：

+   VS 2019 的开发者命令提示符

+   VS 2019 的开发者 PowerShell

+   x64 原生工具命令提示符（VS 2019）

+   x64_x86 跨平台工具命令提示符（VS 2019）

+   x86 原生工具命令提示符（VS 2019）

+   x86_x64 跨平台工具命令提示符（VS 2019）

你可以通过点击**开始**（Windows 图标）在 Windows 任务栏上，然后导航到并点击**Visual Studio 2019**文件夹来找到这些工具。*x86* 指的是 32 位版本，而 *x64* 指的是 64 位版本的 Windows。

开发者命令提示符、开发者 PowerShell、x86 原生工具和 x64_x86 跨平台工具是面向 Windows 的 32 位版本，因此它们超出了本书的范围。x86_x64 跨平台工具面向 64 位 Windows，但环境中的工具本身是 32 位的。基本上，这些是为运行 32 位版本 Windows 的用户准备的工具。x64 原生工具是为面向和运行 64 位版本 Windows 的用户准备的。今天 32 位版本的 Windows 很少见，因此我们没有在 x86_x64 跨平台工具下使用或测试本书的代码。理论上，它应该能够组装和编译 64 位代码，但我们无法在这个 32 位环境中运行它。

我们使用并测试的是运行在 64 位 Windows 下的 x64 原生工具。如果你右键点击**x64 原生工具**，你可以将其固定到开始菜单，或者选择**更多**，你可以将其固定到任务栏。

或者，你可以创建自定义环境，我们现在将介绍这个过程。我们将通过以下步骤创建一个指向 MASM 命令行提示符的快捷方式：

1.  找到名为*vcvars64.bat*的批处理文件（或类似文件）。如果找不到*vcvars64.bat*，可以尝试*vcvarsall.bat*。在编写本章时（使用 Visual Studio 2019），我找到了*vcvars64.bat*文件，路径为：*C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Auxiliary\Build\*。

1.  创建文件的快捷方式（通过在 Windows 资源管理器中右键点击它，并从弹出菜单中选择**创建快捷方式**）。将此快捷方式移到 Windows 桌面上，并将其重命名为*VSCmdLine*。

1.  右键点击桌面上的快捷方式图标，然后点击**属性**▶**快捷方式**。找到包含*vcvars64.bat*文件路径的目标文本框；例如：

    ```
    "C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Auxiliary\Build\vcvars64.bat"
    ```

    在此路径前添加前缀`cmd /k`：

    ```
    **cmd /k** "C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Auxiliary\Build\vcvars64.bat"
    ```

    `cmd`命令是 Microsoft 的*cmd.exe*命令行解释器。`/k`选项告诉*cmd.exe*执行后续的命令（即*vcvars64.bat*文件），并在命令执行完成后保持窗口打开。现在，当你双击桌面上的快捷方式图标时，它将初始化所有环境变量，并保持命令窗口打开，这样你就可以从命令行执行 Visual Studio 工具（例如 MASM 和 MSVC）。

    如果你找不到*vcvars64.bat*，但有*vcvarsall.bat*，也在命令行末尾添加`x64`：

    ```
    cmd /k "C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Auxiliary\Build\vcvarsall.bat" **x64**
    ```

1.  在关闭快捷方式的属性对话框之前，将**起始位置**文本框修改为`C:\`，或者其他你通常在开始使用 Visual Studio 命令行工具时工作的目录。

    双击桌面上的快捷方式图标；你应该看到一个命令窗口，里面有如下文本：

    ```
    **********************************************************************
    ** Visual Studio 2019 Developer Command Prompt v16.9.0
    ** Copyright (c) 2019 Microsoft Corporation
    **********************************************************************
    [vcvarsall.bat] Environment initialized for: 'x64'
    ```

    从命令行输入`ml64`。这应该会产生类似如下的输出：

    ```
    C:\>`ml64`
    Microsoft (R) Macro Assembler (x64) Version 14.28.29910.0
    Copyright (C) Microsoft Corporation.  All rights reserved.

    usage: ML64 [options] filelist [/link linkoptions]
    Run "ML64 /help" or "ML64 /?" for more info
    ```

    尽管 MASM 抱怨你没有提供要编译的文件名，但你收到此消息意味着*ml64.exe*已经在执行路径中，因此系统已正确设置环境变量，使你能够运行 Microsoft 宏汇编器。

1.  作为最终测试，执行`cl`命令以验证是否能够运行 MSVC。你应该会看到类似如下的输出：

    ```
    C:\>`cl`
    Microsoft (R) C/C++ Optimizing Compiler Version 19.28.29910 for x64
    Copyright (C) Microsoft Corporation.  All rights reserved.

    usage: cl [option...] filename... [/link linkoption...]
    ```

1.  最后，做一次最终检查，在 Windows 开始菜单中找到 Visual Studio 应用程序。点击它并验证是否能够启动 Visual Studio IDE。如果你愿意，可以复制此快捷方式并将其放到桌面上，以便通过双击快捷方式图标启动 Visual Studio。

## C.3 编辑、汇编和运行 MASM 源文件

你将使用某种文本编辑器来创建和维护 MASM 汇编语言源文件。如果你还不熟悉 Visual Studio，并且希望使用一个更容易学习和使用的环境，可以考虑下载免费的 Notepad++ 文本编辑器应用程序。Notepad++ 对 MASM 提供了出色的支持，速度快，且易于学习和使用。无论你选择哪种文本编辑器（我使用一款名为 CodeWright 的商业产品），第一步是创建一个简单的汇编语言源文件。

MASM 要求所有源文件都必须有 *.asm* 后缀，所以用编辑器创建文件 *hw64.asm* 并输入以下内容：

```
includelib kernel32.lib

        extrn __imp_GetStdHandle:proc
        extrn __imp_WriteFile:proc

        .CODE
hwStr   byte    "Hello World!"
hwLen   =       $-hwStr

main    PROC

; On entry, stack is aligned at 8 mod 16\. Setting aside 8
; bytes for "bytesWritten" ensures that calls in main have
; their stack aligned to 16 bytes (8 mod 16 inside function).

 lea     rbx, hwStr
        sub     rsp, 8
        mov     rdi, rsp        ; Hold # of bytes written here

; Note: must set aside 32 bytes (20h) for shadow registers for
; parameters (just do this once for all functions). 
; Also, WriteFile has a 5th argument (which is NULL), 
; so we must set aside 8 bytes to hold that pointer (and
; initialize it to zero). Finally, the stack must always be 
; 16-byte-aligned, so reserve another 8 bytes of storage
; to ensure this.

; Shadow storage for args (always 30h bytes).

        sub     rsp, 030h 

; Handle = GetStdHandle(-11);
; Single argument passed in ECX.
; Handle returned in RAX.

        mov     rcx, -11        ; STD_OUTPUT
        call    qword ptr __imp_GetStdHandle 

; WriteFile(handle, "Hello World!", 12, &bytesWritten, NULL);
; Zero out (set to NULL) "LPOverlapped" argument:

        mov     qword ptr [rsp + 4 * 8], 0  ; 5th argument on stack

        mov     r9, rdi         ; Address of "bytesWritten" in R9
        mov     r8d, hwLen      ; Length of string to write in R8D 
        lea     rdx, hwStr      ; Ptr to string data in RDX
        mov     rcx, rax        ; File handle passed in RCX
        call    qword ptr __imp_WriteFile
        add     rsp, 38h
        ret
main    ENDP
        END
```

这个（纯）汇编语言程序没有提供解释。书中的各个章节会解释机器指令。

回看源代码，你会看到第一行如下：

```
includelib kernel32.lib
```

*kernel32.lib* 是一个 Windows 库，其中包含了此汇编语言程序使用的 `GetStdHandle` 和 `WriteFile` 函数。Visual Studio 安装包中包含了此文件，并且 *vcvars64.bat* 文件应该会将它放入包含路径中，以便链接器能够找到它。如果你在汇编和链接程序（在下一步中）时遇到问题，只需复制此文件（无论你在 Visual Studio 安装中找到它的位置），并将该副本包含在你构建 *hw64.asm* 文件的目录中。

要编译（组装）这个文件，打开命令窗口（即之前创建的快捷方式）以获取命令提示符。然后输入以下命令：

```
ml64 hw64.asm /link /subsystem:console /entry:main
```

假设你没有输入错误，命令窗口应输出类似以下内容：

```
C:\MASM64>**ml64 hw64.asm /link /subsystem:console /entry:main**
Microsoft (R) Macro Assembler (x64) Version 14.28.29910.0
Copyright (C) Microsoft Corporation.  All rights reserved.

 Assembling: hw64.asm
Microsoft (R) Incremental Linker Version 14.28.29910.0
Copyright (C) Microsoft Corporation.  All rights reserved.

/OUT:hw64.exe
hw64.obj
/subsystem:console
/entry:main
```

你可以通过在命令行提示符下输入命令`hw64`来运行此汇编产生的 *hw64.exe* 输出文件。输出应如下所示：

```
C:\MASM64>**hw64**
Hello World!
```
