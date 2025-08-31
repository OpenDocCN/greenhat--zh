## 第十四章：智能指针**

*如果你想做好一些小事，就自己做。如果你想做伟大的事情并产生巨大影响，就学会委派。  

—约翰·C·麦克斯韦尔*

![Image](img/common.jpg)

在本章中，你将探索 stdlib 和 Boost 库。这些库包含了一组智能指针，它们使用你在第四章中学到的 RAII 范式来管理动态对象。它们还促进了任何编程语言中最强大的资源管理模型。由于一些智能指针使用*分配器*来定制动态内存分配，本章还概述了如何提供用户定义的分配器。

### 智能指针

动态对象具有最灵活的生命周期。灵活性带来了巨大的责任，因此你必须确保每个动态对象只会被析构*一次*。在小型程序中，这看起来可能不太可怕，但外表常常是欺骗性的。想想异常如何影响动态内存管理吧。每次出现错误或异常时，你都需要追踪已成功分配的内存，并确保按照正确的顺序释放它们。

幸运的是，你可以使用 RAII 来处理这种繁琐的事情。通过在 RAII 对象的构造函数中获取动态存储，在析构函数中释放动态存储，泄漏（或双重释放）动态内存变得相对困难。这使得你能够通过移动和拷贝语义来管理动态对象的生命周期。

你可以自己编写这些 RAII 对象，但你也可以使用一些优秀的预先编写好的实现，称为*智能指针*。智能指针是行为类似指针并实现 RAII 的类模板，用于动态对象。

本节深入探讨了 stdlib 和 Boost 中提供的五种选项：作用域指针、唯一指针、共享指针、弱指针和侵入式指针。它们的所有权模型区分了这五种智能指针类别。

### 智能指针所有权

每个智能指针都有一个*所有权*模型，指定它与动态分配对象的关系。当智能指针拥有一个对象时，智能指针的生命周期保证至少与该对象的生命周期一样长。换句话说，当你使用智能指针时，你可以放心地知道被指向的对象是活的，并且不会泄漏。智能指针管理它所拥有的对象，因此你不会忘记销毁它，因为 RAII 已经为你处理了。

在选择使用哪种智能指针时，你的所有权需求决定了你的选择。

### 作用域指针

*作用域指针*表示对单个动态对象的*不可转移*、*独占拥有权*。不可转移意味着作用域指针不能从一个作用域转移到另一个作用域。独占拥有权意味着它们不能被复制，因此没有其他智能指针可以拥有作用域指针的动态对象。（回想一下在《内存管理》章节中提到的，关于对象的作用域，它是对象在程序中的可见范围，见第 90 页）。

`boost::scoped_ptr` 在 `<boost/smart_ptr/scoped_ptr.hpp>` 头文件中定义。

**注意**

*没有标准库作用域指针。*

#### *构造*

`boost::scoped_ptr` 接受一个模板参数，该参数对应于被指向的类型，例如 `boost::scoped_ptr<int>` 表示“指向 `int` 的作用域指针”类型。

所有智能指针，包括作用域指针，都有两种模式：*空* 和 *满*。空智能指针不拥有任何对象，类似于 `nullptr`。当智能指针被默认构造时，它开始时是空的。

作用域指针提供了一个构造函数，接受一个原始指针。（被指向的类型必须与模板参数匹配。）这将创建一个满作用域指针。通常的惯用法是使用 `new` 创建一个动态对象并将结果传递给构造函数，如下所示：

```
boost::scoped_ptr<PointedToType> my_ptr{ new PointedToType };
```

这一行动态分配了一个 `PointedToType`，并将其指针传递给作用域指针构造函数。

#### *引入誓言破坏者*

为了探索作用域指针，让我们创建一个 Catch 单元测试套件和一个 `DeadMenOfDunharrow` 类，用于跟踪有多少对象仍然存活，如示例 11-1 所示。

```
#define CATCH_CONFIG_MAIN ➊
#include "catch.hpp" ➋
#include <boost/smart_ptr/scoped_ptr.hpp> ➌

struct DeadMenOfDunharrow { ➍
  DeadMenOfDunharrow(const char* m="") ➎
    : message{ m } {
    oaths_to_fulfill++; ➏
  }
  ~DeadMenOfDunharrow() {
    oaths_to_fulfill--; ➐
  }
  const char* message;
  static int oaths_to_fulfill;
};
int DeadMenOfDunharrow::oaths_to_fulfill{};
using ScopedOathbreakers = boost::scoped_ptr<DeadMenOfDunharrow>; ➑
```

*示例 11-1：设置一个带有 `DeadMenOfDunharrow` 类的 Catch 单元测试套件，用于研究作用域指针*

首先，你声明 `CATCH_CONFIG_MAIN`，这样 Catch 会提供一个入口点 ➊，并包含 Catch 头文件 ➋，然后是 Boost 作用域指针的头文件 ➌。接下来，你声明 `DeadMenOfDunharrow` 类 ➍，它接受一个可选的空终止字符串并将其保存到 `message` 字段 ➎。一个名为 `oaths_to_fulfill` 的 `static int` 字段用于跟踪已经构造的 `DeadMenOfDunharrow` 对象的数量。因此，你在构造函数中递增 ➏，在析构函数中递减 ➐。最后，你声明 `ScopedOathbreakers` 类型别名以便于使用 ➑。

**CATCH 示例**

从现在开始，你将在大多数示例中使用 Catch 单元测试。为了简洁起见，示例省略了以下 Catch 流程：

```
#define CATCH_CONFIG_MAIN
#include "catch.hpp"
```

所有包含 `TEST_CASE` 的示例都需要这个前言。

此外，每个示例中的所有测试用例都通过，除非有注释指示相反。为了简洁起见，示例省略了“所有测试通过”这一输出。

最后，使用先前示例中的自定义类型、函数和变量的测试将省略它们，以简化代码。

#### *基于所有权的隐式布尔转换*

有时你需要判断一个 `scoped_ptr` 是否拥有一个对象，或者它是否为空。方便的是，`scoped_ptr` 会根据其所有权状态隐式转换为 `bool`：如果它拥有一个对象则为 `true`，否则为 `false`。清单 11-2 展示了这种隐式转换行为是如何工作的。

```
TEST_CASE("ScopedPtr evaluates to") {
  SECTION("true when full") {
    ScopedOathbreakers aragorn{ new DeadMenOfDunharrow{} }; ➊
    REQUIRE(aragorn); ➋
  }
  SECTION("false when empty") {
    ScopedOathbreakers aragorn; ➌
    REQUIRE_FALSE(aragorn); ➍
  }
}
```

*清单 11-2：boost::scoped_ptr 隐式转换为 `bool`。*

当你使用带指针的构造函数 ➊ 时，`scoped_ptr` 会转换为 `true` ➋。当你使用默认构造函数 ➌ 时，`scoped_ptr` 会转换为 `false` ➍。

#### *RAII 包装器*

当`scoped_ptr`拥有一个动态对象时，它确保正确的动态对象管理。在`scoped_ptr`的析构函数中，它会检查是否拥有一个对象。如果拥有，`scoped_ptr`的析构函数会删除该动态对象。

清单 11-3 通过在 `scoped_ptr` 初始化之间检查静态变量 `oaths_to_fulfill`，展示了这种行为。

```
TEST_CASE("ScopedPtr is an RAII wrapper.") {
  REQUIRE(DeadMenOfDunharrow::oaths_to_fulfill == 0); ➊
  ScopedOathbreakers aragorn{ new DeadMenOfDunharrow{} }; ➋
  REQUIRE(DeadMenOfDunharrow::oaths_to_fulfill == 1); ➌
  {
    ScopedOathbreakers legolas{ new DeadMenOfDunharrow{} }; ➍
    REQUIRE(DeadMenOfDunharrow::oaths_to_fulfill == 2); ➎
  } ➏
  REQUIRE(DeadMenOfDunharrow::oaths_to_fulfill == 1); ➐
}
```

*清单 11-3：`boost::scoped_ptr` 是一个 RAII 包装器。*

