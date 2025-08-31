## 第三章 系统编程概念

本章涵盖了系统编程的各种前提知识。我们首先介绍系统调用，并详细说明它们执行过程中发生的步骤。然后，我们讨论库函数及其与系统调用的区别，并结合对（GNU）C 库的描述。

每当我们发起系统调用或调用库函数时，都应始终检查调用的返回状态，以确定调用是否成功。我们描述了如何执行这些检查，并展示了一组函数，这些函数用于本书中大多数示例程序来诊断来自系统调用和库函数的错误。

我们通过探讨与可移植编程相关的各种问题来结束本章，特别是使用特性测试宏和 SUSv3 定义的标准系统数据类型。

## 系统调用

一个 *系统调用* 是进入内核的受控入口点，允许进程请求内核代表该进程执行某些操作。内核通过系统调用应用程序编程接口（API）向程序提供一系列服务。这些服务包括例如创建新进程、执行 I/O 操作和创建管道进行进程间通信。（*syscalls(2)* 手册页列出了 Linux 系统调用。）

在深入讨论系统调用的工作原理之前，我们先指出一些通用要点：

+   系统调用将处理器状态从用户模式切换到内核模式，以便 CPU 可以访问受保护的内核内存。

+   系统调用集是固定的。每个系统调用都有一个唯一的编号。（这种编号方案通常对程序不可见，程序通过名称识别系统调用。）

+   每个系统调用可能有一组参数，指定从用户空间（即进程的虚拟地址空间）到内核空间的传输信息，以及反向传输的信息。

从编程角度来看，调用系统调用看起来类似于调用 C 函数。然而，在幕后，系统调用的执行过程中会发生许多步骤。为了说明这一点，我们考虑在特定硬件实现（x86-32）上按顺序发生的步骤。具体步骤如下：

1.  应用程序通过调用 C 库中的包装函数来发起系统调用。

1.  包装函数必须将所有系统调用参数传递给系统调用陷入处理程序（稍后描述）。这些参数通过栈传递给包装函数，但内核期望它们位于特定的寄存器中。包装函数将参数复制到这些寄存器中。

1.  由于所有系统调用都以相同的方式进入内核，因此内核需要某种方法来识别系统调用。为此，包装函数将系统调用编号复制到特定的 CPU 寄存器（`%eax`）。

1.  包装函数执行一个*trap*机器指令（`int 0x80`），这会导致处理器从用户模式切换到内核模式，并执行指向系统陷阱向量位置`0x80`（十进制 128）的代码。

    ### 注意

    较新的 x86-32 架构实现了`sysenter`指令，它比传统的`int 0x80`陷阱指令提供了一种更快速的进入内核模式的方法。`sysenter`的使用在 2.6 内核及*glibc* 2.3.2 及以后的版本中得到支持。

1.  在响应到位置`0x80`的陷阱时，内核调用其*system_call()*例程（位于汇编文件`arch/i386/entry.S`中）来处理该陷阱。该处理程序：

    1.  将寄存器值保存到内核栈中（堆栈和堆栈框架）。

    1.  检查系统调用号的有效性。

    1.  调用相应的系统调用服务例程，通过使用系统调用号来索引所有系统调用服务例程的表（内核变量*sys_call_table*）。如果系统调用服务例程有任何参数，它首先检查这些参数的有效性；例如，它检查地址是否指向用户内存中的有效位置。然后，服务例程执行所需的任务，这可能涉及修改给定参数中指定的地址的值，并在用户内存和内核内存之间传输数据（例如，在 I/O 操作中）。最后，服务例程将结果状态返回给*system_call()*例程。

    1.  从内核栈中恢复寄存器值，并将系统调用的返回值放置在栈上。

    1.  返回到包装函数，同时将处理器返回到用户模式。

1.  如果系统调用服务例程的返回值指示出错，包装函数将使用此值设置全局变量*errno*（请参阅处理来自系统调用和库函数的错误）。然后，包装函数返回给调用者，提供一个整数返回值，表示系统调用的成功或失败。

    ### 注意

    在 Linux 中，系统调用服务例程遵循返回非负值以表示成功的约定。如果发生错误，例程返回一个负数，即*errno*常量的取反值。当返回负值时，C 库包装函数将其取反（使其变为正数），将结果复制到*errno*中，并返回-1 作为包装函数的返回值，表示错误返回给调用程序。

    这一约定假设系统调用服务例程在成功时不会返回负值。然而，对于一些例程，这一假设并不成立。通常这不是问题，因为负的 *errno* 值的范围与有效的负返回值不重叠。然而，这一约定在某些情况下会造成问题：例如 *fcntl()* 系统调用的 `F_GETOWN` 操作，我们将在第 63.3 节中描述。

图 3-1 通过 *execve()* 系统调用的例子说明了上述过程。在 Linux/x86-32 上，*execve()* 是系统调用编号 11 (`__NR_execve`)。因此，在 *sys_call_table* 向量中，条目 11 包含 *sys_execve()* 的地址，这是该系统调用的服务例程。（在 Linux 中，系统调用服务例程通常具有 *sys_xyz()* 这样的命名形式，其中 *xyz()* 是相关的系统调用。）

前述段落提供的信息对于本书的其余部分通常不需要了解。然而，它突出了一个重要观点：即使是一个简单的系统调用，也需要执行相当多的工作，因此系统调用会有一个小但显著的开销。

### 注意

作为进行系统调用开销的一个例子，考虑 *getppid()* 系统调用，它仅返回调用进程的父进程的进程 ID。在作者的一台运行 Linux 2.6.25 的 x86-32 系统上，1000 万次调用 *getppid()* 大约需要 2.2 秒才能完成。这相当于每次调用大约 0.3 微秒。相比之下，在同一系统上，1000 万次调用一个简单返回整数的 C 函数仅需要 0.11 秒，即是调用 *getppid()* 所需时间的大约二十分之一。当然，大多数系统调用的开销要远高于 *getppid()*。

由于从 C 程序的角度来看，调用 C 库封装函数等同于调用相应的系统调用服务例程，在本书的其余部分，我们使用诸如“调用系统调用 *xyz()*”之类的表述来表示“调用封装函数，该函数调用系统调用 *xyz()*”。

![系统调用执行步骤](img/03-1_PROGCONC-syscall-scale90.png.jpg)图 3-1. 系统调用执行步骤

附录 A 描述了 *strace* 命令，它可以用来跟踪程序发出的系统调用，无论是用于调试目的，还是仅仅为了调查程序的行为。

有关 Linux 系统调用机制的更多信息，可以参考 [Love, 2010]、[Bovet & Cesati, 2005] 和 [Maxwell, 1999]。

## 库函数

*库函数*仅仅是构成标准 C 库的众多函数之一。（为了简便，在本书的其余部分，我们通常会直接写*function*而不是*library function*。）这些函数的用途非常广泛，包括打开文件、将时间转换为可读格式、以及比较两个字符字符串等任务。

许多库函数并不使用系统调用（例如，字符串处理函数）。另一方面，一些库函数是在系统调用之上构建的。例如，*fopen()*库函数使用*open()*系统调用来实际打开文件。通常，库函数的设计是为了提供比底层系统调用更友好的接口。例如，*printf()*函数提供输出格式化和数据缓冲，而*write()*系统调用只是输出一块字节。类似地，*malloc()*和*free()*函数执行各种记账任务，使得它们比底层*brk()*系统调用更容易分配和释放内存。

