# 第九章：数字转换

![](img/chapterart.png)

本章讨论了不同数字格式之间的转换，包括整数到十进制字符串、整数到十六进制字符串、浮点数到字符串、十六进制字符串到整数、十进制字符串到整数，以及实数字符串到浮点数。除了基本的转换外，本章还讨论了错误处理（对于字符串到数字的转换）和性能优化。本章讨论了标准精度转换（适用于 8 位、16 位、32 位和 64 位整数格式）以及扩展精度转换（例如，128 位整数和字符串转换）。

## 9.1 将数字值转换为字符串

到目前为止，本书依赖于 C 标准库来执行数字输入输出（将数字数据写入显示器并从用户读取数字数据）。然而，C 标准库没有提供扩展精度的数字输入输出功能（甚至 64 位数字输入输出也有问题；本书使用了 Microsoft 扩展的`printf()`来进行 64 位数字输出）。因此，现在是时候解析并讨论如何在汇编语言中进行数字输入输出了——嗯，算是吧。因为大多数操作系统仅支持字符或字符串输入输出，我们不会进行实际的数字输入输出。相反，我们将编写将数字值与字符串之间转换的函数，然后进行字符串输入输出。

本节中的示例专门处理 64 位（非扩展精度）和 128 位值，但算法是通用的，可以扩展到任何位数。

### 9.1.1 将数字值转换为十六进制字符串

将数值转换为十六进制字符串相对简单。只需将二进制表示中的每个半字节（4 位）转换为“0”到“9”或“A”到“F”中的一个字符。请参考清单 9-1 中的`btoh`函数，该函数接收 AL 寄存器中的一个字节，并返回 AH（高半字节）和 AL（低半字节）中的两个对应字符。

```
; btoh - This procedure converts the binary value
;        in the AL register to two hexadecimal
;        characters and returns those characters
;        in the AH (HO nibble) and AL (LO nibble)
;        registers.

btoh        proc

            mov     ah, al      ; Do HO nibble first
            shr     ah, 4       ; Move HO nibble to LO
            or      ah, '0'     ; Convert to char
            cmp     ah, '9' + 1 ; Is it "A" through "F"?
            jb      AHisGood

; Convert 3Ah to 3Fh to "A" through "F":

            add     ah, 7

; Process the LO nibble here:

AHisGood:   and     al, 0Fh     ; Strip away HO nibble
            or      al, '0'     ; Convert to char
            cmp     al, '9' + 1 ; Is it "A" through "F"?
            jb      ALisGood

; Convert 3Ah to 3Fh to "A" through "F":

            add     al, 7
ALisGood:   ret
btoh        endp
```

清单 9-1：一个将字节转换为两个十六进制字符的函数

你可以通过将数值与 0（30h）进行按位或运算，将 0 到 9 范围内的任何数值转换为相应的 ASCII 字符。不幸的是，这会将 0Ah 到 0Fh 的数值映射到 3Ah 到 3Fh。因此，清单 9-1 中的代码会检查其是否产生大于 3Ah 的值，并加上 7，以生成最终的字符代码，范围是 41h 到 46h（“A”到“F”）。

一旦我们能够将单个字节转换为一对十六进制字符，创建一个字符串并输出到显示器就变得简单了。我们可以对数字中的每个字节调用`btoh`（*字节到十六进制*）函数，并将相应的字符存储在字符串中。清单 9-2 提供了`btoStr`（*字节到字符串*）、`wtoStr`（*字词到字符串*）、`dtoStr`（*双字到字符串*）和`qtoStr`（*四字到字符串*）函数的示例。

```
; Listing 9-2

; Numeric-to-hex string functions.

        option  casemap:none

nl          =       10

            .const
ttlStr      byte    "Listing 9-2", 0
fmtStr1     byte    "btoStr: Value=%I64x, string=%s"
            byte    nl, 0

fmtStr2     byte    "wtoStr: Value=%I64x, string=%s"
            byte    nl, 0

fmtStr3     byte    "dtoStr: Value=%I64x, string=%s"
            byte    nl, 0

fmtStr4     byte    "qtoStr: Value=%I64x, string=%s"
            byte    nl, 0

            .data
buffer      byte    20 dup (?)

            .code
            externdef printf:proc

; Return program title to C++ program:

            public  getTitle
getTitle    proc
            lea     rax, ttlStr
 ret
getTitle    endp

; btoh - This procedure converts the binary value
;        in the AL register to two hexadecimal
;        characters and returns those characters
;        in the AH (HO nibble) and AL (LO nibble)
;        registers.

btoh        proc

            mov     ah, al      ; Do HO nibble first
            shr     ah, 4       ; Move HO nibble to LO
            or      ah, '0'     ; Convert to char
            cmp     ah, '9' + 1 ; Is it "A" to "F"?
            jb      AHisGood

; Convert 3Ah through 3Fh to "A" to "F":

            add     ah, 7

; Process the LO nibble here:

AHisGood:   and     al, 0Fh     ; Strip away HO nibble
            or      al, '0'     ; Convert to char
            cmp     al, '9' + 1 ; Is it "A" to "F"?
            jb      ALisGood

; Convert 3Ah through 3Fh to "A" to "F":

            add     al, 7   
ALisGood:   ret

btoh        endp

; btoStr - Converts the byte in AL to a string of hexadecimal
;          characters and stores them at the buffer pointed at
;          by RDI. Buffer must have room for at least 3 bytes.
;          This function zero-terminates the string.

btoStr      proc
            push    rax
            call    btoh        ; Do conversion here

; Create a zero-terminated string at [RDI] from the
; two characters we converted to hex format:

            mov     [rdi], ah
            mov     [rdi + 1], al
            mov     byte ptr [rdi + 2], 0
            pop     rax
            ret
btoStr      endp

; wtoStr - Converts the word in AX to a string of hexadecimal
;          characters and stores them at the buffer pointed at
;          by RDI. Buffer must have room for at least 5 bytes.
;          This function zero-terminates the string.

wtoStr      proc
            push    rdi
            push    rax     ; Note: leaves LO byte at [RSP]

; Use btoStr to convert HO byte to a string:

            mov     al, ah
            call    btoStr

            mov     al, [rsp]       ; Get LO byte
            add     rdi, 2          ; Skip HO chars
            call    btoStr

            pop     rax
            pop     rdi
            ret
wtoStr      endp

; dtoStr - Converts the dword in EAX to a string of hexadecimal
;          characters and stores them at the buffer pointed at
;          by RDI. Buffer must have room for at least 9 bytes.
;          This function zero-terminates the string.

dtoStr      proc
            push    rdi
            push    rax     ; Note: leaves LO word at [RSP]

; Use wtoStr to convert HO word to a string:

            shr     eax, 16
            call    wtoStr

            mov     ax, [rsp]       ; Get LO word
            add     rdi, 4          ; Skip HO chars
            call    wtoStr

            pop     rax
            pop     rdi
            ret
dtoStr      endp

; qtoStr - Converts the qword in RAX to a string of hexadecimal
;          characters and stores them at the buffer pointed at
;          by RDI. Buffer must have room for at least 17 bytes.
;          This function zero-terminates the string.

qtoStr      proc
            push    rdi
            push    rax     ; Note: leaves LO dword at [RSP]

; Use dtoStr to convert HO dword to a string:

            shr     rax, 32
            call    dtoStr

            mov     eax, [rsp]      ; Get LO dword
            add     rdi, 8          ; Skip HO chars
            call    dtoStr

            pop     rax
            pop     rdi
            ret
qtoStr      endp

; Here is the "asmMain" function.

            public  asmMain
asmMain     proc
            push    rdi
            push    rbp
            mov     rbp, rsp
            sub     rsp, 64         ; Shadow storage

; Because all the (`x`)toStr functions preserve RDI,
; we need to do the following only once:

            lea     rdi, buffer

; Demonstrate call to btoStr:

            mov     al, 0aah
            call    btoStr

            lea     rcx, fmtStr1
            mov     edx, eax
            mov     r8, rdi
            call    printf

; Demonstrate call to wtoStr:

            mov     ax, 0a55ah
            call    wtoStr

            lea     rcx, fmtStr2
            mov     edx, eax
            mov     r8, rdi
            call    printf

; Demonstrate call to dtoStr:

            mov     eax, 0aa55FF00h
            call    dtoStr

            lea     rcx, fmtStr3
            mov     edx, eax
 mov     r8, rdi
            call    printf

; Demonstrate call to qtoStr:

            mov     rax, 1234567890abcdefh
            call    qtoStr

            lea     rcx, fmtStr4
            mov     rdx, rax
            mov     r8, rdi
            call    printf

            leave
            pop     rdi
            ret     ; Returns to caller

asmMain     endp
            end
```

清单 9-2：`btoStr`、`wtoStr`、`dtoStr`和`qtoStr`函数

这是构建命令和示例输出：

```
C:\>**build listing9-2**

C:\>**echo off**
 Assembling: listing9-2.asm
c.cpp

C:\>**listing9-2**
Calling Listing 9-2:
btoStr: Value=aa, string=AA
wtoStr: Value=a55a, string=A55A
dtoStr: Value=aa55ff00, string=AA55FF00
qtoStr: Value=1234567890abcdef, string=1234567890ABCDEF
Listing 9-2 terminated
```

清单 9-2 中的每个后续函数都建立在前一个函数的基础上。例如，`wtoStr`调用`btoStr`两次，将 AX 中的 2 个字节转换为 4 个十六进制字符的字符串。如果你在每个调用这些函数的地方都内联展开它们，代码会更快（但也会变得更大）。如果你只需要*其中一个*函数，内联展开它的所有调用会值得付出额外的努力。

这是`qtoStr`的一个版本，包含两个改进：内联展开对`dtoStr`、`wtoStr`和`btoStr`的调用，以及使用一个简单的表查找（数组访问）来进行半字节到十六进制字符的转换（有关表查找的更多信息，请参见第十章）。这个更快版本的`qtoStr`的框架出现在清单 9-3 中。

```
; qtoStr - Converts the qword in RAX to a string of hexadecimal
;          characters and stores them at the buffer pointed at
;          by RDI. Buffer must have room for at least 17 bytes.
;          This function zero-terminates the string.

hexChar             byte    "0123456789ABCDEF"

qtoStr      proc
            push    rdi
            push    rcx
            push    rdx
            push    rax                ; Leaves LO dword at [RSP]

            lea     rcx, hexChar

            xor     edx, edx           ; Zero-extends!
            shld    rdx, rax, 4
            shl     rax, 4
            mov     dl, [rcx][rdx * 1] ; Table lookup
            mov     [rdi], dl

; Emit bits 56-59:

            xor     edx, edx
            shld    rdx, rax, 4
            shl     rax, 4
            mov     dl, [rcx][rdx * 1]
            mov     [rdi + 1], dl

; Emit bits 52-55:

            xor     edx, edx
            shld    rdx, rax, 4
            shl     rax, 4
            mov     dl, [rcx][rdx * 1]
            mov     [rdi + 2], dl
             .
             .
             .
 `Code to emit bits 8-51 was deleted for length reasons.`
 `The code should be obvious if you look at the output`
 `for the other nibbles appearing here.` 
             .
             .
             .
; Emit bits 4-7:

            xor     edx, edx
            shld    rdx, rax, 4
            shl     rax, 4
            mov     dl, [rcx][rdx * 1]
            mov     [rdi + 14], dl

; Emit bits 0-3:

            xor     edx, edx
            shld    rdx, rax, 4
            shl     rax, 4
            mov     dl, [rcx][rdx * 1]
            mov     [rdi + 15], dl

; Zero-terminate string:

            mov     byte ptr [rdi + 16], 0

            pop     rax
            pop     rdx
            pop     rcx
            pop     rdi
            ret
qtoStr      endp
```

清单 9-3：`qtoStr`的更快实现

编写一个简短的主程序，包含以下循环

```
 lea     rdi, buffer
            mov     rax, 07fffffffh
loopit:     call    qtoStr
            dec     eax
            jnz     loopit
```

然后，我使用一台 2012 年款的 2.6 GHz Intel Core i7 处理器，通过秒表得到了`qtoStr`内联版本和原始版本的大致执行时间：

+   内联版本：19 秒

+   原始版本：85 秒

如你所见，内联版本显著（快了四倍）更快，但你可能不会经常将 64 位数字转换为十六进制字符串，因此不足以为内联版本那种不够简洁的代码辩护。

说实话，你可能通过使用一个更大的表（256 个 16 位条目）来表示十六进制字符，并一次转换一个字节，而不是一个半字节，从而将时间几乎减少一半。这将需要比内联版本少一半的指令（尽管表的大小将增加 32 倍）。

### 9.1.2 将扩展精度十六进制值转换为字符串

扩展精度的十六进制到字符串的转换非常简单。它只是上一节中正常十六进制转换例程的扩展。例如，这里是一个 128 位的十六进制转换函数：

```
; otoStr - Converts the oword in RDX:RAX to a string of hexadecimal
;          characters and stores them at the buffer pointed at
;          by RDI. Buffer must have room for at least 33 bytes.
;          This function zero-terminates the string.

otoStr      proc
            push    rdi
            push    rax     ; Note: leaves LO dword at [RSP]

; Use qtoStr to convert each qword to a string:

            mov     rax, rdx
            call    qtoStr

            mov     rax, [rsp]      ; Get LO qword
            add     rdi, 16         ; Skip HO chars
            call    qtoStr

            pop     rax
            pop     rdi
            ret
otoStr      endp
```

### 9.1.3 将无符号十进制值转换为字符串

十进制输出比十六进制输出稍微复杂一些，因为二进制数字的高位（HO 位）会影响十进制表示中的低位数字（十六进制值并不受此影响，这也是为什么十六进制输出如此简单的原因）。因此，我们需要通过从数字中提取每一位十进制数字，来创建二进制数的十进制表示。

输出无符号十进制数的最常见方法是不断地将值除以 10，直到结果变为 0。第一次除法后的余数是一个 0 到 9 之间的数值，这个值对应十进制数的低位数字。通过连续除以 10（以及对应的余数），可以提取数字的每一位。

对这个问题的迭代解决方案通常会分配足够大的存储空间来容纳整个数字的字符字符串。然后，代码在循环中提取十进制数字，并将它们逐一放入字符串中。在转换过程结束时，例程会以相反的顺序打印字符串中的字符（记住，除法算法先提取低位数字，最后提取高位数字，这与你需要打印的顺序正好相反）。

本节采用了*递归解决方案*，因为它稍微更优雅一些。该解决方案首先通过将值除以 10 并将余数保存在局部变量中开始。如果商不为 0，例程会递归调用自己，先输出所有前导数字。递归调用返回后（输出了所有前导数字），递归算法会输出与余数相关的数字，完成操作。当打印十进制值 789 时，操作过程如下：

1.  将 789 除以 10。商为 78，余数为 9。

1.  将余数（9）保存在一个局部变量中，并递归地调用该例程，使用商值作为参数。

1.  *递归入口 1*：将 78 除以 10。商为 7，余数为 8。

1.  将余数（8）保存在局部变量中，并递归地调用该例程，使用商值作为参数。

1.  *递归入口 2*：将 7 除以 10。商为 0，余数为 7。

1.  将余数（7）保存在局部变量中。由于商为 0，不再递归调用例程。

1.  输出保存在局部变量中的余数值（7）。返回到调用者（递归入口 1）。

1.  *返回到递归入口 1*：输出在递归入口 1 中保存在局部变量中的余数值（8）。返回到调用者（原始例程调用）。

1.  *原始调用*：输出原始调用中保存在局部变量中的余数值（9）。返回到输出例程的原始调用者。

列表 9-4 实现了递归算法。

```
; Listing 9-4

; Numeric unsigned integer-to-string function.

        option  casemap:none

nl          =       10

            .const
ttlStr      byte    "Listing 9-4", 0
fmtStr1     byte    "utoStr: Value=%I64u, string=%s"
            byte    nl, 0

            .data
buffer      byte    24 dup (?)

            .code
            externdef printf:proc

; Return program title to C++ program:

            public  getTitle
getTitle    proc
            lea     rax, ttlStr
            ret
getTitle    endp

; utoStr - Unsigned integer to string.

; Inputs:

;    RAX:   Unsigned integer to convert.
;    RDI:   Location to hold string.

; Note: for 64-bit integers, resulting
; string could be as long as 21 bytes
; (including the zero-terminating byte).

utoStr      proc
            push    rax
            push    rdx
            push    rdi

; Handle zero specially:

            test    rax, rax
            jnz     doConvert

            mov     byte ptr [rdi], '0'
            inc     rdi
            jmp     allDone 

doConvert:  call    rcrsvUtoStr

; Zero-terminate the string and return:

allDone:    mov     byte ptr [rdi], 0
            pop     rdi
            pop     rdx
            pop     rax
            ret
utoStr      endp

ten         qword   10

; Here's the recursive code that does the
; actual conversion:

rcrsvUtoStr proc

            xor     rdx, rdx           ; Zero-extend RAX -> RDX
            div     ten
            push    rdx                ; Save output value
            test    eax, eax           ; Quit when RAX is 0
            jz      allDone 

; Recursive call to handle value % 10:

            call    rcrsvUtoStr

allDone:    pop     rax                ; Retrieve char to print
            and     al, 0Fh            ; Convert to "0" to "9"
            or      al, '0'
            mov     byte ptr [rdi], al ; Save in buffer
            inc     rdi                ; Next char position
            ret
rcrsvUtoStr endp

; Here is the "asmMain" function.

            public  asmMain
asmMain     proc
            push    rdi
 push    rbp
            mov     rbp, rsp
            sub     rsp, 56         ; Shadow storage

; Because all the (`x`)toStr functions preserve RDI,
; we need to do the following only once:

            lea     rdi, buffer
            mov     rax, 1234567890
            call    utoStr

; Print the result:

            lea     rcx, fmtStr1
            mov     rdx, rax
            mov     r8, rdi
            call    printf

            leave
            pop     rdi
            ret     ; Returns to caller

asmMain     endp
            end
```

