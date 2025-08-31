## 第十二章。计算表达式

在第六章中，我们了解了序列表达式如何简化序列的创建。在第十章中，我们看到查询表达式如何为从不同数据源查询数据提供统一的方法。类似地，在第十一章中，我们探讨了如何利用异步工作流简化异步操作的创建与执行。这些构造各自服务于不同的目的，但它们共同的特点是，它们都是 F#语言的另一个特性：计算表达式。

*计算表达式*，有时也称为*工作流*，提供了一种方便的构造，用于表达一系列操作，其中数据流和副作用被控制。从这个角度来看，计算表达式类似于其他函数式语言所称的*单子*。然而，计算表达式的不同之处在于，它们被设计成个别表达式看起来像语言的自然部分。

在计算表达式的上下文中，你可以重新利用几个熟悉的语言元素——例如`let`和`use`关键字以及`for`循环——将语法与语言统一。计算表达式还为其中一些元素提供了替代的“bang”语法，允许你嵌套计算表达式进行内联求值。

由于该特性具有广泛的适用性，计算表达式可以简化与复杂类型的交互，并适用于各种场景。例如，我们已经知道内置的计算表达式可以简化序列创建、查询和异步处理，但它们也在日志记录和一些项目中具有应用，例如{m}brace 框架，旨在简化将计算任务卸载到云端。

在本章中，我们将探讨计算表达式的内部工作原理。我们将跳过讨论单子理论，因为它并不能帮助你理解计算表达式如何融入到你的解决方案中。相反，我们将从了解构建器类及其如何启用计算表达式开始。在建立了这个基础后，我们将通过两个自定义计算表达式的示例来进行讲解。

## 计算表达式的结构

你已经熟悉了编写计算表达式的基本模式，但直到现在，你还没有看到它们如何运作，除了在我们创建一些额外查询操作符时对其背后原理进行简短的介绍（参见扩展查询表达式）。为了更一般地重申，计算表达式具有以下形式：

```
*builder-name* { *computation-expression-body* }
```

计算表达式围绕一个基本的*计算类型*（有时称为*单子类型*）设计，我们通过透明地调用*构建器类*暴露的方法来进行计算。在前面的语法中，*builder-name* 表示构建器类的一个具体实例，*computation-expression-body* 表示一系列嵌套的表达式，这些表达式映射到产生计算类型实例所需的方法调用。例如，异步工作流基于 `Async<'T>`，通过 `AsyncBuilder` 构建。类似地，查询表达式基于 `QuerySource<'T, 'Q>`，通过 `QueryBuilder` 构建。

### 注意

*序列表达式在计算表达式领域中是一种特例，因为它们不遵循正常的实现模式。尽管序列表达式使用计算表达式语法并基于 `IEnumerable<'T>`，但它们没有对应的构建器类。相反，通常由构建器类处理的细节直接由 F# 编译器处理。*

构建器类定义了计算表达式支持的操作。定义构建器类在很大程度上是一种约定，因为没有特定的接口需要实现，也没有基类需要继承。对于构建器类的命名没有严格的规则，但通常是通过在基础类型名称后附加 `Builder` 来命名（例如，`AsyncBuilder` 和 `QueryBuilder`）。

虽然计算表达式是语言的一部分，但它们实际上只是语法糖——一种更方便的方式来调用构建器类的方法。当编译器遇到看似计算表达式的代码时，它会尝试通过一个叫做*去糖化*的过程将代码转换为一系列方法调用。这个过程涉及用构建器类型上对应实例方法的调用替换计算表达式中的每个操作（类似于 LINQ 查询表达式如何转换为 C# 和 Visual Basic 中的扩展方法调用和委托）。我喜欢把构建器类的方法分为两组。第一组列在表 12-1 中，控制各种语法元素，如绑定、`for` 和 `while` 循环、以及返回值。

表 12-1. 语法元素控制方法

