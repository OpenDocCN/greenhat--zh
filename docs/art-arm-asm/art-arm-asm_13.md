

## 第十章：10 表格查找



![](img/opener.jpg)

在早期的汇编语言编程中，用表格查找替代昂贵的计算是提高程序性能的常见方法。今天，现代系统中的内存速度限制了通过使用表格查找所能获得的性能提升。然而，对于非常复杂的计算，这仍然是一种编写高性能代码的可行技术。

本章讨论了如何使用表格查找来加速或减少计算的复杂度，展示了其中涉及的空间和速度的权衡。

### 10.1 在汇编语言中使用表格

对于汇编语言程序员来说，*表格* 是一个包含初始化值的数组，这些值在创建后不会发生变化。在汇编语言中，你可以使用表格来实现多种功能：计算函数、控制程序流，或仅仅用于查找数据。一般来说，表格提供了一种快速执行操作的机制，但代价是程序中占用了额外的空间（这些额外的空间存放了表格数据）。

在本节中，我们将探索在汇编语言程序中使用表格的多种可能方式。请记住，由于表格通常包含在程序执行过程中不会变化的初始化数据，因此 .section .rodata，"" 部分是放置表格对象的好地方。

#### 10.1.1 通过表格查找进行函数计算

看似简单的 HLL 算术表达式可能等价于大量的 ARM 汇编语言代码，因此计算可能非常昂贵。汇编语言程序员通常会预先计算许多值，并通过查找这些值来加速程序，这既容易实现，通常也更高效。

考虑以下 Pascal 语句：

```
if (character >= 'a') and (character <= 'z') then
    character := chr(ord(character) - 32);
```

该 if 语句将字符变量的值从小写字母转换为大写字母（如果该字符在 a 到 z 范围内）。相应的汇编代码需要七条机器指令，具体如下：

```
 mov  w1, #'z'
    ldrb w0, [fp, #character]  // Assume "character" is local.
    cmp  w0, #'a'
  ❶ ccmp w0, w1, #0b0010, hs
    bhi  notLower
  ❷ eor  w0, w0, #0x20
notLower:
    strb w0, [fp, #character]
```

NZCV 常量 0b0010 设置进位标志并清除 0，这样当 W0 小于 'a' 时（即 W0 小于 'a' 时，进位标志被设置，零标志被清除，表示“更大或相同”但没有相同部分，因此只是更大）❶，分支会被执行。请注意，条件比较指令只允许 5 位立即数常量；这就是为什么代码将字符常量 'z' 加载到 W1 中并与 W1 进行条件比较的原因。

将小写字母转换为大写字母的常见方法是清除 ASCII 字符代码的第 5 位。但是，w0, w0, #0x5F 并不是一条合法的指令，因为 0x5F 不是一个合法的逻辑常量。该代码使用 eor（异或）指令来反转第 5 位 ❷。因为此时第 5 位必定被设置（所有小写字母的第 5 位都被设置），所以 eor 指令会清除这一位。

查找表解决方案只使用四条指令：

```
lea  x1, xlatTbl
ldrb w0, [fp, #character]
ldrb w0, [x1, w0, uxt2 #0]
strb w0, [fp, #character]
```

转换逻辑完全隐藏在查找表（xlatTbl）中。这个是一个 256 字节的数组；每个索引包含索引值（元素 0 包含值 0，元素 1 包含值 1，依此类推），除了对应小写字符 ASCII 代码的索引（索引 97 到 122）。这些特定的数组元素包含大写字母的 ASCII 代码（值 65 到 90）。

请注意，如果你可以确保只加载 7 位 ASCII 字符到此代码中，你可以使用 128 字节（而不是 256 字节）的数组来实现。

这是一个典型的（128 字节）查找表，用于将小写字母转换为大写字母：

```
xlatTbl:    .byte       0,1,2,3,4,5,6,7
            .byte       8,9,10,11,12,13,14,15
            .byte       16,17,18,19,20,21,22,23
            .byte       24,25,26,27,28,29,30,31
            .byte       32,33,34,35,36,37,38,39
            .byte       40,41,42,43,44,45,46,47
            .byte       48,49,50,51,52,53,54,55
            .byte       56,57,58,59,60,61,62,63
            .byte       64
            .ascii      "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
            .byte       91,92,93,94,95,96
            .ascii      "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
            .byte       123,124,125,126,127
```

