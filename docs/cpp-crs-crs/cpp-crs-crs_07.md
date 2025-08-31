## 第七章：运行时多态性

*有一天，构造师 Trurl 组装了一台能够从 n 开始创造任何东西的机器。*

—*斯坦尼斯瓦夫·莱姆*，《赛博利亚》

![图片](img/common.jpg)

在本章中，你将学习什么是多态性以及它解决了哪些问题。接着，你将学习如何实现运行时多态性，这使你能够通过在程序执行期间替换组件来改变程序的行为。章初将讨论运行时多态性代码中的几个关键概念，包括接口、对象组合和继承。然后，你将开发一个持续的示例，展示如何使用多种类型的日志记录器记录银行交易。最后，你将通过使用更优雅的基于接口的解决方案来重构这个初始的、幼稚的解决方案。

### 多态性

*多态代码*是你只需编写一次，便可与不同类型一起重用的代码。最终，这种灵活性带来了松耦合和高度可重用的代码。它消除了繁琐的复制和粘贴，使代码更加易于维护和可读。

C++提供了两种多态方法。*编译时多态代码*包含了可以在编译时确定的多态类型。另一种方法是*运行时多态性*，它则包含了在运行时确定的类型。你选择哪种方法取决于你是否知道要在编译时还是运行时使用的多态类型。由于这些紧密相关的主题涉及较多内容，因此被分为两章进行讲解。第六章将重点讨论编译时多态性。

### 一个激励示例

假设你负责实现一个`Bank`类，该类用于在账户之间转账。审计对`Bank`类的交易非常重要，因此你提供了通过`ConsoleLogger`类支持日志记录，如示例 5-1 所示。

```
#include <cstdio>

struct ConsoleLogger {
  void log_transfer(long from, long to, double amount) { ➊
    printf("%ld -> %ld: %f\n", from, to, amount); ➋
  }
};

struct Bank {
  void make_transfer(long from, long to, double amount) { ➌
    --snip-- ➍
    logger.log_transfer(from, to, amount); ➎
  }
  ConsoleLogger logger;
};

int main() {
  Bank bank;
  bank.make_transfer(1000, 2000, 49.95);
  bank.make_transfer(2000, 4000, 20.00);
}
--------------------------------------------------------------------------
1000 -> 2000: 49.950000
2000 -> 4000: 20.000000
```

*示例 5-1：一个使用`ConsoleLogger`的`Bank`类*

首先，你实现了一个`ConsoleLogger`，其中包含一个`log_transfer`方法➊，该方法接受交易的详细信息（发送者、接收者、金额）并打印出来➋。`Bank`类有一个`make_transfer`方法➌，该方法（概念上）处理交易➍，然后使用`logger`成员➎记录交易。`Bank`和`ConsoleLogger`有各自不同的关注点——`Bank`处理银行逻辑，`ConsoleLogger`处理日志记录。

假设你需要实现不同类型的日志记录器。例如，你可能需要一个远程服务器日志记录器，一个本地文件日志记录器，或者甚至一个将作业发送到打印机的日志记录器。此外，你还必须能够在运行时更改程序的日志记录方式（例如，管理员可能需要将日志记录从网络日志切换到本地文件系统日志，因为某些服务器维护）。

你如何完成这样的任务？

一种简单的方法是使用`enum` `class`在各种日志记录器之间切换。清单 5-2 为清单 5-1 添加了一个`FileLogger`。