| 方法 | 描述 | 签名 |
| --- | --- | --- |
| `Bind` | 启用 `let!` 和 `do!` 绑定 | `M<'T> * ('T -> M<'U>) -> M<'U>` |
| `For` | 启用 `for` 循环 | `seq<'T> * ('T -> M<'U>) -> M<'U>` 或 `seq<'T> * ('T -> M<'U>) -> seq<M<'U>>` |
| `Return` | 启用 `return` | `'T -> M<'T>` |
| `ReturnFrom` | 启用 `return!` | `M<'T> -> M<'T>` |
| `TryFinally` | 允许通过 `try...finally` 进行异常处理 | `M<'T> * (unit -> unit) -> M<'T>` |
| `TryWith` | 允许通过 `try...with` 进行异常处理 | `M<'T> * (exn -> M<'T>) -> M<'T>` |
| `Using` | 使得可以使用 `use` 和 `use!` 创建 `IDisposable` 对象 | `'T * ('T -> M<'U>) -> M<'U>`当`'U :> IDisposable`时 |
| `While` | 允许在计算表达式中使用 `while...do` 循环 | `(unit -> bool) * M<'T> -> M<'T>` |
| `Yield` | 使用 `yield` 关键字，以类似序列的方式从嵌套的计算表达式中返回项 | `'T -> M<'T>` |
| `YieldFrom` | 使用 `yield!` 关键字，以类似序列的方式从嵌套的计算表达式中返回项 | `M<'T> -> M<'T>` |

第二组方法，控制计算表达式如何被评估的方法，列在表 12-2 中。

表 12-2. 影响计算表达式评估的方法

| 方法 | 描述 | 签名 |
| --- | --- | --- |
| `Combine` | 将计算表达式的两个部分合并成一个 | `M<'T> * M<'T> -> M<'T>`或`M<unit> * M<'T> -> M<'T>` |
| `Delay` | 将计算表达式包装成一个函数，以便延迟执行，从而帮助防止不必要的副作用 | `(unit -> M<'T>) -> M<'T>` |
| `Run` | 在评估计算表达式时作为最后一步执行；可以通过调用 `Delay` 返回的函数来“撤销”延迟，也可以将结果转换为更易消费的格式 | `M<'T> -> M<'T>`或`M<'T> -> 'T` |
| `Zero` | 返回表达式的单子类型的默认值；当计算表达式没有显式返回值时使用 | `unit -> M<'T>`（`'T` 可以是 `unit`） |

由于计算表达式的设计目的是使其能够适用于各种情况，因此保持它们尽可能通用非常重要。这一点通过签名的高度通用结构得以体现。例如，`M<_>`的符号表示底层类型封装了另一个值。

在你的构建器类中并不需要实现表 12-1 中列出的每个方法。然而，如果你省略了某些方法，相应的映射语法将无法在计算表达式中使用，编译器会报错。例如，如果你尝试在自定义计算表达式中包含 `use` 绑定，但省略了构建器类中的 `Using` 方法，编译将失败，并显示如下错误信息：

```
error FS0708: This control construct may only be used if the computation
expression builder defines a 'Using' method
```

同样，并非每个方法都需要从表 12-2 实现，但在某些情况下未实现某些方法可能会导致不希望出现的结果。例如，未实现 `Delay` 方法将阻止你组合返回多个结果的表达式。此外，当计算表达式涉及副作用时，未实现 `Delay` 方法可能会过早引发副作用——无论它们出现在表达式的何处——因为它们会在遇到时立即被评估，而不是被包装在一个函数中以便延迟执行。

当我们仅仅讨论构建器类和方法调用时，计算表达式可能很难理解。我认为，走过一些简单的实现示例，看看这些组件如何协同工作，更加有帮助。本章剩余的部分我们将讨论两个示例。特别是，我们将查看构建器实现、它们对应的表达式语法以及去糖过程。

## 示例：FizzBuzz

在第七章中，我们研究了几种通过使用`Seq.map`迭代序列以及使用带有活动模式和部分活动模式的模式匹配函数来解决 FizzBuzz 问题的方法。然而，FizzBuzz 问题的核心本质上只是一个序列转换的练习。因此，使用计算表达式可以轻松解决该问题。

当作为计算表达式实现时，我们的 FizzBuzz 序列可以以一种方式构建，使其看起来和行为像一个标准的序列表达式。然而，使用计算表达式时，将数字映射到相应的字符串将完全抽象化，隐藏在构建器类中。

由于 FizzBuzz 将整数转换为字符串并且不包含内在状态，我们将跳过创建中介包装类型，直接从创建构建器类开始，逐步实现，首先从 `Yield` 方法开始。