列表 9-4：无符号整数到字符串的转换函数（递归）

这是构建命令和程序输出：

```
C:\>**build listing9-4**

C:\>**echo off**
 Assembling: listing9-4.asm
c.cpp

C:\>**listing9-4**
Calling Listing 9-4:
utoStr: Value=1234567890, string=1234567890
Listing 9-4 terminated
```

与十六进制输出不同，实际上没有必要提供字节大小、字大小或双字大小的数字到十进制字符串的转换函数。只需要将较小的值零扩展到 64 位即可。与十六进制转换不同，`qtoStr` 函数不会输出前导零，因此对于所有大小的变量（64 位及以下），输出是相同的。

与十六进制转换（本身就非常快速，而且你也不常用它）不同，整数到字符串的转换函数你会频繁调用。因为它使用了 `div` 指令，所以可能会比较慢。幸运的是，我们可以通过使用 `fist` 和 `fbstp` 指令来加速它。

`fbstp` 指令将当前位于栈顶的 80 位浮点值转换为一个 18 位的打包 BCD 值（采用第六章中 图 6-7 所示的格式）。`fist` 指令允许将一个 64 位整数加载到 FPU 栈上。因此，通过使用这两个指令，你可以（大部分）将一个 64 位整数转换为打包 BCD 值，该值每 4 位编码一个十进制数字。因此，你可以使用将十六进制数字转换为字符串的相同算法，将 `fbstp` 产生的打包 BCD 结果转换为字符字符串。

使用 `fist` 和 `fbstp` 将整数转换为字符串时，有一个小问题：Intel 打包 BCD 格式（见第六章中的 图 6-7）仅支持 18 位，而 64 位整数最多可以有 19 位。因此，任何基于 `fbstp` 的 `utoStr` 函数都必须处理第 19 位作为特殊情况。考虑到这一点，清单 9-5 提供了这个新的 `utoStr` 函数版本。

```
; Listing 9-5

; Fast unsigned integer-to-string function
; using fist and fbstp.

        option  casemap:none

nl          =       10

            .const
ttlStr      byte    "Listing 9-5", 0
fmtStr1     byte    "utoStr: Value=%I64u, string=%s"
            byte    nl, 0

            .data
buffer      byte    30 dup (?)

            .code
            externdef printf:proc

; Return program title to C++ program:

            public  getTitle
getTitle    proc
            lea     rax, ttlStr
            ret
getTitle    endp

; utoStr - Unsigned integer to string.

; Inputs:

;    RAX:   Unsigned integer to convert.
;    RDI:   Location to hold string.

; Note: for 64-bit integers, resulting
; string could be as long as 21 bytes
; (including the zero-terminating byte).

bigNum      qword   1000000000000000000
utoStr      proc
            push    rcx
            push    rdx
            push    rdi
            push    rax
            sub     rsp, 10

; Quick test for zero to handle that special case:

            test    rax, rax
            jnz     not0
            mov     byte ptr [rdi], '0'
            jmp     allDone

; The FBSTP instruction supports only 18 digits.
; 64-bit integers can have up to 19 digits.
; Handle that 19th possible digit here:

not0:       cmp     rax, bigNum
            jb      lt19Digits

; The number has 19 digits (which can be 0-9).
; Pull off the 19th digit:

            xor     edx, edx
            div     bigNum            ; 19th digit in AL
            mov     [rsp + 10], rdx   ; Remainder
            or      al, '0'
            mov     [rdi], al
            inc     rdi

; The number to convert is nonzero.
; Use BCD load and store to convert
; the integer to BCD:

lt19Digits: fild    qword ptr [rsp + 10]
            fbstp   tbyte ptr [rsp]

; Begin by skipping over leading zeros in
; the BCD value (max 19 digits, so the most
; significant digit will be in the LO nibble
; of DH).

            mov     dx, [rsp + 8]
            mov     rax, [rsp]
            mov     ecx, 20
            jmp     testFor0

Skip0s:     shld    rdx, rax, 4
            shl     rax, 4
testFor0:   dec     ecx         ; Count digits we've processed
            test    dh, 0fh     ; Because the number is not 0
            jz      Skip0s      ; this always terminates

; At this point the code has encountered
; the first nonzero digit. Convert the remaining
; digits to a string:

cnvrtStr:   and     dh, 0fh
            or      dh, '0'
            mov     [rdi], dh
            inc     rdi
            mov     dh, 0
            shld    rdx, rax, 4
            shl     rax, 4
            dec     ecx
            jnz     cnvrtStr

; Zero-terminate the string and return:

allDone:    mov     byte ptr [rdi], 0
            add     rsp, 10
            pop     rax
            pop     rdi
            pop     rdx
            pop     rcx
            ret
utoStr      endp

; Here is the "asmMain" function.

            public  asmMain
asmMain     proc
            push    rbp
            mov     rbp, rsp
            sub     rsp, 64         ; Shadow storage

; Because all the (`x`)toStr functions preserve RDI,
; we need to do the following only once:

            lea     rdi, buffer
            mov     rax, 9123456789012345678
            call    utoStr

            lea     rcx, fmtStr1
            mov     rdx, 9123456789012345678
            lea     r8, buffer
            call    printf

            leave
            ret     ; Returns to caller
asmMain     endp
            end
```

清单 9-5：基于 `fist` 和 `fbstp` 的 `utoStr` 函数

这是该程序的构建命令和示例输出：

```
C:\>**build listing9-5**

C:\>**echo off**
 Assembling: listing9-5.asm
c.cpp

C:\>**listing9-5**
Calling Listing 9-5:
utoStr: Value=9123456789012345678, string=9123456789012345678
Listing 9-5 terminated
```

清单 9-5 中的程序确实使用了 `div` 指令，但它仅执行一到两次，而且仅当数字中有 19 或 20 位时才会执行。因此，这个 `div` 指令的执行时间对 `utoStr` 函数的整体速度影响很小（尤其是在你考虑到实际打印 19 位数字的频率时）。

我在一台 2.6 GHz 的 2012 年左右的 Core i7 处理器上得到了以下执行时间：

+   原始 `utoStr`：108 秒

+   `fist` 和 `fbstp` 实现：11 秒

显然，`fist` 和 `fbstp` 的实现是赢家。

### 9.1.4 带符号整数值转换为字符串

要将带符号整数值转换为字符串，首先检查该数字是否为负数；如果是，则输出一个连字符（-）并取其绝对值。然后调用 `utoStr` 函数完成剩余的转换。清单 9-6 显示了相关代码。

```
; itoStr - Signed integer-to-string conversion.

; Inputs:
;    RAX -   Signed integer to convert.
;    RDI -   Destination buffer address.

itoStr      proc
            push    rdi
            push    rax
            test    rax, rax
            jns     notNeg

; Number was negative, emit "-" and negate
; value.

 mov     byte ptr [rdi], '-'
            inc     rdi
            neg     rax

; Call utoStr to convert non-negative number:

notNeg:     call    utoStr
            pop     rax
            pop     rdi
            ret
itoStr      endp
```

清单 9-6：带符号整数到字符串转换

### 9.1.5 扩展精度无符号整数转换为字符串

对于扩展精度输出，整个字符串转换算法中唯一需要扩展精度运算的操作是除以 10 操作。因为我们要用扩展精度值除以一个轻松适配到四字单元的值，我们可以使用快速（且简单的）扩展精度除法算法，采用 `div` 指令（详见第八章中的《使用 `div` 指令的特殊情况形式》部分）。清单 9-7 实现了一个使用该技术的 128 位十进制输出例程。

```
; Listing 9-7

; Extended-precision numeric unsigned 
; integer-to-string function.

        option  casemap:none

nl          =       10

            .const
ttlStr      byte    "Listing 9-7", 0
fmtStr1     byte    "otoStr(0): string=%s", nl, 0
fmtStr2     byte    "otoStr(1234567890): string=%s", nl, 0
fmtStr3     byte    "otoStr(2147483648): string=%s", nl, 0
fmtStr4     byte    "otoStr(4294967296): string=%s", nl, 0
fmtStr5     byte    "otoStr(FFF...FFFF): string=%s", nl, 0

            .data
buffer      byte    40 dup (?)

b0          oword   0
b1          oword   1234567890
b2          oword   2147483648
b3          oword   4294967296

; Largest oword value
; (decimal=340,282,366,920,938,463,463,374,607,431,768,211,455):

b4          oword   0FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFh

 .code
            externdef printf:proc

; Return program title to C++ program:

            public  getTitle
getTitle    proc
            lea     rax, ttlStr
            ret
getTitle    endp

; DivideBy10 - Divides "divisor" by 10 using fast
;              extended-precision division algorithm
;              that employs the div instruction.

; Returns quotient in "quotient."
; Returns remainder in RAX.
; Trashes RDX.

; RCX - Points at oword dividend and location to
;       receive quotient.

ten         qword   10

DivideBy10  proc
parm        equ     <[rcx]>

            xor     edx, edx       ; Zero-extends!
            mov     rax, parm[8]
            div     ten
            mov     parm[8], rax

            mov     rax, parm
            div     ten
            mov     parm, rax
            mov     eax, edx       ; Remainder (always "0" to "9"!)
            ret    
DivideBy10  endp

; Recursive version of otoStr.
; A separate "shell" procedure calls this so that
; this code does not have to preserve all the registers
; it uses (and DivideBy10 uses) on each recursive call.

; On entry:
;    Stack - Contains oword in/out parameter (dividend in/quotient out).
;    RDI   - Contains location to place output string.

; Note: this function must clean up stack (parameters)
;       on return.

rcrsvOtoStr proc
value       equ     <[rbp + 16]>
remainder   equ     <[rbp - 8]>
            push    rbp
 mov     rbp, rsp
            sub     rsp, 8
            lea     rcx, value
            call    DivideBy10
            mov     remainder, al

; If the quotient (left in value) is not 0, recursively
; call this routine to output the HO digits.

            mov     rax, value
            or      rax, value[8]
            jz      allDone

            mov     rax, value[8]
            push    rax
            mov     rax, value
            push    rax
            call    rcrsvOtoStr

allDone:    mov     al, remainder
            or      al, '0'
            mov     [rdi], al
            inc     rdi
            leave
            ret     16      ; Remove parms from stack
rcrsvOtoStr endp

; Nonrecursive shell to the above routine so we don't bother
; saving all the registers on each recursive call.

; On entry:

;   RDX:RAX - Contains oword to print.
;   RDI     - Buffer to hold string (at least 40 bytes).

otostr      proc

            push    rax
            push    rcx
            push    rdx
            push    rdi

; Special-case zero:

            test    rax, rax
            jnz     not0
            test    rdx, rdx
            jnz     not0
            mov     byte ptr [rdi], '0'
            inc     rdi
            jmp     allDone

not0:       push    rdx
 push    rax
            call    rcrsvOtoStr

; Zero-terminate string before leaving:

allDone:    mov     byte ptr [rdi], 0

            pop     rdi
            pop     rdx
            pop     rcx
            pop     rax
            ret

otostr      endp

; Here is the "asmMain" function.

            public  asmMain
asmMain     proc
            push    rdi
            push    rbp
            mov     rbp, rsp
            sub     rsp, 56         ; Shadow storage

; Because all the (`x`)toStr functions preserve RDI,
; we need to do the following only once:

            lea     rdi, buffer

; Convert b0 to a string and print the result:

            mov     rax, qword ptr b0
            mov     rdx, qword ptr b0[8]
            call    otostr

            lea     rcx, fmtStr1
            lea     rdx, buffer
            call    printf

; Convert b1 to a string and print the result:

            mov     rax, qword ptr b1
            mov     rdx, qword ptr b1[8]
            call    otostr

            lea     rcx, fmtStr2
            lea     rdx, buffer
            call    printf

; Convert b2 to a string and print the result:

            mov     rax, qword ptr b2
            mov     rdx, qword ptr b2[8]
 call    otostr

            lea     rcx, fmtStr3
            lea     rdx, buffer
            call    printf

; Convert b3 to a string and print the result:

            mov     rax, qword ptr b3
            mov     rdx, qword ptr b3[8]
            call    otostr

            lea     rcx, fmtStr4
            lea     rdx, buffer
            call    printf

; Convert b4 to a string and print the result:

            mov     rax, qword ptr b4
            mov     rdx, qword ptr b4[8]
            call    otostr

            lea     rcx, fmtStr5
            lea     rdx, buffer
            call    printf

            leave
            pop     rdi
            ret     ; Returns to caller

asmMain     endp
            end
```

清单 9-7：128 位扩展精度十进制输出例程

这是构建命令和程序输出：

```
C:\>**build listing9-7**

C:\>**echo off**
 Assembling: listing9-7.asm
c.cpp

C:\>**listing9-7**
Calling Listing 9-7:
otoStr(0): string=0
otoStr(1234567890): string=1234567890
otoStr(2147483648): string=2147483648
otoStr(4294967296): string=4294967296
otoStr(FFF...FFFF):
        string=340282366920938463463374607431768211455
Listing 9-7 terminated
```

可惜，我们不能使用`fbstp`指令来提高该算法的性能，因为`fbstp`仅限于 80 位 BCD 值。

### 9.1.6 将扩展精度有符号十进制值转换为字符串

一旦你有了扩展精度无符号十进制输出例程，编写扩展精度有符号十进制输出例程就很简单了。基本算法与之前给出的 64 位整数类似：

1.  检查数字的符号。

1.  如果是正数，调用无符号输出例程打印它。如果是负数，则打印一个负号。然后将该数字取反，并调用无符号输出例程打印它。

要检查扩展精度整数的符号，请测试数字的 HO 位。为了取反一个大数，最好的解决方案可能是从 0 中减去该值。列表 9-8 是一个快速版的`i128toStr`，它使用了上一节中的`otoStr`例程。

```
; i128toStr - Converts a 128-bit signed integer to a string.

; Inputs:
;    RDX:RAX - Signed integer to convert.
;    RDI     - Pointer to buffer to receive string.

i128toStr   proc
            push    rax
            push    rdx
            push    rdi

            test    rdx, rdx  ; Is number negative?
            jns     notNeg

            mov     byte ptr [rdi], '-'
            inc     rdi
            neg     rdx       ; 128-bit negation
            neg     rax
            sbb     rdx, 0

notNeg:     call    otostr
            pop     rdi
            pop     rdx
            pop     rax
            ret
i128toStr   endp
```

列表 9-8：128 位有符号整数到字符串的转换

### 9.1.7 格式化转换

前面部分的代码通过使用最少的必要字符位置将有符号和无符号整数转换为字符串。为了创建格式化良好的值表，你需要编写在输出数字之前为数字字符串提供适当填充的函数。一旦你有了这些例程的“未格式化”版本，实现格式化版本就很容易了。

第一步是编写`iSize`和`uSize`例程，计算显示值所需的最小字符位置数。实现此目标的一个算法类似于数字字符串转换例程。实际上，唯一的区别是进入例程时初始化一个计数器为 0（例如，非递归外壳例程），然后在每次递归调用时增加此计数器，而不是输出一个数字。（不要忘记在数字为负时在`iSize`中增加计数器；你必须为输出负号留出空间。）计算完成后，这些例程应该将操作数的大小返回到 EAX 寄存器。

唯一的问题是这种转换方案速度较慢（使用递归和`div`并不是很快）。事实证明，一个简单的暴力版本，通过将整数值与 1、10、100、1000 等进行比较，运行得要快得多。以下是实现这一点的代码：

