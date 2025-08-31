## 第九章. 我能引用你说的这些吗？

LINQ 引入 .NET Framework 的另一个特性是表达式树。通常使用与 lambda 表达式相同的语法，*表达式树* 编译的不是可执行代码，而是一个描述代码的树结构，并可以被解析以转换成其他形式。这种编程方式通常被称为 *元编程*。就像我们可以将元数据视为描述数据的数据一样，我们也可以将元编程视为描述代码的代码。

本章并不是关于表达式树的；它讨论的是 F# 中类似的结构，叫做 *引用表达式*，也称为 *代码引用*。引用表达式解决了与表达式树相同的基本问题，但采取了根本不同的方法。在深入探讨如何在 F# 代码中构造和解析引用表达式之前，让我们快速比较一下表达式树与引用表达式。

## 比较表达式树和引用表达式

表达式树常常与 LINQ 提供者一起使用，用于将某些 C# 或 Visual Basic 表达式转换为 SQL，但它们不仅仅用于语言间的代码转换。有时，表达式树也被用来为本来可能令人困惑或容易出错的代码增加额外的安全性或可读性。考虑一下在 WPF 和 Silverlight 中常用的 `INotifyPropertyChanged` 接口。

`INotifyPropertyChanged` 定义了一个成员：一个带有字符串参数 `PropertyName` 的事件，该参数标识了发生变化并触发事件的属性。你可以通过创建一个 `PropertyChangedEventArgs` 实例，并将属性名作为字符串传递给构造函数来触发 `PropertyChanged` 事件。然而，这种方法容易出错：因为在传递给 `PropertyChangedEventArgs` 构造函数的字符串没有内在的检查，可能会提供一个无效的名称。表达式树可以帮助避免像这样的错误，如下所示的 C# 类，利用表达式树安全地识别更改的属性，而无需依赖大量的反射代码：

```
// C#
public class PropertyChangedExample
  : INotifyPropertyChanged
{
  public event PropertyChangedEventHandler PropertyChanged;

  private string _myProperty = String.Empty;

  public string MyProperty
  {
    get { return _myProperty; }
    set
    {
      _myProperty = value;
      RaisePropertyChangedEvent(①() => MyProperty);
    }
  }

  protected void RaisePropertyChangedEvent<TValue>(
    ② Expression<Func<TValue>> propertyExpr)
  {
   if(PropertyChanged == null) return;

   var memberExpr = ③(MemberExpression)propertyExpr.Body;
   var ea = new PropertyChangedEventArgs(④ memberExpr.Member.Name);

   PropertyChanged(this, ea);
  }
}
```

上面的示例展示了实现 `INotifyPropertyChanged` 的典型模式的一个变化。它并没有像通常那样传递一个魔法字符串给 `RaisePropertyChangedEvent` 方法①，而是使用了一个 lambda 表达式。然而，这个 lambda 表达式并没有编译成一个委托。相反，C# 编译器通过签名推断出应该将该 lambda 表达式编译为表达式树②。在方法内部，我们随后将表达式的主体强制转换为 `MemberExpression`，在③处提取属性名称，并将其传递给 `PropertyChangedEventArgs` 在④处。

引用表达式在 F# 中的作用类似，但与表达式树不同，它们在设计时强调了函数式编程，而不仅仅是它们的构造方式，还包括它们的解析方式。此外，表达式树并不支持许多 F# 中的重要概念。相比之下，引用表达式完全理解诸如柯里化、部分应用和递归声明（`let rec`）等概念。最后，引用表达式设计为递归解析，这使得遍历整个引用结构几乎变得微不足道。

你可以按如下方式使用引用表达式将前面的 C# 类重写为 F#：

```
// F#
open Microsoft.FSharp.Quotations
open Microsoft.FSharp.Quotations.Patterns
open System.ComponentModel

type PropertyChangedExample() as x =
  let pce = Event<_, _>()
  let mutable _myProperty = ""
① let triggerPce =
    function
    | ② PropertyGet(_, pi, _) ->
        let ea = PropertyChangedEventArgs(pi.Name)
        pce.Trigger(x, ea)
    | _ -> failwith "PropertyGet quotation is required"
  interface INotifyPropertyChanged with
    [<CLIEvent>]
    member x.PropertyChanged = pce.Publish
  member x.MyProperty with get() = _myProperty
                      and set(value) = _myProperty <- value
                                       triggerPce(③ <@@ x.MyProperty @@>)
```

