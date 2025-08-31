## 第十一章：**函数**

*函数应该只做一件事，做好这件事，且只做这件事。*

—罗伯特·C·马丁，*《代码整洁之道》*

![Image](img/common.jpg)

本章将继续讨论函数，这些函数将代码封装成可重用的组件。现在你已经掌握了 C++基础知识，本章首先通过更加深入地讲解修饰符、说明符和返回类型来回顾函数，这些内容出现在函数声明中并专门化函数的行为。

然后你将学习重载解析以及接受可变数量的参数，接着探索函数指针、类型别名、函数对象和久负盛名的 lambda 表达式。本章的最后将介绍`std::function`，然后再次回顾`main`函数并接受命令行参数。

### 函数声明

函数声明具有以下熟悉的形式：

```
prefix-modifiers return-type func-name(arguments) suffix-modifiers;
```

你可以为函数提供多个可选的*修饰符*（或*说明符*）。修饰符会以某种方式改变函数的行为。一些修饰符出现在函数声明或定义的开头（*前缀修饰符*），而其他修饰符出现在结尾（*后缀修饰符*）。前缀修饰符出现在返回类型之前，后缀修饰符出现在参数列表之后。

没有明确的语言原因说明为什么某些修饰符作为前缀或后缀出现：因为 C++有着悠久的历史，这些特性是逐步演变而来的。

#### *前缀修饰符*

到此为止，你已经了解了几个前缀修饰符：

+   前缀`static`表示一个非类成员的函数具有内部链接，意味着该函数在此翻译单元外部不会被使用。不幸的是，这个关键字具有双重作用：如果它修饰的是一个方法（即类中的函数），它表示该函数不与类的实例化关联，而是与类本身关联（见第四章）。

+   修饰符`virtual`表示方法可以被子类重写。修饰符`override`则向编译器表明子类打算重写父类的虚函数（见第五章）。

+   修饰符`constexpr`表示函数应在编译时进行求值（见第七章）。

+   修饰符`[[noreturn]]`表示该函数不会返回（见第八章）。回想一下，这个属性有助于编译器优化你的代码。

另一个前缀修饰符是`inline`，它在优化代码时指导编译器的作用。

在大多数平台上，函数调用会编译成一系列指令，如下所示：

1.  将参数放入寄存器和调用栈中。

1.  将返回地址压入调用栈。

1.  跳转到被调用的函数。

1.  函数完成后，跳转到返回地址。

1.  清理调用栈。

这些步骤通常执行得非常迅速，并且如果你在多个地方使用一个函数，减少的二进制文件大小可能会带来显著的收益。

*内联函数*意味着将函数的内容直接复制并粘贴到执行路径中，省去了五个步骤的必要。这意味着当处理器执行你的代码时，它将立即执行函数的代码，而不是执行调用函数时所需的（适度的）程序。如果你更倾向于这种对速度的轻微提升，而不介意增加的二进制文件大小，可以使用`inline`关键字来向编译器表明这一点。`inline`关键字提示编译器的优化器将函数直接内联，而不是执行函数调用。

向函数添加`inline`不会改变其行为；它只是编译器偏好的表达方式。你必须确保如果你定义了`inline`函数，必须在所有翻译单元中都这么做。另外请注意，现代编译器通常会在适当的地方内联函数，尤其是当一个函数仅在一个翻译单元内使用时。

#### *后缀修饰符*

在本书的这一部分，你已经了解了两个后缀修饰符：

+   修饰符`noexcept`表示该函数*永远*不会抛出异常。它使得某些优化成为可能（见第四章）。

+   修饰符`const`表示该方法不会修改其类的实例，从而允许`const`引用类型调用该方法（见第四章）。

本节将探讨另外三个后缀修饰符：`final`、`override`和`volatile`。

##### final 和 override

`final`修饰符表示一个方法不能被子类重写。它实际上是`virtual`的反义词。列表 9-1 尝试重写一个`final`方法并导致编译错误。

```
#include <cstdio>

struct BostonCorbett {
  virtual void shoot() final➊ {
    printf("What a God we have...God avenged Abraham Lincoln");
  }
};

struct BostonCorbettJunior : BostonCorbett {
  void shoot() override➋ { } // Bang! shoot is final.
};

int main() {
  BostonCorbettJunior junior;
}
```

*列表 9-1：一个类尝试重写一个 final 方法（这段代码无法编译）。*

这个列表将`shoot`方法标记为`final` ➊。在继承自`BostonCorbett`的`BostonCorbettJunior`中，你尝试`override`（重写）`shoot`方法 ➋。这将导致编译错误。

你还可以将`final`关键字应用于整个类，禁止该类成为父类，正如列表 9-2 中所示。

```
#include <cstdio>

struct BostonCorbett final ➊ {
  void shoot()  {
    printf("What a God we have...God avenged Abraham Lincoln");
  }
};

struct BostonCorbettJunior : BostonCorbett ➋ { }; // Bang!

int main() {
  BostonCorbettJunior junior;
}
```

*列表 9-2：一个类尝试从一个 final 类继承（这段代码无法编译）。*

`BostonCorbett`类被标记为`final` ➊，当你尝试在`BostonCorbettJunior`中继承它时会导致编译错误 ➋。

**注意**

*`final`和`override`在技术上不是语言关键字；它们是*标识符*。与关键字不同，标识符只有在特定上下文中使用时才会获得特殊含义。这意味着你可以在程序的其他地方使用`final`和`override`作为符号名，从而导致像`virtual void final() override`这样的疯狂构造。尽量避免这么做。*

每当使用接口继承时，应该将实现类标记为 `final`，因为这个修饰符可以促使编译器执行一种叫做 *去虚拟化*（devirtualization）的优化。当虚拟调用被去虚拟化时，编译器会消除与虚拟调用相关的运行时开销。

##### volatile

回想一下 第七章，`volatile` 对象的值可以随时变化，因此编译器必须将对 `volatile` 对象的所有访问视为可见副作用，以便进行优化。`volatile` 关键字表示可以对 `volatile` 对象调用方法。这类似于 `const` 方法可以应用于 `const` 对象。结合这两个关键字，它们定义了一个方法的 *const/volatile 资格*（有时称为 *cv 资格*），如 列表 9-3 所示。

```
#include <cstdio>

struct Distillate {
 int apply() volatile ➊ {
    return ++applications;
  }
private:
  int applications{};
};

int main() {
  volatile ➋ Distillate ethanol;
  printf("%d Tequila\n", ethanol.apply()➌);
  printf("%d Tequila\n", ethanol.apply());
  printf("%d Tequila\n", ethanol.apply());
  printf("Floor!");
}
--------------------------------------------------------------------------
1 Tequila ➌
2 Tequila
3 Tequila
Floor!
```

*列表 9-3：展示如何使用 `volatile` 方法*

在这个示例中，你在 `Distillate` 类上声明了 `apply` 方法 `vola``tile` ➊。你还在 `main` 中创建了一个名为 `ethanol` 的 `volatile Distillate` ➋。由于 `apply` 方法是 `volatile` 的，你仍然可以调用它 ➌（即使 `ethanol` 是 `volatile`）。

如果你没有标记 `apply volatile` ➊，当你尝试调用它时，编译器会抛出错误 ➌。就像你不能对 `const` 对象调用非 `const` 方法一样，你不能对 `volatile` 对象调用非 `volatile` 方法。想象一下如果可以执行这样的操作会发生什么：非 `volatile` 方法是编译器优化的候选，因为如 第七章 中所述，许多种内存访问可以在不改变程序可观察副作用的情况下被优化掉。

编译器应该如何处理因使用 `volatile` 对象——它要求所有内存访问被视为可观察的副作用——来调用一个非 `volatile` 方法时产生的矛盾？编译器的回答是，将这种矛盾视为错误。

### auto 返回类型

有两种方式声明函数的返回值：

+   （主要）像之前一样，使用返回类型来引导函数声明。

+   （次要）通过使用 `auto`，让编译器推导出正确的返回类型。

和 `auto` 类型推导一样，编译器会推导出返回类型，固定运行时类型。

这个特性应该谨慎使用。因为函数定义本身就是文档，因此在可能的情况下，最好提供具体的返回类型。

