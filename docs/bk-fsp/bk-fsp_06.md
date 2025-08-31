## 第六章：走进集合

编程任务通常需要处理数据集合。 .NET 框架一直通过如*数组*和`ArrayList`类等构造来支持这一场景，但直到 .NET 2.0 引入泛型后，集合支持才真正成熟。

F# 在 .NET 的基础上进行扩展，不仅支持所有现有的集合类型，还带来了一些自己的集合类型。在本章中，我们将看到一些经典集合类型在 F# 中的作用，接着我们将探索 F# 特有的集合类型。在此过程中，我们将看到内建的集合模块如何为传统类型和 F# 特有类型增添一些函数式的特色，使得处理这些类型变得轻松自如。

## 序列

在 .NET 中，*序列*是指一类具有共同类型的值的集合。更具体地说，序列是实现了`IEnumerable<'T>`接口的任何类型。

几乎所有 .NET 中的主要集合类型都是序列。例如，泛型集合类型（如 `Dictionary<'TKey, 'TValue>` 和 `List<'T>`），甚至一些通常不被视为集合的类型（如 `String`）也实现了 `IEnumerable<'T>`。相反，遗留集合类型（如 `ArrayList` 和 `Hashtable`）早于泛型推出，因此它们只实现了非泛型的 `IEnumerable` 接口。因此，它们并不强制执行单一的公共类型，通常被视为可枚举集合，而非序列。

在 F# 中，`IEnumerable<'T>`通常表示为`seq<'T>`或`'T seq`。像`values : 'A seq`这样的类型注解会被编译成`IEnumerable<'A>`，任何实现了`IEnumerable<'T>`的类型都可以在预期序列的地方使用。不过要小心直接使用具体的集合类型，因为底层实现可能是可变的。

### 创建序列

今天的 .NET 开发人员已经习惯了与序列打交道，但在 LINQ 引入之前，直接对 `IEnumerable<'T>` 进行编程相对较为罕见。开发人员通常会针对特定的集合类型编写代码。尽管如此，LINQ 的 `IEnumerable<'T>` 扩展方法将这一抽象推到了前台，教会了开发人员，他们并不总是需要知道集合的任何信息，除了它实现了 `GetEnumerator` 方法。即便 LINQ 带来了所有的便利，它也仅仅是为操作 `IEnumerable<'T>` 提供了一个框架；在 LINQ 中创建任意序列仍然需要一个方法来实例化特定的序列类型。

F#通过像序列和范围表达式这样的概念，进一步将序列创建的抽象化，超越了 LINQ 的范畴。尽管每个序列最终仍然是`IEnumerable<'T>`的实现，编译器可以自由地提供自己的实现。`Seq`模块还包括多个函数，用于创建新的序列。

#### 序列表达式

*序列表达式*允许你通过反复应用其他 F#表达式并*生成*（返回）结果到一个新序列中，从而创建新的序列。在某些情况下，特别是当你处理大型或计算开销大的集合时，序列表达式内部使用的序列类型比其他集合类型更为合适，因为它们只在需要时才会创建值。这些序列类型通常也一次只在内存中存储一个值，因此它们非常适合处理大型数据集。

### 注意

*序列表达式从技术上讲是一种内置的工作流，称为*计算表达式*。我们将在第十二章中详细介绍这些构造。*

你可以通过将一个或多个表达式封装在序列构建器内，并使用`do`绑定配合`yield`关键字来创建一个序列表达式。例如，假设你有一个名为*ArnoldMovies.txt*的文件，其中包含以下数据：

```
The Terminator,1984
Predator,1987
Commando,1985
The Running Man,1987
True Lies,1994
Last Action Hero,1993
Total Recall,1990
Conan the Barbarian,1982
Conan the Destroyer,1984
Hercules in New York,1969
```

你可以使用如下的序列表达式将文本文件中的每一行读入一个序列：

```
let lines = **seq {** use r = new System.IO.StreamReader("ArnoldMovies.txt")
                  while not r.EndOfStream **do yield** r.ReadLine() **}**
```

这里，使用`while`循环反复从`StreamReader`读取每一行，并在每次迭代时返回一行。（在某些简单的序列表达式中，例如使用可枚举的`for`循环时，`do yield`可以用`->`运算符替代，但为了保持一致性，我通常使用`do yield`。）

如果你想将这个序列写入控制台，你可以将它传递给`printfn`函数并使用默认格式化器（通过`%A`标记），但只会输出前四个值，如下所示：

```
> **lines |> printfn "%A";;**
seq ["The Terminator,1984"; "Predator,1987"; "Commando,1985"; "The Running Man,1987"; ...]
val it : unit = ()
```

要打印序列中的每个值，你需要强制对整个构造进行枚举。

#### 范围表达式

尽管*范围表达式*在形式上类似于你在第四章中学到的切片表达式，因为它们使用`..`运算符，但它们实际上是专门的序列表达式，允许你在一个值的范围内创建序列。范围表达式类似于`Enumerable.Range`方法，但它们更强大，因为它们不限于整数。例如，你可以像这样轻松创建一个包含从 0 到 10 的整数的序列：

```
seq { 0..10 }
```

或者，你可以通过以下方式创建一个包含从 0 到 10 的浮点数的序列：

```
seq { 0.0..10.0 }
```

同样，你可以像这样创建一个包含字符*a*到*z*的序列：

```
seq { 'a'..'z' }
```

在大多数情况下，您还可以包括一个值，用于标识生成序列时要跳过多少个值。例如，使用以下表达式创建一个包含从 0 到 100 的 10 的倍数的序列是很容易的：

```
seq { 0..10..100 }
```

这种范围表达式形式仅适用于数值类型，因此不能与字符数据一起使用。例如，以下表达式会导致错误。

```
seq { 'a'..2..'z' }
```

最后，你可以通过使用负步长值来创建一个值递减的序列，如下所示：

```
seq { 99..-1..0 }
```

#### 空序列

当你需要一个没有任何元素的序列时，可以使用`Seq`模块的通用`empty`函数来创建。例如，创建一个空字符串序列，你可以像这样调用`Seq.empty`：

