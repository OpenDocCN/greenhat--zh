## 第八章：**编译时多态**

*越灵活，越有趣。*

—玛莎·斯图尔特*

![图片](img/common.jpg)

在本章中，你将学习如何通过模板实现编译时多态。你将学习如何声明和使用模板，强制类型安全，并探讨模板的更多高级用法。本章最后会对 C++ 中的运行时多态和编译时多态进行比较。

### 模板

C++ 通过*模板*实现编译时多态。模板是一个带有模板参数的类或函数。这些参数可以代表任何类型，包括基本类型和用户自定义类型。当编译器看到模板与某个类型一起使用时，它会生成一个专门的模板实例。

*模板实例化* 是从模板创建类或函数的过程。有些时候，令人困惑的是，你也可以将“模板实例化”称为模板实例化过程的结果。模板实例化有时被称为具体类和具体类型。

这个大致的想法是，与其到处复制粘贴常见代码，不如编写一个模板；当编译器遇到模板参数的新类型组合时，它会生成新的模板实例。

### 声明模板

你用一个*template 前缀*来声明模板，前缀由关键字 `template` 和尖括号 `< >` 组成。在尖括号内，你放置一个或多个模板参数的声明。你可以使用 `typename` 或 `class` 关键字后跟标识符来声明模板参数。例如，模板前缀 `template<typename T>` 表明该模板接受一个模板参数 `T`。

**注意**

*`typename` 和 `class` 关键字的共存是不幸且令人困惑的。它们的意思相同。（由于历史原因，它们都被支持。）本章始终使用 `typename`。*

#### *模板类定义*

考虑 列表 6-1 中的 `MyTemplateClass`，它接受三个模板参数：`X`、`Y` 和 `Z`。

```
template➊<typename X, typename Y, typename Z> ➋
struct MyTemplateClass➌ {
  X foo(Y&); ➍
private:
  Z* member; ➎
};
```

*列表 6-1：一个具有三个模板参数的模板类*

`template` 关键字 ➊ 开始模板前缀，其中包含模板参数 ➋。这个 `template` 前言导致 `MyTemplateClass` ➌ 的剩余声明有些特别。在 `MyTemplateClass` 中，你像使用任何完全指定的类型（如 `int` 或用户定义的类）一样使用 `X`、`Y` 和 `Z`。

`foo` 方法接受一个 `Y` 引用并返回一个 `X` ➍。你可以声明包含模板参数的成员类型，比如指向 `Z` 的指针 ➎。除了特殊的前缀 ➊ 外，这个模板类与非模板类基本相同。

#### *模板函数定义*

你还可以指定模板函数，比如在列表 6-2 中也接受三个模板参数：`X`、`Y` 和 `Z` 的 `my_template_function`。

```
template<typename X, typename Y, typename Z>
X my_template_function(Y& arg1, const Z* arg2) {
  --snip--
}
```

*清单 6-2：一个具有三个模板参数的模板函数*

在 `my_template_function` 的函数体内，你可以根据需要使用 `arg1` 和 `arg2`，只要你返回一个类型为 `X` 的对象。

#### *实例化模板*

要实例化一个模板类，请使用以下语法：

```
tc_name➊<t_param1➋, t_param2, ...> my_concrete_class{ ... }➌;
```

tc_name ➊ 是你放置模板类名称的地方。接下来，你填写你的模板参数 ➋。最后，你将模板名称和参数的组合视为普通类型：你可以使用任何初始化语法 ➌。

实例化一个模板函数是类似的：

```
auto result = tf_name➊<t_param1➋, t_param2, ...>(f_param1➌, f_param2, ...);
```

tf_name ➊ 是你放置模板函数名称的地方。你按照模板类的方式填写参数 ➋。你将模板名称和参数的组合视为普通类型。你通过括号和函数参数来调用这个模板函数实例化 ➌。

所有这些新的符号对初学者来说可能很令人生畏，但一旦习惯了，就不会那么难。实际上，它们在一组语言特性中得到了应用，这些特性被称为命名转换函数。

### 命名转换函数

*命名转换* 是语言特性，用于显式地将一种类型转换为另一种类型。你在无法使用隐式转换或构造函数获取所需类型的情况下，谨慎使用命名转换。

所有命名转换接受一个对象参数，即你希望转换的 `object-to-cast`，以及一个类型参数，即你希望转换成的目标类型 `desired-type`：

```
named-conversion<desired-type>(object-to-cast)
```

例如，如果你需要修改一个 `const` 对象，你首先需要去掉 `const` 限定符。命名转换函数 `const_cast` 允许你执行此操作。其他命名转换帮助你逆转隐式转换（`static_cast`）或以不同类型重新解释内存（`reinterpret_cast`）。

**注意**

*尽管命名转换函数在技术上不是模板函数，但它们在概念上与模板非常相似——这一关系体现在它们的语法相似性上。*

#### *const_cast*

`const_cast` 函数去掉了 `const` 修饰符，允许修改 `const` 值。`object-to-cast` 是某个 `const` 类型的对象，所需的目标类型是去掉 `const` 限定符的该类型。

请考虑清单 6-3 中的 `carbon_thaw` 函数，它接受一个 `const` 引用的 `encased_solo` 参数。

```
void carbon_thaw(const➊ int& encased_solo) {
  //encased_solo++; ➋ // Compiler error; modifying const
  auto& hibernation_sick_solo = const_cast➌<int&➍>(encased_solo➎);
  hibernation_sick_solo++; ➏
}
```

*清单 6-3：使用 `const_cast` 的函数。取消注释会导致编译器错误。*

`encased_solo` 参数是 `const` ➊，因此任何试图修改它的行为 ➋ 都会导致编译器错误。你可以使用 `const_cast` ➌ 来获取非 `const` 引用 `hibernation_sick_solo`。`const_cast` 接受一个模板参数，即你希望转换为的类型 ➍。它还接受一个函数参数，即你希望去除 `const` 的对象 ➎。然后，你就可以通过新的非 `const` 引用 ➏ 来修改 `encased_solo` 指向的 `int`。

只使用 `const_cast` 来获取对 `const` 对象的写访问权限。任何其他类型的转换都将导致编译错误。

**注意**

*显然，你可以使用 `const_cast` 向对象的类型添加 `const`，但不应该这么做，因为它冗长且不必要。最好使用隐式转换。在 第七章 中，你将学习 `volatile` 修饰符是什么。你也可以使用 `const_cast` 从对象中移除 `volatile` 修饰符。*

#### *static_cast*

`static_cast` 反转一个明确定义的隐式转换，例如整数类型到另一个整数类型。`object-to-cast` 是某种类型，`desired-type` 可以隐式地转换成该类型。你可能需要使用 `static_cast` 的原因是，一般来说，隐式转换不可逆。

示例 6-4 中的程序定义了一个 `increment_as_short` 函数，该函数接受一个 `void` 指针参数。它使用 `static_cast` 从这个参数创建一个 `short` 指针，递增指向的 `short`，并返回结果。在一些低级应用中，如网络编程或处理二进制文件格式，你可能需要将原始字节解释为整数类型。

```
#include <cstdio>
short increment_as_short(void*➊ target) {
  auto as_short = static_cast➋<short*➌>(target➍);
  *as_short = *as_short + 1;
  return *as_short;
}

int main() {
  short beast{ 665 };
  auto mark_of_the_beast = increment_as_short(&beast);
  printf("%d is the mark_of_the_beast.", mark_of_the_beast);
}
--------------------------------------------------------------------------
666 is the mark_of_the_beast.
```

*示例 6-4：使用 `static_cast` 的程序*

`target` 参数是一个 `void` 指针 ➊。你使用 `static_cast` 将 `target` 转换为 `short*` ➋。模板参数是所需的类型 ➌，函数参数是你想要转换的对象 ➍。

注意，`short*` 到 `void*` 的隐式转换是明确定义的。尝试使用 `static_cast` 进行未定义的转换，例如将 `char*` 转换为 `float*`，将导致编译错误：

