# 第二十一章：E

问题的答案

## E.1 第一章问题的答案

1.  *cmd.exe*

1.  *ml64.exe*

1.  地址、数据和控制

1.  AL、AH、AX 和 EAX

1.  BL、BH、BX 和 EBX

1.  SIL、SI 和 ESI

1.  R8B、R8W 和 R8D

1.  FLAGS、EFLAGS 或 RFLAGS

1.  (a) 2, (b) 4, (c) 16, (d) 32, (e) 8

1.  任何 8 位寄存器和任何可以用 8 位表示的常量

1.  32

1.  | **目的地** | **常量大小** |
1.  | --- | --- |
    | RAX | 32 |
    | EAX | 32 |
    | AX | 16 |
    | AL | 8 |
    | AH | 8 |
    | mem[32] | 32 |
    | mem[64] | 32 |

1.  64

1.  任何内存操作数都可以工作，无论其大小如何。

1.  `call`

1.  `ret`

1.  应用程序二进制接口

1.  (a) AL, (b) AX, (c) EAX, (d) RAX, (e) XMM0, (f) RAX

1.  RCX 用于整数操作数，XMM0 用于浮点/向量操作数

1.  RDX 用于整数操作数，XMM1 用于浮点/向量操作数

1.  R8 用于整数操作数，XMM2 用于浮点/向量操作数

1.  R9 用于整数操作数，XMM3 用于浮点/向量操作数

1.  `dword` 或 `sdword`

1.  `qword`

## E.2 第二章问题的答案

1.  9 × 10³ + 3 × 10² + 8 × 10¹ + 4 × 10⁰ + 5 × 10^(-1) + 7 × 10^(-2) + 6 × 10^(-3)

1.  (a) 10, (b) 12, (c) 7, (d) 9, (e) 3, (f) 15

1.  (a) A, (b) E, (c) B, (d) D, (e) 2, (f) C, (g) CF, (h) 98D1

1.  (a) 0001_0010_1010_1111, (b) 1001_1011_1110_0111, (c) 0100_1010, (d) 0001_0011_0111_1111, (e) 1111_0000_0000_1101, (f) 1011_1110_1010_1101, (g) 0100_1001_0011_1000

1.  (a) 10, (b) 11, (c) 15, (d) 13, (e) 14, (f) 12

1.  (a) 16, (b) 64, (c) 128, (d) 32, (e) 4, (f) 8, (g) 4

1.  (a) 2, (b) 4, (c) 8, (d) 16

1.  (a) 16, (b) 256, (c) 65,636, (d) 2

1.  4

1.  0 到 7

1.  位 0

1.  位 31

1.  (a) 0, (b) 0, (c) 0, (d) 1

1.  (a) 0, (b) 1, (c) 1, (d) 1

1.  (a) 0, (b) 1, (c) 1, (d) 0

1.  1

1.  AND

1.  OR

1.  NOT

1.  XOR

1.  `not`

1.  1111_1011

1.  0000_0010

1.  (a) 和 (c) 和 (e)

1.  `neg`

1.  (a) 和 (c) 和 (d)

1.  `jmp`

1.  *label:*

1.  进位、溢出、零、符号

1.  JZ

1.  JC

1.  JA, JAE, JBE, JB, JE, JNE（以及同义词 JNA、JNAE、JNB、JNBE，另有其他同义词）

1.  JG、JGE、JL、JLE、JE、JNE（以及同义词 JNG、JNGE、JNL 和 JNLE）

1.  如果移位的结果为 0，则 ZF = 1。

1.  从操作数中移出的 HO 位进入进位标志。

1.  如果下一个 HO 位与移位前的 HO 位不同，OF 会被设置；否则，它会被清除，但仅适用于 1 位移位。

1.  SF 被设置为结果的 HO 位。

1.  如果移位的结果为 0，则 ZF = 1。

1.  从操作数中移出的 LO 位进入进位标志。

1.  如果下一个 HO 位与移位前的 HO 位不同，OF 会被设置；否则，它会被清除，但仅适用于 1 位移位。

1.  在 SHR 指令之后，SF 始终被清除，因为一个 0 总是被移入结果的 HO 位。

1.  如果移位的结果为 0，则 ZF = 1。

1.  从操作数中移出的 LO 位进入进位标志。

1.  SAR 指令之后，OF 总是清除，因为符号不可能发生变化。

1.  SF 被设置为结果的 HO 位，尽管从技术上讲它永远不会改变。

1.  从操作数中移出的 HO 位进入进位标志。

1.  它不会影响 ZF。

1.  从操作数中移出的 LO 位进入进位标志。

1.  它不会影响符号标志。

1.  乘以 2

1.  除以 2

1.  乘法和除法

1.  将它们相减并检查其差值是否小于一个小的误差值。

1.  在 HO 尾数位置上有 1 位的值

1.  7

1.  30h 到 39h

1.  撇号和引号

1.  UTF-8、UTF-16 和 UTF-32

1.  一个标量整数值，表示一个 Unicode 字符

1.  一块 65,536 个不同的 Unicode 字符

## E.3 第三章问题的答案

1.  RIP

1.  操作码，机器指令的数字编码

1.  静态和标量变量

1.  ±2GB

1.  要访问的内存位置的地址

1.  RAX

1.  `lea`

1.  在完成所有寻址模式计算后获得的最终地址

1.  1、2、4 或 8

1.  2GB 总内存

1.  你可以使用 VAR[REG]寻址模式直接访问数组的元素，使用 64 位寄存器作为数组的索引，而无需首先将数组的地址加载到单独的基寄存器中。

1.  `.data`部分可以保存已初始化的数据值；`.data?`部分只能包含未初始化的变量。

1.  `.code` 和 `.const`

1.  `.data`和`.data?`

