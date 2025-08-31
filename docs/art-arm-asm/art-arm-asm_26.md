

## 第二十二章：F 问题的答案



## F.1 第一章

1.  *作为*

2.  地址、数据和控制

3.  PSTATE 寄存器

4.  (a) 4, (b) 8, (c) 16, (d) 8

5.  64 位

6.  bl

7.  ret

8.  应用程序二进制接口

9.  (a) W0 的低字节, (b) W0 的低半字, (c) W0, (d) X0, (e) X0

10.  X0, X1, X2, 和 X3 寄存器（分别）

## F.2 第二章

1.  9 × 10³ + 3 × 10² + 8 × 10¹ + 4 × 10⁰ + 5 × 10 ^(–1) + 7 × 10^(–2) + 6 × 10^(–3)

2.  (a) 10, (b) 12, (c) 7, (d) 9, (e) 3, (f) 15

3.  (a) A, (b) E, (c) B, (d) D, (e) 2, (f) C, (g) CF, (h) 98D1

4.  (a) 0001_0010_1010_1111

(b)  1001_1011_1110_0111

(c)  0100_1010

(d)  0001_0011_0111_1111

(e)  1111_0000_0000_1101

(f)  1011_1110_1010_1101

(g)  0100_1001_0011_1000

5.  (a) 10, (b) 11, (c) 15, (d) 13, (e) 14, (f) 12

6.  (a) 32, (b) 128, (c) 16, (d) 64, (e) 4, (f) 8, (g) 4

7.  (a) 4, (b) 8, (c) 16, (d) 2

8.  (a) 16, (b) 256, (c) 65,536, (d) 2

9.  4

10.  0 到 7

11.  位 0

12.  位 63

13.  (a) 0, (b) 0, (c) 0, (d) 1

14.  (a) 0, (b) 1, (c) 1, (d) 1

15.  (a) 0, (b) 1, (c) 1, (d) 0

16.  与 1 做异或（按位操作，寄存器中所有 1 位）

17.  与

18.  或

19.  非（XOR 也与所有 1 位进行异或）

20.  异或

21.  非（eor 也与所有 1 位进行异或）

22.  1111_1011

23.  0000_0010

24.  (a) 1111_1111b, (c) 1000_0000b, (e) 1000_0001b

25.  neg 指令

26.  (a) 1111_1111_1111_1111

(c)  000_0000_0000_0001

(d)  1111_1111_1111_0000

27.  b (b.al)

28.  标签：

29.  负标志（N），零标志（Z），进位标志（C），溢出标志（V）

30.  Z = 1

31.  C = 0 和 Z = 0

32.  bhi, bhs, bls, blo, beq, 和 bne 条件跳转指令

33.  bgt, bge, blt, ble, beq, 和 bne 条件跳转指令

34.  lsl 指令不会影响零标志位。

35.  乘以 2

36.  除以 2

37.  乘法和除法

38.  规范化的浮点值在高阶尾数位置有一个 1 位。

39.  7 位

40.  0x30 到 0x39

41.  撇号（或单引号）字符

## F.3 第三章

1.  PC 64 位寄存器

2.  操作码，机器指令的数字编码

3.  静态/标量变量和基于内存的常量

4.  大约 ±1 MB，使用 ldr 和 str 指令

5.  访问内存位置的地址

6.  (b) X0 和 (d) SP

7.  lea 宏（或 adr 和 adrp 指令）

8.  完成所有寻址模式计算后得到的最终地址

9.  使用 .align 3 指令将变量在 .data 区段对齐到 8 字节边界。

10.  内存管理单元

11.  计算内存对象（静态）运行时地址的算术表达式

12.  大端值将值的高位部分存储在较低的内存地址中，而小端值将值的低位部分存储在较低的内存地址中。

13.  rev32 指令

14.  rev16 指令

15.  rev 指令

16.  从 SP 减去 16，然后将值存储到 X0 中，存储位置由 SP 指向。

17.  从 SP 指向的地址加载 X0，然后将 16 加到 SP 寄存器。

18.  反转

19.  后进先出

## F.4 第四章

1.  一个常量的符号名称，汇编程序（或预处理器）将在汇编过程中用该常量的数值等效值替换