这个修订版本的 `PropertyChangedExample` 类结构与 C# 版本非常相似。如同 C# 版本一样，`PropertyChangedEvent` 并未直接公开。相反，位于①的 `triggerPce` 函数接受一个引用表达式，并使用模式匹配来判断提供的引用表达式是否代表获取一个属性的值（如②所示）。最后，在对 `triggerPce` 的调用中，③的 lambda 表达式被引用表达式取代，且该引用表达式以 `<@@` 和 `@@>` 包裹属性引用的形式呈现。通过使用引用表达式，我们允许编译器判断所提供的属性是否有效，而不是希望自己输入正确的名称。以这种方式使用引用表达式还能防止未来重构时，我们移除或重命名属性却忘记更新字符串的问题。

尽管引用表达式和表达式树有许多相似之处，但它们并不完全相同。首先，没有内建的方式来评估引用表达式，也没有内建的方式将引用表达式转换为表达式树。如果你需要执行这两项任务，你将需要依赖 F# PowerPack，或其他提供这些功能的库。然而，随着 F# 3.0 引入查询表达式（第十章），这些需求应该会减少。

## 组合引用表达式

引用表达式可以有两种形式：强类型和弱类型。两者之间的区别有些误导，因为所有的引用表达式最终都是基于 `Expr<'T>` 或 `Expr` 类型，这些类型位于 `Microsoft.FSharp.Quotations` 命名空间中。在这个上下文中，强类型和弱类型实际上是指引用是否包含关于表达式类型的信息，而不是通过其组成部分来描述表达式。你可以通过其 `Raw` 属性从强类型的引用表达式获取一个弱类型的引用表达式。

除了 `Expr` 和 `Expr<'T>` 类型之外，`Microsoft.FSharp.Quotations` 命名空间还包含 `Var` 类型。`Var` 类型用于引用表达式中，用来描述绑定信息，包括绑定名称、数据类型以及绑定是否可变。

无论引用表达式是强类型还是弱类型，所有引用表达式都有一些约束条件。首先，引用中禁止出现对象表达式。其次，引用不能解析为泛型表达式。最后，引用必须是一个完整的表达式；即，引用必须做的不仅仅是定义一个 `let` 绑定。尝试创建一个违反任何这些条件的引用表达式将导致编译错误。

### 引用字面量

要创建一个引用表达式，您只需要将表达式包含在 `<@` 和 `@>` 或 `<@@` 和 `@@>` 之间，其中第一种形式创建一个强类型的引用表达式，第二种形式创建一个弱类型的引用表达式。例如，要创建一个表示乘法的强类型引用表达式，您可以像这样编写：

```
> **open Microsoft.FSharp.Quotations**
**let x, y = 10, 10**
**let expr = <@ x * y @>;;**

val x : int = 10
val y : int = 10
val expr : ① Expr<int> =
  Call (None, op_Multiply, [PropertyGet (None, x, []), PropertyGet (None, y, [])])
```

在上面的代码片段中，引用表达式的底层类型是 ① `Expr<int>`。在这种情况下，编译器推断引用表达式的类型为 `int`，并将该类型与表达式一起传递。该表达式的值是源表达式组成元素的列表。稍后我们将深入分析这些部分的含义以及如何使用它们来分解引用表达式。

引用表达式可以像前面的例子一样简单，但也可以表示更复杂的表达式，包括 lambda 表达式。例如，一个乘法两个整数的 lambda 表达式可以像这样被引用：

```
> **open Microsoft.FSharp.Quotations**
**let expr = <@ fun a b -> a * b @>;;**

val expr : Expr<(int -> int -> int)> =
  Lambda (a, Lambda (b, Call (None, op_Multiply, [a, b])))
```

同样，您可以在一个引用表达式中包含多个表达式。在这里，定义了一个 `let` 绑定的函数，并将其应用于两个整数值：

```
> **let expr = <@ let mult x y = x * y**
                 **mult 10 20 @>;;**

val expr : Quotations.Expr<int> =
  Let (mult, Lambda (x, Lambda (y, Call (None, op_Multiply, [x, y]))),
     Application (Application (mult, Value (10)), Value (20)))
```

### .NET 反射

创建引用表达式的另一种方式是通过标准 .NET 反射。通常，引用表达式是从不可执行的代码中创建的，但有时您可能会发现，您已经定义了一个包含要引用的代码的函数。与其复制代码，您可以使用 `ReflectedDefinition` 属性装饰该函数：

```
type Calc =
  [<ReflectedDefinition>]
  static member Multiply x y = x * y
```

