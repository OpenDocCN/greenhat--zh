# 第八章：异步编程

![](img/chapterart.png)

异步编程，顾名思义，就是不按同步方式执行的编程。高层次地说，异步操作是在后台执行的——程序不会等待异步操作完成，而是会立即继续执行下一行代码。如果你还不熟悉异步编程，可能会觉得这个定义不够充分，因为它并没有真正解释异步编程*是什么*。为了真正理解异步编程模型以及它在 Rust 中是如何工作的，我们首先必须了解它的替代模型。也就是说，我们需要理解*同步*编程模型，才能更好地理解*异步*编程模型。这一点很重要，不仅能帮助我们理清概念，还能展示使用异步编程的权衡：异步方案并不总是正确的选择！我们将在本章开始时，快速回顾一下为什么异步编程作为一个概念会被提出；然后，我们将深入探讨 Rust 中异步操作的实现原理。

## 异步操作到底是怎么回事？

在深入了解同步和异步编程模型的细节之前，我们首先需要快速了解一下计算机在运行程序时究竟在做什么。

计算机很快。真的很快。事实上，它们的速度快到大部分时间都在等待事情发生。除非你在解压文件、编码音频或进行复杂的计算，否则你的 CPU 很可能大部分时间都是空闲的，在等待操作完成。它在等待网络数据包的到达，等待鼠标的移动，等待磁盘写入某些字节，甚至可能只是在等待从主内存中读取的数据完成。从 CPU 的角度来看，大多数此类事件之间仿佛过了很长时间。当某个事件发生时，CPU 执行一些指令，然后又回到等待状态。看看你的 CPU 使用率——它可能一直徘徊在个位数的低值，这也是大多数时间它的工作状态。

### 同步接口

同步接口允许你的程序（或者更准确地说，程序中的单个线程）一次只能执行一个操作；每个操作必须等待前一个同步操作完成后才能运行。大多数你在实际开发中遇到的接口都是同步的：你调用它们，它们做一些事情，最终在操作完成后返回，程序才能继续运行。我们稍后在本章中将看到，这背后的原因是：使操作变为异步需要一些额外的机制。除非你需要异步的好处，否则坚持同步模型会少一些繁琐的步骤和复杂的情况。

同步接口隐藏了所有这些等待；应用程序调用一个函数，要求“将这些字节写入这个文件”，过一段时间后，那个函数完成，下一行代码执行。在幕后，实际上发生的是，操作系统将写操作排队到磁盘，并将应用程序挂起，直到磁盘报告写入完成。应用程序体验到的是该函数执行时间很长，但实际上并没有执行，只是处于等待状态。

以这种方式按顺序执行操作的接口通常也被称为*阻塞*，因为接口中的操作必须等待某个外部事件发生，才能继续执行，而这个事件的发生*阻塞*了后续的执行。无论你是称接口为同步还是阻塞，基本的概念是相同的：应用程序不会继续进行，直到当前操作完成。在等待操作时，应用程序也在等待。

同步接口通常被认为易于使用且容易推理，因为你的代码一次只执行一行。但它们也只能让应用程序一次执行一项任务。这意味着，如果你希望程序等待用户输入或网络数据包，你就没戏了，除非操作系统提供了专门的操作来处理这种情况。同样，即使你的应用程序在磁盘写文件时可以做一些其他有用的工作，它也没有这个选择，因为文件写操作会阻塞执行！

### 多线程

到目前为止，允许并发执行的最常见解决方案是使用*多线程*。在一个多线程程序中，每个线程负责执行一系列独立的阻塞操作，操作系统在各个线程之间进行多路复用，以便如果有线程可以继续执行，就会继续执行。如果某个线程被阻塞，其他线程可能仍然可以运行，因此应用程序可以继续进行有用的工作。

通常，这些线程通过使用像锁或通道这样的同步原语相互通信，从而使应用程序仍然能够协调它们的工作。例如，你可能会有一个线程等待用户输入，一个线程等待网络数据包，另一个线程等待这两个线程中的任何一个在线程之间共享的通道上发送消息。

多线程为你提供了*并发*——能够在任何时刻执行多个独立操作的能力。由运行该应用程序的系统（在本例中是操作系统）决定在没有被阻塞的线程中选择哪个线程执行，并决定接下来执行哪个线程。如果某个线程被阻塞，系统可以选择运行另一个可以继续执行的线程。

多线程结合阻塞接口能够让你走得很远，许多生产级软件都是以这种方式构建的。但是这种方法也有它的缺点。首先，快速跟踪所有这些线程会变得繁琐；如果你必须为每一个并发任务创建一个线程，包括像等待键盘输入这样简单的任务，线程会迅速增加，管理这些线程之间的交互、通信和协调的复杂性也随之增加。

其次，线程之间的切换成本随着线程数量的增加而增加。每当一个线程停止运行，另一个线程接替运行时，你需要向操作系统调度器做一次回调，而这不是免费的。在某些平台上，创建新线程也是一个相对重量级的过程。高性能需求的应用程序通常通过重用线程和使用允许你在多个相关操作上进行阻塞的操作系统调用来减轻这一成本，但最终你还是会面临同样的问题：阻塞接口要求你有与要进行的阻塞调用数量相同的线程。

最后，线程为你的程序引入了*并行性*。并发性和并行性的区别微妙但重要：并发性意味着你的任务执行是交替进行的，而并行性意味着多个任务同时执行。如果你有两个任务，它们的执行用 ASCII 表示可能像 `_-_-_`（并发性）与 `=====`（并行性）这样。多线程并不一定意味着并行性——即使你有许多线程，可能只有一个核心，这样只有一个线程在某一时刻执行——但两者通常是密切相关的。你可以通过使用 `Mutex` 或其他同步原语来使两个线程的执行互斥，但这会引入额外的复杂性——线程希望并行运行。并行性通常是一个好事——谁不希望他们的程序在更多核心上运行得更快呢——但这也意味着你的程序必须处理对共享数据结构的真正并发访问。这意味着从 `Rc`、`Cell` 和 `RefCell` 转向更强大但也更慢的 `Arc` 和 `Mutex`。虽然你*可能*想在并发程序中使用后者类型来启用并行性，但线程*迫使*你使用它们。我们将在第十章详细探讨多线程。

### 异步接口

