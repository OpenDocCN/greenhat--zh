

## 第九章：9 数字转换



![](img/opener.jpg)

本章讨论了各种数字格式之间的基本转换，包括整数到十进制字符串、整数到十六进制字符串、浮点数到字符串、十六进制字符串到整数、十进制字符串到整数以及实数字符串到浮点数。还涵盖了字符串到数字转换的错误处理以及性能优化。最后，介绍了标准精度转换（适用于 8 位、16 位、32 位和 64 位整数格式）和扩展精度转换（例如，128 位整数/字符串转换）。

在本章中，您将直接在汇编语言中解决问题，而不是像前几章那样从高级语言（HLL）中翻译解决方案。这里的一些示例首先展示了一个使用 HLL 解决问题的代码，然后提供一个优化后的汇编语言解决方案。这应该帮助您学习在不依赖 HLL 的情况下解决汇编语言问题，从而生成更高质量的程序。

### 9.1 将数字字符串转换为值

到目前为止，本书一直依赖 C 标准库来执行数字输入/输出（将数字数据写入显示屏并从用户读取数字数据）。然而，标准库并不提供扩展精度的数字 I/O 功能（甚至 64 位数字 I/O 都存在问题；本书一直在使用 GCC 扩展的 printf()来执行 64 位数字输出）。因此，现在是时候了解如何在汇编语言中进行数字 I/O 了。

由于大多数操作系统只支持字符或字符串输入和输出，您实际上不会执行数字 I/O。相反，您将编写函数来转换数字值和字符串之间的关系，然后进行字符串 I/O。本节中的示例适用于 64 位（非扩展精度）和 128 位值，但算法是通用的，可以扩展到任何位数。

#### 9.1.1 数字值到十六进制字符串

在本节中，您将学习如何将数值（字节、半字、字、双字等）转换为包含等效十六进制字符的字符串。首先，您需要一个函数，将 4 位的半字节转换为一个单一的 ASCII 字符，字符范围为 '0' 到 '9' 或 'A' 到 'F'。在类似 C 的高级语言（HLL）中，您可以按如下方式编写此函数：

```
// Assume nibbleIn is in the range 0-15: 

charOut = nibbleIn + '0'; 
if(charOut > '9') charOut = charOut + ('A' - '9' - 1); 
```

您可以通过将数字值与 '0'（0x30）进行或运算，将任何 0 到 9 范围内的数值转换为其对应的 ASCII 字符。不幸的是，这会将 0xA 到 0xF 范围内的数字值映射到 0x3A 到 0x3F，因此 C 代码会检查是否产生了大于 0x3A 的值，并加上 7（'A' – '9' – 1），以生成最终的字符代码，范围为 0x41 到 0x46（'A'到'F'）。

使用一个将半字节转换为相应 ASCII 字符的函数，你可以通过获取数字中的所有半字节并对每一个半字节调用该函数来转换字节、半字等。然而，由于 ARM 汇编语言程序通常处理的对象不小于字节，因此编写一个将字节值转换为两个 ASCII 字符的函数会更直接、更高效。我们将这个函数称为 btoh（字节到十六进制）。

清单 9-1 展示了一个直接的 btoh 实现。该函数期望 X1 中的单字节值（忽略 X1 中位 8 到 63 的部分），并返回 X1 中位 0 到 15 的两个字符。清单 9-1 通过使用第七章中描述的技术，将 C 算法转换为汇编语言。

```
// Listing9-1.S 

#include "aoaa.inc"

            proc    btoh_simple 
            and     x1, x1, #0xFF   // Ensure only 8 bits. 
            mov     x0, x1          // Save LO nibble. 

            // Process the HO nibble: 

          ❶ lsr     x1, x1, #4      // Move HO nibble to LO posn. 
            orr     x1, x1, #'0'    // Convert to 0x30 to 0x3F. 
            cmp     x1, #'9'        // See if 0x3A to 0x3F. 
            bls     le9as 
            add     x1, x1, #7      // Convert 0x3A to 0x3F to 
 le9as:                             // 'A' through 'F'. 

            // Process the LO nibble: 

          ❷ and     x0, x0, #0xF    // Strip away HO nibble. 
            orr     x0, x0, #'0'    // Convert to 0x30 to 0x3F. 
            cmp     x0, #'9'        // See if 0x3A to 0x3F. 
            bls     le9bs 
            add     x0, x0, #7      // Convert 0x3A to 0x3F to 
 le9bs:                             // 'A' through 'F'. 
            // Merge the 2 bytes into X1\. 

            orr     x1, x1, x0, lsl #8 
            ret 
            endp    btoh_simple 
```

该函数返回对应于位 0 到 7 中的 HO 半字节❶的字符，以及对应于位 8 到 15 中的 LO 半字节❷的字符。这是因为你通常会使用该函数构建包含转换后的十六进制值的字符字符串。字符字符串本质上是*大端序*，最重要的数字出现在最低的内存地址（因此当你打印字符串时，数字会从左到右读取）。将两个字符交换后返回到 X1，允许你通过单条指令将这两个字符作为半字存储到内存中。

你可能会想知道，为什么 btoh_simple 将值传递给 X1 中的 convert，而不是 X0（标准的“第一个参数”位置）。这是为了预见到将来会有一些函数将字符输出到内存缓冲区（字符串）。对于这些基于字符串的函数，X0 将包含缓冲区的地址。

由于清单 9-1 基本上是手动编译的 C/C++代码，因此性能大致与（或比）优化 C/C++编译器处理之前给出的 C 代码要差。为了编写更快的汇编语言代码，你首先需要测量两个函数的性能，以确定哪一个更快。虽然可以使用许多软件工具（性能分析器或*分析器*）来进行此操作，但我采用了一个简单的解决方案：编写一个主程序，反复调用该函数，然后使用 Unix 时间命令行工具来测量程序运行所需的时间。例如，清单 9-2 展示了这样一个程序。

```
// Listing9-2.S 

#include "aoaa.inc"

`Include both simple and other code here necessary for a working program.` 

            proc    asmMain, public 

            locals  am                  // Preserve the X20 and 
            dword   saveX20             // X21 registers that 
            dword   saveX21             // this program uses 
            byte    stackspace, 64      // as loop-control 
            endl    am                  // variables. 

            enter   am.size    // Create activation record. 

            str     x20, [fp, #saveX20] // Preserve nonvolatile 
            str     x21, [fp, #saveX21] // registers. 

// Outer loop executes 10,000,000 times: 

            ldr     x20, =10000000 
outer: 

// Inner loop executes 256 times, once for each byte value. 
// It just calls the btoh_*** function and ignores the 
// return value. Do this to measure the speed of the 
// function. 

#define funcToCall btoh_x1 // btoh_x1, btoh2, btoh_nob, or btoh_simple 

            mov     x21, #256 
inner:      add     x1, x20, #-1 
            bl      funcToCall 
            adds    x21, x21, #-1 
            bne     inner 
            adds    x20, x20, #-1 
            bne     outer 

            mov     x1, #0x9a       // Value to test 
            mov     x6, x1          // Save for later. 
            bl      funcToCall 

            // Print btoh_*** return result: 

            and     x2, x1, #0xff   // Print HO nibble first. 
            mstr    x2, [sp, #8] 
            lsr     x3, x1, #8      // Print LO nibble second. 
 mstr    x3, [sp, #16] 
            mov     x1, x6          // Retrieve save value. 
            mstr    x1, [sp] 
            lea     x0, fmtStr1 
            bl      printf 
            ldr     x21, [fp, #saveX21] // Restore nonvolatile 
            ldr     x20, [fp, #saveX20] // registers. 
            leave 
            ret 

            endp    asmMain 
```

一位高级软件工程师可能会发现这种测量代码执行时间的技术存在几个缺陷。然而，它简单、易于理解和使用，并且不需要任何特殊的软件工具。尽管它所产生的测量结果并不完美，但对于大多数目的来说已经足够。

这是构建命令和示例输出（使用 Unix 时间命令来计时程序运行）：

```
$ ./build Listing9-2 
$ time ./Listing9-2 
Calling Listing9-2: 
Value=9a, as hex=9A 
Listing9-2 terminated 
./Listing9-2  3.49s user 0.01s system 98% cpu 3.542 total 
```

在我的 Mac mini M1 上，这大约花费了 3.5 秒时间运行。（显然，这会因系统而异；例如，在 Raspberry Pi 3 上，它大约花费了 37 秒。）

如第七章所述，分支指令通常比直线代码运行得更慢。清单 9-2 使用了分支来处理转换后的字符是'0'到'9'或'A'到'F'的情况。我写了一个使用 csel 指令的版本，用于区分这两种情况，方法是在对半字值进行 OR 操作或加上'0'后使用它。代码运行时间为 2.5 秒（在 Mac mini M1 上）。然而，这是通过不保存 X1 和 X2 寄存器实现的。将 X1 和 X2 保存到内存并恢复它们使得执行时间增加到了 4.68 秒。

你刚刚发现了 ARM 汇编代码中的一个大时间瓶颈：访问内存非常慢（而且 ldp/stp 指令比 ldr/str 指令慢得多）。这就是为什么 ARM 定义了非易失性寄存器的原因，这样你就不必在内存中保存某些工作寄存器。然而，保存易失性寄存器有时是值得的，以确保程序的正确性。汇编语言代码可能很快变得复杂，而一个函数覆盖了你在调用代码中忘记保存的寄存器，可能导致长时间的调试过程。一个有缺陷的快速程序永远不如一个运行正常的较慢程序好。

在为 Raspberry Pi 400 编写 32 位 ARM 代码时（这是本系列的第二卷），我发现使用一个 256 元素的查找表（每个元素包含与十六进制值对应的两个字符）比标准算法更快。当我在 64 位 ARM 汇编中尝试这种方法时，运行时间为 4.6 秒。再一次，内存访问（至少在 Apple M1 CPU 上）是昂贵的。在其他系统上，如 Pi 3、4 或 5，你会得到不同的结果。

一旦你能够将单个字节转换为一对十六进制字符，创建字符串并输出到显示器就变得简单了。我们可以为数字中的每个字节调用 btoh（字节到十六进制）函数，并将相应的字符存储在字符串中。通过这个函数，你可以编写 btoStr（字节到字符串）、hwtoStr（半字到字符串）、wtoStr（字到字符串）和 dtoStr（双字到字符串）函数。本章扩展了几个较低级别的函数（btoStr、hwtoStr 和 wtoStr），并使用过程调用来处理较大尺寸的转换（dtoStr）。在第十三章中，我讨论了宏，它们将提供另一种简便的方式来扩展这些函数。

本书的方法是尽量编写快速转换代码。如果你更倾向于节省空间而不是提高速度，请参见以下“减少代码大小”框中的详细信息。

所有的二进制到十六进制字符串函数都将接受两个参数：X1 寄存器中的待转换值，以及一个指向字符串缓冲区的指针，结果将保存在 X0 中。这些函数假设缓冲区足够大，能够容纳字符串结果：btoStr 需要一个 3 字符的缓冲区，hwtoStr 需要一个 5 字符的缓冲区，wtoStr 需要一个 9 字符的缓冲区，dtoStr 需要一个 17 字符的缓冲区。值中的每个字节需要两个字符存放在缓冲区中。除了字符数据外，缓冲区还必须包含 1 字节用于零终止字节。调用者需要负责确保缓冲区足够大。

为了实现这四个十六进制到字符串的函数，我将首先编写四个十六进制到缓冲区的函数。*tobuf 和 *tostr 函数之间有两个不同之处（其中 * 表示根据正则表达式语法的替代：b、hw、w 或 d）。

+   *tobuf 函数不会保留任何寄存器。它们会修改 X0 和 X2 中的值。

+   *tobuf 函数将 X0 指向字符串末尾的零终止字节，这通常是有用的；*tostr 函数则保留 X0 的值（指向输出缓冲区的第一个字符）。

我还将借此机会介绍另一项汇编语言特性：函数的多个入口点。btobuf、htobuf、wtobuf 和 dtobuf 函数都包含公共代码。示例 9-3 将所有这些函数合并为一个单一的函数（dtobuf），并为其他三个函数提供单独的代码入口。

```
// Listing9-3.S 

 `Usual header code snipped` 

// dtobuf 
//
// Convert a dword to a string of 16 hexadecimal digits. 
//
// Inputs: 
//  X0-     Pointer to the buffer. Must have at least 
//          17 bytes available. 
//  X1-     Value to convert 
//
// Outputs: 
//  X0-     Points at zero-terminating byte at the end 
//          of the converted string 
//
// Note:    This function does not preserve any registers. 
//          It is the caller's responsibility to preserve 
//          registers. 
//
//          Registers modified: X0, X2 

            proc    dtobuf 

#define AtoF   ('A'-'9'-1) 

            // Process the HO nibble: 

          ❶ lsr     x2, x1, #60 
            orr     w2, w2, #'0'    // Convert to 0x30 to 0x3F. 
            cmp     w2, #'9'        // See if 0x3A to 0x3F. 
            bls     dec15           // Skip if 0 to 9\. 
            add     w2, w2, #AtoF   // If it was A to F 
 dec15: 
            strb    w2, [x0], #1    // Store byte to memory. 

            // Process nibble 14: 

            lsr     x2, x1, #56     // See comments for HO nibble. 
            and     x2, x2, 0xf 
            orr     w2, w2, #'0' 
            cmp     w2, #'9' 
            bls     dec14 
            add     w2, w2, #AtoF 
dec14:      strb    w2, [x0], #1 

            // Process nibble 13: 

            lsr     x2, x1, #52 
            and     x2, x2, 0xf 
            orr     w2, w2, #'0' 
            cmp     w2, #'9' 
            bls     dec13 
            add     w2, w2, #AtoF 
dec13:      strb    w2, [x0], #1 

            // Process nibble 12: 

            lsr     x2, x1, #48 
            and     x2, x2, 0xf 
            orr     w2, w2, #'0' 
            cmp     w2, #'9' 
            bls     dec12 
            add     w2, w2, #AtoF 
dec12:      strb    w2, [x0], #1 

            // Process nibble 11: 

            lsr     x2, x1, #44 
            and     x2, x2, 0xf 
 orr     w2, w2, #'0' 
            cmp     w2, #'9' 
            bls     dec11 
            add     w2, w2, #AtoF 
dec11:      strb    w2, [x0], #1 

            // Process nibble 10: 

            lsr     x2, x1, #40 
            and     x2, x2, 0xf 
            orr     w2, w2, #'0' 
            cmp     w2, #'9' 
            bls     dec10 
            add     w2, w2, #AtoF 
dec10:      strb    w2, [x0], #1 

            // Process nibble 9: 

            lsr     x2, x1, #36 
            and     x2, x2, 0xf 
            orr     w2, w2, #'0' 
            cmp     w2, #'9' 
            bls     dec9 
            add     w2, w2, #AtoF 
dec9:       strb    w2, [x0], #1 

            // Process nibble 8: 

            lsr     x2, x1, #32 
            and     x2, x2, 0xf 
            orr     w2, w2, #'0' 
            cmp     w2, #'9' 
            bls     dec8 
            add     w2, w2, #AtoF 
dec8:       strb    w2, [x0], #1 

// Entry point for wtobuf 
//
// wtobuf 
//
// Convert a word to a string of 8 hexadecimal digits. 
//
// Inputs: 
//  X0-     Pointer to the buffer. Must have at least 
//          9 bytes available. 
//  X1-     Value to convert 
//
// Outputs: 
//  X0-     Points at zero-terminating byte at the end 
//          of the converted string 
//
// Note:    This function does not preserve any registers. 
//          It is the caller's responsibility to preserve 
//          registers. 
//
//          Registers modified: X0, X2 

❷ wtobuf: 
            // Process nibble 7: 

            lsr     x2, x1, #28 // See comments for nibble 15\. 
            and     x2, x2, 0xf 
            orr     w2, w2, #'0' 
            cmp     w2, #'9' 
            bls     dec7 
            add     w2, w2, #AtoF 
dec7:       strb    w2, [x0], #1 

            // Process nibble 6: 

            lsr     x2, x1, #24 
            and     x2, x2, 0xf 
            orr     w2, w2, #'0' 
            cmp     w2, #'9' 
            bls     dec6 
            add     w2, w2, #AtoF 
dec6:       strb    w2, [x0], #1 

            // Process nibble 5: 

            lsr     x2, x1, #20 
            and     x2, x2, 0xf 
            orr     w2, w2, #'0' 
            cmp     w2, #'9' 
            bls     dec5 
            add     w2, w2, #AtoF 
dec5:       strb    w2, [x0], #1 

            // Process nibble 4: 

            lsr     x2, x1, #16 
            and     x2, x2, 0xf 
            orr     w2, w2, #'0' 
            cmp     w2, #'9' 
            bls     dec4 
            add     w2, w2, #AtoF 
dec4:       strb    w2, [x0], #1 

// Entry point for htobuf: 
//
// htobuf 
//
// Convert a half word to a string of 4 hexadecimal digits. 
//
// Inputs: 
//  X0-     Pointer to the buffer. Must have at least 
//          5 bytes available. 
//  X1-     Value to convert 
//
// Outputs: 
//  X0-     Points at zero-terminating byte at the end 
//          of the converted string 
//
// Note:    This function does not preserve any registers. 
//          It is the caller's responsibility to preserve 
//          registers. 
//
//          Registers modified: X0, X2 

❸ htobuf: 
            // Process nibble 3: 

            lsr     x2, x1, #12 // See comments for nibble 15\. 
            and     x2, x2, 0xf 
            orr     w2, w2, #'0' 
            cmp     w2, #'9' 
            bls     dec3 
            add     w2, w2, #AtoF 
dec3:       strb    w2, [x0], #1 

            // Process nibble 2: 

            lsr     x2, x1, #8 
            and     x2, x2, 0xf 
            orr     w2, w2, #'0' 
            cmp     w2, #'9' 
            bls     dec2 
            add     w2, w2, #AtoF 
dec2:       strb    w2, [x0], #1 

// Entry point for btobuf: 
//
// btobuf 
//
// Convert a byte to a string of two hexadecimal digits. 
//
// Inputs: 
//  X0-     Pointer to the buffer. Must have at least 
//          3 bytes available. 
//  X1-     Value to convert 
//
// Outputs: 
//  X0-     Points at zero-terminating byte at the end 
//          of the converted string 
//
// Note:    This function does not preserve any registers. 
//          It is the caller's responsibility to preserve 
//          registers. 
//
//          Registers modified: X0, X2 

 // Process nibble 1: 

❹ btobuf: 
            lsr     x2, x1, #4      // See comments for nibble 15\. 
            and     x2, x2, 0xf 
            orr     w2, w2, #'0' 
            cmp     w2, #'9' 
            bls     dec1 
            add     w2, w2, #AtoF 
dec1:       strb    w2, [x0], #1 

            // Process LO nibble: 

            and     x2, x1, 0xf 
            orr     x2, x2, #'0' 
            cmp     w2, #'9' 
            bls     dec0 
            add     w2, w2, #AtoF 
dec0:       strb    w2, [x0], #1 

            strb    wzr, [x0]       // Zero-terminate. 
            ret 
            endp    dtobuf 
```

dtobuf 函数首先处理 dword ❶ 的 HO nibble（nibble 15）。为了提高性能，这段代码使用了展开的循环，逐个处理每个 nibble。每个 nibble 使用标准算法将二进制值转换为十六进制字符。

在这段代码处理完 HO 八个十六进制数字后，你会注意到 wtobuf 函数 ❷ 的入口点。调用 wtobuf 的代码会将控制转移到 dtobuf 函数的中间部分（字面意义上）。之所以能这样工作，是因为 dtobuf 不会将任何内容压入堆栈，也不会以其他方式改变环境，因此 wtobuf 进入时不需要特殊处理。类似地，htobuf ❸ 和 btobuf ❹ 的入口点分别位于 nibble 3 和 nibble 1。通过将这些函数合并到一个代码段中，你节省了 wtobuf、htobuf 和 btobuf 函数所需的所有代码。

我做了几次失败的优化尝试。首先，我尝试将 8 个字节保存在一个寄存器中，并按双字（dword）而不是按字节逐个写入内存。这在我的 Mac mini M1 上运行得更慢。我还尝试通过使用 csel 指令消除代码中的分支，结果代码反而变得更慢。甚至我还尝试使用 ubfx 指令（请参见第十二章），但它的执行速度仍然比带分支的代码更慢。我在 Mac mini M1 和 Raspberry Pi 400 上对这些版本进行了计时。尽管这两台机器的计时差异很大，但三种算法的相对性能保持不变（带分支的版本始终更快）。有时候，使用不同的算法技巧反而会适得其反。这就是为什么你应该始终测试你的代码性能（最好是在多种架构上进行测试）的原因。

在处理完 *tobuf 函数后，编写 *toStr 函数相对容易。*toStr 函数仅调用 *tobuf 函数，并保留 *tobuf 函数修改的寄存器。清单 9-4 提供了这些函数的代码（请注意，在线文件中的 *Listing9-4.S* 还包括了 dtobuf 函数的代码；为了避免冗余，我已经将该代码从清单中移除）。

```
// Listing9-4.S 
//
// btoStr, htoStr, wtoStr, and dtoStr functions 
// Also includes btobuf, htobuf, wtobuf, and 
// dtobuf functions 

            #include    "aoaa.inc"

            .section    .rodata, "" 
ttlStr:     .asciz      "Listing 9-4"

            .data 

// Buffer space used by main program 

buffer:     .space      256,0 

            .code 
            .extern     printf 

// Return program title to C++ program: 

            proc    getTitle, public 
            lea     x0, ttlStr 
            ret 
            endp    getTitle 

❶ `Insert the code for dtobuf here. See Listing 9-3.` 

// btoStr-
//
// Inputs: 
//
//  X0- Pointer to buffer that will hold the result 
//      (must allocate at least 3 bytes for buffer) 
//  X1- Value to print (in LO byte) 
//
// Outputs: 
//
//  Buffer pointed at by X0 receives the two-character 
//  conversion of the value in X1 to a hexadecimal string. 
//
//  Preserves all registers. 

          ❷ proc    btoStr 

            str     x2, [sp, #-16]! 
            stp     x0, lr, [sp, #-16]! 

 bl      btobuf 

            // Restore registers and return: 

            ldp     x0, lr, [sp], #16 
            ldr     x2, [sp], #16 
            ret 
            endp    btoStr 

// htoStr 
//
// Inputs: 
//
//  X0- Pointer to buffer that will hold the result 
//      (must allocate at least 5 bytes for buffer) 
//  X1- Value to print (in LO hword) 
//
// Outputs: 
//
//  Buffer pointed at by X0 receives the four-character 
//  conversion of the hword value in X1 to a hexadecimal string. 
//
//  Preserves all registers 

          ❸ proc    htoStr 

            str     x2, [sp, #-16]! 
            stp     x0, lr, [sp, #-16]! 

            bl      htobuf 

            // Restore registers and return: 

            ldp     x0, lr, [sp], #16 
            ldr     x2, [sp], #16 
            ret 
            endp    htoStr 

// wtoStr 
//
// Inputs: 
//
//  X0- Pointer to buffer that will hold the result 
//      (must allocate at least 9 bytes for buffer) 
//  X1- Value to print (in LO word) 
//
// Outputs: 
//
//  Buffer pointed at by X0 receives the eight-character 
//  conversion of the word value in X1 to a hexadecimal string. 
//
//  Preserves all registers 

          ❹ proc    wtoStr 

 str     x2, [sp, #-16]! 
            stp     x0, lr, [sp, #-16]! 

            bl      wtobuf 

            // Restore registers and return: 

            ldp     x0, lr, [sp], #16 
            ldr     x2, [sp], #16 
            ret 
            endp    wtoStr 

// dtoStr 
//
// Inputs: 
//
//  X0- Pointer to buffer that will hold the result 
//      (must allocate at least 17 bytes for buffer) 
//  X1- Value to print 
//
// Outputs: 
//
//  Buffer pointed at by X0 receives the 16-character 
//  conversion of the dword value in X1 to a hexadecimal string. 
//
//  Preserves all registers 

          ❺ proc    dtoStr 

            str     x2, [sp, #-16]! 
            stp     x0, lr, [sp, #-16]! 

            bl      dtobuf 

            // Restore registers and return: 

            ldp     x0, lr, [sp], #16 
            ldr     x2, [sp], #16 
            ret 
            endp    dtoStr 

// Utility functions to print bytes, hwords, words, and dwords: 

pbStr:      wastr   "Byte=%s\n"

            proc    pByte 

            locals  pb 
            qword   pb.saveX0X1 
            byte    pb.buffer, 32 
            byte    pb.stkSpace, 64 
            endl    pb 

            enter   pb.size 
            stp     x0, x1, [fp, #pb.saveX0X1] 

 mov     x1, x0 
            add     x0, fp, #pb.buffer  // lea x0, stkSpace 
            bl      btoStr 

            lea     x0, pbStr 
            add     x1, fp, #pb.buffer 
            mstr    x1, [sp] 
            bl      printf 

            ldp     x0, x1, [fp, #pb.saveX0X1] 
            leave 
            endp    pByte 

phStr:      wastr   "Hword=%s\n"

            proc    pHword 

            locals  ph 
            qword   ph.saveX0X1 
            byte    ph.buffer, 32 
            byte    ph.stkSpace, 64 
            endl    ph 

            enter   ph.size 
            stp     x0, x1, [fp, #ph.saveX0X1] 

            mov     x1, x0 
            add     x0, fp, #ph.buffer  // lea x0, stkSpace 
            bl      htoStr 

            lea     x0, phStr 
            add     x1, fp, #ph.buffer 
            mstr    x1, [sp] 
            bl      printf 

            ldp     x0, x1, [fp, #ph.saveX0X1] 
            leave 
            endp    pHword 

pwStr:      wastr   "Word=%s\n"

            proc    pWord 

            locals  pw 
            qword   pw.saveX0X1 
            byte    pw.buffer, 32 
            byte    pw.stkSpace, 64 
            endl    pw 

            enter   pw.size 
            stp     x0, x1, [fp, #pw.saveX0X1] 

            mov     x1, x0 
            add     x0, fp, #pw.buffer  // lea x0, stkSpace 
            bl      wtoStr 

 lea     x0, pwStr 
            add     x1, fp, #pw.buffer 
            mstr    x1, [sp] 
            bl      printf 

            ldp     x0, x1, [fp, #pw.saveX0X1] 
            leave 
            endp    pWord 

pdStr:      wastr   "Dword=%s\n"

            proc    pDword 

            locals  pd 
            qword   pd.saveX0X1 
            byte    pd.buffer, 32 
            byte    pd.stkSpace, 64 
            endl    pd 

            enter   pd.size 
            stp     x0, x1, [fp, #pd.saveX0X1] 

            mov     x1, x0 
            add     x0, fp, #pd.buffer  // lea x0, stkSpace 
            bl      dtoStr 

            lea     x0, pdStr 
            add     x1, fp, #pd.buffer 
            mstr    x1, [sp] 
            bl      printf 

            ldp     x0, x1, [fp, #pd.saveX0X1] 
            leave 
            endp    pDword 

// Here is the asmMain function: 

            proc    asmMain, public 

            // Local storage: 

            locals  am 
            byte    stackspace, 64 
            endl    am 

            enter   am.size             // Create activation record. 

            ldr     x0, =0x0123456789abcdef 
            bl      pByte 
            bl      pHword 
            bl      pWord 
            bl      pDword 

            leave 
 ret 

            endp    asmMain 
```