## 标准 C 库；GNU C 库（*glibc*）

在不同的 UNIX 实现上，标准 C 库有不同的实现。在 Linux 上最常用的实现是 GNU C 库（*glibc*，[`www.gnu.org/software/libc/`](http://www.gnu.org/software/libc/)）。

### 注意

GNU C 库的主要开发者和维护者最初是 Roland McGrath。如今，这项工作由 Ulrich Drepper 负责。

其他各种 C 库也适用于 Linux，包括适用于嵌入式设备应用的内存需求较小的库。例如，包括*uClibc*（[`www.uclibc.org/`](http://www.uclibc.org/)）和*diet libc*（[`www.fefe.de/dietlibc/`](http://www.fefe.de/dietlibc/)）。在本书中，我们将讨论限制为*glibc*，因为它是大多数在 Linux 上开发的应用程序所使用的 C 库。

#### 确定系统上*glibc*的版本

有时，我们需要确定系统上*glibc*的版本。从 shell 中，我们可以通过将*glibc*共享库文件当作可执行程序运行来做到这一点。当我们将库文件作为可执行文件运行时，它会显示各种文本，包括其版本号：

```
`$ /lib/libc.so.6`
GNU C Library stable release version 2.10.1, by Roland McGrath et al.
Copyright (C) 2009 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.
There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE.
Compiled by GNU CC version 4.4.0 20090506 (Red Hat 4.4.0-4).
Compiled on a Linux >>2.6.18-128.4.1.el5<< system on 2009-08-19.
Available extensions:
        The C stubs add-on version 2.1.2.
        crypt add-on version 2.1 by Michael Glad and others
        GNU Libidn by Simon Josefsson
        Native POSIX Threads Library by Ulrich Drepper et al
        BIND-8.2.3-T5B
        RT using linux kernel aio
For bug reporting instructions, please see:
<http://www.gnu.org/software/libc/bugs.html>.
```

在某些 Linux 发行版中，GNU C 库位于与`/lib/libc.so.6`不同的路径下。确定库文件位置的一种方法是运行*ldd*（列出动态依赖项）程序，针对动态链接到*glibc*的可执行文件（大多数可执行文件都是这样链接的）。然后，我们可以检查生成的库依赖列表，以找到*glibc*共享库的位置：

```
$ ldd myprog | grep libc
        libc.so.6 => /lib/tls/libc.so.6 (0x4004b000)
```

有两种方法可以让应用程序确定系统中当前存在的 GNU C 库的版本：通过测试常量或调用库函数。从 2.0 版本开始，*glibc* 定义了两个常量，`__GLIBC__` 和 `__GLIBC_MINOR__`，可以在编译时进行测试（在 `#ifdef` 语句中）。在安装有 *glibc* 2.12 的系统上，这些常量的值分别为 2 和 12。但是，这些常量在程序在一个系统上编译而在另一个具有不同 *glibc* 版本的系统上运行时，作用有限。为了应对这种情况，程序可以调用 *gnu_get_libc_version()* 函数来确定运行时可用的 *glibc* 版本。

```
#include <gnu/libc-version.h>

const char *`gnu_get_libc_version`(void);
```

### 注意

返回指向以空字符结尾的、静态分配的字符串，包含 GNU C 库版本号

*gnu_get_libc_version()* 函数返回指向字符串的指针，如 *2.12*。

### 注意

我们还可以通过使用 *confstr()* 函数来获取版本信息，从而检索 (*glibc* 特有的) `_CS_GNU_LIBC_VERSION` 配置变量的值。此调用会返回类似 *glibc 2.12* 的字符串。

## 处理系统调用和库函数的错误

几乎每个系统调用和库函数都会返回某种类型的状态值，表示调用是否成功。这个状态值应当*始终*进行检查，以确定调用是否成功。如果没有成功，则应采取适当的措施——至少程序应该显示一个错误消息，警告发生了意外情况。

尽管通过省略这些检查来节省输入时间是很有诱惑力的（特别是在看到一些未检查状态值的 UNIX 和 Linux 程序示例之后），但这是一种错误的节省。因为没有检查系统调用或库函数的状态返回，可能会浪费大量的调试时间，尽管这些调用“看似不会失败”。

### 注意

一些系统调用永远不会失败。例如，*getpid()* 总是成功返回一个进程的 ID，且 *_exit()* 总是成功终止一个进程。对于这样的系统调用，检查其返回值并非必要。

#### 处理系统调用错误

每个系统调用的手册页面都记录了该调用可能的返回值，并指出哪些值表示错误。通常，错误会通过返回 -1 来表示。因此，系统调用可以通过如下代码进行检查：

```
fd = open(pathname, flags, mode);       /* system call to open a file */
if (fd == -1) {
    /* Code to handle the error */
}
...
if (close(fd) == -1) {
    /* Code to handle the error */
}
```

当系统调用失败时，它会将全局整数变量 *errno* 设置为一个正值，用以标识具体的错误。包含 `<errno.h>` 头文件可以声明 *errno*，并提供一组常量来表示各种错误号码。所有这些符号名称都以 `E` 开头。每个手册页面中以 `ERRORS` 为标题的部分提供了每个系统调用可能返回的 *errno* 值的列表。以下是使用 *errno* 来诊断系统调用错误的简单示例：

```
cnt = read(fd, buf, numbytes);
if (cnt == -1) {
    if (errno == EINTR)
        fprintf(stderr, "read was interrupted by a signal\n");
    else {
        /* Some other error occurred */
    }
}
```

成功的系统调用和库函数永远不会将*errno*重置为 0，因此这个变量可能因为之前调用的错误而具有非零值。此外，SUSv3 允许成功的函数调用将*errno*设置为非零值（尽管很少有函数这样做）。因此，在检查错误时，我们应始终首先检查函数返回值是否指示错误，只有在此基础上再检查*errno*以确定错误原因。

一些系统调用（例如，*getpriority()*）在成功时也可以合法地返回-1。要确定此类调用是否发生错误，我们在调用前将*errno*设置为 0，然后在调用后检查它。如果调用返回-1 且*errno*非零，则发生了错误。（类似的说明也适用于一些库函数。）

在系统调用失败后，常见的处理方式是根据*errno*值打印错误消息。为此，提供了*perror()*和*strerror()*库函数。

*perror()*函数打印其*msg*参数指向的字符串，后面跟着与当前*errno*值对应的消息。

```
#include <stdio.h>

void `perror`(const char **msg*);
```

处理系统调用错误的一种简单方法如下：

```
fd = open(pathname, flags, mode);
if (fd == -1) {
    perror("open");
    exit(EXIT_FAILURE);
}
```

*strerror()*函数返回与*errnum*参数中给定的错误编号对应的错误字符串。

```
#include <string.h>

char *`strerror`(int *errnum*);
```

### 注意

返回指向与*errnum*对应的错误字符串的指针

*strerror()*返回的字符串可能是静态分配的，这意味着它可能会被后续的*strerror()*调用覆盖。

如果*errnum*指定了一个无法识别的错误编号，*strerror()*会返回一个形如*Unknown error nnn*的字符串。在一些其他实现中，*strerror()*在这种情况下会返回`NULL`。

因为*perror()*和*strerror()*函数是与区域设置相关的（区域设置），所以错误描述会以本地语言显示。

#### 处理库函数的错误

各种库函数返回不同的数据类型和不同的值来表示失败。（请查阅每个函数的手册页。）对于我们的目的，库函数可以分为以下几类：

+   一些库函数以与系统调用完全相同的方式返回错误信息：-1 的返回值，*errno*指示具体的错误。一个这样的函数是*remove()*，它删除一个文件（使用*unlink()*系统调用）或一个目录（使用*rmdir()*系统调用）。这些函数的错误可以像系统调用的错误一样进行诊断。

+   一些库函数在出错时会返回非-1 的值，但仍然会设置*errno*以指示具体的错误情况。例如，*fopen()*在出错时会返回`NULL`指针，*errno*的值取决于哪个底层系统调用失败。可以使用*perror()*和*strerror()*函数来诊断这些错误。

+   其他库函数根本不使用*errno*。确定错误存在与否以及错误原因的方法取决于具体的函数，并且在该函数的手册页中有详细说明。对于这些函数，使用*errno*、*perror()*或*strerror()*来诊断错误是错误的做法。

## 本书示例程序的说明

在本节中，我们将描述本书中示例程序常用的各种约定和特性。

### 命令行选项和参数

本书中的许多示例程序依赖于命令行选项和参数来确定它们的行为。

传统的 UNIX 命令行选项由一个起始的连字符、一个表示选项的字母和一个可选的参数组成。（GNU 工具提供了一种扩展的选项语法，由两个起始连字符组成，后跟一个表示选项的字符串以及一个可选的参数。）为了解析这些选项，我们使用标准的*getopt()*库函数（该函数在附录 B 中有介绍）。

每个示例程序如果有复杂的命令行语法，都提供了一个简单的帮助功能供用户使用：如果使用`—*help*`选项调用该程序，程序会显示一条用法信息，指示命令行选项和参数的语法。

### 常用函数和头文件

大多数示例程序都包含一个包含常用定义的头文件，并且它们还使用了一组常用函数。我们将在本节中讨论该头文件和这些函数。

#### 常用头文件

示例 3-1 是本书几乎每个程序使用的头文件。这个头文件包含了许多示例程序使用的其他头文件，定义了*Boolean*数据类型，并定义了计算两个数值最小值和最大值的宏。使用这个头文件可以使示例程序稍微简短一些。

示例 3-1. 大多数示例程序使用的头文件

```
`lib/tlpi_hdr.h`
#ifndef TLPI_HDR_H
#define TLPI_HDR_H      /* Prevent accidental double inclusion */

#include <sys/types.h>  /* Type definitions used by many programs */
#include <stdio.h>      /* Standard I/O functions */
#include <stdlib.h>     /* Prototypes of commonly used library functions,
                           plus EXIT_SUCCESS and EXIT_FAILURE constants */
#include <unistd.h>     /* Prototypes for many system calls */
#include <errno.h>      /* Declares errno and defines error constants */
#include <string.h>     /* Commonly used string-handling functions */

#include "get_num.h"    /* Declares our functions for handling numeric
                           arguments (getInt(), getLong()) */

#include "error_functions.h"  /* Declares our error-handling functions */

typedef enum { FALSE, TRUE } Boolean;

#define min(m,n) ((m) < (n) ? (m) : (n))
#define max(m,n) ((m) > (n) ? (m) : (n))

#endif
      `lib/tlpi_hdr.h`
```

#### 错误诊断函数

为了简化示例程序中的错误处理，我们使用了错误诊断函数，相关声明见示例 3-2。

示例 3-2. 常用错误处理函数的声明

```
`lib/error_functions.h`
#ifndef ERROR_FUNCTIONS_H
#define ERROR_FUNCTIONS_H

void errMsg(const char *format, ...);

#ifdef __GNUC__

/* This macro stops 'gcc -Wall' complaining that "control reaches
       end of non-void function" if we use the following functions to
       terminate main() or some other non-void function. */

#define NORETURN __attribute__ ((__noreturn__))
#else
#define NORETURN
#endif

void errExit(const char *format, ...) NORETURN ;

void err_exit(const char *format, ...) NORETURN ;

void errExitEN(int errnum, const char *format, ...) NORETURN ;

void fatal(const char *format, ...) NORETURN ;

void usageErr(const char *format, ...) NORETURN ;

void cmdLineErr(const char *format, ...) NORETURN ;

#endif
      `lib/error_functions.h`
```

为了诊断系统调用和库函数的错误，我们使用*errMsg()*、*errExit()*、*err_exit()*和*errExitEN()*。

```
#include "tlpi_hdr.h"

void `errMsg`(const char **format*, ...);
void `errExit`(const char **format*, ...);
void `err_exit`(const char **format*, ...);
void `errExitEN`(int *errnum*, const char **format*, ...);
```

*errMsg()* 函数将消息打印到标准错误。它的参数列表与 *printf()* 相同，只是输出字符串末尾会自动添加一个换行符。*errMsg()* 函数打印与当前 *errno* 值对应的错误文本——这包括错误名（如 `EPERM`），以及 *strerror()* 返回的错误描述——后跟参数列表中指定的格式化输出。

*errExit()* 函数的操作类似于 *errMsg()*，但它还会终止程序，调用 *exit()*，或者如果环境变量 `EF_DUMPCORE` 被定义且其值非空，则通过调用 *abort()* 来产生核心转储文件，以便与调试器一起使用。（我们在第 22.1 节解释了核心转储文件。）

*err_exit()* 函数类似于 *errExit()*，但有两个不同之处：

+   它不会在打印错误消息之前刷新标准输出。

+   它通过调用 _*exit()* 而不是 *exit()* 来终止进程。这会导致进程终止时不刷新 *stdio* 缓冲区，也不调用退出处理程序。

这些 *err_exit()* 操作差异的细节将在第二十五章中更加清晰地解释，在那里我们描述了 _*exit()* 与 *exit()* 之间的区别，并考虑了由 *fork()* 创建的子进程中 *stdio* 缓冲区和退出处理程序的处理方式。现在，我们只是简单地注意到，如果我们编写一个库函数来创建一个因错误需要终止的子进程，*err_exit()* 特别有用。这个终止应该在不刷新子进程复制的父进程（即调用进程）*stdio* 缓冲区的情况下发生，也不调用父进程建立的退出处理程序。

*errExitEN()* 函数与 *errExit()* 相同，不同之处在于，它不会打印与当前 *errno* 值对应的错误文本，而是打印与传入参数 *errnum* 中给定的错误号对应的错误文本（因此有 *EN* 后缀）。

主要地，我们在使用 POSIX 线程 API 的程序中使用 *errExitEN()*。与传统的 UNIX 系统调用（错误时返回 -1）不同，POSIX 线程函数通过返回错误号（即通常放置在 *errno* 中的正数）来诊断错误，作为它们的函数结果。（POSIX 线程函数在成功时返回 0。）

我们可以通过以下代码诊断来自 POSIX 线程函数的错误：

```
errno = pthread_create(&thread, NULL, func, &arg);
if (errno != 0)
    errExit("pthread_create");
```

然而，这种方法效率较低，因为 *errno* 在线程程序中被定义为一个宏，展开后是一个返回可修改左值的函数调用。因此，每次使用 *errno* 都会导致一次函数调用。*errExitEN()* 函数允许我们编写一个更高效的等效代码：

```
int s;

s = pthread_create(&thread, NULL, func, &arg);
if (s != 0)
    errExitEN(s, "pthread_create");
```

### 注意

在 C 术语中，*lvalue* 是指向存储区域的表达式。最常见的 lvalue 示例是变量的标识符。某些运算符也返回 lvalue。例如，如果 *p* 是指向存储区域的指针，则 **p* 是 lvalue。在 POSIX 线程 API 下，*errno* 被重新定义为返回指向线程特定存储区域的指针的函数（见 线程特定数据）。

为了诊断其他类型的错误，我们使用 *fatal()*、*usageErr()* 和 *cmdLineErr()*。

```
#include "tlpi_hdr.h"

void `fatal`(const char **format*, ...);
void `usageErr`(const char **format*, ...);
void `cmdLineErr`(const char **format*, ...);
```

*fatal()* 函数用于诊断一般错误，包括那些未设置 *errno* 的库函数的错误。它的参数列表与 *printf()* 相同，不同的是，输出字符串自动附加一个换行符。它将格式化的输出打印到标准错误，并像 *errExit()* 一样终止程序。

*usageErr()* 函数用于诊断命令行参数使用中的错误。它采用类似 *printf()* 风格的参数列表，打印字符串 *Usage*：后跟格式化输出到标准错误，然后通过调用 *exit()* 终止程序。（本书中的一些示例程序提供了自己扩展版的 *usageErr()* 函数，命名为 *usageError()*。）

*cmdLineErr()* 函数与 *usageErr()* 类似，但用于诊断指定给程序的命令行参数中的错误。

我们的错误诊断函数的实现如 示例 3-3 所示。

示例 3-3. 所有程序使用的错误处理函数

```
`lib/error_functions.c`
#include <stdarg.h>
#include "error_functions.h"
#include "tlpi_hdr.h"
#include "ename.c.inc"          /* Defines ename and MAX_ENAME */

#ifdef __GNUC__
__attribute__ ((__noreturn__))
#endif
static void
terminate(Boolean useExit3)
{
    char *s;

    /* Dump core if EF_DUMPCORE environment variable is defined and
       is a nonempty string; otherwise call exit(3) or _exit(2),
       depending on the value of 'useExit3'. */

    s = getenv("EF_DUMPCORE");

    if (s != NULL && *s != '\0')
        abort();
    else if (useExit3)
        exit(EXIT_FAILURE);
    else
        _exit(EXIT_FAILURE);
}

static void
outputError(Boolean useErr, int err, Boolean flushStdout,
        const char *format, va_list ap)
{
#define BUF_SIZE 500
    char buf[BUF_SIZE], userMsg[BUF_SIZE], errText[BUF_SIZE];

    vsnprintf(userMsg, BUF_SIZE, format, ap);

    if (useErr)
        snprintf(errText, BUF_SIZE, " [%s %s]",
                (err > 0 && err <= MAX_ENAME) ?
                ename[err] : "?UNKNOWN?", strerror(err));
    else
        snprintf(errText, BUF_SIZE, ":");

    snprintf(buf, BUF_SIZE, "ERROR%s %s\n", errText, userMsg);

    if (flushStdout)
        fflush(stdout);       /* Flush any pending stdout */
    fputs(buf, stderr);
    fflush(stderr);           /* In case stderr is not line-buffered */
}

void
errMsg(const char *format, ...)
{
    va_list argList;
    int savedErrno;

    savedErrno = errno;       /* In case we change it here */

    va_start(argList, format);
    outputError(TRUE, errno, TRUE, format, argList);
    va_end(argList);

    errno = savedErrno;
}

void
errExit(const char *format, ...)
{
    va_list argList;

    va_start(argList, format);
    outputError(TRUE, errno, TRUE, format, argList);
    va_end(argList);

    terminate(TRUE);
}

void
err_exit(const char *format, ...)
{
    va_list argList;

    va_start(argList, format);
    outputError(TRUE, errno, FALSE, format, argList);
    va_end(argList);

    terminate(FALSE);
}

void
errExitEN(int errnum, const char *format, ...)
{
    va_list argList;

    va_start(argList, format);
    outputError(TRUE, errnum, TRUE, format, argList);
    va_end(argList);

    terminate(TRUE);
}

void
fatal(const char *format, ...)
{
    va_list argList;

    va_start(argList, format);
    outputError(FALSE, 0, TRUE, format, argList);
    va_end(argList);

    terminate(TRUE);
}

void
usageErr(const char *format, ...)
{
    va_list argList;

    fflush(stdout);           /* Flush any pending stdout */

    fprintf(stderr, "Usage: ");
    va_start(argList, format);
    vfprintf(stderr, format, argList);
    va_end(argList);

    fflush(stderr);           /* In case stderr is not line-buffered */
    exit(EXIT_FAILURE);
}

void
cmdLineErr(const char *format, ...)
{
    va_list argList;

    fflush(stdout);           /* Flush any pending stdout */

    fprintf(stderr, "Command-line usage error: ");
    va_start(argList, format);
    vfprintf(stderr, format, argList);
    va_end(argList);

    fflush(stderr);           /* In case stderr is not line-buffered */
    exit(EXIT_FAILURE);
}
     `lib/error_functions.c`
```

示例 3-3 中包含的文件 `enames.c.inc` 如 示例 3-4 所示。该文件定义了一个字符串数组 *ename*，它是与每个可能的 *errno* 值对应的符号名称。我们的错误处理函数使用这个数组来打印与特定错误号对应的符号名称。这是一个变通方法，旨在解决以下事实：一方面，*strerror()* 返回的字符串无法识别其错误信息对应的符号常量；另一方面，手册页面用符号名称描述错误。打印符号名称为我们提供了一个简单的方法来查找错误的原因。

### 注意

`ename.c.inc`文件的内容是架构特定的，因为*errno*值在不同的 Linux 硬件架构之间有所不同。示例中显示的版本示例 3-4 适用于 Linux 2.6/x86-32 系统。此文件是使用包含在本书源代码发行版中的脚本（`lib/Build_ename.sh`）构建的。可以使用此脚本构建适用于特定硬件平台和内核版本的`ename.c.inc`版本。

请注意，*ename*数组中的一些字符串为空。这些对应于未使用的错误值。此外，*ename*中的某些字符串由斜杠分隔的两个错误名称组成。这些字符串对应于两个符号错误名称具有相同的数字值的情况。

### 注意

从`ename.c.inc`文件中，我们可以看到`EAGAIN`和`EWOULDBLOCK`错误具有相同的值。（SUSv3 明确允许这种情况，并且这些常量的值在大多数其他 UNIX 系统上是相同的，但不是所有系统都是如此。）这些错误是在系统调用中返回的，通常情况下这些调用会阻塞（即，必须等待才能完成），但调用者请求系统调用返回错误，而不是阻塞。`EAGAIN`起源于 System V，它是执行 I/O、信号量操作、消息队列操作和文件锁定（*fcntl()*)）的系统调用返回的错误。`EWOULDBLOCK`起源于 BSD，它是文件锁定（*flock()*）和与套接字相关的系统调用返回的错误。

在 SUSv3 中，`EWOULDBLOCK`仅在与套接字相关的各种接口的规范中提到。对于这些接口，SUSv3 允许非阻塞调用返回`EAGAIN`或`EWOULDBLOCK`。对于所有其他非阻塞调用，SUSv3 只指定错误`EAGAIN`。

示例 3-4. Linux 错误名称（x86-32 版本）

```
`lib/ename.c.inc`
static char *ename[] = {
    /*   0 */ "",
    /*   1 */ "EPERM", "ENOENT", "ESRCH", "EINTR", "EIO", "ENXIO", "E2BIG",
    /*   8 */ "ENOEXEC", "EBADF", "ECHILD", "EAGAIN/EWOULDBLOCK", "ENOMEM",
    /*  13 */ "EACCES", "EFAULT", "ENOTBLK", "EBUSY", "EEXIST", "EXDEV",
    /*  19 */ "ENODEV", "ENOTDIR", "EISDIR", "EINVAL", "ENFILE", "EMFILE",
    /*  25 */ "ENOTTY", "ETXTBSY", "EFBIG", "ENOSPC", "ESPIPE", "EROFS",
    /*  31 */ "EMLINK", "EPIPE", "EDOM", "ERANGE", "EDEADLK/EDEADLOCK",
    /*  36 */ "ENAMETOOLONG", "ENOLCK", "ENOSYS", "ENOTEMPTY", "ELOOP", "",
    /*  42 */ "ENOMSG", "EIDRM", "ECHRNG", "EL2NSYNC", "EL3HLT", "EL3RST",
    /*  48 */ "ELNRNG", "EUNATCH", "ENOCSI", "EL2HLT", "EBADE", "EBADR",
    /*  54 */ "EXFULL", "ENOANO", "EBADRQC", "EBADSLT", "", "EBFONT", "ENOSTR",
    /*  61 */ "ENODATA", "ETIME", "ENOSR", "ENONET", "ENOPKG", "EREMOTE",
    /*  67 */ "ENOLINK", "EADV", "ESRMNT", "ECOMM", "EPROTO", "EMULTIHOP",
    /*  73 */ "EDOTDOT", "EBADMSG", "EOVERFLOW", "ENOTUNIQ", "EBADFD",
    /*  78 */ "EREMCHG", "ELIBACC", "ELIBBAD", "ELIBSCN", "ELIBMAX",
    /*  83 */ "ELIBEXEC", "EILSEQ", "ERESTART", "ESTRPIPE", "EUSERS",
    /*  88 */ "ENOTSOCK", "EDESTADDRREQ", "EMSGSIZE", "EPROTOTYPE",
    /*  92 */ "ENOPROTOOPT", "EPROTONOSUPPORT", "ESOCKTNOSUPPORT",
    /*  95 */ "EOPNOTSUPP/ENOTSUP", "EPFNOSUPPORT", "EAFNOSUPPORT",
    /*  98 */ "EADDRINUSE", "EADDRNOTAVAIL", "ENETDOWN", "ENETUNREACH",
    /* 102 */ "ENETRESET", "ECONNABORTED", "ECONNRESET", "ENOBUFS", "EISCONN",
    /* 107 */ "ENOTCONN", "ESHUTDOWN", "ETOOMANYREFS", "ETIMEDOUT",
    /* 111 */ "ECONNREFUSED", "EHOSTDOWN", "EHOSTUNREACH", "EALREADY",
    /* 115 */ "EINPROGRESS", "ESTALE", "EUCLEAN", "ENOTNAM", "ENAVAIL",
    /* 120 */ "EISNAM", "EREMOTEIO", "EDQUOT", "ENOMEDIUM", "EMEDIUMTYPE",
    /* 125 */ "ECANCELED", "ENOKEY", "EKEYEXPIRED", "EKEYREVOKED",
    /* 129 */ "EKEYREJECTED", "EOWNERDEAD", "ENOTRECOVERABLE", "ERFKILL"
};

#define MAX_ENAME 132
     `lib/ename.c.inc`
```

#### 用于解析数字命令行参数的函数

示例 3-5 中的头文件提供了我们经常用于解析整数命令行参数的两个函数的声明：*getInt()*和*getLong()*。使用这些函数而不是*atoi()*、*atol()*和*strtol()*的主要优势在于，它们提供了一些基本的数字参数有效性检查。

```
#include "tlpi_hdr.h"

int `getInt`(const char **arg*, int *flags*, const char **name*);
long `getLong`(const char **arg*, int *flags*, const char **name*);
```

### 注意

两者返回*arg*转换为数字形式

*getInt()*和*getLong()*函数分别将*arg*指向的字符串转换为*int*或*long*。如果*arg*不包含有效的整数字符串（即，仅包含数字以及字符`+`和`-`），这些函数将打印错误信息并终止程序。

如果*name*参数非`NULL`，则应包含一个字符串，用于标识*arg*中的参数。这个字符串将作为这些函数显示的任何错误信息的一部分。

*flags* 参数对 *getInt()* 和 *getLong()* 函数的操作提供了一定的控制。默认情况下，这些函数期望包含有符号十进制整数的字符串。通过将一个或多个在示例 3-5 中定义的 `GN_*` 常量通过按位或（|）操作加入 *flags*，我们可以选择不同的进制进行转换，并将数字范围限制为非负数或大于 0 的数值。

*getInt()* 和 *getLong()* 函数的实现见示例 3-6。

### 注

虽然 *flags* 参数允许我们强制执行文中描述的范围检查，但在某些情况下，我们的示例程序中并未请求这些检查，尽管看起来这样做是合理的。例如，在示例 47-1 中，我们没有检查 *init-value* 参数。这意味着用户可以为信号量指定一个负数作为初始值，这将在随后的 *semctl()* 系统调用中导致错误（`ERANGE`），因为信号量不能有负值。在这种情况下省略范围检查，使我们不仅能够实验正确使用系统调用和库函数，还能观察当传入无效参数时会发生什么。现实中的应用程序通常会对其命令行参数进行更严格的检查。

示例 3-5. `get_num.c` 的头文件

```
`lib/get_num.h`
#ifndef GET_NUM_H
#define GET_NUM_H

#define GN_NONNEG       01      /* Value must be >= 0 */
#define GN_GT_0         02      /* Value must be > 0 */

                                /* By default, integers are decimal */
#define GN_ANY_BASE   0100      /* Can use any base - like strtol(3) */
#define GN_BASE_8     0200      /* Value is expressed in octal */
#define GN_BASE_16    0400      /* Value is expressed in hexadecimal */

long getLong(const char *arg, int flags, const char *name);

int getInt(const char *arg, int flags, const char *name);

#endif
      `lib/get_num.h`
```

示例 3-6. 解析数字命令行参数的函数

```
`lib/get_num.c`
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <limits.h>
#include <errno.h>
#include "get_num.h"

static void
gnFail(const char *fname, const char *msg, const char *arg, const char *name)
{
    fprintf(stderr, "%s error", fname);
    if (name != NULL)
        fprintf(stderr, " (in %s)", name);
    fprintf(stderr, ": %s\n", msg);
    if (arg != NULL && *arg != '\0')
        fprintf(stderr, "        offending text: %s\n", arg);

    exit(EXIT_FAILURE);
}

static long
getNum(const char *fname, const char *arg, int flags, const char *name)
{
    long res;
    char *endptr;
    int base;

    if (arg == NULL || *arg == '\0')
        gnFail(fname, "null or empty string", arg, name);

    base = (flags & GN_ANY_BASE) ? 0 : (flags & GN_BASE_8) ? 8 :
                        (flags & GN_BASE_16) ? 16 : 10;

    errno = 0;
    res = strtol(arg, &endptr, base);
    if (errno != 0)
        gnFail(fname, "strtol() failed", arg, name);

    if (*endptr != '\0')
        gnFail(fname, "nonnumeric characters", arg, name);

    if ((flags & GN_NONNEG) && res < 0)
        gnFail(fname, "negative value not allowed", arg, name);

    if ((flags & GN_GT_0) && res <= 0)
        gnFail(fname, "value must be > 0", arg, name);

    return res;
}

long
getLong(const char *arg, int flags, const char *name)
{
    return getNum("getLong", arg, flags, name);
}

int
getInt(const char *arg, int flags, const char *name)
{
    long res;

    res = getNum("getInt", arg, flags, name);

    if (res > INT_MAX || res < INT_MIN)
        gnFail("getInt", "integer out of range", arg, name);

    return (int) res;
}
     `lib/get_num.c`
```

## 可移植性问题

在本节中，我们讨论编写可移植系统程序的话题。我们介绍了特性测试宏和 SUSv3 定义的标准系统数据类型，然后探讨了一些其他的可移植性问题。

### 特性测试宏

各种标准规范着系统调用和库函数 API 的行为（参见标准化）。其中一些标准由标准组织定义，如开放组（Single UNIX Specification），而其他一些则由两个具有历史意义的 UNIX 实现定义：BSD 和 System V Release 4（以及相关的 System V 接口定义）。

有时，在编写可移植的应用程序时，我们可能希望各个头文件仅暴露遵循特定标准的定义（常量、函数原型等）。为此，我们在编译程序时定义一个或多个下列的*特性测试宏*。我们可以通过在程序源代码中包含任何头文件之前，定义该宏来实现这一点：

```
#define _BSD_SOURCE 1
```

或者，我们可以使用 C 编译器的 *-D* 选项：

```
`$ cc -D_BSD_SOURCE prog.c`
```

### 注

*功能测试宏*（feature test macro）这个术语可能令人困惑，但如果从实现的角度来看，它就能解释清楚。实现决定应暴露每个头文件中哪些可用的*功能*，通过*测试*（使用`#if`）应用程序为这些*宏*定义了哪些值。

以下功能测试宏是由相关标准指定的，因此它们的使用是便携的，适用于所有支持这些标准的系统：

`_POSIX_SOURCE`

如果定义（无论值为何），则暴露符合 POSIX.1-1990 和 ISO C（1990）定义的内容。此宏已被`_POSIX_C_SOURCE`所取代。

`_POSIX_C_SOURCE`

如果定义且值为 1，则具有与`_POSIX_SOURCE`相同的效果。如果定义且值大于或等于 199309，则还会暴露 POSIX.1b（实时）定义。如果定义且值大于或等于 199506，则还会暴露 POSIX.1c（线程）定义。如果定义且值为 200112，则还会暴露 POSIX.1-2001 基础规范的定义（即，排除 XSI 扩展）。（在版本 2.3.3 之前，*glibc*头文件不会解释`_POSIX_C_SOURCE`的值 200112。）如果定义且值为 200809，则还会暴露 POSIX.1-2008 基础规范的定义。（在版本 2.10 之前，*glibc*头文件不会解释`_POSIX_C_SOURCE`的值 200809。）

`_XOPEN_SOURCE`

如果定义（无论值为何），则暴露 POSIX.1、POSIX.2 和 X/Open（XPG4）定义。如果值为 500 或更大，还会暴露 SUSv2（UNIX 98 和 XPG5）扩展。将值设置为 600 或更大，还会暴露 SUSv3 XSI（UNIX 03）扩展和 C99 扩展。（在版本 2.2 之前，*glibc*头文件不会解释`_XOPEN_SOURCE`的值 600。）将值设置为 700 或更大，还会暴露 SUSv4 XSI 扩展。（在版本 2.10 之前，*glibc*头文件不会解释`_XOPEN_SOURCE`的值 700。）`_XOPEN_SOURCE`的值 500、600 和 700 的选择是因为 SUSv2、SUSv3 和 SUSv4 分别是 X/Open 规范的第 5、6 和 7 个问题。

以下功能测试宏是*glibc*特有的：

`_BSD_SOURCE`

如果定义（无论值为何），则暴露 BSD 定义。定义此宏也会将`_POSIX_C_SOURCE`定义为 199506 的值。仅显式设置此宏会在标准冲突的情况下，优先使用 BSD 定义。

`_SVID_SOURCE`

如果定义（无论值为何），则暴露 System V 接口定义（SVID）定义。

`_GNU_SOURCE`

如果定义（无论值为何），则暴露通过设置所有前述宏所提供的所有定义，以及各种 GNU 扩展。

当不带特殊选项调用 GNU C 编译器时，默认会定义`_POSIX_SOURCE`、`_POSIX_C_SOURCE=200809`（对于*glibc*版本 2.5 到 2.9 为 200112，或者对于*glibc*版本低于 2.4 为 199506）、`_BSD_SOURCE`和`_SVID_SOURCE`。

如果定义了单独的宏，或者以其标准模式之一调用编译器（例如，*cc -ansi* 或 *cc -std=c99*），则只会提供所请求的定义。有一个例外：如果未定义`_POSIX_C_SOURCE`，并且编译器未以其标准模式之一调用，则会将`_POSIX_C_SOURCE`定义为值 200809（对于*glibc*版本 2.4 到 2.9 为 200112，或者对于早于 2.4 版本的*glibc*为 199506）。

定义多个宏是累加的，因此我们可以例如使用以下*cc*命令显式选择与默认提供的宏设置相同的设置：

```
`$ cc -D_POSIX_SOURCE -D_POSIX_C_SOURCE=199506 \`
                                           `-D_BSD_SOURCE -D_SVID_SOURCE prog.c`
```

`<features.h>`头文件和*feature_test_macros(7)*手册页提供了关于每个功能测试宏具体分配了哪些值的进一步信息。

#### `_POSIX_C_SOURCE`，`_XOPEN_SOURCE` 和 POSIX.1/SUS

只有`_POSIX_C_SOURCE`和`_XOPEN_SOURCE`功能测试宏在 POSIX.1-2001/SUSv3 中指定，要求在符合要求的应用程序中，这些宏必须分别定义为 200112 和 600。将`_POSIX_C_SOURCE`定义为 200112 提供了对 POSIX.1-2001 基础规范的符合性（即，*POSIX 符合性*，不包括 XSI 扩展）。将`_XOPEN_SOURCE`定义为 600 提供了对 SUSv3 的符合性（即，*XSI 符合性*，基础规范加上 XSI 扩展）。对于 POSIX.1-2008/SUSv4 也适用类似的规定，要求这两个宏的值分别定义为 200809 和 700。

SUSv3 规定，设置`_XOPEN_SOURCE`为 600 应当提供所有通过设置`_POSIX_C_SOURCE`为 200112 启用的功能。因此，应用程序只需要定义`_XOPEN_SOURCE`以符合 SUSv3（即 XSI）规范。SUSv4 做出类似规定，设置`_XOPEN_SOURCE`为 700 应当提供所有通过设置`_POSIX_C_SOURCE`为 200809 启用的功能。

#### 函数原型和源代码示例中的功能测试宏

手册页描述了必须定义哪些功能测试宏，以便从头文件中使特定的常量定义或函数声明可见。

本书中的所有源代码示例都编写为可以使用默认的 GNU C 编译器选项或以下选项编译：

```
`$ cc -std=c99 -D_XOPEN_SOURCE=600`
```

本书中显示的每个函数的原型都表示必须定义的功能测试宏，以便在使用默认编译器选项或刚才显示的*cc*命令选项编译的程序中使用该函数。手册页提供了关于每个函数声明所需的功能测试宏的更精确描述。

### 系统数据类型

各种实现数据类型使用标准 C 类型表示，例如进程 ID、用户 ID 和文件偏移量。尽管可以使用 C 基本类型如 *int* 和 *long* 来声明存储这些信息的变量，但这样做会降低在 UNIX 系统间的可移植性，原因如下：

+   这些基本类型的大小在不同的 UNIX 实现中有所不同（例如，*long* 类型在一个系统上可能是 4 字节，而在另一个系统上是 8 字节），有时甚至在同一个实现的不同编译环境中也会不同。此外，不同的实现可能会使用不同的类型来表示相同的信息。例如，一个进程 ID 在某个系统上可能是 *int* 类型，而在另一个系统上可能是 *long* 类型。

+   即使在单一的 UNIX 实现中，用于表示信息的类型也可能在不同的实现版本之间有所不同。在 Linux 上，用户和组 ID 就是一个显著的例子。在 Linux 2.2 及以前版本中，这些值用 16 位表示。而在 Linux 2.4 及以后的版本中，它们是 32 位值。

为了避免这种可移植性问题，SUSv3 指定了各种标准系统数据类型，并要求实现定义并正确使用这些类型。每种类型都是通过 C 的 `typedef` 特性定义的。例如，*pid_t* 数据类型用于表示进程 ID，在 Linux/x86-32 上，这种类型定义如下：

```
typedef int pid_t;
```

大多数标准系统数据类型的名称以*_t*结尾。它们中的许多在头文件`<sys/types.h>`中声明，尽管也有一些定义在其他头文件中。

应用程序应使用这些类型定义来便捷地声明它使用的变量。例如，以下声明将允许应用程序在任何符合 SUSv3 标准的系统上正确表示进程 ID：

```
pid_t mypid;
```

表 3-1 列出了本书中我们将遇到的一些系统数据类型。对于表中的某些类型，SUSv3 要求该类型实现为*算术类型*。这意味着实现可以选择将基础类型定义为整数类型或浮点类型（实数或复数）。

表 3-1. 选定的系统数据类型

| 数据类型 | SUSv3 类型要求 | 描述 |
| --- | --- | --- |
| *blkcnt_t* | 有符号整数 | 文件块计数（检索文件信息：*stat()*")） |
| *blksize_t* | 有符号整数 | 文件块大小（检索文件信息：*stat()*")） |
| *cc_t* | 无符号整数 | 终端特殊字符（终端特殊字符） |
| *clock_t* | 整数或实数浮点数 | 系统时间（以时钟滴答表示）（进程时间） |
| *clockid_t* | 一种算术类型 | POSIX.1b 时钟和定时器函数的时钟标识符（POSIX 区间定时器） |
| *comp_t* | 不在 SUSv3 中 | 压缩时钟滴答（进程记账） |
| *dev_t* | 一种算术类型 | 设备号，由主设备号和次设备号组成（检索文件信息: *stat()*")) |
| *DIR* | 无类型要求 | 目录流（读取目录: *opendir()* 和 *readdir()* 和 readdir()")) |
| *fd_set* | 结构类型 | 用于 *select()* 的文件描述符集（*select()* 系统调用 系统调用")） |
| *fsblkcnt_t* | 无符号整数 | 文件系统块计数（获取文件系统信息: *statvfs()*")) |
| *fsfilcnt_t* | 无符号整数 | 文件计数（获取文件系统信息: *statvfs()*")) |
| *gid_t* | 整数 | 数字组标识符（组文件: `/etc/group`） |
| *id_t* | 整数 | 用于存储标识符的通用类型；足够大，可以至少存储 *pid_t*、*uid_t* 和 *gid_t* |
| *in_addr_t* | 32 位无符号整数 | IPv4 地址（互联网套接字地址) |
| *in_port_t* | 16 位无符号整数 | IP 端口号（互联网套接字地址) |
| *ino_t* | 无符号整数 | 文件 i-node 编号（检索文件信息: *stat()*")) |
| *key_t* | 一种算术类型 | System V IPC 键（IPC 键） |
| *mode_t* | 整数 | 文件权限和类型（检索文件信息: *stat()*")) |
| *mqd_t* | 无类型要求，但不能是数组类型 | POSIX 消息队列描述符 |
| *msglen_t* | 无符号整数 | System V 消息队列中允许的字节数（消息队列相关数据结构） |
| *msgqnum_t* | 无符号整数 | System V 消息队列中的消息计数（消息队列相关数据结构） |
| *nfds_t* | 无符号整数 | *poll()*的文件描述符数量（*poll()* 系统调用 System Call")) |
| *nlink_t* | 整数 | 文件的（硬）链接计数（检索文件信息: *stat()*")) |
| *off_t* | 有符号整数 | 文件偏移量或大小（改变文件偏移量: *lseek()*") 和 检索文件信息: *stat()*")) |
| *pid_t* | 有符号整数 | 进程 ID、进程组 ID 或会话 ID（进程 ID 与父进程 ID，进程组 和 会话） |
| *ptrdiff_t* | 有符号整数 | 两个指针值之间的差异，作为有符号整数 |
| *rlim_t* | 无符号整数 | 资源限制（进程资源限制） |
| *sa_family_t* | 无符号整数 | 套接字地址族（通用套接字地址结构: *struct sockaddr*） |
| *shmatt_t* | 无符号整数 | 系统 V 共享内存段的附加进程计数（共享内存相关数据结构） |
| *sig_atomic_t* | 整数 | 可以原子访问的数据类型（全局变量和 *sig_atomic_t* 数据类型) |
| *siginfo_t* | 结构体类型 | 信号来源信息（`SA_SIGINFO` 标志) |
| *sigset_t* | 整数或结构体类型 | 信号集（信号集） |
| *size_t* | 无符号整数 | 对象的字节大小 |
| *socklen_t* | 至少 32 位的整数类型 | 套接字地址结构体的字节大小（将套接字绑定到地址: *bind()*")) |
| *speed_t* | 无符号整数 | 终端行速（终端行速（比特率）")) |
| *ssize_t* | 有符号整数 | 字节计数或（负数）错误指示 |
| *stack_t* | 结构体类型 | 描述备用信号栈的结构体（处理备用栈上的信号: *sigaltstack()*")) |
| *suseconds_t* | 有符号整数，允许的范围是[-1, 1000000] | 微秒时间间隔（日历时间） |
| *tcflag_t* | 无符号整数 | 终端模式标志位掩码（检索和修改终端属性） |
| *time_t* | 整数或实数浮动类型 | 自纪元以来的秒数（日历时间） |
| *timer_t* | 算术类型 | POSIX.1b 区间定时器函数的定时器标识符（POSIX 区间定时器） |
| *uid_t* | 整数 | 数字用户标识符（密码文件：`/etc/passwd`） |