在测试开始时，`oaths_to_fulfill` 为 0，因为你还没有构造任何 `DeadMenOfDunharrow` 对象 ➊。你构造了 `scoped_ptr` `aragorn` 并传入指向动态 `DeadMenOfDunharrow` 对象的指针 ➋。这使得 `oaths_to_fulfill` 增加到 1 ➌。接着在一个嵌套作用域中，你声明了另一个 `scoped_ptr` `legolas` ➍。由于 `aragorn` 仍然存在，`oaths_to_fulfill` 此时为 2 ➎。等到内层作用域结束，`legolas` 超出作用域并析构，带走了一个 `DeadMenOfDunharrow` ➏。这使得 `DeadMenOfDunharrow` 减少到 1 ➐。

#### *指针语义*

为了方便，`scoped_ptr` 实现了解引用运算符 `operator*` 和成员解引用运算符 `operator->`，这些运算符仅仅将调用委托给被拥有的动态对象。你甚至可以通过 `get` 方法从 `scoped_ptr` 中提取出原始指针，正如 清单 11-4 所演示的那样。

```
TEST_CASE("ScopedPtr supports pointer semantics, like") {
  auto message = "The way is shut";
  ScopedOathbreakers aragorn{ new DeadMenOfDunharrow{ message } }; ➊
  SECTION("operator*") {
    REQUIRE((*aragorn).message == message); ➋
  }
  SECTION("operator->") {
    REQUIRE(aragorn->message == message); ➌
  }
  SECTION("get(), which returns a raw pointer") {
    REQUIRE(aragorn.get() != nullptr); ➍
  }
}
```

*清单 11-4：boost::scoped_ptr 支持指针语义。*

你构造了 `scoped_ptr` `aragorn` 并将 `message` 设置为 `The way is` `shut` ➊，你在三个不同的场景中测试指针语义。首先，你可以使用 `operator*` 来解引用底层指向的动态对象。在这个例子中，你解引用 `aragorn` 并提取 `message` 来验证它是否匹配 ➋。你也可以使用 `operator->` 来执行成员解引用 ➌。最后，如果你想获取指向动态对象的原始指针，可以使用 `get` 方法来提取它 ➍。

#### *与 nullptr 的比较*

`scoped_ptr` 类模板实现了比较运算符 `operator==` 和 `operator!=`，这些运算符仅在比较 `scoped_ptr` 与 `nullptr` 时才有定义。从功能上讲，这与隐式的 `bool` 转换基本相同，正如 清单 11-5 所展示的那样。

```
TEST_CASE("ScopedPtr supports comparison with nullptr") {
  SECTION("operator==") {
    ScopedOathbreakers legolas{};
    REQUIRE(legolas == nullptr); ➊
  }
  SECTION("operator!=") {
    ScopedOathbreakers aragorn{ new DeadMenOfDunharrow{} };
    REQUIRE(aragorn != nullptr); ➋
  }
}
```

*清单 11-5：`boost::scoped_ptr` 支持与 `nullptr` 的比较。*

空的 scoped 指针等于（`==`） `nullptr` ➊，而非空的 scoped 指针不等于（`!=`） `nullptr` ➋。

#### *交换*

有时你希望交换一个 `scoped_ptr` 所拥有的动态对象与另一个 `scoped_ptr` 所拥有的动态对象。这被称为 *对象交换*，`scoped_ptr` 包含一个 `swap` 方法来实现这一行为，如 清单 11-6 所示。

```
TEST_CASE("ScopedPtr supports swap") {
  auto message1 = "The way is shut.";
  auto message2 = "Until the time comes.";
  ScopedOathbreakers aragorn {
    new DeadMenOfDunharrow{ message1 } ➊
  };
  ScopedOathbreakers legolas {
    new DeadMenOfDunharrow{ message2 } ➋
  };
  aragorn.swap(legolas); ➌
  REQUIRE(legolas->message == message1); ➍
  REQUIRE(aragorn->message == message2); ➎
}
```

*清单 11-6：boost::scoped_ptr 支持 `swap`。*

你构造了两个 `scoped_ptr` 对象，`aragorn` ➊ 和 `legolas` ➋，每个对象都有不同的消息。在你执行 `aragorn` 和 `legolas` 之间的交换 ➌ 后，它们交换了动态对象。当你交换后获取它们的消息时，你会发现它们已经交换了 ➍ ➎。

#### *重置与替换 scoped_ptr*

你通常不希望在 `scoped_ptr` 对象销毁之前析构它所拥有的对象。例如，你可能希望用一个新的动态对象替换它所拥有的对象。你可以使用 `scoped_ptr` 的重载 `reset` 方法来处理这两项任务。

如果你不提供任何参数，`reset` 只会销毁所拥有的对象。

如果你提供一个新的动态对象作为参数，`reset` 将首先销毁当前拥有的对象，然后获取该参数的所有权。清单 11-7 通过为每种情况提供一个测试，展示了这种行为。

```
TEST_CASE("ScopedPtr reset") {
  ScopedOathbreakers aragorn{ new DeadMenOfDunharrow{} }; ➊
  SECTION("destructs owned object.") {
    aragorn.reset(); ➋
    REQUIRE(DeadMenOfDunharrow::oaths_to_fulfill == 0); ➌
  }
  SECTION("can replace an owned object.") {
    auto message = "It was made by those who are Dead.";
    auto new_dead_men = new DeadMenOfDunharrow{ message }; ➍
    REQUIRE(DeadMenOfDunharrow::oaths_to_fulfill == 2); ➎
    aragorn.reset(new_dead_men); ➏
    REQUIRE(DeadMenOfDunharrow::oaths_to_fulfill == 1); ➐
    REQUIRE(aragorn->message == new_dead_men->message); ➑
    REQUIRE(aragorn.get() == new_dead_men); ➒
  }
}
```

*清单 11-7：boost::scoped_ptr 支持 `reset`。*

两个测试的第一步都是构造一个 `scoped_ptr` 指针 `aragorn`，它拥有一个 `DeadMenOfDunharrow` ➊。在第一个测试中，你不带参数地调用 `reset` ➋。这会导致 `scoped_ptr` 析构它所拥有的对象，`oaths_to_fulfill` 减少到 0 ➌。

在第二个测试中，你创建了新的、动态分配的 `new_dead_men`，并附加了自定义的 `message` ➍。这将使 `oaths_to_fill` 增加到 2，因为 `aragorn` 依然存活 ➎。接下来，你调用 `reset`，并以 `new_dead_men` 作为参数 ➏，这会做两件事：

+   它导致原本由 `aragorn` 所拥有的 `DeadMenOfDunharrow` 被析构，这使得 `oaths_to_fulfill` 减少到 1 ➐。

+   它将 `new_dead_men` 作为由 `aragorn` 所拥有的动态分配对象。当你解引用 `message` 字段时，会发现它与 `new_dead_men` 所持有的 `message` 匹配 ➑。（等效地，`aragorn.get()` 返回 `new_dead_men` ➒。）

#### *不可转移性*

你不能移动或复制 `scoped_ptr`，使其成为不可转移的。清单 11-8 展示了尝试移动或复制 `scoped_ptr` 会导致无效程序。

```
void by_ref(const ScopedOathbreakers&) { } ➊
void by_val(ScopedOathbreakers) { } ➋

TEST_CASE("ScopedPtr can") {
  ScopedOathbreakers aragorn{ new DeadMenOfDunharrow };
  SECTION("be passed by reference") {
    by_ref(aragorn); ➌
  }
  SECTION("not be copied") {
    // DOES NOT COMPILE:
    by_val(aragorn); ➍
    auto son_of_arathorn = aragorn; ➐
  }
  SECTION("not be moved") {
    // DOES NOT COMPILE:
    by_val(std::move(aragorn)); ➏
    auto son_of_arathorn = std::move(aragorn); ➐
  }
}
```

*清单 11-8：`boost::scoped_ptr` 是不可转移的。（此代码无法编译。）*

首先，你声明接受`scoped_ptr`引用 ➊ 和值 ➋ 的虚拟函数。你仍然可以通过引用 ➌ 传递`scoped_ptr`，但是尝试通过值传递将无法编译 ➍。此外，尝试使用`scoped_ptr`的复制构造函数或复制赋值操作符 ➎ 也将无法编译。如果你尝试使用`std::move`移动一个`scoped_ptr`，你的代码也将无法编译 ➏➐。

