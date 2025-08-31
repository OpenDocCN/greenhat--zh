# 前言

![](img/chapterart.png)

本书是 30 年工作的结晶。这本书的最早版本是我为我的 Cal Poly Pomona 和 UC Riverside 的学生复印的笔记，标题为“如何使用 8088 汇编语言编程 IBM PC”。我得到了许多学生的反馈，以及我一个好朋友 Mary Philips 的建议，这些帮助稍微润色了一下内容。Bill Pollock 将那个早期版本从互联网的遗忘角落拯救了出来，在 Karol Jurado 的帮助下，*《汇编语言的艺术》*的第一版在 2003 年得以问世。

数千名读者（以及他们的建议），以及 Bill Pollock、Alison Peterson、Ansel Staton、Riley Hoffman、Megan Dunchak、Linda Recktenwald、Susan Glinert Stevens 和 Nancy Bell（来自 No Starch Press）的贡献，以及 Nathan Baker 的技术审查，促成了这本书的第二版在 2010 年问世。

十年后，*《汇编语言的艺术》*（或者我称之为*AoA*）因其依赖于已经 35 年的 32 位 Intel x86 设计而逐渐失去人气。今天，如果有人想学习 80x86 汇编语言，他们会想要在更新的 x86-64 CPU 上学习 64 位汇编。因此，在 2020 年初，我开始了将旧版 32 位*AoA*（基于使用高级汇编器，或 HLA）转向 64 位的过程，采用了 Microsoft Macro Assembler（MASM）。

当我第一次开始这个项目时，我以为只需要将几个 HLA 程序翻译成 MASM，稍微修改一些文本，就能轻松完成*64 位汇编的艺术*的翻译工作。我错了。由于 No Starch Press 希望在可读性和理解上做出突破，并且 Tony Tribelli 在对本书每一行文本和代码进行技术审查时做出了令人难以置信的工作，这个项目变得像从头开始写一本新书一样繁重。没关系，我认为你会真正感激这本书中所付出的努力。

## 关于本书中的源代码说明

本书中展示了大量的 x86-64 汇编语言（以及 C/C++）源代码。通常，源代码有三种形式：代码片段、单一的汇编语言过程或函数，以及完整的程序。

*代码片段*是程序的片段；它们不是独立的，不能使用 MASM（或在 C/C++源代码的情况下使用 C++编译器）进行编译（汇编）。代码片段的目的是阐明某个要点或提供编程技巧的小示例。以下是你将在本书中找到的一个典型代码片段示例：

```
someConst = 5
   .
   .
   .
mov eax, someConst
```

垂直省略号（. . .）表示可以在其位置出现的任意代码（并非所有的代码片段都使用省略号，但指出这一点是有意义的）。

*汇编语言过程*也不是独立的代码。尽管你可以组装本书中出现的许多汇编语言过程（只需将代码从书中复制到编辑器中，然后运行 MASM 来处理生成的文本文件），但它们不会自行执行。代码片段和汇编语言过程有一个主要的不同点：过程作为本书的可下载源文件的一部分出现（在 [`artofasm.randallhyde.com/`](https://artofasm.randallhyde.com/)）。

*完整程序*，你可以编译并执行，在本书中被标记为*列表*。它们有一个列表编号/标识符，形式为“Listing *C*-*N*”，其中*C*是章节号，*N*是一个按顺序递增的列表编号，每个章节从 1 开始。以下是本书中出现的一个程序列表示例：

```
; Listing 1-3

; A simple MASM module that contains
; an empty function to be called by
; the C++ code in Listing 1-2.

        .CODE

; The "option casemap:none" statement
; tells MASM to make all identifiers
; case-sensitive (rather than mapping
; them to uppercase). This is necessary
; because C++ identifiers are case-
; sensitive.

        option  casemap:none

; Here is the "asmFunc" function.

        public  asmFunc
asmFunc PROC

; Empty function just returns to C++ code.

        ret     ; Returns to caller

asmFunc ENDP
        END
```

Listing 1：一个由 Listing 1-2 中的 C++ 程序调用的 MASM 程序

像过程一样，所有列表都可以在我的网站上以电子形式获取：[`artofasm.randallhyde.com/`](https://artofasm.randallhyde.com/)。这个链接将引导你到包含本书所有源文件和其他支持信息的页面（如勘误表、电子章节以及其他有用信息）。有几个章节将列表编号附加到过程和宏，这些并非完整的程序，仅为提高可读性。有一些列表演示了 MASM 语法错误或无法运行。源代码仍然会以该列表名的形式出现在电子版分发中。

通常，本书在可执行的列表之后会给出构建命令和示例输出。以下是一个典型的示例（用户输入以粗体显示）：

```
C:\>**build listing4-7**

C:\>**echo off**
 Assembling: listing4-7.asm
c.cpp

C:\>**listing4-7**
Calling Listing 4-7:
aString: maxLen:20, len:20, string data:'Initial String Data'
Listing 4-7 terminated
```

本书中的大多数程序从 Windows *命令行* 运行（即在 *cmd.exe* 应用程序中）。默认情况下，本书假设你是从 C: 驱动器的根目录运行程序。因此，每个构建命令和示例输出通常都会有 `C:\>` 作为命令行中你输入的命令的前缀。然而，你也可以从任何驱动器或目录运行程序。

如果你对 Windows 命令行完全陌生，请花些时间了解 Windows 命令行解释器（CLI）。你可以通过在 Windows 的 `运行` 命令中执行 *cmd.exe* 程序来启动 CLI。由于在阅读本书时你将频繁使用 CLI，我建议在桌面上创建一个 *cmd.exe* 的快捷方式。在附录 C 中，我描述了如何创建该快捷方式，以便自动设置你需要的环境变量，轻松运行 MASM（以及 Microsoft Visual C++ 编译器）。附录 D 为那些不熟悉 CLI 的人提供了一个 Windows CLI 的快速入门。
