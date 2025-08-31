![](img/pg206.jpg)

描述



## 第十章：10 文件作用域变量声明与存储类说明符



![](img/opener-img.jpg)

我们将在第一部分结束时实现一些与函数和变量声明相关的重要特性。我们将支持文件作用域中的变量声明——即在源文件的最顶层——并引入关键字 static 和 extern。这些关键字是*存储类说明符*，用于控制声明的链接性以及声明对象的*存储持续时间*（即该对象在内存中存在的时间）。

本章的大部分内容将集中在语义分析阶段，确定每个声明的链接性和存储持续时间。我们还需要一些新的汇编指令来定义和初始化不同类型的变量，但编译器后端的变化相对简单。让我们首先回顾一下 C 标准对声明和存储类说明符的规定。即使你已经对 C 语言很熟悉，我也建议阅读以下内容。对编译器开发者来说，语言的这一部分与 C 程序员所理解的非常不同，主要是因为你的编译器需要支持一些理智的 C 程序员不会使用的行为。

### 关于声明的一切

每个源文件中的声明都有几个我们需要追踪的属性，我们将在本节中逐一分析。这些属性包括声明的作用域、链接性以及它是否同时是定义和声明。（它的类型也很重要，但在本章中我们不会再讨论这个。）我们还需要追踪程序中每个变量的存储持续时间。

确定这些属性的规则复杂且繁琐。它们取决于标识符是引用函数还是变量，是否在文件范围或*块范围*（函数体内）声明，以及应用了哪种存储类说明符。static说明符有两种不同的含义，适用于不同的上下文。extern说明符有多种看似不相关的效果，这些效果也取决于上下文。（其他存储类说明符——auto、register、_Thread_local和typedef——用于不同的目的，我这里就不展开讨论了。我们不会实现这些。）基本上，这部分 C 标准很混乱，但我们会尽力理清其中的内容。

C 语言中关于声明的术语可能不一致，因此在开始之前，我会先说明我使用的一些术语：

+   *文件*或*源文件*是经过预处理的源文件，在 C 标准中（以及前一章）被称为“翻译单元”。

+   *静态变量*是具有静态存储期的变量（在“存储期”部分讨论过，见第 212 页），不仅仅是使用static存储类说明符声明的变量。所有带有static说明符的变量都是静态变量，但并非所有静态变量都是使用该说明符声明的。

+   *自动变量*是具有自动存储期的变量（也在“存储期”中讨论），与静态存储期的变量相对。我们在之前章节中遇到的所有变量都是自动变量。

+   *外部变量*是具有内部或外部链接性的任何变量，而不仅仅是使用extern存储类说明符声明的变量。正如我们将看到的，所有外部变量也是静态变量，但并非所有静态变量都是外部变量。

#### 作用域

函数和变量遵循相同的作用域规则。变量可以在文件范围或块范围内声明，就像函数一样。文件范围的变量与函数和块范围变量一样，必须在使用之前声明，并且可能会被后来的块范围标识符覆盖。由于你已经了解了确定标识符作用域的规则，因此这里无需再多说。

#### 链接性

到目前为止，函数声明总是具有外部链接：每个特定函数名的声明都指向同一个函数定义。我们到目前为止看到的局部变量声明没有链接：同一个变量名的不同声明总是指向不同的对象。默认情况下，文件范围内的变量声明具有外部链接，就像函数声明一样。每当有多个文件范围的相同标识符声明时，编译器需要要么协调它们，使它们都指向同一个实体，要么抛出错误。

使用 static 说明符，我们还可以声明具有 *内部链接* 的函数和变量。内部链接的工作方式与外部链接相似，只不过具有内部链接的声明永远不会引用其他文件中的实体。为了说明这种区别，我们考虑一个由两个源文件组成的程序。列表 10-1 显示了第一个文件。

```
❶ int foo(void) {
    return 1;
}

❷ int bar(void) {
    return 2;
}
```

列表 10-1：定义两个具有外部链接的函数的源文件

列表 10-2 显示了第二个文件。

```
❸ int foo(void);
❹ static int bar(void);

int main(void) {
    return foo() + bar();
}

❺ static int bar(void) {
    return 4;
}
```

列表 10-2：声明一个具有内部链接（bar）和两个具有外部链接（foo 和 main）的源文件

在列表 10-1 中，我们定义了两个具有外部链接的函数：foo ❶ 和 bar ❷。列表 10-2 还包括了标识符 foo ❸ 和 bar ❹❺ 的声明。首先，让我们弄清楚在列表 10-2 中，foo意味着什么。因为❸处的声明没有包含 static 说明符，它具有外部链接。因此，声明 ❶ 和 ❸ 指向同一个函数，该函数在 ❶ 处定义。

接下来，让我们考虑 bar。由于在❹中的声明包括了 static 说明符，它具有内部链接性。这意味着它并不引用❷处的定义，而是声明了一个全新的函数。该函数的定义出现在后面的❺处。由于❹和❺的声明都具有内部链接性并且出现在同一个文件中，它们指向同一个函数。因此，main 将使用❺处 bar 的定义来计算 1 + 4，并返回 5。

请注意，具有内部链接性的标识符不会覆盖具有外部链接性的标识符，反之亦然。在清单 10-2 中声明的 bar 并没有覆盖❷处在清单 10-1 中的定义；实际上，该定义在清单 10-2 中本来就不可见，因为清单 10-2 中的声明没有引用它。如果一个标识符在同一个文件中同时声明为内部链接性和外部链接性，其行为是未定义的，大多数编译器会抛出错误。

C 标准第 6.2.2 节列出了确定标识符链接性的规则，我将在这里总结一下。声明的链接性取决于两个因素：它包含的存储类说明符（如果有的话），以及它是在块作用域还是文件作用域中声明的。没有存储类说明符的函数声明总是被当作包含 extern 说明符来处理，我们稍后会讨论。如果没有存储类说明符的变量声明出现在块作用域中，它们没有链接性。如果它们出现在文件作用域中，则具有外部链接性。

在文件作用域中，static 说明符表示函数或变量具有内部链接。在块作用域中，static 说明符控制存储持续时间，而不是链接性。使用此说明符在块作用域中声明的变量没有链接性，和没有说明符声明的变量一样。块作用域中声明static 函数是非法的，因为函数没有存储持续时间。

extern 修饰符更加复杂。如果一个标识符在某个地方声明为 extern，且该标识符的先前声明可见，且先前声明具有内部或外部链接性，那么新的声明将与先前的声明具有相同的链接性。如果没有可见的先前声明，或者先前的声明没有链接性，那么 extern 声明将具有外部链接性。

在示例 10-3 中，我们使用 extern 来声明一个已经可见的标识符。

```
static int a;
extern int a;
```

示例 10-3：当先前的声明可见时使用 extern 声明一个标识符

a 的第一次声明由于使用了 static 关键字，所以具有内部链接性。由于第二次声明是在第一次声明可见的地方使用 extern 关键字，因此它也会具有内部链接性。

而在示例 10-4 中，我们在没有任何先前声明可见的地方使用 extern。

```
int main(void) {
    extern int a;
    return a;
}

int a = 5;
```

示例 10-4：使用 extern 在块作用域内声明一个具有外部链接性的变量

在 main 中的 a 声明和文件后面的 a 定义都具有外部链接性，因此它们指向同一个对象。因此，main 会返回 5。

如果一个具有外部链接性的变量被局部变量遮蔽，你可以使用 extern 将其重新引入作用域。示例 10-5 展示了这个过程是如何工作的。

