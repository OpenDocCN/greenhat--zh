## 第五章：让我们来了解函数式编程

我之前多次提到过 F# 是一种函数式语言，但正如你从前面的章节中学到的，你可以在 F# 中构建丰富的应用程序，而不使用任何函数式技巧。这是否意味着 F# 并不是真正的函数式语言呢？不是的。F# 是一种通用的多范式语言，允许你以最适合任务的风格进行编程。它被认为是一种函数式优先语言，这意味着它的结构鼓励使用函数式风格。换句话说，在 F# 中开发时，你应该尽可能倾向于函数式方法，并在合适的情况下切换到其他风格。

在本章中，我们将了解函数式编程的真正含义，以及 F# 中的函数与其他语言中的函数有何不同。一旦我们建立了这一基础，我们将探索几种在函数式编程中常用的数据类型，并简要了解惰性求值。

## 什么是函数式编程？

函数式编程在软件开发中采用了与面向对象编程根本不同的方法。面向对象编程主要关注于管理一个不断变化的系统状态，而函数式编程则强调不可变性和确定性函数的应用。这种差异极大地改变了你构建软件的方式，因为在面向对象编程中，你主要关心的是定义类（或结构体），而在函数式编程中，你的重点是定义函数，特别强调它们的输入和输出。

F# 是一种不纯的函数式语言，其中数据默认是不可变的，尽管你仍然可以定义可变数据或在函数中引起其他副作用。不可变性是函数式编程概念的一部分，称为*参照透明性*，这意味着一个表达式可以被其结果替代，而不影响程序的行为。例如，如果你可以用 `let sum = 15` 替换 `let sum = add 5 10` 而不影响程序的行为，那么 `add` 被称为具有参照透明性。但不可变性和参照透明性只是函数式编程的两个方面，它们并不会单独让一种语言变成函数式语言。

## 使用函数进行编程

如果你从未进行过任何“真实”的函数式编程，F# 将永远改变你对函数的思考方式，因为它的函数在结构和行为上都与数学函数高度相似。例如，第三章介绍了 `unit` 类型，但我避开了讨论它在函数式编程中的重要性。与 C# 和 Visual Basic 不同，F# 不区分返回值的函数和不返回值的函数。事实上，F# 中的每个函数都接受恰好一个输入值并返回恰好一个输出值。`unit` 类型使这种行为成为可能。当一个函数没有特定输入（没有参数）时，它实际上接受 `unit`。类似地，当一个函数没有特定输出时，它返回 `unit`。

每个 F# 函数都返回一个值，这一事实使得编译器可以对你的代码做出某些假设。一个重要的假设是函数中最后一个被评估的表达式是函数的返回值。这意味着，虽然 `return` 是 F# 中的一个关键字，但你不需要明确地标识返回值。

### 函数作为数据

任何函数式语言的一个定义性（也是可能最重要的）特征是它把函数当作任何其他数据类型来处理。虽然 .NET Framework 一直在某种程度上支持这个概念（通过委托），但直到最近，委托由于过于繁琐，几乎只在少数几个有限的场景中可行。直到引入 LINQ 和 Lambda 表达式的好处，以及内置的泛型委托类型（`Action` 和 `Func`），委托才真正发挥了其全部潜力。F# 在幕后使用委托，但与 C# 和 Visual Basic 不同，它的语法通过 `->` 符号抽象化了委托。`->` 符号通常读作“传递到”或“返回”，它标识一个值是一个*函数值*，其中左侧指定的数据类型是函数的输入类型，右侧的数据类型是返回类型。例如，一个既接受又返回字符串的函数签名是 `string -> string`。类似地，一个没有参数并返回字符串的函数表示为 `unit -> string`。

当你开始使用*高阶函数*（接受或返回其他函数的函数）时，函数签名会变得越来越复杂。高阶函数在 F#（以及函数式编程一般）中被广泛使用，因为它们允许你隔离函数的公共部分，并替换那些会变化的部分。

从某种意义上说，高阶函数对函数式编程的意义，就像接口对面向对象编程的意义一样。例如，考虑一个将转换应用于字符串并打印结果的函数。它的签名可能类似于 `(string -> string) -> string -> unit`。这个简单的符号大大提高了代码的可理解性，比直接处理委托要容易得多。

### 注意

*你可以在类型注解中使用函数签名，只要你期待一个函数。和其他数据类型一样，编译器通常能够推断出函数类型。*

### 互操作性考虑

尽管 F# 函数最终是基于委托的，但在与其他 .NET 语言编写的库一起使用时要小心，因为委托类型不能互换。F# 函数依赖于重载的 `FSharpFunc` 委托类型，而传统的 .NET 委托通常基于 `Func` 和 `Action` 类型。如果你需要将 `Func` 和 `Action` 委托传入 F# 程序集中，可以使用以下类来简化转换。

```
open System.Runtime.CompilerServices

[<Extension>]
type public FSharpFuncUtil =
  [<Extension>]
  static member ToFSharpFunc<'a, 'b> (func : System.Func<'a, 'b>) =
    fun x -> func.Invoke(x)

  [<Extension>]
  static member ToFSharpFunc<'a> (act : System.Action<'a>) =
    fun x -> act.Invoke(x)
```

`FSharpFuncUtil` 类定义了重载的 `ToFSharpFunc` 方法，作为传统的 .NET 扩展方法（通过在类和方法上使用 `ExtensionAttribute`），这样你就可以轻松地从其他语言调用它们。第一个重载处理单参数 `Func` 实例，而第二个重载处理单参数 `Action` 实例。这些扩展方法并没有覆盖所有使用场景，但它们无疑是一个不错的起点。

## 柯里化

F# 中的函数与你可能习惯的方式有些不同。例如，考虑在第二章中介绍的简单 `add` 函数。

```
let add a b = a + b
```

你可能会认为 `add` 接受两个参数，但 F# 函数并不是这样工作的。记住，在 F# 中，每个函数接受恰好一个输入并返回恰好一个输出。如果你在 FSI 中创建上述绑定或在 Visual Studio 中悬停在名称上，你会看到它的签名是：

```
val add : a:int -> b:int -> int
```

在这里，`add` 被绑定到一个接受整数（`a`）并返回一个函数的函数。返回的函数接受一个整数（`b`）并返回一个整数。理解这种自动的函数链式调用——称为*柯里化*——对于有效使用 F# 至关重要，因为它启用了多个影响你如何设计函数的其他特性。

为了更好地说明柯里化是如何工作的，让我们重新编写 `add`，使其更接近编译后的代码。

```
> **let add a = fun b -> (+) a b;;**

val add : a:int -> b:int -> int
```

这里最重要的是，这个版本和之前的版本具有完全相同的签名。然而，在这里，`add` 只接受一个参数（`a`）并返回一个由 lambda 表达式定义的独立函数。返回的函数接受第二个参数（`b`）并调用乘法运算符，作为另一个函数调用。

### 部分应用

柯里化函数解锁的一项功能是部分应用。*部分应用* 允许你通过提供部分参数来从现有函数创建新函数。例如，在 `add` 的情况下，你可以使用部分应用来创建一个新的 `addTen` 函数，它总是将 10 加到一个数字上。