### auto 和函数模板

`auto` 类型推导的主要用例是在函数模板中，其中返回类型可能依赖（以潜在复杂的方式）于模板参数。其用法如下：

```
auto my-function(arg1-type arg1, arg2-type arg2, ...) {
  // return any type and the
  // compiler will deduce what auto means
}
```

可以将 `auto` 返回类型推导语法扩展为通过箭头操作符 `->` 提供返回类型作为后缀。这样，你可以附加一个表达式，该表达式计算出函数的返回类型。其用法如下：

```
auto my-function(arg1-type arg1, arg2-type arg2, ...) -> type-expression {
  // return an object with type matching
  // the type-expression above
}
```

通常，你不会使用这种冗长的形式，但在某些情况下它非常有用。例如，这种形式的 `auto` 类型推导通常与 `decltype` 类型表达式搭配使用。`decltype` 类型表达式返回另一个表达式的结果类型。它的用法如下：

```
decltype(expression)
```

这个表达式会解析为表达式的结果类型。例如，以下 `decltype` 表达式返回 `int`，因为整数字面量 100 的类型是 `int`：

```
decltype(100)
```

在模板的泛型编程之外，`decltype` 是一种罕见的用法。

你可以结合 `auto` 返回类型推导和 `decltype` 来记录函数模板的返回类型。考虑 示例 9-4 中的 `add` 函数，它定义了一个 `add` 函数模板，用来将两个参数相加。

```
#include <cstdio>

template <typename X, typename Y>
auto add(X x, Y y) -> decltype(x + y) { ➊
  return x + y;
}
int main() {
  auto my_double = add(100., -10);
  printf("decltype(double + int) = double; %f\n", my_double); ➋

  auto my_uint = add(100U, -20);
  printf("decltype(uint + int) = uint; %u\n", my_uint); ➌

  auto my_ulonglong = add(char{ 100 }, 54'999'900ull);
  printf("decltype(char + ulonglong) = ulonglong; %llu\n", my_ulonglong); ➍
}
--------------------------------------------------------------------------
decltype(double + int) = double; 90.000000 ➋
decltype(uint + int) = uint; 80 ➌
decltype(char + ulonglong) = ulonglong; 55000000 ➍
```

*示例 9-4：使用 `decltype` 和 `auto` 返回类型推导*

`add` 函数使用 `auto` 类型推导结合 `decltype` 类型表达式 ➊。每次你用两个类型 `X` 和 `Y` 实例化模板时，编译器会评估 `decltype(X + Y)`，并确定 `add` 的返回类型。在 `main` 中，你提供了三种实例化。首先，你将一个 `double` 和一个 `int` 相加 ➋。编译器确定 `decltype(double{ 100\. } + int{ -10 })` 是一个 `double`，这就确定了该 `add` 实例化的返回类型。反过来，这也将 `my_double` 的类型设定为 `double` ➋。你还有两个其他的实例化：一个是 `unsigned int` 和 `int`（结果是 `unsigned int` ➌），另一个是 `char` 和 `unsigned long long`（结果是 `unsigned long long` ➍）。

### 重载解析

*重载解析* 是编译器在将函数调用与其正确实现匹配时执行的过程。

回顾 第四章，函数重载允许你指定具有相同名称但不同类型和可能不同参数的函数。编译器通过将函数调用中的参数类型与每个重载声明中的类型进行比较，从而选择其中的一个重载。编译器会在可能的选项中选择最佳的，如果无法选择最佳选项，它将生成编译错误。

大致而言，匹配过程如下：

1.  编译器会寻找一个精确的类型匹配。

1.  编译器会尝试使用整数和浮点数的转换来获得合适的重载（例如，从 `int` 到 `long` 或从 `float` 到 `double`）。

1.  编译器会尝试使用标准转换来进行匹配，比如将整数类型转换为浮点数，或者将指向子类的指针转换为指向父类的指针。

1.  编译器会寻找用户定义的转换。

1.  编译器会寻找一个变参函数。

### 变参函数

*变参函数*接受可变数量的参数。通常，你通过明确列出所有参数来指定函数所接受的参数数量。使用变参函数时，你可以接受任意数量的参数。变参函数 `printf` 就是一个典型的例子：你提供一个格式说明符和任意数量的参数。因为 `printf` 是变参函数，所以它接受任何数量的参数。

**注意**

*机智的 Pythonista 会立刻注意到变参函数与 `*args`/`**kwargs` 之间的概念关系。*

你通过将 `...` 放置为函数参数列表的最后一个参数来声明变参函数。当调用变参函数时，编译器会将传入的参数与声明的参数进行匹配。多余的参数将打包成 `...` 表示的变参。

你不能直接从变参中提取元素。相反，你需要使用 `<cstdarg>` 头文件中的工具函数来访问每个单独的参数。

表 9-1 列出了这些工具函数。

**表 9-1：** `<cstdarg>` 头文件中的工具函数

| **函数** | **描述** |
| --- | --- |
| `va_list` | 用于声明表示变参参数的局部变量 |
| `va_start` | 启用访问变参参数 |
| `va_end` | 用于结束对变参参数的遍历 |
| `va_arg` | 用于遍历变参参数中的每个元素 |
| `va_copy` | 创建变参参数的副本 |

工具函数的使用有些复杂，最好通过一个连贯的示例来展示。考虑 示例 9-5 中的变参 `sum` 函数，它包含一个变参参数。

```
#include <cstdio>
#include <cstdint>
#include <cstdarg>

int sum(size_t n, ...➊) {
  va_list args; ➋
  va_start(args, n); ➌
  int result{};
  while (n--) {
    auto next_element = va_arg(args, int); ➍
      result += next_element;
  }
  va_end(args); ➎
 return result;
}

int main() {
  printf("The answer is %d.", sum(6, 2, 4, 6, 8, 10, 12)); ➏
}
--------------------------------------------------------------------------
The answer is 42\. ➏
```

*示例 9-5：一个具有变参列表的 `sum` 函数*

你将 `sum` 声明为变参函数 ➊。所有变参函数必须声明一个 `va_list`。你将其命名为 `args` ➋。`va_list` 需要通过 `va_start` 初始化 ➌，后者接受两个参数。第一个参数是 `va_list`，第二个是变参参数的大小。你通过 `va_args` 函数遍历变参中的每个元素。第一个参数是 `va_list`，第二个是参数类型 ➍。遍历完成后，你通过 `va_end` 来结束遍历，传入 `va_list` 结构体 ➎。

你调用 `sum` 函数时传入七个参数：第一个是变参参数的数量（六个），后面是六个数字（2, 4, 6, 8, 10, 12）➏。

变参函数是从 C 语言继承下来的。通常，变参函数不安全，是常见的安全漏洞源。

变参函数至少存在两个主要问题：

+   变参参数不是类型安全的。（注意 `va_arg` 的第二个参数是类型。）

+   变参参数的元素数量必须单独跟踪。

编译器无法帮助你解决这些问题。

幸运的是，变参模板提供了一种更安全且性能更高的实现变参函数的方式。

### 变参模板

变参模板使你能够创建接受变参且类型相同的函数模板。它们使你能够利用模板引擎的强大功能。要声明变参模板，你需要添加一个特殊的模板参数，叫做*模板参数包*。清单 9-6 展示了它的用法。

```
template <typename...➊ Args>
return-type func-name(Args...➋ args) {
  // Use parameter pack semantics
  // within function body
}
```

*清单 9-6：一个带有参数包的模板函数*

模板参数包是模板参数列表的一部分 ➊。当你在函数模板 ➋ 中使用`Args`时，它被称为*函数参数包*。有一些特殊的操作符可以与参数包一起使用：

+   你可以使用`sizeof...(args)`来获取参数包的大小。

+   你可以使用特殊语法`other_function(args...)`调用一个函数（例如`other_function`）。这会展开参数包`args`，并允许你对参数包中的参数进行进一步处理。

#### *使用参数包编程*

不幸的是，无法直接对参数包进行索引。你必须从函数模板内部调用自己——这个过程叫做*编译时递归*——以递归地遍历参数包中的元素。

清单 9-7 展示了这一模式。

