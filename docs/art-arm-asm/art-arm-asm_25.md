

## 第二十一章：E 有用的 C 语言函数



![](img/opener.jpg)

本附录包含了来自 C 标准库（以及 Unix 系统库）的几个 C 函数的列表，这些函数可能对汇编语言程序员有用。

这些函数在 macOS 中的变体使用一个以下划线开头的外部名称。例如，在 macOS 中，strlen() 变成了 _strlen() 函数。*aoaa.inc* 头文件包含了许多这些函数名称的 #define 声明，在未修饰的名称前添加了下划线前缀：#define strlen _strlen。

## E.1 字符串函数

本书的各个章节介绍了许多 C 标准库字符串函数（在 *strings.h* 头文件中声明）。本节描述了大多数可用的函数，包括本书未使用的那些：

char *strcat(char *dest, const char *src);

将 X1 (src) 指向的以零结尾的字符串连接到 X0 (dest) 指向的字符串的末尾。返回 X0 中指向 dest 字符串的指针。

char *strchr(const char *str, int c);

在由 str (X0) 指向的字符串中搜索字符 c (X1) 的第一次出现。返回指向字符串中找到该字符的位置的指针（在 X0 中），如果 c 在 str 中不存在，则返回 NULL (0) 指针。

char *strcpy(char *dest, const char *src);

将 src (X1) 指向的字符串复制到 dest (X0)，包括零终止字节。返回指向 dest 的指针（在 X0 中）。

char *strdup(char *str);

在堆上复制一个字符串。传入时，X0 包含要复制的字符串的指针。返回时，X0 包含指向在堆上分配的字符串副本的指针。当应用程序使用完该字符串后，应该调用 C 标准库的 free() 函数将存储空间归还给堆。虽然 strdup() 在 C 标准库中未定义，但大多数系统在其库中包含了它。

char *strncat(char *dest, const char *src, size_t n);

将 X1 (src) 指向的以零结尾的字符串的最多 n 个字符连接到 X0 (dest) 指向的字符串的末尾，并加上一个零终止字节。返回 X0 中指向 dest 字符串的指针。如果 src 的长度小于 n，则该字符串只会将 src 中的前 n 个字符复制到 dest（加上一个零终止字节）。

char *strpbrk(const char *str1, const char *str2);

查找字符串 str1（由 X0 传入）中第一个与 str2（由 X1 传入）中任意字符匹配的字符。返回指向 str1 中匹配字符的指针（如果没有匹配，则返回 NULL）。

char *strrchr(const char *str, int c);

在由参数 str (传入 X0) 指向的字符串中搜索字符 c（在 X1 中传递）的最后一次出现。返回指向 str 中找到该字符的位置的指针（在 X0 中）。如果字符在 str 中未找到，则此函数在 X0 中返回 NULL (0)。

char *strstr(const char *inStr, const char *search4);

在 inStr（传入 X0）中搜索字符串 search4（传入 X1）的第一个出现位置。如果在 inStr 中找不到 search4 字符串，则返回 NULL（0）。

char *strtok(char *str, char *delim);

将字符串 str（传入 X0）按照分隔符 delim（传入 X1）中的字符分割成一系列*标记*（单词）。在第一次调用时，函数期望将 C 字符串作为 str 的参数，其第一个字符用作扫描标记的起始位置。在后续调用中，函数期望一个空指针（NULL，0），并将上一个标记的末尾位置作为新的扫描起始位置（跳过任何前导分隔符字符）。每次调用返回一个指向字符串中下一个标记的指针（在 X0 中）。当字符串中的所有标记耗尽时，此函数返回 NULL。

此函数修改 str（X0 指向的字符串）的内容。如果您的程序不能容忍此操作，请在调用 strtok() 前复制 str。strtok() 函数在静态变量中维护内部状态，因此不适合在多线程应用程序中使用。

int memcmp(void *mem1, void *mem2, size_t n);

比较 mem1（传入 X0）和 mem2（传入 X1）的前 n 个字节。其操作类似于 strcmp()，但此函数在遇到 0 字节时不会结束比较；相比之下，strcmp()会根据比较结果返回负值、0 或正值。

int strcasecmp(const char *str1, const char *str2);

比较 str1（X0 指向的字符串）与 str2（X1 指向的字符串）的内容，采用不区分大小写的比较。如果 str1 < str2，则返回（在 X0 中）一个负数；如果 str1 == str2，则返回 0；如果 str1 > str2，则返回一个正数。尽管 strcasecmp() 在 C 标准库中未定义，但许多系统在其库中包含它；有些系统使用 strcmpi() 或 stricmp() 作为函数名。

int strcmp(const char *str1, const char *str2);

比较 str1（X0 指向的字符串）和 str2（X1 指向的字符串）的内容，并返回（在 X0 中）一个负数、0 或正数。如果 str1 < str2，则返回一个负数；如果 str1 == str2，则返回 0；如果 str1 > str2，则返回一个正数。

int strncmp(char *str1, char *str2, size_t n);

