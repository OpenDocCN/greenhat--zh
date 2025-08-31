# 第十四章：**迭代控制结构**

![image](img/common01.jpg)

大多数程序的大部分时间都在循环中执行机器指令。因此，如果你想提高应用程序的执行速度，首先应该看看能否提高代码中循环的性能。本章将介绍以下几种循环形式：

+   `while` 循环

+   `repeat..until/do..while` 循环

+   `forever`（无限）循环

+   `for`（确定性）循环

### 14.1 while 循环

`while` 循环可能是高级语言（HLL）提供的最通用的迭代语句，因此编译器通常会努力生成最优的代码。`while` 循环在循环体的顶部测试一个布尔表达式，如果表达式计算结果为 `true`，则执行循环体。当循环体执行完毕后，控制权返回到测试语句，过程重复。当布尔控制表达式计算结果为 `false` 时，程序将控制转移到循环体之外的第一个语句。这意味着如果程序首次遇到 `while` 语句时布尔表达式计算结果为 `false`，程序将跳过循环体中的所有语句而不执行它们。以下示例演示了一个 Pascal 的 `while` 循环：

```

			while( a < b ) do begin

  << Statements to execute if a is less than b.
     Presumably, these statements modify the value
     of either a or b so that this loop ultimately
     terminates. >>

end; (* while *)
<< statements that execute when a is not less than b >>
```

你可以通过使用 `if` 语句和 `goto` 语句在高级语言中轻松模拟 `while` 循环。考虑以下 C/C++ 的 `while` 循环以及使用 `if` 和 `goto` 的语义等效代码：

```

			// while loop:

while( x < y )
{
  arr[x] = y;
  ++x;
}

// Conversion to an if and a goto:

whlLabel:
if( x < y )
{
  arr[x] = y;
  ++x;
  goto whlLabel;
}
```

假设为了这个示例，当 `if`/`goto` 组合首次执行时，`x` 小于 `y`。既然这一条件为 `true`，那么循环体（`if` 语句的 `then` 部分）就会执行。在循环体的底部，`goto` 语句将控制转移回到 `if` 语句之前的位置。这意味着代码会再次测试该表达式，就像 `while` 循环那样。每当 `if` 表达式计算结果为 `false` 时，控制将转移到 `if` 后的第一个语句（这将控制转移到 `goto` 语句之后的部分）。

尽管 `if`/`goto` 组合在语义上与 `while` 循环相同，但这并不意味着这里呈现的 `if`/`goto` 方案比典型编译器生成的代码更高效。并不是的。以下汇编代码展示了你从一个平庸的编译器中获得的 `while` 循环的代码：

```

			  // while( x < y )

whlLabel:
    mov( x, eax );
    cmp( eax, y );
    jnl exitWhile;  // jump to exitWhile label if
                    // x is not less than y

    mov( y, edx );
    mov( edx, arr[ eax*4 ] );
    inc( x );
    jmp whlLabel;
exitWhile:
```

一个优秀的编译器会通过使用一种叫做*代码移动*（或*表达式旋转*）的技术稍微改进这一点。考虑一下这个比之前的 `while` 循环更高效的实现：

```

			// while( x < y )

    // Skip over the while loop's body.

    jmp testExpr;

whlLabel:
    // This is the body of the while loop (same as
    // before, except moved up a few instructions).

    mov( y, edx );
    mov( edx, arr[ eax*4 ] );
    inc( x );

// Here is where we test the expression to
// determine if we should repeat the loop body.

testExpr:
    mov( x, eax );
    cmp( eax, y );
    jl whlLabel;    // Transfer control to loop body if x < y.
```

这个示例与前一个示例的机器指令数量完全相同，但循环终止的测试已移至循环的底部。为了保留`while`循环的语义（以便我们在第一次遇到循环时，如果表达式求值为`false`，就不执行循环体），该序列中的第一条语句是一个`jmp`语句，将控制转移到测试循环终止表达式的代码。如果该测试求值为`true`，程序将控制转移到`while`循环体（紧跟在`whlLabel`之后）。

尽管此代码与前一个示例有相同数量的语句，但两者之间有一个微妙的区别。在这个后者的示例中，初始的`jmp`指令仅执行一次——也就是循环执行的第一次。此后的每次迭代，代码都会跳过该语句的执行。在原始示例中，相应的`jmp`语句位于循环体的底部，并且在每次循环迭代时都会执行。因此，如果循环体执行多于一次，第二个版本会运行得更快（另一方面，如果`while`循环即使一次也很少执行循环体，那么第一个版本会稍微更高效）。如果你的编译器没有为`while`语句生成最佳代码，考虑使用不同的编译器。正如第十三章所讨论的那样，尝试通过使用`if`和`goto`语句在高级语言中编写优化代码会产生难以阅读的意大利面条代码，而且通常情况下，代码中的`goto`语句实际上会损害编译器生成良好输出的能力。

**注意**

*当本章讨论 repeat..until/do..while 循环时，你会看到一个替代方案，它不同于 if..goto 方案，并且会产生更结构化的代码，编译器可能更容易处理。尽管如此，如果你的编译器无法进行像这样的简单转换，那么编译后的`while`循环的效率可能是你最小的问题之一。*

做得不错的编译器通常会对`while`循环进行优化，并假设该循环有一个入口点和一个出口点。许多语言提供语句允许提前退出循环（例如，`break`，如“`goto`语句的受限形式”一节中在第 459 页讨论过的那样）。当然，许多语言也提供某种形式的`goto`语句，允许你在任意点进入或退出循环。然而，记住，尽管使用这样的语句可能是合法的，但它们可能会严重影响编译器优化代码的能力。所以要小心使用它们。^(1) `while`循环是一个你应该让编译器处理优化，而不是自己试图优化代码的地方（实际上，这适用于所有循环，因为编译器通常在优化循环时做得很好）。

