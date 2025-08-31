

# 第十章：10 程序结构



*与阿龙·巴尔曼合作*

![](img/opener.jpg)

任何现实世界的系统都由多个组件构成，如源文件、头文件和库。许多组件包含资源，包括图像、声音和配置文件。从更小的逻辑组件组成程序是一种良好的软件工程实践，因为这些组件比一个单独的大文件更容易管理。

在本章中，你将学习如何将程序结构化为多个单元，这些单元包含源文件和头文件。你还将学习如何将多个目标文件链接在一起，创建库和可执行文件。

## 组件化原则

没有什么阻止你在单一源文件的< samp class="SANS_TheSansMonoCd_W5Regular_11">main函数中编写整个程序。然而，随着函数的增长，这种方法将迅速变得难以管理。因此，将程序分解为一组组件，通过共享边界或*接口*交换信息是有意义的。将源代码组织成组件使得它更易于理解，并允许你在程序的其他地方甚至与其他程序一起重用代码。

理解如何最佳地分解程序通常需要经验。程序员做出的许多决策都是由性能驱动的。例如，你可能需要最小化通过高延迟接口的通信。糟糕的硬件只能走这么远；你需要糟糕的软件才能真正破坏性能。

性能只是软件质量属性之一（ISO/IEC 25000:2014），必须与可维护性、代码可读性、可理解性、安全性和安全性平衡。例如，你可能会设计一个客户端应用程序来处理来自用户界面的输入字段验证，以避免到服务器的往返。这有助于性能，但如果服务器的输入没有验证，可能会危害安全性。一个简单的解决方案是在两个地方都验证输入。

开发人员常常做一些奇怪的事情来获得虚幻的收益。其中最奇怪的是通过调用有符号整数溢出的未定义行为来提高性能。通常，这些局部代码优化对整体系统性能没有影响，且被视为*过早的优化*。《计算机程序设计的艺术》（Addison-Wesley，1997）的作者**唐纳德·克努斯**将过早优化描述为“所有邪恶的根源”。

在本节中，我们将介绍一些基于组件的软件工程原则。

### 耦合与内聚

除了性能外，一个结构良好的程序的目标是实现诸如低耦合和高内聚等理想特性。*内聚*是衡量编程接口各元素之间共同性的标准。例如，假设一个头文件暴露了计算字符串长度、计算给定输入值的正切和创建线程的函数。这个头文件的内聚性较低，因为暴露的函数彼此无关。相反，一个暴露了计算字符串长度、连接两个字符串以及在字符串中查找子字符串的函数的头文件则具有较高的内聚性，因为所有功能都是相关的。这样，如果你需要处理字符串，你只需包含字符串头文件。类似地，构成公共接口的相关函数和类型定义应由同一个头文件暴露，以提供一个高度内聚且功能有限的接口。我们将在《数据抽象》一章中进一步讨论公共接口，见第 215 页。

*耦合*是衡量编程接口相互依赖程度的标准。例如，一个紧耦合的头文件不能单独被包含进程序中；相反，它必须与其他头文件按特定顺序一起包含。你可能因为多种原因将接口耦合在一起，比如共同依赖数据结构、函数之间的相互依赖或使用共享的全局状态。但当接口紧耦合时，修改程序行为变得困难，因为更改可能会对系统产生连锁反应。无论这些接口是公共接口的成员还是程序实现的细节，你都应始终力求保持接口组件之间的松耦合。

通过将程序逻辑分离成不同的、高内聚的组件，你可以更容易地推理各个组件的行为并测试程序（因为你可以独立验证每个组件的正确性）。结果是一个更易于维护、错误更少的系统。

### 代码重用

*代码重用*是一次性实现功能并在程序的不同部分重复使用，而不重复编写相同代码的做法。代码重复可能会导致微妙的意外行为、庞大臃肿的可执行文件以及增加维护成本。再说了，为什么要多次编写相同的代码呢？

*函数*是最低级别的可重用功能单元。任何你可能会重复多次的逻辑，都可以考虑封装成一个函数。如果功能之间只有细微差别，你可能能够创建一个参数化的函数，来实现多个用途。每个函数应当执行其他函数未重复的工作。然后，你可以将单独的函数组合起来，解决越来越复杂的问题。

