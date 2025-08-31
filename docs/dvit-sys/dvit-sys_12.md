# 第十三章：代码优化

![image](img/common.jpg)

*代码优化*是通过减少程序的代码大小、复杂度、内存使用或运行时间（或这些的某些组合）来改进程序的过程，而不改变程序的固有功能。许多编译系统将代码优化器作为中间步骤。具体来说，*优化编译器*在编译过程中应用改善代码的转换。几乎所有现代编译器（包括 GCC）都是优化编译器。GCC C 编译器实现了多种*优化标志*，为程序员提供了对已实现优化子集的直接访问。编译器优化标志会以牺牲编译时间和调试便利为代价来优化代码。为了简化，GCC 将这些优化标志的子集包装成不同的*优化等级*，供程序员直接调用。例如，以下命令使用等级 1 优化编译一个示例程序：

```
$ gcc -O1 -o program program.c
```

GCC 中的等级 1（`-O1` 或 `-O`）优化执行基本的优化，以减少代码大小和执行时间，同时尽量保持编译时间最小化。等级 2（`-O2`）优化包括 GCC 实现的大多数优化，这些优化不涉及空间与性能之间的权衡。最后，等级 3（`-O3`）执行额外的优化（如函数内联，稍后本章会讨论），并可能导致程序编译时间显著增加。GCC 文档^(1)详细描述了已实现的优化标志。

关于优化编译器及其构建和操作的详细讨论超出了本教材的范围；我们鼓励有兴趣的读者查阅 Aho、Sethi 和 Ulman 的经典著作《编译原理、技术与工具》。本章的目的是突出大多数编译器可以（以及不能）做的一些事情，以及程序员如何与编译器和性能分析工具合作来帮助改进他们的代码。

#### 编译器已完成的工作

几乎所有编译器执行的一些常见优化将在接下来的部分中简要描述。学生*永远不要*手动实现这些优化，因为它们已经由编译器实现。

##### 常量折叠

代码中的常量在编译时进行计算，以减少生成的指令数量。例如，在下面的代码片段中，*宏展开*将语句`int debug = N-5`替换为`int debug = 5-5`。*常量折叠*随后将该语句更新为`int debug = 0`。

```
#define N 5

int debug = N - 5; //constant folding changes this statement to debug = 0;
```

##### 常量传播

*常量传播*在编译时已知某个值的情况下，用常量值替换变量。考虑以下代码段：

```
int debug = 0;

//sums up all the elements in an array

int doubleSum(int *array, int length){

    int i, total = 0;

    for (i = 0; i < length; i++){

        total += array[i];

        if (debug) {

            printf("array[%d] is: %d\n", i, array[i]);

        }

    }

    return 2 * total;

}

```

一个采用常量传播的编译器会将`if (debug)`改为`if (0)`。

##### 死代码消除

程序中出现未使用的变量、赋值或语句并不罕见。尽管这些不需要的语句通常不是有意引入的，但它们往往是软件开发周期中不断迭代和优化的自然副产品。如果没有被发现，这些所谓的*死代码*序列可能会导致编译器输出不必要的汇编指令，从而浪费处理时间。大多数编译器采用数据流分析等技术来识别无法到达的代码段，并将其删除。*死代码消除*通常通过缩减代码大小和相关指令集来使程序运行得更快。例如，回到`doubleSum`函数，编译器利用常量传播将`debug`替换为`0`，从而简化了`if`语句：

```
int debug = 0;

//sums up all the elements in an array

int doubleSum(int *array, int length){

    int i, total = 0;

    for (i = 0; i < length; i++){

        total += array[i];

        if (0) { //debug replaced by constant propagation by compiler

            printf("array[%d] is: %d\n", i, array[i]);

        }

    }

    return 2 * total;

}
```

使用数据流分析的编译器能够识别出`if`语句总是评估为`false`，并且`printf`语句永远不会执行。因此，编译器会在编译后的可执行文件中去除`if`语句和对`printf`的调用。另一个步骤还会去除语句`debug = 0`。

##### 简化表达式

一些指令的执行成本高于其他指令。例如，汇编中的`imul`和`idiv`算术指令执行时需要较长时间。编译器通常会通过简化数学运算来减少高成本指令的数量。例如，在`doubleSum`函数中，编译器可能会将表达式`2 * total`替换为`total + total`，因为加法指令比乘法指令便宜：

```
//declaration of debug removed through dead-code elimination

//sums up all the elements in an array

int doubleSum(int *array, int length){

    int i, total = 0;

    for (i = 0; i < length; i++){

        total += array[i];

        //if statement removed through data-flow analysis

    }

    return total + total; //simplifying expression

}

```

同样，编译器会通过位移和其他按位运算符来转换代码序列，以简化表达式。例如，编译器可能会将表达式`total * 8`替换为`total << 3`，或将表达式`total % 8`替换为`total & 7`，因为按位运算可以通过一个快速指令执行。

#### 编译器无法始终做到的事情：学习代码优化的好处

尽管优化编译器有很多好处，但学习代码优化为何有用可能并不显而易见。可能会产生这样一种想法：编译器是一个“智能”的神奇黑盒。归根结底，编译器是一个执行一系列代码转换的软件，其目的是加速代码的执行。然而，编译器在其能够执行的优化类型上也存在限制。

##### 算法强度降低是不可能的

代码性能差的主要原因是选择了不合适的数据结构和算法。编译器无法神奇地修复这些糟糕的决策。例如，编译器永远不会将一个实现冒泡排序的程序优化成实现快速排序的程序。虽然编译器及其优化的复杂度不断提高，但任何单一编译器的优化*质量*在不同平台之间是不同的。因此，责任在于程序员，确保其代码利用了最佳的算法和数据结构。

##### 编译器优化标志不能保证使代码“最优”（或一致）

提高编译器优化级别（例如，从`-O2`到`-O3`）并不总是能减少程序的运行时间。有时，程序员可能会发现将优化标志从`-O2`更新到`-O3`反而*降低了*程序的速度，或者根本没有任何性能提升。在其他情况下，程序员可能会发现一个没有优化标志编译的程序似乎没有任何错误，而使用`-O2`或`-O3`编译时却会导致段错误或其他错误。这类编程错误尤其难以调试，因为 gcc 的调试（`-g`）标志与其优化（`-O`）标志不兼容，因为编译器优化在`-O`级别上进行的转换会干扰调试器分析底层代码的能力。许多常见的性能分析工具（如 GDB 和 Valgrind）都要求使用`-g`标志。