**注意**

*通常，使用`boost::scoped_ptr`不会比使用原始指针产生额外的开销。*

#### *boost::scoped_array*

`boost::scoped_array`是一个用于动态数组的作用域指针。它支持与`boost::scoped_ptr`相同的用法，但它还实现了`operator[]`，因此你可以像操作原始数组一样与作用域数组的元素进行交互。清单 11-9 说明了这一附加功能。

```
TEST_CASE("ScopedArray supports operator[]") {
  boost::scoped_array<int➊> squares{
    new int➋[5] { 0, 4, 9, 16, 25 }
  };
  squares[0] = 1; ➌
  REQUIRE(squares[0] == 1); ➍
 REQUIRE(squares[1] == 4);
  REQUIRE(squares[2] == 9);
}
```

*清单 11-9：`boost::scoped_array`实现了`operator[]`。*

你声明`scoped_array`的方式与声明`scoped_ptr`相同，使用单一的模板参数 ➊。对于`scoped_array`，模板参数是数组中包含的类型 ➋，而不是数组的类型。你将一个动态数组传递给`squares`的构造函数，使得动态数组`squares`成为该数组的所有者。你可以使用`operator[]`来写入 ➌ 和读取 ➍ 元素。

#### *支持的部分操作列表*

到目前为止，你已经了解了作用域指针的主要特性。作为参考，表 11-1 列出了所有已讨论的运算符，以及一些尚未覆盖的运算符。在表格中，`ptr`是一个原始指针，而`s_ptr`是一个作用域指针。有关更多信息，请参阅 Boost 文档。

**表 11-1：** 所有支持的`boost::scoped_ptr`操作

| **操作** | **说明** |
| --- | --- |
| `scoped_ptr<...>{ }` 或 `scoped_ptr <...>{ nullptr }` | 创建一个空的作用域指针。 |
| `scoped_ptr <...>{` ptr `}` | 创建一个作用域指针，拥有由 ptr 指向的动态对象。 |
| `~scoped_ptr<...>()` | 如果已满，则对拥有的对象调用`delete`。 |
| s_ptr1`.swap(`s_ptr2`)` | 交换 s_ptr1 和 s_ptr2 之间的拥有对象。 |
| `swap(`s_ptr1, s_ptr2`)` | 与`swap`方法相同的自由函数。 |
| s_ptr`.reset()` | 如果已满，则对`s_ptr`拥有的对象调用`delete`。 |
| s_ptr`.reset(`ptr`)` | 删除当前拥有的对象，然后获取 ptr 的所有权。 |
| ptr `=` s_ptr`.get()` | 返回原始指针`ptr`；`s_ptr`保持所有权。 |
| `*`s_ptr | 对拥有对象的解引用操作符。 |
| s_ptr`->` | 对拥有对象的成员解引用操作符。 |
| `bool{` s_ptr `}` | `bool`转换：如果已满则为`true`，如果为空则为`false`。 |

### 唯一指针

一个*唯一指针*对单一动态对象拥有可转移的独占所有权。你*可以*移动唯一指针，这使得它们具有可转移性。它们也拥有独占所有权，因此*不能*被复制。标准库提供了一个在`<memory>`头文件中的`unique_ptr`。

**注意**

*Boost 并不提供独占指针。*

#### *构造*

`std::unique_ptr`接受一个模板参数，对应于所指向的类型，例如`std::unique_ptr<int>`表示“指向`int`类型的独占指针”。

与作用域指针类似，独占指针具有一个默认构造函数，将独占指针初始化为空。它还提供一个接受原始指针的构造函数，该构造函数获取所指向的动态对象的所有权。一个构造方法是使用`new`创建一个动态对象，并将结果传递给构造函数，像这样：

```
std::unique_ptr<int> my_ptr{ new int{ 808 } };
```

另一种方法是使用`std::make_unique`函数。`make_unique`是一个模板函数，它接受所有参数并将它们转发到模板参数的适当构造函数中。这避免了使用`new`的需要。通过使用`std::make_unique`，你可以将前面的对象初始化重写为：

```
auto my_ptr = make_unique<int>(808);
```

`make_unique`函数是为了避免在使用 C++旧版本的`new`时出现一些微妙的内存泄漏问题而创建的。然而，在 C++的最新版本中，这些内存泄漏问题已经不再发生。你选择使用哪种构造函数主要取决于你的偏好。

#### *支持的操作*

`std::unique_ptr`函数支持`boost::scoped_ptr`支持的所有操作。例如，你可以使用以下类型别名作为清单 11-1 到 11-7 中的`ScopedOathbreakers`的替代：

```
using UniqueOathbreakers = std::unique_ptr<DeadMenOfDunharrow>;
```

独占指针和作用域指针的主要区别之一是，你可以移动独占指针，因为它们是*可转移的*。

#### *可转移的、独占的所有权*

不仅独占指针是可转移的，而且它们具有独占所有权（你*不能*复制它们）。清单 11-10 演示了如何使用`unique_ptr`的移动语义。

```
TEST_CASE("UniquePtr can be used in move") {
  auto aragorn = std::make_unique<DeadMenOfDunharrow>(); ➊
  SECTION("construction") {
    auto son_of_arathorn{ std::move(aragorn) }; ➋
    REQUIRE(DeadMenOfDunharrow::oaths_to_fulfill == 1); ➌
  }
  SECTION("assignment") {
    auto son_of_arathorn = std::make_unique<DeadMenOfDunharrow>(); ➍
 REQUIRE(DeadMenOfDunharrow::oaths_to_fulfill == 2); ➎
    son_of_arathorn = std::move(aragorn); ➏
    REQUIRE(DeadMenOfDunharrow::oaths_to_fulfill == 1); ➐
  }
}
```

*清单 11-10：`std::unique_ptr`支持用于转移所有权的移动语义。*

这个清单创建了一个名为`aragorn`的`unique_ptr` ➊，你将在两个不同的测试中使用它。

在第一次测试中，你将`aragorn`通过`std::move`移动到`son_of_arathorn`的移动构造函数中 ➋。因为`aragorn`将其`DeadMenOfDunharrow`的所有权转移给了`son_of_arathorn`，所以`oaths_to_fulfill`对象的值仍然是 1 ➌。

第二次测试通过`make_unique`构造`son_of_arathorn` ➍，这将`oaths_to_fulfill`的值推至 2 ➎。接下来，你使用移动赋值操作符将`aragorn`移入`son_of_arathorn` ➏。同样，`aragorn`将所有权转移给`son_of_aragorn`。由于`son_of_aragorn`一次只能拥有一个动态对象，因此移动赋值操作符会销毁当前拥有的对象，然后清空`aragorn`的动态对象。这导致`oaths_to_fulfill`的值减小至 1 ➐。

#### *独占数组*

与`boost::scoped_ptr`不同，`std::unique_ptr`内置了对动态数组的支持。你只需将数组类型作为模板参数，像这样使用独占指针的类型：`std::unique_ptr<int[]>`。

*非常重要*的是，你不要使用动态数组 `T[]` 来初始化 `std::unique_ptr<T>`。这样做会导致未定义的行为，因为你会导致对数组执行 `delete`（而不是 `delete[]`）。编译器无法拯救你，因为 `operator new[]` 返回的指针与 `operator new` 返回的指针是无法区分的。

和 `scoped_array` 类似，`unique_ptr` 到数组类型提供了 `operator[]` 来访问元素。清单 11-11 演示了这一概念。

```
TEST_CASE("UniquePtr to array supports operator[]") {
  std::unique_ptr<int[]➊> squares{
    new int[5]{ 1, 4, 9, 16, 25 } ➋
  };
  squares[0] = 1; ➌
  REQUIRE(squares[0] == 1); ➍
  REQUIRE(squares[1] == 4);
  REQUIRE(squares[2] == 9);
}
```

*清单 11-11：`std::unique_ptr` 到数组类型支持 `operator[]`。*