在这里，`Multiply` 正常编译，因此可以直接调用，但 `ReflectedDefinition` 属性指示编译器还需要生成一个弱类型的引用表达式，并将结果嵌入编译后的程序集。要访问生成的引用表达式，您需要获取一个表示编译方法的标准反射 `MethodInfo` 对象，并将其传递给 `Expr` 类的静态方法 `TryGetReflectedDefinition`：

```
> **let expr =**
  **typeof<Calc>**
    **.GetMethod("Multiply")**
  **|> Expr.TryGetReflectedDefinition;;**

val expr : Expr option =
  Some Lambda (x, Lambda (y, Call (None, op_Multiply, [x, y])))
```

当您需要在一个类型中引用多个值时，给每个值加上 `ReflectedDefinition` 属性可能会显得繁琐。幸运的是，您也可以将该属性应用于模块和类型，以分别为它们的每个值或成员生成引用表达式。

### 手动组合

构建引用表达式的最终方法是通过链式调用`Expr`类型的静态方法手动构建一个表达式。`Expr`类型定义了 40 多个方法来创建新的`Expr`实例，每个实例表示在引用表达式中可能出现的各种构造。

`Expr`方法的定义方式使得它们的目的应该已经非常清楚，既然你已经了解了 F#中的数据结构和语言构造，我就不再详细讲解每一个方法了。不过有两点是非常重要的需要注意的。

首先，方法参数是元组形式的，因此不同于柯里化多个参数，它们必须以元组形式提供。其次，许多方法——近 50%的方法——使用.NET 反射来构造相应的表达式。

手动构建引用表达式可能很繁琐，但它能给你最大程度的控制权，来决定表达式的构建方式。更重要的是，这些方法允许你基于你无法控制的代码来构建引用表达式，因此这些代码无法装饰上`ReflectedDefinition`属性。

为了演示手动构建引用表达式的过程，让我们通过构建一个使用乘法操作符将两个值相乘的方法来逐步实现。首先，我们需要使用反射来访问定义乘法操作符的`Operators`模块，如下所示：

```
let operators =
  System.Type.GetType("Microsoft.FSharp.Core.Operators, FSharp.Core")
```

这个绑定使用部分限定名称来标识我们正在寻找的类型。（我们不得不在这里使用反射，因为`typeof<'T>`和`typedefof<'T>`在模块上不起作用。）现在，我们已经有了对`Operators`模块的引用，可以通过方法名称`op_Multiply`使用`GetMethod`方法获取对乘法操作符方法的引用：

```
let multiplyOperator = operators.GetMethod("op_Multiply")
```

接下来，我们检查返回的`MethodInfo`以获取操作符的每个参数。为了将这些参数包含在我们的表达式中，我们需要从相应的`PropertyInfo`实例创建`Var`实例。我们可以通过使用`Array.map`函数轻松地将每个参数进行转换。为了方便起见，我们还可以使用数组模式将结果数组转换为元组，如下所示：

```
let varX, varY =
  multiplyOperator.GetParameters()
  |> Array.map (fun p -> Var(p.Name, p.ParameterType))
  |> (function | [| x; y |] -> x, y
               | _ -> failwith "not supported")
```

我们现在已经有足够的信息来构建引用表达式：

```
let call = Expr.Call(multiplyOperator, [ Expr.Var(varX); Expr.Var(varY) ])
let innerLambda = Expr.Lambda(varY, call)
let outerLambda = Expr.Lambda(varX, innerLambda)
```

前面的绑定逐步构建了一个引用表达式，表示一个柯里化的函数，该函数用于将两个值相乘。正如你所看到的，引用表达式包含了乘法操作符的方法调用，一个内部的 lambda 表达式应用了`y`值，还有一个外部的 lambda 表达式应用了`x`值。如果你检查`outerLambda`的值，你应该会看到如下表示的结果表达式：

```
val outerLambda : Expr =
  Lambda (x, Lambda (y, Call (None, op_Multiply, [x, y])))
```

经过这么多工作，我们终于得到了一个等价于这个弱类型表达式的引用表达式：

```
<@@ fun x y -> x * y @@>
```

为了方便起见，我在这里完整地包含了之前的示例，你可以看到所有部分如何协同工作。

