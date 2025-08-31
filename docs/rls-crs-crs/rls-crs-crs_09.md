# 第七章. 高级 Ruby

你在第一章学习了 Ruby 的基本概念。本章涵盖了一些语言的高级特性，包括模块、Ruby 对象模型、反射以及一些元编程。

模块在 Rails 应用程序中经常用于将类似的功能分组，并在类之间共享行为。Ruby 对象模型决定了方法如何在继承类的层次结构中查找和调用，以及如何从模块共享的代码中查找方法。反射通过允许你查看类的内部结构，从而支持多态性，帮助你了解类能理解哪些方法。元编程允许你的类在运行时通过定义方法来响应那些不存在的方法。

打开终端窗口并启动 IRB 开始实验。本章中的一些例子较长，输入时可能比平常更困难。你可以将代码输入到编辑器中，将其保存为扩展名为*rb*的文件，然后通过在终端输入`ruby` *`filename.rb`*来运行示例。或者，你也可以直接将代码从编辑器复制粘贴到 IRB 中。

# 模块

正如你在第一章中看到的，模块是一个方法和常量的集合，不能实例化。你定义模块的方式基本上和定义类相同。模块定义以`module`开头，后跟一个大写字母的名称，然后以`end`结束。

为了展示如何使用模块，首先我们需要定义一个类。让我们定义一个简单的`Person`类：

```
   class Person
➊    attr_accessor :name

➋    def initialize(name)
       @name = name
     end
   end
```

这个类使用`attr_accessor` ➊来定义实例变量`@name`的获取器和设置器，并在创建时设置`@name`的值 ➋。

类名通常是名词，因为它们代表对象。模块名通常是形容词，因为它们代表行为。许多 Ruby 模块在此约定的基础上更进一步，使用以*able*结尾的形容词命名，例如`Comparable`和`Forwardable`。

这里有一个简单的例子，展示如何使用模块：

```
module Distractable
  def distract
    puts "Ooh, kittens!"
  end
end
```

在 IRB 中输入这个模块，将其包含到你在本章早些时候创建的`Person`类中，看看你是否能分散某人的注意力：

```
irb(main):001:0> **class Person**
irb(main):002:1> **include Distractable**
irb(main):003:1> **end**
 => Person
irb(main):004:0> **p = Person.new("Tony")**
 => #<Person:0x007fceb1163de8 @name="Tony">
irb(main):005:0> **p.distract**
Ooh, kittens!
=> nil
```

在第五章中，你也在使用 Rails 辅助方法时定义了一个模块方法。`ApplicationHelper`是一个模块，Rails 会自动将其混入所有控制器中。

模块在 Ruby 中有两个用途：

+   模块用于将相关方法分组并防止名称冲突。

+   模块定义了可以混入类中的方法，以提供额外的行为。

随着应用程序的成长，组织代码变得越来越重要。通过提供命名空间并使类之间的代码共享变得容易，模块帮助你将代码拆分成可管理的部分。让我们看看这两个用途。

## 模块作为命名空间

Ruby 模块可以用作*命名空间*，即包含常量或相关功能方法的代码容器。

`Math`模块是一个作为命名空间使用的内建 Ruby 模块。它定义了常量`E`和`PI`以及许多常见的三角函数和超越函数方法。双冒号运算符（`::`）用于访问 Ruby 中的常量。以下示例访问`Math`模块中的常量`PI`：

```
irb(main):006:0> **Math::PI**
 => 3.141592653589793
```

在模块中定义的方法通过点（`.`）运算符访问，就像类中的方法一样：

```
irb(main):007:0> **Math.sin(0)**
 => 0.0
```

## 模块作为混入

Ruby 模块也可以作为*混入*，为类提供额外的功能。Ruby 只支持单一继承，即一个类只能继承一个父类。模块使你能够实现类似于多重继承的功能：一个类可以包含多个模块，将每个模块的方法添加到自己的类中。

你可以通过三种方式将模块的方法添加到类中，使用`include`、`prepend`或`extend`。接下来，我将讨论这些关键字的效果。

### include

`include`语句将模块的方法作为实例方法添加到类中，是将模块混入类中的最常见方式。

`Comparable`模块是 Ruby 中常用的混入模块。它在包含时将比较运算符和`between?`方法添加到类中。类只需要实现`<=>`运算符。该运算符比较两个对象，并根据接收者是否小于、等于或大于另一个对象，返回`–1`、`0`或`1`。

