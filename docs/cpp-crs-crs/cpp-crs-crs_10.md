## 第十章：STATEMENTS**

*进步不是来自早起的人——进步是由懒人寻找更简单的方法来做事情所带来的。*

—Robert A. Heinlein，《爱情需要足够的时间》

![图片](img/common.jpg)

每个 C++ 函数由一系列 *语句* 组成，语句是指定执行顺序的编程构造。本章通过理解对象生命周期、模板和表达式，来探索语句的细微差别。

### 表达式语句

*表达式语句* 是一个表达式后跟一个分号（`;`）。表达式语句构成了程序中的大多数语句。您可以将任何表达式转换为语句，这应该在您需要评估一个表达式但又想丢弃其结果时进行。当然，这只有在评估该表达式会引起副作用时才有用，比如打印到控制台或修改程序的状态。

清单 8-1 包含了几个表达式语句。

```
#include <cstdio>

int main() {
  int x{};
  ++x; ➊
  42; ➋
  printf("The %d True Morty\n", x); ➌
}
--------------------------------------------------------------------------
The 1 True Morty ➌
```

*清单 8-1：包含几个表达式语句的简单程序*

➊ 处的表达式语句有副作用（递增 `x`），但 ➋ 处的没有。两者都是有效的（尽管 ➋ 处的没有用）。对 `printf` ➌ 的函数调用也是一个表达式语句。

### 复合语句

*复合语句*，也叫 *块*，是一系列由花括号 `{ }` 包围的语句。块在控制结构中很有用，例如 `if` 语句，因为您可能希望执行多个语句而不仅仅是一个。

每个块声明了一个新的作用域，称为 *块作用域*。正如您在 第四章 中所学，声明在块作用域内的具有自动存储期限的对象，其生命周期与块的生命周期绑定。块内声明的变量按照其声明的反向顺序销毁。

清单 8-2 使用了来自 清单 4-5（位于 第 97 页）的可靠 `Tracer` 类来探索块作用域。

```
#include <cstdio>

struct Tracer {
  Tracer(const char* name) : name{ name } {
    printf("%s constructed.\n", name);
  }
  ~Tracer() {
    printf("%s destructed.\n", name);
  }
private:
  const char* const name;
};

int main() {
  Tracer main{ "main" }; ➊
  {
    printf("Block a\n"); ➋
    Tracer a1{ "a1" }; ➌
    Tracer a2{ "a2" }; ➍
 }
  {
    printf("Block b\n"); ➎
    Tracer b1{ "b1" }; ➏
    Tracer b2{ "b2" }; ➐
  }
}
--------------------------------------------------------------------------
main constructed. ➊
Block a ➋
a1 constructed. ➌
a2 constructed.➍
a2 destructed.
a1 destructed.
Block b ➎
b1 constructed. ➏
b2 constructed. ➐
b2 destructed.
b1 destructed.
main destructed.
```

*清单 8-2：一个使用 `Tracer` 类探索复合语句的程序*

清单 8-2 首先初始化了一个名为 `main` 的 `Tracer` ➊。接着，您会生成两个复合语句。第一个复合语句以左花括号 `{` 开始，后跟该块的第一条语句，该语句打印 `Block a` ➋。您创建了两个 `Tracer`，`a1` ➌ 和 `a2` ➍，然后用右花括号 `}` 结束该块。这两个 tracers 在执行通过 `Block a` 后被销毁。请注意，这两个 tracers 的销毁顺序与它们的初始化顺序相反：先是 `a2`，然后是 `a1`。

还请注意，紧接着 `Block a` 后面是另一个复合语句，您打印了 `Block b` ➎，然后构造了两个 tracers，`b1` ➏ 和 `b2` ➐。它的行为是相同的：先是 `b2` 销毁，然后是 `b1`。一旦执行通过 `Block b`，`main` 的作用域结束，`Tracer main` 最终销毁。

### 声明语句

*声明语句*（或简称*声明*）在程序中引入标识符，例如函数、模板和命名空间。本节将探讨这些熟悉的声明的一些新特性，以及类型别名、属性和结构绑定。

**注意**

*表达式`static_assert`，你在第六章中学过，也是一个声明语句。*

#### *函数*

*函数声明*，也叫做函数的*签名*或*原型*，指定了函数的输入和输出。声明不需要包括参数名，只需要包括它们的类型。例如，下面这行代码声明了一个名为`randomize`的函数，该函数接受一个`uint32_t`的引用并返回`void`：

```
void randomize(uint32_t&);
```

不是成员函数的函数被称为*非成员函数*，有时也叫*自由函数*，它们总是声明在`main()`外部，在命名空间范围内。*函数定义*包括函数声明以及函数的主体。函数的声明定义了函数的接口，而函数的定义则定义了它的实现。例如，下面的定义是`randomize`函数的一种可能实现：

```
void randomize(uint32_t& x) {
  x = 0x3FFFFFFF & (0x41C64E6D * x + 12345) % 0x80000000;
}
```

**注意**

*这个`randomize`实现是一个线性同余生成器，一种原始类型的随机数生成器。有关生成随机数的更多信息，请参见第 241 页的“进一步阅读”部分。*

正如你可能已经注意到的，函数声明是可选的。那么它们为什么存在呢？

答案是，你可以在代码中使用已声明的函数，只要它们最终在某个地方被定义。你的编译工具链可以自动处理这个问题。（你将在第二十一章中了解它是如何工作的。）

清单 8-3 中的程序确定了随机数生成器从数字 0x4c4347 转换到数字 0x474343 需要多少次迭代。

```
#include <cstdio>
#include <cstdint>

void randomize(uint32_t&); ➊

int main() {
  size_t iterations{}; ➋
  uint32_t number{ 0x4c4347 }; ➌
  while (number != 0x474343) { ➍
    randomize(number); ➎
    ++iterations; ➏
  }
  printf("%zu", iterations); ➐
}

void randomize(uint32_t& x) {
  x = 0x3FFFFFFF & (0x41C64E6D * x + 12345) % 0x80000000; ➑
}
--------------------------------------------------------------------------
927393188 ➐
```

*清单 8-3：一个在`main`中使用函数但直到稍后才定义的程序*

首先，你声明`randomize` ➊。在`main`中，你将`iterations`计数变量初始化为零 ➋，并将`number`变量初始化为 0x4c4347 ➌。一个`while`循环检查`number`是否等于目标值 0x4c4347 ➍。如果不相等，你调用`randomize` ➎并递增`iterations` ➏。注意，你还没有定义`randomize`。一旦`number`等于目标值，你会在从`main`返回之前打印`iterations`的值 ➐。最后，你定义`randomize` ➑。程序的输出显示，要随机抽取目标值，几乎需要十亿次迭代。

尝试删除`randomize`的定义并重新编译。你应该会得到一个错误，提示无法找到`randomize`的定义。

