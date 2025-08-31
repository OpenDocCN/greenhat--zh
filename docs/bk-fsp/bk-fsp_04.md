## 第四章. 保持面向目标

多年来，*面向对象（OO）*开发一直是开发业务软件，特别是在企业中的事实标准，所以你可能已经熟悉了其中许多核心原则。作为一门 .NET 语言，F# 支持与其他 .NET 语言中相同的各种结构——包括类、结构体和接口——这应该不令人惊讶。尽管 F# 被认为是一种只适用于学术练习或高度专业化软件的利基语言，但其通用、多范式的特性使其适用于大多数开发场景。然而，既然 C# 和 Visual Basic 已经得到了广泛应用，为什么还要选择 F# 作为面向对象语言呢？

做出选择的一大因素是 F# 的简洁语法，但像类型推断、对象表达式以及将面向对象与函数式风格结合的能力等特性也提供了有力的理由。然而，面对现实：即使你主要采用函数式方式进行开发，当你在 .NET 框架上开发软件时，最终还是需要与对象打交道；这就是平台的特性所在。

在本章中，你将学习如何用更少的代码在 F# 中创建面向对象的结构，同时仍然能够构建强大的框架，这些框架能够与使用更专门的面向对象语言构建的类似框架相抗衡。

## 类

从概念上讲，F# 中的类与其他 OO 语言中的类相同，它们将相关的数据和行为封装为字段、属性、方法和事件（统称为*成员*），以模拟现实世界的对象或概念。像 C# 和 Visual Basic 中的类一样，F# 类是引用类型，支持单一继承和多重接口实现，并且可以控制对其成员的访问。与 F# 中的所有用户定义数据类型一样，类是通过 `type` 关键字来声明的。（编译器根据结构推断构造，而不需要为每种数据类型使用不同的关键字。）

为了说明这一点，我们再来看一下在第三章中讨论类型推断时引入的类定义。

```
type Person (id : Guid, name : string, age : int) =
  member x.Id = id
  member x.Name = name
  member x.Age = age
```

这个示例包含了大量的定义。在仅仅四行代码中，就有一个带有*主构造函数*、三个参数和三个隐式只读属性的类！虽然与其他 .NET 语言有很大不同，但这种简洁性正是 F# 区别于其他语言的方式之一。

### 构造函数

构造函数是创建和初始化新类实例的方式。它们实际上是专门的函数，返回完全初始化的类实例。F# 中的类不需要构造函数，如下所示：

```
type ConstructorlessClass = class end
```

这个示例中的空类是有效的 F#代码，但与 C#不同，如果你没有定义构造函数，编译器不会自动生成*默认构造函数*（无参数的构造函数）。由于一个没有成员、无法实例化的类基本没有用处，所以你的类通常会至少有一个构造函数和一个成员。

### 注意

*你可能选择省略构造函数的一个原因是该类型的每个成员都是静态的；也就是说，它适用于类型本身，而非某个单独的实例。稍后我们将在本章中详细讨论静态成员。*

与其他面向对象（OO）语言一样，你可以通过调用构造函数来创建新的类实例。在我们的`Person`类中只有一个构造函数，所以选择是明确的。

```
let me = **Person(Guid.NewGuid(), "Dave", 33)**
```

使用`new`关键字来创建新的类实例是可选的。按照约定，只有在创建实现了`IDisposable`接口的类实例时，你才会使用`new`关键字。

F#构造函数有两种类型：主构造函数和附加构造函数。

#### 主构造函数

F#类可以拥有一个*主构造函数*，其参数直接嵌入在类型定义中。主构造函数的主体包含一系列`let`和`do`绑定，表示类的字段定义和初始化代码。

```
type Person  ①(name : string, dob : System.DateTime) =
② let age = (System.DateTime.Now - dob).TotalDays / 365.25
③ do printfn "Creating person: %s (Age: %f)" name age
  member x.Name = name
  member x.DateOfBirth = dob
  member x.Age = age
```

在这个示例中，主构造函数包含带有类型注解的参数列表①，一个用于计算年龄的单一字段定义②，以及一个`do`绑定③，在对象构造时打印出人的姓名和年龄。主构造函数的所有参数都会自动作为字段在整个类中使用，因此不需要显式地映射它们。

编译器通常可以推断每个构造函数参数的类型，因此通常无需包含显式的类型注解。在前面的示例中，`dob`参数仍然需要一个类型注解（或者在中间绑定上使用类型注解），这样编译器才能解析正确的减法运算符重载。然而，这种情况更多是例外而非规则，正如下一个示例所示，编译器可以推断`name`和`age`参数的类型分别为`string`和`int`。

```
type Person (name, age) =
  do printfn "Creating person: %s (Age: %i)" name age
  member x.Name = name
  member x.Age = age

let me = Person ("Dave", 33)
```

默认情况下，主构造函数是公开的，但你可以通过在参数列表前添加访问修饰符来更改这一点。如果你在实现*单例模式*时，可能会考虑更改主构造函数的可访问性，单例模式规定类型只能有一个实例，如下所示：

```
type Greeter **private** () =
  static let _instance = lazy (Greeter())
  static member Instance with get() = _instance.Force()
  member x.SayHello() = printfn "hello"

Greeter.Instance.SayHello()
```

关于 F#中的可访问性更多内容

*访问修饰符* 限制了绑定、类型和成员在整个程序中的作用域。F#与 C#和 Visual Basic 不同，F#仅直接支持`public`、`private`和`internal`修饰符。在 F#中不能定义`protected`类成员，部分原因是`protected`成员会使语言的函数式特性变得复杂。尽管如此，F#仍然会遵循在其他语言中定义的`protected`成员，因此它们不会公开访问，并且你仍然可以在派生类中重写它们，而不会破坏抽象。

#### 额外的构造函数

你在主构造函数之外定义的构造函数称为*额外构造函数*。额外的构造函数使用`new`关键字定义，后面跟着参数列表和构造函数体，如下所示。虽然额外的构造函数必须始终调用主构造函数，但它们可以通过另一个构造函数间接调用，从而允许你链式调用构造函数。

```
type Person (name, age) =
  do printfn "Creating person: %s (Age: %i)" name age
  **new (name) = Person(name, 0)**
  **new () = Person("")**
  member x.Name = name
  member x.Age = age
```

额外的构造函数可以包含它们自己的`let`绑定和其他表达式，但与主构造函数中的不同，任何此类元素都将局部于定义它们的构造函数，而不是作为字段暴露出来。

额外的构造函数可以像主构造函数一样调用额外的代码，但它们使用`then`关键字，而不是`do`绑定。在这个例子中，每个额外的构造函数都包含`then`关键字，以便打印一条消息，指示哪个构造函数被调用。

```
type Person (name, age) =
  do printfn "Creating person: %s (Age: %i)" name age
  new (name) = Person(name, 0)
               **then printfn "Creating person with default age"**
  new () = Person("")
           **then printfn "Creating person with default name and age"**
  member x.Name = name
  member x.Age = age
```

没有主构造函数的类在初始化时表现得稍微不同。当你使用它们时，必须显式定义字段，使用`val`关键字，任何额外的构造函数必须初始化没有使用`DefaultValue`属性修饰的字段，如下所示：

```
type Person =
  val _name : string
  val _age : int
  new (name, age) = { _name = name; _age = age }
  new (name) = Person(name, 0)
  new () = Person("")
  member x.Name = x._name
  member x.Age = x._age
```

#### 自引用标识符

有时候你可能希望在构造函数中引用类成员。默认情况下，类成员是不可访问的，因为它们需要递归引用类型，但你可以通过`as`关键字和*自引用标识符*来启用自引用，如下所示：

```
type Person (name, age) **as this** =
  do printfn "Creating person: %s (Age: %i)" this.Name this.Age
  member x.Name = name
  member x.Age = age
```

你可以为自引用标识符选择任何名称，只要你遵循标识符的常规规则。如果你真的想激怒未来的自己或任何维护你代码的人，甚至可以使用像下面这样的带引号的标识符。

```
type Person (name, age) as **``This is a bad identifier``** =
  do
    printfn "Creating person: %s (Age: %i)"
      **``This is a bad identifier``.Name**
      **``This is a bad identifier``.Age**
      member x.Name = name
      member x.Age = age
```

通常最好坚持使用简短的名称。常见的约定是使用`x`或`this`。但无论你选择什么，记得保持一致！

### 警告

*如果你定义了一个自引用标识符，但在构造函数中没有使用它，编译器将生成警告。原因是使用`as`关键字使得类定义变得递归，这会导致额外的运行时验证，从而可能对类层次结构中的类型初始化产生负面影响。只有在真正需要时，才在主构造函数中使用自引用标识符。*