1.  指向特定部分的偏移量（例如，`.data`）

1.  使用`some_ID` `label` `some_type`来告知 MASM 以下数据的类型是`some_type`，尽管实际上它可能是另一种类型。

1.  MASM 将它们合并为一个单独的部分。

1.  使用`align 8`语句。

1.  内存管理单元

1.  如果`b`位于 MMU 页的最后一个字节处且下一个页面不可读，从以`b`开头的内存位置加载一个字会产生一般保护错误。

1.  一个常量表达式加上变量在内存中的基地址

1.  将以下操作数类型强制转换为另一种类型

1.  小端值在内存中以其 LO 字节位于最低地址，HO 字节位于最高地址的形式出现。大端值则相反：它们的 HO 字节出现在最低地址，LO 字节出现在内存中的最高地址。

1.  `xchg al, ah` 或 `xchg ah, al`

1.  `bswap eax`

1.  `bswap rax`

1.  (a) 从 RSP 中减去 8，(b) 将 RAX 中的值存储到 RSP 指向的位置。

1.  (a) 从 RSP 指向的 8 个字节中加载 RAX，(b) 将 8 加到 RSP。

1.  反转

1.  后进先出

1.  使用`[RSP ± const]`寻址模式将数据从栈中移动进出。

1.  Windows ABI 要求栈在 16 字节边界上对齐；推送 RAX 可能会使栈在 8 字节（而不是 16 字节）边界上对齐。

## E.4 第四章问题的答案

1.  `imul` `reg``,` `constant`

1.  `imul` `destreg``,` `srcreg``,` `constant`

1.  `imul` `destreg``,` `srcreg`

1.  一个符号（命名）常量，MASM 将在源文件中每次出现该名称时替换为文字常量。

1.  `=`, `equ`, `textequ`

1.  文本等式替换为可以是任何文本的字符串；数值等式必须分配一个可以用 64 位整数表示的数值常量。

1.  使用文本分隔符`<`和`>`包围字符串字面量；例如，`<"a long string">`。

1.  MASM 在汇编过程中可以计算的算术表达式

1.  `lengthof`。

1.  当前段的偏移量。

1.  `this` 和 `$`。

1.  使用常量表达式 `$-startingLocation`。

1.  使用一系列（数字）等式，每个连续的等式的值设置为前一个等式的值加一；例如：

    ```
    val1 = 0
    val2 = val1 + 1
    val3 = val2 + 1
    etc.
    ```

1.  使用 `typedef` 指令。

1.  指针是内存中的一个变量，它保存另一个内存对象的地址。

1.  将指针变量加载到一个 64 位寄存器中，并使用寄存器间接寻址模式来引用该地址。

1.  使用 `qword` 数据声明，或其他 64 位大小的数据类型。

1.  `offset` 操作符。

1.  (a) 未初始化的指针，(b) 使用指针保存非法值，(c) 在存储已被释放后使用指针（悬空指针），(d) 在不再使用存储后未释放存储（内存泄漏），(e) 使用错误的数据类型访问间接数据。

1.  在存储已被释放后使用指针。

1.  未能在使用完存储后释放它。

1.  由较小的数据对象组成的聚合类型。

1.  一个以 0 字节（或其他 0 值）结尾的字符序列。

1.  一个包含长度值作为第一个元素的字符串。

1.  描述符是一种数据类型，包含一个指向字符数据的指针、字符串长度以及可能描述字符串数据的其他信息。

1.  一种同质的数据元素集合（所有元素类型相同）。

1.  数组第一个元素的内存地址。

1.  `array byte 10 dup (?)`（作为示例）。

1.  只需将初始值填写为字节、字、双字或其他数据声明指令的操作数字段。此外，你还可以使用一个或多个常量值作为`dup`操作符的操作数；例如，`5 dup (2, 3)`。

1.  (a) `base_address` `+` `index` `* 4`（4 是元素大小），(b) `W[i,j] =` `base_address` `+ (i * 8 + j) * 2`（2 是元素大小），(c) `R[i,j,k] =` `base_address` `+(((i * 4) + j) * 6 + k) * 8`（8 是元素大小）。

1.  一种多维数组的组织方式，在这种方式中，你将每一行的元素存储在连续的内存位置中，然后将每一行按顺序存储在内存中。

1.  一种多维数组的组织方式，在这种方式中，你将每一列的元素存储在连续的内存位置中，然后将每一列按顺序存储在内存中。

1.  `W word 4 dup (8 dup (?))`

1.  一种异质的数据元素集合（每个字段可能有不同的类型）。

1.  `struct` 和 `ends`。

1.  点操作符。

1.  一种异质的数据元素集合（每个字段可能有不同的类型）；联合体中每个字段的偏移量从 0 开始。

1.  `union` 和 `ends`。

1.  记录和结构体的字段在结构体内按顺序出现在连续的内存位置（每个字段都有自己的字节块）；而联合体的字段彼此重叠，每个字段都从联合体中的偏移量零开始。

1.  一个未命名的联合体，它的字段被视为外部结构体的字段。

## E.5 第五章问题的答案。

1.  它将返回地址推送到栈上（调用后下一条指令的地址），然后跳转到操作数指定的地址。

1.  它从栈中弹出一个返回地址，并将地址移动到 RIP 寄存器，将控制转移到调用当前过程后面的指令。

1.  弹出返回地址后，CPU 将此值加到 RSP 中，从栈中移除相应字节的参数。

1.  紧接着调用过程指令的地址

1.  命名空间污染发生在源文件中定义了太多符号、标识符或名称，以至于在该源文件中很难选择新的、唯一的名称。

1.  在名称后加两个冒号；例如，`id::`。

1.  在过程之前使用`option noscoped`指令

