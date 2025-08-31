## 第七章 模式，模式，处处是模式

模式匹配是 F# 最强大的特性之一。模式在语言中根深蒂固，许多你已经见过的结构都会使用模式，例如 `let` 绑定、`try...with` 表达式和 lambda 表达式。在本章中，你将学习匹配表达式、预定义模式类型以及如何使用活动模式创建你自己的模式。

## 匹配表达式

尽管 F# 允许通过 if 表达式进行命令式风格的分支，但它们可能难以维护，尤其是当条件逻辑的复杂性增加时。匹配表达式是 F# 主要的分支机制。

从表面上看，许多匹配表达式类似于 C# 的 `switch` 或 Visual Basic 的 `Select Case` 语句，但它们要强大得多。例如，`switch` 和 `Select Case` 只对常量值进行操作，而匹配表达式根据哪个模式与输入匹配来选择要评估的表达式。在最基本的形式中，匹配表达式的结构如下：

```
match ①*test-expression* with
  | ②*pattern1* -> ③*result-expression1*
  | ④*pattern2* -> ⑤*result-expression2*
  | ...
```

在上面的语法中，① 处的表达式首先被求值，并依次与表达式体中的每个模式进行比较，直到找到匹配项。例如，如果结果满足 ② 处的模式，③ 处的表达式将被求值。否则，④ 处的模式将被测试，如果匹配，⑤ 处的表达式将被求值，依此类推。因为匹配表达式也会返回一个值，所以每个结果表达式必须是相同类型的。

模式是按顺序匹配的这一事实对如何构建代码有影响；你必须组织你的匹配表达式，使得模式从最具体到最一般排列。如果更一般的模式排在更具体的模式前面，以至于阻止了后续模式的评估，编译器将会对每个受影响的模式发出警告。

匹配表达式可以与各种数据类型一起使用，包括（但不限于）数字、字符串、元组和记录。例如，下面是一个简单的匹配表达式，它与一个带有区分联合类型的函数一起工作：

```
let testOption opt =
  match opt with
  | Some(v) -> printfn "Some: %i" v
  | None -> printfn "None"
```

在这个代码片段中，`opt` 被推断为 `int option` 类型，匹配表达式包含了 `Some` 和 `None` 两种情况的模式。当匹配表达式被求值时，它首先测试 `opt` 是否与 `Some` 匹配。如果匹配，模式将 `Some` 中的值绑定到 `v`，然后当结果表达式被求值时，`v` 被打印出来。同样地，当匹配到 `None` 时，结果表达式简单地打印出 `"None"`。

### 守卫子句

除了将不同的值与模式进行匹配外，你还可以通过 *守卫子句* 进一步完善每种情况，这些子句允许你指定额外的条件，只有满足这些条件才能满足某个情况。例如，你可以使用守卫子句（通过插入 `when` 后跟条件）来区分正数和负数，如下所示：

```
let testNumber value =
  match value with
  | ①v when v < 0 -> printfn "%i is negative" v
  | ②v when v > 0 -> printfn "%i is positive" v
  | _ -> printfn "zero"
```

在这个例子中，我们有两个具有相同模式但不同守卫子句的情况。尽管任何整数都会匹配这三种模式中的任何一种，但模式①和②上的守卫子句会导致匹配失败，除非捕获的值符合它们的标准。

你可以使用布尔运算符将多个守卫子句结合起来，以实现更复杂的匹配逻辑。例如，你可以构造一个仅匹配正整数的偶数的情况，如下所示：

```
let testNumber value =
  match value with
  | v when v > 0 && v % 2 = 0 -> printfn "%i is positive and even" v
  | v -> printfn "%i is zero, negative, or odd" v
```

### 模式匹配函数

有一种替代的匹配表达式语法，称为*模式匹配函数*。使用模式匹配函数语法，`match...with`部分的匹配表达式被`function`替代，如下所示：

```
> **let testOption =**
  **function**
  | **Some(v) -> printfn "Some: %i" v**
  | **None -> printfn "None";;**

val testOption : _arg1:int option -> unit
```