```
float on = 3.5166666666;
auto not_alright = static_cast<char*>(&on); // Bang!
```

要执行这样的链锯杂技，你需要使用`reinterpret_cast`。

#### *reinterpret_cast*

有时在低级编程中，你必须执行一些未定义类型转换。在系统编程中，尤其是在嵌入式环境下，你通常需要完全控制如何解释内存。`reinterpret_cast` 给了你这种控制，但确保这些转换的正确性完全是你的责任。

假设你的嵌入式设备在内存地址 0x1000 处保存了一个 `unsigned long` 类型的定时器。你可以使用 `reinterpret_cast` 来读取定时器，正如 示例 6-5 中所示。

```
#include <cstdio>
int main() {
  auto timer = reinterpret_cast➊<const unsigned long*➋>(0x1000➌);
  printf("Timer is %lu.", *timer);
}
```

*示例 6-5：使用 `reinterpret_cast` 的程序。该程序将编译，但除非 0x1000 是可读的，否则你应预期程序在运行时崩溃。*

`reinterpret_cast` ➊ 需要一个类型参数，对应于所需的指针类型 ➋ 和结果应指向的内存地址 ➌。

当然，编译器无法知道地址 0x1000 处的内存是否包含一个 `unsigned long`。完全由你负责确保正确性。因为你要为这个非常危险的构造承担全部责任，编译器强制你使用 `reinterpret_cast`。例如，你不能将 `timer` 的初始化替换为以下行：

```
const unsigned long* timer{ 0x1000 };
```

编译器会抱怨将`int`转换为指针。

#### *narrow_cast*

列表 6-6 展示了一个自定义的`static_cast`，它执行运行时检查以检测*缩小*。缩小是信息丢失的过程。想象一下从`int`转换为`short`。只要`int`的值能适应`short`，转换就是可逆的，不会发生缩小。如果`int`的值太大，超出了`short`的最大值，那么转换就是不可逆的，会导致缩小。

让我们实现一个名为`narrow_cast`的转换，它会检查缩小并在检测到时抛出`runtime_error`。

```
#include <stdexcept>
template <typename To➊, typename From➋>
To➌ narrow_cast(From➍ value) {
  const auto converted = static_cast<To>(value); ➎
  const auto backwards = static_cast<From>(converted); ➏
  if (value != backwards) throw std::runtime_error{ "Narrowed!" }; ➐
  return converted; ➑
}
```

*列表 6-6：`narrow_cast`的定义*

`narrow_cast`函数模板有两个模板参数：您要转换的类型`To` ➊和您要转换的类型`From` ➋。您可以看到这些模板参数在函数的返回类型 ➌ 和参数值的类型 ➍ 中的实际应用。首先，您使用`static_cast`执行请求的转换，得到`converted` ➎。接着，您将转换方向反转（从`converted`转换为类型`From`），得到`backwards` ➏。如果`value`不等于`backwards`，说明您进行了缩小，因此抛出一个异常 ➐。否则，返回`converted` ➑。

您可以在列表 6-7 中看到`narrow_cast`的实际应用。

```
#include <cstdio>
#include <stdexcept>

template <typename To, typename From>
To narrow_cast(From value) {
  --snip--
}
int main() {
  int perfect{ 496 }; ➊
  const auto perfect_short = narrow_cast<short>(perfect); ➋
  printf("perfect_short: %d\n", perfect_short); ➌
  try {
    int cyclic{ 142857 }; ➍
    const auto cyclic_short = narrow_cast<short>(cyclic); ➎
    printf("cyclic_short: %d\n", cyclic_short);
  } catch (const std::runtime_error& e) {
    printf("Exception: %s\n", e.what()); ➏
  }
}
--------------------------------------------------------------------------
perfect_short: 496 ➌
Exception: Narrowed! ➏
```

*列表 6-7：使用`narrow_cast`的程序。（输出来自在 Windows 10 x64 上的执行。）*

首先，您将`perfect`初始化为 496 ➊，然后将其`narrow_cast`为短整型`perfect_short` ➋。此操作不会出现异常，因为值 496 可以轻松适应 Windows 10 x64 上的 2 字节`short`（最大值为 32767）。您会看到预期的输出 ➌。接下来，您将`cyclic`初始化为 142857 ➍，并尝试将其`narrow_cast`为短整型`cyclic_short` ➎。这会抛出一个`runtime_error`，因为 142857 大于`short`的最大值 32767。`narrow_cast`中的检查会失败。您会在`output`中看到异常 ➏。

请注意，在实例化时，您只需要提供一个模板参数，即返回类型 ➋➎。编译器可以根据使用情况推断出`From`参数。

### mean：模板函数示例

请参阅列表 6-8 中计算`double`数组均值的函数，该函数使用求和除法方法。

```
#include <cstddef>

double mean(const double* values, size_t length) {
  double result{}; ➊
  for(size_t i{}; i<length; i++) {
    result += values[i]; ➋
  }
  return result / length; ➌
}
```

*列表 6-8：计算数组均值的函数*

您将`result`变量初始化为零 ➊。接下来，通过遍历每个索引`i`，将对应的元素添加到`result`中 ➋。然后，您将`result`除以`length`并返回 ➌。

#### *泛型化均值*

假设您想支持其他数值类型的`mean`计算，例如`float`或`long`。您可能会想，“这就是函数重载的作用！”从本质上来说，您是对的。

清单 6-9 重载了`mean`，使其接受一个`long`数组。最简单的方法是复制并粘贴原始代码，然后将`double`替换为`long`。

```
#include <cstddef>

long➊ mean(const long*➋ values, size_t length) {
  long result{}; ➌
  for(size_t i{}; i<length; i++) {
    result += values[i];
  }
  return result / length;
}
```

*清单 6-9：清单 6-8 的一个重载版本，接受`long`数组*

这确实是大量的复制粘贴，而且你几乎没有做任何改变：返回类型➊，函数参数➋，以及`result` ➌。

随着你添加更多类型，这种方法无法扩展。如果你想支持其他整型类型，比如`short`类型或`uint_64`类型怎么办？`float`类型呢？如果后来你想重构`mean`中的某些逻辑呢？你将面临大量繁琐且容易出错的维护工作。

在清单 6-9 中，`mean`有三个更改，所有更改都涉及将`double`类型替换为`long`类型。理想情况下，每当编译器遇到不同类型的使用时，它可以自动为你生成该函数的版本。关键是逻辑没有变化——只是类型发生了变化。

解决这个复制粘贴问题所需要的是*泛型编程*，这是一种使用尚未指定的类型进行编程的编程风格。你可以利用 C++对模板的支持实现泛型编程。模板允许编译器基于正在使用的类型实例化自定义类或函数。

现在你知道如何声明模板了，再看看`mean`函数。你仍然希望`mean`能够接受广泛的类型——不仅仅是`double`类型——但你不希望一遍又一遍地复制粘贴相同的代码。

考虑如何将清单 6-8 重构为一个模板函数，正如清单 6-10 中所演示的那样。

```
#include <cstddef>

template<typename T> ➊
T➋ mean(constT*➌ values, size_t length) {
  T➍ result{};
  for(size_t i{}; i<length; i++) {
    result += values[i];
 }
  return result / length;
}
```

*清单 6-10：将清单 6-8 重构为模板函数*

清单 6-10 以模板前缀➊开始。这个前缀传递了一个模板参数`T`。接下来，你更新`mean`，将`T`替换为`double` ➋➌➍。

现在，你可以用许多不同的类型来使用`mean`。每当编译器遇到使用新类型的`mean`时，它会执行模板实例化。这就*好像*你做了复制粘贴和替换类型的操作，但编译器在执行细节导向的、单调的任务上比你要强得多。考虑清单 6-11 中的示例，它计算`double`、`float`和`size_t`类型的均值。

