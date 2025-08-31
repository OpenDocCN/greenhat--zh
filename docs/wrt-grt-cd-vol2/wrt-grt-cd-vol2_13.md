# 第十三章：**控制结构和程序决策**

![image](img/common01.jpg)

控制结构是高级语言（HLL）编程的核心。根据给定条件的评估做出决策的能力，对于计算机提供的几乎所有自动化任务都是基础。高级语言控制结构转化为机器码的过程，可能对程序的性能和大小产生最大的影响。正如你将在本章中看到的，知道在特定情况下使用哪种控制结构是编写出色代码的关键。特别是，本章描述了与决策和无条件流相关的控制结构的机器实现，包括：

+   `if` 语句

+   `switch` 或 `case` 语句

+   `goto` 及相关语句

接下来的两章将扩展讨论循环控制结构及过程/函数调用与返回。

### 13.1 控制结构如何影响程序效率

程序中相当一部分机器指令控制着程序的执行路径。由于控制转移指令通常会清空指令流水线（参见 *WGC1*），它们往往比执行简单计算的指令要慢。为了生成高效的程序，你应该减少控制转移指令的数量，或者如果无法避免，选择最快的指令。

CPUs 用于控制程序流的指令集在不同处理器之间有所不同。然而，许多 CPU（包括本书所涵盖的五个家族）使用“比较与跳转”范式来控制程序流。也就是说，在进行比较或其他修改 CPU 标志的指令后，条件跳转指令会根据 CPU 标志设置将控制权转移到另一个位置。有些 CPU 可以用单条指令完成所有这些操作，而其他 CPU 可能需要两条、三条或更多指令。有些 CPU 允许你对两个值进行广泛的不同条件比较，而其他 CPU 仅允许进行少数几种测试。不论使用何种机制，映射到某一 CPU 上给定指令序列的高级语言（HLL）语句，在第二个 CPU 上也会映射到一个相似的序列。因此，如果你理解了某个 CPU 上的基本转换方式，就能较好地理解编译器如何在所有 CPU 上工作。

### 13.2 低级控制结构简介

大多数 CPU 使用两步过程来做程序决策。首先，程序比较两个值，并将比较结果保存在机器寄存器或标志中。然后，程序测试该结果，并根据结果将控制权转移到两个位置之一。通过这种*比较与条件分支*序列，几乎可以合成大多数主要的高级语言控制结构。

即使在比较和条件分支范式中，CPU 通常也使用两种不同的方法来实现条件代码序列。一个技术，尤其在基于堆栈的架构（如 UCSD p-machine、Java 虚拟机和 Microsoft CLR）中常见，是使用不同形式的比较指令来测试特定条件。例如，你可能会有 *相等比较*、*不相等比较*、*小于比较*、*大于比较* 等等。每个比较的结果是一个布尔值。然后，一对条件分支指令，*条件成立时分支* 和 *条件不成立时分支*，可以测试比较的结果，并将控制转移到适当的位置。这些虚拟机可能实际上将比较和分支指令合并为“比较和分支”指令（每个条件一个）。尽管使用了更少的指令，但最终结果完全相同。

第二种，也是历史上更为流行的方法，是让 CPU 的指令集包含一个单一的比较指令，该指令设置（或清除）CPU 的 *程序状态* 或 *标志* 寄存器中的多个位。然后，程序使用几种更具体的条件分支指令之一，将控制转移到其他位置。这些条件分支指令可能有如 *相等跳转*、*不相等跳转*、*小于跳转* 或 *大于跳转* 等名称。由于这种“比较和跳转”技术是 80x86、ARM 和 PowerPC 所使用的，因此本章的示例也采用了这种方法；然而，将其转换为多重比较/跳转真/跳转假范式是很容易的。

ARM 处理器的 32 位变体引入了第三种技术：条件执行。大多数指令（不仅仅是分支指令）在 32 位 ARM 上都提供了这个选项。例如，`addeq` 指令仅在先前的比较（或其他操作）结果设置了零标志时，才会执行两个值的加法。更多详细信息，请参见附录 C 中的《指令的条件后缀》。

条件分支通常是双向分支。也就是说，如果它们测试的条件为 `true`，则将控制转移到程序中的一个位置；如果条件为 `false`，则转移到另一个位置。为了减少指令的大小，大多数 CPU 上的条件分支仅编码两个可能分支位置中的一个地址，并且对于相反条件使用隐式地址。具体来说，大多数条件分支如果条件为 `true`，则将控制转移到目标位置，如果条件为 `false`，则跳过到下一条指令。例如，考虑以下 80x86 的 `je`（相等跳转）指令序列：

```

			// Compare the value in EAX to the value in EBX

        cmp( eax, ebx );

// Branch to label EAXequalsEBX if EAX==EBX

        je EAXequalsEBX;

        mov( 4, ebx );      // Drop down here if EAX != EBX
            .
            .
            .
EAXequalsEBX:
```

该指令序列首先通过比较 EAX 寄存器中的值与 EBX 寄存器中的值（`cmp`指令）；这会设置 80x86 EFLAGS 寄存器中的*条件码位*。特别地，如果 EAX 中的值等于 EBX 中的值，则该指令将 80x86 零标志设置为`1`。`je`指令测试零标志是否被设置，如果是，它将控制转移到紧随`EAXequalsEBX`标签之后的机器指令。如果 EAX 中的值不等于 EBX，则`cmp`指令会清除零标志，`je`指令会直接跳过到`mov`指令，而不会转移控制到目标标签。

某些访问数据的机器指令，如果所访问的内存位置靠近包含该变量的激活记录的基址，则可能会更小（且更快）。此规则同样适用于条件跳转指令。80x86 提供了两种形式的条件跳转指令。一种形式仅为 2 字节长（1 字节用于操作码，1 字节用于范围从-128 到+127 的有符号位移）。另一种形式为 6 字节长（2 字节用于操作码，4 字节用于范围从-20 亿到+20 亿的有符号位移）。位移值指定程序需要跳转多少字节才能到达目标位置。为了将控制转移到附近的位置，程序可以使用短形式的跳转。由于 80x86 指令的长度在 1 到 15 字节之间（通常约为 3 或 4 字节），条件跳转指令的短形式通常可以跳过大约 32 到 40 条机器指令。一旦目标位置超出了±127 字节的范围，这些条件跳转指令的 6 字节版本将扩展跳转范围到当前指令的±20 亿字节。如果你有兴趣编写最有效的代码，那么你会尽量多使用 2 字节形式。

分支在现代（流水线）CPU 中是一个昂贵的操作，因为分支可能要求 CPU 刷新流水线并重新加载（更多细节见*WGC1*）。条件分支只有在分支被执行时才会产生这种成本；如果条件分支指令跳过到下一条指令，CPU 将继续使用流水线中找到的指令，而不会刷新它们。因此，在许多系统中，*跳过到下一条指令的分支比执行跳转的分支更快*。然而，请注意，一些 CPU（如 80x86、PowerPC 和 ARM）支持*分支预测*功能，这可以告诉 CPU 从分支目标位置开始获取指令进入流水线，而不是从紧随其后的条件跳转指令后获取。不幸的是，分支预测算法因处理器而异（甚至在 80x86 系列 CPU 中也有所不同），因此通常难以预测分支预测会如何影响你的高级语言代码。除非你正在为特定的处理器编写代码，否则最安全的假设是，跳过到下一条指令比执行跳转更有效。

虽然比较和条件分支范式是机器代码程序中最常见的控制结构，但也有其他方法可以基于某些计算结果将控制权转移到内存中的另一个位置。毫无疑问，间接跳转（尤其是通过地址表）是最常见的替代形式。考虑以下 32 位 80x86 的`jmp`指令：

```

			readonly
    jmpTable: dword[4] := [&label1, &label2, &label3, &label4];
            .
            .
            .
        jmp( jmpTable[ ebx*4 ] );
```

这个`jmp`指令从`jmpTable`数组中，由 EBX 中的值指定的索引处获取双字值。也就是说，该指令根据 EBX 中的值（`0..3`）将控制权转移到四个不同的位置。例如，如果 EBX 的值为`0`，那么`jmp`指令将从`jmpTable`的索引`0`处获取双字（即由`label1`前缀的指令的地址）。同样地，如果 EBX 的值为`2`，那么该`jmp`指令将从该表中获取第三个双字（即程序中`label3`的地址）。这大致等同于，但通常比以下指令序列更简短：

```

			cmp( ebx, 0 );
je label1;
cmp( ebx, 1 );
je label2;
cmp( ebx, 2 );
je label3;
cmp( ebx, 3 );
je label4;

// Results are undefined if EBX <> 0, 1, 2, or 3
```

一些其他的条件控制转移机制在不同的 CPU 上也可以使用，但这两种机制（比较和条件分支以及间接跳转）是大多数高级语言编译器用来实现标准控制结构的方式。

### 13.3 goto 语句

`goto`语句也许是最基本的低级控制结构。自从 1960 年代末和 1970 年代“结构化编程”浪潮以来，它在高级语言代码中的使用逐渐减少。事实上，一些现代高级编程语言（例如 Java 和 Swift）甚至不提供非结构化的`goto`语句。即使在提供`goto`语句的语言中，编程风格指南通常也会限制其仅在特殊情况下使用。再加上自 1970 年代中期以来，学生程序员被严格教导在程序中避免使用`goto`，因此在现代程序中很少能找到`goto`语句。从可读性的角度来看，这是件好事（可以查看一些 1960 年代的 FORTRAN 程序，了解当代码中充斥着`goto`语句时，代码有多难以阅读）。尽管如此，仍有一些程序员认为，通过在代码中使用`goto`语句，他们可以实现更高的效率。虽然有时这种说法是对的，但效率的提升往往不值得牺牲可读性。