行为不一致的一个重要原因是，C/C++标准并未提供明确的指南来解决未定义行为。因此，通常由编译器决定如何解决歧义。不同优化级别如何处理未定义行为的不一致可能导致结果发生*变化*。考虑以下 John Regehr 的例子：^(2)

```
int silly(int a) {

  return (a + 1) > a;

}

```

假设` silly `是用` a = INT_MAX `运行的。在这种情况下，计算` a + 1 `会导致整数溢出。然而，C/C++标准并没有定义*编译器应该如何处理*整数溢出。事实上，在没有优化的情况下编译程序会使得函数返回 0，而使用`-O3`优化编译时，函数则返回 1。

简而言之，优化标志应该谨慎使用，经过深思熟虑，并在必要时使用。学习使用哪些优化标志也可以帮助程序员与编译器合作，而不是与编译器对抗。

**注意 编译器不要求处理未定义行为**

当`a = INT_MAX`时运行的`silly`函数是未定义行为的一个例子。请注意，编译器生成的不一致输出并不是编译器设计上的缺陷，也不是使用优化标志的结果。编译器专门设计来遵循语言规范。C 语言标准并没有规定当编译器遇到未定义行为时应该做什么；程序可能崩溃、无法编译，或生成不一致或错误的结果。最终，程序员负责识别并消除代码中的未定义行为。`silly`应该返回 0、1 还是其他值，最终是程序员必须做出的决定。欲了解更多关于未定义行为和 C 程序相关问题的信息，请访问 C 语言 FAQ^(3)或 John Regehr 的《未定义行为指南》^(4)。

##### 指针可能会引发问题

回想一下，编译器进行的转换会保持源程序的基本行为不变。如果某个转换可能改变程序的行为，编译器将不会进行该转换。特别是在*内存别名*的情况下，即两个不同的指针指向内存中的相同地址，编译器尤其会遵守这一规则。举个例子，考虑函数`shiftAdd`，它的两个参数是两个整数指针。该函数将第一个数字乘以 10，并将第二个数字加到其中。所以，如果`shiftAdd`函数传入整数 5 和 6，结果将是 56。

未优化版本

```
void shiftAdd(int *a, int *b){

    *a = *a * 10; //multiply by 10

    *a += *b; //add b

}

```

优化版本

```
void shiftAddOpt(int *a, int *b){

    *a = (*a * 10) + *b;

}

```

`shiftAddOpt`函数通过去除对`a`的额外内存引用来优化`shiftAdd`函数，从而在编译的汇编代码中生成较小的指令集。然而，由于内存别名的风险，编译器永远不会做这种优化。为了理解原因，请考虑以下的`main`函数：

```
int main(void){

    int x = 5;

    int y = 6;

    shiftAdd(&x, &y); //should produce 56

    printf("shiftAdd produces: %d\n", x);

    x = 5; //reset x

    shiftAddOpt(&x, &y); //should produce 56

    printf("shiftAddOpt produces: %d\n", x);

    return 0;

}
```

编译并运行此程序会得到预期输出：

```
$ gcc -o shiftadd shiftadd.c

$ ./shiftadd

shiftAdd produces: 56

shiftAddOpt produces: 56

```

假设程序被修改为使得`shiftAdd`现在接收指向`x`的指针作为它的两个参数：

```
int main(void){

    int x = 5;

    shiftAdd(&x, &x); //should produce 55

    printf("shiftAdd produces: %d\n", x);

    x = 5; //reset x

    shiftAddOpt(&x, &x); //should produce 55

    printf("shiftAddOpt produces: %d\n", x);

    return 0;

}

```

预期输出是 55。 然而，重新编译并重新运行更新后的代码会得到两个不同的输出：

```
$ gcc -o shiftadd shiftadd.c

$ ./shiftadd

shiftAdd produces: 100

shiftAddOpt produces: 55

```

回溯通过假设`a`和`b`指向相同的内存位置的`shiftAdd`函数可以揭示问题。在`shiftAdd`中将`a`乘以 10 会将`x`更新为 50。接下来，在`shiftAdd`中将`a`加到`b`中，会使得`x`翻倍变为 100。内存别名的风险表明，尽管程序员可能希望它们是等效的，但`shiftAdd`和`shiftAddOpt`实际上并不相同。为了解决这个问题，应该认识到`shiftAdd`的第二个参数不需要作为指针传递。用整数替换第二个参数可以消除别名风险，并允许编译器将一个函数优化为另一个函数：

未优化版本（已修复）

```
void shiftAdd(int *a, int b){

    *a = *a * 10; //multiply by 10

    *a += b; //add b

}

```

优化版本（已修复）

```
void shiftAddOpt(int *a, int b){

    *a = (*a * 10) + b;

}

```

移除不必要的内存引用使得程序员能够保持原始`shiftAdd`函数的可读性，同时使编译器能够优化该函数。

#### 与编译器的合作：一个示例程序

在接下来的章节中，我们将重点学习更多常见的优化类型，并讨论编程和性能分析策略，以帮助编译器更容易地优化我们的代码。为了说明我们的讨论，我们将致力于优化以下（编写不够优化的）程序，该程序尝试查找 2 到*n*之间的所有质数：^(5)

optExample.c

```
//helper function: checks to see if a number is prime

int isPrime(int x) {

    int i;

    for (i = 2; i < sqrt(x) + 1; i++) { //no prime number is less than 2

        if (x % i == 0) { //if the number is divisible by i

            return 0; //it is not prime

        }

    }

    return 1; //otherwise it is prime

}

// finds the next prime

int getNextPrime(int prev) {

    int next = prev + 1;

    while (!isPrime(next)) { //while the number is not prime

        next++; //increment and check again

    }

    return next;

}

// generates a sequence of primes

int genPrimeSequence(int *array, int limit) {

    int i;

    int len = limit;

    if (len == 0) return 0;

    array[0] = 2; //initialize the first number to 2

    for (i = 1; i < len; i++) {

        array[i] = getNextPrime(array[i-1]); //fill in the array

        if (array[i] > limit) {

            len = i;

            return len;

        }

    }

    return len;

}

int main(int argc, char **argv) {

  //error-handling and timing code omitted for brevity

  int *array = allocateArray(limit);

  int length = genPrimeSequence(array, limit);

  return 0;

}
```

表 12-1 展示了使用以下基本编译命令，在不同优化级别标志下生成 2 到 5,000,000 之间质数的时间结果：

```
$ gcc -o optExample optExample.c -lm
```

**表 12-1：** 生成 2 到 5,000,000 之间质数的时间（单位：秒）

| **未优化** | -O1 | -O2 | -O3 |
| --- | --- | --- | --- |
| 3.86 | 2.32 | 2.14 | 2.15 |