现在我们已经探讨了同步接口，接下来可以看看它的替代方案：异步或*非阻塞*接口。异步接口是指可能不会立刻返回结果，而是可能表示结果会在稍后的某个时间可用。这让调用者有机会在此期间做其他事情，而不必等到特定操作完成才“睡眠”。在 Rust 的术语中，异步接口是返回`Poll`的方法，如示例 8-1 所定义。

```
enum Poll<T> {
    Ready(T),
    Pending
}
```

示例 8-1：异步的核心：“你现在就可以得到，或者稍后再来”的类型

`Poll` 通常出现在以 `poll` 开头的函数的返回类型中——这些方法表示它们可以在不阻塞的情况下尝试某个操作。我们将在本章后面详细讨论它们是如何做到这一点的，但一般来说，它们会在通常会阻塞之前尽可能多地执行操作，然后返回。而且关键是，它们会记住它们暂停的位置，以便当有进一步进展时，可以恢复执行。

这些非阻塞函数使我们能够轻松地并发执行多个任务。例如，如果你想从网络或用户的键盘中读取数据，哪个先有事件可用，你只需要在循环中轮询两个，直到其中一个返回 `Poll::Ready`。不需要任何额外的线程或同步！

这里的*循环*这个词应该让你有点紧张。你可不希望你的程序每秒执行三十亿次循环，而可能还要等几分钟才有下一个输入发生。在阻塞接口的世界里，这不是问题，因为操作系统会简单地让线程进入休眠，然后在发生相关事件时唤醒它，但在这个全新的非阻塞世界里，我们该如何避免在等待时浪费 CPU 周期呢？这正是本章其余部分要讲解的内容。

### 标准化轮询

为了实现一个可以以非阻塞方式使用每个库的世界，我们可以让每个库的作者编写他们自己的 `poll` 方法，虽然这些方法的名字、签名和返回类型可能略有不同，但那样很快就会变得难以管理。相反，在 Rust 中，轮询通过 `Future` 特征来标准化。`Future` 的简化版本如示例 8-2 所示（我们将在本章后面介绍真正的版本）。

```
trait Future {
    type Output;
    fn poll(&mut self) -> Poll<Self::Output>;
}
```

示例 8-2：`Future` 特征的简化视图

实现了 `Future` 特征的类型被称为 *futures*，代表那些可能还不可用的值。一个 future 可能代表下次网络数据包的到来、下次鼠标光标的移动，或者只是某段时间过去的时刻。你可以将 `Future<Output = Foo>` 理解为“一个将来会生成 `Foo` 类型的类型”。这种类型在其他语言中通常被称为 *promises*——它们承诺最终会返回指定的类型。当一个 future 最终返回 `Poll::Ready(T)` 时，我们说这个 future *解析* 为 `T`。

有了这个特征，我们可以将提供 `poll` 方法的模式泛化。我们不需要像 `poll_recv` 和 `poll_keypress` 这样的特定方法，而可以使用像 `recv` 和 `keypress` 这样的方法，它们都会返回 `impl Future` 和适当的 `Output` 类型。这并不改变你必须轮询它们的事实——我们稍后会处理这个问题——但它至少意味着这些挂起值有了标准化的接口，我们不需要到处使用 `poll_` 前缀。

## 人体工程学未来

以我所描述的方式编写一个实现 `Future` 的类型相当麻烦。为了理解为什么，首先来看一下 列表 8-3 中那个相当简单的异步代码块，它只是尝试将输入通道 `rx` 中的消息转发到输出通道 `tx`。

```
async fn forward<T>(rx: Receiver<T>, tx: Sender<T>) {
    while let Some(t) = rx.next().await {
        tx.send(t).await;
    }
}
```

列表 8-3：使用 `async` 和 `await` 实现一个通道转发的未来

这段代码使用 `async` 和 await 语法编写，看起来与其同步代码非常相似，且易于阅读。我们只是简单地在一个循环中发送每条接收到的消息，直到没有更多消息，每个 `await` 点对应一个同步变体可能会阻塞的地方。现在想象一下，如果你必须手动实现 `Future` 特征来表达这段代码会怎样。由于每次调用 `poll` 都从函数顶部开始，你需要打包必要的状态，以便从上次代码暂停的地方继续执行。结果相当难看，正如 列表 8-4 所展示的那样。

```
enum Forward<T> { 1 
    WaitingForReceive(ReceiveFuture<T>, Option<Sender<T>>),
    WaitingForSend(SendFuture<T>, Option<Receiver<T>>),
}

impl<T> Future for Forward<T> {
    type Output = (); 2 
    fn poll(&mut self) -> Poll<Self::Output> {
        match self { 3 
            Forward::WaitingForReceive(recv, tx) => {
                if let Poll::Ready((rx, v)) = recv.poll() {
                    if let Some(v) = v {
                        let tx = tx.take().unwrap(); 4 
                        *self = Forward::WaitingForSend(tx.send(v), Some(rx)); 5 
                        // Try to make progress on sending.
                        return self.poll(); 6 
                    } else {
                        // No more items.
                        Poll::Ready(())
                    }
                } else {
                    Poll::Pending
                }
            }
            Forward::WaitingForSend(send, rx) => {
                if let Poll::Ready(tx) = send.poll() {
                    let rx = rx.take().unwrap();
                    *self = Forward::WaitingForReceive(rx.receive(), Some(tx));
                    // Try to make progress on receiving.
                    return self.poll();
                } else {
                    Poll::Pending
                }
            }
        }
    }
}
```

列表 8-4：手动实现一个通道转发的未来

你在 Rust 中很少需要再写这样的代码了，但它提供了关于底层工作原理的重要洞察，所以让我们一起了解一下。首先，我们将未来类型定义为一个`enum`，用来追踪我们当前在等待什么。这是因为当我们返回`Poll::Pending`时，下一次调用`poll`会从函数的顶部重新开始。我们需要某种方式来知道我们当时停在哪里，这样我们才能知道继续进行哪个操作。此外，根据我们正在做的事情，我们还需要跟踪不同的信息：如果我们在等待`receive`完成，我们需要保留那个`ReceiveFuture`（这个定义在本示例中没有显示），这样我们下次被轮询时就能轮询它，`SendFuture`也是一样。这里的`Option`可能也会让你觉得奇怪；我们很快就会解释它们。

