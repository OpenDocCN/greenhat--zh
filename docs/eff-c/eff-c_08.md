

# 第八章：8 输入/输出



![](img/opener.jpg)

本章将教你如何执行输入/输出（I/O）操作，从终端或文件系统读取数据或写入数据。信息可以通过命令行参数或环境进入程序，并通过返回状态退出程序。然而，大多数信息通常是通过 I/O 操作进入或退出程序的。我们将讨论使用 C 标准流和 POSIX 文件描述符的技术。我们将首先讨论 C 标准文本和二进制流。然后，我们将介绍使用 C 标准库和 POSIX 函数打开和关闭文件的不同方法。

接下来，我们将讨论字符和行的读取与写入、格式化文本的读取与写入，以及从二进制流中读取和写入。我们还将涵盖流缓冲、流定向和文件定位。

还有许多其他设备和 I/O 接口（例如ioctl）可用，但它们超出了本书的范围。

## 标准 I/O 流

C 提供了与存储在受支持的结构化存储设备和终端上的文件进行通信的流。*流*是与文件和设备（如套接字、键盘、通用串行总线（USB）端口和打印机）通信的统一抽象，这些设备或文件消耗或生成顺序数据。

C 使用不透明的FILE数据类型来表示流。FILE对象保存与关联文件连接的内部状态信息，包括文件位置指示器、缓冲区信息、错误指示器和文件结束指示器。你不应该自己分配FILE对象。C 标准库函数操作的是类型为FILE *（即指向FILE类型的指针）的对象。因此，流通常被称为*文件指针*。

C 提供了一个广泛的应用程序接口（API），该接口可以在<stdio.h>中找到，用于操作流；我们将在本章稍后探讨此 API。然而，由于这些 I/O 函数需要与许多平台上各种各样的设备和文件系统协同工作，因此它们具有高度的抽象性，这使得它们不适用于超出最简单应用的场景。

例如，C 标准没有目录的概念，因为它必须能够与非层次化的文件系统兼容。C 标准对文件系统特定细节的引用较少，如文件权限或锁定。然而，函数规范通常指出，某些行为会在“底层系统支持的程度上”发生，这意味着只有在你的实现支持这些行为时，它们才会发生。

因此，你通常需要使用 POSIX、Windows 以及其他平台提供的较不便携的 API 来执行实际应用中的 I/O 操作。通常，应用程序会定义自己的 API，这些 API 又依赖于平台特定的 API 来提供安全、可靠且便携的 I/O 操作。

### 错误和文件结束指示符

如前所述，FILE对象保存与关联文件连接的内部状态信息，包括一个错误指示符，用于记录是否发生了读写错误，以及一个文件结束指示符，用于记录是否已到达文件末尾。文件打开时，流的错误指示符和文件结束指示符会被清除。以下 C 标准库函数会在发生错误时设置流的错误指示符：字节输入函数（getc、fgetc 和 getchar）、字节输出函数（putc、fputc 和 putchar）、fflush、fseek 和 fsetpos。输入函数，如<sup class="SANS_TheSansMonoCd_W5Regular_11">fgetc 和 getchar，如果流已到达文件末尾，也会设置流的文件结束指示符。某些函数，如<sup class="SANS_TheSansMonoCd_W5Regular_11">rewind 和 freopen，会清除流的错误指示符，而函数如<sup class="SANS_TheSansMonoCd_W5Regular_11">rewind、freopen、ungetc、fseek 和 fsetpos，会清除流的文件结束指示符。宽字符 I/O 函数的行为类似。

这些指示符可以显式测试和清除：

+   ferror 函数测试指定流的错误指示符，并且仅在指定流的错误指示符被设置时返回非零值。

+   feof 函数测试指定流的文件结尾指示符，并且仅在指定流的文件结尾指示符被设置时返回非零值。

+   clearerr 函数清除指定流的文件结尾和错误指示符。

以下简短程序展示了这些函数与两个指示符之间的交互：

```
#include <stdio.h>
#include <assert.h>

int main() {
  FILE* tmp = tmpfile();
  fputs("Effective C\n", tmp);
  rewind(tmp);
  for (int c; (c = fgetc(tmp)) != EOF; putchar(c)) {}
  printf("%s", "End-of-file indicator ");
  puts(feof(tmp) ? "set" : "clear");
  printf("%s", "Error indicator ");
  puts(ferror(tmp) ? "set" : "clear");
  clearerr(tmp); // clear both indicators
  printf("%s", "End-of-file indicator ");
  puts(feof(tmp) ? "set" : "clear");
}
```

该程序在 stdout 上生成以下输出：

```
Effective C
End-of-file indicator set
Error indicator clear
End-of-file indicator clear
```

循环通过文件结尾终止，之后设置文件结尾指示符。这两个指示符会通过调用 clearerr 函数被清除。

### 流缓冲

*缓冲* 是将数据暂时存储在内存中的过程，数据在进程和设备或文件之间传递。缓冲提高了 I/O 操作的吞吐量，因为每个 I/O 操作通常会有较高的延迟。类似地，当程序请求写入块设备（如磁盘）时，驱动程序可以将数据缓存到内存中，直到累积足够的数据形成一个或多个设备块，此时会将数据一次性写入磁盘，从而提高吞吐量。这种策略称为*刷新*输出缓冲区。

一个流可以处于以下三种状态之一：

**无缓冲** 字符旨在尽可能快地从源或到达目的地。通常多个程序可能并发访问的数据流，最好使用无缓冲模式。用于错误报告或日志记录的流也可能是无缓冲的。

**全缓冲** 字符被设计成在缓冲区填满时作为一个块传输到主机环境或从主机环境传输。用于文件 I/O 的流通常采用全缓冲方式，以优化吞吐量。

**行缓冲** 字符在遇到换行符时，旨在作为一个块传输到主机环境或从主机环境传输。连接到交互设备（如终端）的流在打开时通常采用行缓冲模式。

在接下来的章节中，我们将介绍预定义流并描述它们是如何进行缓冲的。

### 预定义流

一个 C 程序在启动时会打开并可用三种 *预定义文本流*。这些预定义流在 <stdio.h> 中声明：

```
extern FILE * stdin;  // standard input stream
extern FILE * stdout; // standard output stream
extern FILE * stderr; // standard error stream
```

*标准输出流*（stdout）是程序的传统输出目标。这个流通常与启动程序的终端相关联，但可以被重定向到文件或其他流。在 Linux 或 Unix 的 shell 中，你可以输入以下命令：

```
$ **echo fred**
fred
$ **echo fred > tempfile**
$ **cat tempfile**
fred
```

在这里，echo命令的输出被重定向到*tempfile*。

