## 第十一章。异步编程和并行编程

在计算机历史的大部分时间里，软件开发者一直受益于处理器制造商不断突破其芯片时钟速度的极限。如果你希望软件运行得更快（处理更大的数据集，或因为用户抱怨系统在忙碌时似乎冻结了），通常只需要升级到最新的处理器。然而，过去十年左右，情况发生了变化：处理器制造商开始通过增加处理核心来提高处理器性能，而不是单纯提高时钟速度。

尽管处理器架构已经发生变化，但软件架构大体上仍保持不变。多核处理器已成为常态，但许多应用程序仍然以只有一个核心可用的方式编写，因此无法充分利用底层硬件。长时间运行的任务仍然在 UI 线程上执行，大型数据集通常是同步处理的。这其中一个重要原因是，传统上，异步编程和并行编程足够复杂且容易出错，因此它们通常是专家开发者在高度专业化的软件中使用的领域。

幸运的是，软件正在逐步迎头赶上。程序员们正在意识到，通过更快的硬件来解决性能问题的时代已经过去，越来越重要的是在架构层面考虑并发处理的需求。

尽管它们密切相关，异步编程和并行编程的目标是不同的。异步编程旨在分离处理过程并减少阻塞，以便长时间运行的任务不会阻碍系统在同一进程中完成其他任务。相比之下，并行处理则旨在通过将工作划分为可分配给多个处理器并独立操作的任务，从而提高性能。

自.NET Framework 诞生以来，它通过线程和多种同步机制（如监视器、互斥量、信号量等）支持异步编程和并行编程。*异步编程模型（APM）*，其中类定义了`BeginX`和`EndX`方法用于需要异步执行的操作（例如`System.IO.FileStream`类中的`BeginRead`和`EndRead`方法），长期以来一直是.NET 中异步编程的首选方式。

本章将探讨 F#使异步编程和并行编程更易于访问的几种方式，从而让你可以专注于创建正确的解决方案。我们将从对任务并行库（Task Parallel Library）的简要介绍开始。接下来，我们将讨论另一个 F#构造：异步工作流。最后，我们将介绍`MailboxProcessor`，F#基于代理的异步编程模型。

## 任务并行库

正如其名称所示，*任务并行库（TPL）* 擅长处理并行编程场景，并且是 CPU 密集型操作的首选机制。它通过统一的接口抽象了管理线程、锁、回调、取消和异常处理的大部分复杂性。尽管 TPL 并非专门针对 F#，但理解它的基本概念仍然很有帮助，尤其是当你需要与使用它的库中的代码进行交互时。

TPL 提供两种并行类型：数据并行和任务并行。

+   ****数据并行****。涉及对序列中的每个值执行特定操作，通过有效地将工作分配到可用的处理资源上。在数据并行模型下，你需要指定一个序列及相应的操作，TPL 会决定如何划分数据并相应地分配工作。

+   ****任务并行****。专注于并发执行独立任务。在任务并行中，你需要手动创建和管理任务，但该模型为你提供了更多的控制权。通过各种 `Task` 类，你可以轻松启动异步处理，等待任务完成，返回值，设置后续任务或生成额外任务。

### 注意

*本节内容并非旨在提供关于 TPL 的全面指南。因此，它不会涉及任务创建、调度、管理或其他相关主题的许多细节。这里的目的是建立一个基准，提供足够的信息，使你在使用 TPL 编写代码时能够立即提高生产力。*

### 潜在并行性

直接操作线程与使用 TPL 之间的一个主要区别在于，TPL 是基于任务的，而不是基于线程的。这一差异非常重要，因为 TPL 试图通过从线程池中提取线程来并发执行任务，但它并不保证并行性。这被称为 *潜在并行性*。

每当你直接创建一个线程时，就会产生分配和调度的开销。如果没有足够的系统资源来支持线程，这种开销可能会对整体系统性能造成不利影响。基本的并发机制，如线程池，帮助通过重用现有线程来减少影响，但 TPL 进一步考虑了可用的系统资源。如果没有足够的资源可用，或者 TPL 认为并行执行任务会对性能造成不利影响，它将同步执行任务。随着资源随时间波动，TPL 的任务调度和工作划分算法帮助重新平衡工作，以有效利用可用资源。

### 数据并行

数据并行性主要通过使用位于 `System.Threading.Tasks` 命名空间中的 `Parallel` 类的静态 `For` 和 `ForEach` 方法来实现。正如它们的名称所示，这些方法本质上是简单的 `for` 循环和可枚举 `for` 循环的并行版本。

### 注意

*数据并行性还可以通过 PLINQ（并行 LINQ）的 AsParallel 扩展方法来实现。为了简化在 F# 中处理并行序列，F# PowerPack 中的 PSeq 模块使用与 Seq 模块相同的命名法暴露了许多 ParallelEnumerable 方法。*

对于普通用法，`Parallel.For` 和 `Parallel.ForEach` 仅在输入上有所不同；`Parallel.For` 接受范围边界，而 `Parallel.ForEach` 接受一个序列。这两个方法还接受一个作为循环体的函数，并且它们会隐式地等待所有迭代完成后再将控制权返回给调用者。由于这两个方法非常相似，本节的示例将统一使用 `Parallel.For` 以保持一致性。

最简单的形式，即并行 `for` 循环，仅为范围中的每个值调用一次操作。在这里，我们使用并行 `for` 循环来写出数字 0 到 99。

```
open System
open System.Threading.Tasks

Parallel.For(0, 100, printfn "%i")
```

这个代码片段几乎不需要解释。传递给 `Parallel.For` 的第一个参数标识范围的包含起点，第二个参数标识范围的排除终点。第三个参数是一个将数字写入控制台的函数。

#### 锁定与避免锁定

现在我们正在处理并发，前面的示例中有一个微妙的 bug。内部，`printfn` 会在解析模式时逐步将文本发送到 `System.Console.Out`。因此，随着每个并行迭代的执行，可能会同时调用多个 `printfn`，导致一些项目交织在一起。

### 注意

*用于本讨论的示例在 F# 3.1 中并不是一个问题，因为 printf 和相关函数得到了改进，使其运行速度比以前的版本快了最多 40 倍。*

我们可以通过几种方式来解决这个问题。一个方法是使用 `lock` 操作符控制对 `System.Console.Out` 的访问。`lock` 操作符与 C# 中的 `lock` 语句（Visual Basic 中的 `SyncLock`）具有相同的作用，即在锁定资源释放之前，防止其他线程执行该代码块。以下是将前一个示例重新编写以使用锁定的代码：

```
Parallel.For(0, 100, fun n -> lock Console.Out (fun () -> printfn "%i" n))
```

