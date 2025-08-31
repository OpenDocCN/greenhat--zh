

## 第二十三章：B 汇编生成与代码输出表格



![](img/opener-img.jpg)

在每一章中，关于将 TACKY 转换为汇编语言或代码生成的部分，我都包含了总结这些过程的表格。从第四章开始，这些表格仅展示了该章节中所做的更改，而非整个过程。附录中展示了总结这些过程的完整表格，位于第一部分、第二部分和第三部分的末尾。

## 第一部分

本节中的第一组表格说明了编译器应该如何将每个 TACKY 构造转换为汇编语言，位于第一部分的末尾。第二组表格说明了编译器应该如何输出每个汇编构造，位于第一部分的末尾。

### 将 TACKY 转换为汇编语言

表 B-1 到 B-5 展示了将 TACKY 转换为汇编语言的完整过程，位于第一部分的末尾。

表 B-1: 将顶级 TACKY 构造转换为汇编语言

| TACKY 顶级构造 | 汇编顶级构造 |
| --- | --- |

|

```
Program(top_level_defs)
```

|

```
Program(top_level_defs)
```

|

|

```
Function(name, global, params,
         instructions)
```

|

```
Function(name, global,
         [Mov(Reg(DI), param1),
          Mov(Reg(SI), param2),
 <copy next four parameters from registers>,
 Mov(Stack(16), param7),
          Mov(Stack(24), param8), 
          <copy remaining parameters from stack>] +
         instructions)
```

|

|

```
StaticVariable(name, global, init)
```

|

```
StaticVariable(name, global, init)
```

|

表 B-2: 将 TACKY 指令转换为汇编语言

| TACKY 指令 | 汇编指令 |
| --- | --- |
| Return(val) | Mov(val, Reg(AX)) Ret |

| Unary(非运算, src, dst) | Cmp(Imm(0), src) Mov(Imm(0), dst)

SetCC(E, dst) |

| Unary(一元运算符, src, dst) | Mov(src, dst) Unary(一元运算符, dst) |
| --- | --- |

| Binary(除法, src1, src2, dst) | Mov(src1, Reg(AX)) Cdq

Idiv(src2)

Mov(Reg(AX), dst) |

| Binary(余数, src1, src2, dst) | Mov(src1, Reg(AX)) Cdq

Idiv(src2)

Mov(Reg(DX), dst) |

| Binary(算术运算符, src1, src2, dst) | Mov(src1, dst) Binary(算术运算符, src2, dst) |
| --- | --- |

| Binary(关系操作符, src1, src2, dst) | Cmp(src2, src1) Mov(Imm(0), dst)

SetCC(关系操作符, dst) |

| Jump(目标) | Jmp(目标) |
| --- | --- |
| JumpIfZero(条件, 目标) | Cmp(Imm(0), 条件) JmpCC(E, 目标) |
| JumpIfNotZero(条件, 目标) | Cmp(Imm(0), 条件) JmpCC(NE, 目标) |
| Copy(src, dst) | Mov(src, dst) |
| Label(标识符) | Label(标识符) |

| FunCall(函数名, 参数, dst) | <修复堆栈对齐> <设置参数>

Call(函数名)

<deallocate 参数/填充>

Mov(Reg(AX), dst) |

表 B-3: 将 TACKY 算术运算符转换为汇编

| TACKY 运算符 | 汇编运算符 |
| --- | --- |
| Complement | Not |
| Negate | Neg |
| Add | Add |
| Subtract | Sub |
| Multiply | Mult |

表 B-4: 将 TACKY 比较转换为汇编

| TACKY 比较 | 汇编条件码 |
| --- | --- |
| Equal | E |
| NotEqual | NE |
| LessThan | L |
| LessOrEqual | LE |
| GreaterThan | G |
| GreaterOrEqual | GE |

表 B-5: 将 TACKY 操作数转换为汇编语言

| TACKY 操作数 | 汇编操作数 |
| --- | --- |
| Constant(int) | Imm(int) |
| Var(identifier) | Pseudo(identifier) |

### 代码生成

表 B-6 到 B-10 展示了 第一部分结束时的完整代码生成过程。

表 B-6: 格式化顶层汇编结构

| 汇编语言顶层结构 |  | 输出 |
| --- | --- | --- |

|

```
Program(top_levels)
```

|

```
Print out each top-level construct. On Linux, add at end of file:
 .section .note.GNU-stack,"",@progbits
```

|

|

```
Function(name, global, instructions)
```

|

```
 <global-directive>
 .text
<name>:
    pushq    %rbp
    movq     %rsp, %rbp 
 <instructions>
```

|

|

```
StaticVariable(name, global, init)
```

| 初始化为零 |
| --- |

```
 <global-directive>
 .bss
 <alignment-directive>
<name>:
    .zero 4
```

|

| 初始化为非零值 |
| --- |

```
 <global-directive>
 .data
 <alignment-directive>
<name>:
    .long <init>
```

|

| 全局指令 |
| --- |

```
If global is true:
.globl <identifier>
Otherwise, omit this directive.
```

|

| 对齐指令 | 仅限 Linux | .align 4 |
| --- | --- | --- |
|  | macOS 或 Linux | .balign 4 |

表 B-7: 格式化汇编指令

| 汇编指令 | 输出 |
| --- | --- |

|

```
Mov(src, dst)
```

|

```
movl    <src>, <dst>
```

|

|

```
Ret
```

|

```
movq    %rbp, %rsp
popq    %rbp
ret
```

|

|

```
Unary(unary_operator, operand)
```

|

```
<unary_operator>     <operand>
```

|

|

```
Binary(binary_operator, src, dst)
```

|

```
<binary_operator>    <src>, <dst>
```

|

|

```
Idiv(operand)
```

|

```
idivl   <operand>
```

|

|

```
Cdq
```

|

```
cdq
```

|

|

```
AllocateStack(int)
```

|

```
subq    $<int>, %rsp
```

|

|

```
DeallocateStack(int)
```

|

```
addq    $<int>, %rsp
```

|

|

```
Push(operand)
```

|

```
pushq   <operand>
```

|

|

```
Call(label)
```