从输出中的签名可以看到，通过使用模式匹配函数语法，我们将`testOption`绑定为一个接受`int option`（生成的名称为`_arg1`）并返回`unit`的函数。以这种方式使用`function`关键字是创建模式匹配 lambda 表达式的便捷方式，并且在功能上等价于编写：

```
fun x ->
  match x with
  | Some(v) -> printfn "Some: %i" v
  | None -> printfn "None";;
```

由于模式匹配函数只是 lambda 表达式的简化版，将匹配表达式传递给高阶函数是非常简单的。假设你想从一个包含可选整数的列表中过滤掉所有`None`值，你可以考虑将一个模式匹配函数传递给`List.filter`函数，如下所示：

```
[ Some 10; None; Some 4; None; Some 0; Some 7 ]
|> List.filter (function | Some(_) -> true
                         | None -> false)
```

当执行`filter`函数时，它将对源列表中的每一项调用模式匹配函数，当项为`Some(_)`时返回`true`，当项为`None`时返回`false`。因此，由`filter`创建的列表将只包含`Some 10`、`Some 4`、`Some 0`和`Some 7`。

## 穷尽匹配

当一个匹配表达式包含的模式能够涵盖测试表达式的所有可能结果时，称其为*穷尽*或*覆盖*的。当存在一个未被模式覆盖的值时，编译器会发出警告。考虑当我们匹配一个整数，但只覆盖了少数几个情况时会发生什么。

```
> **let numberToString =**
  **function**
  **| 0 -> "zero"**
  **| 1 -> "one"**
  **| 2 -> "two"**
  **| 3 -> "three";;**

    function
  --^^^^^^^^

stdin(4,3): warning FS0025: Incomplete pattern matches on this expression. For
example, the value '4' may indicate a case not covered by the pattern(s).

val numberToString : _arg1:int -> string
```

在这里，你可以看到如果整数是 0、1、2 或 3 之外的任何值，它将永远不会匹配。编译器甚至提供了一个可能未被覆盖的值——在这个例子中是四。如果`numberToString`被调用时传入一个未被覆盖的值，调用将失败并抛出`MatchFailureException`：

```
> **numberToString 4;;**
Microsoft.FSharp.Core.MatchFailureException: The match cases were incomplete
   at FSI_0025.numberToString(Int32 _arg1)
   at <StartupCode$FSI_0026>.$FSI_0026.main@()
Stopped due to error
```

为了解决这个问题，你可以添加更多模式，尝试匹配所有可能的值，但许多时候（比如整数）匹配所有可能的值是不可行的。其他时候，你可能只关心几个特定的情况。在这两种情况下，你都可以使用匹配任何值的模式：变量模式或通配符模式。

### 变量模式

*变量模式*通过标识符表示，并在你希望匹配任何值并将该值绑定到一个名称时使用。通过变量模式定义的任何名称都可以在该情况的守卫子句和结果表达式中使用。例如，为了使 `numberToString` 函数更加完备，你可以像这样修改函数，加入一个变量模式：

```
let numberToString =
  function
  | 0 -> "zero"
  | 1 -> "one"
  | 2 -> "two"
  | 3 -> "three"
① | n -> sprintf "%O" n
```

当你在 ① 处包含一个变量模式时，除了 0、1、2 或 3 之外的任何值都会被绑定到 `n`，并简单地转换为字符串。

变量模式中定义的标识符应该以小写字母开头，以区分它与标识符模式。现在，调用 `numberToString` 并传入 `4` 将不再出现错误，如下所示：

```
> **numberToString 4;;**
val it : string = "4"
```

### 通配符模式

*通配符模式*，用单个下划线字符（`_`）表示，工作原理与变量模式相同，只是它丢弃匹配的值，而不是将其绑定到名称。

以下是经过修改的 `numberToString` 实现，加入了通配符模式。注意，由于匹配的值会被丢弃，我们需要返回一个通用的字符串，而不是基于匹配值的字符串。

```
let numberToString =
  function
  | 0 -> "zero"
  | 1 -> "one"
  | 2 -> "two"
  | 3 -> "three"
  | _ -> "unknown"
```

## 匹配常量值

