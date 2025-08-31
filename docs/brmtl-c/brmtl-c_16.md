# 第十四章：缓冲区文件 I/O

![](img/chapterart.png)

在本书的第一部分，我们连最简单的控制台输出都很困难。但在这一部分，我们有了操作系统，这让处理输入输出变得更加简单。因为操作系统隐藏了大量的复杂性：你只需写 `"Hello World\n"`，操作系统就会将数据发送到合适的地方。

在这一章中，你将了解 C 语言的输入/输出系统，这不仅包括 `printf` 函数，还包括以高效且灵活的方式读写磁盘文件的函数。

## printf 函数

我们已经使用了几次 `printf` 函数进行简单的输出。该函数的基本格式是：

```
printf(`format-string`, `argument`, ...)
```

格式字符串告诉 `printf` 打印什么内容。除百分号（`%`）外的任何字符都会被打印。`%` 字符开始一个字段说明，告诉 `printf` 去参数列表中取下一个参数，并按照随后的字段说明进行打印。例如：

```
printf("Number: ->%d<-\n", 1234);   // Prints  ->1234<-
```

`%d` 字段说明可以通过数字进行修改：

```
printf("Number: ->%3d<-\n", 12);    // Prints  ->.12<- (using . for space)
printf("Number: ->%-3d<-\n", 12);   // Prints  ->12.<- (using . for space)
printf("Number: ->%3d<-\n", 1234);  // Prints  ->1234<-(at least 3 characters)
```

在这些例子中，`%3d` 告诉 `printf` 至少使用三个字符来打印数字。`%-3d` 字段告诉 `printf` 至少用三个字符打印数字，并左对齐。

到目前为止，我们只讨论了 `d` 转换字符，它用于将整数参数转换为文本以进行打印。表 14-1 列出了主要的转换字符。

表 14-1：主要的 C 语言转换字符

| **转换字符** | **参数类型** | **备注** |
| --- | --- | --- |
| `%d` | 整数 | `char` 和 `short int` 类型在作为参数传递时会被提升为 `int`，因此该格式同样适用于这三种类型。 |
| `%c` | 字符 | 由于提升，这个转换字符实际上接受一个整数参数，并将其打印为字符。 |
| `%o` | 整数 | 以八进制打印。 |
| `%x` | 整数 | 以十六进制打印。 |
| `%f` | 双精度浮点数 | 适用于 `float` 和 `double` 类型，因为所有 `float` 类型的参数在传递时会提升为 `double`。 |
| `%l` | 长整型 | `long int` 类型需要自己的转换，因为 `int` 类型不会自动提升为 `long int`。 |

### 编写 ASCII 表

让我们写一个简短的程序来创建一个表格，包含可打印字符及其十六进制和八进制值，这将展示格式化字符串的实际应用。这个程序（列表 14-1）让我们有机会以四种不同的方式表达相同的数据，并在 `printf` 语句中尝试不同的格式。

**ascii.c**

```
/**
 * Print ASCII character table (only printable characters).
 */

#include <stdio.h>

int main()
{
    for (char curChar = ' '; curChar <= '~'; ++curChar) {
        printf("Char: %c Decimal %3d Hex 0x%02x Octal 0%03o\n",
               curChar, curChar, curChar, curChar);
    }
    return (0);
}
```

列表 14-1：一个创建 ASCII 表的程序

首先，`%c`格式字符串以字符形式打印字符。接下来，我们以三位十进制数（`%3d`）打印字符。准确地说，参数的类型是字符，它被提升为整数。由于参数规范中的`3`，这个数字将是三位的。之后，我们使用`%02x`格式以十六进制打印。零（`0`）告诉`printf`如果需要填充零以匹配所需的宽度（当然，宽度是`2`）。最后，我们使用`%03o`格式以八进制打印。

Listing 14-2 显示了这个程序的输出。

```
Char:   Decimal  32 Hex 0x20 Octal 0040
Char: ! Decimal  33 Hex 0x21 Octal 0041
Char: " Decimal  34 Hex 0x22 Octal 0042
Char: # Decimal  35 Hex 0x23 Octal 0043
Char: $ Decimal  36 Hex 0x24 Octal 0044
Char: % Decimal  37 Hex 0x25 Octal 0045
Char: & Decimal  38 Hex 0x26 Octal 0046
Char: ' Decimal  39 Hex 0x27 Octal 0047
Char: ( Decimal  40 Hex 0x28 Octal 0050
`--snip--`
```