```
> **let addTen = add 10;;**

val addTen : ① (int -> int)
> **addTen 10;;**
val it : int = 20
```

请注意在①处如何列出了 `addTen` 的定义和签名。尽管我们在定义中没有明确包括任何参数，但签名仍然是一个接受并返回整数的函数。编译器根据提供的参数（在这种情况下仅为 `10`）尽可能地计算柯里化的 `add` 函数，并将结果函数绑定到 `addTen` 这个名称上。

柯里化一次应用一个参数，从左到右，因此部分应用的参数必须对应于函数的第一个参数。

### 警告

*一旦你熟悉了柯里化和部分应用，你可能会开始考虑是否可以通过返回 Func 或 Action 实例在 C# 或 Visual Basic 中模拟它们。不要这么做。这两种语言并不支持这种类型的函数式编程，因此模拟这些概念充其量是笨拙的，最糟糕的情况下极容易出错。*

### 管道化

与柯里化（currying）常常关联的另一个特性（并在 F# 中广泛使用）是管道化（pipelining）。*管道化*允许你通过计算一个表达式并将结果作为最终参数传递给另一个函数，来创建自己的函数链。

#### 前向管道化

通常，你会使用*前向管道化操作符*（`|>`）将值传递给下一个函数。如果你不想对函数返回的结果做任何处理（当它返回的结果不是 `unit` 时），你可以像这样将结果传递给 `ignore` 函数：

```
add 2 3 |> ignore
```

管道化不仅限于像忽略结果这样的简单场景。只要接收函数的最后一个参数与源函数的返回类型兼容，你就可以创建复杂的函数链。例如，假设你有一个包含每日温度（以华氏度为单位）的列表，并且你想计算平均温度、将其转换为摄氏度并打印结果。你可以采用传统的过程式方式为每个步骤定义绑定，或者你可以使用管道化将这些步骤链在一起，如下所示：

```
let fahrenheitToCelsius degreesF = (degreesF - 32.0) * (5.0 / 9.0)

let marchHighTemps = [ 33.0; 30.0; 33.0; 38.0; 36.0; 31.0; 35.0;
                       42.0; 53.0; 65.0; 59.0; 42.0; 31.0; 41.0;
                       49.0; 45.0; 37.0; 42.0; 40.0; 32.0; 33.0;
                       42.0; 48.0; 36.0; 34.0; 38.0; 41.0; 46.0;
                       54.0; 57.0; 59.0 ]
**marchHighTemps**
**|> List.average**
**|> fahrenheitToCelsius**
**|> printfn "March Average (C): %f"**
```

这里，`marchHighTemps` 列表被传递到 `List` 模块的 `average` 函数。接着，`average` 函数被计算，其结果传递给 `fahrenheitToCelsius` 函数。最后，摄氏温度的平均值被传递给 `printfn`。

#### 反向管道化

与前向管道化操作符类似，*反向管道化操作符*（`<|`）将表达式的结果作为最终参数传递给另一个函数，但它是从右到左进行的。由于它改变了表达式中的优先级，反向管道化操作符有时可以替代括号使用。

反向管道化操作符可能会改变你代码的语义。例如，在上一节中的 `fahrenheitToCelsius` 示例中，重点放在温度列表上，因为它是首先列出的。若要改变语义以强调输出，你可以将 `printfn` 函数调用放在反向管道化操作符之前。

```
printfn "March Average (F): %f" <| List.average marchHighTemps
```

#### 非柯里化函数

尽管流水线通常与柯里化函数相关联，但它也适用于仅接受单一参数的非柯里化函数（如方法）。例如，为了强制延迟执行，你可以将一个值传递给 `TimeSpan` 类的静态 `FromSeconds` 方法，然后将生成的 `TimeSpan` 对象传递给 `Thread.Sleep`，如下所示。

```
5.0
|> System.TimeSpan.FromSeconds
|> System.Threading.Thread.Sleep
```

因为 `TimeSpan` 类和 `Thread` 类都没有在 F# 中定义，所以这些函数并未被柯里化，但你可以看到如何使用前向流水线运算符将这些函数链在一起。

### 函数组合

与流水线类似，*函数组合*允许你创建函数链。它有两种形式：前向（`>>`）和后向（`<<`）。

函数组合遵循与流水线相同的输入输出规则。函数组合的不同之处在于，组合运算符不仅定义一次性操作，而是生成新的函数。继续使用我们的平均温度示例，你可以使用前向组合运算符轻松地将 `List.average` 和 `fahrenheitToCelsius` 函数组合成一个新函数。

```
> **let averageInCelsius = List.average >> fahrenheitToCelsius;;**

val averageInCelsius : (float list -> float)
```

组合运算符会生成一个新的函数，该函数接受一个浮点数列表并返回一个浮点数。现在，你不再需要独立调用这两个函数，而是可以直接调用 `averageInCelsius`。

```
printfn "March average (C): %f" <| **averageInCelsius marchHighTemps**
```

与流水线一样，你可以将非柯里化函数组合起来。例如，你也可以将强制延迟示例从非柯里化函数中组合起来。

```
> **let delay = System.TimeSpan.FromSeconds >> System.Threading.Thread.Sleep;;**

val delay : (float -> unit)
```

正如你所期望的那样，你现在可以调用 `delay` 函数来暂时暂停执行。

```
> **delay 5.0;;**
val it : unit = ()
```

## 递归函数

通常与命令式代码相关联的循环构造有三种：`while` 循环、简单的 `for` 循环和可枚举的 `for` 循环。因为每种循环都依赖于状态变化来确定退出条件，所以在编写纯函数式代码时，你需要采取不同的循环方式。在函数式编程中，首选的循环机制是*递归*。*递归函数*是指直接或间接通过另一个函数调用自身的函数。

尽管类型内的方法是隐式递归的，但通过 `let` 绑定的函数（如模块内定义的函数）并不是递归的。要使一个 `let` 绑定的函数递归，你必须在其定义中包含 `rec` 关键字，正如这个阶乘函数所示。

```
let **rec** factorial v =
  match v with | 1L -> 1L
               | _ -> v * **factorial (v - 1L)**
```

`rec` 关键字指示编译器在函数内使函数名可用，但不会改变函数的签名（`int64 -> int64`）。

### 尾递归

前面的阶乘示例很简单，但它存在一个重大缺陷。例如，考虑调用`factorial 5`时会发生什么。在每次递归迭代中（当值不为 1 时），该函数都会计算`v`与`v - 1`的阶乘的乘积。换句话说，为给定值计算阶乘本质上需要每个后续的阶乘调用都完成。运行时，它看起来大致如下：

```
5L * (factorial 4L)
5L * (4L * (factorial 3L))
5L * (4L * (3L * (factorial 2L)))
-- *snip* --
```

上面的代码片段显示了每个调用都会被加入到栈中。对于阶乘函数来说，这不太可能成为问题，因为计算很快就会溢出数据类型，但更复杂的递归场景可能会导致栈空间耗尽。为了解决这个问题，可以通过删除对后续迭代的依赖，将函数修改为使用*尾递归*，如下所示：