1.  在进入过程时使用`push`指令将寄存器值保存在栈上；然后使用`pop`指令在从过程返回之前立即恢复寄存器值。

1.  代码难以维护。（其次的问题，虽然不大，是它占用更多空间。）

1.  性能——因为你经常保存一些调用代码不需要保存的寄存器

1.  当子程序尝试返回时，它会使用你在栈上留下的垃圾作为返回地址，这通常会产生未定义的结果（程序崩溃）。

1.  子程序使用调用前栈上任何存在的内容作为返回地址，结果是未定义的。

1.  一组与过程调用（激活）相关的数据，包括参数、局部变量、返回地址和其他项目

1.  RBP

1.  8 字节（64 位）

1.  ```
    push rbp
    mov  rbp, rsp
    sub  rsp, sizeOfLocals ; Assuming there are local variables
    ```

1.  ```
    leave
    ret
    ```

1.  `and rsp, -16`

1.  源文件中的一部分（通常是过程的主体），在程序中符号可见且可用

1.  从为变量分配存储空间开始，到系统释放该存储空间为止

1.  进入代码块（通常是过程）时自动分配存储的变量，并在退出该代码块时自动释放该存储

1.  进入过程时（或与自动变量关联的代码块）

1.  使用`textequ`指令或 MASM 本地指令

1.  `var1`: –2；`local2`: –8（MASM 将变量对齐到 dword 边界）；`dVar`: –9；`qArray`: –32（数组的基地址是最低的内存地址）；`rlocal`: –40（数组的基地址是最低的内存地址）；`ptrVar`: –48

1.  `option prologue:PrologueDef` 和 `option epilogue:EpilogueDef`。还应该提供 `option prologue:none` 和 `option epilogue:none` 来关闭此功能。

1.  在 MASM 生成过程代码之前，在所有本地指令之后

1.  每当出现`ret`指令的地方

1.  实际参数的值

1.  实际参数值的内存地址

1.  RCX, RDX, R8 和 R9（或这些寄存器的较小子组件）

1.  XMM0, XMM1, XMM2 或 XMM3

1.  在栈上，位于为寄存器传递的参数预留的阴影位置（32 字节）之上

1.  程序可以自由修改易失性寄存器，而无需保留其值；但必须在过程调用之间保留非易失性寄存器的值。

1.  RAX、RCX、RDX、R8、R9、R10、R11、XMM0、XMM1、XMM2、XMM3、XMM4、XMM5，以及所有 YMM 和 ZMM 寄存器的 HO 128 位

1.  RBX、RSI、RDI、RBP、RSP、R12、R13、R14、R15 和 XMM6–XMM15。并且，返回过程时方向标志必须被清除。

1.  使用来自 RBP 寄存器的正偏移量

1.  为调用者通过 RCX、RDX、R8 和 R9 寄存器传递的参数在栈上保留的存储空间

1.  32 字节

1.  32 字节

1.  32 字节

1.  `parm1`：RBP + 16；`parm2`：RBP + 24；`parm3`：RBP + 32；`parm4`：RBP + 40

1.  ```
    mov rax, parm4
    mov al, [rax]
    ```

1.  `lclVar1`：RBP – 1；`lclVar2`：RBP – 4（对齐到 2 字节边界）；`lclVar3`：RBP – 8；`lclVar4`：RBP – 16

1.  通过引用

1.  应用程序二进制接口

1.  在 RAX 寄存器中

1.  作为参数传递的过程的地址

1.  间接地。通常通过使用`call` `parm`指令，其中`parm`是过程参数，一个包含过程地址的 qword 变量。你也可以将参数值加载到一个 64 位寄存器中，通过该寄存器间接调用过程。

1.  分配本地存储空间以保存需要保留的寄存器值，并在过程入口时将寄存器数据移入该存储空间，然后在从过程返回前将数据移回寄存器。

## E.6 第六章问题的答案

1.  对于 8 位操作数使用 AL，16 位操作数使用 AX，32 位操作数使用 EAX，64 位操作数使用 RAX

1.  8 位`mul`操作：16 位；16 位`mul`操作：32 位；32 位`mul`操作：64 位；64 位`mul`操作：128 位。CPU 将乘积存放在 AX 中用于 8×8 的乘积，DX:AX 用于 16×16 的乘积，EDX:EAX 用于 32×32 的乘积，RDX:RAX 用于 64×64 的乘积。

1.  商存放在 AL、AX、EAX 或 RAX 中，余数存放在 AH、DX、EDX 或 RDX 中

1.  将 AX 符号扩展到 DX。

1.  将 EAX 零扩展到 EDX。

1.  除以 0 并产生一个无法适应累加器寄存器（AL、AX、EAX 或 RAX）的商

1.  通过设置进位标志和溢出标志

1.  它们会打乱标志；也就是说，它们会将标志置于未定义的状态。

1.  扩展精度的`imul`指令生成一个 2 × *n*位的结果，使用隐式操作数（AL、AX、EAX 和 RAX），并修改 AH、DX、EDX 和 RDX 寄存器。此外，扩展精度的`imul`指令不允许常量操作数，而通用的`imul`指令则允许。

1.  `cbw`、`cwd`、`cdq`、`cqo`

1.  它们会打乱所有标志，留下未定义的状态。

1.  如果两个操作数相等，则设置零标志。

1.  如果第一个操作数小于第二个操作数，则设置进位标志。

1.  如果第一个操作数小于第二个操作数，则符号标志和溢出标志不同；如果第一个操作数大于或等于第二个操作数，则它们相同。

1.  一个 8 位寄存器或内存位置

1.  如果条件为真，它们将操作数设置为 1；如果条件不为真，则设置为 false。

1.  `test` 指令与 `and` 指令相同，唯一的不同是它不将结果存储到目标（第一个）操作数，而只是设置标志。

1.  它们都以相同的方式设置条件码标志。