```
#include <cstdio>
#include <stdexcept>

struct FileLogger {
  void log_transfer(long from, long to, double amount) { ➊
    --snip--
    printf("[file] %ld,%ld,%f\n", from, to, amount);
  }
};

struct ConsoleLogger {
  void log_transfer(long from, long to, double amount) {
    printf("[cons] %ld -> %ld: %f\n", from, to, amount);
  }
};

enum class LoggerType { ➋
  Console,
  File
};

struct Bank {
  Bank() : type { LoggerType::Console } { } ➌
  void set_logger(LoggerType new_type) { ➍
    type = new_type;
  }

  void make_transfer(long from, long to, double amount) {
    --snip--
    switch(type) { ➎
    case LoggerType::Console: {
      consoleLogger.log_transfer(from, to, amount);
      break;
    } case LoggerType::File: {
      fileLogger.log_transfer(from, to, amount);
      break;
    } default: {
 throw std::logic_error("Unknown Logger type encountered.");
    } }
  }
private:
  LoggerType type;
  ConsoleLogger consoleLogger;
  FileLogger fileLogger;
};

int main() {
  Bank bank;
  bank.make_transfer(1000, 2000, 49.95);
  bank.make_transfer(2000, 4000, 20.00);
  bank.set_logger(LoggerType::File); ➏
  bank.make_transfer(3000, 2000, 75.00);
}
--------------------------------------------------------------------------
[cons] 1000 -> 2000: 49.950000
[cons] 2000 -> 4000: 20.000000
[file] 3000,2000,75.000000
```

*清单 5-2：一个更新后的清单 5-1，具有运行时多态的日志记录器*

你（理论上）通过实现一个`FileLogger`来添加日志到文件的能力 ➊。你还创建了一个`enum class LoggerType` ➋，这样你就可以在运行时切换日志记录行为。你在`Bank`构造函数中将类型字段初始化为`Console` ➌。在更新后的`Bank`类中，你添加了一个`set_logger`函数 ➍来执行所需的日志记录行为。你在`make_transfer`中使用`type`来`switch`到正确的日志记录器 ➎。要更改`Bank`类的日志记录行为，你可以使用`set_logger`方法 ➏，对象会在内部处理分派。

#### *添加新的日志记录器*

清单 5-2 是有效的，但这种方法存在一些设计问题。添加新的日志记录类型需要你在代码中进行多次更新：

1.  你需要编写一个新的日志记录器类型。

1.  你需要向`enum class LoggerType`添加一个新的`enum`值。

1.  你必须在`switch`语句中添加一个新的案例 ➎。

1.  你必须将新的日志记录类作为成员添加到`Bank`中。

对于一个简单的更改来说，这可真是很多工作！

考虑一种替代方法，让`Bank`持有一个指向日志记录器的指针。这样，你可以直接设置指针，完全去除`LoggerType`。你利用了所有日志记录器具有相同函数原型的事实。这就是接口的思想：`Bank`类不需要知道它持有的`Logger`引用的实现细节，只需要知道如何调用其方法。

如果我们可以将`ConsoleLogger`替换为另一个支持相同操作的类型，岂不是很好吗？比如一个`FileLogger`？

允许我向你介绍*接口*。

#### *接口*

在软件工程中，*接口*是一个不包含数据或代码的共享边界。它定义了所有接口实现都同意支持的函数签名。*实现*是声明支持接口的代码或数据。你可以把接口看作是实现接口的类与该类的用户（也叫*消费者*）之间的契约。

消费者知道如何使用实现，因为他们知道契约。实际上，消费者从不需要知道底层实现类型。例如，在清单 5-1 中，`Bank`是`ConsoleLogger`的消费者。

接口强加了严格的要求。接口的消费者只能使用接口中明确定义的方法。`Bank`类不需要知道`ConsoleLogger`是如何执行其功能的。它只需要知道如何调用`log_transfer`方法。

接口促进了高度可重用且松耦合的代码。你可以理解指定接口的符号，但你需要了解一些关于对象组合和实现继承的知识。

#### *对象组合与实现继承*

*对象组合*是一种设计模式，其中一个类包含其他类类型的成员。另一种过时的设计模式叫做 *实现继承*，它实现了运行时多态性。实现继承允许你构建类的层次结构，每个子类从其父类继承功能。多年来，积累的实现继承经验使得许多人认为它是一种反模式。例如，Go 和 Rust——两种新兴且越来越受欢迎的系统编程语言——完全不支持实现继承。由于两个原因，简要讨论实现继承是必要的：

+   你可能会在遗留代码中遇到它。

+   你定义 C++ 接口的独特方式与实现继承有共同的血脉，因此你会熟悉这些机制。