要将此模块作为混入模块使用，添加它到你之前创建的`Person`类中：

```
   class Person
➊    include Comparable

➋    def <=>(other)
       name <=> other.name
     end
   end
```

该类现在包含了`Comparable`模块➊，并定义了`<=>`运算符➋，用于将该对象的名称与另一个对象的名称进行比较。

在 IRB 中输入这个命令后，创建一些人物并检查它们是否能相互比较：

```
irb(main):008:0> **p1 = Person.new("Tony")**
 => #<Person:0x007f91b40140a8 @name="Tony">
irb(main):009:0> **p2 = Person.new("Matt")**
 => #<Person:0x007f91b285fea8 @name="Matt">
irb(main):010:0> **p3 = Person.new("Wyatt")**
 => #<Person:0x007f91b401fb88 @name="Wyatt">
irb(main):011:0> **p1 > p2**
 => true
```

在这里，`p1`大于`p2`，因为*T*在字母顺序上大于*M*。`between?`方法告诉你一个对象是否位于另两个对象之间：

```
irb(main):012:0> **p1.between? p2, p3**
 => true
```

在这个例子中，`between?`返回`true`，因为*T*在字母表顺序上位于*M*和*W*之间，这意味着它按预期工作。

### prepend

`prepend`语句也将模块的方法添加到类中，但`prepend`将模块的方法*插入到*类的方法之前。这意味着，如果模块定义了与类同名的方法，模块的方法将优先执行而不是类的方法。通过使用`prepend`，你可以通过在模块中编写同名方法来覆盖类中的方法。

`prepend`的一个实际用法是记忆化。*记忆化*是一种优化技术，程序将计算结果存储起来，以避免多次重复相同的计算。

例如，假设你想在 Ruby 中实现斐波那契数列。斐波那契数列的前两个数字是零和一。每个后续数字是前两个数字的和。以下是在 Ruby 中计算斐波那契数列第*n*项的方法：

```
  class Fibonacci
    def calc(n)
      return n if n < 2
➊     return calc(n - 1) + calc(n - 2)
    end
  end
```

注意 `calc` 方法是递归的。每次用大于 1 的 `n` 值调用 `calc` 时，都将导致对自身的两次调用 ➊。试着创建这个类的实例并计算一些小的 `n` 值：

```
irb(main):013:0> **f = Fibonacci.new**
 => #<Fibonacci:0x007fd8d3269518>
irb(main):014:0> **f.calc 10**
 => 55
irb(main):015:0> **f.calc 30**
 => 832040
```

当你对更大的 `n` 值调用该方法时，方法执行的时间会显著增加。对于约 40 的 `n` 值，方法需要几秒钟才能返回一个答案。

Fibonacci 的 `calc` 方法很慢，因为它重复进行相同的计算多次。但是，如果你定义一个模块来实现记忆化，计算应该会显著缩短时间。现在我们来做这个：

```
   module Memoize
     def calc(n)
➊      @@memo ||= {}
➋      @@memo[n] ||= super
     end
   end
```

`Memoize` 模块也定义了一个 `calc` 方法。这个方法有几个有趣的特点。首先，如果尚未初始化，它会初始化一个名为 `@@memo` ➊ 的类变量为空哈希表。这个哈希表存储每个 `n` 值对应的 `calc` 方法的结果。接着，如果该值尚未分配，它会将 `super` 的返回值赋给 `@@memo` 中键为 `n` 的位置 ➋。因为我们使用 `prepend` 将这个模块添加到 `Fibonacci` 中，`super` 会调用类定义的原始 `calc` 方法。

每次调用 `calc` 方法时，`@@memo` 会存储 `n` 值对应的 Fibonacci 数字。例如，在调用 `calc(3)` 后，`@@memo` 哈希表会包含如下键值对：

```
{
  0 => 0,
  1 => 1,
  2 => 1,
  3 => 2
}
```

在每一行中，键（第一个数字）是 `n` 的值，值（第二个数字）是对应的 Fibonacci 数字。Fibonacci 数字对于 0 是 0，对于 1 是 1，对于 2 是 1，对于 3 是 2。通过存储这些中间值，`calc` 方法就不需要重复计算相同的值。使用 `prepend Memoize` 将 `Memoize` 模块添加到 `Fibonacci` 类中，并试试看：