你也可以像处理非成员函数一样，将方法声明与定义分开。例如，以下`RandomNumberGenerator`类将`ran``domize`函数替换为`next`：

```
struct RandomNumberGenerator {
  explicit RandomNumberGenerator(uint32_t seed) ➊
    : number{ seed } {} ➋
  uint32_t next(); ➌
private:
  uint32_t number;
};
```

你可以构建一个带有`seed`值➊的`RandomNumberGenerator`，它用这个值来初始化`number`成员变量➋。你已按照与非成员函数相同的规则声明了`next`函数➌。为了提供`next`的定义，你必须使用作用域解析符和类名来指定你要定义的方法。否则，定义一个方法与定义一个非成员函数是一样的：

```
uint32_t➊ RandomNumberGenerator::➋next() {
  number = 0x3FFFFFFF & (0x41C64E6D * number + 12345) % 0x80000000; ➌
  return number; ➍
}
```

这个定义与声明➊共享相同的返回类型。`RandomNumberGenerator::`构造指定你正在定义一个方法➋。函数的细节基本相同➌，只是你返回的是随机数生成器的状态的副本，而不是写入参数引用➋。

示例 8-4 演示了如何重构示例 8-3 以包含`RandomNumberGenerator`。

```
#include <cstdio>
#include <cstdint>

struct RandomNumberGenerator {
  explicit RandomNumberGenerator(uint32_t seed)
    : iterations{}➊, number { seed }➋ {}
  uint32_t next(); ➌
  size_t get_iterations() const; ➍
private:
  size_t iterations;
  uint32_t number;
};

int main() {
  RandomNumberGenerator rng{ 0x4c4347 }; ➎
  while (rng.next() != 0x474343) { ➏
    // Do nothing...
  }
  printf("%zu", rng.get_iterations()); ➐
}

uint32_t RandomNumberGenerator::next() { ➑
  ++iterations;
  number = 0x3FFFFFFF & (0x41C64E6D * number + 12345) % 0x80000000;
  return number;
}

size_t RandomNumberGenerator::get_iterations() const { ➒
  return iterations;
}
--------------------------------------------------------------------------
927393188 ➐
```

*示例 8-4：使用`RandomNumberGenerator`类重构示例 8-3*

如示例 8-3 所示，你已将声明与定义分开。声明了一个将`iterations`成员初始化为零➊并将其`number`成员设置为`seed`➋的构造函数后，`next`➌和`get_iterations`➍方法的声明没有包含实现。在`main`函数中，你使用`0x4c4347`的种子值➎初始化`RandomNumberGenerator`类，并调用`next`方法提取新的随机数➏。结果是一样的➐。与之前一样，`next`和`get_iterations`的定义位于`main`函数中的调用之后➑➒。

**注意**

*分离定义和声明的实用性可能不太明显，因为你迄今为止处理的都是单一源文件程序。第二十一章探讨了多个源文件程序，其中分离声明和定义带来了巨大的好处。*

#### *命名空间*

命名空间可以防止命名冲突。在大型项目中或导入库时，命名空间对于消除歧义、精确定位你要查找的符号至关重要。

##### 将符号放入命名空间中

默认情况下，你声明的所有符号都会进入*全局命名空间*。全局命名空间包含所有你可以在不添加命名空间限定符的情况下访问的符号。除了`std`命名空间中的几个类，你所使用的对象都仅存在于全局命名空间中。

要将符号放入除全局命名空间外的命名空间中，你需要在*命名空间块*中声明该符号。命名空间块的形式如下：

```
namespace BroopKidron13 {
  // All symbols declared within this block
  // belong to the BroopKidron13 namespace
}
```

命名空间可以通过两种方式进行嵌套。首先，你可以简单地嵌套命名空间块：

```
namespace BroopKidron13 {
  namespace Shaltanac {
    // All symbols declared within this block
    // belong to the BroopKidron13::Shaltanac namespace
  }
}
```

其次，你可以使用作用域解析符：

```
namespace BroopKidron13::Shaltanac {
  // All symbols declared within this block
  // belong to the BroopKidron13::Shaltanac namespace
}
```

后者的方法更加简洁。

##### 在命名空间中使用符号

要使用命名空间中的符号，您始终可以使用作用域解析运算符来指定符号的完全限定名称。这可以帮助您避免在大型项目中或使用第三方库时的命名冲突。如果您和另一个程序员使用相同的符号，您可以通过将该符号放入命名空间中来避免歧义。

列表 8-5 展示了如何使用完全限定的符号名称来访问命名空间中的符号。

```
#include <cstdio>

namespace BroopKidron13::Shaltanac { ➊
  enum class Color { ➋
    Mauve,
    Pink,
    Russet
 };
}

int main() {
  const auto shaltanac_grass{ BroopKidron13::Shaltanac::Color::Russet➌ };
  if(shaltanac_grass == BroopKidron13::Shaltanac::Color::Russet) {
    printf("The other Shaltanac's joopleberry shrub is always "
           "a more mauvey shade of pinky russet.");
  }
}
--------------------------------------------------------------------------
The other Shaltanac's joopleberry shrub is always a more mauvey shade of pinky russet.
```

*列表 8-5：使用作用域解析运算符的嵌套命名空间块*

列表 8-5 使用了嵌套命名空间 ➊ 并声明了一个 `Color` 类型 ➋。要使用 `Color`，您需要使用作用域解析运算符来指定符号的全名 `BroopKidron13::Shaltanac::Color`。因为 `Color` 是一个 `enum class`，所以您需要使用作用域解析运算符来访问它的值，正如您将 `shaltanac_grass` 赋值给 `Russet` ➌ 时一样。

##### 使用指令

您可以使用 `using` *指令* 来避免大量输入。`using` 指令将符号导入到一个块中，或者如果您在命名空间作用域声明 `using` 指令，则导入到当前命名空间。无论哪种方式，您只需输入一次完全的命名空间路径。其使用模式如下：

```
using my-type;
```

相应的 my-type 被导入到当前命名空间或块中，这意味着您不再需要使用其全名。列表 8-6 通过使用指令重构了列表 8-5。

```
#include <cstdio>

namespace BroopKidron13::Shaltanac {
  enum class Color {
    Mauve,
    Pink,
    Russet
  };
}

int main() {
  using BroopKidron13::Shaltanac::Color; ➊
  const auto shaltanac_grass = Color::Russet➋;
  if(shaltanac_grass == Color::Russet➌) {
    printf("The other Shaltanac's joopleberry shrub is always "
           "a more mauvey shade of pinky russet.");
  }
}
--------------------------------------------------------------------------
The other Shaltanac's joopleberry shrub is always a more mauvey shade of pinky russet.
```

*列表 8-6：使用指令重构列表 8-5*

通过 `main` 中的 `using` 指令 ➊，您不再需要输入命名空间 `BroopKidron13::Shaltanac` 来使用 `Color` ➋➌。