**注意**

*如果你正在处理充满实现继承的 C++ 代码，请参阅* 《C++程序设计语言》第四版，Bjarne Stroustrup 著，第二十章和 21 章。

### 定义接口

不幸的是，C++ 中没有 `interface` 关键字。你必须使用过时的继承机制来定义接口。这只是你在编程这个已有 40 多年历史的语言时必须应对的一个古老遗留问题。

列表 5-3 展示了一个完全指定的 `Logger` 接口以及一个实现该接口的相应 `ConsoleLogger`。在 列表 5-3 中至少有四种构造方式对你来说是陌生的，本节将逐一讲解这些内容。

```
#include <cstdio>

struct Logger {
  virtual➊ ~Logger()➋ = default;
  virtual void log_transfer(long from, long to, double amount) = 0➌;
};

struct ConsoleLogger : Logger ➍ {
  void log_transfer(long from, long to, double amount) override ➎ {
    printf("%ld -> %ld: %f\n", from, to, amount);
  }
};
```

*列表 5-3：一个 `Logger` 接口和一个重构的 `ConsoleLogger`*

为了解析 列表 5-3，你需要了解 `virtual` 关键字 ➊、虚拟析构函数 ➋、`=0` 后缀和纯虚方法 ➌、基类继承 ➍，以及 `override` 关键字 ➎。理解这些后，你将知道如何定义一个接口。接下来的章节将详细讨论这些概念。

#### *基类继承*

第四章深入探讨了 `exception` 类是所有其他标准库异常的基类，以及 `logic_error` 和 `runtime_error` 类是如何从 `exception` 类派生出来的。这两个类反过来又成为描述更详细错误条件的其他派生类的基类，例如 `invalid_argument` 和 `system_error`。嵌套的异常类形成了一个类层次结构的示例，并代表了一种实现继承设计。

你使用以下语法声明派生类：

```
struct DerivedClass : BaseClass {
  --snip--
};
```

要为 `DerivedClass` 定义继承关系，你使用冒号（`:`）后跟基类的名称 `BaseClass`。

派生类的声明方式与其他类相同。其好处在于你可以将派生类的引用当作基类引用类型来使用。列表 5-4 中用`DerivedClass`引用替代了`BaseClass`引用。

```
struct BaseClass {}; ➊
struct DerivedClass : BaseClass {}; ➋
void are_belong_to_us(BaseClass& base) {} ➌

int main() {
  DerivedClass derived;
  are_belong_to_us(derived); ➍
}
```

*列表 5-4：用派生类替代基类的程序*

`DerivedClass` ➋继承自`BaseClass` ➊。`are_belong_to_us`函数接受一个指向`BaseClass`的引用参数`base` ➌。由于`DerivedClass`继承自`BaseClass` ➍，因此你可以用`DerivedClass`的实例来调用它。

相反的情况并不成立。列表 5-5 尝试用基类替代派生类。

```
struct BaseClass {}; ➊
struct DerivedClass : BaseClass {}; ➋
void all_about_that(DerivedClass& derived) {} ➌

int main() {
  BaseClass base;
  all_about_that(base); // No! Trouble! ➍
}
```

*列表 5-5：该程序尝试用基类替代派生类。（此列表无法编译。）*

在这里，`BaseClass` ➊并没有继承自`DerivedClass` ➋。（继承关系是相反的。）`all_about_that`函数接受一个`DerivedClass`类型的参数 ➌。当你尝试用`BaseClass` ➍来调用`all_about_that`时，编译器会报错。

你希望从类中派生的主要原因是为了继承其成员。

#### *成员继承*

派生类继承自基类的非私有成员。类可以像使用普通成员一样使用继承的成员。成员继承的预期好处是，你可以在基类中定义功能，而不必在派生类中重复它。不幸的是，经验使得许多程序员社区的人避免使用成员继承，因为它相比基于组合的多态性，更容易导致脆弱、难以理解的代码。（这也是为什么许多现代编程语言排除了成员继承。）

列表 5-6 中的类展示了成员继承。