当我们为`Forward`实现`Future`时，我们将其输出类型声明为`()`，因为这个未来实际上并不返回任何东西。相反，当它完成从输入通道到输出通道的所有转发时，未来就会解析（没有结果）。在一个更完整的示例中，我们的转发类型的`Output`可能是一个`Result`，这样它就可以将来自`receive()`和`send()`的错误信息传递回堆栈中的函数，以便轮询转发的完成。但这段代码已经足够复杂了，我们暂时将这个问题留到以后再说。

当`Forward`被轮询时，它需要从上次停下的地方恢复，我们通过匹配当前在`self`中持有的枚举变体来找出这一点。无论进入哪个分支，第一步是轮询当前操作阻塞进度的未来；如果我们试图接收，我们就轮询`ReceiveFuture`，如果我们试图发送，我们就轮询`SendFuture`。如果对`poll`的调用返回`Poll::Pending`，那么我们无法取得进展，我们也会返回`Poll::Pending`。但如果当前的未来被解析，我们就有事情要做！

当其中一个内部的未来解析时，我们需要通过切换`self`中存储的枚举变体来更新当前的操作。为了做到这一点，我们必须从`self`中移出，以调用`Receiver::receive`或`Sender::send`——但是我们不能这样做，因为我们只有`&mut self`。因此，我们将必须移出的状态存储在一个`Option`中，我们通过`Option::take`移出它。这看起来有些傻，因为我们即将覆盖`self`，因此这些`Option`总是`Some`，但有时一些技巧是必须的，才能让借用检查器满意。

最后，如果我们确实取得了进展，我们会再次轮询`self`，这样如果我们能立即处理待发送或待接收的操作，就会继续进行。这实际上是实现真实`Future`特质时的必要步骤，稍后我们会回到这个话题，但现在可以把它当作一种优化来理解。

我们刚刚手写了一个 *状态机*：一种具有多个可能状态并根据特定事件在状态之间移动的类型。实际上，这只是一个相当简单的状态机。试想一下，如果你需要为更复杂的用例编写这样的代码，其中还包含额外的中间步骤，会是怎样的一种情况！

除了编写笨重的状态机外，我们还必须知道`Sender::send`和`Receiver::receive`返回的 futures 类型，以便我们能够将它们存储在我们的类型中。如果这些方法返回的是`impl Future`，我们将无法为我们的变体写出类型。`send`和`receive`方法还必须获取发送者和接收者的所有权；如果没有获取所有权，它们返回的 futures 的生命周期将与`self`的借用相关联，而当我们从`poll`返回时，生命周期就会结束。但那样是不行的，因为我们要尝试将这些 futures *存储在* `self`中。

最终，这段代码既难以编写，也难以阅读，还难以修改。例如，如果我们想添加错误处理，代码的复杂度将显著增加。幸运的是，有一种更好的方法！

### async/await

Rust 1.39 为我们引入了`async`关键字和紧密相关的`await`后缀操作符，我们在清单 8-3 中的原始示例中使用了它们。它们一起提供了一种更方便的机制，用于编写像清单 8-5 中那样的异步状态机。具体来说，它们让你可以以一种方式编写代码，看起来根本不像是状态机！

```
async fn forward<T>(rx: Receiver<T>, tx: Sender<T>) {
    while let Some(t) = rx.next().await {
        tx.send(t).await;
    }
}
```

清单 8-5：使用`async`和`await`实现一个频道转发的 future，重复自清单 8-3

如果你对`async`和`await`没有太多经验，清单 8-4 和清单 8-5 之间的区别可能会让你大致明白为什么 Rust 社区对它们的出现如此兴奋。但由于这是一本中级书籍，让我们更深入地探讨一下，理解这一小段代码是如何替代更长的手动实现的。为此，我们首先需要谈论一下 *生成器*——`async`和`await`的实现机制。

#### 生成器

简单来说，生成器是一段代码，带有一些额外的由编译器生成的部分，使得它能够在执行过程中暂停，或 *yield*，然后在稍后从上次暂停的地方恢复。以清单 8-3 中的`forward`函数为例。假设它执行到调用`send`时，频道当前已满。函数无法继续执行，但它也不能阻塞（毕竟这是非阻塞代码），因此它需要返回。现在假设频道最终清空，我们希望继续发送。如果我们从头再次调用`forward`，它会再次调用`next`，而我们之前尝试发送的项目会丢失，这样就不好了。相反，我们将`forward`变成一个生成器。

每当`forward`生成器无法再继续执行时，它需要将当前状态存储在某个地方，以便在执行恢复时，能够在正确的位置和正确的状态下恢复。它通过编译器生成的一个关联数据结构保存状态，该结构包含生成器在某一时刻的所有状态。该数据结构上的一个方法（也是编译器生成的）允许生成器从当前状态（存储在`&mut self`中）恢复，并在生成器再次无法继续执行时更新状态。

这种“返回但允许我稍后恢复”的操作称为*yielding*，其有效含义是它在返回的同时保持一些额外的状态。稍后当我们想恢复对`forward`的调用时，我们会调用生成器的已知入口点（即*恢复方法*，对于`async`生成器来说是`poll`），生成器检查之前存储在`self`中的状态来决定下一步做什么。这与我们在列表 8-4 中手动完成的操作完全相同！换句话说，列表 8-5 中的代码松散地等价于列表 8-6 中所示的假设代码。

```
generator fn forward<T>(rx: Receiver<T>, tx: Sender<T>) {
    loop {
        let mut f = rx.next();
        let r = if let Poll::Ready(r) = f.poll() { r } else { yield };
        if let Some(t) = r {
            let mut f = tx.send(t);
            let _ = if let Poll::Ready(r) = f.poll() { r } else { yield };
        } else { break Poll::Ready(()); }
    }
}
```

列表 8-6：将`async`/`await`转化为生成器

截至目前，生成器在 Rust 中实际上是不可用的——它们仅在编译器内部用于实现`async`/`await`——但这在未来可能会改变。生成器在许多情况下非常有用，例如在不需要携带`struct`的情况下实现迭代器，或者实现一个`impl Iterator`，它能够逐个处理生成项。