2.  使用 `.equ`、`.set` 和 `=` 指令。如果你的源文件名以 *.S* 结尾，你还可以使用 C 预处理器（CPP）的 `#define` 指令。

3.  常量表达式是一个算术表达式，Gas 可以在汇编过程中计算出该表达式的值。你通过计算由逗号分隔的表达式数量来确定字节指令操作数字段中的数据元素数量。

4.  当前在某个段中的偏移量（如 `.data` 或 `.text`）

5.  句点操作符（.）

6.  将第二个声明的标签减去第一个声明的标签（例如，第二 - 第一）。

7.  一个 64 位内存变量，包含另一个内存对象的地址；你可以使用 `.dword` 指令为指针分配存储空间（或使用其他机制保留 64 位空间）。

8.  将指针加载到一个 64 位寄存器中，并使用寄存器间接寻址模式访问内存。

9.  使用 `.dword` 指令。

10.  使用未初始化的指针；使用包含非法值的指针；继续使用已释放的已分配数据（*悬空指针*）；使用完内存后没有释放它（*内存泄漏*）；使用错误的数据类型访问间接数据

11.  指向已分配内存的指针，但该内存已经被释放

12.  内存泄漏发生在程序分配内存（使用 `malloc()`）但在使用完毕后没有释放该存储空间时。

13.  一个由其他数据类型集合构成的对象

14.  一个由零值（通常是字节）分隔的字符序列

15.  一个以长度值开始的字符序列（通常是字节，但也可以是半字、字或其他类型）

16.  描述字符串对象的结构，通常包含长度信息和指向字符串的指针

17.  在连续内存位置中出现的相同类型的对象序列

18.  第一个元素的地址，通常是数组在内存中最低地址的位置

19.  这是一个使用 Gas 的典型数组声明：

```
anArray .space 256, 0  // 256 bytes, all initialize to 0
```

20.  你通常会使用像 `.word` 这样的指令，并列出初始元素值；这是一个示例：

```
initializedArray: .word 1, 2, 3, 4, 5, 6, 7, 8
```

如果你有一个字节数组，并且每个字节都用相同的值初始化，你也可以使用 `.space` 指令。

21.  (a) 将索引乘以 8 并将数组 A 的基地址加到这个乘积中；(b) 要访问 `W[i, j]`，使用地址 = base(W) + (i * 8 + j) * 4；(c) 要访问 `R[i, j, k]`，使用地址 = base(R) + ((i * 4) + j) * 6 + k) * 4。

22.  存储数组的内存机制，其中每行的元素出现在连续的内存位置中，行本身则出现在连续的内存块中

23.  在内存中存储数组的机制，其中每一列的元素出现在连续的内存位置中，而列出现在连续的内存块中

24.  一个典型的二维数组声明，例如字数组 W[4,8]，形式如下：W: .space 4 * 8 * 4, 0。

25.  一种复合数据类型，其元素（字段）不必都具有相同类型

26.  使用如下语句：

```
struct student
    byte  sName, 65 // Includes zero-terminating byte
    hword Major
    byte  SSN, 12   // Includes zero-terminating byte
    hword Midterm1
    hword Midterm2
    hword Final
    hword Homework
    hword Projects
ends student
```

27.  将特定字段的偏移量加到结构的基地址上。

28.  一种结构，其中所有字段占用相同的内存位置

29.  对于结构体，每个字段分配一个独立的内存块（根据其大小），而对于共用体，所有字段分配相同的内存位置。

## F.5 第五章

1.  bl 指令将下一条指令的地址复制到 LR 寄存器中，然后将控制转移到操作数指定的目标地址。

2.  ret 指令将 LR 中的值复制到程序计数器。

3.  调用者保存的最大问题是难以维护。它还会生成更大的目标代码文件。

4.  它保存寄存器，占用了宝贵的 CPU 周期，即使调用者不要求保存这些寄存器。

5.  栈中的存储空间，过程在其中维护参数、返回地址、保存的寄存器值、本地变量以及可能的其他数据

6.  FP 寄存器（X29）

7.  标准入口序列如下：

```
stp fp, lr, [sp, #-16]!   // Save LR and FP values.
mov fp, sp                // Get activation record ptr in FP.
sub sp, sp, #NumVars      // Allocate local storage.
```

8.  标准退出序列如下所示：