```
> **let emptySequence = Seq.empty<string>;;**

val emptySequence : seq<string>
```

或者，如果你不需要特定的类型，可以通过省略类型参数，让编译器自动推断序列类型：

```
> **let emptySequence = Seq.empty;;**

val emptySequence : seq<'a>
```

#### 初始化序列

另一个模块函数，`Seq.init`，可以创建一个包含最多指定数量元素的序列。例如，要创建一个包含 10 个随机数的序列，你可以这样写：

```
> **let rand = System.Random();;**

val rand : System.Random

> **Seq.init 10 (fun _ -> rand.Next(100));;**
val it : seq<int> = seq [22; 34; 73; 42; ...]
```

### 操作序列

`Seq`模块提供了许多用于操作任何序列的函数。接下来将介绍的函数列表是`Seq`模块中最有用的一些函数，但并不全面。

虽然接下来讨论的每个函数都属于`Seq`模块，但许多函数在其他集合模块中也有专门的对应函数。为了节省篇幅，我只会讲解常见的函数，但我强烈鼓励你探索其他模块，并发现适合你任务的工具。

什么时候函数不是一个函数？

你可能已经注意到，在两个空序列示例中，`Seq.empty`都没有传入任何参数。`Seq.empty`与我们迄今为止遇到的每个函数不同，它更像是一个基本的值绑定，而不是一个函数。事实上，如果你传入一个参数调用`Seq.empty`，你会得到一个编译器错误，提示你该值（`Seq.empty`）不是一个函数，不能被应用。

为什么`Seq.empty`被称为一个函数，而编译器却声称不是？因为它与一些其他函数（如`Operators.typeof`和`Operators.typedefof`）一起，是一个特殊情况值，称为*类型函数*。类型函数通常保留给那些根据其类型参数计算值的纯函数，因此——尽管它们在编译后的程序集里表现为方法——它们在 F#代码中仍然被视为值。

#### 查找序列长度

你可以像这样使用`Seq.length`来确定序列包含多少个元素：

```
seq { 0..99 } |> Seq.length
```

不过要小心使用`Seq.length`，因为根据底层集合类型，它可能会强制枚举整个序列，或者以其他方式影响性能。考虑以下代码，它使用`Seq.length = 0`检查序列是否为空：

```
seq { for i in 1..10 do
      printfn "Evaluating %i" i
      yield i }
|> Seq.length = 0
```

要确定序列的长度，系统必须通过调用枚举器的 `MoveNext` 方法来遍历序列，直到它返回 `false`。每次调用 `MoveNext` 都涉及执行获取下一个值所需的任何工作。在这个例子中，获取下一个值涉及将字符串写入控制台，如下所示：

```
Evaluating 1
Evaluating 2
Evaluating 3
-- *snip* --
Evaluating 10
val it : bool = false
```

向控制台写入一些文本是微不足道的，但即便如此，这仍然是多余的工作，因为结果实际上并没有被用于任何地方。如果超出这个简单的例子，你可以很容易地想象每次调用 `MoveNext` 都会触发一个昂贵的计算或数据库调用。如果你只需要确定序列是否有元素，你应该使用 `Seq.isEmpty` 函数。

`Seq.isEmpty` 检查一个序列是否包含任何元素，而不需要强制遍历整个序列。考虑以下代码，它将 `Seq.length = 0` 替换为 `Seq.isEmpty`：

```
seq { for i in 1..10 do
      printfn "Evaluating %i" i
      yield i }
|> Seq.isEmpty
```

因为 `Seq.isEmpty` 一旦找到元素就会返回 `false`，所以 `MoveNext` 只会被调用一次，结果是：

```
Evaluating 1
val it : bool = false
```

如你所见，尽管序列表达式定义了 10 个元素，但只有第一个被打印，因为一旦函数找到一个值，评估就停止了。

#### 遍历序列

`Seq.iter` 函数是功能等价于可枚举 `for` 循环的函数，它遍历一个序列，并对每个元素应用一个函数。例如，要打印一个包含从 0 到 99 的值的序列中的每个元素，你可以这样写：

```
> **seq { 0..99 } |> Seq.iter (printfn "%i");;**
0
1
2
-- *snip* --
97
98
99
val it : unit = ()
```

#### 转换序列

`Seq.map` 类似于 `Seq.iter`，它将一个函数应用于序列中的每个元素，但与 `Seq.iter` 不同的是，它会用结果构建一个新的序列。例如，要创建一个新的序列，其中包含来自序列的元素的平方，你可以这样写：

```
> **seq { 0..99 } |> Seq.map (fun i -> i * i);;**
val it : seq<int> = seq [0; 1; 4; 9; ...]
```

#### 排序序列

`Seq` 模块定义了几个排序序列的函数。每个排序函数都会创建一个新的序列，原始序列保持不变。

最简单的排序函数 `Seq.sort` 使用基于 `IComparable<'T>` 接口的默认比较方式对元素进行排序。例如，你可以将 `Seq.sort` 应用于一组随机整数序列，如下所示：

```
> **let rand = System.Random();;**

val rand : System.Random

> **Seq.init 10 (fun _ -> rand.Next 100) |> Seq.sort;;**
val it : seq<int> = seq [0; 11; 16; 19; ...]
```

对于更复杂的排序需求，你可以使用 `Seq.sortBy` 函数。除了需要排序的序列外，它还接受一个函数，该函数返回用于排序的每个元素的值。

例如，*ArnoldMovies.txt* 中列出的每部电影在 序列表达式 中都包含上映年份。如果你想按上映年份排序这些电影，你可以修改序列表达式，提取出各个值，如下所示：

```
let movies =
  seq { use r = new System.IO.StreamReader("ArnoldMovies.txt")
        while not r.EndOfStream do
          let l = r.ReadLine().Split(',')
          yield ① l.[0], int l.[1] }
```

在 ① 处，序列表达式现在返回包含每部电影标题和上映年份的*元组*。我们可以将序列与 `snd` 函数（获取年份）一起发送到 `Seq.sortBy`，如下所示：