将可重用的逻辑封装成函数可以提高可维护性并消除缺陷。例如，尽管你可以通过编写一个简单的 for 循环来确定一个以空字符结尾的字符串的长度，但使用 C 标准库中的 strlen 函数会更具可维护性。因为其他程序员已经熟悉 strlen 函数，他们更容易理解该函数的作用，而不是理解 for 循环的作用。此外，如果你重用现有的功能，就不太可能在临时实现中引入行为差异，还可以更容易地用性能更高或更安全的算法或实现来全局替换功能。

在设计功能接口时，必须在 *通用性* 和 *特定性* 之间找到平衡。一个特定于当前需求的接口可能非常简洁有效，但当需求变化时，修改起来会很困难。一个通用接口可能适应未来的需求，但对于可预见的需求来说可能会显得繁琐。

### 数据抽象

*数据抽象* 是任何可重用的软件组件，它强制要求抽象的公共接口与实现细节之间有明确的分离。每个数据抽象的 *公共接口* 包括用户需要的类型定义、函数声明和常量定义，并放置在头文件中。数据抽象的实现细节，以及任何私有的辅助函数，都隐藏在源文件中或放在与公共接口头文件分开的地方。公共接口与私有实现的分离，使你可以在不破坏依赖于该组件的代码的情况下更改实现细节。

*头文件* 通常包含组件的函数声明和类型定义。例如，C 标准库的 <string.h> 头文件提供了与字符串相关功能的公共接口，而 <threads.h> 则提供了线程的实用函数。这样的逻辑分离具有低耦合性和高内聚性，使得你可以更容易地只访问所需的特定组件，减少编译时间和名称冲突的可能性。例如，如果你只需要 strlen 函数，你不需要了解线程应用程序编程接口（API）的任何内容。

另一个考虑因素是，是否应该显式包含你的头文件所需的头文件，还是要求头文件的使用者先自行包含它们。数据抽象最好是自包含的，并包括它们所使用的头文件。没有做到这一点会给抽象的使用者带来负担，并泄露关于数据抽象的实现细节。本书中的示例并不总是遵循这种做法，以保持文件列表的简洁。

*源文件*实现给定头文件声明的功能或执行特定程序所需的应用程序逻辑。例如，如果你有一个描述网络通信公共接口的*network.h*头文件，你可能会有一个*network.c*源文件（或者是*network_win32.c*用于仅 Windows，*network_linux.c*用于仅 Linux），它实现了网络通信逻辑。

可以通过使用头文件在两个源文件之间共享实现细节，但头文件应放在与公共接口不同的位置，以防止无意中暴露实现细节。

一个*集合*是数据抽象的一个很好的例子，它将基本功能与实现或底层数据结构分离开来。集合将数据元素分组，并支持诸如向集合中添加元素、从集合中移除数据元素以及检查集合是否包含特定数据元素等操作。

实现集合的方式有很多种。例如，一个数据元素的集合可以表示为一个平坦数组、一个二叉树、一个有向（可能是无环的）图，或其他不同的结构。数据结构的选择会影响算法的性能，具体取决于你表示的数据类型以及需要表示的数据量。例如，对于需要良好查找性能的大量数据，二叉树可能是更好的抽象，而对于少量固定大小的数据，平坦数组可能是更好的抽象。将集合数据抽象的接口与底层数据结构的实现分离开来，可以使实现方式发生变化，而无需更改依赖于集合接口的代码。

### 不透明类型

数据抽象在与隐藏信息的不透明数据类型一起使用时最为有效。在 C 语言中，*不透明*（或*私有*）数据类型是通过不完整类型表示的，如前向声明的结构体类型。*不完整类型*是描述一个标识符但缺少必要信息以确定该类型对象的大小或布局的类型。隐藏仅供内部使用的数据结构可以防止使用数据抽象的程序员编写依赖于实现细节的代码，因为这些细节可能会发生变化。不完整类型对数据抽象的使用者是可见的，而完全定义的类型仅对实现者可访问。

假设我们想要实现一个支持有限操作的集合，比如添加元素、移除元素和搜索元素。以下示例将<сamp class="SANS_TheSansMonoCd_W5Regular_11">collection_type实现为不透明类型，隐藏数据类型的实现细节，使得库的使用者无法访问。为此，我们创建了两个头文件：一个外部*collection.h*头文件由数据类型的使用者包含，另一个内部头文件仅在实现数据类型功能的文件中包含。