```
type FizzBuzzSequenceBuilder() =
  member x.Yield(v) =
    match (v % 3, v % 5) with
    | 0, 0 -> "FizzBuzz"
    | 0, _ -> "Fizz"
    | _, 0 -> "Buzz"
    | _ -> v.ToString()
```

现在我们已经有了一个基础的构建器类，我们可以创建实例，并在每次 FizzBuzz 计算表达式中使用它，像这样：

```
let fizzbuzz = FizzBuzzSequenceBuilder()
```

就这样！没有什么花哨的地方；我们只是通过它的主构造函数创建了类的一个实例。为了将该实例用作计算表达式，我们可以编写如下内容：

```
> **fizzbuzz { yield 1 };;**
val it : string = "1"
```

如你所见，评估前面的表达式并没有给我们预期的结果。它并没有返回一个字符串序列，而是只返回了一个单一的字符串，因为到目前为止，构建器类还不知道如何创建序列；它只是基于整数值返回一个字符串。你可以在去糖后的形式中更清楚地看到这一点，它大致如下：

```
fizzbuzz.Yield 1
```

要获得一个字符串序列，我们可以让`Yield`返回一个单例序列（只包含一个项目的序列），但这样做会使实现其他方法（如`For`和`While`）变得复杂。相反，我们将扩展构建器类，包含`Delay`方法，如下所示（确保在更新构建器类后重新创建构建器实例，以确保使用最新定义来评估`fizzbuzz`表达式）：

```
type FizzBuzzSequenceBuilder() =
-- *snip* --
member x.Delay(f) = f() |> Seq.singleton
```

在`Delay`方法到位的情况下，评估之前的`fizzbuzz`表达式会得到一个稍微更理想的结果：

```
> **fizzbuzz { yield 1 };;**
val it : seq<string> = seq ["1"]
```

同样，去糖化后的表达式可以帮助澄清发生了什么。通过包含`Delay`方法，去糖化后的形式现在如下所示：

```
fizzbuzz.Delay(fun () -> fizzbuzz.Yield 1)
```

但是，如今，来自`fizzbuzz`表达式的所有结果都只是一个单例序列，因为我们无法生成多个值。实际上，试图按照以下方式生成多个值将导致编译器错误，指示构建器类必须定义一个`Combine`方法：

```
fizzbuzz {
  yield 1
  yield 2
  yield 3 }
```

为了使前面的代码片段能够正常工作，我们将提供两个重载版本的`Combine`方法。重载方法的原因是，根据表达式中的位置，我们可能是将单个字符串组合成一个序列，或者是将一个新的字符串附加到现有的序列中。我们需要小心，避免创建包含序列的序列，因此我们还需要重载现有的`Delay`方法，使其简单地返回一个提供的序列。我们可以按如下方式实现这些方法：

```
type FizzBuzzSequenceBuilder() =
  -- *snip* --
  member x.Delay(f : unit -> string seq) = f()
  member x.Combine(l, r) =
    Seq.append (Seq.singleton l) (Seq.singleton r)
  member x.Combine(l, r) =
    Seq.append (Seq.singleton l) r
```

现在，评估前面的`fizzbuzz`表达式将得到一个包含三个字符串的序列：

```
> **fizzbuzz {**
  **yield 1**
  **yield 2**
  **yield 3 };;**
val it : seq<string> = seq ["1"; "2"; "Fizz"]
```

当像这样生成多个结果时，去糖化过程会产生一个更复杂的链式方法调用。例如，去糖化前面的表达式（生成三个项）会得到类似下面的代码：

```
fizzbuzz.Delay (fun () ->
  fizzbuzz.Combine (
    fizzbuzz.Yield 1,
    fizzbuzz.Delay (fun () ->
      fizzbuzz.Combine(
        fizzbuzz.Yield 2,
        fizzbuzz.Delay (fun () -> fizzbuzz.Yield 3)))))
```

一次性生成一个实例的方式（我们一直在使用的这种方式）并不是构建任意长度序列的高效方法。如果我们能够通过一个`for`循环来组合一个`fizzbuzz`表达式，那就会更好。为此，我们需要实现`For`方法。我们采取的方法是简单地包装一次对`Seq.map`的调用，如下所示：

```
type FizzBuzzSequenceBuilder() =
  -- *snip* --
  member x.For(g, f) = Seq.map f g
```