模板参数 `int[]` ➊ 指示 `std::unique_ptr` 拥有一个动态数组。你传入一个新创建的动态数组 ➋，然后使用 `operator[]` 来设置第一个元素 ➌；接着你使用 `operator[]` 来检索元素 ➍。

#### *删除器*

`std::unique_ptr` 有第二个可选模板参数，称为删除器类型。unique pointer 的 *删除器* 是在 unique pointer 需要销毁其拥有的对象时调用的内容。

`unique_ptr` 实例化包含以下模板参数：

```
std::unique_ptr<T, Deleter=std::default_delete<T>>
```

这两个模板参数分别是 `T`，表示拥有的动态对象类型，以及 `Deleter`，表示负责释放拥有对象的对象类型。默认情况下，`Deleter` 是 `std::default_delete<T>`，它调用 `delete` 或 `delete[]` 来删除动态对象。

要编写自定义删除器，所需的只是一个可调用的类似函数的对象，该对象可以使用 `T*` 来调用。（unique pointer 会忽略删除器的返回值。）你将此删除器作为第二个参数传递给 unique pointer 的构造函数，如 清单 11-12 所示。

```
#include <cstdio>

auto my_deleter = [](int* x) { ➊
  printf("Deleting an int at %p.", x);
  delete x;
};
std::unique_ptr<int➋, decltype(my_deleter)➌> my_up{
  new int,
  my_deleter
};
```

*清单 11-12：将自定义删除器传递给 unique pointer*

拥有的对象类型是 `int` ➋，所以你声明了一个 `my_deleter` 函数对象，它接受一个 `int*` ➊。你使用 `decltype` 来设置删除器模板参数 ➌。

#### *自定义删除器和系统编程*

当 `delete` 不提供你需要的资源释放行为时，你会使用自定义删除器。在某些环境下，你可能永远不需要自定义删除器，而在其他情况下，例如系统编程，你可能会发现它们非常有用。考虑一个简单的例子，使用 `<cstdio>` 头文件中的底层 API `fopen`、`fprintf` 和 `fclose` 管理文件。

`fopen` 函数打开一个文件，其签名如下：

```
FILE*➊ fopen(const char *filename➋, const char *mode➌);
```

成功时，`fopen` 返回一个非 `nullptr` 值的 `FILE*` ➊。失败时，`fopen` 返回 `nullptr` 并将静态 `int` 变量 `errno` 设置为一个错误代码，例如访问被拒绝（`EACCES` `= 13`）或没有此文件（`ENOENT` `= 2`）。

**注意**

*请参阅 errno.h 头文件，以查看所有错误条件及其对应的整数值。*

`FILE*`文件句柄是操作系统管理的文件的引用。*句柄*是操作系统中某些资源的一个不透明、抽象的引用。`fopen`函数接受两个参数：`filename` ➋是你想要打开的文件路径，`mode` ➌是表 11-2 中列出的六个选项之一。

**表 11-2：**`fopen`的六种`mode`选项

| **字符串** | **操作** | **文件存在：** | **文件不存在：** | **备注** |
| --- | --- | --- | --- | --- |
| `r` | 读 |  | `fopen`失败 |  |
| `w` | 写 | 覆盖 | 创建 | 如果文件存在，所有内容会被丢弃。 |
| `a` | 附加 |  | 创建 | 总是写入文件末尾。 |
| `r+` | 读/写 |  | `fopen`失败 |  |
| `w+` | 读/写 | 覆盖 | 创建 | 如果文件存在，所有内容会被丢弃。 |
| `a+` | 读/写 |  | 创建 | 总是写入文件末尾。 |

一旦使用完文件，你必须手动用`fclose`关闭它。未关闭文件句柄是资源泄漏的常见来源，如下所示：

```
void fclose(FILE* file);
```

要写入文件，可以使用`fprintf`函数，它类似于将内容打印到控制台的`printf`，但`fprintf`将内容打印到文件中。`fprintf`函数的使用方法与`printf`完全相同，只不过你需要在格式字符串之前提供文件句柄作为第一个参数：

```
int➊ fprintf(FILE* file➋, const char* format_string➌, ...➍);
```

成功时，`fprintf`返回写入打开文件的字符数 ➊ ➋。`format_string`与`printf`的格式字符串相同 ➌，变参也是一样的 ➍。

你可以使用`std::unique_ptr`管理`FILE`。显然，当你准备关闭文件时，你不希望调用`delete`来释放`FILE*`文件句柄。相反，你需要使用`fclose`来关闭。因为`fclose`是一个类似函数的对象，接受`FILE*`作为参数，所以它是一个合适的删除器。

清单 11-13 中的程序将字符串`HELLO, DAVE.`写入文件`HAL9000`，并使用唯一指针来执行打开文件的资源管理。

```
#include <cstdio>
#include <memory>

using FileGuard = std::unique_ptr<FILE, int(*)(FILE*)>; ➊

void say_hello(FileGuard file➋) {
  fprintf(file.get(), "HELLO DAVE"); ➌
}

int main() {
  auto file = fopen("HAL9000", "w"); ➍
  if (!file) return errno; ➎
  FileGuard file_guard{ file, fclose }; ➏
  // File open here
  say_hello(std::move(file_guard)); ➐
  // File closed here
  return 0;
}
```

*清单 11-13：使用`std::unique_ptr`和自定义删除器管理文件句柄的程序*

这个列表将`FileGuard`类型别名简化为➊（注意，删除器类型与`fclose`的类型匹配）。接下来是一个`sa_hello`函数，它按值接受`FileGuard` ➋。在`sa_hello`内，你用`fprintf HELLO DAVE`将内容写入`file` ➌。由于`file`的生命周期与`sa_hello`绑定，文件会在`sa_hello`返回时被关闭。在`main`函数中，你以`w`模式打开文件`HAL9000`，这会创建或覆盖该文件，并将原始`FILE*`文件句柄保存到`file` ➍。你检查`file`是否为`nullptr`，表示打开文件时发生错误，如果`HAL9000`无法打开，则返回`errno` ➎。接着，你通过传递文件句柄`file`和自定义删除器`fclose`来构造一个`FileGuard` ➏。此时，文件已打开，并且由于自定义删除器，`file_guard`会自动管理文件的生命周期。

要调用`say_hello`，需要将所有权传递到该函数中（因为它按值接受`FileGuard`）➐。回想一下在“值类别”中提到的内容（见第 124 页），像`file_guard`这样的变量是左值。这意味着你必须通过`std::move`将它转移到`say_hello`中，这样就会将`HELLO DAVE`写入文件。如果省略了`std::move`，编译器会尝试将其复制到`say_hello`中。由于`unique_ptr`有一个删除的拷贝构造函数，这将导致编译错误。 |

当`say_hello`返回时，它的`FileGuard`参数会被销毁，且自定义删除器会在文件句柄上调用`fclose`。基本上，不可能泄漏文件句柄。你已经将其绑定到了`FileGuard`的生命周期上。 |

#### *支持的操作的部分列表*

表 11-3 列出了所有支持的`std::unique_ptr`操作。在此表中，`ptr`是一个原始指针，`u_ptr`是一个独占指针，`del`是一个删除器。 |

**表 11-3：** 所有支持的`std::unique_ptr`操作