```
irb(main):016:0> **class Fibonacci**
irb(main):017:1> **prepend Memoize**
irb(main):018:1> **end**
 => Fibonacci
irb(main):019:0> **f.calc 40**
 => 102334155
```

现在 `calc` 的值已经被记忆化，你应该能够对更大的 `n` 值调用 `calc` 并几乎立刻得到答案。试试 `n` = 100 或甚至 `n` = 1000。注意，你不需要重启 IRB 或实例化一个新的 Fibonacci 对象。Ruby 中的方法查找是动态的。

### extend

当你使用 `include` 或 `prepend` 将一个模块添加到类中时，模块的方法会作为实例方法被添加到类中。在第一章中，你学习了也有一些类方法，它们是直接在类上调用，而不是在类的实例上调用。`extend` 语句将模块的方法作为类方法添加到类中。使用 `extend` 可以将行为添加到类本身，而不是类的实例。

Ruby 标准库包含一个名为 `Forwardable` 的模块，你可以使用它来扩展一个类。`Forwardable` 模块包含了对委托很有用的方法。*委托*意味着依赖另一个对象来处理一组方法调用。委托是一种通过将某些方法调用的责任分配给另一个类来重用代码的方式。

例如，假设有一个名为 `Library` 的类，用来管理一本书的集合。我们将书籍存储在一个名为 `@books` 的数组中：

```
class Library
  def initialize(books)
    @books = books
  end
end
```

我们可以存储我们的书籍，但目前还无法对它们做任何操作。我们可以使用 `attr_accessor` 来使 `@books` 数组在类外部可用，但那样会让数组的所有方法对类的使用者开放。这样，用户就可以调用诸如 `clear` 或 `reject` 等方法，将图书馆中的所有书籍移除。

让我们将一些方法委托给 `@books` 数组，以提供我们需要的功能——获取图书馆大小和添加书籍的方法。

```
1 require 'forwardable'
  class Library
2   extend Forwardable
3   def_delegators :@books, :size, :push

    def initialize(books)
      @books = books
    end
  end
```

`Forwardable` 模块在 Ruby 标准库中，而不是 Ruby 核心库中，因此我们首先需要 `require` 它 ➊。接着，我们使用 `extend` 将 `Forwardable` 方法添加到我们的类中作为类方法 ➋。最后，我们可以调用 `def_delegators` 方法 ➌。这个方法的第一个参数是一个符号，表示我们要委托方法的实例变量。

在这种情况下，实例变量是 `@books`。其余的参数是表示我们要委托的方法的符号。`size` 方法返回数组中元素的数量。`push` 方法将一个新元素追加到数组的末尾。

在下面的例子中，`lib.size` 初始值为 2，因为我们图书馆中有两本书。添加一本书后，大小更新为 3。

```
irb(main):020:0> **lib = Library.new ["Neuromancer", "Snow Crash"]**
 => #<Library:0x007fe6c91854e0 @books=["Neuromancer", "Snow Crash"]>
irb(main):021:0> **lib.size**
 => 2
irb(main):022:0> **lib.push "The Hobbit"**
 => ["Neuromancer", "Snow Crash", "The Hobbit"]
irb(main):023:0> **lib.size**
 => 3
```

# Ruby 对象模型

*Ruby 对象模型* 解释了 Ruby 在调用方法时如何查找该方法。在继承和模块的情况下，你可能会想知道某个方法到底是在哪里定义的，或者在有多个同名方法的情况下，哪一个方法是由特定调用实际调用的。

## 祖先

继续使用之前定义的简单 `Person` 类，我们可以在 IRB 中了解关于这个类的许多信息。首先，让我们看看哪些类和模块定义了 `Person` 类的方法：

```
irb(main):024:0> **Person.ancestors**
 => [Person, Distractable, Comparable, Object, Kernel, BasicObject]
```

类方法 `ancestors` 返回 `Person` 类继承的类和它包含的模块的列表。在这个例子中，`Person`、`Object` 和 `BasicObject` 是类，而 `Distractable`、`Comparable` 和 `Kernel` 是模块。你可以通过调用 `class` 方法来找出这些是类还是模块，具体内容在下面的 *Class* 部分会解释。