比较两个字符串的前 n 个字符，或者直到遇到第一个零终止字节（在任一字符串中）。指针 str1 和 str2 分别传入 X0 和 X1，n 传入 X2。如果字符串通过前 n 个字符相等（或者两个字符串都相等且长度小于 n），则返回 0。如果 str1 小于 str2，则返回一个负数。如果 str1 大于 str2，则返回一个正数。您可以使用此函数通过设置 n 为 str1 的长度来判断 str1 是否是 str2 的前缀。

size_t strcspn(const char *str1, const char *str2);

计算 str1（由 X0 传递）的初始段的长度，该段只包含不在 str2（由 X1 传递）中的字符。返回此计数值保存在 X0 中。

size_t strlen(char *str);

计算零终止字符串的长度。进入时，X0 包含指向字符串的指针，该函数返回字符串的长度（不包括零终止字节）。

size_t strspn(const char *str1, const char *str2);

计算 str1（由 X0 传递）的初始段的长度，该段只包含在 str2（由 X1 传递）中出现的字符。返回此计数值保存在 X0 中。

strlwr(str);

将字符串中的所有字符转换为小写字母。进入时，X0 包含指向待转换字符串的指针；返回时，X0 指向同一字符串，且其中的大写字母已被转换为小写字母。虽然 strlwr()在 C 标准库中没有定义，但许多系统在其库中包含了该函数。

strncpy(char *dest, const char *src, size_t n);

从 src（由 X1 传递）复制最多 n 个字符（由 X2 传递）到 dest（由 X0 传递）。如果 n 小于或等于 src 的长度，则此函数不会复制零终止字节，调用者需要负责添加该额外字节。此函数有两个主要用途。首先，它可以防止覆盖 dest 末尾之后的数据（当 n 包含 dest 缓冲区的大小，再加 1 时，即 X0 指向的位置）。其次，它作为一个子字符串函数，允许从字符串中的特定位置提取 n 个字符。

strupr(str);

将字符串中的所有小写字母转换为大写字母。进入时，X0 包含指向待转换字符串的指针；返回时，X0 指向同一字符串，且其中的小写字母已被转换为大写字母。虽然 strupr()在 C 标准库中没有定义，但许多系统在其库中包含了该函数。

void *memchr(void *mem, int c, size_t n);

在 mem（由 X0 传递）指向的内存块的前 n（由 X2 传递）字节中搜索字符 c（无符号字符，由 X1 传递）的第一次出现。与 strchr()非常相似，不同之处在于此函数在遇到零字节时不会停止扫描。返回 X0 中，找到字符的位置的指针，如果字符 c 在 mem 中不存在，则返回 NULL（0）。

void *memcpy(void *dest, const void *src, size_t n);

从 src 复制 n 字节到 dest（分别由 X2、X1 和 X0 传递）。返回指向 dest 的指针，保存在 X0 中。如果 dest 定义的内存块与 src 定义的内存块重叠，结果是未定义的。

void *memmove(void *dest, const void *src, size_t n);

从 src 复制 n 字节到 dest（分别由 X2、X1 和 X0 传递）。返回指向 dest 的指针，保存在 X0 中。

memmove()函数正确处理源和目标内存块重叠的情况。然而，由于该函数可能比 memcpy()稍慢，因此只有在无法保证内存块不重叠时，才应使用此函数。

void *memset(void *mem, int c, size_t n);

将传入 X1 的 c 的 LO 字节复制到由参数 mem（传入 X0）指向的内存块的前 n（传入 X2）字节中。返回指向该内存块的指针，存放在 X0 中。

## E.2 其他 C 标准库和 Unix 函数

本附录中介绍的字符串函数只是 C 标准库中众多函数的一小部分。其他有用的函数包括 POSIX 文件 I/O 函数（在 *fcntl.h* 和 *unistd.h* 头文件中声明）、数学库（在 *math.h* 中找到）等。有关这些头文件的更多信息，请参见以下链接：

***fcntl.h***

*[`<wbr>pubs<wbr>.opengroup<wbr>.org<wbr>/onlinepubs<wbr>/000095399<wbr>/basedefs<wbr>/fcntl<wbr>.h<wbr>.html`](https://pubs.opengroup.org/onlinepubs/000095399/basedefs/fcntl.h.html)*

***math.h***

*[`<wbr>pubs<wbr>.opengroup<wbr>.org<wbr>/onlinepubs<wbr>/9699919799<wbr>/basedefs<wbr>/math<wbr>.h<wbr>.html`](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/math.h.html)*

***unistd.h***

*[`<wbr>pubs<wbr>.opengroup<wbr>.org<wbr>/onlinepubs<wbr>/007908775<wbr>/xsh<wbr>/unistd<wbr>.h<wbr>.html`](https://pubs.opengroup.org/onlinepubs/007908775/xsh/unistd.h.html)*

你可以通过指定函数名称来轻松调用这些函数（在 macOS 中调用函数时不要忘记在函数名前加下划线）。你始终通过使用 Linux 的 ARM ABI 或 macOS 的 macOS ABI 来传递参数和获取函数结果（请记住，macOS 在传递变长参数列表给函数时有所不同，例如 printf()）。