```
#include <cstdio>

struct BaseClass {
  int the_answer() const { return 42; } ➊
 const char* member = "gold"; ➋
private:
  const char* holistic_detective = "Dirk Gently"; ➌
};

struct DerivedClass : BaseClass ➍ 
  void announce_agency() {
    // This line doesn't compile:
    // printf("%s's Holistic Detective Agency\n", holistic_detective); { ➎
  }
};

int main() {
  DerivedClass x;
  printf("The answer is %d\n", x.the_answer()); ➏
  printf("%s member\n", x.member); { ➐
}
--------------------------------------------------------------------------
The answer is 42 ➏
gold member ➐
```

*列表 5-6：使用继承成员的程序*

在这里，`BaseClass`有一个公共方法 ➊，一个公共字段 ➋，以及一个私有字段 ➌。你声明一个`DerivedClass`继承自`BaseClass` ➍，然后在`main`中使用它。由于它们作为公共成员被继承，`the_answer` ➏和`member` ➐可以在`DerivedClass x`上访问。然而，取消注释 ➎ 会导致编译错误，因为`holistic_detective`是私有的，因此不会被派生类继承。

#### *虚方法*

如果你希望允许派生类重写基类的方法，可以使用`virtual`关键字。通过在方法定义中添加`virtual`，你声明如果派生类提供了实现，则使用派生类的实现。在实现中，你需要在方法声明中添加`override`关键字，如列表 5-7 所示。

```
#include <cstdio>

struct BaseClass {
  virtual➊ const char* final_message() const {
    return "We apologize for the incontinence.";
  }
};

struct DerivedClass : BaseClass ➋ {
  const char* final_message() const override ➌ {
    return "We apologize for the inconvenience.";
  }
};

int main() {
  BaseClass base;
  DerivedClass derived;
  BaseClass& ref = derived;
 printf("BaseClass:    %s\n", base.final_message()); ➍
  printf("DerivedClass: %s\n", derived.final_message()); ➎
  printf("BaseClass&:   %s\n", ref.final_message()); ➏
}
--------------------------------------------------------------------------
BaseClass:    We apologize for the incontinence. ➍
DerivedClass: We apologize for the inconvenience. ➎
BaseClass&:   We apologize for the inconvenience. ➏
```

*列表 5-7：使用虚拟成员的程序*

`BaseClass`包含一个虚拟成员 ➊。在`DerivedClass`中 ➋，你重写了继承的成员，并使用了`override`关键字 ➌。当手头是`BaseClass`实例时，使用的是`BaseClass`的实现 ➍。当手头是`DerivedClass`实例时，即使你通过`BaseClass`引用来操作，它依然使用的是`DerivedClass`的实现 ➎。

如果你想*要求*派生类实现某个方法，可以在方法定义后加上`=0`后缀。你用`virtual`关键字和`=0`后缀来标记纯虚方法。含有任何纯虚方法的类无法被实例化。在 Listing 5-8 中，考虑基类使用纯虚方法的重构，这与 Listing 5-7 相似。

```
#include <cstdio>

struct BaseClass {
  virtual const char* final_message() const = 0; ➊
};

struct DerivedClass : BaseClass ➋ {
  const char* final_message() const override ➌ {
    return "We apologize for the inconvenience.";
  }
};

int main() {
  // BaseClass base; // Bang! ➍
  DerivedClass derived;
  BaseClass& ref = derived;
  printf("DerivedClass: %s\n", derived.final_message()); ➎
  printf("BaseClass&:   %s\n", ref.final_message()); ➏
}
--------------------------------------------------------------------------
DerivedClass: We apologize for the inconvenience. ➎
BaseClass&:   We apologize for the inconvenience. ➏
```

*Listing 5-8: 使用纯虚方法重构 Listing 5-7 的示例*

`=0`后缀指定了一个纯虚方法 ➊，这意味着你不能实例化`BaseClass`——只能从它派生。`DerivedClass`仍然继承自`BaseClass` ➋，并且你提供了必需的`final_message` ➌。试图实例化`BaseClass`会导致编译错误 ➍。`DerivedClass`和`BaseClass`引用的行为和之前一样 ➎➏。