```
template <typename T, typename... Args>
void my_func(T x➊, Args...args) {
  // Use x, then recurse:
  my_func(args...); ➋
}
```

*清单 9-7：一个示范编译时递归与参数包的模板函数。与其他用法清单不同，清单中包含的省略号是字面上的。*

关键是要在参数包之前添加一个常规模板参数 ➊。每次调用`my_func`时，`x`会吸收第一个参数，其余的会打包到`args`中。要调用时，你使用`args...`构造来展开参数包 ➋。

递归需要一个停止条件，因此你添加一个没有参数的函数模板特化：

```
template <typename T>
void my_func(T x) {
  // Use x, but DON'T recurse
}
```

#### *重新审视求和函数*

考虑在清单 9-8 中作为变参模板实现的（经过大幅改进的）`sum`函数。

```
#include <cstdio>

template <typename T>
constexpr➊ T sum(T x) { ➋
    return x;
}

template <typename T, typename... Args>
constexpr➌ T sum(T x, Args... args) { ➍
    return x + sum(args...➎);
}

int main() {
  printf("The answer is %d.", sum(2, 4, 6, 8, 10, 12)); ➏
}
--------------------------------------------------------------------------
The answer is 42\. ➏
```

*清单 9-8：使用模板参数包替代`va_args`的清单 9-5 的重构版*

第一个函数 ➋ 是处理停止条件的重载；如果函数只有一个参数，你只需返回参数`x,`，因为单个元素的和就是该元素。变参模板 ➍ 遵循清单 9-7 中概述的递归模式。它从参数包`args`中去除一个参数`x`，然后返回`x`加上递归调用`sum`时展开的参数包 ➎ 的结果。由于所有这些通用编程都可以在编译时计算，所以你将这些函数标记为`constexpr` ➊➌。这种编译时计算是*主要*的优势，相较于清单 9-5，虽然它们的输出相同，但会在运行时计算结果 ➏。（既然不需要，为什么要支付运行时的代价呢？）

当你只想对一系列值（如列表 9-5 中的值）应用单一的二元运算符（如加法或减法）时，你可以使用折叠表达式而非递归。

#### *折叠表达式*

*折叠表达式*计算在参数包的所有参数上使用二元运算符的结果。折叠表达式与可变参数模板不同，但相关。它们的使用方法如下：

```
(... binary-operator parameter-pack)
```

例如，你可以使用以下折叠表达式来对名为`args`的参数包中的所有元素进行求和：

```
(... + args)
```

列表 9-9 将 9-8 重构为使用折叠表达式而非递归。

```
#include <cstdio>

template <typename... T>
constexpr auto sum(T... args) {
  return (... + args); ➊
}
int main() {
  printf("The answer is %d.", sum(2, 4, 6, 8, 10, 12)); ➋
}
--------------------------------------------------------------------------
The answer is 42\. ➋
```

*列表 9-9：将 列表 9-8 使用折叠表达式进行重构*

你通过使用折叠表达式来简化`sum`函数，而不是使用递归方法 ➊。最终结果是相同的 ➋。

### 函数指针

*函数式编程*是一种编程范式，强调函数求值和不可变数据。函数式编程中的一个主要概念是将函数作为参数传递给另一个函数。

你可以通过传递函数指针来实现这一点。函数占用内存，就像对象一样。你可以通过常规的指针机制引用这个内存地址。然而，与对象不同的是，你不能修改指向的函数。从这个角度看，函数在概念上类似于`const`对象。你可以获取函数的地址并调用它们，仅此而已。

#### *声明函数指针*

要声明一个函数指针，请使用以下丑陋的语法：

```
return-type (*pointer-name)(arg-type1, arg-type2, ...);
```

这与函数声明的外观相同，只是函数名被替换为（`*pointer-name`）。

像往常一样，你可以使用取地址符号`&`来获取函数的地址。然而，这不是必须的；你也可以直接使用函数名作为指针。

列表 9-10 展示了如何获取并使用函数指针。

```
#include <cstdio>

float add(float a, int b) {
  return a + b;
}

float subtract(float a, int b) {
  return a - b;
}

int main() {
  const float first{ 100 };
  const int second{ 20 };

  float(*operation)(float, int) {}; ➊
  printf("operation initialized to 0x%p\n", operation); ➋
 operation = &add; ➌
  printf("&add = 0x%p\n", operation); ➍
  printf("%g + %d = %g\n", first, second, operation(first, second)); ➎

  operation = subtract; ➏
  printf("&subtract = 0x%p\n", operation); ➐
  printf("%g - %d = %g\n", first, second, operation(first, second)); ➑
}
--------------------------------------------------------------------------
operation initialized to 0x0000000000000000 ➋
&add = 0x00007FF6CDFE1070 ➍
100 + 20 = 120 ➎
&subtract = 0x00007FF6CDFE10A0 ➐
100 - 20 = 80 ➑
```

*列表 9-10：一个展示函数指针的程序。（由于地址空间布局随机化，地址 ➍➐ 在运行时会有所不同。）*

这个列表展示了两个具有相同函数签名的函数，`add`和`subtract`。由于函数签名匹配，这些函数的指针类型也会匹配。你初始化一个接受`float`和`int`作为参数并返回`float`的函数指针`operation` ➊。接下来，你打印初始化后`operation`的值，它是`nullptr` ➋。

然后，你使用取地址符号将`add`的地址赋值给`operation` ➌，并打印其新地址 ➍。你调用`operation`并打印结果 ➎。

为了说明你可以重新赋值函数指针，你将`operation`赋值为`subtract`，而不使用取地址符号 ➏，打印`operation`的新值 ➐，最后打印结果 ➑。

#### *类型别名和函数指针*

类型别名为编程提供了一种简洁的方式来使用函数指针。其使用方法如下：

```
using alias-name = return-type(*)(arg-type1, arg-type2, ...)
```

例如，你可以在清单 9-10 中定义一个`operation_func`类型别名：

```
using operation_func = float(*)(float, int);
```

如果你将使用相同类型的函数指针，这非常有用；它确实可以清理代码。

### 函数调用操作符

你可以通过重载函数调用操作符`operator()()`使用户定义类型可调用或可执行。这样的类型被称为*函数类型*，函数类型的实例被称为*函数对象*。函数调用操作符允许任意组合的参数类型、返回类型和修饰符（除了`static`）。

你可能希望使用户定义类型可调用的主要原因是与期望使用函数调用操作符的代码进行互操作。你会发现许多库（如 stdlib）使用函数调用操作符作为函数对象的接口。例如，在第十九章中，你将学习如何使用`std::async`函数创建一个异步任务，它接受一个可以在单独线程上执行的任意函数对象。它使用函数调用操作符作为接口。发明`std::async`的委员会本可以要求你暴露一个比如`run`的方法，但他们选择了函数调用操作符，因为它允许通用代码使用相同的符号来调用函数或函数对象。

清单 9-11 展示了函数调用操作符的使用。

```
struct type-name {
  return-type➊ operator()➋(arg-type1 arg1, arg-type2 arg2, ...➌) {
    // Body of function-call operator
  }
}
```

*清单 9-11：函数调用操作符的使用*

函数调用操作符具有特殊的`operator()`方法名称 ➋。你声明任意数量的参数 ➌，并且你还决定适当的返回类型 ➊。

当编译器评估函数调用表达式时，它将对第一个操作数调用函数调用操作符，并将其余操作数作为参数传递。函数调用表达式的结果是调用相应的函数调用操作符的结果。

### 一个计数示例

请参阅清单 9-12 中的`CountIf`函数类型，该类型计算特定`char`在空终止字符串中的出现频率。

```
#include <cstdio>
#include <cstdint>

struct CountIf {
  CountIf(char x) : x{ x } { }➊
  size_t operator()(const char* str➋) const {
    size_t index{}➌, result{};
    while (str[index]) {
      if (str[index] == x) result++; ➍
      index++;
    }
    return result;
  }
private:
  const char x;
};

int main() {
  CountIf s_counter{ 's' }; ➎
  auto sally = s_counter("Sally sells seashells by the seashore."); ➏
  printf("Sally: %zu\n", sally);
  auto sailor = s_counter("Sailor went to sea to see what he could see.");
  printf("Sailor: %zu\n", sailor);
  auto buffalo = CountIf{ 'f' }("Buffalo buffalo Buffalo buffalo "
                                "buffalo buffalo Buffalo buffalo."); ➐
  printf("Buffalo: %zu\n", buffalo);
}
--------------------------------------------------------------------------
Sally: 7
Sailor: 3
Buffalo: 16
```