在后续章节中讨论表 3-1 中的数据类型时，我们经常会提到某个类型“是一个整数类型[由 SUSv3 规定]”。这意味着 SUSv3 要求该类型定义为整数类型，但不要求使用特定的本地整数类型（例如，*short*、*int*或*long*）。(通常我们不会说 Linux 中实际使用哪种特定的本地数据类型来表示每个系统数据类型，因为一个可移植的应用程序应该编写得不关心使用了哪种数据类型。)

#### 打印系统数据类型的值

当打印表 3-1 中所示的某个数值系统数据类型的值时（例如，*pid_t*和*uid_t*），我们必须小心不要在*printf()*调用中包含表示依赖性。表示依赖性可能发生，因为 C 语言的参数提升规则会将*short*类型的值转换为*int*类型，但保持*int*和*long*类型的值不变。这意味着，根据系统数据类型的定义，可能会在*printf()*调用中传递*int*或*long*。然而，由于*printf()*无法在运行时确定其参数的类型，调用者必须明确地使用`%d`或`%ld`格式说明符来提供这些信息。问题是，仅仅在*printf()*调用中编码这些说明符之一就会产生实现依赖性。通常的解决方法是使用`%ld`说明符，并始终将相应的值转换为*long*类型，如下所示：

```
pid_t mypid;

mypid = getpid();           /* Returns process ID of calling process */
printf("My PID is %ld\n", (long) mypid);
```