如果小心使用，您可以通过 `using namespace` 指令将给定命名空间中的所有符号导入到全局命名空间中。

列表 8-7 详细说明了列表 8-6：命名空间 `BroopKidron13::Shaltanac` 包含多个符号，您希望将它们导入到全局命名空间中，以避免大量输入。

```
#include <cstdio>

namespace BroopKidron13::Shaltanac {
  enum class Color {
    Mauve,
    Pink,
    Russet
  };

  struct JoopleberryShrub {
    const char* name;
    Color shade;
  };

  bool is_more_mauvey(const JoopleberryShrub& shrub) {
    return shrub.shade == Color::Mauve;
  }
}

using namespace BroopKidron13::Shaltanac; ➊
int main() {
  const JoopleberryShrub➋ yours{
    "The other Shaltanac",
    Color::Mauve➌
  };

  if (is_more_mauvey(yours)➍) {
    printf("%s's joopleberry shrub is always a more mauvey shade of pinky"
           "russet.", yours.name);
  }
}
--------------------------------------------------------------------------
The other Shaltanac's joopleberry shrub is always a more mauvey shade of pinky
russet.
```

*列表 8-7：重构后的列表 8-6，多个符号导入到全局命名空间中*

通过 `using namespace` 指令 ➊，您可以在程序中使用类 ➋、枚举类 ➌、函数 ➍ 等，而无需输入完全限定的名称。当然，您需要非常小心，避免覆盖全局命名空间中的现有类型。通常，在单个翻译单元中出现过多的 `using namespace` 指令是不好的做法。

**注意**

*您绝不应该在头文件中放置 `using namespace` 指令。每个包含您的头文件的源文件都会将该 `using` 指令中的所有符号转存到全局命名空间中。这可能会导致非常难以调试的问题。*

#### *类型别名*

一个 *类型别名* 定义了一个名称，指向一个先前定义的名称。你可以将类型别名作为现有类型名称的同义词使用。

类型和所有引用它的类型别名之间没有区别。此外，类型别名不能改变现有类型名称的含义。

要声明一个类型别名，你可以使用以下格式，其中 type-alias 是类型别名的名称，type-id 是目标类型：

```
using type-alias = type-id;
```

列表 8-8 使用了两个类型别名，`String` 和 `ShaltanacColor`。

```
#include <cstdio>

namespace BroopKidron13::Shaltanac {
  enum class Color {
    Mauve,
    Pink,
    Russet
  };
}

using String = const char[260]; ➊
using ShaltanacColor = BroopKidron13::Shaltanac::Color; ➋

int main() {
  const auto my_color{ ShaltanacColor::Russet }; ➌
  String saying { ➍
    "The other Shaltanac's joopleberry shrub is "
    "always a more mauvey shade of pinky russet."
  };
  if (my_color == ShaltanacColor::Russet) {
    printf("%s", saying);
  }
}
```

*列表 8-8：对 列表 8-7 的重构，使用了类型别名*

列表 8-8 声明了一个类型别名 `String`，它指向 `const char[260]` ➊。该列表还声明了一个 `ShaltanacColor` 类型别名，指向 `BroopKidron13::Shaltanac::Color` ➋。你可以使用这些类型别名作为直接替代，简化代码。在 `main` 中，你使用 `ShaltanacColor` 来去除所有嵌套的命名空间 ➌，并使用 `String` 使 `saying` 的声明更加简洁 ➍。

**注意**

*类型别名可以出现在任何作用域中——块作用域、类作用域或命名空间作用域。*

你可以将模板参数引入类型别名中。这使得有两种重要的用途：

+   你可以对模板参数进行部分应用。*部分应用*是将一些参数固定到模板中，生成一个具有更少模板参数的新模板的过程。

+   你可以为一个模板定义一个类型别名，使用完全指定的模板参数集。

模板实例化可能会非常冗长，而类型别名可以帮助你避免腕管综合症。

列表 8-9 声明了一个具有两个模板参数的 `NarrowCaster` 类。然后，你使用类型别名部分应用其中一个参数，生成一个新类型。

```
#include <cstdio>
#include <stdexcept>

template <typename To, typename From>
struct NarrowCaster const { ➊
  To cast(From value) {
    const auto converted = static_cast<To>(value);
    const auto backwards = static_cast<From>(converted);
    if (value != backwards) throw std::runtime_error{ "Narrowed!" };
    return converted;
  }
};

template <typename From>
using short_caster = NarrowCaster<short, From>; ➋

int main() {
  try {
    const short_caster<int> caster; ➌
    const auto cyclic_short = caster.cast(142857);
    printf("cyclic_short: %d\n", cyclic_short);
  } catch (const std::runtime_error& e) {
    printf("Exception: %s\n", e.what()); ➍
  }
}
--------------------------------------------------------------------------
Exception: Narrowed! ➍
```

*列表 8-9：使用类型别名对 `NarrowCaster` 类进行部分应用*

首先，你实现了一个 `NarrowCaster` 模板类，它具有与 列表 6-6 中的 `narrow_cast` 函数模板相同的功能（在 第 154 页）：它会执行 `static_cast`，然后检查是否发生了缩窄 ➊。接着，你声明了一个类型别名 `short_caster`，将 `short` 部分应用为 `To` 类型到 `NarrowCast` 中。在 `main` 中，你声明了一个类型为 `short_caster<int>` 的 `caster` 对象 ➌。`short_caster` 类型别名中的单个模板参数应用于类型别名中的剩余类型参数——`From` ➋。换句话说，类型 `short_cast<int>` 与 `NarrowCaster<short, int>` 同义。最终结果是相同的：使用 2 字节的 `short` 类型，当你尝试将值为 142857 的 `int` 转换为 `short` 时，会出现缩窄异常 ➍。

#### *结构化绑定*

*结构化绑定*使你能够将对象解包成它们的组成部分。任何其非静态数据成员是公共的类型都可以通过这种方式解包——例如，在第二章中介绍的 POD（普通数据类）类型。*结构化绑定语法*如下：

```
auto [object-1, object-2, ...] = plain-old-data;
```

这一行将通过逐个剥离 POD 对象初始化任意数量的对象（object-1、object-2，依此类推）。这些对象从上到下剥离 POD，并从左到右填充结构化绑定。考虑一个`read_text_file`函数，它接受一个字符串参数，该参数对应文件路径。比如，如果文件被锁定或不存在，函数可能会失败。你有两种处理错误的选项：

+   你可以在`read_text_file`中抛出异常。

+   你可以从函数返回一个成功的状态码。

让我们来探索第二种选择。

示例 8-10 中的 POD 类型将作为`read_text_file`函数的返回类型。

```
struct TextFile {
  bool success; ➊
  const char* contents; ➋
  size_t n_bytes; ➌
};
```

*示例 8-10：一个`TextFile`类型，它将由`read_text_file`函数返回*