### 字段

字段定义了与对象相关联的数据元素。在前一部分中，我们简要地了解了两种创建字段的方式。在本节中，我们将更详细地讨论字段的创建。

#### let 绑定

创建字段的第一种方式是使用主构造函数中的`let`绑定。这些字段必须在主构造函数中初始化，并且*总是*对类是私有的。尽管它们在创建时必须初始化，但你可以像任何`let`绑定一样使其值可变，如下所示：

```
type Person () =
  **let mutable name : string = ""**
  member x.Name
    with get() = name
    and set(v) = name <- v
```

在这里，使用可变的`let`绑定来定义`Name`属性的后备存储。

#### 显式字段

当你想对字段进行更多控制，或者你的类没有主构造函数时，可以使用`val`关键字创建显式字段。显式字段不需要立即初始化，但在具有主构造函数的类中，你需要使用`DefaultValue`属性对它们进行修饰，以确保它们被初始化为适当的“零”值，如下所示：

```
type Person () =
  **[<DefaultValue>] val mutable n : string**
  member x.Name
    with get() = x.n
    and set(v) = x.n <- v
```

在这个例子中，`n`是一个显式字段。由于`n`的类型是`string`，它被初始化为`null`，如你所见：

```
> **let p = Person()**
p.Name;;

val p : Person
val it : string = null
```

显式字段默认是公共的，但你可以通过在定义中包含`private`访问修饰符将它们设置为私有，如下所示：

```
type Person () =
  [<DefaultValue>] val mutable **private** n : string
  -- *snip* --
```

### 属性

和字段一样，*属性*表示与对象相关联的数据。不过与字段不同的是，属性提供了更多的控制，允许你通过某种组合的`get`和/或`set`函数（统称为*访问器*）来控制数据的访问或修改方式。

你可以隐式或显式地定义属性。一个指导原则是，当你暴露一个简单值时，优先使用隐式属性；当你需要在获取或设置属性值时使用自定义逻辑时，改用显式属性。

#### 显式属性

显式属性是指你定义并控制后备存储（通常使用`let`绑定），并且自己实现`get`和`set`函数体。你可以使用`member`关键字定义一个显式属性，后跟自引用、属性名称、类型注解（如果编译器无法推断出类型），以及函数体，如下所示：

```
type Person() =
  **let mutable name = ""**
  **member x.Name**
    **with get() = name**
    **and set(value) = name <- value**
```

在这个例子中，`name`字段是`Name`属性的私有后备存储。一旦你创建了这个`Person`类的实例，就可以像下面这样使用赋值运算符给`Name`属性赋值：

```
let me = Person()
me.Name <- "Dave"
```

你可以使用另一种语法来代替`and`关键字，在这种语法中，`get`和`set`访问器被定义为独立的属性。

```
type Person() =
  let mutable name = ""
  **member x.Name with get() = name**
  **member x.Name with set(value) = name <- value**
```

无论你选择哪种语法，属性默认是公共的，但你可以通过在`with`（或`and`）关键字后插入访问修饰符（`public`、`private`或`internal`）来控制它们的可访问性，如下所示：

```
type Person() =
  let mutable name = ""
  member x.Name
    with **public** get() = name
    and **internal** set(value) = name <- value
```

如果你希望`Name`属性为只读属性，可以通过将值作为主构造函数的一个参数，并去除`and set...`这一行来修改类，方法如下：

```
type Person(name) =
  member x.Name with get() = name
```

当然，这是 F#，所以尽管定义只读属性已经很简单，但通过显式语法还有一种更简单的方式。

```
type Person(name) =
  member x.Name = name
```

在创建只读属性时，编译器会自动为你生成`get`访问器函数。

#### 隐式属性

隐式或自动属性在 F# 3.0 版本中引入（如果你使用的是 2.0 版本，则需要使用显式属性）。它们与 C#中的自动实现属性非常相似，允许编译器生成适当的后备存储和相应的`get`/`set`访问器主体。隐式属性与显式属性非常相似，但有一些区别。

首先，隐式属性被视为类型初始化的一部分，因此必须出现在其他成员定义之前，通常与主构造函数一起定义。接下来，它们通过`member val`关键字对进行定义，并且必须初始化为默认值，如下所示。（它们不能包含自引用标识符。）最后，它们的访问级别只能在属性级别更改，而不能在访问器级别更改。

```
type Person() =
  **member val Name = "" with get, set**
```

如果你的隐式属性是只读的，你可以像这样省略`with`表达式：

```
type Person(name) =
  **member val Name = name**
```

#### 索引属性

F# 类也可以拥有*索引属性*，这些属性对于定义一个类似数组的接口以处理顺序数据非常有用。索引属性的定义方式与普通属性相似，不同之处在于`get`访问器包含一个参数。

在创建索引属性时，命名为`Item`会使其成为*默认索引属性*，并通过点操作符和一对括号来支持便捷的语法（`.[...]`）。例如，考虑一个接受字符串并通过默认索引器暴露每个单词的类，像这样：

```
type Sentence(initial : string) =
  let mutable words = initial.Split ' '
  let mutable text = initial
  **member x.Item**
    **with get i = words.[i]**
    **and set i v =**
      **words.[i] <- v**
      **text <- System.String.Join(" ", words)**
```

请注意，`Item`属性定义方式与普通属性相似，包含`get`，甚至是`set`访问器。因为这个索引器只是`words`数组（`String.Split`返回一个数组）的一个封装，它接受一个整数值并返回对应的单词。

F# 数组是基于零索引的，因此你可以像这样从一个句子中获取第二个单词：

```
> let s = Sentence "Don't forget to drink your Ovaltine"
**s.[1];;**

val s1 : Sentence
val it : string = "forget"
```

要更改第二个单词，你可以以相同的方式引用索引，并使用赋值操作符（`<-`）如下所示：

```
> **s.[1] <- "remember";;**
val it : unit = ()
> **s.[1];;**
val it : string = "remember"
```

此外，默认的索引属性可以是多维的。例如，你可以定义一个属性，通过包含两个参数来返回一个单词中的特定字符。

```
type Sentence(initial : string) =
  -- *snip* --
  member x.Item with get(w, i) = words.[w].[i]
```

现在，你可以轻松地像这样获取第二个单词的第一个字符：

```
> **s.[1, 0];;**
val it : char = 'f'
```

那么，如果你想定义另一个索引属性来获取原始字符串中的某个字符该怎么办？你已经定义了一个接受整数的默认索引属性，所以不能那样做。在 C#中，你必须将其创建为一个方法，但在 F#中，任何属性都可以是一个索引属性。例如：

```
type Sentence(initial : string) =
  -- *snip* --
  member x.Chars with get(i) = text.[i]
```

唯一的注意事项是，你不能像使用默认索引属性时那样使用点/括号语法；你必须将属性当作方法来访问（如实例方法所描述），并通过在属性名称后面加上括号内的索引值来访问：

```
> **s.Chars(0);;**
val it : char = 'D'
```

虽然它看起来像是一个方法调用，但如果`Chars`索引属性包含一个`set`访问器，你会像操作其他属性一样使用赋值运算符来改变底层的值。

#### 初始化时设置

一种替代的对象初始化语法允许你在构造函数调用中直接设置各个属性的值。要使用这种对象初始化语法，你只需在正常构造函数参数之后，紧接着写出每个属性的名称和值（用等号分隔）。我们可以通过重新考虑之前的`Person`类示例来说明这一点。

```
type Person() =
  member val Name = "" with get, set
```

因为`Person`类只有一个单一的无参构造函数，所以你可以先创建一个实例，然后在第二个操作中给`Name`属性赋值。但要一次性完成这一切会更加简洁，像这样：

```
let p = Person(Name = "Dave")
```

使用这种语法时有一个注意点：你初始化的任何属性必须是可写的。

### 方法

方法是与类关联的函数，表示该类型的行为。

#### 实例方法

定义实例方法有两种方式。第一种方式使用`member`关键字定义公共方法，这与定义属性的方式类似，如下面的`GetArea`方法所示。

```
open System

type Circle(diameter : float) =
  member x.Diameter = diameter
  **member x.GetArea() =**
    **let r = diameter / 2.0**
    **System.Math.PI * (r ** 2.0)**
```

这里，`Circle`类通过一个`diameter`值初始化，并包含一个无参的公共方法`GetArea`，该方法计算圆的面积。因为`GetArea`是实例方法，你需要创建一个`Circle`类的实例才能像下面这样调用它：

```
> **let c = Circle 5.0**
**c.GetArea();;**

val c : Circle
val it : float = 19.63495408
```

#### 方法可访问性