collection.h

```
typedef struct collection * collection_type;
// function declarations
extern errno_t create_collection(collection_type *result);
extern void destroy_collection(collection_type col);
extern errno_t add_to_collection(
  collection_type col, const void *data, size_t byteCount
);
extern errno_t remove_from_collection(
  collection_type col, const void *data, size_t byteCount
);
extern errno_t find_in_collection(
  const collection_type col, const void *data, size_t byteCount
);
// `--snip--`
```

collection_type标识符被别名为struct collection_type（一个不完整类型）。因此，公共接口中的函数必须接受指向此类型的指针，而不是实际的值类型，因为在 C 语言中使用不完整类型时有一定的限制。

在内部头文件中，struct collection_type是完全定义的，但对数据抽象的使用者不可见：

collection_priv.h

```
struct node_type {
  void *data;
  size_t size;
  struct node_type *next;
};

struct collection_type {
  size_t num_elements;
 struct node_type *head;
};
```

数据抽象的使用者仅包括外部*collection.h*文件，而实现抽象数据类型的模块还包括内部定义的*collection_priv.h*文件。这使得collection_type数据类型的实现保持私密。

## 可执行文件

在第九章，我们学习了编译器是一个由多个翻译阶段组成的流水线，编译器的最终输出是目标代码。翻译的最后一个阶段，称为*链接阶段*，将程序中所有翻译单元的目标代码链接在一起，形成最终的可执行文件。这可以是一个用户可以运行的可执行文件，比如*a.out*或*foo.exe*，一个库，或者一个更专业的程序，如设备驱动程序或固件映像（要烧录到只读存储器[ROM]中的机器代码）。链接使你能够将代码分割成独立的源文件，这些源文件可以独立编译，有助于构建可重用的组件。

*库*是不能独立执行的可执行组件。相反，你需要将库集成到可执行程序中。你可以通过在源代码中包含库的头文件并调用已声明的函数来调用库的功能。C 标准库就是一个库的例子——你包含来自库的头文件，但不会直接编译实现库功能的源代码。相反，库的实现随预构建版本的库代码一起提供。

库允许你在他人的工作基础上构建程序的通用组件，从而可以专注于开发你程序中独特的逻辑。例如，在编写视频游戏时，重用现有库应当能够让你专注于开发游戏逻辑，而无需担心用户输入、网络通信或图形渲染的细节。使用一个编译器编译的库通常可以被用在使用不同编译器构建的程序中。

库被链接到你的应用程序中，可以是静态的或动态的。*静态库*，也称为*归档文件*，将其机器码或目标代码直接合并到生成的可执行文件中，这意味着静态库通常与程序的特定版本绑定在一起。由于静态库在链接时被集成，因此静态库的内容可以针对程序使用该库进行进一步优化。程序使用的库代码可以用于链接时优化（例如，使用-flto标志），而未使用的库代码则可以从最终的可执行文件中剥离。

*动态库*，也称为*共享库*或*动态共享对象*，是一个没有启动例程的可执行文件。它可以与可执行文件一起打包，或单独安装，但在可执行文件调用动态库提供的函数时必须可用。许多现代操作系统会将动态库的代码加载到内存中一次，并在所有需要它的应用程序之间共享。你可以在应用程序部署后，根据需要替换不同版本的动态库。

让库与程序分开发展有其自身的优点和风险。例如，开发人员可以在应用程序已经发布后修复库中的 bug，而无需重新编译应用程序。然而，动态库提供了潜在的机会，让恶意攻击者用恶意库替换库，或者最终用户意外使用错误版本的库。也有可能在新库发布时做出*破坏性更改*，导致与使用该库的现有应用程序不兼容。静态库的执行速度可能稍微更快，因为目标代码（可执行文件中的二进制代码）被包含在可执行文件中，从而实现进一步的优化。通常使用动态库的好处大于其缺点。

每个库都有一个或多个头文件，包含库的公共接口，以及一个或多个源文件，实现库的逻辑。即使组件没有被转化为实际的库，通过将代码结构化为库的集合，你也能从中受益。使用实际的库可以减少意外设计紧密耦合接口的可能性，因为在这种接口中，一个组件对另一个组件的内部细节有特殊了解。

## 链接性