**注意**

*虚函数可能会带来运行时开销，尽管成本通常较低（通常在常规函数调用的 25%以内）。编译器会生成* 虚函数表（vtables）*，其中包含函数指针。在运行时，接口的消费者通常并不知道其底层类型，但它知道如何调用接口的方法（这要归功于 vtable）。在某些情况下，链接器可以检测到所有接口的使用并* 去虚拟化*函数调用。这会将函数调用从 vtable 中移除，从而消除相关的运行时开销*。

#### *纯虚类和虚析构函数*

你可以通过从只包含纯虚方法的基类派生来实现接口继承。这类类被称为*纯虚类*。在 C++中，接口总是纯虚类。通常，你会为接口添加虚拟析构函数。在一些罕见的情况下，如果没有将析构函数标记为虚拟函数，可能会导致资源泄漏。参见 Listing 5-9，该示例说明了未添加虚拟析构函数的危险。

```
#include <cstdio>

struct BaseClass {};

struct DerivedClass : BaseClass➊ {
  DerivedClass() { ➋
    printf("DerivedClass() invoked.\n");
  }
  ~DerivedClass() { ➌
    printf("~DerivedClass() invoked.\n");
  }
};

int main() {
  printf("Constructing DerivedClass x.\n");
  BaseClass* x{ new DerivedClass{} }; ➍
  printf("Deleting x as a BaseClass*.\n");
  delete x; ➎
}
--------------------------------------------------------------------------
Constructing DerivedClass x.
DerivedClass() invoked.
Deleting x as a BaseClass*.
```

*Listing 5-9: 说明基类中非虚析构函数危险性的示例*

这里你看到一个`DerivedClass`类继承自`BaseClass` ➊。这个类有一个构造函数 ➋ 和析构函数 ➌，它们在被调用时会打印信息。在`main`函数中，你通过`new`分配并初始化一个`DerivedClass`，并将结果赋值给一个`BaseClass`指针 ➍。当你`delete`这个指针 ➎时，`BaseClass`的析构函数会被调用，但`DerivedClass`的析构函数不会被调用！

为`BaseClass`的析构函数添加虚拟关键字可以解决这个问题，正如 Listing 5-10 所示。

```
#include <cstdio>

struct BaseClass {
  virtual ~BaseClass() = default; ➊
};

struct DerivedClass : BaseClass {
  DerivedClass() {
    printf("DerivedClass() invoked.\n");
  }
  ~DerivedClass() {
    printf("~DerivedClass() invoked.\n"); ➋
  }
};

int main() {
  printf("Constructing DerivedClass x.\n");
  BaseClass* x{ new DerivedClass{} };
  printf("Deleting x as a BaseClass*.\n");
  delete x; ➌
}
--------------------------------------------------------------------------
Constructing DerivedClass x.
DerivedClass() invoked.
Deleting x as a BaseClass*.
~DerivedClass() invoked. ➋
```

*列表 5-10：对列表 5-9 的重构，带虚拟析构函数*

添加虚拟析构函数➊会导致在删除`BaseClass`指针➌时调用`DerivedClass`的析构函数，从而导致`DerivedClass`的析构函数打印消息➋。

在声明接口时声明虚拟析构函数是可选的，但要小心。如果你忘记在接口中实现虚拟析构函数，并不小心做了类似列表 5-9 的操作，你可能会泄漏资源，并且编译器不会警告你。

**注意**

*声明一个受保护的非虚拟析构函数是声明一个公共虚拟析构函数的一个不错替代方案，因为它会在编写删除基类指针的代码时导致编译错误。有些人不喜欢这种方法，因为最终你必须创建一个具有公共析构函数的类，如果你从这个类派生，就会遇到相同的问题。*

#### *实现接口*

要声明一个接口，声明一个纯虚类。要实现一个接口，必须从它派生。因为接口是纯虚的，所有实现都必须实现接口的所有方法。