使用优化标志观察到的最快时间约为 2.14 秒。虽然使用优化标志可以将程序运行时间缩短超过一秒，但提高优化标志的级别所带来的改进几乎可以忽略不计。在接下来的章节中，我们将讨论如何修改程序，使编译器更容易进行优化。

### 12.1 代码优化第一步：代码性能分析

*真正的问题是程序员在错误的地方和错误的时间花费了太多时间去担心效率；过早优化是编程中所有问题的根源（至少是大多数问题的根源）。*

—唐纳德·克努斯，*计算机程序设计的艺术*

代码优化中最大的危险之一是*过早优化*的概念。过早优化指的是程序员基于“直觉”而非数据来优化，试图在性能低效的地方进行优化。尽可能地，在开始优化之前，通过测量不同代码部分在不同输入下的运行时间，识别出*热点*或者程序中执行最多指令的区域。

为了找出如何优化`optExample.c`，我们首先来仔细看看`main`函数：

```
int main(int argc, char **argv) {

    //error-handling and timing code omitted for brevity

    int limit = strtol(argv[1], NULL, 10);

    int length = limit;

    int *array = allocateArray(length); //allocates array of specified length

    genPrimeSequence(array, limit, &length); //generates sequence of primes

    return 0;

}

```

`main`函数包含对两个函数的调用：`allocateArray`，它初始化一个用户指定长度（或限制）的数组；`genPrimeSequence`，它生成在指定限制内的质数序列（请注意，在 2 和*n*之间的任何序列中，质数的数量不能超过*n*，并且通常质数的数量要少得多）。`main`函数还包含计时每个函数运行时间的代码。将代码编译并运行，`limit`设置为 5,000,000 时，结果如下：

```
$ gcc -o optExample optExample.c -lm

$ time -p ./optExample 5000000

Time to allocate: 5.5e-05

Time to generate primes: 3.85525

348513 primes found.

real 3.85

user 3.86

sys 0.00
```

`optExample` 程序大约需要 3.86 秒才能完成，几乎所有时间都花费在 `genPrimeSequence` 函数中。没有必要花时间优化 `allocateArray`，因为对整个程序运行时间的改进微乎其微。在接下来的示例中，我们将更加关注 `genPrimeSequence` 函数及其相关函数。为了方便起见，以下是这些函数的代码：

```
// helper function: checks to see if a number is prime

int isPrime(int x) {

    int i;

    for (i = 2; i < sqrt(x) + 1; i++) { //no prime number is less than 2

        if (x % i == 0) { //if the number is divisible by i

            return 0; //it is not prime

        }

    }

    return 1; //otherwise it is prime

}

// finds the next prime

int getNextPrime(int prev) {

    int next = prev + 1;

    while (!isPrime(next)) { //while the number is not prime

        next++; //increment and check again

    }

    return next;

}

// generates a sequence of primes

int genPrimeSequence(int *array, int limit) {

    int i;

    int len = limit;

    if (len == 0) return 0;

    array[0] = 2; //initialize the first number to 2

    for (i = 1; i < len; i++) {

        array[i] = getNextPrime(array[i-1]); //fill in the array

        if (array[i] > limit) {

            len = i;

            return len;

        }

    }

    return len;

}

```

要找出程序中的热点，可以关注包含最多循环的区域。代码的手动检查有助于定位热点，但在尝试优化之前，始终应通过基准测试工具进行验证。对 `optExample` 程序的手动检查得出以下观察结果。

`genPrimeSequence` 函数尝试生成从 2 到某个整数 *n* 之间的所有素数。由于 2 和 *n* 之间的素数个数不能超过 *n*，因此 `genPrimeSequence` 中的 `for` 循环最多执行 *n* 次。每次 `for` 循环迭代都会调用一次 `getNextPrime` 函数。因此，`getNextPrime` 函数最多执行 *n* 次。

`getNextPrime` 函数中的 `while` 循环将持续运行，直到发现一个素数。虽然很难事先根据 *n*（连续素数之间的差距可能非常大）来确定 `while` 循环将执行多少次，但可以确定的是，`isPrime` 会在每次 `while` 循环迭代时执行。

`isPrime` 函数包含一个 `for` 循环。假设该循环总共执行 *k* 次迭代。那么，循环体中的代码将执行 *k* 次。回顾在《C 语言中的循环》一章中的内容，第 33 页提到，`for` 循环的结构包括一个 *初始化语句*（用于将循环变量初始化为特定值），一个 *布尔表达式*（用于确定何时终止循环），以及一个 *步进表达式*（用于在每次迭代时更新循环变量）。表 12-2 显示了在执行 *k* 次迭代的 `for` 循环中，各个循环组件执行的次数。在每个 `for` 循环中，初始化只会执行一次。布尔表达式会执行 *k* + 1 次，因为它必须进行一次最终检查以终止循环。循环体和步进表达式各执行 *k* 次。

**表 12-2：** 循环执行组件（假设执行 k 次迭代）

| **初始化语句** | **布尔表达式** | **步进表达式** | **循环体** |
| --- | --- | --- | --- |
| 1 | *k* + 1 | *k* | *k* |

我们对代码的手动检查表明，程序大部分时间都花费在 `isPrime` 函数中，且 `sqrt` 函数执行的次数最多。接下来，让我们使用代码分析工具来验证这个假设。

#### 12.1.1 使用 Callgrind 进行性能分析

在我们的小程序中，通过手动检查相对简单地形成假设，认为`sqrt`函数出现在代码的“热点”中。然而，在较大的程序中，识别热点可能变得更加复杂。无论如何，使用性能分析工具来验证我们的假设是个好主意。像 Valgrind^(6)这样的代码性能分析工具提供了大量关于程序执行的信息。在这一节中，我们使用`callgrind`工具来检查`OptExample`程序的调用图。

为了使用`callgrind`，我们先通过`-g`标志重新编译`optExample`程序，并在较小的范围（2 到 100,000）内运行`callgrind`。像其他 Valgrind 应用程序一样，`callgrind`作为一个包装器运行在程序周围，添加注释信息，例如函数执行的次数和执行的总指令数。因此，当与`callgrind`一起运行时，`optExample`程序的执行时间会更长。