*清单 9-12：一个计算空终止字符串中字符出现次数的函数类型*

你通过使用构造函数来初始化`CountIf`对象，该构造函数接受一个`char` ➊。你可以像调用函数一样调用这个结果函数对象，传递一个空终止字符串作为参数 ➋，因为你已经实现了函数调用操作符。函数调用操作符通过`index`变量 ➌ 遍历参数`str`中的每个字符，每当字符与`x`字段匹配时，`result`变量就会递增 ➍。由于调用该函数不会修改`CountIf`对象的状态，因此你已将其标记为`const`。

在 `main` 中，你已经初始化了 `CountIf` 函数对象 `s_counter`，它将计算字母 `s` 的频率 ➎。你可以像使用函数一样使用 `s_counter` ➏。你甚至可以初始化一个 `CountIf` 对象，并直接将函数运算符作为右值对象使用 ➐。在某些场景中，这样做可能会很方便，比如你可能只需要调用该对象一次。

你可以将函数对象用作部分应用。列表 9-12 在概念上与 列表 9-13 中的 `count_if` 函数类似。

```
#include <cstdio>
#include <cstdint>

size_t count_if(char x➊, const char* str) {
  size_t index{}, result{};
  while (str[index]) {
    if (str[index] == x) result++;
    index++;
  }
  return result;
}

int main() {
  auto sally = count_if('s', "Sally sells seashells by the seashore.");
  printf("Sally: %zu\n", sally);
  auto sailor = count_if('s', "Sailor went to sea to see what he could see.");
  printf("Sailor: %zu\n", sailor);
 auto buffalo = count_if('f', "Buffalo buffalo Buffalo buffalo "
                               "buffalo buffalo Buffalo buffalo.");
  printf("Buffalo: %zu\n", buffalo);
}
--------------------------------------------------------------------------
Sally: 7
Sailor: 3
Buffalo: 16
```

*列表 9-13：模拟 列表 9-12 的自由函数*

`count_if` 函数有一个额外的参数 `x` ➊，但除此之外，它几乎与 `CountIf` 的函数运算符相同。

**注意**

*在函数式编程术语中，`CountIf` 是将 `x` 部分应用到 `count_if` 的 `partial application`。当你将一个参数部分应用到函数时，你固定了该参数的值。这样的部分应用的产物是另一个接受少一个参数的函数。*

声明函数类型通常比较冗长。你可以通过 Lambda 表达式显著减少样板代码。

### Lambda 表达式

*Lambda 表达式* 简洁地构造了无名的函数对象。函数对象隐含了函数类型，从而提供了一种快速声明函数对象的方法。Lambda 不提供任何额外的功能，只是以传统的方式声明函数类型。但当你只需要在一个特定的上下文中初始化函数对象时，它们非常方便。

#### *用法*

Lambda 表达式有五个组成部分：

+   `*captures*`：函数对象的成员变量（即部分应用的参数）

+   `*参数*`：调用函数对象所需的参数

+   `*body*`：函数对象的代码

+   `*specifiers*`：如 `constexpr`、`mutable`、`noexcept` 和 `[[noreturn]]` 等元素

+   `*返回类型*`：函数对象返回的类型

Lambda 表达式的用法如下：

```
[captures➊] (parameters➋) specifiers➌ -> return-type➍ { body➎ }
```

仅捕获和函数体是必需的，其他部分都是可选的。你将在接下来的几节中深入了解这些组件。

每个 Lambda 组件都有一个与之直接对应的函数对象。为了在函数对象（如 `CountIf`）与 Lambda 表达式之间架起桥梁，查看 列表 9-14，其中列出了来自 列表 9-12 的 `CountIf` 函数类型，并附有注释，表示 Lambda 表达式在使用时的类似部分。

```
struct CountIf {
  CountIf(char x) : x{ x } { } ➊
  size_t➍ operator()(const char* str➋) const➎ {
    --snip--➌
  }
private:
  const char x; ➋
};
```

*列表 9-14：比较 `CountIf` 类型声明与 Lambda 表达式*

您在 `CountIf` 构造函数中设置的成员变量类似于 lambda 的捕获 ➊。函数调用运算符的参数 ➋、主体 ➌ 和返回类型 ➍ 类似于 lambda 的参数、主体和返回类型。最后，修饰符可以应用于函数调用运算符 ➎ 和 lambda。（Lambda 表达式使用示例中的数字与 列表 9-14 相对应。）

#### *Lambda 参数与主体*

Lambda 表达式生成函数对象。作为函数对象，lambda 是可调用的。大多数时候，您希望在调用时让函数对象接受参数。

lambda 的主体就像一个函数的主体：所有的参数都具有函数作用域。

您使用与函数相同的语法来声明 lambda 的参数和主体。

例如，以下 lambda 表达式生成一个函数对象，该对象将对其 `int` 参数进行平方操作：

```
[](int x) { return x*x; }
```

该 lambda 接受一个 `int x`，并在 lambda 的主体内使用它进行平方操作。

列表 9-15 使用了三个不同的 lambda 表达式来转换数组 `1, 2, 3`。

```
#include <cstdio>
#include <cstdint>

template <typename Fn>
void transform(Fn fn, const int* in, int* out, size_t length) { ➊
  for(size_t i{}; i<length; i++) {
    out[i] = fn(in[i]); ➋
  }
}

int main() {
  const size_t len{ 3 };
  int base[]{ 1, 2, 3 }, a[len], b[len], c[len];
  transform([](int x) { return 1; }➌, base, a, len);
  transform([](int x) { return x; }➍, base, b, len);
  transform([](int x) { return 10*x+5; }➎, base, c, len);
  for (size_t i{}; i < len; i++) {
    printf("Element %zu: %d %d %d\n", i, a[i], b[i], c[i]);
  }
}
--------------------------------------------------------------------------
Element 0: 1 1 15
Element 1: 1 2 25
Element 2: 1 3 35
```

*列表 9-15：三个 lambda 表达式和一个 transform 函数*

`transform` 模板函数 ➊ 接受四个参数：一个函数对象 `fn`，一个 `in` 数组和一个 `out` 数组，以及这些数组的相应 `length`。在 `transform` 中，您会对 `in` 的每个元素调用 `fn`，并将结果赋值给 `out` 的相应元素 ➋。

在 `main` 中，您声明了一个 `base` 数组 `1, 2, 3`，它将作为 `in` 数组使用。在同一行中，您还声明了三个未初始化的数组 `a, b` 和 `c`，它们将作为 `out` 数组使用。第一次调用 `transform` 时传递了一个始终返回 1 的 lambda `([](int x) { return 1; })` ➌，结果被存储在 `a` 中。（注意，lambda 不需要名字！）第二次调用 `transform ([](int x) { return x; })` 简单地返回其参数 ➍，结果被存储在 `b` 中。第三次调用 `transform` 时，lambda 将参数乘以 10 并加上 5 ➎。结果被存储在 `c` 中。然后，您将输出打印到一个矩阵中，其中每一列展示了在每种情况下应用于不同 lambda 的转换。

请注意，您将 `transform` 声明为模板函数，这使得您可以使用任何函数对象重复使用它。

#### *默认参数*

您可以为 lambda 提供默认参数。默认的 lambda 参数行为与默认的函数参数相同。调用者可以为默认参数指定值，在这种情况下，lambda 使用调用者提供的值。如果调用者没有指定值，lambda 则使用默认值。

列表 9-16 展示了默认参数的行为。

```
#include <cstdio>

int main() {
  auto increment = [](auto x, int y = 1➊) { return x + y; };
  printf("increment(10)    = %d\n", increment(10)); ➋
  printf("increment(10, 5) = %d\n", increment(10, 5)); ➌
}
--------------------------------------------------------------------------
increment(10)    = 11 ➋
increment(10, 5) = 15 ➌
```

*列表 9-16：使用默认的 lambda 参数*