1.  将要测试的操作数作为第一个（目标）操作数，并提供一个包含单个 1 位的立即常数，该位位于要测试的位位置。测试指令执行后，零标志将包含所需位的状态。

1.  以下是一些可能的解决方案，并非唯一解：

    `x = x + y`

    ```
    mov eax, x
    add eax, y
    mov x, eax
    ```

    `x = y – z`

    ```
    mov eax, y
    sub eax, z
    mov x, eax
    ```

    `x = y * z`

    ```
    mov  eax, y
    imul eax, z
    mov  x, eax
    ```

    `x = y + z * t`

    ```
    mov  eax, z
    imul eax, t
    add  eax, y
    mov  x, eax
    ```

    `x = (y + z) * t`

    ```
    mov  eax, y
    add  eax, z
    imul eax, t
    mov  x, eax
    ```

    `x = -((x*y)/z)`

    ```
    mov  eax, x
    imul y          ; Note: Sign-extends into EDX
    idiv z
    mov  x, eax
    ```

    `x = (y == z) && (t != 0)`

    ```
    mov   eax, y
    cmp   eax, z
    sete  bl
    cmp   t, 0
    setne bh
    and   bl, bh
    movzx eax, bl   ; Because x is a 32-bit integer
    mov   x, eax
    ```

1.  以下是一些可能的解决方案，并非唯一解：

    `x = x * 2`

    ```
    shl   x, 1
    ```

    `x = y * 5`

    ```
    mov   eax, y
    lea   eax, [eax][eax*4]
    mov   x, eax
    ```

    这里是另一种解决方案：

    ```
    mov   eax, y
    mov   ebx, eax
    shl   eax, 2
    add   eax, ebx
    mov   x, eax
    ```

    `x = y * 8`

    ```
    mov   eax, y
    shl   eax, 3
    mov   x, eax
    ```

1.  `x = x /2`

    ```
    shr   x, 1
    ```

    `x = y / 8`

    ```
    mov   ax, y
    shr   ax, 3
    mov   x, ax
    ```

    `x = z / 10`

    ```
    movzx eax, z
    imul  eax, 6554  ; Or 6553
    shr   eax, 16
    mov   x, ax
    ```

1.  `x = x + y`

    ```
    fld   x
    fld   y
    faddp
    fstp  x
    ```

    `x = y – z`

    ```
    fld   y
    fld   z
    fsubp
    fstp  x
    ```

    `x = y * z`

    ```
    fld   y
    fld   z
    fmulp
    fstp  x
    ```

    `x = y + z * t`

    ```
    fld   y
    fld   z
    fld   t
    fmulp
    faddp
    fstp  x
    ```

    `x = (y + z) * t`

    ```
    fld   y
    fld   z
    faddp
    fld   t
    fmulp
    fstp  x
    ```

    `x = -((x * y)/z)`

    ```
    fld   x
    fld   y
    fmulp
    fld   z
    fdivp
    fchs
    fstp  x
    ```

1.  `x = x + y`

    ```
    movss xmm0, x
    addss xmm0, y
    movss x, xmm0
    ```

    `x = y – z`

    ```
    movss xmm0, y
    subss xmm0, z
    movss x, xmm0
    ```

    `x = y * z`

    ```
    movss xmm0, y
    mulss xmm0, z
    movss x, xmm0
    ```

    `x = y + z * t`

    ```
    movss xmm0, z
    mulss xmm0, t
    addss xmm0, y
    movss x, xmm0
    ```

1.  `b = x < y`

    ```
    fld    y
    fld    x
    fcomip st(0), st(1)
    setb   b
    fstp   st(0)
    ```

    `b = x >= y && x < z`

    ```
    fld    y
    fld    x
    fcomip st(0), st(1)
    setae  bl
    fstp   st(0)
    fld    z
    fld    x
    fcomip st(0), st(1)
    setb   bh
    fstp   st(0)
    and    bl, bh
    mov    b, bl
    ```

## E.7 第七章问题的答案

1.  使用 `lea` 指令或 `offset` 操作符。

1.  `option noscoped`

1.  `option scoped`

1.  `jmp` `reg`64 和 `jmp` `mem`64

1.  维护历史信息的代码段，无论是通过变量还是程序计数器

1.  如果跳转助记符的第二个字母是 *n*，则移除 *n*；否则，插入 *n* 作为第二个字符。

1.  `jpo` 和 `jpe`

1.  用于扩展跳转或调用指令范围的短代码序列，超出 ±2GB 范围

1.  `cmov``cc``reg``,` `src`，其中 `cc` 是条件后缀之一（紧随条件跳转之后），`reg` 是一个 16 位、32 位或 64 位寄存器，`src` 是与 `reg` 相同大小的源寄存器或内存位置。

1.  你可以通过使用条件跳转来有条件地执行一大组不同类型的指令，而无需控制转移的时间开销。

1.  目标必须是寄存器，且不允许使用 8 位寄存器。

1.  布尔表达式的完全求值会评估表达式的所有组成部分，即使从逻辑上看不需要这样做；短路求值在确定表达式必须为真或假时会立即停止。

1.  ```
    if(x == y || z > t)
    {
        `Do something` 
    }
        mov  eax, x
        cmp  eax, y
        sete bl
        mov  eax, z
        cmp  eax, t
        seta bh
        or   bl, bh
        jz   skipIF
         `Code for statements that "do something"`
    skipIF:

    if(x != y && z < t)
    {
         `THEN statements`
    }
    Else
    {
         `ELSE statements`
    }
        mov   eax, x
        cmp   eax, y
        setne bl
        mov   eax, z
        cmp   eax, t
        setb  bh
        and   bl, bh
        jz    doElse
        ` Code for THEN statements`
        jmp   endOfIF

    doElse:
        ` Code for ELSE statements`
    endOfIF:
    ```