与属性一样，你可以通过访问修饰符来控制方法的访问权限。例如，要将一个方法设置为私有，只需在方法签名中加入`private`关键字，如下所示的`GetRadius`方法：

```
type Circle(diameter : float) =
  member **private** x.GetRadius() = diameter / 2.0
  member x.Diameter = diameter
  member x.GetArea() = System.Math.PI * (x.GetRadius() ** 2.0)
```

另外，你也可以使用`let`绑定来定义一个私有函数，如下所示：

```
type Circle(diameter : float) =
  **let getRadius() = diameter / 2.0**
  member x.Diameter = diameter
  member x.GetArea() = System.Math.PI * (getRadius() ** 2.0)
```

#### 命名参数

当你调用一个方法时，通常会提供一个以逗号分隔的参数列表，每个参数对应于相同位置的参数。为了提供一些额外的灵活性，F# 允许对方法和构造函数使用*命名参数*。通过命名参数，每个参数都通过名称显式地与特定参数关联。在某些情况下，命名参数有助于澄清你的代码，但它们也允许你按任意顺序指定参数。

以下示例包含一个计算三维空间中两点之间欧几里得距离（准确来说是 RGB 颜色）的函数。

```
open System
open System.Drawing

type ColorDistance() =
  member x.GetEuclideanDistance(c1 : Color, c2 : Color) =
    let getPointDistance p1 p2 = (float p1 - float p2) ** 2.0
    [ getPointDistance c1.R c2.R
      getPointDistance c1.G c2.G
      getPointDistance c1.B c2.B ] |> List.sum |> Math.Sqrt
```

你可以通过指定两个颜色来正常调用`GetEuclideanDistance`方法，或者像这样通过指定参数名称来调用：

```
> **let d = ColorDistance()**
**d.GetEuclideanDistance(Color.White, Color.Black);;**

val d : ColorDistance
val it : float = 441.6729559

> **d.GetEuclideanDistance(c2 = Color.White, c1 = Color.Snow);;**
val it : float = 7.071067812
```

你可以按任意顺序指定命名参数。你也可以将命名参数与未命名参数一起使用，但如果这样做，未命名参数必须首先出现在参数列表中。最后，由于命名参数仅适用于使用成员语法定义的方法，因此不能与通过`let`绑定创建的函数一起使用。

#### 重载方法

*重载方法*与同一类中的一个或多个其他方法共享相同的名称，但具有不同的参数集。重载方法通常定义参数的子集，每个重载调用一个更具体的形式，并为其他参数提供默认值。

例如，如果你正在构建一个与自己喜欢的版本控制系统对接的工具，你可能会定义一个`Commit`方法，该方法接受一个文件列表、描述和目标分支。为了让目标分支成为可选项，你可以像这里展示的那样重载`Commit`函数：

```
open System.IO

type Repository() =
  member ① x.Commit(files, desc, branch) =
    printfn "Committed %i files (%s) to \"%s\"" (List.length files) desc branch
  member ② x.Commit(files, desc) =
    x.Commit(files, desc,  ③"default")
```

在这个示例中，①处的重载负责将更改提交到仓库，而②处的重载则通过在③处提供的默认值，使分支参数变为可选。

#### 可选参数

尽管 F# 支持方法重载，但你可能不会经常使用它，因为 F# 还支持*可选参数*，这些参数通常更方便。如果你在参数名称前加上问号（`?`），编译器会将其视为可选参数。

可选参数在 F# 中与 C# 和 Visual Basic 中有所不同。在其他语言中，可选参数定义时会指定一个默认值，当省略相应参数时会使用该默认值。但在 F# 中，参数实际上会被编译为`option<_>`类型，并默认为`None`。（可选参数的值表现得像任何其他的 option 类型值，因此你仍然需要在方法中使用`defaultArg`或模式匹配来获取有意义的值，具体取决于情况。）

让我们改写上一节中的`Repository`示例，使用一个可选参数，而不是重载方法。

```
open System.IO

type Repository() =
  static member Commit(files, desc, ?branch) =
    let targetBranch = defaultArg branch "default"
    printfn "Committed %i files (%s) to \"%s\"" (List.length files) desc targetBranch
```

尽管你需要在方法中管理可选参数，但现在只需要维护一个方法，而不是多个重载版本。如你所见，可选参数可以减少由于在重载中使用不一致的默认值而产生的缺陷的可能性，并且它们简化了重构，因为只需要更改一个方法。

#### 切片表达式

索引属性，在索引属性中介绍，非常适合处理封装序列中的单个值，但有时你可能需要处理该序列中的一系列值。传统上，你必须通过索引器手动获取每个项，或者实现`IEnumerable<'T>`并通过 LINQ 的`Skip`和`Take`扩展方法的某种组合来获取值。*切片表达式*类似于索引属性，只不过它们使用范围表达式来标识应该包含在结果序列中的项。

要在你的类中使用切片表达式，你需要实现一个`GetSlice`方法。其实`GetSlice`方法并没有什么特别之处；它只是编译器在遇到切片表达式语法时会查找的方法。为了说明切片表达式，让我们回顾一下索引属性部分的`Sentence`类。

```
type Sentence(initial : string) =
  let words = initial.Split ' '
  let text = initial
  member x.GetSlice(lower, upper) =
    match defaultArg lower 0 with
    | l when l >= words.Length -> Array.empty<string>
    | l -> match defaultArg upper (words.Length - 1) with
           | u when u >= words.Length -> words.[l..]
           | u -> words.[l..u]
```

基本的类定义与之前相同，只不过这次我们有一个接受上下边界的`GetSlice()`方法。（不要纠结这里的匹配表达式；有关详细讨论，请参见第七章。现在知道它们只是在进行一些边界检查就足够了。）

你可以在代码中直接调用这个方法，但表达式形式更为方便。例如，要获取句子中的第二、第三和第四个单词，你可以这样写：

```
> **let s = Sentence "Don't forget to drink your Ovaltine"**
**s.[1..3];;**

val s : Sentence
val it : string [] = [|"forget"; "to"; "drink"|]
```

切片表达式的一个优点是，边界参数是可选的，因此你可以使用开放的范围。要指定一个没有下边界的范围，只需在切片表达式中省略第一个值（即 1），在这种情况下，它等同于`[0..3]`。

```
> **s.[..3];;**
val it : string [] = [|"Don't"; "forget"; "to"; "drink"|]
```

类似地，你可以省略第二个参数，获取到集合的末尾的项。

```
> **s.[3..];;**
val it : string [] = [|"drink"; "your"; "Ovaltine"|]
```

与索引属性类似，切片表达式可以作用于二维，但你需要重载`GetSlice`方法，以接受定义上下边界对的四个参数。继续使用`Sentence`示例，我们可以添加一个多维切片重载，以便从一组单词中获取一系列字符，像这样：

```
type Sentence(initial : string) =
  -- *snip* --
  member x.GetSlice(lower1, upper1, lower2, upper2) =
    x.GetSlice(lower1, upper1)
    |> Array.map
        (fun w -> match defaultArg lower2 0 with
                  | l when l >= w.Length -> ""
                  | l -> match defaultArg upper2 (w.Length - 1) with
                         | u when u >= w.Length -> w.[l..]
                         | u -> w.[l..u])
```

要使用这个重载，只需在切片表达式中用逗号分隔范围对。

```
> **s.[1..4, ..1];;**
val it : string [] = [|"fo"; "to"; "dr"; "yo"|]
```

### 事件

最后一种成员类型是*事件*。事件在整个 .NET Framework 中都有应用，一些显著的例子可以在用户界面组件和 ADO.NET 中找到。与其他 .NET 语言一样，F# 的事件本质上是响应某些操作（如按钮点击或异步进程完成）时调用的一系列函数集合。

在许多方面，F# 的事件与传统的 .NET 事件具有相同的作用，但它们是完全不同的机制。然而，为了实现跨语言兼容性，它们可以与 .NET 事件系统结合使用。（稍后我们将在本节中看到，如何通过 `CLIEvent` 属性来利用这种能力）

#### 基本事件处理

F# 中的事件是 `Event<'T>` 类的实例（位于 `FSharp.Core.Control` 中）。`Event<'T>` 类的一个主要特性是，它提供了比你可能习惯的更加明确的发布/订阅模型。在这个模型中，你可以通过调用 `Add` 函数向事件添加事件处理程序，从而订阅发布的事件。

例如，`System.Timers.Timer` 类发布了一个 `Elapsed` 事件，你可以订阅该事件。

```
  let ticks = ref 0
  let t = ① new System.Timers.Timer(500.0)
  t.Elapsed.Add ② (fun ea -> printfn "tick"; ticks := ticks.Value + 1)
③ t.Start()
  while ticks.Value < 5 do ()
  t.Dispose()
```