```
$ gcc -g -o optExample optExample.c -lm

$ valgrind --tool=callgrind ./optExample 100000

==32590== Callgrind, a call-graph generating cache profiler

==32590== Copyright (C) 2002-2015, and GNU GPL'd, by Josef Weidendorfer et al.

==32590== Using Valgrind-3.11.0 and LibVEX; rerun with -h for copyright info

==32590== Command: ./optExample 100000

==32590==

==32590== For interactive control, run 'callgrind_control -h'.

Time to allocate: 0.003869

Time to generate primes: 0.644743

9592 primes found.

==32590==

==32590== Events    : Ir

==32590== Collected : 68338759

==32590==

==32590== I   refs:      68,338,759

```

在终端输入`ls`会显示一个新文件，名为`callgrind.out.xxxxx`，其中`xxxxx`是唯一的标识符。在这种情况下，文件名为`callgrind.out.32590`（即前面输出中左侧列显示的数字）。对这个文件运行`callgrind_annotate`会生成更多关于三个感兴趣函数的信息：

```
$ callgrind_annotate --auto=yes callgrind.out.32590

 ----------------------------------------------------------------

Profile data file 'callgrind.out.32393' (creator: callgrind-3.11.0)

 ----------------------------------------------------------------

...

  .  //helper function: checks to see if a number is prime

   400,004  int isPrime(int x) {

         .      int i;

36,047,657      for (i = 2; i < sqrt(x)+1; i++) { //no prime is less than 2

13,826,015  => ???:sqrt (2765204x)

16,533,672          if (x % i == 0) { //if the number is divisible by i

   180,818              return 0; //it is not prime

         .          }

         .      }

     9,592      return 1; //otherwise it is prime

   200,002  }

         .

         .  // finds the next prime

    38,368  int getNextPrime(int prev) {

    28,776      int next = prev + 1;

   509,597      while (!isPrime(next)) { //while the number is not prime

67,198,556  => optExample.c:isPrime (100001x)

    90,409          next++; //increment and check again

         .      }

     9,592      return next;

    19,184  }

         .

         .  // generates a sequence of primes

         6  int genPrimeSequence(int * array, int limit) {

         .      int i;

         2      int len = limit;

         2      if (len == 0) return 0;

         2      array[0]=2; //initialize the first number to 2

    38,369      for (i = 1; i < len; i++) {

   143,880          array[i] = getNextPrime(array[i-1]); //fill in the array

67,894,482  => optExample.c:getNextPrime (9592x)

    76,736          if (array[i] > limit){

         2              len = i;

         2              return len;

         .          }

         .      }

         .      return len;

         4  }

```

左侧列的数字表示与每一行相关联的总执行指令数。括号中的数字表示某个特定函数运行的次数。通过左侧列的数字，我们能够验证手动检查的结果。在`genPrimeSequence`函数中，`getNextPrime`函数执行的指令最多，总计 6,780 万条指令，对应 9,592 次函数调用（用于生成 2 到 100,000 之间的素数）。检查`getNextPrime`时发现，大部分指令（6,710 万条，占 99%）是来自对`isPrime`的调用，而`isPrime`总共被调用了 100,001 次。最后，检查`isPrime`时发现，总指令数中有 1,300 万条（占 20.5%）来自`sqrt`函数，`sqrt`函数总共执行了 270 万次。

这些结果验证了我们最初的假设：程序大部分时间都花费在`isPrime`函数中，并且`sqrt`函数是所有函数中执行最频繁的。减少执行的总指令数会使程序更快；上述分析表明，我们的初步努力应集中在改进`isPrime`函数，并可能减少`sqrt`执行的次数。

#### 12.1.2 循环不变代码移动

循环不变代码移动是一种优化技术，它将发生在循环内部的静态计算移到循环外部，同时不影响循环的行为。优化编译器能够自动执行大多数循环不变代码优化。具体来说，GCC 中的`-fmove-loop-invariants`编译器标志（在`-O1`级别启用）尝试识别循环不变代码移动的例子，并将它们移到各自的循环外部。

然而，编译器并不能总是识别循环不变代码移动的情况，尤其是在函数调用的情况下。由于函数调用可能会不小心引起*副作用*（意外的行为），大多数编译器会避免尝试判断一个函数调用是否始终返回相同的结果。因此，即使程序员知道`sqrt(x)`始终返回某个输入`x`的平方根，GCC 也不会总是做出这个假设。考虑到`sqrt`函数更新了一个秘密的全局变量`g`，在这种情况下，在函数外调用一次`sqrt`（*对 g 进行一次*更新）与在循环的每次迭代中调用它（*对 g 进行 n 次*更新）是不同的。如果编译器无法确定一个函数始终返回相同的结果，它就不会自动将`sqrt`函数移到循环外部。

然而，程序员知道将计算`sqrt(x) + 1`移到`for`循环外部不会影响循环的行为。更新后的函数在这里展示，并且可以在线查看：^(7)

```
//helper function: checks to see if a number is prime

int isPrime(int x) {

    int i;

    int max = sqrt(x)+1;

    for (i = 2; i < max; i++) { //no prime number is less than 2

        if (x % i == 0) { //if the number is divisible by i

            return 0; //it is not prime

        }

    }

    return 1; //otherwise it is prime

}

```

表 12-3 显示，这一简单的更改使得`optExample2`的运行时间减少了整整两秒（47%），即使在没有使用编译器标志的情况下。此外，编译器似乎更容易优化`optExample2`。

**表 12-3：** 计算 2 到 5,000,000 之间素数所需的时间（单位：秒）

| **版本** | **未优化** | -O1 | -O2 | -O3 |
| --- | --- | --- | --- | --- |
| 原始 | 3.86 | 2.32 | 2.14 | 2.15 |
| 使用循环不变代码移动 | 1.83 | 1.63 | 1.71 | 1.63 |

重新运行`callgrind`在`optExample2`可执行文件上的分析可以揭示为什么观察到了如此大的运行时改进。以下代码片段假设文件`callgrind.out.30086`包含了运行`callgrind`在`optExample2`可执行文件上的注释：

```
$ gcc -g -o optExample2 optExample2.c -lm

$ valgrind --tool=callgrind ./optExample2 100000

$ callgrind_annotate --auto=yes callgrind.out.30086

 ------------------------------------------------------------------

Profile data file 'callgrind.out.30086' (creator: callgrind-3.11.0)

 ------------------------------------------------------------------

 ...

   400,004  int isPrime(int x) {

         .      int i;

   900,013      int max = sqrt(x)+1;

   500,000  => ???:sqrt (100001x)

11,122,449      for (i = 2; i < max; i++) { //no prime number is less than 2

16,476,120          if (x % i == 0) { //if the number is divisible by i

   180,818              return 0; //it is not prime

         .          }

         .      }

     9,592      return 1; //otherwise it is prime

   200,002  }

         .

         .  // finds the next prime

    38,368  int getNextPrime(int prev) {

    28,776      int next = prev + 1;

   509,597      while (!isPrime(next)) { //while the number is not prime

29,789,794  => optExample2.c:isPrime (100001x)

    90,409          next++; //increment and check again

         .      }

     9,592      return next;

    19,184  }

```