```
mov sp, fp    // Deallocate storage for all the local vars.
ldp fp, lr, [sp], #16  // Pop FP and return address.
ret                    // Return to caller.
```

9.  过程在激活记录中自动分配和释放存储的变量

10.  进入过程时

11.  参数的值

12.  参数的地址

13.  X0, X1, X2 和 X3

14.  超过第八个参数的所有参数通过栈传递。

15.  ARM 过程可以使用易失性寄存器而不保存其值；非易失性寄存器的值必须在过程调用间保持。

16.  寄存器 X0, X1, ..., X15

17.  寄存器 X16 到 X31（SP）

18.  过程通过 LR 寄存器中传递的地址访问传递的参数。

19.  大型参数（如数组和记录）应通过引用传递，因为使用引用参数时，过程运行更快且更简短。

20.  X0 寄存器（X8 可以包含指向大型函数返回结果的指针）

21.  传递给过程或函数的调用程序地址，作为参数传递

22.  使用 br 指令调用过程参数（以及通过指针调用任何过程）。

23.  为寄存器预留本地存储空间，并在本地存储中保存这些值。  ## F.6 第六章

1.  cmp 指令会在两个操作数相等时设置零标志。

2.  cmp 指令会在一个无符号操作数（左边）大于或等于另一个无符号操作数（右边）时设置进位标志。

3.  cmp 指令会在左操作数小于右操作数时将负标志和溢出标志设置为相反的值；当左操作数大于或等于右操作数时，它们将被设置为相同的值。

4.  x = x + y:

```
ldr w0, [fp, #x]
ldr w1, [fp, #y]
add w0, w0, w1
str w0, [fp, #x]
```

x = y - z:

```
ldr w0, [fp, #y]
ldr w1, [fp, #z]
sub w0, w0, w1
str w0, [fp, #x]
```

x = y * z:

```
ldr w0, [fp, #y]
ldr w1, [fp, #z]
mul w0, w0, w1
str w0, [fp, #x]
```

x = y + z * t:

```
ldr w0, [fp, #y]
ldr w1, [fp, #z]
ldr w2, [fp, #t]
mul w1, w1, w2
sub w0, w0, w1
str w0, [fp, #x]
```

x = (y + z) * t:

```
ldr w0, [fp, #y]
ldr w1, [fp, #z]
add w0, w0, w1
ldr w1, [fp, #t]
mul w0, w0, w1
str w0, [fp, #x]
```

x = -((x * y) / z):

```
ldr  w0, [fp, #x]
ldr  w1, [fp, #y]
mul  w0, w0, w1
ldr  w1, [fp, #z]
sdiv w0, w0, w1
neg  w0, w0
str  w0, fp, #x]
```

x = (y == z) && (t != 0):

```
ldr  w0, [fp, #y]
ldr  w1, [fp, #z]
cmp  w0, w1
cset w0, eq
ldr  w1, [fp, #t]
cmp  w1, #0
cset w1, ne
and  w0, w0, w1
str  w0, [fp, #w]
```

5.  x = x * 2:

```
ldr w0, [fp, #x]
lsl w0, w0, #1
str w0, [fp, #x]
```

x = y * 5:

```
ldr w0, [fp, #y]
lsl w1, w0, #2
add w0, w0, w1
str w0, [fp, #x]
```

x = y * 8:

```
ldr w0, [fp, #y]
lsl w0, w0, #3
str w0, [fp, #x]
```

6.  x = x / 2:

```
ldr x0, [fp, #x]
lsr x0, #1
str x0, [fp, #x]
```

x = y / 8:

```
ldr x0, [fp, #y]
lsr x0, #3
str x0, [fp, #x]
```

x = z / 10:

```
ldr x0, [fp, #z]
ldr x1, =6554    // 65,536/10
mul x0, x0, x1
lsr x0, x0, #16  // Divide by 65,535.
str x0, [fp, #x]
```

7.  x = x + y:

```
ldr  d0, [fp, #x]
ldr  d1, [fp, #y]
fadd d0, d0, d1
str  d0, [fp, #x]
```

x = y - z:

```
ldr  d0, [fp, #y]
ldr  d1, [fp, #z]
fsub d0, d0, d1
str  d0, [fp, #x]
```

x = y * z:

```
ldr  d0, [fp, #y]
ldr  d1, [fp, #z]
fmul d0, d0, d1
str  d0, [fp, #x]
```

x = y + z * t:

```
ldr  d0, [fp, #y]
ldr  d1, [fp, #z]
ldr  d2, [fp, #t]
fmul d1, d1, d2
fadd d0, d0, d1
str  d0, [fp, #x]
```

x = (y + z) * t:

```
ldr  d0, [fp, #y]
ldr  d1, [fp, #z]
fadd d0, d0, d1
ldr  d1, [fp, #t]
fmul d0, d0, d1
str  d0, [fp, #x]
```

x = -((x * y) / z):

```
ldr  d0, [fp, #x]
ldr. d1, [fp, #y]
fmul d0, d0, d1
ldr  d1, [fp, #z]
div  d0, d0, d1
fneg d0, d0
str  d0, [fp, #x]
```

8.  bb = x < y:

```
ldr  d0, [fp, #x]
ldr  d1, [fp, #y]
fcmp d0, d1
cset x0, lo  // Less than, ordered
strb w0, [fp, #bb]
```

bb = x >= y && x < z:

```
ldr  d0, [fp, #x]
ldr  d1, [fp, #y]
fcmp d0, d1
cset x0, ge  // Greater than or equal, ordered (HS is unordered)
ldr  d1, [fp, #z]
fcmp d0, d1
cset x1, lo // Less than, ordered (LT is unordered)
and  x0, x1
strb w0, [fp, #bb]
```

## F.7 第七章

1.  使用 lea 宏来获取程序中符号的地址。

2.  br reg64

3.  一段通过进入和离开某些状态来跟踪执行历史的代码

4.  扩展分支指令范围的机制

5.  短路布尔运算可能不会执行表达式中的所有条件代码，如果它确定结果为真或假而不执行任何额外代码。完全的布尔运算会评估整个表达式，即使在部分评估之后已知结果。

6.

a.

```
ldr  w0, [fp, #x]
ldr  w1, [fp, #y]
cmp  w0, w1
cset w0, eq
ldr  w1, [fp, #z]
ldr  w2, [fp, #y]
cmp  w0, w1
cset w1, hi
orrs w0, w1
beq  skip

     Do something.
skip:
```

b.

```
ldr  w0, [fp, #x]
ldr  w1, [fp, #y]
cmp  w0, w1
cset w0, ne
ldr  w1, [fp, #z]
ldr  w2, [fp, #t]
cmp  w1, w2
cset w1, lo
ands w0, w1
beq  doElse

  `  then statements`
b.al ifDone

doElse:
    `else statements`
ifDone:
```

7.

a.

```
ldrsh w0, [fp, #x]
ldrsh w1, [fp, #y]
cmp   w0, w1
bne   skip
ldrsh w1, [fp, #z]
ldrsh w2, [fp, #t]
bge   skip

   Do something.
skip:
```

b.

```
ldrsh w0, [fp, #x]
ldrsh w1, [fp, #y]
cmp   w0, w1
beq   doElse
ldrsh w1, [fp, #z]
ldrsh w2, [fp, #t]
cmp   w1, w2
bge   doElse

 then statements
b.al  ifDone

doElse:
    `else statements`
ifDone:
```

8.  以下 switch 语句（假设所有变量都是无符号 32 位整数）将转换为汇编语言代码：

a.

```
ldr  x0, [fp, #t]
cmp  x0, #3
bhi  default
adr  x1, jmpTbl
ldr  x0, [x1, x0, lsl #3]
add  x0, x0, x1
br   x0

jmpTbl: .dword case0-jmpTbl, case1-jmpTbl, case2-jmpTbl, case3-jmpTbl
```

b.

```
ldr  x0, [fp, #t]
cmp  x0, #2
blo  default
cmp  x0, #6
bhi  default
adr  x1, jmpTbl
ldr  x0, [x1, x0, lsl #3]
add  x0, x0, x1
br   x0

jmpTbl: .dword case2-jmpTbl, default-jmpTbl, case4-jmpTbl
        .dword case5-jmpTbl, case6-jmpTbl
```

9.  以下 while 循环将转换为相应的汇编代码（假设所有变量为有符号 32 位整数）：

a.