#### 14.1.1 强制在 `while` 循环中完全布尔求值

`while`语句的执行依赖于布尔表达式求值的语义。与`if`语句类似，有时`while`循环的正确执行取决于布尔表达式是否使用完全求值或短路求值。本节介绍了如何强制`while`循环使用完全布尔求值，下一节将展示如何强制短路求值。

起初，你可能会猜测在`while`循环中强制完全布尔求值的方法与在`if`语句中一样。然而，如果你回顾一下针对`if`语句给出的解决方案（请参见第 465 页的“强制在`if`语句中进行完全布尔求值”），你会意识到我们在`if`语句中使用的方法（嵌套`if`和临时计算）对于`while`语句是行不通的。我们需要一种不同的方法。

##### 14.1.1.1 以简单但低效的方式使用函数

强制完全布尔求值的一个简单方法是写一个函数来计算布尔表达式的结果，并在该函数内使用完全布尔求值。以下 C 代码实现了这个思路：

```

			#include <stdio.h>

static int i;
static int k;

extern int x;
extern int y;
extern int f( int );
extern int g( int );

/*
** Complete Boolean evaluation
** for the expression:
** i < g(y) || k > f(x)
*/

int func( void )
{
    int temp;
    int temp2;

    temp = i < g(y);
    temp2 = k > f(x);
    return temp || temp2;
}

int main( void )
{
    /*
    ** The following while loop
    ** uses complete Boolean evaluation
    */

    while( func() )
    {
      IntoIF:

        printf( "Hello" );
    }

    return( 0 );
}
```

这是 GCC（x86）为这段 C 代码生成的代码（经过一些清理，去除了多余的行）：

```

			func:
.LFB0:
        pushq   %rbp
        movq    %rsp, %rbp
        subq    $16, %rsp
        movl    y(%rip), %eax
        movl    %eax, %edi
        call    g
        movl    %eax, %edx
        movl    i(%rip), %eax
        cmpl    %eax, %edx
        setg    %al
        movzbl  %al, %eax
        movl    %eax, -8(%rbp)
        movl    x(%rip), %eax
        movl    %eax, %edi
        call    f
        movl    %eax, %edx
        movl    k(%rip), %eax
        cmpl    %eax, %edx
        setl    %al
        movzbl  %al, %eax
        movl    %eax, -4(%rbp)
        cmpl    $0, -8(%rbp)
        jne     .L2
        cmpl    $0, -4(%rbp)
        je      .L3
.L2:
        movl    $1, %eax
        jmp     .L4
.L3:
        movl    $0, %eax
.L4:
        leave
        ret
.LFE0:
        .size   func, .-func
        .section        .rodata
.LC0:
        .string "Hello"
        .text
        .globl  main
        .type   main, @function
main:
.LFB1:
        pushq   %rbp
        movq    %rsp, %rbp
        jmp     .L7
.L8:
        movl    $.LC0, %edi
        movl    $0, %eax
        call    printf
.L7:
        call    func
        testl   %eax, %eax
        jne     .L8
        movl    $0, %eax
        popq    %rbp
        ret
```

正如汇编代码所示，这种方法的问题在于，代码必须进行函数调用和返回（这两者都是较慢的操作），才能计算表达式的值。对于许多表达式来说，调用和返回的开销比实际计算表达式值的成本更高。

##### 14.1.1.2 使用内联函数

前一种方法显然并不是你能够得到的最优代码，无论是在空间还是速度上。如果你的编译器支持内联函数，你可以通过在这个例子中将`func()`内联，从而生成一个更好的结果：

```

			#include <stdio.h>

static int i;
static int k;

extern int x;
extern int y;
extern int f( int );
extern int g( int );

inline int func( void )
{
    int temp;
    int temp2;

    temp = i < g(y);
    temp2 = k > f(x);
    return temp || temp2;
}

int main( void )
{
    while( func() )
    {
      IntoIF:

        printf( "Hello" );
    }

    return( 0 );
}
```

这是 GCC 编译器将其转换为（32 位）x86 Gas 汇编的代码：

```

			main:
        pushl   %ebp
        movl    %esp, %ebp
        pushl   %ebx
        pushl   %ecx
        andl    $-16, %esp
        .p2align 2,,3
.L2:
        subl    $12, %esp

; while( i < g(y) || k > f(x) )
;
; Compute g(y) into %EAX:

        pushl   y
        call    g
        popl    %edx
        xorl    %ebx, %ebx
        pushl   x

; See if i < g(y) and leave Boolean result
; in %EBX:

        cmpl    %eax, i
        setl    %bl

; Compute f(x) and leave result in %EAX:

        call    f                ; Note that we call f, even if the
        addl    $16, %esp        ; above evaluates to true

; Compute k > f(x), leaving the result in %EAX.

        cmpl    %eax, k
        setg    %al

; Compute the logical OR of the above two expressions.

        xorl    %edx, %edx
        testl   %ebx, %ebx
        movzbl  %al, %eax
        jne     .L6
        testl   %eax, %eax
        je      .L7
.L6:
        movl    $1, %edx
.L7:
        testl   %edx, %edx
        je      .L10
.L8:

; Loop body:

        subl    $12, %esp
        pushl   $.LC0
        call    printf
        addl    $16, %esp
        jmp     .L2
.L10:
        xorl    %eax, %eax
        movl    -4(%ebp), %ebx
        leave
        ret
```