`Object` 是所有 Ruby 对象的默认根类。`Object` 继承自 `BasicObject` 并混入了 `Kernel` 模块。`BasicObject` 是 Ruby 中所有类的父类。你可以把它看作是一个空白类，所有其他类都建立在这个类之上。`Kernel` 定义了许多 Ruby 方法，这些方法在没有接收者的情况下调用，比如 `puts` 和 `exit`。每次你调用 `puts` 时，实际上是在调用 `Kernel` 模块中的实例方法 `puts`。

这个列表的顺序表示 Ruby 查找方法的顺序。Ruby 首先会在 `Person` 类中查找方法定义，然后继续在列表中查找，直到找到该方法。如果 Ruby 没有找到该方法，它会抛出一个 `NoMethodError` 异常。

## 方法

你可以通过分别调用`methods`和`instance_methods`来查看类定义的类方法和实例方法。这些列表默认包含所有父类定义的方法。传递参数`false`可以仅排除这些方法：

```
irb(main):025:0> **Person.methods**
 => [:allocate, :new, :superclass, :freeze, :===, :==, ... ]
irb(main):026:0> **Person.methods(false)**
 => []
irb(main):027:0> **Person.instance_methods(false)**
 => [:name, :name=, :<=>]
```

`Person`类包含了几乎 100 个从其祖先类继承的类方法，但它自己并没有定义任何类方法，因此调用`methods(false)`会返回一个空数组。调用`instance_methods`会返回由`attr_accessor`定义的`name`和`name=`方法，以及我们在类体内定义的`<=>`方法。

## 类

对象模型的最后一部分涉及到`Person`类本身。在 Ruby 中，一切都是对象，也就是说，它是某个类的实例。因此，`Person`类必须是某个类的实例。

```
irb(main):028:0> **Person.class**
 => Class
```

所有 Ruby 类都是`Class`类的实例。定义一个类，例如`Person`，实际上是创建了`Class`类的一个实例，并将其赋值给一个全局常量，这里是`Person`。`Class`类中最重要的方法是`new`，它负责为新对象分配内存并调用`initialize`方法。

`Class`有自己的祖先列表：

```
irb(main):029:0> **Class.ancestors**
 => [Class, Module, Object, Kernel, BasicObject]
```

`Class`继承自`Module`类，`Module`类继承自`Object`类。`Module`类包含了本节中使用的多个方法的定义，例如`ancestors`和`instance_methods`。

# 反射

*反射*，也叫做*自省*，是指程序运行时，能够检查对象的类型及其他属性。你已经看到过如何通过调用`class`来确定对象的类型，以及如何通过调用`methods`和`instance_methods`来获取对象定义的方法列表，但 Ruby 的`Object`类还定义了几个用于自省对象的方法。例如，给定一个对象，你可能想确定它是否属于某个特定的类：

```
irb(main):030:0> **p = Person.new("Tony")**
 => #<Person:0x007fc0ca1a6278 @name="Tony">
irb(main):031:0> **p.is_a? Person**
 => true
```

如果给定的类是接收对象的类，`is_a?`方法会返回`true`。在这个例子中，它返回`true`，因为对象`p`是`Person`类的实例。

```
irb(main):032:0> **p.is_a? Object**
 => true
```

如果给定的类或模块是接收对象的祖先类，`is_a?`方法也会返回`true`。在这个例子中，`Object`是`Person`的祖先，所以`is_a?`返回`true`。

如果你需要准确判断创建一个对象时使用了哪个类，可以使用`instance_of?`方法：

```
irb(main):033:0> **p.instance_of? Person**
 => true
irb(main):034:0> **p.instance_of? Object**
 => false
```

`instance_of?`方法只有在接收对象是给定类的实例时才返回`true`。对于祖先类和继承自给定类的类，这个方法返回`false`。这种自省方式在某些情况下很有用，但通常你不需要知道创建对象时使用的具体类——只需要知道对象的能力。

# 鸭子类型

在*鸭子类型*中，你只需要知道一个对象是否接受你需要调用的方法。如果对象能响应所需的方法，你就不必关心类名或继承关系。鸭子类型的名字来源于那句话：“如果它走起来像鸭子，叫起来像鸭子，那就把它叫做鸭子。”

在 Ruby 中，你可以使用`respond_to?`方法查看一个对象是否响应某个特定的方法。如果`respond_to?`返回`false`，那么调用该方法会抛出`NoMethodError`异常，如前所述。