标记这些方法时使用`override`关键字是一个好习惯。这表明你打算重写一个虚拟函数，让编译器帮助你避免一些简单的错误。

#### *使用接口*

作为消费者，你只能处理接口的引用或指针。编译器无法预先知道为底层类型分配多少内存：如果编译器能够知道底层类型，你最好使用模板。

设置成员有两种选择：

**构造函数注入** 使用构造函数注入时，通常使用接口引用。因为引用不能重新绑定，它们在对象的生命周期内不会改变。

**属性注入** 使用属性注入时，你通过一个方法来设置指针成员。这样可以改变该成员指向的对象。

你可以通过在构造函数中接受一个接口指针，同时提供一个方法来将指针设置为其他对象，从而结合这些方法。

通常，当注入的字段在对象的生命周期内不会改变时，你会使用构造函数注入。如果你需要更改该字段的灵活性，你将提供方法来执行属性注入。

### 更新银行日志记录器

`Logger`接口允许你提供多个日志记录实现。这允许`Logger`消费者使用`log_transfer`方法记录转账日志，而不需要知道日志记录的实现细节。你已经在列表 5-2 中实现了`ConsoleLogger`，接下来让我们看看如何添加另一个名为`FileLogger`的实现。为了简便起见，在这个代码中，你只修改了日志输出的前缀，但你可以想象如何实现一些更复杂的行为。

列表 5-11 定义了一个 `FileLogger`。

```
#include <cstdio>

struct Logger {
  virtual ~Logger() = default; ➊
  virtual void log_transfer(long from, long to, double amount) = 0; ➋
};

struct ConsoleLogger : Logger ➌ {
  void log_transfer(long from, long to, double amount) override ➍ {
    printf("[cons] %ld -> %ld: %f\n", from, to, amount);
  }
};

struct FileLogger : Logger ➎ {
  void log_transfer(long from, long to, double amount) override ➏ {
    printf("[file] %ld,%ld,%f\n", from, to, amount);
  }
};
```

*列表 5-11：`Logger`，`ConsoleLogger` 和 `FileLogger`*

`Logger` 是一个纯虚类（接口），具有默认的虚析构函数 ➊ 和一个方法 `log_transfer` ➋。`ConsoleLogger` 和 `FileLogger` 是 `Logger` 的实现，因为它们从该接口派生 ➌➎。你已经实现了 `log_transfer` 并在两者上放置了 `override` 关键字 ➍➏。

现在我们将看看如何使用构造函数注入或属性注入来更新 `Bank`。

#### *构造函数注入*

使用构造函数注入，你有一个 `Logger` 引用，并将其传入 `Bank` 类的构造函数。列表 5-12 在 列表 5-11 的基础上，添加了适当的 `Bank` 构造函数。这样，你可以确定特定 `Bank` 实例化时将执行的日志记录类型。

```
--snip--
// Include Listing 5-11
struct Bank {
  Bank(Logger& logger) : logger{ logger }➊ { }
  void make_transfer(long from, long to, double amount) {
    --snip--
    logger.log_transfer(from, to, amount);
  }
private:
  Logger& logger;
};

int main() {
  ConsoleLogger logger;
  Bank bank{ logger }; ➋
  bank.make_transfer(1000, 2000, 49.95);
  bank.make_transfer(2000, 4000, 20.00);
}
--------------------------------------------------------------------------
[cons] 1000 -> 2000: 49.950000
[cons] 2000 -> 4000: 20.000000
```

*列表 5-12：使用构造函数注入、接口和对象组合重构列表 5-2，以取代笨重的 `enum class` 方法*

`Bank` 类的构造函数使用成员初始化器 ➊ 设置 `logger` 的值。引用不能重新赋值，因此 `logger` 所指向的对象在 `Bank` 生命周期内不会改变。你在 `Bank` 构造时就确定了日志记录器的选择 ➋。

#### *属性注入*

你也可以选择使用属性注入来将 `Logger` 插入到 `Bank` 中，而不是使用构造函数注入。这种方法使用指针而不是引用。因为指针可以重新赋值（与引用不同），你可以随时更改 `Bank` 的行为。列表 5-13 是 列表 5-12 的属性注入变体。