如果你需要一个完整的 256 字节表，索引从 128 到 255 的元素将包含从 128 到 255 的值。

`ldrb w0, [x1, w0, uxtw #0]` 指令将 W0 加载为由 W0 中保存的（原始）值指定的索引处的字节，假设 X1 保存的是 xlatTbl 的地址。如果 W0 保存的是非小写字符的代码，索引到该表将把相同的值加载到 W0（所以如果 W0 不是小写字母，这条指令不会改变 W0 的值）。如果 W0 包含小写字母，索引到此表会获取相应大写字母的 ASCII 代码。

Listing 10-1 演示了这两种形式的大小写转换：`if...eor` 和表查找。

```
// Listing10-1.S
//
// Lowercase-to-uppercase conversion

            #include    "aoaa.inc"

            .section    .rodata, ""

ttlStr:     .asciz      "Listing 10-1"

textStr:    .ascii      "abcdefghijklmnopqrstuvwxyz\n"
            .ascii      "ABCDEFGHIJKLMNOPQRSTUVWXYZ\n"
            .asciz      "0123456789\n"

// Translation table to convert lowercase to uppercase:

xlatTbl:    .byte       0, 1, 2, 3, 4, 5, 6, 7
            .byte       8, 9, 10, 11, 12, 13, 14, 15
            .byte       16, 17, 18, 19, 20, 21, 22, 23
            .byte       24, 25, 26, 27, 28, 29, 30, 31
            .byte       32, 33, 34, 35, 36, 37, 38, 39
            .byte       40, 41, 42, 43, 44, 45, 46, 47
            .byte       48, 49, 50, 51, 52, 53, 54, 55
            .byte       56, 57, 58, 59, 60, 61, 62, 63
            .byte       64
            .ascii      "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
            .byte       91, 92, 93, 94, 95, 96
            .ascii      "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
            .byte       123, 124, 125, 126, 127

// Various printf format strings this program uses:

fmtStr1:    .asciz      "Standard conversion:\n"
fmtStr2:    .asciz      "\nConversion via lookup table:\n"
fmtStr:     .asciz      "%c"

            .code
            .extern     printf

////////////////////////////////////////////////////////////////////
//
// Return program title to C++ program:

            proc        getTitle, public
            lea         x0, ttlStr
            ret
            endp        getTitle

////////////////////////////////////////////////////////////////////
//
// Here is the asmMain function:

            proc    asmMain, public

            locals  am
            dword   am.x20
            dword   am.x21
            byte    am.shadow, 64
            endl    am

            enter   am.size
            str     x20, [fp, #am.x20]
            str     x21, [fp, #am.x21]

// Print first title string:

            lea     x0, fmtStr1
            bl      printf

// Convert textStr to uppercase using
// standard "if and EOR" operation:

            lea     x20, textStr    // String to convert
            mov     x21, #'z'       // CCMP doesn't like #'z'.
            b.al    testNot0

// Check to see if W1 is in the range 'a'..'z'. If so,
// invert bit 5 to convert it to uppercase:

stdLoop:    cmp     w1, #'a'
            ccmp    w1, w21, #0b0010, hs
            bhi     notLower
            eor     w1, w1, #0x20
notLower:

// Print the converted character:

            lea     x0, fmtStr
            mstr    x1, [sp]
            bl      printf

// Fetch the next character from the string:

testNot0:   ldrb    w1, [x20], #1
            cmp     w1, #0
            bne     stdLoop

// Convert textStr to uppercase by using
// a lookup table. Begin by printing
// an explanatory string before the
// output:

            lea x0, fmtStr2
            bl      printf

// textStr is the string to convert.
// xlatTbl is the lookup table that will convert
// lowercase characters to uppercase:

            lea     x20, textStr
            lea     x21, xlatTbl
            b.al    testNot0a

// Convert the character from lowercase to
// uppercase via a lookup table:

xlatLoop:   ldrb    w1, [x21, w1, uxtw #0]

// Print the character:

            lea     x0, fmtStr
            mstr    x1, [sp]
            bl      printf

// Fetch the next character from the string:

testNot0a:  ldrb    w1, [x20], #1
            cmp     w1, #0
            bne     xlatLoop

allDone:    ldr     x20, [fp, #am.x20]
            ldr     x21, [fp, #am.x21]
            leave   // Returns to caller
            endp    asmMain
```