|

```
call    <label>
or
call    <label>@PLT
```

|

|

```
Cmp(operand, operand)
```

|

```
cmpl    <operand>, <operand>
```

|

|

```
Jmp(label)
```

|

```
jmp     .L<label>
```

|

|

```
JmpCC(cond_code, label)
```

|

```
j<cond_code>      .L<label>
```

|

|

```
SetCC(cond_code, operand)
```

|

```
set<cond_code>    <operand>
```

|

|

```
Label(label)
```

|

```
.L<label>:
```

|

表 B-8: 汇编运算符的指令名称

| 汇编运算符 | 指令名称 |
| --- | --- |
| Neg | negl |
| Not | notl |
| Add | addl |
| Sub | subl |
| Mult | imull |

表 B-9： 条件码的指令后缀

| 条件码 | 指令后缀 |
| --- | --- |
| E | e |
| NE | ne |
| L | l |
| LE | le |
| G | g |
| GE | ge |

表 B-10： 汇编操作数格式

| 汇编操作数 |  | 输出 |
| --- | --- | --- |
| Reg(AX) | 8 字节 | %rax |
|  | 4 字节 | %eax |
|  | 1 字节 | %al |
| Reg(DX) | 8 字节 | %rdx |
|  | 4 字节 | %edx |
|  | 1 字节 | %dl |
| Reg(CX) | 8 字节 | %rcx |
|  | 4 字节 | %ecx |
|  | 1 字节 | %cl |
| Reg(DI) | 8 字节 | %rdi |
|  | 4 字节 | %edi |
|  | 1 字节 | %dil |
| Reg(SI) | 8 字节 | %rsi |
|  | 4 字节 | %esi |
|  | 1 字节 | %sil |
| Reg(R8) | 8 字节 | %r8 |
|  | 4 字节 | %r8d |
|  | 1 字节 | %r8b |
| Reg(R9) | 8 字节 | %r9 |
|  | 4 字节 | %r9d |
|  | 1 字节 | %r9b |
| Reg(R10) | 8 字节 | %r10 |
|  | 4 字节 | %r10d |
|  | 1 字节 | %r10b |
| Reg(R11) | 8 字节 | %r11 |
|  | 4 字节 | %r11d |
|  | 1 字节 | %r11b |
| Stack(int) |  | <int>(%rbp) |
| 立即数（int） |  | $<int> |
| 数据（标识符） |  | <标识符>（%rip） |

## 第二部分

本节的第一组表格展示了编译器如何将每个 TACKY 构造转换为汇编语言，在第二部分结束时。第二组表格展示了编译器如何生成每个汇编构造，同样在第二部分结束时。

### 将 TACKY 转换为汇编

表 B-11 至 B-16 展示了从 TACKY 到汇编的完整转换，见第二部分结束。

表 B-11： 将顶层 TACKY 构造转换为汇编

| TACKY 顶层构造 | 汇编顶层构造 |
| --- | --- |
| 程序（顶层定义） |

```
Program(top_level_defs + <all StaticConstant constructs for
       floating-point constants>)
```

|

|

```
Function(name,
         global,
         params,
         instructions)
```

| 寄存器中的返回值或无返回值 |
| --- |

```
Function(name, global, 
  [<copy Reg(DI) into first int param/eightbyte>,
 <copy Reg(SI) into second int param/eightbyte>,
 <copy next four int params/eightbytes from registers>,
    Mov(Double,
        Reg(XMM0),
        <first double param/eightbyte>),
```

|

|  |  |
| --- | --- |

```
 Mov(Double,
        Reg(XMM1),
        <second double param/eightbyte>),
    <copy next six double params/eightbytes from registers>,
    <copy Memory(BP, 16) into first stack param/eightbyte>,
    <copy Memory(BP, 24) into second stack param/eightbyte>,
    <copy remaining params/eightbytes from stack>] +
  instructions)
```

|

|  | 栈上的返回值 |
| --- | --- |

```
Function(name, global,
    [Mov(Quadword,
        Reg(DI),
        Memory(BP, -8)),
    <copy Reg(SI) into first int param/eightbyte>,
    <copy Reg(DX) into second int param/eightbyte>,
    <copy next three int params/eightbytes from registers>,
    Mov(Double,
        Reg(XMM0),
        <first double param/eightbyte>),
    Mov(Double,
        Reg(XMM1),
        <second double param/eightbyte>),
    <copy next six double params/eightbytes from registers>,
    <copy Memory(BP, 16) into first stack param/eightbyte>,
    <copy Memory(BP, 24) into second stack param/eightbyte>,
    <copy remaining params/eightbytes from stack>] +
  instructions)
```

|

|

```
StaticVariable(name, global, t,
               init_list)
```

|

```
StaticVariable(name, global, <alignment of t>,
               init_list)
```

|

|

```
StaticConstant(name, t, init)
```

|

```
StaticConstant(name, <alignment of t>, init)
```

|

表 B-12： 将 TACKY 指令转换为汇编

| TACKY 指令 | 汇编指令 |
| --- | --- |
| 返回（val） | 栈上的返回值 |

```
Mov(Quadword, Memory(BP, -8), Reg(AX))
Mov(Quadword,
     <first eightbyte of return value>,
     Memory(AX, 0))
 Mov(Quadword,
     <second eightbyte of return value>,
     Memory(AX, 8))
 <copy rest of return value>
 Ret
```

|

|  | 寄存器中的返回值 |
| --- | --- |

```
<move integer parts of return value into RAX, RDX>
<move double parts of return value into XMM0, XMM1>
Ret
```

|

|  | 无返回值 |
| --- | --- |

```
Ret
```

|

| 一元运算符（Not，src，dst） | 整数 |
| --- | --- |

```
Cmp(<src type>, Imm(0), src)
Mov(<dst type>, Imm(0), dst)
SetCC(E, dst)
```

|

| double |
| --- |

```
Binary(Xor, Double, Reg(<X>), Reg(<X>))
Cmp(Double, src, Reg(<X>))
Mov(<dst type>, Imm(0), dst)
SetCC(E, dst)
```

|

|