```
let operators =
  System.Type.GetType("Microsoft.FSharp.Core.Operators, FSharp.Core")
let multiplyOperator = operators.GetMethod("op_Multiply")
let varX, varY =
  multiplyOperator.GetParameters()
  |> Array.map (fun p -> Var(p.Name, p.ParameterType))
  |> (function | [| x; y |] -> x, y
               | _ -> failwith "not supported")

let call = Expr.Call(multiplyOperator, [ Expr.Var(varX); Expr.Var(varY) ])
let innerLambda = Expr.Lambda(varY, call)
let outerLambda = Expr.Lambda(varX, innerLambda)
```

### 引用表达式拼接

如果你需要合并多个引用表达式，你可以通过将每个引用表达式传递给 `Expr` 类上的适当静态方法（通常是 `Call`）手动构建一个新的引用表达式，但有一种更简单的方法：你可以通过使用拼接运算符将它们拼接在一起，从而创建一个新的字面量引用表达式。例如，假设你有以下序列和强类型引用表达式：

```
let numbers = seq { 1..10 }
let sum = <@ Seq.sum numbers @>
let count = <@ Seq.length numbers @>
```

你可以将 `sum` 和 `count` 合并成一个新的引用表达式，表示通过强类型拼接运算符 (`%`) 计算序列的平均值，如下所示：

```
let avgExpr = <@ %sum / %count @>
```

弱类型引用表达式也可以进行拼接。如果 `sum` 和 `count` 被定义为弱类型引用表达式（通过 `<@@ ... @@>` 语法），你可以使用弱类型拼接运算符 (`%%`) 进行拼接，如下所示：

```
let avgExpr = <@@ %%sum / %%count @@>
```

## 引用表达式的分解

虽然代码引用有助于你理解代码的结构，但它们的主要优势在于分解。F# 包含三个模块，这些模块也位于 `Microsoft.FSharp.Quotations` 命名空间中，定义了大量的完整和部分活跃模式，你可以使用它们将引用的表达式按不同粒度的程度分解为其组成部分。

+   ****`Pattern` 模块****。`Pattern` 模块中的部分活跃模式匹配 F# 语言的基本特性，如函数调用、函数应用、循环结构、原始值、绑定定义和对象创建。它们几乎一对一地对应于 `Expr` 类型上定义的函数，帮助你识别在最常见的表达式中使用哪个模式。

+   ****`DerivedPatterns` 模块****。`DerivedPatterns` 模块包含部分活跃模式，主要用于匹配表示原始字面量的引号表达式、基本布尔运算符（如 `&&` 和 `||`）以及使用 `ReflectedDefinition` 装饰的结构。

+   ****`ExprShape` 模块****。`ExprShape` 模块定义了一个完整的活跃模式，包含三个情况：`ShapeVar`、`ShapeLambda` 和 `ShapeCombination`。它设计用于递归模式匹配，因此你可以轻松地遍历引用的表达式，在整个过程中匹配每一个表达式。

### 引用表达式解析

与其详细讲解每个模块中定义的具体活跃模式，我认为更有帮助的是看看它们如何协同工作。我们将从一个典型示例开始，使用每个模块中的一些模式来构建一个表示 F# 引用语法的字符串。

```
open System.Text
open Microsoft.FSharp.Quotations.Patterns
open Microsoft.FSharp.Quotations.DerivedPatterns
open Microsoft.FSharp.Quotations.ExprShape

let rec showSyntax =
  function
  | Int32 v ->
      sprintf "%i" v
  | Value (v, _) ->
      sprintf "%s" (v.ToString())
  | SpecificCall <@@ (+) @@> (_, _, exprs) ->
      let left = showSyntax exprs.Head
      let right = showSyntax exprs.Tail.Head
      sprintf "%s + %s" left right
  | SpecificCall <@@ (-) @@> (_, _, exprs) ->
      let left = showSyntax exprs.Head
      let right = showSyntax exprs.Tail.Head
      sprintf "%s - %s" left right
  | Call (opt, mi, exprs) ->
      let owner = match opt with
                  | Some expr -> showSyntax expr
                  | None -> sprintf "%s" mi.DeclaringType.Name
      if exprs.IsEmpty then
        sprintf "%s.%s ()" owner mi.Name
      else
        let sb = StringBuilder(showSyntax exprs.Head)
        exprs.Tail
        |> List.iter (fun expr ->
                           sb
                             .Append(",")
                             .Append(showSyntax expr) |> ignore)
        sprintf "%s.%s (%s)" owner mi.Name (sb.ToString())
  | ShapeVar var ->
      sprintf "%A" var
  | ShapeLambda (p, body) ->
      sprintf "fun %s -> %s" p.Name (showSyntax body)
  | ShapeCombination (o, exprs) ->
      let sb = StringBuilder()
      exprs |> List.iter (fun expr -> sb.Append(showSyntax expr) |> ignore)
      sb.ToString()
```

