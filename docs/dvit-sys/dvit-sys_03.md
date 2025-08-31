# 第四章：C 调试工具

![image](img/common.jpg)

在本节中，我们介绍了两个调试工具：GNU 调试器（GDB），^(1)它有助于检查程序的运行时状态，和 Valgrind^(2)（发音为“Val-grinned”），一个流行的代码分析工具套件。具体来说，我们介绍了 Valgrind 的 Memcheck 工具，^(3)它分析程序的内存访问情况，以检测无效内存使用、未初始化内存使用和内存泄漏。

GDB 部分包括两个示范 GDB 会话，展示了常用的 GDB 命令，用于查找程序中的错误。我们还讨论了一些高级 GDB 功能，包括将 GDB 附加到正在运行的进程、GDB 与 Makefile 的结合、GDB 中的信号控制、在汇编级别调试，以及调试多线程 Pthreads 程序。

Valgrind 部分讨论了内存访问错误及其为何如此难以检测。它还包括对一个存在错误内存访问的程序执行 Memcheck 的示例。Valgrind 套件包括其他程序分析和调试工具，我们将在后续章节中介绍。例如，我们将在第十一章的《缓存分析与 Valgrind》一节中介绍缓存分析工具 Cachegrind^(4)，以及在第十二章的《使用 Callgrind 进行分析》一节中介绍函数调用分析工具 Callgrind^(5)。

### 3.1 使用 GDB 调试

GDB 可以帮助程序员发现并修复程序中的错误。GDB 支持多种编程语言的编译程序，但我们这里主要关注 C 语言。调试器是一个控制另一个程序（即被调试程序）执行的程序——它允许程序员在程序运行时看到程序的行为。使用调试器可以帮助程序员发现错误并找出错误的原因。以下是 GDB 可以执行的一些有用操作：

+   启动程序并逐行调试

+   当程序执行到代码中的某些位置时暂停其执行

+   在用户指定的条件下暂停程序执行

+   显示程序在暂停执行时的变量值

+   在暂停后继续程序的执行

+   检查程序在崩溃时的执行状态

+   检查调用栈中任何栈帧的内容

GDB 用户通常会在程序中设置*断点*。断点指定了程序中的某个位置，GDB 将在此位置暂停程序的执行。当执行中的程序达到断点时，GDB 会暂停程序的执行，并允许用户输入 GDB 命令来检查程序变量和栈内容，逐行执行程序，添加新的断点，并继续执行程序直到达到下一个断点。

许多 Unix 系统还提供了数据展示调试器（DDD），这是一个易于使用的图形化界面（GUI）程序，它将命令行调试器（例如 GDB）包装成图形界面。DDD 程序接受与 GDB 相同的参数和命令，但它提供了图形界面以及 GDB 的命令行接口。

在讨论了如何开始使用 GDB 的一些基本内容后，我们展示了两个示例 GDB 调试会话，介绍了在寻找不同类型的 bug 时常用的 GDB 命令。第一个会话，“使用 GDB 调试程序示例（badprog.c）”在第 152 页上，展示了如何使用 GDB 命令来寻找 C 程序中的逻辑错误。第二个会话，“使用 GDB 调试崩溃程序示例（segfaulter.c）”在第 159 页上，展示了如何使用 GDB 命令检查程序崩溃时的执行状态，以找出崩溃的原因。

在第 161 页的“常用 GDB 命令”部分，我们更详细地描述了常用的 GDB 命令，展示了更多命令的示例。在后续章节中，我们将讨论一些高级的 GDB 功能。

#### 3.1.1 开始使用 GDB

在调试程序时，建议使用 `-g` 选项进行编译，这会将额外的调试信息添加到二进制可执行文件中。这些额外的信息帮助调试器在二进制可执行文件中找到程序的变量和函数，并使它能够将机器代码指令映射到 C 源代码的行号（这是 C 程序员能够理解的程序形式）。此外，在进行调试编译时，避免使用编译器优化（例如，不要使用 `-O2` 编译）。编译器优化后的代码通常很难调试，因为优化后的机器代码序列往往不能清晰地映射回 C 源代码。尽管我们在后续部分会讲解 `-g` 标志的使用，某些用户可能会发现使用 `-g3` 标志能获得更好的调试效果，它会提供更多的调试信息。

这是一个示例的 `gcc` 命令，它将构建一个适合用于 GDB 调试的可执行文件：

```
$ gcc -g myprog.c
```

要启动 GDB，可以在可执行文件上调用它。例如：

```
$ gdb a.out

(gdb)          # the gdb command prompt
```

当 GDB 启动时，它会打印 `(gdb)` 提示符，允许用户在运行 `a.out` 程序之前输入 GDB 命令（例如设置断点）。

同样，要在可执行文件上调用 DDD：

```
$ ddd a.out
```

有时，当程序因错误终止时，操作系统会生成一个核心文件，其中包含程序崩溃时的状态信息。可以通过在 GDB 中运行该核心文件和生成该文件的可执行文件来检查其内容：

```
$ gdb core a.out

(gdb) where       # the where command shows point of crash
```

#### 3.1.2 示例 GDB 会话

我们通过两个示例会话演示 GDB 的常见功能，第一例是使用 GDB 查找并修复程序中的两个 bug，第二例是使用 GDB 调试一个崩溃的程序。这两个示例会话中，我们展示的 GDB 命令集包括下表中列出的命令。

| **命令** | **描述** |
| --- | --- |
| `break` | 设置一个断点 |
| `run` | 从头开始启动程序 |
| `cont` | 继续执行程序，直到它命中一个断点 |
| `quit` | 退出 GDB 会话 |
| `next` | 允许程序执行下一行 C 代码，然后暂停 |
| `step` | 允许程序执行下一行 C 代码；如果下一行 |
|  | 包含一个函数调用，进入函数并暂停 |
| `list` | 列出暂停点附近或指定位置的 C 源代码 |
| `print` | 打印程序变量（或表达式）的值 |
| `where` | 打印调用栈 |
| `frame` | 进入特定栈帧的上下文 |

##### 使用 GDB 调试程序示例（badprog.c）

第一个示例 GDB 会话调试的是 `badprog.c` 程序。这个程序的目的是在一个 `int` 类型数组中找到最大值。然而，当程序运行时，它错误地认为 17 是数组中的最大值，而不是正确的最大值 60。这个示例展示了如何使用 GDB 来检查程序的运行时状态，以确定程序为何没有计算出预期的结果。特别地，这个调试会话揭示了两个 bug：

1. 循环边界的错误，导致程序访问数组边界之外的元素。

2. 函数未返回正确的值给调用者的错误。

要使用 GDB 检查程序，首先使用 `-g` 编译程序，以将调试信息添加到可执行文件中：

```
$ gcc -g badprog.c
```

接下来，运行 GDB 对二进制可执行程序（`a.out`）进行调试。GDB 初始化并打印 `(gdb)` 提示符，用户可以在此处输入 GDB 命令：

```
$ gdb ./a.out

GNU gdb (Ubuntu 8.1-0ubuntu3) 8.1.0.20180409-git

Copyright (C) 2018 Free Software Foundation, Inc.

  ...

(gdb)
```

此时，GDB 尚未开始运行程序。一个常见的调试步骤是在 `main()` 函数中设置一个断点，以在程序执行 `main()` 函数中的第一条指令之前暂停程序的执行。`break` 命令将在指定的位置设置一个“断点”（暂停程序），在此例中就是在 `main()` 函数的开始处：

```
(gdb) break main

Breakpoint 1 at 0x8048436: file badprog.c, line 36.
```

`run` 命令告诉 GDB 启动程序：

```
(gdb) run

Starting program: ./a.out
```

如果程序接受命令行参数，可以在 `run` 命令后提供这些参数（例如，运行 `100 200` 将以命令行参数 `100` 和 `200` 启动 `a.out`）。

在输入`run`后，GDB 从程序的开头开始执行，并一直执行到遇到断点为止。到达断点后，GDB 在执行断点所在的代码行之前暂停程序，并打印出与断点相关的断点编号和源代码行。在这个例子中，GDB 在执行程序的第 36 行之前暂停程序。然后，它会打印出 `(gdb)` 提示符，等待进一步的指令：

```
Breakpoint 1, main (argc=1, argv=0x7fffffffe398) at badprog.c:36

36     int main(int argc, char *argv[]) {

(gdb)
```

当程序在断点处暂停时，用户通常希望查看断点周围的 C 源代码。GDB 的`list`命令显示断点周围的代码：

```
(gdb) list

29	    }

30	    return 0;

31	}

32

33	/***************************************/

34	int main(int argc, char *argv[]) {

35

36	    int arr[5] = { 17, 21, 44, 2, 60 };

37

38	    int max = arr[0];
```

随后的`list`命令调用会显示这些代码之后的下一行源代码。`list`也可以与特定的行号一起使用（例如，`list 11`）或与函数名一起使用，以列出程序中指定部分的源代码。例如：