```
> **movies |> Seq.sortBy snd;;**
val it : seq<string * int> =
  seq
    [("Hercules in New York", 1969); ("Conan the Barbarian", 1982);
     ("The Terminator", 1984); ("Conan the Destroyer", 1984); ...]
```

或者，为了按标题排序电影，你可以将 `snd` 替换为 `fst`。

```
> seq { use r = new System.IO.StreamReader(fileName)
      while not r.EndOfStream do
        let l = r.ReadLine().Split(',')
        yield l.[0], int l.[1] }
|> **Seq.sortBy fst**;;
val it : seq<string * int> =
  seq
    [("Commando", 1985); ("Conan the Barbarian", 1982);
     ("Conan the Destroyer", 1984); ("Hercules in New York", 1969); ...]
```

#### 过滤序列

当你只希望处理符合特定条件的元素时，可以使用 `Seq.filter` 函数来创建一个只包含符合条件的元素的新序列。例如，继续使用电影主题，你可以像这样获取 1984 年之前上映的电影：

```
> **movies |> Seq.filter (fun (_, year) -> year < 1985);;**
val it : seq<string * int> =
  seq
    [("The Terminator", 1984); ("Conan the Barbarian", 1982);
     ("Conan the Destroyer", 1984); ("Hercules in New York", 1969)]
```

#### 聚合序列

`Seq` 模块提供了多个函数，用于聚合序列中的元素。最灵活（也是最复杂）的聚合函数是 `Seq.fold`，它遍历序列，对每个元素应用一个函数，并将结果作为累加器值返回。例如，`Seq.fold` 使得计算序列元素之和变得非常简单：

```
> **seq { 1 .. 10 } |> Seq.fold** ① **(fun s c -> s + c)** ② **0;;**
val it : int = 55
```

这个例子展示了如何以一种方式将 1 到 10 的值相加。`Seq.fold` 用于聚合的函数①接受两个值：一个聚合值（本质上是一个累加总和），以及当前元素。我们还需要给 `fold` 函数提供一个初始聚合值②，通常用 `0` 来表示。随着 `fold` 的执行，它会将聚合函数应用到序列中的每个元素，并返回新的聚合值以供下一次迭代使用。

由于加法运算符本身满足聚合函数的要求，我们可以像这样简化之前的表达式：

```
> **seq { 1..10 } |> Seq.fold (+) 0;;**
val it : int = 55
```

一个稍微更为专门的聚合函数是 `Seq.reduce`。`reduce` 函数与 `fold` 函数非常相似，不同之处在于传递给计算的聚合值始终与序列的元素类型相同，而 `fold` 可以将数据转换为其他类型。`reduce` 函数与 `fold` 的另一个区别是，它不接受初始聚合值。相反，`reduce` 会将聚合值初始化为序列中的第一个值。为了看到 `Seq.reduce` 的实际效果，我们可以将之前的表达式改写如下：

```
> **seq { 1 .. 10 } |> Seq.reduce (+);;**
val it : int = 55
```

如预期的那样，不论是使用 `Seq.fold` 还是 `Seq.reduce`，序列中元素相加的结果是相同的。

`Seq.fold` 和 `Seq.reduce` 并不是计算序列聚合值的唯一方法；一些常见的聚合操作，如求和和平均数，已经有了专门的函数。例如，我们可以使用 `Seq.sum` 来计算元素的总和，而不必像之前那样使用 `Seq.reduce`：

```
> seq { 1..10 } |> **Seq.sum;;**
val it : int = 55
```

同样地，要计算平均数，可以像这样使用 `Seq.average`：

```
> seq { 1.0..10.0 } |> **Seq.average;;**
val it : float = 5.5
```

需要注意的是，`Seq.average` 只适用于支持整数除法的类型。如果你尝试用一个整数序列来使用它，你会遇到如下错误：

```
> seq { 1..10 } |> Seq.average;;

  seq { 1..10 } |> Seq.average;;
  -----------------^^^^^^^^^^^

stdin(2,18): error FS0001: The type 'int' does not support the operator 'DivideByInt'
```

和 `Seq.sort` 类似，`Seq.sum` 和 `Seq.average` 函数也有对应的 `Seq.sumBy` 和 `Seq.averageBy` 函数，它们接受一个函数，让你指定应使用哪个值来进行计算。这些函数的语法与 `Seq.sortBy` 相同，因此我会留给你自己去多做一些关于 `Seq` 模块的实验。

## 数组

F#数组和传统的.NET 数组是相同的结构。它们包含固定数量的值（每个值的类型相同），并且是零索引的。尽管数组绑定本身是不可变的，但单个数组元素是可变的，因此你需要小心不要引入不必要的副作用。也就是说，数组的可变性在某些情况下使得它们比其他集合构造更具吸引力，因为改变元素值不需要进一步的内存分配。

### 创建数组

F#提供了多种方式来创建新的数组，并控制每个元素的初始值，既可以使用原生语法，也可以使用模块函数。

#### 数组表达式

创建数组的最常见方式之一是使用*数组表达式*。数组表达式由一个以分号分隔的值列表组成，这些值被`[|`和`|]`标记包围。例如，你可以像这样创建一个字符串数组（如果每个值单独写在一行上，你可以省略分号）：

```
> **let names = [| "Rose"; "Martha"; "Donna"; "Amy"; "Clara" |];;**

val names : string [] = [|"Rose"; "Martha"; "Donna"; "Amy"; "Clara"|]
```

最后，你可以通过将一个序列表达式包含在`[|`和`|]`之间来生成一个数组。然而，与序列构造器不同的是，当数组表达式被求值时，数组将完全构造出来。将这个例子与序列表达式讨论中的对应例子进行比较：

```
> **let lines = [| use r = new System.IO.StreamReader("ArnoldMovies.txt")**
                  **while not r.EndOfStream do yield r.ReadLine() |];;**

val lines : string [] =
  [|"The Terminator,1984"; "Predator,1987"; "Commando,1985";
    "The Running Man,1987"; "True Lies,1994"; "Last Action Hero,1993";
    "Total Recall,1990"; "Conan the Barbarian,1982";
    "Conan the Destroyer,1984"; "Hercules in New York,1969"|]
```