例如，想象一个简单的方法，用来将带有时间戳的信息打印到文件中：

```
def write_with_time(file, info)
  file.puts "#{Time.now} - #{info}"
end
```

你可以在 IRB 中尝试这个方法。

```
➊ irb(main):001:0> **f = File.open("temp.txt", "w")**
   => #<File:temp.txt>
➋ irb(main):002:0> **write_with_time(f, "Hello, World!")**
   => nil
➌ irb(main):003:0> **f.close**
   => nil
```

首先，打开当前目录下名为*temp.txt*的`File`并将`File`实例存储在变量`f`中 ➊。然后，将`f`和消息`"Hello, World!"`传递给`write_with_time`方法 ➋。最后，使用`f.close`关闭`File` ➌。

当前目录下的文件*temp.txt*现在包含类似于下面这一行的内容：

```
2014-05-21 16:52:07 -0500 - Hello, World!
```

这个方法运行得很好，直到有人不小心传递了一个不是文件的值，比如`nil`。这是可能的修复方案：

```
  def write_with_time(file, info)
➊  if file.instance_of? File
      file.puts "#{Time.now} - #{info}"
    else
      raise ArgumentError
    end
  end
```

这个修复通过检查`file`是否是`File`类的实例来解决问题 ➊，但它也限制了这个方法的适用性。现在它*仅仅*适用于文件。如果你想通过`Socket`写入网络，或者使用`STDOUT`写入控制台，怎么办呢？

与其测试`file`的*类型*，不如测试它的*功能*：

```
  def write_with_time(file, info)
➊   if file.respond_to?(:puts)
      file.puts "#{Time.now} - #{info}"
    else
      raise ArgumentError
    end
  end
```

你知道`write_with_time`方法调用了`puts`方法，所以检查`file`是否响应`puts`方法 ➊。现在，`write_with_time`可以与任何响应`puts`方法的数据类型一起使用。

使用鸭子类型编程可以使代码更加易于复用。在构建应用程序时，寻找更多应用鸭子类型编程的机会。

# 元编程

*元编程*是编写与代码而非数据打交道的代码的实践。在 Ruby 中，你可以编写代码，在运行时定义新的行为。本节中的技术可以节省时间并消除代码中的重复，允许 Ruby 在程序加载时或运行时生成方法。

本节介绍了两种动态定义方法的不同方式：`define_method`和`class_eval`。它还涉及了`method_missing`，使你能够响应那些未定义的方法。

## define_method

假设我们有一个应用程序，其中包含可以为用户启用的功能列表。`User`类将这些功能存储在名为`@features`的哈希表中。如果某个用户可以访问某个功能，那么对应的哈希值将为`true`。

我们希望添加形式为`can_` *`feature`*`!` 和 `can_` *`feature`*`?`的方法，分别用于启用某个功能和检查某个功能是否启用。与其编写多个大致相同的方法，不如迭代可用功能的列表，并使用`define_method`来定义这些方法，如下所示：

```
  class User
➊   FEATURES = ['create', 'update', 'delete']

    FEATURES.each do |f|
➋     define_method "can_#{f}!" do
        @features[f] = true
      end

➌     define_method "can_#{f}?" do
➍        !!@features[f]
       end
     end
     def initialize
       @features = {}
     end
   end
```

`User`类首先创建了一个常量数组 ➊，命名为`FEATURES`，其中包含可用的功能。然后，它使用`each`遍历`FEATURES`，并调用`define_method`来创建形如`can_`*`feature`*`!` ➋的方法，允许用户访问某个功能。仍然在`each`块中，类还定义了形如`can_`*`feature`*`?` ➌的方法，用来判断用户是否具有访问该功能的权限。这个方法通过使用两个 NOT 运算符 ➍将`@features[f]`的值转换为`true`或`false`。

### 注意

*使用两个 NOT 运算符并非绝对必要，因为`@features`哈希表对没有值的键返回`nil`，而 Ruby 将`nil`视为`false`，但这种技巧通常被使用。*

现在，让我们创建一个新的`User`并尝试动态定义的方法：