```
(gdb) list findAndReturnMax

12  *  array: array of integer values

13  *  len: size of the array

 14  *  max: set to the largest value in the array

15  *   returns: 0 on success and non-zero on an error

16  */

17 int findAndReturnMax(int *array1, int len, int max) {

18

19     int i;

20

21     if (!array1 || (len <=0) ) {
```

用户可能希望在击中断点后逐行执行代码，在每行执行后检查程序状态。GDB 的`next`命令仅执行下一行 C 代码。程序执行该行代码后，GDB 会再次暂停程序。`print`命令用于打印程序变量的值。以下是一些`next`和`print`命令的调用，展示它们对接下来的两行执行的影响。请注意，`next`后的源代码行尚未执行——它显示的是程序暂停的地方，代表着下一行将被执行的地方：

```
(gdb) next

36   int arr[5] = { 17, 21, 44, 2, 60 };

(gdb) next

38   int max = arr[0];

(gdb) print max

$3 = 0

(gdb) print arr[3]

$4 = 2

(gdb) next

40   if ( findAndReturnMax(arr, 5, max) != 0 ) {

(gdb) print max

$5 = 17

(gdb)
```

在程序执行的这个时刻，主函数已经初始化了它的局部变量`arr`和`max`，并即将调用`findAnd` `ReturnMax()`函数。GDB 的`next`命令执行下一行完整的 C 源代码。如果该行包含函数调用，那么该函数的完整执行及返回会作为单个`next`命令的一部分执行。希望观察函数执行过程的用户应该使用 GDB 的`step`命令，而不是`next`命令：`step`会进入函数调用，在执行函数的第一行代码之前暂停程序。

因为我们怀疑程序中的 bug 与`findAnd` `ReturnMax()`函数相关，所以我们希望进入该函数的执行，而不是跳过它。因此，在第 40 行暂停时，`step`命令将使程序在`findAndReturnMax()`的开始处暂停（另外，用户也可以在`findAndReturnMax()`设置断点，来在该点暂停程序的执行）：

```
(gdb) next

40   if ( findAndReturnMax(arr, 5, max) != 0 ) {

(gdb) step

findAndReturnMax (array1=0x7fffffffe290, len=5, max=17) at badprog.c:21

21   if (!array1 || (len <=0) ) {

(gdb)
```

程序此时暂停在`findAndReturnMax`函数内部，该函数的局部变量和参数现在处于作用域内。`print`命令显示它们的值，`list`命令显示暂停点周围的 C 源代码：

```
(gdb) print array1[0]

$6 = 17

(gdb) print max

$7 = 17

(gdb) list

16  */

17 int findAndReturnMax(int *array1, int len, int max) {

18

19     int i;

20

21     if (!array1 || (len <=0) ) {

22         return -1;

23     }

24     max = array1[0];

25     for (i=1; i <= len; i++) {

(gdb) list

26         if(max < array1[i]) {

27             max = array1[i];

28         }

29     }

30     return 0;

31 }

32

33 /***************************************/

34 int main(int argc, char *argv[]) {

35
```

因为我们认为这个函数可能有 bug，所以我们可能想在函数内部设置一个断点，以便我们可以在执行过程中检查运行时状态。特别是，设置一个在 `max` 发生变化时的断点，可能有助于我们看到这个函数在做什么。

我们可以在程序中的特定行号（第 27 行）设置断点，并使用 `cont` 命令告诉 GDB 继续执行应用程序，直到遇到下一个断点。只有当程序遇到断点时，GDB 才会暂停程序并重新控制它，允许用户输入其他 GDB 命令。

```
(gdb) break 27

Breakpoint 2 at 0x555555554789: file badprog.c, line 27.

(gdb) cont

Continuing.

Breakpoint 2, findAndReturnMax (array1=0x...e290,len=5,max=17) at badprog.c:27

27       max = array1[i];

(gdb) print max

$10 = 17

(gdb) print i

$11 = 1
```

`display` 命令要求 GDB 在每次遇到断点时自动打印出相同的一组程序变量。例如，我们将在每次程序遇到断点时显示 `i`、`max` 和 `array1[i]` 的值（在 `findAndReturnMax()` 中的每次循环迭代时）：

```
(gdb) display i

1: i = 1

(gdb) display max

2: max = 17

(gdb) display array1[i]

3: array1[i] = 21

(gdb) cont

Continuing.

Breakpoint 2, findAndReturnMax (array1=0x7fffffffe290, len=5, max=21)

    at badprog.c:27

27       max = array1[i];

1: i = 2

2: max = 21

3: array1[i] = 44

(gdb) cont

Continuing.

Breakpoint 2, findAndReturnMax (array1=0x7fffffffe290, len=5, max=21)

    at badprog.c:27

27       max = array1[i];

1: i = 3

2: max = 44

3: array1[i] = 2

(gdb) cont
 Breakpoint 2, findAndReturnMax (array1=0x7fffffffe290, len=5, max=44)

    at badprog.c:27

27       max = array1[i];

1: i = 4

2: max = 44

3: array1[i] = 60

(gdb) cont

Breakpoint 2, findAndReturnMax (array1=0x7fffffffe290, len=5, max=60)

    at badprog.c:27

27       max = array1[i];

1: i = 5

2: max = 60

3: array1[i] = 32767

(gdb)
```

我们发现了第一个 bug！`array1[i]` 的值是 32767，这是一个不在传入数组中的值，而 `i` 的值是 5，但 5 不是这个数组的有效索引。通过 GDB，我们发现 `for` 循环的边界需要修正为 `i < len`。

到了这个时候，我们可以退出 GDB 会话并修复代码中的这个 bug。要退出 GDB 会话，输入 quit：

```
(gdb) quit

The program is running.  Exit anyway? (y or n) y

$
```

修复了这个 bug 后，重新编译并运行程序，它仍然没有找到正确的最大值（它仍然认为 17 是最大值而不是 60）。根据我们之前的 GDB 调试，我们可能怀疑在调用或从 `findAndReturnMax()` 函数返回时存在错误。我们重新在 GDB 中运行程序，并这次在 `findAndReturnMax()` 函数入口处设置了断点：

```
$ gdb ./a.out

...

(gdb) break main

Breakpoint 1 at 0x7c4: file badprog.c, line 36.

(gdb) break findAndReturnMax

Breakpoint 2 at 0x748: file badprog.c, line 21.

(gdb) run

Starting program: ./a.out

Breakpoint 1, main (argc=1, argv=0x7fffffffe398) at badprog.c:36

36   int main(int argc, char *argv[]) {

(gdb) cont

 Continuing.

Breakpoint 2, findAndReturnMax (array1=0x7fffffffe290, len=5, max=17)

    at badprog.c:21

21   if (!array1 || (len <=0) ) {

(gdb)
```

如果我们怀疑函数的参数或返回值有 bug，检查堆栈内容可能会很有帮助。`where`（或 `bt`，即“回溯”）GDB 命令会打印堆栈的当前状态。在这个例子中，`main()` 函数位于堆栈的底部（在帧 1 中），并且正在执行对 `findAndReturnMax()` 的调用（第 40 行）。`findAndReturnMax()` 函数位于堆栈的顶部（在帧 0 中），并且当前暂停在第 21 行：

```
(gdb) where

#0  findAndReturnMax (array1=0x7fffffffe290, len=5, max=17) at badprog.c:21

#1  0x0000555555554810 in main (argc=1, argv=0x7fffffffe398) at badprog.c:40
```

GDB 的 `frame` 命令可以让我们进入堆栈中任何一个帧的上下文。在每个堆栈帧的上下文中，用户可以检查该帧的局部变量和参数。在这个例子中，我们进入了堆栈帧 1（调用者的上下文），并打印出 `main()` 函数传递给 `findAndReturnMax()` 的参数值（例如，`arr` 和 `max`）：

```
(gdb) frame 1

#1  0x0000555555554810 in main (argc=1, argv=0x7fffffffe398) at badprog.c:40

40   if ( findAndReturnMax(arr, 5, max) != 0 ) {

(gdb) print arr

$1 = {17, 21, 44, 2, 60}

(gdb) print max

$2 = 17

(gdb)
```

参数值看起来没有问题，因此我们检查 `findAndReturnMax()` 函数的返回值。为此，我们在 `findAndReturnMax()` 返回之前设置了一个断点，看看它计算出的 `max` 值是什么：

```
(gdb) break 30

Breakpoint 3 at 0x5555555547ae: file badprog.c, line 30.

(gdb) cont

Continuing.

Breakpoint 3, findAndReturnMax (array1=0x7fffffffe290, len=5, max=60)

    at badprog.c:30

30   return 0;

(gdb) print max

$3 = 60
```

这显示出函数已经找到了正确的最大值（60）。让我们执行接下来的几行代码，看看 `main()` 函数接收到的值是什么：