```
; uSize - Determines how many character positions it will take
;         to hold a 64-bit numeric-to-string conversion.

; Input:
;   RAX -    Number to check.

; Returns:
;   RAX -    Number of character positions required.

dig2        qword   10
dig3        qword   100
dig4        qword   1000
dig5        qword   10000
dig6        qword   100000
dig7        qword   1000000
dig8        qword   10000000
dig9        qword   100000000
dig10       qword   1000000000
dig11       qword   10000000000
dig12       qword   100000000000
dig13       qword   1000000000000
dig14       qword   10000000000000
dig15       qword   100000000000000
dig16       qword   1000000000000000
dig17       qword   10000000000000000
dig18       qword   100000000000000000
dig19       qword   1000000000000000000
dig20       qword   10000000000000000000

uSize       proc
            push    rdx
            cmp     rax, dig10
            jae     ge10
            cmp     rax, dig5
            jae     ge5
            mov     edx, 4
            cmp     rax, dig4
            jae     allDone
            dec     edx
            cmp     rax, dig3
            jae     allDone
            dec     edx
            cmp     rax, dig2
            jae     allDone
            dec     edx
            jmp     allDone

ge5:        mov     edx, 9
            cmp     rax, dig9
            jae     allDone
            dec     edx
            cmp     rax, dig8
            jae     allDone
            dec     edx
            cmp     rax, dig7
            jae     allDone
            dec     edx
            cmp     rax, dig6
            jae     allDone
            dec     edx      ; Must be 5
            jmp     allDone

ge10:       cmp     rax, dig14
            jae     ge14
            mov     edx, 13
            cmp     rax, dig13
            jae     allDone
            dec     edx
            cmp     rax, dig12
            jae     allDone
            dec     edx
            cmp     rax, dig11
            jae     allDone
            dec     edx      ; Must be 10
            jmp     allDone

ge14:       mov     edx, 20
            cmp     rax, dig20
            jae     allDone
            dec     edx
            cmp     rax, dig19
            jae     allDone
            dec     edx
            cmp     rax, dig18
 jae     allDone
            dec     edx
            cmp     rax, dig17
            jae     allDone
            dec     edx
            cmp     rax, dig16
            jae     allDone
            dec     edx
            cmp     rax, dig15
            jae     allDone
            dec     edx      ; Must be 14

allDone:    mov     rax, rdx ; Return digit count
            pop     rdx
            ret
uSize       endp
```

对于有符号整数，可以使用以下代码：

```
; iSize - Determines the number of print positions required by 
;         a 64-bit signed integer.

iSize       proc
            test    rax, rax
            js      isNeg

            jmp     uSize   ; Effectively a call and ret

; If the number is negative, negate it, call uSize,
; and then bump the size up by 1 (for the "-" character):

isNeg:      neg     rax
            call    uSize
            inc     rax
            ret
iSize       endp
```

对于扩展精度的大小操作，暴力算法方法很快就变得不切实际（64 位已经够糟糕了）。最佳解决方案是将扩展精度值除以 10 的幂（例如，1e+18）。这样可以将数字的大小减少 18 位。只要商大于 64 位（并跟踪除以 1e+18 的次数），就重复这一过程。当商适合 64 位（19 或 20 位数字）时，调用 64 位的 `uSize` 函数，并加上你通过除法操作消除的数字位数（每除以 1e+18 减少 18 位）。这个实现留给你自己完成……

一旦你有了 `iSize` 和 `uSize` 例程，编写格式化输出例程 `utoStrSize` 或 `itoStrSize` 就变得容易了。初次进入时，这些例程会调用相应的 `iSize` 或 `uSize` 例程来确定数字所需的字符位置数。如果 `iSize` 或 `uSize` 例程返回的值大于最小大小参数（传入 `utoStrSize` 或 `itoStrSize` 的值），则不需要其他格式化操作。如果参数大小的值大于 `iSize` 或 `uSize` 返回的值，程序必须计算这两个值之间的差异，并在数字转换之前将相应数量的空格（或其他填充字符）输出到字符串中。清单 9-9 显示了 `utoStrSize` 和 `itoStrSize` 函数。

```
; utoStrSize - Converts an unsigned integer to a formatted string
;              having at least "minDigits" character positions.
;              If the actual number of digits is smaller than
;              "minDigits" then this procedure inserts enough
;              "pad" characters to extend the size of the string.

; Inputs:
;    RAX -   Number to convert to string.
;    CL  -   minDigits (minimum print positions).
;    CH  -   Padding character.
;    RDI -   Buffer pointer for output string.

utoStrSize  proc
            push    rcx
            push    rdi
            push    rax

            call    uSize           ; Get actual number of digits
            sub     cl, al          ; >= the minimum size?
            jbe     justConvert

; If the minimum size is greater than the number of actual
; digits, we need to emit padding characters here.

; Note that this code used "sub" rather than "cmp" above.
; As a result, CL now contains the number of padding
; characters to emit to the string (CL is always positive
; at this point as negative and zero results would have
; branched to justConvert).

padLoop:    mov     [rdi], ch
            inc     rdi
            dec     cl
            jne     padLoop

; Okay, any necessary padding characters have already been
; added to the string. Call utoStr to convert the number
; to a string and append to the buffer:

justConvert:
            mov     rax, [rsp]      ; Retrieve original value
            call    utoStr

            pop     rax
            pop     rdi
 pop     rcx
            ret
utoStrSize  endp

; itoStrSize - Converts a signed integer to a formatted string
;              having at least "minDigits" character positions.
;              If the actual number of digits is smaller than
;              "minDigits" then this procedure inserts enough
;              "pad" characters to extend the size of the string.

; Inputs:
;    RAX -   Number to convert to string.
;    CL  -   minDigits (minimum print positions).
;    CH  -   Padding character.
;    RDI -   Buffer pointer for output string.

itoStrSize  proc
            push    rcx
            push    rdi
            push    rax

            call    iSize           ; Get actual number of digits
            sub     cl, al          ; >= the minimum size?
            jbe     justConvert

; If the minimum size is greater than the number of actual
; digits, we need to emit padding characters here.

; Note that this code used "sub" rather than "cmp" above.
; As a result, CL now contains the number of padding
; characters to emit to the string (CL is always positive
; at this point as negative and zero results would have
; branched to justConvert).

padLoop:    mov     [rdi], ch
            inc     rdi
            dec     cl
            jne     padLoop

; Okay, any necessary padding characters have already been
; added to the string. Call utoStr to convert the number
; to a string and append to the buffer:

justConvert:
            mov     rax, [rsp]     ; Retrieve original value
            call    itoStr

            pop     rax
            pop     rdi
            pop     rcx
            ret
itoStrSize  endp
```

清单 9-9：格式化整数到字符串的转换函数

### 9.1.8 将浮点值转换为字符串

本章迄今为止的代码涉及将整数数值转换为字符字符串（通常用于输出给用户）。将浮点数值转换为字符串同样重要。本节（及其子节）涵盖了这一转换。

浮点数值可以转换为两种形式的字符串：

+   十进制表示法转换（例如，± *xxx.yyy* 格式）

+   指数（或科学）表示法转换（例如，± *x.yyyyye* ± *zz* 格式）

无论最终的输出格式如何，都需要两个不同的操作来将浮点值转换为字符字符串。首先，你必须将尾数转换为适当的数字字符串。其次，你必须将指数转换为数字字符串。

然而，这并不是一个简单的将两个整数值转换为十进制字符串并连接它们（在尾数和指数之间加上一个*e*）的情况。首先，尾数不是一个整数值：它是一个定点小数二进制值。简单地将它视为一个*n*位的二进制值（其中*n*是尾数位数）几乎总会导致转换错误。其次，虽然指数在某种程度上是一个整数值，^(1) 它表示的是 2 的幂，而不是 10 的幂。将 2 的幂以整数形式显示并不适合十进制浮动点表示。处理这两个问题（分数尾数和二进制指数）是将浮动点值转换为字符串的主要复杂性所在。

尽管在 x86-64 上有三种浮动点格式——单精度（32 位`real4`）、双精度（64 位`real8`）和扩展精度（80 位`real10`）——x87 FPU 在将值加载到 FPU 时会自动将`real4`和`real8`格式转换为`real10`格式。因此，通过在转换过程中使用 x87 FPU 进行所有浮动点算术操作，我们只需要编写代码将`real10`值转换为字符串形式。

`real10`浮动点值具有 64 位尾数。这不是一个 64 位整数。相反，这 64 位表示的值介于 0 和略小于 2 之间。（有关 IEEE 80 位浮动点格式的更多细节，请参见第二章中的《IEEE 浮动点格式》）第 63 位通常为 1。如果第 63 位为 0，则尾数是非规格化的，表示介于 0 和大约 3.65 × 10^(-4951)之间的数字。

要以大约 18 位精度以十进制形式输出尾数，诀窍是反复将浮动点值乘以或除以 10，直到该数字位于 1e+18 和略小于 1e+19 之间（即 9.9999...e+18）。一旦指数在适当的范围内，尾数位将形成一个 18 位的整数值（没有小数部分），该值可以转换为十进制字符串，从而获得组成尾数值的 18 个数字（使用我们的好朋友`fbstp`指令）。实际上，你可以通过将浮动点值乘以或除以大的 10 的幂来将其值调整到 1e+18 到 1e+19 的范围。这种方法更快（浮动点操作较少），也更精确（同样因为浮动点操作较少）。

要将指数转换为适当的十进制字符串，你需要追踪除以或乘以 10 的次数。每次除以 10 时，将十进制指数值加 1；每次乘以 10 时，将十进制指数值减 1。过程结束时，从十进制指数值中减去 18（因为此过程产生的值的指数是 18），然后将十进制指数值转换为字符串。

#### 9.1.8.1 转换浮动点指数

要将指数转换为十进制数字字符串，请使用以下算法：

1.  如果数字是 0.0，直接输出尾数字符串“ 000000000000000000”（注意字符串开头的空格）。

1.  将十进制指数初始化为 0。

1.  如果指数为负，输出一个连字符（-）并取反值；如果是正数，则输出一个空格字符。

1.  如果（可能为负的）指数值小于 1.0，跳至步骤 8。

1.  *正指数*：将数字与逐渐减小的 10 的幂进行比较，从 10^(+4096) 开始，然后是 10^(+2048)，然后是 10^(+1024)，然后是...，最后是 10⁰。每次比较后，如果当前值大于该幂次，则除以该幂次，并将该幂次的指数（4096, 2048, ... , 0）加到十进制指数值上。

1.  重复步骤 5，直到指数为 0（即值处于 1.0 ≤ value < 10.0 范围内）。

1.  跳至步骤 10。

1.  *负指数*：将数字与逐渐增大的 10 的幂进行比较，从 10^(-4096) 开始，然后是 10^(-2048)，然后是 10^(-1024)，然后是...，最后是 10⁰。每次比较后，如果当前值小于该幂次，则除以该幂次，并将该幂次的指数（4096, 2048, ... , 0）从十进制指数值中减去。

1.  重复步骤 8，直到指数为 0（即值处于 1.0 ≤ value < 10.0 范围内）。

1.  某些合法的浮点值太大，无法用 18 位数字表示（例如，9,223,372,036,854,775,807 可以适配到 63 位，但需要超过 18 位有效数字才能表示）。具体来说，范围在 403A_DE0B_6B3A_763F_FF01h 到 403A_DE0B_6B3A_763F_FFFFh 之间的值大于 999,999,999,999,999,999，但仍然适配到 64 位尾数。`fbstp` 指令无法将这些值转换为压缩 BCD 值。

    为了解决这个问题，代码应该显式地测试该范围内的值，并将其向上舍入为 1e+17（如果发生这种情况，还要增加十进制指数值）。在某些情况下，值可能大于 1e+19。此时，最后一次除以 10.0 将解决这个问题。

1.  此时，浮点值已经是 `fbstp` 指令可以转换为压缩 BCD 值的合理数值，因此转换函数使用 `fbstp` 来进行此转换。

1.  最后，使用将数值转换为十六进制（BCD）字符串的操作，将压缩 BCD 值转换为 ASCII 字符串（参见第 500 页的“将无符号十进制值转换为字符串”和清单 9-5）。

清单 9-10 提供了（简化的）代码和数据，用于实现尾数到字符串的转换函数`FPDigits`。`FPDigits` 将尾数转换为 18 位数字序列，并返回 EAX 寄存器中的十进制指数值。它不会在字符串中放置小数点，也不会处理指数部分。

```
 .data

            align   4

; TenTo17 - Holds the value 1.0e+17\. Used to get a floating-
;           point number into the range `x.xxxxxxxxxxxx`e+17.

TenTo17     real10  1.0e+17

; PotTblN - Hold powers of 10 raised to negative powers of 2.

PotTblN     real10  1.0,
                    1.0e-1,
                    1.0e-2,
                    1.0e-4,
                    1.0e-8,
                    1.0e-16,
                    1.0e-32,
 1.0e-64,
                    1.0e-128,
                    1.0e-256,
                    1.0e-512,
                    1.0e-1024,
                    1.0e-2048,
                    1.0e-4096

; PotTblP - Hold powers of 10 raised to positive powers of 2.

            align   4
PotTblP     real10  1.0,
                    1.0e+1,
                    1.0e+2,
                    1.0e+4,
                    1.0e+8,
                    1.0e+16,
                    1.0e+32,
                    1.0e+64,
                    1.0e+128,
                    1.0e+256,
                    1.0e+512,
                    1.0e+1024,
                    1.0e+2048,
                    1.0e+4096

; ExpTbl - Integer equivalents to the powers
;          in the tables above.

            align   4
ExpTab      dword   0,
                    1,
                    2,
                    4,
                    8,
                    16,
                    32,
                    64,
                    128,
                    256,
                    512,
                    1024,
                    2048,
                    4096
               .
               .
               .

*************************************************************

; FPDigits - Used to convert a floating-point number on the FPU
;            stack (ST(0)) to a string of digits.

; Entry Conditions:

; ST(0) -    80-bit number to convert.
;            Note: code requires two free FPU stack elements.
; RDI   -    Points at array of at least 18 bytes where 
;            FPDigits stores the output string.

; Exit Conditions:

; RDI   -    Converted digits are found here.
; RAX   -    Contains exponent of the number.
; CL    -    Contains the sign of the mantissa (" " or "-").
; ST(0) -    Popped from stack.

*************************************************************

P10TblN     equ     <real10 ptr [r8]>
P10TblP     equ     <real10 ptr [r9]>
xTab        equ     <dword ptr [r10]>

FPDigits    proc
            push    rbx
            push    rdx
            push    rsi
            push    r8
            push    r9
            push    r10

; Special case if the number is zero.

            ftst
            fstsw   ax
            sahf
            jnz     fpdNotZero

; The number is zero, output it as a special case.

            fstp    tbyte ptr [rdi] ; Pop value off FPU stack
            mov     rax, "00000000"
            mov     [rdi], rax 
            mov     [rdi + 8], rax 
            mov     [rdi + 16], ax
            add     rdi, 18 
            xor     edx, edx        ; Return an exponent of 0
            mov     bl, ' '         ; Sign is positive
            jmp     fpdDone

fpdNotZero:

; If the number is not zero, then fix the sign of the value.

            mov     bl, ' '         ; Assume it's positive
            jnc     WasPositive     ; Flags set from sahf above

 fabs                 ; Deal only with positive numbers
            mov     bl, '-'      ; Set the sign return result

WasPositive:

; Get the number between 1 and 10 so we can figure out 
; what the exponent is.  Begin by checking to see if we have
; a positive or negative exponent.

            xor     edx, edx     ; Initialize exponent to 0
            fld1
            fcomip  st(0), st(1)
            jbe     PosExp

; We've got a value between zero and one, exclusive,
; at this point.  That means this number has a negative
; exponent.  Multiply the number by an appropriate power
; of 10 until we get it in the range 1 through 10.

            mov     esi, sizeof PotTblN  ; After last element
            mov     ecx, sizeof ExpTab   ; Ditto
            lea     r8, PotTblN
            lea     r9, PotTblP
            lea     r10, ExpTab

CmpNegExp:
            sub     esi, 10          ; Move to previous element
            sub     ecx, 4           ; Zeroes HO bytes
            jz      test1

            fld     P10TblN[rsi * 1] ; Get current power of 10
            fcomip  st(0), st(1)     ; Compare against NOS
            jbe     CmpNegExp        ; While Table >= value

            mov     eax, xTab[rcx * 1]
            test    eax, eax
            jz      didAllDigits

            sub     edx, eax
            fld     P10TblP[rsi * 1]
            fmulp
            jmp     CmpNegExp

; If the remainder is *exactly* 1.0, then we can branch
; on to InRange1_10; otherwise, we still have to multiply
; by 10.0 because we've overshot the mark a bit.

test1:
            fld1
            fcomip  st(0), st(1)
            je      InRange1_10

didAllDigits:

; If we get to this point, then we've indexed through
; all the elements in the PotTblN and it's time to stop.

            fld     P10TblP[10]   ; 10.0
            fmulp
            dec     edx
            jmp     InRange1_10

; At this point, we've got a number that is 1 or greater.
; Once again, our task is to get the value between 1 and 10.

PosExp:

            mov     esi, sizeof PotTblP ; After last element
            mov     ecx, sizeof ExpTab  ; Ditto
            lea     r9, PotTblP
            lea     r10, ExpTab

CmpPosExp:
            sub     esi, 10             ; Move back 1 element in
            sub     ecx, 4              ; PotTblP and ExpTbl
            fld     P10TblP[rsi * 1]
            fcomip  st(0), st(1)
            ja      CmpPosExp;
            mov     eax, xTab[rcx * 1]
            test    eax, eax
            jz      InRange1_10

            add     edx, eax
            fld     P10TblP[rsi * 1]
            fdivp
            jmp     CmpPosExp

InRange1_10:

; Okay, at this point the number is in the range 1 <= x < 10.
; Let's multiply it by 1e+18 to put the most significant digit
; into the 18th print position.  Then convert the result to
; a BCD value and store away in memory.

            sub     rsp, 24         ; Make room for BCD result
            fld     TenTo17
            fmulp

; We need to check the floating-point result to make sure it
; is not outside the range we can legally convert to a BCD 
; value.

; Illegal values will be in the range:

; >999,999,999,999,999,999 ... <1,000,000,000,000,000,000
; $403a_de0b_6b3a_763f_ff01 ... $403a_de0b_6b3a_763f_ffff

; Should one of these values appear, round the result up to
; $403a_de0b_6b3a_7640_0000:

            fstp    real10 ptr [rsp]
            cmp     word ptr [rsp + 8], 403ah
            jne     noRounding

            cmp     dword ptr [rsp + 4], 0de0b6b3ah
            jne     noRounding

            mov     eax, [rsp]
            cmp     eax, 763fff01h
            jb      noRounding;
            cmp     eax, 76400000h
            jae     TooBig

            fld     TenTo17
            inc     edx           ; Inc exp as this is really 10¹⁸
            jmp     didRound

; If we get down here, there were problems getting the
; value in the range 1 <= x <= 10 above and we've got a value
; that is 10e+18 or slightly larger. We need to compensate for
; that here.

TooBig:
            lea     r9, PotTblP
            fld     real10 ptr [rsp]
            fld     P10TblP[10]   ; /10
            fdivp
            inc     edx           ; Adjust exp due to fdiv
            jmp     didRound

noRounding:
            fld     real10 ptr [rsp]
didRound:   
            fbstp   tbyte ptr [rsp]

; The data on the stack contains 18 BCD digits. Convert these
; to ASCII characters and store them at the destination location
; pointed at by EDI.

            mov     ecx, 8
repeatLp:
            mov     al, byte ptr [rsp + rcx]
            shr     al, 4         ; Always in the
            or      al, '0'       ; range "0" to "9"
            mov     [rdi], al
            inc     rdi

            mov     al, byte ptr [rsp + rcx]
            and     al, 0fh
 or      al, '0'
            mov     [rdi], al
            inc     rdi

            dec     ecx
            jns     repeatLp

            add     rsp, 24         ; Remove BCD data from stack

fpdDone:

            mov     eax, edx        ; Return exponent in EAX
            mov     cl, bl          ; Return sign in CL
            pop     r10
            pop     r9
            pop     r8
            pop     rsi
            pop     rdx
            pop     rbx
            ret

FPDigits    endp
```