`goto`的一个重要效率论点是它有助于避免重复代码。考虑以下简单的 C/C++示例：

```

			if( a == b || c < d )
{
    << execute some number of statements >>

    if( x == y )
    {
        << execute some statements if x == y >>
    }
    else
    {
        << execute some statements if x != y >>
    }
}
else
{
    << execute the same sequence of statements
       that the code executes if x!= y in the
       previous else section >>
}
```

寻找提高程序效率方法的程序员会立即注意到所有重复的代码，并可能会被诱使将示例重写如下：

```

			if( a == b || c < d )
{
    << execute some number of statements >>

    if( x != y ) goto DuplicatedCode;

    << execute some statements if x == y >>
}
else
{
DuplicatedCode:
    << execute the same sequence of statements
       if x != y or the original
       Boolean expression is false >>
}
```

当然，这段代码存在一些软件工程问题，包括它比原始示例稍难阅读、修改和维护。（你*可以*辩称它实际上更容易维护，因为你不再有重复的代码，只需要在一个地方修复公共代码中的缺陷。）然而，不能否认的是，这个示例中的代码量确实较少。或者说，真的少吗？

许多现代编译器中的优化器实际上会寻找像第一个示例中的代码序列，并生成与第二个示例预期完全相同的代码。因此，一个*优秀*的编译器即使源文件中存在重复，也会避免生成重复的机器代码，正如第一个示例中的情况。

考虑以下 C/C++示例：

```

			#include <stdio.h>

static int a;
static int b;

extern int x;
extern int y;
extern int f( int );
extern int g( int );
int main( void )
{
    if( a==f(x))
    {
        if( b==g(y))
        {
            a=0;
        }
        else
        {
            printf( "%d %d\n", a, b );
            a=1;
            b=0;
        }
    }
    else
    {
        printf( "%d %d\n", a, b );
        a=1;
        b=0;
    }

    return( 0 );
}
```

这是 GCC 将`if`语句序列编译成 PowerPC 代码的过程：

```

			        ; f(x):

        lwz r3,0(r9)
        bl L_f$stub

        ; Compute a==f(x), jump to L2 if false

        lwz r4,0(r30)
        cmpw cr0,r4,r3
        bne+ cr0,L2

        ; g(y):

        addis r9,r31,ha16(L_y$non_lazy_ptr-L1$pb)
        addis r29,r31,ha16(_b-L1$pb)
        lwz r9,lo16(L_y$non_lazy_ptr-L1$pb)(r9)
        la r29,lo16(_b-L1$pb)(r29)
        lwz r3,0(r9)
        bl L_g$stub

        ; Compute b==g(y), jump to L3 if false:

        lwz r5,0(r29)
        cmpw cr0,r5,r3
        bne- cr0,L3

        ; a=0
        li r0,0
        stw r0,0(r30)
        b L5

        ; Set up a and b parameters if
        ; a==f(x) but b!=g(y):

L3:
        lwz r4,0(r30)
        addis r3,r31,ha16(LC0-L1$pb)
        b L6

        ; Set up parameters if a!=f(x):
L2:
        addis r29,r31,ha16(_b-L1$pb)
        addis r3,r31,ha16(LC0-L1$pb)
        la r29,lo16(_b-L1$pb)(r29)
        lwz r5,0(r29)

        ; Common code shared by both
        ; ELSE sections:
L6:
        la r3,lo16(LC0-L1$pb)(r3) ; Call printf
        bl L_printf$stub
        li r9,1                 ; a=1
        li r0,0                 ; b=0
        stw r9,0(r30)           ; Store a
        stw r0,0(r29)           ; Store b
L5:
```

当然，并非每个编译器都有优化器能够识别重复代码。因此，如果你想编写一个无论编译器如何都能编译成高效机器代码的程序，你可能会倾向于使用带有`goto`语句的代码版本。实际上，你可以提出一个强有力的软件工程论点，即源代码中的重复代码使得程序更难以阅读和维护。（如果你在某个代码副本中修复了一个缺陷，可能会忘记在其他副本中修复相同的缺陷。）虽然这确实是事实，但如果你在目标标签处对代码进行修改，那么是否每个跳转到该目标标签的代码段都适合这种更改并不立刻显现。而且，在阅读源代码时，也不容易立刻看出有多少个`goto`语句将控制转移到同一个目标标签。

传统的软件工程方法是将公共代码放入一个过程或函数中，然后简单地调用该函数。然而，函数调用和返回的开销可能相当大（尤其是在没有太多重复代码的情况下），因此从性能角度来看，这种方法可能不尽如人意。对于短小的公共代码序列，创建宏或内联函数可能是最佳解决方案。更复杂的是，你可能需要对仅影响一个实例的重复代码进行更改（也就是说，它不再是重复的）。总的来说，以这种方式使用`goto`语句来提高效率应当是你的最后手段。

另一个常见的`goto`语句用法是处理异常情况。当你发现自己深深地嵌套在多个语句中，并且遇到一种需要退出所有这些语句的情况时，普遍共识是，如果重构代码不会使其更加可读，那么使用`goto`是可以接受的。然而，从嵌套块跳出可能会妨碍优化器为整个过程或函数生成良好代码的能力。`goto`语句的使用可能会在它直接影响的代码中节省一些字节或处理器周期，但它也可能对函数的其余部分产生不利影响，导致整体代码效率降低。因此，在代码中插入`goto`语句时要小心——它们可能会使源代码更难以阅读，并且可能最终使其效率降低。

如果你觉得有用，下面是一个可以用来解决原始问题的编程技巧。考虑对代码进行如下修改：

```

			switch( a == b || c < d )
{
    case 1:
        << execute some number of statements >>

        if( x == y )
        {
            << execute some statements if x == y >>
            break;
        }
        // Fall through if x != y

    case 0:

        << execute some statements if x!= y or
            if !( a == b || c < d )  >>
}
```

当然，这段代码比较棘手，而棘手的代码通常不是优秀的代码。然而，它确实有一个好处，就是避免了程序中源代码的重复。

#### 13.3.1 限制形式的 goto 语句

为了支持结构化的“无 goto”编程，许多编程语言添加了受限形式的 `goto` 语句，允许程序员立即退出控制结构，如循环或过程/函数。典型的语句包括 `break` 和 `exit`，它们会跳出包围的循环；`continue`、`cycle` 和 `next`，它们会重新启动包围的循环；以及 `return` 和 `exit`，它们会立即从包围的过程或函数中返回。这些语句比标准的 `goto` 更具结构性，因为程序员并不选择跳转目标；相反，控制会转移到一个固定的位置，这个位置取决于包含该语句的控制语句（或函数或过程）。

几乎所有这些语句都会编译为单个 `jmp` 指令。那些跳出循环的语句（如 `break`）会编译为一个 `jmp` 指令，将控制权转移到循环底部之后的第一个语句。那些重新启动循环的语句（例如，`continue`、`next` 或 `cycle`）会编译为一个 `jmp` 指令，将控制权转移到循环终止测试（对于 `while` 或 `repeat..until`/`do..while`）或循环顶部（对于其他大多数循环）。

然而，仅仅因为这些语句通常会编译为单个 `jmp` 指令，并不意味着它们使用起来高效。即使忽略了 `jmp` 可能比较昂贵的事实（因为它会迫使 CPU 清空指令流水线），从循环中分支出来的语句可能会对编译器的优化器产生严重影响，显著降低生成高质量代码的机会。因此，您应该尽可能节省使用这些语句。

### 13.4 if 语句

也许最基本的高级控制结构就是 `if` 语句。事实上，仅凭 `if` 和 `goto` 语句，您就可以（语义上）实现所有其他控制结构。^(1) 我们将在讨论其他控制结构时再次提到这一点，但现在我们先来看一下典型编译器是如何将 `if` 语句转换为机器代码的。

为了实现一个简单的 `if` 语句，它比较两个值并在条件为 `true` 时执行主体，您可以使用单个比较和条件跳转指令。考虑下面的 Pascal `if` 语句：

```

			if( EAX = EBX ) then begin

    writeln( 'EAX is equal to EBX' );
    i := i + 1;

end;
```

下面是转换为 80x86/HLA 代码的示例：

```

			    cmp( EAX, EBX );
    jne skipIfBody;
    stdout.put( "EAX is equal to EBX", nl );
    inc( i );
skipIfBody:
```

在 Pascal 源代码中，`if` 语句的主体会在 EAX 的值等于 EBX 时执行。在生成的汇编代码中，程序会比较 EAX 和 EBX，然后，如果 EAX 不等于 EBX，跳过对应于 `if` 语句主体的语句。这是将高层语言 `if` 语句转换为机器代码的“模板”：测试某个条件，如果条件为 `false`，则跳过 `if` 语句的主体。

`if..then..else` 语句的实现比基本的 `if` 语句稍微复杂一些。`if..then..else` 语句通常采用如下的语法和语义：

```

			if( some_boolean_expression ) then

    << Statements to execute if the expression is true >>

else

    << Statements to execute if the expression is false >>

endif
```

在机器代码中实现这一代码序列，只需比简单的 `if` 语句多一个机器指令。考虑以下 C/C++ 示例代码：

```

			if( EAX == EBX )
{
    printf( "EAX is equal to EBX\n" );
    ++i;
}
else
{
    printf( "EAX is not equal to EBX\n" );
}
```

下面是转换成 80x86 汇编语言代码：

```

			    cmp( EAX, EBX );        // See if EAX == EBX
    jne doElse;             // Branch around "then" code
    stdout.put( "EAX is equal to EBX", nl );
    inc( i );
    jmp skipElseBody;        // Skip over "else" section.

// if they are not equal.

doElse:
    stdout.put( "EAX is not equal to EBX", nl );

skipElseBody:
```