*标准输入流*（stdin）是程序的传统输入源。默认情况下，stdin与键盘相关联，但也可以被重定向为来自文件的输入，例如，使用以下命令：

```
$ **echo "one two three four five six seven" > tempfile**
$ **wc < tempfile**
1 7 34
```

文件*tempfile*的内容被重定向到stdin流，传递给wc命令，输出*tempfile*的换行符（1）、单词数（7）和字节数（34）。stdin和stdout流会在仅当流不指向交互式设备时，才完全缓冲。

*标准错误流*（stderr）用于写入诊断输出。stderr流不会完全缓冲，以便尽快查看错误信息。

图 8-1 显示了预定义的流stdin、stdout和stderr，它们附加在用户终端的键盘和显示器上。

![](img/f08001.jpg)

图 8-1：附加到 I/O 通信通道的标准流

一个程序的输出流可以通过使用 POSIX 管道被重定向到另一个应用程序的输入流：

```
$ **echo "Hello Robert" | sed "s/Hello/Hi/" | sed "s/Robert/robot/"**
Hi robot
```

流编辑器sed是一个用于过滤和转换文本的 Unix 工具。竖线字符（|）在许多平台上可用于链式命令。

### 流方向

每个流都有一个*方向*，它表示该流是包含窄字符还是宽字符。在一个流与外部文件关联之后，但在进行任何操作之前，该流没有方向。一旦应用了宽字符 I/O 函数到一个没有方向的流，该流就变成了*宽字符导向流*。类似地，一旦应用了字节 I/O 函数到一个没有方向的流，该流就变成了*字节导向流*。可以作为 char 类型对象表示的多字节字符序列或窄字符（根据 C 标准，这些字符需要占用 1 个字节）可以写入字节导向流中。

你可以通过使用 fwide 函数或者通过关闭并重新打开文件来重置流的方向。如果对宽字符流应用字节 I/O 函数，或者对字节导向流应用宽字符 I/O 函数，将导致未定义的行为。永远不要将窄字符数据、宽字符数据和二进制数据混合存储在同一文件中。

所有三种预定义流（stderr，stdin 和 stdout）在程序启动时都是无方向的。

### 文本流与二进制流

C 标准支持文本流和二进制流。*文本流* 是由字符组成的有序序列，字符按行排列，每行由零个或多个字符及一个终止换行符序列组成。在类 Unix 系统中，你可以使用换行符（\n）表示单一的换行。大多数微软 Windows 程序使用回车符（\r）后跟换行符（\n）。

不同的换行符约定可能导致在不同约定的系统之间传输的文本文件显示或解析不正确，尽管在现代系统中这种情况已经变得不常见，因为这些系统现在能够理解外部的换行符约定。

*二进制流* 是一种有序的任意二进制数据序列。从二进制流读取的数据将与之前写入该流的数据相同，在相同的实现下也是如此。在非 POSIX 系统中，流的末尾可能会附加由实现定义的数量的空字节。

二进制流总是比文本流更强大、更可预测。然而，读取或写入一个普通的文本文件，且能与其他文本导向的程序兼容，最简单的方法是通过文本流。

## 打开和创建文件

当你打开或创建一个文件时，它会与一个流关联。fopen 和 POSIX open 函数用于打开或创建文件。

### fopen

fopen 函数打开一个文件，该文件的名称由字符串给出并由 filename 指向，然后将一个流与之关联：

```
FILE *fopen(
  const char * restrict filename,
  const char * restrict mode
);
```

mode 参数指向表 8-1 中显示的字符串之一，用于确定如何打开文件。

表 8-1： 有效的文件模式字符串

| 模式字符串 | 描述 |
| --- | --- |
| r | 打开现有文本文件以供读取 |
| w | 截断为零长度或创建文本文件以供写入 |
| a | 附加、打开或创建文本文件以在文件末尾写入 |
| rb | 打开现有二进制文件以供读取 |
| wb | 截断文件为零长度或创建二进制文件以供写入 |
| ab | 附加、打开或创建二进制文件以在文件末尾写入 |
| r+ | 打开现有文本文件以供读写 |
| w+ | 截断为零长度或创建文本文件以供读写 |
| a+ | 附加、打开或创建文本文件以供更新，在文件当前末尾写入 |
| r+b 或 rb+ | 打开现有二进制文件以供读写 |
| w+b 或 wb+ | 截断为零长度或创建用于读取和写入的二进制文件 |
| a+b 或 ab+ | 追加，打开或创建用于更新的二进制文件，在当前文件末尾写入 |

以读取模式打开文件（通过将 r 作为 mode 参数的第一个字符传递）会失败，如果文件不存在或无法读取。

以追加模式打开文件（通过将 a 作为 mode 参数的第一个字符传递）会导致所有后续写入文件的操作发生在当前文件末尾，直到缓冲区刷新或实际写入时，无论是否有对 fseek、fsetpos 或 rewind 函数的调用。将当前文件末尾的指针按写入的数据量递增是原子性的，只要文件也以追加模式打开，并且其他线程在写入同一文件时不干扰。如果实现无法原子地递增当前文件末尾，它将失败，而不是进行非原子性写入。在某些实现中，以追加模式打开二进制文件（通过将 b 作为 mode 参数的第二个或第三个字符传递）可能会由于空字符填充而将文件位置指示器设置在最后一个数据写入之后。

你可以通过将 + 作为 mode 参数的第二个或第三个字符传递来以更新模式打开文件，从而可以对相关流执行读取和写入操作。在某些实现中，以更新模式打开（或创建）文本文件可能会改为打开（或创建）二进制流。在 POSIX 系统中，文本流和二进制流的行为完全相同。

C11 标准增加了*独占模式*，用于读取和写入二进制文件和文本文件，如 表 8-2 所示。

表 8-2： C11 添加的有效文件模式字符串

| 模式字符串 | 描述 |
| --- | --- |
| < sAmp class="SANS_TheSansMonoCd_W5Regular_11">wx | < sAmp class="SANS_Futura_Std_Book_11">创建独占文本文件用于写入 |
| < sAmp class="SANS_TheSansMonoCd_W5Regular_11">wbx | < sAmp class="SANS_Futura_Std_Book_11">创建独占二进制文件用于写入 |
| < sAmp class="SANS_TheSansMonoCd_W5Regular_11">w+x | < sAmp class="SANS_Futura_Std_Book_11">创建独占文本文件用于读写 |
| < sAmp class="SANS_TheSansMonoCd_W5Regular_11">w+bx < sAmp class="SANS_Futura_Std_Book_11">或 < sAmp class="SANS_TheSansMonoCd_W5Regular_11">wb+x | < sAmp class="SANS_Futura_Std_Book_11">创建独占二进制文件用于读写 |