清单 9-10：浮点尾数到字符串的转换

#### 9.1.8.2 将浮点值转换为十进制字符串

`FPDigits` 函数执行将浮点值转换为十进制字符串所需的大部分工作：它将尾数转换为一串数字，并以十进制整数形式提供指数。尽管十进制格式没有明确显示指数值，但将浮点值转换为十进制字符串的过程需要指数（十进制）值，以确定小数点的位置。结合调用者提供的几个附加参数，从 `FPDigits` 获取输出并将其转换为适当格式化的十进制数字字符串相对容易。

最终要写入的函数是 `r10ToStr`，这是将 `real10` 值转换为字符串时调用的主要函数。这是一个格式化输出函数，通过使用标准格式化选项来转换二进制浮点值，控制输出宽度、小数点后的位置数以及在没有出现数字的地方填充字符（通常是空格）。调用 `r10ToStr` 函数时需要以下参数：

`r10`

1.  要转换为字符串的 `real10` 值（如果 `r10` 是 `real4` 或 `real8` 值，FPU 会在将其加载到 FPU 时自动将其转换为 `real10` 值）。

`fWidth`

1.  字段宽度。这是字符串将占用的总字符位置数。此计数包括符号的空间（可以是空格或连字符），但不包括字符串的零终止字节空间。字段宽度必须大于 0 且小于或等于 1024。

`decDigits`

1.  小数点右侧的数字个数。此值必须至少比 `fWidth` 小 3，因为必须为符号字符、至少一个小数点左侧的数字以及小数点留出空间。如果此值为 0，则转换例程不会在字符串中发出小数点。这是一个无符号值；如果调用者在此处提供负数，程序将把它当作一个非常大的正数（并将返回错误）。

`fill`

1.  填充字符。如果 `r10ToStr` 生成的数字字符串使用的字符少于 `fWidth`，程序将把数字值右对齐，并用此 `fill` 字符（通常是空格字符）填充最左侧的字符。

`buffer`

1.  用于接收数字字符串的缓冲区。

`maxLength`

1.  缓冲区的大小（包括零终止字节）。如果转换例程尝试创建比此值更大的字符串（即 `fWidth` 大于或等于此值），则会返回错误。

字符串输出操作只有三个实际任务：正确放置小数点（如果存在），仅复制由 `fWidth` 值指定的数字，并将截断的数字四舍五入为输出数字。

舍入操作是该过程最有趣的部分。`r10ToStr` 函数在舍入之前将 `real10` 值转换为 ASCII 字符，因为转换后的结果更容易进行舍入。所以，舍入操作的过程包括将 5 加到最不重要显示数字之后的（ASCII）数字上。如果这个和超过了（字符）9，舍入算法必须将 1 加到最不重要显示数字上。如果这个和超过了 9，算法必须从字符中减去（值）10，并将 1 加到下一个不那么重要的数字上。这个过程会重复进行，直到达到最重要的数字，或者直到没有进位（即和不超过 9）。在（罕见的）舍入通过所有数字的情况下（例如字符串为“999999 . . . 9”），舍入算法必须将字符串替换为“10000 . . . 0”，并将十进制指数加 1。

输出字符串的算法对于负指数和非负指数的值有所不同。负指数的处理可能是最简单的。以下是输出负指数值的算法：

1.  函数首先将 3 加到 `decDigits`。

1.  如果 `decDigits` 小于 4，则将其设置为 4 作为默认值。^(3)

1.  如果 `decDigits` 大于 `fWidth`，函数向字符串中输出 `fWidth` 个 `"#"` 字符，然后返回。

1.  如果 `decDigits` 小于 `fWidth`，则输出 `(fWidth - decDigits)` 个填充字符 (`fill`) 到输出字符串中。

1.  如果 `r10` 为负数，向字符串中输出 `-0.`；否则，输出 `0.`（如果是非负数，则在 0 前面加上空格）。

1.  接下来，输出转换后的数字的数字。如果字段宽度小于 21（18 位数字加上 3 位前导 `0.` 或 `-0.` 字符），则函数从转换后的数字字符串中输出指定的 (`fWidth`) 字符。如果宽度大于 21，则函数输出转换后的所有 18 位数字，并在其后跟随需要填充字段宽度的零字符。

1.  最后，函数将字符串以零终止并返回。

如果指数为正数或 0，则转换稍微复杂一些。首先，代码需要确定结果所需的字符位置数量。其计算方式如下：

```
`exponent` + 2 + `decDigits` + (0 if `decDigits` is 0, 1 otherwise)
```

`exponent` 值是小数点左侧的数字数量（减去 1）。`2` 组件存在是因为始终有一个位置用于符号字符（空格或连字符），并且小数点左侧始终至少有一个数字。`decDigits` 组件添加了小数点后面显示的数字数量。最后，如果小数点存在（即如果 `decDigits` 大于 0），此方程式会为点字符加上 1。

一旦计算出所需的宽度，函数会将该值与调用者提供的 `fWidth` 值进行比较。如果计算出的值大于 `fWidth`，函数将输出 `fWidth` 个 "`#`" 字符并返回。否则，它可以将数字输出到字符串中。

正如负指数情况那样，代码首先确定数字是否会占用输出字符串中的所有字符位置。如果不会，它会计算 `fWidth` 与实际字符数之间的差异，并输出 `fill` 字符来填充数字字符串。接着，输出一个空格或连字符字符（取决于原始值的符号）。然后，函数输出小数点左侧的数字（通过递减 `exponent` 值）。如果 `decDigits` 值不为零，函数会输出点字符并输出 `FPDigits` 生成的数字字符串中的任何剩余数字。如果函数超过了 `FPDigits` 生成的 18 个数字（无论是在小数点之前还是之后），函数会用 0 字符填充剩余的位置。最后，函数输出字符串的零终止字节并返回给调用者。

清单 9-11 提供了 `r10ToStr` 函数的源代码。

```
***********************************************************

; r10ToStr -  Converts a real10 floating-point number to the
;             corresponding string of digits.  Note that this
;             function always emits the string using decimal
;             notation.  For scientific notation, use the e10ToBuf
;             routine.

; On Entry:

;    r10        -    real10 value to convert.
;                    Passed in ST(0).

;    fWidth     -    Field width for the number (note that this
;                    is an *exact* field width, not a minimum
;                    field width).
;                    Passed in EAX (RAX).

;    decimalpts -    # of digits to display after the decimal pt.
;                    Passed in EDX (RDX). 

;    fill       -    Padding character if the number is smaller
;                    than the specified field width.
;                    Passed in CL (RCX).

;    buffer     -    Stores the resulting characters in
;                    this string.
;                    Address passed in RDI.

;    maxLength  -    Maximum string length.
;                    Passed in R8d (R8).

; On Exit:

; Buffer contains the newly formatted string.  If the
; formatted value does not fit in the width specified,
; r10ToStr will store "#" characters into this string.

; Carry -    Clear if success; set if an exception occurs.
;            If width is larger than the maximum length of
;            the string specified by buffer, this routine
;            will return with the carry set and RAX = -1,
;            -2, or -3.

***********************************************************

r10ToStr    proc

; Local variables:

fWidth      equ     <dword ptr [rbp - 8]>    ; RAX: uns32
decDigits   equ     <dword ptr [rbp - 16]>   ; RDX: uns32
fill        equ     <[rbp - 24]>             ; CL: char
bufPtr      equ     <[rbp - 32]>             ; RDI: pointer
exponent    equ     <dword ptr [rbp - 40]>   ; uns32
sign        equ     <byte ptr [rbp - 48]>    ; char
digits      equ     <byte ptr [rbp - 128]>   ; char[80]
maxWidth    =       64              ; Must be smaller than 80 - 2

            push    rdi
            push    rbx
            push    rcx
            push    rdx
            push    rsi
            push    rax
            push    rbp
            mov     rbp, rsp
            sub     rsp, 128        ; 128 bytes of local vars

; First, make sure the number will fit into the 
; specified string.

            cmp     eax, r8d        ; R8d = max length
            jae     strOverflow

; If the width is zero, raise an exception:

            test    eax, eax
            jz      voor            ; Value out of range

            mov     bufPtr, rdi
            mov     qword ptr decDigits, rdx
            mov     fill, rcx
            mov     qword ptr fWidth, rax

; If the width is too big, raise an exception:

            cmp     eax, maxWidth
            ja      badWidth

; Okay, do the conversion.
; Begin by processing the mantissa digits:

            lea     rdi, digits     ; Store result here
            call    FPDigits        ; Convert r80 to string
            mov     exponent, eax   ; Save exp result
            mov     sign, cl        ; Save mantissa sign char

; Round the string of digits to the number of significant 
; digits we want to display for this number:

            cmp     eax, 17
            jl      dontForceWidthZero

            xor     rax, rax        ; If the exp is negative or
                                    ; too large, set width to 0
dontForceWidthZero:
            mov     rbx, rax        ; Really just 8 bits
            add     ebx, decDigits  ; Compute rounding position
            cmp     ebx, 17
            jge     dontRound       ; Don't bother if a big #

; To round the value to the number of significant digits,
; go to the digit just beyond the last one we are considering
; (EAX currently contains the number of decimal positions)
; and add 5 to that digit.  Propagate any overflow into the
; remaining digit positions.

            inc     ebx                 ; Index + 1 of last sig digit
            mov     al, digits[rbx * 1] ; Get that digit
            add     al, 5               ; Round (for example, +0.5)
            cmp     al, '9'
            jbe     dontRound

            mov     digits[rbx * 1], '0' + 10 ; Force to zero

whileDigitGT9:                                ; (See sub 10 below)
            sub     digits[rbx * 1], 10       ; Sub out overflow, 
            dec     ebx                       ; carry, into prev
            js      hitFirstDigit;            ; digit (until 1st
                                              ; digit in the #)
            inc     digits[rbx * 1]
            cmp     digits[rbx], '9'          ; Overflow if > "9"
            ja      whileDigitGT9
            jmp     dontRound

hitFirstDigit:

; If we get to this point, then we've hit the first
; digit in the number.  So we've got to shift all
; the characters down one position in the string of
; bytes and put a "1" in the first character position.

            mov     ebx, 17

repeatUntilEBXeq0:

            mov     al, digits[rbx * 1]
            mov     digits[rbx * 1 + 1], al
            dec     ebx
            jnz     repeatUntilEBXeq0

            mov     digits, '1'
 inc     exponent    ; Because we added a digit

dontRound: 

; Handle positive and negative exponents separately.

            mov     rdi, bufPtr ; Store the output here
            cmp     exponent, 0
            jge     positiveExponent

; Negative exponents:
; Handle values between 0 and 1.0 here (negative exponents
; imply negative powers of 10).

; Compute the number's width.  Since this value is between
; 0 and 1, the width calculation is easy: it's just the
; number of decimal positions they've specified plus three
; (since we need to allow room for a leading "-0.").

            mov     ecx, decDigits
            add     ecx, 3
            cmp     ecx, 4
            jae     minimumWidthIs4

            mov     ecx, 4      ; Minimum possible width is four

minimumWidthIs4:
            cmp     ecx, fWidth
            ja      widthTooBig 

; This number will fit in the specified field width,
; so output any necessary leading pad characters.

            mov     al, fill
            mov     edx, fWidth
            sub     edx, ecx
            jmp     testWhileECXltWidth

whileECXltWidth:
            mov     [rdi], al
            inc     rdi
            inc     ecx

testWhileECXltWidth:
            cmp     ecx, fWidth
            jb      whileECXltWidth

; Output " 0." or "-0.", depending on the sign of the number.

            mov     al, sign
            cmp     al, '-'
            je      isMinus

            mov     al, ' '

isMinus:    mov     [rdi], al
            inc     rdi
            inc     edx

            mov     word ptr [rdi], '.0'
            add     rdi, 2
            add     edx, 2

; Now output the digits after the decimal point:

            xor     ecx, ecx        ; Count the digits in ECX
            lea     rbx, digits     ; Pointer to data to output d

; If the exponent is currently negative, or if
; we've output more than 18 significant digits,
; just output a zero character.

repeatUntilEDXgeWidth: 
            mov     al, '0'
            inc     exponent
            js      noMoreOutput

            cmp     ecx, 18
            jge     noMoreOutput

            mov     al, [rbx]
            inc     ebx

noMoreOutput:
            mov     [rdi], al
            inc     rdi
            inc     ecx
            inc     edx
            cmp     edx, fWidth
            jb      repeatUntilEDXgeWidth
            jmp     r10BufDone

; If the number's actual width was bigger than the width
; specified by the caller, emit a sequence of "#" characters
; to denote the error.

widthTooBig:

; The number won't fit in the specified field width,
; so fill the string with the "#" character to indicate
; an error.

            mov     ecx, fWidth
            mov     al, '#'
fillPound:  mov     [rdi], al
            inc     rdi
            dec     ecx
            jnz     fillPound
            jmp     r10BufDone

; Handle numbers with a positive exponent here.

positiveExponent:

; Compute # of digits to the left of the ".".
; This is given by:

;                   Exponent        ; # of digits to left of "."
;           +       2               ; Allow for sign and there
;                                   ; is always 1 digit left of "."
;           +       decimalpts      ; Add in digits right of "."
;           +       1               ; If there is a decimal point

            mov     edx, exponent   ; Digits to left of "."
            add     edx, 2          ; 1 digit + sign posn
            cmp     decDigits, 0
            je      decPtsIs0

            add     edx, decDigits  ; Digits to right of "."
            inc     edx             ; Make room for the "."

decPtsIs0:

; Make sure the result will fit in the
; specified field width.

            cmp     edx, fWidth
            ja      widthTooBig

; If the actual number of print positions
; is fewer than the specified field width,
; output leading pad characters here.

            cmp     edx, fWidth
            jae     noFillChars

            mov     ecx, fWidth
            sub     ecx, edx
            jz      noFillChars
            mov     al, fill
fillChars:  mov     [rdi], al
            inc     rdi
            dec     ecx
            jnz     fillChars

noFillChars:

; Output the sign character.

            mov     al, sign
            cmp     al, '-'
            je      outputMinus;

            mov     al, ' '

outputMinus:
            mov     [rdi], al
            inc     rdi

; Okay, output the digits for the number here.

            xor     ecx, ecx        ; Counts # of output chars
            lea     rbx, digits     ; Ptr to digits to output

; Calculate the number of digits to output
; before and after the decimal point.

            mov     edx, decDigits  ; Chars after "."
            add     edx, exponent   ; # chars before "."
            inc     edx             ; Always one digit before "."

; If we've output fewer than 18 digits, go ahead
; and output the next digit.  Beyond 18 digits,
; output zeros.

repeatUntilEDXeq0:
            mov     al, '0'
            cmp     ecx, 18
            jnb     putChar

            mov     al, [rbx]
            inc     rbx

putChar:    mov     [rdi], al
            inc     rdi

; If the exponent decrements to zero,
; then output a decimal point.

            cmp     exponent, 0
            jne     noDecimalPt
            cmp     decDigits, 0
            je      noDecimalPt

            mov     al, '.'
            mov     [rdi], al
            inc     rdi

noDecimalPt:
            dec     exponent        ; Count down to "." output
            inc     ecx             ; # of digits thus far
            dec     edx             ; Total # of digits to output
            jnz     repeatUntilEDXeq0

; Zero-terminate string and leave:

r10BufDone: mov     byte ptr [rdi], 0
            leave
            clc                     ; No error
            jmp     popRet

badWidth:   mov     rax, -2     ; Illegal width
            jmp     ErrorExit

strOverflow:
            mov     rax, -3     ; String overflow
            jmp     ErrorExit

voor:       or      rax, -1     ; Range error
ErrorExit:  leave
            stc     ; Error
            mov     [rsp], rax  ; Change RAX on return

popRet:     pop     rax
            pop     rsi
            pop     rdx
            pop     rcx
            pop     rbx
            pop     rdi
            ret

r10ToStr    endp
```