```
#include <cstddef>
#include <cstdio>

template<typename T>
T mean(const T* values, size_t length) {
  --snip--
}

int main() {
  const double nums_d[] { 1.0, 2.0, 3.0, 4.0 };
  const auto result1 = mean<double>(nums_d, 4); ➊
  printf("double: %f\n", result1);

  const float nums_f[] { 1.0f, 2.0f, 3.0f, 4.0f };
  const auto result2 = mean<float>(nums_f, 4); ➋
  printf("float: %f\n", result2);

  const size_t nums_c[] { 1, 2, 3, 4 };
  const auto result3 = mean<size_t>(nums_c, 4); ➌
  printf("size_t: %zu\n", result3);
}
--------------------------------------------------------------------------
double: 2.500000
float: 2.500000
size_t: 2
```

*清单 6-11：使用模板函数`mean`的程序*

三个模板被实例化了➊➋➌；这就像你手动生成了清单 6-12 中孤立的重载函数。（每个模板实例化包含了类型，类型以粗体显示，表示编译器为模板参数替换了类型。）

```
double mean(const double* values, size_t length) {
  double result{};
  for(size_t i{}; i<length; i++) {
    result += values[i];
  }
 return result / length;
}

float mean(const float* values, size_t length) {
  float result{};
  for(size_t i{}; i<length; i++) {
    result += values[i];
  }
  return result / length;
}

size_t mean(const size_t* values, size_t length) {
  size_t result{};
  for(size_t i{}; i<length; i++) {
    result += values[i];
  }
  return result / length;
}
```

*清单 6-12：为清单 6-11 生成的模板实例化*

编译器为你做了很多工作，但你可能已经注意到，你必须两次输入指向数组的类型：一次是声明数组，另一次是指定模板参数。这变得很繁琐，并且可能导致错误。如果模板参数不匹配，通常会得到编译器错误或导致意外的类型转换。

幸运的是，调用模板函数时通常可以省略模板参数。编译器用来确定正确模板参数的过程叫做*模板类型推导*。

#### *模板类型推导*

通常情况下，你不需要提供模板函数的参数。编译器可以从使用情况中推导出这些参数，因此可以看到清单 6-11 在没有显式模板参数的情况下的重写版本，见清单 6-13。

```
#include <cstddef>
#include <cstdio>

template<typename T>
T mean(const T* values, size_t length) {
  --snip--
}

int main() {
  const double nums_d[] { 1.0, 2.0, 3.0, 4.0 };
  const auto result1 = mean(nums_d, 4); ➊
  printf("double: %f\n", result1);

  const float nums_f[] { 1.0f, 2.0f, 3.0f, 4.0f };
  const auto result2 = mean(nums_f, 4); ➋
  printf("float: %f\n", result2);

  const size_t nums_c[] { 1, 2, 3, 4 };
 const auto result3 = mean(nums_c, 4); ➌
  printf("size_t: %zu\n", result3);
}
--------------------------------------------------------------------------
double: 2.500000
float: 2.500000
size_t: 2
```

*清单 6-13：一个没有显式模板参数的清单 6-11 重构版本*

从使用情况来看，模板参数分别是`double` ➊、`float` ➋和`size_t` ➌。

**注意**

*模板类型推导通常按照你预期的方式工作，但如果你编写大量通用代码，你可能会遇到一些细节问题。有关更多信息，请参阅 ISO 标准[temp]。另外，参考 Scott Meyers 的《Effective Modern C++》中的第 1 条和 Bjarne Stroustrup 的《C++程序设计语言（第 4 版）》中的第 23.5.1 节。*

有时，模板参数无法推导。例如，如果模板函数的返回类型是一个完全独立于其他函数和模板参数的模板参数，你必须显式指定模板参数。

### SimpleUniquePointer：一个模板类示例

*唯一指针*是一个围绕自由存储分配对象的 RAII 封装器。正如其名称所示，唯一指针在任何时刻只有一个所有者，因此当唯一指针的生命周期结束时，所指向的对象会被销毁。

在唯一指针中，底层对象的类型并不重要，这使得它们成为模板类的理想候选。考虑清单 6-14 中的实现。

```
template <typename T> ➊
struct SimpleUniquePointer {
  SimpleUniquePointer() = default; ➋
  SimpleUniquePointer(T* pointer)
    : pointer{ pointer } { ➌
  }
  ~SimpleUniquePointer() { ➍
    if(pointer) delete pointer;
  }
  SimpleUniquePointer(const SimpleUniquePointer&) = delete;
  SimpleUniquePointer& operator=(const SimpleUniquePointer&) = delete; ➎
  SimpleUniquePointer(SimpleUniquePointer&& other) noexcept ➏
    : pointer{ other.pointer } {
    other.pointer = nullptr;
  }
  SimpleUniquePointer& operator=(SimpleUniquePointer&& other) noexcept { ➐
    if(pointer) delete pointer;
    pointer = other.pointer;
    other.pointer = nullptr;
    return *this;
 }
  T* get() { ➑
    return pointer;
  }
private:
  T* pointer;
};
```

*清单 6-14：一个简单的唯一指针实现*

你通过一个模板前缀➊声明模板类，这样就确立了`T`作为封装对象的类型。接下来，使用`default`关键字➋指定默认构造函数。（回想一下第四章，当你需要一个默认构造函数*和*一个非默认构造函数时，必须使用`default`。）生成的默认构造函数会根据默认初始化规则将私有成员`T*`指针初始化为`nullptr`。你还有一个非默认构造函数，它接受一个`T*`并将私有成员指针设置为➌。因为指针可能是`nullptr`，析构函数在删除之前会进行检查➍。

因为你只想允许指向对象的唯一所有者，所以你删除了拷贝构造函数和拷贝赋值运算符 ➎。这样可以防止双重释放问题，正如在第四章中讨论的那样。然而，你可以通过添加移动构造函数 ➏ 来使你的唯一指针可移动。这会从 `other` 中窃取 `pointer` 的值，然后将 `other` 的指针设置为 `nullptr`，将指向对象的责任交给 `this`。一旦移动构造函数返回，已移动的对象会被销毁。因为已移动对象的指针被设置为 `nullptr`，所以析构函数不会删除指向的对象。

由于 `this` 可能已经拥有一个对象，这使得移动赋值变得复杂 ➐。你必须显式检查是否已经拥有该对象，因为如果未能删除指针，会导致资源泄漏。通过这次检查后，你执行与拷贝构造函数相同的操作：将 `pointer` 设置为 `other.pointer` 的值，然后将 `other.pointer` 设置为 `nullptr`。这确保了被移动的对象不会删除指向的对象。

你可以通过调用 `get` 方法直接访问底层指针 ➑。

让我们请出老朋友 `Tracer`，它出现在列表 4-5 中，来调查 `SimpleUniquePointer`。考虑一下列表 6-15 中的程序。

```
#include <cstdio>
#include <utility>

template <typename T>
struct SimpleUniquePointer {
  --snip--
};

struct Tracer {
  Tracer(const char* name) : name{ name } {
    printf("%s constructed.\n", name); ➊
  }
  ~Tracer() {
 printf("%s destructed.\n", name); ➋
  }
private:
  const char* const name;
};

void consumer(SimpleUniquePointer<Tracer> consumer_ptr) {
  printf("(cons) consumer_ptr: 0x%p\n", consumer_ptr.get()); ➌
}

int main() {
  auto ptr_a = SimpleUniquePointer(new Tracer{ "ptr_a" });
  printf("(main) ptr_a: 0x%p\n", ptr_a.get()); ➍
  consumer(std::move(ptr_a));
  printf("(main) ptr_a: 0x%p\n", ptr_a.get()); ➎
}
--------------------------------------------------------------------------
ptr_a constructed. ➊
(main) ptr_a: 0x000001936B5A2970 ➍
(cons) consumer_ptr: 0x000001936B5A2970 ➌
ptr_a destructed. ➋
(main) ptr_a: 0x0000000000000000 ➎
```

*列表 6-15：一个使用 `Tracer` 类调查 `SimpleUniquePointers` 的程序*