```
whlLp:
    ldr x0, [fp, #i]
    ldr x1, [fp, #j]
    cmp x0, x1
    bgt endWhl

    Code for loop body

    b.al whlLp
endWhl:
```

b.

```
do...while:

rptLp:
    `Code for loop body`

    ldr x0, [fp, #i]
    ldr x1, [fp, #j]
    cmp x0, x1
    bne rptLp
```

c.

```
 str  wzr [fp, #i]
forLp:
   ldr  x0, [fp, #i]
   cmp  x0, #10
   bge  forDone

   Code for loop body

   ldr  x0, [fp, #i]
   add  x0, x0, #1
   str  x0, [fp, #i]
   b.al forLp
forDone:
```

## F.8 第八章

1.

a.

```
 ldp  x0, x1, [fp, #y]
 ldp  x2, x3, [fp, #z]
 adds x0, x0, x2
 adc  x1, x1, x3
 stp  x0, x1, [fp, #x]
```

b.

```
 ldr  x0, [fp, #y]
 ldr  w1, [fp, #y+8]
 ldr  x2, [fp, #z]
 adds x0, x0, x2
 adc  w1, w1, wzr
 str  x0, [fp, #x]
 str  w1, [fp, #x+8]
```

c.

```
 ldr  w0, [fp, #y]
 ldrh w1, [fp, #y+4]
 ldr  w2, [fp, #z]
 ldrh w3, [fp, #z+4]
 adds w0, w0, w2
 adc  w1, w1, w3
 str  w0, [fp, #x]
 strh w1, [fp, #x+4]
```

2.

a.

```
 ldp  x0, x1, [fp, #y]
 ldr  x2, [fp, #y+16]
 ldp  x3, x4, [fp, #z]
 ldr  x5, [fp, #z+16]
 subs x0, x0, x3
 sbc  x1, x1, x4
 sbc  x2, x2, x5
 stp  x0, x1, [fp, #x]
 str  x2, [fp, #x+16]
```

b.

```
 ldr  x0, [fp, #y]
 ldr  w1, [fp, #y+8]
 ldp  x2, [fp, #z]
 ldr  w3, [fp, #z+8]
 subs x0, x0, x2
 sbc  w1, w1, w3
 str  x0, [fp, #x]
 str  w1, [fp, #x+8]
```

3.

```
 ldr     x0, [fp, #y]
            ldr     x1, [fp, #y + 8]
            ldr     x2, [fp, #z]
            ldr     x3, [fp, #z + 8]
// X5:X4 = X0 * X2

            mul     x4, x0, x2
            umulh   x5, x0, x2

// X6:X7 = X1 * X2, then X5 = X5 + X7 (and save carry for later):

            mul     x7, x1, x2
            umulh   x6, x1, x2
            adds    x5, x5, x7

// X7 = X0 * X3, then X5 = X5 + X7 + C (from earlier):

 mul     x7, x0, x3
            adcs    x5, x5, x7
            umulh   x7, x0, x3
            adcs    x6, x6, x7  // Add in carry from adcs earlier.

// X7:X2 = X3 * X1
            mul     x2, x3, x1
            umulh   x7, x3, x1

            adc     x7, x7, xzr  // Add in C from previous adcs.
            adds    x6, x6, x2   // X6 = X6 + X2
            adc     x7, x7, xzr  // Add in carry from adds.

// X7:X6:X5:X4 contains 256-bit result at this point, ignore overflow:
            stp     x4, x5, [fp, #x]   // Save result to location.
```

4.  转换如下：

a.

```
 ldp x0, x1, [fp, #x]
 ldp x2, x3, [fp, #y]
 cmp x0, x2
 bne isFalse
 cmp x1, x3
 bne isFalse

 Code

isFalse:
```

b.

```
 ldp x0, x1, [fp, #x]
 ldp x2, x3, [fp, #y]
 cmp x1, x3
 bhi isFalse
 blo isTrue
 cmp x1, x3
 bhs isFalse

isTrue:
 `Code`

isFalse:
```

c.

```
 ldp x0, x1, [fp, #x]
 ldp x2, x3, [fp, #y]
 cmp x1, x3
 blo isFalse
 bhi isTrue
 cmp x1, x3
 bls isFalse

isTrue:
 `Code`

isFalse:
```

d.