*常量模式*由硬编码的数字、字符、字符串和枚举值组成。你已经看到了一些常量模式的例子，但为了重申，接下来 `numberToString` 函数中的前四个情况都是常量模式。

```
let numberToString =
  function
  | 0 -> "zero"
  | 1 -> "one"
  | 2 -> "two"
  | 3 -> "three"
  | _ -> "..."
```

在这里，数字 0 到 3 被明确匹配并返回数字对应的单词。所有其他值都进入通配符情况。

## 标识符模式

当一个模式由多个字符组成并且以大写字母开头时，编译器会尝试将其解析为名称。这被称为 *标识符模式*，通常指代判别联合情况、带有 `LiteralAttribute` 的标识符或异常名称（如在 `try...with` 块中所见）。

### 匹配联合情况

当标识符是一个判别联合情况时，该模式被称为 *联合情况模式*。联合情况模式必须为该情况关联的每个数据项包括一个通配符或标识符。如果该情况没有任何关联数据，则情况标签可以单独出现。

考虑以下定义了一些形状的判别联合：

```
type Shape =
| Circle of float
| Rectangle of float * float
| Triangle of float * float * float
```

从这个定义开始，定义一个函数来使用匹配表达式计算任何包含形状的周长就变得很简单了。以下是一个可能的实现：

```
let getPerimeter =
  function
  | Circle(r) -> 2.0 * System.Math.PI * r
  | Rectangle(w, h) -> 2.0 * (w + h)
  | Triangle(l1, l2, l3) -> l1 + l2 + l3
```

如你所见，由判别联合定义的每个形状都被涵盖，且每个情况中的数据项被提取为有意义的名称，例如圆的半径 `r`，矩形的宽度 `w` 和高度 `h`。

### 匹配字面量

当编译器遇到一个使用 `LiteralAttribute` 定义的标识符作为情况时，它被称为 *字面量模式*，但它的处理方式和常量模式一样。

这是修改后的 `numberToString` 函数，使用了一些字面量模式代替常量模式：

```
[<LiteralAttribute>]
let Zero = 0
[<LiteralAttribute>]
let One = 1
[<LiteralAttribute>]
let Two = 2
[<LiteralAttribute>]
let Three = 3

let numberToString =
  function
  | Zero -> "zero"
  | One -> "one"
  | Two -> "two"
  | Three -> "three"
  | _ -> "unknown"
```

## 匹配空值

当对包含 `null` 为有效值的类型进行模式匹配时，你通常会想包括一个*空值模式*，以尽可能隔离所有的 `null` 值。空值模式通过 `null` 关键字表示。

考虑这个 `matchString` 模式匹配函数：

```
> **let matchString =**
  **function**
  **| "" -> None**
  **| v -> Some(v.ToString());;**

val matchString : _arg1:string -> string option
```

`matchString` 函数包含两种情况：一个用于空字符串的常量模式和一个用于其他所有内容的变量模式。编译器很高兴为我们创建这个函数，并且没有警告我们关于不完整的模式匹配，但这里有一个潜在的严重问题：`null` 是字符串的有效值，但变量模式匹配任何值，包括 `null`！如果一个 `null` 字符串传递给 `matchString`，当对 `v` 调用 `ToString` 方法时，`NullReferenceException` 将被抛出，因为变量模式匹配 `null`，并因此将 `v` 设置为 `null`，如下面所示：

```
> **matchString null;;**
System.NullReferenceException: Object reference not set to an instance of an object.
   at FSI_0070.matchString(String _arg1) in C:\Users\Dave\AppData\Local\Temp\~vsE434.fsx:line 68
   at <StartupCode$FSI_0071>.$FSI_0071.main@()
Stopped due to error
```

在变量模式之前添加空值模式将确保 `null` 值不会泄露到应用程序的其他部分。按照惯例，空值模式通常列在最前面，因此这里的做法是将 `null` 和空字符串模式与“或”模式结合使用：

```
let matchString =
  function
  **| null**
  | "" -> None
  | v -> Some(v.ToString())
```

## 匹配元组