首先，一个标志会告诉调用者函数调用是否成功 ➊。接下来是`file`的内容 ➋及其大小`n_bytes` ➌。

`read_text_file`的原型如下所示：

```
TextFile read_text_file(const char* path);
```

你可以使用结构化绑定声明将`TextFile`解包成程序中的各个部分，正如在示例 8-11 中所示。

```
#include <cstdio>

struct TextFile { ➊
  bool success;
  const char* data;
  size_t n_bytes;
};

TextFile read_text_file(const char* path) { ➋
  const static char contents[]{ "Sometimes the goat is you." };
  return TextFile{
    true,
    contents,
    sizeof(contents)
  };
}

int main() {
  const auto [success, contents, length]➌ = read_text_file("REAMDE.txt"); ➍
  if (success➎) {
    printf("Read %zu bytes: %s\n", length➏, contents➐);
  } else {
    printf("Failed to open REAMDE.txt.");
  }
}
--------------------------------------------------------------------------
Read 27 bytes: Sometimes the goat is you.
```

*示例 8-11：一个模拟读取文本文件的程序，它返回一个 POD，你可以在结构化绑定中使用它*

你声明了`TextFile` ➊，然后为`read_text_file`提供了一个虚拟定义 ➋。（它实际上并不读取文件；更多内容将在第二部分中讨论。）

在`main`函数内，你调用`read_text_file` ➍并使用结构化绑定声明将结果解包到三个不同的变量中：`success`、`contents`和`length` ➌。在结构化绑定之后，你可以像声明这些变量时一样使用它们 ➎➏➐。

**注意**

*结构化绑定声明中的类型不必匹配。*

#### *属性*

属性将实现定义的特性应用于表达式语句。你通过使用包含一个或多个以逗号分隔的属性元素的双括号`[[ ]]`来引入属性。

表 8-1 列出了标准属性。

**表 8-1：** 标准属性

| **属性** | **含义** |
| --- | --- |
| `[[noreturn]]` | 表示一个函数没有返回值。 |
| `[[deprecated("`reason`")]]` | 表示该表达式已弃用；即不推荐使用它。`"reason"`是可选的，表示弃用的原因。 |
| `[[fallthrough]]` | 表示一个 switch 语句的 case 打算穿透到下一个 switch 语句的 case。这可以避免编译器检查 switch case 穿透错误，因为这种情况不常见。 |
| `[[nodiscard]]` | 表示应使用以下函数或类型声明。如果使用该元素的代码丢弃了值，编译器应发出警告。 |
| `[[maybe_unused]]` | 表示以下元素可能未被使用，编译器不应对此发出警告。 |
| `[[carries_dependency]]` | 在 `<atomic>` 头文件中使用，帮助编译器优化某些内存操作。你不太可能直接遇到这个。 |

列表 8-12 演示了通过定义一个永不返回的函数来使用 `[[noreturn]]` 属性。

```
#include <cstdio>
#include <stdexcept>

[[noreturn]] void pitcher() { ➊
  throw std::runtime_error{ "Knuckleball." }; ➋
}

int main() {
  try {
    pitcher(); ➌
  } catch(const std::exception& e) {
    printf("exception: %s\n", e.what()); ➍
  }
}
--------------------------------------------------------------------------
Exception: Knuckleball. ➍
```

*列表 8-12：演示使用 `[[noreturn]]` 属性的程序*

首先，你使用 `[[noreturn]]` 属性声明 `pitcher` 函数 ➊。在该函数中，你抛出一个异常 ➋。因为你总是抛出异常，所以 `pitcher` 永远不会返回（因此使用 `[[noreturn]]` 属性）。在 `main` 中，你调用 `pitcher` ➌ 并处理捕获的异常 ➍。当然，这段代码即使没有 `[[noreturn]]` 属性也能正常工作，但向编译器提供这些信息可以让它更全面地推理你的代码（并有可能优化你的程序）。

使用属性的情况较少，但它们仍然能向编译器传达有用的信息。

### 选择语句

*选择语句* 表示条件控制流。选择语句有两种类型，分别是 `if` 语句和 `switch` 语句。

#### *if 语句*

`if` 语句具有在 列表 8-13 中显示的熟悉形式。

```
if (condition-1) {
  // Execute only if condition-1 is true ➊
} else if (condition-2) { // optional
  // Execute only if condition-2 is true ➋
}
// ... as many else ifs as desired
--snip--
} else { // optional
  // Execute only if none of the conditionals is true ➌
}
```

*列表 8-13：`if` 语句的语法*

遇到 `if` 语句时，首先评估条件 1 表达式。如果它为 `true`，则执行 ➊ 处的代码块，`if` 语句停止执行（不会考虑任何 `else if` 或 `else` 语句）。如果为 `false`，则按顺序评估 `else if` 语句的条件。这些是可选的，你可以根据需要提供任意数量。

例如，如果条件 2 评估为 `true`，则会执行 ➋ 处的代码块，剩余的 `else if` 或 `else` 语句不会被考虑。最后，如果所有前面的条件都评估为 `false`，则执行 ➌ 处的 `else` 块。与 `else if` 块一样，`else` 块是可选的。

列表 8-14 中的函数模板将 `else` 参数转换为 `Positive`、`Negative` 或 `Zero`。

```
#include <cstdio>

template<typename T>
constexpr const char* sign(const T& x) {
  const char* result{};
  if (x == 0) { ➊
    result = "zero";
  } else if (x > 0) { ➋
    result = "positive";
  } else { ➌
    result = "negative";
  }
  return result;
}

int main() {
  printf("float 100 is %s\n", sign(100.0f));
  printf("int  -200 is %s\n", sign(-200));
 printf("char    0 is %s\n", sign(char{}));
}
--------------------------------------------------------------------------
float 100 is positive
int  -200 is negative
char    0 is zero
```

*列表 8-14：`if` 语句的示例用法*

`sign` 函数接受一个参数，并确定该参数是等于 0 ➊、大于 0 ➋，还是小于 0 ➌。根据匹配的条件，它将自动变量 `result` 设置为三种字符串之一——`zero`、`positive` 或 `negative`，并将此值返回给调用者。

##### 初始化语句与 if

你可以通过向 `if` 和 `else if` 声明中添加一个 `init-state` 语句来绑定对象的作用域，如 列表 8-15 所示。

```
if (init-statement; condition-1) {
  // Execute only if condition-1 is true
} else if (init-statement; condition-2) { // optional
  // Execute only if condition-2 is true
}
--snip--
```

*列表 8-15：带有初始化的`if`语句*

你可以将此模式与结构化绑定一起使用，以实现优雅的错误处理。列表 8-16 通过使用初始化语句将`TextFile`限定在`if`语句中，重构了列表 8-11。