如果你仔细观察列表 8-5 和 8-6，你可能会发现一旦知道每个`await`或`yield`实际上是函数的一个返回，你就会觉得它们有些神奇。毕竟，函数中有几个局部变量，且不清楚它们在我们稍后恢复时是如何恢复的。这正是编译器生成的生成器部分发挥作用的地方。编译器透明地注入代码，在执行时将这些变量持久化到生成器的关联数据结构中，而不是栈中。因此，如果你声明、写入或读取某个局部变量`a`，你实际上是在操作类似于`self.a`的东西。问题解决！这一切实际上都非常神奇。

手动实现的 `forward` 和 `async`/`await` 版本之间有一个微妙但重要的区别，即后者可以跨 `yield` 点持有引用。这使得像清单 8-5 中的 `Receiver::next` 和 `Sender::send` 这样的函数可以使用 `&mut self`，而不是像清单 8-4 中那样使用 `self`。如果我们在手动状态机实现中尝试使用 `&mut self` 的接收器，借用检查器将无法确保 `Receiver` 存储在 `Forward` 中的实例在 `Receiver::next` 被调用和它返回的 future 解析之间不会被引用，因此它会拒绝这段代码。只有将 `Receiver` 移动到 future 中，我们才能说服编译器相信 `Receiver` 不会被其他方式访问。与此同时，使用 `async`/`await` 时，借用检查器可以在编译器将代码转换成状态机之前检查代码，并验证 `rx` 在 future 被丢弃之前确实没有再次被访问，直到 `await` 返回。

### Pin 和 Unpin

我们还没有完全完成。虽然生成器很有趣，但从目前为止我描述的技术中会出现一个挑战。特别是，如果生成器中的代码（或者等价地，`async` 块）引用了局部变量，那么到底会发生什么并不明确。在清单 8-5 中的代码中，如果下一个消息不可立即获取，则`rx.next()`返回的 future 必须持有 `rx` 的引用，以便它知道生成器下一次恢复时该从哪里重新开始。当生成器 `yield` 时，future 和它所包含的引用会被存储在生成器内部。但是如果生成器被移动了，现在会发生什么呢？特别地，看看清单 8-7 中的代码，它调用了 `forward`。

```
async fn try_forward<T>(rx: Receiver<T>, tx: Sender<T>) -> Option<impl Future> {
    let mut f = forward(rx, tx);
    if f.poll().is_pending() { Some(f) } else { None }
}
```

清单 8-7：轮询之后移动一个 future

`try_forward` 函数只轮询一次 `forward`，以便尽可能多地转发消息而不阻塞。如果接收方可能仍会生成更多消息（即，如果它返回的是 `Poll::Pending` 而不是 `Poll::Ready(None)`），这些消息会被推迟到稍后转发，通过将转发的 future 返回给调用者，调用者可以选择在合适的时机再次轮询。

让我们结合目前对`async`和`await`的理解，逐步分析这里发生了什么。当我们轮询`forward`生成器时，它会进入`while`循环若干次，最终返回`Poll::Ready(())`，如果接收方已结束，或者返回`Poll::Pending`，否则。如果返回`Poll::Pending`，则生成器包含一个由`rx.next()`或`tx.send(t)`返回的`future`。这两个`future`都包含对最初传递给`forward`的某个参数的引用（分别是`rx`和`tx`），这些引用也必须存储在生成器中。但是，当`try_forward`返回整个生成器时，生成器的字段也会移动。因此，`rx`和`tx`不再位于内存中的相同位置，存储在临时`future`中的引用不再指向正确的数据！

我们在这里遇到的是一个*自指*数据结构的案例：一种同时包含数据和对该数据的引用的结构。使用生成器，这些自指结构非常容易构造，不能支持它们将对易用性产生重大打击，因为这意味着你将无法在任何`yield`点之间保持引用。Rust 支持自指数据结构的（巧妙的）解决方案是`Pin`类型和`Unpin`特征。简而言之，`Pin`是一个包装类型，它阻止被包装类型（安全地）移动，而`Unpin`是一个标记特征，表示实现该特征的类型*可以*从`Pin`中安全地移除。

#### Pin

这里有很多细节需要探讨，让我们从`Pin`包装器的一个具体用法开始。Listing 8-2 给出了`Future`特征的简化版本，但现在我们准备好揭示简化的部分内容。Listing 8-8 展示了更接近最终形式的`Future`特征。

```
trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>) -> Poll<Self::Output>;
}
```

Listing 8-8: 带有`Pin`的`Future`特征的较简化视图

特别地，这个定义要求你在`Pin<&mut Self>`上调用`poll`。一旦你得到了一个`Pin`包装的值，就意味着你与这个值之间的契约：这个值永远不会再移动。这意味着你可以根据需要在内部构造自引用，正如你为生成器所期望的那样。

但是，如何让`Pin`调用`poll`呢？`Pin`如何确保包含的值不会移动？要了解这个魔法是如何运作的，我们来看看`std::pin::Pin`的定义和一些关键方法，如 Listing 8-9 所示。

```
struct Pin<P> { pointer: P }
impl<P> Pin<P> where P: Deref {
    pub unsafe fn new_unchecked(pointer: P) -> Self;
}
impl<'a, T> Pin<&'a mut T> {
    pub unsafe fn get_unchecked_mut(self) -> &'a mut T;
}
impl<P> Deref for Pin<P> where P: Deref {
    type Target = P::Target;
    fn deref(&self) -> &Self::Target;
}
```

Listing 8-9: `std::pin::Pin`及其关键方法

这里有很多内容需要理解，我们需要多次查看 Listing 8-9 的定义，直到所有细节都能理顺，所以请耐心一点。

首先，你会注意到`Pin`持有的是*指针类型*。也就是说，它并不是直接持有某个`T`，而是持有一个通过`Deref`解引用到`T`的类型`P`。这意味着，你不会直接拥有一个`Pin<MyType>`，而是会拥有`Pin<Box<MyType>>`、`Pin<Rc<MyType>>`或`Pin<&mut MyType>`。这样设计的原因很简单——`Pin`的主要目的是确保一旦你把`T`放在`Pin`后面，`T`就不会移动，因为这样做可能会使存储在`T`中的自引用失效。如果`Pin`直接持有`T`，那么仅仅移动`Pin`就足以使这个不变式失效！在本节的其余部分，我将`P`称为*指针*类型，将`T`称为*目标*类型。