```
Unary(Negate, src, dst)
(double negation)
```

|

```
Mov(Double, src, dst)
Binary(Xor, Double, Data(<negative-zero>, 0), dst)
And add a top-level constant:
StaticConstant(<negative-zero>, 16,
                DoubleInit(-0.0))
```

|

| 一元运算符（unary_operator，src，dst） |
| --- |

```
Mov(<src type>, src, dst)
Unary(unary_operator, <src type>, dst)
```

|

|

```
Binary(Divide, src1,
       src2, dst)
(integer division)
```

| 有符号 |
| --- |

```
Mov(<src1 type>, src1, Reg(AX))
Cdq(<src1 type>)
Idiv(<src1 type>, src2)
Mov(<src1 type>, Reg(AX), dst)
```

|

|  | 无符号 |
| --- | --- |

```
Mov(<src1 type>, src1, Reg(AX))
Mov(<src1 type>, Imm(0), Reg(DX))
Div(<src1 type>, src2)
Mov(<src1 type>, Reg(AX), dst)
```

|

|

```
Binary(Remainder, src1,
       src2, dst)
```

| 有符号 |
| --- |

```
Mov(<src1 type>, src1, Reg(AX))
Cdq(<src1 type>) 
div(<src1 type>, src2)
Mov(<src1 type>, Reg(DX), dst)
```

|

| 无符号 |
| --- |

```
Mov(<src1 type>, src1, Reg(AX))
Mov(<src1 type>, Imm(0), Reg(DX))
Div(<src1 type>, src2)
Mov(<src1 type>, Reg(DX), dst)
```

|

|

```
Binary(arithmetic_operator, src1,
       src2, dst)
```

|

```
Mov(<src1 type>, src1, dst)
Binary(arithmetic_operator, <src1 type>, src2, dst)
```

|

|

```
Binary(relational_operator, src1,
      src2, dst)
```

|

```
Cmp(<src1 type>, src2, src1)
Mov(<dst type>, Imm(0), dst)
SetCC(relational_operator, dst)
```

|

| 跳转（目标） |
| --- |

```
Jmp(target)
```

|

|

```
JumpIfZero(condition,
           target)
```

| 整数 |
| --- |

```
Cmp(<condition type>, Imm(0), condition)
JmpCC(E, target)
```

|

| double |
| --- |

```
Binary(Xor, Double, Reg(<X>), Reg(<X>))
Cmp(Double, condition, Reg(<X>))
JmpCC(E, target)
```

|

|

```
JumpIfNotZero(condition,
              target)
```

| 整数 |
| --- |

```
Cmp(<condition type>, Imm(0), condition)
JmpCC(NE, target)
```

|

| double |
| --- |

```
Binary(Xor, Double, Reg(<X>), Reg(<X>))
Cmp(Double, condition, Reg(<X>))
JmpCC(NE, target)
```

|

| Copy(src, dst) | 标量 |
| --- | --- |

```
Mov(<src type>, src, dst)
```

|

| 结构 |
| --- |

```
Mov(<first chunk type>,
     PseudoMem(src, 0),
     PseudoMem(dst, 0))
Mov(<next chunk type>,
     PseudoMem(src, <first chunk size>),
     PseudoMem(dst, <first chunk size>))
<copy remaining chunks>
```

|

| Load(ptr, dst) | 标量 |
| --- | --- |

```
Mov(Quadword, ptr, Reg(<R>))
Mov(<dst type>, Memory(<R>, 0), dst)
```

|

| 结构 |
| --- |

```
Mov(Quadword, ptr, Reg(<R>))
Mov(<first chunk type>,
     Memory(<R>, 0),
     PseudoMem(dst, 0))
Mov(<next chunk type>,
     Memory(<R>, <first chunk size>),
     PseudoMem(dst, <first chunk size>))
<copy remaining chunks>
```

|

| Store(src, ptr) | 标量 |
| --- | --- |

```
Mov(Quadword, ptr, Reg(<R>))
Mov(<src type>, src, Memory(<R>, 0))
```

|

| 结构 |
| --- |

```
Mov(Quadword, ptr, Reg(<R>))
Mov(<first chunk type>,
     PseudoMem(src, 0),
     Memory(<R>, 0))
Mov(<next chunk type>,
     PseudoMem(src, <first chunk size>),
     Memory(<R>, <first chunk size>))
<copy remaining chunks>
```

|

| GetAddress(src, dst) |
| --- |

```
Lea(src, dst)
```

|

|

```
AddPtr(ptr, index, scale,
        dst)
```

| 常量索引 |
| --- |

```
Mov(Quadword, ptr, Reg(<R>))
Lea(Memory(<R>, index * scale), dst)
```

|

| 变量索引和 1, 2, 4 或 8 的尺度 |
| --- |

```
Mov(Quadword, ptr, Reg(<R1>))
Mov(Quadword, index, Reg(<R2>))
Lea(Indexed(<R1>, <R2>, scale), dst)
```

|

| 变量索引和其他尺度 |
| --- |

```
Mov(Quadword, ptr, Reg(<R1>))
Mov(Quadword, index, Reg(<R2>))
Binary(Mult, Quadword, Imm(scale), Reg(<R2>))
Lea(Indexed(<R1>, <R2>, 1), dst)
```

|

|

```
CopyToOffset(src, dst,
             offset)
```

| src 是标量 |
| --- |

```
Mov(<src type>, src, PseudoMem(dst, offset))
```

|

| src 是一个结构 |
| --- |

```
Mov(<first chunk type>,
     PseudoMem(src, 0),
     PseudoMem(dst, offset))
Mov(<next chunk type>,
     PseudoMem(src, <first chunk size>),
     PseudoMem(dst, offset + <first chunk size>))
<copy remaining chunks>
```

|

|  ``` CopyFromOffset(src,
               offset,
               dst)
```  | dst 是标量 |
| --- | --- |

```
Mov(<dst type>, PseudoMem(src, offset), dst)
```

|

| dst 是一个结构 |
| --- |

```
Mov(<first chunk type>,
     PseudoMem(src, offset),
     PseudoMem(dst, 0))
Mov(<next chunk type>,
     PseudoMem(src, offset + <first chunk size>),
     PseudoMem(dst, <first chunk size>))
<copy remaining chunks>
```