在这里，我们在①处创建一个新的 `Timer` 类实例。在②处，我们使用*lambda 表达式*（匿名函数）作为事件处理程序订阅 `Elapsed` 函数。定时器在③处启动后，事件处理程序每半秒打印 `tick` 并增加一个引用单元格的值（记住，像 lambda 表达式创建的闭包不能使用可变的 `let` 绑定）。当计时器的计数器达到五时，循环将终止，定时器将停止并被释放。

#### 观察事件

F# 事件的另一个主要好处是，它们使你能够将事件视为可以智能地分区、过滤、聚合或以其他方式在触发时处理的序列。`Event` 模块定义了许多函数——如 `add`、`filter`、`partition` 和 `pairwise`——可以接受发布的事件。

为了演示这一原则，让我们来看一个 ADO.NET 中的例子。`DataTable` 类会在响应某些操作（如行的更改或删除）时触发多种事件。如果你想处理 `RowChanged` 事件，可以添加一个事件处理程序（就像前一部分一样），并包含逻辑来过滤掉你不关心的事件，或者你也可以使用 `Event` 模块中的 `filter` 函数，仅在需要时调用你的处理程序，如下所示：

```
  open System
  open System.Data

  let dt = new DataTable("person")
  dt.Columns.AddRange
    [| new DataColumn("person_id", typedefof<int>)
       new DataColumn("first_name", typedefof<string>)
       new DataColumn("last_name", typedefof<string>) |]
  dt.Constraints.Add("pk_person", dt.Columns.[0], true)

  let 1 h1, h2 =
2 dt.RowChanged
    |> 3 Event.partition
           4(fun ea ->
              let ln = ea.Row.["last_name"] :?> string
              ln.Equals("Pond", StringComparison.InvariantCultureIgnoreCase))

5 h1.Add (fun _ -> printfn "Come along, Pond")
6 h2.Add (fun _ -> printfn "Row changed")
```

我们将跳过这个例子前半部分的讨论；对我们来说，重要的是它设置了一个带有三列和主键的 `DataTable`。这里真正重要的是 `partition` 函数。

在这个例子中，我们通过在④提供一个委托（以 lambda 表达式的形式）和在②提供`DataTable`的`RowChanged`事件发布的`Event`对象，来在③调用`partition`函数。然后，`partition`函数返回两个新事件，我们在①绑定到`h1`和`h2`。最后，我们通过在⑤和⑥调用它们的`Add`方法来订阅这两个新事件。

现在表结构和事件处理程序已经就绪，我们可以添加一些行并查看事件是如何被触发的。

```
**> dt.Rows.Add(1, "Rory", "Williams") |> ignore;;**
Row changed
val it : unit = ()
**> dt.Rows.Add(2, "Amelia", "Pond") |> ignore;;**
Come along, Pond
val it : unit = ()
```

正如你所看到的，当第一行被添加时，姓氏不符合筛选条件，因此触发了`h2`。然而，第二行符合条件，因此触发了`h1`。

如果调用分区函数的语法看起来像是反向的，那是因为它确实如此；*前向管道操作符*（`|>`）将其左操作数作为右操作数指定函数的最终参数。（前向管道操作符在 F#中使用频繁，我们将在第五章中更详细地探讨它。）

#### 自定义事件

你可以在你的类型中定义自定义事件。然而，做到这一点与其他.NET 语言有所不同，因为事件仅作为对象存在于 F#中，并且缺少关键字支持。

除了定义类型外，首先你需要做的是为你的事件对象创建一个字段（使用`let`绑定）。这是用来协调发布和触发事件的对象。一旦字段被定义，你可以通过自己定义的属性将事件的`Publish`属性暴露给外部。最后，你需要在某个地方通过调用`Trigger`函数来触发事件。

```
type Toggle() =
  **let toggleChangedEvent = Event<_>()**
  let mutable isOn = false

  member x.ToggleChanged = **toggleChangedEvent.Publish**

  member x.Toggle() =
    isOn <- not isOn
    **toggleChangedEvent.Trigger (x, isOn)**
```

定义了类型后，你可以创建一个新实例，并像任何内建类型一样订阅`ToggleChanged`事件。例如，接下来我们使用分区来创建两个新的事件处理程序，一个处理切换打开时的情况，另一个处理切换关闭时的情况。调用`Event.map`只是通过丢弃第一个参数（源或发送者，按.NET 约定）重新表述事件，然后再调用`partition`函数。

```
let myToggle = Toggle()
let onHandler, offHandler =
  **myToggle.ToggleChanged**
  |> **Event.map** (fun (_, isOn) -> isOn)
  |> **Event.partition** (fun isOn -> isOn)

onHandler |> **Event.add** (fun _ -> printfn "Turned on!")
offHandler |> **Event.add** (fun _ -> printfn "Turned off!")
```

现在，每次调用`Toggle`方法都会触发`ToggleChanged`事件，并执行两个处理程序中的一个。

```
**> myToggle.Toggle();;**
Turned on!
val it : unit = ()
**> myToggle.Toggle();;**
Turned off!
val it : unit = ()
```

正如你刚才看到的，`ToggleChanged`事件在 F#中是完全启用的。如果你的类只会在 F#程序集内部使用，你可以到此为止。然而，如果你需要在其他语言编写的程序集里使用它，你还需要做一件事：用`CLIEvent`特性装饰`ToggleChanged`属性。

```
**[<CLIEvent>]**
member x.ToggleChanged = toggleChangedEvent.Publish
```

`CLIEvent`特性指示编译器包括适当的元数据，使得该事件可以从其他.NET 语言中消费。

## 结构

*结构*（或 *结构体*）与类类似，都可以拥有字段、属性、方法和事件。结构体的定义方式与类相同，不同之处在于类型必须使用 `Struct` 特性进行修饰。

```
**[<Struct>]**
type Circle(diameter : float) =
  member x.getRadius() = diameter / 2.0
  member x.Diameter = diameter
  member x.GetArea() = System.Math.PI * (x.getRadius() ** 2.0)
```

然而，尽管类和结构体有相似之处，实际上它们在幕后是非常不同的。它们之间的主要区别在于结构体是 *值类型*。

这个差异很重要，因为它不仅影响你如何与数据交互，还影响值类型在计算机内存中的表示方式。对于这两种类型，运行时都会在内存中分配空间来存储值。值类型总是会导致新的内存分配，并将数据复制到该空间。而对于引用类型，内存只会分配一次，通过引用来访问其位置。

当你将一个引用类型传递给函数时，运行时会在内存中创建该位置的新引用，而不是数据的副本。因此，引用类型更容易通过副作用造成破坏，因为当你将引用类型传递给函数时，对该对象所做的任何修改都会立即反映到该对象被引用的地方。相反，传递值类型给函数会创建该值的副本，因此对其所做的任何修改仅限于该实例。

结构体的初始化方式也不同于类。与类不同，编译器会为结构体生成一个默认的（无参数）构造函数，该构造函数将所有字段初始化为适当的零值（`zero`、`null` 等）。这意味着，除非是静态字段，否则你不能使用 `let` 绑定在结构体中创建私有实例字段或方法；相反，你必须使用 `val` 来定义结构体实例字段。此外，你不能定义自己的默认构造函数，因此你定义的任何附加构造函数必须至少接受一个参数。（只要不包含主构造函数，你的字段仍然可以是可变的。）

由于引用类型和值类型在内存分配方式上的不同，结构体不能包含其自身类型的字段。如果没有这个限制，结构体实例的内存需求将是无限大的，因为每个实例都会递归地要求另一个相同类型实例所需的空间。

最后，结构体可以实现接口，但不能参与继承。无论如何，结构体仍然从 `System.Object` 派生，因此你可以重写方法（如 `ToString`）。

## 继承

在面向对象编程中，*继承* 描述了两个类型之间的 *身份* 关系，类似于苹果 *是* 一种水果。F# 类支持 *单继承*，这意味着任何给定的类只能直接继承另一个类，以建立类层次结构。通过继承，基类公开的公共（有时是内部）成员会自动在派生类中可用。你可以在以下代码片段中看到这一原则的应用。

```
type BaseType() =
  member x.SayHello name = printfn "Hello, %s" name

type DerivedType() =
  inherit BaseType()
```

这里定义的`DerivedType`没有定义任何自己的功能，但由于它继承自`BaseType`，所以可以通过`DerivedType`访问`SayHello`方法。

F# 的继承要求有一个主构造函数。要指定基类，可以在主构造函数中包含`inherit`关键字，后跟基类型名称及其构造函数参数，然后再进行任何绑定或成员定义。例如，一个任务管理系统可能有一个`WorkItem`类，表示系统中的所有工作项，以及像`Defect`和`Enhancement`这样的专门化类，这些类继承自`WorkItem`类，具体如下面加粗所示。