增量 lambda 有两个参数，`x` 和 `y`。但 `y` 参数是可选的，因为它具有默认参数 1 ➊。如果你在调用函数时没有为 `y` 指定参数 ➋，则增量返回 `1 + x`。如果你确实为 `y` 提供了一个参数 ➌，则使用该值。

#### *通用 Lambda*

通用 lambda 是 lambda 表达式模板。对于一个或多个参数，你可以指定 `auto` 而不是具体类型。这些 `auto` 类型将成为模板参数，意味着编译器会为该 lambda 创建一个自定义实例化。

列表 9-17 演示了如何将通用 lambda 分配给一个变量，然后在两个不同的模板实例化中使用该 lambda。

```
#include <cstdio>
#include <cstdint>

template <typename Fn, typename T➊>
void transform(Fn fn, const T* in, T* out, size_t len) {
  for(size_t i{}; i<len; i++) {
    out[i] = fn(in[i]);
  }
}

int main() {
  constexpr size_t len{ 3 };
  int base_int[]{ 1, 2, 3 }, a[len]; ➋
  float base_float[]{ 10.f, 20.f, 30.f }, b[len]; ➌
  auto translate = [](auto x) { return 10 * x + 5; }; ➍
  transform(translate, base_int, a, l); ➎
  transform(translate, base_float, b, l); ➏

  for (size_t i{}; i < l; i++) {
    printf("Element %zu: %d %f\n", i, a[i], b[i]);
  }
}
--------------------------------------------------------------------------
Element 0: 15 105.000000
Element 1: 25 205.000000
Element 2: 35 305.000000
```

*列表 9-17：使用通用 lambda*

你向 `transform` 添加了第二个模板参数 ➊，你用它作为 `in` 和 `out` 的指向类型。这使你可以将 transform 应用于任何类型的数组，而不仅仅是 `int` 类型的数组。为了测试升级后的 transform 模板，你声明了两个具有不同指向类型的数组：`int` ➋ 和 `float` ➌。（回想一下第三章，`10.f` 中的 `f` 表示一个 `float` 字面量。）接下来，你将一个通用的 lambda 表达式赋值给 `translate` ➍。这使你可以在每次实例化 transform 时使用相同的 lambda：当你用 `base_int` ➎ 和 `base_float` ➏ 进行实例化时。

如果没有通用 lambda，你将需要像下面这样显式声明参数类型：

```
--snip–
  transform([](int x) { return 10 * x + 5; }, base_int, a, l); ➎
  transform([](double x) { return 10 * x + 5; }, base_float, b, l); ➏
```

到目前为止，你一直依赖编译器推断 lambda 的返回类型。这对于通用 lambda 尤其有用，因为通常 lambda 的返回类型会依赖于其参数类型。但是，如果你愿意，你也可以显式声明返回类型。

#### *Lambda 返回类型*

编译器会为你推断 lambda 的返回类型。要接管编译器的推断，你可以使用箭头 `->` 语法，如下所示：

```
[](int x, double y) -> double { return x + y; }
```

这个 lambda 表达式接受一个 `int` 和一个 `double`，并返回一个 `double`。

你还可以使用 `decltype` 表达式，这在使用通用 lambda 时非常有用。例如，考虑以下 lambda：

```
[](auto x, double y) -> decltype(x+y) { return x + y; }
```

在这里，你显式声明 lambda 的返回类型为将 `x` 加到 `y` 后得到的类型。

你很少需要显式指定 lambda 的返回类型。

一个更常见的需求是你必须在调用之前将一个对象注入到 lambda 中。这就是 lambda 捕获的作用。

#### *Lambda 捕获*

*Lambda 捕获*将对象注入到 lambda 中。注入的对象有助于修改 lambda 的行为。

通过在括号`[]`内指定捕获列表来声明 lambda 的捕获。捕获列表位于参数列表之前，可以包含任意数量的逗号分隔的参数。然后，在 lambda 的主体内使用这些参数。

一个 lambda 可以按引用捕获或按值捕获。默认情况下，lambda 按值捕获。

lambda 的捕获列表类似于函数类型的构造函数。清单 9-18 将清单 9-12 中的 `CountIf` 改写为 lambda `s_counter`。

```
#include <cstdio>
#include <cstdint>

int main() {
  char to_count{ 's' }; ➊
  auto s_counter = to_count➋ {
    size_t index{}, result{};
    while (str[index]) {
      if (str[index] == to_count➌) result++;
      index++;
    }
    return result;
  };
  auto sally = s_counter("Sally sells seashells by the seashore."➍);
  printf("Sally: %zu\n", sally);
  auto sailor = s_counter("Sailor went to sea to see what he could see.");
  printf("Sailor: %zu\n", sailor);
}
--------------------------------------------------------------------------
Sally: 7
Sailor: 3
```

*清单 9-18：将清单 9-12 中的 `CountIf` 改写为 lambda*

你初始化一个名为 `to_count` 的 `char` 类型变量，赋值为字母 `s` ➊。接下来，你在分配给 `s_counter` 的 lambda 表达式中捕获 `to_count` ➋。这样，`to_count` 就可以在 lambda 表达式的主体内使用 ➌。

要通过引用捕获一个元素，而不是通过值捕获，可以在捕获对象的名称前加上与号 `&`。清单 9-19 在 `s_counter` 中添加了一个引用捕获，使其在 lambda 调用中保持累积计数。

```
#include <cstdio>
#include <cstdint>

int main() {
  char to_count{ 's' };
  size_t tally{};➊
  auto s_counter = to_count, &tally➋ {
    size_t index{}, result{};
    while (str[index]) {
      if (str[index] == to_count) result++;
      index++;
    }
    tally += result;➌
    return result;
  };
  printf("Tally: %zu\n", tally); ➍
  auto sally = s_counter("Sally sells seashells by the seashore.");
  printf("Sally: %zu\n", sally);
  printf("Tally: %zu\n", tally); ➎
  auto sailor = s_counter("Sailor went to sea to see what he could see.");
  printf("Sailor: %zu\n", sailor);
 printf("Tally: %zu\n", tally); ➏
}
--------------------------------------------------------------------------
Tally: 0 ➍
Sally: 7
Tally: 7 ➎
Sailor: 3
Tally: 10 ➏
```

*清单 9-19：在 lambda 中使用引用捕获*

你将计数器变量 `tally` 初始化为零 ➊，然后 `s_counter` lambda 通过引用捕获 `tally`（注意与号 `&`） ➋。在 lambda 的主体中，你添加一条语句，在每次调用时通过 `result` 增加 `tally`，然后返回 ➌。结果是，无论你调用多少次 lambda，`tally` 都会跟踪总计数。在第一次调用 `s_counter` 之前，你打印 `tally` 的值 ➍（此时为零）。当你用 `Sally sells seashells by the seashore.` 调用 `s_counter` 后，`tally` 的值为 7 ➎。最后一次调用 `s_counter`，传入 `Sailor went to sea to see what he could see.` 时返回 3，因此 `tally` 的值为 7 + 3 = 10 ➏。

##### 默认捕获

到目前为止，你需要通过名称捕获每个元素。有时，这种捕获方式被称为*命名捕获*。如果你懒得一个个捕获，可以通过*默认捕获*来捕获 lambda 中所有使用的自动变量。要在捕获列表中指定值捕获，使用单一的等号 `=`。要指定引用捕获，使用单一的与号 `&`。

例如，你可以将清单 9-19 中的 lambda 表达式“简化”，通过引用执行默认捕获，如清单 9-20 中所示。

```
--snip--
  auto s_counter = &➊ {
    size_t index{}, result{};
    while (str[index]) {
      if (str[index] == to_count➋) result++;
      index++;
    }
    tally➌ += result;
    return result;
  };
--snip--
```

*清单 9-20：通过引用的默认捕获简化 lambda 表达式*

你通过➊指定默认引用捕获，这意味着 lambda 表达式体内的任何自动变量都会通过引用捕获。这里有两个变量：`to_count` ➋ 和 `tally` ➌。

如果你编译并运行重构后的清单，你将获得相同的输出。然而，请注意，`to_count` 现在是通过引用捕获的。如果你在 lambda 表达式体内不小心修改了它，变化会影响到所有 lambda 调用以及 `main` 中的 `to_count`（它是一个自动变量）。