|

| 标签（标识符） |
| --- |

```
Label(identifier)
```

|

|

```
FunCall(fun_name, args,
         dst)
```

| dst 将会存储在内存中 |
| --- |

```
Lea(dst, Reg(DI))
<fix stack alignment>
<move arguments to general-purpose registers, starting with RSI>
<move arguments to XMM registers>
<push arguments onto the stack>
Call(fun_name)
<deallocate arguments/padding>
```

|

|  | dst 将会通过寄存器返回 |
| --- | --- |

```
<fix stack alignment>
<move arguments to general-purpose registers>
<move arguments to XMM registers>
<push arguments onto the stack>
Call(fun_name)
<deallocate arguments/padding>
<move integer parts of return value from RAX, RDX into dst>
<move double parts of return value from XMM0, XMM1 into dst>
```

|

|  | dst 不存在 |
| --- | --- |

```
<fix stack alignment>
<move arguments to general-purpose registers>
<move arguments to XMM registers>
<push arguments onto the stack>
Call(fun_name)
<deallocate arguments/padding>
```

|

| ZeroExtend(src, dst) |
| --- |

```
MovZeroExtend(<src type>, <dst type>, src, dst)
```

|

| SignExtend(src, dst) |
| --- |

```
Movsx(<src type>, <dst type>, src, dst)
```

|

| Truncate(src, dst) |
| --- |

```
Mov(<dst type>, src, dst)
```

|

| IntToDouble(src, dst) | char 或 signed char |
| --- | --- |

```
Movsx(Byte, Longword, src, Reg(<R>))
Cvtsi2sd(Longword, Reg(<R>), dst)
```

|

| int 或 long |
| --- |

```
Cvtsi2sd(<src type>, src, dst)
```

|

| DoubleToInt(src, dst) | char 或 signed char |
| --- | --- |

```
Cvttsd2si(Longword, src, Reg(<R>))
Mov(Byte, Reg(<R>), dst)
```

|

| 整数 或 长整数 |
| --- |

```
Cvttsd2si(<dst type>, src, dst)
```

|

| UIntToDouble(src, dst) | 无符号字符 |
| --- | --- |

```
MovZeroExtend(Byte, Longword, src, Reg(<R>))
Cvtsi2sd(Longword, Reg(<R>), dst)
```

|

|  | 无符号整数 |
| --- | --- |

```
MovZeroExtend(Longword, Quadword, src, Reg(<R>))
Cvtsi2sd(Quadword, Reg(<R>), dst)
```

|

|  | 无符号长整数 |
| --- | --- |

```
Cmp(Quadword, Imm(0), src)
JmpCC(L, <label1>)
Cvtsi2sd(Quadword, src, dst)
Jmp(<label2>)
Label(<label1>)
Mov(Quadword, src, Reg(<R1>))
Mov(Quadword, Reg(<R1>), Reg(<R2>))
Unary(Shr, Quadword, Reg(<R2>))
Binary(And, Quadword, Imm(1), Reg(<R1>))
Binary(Or, Quadword, Reg(<R1>), Reg(<R2>))
Cvtsi2sd(Quadword, Reg(<R2>), dst)
Binary(Add, Double, dst, dst) Label(<label2>)
```

|

| DoubleToUInt(src, dst) | 无符号字符 |
| --- | --- |

```
Cvttsd2si(Longword, src, Reg(<R>))
Mov(Byte, Reg(<R>), dst)
```

|

| 无符号整数 |
| --- |

```
Cvttsd2si(Quadword, src, Reg(<R>))
Mov(Longword, Reg(<R>), dst)
```

|

| 无符号长整数 |
| --- |

```
Cmp(Double, Data(<upper-bound>, 0), src)
JmpCC(AE, <label1>)
Cvttsd2si(Quadword, src, dst)
Jmp(<label2>)
Label(<label1>)
Mov(Double, src, Reg(<X>))
Binary(Sub, Double, Data(<upper-bound>, 0), Reg(<X>))
Cvttsd2si(Quadword, Reg(<X>), dst)
Mov(Quadword, Imm(9223372036854775808), Reg(<R>))
Binary(Add, Quadword, Reg(<R>), dst)
Label(<label2>)
And add a top-level constant:
StaticConstant(<upper-bound>, 8,
                DoubleInit(9223372036854775808.0))
```

|

表 B-13: 将 TACKY 算术运算符转换为汇编

| TACKY 运算符 | 汇编运算符 |
| --- | --- |
| 补码 | Not |
| 取反 (整数取反) | Neg |
| 加法 | Add |
| 减法 | Sub |
| 乘法 | Mult |
| 除法 (双精度 除法) | DivDouble |

表 B-14: 将 TACKY 比较操作转换为汇编

| TACKY 比较 | 汇编条件码 |
| --- | --- |
| 等于 | E |
| 不相等 | NE |
| 小于 | 符号数 | L |
| 无符号，指针，或 双精度 | B |
| 小于等于 | 有符号 | LE |
|  | 无符号、指针或 双精度 | BE |
| 大于 | 有符号 | G |
|  | 无符号、指针或 双精度 | A |
| 大于等于 | 有符号 | GE |
|  | 无符号、指针或 双精度 | AE |

表 B-15： 将 TACKY 操作数转换为汇编

| TACKY 操作数 |  | 汇编操作数 |
| --- | --- | --- |
| 常量(ConstChar(int)) |  | 立即数(int) |
| 常量(ConstInt(int)) |  | 立即数(int) |
| 常量(ConstLong(int)) |  | 立即数(int) |
| 常量(ConstUChar(int)) |  | 立即数(int) |
| 常量(ConstUInt(int)) |  | 立即数(int) |
| 常量(ConstULong(int)) |  | 立即数(int) |

| 常量(ConstDouble(double)) |  | 数据(<ident>, 0) 并添加一个顶层常量：

静态常量(<ident>, 8, DoubleInit(double)) |

| Var(identifier) | 标量值 | Pseudo(identifier) |
| --- | --- | --- |
|  | 聚合值 | PseudoMem(identifier, 0) |