```
int a = 4;
int main(void) {
    int a = 3;
    {
      ❶ extern int a;
        return a;
    }
}
```

示例 10-5：使用 extern 将具有外部链接的变量重新引入作用域

当我们在 main ❶ 中用 extern 说明符声明 a 时，之前没有任何带有内部或外部链接性的声明可见。（a 的初始文件作用域声明具有外部链接性，但被第二个在块作用域中的声明隐藏了。块作用域声明是可见的，但没有链接性。）因此，这个 extern 声明具有外部链接性。由于之前文件作用域中的 a 也具有外部链接性，因此两个声明引用的是同一个变量。接下来，我们在下一行的 return 语句中使用了这个变量。结果，main 返回 4。

之前，我提到过没有存储类说明符的函数声明总是被视为包含了 extern 说明符。考虑一下这个规则如何影响示例 10-6 中函数定义的链接性。

```
static int my_fun(void);
int my_fun(void) {
    return 0;
}
```

示例 10-6：带有 static 说明符的函数声明，后跟一个没有存储类说明符的函数定义

正如我们在示例 10-3 中看到的，带有 extern 说明符的声明与该标识符的前一个声明具有相同的链接性，如果该声明可见的话。由于我们将 my_fun 的定义视为带有 extern 说明符，它将与前一行的声明具有相同的链接性；也就是说，具有内部链接性。这个规则意味着，在函数声明中包括 extern 始终是多余的（除非是内联函数，我们不会实现）。

接下来，我们将考虑本章的新概念：存储持续时间。

#### 存储持续时间

存储持续时间是变量的一个属性；函数没有存储持续时间。C 标准第 6.2.4 节，第 1 至第 2 段提供了如下描述：“一个对象具有*存储持续时间*，决定了它的生命周期……一个对象的*生命周期*是程序执行过程中保证为其保留存储空间的那部分时间。在它的生命周期内，一个对象存在，具有恒定的地址，并保持其最后存储的值。”换句话说，在一个对象的生命周期内，你可以像平常一样使用它：你可以写入它、读取它，并得到你最后写入的值。在此期间，对象不会被释放或重新初始化。

在本章中，我们将讨论两种类型的存储持续时间：自动和静态。我们在前几章看到的所有变量都有*自动存储持续时间*。具有自动存储持续时间的变量的生命周期从进入声明它的代码块时开始，到退出该代码块时结束。这意味着你不能使用自动变量来跟踪一个函数被调用的次数。例如，要理解为什么，看看清单 10-7，它正是尝试这么做的。

```
#include <stdio.h>

int recursive_call(int count_was_initialized) {
    int count;
    if(!count_was_initialized) {
        count = 0;
        count_was_initialized = 1;
    }
    count = count + 1;
    printf("This function has been called %d times\n", count);
    if (count < 20) {
        recursive_call(count_was_initialized);
    }
    return 0;
}
```

清单 10-7：错误的尝试在多个函数调用之间共享自动变量的值

清单 10-7 中的 recursive_call 函数试图在第一次调用时初始化一个局部变量 count，然后在每次后续调用时递增它。这是行不通的，因为 count 具有自动存储持续时间；每次调用 recursive_call 都会分配一个新的、未初始化的 count 副本，且当该调用返回时，这个副本会被释放。

另一方面，如果一个变量具有*静态存储持续时间*，那么它的生命周期将持续整个程序的执行时间。具有静态存储持续时间的变量会在程序开始之前初始化一次，并在程序退出时结束其生命周期。

确定存储持续时间的规则很简单：所有在文件作用域声明的变量都具有静态存储持续时间，所有在块作用域中声明并带有 static 或 extern 关键字的变量也具有静态存储持续时间。所有在块作用域中声明且没有存储类说明符的变量具有自动存储持续时间。标准还定义了*分配存储持续时间*，我们将在第二部分中讨论它，另外还有*线程存储持续时间*，但本书中我们不会实现它。

我们可以使用静态计数器来修复清单 10-7。清单 10-8 显示了正确实现的recursive_call。

```
#include <stdio.h>

int recursive_call(void) {
  ❶ static int count = 0;
    count = count + 1;
    printf("This function has been called %d times\n", count);
    if (count < 20) {
        recursive_call();
    }
    return 0;
}
```

清单 10-8：在多个函数调用之间正确共享静态变量的值

现在，由于count是使用static关键字声明的❶，它具有静态存储持续时间。我们将在程序开始前，只分配一次count并将其初始化为0。然后，在每次调用recursive_call时，我们将递增这个相同的count变量。

当我们进入recursive_call中的声明时，我们不会再次初始化count。声明标志着变量引入作用域的地方，而不是它在执行时初始化的地方。重要的是要理解，静态变量的作用域和生命周期是无关的。在第七章中，我将变量的作用域描述为程序中可以使用它的部分。现在我们需要细化这个定义，明确指出它是程序的*源代码*中可以使用变量的部分。而变量的生命周期是*程序执行*期间，变量有地址和值的部分。对于自动变量，作用域和生命周期是如此紧密相连，以至于这种区别几乎无关紧要：变量的生命周期从你开始执行它所在作用域的代码块时开始，执行完该代码块时结束。但静态变量的生命周期独立于它的作用域。例如，在清单 10-8 中，count的生命周期持续到程序的整个执行过程，但它的作用域仅从在recursive_call中声明的地方开始，直到函数结束。

由于静态变量在启动前就会初始化，因此它们的初始化器必须是常量。清单 10-9 显示了两个文件作用域声明，其中一个具有无效的初始化器。

```
int first_var = 3;
int second_var = first_var + 1;
```

清单 10-9：文件作用域变量声明，带有效和无效初始化器

first_var和second_var都有静态存储持续时间，因为它们是在文件作用域内声明的。first_var的初始化器是有效的，因为它是常量。然而，second_var的初始化器是无效的，因为你无法在程序开始之前计算像first_var + 1这样的表达式。

> 注意

*C 标准允许用常量表达式初始化静态变量，比如 1 + 1，因为这些可以在编译时计算出来。为了让我们的工作稍微轻松一点，我们的编译器将只支持常量值作为初始化器，而不支持常量表达式。*

#### 定义与声明

在上一章中，我们需要区分函数定义和函数声明。在本章中，我们将把这种区分扩展到变量上。如果一个变量被定义，我们的汇编程序需要为其分配存储空间，并可能对其进行初始化。如果它被声明但没有定义，我们则不会为它分配存储空间；我们会依赖链接器在另一个目标文件中找到其定义。就像函数一样，变量可以声明多次，但只能定义一次。

识别函数定义很容易，因为它们有函数体。弄清楚什么算作变量定义则稍微复杂一些。让我们通过规则来了解哪些变量声明也是定义，哪些不是。我们还将讨论如何（以及何时）初始化没有显式初始化器的定义变量。

首先，每个带有初始化器的变量声明都是一个定义。这并不令人惊讶，因为如果没有为变量分配存储空间，就无法初始化它。其次，每个没有链接的变量声明也是一个定义。没有链接且不是定义的变量声明是完全没有意义的：没有链接的变量不能声明多次，因此你无法在程序的其他地方定义该变量。

我们如何初始化一个没有链接的变量取决于它的存储持续时间。回想一下，在前几章中，局部变量是在栈上分配空间的，但不一定会被初始化。局部静态变量，如我们稍后将看到的，它们在不同的内存段上分配空间，并且总是会被初始化。如果没有提供显式的初始化器，它们会被初始化为零。