```
--snip--
// Include Listing 5-11

struct Bank {
  void set_logger(Logger* new_logger) {
    logger = new_logger;
  }
  void make_transfer(long from, long to, double amount) {
    if (logger) logger->log_transfer(from, to, amount);
  }
private:
  Logger* logger{};
};

int main() {
  ConsoleLogger console_logger;
  FileLogger file_logger;
  Bank bank;
  bank.set_logger(&console_logger); ➊
  bank.make_transfer(1000, 2000, 49.95); ➋
  bank.set_logger(&file_logger); ➌
  bank.make_transfer(2000, 4000, 20.00); ➍
}
--------------------------------------------------------------------------
[cons] 1000 -> 2000: 49.950000 ➋
[file] 2000,4000,20.000000 ➍
```

*列表 5-13：使用属性注入重构列表 5-12*

`set_logger` 方法使你能够在 `Bank` 对象的生命周期中的任何时刻注入新的日志记录器。当你将日志记录器设置为 `ConsoleLogger` 实例 ➊ 时，你会在日志输出中得到一个 `[cons]` 前缀 ➋。当你将日志记录器设置为 `FileLogger` 实例 ➌ 时，你会得到一个 `[file]` 前缀 ➍。

#### *选择构造函数注入或属性注入*

无论选择构造函数注入还是属性注入，取决于设计需求。如果你需要能够在对象生命周期内修改对象成员的基础类型，你应该选择指针和属性注入方法。但使用指针和属性注入的灵活性是有代价的。在本章中的 `Bank` 示例中，你必须确保不要将 `logger` 设置为 `nullptr`，或者在使用 `logger` 之前检查这个条件。还有一个问题是默认行为是什么：`logger` 的初始值是多少？

一种可能的做法是提供构造函数注入和属性注入。这鼓励任何使用你的类的人考虑如何初始化它。列表 5-14 说明了实现这种策略的一种方式。

```
#include <cstdio>
struct Logger {
  --snip--
};

struct Bank {
  Bank(Logger* logger) : logger{ logger }{} ➊
  void set_logger(Logger* new_logger) { ➋
    logger = new_logger;
  }
  void make_transfer(long from, long to, double amount) {
    if (logger) logger->log_transfer(from, to, amount);
  }
private:
    Logger* logger;
};
```

*代码清单 5-14：对 `Bank` 的重构，包含构造函数和属性注入*

如你所见，你可以包括一个构造函数 ➊ 和一个 setter ➋。这要求 `Bank` 的用户初始化日志记录器，哪怕是 `nullptr`。之后，用户可以通过属性注入轻松更换这个值。

### 总结

在本章中，你学习了如何定义接口、虚函数在使继承有效方面扮演的核心角色，以及一些使用构造函数和属性注入器的通用规则。无论你选择哪种方法，接口继承与组合的结合为大多数运行时多态应用提供了足够的灵活性。你可以以几乎没有开销的方式实现类型安全的运行时多态性。接口鼓励封装和松耦合设计。通过简单、专注的接口，你可以通过使代码跨项目可移植，来促进代码重用。

**练习**

**5-1.** 你没有在你的 `Bank` 中实现一个会计系统。设计一个名为 `AccountDatabase` 的接口，能够在银行账户中获取和设置金额（通过 `long` 类型的 id 来标识账户）。

**5-2.** 生成一个实现了 `AccountDatabase` 的 `InMemoryAccountDatabase`。

**5-3.** 在 `Bank` 中添加一个 `AccountDatabase` 引用成员。使用构造函数注入将 `InMemoryAccountDatabase` 添加到 `Bank`。

**5-4.** 修改 `ConsoleLogger` 以接受一个 `const char*` 类型的参数进行构造。当 `ConsoleLogger` 进行日志记录时，将该字符串添加到日志输出的前面。注意，你可以在不修改 `Bank` 的情况下修改日志记录行为。

**进一步阅读**

+   *C++ API 设计* 作者：Martin Reddy（Elsevier，2011）