```
① let factorial v =
  let ② rec fact c p =
    match c with | 0L -> p
                 | _ ->  ③ fact <| c - 1L <| c * p
  ④ fact v 1L
```

修改后的阶乘函数①创建并调用了一个嵌套的递归函数`fact`②，来隔离实现细节。`fact`函数接收当前迭代值（`c`）和前一次迭代计算的积（`p`）。在③（非零情况下），`fact`函数进行递归调用。（注意，递归调用的参数只有在此处计算。）最后，为了启动递归，`factorial`函数④调用第一个`fact`迭代，传入提供的值和`1L`。

尽管递归调用仍然存在于代码中，但当 F#编译器检测到没有迭代依赖于后续迭代时，它会通过将递归替换为命令式循环来优化编译后的形式。这允许系统根据需要进行迭代。你可以通过插入断点并查看调用栈窗口（如果你以控制台应用程序运行此代码）或打印出从`System.Diagnostics.StackTrace`返回的栈信息来观察这种优化，如下所示。（请注意，你的命名空间可能会有所不同。）

```
**Standard recursion**
   at FSI_0024.printTrace()
   at FSI_0028.factorial(Int64 v)
   at FSI_0028.factorial(Int64 v)
   at FSI_0028.factorial(Int64 v)
   at FSI_0028.factorial(Int64 v)
   at FSI_0028.factorial(Int64 v)
   at <StartupCode$FSI_0029>.$FSI_0029.main@()
   -- *snip* --

**Tail recursion**
   at FSI_0024.printTrace()
   at FSI_0030.fact@75-8(Int64 c, Int64 p)
   at <StartupCode$FSI_0031>.$FSI_0031.main@()
   -- *snip* --
```

### 互递归函数

当两个或更多的函数互相递归调用时，它们被称为*互递归*。像互递归类型（在第四章中描述的那样）一样，互递归函数必须一起定义，并使用`and`关键字。例如，斐波那契数的计算可以通过互递归轻松表达。

```
let fibonacci n =
  **let rec f = function**
              **| 1 -> 1**
              **| n -> g (n - 1)**
  **and g = function**
          **| 1 -> 0**
          **| n -> g (n - 1) + f (n - 1)**
  f n + g n
```

上面的`fibonacci`函数定义了两个互递归函数，`f`和`g`。（每个内部的`function`关键字是模式匹配的快捷方式。）对于所有值不为 1 的情况，`f`调用`g`。类似地，`g`递归地调用自己和`f`。

由于互递归被隐藏在`fibonacci`内部，代码的使用者可以直接调用`fibonacci`。例如，要计算斐波那契数列中的第六个数字，可以这样写：

```
> **fibonacci 6;;**
val it : int = 8
```

互递归可能很有用，但这个例子实际上仅适用于说明概念。出于性能考虑，一个更现实的 Fibonacci 示例可能会放弃互递归，改为使用一种叫做 *备忘录化* 的技术，其中昂贵的计算只进行一次，结果会被缓存，以避免多次计算相同的值。

## Lambda 表达式

如果你曾经使用过 LINQ 或做过其他函数式编程，你可能已经熟悉 *lambda 表达式*（有时也叫 *函数表达式*）。lambda 表达式在函数式编程中被广泛使用。简而言之，它们提供了一种方便的方式来定义简单的、单次使用的匿名（无名）函数。当函数仅在其上下文中有意义时（例如，在过滤集合时），lambda 表达式通常比 `let` 绑定的函数更受欢迎。

Lambda 表达式的语法类似于函数值，只不过它以 `fun` 关键字开头，省略了函数标识符，并且使用箭头符号（`->`）代替等号。例如，你可以将华氏度到摄氏度的转换函数作为 lambda 表达式内联表示，并立即像这样求值：

```
**(fun degreesF -> (degreesF - 32.0) * (5.0 / 9.0))** 212.0
```

尽管像这样定义临时函数确实是 lambda 表达式的一种用途，但它们更常见的是与高阶函数的调用一起内联创建，或者被包含在管道链中。

## 闭包

*闭包*使得函数能够访问在其定义的作用域内可见的值，无论该值是否是函数的一部分。尽管闭包通常与 lambda 表达式相关联，但使用 `let` 绑定创建的嵌套函数也可以是闭包，因为它们最终都会编译为 `FSharpFunc` 或正式的方法。闭包通常用于隔离某些状态。例如，考虑经典的闭包示例——一个返回能够操作内部计数器值的函数，如下所示：

```
let createCounter() =
  let count = ref 0
  (fun () -> count := !count + 1
             !count)
```

`createCounter` 函数定义了一个由返回的函数捕获的引用单元。因为引用单元在返回函数创建时处于作用域内，所以该函数无论何时被调用，都可以访问它。这使得你可以在没有正式类型定义的情况下模拟一个有状态的对象。

要观察函数修改引用单元值的过程，我们只需要调用生成的函数，并像这样调用它：

```
let increment = createCounter()
for i in [1..10] do printfn "%i" (increment())
```

## 函数类型

F# 原生支持几种额外的数据类型。这些类型——元组、记录和区分联合类型——通常与函数式编程相关联，但它们在混合范式开发中也非常有用。虽然这些类型各有特定的用途，但它们的共同目标是帮助你始终关注你的软件正在解决的问题。

### 元组

最基本的函数式类型是 *元组*。元组是将多个值组合成一个单一不可变结构的便捷方式，而无需创建自定义类型。元组通常表示为以逗号分隔的列表，有时会被括在括号中。例如，下面两个表示几何点的元组定义都是有效的。

```
> **let point1 = 10.0, 10.0;;**

val point1 : float * float = (10.0, 10.0)

> **let point2 = (20.0, 20.0);;**

val point2 : float * float = (20.0, 20.0)
```

元组类型的签名包括每个值的类型，类型之间用星号（`*`）分隔。星号作为元组元素的分隔符是出于数学原因：元组表示它们元素所包含的所有值的笛卡尔积。因此，要在类型注解中表达元组，你应该将其写成一个以星号分隔的类型列表，如下所示：

```
let point : **float * float** = 0.0, 0.0
```

尽管在语法上有一些相似之处，尤其是当值被括号括起来时，但重要的是要认识到，除了包含多个值这一点之外，元组并不是集合；它们只是将固定数量的值组合在一个单一的结构中。元组类型并未实现 `IEnumerable<'T>`，因此不能在可枚举的 `for` 循环中进行枚举或迭代，并且单个元组值只能通过像 `Item1` 和 `Item2` 这样的非特定名称的属性来访问。

.NET 中的元组

元组一直是 F# 的一部分，但直到 .NET 4 才被引入到更大的 .NET Framework 中。在 .NET 4 之前，元组类位于 `FSharp.Core` 库中，但它们现在已被移到 `mscorlib`。这个差异只有在你打算编写针对早期版本 .NET Framework 的跨语言代码时才重要，因为它会影响你引用的程序集。