如果一个变量声明带有extern说明符且没有初始化器，它就不是一个定义。请注意，块作用域中的extern变量声明不能有初始化器。因此，它们永远不是定义。（这类似于你可以在块作用域中声明函数，但不能定义它们。）我们可以使用extern说明符来声明在同一文件的其他地方定义的变量，如清单 10-10 所示。

```
extern int three;

int main(void) {
    return three;
}

int three = 3;
```

清单 10-10：在文件开头声明一个外部变量，并在文件末尾定义它

清单开头的声明使得three进入作用域，而清单末尾的定义确定了它的初始值，3。extern说明符也允许我们声明在其他文件中定义的变量，如清单 10-11 所示。

```
extern int external_var;

int main(void) {
    return 1 + external_var;
}
```

清单 10-11：声明一个变量但没有定义它

由于external_var在本文件中没有定义，编译器不会为其分配或初始化内存。链接器会在另一个文件中找到它的定义，或者抛出错误。

一个具有内部或外部链接的变量声明，没有extern说明符，也没有初始化器，就是一个*初步定义*。清单 10-12 展示了一个例子。

```
int x;

int main(void) {
    return x;
}
```

清单 10-12：一个初步定义

本文件中唯一的x的定义是第一行的初步定义。如果一个变量是初步定义的，我们会将其初始化为零。因此，清单 10-12 的第一行的处理方式就像下面这个非初步定义一样：

```
int x = 0;
```

如果一个文件同时包含同一个变量的初步定义和明确初始化的定义，如清单 10-13 所示，明确的定义优先。

```
int x;

int main(void) {
    return x;
}

int x = 3;
```

清单 10-13：一个初步定义后跟一个明确的定义

该清单以<sup class="SANS_TheSansMonoCd_W5Regular_11">x</sup>的临时定义开始，以非临时定义结束。非临时定义具有优先权，因此<sup class="SANS_TheSansMonoCd_W5Regular_11">x</sup>被初始化为<sup class="SANS_TheSansMonoCd_W5Regular_11">3</sup>。第一行被当作声明处理，就像它包含了<sup class="SANS_TheSansMonoCd_W5Regular_11">extern</sup>说明符一样。

虽然定义一个变量不可以多次进行，但拥有多个临时定义的变量是完全合法的。考虑清单 10-14 中的文件作用域声明。

```
int a;
int a;
extern int a;
int a;
```

清单 10-14：三个临时定义和一个声明

在这里，我们有三个<sup class="SANS_TheSansMonoCd_W5Regular_11">a</sup>的临时定义和一个<sup class="SANS_TheSansMonoCd_W5Regular_11">a</sup>的声明，由于它的<sup class="SANS_TheSansMonoCd_W5Regular_11">extern</sup>说明符，它不是定义。因为没有非临时的<sup class="SANS_TheSansMonoCd_W5Regular_11">a</sup>的定义，所以它将被初始化为零。因此，清单 10-14 将会被编译为包含以下行的形式：

```
int a = 0;
```

表 10-1 和表 10-2 总结了标识符的链接性、存储持续时间和定义状态是如何确定的。最左侧的列，作用域和说明符，指的是声明的语法；我们将在解析后知道声明的作用域和存储类说明符。其余的列是属性，我们将在语义分析阶段基于声明的语法来确定。

表 10-1 涵盖了变量声明。

表 10-1： 变量声明的属性

| 作用域 | 说明符 | 链接性 | 存储持续时间 | 是否定义？ |
| --- | --- | --- | --- | --- |
| 有初始化器 | 无初始化器 |
| --- | --- |
| 文件作用域 | 无 | 外部 | 静态 | 是 | 临时 |
|  | static | 内部的 | 静态的 | 是 | 暂定的 |
|  | extern | 与之前可见声明匹配；默认是外部的 | 静态的 | 是 | 否 |
| 块作用域 | 无 | 无 | 自动 | 是 | 是（已定义但未初始化） |
|  | static | 无 | 静态的 | 是 | 是（初始化为零） |
|  | extern | 与之前可见声明匹配；默认是外部的 | 静态的 | 无效 | 否 |

表 10-2 介绍了函数声明。

表 10-2: 函数声明的属性

| 作用域 | 说明符 | 连接性 | 定义？ |
| --- | --- | --- | --- |
| 有函数体 | 无函数体 |
| --- | --- |
| 文件作用域 | 无或 extern | 与之前可见声明匹配；默认是外部的 | 是 | 否 |
|  | static | 内部的 | 是 | 否 |
| 块作用域 | 无或 extern | 与先前可见声明匹配；默认外部链接性 | 无效 | 无 |
|  | static | 无效 | 无效 | 无效 |

请注意，函数定义中的参数具有自动存储持续时间且没有链接性，类似于没有存储类说明符的块作用域变量。

到此为止，你已经理解了声明的最重要特性。你知道如何确定声明的链接性、存储持续时间，以及它是否定义了一个实体并声明了它。你还明白了这些特性如何影响你可以对标识符进行的操作。接下来，让我们讨论可能出现的问题。

#### 错误案例

我们将在本章中需要检测一大堆错误案例。其中一些错误案例会让你觉得熟悉，因为它们来自早期的章节，尽管细节会有所变化，以适应我们新的语言结构。我们还将处理一些全新的错误案例。

##### 冲突声明

声明可能发生冲突的方式有很多。我们的编译器已经能检测到其中的一些。例如，当同一个标识符的两个声明出现在同一局部作用域内，并且至少其中一个声明没有链接性时，编译器会检测到错误。这是一个错误，因为你无法将后续对该标识符的使用解析为单一实体。

如我之前提到的，将相同的标识符同时声明为具有内部和外部链接性也是一个错误。即使这两个声明位于源文件的完全不同部分，这也是一个问题。例如，示例 10-15 中就包含了冲突声明。

```
int main(void) {
    extern int foo;
    return foo;
}

static int foo = 3;
```

示例 10-15：具有冲突链接性的变量声明

在foo在main中声明时，其他声明是不可见的。（变量何时变得可见取决于它在程序源代码中的声明位置，而不是它在程序执行期间何时初始化。）根据我们之前讨论的规则，这意味着foo具有外部链接性。然而，在后面的代码中，foo在文件作用域中被声明为具有内部链接性。你不能同时定义一个具有内部和外部链接性的相同对象，所以这是不合法的。

最后，如果两个相同实体的声明具有不同类型，则它们会发生冲突。声明一个外部变量和一个具有相同名称的函数是非法的。同样，即使在像 清单 10-16 这样的程序中，声明发生在程序的完全不同部分，冲突声明仍然会导致非法。

```
int foo = 3;

int main(void) {
    int foo(void);
    return foo();
}
```

清单 10-16：具有冲突类型的声明

由于 foo 的两个声明都有外部链接性，它们应该引用同一个实体，但这不可能，因为一个是函数声明，另一个是变量声明。因此，这个程序是无效的。

##### 多重定义

我们已经看到，在同一个程序中多次定义同一个函数是非法的。外部变量的多重定义也是非法的。如果在同一个文件中定义了多个外部变量，编译器应当报错。如果函数或变量在多个文件中定义，编译器无法捕获该错误，但链接器会发现。

##### 无定义

这种错误适用于函数和变量。如果你使用一个已声明但未定义的标识符，在链接时会发生错误，当链接器试图查找定义时会失败。由于这是一个链接时错误，编译器不需要检测它。

##### 无效的初始化器

正如我们已经看到的，静态变量的初始化器必须是常量。在块作用域内，extern 声明不能有任何初始化器，甚至不能有常量初始化器。