接下来，注意到`Pin`的构造函数`new_unchecked`是一个不安全的函数。这是因为编译器无法实际检查指针类型是否真的承诺被指向的（目标）类型不会再移动。例如，考虑一个栈上的变量`foo`。如果`Pin`的构造函数是安全的，我们可以执行`Pin::new(&mut foo)`，然后调用一个需要`Pin<&mut Self>`的方法（因此假设`Self`不会再移动），接着丢弃`Pin`。此时，我们可以任意修改`foo`，因为它不再被借用——包括移动它！然后我们可以再次将它固定，并调用相同的方法，但该方法并不会察觉任何它可能在第一次调用时构造的自引用指针现在已经无效了。

接着是`get_unchecked_mut`方法，它返回一个对`Pin`的指针类型后面的`T`的可变引用。这个方法也是不安全的，因为一旦我们给出了一个`&mut T`，调用者必须保证不会使用这个`&mut T`来移动`T`或以其他方式使其内存无效，否则任何自引用都会失效。如果这个方法不是不安全的，调用者可以调用一个接受`Pin<&mut Self>`的方法，然后在两个`Pin<&mut _>`上调用`get_unchecked_mut`的安全版本，再使用`mem::swap`交换`Pin`后面的值。如果我们随后再次在任一`Pin`上调用一个接受`Pin<&mut Self>`的方法，它会假设`Self`没有移动，但这种假设会被破坏，任何它存储的内部引用都会无效！

或许令人惊讶的是，`Pin<P>`总是实现了`Deref<Target = T>`，而且这是完全安全的。原因在于，`&T`不会让你在没有写其他不安全代码（例如`UnsafeCell`，我们将在第九章讨论）的情况下移动`T`。这是一个很好的例子，说明了为什么不安全代码块的作用范围不仅限于它所包含的代码。如果你在应用程序的某个地方（不安全地）用`UnsafeCell`替换了一个`&`后面的`T`，那么*可能*这个`&T`最初来自一个`Pin<&mut T>`，而你现在破坏了`Pin`后面`T`永远不能移动的这个不变式，即使你在不安全地替换`&T`的地方根本没有提到`Pin`！

#### Unpin: 安全固定的关键

在此时，你可能会问：既然获取可变引用本身就不安全，为什么不让`Pin`直接持有`T`呢？也就是说，为什么不通过指针类型间接访问，而是将`get_unchecked_mut`的契约设为：只有在你没有移动`Pin`时调用它才是安全的。这个问题的答案就在于，`Pin`的指针设计允许我们进行一种巧妙的安全使用。回想一下，我们最初需要`Pin`的原因是希望能够使用可能包含自身引用的目标类型（比如生成器），并为其方法提供一个保证，即目标类型未发生移动，从而确保内部的自引用仍然有效。`Pin`使我们能够使用类型系统来强制执行这个保证，这是很棒的。但不幸的是，按目前的设计，`Pin`使用起来非常笨拙。这是因为它总是需要不安全的代码，即使你正在处理一个不包含任何自引用的目标类型，也不关心它是否已被移动。

这时标记特征`Unpin`就发挥作用了。为某个类型实现`Unpin`，简单地声明该类型在作为目标类型时，能够安全地从`Pin`中移出。也就是说，该类型保证在作为目标类型使用时，永远不会使用`Pin`所提供的关于引用对象不再移动的任何保证，因此这些保证可以被打破。`Unpin`是一个自动特征，就像`Send`和`Sync`一样，因此编译器会为任何只包含`Unpin`成员的类型自动实现`Unpin`。只有那些明确选择不实现`Unpin`的类型（比如生成器）以及包含这些类型的类型才是`!Unpin`。

对于`Unpin`类型的目标，我们可以提供一个更简单的安全接口给`Pin`，正如在清单 8-10 中所示。

```
impl<P> Pin<P> where P: Deref, P::Target: Unpin {
    pub fn new(pointer: P) -> Self;
}
impl<P> DerefMut for Pin<P> where P: DerefMut, P::Target: Unpin {
    fn deref_mut(&mut self) -> &mut Self::Target;
}
```

清单 8-10：用于`Unpin`目标类型的安全 API `Pin`

要理解清单 8-10 中的安全 API，可以思考清单 8-9 中的不安全方法的安全要求：`Pin::new_unchecked`函数是不安全的，因为调用者必须保证引用对象不能被移出`Pin`，并且指针类型的`Deref`、`DerefMut`和`Drop`的实现不会通过它们接收到的引用移动引用对象。这些要求是为了确保一旦我们将`Pin`交给某个`T`，就不再移动该`T`。但是，如果`T`是`Unpin`，它已经声明自己不关心是否被移动，即使之前它是被固定的，因此如果调用者没有满足这些要求也没问题！

类似地，`get_unchecked_mut`是`unsafe`的，因为调用者必须保证它不会将`T`从`&mut T`中移出——但对于`T: Unpin`，`T`已经声明它在被固定后仍然可以被移动，因此这个安全要求不再重要。这意味着对于`Pin<P> where P::Target: Unpin`，我们可以简单地提供这两个方法的安全版本（`DerefMut`是`get_unchecked_mut`的安全版本）。事实上，我们甚至可以提供一个`Pin::into_inner`，它会在目标类型是`Unpin`时简单地返回拥有的`P`，因为`Pin`在这种情况下基本上没有意义！

#### 获取 Pin 的方式

通过我们对`Pin`和`Unpin`的新理解，我们现在可以朝着使用需要`Pin<&mut Self>`的新`Future`定义的方向前进，这个定义来自于列表 8-8。第一步是构造所需的类型。如果未来类型是`Unpin`，那一步很简单——我们只需要使用`Pin::new(&mut future)`。如果它不是`Unpin`，我们可以通过两种主要方式之一将未来固定：通过将它固定到堆上或固定到栈上。

让我们从将值固定到堆上开始。`Pin`的主要合同是，一旦某个对象被固定，它就不能再移动。固定 API 会确保所有方法和特性都遵守这一合同，因此构造`Pin`的任何函数的主要作用是确保如果`Pin` *本身*移动，引用的值也不会移动。确保这一点最简单的方法是将引用的值放在堆上，然后在`Pin`中放置对引用值的指针。你可以随心所欲地移动`Pin`，但是目标值会保持原样。这就是（安全）方法`Box::pin`的逻辑，它接受一个`T`并返回一个`Pin<Box<T>>`。这并没有什么神奇之处；它只是确保`Box`遵循`Pin`构造函数、`Deref`和`Drop`合同。