*链接性*是一个过程，控制接口是公共的还是私有的，并决定是否有两个标识符指向相同的实体。忽略在翻译阶段早期替换的宏和宏参数，一个*标识符*可以表示一个标准属性、属性前缀或属性名称；一个对象；一个函数；结构体、联合体或枚举的标签或成员；一个typedef名称；或一个标签名称。

C 语言提供了三种链接性：外部链接性、内部链接性或无链接性。每个具有*外部链接性*的标识符声明在程序中的所有地方都指向相同的函数或对象。引用内部链接性声明的标识符仅在包含该声明的翻译单元内指向同一个实体。如果两个翻译单元都引用相同的内部链接性标识符，它们指向的是该实体的不同实例。如果声明没有*链接性*，它在每个翻译单元中都是一个唯一的实体。

声明的链接性要么是显式声明的，要么是隐含的。如果你在文件作用域内声明一个实体，而没有显式指定extern或static，则该实体会被隐式赋予外部链接性。没有链接性的标识符包括函数参数、没有使用extern存储类说明符声明的块作用域标识符或枚举常量。

清单 10-1 显示了每种链接类型声明的示例。

```
static int i; // i has explicit internal linkage
extern void foo(int j) {
  // foo has explicit external linkage
  // j has no linkage because it is a parameter
}
```

清单 10-1：内部链接、外部链接和无链接的示例

如果你在文件作用域内显式声明一个标识符为 static 存储类说明符，它将具有内部链接。static 关键字仅对文件作用域的实体赋予内部链接。如果你在块作用域内将一个变量声明为 static，它将创建一个无链接的标识符，但它确实为该变量提供了静态存储持续时间。提醒一下，静态存储持续时间意味着它的生命周期是程序的整个执行过程，而且它的值只会初始化一次，在程序启动之前。*static* 在不同上下文中的不同含义显然是令人困惑的，因此它常常成为面试中的一个常见问题。

你可以通过使用 extern 存储类说明符声明一个外部链接标识符。只有在你之前没有声明该标识符的链接时，这才有效。如果之前的声明已经为该标识符指定了链接，那么 extern 存储类说明符将不起作用。

声明中有冲突的链接可能会导致未定义行为；有关更多信息，请参阅 CERT C 规则 DCL36-C，“不要声明具有冲突链接分类的标识符”。

清单 10-2 显示了带有隐式链接的示例声明。

foo.c

```
void func(int i) {// implicit external linkage
  // i has no linkage
}
static void bar(); // internal linkage, different bar from bar.c
extern void bar() {
  // bar still has internal linkage because the initial declaration
  // was declared as static; this extern specifier has no effect
}
```

清单 10-2：隐式链接的示例

清单 10-3 显示了带有显式链接的示例声明。

bar.c

```
extern void func(int i); // explicit external linkage
static void bar() {  // internal linkage; different bar from foo.c
  func(12); // calls func from foo.c
}
int i; // external linkage; doesn’t conflict with i from foo.c or bar.c
void baz(int k) {// implicit external linkage
  bar(); // calls bar from bar.c, not foo.c
}
```

清单 10-3：显式链接的示例

你的公共接口中的标识符应该具有外部链接，以便可以从其翻译单元外部调用它们。作为实现细节的标识符应该使用内部链接或无链接进行声明（前提是它们不需要从另一个翻译单元中引用）。实现这一目标的常见方法是在头文件中声明公共接口函数，可以使用也可以不使用 extern 存储类说明符（这些声明默认具有外部链接，但明确声明它们为 extern 也没有坏处），并以类似的方式在源文件中定义公共接口函数。

然而，在源文件中，所有实现细节的声明应显式声明为static，以保持它们的私有性——仅对该源文件可访问。你可以通过使用#include预处理指令来包含头文件中声明的公共接口，从而访问另一个文件中的接口。一个好的经验法则是，不需要在文件外部可见的文件作用域实体应声明为static。这种做法有助于减少全局命名空间污染，并降低翻译单元之间发生意外交互的可能性。

## 构建一个简单的程序

为了学习如何构建一个复杂的真实世界程序，我们先开发一个简单的程序来判断一个数字是否为质数。*质数*（或称为*素数*）是一个自然数，不能通过将两个较小的自然数相乘得到。我们将编写两个独立的组件：一个包含测试功能的静态库和一个提供库用户界面的命令行应用程序。