以独占模式打开文件（通过在< sAmp class="SANS_TheSansMonoCd_W5Regular_11">mode 参数的最后一个字符传递 x）如果文件已存在或无法创建则会失败。文件存在性的检查和如果文件不存在则创建文件的操作是原子性的，涉及到其他线程和并发程序执行。如果实现无法原子地执行文件存在性检查和文件创建，它会失败，而不是进行非原子检查和创建。

最后，请确保永远不要复制一个 < sAmp class="SANS_TheSansMonoCd_W5Regular_11">FILE 对象。例如，下面的程序可能会失败，因为在调用 < sAmp class="SANS_TheSansMonoCd_W5Regular_11">fputs 时，使用了 < sAmp class="SANS_TheSansMonoCd_W5Regular_11">stdout 的按值复制：

```
#include <stdio.h>
#include <stdlib.h>

int main() {
  FILE my_stdout = *stdout;
  if (fputs("Hello, World!\n", &my_stdout) == EOF) {
    return EXIT_FAILURE;
  }
  return EXIT_SUCCESS;
}
```

该程序具有未定义的行为，通常会在运行时崩溃。

### < sAmp class="SANS_Futura_Std_Bold_Condensed_Oblique_BI_11">open

在 POSIX 系统中，< sAmp class="SANS_TheSansMonoCd_W5Regular_11">open 函数（IEEE Std 1003.1:2018）建立了文件 < sAmp class="SANS_TheSansMonoCd_W5Regular_11">path 与一个叫做 *文件描述符* 的值之间的连接：

```
int open(const char *path, int oflag, ...);
```

*文件描述符* 是一个非负整数，指向表示文件的结构（称为 *打开文件描述*）。由< sAmp class="SANS_TheSansMonoCd_W5Regular_11">open 函数返回的文件描述符是未使用的最低编号文件描述符，并且是唯一的，属于调用该函数的进程。文件描述符被其他 I/O 函数用来引用该文件。< sAmp class="SANS_TheSansMonoCd_W5Regular_11">open 函数将文件偏移量设置为标记文件内当前的位置，从文件的开始位置开始。对于一个流的文件描述符，这个文件偏移量与流的文件位置指示器是分开的。

oflag 参数的值设置了打开文件描述符的 *文件访问模式*，指定文件是以读取、写入还是两者同时进行打开。oflag 的值是通过按位或操作组合文件访问模式和任何访问标志。应用程序必须在 oflag 的值中指定以下文件访问模式之一：

O_EXEC  仅用于执行（非目录文件）

O_RDONLY 仅用于读取

O_RDWR 同时用于读取和写入

O_SEARCH 仅用于搜索的目录打开

O_WRONLY 仅用于写入

oflag 参数的值还设置了 *文件状态标志*，这些标志控制 open 函数的行为，并影响文件操作的执行方式。这些标志包括以下内容：

O_APPEND 在每次写入之前，将文件偏移量设置为文件末尾

O_TRUNC 将文件长度截断为 0

O_CREAT 创建文件

O_EXCL 如果同时设置了 O_CREAT 并且文件已存在，则导致打开失败

open 函数接受可变数量的参数。紧随 oflag 参数后的值指定文件模式位（当创建新文件时的文件权限），类型为 mode_t。

清单 8-1 展示了一个使用 open 函数打开文件进行写入的示例。

```
#include <err.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
  int fd;
  int flags = O_WRONLY | O_CREAT | O_TRUNC;
❶ mode_t mode = S_IRUSR | S_IWUSR | S_IRGRP | S_IROTH;
  const char *pathname = "/tmp/file";
  if ((fd = open(pathname, flags, mode) ❷) == -1) {
    err(EXIT_FAILURE, "Can't open %s", pathname);
  }
  // `--snip--`
}
```

清单 8-1：以写入模式打开文件

调用 open ❷ 时需要多个参数，包括文件的路径名、oflag 和模式。我们创建了一个 mode 标志 ❶，它是以下访问权限模式位的按位或组合：

S_IRUSR 文件所有者的读取权限位

S_IWUSR 文件所有者的写权限位

S_IRGRP 文件组所有者的读权限位

S_IROTH 其他用户的读权限位

open 函数仅在创建文件时设置这些权限。如果文件已存在，它的当前权限将保持不变。文件访问模式为 O_WRONLY，表示文件仅用于写入。O_CREAT 文件状态标志通知 open 创建该文件；O_TRUNC 文件状态标志通知 open 如果文件存在且成功打开，应丢弃文件的先前内容。

如果文件成功打开，open 函数返回一个非负整数，表示文件描述符。否则，open 返回 −1，并将 errno 设置为指示错误的值。Listing 8-1 检查是否返回 −1，如果发生错误，则将诊断消息写入预定义的 stderr 流，并退出。

除了 open，POSIX 还提供了其他有用的函数来处理文件描述符，例如 fileno 函数用于获取与现有文件指针关联的文件描述符，fdopen 函数用于通过现有的文件描述符创建一个新的流文件指针。通过文件描述符提供的 POSIX API 允许访问 POSIX 文件系统的功能，这些功能通常不会通过文件指针接口暴露出来，例如目录（posix_getdents，fdopendir，readdir）、文件权限（fchmod）和文件锁（fcntl）。

## 关闭文件

打开文件会分配资源。如果你不断地打开文件却没有关闭它们，最终你的进程将用尽可用的文件描述符或句柄，尝试打开更多文件时将会失败。因此，使用完文件后关闭文件是非常重要的。

### fclose

C 标准库中的fclose函数用于关闭文件：

```
int fclose(FILE *stream);
```

流中任何未写入的缓冲数据将传递给主机环境，写入文件中。任何未读取的缓冲数据将被丢弃。

fclose函数可能会失败。例如，当<fclose写入剩余的缓冲输出时，可能会因为磁盘已满而返回错误。即使你知道缓冲区已空，如果使用网络文件系统（NFS）协议关闭文件时，仍然可能发生错误。尽管可能会失败，但通常无法恢复，因此程序员通常忽略<fclose返回的错误。关闭文件失败时，一种常见的做法是中止进程或截断文件，使其内容在下次读取时仍然有意义。

为了确保你的代码稳健，务必检查错误。文件 I/O 可能由于各种原因失败。如果检测到错误，fclose函数会返回EOF：

```
if (fclose(fp) == EOF) {
  err(EXIT_FAILURE, "Failed to close file\n");
}
```

你需要显式调用fflush或fclose，以刷新程序写入的任何缓冲流，而不是让exit（或从main返回）来刷新它，以执行错误检查。

在相关文件被关闭后，指向FILE对象的指针值是未定义的。是否存在一个零长度的文件（即没有写入任何数据的输出流）由实现定义。