如你所见，默认的数组打印格式化器会打印每个元素（它将输出限制在 100 个元素），而不是仅打印前四个元素。

#### 空数组

如果你需要创建一个空数组，可以使用一对空的方括号：

```
let emptyArray = [| |]
```

这种方法的缺点是，根据上下文的不同，你可能需要添加类型注解，以确保编译器不会自动将数组泛化。这样的定义看起来可能是这样的：

```
let emptyArray : int array = [| |];;
```

在前面的例子中，类型注解`int array`是一种类似英语的语法。如果你更喜欢传统的形式，你也可以使用`int[]`。如果没有类型注解，编译器将把数组定义为`'a []`。

创建空数组的另一种方式是使用`Array.empty`函数。和`Seq`模块中的对应函数一样，`Array.empty`是一个类型函数，因此你可以不带任何参数调用它来创建一个零长度的数组。要使用这个函数创建一个空的字符串数组，只需写：

```
Array.empty<string>
```

如果你更愿意让编译器推断底层类型或自动泛化它，你可以省略类型参数。

#### 初始化数组

如果你想快速创建一个所有元素都初始化为基础类型默认值的数组，可以使用`Array.zeroCreate`。假设你知道需要一个包含五个字符串的数组，但还不知道每个元素中将存储什么值。你可以像这样创建这个数组：

```
> **let stringArray = Array.zeroCreate<string> 5;;**

val stringArray : string [] = [|null; null; null; null; null|]
```

因为`Array.zeroCreate`使用底层类型的默认值，因此可能会将元素初始化为`null`，就像这里一样。如果`null`对该类型有效，并且你正在像这样创建数组，那么你需要编写代码以防止`NullReferenceException`。

或者，`Array.init`允许你将每个元素初始化为特定值。`Array.init`是`Seq.init`的数组专用等价物。它的语法相同，但它创建并返回一个数组。例如，要创建一个新数组，其中的元素被初始化为空字符串，你可以这样写：

```
> **let stringArray = Array.init 5 (fun _ -> "");;**

val stringArray : string [] = [|""; ""; ""; ""; ""|]
```

这里，提供的函数仅返回空字符串，但你的初始化函数可以轻松地包含更复杂的逻辑，允许你为每个元素计算不同的值。

### 使用数组

在 F#中使用数组与在其他.NET 语言中使用它们类似，但 F#通过诸如切片表达式和`Array`模块等构造扩展了数组的用途。

#### 访问元素

通过索引属性可以访问单个数组元素。例如，要从之前定义的`lines`数组中检索第四个元素，你可以这样写：

```
> **lines.[3];;**
val it : string = "The Running Man,1987"
```

你可以将索引器语法与赋值运算符结合，来改变数组的单个元素。例如，要替换*Last Action Hero*，你可以这样写：

```
lines.[5] <- "Batman & Robin,1997"
```

如果你更喜欢以一种更函数式的方法来检索和修改数组元素，`Array`模块通过`get`和`set`函数提供了支持。在下面的示例中，我们将创建一个数组，改变第二个元素的值，检索新值，并将其输出到控制台。

```
> **let movies = [| "The Terminator"; "Predator"; "Commando" |];;**

val movies : string [] = [|"The Terminator"; "Predator"; "Commando"|]

> **Array.set movies 1 "Batman & Robin"**
**Array.get movies 1 |> printfn "%s";;**
Batman & Robin

val it : unit = ()
```

最后，数组还支持切片表达式。如第四章中所述，切片表达式可以让你轻松地从集合中检索一系列值，像这样：

```
> **lines.[1..3];;**
val it : string [] =
  [|"Predator,1987"; "Commando,1985"; "The Running Man,1987"|]
```

#### 复制数组

你可以通过`Array.copy`轻松地将一个数组的元素复制到新数组中。在这里，我们创建一个包含数字 1 到 10 的数组，并立即将它们复制到另一个数组中。

```
[| 1..10 |] |> Array.copy
```

在后台，`Array.copy`是对 CLR 的`Array.Clone`方法的封装，该方法创建源数组的浅拷贝。`Array.copy`提供了额外的好处，即自动将`Clone`返回的对象实例强制转换为适当的数组类型；也就是说，将一个整数数组直接传递给`Array.Clone`会得到一个`obj`实例，而将该数组传递给`Array.copy`则会得到一个`int array`实例。

#### 排序数组

数组可以像其他序列一样进行排序，但`Array`模块提供了一些专门的排序函数，以利用单个数组元素是可变这一事实。不幸的是，这些函数中的每一个都会返回`unit`而不是排序后的数组，因此它们在管道或组合链中并不特别有效。

第一个就地排序函数`sortInPlace`使用默认比较机制对数组进行排序。下面的代码片段演示了如何对一组随机整数进行排序。

```
> **let r = System.Random()**
**let ints = Array.init 5 (fun _ -> r.Next(-100, 100));;**

val r : System.Random
val ints : int [] = [|-94; 20; 13; -99; 0|]

> **ints |> Array.sortInPlace;;**
val it : unit = ()
> **ints;;**
val it : int [] = [|-99; -94; 0; 13; 20|]
```

如果您需要更多的排序控制，您可以使用`sortInPlaceBy`或`sortInPlaceWith`函数。`sortInPlaceBy`函数让您提供一个转换函数，这个函数将在排序过程中使用。`sortInPlaceWith`函数接受一个比较函数，该函数返回一个整数，若小于零表示第一个值大于第二个值，若大于零表示第一个值小于第二个值，若等于零表示第一个和第二个值相等。

为了更好地理解这两种方法，考虑下面这个包含一些电影及其上映年份的元组数组。

```
let movies = [| ("The Terminator", "1984")
                ("Predator", "1987")
                ("Commando", "1985")
                ("Total Recall", "1990")
                ("Conan the Destroyer", "1984") |]
```

排序年份的最简单方法是通过`sortInPlaceBy`投影年份值，像这样：

```
> **movies |> Array.sortInPlaceBy (fun (_, y) -> y)**
**movies;;**

val it : (string * string) [] =
  [|("The Terminator", "1984"); ("Conan the Destroyer", "1984");
    ("Commando", "1985"); ("Predator", "1987"); ("Total Recall", "1990")|]
```