1.  ```
    1st IF:
        mov  ax, x
        cmp  ax, y
        jeq  doBlock
        mov  eax, z
        cmp  eax, t
        jnl  skipIF
    doBlock:     `Code for statements that "do something"`
    skipIF:

    2nd IF:
        mov   eax, x
        cmp   eax, y
        je    doElse
        mov   eax, z
        cmp   eax, t
        jnl   doElse
        ` Code for THEN statements`
        jmp   endOfIF

    doElse:
        ` Code for ELSE statements`
    endOfIF:
    ```

1.  ```
    switch(s)
    {
       case 0:   `case 0 code`  break;
       case 1:   `case 1 code`  break;
       case 2:   `case 2 code`  break;
       case 3:   `case 3 code`  break;
    }

        mov eax, s ; Zero-extends!
        cmp eax, 3
        ja  skipSwitch
        lea rbx, jmpTbl
        jmp [rbx][rax * 8]
    jmpTbl qword case0, case1, case2, case3

    case0: `case 0 code`
           jmp skipSwitch

    case1: `case 1 code`
           jmp skipSwitch

    case2: `case 2 code`
           jmp skipSwitch

    case3: `case 3 code`
     skipSwitch:

    switch(t)
    {
       case 2:  `case 0 code` break;
       case 4:  `case 4 code` break;
       case 5:  `case 5 code` break;
       case 6:  `case 6 code` break;
       default: `default code`
    }
        mov eax, t ; Zero-extends!
        cmp eax, 2
        jb  swDefault
        cmp eax, 6
        ja  swDefault
        lea rbx, jmpTbl
        jmp [rbx][rax * 8 – 2 * 8]
    jmpTbl qword case2, swDefault, case4, case5, case6

    swDefault: `default code`
           jmp endSwitch

    case2: `case 2 code`
           jmp endSwitch

    case4: `case 4 code`
           jmp endSwitch

    case5: `case 5 code`
           jmp endSwitch

    case6: `case 6 code`

    endSwitch:

    switch(u)
    {
       case 10:  ` case 10 code ` break;
       case 11:  ` case 11 code ` break;
       case 12:  ` case 12 code ` break;
       case 25:  ` case 25 code ` break;
       case 26:  ` case 26 code ` break;
       case 27:  ` case 27 code ` break;
       default:  ` default code`
    } 
         lea rbx, jmpTbl1  ; Assume cases 10-12
         mov eax, u        ; Zero-extends!
         cmp eax, 10
         jb  swDefault
         cmp eax, 12
         jbe sw1
         cmp eax, 25
         jb  swDefault
         cmp eax, 27
     ja  swDefault
         lea rbx, jmpTbl2
         jmp [rbx][rax * 8 – 25 * 8]
    sw1: jmp [rbx][rax*8-2*8]
    jmpTbl1 qword case10, case11, case12
    jmpTbl2 qword case25, case26, case27

    swDefault: `default code`
           jmp endSwitch

    case10: `case 10 code`
           jmp endSwitch

    case11: `case 11 code`
           jmp endSwitch

    case12: `case 12 code`
           jmp endSwitch

    case25: `case 25 code`
           jmp endSwitch

    case26: `case 26 code`
           jmp endSwitch

    case27: `case 27 code`

    endSwitch:
    ```

1.  ```
    while(i < j)
    {
         `Code for loop body`
    }

    whlLp:
         mov eax, i
         cmp eax, j
         jnl endWhl
          `Code for loop body`
         jmp whlLp
    endWhl:

    while(i < j && k != 0)
    {
         `Code for loop body, part a`
        if(m == 5) continue;
         `Code for loop body, part b`
        if(n < 6) break;
         `Code for loop body, part c`
    }

    ; Assume short-circuit evaluation:
     whlLp:
         mov eax, i
         cmp eax, j
         jnl endWhl
         mov eax, k
         cmp eax, 0
         je  endWhl
         ` Code for loop body, part a`
         cmp m, 5
         je  whlLp
         ` Code for loop body, part b`
         cmp n, 6
         jl  endWhl
        `  Code for loop body, part c`
         jmp whlLp
    endWhl:

    do
    {
       `Code for loop body`
    } while(i != j);

    doLp:
       `Code for loop body`
         mov eax, i
         cmp eax, j
         jne doLp

    do
    {
       `Code for loop body, part a`
        if(m != 5) continue;
       `Code for loop body, part b`
        if(n == 6) break;
       `Code for loop body, part c`
    } while(i < j && k > j);

    doLp:
      ` Code for loop body, part a`
         cmp m, 5
         jne doCont
      ` Code for loop body, part b`
         cmp n, 6
         je  doExit
      ` Code for loop body, part c`
    doCont:     mov eax, i
         cmp eax, j
         jnl doExit
         mov eax, k
         cmp eax, j
         jg  doLp
    doExit:

    for(int i = 0; i < 10; ++i)
    {
       `Code for loop body`
    }

           mov i, 0
    forLp: cmp i, 10
           jnl forDone
           ` Code for loop body`
           inc i
           jmp forLp
    forDone:
    ```

## E.8 第八章问题的答案

1.  你可以通过以下方式计算 *x* = *y* + *z*：

    1.  ```
        mov rax, qword ptr y
        add rax, qword ptr z
        mov qword ptr x, rax
        mov rax, qword ptr y[8]
        adc rax, qword ptr z[8]
        mov qword ptr x[8], rax
        ```

    1.  ```
        mov rax, qword ptr y
        add rax, qword ptr z
        mov qword ptr x, rax
        mov eax, dword ptr z[8] 
        adc eax, qword ptr y[8]
        mov dword ptr x[8], eax
        ```

    1.  ```
        mov eax, dword ptr y
        add eax, dword ptr z
        mov dword ptr x, eax
        mov ax, word ptr z[4]
        adc ax, word ptr y[4]
        mov word ptr x[4], ax
        ```