你可以在同一程序或另一个程序中重新打开已关闭的文件，并且可以恢复或修改其内容。如果初始调用的main函数返回或调用了exit函数，所有打开的文件会在程序终止前关闭（并且所有输出流会被刷新）。

程序终止的其他路径，例如调用abort函数，可能无法正确关闭所有文件，这意味着尚未写入磁盘的缓冲数据可能会丢失。

### close

在 POSIX 系统上，你可以使用close函数来释放由fd指定的文件描述符：

```
int close(int fd);
```

如果在调用 close 时，从文件系统读取或写入数据发生 I/O 错误，可能会返回 −1，并且 errno 会被设置为错误原因。如果返回错误，则 fd 的状态是未定义的，这意味着你无法再读取或写入数据到该描述符，也不能再次尝试关闭它——这实际上导致文件描述符泄漏。为了解决这个问题，posix_close 函数已被添加到《开放组基础规范》第 8 版中。

一旦文件成功关闭，文件描述符将不再存在，因为与其对应的整数不再指向任何文件。当拥有该文件描述符的进程终止时，文件也会被关闭。

除非在极少数情况下，使用 fopen 打开文件的应用程序会使用 fclose 来关闭文件；使用 open 打开文件的应用程序会使用 close 来关闭文件（除非它将描述符传递给了 fdopen，在这种情况下，它必须通过调用 fclose 来关闭文件）。

## 读取和写入字符与行

C 标准定义了用于读取和写入特定字符或行的函数。

大多数字节流函数都有相应的版本，可以使用宽字符（wchar_t）或宽字符字符串来替代窄字符（char）或字符串（参见 表 8-3）。字节流函数在头文件 <stdio.h> 中声明，而宽字符流函数在 <wchar.h> 中声明。宽字符函数在相同的流（例如 stdout）上操作。

表 8-3： 窄字符与宽字符 I/O 函数

| char | wchar_t | 描述 |
| --- | --- | --- |
| fgetc | fgetwc | 从流中读取一个字符。 |
| getc | getwc | 从流中读取一个字符。 |
| getchar | getwchar | 从 stdin读取一个字符。 |
| fgets | fgetws | 从流中读取一行。 |
| fputc | fputwc | 将字符写入流中。 |
| putc | putwc | 将字符写入流中。 |
| fputs | fputws | 将字符串写入流中。 |
| putchar | putwchar | 将字符写入 stdout。 |
| puts | N/A | 将字符串写入 stdout。 |
| ungetc | ungetwc | 将字符返回到流中。 |
| scanf | wscanf | 从 stdin读取格式化的字符输入。 |
| fscanf | fwscanf | 从流中读取格式化的字符输入。 |
| sscanf | swscanf | 从缓冲区中读取格式化的字符输入。 |
| printf | wprintf | 将格式化字符输出打印到 stdout。 |
| fprintf | fwprintf | 将格式化字符输出打印到流中。 |
| sprintf | swprintf | 将格式化字符输出打印到缓冲区。 |
| snprintf | N/A | 这与 sprintf 相同，但带有截断功能。 swprintf 函数也接受一个长度参数，但其处理方式与 snprintf 不同。 |

在本章中，我们将只讨论字节流函数。如果可能的话，您可能想完全避免使用宽字符函数变体，专门使用 UTF-8 字符编码，因为这些函数不太容易导致程序员错误和安全漏洞。

fputc 函数将字符 c 转换为 unsigned char 类型，并将其写入 stream：

```
int fputc(int c, FILE *stream);
```

如果发生写入错误，它返回 EOF；否则，它返回已写入的字符。

putc 函数与 fputc 类似，唯一的区别是大多数库将其实现为宏：

```
int putc(int c, FILE *stream);
```

如果 putc 被实现为宏，它可能会多次评估其 stream 参数。通常使用 fputc 更加安全。更多信息请参见 CERT C 规则 FIO41-C，“不要使用具有副作用的流参数调用 getc()、putc()、getwc() 或 putwc()”。

putchar函数等同于putc函数，不同之处在于它使用stdout作为流参数的值。

fputs函数将字符串s写入流stream：

```
int fputs(const char * restrict s, FILE * restrict stream);
```

该函数不会写入字符串s中的空字符，也不会写入换行符，而只输出字符串中的字符。如果发生写入错误，fputs将返回EOF。否则，它将返回一个非负值。例如，以下语句输出文本I am Groot，后跟一个换行符：

```
fputs("I ", stdout);
fputs("am ", stdout);
fputs("Groot\n", stdout);
```

puts函数将字符串s写入流stdout，后跟一个换行符：

```
int puts(const char *s);
```

puts函数是打印简单消息时最方便的函数，因为它只需要一个参数。以下是一个示例：

```
puts("This is a message.");
```

fgetc函数从流中读取下一个字符，将其作为unsigned char类型，并返回其值，转换为int类型：

```
int fgetc(FILE *stream);
```

如果发生文件结束或读取错误，函数将返回EOF。

getc函数等同于fgetc，不同之处在于如果它作为宏实现，可能会多次评估其流参数。因此，该参数不应为带有副作用的表达式。类似于fputc函数，使用fgetc通常更安全，应优先使用fgetc而非getc。

getchar函数等同于getc函数，不同之处在于它使用stdout作为流参数的值。

你可能记得，gets 函数从 stdin 中读取字符，并将它们写入字符数组，直到遇到换行符或 EOF。gets 函数本质上是不安全的。它在 C99 中被弃用，并在 C11 中被移除，*永远不应该使用*。如果你需要从 stdin 读取字符串，考虑改用 fgets 函数。fgets 函数最多从流中读取 n 个字符减去 1，读取到指向的字符数组 s 中：

```
char *fgets(char * restrict s, int n, FILE * restrict stream);
```

在遇到（保留的）换行符或 EOF 后，不会再读取其他字符。读取的最后一个字符后会立即写入一个空字符。

## 流刷新

如本章前面所述，流可以是完全或部分缓冲的，这意味着你认为已经写入的数据可能尚未传递到主机环境中。当程序突然终止时，这可能会成为一个问题。fflush 函数将任何未写入的数据从指定流传递到主机环境，以便写入文件：

```
int fflush(FILE *stream);
```

如果流的最后一个操作是输入操作，则行为未定义。如果流是空指针，fflush 函数将在所有流上执行此刷新操作。如果这不是你的意图，请确保在调用 fflush 时，文件指针不是空指针。

## 设置文件中的位置

随机访问文件（例如磁盘文件，但不包括终端）维护一个与流关联的文件位置指示符。*文件位置指示符* 描述了流当前在文件中读取或写入的位置。