```
#include <cstdio>

struct TextFile {
  bool success;
  const char* data;
  size_t n_bytes;
};

TextFile read_text_file(const char* path) {
  --snip--
}

int main() {
  if(const auto [success, txt, len]➊ = read_text_file("REAMDE.txt"); success➋)
  {
    printf("Read %d bytes: %s\n", len, txt); ➌
  } else {
    printf("Failed to open REAMDE.txt."); ➍
  }
}
--------------------------------------------------------------------------
Read 27 bytes: Sometimes the goat is you. ➌
```

*列表 8-16：使用结构化绑定和`if`语句处理错误的列表 8-11 的扩展*

你将结构化绑定声明移到了`if`语句的初始化语句部分 ➊。这样每个解包的对象——`success`、`txt`和`len`——的作用域就限制在了`if`块中。你直接在`if`的条件表达式中使用`success`来判断`read_text_file`是否成功 ➋。如果成功，你会打印`REAMDE.txt`的内容 ➌；如果失败，则打印错误信息 ➍。

##### constexpr if 语句

你可以使`if`语句成为`constexpr`语句；这样的语句称为`constexpr if`语句。`constexpr if`语句在编译时被求值。对应于`true`条件的代码块会被执行，而其余部分会被忽略。

`constexpr if`的使用方式与常规的`if`语句相同，正如列表 8-17 所示。

```
if constexpr (condition-1) {
  // Compile only if condition-1 is true
} else if constexpr (condition-2) { // optional; can be multiple else ifs
  // Compile only if condition-2 is true
}
--snip--
} else { // optional
  // Compile only if none of the conditionals is true
}
```

*列表 8-17：`constexpr if` 语句的使用*

与模板和`<type_traits>`头文件结合使用时，`constexpr` `if`语句非常强大。`constexpr if`的一个主要用途是根据类型参数的一些特性，在函数模板中提供自定义行为。

列表 8-18 中的函数模板`value_of`接受指针、引用和值。根据传入参数的对象类型，`value_of`返回指向的值或值本身。

```
#include <cstdio>
#include <stdexcept>
#include <type_traits>

template <typename T>
auto value_of(T x➊) {
  if constexpr (std::is_pointer<T>::value) { ➋
    if (!x) throw std::runtime_error{ "Null pointer dereference." }; ➌
    return *x; ➍
 } else {
    return x; ➎
  }
}

int main() {
  unsigned long level{ 8998 };
  auto level_ptr = &level;
  auto &level_ref = level;
  printf("Power level = %lu\n", value_of(level_ptr)); ➏
  ++*level_ptr;
  printf("Power level = %lu\n", value_of(level_ref)); ➐
  ++level_ref;
  printf("It's over %lu!\n", value_of(level++)); ➑
  try {
    level_ptr = nullptr;
    value_of(level_ptr);
  } catch(const std::exception& e) {
    printf("Exception: %s\n", e.what()); ➒
  }
}
--------------------------------------------------------------------------
Power level = 8998 ➏
Power level = 8999 ➐
It's over 9000! ➑
Exception: Null pointer dereference. ➒
```

*列表 8-18：一个使用`constexpr if`语句的示例函数模板`value_of`*

`value_of`函数模板接受一个参数`x` ➊。你使用`std::is_pointer<T>`类型特征来判断参数是否为指针类型，并作为`constexpr if`语句中的条件表达式 ➋。如果`x`是指针类型，你检查是否为`nullptr`，如果遇到`nullptr`则抛出异常 ➌。如果`x`不是`nullptr`，你解引用它并返回结果 ➍。否则，`x`不是指针类型，因此直接返回它（因为它是一个值） ➎。

在`main`函数中，你多次实例化`value_of`，分别使用`unsigned long`指针 ➏、`unsigned long`引用 ➐、`unsigned long` ➑和`nullptr` ➒。

在运行时，`constexpr if` 语句消失；每个 `value_of` 的实例化包含一个分支语句或另一个分支。你可能会想知道为什么这样的功能有用。毕竟，程序应该在运行时做有用的事情，而不是在编译时。只要回到示例 7-17（见第 206 页），你会发现编译时求值通过消除魔法值，能显著简化你的程序。

还有其他一些例子，其中编译时求值非常流行，特别是在为他人创建库时。因为库的编写者通常无法知道用户将如何使用他们的库，他们需要编写通用代码。通常，他们会使用你在第六章中学到的技巧，这样他们就可以实现编译时多态。像 `constexpr` 这样的构造可以在编写此类代码时提供帮助。

**注意**

*如果你有 C 语言背景，你会立刻意识到编译时求值的实用性，因为它几乎完全取代了预处理器宏的需求。*

#### *switch 语句*

第二章 首次介绍了著名的 `switch` 语句。本节深入探讨了将初始化语句添加到 `switch` 声明中的方法。用法如下：

```
switch (init-expression➊; condition) {
  case (case-a): {
    // Handle case-a here
  } break;
  case (case-b): {
    // Handle case-b here
  } break;
    // Handle other conditions as desired
  default: {
    // Handle the default case here
  }
}
```

与 `if` 语句一样，你可以在 `switch` 语句中进行实例化 ➊。

示例 8-19 在 `switch` 语句中使用了初始化语句。

```
#include <cstdio>

enum class Color { ➊
  Mauve,
  Pink,
  Russet
};

struct Result { ➋
  const char* name;
  Color color;
};

Result observe_shrub(const char* name) { ➌
  return Result{ name, Color::Russet };
}

int main() {
  const char* description;
  switch (const auto result➍ = observe_shrub("Zaphod"); result.color➎) {
  case Color::Mauve: {
    description = "mauvey shade of pinky russet";
    break;
  } case Color::Pink: {
    description = "pinky shade of mauvey russet";
    break;
  } case Color::Russet: {
 description = "russety shade of pinky mauve";
    break;
  } default: {
    description = "enigmatic shade of whitish black";
  }}
  printf("The other Shaltanac's joopleberry shrub is "
         "always a more %s.", description); ➏
}
--------------------------------------------------------------------------
The other Shaltanac's joopleberry shrub is always a more russety shade of
pinky mauve. ➏
```

*示例 8-19：在 `switch` 语句中使用初始化表达式*

你声明了熟悉的 `Color` `enum class` ➊，并将其与 `char*` 成员连接，形成了 POD 类型 `Result` ➋。函数 `observe_shrub` 返回一个 `Result` ➌。在 `main` 中，你在初始化表达式中调用 `observe_shrub` 并将结果存储在 `result` 变量 ➍ 中。在 `switch` 的条件表达式中，你提取了此 `result` 的 `color` 元素 ➎。该元素决定了执行的 case（并设置 `description` 指针） ➏。

与 `if` 语句加初始化器语法一样，在初始化表达式中初始化的任何对象都绑定到 `switch` 语句的作用域内。

### 迭代语句

*迭代语句* 会重复执行一个语句。四种迭代语句分别是 `while` 循环、`do`-`while` 循环、`for` 循环和基于范围的 `for` 循环。

#### *while 循环*

`while` 循环是基本的迭代机制。用法如下：