将对`sqrt`的调用移到`for`循环外部，将程序中`sqrt`函数的调用次数从 270 万次减少到 10 万次（减少 96%）。这个数字对应于`isPrime`函数的调用次数，确认了每次调用`isPrime`函数时，`sqrt`函数只执行一次。

请注意，即使程序员没有手动执行代码运动，编译器在指定优化标志时也能够执行显著的优化。在这种情况下，原因是 x86 指令集架构（ISA）中有一个特殊的指令 `fsqrt`。当启用优化标志时，编译器会将所有 `sqrt` 函数的调用替换为 `fsqrt` 指令。这一过程被称为 *内联*，我们将在接下来的章节中详细介绍。由于 `fsqrt` 不再是一个函数，它更容易被编译器识别为循环不变的代码，从而将其移出循环体外。

### 12.2 其他编译器优化：循环展开与函数内联

前一节中描述的循环不变代码运动优化是一个简单的改动，导致了执行时间的显著减少。然而，这种优化是依赖于特定情况的，并不总是能带来性能提升。在大多数情况下，循环不变代码运动由编译器自动处理。

今天，代码更多的是被阅读而不是编写。在大多数情况下，微小的性能提升并不值得为了提高性能而牺牲代码的可读性。一般来说，程序员应该尽可能让编译器进行优化。在本节中，我们将介绍一些过去由程序员手动实现的优化技术，但如今这些技术通常由编译器实现。

网上有一些资源提倡手动实现我们在以下章节中描述的技术。然而，我们建议读者在尝试手动实现这些优化之前，先检查他们的编译器是否支持这些优化。本文所述的所有优化在 GCC 中都有实现，但在旧版编译器中可能不可用。

#### 12.2.1 函数内联

编译器尝试执行的一个优化步骤是 *函数内联*，它将对函数的调用替换为函数体的内容。例如，在 `main` 函数中，如果编译器内联 `allocateArray` 函数，它将把对 `allocateArray` 的调用替换为直接调用 `malloc`：

原始版本

```
int main(int argc, char **argv) {

    // omitted for brevity

    // some variables shortened for space considerations

    int lim = strtol(argv[1], NULL, 10);

    // allocation of array

    int *a = allocateArray(lim);

    // generates sequence of primes

    int len = genPrimeSequence(a, lim);

    return 0;

}

```

将 `allocateArray` 内联

```
int main(int argc, char **argv) {

    // omitted for brevity

    // some variables shortened for space considerations

    int lim = strtol(argv[1], NULL, 10);

    // allocation of array (in-lined)

    int *a = malloc(lim * sizeof(int));

    // generates sequence of primes

    int len = genPrimeSequence(a, lim);

    return 0;

}

```

函数内联可以为程序带来一定的运行时节省。回想一下，每当程序调用一个函数时，都会生成与函数创建和销毁相关的许多指令。函数内联使编译器能够消除这些多余的调用，并且使编译器更容易识别其他潜在的优化，包括常量传播、常量折叠和死代码消除。在 `optExample` 程序中，函数内联可能使编译器将对 `sqrt` 的调用替换为 `fsqrt` 指令，并将其移出循环。

`-finline-functions`标志向 GCC 建议应该进行函数内联。这项优化在级别 3 时启用。即使`-finline-functions`可以独立于`-O3`标志使用，它仍然是向编译器提出的 *建议*，让编译器寻找可以内联的函数。同样，`static inline`关键字也可以用于向编译器建议某个特定函数应该被内联。请记住，编译器并不会内联所有函数，而且函数内联并不一定会使代码变得更快。

程序员通常应避免手动内联函数。内联函数的风险较高，可能会显著降低代码的可读性，增加出错的可能性，并使得更新和维护函数变得更加困难。例如，尝试将`isPrime`函数内联到`getNextPrime`函数中，将大大降低`getNextPrime`的可读性。

#### 12.2.2 循环展开

我们在本节讨论的最后一个编译器优化策略是循环展开。让我们重新审视一下`isPrime`函数：

```
// helper function: checks to see if a number is prime

int isPrime(int x) {

    int i;

    int max = sqrt(x) + 1;

    // no prime number is less than 2

    for (i = 2; i < max; i++) {

        // if the number is divisible by i

        if (x % i == 0) {

            return 0; // it's not prime

        }

    }

    return 1; // otherwise it is

}

```

`for`循环总共执行`max`次，其中`max`是整数`x`的平方根加一。在汇编级别，每次执行循环时都会检查`i`是否小于`max`。如果是，指令指针跳转到循环体，计算模运算。如果模运算结果为 0，程序立即退出循环并返回 0。否则，循环继续执行。虽然分支预测器在预测条件表达式的结果时表现得相当不错（尤其是在循环内），但错误的猜测可能会导致性能下降，因为指令流水线可能会中断。

*循环展开*是一种编译器执行的优化，用来减少错误猜测的影响。在循环展开中，目标是通过增加每次迭代的工作量来减少循环的迭代次数，通常按 *n* 的倍数来进行。当一个循环按 2 的倍数展开时，循环中的迭代次数将减少 *一半*，而每次迭代所执行的工作量将 *翻倍*。

让我们手动将 2 倍循环展开应用到`isPrime`函数中：^(8)

```
// helper function: checks to see if a number is prime

int isPrime(int x) {

    int i;

    int max = sqrt(x)+1;

    // no prime number is less than 2

    for (i = 2; i < max; i+=2) {

        // if the number is divisible by i or i+1

        if ( (x % i == 0) || (x % (i+1) == 0) ) {

            return 0; // it's not prime

        }

    }

    return 1; // otherwise it is

}

```

注意，尽管我们已经将`for`循环的迭代次数减少了一半，但每次迭代现在执行了两个模运算，导致每次迭代的工作量 *翻倍*。重新编译并重新运行程序后，结果是时间略有改进（参见表 12-4）。

代码的可读性也因此降低。更好的使用循环展开的方法是调用`-funroll-loops`编译器优化标志，这个标志告诉编译器展开那些可以在编译时确定迭代次数的循环。`-funroll-all-loops`编译器标志是一个更激进的选项，它会展开所有循环，无论编译器是否确定迭代次数。表 12-4 展示了手动 2 因子循环展开^(9)与将`-funroll-loops`和`-funroll-all-loops`编译器优化标志添加到之前程序中的运行时对比。