正如这个例子所示，GCC 将函数直接编译到`while`循环的测试中，避免了与函数调用和返回相关的开销。

##### 14.1.1.3 使用按位逻辑运算

在 C 语言中，支持对位进行布尔操作（也称为*按位逻辑操作*），你可以使用与`if`语句相同的技巧来强制进行完全的布尔求值——只需使用按位运算符。在`&&`或*||*运算符的左右操作数始终为`0`或`1`的特殊情况下，你可以像下面这样写代码，强制进行完全的布尔求值：

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
    // Use "|" rather than "||"
    // to force complete Boolean
    // evaluation here.

    while( i < g(y) | k > f(x) )
    {
        printf( "Hello" );
    }

    return( 0 );
}
```

这是 Borland C++为这段 C 源代码生成的汇编代码：

```

			_main   proc    near
?live1@0:
   ;
   ;    int main( void )
   ;
@1:
        push      ebx
        jmp       short @3 ; Skip to expr test.
   ;
   ;    {
   ;            while( i < g(y) | k > f(x) )
   ;            {
   ;                    printf( "Hello" );
   ;
@2:
        ; Loop body.

        push      offset s@
        call      _printf
        pop       ecx

; Here's where the test of the expression
; begins:

@3:
        ; Compute "i < g(y)" into ebx:

        mov       eax,dword ptr [_y]
        push      eax
        call      _g
        pop       ecx
        cmp       eax,dword ptr [_i]
        setg      bl
        and       ebx,1

        ;  Compute "k > f(x)" into EDX:

        mov       eax,dword ptr [_x]
        push      eax
        call      _f
        pop       ecx
        cmp       eax,dword ptr [_k]
        setl      dl
        and       edx,1
        ; Compute the logical OR of
        ; the two results above:

        or        ebx,edx

        ; Repeat loop body if true:

        jne       short @2
   ;
   ;            }
   ;
   ;            return( 0 );
   ;
        xor       eax,eax
   ;
   ;    }
   ;
@5:
@4:
        pop       ebx
        ret
_main   endp
```

正如你在这段 80x86 输出中看到的，使用位运算逻辑运算符时，编译器生成的是语义等效的代码。只需记住，这段代码只有在你使用`0`和`1`分别表示布尔值`false`和`true`时才有效。

##### 14.1.1.4 使用非结构化代码

如果你没有内联函数的能力，或者位运算逻辑运算符不可用，你可以使用非结构化代码强制进行完全的布尔运算，作为最后的手段。基本思路是创建一个无限循环，然后编写代码在条件失败时显式退出循环。通常，你会使用`goto`语句（或类似 C 的`break`或`continue`语句的有限形式）来控制循环终止。请看下面的 C 语言示例：

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
    int temp2;
 for( ;; )                 //Infinite loop in C/C++
    {
        temp = i < g(y);
        temp2 = k > f(x);
        if( !temp && !temp2 ) break;
        printf( "Hello" );
    }

    return( 0 );
}
```

通过使用无限循环并显式地中断，我们能够通过独立的 C 语句计算布尔表达式的两个部分（因此，强制编译器执行两个子表达式）。这是 MSVC++ 编译器生成的代码：

```

			main    PROC
; File c:\users\rhyde\test\t\t\t.cpp
; Line 16
$LN9:
        sub     rsp, 56                                 ; 00000038H

; Infinite loop jumps here:

$LN2@main:
; Line 21
;
; temp = i < g(y);
;
        mov     ecx, DWORD PTR ?y@@3HA                  ; y
        call    ?g@@YAHH@Z                              ; g

; compute i < g(y) and leave result in eax:

        cmp     DWORD PTR ?i@@3HA, eax
        jge     SHORT $LN5@main
        mov     DWORD PTR tv67[rsp], 1
        jmp     SHORT $LN6@main
$LN5@main:
        mov     DWORD PTR tv67[rsp], 0

$LN6@main:

; temp2 = k > f(x);

        mov     ecx, DWORD PTR ?x@@3HA                  ; x
        call    ?f@@YAHH@Z                              ; f

; compute k > f(x) and leave result in eax:

        cmp     DWORD PTR ?k@@3HA, eax
        jle     SHORT $LN7@main
        mov     DWORD PTR tv71[rsp], 1
        jmp     SHORT $LN8@main
$LN7@main:
        mov     DWORD PTR tv71[rsp], 0
$LN8@main:

; if( !temp && !temp2 ) break;

        or      ecx, eax
        mov     eax, ecx
        test    eax, eax
        je      SHORT $LN3@main
; Line 23
        lea     rcx, OFFSET FLAT:$SG6924
        call    printf

; Jump back to beginning of for(;;) loop.
;
; Line 24
        jmp     SHORT $LN2@main

$LN3@main:
; Line 26
        xor     eax, eax
; Line 27
        add     rsp, 56                                 ; 00000038H
        ret     0
main    ENDP
```

如你所见，这个程序总是会评估原始布尔表达式的两个部分（也就是说，你得到了完整的布尔运算）。

你在以这种方式使用非结构化代码时应该小心。不仅结果更难以阅读，而且很难迫使编译器生成你想要的代码。此外，在一个编译器上生成的有效代码序列，在其他编译器上可能无法生成相同的代码。

如果你的编程语言不支持像`break`这样的语句，你可以始终使用`goto`语句跳出循环并实现相同的效果。尽管将`goto`语句注入到代码中并不是一个好主意，但在某些情况下，它是你唯一的选择。

#### 14.1.2 强制在 while 循环中进行短路布尔运算