另一个选项是将值固定到栈上，这有点复杂，在撰写时需要一些不安全的代码。我们必须确保在`Pin`与`&mut`引用已被丢弃后，固定的值无法再被访问。我们通过像列表 8-11 中宏所示的那样对值进行遮蔽，或使用提供此类宏的其中一个库来实现这一点。也许有一天它甚至会进入标准库！

```
macro_rules! pin_mut {
    ($var:ident) => {
        let mut $var = $var;
        let mut $var = unsafe { Pin::new_unchecked(&mut $var) };
    }
}
```

列表 8-11：用于将值固定到栈上的宏

通过获取要固定到栈上的变量名，宏确保调用者已经在栈上的某个地方有了它想要固定的值。对`$var`的遮蔽确保了调用者无法丢弃`Pin`并继续使用未固定的值（这会违反对任何`!Unpin`目标类型的`Pin`合同）。通过移动存储在`$var`中的值，宏还确保调用者不能在不丢弃原始变量的情况下丢弃绑定宏声明的`$var`。具体来说，如果没有那行代码，调用者可能会写出（请注意额外的作用域）：

```
let foo = /* */; { pin_mut!(foo); foo.poll() }; foo.mut_self_method();
```

在这里，我们将一个`foo`的固定实例传递给`poll`，但是之后我们又使用了一个`&mut`来传递`foo`，而没有使用`Pin`，这违反了`Pin`契约。另一方面，通过额外的重新赋值，这段代码会将`foo`移动到新的作用域中，使得它在作用域结束后变得不可用。

因此，栈上的固定操作需要不安全的代码，这与`Box::pin`不同，但避免了`Box`引入的额外分配，并且在`no_std`环境中也能正常工作。

#### 回到未来

现在我们有了固定的 future，并且我们知道这意味着什么。但你可能已经注意到，尽管这些重要的固定操作在大多数你写的异步代码中并没有显现出来——比如使用`async`和`await`，而且这是因为编译器将它们隐藏了。

回想一下我们讨论过的列表 8-5，当时我告诉你，`<expr>.await`去糖化成类似于以下内容：

```
loop { if let Poll::Ready(r) = expr.poll() { break r } else { yield } }
```

那只是一个非常轻微的简化，因为正如我们所看到的，只有当你拥有一个`Pin<&mut Self>`类型的 future 时，才能调用`Future::poll`。去糖化实际上要复杂一些，如列表 8-12 所示。

```
1 match expr {
      mut pinned => loop {
        2 match unsafe { Pin::new_unchecked(&mut pinned) }.poll() {
              Poll::Ready(r) => break r,
              Poll::Pending => yield,
          }
    }
}
```

列表 8-12：`<expr>.await`的更准确的去糖化

匹配 1 是一种巧妙的简写，不仅确保扩展保持有效表达式，还将表达式的结果移动到一个变量中，我们可以在栈上对其进行固定。除此之外，主要的新内容是调用了`Pin::new_unchecked` 2。该调用是安全的，因为为了能够对包含的异步块进行轮询，它必须已经被固定，因为`Future::poll`的签名要求如此。而异步块已经被轮询过，因此我们达到了调用`Pin::new_unchecked`的步骤，所以生成器状态已经被固定。由于`pinned`存储在与异步块对应的生成器中（它必须如此，以确保`yield`能正确恢复），我们知道`pinned`不会再移动。而且，一旦进入循环，`pinned`无法被访问，除非通过`Pin`，所以没有代码能够将值从`pinned`中移出。因此，我们满足了`Pin::new_unchecked`的所有安全要求，代码是安全的。

## 睡觉中

我们已经深入探讨了`Pin`，但现在我们已经走出那一段，关于 future 还有另一个可能让你大脑痒痒的问题。如果`Future::poll`调用返回`Poll::Pending`，你需要某种机制在稍后的时间再次调用`poll`来检查是否可以继续推进。这个机制通常被称为*执行器*。你的执行器可以是一个简单的循环，轮询你等待的所有 future，直到它们都返回`Poll::Ready`，但这样会浪费很多 CPU 周期，而你本可以将它们用于其他更有用的事情，比如运行你的网页浏览器。相反，我们希望执行器做它能做的任何有用工作，然后进入睡眠状态。它应该保持睡眠，直到某个 future 能够继续推进，只有到那个时候，才会醒来做下一次轮询，再次进入睡眠。

### 醒来

决定何时检查给定 future 的条件差异很大。它可能是“当网络包到达此端口时”，“当鼠标光标移动时”，“当有人在此通道上发送时”，“当 CPU 收到特定中断时”，甚至是“经过了这么多时间之后”。此外，开发人员还可以编写自己的 futures，它们可能会包装多个其他 futures，因此可能有多个唤醒条件。一些 futures 甚至可能引入完全自定义的唤醒事件。

为了适应这些不同的使用场景，Rust 引入了 `Waker` 的概念：一种唤醒执行器以信号推进的方式。`Waker` 是整个未来（futures）机制能够工作的关键。执行器构造一个与其睡眠机制整合的 `Waker`，并将该 `Waker` 传递给它轮询的每一个 `Future`。怎么做的？就是通过我之前一直隐藏的 `Future::poll` 的额外参数。抱歉之前没告诉你。Listing 8-13 给出了 `Future` 的最终和真实定义——没有更多的谎言！