如上所述，我已将 dtobuf 函数从此清单中移除；请插入该代码 ❶。btoStr 函数 ❷ 将 X0、X2 和 LR 寄存器保存在堆栈中（这些寄存器会被 *tobuf 函数调用修改），调用 btobuf 函数将两个十六进制数字写入 X0 所指向的缓冲区，然后恢复寄存器并返回。对于 htoStr ❸、wtoStr ❹ 和 dtoStr ❺，代码基本相同，唯一的区别是它们调用的转换函数不同。

以下是清单 9-4 中程序的构建命令和示例输出：

```
$ ./build Listing9-4 
$ ./Listing9-4 
Calling Listing9-4: 
Byte=EF 
Hword=CDEF 
Word=89ABCDEF 
Dword=0123456789ABCDEF 
Listing9-4 terminated 
```

由于本书中出现的汇编代码调用了 C/C++ 标准库函数进行 I/O 操作，因此这些二进制到十六进制字符串的函数将生成零终止的 C 兼容字符串。如果需要的话，它们足够简单，能够修改以生成其他字符串格式。更多关于字符串函数的内容，请参见第十四章。

#### 9.1.2 扩展精度的十六进制值转换为字符串

扩展精度的十六进制到字符串转换很简单：它只是上一节中正常的十六进制转换例程的扩展。例如，清单 9-5 是一个 128 位十六进制转换函数 qtoStr，它期望 X2:X1 中的指针指向一个 128 位值，而 X0 中的指针指向一个缓冲区。*Listing9-5.S* 基本上是基于 *Listing9-4.S*；为了避免冗余，这里只包含了 qtoStr 函数。

```
// Listing9-5.S 
//
// qtoStr 
//
// Inputs: 
//
//  X0-     Pointer to buffer that will hold the result 
//          (must allocate at least 33 bytes for buffer) 
//  X2:X1-  Value to print 
//
// Outputs: 
//
//  Buffer pointed at by X0 receives the 32-character 
//  conversion of the dword value in X2:X1 to a hexadecimal string. 
//
//  Preserves all registers 

            proc    qtoStr 

            str     x2, [sp, #-16]! 
            stp     x0, lr, [sp, #-16]! 
            str     x1, [sp, #-16]!     // Save for later. 

            mov     x1, x2              // Convert HO dword first. 
            bl      dtobuf 
            ldr     x1, [sp], #16       // Restore X1 value. 
            bl      dtobuf 

            // Restore registers and return: 

            ldp     x0, lr, [sp], #16 
            ld4     x2, [sp], #16 
            ret 
            endp    qtoStr 
```

清单 9-5 中的函数调用了 dtobuf 两次，通过首先转换 HO 双字（dword），然后转换 LO 双字（dword），并连接它们的结果，将 128 位的 qword 值转换为字符串。要将此转换扩展到任意数量的字节，只需将 HO 字节转换为大对象的 LO 字节即可。

#### 9.1.3 无符号十进制值转换为字符串

十进制输出比十六进制输出稍微复杂一些，因为与十六进制值不同，二进制数的高位（HO）位会影响十进制表示中的低位（LO）数字。因此，必须通过一次提取一个十进制数字的方式来创建二进制数的十进制表示。

无符号十进制输出的最常见解决方案是不断将值除以 10，直到结果变为 0。第一次除法后的余数是 0 到 9 之间的值，对应十进制数字的低位。不断进行除法（以及其相应的余数）会依次提取数字中的各个数字。

该问题的迭代解决方案通常会为一个足够大的字符字符串分配存储空间，以容纳整个数字。代码接着会在循环中提取十进制数字，并逐个将它们放入该字符串中。在转换过程结束时，例程会按反向顺序打印字符串中的字符（记住，除法算法首先提取低位数字，最后提取高位数字，这与打印时需要的顺序相反）。

本节采用了*递归解决方案*，因为它更为优雅。该解决方案通过将值除以 10 并将余数保存在局部变量中开始。如果商不为 0，例程将递归调用自身以首先输出所有前导数字。从递归调用返回（递归调用输出所有前导数字）后，递归算法将输出与余数相关的数字以完成操作。例如，下面是打印十进制值 789 时操作的执行过程：

1.  将 789 除以 10，商为 78，余数为 9。

2.  将余数（9）保存在局部变量中，并使用商递归调用该例程。

3.  递归入口 1：将 78 除以 10，商为 7，余数为 8。

4.  将余数（8）保存在局部变量中，并使用商递归调用该例程。

5.  递归入口 2：将 7 除以 10，商为 0，余数为 7。

6.  将余数（7）保存在局部变量中。由于商为 0，因此不再递归调用该例程。

7.  输出保存在局部变量（7）中的余数值。返回到调用者（递归入口 1）。

8.  返回到递归入口 1：输出在递归入口 1（8）中保存在局部变量中的余数值。返回到调用者（最初调用该过程的地方）。

9.  最初的调用：输出在最初调用（9）中保存在局部变量中的余数值。返回到输出例程的最初调用者。

列表 9-6 提供了 64 位无符号整数的递归算法实现。

```
// Listing9-6.S 
//
// u64toBuf function 

            #include    "aoaa.inc"

            .section    .rodata, "" 
ttlStr:     .asciz      "Listing 9-6"
fmtStr1:    .asciz      "Value(%llu) = string(%s)\n"

            .align      3 
qwordVal:   .dword      0x1234567890abcdef 
            .dword      0xfedcba0987654321 

            .data 
buffer:     .space      256,0 

            .code 
            .extern     printf 

// Return program title to C++ program: 

 proc    getTitle, public 
            lea     x0, ttlStr 
            ret 
            endp    getTitle 

// u64ToStr 
//
//  Converts a 64-bit unsigned integer to a string 
//
//  Inputs: 
//      X0-     Pointer to buffer to receive string 
//      X1-     Unsigned 64-bit integer to convert 
//
//  Outputs: 
//      Buffer- Receives the zero-terminated string 
//
//  Buffer must have at least 21 bytes allocated for it. 
//  This function preserves all registers. 

          ❶ proc    u64ToStr 
            stp     x0, x1, [sp, #-16]! 
            stp     x2, x3, [sp, #-16]! 
            str     lr, [sp, #-16]! 

            bl      u64ToBuf 

            ldr     lr, [sp], #16 
            ldp     x2, x3, [sp], #16 
            ldp     x0, x1, [sp], #16 
            ret 
            endp    u64ToStr 

// u64ToBuf 
//
//  Converts a 64-bit unsigned integer to a string 
//
//  Inputs: 
//      X0-     Pointer to buffer to receive string 
//      X1-     Unsigned 64-bit integer to convert 
//
//  Outputs: 
//      X0-     Points at zero-terminating byte 
//      Buffer- Receives the zero-terminated string 
//
//  Buffer must have at least 21 bytes allocated for it. 
//
//  Caller must preserve X0, X1, X2, and X3! 

          ❷ proc    u64ToBuf 
            cmp     x1, xzr         // See if X1 is 0\. 
            bne     u64ToBufRec 

            // Special case for zero, just write 
            // "0" to the buffer. Leave X0 pointing 
 // at the zero-terminating byte. 

            mov     w1, #'0' 
            strh    w1, [x0], #1    // Also emits zero byte 
            ret 
            endp    u64ToBuf 

// u64ToBufRec is the recursive version that handles 
// nonzero values: 

          ❸ proc    u64ToBufRec 
            stp     x2, lr, [sp, #-16]! // Preserve remainder. 

            // Divide X1 by 10 and save quotient and remainder: 

          ❹ mov     x2, #10 
            udiv    x3, x1, x2      // X3 = quotient 
            msub    x2, x3, x2, x1  // X2 = remainder 

            // Make recursive call if quotient is not 0: 

            cmp     x3, xzr 
            beq     allDone 

          ❺ mov     x1, x3              // Set up for call. 
            bl      u64ToBufRec 

            // When this function has processed all the 
            // digits, write them to the buffer. Also 
            // write a zero-terminating byte, in case 
            // this is the last digit to output. 

❻ allDone:    orr     w2, w2, #'0'    // Convert to char. 
            strh    w2, [x0], #1    // Bump pointer after store. 
          ❼ ldp     x2, lr, [sp], #16 
            ret 
            endp    u64ToBufRec 

// Here is the "asmMain" function. 

            proc    asmMain, public 

            enter   64              // Reserve space on stack. 

// Test u64ToBuf: 

            mov     x1, 0xFFFF 
            movk    x1, 0xFFFF, lsl #16 
            movk    x1, 0xFFFF, lsl #32 
            movk    x1, 0xFFFF, lsl #48 
            lea     x0, buffer 
            bl      u64ToStr 

            lea     x2, buffer 
 mstr    x2, [sp, #8] 
            mov     x1, 0xFFFF 
            movk    x1, 0xFFFF, lsl #16 
            movk    x1, 0xFFFF, lsl #32 
            movk    x1, 0xFFFF, lsl #48 
            mstr    x1, [sp] 
            lea     x0, fmtStr1 
            bl      printf 

            leave 
            ret 
            endp    asmMain 
```

u64toStr 函数❶是一个外观函数，在调用 u64ToBuf 过程时保存寄存器。u64ToBuf 函数❷处理 X1 包含 0 的特殊情况（当结果为 0 时，递归代码终止）。如果 X1 在进入时为 0，代码会立即将字符'0'写入输出缓冲区，递增 X0 并返回。如果 X1 非零，则控制权转交给递归的 u64toBufRec 函数❸来处理该值。为了性能考虑，u64ToBufRec 只保留 X2（在递归调用中包含余数值）和 LR 寄存器。

递归函数计算商和余数❹。商保存在 X3 中，余数保存在 X2 中。如果商不为零，说明还有更多的高位数字需要处理：将商复制到 X1 并进行递归调用 u64toBufRec❺。从递归调用返回❻（或者如果跳过了递归调用），所有的高位数字已经被输出到缓冲区，因此将当前数字转换为字符并添加到缓冲区的末尾。注意，后增量寻址模式会自动将 X0 递增，以指向由 strh 指令发出的以零结束的字节。代码恢复 X2 中的值❼，以防这是一次递归调用。

以下是第 9-6 列表的构建命令和示例输出：

```
$ ./build Listing9-6 
$ ./Listing9-6 
Calling Listing9-6: 
Value(18446744073709551615) = string(18446744073709551615) 
Listing9-6 terminated 
```

与十六进制输出不同，这里不需要提供字节大小、半字大小或字大小的数字到十进制字符串转换函数。只需将较小的值零扩展到 64 位即可。与十六进制转换不同，u64toStr 函数不会输出前导零，因此对所有大小的变量（64 位及更小）输出都是相同的。

这段代码有几个优化的机会。由于十进制到字符串的转换很常见（大多数程序输出都使用这个函数），而且该算法的速度不如十六进制转换快，因此优化这段代码可能是值得的。

去除递归并实现 u64toStr 的迭代版本其实很简单。这可以消除在多次递归调用中保存寄存器和返回地址的需求（通常每个数字转换会有一次递归调用），以及在每次调用时构建激活记录的需要。第 9-7 列表进一步改进了这一点，将循环展开（最多 20 次迭代，每次迭代处理一个可能的数字）。

```
// Listing9-7.S 
//
// u64toStr function (nonrecursive, straight-line 
// code version) 

            #include    "aoaa.inc"

            .section    .rodata, "" 
ttlStr:     .asciz      "Listing 9-7"
fmtStr1:    .asciz      "low=%s, " 
fmtStr2:    .asciz      "hi=%s\n"

loData:     .dword      0, 1, 10, 100, 1000, 10000, 100000 
            .dword      1000000, 10000000, 100000000 
            .dword      1000000000, 10000000000, 100000000000 
            .dword      1000000000000, 10000000000000 
            .dword      100000000000000, 1000000000000000 
            .dword      10000000000000000, 100000000000000000 
            .dword      1000000000000000000, 10000000000000000000 
            .equ        dataCnt, .-loData 

hiData:     .dword      9, 9, 99, 999, 9999, 99999, 999999 
            .dword      9999999, 99999999, 999999999 
            .dword      9999999999, 99999999999, 999999999999 
            .dword      9999999999999, 99999999999999 
            .dword      999999999999999, 9999999999999999 
            .dword      99999999999999999, 999999999999999999 
            .dword      9999999999999999999 
            .dword      -1 

            .data 
buffer:     .space      256, 0 

            .code 
            .extern     printf 

// Return program title to C++ program: 

            proc    getTitle, public 
            lea     x0, ttlStr 
            ret 
            endp    getTitle 

// u64ToBuf 
//
//  Converts a 64-bit unsigned integer to a string 
//
//  Inputs: 
//      X0-     Pointer to buffer to receive string 
//      X1-     Unsigned 64-bit integer to convert 
//
//  Outputs: 
//      Buffer- Receives the zero-terminated string 
//      X0-     Points at zero-terminating byte in string 
//
//  Buffer must have at least 21 bytes allocated for it. 
//  Note: Caller is responsible for preserving X0-X7! 

          ❶ proc    u64ToBuf 

          ❷ mov     x4, #10 
            mov     x5, xzr 
            mov     x6, xzr 
            mov     x7, xzr 

            // Handle the LO digit here: 

          ❸ udiv    x2, x1, x4      // X2 = quotient 
            msub    x3, x2, x4, x1  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 
            cmp     x2, #0 
            beq     allDone1 

            // Handle the 10's digit here: 

          ❹ udiv    x1, x2, x4      // X1 = quotient 
            msub    x3, x1, x4, x2  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 
            cmp     x1, #0 
            beq     allDone2 

            // Handle the 100's digit here: 

            udiv    x2, x1, x4      // X2 = quotient 
            msub    x3, x2, x4, x1  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 
            cmp     x2, #0 
            beq     allDone3 

            // Handle the 1000's digit here: 

            udiv    x1, x2, x4      // X1 = quotient 
            msub    x3, x1, x4, x2  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 
            cmp     x1, #0 
            beq     allDone4 

            // Handle the 10,000's digit here: 

            udiv    x2, x1, x4      // X2 = quotient 
            msub    x3, x2, x4, x1  // X3 = remainder 
 orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 
            cmp     x2, #0 
            beq     allDone5 

            // Handle the 100,000's digit here: 

            udiv    x1, x2, x4      // X1 = quotient 
            msub    x3, x1, x4, x2  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 
            cmp     x1, #0 
            beq     allDone6 

            // Handle the 1,000,000's digit here: 

            udiv    x2, x1, x4      // X2 = quotient 
            msub    x3, x2, x4, x1  // X3 = remainder 
            orr     x6, x3, #'0' 
            cmp     x2, #0 
            beq     allDone7 

            // Handle the 10,000,000's digit here: 

            udiv    x1, x2, x4      // X1 = quotient 
            msub    x3, x1, x4, x2  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x6, x3, x6, lsl #8 
            cmp     x1, #0 
            beq     allDone8 

            // Handle the 100,000,000's digit here: 

            udiv    x2, x1, x4      // X2 = quotient 
            msub    x3, x2, x4, x1  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x6, x3, x6, lsl #8 
            cmp     x2, #0 
            beq     allDone9 

            // Handle the 1,000,000,000's digit here: 

            udiv    x1, x2, x4      // X1 = quotient 
            msub    x3, x1, x4, x2  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x6, x3, x6, lsl #8 
            cmp     x1, #0 
            beq     allDone10 

            // Handle the 10,000,000,000's digit here: 

            udiv    x2, x1, x4      // X2 = quotient 
            msub    x3, x2, x4, x1  // X3 = remainder 
 orr     x3, x3, #'0' 
            orr     x6, x3, x6, lsl #8 
            cmp     x2, #0 
            beq     allDone11 

            // Handle the 100,000,000,000's digit here: 

            udiv    x1, x2, x4      // X1 = quotient 
            msub    x3, x1, x4, x2  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x6, x3, x6, lsl #8 
            cmp     x1, #0 
            beq     allDone12 

            // Handle the 1,000,000,000,000's digit here: 

            udiv    x2, x1, x4      // X2 = quotient 
            msub    x3, x2, x4, x1  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x6, x3, x6, lsl #8 
            cmp     x2, #0 
            beq     allDone13 

            // Handle the 10,000,000,000,000's digit here: 

            udiv    x1, x2, x4      // X1 = quotient 
            msub    x3, x1, x4, x2  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x6, x3, x6, lsl #8 
            cmp     x1, #0 
            beq     allDone14 

            // Handle the 100,000,000,000,000's digit here: 

            udiv    x2, x1, x4      // X2 = quotient 
            msub    x3, x2, x4, x1  // X3 = remainder 
            orr     x7, x3, #'0' 
            orr     x6, x3, x6, lsl #8
            cmp     x2, #0 
            beq     allDone15 

            // Handle the 1,000,000,000,000,000's digit here: 

            udiv    x1, x2, x4      // X1 = quotient 
            msub    x3, x1, x4, x2  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x7, x3, x7, lsl #8 
            cmp     x1, #0 
            beq     allDone16 

            // Handle the 10,000,000,000,000,000's digit here: 

            udiv    x2, x1, x4      // X2 = quotient 
            msub    x3, x2, x4, x1  // X3 = remainder 
 orr     x3, x3, #'0' 
            orr     x7, x3, x7, lsl #8     
            cmp     x2, #0 
            beq     allDone17 

            // Handle the 100,000,000,000,000,000's digit here: 

            udiv    x1, x2, x4      // X1 = quotient 
            msub    x3, x1, x4, x2  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x7, x3, x7, lsl #8 
            cmp     x1, #0 
            beq     allDone18 

            // Handle the 1,000,000,000,000,000,000's digit here: 

            udiv    x2, x1, x4      // X2 = quotient 
            msub    x3, x2, x4, x1  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x7, x3, x7, lsl #8 
            cmp     x2, #0 
            beq     allDone19 

          ❺ udiv    x1, x2, x4      // X1 = quotient 
            msub    x3, x1, x4, x2  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x7, x3, x7, lsl #8 

allDone20:  str     x7, [x0], #6 
            str     x6, [x0], #8 
            str     x5, [x0], #7 
            ret 

            // When this function has processed all the 
            // digits, write them to the buffer. Also 
            // write a zero-terminating byte, in case 
            // this is the last digit to output. 

❻ allDone1: strh    w5, [x0], #1 
            ret 

  allDone2: strh    w5, [x0], #2 
            strb    wzr, [x0] 
            ret 

  allDone3: str     w5, [x0], #3 
            ret 

  allDone4: str     w5, [x0], #4 
            strb    wzr, [x0] 
            ret 

  allDone5: str     x5, [x0], #4 
            lsr     x5, x5, #32 
 strh    w5, [x0], #1 
            ret 

  allDone6: str     w5, [x0], #4 
            lsr     x5, x5, #32 
            strh    w5, [x0], #2 
            strb    wzr, [x0] 
            ret 

❼ allDone7: strb    w6, [x0], #1 
            str     x5, [x0], #7 
            ret 

  allDone8: strh    w6, [x0], #2 
            str     x5, [x0], #7    // Writes an extra garbage byte 
            ret 

  allDone9: str     w6, [x0], #3 
            str     x5, [x0], #7 
            ret 

  allDone10: 
            str     w6, [x0], #4 
            str     x5, [x0], #7 
            ret 

  allDone11: 
            str     x6, [x0], #5 
            str     x5, [x0], #7 
            ret 

  allDone12: 
            str     x6, [x0], #6 
            str     x5, [x0], #7 
            ret 

  allDone13: 
            str     x6, [x0], #7 
            str     x5, [x0], #7 
            ret 

  allDone14: 

            str     x6, [x0], #8 
            str     x5, [x0], #7 
            ret 

❽ allDone15: 
            strb    w7, [x0], #1 
            str     x6, [x0], #8 
            str     x5, [x0], #7 
            ret 

  allDone16: 
            strh    w7, [x0], #2 
            str     x6, [x0], #8 
 str     x5, [x0], #7 
            ret 

  allDone17: 
            str     w7, [x0], #3 
            str     x6, [x0], #8 
            str     x5, [x0], #7 
            ret 

  allDone18: 
            str     w7, [x0], #4 
            str     x6, [x0], #8 
            str     x5, [x0], #7 
            ret 

  allDone19: 
            str     x7, [x0], #5 
            str     x6, [x0], #8 
            str     x5, [x0], #7 
            ret 
            endp    u64ToBuf 

// u64ToStr 
//
//  Version of u64ToBuf that preserves the registers 

          ❾ proc    u64ToStr 
            stp     x0, x1, [sp, #-16]! // Preserve registers. 
            stp     x2, x3, [sp, #-16]! 
            stp     x4, x5, [sp, #-16]! 
            stp     x6, x7, [sp, #-16]! 
            str     lr, [sp, #-16]! 
            bl      u64ToBuf 
            ldr     lr, [sp], #16 
            ldp     x6, x7, [sp], #16   // Restore registers. 
            ldp     x4, x5, [sp], #16 
            ldp     x2, x3, [sp], #16 
            ldp     x0, x1, [sp], #16 
            ret 
            endp    u64ToStr 

// Here is the asmMain function: 

            proc    asmMain, public 

            locals  am 
            qword   am.x20_x21 
            dword   am.x22 
            byte    stk, 64 
            endl    am 

            enter   am.size             // Create act rec. 

 // Preserve nonvolatile registers: 

            stp     x20, x21, [fp, #am.x20_x21] 
            str     x22, [fp, #am.x22] 

            lea     x20, loData 
            lea     x21, hiData 
            mov     x22, xzr 
 loop: 
            lea     x0, buffer 
            ldr     x1, [x20, x22, lsl #3] 
            bl      u64ToStr 

            lea     x0, fmtStr1 
            lea     x1, buffer 
            mstr    x1, [sp] 
            bl      printf 

            lea     x0, buffer 
            ldr     x1, [x21, x22, lsl #3] 
            bl      u64ToStr 

            lea     x0, fmtStr2 
            lea     x1, buffer 
            mstr    x1, [sp] 
            bl      printf 

            add     x22, x22, #1 
            cmp     x22, #(dataCnt / 8) 
            blo     loop 

            ldr     x22, [fp, #am.x22] 
            ldp     x20, x21, [fp, #am.x20_x21] 

            leave 
            endp    asmMain 
```

u64ToBuf 函数❶是 u64ToStr 的一个变体，它不会保存任何寄存器。它会覆盖 X0 到 X7 寄存器，调用者需要负责保存任何需要保留的寄存器。

该函数将常数 10 初始化到 X4 ❷，因为每次数字转换都将除以并乘以该常数，而该常数必须存储在寄存器中。将 X4 保留为常数可以避免代码每次都重新加载该常数。此代码将 X5、X6 和 X7 清零，这些寄存器将存储转换后的字符串字符；这也初始化了零终止字节（根据输出数字的数量，可能位于这些寄存器的不同位置）。

该函数通过使用与清单 9-6 ❸ 中程序相同的基本“除法和余数”算法，将二进制数转换为一串数字字符。函数将值除以 10，余数是一个范围在 0 到 9 之间的值，函数将其转换为相应的 ASCII 字符。代码将转换后的数字移到 X5、X6 或 X7 寄存器的最终输出位置。1 到 6 位数字，即高位数字，最终存储在 X5 中；7 到 14 位数字存储在 X6 中；15 到 20 位数字存储在 X7 中。零字节填充所有未使用的数字位置。例如，如果数字只有三位，X6 和 X7 将包含 0，而 X5 中的 24 到 63 位将全部为 0。

每个可能的输出数字转换使用一组单独的除法/余数指令（因此称为 *展开/直线代码*） ❹。每个数字的转换序列大致相同，尽管有两个变体在 X1 和 X2 之间交替，因为除法的商成为下一步要除的值。每当商变为 0，转换完成，控制转移到不同的位置，将转换后的数字写入缓冲区。函数中只会采取单个分支，因为这些分支会继续执行下一条指令序列，直到转换完成。此外，这些数字转换序列可能会根据数字的最终位置将转换后的数字放入不同的输出寄存器。

如果代码执行到第 20 位数字，没有对 0 结果进行测试；此时商将始终为 0，因此函数简单地将数字存储到缓冲区中并返回 ❺。

如果数字有六位或更少，函数将 X5 中的字符写入缓冲区 ❻。X5 将始终包含数字的低位数字。通过将最多六个字符放入 X5，X5 的高位 2 字节将始终为 0（并为更大的字符串提供零终止字节）。对于少于六位的数字，代码必须显式地将零终止字节写入缓冲区。对于 7 到 14 位的数字，函数将寄存器 X6 和 X5（按此顺序）写入缓冲区 ❼。X5 提供零终止字节，因此代码不需要显式地写入任何 0 字节。对于 15 位或更多位的数字，代码将寄存器 X7、X6 和 X5 中的数据写出（X5 提供零终止字节） ❽。

实际的 u64ToStr 函数❾是一个简单的外观函数，它在调用 u64ToBuf 时保留所有寄存器的值。通过将 u64ToStr 分解为这两个函数，如果你希望 X0 指向字符串的末尾，可以直接调用 u64ToBuf（尽管如果需要，你必须保留 X1 到 X7）。此外，将寄存器保存代码放入 u64ToStr 中，允许 u64ToBuf 代码避免在所有 ret 指令之前恢复寄存器（或者避免再次跳转到处理恢复寄存器的代码）。

以下是来自清单 9-7 的构建命令和示例输出：

```
$ ./build Listing9-7 
$ time ./Listing9-7 
Calling Listing9-7: 
low=0, hi=9 
low=1, hi=9 
low=10, hi=99 
low=100, hi=999 
low=1000, hi=9999 
low=1000, hi=9999 
low=100000, hi=999999 
low=1000000, hi=9999999 
low=10000000, hi=99999999 
low=100000000, hi=999999999 
low=1000000000, hi=9999999999 
low=10000000000, hi=99999999999 
low=100000000000, hi=999999999999 
low=1000000000000, hi=9999999999999 
low=10000000000000, hi=99999999999999 
low=100000000000000, hi=999999999999999 
low=1000000000000000, hi=9999999999999999 
low=10000000000000000, hi=99999999999999999 
low=100000000000000000, hi=999999999999999999 
low=1000000000000000000, hi=9999999999999999999 
low=10000000000000000000, hi=18446744073709551615 
Listing9-7 terminated 
```

我修改了 u64toStr 的两个版本，以便对它们的执行时间进行计时。在递归版本中，我在我的 Mac mini 上获得了以下的计时结果：

```
Listing9-7a  404.58s user 0.42s system 99% cpu 6:46.25 total 
```

对于直线代码，运行时间如下：

```
Listing9-7a  173.60s user 0.15s system 99% cpu 2:53.78 total 
```

后者的代码运行速度比递归版本快了约 2.3 倍，这是一个很大的进步。

我还创建了一个版本的 u64ToStr，它首先计算输出数字的数量（使用二分查找），然后跳转到适当的代码以精确地转换该数量的数字。可惜的是，代码比清单 9-7 运行得稍慢。我还尝试了一个变体，它首先输出高位数字（通过 1e+19 进行除法，接下来每次除以 10）。它比数字计数版本略快，比清单 9-7 略慢。我已将两个实验的源代码包含在在线文件中，供你参考。

#### 9.1.4 将带符号整数值转换为字符串

要将带符号的整数值转换为字符串，首先检查数字是否为负。如果是，则输出一个短横线（-）字符并取反该值，然后调用 u64toStr 函数完成转换。清单 9-8 显示了相关代码。