上面的示例可能看起来令人畏惧，但尽管包含了许多匹配案例，实际上当你把它拆开看时，它并不是特别复杂。首先要注意的是，`showSyntax`函数是递归的，这使得我们能够遍历树状结构中的任何嵌套表达式。每个匹配案例都属于三个引号表达式模块之一，并且匹配特定类型的表达式。我不会详细介绍每个案例的主体，因为它们没有引入新的概念，但我鼓励你尝试实验。

前两个案例，`Int32`和`Value`，匹配单个字面值。`Int32`模式是一个派生模式，只匹配整数值，而`Value`是一个基础模式，匹配任何字面值。从定义中可以看出，这两个模式都提取了字面值。`Value`模式还会提取相应的数据类型，但由于我们在这里没有使用它，我们仅用通配符模式将其丢弃。

紧接着`Value`案例后面是两个`SpecificCall`案例和一个通用的`Call`案例。`SpecificCall`案例是派生的模式，分别匹配加法和减法运算符的调用（作为内联弱类型的引号表达式）。另一方面，`Call`案例是一个基础模式，匹配任何函数调用。`SpecificCall`案例比`Call`案例要简单得多，因为我们可以在了解匹配构成的情况下，对代码做出某些假设。而`Call`案例则需要做更多的工作来展开表达式。

最后，我们到了最后三个案例：`ShapeVar`、`ShapeLambda`和`ShapeCombination`。其中最简单的`ShapeVar`，匹配任何变量定义。（注意，这里使用*变量*一词比使用*绑定*更合适，因为它代表了代码中的一个占位符。）`ShapeVar`捕获的值包括变量名、数据类型和可变性等信息。`ShapeLambda`匹配任何 lambda 表达式，捕获其参数定义和作为嵌套表达式的主体。最后一个案例，`ShapeCombination`，匹配任何其他表达式，并且为了完整性也包括在内。

要查看`showSyntax`函数的实际效果，你可以传入任何引号表达式。只需记住，这种实现几乎无法覆盖所有可能的情况，因此对于更复杂的表达式，结果可能不会特别理想。不过，作为开始，这里有一些示例输入和结果：

```
> **showSyntax <@ fun x y -> x + y @>;;**
val it : string = "fun x -> fun y -> x + y"
> **showSyntax <@ fun x y -> x - y @>;;**
val it : string = "fun x -> fun y -> x - y"
> **showSyntax <@ 10 * 20 @>;;**
val it : string = "Operators.op_Multiply (10,20)"
> **showSyntax <@@ System.Math.Max(10, 20) @@>;;**
val it : string = "Math.Max (10,20)"
```

### 替代反射

就像你可以使用表达式树来实现类似反射的功能（正如你在本章开头看到的那样），你也可以使用引号表达式来实现类似的效果。为了演示，我将使用一个经过改编的版本，这个示例在我第一次学习引号表达式时非常有帮助。