```
while (condition) {
  // The statement in the body of the loop
  // executes upon each iteration
}
```

在执行循环的每次迭代之前，`while` 循环会先评估 `condition` 表达式。如果为 `true`，循环继续。如果为 `false`，循环终止，如示例 8-20 所示。

```
#include <cstdio>
#include <cstdint>

bool double_return_overflow(uint8_t& x) { ➊
  const auto original = x;
  x *= 2;
  return original > x;
}
int main() {
  uint8_t x{ 1 }; ➋
  printf("uint8_t:\n===\n");
  while (!double_return_overflow(x)➌) {
    printf("%u ", x); ➍
  }
}
--------------------------------------------------------------------------
uint8_t:
===
2 4 8 16 32 64 128 ➍
```

*示例 8-20：一个程序，每次迭代时将 `uint8_t` 类型的值加倍，并打印新的值*

你声明了一个 `double_return_overflow` 函数，该函数通过引用接收一个 8 位无符号整数 ➊。该函数将参数加倍，并检查是否导致溢出。如果发生溢出，它返回 `true`。如果没有溢出，返回 `false`。

在进入 `while` 循环之前，你将变量 `x` 初始化为 1 ➋。`while` 循环中的条件表达式会评估 `double_return_overflow(x)` ➌。由于你是通过引用传递 `x`，它会对 `x` 进行加倍，这是它的副作用。该函数还会返回一个值，告诉你加倍是否导致了 `x` 的溢出。当条件表达式的结果为 `true` 时，循环将继续执行，但 `double_return_overflow` 被写成返回 `true`，当循环应该停止时。你通过在前面加上逻辑非运算符（`!`）来修复这个问题。（回顾 第七章，该操作会将 `true` 转换为 `false`，将 `false` 转换为 `true`。）因此，`while` 循环实际上是在问：“如果不是 `double_return_overflow` 为 true...”

最终结果是，你依次打印出 2、4、8，依此类推直到 128 ➍。

注意，值 1 从未打印，因为评估条件表达式会将 `x` 加倍。你可以通过将条件语句放在循环末尾来修改这种行为，这样就会得到一个 `do`-`while` 循环。

#### *do-while 循环*

`do`-`while` 循环与 `while` 循环相同，只是条件语句在循环完成后评估，而不是在循环之前。其用法如下：

```
do {
  // The statement in the body of the loop
  // executes upon each iteration
} while (condition);
```

由于条件在循环结束时进行评估，你可以保证循环至少会执行一次。

示例 8-21 将 示例 8-20 重构为 `do`-`while` 循环。

```
#include <cstdio>
#include <cstdint>

bool double_return_overflow(uint8_t& x) {
  --snip--
}

int main() {
  uint8_t x{ 1 };
  printf("uint8_t:\n===\n");
  do {
    printf("%u ", x); ➊
  } while (!double_return_overflow(x)➋);
}
--------------------------------------------------------------------------
uint8_t:
===
1 2 4 8 16 32 64 128 ➊
```

*示例 8-21：一个程序，它在每次迭代时将 `uint8_t` 的值加倍并打印新值*

注意，来自 示例 8-21 的输出现在以 1 开始 ➊。你所需要做的只是重新格式化 `while` 循环，将条件放在循环的末尾 ➋。

在大多数涉及迭代的情况中，你有三个任务：

1.  初始化某个对象。

1.  在每次迭代前更新对象。

1.  检查对象的值以满足某个条件。

你可以使用 `while` 或 `do`-`while` 循环来完成这些任务的一部分，但 `for` 循环提供了内建的功能，使得这些操作变得更加简便。

#### *for 循环*

`for` 循环是一个包含三个特殊表达式的迭代语句：*初始化*、*条件* 和 *迭代*，这些将在接下来的部分中进行描述。

##### 初始化表达式

初始化表达式类似于 `if` 的初始化：它只会在第一次迭代之前执行一次。在初始化表达式中声明的任何对象的生命周期都被限制在 `for` 循环的作用域内。

##### 条件表达式

`for` 循环的条件表达式会在每次循环迭代之前进行评估。如果条件为 `true`，则循环继续执行。如果条件为 `false`，则循环终止（这种行为与 `while` 循环和 `do`-`while` 循环的条件完全相同）。

与 `if` 和 `switch` 语句类似，`for` 允许你初始化具有与语句相同作用域的对象。

##### 迭代表达式

在每次 `for` 循环的迭代后，迭代表达式会进行评估。这个评估发生在条件表达式评估之前。请注意，迭代表达式在成功迭代后进行评估，因此在第一次迭代之前不会执行迭代表达式。

为了更清晰地说明，以下列表列出了 `for` 循环的典型执行顺序：

1.  初始化表达式

1.  条件表达式

1.  （循环主体）

1.  迭代表达式

1.  条件表达式

1.  （循环主体）

步骤 4 到 6 会重复执行，直到条件表达式返回 `false`。

##### 用法

列表 8-22 演示了如何使用 `for` 循环。

```
for(initialization➊; conditional➋; iteration➌) {
  // The statement in the body of the loop
  // executes upon each iteration
}
```

*列表 8-22：使用 `for` 循环*

`for` 循环的初始化 ➊、条件 ➋ 和迭代 ➌ 表达式位于括号中，位于 `for` 循环主体之前。

##### 使用索引进行迭代

`for` 循环非常适合遍历类数组对象的组成元素。你使用一个辅助的 *索引* 变量来遍历数组对象有效索引的范围。你可以使用这个索引按顺序与每个数组元素进行交互。列表 8-23 使用一个索引变量来打印数组的每个元素及其索引。

```
#include <cstdio>

int main() {
  const int x[]{ 1, 1, 2, 3, 5, 8 }; ➊
  printf("i: x[i]\n"); ➋
  for (int i{}➌; i < 6➍; i++➎) {
    printf("%d: %d\n", i, x[i]);
  }
}
--------------------------------------------------------------------------
i: x[i] ➋
0: 1
1: 1
2: 2
3: 3
4: 5
5: 8
```

*列表 8-23：遍历斐波那契数列数组的程序*

你初始化一个名为 `x` 的 `int` 数组，包含前六个斐波那契数 ➊。在打印输出标题 ➋ 后，你构建一个包含初始化 ➌、条件 ➍ 和迭代 ➎ 表达式的 `for` 循环。初始化表达式首先执行，并将索引变量 `i` 初始化为零。

列表 8-23 显示了一种自 1950 年代以来未曾改变的编码模式。你可以通过使用现代的基于范围的 `for` 循环来消除大量样板代码。

#### *基于范围的 `for` 循环*

基于范围的 `for` 循环在没有索引变量的情况下遍历一系列值。范围（或 *范围表达式*）是一个对象，基于范围的 `for` 循环知道如何遍历它。许多 C++ 对象是有效的范围表达式，包括数组。（你将在 第二部分 中学习到的所有 stdlib 容器也是有效的范围表达式。）