```
trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

Listing 8-13：实际的 `Future` trait 和 `Context`

`&mut Context` 包含了 `Waker`。该参数是一个 `Context`，而不是直接传递一个 `Waker`，这样我们可以在需要时为 futures 提供额外的上下文，从而扩展异步生态系统。

`Waker` 的主要方法是 `wake`（以及按引用变体 `wake_by_ref`），当 future 可以继续推进时应该调用该方法。`wake` 方法不接受任何参数，其效果完全由构造 `Waker` 的执行器定义。你看，`Waker` 在幕后是对执行器进行泛型化的。或者更准确地说，构造 `Waker` 的对象决定了当调用 `Waker::wake`、克隆 `Waker` 以及丢弃 `Waker` 时会发生什么。这一切都是通过手动实现的 vtable 来实现的，类似于我们在第二章讨论的动态派发。

构造一个 `Waker` 是一个相对复杂的过程，其机制对于使用它并不是特别重要，但你可以在标准库中的 `RawWakerVTable` 类型中看到构建块。它有一个构造函数，接受 `wake` 和 `wake_by_ref` 的函数指针，以及 `Clone` 和 `Drop`。`RawWakerVTable` 通常在所有执行器的 waker 之间共享，它与一个原始指针捆绑在一起，该指针用于存储特定于每个 `Waker` 实例的数据（比如它是针对哪个 future），然后转换为一个 `RawWaker`。接着，它被传递到 `Waker::from_raw` 以生成一个安全的 `Waker`，可以传递给 `Future::poll`。

### 履行 Poll 合约

到目前为止，我们已经回避了 future 如何使用 `Waker` 的问题。这个概念相当简单：如果 `Future::poll` 返回 `Poll::Pending`，那么 future 的责任是确保当 future 下次能够进展时，*某些东西* 会调用提供的 `Waker` 的 `wake` 方法。大多数 future 通过仅在其他 future 也返回 `Poll::Pending` 时才返回 `Poll::Pending` 来遵守这一规则；通过这种方式，它轻松履行了 `poll` 的合同，因为内部的 future 必须遵循相同的合同。但事情不可能永远这么简单。最终，你会遇到一个不对其他 future 进行轮询的 future，而是做一些像写入网络套接字或尝试从通道接收的操作。这些通常被称为 *叶子 future*，因为它们没有子 future。叶子 future 没有内部 future，而是直接表示某个可能还未准备好返回结果的资源。

叶子 future 通常有两种形态：一种是等待来自同一进程内的事件（比如通道接收器），另一种是等待来自进程外部的事件（比如 TCP 数据包读取）。那些等待内部事件的叶子 future 都倾向于遵循相同的模式：将 `Waker` 存储在代码中，确保唤醒代码可以找到它，并在生成相关事件时调用 `Waker` 的 `wake` 方法。例如，考虑一个必须等待内存通道中消息的叶子 future。它将其 `Waker` 存储在通道的发送者和接收者共享的部分，然后返回 `Poll::Pending`。当发送者稍后往通道中注入一条消息时，它会注意到接收者留下的 `Waker`，并在从 `send` 返回之前调用 `wake` 方法。现在接收者被唤醒，轮询合同得到了遵守。

处理外部事件的叶子 future 更为复杂，因为生成它们所等待事件的代码并不了解 future 或 waker。最常见的生成代码是操作系统内核，它知道何时磁盘准备就绪或定时器到期，但它也可能是一个 C 库，在操作完成时调用回调进入 Rust 或其他类似的外部实体。这样一个包装了外部资源的叶子 future 可能会启动一个执行阻塞系统调用（或等待 C 回调）的线程，然后使用内部唤醒机制，但那样会浪费资源；每次操作需要等待时都会启动一个线程，结果就会有很多只使用一次的线程闲置在那里等待事件。

相反，执行器通常提供叶子 future 的实现，这些实现与执行器在幕后通信，以安排与操作系统的适当交互。具体如何协调取决于执行器和操作系统，但大致来说，执行器会跟踪所有它应该监听的事件源，以便下次进入休眠时使用。当叶子 future 意识到必须等待外部事件时，它会更新该执行器的状态（它知道这个状态，因为它由执行器 crate 提供），将外部事件源与其 `Waker` 一起包括在内。当执行器无法继续执行时，它会收集所有正在等待的叶子 future 的事件源，并进行一次大的阻塞调用，告诉操作系统，当*任何*叶子 future 正在等待的资源有新事件时返回。在 Linux 上，通常通过 `epoll` 系统调用实现；Windows、BSD、macOS 以及几乎所有其他操作系统也提供类似的机制。当该调用返回时，执行器会对所有与操作系统报告事件的事件源相关联的 waker 调用 `wake`，从而完成轮询契约。

叶子 future 与执行器之间紧密集成的一个连锁反应是，来自一个执行器 crate 的叶子 future 通常不能与另一个执行器一起使用。或者至少，除非叶子 future 的执行器*也*在运行，否则无法使用。当叶子 future 要存储其 `Waker` 并注册它正在等待的事件源时，它所构建的执行器需要设置该状态，并且需要运行，以便事件源实际上会被监视，并最终调用 `wake`。有一些方法可以绕过这个问题，例如在没有执行器运行的情况下让叶子 future 生成一个执行器，但这并不总是可取的，因为这意味着应用程序可能会透明地在同一时间运行多个执行器，这会降低性能，并且在调试时必须检查多个执行器的状态。

希望支持多个执行器的库 crate 必须在其叶子资源上使用泛型。例如，库可以存储一个泛型的 `T: AsyncRead + AsyncWrite`，而不是使用特定执行器的``TcpStream 或 `File` future 类型。然而，生态系统尚未确定这些 trait 应该是什么样子以及需要哪些 trait，因此目前很难使代码在执行器上真正具有泛型性。例如，虽然 `AsyncRead` 和 `AsyncWrite` 在生态系统中有些常见（或者如果需要，可以很容易地进行适配），但目前还没有为在后台运行 future（*生成*，我们稍后将讨论）或表示定时器的 trait。

````### Waking Is a Misnomer    You may already have realized that `Waker::wake` doesn’t necessarily seem to *wake* anything. For example, for external events (as described in the previous section), the executor is already awake, and it might seem silly for it to then call `wake` on a `Waker` that belongs to that executor anyway! The reality is that `Waker::wake` is a bit of a misnomer—in reality, it signals that a particular future is *runnable*. That is, it tells the executor that it should make sure to poll this particular future when it gets around to it rather than go to sleep again, since this future can make progress. This might wake the executor if it is currently sleeping so it will go poll that future, but that’s more of a side effect than its primary purpose.    It is important for the executor to know which futures are runnable for two reasons. First, it needs to know when it can stop polling a future and go to sleep; it’s not sufficient to just poll each future until it returns `Poll::Pending`, since polling a later future might make it possible to progress an earlier future. Consider the case where two futures bounce messages back and forth on channels to one another. When you poll one, the other becomes ready, and vice versa. In this case, the executor should never go to sleep, as there is always more work to do.    Second, knowing which futures are runnable lets the executor avoid polling futures unnecessarily. If an executor manages thousands of pending futures, it shouldn’t poll all of them just because an event made one of them runnable. If it did, executing asynchronous code would get very slow indeed.    ### Tasks and Subexecutors    The futures in an asynchronous program form a tree: a future may contain any number of other futures, which in turn may contain other futures, all the way down to the leaf futures that interact with wakers. The root of each tree is the future you give to whatever the executor’s main “run” function is. These root futures are called *tasks*, and they are the only point of contact between the executor and the futures tree. The executor calls `poll` on the task, and from that point forward the code of each contained future must figure out which inner future(s) to poll in response, all the way down to the relevant leaf.    Executors generally construct a separate `Waker` for each task they poll so that when `wake` is later called, they know which task was just made runnable and can mark it as such. That is what the raw pointer in `RawWaker` is for—to differentiate between tasks while sharing the code for the various `Waker` methods.    When the executor eventually polls a task, that task starts running from the top of its implementation of `Future::poll` and must decide from there how to get to the future deeper down that can now make progress. Since each future knows only about its own fields, and nothing about the whole tree, this all happens through calls to `poll` that each traverse one edge in the tree.    The choice of which inner future to poll is often obvious, but not always. In the case of `async`/`await`, the future to poll is the one we’re blocked waiting for. But in a future that waits for the first of several futures to make progress (often called a *select*), or for all of a set of futures (often called a *join*), there are many options. A future that has to make such a choice is basically a subexecutor. It could poll all of its inner futures, but doing so could be quite wasteful. Instead, these subexecutors often wrap the `Waker` they receive in `poll`’s `Context` with their own `Waker` type before they invoke `poll` on any inner future. In the wrapping code, they mark the future they just polled as runnable in their own state before they call `wake` on the original `Waker`. That way, when the executor eventually polls the subexecutor future again, the subexecutor can consult its own internal state to figure out which of its inner futures caused the current call to `poll`, and then only poll those.    ## Tying It All Together with spawn    When working with asynchronous executors, you may come across an operation that spawns a future. We’re now in a position to explore what that means! Let’s do so by way of example. First, consider the simple server implementation in Listing 8-14.    ``` async fn handle_client(socket: TcpStream) -> Result<()> {     // Interact with the client over the given socket. }  async fn server(socket: TcpListener) -> Result<()> {     while let Some(stream) = socket.accept().await? {         handle_client(stream).await?;     } } ```    Listing 8-14: Handling connections sequentially    The top-level `server` function is essentially one big future that listens for new connections and does something when a new connection arrives. You hand that future to the executor and say “run this,” and since you don’t want your program to then exit immediately, you’ll probably have the executor block on that future. That is, the call to the executor to run the server future will not return until the server future resolves, which may be never (another client could always arrive later).    Now, every time a new client connection comes in, the code in Listing 8-14 makes a new future (by calling `handle_client`) to handle that connection. Since the handling is itself a future, we `await` it and then move on to the next client connection.    The downside of this approach is that we only ever handle one connection at a time—there is no concurrency. Once the server accepts a connection, the `handle_client` function is called, and since we `await` it, we don’t go around the loop again until `handle_client`’s return future resolves (presumably when that client has left).    We could improve on this by keeping a set of all the client futures and having the loop in which the server accepts new connections also check all the client futures to see if any can make progress. Listing 8-15 shows what that might look like.    ``` async fn server(socket: TcpListener) -> Result<()> {     let mut clients = Vec::new();     loop {         poll_client_futures(&mut clients)?;         if let Some(stream) = socket.try_accept()? {             clients.push(handle_client(stream));         }     } } ```    Listing 8-15: Handling connections with a manual executor    This at least handles many connections concurrently, but it’s quite convoluted. It’s also not very efficient because the code now busy-loops, switching between handling the connections we already have and accepting new ones. And it has to check each connection each time, since it won’t know which ones can make progress (if any). It also can’t `await` at any point, since that would prevent the other futures from making progress. You could implement your own wakers to ensure that the code polls only the futures that can make progress, but ultimately this is going down the path of developing your own mini-executor.    Another downside of sticking with just the one task for the server that internally contains the futures for all of the client connections is that the server ends up being single-threaded. There is just the one task and to poll it the code must hold an exclusive reference to the task’s future (`poll` takes `Pin<&mut Self>`), which only one thread can hold at a time.    The solution is to make each client future its own task and leave it to the executor to multiplex among all the tasks. Which, you guessed it, you do by spawning the future. The executor will continue to block on the server future, but if it cannot make progress on that future, it will use its execution machinery to make progress on the other tasks in the meantime behind the scenes. And best of all, if the executor is multithreaded and your client futures are `Send`, it can run them in parallel since it can hold `&mut`s to the separate tasks concurrently. Listing 8-16 gives an example of what this might look like.    ``` async fn server(socket: TcpListener) -> Result<()> {     while let Some(stream) = socket.accept().await? {         // Spawn a new task with the Future that represents this client.         // The current task will continue to just poll for more connections         // and will run concurrently (and possibly in parallel) with handle_client.         spawn(handle_client(stream));     } } ```    Listing 8-16: Spawning futures to create more tasks that can be polled concurrently    When you spawn a future and thus make it a task, it’s sort of like spawning a thread. The future continues running in the background and is multiplexed concurrently with any other tasks given to the executor. However, unlike a spawned thread, spawned tasks still depend on being polled by the executor. If the executor stops running, either because you drop it or because your code no longer runs the executor’s code, those spawned tasks will stop making progress. In the server example, imagine what will happen if the main server future resolves for some reason. Since the executor has returned control back to your code, it cannot continue doing, well, anything. Multi-threaded executors often spawn background threads that continue to poll tasks even if the executor yields control back to the user’s code, but not all executors do this, so check your executor before you rely on that behavior!    ## Summary    In this chapter, we’ve taken a look behind the scenes of the asynchronous constructs available in Rust. We’ve seen how the compiler implements generators and self-referential types, and why that work was necessary to support what we now know as `async`/`await`. We’ve also explored how futures are executed, and how wakers allow executors to multiplex among tasks when only some of them can make progress at any given moment. In the next chapter, we’ll tackle what is perhaps the deepest and most discussed area of Rust: unsafe code. Take a deep breath, and then turn the page.````