| **操作** | **说明** |
| --- | --- |
| `unique_ptr<...>{ }` 或 `unique_ptr<...>{ nullptr }` | 创建一个空的独占指针，使用`std::default_delete<...>`删除器。 |
| `unique_ptr<...>{` ptr `}` | 创建一个拥有`ptr`指向的动态对象的独占指针。使用`std::default_delete<...>`删除器。 |
| `unique_ptr<...>{` ptr, del `}` | 创建一个拥有`ptr`指向的动态对象的独占指针。使用 del 作为删除器。 |
| `unique_ptr<...>{ move(`u_ptr`) }` | 创建一个拥有`u_ptr`指向的动态对象的独占指针。将所有权从 u_ptr 转移到新创建的独占指针。还会移动 u_ptr 的删除器。 |
| `~unique_ptr<...>()` | 如果已满，则对拥有的对象调用删除器。 |
| u_ptr1 `= move(`u_ptr2`)` | 将 u_ptr2 的拥有对象和删除器的所有权转移到 u_ptr1。如果已经有对象，则销毁当前拥有的对象。 |
| u_ptr1.`swap(`u_ptr2`)` | 在 u_ptr1 和 u_ptr2 之间交换拥有的对象和删除器。 |
| `swap(`u_ptr1`,` u_ptr2`)` | 一个与`swap`方法相同的自由函数。 |
| u_ptr`.reset()` | 如果已满，则对 u_ptr 拥有的对象调用删除器。 |
| u_ptr`.reset(`ptr`)` | 删除当前拥有的对象；然后获得 ptr 的所有权。 |
| ptr `=` u_ptr`.release()` | 返回原始指针 ptr；u_ptr 变为空。删除器*不会*被调用。 |
| ptr `=` u_ptr`.get()` | 返回原始指针 ptr；u_ptr 保持所有权。 |
| `*`u_ptr | 对拥有的对象执行解引用操作符。 |
| u_ptr`->` | 对拥有的对象执行成员解引用操作符。 |
| u_ptr`[`index`]` | 引用索引处的元素（仅限数组）。 |
| `bool{` u_ptr `}` | `bool`转换：如果已满则为`true`，如果为空则为`false`。 |
| u_ptr1 `==` u_ptr2u_ptr1 `!=` u_ptr2u_ptr1 `>` u_ptr2u_ptr1 `>=` u_ptr2u_ptr1 `<` u_ptr2u_ptr1 `<=` u_ptr2 | 比较操作符；相当于对原始指针执行比较操作符。 |
| u_ptr`.get_deleter()` | 返回对删除器的引用。 |

### 共享指针

*共享指针*对单个动态对象拥有可转移、非独占的所有权。你可以移动共享指针，这使得它们是可转移的，而且你*可以*复制它们，这使得它们的所有权是非独占的。

非独占所有权意味着`shared_ptr`会检查是否有其他`shared_ptr`对象也拥有该对象，在销毁它之前。这样，最后一个拥有者将是释放该对象的对象。

标准库中在`<memory>`头文件中提供了`std::shared_ptr`，而 Boost 则在`<boost/smart_ptr/shared_ptr.hpp>`头文件中提供了`boost::shared_ptr`。这里我们使用标准库版本。

**注意**

*标准库和 Boost 的`shared_ptr`基本相同，唯一的显著区别是 Boost 的 shared pointer 不支持数组，并且需要使用`boost::shared_array`类（位于`<boost/smart_ptr/shared_array.hpp>`中）。Boost 提供了一个共享指针是为了向后兼容，但你应该使用标准库的共享指针。*

#### *构造*

`std::shared_ptr`指针支持与`std::unique_ptr`相同的所有构造函数。默认构造函数会生成一个空的共享指针。若要建立对动态对象的所有权，你可以将一个指针传递给`shared_ptr`构造函数，如下所示：

```
std::shared_ptr<int> my_ptr{ new int{ 808 } };
```

你还可以使用一个推导参数的`std::make_shared`模板函数，将参数传递给所指向类型的构造函数：

```
auto my_ptr = std::make_shared<int>(808);
```

通常你应该使用`make_shared`。共享指针需要一个*控制块*，它跟踪多个量，包括共享所有者的数量。当你使用`make_shared`时，你可以同时分配控制块和被拥有的动态对象。如果你先使用`operator new`，然后再分配一个共享指针，那你就是进行了两次分配，而不是一次。

**注意**

*有时你可能不想使用`make_shared`。例如，如果你要使用`weak_ptr`，即使你能释放对象，你仍然需要控制块。在这种情况下，你可能会更倾向于使用两个分配。*

由于控制块是一个动态对象，`shared_ptr`对象有时需要分配动态对象。如果你想控制`shared_ptr`的分配方式，可以重载`operator new`。但这就像用大炮打麻雀一样。一个更合适的方法是提供一个可选的模板参数，称为*分配器类型*。

#### *指定分配器*

分配器负责分配、创建、销毁和释放对象。默认分配器`std::allocator`是一个在`<memory>`头文件中定义的模板类。默认分配器从动态存储区分配内存，并接受一个模板参数。（你将在“分配器”一章中了解如何使用用户自定义分配器来定制这一行为，见第 365 页）。

`shared_ptr` 构造函数和 `make_shared` 都有一个分配器类型模板参数，总共包含三个模板参数：指向的类型、删除器类型和分配器类型。由于复杂的原因，你只需要声明*指向的类型*参数。你可以将其他参数类型视为从指向的类型中推导出来的。

例如，以下是一个完整的 `make_shared` 调用，包含一个构造函数参数、一个自定义删除器和一个显式的 `std::allocator`：

```
std::shared_ptr<int➊> sh_ptr{
  new int{ 10 }➋,
  [](int* x) { delete x; } ➌,
  std::allocator<int>{} ➍
};
```

在这里，你为指向的类型 ➊ 指定了一个单一的模板参数 `int`。在第一个参数中，你为 `int` 分配并初始化内存 ➋。接下来是一个自定义删除器 ➌，作为第三个参数，你传递一个 `std::allocator` ➍。

出于技术原因，你无法在 `make_shared` 中使用自定义删除器或自定义分配器。如果你需要自定义分配器，可以使用 `make_shared` 的姐妹函数，即 `std::allocate_shared`。`std::allocate_shared` 函数将分配器作为第一个参数，并将其余的参数转发给拥有对象的构造函数：

```
auto sh_ptr = std::allocate_shared<int➊>(std::allocator<int>{}➋, 10➌);
```

与 `make_shared` 一样，你将拥有的类型指定为模板参数 ➊，但是将分配器作为第一个参数 ➋。其余的参数会转发给 `int` 的构造函数 ➌。

**注意**

*对于好奇的人，以下是不能使用自定义删除器与 `make_shared` 的两个原因。首先，`make_shared` 使用 `new` 来为拥有的对象和控制块分配空间。适合 `new` 的删除器是 `delete`，因此通常自定义删除器不合适。其次，自定义删除器通常无法知道如何处理控制块，只能处理拥有的对象。*

无法使用 `make_shared` 或 `allocate_shared` 指定自定义删除器。如果你想在共享指针中使用自定义删除器，必须直接使用适当的 `shared_ptr` 构造函数之一。

#### *支持的操作*

`std::shared_ptr` 支持 `std::unique_ptr` 和 `boost::scoped_ptr` 支持的所有操作。你可以使用以下类型别名来替代 Listings 11-1 到 11-7 中的 `ScopedOathbreakers` 和 Listings 11-10 到 11-13 中的 `UniqueOathbreakers`：

```
using SharedOathbreakers = std::shared_ptr<DeadMenOfDunharrow>;
```

共享指针和独占指针之间的主要功能差异在于，你可以复制共享指针。

#### *可转移的、非独占所有权*

共享指针是可转移的（你*可以*移动它们），并且具有非独占所有权（你*可以*复制它们）。Listing 11-10，展示了独占指针的移动语义，对于共享指针也是一样的。 Listing 11-14 证明共享指针也支持复制语义。

```
TEST_CASE("SharedPtr can be used in copy") {
  auto aragorn = std::make_shared<DeadMenOfDunharrow>();
  SECTION("construction") {
    auto son_of_arathorn{ aragorn }; ➊
    REQUIRE(DeadMenOfDunharrow::oaths_to_fulfill == 1); ➋
  }
  SECTION("assignment") {
    SharedOathbreakers son_of_arathorn; ➌
    son_of_arathorn = aragorn; ➍
    REQUIRE(DeadMenOfDunharrow::oaths_to_fulfill == 1); ➎
  }
  SECTION("assignment, and original gets discarded") {
    auto son_of_arathorn = std::make_shared<DeadMenOfDunharrow>(); ➏
    REQUIRE(DeadMenOfDunharrow::oaths_to_fulfill == 2);➐
    son_of_arathorn = aragorn; ➑
    REQUIRE(DeadMenOfDunharrow::oaths_to_fulfill == 1); ➒
  }
}
```

*Listing 11-14: `std::shared_ptr` 支持复制。*