这是 Listing 10-1 的构建命令和示例输出：

```
% ./build Listing10-1
% ./Listing10-1
Calling Listing10-1:
Standard conversion:
ABCDEFGHIJKLMNOPQRSTUVWXYZ
ABCDEFGHIJKLMNOPQRSTUVWXYZ
0123456789

Conversion via lookup table:
ABCDEFGHIJKLMNOPQRSTUVWXYZ
ABCDEFGHIJKLMNOPQRSTUVWXYZ
0123456789
Listing10-1 terminated
```

我没有尝试对两种版本进行计时，因为对 `printf()` 的调用主导了两个算法的执行时间。然而，由于查找表算法在每个字符上访问内存（从查找表中获取字节），即使它使用了更少的指令，过程也并不更短。查找表增加了程序代码的 128 字节（或 256 字节）。

对于像小写字母转大写字母这样的简单计算，使用查找表几乎没有什么好处。但随着计算复杂度的增加，查找表算法可能会变得更快。考虑以下交换大小写的代码（将小写字母转换为大写字母，反之亦然）：

```
// If it's lowercase, convert it to uppercase:

    mov  w1, #'z'
    ldrb w0, [fp, #character]  // Assume "character" is local.
    cmp  w0, #'a'
    ccmp w0, w1, #0b0010, hs
 bhi  notLower
    eor  w0, w0, #0x20
    b.al allDone

// If it's uppercase, convert it to lowercase:

notLower:
    mov  w1, #'Z'
    cmp  w0, #'A'
    ccmp w0, w1, #0b0010, hs
    bhi  allDone
    eor  w0, w0, #0x20

allDone:
    strb w0, [fp, #character]
```

查找表版本几乎与 Listing 10-1 相同。只是查找表中的值发生了变化：

```
 lea  x1, xlatTbl2
    ldrb w0, [fp, #character]
    ldrb w0, [x1, w0, uxtw #0]
    strb w0, [fp, #character]
```

xlatTbl2 数组将包含与大写字母对应的索引位置的小写 ASCII 代码，同时也会在与小写 ASCII 代码对应的索引位置保存大写 ASCII 代码。

这个大小写转换算法可能仍然不足够复杂，无法证明使用查找表来提高性能是合理的。然而，它表明随着算法复杂度的增加（如果没有查找表则执行时间更长），查找表算法的执行时间保持恒定。

#### 10.1.2 函数域与范围

通过查找表计算的函数有一个有限的*定义域*，即它们接受的所有可能输入值的集合。这是因为函数定义域中的每个元素都需要在查找表中有一个条目。例如，之前的大小写转换函数的定义域是 256 字符的扩展 ASCII 字符集。像 sin()或 cos()这样的函数接受的是（无限的）实数集作为可能的输入值。你不会发现通过查找表实现一个定义域为实数集的函数非常实用，因为你必须将定义域限制为一个较小的集合。

大多数查找表相当小，通常只有 10 到 256 个条目。它们很少会超过 1,000 个条目。大多数程序员没有足够的耐心去创建和验证一个 1,000 条目的表（但请参阅第 10.1.4 节，“表生成”，在第 615 页讨论如何通过编程生成表）。

基于查找表的函数的另一个限制是，定义域中的元素必须相当连续。查找表使用函数的输入值作为查找表的索引，并返回该条目处的值。一个接受 0、100、1,000 和 10,000 作为输入值的函数需要 10,001 个元素，因为输入值的范围。因此，你不能通过查找表高效地创建这样的函数。本节讨论的查找表假设函数的定义域是一个相当连续的值集。

一个函数的*范围*是它所产生的所有可能输出值的集合。从查找表的角度来看，函数的范围决定了每个表项的大小。例如，如果函数的范围是整数值 0 到 255，则每个表项需要一个字节；如果范围是 0 到 65,535，则每个表项需要 2 个字节，依此类推。

你可以通过查找表实现的最佳函数是那些其定义域和范围始终为 0 到 255（或该范围的子集）的函数。任何这样的函数都可以通过以下两条指令来计算：

```
lea x1, table
ldrb w0, [x1, w0, uxtw #0]
```

唯一改变的是查找表。之前介绍的大小写转换程序是这种函数的好例子。