#### 提取值

元组常用于从函数返回多个值，或者在不进行柯里化的情况下将多个值传递给函数。例如，计算一条直线的斜率时，你可以将两个点作为元组传递给 `slope` 函数。为了使函数工作，你需要某种方式来访问单独的值。（幸运的是，元组值总是按定义的顺序可以访问，因此减少了很多猜测的工作。）

在处理 *对偶*（包含两个值的元组，例如我们之前讨论的几何点）时，你可以使用 `fst` 和 `snd` 函数分别获取第一个和第二个值，如此处所示。

```
let slope p1 p2 =
  let x1 = **fst** p1
  let y1 = **snd** p1
  let x2 = **fst** p2
  let y2 = **snd** p2
  (y1 - y2) / (x1 - x2)

slope (13.0, 8.0) (1.0, 2.0)
```

注意我们如何使用 `fst` 和 `snd` 函数定义不同坐标的绑定。不过，如你所见，以这种方式提取每个值可能会变得相当繁琐，而且这些函数仅适用于对偶（包含两个值的元组）；如果你尝试在 *三元组*（包含三个值的元组）上使用它们，你会遇到类型不匹配的问题。（原因在于，元组在底层会编译为 `Tuple` 类的九种通用重载之一。）除了共享相同的名称外，元组类相互独立且通常不兼容。

更实际的提取元组值的方法是引入*元组模式*。元组模式允许你通过用逗号分隔标识符来为元组中的每个值指定一个标识符。例如，这里是修改后的 `slope` 函数，使用元组模式而不是对偶函数。

```
let slope p1 p2 =
  let **x1, y1** = p1
  let **x2, y2** = p2
  (y1 - y2) / (x1 - x2)
```

你可以看到元组模式如何提供帮助，但你需要小心使用它们。如果你的模式与元组中的值数量不匹配，你将得到类型不匹配的错误。

幸运的是，不像对偶函数那样，解决这个问题仅仅是添加或删除标识符的问题。如果你不关心元组模式中的某个特定值，可以使用通配符模式（`_`）忽略它。例如，如果你有三维坐标，但只关心 z 坐标，你可以按如下方式忽略 x 和 y 值：

```
> let **_, _, z** = (10.0, 10.0, 10.0);;

val z : int = 10
```

元组模式不限于 `let` 绑定。实际上，我们可以进一步修改 `slope` 函数，并直接在函数签名中包含模式！

```
let slope **(x1, y1) (x2, y2)** = (y1 - y2) / (x1 - x2)
```

#### 相等性语义

尽管它们在形式上是引用类型，但每种内置的元组类型都实现了 `IStructuralEquatable` 接口。这确保了所有的相等性比较都涉及比较每个组件的值，而不是检查两个元组实例是否引用了内存中相同的 `Tuple` 对象。换句话说，当两个元组实例中对应组件的值相同时，它们被认为是相等的，如下所示：

```
> **(1, 2) = (1, 2);;**
val it : bool = true
> **(2, 1) = (1, 2);;**
val it : bool = false
```

由于 `fst` 和 `snd` 函数仅适用于对偶，比较不同长度的元组将会导致错误。

#### 语法元组

到目前为止，我们查看的所有元组都是具体的元组，但 F# 还包括了*语法元组*。在大多数情况下，语法元组是 F# 处理其他语言中非柯里化函数的方式。因为 F# 函数总是接受一个参数，而 C# 和 Visual Basic 中的函数可以接受多个参数，为了调用其他语言编写的库中的函数，你可以使用语法元组，让编译器处理细节。

例如，`String` 类的 `Format` 方法同时接受一个格式字符串和一个 `params` 参数数组。如果 `String.Format` 是一个柯里化函数，你会期望它的签名类似于 `Format : format:string -> params args : obj [] -> string`，但实际并非如此。相反，如果你将鼠标悬停在 Visual Studio 中的函数名上，你会看到它的签名实际上是 `Format(format:string, params args : obj []) : string`。这种区别非常重要，因为它意味着参数必须作为一个整体传递，而不是像柯里化函数那样逐个传递。如果你尝试以柯里化 F# 函数的方式调用这个方法，你将得到类似如下的错误：

```
> **System.String.Format "hello {0}" "Dave";;**

  System.String.Format "hello {0}" "Dave";;
  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

stdin(3,1): error FS0003: This value is not a function and cannot be applied
```

在 F# 中调用 `String.Format` 的正确方式是使用语法元组，如下所示：

```
> **System.String.Format ("hello {0}", "Dave");;**
val it : string = "hello Dave"
```

你可能已经注意到，F# 通常在调用函数时不需要在参数周围加括号；它主要使用括号来确定优先级。由于函数是从左到右应用的，你主要会在函数调用中使用括号来将另一个函数的结果作为参数传递。在这种情况下，参数周围的括号是必要的。没有它们，左到右的求值将导致编译器基本上将表达式当作 `((System.String.Format "hello {0}"), "Dave")` 处理。一般来说，最好在语法元组周围加上括号，以消除任何歧义。

#### `out` 参数

F# 并不直接支持 `out` 参数——即通过引用传递并在方法体内赋值，以便返回给调用者的参数。为了完全支持 .NET Framework，F# 需要一种方法来访问 `out` 参数值。例如，多个数值数据类型类中的 `TryParse` 方法尝试将字符串转换为相应的数值类型，并返回一个表示成功或失败的布尔值。如果转换成功，`TryParse` 方法将 `out` 参数设置为适当的转换值。例如，调用 `System.Int32.TryParse` 并传入 `"10"` 将返回 `true` 并将 `out` 参数设置为 `10`。类似地，调用相同的函数并传入 `"abc"` 将返回 `false` 并保持 `out` 参数不变。

在 C# 中，调用 `System.Int32.TryParse` 看起来是这样的：

```
  // C#
① int v;
  var r = System.Int32.TryParse("10", **out v**);
```

在函数式语言中，`out` 参数的问题在于它们需要副作用，正如①处的未初始化变量所示。为了绕过这个问题，F# 编译器将返回值和 `out` 参数转换为一对。因此，当你在 F# 中调用带有 `out` 参数的方法时，你就像调用任何返回元组的函数一样处理它。

在 F# 中调用相同的 `Int32.TryParse` 方法看起来是这样的：

```
// F#
let r, v = System.Int32.TryParse "10"
```

要查看生成类的幕后细节，我们可以再次使用 ILSpy 来查看它在 C# 中的表示方式。

```
// C#
using System;
using System.Diagnostics;
using System.Runtime.CompilerServices;
namespace <StartupCode$Samples>
{
  internal static class $Samples
  {
    [DebuggerBrowsable(DebuggerBrowsableState.Never)]
    internal static readonly Tuple<bool, int> patternInput@3;
    [DebuggerBrowsable(DebuggerBrowsableState.Never)]
    internal static readonly int v@3;
    [DebuggerBrowsable(DebuggerBrowsableState.Never)]
    internal static readonly bool r@3;
    [DebuggerBrowsable(DebuggerBrowsableState.Never), DebuggerNonUserCode, CompilerGenerated]
    internal static int init@;
  ① static $Samples()
    {
      int item = 0;
      $Samples.patternInput@3 = ② new Tuple<bool, int>(③ int.TryParse("10", out item), item);
      ④ $Samples.v@3 = Samples.patternInput@3.Item2;
      ⑤ $Samples.r@3 = Samples.patternInput@3.Item1;
    }
  }
}
```