当你打开一个文件时，指示器会定位到文件的起始位置（除非你以追加模式打开它）。你可以将指示器放置在任何你想读取或写入文件部分的位置。ftell 函数获取当前文件位置指示器的值，而 fseek 函数则设置文件位置指示器。这些函数使用 long int 类型表示文件中的偏移量（位置），因此它们的偏移量限制在可以表示为 long int 的范围内。Listing 8-2 演示了 ftell 和 fseek 函数的使用。

```
#include <err.h>
#include <stdio.h>
#include <stdlib.h>

long int get_file_size(FILE *fp) {
  if (fseek(fp, 0, SEEK_END) != 0) {
    err(EXIT_FAILURE, "Seek to end-of-file failed");
  }
  long int fpi = ftell(fp);
  if (fpi == -1L) {
    err(EXIT_FAILURE, "ftell failed");
  }
  return fpi;
}

int main() {
  FILE *fp = fopen("fred.txt", "rb");
  if (fp  == nullptr) {
    err(EXIT_FAILURE, "Cannot open fred.txt file");
  }
  printf("file size: %ld\n", get_file_size(fp));
  if (fclose(fp) == EOF) {
    err(EXIT_FAILURE, "Failed to close file");
  }
  return EXIT_SUCCESS;
}
```

Listing 8-2: 使用 ftell 和 fseek 函数

该程序打开一个名为 *fred.txt* 的文件，并调用 get_file_size 函数来获取文件大小。get_file_size 函数调用 fseek 将文件位置指示器设置到文件的末尾（由 SEEK_END 指示），并调用 ftell 函数获取文件流当前的文件位置指示器值，作为 long int 类型返回。该值由 get_file_size 函数返回，并在 main 函数中打印出来。最后，我们关闭由 fp 文件指针引用的文件。

fseek 函数对文本文件和二进制文件有不同的限制。对于文本文件，偏移量必须为零或之前由 ftell 返回的值，而对于二进制文件，你可以使用计算出的偏移量。

为确保你的代码健壮，务必检查错误。文件输入输出（File I/O）可能因各种原因而失败。fopen 函数在失败时返回空指针。fseek 函数只有在无法满足请求时才会返回非零值。失败时，ftell 函数返回 −1L，并将一个由实现定义的值存储在 errno 中。如果 ftell 的返回值等于 −1L，我们使用 err 函数打印程序名称的最后一个组件、冒号字符、一个空格，然后是与存储在 errno 中的值对应的错误信息，最后是一个换行符。fclose 函数如果检测到任何错误，将返回 EOF。这个简短程序所展示的 C 标准库的一个不幸之处是，每个函数往往以独特的方式报告错误，因此通常需要参考文档，了解如何测试错误。

`fgetpos` 和 `fsetpos` 函数使用 `fpos_t` 类型来表示偏移量。该类型可以表示任意大的偏移量，这意味着你可以使用 `fgetpos` 和 `fsetpos` 来操作任意大的文件。宽字符流具有一个关联的 `mbstate_t` 对象，该对象存储流的当前解析状态。成功调用 `fgetpos` 会将此多字节状态信息作为 `fpos_t` 对象的一部分存储。之后，使用相同存储的 `fpos_t` 值成功调用 `fsetpos` 会恢复解析状态以及在控制流中的位置。除非间接通过调用 `fsetpos` 后跟 `ftell`，否则无法将 `fpos_t` 对象转换为流中的整数字节或字符偏移量。 清单 8-3 中展示了 `fgetpos` 和 `fsetpos` 函数的使用示例。

```
#include <err.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
  FILE *fp = fopen("fred.txt", "w+");
  if (fp == nullptr) {
    err(EXIT_FAILURE, "Cannot open fred.txt file");
  }
  fpos_t pos;
  if (fgetpos(fp, &pos) != 0) {
    err(EXIT_FAILURE, "get position");
  }
  if (fputs("abcdefghijklmnopqrstuvwxyz", fp) == EOF) {
      fputs("Cannot write to fred.txt file\n", stderr);
  }
  if (fsetpos(fp, &pos) != 0) {
    err(EXIT_FAILURE, "set position");
  }
  long int fpi = ftell(fp);
  if (fpi == -1L) {
    err(EXIT_FAILURE, "ftell");
  }
  printf("file position = %ld\n", fpi);
  if (fputs("0123456789", fp) == EOF) {
    fputs("Cannot write to fred.txt file\n", stderr);
  }
  if (fclose(fp) == EOF) {
    err(EXIT_FAILURE, "Failed to close file\n");
  }
  return EXIT_SUCCESS;
}
```

清单 8-3：使用 fgetpos 和 fsetpos 函数

该程序打开 *fred.txt* 文件进行写入，然后调用 `fgetpos` 来获取当前文件在文件中的位置，该位置存储在 `pos` 中。接着，我们向文件写入一些文本，然后调用 `fsetpos` 将文件位置指示符恢复到存储在 `pos` 中的位置。此时，我们可以使用 `ftell` 函数来检索并打印文件位置，结果应为 0。运行该程序后，*fred.txt* 包含以下文本：

```
0123456789klmnopqrstuvwxyz
```

你不能在写入流之后再读取它，除非先调用 fflush 函数来写入任何未写入的数据，或者调用文件定位函数（fseek、fsetpos 或 rewind）。同样，不能先从流中读取数据再写入，除非先调用文件定位函数。

rewind 函数将文件位置指示器设置为文件的开头：

```
void rewind(FILE *stream);
```

rewind 函数相当于调用 fseek，然后调用 clearerr 来清除流的错误指示器：

```
fseek(stream, 0L, SEEK_SET);
clearerr(stream);
```

因为无法确定 rewind 是否失败，所以应该使用 fseek，以便可以检查错误。

不应尝试在以追加模式打开的文件中使用文件位置，因为许多系统不会修改当前的文件位置指示器用于追加，或者在写入时强制将文件指示器重置为文件末尾。如果使用需要文件位置的 API，则文件位置指示器会通过随后的读取、写入和定位请求保持更新。POSIX 和 Windows 都有一些 API 永远不使用文件位置指示器；对于这些 API，始终需要指定执行 I/O 操作时的文件偏移量。POSIX 定义了 lseek 函数，它的行为与 fseek 类似，但它作用于打开的文件描述符（IEEE Std 1003.1:2018）。

## 删除和重命名文件

C 标准库提供了 remove 函数来删除文件，以及 rename 函数来移动或重命名文件：

```
int remove(const char *filename);
int rename(const char *old, const char *new);
```

在 POSIX 中，文件删除函数是 unlink，目录删除函数是 rmdir：

```
int unlink(const char *path);
int rmdir(const char *path);
```