```
 ldp x0, x1, [fp, #x]
 ldp x2, x3, [fp, #y]
 cmp x1, x3
 bne isTrue
 cmp x1, x3
 beq isFalse

isTrue:
 `Code`

isFalse:
```

5.  转换如下：

a.

```
 ldp  x0, x1, [fp, #x]
 subs x0, xzr, x0
 sbc  x1, xzr, x1
 stp  x0, x1, [fp, #x]
```

b.

```
 ldp  x0, x1, [fp, #y]
 subs x0, xzr, x0
 sbc  x1, xzr, x1
 stp  x0, x1, [fp, #x]
```

6.  转换如下：

a.

```
 ldp x0, x1, [fp, #y]
 ldp x2, x3, [fp, #z]
 and x0, x0, x2
 and x1, x1, x3
 stp x0, x1, [fp, #x]
```

b.

```
 ldp x0, x1, [fp, #y]
 ldp x2, x3, [fp, #z]
 orr x0, x0, x2
 orr x1, x1, x3
 stp x0, x1, [fp, #x]
```

c.

```
 ldp x0, x1, [fp, #y]
 ldp x2, x3, [fp, #z]
 eor x0, x0, x2
 eor x1, x1, x3
 stp x0, x1, [fp, #x]
```

d.

```
 ldp x0, x1, [fp, #y]
 not x0, x0
 not x1, x1
 stp x0, x1, [fp, #x]
```

e.

```
 ldp  x0, x1, [fp, #y] // The easy way
 adds x0, x0, x0
 adc  x1, x1, x1
 stp  x0, x1, [fp, #x]
```

f.

```
 ldp x0, x1, [fp, #y]  // The easy way
 ror x2, x1, #1
 and x2, x2, #1 << 63
 lsr x0, x0, #1
 orr x0, x0, x2
 lsr x1, x1, #1
 stp x0, x1, [fp, #x]
```

7.

```
 ldp x0, x1, [fp, #y]  // The easy way
 ror x2, x1, #1
 and x2, x2, #1 << 63
 lsr x0, x0, #1
 orr x0, x0, x2
 asr x1, x1, #1
 stp x0, x1, [fp, #x]
```

8.

```
 ldp  x0, x1, [fp, #x]  // The easy way
 adcs x0, x0, x0
 adcs x1, x1, x1
 stp  x0, x1, [fp, #x]
```

## F.9 第九章

1.  四位输出数字

2.  调用 qToStr 两次，第一次传入高字节（HO），第二次传入低字节（LO）。

3.  获取输入值并检查是否为负数。如果是，输出一个减号(-)字符并将值取反。无论数值是负数还是非负数，调用无符号转换函数处理其余部分。

4.  u64toSizeStr 函数期望 X0 中传递指向目标缓冲区的指针，X1 中传递要转换为字符串的值，X3 中传递最小字段宽度。

5.  该函数将输出足够的字符以正确表示值。

6.  r64ToStr 函数期望 D0 中传递要转换的浮点值，X0 中传递缓冲区指针，X1 中传递字段宽度，X2 中传递小数点后的位数，X3 的低字节中传递填充字符，X4 中传递最大字符串长度。

7.  一个包含#字符的 fWidth 长度的字符串，如果无法正确格式化输出

8.  D0 包含要转换的值；X0 包含输出缓冲区的地址；X1 包含字段宽度；X2 为填充字符；X3 包含指数数字的个数；X4 为最大字符串宽度。

9.  一个用于开始、结束和分隔输入值的字符

10.  溢出和非法输入字符

## F.10 第十章

1.  合法输入值的集合

2.  可能输出值的集合

3.

a.

```
 // Assume "input" passed in X0.
 lea  x1, f        // Lookup table
 ldrb w0, [x1, x0] // Function result is left in W0.
```

b.

```
 // Assume "input" passed in X0.
 lea  x1, f        // Lookup table
 ldrh w0, [x1, x0, uxtw #1] // Function result is left in W0.
```

c.

```
// Assume "input" passed in X0.
 lea  x1, f        // Lookup table
 ldrb w0, [x1, x0] // Function result is left in W0.
```

d.

```
 // Assume "input" passed in X0.
 lea  x1, f        // Lookup table
 ldr w0, [x1, x0, uxtw #2] // Function result is left in W0.
```