在这里，F# 编译器将 `Int32.TryParse` 调用封装在一个静态类中。生成的类的静态构造函数 ① 在 ③ 处调用 `TryParse` 并将结果封装在元组中 ②。然后，内部的 `v@3` 和 `r@3` 字段分别在 ④ 和 ⑤ 处被赋值为 `out` 参数值和返回值。反过来，通过 `let` 绑定定义的 `v` 和 `r` 值被编译成只读属性，这些属性返回 `v@3` 和 `r@3` 的值。

### 记录类型

像元组一样，*记录类型*允许你将值组合成一个不可变的结构。你可以把它们看作是在元组和自定义类之间架起的功能性桥梁。记录类型提供了与元组相同的许多便利性，如简单的语法和值相等语义，同时还允许你对其内部结构进行一定控制，并能够添加自定义功能。

#### 定义记录类型

记录类型定义由`type`关键字、标识符以及所有用大括号括起来的标签与类型注释列表组成。例如，以下列出的是一个表示 RGB 颜色的简单记录类型。

```
> **type rgbColor = { R : byte; G : byte; B : byte };;**

type rgbColor =
  {R: byte;
   G: byte;
   B: byte;}
```

如果你查看编译器从这个定义生成的内容，你会看到一个封闭的类，带有只读属性、相等语义以及一个用于初始化所有值的构造函数。

### 注意

*在定义单行记录类型时，必须用分号分隔每个标签和类型注释对。如果将每对标签和类型注释放在单独的一行，则可以安全地省略分号。*

#### 创建记录

新的记录是通过*记录表达式*创建的。记录表达式允许您为记录类型中的每个标签指定一个值。例如，您可以使用记录表达式创建一个新的`rgbColor`实例，如下所示。（请注意，与定义记录类型时一样，您必须用分号分隔每个标签或赋值对，或者将其放在单独的一行。）

```
> **let red = { R = 255uy; G = 0uy; B = 0uy };;**

val red : rgbColor = {R = 255uy;
                      G = 0uy;
                      B = 0uy;}
```

请注意，在记录表达式中，我们没有包含对`rgbColor`类型的显式引用。这是 F#类型推断引擎工作原理的另一个例子。仅凭标签，编译器就能够推断出我们正在创建一个`rgbColor`实例。因为编译器依赖标签而非位置来确定正确的类型，所以顺序不重要。这意味着您可以以任意顺序放置标签和值对。在这里，我们以`G`、`B`、`R`的顺序创建了一个`rgbColor`实例。

```
> **let red = { G = 0uy; B = 0uy; R = 255uy };;**

val red : rgbColor = {R = 255uy;
                      G = 0uy;
                      B = 0uy;}
```

与元组不同，我们不需要像`fst`或`snd`这样的特殊值提取函数来处理记录类型，因为每个值可以通过它的标签来访问。例如，一个将`rgbColor`值转换为其十六进制字符串等效值的函数可能是这样的：

```
let rgbColorToHex (c : rgbColor) =
  sprintf "#%02X%02X%02X" **c.R c.G c.B**
```

#### 避免命名冲突

编译器通常能够推断出正确的类型，但也可能定义两个具有相同结构的记录类型。考虑一下当你添加一个与`rgbColor`结构相同的`color`类型时会发生什么。

```
> **type rgbColor = { R : byte; G : byte; B : byte }**
**type color = { R : byte; G : byte; B : byte };;**

type rgbColor =
  {R: byte;
   G: byte;
   B: byte;}
type color =
  {R: byte;
   G: byte;
   B: byte;}

> **let red = { R = 255uy; G = 0uy; B = 0uy };;**

val red : ① color = {R = 255uy;
                     G = 0uy;
                     B = 0uy;}
```

尽管有两个结构相同的记录类型，类型推断仍然成功，但请注意，在①处，生成的类型是`color`。由于 F#的自顶向下评估，编译器使用最最近定义的与标签匹配的类型。如果您的目标是将`red`定义为`color`，那是没问题的，但如果您希望使用`rgbColor`，则必须在记录表达式中稍微明确一些，并包括类型名，如下所示：

```
> **let red = {** ① **rgbColor.R = 255uy; G = 0uy; B = 0uy };;**

val red : ② rgbColor = {R = 255uy;
                        G = 0uy;
                        B = 0uy;}
```

通过在①处使用类型名限定其中一个名称，您可以绕过类型推断，正确的类型会在②处被解析。（尽管技术上您可以在任何名称上限定类型，但惯例是只在第一个名称或所有名称上限定类型。）

#### 复制记录

你不仅可以使用记录表达式从头创建新的记录实例，还可以通过将值向前复制并为一个或多个属性设置新值来从现有实例创建新的记录实例。另一种语法，称为*复制和更新记录表达式*，使得从红色创建黄色变得简单，如下所示：

```
> let red = { R = 255uy; G = 0uy; B = 0uy }
let yellow = **{ red with G = 255uy };;**

val red : color = {R = 255uy;
                   G = 0uy;
                   B = 0uy;}
val yellow : color = {R = 255uy;
                      G = 255uy;
                      B = 0uy;}
```

要为多个属性指定新值，使用分号分隔它们。

#### 可变性

像 F#中的几乎所有其他内容一样，记录类型默认是不可变的。然而，由于其语法非常便捷，它们通常被用来代替类。然而，在许多情况下，这些场景要求可变性。为了在 F#中使记录类型的属性可变，可以像使用`let`绑定一样使用`mutable`关键字。例如，你可以像这样使`rgbColor`的所有成员变为可变：

```
> **type rgbColor = { mutable R : byte**
                  **mutable G : byte**
                  **mutable B : byte };;**

type rgbColor =
  {mutable R: byte;
   mutable G: byte;
   mutable B: byte;}
```

当记录类型的属性是可变的时，你可以像这样使用标准赋值运算符（`<-`）来改变其值：

```
let myColor = { R = 255uy; G = 255uy; B = 255uy }
myColor.G <- 100uy
```

Climutable

虽然记录类型默认支持二进制序列化，但其他形式的序列化需要默认构造函数和可写属性。为了在更多情况下使用记录类型代替类，F#团队在 F# 3.0 中引入了`CLIMutable`属性。

使用此属性装饰记录类型会指示编译器包含一个默认构造函数，并使生成的属性可读/可写，但编译器不会在 F#中暴露这些功能。即使生成的属性是可写的，除非它们在记录类型定义中明确标记为`mutable`，否则它们的值无法在 F#代码中更改。因此，在跨语言边界使用`CLIMutable`记录类型时，要小心以确保不会无意中改变某些内容。

#### 附加成员