##### 存储类说明符的限制

你不能将 extern 或 static 说明符应用于函数参数或在 for 循环头部声明的变量。你也不能将 static 应用于块作用域内的函数声明。（你可以将 extern 应用于它们，但它没有任何作用。）

### 汇编中的链接性和存储持续时间

当我们扩展编译器的各个阶段时——特别是语义分析阶段——理解我们在前一部分讨论的概念如何转化为汇编代码将非常有帮助。我将首先讨论链接性，然后是存储持续时间。链接性比较简单：如果标识符具有外部链接性，我们将为对应的汇编标签发出一个.globl指令。如果标识符没有外部链接性，我们则不会发出.globl指令。.globl指令适用于函数和变量名。

现在我们来谈谈存储持续时间。我们在前面章节中处理的那些具有自动存储持续时间的变量，都是存在栈上的。静态变量则存在内存的另一个区域——*数据段*。（有些静态变量存在紧密相关的 BSS 段，我稍后会讨论。）与栈一样，数据段是程序可以读写的内存区域。

然而，尽管栈被划分为由成熟调用约定管理的帧，数据段却是一个大的内存块，无论你在哪个函数中，它始终存在。这使得数据段成为存储具有静态存储持续时间变量的理想场所：数据段中的对象在我们调用和返回函数时不会被释放或覆盖。我们没有像 RSP 或 RBP 那样指向数据段特定位置的专用寄存器，也不需要它们；正如你将看到的，我们可以通过名称来引用这个区域中的变量。

默认情况下，汇编器将写入文本段，这是存放机器指令的内存区域。.data指令告诉汇编器改为开始写入数据段。示例 10-17 展示了如何在数据段初始化一个变量。

```
 .data
    .align 4
var:
    .long 3
```

示例 10-17：在数据段初始化变量

清单 10-17 的第一行表示我们正在写入数据区段。下一行的 .align 指令确定我们将写入的下一个值的对齐方式；4 字节对齐意味着该值的地址必须是 4 的倍数。.align 指令的含义因平台而异。在 Linux 上，.align n 会产生 *n* 字节对齐。在 macOS 上，.align n 会产生 2^n 字节对齐。这意味着在 Linux 上 .align 4 会使下一个值进行 4 字节对齐，而在 macOS 上则会进行 16 字节对齐。

第三行是一个标签；你可以像标记文本区段中的位置一样标记数据区段中的位置。最后一行将 32 位整数 3 写入当前区段；这是数据区段，因为之前的 .data 指令。由于在 x64 汇编中 *long* 表示 32 位，.long 指令始终写入一个 32 位整数。（回忆一下，像 movl 这样的 32 位操作数指令中的 l 后缀代表 *long*。）

和其他标签一样，var 标签默认是内部标签，仅在该目标文件中可见。我们可以包含 .globl 指令，使其在其他目标文件中也可见：

```
 .globl var
```

我之前提到过，一些静态变量存储在 *BSS 区段* 中。（由于一些晦涩的历史原因，BSS 代表 *Block Started by Symbol*。）这个区段的工作方式几乎与数据区段相同，唯一不同的是它仅包含初始化为零的变量。这是节省磁盘空间的一种技巧；可执行文件或目标文件只需要记录 BSS 区段的大小，而不需要记录其内容，因为它的内容全都是零。

清单 10-18 在 BSS 区段初始化一个变量。

```
 .bss
    .align 4
var:
    .zero 4
```

清单 10-18：在 BSS 区段初始化变量

这段代码与清单 10-17 有两个不同之处。首先，我们使用 .bss 指令向 BSS 段写入数据，而不是数据段。其次，我们使用 .zero n 指令写入 *n* 字节的零。例如，.zero 4 将一个 4 字节的整数初始化为零。无论是在处理数据段还是 BSS 段时，我们都会使用 .align 指令，声明标签，并根据需要包含或省略 .globl 指令。

如果在你编译的文件中声明了一个变量，但没有定义它，你将不会向数据段或 BSS 段写入任何内容。

最后，让我们看看如何在汇编指令中引用数据段中的标签。这一行代码将立即数值 4 写入标签为 var 的内存地址：

```
movl    $4, var(%rip)
```

操作数像 var(%rip) 使用了 *RIP 相对寻址*，它是指相对于指令指针的内存地址。显然，我们不能像引用栈变量那样，使用 RBP 和 RSP 来引用数据段中的符号。我们也不能在链接时将它们替换为绝对地址，因为我们正在编译位置无关代码，该代码可以加载到程序内存中的任何位置。相反，我们使用 RIP 寄存器，它保存程序文本段中当前指令的地址，用来计算类似 var 这样的变量在程序数据段中的地址。

RIP 相对寻址的细节较为复杂，因此我在这里不再详细讲解。相反，我再次推荐 Eli Bendersky 关于位置无关代码的优秀博客文章，相关链接我已经在第一章的“附加资源”中提供了，在第 21 页也有详细说明。

现在你已经理解了存储持续时间、链接和变量初始化在 C 语言和汇编中的工作原理，接下来你可以开始扩展你的编译器了。

### 词法分析器

你将在这一章中添加两个新的关键字：

static

extern

### 语法分析器

在这一章，我们将对抽象语法树（AST）进行两项修改：我们将添加变量声明作为顶层构造，并且为函数和变量声明添加可选的存储类说明符。清单 10-19 展示了更新后的 AST 定义。

```
program = Program(**declaration***)
declaration = FunDecl(function_declaration) | VarDecl(variable_declaration)
variable_declaration = (identifier name, exp? init, **storage_class?**)
function_declaration = (identifier name, identifier* params,
                        block? body, **storage_class?**)
**storage_class = Static | Extern**
block_item = S(statement) | D(declaration)
block = Block(block_item*)
for_init = InitDecl(variable_declaration) | InitExp(exp?)
statement = Return(exp)
          | Expression(exp)
          | If(exp condition, statement then, statement? else)
          | Compound(block)
          | Break
          | Continue
          | While(exp condition, statement body)
          | DoWhile(statement body, exp condition)
          | For(for_init init, exp? condition, exp? post, statement body)
          | Null
exp = Constant(int)
    | Var(identifier)
    | Unary(unary_operator, exp)
    | Binary(binary_operator, exp, exp)
    | Assignment(exp, exp)
    | Conditional(exp condition, exp, exp)
    | FunctionCall(identifier, exp* args)
unary_operator = Complement | Negate | Not
binary_operator = Add | Subtract | Multiply | Divide | Remainder | And | Or
                | Equal | NotEqual | LessThan | LessOrEqual
                | GreaterThan | GreaterOrEqual
```

列表 10-19：带有文件作用域变量和存储类说明符的抽象语法树

我们已经定义了一个 declaration AST 节点，其中包括函数和变量声明。现在我们支持文件作用域变量声明，因此我们将在顶层使用 declaration 节点。

列表 10-20 显示了语法的相应变化。