```
type WorkItem(summary : string, desc : string) =
  member val Summary = summary
  member val Description = desc

type Defect(summary, desc, severity : int) =
  **inherit WorkItem(summary, desc)**
  member val Severity = severity
type Enhancement(summary, desc, requestedBy : string) =
  **inherit WorkItem(summary, desc)**
  member val RequestedBy = requestedBy
```

每个 .NET 类，包括原始数据类型，最终都会参与继承。同时，当你定义一个类而没有显式指定基类时，定义的类会隐式地继承自`System.Object`。

### 类型转换

在第三章中，你学习了如何在数值类型之间进行转换。类型也可以通过向上转换和向下转换运算符在其类型层次结构内进行转换。

#### 向上转换

直到现在我一直坚持认为 F# 中没有隐式转换，但这并不完全正确。唯一的隐式*向上转换*（转换为继承结构中更高层次的类型）发生的情况是，当类型被传递给一个方法或一个`let`绑定的函数，而该方法的参数类型是灵活类型。在其他所有情况下，你必须显式地使用*静态类型转换*运算符（`:>`）来转换类型。

为了展示静态类型转换运算符的实际应用，让我们继续使用`WorkItem`示例，创建一个`Defect`并立即将其转换为`WorkItem`。

```
**> let w = Defect("Incompatibility detected", "Delete", 1) :> WorkItem;;**
val w : WorkItem
```

静态类型转换运算符在编译时解析有效的转换。如果代码能够编译，转换就一定会成功。

#### 向下转换

向上转换的相反操作是*向下转换*。向下转换用于将一个类型转换为其层次结构中更低的类型，即将基类型转换为派生类型。要执行向下转换，可以使用*动态类型转换*运算符（`:?>`）。

因为我们在前面的示例中创建的`WorkItem`实例仍然是一个`Defect`，所以我们可以使用动态类型转换运算符将其转换回`WorkItem`。

```
**> let d = w :?> Defect;;**
val d : Defect
```

与静态类型转换运算符不同，动态类型转换运算符直到运行时才会解析，因此，如果目标类型不适用于源对象，可能会出现`InvalidCastException`。例如，如果你尝试将`w`向下转换为`Enhancement`，转换将失败。

```
**> let e = w :?> Enhancement;;**
System.InvalidCastException: Unable to cast object of type 'Defect' to type 'Enhancement'.
   at Microsoft.FSharp.Core.LanguagePrimitives.IntrinsicFunctions.UnboxGenericT
   at <StartupCode$FSI_0007>.$FSI_0007.main@()
Stopped due to error
```

### 重写成员

除了重用代码外，你还可以通过重写基类的成员来改变基类所提供的功能。

例如，`System.Object`上定义的`ToString`方法是一个很好的（但常被忽视的）调试工具，其默认实现并不特别有用，因为它仅仅返回类型名称。为了使其更有用，你的类可以重写默认功能，并返回一个真正描述对象的字符串。

为了说明这一点，考虑之前的`WorkItem`类。如果你调用它的`ToString`方法，你将看到类似下面的内容：

```
**> let w = WorkItem("Take out the trash", "It's overflowing!")**
**w.ToString();;**

val w : WorkItem
val it : string = "FSI_0002+WorkItem"
```

### 注意

*在前面的例子中，FSI_0002+是调用 FSI 代码时的产物。你的类型名称可能会有所不同。*

要覆盖默认行为并使`ToString`返回更有用的内容，请使用`override`关键字定义一个新方法。

```
type WorkItem(summary : string, desc : string) =
  -- *snip* --
  **override** x.ToString() = sprintf "%s" x.Summary
```

如果现在调用`ToString`，结果将是摘要文本，而不是类型名称。

```
**> let w = WorkItem("Take out the trash", "It's overflowing!")**
**w.ToString();;**

val w : WorkItem = Take out the trash
val it : string = "Take out the trash"
```

每个类型只能覆盖给定函数一次，但你可以在层次结构的多个级别进行覆盖。例如，这里展示了如何在`Defect`类中再次覆盖`ToString`，以显示缺陷的严重性：

```
type Defect(summary, desc, severity : int) =
  inherit WorkItem(summary, desc)
  member val Severity = severity
  **override x.ToString() = sprintf "%s (%i)" x.Summary x.Severity**
```

当覆盖*虚拟成员*（一个具有默认实现的抽象成员）时，可以通过`base`关键字调用基类的功能。`base`关键字的行为像自我标识符，只不过它代表的是基类。

继续我们的`ToString`重写主题，为了增强默认行为，你的重写可以像这样调用`base.ToString()`：

```
type Defect(summary, desc, severity : int) =
  -- *snip* --
  override x.ToString() =
    sprintf "%s (%i)" (**base.ToString()**) x.Severity
```

请注意，`base`关键字仅在显式继承自其他类型的类中可用。要在继承自`System.Object`的类中使用`base`关键字，你需要显式地继承它，如下所示：

```
type WorkItem(summary : string, desc : string) =
  inherit System.Object()
  -- *snip* --
  override x.ToString() =
    sprintf "[%s] %s" (base.ToString()) x.Summary
```

### 抽象类

*抽象类*是不能直接实例化的类；它只能通过派生类访问。抽象类通常为一组相关类定义公共接口和可选实现，这些类以不同方式满足类似需求。抽象类在.NET 框架中被广泛使用，一个很好的例子是`System.IO`命名空间中的`TextWriter`类。

`TextWriter`类定义了一个写入字符到*某个地方*的通用机制。它不关心字符写入的位置或方式，但它协调了这一过程，具体的实现细节则交给像`StreamWriter`、`StringWriter`和`HttpWriter`这样的派生类来完成。

你可以通过用`AbstractClass`属性修饰类型定义来定义自己的抽象类。例如，要创建一个简单的树形结构，你可以使用如下的抽象类：

```
**[<AbstractClass>]**
type Node(name : string, ?content : Node list) =
  member x.Name = name
  member x.Content = content
```

### 抽象成员

定义抽象类的一个原因是为了定义*抽象成员*，即没有实现的成员。抽象成员仅允许出现在抽象类中（或者接口中，见接口），并且必须在派生类中实现。当你想定义一个类做什么，但不关心它是如何做的时，抽象成员非常有用。

#### 抽象属性

当你想定义与特定类型关联的数据，但不关心这些数据是如何存储的或在访问时会发生什么时，你可以使用`abstract`关键字定义*抽象属性*。

例如，这个抽象类包含一个抽象属性：

```
[<AbstractClass>]
type AbstractBaseClass() =
  **abstract member SomeData : string with get, set**
```

`AbstractBaseClass` 仅要求其子类实现`SomeData`属性，但它们可以自由实现自己的存储机制。例如，一个派生类可能使用传统的后备存储，而另一个则可能选择使用 .NET 泛型字典，如下所示：

```
type BindingBackedClass() =
  **inherit AbstractBaseClass()**
  let mutable someData = ""
  **override x.SomeData**
    with get() = someData
    and set(v) = someData <- v

type DictionaryBackedClass() =
  **inherit AbstractBaseClass()**
  let dict = System.Collections.Generic.Dictionary<string, string>()
[<Literal>]
let SomeDataKey = "SomeData"
**override x.SomeData**
  with get() =
    match dict.TryGetValue(SomeDataKey) with
    | true, v -> v
    | _, _ -> ""
  and set(v) =
    match System.String.IsNullOrEmpty(v) with
    | true when dict.ContainsKey(SomeDataKey) ->
        dict.Remove(SomeDataKey) |> ignore
    | _ -> dict.[SomeDataKey] <- v
```

如你所见，`BindingBackedClass`和`DictionaryBackedClass`都继承自`AbstractBaseClass`，但它们以非常不同的方式实现了`SomeData`属性。

#### 抽象方法

尽管你可以定义抽象属性，但你更有可能使用*抽象方法*。像抽象属性一样，抽象方法允许你定义派生类必须实现的能力，而无需指定任何实现细节。例如，在计算形状的面积时，你可能会定义一个抽象的`Shape`类，其中包含一个抽象的`GetArea`方法。

```
[<AbstractClass>]
type Shape() =
  **abstract member GetArea : unit -> float**
```

由于该方法没有实现，你必须显式定义整个签名。在这种情况下，`GetArea`方法接受`unit`并返回一个浮动值。

重写方法也类似于重写属性，正如你在以下`Circle`和`Rectangle`类中看到的那样：