清单 9-11：`r10ToStr` 转换函数

#### 9.1.8.3 将浮点值转换为指数形式

将浮点值转换为指数（科学）形式比转换为十进制形式要容易一些。尾数总是呈现为 `sx.y` 形式，其中 `s` 是一个连字符或空格，`x` 是恰好一个小数位，`y` 是一个或多个小数位。`FPDigits` 函数几乎完成了创建该字符串的所有工作。指数转换函数需要输出带符号和小数点字符的尾数字符串，然后输出该数字的十进指数。将指数值（由 `FPDigits` 在 EAX 寄存器中以十进制整数形式返回）转换为字符串，实际上只是本章早些时候提到的数字到十进制字符串转换，使用不同的输出格式。

本章介绍的函数允许你指定指数的数字位数为 1、2、3 或 4。如果指数需要的位数超过调用者指定的数字，函数将返回失败。如果需要的位数少于调用者指定的数字，函数会在指数前填充 0。为了模拟典型的浮点转换形式，对于单精度值，指定 2 位的指数；对于双精度值，指定 3 位的指数；对于扩展精度值，指定 4 位的指数。

列表 9-12 提供了一个快速且粗略的函数，将十进制指数值转换为适当的字符串形式，并将这些字符输出到缓冲区。此函数将 RDI 指向超出最后一个指数数字的位置，并且没有对字符串进行零终止。它实际上只是一个辅助函数，用于输出 `e10ToStr` 函数的字符，该函数将在下一个列表中出现。

```
*************************************************************

; expToBuf - Unsigned integer to buffer.
;            Used to output up to 4-digit exponents.

; Inputs:

;    EAX:   Unsigned integer to convert.
;    ECX:   Print width 1-4.
;    RDI:   Points at buffer.

;    FPU:   Uses FPU stack.

; Returns:

;    RDI:   Points at end of buffer.

expToBuf    proc

expWidth    equ     <[rbp + 16]>
exp         equ     <[rbp + 8]>
bcd         equ     <[rbp - 16]>

            push    rdx
            push    rcx            ; At [RBP + 16]
            push    rax            ; At [RBP + 8]
            push    rbp
            mov     rbp, rsp
            sub     rsp, 16

; Verify exponent digit count is in the range 1-4:

            cmp     rcx, 1
            jb      badExp
            cmp     rcx, 4
            ja      badExp
            mov     rdx, rcx

; Verify the actual exponent will fit in the number of digits:

            cmp     rcx, 2
            jb      oneDigit
            je      twoDigits
            cmp     rcx, 3
            ja      fillZeros      ; 4 digits, no error
            cmp     eax, 1000
            jae     badExp
            jmp     fillZeros

oneDigit:   cmp     eax, 10
            jae     badExp
            jmp     fillZeros

twoDigits:  cmp     eax, 100
            jae     badExp

; Fill in zeros for exponent:

fillZeros:  mov     byte ptr [rdi + rcx * 1 - 1], '0'
            dec     ecx
            jnz     fillZeros

; Point RDI at the end of the buffer:

            lea     rdi, [rdi + rdx * 1 - 1]
            mov     byte ptr [rdi + 1], 0
            push    rdi             ; Save pointer to end

; Quick test for zero to handle that special case:

            test    eax, eax
            jz      allDone

; The number to convert is nonzero.
; Use BCD load and store to convert
; the integer to BCD:

            fild    dword ptr exp   ; Get integer value
            fbstp   tbyte ptr bcd   ; Convert to BCD

; Begin by skipping over leading zeros in
; the BCD value (max 10 digits, so the most
; significant digit will be in the HO nibble
; of byte 4).

            mov     eax, bcd        ; Get exponent digits
            mov     ecx, expWidth   ; Number of total digits

OutputExp:  mov     dl, al
            and     dl, 0fh
            or      dl, '0'
            mov     [rdi], dl
            dec     rdi
            shr     ax, 4
            jnz     OutputExp

; Zero-terminate the string and return:

allDone:    pop     rdi
            leave
            pop     rax
            pop     rcx
            pop     rdx
            clc
            ret

badExp:     leave
            pop     rax
 pop     rcx
            pop     rdx
            stc
            ret

expToBuf    endp
```

列表 9-12: 指数转换函数

实际的 `e10ToStr` 函数在列表 9-13 中，类似于 `r10ToStr` 函数。由于形式固定，尾数的输出不那么复杂，但在输出指数时需要做一些额外的工作。有关此代码的操作细节，请参考第 527 页的“将浮点值转换为十进制字符串”。

```
***********************************************************

; e10ToStr - Converts a real10 floating-point number to the
;            corresponding string of digits.  Note that this
;            function always emits the string using scientific
;            notation; use the r10ToStr routine for decimal notation.  

; On Entry:

;    e10         -   real10 value to convert.
;                    Passed in ST(0).

;    width       -   Field width for the number (note that this
;                    is an *exact* field width, not a minimum
;                    field width).
;                    Passed in RAX (LO 32 bits).

;    fill        -   Padding character if the number is smaller
;                    than the specified field width.
;                    Passed in RCX.

;    buffer      -   e10ToStr stores the resulting characters in
;                    this buffer (passed in RDI).

;    expDigs     -   Number of exponent digits (2 for real4,
;                    3 for real8, and 4 for real10).
;                    Passed in RDX (LO 8 bits).

;    maxLength   -   Maximum buffer size.
;                    Passed in R8\.                           

; On Exit:                                                  

;    RDI         -  Points at end of converted string.      

; Buffer contains the newly formatted string.  If the    
; formatted value does not fit in the width specified,   
; e10ToStr will store "#" characters into this string.   

; If there was an error, EAX contains -1, -2, or -3      
; denoting the error (value out of range, bad width,     
; or string overflow, respectively).                     

***********************************************************

; Unlike the integer-to-string conversions, this routine    
; always right-justifies the number in the specified        
; string.  Width must be a positive number; negative        
; values are illegal (actually, they are treated as         
; *really* big positive numbers that will always raise      
; a string overflow exception).                              

***********************************************************

e10ToStr    proc

fWidth      equ     <[rbp - 8]>       ; RAX
buffer      equ     <[rbp - 16]>      ; RDI
expDigs     equ     <[rbp - 24]>      ; RDX
rbxSave     equ     <[rbp - 32]>
rcxSave     equ     <[rbp - 40]>
rsiSave     equ     <[rbp - 48]>
Exponent    equ     <dword ptr [rbp - 52]>
MantSize    equ     <dword ptr [rbp - 56]>
Sign        equ     <byte ptr [rbp - 60]>
Digits      equ     <byte ptr [rbp - 128]>

            push    rbp
            mov     rbp, rsp
            sub     rsp, 128

            mov     buffer, rdi
            mov     rsiSave, rsi
            mov     rcxSave, rcx
            mov     rbxSave, rbx
            mov     fWidth, rax
            mov     expDigs, rdx

            cmp     eax, r8d
            jae     strOvfl
            mov     byte ptr [rdi + rax * 1], 0 ; Zero-terminate str

; First, make sure the width isn't zero.

            test    eax, eax
            jz      voor

; Just to be on the safe side, don't allow widths greater 
; than 1024:

            cmp     eax, 1024
            ja      badWidth

; Okay, do the conversion.

            lea     rdi, Digits     ; Store result string here
            call    FPDigits        ; Convert e80 to digit str
            mov     Exponent, eax   ; Save away exponent result
            mov     Sign, cl        ; Save mantissa sign char

; Verify that there is sufficient room for the mantissa's sign,
; the decimal point, two mantissa digits, the "E", and the
; exponent's sign.  Also add in the number of digits required
; by the exponent (2 for real4, 3 for real8, 4 for real10).

; -1.2e+00    :real4
; -1.2e+000   :real8
; -1.2e+0000  :real10

            mov     ecx, 6          ; Char posns for above chars
            add     ecx, expDigs    ; # of digits for the exp
            cmp     ecx, fWidth
            jbe     goodWidth

; Output a sequence of "#...#" chars (to the specified width)
; if the width value is not large enough to hold the 
; conversion:

            mov     ecx, fWidth
            mov     al, '#'
            mov     rdi, buffer
fillPound:  mov     [rdi], al
            inc     rdi
            dec     ecx
            jnz     fillPound
            jmp     exit_eToBuf

; Okay, the width is sufficient to hold the number; do the
; conversion and output the string here:

goodWidth:

            mov     ebx, fWidth     ; Compute the # of mantissa
            sub     ebx, ecx        ; digits to display
            add     ebx, 2          ; ECX allows for 2 mant digs
            mov     MantSize,ebx

; Round the number to the specified number of print positions.
; (Note: since there are a maximum of 18 significant digits,
; don't bother with the rounding if the field width is greater
; than 18 digits.)

 cmp     ebx, 18
            jae     noNeedToRound

; To round the value to the number of significant digits,
; go to the digit just beyond the last one we are considering
; (EBX currently contains the number of decimal positions)
; and add 5 to that digit.  Propagate any overflow into the
; remaining digit positions.

            mov     al, Digits[rbx * 1] ; Get least sig digit + 1
            add     al, 5               ; Round (for example, +0.5)
            cmp     al, '9'
            jbe     noNeedToRound
            mov     Digits[rbx * 1], '9' + 1
            jmp     whileDigitGT9Test

whileDigitGT9:

; Subtract out overflow and add the carry into the previous
; digit (unless we hit the first digit in the number).

            sub     Digits[rbx * 1], 10     
            dec     ebx                     
            cmp     ebx, 0                  
            jl      firstDigitInNumber      

            inc     Digits[rbx * 1]
            jmp     whileDigitGT9Test

firstDigitInNumber:

; If we get to this point, then we've hit the first
; digit in the number.  So we've got to shift all
; the characters down one position in the string of
; bytes and put a "1" in the first character position.

            mov     ebx, 17
repeatUntilEBXeq0:

            mov     al, Digits[rbx * 1]
            mov     Digits[rbx * 1 + 1], al
            dec     ebx
            jnz     repeatUntilEBXeq0

            mov     Digits, '1'
            inc     Exponent         ; Because we added a digit
            jmp     noNeedToRound

whileDigitGT9Test:
            cmp     Digits[rbx], '9' ; Overflow if char > "9"
            ja      whileDigitGT9 

noNeedToRound:      

; Okay, emit the string at this point.  This is pretty easy
; since all we really need to do is copy data from the
; digits array and add an exponent (plus a few other simple chars).

            xor     ecx, ecx    ; Count output mantissa digits
            mov     rdi, buffer
            xor     edx, edx    ; Count output chars
            mov     al, Sign
            cmp     al, '-'
            je      noMinus

            mov     al, ' '

noMinus:    mov     [rdi], al

; Output the first character and a following decimal point
; if there are more than two mantissa digits to output.

            mov     al, Digits
            mov     [rdi + 1], al
            add     rdi, 2
            add     edx, 2
            inc     ecx
            cmp     ecx, MantSize
            je      noDecPt

            mov     al, '.'
            mov     [rdi], al
            inc     rdi
            inc     edx

noDecPt:

; Output any remaining mantissa digits here.
; Note that if the caller requests the output of
; more than 18 digits, this routine will output zeros
; for the additional digits.

            jmp     whileECXltMantSizeTest

whileECXltMantSize:

            mov     al, '0'
            cmp     ecx, 18
            jae     justPut0

            mov     al, Digits[rcx * 1]

justPut0:
            mov     [rdi], al
            inc     rdi
            inc     ecx
            inc     edx

whileECXltMantSizeTest:
            cmp     ecx, MantSize
            jb      whileECXltMantSize

; Output the exponent:

            mov     byte ptr [rdi], 'e'
            inc     rdi
            inc     edx
            mov     al, '+'
            cmp     Exponent, 0
            jge     noNegExp

            mov     al, '-'
            neg     Exponent

noNegExp:
            mov     [rdi], al
            inc     rdi
            inc     edx

            mov     eax, Exponent
            mov     ecx, expDigs
            call    expToBuf
            jc      error

exit_eToBuf:
            mov     rsi, rsiSave
            mov     rcx, rcxSave
            mov     rbx, rbxSave
            mov     rax, fWidth
            mov     rdx, expDigs
            leave
            clc
            ret

strOvfl:    mov     rax, -3
            jmp     error

badWidth:   mov     rax, -2
            jmp     error

voor:       mov     rax, -1
error:      mov     rsi, rsiSave
            mov     rcx, rcxSave
            mov     rbx, rbxSave
            mov     rdx, expDigs
            leave
            stc
            ret

e10ToStr   endp
```

列表 9-13: `e10ToStr` 转换函数

## 9.2 字符串与数字转换例程

数值到字符串的转换例程和字符串到数字的转换例程有一些基本的区别。首先，数字到字符串的转换通常不会发生错误；^(4) 而字符串到数字的转换则必须处理实际可能出现的错误，如非法字符和数字溢出。

一个典型的数字输入操作包括从用户读取一串字符，然后将这串字符转换为内部数字表示。例如，在 C++ 中，像 `cin >> i32;` 这样的语句从用户那里读取一行文本，并将该行文本开头的一串数字字符转换为一个 32 位带符号整数（假设 `i32` 是一个 32 位的 `int` 对象）。`cin >> i32;` 语句跳过某些字符，如开头的空格，这些字符可能出现在实际的数字字符之前。输入字符串也可能包含数字输入后的额外数据（例如，可能从同一行输入中读取两个整数值），因此输入转换例程必须确定数字数据在输入流中的结束位置。

通常，C++ 通过查找一组*分隔符*字符来实现这一点。分隔符字符集可能是简单的“任何非数字字符”，或者是空白字符集（空格、制表符等），也可能是其他一些字符，如逗号（`,`）或其他标点符号字符。为了举例说明，本节中的代码假设任何开头的空格或制表符字符（ASCII 码 9）可能出现在数字字符之前，转换在遇到第一个非数字字符时停止。可能的错误情况如下：

+   字符串开头完全没有数字（跳过任何空格或制表符）。

+   数字串是一个值，其大小超出了目标数字类型的范围（例如，64 位）。

由调用者来确定数字字符串是否以无效字符结尾（从函数调用返回时）。

### 9.2.1 将十进制字符串转换为整数

将包含十进制数字的字符串转换为数字的基本算法如下：

1.  初始化累加器变量为 0。

1.  跳过字符串中的任何前导空格或制表符。

1.  获取空格或制表符之后的第一个字符。

1.  如果字符不是数字字符，则返回错误。如果字符是数字字符，则继续到第 5 步。

1.  将数字字符转换为数值（使用 AND 0Fh）。

1.  设置累加器 =（累加器 × 10）+ 当前的数字值。