有时候，即使语言（如 BASIC 或 Pascal）没有实现该方案，你也需要确保在`while`语句中强制进行短路运算。对于`if`语句，你可以通过重新安排程序中计算循环控制表达式的方式来强制进行短路运算。与`if`语句不同的是，你不能使用嵌套的`while`语句或在`while`循环前加上其他语句来强制短路运算，但在大多数编程语言中，依然可以做到这一点。

请考虑以下 C 代码片段：

```

			while( ptr != NULL && ptr->data != 0 )
{
    << loop body >>
    ptr = ptr->Next; // Step through a linked list.
}
```

如果 C 没有保证布尔表达式的短路运算，这段代码可能会失败。

和强制进行完全布尔运算一样，在像 Pascal 这样的语言中，最简单的方法是编写一个函数，使用短路布尔运算计算并返回布尔结果。然而，由于函数调用的高开销，这种方法相对较慢。请看下面的 Pascal 示例：^(2)

```

			program shortcircuit;
{$APPTYPE CONSOLE}
uses SysUtils;
var
    ptr     :Pchar;

    function shortCir( thePtr:Pchar ):boolean;
    begin

        shortCir := false;
        if( thePtr <> NIL ) then begin

            shortCir := thePtr^ <> #0;

        end; //if

    end;  // shortCircuit

begin

    ptr := 'Hello world';
    while( shortCir( ptr )) do begin

        write( ptr^ );
        inc( ptr );

    end; // while
    writeln;

end.
```

现在考虑一下这个由 Borland 的 Delphi 编译器生成的 80x86 汇编代码（并通过 IDAPro 反汇编）：

```

			; function shortCir( thePtr:Pchar ):boolean
;
; Note: thePtr is passed into this function in
; the EAX register.

sub_408570  proc near

            ; EDX holds function return
            ; result (assume false).
            ;
            ; shortCir := false;

            xor     edx, edx

            ; if( thePtr <> NIL ) then begin

            test    eax, eax
            jz      short loc_40857C    ; branch if NIL

            ; shortCir := thePtr^ <> #0;

            cmp     byte ptr [eax], 0
            setnz   dl  ; DL = 1 if not #0

loc_40857C:

            ; Return result in EAX:

            mov     eax, edx
            retn
sub_408570  endp

; Main program (pertinent section):
;
; Load EBX with the address of the global "ptr" variable and
; then enter the "while" loop (Delphi moves the test for the
; while loop to the physical end of the loop's body):

                mov     ebx, offset loc_408628
                jmp     short loc_408617
; --------------------------------------------------------

loc_408600:
                ; Print the current character whose address
                ; "ptr" contains:

                mov     eax, ds:off_4092EC  ; ptr pointer
                mov     dl, [ebx]           ; fetch char
                call    sub_404523          ; print char
                call    sub_404391
                call    sub_402600

                inc     ebx                 ; inc( ptr )

; while( shortCir( ptr )) do ...

loc_408617:
                mov     eax, ebx         ; Pass ptr in EAX
                call    sub_408570       ; shortCir
                test    al, al           ; Returns true/false
                jnz     short loc_408600 ; branch if true
```

`sub_408570`过程包含计算类似于早期 C 代码中表达式的短路布尔求值的函数。正如你所看到的，如果`thePtr`包含 NIL（`0`），则解引用`thePtr`的代码永远不会执行。

如果函数调用不可行，那么唯一合理的解决方案就是使用非结构化的方法。以下是早期 C 代码中`while`循环的 Pascal 版本，强制短路布尔求值：

```

			    while( true ) do begin

        if( ptr = NIL ) then goto 2;
        if( ptr^.data = 0 ) then goto 2;
        << loop body >>
        ptr := ptr^.Next;

    end;
2:
```

再次强调，像本例中的非结构化代码，应该仅作为最后的手段来编写。但是，如果你使用的语言（或编译器）不保证短路求值，并且你需要这些语义，那么非结构化代码或低效代码（使用函数调用）可能是唯一的解决方案。

### 14.2 repeat..until（do..until/do..while）循环

另一个在大多数现代编程语言中常见的循环是`repeat..until`。这个循环在循环底部测试终止条件。这意味着循环体至少会执行一次，即使布尔控制表达式在循环的第一次迭代中评估为`false`。尽管`repeat..until`循环的适用范围比`while`循环小，你也不会像使用`while`循环那样频繁使用它，但在很多情况下，`repeat..until`循环是最适合的控制结构。经典的例子可能是读取用户输入，直到用户输入某个特定值。以下是一个典型的 Pascal 代码片段：

```

			repeat

      write( 'Enter a value (negative quits): ');
      readln( i );
      // do something with i's value

until( i < 0 );
```

这个循环总是执行一次循环体。当然，这是必要的，因为你必须执行循环体来读取用户输入的值，程序会检查这个值来判断循环何时结束。

`repeat..until`循环在布尔控制表达式评估为`true`时终止（而不是`false`，像`while`循环那样），正如词语*until*所暗示的那样。然而，值得注意的是，这是一个小的语法问题；C/C++/Java/Swift 语言（以及许多继承了 C 语言的语言）提供了`do..while`循环，它会在循环条件评估为`true`时重复执行循环体。从效率的角度来看，这两种循环完全没有区别，你可以通过使用语言的逻辑非操作符轻松将一种循环终止条件转换为另一种。以下示例演示了 Pascal、HLA 和 C/C++中的`repeat..until`和`do..while`循环的语法。以下是 Pascal 的`repeat..until`循环示例：

```

			repeat

    (* Read a raw character from the "input" file, which in this case is the keyboard *)

    ch := rawInput( input );

    (* Save the character away. *)

    inputArray[ i ] := ch;
    i := i + 1;

    (* Repeat until the user hits the enter key *)

until( ch = chr( 13 ));
```