如果函数的定义域或范围不是 0 到 255，查找表的效率会稍微降低。如果一个函数的定义域超出了 0 到 255 的范围，但其范围落在该值集合之内，那么你的查找表将需要超过 256 个条目，但你可以用一个字节表示每个条目。因此，查找表可以是一个字节数组。C/C++函数调用

```
B = Func(X);
```

其中 Func 是

```
byte Func(word `parm)` {...}
```

它可以很容易地转换为以下 ARM 代码：

```
lea  x1, FuncTbl
ldr  w0, X       // Using appropriate addressing mode
ldrb w0, [x1, w0, uxtw #0]
strb w0, B       // Using appropriate addressing mode
```

这段代码将函数参数加载到 W0 中，使用该值（在 0 到*maxParmValue* 范围内）作为 FuncTbl 表的索引，获取该位置的字节，并将结果存储到 B 中。显然，表中必须为 X 的每个可能值（最多为 *maxParmValue*）包含有效条目。例如，假设你想要将 80 × 25 文本视频显示器上的光标位置（范围为 0 到 1999，80 × 25 显示器有 2000 个字符位置）映射到屏幕上的 X（0 到 79）或 Y（0 到 24）坐标。

你可以通过这个函数计算 X 坐标

```
X = Posn % 80;
```

并使用以下公式计算 Y 坐标：

```
Y = Posn / 25;
```

以下代码通过表查找实现这两个函数，可能会提高代码的性能，特别是在频繁访问表且表位于处理器缓存中的情况下：

```
lea  x2, xTbl
lea  x3, yTbl
ldr  w4, Posn   // Using an appropriate addressing mode
ldrb w0, [x2, w4, uxtw #0] // Get X.
ldrb w1, [x3, w4, uxtw #0] // Get Y.
```

给定 xTbl 和 yTbl 中的适当值，这将把 x 坐标留在 W0 中，y 坐标留在 W1 中。

如果一个函数的定义域在 0 到 255 之间，但其值域超出该范围，查找表将包含 256 个或更少的条目，但每个条目将需要 2 个或更多字节。如果函数的值域和定义域都超出 0 到 255 的范围，每个条目将需要 2 个或更多字节，并且表格将包含超过 256 个条目。

回顾第四章，索引单维数组（表格是特殊情况）的公式如下：

```
`Element_Address` = `Base` + `Index` × `Element_Size`
```

如果函数值域中的元素需要 2 个字节，你必须将索引乘以 2，然后才能索引到表中。同样，如果每个条目需要 3、4 或更多字节，则在作为索引使用之前，必须将索引乘以每个表项的字节大小。例如，假设你有一个函数 F(x)，由以下 C/C++ 声明定义：

```
short F(word x) {...} // short is a half word (16 bits).
```

你可以通过使用以下 ARM 代码创建这个函数（当然，还有合适的表格，名为 F）：

```
lea  x1, F
ldrh w0, x    // Using an appropriate addressing mode
ldrh w0, [x1, w0, uxtw #1] // Shift left does multiply by 2.
```

任何定义域较小且大多是连续的函数，都适合通过表查找进行计算。在某些情况下，不连续的定义域也是可以接受的，只要能够将定义域转化为合适的值集合（之前讨论过的例子是处理 `switch` 语句的表达式）。这种操作叫做*条件化*，是下一节的主题。

#### 10.1.3 定义域条件化

*定义域条件化* 是指将函数定义域中的一组值进行处理，使它们更适合作为该函数的输入。考虑以下函数：

```
sin x = sin x|(x∈[-2π,2π])
```

这表示（计算机）函数 sin(x) 等价于（数学）函数 sin *x*，其中：

```
-2π <= x <= 2π
```

正如你所知道的，正弦是一个循环函数，它可以接受任何实数输入。然而，用于计算正弦的公式只接受这些值中的一小部分。这个范围限制不会带来实际问题；只需通过计算 sin(y mod (2π))，你就能计算出任何输入值的正弦。修改输入值，以便你能够轻松计算函数的过程叫做 *输入条件调整*。前面的例子计算了 (x % 2) * pi，并将结果作为 sin() 函数的输入。这会将 x 截断到 sin() 所需的领域，而不会影响结果。

你也可以将输入条件调整应用于表查找。实际上，缩放索引以处理字条目就是一种输入条件调整。考虑以下 C/C++ 函数：