*primetest*程序接受以空格分隔的整数值列表作为输入，然后输出每个值是否为质数。如果任何输入无效，程序将输出一条有用的消息，解释如何使用该界面。

在探讨如何构建程序之前，我们先来看看用户界面。首先，我们打印命令行程序的帮助文本，如清单 10-4 所示。

```
// print command line help text
static void print_help() {
  puts("primetest num1 [num2 num3 ... numN]\n");
  puts("Tests positive integers for primality.");
  printf("Tests numbers in the range [2-%llu].\n", ULLONG_MAX);
}
```

清单 10-4：打印帮助文本

print_help函数将使用信息打印到标准输出，说明如何使用该命令。

接下来，由于命令行参数以文本形式传递给程序，我们定义了一个实用函数，将它们转换为整数值，如清单 10-5 所示。

```
// converts a string argument arg to an unsigned long long value referenced by val
// returns true if the argument conversion succeeds and false if it fails
static bool convert_arg(const char *arg, unsigned long long *val) {
  char *end;

  // strtoull returns an in-band error indicator; clear errno before the call
  errno = 0;
  *val = strtoull(arg, &end, 10);

  // check for failures where the call returns a sentinel value and sets errno
  if ((*val == ULLONG_MAX) && errno) return false;
  if (*val == 0 && errno) return false;
  if (end == arg) return false;

  // If we got here, the argument conversion was successful.
  // However, we want to allow only values greater than one,
  // so we reject values <= 1.
  if (*val <= 1) return false;
  return true;
}
```

清单 10-5：转换单个命令行参数

<`samp class="SANS_TheSansMonoCd_W5Regular_11">convert_arg`函数接受一个字符串参数作为输入，并使用输出参数报告转换后的结果。*输出参数*通过指针将函数结果返回给调用者，使得除了函数返回值外，还能返回多个值。若参数转换成功，函数返回<`samp class="SANS_TheSansMonoCd_W5Regular_11">true，如果失败则返回<`samp class="SANS_TheSansMonoCd_W5Regular_11">false。`convert_arg`函数使用<`samp class="SANS_TheSansMonoCd_W5Regular_11">strtoull`函数将字符串转换为<`samp class="SANS_TheSansMonoCd_W5Regular_11">unsigned long long整数值，并注意妥善处理转换错误。此外，由于质数的定义排除了 0、1 和负数，`convert_arg`函数会将这些视为无效输入。

我们在 sample 中使用了<`samp class="SANS_TheSansMonoCd_W5Regular_11">convert_arg`工具函数，这个函数位于<`samp class="SANS_TheSansMonoCd_W5Regular_11">convert_cmd_line_args`函数，主要作用是对所有提供的命令行参数进行循环，并尝试将每个参数从字符串转换为整数。

```
static unsigned long long *convert_cmd_line_args(int argc,
                                                 const char *argv[],
                                                 size_t *num_args) {
  *num_args = 0;

  if (argc <= 1) {
    // no command line arguments given (the first argument is the
    // name of the program being executed)
    print_help();
    return nullptr;
  }

  // We know the maximum number of arguments the user could have passed,
  // so allocate an array large enough to hold all the elements. Subtract
  // one for the program name itself. If the allocation fails, treat it as
  // a failed conversion (it is OK to call free(nullptr)).
 unsigned long long *args =
      (unsigned long long *)malloc(sizeof(unsigned long long) * (argc - 1));
  bool failed_conversion = (args == nullptr);
  for (int i = 1; i < argc && !failed_conversion; ++i) {
    // Attempt to convert the argument to an integer. If we
    // couldn't convert it, set failed_conversion to true.
    unsigned long long one_arg;
    failed_conversion |= !convert_arg(argv[i], &one_arg);
    args[i - 1] = one_arg;
  }

  if (failed_conversion) {
    // free the array, print the help, and bail out
    free(args);
    print_help();
    return nullptr;
  }

  *num_args = argc - 1;
  return args;
}
```

清单 10-6：处理所有命令行参数

如果任何一个参数无法转换，它会调用<`samp class="SANS_TheSansMonoCd_W5Regular_11">print_help`函数来向用户报告正确的命令行用法，并返回一个空指针。该函数负责分配一个足够大的缓冲区来存储整数数组。它还处理所有错误情况，比如内存不足或参数转换失败。如果函数成功，它会返回一个整数数组给调用者，并将转换后的参数个数写入<`samp class="SANS_TheSansMonoCd_W5Regular_11">num_args`参数。返回的数组是已分配的存储空间，当不再需要时必须进行释放。

有多种方法可以判断一个数字是否为质数。最直接的方法是通过测试一个值*N*，判断它是否能被[2..*N* – 1]整除。这种方法随着*N*值的增大，性能表现较差。相反，我们将使用一种为质数测试设计的算法。清单 10-7 展示了 Miller-Rabin 质数测试的非确定性实现，适用于快速测试一个值是否可能是质数（Schoof 2008）。请参阅 Schoof 的论文，了解 Miller-Rabin 质数测试算法背后的数学原理。

```
static unsigned long long power(unsigned long long x, unsigned long long y,
                                unsigned long long p) {
  unsigned long long result = 1;
  x %= p;

  while (y) {
    if (y & 1) result = (result * x) % p;
    y >>= 1;
    x = (x * x) % p;
  }
 return result;
}