现在，这里是相同循环的 C/C++ `do..while`版本：

```

			do
{
    /* Read a raw character from the "input" file, which in this case is the keyboard */

    ch = getKbd();

    /* Save the character away. */
 inputArray[ i++ ] = ch;

    /* Repeat until the user hits the enter key */
}
while( ch != '\r' );
```

这里是 HLA 的`repeat..until`循环：

```

			repeat

    // Read a character from the standard input device.

    stdin.getc();

    // Save the character away.

    mov( al, inputArray[ ebx ] );
    inc( ebx );

    // Repeat until the user hits the enter key.

until( al = stdio.cr );
```

将`repeat..until`（或`do..while`）循环转换为汇编语言相对简单直接。编译器只需要为布尔循环控制表达式替换代码，并在表达式为真时（对于`repeat..until`是`false`，对于`do..while`是`true`）跳转回循环体的开头。下面是早期 HLA `repeat..until`循环的直接纯汇编实现（C/C++和 Pascal 编译器对于其他示例会生成几乎相同的代码）：

```

			rptLoop:

    // Read a character from the standard input.

    call stdin.getc;

    // Store away the character.

    mov( al, inputArray[ ebx ] );
    inc( ebx );

    // Repeat the loop if the user did not hit
    // the enter key.

    cmp( al, stdio.cr );
    jne rptLoop;
```

正如你所看到的，典型编译器为`repeat..until`（或`do..while`）循环生成的代码通常比常规的`while`循环生成的代码更高效。因此，如果语义上可行，你应该考虑使用`repeat..until`/`do..while`形式。在许多程序中，布尔控制表达式在某些循环构造的第一次迭代时总是评估为`true`。例如，在应用程序中，遇到如下循环并不罕见：

```

			i = 0;
while( i < 100 )
{
      printf( "i: %d\n", i );
      i = i * 2 + 1;
      if( i < 50 )
      {
            i += j;
      }
}
```

这个`while`循环可以轻松地转换为如下的`do..while`循环：

```

			i = 0;
do
{
      printf( "i: %d\n", i );
      i = i * 2 + 1;
      if( i < 50 )
      {
            i += j;
      }
} while( i < 100 );
```

这种转换之所以可行，是因为我们知道`i`的初始值（`0`）小于`100`，因此循环体总是至少执行一次。

如你所见，通过使用更合适的`repeat..until`/`do..while`循环，而非常规的`while`循环，你可以帮助编译器生成更好的代码。然而，请记住，效率的提升很小，因此确保你不会因此牺牲可读性或可维护性。始终使用最合乎逻辑的循环结构。如果循环体总是至少执行一次，你应该使用`repeat..until`/`do..while`循环，即使`while`循环也同样有效。

#### 14.2.1 强制`repeat..until`循环中的完全布尔求值

由于在`repeat..until`（或`do..while`）循环中，测试循环终止发生在循环的底部，因此你可以像在`if`语句中一样强制完全布尔求值。考虑以下 C/C++代码：

```

			extern int x;
extern int y;
extern int f( int );
extern int g( int );
extern int a;
extern int b;
int main( void )
{

    do
        {
            ++a;
            --b;
        }while( a < f(x) && b > g(y));

    return( 0 );
}
```

下面是 GCC 为 PowerPC（使用短路求值，这是 C 的标准）生成的`do..while`循环的输出：

```

			L2:
        // ++a
        // --b

        lwz r9,0(r30)  ; get a
        lwz r11,0(r29) ; get b
        addi r9,r9,-1  ; --a
        lwz r3,0(r27)  ; Set up x parm for f
        stw r9,0(r30)  ; store back into a
        addi r11,r11,1 ; ++b
        stw r11,0(r29) ; store back into b

        ; compute f(x)

        bl L_f$stub    ; call f, result to R3

        ; is a >= f(x)? If so, quit loop

        lwz r0,0(r29)  ; get a
        cmpw cr0,r0,r3 ; Compare a with f's value
        bge- cr0,L3

        lwz r3,0(r28)  ; Set up y parm for g
        bl L_g$stub    ; call g

        lwz r0,0(r30)  ; get b
        cmpw cr0,r0,r3 ; Compare b with g's value
        bgt+ cr0,L2    ; Repeat if b > g's value
L3:
```

如果表达式`a < f(x)`为`false`（即`a >= f(x)`），这个程序会跳过`b > g(y)`的测试，直接跳转到标签`L3`。

为了强制完全布尔求值，我们的 C 源代码需要在`while`子句之前计算布尔表达式的子组件（将子表达式的结果保存在临时变量中），然后仅测试`while`子句中的结果：

```

			static int a;
static int b;

extern int x;
extern int y;
extern int f( int );
extern int g( int );

int main( void )
{
    int temp1;
    int temp2;

    do
        {
            ++a;
            --b;
            temp1 = a < f(x);
            temp2 = b > g(y);
        }while( temp1 && temp2 );

    return( 0 );
}
```

下面是 GCC 将代码转换为 PowerPC 的结果：