你可以使用*元组模式*匹配并解构元组到其组成元素。例如，一个表示二维点的元组可以通过 `let` 绑定中的元组模式解构为单独的 x 和 y 坐标，像这样：

```
let point = 10, 20
let x, y = point
```

在这个示例中，值 `10` 和 `20` 从 `point` 中提取并分别绑定到 `x` 和 `y` 标识符。

类似地，你可以在匹配表达式中使用多个元组模式，基于元组值执行分支操作。以点的主题为例，假设要判断某个点是否位于原点或沿轴线，你可以写类似以下代码：

```
let locatePoint p =
  match p with
  | ① (0, 0) -> sprintf "%A is at the origin" p
  | ② (_, 0) -> sprintf "%A is on the x-axis" p
  | ③ (0, _) -> sprintf "%A is on the y-axis" p
  | ④ (x, y) -> sprintf "Point (%i, %i)" x y
```

`locatePoint` 函数不仅突出了使用多个元组模式，还展示了如何将多种模式类型结合起来形成更复杂的分支逻辑。例如，① 在元组模式中使用了两个常量模式，而 ② 和 ③ 分别在元组模式中使用了一个常量模式和一个通配符模式。最后，④ 在元组模式中使用了两个变量模式。

请记住，元组模式中的项数必须与元组本身的项数相匹配。例如，尝试将一个包含两项的元组模式与一个包含三项的元组进行匹配将导致编译错误，因为它们的基础类型不兼容。

## 匹配记录

记录类型可以通过记录模式参与模式匹配。使用*记录模式*，可以匹配并解构单个记录实例，提取出其各个值。

考虑以下基于典型美国姓名的记录类型定义：

```
type Name = { First : string; Middle : string option; Last : string }
```

在这种记录类型中，名字和姓氏都是必填的，但中间名是可选的。你可以使用匹配表达式根据是否指定中间名来格式化名字，如下所示：

```
let formatName =
  function
  | { First = f; Middle = Some(m); Last = l } -> sprintf "%s, %s %s" l f m
  | { First = f; Middle = None; Last = l } -> sprintf "%s, %s" l f
```

在这里，两个模式分别将名字和姓氏绑定到标识符 `f` 和 `l`。更有趣的是，模式如何将中间名与 `Some(m)` 和 `None` 的联合情况进行匹配。当匹配表达式与包含中间名的 `Name` 进行评估时，中间名将绑定到 `m`。否则，匹配失败，`None` 情况会被评估。

`formatName` 函数中的模式从记录中提取每个值，但记录模式也可以作用于标签的子集。例如，如果你只想确定一个名字是否包含中间名，你可以构造一个像下面这样的匹配表达式：

```
let hasMiddleName =
  function
  | { Middle = Some(_) } -> true
  | { Middle = None } -> false
```

编译器通常可以自动解析模式是针对哪种记录类型构造的，但如果无法确定，你可以像下面这样指定类型名称：

```
let hasMiddleName =
  function
  | { **Name.**Middle = Some(_) } -> true
  | { **Name.**Middle = None } -> false
```

通过像这样限定模式，通常只有在存在多个具有冲突定义的记录类型时才需要。

## 匹配集合

模式匹配不仅限于单一值或类似元组和记录这样的结构化数据。F# 还包括几种模式，用于匹配一维数组和列表。如果你想匹配另一种集合类型，通常需要通过 `List.ofSeq`、`Array.ofSeq` 或类似的机制将集合转换为列表或数组。

### 数组模式

*数组模式*与数组定义非常相似，允许你匹配具有特定元素个数的数组。例如，你可以使用数组模式来确定数组的长度，如下所示：

```
let getLength =
  function
  | null -> 0
  | [| |] -> 0
  | [| _ |] -> 1
  | [| _; _; |] -> 2
  | [| _; _; _ |] -> 3
  | a -> a |> Array.length
```

忽略数组长度的计算通常直接通过 `Array.length` 属性检查，而不通过这种人为的模式匹配例子，`getLength` 函数展示了数组模式如何匹配固定大小数组中的单个元素。

### 列表模式