```
short val(short x)
{
    switch (x)
    {
        case 0: return 1;
        case 1: return 1;
        case 2: return 4;
        case 3: return 27;
        case 4: return 256;
    }
    return 0;
}
```

这个函数计算 x 在 0 到 4 范围内的值，如果 x 超出此范围，则返回 0。由于 x 可以取 65,536 个值（作为一个 16 位字），创建一个包含 65,536 个字的表，其中只有前五个条目非零，似乎非常浪费。然而，如果使用输入条件调整，你仍然可以通过表查找来计算这个函数。以下汇编语言代码展示了这个原理：

```
 mov  w0, #0       // Result = 0, assume x > 4
    ldrh w1, [fp, #x] // Assume x is local.
 cmp  w1, #4       // See if in the range 0 to 4.
    bhi  outOfRange
    lea  x2, valTbl   // Address of lookup table
    ldrh w0, [x2, w1, uxtw #1] // index * 2 (half-word table)
outOfRange:
```

这段代码检查 x 是否超出 0 到 4 的范围。如果超出，它会手动将 W0 设置为 0；否则，它会通过 valTbl 表查找函数值。通过输入条件调整，你可以实现一些通过表查找通常无法做到的功能。

#### 10.1.4 表格生成

使用表查找的一个大问题是首先创建表格。如果表格包含许多条目，这个问题尤其突出。弄清楚表格中应该放入哪些数据，然后繁琐地输入这些数据，最后检查数据以确保其有效性，是一个既耗时又枯燥的过程。

对于许多表来说，这是无法避免的。然而，对于其他表，你可以利用计算机为你生成表。我将通过一个例子来解释这一点。考虑以下对正弦函数的修改：

$方程$![](img/pg615.jpg)

这说明 *x* 是一个在 0 到 359（度）范围内的整数，并且 *r* 必须是一个整数。计算机可以通过以下代码轻松计算此内容：

```
lea   x1, Sines     // Table of 16-bit values
ldr   w0, [fp, #x]  // Assume x is local.
ldrh  w0, [x1, w0, uxtw #1]  // index * 2 for half words
ldrh  w2, [fp, #r]  // Assume r is local.
sxth  x0, w0
sxth  x2, w2
smul  w0, w0, w2    // r *(1000 * sin(x))
mov   w2, #1000
sdiv  x0, x0, x2    // r *(1000 * sin(x))/ 1000
```

请注意，整数的乘法和除法不是结合律的。你不能简单地去掉乘以 1,000 和除以 1,000，因为它们看似互相抵消。此外，这段代码必须严格按照这个顺序来计算该函数。

完成此功能所需的所有内容是正弦值，一个包含 360 个值的表，表示角度（以度为单位）的正弦值乘以 1,000。列表 10-2 中的 C/C++ 程序生成了这个表。

```
// Listing10-2.cpp
//
// g++ -o Listing10-2 Listing10-2.c -lm
//
// GenerateSines
//
// A C program that generates a table of sine values for
// an assembly language lookup table

#include <stdlib.h>
#include <stdio.h>
#include <math.h>

int main(int argc, char **argv)
{
    FILE *outFile;
    int angle;
    int r;

    // Open the file:

    outFile = fopen("sines.inc", "w");

    // Emit the initial part of the declaration to
    // the output file:

    fprintf
    (
        outFile,
        "Sines:"  // sin(0) = 0
    );

    // Emit the Sines table:

    for(angle = 0; angle <= 359; ++angle)
    {
        // Convert angle in degrees to an angle in
        // radians using:
        //
        // radians = angle * 2.0 * pi / 360.0;
        //
        // Multiply by 1000 and store the rounded
        // result into the integer variable r.

        double theSine =
            sin
            (
                angle * 2.0 *
                3.14159265358979323846 /
                360.0
            );
        r = (int) (theSine * 1000.0);

        // Write out the integers eight per line to the
        // source file.
        // Note: If (angle AND %111) is 0, then angle
        // is divisible by 8 and you should output a
        // newline first.

 if((angle & 7) == 0)
        {
            fprintf(outFile, "\n\t.hword\t");
        }
        fprintf(outFile, "%5d", r);
        if ((angle & 7) != 7)
        {
            fprintf(outFile, ",");
        }

    } // endfor
    fprintf(outFile, "\n");

    fclose(outFile);
    return 0;

} // end main
```