在构造共享指针`aragorn`之后，您有三个测试。第一个测试说明，您用来构建`son_``of_arathorn` ➊的复制构造函数共享同一个`DeadMenOfDunharrow` ➋。

在第二个测试中，您构造了一个空的共享指针`son_of _ara``thorn` ➌，然后展示复制赋值 ➍ 也不会改变`DeadMenOfDunharrow`的数量 ➎。

第三个测试说明，当您构造完整的共享指针`son_of_arathorn` ➏时，`DeadMenOfDunharrow`的数量增加到 2 ➐。当您将`aragorn`复制赋值给`son_of_arathorn` ➑时，`son_of_arathorn`删除了其`DeadMenOfDunharrow`，因为它拥有唯一所有权。然后增加了`aragorn`拥有的`DeadMenOfDunharrow`的引用计数。因为两个共享指针拥有同一个`DeadMenOfDunharrow`，所以`oaths_to_fulfill`从 2 减少到 1 ➒。

#### *共享数组*

`shared array`是拥有动态数组并支持`operator[]`的共享指针。它的工作方式与唯一数组相同，只是它具有非排他性所有权。

#### *删除器*

对于共享指针而言，删除器的工作方式与对唯一指针的工作方式相同，只是您无需提供删除器类型的模板参数。只需将删除器作为第二个构造函数参数传递即可。例如，要将清单 11-12 转换为使用共享指针，您只需插入以下类型别名：

```
using FileGuard = std::shared_ptr<FILE>;
```

现在，您正在管理具有共享所有权的`FILE*`文件句柄。

#### *支持操作的部分列表*

表 11-4 提供了支持的`shared_ptr`构造函数的大部分完整列表。在本表中，`ptr`是原始指针，`sh_ptr`是共享指针，`u_ptr`是唯一指针，`del`是删除器，`alc`是分配器。

**表 11-4:** 所有支持的`std::shared_ptr`构造函数

| **操作** | **注释** |
| --- | --- |
| `shared_ptr<...>{ }` or `shared_ptr<...>{ nullptr }` | 创建一个空的共享指针，使用`std::default_delete<T>`和`std::allocator<T>`。 |
| `shared_ptr<...>{` ptr, [del], [alc] `}` | 创建一个共享指针，拥有由 ptr 指向的动态对象。默认情况下使用`std::default_delete<T>`和`std::allocator<T>`；否则，使用 del 作为删除器，alc 作为分配器（如果提供）。 |
| `shared_ptr<...>{` sh_ptr `}` | 创建一个共享指针，拥有由共享指针`sh_ptr`指向的动态对象。从`sh_ptr`复制所有权到新创建的共享指针。还复制了`sh_ptr`的删除器和分配器。 |
| `shared_ptr<...>{` sh_ptr , ptr `}` | 一个*别名构造函数*：生成的共享指针持有对 ptr 的未管理引用，但参与 sh_ptr 的引用计数。 |
| `shared_ptr<...>{ move(`sh_ptr`) }` | 创建一个共享指针，拥有由共享指针`sh_ptr`指向的动态对象。将所有权从`sh_ptr`转移到新创建的共享指针。还移动了`sh_ptr`的删除器。 |
| `shared_ptr<...>{ move(`u_ptr`) }` | 创建一个共享指针，拥有由独占指针 u_ptr 指向的动态对象。将所有权从 u_ptr 转移到新创建的共享指针，并移动 u_ptr 的删除器。 |

表 11-5 列出了大多数支持的`std::shared_ptr`操作。在此表中，`ptr`是原始指针，`sh_ptr`是共享指针，`u_ptr`是独占指针，`del`是删除器，`alc`是分配器。

**表 11-5:** 大多数支持的`std::shared_ptr`操作

| **操作** | **备注** |
| --- | --- |
| `~shared_ptr<...>()` | 如果没有其他拥有者，则调用删除器删除拥有的对象。 |
| sh_ptr1 `=` sh_ptr2 | 将 sh_ptr2 的拥有权和删除器复制到 sh_ptr1，拥有者数量加 1。如果没有其他拥有者，则销毁当前拥有的对象。 |
| sh_ptr `= move(`u_ptr`)` | 将拥有的对象和删除器的所有权从 u_ptr 转移到 sh_ptr。如果没有其他拥有者，则销毁当前拥有的对象。 |
| sh_ptr1 `= move(`sh_ptr2`)` | 将拥有的对象和删除器的所有权从 sh_ptr2 转移到 sh_ptr1。如果没有其他拥有者，则销毁当前拥有的对象。 |
| sh_ptr1`.swap(`sh_ptr2`)` | 在 sh_ptr1 和 sh_ptr2 之间交换拥有的对象和删除器。 |
| `swap(`sh_ptr1`,` sh_ptr2`)` | 一个与`swap`方法相同的自由函数。 |
| sh_ptr`.reset()` | 如果满了，并且没有其他拥有者，则调用删除器删除 sh_ptr 拥有的对象。 |
| sh_ptr`.reset(`ptr`,` [del]`,` [alc]`)` | 如果没有其他拥有者，则删除当前拥有的对象；然后接管 ptr 的拥有权。可以选择提供删除器 del 和分配器 alc，默认为`std::default_delete<T>`和`std::allocator<T>`。 |
| ptr `=` sh_ptr`.get()` | 返回原始指针 ptr；sh_ptr 保留拥有权。 |
| `*`sh_ptr | 对拥有对象的解引用操作符。 |
| sh_ptr`->` | 对拥有对象的成员解引用操作符。 |
| sh_ptr`.use_count()` | 引用拥有当前对象的共享指针总数；如果为空则为零。 |
| sh_ptr`[`index`]` | 返回索引处的元素（仅适用于数组）。 |
| `bool{` sh_ptr `}` | `bool`转换：如果满了返回`true`，如果为空返回`false`。 |
| sh_ptr1 `==` sh_ptr2sh_ptr1 `!=` sh_ptr2sh_ptr1 `>` sh_ptr2sh_ptr1 `>=` sh_ptr2sh_ptr1 `<` sh_ptr2sh_ptr1 `<=` sh_ptr2 | 比较操作符；等价于在原始指针上评估比较操作符。 |
| sh_ptr`.get_deleter()` | 返回删除器的引用。 |

### 弱指针

*弱指针*是一种特殊的智能指针，它不拥有所引用对象的所有权。弱指针允许你跟踪一个对象，并且*仅在被跟踪的对象仍然存在时*才能将弱指针转换为共享指针。这允许你对对象生成临时拥有权。像共享指针一样，弱指针是可移动和可复制的。

弱指针的一个常见用途是*缓存*。在软件工程中，缓存是一个临时存储数据的数据结构，目的是加速数据的读取。缓存可以保持指向对象的弱指针，这样一旦所有其他所有者释放它们，缓存中的对象就会被销毁。定期，缓存可以扫描其存储的弱指针，并修剪掉那些没有其他所有者的指针。

标准库提供了`std::weak_ptr`，而 Boost 库提供了`boost::weak_ptr`。这两者本质上是相同的，仅供与各自的共享指针`std::shared_ptr`和`boost::shared_ptr`一起使用。

#### *构造*

弱指针的构造函数与作用域指针、唯一指针和共享指针完全不同，因为弱指针并不直接拥有动态对象。默认构造函数会构造一个空的弱指针。要构造一个跟踪动态对象的弱指针，必须使用共享指针或另一个弱指针来构造。

例如，以下代码将一个共享指针传递给弱指针的构造函数：

```
auto sp = std::make_shared<int>(808);
std::weak_ptr<int> wp{ sp };
```

现在，弱指针`wp`将跟踪由共享指针`sp`拥有的对象。

#### *获取暂时所有权*

弱指针通过调用其`lock`方法来暂时拥有它所跟踪的对象。`lock`方法总是创建一个共享指针。如果被跟踪的对象仍然存活，返回的共享指针会拥有该对象。如果被跟踪的对象已不再存活，返回的共享指针则为空。参考示例 11-15。