1.  如果发生溢出，返回并报告错误。如果没有溢出，继续执行第 8 步。

1.  从字符串中获取下一个字符。

1.  如果字符是数字字符，返回到第 5 步，否则继续到第 10 步。

1.  返回成功，累加器包含转换后的值。

对于有符号整数输入，您使用相同的算法，进行以下修改：

+   如果第一个非空格或制表符字符是一个连字符（`-`），则设置一个标志，表示该数字为负数，并跳过“`-`”字符（如果第一个字符不是`-`，则清除标志）。

+   在成功转换结束时，如果设置了标志，则在返回之前对整数结果取负（必须检查取负操作是否溢出）。

清单 9-14 实现了转换算法。

```
; Listing 9-14

; String-to-numeric conversion.

        option  casemap:none

false       =       0
true        =       1
tab         =       9
nl          =       10

            .const
ttlStr      byte    "Listing 9-14", 0
fmtStr1     byte    "strtou: String='%s'", nl
            byte    "    value=%I64u", nl, 0

fmtStr2     byte    "Overflow: String='%s'", nl
            byte    "    value=%I64x", nl, 0

fmtStr3     byte    "strtoi: String='%s'", nl
            byte    "    value=%I64i",nl, 0

unexError   byte    "Unexpected error in program", nl, 0

value1      byte    "  1", 0
value2      byte    "12 ", 0
value3      byte    " 123 ", 0
value4      byte    "1234", 0
value5      byte    "1234567890123456789", 0
value6      byte    "18446744073709551615", 0
OFvalue     byte    "18446744073709551616", 0
OFvalue2    byte    "999999999999999999999", 0

ivalue1     byte    "  -1", 0
ivalue2     byte    "-12 ", 0
ivalue3     byte    " -123 ", 0
ivalue4     byte    "-1234", 0
ivalue5     byte    "-1234567890123456789", 0
ivalue6     byte    "-9223372036854775807", 0
OFivalue    byte    "-9223372036854775808", 0
OFivalue2   byte    "-999999999999999999999", 0

            .data
buffer      byte    30 dup (?)

            .code
            externdef printf:proc

; Return program title to C++ program:

            public  getTitle
getTitle    proc
            lea     rax, ttlStr
            ret
getTitle    endp

; strtou -   Converts string data to a 64-bit unsigned integer.

; Input:
;   RDI  -   Pointer to buffer containing string to convert.

; Output:
;   RAX  -   Contains converted string (if success), error code
;            if an error occurs.

;   RDI  -   Points at first char beyond end of numeric string.
;            If error, RDI's value is restored to original value.
;            Caller can check character at [RDI] after a
;            successful result to see if the character following
;            the numeric digits is a legal numeric delimiter.

;   C    -   (carry flag) Set if error occurs, clear if
;            conversion was successful. On error, RAX will
;            contain 0 (illegal initial character) or
;            0FFFFFFFFFFFFFFFFh (overflow).

strtou      proc
            push    rdi      ; In case we have to restore RDI
            push    rdx      ; Munged by mul 
            push    rcx      ; Holds input char

 xor     edx, edx ; Zero-extends!
            xor     eax, eax ; Zero-extends!

; The following loop skips over any whitespace (spaces and
; tabs) that appears at the beginning of the string.

            dec     rdi      ; Because of inc below
skipWS:     inc     rdi
            mov     cl, [rdi]
            cmp     cl, ' '
            je      skipWS
            cmp     al, tab
            je      skipWS

; If we don't have a numeric digit at this point,
; return an error.

            cmp     cl, '0'  ; Note: "0" < "1" < ... < "9"
            jb      badNumber
            cmp     cl, '9'
            ja      badNumber

; Okay, the first digit is good. Convert the string
; of digits to numeric form:

convert:    and     ecx, 0fh ; Convert to numeric in RCX
            mul     ten      ; Accumulator *= 10
            jc      overflow
            add     rax, rcx ; Accumulator += digit
            jc      overflow
            inc     rdi      ; Move on to next character
            mov     cl, [rdi]
            cmp     cl, '0'
            jb      endOfNum
            cmp     cl, '9'
            jbe     convert

; If we get to this point, we've successfully converted
; the string to numeric form:

endOfNum:   pop     rcx
            pop     rdx

; Because the conversion was successful, this procedure
; leaves RDI pointing at the first character beyond the
; converted digits. As such, we don't restore RDI from
; the stack. Just bump the stack pointer up by 8 bytes
; to throw away RDI's saved value.

            add     rsp, 8
            clc              ; Return success in carry flag
            ret

; badNumber - Drop down here if the first character in
;             the string was not a valid digit.

badNumber:  mov     rax, 0
            pop     rcx
            pop     rdx
            pop     rdi
            stc              ; Return error in carry flag
            ret     

overflow:   mov     rax, -1  ; 0FFFFFFFFFFFFFFFFh
            pop     rcx
            pop     rdx
            pop     rdi
            stc              ; Return error in carry flag
            ret

ten         qword   10

strtou      endp

; strtoi - Converts string data to a 64-bit signed integer.

; Input:
;   RDI  -   Pointer to buffer containing string to convert.

; Output:
;   RAX  -   Contains converted string (if success), error code
;            if an error occurs.

;   RDI  -   Points at first char beyond end of numeric string.
;            If error, RDI's value is restored to original value.
;            Caller can check character at [RDI] after a
;            successful result to see if the character following
;            the numeric digits is a legal numeric delimiter.

;   C    -   (carry flag) Set if error occurs, clear if
;            conversion was successful. On error, RAX will
;            contain 0 (illegal initial character) or
;            0FFFFFFFFFFFFFFFFh (-1, indicating overflow).

strtoi      proc
negFlag     equ     <byte ptr [rsp]>

            push    rdi      ; In case we have to restore RDI
            sub     rsp, 8

; Assume we have a non-negative number.

            mov     negFlag, false

; The following loop skips over any whitespace (spaces and
; tabs) that appears at the beginning of the string.

            dec     rdi      ; Because of inc below
skipWS:     inc     rdi
            mov     al, [rdi]
            cmp     al, ' '
            je      skipWS
            cmp     al, tab
            je      skipWS

; If the first character we've encountered is "-",
; then skip it, but remember that this is a negative
; number.

            cmp     al, '-'
            jne     notNeg
            mov     negFlag, true
            inc     rdi             ; Skip "-"

notNeg:     call    strtou          ; Convert string to integer
            jc      hadError

; strtou returned success. Check the negative flag and
; negate the input if the flag contains true.

            cmp     negFlag, true
            jne     itsPosOr0

            cmp     rax, tooBig     ; Number is too big
            ja      overflow
            neg     rax
itsPosOr0:  add     rsp, 16         ; Success, so don't restore RDI
            clc                     ; Return success in carry flag
            ret

; If we have an error, we need to restore RDI from the stack:

overflow:   mov     rax, -1         ; Indicate overflow
hadError:   add     rsp, 8          ; Remove locals
            pop     rdi
            stc                     ; Return error in carry flag
            ret 

tooBig      qword   7fffffffffffffffh
strtoi      endp

; Here is the "asmMain" function.

            public  asmMain
asmMain     proc
            push    rbp
            mov     rbp, rsp
            sub     rsp, 64         ; Shadow storage

; Test unsigned conversions:

            lea     rdi, value1
            call    strtou

jc      UnexpectedError

            lea     rcx, fmtStr1
            lea     rdx, value1
            mov     r8, rax
            call    printf

            lea     rdi, value2
            call    strtou
            jc      UnexpectedError

            lea     rcx, fmtStr1
            lea     rdx, value2
            mov     r8, rax
            call    printf

            lea     rdi, value3
            call    strtou
            jc      UnexpectedError

            lea     rcx, fmtStr1
            lea     rdx, value3
            mov     r8, rax
            call    printf

            lea     rdi, value4
            call    strtou
            jc      UnexpectedError

            lea     rcx, fmtStr1
            lea     rdx, value4
            mov     r8, rax
            call    printf

            lea     rdi, value5
            call    strtou
            jc      UnexpectedError

            lea     rcx, fmtStr1
            lea     rdx, value5
            mov     r8, rax
            call    printf

            lea     rdi, value6
            call    strtou
            jc      UnexpectedError

            lea     rcx, fmtStr1
            lea     rdx, value6
            mov     r8, rax
            call    printf

 lea     rdi, OFvalue
            call    strtou
            jnc     UnexpectedError
            test    rax, rax        ; Nonzero for overflow
            jz      UnexpectedError

            lea     rcx, fmtStr2
            lea     rdx, OFvalue
            mov     r8, rax
            call    printf

            lea     rdi, OFvalue2
            call    strtou
            jnc     UnexpectedError
            test    rax, rax        ; Nonzero for overflow
            jz      UnexpectedError

            lea     rcx, fmtStr2
            lea     rdx, OFvalue2
            mov     r8, rax
            call    printf

; Test signed conversions:

            lea     rdi, ivalue1
            call    strtoi
            jc      UnexpectedError

            lea     rcx, fmtStr3
            lea     rdx, ivalue1
            mov     r8, rax
            call    printf

            lea     rdi, ivalue2
            call    strtoi
            jc      UnexpectedError

            lea     rcx, fmtStr3
            lea     rdx, ivalue2
            mov     r8, rax
            call    printf

            lea     rdi, ivalue3
            call    strtoi
            jc      UnexpectedError

            lea     rcx, fmtStr3
            lea     rdx, ivalue3
            mov     r8, rax
            call    printf

 lea     rdi, ivalue4
            call    strtoi
            jc      UnexpectedError

            lea     rcx, fmtStr3
            lea     rdx, ivalue4
            mov     r8, rax
            call    printf

            lea     rdi, ivalue5
            call    strtoi
            jc      UnexpectedError

            lea     rcx, fmtStr3
            lea     rdx, ivalue5
            mov     r8, rax
            call    printf

            lea     rdi, ivalue6
            call    strtoi
            jc      UnexpectedError

            lea     rcx, fmtStr3
            lea     rdx, ivalue6
            mov     r8, rax
            call    printf

            lea     rdi, OFivalue
            call    strtoi
            jnc     UnexpectedError
            test    rax, rax        ; Nonzero for overflow
            jz      UnexpectedError

            lea     rcx, fmtStr2
            lea     rdx, OFivalue
            mov     r8, rax
            call    printf

            lea     rdi, OFivalue2
            call    strtoi
            jnc     UnexpectedError
            test    rax, rax        ; Nonzero for overflow
            jz      UnexpectedError

            lea     rcx, fmtStr2
            lea     rdx, OFivalue2
            mov     r8, rax
            call    printf

            jmp     allDone

UnexpectedError:
            lea     rcx, unexError
            call    printf

allDone:    leave
            ret     ; Returns to caller
asmMain     endp
            end
```

清单 9-14：数字到字符串的转换

以下是该程序的构建命令和示例输出：

```
C:\>**build listing9-14**

C:\>**echo off**
 Assembling: listing9-14.asm
c.cpp

C:\>**listing9-14**
Calling Listing 9-14:
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
strtoi: String='-9223372036854775807'
    value=-9223372036854775807
Overflow: String='-9223372036854775808'
    value=ffffffffffffffff
Overflow: String='-999999999999999999999'
    value=ffffffffffffffff
Listing 9-14 terminated
```

对于扩展精度的字符串到数字转换，您只需修改`strtou`函数，使其具有扩展精度累加器，然后进行扩展精度的乘法（而不是标准乘法）。

### 9.2.2 将十六进制字符串转换为数字形式

与数字输出类似，十六进制输入是最容易编写的数字输入程序。十六进制字符串到数字转换的基本算法如下：

1.  将扩展精度累加器值初始化为 0。

1.  对于每个有效的十六进制数字字符，重复步骤 3 到 6；如果不是有效的十六进制数字字符，则跳到步骤 7。

1.  将十六进制字符转换为 0 到 15（0h 到 0Fh）范围内的值。

1.  如果扩展精度累加器值的高 4 位非零，则引发异常。

1.  将当前的扩展精度值乘以 16（即向左移动 4 位）。

1.  将转换后的十六进制数字值添加到累加器中。

1.  检查当前输入字符以确保它是一个有效的分隔符。如果不是，则引发异常。

清单 9-15 实现了这个 64 位值的扩展精度十六进制输入程序。

```
; Listing 9-15

; Hexadecimal string-to-numeric conversion.

        option  casemap:none

false       =       0
true        =       1
tab         =       9
nl          =       10

            .const
ttlStr      byte    "Listing 9-15", 0
fmtStr1     byte    "strtoh: String='%s' "
            byte    "value=%I64x", nl, 0

fmtStr2     byte    "Error, RAX=%I64x, str='%s'", nl, 0 
fmtStr3     byte    "Error, expected overflow: RAX=%I64x, "
            byte    "str='%s'", nl, 0

fmtStr4     byte    "Error, expected bad char: RAX=%I64x, "
            byte    "str='%s'", nl, 0 

hexStr      byte    "1234567890abcdef", 0
hexStrOVFL  byte    "1234567890abcdef0", 0
hexStrBAD   byte    "x123", 0

            .code
            externdef printf:proc

; Return program title to C++ program:

            public  getTitle
getTitle    proc
            lea     rax, ttlStr
            ret
getTitle    endp

; strtoh -   Converts string data to a 64-bit unsigned integer.

; Input:
;   RDI  -   Pointer to buffer containing string to convert.

; Output:
;   RAX  -   Contains converted string (if success), error code
;            if an error occurs.

;   RDI  -   Points at first char beyond end of hexadecimal string.
;            If error, RDI's value is restored to original value.
;            Caller can check character at [RDI] after a
;            successful result to see if the character following
;            the numeric digits is a legal numeric delimiter.

;   C    -   (carry flag) Set if error occurs, clear if
;            conversion was successful. On error, RAX will
;            contain 0 (illegal initial character) or
;            0FFFFFFFFFFFFFFFFh (overflow).

strtoh      proc
            push    rcx      ; Holds input char
            push    rdx      ; Special mask value
            push    rdi      ; In case we have to restore RDI

; This code will use the value in RDX to test and see if overflow
; will occur in RAX when shifting to the left 4 bits:

            mov     rdx, 0F000000000000000h
            xor     eax, eax ; Zero out accumulator

; The following loop skips over any whitespace (spaces and
; tabs) that appears at the beginning of the string.

            dec     rdi      ; Because of inc below
skipWS:     inc     rdi
            mov     cl, [rdi]
            cmp     cl, ' '
            je      skipWS
            cmp     al, tab
            je      skipWS

; If we don't have a hexadecimal digit at this point,
; return an error.

 cmp     cl, '0'  ; Note: "0" < "1" < ... < "9"
            jb      badNumber
            cmp     cl, '9'
            jbe     convert
            and     cl, 5fh  ; Cheesy LC -> UC conversion
            cmp     cl, 'A'
            jb      badNumber
            cmp     cl, 'F'
            ja      badNumber
            sub     cl, 7    ; Maps 41h to 46h -> 3Ah to 3Fh

; Okay, the first digit is good. Convert the string
; of digits to numeric form:

convert:    test    rdx, rax ; See if adding in the current
            jnz     overflow ; digit will cause an overflow

            and     ecx, 0fh ; Convert to numeric in RCX

; Multiply 64-bit accumulator by 16 and add in new digit:

            shl     rax, 4
            add     al, cl   ; Never overflows outside LO 4 bits

; Move on to next character:

            inc     rdi
            mov     cl, [rdi]
            cmp     cl, '0'
            jb      endOfNum
            cmp     cl, '9'
            jbe     convert

            and     cl, 5fh  ; Cheesy LC -> UC conversion
            cmp     cl, 'A'
            jb      endOfNum
            cmp     cl, 'F'
            ja      endOfNum
            sub     cl, 7    ; Maps 41h to 46h -> 3Ah to 3Fh
            jmp     convert

; If we get to this point, we've successfully converted
; the string to numeric form:

endOfNum:

; Because the conversion was successful, this procedure
; leaves RDI pointing at the first character beyond the
; converted digits. As such, we don't restore RDI from
; the stack. Just bump the stack pointer up by 8 bytes
; to throw away RDI's saved value.

 add     rsp, 8   ; Remove original RDI value
            pop     rdx      ; Restore RDX
            pop     rcx      ; Restore RCX
            clc              ; Return success in carry flag
            ret

; badNumber- Drop down here if the first character in
;            the string was not a valid digit.

badNumber:  xor     rax, rax
            jmp     errorExit

overflow:   or      rax, -1  ; Return -1 as error on overflow
errorExit:  pop     rdi      ; Restore RDI if an error occurs
            pop     rdx
            pop     rcx
            stc              ; Return error in carry flag
            ret

strtoh      endp

; Here is the "asmMain" function.

            public  asmMain
asmMain     proc
            push    rbp
            mov     rbp, rsp
            sub     rsp, 64  ; Shadow storage

; Test hexadecimal conversion:

            lea     rdi, hexStr
            call    strtoh
            jc      error

            lea     rcx, fmtStr1
            mov     r8, rax
            lea     rdx, hexStr
            call    printf

; Test overflow conversion:

            lea     rdi, hexStrOVFL
            call    strtoh
            jnc     unexpected

            lea     rcx, fmtStr2
            mov     rdx, rax
            mov     r8, rdi
            call    printf

; Test bad character:

            lea     rdi, hexStrBAD
            call    strtoh
            jnc     unexp2

            lea     rcx, fmtStr2
            mov     rdx, rax
            mov     r8, rdi
            call    printf
            jmp     allDone

unexpected: lea     rcx, fmtStr3
            mov     rdx, rax
            mov     r8, rdi
            call    printf
            jmp     allDone

unexp2:     lea     rcx, fmtStr4
            mov     rdx, rax
            mov     r8, rdi
            call    printf
            jmp     allDone

error:      lea     rcx, fmtStr2
            mov     rdx, rax
            mov     r8, rdi
            call    printf

allDone:    leave
            ret     ; Returns to caller
asmMain     endp
            end
```