编译并运行列表 10-2 中的程序会生成文件 *sines.inc*，该文件包含以下内容（为了简洁起见，已截断）：

```
Sines:
     .hword      0,   17,   34,   52,   69,   87,  104,  121
     .hword    139,  156,  173,  190,  207,  224,  241,  258
     .hword    275,  292,  309,  325,  342,  358,  374,  390
     .hword    406,  422,  438,  453,  469,  484,  499,  515
     .hword    529,  544,  559,  573,  587,  601,  615,  629
     .hword    642,  656,  669,  681,  694,  707,  719,  731
     .hword    743,  754,  766,  777,  788,  798,  809,  819
        .
        .
        .
     .hword   -529, -515, -500, -484, -469, -453, -438, -422
     .hword   -406, -390, -374, -358, -342, -325, -309, -292
     .hword   -275, -258, -241, -224, -207, -190, -173, -156
     .hword   -139, -121, -104,  -87,  -69,  -52,  -34,  -17
```

显然，编写生成这些数据的 C 程序比手动输入和验证这些数据要容易得多。你还可以使用 Pascal/Delphi、Java、C#、Swift 或其他高级语言（HLL）编写表格生成程序。由于该程序只会执行一次，因此其性能不是问题。

一旦你运行了表格生成程序，剩下的步骤就是从文件（本示例中的*sines.inc*）中剪切并粘贴表格到实际使用该表格的程序中（或者，使用#include "sines.inc"指令将文本包含到源文件中）。

### 10.2 表格查找性能

在早期的 PC 时代，表格查找是进行高性能计算的首选方式。如今，CPU 的速度通常是主存储器的 10 到 100 倍。因此，使用表格查找可能不比使用机器指令进行相同计算更快。然而，片上 CPU 缓存内存子系统的速度接近 CPU 速度。因此，如果你的表格存储在 CPU 的缓存内存中，表格查找可能是成本效益较高的选择。这意味着，从表格查找中获得良好性能的方法是使用小表格（因为缓存空间有限）并使用你频繁访问的表格项（以便表格保持在缓存中）。

最终，确定表格查找是否比计算更快的最佳方法是编写两种版本的代码并进行计时。尽管“1000 万次循环计时”方法可能足够用于粗略测量，但你可能还希望找到并使用一个合适的性能分析工具，它将提供更精确的计时结果。有关更多详细信息，请参阅“更多信息”部分。

### 10.3 继续前进

随着 CPU 速度的提升和内存访问时间未能跟上，使用表格查找来优化应用程序已逐渐不再流行。然而，本章简短地讨论了表格查找仍然有用的情况。它首先讨论了基本的表格查找操作，然后讲解了领域条件化和使用软件自动生成表格。最后总结了几条关于如何判断表格查找是否适合特定项目的建议。

在现代 CPU 中，多个核心和 SIMD 指令集是提高应用程序性能的常见方式。下一章将讨论 ARM Neon/SIMD 指令集，以及如何使用它来提高程序性能。

### 10.4 更多信息

+   唐纳德·克努斯（Donald Knuth）的*《计算机程序设计艺术》第三卷：查找与排序*（第二版，Addison-Wesley Professional，1998 年）包含了很多关于在表格中查找数据的有用信息。

+   请参阅我的书*《写出优秀代码》*（第一卷，第二版，No Starch Press，2020 年）或*《汇编语言艺术》*的电子版，了解有关缓存内存操作以及如何优化其使用的详细信息，网址是*[`www.randallhyde.com`](https://www.randallhyde.com)。

+   有关分析器程序的更多信息，请参阅 Maarten Balliauw 撰写的《在 macOS 和 Linux 上使用 dotTrace 入门》*，[`<wbr>blog<wbr>.jetbrains<wbr>.com<wbr>/dotnet<wbr>/2023<wbr>/02<wbr>/22<wbr>/getting<wbr>-started<wbr>-with<wbr>-dottrace<wbr>-on<wbr>-macos<wbr>-and<wbr>-linux`](https://blog.jetbrains.com/dotnet/2023/02/22/getting-started-with-dottrace-on-macos-and-linux)*。你还可以查看 Amrita Pathak 撰写的《13 款用于调试应用程序性能问题的分析软件》*，[`<wbr>geekflare<wbr>.com<wbr>/application<wbr>-profiling<wbr>-software<wbr>/`](https://geekflare.com/application-profiling-software/)*。