这段代码有两点需要注意。首先，如果条件计算结果为 `false`，代码将跳转到 `else` 块的第一条语句，而不是跳转到（整个）`if` 语句后的第一条语句。第二点需要注意的是，在 `true` 条件语句末尾的 `jmp` 指令跳过了 `else` 块。

一些语言，包括 HLA，支持 `if` 语句中的 `elseif` 子句，当第一个条件失败时，它会评估第二个条件。这是对我所展示的 `if` 语句代码生成的一个直接扩展。考虑以下 HLA `if..elseif..else..endif` 语句：

```

			if( EAX = EBX ) then

    stdout.put( "EAX is equal to EBX" nl );
    inc( i );

elseif( EAX = ECX ) then

    stdout.put( "EAX is equal to ECX" nl );

else

    stdout.put( "EAX is not equal to EBX or ECX" nl);

endif;
```

下面是转换成纯 80x86/HLA 汇编语言代码：

```

			// Test to see if EAX = EBX

    cmp( eax, ebx );
    jne tryElseif;    // Skip "then" section if equal

    // Start of the "then" section

    stdout.put( "EAX is equal to EBX", nl );
    inc( i );
    jmp skipElseBody  // End of "then" section, skip
                      // over the elseif clause.
tryElseif:
    cmp( eax, ecx );  // ELSEIF test for EAX = ECX
    jne doElse;       // Skip "then" clause if not equal

    // elseif "then" clause

    stdout.put( "EAX is equal to ECX", nl );
    jmp skipElseBody; // Skip over the "else" section

doElse: // else clause begins here
    stdout.put( "EAX is not equal to EBX or ECX", nl );

skipElseBody:
```

`elseif` 子句的翻译非常直接；它的机器码与 `if` 语句是相同的。这里值得注意的是，编译器如何在 `if..then` 子句末尾发出 `jmp` 指令，以跳过为 `elseif` 子句发出的布尔测试。

#### 13.4.1 提高某些 if/else 语句的效率

从效率角度来看，需要注意的是，`if..else` 语句中没有路径是不会涉及控制转移的（与简单的 `if` 语句不同，如果条件表达式为 `true`，它会直接跳过）。正如本章所指出的，分支是有问题的，因为它们通常会刷新 CPU 的指令流水线，重新填充需要几个 CPU 周期。如果布尔表达式的两个结果（`true` 和 `false`）的可能性相等，那么通过重新安排 `if..else` 语句来提高代码的性能几乎没有什么可做的。然而，对于大多数 `if` 语句来说，一个结果往往比另一个更可能——甚至可能大大更可能——出现。理解一种比较比另一种比较更可能的汇编程序员通常会按如下方式编码他们的 `if..else` 语句：

```

			// if( eax == ebx ) then
//    //<likely case>
//    stdout.put( "EAX is equal to EBX", nl );
// else
//    // unlikely case
//    stdout.put( "EAX is not equal to EBX" nl );
// endif;

    cmp( EAX, EBX );
    jne goDoElse;
    stdout.put( "EAX is equal to EBX", nl );
backFromElse:
        .
        .
        .
// Somewhere else in the code (not in the direct path of the above):

goDoElse:
    stdout.put( "EAX is not equal to EBX", nl );
    jmp backFromElse
```

请注意，在最常见的情况下（即表达式求值为`true`时），代码会直接跳转到`then`部分，然后继续执行`if`语句后面的代码。因此，如果布尔表达式（`eax == ebx`）大部分时间为`true`，这段代码会毫不分支地直接执行。在极少数情况下，当 EAX 不等于 EBX 时，程序实际上需要执行两次分支：一次转移控制到处理`else`子句的代码段，另一次将控制权返回到`if`后面的第一条语句。只要这种情况发生的频率不到一半，软件的整体性能就会得到提升。你可以在像 C 这样的高级语言中通过`goto`语句实现相同的结果。例如：

```

			if( eax != ebx ) goto doElseStuff;

    // << body of the if statement goes here>>
    // (statements between then and else)
endOfIF:
// << statements following the if..endif statement >>
    .
    .
    .
// Somewhere outside the direct execution path of the above

doElseStuff:
    << Code to do if the expression is false >>
    goto endOfIF;
```

当然，这种方案的缺点是它产生了*意大利面条代码*，一旦你加入多个这种权宜之计，它就变得难以阅读。汇编语言程序员可以使用这种方式，因为大多数汇编语言代码本质上就是意大利面条代码。^(2) 然而，对于高级语言（HLL）代码而言，这种编程风格通常是不可接受的，只有在必要时才应使用它。（请参见“`goto`语句”在第 455 页的内容。）

以下是一个在高级语言（如 C）中常见的通用`if`语句：

```

			if( eax == ebx )
{
    // Set i to some value along this execution path.

    i = j+5;
}
else
{
    // Set i to a different value along this path

    i = 0;
}
```

这是将此 C 代码转换为 80x86/HLA 汇编代码的形式：

```

			        cmp( eax, ebx );
        jne doElse;
        mov( j, edx );
        add( 5, edx );
        mov( edx, i );
        jmp ifDone;

doElse:
        mov( 0, i );
ifDone:
```

正如你在前面的示例中看到的，`if..then..else`语句的汇编语言转换需要两条控制转移指令：

+   测试 EAX 和 EBX 之间比较的`jne`指令

+   无条件的`jmp`指令跳过`if`语句中的`else`部分

无论程序采取哪条路径（通过`then`部分还是`else`部分），CPU 都会执行一个慢速的分支指令，最终导致指令流水线被刷新。考虑以下代码，它没有这个问题：

```

			i = 0;
if( eax == ebx )
{
    i = j + 5;
}
```

这是它转换为纯 80x86/HLA 汇编代码的形式：

```

			        mov( 0, i );
        cmp( eax, ebx );
        jne skipIf;
        mov( j, edx );
        add( 5, edx );
        mov( edx, i );
skipIf:
```

正如你所见，如果表达式求值为`true`，CPU 根本不会执行任何控制转移指令。是的，CPU 执行了一条额外的`mov`指令，其结果会立即被覆盖（因此第一次`mov`指令的执行是浪费的）；然而，这条额外的`mov`指令的执行速度远快于`jmp`指令的执行。这个技巧是一个典型例子，说明了为什么了解一些汇编语言代码（以及了解编译器如何从高级语言代码生成机器代码）是一个好主意。第二个序列比第一个更优这一点并不明显。事实上，初学者可能会认为它更差，因为当表达式求值为`true`时，程序“浪费”了对`i`的赋值（而第一个版本没有进行这样的赋值）。这也是本章存在的原因之一——确保你理解使用高级控制结构所涉及的成本。

#### 13.4.2 强制在 if 语句中进行完全布尔求值

因为完全布尔求值和短路布尔求值可能会产生不同的结果（参见 第 441 页的“短路求值”），因此在计算布尔表达式的结果时，有时需要强制代码使用其中一种形式。

强制完全布尔求值的一般方法是评估表达式的每个子组件，并将子结果存储到临时变量中。然后，你可以在计算完这些临时结果后，将它们组合成完整的结果。例如，考虑以下 Pascal 代码片段：

```

			if( (i < g(y)) and (k > f(x)) ) then begin

    i := 0;

end;
```

因为 Pascal 不保证完全布尔求值，函数 `f()` 可能不会在此表达式中被调用——如果 `i` 小于 `g(y)`——因此，调用 `f()` 可能产生的副作用可能不会发生。（参见 第 430 页的“算术表达式中的副作用”）。如果应用程序的逻辑依赖于 `f()` 和 `g()` 的调用产生的副作用，则必须确保应用程序调用这两个函数。请注意，简单地交换 AND 运算符两边的子表达式不足以解决这个问题；通过这种修改，应用程序可能不会调用 `g()`。

解决这个问题的一种方法是使用单独的赋值语句计算两个子表达式的布尔结果，然后在 `if` 表达式中计算这两个结果的逻辑与：

```

			lexpr := i < g(y);
rexpr := k > f(x);
if( lexpr AND rexpr ) then begin

    i := 0;

end;
```

不必过于担心使用这些临时变量可能导致的效率损失。任何提供优化功能的编译器都会将这些值放入寄存器，而不会使用实际的内存位置。考虑以下用 C 语言编写并通过 Visual C++ 编译器编译的 Pascal 程序变体：

```

			#include <stdio.h>

static int i;
static int k;

extern int x;
extern int y;
extern int f( int );
extern int g( int );

int main( void )
{
    int lExpr;
    int rExpr;

    lExpr = i < g(y);
    rExpr = k > f(x);
    if( lExpr && rExpr )
    {
        printf( "Hello" );
    }

    return( 0 );
}
```

以下是 Visual C++ 编译器转换为 32 位 MASM 代码的结果（为了更清楚地表达意图，几条指令已经重新排列）：

```

			main    PROC

$LN7:
        mov     QWORD PTR [rsp+8], rbx
        push    rdi
        sub     rsp, 32                                 ; 00000020H

; eax = g(y)
        mov     ecx, DWORD PTR y
        call    g
; ebx (lExpr) = i < g(y)
        xor     edi, edi
        cmp     DWORD PTR i, eax
        mov     ebx, edi ; ebx = 0
        setl    bl ;if i < g(y), set EBX to 1.

; eax = f(x)
        mov     ecx, DWORD PTR x
        call    f

; EDI = k > f(x)

        cmp     DWORD PTR k, eax
        setg    dil ; Sets EDI to 1 if k > f(x)

; See if lExpr is false:

        test    ebx, ebx
        je      SHORT $LN4@main

; See if rExpr is false:

        test    edi, edi
        je      SHORT $LN4@main

; "then" section of the if statement:

        lea     rcx, OFFSET FLAT:$SG7893
        call    printf
 $LN4@main:

; return(0);
        xor     eax, eax

        mov     rbx, QWORD PTR [rsp+48]
        add     rsp, 32                                 ; 00000020H
        pop     rdi
        ret     0
main    ENDP
```