清单 9-15：十六进制字符串到数字的转换

以下是构建命令和程序输出：

```
C:\>**build listing9-15**

C:\>**echo off**
 Assembling: listing9-15.asm
c.cpp

C:\>**listing9-15**
Calling Listing 9-15:
strtoh: String='1234567890abcdef' value=1234567890abcdef
Error, RAX=ffffffffffffffff, str='1234567890abcdef0'
Error, RAX=0, str='x123'
Listing 9-15 terminated
```

对于处理大于 64 位数字的十六进制字符串转换，你需要使用扩展精度左移 4 位。列表 9-16 演示了对 `strtoh` 函数进行必要修改以支持 128 位转换。

```
; strtoh128 - Converts string data to a 128-bit unsigned integer.

; Input:
;   RDI     - Pointer to buffer containing string to convert.

; Output:
;   RDX:RAX - Contains converted string (if success), error code
;             if an error occurs.

;   RDI     - Points at first char beyond end of hex string.
;             If error, RDI's value is restored to original value.
;             Caller can check character at [RDI] after a
;             successful result to see if the character following
;             the numeric digits is a legal numeric delimiter.

;   C       - (carry flag) Set if error occurs, clear if
;             conversion was successful. On error, RAX will
;             contain 0 (illegal initial character) or
;             0FFFFFFFFFFFFFFFFh (overflow).

strtoh128   proc
            push    rbx      ; Special mask value
            push    rcx      ; Input char to process
            push    rdi      ; In case we have to restore RDI

; This code will use the value in RDX to test and see if overflow
; will occur in RAX when shifting to the left 4 bits:

            mov     rbx, 0F000000000000000h
            xor     eax, eax ; Zero out accumulator
            xor     edx, edx

; The following loop skips over any whitespace (spaces and
; tabs) that appears at the beginning of the string.

            dec     rdi      ; Because of inc below
skipWS:     inc     rdi
            mov     cl, [rdi]
            cmp     cl, ' '
            je      skipWS
            cmp     al, tab
            je      skipWS

; If we don't have a hexadecimal digit at this point,
; return an error.

            cmp     cl, '0'  ; Note: "0" < "1" < ... < "9"
            jb      badNumber
            cmp     cl, '9'
            jbe     convert
 and     cl, 5fh  ; Cheesy LC -> UC conversion
            cmp     cl, 'A'
            jb      badNumber
            cmp     cl, 'F'
            ja      badNumber
            sub     cl, 7    ; Maps 41h to 46h -> 3Ah to 3Fh

; Okay, the first digit is good. Convert the string
; of digits to numeric form:

convert:    test    rdx, rbx ; See if adding in the current
            jnz     overflow ; digit will cause an overflow

            and     ecx, 0fh ; Convert to numeric in RCX

; Multiply 64-bit accumulator by 16 and add in new digit:

            shld    rdx, rax, 4
            shl     rax, 4
            add     al, cl   ; Never overflows outside LO 4 bits

; Move on to next character:

            inc     rdi      
            mov     cl, [rdi]
            cmp     cl, '0'
            jb      endOfNum
            cmp     cl, '9'
            jbe     convert

            and     cl, 5fh  ; Cheesy LC -> UC conversion
            cmp     cl, 'A'
            jb      endOfNum
            cmp     cl, 'F'
            ja      endOfNum
            sub     cl, 7    ; Maps 41h to 46h -> 3Ah to 3Fh
            jmp     convert

; If we get to this point, we've successfully converted
; the string to numeric form:

endOfNum:

; Because the conversion was successful, this procedure
; leaves RDI pointing at the first character beyond the
; converted digits. As such, we don't restore RDI from
; the stack. Just bump the stack pointer up by 8 bytes
; to throw away RDI's saved value.

            add     rsp, 8   ; Remove original RDI value
            pop     rcx      ; Restore RCX
            pop     rbx      ; Restore RBX
            clc              ; Return success in carry flag
            ret

; badNumber - Drop down here if the first character in
;             the string was not a valid digit.

badNumber:  xor     rax, rax
            jmp     errorExit

overflow:   or      rax, -1  ; Return -1 as error on overflow
errorExit:  pop     rdi      ; Restore RDI if an error occurs
            pop     rcx
            pop     rbx
            stc              ; Return error in carry flag
            ret

strtoh128   endp
```

列表 9-16：128 位十六进制字符串到数值的转换

### 9.2.3 无符号十进制字符串转换为整数

无符号十进制输入的算法与十六进制输入几乎完全相同。事实上，唯一的区别（除了仅接受十进制数字外）是，对于每个输入字符，你将累积值乘以 10 而不是 16（一般来说，任何进制的算法都是一样的；只需将累积值乘以输入的进制）。列表 9-17 演示了如何编写一个 64 位无符号十进制输入例程。

```
; Listing 9-17

; 64-bit unsigned decimal string-to-numeric conversion.

        option  casemap:none

false       =       0
true        =       1
tab         =       9
nl          =       10

            .const
ttlStr      byte    "Listing 9-17", 0
fmtStr1     byte    "strtou: String='%s' value=%I64u", nl, 0
fmtStr2     byte    "strtou: error, rax=%d", nl, 0

qStr      byte    "12345678901234567", 0

            .code
            externdef printf:proc

; Return program title to C++ program:

            public  getTitle
getTitle    proc
            lea     rax, ttlStr
 ret
getTitle    endp

; strtou -   Converts string data to a 64-bit unsigned integer.

; Input:
;   RDI  -   Pointer to buffer containing string to convert.

; Output:
;   RAX  -   Contains converted string (if success), error code
;            if an error occurs.

;   RDI  -   Points at first char beyond end of numeric string.
;            If error, RDI's value is restored to original value.
;            Caller can check character at [RDI] after a
;            successful result to see if the character following
;            the numeric digits is a legal numeric delimiter.

;   C    -   (carry flag) Set if error occurs, clear if
;            conversion was successful. On error, RAX will
;            contain 0 (illegal initial character) or
;            0FFFFFFFFFFFFFFFFh (overflow).

strtou      proc
            push    rcx      ; Holds input char
            push    rdx      ; Save, used for multiplication
            push    rdi      ; In case we have to restore RDI

            xor     rax, rax ; Zero out accumulator

; The following loop skips over any whitespace (spaces and
; tabs) that appears at the beginning of the string.

            dec     rdi      ; Because of inc below
skipWS:     inc     rdi
            mov     cl, [rdi]
            cmp     cl, ' '
            je      skipWS
            cmp     al, tab
            je      skipWS

; If we don't have a numeric digit at this point,
; return an error.

            cmp     cl, '0'  ; Note: "0" < "1" < ... < "9"
            jb      badNumber
            cmp     cl, '9'
            ja      badNumber

; Okay, the first digit is good. Convert the string
; of digits to numeric form:

convert:    and     ecx, 0fh ; Convert to numeric in RCX

; Multiple 64-bit accumulator by 10:

            mul     ten
            test    rdx, rdx ; Test for overflow
            jnz     overflow

            add     rax, rcx
            jc      overflow

; Move on to next character:

            inc     rdi
            mov     cl, [rdi]
            cmp     cl, '0'
            jb      endOfNum
            cmp     cl, '9'
            jbe     convert

; If we get to this point, we've successfully converted
; the string to numeric form:

endOfNum:

; Because the conversion was successful, this procedure
; leaves RDI pointing at the first character beyond the
; converted digits. As such, we don't restore RDI from
; the stack. Just bump the stack pointer up by 8 bytes
; to throw away RDI's saved value.

            add     rsp, 8   ; Remove original RDI value
            pop     rdx
            pop     rcx      ; Restore RCX
            clc              ; Return success in carry flag
            ret

; badNumber - Drop down here if the first character in
;             the string was not a valid digit.

badNumber:  xor     rax, rax
            jmp     errorExit

overflow:   mov     rax, -1  ; 0FFFFFFFFFFFFFFFFh
errorExit:  pop     rdi
            pop     rdx
            pop     rcx
            stc              ; Return error in carry flag
            ret

ten         qword   10

strtou      endp

; Here is the "asmMain" function.

 public  asmMain
asmMain     proc
            push    rbp
            mov     rbp, rsp
            sub     rsp, 64  ; Shadow storage

; Test hexadecimal conversion:

            lea     rdi, qStr
            call    strtou
            jc      error

            lea     rcx, fmtStr1
            mov     r8, rax
            lea     rdx, qStr
            call    printf
            jmp     allDone

error:      lea     rcx, fmtStr2
            mov     rdx, rax
            call    printf

allDone:    leave
            ret     ; Returns to caller
asmMain     endp
            end
```

列表 9-17：无符号十进制字符串到数值的转换

这是列表 9-17 中程序的构建命令和示例输出：

```
C:\>**build listing9-17**

C:\>**echo off**
 Assembling: listing9-17.asm
c.cpp

C:\>**listing9-17**
Calling Listing 9-17:
strtou: String='12345678901234567' value=12345678901234567
Listing 9-17 terminated
```

是否可以创建一个更快的函数，使用 `fbld`（x87 FPU BCD 存储）指令？可能不行。`fbstp` 指令在整数转换中要快得多，因为标准算法使用了多次执行（非常慢的）`div` 指令。十进制到数值的转换使用的是 `mul` 指令，这比 `div` 快得多。虽然我没有实际尝试过，但我怀疑使用 `fbld` 不会产生更快的运行代码。

### 9.2.4 扩展精度字符串转换为无符号整数

（十进制）字符串到数值的转换算法是相同的，无论整数的大小如何。你读取一个十进制字符，将其转换为整数，将累积结果乘以 10，然后将转换后的字符加进去。对于大于 64 位的值，唯一变化的是乘以 10 和加法操作。例如，要将一个字符串转换为 128 位整数，你需要能够将 128 位的值乘以 10，并将一个 8 位值（零扩展到 128 位）加到 128 位值上。

列表 9-18 演示了如何编写一个 128 位无符号十进制输入例程。除了 128 位乘以 10 和 128 位加法操作外，这段代码在功能上与 64 位字符串到整数的转换完全相同。

```
; strtou128 - Converts string data to a 128-bit unsigned integer.

; Input:
;   RDI     - Pointer to buffer containing string to convert.

; Output:
;   RDX:RAX - Contains converted string (if success), error code
;             if an error occurs.

;   RDI     - Points at first char beyond end of numeric string.
;             If error, RDI's value is restored to original value.
;             Caller can check character at [RDI] after a
;             successful result to see if the character following
;             the numeric digits is a legal numeric delimiter.

;   C       - (carry flag) Set if error occurs, clear if
;             conversion was successful. On error, RAX will
;             contain 0 (illegal initial character) or
;             0FFFFFFFFFFFFFFFFh (overflow).

strtou128   proc
accumulator equ     <[rbp - 16]>
partial     equ     <[rbp - 24]>
            push    rcx      ; Holds input char
            push    rdi      ; In case we have to restore RDI
            push    rbp
            mov     rbp, rsp
            sub     rsp, 24  ; Accumulate result here

            xor     edx, edx ; Zero-extends!
            mov     accumulator, rdx
            mov     accumulator[8], rdx

; The following loop skips over any whitespace (spaces and
; tabs) that appears at the beginning of the string.

            dec     rdi      ; Because of inc below
skipWS:     inc     rdi
            mov     cl, [rdi]
 cmp     cl, ' '
            je      skipWS
            cmp     al, tab
            je      skipWS

; If we don't have a numeric digit at this point,
; return an error.

            cmp     cl, '0'         ; Note: "0" < "1" < ... < "9"
            jb      badNumber
            cmp     cl, '9'
            ja      badNumber

; Okay, the first digit is good. Convert the string
; of digits to numeric form:

convert:    and     ecx, 0fh        ; Convert to numeric in RCX

; Multiply 128-bit accumulator by 10:

            mov     rax, accumulator 
            mul     ten
            mov     accumulator, rax
            mov     partial, rdx    ; Save partial product
            mov     rax, accumulator[8]
            mul     ten
            jc      overflow1
            add     rax, partial
            mov     accumulator[8], rax
            jc      overflow1

; Add in the current character to the 128-bit accumulator:

            mov     rax, accumulator
            add     rax, rcx
            mov     accumulator, rax
            mov     rax, accumulator[8]
            adc     rax, 0
            mov     accumulator[8], rax
            jc      overflow2

; Move on to next character:

            inc     rdi
            mov     cl, [rdi]
            cmp     cl, '0'
            jb      endOfNum
            cmp     cl, '9'
            jbe     convert

; If we get to this point, we've successfully converted
; the string to numeric form:

endOfNum:

; Because the conversion was successful, this procedure
; leaves RDI pointing at the first character beyond the
; converted digits. As such, we don't restore RDI from
; the stack. Just bump the stack pointer up by 8 bytes
; to throw away RDI's saved value.

            mov     rax, accumulator
            mov     rdx, accumulator[8]
            leave
            add     rsp, 8   ; Remove original RDI value
            pop     rcx      ; Restore RCX
            clc              ; Return success in carry flag
            ret

; badNumber - Drop down here if the first character in
;             the string was not a valid digit.

badNumber:  xor     rax, rax
            xor     rdx, rdx
            jmp     errorExit

overflow1:  mov     rax, -1
            cqo              ; RDX = -1, too
            jmp     errorExit

overflow2:  mov     rax, -2  ; 0FFFFFFFFFFFFFFFEh
            cqo              ; Just to be consistent
errorExit:  leave            ; Remove accumulator from stack
            pop     rdi
            pop     rcx
            stc              ; Return error in carry flag
            ret

ten         qword   10

strtou128   endp
```

列表 9-18：扩展精度无符号十进制输入

### 9.2.5 扩展精度有符号十进制字符串转换为整数

一旦你有了一个无符号十进制输入例程，编写一个有符号十进制输入例程就很简单，具体算法如下：

1.  消耗输入流开始部分的所有分隔符字符。

1.  如果下一个输入字符是减号，消耗此字符并设置一个标志，表示该数字是负数；否则直接跳到步骤 3。

1.  调用无符号十进制输入例程，将其余部分的字符串转换为整数。

1.  检查返回结果，确保其高位（HO）位是清除的。如果结果的高位是设置的，则引发超出范围的异常。

1.  如果代码在第 2 步中遇到了减号，则取结果的相反值。

我会把实际的代码实现留给你作为编程练习。

### 9.2.6 实现字符串到浮点数的转换

将表示浮点数的字符字符串转换为 80 位的`real10`格式，比本章前面出现的`real10`到字符串的转换稍微简单一些。因为十进制转换（没有指数）是更一般的科学计数法转换的一个子集，所以如果你能处理科学计数法，你就能免费处理十进制转换。除此之外，基本的算法是将尾数字符转换为压缩的 BCD 格式（这样该函数就可以使用`fbld`指令来进行字符串到数字的转换），然后读取（可选的）指数并相应地调整`real10`的指数。进行转换的算法如下：

1.  从去除任何前导的空格或制表符字符（以及其他分隔符）开始。

1.  检查是否有前导的加号（`+`）或减号（`-`）字符。如果有，跳过它。如果数字是负数，则将符号标志设置为真（如果是非负数，则设置为假）。

1.  初始化指数值为-18。该算法将根据字符串中的尾数数字创建一个左对齐的压缩 BCD 值，提供给`fbld`指令，而左对齐的压缩 BCD 值总是大于或等于 10¹⁸。初始化指数为-18 是为了考虑到这一点。

1.  初始化一个有效数字计数器变量，记录到目前为止已处理的有效数字的数量，初始值为 18。

1.  如果数字以任何前导零开头，则跳过这些零（不改变小数点左侧的前导零的指数或有效数字计数器）。

1.  如果扫描在处理完任何前导零后遇到小数点，则跳到第 11 步；否则跳到第 7 步。