```

			L2:
        lwz r9,0(r30)    ; r9 = b
        li r28,1         ; temp1 = true
        lwz r11,0(r29)   ; r11 = a
        addi r9,r9,-1    ; --b
        lwz r3,0(r26)    ; r3 = x (set up f's parm)
        stw r9,0(r30)    ; Save b
        addi r11,r11,1   ; ++a
        stw r11,0(r29)   ; Save a
        bl L_f$stub      ; Call f
        lwz r0,0(r29)    ; Fetch a
        cmpw cr0,r0,r3   ; Compute temp1 = a < f(x)
        blt- cr0,L5      ; Leave temp1 true if a < f(x)
        li r28,0         ; temp1 = false
L5:
        lwz r3,0(r27)    ; r3 = y, set up g's parm
        bl L_g$stub      ; Call g
        li r9,1          ; temp2 = true
        lwz r0,0(r30)    ; Fetch b
        cmpw cr0,r0,r3   ; Compute b > g(y)
        bgt- cr0,L4      ; Leave temp2 true if b > g(y)
        li r9,0          ; Else set temp2 false
L4:
        ; Here's the actual termination test in
        ; the while clause:

        cmpwi cr0,r28,0
        beq- cr0,L3
        cmpwi cr0,r9,0
        bne+ cr0,L2
L3:
```

当然，实际的布尔表达式（`temp1 && temp2`）仍然使用短路求值，但仅针对所创建的临时变量。无论第一个子表达式的结果如何，循环都会计算两个原始子表达式。

#### 14.2.2 强制`repeat..until`循环中的短路布尔求值

如果你的编程语言提供了一种能够跳出`repeat..until`循环的功能，例如 C 语言的`break`语句，那么强制短路运算就非常简单。考虑前一部分中强制进行完全布尔运算的 C 语言`do..while`循环：

```

			do
{
    ++a;
    --b;
    temp1 = a < f(x);
    temp2 = b > g(y);

}while( temp1 && temp2 );
```

以下展示了一种转换代码的方式，使其使用短路布尔运算来评估终止表达式：

```

			static int a;
static int b;

extern int x;
extern int y;
extern int f( int );
extern int g( int );

int main( void )
{
    do
    {
        ++a;
        --b;

        if( !( a < f(x) )) break;
    } while( b > g(y) );

    return( 0 );
}
```

下面是 GCC 为 PowerPC 生成的`do..while`循环代码：

```

			L2:
        lwz r9,0(r30)   ; r9 = b
        lwz r11,0(r29)  ; r11 = a
        addi r9,r9,-1   ; --b
        lwz r3,0(r27)   ; Set up f(x) parm
        stw r9,0(r30)   ; Save b
        addi r11,r11,1  ; ++a
        stw r11,0(r29)  ; Save a
        bl L_f$stub     ; Call f

        ; break if !(a < f(x)):

        lwz r0,0(r29)
        cmpw cr0,r0,r3
        bge- cr0,L3

        ; while( b > g(y) ):

        lwz r3,0(r28)   ; Set up y parm
        bl L_g$stub     ; Call g
        lwz r0,0(r30)   ; Compute b > g(y)
        cmpw cr0,r0,r3
        bgt+ cr0,L2     ; Branch if true
L3:
```

如果`a`大于或等于`f(x)`返回的值，则该代码会立即跳出循环（在标签`L3`处），而不需要测试`b`是否大于`g(y)`返回的值。因此，这段代码模拟了 C/C++表达式`a < f(x) && b > g(y)`的短路布尔运算。

如果你使用的编译器不支持类似于 C/C++的`break`语句的语句，你将需要使用稍微复杂一点的逻辑。这里有一种方法：

```

			static int a;
static int b;

extern int x;
extern int y;
extern int f( int );
extern int g( int );

int main( void )
{
    int temp;

    do
    {
        ++a;
        --b;
        temp = a < f(x);
        if( temp )
        {
            temp = b > g(y);
        };
    }while( temp );

    return( 0 );
}
```

这是 GCC 为此示例生成的 PowerPC 代码：

```

			L2:
        lwz r9,0(r30)   ; r9 = b
        lwz r11,0(r29)  ; r11 = a
        addi r9,r9,-1   ; --b
        lwz r3,0(r27)   ; Set up f(x) parm
        stw r9,0(r30)   ; Save b
        addi r11,r11,1  ; ++a
        stw r11,0(r29)  ; Save a
        bl L_f$stub     ; Call f
        li r9,1         ; Assume temp is true
        lwz r0,0(r29)   ; Set temp false if
        cmpw cr0,r0,r3  ; a < f(x)
        blt- cr0,L5
        li r9,0
L5:
        cmpwi cr0,r9,0  ; If !(a < f(x)) then bail
        beq- cr0,L10    ; on the do..while loop
        lwz r3,0(r28)   ; Compute temp = b > f(y)
        bl L_g$stub     ; using a code sequence
        li r9,1         ; that is comparable to
        lwz r0,0(r30)   ; the above.
        cmpw cr0,r0,r3
        bgt- cr0,L9
        li r9,0
L9:
        ; Test the while termination expression:

        cmpwi cr0,r9,0
        bne+ cr0,L2
L10:
```

虽然这些示例使用了与运算符（逻辑与），但使用或运算符（逻辑或）同样简单。为了结束这一部分，考虑这个 Pascal 序列及其转换：

```

			repeat

      a := a + 1;
      b := b - 1;

until( (a < f(x)) OR (b > g(y)) );
```

这是强制进行完全布尔运算的转换：

```

			repeat

      a := a + 1;
      b := b - 1;
      temp := a < f(x);
      if( not temp ) then begin

            temp := b > g(y);

    end;
until( temp );
```

下面是 Borland Delphi 为这两个循环生成的代码（假设在编译器选项中选择了*完全布尔运算*）：