```
<program> ::= {**<declaration>**}
<declaration> ::= <variable-declaration> | <function-declaration>
<variable-declaration> ::= **{<specifier>}+** <identifier> ["=" <exp>] ";"
<function-declaration> ::= **{<specifier>}+** <identifier> "(" <param-list> ")" (<block> | ";")
<param-list> ::= "void" | "int" <identifier> {"," "int" <identifier>}
**<specifier> ::= "int" | "static" | "extern"**
<block> ::= "{" {<block-item>} "}"
<block-item> ::= <statement> | <declaration>
<for-init> ::= <variable-declaration> | [<exp>] ";"
<statement> ::= "return" <exp> ";"
              | <exp> ";"
              | "if" "(" <exp> ")" <statement> ["else" <statement>]
              | <block>
              | "break" ";"
              | "continue" ";"
              | "while" "(" <exp> ")" <statement>
              | "do" <statement> "while" "(" <exp> ")" ";"
              | "for" "(" <for-init> [<exp>] ";" [<exp>] ")" <statement>
              | ";"
<exp> ::= <factor> | <exp> <binop> <exp> | <exp> "?" <exp> ":" <exp>
<factor> ::= <int> | <identifier> | <unop> <factor> | "(" <exp> ")"
           | <identifier> "(" [<argument-list>] ")"
<argument-list> ::= <exp> {"," <exp>}
<unop> ::= "-" | "~" | "!"
<binop> ::= "-" | "+" | "*" | "/" | "%" | "&&" | "||"
          | "==" | "!=" | "<" | "<=" | ">" | ">=" | "="
<identifier> ::= ? An identifier token ?
<int> ::= ? A constant token ?
```

列表 10-20：带有文件作用域变量和存储类说明符的语法

我们将 <program> 定义为一个 <declaration> 符号的列表，就像我们在 列表 10-19 中所做的那样。我们还引入了一个新的 <specifier> 符号，它表示类型和存储类说明符，并且我们要求每个声明都以说明符列表开始。我们在这里添加了一个新的 EBNF 记法：将某个内容用大括号括起来并跟随一个 + 符号表示该内容必须至少重复一次。因此，{<specifier>}+ 表示一个非空的说明符列表。注意，<param-list> 规则没有变化；我们仍然期望每个参数使用单个 int 关键字声明，而不是一个说明符列表。如果解析器遇到一个 static 或 extern 参数，它应该抛出一个错误。

#### 解析类型和存储类说明符

我们将类型和存储类说明符合并为一个符号，因为它们可以在声明中以任何顺序出现。换句话说，声明

```
static int a = 3;
```

等同于：

```
int static a = 3;
```

当我们在 第二部分 中添加更多类型说明符时，事情会变得更加复杂。一个声明可能包含多个类型说明符（如 long 和 unsigned），这些说明符可以相对于存储类说明符和彼此以任何顺序出现。

为了构建 AST，解析器需要在声明开始时处理说明符列表，然后将其转换为一个类型和至多一个存储类说明符。列表 10-21 中的伪代码概述了如何处理说明符列表。

```
parse_type_and_storage_class(specifier_list):
    types = []
    storage_classes = []
  ❶ for specifier in specifier_list:
        if specifier is "int":
            types.append(specifier)
        else:
            storage_classes.append(specifier)

    if length(types) != 1:
        fail("Invalid type specifier")
    if length(storage_classes) > 1:
        fail("Invalid storage class")

  ❷ type = Int

    if length(storage_classes) == 1:
      ❸ storage_class = parse_storage_class(storage_classes[0])
    else:
        storage_class = null

    return (type, storage_class)
```

列表 10-21：确定声明的类型和存储类

我们首先将列表划分为类型说明符和存储类说明符❶。然后，我们验证每个列表。类型说明符列表必须只有一个值。存储类说明符列表可以为空，也可以包含一个值。最后，我们返回结果。此时，Int是唯一的可能类型❷。如果存储类说明符列表不为空，我们将把它的唯一元素转换为对应的 storage_class AST 节点❸。 （我省略了 parse_storage_class 的伪代码，因为它没有太多内容。）如果存储类说明符列表为空，则声明没有存储类。

列表 10-21 比我们目前需要的要复杂一些，但随着我们在后续章节中添加更多类型说明符，它将很容易扩展。

#### 区分函数声明和变量声明

我们唯一剩下的挑战是，无法区分 <function -declaration> 和 <variable-declaration> 符号，除非解析整个类型和存储类说明符列表。随着我们在后续章节中支持更复杂的声明，这两个符号将具有更多共同的解析逻辑。这意味着，编写单独的函数来解析这两个语法符号并不实际；相反，你应该编写一个函数来解析两者，并返回一个 declaration AST 节点。你可以只存在一种声明而没有另一种的唯一地方是 for 循环的初始子句。为了处理这种情况，只需解析整个声明，然后如果它是函数声明，则失败。

现在，你拥有了扩展解析器所需的所有内容。

### 语义分析

接下来，我们需要扩展标识符解析和类型检查的过程。在标识符解析过程中，我们将处理顶层变量声明，并检查同一作用域内是否有重复声明。在类型检查过程中，我们将向符号表中添加存储类和链接信息，因为生成汇编代码时需要这些信息。我们还将在类型检查器中处理剩余的错误情况。

#### 标识符解析：解析外部变量

与函数一样，外部变量在标识符解析过程中不会被重命名。我们的标识符映射会跟踪每个标识符是否具有链接（内部或外部链接）。在类型检查阶段之前，我们无需区分内部链接和外部链接。

我们需要分别处理块作用域和文件作用域的变量声明，因为这两种作用域中确定链接性的方法规则不同。列表 10-22 演示了如何解析文件作用域的变量声明。

```
resolve_file_scope_variable_declaration(decl, identifier_map):
    identifier_map.add(decl.name, MapEntry(new_name=decl.name,
                                           from_current_scope=True,
                                           has_linkage=True))
    return decl
```

列表 10-22：解析文件作用域的变量声明

如你即将看到的，这比处理块作用域变量声明的代码要简单得多。我们不需要生成唯一名称，因为外部变量在此阶段会保持其原始名称。我们不需要担心该变量的前期声明；任何先前的声明也必须具有内部或外部链接，因此它们会引用相同的对象并在标识符映射中有相同的条目。（文件作用域声明可能会以其他方式发生冲突，但我们将在类型检查器中处理这些冲突。）无论声明是否为 static，我们都可以统一处理这些声明。由于我们不需要区分内部链接和外部链接，我们将继续使用上一章中的布尔值 has_linkage 属性。对于文件作用域的标识符，该属性始终为 True。我们也不需要递归处理初始化器，因为它应该是常量，因此不应该包含需要重命名的变量。如果初始化器不是常量，我们将在类型检查过程中捕获这一点。

现在让我们考虑块作用域中的变量。如果一个变量是用 `extern` 关键字声明的，我们会在标识符映射中记录它有链接，并保留其原始名称。否则，我们像处理本地变量一样处理它。如果一个标识符在同一作用域中同时声明了有链接和没有链接的情况，我们就无法保持一致的标识符映射，因此会抛出错误。列表 10-23 显示了如何用伪代码实现这一点。

```
resolve_local_variable_declaration(decl, identifier_map):
    if decl.name is in identifier_map:
        prev_entry = identifier_map.get(decl.name)
      ❶ if prev_entry.from_current_scope:
            if not (prev_entry.has_linkage and decl.storage_class == Extern):
                fail("Conflicting local declarations")

    if decl.storage_class == Extern:
      ❷ identifier_map.add(decl.name, MapEntry(new_name=decl.name,
                                               from_current_scope=True,
                                               has_linkage=True))
        return decl
    else:
        unique_name = make_temporary()
      ❸ identifier_map.add(decl.name, MapEntry(new_name=unique_name,
                                               from_current_scope=True,
                                               has_linkage=False))
        `--snip--`
```

列表 10-23：解析块作用域的变量声明