如果你查看汇编代码，你会发现这段代码片段始终执行对 `f()` 和 `g()` 的调用。与以下 C 代码和汇编输出对比：

```

			#include <stdio.h>

static int i;
static int k;

extern int x;
extern int y;
extern int f( int );
extern int g( int );

int main( void )
{
    if( i < g(y) && k > f(x) )
    {
        printf( "Hello" );
    }

    return( 0 );
}
```

以下是 MASM 汇编输出：

```

			main    PROC

$LN7:
        sub     rsp, 40                                 ; 00000028H

; if (!(i < g(y))) then bail on the rest of the code:

        mov     ecx, DWORD PTR y
        call    g
        cmp     DWORD PTR i, eax
        jge     SHORT $LN4@main

; if (!(k > f(x))) then skip printf:

        mov     ecx, DWORD PTR x
        call    f
        cmp     DWORD PTR k, eax
        jle     SHORT $LN4@main

; Here's the body of the if statement.

        lea     rcx, OFFSET FLAT:$SG7891
        call    printf
$LN4@main:

; return 0
        xor     eax, eax

        add     rsp, 40                                 ; 00000028H
        ret     0
main    ENDP
```

在 C 语言中，你可以使用另一种技巧来强制在任何布尔表达式中进行完全的布尔求值。C 语言的位运算符不支持短路布尔求值。如果你的布尔表达式中的子表达式始终产生 `0` 或 `1`，那么位运算布尔与（`&`）和布尔或（`|`）运算符的结果与逻辑布尔运算符（`&&` 和 `||`）产生的结果是相同的。考虑以下 C 代码和 Visual C++ 编译器生成的 MASM 代码：

```

			#include <stdio.h>

static int i;
static int k;

extern int x;
extern int y;
extern int f( int );
extern int g( int );

int main( void )
{
    if( i < g(y) & k > f(x) )
    {
        printf( "Hello" );
    }
    return( 0 );
}
```

以下是 Visual C++ 生成的 MASM 代码：

```

			main    PROC

$LN6:
        mov     QWORD PTR [rsp+8], rbx
        push    rdi
        sub     rsp, 32                                 ; 00000020H

        mov     ecx, DWORD PTR x
        call    f
        mov     ecx, DWORD PTR y
        xor     edi, edi
        cmp     DWORD PTR k, eax
        mov     ebx, edi
        setg    bl
        call    g
        cmp     DWORD PTR i, eax
        setl    dil
        test    edi, ebx
        je      SHORT $LN4@main

        lea     rcx, OFFSET FLAT:$SG7891
        call    printf
$LN4@main:

        xor     eax, eax

        mov     rbx, QWORD PTR [rsp+48]
        add     rsp, 32                                 ; 00000020H
        pop     rdi
        ret     0
main    ENDP
```

注意位运算符的使用如何生成与早期使用临时变量的代码段相似的代码。这会减少原始 C 源文件中的杂乱。

然而，值得记住的是，C 的按位运算符只有在操作数为`0`和`1`时，才会产生与逻辑运算符相同的结果。幸运的是，你可以在这里使用一个小技巧：只需写`!!(expr)`，如果表达式的值是零或非零，C 将把结果转换为`0`或`1`。为了演示这一点，考虑以下 C/C++代码片段：

```

			#include <stdlib.h>
#include <math.h>
#include <stdio.h>

int main( int argc, char **argv )
{
    int boolResult;

    boolResult = !!argc;
    printf( "!!(argc) = %d\n", boolResult );
    return 0;
}
```

这是微软 Visual C++编译器为此短程序生成的 80x86 汇编代码：

```

			main    PROC
$LN4:
        sub     rsp, 40      ; 00000028H

        xor     edx, edx     ; EDX = 0
        test    ecx, ecx     ; System passes ARGC in ECX register
        setne   dl           ; If ECX==0, sets EDX=1, else EDX=0

        lea     rcx, OFFSET FLAT:$SG7886 ; Zero flag unchanged!
        call    printf       ; printf parm1 in RCX, parm2 in EDX

; Return 0;
        xor     eax, eax

        add     rsp, 40                   ; 00000028H
        ret     0
main    ENDP
```

正如你在 80x86 汇编输出中所看到的，只需要三条机器指令（不涉及昂贵的分支操作）就能将零/非零转换为`0`/`1`。

#### 13.4.3 在 if 语句中强制短路求值

虽然偶尔强制完全布尔求值是有用的，但强制短路求值的需求可能更为常见。考虑以下 Pascal 语句：

```

			if( (ptrVar <> NIL) AND (ptrVar^ < 0) ) then begin

    ptrVar^ := 0;

end;
```

Pascal 语言的定义将是否使用完全的布尔求值还是短路求值留给编译器编写者决定。实际上，编写者可以根据需要自由选择两种方案。因此，完全有可能同一个编译器在代码的一个部分使用完全布尔求值，而在另一个部分使用短路求值。

你可以看到，如果`ptrVar`包含 NIL 指针值，并且编译器使用完全布尔求值，这个布尔表达式将失败。要使此语句正确工作，唯一的方法就是使用短路布尔求值。

使用 AND 运算符模拟短路布尔求值实际上非常简单。你所需要做的就是创建一对嵌套的`if`语句，并将每个子表达式分别放入其中。例如，你可以通过以下方式在当前 Pascal 示例中保证短路布尔求值：

```

			if( ptrVar <> NIL ) then begin

    if( ptrVar^ < 0 ) then begin

        ptrVar^ := 0;
 end;

end;
```

这条语句在语义上与前一个相同。应该很清楚，如果第一个表达式求值为`false`，第二个子表达式将不会执行。尽管这种方法会让源文件稍显冗杂，但它确实能保证无论编译器是否支持该方案，都会进行短路求值。

处理逻辑“或”操作要稍微复杂一些。如果左操作数求值为`true`，则需要额外的测试来保证右操作数不执行。考虑以下 C 代码（记住，C 默认支持短路求值）：

```

			#include <stdio.h>

static int i;
static int k;

extern int x;
extern int y;
extern int f( int );
extern int g( int );

int main( void )
{
    if( i < g(y) || k > f(x) )
    {
        printf( "Hello" );
    }

    return( 0 );
}
```

这是微软 Visual C++编译器生成的机器代码：

```

			main    PROC

$LN8:
        sub     rsp, 40             ; 00000028H

        mov     ecx, DWORD PTR y
        call    g
        cmp     DWORD PTR i, eax
        jl      SHORT $LN3@main
        mov     ecx, DWORD PTR x
        call    f
        cmp     DWORD PTR k, eax
        jle     SHORT $LN6@main
$LN3@main:

        lea     rcx, OFFSET FLAT:$SG6880
        call    printf
$LN6@main:

        xor     eax, eax

        add     rsp, 40              ; 00000028H
        ret     0
main    ENDP
_TEXT   ENDS
```

这是一个 C 程序版本，它实现了短路求值，而不依赖 C 编译器的实现（值得注意的是，C 的语言定义保证了短路求值，但你可以在任何语言中使用这种方法）：

```

			#include <stdio.h>

static int i;
static int k;

extern int x;
extern int y;
extern int f( int );
extern int g( int );

int main( void )
{
    int temp;

        // Compute left subexpression and
        // save.

    temp = i < g(y);

        // If the left subexpression
        // evaluates to false, then try
        // the right subexpression.

    if( !temp )
    {
        temp = k > f(x);
    }

        // If either subexpression evaluates
        // to true, then print "Hello"

    if( temp )
    {
        printf( "Hello" );
    }

    return( 0 );
}
```

这是微软 Visual C++编译器为此短程序生成的相应 MASM 代码：

```

			main    PROC

$LN9:
        sub     rsp, 40         ; 00000028H

        mov     ecx, DWORD PTR y
        call    g
        xor     ecx, ecx
        cmp     DWORD PTR i, eax
        setl    cl
        test    ecx, ecx

        jne     SHORT $LN7@main

        mov     ecx, DWORD PTR x
        call    f
        xor     ecx, ecx
        cmp     DWORD PTR k, eax
        setg    cl
        test    ecx, ecx

        je      SHORT $LN5@main
$LN7@main:

        lea     rcx, OFFSET FLAT:$SG6881
        call    printf
$LN5@main:

        xor     eax, eax

        add     rsp, 40            ; 00000028H
        ret     0
main    ENDP
```

如你所见，编译器为第二版例程（手动强制短路求值）生成的代码不如 C 编译器为第一个示例生成的代码那样好。然而，如果你需要短路求值的语义以确保程序正确执行，那么你只能接受比编译器直接支持这种方案时生成的代码效率低的情况。

如果速度、最小化大小和短路求值都是必要的，并且你愿意牺牲代码的可读性和可维护性来实现这些目标，那么你可以解构代码，生成类似于 C 编译器通过短路求值生成的代码。请看以下 C 代码及其生成的输出：

```

			#include <stdio.h>

static int i;
static int k;

extern int x;
extern int y;
extern int f( int );
extern int g( int );

int main( void )
{
    if( i < g(y)) goto IntoIF;
    if( k > f(x) )
    {
      IntoIF:

        printf( "Hello" );
    }

    return( 0 );
}
```

这是 Visual C++的 MASM 输出：

```

			main    PROC

$LN8:
        sub     rsp, 40         ; 00000028H

        mov     ecx, DWORD PTR y
        call    g
        cmp     DWORD PTR i, eax
        jl      SHORT $IntoIF$9

        mov     ecx, DWORD PTR x
        call    f
        cmp     DWORD PTR k, eax
        jle     SHORT $LN6@main
$IntoIF$9:

        lea     rcx, OFFSET FLAT:$SG6881
        call    printf
$LN6@main:

        xor     eax, eax

        add     rsp, 40         ; 00000028H
        ret     0
main    ENDP
```