现在生成 FizzBuzz 序列变得非常简单，因为我们可以将一个单独的`yield`表达式嵌套在`for`循环中，而不是使用多个`yield`表达式，像这样：

```
fizzbuzz { for x = 1 to 99 do yield x }
```

在构建器类中实现`Yield`、`Delay`、`Combine`和`For`方法的一个优点是，我们可以将这些风格组合起来，从而实现更灵活的表达式。例如，我们可以直接在循环中生成值，然后再将它们输出：

```
fizzbuzz { yield 1
           yield 2
           for x = 3 to 50 do yield x }
```

如目前所写，构建器类并不支持你可以组合各种表达式的每一种方式，但你不应该在添加适当的重载以支持更多场景时遇到问题。

为了方便起见，这里是完整的构建器类：

```
type FizzBuzzSequenceBuilder() =
  member x.Yield(v) =
    match (v % 3, v % 5) with
    | 0, 0 -> "FizzBuzz"
    | 0, _ -> "Fizz"
    | _, 0 -> "Buzz"
    | _ -> v.ToString()
  member x.Delay(f) = f() |> Seq.singleton
  member x.Delay(f : unit -> string seq) = f()
  member x.Combine(l, r) =
    Seq.append (Seq.singleton l) (Seq.singleton r)
  member x.Combine(l, r) =
    Seq.append (Seq.singleton l) r
  member x.For(g, f) = Seq.map f g
```

## 示例：构建字符串

FizzBuzz 很好地展示了如何使用计算表达式通过`For`和`Yield`方法创建自己的类似序列的构造，但它对于日常计算并不特别实用。为了得到一个更实用的例子，我们转向一个常见的编程任务：合并字符串。

长久以来，使用`StringBuilder`构建字符串通常比连接字符串更高效已被广泛认可。`StringBuilder`的流畅接口使代码保持相当简洁，如下所示：

```
open System.Text

StringBuilder("The quick ")
  .Append("brown fox ")
  .Append("jumps over ")
  .Append("the lazy dog")
  .ToString()
```

创建一个`StringBuider`实例并将不同的`Append`方法链接调用并不完全符合函数式优先的范式，然而，`Printf`模块通过`bprintf`函数试图解决这种脱节问题，`bprintf`函数格式化一个字符串并将其附加到`StringBuilder`实例中，如下所示：

```
let sb = System.Text.StringBuilder()
Printf.bprintf sb "The quick "
Printf.bprintf sb "brown fox "
Printf.bprintf sb "jumps over "
Printf.bprintf sb "the lazy dog"
sb.ToString() |> printfn "%s"
```

然而，`bprintf`所完成的事情仅仅是将实例方法调用替换为一个接收`StringBuilder`作为参数的函数调用。更重要的是，你仍然需要管理`StringBuilder`实例，并将其传递给每一个`bprintf`调用。通过计算表达式，你不仅可以让字符串构造看起来像 F#语言的自然部分，还可以抽象掉`StringBuilder`！我们将很快定义的计算表达式将允许我们使用以下语法组合字符串：

```
buildstring {
  yield "The quick "
  yield "brown fox "
  yield "jumps over "
  yield "the lazy dog" }
```

在这里，我们通过在`buildstring`表达式中`yield`多个字符串来将它们串联起来。为了实现这一点，我们首先需要定义表达式的基础类型。为了方便起见，我们将使用一个称为`StringFragment`的判别联合来跟踪我们在`yield`时所有的字符串。`StringFragment`类型定义如下：

```
open System.Text

type StringFragment =
| ① Empty
| ② Fragment of string
| ③ Concat of StringFragment * StringFragment
  override x.ToString() =
    let rec flatten frag (sb : StringBuilder) =
      match frag with
      | Empty -> sb
      | Fragment(s) -> sb.Append(s)
      | Concat(s1, s2) -> sb |> flatten s1 |> flatten s2
    (StringBuilder() |> flatten x).ToString()
```

`StringFragment`联合体有三种情况，`Empty`①，`Fragment`②和`Concat`③。`Empty`表示空字符串，而`Fragment`包含一个单一的字符串。最后的情况，`Concat`，形成一个`StringFragment`实例的层次结构，最终通过`ToString`方法将它们连接在一起。这种类型的优点在于，一旦构建器就位，你就不需要手动管理这些实例或`StringBuilder`了。