```
(gdb) next

31   }

(gdb) next

main (argc=1, argv=0x7fffffffe398) at badprog.c:44

44   printf("max value in the array is %d\n", max);

(gdb) where

#0  main (argc=1, argv=0x7fffffffe398) at badprog.c:44

(gdb) print max

$4 = 17
```

我们找到了第二个 bug！`findAndReturnMax()` 函数识别了传递数组中的最大值（60），但是它没有将该值返回给 `main()` 函数。为了解决这个错误，我们需要修改 `findAndReturnMax()` 函数，使其返回 `max` 值，或者添加一个“按指针传递”的参数，供函数用来修改 `main()` 函数中 `max` 局部变量的值。

##### 使用 GDB 调试崩溃程序的示例（segfaulter.c）

第二个示例 GDB 会话（在 `segfaulter.c` 程序上运行）演示了当程序崩溃时 GDB 的行为，以及我们如何利用 GDB 帮助发现崩溃发生的原因。

在这个例子中，我们只是运行 `segfaulter` 程序并让它崩溃：

```
$ gcc -g -o segfaulter segfaulter.c

$ gdb ./segfaulter

(gdb) run

Starting program: ./segfaulter

Program received signal SIGSEGV, Segmentation fault.

0x00005555555546f5 in initfunc (array=0x0, len=100) at segfaulter.c:14

14     array[i] = i;
```

一旦程序崩溃，GDB 会在崩溃点暂停程序的执行并接管控制。GDB 允许用户发出命令检查程序在崩溃点的运行时状态，这通常能帮助我们发现程序崩溃的原因，并找出修复崩溃的办法。GDB 的 `where` 和 `list` 命令对于确定程序崩溃的位置特别有用：

```
(gdb) where

#0 0x00005555555546f5 in initfunc (array=0x0, len=100) at segfaulter.c:14

 #1 0x00005555555547a0 in main (argc=1, argv=0x7fffffffe378) at segfaulter.c:37

(gdb) list

9 int initfunc(int *array, int len) {

10

11     int i;

12

13     for(i=1; i <= len; i++) {

14         array[i] = i;

15     }

16     return 0;

17 }

18
```

该输出告诉我们程序在第 14 行的 `initfunc()` 函数处崩溃。检查第 14 行参数和局部变量的值可能能告诉我们崩溃的原因：

```
(gdb) print i

$2 = 1

(gdb) print array[i]

Cannot access memory at address 0x4
```

`i` 的值看起来没问题，但我们在尝试访问 `array` 的索引 `i` 时看到错误。让我们打印出 `array` 的值（即数组基地址的值），看看它是否能告诉我们什么：

```
(gdb) print array

$3 = (int *) 0x0
```

我们已经找到了崩溃的原因！数组的基地址为零（或 `NULL`），我们知道解引用空指针（通过 `array[i]`）会导致程序崩溃。

让我们看看通过查看调用者的栈帧，是否能找出为什么 `array` 参数是 `NULL`：

```
(gdb) frame 1

#1 0x00005555555547a0 in main (argc=1, argv=0x7fffffffe378) at segfaulter.c:37

37   if(initfunc(arr, 100) != 0 ) {

(gdb) list

32 int main(int argc, char *argv[]) {

33

34     int *arr = NULL;

35     int max = 6;

36

37     if(initfunc(arr, 100) != 0 ) {

38         printf("init error\n");

39         exit(1);

40     }

 41

(gdb) print arr

$4 = (int *) 0x0

(gdb)
```

进入调用者的栈帧并打印出 `main()` 传递给 `initfunc()` 的参数值显示，`main()` 函数向 `initfunc()` 传递了一个空指针。换句话说，用户在调用 `initfunc()` 之前忘记为 `arr` 数组分配内存。解决方法是在第 34 行使用 `malloc()` 函数为 `arr` 分配内存空间。

这两个 GDB 示例会话展示了常用的命令，用于查找程序中的 bug。在接下来的章节中，我们将更详细地讨论这些和其他 GDB 命令。

### 3.2 GDB 命令详解

在本节中，我们列出了常见的 GDB 命令，并通过示例展示了一些功能。我们首先讨论一些常用的键盘快捷键，使 GDB 更易使用。

#### 3.2.1 GDB 中的键盘快捷键

GDB 支持*命令行补全*。用户可以输入命令的唯一前缀并按下 TAB 键，GDB 会尝试补全命令行。此外，用户还可以使用唯一的*简短缩写*来执行许多常见的 GDB 命令。例如，用户可以输入 `p x` 来打印变量 `x` 的值，而不必输入完整的命令 `print x`；`l` 可以用来执行 `list` 命令，`n` 用来执行 `next` 命令。

*上下箭头键*可滚动查看之前的 GDB 命令行，省去了每次重新输入命令的需要。

在 GDB 提示符下按下 RETURN 键会执行*最近的上一条命令*。这在通过一系列 `next` 或 `step` 命令逐步执行时特别有用；只需按下 RETURN，GDB 就会执行下一条指令。

#### 3.2.2 常用 GDB 命令

我们在这里总结了 GDB 最常用的命令，将它们按相似功能分组：控制程序执行的命令；评估程序执行点的命令；设置和控制断点的命令；打印程序状态和评估表达式的命令。GDB 的 `help` 命令提供了关于所有 GDB 命令的信息：

help   显示有关主题和 GDB 命令的帮助文档。

```
help <topic or command>   Shows help available for topic or command

help breakpoints    Shows help information about breakpoints

help print        Shows help information about print command
```

##### 执行控制流命令

break   设置断点。

```
break <func-name>   Set breakpoint at start of function <func-name>

break <line>        Set breakpoint at line number <line>

break <filename:><line>  Set breakpoint at <line> in file <filename>

break main          Set breakpoint at beginning of main

break 13            Set breakpoint at line 13

break gofish.c:34   Set breakpoint at line 34 in gofish.c

break main.c:34     Set breakpoint at line 34 in main.c
```

在特定文件中指定一行（如`break gofish.c:34`）允许用户在多个 C 源代码文件（`.c` 文件）中设置断点。此功能在设置的断点不在程序暂停点的同一文件时特别有用。

run   从头开始运行调试程序。

```
run <command line arguments>

run             Run with no command line arguments

run 2 40 100    Run with 3 command line arguments: 2, 40, 100
```

continue **（cont）**   从断点继续执行。

```
continue
```

step **（s）**   执行程序 C 源代码的下一行，如果该行执行了函数调用，则进入该函数。

```
step          Execute next line (stepping into a function)

step <count>  Executes next <count> lines of program code

step 10       Executes the next 10 lines (stepping into functions)
```

在 `step <count>` 命令的情况下，如果一行包含函数调用，则该行调用的函数的行数会计入 `count` 总数中。因此，`step <count>` 可能会导致程序在一个函数内部暂停，这个函数是从 `step <count>` 命令发出时程序的暂停点调用的。

next   类似于 `step` 命令，但它将函数调用视为单独的一行。换句话说，当下一条指令包含函数调用时，`next` 不会进入函数的执行，而是在函数调用返回后暂停程序（在函数调用后面紧接着的下一行暂停程序）。

```
next            Execute the next line

next <count>    Executes next <count> instructions
```

until   执行程序，直到达到指定的源代码行号。

```
until <line>    Executes until hit line number <line>
```

quit   退出 GDB。

```
quit
```

##### 检查执行点和列出程序代码的命令

list   列出程序源代码。

```
list                Lists next few lines of program source code

list <line>         Lists lines around line number <line> of program

list <start> <end>  Lists line numbers <start> through <end>

list <func-name>    Lists lines around beginning of function <func-name>

list 30 100         List source code lines 30 to 100
```

where **(**backtrace**,** bt**)**   显示堆栈内容（当前程序执行点的函数调用序列）。`where` 命令有助于定位程序崩溃的位置，并检查函数调用和返回之间接口的状态，例如传递给函数的参数值。

```
where
```

frame <frame-num>   进入栈帧编号 `<frame-num>` 的上下文。默认情况下，程序会在栈顶的帧 0 上暂停。`frame` 命令可以用来进入另一个栈帧的上下文。通常，GDB 用户会进入另一个栈帧，以打印出另一个函数的参数值和局部变量。

```
frame <frame-num>   Sets current stack frame to <frame-num>

info frame          Show state about current stack frame

frame 3             Move into stack frame 3's context (0 is top frame)
```

##### 设置和操作断点的命令

break   设置一个断点（关于此命令的更多解释见“执行控制流命令”部分，参见 第 162 页）。

```
break <func-name>   Set a breakpoint at start of a function

break <line>        Set a breakpoint at a line number

break main          Set a breakpoint at start of main

break 12            Set a breakpoint at line 12

break file.c:34     Set a breakpoint at line 34 of file.c
```

enable**,** disable**,** ignore**,** delete**,** clear   启用、禁用、忽略一定次数，或者删除一个或多个断点。`delete` 命令通过编号删除一个断点。与此相对，使用 `clear` 命令可以删除源代码中特定位置的断点。