POSIX 也使用 rename 来重命名文件。C 标准与 POSIX 之间一个显著的区别是，C 标准没有目录的概念，而 POSIX 有。因此，C 标准没有为处理目录定义特定的语义。

unlink 函数比 remove 函数具有更明确定义的语义，因为它是专门针对 POSIX 文件系统的。在 POSIX 和 Windows 中，我们可以有任意数量的文件链接，包括硬链接和打开的文件描述符。unlink 函数始终会删除文件的目录条目，但只有在没有更多的链接或打开的文件描述符引用该文件时，才会删除文件。即使在删除后，文件的内容可能仍然保存在永久存储中。rmdir 函数仅在目录为空时，删除由path指定的目录。

在 POSIX 中，当参数不是目录时，remove 函数的行为必须与 unlink 函数相同；当参数是目录时，它的行为必须与 rmdir 函数相同。remove 函数在其他操作系统上可能表现不同。

文件系统与其他与你的程序同时运行的程序共享。这些其他程序将在你的程序运行期间修改文件系统。这意味着文件条目可能会消失或被其他文件条目替代，这可能成为安全漏洞和意外数据丢失的来源。POSIX 提供了函数，允许你解除链接并重命名由打开的文件描述符或句柄引用的文件。可以使用这些函数来防止在共享公共文件系统中发生安全漏洞和可能的意外数据丢失。

## 使用临时文件

我们经常使用*临时文件*作为进程间通信机制，或者为了将信息暂时存储到磁盘中以释放随机存取内存（RAM）。例如，一个进程可能会写入一个临时文件，另一个进程则从该文件中读取。这些文件通常通过使用像 C 标准库的tmpfile和tmpnam，或 POSIX 的mkstemp等函数在临时目录中创建。

临时目录可以是全局的，也可以是用户特定的。在 Unix 和 Linux 中，TMPDIR 环境变量用于指定全局临时目录的位置，通常是 */tmp* 和 */var/tmp*。运行 Wayland 或 X11 窗口系统的系统通常会通过 $XDG_RUNTIME_DIR 环境变量定义用户特定的临时目录，该变量通常设置为 */run/user/$uid*。在 Windows 中，你可以在用户配置文件的 *AppData* 部分找到用户特定的临时目录，通常为 *C:\Users\User Name\AppData\Local\Temp* (*%USERPROFILE%\AppData\Local\Temp*)。在 Windows 中，全局临时目录由 TMP 或 TEMP 环境变量指定。*C:\Windows\Temp* 目录是 Windows 用来存储临时文件的系统文件夹。

出于安全原因，最好为每个用户配置自己的临时目录，因为使用全局临时目录常常会导致安全漏洞。创建临时文件的最安全函数是 POSIX mkstemp 函数。然而，由于在共享目录中访问文件可能会很困难，甚至无法安全实现，因此我们建议你不要使用任何现有的函数，而是通过使用套接字、共享内存或其他为此目的设计的机制来执行进程间通信。

## 读取格式化文本流

在本节中，我们将演示如何使用 fscanf 函数来读取格式化输入。fscanf 函数是我们在第一章中介绍过的 fprintf 函数的对应输入版本，其函数签名如下：

```
int fscanf(FILE * restrict stream, const char * restrict format, ...);
```

fscanf 函数从由 stream 指向的流中读取输入，按照 format 字符串的控制来处理，该字符串告诉函数预期的参数数量、类型以及如何将它们转换为赋值。后续的参数是指向接收转换输入的对象的指针。如果 format 字符串的参数不足，结果是未定义的。如果提供的参数多于转换说明符，超出的参数会被评估，但会被忽略。fscanf 函数有很多功能，这里我们仅触及其中一部分。有关更多信息，请参阅 C 标准。

为了演示 fscanf 的使用，以及一些其他 I/O 函数，我们将实现一个程序，该程序读取 清单 8-4 中显示的 *signals.txt* 文件并打印出每一行。

```
1 HUP Hangup
2 INT Interrupt
3 QUIT Quit
4 ILL Illegal instruction
5 TRAP Trace trap
6 ABRT Abort
7 EMT EMT trap
8 FPE Floating-point exception
```

清单 8-4: The signals.txt 文件

该文件的每一行包含以下内容：一个信号号（一个小的正整数值）、信号 ID（最多六个字母数字字符的短字符串），以及一个简短的描述信号的字符串。字段之间由空格分隔，描述字段的分隔符为开始处的一个或多个空格或制表符字符，结束处是换行符。

清单 8-5 显示了信号程序，该程序读取此文件并打印出每一行。

```
#include <err.h>
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

#define TO_STR_HELPER(x) #x
#define TO_STR(x) TO_STR_HELPER(x)

#define DESC_MAX_LEN 99

int main() {
  int status = EXIT_SUCCESS;
  FILE *in;

  struct sigrecord {
    int signum;
    char signame[10];
    char sigdesc[DESC_MAX_LEN + 1];
❶} rec;

 if ((in = fopen("signals.txt", "r")) == nullptr) {
    err(EXIT_FAILURE, "Cannot open signals.txt file");
  }

❷ while (true) {
  ❸ int n = fscanf(in, "%d%9s%*[\t]%" TO_STR(DESC_MAX_LEN) "[^\n]",
      &rec.signum, rec.signame, rec.sigdesc
    );
    if (n == 3) {
      printf(
        "Signal\n  number = %d\n  name = %s\n  description = %s\n\n",
        rec.signum, rec.signame, rec.sigdesc
      );
    }
    else if (ferror(in)) {
      perror("Error indicated");
      status = EXIT_FAILURE;
      break;
    }
    else if (n == EOF) {
      // normal end-of-file
      break;
    }
    else if (feof(in)) {
      fputs("Premature end-of-file detected\n", stderr);
      status = EXIT_FAILURE;
      break;
    }
    else {
      fputs("Failed to match signum, signame, or sigdesc\n\n", stderr);
      int c;
      while ((c = getc(in)) != '\n' && c != EOF);
      status = EXIT_FAILURE;
    }
  }

❹ if (fclose(in) == EOF) {
    err(EXIT_FAILURE, "Failed to close file\n");
  }

  return status;
}
```

清单 8-5: 信号程序

我们在main函数中定义了几个变量，包括rec结构体❶，它用于存储文件中每一行找到的信号信息。rec结构体包含三个成员：一个类型为int的<sup class="SANS_TheSansMonoCd_W5Regular_11">signum</sup>成员，用于保存信号编号；一个signame成员，它是一个<char>类型的数组，用于保存信号 ID；还有一个sigdesc成员，也是一个<char>类型的数组，用于保存信号的描述。两个数组的大小是固定的，我们确定它们的大小足以容纳从文件读取的字符串。如果从文件读取的字符串过长，无法容纳到这些数组中，程序会将其视为错误。