```

			;    repeat
;
;        a := a + 1;
;        b := b - 1;
;
;    until( (a < f(x)) or (b > g(y)));

loc_4085F8:
                inc     ebx                  ; a := a + 1;
                dec     esi                  ; b := b - 1;
                mov     eax, [edi]           ; EDI points at x
                call    locret_408570
                cmp     ebx, eax             ; Set AL to 1 if
                setl    al                   ; a < f(x)
                push    eax                  ; Save Boolean result.

                mov     eax, ds:dword_409288 ; y
                call    locret_408574        ; g(6)

                cmp     esi, eax             ; Set AL to 1 if
                setnle  al                   ; b > g(y)
                pop     edx                  ; Retrieve last value.
                or      dl, al               ; Compute their OR
                jz      short loc_4085F8     ; Repeat if false.

;    repeat
;
;        a := a + 1;
;        b := b - 1;
;        temp := a < f(x);
;        if( not temp ) then begin
;
;            temp := b > g(y);
;
;        end;
;
;    until( temp );
loc_40861B:
                inc     ebx                  ; a := a + 1;
                dec     esi                  ; b := b - 1;
                mov     eax, [edi]           ; Fetch x
                call    locret_408570        ; call f
                cmp     ebx, eax             ; is a < f(x)?
                setl    al                   ; Set AL to 1 if so.

            ; If the result of the above calculation is
            ; true, then don't bother with the second
            ; test (that is, short-circuit evaluation)

                test    al, al
                jnz     short loc_40863C

            ; Now check to see if b > g(y)

                mov     eax, ds:dword_409288
                call    locret_408574

            ; Set AL = 1 if b > g(y):

                cmp     esi, eax
                setnle  al

; Repeat loop if both conditions were false:

loc_40863C:
                test    al, al
                jz      short loc_40861B
```

Delphi 编译器为这种强制短路运算生成的代码，远不如它为你做这项工作的代码效果好。下面是未选中*完全布尔运算*选项的 Delphi 代码（即指示 Delphi 使用短路运算）：

```

			loc_4085F8:
                inc     ebx
                dec     esi
                mov     eax, [edi]
                call    nullsub_1 ;f
                cmp     ebx, eax
                jl      short loc_408613
                mov     eax, ds:dword_409288
                call    nullsub_2 ;g
                cmp     esi, eax
                jle     short loc_4085F8
```

尽管这种技巧在编译器不支持时很有用，可以强制短路运算，但这个 Delphi 示例再次强调，如果可能的话，应该使用编译器提供的功能——通常这样会生成更好的机器代码。

### 14.3 forever..endfor 循环

`while`循环在循环开始（顶部）时测试是否结束。`repeat..until`循环在循环结束（底部）时测试是否结束。唯一可以在循环体的中间测试循环终止的位置是循环体的某个位置。`forever..endfor`循环以及一些特殊的循环终止语句处理这种情况。

大多数现代编程语言提供了`while`循环和`repeat..until`循环（或它们的等效形式）。有趣的是，只有少数现代命令式编程语言提供了显式的`forever..endfor`循环。^(3) 这尤其令人惊讶，因为`forever..endfor`循环（以及一个循环终止测试）实际上是三种形式中最通用的一种。你可以轻松地从单个`forever..endfor`循环合成`while`循环或`repeat..until`循环。

幸运的是，在任何提供`while`循环或`repeat..until`/`do..while`循环的语言中，创建一个简单的`forever..endfor`循环都很容易。你只需要提供一个布尔控制表达式，对于`repeat..until`，它始终评估为`false`，对于`do..while`，它始终评估为`true`。例如，在 Pascal 中，你可以使用如下代码：

```

			const
    forever = true;
        .
        .
        .
    while( forever ) do begin

        << code to execute in an infinite loop >>

    end;
```

标准 Pascal 的一个大问题是，它没有提供一种机制（除了通用的`goto`语句）来显式地跳出循环。幸运的是，许多现代 Pascal 语言，如 Delphi 和 Free Pascal，提供了类似`break`的语句，可以立即退出当前的循环。

尽管 C/C++语言没有提供显式的语句来创建`forever`循环，但语法上奇怪的`for(;;)`语句自从第一个 C 编译器编写以来，就一直用于此目的。因此，C/C++程序员可以按如下方式创建`forever..endfor`循环：

```

			for(;;)
{
    << code to execute in an infinite loop >>
}
```

C/C++程序员可以使用 C 语言的`break`语句（与`if`语句一起）在循环中间设置一个循环终止条件，如下所示：

```

			for(;;)
{
    << Code to execute (at least once)
       prior to the termination test >>

    if( termination_expression ) break;

    << Code to execute after the loop termination test >>
}
```

HLA 语言提供了一个显式的（高级）`forever..endfor`语句（以及`break`和`breakif`语句），允许你在循环中途终止循环。这个 HLA 的`forever..endfor`循环在循环中间测试是否终止循环：

```

			forever

    << Code to execute (at least once) prior to
       the termination test >>

    breakif( termination_expression );

    << Code to execute after the loop termination test >>

endfor;
```

将`forever..endfor`循环转换为纯汇编语言是很简单的——你只需要一个`jmp`指令，它可以将控制从循环底部转移回循环顶部。`break`语句的实现也同样简单：它只是一个跳转（或条件跳转）到循环之后的第一条语句。以下两个代码片段展示了一个 HLA 的`forever..endfor`循环（以及一个`breakif`）和相应的“纯”汇编代码：

```

			// High-level forever statement in HLA:

forever

    stdout.put
    (
     "Enter an unsigned integer less than five:"
    );
    stdin.get( u );
    breakif( u < 5);
    stdout.put
    (
      "Error: the value must be between zero and five" nl
    );
endfor;

// Low-level coding of the forever loop in HLA:

foreverLabel:
    stdout.put
    (
      "Enter an unsigned integer less than five:"
    );
    stdin.get( u );
    cmp( u, 5 );
    jbe endForeverLabel;
    stdout.put
    (
      "Error: the value must be between zero and five" nl
    );
    jmp foreverLabel;

endForeverLabel:
```