static bool miller_rabin_test(unsigned long long d, unsigned long long n) {
  unsigned long long a = 2 + rand() % (n - 4);
  unsigned long long x = power(a, d, n);

  if (x == 1 || x == n - 1) return true;

  while (d != n - 1) {
    x = (x * x) % n;
    d *= 2;

    if (x == 1) return false;
    if (x == n - 1) return true;
  }
  return false;
}
```

清单 10-7：Miller-Rabin 质数测试算法

Miller-Rabin 素性测试的接口是 清单 10-8 中显示的 is_prime 函数。该函数接受两个参数：待测试的数字 (n) 和执行测试的次数 (k)。较大的 k 值提供更精确的结果，但会降低性能。我们将把 清单 10-6 中的算法与 is_prime 函数一起放入静态库中，该库将提供公共接口。

```
bool is_prime(unsigned long long n, unsigned int k) {
  if (n <= 1 || n == 4) return false;
  if (n <= 3) return true;

  unsigned long long d = n - 1;
  while (d % 2 == 0) d /= 2;

  for (; k != 0; --k) {
    if (!miller_rabin_test(d, n)) return false;
  }
  return true;
}
```

清单 10-8：Miller-Rabin 素性测试算法的接口

最后，我们需要将这些工具函数组合成一个程序。清单 10-9 展示了 main 函数的实现。它使用固定次数的 Miller-Rabin 测试，并报告输入值是可能是素数还是绝对不是素数。它还负责释放由 convert_cmd_line_args 分配的内存。

```
int main(int argc, char *argv[]) {
  size_t num_args;
  unsigned long long *vals = convert_cmd_line_args(argc, argv, &num_args);

 if (!vals) return EXIT_FAILURE;

  for (size_t i = 0; i < num_args; ++i) {
    printf("%llu is %s.\n", vals[i],
           is_prime(vals[i], 100) ? "probably prime" : "not prime");
  }

  free(vals);
  return EXIT_SUCCESS;
}
```

清单 10-9：  main 函数

main 函数调用 convert_cmd_line_args 函数，将命令行参数转换为 unsigned long long 类型的整数数组。程序对该数组中的每个参数进行循环，调用 is_prime 来判断每个值是可能是素数，还是根据 Miller-Rabin 素性测试判断为非素数。

现在我们已经实现了程序逻辑，我们将生成所需的构建产物。我们的目标是生成一个静态库，其中包含 Miller-Rabin 实现和一个命令行应用程序驱动程序。

## 构建代码

创建一个名为*isprime.c*的新文件，包含来自清单 10-8 和 10-9 的代码（按此顺序），并在文件顶部添加 #include 指令，分别为 "isprime.h" 和 <stdlib.h>。引号和尖括号环绕头文件，对于告诉预处理器在哪里查找这些文件非常重要，正如第九章中讨论的那样。接下来，创建一个名为 *isprime.h* 的头文件，包含来自清单 10-10 的代码，以提供静态库的公共接口，并添加头文件保护。

```
#ifndef PRIMETEST_IS_PRIME_H
#define PRIMETEST_IS_PRIME_H

bool is_prime(unsigned long long n, unsigned k);