构建器类，我们称之为`StringFragmentBuilder`，与

`FizzBuzzBuilder`，但它不是创建序列，而是创建`StringFragment`。根据之前的语法，我们已经知道我们将使用`yield`关键字，因此我们需要提供一个`Yield`方法。为了生成多个项，我们还需要实现`Combine`和`Delay`方法。此外，允许嵌套表达式也是一个不错的主意，因此我们将实现一个`YieldFrom`方法。以下是完整的`StringFragmentBuilder`类，以及与`buildString`表达式一起使用的实例：

```
type StringFragmentBuilder() =
  member x.Zero() = Empty
  member x.Yield(v) = Fragment(v)
  member x.YieldFrom(v) = v
  member x.Combine(l, r) = Concat(l, r)
  member x.Delay(f) = f()
  member x.For(s, f) =
    Seq.map f s
    |> Seq.reduce (fun l r -> x.Combine(l, r))

let buildstring = StringFragmentBuilder()
```

`StringFragmentBuilder`类比`FizzBuzzSequenceBuilder`简单得多，因为它仅关注将字符串映射到`StringFragments`并控制执行。我们逐一查看每个方法，以了解它们在计算表达式中的使用方式。

第一个方法`Zero`为表达式返回一个默认值。在这种情况下，我们返回`Empty`表示一个空字符串。在去糖化过程中，当表达式返回`unit`或嵌套的`if`表达式不包括`else`分支时，会自动插入对`Zero`的调用。

`Yield`方法在`buildstring`表达式中启用了`yield`关键字。在这个实现中，`Yield`接受一个字符串，并将其包装在一个新的`Fragment`实例中。

`YieldFrom`方法允许你通过`yield!`关键字求值一个嵌套的`buildstring`表达式。这个方法类似于`Yield`，但它返回的是嵌套表达式创建的`StringFragment`，而不是返回一个新的`StringFragment`。

每个`yield`或`yield!`在计算表达式中代表着表达式的一部分结束，因此我们需要一种方法将它们合并在一起。为此，我们使用`Combine`方法，它本质上将表达式的其余部分视为一个延续。`Combine`接受两个`StringFragments`，并将它们各自包装在一个`Concat`实例中。

Combine，暴露

我认为通过查看去糖化的形式，更容易理解`Combine`方法的作用。假设你正在编写一个`buildstring`表达式，将`"A"`和`"B"`合并为一个字符串，如下所示：

```
buildstring {
  yield "A"
  yield "B" }
```

该表达式的相应去糖化形式将非常类似于此：

```
buildstring.Combine(
  buildstring.Yield("A"),
  buildstring.Yield("B"))
```

为了更清晰地理解，我将去糖化的形式简化为仅包含理解过程所需的部分。这里，第一个`Yield`调用返回`Fragment("A")`，第二个返回`Fragment("B")`。`Combine`方法接受这两者并生成以下内容：

```
Concat (Fragment "A", Fragment "B")
```

`Combine`会在第一个`yield`之后为每个`yield`调用。如果我们的假设示例扩展到也`yield` `"C"`，那么去糖化后的形式将类似于以下简化代码：

```
buildstring.Combine(
  buildstring.Yield("A"),
  buildstring.Combine(
    buildstring.Yield("B"),
    buildstring.Yield("C")))
```

结果的`StringFragment`应为：

```
Concat (Fragment "A", Concat (Fragment "B", Fragment "C"))
```

`StringFragmentBuilder`类中的下一个方法`Delay`控制计算表达式何时被求值。当一个计算表达式有多个部分时，编译器要求你定义`Delay`以避免过早求值包含副作用的表达式，并在表达式组合时控制执行。许多方法调用被包装在传递给`Delay`的函数中，这样这些表达式部分直到调用`Delay`时才会被求值。更具体地说，整个表达式被包装在一个`Delay`调用中，每个`Combine`调用的第二个参数计算也被如此包装。去糖化后的形式大致如下（为清晰起见简化）：

```
buildstring.Delay(
  fun () ->
    buildstring.Combine(
      buildstring.Yield("A"),
      buildstring.Delay(
        fun () ->
          buildstring.Combine(
           buildstring.Yield("B"),
             buildstring.Delay(
               fun () ->
                 buildstring.Yield("C"))))))
```