因为记录类型实际上只是类的语法糖，所以你可以像在类中一样定义附加成员。例如，你可以像这样增加一个方法，使`rgbColor`返回其十六进制字符串等价物：

```
type rgbColor = { R : byte; G : byte; B : byte }
                **member x.ToHexString() =**
                  **sprintf "#%02X%02X%02X" x.R x.G x.B**
```

现在，你可以在任何`rgbColor`实例上调用`ToHexString`方法。

```
> **red.ToHexString();;**
val it : string = "#FF0000"
```

记录类型上的附加成员也可以是静态的。例如，假设你想将几个常见颜色暴露为记录类型上的静态属性。你可以这样做：

```
type rgbColor = { R : byte; G : byte; B : byte }
                -- *snip* --
                **static member Red = { R = 255uy; G = 0uy; B = 0uy }**
                **static member Green = { R = 0uy; G = 255uy; B = 0uy }**
                **static member Blue = { R = 0uy; G = 0uy; B = 255uy }**
```

静态的`Red`、`Green`和`Blue`属性像其他任何静态成员一样，可以在需要`rgbColor`实例的地方使用。

```
> **rgbColor.Red.ToHexString();;**
val it : string = "#FF0000"
```

你还可以为记录类型创建自定义运算符作为静态成员。我们来实现一个加法运算符来添加两个`rgbColor`实例。

```
open System
type rgbColor = { R : byte; G : byte; B : byte }
                -- *snip* --
                **static member (+) (l : rgbColor, r : rgbColor) =**
                  **{ R = Math.Min(255uy, l.R + r.R)**
                   **G = Math.Min(255uy, l.G + r.G)**
                   **B = Math.Min(255uy, l.B + r.B) }**
```

对`rgbColor`的运算符重载像任何其他运算符一样被定义和调用：

```
> **let yellow = { R = 255uy; G = 0uy; B = 0uy } +**
             **{ R = 0uy; G = 255uy; B = 0uy };;**

val yellow : rgbColor = {R = 255uy;
                         G = 255uy;
                         B = 0uy;}
```

## 区别联合

*区别联合*是用户定义的数据类型，其值被限制为一组已知的值，称为*联合案例*。其他流行的.NET 语言中没有等价的结构。

刚开始时，你可能会因为语法非常相似而把一些简单的区分联合类型误认为枚举类型，但它们实际上是完全不同的构造。首先，枚举类型仅仅是为已知的整数值定义标签，但它们并不局限于这些值。相比之下，区分联合类型的唯一有效值是它们的联合案例。此外，每个联合案例可以独立存在，或者包含关联的不可变数据。

内置的`Option<'T>`类型突出了这些要点。我们这里只对它的定义感兴趣，因此我们来看一下它的定义。

```
type Option<'T> =
| None
| Some of 'T
```

`Option<'T>`定义了两个案例，`None`和`Some`。`None`是一个空的联合案例，意味着它不包含任何关联的数据。另一方面，`Some`有一个与之关联的`'T`实例，如`of`关键字所示。

为了演示区分联合类型如何强制执行一组特定的值，让我们定义一个简单的函数，接受一个泛型选项，当选项是`Some`时输出关联的值，或者当选项是`None`时输出`"None"`：

```
let showValue (v : _ option) =
  printfn "%s" (match v with
                | Some x -> x.ToString()
                | None -> "None")
```

当我们调用这个函数时，我们只需要提供其中一个选项案例：

```
> **Some 123 |> showValue;;**
123
val it : unit = ()
> **Some "abc" |> showValue;;**
abc
val it : unit = ()
> **None |> showValue;;**
None
val it : unit = ()
```

注意，在对`showValue`的三次调用中，我们只指定了联合案例的名称。编译器将`Some`和`None`都解析为`Option<'T>`。 (如果发生命名冲突，你可以像使用记录类型时一样，使用区分联合类型名称来限定案例名称。) 然而，如果你用除`Some`或`None`之外的值来调用`showValue`，编译器会抛出如下错误：

```
> **showValue "xyz";;**

  showValue "xyz";;
  ----------^^^^^

stdin(9,11): error FS0001: This expression was expected to have type
    Option<'a>
but here has type
    string
```

### 定义区分联合类型

与其他类型一样，区分联合类型的定义以`type`关键字开头。联合案例之间用竖线分隔。第一个联合案例前的竖线是可选的，但在只有一个案例的情况下省略它可能会引起混淆，因为它会使定义看起来像是类型缩写。事实上，如果在单案例的区分联合类型中省略竖线，并且该案例没有与之关联的数据，当与其他类型发生命名冲突时，编译器会将该定义视为类型缩写。

定义联合案例时，标识符的正常规则适用，但有一个例外：联合案例的名称必须以大写字母开头，以帮助编译器在模式匹配中区分联合案例与其他标识符。如果案例名称不是以大写字母开头，编译器会抛出错误。

实际上，区分联合类型通常有三个用途：

+   表示简单的对象层次结构

+   表示树形结构

+   替代类型缩写

#### 简单的对象层次结构

区分联合类型通常用于表示简单的对象层次结构。实际上，它们在这方面表现得非常出色，以至于它们常常被用作正式类和继承的替代方案。

假设你正在开发一个需要一些基本几何功能的系统。在面向对象的环境中，这些功能可能包括一个 `IShape` 接口和一些具体的形状类，例如 `Circle`、`Rectangle` 和 `Triangle`，它们都实现了 `IShape` 接口。一个可能的实现如下所示：

```
type IShape = interface end

type Circle(r : float) =
  interface IShape
  member x.Radius = r

type Rectangle(w : float, h : float) =
  interface IShape
  member x.Width = w
  member x.Height = h

type Triangle(l1 : float, l2 : float, l3 : float) =
  interface IShape
  member x.Leg1 = l1
  member x.Leg2 = l2
  member x.Leg3 = l3
```

被区分的联合类型提供了一种更简洁的替代方案，且不易引发副作用。以下是该对象层次结构在被区分联合类型下可能的样子：

```
type Shape =
/// Describes a circle by its radius
| Circle of float
/// Describes a rectangle by its width and height
| Rectangle of ① float * float
/// Describes a triangle by its three sides
| Triangle of ② float * float * float
```

它在内部更大

被区分的联合类型比其语法看起来要复杂得多。每个被区分的联合类型会编译成一个抽象类，负责处理相等性和比较语义、类型检查以及联合情况的创建。类似地，每个联合情况会编译成一个类，既嵌套在联合类内部，又继承自联合类。联合情况类定义了每个关联值的属性和存储，并包含一个内部构造函数。

尽管可以在其他语言中模拟某些被区分联合类型的功能，但这样做并不简单。为了证明被区分联合类型的复杂性，我们在 ILSpy 中检查刚刚定义的 `Shape` 类型时，发现它竟然生成了近 700 行 C# 代码！