调用 fscanf ❸ 读取文件中的每一行输入。它出现在一个无限的 while (true) 循环 ❷ 内，我们必须打破这个循环才能终止程序。我们将 fscanf 函数的返回值赋给一个局部变量 n。如果在第一个转换完成之前发生输入错误，fscanf 函数会返回 EOF。否则，函数返回分配的输入项数，这个数可能少于预期的输入项数，甚至为零，如果发生了提前匹配失败。调用 fscanf 会分配三个输入项，因此只有当 n 等于 3 时，我们才会打印信号描述。接下来，我们调用 ferror(in) 来判断 fscanf 是否设置了错误指示器。如果设置了，我们通过调用 perror 函数打印 errno，然后将状态设置为 EXIT_FAILURE。接下来，如果 n 等于 EOF，我们会退出循环，因为我们已经成功处理了所有输入。最后的可能情况是，fscanf 返回的值既不是预期的输入项数量，也不是表示提前匹配失败的 EOF。在这种情况下，我们将该条件视为非致命错误：

```
fputs("Failed to match signum, signame, or sigdesc\n\n", stderr);
int c;
while ((c = getc(in)) != '\n' && c != EOF);
status = EXIT_FAILURE;
```

我们向 stderr 输出一条信息，通知用户文件中某个信号描述存在问题，但我们继续处理其余条目。循环丢弃有缺陷的行，并将 status 赋值为 EXIT_FAILURE，以指示调用程序发生了错误。你会注意到，程序中的错误处理占据了大部分代码。

fscanf 函数使用一个 *格式字符串*，该字符串决定了输入文本如何分配给每个参数。在这种情况下，"%d%9s%*[\t]%99[^\n]" 格式字符串包含四个 *转换说明符*，它们指定如何将从输入流中读取的输入转换为存储在格式字符串参数引用的对象中的值。我们通过百分号字符 (%) 引入每个转换说明符。在 % 后，可能按顺序出现以下内容：

+   一个可选的字符 *，用于丢弃输入而不将其分配给任何参数

+   一个大于零的可选整数，用于指定最大字段宽度（以字符为单位）

+   一个可选的长度修饰符，用于指定对象的大小

+   一个转换说明符字符，用于指定要应用的转换类型

格式字符串中的第一个转换说明符是 %d。此转换说明符匹配第一个可选符号的十进制整数，该整数应对应于文件中的信号编号，并将值存储在第三个由 rec.signum 引用的参数中。如果没有可选的长度修饰符，则输入的长度取决于转换说明符的默认类型。对于 d 转换说明符，参数必须指向一个 signed int。

此格式字符串中的第二个转换说明符是 %9s，它匹配输入流中的下一个非空白字符序列——对应于信号名称——并将这些字符作为字符串存储在第四个由 rec.signame 引用的参数中。长度修饰符防止输入超过九个字符，并在匹配的字符后在 rec.signame 中写入空字符。此示例中的 %10s 转换说明符将允许发生缓冲区溢出。即便如此，%9s 转换说明符仍然可能无法读取整个字符串，从而导致匹配错误。在将数据读取到固定大小的缓冲区时，如我们所做的，你应当测试精确匹配或稍微超出固定缓冲区长度的输入，以确保不会发生缓冲区溢出，并且字符串正确地以空字符结束。

我们暂时跳过第三个转换说明符，来讲讲第四个转换说明符：%99[^\n]。这个复杂的转换说明符将匹配文件中的信号描述字段。括号（[]）包含一个*扫描集*，类似于正则表达式。这个扫描集使用脱字符（^）来排除 \n 字符。综合起来，%99[^\n] 会读取所有字符，直到遇到 \n（或 EOF）并将它们存储在由 rec.sigdesc 引用的第五个参数中。C 程序员通常使用这种语法来读取整行。此转换说明符还包括 99 字符的最大字符串长度，以避免缓冲区溢出。

现在我们可以重新审视第三个转换说明符：%*[\t]。正如我们刚刚看到的，第四个转换说明符会读取所有字符，从信号 ID 的末尾开始。不幸的是，这包括信号 ID 和描述开始之间的任何空白字符。%*[\t] 转换说明符的目的是消耗这两个字段之间的任何空格或水平制表符字符，并通过使用分配抑制说明符 * 来抑制它们。还可以在此转换说明符的扫描集内包含其他空白字符。

最后，我们调用 fclose 函数 ❹ 来关闭文件。

## 从二进制流中读取和写入

C 标准库中的 fread 和 fwrite 函数可以操作文本流和二进制流。fwrite 函数具有以下签名：

```
size_t fwrite(const void * restrict ptr, size_t size, size_t nmemb,
  FILE * restrict stream);
```

该函数将最多 nmemb 个 size 字节的元素，从 ptr 指向的数组写入到 stream。fwrite 函数的行为类似于将每个对象转换为 unsigned char 数组（每个对象都可以转换为这种类型的数组），然后调用 fputc 函数按顺序写入数组中每个字符的值。流的文件位置指示器会根据成功写入的字符数量进行更新。

POSIX 定义了类似的 read 和 write 函数，它们操作的是文件描述符而非流（IEEE Std 1003.1:2018）。

示例 8-6 演示了使用 fwrite 函数将信号记录写入 *signals.bin* 文件。

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct sigrecord {
  int signum;
  char signame[10];
  char sigdesc[100];
} rec;