```
irb(main):001:0> **user = User.new**
 => #<User:0x007fc01b95abe0 @features={}>
irb(main):002:0> **user.can_create!**
 => true
irb(main):003:0> **user.can_create?**
 => true
irb(main):004:0> **user.can_update?**
 => false
irb(main):005:0> **user.can_delete?**
 => false
```

如果你想更多地练习`define_method`，看看你能否添加形如`cannot_`*`feature`*`!`的方法，用于禁用用户的某个功能。更多细节可以在本章末的练习 3 中找到。

## class_eval

`class_eval`方法将代码字符串作为类定义中的代码直接执行。使用`class_eval`是向类在运行时添加实例方法的一个简单方法。

当我在第一章中讨论`attr_accessor`时，你了解到它为类中的实例变量定义了 getter 和 setter 方法，但我并没有详细讨论这些方法是如何定义的。`attr_accessor`方法是 Ruby 内置的，你不需要自己定义它，但你可以通过实现自己的`attr_accessor`版本来了解`class_eval`。

```
➊ class Accessor
➋   def self.accessor(attr)
      class_eval "
➌       def #{attr}
          @#{attr}
          end

➍         def #{attr}=(val)
            @#{attr} = val
          end
         "
       end
     end
```

在这里，你定义了一个名为`Accessor`的类 ➊，并且它有一个名为`accessor`的类方法 ➋。这个方法的工作方式类似于内建的`attr_accessor`。它接受一个参数，表示你正在为其创建 getter 和 setter 方法的属性。将字符串传递给`class_eval`，它使用字符串插值将`attr`的值插入到需要的地方，从而定义两个方法。第一个方法的名称与属性相同，并返回属性的值 ➌。第二个方法的名称是属性名后跟一个等号。它将属性设置为指定的值`val` ➍。

例如，如果`attr`是`:name`，那么`accessor`通过将`attr`替换为*name*来定义`name`和`name=`这两个方法。这在没有示例的情况下有些难以理解。以下代码在一个类中使用了`accessor`方法：

```
➊ class Element < Accessor
➋   accessor :name

    def initialize(name)
      @name = name
    end
  end
```

首先，你让`Element`类继承自`Accessor`类 ➊，这样就可以使用`accessor`方法。然后，将实例变量的名称传递给`accessor` ➋。在这里，你传递了符号`:name`。当程序运行时，对`class_eval`的调用会自动在`Element`类中生成如下代码：

```
➊ def name
    @name
  end

➋ def name=(val)
    @name = val
  end
```

`name` 方法返回实例变量 `@name` 的当前值 ➊。`name=` 方法接受一个值并将其赋给 `@name` ➋。通过创建一个 `Element` 类的实例并尝试获取和设置 `name` 的值来测试它：

```
➊ irb(main):001:0> **e = Element.new "lead"**
   => #<Element:0x007fc01b840110 @name="lead">
➋ irb(main):002:0> **e.name = "gold"**
   => "gold"
➌ irb(main):003:0> **puts e.name**
  gold
   => nil
```

首先，创建一个新的 `Element` 并将其名称初始化为 `"lead"` ➊。接下来，使用 `name=` 方法将新名称 `"gold"` 赋给它 ➋。最后，使用 `name` 方法显示 `@name` 的值 ➌。就这样，通过一点元编程的魔法，你将铅变成了金。

## method_missing

每当 Ruby 找不到一个方法时，它会在接收者上调用 `method_missing`。该方法会接收原始方法名（作为符号）、一个参数数组以及传递给方法调用的任何块。

默认情况下，`method_missing` 会调用 `super`，这会将方法向上传递到祖先链，直到找到包含该方法的祖先类。如果方法到达 `BasicObject` 类，它会抛出一个 `NoMethodError` 异常。你可以通过在类中定义自己的实现来覆盖 `method_missing`，拦截这些方法调用并添加自己的行为。

让我们从一个简单的例子开始，这样你就能看到它是如何工作的。这个类会将任何未知的方法调用返回给你三次：

```
class Echo
  def method_missing(name, *args, &block)
    word = name
    puts "#{word}, #{word}, #{word}"
  end
end
```

现在，`method_missing` 被覆盖了，如果你尝试在该类的实例上调用一个不存在的方法，你会在终端中看到该方法的“回音”：

```
irb(main):001:0> **echo = Echo.new**
 => #<Echo:0x007fa8131c9590>
irb(main):002:0> **echo.hello**
 => hello, hello, hello
```

