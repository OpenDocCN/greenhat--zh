# 第十五章：命令行参数和原始 I/O

![](img/chapterart.png)

在本章中，我们探讨了命令行参数如何允许操作系统在程序被调用时将信息传递给程序。我们还将了解一个与操作系统紧密相关的功能：原始输入/输出（I/O）系统。这个系统使我们能够精确控制程序如何执行 I/O 操作。如果正确使用，它可以成为程序的一项巨大资产。

我们将使用原始 I/O 系统执行高速文件复制。这个程序还将使用命令行参数来指定源文件和目标文件，这样我们就不需要将它们硬编码到程序中了。

## 命令行参数

操作系统允许用户在程序运行时通过命令行选项向程序提供多个参数：

```
$ ./`prog` `argument1` `argument2` `argument3`
```

C 通过两个参数 `argc` 和 `argv` 将这些参数传递给 `main`：

```
int main(const int argc, const char* const argv[])
```

第一个参数 `argc` 包含参数的数量。由于历史原因，它是一个整数，而不是无符号整数。第二个参数 `argv` 是一个字符串数组，表示实际的参数。

如果你运行一个像这样的程序：

```
./`prog` `first` `second third`
```

`argv` 和 `argc` 参数将包含以下内容：

```
argc	4
argv[0]    ./`prog`
argv[1]    `first`
argv[2]    `second`
argv[3]    `third`
```

第一个参数是程序的名称。下一个参数是命令行上的 `first` 参数，依此类推。

示例 15-1 包含了一个简短的程序，旨在打印命令行参数。

**echo.c**

```
/**
 * Echo the command line arguments.
 */
#include <stdio.h>

int main(const int argc, const char* argv[])
{
    for (int i = 0; i < argc; ++i) {
        printf("argv[%d] = %s\n", i, argv[i]);
    }
    return (0);
}
```

示例 15-1：打印命令行参数

你不一定需要将参数数量命名为 `argc`，将参数向量命名为 `argv`，也不需要声明 `argv` 和 `argc const`，但这样做是惯例。

## 原始 I/O

C 程序员可以使用的两种主要文件 I/O 系统是*缓冲 I/O*和*无缓冲 I/O*。我们在第十四章讨论的标准 I/O 系统（`printf`）使用了缓冲区。在本章中，我们将使用无缓冲 I/O。为了展示这两者之间的区别，我们来考虑一个例子。假设你想整理衣橱，并且有 500 根旧电源线需要丢弃。你可以这样做：

1.  拿起一根电源线。

1.  走到户外的垃圾桶旁。

1.  丢掉它。

1.  重复 500 次。

这种方法就像使用无缓冲 I/O 丢弃电源线一样，*吞吐量*（你完成工作的速度）非常低。

让我们添加一个缓冲区——在这个例子中，就是一个垃圾袋。现在程序的步骤如下：

1.  把电源线放进袋子里。

1.  一直往袋子里放电源线，直到它满了。（它能装下 100 根电源线。）

1.  走到户外的垃圾桶旁。

1.  丢掉袋子。

1.  重复五次。

缓冲会使重复的过程更高效，那么什么时候你会选择使用无缓冲 I/O 呢？你会在某些情况下使用它，这些情况下单独丢弃每个物品会更高效。假设你要丢掉五台冰箱。你不会把五台冰箱放进垃圾袋然后一起丢掉。相反，你会把每一台冰箱单独丢掉。

### 使用原始 I/O

如果我们想复制一个文件，可以使用缓冲 I/O 系统来实现，但那样的话，我们需要让缓冲 I/O 系统选择缓冲区的大小。相反，我们希望设置自己的缓冲区大小。在这种情况下，我们知道 1,024 字节的大小是适用于我们设备的最佳大小，因此我们创建了在 Listing 15-2 中显示的程序，使用原始 I/O 来复制文件，缓冲区大小为 1,024 字节。

**copy.c**