如果将这段代码与原始 C 示例的 MASM 输出进行比较（原始 C 示例依赖于短路求值），你会发现这段代码同样高效。这是为什么在 1970 年代，一些程序员对结构化编程存在相当大抵制的经典例子——有时候它会导致不那么高效的代码。当然，代码的可读性和可维护性通常比几个字节或机器周期更重要。但永远不要忘记，如果性能对某段小代码至关重要，解构这段代码在某些特殊情况下可以提高效率。

### 13.5 `switch`/`case`语句

`switch`（或`case`）高阶控制语句是 HLL（高级语言）中另一种条件语句。如你所见，`if`语句测试布尔表达式，并根据表达式的结果执行代码中的两条不同路径。另一方面，`switch/case`语句可以根据序数（整数）表达式的结果跳转到代码中的多个不同位置。以下示例展示了 C/C++、Pascal 和 HLA 中的`switch`和`case`语句。首先是 C/C++中的`switch`语句：

```

			switch( expression )
{
  case 0:
    << statements to execute if the
       expression evaluates to 0 >>
    break;

  case 1:
    << statements to execute if the
       expression evaluates to 1 >>
    break;

  case 2:
    << statements to execute if the
       expression evaluates to 2>>
    break;

  <<etc>>

  default:
    << statements to execute if the expression is
       not equal to any of these cases >>
}
```

Java 和 Swift 为`switch`语句提供了类似 C/C++的语法，尽管 Swift 的版本有许多附加特性。我们将在“Swift `switch`语句”章节中探讨一些附加特性，具体见第 500 页。

这是一个 Pascal `case`语句的示例：

```

			case ( expression ) of
  0: begin
    << statements to execute if the
       expression evaluates to 0 >>
    end;

  1: begin
    << statements to execute if the
       expression evaluates to 1 >>
    end;

  2: begin
    << statements to execute if the
       expression evaluates to 2>>
    end;

  <<etc>>

  else
    << statements to execute if
       REG32 is not equal to any of these cases >>

end; (* case *)
```

最后，以下是 HLA 中的`switch`语句：

```

			switch( REG32 )

  case( 0 )
    << statements to execute if
       REG32 contains 0 >>

  case( 1 )
    << statements to execute
       REG32 contains 1 >>

  case( 2 )
    << statements to execute if
       REG32 contains 2>>

  <<etc>>

  default
    << statements to execute if
       REG32 is not equal to any of these cases >>

endswitch;
```

从这些例子中你可以看出，这些语句的语法都非常相似。

#### 13.5.1 `switch`/`case`语句的语义

大多数初学编程的课程和教材通过将`switch`/`case`语句与一系列`if..else..if`语句进行比较，来讲解`switch`/`case`语句的语义；这种方式用学生已经理解的概念介绍`switch`/`case`语句。不幸的是，这种方法可能具有误导性。为了解释原因，请看以下代码，一本入门级 Pascal 编程书籍可能会声称它等同于我们的 Pascal `case`语句：

```

			if( expression = 0 ) then begin

  << statements to execute if expression is 0 >>

end
else if( expression = 1 ) then begin

  << statements to execute if expression is 1 >>

end
else if( expression = 2 ) then begin

  << statements to execute if expression is 2 >>

end
else
  << statements to execute if expression is not 1 or 2 >>

end;
```

尽管这个特定的序列会与 `case` 语句达到相同的效果，但 `if..then..elseif` 序列和 Pascal `case` 实现之间有几个根本性的区别。首先，`case` 语句中的 `case` 标签必须都是常量，但在 `if..then..elseif` 链中，你实际上可以将变量和其他非常量值与控制变量进行比较。`switch`/`case` 语句的另一个限制是，你只能将单一表达式的值与常量集合进行比较；而在 `if..then..elseif` 链中，你可以将一个表达式与一个常量进行比较，并将另一个表达式与第二个常量进行比较。稍后会解释这些限制的原因，但这里要记住的是，`if..then..elseif` 链在语义上与 `switch`/`case` 语句不同——并且比其功能更强大。

#### 13.5.2 跳转表与链式比较

尽管 `switch`/`case` 语句在可读性和便利性上可能比 `if..then..elseif` 链更好，但它最初添加到高级语言中是为了效率，而非可读性或便利性。考虑一个包含 10 个独立表达式的 `if..then..elseif` 链。如果所有的 `case` 是互斥的并且同样可能，那么程序平均需要执行五次比较，才能遇到一个计算结果为 `true` 的表达式。在汇编语言中，使用表查找和间接跳转，可以在固定时间内将控制转移到多个不同的目标地址，而不受 `case` 数量的影响。实际上，这段代码利用 `switch`/`case` 表达式的值作为索引，查找地址表中的一个地址，然后（间接地）跳转到表项指定的语句。当 `case` 数量超过三四个时，这种方案通常比相应的 `if..then..elseif` 链更快，且占用更少的内存。考虑以下汇编语言中 `switch`/`case` 语句的简单实现：

```

			// Conversion of
//    switch(i)
//    { case 0:...case 1:...case 2:...case 3:...}
// into assembly

static
  jmpTable: dword[4] :=
    [ &label0, &label1, &label2, &label3 ];
      .
      .
      .
    // jmps to address specified by jmpTable[i]

    mov( i, eax );
    jmp( jmpTable[ eax*4 ] );

label0:
    << code to execute if i = 0 >>
    jmp switchDone;

label1:
    << code to execute if i = 1 >>
    jmp switchDone;

label2:
    << code to execute if i = 2 >>
    jmp switchDone;

label3:
    << code to execute if i = 3 >>

switchDone:
  << Code that follows the switch statement >>
```

为了查看这段代码如何运作，我们将逐条指令进行分析。`jmpTable` 声明定义了一个包含四个双字指针的数组，每个指针对应 `switch` 语句模拟中的一个 `case`。数组中的第 0 项保存了 `switch` 表达式计算结果为 `0` 时需要跳转到的语句地址，第 1 项保存了 `switch` 表达式计算结果为 `1` 时需要执行的语句地址，依此类推。请注意，数组必须包含一个元素，其索引与 `switch` 语句中每个可能的 `case` 匹配（在此例中为 `0` 至 `3`）。

这个例子中的第一条机器指令将`switch`表达式的值（变量`i`的值）加载到 EAX 寄存器中。因为这段代码使用`switch`表达式的值作为索引来访问`jmpTable`数组，所以这个值必须是一个整数（整型）值，存储在 80x86 的 32 位寄存器中。接下来的指令（`jmp`）执行了`switch`语句仿真的实际工作：它跳转到由`jmpTable`数组中由 EAX 索引的条目所指定的地址。如果 EAX 在执行这条`jmp`指令时的值为`0`，程序将从`jmpTable[0]`获取双字，并将控制转移到该地址；这是程序代码中`label0`标签后面的第一条指令的地址。如果 EAX 的值为`1`，则`jmp`指令从内存地址`jmpTable + 4`获取双字（注意，这段代码使用了`*4`的缩放索引寻址模式；有关更多详细信息，请参阅第 34 页）。同样，如果 EAX 的值为`2`或`3`，则`jmp`指令将控制转移到存储在`jmpTable + 8`或`jmpTable + 12`（分别）的双字地址。因为`jmpTable`数组已经初始化了`label0`、`label1`、`label2`和`label3`的地址，分别位于偏移量 0、4、8 和 12，因此这个特定的间接`jmp`指令将把控制转移到与`i`的值相对应的标签语句（`label0`、`label1`、`label2`或`label3`）。

这个`switch`语句仿真最有趣的第一点是，它只需要两条机器指令（和一个跳转表）就能将控制转移到四个可能的案例中的任何一个。与此相比，`if..then..elseif`的实现，每个案例至少需要两条机器指令。实际上，随着你向`if..then..elseif`实现中添加更多的案例，比较和条件分支指令的数量会增加，而跳转表实现的机器指令数始终固定为两条（尽管跳转表的大小会因每个案例增加一个条目而增大）。因此，随着案例的增加，`if..then..elseif`实现会逐渐变慢，而跳转表实现则保持恒定的执行时间（无论案例的数量如何）。假设你的 HLL 编译器为`switch`语句使用了跳转表实现，那么如果有大量案例，`switch`语句通常会比`if..then..elseif`序列更快。

然而，`switch`语句的跳转表实现也有几个缺点。首先，由于跳转表是内存中的一个数组，而访问（非缓存的）内存可能较慢，因此访问跳转表数组可能会影响系统性能。

另一个缺点是，你必须在表格中为每一个可能的情况（从最大值到最小值之间的所有情况）创建一个条目，包括那些你并未明确提供的情况。在目前为止的示例中，这并不是一个问题，因为情况值从`0`开始，并且是连续的，直到`3`。然而，考虑以下的 Pascal `case`语句：

```

			case( i ) of

  0: begin
      << statements to execute if i = 0 >>
     end;

  1: begin
      << statements to execute if i = 1 >>
     end;
  5: begin
      << statements to execute if i = 5 >>
     end;

  8: begin
      << statements to execute if i = 8 >>
     end;

end; (* case *)
```

我们无法通过一个包含四个条目的跳转表来实现这个`case`语句。如果`i`的值是`0`或`1`，则会获取正确的地址。然而，对于情况 5，跳转表的索引将是`20`（5 × 4），而不是跳转表中的第三个（2 × 4 = 8）条目。如果跳转表只包含四个条目（16 字节），那么使用值`20`进行索引将会获取到表格末尾之后的地址，并可能导致应用崩溃。这正是为什么在 Pascal 的原始定义中，如果程序提供了一个在特定`case`语句的标签集合中不存在的情况值，结果将是未定义的原因。