这个示例，原始形式可以在 *[`fssnip.net/eu/`](http://fssnip.net/eu/)* 中找到，定义了一个广泛使用高阶函数、部分应用和引用表达式的模块，允许你为你的类型定义临时验证函数。我们将从完整的代码列表开始，在你有机会消化它之后再进行详细解析。

```
module Validation =
  open System
  open Microsoft.FSharp.Quotations
  open Microsoft.FSharp.Quotations.Patterns

  type Test<'e> = | Test of ('e -> (string * string) option)

  ① let private add (quote : Expr<'x>) message args validate (xs : Test<'e> list) =
    let propName, eval =
      match quote with
      | PropertyGet (_, p, _) -> p.Name, fun x -> p.GetValue(x, [||])
      | Value (_, ty) when ty = typeof<'e> -> "x", box
      | _ -> failwith "Unsupported expression"
    let test entity =
      let value = eval entity
      if validate (unbox value) then None
      else Some (propName, String.Format(message, Array.ofList (value :: args)))
    Test(test) :: xs

  ② let notNull quote =
    let validator = (fun v -> v <> null)
    add quote "Is a required field" [] validator

  ③ let notEmpty quote =
    add quote "Cannot be empty" [] (String.IsNullOrWhiteSpace >> not)

  ④ let between quote min max =
    let validator = (fun v -> v >= min && v <= max)
    add quote "Must be at least {2} and greater than {1}" [min; max] validator

  ⑤ let createValidator (f : 'e -> Test<'e> list -> Test<'e> list) =
    let entries = f Unchecked.defaultof<_> []
    fun entity -> List.choose (fun (Test test) -> test entity) entries
```

`Validation` 模块的核心是私有的 `add` 函数，位于 ① 处。此函数接受五个参数，每个参数都参与验证。最为关键的是第一个参数 `quote`，第三个参数 `validate`，以及最后一个参数 `xs`。这三个参数分别代表标识正在验证属性的引用、验证函数和测试函数列表。

在 `add` 函数内部，我们首先尝试将 `quote` 与 `PropertyGet` 和 `Value` 活跃模式匹配，以适当地从源对象中提取值，以便稍后将其传递给验证函数。接着，我们定义了一个名为 `test` 的函数，调用提供的 `validate` 函数，并返回一个选项值，指示提取的值是否有效。最后，`test` 函数被包装在 `Test` 联合类型中，并添加到 `xs` 列表前面，最终返回整个列表。

在 `add` 函数到位后，我们定义了多种函数，这些函数返回部分应用版的 `add`，从而使我们拥有了富有表现力的验证语法。在这个例子中，我们定义了 `notNull` ②、`notEmpty` ③ 和 `between` ④。每个函数接受一个被引用的表达式，并将其与接下来的三个参数一起应用于 `add`，从而生成新的函数，这些函数仅接受一个 `Test` 联合类型的列表并返回相同的列表。

`createValidator` ⑤ 函数是进入 `Validation` 模块的主要入口。`createValidator` 接受一个柯里化函数，其参数包括一个通用值和一个 `Test` 联合类型的列表（类型相同），最终返回另一个 `Test` 联合类型的列表。注意第二个参数和返回值与 `notNull`、`notEmpty` 和 `between` 函数返回的函数是相对应的。这里的含义是，我们可以组合一个验证函数传递给 `createValidator`，以便稍后随意调用。

现在 `Validation` 模块已完全定义，我们可以看到如何使用它。让我们从打开 `Validation` 模块并定义一个简单的记录类型定义开始，之后我们可以针对这个类型进行验证。

```
open Validation
type TestType = { ObjectValue : obj
                  StringValue : string
                  IntValue : int }
```

这个类型没有什么特别之处，它仅包含了我们可以引用用于验证的三个标签。现在，我们可以通过如下方式调用 `createValidator` 来创建一个验证方法：

```
let validate =
  createValidator <| fun x -> notNull <@ x.ObjectValue @> >>
                              notEmpty <@ x.StringValue @> >>
                              between <@ x.IntValue @> 1 100
```

在这里，我们通过在传递给`createValidator`的函数中使用组合操作符，将对`notNull`、`notEmpty`和`between`的调用链式连接起来。最终返回的函数（由`createValidator`返回）然后绑定到`validate`。每个链式调用都包含一个引用表达式，用于标识`TestType`的标签。你甚至可以看到这里，F#的类型推断在确定表达式中`x`的类型时发挥了作用。

现在我们需要做的就是通过传递`TestType`的实例来调用`validate`函数。当所有值满足验证时，`validate`会像这样简单地返回一个空列表：

```
> **{ ObjectValue = obj(); StringValue = "Sample"; IntValue = 35 }**
**|> validate;;**
val it : (string * string) list = []
```

另一方面，当一个或多个值未通过验证时，`validate`函数会返回一个列表，包含失败的成员名称以及失败信息，如这里所示，所有三个值都失败了：

```
> **{ ObjectValue = null; StringValue = ""; IntValue = 1000 }**
**|> validate;;**
val it : (string * string) list =
  [("IntValue", "Must be at least 100 and greater than 1");
   ("StringValue", "Cannot be empty"); ("ObjectValue", "Is a required field")]
```

## 概要

尽管引用表达式的作用与 LINQ 引入的表达式树类似，但 F#的引用表达式更适合函数式编程。正如你所看到的，你可以通过字面表达式、使用`ReflectedDefinition`特性通过反射直接构造引用表达式，或通过反射和`Expr`类中的静态方法编程构造引用表达式。然而，引用表达式的真正力量来自于它们的解构。通过使用在`Patterns`、`DerivedPatterns`和`ExprShape`模块中定义的活动模式，你可以在不同粒度上解构引用表达式，从而完成多种任务，如语言翻译甚至灵活的验证。