如果你改为使用值捕获，会发生什么呢？你只需要将捕获列表中的 `=` 改为 `&`，如清单 9-21 中所示。

```
--snip--
  auto s_counter = =➊ {
    size_t index{}, result{};
    while (str[index]) {
      if (str[index] == to_count➋) result++;
      index++;
    }
    tally➌ += result;
    return result;
  };
--snip--
```

*清单 9-21：将清单 9-20 修改为按值捕获而不是按引用捕获（此代码无法编译。）*

你将默认捕获更改为按值捕获 ➊。`to_count`的捕获不受影响 ➋，但尝试修改`tally`会导致编译错误 ➌。你不能修改按值捕获的变量，除非你在 lambda 表达式中添加`mutable`关键字。`mutable`关键字允许你修改按值捕获的变量，这包括调用该对象的非`const`方法。

清单 9-22 添加了`mutable`修饰符，并具有默认的按值捕获。

```
#include <cstdio>
#include <cstdint>

int main() {
  char to_count{ 's' };
  size_t tally{};
  auto s_counter = =➊ mutable➋ {
    size_t index{}, result{};
    while (str[index]) {
      if (str[index] == to_count) result++;
      index++;
    }
    tally += result;
    return result;
  };
  auto sally = s_counter("Sally sells seashells by the seashore.");
  printf("Tally: %zu\n", tally); ➌
  printf("Sally: %zu\n", sally);
  printf("Tally: %zu\n", tally); ➍
  auto sailor = s_counter("Sailor went to sea to see what he could see.");
  printf("Sailor: %zu\n", sailor);
 printf("Tally: %zu\n", tally); ➎
}
--------------------------------------------------------------------------
Tally: 0
Sally: 7
Tally: 0
Sailor: 3
Tally: 0
```

*清单 9-22：一个使用默认按值捕获的`mutable` lambda 表达式*

你通过值声明了默认捕获 ➊，并使得 lambda 表达式` s_counter`成为`mutable` ➋。每次打印`tally` ➌➍➎时，你都得到零值。为什么呢？

因为`tally`是按值复制的（通过默认捕获），lambda 表达式中的`tally`本质上是一个完全不同的变量，只是恰好有相同的名字。对 lambda 表达式中`tally`的修改不会影响`main`中的自动`tally`变量。`main()`中的`tally`被初始化为零，并且从未被修改。

你也可以将默认捕获与命名捕获混合使用。例如，你可以使用以下方式，通过引用进行默认捕获，并通过值复制`to_count`：

```
  auto s_counter = &➊,to_count➋ {
    --snip--
  };
```

这指定了通过引用进行默认捕获 ➊，并通过值捕获`to_count` ➋。

尽管执行默认捕获看起来像是一种简单的捷径，但最好避免使用它。明确声明捕获要比使用默认捕获更好。如果你发现自己在说“我就使用默认捕获，因为变量太多了，不想一一列出”，那么你可能需要重构代码。

##### 捕获列表中的初始化表达式

有时你希望在捕获列表中初始化一个全新的变量。也许重命名一个捕获的变量可以让 lambda 表达式的意图更加清晰。或者你可能想把一个对象传入 lambda 中，因此需要初始化一个变量。

要使用初始化表达式，只需声明新变量的名称，后跟等号以及你想初始化变量的值，正如清单 9-23 所演示的那样。

```
  auto s_counter = &tally➊,my_char=to_count➋ {
    size_t index{}, result{};
    while (str[index]) {
      if (str[index] == my_char➌) result++;
    --snip--
  };
```

*清单 9-23：在 lambda 捕获中使用初始化表达式*

捕获列表包含一个简单的命名捕获，你通过引用捕获了`tally` ➊。lambda 表达式还按值捕获了`to_count`，但是你选择使用变量名`my_char`来代替 ➋。当然，你需要在 lambda 表达式内部使用`my_char`而不是`to_count` ➌。

**注意**

*捕获列表中的初始化表达式也被称为初始化捕获（init capture）。*

##### 捕获 this

有时 lambda 表达式包含一个外部类。你可以使用`[*this]`或`[this]`来分别通过值或通过引用捕获外部对象（由`this`指向）。

Listing 9-24 实现了一个 `LambdaFactory`，它生成计数的 `lambda` 并跟踪 `tally`。

```
#include <cstdio>
#include <cstdint>

struct LambdaFactory {
  LambdaFactory(char in) : to_count{ in }, tally{} { }
  auto make_lambda() { ➊
    return this➋ {
      size_t index{}, result{};
      while (str[index]) {
        if (str[index] == to_count➌) result++;
        index++;
      }
      tally➍ += result;
      return result;
    };
  }
  const char to_count;
  size_t tally;
};

int main() {
  LambdaFactory factory{ 's' }; ➎
  auto lambda = factory.make_lambda(); ➏
  printf("Tally: %zu\n", factory.tally);
  printf("Sally: %zu\n", lambda("Sally sells seashells by the seashore."));
  printf("Tally: %zu\n", factory.tally);
  printf("Sailor: %zu\n", lambda("Sailor went to sea to see what he could see."));
  printf("Tally: %zu\n", factory.tally);
}
--------------------------------------------------------------------------
Tally: 0
Sally: 7
Tally: 7
Sailor: 3
Tally: 10
```

*Listing 9-24：一个 `LambdaFactory` 示例，展示了如何使用 `this` 捕获*

`LambdaFactory` 构造函数接受一个字符并使用它初始化 `to_count` 字段。`make_lambda` ➊ 方法展示了如何按引用捕获 `this` ➋ 并在 `lambda` 表达式中使用 `to_count` ➌ 和 `tally` ➍ 成员变量。

在 `main` 中，你初始化了一个 `factory` ➎ 并使用 `make_``lambda` 方法 ➏ 创建了一个 `lambda`。输出与 Listing 9-19 相同，因为你按引用捕获了 `this`，并且 `tally` 的状态在每次调用 `lambda` 时都会持续。

##### 澄清示例

捕获列表有很多种可能性，但一旦你掌握了基础——按值和按引用捕获——就不会有太多意外。Table 9-2 提供了一些简短的澄清示例，供你将来参考。

**表 9-2：** Lambda 捕获列表的澄清示例

| **捕获列表** | **含义** |
| --- | --- |
| `[&]` | 默认按引用捕获 |
| `[&,i]` | 默认按引用捕获；按值捕获 `i` |
| `[=]` | 默认按值捕获 |
| `[=,&i]` | 默认按值捕获；按引用捕获 `i` |
| `[i]` | 按值捕获 `i` |
| `[&i]` | 按引用捕获 `i` |
| `[i,&j]` | 按值捕获 `i`；按引用捕获 `j` |
| `[i=j,&k]` | 按值捕获 `j` 为 `i`；按引用捕获 `k` |
| `[this]` | 按引用捕获 `enclosing object` |
| `[*this]` | 按值捕获 `enclosing object` |
| `[=,*this,i,&j]` | 默认按值捕获；按值捕获 `this` 和 `i`；按引用捕获 `j` |

#### *constexpr Lambda 表达式*

所有的 `lambda` 表达式都是 `constexpr`，只要该 `lambda` 可以在编译时调用。你可以选择明确声明 `constexpr`，如下所示：

```
[] (int x) constexpr { return x * x; }
```

如果你希望确保 `lambda` 满足所有 `constexpr` 要求，则应将其标记为 `constexpr`。从 C++17 开始，这意味着不能进行动态内存分配，不能调用非 `constexpr` 函数等。标准委员会计划在每次发布中放宽这些限制，因此如果你编写了大量使用 `constexpr` 的代码，务必复习最新的 `constexpr` 约束。

### std::function

有时你只是想要一个统一的容器来存储可调用对象。`<functional>` 头文件中的 `std::function` 类模板是一个多态封装器，封装了一个可调用对象。换句话说，它是一个通用的函数指针。你可以将静态函数、函数对象或 `lambda` 存储到一个 `std::function` 中。

**注意**

*`*function*` 类在标准库中。我们提前展示它，因为它自然地适应了这个场景。*

使用 `functions`，你可以：

+   在调用者不需要知道函数实现的情况下调用