有时锁定是合适的，但像这样使用它是一个糟糕的主意。通过锁定，我们抵消了并行化循环的大部分好处，因为一次只能写入一个项目！相反，我们希望尝试另一种方法，避免锁定并且不交织结果。

实现满意结果的最简单方法之一是函数组合。在这里，我们使用 `sprint` 函数来格式化数字，并将结果传递给 `Console.WriteLine`：

```
Parallel.For(0, 100, (sprintf "%i") >> Console.WriteLine)
```

这种方法有效，因为每次调用 `sprintf` 都是写入一个独立的 `StringBuilder`，而不是共享的 `TextWriter`。这消除了锁的需求，从而消除了应用程序中的潜在瓶颈。

#### 并行循环的短路操作

与 F# 内建的 `for` 循环不同，并行循环通过 `ParallelLoopState` 类的 `Break` 和 `Stop` 方法提供了一些短路机制。TPL 负责创建和管理循环状态，因此你所需要做的就是使用暴露这些方法的重载。考虑以下 `shortCircuitExample` 函数：

```
open System.Collections.Concurrent
open System.Threading.Tasks

let shortCircuitExample shortCircuit =
  let bag = ConcurrentBag<_>()
  Parallel.For(
    0,
    999999,
  ① fun i s -> if i < 10000 then bag.Add i else shortCircuit s) |> ignore
  (bag, bag.Count)
```

与之前的示例一样，`shortCircuitExample` 函数使用 `Parallel.For`，但请注意在①处，提供的函数接受两个参数，而不是一个。第二个参数 `s` 是循环状态。

使用 `shortCircuitExample` 后，我们可以调用它，传递一个接受 `ParallelLoopState` 实例并调用 `Stop` 或 `Break` 的函数，像这样：

```
shortCircuitExample (fun s -> s.Stop()) |> printfn "%A"
shortCircuitExample (fun s -> s.Break()) |> printfn "%A"
```

上述两行都会强制并行循环在所有迭代完成之前终止，但它们的效果截然不同。`Stop` 会使循环在最早的便利时终止，但允许正在执行的任何迭代继续；而`Break` 会使循环在当前迭代之后的最早时刻终止。你还需要注意，避免连续调用 `Stop` 和 `Break`，以免引发 `InvalidOperationException`。

这两个方法的差异可能非常明显。例如，在我桌面的一次运行中，`Break` 版本处理了 10,000 个项，而 `Stop` 版本仅处理了 975 个。

#### 取消并行循环

取消并行 `for` 循环类似于短路操作，只不过它不是通过 `Stop` 或 `Break` 方法从内部终止循环，而是识别一个外部的 *取消令牌*，该令牌由循环监视并做出响应。与短路机制不同，取消操作会强制所有使用相同令牌配置的任务停止。取消操作会引发 `OperationCanceledException`，因此你需要适当处理这个异常。

以下函数演示了如何取消并行 `for` 循环：

```
open System
open System.Threading.Tasks

let parallelForWithCancellation (wait : int) =
  use tokenSource = new ① System.Threading.CancellationTokenSource(wait)

  try
    Parallel.For(
      0,
      Int32.MaxValue,
    ② ParallelOptions(③ CancellationToken = ④ tokenSource.Token),
      fun (i : int) -> Console.WriteLine i
    ) |> ignore
  with
  | :? ⑤ OperationCanceledException -> printfn "Cancelled!"
  | ex -> printfn "%O" ex
```

在上面的代码中，我们在①创建了一个 `CancellationTokenSource`。该对象初始化为在指定的毫秒数后自动取消。在 `try` 块内部，我们使用了一个重载的 `Parallel.For`，它接受一个 `ParallelOptions` 实例，如②所示。通过这个 `ParallelOptions` 实例，我们将 `CancellationToken` 属性③初始化为 `CancellationTokenSource` ④所暴露的令牌。当令牌源的内部定时器到期时，并行循环会引发一个异常，然后在⑤处捕获并处理。虽然我们依赖于一个自动取消的 `CancellationTokenSource`，你也可以通过调用 `Cancel` 方法手动强制取消，通常是在另一个任务或线程中执行。

### 任务并行性

任务并行性使你可以在执行代码并行时，仍能对执行过程有最大的控制，同时抽象掉了许多实现细节。

#### 创建并启动任务

任务可以通过几种方式创建和启动。最简单，但最不灵活的方式是 `Parallel.Invoke` 方法，它接受一个或多个函数并行执行，并隐式等待它们完成，像这样：

```
open System
open System.Threading.Tasks

Parallel.Invoke(
  (fun () -> printfn "Task 1"),
  (fun () -> Task.Delay(100).Wait()
             printfn "Task 2"),
  (fun () -> printfn "Task 3")
)

printfn "Done"
```

在这里，`Parallel.Invoke` 创建并启动了三个独立的任务。第一个和第三个任务只是简单地打印消息，而第二个任务在打印消息之前等待了 100 毫秒。

`Parallel.Invoke` 限制了你可以做的事情，因为它不暴露任何关于单个任务的信息，也没有提供任务成功或失败的反馈。你可以通过提供取消令牌来捕获和处理任务引发的异常并取消任务（类似于在取消并行循环中使用的方法），但就是这么多了。当你想对任务做更高级的操作时，你需要手动创建它们。

创建任务有两种手动方式：通过构造函数直接创建，或者通过 `TaskFactory`。就我们的目的而言，这两种方法的主要区别在于，使用构造函数创建任务时，你必须手动启动它们。微软推荐在任务创建和调度不需要分开时，优先使用 `TaskFactory`。

要使用 `Task` 构造函数创建一个新任务，你只需要提供一个作为任务主体的函数，像这样：

```
open System.Threading.Tasks

let t = new Task(fun () -> printfn "Manual Task")
```

这会创建一个打印字符串的新任务。要启动任务，调用它的 `Start` 方法。

```
t.Start()
```

或者，你可以通过 `TaskFactory` 将这两个步骤合并为一步。方便的是，`Task` 类有一个静态的 `Factory` 属性，已经预设为一个默认的 `TaskFactory`，因此你无需自己创建一个。在这里，我们使用默认工厂的 `StartNew` 方法创建并启动一个任务：

```
open System.Threading.Tasks

let t = Task.Factory.StartNew(fun () -> printfn "Factory Task")
```

#### 从任务中返回值

我们迄今为止所看到的任务只是调用一个操作，但你还需要知道如何返回值——这是传统异步模型中一个常见且繁琐的过程。TPL 通过一个泛型 `Task<'T>` 类使返回值变得简单，其中 `'T` 代表任务的返回类型。

### 警告