最后，`For`方法允许我们在`buildstring`表达式中使用`for`循环。然而，与 FizzBuzz 实现不同，这个版本采用了 Map/Reduce 模式，将提供的序列值映射到单独的`StringFragment`实例，然后通过`Combine`方法将它们减少成一个单一的`StringFragment`实例。这个扁平化的实例可以与其他实例一起使用。

现在你已经看过构建器类，并理解了这些方法是如何通过去糖化过程协同工作的，让我们来看一个完整的例子，展示如何执行整个链条。为此，我们可以使用`buildstring`表达式来构建一首关于农夫和他的小狗 Bingo 的儿童歌曲的歌词。这首歌的简单歌词和重复性质使得它很容易用编程的方式表示，如下所示：

```
let bingo() =
  let buildNamePhrase fullName =
    buildstring {
      yield "And "
      yield fullName
      yield " was his name-o\n"
   }
  let buildClapAndSpellPhrases maxChars chars =
    let clapCount = maxChars - (List.length chars)
    let spellPart =
      List.init clapCount (fun _ -> "*clap*") @ chars
      |> Seq.ofList
      |> String.concat "-"
    buildstring {
      for i in 1..3 do yield spellPart
                       yield "\n" }
  let rec buildVerse fullName (chars : string list) =
    buildstring {
      yield "There was a farmer who had a dog,\n"
      yield! buildNamePhrase fullName
      yield! buildClapAndSpellPhrases fullName.Length chars
      yield! buildNamePhrase fullName
      match chars with
      | [] -> ()
      | _::nextChars -> yield "\n"
                        yield! buildVerse fullName nextChars
    }
  let name = "Bingo"
  let letters = [ for c in name.ToUpper() -> c.ToString() ]
  buildVerse name letters
```

`bingo`函数内部嵌套了三个函数：`buildNamePhrase`、`buildClapAndSpellPhrases`和`buildVerse`。这三个函数通过`buildstring`表达式构建一个`StringFragment`。在每个诗句的末尾，`buildstring`表达式包含一个`match`表达式，用来判断是否应该以`Zero`值（通过返回`unit`来隐含表示）结束，或者通过`yield!`关键字递归地包含另一个完全构造的诗句。

评估前面的代码片段应该会打印出以下字符串（记住，`%O`标记会通过调用相应对象的`ToString`方法来格式化该参数）：

```
> **bingo() |> printfn "%O";;**
There was a farmer who had a dog,
And Bingo was his name-o!
B-I-N-G-O
B-I-N-G-O
B-I-N-G-O
And Bingo was his name-o!

There was a farmer who had a dog,
And Bingo was his name-o!
*clap*-I-N-G-O
*clap*-I-N-G-O
*clap*-I-N-G-O
And Bingo was his name-o!

There was a farmer who had a dog,
And Bingo was his name-o!
*clap*-*clap*-N-G-O
*clap*-*clap*-N-G-O
*clap*-*clap*-N-G-O
And Bingo was his name-o!
-- *snip* --
```

## 总结

计算表达式在 F#中扮演着重要角色。开箱即用，它们使得创建序列、从不同数据源查询数据以及管理异步操作看起来像是语言的原生功能，借助了语言中熟悉的元素。它们还具有完全的可扩展性，因此你可以通过创建构建器类来定义自己的计算表达式，构造底层类型的实例。创建自定义的计算表达式可能是一个具有挑战性的任务，但一旦理解了每个构建器类方法的目的和去糖化过程，最终结果可以使代码更加简洁、具有描述性。

关于计算表达式的信息可能比较难找，但你可以使用一些资源进行深入学习。首先，*F# for Fun and Profit*系列文章（* [`fsharpforfunandprofit.com/series/computation-expressions.htm`](http://fsharpforfunandprofit.com/series/computation-expressions.htm) *）提供了许多涵盖不同构建器方法的示例。如果你需要一些更真实的应用实例，可以查看 GitHub 上的 ExtCore 项目（* [`github.com/jack-pappas/ExtCore/`](https://github.com/jack-pappas/ExtCore/)*），其中包含了多个计算表达式的实际应用，如懒加载列表实现。