#endif // PRIMETEST_IS_PRIME_H
```

清单 10-10：静态库的公共接口

创建一个名为 *driver.c* 的新文件，包含来自清单 10-5、10-6、10-7 和 10-10 的代码（按此顺序），并在文件顶部添加以下 #include 指令："isprime.h"、<assert.h>、<errno.h>、<limits.h>、<stdio.h> 和 <stdlib.h>。在我们的示例中，所有三个文件都在同一个目录中，但在实际项目中，您可能会根据构建系统的约定将文件放置在不同的目录中。创建一个名为 *bin* 的本地目录，用于存放本示例的构建产物。

我们使用 Clang 来创建静态库和可执行程序，但 GCC 和 Clang 都支持示例中的命令行参数，因此两者的编译器都能使用。首先，将两个 C 源文件编译成目标文件，并将它们放置在 *bin* 目录中：

```
% **cc -c -std=c23 -Wall -Wextra -pedantic isprime.c -o bin/isprime.o**
% **cc -c -std=c23 -Wall -Wextra -pedantic driver.c -o bin/driver.o**
```

对于旧版编译器，可能需要将 -std=c23 替换为 -std=c2x。

如果执行命令时出现错误，例如

```
unable to open output file 'bin/isprime.o': 'No such file or directory'
```

然后创建本地 *bin* 目录，并再次尝试该命令。-c 标志指示编译器将源代码编译为目标文件，而不调用链接器生成可执行输出。我们需要目标文件来创建库。-o 标志指定输出文件的路径名。

执行命令后，*bin*目录应包含两个对象文件：*isprime.o*和*driver.o*。这些文件包含每个翻译单元的对象代码。您可以将它们直接链接在一起，生成可执行程序。然而，在这种情况下，我们将创建一个静态库。为此，执行ar命令，在*bin*目录中生成名为*libPrimalityUtilities.a*的静态库：

```
% **ar rcs bin/libPrimalityUtilities.a bin/isprime.o**
```

r选项指示ar命令用新文件替换档案中现有的文件，c选项创建档案，s选项将对象文件索引写入档案（这等同于运行ranlib命令）。这将创建一个单一的档案文件，其结构允许检索用于创建档案的原始对象文件，就像一个压缩的 tarball 或 ZIP 文件。根据约定，Unix 系统上的静态库以*lib*为前缀，文件扩展名为*.a*。

现在，您可以将驱动程序对象文件链接到*libPrimalityUtilities.a*静态库，以生成名为*primetest*的可执行文件。您可以通过不带-c标志来调用编译器，这样会调用默认系统链接器并传递适当的参数，或者直接调用链接器来完成。通过以下方式调用编译器，使用默认系统链接器：

```
% **cc bin/driver.o -Lbin -lPrimalityUtilities -o bin/primetest**
```

-L标志指示链接器在本地*bin*目录中查找要链接的库，-l标志指示链接器将*libPrimalityUtilities.a*库链接到输出中。在命令行参数中省略*lib*前缀和*.a*后缀，因为链接器会自动添加它们。例如，要链接*libm*数学库，指定-lm作为链接目标。与编译源文件一样，链接文件的输出由-o标志指定。

现在，您可以测试程序，看它是否能判断值是可能为素数还是绝对不是素数。一定要尝试负数、已知的素数和非素数，以及不正确的输入，具体请参见 Listing 10-11。

```
% **./bin/primetest 899180**
899180 is not prime
% **./bin/primetest 8675309**
8675309 is probably prime
% **./bin/primetest 0**
primetest num1 [num2 num3 ... numN]

Tests positive integers for primality.
Tests numbers in the range [2-18446744073709551615].
```

Listing 10-11：使用示例输入运行 primetest 程序

数字 8,675,309 是素数。

## 总结

在本章中，你了解了松散耦合、高内聚、数据抽象和代码重用的好处。此外，你还学习了相关的语言构造，如不透明数据类型和链接。你了解了一些关于如何在项目中组织代码的最佳实践，并看到了一个通过不同类型的可执行组件构建简单程序的示例。这些技能在你从编写练习程序转向开发和部署现实世界的系统时非常重要。

在下一章中，我们将学习如何使用各种工具和技术来创建高质量的系统，包括断言、调试、测试，以及静态和动态分析。这些技能都是开发安全、可靠和高效的现代系统所必需的。