*以下示例中使用的随机数生成方法足以用于演示，但请注意，`System.Random` 类不是线程安全的，即使每个任务创建一个新的实例也可能不足以保证线程安全。如果你的解决方案需要更强大的并发随机数生成方法，建议阅读 Stephen Toub 关于该主题的文章，地址是* [`blogs.msdn.com/b/pfxteam/archive/2009/02/19/9434171.aspx`](http://blogs.msdn.com/b/pfxteam/archive/2009/02/19/9434171.aspx)。

创建返回值的任务几乎与我们已经看过的基本任务相同。`Task<'T>`类提供了一组构造函数重载，这些重载与非泛型`Task`类的构造函数类似，`TaskFactory`包括`StartNew`的泛型重载。为了演示，让我们使用`StartNew<'T>`来创建并运行一个返回随机数的任务。

```
let t = Task.Factory.StartNew(fun () -> System.Random().Next())
```

这个示例中唯一真正值得注意的地方是传递给`StartNew`的函数返回一个整数，并且泛型重载是被推断出来的。当然，返回一个值如果没有方法来访问它，是没有太大意义的，这就是为什么`Task<'T>`提供了`Result`属性，当任务完成时，它将包含返回值。在这里，我们展示了如何访问返回值：

```
**t.Result** |> printfn "Result: %i"
```

由于这是一个异步操作，因此无法保证在访问`Result`属性之前，任务已经执行完成。为此，`Result`的`get`访问器会检查任务是否已完成，并在必要时等待任务完成后再返回结果。通常，在任务开始后，立即访问结果不太常见，而是作为后续操作的一部分进行访问（如本章稍后所示）。

#### 等待任务完成

当你的程序依赖于一个或多个任务完成后才能继续处理时，你可以使用其中一种等待机制来等待这些任务。为了方便，本节中的示例将使用以下函数，该函数返回一个新函数，该函数在打印消息之前会随机睡眠一段时间（模拟一个持续时间最长为`delayMs`的长时间操作）：

```
let randomWait (delayMs : int) (msg : string) =
  fun () -> (System.Random().Next delayMs |> Task.Delay).Wait()
            Console.WriteLine msg
```

我们可以使用`TaskFactory`来创建任务，并通过任务的`Wait`方法等待它完成，如下所示：

```
let waitTask = Task.Factory.StartNew(randomWait 1000 "Task Finished")
**waitTask.Wait()**
printfn "Done Waiting"
```

在这段代码中，一个新的任务被创建并启动，但由于显式等待，直到任务完成后，消息“Done Waiting”才会被写入控制台。当后续代码依赖于任务完成时，这种方式非常有用。

你通常会希望并行运行多个任务，并在其中一个任务完成之前阻塞。为此，你可以使用`Task`类的静态`WaitAny`方法。最基本的`WaitAny`重载接受一个任务数组，并且只要数组中的任何一个任务完成，它就会停止阻塞。这里，我们将三个已启动的任务传递给`WaitAny`：

```
Task.WaitAny(
    Task.Factory.StartNew(randomWait 2000 "Task 0 Finished"),
    Task.Factory.StartNew(randomWait 2000 "Task 1 Finished"),
    Task.Factory.StartNew(randomWait 2000 "Task 2 Finished"))
Console.WriteLine "Done Waiting"
```

当三个任务中的任何一个完成时，`WaitAny`将停止阻塞，从而允许执行继续进行到`Console.WriteLine`调用。请注意，`WaitAny`在解除阻塞时不会终止剩余的任务，因此它们会继续与源线程并行执行。

与`WaitAny`类似，`Task`类提供了一个静态的`WaitAll`方法。`WaitAll`同样接受一个`params`任务数组，但与允许执行在一个任务完成时继续不同，`WaitAll`只有在*所有*任务都完成时才会解除阻塞。由于代码的区别仅在于调用了哪个方法，所以我没有包括示例，但我鼓励你尝试每种方法。在尝试时，可以多次运行每种形式并观察其差异。

#### 延续任务

传统上，每当你希望在某些并行或异步代码完成后立即执行某些代码时，你需要将一个函数（称为*回调*）传递给异步代码。在 .NET 中，回调通常通过内置的`AsyncCallback`委托类型来实现。

使用回调是有效的，但它们可能会使代码变得复杂且难以维护。TPL 通过*延续任务*大大简化了这个过程，延续任务是配置为在一个或多个任务（称为*先行任务*）完成时启动的任务。

最简单的延续任务是由单个任务创建的。我们从创建一个作为先行任务的任务开始：

```
let antecedent =
  new Task<string>(
    fun () ->
      Console.WriteLine("Started antecedent")
      System.Threading.Thread.Sleep(1000)
      Console.WriteLine("Completed antecedent")
      "Job's done")
```

现在我们有了一个任务，我们可以通过将一个函数传递给任务的`ContinueWith`方法来设置延续任务，像这样：

```
let continuation =
  antecedent.ContinueWith(
    fun ① (a : Task<string>) ->
      Console.WriteLine("Started continuation")
      Console.WriteLine("Antecedent status: {0}", a.Status)
      Console.WriteLine("Antecedent result: {0}", a.Result)
      Console.WriteLine("Completed continuation"))
```

如你所见，创建一个延续任务与创建常规任务非常相似，但请注意在①处传递给`ContinueWith`方法的函数如何接受一个类型为`Task<string>`的参数。这个参数代表先行任务，以便延续任务可以根据先行任务的状态（例如，`RanToCompletion`、`Faulted`、`Canceled`等）或其结果（如果有的话）来分支。

此时，两个任务都尚未开始，因此我们将启动`antecedent`。当它完成时，TPL 将自动启动`continuation`。我们可以通过以下方式观察这种行为：

```
antecedent.Start()
Console.WriteLine("Waiting for continuation")
continuation.Wait()
Console.WriteLine("Done")
```

应该打印以下信息：

```
Waiting for continuation
Started antecedent
Completed antecedent
Started continuation
Antecedent status: RanToCompletion
Completed continuation
Done
```

`ContinueWith`方法在你处理单个任务时非常有用。当你有多个任务时，你可以转向`TaskFactory`的`ContinueWhenAny`或`ContinueWhenAll`方法。像它们的`WaitAny`和`WaitAll`对应方法一样，`ContinueWhenAny`和`ContinueWhenAll`方法将在数组中的任何任务或所有任务完成时启动延续任务。为了简洁起见，我们将重点介绍`ContinueWhenAll`方法。

```
let antecedents =
  [|
    new Task(
        fun () ->
          Console.WriteLine("Started first antecedent")
          System.Threading.Thread.Sleep(1000)
          Console.WriteLine("Completed first antecedent"))
    new Task(
        fun () ->
          Console.WriteLine("Started second antecedent")
          System.Threading.Thread.Sleep(1250)
          Console.WriteLine("Completed second antecedent"))
    new Task(
        fun () ->
          Console.WriteLine("Started third antecedent")
          System.Threading.Thread.Sleep(1000)
          Console.WriteLine("Completed third antecedent"))
  |]

let continuation =
  ① Task.Factory.ContinueWhenAll(
    antecedents,
    fun ② (a : Task array) ->
      Console.WriteLine("Started continuation")
      for x in a do Console.WriteLine("Antecedent status: {0}", x.Status)
      Console.WriteLine("Completed continuation"))

for a in antecedents do a.Start()

Console.WriteLine("Waiting for continuation")
continuation.Wait()
Console.WriteLine("Done")
```

`ContinueWhenAny`遵循与`WaitAny`相同的模式。在这里，我们定义了三个任务，并在创建延续任务后手动启动它们，在①处创建延续任务。请注意在②处延续任务的参数。与使用`ContinueWith`或`ContinueWhenAny`时传递单个先行任务不同，使用`ContinueWhenAll`创建的延续任务接受一个任务数组。这个数组包含传递给`ContinueWhenAll`的所有任务，而不是启动延续任务的单个任务。这使你能够检查每个先行任务并根据需要细粒度地处理成功和失败的场景。

#### 取消任务

取消任务在本质上与取消并行`for`循环相同，但它需要更多的工作，因为并行`for`循环会为你处理取消的细节。以下函数演示了取消任务，并遵循了典型的取消处理模式：

```
let taskWithCancellation (cancelDelay : int) (taskDelay : int) =
① use tokenSource = new System.Threading.CancellationTokenSource(cancelDelay)
② let token = tokenSource.Token

  try
    let t =
      Task.Factory.StartNew(
        (fun () ->
         ③ token.ThrowIfCancellationRequested()
          printfn "passed cancellation check; waiting"
          System.Threading.Thread.Sleep taskDelay
         ④ token.ThrowIfCancellationRequested()),
         token)
      ⑤ t.Wait()
  with
  | ex -> printfn "%O" ex
  printfn "Done"
```

与取消并行`for`循环类似，我们首先在①创建一个`CancellationTokenSource`。为了方便起见，我们在②将该令牌绑定到一个名称，以便在任务基于的函数内引用它。在任务体内，我们首先在③调用令牌的`ThrowIfCancellationRequested`方法，该方法检查令牌的`IsCancellationRequested`属性，如果该属性返回`true`，则抛出`OperationCanceledException`。我们这样做是为了确保在任务启动时如果请求了取消，就不会执行不必要的工作。当没有抛出异常时，执行将继续进行。在④，我们再次检查取消状态，以避免任务成功完成。最后，在⑤我们等待任务完成，以便处理任务抛出的任何异常。

#### 异常处理

异常可以由任何数量的执行任务在任何时候抛出。当这种情况发生时，我们需要一种方法来捕获和处理它们。在前一节中，我们以通用方式处理了异常——通过匹配任何异常并将其写入控制台。如果你执行了`taskWithCancellation`函数，你可能注意到我们捕获的异常不是`OperationCanceledException`，而是一个包含`OperationCanceledException`的`AggregateException`。基本的异常类不太适合并行场景，因为它们只表示单一的失败。为了弥补这一点，介绍了一个新的异常类型`AggregateException`，它允许我们在一个构造体中报告一个或多个失败。

尽管你完全可以直接处理`AggregateException`，但通常你会希望在其中找到一个特定的异常。为此，`AggregateException`类提供了`Handle`方法，该方法遍历其`InnerExceptions`集合中的异常，以便你找到真正关心的异常并进行相应处理。

```
try
  raise (AggregateException(
          NotSupportedException(),
          ArgumentException(),
          AggregateException(
            ArgumentNullException(),
            NotImplementedException())))
with
| :? AggregateException as ex ->
      ex.Handle(
        ① Func<_, _>(
          function
          ② | :? AggregateException as ex1 ->
               ③ ex1.Handle(
                 Func<_, _>(
                   function
                   | :? NotImplementedException as ex2 -> printfn "%O" ex2; true
                   | _ -> true))
               true
           | _ -> true))
```

处理`AggregateException`遵循熟悉的异常处理模式：我们匹配`AggregateException`并将其绑定到名称`ex`，正如你所预期的那样。在处理程序内部，我们调用`Handle`方法①，接受一个`Func<exn, bool>`，表示提供的函数接受一个异常并返回布尔值。（为了像这里一样使用模式匹配函数，我们显式构造`Func<_, _>`实例，并让编译器推断出适当的类型参数。）在模式匹配函数②内部，我们检测是否有嵌套的`AggregateException`并在③处进行处理。在每一层，我们需要返回一个布尔值，指示特定的异常是否已处理。如果我们对任何异常返回`false`，则会抛出一个新的`AggregateException`，该异常包含未处理的异常。

处理`AggregateException`像这样可能变得相当繁琐、复杂和乏味。幸运的是，`AggregateException`提供了另一个方法`Flatten`，通过迭代`InnerExceptions`集合并递归遍历每个嵌套的`AggregateException`，来简化错误处理，构造一个新的`AggregateException`实例，该实例直接包含源异常层次结构中的所有异常。例如，我们可以修改之前的示例，使用`Flatten`来简化处理程序，如下所示：

```
try
  raise (AggregateException(
          NotSupportedException(),
          ArgumentException(),
          AggregateException(
            ArgumentNullException(),
            NotImplementedException())))
with
| :? AggregateException as ex ->
      ex.**Flatten()**.Handle(
        Func<_, _>(
          function
          | :? NotImplementedException as ex2 -> printfn "%O" ex2; true
          | _ -> true))
```

在这个修改后的示例中，我们对已展平的`AggregateException`调用`Handle`。由于只有一层需要处理，我们可以省略对嵌套`AggregateExceptions`的检查，直接处理`NotImplementedException`。

## 异步工作流

尽管 TPL 为异步和并行编程带来了许多改进，但 F#提供了自己的模型，这种模型更好地匹配了语言强调的函数式范式。虽然有时在 F#中使用 TPL 是可取的（特别是在跨语言边界工作时），但你通常会转向 F#的异步工作流，它们最适合 I/O 操作。

*异步工作流*提供了一种统一且符合习惯的方式，用于在线程池上组合和执行异步代码。此外，它们的特性通常使得我们很难（如果不是不可能的话）陷入即使在 TPL 中也存在的某些异步陷阱。

### 注意

*就像我们的 TPL 讨论一样，本节旨在为你提供异步工作流的基本工作知识，而不是作为一个全面的指南。*

### 创建和启动异步工作流

异步工作流基于位于`Microsoft.FSharp.Control`命名空间中的`Async<'T>`类。该类型表示你希望异步运行的一段代码，最终返回某个值。不过，不是直接创建`Async<'T>`实例，我们通过异步表达式来组合它们，就像我们组合序列或查询一样。

异步表达式采用以下形式：

`async {` *异步表达式* `}`

在这里，*async-expressions* 代表一个或多个将参与异步操作的表达式。除了我们在本书中看到的标准表达式外，异步工作流允许你轻松地调用额外的工作流，并等待结果而不阻塞，通过一些熟悉的关键字如 `let` 和 `use` 的特殊变体。例如，`let!` 关键字调用一个异步工作流，并将结果绑定到一个名称。类似地，`use!` 关键字调用一个异步工作流，该工作流返回一个可处置对象，将结果绑定到一个名称，并在超出作用域时处置该对象。还可以使用 `return!` 关键字调用一个异步工作流并立即返回结果。

为了演示，我们将使用异步工作流的“hello world”示例：请求多个网页。首先，让我们定义一些函数来封装创建异步页面请求所需的逻辑（请注意，在 `FSharp.Data` 框架中有一个类似的函数 `Http.AsyncRequestString`）：

```
open System
open System.IO
open System.Net

type StreamReader with
  member x.AsyncReadToEnd () =
    async { do! Async.SwitchToNewThread()
            let content = x.ReadToEnd()
            do! Async.SwitchToThreadPool()
            return content }

let getPage (uri : Uri) =
  async {

  let req = WebRequest.Create uri
  use! response = req.AsyncGetResponse()
  use stream = response.GetResponseStream()
  use reader = new StreamReader(stream)
  return! reader.AsyncReadToEnd()
}
```

在打开相关命名空间之后，我们通过单个 `AsyncReadToEnd` 方法扩展了 `StreamReader` 类。这个方法来自 F# PowerPack，类似于现有的 `ReadToEndAsync` 方法，不同之处在于，它并没有使用 TPL，而是返回一个异步工作流，我们可以在描述如何发起页面请求的 `getPage` 函数的最终步骤中进行评估。整体表达式的流程非常标准：创建一个 `WebRequest`，等待响应，然后显式返回响应流的内容。

### 注意

*`AsyncGetResponseMethod` 是 F# 核心库中定义的一个扩展方法。它方便地将标准 .NET 代码包装在另一个异步工作流中，这使得使用 `use!` 成为可能，并大大简化了代码。*

重要的是要认识到，`getPage` 实际上并不执行请求；它仅仅创建了一个表示请求的 `Async<string>` 实例。这使我们能够定义多个请求，或者将它们传递给其他函数。我们甚至可以多次执行请求。要执行请求，我们需要转向静态的 `Async` 类，你可以将其视为异步工作流的控制器。

启动异步工作流有多种方法。一些常见的方法列在表 11-1 中。

表 11-1. 常见的异步启动方法

| 方法 | 描述 |
| --- | --- |
| `RunSynchronously` | 启动异步工作流并等待其结果。 |
| `Start` | 启动异步工作流，但不等待结果。 |
| `StartImmediate` | 使用当前线程立即启动异步工作流。适用于 UI 更新。 |
| `StartWithContinuations` | 立即使用当前线程启动一个异步工作流，根据操作完成的情况调用成功、异常或取消的延续。 |

你选择的方法主要取决于工作流的具体任务，但通常情况下，除非应用程序需要其他方法，否则你会使用`Start`。由`getPage`函数创建的工作流返回的是一个网页请求的结果。由于我们在发起请求，通常我们不希望忽略结果，因此需要通过延续来处理结果。最简单的方法是将`getPage`的调用包装在另一个异步表达式中，当它完成时将结果传递给另一个函数，并使用`Start`启动整个工作流。在这里，我们调用`getPage`并打印结果：

```
async {
  let! content = Uri "http://nostarch.com" |> getPage
  content.Substring(0, 50) |> printfn "%s" }
|> Async.Start
```

使用 Async

`Async`是一个静态类而不是一个模块，这对你与其交互的方式有一定影响。与模块提供`let`绑定的函数不同，`Async`提供方法，其中许多方法是重载的，主要是为了帮助取消操作。此外，`Async`的方法通常采用面向对象的方法设计，这与核心 F#库中常见的设计方式不同。因此，它们的参数通常是元组，这使得使用管道操作时会比较困难。

另外，我们可以使用`StartWithContinuations`方法，该方法接受一个异步工作流以及三个函数，分别在工作流成功完成、抛出异常或被取消时调用。以下代码展示了这种方法：

```
Async.StartWithContinuations(
  ① getPage(Uri "http://nostarch.com"),
  ② (fun c -> c.Substring(0, 50) |> printfn "%s..."),
  ③ (printfn "Exception: %O"),
  ④ (fun _ -> printfn "Cancelled")
)
```

当异步操作①成功完成时，成功的延续②将被调用，并且页面源代码的前 50 个字符将被打印。如果操作抛出异常，异常延续③将被执行，并打印异常信息。最后，如果操作被取消，正如在取消异步工作流中所描述的，取消延续④将被执行，并显示一条通知用户操作已取消的信息。

我们也可以不依赖延续，而是使用`RunSynchronously`方法直接获取结果，如下所示：

```
let html =
  Uri "http://nostarch.com"
  |> getPage
  |> Async.RunSynchronously
```

当然，像这样运行一个单一的异步工作流实际上违背了异步运行的初衷，因为`RunSynchronously`会等待结果。相反，`RunSynchronously`通常与`Async.Parallel`一起使用，用于并行运行多个工作流并等待它们全部完成。例如，我们可以通过一个异步工作流数组来发起多个请求，如下所示：

```
open System.Text.RegularExpressions

[| getPage(Uri "http://nostarch.com")
   getPage(Uri "http://microsoft.com")
   getPage(Uri "http://fsharp.org") |]
|> Async.Parallel
|> Async.RunSynchronously
|> Seq.iter (fun c -> let sample = c.Substring(0, 50)
                      Regex.Replace(sample, @"[\r\n]| {2,}", "")
                      |> printfn "%s...")
```

在这里，我们使用 `Parallel` 方法将每个异步工作流合并成一个单一的工作流，然后将其传递给 `RunSynchronously` 方法。当每个请求完成时，我们遍历结果数组，去除一些字符以便于阅读，并打印结果。

### 取消异步工作流

在上一节中，我提到过异步工作流可以被取消。就像在 TPL 中一样，异步工作流使用取消令牌来控制取消。你可以自己管理令牌，这在某些情况下甚至是必要的，但在很多情况下，你可以依赖于 `Async` 类的默认令牌。

对于简单的场景，例如当你通过 `Start` 或 `StartWithContinuations` 方法启动单个工作流时，你可以使用 `CancelDefaultToken` 方法来取消工作流，如下所示：

```
① Async.StartWithContinuations(
    getPage(Uri "http://nostarch.com"),
    (fun c -> c.Substring(0, 50) |> printfn "%s..."),
    (printfn "Exception: %O"),
    (fun _ -> printfn "Cancelled")
  )

② Async.CancelDefaultToken()
```

`StartWithContinuations` 方法① 监视默认令牌，并在通过 `CancelDefaultToken` 方法② 标记令牌为已取消时取消工作流。在此示例中，由于工作流在完成之前被取消，因此会调用取消回调，而不是成功回调，导致显示取消消息。

`TryCancelled` 方法接受一个工作流和一个在请求取消时将被调用的函数，这是一个很好的替代方案，适用于不返回值的工作流。在这里，`displayPartialPage` 函数将对 `getPage` 的调用包装在另一个异步工作流中。外部工作流等待响应并在收到消息时输出前 50 个字符。由于 `TryCancelled` 返回另一个工作流，并且不会自动启动它，我们需要通过调用 `Start` 显式启动它。

```
let displayPartialPage uri =
  Async.TryCancelled(
    async {
      let! c = getPage uri
      Regex.Replace(c.Substring(0, 50), @"[\r\n]| {2,}", "")
      |> sprintf "[%O] %s..." uri
      |> Console.WriteLine },
    (sprintf "[%O] Cancelled: %O" uri >> Console.WriteLine))

Async.Start(displayPartialPage (Uri "http://nostarch.com"))

Async.CancelDefaultToken()
```

默认令牌通常足以取消工作流。当你执行多个工作流并希望协调取消时，或者如果你希望对取消有更多控制时，你可以提供自己的令牌。考虑一下，当你请求三个页面并使用默认令牌请求取消时，会发生什么。

```
[| Uri "http://nostarch.com"
   Uri "http://microsoft.com"
   Uri "http://fsharp.org" |]
|> Array.iter (fun u -> Async.Start(displayPartialPage u))

Async.CancelDefaultToken()
```

执行上述代码通常会导致所有三个工作流被取消。（通常是这样，但并非总是如此，因为有可能一个或多个工作流在取消处理之前就完成了。）为了隔离每个工作流的取消，我们可以使用一个重载的 `Start` 方法，接受用户指定的令牌，如下所示：

```
  open System.Threading

  let tokens =
    [| Uri "http://nostarch.com"
       Uri "http://didacticcode.com"
       Uri "http://fsharp.org" |]
    |> Array.map (fun u -> ① let ts = new CancellationTokenSource()
                           Async.Start(displayPartialPage u, ② ts.Token)
                           ts)
③ tokens.[0].Cancel()
④ tokens.[1].Cancel()
```

在这个修改版中，我们使用 `Array.map` 将每个 `Uri` 映射到一个具有自己 `CancellationTokenSource` 的工作流，该源在①处创建。然后，我们将相关的令牌作为第二个参数传递给 `Async.Start`②，最后返回 `CancellationTokenSource`。最后，在③和④处，我们分别请求取消第一个和第二个请求，允许第三个请求正常进行。

取消异步工作流的一个特别好的地方在于，与 TPL 不同，取消令牌会自动传播到整个工作流。这意味着你无需手动确保每个新工作流都获得一个令牌，从而使代码更加简洁。

### 异常处理

由于异常可能在异步工作流中发生，因此了解如何正确处理它们是非常重要的。有几种异常处理选项可供选择，但它们的有效性可能会根据你的具体操作有所不同。

处理异步工作流中异常的最统一方式是将潜在有问题的代码包装在异步表达式中的 `try...with` 块内。例如，我们可以提供一个版本的 `getPage` 函数来处理页面请求和读取过程中抛出的异常，如下所示：

```
let getPageSafe uri =
  async {
    try
      let! content = getPage uri
      return Some content
    with
    | :? NotSupportedException as ex ->
      Console.WriteLine "Caught NotSupportedException"
      return None
    | :? OutOfMemoryException as ex ->
      Console.WriteLine "Caught OutOfMemoryException"
      return None
    | ex ->
      ex |> sprintf "Caught general exception: %O" |> Console.WriteLine
      return None }
```

上述代码中的 `try...with` 块没有什么异常——我们只是将对 `getPage` 的异步调用包装在 `try...with` 块中，并将成功读取的结果作为一个选项返回。如果操作抛出异常，我们匹配异常类型，打印消息，并返回 `None`。

处理异步工作流异常的另一种方式是使用 `Async.Catch` 方法。与 `StartWithContinuations` 相比，`Async.Catch` 采用了更具函数式风格的方式：它返回 `Choice<'T, exn>`，其中 `'T` 是异步工作流的返回类型，`exn` 是工作流抛出的异常，而不是接受一个异常处理函数。

`Choice` 类型是一个带有两个联合情况的区分联合类型：`Choice1Of2` 和 `Choice2Of2`。对于 `Async.Catch`，`Choice1Of2` 代表工作流的成功完成并包含结果，而 `Choice2Of2` 代表失败并包含第一个抛出的异常。

使用 `Async.Catch` 处理异常使你能够结构化异步代码，创建符合惯用法的管道化数据流。例如，以下代码展示了如何将异步操作建模为一系列函数应用，从一个 `Uri` 开始。

```
Uri "http://nostarch.com"
|> getPage
|> Async.Catch
|> Async.RunSynchronously
|> function
   | Choice1Of2 result -> Some result
   | Choice2Of2 ex ->
      match ex with
      | :? NotSupportedException ->
        Console.WriteLine "Caught NotSupportedException"
      | :? OutOfMemoryException ->
        Console.WriteLine "Caught OutOfMemoryException"
      | ex ->
        ex.Message |> sprintf "Exception: %s" |> Console.WriteLine
      None
```

在这里，一个 `Uri` 被传递到 `getPage` 函数以创建一个异步工作流。生成的工作流被传递到 `Async.Catch` 中，设置另一个工作流，然后我们将其传递到 `Async.RunSynchronously`，以便等待结果。最后，我们将 `Choice` 传递给模式匹配函数，在该函数中我们要么返回 `Some result`，要么处理异常后返回 `None`。

### 异步工作流与任务并行库

除了我们迄今为止看到的基于 `ThreadPool` 的异步操作外，`Async` 类还提供了几个用于处理 TPL 任务的方法。其中最显著的是 `StartAsTask` 和 `AwaitTask`。

`StartAsTask` 方法作为 TPL 任务调用一个异步工作流。通常，你会在 CPU 密集型操作或者需要将异步工作流暴露给使用 TPL 的 C# 或 Visual Basic 代码时使用它。例如，我们可以像这样将 `getPage` 函数的结果视为 TPL 任务：

```
Uri "http://nostarch.com"
|> getPage
|> Async.StartAsTask
|> (fun t -> ① t.Result.Substring(0, 50))
|> printfn "%s"
```

① 处存在 `Result` 属性，表明 `StartAsTask` 的结果确实是一个 `Task`。在更现实的场景中，你可能不会启动一个任务后立刻阻塞等待结果，但这个示例仅用于展示如何将异步工作流作为 TPL `Task` 启动。

`StartAsTask` 方法在你需要创建新任务时非常方便，但如果你需要处理一个已有的任务怎么办？考虑一下在 .NET 4.5 中添加到 `System.Net.WebClient` 类中的 `DownloadStringTaskAsync` 方法。这个方法与我们的 `getPage` 函数的作用相同，区别在于它将下载资源封装在 TPL 任务中。

在 C# 中，你可以像这样通过 `async` 修饰符和 `await` 操作符轻松处理此类方法：

```
// C#
// using System.Threading.Tasks

private static ① async Task<string> GetPageAsync(string uri)
{
    using (var client = new System.Net.WebClient())
    {
      return ② await client.DownloadStringTaskAsync(uri);
    }
}

static void Main()
{
    var result = GetPageAsync("http://nostarch.com").Result;
    Console.WriteLine("{0}", result.Substring(0, 50));
    Console.ReadLine();
}
```

从一个大大简化的角度来看，在前面的 C# 代码中发生的事情是这样的：`async` 修饰符①被应用于 `GetPageAsync` 方法，表示方法的一部分将异步执行。接着，`await` 操作符②表示执行应该返回给调用者，方法的其余部分应作为一个后续操作，待任务完成时执行。

异步工作流使我们能够在 F# 中使用 `AwaitTask` 方法结合 TPL 任务和 `let!`、`use!` 或 `return!` 跟随类似的模式。下面是 F# 中的对应代码：

```
// F#
open System.Threading.Tasks

let getPageAsync (uri : string) =
  async {
    use client = new System.Net.WebClient()
  ① return! Async.AwaitTask (client.DownloadStringTaskAsync uri)
  }

async {
② let! result = getPageAsync "http://nostarch.com"
  result.Substring(0, 50) |> printfn "%s"
} |> Async.Start
```

虽然它们在功能上并不完全等价（C# 版本会在 `Main` 中等待结果，而 F# 版本则将结果传递给一个后续操作），但 F# 方法与 C# 的方法类似。在 F# 版本中，通过 `getPageAsync` 函数创建的异步工作流使用 `return!` 和 `Async.AwaitTask` ① 来等待任务完成后返回结果。然后，在第二个异步工作流中，`let!` ② 用于求值 `getPageAsync`，同时打印结果被视为一个后续操作。

## 基于代理的编程

如果说 TPL 和异步工作流还不足以让并行和异步编程变得足够易用，F# 还借用了来自 Erlang 的消息处理机制。`MailboxProcessor<'T>` 类实现了一个基于队列的系统，用于通过共享内存异步路由消息（数据项）到处理器。这在多个源（客户端）需要从单一目标（服务器）请求某些内容的场景中非常有用，经典的例子就是 web 服务器。此外，由于 `MailboxProcessor` 实例极其轻量化，应用程序可以管理成千上万个实例而不至于出现问题。这一特性使得邮件处理器能够独立工作或通过实例之间传递消息共同工作。

`MailboxProcessor`实例通常被称为*代理*，我将在本节中遵循这一惯例。在这方面，基于代理的编程中常见的做法是将`MailboxProcessor<'T>`别名为`Agent<'T>`，如下所示：

```
type Agent<'T> = MailboxProcessor<'T>
```

通过类型别名化，我们可以使用更方便的名称创建代理。

### 入门

我认为理解基于代理的编程的最佳方式是通过示例。我们从一个简单的代理开始，它会打印传递给它的任何内容。

```
type Message = | Message of obj

let echoAgent =
① Agent<Message>.Start(
    fun inbox ->
    ② let rec loop () =
        async {
          let! (Message(content)) =  ③ inbox.Receive()
          printfn "%O" content
        ④ return! loop()}
    ⑤ loop())
```

在前面的代码中，我们通过将一个函数传递给`Start`方法创建了一个名为`echoAgent`的代理，如①所示。根据惯例，函数的参数被命名为`inbox`，因为它是我们接收新消息的*邮箱*。在②处，我们定义了递归的`loop`函数，我们将不断调用该函数来接收新消息。

### 注意

*使用 while 循环来进行命令式循环当然是可能的，但递归函数是更典型的方法。函数式循环提供了额外的好处，当你需要管理多个状态时，它使得提供不同的循环逻辑变得更加容易。例如，如果你的代理在暂停状态下需要与运行状态下有不同的行为，你可以定义一对互相递归的函数，它们都返回一个工作流，根据相应的状态进行处理。*

在循环内部，我们创建了一个异步工作流，首先使用`Receive`方法异步地从`inbox`接收消息，如③所示。接下来，接收到的消息会被打印出来，然后在④处进行异步递归调用`loop`。最后，在⑤处，我们通过进行标准的同步调用来启动递归，调用`loop`。

在`echoAgent`处于主动监听状态时，我们可以通过`Post`方法向它发送一些消息，如下所示：

```
> **Message "nuqneH" |> echoAgent.Post;;**
nuqneH
> **Message 123 |> echoAgent.Post;;**
123
> **Message [ 1; 2; 3 ] |> echoAgent.Post;;**
[1; 2; 3]
```

如你所见，当`echoAgent`接收到一条消息时，它会将其写入控制台，然后`echoAgent`等待另一条消息，整个过程不断重复。

### 扫描消息

在`echoAgent`示例中，我们使用`Receive`方法从底层队列获取消息。在许多情况下，`Receive`方法是合适的，但它会将消息从队列中移除，导致难以过滤消息。为了有选择性地处理消息，你可以考虑改用`Scan`方法。

扫描消息的方式与直接接收消息的方式不同。`Scan`方法不是直接内联处理消息并始终返回异步工作流，而是接受一个过滤函数，该函数接受一条消息并返回`Async<'T>`选项。换句话说，当消息是你想处理的内容时，你返回`Some<Async<'T>>`；否则，返回`None`。为了演示，我们将修改`echoAgent`，只处理字符串和整数。

```
let echoAgent2 =
  Agent<Message>.Start(fun inbox ->
    let rec loop () =
      inbox.Scan(fun (Message(x)) ->
       match x with
       | ① :? string
       | ② :? int ->
         Some (async { printfn "%O" x
                       return! loop() })
       | _ -> printfn "<not handled>"; None)
   loop())
```

在①和②处，你可以看到标准的动态类型测试模式，用来分别将传入的消息过滤为字符串和整数。当消息是这两种类型之一时，我们会将一个异步工作流与`Some`关联并返回它。对于所有其他消息，我们返回`None`。`Scan`检查返回的值，当它是`Some`时，消息被消费（从队列中移除）并触发工作流。返回值是`None`时，`Scan`立即等待另一个消息。

向`echoAgent2`传递消息和之前一样——只需通过`Post`方法传递消息：

```
> **Message "nuqneH" |> echoAgent2.Post;;**
nuqneH
> **Message 123 |> echoAgent2.Post;;**
123
> **Message [ 1; 2; 3 ] |> echoAgent2.Post;;**
<not handled>
```

扫描消息确实提供了一些处理消息的灵活性，但你需要注意你向代理发布了什么，因为`Scan`未处理的消息会保留在队列中。随着队列大小的增加，扫描将需要更长的时间，因此，如果不小心使用这种方法，你可能会很快遇到性能问题。你可以通过检查`CurrentQueueLength`属性来查看队列中有多少消息。如果你需要从队列中移除消息，可以通过调用`Receive`来处理队列中的每条消息，但如果你需要这样做，这可能意味着一个更大的设计问题需要解决。

### 回复消息

到目前为止，我们创建的代理都是自包含的：它们接收消息，处理消息，然后等待下一个消息。不过，代理不必在孤立中工作。有一种方法可以使代理更加互动，那就是让它们通过`AsyncReplyChannel`来回复。为了演示这一点，我们再次修改`echoAgent`，这次，我们不再在代理内部打印消息，而是让它进行回复。

```
① type ReplyMessage = | ReplyMessage of obj * AsyncReplyChannel<obj>

  let echoAgent3 =
    Agent.Start(fun inbox ->
      let rec loop () =
        async {
          let! ② (ReplyMessage(m, c)) = inbox.Receive()
          ③ c.Reply m
          return! loop()
      }
    loop())
```

`echoAgent3`的整体结构与之前的版本差别不大。为了方便起见，我们使用了一个判别联合类型①作为我们的消息类型，这在基于代理的编程中是典型的做法。在这种情况下，`ReplyMessage`联合类型有一个单一的案例，包含两个关联值，一个对象和一个回复通道。

在循环体内，我们使用模式匹配②来识别联合案例并提取消息和通道。然后，我们将消息传递给通道的`Reply`方法③，然后再继续循环。现在剩下的就是向代理发送消息了。

`ReplyMessage`的第二个值是一个`AsyncReplyChannel<obj>`，正如你已经看到的那样。理论上，我们可以手动构建一个回复通道，并使用`Post`方法将`ReplyMessage`发送给代理，但那样我们就必须手动处理等待结果的过程。获取回复通道有更好的方法——即`PostAndReply`方法及其变种。

`PostAndReply`方法与`Post`方法略有不同，它们并不直接接受消息，而是高阶函数，接受一个函数，该函数接收一个预构建的回复通道并返回完全构建好的方法。为了方便起见，我们会简单地创建一个`ReplyMessage`，如下所示：

```
echoAgent3.PostAndReply(fun c -> ReplyMessage("hello", c))
|> printfn "Response: %O"
```

在内部，`PostAndReply`（及其变体）构建回复通道，并将其传递给提供的函数，该函数随后创建最终发布到代理的消息。

### 示例：基于代理的计算器

现在，您已经看到了多种创建和与代理交互的方式，让我们来看看一个更有趣的示例，将多个概念结合在一起，做一些比简单重复输入更有用的事情：一个基于代理的计算器。我们将从定义一个代表计算器将支持的消息的区分联合类型开始。

```
type Operation =
| Add of float
| Subtract of float
| Multiply of float
| Divide of float
| Clear
| Current of AsyncReplyChannel<float>
```

`Operation`联合类型定义了六个案例。在这些案例中，四个表示基本的数学操作，并且有一个与之关联的`float`值用于计算。`Clear`案例允许我们清除存储的值。最后，`Current`案例允许我们通过其关联的回复通道查询代理的当前值。从这个定义中，我们可以创建一个新的代理来处理每个案例，如下所示：

```
let calcAgent =
  Agent.Start(fun inbox ->
   let rec loop total =
     async {
       let! msg = inbox.Receive()
       let newValue =
         match msg with
         | Add x -> total + x
         | Subtract x -> total - x
         | Multiply x -> total * x
         | Divide x -> total / x
         | Clear -> 0.0
         | Current channel ->
           channel.Reply total
           total
       return! loop newValue }
   loop 0.0)
```

即使`calcAgent`看起来保持了一个运行中的总和，它实际上有点是一个假象，因为我们仅通过将一个值（`total`）传递给递归的`loop`函数来保持状态。当`calcAgent`收到消息时，它使用模式匹配来确定适当的操作，并将结果绑定到`newValue`。例如，当它收到`Add`、`Subtract`、`Multiply`或`Divide`操作时，它会对`total`应用相应的数学操作。同样，当它收到`Clear`操作时，它仅返回`0.0`，而`Current`在回复后返回`total`。

要查看`calcAgent`的实际操作，我们只需要发送一些消息给它：

```
[ Add 10.0
  Subtract 5.0
  Multiply 10.0
  Divide 2.0 ]
|> List.iter (calcAgent.Post)

calcAgent.PostAndReply(Current) |> printfn "Result: %f"
calcAgent.Post(Clear)
calcAgent.PostAndReply(Current) |> printfn "Result: %f"
```

在前面的代码片段中，我们简单地将一个`Operations`列表传递给`List.iter`，并将每个消息发送到`calcAgent`。当这些消息被处理后，我们查询当前值，清空，然后再次查询以确保总数已经归零。调用前面的代码片段会产生以下结果：

```
Result: 25.000000
Result: 0.000000
```

## 总结

异步和并行编程长期以来被视为专门用于特定软件的工具，仅限于经验丰富的开发者。随着处理器制造商通过增加核心而不是提高时钟速度来提升处理器性能，软件开发者不再能够仅通过升级硬件来解决性能问题，也无法继续期待用户在长时间运行的操作完成之前一直等待返回控制。

像 F# 这样的语言通过提供多种强大的机制，使得异步和并行编程变得更加易于接触。TPL（任务并行库）使开发者能够高效地处理 CPU 密集型操作，如处理大型数据集，同时有效利用系统资源。像异步工作流这样的语言特性，能够在 IO 密集型操作（例如 Web 请求或文件访问）期间保持应用程序的响应性。最后，基于代理的编程让你可以轻松协调复杂的系统，通过启动独立的异步进程，而无需直接管理传统线程模型的复杂性。通过这些方法，你可以构建出可扩展、响应迅速的应用程序，满足现代计算的需求，同时让你专注于软件要解决的实际问题。