```
open System

type Circle(r : float) =
  inherit Shape()
  member val Radius = r
  **override x.GetArea()** =
    Math.Pow(Math.PI * r, 2.0)

type Rectangle(w : float, h : float) =
  inherit Shape()
  member val Width = w
  member val Height = h
  **override x.GetArea()** = w * h
```

### 虚拟成员

与 C# 和 Visual Basic 一样，F# 也允许*虚拟成员*——即可以在派生类中重写的属性或方法。但与其他 .NET 语言不同，F# 对虚拟成员采取了更为字面化的方法。例如，在 C# 中，你会在非私有实例成员定义中包含`virtual`修饰符，而在 Visual Basic 中，你则使用`Overridable`修饰符来实现相同的效果。

F#中的虚拟成员与抽象成员密切相关。实际上，要创建一个虚拟成员，你首先定义一个抽象成员，然后使用`default`关键字提供默认实现。例如，在以下代码中，`Node`类是一个简单树结构的基础。它提供了两个虚拟方法，`AddChild`和`RemoveChild`，用于帮助控制树结构。

```
open System
open System.Collections.Generic

type Node(name : string) =
  let children = List<Node>()
  member x.Children with get() = children.AsReadOnly()
  **abstract member AddChild : Node -> unit**
  **abstract member RemoveChild : Node -> unit**
  **default x.AddChild(n) = children.Add n**
  **default x.RemoveChild(n) = children.Remove n |> ignore**
```

通过这个定义，所有`Node`类的实例（包括任何派生类型）都将允许子节点。为了创建一个不允许子节点的特定`Node`类，你可以定义一个`TerminalNode`类，并重写这两个虚拟方法，以防止添加或移除子节点。

```
type TerminalNode(name : string) =
  inherit Node(name)
  [<Literal>]
  let notSupportedMsg = "Cannot add or remove children"
  **override x.AddChild(n)** =
    raise (NotSupportedException(notSupportedMsg))
  **override x.RemoveChild(n)** =
    raise (NotSupportedException(notSupportedMsg))
```

### 密封类

*密封类*是不能作为其他类基类的类。在 .NET Framework 中，最著名的密封类之一是`System.String`。

你可以通过使用`Sealed`属性来创建你自己的密封类，如以下代码片段所示：

```
**[<Sealed>]**
type NotInheritable() = class end
```

如果你试图创建一个继承自`NotInheritable`类的其他类，编译器将会抛出类似如下的错误：

```
> **type InvalidClass() =**
     **inherit NotInheritable();;**

       inherit NotInheritable();;
       --^^^^^^^^^^^^^^^^^^^^^^^^

stdin(4,3): error FS0945: Cannot inherit a sealed type
```

## 静态成员

字段、属性和方法默认是实例成员。你可以通过在成员定义前加上`static`关键字，将它们设为静态成员，使其适用于类型而不是特定实例。

关于静态类的说明

在 C# 中，*静态类* 是一个隐式封闭的类，无法实例化，并且其中的所有成员都是静态的。在 F# 中，大多数情况下，当你需要类似静态类的功能时，会将其放在模块中。然而，模块有一定的局限性。例如，模块不允许你重载函数。

尽管 F# 并不像 C# 那样直接支持静态类，但你可以通过一些语法技巧来实现类似的效果。为此，需要省略主构造函数（或者如果需要静态初始化器，则将其设为私有），以确保无法创建实例，然后验证每个成员是否为静态成员（F# 编译器不会为你强制执行此检查）。为了完整性，可以使用 `SealedAttribute` 装饰类，确保没有任何类继承自它。

### 静态初始化器

*静态初始化器*，或称 *静态构造函数*，每个类只会执行一次，并确保某些代码在类首次使用之前执行。在 F# 中，可以通过一系列静态的 `let` 和 `do` 绑定来创建静态初始化器，就像定义主构造函数时一样。事实上，如果你的类需要静态初始化器，必须包含一个主构造函数来容纳这些静态绑定，如下所示：

```
type ClassWithStaticCtor() =
  **static let mutable staticField = 0**
  **static do printfn "Invoking static initializer"**
            **staticField <- 10**
  do printfn "Static Field Value: %i" staticField
```

静态初始化器只能访问它所包含类的静态成员。如果你尝试在静态初始化器中访问实例成员，将会收到编译错误。

### 静态字段

*静态字段* 常用于作为单一引用，以便你需要反复使用某些数据。例如，为了将某些数据与类本身关联，可以通过在 `let` 绑定前加上 `static` 关键字来定义一个静态字段，如下所示：

```
module Logger =
  let private log l c m = printfn "%-5s [%s] %s" l c m
  let LogInfo = log "INFO"
  let LogError = log "ERROR"

type MyService() =
  **static let logCategory = "MyService"**
  member x.DoSomething() =
    Logger.LogInfo logCategory "Doing something"
  member x.DoSomethingElse() =
    Logger.LogError logCategory "Doing something else"
```

当调用 `DoSomething` 和 `DoSomethingElse` 方法时，每个方法都会调用 `Logger` 模块中的一个函数，写入相同类别的日志消息，但不会重复数据。

```
> **let svc = MyService()**
**svc.DoSomething()**
**svc.DoSomethingElse();;**
INFO [MyService] Doing something
ERROR [MyService] Doing something else
```

### 静态属性

属性也可以是静态的。这里使用了一个只读的 *静态属性* 来暴露特定方法在所有类实例中被调用的次数。

```
type Processor() =
  static let mutable itemsProcessed = 0
  **static member ItemsProcessed = itemsProcessed**
  member x.Process() =
    **itemsProcessed <- itemsProcessed + 1**
    printfn "Processing..."
```

每次调用 `Process` 方法时，它会递增 `itemsProcessed` 字段并打印一条消息。要查看 `Process` 方法在所有实例中被调用的次数，请检查 `Processor` 类本身的 `ItemsProcessed` 属性。

```
> **while Processor.ItemsProcessed < 5 do (Processor()).Process();;**
Processing...
Processing...
Processing...
Processing...
Processing...
val it : unit = ()
```

这个示例会迭代，直到 `Process` 方法被调用的次数少于五次。每次迭代都会创建一个新的 `Processor` 类实例，并调用它的 `Process` 方法（这展示了静态属性与实例无关的特性）。

### 静态方法

和其他静态成员一样，*静态方法*是应用于类型而不是实例的。例如，静态方法常用于*工厂模式*（一种创建相似类实例的常见方法，而无需依赖特定的实现）。在一些工厂模式的变种中，静态方法返回符合特定接口的对象的新实例。为了说明这一概念，假设有一个需要处理不同图像格式的应用程序。你可能有一个抽象的`ImageReader`类，其他类型可以从该类派生，以处理特定的格式，如 JPEG、GIF 和 PNG。

```
[<AbstractClass>]
type ImageReader() =
  abstract member Dimensions : int * int with get
  abstract member Resolution : int * int with get
  abstract member Content : byte array with get

type JpgImageReader(fileName : string) =
  inherit ImageReader()
  -- *snip* --

type GifImageReader(fileName : string) =
  inherit ImageReader()
  -- *snip* --

type PngImageReader(fileName : string) =
  inherit ImageReader()
  -- *snip* --
```

创建这些类实例的工厂方法可能如下所示：

```
open System.IO

[<Sealed>]
type ImageReaderFactory private() =
  static member CreateReader(fileName) =
    let fi = FileInfo(fileName)

    match fi.Extension.ToUpper() with
    | ".JPG" -> JpgImageReader(fileName) :> ImageReader
    | ".GIF" -> GifImageReader(fileName) :> ImageReader
    | ".PNG" -> PngImageReader(fileName) :> ImageReader
    | ext -> failwith (sprintf "Unsupported extension: %s" ext)
```

上述代码段中的静态`CreateReader`方法使用 F#模式匹配，根据提供的文件名创建适当的`ImageReader`实现。当文件扩展名无法识别时，它会抛出一个异常，指示该格式不受支持。由于该方法是静态的，因此可以在不创建`ImageReaderFactory`类实例的情况下调用它，如下所示：

```
ImageReaderFactory.CreateReader "MyPicture.jpg"
ImageReaderFactory.CreateReader "MyPicture.gif"
ImageReaderFactory.CreateReader "MyPicture.png"
ImageReaderFactory.CreateReader "MyPicture.targa"
```

## 相互递归类型

当两个或多个类型相互依赖，无法单独使用其中一个类型而不依赖另一个时，这些类型被称为*相互递归*。

举个例子，想象一本书和它的页面。书本可以包含一组页面，但每个页面也可能会引用回书本。请记住，F#是自上而下进行评估的，那么你会先定义哪个类型呢？书本还是页面？由于书本依赖于其页面，并且页面又引用回书本，这里就有了相互递归。这意味着你必须使用`and`关键字将这些类型一起定义，如下所示：