Listing 14-1 的输出（*ascii.c*）

`printf`函数是 C 语言 I/O 系统的核心工具。它帮助我们将各种不同类型的数据打印到控制台。但这不是我们唯一可以写入的地方，正如我们接下来几节所见。

### 写入预定义文件

当程序启动时，操作系统会打开三个预定义文件：

1.  `stdin` 标准输入，程序的正常输入

1.  `stdout` 标准输出，用于程序的正常输出

1.  `stderr` 标准错误，用于错误输出

默认情况下，这些文件连接到控制台，但你的命令行解释器可以将它们连接到磁盘文件、管道或其他地方。

`fprintf`函数将数据发送到指定的文件。例如：

```
fprintf(stdout, "Everything is OK\n");
fprintf(stderr, "ERROR: Something bad happened\n");
```

`printf`函数只是一个便利函数，它替代了`fprintf(stdout, ...)`。

## 读取数据

读取数据的函数设计得很简单，但不幸的是，它们并非如此。`printf`函数有一个对应的函数叫做`scanf`，用于读取数据。例如：

```
// Reads two numbers (do not use this code)
scanf("%d %d", &aInteger, &anotherInteger);
```

首先，注意在参数前面的符号（`&`），这是因为`scanf`需要修改参数；因此，参数必须通过地址传递。

传递给`scanf`的格式字符串看起来与`printf`的格式字符串非常相似，但`scanf`有一个大问题：除非你是一个极其专业的专家，否则你永远不知道它如何处理空白字符。所以，我们不使用它。

相反，我们使用`fgets`函数从输入中获取一行，然后使用`sscanf`解析得到的字符串：

```
fgets(line, sizeof(line), stdin);   // Read a line
sscanf(line, "%d %d", &aInteger, &anotherInteger);
```

`fgets`的一般形式是：

```
char* `result` = fgets(`buffer`, `size`, `file`);
```

其中，`result`是指向刚读取的字符串（`buffer`）的指针，或者当我们到达文件末尾（EOF）时是`NULL`。`buffer`是一个字符数组，用来存储读取的行，`file`是一个文件句柄，指示要读取的文件（此时我们只知道`stdin`）。

`buffer`将始终以 null 字符（`\0`）结尾，因此最多会将`size`-1 个字符放入`buffer`中。（即使`buffer`不够大，也会读取整行。）

`sscanf`函数与`scanf`函数非常相似，不同之处在于第一个参数现在是字符串。其余的参数相同。`sscanf`函数返回它转换的项数。

上述代码假设一切正常。让我们重写它，这次检查错误：

```
if (fgets(line, sizeof(line), stdin) == NULL) {
    fprintf(stderr, "ERROR: Expected two integers, got EOF\ n");
    return (ERROR);
}
if (sscanf(line, "%d %d", &aInteger, &anotherInteger) != 2) {
    fprintf(stderr, "ERROR: Expected two integers.\n");
    return (ERROR)
}
```

如果第一次调用`fgets`返回`NULL`，说明出了问题。然后我们会将错误信息打印到预定义的错误文件（`stderr`）并返回错误代码给调用者。接着，我们执行`sscanf`，它应该能找到两个整数。如果没有，我们再次打印错误信息并返回错误代码。

## 恶魔的`gets`函数

`fgets`函数有一个对应的简写函数来从`stdin`读取数据。它叫做`gets`，一般形式是：

```
`result` = gets(`buffer`);
```

`gets`函数读取一行数据并将其放入`buffer`中，*无论* `buffer` *是否能容纳它。*

当前的 GCC 编译器使得`gets`变得难以使用。首先，*stdio.h*不会定义它，除非你正确地定义一个条件编译宏。当你编译程序时，编译器会给出警告，接着当程序链接时，链接器也会给出警告。

示例 14-3 展示了使用`gets`编译程序时发生的情况。

```
$ **gcc -Wall -Wextra -o gets gets.c**
Agets.c: In function 'main':
gets.c:17:5: warning: 'gets' is deprecated [-Wdeprecated-declarations]
     gets(line);
 ^~~~
In file included from gets.c:11:0:
/usr/include/stdio.h:577:14: note: declared here
 extern char *gets (char *__s) __wur __attribute_deprecated__;
              ^~~~
/tmp/cc5H1KMF.o: In function `main':
gets.c:(.text+0x1f): warning: the `gets' function is dangerous and should not be used.
```