##### 用法

基于范围的 `for` 循环用法如下所示：

```
for(range-declaration : range-expression) {
  // The statement in the body of the loop
  // executes upon each iteration
}
```

*范围声明* 声明一个命名变量。这个变量必须与范围表达式所暗示的类型相同（你可以使用 `auto`）。

列表 8-24 重构了 列表 8-23 ，使用基于范围的 `for` 循环。

```
#include <cstdio>

int main() {
  const int x[]{ 1, 1, 2, 3, 5, 8 }; ➊
  for (const auto element➋ : x➌) {
    printf("%d ", element➍);
  }
}
--------------------------------------------------------------------------
1 1 2 3 5 8
```

*列表 8-24：一个基于范围的`for`循环，迭代前六个斐波那契数*

你仍然声明一个数组`x`，包含六个斐波那契数 ➊。基于范围的`for`循环包含一个范围声明表达式 ➋，在其中声明`element`变量来保存范围的每个元素。它还包含范围表达式`x` ➌，其中包含你希望迭代并打印的元素 ➍。

这段代码整洁多了！

##### 范围表达式

你可以定义自己的类型，这些类型也可以作为有效的范围表达式。但是，你需要在你的类型上指定几个函数。

每个范围都暴露了`begin`和`end`方法。这些函数代表了基于范围的`for`循环与范围交互的通用接口。两个方法都返回*迭代器*。迭代器是一个支持`operator!=`、`operator++`和`operator*`的对象。

让我们看看这些部分是如何结合在一起的。在底层，基于范围的`for`循环看起来就像列表 8-25 中的循环。

```
const auto e = range.end();➊
for(auto b = range.begin()➋; b != e➌; ++b➍) {
  const auto& element➎ = *b;
}
```

*列表 8-25：一个模拟基于范围的`for`循环的`for`循环*

初始化表达式存储了两个变量，`b` ➋ 和 `e` ➊，分别初始化为`range.begin()`和`range.end()`。条件表达式检查`b`是否等于`e`，如果相等，则表示循环已完成 ➌（这是惯例）。迭代表达式使用前缀操作符 ➍ 增加`b`。最后，迭代器支持解引用操作符`*`，因此可以提取指向的元素 ➎。

**注意**

*`begin`和`end`返回的类型不需要相同。要求是`begin`上的`operator!=`接受一个`end`参数，以支持比较`begin != end`。*

##### 一个斐波那契范围

你可以实现一个`FibonacciRange`，它将生成一个任意长的斐波那契数列。从上一节中，你知道这个范围必须提供一个返回迭代器的`begin`和`end`方法。在本示例中，这个迭代器称为`FibonacciIterator`，它必须提供`operator!=`、`operator++`和`operator*`。

列表 8-26 实现了一个`FibonacciIterator`和一个`FibonacciRange`。

```
struct FibonacciIterator {
  bool operator!=(int x) const {
    return x >= current; ➊
  }

  FibonacciIterator& operator++() {
    const auto tmp = current; ➋
    current += last; ➌
    last = tmp; ➍
    return *this; ➎
  }

  int operator*() const {
    return current; ➏
 }
private:
  int current{ 1 }, last{ 1 };
};

struct FibonacciRange {
  explicit FibonacciRange(int max➐) : max{ max } { }
  FibonacciIterator begin() const { ➑
    return FibonacciIterator{};
  }
  int end() const { ➒
    return max;
  }
private:
  const int max;
};
```

*列表 8-26：`FibonacciIterator`和`FibonacciRange`的实现*

`FibonacciIterator` 有两个字段，`current` 和 `last`，它们初始化为 1。它们跟踪 Fibonacci 序列中的两个值。其 `operator!=` 检查传入的参数是否大于或等于 `current` ➊。回想一下，这个参数是在基于范围的 `for` 循环中的条件表达式里使用的。如果范围内还有元素，它应该返回 `true`；否则返回 `false`。`operator++` 出现在迭代表达式中，负责为下一次迭代设置迭代器。你首先将 `current` 值保存到临时变量 `tmp` ➋。接下来，你通过 `last` 递增 `current`，得到下一个 Fibonacci 数字 ➌。（这遵循 Fibonacci 序列的定义。）然后你将 `last` 设置为 `tmp` ➍ 并返回对 `this` 的引用 ➎。最后，你实现了 `operator*`，它直接返回 `current` ➏。

`FibonacciRange` 要简单得多。它的构造函数接受一个最大参数，定义了范围的上限 ➐。`begin` 方法返回一个新的 `FibonacciIterator` ➑，而 `end` 方法返回 `max` ➒。

现在应该显而易见为什么你需要在 `FibonacciIterator` 上实现 `bool operator!=(int x)`，而不是比如说在 `bool operator!=(const FibonacciIterator& x)` 上实现：一个 `FibonacciRange` 从 `end()` 返回一个 `int`。

你可以在基于范围的 `for` 循环中使用 `FibonacciRange`，正如在 清单 8-27 中所展示的那样。

```
#include <cstdio>

struct FibonacciIterator {
  --snip--
};

struct FibonacciRange {
  --snip--;
};

int main() {
 for (const auto i : FibonacciRange{ 5000 }➊) {
    printf("%d ", i); ➋
  }
}
--------------------------------------------------------------------------
1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 1597 2584 4181 ➋
```

*清单 8-27：在程序中使用 `FibonacciRange`*

在 清单 8-26 中实现 `FibonacciIterator` 和 `FibonacciRange` 需要一些工作，但其回报是巨大的。在 `main` 中，你只需构造一个带有所需上限的 `FibonacciRange` ➊，基于范围的 `for` 循环会为你处理其他所有事情。你只需在 `for` 循环中使用生成的元素 ➋。

清单 8-27 与 清单 8-28 功能上是等价的，后者将基于范围的 `for` 循环转换成传统的 `for` 循环。

```
#include <cstdio>

struct FibonacciIterator {
  --snip--
};

struct FibonacciRange {
  --snip--;
};

int main() {
  FibonacciRange range{ 5000 };
  const auto end = range.end();➊
  for (auto x = range.begin()➋; x != end ➌; ++x ➍) {
    const auto i = *x;
    printf("%d ", i);
  }
}
--------------------------------------------------------------------------
1 2 3 5 8 13 21 34 55 89 144 233 377 610 987 1597 2584 4181
```

*清单 8-28：使用传统 `for` 循环重构 清单 8-27*

清单 8-28 展示了所有部分如何结合在一起。调用 `range.begin()` ➋ 会返回一个 `FibonacciIterator`。当你调用 `range.end()` ➊ 时，它会返回一个 `int`。这些类型直接来源于 `FibonacciRange` 中 `begin()` 和 `end()` 方法的定义。条件语句 ➌ 在 `FibonacciIterator` 上使用 `operator!=(int)` 来实现以下行为：如果迭代器 `x` 已经超过了传给 `operator!=` 的 `int` 参数，条件语句将评估为 `false`，并且循环结束。你还实现了 `FibonacciIterator` 上的 `operator++`，所以 `++x` ➍ 会在 `FibonacciIterator` 中递增 Fibonacci 数字。