1.  对于小数点左侧的每个非零数字，如果有效数字计数器不为零，将该非零数字插入到“数字字符串”数组中，位置由有效数字计数器（减去 1）指定。^(5) 请注意，这将以反向位置将字符插入到字符串中。

1.  对于小数点左侧的每个数字，将指数值（最初初始化为-18）增加 1。

1.  如果有效数字计数器不为零，递减有效数字计数器（这也将提供对数字字符串数组的索引）。

1.  如果遇到的第一个非数字字符不是小数点，跳到第 14 步。

1.  跳过小数点字符。

1.  对于小数点右侧的每个数字，继续将这些数字（按相反顺序）添加到数字字符串数组中，只要有效数字计数器不为零。如果有效数字计数器大于零，则递减它。同时，递减指数值。

1.  *如果算法在此时还未遇到至少一个十进制数字，则报告非法字符异常并返回*。

1.  如果当前字符不是`e`或`E`，则跳到步骤 20。^(6)否则，跳过`e`或`E`字符，继续执行步骤 15。

1.  如果下一个字符是`+`或`-`，则跳过它。如果符号字符是`-`，则将标志设置为 true，否则设置为 false（请注意，该指数符号标志与算法中较早设置的尾数符号标志不同）。

1.  如果下一个字符不是十进制数字，则报告错误。

1.  将数字字符串（从当前的十进制数字字符开始）转换为整数。

1.  将转换后的整数加到指数值上（该值在算法开始时被初始化为–18）。

1.  如果指数值超出了–4930 到+4930 的范围，则报告超出范围异常。

1.  将数字字符数组转换为 18 位（9 字节）打包的 BCD 值，通过去除每个字符的高 4 位，将成对的字符合并为一个字节（通过将奇数索引字节左移 4 位，并与每对中的偶数索引字节进行逻辑或运算），然后将高字节（第 10 个字节）设为 0。

1.  将打包的 BCD 值转换为`real10`值（使用`fbld`指令）。

1.  取指数的绝对值（但保留指数的符号）。该值将是 13 位或更小（4096 有第 12 位被设置，因此 4930 或更小的值会有一些第 0 到第 13 位的组合被设置为 1，其他位为 0）。

1.  如果指数为正，则对于指数中每一位被设置的位，将当前的`real10`值乘以 10 的该位指定的幂次方。例如，如果位 12、10 和 1 被设置，则将`real10`值分别乘以 10⁴⁰⁹⁶、10¹⁰²⁴和 10²。

1.  如果指数为负，则对于指数中每一位被设置的位，将当前的`real10`值除以 10 的该位指定的幂次方。例如，如果位 12、10 和 1 被设置，则将`real10`值分别除以 10⁴⁰⁹⁶、10¹⁰²⁴和 10²。

1.  如果尾数为负（算法开始时设置了第一个符号标志），则取反浮点数。

列表 9-19 提供了该算法的实现。

```
; Listing 9-19

; Real string-to-floating-point conversion.

        option  casemap:none

false       =       0
true        =       1
tab         =       9
nl          =       10

            .const
ttlStr      byte    "Listing 9-19", 0
fmtStr1     byte    "strToR10: str='%s', value=%e", nl, 0

fStr1a      byte    "1.234e56",0
fStr1b      byte    "-1.234e56",0
fStr1c      byte    "1.234e-56",0
fStr1d      byte    "-1.234e-56",0
fStr2a      byte    "1.23",0
fStr2b      byte    "-1.23",0
fStr3a      byte    "1",0
fStr3b      byte    "-1",0
fStr4a      byte    "0.1",0
fStr4b      byte    "-0.1",0
fStr4c      byte    "0000000.1",0
fStr4d      byte    "-0000000.1",0
fStr4e      byte    "0.1000000",0
fStr4f      byte    "-0.1000000",0
fStr4g      byte    "0.0000001",0
fStr4h      byte    "-0.0000001",0
fStr4i      byte    ".1",0
fStr4j      byte    "-.1",0

values      qword   fStr1a, fStr1b, fStr1c, fStr1d,
                    fStr2a, fStr2b,
                    fStr3a, fStr3b,
                    fStr4a, fStr4b, fStr4c, fStr4d,
                    fStr4e, fStr4f, fStr4g, fStr4h,
                    fStr4i, fStr4j,
                    0

            align   4
PotTbl      real10  1.0e+4096,
                    1.0e+2048,
 1.0e+1024,
                    1.0e+512,
                    1.0e+256,
                    1.0e+128,
                    1.0e+64,
                    1.0e+32,
                    1.0e+16,
                    1.0e+8,
                    1.0e+4,
                    1.0e+2,
                    1.0e+1,
                    1.0e+0

            .data
r8Val       real8   ?

            .code
            externdef printf:proc

; Return program title to C++ program:

            public  getTitle
getTitle    proc
            lea     rax, ttlStr
            ret
getTitle    endp

*********************************************************

; strToR10 - RSI points at a string of characters that represent a
;            floating-point value. This routine converts that string
;            to the corresponding FP value and leaves the result on
;            the top of the FPU stack. On return, ESI points at the
;            first character this routine couldn't convert.

; Like the other ATOx routines, this routine raises an
; exception if there is a conversion error or if ESI
; contains NULL.

*********************************************************

strToR10    proc

sign        equ     <cl>
expSign     equ     <ch>

DigitStr    equ     <[rbp - 20]>
BCDValue    equ     <[rbp - 30]>
rsiSave     equ     <[rbp - 40]>

            push    rbp
            mov     rbp, rsp
            sub     rsp, 40

            push    rbx
 push    rcx
            push    rdx
            push    r8
            push    rax

; Verify that RSI is not NULL.

            test    rsi, rsi
            jz      refNULL

; Zero out the DigitStr and BCDValue arrays.

            xor     rax, rax
            mov     qword ptr DigitStr, rax
            mov     qword ptr DigitStr[8], rax
            mov     dword ptr DigitStr[16], eax

            mov     qword ptr BCDValue, rax
            mov     word ptr BCDValue[8], ax

; Skip over any leading space or tab characters in the sequence.

            dec     rsi
whileDelimLoop:
            inc     rsi
            mov     al, [rsi]
            cmp     al, ' '
            je      whileDelimLoop
            cmp     al, tab
            je      whileDelimLoop

; Check for "+" or "-".

            cmp     al, '-'
            sete    sign
            je      doNextChar
            cmp     al, '+'
            jne     notPlus
doNextChar: inc     rsi             ; Skip the "+" or "-"
            mov     al, [rsi]

notPlus:

; Initialize EDX with -18 since we have to account
; for BCD conversion (which generates a number * 10¹⁸ by
; default). EDX holds the value's decimal exponent.

            mov     rdx, -18

; Initialize EBX with 18, which is the number of significant
; digits left to process and it is also the index into the
; DigitStr array.

 mov     ebx, 18         ; Zero-extends!

; At this point, we're beyond any leading sign character.
; Therefore, the next character must be a decimal digit
; or a decimal point.

            mov     rsiSave, rsi    ; Save to look ahead 1 digit
            cmp     al, '.'
            jne     notPeriod

; If the first character is a decimal point, then the
; second character needs to be a decimal digit.

            inc     rsi
            mov     al, [rsi]

notPeriod:
            cmp     al, '0'
            jb      convError
            cmp     al, '9'
            ja      convError
            mov     rsi, rsiSave    ; Go back to orig char
            mov     al, [rsi]
            jmp     testWhlAL0

; Eliminate any leading zeros (they do not affect the value or
; the number of significant digits).

whileAL0:   inc     rsi
            mov     al, [rsi]
testWhlAL0: cmp     al, '0'
            je      whileAL0

; If we're looking at a decimal point, we need to get rid of the
; zeros immediately after the decimal point since they don't
; count as significant digits.  Unlike zeros before the decimal
; point, however, these zeros do affect the number's value as
; we must decrement the current exponent for each such zero.

            cmp     al, '.'
            jne     testDigit

            inc     edx             ; Counteract dec below
repeatUntilALnot0:
            dec     edx
            inc     rsi
            mov     al, [rsi]
            cmp     al, '0'
            je      repeatUntilALnot0
            jmp     testDigit2

; If we didn't encounter a decimal point after removing leading
; zeros, then we've got a sequence of digits before a decimal
; point.  Process those digits here.

; Each digit to the left of the decimal point increases
; the number by an additional power of 10\.  Deal with
; that here.

whileADigit:
            inc     edx     

; Save all the significant digits, but ignore any digits
; beyond the 18th digit.

            test    ebx, ebx
            jz      Beyond18

            mov     DigitStr[rbx * 1], al
            dec     ebx

Beyond18:   inc     rsi
            mov     al, [rsi]

testDigit:  
            sub     al, '0'
            cmp     al, 10
            jb      whileADigit

            cmp     al, '.'-'0'
            jne     testDigit2

            inc     rsi             ; Skip over decimal point
            mov     al, [rsi]
            jmp     testDigit2

; Okay, process any digits to the right of the decimal point.

whileDigit2:
            test    ebx, ebx
            jz      Beyond18_2

            mov     DigitStr[rbx * 1], al
            dec     ebx

Beyond18_2: inc     rsi
            mov     al, [rsi]

testDigit2: sub     al, '0'
            cmp     al, 10
            jb      whileDigit2

; At this point, we've finished processing the mantissa.
; Now see if there is an exponent we need to deal with.

            mov     al, [rsi]       
            cmp     al, 'E'
            je      hasExponent
            cmp     al, 'e'
            jne     noExponent

hasExponent:
            inc     rsi
            mov     al, [rsi]       ; Skip the "E".
            cmp     al, '-'
            sete    expSign
            je      doNextChar_2
            cmp     al, '+'
            jne     getExponent;

doNextChar_2:
            inc     rsi             ; Skip "+" or "-"
            mov     al, [rsi]

; Okay, we're past the "E" and the optional sign at this
; point.  We must have at least one decimal digit.

getExponent:
            sub     al, '0'
            cmp     al, 10
            jae     convError

            xor     ebx, ebx        ; Compute exponent value in EBX
ExpLoop:    movzx   eax, byte ptr [rsi] ; Zero-extends to RAX!
            sub     al, '0'
            cmp     al, 10
            jae     ExpDone

            imul    ebx, 10
            add     ebx, eax
            inc     rsi
            jmp     ExpLoop

; If the exponent was negative, negate our computed result.

ExpDone:
            cmp     expSign, false
            je      noNegExp

            neg     ebx

noNegExp:

; Add in the BCD adjustment (remember, values in DigitStr, when
; loaded into the FPU, are multiplied by 10¹⁸ by default.
; The value in EDX adjusts for this).

            add     edx, ebx

noExponent:

; Verify that the exponent is between -4930 and +4930 (which
; is the maximum dynamic range for an 80-bit FP value).

            cmp     edx, 4930
            jg      voor            ; Value out of range
 cmp     edx, -4930
            jl      voor

; Now convert the DigitStr variable (unpacked BCD) to a packed
; BCD value.

            mov     r8, 8
for9:       mov     al, DigitStr[r8 * 2 + 2]
            shl     al, 4
            or      al, DigitStr[r8 * 2 + 1]
            mov     BCDValue[r8 * 1], al

            dec     r8
            jns     for9

            fbld    tbyte ptr BCDValue

; Okay, we've got the mantissa into the FPU.  Now multiply the
; mantissa by 10 raised to the value of the computed exponent
; (currently in EDX).

; This code uses power of 10 tables to help make the 
; computation a little more accurate.

; We want to determine which power of 10 is just less than the
; value of our exponent.  The powers of 10 we are checking are
; 10**4096, 10**2048, 10**1024, 10**512, and so on. A slick way to
; do this check is by shifting the bits in the exponent
; to the left.  Bit #12 is the 4096 bit.  So if this bit is set,
; our exponent is >= 10**4096\.  If not, check the next bit down
; to see if our exponent >= 10**2048, etc.

            mov     ebx, -10 ; Initial index into power of 10 table
            test    edx, edx
            jns     positiveExponent

; Handle negative exponents here.

            neg     edx
            shl     edx, 19 ; Bits 0 to 12 -> 19 to 31
            lea     r8, PotTbl

whileEDXne0:
            add     ebx, 10
            shl     edx, 1
            jnc     testEDX0

            fld     real10 ptr [r8][rbx * 1]
            fdivp

testEDX0:   test    edx, edx
            jnz     whileEDXne0
            jmp     doMantissaSign

; Handle positive exponents here.

positiveExponent:
            lea     r8, PotTbl
            shl     edx, 19 ; Bits 0 to 12 -> 19 to 31
            jmp     testEDX0_2

whileEDXne0_2:
            add     ebx, 10
            shl     edx, 1
            jnc     testEDX0_2

            fld     real10 ptr [r8][rbx * 1]
            fmulp

testEDX0_2: test    edx, edx
            jnz     whileEDXne0_2

; If the mantissa was negative, negate the result down here.

doMantissaSign:
            cmp     sign, false
            je      mantNotNegative

            fchs

mantNotNegative:
            clc                     ; Indicate success
            jmp     Exit

refNULL:    mov     rax, -3
            jmp     ErrorExit

convError:  mov     rax, -2
            jmp     ErrorExit

voor:       mov     rax, -1         ; Value out of range
            jmp     ErrorExit

illChar:    mov     rax, -4

ErrorExit:  stc                     ; Indicate failure
            mov     [rsp], rax      ; Save error code
Exit:       pop     rax
            pop     r8
            pop     rdx
            pop     rcx
            pop     rbx
            leave
            ret

strToR10    endp

; Here is the "asmMain" function.

            public  asmMain
asmMain     proc
            push    rbx
            push    rsi
            push    rbp
            mov     rbp, rsp
            sub     rsp, 64         ; Shadow storage

; Test floating-point conversion:

            lea     rbx, values
ValuesLp:   cmp     qword ptr [rbx], 0
            je      allDone

            mov     rsi, [rbx]
            call    strToR10
            fstp    r8Val

            lea     rcx, fmtStr1
            mov     rdx, [rbx]
            mov     r8, qword ptr r8Val
            call    printf
            add     rbx, 8
            jmp     ValuesLp

allDone:    leave
            pop     rsi
            pop     rbx
            ret     ; Returns to caller
asmMain     endp
            end
```

列表 9-19：`strToR10`函数

这里是列表 9-19 的构建命令和示例输出。

```
C:\>**build listing9-19**

C:\>**echo off**
 Assembling: listing9-19.asm
c.cpp

C:\>**listing9-19**
Calling Listing 9-19:
strToR10: str='1.234e56', value=1.234000e+56
strToR10: str='-1.234e56', value=-1.234000e+56
strToR10: str='1.234e-56', value=1.234000e-56
strToR10: str='-1.234e-56', value=-1.234000e-56
strToR10: str='1.23', value=1.230000e+00
strToR10: str='-1.23', value=-1.230000e+00
strToR10: str='1', value=1.000000e+00
strToR10: str='-1', value=-1.000000e+00
strToR10: str='0.1', value=1.000000e-01
strToR10: str='-0.1', value=-1.000000e-01
strToR10: str='0000000.1', value=1.000000e-01
strToR10: str='-0000000.1', value=-1.000000e-01
strToR10: str='0.1000000', value=1.000000e-01
strToR10: str='-0.1000000', value=-1.000000e-01
strToR10: str='0.0000001', value=1.000000e-07
strToR10: str='-0.0000001', value=-1.000000e-07
strToR10: str='.1', value=1.000000e-01
strToR10: str='-.1', value=-1.000000e-01
Listing 9-19 terminated
```

## 9.3 更多信息

唐纳德·克努斯的*《计算机程序设计的艺术》*第二卷：*半数值算法*（Addison-Wesley Professional，1997 年）包含了许多关于十进制算术和扩展精度算术的有用信息，尽管该文本是通用的，并没有描述如何在 x86 汇编语言中实现此操作。

## 9.4 自测

1.  将 8 位十六进制值从 AL 转换为两个十六进制数字（分别存入 AH 和 AL）的代码是什么？

1.  `dToStr`将生成多少个十六进制数字？

1.  解释如何使用`qToStr`编写一个 128 位十六进制输出例程。

1.  你应该使用什么指令来产生最快的 64 位十进制到字符串转换函数？

1.  如果给定一个无符号十进制到字符串的转换函数，如何编写一个有符号十进制到字符串的转换？

1.  `utoStrSize` 函数的参数是什么？

1.  如果数字需要的打印位置超过 `minDigits` 参数指定的数量，`uSizeToStr` 会输出什么字符串？

1.  `r10ToStr` 函数的参数是什么？

1.  如果输出不能适应 `fWidth` 参数指定的字符串大小，`r10ToStr` 会输出什么字符串？

1.  `e10ToStr` 函数的参数是什么？

1.  什么是分隔符字符？

1.  在字符串到数字的转换过程中，可能出现的两种错误是什么？