示例 14-3: 尝试使用`gets`

从输出量可以看出，GCC 编译器为了劝说你不要使用`gets`，付出了多大的努力。

现在我们已经看了一些不应该使用的东西，接下来让我们看一下应该使用的东西。

## 打开文件

预定义文件`stdin`、`stdout`和`stdout`是文件句柄。`fopen`函数允许你创建文件句柄。示例 14-4 展示了一个简单的例子。

**file.c**

```
#include <stdio.h>

int main()
{
  1 FILE* outFile = 2 fopen("hello.txt", "w");
    if (outFile == NULL) {
        fprintf(stderr, "ERROR: Unable to open 'hello.txt'\n");
        exit(8);
    }
    if (fprintf(outFile, "Hello World!\n") <= 0) {
        fprintf(stderr, "ERROR: Unable to write to 'hello.txt'\n");
        exit(8);
    }
    if (fclose(outFile) != 0) {
        fprintf(outfile, “ERROR: Unable to close 'hello.txt'\n");
        exit(8);
    }
    return (0);
}
```

示例 14-4: 一个文件版本的“Hello World”

首先，`FILE*`声明 1 声明一个新的文件句柄。所有文件操作都需要文件句柄。接下来是`fopen`调用 2，它的一般形式是：

```
`result` = fopen(`filename`, `mode`);
```

`mode`可以是以下之一：

1.  `r` 只读

1.  `w` 仅写

1.  `r+` 读写

1.  `a` 追加（写入但从文件末尾开始）

1.  `b` 与其他模式结合使用，用于二进制文件（将在下一节讨论）

现在我们已经打开文件，可以进行读写操作了。文本可以通过`fprintf`写入，通过`fgets`读取。接下来，让我们看看另一种类型的文件：二进制文件。

## 二进制 I/O

到目前为止，我们只限制在文本文件，但 C I/O 系统可以通过使用`fread`和`fwrite`函数处理二进制文件。`fread`函数的一般形式是：

```
`result` = fread(`buffer`, `elementSize`, `size`, `inFile`);
```

这里，`buffer`是指向数据缓冲区的指针，数据将被存放在该缓冲区中。`elementSize`始终为`1`（请参见下面的框进行解释）。`size`是缓冲区的大小，通常是`sizeof(``buffer``)`，`inFile`是要读取的文件。

该函数返回读取的项目数，由于`elementSize`为`1`，因此是读取的字节数。文件结束时返回`0`，如果发生 I/O 错误，则返回负数。

`fwrite`函数有一个类似的结构：

```
`result` = fwrite(`buffer`, `elementSize`, `size`, `inFile`);
```

一切都是一样的，只是写入数据而不是读取。

## 复制文件

我们将使用`fread`和`fwrite`调用来复制一个文件。由于我们还不知道如何在命令行中传递参数（见第十五章），文件名被硬编码为*infile.bin*和*outfile.bin*。示例 14-5 包含了相关代码。

**copy.c**

```
/**
 * Copy infile.bin to outfile.bin.
 */

#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
 int main()
{
    // The input file
  1 FILE* inFile = fopen("infile.bin", "rb");
    if (inFile == NULL) {
        fprintf(stderr, "ERROR: Could not open infile.bin\n");
        exit(8);
    }
    // The output file
    FILE* outFile = fopen("outfile.bin", "wb");
    if (outFile == NULL) {
        fprintf(stderr, "ERROR: Could not create outfile.bin\n");
        exit(8);
    }
    char buffer[512];   // A data buffer

    while (true) {
        // Read data, collect size
      2 ssize_t readSize = fread(buffer, 1, sizeof(buffer), inFile);
        if (readSize < 0) {
            fprintf(stderr, "ERROR: Read error seen\n");
            exit(8);
        }
      3 if (readSize == 0) {
            break;
        }
      4 if (fwrite(buffer, 1, readSize, outFile) !=(size_t)readSize) {
            fprintf(stderr, "ERROR: Write error seen\n");
            exit(8);
        }
    }
    fclose(inFile);
    fclose(outFile);
    return (0);
}
```

示例 14-5：复制一个文件

首先，注意`fopen`调用 1。我们使用`rb`模式打开文件，这告诉系统我们将读取文件（`r`）并且文件是二进制的（`b`）。

接下来，我们来看看`fread`调用 2。这个函数的返回值是`ssize_t`，它是一个标准类型，足够大可以存储可能存在的最大对象（结构体、数组、联合体）的大小。它还可以存储`-1`来表示错误条件。