```
**type Book() =**
  let pages = List<Page>()
  member x.Pages with get() = pages.AsReadOnly()
  member x.AddPage(pageNumber : int, page : Page) =
    if page.Owner = Some(x) then failwith "Page is already part of a book"
    pages.Insert(pageNumber - 1, page)
**and Page(content : string) =**
  let mutable owner : Book option = None
  member x.Content = content
  member x.Owner with get() = owner
  member internal x.Owner with set(v) = owner <- v
  override x.ToString() = content
```

## 接口

在面向对象编程中，*接口*指定了类型必须支持的属性、方法，有时还包括事件。在某些方面，接口类似于抽象类，但存在一些重要的区别。首先，与抽象类不同，接口不能包含其成员的实现；它们的成员必须是抽象的。此外，由于接口定义了实现者必须支持的功能，因此所有接口成员默认是公开的。最后，接口不受与类相同的继承限制：一个类可以实现任意数量的接口（结构体也可以）。

### 实现接口

F#在接口实现方面与其.NET 语言的同行有所不同。C#和 Visual Basic 都允许隐式和显式实现。通过*隐式实现*，接口成员可以通过实现类直接访问，而通过*显式实现*，接口成员只能在将实现类型视为接口时访问。

考虑这个 C#示例，其中有两个类都实现了`IDisposable`接口：

```
// C#

class ImplicitExample : IDisposable
{
① **public void Dispose()**
  {
    Console.WriteLine("Disposing");
  }
}

class ExplicitExample : IDisposable
{
② **void IDisposable.Dispose()**
  {
    Console.WriteLine("Disposing");
  }
}
```

这两个类都实现了`IDisposable`，但`ImplicitExample`①是隐式实现的，而`ExplicitExample`②是显式实现的。这种差异会显著影响你在每个类中调用`Dispose`方法的方式，如下所示：

```
  // C#

  var ex1 = ①new ImplicitExample();
② ex1.Dispose();

  var ex2 = ③ new ExplicitExample();
④ ((IDisposable)ex2).Dispose();
```

在①处我们实例化了`ImplicitExample`，在③处我们实例化了`ExplicitExample`。对于这两个类，我们都调用了`Dispose`方法，但因为`Dispose`在`ImplicitExample`类中是隐式实现的，所以我们可以直接通过`ex1`调用它，如在②处所示。如果我们尝试对`ex2`采用相同的方法，编译器会报错，因为`Dispose`在`ExplicitExample`类中是显式实现的。相反，我们需要将`ex2`强制转换为`IDisposable`，如在④处所示，才能调用其`Dispose`方法。

### 注意

*F#中的所有接口实现都是显式的。尽管 F#支持在其他语言中定义的类型上进行隐式接口实现，但你在 F#中定义的任何实现都会是显式的。*

在 F#中实现接口类似于继承其他类，只是它使用了`interface`关键字。例如，要在某个类型中实现`IDisposable`，你可以这样做：

```
open System

type MyDisposable() =
  **interface IDisposable with**
    member x.Dispose() = printfn "Disposing"
```

要手动调用`MyDisposable`类的`Dispose`方法，你需要将实例强制转换为`IDisposable`，如下所示，使用静态强制转换运算符：

```
let d = new MyDisposable()
(d :> IDisposable).Dispose()
```

### 定义接口

当你定义一个没有任何构造函数并且只有抽象成员的类型时，F# 编译器会推断该类型是一个接口。例如，一个用于处理图像数据的接口可能像这样：

```
open System.Drawing
open System.IO

**type IImageAdapter =**
  **abstract member PixelDimensions : SizeF with get**
  **abstract member VerticalResolution : int with get**
  **abstract member HorizontalResolution : int with get**
  **abstract member GetRawData : unit -> Stream**
```

如你所见，`IImageAdapter`类型没有构造函数，且它的四个成员都是抽象的。要定义一个空接口，或者说是*标记接口*，你可以用`interface end`关键字对来结束定义：

```
type IMarker = **interface end**
```

### 注意

*在.NET 开发中，接口名通常以大写字母 I 开头*。为了保持一致性，你应该遵循这一惯例。

与类一样，接口也可以继承彼此，以定义更为专门的契约。同样，接口继承也是通过`inherit`关键字实现的。

让我们继续以图像处理为例。`IImageAdapter`接口对于处理任何图像格式都很有帮助，但某些格式包含其他格式没有的功能。为了处理这些功能，你可以定义额外的接口来表示这些功能。例如，在处理支持透明度的格式时，你可能会创建一个从`IImageAdapter`继承的`ITransparentImageAdapter`，如下所示：

```
type ITransparentImageAdapter =
  inherit IImageAdapter
  abstract member TransparentColor : Color with get, set
```

现在，任何实现了`ITransparentImageAdapter`的类型必须实现`IImageAdapter`和`ITransparentImageAdapter`中定义的所有成员。

## 自定义操作符

在第三章中，你看到了许多用于处理内置数据类型的预定义操作符。你可以使用*操作符重载*来扩展这些操作符到你的类型上。通过重载操作符，你可以让自定义类型的交互更自然一些。

F#中的操作符有两种形式：前缀和中缀。*前缀操作符*放在操作数之前，而*中缀操作符*则放在操作数之间。F#操作符还可以是*一元*或*二元*的，意味着它们分别对一个或两个参数进行操作。自定义操作符定义为静态方法，只不过名称是操作符并用括号括起来。

### 前缀操作符

当定义前缀操作符时，你必须以波浪号（`~`）开头，以便与具有相同名称的中缀操作符区分开来。波浪号本身不是操作符的一部分。为了演示操作符重载，我们将定义一个表示基本 RGB 颜色的类型。请参考以下类定义：

```
type RgbColor(r, g, b) =
  member x.Red = r
  member x.Green = g
  member x.Blue = b
  override x.ToString() = sprintf "(%i, %i, %i)" r g b
```

要计算负颜色，你可以定义一个`GetNegative`函数，但将负号（`-`）作为前缀放在实例前面是不是更直观呢？

```
type RgbColor(r, g, b) =
  -- *snip* --
  /// Negate a color
  **static member (~-) (r : RgbColor) =**
    RgbColor(
      r.Red ^^^ 0xFF,
      r.Green ^^^ 0xFF,
      r.Blue ^^^ 0xFF
    )
```

定义好自定义操作符后，你现在可以创建一个颜色实例，并用如下便捷的语法来查找它的负值：

```
**> let yellow = RgbColor(255, 255, 0)**
**let blue = -yellow;;**

val yellow : RgbColor = (255, 255, 0)
val blue : RgbColor = (0, 0, 255)
```

### 中缀操作符

创建中缀操作符几乎就像创建前缀操作符，只是你不需要在名称前加波浪号字符。

继续使用`RgbColor`的示例，使用我们熟悉且自然的`+`和`-`操作符来加减两种颜色会非常方便。

```
open System

type RgbColor(r, g, b) =
  -- *snip* --
  /// Add two colors
  **static member (+) (l : RgbColor, r : RgbColor) =**
    RgbColor(
      Math.Min(255, l.Red + r.Red),
      Math.Min(255, l.Green + r.Green),
      Math.Min(255, l.Blue + r.Blue)
    )
  /// Subtract two colors
  **static member (-) (l : RgbColor, r : RgbColor) =**
    RgbColor(
      Math.Max(0, l.Red - r.Red),
      Math.Max(0, l.Green - r.Green),
      Math.Max(0, l.Blue - r.Blue)
    )
```

现在，我们可以像加减数字一样加减颜色。

```
**> let red = RgbColor(255, 0, 0)**
**let green = RgbColor(0, 255, 0)**
**let yellow = red + green;;**

val red : RgbColor = (255, 0, 0)
val green : RgbColor = (0, 255, 0)
val yellow : RgbColor = (255, 255, 0)

**> let magenta = RgbColor(255, 0, 255)**
**let blue = magenta - red;;**

val magenta : RgbColor = (255, 0, 255)
val blue : RgbColor = (0, 0, 255)
```

### 新操作符

你不仅可以重载现有的操作符，还可以使用`!`、`%`、`&`、`*`、`+`、`-`、`.`、`/`、`<`、`=`、`>`、`?`、`@`、`^`、`|` 和 `~`等字符的各种组合来定义自定义操作符。创建自定义操作符可能会很复杂，因为你选择的组合决定了操作的优先级（precedence）和结合性（右到左或左到右）。此外，如果选择的操作符不直观，可能会影响代码的可读性。因此，如果你仍然想定义一个新的操作符，其定义方式与重载操作符相同。