首先，你动态分配一个名为 `ptr_a` 的 `Tracer`。这会打印出第一条消息 ➊。然后，你使用得到的 `Tracer` 指针来构造一个名为 `ptr_a` 的 `SimpleUniquePointer`。接下来，你使用 `ptr_a` 的 `get()` 方法来获取其 `Tracer` 的地址，并打印 ➍。然后你使用 `std::move` 将 `ptr_a` 的 `Tracer` 转交给 `consumer` 函数，这会将 `ptr_a` 移动到 `consumer_ptr` 参数中。

现在，`consumer_ptr` 拥有 `Tracer`。你使用 `consumer_ptr` 的 `get()` 方法来获取 `Tracer` 的地址，然后打印 ➌。注意这个地址与 ➍ 打印的地址相同。当 `consumer` 返回时，`consumer_ptr` 被销毁，因为它的生命周期是 `consumer` 的作用域。因此，`ptr_a` 会被析构 ➋。

请记住，`ptr_a` 已经处于一个“已移动”状态——你已经将它的 `Tracer` 移动到 `consumer`。你使用 `ptr_a` 的 `get()` 方法来说明它现在持有一个 `nullptr` ➎。

由于有了 `SimpleUniquePointer`，你就不会泄漏一个动态分配的对象；此外，因为 `SimpleUniquePointer` 仅在背后携带一个指针，所以移动语义非常高效。

**注意**

*`SimpleUniquePointer` 是对 stdlib 的 `std::unique_ptr` 的教学性实现，它是称为智能指针的 RAII 模板家族的一员。你将在第二部分中学习这些内容。*

### 模板中的类型检查

模板是类型安全的。在模板实例化过程中，编译器将模板参数粘贴到模板中。如果生成的代码不正确，编译器将不会生成该实例化。

考虑列表 6-16 中的模板函数，它对一个元素进行平方并返回结果。

```
template<typename T>
T square(T value) {
  return value * value; ➊
}
```

*列表 6-16：一个对值进行平方的模板函数*

`T` 有一个隐式要求：它必须支持乘法 ➊。

如果你尝试使用 `square`，例如使用 `char*`，编译将失败，如列表 6-17 所示。

```
template<typename T>
T square(T value) {
  return value * value;
}

int main() {
  char my_char{ 'Q' };
  auto result = square(&my_char); ➊ // Bang!
}
```

*列表 6-17：一个模板实例化失败的程序。（这个程序无法编译。）*

指针不支持乘法，因此模板初始化失败 ➊。

`square` 函数非常简单，但失败的模板初始化错误信息却不简单。在 MSVC v141 上，你会看到这个：

```
main.cpp(3): error C2296: '*': illegal, left operand has type 'char *'
main.cpp(8): note: see reference to function template instantiation 'T *square<char*>(T)' being compiled
        with
        [
            T=char *
        ]
main.cpp(3): error C2297: '*': illegal, right operand has type 'char *'
```

在 GCC 7.3 上，你会看到这个：

```
main.cpp: In instantiation of 'T square(T) [with T = char*]':
main.cpp:8:32:   required from here
main.cpp:3:16: error: invalid operands of types 'char*' and 'char*' to binary
'operator*'
   return value * value;
          ~~~~~~^~~~~~~
```

这些错误信息展示了模板初始化失败时 notoriously cryptic 的错误信息。

尽管模板实例化确保了类型安全，但检查发生在编译过程的非常晚阶段。当编译器实例化模板时，它将模板参数类型粘贴到模板中。类型插入之后，编译器尝试编译结果。如果实例化失败，编译器会在模板实例化内发出错误信息。

C++ 模板编程与*鸭子类型语言*有相似之处。鸭子类型语言（如 Python）会推迟类型检查，直到运行时。其基本哲学是，如果一个对象看起来像鸭子并且叫声像鸭子，那么它就应该是鸭子类型。不幸的是，这意味着你无法在程序执行之前判断一个对象是否支持某个特定操作。

使用模板时，直到你尝试编译它，你才知道实例化是否会成功。尽管鸭子类型语言可能会在运行时崩溃，但模板可能会在编译时崩溃。

这种情况在 C++ 社区中被认为是不可接受的，因此有一个精彩的解决方案，叫做*概念*。

### 概念

*概念* 限制模板参数，允许在实例化时而不是首次使用时进行参数检查。通过在实例化时捕获使用问题，编译器可以为你提供友好的、有用的错误代码——例如，“你尝试使用 `char*` 实例化这个模板，但该模板需要一个支持乘法的类型。”

概念允许你直接在语言中表达模板参数的要求。

不幸的是，概念尚未正式成为 C++ 标准的一部分，尽管它们已经被投票纳入 C++ 20。截止目前，GCC 6.0 及之后的版本支持概念技术规范，而微软正在积极努力在其 C++ 编译器 MSVC 中实现概念。尽管它们还不是正式标准，但出于以下几个原因，深入了解概念是值得的：

+   它们将从根本上改变你实现编译时多态性的方法。熟悉概念将带来巨大的回报。

+   它们提供了一个概念框架，用于理解在模板被误用时，你可以采取的一些临时解决方案，以获得更好的编译器错误信息。

+   它们提供了从编译时模板到接口的优秀概念桥梁，接口是实现运行时多态性的主要机制（详见第五章）。

+   如果你可以使用 GCC 6.0 或更高版本，概念*是*可用的，只需启用`-fconcepts`编译器标志。

**警告**

*C++ 20 的最终概念规范几乎肯定会与概念技术规范有所不同。本节介绍了根据概念技术规范指定的概念，以便你可以跟上。*

#### *定义一个概念*

概念是一个模板。它是一个常量表达式，涉及模板参数，在编译时评估。把概念看作是一个大的*谓词*：一个评估为`true`或`false`的函数。

如果一组模板参数符合给定概念的标准，那么在用这些参数实例化时，该概念会评估为`true`；否则，评估为`false`。当概念评估为`false`时，模板实例化将失败。

你可以使用关键字`concept`声明概念，语法与常规模板函数定义类似：

```
template<typename T1, typename T2, ...>
concept bool ConceptName() {
  --snip--
}
```

#### *类型特征*

概念验证类型参数。在概念中，你操作类型以检查其属性。你可以手动实现这些操作，也可以使用标准库中内建的类型支持库。该库包含检查类型属性的工具，这些工具统称为*类型特征*。它们可以在`<type_traits>`头文件中找到，并且属于`std`命名空间。表 6-1 列出了常用的类型特征。

**备注**

*有关标准库中可用的类型特征的详细列表，请参见 Nicolai M. Josuttis 的《C++标准库》第 2 版第 5.4 章。*

**表 6-1：** 选自`<type_traits>`头文件的类型特征

| **类型特征** | **检查模板参数是否是…** |
| --- | --- |
| `is_void` | `void` |
| `is_null_pointer` | `nullptr` |
| `is_integral` | `bool`、`char`类型、`int`类型、`short`类型、`long`类型或`long long`类型 |
| `is_floating_point` | `float`、`double`或`long double` |
| `is_fundamental` | 任何一个`is_void`、`is_null_pointer`、`is_integral`或`is_floating_point` |
| `is_array` | 数组类型；即包含方括号`[]`的类型 |
| `is_enum` | 枚举类型（`enum`） |
| `is_class` | 类类型（但不是联合类型） |
| `is_function` | 函数类型 |
| `is_pointer` | 指针；包括函数指针，但不包括类成员指针和`nullptr` |
| `is_reference` | 引用类型（包括左值和右值） |
| `is_arithmetic` | `is_floating_point`或`is_integral` |
|  `is_pod`  | 一个简单的旧数据类型；即，可以作为普通 C 中的数据类型表示的类型 |
| `is_default_constructible` | 可以默认构造；即，可以没有参数或初始化值地构造 |
| `is_constructible` | 是否可以使用给定的模板参数构造：此类型特征允许用户提供超出当前考虑类型的其他模板参数 |
| `is_copy_constructible` | 可以通过复制构造 |
| `is_move_constructible` | 可以通过移动构造 |
| `is_destructible` | 是否可以析构 |
| `is_same` | 与附加模板参数类型相同（包括`const`和`volatile`修饰符） |
| `is_invocable` | 可以使用给定的模板参数调用：此类型特征允许用户提供超出当前考虑类型的其他模板参数 |