*列表模式*类似于数组模式，只不过它们看起来像并且作用于 F# 列表。这里是 `getLength` 函数的修改版，已调整为与 F# 列表而非数组配合使用。

```
let getLength =
  function
  | [ ] -> 0
  | [ _ ] -> 1
  | [ _; _; ] -> 2
  | [ _; _; _ ] -> 3
  | lst -> lst |> List.length
```

请注意，没有 `null` 情况，因为 `null` 不是 F# 列表的有效值。

### Cons 模式

另一种匹配 F# 列表的方法是使用 *Cons 模式*。在模式匹配中，cons 操作符 (`::`) 是反向工作的；它不是将元素添加到列表前面，而是将列表的头部和尾部分开。这使得你能够递归地匹配具有任意数量元素的列表。

与我们的主题一致，下面是如何使用 Cons 模式通过模式匹配来查找集合的长度：

```
let getLength n =
  ① let rec len c l =
    match l with
    | ② [] -> c
    | ③ _ :: t -> len (c + 1) t
  len 0 n
```

这个版本的`getLength`函数与 F#列表的内部`length`属性实现非常相似。它定义了`len` ①，一个内部函数，递归地匹配空模式 ② 或 Cons 模式 ③。匹配到空列表时，`len`返回提供的计数值（`c`）；否则，它会递归调用，递增计数并传递尾部。`getLength`中的 Cons 模式使用通配符模式来匹配头值，因为在后续操作中不需要它。

## 按类型匹配

F#有两种方式可以匹配特定的数据类型：类型注解模式和动态类型测试模式。

### 类型注解模式

*类型注解模式*允许你指定匹配值的类型。它们在模式匹配函数中特别有用，在这些函数中，编译器需要一些额外的帮助来确定函数隐式参数的预期类型。例如，以下函数用于检查一个字符串是否以大写字母开头：

```
// Does not compile
let startsWithUpperCase =
  function
  | ① s when ② s.Length > 0 && s.[0] = System.Char.ToUpper s.[0] -> true
  | _ -> false
```

然而，按目前的写法，`startsWithUpperCase`函数无法编译。它会失败并显示以下错误：

```
~vsD607.fsx(83,12): error FS0072: Lookup on object of indeterminate type based
on information prior to this program point. A type annotation may be needed
prior to this program point to constrain the type of the object. This may
allow the lookup to be resolved.
```

该编译失败的原因是守卫条件在 ② 依赖于字符串属性，但这些属性不可用，因为编译器已经自动泛化了函数的隐式参数。为了解决这个问题，我们可以修改函数，使其显式地使用字符串参数，或者我们可以在 ① 的模式中包括类型注解，像这样（注意括号是必须的）：

```
let startsWithUpperCase =
  function
  | **(s : string)** when s.Length > 0 && s.[0] = System.Char.ToUpper s.[0] ->
    true
  | _ -> false
```

使用类型注解后，参数不再自动泛化，从而使得字符串的属性可以在守卫条件中使用。

### 动态类型测试模式

*动态类型测试模式*在某种程度上是类型注解模式的对立面。类型注解模式要求每个案例都匹配相同的数据类型，而动态类型测试模式在匹配的值是特定类型的实例时满足条件；也就是说，如果你注解一个模式以匹配字符串，每个案例都必须匹配字符串。因此，动态类型测试模式非常适合匹配类型层次结构。例如，你可能会匹配一个接口实例，但使用动态类型测试模式为特定的实现提供不同的逻辑。动态类型测试模式类似于动态类型转换操作符（`:?>`），除了省略了`>`符号。

以下`detectColorSpace`函数展示了如何通过匹配三种记录类型来使用动态类型测试模式。如果没有类型匹配，该函数会抛出异常。

```
type RgbColor = { R : int; G : int; B : int }
type CmykColor = { C : int; M : int; Y : int; K : int }
type HslColor = { H : int; S : int; L : int }

let detectColorSpace (cs : obj) =
  match cs with
  **| :? RgbColor -> printfn "RGB"**
  **| :? CmykColor -> printfn "CMYK"**
  **| :? HslColor -> printfn "HSL"**
  | _ -> failwith "Unrecognized"
```