**表 12-4：** 生成 5,000,000 个质数所需的时间（秒）

| **版本** | **文件** | **未优化** | -O1 | -O2 | -O3 |
| --- | --- | --- | --- | --- | --- |
| 原始版本 | `optExample.c` | 3.86 | 2.32 | 2.14 | 2.15 |
| 循环不变代码移动 | `optExample2.c` | 1.83 | 1.63 | 1.71 | 1.63 |
| 手动二分循环 | `optExample3.c` | 1.65 | 1.53 | 1.45 | 1.45 |
| 循环展开 |
| `-funroll-loops` | `optExample2.c` | 1.82 | 1.48 | 1.46 | 1.46 |
| `-funroll-all-loops` | `optExample2.c` | 1.81 | 1.47 | 1.47 | 1.46 |

手动循环展开确实能带来一些性能提升；然而，当编译器的内建循环展开标志与其他优化标志结合使用时，能够实现相当的性能表现。如果程序员希望将循环展开优化融入到他们的代码中，应该默认使用适当的编译器标志，而*不*手动展开循环。

### 12.3 内存考虑

程序员应特别注意内存使用，特别是在使用像矩阵和数组这样的内存密集型数据结构时。尽管编译器提供了强大的优化功能，但编译器并非总是能做出改善程序内存使用的优化。在本节中，我们使用一个矩阵-向量程序`matrixVector.c`的实现^(10)来引导讨论提高内存使用的技术和工具。

程序的`main`函数执行两个步骤。首先，它分配并初始化输入矩阵、输入向量和输出矩阵。接下来，它执行矩阵-向量乘法。在矩阵-向量维度为 10,000 × 10,000 时运行代码，结果表明`matrixVectorMultiply`函数占用了大部分时间：

```
$ gcc -o matrixVector matrixVector.c

$ ./matrixVector 10000 10000

Time to allocate and fill matrices: 1.2827

Time to allocate vector: 9.6e-05

Time to matrix-vector multiply: 1.98402
```

因此，我们的讨论将集中在`matrixVectorMultiply`函数上。

#### 12.3.1 循环交换

循环交换优化通过交换嵌套循环中的内外循环顺序，以最大化缓存局部性。自动执行这个任务对编译器来说是比较困难的。在 GCC 中，存在`-floop-interchange`编译器标志，但目前默认情况下不可用。因此，程序员应该特别注意代码如何访问内存组合数据结构，如数组和矩阵。作为示例，让我们仔细看看`matrixVector.c`中的`matrixVectorMultiply`函数：

原始版本

```
void matrixVectorMultiply(int **m,

                          int *v,

                          int **res,

                          int row,

                          int col) {

    int i, j;

    //cycles through every matrix column

    //in inner-most loop (inefficient)

    for (j = 0; j < col; j++){

        for (i = 0; i < row; i++){

            res[i][j] = m[i][j] * v[j];

        }

    }

}

```

循环交换版本

```
void matrixVectorMultiply(int **m,

                          int *v,

                          int **res,

                          int row,

                          int col) {

    int i, j;

    //cycles through every row of matrix

    //in inner-most loop

    for (i = 0; i < row; i++){

        for (j = 0; j < col; j++){

            res[i][j] = m[i][j] * v[j];

        }

    }

}

```

输入和输出矩阵是动态分配的（参见“方法 2：程序员友好方式”，第 90 页）。因此，矩阵中的行并不彼此连续，而每行中的元素是连续的。当前的循环顺序导致程序遍历每一列，而不是每一行。请记住，数据是以块而非元素的形式加载到缓存中的（参见“直接映射缓存”，第 558 页）。因此，当访问`res`或`m`数组中的元素*x*时，*与* x 相邻的元素也会被加载到缓存中。遍历矩阵的每一“列”会导致更多的缓存未命中，因为每次访问时缓存都需要加载新的数据块。表 12-5 显示，添加优化标志并不会减少函数的运行时间。然而，仅仅交换循环的顺序（如前面的代码示例所示）使得函数速度几乎提高了八倍，并且允许编译器执行额外的优化。

**表 12-5：** 在 10,000 × 10,000 元素上执行矩阵乘法的秒数

| **版本** | **程序** | **未优化** | -O1 | -O2 | -O3 |
| --- | --- | --- | --- | --- | --- |
| 原始 | `matrixVector` | 2.01 | 2.05 | 2.07 | 2.08 |
| 循环交换 | `matrixVector2` | 0.27 | 0.08 | 0.06 | 0.06 |

Valgrind 工具 `cachegrind`（在“缓存分析与 Valgrind”中讨论，参见 第 575 页）是识别数据局部性问题的一个很好的工具，并揭示了在前面示例中展示的两种 `matrixVectorMultiply` 函数版本之间的缓存访问差异。

#### 12.3.2 改善局部性的一些其他编译器优化：分裂与融合

重新运行改进后的程序，使用 10,000 × 10,000 个元素，得到以下运行时间数据：

```
$ gcc -o matrixVector2 matrixVector2.c

$ ./matrixVector2 10000 10000

Time to allocate and fill matrices: 1.29203

Time to allocate vector: 0.000107

Time to matrix-vector multiply: 0.271369
```

现在，矩阵的分配和填充占用了最多的时间。进一步的计时显示，实际上是矩阵的填充过程花费了最多时间。我们来仔细看看这段代码：

```
//fill matrices

for (i = 0; i < rows; i++){

    fillArrayRandom(matrix[i], cols);

    fillArrayZeros(result[i], cols);

}

```

为了填充输入和输出矩阵，一个`for`循环遍历所有的行，并在每个矩阵上调用`fillArrayRandom`和`fillArrayZeros`函数。在某些情况下，编译器将单个循环拆分成两个独立的循环（称为*循环分裂*）可能是有利的，如下所示：

原始版本

```
for (i = 0; i < rows; i++) {

    fillArrayRandom(matrix[i], cols);

    fillArrayZeros(result[i], cols);

 } 
```

使用循环分裂

```
for (i = 0; i < rows; i++) {

    fillArrayRandom(matrix[i], cols);

}

for (i = 0; i < rows; i++) {

    fillArrayZeros(result[i], cols);

}

```

将两个对相同范围操作的循环合并成一个循环的过程（即循环裂解的反过程）被称为*循环融合*。循环裂解和融合是编译器可能进行的优化示例，目的是提高数据局部性。多核处理器的编译器还可能使用循环裂解或融合，使得循环能够在多个核心上高效执行。例如，编译器可能使用循环裂解将两个循环分配给不同的核心。同样，编译器可能使用循环融合将相关操作合并到循环体内，并将循环迭代的子集分配到每个核心（假设各次迭代之间的数据是独立的）。