每个类型特征都是一个模板类，接受一个模板参数，即你想要检查的类型。你可以通过模板的静态成员`value`提取结果。如果类型参数满足条件，该成员的值为`true`；否则为`false`。

考虑类型特征类`is_integral`和`is_floating_point`。它们用于检查一个类型是否是（你猜对了）整数类型或浮点类型。这两个模板都接受一个模板参数。在清单 6-18 中的示例检查了多个类型的类型特征。

```
#include <type_traits>
#include <cstdio>
#include <cstdint>

constexpr const char* as_str(bool x) { return x ? "True" : "False"; } ➊

int main() {
  printf("%s\n", as_str(std::is_integral<int>::value)); ➋
  printf("%s\n", as_str(std::is_integral<const int>::value)); ➌
  printf("%s\n", as_str(std::is_integral<char>::value)); ➍
  printf("%s\n", as_str(std::is_integral<uint64_t>::value)); ➎
  printf("%s\n", as_str(std::is_integral<int&>::value)); ➏
  printf("%s\n", as_str(std::is_integral<int*>::value)); ➐
  printf("%s\n", as_str(std::is_integral<float>::value)); ➑
}
--------------------------------------------------------------------------
True ➋
True ➌
True ➍
True ➎
False ➏
False ➐
False ➑
```

*清单 6-18：使用类型特征的程序*

清单 6-18 定义了便捷函数`as_str` ➊，用来打印布尔值，返回字符串`True`或`False`。在`main`函数中，你打印了各种类型特征实例化的结果。模板参数`int` ➋、`const int` ➌、`char` ➍和`uint64_t` ➎传递给`is_integral`时，都会返回`true`。引用类型 ➏➐ 和浮点类型 ➑ 返回`false`。

**注意**

*请记住，`printf`没有为`bool`类型提供格式说明符。与其使用整数格式说明符`%d`作为替代，清单 6-18 使用了`as_str`函数，根据`bool`的值返回字符串字面量`True`或`False`。由于这些值是字符串字面量，你可以根据需要对它们进行大小写转换*。

类型特征通常是概念的构建块，但有时你需要更多的灵活性。类型特征告诉你*什么*类型是，但有时你还必须指定模板如何使用这些类型。为此，你需要使用要求（requirements）。

#### *要求*

*要求*是对模板参数的临时约束。每个概念可以为其模板参数指定任意数量的要求。要求被编码为`requires`关键字后跟函数参数和主体的要求表达式。

一系列语法要求构成了要求表达式的主体。每个语法要求对模板参数施加约束。要求表达式的形式如下：

```
requires (arg-1, arg-2, ...➊) {
  { expression1➋ } -> return-type1➌;
  { expression2 } -> return-type2;
  --snip--
}
```

`requires`表达式接受你在`requires`关键字后面放置的参数 ➊。这些参数的类型来源于模板参数。接下来是语法要求，每个要求用`{ } ->`表示。你在每对大括号内放置一个任意表达式 ➋。这个表达式可以涉及任何数量的参数表达式。

如果实例化导致语法表达式无法编译，则该语法要求失败。假设表达式在没有错误的情况下计算，接下来的检查是该表达式的返回类型是否与箭头`->`后面的类型匹配 ➌。如果表达式结果的计算类型不能隐式转换为返回类型 ➌，则语法要求失败。

如果任何语法要求失败，`requires`表达式的求值结果为`false`。如果所有语法要求都通过，`requires`表达式的求值结果为`true`。

假设你有两种类型，`T`和`U`，你想知道是否可以使用相等`==`和不等`!=`运算符比较这两种类型的对象。编码此要求的一种方法是使用以下表达式。

```
// T, U are types
requires (T t, U u) {
  { t == u } -> bool; // syntactic requirement 1
  { u == t } -> bool; // syntactic requirement 2
  { t != u } -> bool; // syntactic requirement 3
  { u != t } -> bool; // syntactic requirement 4
}
```

`requires`表达式接受两个参数，每个参数的类型分别为`T`和`U`。`requires`表达式中的每个语法要求都是使用`t`和`u`进行`==`或`!=`比较的表达式。所有四个语法要求都强制要求返回`bool`类型的结果。任何两个满足该`requires`表达式的类型，都能保证支持`==`和`!=`的比较。

#### *从 Requires 表达式构建概念*

因为`requires`表达式在编译时求值，概念可以包含任意数量的它们。尝试构造一个防止误用`mean`的概念。清单 6-19 注释了一些之前在清单 6-10 中使用的隐式要求。

```
template<typename T>
T mean(T* values, size_t length) {
  T result{}; ➊
  for(size_t i{}; i<length; i++) {
    result ➋+= values[i];
  }
  ➌return result / length;
}
```

*清单 6-19：带有对`T`隐式要求的注释的 6-10 重新列出*

你可以看到这段代码暗示了三个要求：

+   `T`必须是默认可构造的 ➊。

+   `T`支持`operator+=` ➋。

+   将一个`T`除以一个`size_t`得到一个`T` ➌。

从这些要求中，你可以创建一个名为`Averageable`的概念，正如清单 6-20 中演示的那样。

```
template<typename T>
concept bool Averageable() {
  return std::is_default_constructible<T>::value ➊
    && requires (T a, T b) {
      { a += b } -> T; ➋
      { a / size_t{ 1 } } -> T; ➌
    };
}
```

*清单 6-20：一个`Averageable`概念。注释与要求和`mean`的主体一致。*

你使用类型特性`is_default_constructible`来确保`T`是默认可构造的 ➊，可以对两个`T`类型进行加法操作 ➋，并且能够将`T`除以`size_t` ➌并得到`T`类型的结果。

记住，概念只是谓词；你正在构建一个布尔表达式，当模板参数被支持时，它返回`true`，而当不被支持时，它返回`false`。这个概念由一个类型特性 ➊ 和一个包含两个要求表达式 ➋➌ 的`requires`组成。如果三个要求中的任何一个返回`false`，那么该概念的约束未被满足。

#### *使用概念*

声明概念比使用它们要麻烦得多。要使用一个概念，只需在`typename`关键字的位置使用该概念的名称。

例如，你可以通过`Averageable`概念重构清单 6-13，如清单 6-21 所示。

```
#include <cstddef>
#include <type_traits>

template<typename T>
concept bool Averageable() { ➊
  --snip--
}

template<Averageable➋ T>
T mean(const T* values, size_t length) {
  --snip--
}

int main() {
  const double nums_d[] { 1.0f, 2.0f, 3.0f, 4.0f };
  const auto result1 = mean(nums_d, 4);
  printf("double: %f\n", result1);

  const float nums_f[] { 1.0, 2.0, 3.0, 4.0 };
  const auto result2 = mean(nums_f, 4);
  printf("float: %f\n", result2);

  const size_t nums_c[] { 1, 2, 3, 4 };
  const auto result3 = mean(nums_c, 4);
  printf("size_t: %d\n", result3);
}
--------------------------------------------------------------------------
double: 2.500000
float: 2.500000
size_t: 2
```

*清单 6-21：使用`Averageable`重构清单 6-13*

定义`Averageable` ➊后，你只需将其替代`typename` ➋使用即可。无需进一步修改。从编译清单 6-13 生成的代码与从编译清单 6-21 生成的代码是完全相同的。

其回报是在你尝试使用一个非`Averageable`类型的`mean`时：你会在实例化时收到编译器错误。这比你从原始模板中得到的编译器错误信息要清晰得多。

看看在清单 6-22 中`mean`的实例化，在那里你“意外”尝试对`double`指针数组求平均值。