表 B-16： 类型转换为汇编语言

| 源类型 |  | 汇编类型 | 对齐方式 |
| --- | --- | --- | --- |
| Char |  | 字节 | 1 |
| SChar |  | 字节 | 1 |
| UChar |  | 字节 | 1 |
| Int |  | 长整型 | 4 |
| UInt |  | 长整型 | 4 |
| Long |  | 四字长整型 | 8 |
| ULong |  | 四字长整型 | 8 |
| Double |  | 双精度 | 8 |
| Pointer(referenced_t) |  | 四字长整型 | 8 |
| Array(element, size) | 大小为 16 字节或更大的变量 | ByteArray(<size of element> * 大小, 16) | 16 |
|  | 其他所有内容 | 字节数组（<size of element> * 大小， <alignment of element>) | 与 元素 相同的对齐方式 |
| 结构（标签） |  | 字节数组（<size from type table>, <alignment from type table>) | 来自类型表的对齐 |

### 代码发射

表 B-17 至 B-23 显示了在 第二部分 结束时的完整代码发射过程。

表 B-17: 格式化顶级汇编构造

| 汇编顶级构造 |  | 输出 |
| --- | --- | --- |
| 程序（顶层构造） |

```
Print out each top-level construct.
On Linux, add at end of file:
     .section .note.GNU-stack,"",@progbits
```

|

| 函数（名称，全局，指令） |
| --- |

```
 <global-directive>
     .text
 <name>:
     pushq    %rbp
     movq     %rsp, %rbp
     <instructions>
```

|

|

```
StaticVariable(name, global,
               alignment,
               init_list)
```

| 初始化为零的整数，或任何仅用 ZeroInit 初始化的变量 |
| --- |

```
 <global-directive>
     .bss
     <alignment-directive>
 <name>:
     <init_list>
```

|

| 所有其他变量 |
| --- |

```
 <global-directive>
     .data
     <alignment-directive>
 <name>:
     <init_list>
```

|

|

```
StaticConstant(name, alignment,
               init)
```

| Linux |
| --- |

```
 .section .rodata
     <alignment-directive>
 <name>:
     <init>
```

|

|  | macOS（8 字节对齐的数字常量） |
| --- | --- |

```
 .literal8
    .balign 8
 <name>:
     <init>
```

|

|  | macOS（16 字节对齐的数字常量） |
| --- | --- |

```
 .literal16
    .balign 16
 <name>:
     <init>
     .quad 0
```

|

|  | macOS（字符串常量） |
| --- | --- |

```
 .cstring
 <name>:
     <init>
```

|

| 全局指令 |  |
| --- | --- |

```
 If global is true:
 .globl <identifier>
 Otherwise, omit this directive.
```

|

| 对齐指令 | 仅限 Linux |
| --- | --- |

```
.align <alignment>
```

|

|  | macOS 或 Linux |
| --- | --- |

```
.balign <alignment>
```

|

表 B-18: 格式化静态初始化器

| 静态初始化器 | 输出 |
| --- | --- |
| CharInit（0） | .zero 1 |
| CharInit(i) | .byte <i> |
| IntInit(0) | .zero 4 |
| IntInit(i) | .long <i> |
| LongInit(0) | .zero 8 |
| LongInit(i) | .quad <i> |
| UCharInit(0) | .zero 1 |
| UCharInit(i) | .byte <i> |
| UIntInit(0) | .zero 4 |
| UIntInit(i) | .long <i> |
| ULongInit(0) | .zero 8 |
| ULongInit(i) | .quad <i> |
| ZeroInit(n) | .zero <n> |
| DoubleInit(d) |

```
.double <d>
 or
 .quad <d-interpreted-as-long>
```

|

| StringInit(s, True) | .asciz "<s>" |
| --- | --- |
| StringInit(s, False) | .ascii "<s>" |
| PointerInit(label) | .quad <label> |

表 B-19： 格式化汇编指令

| 汇编指令 | 输出 |
| --- | --- |
| Mov(t, src, dst) |

```
mov<t>   <src>, <dst>
```

|

| Movsx(src_t, dst_t, src, dst) |
| --- |

```
movs<src_t><dst_t>    <src>, <dst>
```

|

| MovZeroExtend(src_t, dst_t, src, dst) |
| --- |

```
movz<src_t><dst_t>    <src>, <dst>
```

|

| Lea |
| --- |

```
leaq     <src>, <dst>
```

|

| Cvtsi2sd(t, src, dst) |
| --- |

```
cvtsi2sd<t>     <src>, <dst>
```

|

| Cvttsd2si(t, src, dst) |
| --- |

```
cvttsd2si<t>    <src>, <dst>
```

|

| Ret |
| --- |

```
movq     %rbp, %rsp
popq     %rbp
ret
```

|

| Unary(unary_operator, t, operand) |
| --- |

```
<unary_operator><t>     <operand>
```

|

| Binary(Xor, Double, src, dst) |
| --- |

```
xorpd    <src>, <dst>
```

|

| Binary(Mult, Double, src, dst) |
| --- |

```
mulsd    <src>, <dst>
```

|

| Binary(binary_operator, t, src, dst) |
| --- |

```
<binary_operator><t>    <src>, <dst>
```

|

| Idiv(t, operand) |
| --- |

```
idiv<t>  <operand>
```

|

| Div(t, operand) |
| --- |

```
div<t>  <operand>
```

|

| Cdq(Longword) |
| --- |

```
cdq
```

|

| Cdq(Quadword) |
| --- |

```
cqo
```

|

| Push(operand) |
| --- |

```
pushq    <operand>
```

|

| Call(label) |
| --- |

```
call     <label>
or
call     <label>@PLT
```

|

| Cmp(Double, operand, operand) |
| --- |

```
comisd   <operand>, <operand>
```

|

| Cmp(t, operand, operand) |
| --- |

```
cmp<t>   <operand>, <operand>
```

|

| Jmp(label) |
| --- |

```
jmp      .L<label>
```

|

| JmpCC(cond_code, label) |
| --- |

```
j<cond_code> .L<label>
```