或者，我们可以直接使用`sortInPlaceWith`来比较两个元素：

```
> **movies |> Array.sortInPlaceWith (fun (_, y1) (_, y2) -> if y1 < y2 then -1**
                                                        **elif y1 > y2 then 1**
                                                        **else 0)**
**movies;;**

val it : (string * string) [] =
  [|("The Terminator", "1984"); ("Conan the Destroyer", "1984");
    ("Commando", "1985"); ("Predator", "1987"); ("Total Recall", "1990")|]
```

如您所见，`sortInPlaceBy`允许您根据特定元素底层类型的默认相等语义进行排序，而`sortInPlaceWith`则允许您为数组中的每个元素定义自己的相等语义。

### 多维数组

到目前为止，我们看到的所有数组都是一维数组。虽然也可以创建多维数组，但由于没有直接的语法支持，这稍微复杂一些。对于二维数组，您可以将一个序列的序列（通常是数组或列表）传递给`array2D`运算符。要创建超过二维的数组，您需要使用`Array3D.init`或`Array4D.init`函数。多维数组有模块（如`Array2D`和`Array3D`），这些模块包含`Array`模块中定义的专门子集。

### 注意

*F#支持的最大维度数是四。*

假设您想将前面章节中的电影表示为一个二维数组，而不是元组数组。您可以写类似以下的代码，将一个数组的数组传递给`array2D`运算符：

```
let movies = array2D [| [| "The Terminator"; "1984" |]
                        [| "Predator"; "1987" |]
                        [| "Commando"; "1985" |]
                        [| "The Running Man"; "1987" |]
                        [| "True Lies"; "1994" |]
                        [| "Last Action Hero"; "1993" |]
                        [| "Total Recall"; "1990" |]
                        [| "Conan the Barbarian"; "1982" |]
                        [| "Conan the Destroyer"; "1984" |]
                        [| "Hercules in New York"; "1969" |] |]
```

您可以使用熟悉的索引器语法访问二维数组中的任何值。例如，要获取*Commando*的上映年份，您可以写**`movies.[2, 1]`**，这将返回`1985`。然而，更有趣的是，您可以通过切片表达式进行更多操作。

切片表达式使得从源数组中创建包含子集的新数组变得非常容易。例如，您可以垂直切片`movies`数组，创建只包含电影名称或上映年份的新数组，像这样：

```
> **movies.[0..,0..0];;**
val it : string [,] = [["The Terminator"]
                       ["Predator"]
                       ["Commando"]
                       ["The Running Man"]
                       -- *snip* --]

> **movies.[0..,1..1];;**
val it : string [,] = [["1984"]
                       ["1987"]
                       ["1985"]
                       ["1987"]
                       -- *snip* --]
```

您还可以水平切片数组，创建只包含几行的新数组：

```
> **movies.[1..3,0..];;**
val it : string [,] = [["Predator"; "1987"]
                       ["Commando"; "1985"]
                       ["The Running Man"; "1987"]]
```

多维数组在数据具有良好的矩形形状时非常有用，但当哪怕只有一行的元素数量不同，它们就不适用了。考虑一下如果我们试图在二维`movies`数组中包含导演名会发生什么（为了简洁起见，这里我们只使用三个标题）。

```
> **let movies = array2D [| [| "The Terminator"; "1984"; "James Cameron" |]**
                        **[| "Predator"; "1987"; "John McTiernan" |]**
                        **[| "Commando"; "1985" |] |];;**
System.ArgumentException: The arrays have different lengths.
Parameter name: vals
-- *snip* --
Stopped due to error
```

当然，一种可能的解决方案是为缺少导演名的行提供一个空字符串作为第三个元素。或者，你可以使用一个锯齿数组。

### 锯齿数组

*锯齿数组*是数组的数组。与多维数组不同，锯齿数组不要求具有矩形结构。要转换前面的失败示例，我们只需要移除对`array2D`函数的调用。

```
> **let movies = [| [| "The Terminator"; "1984"; "James Cameron" |]**
                **[| "Predator"; "1987"; "John McTiernan" |]**
                **[| "Commando"; "1985" |] |];;**

val movies : string [] [] =
  [|[|"The Terminator"; "1984"; "James Cameron"|];
    [|"Predator"; "1987"; "John McTiernan"|]; [|"Commando"; "1985"|]|]
```

正如你可能预料到的，既然`movies`现在是一个锯齿数组，你需要使用不同的语法来访问每个元素。在使用锯齿数组时，你还需要编写更多的防御性代码，因为无法保证某个特定索引在任何给定的行中都是有效的。也就是说，你可以像这样从第二行获取导演的名字：

```
> **movies.[1].[2];;**
val it : string = "John McTiernan"
```

不管你怎么切片

F# 3.1 版本新增了一些数组切片的扩展功能，这些功能在这里没有涉及，但确实很有用。在 F# 3.0 中，数组切片要求切片的维度与源数组相同。而在 F# 3.1 中，这一限制已被取消，因此你可以从一个二维数组中创建一维切片，依此类推。

## 列表

*列表*在 F#开发中广泛使用。当.NET 开发人员讨论列表时，他们通常指的是泛型`List<'T>`类。尽管在 F#中使用泛型列表是可能的（有时甚至是可取的），但该语言定义了另一个基于单链表的不可变构造。在 F#中，使用列表语法创建的列表会编译为`Microsoft.FSharp.Collections`命名空间中的`FSharpList<'T>`类的实例，这也是我们在本节中将要讨论的列表类型。

除了`List<'T>`和`FSharpList<'T>`都是泛型序列类型（它们都实现了`IEnumerable<'T>`接口）外，它们几乎没有什么共同点，不能互换使用。在多语言解决方案中工作时，你需要小心不要混用这两种列表类型。

### 注意

*你可以通过打开`System.Collections.Generic`命名空间或通过内置的`ResizeArray<'T>`类型缩写，直接使用泛型`List<'T>`类。*

### 创建列表