```
// Listing9-8.S 

`Code taken from Listing 9-7 goes here.` 

// i64ToStr 
//
//  Converts a signed 64-bit integer to a string 
//  If the number is negative, this function will 
//  print a '-' character followed by the conversion 
//  of the absolute value of the number. 
//
// Inputs: 
//
//      X0- Pointer to buffer to hold the result. 
//          Buffer should be capable of receiving 
//          as many as 22 bytes (including zero-
//          terminating byte). 
//      X1- Signed 64-bit integer to convert 
//
// Outputs: 
//
//      Buffer- Contains the converted string 

            proc    i64ToStr 

            locals  i64 
            dword   i64.x0 
            byte    i64.stk, 32 
            endl    i64 

            enter   i64.size 

            // Need to preserve X1 in 
            // case this code negates it. 

            str     x1, [fp, #i64.x0] 

            cmp     x1, #0 
            bpl     isPositive 

            mov     w1, #'-'    // Emit '-' 
            strb    w1, [x0], #1 

            // Negate X0 and convert 
            // unsigned value to integer: 

            ldr     x1, [fp, #i64.x0] 
            neg     x1, x1 

isPositive: bl      u64ToStr 
            ldr     x1, [fp, #i64.x0] 
            leave 
            endp    i64ToStr 

`Code taken from Listing 9-7 goes here.` 
```

清单 9-8 仅显示了 i64ToStr 函数（程序的其余部分来自清单 9-7）。完整的源代码可以在线获取。

#### 9.1.5 扩展精度无符号整数转换为字符串

整个字符串转换算法中唯一需要扩展精度算术的操作是除以 10。清单 9-9 实现了一个利用该技术的 128 位十进制输出例程。我修改了第八章中的 div128 算法，使其进行显式的除以 10 操作（稍微加快了 div128 的速度），并修改了来自清单 9-6 的递归转换例程以执行该转换。

```
// Listing9-9.S 
//
// u128toStr function 

            #include    "aoaa.inc"

            .section    .rodata, "" 
ttlStr:     .asciz      "Listing 9-9"
fmtStr1:    .asciz      "Value = %s\n"

qdata:      .qword      1 
            .qword      21 
            .qword      302 
            .qword      4003 
            .qword      50004 
            .qword      600005 
            .qword      7000006 
            .qword      80000007 
            .qword      900000008 
            .qword      1000000009 
            .qword      11000000010 
            .qword      120000000011 
            .qword      1300000000012 
            .qword      14000000000013 
            .qword      150000000000014 
            .qword      1600000000000015 
            .qword      17000000000000016 
            .qword      180000000000000017 
            .qword      1900000000000000018 
            .qword      20000000000000000019 
            .qword      210000000000000000020 
            .qword      2200000000000000000021 
            .qword      23000000000000000000022 
            .qword      240000000000000000000023 
            .qword      2500000000000000000000024 
            .qword      26000000000000000000000025 
            .qword      270000000000000000000000026 
            .qword      2800000000000000000000000027 
            .qword      29000000000000000000000000028 
            .qword      300000000000000000000000000029 
            .qword      3100000000000000000000000000030 
            .qword      32000000000000000000000000000031 
            .qword      330000000000000000000000000000032 
            .qword      3400000000000000000000000000000033 
            .qword      35000000000000000000000000000000034 
            .qword      360000000000000000000000000000000035 
            .qword      3700000000000000000000000000000000036 
            .qword      38000000000000000000000000000000000037 
            .qword      300000000000000000000000000000000000038 
            .qword      340282366920938463463374607431768211455 
qcnt        =           (.-qdata)/16 

 .data 
buffer:     .space      256,0 

            .code 
            .extern     printf 

// Return program title to C++ program: 

            proc    getTitle, public 
            lea     x0, ttlStr 
            ret 
            endp    getTitle 

// div10 
//
// This procedure does a general 128-bit / 10 division operation 
// using the following algorithm (assume all variables except 
// Remainder are 128-bit objects; Remainder is 64 bits): 
//
// Quotient := Dividend; 
// Remainder := 0; 
// for i := 1 to NumberBits do 
//
//  Remainder:Quotient := Remainder:Quotient SHL 1; 
//  if Remainder >= 10 then 
//
//     Remainder := Remainder - 10; 
//     Quotient := Quotient + 1; 
//
//  endif 
// endfor 
//
// Data passed: 
//
// 128-bit dividend in X6:X5 
//
// Data returned: 
//
// 128-bit quotient in X6:X5 
// 64-bit remainder in X4 
//
// Modifies X1 

          ❶ proc    div10 

#define remainder  x4 
#define dividendL  x5 
#define dividendH  x6 
#define quotientL  dividendL 
#define quotientH  dividendH 

// Initialize remainder with 0: 

            mov     remainder, #0 

// Copy the dividend to local storage: 

            mov     w1, #128           // Count off bits in W0\. 

// Compute Remainder:Quotient := Remainder:Quotient LSL 1 
//
// Note: adds x, x, x is equivalent to lsl x, x, #1 
//       adcs x, x, x is equivalent to rol x, x, #1 
//                    (if rol existed) 
//
// The following four instructions perform a 256-bit 
// extended-precision shift (left) dividend through 
// remainder. 

repeatLp:   adds    dividendL, dividendL, dividendL 
            adcs    dividendH, dividendH, dividendH 
            adc     remainder, remainder, remainder 

// Do a comparison to see if the remainder 
// is greater than or equal to 10: 

            cmp     remainder, #10 
            blo     notGE 

// Remainder := Remainder - Divisor 

isGE:       sub     remainder, remainder, #10 

// Quotient := Quotient + 1 

            adds    quotientL, quotientL, #1 
            adc     quotientH, quotientH, xzr 

// Repeat for 128 bits: 

notGE:      subs    w1, w1, #1 
            bne     repeatLp 

            ret     // Return to caller. 
            endp    div10 

// u128toStr: 
//
//  Converts a 128-bit unsigned integer to a string 
//
//  Inputs: 
//      X0-     Pointer to buffer to receive string 
//      X1-     Points at the unsigned 128-bit integer to convert 
//
//  Outputs: 
//      Buffer- Receives the zero-terminated string 
//
//  Buffer must have at least 40 bytes allocated for it. 

 ❷ proc    u128toStr 
            stp     x0, x1, [sp, #-16]! 
            stp     x4, x5, [sp, #-16]! 
            stp     x6, lr, [sp, #-16]! 

            ldp     x5, x6, [x1]    // Test value for 0\. 
            orr     x4, x5, x6 
            cmp     x4, xzr         // Z = 1 if X6:X5 is 0\. 
            bne     doRec128 

            // Special case for zero, just write 
            // "0" to the buffer 

            mov     w4, #'0' 
            strb    w4, [x0], #1 
            b.al    allDone2 

doRec128:   bl      u128toStrRec    // X6:X5 contain value. 

            // Restore registers: 

allDone2:   strb    wzr, [x0]       // Zero-terminating byte 
            ldp     x6, lr, [sp], #16 
            ldp     x4, x5, [sp], #16 
            ldp     x0, x1, [sp], #16 
            ret 
            endp    u128toStr 

// u128toStrRec is the recursive version that handles 
// nonzero values. 
//
// Value to convert is passed in X6:X5\. 

          ❸ proc    u128toStrRec 
            stp     x4, lr, [sp, #-16]! 

            // Convert LO digit to a character: 

            bl      div10          // Quotient -> X6:X5, Rem -> W4 

            // Make recursive call if quotient is not 0: 

            orr     lr, x5, x6     // Use LR as a temporary. 
            cmp     lr, #0 
            beq     allDone 

            // New value is quotient (X6:X5) from above: 

            bl      u128toStrRec 

            // When this function has processed all the 
            // digits, write them to the buffer: 

allDone:    orr     w4, w4, #'0'    // Convert to char. 
            strb    w4, [x0], #1    // Bump pointer after store. 

 // Restore state and return: 

            ldp     x4, lr, [sp], #16    // Restore prev char. 
            ret 
            endp    u128toStrRec 

// Here is the asmMain function. 

            proc    asmMain, public 

            locals  am 
            dword   am.x2021 
            byte    stk, 64 
            endl    am 

            enter   am.size              // Reserve space on stack. 

            stp     x20, x21, [fp, #am.x2021] 

            lea     x20, qdata 
            mov     x21, #qcnt 
loop:       mov     x1, x20 
            lea     x0, buffer 
            bl      u128toStr 

            lea     x1, buffer 
            mstr    x1, [sp] 
            lea     x0, fmtStr1 
            bl      printf 

            add     x20, x20, #16       // Next value to convert 
            subs    x21, x21, #1 
            bne     loop 

            ldp     x20, x21, [fp, #am.x2021] 
            leave 
            ret 
            endp    asmMain 
```

代码包括一个优化版的 128 位除法函数，它将数字除以 10❶。接下来是 u128toStr 的非递归入口点，它将 0 作为特例处理，并对所有其他值调用递归版本❷，然后是 u128toStr 的递归代码❸。由于这些函数几乎与递归的 64 位字符串输出函数相同，请参考清单 9-6 中的代码了解更多细节。

u128toStr 函数的一个问题是，它比其他数字到字符串的函数慢得多。这完全是因为 div10 子程序的性能。由于 128 位除以 10 的算法非常慢，我不会费力改进 u128toStr 转换函数的性能。除非你能想出一个高性能的 div10 子程序（或许可以使用倒数相乘；请参见第 9.6 节“更多信息”，第 603 页），否则优化 u128toStr 可能是浪费时间。幸运的是，这个函数可能不会经常被调用，所以它的性能不会有太大影响。

这是来自清单 9-9 的构建命令和示例输出：

```
$ ./build Listing9-9 
$ ./Listing9-9 
Calling Listing9-9: 
Value = 1 
Value = 21 
Value = 302 
Value = 4003 
Value = 50004 
Value = 600005 
Value = 7000006 
Value = 80000007 
Value = 900000008 
Value = 1000000009 
Value = 11000000010 
Value = 120000000011 
Value = 1300000000012 
Value = 14000000000013 
Value = 150000000000014 
Value = 1600000000000015 
Value = 17000000000000016 
Value = 180000000000000017 
Value = 1900000000000000018 
Value = 20000000000000000019 
Value = 210000000000000000020 
Value = 2200000000000000000021 
Value = 23000000000000000000022 
Value = 240000000000000000000023 
Value = 2500000000000000000000024 
Value = 26000000000000000000000025 
Value = 270000000000000000000000026 
Value = 2800000000000000000000000027 
Value = 29000000000000000000000000028 
Value = 300000000000000000000000000029 
Value = 3100000000000000000000000000030 
Value = 32000000000000000000000000000031 
Value = 330000000000000000000000000000032 
Value = 3400000000000000000000000000000033 
Value = 35000000000000000000000000000000034 
Value = 360000000000000000000000000000000035 
Value = 3700000000000000000000000000000000036 
Value = 38000000000000000000000000000000000037 
Value = 300000000000000000000000000000000000038 
Value = 340282366920938463463374607431768211455 
Listing9-9 terminated 
```

由于代码几乎与 i64toStr 相同（见清单 9-8），我会留给你来创建一个 128 位有符号整数转换函数；你只需提供 128 位取反和比较操作。作为提示，对于比较，只需检查 HO 字（高位字）是否设置了符号位。

#### 9.1.6 格式化转换

前面章节中的代码通过使用最小的字符位置数将有符号和无符号整数转换为字符串。为了创建格式良好的数值表，你需要编写函数，在实际输出数字之前，先在数字字符串前添加适当的填充。等你有了这些“未格式化”版本的例程，实现格式化版本就很容易了。

第一步是编写 iSize 和 uSize 例程，计算显示值所需的最小字符位置数。实现此功能的一种算法类似于数字字符串转换例程。唯一的区别是，在进入例程时将计数器初始化为 0，并且在每次递归调用时增加该计数器，而不是输出一个数字。（如果数字为负，请不要忘记在 iSize 内部递增计数器；你必须考虑到负号的输出。）计算完成后，这些例程应返回操作数在 X0 寄存器中的大小。

然而，由于它使用了递归和除法，因此这种转换方案较慢。清单 9-10 展示了使用二分查找的暴力转换方法。

```
// Listing9-10.S 
//
// u64Size function: Computes the size 
// of an unsigned 64-bit integer (in 
// print positions) 

            #include    "aoaa.inc"

            .section    .rodata, "" 
ttlStr:     .asciz      "Listing 9-10"
fmtStr:     .asciz      "Value = %llu, size=%d\n"

// Values to test the u64Size function: 

dVals:      .dword      1 
            .dword      10 
            .dword      100 
            .dword      1000 
            .dword      10000 
            .dword      100000 
            .dword      1000000 
            .dword      10000000 
            .dword      100000000 
            .dword      1000000000 
            .dword      10000000000 
            .dword      100000000000 
            .dword      1000000000000 
            .dword      10000000000000 
            .dword      100000000000000 
            .dword      1000000000000000 
            .dword      10000000000000000 
 .dword      100000000000000000 
            .dword      1000000000000000000 
            .dword      10000000000000000000 
dCnt        =           (.-dVals) / 8 

            .code 
            .extern     printf 

// Return program title to C++ program: 

            proc    getTitle, public 
            lea     x0, ttlStr 
            ret 
            endp    getTitle 

// u64Size 
//
//  Counts the number of output positions 
//  required for an integer-to-decimal-
//  string conversion 
//
//  Uses a binary search to quickly 
//  count the digits required by a value 
//
// Input: 
//  X1- Unsigned integer to count 
//
// Output: 
//  X1- Digit count 
//
// Table of digit counts and values: 
//
//   1: 1 
//   2: 10 
//   3: 100 
//   4: 1,000 
//   5: 10,000 
//   6: 100,000 
//   7: 1,000,000 
//   8: 10,000,000 
//   9: 100,000,000 
//  10: 1,000,000,000 
//  11: 10,000,000,000 
//  12: 100,000,000,000 
//  13: 1,000,000,000,000 
//  14: 10,000,000,000,000 
//  15: 100,000,000,000,000 
//  16: 1,000,000,000,000,000 
//  17: 10,000,000,000,000,000 
//  18: 100,000,000,000,000,000 
//  19: 1,000,000,000,000,000,000 
//  20: 10,000,000,000,000,000,000 

          ❶ proc    u64Size 
            stp     x0, x2, [sp, #-16]! 

 ❷ mov     x2, x1 
            ldr     x0, =1000000000 // 10: 1,000,000,000 
            cmp     x2, x0 
            bhs     ge10 

            ldr     x0, =10000 
            cmp     x2, x0 
            bhs     ge5 

            // Must be 1 to 4 digits here: 

            mov     x1, #1 
            cmp     x2, #1000 
            cinc    x1, x1, hs 
            cmp     x2, #100 
            cinc    x1, x1, hs 
            cmp     x2, #10 
            cinc    x1, x1, hs 
            ldp     x0, x2, [sp], #16 
            ret 

// Must be 5 to 9 digits here: 

ge5:        ldr     x0, =1000000    // 7: 1,000,000 
            cmp     x2, x0 
            bhs     ge7 

            // Must be 5 or 6 digits: 

            mov     x1, #5 
            ldr     x0, =100000     // 6: 100,000 
            cmp     x2, x0 
            cinc    x1, x1, hs 
            ldp     x0, x2, [sp], #16 
            ret 

// Must be 7 to 9 digits here: 

ge7:        mov     x1, #7 
            ldr     x0, =10000000   // 8: 10,000,000 
            cmp     x2, x0 
            cinc    x1, x1, hs 
            ldr     x0, =100000000  // 9: 100,000,000 
            cmp     x2, x0 
            cinc    x1, x1, hs 
            ldp     x0, x2, [sp], #16 
            ret 

// Handle 10 or more digits here: 

ge10:       ldr     x0, =100000000000000    // 15: 100,000,000,000,000 
            cmp     x2, x0 
            bhs     ge15 

 // 10 to 14 digits here: 

            ldr     x0, =1000000000000      // 13: 1,000,000,000,000 
            cmp     x2, x0 
            bhs     ge13 

            // 10 to 12 digits here: 

            mov     x1, #10 
            ldr     x0, =10000000000        // 11: 10,000,000,000 
            cmp     x2, x0 
          ❸ cinc    x1, x1, hs 
            ldr     x0, =100000000000       // 12: 100,000,000,000 
            cmp     x2, x0 
            cinc    x1, x1, hs 
            ldp     x0, x2, [sp], #16 
            ret 

// 13 or 14 digits here: 

ge13:       mov     x1, #13 
            ldr     x0, =10000000000000     // 14: 10,000,000,000,000 
            cmp     x2, x0 
            cinc    x1, x1, hs 
            ldp     x0, x2, [sp], #16 
            ret 

// 15 to 20 digits here: 

ge15:       ldr     x0, =100000000000000000 // 18: 100,000,000,000,000,000 
            cmp     x2, x0 
            bhs     ge18 

            // 15, 16, or 17 digits here: 

            mov     x1, #15 
            ldr     x0, =1000000000000000   // 16: 1,000,000,000,000,000 
            cmp     x2, x0 
            cinc    x1, x1, hs 
            ldr     x0, =10000000000000000  // 17: 10,000,000,000,000,000 
            cmp     x2, x0 
            cinc    x1, x1, hs 
            ldp     x0, x2, [sp], #16 
            ret 

// 18 to 20 digits here: 

ge18:       mov     x1, #18 
            ldr     x0, =1000000000000000000  // 19: 1,000,000,000,000,000,000 
            cmp     x2, x0 
            cinc    x1, x1, hs 
            ldr     x0, =10000000000000000000 // 20 digits 
            cmp     x2, x0 
            cinc    x1, x1, hs 
 ldp     x0, x2, [sp], #16 
            ret 
            endp    u64Size 
```

实际的 u64Size 函数❶使用二分查找算法快速扫描所有可能的值，以确定数字的位数。它通过将搜索空间对半分，比较输入值（移到 X2）与一个 10 位数字值❷来开始。在常规的二分查找方式中，代码的两部分将分别测试 1 到 9 位数字和 10 到 20 位数字。在这些范围内，搜索将（大致）反复对半分割，直到算法锁定确切的数字位数。当代码处理到 2 到 4 位数字时，它使用一些直线代码和一系列 cinc 指令，以快速处理最后几个案例，而无需执行分支❸。

这是构建命令和示例输出：

```
$ ./build Listing9-10 
$ ./Listing9-10 
Calling Listing9-10: 
Value = 1, size=1 
Value = 10, size=2 
Value = 100, size=3 
Value = 1000, size=4 
Value = 10000, size=5 
Value = 100000, size=6 
Value = 1000000, size=7 
Value = 10000000, size=8 
Value = 100000000, size=9 
Value = 1000000000, size=10 
Value = 10000000000, size=11 
Value = 100000000000, size=12 
Value = 1000000000000, size=13 
Value = 10000000000000, size=14 
Value = 100000000000000, size=15 
Value = 1000000000000000, size=16 
Value = 10000000000000000, size=17 
Value = 100000000000000000, size=18 
Value = 1000000000000000000, size=19 
Value = 10000000000000000000, size=20 
Listing9-10 terminated 
```

