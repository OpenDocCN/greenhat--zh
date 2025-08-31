## 第一章：致 C 程序员的前言

*阿瑟·丹特：他怎么了？ 高·赫滕弗斯特：他的脚与鞋子不匹配。*

—道格拉斯·亚当斯，《银河系漫游指南》，“*适配第十一条*”

![图片](img/common.jpg)

本前言是为有经验的 C 程序员准备的，帮助他们决定是否阅读本书。非 C 程序员可以跳过这一部分。

比雅尔内·斯特劳斯特鲁普（Bjarne Stroustrup）从 C 语言发展出了 C++。尽管 C++ 并不完全兼容 C，但写得好的 C 程序通常也是合法的 C++ 程序。举例来说，Brian Kernighan 和 Dennis Ritchie 编写的《C 程序设计语言》中的每个例子，都是合法的 C++ 程序。

C 语言在系统编程社区广泛使用的一个主要原因是，C 允许程序员以比汇编语言更高的抽象层次进行编程。这通常能产生更清晰、少出错、且更易维护的代码。

一般来说，系统程序员不愿为编程便利性支付额外开销，因此 C 语言遵循零开销原则：*你不用的，就不需要为它付费*。强类型系统就是零开销抽象的典型例子。它只在编译时用于检查程序正确性。编译后，类型信息将消失，生成的汇编代码将不再体现类型系统的痕迹。

作为 C 语言的后代，C++ 也非常重视零开销抽象和直接映射到硬件。这种承诺不仅仅局限于 C++ 支持的 C 语言特性。C++ 在 C 基础上构建的一切，包括新的语言特性，都遵循这些原则，任何偏离这些原则的地方都是经过深思熟虑的。实际上，一些 C++ 特性比对应的 C 代码还要少开销。例如，`constexpr` 关键字就是一个例子。它指示编译器在编译时评估表达式（如果可能的话），如 清单 1 中的程序所示。

```
#include <cstdio>

constexpr int isqrt(int n) {
  int i=1;
  while (i*i<n) ++i;
  return i-(i*i!=n);
}

int main() {
  constexpr int x = isqrt(1764); ➊
  printf("%d", x);
}
```

*清单 1：演示 `constexpr` 的程序*

`isqrt` 函数计算参数 `n` 的平方根。从 `1` 开始，该函数递增局部变量 `i`，直到 `i*i` 大于或等于 `n`。如果 `i*i == n`，则返回 `i`；否则，返回 `i-1`。请注意，`isqrt` 的调用有一个字面值，因此编译器理论上可以为你计算结果。结果最终只会是一个值 ➊。

在 GCC 8.3 上编译 清单 1，目标为 x86-64，使用 `-O2` 优化选项，生成的汇编代码可见于 清单 2。

```
.LC0:
        .string "%d"
main:
        sub     rsp, 8
        mov     esi, 42 ➊
        mov     edi, OFFSET FLAT:.LC0
        xor     eax, eax
        call    printf
        xor     eax, eax
        add     rsp, 8
        ret
```

*清单 2：编译 清单 1 后生成的汇编代码*

这里最显著的结果是 `main` 中的第二条指令 ➊；编译器并不是在运行时计算 `1764` 的平方根，而是计算出结果并输出指令，将 `x` 处理为 `42`。当然，你可以使用计算器计算平方根并手动插入结果，但使用 `constexpr` 提供了许多好处。这种方法可以减少与手动复制粘贴相关的许多错误，使你的代码更加富有表现力。

**注意**

*如果你不熟悉 x86 汇编语言，请参阅《汇编语言艺术》（第 2 版，Randall Hyde 著）和《专业汇编语言》（Richard Blum 著）。*

### 升级到超级 C

现代 C++ 编译器会支持大部分你的 C 编程习惯。这使得你能够轻松接受一些 C++ 语言提供的策略性优点，同时故意避开该语言的深层主题。我们可以将这种 C++ 称为*超级 C*，它有几个值得讨论的原因。首先，经验丰富的 C 程序员可以通过将简单的、策略层面的 C++ 概念应用到他们的程序中，立即受益。其次，超级 C*并非*惯用的 C++。简单地在 C 程序中撒上引用和 `auto` 实例，可能会使你的代码更健壮、更易读，但要想充分利用这些特性，你还需要学习其他概念。第三，在一些苛刻的环境中（例如，嵌入式软件、某些操作系统内核和异构计算），可用的工具链对 C++ 的支持不完全。在这种情况下，你仍然可以从一些 C++ 惯用语中获益，而超级 C 很可能是被支持的。本节介绍了一些可以立即应用到代码中的超级 C 概念。

**注意**