int main() {
  int status = EXIT_SUCCESS;
  FILE *fp;

❶ if ((fp = fopen("signals.bin", "wb")) == nullptr) {
    fputs("Cannot open signals.bin file\n", stderr);
    return EXIT_FAILURE;
  }

❷ rec sigrec30 = {30, "USR1", "user-defined signal 1"};
  rec sigrec31 = {
    .signum = 31, .signame = "USR2", .sigdesc = "user-defined signal 2"
  };

  size_t size = sizeof(rec);

❸ if (fwrite(&sigrec30, size, 1, fp) != 1) {
    fputs("Cannot write sigrec30 to signals.bin file\n", stderr);
    status = EXIT_FAILURE;
    goto close_files;
  }

  if (fwrite(&sigrec31, size, 1, fp) != 1) {
    fputs("Cannot write sigrec31 to signals.bin file\n", stderr);
    status = EXIT_FAILURE;
  }

close_files:
  if (fclose(fp) == EOF) {
    fputs("Failed to close file\n", stderr);
    status = EXIT_FAILURE;
  }

  return status;
}
```

示例 8-6：使用直接 I/O 向二进制文件写入

我们以 wb 模式打开 *signals.bin* 文件 ❶ 来创建一个用于写入的二进制文件。我们声明两个 rec 结构体 ❷ 并用我们想要写入文件的信号值初始化它们。为了比较，sigrec30 结构体使用位置初始化器初始化，而 sigrec31 则使用指定初始化器进行初始化。两种初始化方式的行为相同；指定初始化器使声明更加清晰，尽管稍微冗长。实际的写入操作从 ❸ 开始。我们检查每次调用 fwrite 函数的返回值，以确保它写入了正确数量的元素。

示例 8-7 使用 fread 函数从 *signals.bin* 文件中读取我们刚刚写入的数据。

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct rec {
  int signum;
  char signame[10];
  char sigdesc[100];
} rec;

int main() {
  int status = EXIT_SUCCESS;
  FILE *fp;
  rec sigrec;

❶ if ((fp = fopen("signals.bin", "rb")) == nullptr) {
    fputs("Cannot open signals.bin file\n", stderr);
    return EXIT_FAILURE;
  }

  // read the second signal
❷ if (fseek(fp, sizeof(rec), SEEK_SET)  != 0) {
    fputs("fseek in signals.bin file failed\n", stderr);
    status = EXIT_FAILURE;
    goto close_files;
  }

❸ if (fread(&sigrec, sizeof(rec), 1, fp) != 1) {
    fputs("Cannot read from signals.bin file\n", stderr);
    status = EXIT_FAILURE;
    goto close_files;
  }

  printf(
    "Signal\n  number = %d\n  name = %s\n  description = %s\n\n",
    sigrec.signum, sigrec.signame, sigrec.sigdesc
  );

close_files:
  if (fclose(fp) == EOF) {
    fputs("Failed to close file\n", stderr);
    status = EXIT_FAILURE;
  }

  return status;
}
```

示例 8-7：使用直接 I/O 从二进制文件读取

我们使用 rb 模式 ❶ 打开二进制文件进行读取。接下来，为了让这个示例更加有趣，程序读取并打印特定信号的信息，而不是读取整个文件。我们可以通过程序参数指定读取哪个信号，但为了这个示例，我们将其硬编码为第二个信号。为此，程序调用 fseek 函数 ❷ 来设置由 fp 引用的流的文件位置指示器。如本章前面所述，文件位置指示器决定了随后的 I/O 操作的文件位置。对于二进制流，我们通过将偏移量（以字节为单位）加到由最后一个参数指定的位置（即文件开头，使用 SEEK_SET 指示）来设置新位置。第一个信号位于文件的 0 位置，随后的每个信号都位于文件开头的结构大小的整数倍位置。

文件位置指示器定位到第二个信号的开始后，我们调用 fread 函数 ❸ 从二进制文件读取数据到由 &sigrec 引用的结构中。调用 fread 读取一个元素，该元素的大小由 sizeof(rec) 指定，从由 fp 指向的流中读取。在大多数情况下，这个对象的大小和类型与相应的 fwrite 调用相同。流的文件位置指示器将根据成功读取的字符数而前进。我们检查 fread 函数的返回值，以确保读取了正确数量的元素，这里是一个元素。

## 字节序

除字符类型外的对象类型可能包含填充和数值表示位。不同的目标平台可以以不同的方式将字节打包为多字节字，这种方式称为 *字节序*。

> 注意

*字节序* 这一术语来源于乔纳森·斯威夫特 1726 年的讽刺作品《格列佛游记》，其中发生了内战，争论的是煮蛋时，应该从蛋的大端还是小端打开。

*大端序*将最高有效字节放在最前面，最低有效字节放在最后，而*小端序*则相反。例如，考虑无符号十六进制数字0x1234，它需要至少两个字节来表示。在大端序中，这两个字节是0x12和0x34，而在小端序中，字节排列为0x34和0x12。Intel 和 AMD 处理器使用小端格式，而 ARM 和 POWER 系列处理器可以在小端和大端格式之间切换。然而，大端序是网络协议中占主导地位的顺序，如互联网协议（IP）、传输控制协议（TCP）和用户数据报协议（UDP）。字节序可能会导致问题，当一个计算机上创建了二进制文件并在另一台具有不同字节序的计算机上读取时。

C23 增加了一种机制，用于在运行时使用三个宏确定实现的字节顺序，这些宏扩展为整数常量表达式。__STDC_ENDIAN_LITTLE__宏表示一种字节顺序存储，其中最低有效字节最先放置，其余字节按升序排列。__STDC_ENDIAN_BIG__宏表示一种字节顺序存储，其中最高有效字节最先放置，其余字节按降序排列。

__STDC_ENDIAN_NATIVE__宏描述执行环境中与位精确整数类型、标准整数类型和大多数扩展整数类型相关的字节序。清单 8-8 中的简短程序通过测试__STDC_ENDIAN_NATIVE__宏的值来确定执行环境的字节顺序。如果执行环境既不是小端序也不是大端序，并且有某种其他实现定义的字节顺序，则__STDC_ENDIAN_NATIVE__宏将具有不同的值。

```
#include <stdbit.h>
#include <stdio.h>

int main (int argc, char* argv[]) {
  if (__STDC_ENDIAN_NATIVE__ == __STDC_ENDIAN_LITTLE__) {
    puts("little endian");
  }
  else if (__STDC_ENDIAN_NATIVE__ == __STDC_ENDIAN_BIG__) {
    puts("big endian");
  }
  else {
    puts("other byte ordering");
  }
  return 0;
}
```

清单 8-8：确定字节顺序

各平台之间的这些差异意味着，对于主机间通信，你应当采用一个标准的外部格式，并使用格式转换函数将外部数据的数组*编组*到多个字节的本地对象之间（使用精确宽度的类型）。POSIX 提供了一些适合此目的的函数，包括 htonl、htons、ntohl 和 ntohs，它们用于在主机字节序和网络字节序之间转换值。

在二进制数据格式中，可以通过始终以固定字节序存储数据，或在二进制文件中包含一个字段来指示数据的字节序，从而实现字节序的独立性。

## 总结

在本章中，你学习了流的相关内容，包括流缓冲、预定义流、流方向，以及文本流和二进制流之间的区别。

然后，你学习了如何使用 C 标准库和 POSIX API 创建、打开和关闭文件。你还学习了如何读取和写入字符和行，读取和写入格式化文本，以及从二进制流中读取和写入数据。你了解了如何刷新流、设置文件位置、删除文件和重命名文件。如果没有输入/输出，用户与程序的通信将仅限于程序的返回值。最后，你学习了临时文件以及如何避免使用它们。

在下一章中，你将学习编译过程和预处理器的相关内容，包括文件包含、条件包含和宏定义。