我们对上述技巧做一个例外处理。由于在某些编译环境中，*off_t*数据类型是*long long*的大小，因此我们将*off_t*值转换为该类型，并使用`%lld`说明符，如在大文件 I/O 中所述。

### 注意

C99 标准为*printf()*定义了`z`长度修饰符，用于指示随后的整数转换对应于*size_t*或*ssize_t*类型。因此，我们可以写`%zd`，而不必使用`%ld`加上强制类型转换。尽管这个修饰符在*glibc*中可用，但我们避免使用它，因为并非所有 UNIX 实现都支持它。

C99 标准还定义了`j`长度修饰符，表示相应的参数是*intmax_t*（或*uintmax_t*）类型，这是一种足够大以能够表示任何类型整数的整数类型。因此，使用`%jd`替代`%ld`加上强制转换应该是打印数字系统数据类型值的最佳方式，因为前者也能处理*long long*值和任何扩展的整数类型，如*int128_t*。然而，（再次强调）我们避免使用这一技术，因为并非所有 UNIX 实现都支持。

### 杂项移植性问题

本节中，我们讨论了编写系统程序时可能遇到的其他移植性问题。

#### 初始化和使用结构

每个 UNIX 实现都规定了一系列标准结构，这些结构在各种系统调用和库函数中使用。举个例子，考虑*sembuf*结构，它用于表示*semop()*系统调用中要执行的信号量操作：