4.  调整函数输入值的过程，使最小值和最大值受到限制，以便能够使用更小的表格

5.  因为内存访问相较于计算性能非常缓慢

## F.11 第十一章

1.  通道是向量寄存器中一个字节、半字、字或双字数组的元素。当对一对向量寄存器进行操作时，通道是两个向量中的对应元素。

2.  标量指令对单个数据项进行操作，而向量指令对向量寄存器中的多个数据项（通道）进行操作。

3.  fmov Sd, Ws 指令

4.  fmov Dd, Xs 指令

5.  tbl 或 tbx 指令

6.  mov Vd.t[index], Rs 指令（Rn = Xn 或 Wn）

7.  shl Vd.2D, Vs.2D, #n 指令

8.  垂直加法将两个向量寄存器中的对应通道相加，而水平加法将一个向量寄存器中的相邻通道相加。

9.  使用 movi v0.16B, #0 指令。

10.  使用 movi v0.16B, #0xff 指令。

## F.12 第十二章

1.  and 和 bic 指令

2.  bic 指令

3.  orr 指令

4.  eor 指令

5.  tst 指令

6.  bfxil（或 bfm）指令

7.  bfi（或 bfm）指令

8.  clz 指令

9.  你可以反转寄存器中的位，反转所有位，然后使用 clz 指令找到第一个非零位。

10.  cnt 指令

## F.13 第十三章

1.  编译时语言

2.  在汇编过程中（编译时）

3.  #warning

4..warning

5.  #error

6..error

7.  #define

8..equ、.set 和 =

9.  #ifdef, #ifndef, #if, #elif, #else 和 #endif

10.  主要的 Gas 条件汇编指令是 .if、.elseif、.else 和 .endif。次要的汇编指令有 .ifdef、.ifb、.ifc、.ifeq、.ifeqs、.ifge、.ifgt、.ifile、.iflt、.ifnb、.ifnc、.ifndef/.ifnotdef、.ifne、.ifnc 和 .ifnes。

11.  CPP 映射宏

12..rept、.irp、.irpc 和 .endr

13..irpc

14.  #define

15..宏和 .endm

16.  在文件中指令助记符应出现的地方指定宏名称。

17.  使用函数符号。例如：mymacro(p1, p2)。

18.  在指令操作数字段中指定 Gas 宏参数作为操作数。例如：lea x0, label（x0 和 label 是 lea 宏的参数）。

19.  在宏声明中将 :req 放在参数后面。

20.  在宏声明中指定参数名称，不带后缀（:req、:varargs 或 =expression）。默认情况下，Gas 宏参数是可选的。

21.  在 #define 宏定义中，将 ... 用作最后（或唯一）一个参数。

22.  在 Gas 宏定义中，将 :varargs 放在最后（或唯一）一个参数后面。

23.  使用 .ifb (if blank) 条件汇编指令。

24..exitm

## F.14 第十四章

1.  内存中以一个包含 0 的字节结尾的零个或多个字符序列

2.  因为程序通常必须扫描整个字符串以确定其长度

3.  因为这种字符串汇编语言类型（a）将字符串的长度作为数据类型的一部分进行编码，（b）将字符串数据对齐到 16 字节边界，并且（c）保证字符串的存储空间是 16 字节的倍数。这允许算法获取字符串末尾之外的额外数据，只要所有数据都适合在对齐到 16 字节边界的 16 字节块内。

4.  因为起始索引参数可以是任何值

5.  因为它们必须处理可变长度的字符

## F.15 第十五章

1.  #ifndef 或 .ifndef

2.  一个源文件的汇编加上它直接或间接包含的任何文件

3..global

4..extern。严格来说，使用这个指令是可选的，因为 Gas 假设所有未定义的符号都是外部符号。

5.

```
`target: dependencies`
    commands
```

6.  一个依赖于 makefile 的文件是必须构建或更新的文件，以便正确构建当前文件（也就是说，当前文件依赖于该 makefile 依赖的文件才能被构建）。

7.  删除通过 make 操作生成的所有可执行文件和目标代码文件。

8.  链接器可以使用的对象模块集合，以便仅提取它所需要的对象模块

## F.16 第十六章

1.  操作系统通常使用 svc 调用操作系统的 API 函数。

2.  #0

3.  #0x80