如果我们已经从文件中读取了所有数据，`fread`会返回`0`。当发生这种情况时，表示我们已经完成，因此退出主循环 3。

现在我们来看看`fwrite`调用 4，它返回一个`size_t`值。这个是一个无符号类型，可以存储程序中能容纳的最大对象的大小，但由于它是无符号的，它不能存储错误值。当`fwrite`在写入时遇到错误时会发生什么？它会尽可能多地写入，并返回已写入的字节数，因此它永远不会返回错误代码，只会返回部分写入的字节数。

请注意，`fread`返回一个`ssize_t`类型的结果，而`fwrite`返回一个`size_t`类型的结果。这样做有其合理的原因，但也意味着，如果我们检查试图写入的字节数是否与实际要求`fwrite`写入的字节数相同，编译器会发出警告：

```
35          if (fwrite(`buffer`, 1, `readSize`, `outFile`) != `readSize`) {
                                      Warning: signed vs. unsigned compare
```

为了消除警告，我们需要插入一个类型转换，从而告诉 C 语言：“是的，我知道我们正在混合有符号和无符号类型，但我们必须这么做，因为`fread`和`fwrite`的定义很笨拙”：

```
if (fwrite(`buffer`, 1, `readSize`, `outFile`) != (size_t)`readSize`) {
```

另外请注意，在最后一次读取时，我们可能不会读取到完整的 512 字节。这就是为什么在`fwrite`语句中我们使用了`readSize`而不是`sizeof(``buffer``)`的原因。

## 缓冲区和刷新

C 的 I/O 系统使用*缓冲 I/O*，这意味着当你执行`printf`或`fwrite`时，数据可能不会立即发送到输出设备。相反，它将被存储在内存中，直到系统有足够的数据来提高效率。

发送到控制台的数据是*行缓冲*的，这意味着如果你只打印了一行的部分内容，它可能不会立即显示，直到该行的其他部分也发送出去。让我们看看这个程序是如何在示例 14-6 中给我们带来麻烦的。

```
/**
 * Demonstrate how buffering can fool
 * us with a divide-by-zero bug.
 */

#include <stdio.h>

int main()
{
    int zero = 0;    // The constant zero, to trick the
                     // compiler into letting us divide by 0
    int result;      // Something to put a result in

    printf("Before divide ");
    result = 5 / zero;
    printf("Divide done\n");
    printf("Result is %d\n", result);
    return (0);
}
```

示例 14-6：除以零

运行这个程序时，你会期望看到以下输出：

```
Before divide Floating point exception (core dumped)
```

但你实际看到的是：

```
Floating point exception (core dumped)
```

你最初可能认为`printf`没有执行，但实际上它执行了。数据进入了缓冲区，并在程序中止时停留在缓冲区，导致误导性的显示`printf`没有起作用。

为了解决这个问题，我们需要告诉 I/O 系统“现在写入缓冲区数据”，这可以通过`fflush`函数来完成：

```
 printf("Before divide ");   fflush(stdout);
```

刷新数据可以确保我们能够看到它。另一方面，我们不想在每次写入后都刷新，因为那样会违背缓冲区的目的，缓冲区的目的是提高 I/O 效率。

## 关闭文件

最后，在我们完成文件操作后，需要告诉 C 语言我们已经处理完该文件。我们通过使用`fclose`函数来完成：

```
int `result` = fclose(`file`);
```

其中，`file`是要关闭的`FILE*`，`result`如果成功返回`0`，如果失败则返回非零值。

## 总结

在嵌入式世界中，I/O 操作比较困难，因为你必须编写代码直接与设备交互，并且需要为每种不同类型的设备编写不同的代码。

C 语言的 I/O 系统设计是为了将这些细节隐藏在你背后。它还提供了许多优秀的功能，如格式化、缓冲和设备独立性。缓冲 I/O 系统对于大多数一般应用程序来说非常有效。

## 编程问题

1.  看看当你在`printf`语句中放入过多或过少的参数时会发生什么。如果你放入了错误的类型（例如，`double`而不是`int`）会怎样？

1.  编写一个程序，要求用户输入摄氏温度并将其转换为华氏温度。

1.  编写一个程序，计算文件中单词的数量。请确保你对“单词”的定义进行文档说明，因为不同的人对“单词”的理解可能与你不同。

1.  编写一个程序，逐行比较两个文件，并输出不同的行。