|

| SetCC(cond_code, operand) |
| --- |

```
set<cond_code>    <operand>
```

|

| Label(label) |
| --- |

```
.L<label>:
```

|

表 B-20： 汇编操作符的指令名称

| 汇编操作符 | 指令名称 |
| --- | --- |
| Neg | neg |
| Not | not |
| Shr | shr |
| Add | add |
| Sub | sub |
| Mult (仅限整数乘法) | imul |
| DivDouble | div |
| And | and |
| Or | or |
| Shl | shl |
| ShrTwoOp | shr |

表 B-21: 汇编类型的指令后缀

| 汇编类型 | 指令后缀 |
| --- | --- |
| Byte | b |
| Longword | l |
| Quadword | q |
| Double | sd |

表 B-22: 条件码的指令后缀

| 条件码 | 指令后缀 |
| --- | --- |
| E | e |
| NE | ne |
| L | l |
| LE | le |
| G | g |
| GE | ge |
| A | a |
| AE | ae |
| B | b |
| BE | be |

表 B-23: 汇编操作数格式

| 汇编操作数 |  | 输出 |
| --- | --- | --- |
| Reg(AX) | 8 字节 | %rax |
|  | 4 字节 | %eax |
|  | 1 字节 | %al |
| Reg(DX) | 8 字节 | %rdx |
|  | 4 字节 | %edx |
|  | 1 字节 | %dl |
| Reg(CX) | 8 字节 | %rcx |
|  | 4 字节 | %ecx |
|  | 1 字节 | %cl |
| Reg(DI) | 8 字节 | %rdi |
|  | 4 字节 | %edi |
|  | 1 字节 | %dil |
| Reg(SI) | 8 字节 | %rsi |
|  | 4 字节 | %esi |
|  | 1 字节 | %sil |
| Reg(R8) | 8 字节 | %r8 |
|  | 4 字节 | %r8d |
|  | 1 字节 | %r8b |
| Reg(R9) | 8-byte | %r9 |
|  | 4-byte | %r9d |
|  | 1-byte | %r9b |
| Reg(R10) | 8-byte | %r10 |
|  | 4-byte | %r10d |
|  | 1-byte | %r10b |
| Reg(R11) | 8-byte | %r11 |
|  | 4-byte | %r11d |
|  | 1-byte | %r11b |
| Reg(SP) |  | %rsp |
| Reg(BP) |  | %rbp |
| Reg(XMM0) |  | %xmm0 |
| Reg(XMM1) |  | %xmm1 |
| Reg(XMM2) |  | %xmm2 |
| Reg(XMM3) |  | %xmm3 |
| Reg(XMM4) |  | %xmm4 |
| Reg(XMM5) |  | %xmm5 |
| Reg(XMM6) |  | %xmm6 |
| Reg(XMM7) |  | %xmm7 |
| Reg(XMM14) |  | %xmm14 |
| Reg(XMM15) |  | %xmm15 |
| Memory(reg, int) |  | <int>(<reg>) |
| Indexed(reg1, reg2, int) |

```
(<reg1>,
<reg2>, <int>)
```

|

| Imm(int) |  |
| --- | --- |

```
$<int>
```

|

| Data(identifier, int) |  |
| --- | --- |

```
<identifier>
+<int>(%rip)
```

|

## 第三部分

在第三部分中，我们不会改变从 TACKY 到汇编的转换，但我们会向汇编的 AST 添加一些新的寄存器，并相应地更新代码生成步骤。本节末尾的代码生成步骤如何呈现，取决于你是否先完成了第二部分，或者是直接从第一部分跳到第三部分。

表 B-24 到 B-28 展示了如果你跳过第二部分，在第三部分末尾的完整代码生成步骤。

表 B-24： 格式化顶层汇编构造

| 汇编顶层构造 |  | 输出 |
| --- | --- | --- |
| Program(top_levels) |

```
Print out each top-level construct.
On Linux, add at end of file:
     .section .note.GNU-stack,"",@progbits
```

|

| Function(name, global, instructions) |
| --- |

```
 <global-directive>
     .text
 <name>:
     pushq    %rbp
     movq     %rsp, %rbp
     <instructions>
```

|

| StaticVariable(name, global, init) | 初始化为零 |
| --- | --- |

```
 <global-directive>
     .bss
     <alignment-directive>
<name>:
     .zero 4
```

|

|  | 初始化为非零值 |
| --- | --- |

```
 <global-directive>
     .data
     <alignment-directive>
 <name>:
     .long <init>
```

|

| 全局指令 |
| --- |

```
 If global is true:
.globl <identifier>
 Otherwise, omit this directive.
```

|

| 对齐指令 | 仅限 Linux |
| --- | --- |

```
.align 4
```

|

|  | macOS 或 Linux |
| --- | --- |

```
.balign 4
```

|

表 B-25： 格式化汇编指令

| 汇编指令 | 输出 |
| --- | --- |
| Mov(src, dst) |

```
movl     <src>, <dst>
```

|

| Ret |
| --- |

```
movq     %rbp, %rsp
popq     %rbp
ret
```

|

| Unary(unary_operator, operand) |
| --- |

```
<unary_operator>     <operand>
```

|

| Binary(binary_operator, src, dst) |
| --- |

```
<binary_operator>    <src>, <dst>
```

|

| Idiv(operand) |
| --- |

```
idivl    <operand>
```

|

| Cdq |
| --- |

```
cdq
```

|

| AllocateStack(int) |
| --- |

```
subq     $<int>, %rsp
```

|

| DeallocateStack(int) |
| --- |

```
addq     $<int>, %rsp
```

|

| Push(operand) |
| --- |

```
pushq    <operand>
```

|

| Pop(reg) |
| --- |

```
popq     <reg>
```

|

| Call(label) |
| --- |

```
call    <label>
or
call    <label>@PLT
```

|

| Cmp(operand, operand) |
| --- |

```
cmpl    <operand>, <operand>
```

|

| Jmp(label) |
| --- |

```
jmp     .L<label>
```

|

| JmpCC(cond_code, label) |
| --- |

```
j<cond_code> .L<label>
```