首先，我们检查是否有冲突的声明❶。如果该标识符已经在当前作用域中声明，我们检查之前声明的链接性。如果它有链接性且当前声明也有链接性（由extern关键字表示），则它们都指向相同的对象。在这种情况下，声明是一致的，至少在标识符解析的目的下是如此。如果任一标识符或两个标识符都没有链接性，则它们指向两个不同的对象，因此我们会抛出错误。

假设没有冲突，我们更新标识符映射。如果该声明有链接性，它保持当前的名称❷；否则，我们重命名它❸。请注意，无链接的变量在此处理时无论是否为static都一样。还要注意，我们不需要递归处理无链接变量的初始化器，因为它们根本不应该有初始化器。（我已省略了解决无链接变量初始化器的代码，因为它与前面章节相同。）

你不需要更改这个阶段处理函数声明的方式，唯一的小例外是：如果一个块作用域函数声明包含static修饰符，应该抛出一个错误。你可以在标识符解析阶段进行此操作，正好是在验证块作用域函数声明没有函数体的地方。然而，在类型检查器中抛出此错误，甚至在解析器中抛出，也同样有效。

#### 类型检查：跟踪静态函数和变量

接下来，我们将更新符号表并处理剩余的错误情况。我们将向符号表中添加几项新信息。首先，我们将记录每个变量的存储持续时间。其次，我们将记录具有静态存储持续时间的变量的初始值。最后，我们将记录具有静态存储持续时间的函数和变量是否是全局可见的。这些信息都将影响我们稍后生成的汇编代码。

我们正在添加到类型检查器中的大部分逻辑本身并不是类型检查，因为标识符的存储类和链接性与其类型是分开的。但类型检查器是一个自然的位置来处理这些逻辑，因为我们将在符号表中一起跟踪每个标识符的类型、链接性和存储类。

##### 符号表中的标识符属性

我们需要在符号表中跟踪每种标识符的不同信息：函数、具有静态存储持续时间的变量，以及具有自动存储持续时间的变量。清单 10-24 展示了一种表示这些信息的方式。

```
identifier_attrs = FunAttr(bool defined, bool global)
                 | StaticAttr(initial_value init, bool global)
                 | LocalAttr

initial_value = Tentative | Initial(int) | NoInitializer
```

列表 10-24：不同类型标识符的符号表属性

StaticAttr表示我们需要跟踪的具有静态存储持续时间的变量的属性。initial_value类型让我们能够区分带有初始化器的变量定义、没有初始化器的暂定定义，以及extern变量声明。FunAttr表示函数，LocalAttr表示具有自动存储持续时间的函数参数和变量。每个符号表条目应包括类型（如前一章所定义）和identifier_attrs。

现在我们可以在符号表中表示所需的信息，让我们来看一下我们需要进行类型检查的三种声明：函数声明、文件作用域变量声明和块作用域变量声明。

##### 函数声明

这里的大部分逻辑将保持不变。我们将检查当前声明是否与之前的声明类型相同，并且确保函数没有被多次定义。唯一的不同是，我们还会记录函数是否是全局可见的。列表 10-25 中的伪代码展示了我们如何进行函数声明的类型检查，和列表 9-21 相比，新增的更改已加粗，部分未更改的代码已省略。（我还对代码做了一些调整，以适应我们符号表表示法的变化，尽管逻辑本质上保持不变。这些更改没有加粗。）

```
typecheck_function_declaration(decl, symbols):
    fun_type = FunType(length(decl.params))
    has_body = decl.body is not null
    already_defined = False
  ❶ **global = decl.storage_class != Static**

  ❷ if decl.name is in symbols:
        old_decl = symbols.get(decl.name)
        if old_decl.type != fun_type:
            fail("Incompatible function declarations")
        already_defined = old_decl.attrs.defined
        if already_defined and has_body:
            fail("Function is defined more than once")

 **if old_decl.attrs.global and decl.storage_class == Static:**
 **fail("Static function declaration follows non-static")**
❸ **global = old_decl.attrs.global**

    **attrs = FunAttr(defined=(already_defined or has_body), global=global)**
    symbols.add(decl.name, fun_type, **attrs=attrs**)
 `--snip--`
```

列表 10-25：类型检查函数声明

首先，我们查看函数的存储类 ❶。如果是static，该函数将不可全局可见，因为它的链接是内部的。如果是extern（或完全没有该声明，效果相同），我们暂时认为该函数是全局可见的，因为它的链接是外部的。然而，这可能会根据其他声明的作用域发生变化。

接下来，我们查看是否有其他声明 ❷。我们检查类型不匹配和重复定义，就像在上一章一样。然后，我们考虑链接性。如果当前声明包含显式或隐式的extern关键字，我们将保留先前声明的链接性（因此保留其global属性）。如果当前和过去的声明都有内部链接性，则没有冲突。无论哪种情况，之前声明的链接性保持不变 ❸。但是，如果函数之前声明为外部链接性，而现在声明为static关键字，则声明发生冲突，因此会抛出错误。

我已将此函数的其余部分删减掉，因为它与上一章相同。

##### 文件范围变量声明

当我们遇到文件范围的变量声明时，我们需要确定该变量的初始值以及是否全局可见。这些属性依赖于当前声明和任何之前对同一变量的声明。清单 10-26 展示了如何进行文件范围变量声明的类型检查。

```
typecheck_file_scope_variable_declaration(decl, symbols):
    if decl.init is constant integer i: ❶
        initial_value = Initial(i)
    else if decl.init is null: ❷
        if decl.storage_class == Extern:
            initial_value = NoInitializer
        else:
            initial_value = Tentative
    else: ❸
        fail("Non-constant initializer!")

    global = (decl.storage_class != Static) ❹

    if decl.name is in symbols: ❺
        old_decl = symbols.get(decl.name)
        if old_decl.type != Int:
            fail("Function redeclared as variable")
        if decl.storage_class == Extern:
            global = old_decl.attrs.global
        else if old_decl.attrs.global != global:
            fail("Conflicting variable linkage")

        if old_decl.attrs.init is a constant:
            if initial_value is a constant:
                fail("Conflicting file scope variable definitions") ❻
            else:
                initial_value = old_decl.attrs.init
        else if initial_value is not a constant and old_decl.attrs.init == Tentative:
            initial_value = Tentative

 attrs = StaticAttr(init=initial_value, global=global)
    symbols.add(decl.name, Int, attrs=attrs) ❼
```

清单 10-26：文件范围变量声明的类型检查

首先，我们确定变量的初始值。这取决于声明的初始化器和其存储类说明符。如果初始化器是常量，我们将使用它 ❶。如果没有初始化器 ❷，我们将记录该变量是暂时定义的，还是根本未定义，这取决于是否是extern声明。如果初始化器是任何常量以外的表达式，我们将抛出错误 ❸。

接下来，我们确定该变量是否全局可见 ❹。除非存储类说明符是static，我们暂时认为它是可见的。

然后，如果我们在符号表中记录了此标识符之前的声明，我们也会考虑这些声明 ❺。我们验证之前的声明是否为类型Int，而不是函数类型，然后我们尝试调和global属性与之前声明的匹配。如果这是一个extern声明，我们只需采用先前声明的global属性。否则，如果新的和旧的global属性不一致，我们会抛出错误。

考虑前一个声明的初始化器更加复杂。如果此声明或前一个声明有显式初始化器，我们将使用该初始化器。否则，如果新声明或前一个声明是暂时定义，我们将使用Tentative初始化器。如果到目前为止我们还没有看到任何显式或暂时定义，我们将坚持使用NoInitializer。如果新旧声明都有显式初始化器，我们将抛出一个错误❻。