例如，在前一节中我们重载了`+`操作符来加两个颜色，但如果是混合颜色呢？`+`操作符本来是一个不错的混合操作符选择，但由于它已经被用于加颜色，我们可以改用`+=`操作符来代替。

```
type RgbColor(r, g, b) =
  -- *snip* --
  /// Blend two colors
  **static member (+=) (l : RgbColor, r : RgbColor) =**
    RgbColor(
      (l.Red + r.Red) / 2,
      (l.Green + r.Green) / 2,
      (l.Blue + r.Blue) / 2
    )
```

现在，混合两种颜色就像加它们一样简单：

```
**> let grey = yellow += blue;;**

val grey : RgbColor = (127, 127, 127)
```

### 全局操作符

F#不仅允许你在类型上重载操作符，还可以在全局范围内定义操作符。这使你能够为你无法控制的类型创建新操作符！例如，要为标准`System.Drawing.Color`结构定义任何自定义操作符，你可以使用`let`绑定在全局层面定义一个新操作符，如下所示：

```
open System
open System.Drawing

**let (+) (l : Color) (r : Color) =**
    Color.FromArgb(
      255, // Alpha channel
      Math.Min(255, int <| l.R + r.R),
      Math.Min(255, int <| l.G + r.G),
      Math.Min(255, int <| l.B + r.B)
    )
```

### 警告

*在定义全局操作符时要小心。你定义的任何与内置操作符冲突的操作符都会优先级更高，这意味着你可能会无意中替换核心功能。*

## 对象表达式

作为正式继承的替代，F#提供了*对象表达式*，这是一种创建基于现有类或接口的临时（匿名）类型的便捷构造。对象表达式在你需要一个一次性类型，但又不想创建正式类型时很有用。（虽然这个类比并不完全完美，但你可以将对象表达式视为类型的 lambda 表达式，因为对象表达式的结果是一个实现接口或继承自基类的新类型的实例。）

例如，考虑一个简化的游戏场景，其中角色可以装备武器。你可能会看到像这样的武器接口和角色类：

```
type IWeapon =
  abstract Description : string with get
  abstract Power : int with get

type Character(name : string, maxHP : int) =
  member x.Name = name
  member val HP = maxHP with get, set
  member val Weapon : IWeapon option = None with get, set
  member x.Attack(o : Character) =
    let power = match x.Weapon with
                | Some(w) -> w.Power
                | None -> 1 // fists
    o.HP <- System.Math.Max(0, o.HP - power)
  override x.ToString() =
    sprintf "%s: %i/%i" name x.HP maxHP
```

你可以使用这些定义来创建几个角色：

```
let witchKing = Character("Witch-king", 100)
let frodo = Character("Frodo", 50)
```

按照目前的写法，如果其中一个角色攻击另一个角色，他不会造成太大伤害，因为他只有拳头。给每个角色配备武器会是个不错的主意，但我们现在只有`IWeapon`接口。我们可以为每个想得出的武器定义类型，但通过对象表达式编写一个函数来创建武器会更方便。

对象表达式，如以下`forgeWeapon`函数中的表达式，通过`new`关键字后跟类型名称、`with`关键字和所有成员定义（用大括号包裹）来定义。

```
let forgeWeapon desc power =
  **{ new IWeapon with**
      **member x.Description with get() = desc**
      **member x.Power with get() = power }**
```

有了`forgeWeapon`函数，我们可以为我们的角色创建一些武器。

```
> **let morgulBlade = forgeWeapon "Morgul-blade" 25**
**let sting = forgeWeapon "Sting" 10;;**

val morgulBlade : IWeapon
val sting : IWeapon
```

正如你所看到的，两次调用`forgeWeapon`都将返回`IWeapon`的新实例。它们可以像通过类型定义正式定义的那样使用，例如将它们分别分配给角色并调用`Attack`函数：

```
witchKing.Weapon <- Some(morgulBlade)
frodo.Weapon <- Some(sting)

witchKing.Attack frodo
```

尽管对象表达式很方便，但并不适用于每种情况。它们的一个主要缺点是，必须实现底层类型的每个抽象成员。如果底层接口或基类有很多抽象成员，对象表达式可能会变得非常繁琐，因此你可能会考虑使用其他构造。

对象表达式不限于单一基类型。要使用对象表达式实现多个基类型，可以使用类似继承的语法。例如，如果你希望通过`forgeWeapon`函数创建的武器也实现`IDisposable`，你可以使用以下代码：

```
let forgeWeapon desc power =
  { new IWeapon with
      member x.Description with get() = desc
      member x.Power with get() = power
    **interface System.IDisposable with**
      **member x.Dispose() = printfn "Disposing"** }
```

创建新武器和之前一样：

```
let narsil = forgeWeapon "Narsil" 25
```

通过对象表达式创建的包含多个基类型的对象，总是被视为`new`关键字后面紧接着列出的类型，除非它们被显式地转换为其他类型。例如，在`forgeWeapon`函数的情况下，返回的对象将是`IWeapon`，除非你将其转换为`IDisposable`。

```
(narsil :?> System.IDisposable).Dispose()
```

## 类型扩展

当 LINQ 被加入到.NET 框架时，它为 C#和 Visual Basic 引入的一个令人兴奋的特性是扩展方法。*扩展方法*允许你在不依赖继承或其他设计模式（如装饰器模式）的情况下向现有类型添加新方法。F#提供了类似的功能，不过它不仅限于方法。在 F#中，你可以创建扩展方法、属性、事件，甚至静态成员！

你可以通过*类型扩展*或*类型增强*来扩展 F#中的现有类型。类型扩展有两种类型：内在扩展和可选扩展。

*内在扩展*必须在与被扩展类型相同的命名空间或模块中定义，并且与该类型在同一个源文件中。新扩展在代码编译时成为扩展类型的一部分，并且可以通过反射查看。内在扩展非常适合在构建类型时按增量方式分组相关部分，或作为构建互递归类型定义的替代方案。

*可选扩展*必须在一个模块中定义。与它们在 C#和 Visual Basic 中的对应物一样，可选扩展只有在其包含的命名空间或模块被打开时才可访问，但无法通过反射看到。可选扩展最适合为你无法控制或在其他程序集定义的类型添加自定义功能。

无论是定义内在扩展还是可选扩展，语法都是相同的。你从一个新的类型定义开始。不同之处在于，你不是使用主构造函数和等号，而是使用`with`关键字，后跟扩展定义。例如，在这里我们通过静态方法和实例方法扩展了`Color`结构体（位于`System.Drawing`命名空间）。

```
module ColorExtensions =
  open System
  open System.Drawing
  open System.Text.RegularExpressions

  // Regular expression to parse the ARGB components from a hex string
① let private hexPattern =
    Regex("^#(?<color>[\dA-F]{8})$", RegexOptions.IgnoreCase ||| RegexOptions.Compiled)

  // Type extension
② type Color with
  ③ static member FromHex(hex) =
      match hexPattern.Match hex with
      | matches when matches.Success ->
        Color.FromArgb <| Convert.ToInt32(matches.Groups.["color"].Value, 16)
      | _ -> Color.Empty
  ④ member x.ToHex() = sprintf "#%02X%02X%02X%02X" x.A x.R x.G x.B
```

这个可选的类型扩展通过允许你从已知的十六进制颜色字符串创建新的实例或将颜色转换为十六进制颜色字符串，增强了`Color`结构体的可用性。类型扩展本身位于②。静态扩展方法③依赖于正则表达式（用于解析字符串的领域特定语言）①来匹配并提取十六进制值，将其转换为传递给`Color`构造函数的 ARGB 值。实例扩展方法④则简单地返回格式化为十六进制字符串的 ARGB 值。

跨语言考虑

尽管目标相似，F#中的扩展方法实现与.NET 框架中的其他部分有所不同。因此，在 F#中定义的可选扩展方法，在 C#或 Visual Basic 中无法作为扩展方法访问，除非你在类型定义和扩展方法中都包含`Extension`特性。

## 摘要

尽管 F# 在 .NET 框架中被视为一种小众的函数式语言，但在本章中你已经看到，F# 同时也是一门功能完备的面向对象语言。通过众多示例，你了解了 F# 简洁的语法如何帮助你开发健壮的面向对象框架，其中包括类、结构和接口。你甚至看到如何实现一些常见的设计模式，如单例模式和工厂模式。

尽管 F# 支持与其更成熟的对等语言相同的常见面向对象概念，但你也学到了它如何通过观察和类型增强，将运算符重载、事件和扩展方法等熟悉的概念扩展为更强大的功能。最后，你还了解了如何利用全新的构造，如对象表达式，通过在需要时创建临时类型来提高代码质量。