`Shape` 类型定义了三种情况：`Circle`、`Rectangle` 和 `Triangle`。每种情况都有至少一个与其代表的形状相关联的值。请注意在①和②中，如何使用元组语法将多个数据值与一个情况关联。尽管使用了元组语法，但情况实际上并不会编译成元组。相反，每个关联的数据项会编译成一个独立的属性，并遵循元组命名模式（即 `Item1`、`Item2` 等）。这一区别很重要，因为从联合情况到元组没有直接的转换，这意味着你不能将它们互换使用。唯一的例外是，当类型被括号包裹时，编译器会将分组解释为元组。换句话说，编译器将 `of string * int` 和 `of (string * int)` 区别对待；前者是类似元组的，而后者实际上就是元组。不过，除非你确实需要一个真正的元组，否则请使用默认格式。

正如你所预期的，创建 `Shape` 实例的方式与创建 `Option<'T>` 实例的方式相同。例如，以下是如何创建每个情况的实例：

```
let c = Circle(3.0)
let r = Rectangle(10.0, 12.0)
let t = Triangle(25.0, 20.0, 7.0)
```

使用元组语法表示多个关联值的一个主要问题是很容易忘记每个位置代表什么。为了解决这个问题，可以在每个情况前面包含 XML 文档注释——就像本节中 `Shape` 定义前面的注释那样，作为提醒。

幸运的是，问题已经得到解决。F# 3.1 中的一项语言增强是支持命名联合体类型字段。经过精炼的语法看起来像是当前的元组语法和类型注解字段定义的混合。例如，在新语法下，`Shape`可以重新定义如下。

```
type Shape =
| Circle of Radius : float
| Rectangle of Width : float * Height : float
| Triangle of Leg1 : float * Leg2 : float * Leg3 : float
```

对于使用 F# 3.1 语法定义的区分联合体，创建新的案例实例对开发者更友好——不仅因为标签会出现在 IntelliSense 中，还因为您可以像这样使用命名参数：

```
let c = Circle(Radius = 3.0)
let r = Rectangle(Width = 10.0, Height = 12.0)
let t = Triangle(Leg1 = 25.0, Leg2 = 20.0, Leg3 = 7.0)
```

#### 树结构

区分联合体也可以是*自引用的*，这意味着与联合体某个案例相关的数据可以是同一联合体中的另一个案例。这对于创建像这样简单的树结构非常有用，它表示一个基本的标记结构：

```
type Markup =
| ContentElement of string * ① Markup list
| EmptyElement of string
| Content of string
```

这个定义中的大部分应该已经很熟悉了，但请注意，`ContentElement`案例有一个关联的字符串和`Markup`类型值的列表。

嵌套的`Markup`列表①使得构建一个简单的 HTML 文档变得非常简单，像下面这样。在这里，`ContentElement`节点表示包含额外内容的元素（如`html`、`head`和`body`），而`Content`节点表示包含在`ContentElement`中的原始文本。

```
let movieList =
  ContentElement("html",
    [ ContentElement("head", [ ContentElement("title", [ Content "Guilty Pleasures" ])])
      ContentElement("body",
        [ ContentElement("article",
            [ ContentElement("h1", [ Content "Some Guilty Pleasures" ])
              ContentElement("p",
                [ Content "These are "
                  ContentElement("strong", [ Content "a few" ])
                  Content " of my guilty pleasures" ])
              ContentElement("ul",
                [ ContentElement("li", [ Content "Crank (2006)" ])
                  ContentElement("li", [ Content "Starship Troopers (1997)" ])
                  ContentElement("li", [ Content "RoboCop (1987)" ])])])])])
```

要将前面的树结构转换为实际的 HTML 文档，您可以编写一个简单的递归函数，并使用匹配表达式处理每个联合体案例，像这样：

```
let rec toHtml markup =
  match markup with
  | ① ContentElement (tag, children) ->
        use w = new System.IO.StringWriter()
        children
          |> Seq.map toHtml
          |> Seq.iter (fun (s : string) -> w.Write(s))
        sprintf "<%s>%s</%s>" tag (w.ToString()) tag
  | ② EmptyElement (tag) -> sprintf "<%s />" tag
  | ③ Content (c) -> sprintf "%s" c
```

这里的`match`表达式大致类似于 C# 中的 `switch` 语句或 Visual Basic 中的 `SELECT CASE` 语句。每个匹配案例，由竖线符号（`|`）表示，与一个标识符模式匹配，该模式包括联合体案例的名称和其所有相关值的标识符。例如，①处的匹配案例匹配`ContentElement`项，并在案例体（箭头后面的部分）中使用`tag`和`children`标识符表示相关值。同样，②和③处的匹配案例分别匹配`EmptyElement`和`Content`案例。（请注意，由于匹配表达式会返回一个值，因此每个匹配案例的返回类型必须相同。）

使用`movieList`调用`toHtml`函数会生成以下 HTML（已格式化以便阅读）。在查看生成的 HTML 时，试着追溯每个元素在`movieList`中的节点。

```
<html>
  <head>
    <title>Guilty Pleasures</title>
  </head>
  <body>
    <article>
        <h1>Some Guilty Pleasures</h1>
        <p>These are <strong>a few</strong> of my guilty pleasures</p>
        <ul>
            <li>Crank (2006)</li>
            <li>Starship Troopers (1997)</li>
            <li>RoboCop (1987)</li>
        </ul>
    </article>
  </body>
</html>
```

#### 替换类型缩写

单案例区分联合体可以作为类型缩写的有用替代方案，虽然类型缩写对于创建现有类型的别名很方便，但它们并不会提供额外的类型安全性。例如，假设您将`UserId`定义为`System.Guid`的别名，并且有一个函数`UserId -> User`。尽管该函数接受`UserId`，但没有任何东西可以阻止您传入任意的`Guid`，无论该`Guid`实际代表什么。

让我们扩展前一节的标记示例，展示单案例区分联合体如何解决这个问题。如果您想在浏览器中显示生成的 HTML，您可以定义一个像这样的函数：

```
  open System.IO

① type HtmlString = string

  let displayHtml (html ②: HtmlString) =
    let fn = Path.Combine(Path.GetTempPath(), "HtmlDemo.htm")
    let bytes = System.Text.UTF8Encoding.UTF8.GetBytes html
    using (new FileStream(fn, FileMode.Create, FileAccess.Write))
          (fun fs -> fs.Write(bytes, 0, bytes.Length))
    System.Diagnostics.Process.Start(fn).WaitForExit()
    File.Delete fn
```

`displayHtml`函数的实际机制对于这个讨论并不重要。相反，请将注意力集中在① `HtmlString`类型别名和② 类型注解明确声明`html`参数是`HtmlString`。

从签名可以明显看出，`displayHtml`函数期望传入的字符串包含 HTML，但由于`HtmlString`仅仅是类型的别名，并不能确保它实际上是 HTML。按照当前写法，`movieList |> toHtml |> displayHtml`和`"abc123" |> displayHtml`都是有效的。

为了引入更多的类型安全性，我们可以用单案例区分联合类型替换`HtmlString`定义，如下所示：

```
type HtmlString = | HtmlString of string
```