`method_missing` 的一个现实应用是 Rails 的动态查找器。通过使用动态查找器，你可以写出像 `Post.find_by_title("First Post")` 这样的 Active Record 查询，而不是 `Post.where(title: "First Post").first`。

动态查找器可以使用 `method_missing` 实现。让我们定义我们自己的动态查找器版本。我们将使用 `query_by_`*`attribute`* 而不是像 `find_by_`*`attribute`* 这样的方式，这样可以避免与内置方法发生冲突。

打开你博客目录中的 `app/models/post.rb` 文件，按照这个例子继续操作：

```
  class Post < ActiveRecord::Base
    validates :title, :presence => true
    has_many :comments

➊   **def self.method_missing(name, *args, &block)**
➋     **if name =~ /\Aquery_by_(.+)\z/**
➌       **where($1 => args[0]).first**
      **else**
➍       **super**
      **end**
    **end**
 end
```

首先，定义 `method_missing` 类方法 ➊，因为我们的 `query_by_`*`attribute`* 方法将被调用到 `Post` 类上。接下来，测试名称是否符合正则表达式 ➋。

最后，使用正则表达式捕获的字符串和传递给方法的第一个参数来调用内置的 `where` 方法 ➌。如果字符串不匹配，一定要调用 `super` ➍；这确保了未知的方法会被发送到父类。

### 注意

*正则表达式 `/\Aquery_by_(.+)\z/` 匹配以 “query_by_” 开头的字符串，并使用括号捕获字符串的其余部分。正则表达式的全面讨论超出了本书的范围。网站* [`rubular.com/`](http://rubular.com/) *是一个在线编辑和测试正则表达式的好方法。*

真实的动态查找器还会检查捕获的字符串是否与模型的属性匹配。如果你尝试用不存在的列调用我们的 `query_by_`*`attribute`* 方法，它会抛出一个 `SQLException`。

```
irb(main):001:0> **Post.query_by_title "First Post"**
 => #<Post id: 1, ...>
```

我们实现的`query_by_`*`attribute`*还有一个问题：

```
irb(main):002:0> **Post.respond_to? :query_by_title**
 => false
```

因为我们重写了`method_missing`来调用这个方法，Ruby 不知道`Post`类能够响应它。为了解决这个问题，我们还需要在`Post`模型的*app/models/post.rb*中重写`respond_to_missing?`方法。

```
   class Post < ActiveRecord::Base
     --*snip*--

     **def self.respond_to_missing?(name, include_all=false)**
➊      **name.to_s.start_with?("query_by_") || super**
    **end**
  end
```

我们不再使用`method_missing`中的正则表达式，而是检查方法名是否以`"query_by_"`开头 ➊。如果是，这个方法会返回`true`。否则，调用`super`。现在重新启动 Rails 控制台并再次尝试：

```
irb(main):001:0> **Post.respond_to? :query_by_title**
 => true
```

在做出这个改变之后，`respond_to?`按预期返回`true`。记住，在使用`method_missing`时，始终要覆盖`respond_to_missing?`。否则，使用你类的用户就无法知道它接受哪些方法，之前提到的鸭子类型技巧也会失效。

# 总结

如果你写足够多的 Ruby 代码，你最终会在实际的程序中看到本章所介绍的所有技巧。到那时，你可以确信你能理解代码的作用，而不仅仅是认为元编程是一种神奇的东西。

在下一章中，你将从头开始构建一个新的 Rails 应用程序。在这个过程中，我将介绍一些高级数据建模技巧，你还将深入了解 Active Record。

现在，尝试这些练习吧。

# 练习

| 问题： | 1\. Rails 框架广泛使用模块作为命名空间，并向类添加行为。在你的*blog*目录中打开 Rails 控制台，并查看`Post`的祖先类。它有多少个祖先？根据它们的名称，你能猜出它们的作用吗？ |
| --- | --- |
| 问题： | 2\. 更新`define_method`示例，添加一个`cannot_` *`feature`*`!`方法。此方法应将`@features`哈希中对应正确键的值设置为`false`。 |
| 问题： | 3\. 通过调用`Element.instance_methods(false)`验证`class_eval`是否创建了你预期的实例方法。然后重新打开`Element`类，并调用`accessor :symbol`，为名为`@symbol`的实例变量添加两个方法。 |