在 F#中创建列表与创建数组非常相似，因此我不会花太多时间解释各种形式。创建数组和列表的唯一语法区别是括号样式。要创建一个新列表，你需要将分号分隔的值、范围表达式或列表序列表达式放在方括号（`[]`）中，像这样：

```
> **let names = [ "Rose"; "Martha"; "Donna"; "Amy"; "Clara" ];;**

val names : string list = ["Rose"; "Martha"; "Donna"; "Amy"; "Clara"]

> **let numbers = [ 1..11 ];;**

val numbers : int list = [1; 2; 3; 4; 5; 6; 7; 8; 9; 10; 11]
```

要创建一个空列表，你可以使用`List.empty`或一对空的括号。

### 使用列表

尽管在操作 F#列表和`List<'T>`时有一些相似之处，但它们主要是语法上的，涉及访问单个已知元素。除此之外，F#列表非常独特，特别是它们的头尾结构，非常适合函数式编程，尤其是递归技术。

#### 访问元素

当你想获取某个特定位置的元素时，你可以使用熟悉的索引语法，就像操作数组一样。或者，你也可以使用`List.nth`来获得相同的结果：

```
> **List.nth [ 'A'..'Z' ] 3;;**
val it : char = 'D'
```

比通过索引访问特定元素更有趣（且通常更有用）的是列表的*头*和*尾*。列表的头是它的第一个元素，而它的尾是除了头以外的所有元素。你可以通过`Head`或`Tail`属性，或`List.head`和`List.tail`模块函数获取列表的头和尾。以下是使用模块函数的示例：

```
> **let names = [ "Rose"; "Martha"; "Donna"; "Amy"; "Clara" ];;**

val names : string list = ["Rose"; "Martha"; "Donna"; "Amy"; "Clara"]

> **List.head names;;**
val it : string = "Rose"
> **List.tail names;;**
val it : string list = ["Martha"; "Donna"; "Amy"; "Clara"]
```

### 注意

*模式匹配是获取头部和尾部的另一种方式，但我们将把这个讨论留到第七章。*

为什么你只想获取第一个元素或其他所有元素？递归。如果你必须使用索引遍历列表，你需要同时跟踪列表和当前位置。通过将列表分成头和尾部分，你可以在操作头部分后继续递归遍历尾部分。

考虑这个函数，它返回一个布尔值，指示一个列表是否包含特定的值（类似于`List.exists`模块函数）。

```
let rec contains fn l =
  if l = [] then false
  else fn(List.head l) || contains fn (List.tail l)
```

`contains`函数接受一个用于测试元素的函数和一个要扫描的列表。`contains`首先检查提供的列表是否为空。如果列表为空，`contains`会立即返回`false`；否则，它会使用提供的函数测试列表的头部，或者递归地调用`contains`，传入该函数和列表的尾部。

现在让我们从一个空列表开始测试几个值：

```
> **[] |> contains (fun n -> n = "Rose");;**
val it : bool = false
```

你可以看到，当列表为空时，`contains`正确地返回了`false`，但对于一个有元素的列表呢？

```
> **let names = [ "Rose"; "Martha"; "Donna"; "Amy"; "Clara" ];;**

val names : string list = ["Rose"; "Martha"; "Donna"; "Amy"; "Clara"]

> **names |> contains (fun n -> n = "Amy");;**
val it : bool = true
> **names |> contains (fun n -> n = "Rory");;**
val it : bool = false
```

`contains`函数递归地遍历列表，使用提供的函数检查每个元素，如果元素不匹配，就将尾部传递给`contains`。

#### 合并列表

虽然 F#列表是不可变的，但我们仍然可以从现有列表构造新列表。F#提供了两种主要机制：`cons`运算符（`::`）和通过`@`运算符进行的列表连接。

`cons`运算符（之所以命名为`cons`，是因为它*构造*了一个新列表）本质上是将一个元素添加到现有列表的前面，如下所示：

```
> **let names = [ "Rose"; "Martha"; "Donna"; "Amy"; "Clara" ]**
**let newNames = "Ace" :: names;;**

val names : string list = ["Rose"; "Martha"; "Donna"; "Amy"; "Clara"]
val newNames : string list =
  ["Ace"; "Rose"; "Martha"; "Donna"; "Amy"; "Clara"]
```

`cons` 操作符不会对现有列表进行任何更改。相反，它只是创建一个新的列表，头部设置为新值，尾部设置为现有列表。`cons` 操作符只能将单个项添加到列表中，但由于它位于列表的开头，因此是一个快速操作。如果你想合并两个列表，你需要使用列表连接。

要连接两个列表，你可以使用列表连接操作符（`@`）或 `List.append` 模块函数，如下所示：

```
> **let classicNames = [ "Susan"; "Barbara"; "Sarah Jane" ]**
**let modernNames = [ "Rose"; "Martha"; "Donna"; "Amy"; "Clara" ];;**

val classicNames : string list = ["Susan"; "Barbara"; "Sarah Jane"]
val modernNames : string list = ["Rose"; "Martha"; "Donna"; "Amy"; "Clara"]

> **classicNames @ modernNames;;**
val it : string list =
  ["Susan"; "Barbara"; "Sarah Jane"; "Rose"; "Martha"; "Donna"; "Amy"; "Clara"]
> **List.append classicNames modernNames;;**
val it : string list =
  ["Susan"; "Barbara"; "Sarah Jane"; "Rose"; "Martha"; "Donna"; "Amy"; "Clara"]
```

使用连接操作符创建的列表与使用 `List.append` 创建的列表没有区别。从内部实现来看，`List.append` 封装了追加操作符，因此它们在功能上是等效的。

要同时合并多个列表，你可以像这样将一系列列表传递给 `List.concat`：

```
> **List.concat [[ "Susan"; "Sarah Jane" ]**
             **[ "Rose"; "Martha" ]**
             **["Donna"; "Amy"; "Clara"]];;**
val it : string list =
  ["Susan"; "Sarah Jane"; "Rose"; "Martha"; "Donna"; "Amy"; "Clara"]
```

现在，最初的三个独立列表已经合并成一个包含每个项的单一列表。

## 集合