最后，我们在符号表中添加（或更新）此变量的条目❼。

##### 块作用域变量声明

我们将使用 Listing 10-27 中的伪代码来对块作用域中的变量声明进行类型检查。

```
typecheck_local_variable_declaration(decl, symbols):
    if decl.storage_class == Extern:
        if decl.init is not null: ❶
            fail("Initializer on local extern variable declaration")
        if decl.name is in symbols:
            old_decl = symbols.get(decl.name)
            if old_decl.type != Int: ❷
                fail("Function redeclared as variable")
        else:
            symbols.add(decl.name, Int, attrs=StaticAttr(init=NoInitializer, global=True)) ❸

    else if decl.storage_class == Static:
        if decl.init is constant integer i: ❹
            initial_value = Initial(i)
        else if decl.init is null: ❺
            initial_value = Initial(0)
 else:
            fail("Non-constant initializer on local static variable")
        symbols.add(decl.name, Int, attrs=StaticAttr(init=initial_value, global=False)) ❻

    else:
        symbols.add(decl.name, Int, attrs=LocalAttr) ❼
        if decl.init is not null:
            typecheck_exp(decl.init, symbols)
```

Listing 10-27: 类型检查块作用域变量声明

为了处理extern变量，我们首先确保它没有初始化器❶，并且之前没有声明为函数❷。然后，如果该变量之前没有声明，我们将在符号表中记录它是全局可见且未初始化❸。如果它已经声明过，我们什么都不做：局部的extern声明永远不会改变我们已经记录的初始值或链接。

静态局部变量没有链接，因此我们不需要考虑早期的声明。我们只检查变量的初始化器：如果它是常量，我们使用它❹；如果它不存在，我们将变量初始化为零❺；如果它不是常量，我们抛出一个错误。然后，我们将该变量添加到符号表中，记录它不可全局可见❻。

我们将在符号表中为自动变量的条目中包含LocalAttr属性❼。除此之外，我们像上一章那样对这些变量进行类型检查。

当你处理for循环头中的声明时，验证它是否没有包含存储类说明符，然后再调用 Listing 10-27 中的代码。（或者，你可以在标识符解析阶段处理此错误情况，甚至在解析过程中处理。）

类型检查阶段完成了！实现 C 标准中有关定义、声明、链接和存储持续时间的复杂规则花费了不少精力。幸运的是，现在符号表已经包含了我们需要的所有信息，接下来的章节应该会轻松很多。

### TACKY 生成

我们需要在 TACKY IR 中进行两个新增。首先，我们将在函数定义中添加一个新的 global 字段，这对应于最终汇编输出中的 .globl 指令：

```
Function(identifier, **bool global**, identifier* params, instruction* body)
```

第二步，我们将添加一个顶层构造来表示静态变量：

```
StaticVariable(identifier, bool global, int init)
```

我们将使用此构造来表示外部和局部静态变量。最终，我们将把每个 StaticVariable 构造转换为一组汇编指令，用于初始化数据段或 BSS 段中的对象。列表 10-28 展示了完整的 TACKY IR，并对上一章的更改进行了加粗。

```
program = Program(**top_level***)
**top_level** = Function(identifier, **bool global**, identifier* params, instruction* body)
          **| StaticVariable(identifier, bool global, int init)**
instruction = Return(val)
            | Unary(unary_operator, val src, val dst)
            | Binary(binary_operator, val src1, val src2, val dst)
            | Copy(val src, val dst)
            | Jump(identifier target)
            | JumpIfZero(val condition, identifier target)
            | JumpIfNotZero(val condition, identifier target)
            | Label(identifier)
            | FunCall(identifier fun_name, val* args, val dst)
val = Constant(int) | Var(identifier)
unary_operator = Complement | Negate | Not
binary_operator = Add | Subtract | Multiply | Divide | Remainder | Equal | NotEqual
                | LessThan | LessOrEqual | GreaterThan | GreaterOrEqual
```

列表 10-28：向 TACKY 添加静态变量和 global 属性

我们已将 function_definition 节点重命名为 top_level，因为它不再仅仅表示函数。请注意，当我们将程序转换为 TACKY 时，我们将局部静态变量定义移到顶层；它们变成了 StaticVariable 构造，而不是函数体中的指令。

当我们遍历抽象语法树（AST）并将其转换为 TACKY 时，我们将在每个顶层的 Function 上设置新的 global 属性。我们可以在符号表中查找此属性。对于文件作用域变量声明或带有 static 或 extern 说明符的局部变量声明，我们不会生成任何 TACKY。相反，*在*我们遍历 AST 后，我们将执行一个额外的步骤，检查符号表中的每个条目，并为其中一些条目生成 StaticVariable 构造。我们的最终 TACKY 程序将包括从原始 AST 转换来的函数定义和从符号表生成的变量定义。

列表 10-29 演示了如何将符号表条目转换为 TACKY 变量定义。

```
convert_symbols_to_tacky(symbols):
    tacky_defs = []
    for (name, entry) in symbols:
        match entry.attrs with
        | StaticAttr(init, global) ->
            match init with
            | Initial(i) -> tacky_defs.append(StaticVariable(name, global, i))
            | Tentative -> tacky_defs.append(StaticVariable(name, global, 0))
            | NoInitializer -> continue
        | _ -> continue
    return tacky_defs
```

列表 10-29：将符号表条目转换为 TACKY

我们查看每个符号表条目，以确定它是否应该转换为 StaticVariable。如果它没有 StaticAttr 属性，我们就跳过它，因为它不是静态变量。如果它的初始值是 NoInitializer，我们也跳过它，因为它在此翻译单元中未定义。任何没有被跳过的符号都会转换为 TACKY StaticVariable 并添加到 TACKY 程序中。具有临时定义的静态变量将初始化为零。

现在，先处理 AST 还是符号表并不重要。从 第十六章 开始，先处理 AST 再处理符号表将变得非常重要。在该章中，我们将在将 AST 转换为 TACKY 时向符号表中添加新的静态对象；然后，在遍历符号表时，我们将把这些新条目转换为 TACKY 构造。

### 汇编生成

我们将在本章对汇编 AST 进行一些小的修改。这些更改在 列表 10-30 中已加粗。

```
program = Program(**top_level***)
**top_level** = Function(identifier name, **bool global,** instruction* instructions)
          **| StaticVariable(identifier name, bool global, int init)**
instruction = Mov(operand src, operand dst)
            | Unary(unary_operator, operand)
            | Binary(binary_operator, operand, operand)
            | Cmp(operand, operand)
            | Idiv(operand)
            | Cdq
            | Jmp(identifier)
            | JmpCC(cond_code, identifier)
            | SetCC(cond_code, operand)
            | Label(identifier)
            | AllocateStack(int)
            | DeallocateStack(int)
            | Push(operand)
            | Call(identifier)
            | Ret

unary_operator = Neg | Not
binary_operator = Add | Sub | Mult
operand = Imm(int) | Reg(reg) | Pseudo(identifier) | Stack(int) | **Data(identifier)**
cond_code = E | NE | G | GE | L | LE
reg = AX | CX | DX | DI | SI | R8 | R9 | R10 | R11
```

列表 10-30：带静态变量的汇编 AST

就像在 TACKY 中一样，我们将 function_definition 重命名为 top_level，并添加一个顶层 StaticVariable，表示每个静态变量的名称、初始值以及它是否在全局可见。我们还会给函数定义添加一个 global 属性。最后，我们添加一个新的汇编操作数 Data，用于对数据和 BSS 段的 RIP 相对访问。在伪寄存器替换过程中，我们将根据需要用 Data 操作数替换伪寄存器。