为了解决汇编语言中的这个问题，你必须确保每一个可能的情况标签都有相应的条目，并且所有这些标签之间的值也要包含在内。在当前示例中，跳转表需要九个条目来处理所有可能的情况值，从`0`到`8`：

```

			// Conversion of
//    switch(i)
//    { case 0:...case 1:...case 5:...case 8:}
// into assembly

static
  jmpTable: dword[9] :=
          [
            &label0, &label1, &switchDone,
            &switchDone, &switchDone,
            &label5, &switchDone, &switchDone,
            &label8
          ];
      .
      .
      .
    // jumps to address specified by jmpTable[i]

    mov( i, eax );
    jmp( jmpTable[ eax*4 ] );

label0:
    << code to execute if i = 0 >>
    jmp switchDone;

label1:
    << code to execute if i = 1 >>
    jmp switchDone;
label5:
    << code to execute if i = 5 >>
    jmp switchDone;

label8:
    << code to execute if i = 8 >>

switchDone:
  << Code that follows the switch statement >>
```

注意，如果`i`的值为`2`、`3`、`4`、`6`或`7`，那么这段代码会将控制转移到`switch`语句之后的第一个语句（这是 C 语言的`switch`语句和大多数现代 Pascal 变种中的`case`语句的标准语义）。当然，如果`switch`/`case`表达式的值大于最大情况值，C 语言也会将控制转移到这段代码中的这一点。大多数编译器通过在间接跳转之前立即进行比较和条件分支来实现此功能。例如：

```

			// Conversion of
//    switch(i)
//    { case 0:...case 1:...case 5:...case 8:}
// into assembly, that automatically
// handles values greater than 8.

static
  jmpTable: dword[9] :=
          [
            &label0, &label1, &switchDone,
            &switchDone, &switchDone,
            &label5, &switchDone, &switchDone,
            &label8
          ];
      .
      .
      .
    // Check to see if the value is outside the range
    // of values allowed by this switch/case stmt.

    mov( i, eax );
    cmp( eax, 8 );
    ja switchDone;

    // jmps to address specified by jmpTable[i]

    jmp( jmpTable[ eax*4 ] );

      .
      .
      .

switchDone:
  << Code that follows the switch statement >>
```

你可能已经注意到这段代码做出了另一个假设——即情况值从`0`开始。修改代码以处理任意范围的情况值是很简单的。考虑以下示例：

```

			// Conversion of
//    switch(i)
//    { case 10:...case 11:...case 12:...case 15:...case 16:}
// into assembly, that automatically handles values
// greater than 16 and values less than 10.

static
  jmpTable: dword[7] :=
          [
            &label10, &label11, &label12,
            &switchDone, &switchDone,
            &label15, &label16
          ];
      .
      .
      .
    // Check to see if the value is outside the
    // range 10..16.

    mov( i, eax );
    cmp( eax, 10 );
    jb switchDone;
    cmp( eax, 16 );
    ja switchDone;

    // The "- 10*4" part of the following expression
    // adjusts for the fact that EAX starts at 10
    // rather than 0, but we still need a zero-based
    // index into our array.

    jmp( jmpTable[ eax*4 - 10*4] );

      .
      .
      .

switchDone:
  << Code that follows the switch statement >>
```

这个例子和之前的例子有两个区别。首先，这个例子将 EAX 中的值与范围`10..16`进行比较，如果值超出此范围，则跳转到`switchDone`标签（换句话说，EAX 中的值没有对应的 case 标签）。其次，`jmpTable`的索引被修改为`[eax*4 - 10*4]`。在机器级别，数组总是从索引`0`开始；该表达式中的“`- 10*4`”部分调整了 EAX 实际上包含的是从`10`开始的值，而不是从`0`开始。实际上，这个表达式使得`jmpTable`在内存中的起始位置比声明中所示提前了 40 个字节。因为 EAX 的值总是大于等于 10（由于`eax*4`的作用，实际上是 40 字节或更大），所以这段代码从`jmpTable`声明的起始位置开始访问该表。需要注意的是，HLA 从`jmpTable`的地址中减去这个偏移量；CPU 在运行时并不会实际执行这次减法操作。因此，创建这个基于零的索引不会导致额外的效率损失。

请注意，完全通用的`switch`/`case`语句实际上需要六条指令来实现：原始的两条指令加上四条用于测试范围的指令。^(3) 这一点，再加上间接跳转的执行成本略高于条件分支的事实，解释了为什么`switch`/`case`语句（相对于`if..then..elseif`链）的盈亏平衡点大约在三到四个分支之间。

如前所述，`switch`/`case`语句的跳转表实现有一个严重的缺点，即你必须为从最小 case 到最大 case 之间的每一个可能的值准备一个表项。考虑以下 C/C++ `switch`语句：

```

			switch( i )
{
  case 0:
      << statements to execute if i == 0 >>
      break;

  case 1:
      << statements to execute if i == 1 >>
      break;

  case 10:
      << statements to execute if i == 10 >>
      break;

  case 100:
      << statements to execute if i == 100 >>
      break;

  case 1000:
      << statements to execute if i == 1000 >>
      break;

  case 10000:
      << statements to execute if i == 10000 >>
      break;
}
```

如果 C/C++编译器使用跳转表来实现这个`switch`语句，那么该表将需要 10,001 个条目（也就是说，在 32 位处理器上需要 40,004 字节的内存）。对于这样一个简单的语句来说，这可是相当大的一块内存！虽然各个 case 之间的宽大间隔对内存使用有很大的影响，但它对`switch`语句的执行速度影响却微乎其微。程序执行的仍然是与值是连续的情况相同的四条指令（只需要四条指令，因为 case 值从`0`开始，所以无需检查`switch`表达式是否符合下界）。实际上，唯一导致性能差异的原因是表的大小对缓存的影响（当表非常大时，查找某个特定的表项时，缓存命中率较低）。抛开速度问题不谈，跳转表的内存使用对于大多数应用来说是难以证明其合理性的。因此，如果你的编译器为所有`switch`/`case`语句生成了跳转表（你可以通过查看它生成的代码来确定），那么你应该小心创建那些 case 分布较远的`switch`/`case`语句。

#### 13.5.3 switch/case 的其他实现

由于跳转表大小的问题，一些高级语言编译器并未使用跳转表实现`switch`/`case`语句。有些编译器会将`switch`/`case`语句简单地转换为相应的`if..then..elseif`链（Swift 就是这种情况）。显然，这种编译器在跳转表合适的情况下会生成低质量的代码（从速度角度来看）。许多现代编译器在代码生成方面相对智能。它们会根据`switch`/`case`语句中的案例数量以及案例值的分布来选择使用跳转表还是`if..then..elseif`实现，这取决于一些阈值标准（代码大小与速度的平衡）。有些编译器甚至可能使用这些技术的组合。例如，考虑以下 Pascal 的`case`语句：

```

			case( i ) of
  0: begin
      << statements to execute if i = 0 >>
     end;

  1: begin
      << statements to execute if i = 1 >>
     end;

  2: begin
      << statements to execute if i = 2 >>
     end;

  3: begin
      << statements to execute if i = 3 >>
     end;

  4: begin
      << statements to execute if i = 4 >>
     end;

  1000: begin
      << statements to execute if i = 1000 >>
        end;
end; (* case *)
```

一个好的编译器会识别出大多数案例适合使用跳转表，只有少数（一个或几个）案例不适合。它会将代码转换为一系列结合了`if..then`和跳转表实现的指令。例如：

```

			    mov( i, eax );
    cmp( eax, 4 );
    ja try1000;
    jmp( jmpTable[ eax*4 ] );
      .
      .
      .
try1000:
    cmp( eax, 1000 );
    jne switchDone;
    << code to do if i = 1000 >>
switchDone:
```

尽管`switch`/`case`语句最初是为了在高级语言中使用高效的跳转表传输机制而创建的，但很少有语言定义要求特定的控制结构实现。因此，除非你坚持使用某个特定的编译器，并且知道该编译器在所有情况下如何生成代码，否则完全无法保证你的`switch`/`case`语句会编译成跳转表、`if..then..elseif`链、两者的组合，或者完全不同的东西。例如，考虑以下简短的 C 程序及其生成的汇编输出：

```

			extern void f( void );
extern void g( void );
extern void h( void );
int main( int argc, char **argv )
{
    int boolResult;

    switch( argc )
    {
        case 1:
            f();
            break;

        case 2:
            g();
            break;

        case 10:
            h();
            break;

        case 11:
            f();
            break;
    }
    return 0;
}
```

这是（较旧版本的）Borland C++ v5.0 编译器的 80x86 输出：

```

			_main   proc    near
?live1@0:
   ;
   ;    int main( int argc, char **argv )
   ;
@1:
    push      ebp
    mov       ebp,esp
   ;
   ;    {
   ;        int boolResult;
   ;
   ;        switch( argc )
   ;

; Is argc == 1?

    mov       eax,dword ptr [ebp+8]
    dec       eax
    je        short @7

; Is argc == 2?

    dec       eax
    je        short @6

; Is argc == 10?

    sub       eax,8
    je        short @5

; Is argc == 11?

    dec       eax
    je        short @4

; If none of the above

    jmp       short @2
   ;
   ;        {
   ;            case 1:
   ;                f();
   ;
@7:
    call      _f
   ;
   ;                break;
   ;
    jmp       short @8
   ;
   ;
   ;            case 2:
   ;                g();
   ;
@6:
    call      _g
   ;
   ;                break;
   ;
    jmp       short @8
   ;
   ;
   ;            case 10:
   ;                h();
   ;
@5:
    call      _h
   ;
   ;                break;
   ;
    jmp       short @8
   ;
   ;
   ;            case 11:
   ;                f();
   ;
@4:
    call      _f
   ;
   ;                break;
   ;
   ;        }
   ;        return 0;
   ;
@2:
@8:
    xor       eax,eax
   ;
   ;    }
   ;
@10:
@9:
    pop       ebp
    ret
_main   endp
```