+   赋值、移动和复制

+   具有空状态，类似于`nullptr`

#### *声明函数*

要声明一个`function`，必须提供一个包含可调用对象原型的单一模板参数：

```
std::function<return-type(arg-type-1, arg-type-2, etc.)>
```

`std::function`类模板有多个构造函数。默认构造函数以空模式构造`std::function`，意味着它不包含任何可调用对象。

##### 空函数

如果你调用一个没有包含对象的`std::function`，`std::function`将抛出一个`std::bad_function_call`异常。请参阅 Listing 9-25。

```
#include <cstdio>
#include <functional>

int main() {
    std::function<void()> func; ➊
    try {
        func(); ➋
    } catch(const std::bad_function_call& e) {
        printf("Exception: %s", e.what()); ➌
    }
}
--------------------------------------------------------------------------
Exception: bad function call ➌
```

*Listing 9-25：默认`std::function`构造函数和`std::bad_function_call`异常*

你使用默认构造函数构造了一个`std::function` ➊。模板参数`void()`表示一个不接受参数并返回`void`的函数。因为你没有给`func`赋值一个可调用对象，所以它处于空状态。当你调用`func` ➋时，它抛出一个`std::bad_function_call`异常，你捕获并打印出来 ➌。

##### 将可调用对象赋给函数

要将可调用对象赋给`function`，你可以使用`function`的构造函数或赋值运算符，如 Listing 9-26 所示。

```
#include <cstdio>
#include <functional>

void static_func() { ➊
  printf("A static function.\n");
}

int main() {
  std::function<void()> func { [] { printf("A lambda.\n"); } }; ➋
  func(); ➌
  func = static_func; ➍
  func(); ➎
}
--------------------------------------------------------------------------
A lambda. ➌
A static function. ➎
```

*Listing 9-26：使用`function`的构造函数和赋值运算符*

你声明了一个静态函数`static_func`，它不接受任何参数并返回`void` ➊。在`main`函数中，你创建了一个名为`func`的函数 ➋。模板参数表示`func`包含的可调用对象不接受任何参数并返回`void`。你用一个打印消息`A lambda`的 lambda 表达式初始化了`func`。然后你立即调用`func` ➌，它调用了包含的 lambda 并打印了预期的消息。接下来，你将`static_func`赋值给`func`，这替换了你在构造时赋给它的 lambda ➍。然后你调用`func`，它调用了`static_func`而不是 lambda，因此你看到打印出了`A static function.` ➎。

#### *扩展示例*

你可以用可调用对象构造一个`function`，只要该对象支持由`function`的模板参数所隐含的函数语义。

Listing 9-27 使用了一个`std::function`实例数组，并将其填充了一个静态函数（用于计数空格）、一个来自 Listing 9-12 的`CountIf`函数对象，以及一个计算字符串长度的 lambda。

```
#include <cstdio>
#include <cstdint>
#include <functional>

struct CountIf {
 --snip--
};

size_t count_spaces(const char* str) {
  size_t index{}, result{};
  while (str[index]) {
    if (str[index] == ' ') result++;
    index++;
  }
  return result;
}

std::function➊<size_t(const char*)➋> funcs[]{
  count_spaces, ➌
  CountIf{ 'e' }, ➍
  [](const char* str) { ➎
    size_t index{};
    while (str[index]) index++;
    return index;
  }
};

auto text = "Sailor went to sea to see what he could see.";

int main() {
  size_t index{};
  for(const auto& func : funcs➏) {
    printf("func #%zu: %zu\n", index++, func(text)➐);
  }
}
--------------------------------------------------------------------------
func #0: 9 ➌
func #1: 7 ➍
func #2: 44 ➎
```

*Listing 9-27：使用`std::function`数组遍历具有不同底层类型的统一可调用对象集合*

你声明了一个名为`funcs`的`std::function`数组 ➊，它具有静态存储持续时间。模板参数是一个接受`const char*`并返回`size_t`的函数原型 ➋。在`funcs`数组中，你传入了一个静态函数指针 ➌，一个函数对象 ➍，以及一个 lambda ➎。在`main`函数中，你使用基于范围的`for`循环遍历`funcs`中的每个函数 ➏。你将文本`Sailor went to sea to see what he could see.`传递给每个`func`，并打印结果。

注意，从`main`的角度来看，`funcs`中的所有元素都是相同的：你只需要用一个以空字符结尾的字符串来调用它们，并返回一个`size_t` ➐。

**注意**

*使用`function`可能会带来运行时开销。出于技术原因，`function`可能需要进行动态分配以存储可调用对象。编译器也很难优化掉`function`调用，因此你通常会遭遇间接函数调用。间接函数调用需要额外的指针解引用*。

### 主函数和命令行

所有 C++程序必须包含一个名为`main`的全局函数。这个函数被定义为程序的入口点，即程序启动时调用的函数。程序在启动时可以接受任何数量的环境提供的参数，这些参数称为*命令行参数*。

用户通过命令行参数向程序传递信息，以定制程序行为。当你执行命令行程序时，你可能已经使用过此功能，例如在执行`copy`（在 Linux 中为`cp`）命令时：

```
$ copy file_a.txt file_b.txt
```

当调用这个命令时，你指示程序通过将这些值作为命令行参数传递，将`file_a.txt`复制到`file_b.txt`。就像你可能习惯的命令行程序一样，你可以将值作为命令行参数传递给 C++程序。

你可以通过如何声明`main`来选择你的程序是否处理命令行参数。

#### *三个主要的重载*

你可以通过向`main`声明添加参数来访问命令行参数。

`main`有三种有效的重载形式，如清单 9-28 所示。

```
int main(); ➊
int main(int argc, char* argv[]); ➋
int main(int argc, char* argv[], impl-parameters); ➌
```

*清单 9-28：`main`的有效重载*

第一个重载 ➊ 不接受任何参数，这就是你在本书中迄今为止使用`main()`的方式。如果你想忽略程序提供的任何参数，使用这种形式。

第二个重载 ➋ 接受两个参数，`argc`和`argv`。第一个参数`argc`是一个非负数，对应于`argv`中元素的数量。环境会自动计算这个值：你不需要为`argc`提供元素数量。第二个参数`argv`是一个指向以空字符结尾的字符串的指针数组，对应于从执行环境传递的一个参数。

第三个重载 ➌：是第二个重载 ➋：的扩展，它接受任意数量的额外实现参数。这样，目标平台可以向程序提供一些附加参数。在现代桌面环境中，实现参数并不常见。

通常，操作系统会将程序可执行文件的完整路径作为第一个命令行参数传递。这种行为取决于你的操作环境。在 macOS、Linux 和 Windows 上，执行文件的路径是第一个参数。该路径的格式依赖于操作系统。（第十七章深入讨论了文件系统。）

#### *探索程序参数*

让我们构建一个程序，探索操作系统如何将参数传递给你的程序。清单 9-29 打印命令行参数的数量，然后逐行打印每个参数的索引和值。

```
#include <cstdio>
#include <cstdint>

int main(int argc, char** argv) { ➊
  printf("Arguments: %d\n", argc); ➋
  for(size_t i{}; i<argc; i++) {
    printf("%zu: %s\n", i, argv[i]); ➌
  }
}
```

*清单 9-29：一个打印命令行参数的程序。将此程序编译为`list_929`。*

你使用`argc`/`argv`重载来声明`main`，这使得命令行参数可以传递给你的程序 ➊。首先，通过`argc`打印命令行参数的数量 ➋。然后，你遍历每个参数，打印它的索引和值 ➌。

让我们看看一些示例输出（在 Windows 10 x64 上）。这是一次程序调用：

```
$ list_929 ➊
Arguments: 1 ➋
0: list_929.exe ➌
```

在这里，除了程序的名称`list_929` ➊之外，你没有提供其他命令行参数。（根据你编译清单的方式，你应该将此替换为你的可执行文件的名称。）在一台 Windows 10 x64 机器上，结果是程序接收到一个参数 ➋，即可执行文件的名称 ➌。

这里是另一次调用：

```
$ list_929 Violence is the last refuge of the incompetent. ➊
Arguments: 9
0: list_929.exe
1: Violence
2: is
3: the
4: last
5: refuge
6: of
7: the
8: incompetent.
```