```
/**
 * Copy one file to another.
 *
 * Usage:
 *     copy <from> <to>
 */

#include <stdio.h>
#include <stdbool.h>
#include <stdlib.h>

#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

#ifndef O_BINARY
#define O_BINARY 0      // Define O_BINARY if not defined.
#endif // O_BINARY

int main(int argc, char* argv[])
{
  1 if (argc != 3) {
        fprintf(stderr, "Usage is %s <infile> <outfile>\n", argv[0]);
        exit(8);
    }

    // The fd of the input file
  2 int inFd = open(argv[1], O_RDONLY|O_BINARY);

  3 if (inFd < 0) {
        fprintf(stderr, "ERROR: Could not open %s for input\n", argv[1]);
        exit(8);
    }

    // The fd of the output file
  4 int outFd = open(argv[2], O_WRONLY|O_CREAT|O_BINARY, 0666);
    if (outFd < 0) {
        fprintf(stderr, "ERROR: Could not open %s for writing\n", argv[2]);
        exit(8);
    }

    while (true)
    {
        char buffer[1024];      // Buffer to read and write
        size_t readSize;        // Size of the last read

      5 readSize = read(inFd, buffer, sizeof(buffer));
      6 if (readSize < 0) {
            fprintf(stderr, "ERROR: Read error for file %s\n", argv[1]);
            exit(8);
        }
      7 if (readSize == 0)
            break;

      8 if (write(outFd, buffer, readSize) != readSize) {
            fprintf(stderr, "ERROR: Write error for %s\n", argv[2]);
            exit(8);
        }
    }
  9 close(inFd);
    close(outFd);
    return (0);
}
```

Listing 15-2：使用原始 I/O 复制一个文件到另一个文件的程序

要使用 Listing 15-2 中的程序，我们必须指定一个输入文件和一个输出文件：

```
$ ./copy `input-file output-file`
```

程序首先检查是否提供了正确数量的参数 1。接着，它打开输入文件 2。`open` 函数的一般形式是 `file-descriptor` `= open(``filename``,` `flags``)`。标志指示文件如何打开。`O_RDONLY` 标志表示文件以只读模式打开，`O_BINARY` 标志表示文件是二进制的。`O_BINARY` 标志是一个有趣的标志（我将在下一节中解释）。

`open` 命令返回一个称为 *文件描述符* 的数字。如果发生错误，它返回一个负数，这意味着程序的下一步是检查错误 3。

然后我们使用 `O_WRONLY`（仅写模式）和 `O_CREAT`（如果需要则创建文件）标志打开输出文件 4。

额外的 `0666` 参数表示如果文件被创建，它会处于保护模式。这是一个八进制数字，每一位代表一个保护用户集，每一位代表一种保护类型：

1.  4 读取

1.  2 写入

1.  1 执行

数字的顺序如下：*<user>*、*<group>*、*<other>*。`0666` 参数告诉系统创建文件，使得用户可以读取和写入它（`6`），使得与用户同组的账户可以读写（`6`），并且其他任何人也拥有相同的读写权限（`6`）。

一旦文件被打开，我们就进行复制 5。`read` 函数的一般形式是：

```
`bytes_read` `= read(``fd``,` `buffer``,` `size``);`
```

其中 `fd` 是文件描述符，`buffer` 是接收数据的缓冲区，`size` 是读取的最大字符数。该函数返回读取的字节数（`bytes read`），`0` 表示文件结束（EOF），或者返回负数表示发生错误。

读取后，我们检查是否有错误 6。然后我们检查是否已到达文件末尾 7。如果是，我们就完成了数据传输。

在这一点上，我们肯定已经有一些数据了，因此我们开始写入 8。`write` 函数的一般形式是：

```
`bytes_written` `= write(``fd``,` `buffer``,` `size``);`
```

其中 `fd` 是文件描述符，`buffer` 是包含数据的缓冲区，`size` 是要写入的字符数。该函数返回写入的字节数或返回负数表示发生错误。一旦写入完成，我们关闭文件描述符 9。

### 使用二进制模式

不幸的是，文本文件在操作系统之间不可移植，因为不同的操作系统使用不同的字符来表示行结束。C 最初是为 Unix 编写的，而 Unix 又启发了 Linux。这两个操作系统都使用换行符（字符编号`0x0a`）作为行结束符。

假设你打开一个没有`O_BINARY`标志的文本文件，并且想向其中写入数据。如果你使用以下方式将字符串写入文件：