1.  你可以通过以下方式计算 *x* = *y* – *z*：

    1.  ```
        mov rax, qword ptr y
        sub rax, qword ptr z
        mov qword ptr x, rax
        mov rax, qword ptr y[8]
        sbb rax, qword ptr z[8]
        mov qword ptr x[8], rax
        mov rax, qword ptr y[16]
        sbb rax, qword ptr z[16]
        mov qword ptr x[16], rax
        ```

    1.  ```
        mov rax, qword ptr y
        sub rax, qword ptr z
        mov qword ptr x, rax
        mov eax, dword ptr y[8]
        sbb eax, dword ptr z[8]
        mov dword ptr x[8], eax
        ```

1.  ```
    mov rax, qword ptr y
    mul qword ptr z
    mov qword ptr x, rax
    mov rbx, rdx

    mov rax, qword ptr y
    mul qword ptr z[8]
    add rax, rbx
    adc rdx, 0
    mov qword ptr x[8], rax
    mov rbx, rdx

    mov rax, qword ptr y[8]
    mul qword ptr z
    add x[8], rax
    adc rbx, rdx

    mov rax, qword ptr y[8]
    mul qword ptr z[8]
    add rax, rbx
    mov qword ptr x[16], rax
    adc rdx, 0
    mov qword ptr x[24], rdx
    ```

1.  ```
    mov  rax, qword ptr y[8]
    cqo
    idiv qword ptr z
    mov  qword ptr x[8], rax
    mov  rax, qword ptr y
    idiv qword ptr z
    mov  qword ptr x, rax
    ```

1.  转换如下：

    1.  ```
        ; Note: order of comparison (HO vs. LO) is irrelevant
        ; for "==" comparison.

         mov rax, qword ptr x[8]
            cmp rax, qword ptr y[8]
            jne skipElse
            mov rax, qword ptr x
            cmp rax, qword ptr y
            jne skipElse
            `then code`
        skipElse:
        ```

    1.  ```
         mov rax, qword ptr x[8]
            cmp rax, qword ptr y[8]
            jnb skipElse
            mov rax, qword ptr x
            cmp rax, qword ptr y
            jnb skipElse
           ` then code`
        skipElse:
        ```

    1.  ```
         mov rax, qword ptr x[8]
            cmp rax, qword ptr y[8]
            jna skipElse
            mov rax, qword ptr x
            cmp rax, qword ptr y
            jna skipElse
            `then code`
        skipElse:
        ```

    1.  ```
        ; Note: order of comparison (HO vs. LO) is irrelevant
        ; for "!=" comparison.

            mov rax, qword ptr x[8]
            cmp rax, qword ptr y[8]
            jne doElse
            mov rax, qword ptr x
            cmp rax, qword ptr y
            je skipElse
        doElse:
            `then code`
        skipElse:
        ```

1.  转换如下：

    1.  ```
        ; Note: order of comparison (HO vs. LO) is irrelevant
        ; for "==" comparison.

            mov eax, dword ptr x[8]
            cmp eax, dword ptr y[8]
         jne skipElse
            mov rax, qword ptr x
            cmp rax, qword ptr y
            jne skipElse
            `then code`
        skipElse:
        ```

    1.  ```
         mov eax, dword ptr x[8]
            cmp eax, dword ptr y[8]
            jnb skipElse
            mov rax, qword ptr x
            cmp rax, qword ptr y
            jnb skipElse
            `then code`
        skipElse:
        ```

    1.  ```
         mov eax, dword ptr x[8]
            cmp eax, dword ptr y[8]
            jna skipElse
            mov rax, qword ptr x
            cmp rax, qword ptr y
            jna skipElse
            `then code`
        skipElse:
        ```

1.  转换如下：

    1.  ```
        neg qword ptr x[8]
        neg qword ptr x
        sbb qword ptr x[8], 0

        xor rax, rax
        xor rdx, rdx
        sub rax, qword ptr x
        sbb rdx, qword ptr x[8]
        mov qword ptr x, rax
        mov qword ptr x[8], rdx
        ```

    1.  ```
        mov rax, qword ptr y
        mov rdx, qword ptr y[8]
        neg rdx
        neg rax
        sbb rdx, 0
        mov qword ptr x, rax
        mov qword ptr x[8], rdx

        xor rdx, rdx
        xor rax, rax
        sub rax, qword ptr y
        sbb rdx, qword ptr y[8]
        mov qword ptr x, rax
        mov qword ptr x[8], rdx
        ```

1.  转换如下：

    1.  ```
        mov rax, qword ptr y
        and rax, qword ptr z
        mov qword ptr x, rax
        mov rax, qword ptr y[8]
        and rax, qword ptr z[8]
        mov qword ptr x[8], rax
        ```

    1.  ```
        mov rax, qword ptr y
        or  rax, qword ptr z
        mov qword ptr x, rax
        mov rax, qword ptr y[8]
        or  rax, qword ptr z[8]
        mov qword ptr x[8], rax
        ```

    1.  ```
        mov rax, qword ptr y
        xor rax, qword ptr z
        mov qword ptr x, rax
        mov rax, qword ptr y[8]
        xor rax, qword ptr z[8]
        mov qword ptr x[8], rax
        ```

    1.  ```
        mov rax, qword ptr y
        not rax
        mov qword ptr x, rax
        mov rax, qword ptr y[8]
        not rax
        mov qword ptr x[8], rax
        ```

    1.  ```
        mov rax, qword ptr y
        shl rax, 1
        mov qword ptr x, rax
        mov rax, qword ptr y[8]
        rcl rax, 1
        mov qword ptr x[8], rax
        ```

    1.  ```
        mov rax, qword ptr y[8]
        shr rax, 1
        mov qword ptr x[8], rax
        mov rax, qword ptr y
        rcr rax, 1
        mov qword ptr x rax
        ```