|

| SetCC(cond_code, operand) |
| --- |

```
set<cond_code>    <operand>
```

|

| Label(label) |
| --- |

```
.L<label>:
```

|

表 B-26: 汇编操作符的指令名称

| 汇编操作符 | 指令名称 |
| --- | --- |
| Neg | negl |
| Not | notl |
| Add | addl |
| Sub | subl |
| Mult | imull |

表 B-27: 条件代码的指令后缀

| 条件代码 | 指令后缀 |
| --- | --- |
| E | e |
| NE | ne |
| L | l |
| LE | le |
| G | g |
| GE | ge |

表 B-28: 汇编操作数的格式化

| 汇编操作数 | 输出 |
| --- | --- |
| Reg(AX) | 8 字节 | %rax |
|  | 4 字节 | %eax |
|  | 1 字节 | %al |
| Reg(DX) | 8 字节 | %rdx |
|  | 4 字节 | %edx |
|  | 1 字节 | %dl |
| Reg(CX) | 8 字节 | %rcx |
|  | 4 字节 | %ecx |
|  | 1 字节 | %cl |
| Reg(BX) | 8 字节 | %rbx |
|  | 4 字节 | %ebx |
|  | 1 字节 | %bl |
| Reg(DI) | 8 字节 | %rdi |
|  | 4 字节 | %edi |
|  | 1 字节 | %dil |
| Reg(SI) | 8 字节 | %rsi |
|  | 4 字节 | %esi |
|  | 1 字节 | %sil |
| Reg(R8) | 8 字节 | %r8 |
|  | 4 字节 | %r8d |
|  | 1 字节 | %r8b |
| Reg(R9) | 8 字节 | %r9 |
|  | 4 字节 | %r9d |
|  | 1 字节 | %r9b |
| Reg(R10) | 8 字节 | %r10 |
|  | 4 字节 | %r10d |
|  | 1 字节 | %r10b |
| Reg(R11) | 8 字节 | %r11 |
|  | 4 字节 | %r11d |
|  | 1 字节 | %r11b |
| Reg(R12) | 8 字节 | %r12 |
|  | 4 字节 | %r12d |
|  | 1 字节 | %r12b |
| Reg(R13) | 8 字节 | %r13 |
|  | 4 字节 | %r13d |
|  | 1 字节 | %r13b |
| Reg(R14) | 8 字节 | %r14 |
|  | 4 字节 | %r14d |
|  | 1 字节 | %r14b |
| Reg(R15) | 8 字节 | %r15 |
|  | 4 字节 | %r15d |
|  | 1 字节 | %r15b |
| Stack(int) | <int>(%rbp) |
| Imm(int) | $<int> |
| Data(identifier) | <identifier>(%rip) |

表 B-29 至 B-35 显示了完成 第一部分、第二部分 和 第三部分 后的完整代码输出过程。

表 B-29： 格式化顶级汇编构造

| 汇编顶级构造 |  | 输出 |
| --- | --- | --- |
| Program(top_levels) |

```
Print out each top-level construct.
On Linux, add at end of file: 
    .section .note.GNU-stack,"",@progbits
```

|

| Function(name, global, instructions) |
| --- |

```
 <global-directive>
 .text
 <name>:
     pushq    %rbp
     movq     %rsp, %rbp
 <instructions>
```

|

|

```
StaticVariable(name, global,
               alignment,
               init_list)
```

| 初始化为零的整数，或者仅用 ZeroInit 初始化的任何变量 |
| --- |

```
 <global-directive>
     .bss
     <alignment-directive>
 <name>:
     <init_list>
```

|

|  | 所有其他变量 |
| --- | --- |

```
 <global-directive>
     .data
     <alignment-directive>
 <name>:
     <init_list>
```

|

|

```
StaticConstant(name, alignment,
               init)
```

| Linux |
| --- |

```
 .section .rodata
     <alignment-directive>
 <name>:
     <init>
```

|

|  | macOS（8 字节对齐的数字常量） |
| --- | --- |

```
 .literal8
    .balign 8
 <name>:
     <init>
```

|

|  | macOS（16 字节对齐的数字常量） |
| --- | --- |

```
 .literal16
    .balign 16
 <name>:
     <init>  
     .quad 0
```

|

|  | macOS（字符串常量） |
| --- | --- |

```
 .cstring
 <name>:
     <init>
```

|

| 全局指令 |
| --- |

```
 If global is true:
 .globl <identifier>
 Otherwise, omit this directive.
```

|

| 对齐指令 | 仅限 Linux |
| --- | --- |

```
.align <alignment>
```

|

|  | macOS 或 Linux |
| --- | --- |

```
.balign <alignment>
```

|

表 B-30： 格式化静态初始化器

| 静态初始化器 | 输出 |
| --- | --- |
| CharInit(0) | .zero 1 |
| CharInit(i) | .byte <i> |
| IntInit(0) | .zero 4 |
| IntInit(i) | .long <i> |
| LongInit(0) | .zero 8 |
| LongInit(i) | .quad <i> |
| UCharInit(0) | .zero 1 |
| UCharInit(i) | .byte <i> |
| UIntInit(0) | .zero 4 |
| UIntInit(i) | .long <i> |
| ULongInit(0) | .zero 8 |
| ULongInit(i) | .quad <i> |
| ZeroInit(n) | .zero <n> |
| DoubleInit(d) |

```
.double <d>
or
.quad <d-interpreted-as-long>
```

|

| StringInit(s, True) | .asciz "<s>" |
| --- | --- |
| StringInit(s, False) | .ascii "<s>" |
| PointerInit(label) | .quad <label> |

表 B-31: 格式化汇编指令

| 汇编指令 | 输出 |
| --- | --- |
| Mov(t, src, dst) |

```
mov<t>   <src>, <dst>
```

|

| Movsx(src_t, dst_t, src, dst) |
| --- |

```
movs<src_t><dst_t>    <src>, <dst>
```

|

| MovZeroExtend(源类型, 目标类型, 源, 目标) |
| --- |

```
movz<src_t><dst_t>    <src>, <dst>
```

|

| Lea |
| --- |