## 作为模式

*As 模式*让你将一个名称绑定到整个匹配值，尤其在使用模式匹配和模式匹配函数的`let`绑定中很有用，因为在这些情况下，你没有直接访问匹配值的命名方式。

通常，`let`绑定只是将一个名字绑定到一个值，但正如你所见，你还可以在`let`绑定中使用模式来分解一个值，并将名字绑定到它的每个组成部分，像这样：

```
> **let x, y = (10, 20);;**

val y : int = 20
val x : int = 10
```

如果你想绑定不仅仅是组成部分，而是整个值，你可以像这样显式使用两个`let`绑定：

```
> **let point = (10, 20)**
**let x, y = point;;**

val point : int * int = (10, 20)
val y : int = 20
val x : int = 10
```

拥有两个独立的`let`绑定当然是可行的，但通过将它们合并为一个使用 As 模式的绑定会更加简洁，如下所示：

```
> **let x, y as point = (10, 20);;**

val point : int * int = (10, 20)
val y : int = 20
val x : int = 10
```

As 模式不仅限于在`let`绑定中使用；你还可以在匹配表达式中使用它。在这里，我们在每个案例中都包含了一个 As 模式，用来将匹配到的元组绑定到一个名称上。

```
let locatePoint =
  function
  | (0, 0) as p -> sprintf "%A is at the origin" p
  | (_, 0) as p -> sprintf "%A is on the X-Axis" p
  | (0, _) as p -> sprintf "%A is on the Y-Axis" p
  | (x, y) as p -> sprintf "Point (%i, %i)" x y
```

## 通过 AND 组合模式

使用*AND 模式*，有时也叫*合取模式*，你可以通过将多个兼容的模式与一个和符号（`&`）结合，来匹配输入。要匹配成功，输入必须满足每一个模式。

一般来说，在基本的模式匹配场景中，AND 模式并不是特别有用，因为更具表现力的守卫子句通常更适合完成任务。尽管如此，AND 模式在某些情况下仍然有用，例如在匹配另一个模式时提取值。（AND 模式在活跃模式中也被广泛使用，稍后我们会讨论。）例如，要确定一个二维点是否位于原点或沿坐标轴上，你可以写出类似这样的代码：

```
let locatePoint =
  function
  | (0, 0) as p -> sprintf "%A is at the origin" p
  | ① (x, y) & (_, 0) -> sprintf "(%i, %i) is on the x-axis" x y
  | ② (x, y) & (0, _) -> sprintf "(%i, %i) is on the y-axis" x y
  | (x, y) -> sprintf "Point (%i, %i)" x y
```

`locatePoint`函数在①和②使用与模式相结合的 AND 模式，从元组中提取`x`和`y`的值，当第二个或第一个值分别为 0 时。

## 通过 OR 组合模式

如果多个模式在匹配时应该执行相同的代码，你可以使用 OR（或*析取*）模式将它们结合起来。*OR 模式*通过竖线字符（`|`）将多个模式组合在一起。在许多方面，OR 模式类似于 C#中`switch`语句的穿透案例。

在这里，`locatePoint`函数已经被修改为使用 OR 模式，这样就能为位于任一坐标轴上的点打印相同的信息：

```
let locatePoint =
  function
  | (0, 0) as p -> sprintf "%A is at the origin" p
  | ① (_, 0) | ② (0, _) as p -> ③ sprintf "%A is on an axis" p
  | p -> sprintf "Point %A" p
```

在这个版本的`locatePoint`中，当①或②处的模式满足时，③处的表达式会被求值。

## 模式中的括号

在组合模式时，你可以通过括号来确定优先级。例如，要从一个点中提取`x`和`y`的值，并且匹配该点是否位于任一坐标轴上，你可以写出类似这样的代码：

```
let locatePoint =
  function
  | (0, 0) as p -> sprintf "%A is at the origin" p
  | (x, y) & ① ((_, 0) | (0, _)) -> sprintf "(%i, %i) is on an axis" x y
  | p -> sprintf "Point %A" p
```

在这里，你匹配了三个模式，通过将两个坐标轴检查模式用括号括起来，在①处建立了结合性。