*一些 C 支持的结构在 C++ 中无法使用。请参见本书配套网站的链接部分，* [`ccc.codes`](https://ccc.codes)。

#### *函数重载*

请考虑以下来自标准 C 库的转换函数：

```
char* itoa(int value, char* str, int base);
char* ltoa(long value, char* buffer, int base);
char* ultoa(unsigned long value, char* buffer, int base);
```

这些函数实现相同的目标：它们将一个整型转换为 C 风格的字符串。在 C 语言中，每个函数必须有唯一的名称。但在 C++ 中，只要函数的参数不同，多个函数可以共享相同的名称；这就是所谓的*函数重载*。你可以利用函数重载创建自己的转换函数，正如列表 3 所示。

```
char* toa(int value, char* buffer, int base) {
  --snip--
}

char* toa(long value, char* buffer, int base)
  --snip--
}

char* toa(unsigned long value, char* buffer, int base) {
  --snip--
}

int main() {
  char buff[10];
  int a = 1; ➊
  long b = 2; ➋
  unsigned long c = 3; ➌
  toa(a, buff, 10);
  toa(b, buff, 10);
  toa(c, buff, 10);
}
```

*列表 3：调用重载函数*

每个函数中第一个参数的数据类型不同，因此 C++ 编译器从传递给 `toa` 的参数中获得足够的信息，以调用正确的函数。每次 `toa` 调用都是指向一个唯一的函数。这里，你创建了变量 `a` ➊、`b` ➋ 和 `c` ➌，它们是不同类型的 `int` 对象，对应于三个 `toa` 函数中的一个。这比定义不同名称的函数更方便，因为你只需要记住一个名称，编译器会搞清楚调用哪个函数。

#### *参考资料*

指针是 C 语言（以及扩展到大多数系统编程）的一个关键特性。它们通过传递数据地址而不是实际数据，使你能够高效地处理大量数据。指针对 C++也同样重要，但你可以使用额外的安全特性来防止空指针解引用和无意的指针重新赋值。

*引用*是对指针处理的重大改进。它们与指针类似，但有一些关键的区别。从语法上讲，引用与指针在两个重要方面有所不同。首先，你使用`&`来声明引用，而不是`*`，正如示例 4 所展示的那样。

```
struct HolmesIV {
  bool is_sentient;
  int sense_of_humor_rating;
};
void make_sentient(HolmesIV*); // Takes a pointer to a HolmesIV
void make_sentient(HolmesIV&); // Takes a reference to a HolmesIV
```

*示例 4：展示如何声明接受指针和引用的函数的代码*

其次，你使用点操作符`.`与成员进行交互，而不是箭头操作符`->`，正如示例 5 所示。

```
void make_sentient(HolmesIV* mike) {
  mike->is_sentient = true;
}

void make_sentient(HolmesIV& mike) {
  mike.is_sentient = true;
}
```

*示例 5：演示点操作符和箭头操作符使用的程序*

在底层，引用等同于指针，因为它们也是一种零开销的抽象。编译器生成的代码相似。为了说明这一点，考虑在 GCC 8.3 上编译`make_sentient`函数的结果，目标架构为 x86-64，使用`-O2`优化选项。示例 6 包含了通过编译示例 5 生成的汇编代码。

```
make_sentient(HolmesIV*):
        mov     BYTE PTR [rdi], 1
        ret
make_sentient(HolmesIV&):
        mov     BYTE PTR [rdi], 1
        ret
```

*示例 6：通过编译示例 5 生成的汇编代码*

然而，在编译时，引用相比原始指针提供了一些安全性，因为一般来说，引用不能为 null。

对于指针，你可能会添加一个`nullptr`检查以确保安全。例如，你可能会对`make_sentient`添加检查，就像在示例 7 中所示的那样。

```
void make_sentient(HolmesIV* mike) {
  if(mike == nullptr) return;
  mike->is_sentient = true;
}
```

*示例 7：对示例 5 中的`make_sentient`函数进行重构，以执行`nullptr`检查*

在使用引用时，这样的检查是不必要的；然而，这并不意味着引用总是有效的。考虑以下函数：

```
HolmesIV& not_dinkum() {
  HolmesIV mike;
  return mike;
}
```

`not_dinkum`函数返回一个引用，该引用保证非 null。但它指向的是垃圾内存（可能是从`not_dinkum`返回的栈帧中）。你绝不能这样做。结果将是彻底的痛苦，也就是*未定义的运行时行为*：它可能崩溃，可能给出错误，或者可能做出完全意想不到的事情。

引用的另一个安全特性是它们不能被*重新设置*。换句话说，一旦引用被初始化，就不能再指向另一个内存地址，正如示例 8 所示。

```
int main() {
  int a = 42;
  int& a_ref = a; ➊
  int b = 100;
  a_ref = b; ➋
}
```

*示例 8：演示引用不能被重新设置的程序*

你将`a_ref`声明为对`int a`的引用 ➊。无法重新为`a_ref`指向另一个`int`。你可能尝试使用赋值操作符`=`重置`a` ➋，但这实际上是将`a`的值设置为`b`的值，而不是将`a_ref`设置为引用`b`。在该代码片段运行后，`a`和`b`都等于`100`，并且`a_ref`仍然指向`a`。清单 9 提供了使用指针的等效代码。

```
int main() {
  int a = 42;
  int* a_ptr = &a; ➊
  int b = 100;
  *a_ptr = b; ➋
}
```

*清单 9：使用指针的等效程序，参考清单 8*

在这里，你使用`*`声明指针，而不是`&` ➊。你将`b`的值赋给`a_ptr`指向的内存 ➋。使用引用时，你不需要在等号左边加任何装饰。但如果你省略`*`，例如在`*a_ptr`中，编译器会抱怨你试图将`int`类型赋给指针类型。

引用实际上是具有额外安全防护和一些语法糖的指针。当你将引用放在等号的左侧时，你是在将右侧等号的值赋给指针所指向的值。

#### *auto 初始化*

C 语言通常要求你重复多次类型信息，而在 C++中，你只需使用`auto`关键字一次，就可以表达变量的类型信息。编译器将知道变量的类型，因为它知道用于初始化变量的值的类型。考虑以下 C++变量初始化示例：

```
int x = 42;
auto y = 42;
```

在这里，`x`和`y`都是`int`类型。你可能会惊讶地发现编译器能够推导出`y`的类型，但请注意，42 是一个整数字面量。使用`auto`时，编译器会推导出等号右侧的类型`=`，并将变量的类型设置为相同类型。由于整数字面量是`int`类型，因此在此示例中，编译器推导出`y`的类型也是`int`。在如此简单的示例中，这似乎没有太大好处，但请考虑用一个函数的返回值初始化变量，如清单 10 所示。

```
#include <cstdlib>

struct HolmesIV {
  --snip--
};
HolmesIV* make_mike(int sense_of_humor) {
  --snip--
}

int main() {
  auto mike = make_mike(1000);
  free(mike);
}
```

*清单 10：一个使用函数返回值初始化变量的玩具程序*

`auto`关键字更易读，且比显式声明变量类型更有利于代码重构。如果你在声明函数时自由使用`auto`，当你需要更改`make_mike`的返回类型时，你将需要做的工作会更少。随着代码复杂性增加，特别是涉及到标准库中模板代码时，`auto`的优势更加明显。`auto`关键字使编译器为你做所有类型推导的工作。

**注意**

*你也可以在`auto`后添加`const`、`volatile`、`&`和`*`限定符。*

#### *命名空间和结构体、联合体和枚举的隐式类型定义*

C++将类型标签视为隐式`typedef`名称。在 C 语言中，当你想使用`struct`、`union`或`enum`时，你必须使用`typedef`关键字为你创建的类型指定一个名称。例如：

```
typedef struct Jabberwocks {
  void* tulgey_wood;
  int is_galumphing;
} Jabberwock;
```

在 C++中，你可能会对这样的代码嗤之以鼻。因为`typedef`关键字可以是隐式的，C++允许你像这样声明`Jabberwock`类型：

```
struct Jabberwock {
  void* tulgey_wood;
  int is_galumphing;
};
```

这样做更加方便，并且可以节省一些输入时间。如果你还想定义一个`Jabberwock`函数会怎样呢？嗯，你不应该这么做，因为将数据类型和函数使用相同的名称可能会引起混淆。不过，如果你真的决定这么做，C++允许你声明一个`namespace`来为标识符创建不同的作用域。这有助于保持用户类型和函数的整洁，如列表 11 所示。

```
#include <cstdio>

namespace Creature { ➊
  struct Jabberwock {
    void* tulgey_wood;
    int is_galumphing;
  };
}
namespace Func { ➋
  void Jabberwock() {
    printf("Burble!");
  }
}
```

*列表 11：使用命名空间消除具有相同名称的函数和类型的歧义*

在这个例子中，`Jabberwock`结构体和`Jabberwock`函数现在和谐共存。通过将每个元素放置在自己的`namespace`中——结构体放在`Creature`命名空间 ➊，函数放在`Func`命名空间 ➋——你就能消除歧义，明确你指的是哪个 Jabberwock。你可以通过几种方式来消除歧义。最简单的方法是用它的`namespace`来限定名称，例如：

```
Creature::Jabberwock x;
Func::Jabberwock();
```

你还可以使用`using`指令导入`namespace`中的所有名称，这样你就不再需要使用完全限定的元素名称了。列表 12 使用了`Creature`命名空间。

```
#include <cstdio>

namespace Creature {
  struct Jabberwock {
    void* tulgey_wood;
    int is_galumphing;
  };
}

namespace Func {
  void Jabberwock() {
    printf("Burble!");
  }
}

using namespace Creature; ➊

int main() {
  Jabberwock x; ➋
  Func::Jabberwock();
}
```

*列表 12：使用`using namespace`来引用`Creature`命名空间中的类型*

`using namespace` ➊使你能够省略`namespace`限定符 ➋。但你仍然需要在`Func::Jabberwock`前加上限定符，因为它不属于`Creature`命名空间。

使用`namespace`是 C++的惯用法，是一种零开销的抽象。就像类型的其他标识符一样，`namespace`在编译器生成汇编代码时会被去除。在大型项目中，它对于将不同库的代码进行分离非常有帮助。

#### *C 和 C++目标文件的混合使用*

如果你小心操作，C 和 C++代码是可以和平共存的。有时，C 编译器需要链接由 C++编译器生成的目标文件（反之亦然）。虽然这是可能的，但需要一些额外的工作。

有两个问题与链接文件相关。首先，C 和 C++ 代码中的调用约定可能不匹配。例如，调用函数时堆栈和寄存器的设置协议可能不同。这些调用约定是语言级别的不匹配，通常与函数的编写方式无关。其次，C++ 编译器生成的符号与 C 编译器不同。有时，链接器必须通过名称识别一个对象。C++ 编译器通过修饰对象，将一个名为 *修饰名* 的字符串与对象关联，来提供帮助。由于函数重载、调用约定和 `namespace` 的使用，编译器必须通过装饰对函数进行额外的信息编码，而不仅仅是它的名称。这是为了确保链接器能够唯一地识别该函数。不幸的是，C++ 中关于如何进行修饰没有标准（这就是为什么在链接翻译单元时，你应该使用相同的工具链和设置）。C 链接器不了解 C++ 名称修饰，如果在 C++ 中链接 C 代码时没有抑制修饰（反之亦然），这可能会引发问题。

解决方法很简单。你只需使用 `extern "C"` 语句包裹你希望以 C 风格链接的代码，如清单 13 所示。

```
// header.h
#ifdef __cplusplus
extern "C" {
#endif
void extract_arkenstone();

struct MistyMountains {
  int goblin_count;
};
#ifdef __cplusplus
}
#endif
```

*清单 13：使用 C 风格链接*

这个头文件可以在 C 和 C++ 代码之间共享。之所以可行，是因为 `__cplusplus` 是一个 C++ 编译器定义的特殊标识符（但 C 编译器没有定义）。因此，C 编译器在预处理完成后会看到清单 14 中的代码。清单 14 显示了剩余的代码。

```
void extract_arkenstone();

struct MistyMountains {
  int goblin_count;
};
```

*清单 14：在 C 环境中，预处理器处理清单 13 后剩下的代码*

这只是一个简单的 C 头文件。在预处理过程中，`#ifdef __cplusplus` 语句之间的代码会被移除，因此 `extern "C"` 包裹器不可见。对于 C++ 编译器，`__cplusplus` *在* `header.h` 中定义，因此它会看到清单 15 的内容。

```
extern "C" {
  void extract_arkenstone();

  struct MistyMountains {
    int goblin_count;
  };
}
```

*清单 15：在 C++ 环境中，预处理器处理清单 13 后剩下的代码*

现在 `extract_arkenstone` 和 `MistyMountains` 都已用 `extern "C"` 包裹，因此编译器知道使用 C 链接。现在你的 C 源代码可以调用已编译的 C++ 代码，你的 C++ 源代码也可以调用已编译的 C 代码。

### C++ 主题

本节将简要介绍一些使 C++ 成为首选系统编程语言的核心主题。无需过于担心细节。以下小节的重点是激发你的兴趣。

#### *简洁表达思想和重用代码*

精心编写的 C++ 代码具有优雅和紧凑的特质。考虑以下简单操作，从 ANSI-C 到现代 C++ 的演变：遍历一个包含 `n` 个元素的数组 `v`，正如 列表 16 所示。

```
#include <cstddef>

int main() {
  const size_t n{ 100 };
  int v[n];

  // ANSI-C
  size_t i;
  for (i=0; i<n; i++) v[i] = 0; ➊
  // C99
  for (size_t i=0; i<n; i++)  v[i] = 0; ➋

  // C++17
  for (auto& x : v) x = 0; ➌
}
```

*列表 16：一个展示多种方式遍历数组的程序*

这个代码片段展示了在 ANSI-C、C99 和 C++ 中声明循环的不同方式。在 ANSI-C ➊ 和 C99 ➋ 示例中，索引变量 `i` 对你要完成的任务没有直接帮助，你要做的是访问 `v` 中的每个元素。C++ 版本 ➌ 使用了 *基于范围* 的 `for` 循环，它遍历 `v` 中的值范围，同时隐藏了迭代如何实现的细节。像 C++ 中许多零开销抽象一样，这种构造让你可以专注于意义而不是语法。基于范围的 `for` 循环可以与许多类型一起使用，甚至可以让它们与用户定义的类型一起工作。

说到用户定义类型，它们允许你直接在代码中表达思想。假设你想设计一个名为`navigate_to`的函数，告诉一个假设的机器人根据 x 和 y 坐标导航到某个位置。请看下面的原型函数：

```
void navigate_to(double x, double y);
```

`x` 和 `y` 是什么？它们的单位是什么？用户必须阅读文档（或可能是源代码）才能弄清楚。比较以下改进后的原型：

```
struct Position{
--snip--
};
void navigate_to(const Position& p);
```

这个函数要清晰得多。关于 `navigate_to` 接受什么参数没有任何模糊之处。只要你有一个有效构造的 `Position`，你就知道该如何调用 `navigate_to`。关于单位、转换等的担忧现在归构造 `Position` 类的人员负责。

你也可以使用 `const` 指针在 C99/C11 中接近这种清晰度，但 C++ 也使返回类型紧凑且富有表现力。假设你想为机器人写一个名为 `get_position` 的附属函数，顾名思义，它获取位置。在 C 中，你有两种选择，如 列表 17 所示。

```
Position* get_position(); ➊
void get_position(Position* p); ➋
```

*列表 17：返回用户定义类型的 C 风格 API*

在第一个选项中，调用者负责清理返回值 ➊，它可能已经进行了动态分配（尽管从代码中无法看出）。调用者负责在某个地方分配一个 `Position` 并将其传递给 `get_position` ➋。这种方式更符合 C 风格，但语言却成了障碍：你只是想获取一个位置对象，却不得不担心是调用者还是被调用函数负责分配和释放内存。C++ 让你通过直接从函数返回用户定义的类型来简洁地完成所有这些操作，正如 列表 18 所示。

```
Position➊ get_position() {
  --snip--
}
void navigate() {
  auto p = get_position(); ➋
  // p is now available for use
  --snip--
}
```

*列表 18：在 C++ 中按值返回用户定义类型*

因为`get_position`返回一个值➊，编译器可以*省略复制*，所以就像是你直接构造了一个自动的`Position`变量➋；没有运行时开销。从功能上讲，你实际上处于类似于 C 风格通过引用传递的示例 17 的情况。

#### *C++ 标准库*

C++标准库（stdlib）是从 C 迁移的重要原因之一。它包含高性能的通用代码，并且保证在符合标准的环境中立即可用。stdlib 的三个主要组件是容器、迭代器和算法。

*容器*是数据结构。它们负责存储对象序列。它们是正确、安全的，并且（通常）至少和你手动实现的效率相当，这意味着写你自己的这些容器版本将需要巨大努力，而且不可能比 stdlib 容器更好。容器被清晰地分为两大类：*顺序容器*和*关联容器*。顺序容器在概念上类似于数组；它们提供对元素序列的访问。关联容器包含键/值对，因此容器中的元素可以通过键查找。

stdlib 的*算法*是用于常见编程任务的通用函数，例如计数、查找、排序和转换。就像容器一样，stdlib 算法质量极高，并且适用范围广泛。用户通常不需要实现自己的版本，使用 stdlib 算法能大大提高程序员的生产力、代码安全性和可读性。

*迭代器*将容器与算法连接起来。对于许多 stdlib 算法应用，您想操作的数据通常存储在容器中。容器暴露迭代器以提供一个统一的接口，算法则消费这些迭代器，避免程序员（包括 stdlib 的实现者）为每种容器类型实现自定义算法。

示例 19 展示了如何使用几行代码对值容器进行排序。

```
#include <vector>
#include <algorithm>
#include <iostream>

int main() {
  std::vector<int> x{ 0, 1, 8, 13, 5, 2, 3 }; ➊
  x[0] = 21; ➋
  x.push_back(1); ➌
  std::sort(x.begin(), x.end()); ➍
  std::cout << "Printing " << x.size() << " Fibonacci numbers.\n"; ➎
  for (auto number : x) {
    std::cout << number << std::endl; ➏
  }
}
```

*示例 19：使用 stdlib 对值容器进行排序*

背后有大量计算在进行，但代码简洁且富有表现力。首先，你初始化了一个 `std::vector` 容器 ➊。*向量（Vector）*是标准库中的动态数组。*初始化括号*（`{0, 1, ...}`）设置了 `x` 中包含的初始值。你可以像访问数组元素一样，通过括号（`[]`）和索引号访问 `vector` 中的元素。你用这种方法将第一个元素设置为 `21` ➋。因为 `vector` 数组是动态大小的，你可以使用 `push_back` 方法向其中添加元素 ➌。`std::sort` 的神奇调用展示了标准库算法的强大功能 ➍。`x.begin()` 和 `x.end()` 方法返回的迭代器被 `std::sort` 用来就地排序 `x`。`sort` 算法通过使用迭代器与 `vector` 解耦。

得益于迭代器，你可以类似地使用标准库中的其他容器。例如，你可以使用 `list`（标准库中的双向链表）而不是使用 `vector`。因为 `list` 也通过 `.begin()` 和 `.end()` 方法暴露了迭代器，你可以像对待 `vector` 迭代器一样对 `list` 迭代器调用 `sort`。

此外，清单 19 使用了输入输出流（iostreams）。*输入输出流*是标准库用于执行缓冲输入输出的机制。你使用输出运算符 (`<<`) 将 `x.size()`（`x` 中元素的数量）、一些字符串字面量和斐波那契数列元素 `number` 流式传输到 `std::cout`，它封装了标准输出流 ➎ ➏。`std::endl` 对象是一个输入输出操控符，它会写入 `\n` 并刷新缓冲区，确保整个流在执行下一条指令之前被写入标准输出。

现在，想象一下你需要跳过多少环节才能用 C 语言写出一个等效的程序，你就会明白为什么标准库（stdlib）是如此有价值的工具。

#### *Lambda 表达式*

*Lambda 表达式*，在某些圈子里也被称为*匿名函数*，是另一种强大的语言特性，它提升了代码的局部性。在某些情况下，你需要将指针传递给函数，以便将指针作为新创建线程的目标，或者对序列中的每个元素执行某种变换。定义一个一次性使用的自由函数通常不方便。这时，Lambda 表达式就派上用场了。Lambda 表达式是一个新的、与调用参数同行定义的自定义函数。考虑下面这个一行代码，它计算 `x` 中偶数的数量：

```
auto n_evens = std::count_if(x.begin(), x.end(),
                             [] (auto number) { return number % 2 == 0; });
```

这个代码片段使用了标准库的 `count_if` 算法来计算 `x` 中偶数的数量。`std::count_if` 的前两个参数与 `std::sort` 相同；它们是定义算法操作范围的迭代器。第三个参数是 lambda 表达式。这个语法可能看起来有点陌生，但基础知识其实非常简单：

```
[capture] (arguments) { body }
```

*捕获*包含了你需要从 lambda 定义的作用域中获取的对象，用于在函数体内进行计算。*参数*定义了 lambda 预期被调用时所接受的参数名称和类型。*函数体*包含了你希望在调用时完成的计算。它可能会返回值，也可能不会。编译器会根据你暗示的类型推导出函数的原型。

在上面的`std::count_if`调用中，lambda 不需要捕获任何变量。它所需的所有信息都作为一个单独的参数`number`传入。因为编译器知道`x`中包含元素的类型，所以你用`auto`声明`number`的类型，编译器会为你推导出类型。lambda 会被调用，并将`x`中的每个元素作为`number`参数传入。在函数体内，当`number`能被`2`整除时，lambda 才返回`true`，因此只有偶数会被计入。

Lambda 在 C 语言中不存在，实际上也不可能重建它们。每次需要一个函数对象时，你必须声明一个单独的函数，而且无法像在其他语言中那样将对象捕获到函数中。

#### *使用模板的通用编程*

*通用编程*是编写一次代码，使其能与不同的类型一起工作，而不必通过复制和粘贴每种你希望支持的类型来多次重复相同的代码。在 C++中，你使用*模板*来生成通用代码。模板是一种特殊的参数，它告诉编译器表示多种可能类型。

你已经使用过模板：stdlib 中的所有容器都使用模板。在大多数情况下，这些容器中对象的类型并不重要。例如，判断容器中元素数量的逻辑或返回其第一个元素的逻辑并不依赖于元素的类型。

假设你想编写一个函数来加和三个相同类型的数字。你希望接受任何可加的类型。在 C++中，这是一个直接的通用编程问题，你可以通过模板直接解决，就像示例 20 所示。

```
template <typename T>
T add(T x, T y, T z) { ➊
  return x + y + z;
}

int main() {
  auto a = add(1, 2, 3);       // a is an int
  auto b = add(1L, 2L, 3L);    // b is a long
  auto c = add(1.F, 2.F, 3.F); // c is a float
}
```

*示例 20：使用模板创建通用的`add`函数*

当你声明`add` ➊时，你不需要知道`T`。你只需要知道所有的参数和返回值都是`T`类型，并且`T`是可加的。当编译器遇到`add`被调用时，它会推导出`T`并为你生成一个定制的函数。这就是一种真正的代码重用！

#### *类不变式与资源管理*

也许 C++带给系统编程的最大创新是*对象生命周期*。这个概念源自 C 语言，在 C 语言中，根据对象在代码中的声明方式，对象具有不同的存储持续时间。

C++ 在此内存管理模型的基础上，提供了构造函数和析构函数。这些特殊函数是属于 *用户定义类型* 的方法。用户定义类型是 C++ 应用程序的基本构建块。可以将它们视为可以包含函数的 `struct` 对象。

对象的构造函数在其存储持续时间开始后立即调用，析构函数在其存储持续时间结束前立即调用。构造函数和析构函数都是没有返回类型的函数，且名称与封闭类相同。要声明析构函数，可以在类名的开头加上 `~`，正如 列表 21 所示。

```
#include <cstdio>

struct Hal {
  Hal() : version{ 9000 } { // Constructor ➊
    printf("I'm completely operational.\n");
  }
  ~Hal() { // Destructor ➋
    printf("Stop, Dave.\n");
  }
  const int version;
};
```

*列表 21：包含构造函数和析构函数的 `Hal` 类*

`Hal` 类中的第一个方法是 *构造函数* ➊。它设置 `Hal` 对象并建立其 *类不变量*。不变量是类的特性，一旦构造完成便不会改变。借助编译器和运行时的帮助，程序员决定类的不变量是什么，并确保代码强制执行这些不变量。在这种情况下，构造函数将不变量 `version` 设置为 `9000`。*析构函数* 是第二个方法 ➋。每当 `Hal` 即将被释放时，它会在控制台上打印 `"Stop, Dave."`（让 `Hal` 唱“Daisy Bell”留给读者作为练习）。

编译器确保对于具有静态、局部和线程局部存储持续时间的对象，构造函数和析构函数会自动调用。对于具有动态存储持续时间的对象，您需要使用关键字 `new` 和 `delete` 来替代 `malloc` 和 `free`，列表 22 做了说明。

```
#include <cstdio>

struct Hal {
--snip--
};

int main() {
  auto hal = new Hal{};  // Memory is allocated, then constructor is called
  delete hal;            // Destructor is called, then memory is deallocated
}
-----------------------------------------------------------------------
I'm completely operational.
Stop, Dave.
```

*列表 22：创建和销毁 `Hal` 对象的程序*

如果（无论出于何种原因）构造函数无法使对象达到良好状态，它通常会抛出一个 *异常*。作为 C 程序员，您可能在使用某些操作系统 API（例如，Windows 结构化异常处理）时处理过异常。当抛出异常时，栈会被展开，直到找到一个异常处理器，程序在此时会恢复。合理使用异常可以清理代码，因为您只需要在合适的地方检查错误条件。C++ 对异常提供了语言级的支持，正如 列表 23 所示。

```
#include <exception>

try {
  // Some code that might throw a std::exception ➊
} catch (const std::exception &e) {
  // Recover the program here. ➋
}
```

*列表 23：`try`-`catch` 块*

您可以将可能抛出异常的代码放在 `try` 语句后面的代码块中 ➊。如果在任何时候抛出异常，栈将展开（优雅地销毁任何超出作用域的对象），并运行您在 `catch` 表达式后面放置的代码 ➋。如果没有抛出异常，则此 `catch` 代码不会执行。

构造函数、析构函数和异常与 C++的另一个核心主题密切相关，那就是将对象的生命周期与它所拥有的资源绑定。这就是资源分配即初始化（RAII）概念（有时也叫做*构造函数获取，析构函数释放*）。考虑列表 24 中的 C++类。

```
#include <system_error>
#include <cstdio>

struct File {
  File(const char* path, bool write) { ➊
    auto file_mode = write ? "w" : "r"; ➋
    file_pointer = fopen(path, file_mode); ➌
    if (!file_pointer) throw std::system_error(errno, std::system_category()); ➍
  }
  ~File() {
    fclose(file_pointer);
  }
  FILE* file_pointer;
};
```

*列表 24：一个`File`类*

`File`的构造函数➊接受两个参数。第一个参数与文件的`path`相对应，第二个参数是一个`bool`，表示文件模式是应该以写模式（`true`）还是读模式（`false`）打开。这个参数的值通过*三元运算符*`?:`设置`file_mode`➋。三元运算符会评估一个布尔表达式，并根据布尔值返回两个值中的一个。例如：

```
x ? val_if_true : val_if_false
```

如果布尔表达式`x`为`true`，则表达式的值为`val_if_true`。如果`x`为`false`，则值为`val_if_false`。

在列表 24 中的`File`构造函数代码片段中，构造函数尝试以读/写访问权限打开位于`path`的文件 ➌。如果出现任何问题，调用将把`file_pointer`设置为`nullptr`，这是 C++中一个类似于 0 的特殊值。当发生这种情况时，你会抛出一个`system_error` ➍。`system_error`只是一个封装了系统错误详细信息的对象。如果`file_pointer`不是`nullptr`，那么它是有效的。这就是该类的不变量。

现在考虑列表 25 中的程序，它使用了`File`。

```
#include <cstdio>
#include <system_error>
#include <cstring>

struct File {
--snip–
};

int main() {
  { ➊
    File file("last_message.txt", true); ➋
    const auto message = "We apologize for the inconvenience.";
    fwrite(message, strlen(message), 1, file.file_pointer);
  } ➌
  // last_message.txt is closed here!
  {
    File file("last_message.txt", false); ➍
    char read_message[37]{};
    fread(read_message, sizeof(read_message), 1, file.file_pointer);
    printf("Read last message: %s\n", read_message);
  }
}
-----------------------------------------------------------------------
We apologize for the inconvenience.
```

*列表 25：一个使用`File`类的程序*

大括号 ➊ ➌ 定义了一个作用域。因为第一个`file`位于这个作用域内，作用域定义了`file`的生命周期。一旦构造函数返回➋，你就知道`file.file_pointer`是有效的，这要归功于类的不变量；根据`File`构造函数的设计，你知道`file.file_pointer`在`File`对象的生命周期内必须是有效的。你使用`fwrite`写入消息。无需显式调用`fclose`，因为`file`过期，析构函数会为你清理`file.file_pointer`➌。你再次打开`File`，但这次是为了读访问 ➍。只要构造函数返回，你就知道*last_message.txt*已成功打开，并继续读取到`read_message`中。打印完消息后，`file`的析构函数被调用，`file.file_pointer`再次被清理。

有时你需要动态内存分配的灵活性，但仍希望依赖 C++的对象生命周期，以确保不会泄漏内存或不小心出现“使用已释放内存”的问题。这正是*智能指针*的作用，它通过所有权模型管理动态对象的生命周期。一旦没有智能指针拥有某个动态对象，该对象会被销毁。

其中一个智能指针是`unique_ptr`，它模拟了独占所有权。列表 26 展示了它的基本用法。

```
#include <memory>

struct Foundation{
  const char* founder;
};

int main() {
  std::unique_ptr<Foundation> second_foundation{ new Foundation{} }; ➊
  // Access founder member variable just like a pointer:
  second_foundation->founder = "Wanda";
} ➋
```

*示例 26：使用`unique_ptr`的程序*

你动态分配了一个`Foundation`，并使用大括号初始化语法将得到的`Foundation*`指针传递给`second_foundation`的构造函数 ➊。`second_foundation`的类型是`unique_ptr`，它只是一个 RAII 对象，包装了动态`Foundation`。当`second_foundation`被销毁时 ➋，动态`Foundation`会被适当地销毁。

智能指针与普通的*裸*指针不同，因为裸指针只是一个内存地址。你必须手动管理与地址相关的所有内存管理工作。另一方面，智能指针处理了所有这些繁琐的细节。通过将动态对象包装在智能指针中，你可以放心，当对象不再需要时，内存会被适当地清理。编译器知道对象不再需要，因为当智能指针超出作用域时，它的析构函数会被调用。

#### *移动语义*

有时，你想要转移一个对象的所有权；这在很多情况下都会遇到，例如使用`unique_ptr`时。你不能复制一个`unique_ptr`，因为一旦其中一个`unique_ptr`的副本被销毁，剩下的`unique_ptr`会持有对已删除对象的引用。与其复制对象，你可以利用 C++的`move`语义将所有权从一个`unique_ptr`转移到另一个，如示例 27 所示。

```
#include <memory>

struct Foundation{
  const char* founder;
};

struct Mutant {
  // Constructor sets foundation appropriately:
  Mutant(std::unique_ptr<Foundation> foundation)
    : foundation(std::move(foundation)) {}
  std::unique_ptr<Foundation> foundation;
};

int main() {
  std::unique_ptr<Foundation> second_foundation{ new Foundation{} }; ➊
  // ... use second_foundation
  Mutant the_mule{ std::move(second_foundation) }; ➋
  // second_foundation is in a 'moved-from' state
  // the_mule owns the Foundation
}
```

*示例 27：移动`unique_ptr`的程序*

和之前一样，你创建了`unique_ptr<Foundation>` ➊。你使用它一段时间后，决定将所有权转移给`Mutant`对象。`move`函数告诉编译器你想进行转移。在构造`the_mule` ➋后，`Foundation`的生命周期通过它的成员变量与`the_mule`的生命周期关联。

### 放松并享受你的鞋子

C++是*最*优秀的系统编程语言。你在 C 语言中的大部分知识可以直接迁移到 C++中，但你也将学习到许多新概念。你可以通过使用 Super C 逐步将 C++融入到你的 C 程序中。当你掌握 C++的一些深层主题后，你会发现写现代 C++相比 C 带来了许多显著的优势。你将能够用简洁的代码表达思想，利用强大的标准库在更高的抽象层次上工作，使用模板来提高运行时性能和代码重用，并依赖 C++的对象生命周期来管理资源。

我相信你在学习 C++时所做的投资将带来巨大的回报。读完这本书后，我想你会同意这个观点。