```
struct sembuf {
    unsigned short sem_num;         /* Semaphore number */
    short          sem_op;          /* Operation to be performed */
    short          sem_flg;         /* Operation flags */
};
```

尽管 SUSv3 规定了像*sembuf*这样的结构，但需要注意以下几点：

+   一般来说，这种结构内字段的定义顺序是没有规定的。

+   在某些情况下，这些结构中可能包含额外的特定于实现的字段。

因此，使用如下结构初始化器是不可移植的：

```
struct sembuf s = { 3, -1, SEM_UNDO };
```

尽管这个初始化器在 Linux 上有效，但在另一个实现中，由于*sembuf*结构中的字段顺序不同，它将不起作用。为了便携地初始化这些结构，我们必须使用显式赋值语句，如下所示：

```
struct sembuf s;

s.sem_num = 3;
s.sem_op  = -1;
s.sem_flg = SEM_UNDO;
```

如果我们使用 C99 标准，那么我们可以采用该语言的新语法进行结构初始化，以编写等效的初始化代码：

```
struct sembuf s = { .sem_num = 3, .sem_op = -1, .sem_flg = SEM_UNDO };
```

如果我们希望将标准结构的内容写入文件，关于标准结构成员顺序的注意事项同样适用。为了做到便携性，我们不能简单地对结构进行二进制写入。相反，结构字段必须按照指定的顺序逐个写入（可能是文本形式）。