## 活跃模式

当内置的模式类型无法完全满足需求时，你可以使用活跃模式。*活跃模式*是一种特殊的函数定义，称为*活跃识别器*，在这种模式下，你定义一个或多个案例名称，以便在模式匹配表达式中使用。

活动模式具有许多与内置模式类型相同的特征；它们接受一个输入值，并能将该值分解为其组成部分。然而，与基本模式不同的是，活动模式不仅允许你定义每个命名情况的匹配条件，还可以接受其他输入。

活动模式的定义语法如下：

```
let (|CaseName1|CaseName2|...|CaseNameN|) [parameters] -> expression
```

如你所见，情况名称被包含在 `(|` 和 `|)` 之间（称为 *香蕉夹*），并且以管道符分隔。活动模式定义必须至少包括一个参数用于匹配值，并且由于活动识别器函数是柯里化的，匹配的值必须是最终参数，以便与匹配表达式正确配合。最后，表达式的返回值必须是其中一个命名的情况，并附带任何相关的数据。

活动模式有许多用途，但一个好的例子是可能解决著名的 FizzBuzz 问题。对于那些未接触过的人，FizzBuzz 是一个面试中雇主有时用来筛选候选人的谜题。问题的核心任务很简单，通常表述如下：

> 编写一个程序，打印从 1 到 100 的数字。但是，对于 3 的倍数，打印 `"Fizz"` 代替数字；对于 5 的倍数，打印 `"Buzz"`。对于同时是 3 和 5 的倍数的数字，打印 `"FizzBuzz"`。

明确来说，活动模式当然不是解决 FizzBuzz 问题的唯一方式（也不一定是最好的方式）。但是，FizzBuzz 问题—其包含多个重叠的规则—使我们能够展示活动模式的强大。

我们可以从定义活动识别器开始。从前面的描述中，我们知道需要四个模式：`Fizz`、`Buzz`、`FizzBuzz`，以及一个默认情况用于其他所有情况。我们还知道每种情况的标准，因此我们的识别器可能长得像这样：

```
let (|Fizz|Buzz|FizzBuzz|Other|) n =
  match ① (n % 3, n % 5) with
  | ② 0, 0 -> FizzBuzz
  | ③ 0, _ -> Fizz
  | ④ _, 0 -> Buzz
  | ⑤ _ -> Other n
```

这里我们有一个活动识别器，它定义了四个情况名称。识别器的主体依赖于进一步的模式匹配来选择适当的情况。在 ① 处，我们构造一个元组，包含 *n* 对 3 和 5 的取余值。然后，我们使用一系列元组模式来识别正确的情况，最具体的是 ②，其中两个元素都为 0。③ 和 ④ 处的情况分别匹配当 *n* 能被 3 和 5 整除时。最后一个情况⑤，使用通配符模式来匹配所有其他情况，并返回 `Other` 以及提供的数字。尽管活动模式让我们解决了一部分问题，但我们仍然需要打印结果。

活动识别器仅识别给定数字符合哪种情况，因此我们仍然需要一种方法将每个情况转换为字符串。我们可以使用像这样的模式匹配函数轻松地映射这些情况：

```
let fizzBuzz =
  function
  | Fizz -> "Fizz"
  | Buzz -> "Buzz"
  | FizzBuzz -> "FizzBuzz"
  | Other n -> n.ToString()
```

上面的`fizzBuzz`函数使用了基本的模式匹配，但它没有使用内置的模式，而是使用了由活动识别器定义的模式。注意`Other`案例包含了一个变量模式`n`，用于保存与其关联的数字。

最后，我们可以通过打印结果来完成任务。我们可以用命令式的方式来做，但因为函数式编程更有趣，所以我们用类似这样的序列：

```
seq { 1..100 }
|> Seq.map fizzBuzz
|> Seq.iter (printfn "%s")
```

在这里，我们创建一个包含 1 到 100 的数字的序列，并将其传递给`Seq.map`，后者创建一个包含从`fizzBuzz`返回的字符串的新序列。然后，将生成的序列传递给`Seq.iter`以打印每个值。