```
TEST_CASE("WeakPtr lock() yields") {
  auto message = "The way is shut.";
  SECTION("a shared pointer when tracked object is alive") {
    auto aragorn = std::make_shared<DeadMenOfDunharrow>(message); ➊
    std::weak_ptr<DeadMenOfDunharrow> legolas{ aragorn }; ➋
    auto sh_ptr = legolas.lock(); ➌
    REQUIRE(sh_ptr->message == message); ➍
    REQUIRE(sh_ptr.use_count() == 2); ➎
  }
  SECTION("empty when shared pointer empty") {
    std::weak_ptr<DeadMenOfDunharrow> legolas;
    {
      auto aragorn = std::make_shared<DeadMenOfDunharrow>(message); ➏
      legolas = aragorn; ➐
    }
 auto sh_ptr = legolas.lock(); ➑
    REQUIRE(nullptr == sh_ptr); ➒
  }
}
```

*示例 11-15：`std::weak_ptr`暴露了一个`lock`方法，用于获取暂时的所有权。*

在第一次测试中，你创建了一个共享指针`aragorn` ➊，并赋予它一个消息。接着，使用`aragorn` ➋构造一个弱指针`legolas`。这样，`legolas`就开始跟踪由`aragorn`拥有的动态对象。当你调用弱指针的`lock`方法 ➌ 时，`aragorn`仍然存活，因此你获得了共享指针`sh_ptr`，它也拥有同样的`DeadMenOfDunharrow`对象。你通过断言`message`相同 ➍，并且*使用计数*为 2 ➎来确认这一点。

在第二次测试中，你也创建了一个`aragorn`共享指针 ➏，但这次你使用了赋值运算符 ➐，因此之前为空的弱指针`legolas`现在开始跟踪由`aragorn`拥有的动态对象。接下来，`aragorn`超出作用域并死亡。此时，`legolas`继续跟踪一个已死的对象。当你此时调用`lock`方法 ➑ 时，得到的是一个空的共享指针 ➒。

#### *高级模式*

在一些共享指针的高级用法中，你可能需要创建一个类，使得实例能够创建指向自身的共享指针。`std::enable_shared_from_this`类模板实现了这种行为。从用户的角度来看，唯一需要做的就是在类定义中继承`enable_shared_from_this`。这将暴露出`shared_from_this`和`weak_from_this`方法，它们分别生成指向当前对象的`shared_ptr`或`weak_ptr`。这是一个小众情况，但如果你想查看更多细节，请参考[util.smartptr.enab]。

#### *支持的操作*

表 11-6 列出了大多数支持的弱指针操作。在该表中，`w_ptr`是一个弱指针，`sh_ptr`是一个共享指针。

**表 11-6：** 大多数支持的`std::shared_ptr`操作

| **操作** | **说明** |
| --- | --- |
| `weak_ptr<...>{ }` | 创建一个空的弱指针。 |
| `weak_ptr<...>{` w_ptr `}` 或 `weak_ptr<...>{` sh_ptr `}` | 跟踪弱指针 w_ptr 或共享指针 sh_ptr 所指向的对象。 |
| `weak_ptr<...>{ move(`w_ptr`) }` | 跟踪 w_ptr 所指向的对象；然后清空 w_ptr。 |
| `~weak_ptr<...>()` | 对跟踪的对象没有影响。 |
| w_ptr1 `=` sh_ptr 或 w_ptr1 `=` w_ptr2 | 用 sh_ptr 所拥有的对象或 w_ptr2 所跟踪的对象替换当前跟踪的对象。 |
| w_ptr1 `= move(`w_ptr2`)` | 用 w_ptr2 所跟踪的对象替换当前跟踪的对象，并清空 w_ptr2。 |
| sh_ptr `=` w_ptr.`lock()` | 创建共享指针 sh_ptr，拥有 w_ptr 所跟踪的对象。如果跟踪的对象已过期，则 sh_ptr 为空。 |
| w_ptr1`.swap(`w_ptr2`)` | 交换 w_ptr1 和 w_ptr2 之间的跟踪对象。 |
| `swap(`w_ptr1`,` w_ptr2`)` | 与`swap`方法相同的自由函数。 |
| w_ptr`.reset()` | 清空弱指针。 |
| w_ptr`.use_count()` | 返回拥有跟踪对象的共享指针数量。 |
| w_ptr`.expired()` | 如果跟踪的对象已过期，则返回`true`，否则返回`false`。 |
| sh_ptr`.use_count()` | 返回拥有所拥有对象的共享指针的总数；如果为空则为零。 |

### 侵入式指针

*侵入式指针*是指向具有嵌入式引用计数的对象的共享指针。因为共享指针通常保持引用计数，所以它们不适合拥有此类对象。Boost 提供了一种实现，称为`boost::intrusive_ptr`，在`<boost/smart_ptr/intrusive_ptr.hpp>`头文件中定义。

很少会遇到需要使用侵入式指针的情况。但有时你会使用包含嵌入式引用的操作系统或框架。例如，在 Windows COM 编程中，侵入式指针非常有用：继承自`IUnknown`接口的 COM 对象具有`AddRef`和`Release`方法，分别用于增加和减少嵌入式引用计数。

每次创建一个`intrusive_ptr`时，都会调用`intrusive_ptr_add_ref`函数。当`intrusive_ptr`被销毁时，它会调用`intrusive_ptr_release`自由函数。当引用计数降到零时，你负责在`intrusive_ptr_release`中释放适当的资源。要使用`intrusive_ptr`，你必须提供这些函数的合适实现。

清单 11-16 演示了使用 `DeadMenOfDunharrow` 类的侵入式指针。请参考该清单中的 `intrusive_ptr_add_ref` 和 `intrusive_ptr_release` 的实现。

```
#include <boost/smart_ptr/intrusive_ptr.hpp>

using IntrusivePtr = boost::intrusive_ptr<DeadMenOfDunharrow>; ➊
size_t ref_count{}; ➋

void intrusive_ptr_add_ref(DeadMenOfDunharrow* d) {
  ref_count++; ➌
}

void intrusive_ptr_release(DeadMenOfDunharrow* d) {
 ref_count--; ➍
  if (ref_count == 0) delete d; ➎
}
```

*清单 11-16: `intrusive_ptr_add_ref` 和 `intrusive_ptr_release` 的实现*

使用类型别名`IntrusivePtr`可以减少一些输入量 ➊。接下来，你声明了一个具有静态存储期的`ref_count` ➋。这个变量跟踪活动侵入式指针的数量。在`intrusive_ptr_add_ref`中，你会增加`ref_count` ➌。在`intrusive_ptr_release`中，你会减少`ref_count` ➍。当`ref_count`降至零时，你删除`DeadMenOfDunharrow`对象 ➎。

**注意**

*在使用清单 11-16 中的设置时，务必确保只使用一个动态的 `DeadMenOfDunharrow` 对象与侵入式指针。`ref_count` 方法只能正确追踪一个对象。如果你有多个由不同侵入式指针拥有的动态对象，`ref_count` 将变得无效，导致错误的 `delete` 行为 ➎。*

清单 11-17 展示了如何在清单 11-16 的设置中使用侵入式指针。

```
TEST_CASE("IntrusivePtr uses an embedded reference counter.") {
  REQUIRE(ref_count == 0); ➊
  IntrusivePtr aragorn{ new DeadMenOfDunharrow{} }; ➋
  REQUIRE(ref_count == 1); ➌
  {
    IntrusivePtr legolas{ aragorn }; ➍
    REQUIRE(ref_count == 2); ➎
  }
  REQUIRE(DeadMenOfDunharrow::oaths_to_fulfill == 1); ➏
}
```

*清单 11-17: 使用 `boost::intrusive_ptr`*

这个测试首先检查`ref_count`是否为零 ➊。接下来，通过传递动态分配的`DeadMenOfDunharrow`对象 ➋ 来构造一个侵入式指针。这会将`ref_count`增加到 1，因为创建侵入式指针会调用`intrusive_ptr_add_ref` ➌。在一个块作用域内，你构造了另一个侵入式指针`legolas`，它与`aragorn`共享所有权 ➍。这将`ref_count`增加到 2 ➎，因为创建侵入式指针会调用`intrusive_ptr_add_ref`。当`legolas`超出块作用域时，它会被析构，从而调用`intrusive_ptr_release`。这会将`ref_count`减少到 1，但不会导致删除所拥有的对象 ➏。