```
--snip—
int main() {
  auto value1 = 0.0;
  auto value2 = 1.0;
  const double* values[] { &value1, &value2 };
  mean(values➊, 2);
}
```

*清单 6-22：使用非`Averageable`参数的错误模板实例化*

使用`values` ➊时存在几个问题。编译器能告诉你这些问题吗？

如果没有概念，GCC 6.3 会产生清单 6-23 中显示的错误信息。

```
<source>: In instantiation of 'T mean(const T*, size_t) [with T = const
double*; size_t = long unsigned int]':
<source>:17:17:   required from here
<source>:8:12: error: invalid operands of types 'const double*' and 'const
double*' to binary 'operator+'
     result += values[i]; ➊
     ~~~~~~~^~~~~~~~~~
<source>:8:12: error:   in evaluation of 'operator+=(const double*, const
double*)'
<source>:10:17: error: invalid operands of types 'const double*' and 'size_t'
{aka 'long unsigned int'} to binary 'operator/'
   return result / length; ➋
          ~~~~~~~^~~~~~~~
```

*清单 6-23：使用 GCC 6.3 编译清单 6-22 时的错误信息*

你可能会觉得`mean`的普通用户看到这个错误信息时会非常困惑。`i` ➊是什么？为什么`const double*`会参与到除法运算中 ➋？

概念提供了更具启发性的错误信息，正如清单 6-24 所展示的那样。

```
<source>: In function 'int main()':
<source>:28:17: error: cannot call function 'T mean(const T*, size_t) [with T
= const double*; size_t = long unsigned int]'
   mean(values, 2); ➊
                 ^
<source>:16:3: note:   constraints not satisfied
 T mean(const T* values, size_t length) {
   ^~~~

<source>:6:14: note: within 'template<class T> concept bool Averageable()
[with T = const double*]'
 concept bool Averageable() {
              ^~~~~~~~~~~
<source>:6:14: note:     with 'const double* a'
<source>:6:14: note:     with 'const double* b'
<source>:6:14: note: the required expression '(a + b)' would be ill-formed ➋
<source>:6:14: note: the required expression '(a / b)' would be ill-formed ➌
```

*清单 6-24：使用 GCC 7.2 编译启用概念的清单 6-22 时的错误信息*

这个错误信息非常棒。编译器告诉你哪个参数（`values`）没有满足某个约束 ➊。然后它告诉你`values`不是`Averageable`，因为它没有满足两个必需的表达式 ➋➌。你立刻知道如何修改你的参数，以便成功实例化这个模板。

当概念被纳入 C++标准时，std 库可能会包含许多概念。概念的设计目标是程序员不必自己定义太多的概念；相反，他们应该能够在模板前缀中组合概念和临时需求。表 6-2 提供了你可能期望包含的一些概念的部分列表，这些概念借用了 Andrew Sutton 在 Origins 库中实现的概念。

**注意**