#### 生成变量定义的汇编

将我们的新 TACKY 构造转换为汇编是简单的，因为我们只需将一些字段从 TACKY 传递到相应的汇编构造。表 10-3 总结了此转换的最新更新，新的构造和现有构造的更改已加粗。附录 B 包含了本章的完整 TACKY 到汇编转换过程，这也是 Part I 中此过程的最终版本。

表 10-3： 将顶层 TACKY 构造转换为汇编

| TACKY 顶层结构 | 汇编顶层结构 |
| --- | --- |
| 程序（top_level_defs） | 程序（top_level_defs） |
| 函数（name， global， params， instructions） |

```
Function(name, global, 
 [Mov(Reg(DI), param1),
          Mov(Reg(SI), param2), 
           <copy next four parameters from registers>, 
 Mov(Stack(16), param7),
          Mov(Stack(24), param8), 
           <copy remaining parameters from stack>] +
        instructions)
```

|

| 静态变量（name，global，init） | 静态变量（name，global，init） |
| --- | --- |

我们将所有其他 TACKY 结构转换为汇编的方式不会改变。特别是，我们将每个 TACKY Var 操作数转换为汇编中的 Pseudo 操作数，无论它是具有静态存储持续时间还是自动存储持续时间。这意味着名称 Pseudo 不再完全适用；术语 *伪寄存器* 通常指的是那些理论上可以驻留在寄存器中的操作数，而静态变量不能这么做。我们不会费心去重新命名这个操作数，但你应该记住，我们在使用 *伪寄存器* 这个术语时有些不太常规。

#### 根据存储持续时间替换伪寄存器

接下来，我们将调整如何用具体位置替换伪寄存器。在之前的章节中，每个伪寄存器都被分配到了栈上的一个位置。这一次，并非每个变量都应该放在栈上；其中一些变量存储在数据区或 BSS 段。我们将检查符号表来区分它们。回想一下，我们在伪寄存器替换过程中建立了一个从伪寄存器到具体地址的映射。当我们遇到一个不在此映射中的伪寄存器时，我们会在符号表中查找它。如果我们发现它具有静态存储持续时间，我们将把它映射到具有相同名称的 Data 操作数上。否则，我们将像往常一样为它分配栈上的新槽位。（如果它不在符号表中，说明它是一个 TACKY 临时变量，因此具有自动存储持续时间。）例如，如果 foo 是一个静态变量，那么汇编指令

```
Mov(Imm(0), Pseudo("foo"))
```

应该重写为：

```
Mov(Imm(0), Data("foo"))
```

因为静态变量不驻留在栈上，所以它们不会计入我们需要追踪的每个函数的总栈大小。

#### 修正指令

你已经编写了多个重写规则，这些规则适用于操作数为内存地址的情况。记住，Data 操作数也是内存地址！例如，如果你遇到以下指令：

```
Mov(Data("x"), Stack(-4))
```

你应该为 Mov 指令应用通常的重写规则，前提是源操作数和目标操作数都在内存中。重写后的汇编代码将是：

```
Mov(Data("x"), Reg(R10))
Mov(Reg(R10), Stack(-4))
```

否则，此阶段不会有变化。

### 代码生成

为了结束本章，你将扩展代码生成阶段来处理 Listing 10-30 中的更改。你应根据汇编 AST 中 global 属性为函数包含或省略 .globl 指令。你还应该在每个函数定义的开始处包含 .text 指令。此指令告诉汇编器写入文本段；现在你也要写入数据段和 BSS 段，因此需要包括此指令。

使用 RIP 相对寻址生成 Data 操作数。例如，Data("foo") 在 Linux 上会变为 foo(%rip)，在 macOS 上会变为 _foo(%rip)。将每个 StaticVariable 生成一组汇编指令。在 Linux 上，如果你有一个 StaticVariable(name, global, init)，且 global 为 true 且 init 非零，你应该生成 Listing 10-31 中的汇编代码。

```
 .globl `<name>`
    .data
    .align 4
`<name>`:
    .long `<init>`
```

Listing 10-31: 全局非零静态变量的汇编代码

如果 global 为 true 且 init 为零，你应该生成 Listing 10-32 中的汇编代码。

```
 .globl `<name>`
    .bss
    .align 4
`<name>`:
    .zero 4
```

Listing 10-32: 全局静态变量的汇编代码，初始化为零

如果 global 为 false，生成 Listing 10-31 或 10-32，而不包含 .globl 指令。

在 macOS 上，你将发出几乎相同的汇编代码用于 StaticVariable，只是有一些细微的差别。首先，符号应当以下划线开始，和往常一样。其次，你应使用 .balign 指令，而不是 .align。我之前提到过，.align 指令的行为是平台相关的，所以 .align 4 在 macOS 上会生成 16 字节对齐的值。.balign 指令的工作方式与 .align 相同，只是它的行为在不同平台之间保持一致：.balign n 总是将值对齐到 *n* 字节，而不是 2*^n* 字节。（在 Linux 上，.balign 和 .align 是可以互换的，因此使用其中任何一个都可以。）

表格 10-4 和 10-5 总结了代码发射过程中的最新更新，新构造和对现有构造的更改已加粗。 附录 B 包含了本章的完整代码发射过程（这也是 第一部分 的完整代码发射过程）。

表格 10-4： 格式化顶层汇编构造

| 汇编顶层构造 |  | 输出 |
| --- | --- | --- |
| 程序（顶层） |

```
Print out each top-level construct. 
On Linux, add at end of file: 
 .section .note.GNU-stack,"",@progbits
```

|

| 函数（名称， 全局，指令） |
| --- |

```
  <global-directive> 
    .text 
<name>:
    pushq    %rbp
    movq     %rsp, %rbp     
    <instructions>
```

|

| StaticVariable（名称，全局，初始化） | 初始化为零 |
| --- | --- |

```
  <global-directive> 
 .bss 
 <alignment-directive>
<name>:
    .zero 4
```

|

| 初始化为非零值 |
| --- |

```
  <global-directive> 
 .data 
    <alignment-directive>
<name>:
    .long <init>
```

|

| 全局指令 |
| --- |

```
If global is true:
.globl <identifier> 
Otherwise, omit this directive.
```

|

| 对齐指令 | 仅限 Linux | .align 4 |
| --- | --- | --- |
|  | macOS 或 Linux | .balign 4 |

表 10-5： 格式化汇编操作数

| 汇编操作数 | 输出 |
| --- | --- |
| 数据（标识符） | <标识符>（%rip） |

一旦你更新了代码生成过程，就可以开始测试你的编译器了。

### 总结

你刚刚完成了第一部分的内容！你的编译器可以处理具有各种链接方式的标识符，以及静态和自动存储持续时间的标识符。你还学会了如何编写汇编程序，以定义和使用目标文件中数据和 BSS 段的值。

你现在已经实现了 C 语言的所有基本机制，从局部变量和文件作用域变量到控制流语句再到函数调用。你还通过区分函数类型和int，迈出了建立类型系统的第一步。在第二部分中，你将实现更多类型，包括不同大小的有符号和无符号整数、浮点数、指针、数组和结构体。或者，如果你愿意，你也可以直接跳到第三部分，在那里你将实现几种经典的编译器优化。到目前为止，你所做的工作为你接下来决定继续学习的部分奠定了坚实的基础。