```
// Bad style; 3 should be a named constant.
write(fd, "Hi\n", 3);
```

在 Linux 上，你将得到一个包含三个字符的文件：

```
48  69  0a
 H   i  \n
```

其他操作系统必须将行尾序列转换为其本地的行结束符。表 14-1 列出了各种行结束符。

表 15-1：文件行结束符

| **操作系统** | **行结束符** | **字符** | **转换** |
| --- | --- | --- | --- |
| Linux | 换行符 | `\n` | 无 |
| macOS | 回车符 | `\r` | 在输出时将`\n`替换为`\r` |
| Windows | 回车符、换行符 | `\r\n` | 在每个`\n`前插入`\r` |

如果你在 Windows 上运行 C 程序并执行以下操作：

```
// Bad style; 3 should be a named constant.
write(fd, "Hi\n", 3);
```

这与之前的代码相同，写入了四个字符：

```
48  69  0d  0a
 H   i  \r  \n
```

然而，有时你会在写入二进制文件时，希望字节`0a`以`0a`的形式原样写入，而不做任何转换。在 Linux 上，这很简单，因为 Linux 永远不会进行转换。然而，其他操作系统会进行转换，因此它们添加了一个新的`O_BINARY`标志，告诉库正在使用二进制文件，并跳过文件转换。

Linux 没有`O_BINARY`标志，因为它不区分二进制文件和文本文件。实际上，你可以拥有一个半二进制/半文本的文件。（我不知道为什么你会想这么做，但 Linux 会允许你这么做。）

我在清单 15-2 中包含了`O_BINARY`标志，因为我希望复制程序具有可移植性。我们需要在使用 Apple 和 Microsoft 系统时提供`O_BINARY`模式，但如果我们在 Linux 系统上编译程序，则`O_BINARY`未定义。

因此，解决方法是如果操作系统的头文件中没有定义该标志，则自己定义它：

```
#ifndef O_BINARY
#define O_BINARY 0      // Define O_BINARY if not defined.
#endif // O_BINARY
```

如果操作系统已经定义了`O_BINARY`，则`#define`将不会被编译。如果我们使用的是没有`O_BINARY`的类 Linux 操作系统，`#define O_BINARY 0`将被编译，并且`O_BINARY`将被赋值为`0`，这样就什么都不做——而在 Linux 上，正是“不做任何事情”是我们需要的。

## ioctl

除了读取和写入之外，原始 I/O 系统还提供了一个名为`ioctl`的函数，用于执行 I/O 控制。它的一般形式是：

```
`result` `= ioctl(``fd``,` `request``,` `parameter``);`
```

其中`fd`是文件描述符，`request`是设备特定的控制请求，`parameter`是请求的参数。对于大多数请求，如果请求成功，函数返回`0`，否则返回非零值（某些`ioctl`调用返回不同的值）。

你可以使用`ioctl`来弹出可移动媒体、倒带或快进磁带驱动器、设置串口设备的速度和其他参数，以及设置网络设备的地址信息。由于`ioctl`规范是开放式的，许多功能已经被压缩到这个接口中。

## 总结

原始 I/O 系统提供了对 I/O 操作的最佳控制。操作系统的编辑或干预最小，但这种控制是有代价的。缓冲 I/O 系统有助于限制你的错误，而原始 I/O 系统则没有。不过，如果你知道自己在做什么，它可以成为一个巨大的资产。

## 编程问题

1.  编写一个程序，接受一个参数：运行该程序的人的名字。然后打印`Hello` `<name>`。例如：

    ```
    ./hello Fred
    Hello Fred

    ./hello
    Hello stranger
    ```

1.  编写一个程序，扫描参数列表，如果`-d`是一个参数，则打印`调试模式`。如果`-d`缺失，则打印`发布模式`。还可以添加其他选项。

1.  测量列表 15-2 中的复制程序复制大文件所花费的时间。现在将缓冲区大小更改为 1。查看程序运行速度。将缓冲区大小更改为 16384。查看程序运行速度。尝试 17000。注意：几乎每个磁盘都是以 512 字节块进行读写的。这个事实如何解释你所看到的时间？

1.  研究`getopt`函数并使用它解析你为问题 1 发明的命令行参数。