1.  ```
    mov rax, qword ptr y[8]
    sar rax, 1
    mov qword ptr x[8], rax
    mov rax, qword ptr y
    rcr rax, 1
    mov qword ptr x, rax
    ```

1.  ```
    rcl qword ptr x, 1
    rcl qword ptr x[8], 1
    ```

1.  ```
    rcr qword ptr x[8], 1
    rcr qword ptr x, 1
    ```

## E.9 第九章问题的答案

1.  ```
    btoh        proc

                mov     ah, al      ; Do HO nibble first
                shr     ah, 4       ; Move HO nibble to LO
                or      ah, '0'     ; Convert to char
                cmp     ah, '9' + 1 ; Is it "A" to "F"?
                jb      AHisGood

    ; Convert 3Ah to 3Fh to "A" to "F".

                add     ah, 7

    ; Process the LO nibble here.

    AHisGood:   and     al, 0Fh     ; Strip away HO nibble
                or      al, '0'     ; Convert to char
                cmp     al, '9' + 1 ; Is it "A" to "F"?
                jb      ALisGood

    ; Convert 3Ah to 3Fh to "A" to "F".

     add     al, 7
    ALisGood:   ret
    btoh        endp
    ```

1.  8

1.  调用 `qToStr` 两次：一次使用高 64 位，一次使用低 64 位。然后将两个字符串连接起来。

1.  `fbstp`

1.  如果输入值为负数，发出一个连字符（`-`）字符并取其负值；然后调用无符号十进制转换函数。如果数字为 0 或正数，仅调用无符号十进制转换函数。

1.  ```
    ; Inputs:
    ;    RAX -   Number to convert to string.
    ;    CL  -   minDigits (minimum print positions).
    ;    CH  -   Padding character.
    ;    RDI -   Buffer pointer for output string.
    ```

1.  它将生成所需的完整字符串；`minDigits` 参数指定字符串的最小大小。

1.  ```
    ; On Entry:

       ; r10        - Real10 value to convert.
       ;              Passed in ST(0).

       ; fWidth     - Field width for the number (note that this
       ;              is an *exact* field width, not a minimum
       ;              field width).
       ;              Passed in EAX (RAX).

       ; decimalpts - # of digits to display after the decimal pt.
       ;              Passed in EDX (RDX). 

       ; fill       - Padding character if the number is smaller
       ;              than the specified field width.
       ;              Passed in CL (RCX).

       ; buffer     - r10ToStr stores the resulting characters
       ;              in this string.
       ;              Address passed in RDI.

       ; maxLength  - Maximum string length.
       ;              Passed in R8D (R8).
    ```

1.  一个包含 `fWidth` 个 `#` 字符的字符串。

1.  ```
    ; On Entry:

    ;    e10     - Real10 value to convert.
    ;              Passed in ST(0).

    ;    width   - Field width for the number (note that this
    ;              is an *exact* field width, not a minimum
    ;              field width).
    ;              Passed in RAX (LO 32 bits).

    ;    fill    - Padding character if the number is smaller
    ;              than the specified field width.
    ;              Passed in RCX.

    ;    buffer  - e10ToStr stores the resulting characters in
    ;              this buffer (passed in EDI).
    ;              Passed in RDI (LO 32 bits).

    ;    expDigs - Number of exponent digits (2 for real4,
    ;              3 for real8, and 4 for real10).
    ;              Passed in RDX (LO 8 bits).
    ```

1.  用于分隔字符序列与其他此类序列的字符，例如开始或结束一个数字字符串

1.  输入中的非法字符和转换过程中的数值溢出

## E.10 第十章问题的答案

1.  所有可能的输入（参数）值的集合

1.  所有可能的函数输出（返回）值的集合

1.  计算 AL = [RBX + AL × 1]

1.  字节值：域是从 0 到 255 的所有整数集合，范围也是从 0 到 255 的所有整数集合。

1.  实现这些功能的代码如下：

    1.  ```
        lea rbx, f
        mov al, input
        xlat
        ```

    1.  ```
        lea rbx, f
        movzx rax, input
        mov ax, [rbx][rax * 2]
        ```

    1.  ```
        lea rbx, f
        movzx rax, input
        mov al, [rbx][rax * 1]
        ```

    1.  ```
        lea rbx, f
        movzx rax, input
        mov ax, [rbx][rax * 2]
        ```

1.  修改输入值，使其位于函数的输入域内

1.  主存储器非常慢，查找表中的值可能比计算值更快。

## E.11 第十一章问题的答案

1.  使用 `cpuid` 指令。

1.  因为 Intel 和 AMD 有不同的功能集

1.  EAX = 1

1.  ECX 的第 20 位

1.  (a) `_TEXT`，(b) `_DATA`，(c) `_BSS`，(d) `CONST`

1.  `PARA` 或 16 字节

1.  ```
    data  segment align(64) 'DATA'
               .
               .
               .
    data  ends
    ```

1.  AVX/AVX2/AVX-256/AVX-512

1.  SIMD 寄存器中的数据类型；通常为 1、2、4 或 8 字节宽

1.  标量指令对单一数据项进行操作；向量指令同时对两个或更多数据项进行操作。

1.  16 字节

1.  32 字节

1.  64 字节

1.  `movd`

1.  `movq`

1.  `movaps`、`movapd` 和 `movdqa`

1.  `movups`、`movupd` 和 `movdqu`

1.  `movhps` 或 `movhpd`

1.  `movddup`

1.  `pshufb`

1.  `pshufd`，不过`pshufb`也可以起作用