#### 使用可能在所有实现中都不存在的宏

在某些情况下，某些宏在所有 UNIX 实现中可能没有定义。例如，`WCOREDUMP()`宏（用于检查子进程是否生成了核心转储文件）是广泛可用的，但它并没有在 SUSv3 中指定。因此，这个宏在某些 UNIX 实现中可能不存在。为了可移植地处理这种可能性，我们可以使用 C 预处理器的`#ifdef`指令，如下面的示例所示：

```
#ifdef WCOREDUMP
    /* Use WCOREDUMP() macro */
#endif
```

#### 跨实现的所需头文件差异

在某些情况下，所需的头文件在不同 UNIX 实现中可能有所不同，用于原型化各种系统调用和库函数。在本书中，我们展示了 Linux 上的要求，并注明了与 SUSv3 的任何差异。

本书中的一些函数概要显示了一个特定的头文件，并附有注释*/* 为了可移植性 */*。这表示该头文件在 Linux 上或 SUSv3 中并不是必需的，但由于一些其他（尤其是旧版）实现可能需要它，因此在可移植程序中应该包含它。

### 注意

对于它指定的许多函数，POSIX.1-1990 要求在包含与该函数相关的任何其他头文件之前，必须包含头文件`<sys/types.h>`。然而，这一要求是多余的，因为大多数现代 UNIX 实现并不要求应用程序为这些函数包含该头文件。因此，SUSv1 去除了这一要求。然而，在编写可移植程序时，最好将其作为最先包含的头文件之一。（但是，我们在示例程序中省略了该头文件，因为在 Linux 上并不需要，省略它可以使示例程序少一行。）