### 智能指针选项总结

表 11-7 总结了可在 stdlib 和 Boost 中使用的所有智能指针选项。

**表 11-7:** stdlib 和 Boost 中的智能指针

| **类型名称** | **stdlib 头文件** | **Boost 头文件** | **可移动/可转移所有权** | **可复制/非独占所有权** |
| --- | --- | --- | --- | --- |
| `scoped_ptr` |  | `<boost/smart_ptr/scoped_ptr.hpp>` |  |  |
| `scoped_array` |  | `<boost/smart_ptr/scoped_array.hpp>` |  |  |
| `unique_ptr` | `<memory>` |  | ✓ |  |
| `shared_ptr` | `<memory>` | `<boost/smart_ptr/shared_ptr.hpp>` | ✓ | ✓ |
| `shared_array` |  | `<boost/smart_ptr/shared_array.hpp>` | ✓ | ✓ |
| `weak_ptr` | `<memory>` | `<boost/smart_ptr/weak_ptr.hpp>` | ✓ | ✓ |
| `intrusive_ptr` |  | `<boost/smart_ptr/intrusive_ptr.hpp>` | ✓ | ✓ |

### 分配器

分配器是低级对象，负责处理内存请求。stdlib 和 Boost 库使你能够提供分配器，定制库如何分配动态内存。

在大多数情况下，默认分配器`std::allocate`完全足够。它使用`operator new(size_t)`分配内存，该操作从自由存储区（即堆）中分配原始内存。它使用`operator delete(void*)`释放内存，该操作从自由存储区中释放原始内存。（请回顾《重载`new`操作符》中的内容，在第 189 页中提到，`operator new`和`operator delete`是在`<new>`头文件中定义的。）

在某些场景中，比如游戏、高频交易、科学分析和嵌入式应用，默认自由存储操作所带来的内存和计算开销是不可接受的。在这些场景中，实现自定义分配器相对容易。请注意，除非你进行了一些性能测试，表明默认分配器是瓶颈，否则你真的不应该实现自定义分配器。自定义分配器的背后理念是，你对自己特定程序的了解远超过默认分配器模型的设计者，因此你可以做出改进，提升分配性能。

至少，你需要提供一个具有以下特征的模板类，才能使其作为分配器工作：

+   一个合适的默认构造函数

+   一个对应模板参数的`value_type`成员

+   一个模板构造函数，可以在处理`value_type`变化时复制分配器的内部状态

+   一个`allocate`方法

+   一个`deallocate`方法

+   一个`operator==`和一个`operator!=`

列表 11-18 中的`MyAllocator`类实现了一个简单的教学版本的`std::allocate`，用于跟踪你进行了多少次分配和释放。

```
#include <new>

static size_t n_allocated, n_deallocated;

template <typename T>
struct MyAllocator {
  using value_type = T; ➊
  MyAllocator() noexcept{ } ➋
  template <typename U>
  MyAllocator(const MyAllocator<U>&) noexcept { } ➌
  T* allocate(size_t n) { ➍
    auto p = operator new(sizeof(T) * n);
    ++n_allocated;
    return static_cast<T*>(p);
  }
  void deallocate(T* p, size_t n) { ➎
    operator delete(p);
    ++n_deallocated;
  }
};

template <typename T1, typename T2>
bool operator==(const MyAllocator<T1>&, const MyAllocator<T2>&) {
  return true; ➏
}
template <typename T1, typename T2>
bool operator!=(const MyAllocator<T1>&, const MyAllocator<T2>&) {
  return false; ➐
}
```

*列表 11-18：一个基于`std::allocate`的`MyAllocator`类*

首先，你声明`value_type`类型别名为`T`，这是实现分配器的要求之一➊。接下来是默认构造函数➋和模板构造函数➌。这两个构造函数都是空的，因为分配器没有状态可以传递。

`allocate`方法➍通过使用`operator new`分配所需字节数`sizeof(T) * n`来模拟`std::allocate`。接下来，它增加了静态变量`n_allocated`，这样你就可以跟踪分配次数以进行测试。`allocate`方法随后返回指向新分配内存的指针，在返回之前将`void*`转换为相关的指针类型。

`deallocate`方法➎通过调用`operator delete`来模拟`std::allocate`。类似于`allocate`，它增加了用于测试的`n_deallocated`静态变量，并返回。

最后的任务是实现一个`operator==`和一个`operator!=`，接受新的类模板。因为分配器没有状态，任何实例都与其他实例相同，因此`operator==`返回`true` ➏，而`operator!=`返回`true` ➐。

**注意**

*示例 11-18 是一个教学工具，实际上并没有提高分配效率。它只是包装了`new`和`delete`的调用*。

到目前为止，唯一你知道使用分配器的类是`std::shared_ptr`。考虑一下示例 11-19 如何将`MyAllocator`与`std::allocate`共享一起使用。

```
TEST_CASE("Allocator") {
  auto message = "The way is shut.";
  MyAllocator<DeadMenOfDunharrow> alloc; ➊
  {
    auto aragorn = std::allocate_shared<DeadMenOfDunharrow>(alloc➋, message➌);
    REQUIRE(aragorn->message == message); ➍
    REQUIRE(n_allocated == 1); ➎
    REQUIRE(n_deallocated == 0); ➏
  }
  REQUIRE(n_allocated == 1); ➐
  REQUIRE(n_deallocated == 1); ➑
}
```

*示例 11-19：使用`MyAllocator`与`std::shared_ptr`*

你创建了一个名为`alloc`的`MyAllocator`实例 ➊。在一个块内，你将`alloc`作为第一个参数传递给`allocate_shared` ➋，它创建了一个包含自定义`message`的共享指针`aragorn` ➌。接着，你确认`aragorn`包含正确的`message` ➍，`n_allocated`为 1 ➎，`n_deallocated`为 0 ➏。

在`aragorn`超出块作用域并被销毁后，你可以验证`n_allocated`仍为 1 ➐，而`n_deallocated`现在为 1 ➑。

**注意**

*因为分配器处理底层细节，你可以深入到非常细微的地方来指定它们的行为。参见 ISO C++ 17 标准中的[allocator.requirements]，以获取详细的说明。*

### 总结

智能指针通过 RAII 管理动态对象，你可以提供分配器来定制动态内存分配。根据你选择的智能指针，你可以将不同的所有权模式编码到动态对象中。

**练习**

**11-1.** 重新实现示例 11-13，使用`std::shared_ptr`而不是`std::unique_ptr`。注意，尽管你将所有权要求从独占变为非独占，但你仍然将所有权转移给了`call_hello`函数。

**11-2.** 从调用`say_hello`中移除`std::move`。然后再调用一次`say_hello`。注意，`file_guard`的所有权不再被`转移`到`say_hello`函数中。这允许多次调用。

**11-3.** 实现一个`Hal`类，在其构造函数中接受一个`std::shared_ptr<FILE>`。在 Hal 的析构函数中，将短语`Stop, Dave.`写入共享指针持有的文件句柄。实现一个`write_status`函数，将短语`I'm completely operational.`写入文件句柄。以下是你可以使用的类声明：

```
struct Hal {
  Hal(std::shared_ptr<FILE> file);
  ~Hal();
  void write_status();
  std::shared_ptr<FILE> file;
};
```

**11-4.** 创建多个`Hal`实例并调用`write_status`。注意，你不需要跟踪有多少`Hal`实例是打开的：文件管理通过共享指针的共享所有权模型来处理。

**进一步阅读**

+   *ISO 国际标准 ISO/IEC (2017) — C++编程语言*（国际标准化组织；瑞士日内瓦；*[`isocpp.org/std/the-standard/`](https://isocpp.org/std/the-standard/)*）

+   *《C++程序设计语言》*（第 4 版），作者：比雅尼·斯特劳斯特鲁普（Pearson Education，2013 年）

+   *《Boost C++库》*（第 2 版），作者：博里斯·舍林（XML Press，2014 年）

+   *《C++标准库：教程与参考》*（第 2 版），作者：尼古拉·M·乔苏蒂斯（Addison-Wesley Professional，2012 年）