## 部分活动模式

尽管活动模式很方便，但它们确实有一些缺点。首先，每个输入必须映射到一个命名的案例。其次，活动模式最多只能有七个命名案例。如果你的情况不需要映射每个可能的输入，或者你需要超过七个案例，你可以转向部分活动模式。

*部分活动模式*的结构与完全活动模式相同，但它们不包括案例名称的列表，而是只包括一个单一的案例名称，后跟一个下划线。部分活动模式的基本语法如下：

```
let (|CaseName|_|) [parameters] = expression
```

部分活动模式返回的值与完全活动模式有所不同。它们不会直接返回案例，而是返回模式类型的一个选项。例如，如果你有一个`Fizz`的部分活动模式，表达式需要返回`Some(Fizz)`或`None`。不过，对于匹配表达式来说，选项是透明的，因此你只需要处理案例名称。

### 注意

*如果你正在 F#交互式窗口中跟进，建议在继续下一个示例之前重置会话，以避免活动模式之间可能发生的命名冲突。*

为了看到部分活动模式的实际应用，我们可以回到 FizzBuzz 问题。使用部分活动模式可以让我们更简洁地重写解决方案。我们可以像这样定义部分活动模式：

```
let (|Fizz|_|) n = if n % 3 = 0 then Some Fizz else None
let (|Buzz|_|) n = if n % 5 = 0 then Some Buzz else None
```

在阅读上面的代码片段后，你可能首先想到的是“为什么只有两个案例，而问题明确指定了三个？”原因是部分活动模式是独立评估的。因此，为了满足要求，我们可以构造一个匹配表达式，使得一个案例同时匹配`Fizz`和`Buzz`，使用一个 AND 模式，如下所示：

```
let fizzBuzz =
  function
  | Fizz & Buzz -> "FizzBuzz"
  | Fizz -> "Fizz"
  | Buzz -> "Buzz"
  | n -> n.ToString()
```

现在，剩下的就是像之前一样打印所需的值：

```
seq { 1..100 }
|> Seq.map fizzBuzz
|> Seq.iter (printfn "%s")
```

## 参数化活动模式

到目前为止，我们看到的所有活动模式都只接受单一的匹配值；我们还没有看到接受额外参数来帮助匹配的模式。记住，活动识别器函数是柯里化的，因此，要在活动模式定义中包括额外的参数，你需要在匹配输入参数之前列出它们。

仅使用一个*参数化部分激活模式*，也可以构造另一种 FizzBuzz 问题的解决方案。考虑以下定义：

```
let (|DivisibleBy|_|) d n = if n % d = 0 then Some DivisibleBy else None
```

这个部分激活模式看起来与我们在上一节中定义的`Fizz`和`Buzz`部分激活模式完全相同，唯一的区别是它包括了`d`参数，并在表达式中使用它。现在我们可以使用这个模式从任何输入中解析出正确的单词，如下所示：

```
let fizzBuzz =
  function
  | DivisibleBy 3 & DivisibleBy 5 -> "FizzBuzz"
  | DivisibleBy 3 -> "Fizz"
  | DivisibleBy 5 -> "Buzz"
  | n -> n.ToString()
```

现在，我们不再为`Fizz`和`Buzz`编写专门的案例，而是通过参数化模式匹配输入是否能被三或五整除。输出结果与之前没有区别：

```
seq { 1..100 }
|> Seq.map fizzBuzz
|> Seq.iter (printfn "%s")
```

## 总结

模式匹配是 F#最强大、最灵活的特性之一。尽管它在表面上与其他语言中的基于案例的分支结构有些相似，但 F#的匹配表达式完全是另一种形式。模式匹配不仅提供了一种表达式丰富的方式来匹配和分解几乎任何数据类型，甚至还能够返回值。

在本章中，你学习了如何直接使用`match...with`构造匹配表达式，以及如何间接使用`function`关键字。你还看到简单的模式类型，如通配符、变量和常量模式，如何独立使用或与记录和列表等更复杂的模式结合使用。最后，你学习了如何使用完全和部分激活模式创建自己的自定义模式。