在我们的案例中，手动应用循环裂解并不会直接改善程序性能；填充数组所需的时间几乎没有变化。然而，这可能揭示一个更微妙的优化：包含`fillArrayZeros`的循环是多余的。`matrixVectorMultiply`函数已将值分配给`result`数组中的每个元素，因此事先将其初始化为零是不必要的。

旧版本 matrixVector2.c

```
for (i = 0; i < rows; i++) {

    matrix[i] = allocateArray(cols);

    result[i] = allocateArray(cols);

}

for (i = 0; i < rows; i++) {

    fillArrayRandom(matrix[i], cols);

    fillArrayZeros(result[i], cols);

}

```

更新后的版本 matrixVector3.c

```
for (i = 0; i < rows; i++) {

    matrix[i] = allocateArray(cols);

    result[i] = allocateArray(cols);

}

for (i = 0; i < rows; i++) {

    fillArrayRandom(matrix[i], cols);

    //fillArrayZeros(result[i], cols); //no longer needed

}

```

#### 12.3.3 使用 Massif 进行内存分析

做出前述更改后，运行时间仅略微下降。尽管它消除了用零填充结果矩阵中所有元素的步骤，但仍需要相当多的时间来用随机数填充输入矩阵：

```
$ gcc -o matrixVector3 matrixVector3.c

$ ./matrixVector3 10000 10000

Time to allocate matrices: 0.049073

Time to fill matrices: 0.946801

Time to allocate vector: 9.3e-05

Time to matrix-vector multiply: 0.359525
```

尽管每个数组在内存中是非连续存储的，但每个数组占用 10,000 × `sizeof(int)`字节，即 40,000 字节。由于一共分配了 20,000 个数组（初始矩阵和结果矩阵各 10,000 个），这对应于 8 亿字节，约合 762 MB 的空间。用随机数填充 762 MB 显然需要大量时间。对于矩阵而言，内存使用量随着输入规模的增大呈平方增长，因此在性能中可能起到重要作用。

Valgrind 的`massif`工具可以帮助你分析内存使用情况。与本书中我们介绍的其他 Valgrind 工具（请参见第 168 页的“使用 Valgrind 调试内存”、第 575 页的“缓存分析与 Valgrind”以及第 600 页的“使用 Callgrind 进行性能分析”）类似，`massif`作为程序可执行文件的包装器运行。具体而言，`massif`会在程序运行过程中拍摄程序内存使用情况的快照，并分析内存使用的波动。程序员可能会发现`massif`工具对于跟踪程序如何使用堆内存，以及识别改善内存使用的机会非常有用。让我们在`matrixVector3`可执行文件上运行`massif`工具：

```
$ valgrind --tool=massif ./matrixVector3 10000 10000

==7030== Massif, a heap profiler

==7030== Copyright (C) 2003-2015, and GNU GPL'd, by Nicholas Nethercote

==7030== Using Valgrind-3.11.0 and LibVEX; rerun with -h for copyright info

==7030== Command: ./matrixVector3 10000 10000

==7030==

Time to allocate matrices: 0.049511

Time to fill matrices: 4.31627

Time to allocate vector: 0.001015

Time to matrix-vector multiply: 0.62672

==7030==
```

运行`massif`后会生成一个`massif.out.xxxx`文件，其中`xxxx`是唯一的 ID 编号。如果你正在跟随输入，可以输入 ls 命令查看对应的 massif 文件。在以下示例中，相关文件为`massif.out.7030`。使用 ms_print 命令查看`massif`输出：

```
$ ms_print massif.out.7030

-----------------------------------------------------------------------------

Command:            ./matrixVector3 10000 10000

Massif arguments:   (none)

ms_print arguments: massif.out.7030

-----------------------------------------------------------------------------

    MB

763.3^                                                ::::::::::::::::::::::#

     |:::::::::::::::::::::::::::::::::::::::::::::::::                     #

     |:                                               :                     #

     |@                                               :                     #

     |@                                               :                     #

     |@                                               :                     #

     |@                                               :                     #

     |@                                               :                     #

     |@                                               :                     #

     |@                                               :                     #

     |@                                               :                     #

     |@                                               :                     #

     |@                                               :                     #

     |@                                               :                     #

     |@                                               :                     #

     |@                                               :                     #

     |@                                               :                     #

     |@                                               :                     #

     |@                                               :                     #

     |@                                               :                     #

   0 +-------------------------------------------------------------------->Gi

     0                                                                  9.778

Number of snapshots: 80

 Detailed snapshots: [3, 12, 17, 22, 49, 59, 69, 79 (peak)]

```

输出的顶部是内存使用图。*x* 轴表示执行的指令数量，*y* 轴表示内存使用量。上面的图表表明，在我们运行 `matrixVector3` 时，总共执行了 97.78 亿 (Gi) 条指令。在执行过程中，`massif` 总共拍摄了 80 张快照来衡量堆内存的使用情况。内存使用在最后一张快照（79）时达到了峰值。程序的峰值内存使用为 763.3 MB，并且在整个程序执行过程中保持相对稳定。

所有快照的摘要会出现在图表之后。例如，以下表格对应的是第 79 张快照前后的数据：

```
····

------------------------------------------------------------------------------

  n        time(i)         total(B)   useful-heap(B) extra-heap(B)   stacks(B)

------------------------------------------------------------------------------

 70      1,081,926      727,225,400      727,080,000       145,400          0

 71      1,095,494      737,467,448      737,320,000       147,448          0

 72      1,109,062      747,709,496      747,560,000       149,496          0

 73      1,122,630      757,951,544      757,800,000       151,544          0

 74      1,136,198      768,193,592      768,040,000       153,592          0

 75      1,149,766      778,435,640      778,280,000       155,640          0

 76      1,163,334      788,677,688      788,520,000       157,688          0

 77      1,176,902      798,919,736      798,760,000       159,736          0

 78  7,198,260,935      800,361,056      800,201,024       160,032          0

 79 10,499,078,349      800,361,056      800,201,024       160,032          0

99.98% (800,201,024B) (heap allocations) malloc/new/new[], --alloc-fns, etc.

->99.96% (800,040,000B) 0x40089D: allocateArray (in matrixVector3)

```