*请参见[`github.com/asutton/origin/`](https://github.com/asutton/origin/)了解更多关于 Origins 库的信息。要编译接下来的示例，你可以安装 Origins 并使用 GCC 6.0 或更高版本，并加上`-fconcepts`标志。*

**表 6-2：**Origins 库中包含的概念

| **概念** | **一种类型，其…** |
| --- | --- |
| `Conditional` | 可以显式转换为 `bool` |
| `Boolean` | 是 `Conditional` 并支持 `!`、`&&` 和 `&#124;&#124;` 布尔运算 |
| `Equality_comparable` | 支持 `==` 和 `!=` 操作，返回一个 `Boolean` |
| `Destructible` | 可以被销毁（比较 `is_destructible`） |
| `Default_constructible` | 可以默认构造（比较 `is_default_constructible`） |
| `Movable` | 支持移动语义：它必须是可移动赋值和可移动构造的（比较 `is_move_assignable`，`is_move_constructible`） |
| `Copyable` | 支持复制语义：它必须是可复制赋值和可复制构造的（比较 `is_copy_assignable`，`is_copy_constructible`） |
| `Regular` | 是默认可构造的，可复制的，并且是 `Equality_comparable` |
| `Ordered` | 是 `Regular` 且完全有序（本质上，它可以被排序） |
| `Number` | 是 `Ordered` 并支持诸如 `+`、`-`、`/` 和 `*` 等数学运算 |
| `Function` | 支持调用；也就是说，你可以调用它（比较 `is_invocable`） |
| `Predicate` | 是一个 `Function` 并返回 `bool` |
| `Range` | 可以在基于范围的 `for` 循环中进行迭代 |

有几种方法可以将约束构建到模板前缀中。如果模板参数仅用于声明函数参数的类型，你可以完全省略模板前缀：

```
return-type function-name(Concept1➊ arg-1, …) {
  --snip--
}
```

因为你使用的是概念而不是 `typename` 来定义参数的类型 ➊，所以编译器知道相关的函数是一个模板。你甚至可以在参数列表中混合使用概念和具体类型。换句话说，每当你在函数定义中使用概念时，该函数就变成了一个模板。

列表 6-25 中的模板函数接受一个 `Ordered` 元素的数组并找到最小值。

```
#include <origin/core/concepts.hpp>
size_t index_of_minimum(Ordered➊* x, size_t length) {
  size_t min_index{};
  for(size_t i{ 1 }; i<length; i++) {
    if(x[i] < x[min_index]) min_index = i;
  }
  return min_index;
}
```

*列表 6-25：使用 `Ordered` 概念的模板函数*

即使没有模板前缀，`index_of_minimum` 也是一个模板，因为 `Ordered` ➊ 是一个概念。这个模板可以像其他模板函数一样进行实例化，正如列表 6-26 中所示。

```
#include <cstdio>
#include <cstdint>
#include <origin/core/concepts.hpp>

struct Goblin{};

size_t index_of_minimum(Ordered* x, size_t length) {
  --snip--
}

int main() {
  int x1[] { -20, 0, 100, 400, -21, 5123 };
  printf("%zu\n", index_of_minimum(x1, 6)); ➊

  unsigned short x2[] { 42, 51, 900, 400 };
  printf("%zu\n", index_of_minimum(x2, 4)); ➋

  Goblin x3[] { Goblin{}, Goblin{} };
  //index_of_minimum(x3, 2); ➌ // Bang! Goblin is not Ordered.
}
--------------------------------------------------------------------------
4 ➊
0 ➋
```

*列表 6-26：一个使用列表 6-25 中 `index_of_minimum` 的例子。取消注释* ➌ *会导致编译失败。*

`int` ➊ 和 `unsigned short` ➋ 数组的实例化成功，因为这些类型是 `Ordered`（见表 6-2）。

然而，`Goblin` 类不是 `Ordered`，如果你尝试编译 ➌，模板实例化会失败。重要的是，错误信息会很有帮助：

```
error: cannot call function 'size_t index_
of_minimum(auto:1*, size_t) [with auto:1 = Goblin; size_t = long unsigned int]'
   index_of_minimum(x3, 2); // Bang! Goblin is not Ordered.
                         ^
note:   constraints not satisfied
 size_t index_of_minimum(Ordered* x, size_t length) {
        ^~~~~~~~~~~~~~~~

note: within 'template<class T> concept bool origin::Ordered() [with T =
Goblin]'
 Ordered()
```

你知道 `index_of_minimum` 的实例化失败了，问题出在 `Ordered` 概念上。

#### *特定需求表达式*

概念是一种相对重量级的机制，用于强制执行类型安全性。有时，您只需要在模板前缀中直接强制执行某些要求。您可以将 requires 表达式直接嵌入到模板定义中，以实现这一点。请考虑 清单 6-27 中的 `get_copy` 函数，它接受一个指针并安全地返回指向对象的副本。

```
#include <stdexcept>

template<typename T>
  requires➊ is_copy_constructible<T>::value ➋
T get_copy(T* pointer) {
  if (!pointer) throw std::runtime_error{ "Null-pointer dereference" };
  return *pointer;
}
```

*清单 6-27：一个具有特定要求表达式的模板函数*

模板前缀包含 `requires` 关键字 ➊，它开始了要求表达式。在这种情况下，类型特征 `is_copy_constructible` 确保 `T` 是可拷贝的 ➋。这样，如果用户错误地尝试使用指向不可拷贝对象的指针来 `get_copy`，他们会看到模板实例化失败的清晰解释。请参考 清单 6-28 中的示例。

```
#include <stdexcept>
#include <type_traits>

template<typename T>
  requires std::is_copy_constructible<T>::value
T get_copy(T* pointer) { ➊
  --snip--
}

struct Highlander {
  Highlander() = default; ➋
  Highlander(const Highlander&) = delete; ➌
};

int main() {
  Highlander connor; ➍
  auto connor_ptr = &connor; ➎
  auto connor_copy = get_copy(connor_ptr); ➏
}
--------------------------------------------------------------------------
In function 'int main()':
error: cannot call function 'T get_copy(T*) [with T = Highlander]'
   auto connor_copy = get_copy(connor_ptr);
                                         ^
note:   constraints not satisfied
 T get_copy(T* pointer) {
   ^~~~~~~~

note: 'std::is_copy_constructible::value' evaluated to false
```

*清单 6-28：使用 清单 6-27 中的 `get_copy` 模板的程序。此代码无法编译。*

`get_copy` ➊ 的定义后跟着一个 `Highlander` 类的定义，该类包含一个默认构造函数 ➋ 和一个已删除的拷贝构造函数 ➌。在 `main` 中，您初始化了一个 `Highlander` ➍，获取了它的引用 ➎，并尝试用结果实例化 `get_copy` ➏。由于 `Highlander` 只能有一个（它不可拷贝），清单 6-28 会产生一个非常清晰的错误消息。

### static_assert：前提的临时解决方案

从 C++17 开始，概念不再是标准的一部分，因此它们在不同编译器之间不一定可用。在此期间，您可以应用一个临时的解决方案：`static_assert` 表达式。这些断言在编译时进行评估。如果断言失败，编译器会发出错误，并可选地提供诊断消息。`static_assert` 的形式如下：

```
static_assert(boolean-expression, optional-message);
```

在没有概念的情况下，您可以在模板的主体中包含一个或多个 `static_assert` 表达式，以帮助用户诊断使用错误。

假设您想在不依赖概念的情况下改进 `mean` 的错误消息。您可以结合使用类型特征和 `static_assert` 来实现类似的效果，如 清单 6-29 所示。

```
#include <type_traits>

template <typename T>
T mean(T* values, size_t length) {
  static_assert(std::is_default_constructible<T>(),
    "Type must be default constructible."); ➊
  static_assert(std::is_copy_constructible<T>(),
    "Type must be copy constructible."); ➋
  static_assert(std::is_arithmetic<T>(),
    "Type must support addition and division."); ➌
  static_assert(std::is_constructible<T, size_t>(),
    "Type must be constructible from size_t."); ➍
  --snip--
}
```

*清单 6-29：使用 `static_assert` 表达式改善 清单 6-10 中 `mean` 的编译时错误。*

您会看到常见的类型特征，用于检查 `T` 是否可以默认构造 ➊ 和拷贝构造 ➋，并且您提供了错误方法以帮助用户诊断模板实例化问题。您使用 `is_arithmetic` ➌，该方法如果类型参数支持算术操作（`+，-，/` 和 `*`）则返回 `true`，以及 `is_constructible` ➍，它确定是否可以从 `size_t` 构造一个 `T`。

使用`static_assert`作为概念的代理是一种变通方法，但它被广泛使用。通过使用类型特征，你可以暂时解决问题，直到概念被纳入标准。如果你使用现代的第三方库，你会经常看到`static_assert`；如果你为他人（包括未来的自己）编写代码，考虑使用`static_assert`和类型特征。

编译器，通常程序员，也不会阅读文档。通过将要求直接嵌入代码中，你可以避免过时的文档问题。在缺乏概念的情况下，`static_assert`是一个很好的临时替代方案。

### 非类型模板参数

使用`typename`（或`class`）关键字声明的模板参数称为*类型模板参数*，它代表某种尚未指定的*类型*。另外，你可以使用*非类型模板参数*，它们代表某种尚未指定的*值*。非类型模板参数可以是以下任意类型：

+   一个整数类型

+   一个左值引用类型

+   一个指针（或指向成员的指针）类型

+   一个`std::nullptr_t`（即`nullptr`的类型）

+   一个`enum class`

使用非类型模板参数允许你在编译时将一个值注入到通用代码中。例如，你可以构建一个名为`get`的模板函数，在编译时检查数组越界访问，通过将你想访问的索引作为非类型模板参数传入。

回想一下第三章，如果你将一个数组传递给函数，它会衰变为指针。你可以改为传递数组引用，尽管它的语法比较难以接受：

```
element-type(¶m-name)[array-length]
```

例如，示例 6-30 包含一个`get`函数，它首次尝试执行带边界检查的数组访问。

```
#include <stdexcept>

int& get(int (&arr)[10]➊, size_t index➋) {
  if (index >= 10) throw std::out_of_range{ "Out of bounds" }; ➌
  return arr[index]; ➍
}
```

*示例 6-30：带有边界检查的数组元素访问函数*

`get`函数接受一个长度为 10 的`int`数组引用 ➊ 和一个要提取的`index` ➋。如果`index`超出范围，它会抛出一个`out_of_bounds`异常 ➌；否则，它会返回对应元素的引用 ➍。

你可以在三方面改进示例 6-30，这些都通过非类型模板参数实现，使得`get`函数中的值变得通用。

首先，你可以通过将`get`函数改为模板函数来放宽`arr`引用`int`数组的要求，如示例 6-31 所示。

```
#include <stdexcept>

template <typename T➊>
T&➋ get(T➌ (&arr)[10], size_t index) {
  if (index >= 10) throw std::out_of_range{ "Out of bounds" };
  return arr[index];
}
```

*示例 6-31：对示例 6-30 的重构，以接受一个通用类型的数组*

正如你在本章中所做的，你已经通过将具体类型（此处为`int`）替换为模板参数来使函数通用化 ➊➋➌。

其次，你可以通过引入一个非类型模板参数`Length`来放宽`arr`引用长度为 10 的数组的要求。示例 6-32 展示了如何做：只需声明一个`size_t Length`模板参数，并在代码中替代 10。

```
#include <stdexcept>

template <typename T, size_t Length➊>
T& get (T(&arr)[Length➋], size_t index) {
  if (index >= Length➌) throw std::out_of_range{ "Out of bounds" };
  return arr[index];
}
```

*示例 6-32：对示例 6-31 的重构，以接受一个长度为通用值的数组*

这个思想是一样的：你不是替换一个特定的类型（`int`），而是替换一个特定的整数值（`10`）➊➋➌。现在，你可以在任何大小的数组中使用这个函数。

第三，你可以通过将`size_t index`作为另一个非类型模板参数来执行编译时边界检查。这允许你用`static_assert`替换`std::out_of_range`，如示例 6-33 所示。

```
#include <cstdio>

template <size_t Index➊, typename T, size_t Length>
T& get(T (&arr)[Length]) {
  static_assert(Index < Length, "Out-of-bounds access"); ➋
  return arr[Index➌];
}

int main() {
  int fib[]{ 1, 1, 2, 0 }; ➍
  printf("%d %d %d ", get<0>(fib), get<1>(fib), get<2>(fib)); ➎
  get<3>(fib) = get<1>(fib) + get<2>(fib); ➏
  printf("%d", get<3>(fib)); ➐
  //printf("%d", get<4>(fib)); ➑
}
--------------------------------------------------------------------------
1 1 2 ➎3 ➐
```

*示例 6-33：一个使用编译时边界检查数组访问的程序*

你将`size_t`索引参数移到了一个非类型模板参数中 ➊，并用正确的名称`Index` ➌更新了数组访问。因为`Index`现在是一个编译时常量，你还将`logic_error`替换为`static_assert`，当你不小心尝试访问越界元素时，它会打印友好的信息`Out-of-bounds access` ➋。

示例 6-33 还展示了在`main`中使用`get`的示例。你首先声明了一个长度为 4 的`int`数组`fib` ➍。然后，你使用`get` ➎打印数组的前三个元素，设置第四个元素 ➏，并打印它 ➐。如果你取消注释越界访问 ➑，编译器会因为`static_assert`而生成错误。

### 变参模板

有时候，模板必须接受一个未知数量的参数。编译器在模板实例化时知道这些参数，但你希望避免为每种不同数量的参数编写许多不同的模板。这就是变参模板的存在意义。*变参模板*接受一个可变数量的参数。

你通过一个具有特殊语法的最终模板参数来表示变参模板，即`typename... arguments`。省略号表示`arguments`是一个*参数包类型*，意味着你可以在模板中声明参数包。参数包是一个接受零个或多个函数参数的模板参数。这些定义可能看起来有些抽象，因此请考虑以下基于`SimpleUniquePointer`的变参模板示例。

回想一下示例 6-14，你将一个原始指针传递给`SimpleUniquePointer`的构造函数。示例 6-34 实现了一个`make_simple_unique`函数，用于处理基础类型的构造。

```
template <typename T, typename... Arguments➊>
SimpleUniquePointer<T> make_simple_unique(Arguments... arguments➋) {
  return SimpleUniquePointer<T>{ new T{ arguments...➌ } };
}
```

*示例 6-34：实现一个`make_simple_unique`函数，以简化`SimpleUniquePointer`的使用*

你定义了参数包类型`Arguments` ➊，这声明了`make_simple_unique`为一个变参模板。这个函数将参数 ➋ 传递给模板参数`T`的构造函数 ➌。

结果是，现在你可以非常轻松地创建`SimpleUniquePointer`，即使所指向的对象有一个非默认构造函数。

**注意**

*示例 6-34 有一个略微更高效的实现。如果`arguments`是一个右值，你可以直接将其移动到`T`的构造函数中。标准库包含一个名为`std::forward`的函数，位于`<utility>`头文件中，它将检测`arguments`是左值还是右值，并分别执行复制或移动操作。有关更多信息，请参阅 Scott Meyers 的《Effective Modern C++》中的第 23 条。*

### 高级模板话题

对于日常的多态编程，模板是你最常用的工具。事实证明，模板也被广泛应用于各种高级设置，特别是在实现库、高性能程序和嵌入式系统固件时。本节概述了这一广阔领域的一些主要特征。

#### *模板特化*

要理解高级模板用法，首先必须理解*模板特化*。模板实际上不仅可以接受`concept`和`typename`参数（类型参数）。它们还可以接受基本类型，如`char`（值参数），以及其他模板。由于模板参数具有极大的灵活性，你可以根据这些参数的特性做出许多编译时决定。你可以根据这些参数的不同特性拥有不同版本的模板。例如，如果类型参数是`Ordered`而不是`Regular`，你可能能够使一个通用程序更加高效。以这种方式编程被称为*模板特化*。有关模板特化的更多信息，请参阅 ISO 标准[temp.spec]。

#### *名称绑定*

模板实例化的另一个关键组件是名称绑定。名称绑定有助于确定编译器在模板中匹配命名元素到具体实现的规则。例如，命名元素可以是模板定义的一部分、局部名称、全局名称，或者来自某个命名空间。如果你想编写大量模板代码，你需要了解绑定是如何发生的。如果你处于这种情况，请参考 David Vandevoorde 等人的《*C++ Templates: The Complete Guide*》中的第九章，“模板中的名称”，以及[temp.res]。

#### *类型函数*

*类型函数*接受类型作为参数并返回一个类型。构建概念的类型特征与类型函数密切相关。你可以将类型函数与编译时控制结构结合使用，以便在编译时进行一般计算，例如编程控制流。通常，使用这些技术进行编程被称为*模板元编程*。

#### *模板元编程*

模板元编程以生成极为巧妙且对除最强大的程序员外几乎无人能懂的代码而闻名。幸运的是，一旦概念成为 C++标准的一部分，模板元编程应该会变得更容易为我们这些普通人所理解。在那之前，请小心谨慎。对于那些希望深入了解这一主题的人，可以参考*Modern C++ Design: Generic Programming and Design Patterns Applied*（安德烈·亚历山大斯库著）和*C++ Templates: The Complete Guide*（大卫·范德沃尔德等著）。

### 模板源代码组织

每次实例化模板时，编译器必须能够生成使用该模板所需的所有代码。这意味着关于如何实例化自定义类或函数的所有信息必须在与模板实例化相同的翻译单元内可用。到目前为止，最流行的实现方法是在头文件中完全实现模板。

这种方法有一些小的 inconveniences。编译时间可能会增加，因为具有相同参数的模板可能会被多次实例化。它还减少了隐藏实现细节的能力。幸运的是，泛型编程的好处远远超过这些不便。（主要的编译器可能会尽量减少编译时间和代码重复的问题。）

头文件模板也有一些优势：

+   让其他人使用你的代码非常容易：只需要对一些头文件应用`#include`（而不是编译库，确保结果对象文件对链接器可见，等等）。

+   对于编译器来说，将仅包含头文件的模板内联是非常容易的，这可以在运行时提高代码的执行速度。

+   编译器通常可以在所有源代码都可用时做得更好，从而优化代码。

### 运行时多态与编译时多态

当你需要多态时，应该使用模板。但有时你不能使用模板，因为你直到运行时才知道与你的代码一起使用的类型。记住，模板实例化仅在你将模板的参数与类型配对时才会发生。此时，编译器可以为你实例化一个自定义类。在某些情况下，你可能无法在编译时执行这种配对，或者至少，在编译时执行配对会非常繁琐，直到程序执行时才能进行。

在这种情况下，你可以使用运行时多态。而模板是实现编译时多态的机制，运行时机制是接口。

### 总结

在本章中，你探讨了 C++中的多态。本章开始时讨论了多态是什么，以及它为何如此有用。你探索了如何通过模板在编译时实现多态。你了解了使用概念进行类型检查，然后探讨了一些高级主题，如变参模板和模板元编程。

**练习**

**6-1.** 一系列值的众数是最常出现的值。使用以下签名实现一个众数函数：`int mode(const int* values, size_t length)`。如果遇到错误情况，如输入具有多个众数或没有值，则返回零。

**6-2.** 将 `mode` 实现为模板函数。

**6-3.** 修改 `mode` 函数以接受 `Integer` 类型概念。验证 `mode` 无法使用浮动类型（如 `double`）实例化。

**6-4.** 重构 列表 6-13 中的 `mean` 函数，使其接受数组，而不是指针和长度参数。使用 列表 6-33 作为参考。

**6-5.** 使用 第五章的示例，将 `Bank` 改为接受模板参数的模板类。使用该类型参数作为账户类型，而不是 `long`。验证使用 `Bank<long>` 类时代码是否仍然有效。

**6-6.** 实现一个 `Account` 类并实例化一个 `Bank<Account>`。在 `Account` 类中实现函数以跟踪余额。

**6-7.** 将 `Account` 变为接口。实现 `CheckingAccount` 和 `SavingsAccount`。创建一个程序，包含多个支票账户和储蓄账户。使用 `Bank<Account>` 实现账户间的多个交易。

**进一步阅读**

+   *C++ 模板：完全指南*（第 2 版），由 David Vandevoorde、Nicolai M. Josuttis 和 Douglas Gregor 编写（Addison-Wesley，2017 年）

+   *有效的现代 C++：42 种方法改进你对 C++11 和 C++14 的使用* 由 Scott Meyers 编写（O'Reilly Media，2015 年）

+   *C++ 编程语言*（第 4 版），由 Bjarne Stroustrup 编写（Pearson Education，2013 年）

+   *现代 C++ 设计：通用编程与设计模式应用* 由 Andrei Alexandrescu 编写（Addison-Wesley，2001 年）