## 总结

系统调用允许进程请求内核提供服务。即使是最简单的系统调用，相较于用户空间的函数调用也有显著的开销，因为系统必须暂时切换到内核模式以执行系统调用，并且内核必须验证系统调用参数并在用户内存与内核内存之间传输数据。

标准 C 库提供了许多执行各种任务的库函数。有些库函数使用系统调用来完成它们的工作；而有些则完全在用户空间内执行任务。在 Linux 上，通常使用的标准 C 库实现是*glibc*。

大多数系统调用和库函数都会返回一个状态，指示调用是否成功或失败。此类状态返回值应始终进行检查。

我们介绍了若干个在本书示例程序中使用的函数。这些函数执行的任务包括诊断错误和解析命令行参数。

我们讨论了各种准则和技巧，这些方法可以帮助我们编写可在任何符合标准的系统上运行的可移植系统程序。

在编译应用程序时，我们可以定义各种特性测试宏来控制头文件中暴露的定义。如果我们希望确保程序符合某些正式或实现定义的标准，这非常有用。

我们可以通过使用各种标准中定义的系统数据类型，而不是使用原生 C 类型，来提高系统程序的可移植性。SUSv3 指定了实现应该支持、应用程序应该使用的广泛系统数据类型。

## 练习

1.  在使用 Linux 特有的 *reboot()* 系统调用重启系统时，第二个参数 *magic2* 必须指定为一组魔数中的一个（例如，`LINUX_REBOOT_MAGIC2`）。这些数字的意义是什么？（将它们转换为十六进制可以提供线索。）