在 F# 中，*集合* 是一个不可变的唯一值集合，其顺序不被保留。F# 的集合与数学集合密切相关（可以参考维恩图），并提供许多有助于比较集合的操作。

### 创建集合

创建集合时没有像特殊括号格式这样的语法糖，因此，如果你想使用集合，你需要依赖类型构造器或一些 `Set` 模块函数（如 `Set.ofList`，它可以从 F# 列表创建集合）。例如，要创建一个包含字母表字母的集合，你可以这样写：

```
> **let alphabet = [ 'A'..'Z' ] |> Set.ofList;;**

val alphabet : Set<char> =
  set ['A'; 'B'; 'C'; 'D'; 'E'; 'F'; 'G'; 'H'; 'I'; ...]
```

`Set<'T>` 类定义了添加和移除集合中值的方法，但由于 F# 集合是不可变的，这两个方法都会返回新的集合，并保持原集合不变。`Add` 方法对于从空集合填充新集合非常有用，例如：

```
> **let vowels = Set.empty.Add('A').Add('E').Add('I').Add('O').Add('U');;**

val vowels : Set<char> = set ['A'; 'E'; 'I'; 'O'; 'U']
```

当然，以这种方式创建集合比 F# 中典型的做法更加面向对象。

### 集合操作

因为集合与数学中的集合关系密切，`Set` 模块提供了多个函数，用于执行各种集合操作，如查找并集、交集和差集，甚至可以确定两个集合是否作为子集或超集相关联。

#### 并集

要找到两个集合的并集——即包含在第一个或第二个集合中的那些元素——你可以使用如下的 `Set.union` 函数：

```
> **let set1 = [ 1..5 ] |> Set.ofList**
**let set2 = [ 3..7 ] |> Set.ofList**
**Set.union set1 set2;;**

val set1 : Set<int> = set [1; 2; 3; 4; 5]
val set2 : Set<int> = set [3; 4; 5; 6; 7]
val it : Set<int> = set [1; 2; 3; 4; 5; 6; 7]
```

这里，`set1` 包含整数一到五，而 `set2` 包含整数三到七。由于两个集合的并集包含在任一集合中找到的每个不同的值，`set1` 和 `set2` 的并集是从一到七的整数范围。

`Set<'T>` 类还定义了一个自定义的 `+` 操作符，可以用来找到两个集合的并集：

```
> **set1 + set2;;**
val it : Set<int> = set [1; 2; 3; 4; 5; 6; 7]
```

#### 交集

`Set.intersect` 函数返回一个新集合，仅包含在两个集合中都存在的元素。例如，如果你有一个包含从一到五的元素的集合，另一个集合包含从三到七的元素，你可以这样找到交集：

```
> **let set1 = [ 1..5 ] |> Set.ofList**
**let set2 = [ 3..7 ] |> Set.ofList**
**Set.intersect set1 set2;;**

val set1 : Set<int> = set [1; 2; 3; 4; 5]
val set2 : Set<int> = set [3; 4; 5; 6; 7]
val it : Set<int> = set [3; 4; 5]
```

结果交集集合只包含`set1`和`set2`中共有的三个值——在本例中为 3、4 和 5。

#### 区别

虽然交集包含两个集合共有的所有元素，但差集包含仅在第一个集合中找到的元素。你可以使用`Set.difference`函数来找到两个集合之间的差异。

```
> **let set1 = [ 1..5 ] |> Set.ofList**
**let set2 = [ 3..7 ] |> Set.ofList**
**Set.difference set1 set2;;**

val set1 : Set<int> = set [1; 2; 3; 4; 5]
val set2 : Set<int> = set [3; 4; 5; 6; 7]
val it : Set<int> = set [1; 2]
```

这里，第一个集合包含第二个集合中没有的两个元素，`1`和`2`；因此，差集只包含这些值。

就像交集一样，`Set<'T>`类定义了一个自定义的`–`运算符，该运算符返回一个包含两个集合差异的集合。

```
> **set1 - set2;;**
val it : Set<int> = set [1; 2]
```

#### 子集和超集

`Set`模块通过四个函数使我们容易判断两个集合是否存在子集或超集关系：`isSubset`、`isProperSubset`、`isSuperset`和`isProperSuperset`。基本子集/超集与真正子集/超集之间的区别在于，真正的子集/超集需要至少有一个在对方集合中不存在的额外元素。以下集合可以说明这一点：

```
> **let set1 = [ 1..5 ] |> Set.ofList**
**let set2 = [ 1..5 ] |> Set.ofList;;**

val set1 : Set<int> = set [1; 2; 3; 4; 5]
val set2 : Set<int> = set [1; 2; 3; 4; 5]
```

因为`set1`和`set2`包含相同的值，所以可以认为`set1`是`set2`的超集。相反，`set2`可以被认为是`set1`的子集。然而，基于同样的原因，`set2`不能是`set1`的真正子集，正如以下代码片段所示。

```
> **Set.isSuperset set1 set2;;**
val it : bool = true
> **Set.isProperSuperset set1 set2;;**
val it : bool = false
> **Set.isSubset set2 set1;;**
val it : bool = true
> **Set.isProperSubset set2 set1;;**
val it : bool = false
```

要使`set2`成为`set1`的一个真正子集，我们需要重新定义`set1`，使其至少包含一个额外的值。

```
> **let set1 = [ 0..5 ] |> Set.ofList;;**

val set1 : Set<int> = set [0; 1; 2; 3; 4; 5]
```

现在，如果我们再次测试子集和超集，我们应该会看到`set2`既是`set1`的子集，也是其真正子集。

```
> **Set.isSuperset set1 set2;;**
val it : bool = true
> **Set.isProperSuperset set1 set2;;**
val it : bool = true
> **Set.isSubset set2 set1;;**
val it : bool = true
> **Set.isProperSubset set2 set1;;**
val it : bool = true
```

## 映射

`Map`类型表示一个无序的不可变字典（键到值的映射），并提供了与通用`Dictionary<'TKey, 'TValue>`类相同的许多功能。

### 注意