```
leaq     <src>, <dst>
```

|

| Cvtsi2sd(t, 源, 目标) |
| --- |

```
cvtsi2sd<t>     <src>, <dst>
```

|

| Cvttsd2si(t, 源, 目标) |
| --- |

```
cvttsd2si<t>    <src>, <dst>  
```

|

| Ret |
| --- |

```
movq     %rbp, %rsp
popq     %rbp
ret
```

|

| Unary(一元操作符, t, 操数) |
| --- |

```
<unary_operator><t>     <operand>
```

|

| Binary(Xor, Double, 源, 目标) |
| --- |

```
xorpd    <src>, <dst>
```

|

| Binary(Mult, Double, 源, 目标) |
| --- |

```
mulsd    <src>, <dst>
```

|

| Binary(二元操作符, t, 源, 目标) |
| --- |

```
<binary_operator><t>    <src>, <dst>
```

|

| Idiv(t, 操数) |
| --- |

```
idiv<t>  <operand> 
```

|

| Div(t, 操数) |
| --- |

```
div<t>   <operand> 
```

|

| Cdq(长字) |
| --- |

```
cdq
```

|

| Cdq(四字) |
| --- |

```
cqo
```

|

| Push(操作数) |
| --- |

```
pushq    <operand>
```

|

| Pop(寄存器) |
| --- |

```
popq     <reg>
```

|

| Call(标签) |
| --- |

```
call     <label>
or
call     <label>@PLT
```

|

| Cmp(Double, 操数, 操数) |
| --- |

```
comisd   <operand>, <operand>
```

|

| Cmp(t, 操数, 操数) |
| --- |

```
cmp<t>  <operand>, <operand>
```

|

| Jmp(标签) |
| --- |

```
jmp      .L<label>
```

|

| JmpCC(条件码, 标签) |
| --- |

```
j<cond_code> .L<label>
```

|

| SetCC(条件码, 操数) |
| --- |

```
set<cond_code>  <operand>
```

|

| Label(标签) |
| --- |

```
.L<label>:
```

|

表 B-32： 汇编操作符的指令名称

| 汇编操作符 | 指令名称 |
| --- | --- |
| Neg | neg |
| Not | not |
| Shr | shr |
| Add | add |
| Sub | sub |
| Mult (仅限整数乘法) | imul |
| DivDouble | div |
| 与 | and |
| 或 | or |
| Shl | shl |
| ShrTwoOp | shr |

表 B-33： 汇编类型的指令后缀

| 汇编类型 | 指令后缀 |
| --- | --- |
| 字节 | b |
| 长字 | l |
| 四字 | q |
| 双精度 | sd |

表 B-34： 条件码的指令后缀

| 条件码 | 指令后缀 |
| --- | --- |
| E | e |
| 不等 | ne |
| L | l |
| LE | le |
| G | g |
| GE | ge |
| A | a |
| AE | ae |
| B | b |
| BE | be |

表 B-35： 汇编操作数的格式化

| 汇编操作数 | 输出 |
| --- | --- |
| 寄存器(AX) | 8 字节 | %rax |
|  | 4 字节 | %eax |
|  | 1 字节 | %al |
| 寄存器(DX) | 8 字节 | %rdx |
|  | 4 字节 | %edx |
|  | 1 字节 | %dl |
| 寄存器(CX) | 8 字节 | %rcx |
|  | 4 字节 | %ecx |
|  | 1 字节 | %cl |
| 寄存器(BX) | 8 字节 | %rbx |
|  | 4 字节 | %ebx |
|  | 1 字节 | %bl |
| 寄存器(DI) | 8 字节 | %rdi |
|  | 4 字节 | %edi |
|  | 1 字节 | %dil |
| 寄存器(SI) | 8 字节 | %rsi |
|  | 4 字节 | %esi |
|  | 1 字节 | %sil |
| 寄存器(R8) | 8 字节 | %r8 |
|  | 4 字节 | %r8d |
|  | 1 字节 | %r8b |
| Reg(R9) | 8 字节 | %r9 |
|  | 4 字节 | %r9d |
|  | 1 字节 | %r9b |
| Reg(R10) | 8 字节 | %r10 |
|  | 4 字节 | %r10d |
|  | 1 字节 | %r10b |
| Reg(R11) | 8 字节 | %r11 |
|  | 4 字节 | %r11d |
|  | 1 字节 | %r11b |
| Reg(R12) | 8 字节 | %r12 |
|  | 4 字节 | %r12d |
|  | 1 字节 | %r12b |
| Reg(R13) | 8 字节 | %r13 |
|  | 4 字节 | %r13d |
|  | 1 字节 | %r13b |
| Reg(R14) | 8 字节 | %r14 |
|  | 4 字节 | %r14d |
|  | 1 字节 | %r14b |
| Reg(R15) | 8 字节 | %r15 |
|  | 4 字节 | %r15d |
|  | 1 字节 | %r15b |
| Reg(SP) |  | %rsp |
| Reg(BP) |  | %rbp |
| Reg(XMM0) |  | %xmm0 |
| Reg(XMM1) |  | %xmm1 |
| Reg(XMM2) |  | %xmm2 |
| Reg(XMM3) |  | %xmm3 |
| Reg(XMM4) |  | %xmm4 |
| Reg(XMM5) |  | %xmm5 |
| Reg(XMM6) |  | %xmm6 |
| Reg(XMM7) |  | %xmm7 |
| Reg(XMM8) |  | %xmm8 |
| Reg(XMM9) |  | %xmm9 |
| Reg(XMM10) |  | %xmm10 |
| Reg(XMM11) |  | %xmm11 |
| Reg(XMM12) |  | %xmm12 |
| Reg(XMM13) |  | %xmm13 |
| Reg(XMM14) |  | %xmm14 |
| Reg(XMM15) |  | %xmm15 |
| Memory(reg, int) | <int>(<reg>) |
| 索引（reg1，reg2，int） | (<reg1>, <reg2>, <int>) |
| Imm（int） |  | $<int> |
| 数据（identifier，0） | <identifier>（%rip） |
| 数据（identifier，int） | <identifier>+<int>（%rip） |