```
disable <bnums ...>    Disable one or more breakpoints

enable  <bnums ...>    Enable one or more breakpoints

ignore  <bpnum> <num>  Don't pause at breakpoint <bpnum>

                         the next <num> times it's hit

delete  <bpnum>        Delete breakpoint number <bpnum>

delete                 Deletes all breakpoints

clear <line>           Delete breakpoint at line <line>

clear <func-name>      Delete breakpoint at function <func-name>

info break      List breakpoint info (including breakpoint bnums)

disable 3       Disable breakpoint number 3

ignore  2  5    Ignore the next 5 times breakpoint 2 is hit

enable  3       Enable breakpoint number 3

delete  1       Delete breakpoint number 1

clear   124     Delete breakpoint at source code line 124
```

condition   设置断点条件。条件断点是在某个条件为真时，才会将控制权转交给 GDB。它可以用来在循环内某个断点处，仅在循环迭代了指定次数后暂停（通过为循环计数器变量添加条件），或者仅当变量值对调试有意义时暂停程序（避免在其他情况下暂停程序）。

```
condition <bpnum> <exp>    Sets breakpoint number <bpnum> to break

                           only when expression <exp> is true

break 28            Set breakpoint at line 28 (in function play)

info break          Lists information about all breakpoints

  Num Type           Disp Enb Address    What

   1   breakpoint    keep y   0x080483a3 in play at gofish.c:28

condition 1 (i > 1000)     Set condition on breakpoint 1
```

##### 检查和评估程序状态及表达式的命令

print **(**p**)**   显示表达式的值。尽管 GDB 用户通常打印程序变量的值，但 GDB 会打印任何 C 表达式的值（即使是程序代码中没有的表达式）。`print` 命令支持以不同格式打印，并支持不同数字表示法的操作数。

```
print <exp>     Display the value of expression <exp>

p i             print the value of i

p i+3           print the value of (i+3)
```

要以不同格式打印：

```
print    <exp>     Print value of the expression as unsigned int

print/x  <exp>     Print value of the expression in hexadecimal

print/t  <exp>     Print value of the expression in binary

print/d  <exp>     Print value of the expression as signed int

print/c  <exp>     Print ASCII value of the expression

print  (int)<exp>  Print value of the expression as unsigned int

print/x 123        Prints  0x7b

print/t 123        Print  1111011

print/d 0x1c       Prints 28

print/c 99         Prints 'c'

print (int)'c'     Prints  99
```

要在表达式中指定不同的数字表示法（数字的默认表示法为十进制表示法）：

```
0x prefix for hex: 0x1c

0b prefix for binary: 0b101

print 0b101        Prints 5 (default format is decimal)

print 0b101 + 3    Prints 8

print 0x12  + 2    Prints 20 (hex 12 is 18 in decimal)

print/x 0x12  + 2  Prints 0x14 (decimal 20 in hexadecimal format)
```

有时，表达式可能需要显式的类型转换，以告知 `print` 如何解释它们。例如，在这里，需要将地址值重新转换为特定类型（`int *`），才能解引用该地址（否则，GDB 不知道如何解引用该地址）：

```
print *(int *)0x8ff4bc10   Print int value at address 0x8ff4bc10
```

当使用 `print` 显示解引用指针变量的值时，不需要类型转换，因为 GDB 知道指针变量的类型，并知道如何解引用它的值。例如，如果 `ptr` 被声明为 `int *`，则可以这样显示它指向的 int 值：

```
print *ptr      Print the int value pointed to by ptr
```

要打印存储在硬件寄存器中的值：

```
print $eax      Print the value stored in the eax register
```

display   在达到断点时自动显示表达式的值。表达式语法与`print`命令相同。  

```
display <exp>   Display value of <exp> at every breakpoint

display i

display array[i]
```

x **(检查内存)**   显示内存位置的内容。此命令类似于`print`，但它将参数解释为一个地址值，并通过解引用该地址来打印存储在该地址上的值。  

```
x <memory address expression>

x  0x5678       Examine the contents of memory location 0x5678

x  ptr          Examine the contents of memory that ptr points to

x  &temp        Can specify the address of a variable

                 (this command is equivalent to: print temp)
```

像`print`一样，`x`可以以不同的格式显示值（例如，作为`int`、`char`或字符串）。  

**警告 EXAMINE 的格式设置是粘性的**  

*粘性格式*意味着 GDB 会记住当前的格式设置，并将其应用于后续没有指定格式的`x`调用。例如，如果用户输入命令`x/c`，那么所有后续没有格式设置的`x`执行将使用`/c`格式。因此，只有在用户希望更改最近一次调用`x`的内存地址单位、重复次数或显示格式时，才需要显式指定`x`命令的格式选项。  

一般而言，`x`最多接受三个格式化参数（`x/nfu <memory` `address>`）；它们的列出顺序无关紧要：  

n   重复计数（正整数值）  

f   显示格式（`s`：字符串；`i`：指令；`x`：十六进制；`d`：十进制；`t`：二进制；`a`：地址；……）  

u   单位格式（字节数）（`b`：字节；`h`：2 字节；`w`：4 字节；`g`：8 字节）

下面是一些例子（假设`s1 = "Hello There"`位于内存地址`0x40062d`）：  

```
x/d   ptr      Print value stored at what ptr points to, in decimal

x/a   &ptr     Print value stored at address of ptr, as an address

x/wx  &temp    Print 4-byte value at address of temp, in hexadecimal

x/10dh 0x1234  Print 10 short values starting at address 0x1234, in decimal

x/4c s1        Examine the first 4 chars in s1

    0x40062d   72 'H'  101 'e'  108 'l'  108 'l'

x/s s1         Examine memory location associated with var s1 as a string

    0x40062d   "Hello There"

x/wd s1        Examine the memory location assoc with var s1 as an int

               (because formatting is sticky, need to explicitly set

               units to word (w) after x/s command sets units to byte)

    0x40062d   72

x/8d s1        Examine ASCII values of the first 8 chars of s1

    0x40062d:  72  101 108 108 111 32  84  104
```

whatis   显示表达式的类型。  

```
whatis <exp>       Display the data type of an expression

whatis (x + 3.4)   Displays:  type = double
```

set   分配/更改程序变量的值，或者分配一个值到特定的内存地址或特定的机器寄存器中。  

```
set <variable> = <exp>   Sets variable <variable> to expression <exp>

set x = 123*y            Set var x's value to (123*y)
```

info     列出有关程序状态和调试器状态的信息。`info`有许多选项，可以获取关于程序当前执行状态和调试器的各种信息。一些例子包括：  

```
help info         Shows all the info options

help status       Lists more info and show commands

info locals       Shows local variables in current stack frame

info args         Shows the argument variable of current stack frame

info break        Shows breakpoints

info frame        Shows information about the current stack frame

info registers    Shows register values

info breakpoints  Shows the status of all breakpoints
```