*尽管`Map<'Key`, `'Value>`类和相关的`Map`模块提供了添加和移除条目的方法，但作为不可变构造体，只有在底层条目不会改变时，映射才有意义。从映射中添加和删除条目需要创建一个新的映射实例，并将数据从源实例复制过来，因此比修改可变字典要慢得多。*

### 创建映射

与集合一样，F#不提供直接的语法支持来创建映射，因此也需要使用类型构造器或`Map`模块函数来创建它们。无论你选择哪种方式，映射总是基于一系列包含键和值的元组。在这里，我们将一个包含各州及其相应首府的列表传递给类型构造器：

```
> **let stateCapitals =**
  **Map [("Indiana", "Indianapolis")**
       **("Michigan", "Lansing")**
       **("Ohio", "Columbus")**
       **("Kentucky", "Frankfort")**
       **("Illinois", "Springfield")];;**

val stateCapitals : Map<string, string> =
  map
    [("Illinois", "Springfield"); ("Indiana", "Indianapolis");
     ("Kentucky", "Frankfort"); ("Michigan", "Lansing"); ("Ohio", "Columbus")]
```

### 使用映射

由于映射类似于不可变字典，与它们交互的方式类似于`Dictionary<'TKey, 'TValue>`。

#### 查找值

与通用字典类似，`Map`类型提供了一个索引属性，通过已知键访问值。例如，使用`stateCapitals`映射，我们可以这样查找印第安纳州的首府：

```
> **stateCapitals.["Indiana"];;**
val it : string = "Indianapolis"
```

`Map.find`函数让我们通过函数式的方式做同样的事情。

```
> **stateCapitals |> Map.find "Indiana";;**
val it : string = "Indianapolis"
```

前面两种方法的最大问题是，当映射中没有该键时，它们会抛出`KeyNotFoundException`。为了避免这种异常，你可以使用`Map.containsKey`函数来检查映射中是否包含某个特定键。如果你想测试`stateCapitals`是否包含华盛顿，可以写出如下代码：

```
> **stateCapitals |> Map.containsKey "Washington";;**
val it : bool = false
```

最后，如果你更倾向于通过单次操作来测试键并获取映射的值，你可以使用`Map.tryFind`函数，它返回一个`option`，指示是否找到键以及相关的值，如下所示：

```
> **stateCapitals |> Map.tryFind "Washington";;**
val it : string option = None
> **stateCapitals |> Map.tryFind "Indiana";;**
val it : string option = Some "Indianapolis"
```

#### 查找键

有时，你可能需要根据映射的值查找键。`Map`模块提供了两个函数来实现这一点：`findKey`和`tryFindKey`。就像它们的值查找对应函数一样，`findKey`和`tryFindKey`的区别在于，当无法找到符合条件的值时，`findKey`会抛出`KeyNotFoundException`，而`tryFindKey`则不会。

要查找键，你需要传递一个接受键及其映射值的函数，并返回一个布尔值，指示值是否符合你的标准。例如，要通过首都查找一个州，可以使用`Map.tryFindKey`，你可以写出如下代码：

```
> **stateCapitals |> Map.tryFindKey (fun k v -> v = "Indianapolis");;**
val it : string option = Some "Indiana"
> **stateCapitals |> Map.tryFindKey (fun k v -> v = "Olympia");;**
val it : string option = None
```

如你所见，`tryFindKey`返回一个`option`，因此你需要根据`Some`和`None`进行相应的测试。

## 在集合类型之间进行转换

有时你会有一个集合类型的实例，但你实际上需要一个不同的类型。例如，你可能正在处理一个 F#列表，但想要应用一个仅适用于数组的函数。每个集合模块都包含几个函数，可以轻松地在许多其他集合类型之间进行转换。

在每个模块中，转换函数的命名是根据转换方向和目标类型来命名的。例如，要将一个序列转换为数组，你可以将序列传递给`Seq.toArray`或`Array.ofSeq`，像这样：

```
> **seq { 1..10 } |> Seq.toArray;;**
val it : int [] = [|1; 2; 3; 4; 5; 6; 7; 8; 9; 10|]
> **seq { 1..10 } |> Array.ofSeq;;**
val it : int [] = [|1; 2; 3; 4; 5; 6; 7; 8; 9; 10|]
```

类似地，要将一个列表转换为序列，你可以将列表传递给`List.toSeq`或`Seq.ofList`。`Set`和`Map`模块也允许你根据相同的约定，在序列、数组和映射之间进行转换。

尽管大多数转换函数会创建一个新的集合，但其中一些通过类型转换工作。例如，`Seq.ofList`只是将源列表转换为`seq<'t>`（记住，`FSharpList<'T>`实现了`IEnumerable<'T>`，所以这是一个有效的转换），而`List.ofArray`则创建一个新的数组，并用列表的值填充它。如果有任何问题关于结果集合是类型转换还是新对象，你可以使用静态方法`obj.ReferenceEquals`来检查，如下所示：

```
> **let l = [ 1..10 ]**
**obj.ReferenceEquals(l, Seq.ofList l);;**

val l : int list = [1; 2; 3; 4; 5; 6; 7; 8; 9; 10]
val it : bool = ① true

> **let a = [| 1..10 |]**
**obj.ReferenceEquals(a, List.ofArray a);;**

val a : int [] = [|1; 2; 3; 4; 5; 6; 7; 8; 9; 10|]
val it : bool = ② false
```

上面的代码片段展示了调用`Seq.ofList`和`List.ofArray`的结果。你可以看到，① `Seq.ofList`返回相同的对象，而`List.ofArray`②返回一个新的对象。

## 总结

处理数据集合是几乎每个复杂应用程序都必须做的事情。F#让你能够使用所有传统的.NET 集合，如数组和泛型列表，同时还添加了其他几种类型，比如 F#列表、集合和映射，这些更适合函数式编程。

在许多方面，使用 F#处理数据集合比传统.NET 开发更加简化，因为像序列表达式、范围表达式和切片表达式这样的语言特性使得创建集合变得更容易，同时也能更方便地访问单个元素。

最后，像`Seq`、`Array`和`List`这样的各种集合模块提供了一种便捷的机制，可以在各自的集合类型上执行许多常见任务。