在这里，你提供了额外的程序参数：`Violence is the last refuge of the incompetent.` ➊。从输出中可以看出，Windows 将命令行按空格拆分，结果是总共有九个参数。

在主要的桌面操作系统中，你可以通过将短语用引号括起来来强制操作系统将其视为单一参数，如下所示：

```
$ list_929 "Violence is the last refuge of the incompetent."
Arguments: 2
0: list_929.exe
1: Violence is the last refuge of the incompetent.
```

#### *一个更复杂的例子*

现在你已经了解了如何处理命令行输入，接下来我们考虑一个更复杂的例子。*直方图*是一种显示分布相对频率的图示。让我们构建一个程序，计算命令行参数中字母分布的直方图。

从两个辅助函数开始，这两个函数判断给定的`char`是否是大写字母或小写字母：

```
constexpr char pos_A{ 65 }, pos_Z{ 90 }, pos_a{ 97 }, pos_z{ 122 };
constexpr bool within_AZ(char x) { return pos_A <= x && pos_Z >= x; } ➊
constexpr bool within_az(char x) { return pos_a <= x && pos_z >= x; } ➋
```

`pos_A, pos_Z, pos_a`和`pos_z`常量分别包含字母 A、Z、小写字母 a 和 z 的 ASCII 值（参见表 2-4 中的 ASCII 表）。`within_AZ`函数通过判断某个`char x`的值是否介于`pos_A`和`pos_Z`之间（包含边界）来确定它是否是大写字母 ➊。`within_az`函数对小写字母执行相同的操作 ➋。

现在你已经有了一些处理命令行的 ASCII 数据的元素，让我们构建一个`AlphaHistogram`类，它可以接受命令行元素并存储字符频率，如清单 9-30 所示。

```
struct AlphaHistogram {
  void ingest(const char* x); ➊
  void print() const; ➋
private:
  size_t counts[26]{}; ➌
};
```

*清单 9-30：一个接受命令行元素的`AlphaHistogram`*

`AlphaHistogram`将把每个字母的频率存储在`counts`数组中 ➌。每当构造一个`AlphaHistogram`时，这个数组会初始化为零。`ingest`方法将接受一个以空字符结束的字符串并适当地更新`counts` ➊。然后，`print`方法将显示存储在`counts`中的直方图信息 ➋。

首先，考虑清单 9-31 中`ingest`方法的实现。

```
void AlphaHistogram::ingest(const char* x) {
  size_t index{}; ➊
  while(const auto c = x[index]) { ➋
    if (within_AZ(c)) counts[c - pos_A]++; ➌
    else if (within_az(c)) counts[c - pos_a]++; ➍
    index++; ➎
  }
}
```

*清单 9-31：`ingest` 方法的实现*

因为 `x` 是一个以 null 结尾的字符串，你事先不知道它的长度。所以，你初始化一个 `index` 变量 ➊，并使用 `while` 循环一次提取一个 `char c` ➋。当 `c` 为 null 时，循环终止，这意味着字符串的结束。在循环内部，你使用 `within_AZ` 辅助函数判断 `c` 是否为大写字母 ➌。如果是，你将 `pos_A` 从 `c` 中减去，这样就能将大写字母标准化到 0 到 25 的区间，以便与 `counts` 对应。对于小写字母，你使用 `within_az` 辅助函数 ➍ 进行同样的检查，并在 `c` 为小写字母时更新 `counts`。如果 `c` 既不是大写字母也不是小写字母，`counts` 不受影响。最后，在继续循环前，你递增 `index` ➎。

现在，考虑如何 `打印` `counts`，如 清单 9-32 所示。

```
void AlphaHistogram::print() const {
  for(auto index{ pos_A }; index <= pos_Z; index++) { ➊
    printf("%c: ", index); ➋
    auto n_asterisks = counts[index - pos_A]; ➌
    while (n_asterisks--) printf("*"); ➍
    printf("\n"); ➎
  }
}
```

*清单 9-32：`print` 方法的实现*

为了打印直方图，你需要循环遍历从 A 到 Z 的每个字母 ➊。在循环内部，首先打印 `index` 字母 ➋，然后通过从 `counts` 中提取正确的字母来确定打印多少个星号 ➌。你使用 `while` 循环 ➍ 打印正确数量的星号，最后打印一个换行符 ➎。

清单 9-33 展示了 `AlphaHistogram` 的应用。

```
#include <cstdio>
#include <cstdint>

constexpr char pos_A{ 65 }, pos_Z{ 90 }, pos_a{ 97 }, pos_z{ 122 };
constexpr bool within_AZ(char x) { return pos_A <= x && pos_Z >= x; }
constexpr bool within_az(char x) { return pos_a <= x && pos_z >= x; }

struct AlphaHistogram {
  --snip--
};

int main(int argc, char** argv) {
  AlphaHistogram hist;
  for(size_t i{ 1 }; i<argc; i++) { ➊
    hist.ingest(argv[i]); ➋
  }
  hist.print(); ➌
}
--------------------------------------------------------------------------
$ list_933 The quick brown fox jumps over the lazy dog
A: *
B: *
C: *
D: *
E: ***
F: *
G: *
H: **
I: *
J: *
K: *
L: *
M: *
N: *
O: ****
P: *
Q: *
R: **
S: *
T: **
U: **
V: *
W: *
X: *
Y: *
Z: *
```

*清单 9-33：一个展示 `AlphaHistogram` 的程序*

在程序名称之后，你遍历每个命令行参数 ➊，并将每个参数传入 `AlphaHistogram` 对象的 `ingest` 方法 ➋。所有参数都处理完后，你打印出 `histogram` ➌。每一行对应一个字母，星号显示对应字母的绝对频率。如你所见，短语 `The quick brown fox jumps over the lazy dog` 包含了英语字母表中的每个字母。

#### *退出状态*

`main` 函数可以返回一个 `int`，表示程序的退出状态。返回值的含义由环境定义。例如，在现代桌面系统中，返回值为零表示程序执行成功。如果没有显式给出 `return` 语句，编译器会自动添加一个隐式的 `return 0`。

### 总结

本章深入探讨了函数，包括如何声明和定义函数，如何使用众多关键字修改函数行为，如何指定返回类型，如何进行重载解析，以及如何处理可变数量的参数。在讨论了如何获取指向函数的指针之后，你还学习了 lambda 表达式及其与函数对象的关系。然后，你了解了程序的入口点——`main` 函数，以及如何获取命令行参数。

**练习**

**9-1.** 实现一个 `fold` 函数模板，原型如下：

```
template <typename Fn, typename In, typename Out>
constexpr Out fold(Fn function, In* input, size_t length, Out initial);
```

例如，你的实现必须支持以下用法：

```
int main() {
  int data[]{ 100, 200, 300, 400, 500 };
  size_t data_len = 5;
  auto sum = fold([](auto x, auto y) { return x + y; }, data, data_len,
0);
  printf("Sum: %d\n", sum);
}
```

`sum`的值应该是 1,500。使用`fold`来计算以下量：`最大值`、`最小值`和大于 200 的元素数量。

**9-2.** 实现一个程序，接受任意数量的命令行参数，计算每个参数的字符长度，并打印出参数长度分布的直方图。

**9-3.** 实现一个`all`函数，其原型如下：

```
template <typename Fn, typename In>
constexpr bool all(Fn function, In* input, size_t length);
```

`Fn`函数类型是一个`谓词`，支持`bool operator()(In)`。你的`all`函数必须测试`function`是否对`input`的每个元素返回`true`。如果是，返回`true`；否则，返回`false`。

例如，你的实现必须支持以下用法：

```
int main() {
  int data[]{ 100, 200, 300, 400, 500 };
  size_t data_len = 5;
  auto all_gt100 = all([](auto x) { return x > 100; }, data, data_len);
  if(all_gt100) printf("All elements greater than 100.\n");
}
```

**进一步阅读**

+   *C++中的函数式编程：如何通过函数式技巧提升你的 C++程序*，作者：Ivan Čukić（Manning，2019）

+   *清洁代码：敏捷软件工艺手册*，作者：Robert C. Martin（Pearson Education，2009）