每一行对应一个特定的快照，包括快照的时间点、此时的总堆内存消耗（以字节为单位）、程序请求的字节数（"useful-heap"）、程序请求字节数的超额分配部分，以及堆栈的大小。默认情况下，堆栈分析是关闭的（这会显著减慢 `massif` 的速度）。要启用堆栈分析，可以在运行 `massif` 时使用 `--stacks=yes` 选项。

`massif` 工具显示，99.96% 的程序堆内存使用发生在 `allocateArray` 函数中，总共分配了 8 亿字节内存，这与我们之前做的粗略计算一致。读者可能会发现 `massif` 是一个有用的工具，可以帮助识别程序中堆内存使用高的地方，而这些地方通常会导致程序变慢。例如，*内存泄漏* 在程序中可能会发生，当程序员频繁调用 `malloc` 而没有在首次正确的机会调用 `free` 时。`massif` 工具对于检测此类内存泄漏非常有用。

### 12.4 关键要点与总结

我们短暂的（也许是令人沮丧的）代码优化之旅应该向读者传达一个非常重要的信息：如果你打算手动优化代码，请仔细考虑什么值得花时间去做，什么应该留给编译器去处理。接下来是一些在提升代码性能时需要考虑的重要建议。

#### 选择良好的数据结构和算法

使用合适的算法和数据结构是无法替代的；不这样做通常是代码性能差的主要原因。例如，著名的埃拉托斯特尼筛法（Sieve of Eratosthenes）算法比我们在 `optExample` 中的自定义算法生成素数的效率要高得多，且显著提高了性能。以下列出了使用筛法实现生成 2 到 500 万之间所有素数所需的时间：

```
$ gcc -o genPrimes genPrimes.c

$ ./genPrimes 5000000

Found 348513 primes (0.122245 s)

```

筛选算法只需 0.12 秒即可找出 2 到 500 万之间的所有质数，而 `optExample2` 在开启 `-O3` 优化标志时，生成相同的质数集需要 1.46 秒（提高了 12 倍）。筛选算法的实现留作读者练习；然而，应该清楚的是，提前选择更好的算法可以节省数小时的繁琐优化工作。我们的示例展示了为什么了解数据结构和算法对计算机科学家来说至关重要。

#### 尽可能使用标准库函数

不要重复造轮子。如果在编程过程中，你需要一个应该做某件非常标准化的事情的函数（例如，求绝对值，或者求一组数字的最大值或最小值），停下来检查一下，看看该函数是否已经作为更高层语言的标准库的一部分存在。标准库中的函数经过充分测试，通常经过性能优化。例如，如果读者手动实现了自己的 `sqrt` 函数，编译器可能不知道自动用 `fsqrt` 指令替换该函数调用。

#### 基于数据进行优化，而非凭感觉

如果在选择了最佳数据结构和算法 *并且* 使用了标准库函数后，仍然需要进一步提高性能，请借助像 Valgrind 这样的优秀代码分析工具。优化*绝不*应该基于直觉。过于关注自己“感觉”应该优化的部分（而没有数据支持这一想法）通常会导致时间浪费。

#### 将复杂的代码拆分为多个函数

手动内联代码通常不会比现代编译器能实现的性能提升更大。相反，应该让编译器更容易地帮助你进行优化。编译器更容易优化较短的代码段。将复杂操作拆分为多个函数，不仅提高了代码的可读性，还让编译器更容易进行优化。检查一下你的编译器是否默认尝试内联，或者是否有独立的标志来尝试内联代码。让编译器来执行内联操作通常比手动内联更好。

#### 优先考虑代码可读性

在当今许多应用程序中，可读性至关重要。事实上，代码被阅读的次数远远超过被编写的次数。许多公司花费大量时间培训他们的软件工程师，以特定的方式编写代码，以最大化可读性。如果优化代码导致可读性显著降低，请检查所获得的性能改进是否值得。例如，许多编译器今天都有启用循环展开的优化标志。程序员应始终使用可用的循环展开优化标志，而不是尝试手动展开循环，这可能会显著降低代码的可读性。降低代码的可读性通常会增加无意中引入代码中的错误的可能性，这可能导致安全漏洞。

#### 注意内存使用

程序的内存使用对程序的执行时间影响更大，而不是它执行的指令数量。循环置换示例就是一个例子。在两种情况下，循环执行相同数量的指令。然而，循环的顺序对内存访问和局部性有显著影响。在尝试优化程序时，还要探索像`massif`和`cachegrind`这样的内存分析工具。

#### 编译器不断改进

编译器编写者持续更新编译器，以安全地执行更复杂的优化。例如，GCC 从 4.0 版本开始切换到静态单赋值（SSA）形式^(11)，显著改进了部分优化效果。GCC 代码库的`GRAPHITE`分支实现了多面体模型^(12)，使编译器能够执行更复杂的循环转换类型。随着编译器变得更加复杂，手动优化的好处显著减少。

### 注意事项

1. *[`gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html`](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html)*

2. John Regehr，“C 和 C++中未定义行为指南，第一部分”，*[`blog.regehr.org/archives/213`](https://blog.regehr.org/archives/213)*，2010 年。

3. C FAQ，“comp.lang.c FAQ list: Question 11.33”，*[`c-faq.com/ansi/undef.html`](http://c-faq.com/ansi/undef.html)*

4. John Regehr，“C 和 C++中未定义行为指南，第一部分”，*[`blog.regehr.org/archives/213`](https://blog.regehr.org/archives/213)*，2010 年。

5. 源代码可在*[`diveintosystems.org/book/C12-CodeOpt/_attachments/optExample.c`](https://diveintosystems.org/book/C12-CodeOpt/_attachments/optExample.c)*找到

6. *[`valgrind.org/`](http://valgrind.org/)*

7. *[`diveintosystems.org/book/C12-CodeOpt/_attachments/optExample2.c`](https://diveintosystems.org/book/C12-CodeOpt/_attachments/optExample2.c)*

8. *[`diveintosystems.org/book/C12-CodeOpt/_attachments/optExample3.c`](https://diveintosystems.org/book/C12-CodeOpt/_attachments/optExample3.c)*

9. *[`diveintosystems.org/book/C12-CodeOpt/_attachments/optExample3.c`](https://diveintosystems.org/book/C12-CodeOpt/_attachments/optExample3.c)*

10. *[`diveintosystems.org/book/C12-CodeOpt/_attachments/matrixVector.c`](https://diveintosystems.org/book/C12-CodeOpt/_attachments/matrixVector.c)*

11. *[`gcc.gnu.org/onlinedocs/gccint/SSA.html`](https://gcc.gnu.org/onlinedocs/gccint/SSA.html)*

12. *[`polyhedral.info/`](https://polyhedral.info/)*