如你所见，在主程序开始部分，代码将`argc`中的值依次与四个值（`1`、`2`、`10`和`11`）进行比较。对于这样一个小的`switch`语句，这并不是一个糟糕的实现。

当有相当多的案例且跳转表会太大时，许多现代优化编译器会生成二叉搜索树来测试各个案例。例如，考虑以下 C 程序及其相应的输出：

```

			#include <stdio.h>

extern void f( void );
int main( int argc, char **argv )
{
    int boolResult;

    switch( argc )
    {
        case 1:
            f();
            break;

        case 10:
            f();
            break;

        case 100:
            f();
            break;

        case 1000:
            f();
            break;

        case 10000:
            f();
            break;

        case 100000:
            f();
            break;

        case 1000000:
            f();
            break;

        case 10000000:
            f();
            break;

        case 100000000:
            f();
            break;

        case 1000000000:
            f();
            break;
    }
    return 0;
}
```

这是来自 Visual C++编译器的 64 位 MASM 输出。注意，微软的编译器如何通过对每个 10 个案例进行部分二叉搜索：

```

			main    PROC

$LN18:
        sub     rsp, 40                                 ; 00000028H

; >+ 100,000?
        cmp     ecx, 100000                             ; 000186a0H
        jg      SHORT $LN15@main
        je      SHORT $LN10@main

; handle cases where argc is less than 100,000
;
; Check for argc = 1

        sub     ecx, 1
        je      SHORT $LN10@main

; check for argc = 10

        sub     ecx, 9
        je      SHORT $LN10@main

;check for argc = 100

        sub     ecx, 90                                 ; 0000005aH
        je      SHORT $LN10@main

; check for argc = 1000

        sub     ecx, 900                                ; 00000384H
        je      SHORT $LN10@main

; check for argc = 1000
        cmp     ecx, 9000                               ; 00002328H

        jmp     SHORT $LN16@main
$LN15@main:

; Check for argc = 100,000

      cmp     ecx, 1000000                              ; 000f4240H
        je      SHORT $LN10@main

; check for argc = 1,000,000
        cmp     ecx, 10000000                           ; 00989680H
        je      SHORT $LN10@main

; check for argc = 10,000,000
        cmp     ecx, 100000000                          ; 05f5e100H
        je      SHORT $LN10@main

; check for argc = 100,000,000

        cmp     ecx, 1000000000                         ; 3b9aca00H
$LN16@main:
        jne     SHORT $LN2@main
$LN10@main:

        call    f
$LN2@main:

        xor     eax, eax

        add     rsp, 40                                 ; 00000028H
        ret     0
main    ENDP
```

有趣的是，在编译成 32 位代码时，Visual C++会生成真正的二叉搜索。以下是 Visual C++ 32 位版本的 MASM32 输出：

```

			_main   PROC

        mov     eax, DWORD PTR _argc$[esp-4] ; argc is passed on stack in 32-bit code

; Start with >100,000, = 100,000, or < 100,000

        cmp     eax, 100000                  ; 000186a0H
        jg      SHORT $LN15@main             ; Go if >100,000
        je      SHORT $LN4@main              ; Match if equal

; Handle cases where argc < 100,000
;
; Divide it into >100 and < 100

        cmp     eax, 100                     ; 00000064H
        jg      SHORT $LN16@main             ; Branch if > 100
        je      SHORT $LN4@main              ; = 100

; Down here if < 100

        sub     eax, 1
        je      SHORT $LN4@main              ; branch if it was 1

        sub     eax, 9                       ; Test for 10
        jmp     SHORT $LN18@main

; Come down here if >100 and <100,000
$LN16@main:

        cmp     eax, 1000                    ; 000003e8H
        je      SHORT $LN4@main              ; Branch if 1000
        cmp     eax, 10000                   ; 00002710H
        jmp     SHORT $LN18@main             ; Handle =10,000 or not in range

; Handle > 100,000 here.

$LN15@main:
        cmp     eax, 100000000               ; 05f5e100H
        jg      SHORT $LN17@main             ; > 100,000,000
        je      SHORT $LN4@main              ; = 100,000

; Handle < 100,000,000 and > 100,000 here:

        cmp     eax, 1000000                 ; 000f4240H
        je      SHORT $LN4@main              ; =1,000,000
        cmp     eax, 10000000                ; 00989680H

        jmp     SHORT $LN18@main             ; Handle 10,000,000 or not in range

; Handle > 100,000,000 here
$LN17@main:
; check for 1,000,000,000
        cmp     eax, 1000000000              ; 3b9aca00H
$LN18@main:
        jne     SHORT $LN2@main
$LN4@main:

        call    _f
$LN2@main:

        xor     eax, eax

        ret     0
_main   ENDP
```

一些编译器，特别是某些微控制器设备的编译器，会生成一个 *2 元组* 表（成对的记录/结构），元组的一个元素是 case 值，第二个元素是如果值匹配时跳转到的地址。然后编译器会生成一个循环，扫描这个小表，寻找当前 `switch`/`case` 表达式的值。如果这是线性搜索，这种实现比 `if..then..elseif` 链还慢。如果编译器生成的是二分查找，代码可能会比 `if..then..elseif` 链更快，但可能不如跳转表实现快。

这是一个 Java `switch` 语句的例子，以及编译器生成的 Java 字节码：

```

			public class Welcome
{
    public static void f(){}
    public static void main( String[] args )
    {
        int i = 10;
        switch (i)
        {
            case 1:
                f();
                break;

            case 10:
                f();
                break;

            case 100:
                f();
                break;

            case 1000:
                f();
                break;

            case 10000:
                f();
                break;

            case 100000:
                f();
                break;

            case 1000000:
                f();
                break;

            case 10000000:
                f();
                break;

            case 100000000:
                f();
                break;

            case 1000000000:
                f();
                break;

        }
    }
}

// JBC output:

Compiled from "Welcome.java"
public class Welcome extends java.lang.Object{
public Welcome();
  Code:
   0:   aload_0
   1:   invokespecial   #1; //Method java/lang/Object."<init>":()V
   4:   return

public static void f();
  Code:
   0:   return

public static void main(java.lang.String[]);
  Code:
   0:   bipush  10
   2:   istore_1
   3:   iload_1
   4:   lookupswitch{ //10
        1: 96;
        10: 102;
        100: 108;
        1000: 114;
        10000: 120;
        100000: 126;
        1000000: 132;
        10000000: 138;
        100000000: 144;
        1000000000: 150;
        default: 153 }
   96:  invokestatic    #2; //Method f:()V
   99:  goto    153
   102: invokestatic    #2; //Method f:()V
   105: goto    153
   108: invokestatic    #2; //Method f:()V
   111: goto    153
   114: invokestatic    #2; //Method f:()V
   117: goto    153
   120: invokestatic    #2; //Method f:()V
   123: goto    153
   126: invokestatic    #2; //Method f:()V
   129: goto    153
   132: invokestatic    #2; //Method f:()V
   135: goto    153
   138: invokestatic    #2; //Method f:()V
   141: goto    153
   144: invokestatic    #2; //Method f:()V
   147: goto    153
   150: invokestatic    #2; //Method f:()V
   153: return
}
```

`lookupswitch` 字节码指令包含一个由 2 元组组成的表。如前所述，元组的第一个值是 case 值，第二个值是匹配时代码跳转的目标地址。可以推测，字节码解释器对这些值执行二分查找，而不是线性查找（希望是这样！）。注意，Java 编译器为每个 case 生成了单独的 `f()` 方法调用；它并没有像 GCC 和 Visual C++ 那样将它们优化为单个调用。

**注意**

*Java 还具有一个 tableswitch 虚拟机指令，用于执行基于表驱动的 switch 操作。Java 编译器会根据 case 值的密度选择使用 tableswitch 还是 lookupswitch 指令。*

有时候，编译器会采用一些代码技巧，在特定情况下生成略微更好的代码。再看看导致 Borland 编译器生成线性查找的简短 `switch` 语句：

```

			switch( argc )
    {
        case 1:
            f();
            break;

        case 2:
            g();
            break;

        case 10:
            h();
            break;

        case 11:
            f();
            break;

    }
```

这是微软 Visual C++ 32 位编译器为这个 `switch` 语句生成的代码：

```

			; File t.c
; Line 13
;
; Use ARGC as an index into the $L1240 table,
; which returns an offset into the $L1241 table:

    mov eax, DWORD PTR _argc$[esp-4]
    dec eax         ; --argc, 1=0, 2=1, 10=9, 11=10
    cmp eax, 10     ; Out of range of cases?
    ja  SHORT $L1229
    xor ecx, ecx
    mov cl, BYTE PTR $L1240[eax]
    jmp DWORD PTR $L1241[ecx*4]

    npad    3
$L1241:
    DD  $L1232  ; cases that call f
    DD  $L1233  ; cases that call g
    DD  $L1234  ; cases that call h
    DD  $L1229  ; Default case

$L1240:
    DB  0   ; case 1 calls f
    DB  1   ; case 2 calls g
    DB  3   ; default
    DB  3   ; default
    DB  3   ; default
    DB  3   ; default
    DB  3   ; default
    DB  3   ; default
    DB  3   ; default
    DB  2   ; case 10 calls h
    DB  0   ; case 11 calls f

; Here is the code for the various cases:

$L1233:
; Line 19
    call    _g
; Line 31
    xor eax, eax
; Line 32
    ret 0

$L1234:
; Line 23
    call    _h
; Line 31
    xor eax, eax
; Line 32
    ret 0

$L1232:
; Line 27
    call    _f
$L1229:
; Line 31
    xor eax, eax
; Line 32
    ret 0
```