当你对比 清单 8-27 和 8-28 时，你可以看到基于范围的 `for` 循环隐藏了多少繁琐的工作。

**注意**

*你可能会想：“当然，基于范围的 `for` 循环看起来更简洁，但实现 `FibonacciIterator` 和 `FibonacciRange` 需要做很多工作。”这是一个很好的观点，对于一次性使用的代码，你可能不会以这种方式重构代码。范围的主要用途是，当你编写库代码、编写你会经常重用的代码，或者只是使用别人编写的范围时。*

### 跳转语句

*跳转语句*，包括 `break`、`continue` 和 `goto`，用于转移控制流。与选择语句不同，跳转语句并不具有条件性。你应该避免使用它们，因为它们几乎总是可以被更高级的控制结构所替代。这里讨论这些语句是因为你可能在旧版 C++ 代码中看到它们，它们仍然在许多 C 代码中起着核心作用。

#### *跳出语句*

`break` 语句终止外层迭代或 `switch` 语句的执行。一旦 `break` 完成，控制流会转移到紧跟在 `for`、基于范围的 `for`、`while`、`do`-`while` 或 `switch` 语句之后的语句。

你已经在 `switch` 语句中使用过 `break`；一旦某个分支执行完毕，`break` 语句就会终止 `switch` 语句。回想一下，如果没有 `break` 语句，`switch` 语句会继续执行所有后续的分支。

清单 8-29 重构了 清单 8-27，当迭代器 `i` 等于 21 时跳出基于范围的 `for` 循环。

```
#include <cstdio>

struct FibonacciIterator {
  --snip--
};

struct FibonacciRange {
  --snip--;
};

int main() {
  for (auto i : FibonacciRange{ 5000 }) {
    if (i == 21) { ➊
      printf("*** "); ➋
      break; ➌
    }
    printf("%d ", i);
  }
}
--------------------------------------------------------------------------
1 2 3 5 8 13 *** ➋
```

*清单 8-29：重构自 清单 8-27，当迭代器等于 21 时跳出*

添加了一个 `if` 语句，用来检查 `i` 是否等于 21 ➊。若是，它会打印三个星号 `***` ➋ 并执行 `break` ➌。注意输出结果：程序没有打印 21，而是打印了三个星号，并且 `for` 循环终止了。与 清单 8-27 的输出结果比较。

#### *continue 语句*

`continue` 语句跳过外层迭代语句的其余部分，并继续下一次迭代。清单 8-30 将 清单 8-29 中的 `break` 替换为 `continue`。

```
#include <cstdio>

struct FibonacciIterator {
  --snip--
};

struct FibonacciRange {
  --snip--;
};

int main() {
  for (auto i : FibonacciRange{ 5000 }) {
    if (i == 21) {
      printf("*** "); ➊
      continue; ➋
    }
    printf("%d ", i);
  }
}
--------------------------------------------------------------------------
1 2 3 5 8 13 *** ➊34 55 89 144 233 377 610 987 1597 2584 4181
```

*清单 8-30：将 清单 8-29 重构为使用 `continue` 替代 `break`*

当 `i` 等于 21 时，你仍然打印三个星号 ➊，但你使用 `continue` 替代 `break` ➋。这导致 21 不再打印，类似于 清单 8-29；然而，与 清单 8-29 不同， 清单 8-30 会继续迭代。（比较输出结果。）

#### *goto 语句*

`goto` 语句是一个无条件跳转。`goto` 语句的目标是一个标签。

##### 标签

*标签* 是你可以添加到任何语句的标识符。标签为语句赋予了名称，但它们对程序没有直接影响。要分配标签，只需在语句前加上所需标签的名称，后跟一个冒号。

列表 8-31 为一个简单程序添加了 `luke` 和 `yoda` 标签。

```
#include <cstdio>

int main() {
luke: ➊
  printf("I'm not afraid.\n");
yoda: ➋
  printf("You will be.");
}
--------------------------------------------------------------------------
I'm not afraid.
You will be.
```

*列表 8-31：带标签的简单程序*

标签 ➊➋ 本身不执行任何操作。

##### `goto` 的使用

`goto` 语句的用法如下：

```
goto label;
```

例如，你可以使用 `goto` 语句不必要地使列表 8-32 中的简单程序变得晦涩。

```
#include <cstdio>

int main() {
  goto silent_bob; ➊
luke:
  printf("I'm not afraid.\n");
  goto yoda; ➌
silent_bob:
  goto luke; ➋
yoda:
  printf("You will be.");
}
--------------------------------------------------------------------------
I'm not afraid.
You will be.
```

*列表 8-32：展示 `goto` 语句的意大利面代码*

在列表 8-32 中的控制流先跳转到 `silent_bob` ➊，再到 `luke` ➋，然后到 `yoda` ➌。

##### `goto` 在现代 C++ 程序中的作用

在现代 C++ 中，`goto` 语句没有什么好的用途。不要使用它们。

**注意**

*在写得不好的 C++（以及大多数 C 代码）中，你可能会看到 `goto` 被用作一种原始的错误处理机制。很多系统编程涉及获取资源、检查错误条件以及清理资源。RAII（资源获取即初始化）范式巧妙地抽象了这些细节，但 C 语言并没有 RAII。有关更多信息，请参见 C 程序员的序言，见第 xxxvii 页。*

### 总结

在本章中，你学习了可以在程序中使用的不同类型的语句。它们包括声明和初始化、选择语句以及迭代语句。

**注意**

*请记住，`try-catch` 块也是语句，但它们已经在第四章中详细讨论过。*

**习题**

**8-1.** 将列表 8-27 重构为独立的翻译单元：一个用于 `main`，另一个用于 `FibonacciRange` 和 `FibonacciIterator`。使用头文件共享两个翻译单元之间的定义。

**8-2.** 实现一个 `PrimeNumberRange` 类，可用于在范围表达式中迭代所有小于给定值的素数。再次使用单独的头文件和源文件。

**8-3.** 将 `PrimeNumberRange` 集成到列表 8-27 中，增加另一个循环，生成所有小于 5,000 的素数。

**进一步阅读**

+   *ISO 国际标准 ISO/IEC（2017）— 编程语言 C++*（国际标准化组织；瑞士日内瓦； *[`isocpp.org/std/the-standard/`](https://isocpp.org/std/the-standard/)*）

+   *《随机数生成与蒙特卡罗方法》，第二版，詹姆斯·E·詹特尔著（Springer-Verlag，2003）*

+   *《随机数生成与准蒙特卡罗方法》，哈拉尔德·尼德赖特著（SIAM 第 63 卷，1992）*