对于有符号整数，将清单 9-11 中的函数添加到清单 9-10 中的代码中（你可以在本书的可下载代码文件中找到完整的清单 9-11，地址是*[`<wbr>artofarm<wbr>.randallhyde<wbr>.com`](https://artofarm.randallhyde.com)*）。

```
// Listing9-11.S 
//
// i64Size: 
//
// Computes the number of character positions that 
// the i64toStr function will emit 

 proc    i64Size 
            str     lr, [sp, #-16]! 

            cmp     x1, #0          // If less than zero, 
            bge     isPositive      // negate and treat 
                                    // like an uns64\. 
            neg     x1, x1 

            bl      u64Size 
            add     x1, x1, #1      // Adjust for "-". 
            ldr     lr, [sp], #16 
            ret 

isPositive: bl      u64Size 
            ldr     lr, [sp], #16 
            ret 
            endp    i64Size 
```

对于扩展精度大小操作，二分查找方法很快会变得难以处理（64 位已经足够糟糕了）。最佳解决方案是将你的扩展精度值除以一个 10 的幂（比如，1e+16）。这将把数字的大小减少 16 位数字。只要商大于 64 位，就重复这个过程，并跟踪你每次用 1e+16 进行除法操作时减少的位数。当商适合 64 位时（19 或 20 位），调用 64 位的 u64Size 函数，并加上你通过除法操作减少的数字位数（每次除以 1e+16 减少 16 位）。这个实现我留给你来完成。

一旦你有了 i64Size 和 u64Size 例程，编写格式化输出例程 u64toStrSize 或 i64toStrSize 就很简单了。初始进入时，这些例程会调用相应的 i64Size/u64Size 例程来确定数字所需的字符位置数。如果 i64Size/u64Size 例程返回的值大于或等于最小大小参数（传递给 u64toStrSize 或 i64toStrSize 的值），则无需其他格式化。如果参数 size 的值大于 i64Size/u64Size 返回的值，程序必须计算这两个值之间的差异，并在数字转换之前输出相应数量的空格（或其他填充字符）（假设是右对齐，这正是本章所介绍的）。

清单 9-12 展示了 utoStrSize/itoStrSize 函数（完整的源代码在线上可以找到）；在这里，我省略了除了 utoStrSize/itoStrSize 函数本身之外的所有内容。

```
// Listing9-12.S (partial) 
//
// u64ToSizeStr 
//
//  Converts an unsigned 64-bit integer to 
//  a character string, using a minimum field 
//  width 
//
//  Inputs: 
//      X0- Pointer to buffer to receive string 
//
//      X1- Unsigned 64-bit integer to convert 
//          to a string 
//
//      X2- Minimum field width for the string 
//          (maximum value is 1,024). Note: if 
//          the minimum field width value is less 
//          than the actual output size of the 
//          integer, this function will ignore 
//          the value in X2 and use the correct 
//          number of output positions for the 
//          value. 
//
//  Outputs: 
//
//      Buffer- Receives converted characters. 
//              Buffer must be at least 22 bytes 
//              or X1 + 1 bytes long. 

          ❶ proc    u64ToStrSize 
            stp     x0, lr, [sp, #-16]! 
            stp     x1, x2, [sp, #-16]! 
            stp     x23, x24, [sp, #-16]! 
            stp     x25, x26, [sp, #-16]! 

            // Initialize x25 and x26 with 
            // appropriate functions to call: 

            lea     x25, u64Size 
            lea     x26, u64ToStr 

            b.al    toSizeStr 
            endp    u64ToStrSize 

/////////////////////////////////////////////////////
//
// i64ToStrSize: 
//
//  Just like u64ToStrSize, but handles signed integers 
//
//  Inputs: 
//      X0- Pointer to buffer to receive string 
//
//      X1- Signed 64-bit integer to convert 
//          to a string 
//
//      X2- Minimum field width for the string 
//          (maximum value is 1,024). Note: if 
//          the minimum field width value is less 
//          than the actual output size of the 
//          integer, this function will ignore 
//          the value in X2 and use the correct 
//          number of output positions for the 
//          value. 
//
//      Note:   Don't forget that if the number 
//              is negative, the '-' consumes 
//              an output position. 
//
//  Outputs: 
//      Buffer- Receives converted character. 
//              Buffer must be at least 22 bytes 
//              or X2 + 1 bytes long. 

          ❷ proc    i64ToStrSize 
            stp     x0, lr, [sp, #-16]! 
            stp     x1, x2, [sp, #-16]! 
            stp     x23, x24, [sp, #-16]! 
            stp     x25, x26, [sp, #-16]! 

            // Initialize x25 and x26 with 
            // appropriate functions to call: 

            lea     x25, i64Size 
            lea     x26, i64ToStr 

            b.al    toSizeStr   // Technically, this could just fall through. 
            endp    i64ToStrSize 

///////////////////////////////////////////////////////
//
// toSizeStr: 
//
//  Special function to handle signed and 
//  unsigned conversions for u64ToSize and i64ToSize 

          ❸ proc    toSizeStr 

            mov     x24, x1 // Save for now. 
          ❹ blr     x25     // Compute size of number. 

            // Compute difference between actual size 
            // and desired size. Set to the larger of 
            // the two: 

          ❺ cmp     x2, x1 
            csel    x23, x2, x1, ge 

            // Just as a precaution, limit the 
            // size to 1,024 characters (including 
            // the zero-terminating byte): 

            mov     x2, #1023   // Don't count 0 byte here. 
            cmp     x23, x2 
            csel    x23, x23, x2, ls 

 // Compute the number of spaces to emit before 
            // the first digit of the number: 

            subs    x23, x23, x1 
            beq     spacesDone 

            // Emit that many spaces to the buffer: 

          ❻ mov     x1, #0x2020 
            movk    x1, #0x2020, lsl #16 
            movk    x1, #0x2020, lsl #32 
            movk    x1, #0x2020, lsl #48 
            b.al    tst8 

            // Handle sequences of eight spaces: 

whl8:       str     x1, [x0], #8 
            sub     x23, x23, #8 
tst8:       cmp     x23, #8 
            bge     whl8 

            // If four to seven spaces, emit four 
            // spaces here: 

            cmp     x23, #4 
            blt     try2 
            str     w1, [x0], #4 
            sub     x23, x23, #4 

            // If two or three spaces, emit two 
            // here: 

try2:       cmp     x23, #2 
            blt     try1 
            strh    w1, [x0], #2 
            sub     x23, x23, #2 

            // If one space left, emit it here: 

try1:       cmp     x23, #1 
            blt     spacesDone 
            strb    w1, [x0], #1 

            // Okay, emit the digits here: 

spacesDone: mov     x1, x24 // Retrieve value. 
          ❼ blr     x26     // XXXToStr 

            ldp     x25, x26, [sp], #16 
            ldp     x23, x24, [sp], #16 
            ldp     x1, x2,   [sp], #16 
            ldp     x0, lr,   [sp], #16 
            ret 
            endp    toSizeStr 

///////////////////////////////////////////////////////
//
// printSize 
//
// Utility used by the main program to 
// compute sizes and print them 

          ❽ proc    printSize 

            locals  ps 
            dword   stk, 64 
            endl    ps 

            enter   ps.size 

            mov     x6, x1 
            lea     x0, buffer 
            blr     x27         // Call XXXToStrSize. 

            mov     x1, x6 
            mstr    x1, [sp] 
            mstr    x2, [sp, #8] 
            lea     x3, buffer 
            mstr    x3, [sp, #16] 
            lea     x0, fmtStr 
            bl      printf 

            leave 
            endp    printSize 

values:     .dword  1, 10, 100, 1000, 10000, 100000, 1000000 
            .dword  10000000, 100000000, 1000000000, 10000000000 
            .dword  100000000000, 1000000000000, 10000000000000 
            .dword  100000000000000, 1000000000000000 
            .dword  10000000000000000, 100000000000000000 
            .dword  1000000000000000000, 10000000000000000000 
            .dword  0x7fffffffffffffff 
            .set    valSize, (.-values)/8 

negValues:  .dword  -1, -10, -100, -1000, -10000, -100000, -1000000 
            .dword  -10000000, -100000000, -1000000000, -10000000000 
            .dword  -100000000000, -1000000000000, -10000000000000 
            .dword  -100000000000000, -1000000000000000 
            .dword  -10000000000000000, -100000000000000000 
            .dword  -1000000000000000000, -10000000000000000000 
            .dword  0x8000000000000000 

sizes:      .word   5, 6, 7, 8, 9, 10, 15, 15, 15, 15 
            .word   20, 20, 20, 20, 20, 25, 25, 25, 25, 25, 30 

///////////////////////////////////////////////////////
//
// Here is the asmMain function: 

 ❾ proc    asmMain, public 

            locals  am 
            qword   am.x26x27 
            qword   am.x24x25 
            byte    am.stk, 64 
            endl    am 

            enter   am.size     // Activation record 
            stp     x26, x27, [fp, #am.x26x27] 
            stp     x24, x25, [fp, #am.x24x25] 

// Test unsigned integers: 

            lea     x27, u64ToStrSize 
            lea     x24, values 
            lea     x25, sizes 
            mov     x26, #valSize 
tstLp:      ldr     x1, [x24], #8 
            ldr     w2, [x25], #4 
            bl      printSize 
            subs    x26, x26, #1 
            bne     tstLp 

            lea     x27, i64ToStrSize 
            lea     x24, negValues 
            lea     x25, sizes 
            mov     x26, #valSize 
ntstLp:     ldr     x1, [x24], #8 
            ldr     w2, [x25], #4 
            bl      printSize 
            subs    x26, x26, #1 
            bne     ntstLp 

            ldp     x26, x27, [fp, #am.x26x27] 
            ldp     x24, x25, [fp, #am.x24x25] 
            leave 
            endp    asmMain 
```

u64toStrSize 函数❶只需加载 X25 和 X26 为适当的地址，并跳转到通用的 toSizeStr 函数以处理实际工作。i64ToStrSize 函数❷对有符号整数转换做同样的事情。

toSizeStr 函数❸处理了实际工作。首先，它调用适当的 toSize 函数（该函数的地址通过 X25 传递）来计算值所需的最小打印位置数❹。然后，它计算出需要多少个填充字符才能将数字右对齐到输出字段中❺。它在输出数字字符串❼之前，先输出所需数量的填充字符❻。值得注意的唯一一点是，代码尝试每次输出八个空格以提高性能，只要至少有八个填充字符，然后是四个，再是两个，最后是一个。

printSize 过程❽是一个小工具函数，asmMain 过程用它来显示值，而 asmMain 过程❾测试了 u64ToStrSize 和 i64ToStrSize 过程。

以下是清单 9-12 的构建命令和示例输出（请记住，实际的主程序仅出现在在线源代码中）：

```
$ ./build Listing9-12 
$ ./Listing9-12 
Calling Listing9-12: 
                   1:   5='    1' 
                  10:   6='    10' 
                 100:   7='    100' 
                1000:   8='    1000' 
               10000:   9='    10000' 
              100000:  10='    100000' 
             1000000:  15='        1000000' 
            10000000:  15='       10000000' 
           100000000:  15='      100000000' 
          1000000000:  15='     1000000000' 
         10000000000:  20='         10000000000' 
        100000000000:  20='        100000000000' 
       1000000000000:  20='       1000000000000' 
      10000000000000:  20='      10000000000000' 
     100000000000000:  20='     100000000000000' 
    1000000000000000:  25='         1000000000000000' 
   10000000000000000:  25='        10000000000000000' 
  100000000000000000:  25='       100000000000000000' 
 1000000000000000000:  25='      1000000000000000000' 
-8446744073709551616:  25='     10000000000000000000' 
 9223372036854775807:  30='           9223372036854775807' 
                  -1:   5='   -1' 
                 -10:   6='   -10' 
                -100:   7='   -100' 
               -1000:   8='   -1000' 
              -10000:   9='   -10000' 
             -100000:  10='   -100000' 
            -1000000:  15='       -1000000' 
           -10000000:  15='      -10000000' 
          -100000000:  15='     -100000000' 
         -1000000000:  15='    -1000000000' 
        -10000000000:  20='        -10000000000' 
       -100000000000:  20='       -100000000000' 
      -1000000000000:  20='      -1000000000000' 
     -10000000000000:  20='     -10000000000000' 
    -100000000000000:  20='    -100000000000000' 
   -1000000000000000:  25='        -1000000000000000' 
  -10000000000000000:  25='       -10000000000000000' 
 -100000000000000000:  25='      -100000000000000000' 
-1000000000000000000:  25='     -1000000000000000000' 
 8446744073709551616:  25='      8446744073709551616' 
-9223372036854775808:  30='          -9223372036854775808' 
Listing9-12 terminated 
```

输出是值：大小 = '转换'。### 9.2 将浮点值转换为字符串

到目前为止，本章已经讨论了将整数数值转换为字符字符串（通常是输出给用户）。本节讨论了将浮点值转换为字符串，这同样非常重要。

将浮点值转换为字符串可以有两种形式：

+   十进制表示法转换（例如 ±xxx.yyy 格式）

+   指数（或科学）表示法转换（例如 ±x.yyyyye±zz 格式）

无论最终的输出格式如何，您需要进行两项不同的操作，将浮点值转换为字符字符串。首先，您必须将尾数转换为适当的数字字符串。其次，您需要将指数转换为数字字符串。

然而，这并不是将两个整数值简单地转换为十进制字符串并将它们连接在一起（在尾数和指数之间用 *e*）。首先，尾数不是一个整数值，它是一个定点分数二进制值。仅仅将它视为一个 *n* 位二进制值（其中 *n* 是尾数位数）几乎总会导致转换错误。其次，虽然指数或多或少是一个整数值，但它表示的是 2 的幂，而不是 10 的幂。将这个 2 的幂作为整数值表示并不适合十进制浮点表示。上述这两个问题（分数尾数和二进制指数）是将浮点值转换为字符串时的主要复杂性来源。

> 注意

*指数实际上是一个带偏移的指数值。然而，这很容易转换为一个带符号的二进制整数。*

双精度浮点值具有 53 位尾数（包括隐含位）。这不是一个 53 位的整数，而是这 53 位表示从 1.0 到略小于 2.0 之间的值。（有关 IEEE 64 位浮点格式的更多细节，请参见第 2.13 节“IEEE 浮点格式”，第 93 页。）双精度格式可以表示从 0 到大约 5 × 10^(–324) 的数字（使用标准化值时，大约为 ±1 × 10^(±308)）。

为了以大约 16 位精度输出尾数的十进制形式，依次将浮点值乘以或除以 10，直到该数字的范围从 1e+15 到略小于 1e+16（即 9.9999 … e+15）。一旦指数进入适当的范围，尾数位就形成一个 16 位整数值（没有小数部分），可以将其转换为十进制字符串，得到构成尾数值的 16 位数字。

为了将指数转换为适当的十进制字符串，跟踪乘以或除以 10 的次数。每次除以 10 时，十进制指数值加 1；每次乘以 10 时，十进制指数值减 1。完成该过程后，从十进制指数值中减去 16（因为这个过程产生的值其指数为 16），然后将十进制指数值转换为字符串。

以下各节中的转换假设你始终希望生成一个具有 16 位有效数字的尾数。要生成格式化输出并且有效数字少，请参见第 9.2.4 节，“双精度值转换为字符串”，下一页。

#### 9.2.1 将浮点指数转换为十进制数字字符串

为了将指数转换为十进制数字字符串，使用以下算法：

1.  如果数字是 0.0，直接生成尾数输出字符串“0000000000000000”（注意字符串开头的空格），将指数设置为 0，完成。否则，继续执行以下步骤。

2.  将十进制指数初始化为 0。

3.  如果指数为负数，输出一个连字符（-）并将值取负；如果指数为正数，输出一个空格字符。

4.  如果（可能是取负的）指数值小于 1.0，则跳到第 8 步。

5.  正指数：将数字与逐渐较小的 10 的幂进行比较，从 10^(+256)开始，然后是 10^(+128)，然后是 10^(+64)，然后……最后是 10⁰。每次比较后，如果当前值大于该 10 的幂，则除以该 10 的幂，并将该 10 的幂指数（256、128、……、0）加到十进制指数值中。

6.  重复第 5 步，直到指数为 0（即值的范围为 1.0 ≤ *value* < 10.0）。

7.  跳到第 10 步。

8.  负指数：将数字与从 10^(–256)开始的逐渐较大的 10 的幂进行比较，然后是 10^(–128)，然后是 10^(–64)，然后……最后是 10⁰。每次比较后，如果当前值小于该 10 的幂，则除以该 10 的幂，并将该 10 的幂指数（256、128、……、0）从十进制指数值中减去。

9.  重复第 8 步，直到指数为 0（即值的范围为 1.0 ≤ *value* < 10.0）。

10.  此时，指数值是一个合理的数字，可以通过使用标准的无符号到字符串转换将其转换为整数值（参见第 9.1.3 节，“无符号十进制值转换为字符串”，第 495 页）。

#### 9.2.2 将浮点尾数转换为数字字符串

为了将尾数转换为数字字符串，不能简单地将前一节中产生的 53 位尾数当作整数值来处理，因为它仍然表示一个从 1.0 到小于 2.0 的整数。然而，如果将该浮点值（已转换为从 1.0 到略小于 10.0 的值）乘以 10^(+15)，则实际上会生成一个整数，并且尾数的数字会向左移动 15 个打印位置（16 位数字是双精度值能够输出的数字位数）。然后可以将这个“整数”转换为字符串。结果将包括 16 个尾数字符。要将尾数转换为字符串，请执行以下步骤：

1.  将前一节中指数计算得到的值乘以 1e+15。这会产生一个数字，将小数位左移 15 个打印位置。

2.  获取 52 位尾数，并将一个隐式的 52 位比特设置为 1，然后将该 53 位值进行零扩展到 64 位。

3.  通过使用本章早些时候介绍的无符号整数到字符串的函数，将结果 64 位值转换为字符串（参见第 9.1.3 节“无符号十进制值到字符串”，在第 495 页）。

#### 9.2.3 十进制和指数格式的字符串

要生成一个十进制字符串（而不是指数形式的数字），剩下的任务是将小数点正确地放置在数字字符串中。如果指数大于或等于 0，需要将小数点插入到位置*指数* + 1，位置从前一节中生成的第一个尾数字符开始。例如，如果尾数转换结果为 1234567890123456，且指数为 3，则需要在索引 4 的位置（3 + 1）前插入小数点，结果将是 1234.567890123456。

如果指数大于 16，则在字符串末尾插入*指数* – 16 个零字符（或者如果不希望允许将大于 1e+16 的值转换为十进制形式，则返回错误）。如果指数小于 0，则在数字字符串前插入 0.，后跟*abs*（*exp*） – 1 个零字符。如果指数小于–16（或其他任意值），你可能选择返回错误或自动切换到指数形式。

生成指数输出比生成十进制输出稍微容易一些。始终在转换后的尾数字符串中的第一个和第二个字符之间插入小数点，然后在字符串后面加上 e±xxx，其中±xxx 是指数值的字符串转换。例如，如果尾数转换结果为 1234567890123456，且指数为–3，那么生成的字符串将是 1.234567890123456e-003（注意指数数字前的 0）。

#### 9.2.4 双精度值转换为字符串

本节展示了将双精度值转换为字符串的代码，可以是十进制或指数形式，分别为这两种输出格式提供了不同的函数。由于列表 9-13 比较长，我已将其拆分为多个部分，并对每个部分进行了注释。

```
// Listing9-13.S 
//
// Floating-point (double) to string conversion 
//
// Provides both exponential (scientific notation) 
// and decimal output formats 
            #include    "aoaa.inc"

          ❶ .section    .rodata, "" 
ttlStr:     .asciz      "Listing 9-13"
fmtStr1:    .asciz      "r64ToStr: value='%s'\n"
fmtStr2:    .asciz      "fpError: code=%lld\n"
fmtStr3:    .asciz      "e64ToStr: value='%s'\n"
newlines:   .asciz      "\n\n"
expStr:     .asciz      "\n\nTesting e64ToStr:\n\n"

// r10str_1: A global character array that will 
// hold the converted string 

          ❷ .data 
r64str_1:   .space      32, 0 

            .code 
            .extern     printf 

// tenTo15: Used to multiply a value from 1.0 
// to less than 2.0 in order to convert the mantissa 
// to an actual integer 

❸ tenTo15:    .double     1.0e+15 

// potPos, potNeg, and expTbl: 
//
// Power of 10s tables (pot) used to quickly 
// multiply or divide a floating-point value 
// by powers of 10\. expTbl is the power-of-
// 10 exponent (absolute value) for each of
// the entries in these tables. 

❹ potPos:     .double     1.0e+0 
            .double     1.0e+1 
            .double     1.0e+2 
            .double     1.0e+4 
            .double     1.0e+8 
            .double     1.0e+16 
            .double     1.0e+32 
            .double     1.0e+64 
            .double     1.0e+128 
            .double     1.0e+256 
expCnt      =           (.-potPos) / 8 

potNeg:     .double     1.0e-0 
            .double     1.0e-1 
            .double     1.0e-2 
            .double     1.0e-4 
            .double     1.0e-8 
            .double     1.0e-16 
            .double     1.0e-32 
 .double     1.0e-64 
            .double     1.0e-128 
            .double     1.0e-256 

expTbl:     .dword      0 
            .dword      1 
            .dword      2 
            .dword      4 
            .dword      8 
            .dword      16 
            .dword      32 
            .dword      64 
            .dword      128 
            .dword      256 

// Maximum number of significant digits for 
// a double-precision value: 

❺ maxDigits   =           16 

// Return program title to C++ program: 

          ❻ proc    getTitle, public 
            lea     x0, ttlStr 
            ret 
            endp    getTitle 
```

像本章中的示例程序一样，列表 9-13 以一个只读数据部分 ❶ 开头，其中包含程序的标题字符串和主程序中 printf() 调用所使用的各种格式字符串。此程序中的唯一数据变量是 r64str_1 ❷，它是一个 32 字节的字符字符串，用于存放转换后的字符串。程序负责确保所有转换都能适应 32 字节的空间。

列表 9-13 将几个只读常量放置在 .code 部分，以便程序可以通过使用相对 PC 定址模式直接访问这些常量（而不是使用多个指令获取对象的地址并间接访问它）。第一个这样的常量是 tenTo15 ❸，它保存值 1.0e+15\。转换代码使用这个常量将范围在 1.0 到略小于 10.0 之间的浮动点值乘以 1e+15，从而在将尾数转换为整数值时获得略小于 1e+16 的值。

potPos、potNeg 和 expTbl 表 ❹ 包含用于将浮动点值乘以不同 10 的幂次的正负 10 的幂次（*pot*）表，用于将值处理到 1.0 到 10.0 的范围内。expTbl 包含与 potPos 和 potNeg 表中的相同条目对应的指数的绝对值。代码在将尾数转换到 1.0 到 10.0 的范围时，会将此值加到或从累积的小数指数中减去。

maxDigits 清单常量 ❺ 指定了该转换代码支持的有效数字的数量（对于双精度浮动点数是 16 位）。最后，这段代码包含了无处不在的 getTitle 函数 ❻，它返回程序标题字符串的地址，供 C++ shell 代码使用。

以下代码将浮动点值转换为字符串：

```
// Listing9-13.S (cont.) 
//
// u53toStr 
//
//  Converts a 53-bit unsigned integer to a string containing 
//  exactly 16 digits (technically, it does 64-bit arithmetic, 
//  but is limited to 53 bits because of the 16-digit output 
//  format) 
//
// Inputs: 
//  X0-     Pointer to buffer to receive string 
//  X1-     Unsigned 53-bit integer to convert 
//
// Outputs: 
//  Buffer- Receives the zero-terminated string 
//  X0-     Points at zero-terminating byte in string 
//
//  Buffer must have at least 17 bytes allocated for it. 
//
// This code is a bit simplified from the u64toStr function 
// because it always emits exactly 16 digits 
// (never any leading 0s). 

          ❶ proc    u53toStr 

            stp     x1, x2, [sp, #-16]! 
            stp     x3, x4, [sp, #-16]! 
            str     x5, [sp, #-16]! 

            mov     x4, #10     // Mul/div by 10 using X4 
            mov     x5, xzr     // Holds string of 8 chars 

            // Handle LO digit here. Note that the LO 
            // digit will ultimately be moved into 
            // bit positions 56-63 of X5 because numeric 
            // strings are, intrinsically, big-endian (with 
            // the HO digit appearing first in memory). 

          ❷ udiv    x2, x1, x4      // X2 = quotient 
            msub    x3, x2, x4, x1  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 

            // The following is an unrolled loop 
            // (for speed) that processes the 
            // remaining 15 digits. 
            // 
 // Handle digit 1 here: 

            udiv    x1, x2, x4      // X1 = quotient 
            msub    x3, x1, x4, x2  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 

            // Handle digit 2 here: 

            udiv    x2, x1, x4      // X2 = quotient 
            msub    x3, x2, x4, x1  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 

            // Handle digit 3 here: 

            udiv    x1, x2, x4      // X1 = quotient 
            msub    x3, x1, x4, x2  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 

            // Handle digit 4 here: 

            udiv    x2, x1, x4      // X2 = quotient 
            msub    x3, x2, x4, x1  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 

            // Handle digit 5 here: 

            udiv    x1, x2, x4      // X1 = quotient 
            msub    x3, x1, x4, x2  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 

            // Handle digit 6 here: 

            udiv    x2, x1, x4      // X2 = quotient 
            msub    x3, x2, x4, x1  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 

            // Handle digit 7 here: 

            udiv    x1, x2, x4      // X1 = quotient 
            msub    x3, x1, x4, x2  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 

            // Store away LO 8 digits: 

            str     x5, [x0, #8] 
            mov     x5, xzr 

 // Handle digit 8 here: 

          ❸ udiv    x2, x1, x4      // X2 = quotient 
            msub    x3, x2, x4, x1  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 

            // Handle digit 9 here: 

            udiv    x1, x2, x4      // X1 = quotient 
            msub    x3, x1, x4, x2  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 

            // Handle digit 10 here: 

            udiv    x2, x1, x4      // X2 = quotient 
            msub    x3, x2, x4, x1  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 

            // Handle digit 11 here: 

            udiv    x1, x2, x4      // X1 = quotient 
            msub    x3, x1, x4, x2  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 

            // Handle digit 12 here: 

            udiv    x2, x1, x4      // X2 = quotient 
            msub    x3, x2, x4, x1  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 

            // Handle digit 13 here: 

            udiv    x1, x2, x4      // X1 = quotient 
            msub    x3, x1, x4, x2  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 

            // Handle digit 14 here: 

            udiv    x2, x1, x4      // X2 = quotient 
            msub    x3, x2, x4, x1  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 

            // Handle digit 15 here: 

            udiv    x1, x2, x4      // X1 = quotient 
            msub    x3, x1, x4, x2  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 

 // Store away HO 8 digits: 

            str     x5, [x0] 
            strb    wzr, [x0, #maxDigits]!  // Zero-terminating byte 

            ldr     x5, [sp], #16 
            ldp     x3, x4, [sp], #16 
            ldp     x1, x2, [sp], #16 
            ret 
            endp    u53toStr 
```

u53ToStr 函数 ❶ 负责将一个 53 位无符号整数转换为一个恰好包含 16 位数字的字符串。理论上，代码可以使用来自列表 9-12 的 u64toSizeStr 函数将 53 位的值（零扩展为 64 位）转换为字符串。然而，浮动点尾数转换为字符串时总是会生成一个 16 字符的字符串（如果需要，会有前导零），因此十进制整数到字符串的转换比 u64toSizeStr 函数更高效，后者可能会生成可变长度的字符串。为了优先节省空间，如果你的代码中已经使用了 u64toSizeStr 函数，你可以去掉 u53ToStr，改为调用 u64toSizeStr（并指定 '0' 作为填充字符）。

u53ToStr 使用的转换算法非常直接且暴力：它将低 8 位数字转换为 8 个字符的序列并输出❷，然后将高 8 位数字转换为 8 个字符的序列并输出❸。在这两种情况下，它都使用除以 10 和除以 10 的余数算法将每个数字转换为字符（更多细节请参考 Listing 9-6 中的 u64ToStr 讨论）。

此函数由 FPDigits 使用，将尾数转换为十进制数字字符串：

```
// Listing9-13.S (cont.) 
//
// FPDigits 
//
//  Used to convert a floating-point value 
//  in D0 to a string of digits 
//
// Inputs: 
//  D0-     Double-precision value to convert 
//  X0-     Pointer to buffer to receive chars 
//
// Outputs: 
//  X0-     Still points at buffer 
//  X1-     Contains exponent of the number 
//  X2-     Contains sign (space or '-') 

            proc    FPDigits 
            str     lr,       [sp, #-16]! 
            str     d0,       [sp, #-16]! 
            stp     d1, d2,   [sp, #-16]! 
            stp     x22, x23, [sp, #-16]! 
 stp     x24, x25, [sp, #-16]! 
            stp     x26, x27, [sp, #-16]! 

            mov     x2, #' '        // Assume sign is +. 

#define fp1 d2                      // D2 holds 1.0\. 

            fmov    fp1, #1.0 

             // Special case for 0.0: 

          ❶ fcmp    d0, #0.0 
            bne     d0not0 

            // Check for -0.0: 

          ❷ fmov    x1, d0 
            ands    x1, x1, #0x8000000000000000 
            beq     posZero 
            mov     x2, #'-' 

posZero: 
            mov     x1, #0x3030 
            movk    x1, #0x3030, lsl #16 
            movk    x1, #0x3030, lsl #32 
            movk    x1, #0x3030, lsl #48 
            str     x1, [x0] 
            str     x1, [x0, #8] 
            mov     x1, #0          // Exponent = 0 

            // For debugging purposes, zero-terminate this 
            // string (the actual code just grabs 16 bytes, 
            // so this isn't strictly necessary): 

            strb    w0, [x0, #16] 
            b.al    fpdDone 

// If the number is nonzero, deal with it here. Note 
// that the flags were set by comparing D0 to 0.0 earlier. 

❸ d0not0:     bge     fpIsPositive    // See if positive or negative. 

            // If negative, negate and change the sign 
            // character to '-'. 

            fabs    d0, d0 
            mov     x2, #'-' 

// Get the number from 1.0 to <10.0 so you can figure out 
// what the exponent is. Begin by checking to see if you have 
// a positive or negative exponent. 

fpIsPositive: 
            mov     x1, xzr         // Initialize exponent. 
 ❹ fcmp    d0, fp1 
            bge     posExp 

            // The value is in the range 0.0 to 1.0, 
            // exclusive, at this point. That means this 
            // number has a negative exponent. Multiply 
            // the number by an appropriate power of 10 
            // until you get it in the range 1 through 10\. 

            lea     x27, potNeg 
            lea     x26, potPos 
            lea     x25, expTbl 
            mov     x24, #expCnt 

// Search through the potNeg table until you find a power 
// of 10 that is less than the value in D0: 

cmpNegExp: 
          ❺ subs    x24, x24, #1 
            blt     test1       // Branch if X24 < 1\. 

            ldr     d1, [x27, x24, lsl #3]  // D1 = potNeg[X24 * 8] 
            fcmp    d1, d0      // Repeat while 
            ble     cmpNegExp   // table <= value. 

            // Eliminate the current exponent indexed by 
            // X24 by multiplying by the corresponding 
            // entry in potPos: 

            ldr     x22, [x25, x24, lsl #3] // X22 = expTbl[X24 * 8] 
            sub     x1, x1, x22 
            ldr     d1, [x26, x24, lsl #3]  // D1 = potPos[X24 * 8] 
            fmul    d0, d0, d1 
            b.al    cmpNegExp 

// If you get to this point, you've indexed through 
// all the elements in the potNeg and it's time to stop. 
//
// If the remainder is *exactly* 1.0, you can branch 
// on to InRange1_10; otherwise, you still have to multiply 
// by 10.0 because you've overshot the mark a bit. 

test1:      fcmp    d0, fp1 
            beq     inRange1_10 

            fmov    d1, #10.0 
            fmul    d0, d0, d1 
            sub     x1, x1, #1      // Decrement exponent. 
            b.al    inRange1_10 

// At this point, you have a number that is 1 or greater. 
// Once again, your task is to get the value from 1.0 to <10.0\. 

posExp: 
            lea     x26, potPos 
            lea     x25, expTbl 
            mov     x24, #expCnt 

❻ cmpPosExp:  subs    x24, x24, #1 
            blt     inRange1_10     // If X24 < 1 

            ldr     d1, [x26, x24, lsl #3]  // D1 = potPos[X24 * 8] 
            fcmp    d1, d0 
            bgt     cmpPosExp 

            ldr     x22, [x25, x24, lsl #3] // X22 = expTbl[X24 * 8] 
            add     x1, x1, x22 
            fdiv    d0, d0, d1 
            b.al    cmpPosExp 

// Okay, at this point the number is in the range 1 <= x < 10\. 
// Let's multiply it by 1e+15 to put the most significant digit 
// into the 16th print position, then convert the result to 
// a string and store away in memory. 

❼ inRange1_10: 
            ldr     d1, tenTo15 
            fmul    d0, d0, d1 
            fcvtau  x22, d0     // Convert to unsigned integer. 

            // Convert the integer mantissa to a 
            // string of digits: 

            stp     x0, x1, [sp, #-16]! 
            mov     x1, x22 
            bl      u53toStr 
            ldp     x0, x1, [sp], #16 

fpdDone: 
            ldp     x26, x27,   [sp], #16 
            ldp     x24, x25,   [sp], #16 
            ldp     x22, x23,   [sp], #16 
            ldp     d1, d2,     [sp], #16 
            ldr     d0,         [sp], #16 
            ldr     lr,         [sp], #16 
            ret 
            endp    FPDigits 
```

FPDigits 将任意的双精度尾数转换为十进制数字字符串。它假定要转换的浮点值存储在 D0 寄存器中，且 X0 包含指向将存储字符串转换结果的缓冲区的指针。该函数还将二进制（2 的幂次）指数转换为十进制整数，并将指数值返回至 X1 寄存器，将值的符号（空格字符表示非负值，或'-'）返回至 X2 寄存器。

FPDigits 首先检查特殊情况 0.0❶。如果 D0 包含 0，该函数将字符串缓冲区初始化为 0000000000000000（16 个 0 字符），并返回，X0 包含 0，X2 包含空格字符。代码会检查特殊情况-0.0，如果结果为-0.0，则返回 X2 包含负号❷。接下来，FPDigits 检查浮点值的符号，并根据需要将 X2 设置为'-'❸。代码还将十进制指数累加器（保存在 X0 中）初始化为 0。

在设置符号后，FPDigits 函数检查浮点值的指数，看看它是正数还是负数❹。代码分别处理正指数和负指数的值。如果指数为负，cmpNegExp 循环会通过 potNeg 表查找大于 D0 中值的项❺。当循环找到这样的值时，它将 D0 乘以 potNeg 中的该项，然后从 X1 中存储的十进制指数值中减去 expTbl 中相应的项。cmpNegExp 循环会重复这个过程，直到 D0 中的值大于 1.0。每当结果不大于 1.0 时，代码会将 D0 中的值乘以 10.0，因为代码需要调整之前发生的 0.1 的乘法。另一方面，如果指数为正❻，cmpPosExp 循环则做同样的工作，但会除以 potPos 表中的项，并将 expTbl 中相应的项加到 X1 中存储的十进制指数值。

一旦 cmpPosExp 或 cmpNegExp 循环将值调整到 1.0 到接近 10.0 的范围，它会将该值乘以 10¹⁵，并将其转换为整数（存储在 X22 中）❼。然后，FPDigits 调用 u53toStr 函数将此整数转换为一个精确的 16 位数字字符串。该函数将符号字符（非负值为空格，负值为'-'）返回至 X2，十进制指数返回至 X1。

请注意，FPDigits 仅将尾数转换为数字字符串。这是 r64ToStr 和 e64ToStr 函数的基础代码，用于将浮点值转换为可识别的字符串。在介绍这些函数之前，有一个实用函数需要解释：chkNaNINF。

某些浮点操作会产生无效结果。IEEE 754 浮点标准定义了三种特殊值来表示这些无效结果：NaN（非数字）、+INF（正无穷大）和-INF（负无穷大）。由于 ARM 浮点硬件可能会产生这些结果，因此浮点到字符串的转换必须处理这三种特殊值。NaN、+INF 和-INF 的指数值都包含 0x7FF（没有其他有效值使用此指数）。如果指数是 0x7FF 且尾数位全为 0，则值为+INF 或-INF（由符号位决定）。如果尾数非零，则值为 NaN（符号位可以忽略）。chkNaNINF 函数会检查这些值，并在数字无效时输出字符串 NaN、INF 或-INF：

```
// Listing9-13.S (cont.) 
//
// chkNaNINF 
//
// Utility function used by r64ToStr and e64ToStr to check 
// for NaN and INF 
//
// Inputs: 
//  D0-     Number to check against NaN and INF 
//  X19-    Field width for output 
//  X21-    Fill character 
//  X22-    (outBuf) Pointer to output buffer 
//  X25-    Return address to use if number is invalid 
//
// Outputs: 
//  Buffer- Will be set to the string NaN, INF, 
//          or -INF if the number is not valid 
//
//  Note: Modifies value in X0 

            proc    chkNaNINF 

            // Handle NaN and INF special cases: 

          ❶ fmov    x0, d0 
            lsr     x0, x0, #52 
            and     x0, x0, #0x7ff 
            cmp     x0, #0x7ff 
            blo     notINFNaN 

            // At this point, it's NaN or INF. INF has a 
            // mantissa containing 0, NaN has a nonzero 
            // mantissa: 

          ❷ fmov    x0, d0 
            ands    x0, x0, #0x000fffffffffffff 
            beq     isINF 

            // Is NaN here: 

          ❸ ldr     w0, ='N' + ('a' << 8) + ('N' << 16) 
            str     w0, [x22] 
            mov     x0, #3 
            b.al    fillSpecial 

            // INF can be positive or negative. Must output a 
            // '-' character if it is -INF: 

❹ isINF:      fmov    x0, d0 
            ands    x0, x0, #0x8000000000000000 // See if -INF. 
            bne     minusINF 

            ldr     w0, ='I' + ('N' << 8) + ('F' << 16) 
            str     w0, [x22] 
            mov     x0, #3 
            b.al    fillSpecial 

❺ minusINF:   ldr     w0, ='-' + ('I' << 8) + ('N' << 16) + ('F' << 24) 
            str     w0, [x22] 
 strb    wzr, [x22, #4] 
            mov     x0, #4 

// For NaN and INF, fill the remainder of the string, as appropriate: 

❻ fillSpecial: 
            b.al    whlLTwidth 

fsLoop:     strb    w21, [x22, x0] 
            add     x0, x0, #1 
 whlLTwidth: 
            cmp     x0, x19 
            blo     fsLoop 
          ❼ mov     lr, x25         // Return to alternate address. 

notINFNaN:  ret 
            endp    chkNaNINF 
```

代码将 D0 中的浮点值移动到 X0，然后检查指数位是否包含 0x7FF ❶。如果指数位不包含此值，过程将返回给调用者（使用 LR 中的返回地址）。

如果指数位是 0x7FF，代码会检查尾数以判断其是 0 还是非零 ❷。如果是非零，代码会将字符字符串 NaN 输出到由 X22 指向的缓冲区 ❸。如果尾数非零，代码会检查符号位是否被设置 ❹。如果没有，代码会将 INF 输出到输出缓冲区。如果符号位被设置，代码会将-INF 输出到输出缓冲区 ❺。

在所有三种情况下（NaN、INF 或-INF），代码会转移到 fillSpecial ❻，在那里它添加足够的填充字符（填充字符在 W21 中，字段宽度在 X19 中）。此时，代码不会返回给调用者，而是将控制权转移到 X25 中保存的地址 ❼。调用者（r64ToStr 或 e64ToStr）在调用 chkNaNINF 之前将无效值返回地址加载到 X25 中。我本可以设置一个标志，比如进位标志，并在返回时测试它。然而，我想展示另一种实现方法，这种方法稍显优雅（尽管可以说不那么易读）。

在 chkNaNINF 处理完毕后，是时候看看用户调用的 r64ToStr 函数，它将浮点值转换为字符串：

```
// Listing9-13.S (cont.) 
//
// r64ToStr 
//
// Converts a REAL64 floating-point number to the 
// corresponding string of digits. Note that this 
// function always emits the string using decimal 
// notation. For scientific notation, use the e10ToBuf 
// routine. 
//
// On entry: 
//
//  D0-         (r64) Real64 value to convert 
//
//  X0-         (outBuf) r64ToStr stores the resulting 
//              characters in this string. 
//
//  X1-         (fWidth) Field width for the number (note 
//              that this is an *exact* field width, not a 
//              minimum field width) 
//
//  X2-         (decDigits) # of digits to display after the 
//              decimal pt 
//
//  X3-         (fill) Padding character if the number of 
//              digits is smaller than the specified field 
//              width 
//
//  X4-         (maxLength) Maximum string length 
//
// On exit: 
//
// Buffer contains the newly formatted string. If the 
// formatted value does not fit in the width specified, 
// r64ToStr will store "#" characters into this string. 
//
// Carry-    Clear if success, set if an exception occurs. 
//           If width is larger than the maximum length of 
//           the string specified by buffer, this routine 
//           will return with the carry set. 
//
//***********************************************************

            proc    r64ToStr 

            // Local variables: 

            locals  rts 
            qword   rts.x0x1 
            qword   rts.x2x3 
            qword   rts.x4x5 
            qword   rts.x19x20 
            qword   rts.x21x22 
            qword   rts.x23x24 

            dword   rts.x25 
            byte    rts.digits, 80 
            byte    rts.stk, 64 
            endl    rts 

            enter   rts.size 

            // Use meaningful names for the nonvolatile 
            // registers that hold local/parameter values: 

            #define fpVal d0 
            #define fWidth x19      // chkNaNINF expects this here. 
 #define decDigits x20 
            #define fill w21        // chkNaNINF expects this here. 
            #define outBuf x22      // chkNaNINF expects this here. 
            #define maxLength x23 
            #define exponent x24 
            #define sign w25 
            #define failAdrs x25    // chkNaNINF expects this here. 

            // Preserve registers: 

            stp     x0,   x1, [fp, #rts.x0x1] 
            stp     x2,   x3, [fp, #rts.x2x3] 
            stp     x4,   x5, [fp, #rts.x4x5] 
            stp     x19, x20, [fp, #rts.x19x20] 
            stp     x21, x22, [fp, #rts.x21x22] 
            stp     x23, x24, [fp, #rts.x23x24] 
            str     x25,      [fp, #rts.x25] 

            // Move parameter values to nonvolatile 
            // storage: 

            mov     outBuf, x0 
            mov     fWidth, x1 
            mov     decDigits, x2 
            mov     fill, w3 
            mov     maxLength, x4 

            // First, make sure the number will fit into 
            // the specified string. 

            cmp     fWidth, maxLength 
            bhs     strOverflow 

            // If the width is 0, return an error: 

            cmp     fWidth, #0 
            beq     valOutOfRange 

            // Handle NaN and INF special cases. 
            // Note: if the value is invalid, control 
            // transfers to clcAndRet rather than simply 
            // returning. 

          ❶ lea     failAdrs, clcAndRet 
            bl      chkNaNINF 

            // Okay, do the conversion. Begin by 
            // processing the mantissa digits: 

            add     x0, fp, #rts.digits // lea x0, rts.digits 
          ❷ bl      FPDigits            // Convert r64 to string. 
            mov     exponent, x1        // Save away exponent result. 
            mov     sign, w2            // Save mantissa sign char. 

// Round the string of digits to the number of significant 
// digits you want to display for this number. Note that 
// a maximum of 16 digits are produced for a 53-bit value. 

          ❸ cmp     exponent, #maxDigits 
            ble     dontForceWidthZero 
            mov     x0, xzr         // If the exponent is negative or 
                                    // too large, set width to 0\. 
dontForceWidthZero: 
            add     x2, x0, decDigits // Compute rounding position. 
            cmp     x2, #maxDigits 
            bhs     dontRound       // Don't bother if a big #. 

            // To round the value to the number of 
            // significant digits, go to the digit just 
            // beyond the last one you are considering (X2 
            // currently contains the number of decimal 
            // positions) and add 5 to that digit. 
            // Propagate any overflow into the remaining 
            // digit positions. 

            add     x2, x2, #1          // Index + 1 of last sig digit 
            ldrb    w0, [x1, x2]        // Get that digit. 

            add     w0, w0, #5          // Round (for example, +0.5) 
            cmp     w0, #'9' 
            bls     dontRound 

            mov     x0, #('0' + 10)     // Force to 0\. 
whileDigitGT9: 
            sub     w0, w0, #10         // Sub out overflow, 
            strb    w0, [x1, x2]        // carry, into prev 
            subs    x2, x2, #1          // digit (until first 
            bmi     hitFirstDigit       // digit in the #). 

            ldrb    w0, [x1, x2]        // Increment previous 
            add     w0, w0, #1          // digit. 
            strb    w0, [x1, x2] 

            cmp     w0, #'9'            // Overflow if > '9' 
            bhi     whileDigitGT9 
            b.al    dontRound 

hitFirstDigit: 

            // If you get to this point, you've hit the 
            // first digit in the number, so you have to 
            // shift all the characters down one position 
            // in the string of bytes and put a "1" in the 
            // first character position. 

          ❹ mov     x2, #maxDigits      // Max digits in value 
repeatUntilX2eq0: 

 ldrb    w0, [x1, x2] 
            add     x2, x2, #1 
            strb    w0, [x1, x2] 
            subs    x2, x2, #2 
            bne     repeatUntilX2eq0 

            mov     w0, #'1' 
            strb    w0, [x1, x2] 

            add     exponent, exponent, #1 // Increment exponent because 
                                           // you added a digit. 

dontRound: 

            // Handle positive and negative exponents separately. 

          ❺ mov     x5, xzr             // Index into output buf. 
            cmp     exponent, #0 
            bge     positiveExponent 

            // Negative exponents: 
            // Handle values from 0 to 1.0 here (negative 
            // exponents imply negative powers of 10). 
            // 
            // Compute the number's width. Since this 
            // value is from 0 to 1, the width 
            // calculation is easy: it's just the number of 
            // decimal positions they've specified plus 
            // 3 (since you need to allow room for a 
            // leading "-0."). X2 = number of digits to emit 
            // after "." 

            mov     x4, #4 
            add     x2, decDigits, #3 
            cmp     x2, x4 
            csel    x2, x2, x4, hs  // If X2 < X4, X2 = X4 

            cmp     x2, fWidth 
            bhi     widthTooBig 

            // This number will fit in the specified field 
            // width, so output any necessary leading pad 
            // characters. X3 = number of padding characters 
            // to output. 

          ❻ sub     x3, fWidth, x2 
            b.al    testWhileX3ltWidth 

whileX3ltWidth: 
            strb    fill, [outBuf, x5] 
            add     x5, x5, #1          // Index 
            add     x2, x2, #1          // Digits processed 
testWhileX3ltWidth: 
            cmp     x2, fWidth 
            blo     whileX3ltWidth 

 // Output " 0." or "-0.", depending on 
            // the sign of the number: 

            strb    sign, [outBuf, x5] 
            add     x5, x5, #1 
            mov     w0, #'0' 
            strb    w0, [outBuf, x5] 
            add     x5, x5, #1 
            mov     w0, #'.' 
            strb    w0, [outBuf, x5] 
            add     x5, x5, #1 
            add     x3, x3, #3 

            // Now output the digits after the decimal point: 

            mov     x2, xzr             // Count the digits here. 
            add     x1, fp, #rts.digits // lea x1, rts.digits 

// If the exponent is currently negative, or if 
// you've output more than 16 significant digits, 
// just output a 0 character. 

repeatUntilX3geWidth: 
            mov     x0, #'0' 
            adds    exponent, exponent, #1 
            bmi     noMoreOutput 

            cmp     x2, #maxDigits 
            bge     noMoreOutput 

            ldrb    w0, [x1] 
            add     x1, x1, #1 

noMoreOutput: 
            strb    w0, [outBuf, x5] 
            add     x5, x5, #1          // Index 
            add     x2, x2, #1          // Digits processed 
            add     x3, x3, #1          // Digit count 
            cmp     x3, fWidth 
            blo     repeatUntilX3geWidth 
            b.al    r64BufDone 

// If the number's actual width was bigger than the width 
// specified by the caller, emit a sequence of '#' characters 
// to denote the error. 

❼ widthTooBig: 

            // The number won't fit in the specified field 
            // width, so fill the string with the "#" 
            // character to indicate an error. 

            mov     x2, fWidth 
            mov     w0, #'#' 
fillPound:  strb    w0, [outBuf, x5] 
            add     x5, x5, #1          // Index 
            subs    x2, x2, #1 
            bne     fillPound 
            b.al    r64BufDone 

// Handle numbers with a positive exponent here. 
//
// Compute # of print positions consumed by output string. 
// This is given by: 
//
//                   Exponent     // # of digits to left of "." 
//           +       2            // Sign + 1's digit 
//           +       decDigits    // Add in digits right of "." 
//           +       1            // If there is a decimal point 

❽ positiveExponent: 

            mov     x3, exponent    // Digits to left of "." 
            add     x3, x3, #2      // sign posn 
            cmp     decDigits, #0   // See if any fractional 
            beq     decPtsIs0       // part. 

            add     x3, x3, decDigits // Digits to right of "." 
            add     x3, x3, #1        // Make room for the "." 

decPtsIs0: 

            // Make sure the result will fit in the 
            // specified field width. 

            cmp     x3, fWidth 
            bhi     widthTooBig 
            beq     noFillChars 

            // If the actual number of print positions 
            // is less than the specified field width, 
            // output leading pad characters here. 

            subs    x2, fWidth, x3 
            beq     noFillChars 

fillChars:  strb    fill, [outBuf, x5] 
            add     x5, x5, #1 
            subs    x2, x2, #1 
            bne     fillChars 

noFillChars: 

            // Output the sign character: 

            strb    sign, [outBuf, x5] 
            add     x5, x5, #1 

 // Okay, output the digits for the number here: 

            mov     x2, xzr             // Counts # of output chars 
            add     x1, fp, #rts.digits // lea x1, rts.digits 

            // Calculate the number of digits to output 
            // before and after the decimal point: 

            add     x3, decDigits, exponent 
            add     x3, x3, #1          // Always one digit before "." 

// If we've output fewer than 16 digits, go ahead 
// and output the next digit. Beyond 16 digits, 
// output 0s. 

repeatUntilX3eq0: 

            mov     w0, #'0' 
            cmp     x2, #maxDigits 
            bhs     putChar 

            ldrb    w0, [x1] 
            add     x1, x1, #1 

putChar:    strb    w0, [outBuf, x5] 
            add     x5, x5, #1 

            // If the exponent decrements down to 0, 
            // output a decimal point: 

            cmp     exponent, #0 
            bne     noDecimalPt 

            cmp     decDigits, #0 
            beq     noDecimalPt 

            mov     w0, #'.' 
            strb    w0, [outBuf, x5] 
            add     x5, x5, #1 

noDecimalPt: 
            sub     exponent, exponent, #1  // Count down to "." output. 
            add     x2, x2, #1    // # of digits thus far 
            subs    x3, x3, #1    // Total # of digits to output 
            bne     repeatUntilX3eq0 

// Zero-terminate string and leave: 

r64BufDone: strb    wzr, [outBuf, x5] 
❾ clcAndRet:  msr     nzcv, xzr    // clc = no error 
            b.al    popRet 

strOverflow: 
            mov     x0, #-3 // String overflow 
            b.al    ErrorExit 

valOutOfRange: 
            mov     x0, #-1 // Range error 
❿ ErrorExit:  mrs     x1, nzcv 
            orr     x1, x1, #(1 << 29) 
            msr     nzcv, x1        // stc = error 
            strb    wzr, [outBuf]   // Just to be safe 

            // Change X0 on return: 

            str     x0, [fp, #rts.x0x1] 

popRet: 
            ldp     x0, x1,   [fp, #rts.x0x1] 
            ldp     x2, x3,   [fp, #rts.x2x3] 
            ldp     x4, x5,   [fp, #rts.x4x5] 
            ldp     x19, x20, [fp, #rts.x19x20] 
            ldp     x21, x22, [fp, #rts.x21x22] 
            ldp     x23, x24, [fp, #rts.x23x24] 
            ldr     x25,      [fp, #rts.x25] 
            leave 
            endp    r64ToStr 
```

r64ToStr 函数将 D0 中的浮点值转换为标准十进制格式的字符串，支持输出字段宽度、小数点后的数字位数以及用于填充通常为空白的前导位置的字符。

在适当初始化后，r64ToStr 首先检查 NaN（非数值）、INF（无穷大）和 -INF（负无穷大） ❶；这些值需要特殊的非数值输出字符串，但仍然需要填充至 fWidth 字符数。r64ToStr 调用 FPDigits 将尾数转换为十进制数字字符的字符串（并以整数形式获得十的幂指数） ❷。接下来的步骤是根据小数点后出现的数字位数对数字进行四舍五入 ❸。该代码计算出由 FPDigits 生成的字符串中的索引位置，该位置在 decDigits 参数指定的数字位数之后。它获取该字符（该字符将是 '0' 到 '9'）并将其 ASCII 码加 5。如果结果大于字符 '9' 的 ASCII 码，代码必须将字符串中的前一个数字加 1。当然，如果该字符是 '9'，则会发生溢出，进位必须向前传递到前一个字符。如果进位一直传递到字符串的第一个字符，代码必须将所有字符向右移动一位，并在字符串的开头插入 '1' ❹。

接下来，代码输出与最终小数字符串相关的字符。算法分为两个部分 ❺，其中一部分处理正指数（包括 0），另一部分处理负指数。对于负指数，代码将输出任何填充字符、数字符号（仍保存在 X2 中）以及从尾数字符串转换得到的 decDigits 位数字 ❻。如果字段宽度和 decDigits 足够大，代码将简单地输出 '0' 字符，直到超过第 16 位有效数字。如果输出的数字位数超过调用者传递的字段宽度，widthTooBig 代码 ❼ 将输出 # 字符以指示格式错误（在浮点转换中的标准 HLL 格式错误处理方式）。

该代码处理大于或等于 1.0 的浮点数值转换（正指数） ❽。该代码输出必要的填充字符和数值的符号，然后计算输出字符串中小数点的位置，并按照之前描述的方式四舍五入字符串中的最后一个数字。然后，它输出由 FPDigits 返回的字符直到该位置。最后，输出小数点，后跟剩余的小数位。如果代码无法将数字适配到指定的字段宽度（以及小数位数），则会将控制转移到 widthTooBig 来生成错误字符串。

为了通知调用者可能发生的错误，该代码在返回时会清除进位标志 ❾（如果转换成功），或者如果发生错误，则在返回时设置进位标志 ❿。这允许调用者在调用 r64ToStr 后，通过简单的 bcs 或 bcc 指令来轻松测试成功/失败。

列表 9-13 中处理的最终输出格式为指数（科学）形式。这个转换由两个函数处理：expToBuf 和 e64ToStr。前者处理输出字符串中指数部分的格式化：

```
// Listing9-13.S (cont.) 
//
// expToBuf 
//
// Unsigned integer to buffer 
// Used to output up to three-digit exponents 
//
// Inputs: 
//
//  X0-   Unsigned integer to convert 
//  X1-   Exponent print width 1-3 
//  X2-   Points at buffer (must have at least 4 bytes) 
//
// Outputs: 
//
//  Buffer contains the string representing the converted 
//  exponent. 
//
//  Carry is clear on success, set on error. 

            proc    expToBuf 
            stp     x0, lr, [sp, #-16]! 
            stp     x1, x3, [sp, #-16]! 
            stp     x4, x5, [sp, #-16]! 

            mov     x5, xzr     // Initialize output string. 
            mov     x4, #10     // For division by 10 

// Verify exponent digit count is in the range 1-3: 

 ❶ cmp     x1, #1 
            blo     badExp 
            cmp     x1, #3 
            bhi     badExp 

// Verify the actual exponent will fit in the number of digits: 

          ❷ cmp     x1, #2 
            blo     oneDigit 
            beq     twoDigits 

            // Must be 3: 

            cmp     x0, #1000 
            bhs     badExp 

// Convert three-digit value to a string: 

          ❸ udiv    x1, x0, x4      // X1 = quotient 
            msub    x3, x1, x4, x0  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 

            udiv    x0, x1, x4      // X0 = quotient 
            msub    x3, x0, x4, x1  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 

            udiv    x1, x0, x4      // X1 = quotient 
            msub    x3, x1, x4, x0  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 

            b.al    outputExp 

// Single digit is easy: 

oneDigit: 
          ❹ cmp     x0, #10 
            bhs     badExp 

            orr     x5, x0, #'0' 
            b.al    outputExp 

// Convert value in the range 10-99 to a string 
// containing two characters: 

twoDigits: 
          ❺ cmp     x0, #100 
            bhs     badExp 

            udiv    x1, x0, x4      // X1 = quotient 
            msub    x3, x1, x4, x0  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 

 udiv    x0, x1, x4      // X0 = quotient 
            msub    x3, x0, x4, x1  // X3 = remainder 
            orr     x3, x3, #'0' 
            orr     x5, x3, x5, lsl #8 

// Store the string into the buffer (includes a 0 
// byte in the HO positions of W5): 

outputExp: 
          ❻ str     w5, [x2] 
            ldp     x4, x5, [sp], #16 
            ldp     x1, x3, [sp], #16 
            ldp     x0, lr, [sp], #16 
            msr     nzcv, xzr    // clc = no error 
            ret 
            leave 

badExp: 
            ldp     x4, x5, [sp], #16 
            ldp     x1, x3, [sp], #16 
            ldp     x0, lr, [sp], #16 
            mrs     x0, nzcv 
            orr     x0, x0, #(1 << 29) 
            msr     nzcv, x0        // stc = error 
            mov     x0, #-1         // Value out of range ... 
            ret 
            endp    expToBuf 
```

expToBuf 函数生成一个恰好由一、两或三个数字组成的字符串（具体根据调用者传递的 X0 和 X1 参数决定）。expToBuf 函数首先验证指数位数是否在范围内 ❶，并且实际的指数值是否能够适应指定的数字位数 ❷。如果指数输出为三位数（正常情况 ❸）、一位数 ❹ 或两位数 ❺，代码会跳转到三个不同的输出转换代码序列。代码将这些字符存储到 X2 指向的缓冲区中 ❻。

该函数通过进位标志返回错误状态，若操作成功，则进位标志清除；如果指数过大或转换后的数字无法适应 X1 指定的字符位置数，则进位标志被设置。除此之外，expToBuf 基本上是一个 switch 语句（使用 if...then...else 逻辑实现），它有三个情况：分别对应每种指数大小（一位、两位或三位）。

e64ToStr 函数处理从双精度到字符串的转换，采用指数格式：

```
// Listing9-13.S (cont.) 
//
// e64ToStr 
//
// Converts a REAL64 floating-point number to the 
// corresponding string of digits. Note that this 
// function always emits the string using scientific 
// notation; use the r64ToStr routine for decimal notation. 
//
// On entry: 
//
//  D0-     (e64) Double-precision value to convert 
//
//  X0-     (buffer) e64ToStr stores the resulting characters in 
//          this buffer. 
//
//  X1-     (width) Field width for the number (note that this 
//          is an *exact* field width, not a minimum 
//          field width) 
//
//  X2-     (fill) Padding character if the number is smaller 
//          than the specified field width 
//
//  X3-     (expDigs) Number of exponent digits (2 for real32 
//          and 3 for real64) 
//
//  X4-     (maxLength) Maximum buffer size 
//
// On exit: 
//
//  Buffer contains the newly formatted string. If the 
//  formatted value does not fit in the width specified, 
//  e64ToStr will store "#" characters into this string. 
//
//  Carry-  Clear if no error, set if error. 
//          If error, X0 is 
//              -3 if string overflow 
//              -2 if bad width 
//              -1 if value out of range 
//
//-----------------------------------------------------------
//
// Unlike the integer-to-string conversions, this routine 
// always right-justifies the number in the specified 
// string. Width must be a positive number; negative 
// values are illegal (actually, they are treated as 
// *really* big positive numbers that will always raise 
// a string overflow exception). 
//
//***********************************************************

            proc       e64ToStr 

#define     e2sWidth   x19      // chkNaNINF expects this here. 
#define     e2sExp     x20 
#define     e2sFill    x21      // chkNaNINF expects this here. 
#define     e2sBuffer  x22      // chkNaNINF expects this here. 
#define     e2sMaxLen  x23 
#define     e2sExpDigs x24 

#define     e2sSign    w25 
#define     eFailAdrs  x25      // chkNaNINF expects this here. 
#define     e2sMantSz  x26 

            locals  e2s 
            qword   e2s.x1x2 
            qword   e2s.x3x4 
            qword   e2s.x5x19 
            qword   e2s.x20x21 
            qword   e2s.x22x23 
            qword   e2s.x24x25 
            qword   e2s.x26x27 
            dword   e2s.x0 
            dword   e2s.d0 
            byte    e2s.digits, 64 
            byte    e2s.stack, 64 
            endl    e2s 

            // Build activation record and preserve registers: 

            enter   e2s.size 
            str     x0,       [fp, #e2s.x0] 
            stp     x1,  x2,  [fp, #e2s.x1x2] 
            stp     x3,  x4,  [fp, #e2s.x3x4] 
            stp     x5,  x19, [fp, #e2s.x5x19] 
            stp     x20, x21, [fp, #e2s.x20x21] 
            stp     x22, x23, [fp, #e2s.x22x23] 
            stp     x24, x25, [fp, #e2s.x24x25] 
            stp     x26, x27, [fp, #e2s.x26x27] 
            str     d0,       [fp, #e2s.d0] 

            // Move important data to nonvolatile registers: 

            mov     e2sBuffer, x0 
            mov     e2sWidth, x1 
            mov     e2sFill, x2 
            mov     e2sExpDigs, x3 
            mov     e2sMaxLen, x4 

            // See if the width is greater than the buffer size: 

            cmp     e2sWidth, e2sMaxLen 
            bhs     strOvfl 

            strb    wzr, [e2sBuffer, e2sWidth]  // Zero-terminate str. 

// First, make sure the width isn't 0: 

          ❶ cmp     e2sWidth, #0 
            beq     valOutOfRng 

// Just to be on the safe side, don't allow widths greater 
// than 1024: 

 cmp     e2sWidth, #1024 
            bhi     badWidth 

// Check for NaN and INF: 

          ❷ lea     failAdrs, exit_eToBuf   // Note: X25, used before 
            bl      chkNaNINF               // e2sSign (also X25) 

// Okay, do the conversion: 

          ❸ add     x0, fp, #e2s.digits // lea x1, e2s.digits 
            bl      FPDigits        // Convert D0 to digit str. 
            mov     e2sExp, x1      // Save away exponent result. 
            mov     e2sSign, w2     // Save mantissa sign char. 

// Verify that there is sufficient room for the mantissa's sign, 
// the decimal point, two mantissa digits, the "E",
// and the exponent's sign. Also add in the number of digits 
// required by the exponent (2 for single, 3 for double). 
//
// -1.2e+00    :real4 
// -1.2e+000   :real8 

          ❹ add     x2, e2sExpDigs, #6    // Minimum number of posns 
            cmp     x2, e2sWidth 
            bls     goodWidth 

// Output a sequence of "#...#" chars (to the specified width) 
// if the width value is not large enough to hold the 
// conversion: 

            mov     x2, e2sWidth 
            mov     x0, #'#' 
            mov     x1, e2sBuffer 
fillPnd:    strb    w0, [x1] 
            add     x1, x1, #1 
            subs    x2, x2, #1 
            bne     fillPnd 
            b.al    exit_eToBuf 

// Okay, the width is sufficient to hold the number; do the 
// conversion and output the string here: 

goodWidth: 
            // Compute the # of mantissa digits to display, 
            // not counting mantissa sign, decimal point, 
            // "E", and exponent sign: 

          ❺ sub     e2sMantSz, e2sWidth, e2sExpDigs 
            sub     e2sMantSz, e2sMantSz, #4 

            // Round the number to the specified number of 
            // print positions. (Note: since there are a 
            // maximum of 16 significant digits, don't 
 // bother with the rounding if the field width 
            // is greater than 16 digits.) 

            cmp     e2sMantSz, #maxDigits 
            bhs     noNeedToRound 

            // To round the value to the number of 
            // significant digits, go to the digit just 
            // beyond the last one you are considering (e2sMantSz 
            // currently contains the number of decimal 
            // positions) and add 5 to that digit. 
            // Propagate any overflow into the remaining 
            // digit positions. 

            add     x1, e2sMantSz, #1 
            add     x2, fp, #e2s.digits // lea x2, e2s.digits 
            ldrb    w0, [x2, x1]        // Get least sig digit + 1\. 
            add     w0, w0, #5          // Round (for example, +0.5). 
            cmp     w0, #'9' 
            bhi     whileDigGT9 
            b.al    noNeedToRound 

// Sneak this code in here, after a branch, so the 
// loop below doesn't get broken up. 

firstDigitInNumber: 

            // If you get to this point, you've hit the 
            // first digit in the number, so you have to 
            // shift all the characters down one position 
            // in the string of bytes and put a "1" in the 
            // first character position. 

            ldr     x0, [x2, #8] 
            str     x0, [x2, #9] 
            ldr     x0, [x2] 
            str     x0, [x2, #1] 

            mov     x0, #'1'        // Store '1' in 1st 
            strb    w0, [x2]        // digit position. 

            // Bump exponent by 1, as the shift did 
            // a divide by 10\. 

            add     e2sExp, e2sExp, #1 
            b.al    noNeedToRound 

// Subtract out overflow and add the carry into the previous 
// digit (unless you hit the first digit in the number): 

whileDigGT9: 
            sub     w0, w0, #10 
            strb    w0, [x2, x1] 
            subs    x1, x1, #1 
            bmi     firstDigitInNumber 

 // Add in carry to previous digit: 

            ldrb    w0, [x2, x1] 
            add     w0, w0, #1 
            strb    w0, [x2, x1] 
            cmp     w0, #'9'        // Overflow if char > '9' 
            bhi     whileDigGT9 

noNeedToRound: 
            add     x2, fp, #e2s.digits // lea x2, e2s.digits 

// Okay, emit the string at this point. This is pretty easy, 
// since all you really need to do is copy data from the 
// digits array and add an exponent (plus a few other simple chars). 

          ❻ mov     x1, #0      // Count output mantissa digits. 
            strb    e2sSign, [e2sBuffer], #1 

// Output the first character and a following decimal point 
// if there are more than two mantissa digits to output. 

            ldrb    w0, [x2] 
            strb    w0, [e2sBuffer], #1 
            add     x1, x1, #1 
            cmp     x1, e2sMantSz 
            beq     noDecPt 

            mov     w0, #'.' 
            strb    w0, [e2sBuffer], #1 

noDecPt: 

// Output any remaining mantissa digits here. 
// Note that if the caller requests the output of 
// more than 16 digits, this routine will output 0s 
// for the additional digits. 

            b.al    whileX2ltMantSizeTest 

whileX2ltMantSize: 

            mov     w0, #'0' 
            cmp     x1, #maxDigits 
            bhs     justPut0 

            ldrb    w0, [x2, x1] 

justPut0: 
            strb    w0, [e2sBuffer], #1 
            add     x1, x1, #1 

whileX2ltMantSizeTest: 

            cmp     x1, e2sMantSz 
            blo     whileX2ltMantSize 

// Output the exponent: 

          ❼ mov     w0, #'e' 
            strb    w0, [e2sBuffer], #1 
            mov     w0, #'+' 
            mov     w4, #'-' 
            neg     x5, e2sExp 

            cmp     e2sExp, #0 
            csel    w0, w0, w4, ge 
            csel    e2sExp, e2sExp, x5, ge 

            strb    w0, [e2sBuffer], #1 

            mov     x0, e2sExp 
            mov     x1, e2sExpDigs 
            mov     x2, e2sBuffer 
            bl      expToBuf 
            bcs     error 

exit_eToBuf: 
            msr     nzcv, xzr    // clc = no error 
            ldr     x0, [fp, #e2s.x0] 

returnE64: 
            ldp     x1,  x2,  [fp, #e2s.x1x2] 
            ldp     x3,  x4,  [fp, #e2s.x3x4] 
            ldp     x5,  x19, [fp, #e2s.x5x19] 
            ldp     x20, x21, [fp, #e2s.x20x21] 
            ldp     x22, x23, [fp, #e2s.x22x23] 
            ldp     x24, x25, [fp, #e2s.x24x25] 
            ldp     x26, x27, [fp, #e2s.x26x27] 
            ldr     d0,       [fp, #e2s.d0] 
            leave 

strOvfl:    mov     x0, #-3 
            b.al    error 

badWidth:   mov     x0, #-2 
            b.al    error 

valOutOfRng: 
            mov     x0, #-1 
error: 
            mrs     x1, nzcv 
            orr     x1, x1, #(1 << 29) 
            msr     nzcv, x1        // stc = error 
            b.al    returnE64 

            endp    e64ToStr 
```

将尾数转换为字符串与 r64ToStr 中的例程非常相似，尽管指数形式稍微简单一些，因为格式总是将小数点放置在第一个尾数数字后面。与 r64ToStr 一样，e64ToStr 通过检查输入参数是否有效来开始 ❶（如果发生错误，将返回并在 X0 中设置错误代码及进位标志）。在参数验证之后，代码检查是否为 NaN 或 INF ❷。然后，它调用 FPDigits 将尾数转换为数字字符串 ❸（保存在本地缓冲区中）。此调用还返回值的符号以及十进制整数指数。

在计算出十进制指数值后，e64ToStr 函数会检查转换后的值是否适合由 Width 输入参数指定的空间 ❹。如果转换后的数字过大，e64ToStr 会生成一串 # 字符以表示错误。

请注意，这种情况不被视为返回进位标志已设置的错误。如果调用者指定了不足的字段宽度，函数仍然能够成功生成字符串转换；不过该字符串可能会被填充为 # 字符。只有在 e64ToStr 无法生成输出字符串时，进位标志才会被设置为错误。

在验证字符串能够适应指定的字段宽度后，e64ToStr 函数将结果四舍五入到指定的小数位数❺。这个算法与 r64ToStr 使用的算法相同。接下来，代码输出尾数位数❻。同样，这与 r64ToStr 的工作方式类似，只不过小数点总是放在第一个数字后（不需要计算它的位置）。最后，代码输出 e 后跟指数的符号字符❼，然后调用 expToBuf 将指数转换为一位、两位或三位的字符序列（由调用者通过 X3 传递的 expDigs 参数指定）。

列表 9-13 中的其余代码提供了主程序用来显示数据的实用函数（r64Print 和 e64Print），以及演示如何使用本节中函数的 asmMain 过程：

```
// Listing9-13.S (cont.) 
//
            proc    r64Print 

            stp     x0, x1, [sp, #-16]! 
            stp     x2, x3, [sp, #-16]! 
            stp     x4, x5, [sp, #-16]! 
            stp     x6, x7, [sp, #-16]! 
            stp     x8, lr, [sp, #-16]! 
            sub     sp, sp, #64 

            lea     x0, fmtStr1 
            lea     x1, r64str_1 
            mstr    x1, [sp] 
            bl      printf 

            add     sp, sp, #64 
            ldp     x8, lr, [sp], #16 
            ldp     x6, x7, [sp], #16 
            ldp     x4, x5, [sp], #16 
            ldp     x2, x3, [sp], #16 
 ldp     x0, x1, [sp], #16 
            ret 
            endp    r64Print 

            proc    e64Print 
            stp     x0, x1, [sp, #-16]! 
            stp     x2, x3, [sp, #-16]! 
            stp     x4, x5, [sp, #-16]! 
            stp     x6, x7, [sp, #-16]! 
            stp     x8, lr, [sp, #-16]! 
            sub     sp, sp, #64 

            lea     x0, fmtStr3 
            lea     x1, r64str_1 
            mstr    x1, [sp] 
            bl      printf 

            add     sp, sp, #64 
            ldp     x8, lr, [sp], #16 
            ldp     x6, x7, [sp], #16 
            ldp     x4, x5, [sp], #16 
            ldp     x2, x3, [sp], #16 
            ldp     x0, x1, [sp], #16 
            ret 
            endp    e64Print 
```

请注意，这些函数保留了所有非易失性寄存器，因为 printf()可以修改它们。

asmMain 函数是一个典型的浮点数字符串转换函数示范程序。它使用不同的输入参数调用 r64ToStr 和 e64ToStr 函数，演示这些函数的用法：

```
// Listing9-13.S (cont.) 
//
❶ r64_1:      .double  1.234567890123456 
            .double  0.0000000000000001 
            .double  1234567890123456.0 
            .double  1234567890.123456 
            .double  99499999999999999.0 
            .dword   0x7ff0000000000000 
            .dword   0xfff0000000000000 
            .dword   0x7fffffffffffffff 
            .dword   0xffffffffffffffff 
            .double  0.0 
            .double  -0.0 
fCnt         =       (. - r64_1) 

rSizes:     .word    12, 12, 2, 7, 0, 0, 0, 0, 0, 2, 2 

e64_1:      .double  1.234567890123456e123 
            .double  1.234567890123456e-123 
e64_3:      .double  1.234567890123456e1 
 .double  1.234567890123456e-1 
            .double  1.234567890123456e10 
            .double  1.234567890123456e-10 
            .double  1.234567890123456e100 
            .double  1.234567890123456e-100 
            .dword   0x7ff0000000000000 
            .dword   0xfff0000000000000 
            .dword   0x7fffffffffffffff 
            .dword   0xffffffffffffffff 
            .double  0.0 
            .double  -0.0 
eCnt         =       (. - e64_1) 

eSizes:     .word    6, 9, 8, 12, 14, 16, 18, 20, 12, 12, 12, 12, 8, 8 
expSizes:   .word    3, 3, 2, 2, 2, 2, 3, 3, 2, 2, 2, 2, 2, 2 

// Here is the asmMain function: 

            proc    asmMain, public 

            locals  am 
            dword   am.x8x9 
            dword   am.x27 
            byte    am.stk, 64 
            endl    am 

            enter   am.size     // Activation record 
            stp     x8, x9, [fp, #am.x8x9] 
            str     x27,    [fp, #am.x27] 

// F output 

            mov     x2, #16         // decDigits 
fLoop: 
            ldr     d0, r64_1 
            lea     x0, r64str_1    // Buffer 
            mov     x1, #30         // fWidth 
            mov     x3, #'.'        // Fill 
            mov     x4, 32          // maxLength 
            bl      r64ToStr 
            bcs     fpError 
            bl      r64Print 
            subs    x2, x2, #1 
            bpl     fLoop 

            lea     x0, newlines 
            bl      printf 

            lea     x5, r64_1 
            lea     x6, rSizes 
            mov     x7, #fCnt/8 
f2Loop:     ldr     d0, [x5], #8 
            lea     x0, r64str_1    // Buffer 
            mov     x1, #30         // fWidth 
 ldr     w2, [x6], #4    // decDigits 
            mov     x3, #'.'        // Fill 
            mov     x4, #32         // maxLength 
            bl      r64ToStr 
            bcs     fpError 
            bl      r64Print 
            subs    x7, x7, #1 
            bne     f2Loop 

// E output 

            lea     x0, expStr 
            bl      printf 

            lea     x5, e64_1 
            lea     x6, eSizes 
            lea     x7, expSizes 
            mov     x8, #eCnt/8 
eLoop: 
            ldr     d0, [x5], #8 
            lea     x0, r64str_1    // Buffer 
            ldr     w1, [x6], #4    // fWidth 
            mov     x2, #'.'        // Fill 
            ldr     w3, [x7], #4    // expDigits 
            mov     x4, #32         // maxLength 
            bl      e64ToStr 
            bcs     fpError 
            bl      e64Print 
            subs    x8, x8, #1 
            bne     eLoop 
            b.al    allDone 

fpError: 
            mov     x1, x0 
            lea     x0, fmtStr2 
            mstr    x1, [sp] 
            bl      printf 

allDone: 
            ldp     x8, x9, [fp, #am.x8x9] 
            ldr     x27,    [fp, #am.x27] 
            leave 
            endp    asmMain 
```

列表 9-13 将浮点常量值放在代码段中，而不是只读数据段❶，这样在查看主程序时更容易修改它们。

以下是列表 9-13 的构建命令和示例输出：

```
% ./build Listing9-13 
% 1G 
Calling Listing9-13: 
r64ToStr: value='........... 1.2345678901234560' 
r64ToStr: value='............ 1.234567890123456' 
r64ToStr: value='............. 1.23456789012345' 
r64ToStr: value='.............. 1.2345678901234' 
r64ToStr: value='............... 1.234567890123' 
r64ToStr: value='................ 1.23456789012' 
r64ToStr: value='................. 1.2345678901' 
r64ToStr: value='.................. 1.234567890' 
r64ToStr: value='................... 1.23456789' 
r64ToStr: value='.................... 1.2345678' 
r64ToStr: value='..................... 1.234567' 
r64ToStr: value='...................... 1.23456' 
r64ToStr: value='....................... 1.2345' 
r64ToStr: value='........................ 1.234' 
r64ToStr: value='......................... 1.23' 
r64ToStr: value='.......................... 1.2' 
r64ToStr: value='............................ 1' 

r64ToStr: value='............... 1.234567890123' 
r64ToStr: value='............... 0.000000000000' 
r64ToStr: value='.......... 1234567890123456.00' 
r64ToStr: value='........... 1234567890.1234560' 
r64ToStr: value='............ 99500000000000000' 
r64ToStr: value='INF                           ' 
r64ToStr: value='-INF                          ' 
r64ToStr: value='NaN                           ' 
r64ToStr: value='NaN                           ' 
r64ToStr: value='......................... 0.00' 
r64ToStr: value='.........................-0.00' 

Testing e64ToStr: 

e64ToStr: value='######' 
e64ToStr: value=' 1.2e-123' 
e64ToStr: value=' 1.2e+01' 
e64ToStr: value=' 1.23456e-01' 
e64ToStr: value=' 1.2345678e+10' 
e64ToStr: value=' 1.234567890e-10' 
e64ToStr: value=' 1.2345678901e+100' 
e64ToStr: value=' 1.234567890123e-100' 
e64ToStr: value='INF         ' 
e64ToStr: value='-INF        ' 
e64ToStr: value='NaN         ' 
e64ToStr: value='NaN         ' 
e64ToStr: value=' 0.0e+00' 
e64ToStr: value='-0.0e+00' 
Listing9-13 terminated 
```

该输出演示了双精度浮点数输出。如果你想将一个单精度值转换为字符串，首先将单精度值转换为双精度值，然后使用这段代码将得到的双精度值转换为字符串。 ### 9.3 字符串与数值的转换

数值转换为字符串和字符串转换为数值的过程有两个基本区别。首先，数值到字符串的转换通常不会出错（前提是你分配了足够大的缓冲区，以防转换函数写入缓冲区末尾之外的数据）。而字符串到数值的转换则必须处理如非法字符和数值溢出等错误的实际可能性。

一个典型的数字输入操作包括从用户读取一个字符字符串，然后将这个字符字符串转换为内部数字表示。例如，在 C++ 中，像 `cin >> i32;` 这样的语句从用户读取一行文本，并将该行文本开头的数字序列转换为 32 位有符号整数（假设 i32 是一个 32 位整数对象）。`cin >> i32;` 语句会跳过字符串中可能出现在实际数字字符前面的某些字符，如前导空格。输入字符串还可能包含超出数字输入末尾的额外数据（例如，可能从同一输入行读取两个整数值），因此输入转换程序必须确定数字数据在输入流中的结束位置。

通常，C++ 通过查找一组 *分隔符* 字符来实现这一点。分隔符字符集可以是简单的任何非数字字符；或者该集合可能包括空白字符（空格、制表符等），以及可能的一些其他字符，如逗号（,）或其他标点符号。为了举例，本节代码假设任何前导空格或制表符字符（ASCII 代码 9）可能出现在第一个数字字符之前，并且转换会在遇到第一个非数字字符时停止。可能的错误情况如下：

+   字符串开头完全没有数字字符（跳过任何空格或制表符后）。

+   数字字符串的值可能太大，无法适应预定的数字大小（例如，64 位）。

调用者需要确定在函数调用返回后，数字字符串是否以无效字符结尾。

#### 9.3.1 十进制字符串转整数

将包含十进制数字的字符串转换为数字的基本算法如下：

1.  将累加器变量初始化为 0。

2.  跳过字符串中的任何前导空格或制表符。

3.  获取空格/制表符后的第一个字符。

4.  如果字符不是数字字符，返回错误。如果字符是数字字符，继续执行第 5 步。

5.  将数字字符转换为数字值（使用 AND 0xf）。

6.  将累加器设置为 = (累加器 × 10) + 当前数字值。

7.  如果发生溢出，返回并报告错误。如果没有发生溢出，则继续到第 8 步。

8.  从字符串中获取下一个字符。

9.  如果字符是数字字符，回到第 5 步；否则，继续执行第 10 步。

10.  返回成功，累加器中包含转换后的值。

对于有符号整数输入，使用相同的算法，并做以下修改：

+   如果第一个非空格/制表符字符是连字符（-），设置一个标志，表示数字是负数，并跳过 - 字符。如果第一个字符不是 -，则清除标志。

+   在成功转换结束时，如果标志被设置，在返回之前需要对整数结果进行取反（必须检查取反操作是否溢出）。

列表 9-14 实现了转换算法；我再次将这个列表分成几个部分，以便更好地注释它。第一部分包含通常的格式字符串，以及主程序用来测试 strtou 和 strtoi 函数的各种示例字符串。

```
// Listing9-14.S 
//
// String-to-numeric conversion 

            #include    "aoaa.inc"

false       =           0 
true        =           1 
tab         =           9 

            .section    .rodata, "" 
ttlStr:     .asciz      "Listing 9-14"
fmtStr1:    .ascii      "strtou: String='%s'\n"
            .asciz      "    value=%llu\n"

fmtStr2:    .ascii      "Overflow: String='%s'\n"
            .asciz      "    value=%llx\n"

fmtStr3:    .ascii      "strtoi: String='%s'\n"
            .asciz      "    value=%lli\n"

unexError:  .asciz      "Unexpected error in program\n"

value1:     .asciz      "  1"
value2:     .asciz      "12 " 
value3:     .asciz      " 123 " 
value4:     .asciz      "1234"
value5:     .asciz      "1234567890123456789"
value6:     .asciz      "18446744073709551615"
OFvalue:    .asciz      "18446744073709551616"
OFvalue2:   .asciz      "999999999999999999999"

ivalue1:    .asciz      "  -1"
ivalue2:    .asciz      "-12 " 
ivalue3:    .asciz      " -123 " 
ivalue4:    .asciz      "-1234"
ivalue5:    .asciz      "-1234567890123456789"
ivalue6:    .asciz      "-18446744073709551615"
OFivalue:   .asciz      "18446744073709551616"
OFivalue2:  .asciz      "-18446744073709551616"

            .code 
            .extern     printf 

////////////////////////////////////////////////////////////////////
//
// Return program title to C++ program: 

            proc        getTitle, public 
            lea         x0, ttlStr 
            ret 
            endp        getTitle 
```

这个程序没有任何静态、可写的数据；所有变量数据都保存在寄存器或局部变量中。

以下代码是 strtou 函数，它将包含十进制数字的字符串转换为无符号整数：

```
// Listing9-14.S (cont.) 
//
////////////////////////////////////////////////////////////////////
//
// strtou 
//
// Converts string data to a 64-bit unsigned integer 
//
// Input: 
//
//   X1-    Pointer to buffer containing string to convert 
//
// Outputs: 
//
//   X0-    Contains converted string (if success), error code 
//          if an error occurs 
//
//   X1-    Points at first char beyond end of numeric string 
//          If error, X1's value is restored to original value. 
//          Caller can check character at [X1] after a 
//          successful result to see if the character following 
//          the numeric digits is a legal numeric delimiter. 
//
//   C-     (carry flag) Set if error occurs, clear if 
//          conversion was successful. On error, X0 will 
//          contain 0 (illegal initial character) or 
//          0ffffffffffffffffh (overflow). 

            proc    strtou 

            str     x5, [sp, #-16]! 
            stp     x3, x4, [sp, #-16]! 
            stp     x1, x2, [sp, #-16]! 

            mov     x3, xzr 
            mov     x0, xzr 
            mov     x4, #10     // Used to mul by 10 

            // The following loop skips over any whitespace (spaces and 
            // tabs) that appear at the beginning of the string: 

          ❶ sub     x1, x1, #1      // Incremented below 
skipWS:     ldrb    w2, [x1, #1]!   // Fetch next (first) char. 
            cmp     w2, #' ' 
            beq     skipWS 
            cmp     w2, #tab 
            beq     skipWS 

            // If you don't have a numeric digit at this 
            // point, return an error. 

          ❷ cmp     w2, #'0'  // Note: '0' < '1' < ... < '9' 
            blo     badNumber 
            cmp     w2, #'9' 
            bhi     badNumber 

// Okay, the first digit is good. Convert the string 
// of digits to numeric form. 
//
// Have to check for unsigned integer overflow here. 
// Unfortunately, madd does not set the carry or 
// overflow flag, so you have to use umulh to see if 
// overflow occurs after a multiplication and do 
// an explicit add (rather than madd) to add the 
// digit into the accumulator (X0). 

❸ convert:    umulh   x5, x0, x4      // Acc * 10 
            cmp     x5, xzr 
            bne     overflow 
            and     x2, x2, #0xf    // Char -> numeric in X2 
            mul     x0, x0, x4      // Can't use madd! 
            adds    x0, x0, x2      // Add in digit. 
            bcs     overflow 

 ❹ ldrb    w2, [x1, #1]!   // Get next char. 
            cmp     w2, #'0'        // Check for digit. 
            blo     endOfNum 
            cmp     w2, #'9' 
            bls     convert 

// If you get to this point, you've successfully converted 
// the string to numeric form. Return without restoring 
// the value in X1 (X1 points at end of digits). 

❺ endOfNum:   ldp     x3, x4, [sp], #16   // Really X1, X2 
            mov     x2, x4 
            ldp     x3, x4, [sp], #16 
            ldr     x5, [sp], #16 

            // Because the conversion was successful, this 
            // procedure leaves X1 pointing at the first 
            // character beyond the converted digits. 
            // Therefore, we don't restore X1 from the stack. 

            msr     nzcv, xzr    // clr c = no error 
            ret 

// badNumber- Drop down here if the first character in 
//            the string was not a valid digit. 

❻ badNumber:  mov     x0, xzr 
errorRet:   mrs     x1, nzcv    // Return error in carry flag. 
            orr     x1, x1, #(1 << 29) 
            msr     nzcv, x1    // Set c = error. 

            ldp     x1, x2, [sp], #16 
            ldp     x3, x4, [sp], #16 
            ldr     x5, [sp], #16 
            ret 

// overflow- Drop down here if the accumulator overflowed 
//           while adding in the current character. 

overflow:   mov     x0, #-1  // 0xFFFFFFFFFFFFFFFF 
            b.al    errorRet 
            endp    strtou 
```

进入 strtou 时，X1 寄存器指向待转换字符串的第一个字符。该函数首先跳过字符串中的任何空白字符（空格和制表符），使 X1 指向第一个非空白/非制表符字符❶。

在任何空白字符之后，第一个字符必须是十进制数字，否则 strtou 必须返回转换错误。因此，在找到非空白字符后，代码会检查该字符是否在'0'到'9'的范围内❷。

在验证第一个字符是数字之后，代码进入主要的转换循环❸。通常，你只需将字符转换为整数（与 0xF 进行按位与操作），然后将 X0 寄存器中的累加器乘以 10，并加上字符的值。这可以通过两条指令完成：

```
and  x2, x2, #0xf 
madd x0, x0, x4, x2 // X4 contains 10\. 
```

唯一的问题是，使用这两条指令不能检测溢出（strtou 函数必须执行这一操作）。为了检测因乘以 10 而导致的溢出，代码必须使用 umulh 指令，并检查结果是否为 0（如果不是 0，说明发生了溢出）❸。如果 umulh 的结果为 0，代码可以放心地将累加器（X0）乘以 10 而不必担心溢出。当然，在将字符的值加到 X0 和 10 的积上时，仍然可能发生溢出，因此你仍然不能使用 madd 指令；相反，你必须先将累加器乘以 10，然后使用 adds 指令将字符值加进去，并立即检查进位标志。

转换循环会重复这一过程，直到发生溢出或遇到非数字字符。一旦遇到非数字字符❹，转换后的整数值会保存在 X0 寄存器中，函数返回时，进位标志被清除。注意，如果转换成功，strtou 函数不会恢复 X1 寄存器的值；相反，它会返回时让 X1 指向第一个非数字字符❺。调用者有责任检查这个字符，看看它是否合法。

如果发生溢出或遇到非法的起始字符，函数会返回时设置进位标志，并在 X0 寄存器中放入错误代码❻。

以下代码是 strtoi 过程，它是 strtou 过程的有符号整数版本：

```
// Listing9-14.S (cont.) 
//
// strtoi 
//
// Converts string data to a 64-bit signed integer 
//
// Input: 
//
//   X1-    Pointer to buffer containing string to convert 
//
// Outputs: 
//
//   X0-    Contains converted string (if success), error code 
//          if an error occurs 
//
//   X1-    Points at first char beyond end of numeric string. 
//          If error, X1's value is restored to original value. 
//          Caller can check character at [X1] after a 
//          successful result to see if the character following 
//          the numeric digits is a legal numeric delimiter. 
//
//   C-    (carry flag) Set if error occurs, clear if 
//         conversion was successful. On error, X0 will 
//         contain 0 (illegal initial character) or 
//         -1 (overflow). 

tooBig:     .dword  0x7fffffffffffffff 

            proc    strtoi 

            locals  si 
            qword   si.saveX1X2 
            endl    si 

            enter   si.size 

            // Preserve X1 in case you have to restore it; 
            // X2 is the sign flag: 

            stp     x1, x2, [fp, #si.saveX1X2] 

            // Assume you have a nonnegative number: 

            mov     x2, #false 

// The following loop skips over any whitespace (spaces and 
// tabs) that appear at the beginning of the string: 

          ❶ sub     x1, x1, #1  // Adjust for +1 below. 
skipWSi:    ldrb    w0, [x1, #1]! 
            cmp     w0, #' ' 
            beq     skipWSi 
            cmp     w0, #tab 
            beq     skipWSi 

            // If the first character you've encountered is 
            // '-', then skip it, but remember that this is 
            // a negative number: 

          ❷ cmp     w0, #'-' 
            bne     notNeg 
            mov     w2, #true 
            add     x1, x1, #1  // Skip '-' 

❸ notNeg:     bl      strtou       // Convert string to integer. 
            bcs     hadError 

            // strtou returned success. Check the negative 
            // flag and negate the input if the flag 
            // contains true: 

          ❹ cmp     w2, #true 
            bne     itsPosOr0 

            negs    x0, x0 
            bvs     overflowi 
            ldr     x2, [fp, #si.saveX1X2+8] 
 msr     nzcv, xzr   // clr c = no error 
            leave 

// Success, so don't restore X1: 

itsPosOr0: 
            ldr     x2, tooBig 
            cmp     x0, x2     // Number is too big. 
            bhi     overflowi 
            ldr     x2, [fp, #si.saveX1X2+8] 
            msr     nzcv, xzr  // clr c = no error 
            leave 

// If you have an error, you need to restore RDI from the stack: 

overflowi:  mov     x0, #-1     // Indicate overflow. 
hadError: 
            mrs     x2, nzcv    // Return error in carry flag. 
            orr     x2, x2, #(1 << 29) 
            msr     nzcv, x2    // Set c = error. 
            ldp     x1, x2, [fp, #si.saveX1X2] 
            leave 
            endp    strtoi 
```

strtoi 函数将包含有符号整数的字符串转换为 X0 中的相应值。代码首先消除空白❶，然后检查是否存在'-'字符❷。该函数在 X2 寄存器中维护一个“负数标志”（0 = 非负数，1 = 负数）。跳过可选的符号字符后，代码调用 strtou 函数将后续的字符串转换为无符号值❸。

从 strtou 返回后，strtoi 函数检查 X2 中的符号标志，如果应该是负数，则对数字进行取反❹。无论是负数还是非负数，代码还会检查是否发生溢出，并在发生溢出时返回错误。

与 strtou 一样，strtoi 函数在转换成功时不会恢复 X1。然而，如果发生溢出或 strtou 报告错误，它将恢复 X1。

当你调用 strtou 将字符串转换为整数时，strtoi 允许在表示负数的字符串中的减号和第一个数字之间有任意数量的空格。如果这对你来说是个问题，可以修改 strtou 来跳过空格，然后调用一个从属例程进行转换；接着，让 strtoi 调用该从属例程（如果合适的话，它会返回非法初始字符错误），而不是直接使用 strtou。

asmMain 函数演示了调用 strtou 和 strtoi 函数：

```
// Listing9-14.S (cont.) 
//
////////////////////////////////////////////////////////////////////
//
// Here is the asmMain function: 

            proc    asmMain, public 

            locals  am 
            byte    am.shadow, 64 
            endl    am 

            enter   am.size 

// Test unsigned conversions: 

            lea     x1, value1 
            bl      strtou 
            bcs     UnexpectedError 

            mov     x2, x0 
            lea     x0, fmtStr1 
            lea     x1, value1 
            mstr    x1, [sp] 
            mstr    x2, [sp, #8] 
            bl      printf 

            lea     x1, value2 
            bl      strtou 
            bcs     UnexpectedError 

            mov     x2, x0 
            lea     x0, fmtStr1 
            lea     x1, value2 
            mstr    x1, [sp] 
            mstr    x2, [sp, #8] 
            bl      printf 

            lea     x1, value3 
            bl      strtou 
            bcs     UnexpectedError 

            mov     x2, x0 
            lea     x0, fmtStr1 
            lea     x1, value3 
            mstr    x1, [sp] 
            mstr    x2, [sp, #8] 
            bl      printf 

            lea     x1, value4 
            bl      strtou 
            bcs     UnexpectedError 

            mov     x2, x0 
            lea     x0, fmtStr1 
            lea     x1, value4 
            mstr    x1, [sp] 
            mstr    x2, [sp, #8] 
            bl      printf 

 lea     x1, value5 
            bl      strtou 
            bcs     UnexpectedError 

            mov     x2, x0 
            lea     x0, fmtStr1 
            lea     x1, value5 
            mstr    x1, [sp] 
            mstr    x2, [sp, #8] 
            bl      printf 

            lea     x1, value6 
            bl      strtou 
            bcs     UnexpectedError 

            mov     x2, x0 
            lea     x0, fmtStr1 
            lea     x1, value6 
            mstr    x1, [sp] 
            mstr    x2, [sp, #8] 
            bl      printf 

            lea     x1, OFvalue 
            bl      strtou 
            bcc     UnexpectedError 
            cmp     x0, xzr        // Nonzero for overflow 
            beq     UnexpectedError 

            mov     x2, x0 
            lea     x0, fmtStr2 
            lea     x1, OFvalue 
            mstr    x1, [sp] 
            mstr    x2, [sp, #8] 
            bl      printf 

            lea     x1, OFvalue2 
            bl      strtou 
            bcc     UnexpectedError 
            cmp     x0, xzr        // Nonzero for overflow 
            beq     UnexpectedError 

            mov     x2, x0 
            lea     x0, fmtStr2 
            lea     x1, OFvalue2 
            mstr    x1, [sp] 
            mstr    x2, [sp, #8] 
            bl      printf 

// Test signed conversions: 

            lea     x1, ivalue1 
            bl      strtoi 
            bcs     UnexpectedError 

            mov     x2, x0 
 lea     x0, fmtStr3 
            lea     x1, ivalue1 
            mstr    x1, [sp] 
            mstr    x2, [sp, #8] 
            bl      printf 

            lea     x1, ivalue2 
            bl      strtoi 
            bcs     UnexpectedError 

            mov     x2, x0 
            lea     x0, fmtStr3 
            lea     x1, ivalue2 
            mstr    x1, [sp] 
            mstr    x2, [sp, #8] 
            bl      printf 

            lea     x1, ivalue3 
            bl      strtoi 
            bcs     UnexpectedError 

            mov     x2, x0 
            lea     x0, fmtStr3 
            lea     x1, ivalue3 
            mstr    x1, [sp] 
            mstr    x2, [sp, #8] 
            bl      printf 

            lea     x1, ivalue4 
            bl      strtoi 
            bcs     UnexpectedError 

            mov     x2, x0 
            lea     x0, fmtStr3 
            lea     x1, ivalue4 
            mstr    x1, [sp] 
            mstr    x2, [sp, #8] 
            bl      printf 

            lea     x1, ivalue5 
            bl      strtoi 
            bcs     UnexpectedError 

            mov     x2, x0 
            lea     x0, fmtStr3 
            lea     x1, ivalue5 
            mstr    x1, [sp] 
            mstr    x2, [sp, #8] 
            bl      printf 

            lea     x1, ivalue6 
            bl      strtoi 
            bcs     UnexpectedError 

 mov     x2, x0 
            lea     x0, fmtStr3 
            lea     x1, ivalue6 
            mstr    x1, [sp] 
            mstr    x2, [sp, #8] 
            bl      printf 

            lea     x1, OFivalue 
            bl      strtoi 
            bcc     UnexpectedError 
            cmp     x0, xzr        // Nonzero for overflow 
            beq     UnexpectedError 

            mov     x2, x0 
            lea     x0, fmtStr2 
            lea     x1, OFivalue 
            mstr    x1, [sp] 
            mstr    x2, [sp, #8] 
            bl      printf 

            lea     x1, OFivalue2 
            bl      strtoi 
            bcc     UnexpectedError 
            cmp     x0, xzr        // Nonzero for overflow 
            beq     UnexpectedError 

            mov     x2, x0 
            lea     x0, fmtStr2 
            lea     x1, OFivalue2 
            mstr    x1, [sp] 
            mstr    x2, [sp, #8] 
            bl      printf 

            b.al    allDone 

UnexpectedError: 
            lea     x0, unexError 
            bl      printf 

allDone:    leave   // Returns to caller 
            endp    asmMain 
```

Listing 9-14 中的 asmMain 函数是一个典型的测试程序；它将只读数据段中出现的各种字符串转换为相应的整数值并显示出来。它还测试了几个溢出条件，以验证例程是否正确处理溢出。

以下是 Listing 9-14 中程序的构建命令和示例输出：

```
% ./build Listing9-14 
% ./Listing9-14 
Calling Listing9-14: 
strtou: String='  1' 
    value=1 
strtou: String='12 ' 
    value=12 
strtou: String=' 123 ' 
    value=123 
strtou: String='1234' 
    value=1234 
strtou: String='1234567890123456789' 
    value=1234567890123456789 
strtou: String='18446744073709551615' 
    value=18446744073709551615 
Overflow: String='18446744073709551616' 
    value=ffffffffffffffff 
Overflow: String='999999999999999999999' 
    value=ffffffffffffffff 
strtoi: String='  -1' 
    value=-1 
strtoi: String='-12 ' 
    value=-12 
strtoi: String=' -123 ' 
    value=-123 
strtoi: String='-1234' 
    value=-1234 
strtoi: String='-1234567890123456789' 
    value=-1234567890123456789 
strtoi: String='-18446744073709551615' 
    value=1 
Overflow: String='18446744073709551616' 
    value=ffffffffffffffff 
Overflow: String='-18446744073709551616' 
    value=ffffffffffffffff 
Listing9-14 terminated 
```

对于扩展精度的字符串到数值转换，只需修改 strtou 函数，加入扩展精度累加器，然后进行扩展精度的乘法运算（而不是标准乘法）。

#### 9.3.2 十六进制字符串转换为数值形式

与数值输出类似，十六进制输入是最简单的数值输入例程。将十六进制字符串转换为数值形式的基本算法如下：

1.  将累加器值初始化为 0。

对于每个有效的十六进制数字字符，重复步骤 3 到步骤 6；如果字符不是有效的十六进制数字，则跳到步骤 7。

3.  将十六进制字符转换为 0 到 15 的值（0h 到 0Fh）。

4.  如果累加器值的高 4 位不为零，则抛出异常。

5.  将当前值乘以 16（即左移 4 位）。

6.  将转换后的十六进制数字值添加到累加器中。

7.  检查当前输入字符，确保它是有效的分隔符。如果不是，抛出异常。

Listing 9-15 实现了这个针对 64 位值的十六进制输入例程。

```
// Listing9-15.S 
//
// Hexadecimal-string-to-numeric conversion 

            #include    "aoaa.inc"

false       =           0 
true        =           1 
tab         =           9 

            .section    .rodata, "" 
ttlStr:     .asciz      "Listing 9-15"
fmtStr1:    .ascii      "strtoh: String='%s' " 
            .asciz      "value=%llx\n"

fmtStr2:    .asciz      "Error, str='%s', x0=%lld\n"

fmtStr3:    .ascii      "Error, expected overflow: x0=%llx, " 
            .asciz      "str='%s'\n"

fmtStr4:    .ascii      "Error, expected bad char: x0=%llx, " 
            .asciz      "str='%s'\n"

hexStr:     .asciz      "1234567890abcdef"
hexStrOVFL: .asciz      "1234567890abcdef0"
hexStrBAD:  .asciz      "x123"

            .code 
            .extern     printf 

/////////////////////////////////////////////////////////////
//
// Return program title to C++ program: 

            proc    getTitle, public 
            lea     x0, ttlStr 
            ret 
            endp    getTitle 

/////////////////////////////////////////////////////////////
//
// strtoh: 
//
// Converts string data to a 64-bit unsigned integer 
//
// Input: 
//
//   X1-    Pointer to buffer containing string to convert 
//
// Outputs: 
//
//   X0-    Contains converted string (if success), error code 
//          if an error occurs 
//
//   X1-    Points at first char beyond end of hexadecimal string. 
//          If error, X1's value is restored to original value. 
//          Caller can check character at [X1] after a 
//          successful result to see if the character following 
//          the hexadecimal digits is a legal delimiter. 
//
//   C-     (carry flag) Set if error occurs, clear if 
//          conversion was successful. On error, X0 will 
//          contain 0 (illegal initial character) or 
//          -1 = 0xffffffffffffffff (overflow). 

            proc    strtoh 

            stp     x3, x4, [sp, #-16]! 
            stp     x1, x2, [sp, #-16]! 

            // This code will use the value in X3 to test 
            // whether overflow will occur in X0 when 
            // shifting to the left 4 bits: 

            mov     x3, 0xF000000000000000 
            mov     x0, xzr // Zero out accumulator. 

            // 0x5f is used to convert lowercase to 
            // uppercase: 

            mov     x4, 0x5f 

// The following loop skips over any whitespace (spaces and 
// tabs) that appear at the beginning of the string: 

            sub     x1, x1, #1  // Because of inc below 
skipWS:     ldrb    w2, [x1, #1]! 
            cmp     w2, #' ' 
            beq     skipWS 
            cmp     w2, #tab 
            beq     skipWS 

            // If you don't have a hexadecimal digit at this 
            // point, return an error: 

 ❶ cmp     w2, #'0'    // Note: '0' < '1' < ... < '9' 
            blo     badNumber 
            cmp     w2, #'9' 
            bls     convert 
            and     x2, x2, x4  // Cheesy LC -> UC conversion 
            cmp     w2, #'A' 
            blo     badNumber 
            cmp     w2, #'F' 
            bhi     badNumber 
            sub     w2, w2, #7  // Maps 41h..46h -> 3ah..3fh 

            // Okay, the first digit is good. Convert the 
            // string of digits to numeric form: 

❷ convert:    ands    xzr, x3, x0  // See if adding in the current 
            bne     overflow     // digit will cause an overflow. 

            and     x2, x2, #0xf // Convert to numeric in X2\. 

            // Multiply 64-bit accumulator by 16 and add in 
            // new digit: 

          ❸ lsl     x0, x0, #4 
            add     x0, x0, x2  // Never overflows 

            // Move on to next character: 

            ldrb    w2, [x1, #1]! 
            cmp     w2, #'0' 
            blo     endOfNum 
            cmp     w2, #'9' 
            bls     convert 

            and     x2, x2, x4  // Cheesy LC -> UC conversion 
            cmp     x2, #'A' 
            blo     endOfNum 
            cmp     x2, #'F' 
            bhi     endOfNum 
            sub     x2, x2, #7  // Maps 41h..46h -> 3ah..3fh 
            b.al    convert 

// If you get to this point, you've successfully converted 
// the string to numeric form: 

endOfNum: 

            // Because the conversion was successful, this 
            // procedure leaves X1 pointing at the first 
            // character beyond the converted digits. 
            // Therefore, don't restore X1 from the stack. 

            ldp     x3, x2, [sp], #16   // X3 holds old X1 
            ldp     x3, x4, [sp], #16 
            msr     nzcv, xzr   // clr c = no error 
            ret 

// badNumber- Drop down here if the first character in 
//            the string was not a valid digit. 

badNumber:  mov     x0, xzr 
            b.al    errorExit 

overflow:   mov     x0, #-1     // Return -1 as error on overflow. 
errorExit: 
            mrs     x1, nzcv    // Return error in carry flag. 
            orr     x1, x1, #(1 << 29) 
            msr     nzcv, x1    // Set c = error. 

            ldp     x1, x2, [sp], #16 
            ldp     x3, x4, [sp], #16 
            ret 
            endp    strtoh 

/////////////////////////////////////////////////////////////
//
// Here is the asmMain function: 

            proc    asmMain, public 

            locals  am 
            byte    am.stack, 64 
            endl    am 

            enter   am.size 

            // Test hexadecimal conversion: 

            lea     x1, hexStr 
            bl      strtoh 
            bcs     error 

            mov     x2, x0 
            lea     x1, hexStr 
            lea     x0, fmtStr1 
            mstr    x1, [sp] 
            mstr    x2, [sp, #8] 
            bl      printf 

 // Test overflow conversion: 

            lea     x1, hexStrOVFL 
            bl      strtoh 
            bcc     unexpected 

            mov     x2, x0 
            lea     x0, fmtStr2 
            mstr    x1, [sp] 
            mstr    x2, [sp, #8] 
            bl      printf 

// Test bad character: 

            lea     x1, hexStrBAD 
            bl      strtoh 
            bcc     unexp2 

            mov     x2, x0 
            lea     x0, fmtStr2 
            mstr    x1, [sp] 
            mstr    x2, [sp, #8] 
            bl      printf 

            b.al    allDone 

unexpected: mov     x3, x0 
            lea     x0, fmtStr3 
            mstr    x1, [sp] 
            mstr    x2, [sp, #8] 
            mstr    x3, [sp, #16] 
            bl      printf 
            b.al    allDone 

unexp2:     mov     x3, x0 
            lea     x0, fmtStr4 
            mstr    x1, [sp] 
            mstr    x2, [sp, #8] 
            mstr    x3, [sp, #16] 
            bl      printf 
            b.al    allDone 

error:      mov     x2, x0 
            lea     x0, fmtStr2 
            mstr    x1, [sp] 
            mstr    x2, [sp, #8] 
            bl      printf 

allDone:    leave 
            endp    asmMain 
```

strtoh 函数类似于 strtou，不同之处在于它测试十六进制数字❶（而不仅仅是十进制数字），测试 HO 的 4 位以确定是否发生溢出❷（比十进制情况要简单得多），并且乘以十六进制基数（16）而不是 10❸。

以下是清单 9-15 中程序的构建命令和示例输出：

```
% ./build Listing9-15 
% ./Listing9-15 
Calling Listing9-15: 
strtoh: String='1234567890abcdef' value=1234567890abcdef 
Error, str='1234567890abcdef0', x0=-1 
Error, str='x123', x0 = 0 
Listing9-15 terminated 
```

对于处理大于 64 位的数字的十六进制字符串转换，你必须使用扩展精度的向左移 4 位。清单 9-16 演示了对 strtoh 函数进行必要的修改以进行 128 位转换。

```
// Listing9-16.S 
//
// 128-bit Hexadecimal-string-to-numeric conversion 

            #include    "aoaa.inc"

false       =           0 
true        =           1 
tab         =           9 

            .section    .rodata, "" 
 tlStr:     .asciz      "Listing 9-16"

fmtStr1:    .asciz      "strtoh128: value=%llx%llx, String='%s'\n"

hexStr:     .asciz      "1234567890abcdeffedcba0987654321"

            .code 
            .extern     printf 

/////////////////////////////////////////////////////////////
//
// Return program title to C++ program: 

            proc    getTitle, public 
            lea     x0, ttlStr 
            ret 
            endp    getTitle 

/////////////////////////////////////////////////////////////
//
// strtoh128 
//
// Converts string data to a 128-bit unsigned integer 
//
// Input: 
//
//   X2-    Pointer to buffer containing string to convert 
//
// Outputs: 
//
//   X1:X0- Contains converted string (if success), error code 
//          if an error occurs 
//
//   X2-    Points at first char beyond end of hexadecimal 
//          string. If error, X2's value is restored to 
//          original value. 
//          Caller can check character at [X2] after a 
//          successful result to see if the character following 
//          the hexadecimal digits is a legal delimiter. 
//
//   C-     (carry flag) Set if error occurs, clear if 
//          conversion was successful. On error, X0 will 
//          contain 0 (illegal initial character) or 
//          -1 = 0xffffffffffffffff (overflow). 

            proc    strtoh128 

            stp     x4, x5, [sp, #-16]! 
            stp     x2, x3, [sp, #-16]! 

            // This code will use the value in X4 to test 
            // whether overflow will occur in X1 when 
            // shifting to the left 4 bits: 

            mov     x4, 0xF000000000000000 
            mov     x0, xzr // Zero out LO accumulator. 
            mov     x1, xzr // Zero out HO accumulator. 

            // 0x5f is used to convert lowercase to 
            // uppercase: 

            mov     x5, 0x5f 

// The following loop skips over any whitespace (spaces and 
// tabs) that appear at the beginning of the string: 

            sub     x2, x2, #1 // Because of inc below 
skipWS:     ldrb    w3, [x2, #1]! 
            cmp     w3, #' ' 
            beq     skipWS 
            cmp     w3, #tab 
            beq     skipWS 

            // If you don't have a hexadecimal digit at this 
            // point, return an error: 

            cmp     w3, #'0'   // Note: '0' < '1' < ... < '9' 
            blo     badNumber 
            cmp     w3, #'9' 
            bls     convert 
            and     x3, x3, x5 // Cheesy LC -> UC conversion 
            cmp     w3, #'A' 
            blo     badNumber 
            cmp     w3, #'F' 
            bhi     badNumber 
            sub     w3, w3, #7 // Maps 41h..46h -> 3ah..3fh 

            // Okay, the first digit is good. Convert the 
            // string of digits to numeric form: 

convert:    ands    xzr, x4, x1  // See whether adding in the current 
            bne     overflow     // digit will cause an overflow. 

 and     x3, x3, #0xf // Convert to numeric in X3\. 

            // Multiply 128-bit accumulator by 16 and add in 
            // new digit (128-bit extended-precision shift 
            // by 4 bits): 

          ❶ lsl     x1, x1, #4  // 128 bits shifted left 4 bits 
            orr     x1, x1, x0, lsr #60 
            lsl     x0, x0, #4 
            add     x0, x0, x3  // Never overflows 

            // Move on to next character: 

            ldrb    w3, [x2, #1]! 
            cmp     w3, #'0' 
            blo     endOfNum 
            cmp     w3, #'9' 
            bls     convert 

            and     x3, x3, x5  // Cheesy LC -> UC conversion 
            cmp     x3, #'A' 
            blo     endOfNum 
            cmp     x3, #'F' 
            bhi     endOfNum 
            sub     x3, x3, #7 // Maps 41h..46h -> 3ah..3fh 
            b.al    convert 

// If you get to this point, you've successfully converted 
// the string to numeric form: 

endOfNum: 

            // Because the conversion was successful, this 
            // procedure leaves X2 pointing at the first 
            // character beyond the converted digits.
            // Therefore, we don't restore X2 from the stack. 

            ldp     x4, x3, [sp], #16   // X4 holds old X2\. 
            ldp     x4, x5, [sp], #16 
            msr     nzcv, xzr   // clr c = no error 

            ret 

// badNumber- Drop down here if the first character in 
//            the string was not a valid digit. 

badNumber:  mov     x0, xzr 
            b.al    errorExit 

overflow:   mov     x0, #-1     // Return -1 as error on overflow. 
errorExit: 
 mrs     x1, nzcv    // Return error in carry flag. 
            orr     x1, x1, #(1 << 29) 
            msr     nzcv, x1    // Set c = error. 
            ldp     x2, x3, [sp], #16 
            ldp     x4, x5, [sp], #16 
            ret 
            endp    strtoh128 

/////////////////////////////////////////////////////////////
//
// Here is the asmMain function: 

            proc    asmMain, public 

            locals  am 
            byte    am.stack, 64 
            endl    am 

            enter   am.size 

// Test hexadecimal conversion: 

            lea     x2, hexStr 
            bl      strtoh128 

            lea     x3, hexStr 
            mov     x2, x0 
            lea     x0, fmtStr1 
            mstr    x1, [sp] 
            mstr    x2, [sp, #8] 
            mstr    x3, [sp, #16] 
            bl      printf 

allDone:    leave 
            endp    asmMain 
```

这段代码的工作方式类似于清单 9-15 中的代码。主要区别在于清单 9-16 中的 128 位向左移 4 位❶。该代码将 X0 向右移 60 位，然后将其向左移 4 位后进行 OR 操作，将 X0 的 4 位移入 X1。

以下是清单 9-16 的构建命令和示例输出：

```
% ./build Listing9-16 
% ./Listing9-16 
Calling Listing9-16: 
strtoh128: value=1234567890abcdeffedcba0987654321, String='1234567890abcdeffedcba0987654321' 
Listing9-16 terminated 
```

十六进制字符串到数字的函数按预期工作。

#### 9.3.3 字符串转浮点数

将表示浮点数的字符字符串转换为 64 位双精度格式比本章前面出现的双精度转字符串转换稍微简单一些。因为十进制转换（没有指数）是更通用的科学计数法转换的一个子集，如果你能处理科学计数法，就可以轻松处理十进制转换。除此之外，基本算法是将尾数字符转换为整数形式以进行浮点数转换，然后读取（可选的）指数并相应调整双精度指数。转换的算法如下：

1.  首先去除任何前导空格或制表符字符（以及其他分隔符）。

2.  检查是否有前导加号（+）或减号（-）字符。如果有，跳过它。如果数字是负数，则将符号标志设置为 true（非负数则设置为 false）。

3.  将指数值初始化为 –16。该算法将根据字符串中的尾数数字创建一个整数值。由于双精度浮点数支持最多 16 位有效数字，因此将指数初始化为 –16 是考虑到了这一点。

4.  初始化一个有效数字计数器变量，记录到目前为止已处理的有效数字数量，初始值为 16。

5.  如果数字以任何前导零开头，则跳过它们（不要更改小数点左侧前导零的指数或有效数字计数）。

6.  如果扫描在处理完任何前导零后遇到小数点，则转到步骤 11；否则，继续执行步骤 7。

对于小数点左侧的每个非零数字，如果有效数字计数器不为 0，将整数累加器乘以 10 并加上该数字的数值。这是标准的整数转换。（如果有效数字计数器为 0，算法已经处理了 16 个有效数字，将忽略任何额外的数字，因为双精度格式无法表示超过 16 个有效数字的数值。）

对于小数点左侧的每个数字，将指数值（最初初始化为-16）加 1。

如果有效数字计数器不为 0，则递减有效数字计数器（这也会提供数字字符串数组的索引）。

如果遇到的第一个非数字字符不是小数点，则跳至步骤 14。

跳过小数点字符。

对于遇到的小数点右侧的每个数字，只要有效数字计数器不为 0，就继续将数字加到整数累加器中。如果有效数字计数器大于 0，则递减它。同时递减指数值。

如果到此为止算法没有遇到至少一个小数位数字，则报告非法字符异常并返回。

如果当前字符不是 e 或 E，则转到步骤 20。否则，跳过 e 或 E 字符并继续执行步骤 15。（注意，某些字符串格式也允许 d 或 D 表示双精度值。你也可以选择允许此格式，并在算法遇到 e 或 E 与 d 或 D 时检查值的范围。）

如果下一个字符是+或-，跳过它。如果符号字符是-，则将标志设置为 true；否则设置为 false（注意，这个指数符号标志不同于算法早期设置的尾数符号标志）。

如果下一个字符不是小数数字，则报告错误。

将从当前小数数字字符开始的数字字符串转换为整数。

将转换后的整数加到初始化为-16 的指数值上。

如果指数值超出范围–324 到+308，则报告超出范围的异常。

将当前为整数的尾数转换为浮点值。

取指数的绝对值，保留指数符号。该值将是 9 位或更少。

如果指数为正，则对于指数中每个设置的位，按该位位置所指定的幂次将当前尾数值乘以 10。例如，如果位 4、2 和 1 被设置，则将尾数值分别乘以 10¹⁶、10⁴和 10²。

如果指数为负，则对于指数中每个设置的位，按该位位置所指定的幂次将当前尾数值除以 10。例如，如果位 4、3 和 2 被设置，则将尾数值分别除以 10¹⁶、10⁸和 10⁴（从较大的值开始，逐步减少）。

24.  如果尾数是负数（在算法开始时设置了第一个符号标志），则将浮点数取反。

列表 9-17 提供了该算法的实现，并逐节进行了解释。第一部分对于本书中的示例程序来说是典型的，包含了一些常量声明、静态数据和 getTitle 函数。

```
// Listing9-17.S 
//
// Real string to floating-point conversion 

            #include    "aoaa.inc"

false       =           0 
true        =           1 
tab         =           9 

 .section    .rodata, "" 
ttlStr:     .asciz      "Listing 9-17"
fmtStr1:    .asciz      "strToR64: str='%s', value=%e\n"
errFmtStr:  .asciz      "strToR64 error, code=%ld\n"

❶ fStr1a:     .asciz      " 1.234e56"
fStr1b:     .asciz      "\t-1.234e+56"
fStr1c:     .asciz      "1.234e-56"
fStr1d:     .asciz      "-1.234e-56"
fStr2a:     .asciz      "1.23"
fStr2b:     .asciz      "-1.23"
fStr2c:     .asciz      "001.23"
fStr2d:     .asciz      "-001.23"
fStr3a:     .asciz      "1"
fStr3b:     .asciz      "-1"
fStr4a:     .asciz      "0.1"
fStr4b:     .asciz      "-0.1"
fStr4c:     .asciz      "0000000.1"
fStr4d:     .asciz      "-0000000.1"
fStr4e:     .asciz      "0.1000000"
fStr4f:     .asciz      "-0.1000000"
fStr4g:     .asciz      "0.0000001"
fStr4h:     .asciz      "-0.0000001"
fStr4i:     .asciz      ".1"
fStr4j:     .asciz      "-.1"
fStr5a:     .asciz      "123456"
fStr5b:     .asciz      "12345678901234567890"
fStr5c:     .asciz      "0"
fStr5d:     .asciz      "1." 
fStr6a:     .asciz      "0.000000000000000000001"

          ❷ .align      3 
values:     .dword      fStr1a, fStr1b, fStr1c, fStr1d 
            .dword      fStr2a, fStr2b, fStr2c, fStr2d 
            .dword      fStr3a, fStr3b 
            .dword      fStr4a, fStr4b, fStr4c, fStr4d 
            .dword      fStr4e, fStr4f, fStr4g, fStr4h 
            .dword      fStr4i, fStr4j 
            .dword      fStr5a, fStr5b, fStr5c, fStr5d 
            .dword      fStr6a 
            .dword      0 

❸ PotTbl:     .double     1.0e+256 
            .double     1.0e+128 
            .double     1.0e+64 
            .double     1.0e+32 
            .double     1.0e+16 
            .double     1.0e+8 
            .double     1.0e+4 
            .double     1.0e+2 
            .double     1.0e+1 
            .double     1.0e+0 

            .data 
r8Val:      .double     0.0 

 .code 
            .extern     printf 

///////////////////////////////////////////////////////////
//
// Return program title to C++ program: 

            proc    getTitle, public 
            lea     x0, ttlStr 
            ret 
            endp    getTitle 
```

只读部分包含了该程序将转换为浮点值的各种测试字符串❶。这些测试字符串经过精心挑选，以测试 strToR64 函数中大多数（成功的）路径。为了减少主程序的大小，列表 9-17 在一个循环中处理这些字符串。指针数组❷指向每个测试字符串，NULL 指针（0）标记列表的末尾。主程序将循环遍历这些指针来测试输入字符串。

PotTbl（10 的幂表）数组❸包含了各种 10 的幂。strToR64 函数使用这个表来将十进制指数（整数格式）转换为适当的 10 的幂：

```
// Listing9-17.S (cont.) 
//
// strToR64 
//
// On entry: 
//
//  X0- Points at a string of characters that represent a 
//      floating-point value 
//
// On return: 
//
//  D0- Converted result 
//  X0- On return, X0 points at the first character this 
//      routine couldn't convert (if no error). 
//
//  C-  Carry flag is clear if no error, set if error. 
//      X7 is preserved if an error, X1 contains an 
//      error code if an error occurs (else X1 is 
//      preserved). 

            proc    strToR64 

            locals  sr 
            qword   sr.x1x2 
            qword   sr.x3x4 
            qword   sr.x5x6 
            qword   sr.x7x0 
            dword   sr.d1 
            byte    sr.stack, 64    // Not really needed, but ... 
 endl    sr 

            enter   sr.size 

// Defines to give registers more 
// meaningful names: 

❶ #define mant    x1      // Mantissa value 
#define sigDig  x2      // Mantissa significant digits 
#define expAcc  x2      // Exponent accumulator 
#define sign    w3      // Mantissa sign 
#define fpExp   x4      // Exponent 
#define expSign w5      // Exponent sign 
#define ch      w6      // Current character 
#define xch     x6      // Current character (64 bits) 
#define ten     x7      // The value 10 

            // Preserve the registers this 
            // code modifies: 

          ❷ stp     x1, x2, [fp, #sr.x1x2] 
            stp     x3, x4, [fp, #sr.x3x4] 
            stp     x5, x6, [fp, #sr.x5x6] 
            stp     x7, x0, [fp, #sr.x7x0] 
            str     d1,     [fp, #sr.d1  ] 

            // Useful initialization: 

            mov     fpExp, xzr      // X3 Decimal exponent value 
            mov     mant, xzr       // X0 Mantissa value   
            mov     sign, wzr       // W2 Assume nonnegative. 

            // Initialize sigDig with 16, the number of 
            // significant digits left to process. 

            mov     sigDig, #16     // X1 

            // Verify that X0 is not NULL. 

            cmp     x0, xzr 
            beq     refNULL 
```

strToR64 函数使用#define 语句❶为它在各种寄存器中维护的局部变量创建了有意义的、可读性更强的名称。

尽管这个函数仅使用了寄存器 X0 到 X7 和 D1（它们在 ARM ABI 中都是易失的），但该函数会保存它修改过的所有寄存器❷。在汇编语言中，保持修改过的寄存器是良好的编程风格。这段代码没有保存 X0（假设转换成功），因为它返回 X0，指向已成功转换的字符串的末尾，作为函数结果。请注意，这段代码在 D0 中返回主函数结果。

在函数初始化之后，strToR64 函数通过跳过字符串开头的所有空白字符（空格和制表符）开始：

```
// Listing9-17.S (cont.) 

            sub     x0, x0, #1      // Will inc'd in loop 
whileWSLoop: 
            ldrb    ch, [x0, #1]!   // W5 
            cmp     ch, #' ' 
            beq     whileWSLoop 
            cmp     ch, #tab 
            beq     whileWSLoop 
```

这段代码退出时，ch（W6）包含第一个非空白字符，X0 指向该字符在内存中的位置。

在任何空白字符后面，字符串可能会可选地包含一个+或-字符。如果存在这些字符，代码会跳过它们，并且如果是'-'字符，设置尾数符号标志（sign）为 1：

```
// Listing9-17.S (cont.) 

            // Check for + or - 

            cmp     ch, #'+' 
            beq     skipSign 

            cmp     ch, #'-' 
            cinc    sign, sign, eq  // W2 
            bne     noSign 

skipSign:   ldrb    ch, [x0, #1]!   // Skip '-' 
noSign: 
```

紧接着一个符号字符（或者如果没有可选的符号字符）之后，字符串必须包含一个十进制数字字符或一个小数点。代码测试这两种条件之一，如果条件失败，则报告转换错误：

```
// Listing9-17.S (cont.) 

          ❶ sub     ch, ch, #'0'    // Quick test for '0' to '9' 
            cmp     ch, #9 
            bls     scanDigits      // Branch if '0' to '9' 

          ❷ cmp     ch, #'.'-'0'    // Check for '.' 
            bne     convError 

            // If the first character is a decimal point, 
            // the second character needs to be a 
            // decimal digit. 

          ❸ ldrb    ch, [x0, #1]!   // W5 Skip period. 
            cmp     ch, #'0' 
            blo     convError 
 cmp     ch, #'9' 
            bhi     convError 
            b.al    whileDigit2 
```

这段代码使用了一种常见的技巧来检查字符是否在'0'到'9'的范围内。它通过从字符❶的 ASCII 码中减去'0'的 ASCII 码来实现。如果字符在'0'到'9'的范围内，这将把它的值转换为 0 到 9 的范围。对 9 进行一次*无符号*比较，告诉我们字符值是否在'0'到'9'的范围内。如果是，这段代码将控制权转交给处理小数点左边数字的代码。

因为代码已将字符的 ASCII 码减去'0'，所以不能简单地将字符与小数点进行比较。`cmp ch, #'.'-'0'`指令通过从'.'中减去'0'的字符编码来正确地比较字符是否是小数点❷。如果字符是小数点，代码将验证后面的字符是否也是数字❸。

接下来，从`scanDigits`开始的代码处理小数点左侧的尾数数字（如果有）：

```
// Listing9-17.S (cont.) 
//
// Scan for digits at the beginning of the number: 

scanDigits: mov     ten, #10        // X7 used to multiply by 10 
            add     ch, ch, #'0'    // Restore character. 
 whileADigit: 
            sub     ch, ch, #'0'    // Quick way to test for 
            cmp     ch, #10         // a range and convert 
            bhs     notDigit        // to an integer 

            // Ignore any leading 0s in the number. 
            // You have a leading '0' if the mantissa is 0 
            // and the current character is '0'. 

          ❶ cmp     mant, xzr       // Ignore leading 0s. 
            ccmp    ch, #0, #0, eq 
            beq     Beyond16 

            // Each digit to the left of the decimal 
            // point increases the number by an 
            // additional power of 10\. Deal with that 
            // here. 

          ❷ add     fpExp, fpExp, #1 

            // Save all the significant digits but ignore 
            // any digits beyond the 16th digit. 

          ❸ cmp     sigDig, xzr     // X1 
            beq     Beyond16 

            // Count down the number of significant digits. 

            sub     sigDig, sigDig, #1 

 // Multiply the accumulator (mant) by 10 and 
            // add in the current digit. Note that ch 
            // has already been converted to an integer. 

          ❹ madd    mant, mant, ten, xch    // X0, X6, X5 

            // Because you multiplied the exponent by 10, 
            // you need to undo the increment of fpExp. 

          ❺ sub     fpExp, fpExp, #1 

Beyond16:   ldrb    ch, [x0, #1]!   // Get next char. 
            b.al    whileADigit 
```

这段代码通过检查如果尾数值为 0 且当前字符是'0'，则是前导零❶，从而跳过前导零。每处理一个尾数数字，代码会通过将尾数乘以 10 并加上数字的数值来调整尾数值❹。然而，如果循环处理超过 16 个有效数字❸，它不会将字符添加到尾数累加器中（因为双精度对象支持最多 16 位有效数字）。如果输入字符串超过 16 位有效数字，代码将递增`fpExp`变量❷来追踪最终的指数值。如果尾数乘以了 10（这种情况下指数不需要递增），代码会在❺处撤销这一增量。

下一部分代码处理小数点后的数字：

```
// Listing9-17.S (cont.) 
//
// If you encountered a nondigit character, 
// check for a decimal point: 

notDigit: 
            cmp     ch, #'.'-'0'    // See if a decimal point. 
            bne     whileDigit2 

// Okay, process any digits to the right of the decimal point. 
// If this code falls through from the above, it skips the 
// decimal point. 

getNextChar: 
            ldrb    ch, [x0, #1]!   // Get the next character. 
whileDigit2: 
            sub     ch, ch, #'0' 
            cmp     ch, #10 
            bhs     noDigit2 

            // Ignore digits after the 16th significant 
            // digit but don't count leading 0s 
            // as significant digits: 

          ❶ cmp     mant, xzr            // Ignore leading 0s. 
            ccmp    ch, wzr, #0, eq 
 ccmp    sigDig, xzr, #0, eq  // X2 
            beq     getNextChar 

            // Each digit to the right of the decimal point decreases 
            // the number by an additional power of 10\. Deal with 
            // that here. 

          ❷ sub     fpExp, fpExp, #1 

            // Count down the number of significant digits: 

            sub     sigDig, sigDig, #1 

            // Multiply the accumulator (mant) by 10 and 
            // add in the current digit. Note that ch 
            // has already been converted to an integer: 

            Madd    mant, mant, ten, xch    // X1, X7, X6 
            b.al    getNextChar 
```

这段代码与处理左侧数字相似，不同之处在于它每处理一个数字就递减运行中的指数值❷。这是因为尾数被作为整数维护，代码通过乘以 10 并将数字的值加进去，继续将小数部分的数字插入尾数。如果有效数字的总数超过 16 位（不包括前导零❶），该函数将忽略任何后续的数字。

接下来是处理字符串的可选指数部分：

```
// Listing9-17.S (cont.) 

❶ noDigit2: 
            mov     expSign, wzr    // W5 Initialize exp sign. 
            mov     expAcc, xzr     // X2 Initialize exponent. 
            cmp     ch, #'e'-'0' 
            beq     hasExponent 
            cmp     ch, #'E'-'0' 
            bne     noExponent 

❷ hasExponent: 
            ldrb    ch, [x0, #1]!           // Skip the "E".
            cmp     ch, #'-'                // W6 
            cinc    expSign, expSign, eq    // W5 
            beq     doNextChar_2 
            cmp     ch, #'+' 
            bne     getExponent 

doNextChar_2: 
            ldrb    ch, [x0, #1]!   // Skip '+' or '-'. 

// Okay, you're past the "E" and the optional sign at this 
// point. You must have at least one decimal digit. 

❸ getExponent: 
            sub     ch, ch, #'0'    // W5 
            cmp     ch, #10 
            bhs     convError 

            mov     expAcc, xzr     // Compute exponent value in X2\. 
ExpLoop:    ldrb    ch, [x0], #1 
            sub     ch, ch, #'0' 
            cmp     ch, #10 
            bhs     ExpDone 

            madd    expAcc, expAcc, ten, xch    // X2, X7, X6 
            b.al    ExpLoop 

// If the exponent was negative, negate your computed result: 

❹ ExpDone: 
            cmp     expSign, #false // W5 
            beq     noNegExp 

            neg     expAcc, expAcc  // X2 

noNegExp: 

// Add in the computed decimal exponent with the exponent 
// accumulator: 

          ❺ add     fpExp, fpExp, expAcc    // X4, X2 

noExponent: 

// Verify that the exponent is from -324 to +308 (which 
// is the maximum dynamic range for a 64-bit FP value): 

          ❻ mov     x5, #308        // Reuse expSign here. 
            cmp     fpExp, x5 
            bgt     voor            // Value out of range 
            mov     x5, #-324 
            cmp     fpExp, x5 
            blt     voor 
          ❼ ucvtf   d0, mant        // X1 
```

这段代码首先检查是否有'e'或'E'字符，表示指数的开始❶。如果字符串有指数，代码会检查是否有可选的符号字符❷。如果存在'-'字符，代码将`expSign`设置为 1（默认是 0），以指定指数为负。

处理完指数符号后，代码期望看到小数点后的数字❸，并将这些数字转换为整数（保存在`expAcc`变量中）。如果`expSign`为真（非零），代码将对`expAcc`中的值取反❹。然后，指数代码将`expAcc`加到在处理尾数数字时获得的指数值上，以得到实际的指数值❺。

最后，代码检查指数是否在-324 到+308 的范围内❻。这是 64 位双精度浮点值的最大动态范围。如果指数超出此范围，代码将返回一个值超出范围的错误。

此时，代码已经完全处理了字符串数据，X0 寄存器指向内存中不再是浮动点值的第一个字节。为了将尾数和指数值从整数转换为双精度值，首先使用 ucvtf 指令❼将尾数值（在 mant 中）转换为浮动点值。

接下来，处理指数有些棘手。fpExp 变量包含十进制指数，但这是一个表示 10 的幂的整数值。你必须将 D0 中的值（尾数）乘以 10^(fpExp)，但不幸的是，ARM 指令集没有提供一个可以计算 10 的整数幂的指令。你需要编写自己的代码来实现这一点：

```
// Listing9-17.S (cont.) 
//
// Okay, you have the mantissa into D0\. Now multiply 
// D0 by 10 raised to the value of the computed exponent 
// (currently in fpExp). 
//
// This code uses power-of-10 tables to help make the 
// computation a little more accurate. 
//
// You want to determine which power of 10 is just less than the 
// value of our exponent. The powers of 10 you are checking are 
// 10**256, 10**128, 10**64, 10**32, and so on. A slick way to 
// check is by shifting the bits in the exponent 
// to the left. Bit #8 is the 256 bit, so if this bit is set, 
// your exponent is >= 10**256\. If not, check the next bit down 
// to see if your exponent >= 10**128, and so on. 

            mov     x1, -8      // Initial index into power-of-10 table 
            cmp     fpExp, xzr  // X4 
            bpl     positiveExponent 

          ❶ // Handle negative exponents here: 

            neg     fpExp, fpExp 
            lsl     fpExp, fpExp, #55   // Bits 0..8 -> 55..63 
            lea     x6, PotTbl 
 ❷ whileExpNE0: 
            add     x1, x1, #8          // Next index into PotTbl. 
            adds    fpExp, fpExp, fpExp // (LSL) Need current POT? 
            bcc     testExp0 

            ldr     d1, [x6, x1] 
            fdiv    d0, d0, d1 

testExp0:   cmp     fpExp, xzr 
            bne     whileExpNE0 
            b.al    doMantissaSign 

// Handle positive exponents here. 

❸ positiveExponent: 
            lea     x6, PotTbl 
            lsl     fpExp, fpExp, #55       // Bits 0..8 -> 55..63 
            b.al    testExpis0_2 

whileExpNE0_2: 
            add     x1, x1, #8 
            adds    fpExp, fpExp, fpExp     // (LSL) 
            bcc     testExpis0_2 

            ldr     d1, [x6, x1] 
            fmul    d0, d0, d1 

testExpis0_2: 
            cmp     fpExp, xzr 
            bne     whileExpNE0_2 
```

这段代码使用了两段几乎相同的代码来处理负❶和正❸指数。两段代码的区别在于选择了 fdiv 指令（用于负指数）或 fmul 指令（用于正指数）。每个部分包含一个循环❷，该循环遍历 PotTbl（10 的幂）表中的每个条目。指数是一个 9 位值，因为最大无符号指数值为 324，可以容纳在 9 位或更少的位中。

对于此整数中的每个设置位，代码必须将浮动点结果乘以来自 PotTbl 的相应 10 的幂。例如，如果第 9 位被设置，则将尾数乘以或除以 10²⁵⁶（PotTbl 中的第一个条目）；如果第 8 位被设置，则将尾数乘以或除以 10¹²⁸（PotTbl 中的第二个条目），...；如果第 0 位被设置，则将尾数乘以或除以 10⁰（PotTbl 中的最后一个条目）。代码中的两个循环通过将 9 个位移入 fpExp 的高位位置来完成此操作，然后逐位移出，并在设置进位标志时进行乘法（对于正指数）或除法（对于负指数），使用 PotTbl 中的连续条目。

接下来，如果值为负数（标志保存在 sign 变量中），则代码将其取反，并将浮动点值返回给调用者，保存在 D0 寄存器中：

```
// Listing9-17.S (cont.) 

doMantissaSign: 
            cmp     sign, #false            // W3 
            beq     mantNotNegative 

            fneg    d0, d0 

// Successful return here. Note: does not restore X0 
// on successful conversion. 

mantNotNegative: 
            msr     nzcv, xzr   // clr c = no error 
            ldp     x1, x2, [fp, #sr.x1x2] 
            ldp     x3, x4, [fp, #sr.x3x4] 
            ldp     x5, x6, [fp, #sr.x5x6] 
            ldr     x7,     [fp, #sr.x7x0] 
            ldr     d1,     [fp, #sr.d1  ] 
            leave 
```

在成功转换时，此函数返回 X0，指向浮动点字符串后面的第一个字符。此代码在成功转换时不会将 X0 恢复到原始值。

strToR64 函数的最后部分是错误处理代码：

```
// Listing9-17.S (cont.) 
//
// Error returns down here. Returns error code in X0: 

refNULL:    mov     x1, #-3 
            b.al    ErrorExit 

convError:  mov     x1, #-2 
            b.al    ErrorExit 

voor:       mov     x1, #-1 // Value out of range 
            b.al    ErrorExit 

illChar:    mov     x1, #-4 

// Note: on error, this code restores X0\. 

ErrorExit: 
            str     x1, [fp, #sr.x1x2]  // Return error code in X1\. 
            mrs     x1, nzcv            // Return error in carry flag. 
            orr     x1, x1, #(1 << 29) 
            msr     nzcv, x1            // Set c = error. 
            ldp     x1, x2, [fp, #sr.x1x2] 
            ldp     x3, x4, [fp, #sr.x3x4] 
            ldp     x5, x6, [fp, #sr.x5x6] 
            ldp     x7, x0, [fp, #sr.x7x0] 
            ldr     d1,     [fp, #sr.d1  ] 
            leave 

            endp    strToR64 
```

每个错误返回一个特殊的错误代码到 X1。所以此代码在返回时不会恢复 X1。与成功返回不同，错误返回代码将恢复 X0 到其原始值。

最后，asmMain 函数包含一个循环，处理通过 values 数组中的指针找到的每个字符串。它简单地遍历每个指针，并将其传递给 strToR64，直到遇到 NULL（0）值：

```
// Listing9-17.S (cont.) 

// Here is the asmMain function: 

            proc    asmMain, public 

            locals  am 
            dword   am.x20 
            byte    stack, 64 
            endl    am 

            enter   am.size 
            str     x20, [fp, #am.x20] 

// Test floating-point conversion: 

            lea     x20, values 
ValuesLp:   ldr     x0, [x20] 
            cmp     x0, xzr 
            beq     allDone 
            bl      strToR64 

            lea     x0, fmtStr1 
            ldr     x1, [x20] 
            mstr    x1, [sp] 
            mstr    d0, [sp, #8] 
            bl      printf 
            add     x20, x20, #8 
            b.al    ValuesLp 

allDone:    ldr     x20, [fp, #am.x20] 
            leave 
            endp    asmMain 
```

这是清单 9-17 的构建命令和示例输出：

```
% ./build Listing9-17 
% ./Listing9-17 
Calling Listing9-17: 
strToR64: str=' 1.234e56', value=1.234000e+56 
strToR64: str='    -1.234e+56', value=-1.234000e+56 
strToR64: str='1.234e-56', value=1.234000e-56 
strToR64: str='-1.234e-56', value=-1.234000e-56 
strToR64: str='1.23', value=1.230000e+00 
strToR64: str='-1.23', value=-1.230000e+00 
strToR64: str='001.23', value=1.230000e+00 
strToR64: str='-001.23', value=-1.230000e+00 
strToR64: str='1', value=1.000000e+00 
strToR64: str='-1', value=-1.000000e+00 
strToR64: str='0.1', value=1.000000e-01 
strToR64: str='-0.1', value=-1.000000e-01 
strToR64: str='0000000.1', value=1.000000e-01 
strToR64: str='-0000000.1', value=-1.000000e-01 
strToR64: str='0.1000000', value=1.000000e-01 
strToR64: str='-0.1000000', value=-1.000000e-01 
strToR64: str='0.0000001', value=1.000000e-07 
strToR64: str='-0.0000001', value=-1.000000e-07 
strToR64: str='.1', value=1.000000e-01 
strToR64: str='-.1', value=-1.000000e-01 
strToR64: str='123456', value=1.234560e+05 
strToR64: str='12345678901234567890', value=1.234568e+19 
strToR64: str='0', value=0.000000e+00 
strToR64: str='1.', value=1.000000e+00 
strToR64: str='0.000000000000000000001', value=1.000000e-17 
Listing9-17 terminated 
```

将实数到字符串以及字符串到实数的程序修改成进行“往返”转换，即从实数到字符串再到实数，看看是否能得到接近输入的相同结果，这将是一个有趣的尝试。（由于舍入和截断错误，你并不总是能得到完全相同的值，但应该会很接近。）我会留给你自己去尝试这个。

### 9.4 其他数值转换

本章介绍了更常见的数值转换算法：十进制整数、十六进制整数和浮动点。其他的转换有时也很有用。例如，一些应用程序可能需要八进制（基数为 8）转换或任意基数的转换。对于基数为 2 到 9 的情况，算法实际上与十进制整数转换是一样的，唯一的区别是，不是除以 10（并取余数），而是除以所需的基数。事实上，编写一个通用函数，传入基数（radix）即可获得相应的转换，是相当简单的。

当然，基数为 2 的输出几乎是微不足道的，因为 ARM CPU 内部存储的值是二进制的。你需要做的就是从数值中移出位（进入进位标志），并根据进位的状态输出 0 或 1。基数为 4 和基数为 8 的转换也相对简单，分别操作 2 位或 3 位的组。

一些浮动点格式不遵循 IEEE 标准。为了处理这些情况，可以编写一个函数，将这些格式转换为 IEEE 格式（如果可能的话），然后使用本章的示例来进行浮动点和字符串之间的转换。如果你需要直接处理这些格式，本章中的算法应该足够通用，并且易于修改以适应你的需求。

### 9.5 继续前进

本章详细讨论了两个主要话题：将数值转换为字符串和将字符串转换为数值。对于前者，本章介绍了数值到十六进制转换（字节、半字、字、双字和四字）、数值到无符号十进制转换（64 位和 128 位）以及数值到有符号十进制转换（64 位和 128 位）。同时，讨论了在进行数值到字符串转换时，如何通过格式化转换来控制输出格式，还讨论了格式化浮动点到字符串的转换，包括十进制和指数格式，以及计算转换所需的打印位置数。

在讨论字符串到数值的转换时，本章介绍了将无符号十进制字符串转换为数值形式、将有符号十进制字符串转换为数值形式、将十六进制字符串转换为数值形式以及将浮动点字符串转换为双精度数值形式。最后，本章简要讨论了其他可能的数值输出格式。

虽然本书将继续使用 C 语言的 printf()函数进行格式化输出，但你可以使用本章中的方法，在编写自己的汇编代码时避免依赖 C 语言。这些方法也构成了一个汇编语言库的基础，您可以利用它简化汇编代码的编写。

### 9.6 更多信息

+   唐纳德·克努斯的*《计算机程序设计艺术，第 2 卷：半数值算法》*（第三版，Addison-Wesley Professional，1997 年）包含了许多有关十进制算术和扩展精度算术的有用信息，尽管该文本是通用的，并没有描述如何在 ARM 汇编语言中实现这些内容。

+   关于通过乘以倒数进行除法的更多信息，请参阅爱荷华大学的教程，地址为*[`<wbr>homepage<wbr>.cs<wbr>.uiowa<wbr>.edu<wbr>/~jones<wbr>/bcd<wbr>/divide<wbr>.html`](http://homepage.cs.uiowa.edu/~jones/bcd/divide.html)*。