这个 80x86 代码的技巧在于，Visual C++ 首先通过表查找将 `argc` 值范围 `1..11` 映射到值范围 `0..3`（对应于出现的三个不同的代码体，以及一个默认 case）。这段代码比跳转表要短，双字条目对应于默认 case，尽管它比跳转表稍慢，因为它需要访问内存中的两个不同表。（至于这段代码的速度与二分查找或线性查找相比如何，这个研究留给你自己；答案可能因处理器而异。）然而需要注意的是，当生成 64 位代码时，Visual C++ 会回退到线性查找：

```

			main    PROC

$LN12:
        sub     rsp, 40                                 ; 00000028H

; ARGC is passed in ECX

        sub     ecx, 1
        je      SHORT $LN4@main  ; case 1
        sub     ecx, 1
        je      SHORT $LN5@main  ; case 2
        sub     ecx, 8
        je      SHORT $LN6@main  ; case 10
        cmp     ecx, 1
        jne     SHORT $LN10@main ; case 11
$LN4@main:

        call    f
$LN10@main:

        xor     eax, eax

        add     rsp, 40                                 ; 00000028H
        ret     0
$LN6@main:

        call    h

        xor     eax, eax

        add     rsp, 40                                 ; 00000028H
        ret     0
$LN5@main:

        call    g

        xor     eax, eax

        add     rsp, 40                                 ; 00000028H
        ret     0
main    ENDP
```

很少有编译器允许你显式指定编译器如何转换特定的`switch`/`case`语句。例如，如果你真的希望之前提到的包含 0、1、10、100、1000 和 10000 这几个 case 的`switch`语句生成跳转表，你必须使用汇编语言编写代码，或者使用你理解其代码生成特性的特定编译器。然而，任何依赖于编译器生成跳转表的 HLL 代码都无法在其他编译器中移植，因为很少有语言指定高级控制结构的实际机器代码实现。

当然，你不必完全依赖编译器为`switch`/`case`语句生成高效的代码。假设你的编译器对所有`switch`/`case`语句使用跳转表实现，当对你的 HLL 源代码做出修改时，可能会生成一个巨大的跳转表，你可以帮助编译器生成更好的代码。例如，考虑之前给出的`switch`语句，包含 0、1、2、3、4 和 1000 这几个 case。如果你的编译器生成一个包含 1001 个条目的跳转表（占用超过 4KB 的内存），你可以通过编写以下 Pascal 代码来改善编译器的输出：

```

			if( i = 1000 ) then begin

  << statements to execute if i = 1000 >>

end
else begin

  case( i ) of
    0: begin
        << statements to execute if i = 0 >>
       end;

    1: begin
        << statements to execute if i = 1 >>
       end;

    2: begin
        << statements to execute if i = 2 >>
       end;

    3: begin
        << statements to execute if i = 3 >>
       end;

    4: begin
        << statements to execute if i = 4 >>
       end;
  end; (* case *)
end; (* if *)
```

通过将 case 值`1000`放在`switch`语句之外，编译器可以为主要的、连续的 case 生成一个简短的跳转表。

另一个可能性（可以说更容易阅读）是以下 C/C++代码：

```

			switch( i )
{
  case 0:
      << statements to execute if i == 0 >>
      break;

  case 1:
      << statements to execute if i == 1 >>
      break;

  case 2:
      << statements to execute if i == 2 >>
      break;

  case 3:
      << statements to execute if i == 3 >>
      break;

  case 4:
      << statements to execute if i == 4 >>
     break;

  default:
    if( i == 1000 )
    {
      << statements to execute if i == 1000 >>
    }
    else
    {
      << Statements to execute if none of the cases match >>
    }
}
```

使这个例子稍微容易阅读的原因是，当`i`等于`1000`时的代码已经被移入了`switch`语句中（得益于默认子句），因此它看起来不会与`switch`中进行的所有测试分开。

一些编译器根本不会为`switch`/`case`语句生成跳转表。如果你使用的是这样的编译器，并且希望生成跳转表，那么除了使用汇编语言或非标准的 C 扩展之外，你几乎无能为力。

尽管`switch`/`case`语句的跳转表实现通常在有大量 case 且它们的可能性相等时效率较高，但请记住，如果其中一两个 case 的可能性远高于其他 case，使用`if..then..elseif`链可能更高效。例如，如果一个变量的值为`15`的时间超过一半，`20`的时间大约四分之一，而其余 25%的时间则是其他几种不同的值，那么使用`if..then..elseif`链（或`if..then..elseif`和`switch`/`case`语句的组合）来实现多路选择可能更为高效。通过先测试最常见的 case，你通常可以减少多路选择语句执行所需的平均时间。例如：

```

			if( i == 15 )
{
  // If i = 15 better than 50% of the time,
  // then we only execute a single test
  // better than 50% of the time:
}
else if( i == 20 )
{
  // if i == 20 better than 25% of the time,
  // then we only execute one or
  // two comparisons 75% of the time.
}
else if etc....
```

如果 `i` 的值为 `15` 的情况出现得更多，那么大多数时候这段代码会在执行仅两条指令后，执行第一个 `if` 语句的主体。即使是在最好的 `switch` 语句实现中，你仍然需要比这更多的指令。

#### 13.5.4 Swift `switch` 语句

Swift 的 `switch` 语句在语义上与大多数其他语言不同。Swift 的 `switch` 和典型的 C/C++ `switch` 或 Pascal `case` 语句之间有四个主要区别：

+   Swift 的 `switch` 提供了一个特殊的 `where` 子句，允许你对 `switch` 应用条件。

+   Swift 的 `switch` 允许你在多个 `case` 语句中使用相同的值（通过 `where` 子句区分）。

+   Swift 的 `switch` 允许使用非整数/序数数据类型，如元组、字符串和集合，作为选择值（并配有适当类型的 case 值）。

+   Swift 的 `switch` 语句支持对 case 值进行模式匹配。

请查阅 Swift 语言参考手册以获取更多细节。本节的目的是讨论 Swift 设计如何影响其实现，而不是提供 Swift `switch` 的语法和语义。

由于它允许任意类型作为 `switch` 选择值，因此 Swift 无法使用跳转表来实现 `switch` 语句。跳转表实现需要一个序数值（可以表示为整数），编译器可以将其用作跳转表的索引。例如，字符串选择值不能用作数组的索引。此外，Swift 允许你指定相同的 case 值两次^(4)，这就导致了一个一致性问题，因为同一个跳转表条目会映射到代码的两个不同部分（这对于跳转表来说是不可能的）。

鉴于 Swift `switch` 语句的设计，唯一的解决方案是线性搜索（实际上，`switch` 语句等同于一系列 `if..else if..else if..etc` 语句）。最重要的是，使用 `switch` 语句并不会比使用一组 `if` 语句带来性能上的好处。

#### 13.5.5 `switch` 语句的编译器输出

在你去帮助你的编译器生成更好的 `switch` 语句代码之前，你可能想检查一下它实际生成的代码。本章已经描述了各种编译器在机器码层面实现 `switch`/`case` 语句时使用的几种技术，但还有一些额外的实现本书未能覆盖。尽管你不能假设编译器总是为 `switch`/`case` 语句生成相同的代码，观察它的输出有助于你了解编译器作者使用的不同实现方式。

### 13.6 更多信息

Aho, Alfred V., Monica S. Lam, Ravi Sethi 和 Jeffrey D. Ullman. *编译器：原理、技术与工具*（第 2 版）。英国埃塞克斯：皮尔逊教育有限公司，1986 年。

Barrett, William, 和 John Couch. *编译器构造：理论与实践*. 芝加哥：SRA, 1986.

Dershem, Herbert, 和 Michael Jipping. *编程语言、结构与模型*. 贝尔蒙特，加利福尼亚州：Wadsworth, 1990.

Duntemann, Jeff. *汇编语言一步步学*. 第 3 版. 印第安纳波利斯：Wiley, 2009.

Fraser, Christopher, 和 David Hansen. *可重定向的 C 编译器：设计与实现*. 波士顿：Addison-Wesley Professional, 1995.

Ghezzi, Carlo, 和 Jehdi Jazayeri. *编程语言概念*. 第 3 版. 纽约：Wiley, 2008.

Hoxey, Steve, Faraydon Karim, Bill Hay, 和 Hank Warren, 编辑. *PowerPC 编译器编写者指南*. 帕洛阿尔托，加利福尼亚州：Warthman Associates 为 IBM 出版, 1996.

Hyde, Randall. *汇编语言艺术*. 第 2 版. 旧金山：No Starch Press, 2010.

Intel. “Intel 64 和 IA-32 架构软件开发者手册。”更新于 2019 年 11 月 11 日。*[`software.intel.com/en-us/articles/intel-sdm`](https://software.intel.com/en-us/articles/intel-sdm)*.

Ledgard, Henry, 和 Michael Marcotty. *编程语言的全景图*. 芝加哥：SRA, 1986.

Louden, Kenneth C. *编译器构造：原理与实践*. 波士顿：Cengage, 1997.

Louden, Kenneth C., 和 Kenneth A. Lambert. *编程语言：原理与实践*. 第 3 版. 波士顿：Course Technology, 2012.

Parsons, Thomas W. *编译器构造导论*. 纽约：W. H. Freeman, 1992.

Pratt, Terrence W., 和 Marvin V. Zelkowitz. *编程语言：设计与实现*. 第 4 版. 上萨德尔河，新泽西州：Prentice Hall, 2001.

Sebesta, Robert. *编程语言概念*. 第 11 版. 波士顿：Pearson, 2016.