当然，你也可以调整这段代码，创建一个稍微更高效的版本：

```

			// Low-level coding of the forever loop in HLA
// using code rotation:

jmp foreverEnter;
foreverLabel:
        stdout.put
        (
          "Error: the value must be between zero and five"
          nl
        );
    foreverEnter:
        stdout.put
        (
          "Enter an unsigned integer less "
          "than five:"
        );
        stdin.get( u );
        cmp( u, 5 );
        ja foreverLabel;
```

如果你使用的语言不支持`forever..endfor`循环，任何一款不错的编译器都会将`while(true)`语句转换为一个单一的跳转指令。如果你的编译器没有这么做，那它在优化方面做得很差，任何尝试手动优化代码的努力都是徒劳的。出于你很快就会明白的原因，你不应该尝试使用`goto`语句来创建`forever..endfor`循环。

#### 14.3.1 强制在`forever`循环中进行完整的布尔评估

因为你是通过`if`语句退出`forever`循环的，所以强制在退出`forever`循环时进行完整的布尔评估的技巧与在`if`语句中相同。有关详细信息，请参见第 465 页的《强制在`if`语句中进行完整布尔评估》。

#### 14.3.2 强制在`forever`循环中进行短路布尔评估

同样地，由于你是通过 `if` 语句退出 `forever` 循环，因此在退出 `forever` 循环时强制短路布尔运算的技巧与在 `repeat..until` 语句中的技巧相同。详细内容请参见 第 524 页 中的“在 `repeat..until` 循环中强制短路布尔运算”。

### 14.4 确定性循环（for 循环）

`forever..endfor` 循环是一个 *无限* 循环（假设你没有通过 `break` 语句跳出它）。`while` 和 `repeat..until` 循环是 *不确定* 循环的例子，因为通常情况下，程序无法在首次遇到它们时确定它们将执行多少次迭代。另一方面，对于 *确定* 循环，程序可以在执行循环体的第一条语句之前准确地确定循环将执行多少次迭代。传统高级语言中的一个确定性循环的好例子是 Pascal 的 `for` 循环，它使用以下语法：

```

			for variable := expr1 to expr2 do
        statement
```

当 `expr1` 小于或等于 `expr2` 时，它会遍历 `expr1..expr2` 范围，或者

```

			for variable := expr1 downto expr2 do
        statement
```

当 `expr1` 大于或等于 `expr2` 时，它会遍历 `expr1..expr2` 范围。以下是一个典型的 Pascal `for` 循环示例：

```

			for i := 1 to 10 do
    writeln( 'hello world');
```

这个循环总是精确执行 10 次；因此，它是一个确定性循环。然而，这并不意味着编译器必须能够在编译时确定循环迭代次数。确定性循环也允许使用表达式，强制程序在运行时确定迭代次数。例如：

```

			write( 'Enter an integer:');
readln( cnt );
for i := 1 to cnt do
    writeln( 'Hello World');
```

Pascal 编译器无法确定此循环将执行的迭代次数。事实上，由于迭代次数依赖于用户输入，它在一次程序执行中每次执行时都可能有所不同。然而，每当程序遇到此循环时，它可以确定循环将执行多少次迭代，这由 `cnt` 变量中的值指示。请注意，Pascal（像大多数支持确定性循环的语言一样）明确禁止如下代码：

```

			for i := 1 to j do begin

    << some statements >>
    i := <<some value>>;
    << some other statements >>

end;
```

在循环体执行过程中不允许更改循环控制变量的值。在这个例子中，如果你试图更改 `for` 循环的控制变量，一个高质量的 Pascal 编译器会检测到这个尝试并报告错误。此外，确定性循环只计算起始值和结束值一次。因此，如果 `for` 循环的体内修改了作为第二个表达式的变量，它不会在每次循环迭代时重新计算该表达式。例如，如果前面示例中的 `for` 循环体修改了 `j` 的值，这不会影响循环迭代次数。^(4)

确定性循环具有某些特殊属性，允许（好的）编译器生成更好的机器代码。特别是，因为编译器可以在执行循环体的第一条语句之前确定循环将执行多少次，它通常可以不需要复杂的循环终止测试，而直接将一个寄存器递减至`0`来控制循环的迭代次数。编译器还可以使用归纳法优化在确定性循环中对循环控制变量的访问（有关归纳法的描述请参见第 397 页的“算术语句优化”部分）。

C/C++/Java 用户应注意，这些语言中的`for`循环并不是一个真正的确定性循环；它只是一个不确定的`while`循环的特例。大多数优秀的 C/C++编译器会尝试确定一个`for`循环是否是确定性循环，如果是，它们会生成高效的代码。你可以通过遵循以下指导原则来帮助编译器：

+   你的 C/C++ `for`循环应使用与像 Pascal 这样的语言中的确定性(`for`)循环相同的语义。也就是说，`for`循环应初始化一个单一的循环控制变量，当该值小于或大于某个结束值时进行终止条件测试，并且将循环控制变量增减 1。

+   你的 C/C++ `for`循环不应在循环内修改循环控制变量的值。

+   循环终止条件的测试在循环体执行过程中保持静态。也就是说，循环体不应能够改变终止条件（这将使得循环变为不确定循环）。例如，如果循环终止条件是`i < j`，则循环体不应修改`i`或`j`的值。

+   循环体不会通过引用将循环控制变量或出现在循环终止条件中的任何变量传递给函数，如果该函数会修改实际参数。

### 14.5 更多信息

在第 501 页的“更多信息”部分同样适用于本章节。请参阅该部分以获取更多详细信息。