1.  `(v)pextrb`、`(v)pextrw`、`(v)pextrd` 或 `(v)pextrq`

1.  `(v)pinsrb`、`(v)pinsrw`、`(v)pinsrd` 或 `(v)pinsrq`

1.  它获取第二操作数的位，反转它们，然后将这些反转的位与第一个（目标）操作数进行逻辑与运算。

1.  `pslldq`

1.  `pslrdq`

1.  `psllq`

1.  `pslrq`

1.  HO 位的进位被丢失。

1.  在垂直加法中，CPU 将两个不同 XMM 寄存器相同通道中的值相加；在水平加法中，CPU 将同一个 XMM 寄存器相邻通道中的值相加。

1.  在目标 XMM 寄存器中，通过将 0FFh 存储到目标 XMM 寄存器的相应通道中（0 表示假）

1.  交换 `pcmpgtq` 指令的操作数。

1.  它将每个字节的 HO 位从 XMM 寄存器复制到通用 16 位寄存器的相应位位置；例如，通道 0 的第 7 位进入第 0 位。

1.  (a) SSE 上为 4，AVX2 上为 8，(b) SSE 上为 2，AVX2 上为 4

1.  `and rax, -16`

1.  `pxor xmm0, xmm0`

1.  `pcmpeqb xmm1, xmm1`

1.  `include`

## E.12 第十二章问题的答案

1.  `and`/`andn`

1.  `btr`

1.  `or`

1.  `bts`

1.  `xor`

1.  `btc`

1.  `test`/`and`

1.  `bt`

1.  `pext`

1.  `pdep`

1.  `bextr`

1.  `bsf`

1.  `bsr`

1.  反转寄存器并使用 `bsf`。

1.  反转寄存器并使用`bsr`。

1.  `popcnt`

## E.13 第十三章问题的答案

1.  编译时语言

1.  在汇编和编译过程中

1.  `echo`（或 `%out`）

1.  `.err`

1.  `=` 指令

1.  `!`

1.  它用表示该编译时表达式值的文本替换表达式。

1.  它用文本符号的展开替换文本符号。

1.  它在汇编时将两个或更多文本字符串连接起来，并将结果存储到文本符号中。

1.  它在 MASM 文本对象中搜索一个子字符串，并返回该子字符串在该对象中的索引；如果子字符串没有出现在更大的字符串中，则返回 0。

1.  它返回一个 MASM 文本字符串的长度。

1.  它从更大的 MASM 文本字符串中返回一个子字符串。

1.  `if`、`elseif`、`else` 和 `endif`

1.  `while`、`for`、`forc` 和 `endm`

1.  `forc`

1.  `macro`、`endm`

1.  指定宏的名称，宏扩展将在该位置发生。

1.  作为宏指令的操作数

1.  在宏操作数字段的参数名称后指定 `:req`。

1.  宏参数是可选的，默认情况下，如果没有 `:req` 后缀。

1.  在最后一个宏参数声明后使用 `:vararg` 后缀。

1.  使用条件汇编指令，如 `ifb` 或 `ifnb`，查看实际的宏参数是否为空。

1.  使用 `local` 指令。

1.  `exitm`

1.  使用 `exitm <text>`。

1.  `opattr`

## E.14 第十四章问题的答案

1.  字节、字、双字和四字

1.  `movs`、`cmps`、`scas`、`stos` 和 `lods`

1.  字节和字

1.  RSI、RDI 和 RCX

1.  RSI 和 RDI

1.  RCX、RSI 和 AL

1.  RDI 和 EAX

1.  Dir = 0

1.  Dir = 1

1.  清除方向标志；或者保留其值。

1.  清除

1.  `movs` 和 `stos`

1.  当源和目标块重叠，且源地址起始位置比目标块的内存地址更低时

1.  这是默认条件；当源地址和目标块重叠，且源地址起始位置比目标块的内存地址更高时，你还需要清除方向标志。

1.  源块的部分内容可以在目标块中复制。

1.  `repe`

1.  方向标志应清除。

1.  不，字符串指令在使用重复前缀时，会在字符串操作之前测试 RCX。

1.  `scasb`

1.  `stos`

1.  `lods` 和 `stos`

1.  `lods`

1.  验证 CPU 是否支持 SSE 4.2 指令。

1.  `pcmpistri` 和 `pcmpistrm`

1.  `pcmpestri` 和 `pcmpestrm`

1.  RAX 存储 `src1` 长度，RDX 存储 `src2` 长度。

1.  等于任何，或可能等于的范围

1.  等于每个

1.  等于已排序

1.  `pcmp``X``str``Y` 指令总是读取 16 字节的内存，即使字符串长度不足，也有可能在读取字符串末尾超出时发生 MMU 页面错误。

## E.15 第十五章问题的答案

1.  `ifndef` 和 `endif`

1.  汇编源文件及其包含或间接包含的所有文件

1.  `public`

1.  `extern` 和 `externdef`

1.  `externdef`

1.  `abs`

1.  `proc`

1.  *nmake.exe*

1.  多个如下形式的块：

    ```
    `target`: `dependencies`
        `commands`
    ```

1.  依赖文件是当前文件正常操作所依赖的文件；该依赖文件必须在当前文件的编译和链接之前更新和构建。

1.  删除旧的对象和可执行文件，并删除其他杂项文件。

1.  一组目标文件

## E.16 第十六章问题的答案

1.  `/subsystem:console`

1.  [`www.masm32.com/`](https://www.masm32.com/)

1.  它会减慢汇编过程。

1.  `/entry:``procedure_name`

1.  `MessageBox`

1.  包围函数调用并改变你调用函数方式的代码（例如，参数顺序和位置）

1.  `__imp_CreateFileA`

1.  `__imp_GetLastError`