由于`HtmlString`现在是一个区分联合类型，我们需要修改`displayHtml`函数以提取关联的字符串。我们可以通过两种方式实现这一点。第一种方法需要我们更改函数的签名以包括标识符模式。或者，我们可以不改变签名，改为引入一个中间绑定（同样使用标识符模式）来处理关联值。第一种方法更简洁，因此我们将使用这种方法。

```
let displayHtml **(HtmlString(html))** =
  let fn = Path.Combine(Path.GetTempPath(), "HtmlDemo.htm")
  let bytes = System.Text.UTF8Encoding.UTF8.GetBytes html
  using (new FileStream(fn, FileMode.Create, FileAccess.Write))
        (fun fs -> fs.Write(bytes, 0, bytes.Length))
  System.Diagnostics.Process.Start(fn).WaitForExit()
  File.Delete fn
```

要调用`displayHtml`函数，我们只需要将`toHtml`函数的字符串包装在`HtmlString`实例中，并将其传递给`displayHtml`，如下所示：

```
HtmlString(movieList |> toHtml) |> displayHtml
```

最后，我们可以通过修改`toHtml`函数返回`HtmlString`而不是字符串来进一步简化这段代码。一种做法如下所示：

```
let rec toHtml markup =
  match markup with
  | ContentElement (tag, children) ->
        use w = new System.IO.StringWriter()
        children
          |> Seq.map toHtml
          |> Seq.iter (fun ① (HtmlString(html)) -> w.Write(html))
        HtmlString (sprintf "<%s>%s</%s>" tag (w.ToString()) tag)
  | EmptyElement (tag) -> HtmlString (sprintf "<%s />" tag)
  | Content (c) -> HtmlString (sprintf "%s" c)
```

在这个修订版中，我们将每个案例的返回值包装在`HtmlString`实例中。然而，更不平凡的是①，现使用标识符模式从递归结果中提取 HTML，以便将原始文本写入`StringWriter`。

现在，`toHtml`函数返回一个`HtmlString`，将其结果传递给`displayHtml`简化为如下代码：

```
movieList |> toHtml |> displayHtml
```

单案例区分联合类型无法保证任何关联值实际上是正确的，但它们提供了一些额外的安全性，迫使开发者在传递给函数时做出有意识的决定。开发者可以创建一个包含任意字符串的`HtmlString`实例，但如果这样做，他们将被迫考虑数据是否正确。

### 附加成员

与记录类型类似，区分联合类型也允许附加成员。例如，我们可以将`toHtml`函数重新定义为`Markup`区分联合类型上的方法，如下所示：

```
type Markup =
| ContentElement of string * Markup list
| EmptyElement of string
| Content of string

member x.toHtml() =
  match x with
  | ContentElement (tag, children) ->
        use w = new System.IO.StringWriter()
        children
          |> Seq.map (fun m -> m.toHtml())
          |> Seq.iter (fun (HtmlString(html)) -> w.Write(html))
        HtmlString (sprintf "<%s>%s</%s>" tag (w.ToString()) tag)
  | EmptyElement (tag) -> HtmlString (sprintf "<%s />" tag)
  | Content (c) -> HtmlString (sprintf "%s" c)
```

调用这个方法就像调用其他类型的方法一样：

```
movieList.toHtml() |> displayHtml
```

## 惰性求值

默认情况下，F#使用*急切求值*，这意味着表达式会立即求值。大多数情况下，急切求值在 F#中是没问题的，但有时你可以通过推迟执行直到结果真正需要时来提高感知性能，这就是*惰性求值*。

F#支持一些启用懒惰求值的机制，但最简单和最常见的方法之一是通过使用`lazy`关键字。在这里，`lazy`关键字与一系列包含延迟的表达式结合使用，以模拟一个长时间运行的操作。

```
**> let lazyOperation = lazy (printfn "evaluating lazy expression"**
                          **System.Threading.Thread.Sleep(1000)**
                          **42);;**

val lazyOperation : Lazy<int> = Value is not created.
```

你可以看到`lazy`关键字的影响。如果这个表达式是立即求值的，那么`evaluating lazy expression`将会被打印，并且在返回`42`之前会有一个即时的延迟一秒钟。相反，这个表达式的结果是内置的`Lazy<'T>`类型的一个实例。在这种情况下，编译器推断出返回类型，并创建了一个`Lazy<int>`的实例。

### 注意

*小心跨语言边界使用懒类型。在 F# 3.0 之前，`Lazy<'T>`类位于 FSharp.Core 程序集。在.NET 4.0 中，`Lazy<'T>`被移动到了`mscorlib`。*

由`lazy`关键字创建的`Lazy<'T>`实例可以像其他类型一样传递，但底层的表达式不会被求值，直到你通过调用`Force`方法或访问它的`Value`属性来强制求值，如下所示。通常约定更倾向于使用`Force`方法，但其实无论你使用它还是`Value`属性来强制求值都没有关系。在内部，`Force`只是一个扩展方法，它封装了`Value`属性。

```
> **lazyOperation.Force() |> printfn "Result: %i";;**
evaluating lazy expression
Result: 42
val it : unit = ()
```

现在我们已经强制求值，我们看到底层的表达式已经打印了它的信息，休眠了一段时间，并返回了`42`。`Lazy<'T>`类型也可以通过记忆化提高应用程序性能。一旦相关的表达式被求值，它的结果会被缓存到`Lazy<'T>`实例中，并在随后的请求中使用。如果该表达式涉及一个昂贵或耗时的操作，结果可能会非常显著。

为了更有效地观察记忆化的影响，我们可以在 FSI 中启用计时，并重复强制求值，如下所示：

```
> **let lazyOperation = lazy (System.Threading.Thread.Sleep(1000); 42)**
**#time "on";;**

val lazyOperation : Lazy<int> = Value is not created.

--> Timing now on

> **lazyOperation.Force() |> printfn "Result: %i";;**
Result: 42
Real: ① 00:00:01.004, CPU: 00:00:00.000, GC gen0: 0, gen1: 0, gen2: 0
val it : unit = ()
> **lazyOperation.Force() |> printfn "Result: %i";;**
Result: 42
Real: ② 00:00:00.001, CPU: 00:00:00.000, GC gen0: 0, gen1: 0, gen2: 0
val it : unit = ()
> **lazyOperation.Force() |> printfn "Result: %i";;**
Result: 42
Real: ③ 00:00:00.001, CPU: 00:00:00.000, GC gen0: 0, gen1: 0, gen2: 0
val it : unit = ()
```

如你所见，在①处，第一次调用`Force`时，我们付出了让线程休眠的代价。随后在②和③处的调用则瞬间完成，因为记忆化机制已经缓存了结果。

## 总结

正如你在本章中看到的，函数式编程要求与面向对象编程不同的思维方式。面向对象编程强调管理系统状态，而函数式编程则更关注通过将无副作用的函数应用于数据来确保程序的正确性和可预测性。像 F#这样的函数式语言将函数视为数据。这样，它们通过更高阶函数、柯里化、部分应用、管道化和函数组合等概念，在系统内部实现了更大的组合性。像元组、记录类型和判别联合等函数式数据类型，通过让你专注于解决问题，而不是试图满足编译器，帮助你编写正确的代码。