有关这些及其他 GDB 命令的更多信息，请参阅 GDB 手册（`man gdb`）和 GNU 调试器主页：[`www.gnu.org/software/gdb/`](https://www.gnu.org/software/gdb/)。  

### 3.3 使用 Valgrind 调试内存  

Valgrind 的 Memcheck 调试工具可以突出显示程序中的堆内存错误。堆内存是运行程序的内存部分，通过调用`malloc()`动态分配，并通过调用`free()`释放。在 C 程序中，Valgrind 可以发现的内存错误类型包括：  

+   从未初始化的内存读取（获取）值。例如：  

    ```
    int *ptr, x;

    ptr = malloc(sizeof(int) * 10);

    x = ptr[3];    // reading from uninitialized memory
    ```

+   在未分配的内存位置读取（获取）或写入（设置）值，这通常表示数组越界错误。例如：  

    ```
    ptr[11] = 100;  // writing to unallocated memory (no 11th element)

    x = ptr[11];    // reading from unallocated memory
    ```

+   释放已经释放的内存。例如：  

    ```
    free(ptr);

    free(ptr); // freeing the same pointer a second time
    ```

+   内存泄漏。*内存泄漏*是指一块已分配的堆内存空间没有被程序中的任何指针变量引用，因此无法释放。也就是说，当程序丢失了已分配堆空间的地址时，就会发生内存泄漏。例如：  

    ```
    ptr = malloc(sizeof(int) * 10);

    ptr = malloc(sizeof(int) * 5);  // memory leak of first malloc of

                                    // 10 ints
    ```

内存泄漏最终可能导致程序耗尽堆内存空间，从而导致后续对`malloc()`的调用失败。其他类型的内存访问错误，如无效的读取和写入，可能会导致程序崩溃，或导致程序内存中的某些内容以看似神秘的方式被修改。

内存访问错误是程序中最难以发现的错误之一。通常，内存访问错误不会立即导致程序执行中明显的错误。相反，它可能会触发一个在后续执行中出现的错误，通常发生在与错误源看似无关的程序部分。有时，带有内存访问错误的程序可能在某些输入下运行正常，而在其他输入下崩溃，这使得错误的原因难以找到和修复。

使用 Valgrind 可以帮助程序员发现这些难以找到和修复的堆内存访问错误，节省大量的调试时间和精力。Valgrind 还帮助程序员识别在代码测试和调试中未发现的潜在堆内存错误。

#### 3.3.1 带有堆内存访问错误的示例程序

作为一个发现和修复内存访问错误的难度的例子，考虑以下这个小程序。这个程序在第二个`for`循环中展示了一个“写入未分配堆内存”的错误，当它赋值超出了`bigfish`数组的边界（注意：代码列出了源代码的行号，`print_array()`函数的定义未显示，但其行为如描述所示）：

bigfish.c

```
 1  #include <stdio.h>

 2  #include <stdlib.h>

 3

 4  /* print size elms of array p with name name */

 5  void print_array(int *p, int size, char *name) ;

 6

 7  int main(int argc, char *argv[]) {

 8      int *bigfish, *littlefish, i;

 9

10      // allocate space for two int arrays

11      bigfish = (int *)malloc(sizeof(int) * 10);

12      littlefish = (int *)malloc(sizeof(int) * 10);

13      if (!bigfish || !littlefish) {

14          printf("Error: malloc failed\n");

15          exit(1);

16      }

17      for (i=0; i < 10; i++) {

18          bigfish[i] = 10 + i;

19          littlefish[i] = i;

20      }

21      print_array(bigfish,10, "bigfish");

22      print_array(littlefish,10, "littlefish");

23

24      // here is a heap memory access error

25      // (write beyond bounds of allocated memory):

26      for (i=0; i < 13; i++) {

27          bigfish[i] = 66 + i;

28      }

29      printf("\nafter loop:\n");

30      print_array(bigfish,10, "bigfish");

31      print_array(littlefish,10, "littlefish");

32

33      free(bigfish);

34      free(littlefish);  // program will crash here

35      return 0;

36  }
```

在`main()`函数中，第二个`for`循环在写入超出`bigfish`数组边界的三个索引（索引 10、11 和 12）时，会导致堆内存访问错误。程序不会在错误发生时（第二个`for`循环执行时）崩溃；相反，它会在稍后的执行中，在调用`free(littlefish)`时崩溃：

```
bigfish:

 10  11  12  13  14  15  16  17  18  19

littlefish:

  0   1   2   3   4   5   6   7   8   9

after loop:

bigfish:

 66  67  68  69  70  71  72  73  74  75

littlefish:

 78   1   2   3   4   5   6   7   8   9

Segmentation fault (core dumped)
```

在 GDB 中运行此程序时，会显示程序在调用`free(littlefish)`时发生段错误（segfault）并崩溃。此时崩溃可能让程序员怀疑是`littlefish`数组的访问存在问题。然而，错误的根本原因是对`bigfish`数组的写操作，与程序访问`littlefish`数组的方式无关。

程序崩溃最可能的原因是`for`循环超出了`bigfish`数组的边界，并覆盖了`bigfish`最后一个元素的堆内存位置和`littlefish`第一个元素的堆内存位置之间的内存。这两者之间的堆内存位置（以及`littlefish`第一个元素之前的内存）被`malloc()`用来存储关于分配给`littlefish`数组的堆内存的元数据。在内部，`free()`函数使用这些元数据来确定需要释放多少堆内存。对`bigfish`的索引 10 和 11 的修改覆盖了这些元数据值，导致程序在调用`free(littlefish)`时崩溃。然而，我们需要注意的是，并非所有`malloc()`函数的实现都使用这种策略。

由于程序包含了在内存访问`bigfish`出错后打印`littlefish`的代码，错误的原因可能对程序员来说更加明显：第二个`for`循环以某种方式修改了`littlefish`数组的内容（它的元素 0 值在循环后“神秘地”从`0`变成了`78`）。然而，即使在这个非常小的程序中，也可能很难找到真正的错误：如果程序在第二个`for`循环发生内存访问错误后没有打印出`littlefish`，或者如果`for`循环的上限是`12`而不是`13`，那么程序变量的值就不会有明显的神秘变化，这也就无法帮助程序员发现程序在访问`bigfish`数组时存在错误。

在更大的程序中，这种类型的内存访问错误可能出现在程序代码的完全不同部分，而不是崩溃的那部分。可能没有逻辑关联性在访问已被破坏的堆内存的变量和那些错误覆盖该内存的变量之间；它们唯一的关联是它们恰好引用了堆中相邻分配的内存地址。请注意，这种情况可能在程序的每次运行中有所不同，而且这种行为通常对程序员是隐藏的。同样，有时不良的内存访问可能对程序的运行没有明显的影响，这使得这些错误很难被发现。每当程序对某些输入运行正常，但对其他输入崩溃时，这就是程序中内存访问错误的一个信号。

像 Valgrind 这样的工具可以通过迅速指示程序员代码中堆内存访问错误的来源和类型，从而节省数天的调试时间。在前面的程序中，Valgrind 划定了错误发生的位置（当程序访问超出 `bigfish` 数组边界的元素时）。Valgrind 错误信息包括错误类型、程序中错误发生的位置，以及程序中堆内存分配的位置，该内存紧邻错误内存访问附近。例如，以下是当程序执行第 27 行时 Valgrind 会显示的信息（实际 Valgrind 错误信息中的一些细节已省略）：

```
Invalid write

 at main (bigfish.c:27)

 Address is 0 bytes after a block of size 40 alloc'd

   by main (bigfish.c:11)
```

该 Valgrind 错误信息表明程序在第 27 行写入了无效（未分配）的堆内存，并且该无效内存在第 11 行分配的一块内存后立即出现，表明循环访问了一些超出 `bigfish` 所指向的分配内存边界的元素。解决此 bug 的潜在方法是增加传递给 `malloc()` 的字节数，或者更改第二个 `for` 循环的边界，以避免写入超过分配的堆内存空间的边界。

除了能够发现堆内存中的内存访问错误外，Valgrind 还可以发现一些与栈内存访问相关的错误，例如使用未初始化的局部变量或尝试访问超出当前栈边界的栈内存位置。然而，Valgrind 在检测栈内存访问错误时，粒度不如堆内存，并且它无法检测到全局数据内存的内存访问错误。

一个程序可能会有栈内存和全局内存的访问错误，Valgrind 无法找到这些错误。然而，这些错误会导致程序行为异常或程序崩溃，这种现象与堆内存访问错误引起的行为类似。例如，在栈上超出静态声明数组边界的内存位置写入可能会“神秘地”改变其他局部变量的值，或者可能会覆盖用于从函数调用返回时保存的栈状态，导致函数返回时崩溃。使用 Valgrind 调试堆内存错误的经验可以帮助程序员识别和修复类似的栈内存和全局内存访问错误。

#### 3.3.2 如何使用 Memcheck

我们通过一个包含多个错误内存访问的示例程序 `valgrindbadprog.c` 来展示 Valgrind Memcheck 内存分析工具的主要功能（代码中的注释描述了错误类型）。Valgrind 默认运行 Memcheck 工具；我们在接下来的代码片段中依赖于这种默认行为。您也可以通过使用 `--tool=memcheck` 选项显式指定 Memcheck 工具。在后续部分，我们将通过调用 `--tool` 选项来调用其他 Valgrind 性能分析工具。

要运行 Memcheck，首先使用`-g`标志编译`valgrindbadprog.c`程序，向可执行文件添加调试信息。然后，使用`valgrind`运行可执行文件。请注意，对于非交互式程序，将 Valgrind 的输出重定向到文件以在程序退出后查看可能是有帮助的：

```
$ gcc -g valgrindbadprog.c

$ valgrind -v ./a.out

# re-direct valgrind (and a.out) output to file 'output.txt'

$ valgrind -v ./a.out >& output.txt

# view program and valgrind output saved to out file

$ vim output.txt
```

Valgrind 的 Memcheck 工具会在程序执行期间打印出内存访问错误和警告。在程序执行结束时，Memcheck 还会打印关于程序中任何内存泄漏的摘要。尽管修复内存泄漏很重要，但程序正确性的其他类型的内存访问错误则更为关键。因此，除非内存泄漏导致程序耗尽堆内存空间并崩溃，否则程序员应首先专注于修复这些其他类型的内存访问错误，而不是考虑内存泄漏。要查看单个内存泄漏的详细信息，请使用`--leak-check=yes`选项。

初次使用 Valgrind 时，其输出可能看起来有些难以解析。然而，输出都遵循同一基本格式，一旦了解了这个格式，就更容易理解 Valgrind 显示的关于堆内存访问错误和警告的信息。以下是运行`valgrindbadprog.c`程序时的一个 Valgrind 错误示例：

```
==31059== Invalid write of size 1

==31059==    at 0x4006C5: foo (valgrindbadprog.c:29)

==31059==    by 0x40079A: main (valgrindbadprog.c:56)

==31059==  Address 0x52045c5 is 0 bytes after a block of size 5 alloc'd

==31059==    at 0x4C2DB8F: malloc (in /usr/lib/valgrind/...)

==31059==    by 0x400660: foo (valgrindbadprog.c:18)

==31059==    by 0x40079A: main (valgrindbadprog.c:56)
```

每行 Valgrind 输出都以进程的 ID（PID）号（例如此处的 31059）开头。

```
==31059==
```

大多数 Valgrind 的错误和警告具有以下格式：

+   错误或警告的类型。

+   错误发生的位置（程序执行到达错误位置时的堆栈跟踪。）

+   错误周围的堆内存分配位置（通常是与错误相关的内存分配。）

在上述示例错误中，第一行指示对内存的无效写入（写入到堆中未分配的内存——这是一个非常严重的错误！）：

```
==31059== Invalid write of size 1
```

接下来的几行显示了错误发生的堆栈跟踪。这些指示表明，在`foo()`函数的第 29 行发生了无效写入，而`foo()`函数是由`main()`函数在第 56 行调用的。

```
==31059== Invalid write of size 1

==31059==    at 0x4006C5: foo (valgrindbadprog.c:29)

==31059==    by 0x40079A: main (valgrindbadprog.c:56)
```

剩余的行显示了发生无效写入附近的堆空间位置。Valgrind 的输出部分显示，该无效写入是在分配了一个 5 字节堆内存空间块之后（由第 18 行`foo()`函数中的`malloc()`调用分配），立即发生的，由`main()`函数在第 56 行调用`foo()`函数：

```
==31059==  Address 0x52045c5 is 0 bytes after a block of size 5 alloc'd

==31059==    at 0x4C2DB8F: malloc (in /usr/lib/valgrind/...)

==31059==    by 0x400660: foo (valgrindbadprog.c:18)

==31059==    by 0x40079A: main (valgrindbadprog.c:56)
```

此错误信息显示了程序中存在一个未分配的堆内存写入错误，并指导用户到程序中特定的错误发生位置（第 29 行）和错误周围的内存分配位置（第 18 行）。通过查看程序中的这些点，程序员可以找到错误的原因和修复方法：

```
 18   c = (char *)malloc(sizeof(char) * 5);

 ...

 22   strcpy(c, "cccc");

 ...

 28   for (i = 0; i <= 5; i++) {

 29       c[i] = str[i];

 30   }
```

原因是`for`循环执行了一次过多，访问了数组`c`的`c[5]`，这超出了数组`c`的末尾。修复方法可以是修改第 29 行的循环边界，或者在第 18 行分配一个更大的数组。

如果仅查看 Valgrind 错误周围的代码对程序员来说不足以理解或修复错误，可以考虑使用 GDB 进行调试。在与 Valgrind 错误相关联的代码点设置断点可以帮助程序员评估程序的运行状态，并理解 Valgrind 错误的原因。例如，在第 29 行设置断点并打印`i`和`str`的值，程序员可以看到当`i`为 5 时的数组越界错误。在这种情况下，结合使用 Valgrind 和 GDB 有助于程序员确定如何修复 Valgrind 发现的内存访问错误。

虽然本章重点介绍了 Valgrind 的默认 Memcheck 工具，但我们稍后在书中还将详细介绍 Valgrind 的其他功能，包括 Cachegrind 缓存分析工具（第十一章）、Callgrind 代码分析工具（第十二章）和 Massif 内存分析工具（第十二章）。有关使用 Valgrind 的更多信息，请参阅 Valgrind 主页 *[`valgrind.org`](https://valgrind.org)*，以及其在线手册 *[`valgrind.org/docs/manual/`](https://valgrind.org/docs/manual/)*。

### 3.4 高级 GDB 功能

本节介绍了高级 GDB 功能，其中一些功能只有在阅读 第十三章 “Notes” 后才会有意义。

#### 3.4.1 GDB 和 make

GDB 接受`make`命令在调试会话期间重新构建可执行文件，如果构建成功，则会运行新构建的程序（当发出`run`命令时）。

```
(gdb) make

(gdb) run
```

在 GDB 中重新构建是方便的，适用于已设置许多断点并已修复一个错误，但希望继续调试会话的用户。在这种情况下，用户无需退出 GDB，重新编译，用新的可执行文件重新启动 GDB，并重新设置所有断点，而是可以运行`make`并开始调试程序的新版本，所有断点仍然保持设置状态。然而，需要注意的是，如果在 GDB 中运行`make`修改 C 源代码并重新编译可能导致断点在新程序版本中不在与旧版本相同的逻辑位置，因为源代码行可能已经添加或删除。当出现此问题时，要么退出 GDB 并在新的可执行文件上重新启动 GDB 会话，要么使用`disable`或`delete`禁用或删除旧断点，然后使用`break`在新编译的程序版本中设置新的断点位置。

#### 3.4.2 将 GDB 附加到正在运行的进程

GDB 支持调试已经运行的程序（而不是从 GDB 会话内启动程序），通过*附加*GDB 到正在运行的进程。为此，用户需要获取进程 ID（PID）值：

1\. 使用`ps` shell 命令获取进程的 PID：

# 使用 ps 命令获取进程的 PID（列出在当前 shell 中启动的所有进程）：

$ ps

# 列出所有进程并通过 grep 管道过滤只有名为 a.out 的进程：

$ ps -A | grep a.out

PID TTY          TIME CMD

12345 pts/3      00:00:00 a.out

2. 启动 GDB 并将其附加到特定的运行中进程（PID 为 12345）：

# gdb <可执行文件> <pid>

$ gdb a.out 12345

(gdb)

# 或者另一种语法：gdb attach <pid> <可执行文件>

$ gdb attach 12345 a.out

(gdb)

将 GDB 附加到进程会暂停该进程，用户可以在继续执行之前输入 GDB 命令。

或者，程序可以通过调用`kill(getpid(), SIGSTOP)`（如`attach_example.c`示例中所示）显式地暂停自身，等待调试。当程序在这一点暂停时，程序员可以将 GDB 附加到该进程进行调试。

无论程序如何暂停，在 GDB 附加后并且用户输入一些 GDB 命令后，程序的执行将从附加点继续，使用`cont`命令。如果`cont`不起作用，GDB 可能需要显式地发送`SIGCONT`信号给进程以继续执行：

```
(gdb) signal SIGCONT
```

#### 3.4.3 在 fork 时跟随进程

当 GDB 调试调用`fork()`函数创建新子进程的程序时，GDB 可以被设置为跟随（调试）父进程或子进程，从而使另一个进程的执行不受 GDB 的影响。默认情况下，GDB 在调用`fork()`后会跟随父进程。如果要让 GDB 跟随子进程，可以使用`set follow-fork-mode`命令：

```
(gdb) set follow-fork-mode child     # Set gdb to follow child on fork

(gdb) set follow-fork-mode parent    # Set gdb to follow parent on fork

(gdb) show follow-fork-mode          # Display gdb's follow mode
```

在程序的`fork()`调用处设置断点对用户在 GDB 会话中想要改变这一行为时非常有用。

`attach_example.c`示例展示了如何在 fork 后“跟随”两个进程：GDB 在 fork 后跟随父进程，子进程通过发送`SIGSTOP`信号显式地暂停自己，允许程序员在子进程继续执行之前将第二个 GDB 进程附加到子进程上。

#### 3.4.4 信号控制

GDB 进程可以向它正在调试的目标进程发送信号，并处理目标进程接收到的信号。

GDB 可以通过使用`signal`命令向它正在调试的进程发送信号：

```
(gdb) signal SIGCONT

(gdb) signal SIGALRM

...
```

有时，用户希望 GDB 在调试进程接收到信号时执行某些操作。例如，如果程序尝试访问与它正在访问的类型不匹配的内存地址，它会接收到`SIGBUS`信号并通常退出。GDB 对`SIGBUS`的默认行为也是让进程退出。然而，如果你希望 GDB 在接收到`SIGBUS`信号时检查程序状态，你可以通过`handle`命令指定 GDB 以不同的方式处理`SIGBUS`信号（`info`命令显示有关 GDB 如何处理调试过程中接收到的信号的更多信息）：

```
(gdb) handle SIGBUS stop    # if program gets a SIGBUS, gdb gets control

(gdb) info signal           # list info on all signals

(gdb) info SIGALRM          # list info just for the SIGALRM signal
```

#### 3.4.5 DDD 设置与 Bug 修复

运行 DDD 会在你的主目录下创建一个`.ddd`目录，DDD 使用这个目录存储它的设置，这样用户就不需要在每次启动时重新设置所有偏好设置。保存的设置包括子窗口的大小、菜单显示选项以及启用窗口查看寄存器值和汇编代码的功能。

有时 DDD 在启动时会挂起，显示“Waiting until GDB ready”消息。这通常表示其保存的设置文件存在错误。修复这个问题的最简单方法是删除`.ddd`目录（你会丢失所有保存的设置，并且需要在它重新启动时重新设置）：

```
$ rm -rf ~/.ddd  # Be careful when entering this command!

$ ddd ./a.out
```

### 3.5 调试汇编代码

除了高级的 C 和 C++调试外，GDB 还可以在汇编代码层面调试程序。这样，GDB 能够列出来自函数的反汇编代码序列、在汇编指令级别设置断点、逐条执行汇编指令以及在程序运行时检查存储在机器寄存器、栈和堆内存地址中的值。我们在本节使用 IA32 作为示例汇编语言，但这里介绍的 GDB 命令适用于 GCC 支持的任何汇编语言。我们提到，读者可能会在后续章节深入了解汇编代码后，发现本小节特别有用。

我们使用以下简短的 C 程序作为示例：

```
int main() {

    int x, y;

    x = 1;

    x = x + 2;

    x = x - 14;

    y = x * 100;

    x = x + y * 6;

 return 0;

}
```

要编译为 IA32 可执行文件，请使用`-m32`标志：

```
$ gcc -m32 -o simpleops simpleops.c
```

可选择使用`gcc`的`-fno-asynchronous-unwind-tables`命令行选项进行编译，这会生成一个对程序员来说稍微容易阅读和理解的 IA32 代码：

```
$ gcc -m32 -fno-asynchronous-unwind-tables -o simpleops simpleops.c
```

#### 3.5.1 使用 GDB 检查二进制代码

本节展示了一些示例 GDB 命令，用于在汇编代码层面调试一个简短的 C 程序。以下表格总结了本节展示的许多命令：

| **GDB 命令** | **描述** |
| --- | --- |
| `break sum` | 在函数`sum`的开始处设置断点 |
| `break *0x0804851a` | 在内存地址 0x0804851a 处设置断点 |
| `disass main` | 反汇编`main`函数 |
| `ni` | 执行下一条指令 |
| `si` | 进入函数调用（逐条指令） |
| `info registers` | 列出寄存器内容 |
| `p $eax` | 打印寄存器%eax 中存储的值 |
| `p *(int *)($ebp+8)` | 打印地址（%ebp+8）处的整数值 |
| `x/d $ebp+8` | 检查地址处的内存内容 |

首先，编译为 IA32 汇编并在 IA32 可执行程序`simpleops`上运行 GDB：

```
$ gcc -m32 -fno-asynchronous-unwind-tables -o simpleops simpleops.c

$ gdb ./simpleops
```

然后，在`main`中设置断点，然后使用`run`命令开始运行程序：

```
(gdb) break main

(gdb) run
```

`disass`命令反汇编（列出与之关联的汇编代码）程序的部分代码。例如，要查看`main`函数的汇编指令：

```
(gdb) disass main         # Disassemble the main function
```

GDB 允许程序员通过解引用指令的内存地址来在单个汇编指令上设置断点：

```
(gdb) break *0x080483c1     # Set breakpoint at instruction at 0x080483c1
```

程序的执行可以通过`si`或`ni`逐条执行汇编指令来进行，其中`si`步入调用，`ni`跳过调用指令：

```
(gdb) ni      # Execute the next instruction

(gdb) si      # Execute next instruction; if it is a call instruction,

              # step into the function
```

`si`命令会逐步进入函数调用，这意味着 GDB 会在被调用函数的第一条指令处暂停程序。`ni`命令会跳过它们，这意味着 GDB 会在调用指令执行并返回到调用者后，暂停程序并停在下一条指令。

程序员可以使用`print`命令和以`$`前缀的寄存器名称打印存储在机器寄存器中的值：

```
(gdb) print $eax    # print the value stored in register eax
```

`display`命令在达到断点时自动显示值：

```
(gdb) display $eax

(gdb) display $edx
```

`info registers`命令显示所有存储在机器寄存器中的值：

```
(gdb) info registers
```

#### 3.5.2 使用 DDD 进行汇编级别调试

DDD 调试器为另一个调试器（在此为 GDB）提供了图形界面。它提供了一个很好的界面，用于显示汇编代码、查看寄存器以及逐步执行 IA32 指令。由于 DDD 具有用于显示反汇编代码、寄存器值和 GDB 命令提示符的独立窗口，因此在汇编代码级别调试时，它通常比 GDB 更容易使用。

要使用 DDD 调试，请将`ddd`替换为`gdb`：

```
$ ddd ./simpleops
```

GDB 提示符出现在底部窗口，在这里接受 GDB 命令。尽管它提供了一些 GDB 命令的菜单选项和按钮，但通常底部的 GDB 提示符更容易使用。

通过选择视图 ▶机器代码窗口菜单选项，DDD 显示程序的汇编代码视图。该选项会创建一个新子窗口，列出程序的汇编代码（你可能需要调整窗口大小以使其变大）。

要在单独的窗口中查看程序的所有寄存器值，请启用状态 ▶寄存器菜单选项。

#### 3.5.3 GDB 汇编代码调试命令和示例

以下是一些对在汇编代码级别调试有用的 GDB 命令的详细信息和示例（有关这些命令的更多详细信息，特别是`print`和`x`格式选项，请参见第 161 页的“常见 GDB 命令”部分）：

disass   反汇编函数或地址范围的代码。

```
disass <func_name>   # Lists assembly code for function

disass <start> <end> # Lists instructions between start & end address

disass main          # Disassemble main function

disass 0x1234 0x1248 # Disassemble instructions between addr 0x1234 & 0x1248
```

break   在指令地址处设置一个断点。

```
break *0x80dbef10 # Sets breakpoint at the instruction at address 0x80dbef10
```

stepi **（si）**, nexti **（ni）**

```
stepi, si       # Execute next machine code instruction,

                # stepping into function call if it is a call instruction

nexti,  ni      # Execute next machine code instruction,

                # treating function call as a single instruction
```

info registers   列出所有寄存器的值。

print   显示表达式的值。

```
print $eax               # Print the value stored in the eax register

print *(int *)0x8ff4bc10 # Print int value stored at memory addr 0x8ff4bc10
```

x   显示给定地址的内存位置的内容。请记住，`x`的格式是固定的，因此需要显式更改。

```
(gdb) x $ebp-4      # Examine memory at address: (contents of register ebp)-4

                    # if the location stores an address x/a, an int x/wd, ...

(gdb) x/s 0x40062d  # Examine the memory location 0x40062d as a string

0x40062d   "Hello There"

(gdb) x/4c 0x40062d # Examine the first 4 char memory locations

                    # starting at address 0x40062d

0x40062d   72 'H'  101 'e' 108 'l' 108 'l'

(gdb) x/d 0x40062d  # Examine the memory location 0x40062d in decimal

0x40062d   72       # NOTE: units is 1 byte, set by previous x/4c command

(gdb) x/wd 0x400000 # Examine memory location 0x400000 as 4 bytes in decimal

0x400000   100      # NOTE: units was 1 byte set, need to reset to w
```

set   设置内存位置和寄存器的内容。

```
set $eax = 10               # Set the value of register eax to 10

set $esp = $esp + 4         # Pop a 4-byte value off the stack

set *(int *)0x8ff4bc10 = 44 # Store 44 at address 0x8ff4bc10
```

display   每次命中断点时打印表达式的值。

```
display $eax       # Display value of register eax
```

#### 3.5.4 常用汇编调试命令快速总结

```
$ ddd ./a.out

(gdb) break main

(gdb) run

(gdb) disass main         # Disassemble the main function

(gdb) break sum           # Set a breakpoint at the beginning of a function

(gdb) cont                # Continue execution of the program

(gdb) break *0x0804851a   # Set a breakpoint at memory address 0x0804851a

(gdb) ni                  # Execute the next instruction

(gdb) si                  # Step into a function call (step instruction)

(gdb) info registers      # List the register contents

(gdb) p $eax              # Print the value stored in register %eax

(gdb) p  *(int *)($ebp+8) # Print out value of an int at addr (%ebp+8)

(gdb) x/d $ebp+8          # Examine the contents of memory at the given

                          # address (/d: prints the value as an int)

(gdb) x/s 0x0800004       # Examine contents of memory at address as a string

(gdb) x/wd 0xff5634       # After x/s, the unit size is 1 byte, so if want

                          # to examine as an int specify both the width w \& d
```

### 3.6 使用 GDB 调试多线程程序

调试多线程程序可能会比较棘手，因为存在多个执行流，并且线程之间的交互会影响程序行为。通常来说，以下是一些可以让多线程调试变得稍微容易的技巧。

+   如果可以，尽量调试一个线程较少的程序版本。

+   在代码中添加调试 `printf` 语句时，打印出执行线程的 ID，以识别哪个线程正在打印，并且在行尾加上 `\n`。

+   通过让只有一个线程打印其信息和公共信息来限制调试输出的数量。例如，如果每个线程将其逻辑 ID 存储在名为 `my_tid` 的局部变量中，则可以使用一个条件语句来根据 `my_tid` 的值限制打印调试输出，只对一个线程进行打印：

```
if (my_tid == 1) {

    printf("Tid:%d: value of count is %d and my i is %d\n", my_tid, count, i);

    fflush(stdout);

}
```

#### 3.6.1 GDB 和 Pthreads

GDB 调试器对调试多线程程序提供了特定支持，包括为单独线程设置断点和检查单独线程的堆栈。在 GDB 中调试 Pthreads 程序时，需要注意的一点是，每个线程至少有三个标识符：

+   Pthreads 库中线程的 ID（其 `pthread_t` 值）。

+   操作系统的轻量级进程（LWP）ID 值，用于线程。此 ID 部分用于操作系统跟踪该线程的调度。

+   线程的 GDB ID。在 GDB 命令中指定特定线程时使用该 ID。

线程 ID 之间的具体关系可能因操作系统和 Pthreads 库的实现而有所不同，但在大多数系统中，Pthreads ID、LWP ID 和 GDB 线程 ID 之间是三者一一对应的。

我们提供了一些 GDB 基础知识，用于在 GDB 中调试线程程序。有关在 GDB 中调试线程程序的更多信息，请参阅 *[`www.sourceware.org/gdb/current/onlinedocs/gdb/Threads.html`](https://www.sourceware.org/gdb/current/onlinedocs/gdb/Threads.html)。*

#### 3.6.2 GDB 线程特定命令

启用打印线程启动和退出事件：

```
set print thread-events
```

列出程序中所有现有线程（GDB 线程编号是第一个列出的值，触发断点的线程用 `*` 表示）：

```
info threads
```

切换到特定线程的执行上下文（例如，执行 `where` 时检查其堆栈），通过线程 ID 指定线程：

```
thread <threadno>

thread 12        # Switch to thread 12's execution context

where            # Thread 12's stack trace
```

为特定线程设置断点。在断点设置的位置，其他线程不会触发断点暂停程序并显示 GDB 提示符：

```
break <where> thread <threadno>

break foo thread 12    # Break when thread 12 executes function foo
```

要将特定的 GDB 命令应用于所有或某些线程，可以在 GDB 命令前添加前缀 `thread apply <threadno | all>`，其中 `threadno` 指的是 GDB 线程 ID：

```
thread apply <threadno|all> command
```

这并不适用于所有 GDB 命令，特别是设置断点时，因此设置线程特定断点时请改用此语法：

```
break <where> thread <threadno>
```

到达断点时，默认情况下，GDB 会暂停所有线程，直到用户输入 `cont`。用户可以改变此行为，要求 GDB 只暂停触发断点的线程，从而让其他线程继续执行。

#### 3.6.3 示例

我们展示了一些 GDB 命令及其在一个从文件 `racecond.c` 编译的多线程可执行程序中的 GDB 运行输出。

这个错误的程序在访问共享变量 `count` 时缺乏同步。因此，程序的不同运行产生了不同的 `count` 最终值，表明存在竞态条件。例如，以下是程序的两次运行，它们有五个线程并产生了不同的结果：

```
./a.out 5

hello I'm thread 0 with pthread_id 139673141077760

hello I'm thread 3 with pthread_id 139673115899648

hello I'm thread 4 with pthread_id 139673107506944

hello I'm thread 1 with pthread_id 139673132685056

hello I'm thread 2 with pthread_id 139673124292352

count = 159276966

./a.out 5

hello I'm thread 0 with pthread_id 140580986918656

hello I'm thread 1 with pthread_id 140580978525952

hello I'm thread 3 with pthread_id 140580961740544

hello I'm thread 2 with pthread_id 140580970133248

hello I'm thread 4 with pthread_id 140580953347840

count = 132356636
```

修复方法是将对 `count` 的访问放在临界区内，使用 `pthread_mutex_t` 变量。如果用户仅通过检查 C 代码无法看到此修复，那么在 GDB 中运行并将断点设置在对 `count` 变量的访问处，可能会帮助程序员发现问题。

以下是从此程序的 GDB 运行中提取的一些示例命令：

```
(gdb) break worker_loop     # Set a breakpoint for all spawned threads

(gdb) break 77 thread 4     # Set a breakpoint just for thread 4

(gdb) info threads          # List information about all threads

(gdb) where                 # List stack of thread that hit the breakpoint

(gdb) print i               # List values of its local variable i

(gdb) thread 2              # Switch to different thread's (2) context

(gdb) print i               # List thread 2's local variables i
```

以下示例展示了 `racecond.c` 程序的 GDB 运行的部分输出，程序有三个线程（`run 3`），并展示了在 GDB 调试会话中 GDB 线程命令的示例。主线程始终是 GDB 线程编号 1，三个派生线程是 GDB 线程 2 到 4。

在调试多线程程序时，GDB 用户必须跟踪当前存在哪些线程，然后再执行命令。例如，当 `main` 中的断点被触发时，只有线程 1（主线程）存在。因此，GDB 用户必须等待线程创建之后，才能为特定线程设置断点（本示例展示了在程序的第 77 行为线程 4 设置断点）。查看此输出时，请注意何时设置和删除断点，并注意每个线程的局部变量 `i` 的值，当线程上下文通过 GDB 的 `thread` 命令切换时：

```
$ gcc -g racecond.c -lpthread

$ gdb ./a.out

(gdb) break main

Breakpoint 1 at 0x919: file racecond.c, line 28.

(gdb) run 3

Starting program: ...

[Thread debugging using libthread_db enabled] ...

Breakpoint 1, main (argc=2, argv=0x7fffffffe388) at racecond.c:28

28     if (argc != 2) {

(gdb) list 76

71   myid = *((int *)arg);

72

73   printf("hello I'm thread %d with pthread_id %lu\n",

74       myid, pthread_self());

75

76   for (i = 0; i < 10000; i++) {

77       count += i;

78   }

79

80   return (void *)0;

(gdb) break 76

Breakpoint 2 at 0x555555554b06: file racecond.c, line 76.

(gdb) cont

Continuing.

[New Thread 0x7ffff77c4700 (LWP 5833)]

hello I'm thread 0 with pthread_id 140737345505024

[New Thread 0x7ffff6fc3700 (LWP 5834)]

hello I'm thread 1 with pthread_id 140737337112320

[New Thread 0x7ffff67c2700 (LWP 5835)]

[Switching to Thread 0x7ffff77c4700 (LWP 5833)]

Thread 2 "a.out" hit Breakpoint 2, worker_loop (arg=0x555555757280)

    at racecond.c:76

76   for (i = 0; i < 10000; i++) {

(gdb) delete 2

(gdb) break 77 thread 4

Breakpoint 3 at 0x555555554b0f: file racecond.c, line 77.

(gdb) cont

Continuing.

hello I'm thread 2 with pthread_id 140737328719616

[Switching to Thread 0x7ffff67c2700 (LWP 5835)]

Thread 4 "a.out" hit Breakpoint 3, worker_loop (arg=0x555555757288)

    at racecond.c:77

77       count += i;

(gdb) print i

$2 = 0

(gdb) cont

Continuing.

[Switching to Thread 0x7ffff67c2700 (LWP 5835)]

Thread 4 "a.out" hit Breakpoint 3, worker_loop (arg=0x555555757288)

    at racecond.c:77

77       count += i;

(gdb) print i

$4 = 1

(gdb) thread 3

[Switching to thread 3 (Thread 0x7ffff6fc3700 (LWP 5834))]

#0  0x0000555555554b12 in worker_loop (arg=0x555555757284) at racecond.c:77

77       count += i;

(gdb) print i

$5 = 0

(gdb) thread 2

[Switching to thread 2 (Thread 0x7ffff77c4700 (LWP 5833))]

#0  worker_loop (arg=0x555555757280) at racecond.c:77

77       count += i;

(gdb) print i

$6 = 1
```

### 3.7 总结

本章总结了我们对 C 编程语言的介绍。与其他高级编程语言相比，C 是一种相对较小的编程语言，只有少数几个基本构造，程序员通过这些构造来编写程序。由于 C 语言的抽象更接近计算机执行的底层机器代码，C 程序员可以编写比使用其他编程语言提供的高级抽象写出的等效代码更加高效的代码。特别是，C 程序员对程序如何使用内存有更多的控制权，这对程序的性能有重要影响。C 是计算机系统编程的语言，其中低级控制和效率至关重要。

在后续章节中，我们将使用 C 语言示例来说明计算机系统如何设计来运行程序。

### 注释

1. GDB 可通过 *[`www.gnu.org/software/gdb`](https://www.gnu.org/software/gdb)* 获得

2. Valgrind 可以在 *[`valgrind.org/info/tools.html`](https://valgrind.org/info/tools.html)* 获取。

3. Memcheck 工具可以在 *[`valgrind.org/docs/manual/mc-manual.html`](https://valgrind.org/docs/manual/mc-manual.html)* 获取。

4. *[`valgrind.org/docs/manual/cg-manual.html`](https://valgrind.org/docs/manual/cg-manual.html)*

5. *[`valgrind.org/docs/manual/cl-manual.html`](http://valgrind.org/docs/manual/cl-manual.html)*
