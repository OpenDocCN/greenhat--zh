# 第十三章：日志记录和指标

![](img/chapterart.png)

在理想的世界里，我们的代码从一开始就没有缺陷。我们的网络服务将超出我们对性能和容量的预期，并且足够强大，可以在没有我们干预的情况下适应意外的输入。但在现实世界中，我们需要担心意外的和潜在的恶意输入、硬件退化、网络中断以及代码中的明显 bug。

无论我们的应用程序是在本地还是在云端，监控它们对提供强健、功能完善的服务至关重要。全面的日志记录让我们能够及时获得关于错误、异常或其他可操作事件的详细信息，而指标则帮助我们了解服务的当前状态，并识别瓶颈。综合来看，日志和指标帮助我们管理服务问题，并集中开发力量避免未来的故障。

在之前的章节中，你使用了 Go 的`log`和`fmt`包来获取反馈，但本章将深入探讨日志记录和为服务添加监控。你将学习如何使用日志级别来增加或减少日志的详细程度，以及何时使用每个日志级别。你将学会如何为日志条目添加结构，这样软件可以帮助你更好地理解日志条目，并聚焦于相关日志。我将向你介绍广泛事件日志的概念，这将帮助你在服务规模扩大时控制日志数据量。你将学习如何从代码中动态启用调试日志记录和管理日志文件的轮换。

本章还将介绍 Go kit 的`metrics`包。根据 Go kit 文档，`metrics`包“提供了一组统一的接口，用于服务的监控”。你将学习如何通过使用计数器、仪表和直方图来为你的服务添加监控。

本章结束时，你应该掌握如何处理日志记录，如何管理日志文件以防止它们占用过多硬盘空间，以及如何为你的服务添加监控，以便深入了解其当前状态。

## 事件日志

日志记录很难。即使是经验丰富的开发者也常常难以做到完美。很难预见到在服务发生故障时，你的日志需要回答哪些问题——然而，你应该抵制记录一切日志以防万一的冲动。你需要找到一个平衡点，既能记录正确的信息来回答这些问题，又不会被无关的日志信息淹没。过度的日志记录在开发阶段可能对你来说没问题，因为你控制了测试的规模和服务的整体熵，但它会在你需要诊断生产环境故障时迅速降低你在信息海洋中找到关键线索的能力。

除了搞清楚要记录什么，你还需要考虑日志记录并非无代价。它会消耗 CPU 和 I/O 时间，而这些时间本可以被应用程序用来做其他事情。向一个繁忙的`for`循环中添加日志条目，在开发过程中可能有助于你理解你的服务在做什么。但在生产环境中，它可能成为瓶颈，悄悄地为你的服务增加延迟。

相反，采样这些日志条目，或按需记录日志，可能提供日志输出和开销之间的适当折衷。你可能会发现使用*宽事件*日志条目很有帮助，它们总结了一个事务。例如，一个正在开发中的服务可能会记录关于请求的半打条目、任何中间步骤和响应。而在生产环境中，单个宽事件日志条目提供这些详细信息的方式更具扩展性。你将在第 312 页的《通过宽事件日志扩展》一节中了解更多宽事件日志条目的内容。

最后，日志记录是主观的。一个异常在我的应用程序中可能是微不足道的，但在你的应用程序中却可能表明一个更大的问题。虽然我可以忽略这个异常，但你可能希望知道它。出于这个原因，最好我们在最佳实践的框架下讨论日志记录。这些实践是一个良好的基准方法，但你应该根据每个应用程序量身定制它们。

### log 包

在前面的章节中，你已经有了使用 Go 的`log`包的基本经验，用于基本的日志记录需求，比如给日志条目加上时间戳，或者选择性地使用`log.Fatal`退出应用程序。但它还有一些我们尚未探讨的功能。这些功能要求我们超越包级日志记录器，实例化我们自己的`*log.Logger`实例。你可以使用`log.New`函数来做到这一点：

```
func New(out io.Writer, prefix string, flag int) *Logger
```

`log.New`函数接受一个`io.Writer`、一个用于每条日志行的字符串前缀，以及一组修改日志记录器输出的标志。接受`io.Writer`意味着日志记录器可以将日志写入任何满足该接口的对象，包括内存缓冲区或网络套接字。

默认的日志记录器将输出写入`os.Stderr`，即标准错误。让我们看一个示例日志记录器，位于 Listing 13-1，它将输出写入`os.Stdout`，即标准输出。

```
func Example_log() {
    l := log.New(1os.Stdout, 2"example: ", 3log.Lshortfile)
    l.Print("logging to standard output")

    // Output:
    // example: 4log_test.go:12: logging to standard output
}
```

Listing 13-1：将日志条目写入标准输出（*log_test.go*）

你创建了一个新的`*log.Logger`实例，将其写入标准输出 1。日志记录器将每一行前缀加上字符串*example:*2。默认日志记录器的标志是`log.Ldate`和`log.Ltime`，合起来是`log.LstdFlags`，它们打印每条日志条目的时间戳。由于你希望简化输出以便在命令行运行示例时进行测试，你省略了时间戳，并配置日志记录器以写入每条日志条目的源代码文件名和行号 3。*log_test.go*文件中第 12 行的`l.Print`函数输出了这些值 4。这种行为有助于开发和调试，使你能够精确定位到感兴趣的日志条目的文件和行号。

你可能会意识到，日志记录器接受`io.Writer`，这意味着你可以使用多个写入器，比如日志文件和标准输出，或者内存环形缓冲区和网络上的集中式日志服务器。不幸的是，`io.MultiWriter`并不适合日志记录中使用。`io.MultiWriter`按顺序写入每个写入器，如果它从任何`Write`调用收到错误，就会中止。这意味着，如果你配置`io.MultiWriter`按这个顺序写入日志文件和标准输出，那么如果在写入日志文件时发生错误，标准输出将永远不会收到日志条目。

别担心，这是一个容易解决的问题。让我们从示例 13-2 开始，创建我们自己的`io.MultiWriter`实现，使其能够跨多个写入器维持写入并累积遇到的任何错误。

```
package ch13

import (
    "io"

    "go.uber.org/multierr"
)

type sustainedMultiWriter struct {
    writers []io.Writer
}

func (s *sustainedMultiWriter) 1Write(p []byte) (n int, err error) {
    for _, w := range s.writers {
        i, wErr := 2w.Write(p)
        n += i
        err = 3multierr.Append(err, wErr)
    }

    return n, err
}
```

示例 13-2：一个即使遇到错误后仍能持续写入的多写入器 (*writer.go*)

和`io.MultiWriter`一样，你将使用一个包含`io.Writer`实例切片的结构体作为持续多写入器。你的多写入器实现了`io.Writer`接口 1，因此可以将其传递给你的日志记录器。它调用每个写入器的`Write`方法 2，在 Uber 的`multierr`包 3 的帮助下累积任何错误，最终返回总共写入的字节数和累积的错误。

示例 13-3 添加了一个函数，用于从一个或多个写入器初始化一个新的持续多写入器。

```
`--snip--`

func SustainedMultiWriter(writers ...io.Writer) io.Writer {
    mw := &sustainedMultiWriter{writers: 1make([]io.Writer, 0, len(writers))}

    for _, w := range writers {
        if m, ok := 2w.(*sustainedMultiWriter); ok {
            mw.writers = 3append(mw.writers, m.writers...)
            continue
        }

        mw.writers = 4append(mw.writers, w)
 }

    return mw
}
```

示例 13-3：创建一个持续多写入器 (*writer.go*)

首先，你实例化一个新的`*sustainedMultiWriter`，初始化其写入器切片 1，并将其限制为写入器的预期长度。然后，你遍历给定的写入器并将它们追加到切片中 4。如果某个写入器本身是`*sustainedMultiWriter`2，你将改为追加它的写入器 3。最后，你返回初始化的`sustainedMultiWriter`的指针。

现在你可以在示例 13-4 中充分利用你的持续多写入器了。

```
package ch13

import (
    "bytes"
    "fmt"
    "log"
    "os"
)

func Example_logMultiWriter() {
    logFile := new(bytes.Buffer)
    w := 1SustainedMultiWriter(os.Stdout, logFile)
    l := log.New(w, "example: ", 2log.Lshortfile|log.Lmsgprefix)

    fmt.Println("standard output:")
    l.Print("Canada is south of Detroit")

    fmt.Print("\nlog file contents:\n", logFile.String())

    // Output:
    // standard output:
    // log_test.go:24: example: Canada is south of Detroit
    //
    // log file contents:
    // log_test.go:24: example: Canada is south of Detroit
}
```

示例 13-4：同时写入日志文件和标准输出 (*log_test.go*)

你在这个示例中创建了一个新的持续多写入器 1，写入标准输出，并使用一个`bytes.Buffer`作为模拟日志文件。接着，你使用持续多写入器、前缀*`example`:*和两个标志 2 来创建一个新的日志记录器，以修改日志记录器的行为。`log.Lmsgprefix`标志的添加（在 Go 1.14 中首次引入）告诉日志记录器在日志消息之前查找前缀。你可以看到这对示例输出中日志条目的影响。当你运行这个示例时，你会看到日志记录器将日志条目写入持续多写入器，而持续多写入器又将日志条目写入标准输出和日志文件。

### 分层日志条目

我在本章早些时候提到过，冗长的日志记录在生产环境中可能效率低下，并且随着服务的扩展，日志条目的数量可能会让你不堪重负。避免这种情况的一种方法是实施*日志级别*，为每种事件分配优先级，使你能够始终记录高优先级的错误日志，但根据条件记录更适合调试和开发的低优先级条目。例如，你总是希望知道服务无法连接到数据库，但在开发或诊断故障时，你可能只关心记录关于单个连接的详细信息。

我建议你开始时只创建几个日志级别。根据我的经验，你可以通过只设置一个*错误*级别和一个*调试*级别来处理大多数用例，偶尔也可以设置一个*信息*级别。错误日志条目应该伴随某种警报，因为这些条目表示需要你注意的条件。信息日志条目通常记录非错误信息。例如，记录成功的数据库连接，或在网络套接字上监听器准备好接受连接时添加日志条目，可能适合你的用例。调试日志条目应该详细，并帮助你诊断故障，同时通过帮助你理解工作流程来支持开发。

Go 的生态系统提供了几种日志包，其中大多数支持多个日志级别。尽管 Go 的`log`包本身不支持分级日志条目，但你可以通过为每个所需的日志级别创建单独的日志记录器来添加类似的功能。列表 13-5 就实现了这一点：它将日志条目写入日志文件，但也将调试日志写入标准输出。

```
`--snip--`

func Example_logLevels() {
    lDebug := log.New(os.Stdout, 1"DEBUG: ", log.Lshortfile)
    logFile := new(bytes.Buffer)
    w := SustainedMultiWriter(logFile, 2lDebug.Writer())
    lError := log.New(w, 3"ERROR: ", log.Lshortfile)

    fmt.Println("standard output:")
    lError.Print("cannot communicate with the database")
    lDebug.Print("you cannot hum while holding your nose")

    fmt.Print("\nlog file contents:\n", logFile.String())

    // Output:
    // standard output:
    // ERROR: log_test.go:43: cannot communicate with the database
    // DEBUG: log_test.go:44: you cannot hum while holding your nose
    //
    // log file contents:
    // ERROR: log_test.go:43: cannot communicate with the database
}
```

列表 13-5：将调试条目写入标准输出，并将错误同时写入日志文件和标准输出（*log_test.go*）

首先，创建一个调试日志记录器，它写入标准输出，并使用`DEBUG:`前缀 1。接下来，创建一个`*bytes.Buffer`，作为日志文件的替代，实例化一个持续的多重写入器。持续的多重写入器同时写入日志文件和调试日志记录器的`io.Writer`2。然后，创建一个错误日志记录器，它通过使用`ERROR:`前缀 3 写入持续的多重写入器，从而将其日志条目与调试日志区分开来。最后，使用每个日志记录器并验证它们的输出，确保它们的输出符合预期。标准输出应显示两个日志记录器的日志条目，而日志文件应仅包含错误日志条目。

作为练习，找出如何使调试日志记录器具有条件性，而不是将其`Print`调用包装在条件语句中。如果你需要提示，可以在`io/ioutil`包中找到一个合适的写入器，它允许你丢弃输出。

本节旨在展示 `log` 包的额外用途，超出了本书中到目前为止所使用的内容。虽然可以使用此技术在不同级别进行日志记录，但使用本节后面描述的具有内置日志级别支持的日志记录器（如 Zap 日志记录器）会更有帮助。

### 结构化日志记录

你目前所编写的代码生成的日志条目是供人类阅读的。它们对你来说很容易阅读，因为每个日志条目不过是一个消息。这意味着，找到与某个问题相关的日志行通常需要广泛使用 `grep` 命令，或者在最坏的情况下，手动浏览日志条目。但如果日志条目的数量增加，这可能会变得更加困难。你可能会发现自己在大海捞针。记住，日志记录只有在你能快速找到所需信息时才有用。

解决这个问题的常见方法是向日志条目中添加元数据，然后通过软件解析这些元数据，以帮助你组织它们。这种类型的日志记录称为*结构化日志记录*。创建结构化日志条目需要向每个日志条目添加键值对。在这些条目中，你可以包括日志条目记录的时间、生成日志条目的应用程序部分、日志级别、生成日志条目的节点的主机名或 IP 地址，以及其他你可以用于索引和筛选的元数据。大多数结构化日志记录器会在将日志条目写入日志文件或发送到集中式日志服务器之前，将其编码为 JSON。结构化日志记录使得将日志收集到集中式服务器的整个过程变得简单，因为与每个日志条目相关的附加元数据使得服务器能够跨服务组织和整理日志条目。一旦它们被索引，你就可以查询日志服务器以找到特定的日志条目，从而更好地找到问题的及时答案。

#### 使用 Zap 日志记录器

讨论具体的集中式日志解决方案超出了本书的范围。如果你有兴趣了解更多，我建议你首先研究 Elasticsearch 或 Apache Solr。不过，本节将重点介绍如何实现日志记录器本身。你将使用来自 Uber 的 Zap 日志记录器，网址是[`pkg.go.dev/go.uber.org/zap/`](https://pkg.go.dev/go.uber.org/zap/)，它允许你集成日志文件轮转功能。

*日志文件轮换*是指在当前日志文件达到特定的年龄或大小阈值后，关闭当前日志文件、重命名它，并打开一个新的日志文件。轮换日志文件是一个好习惯，可以防止它们填满你的硬盘空间。而且，搜索较小的、按日期分隔的日志文件，比搜索一个庞大的日志文件更高效。例如，你可能希望每周轮换一次日志文件，并只保留八周的轮换日志文件。如果你想查看上周发生的事件的日志条目，你可以将搜索范围限制在一个日志文件中。此外，你还可以压缩轮换的日志文件，进一步节省硬盘空间。

我在大型项目中使用过其他结构化日志记录器，根据我的经验，Zap 造成的开销最小；我可以在繁忙的代码部分使用它而不会显著影响性能，和其他重量级的结构化日志记录器不同。但是你的使用情况可能会有所不同，因此我鼓励你找到最适合自己的工具。你可以将这里描述的结构化日志记录原理和日志文件管理技术应用到其他结构化日志记录器上。

Zap 日志记录器包含`zap.Core`及其选项。`zap.Core`有三个组成部分：日志级别阈值、输出和编码器。*日志级别阈值*设置 Zap 记录日志的最低级别；Zap 会忽略低于该级别的日志条目，这样你可以在代码中保留调试日志，并配置 Zap 有条件地忽略它。Zap 的*输出*是一个`zapcore.WriteSyncer`，它是一个带有额外`Sync`方法的`io.Writer`。Zap 可以将日志条目写入任何实现该接口的对象。而*编码器*则可以在将日志条目写入输出之前对其进行编码。

#### 编写编码器

虽然 Zap 提供了一些辅助函数，比如`zap.NewProduction`或`zap.NewDevelopment`，可以快速创建生产和开发环境的日志记录器，但你将从头开始创建一个日志记录器，首先从清单 13-6 中的编码器配置开始。

```
package ch13

import (
    "bytes"
    "fmt"
    "io/ioutil"
    "log"
    "os"
    "path/filepath"
    "runtime"
    "testing"
    "time"

    "go.uber.org/zap"
    "go.uber.org/zap/zapcore"
    "gopkg.in/fsnotify.v1"
 "gopkg.in/natefinch/lumberjack.v2"
)

var encoderCfg = zapcore.EncoderConfig{
    MessageKey: 1"msg",
    NameKey:    2"name",

    LevelKey:    "level",
    EncodeLevel: 3zapcore.LowercaseLevelEncoder,

    CallerKey:    "caller",
    EncodeCaller: 4zapcore.ShortCallerEncoder,

 5 // TimeKey:   "time",
    // EncodeTime: zapcore.ISO8601TimeEncoder,
}
```

清单 13-6：你的 Zap 日志记录器的编码器配置（*zap_test.go*）

编码器配置与编码器本身是独立的，你可以使用相同的编码器配置，无论是将它传递给 JSON 编码器还是控制台编码器。编码器将使用你的配置来决定其输出格式。在这里，你的编码器配置决定了编码器使用`msg`1 作为日志消息的键，使用`name`2 作为日志条目中的日志记录器名称的键。同样，编码器配置要求编码器使用`level`作为日志级别的键，并使用全小写字符 3 来编码级别名称。如果日志记录器被配置为添加调用者详细信息，你希望编码器将这些详细信息与`caller`键关联，并以简化格式 4 对其进行编码。

由于您需要保持以下示例的输出一致，因此将省略 `time` 键 5，以避免它出现在输出中。实际使用时，您应取消注释这两个字段。

#### 创建日志记录器及其选项

现在您已经定义了编码器配置，让我们通过实例化一个 Zap 日志记录器在 示例 13-7 中使用它。

```
`--snip--`

func Example_zapJSON() {
    zl := zap.New(
     1 zapcore.NewCore(
         2 zapcore.NewJSONEncoder(encoderCfg),
         3 zapcore.Lock(os.Stdout),
         4 zapcore.DebugLevel,
        ),
     5 zap.AddCaller(),
        zap.Fields(
         6 zap.String("version", runtime.Version()),
        ),
    )
    defer func() { _ = 7zl.Sync() }()

 example := 8zl.Named("example")
    example.Debug("test debug message")
    example.Info("test info message")

    // Output:
 9 // {"level":"debug","name":"example","caller":"ch13/zap_test.go:49",
"msg":"test debug message","version":"ago1.15.5"}
    // {"level":"info","name":"example","caller":"ch13/zap_test.go:50",
"msg":"test info message","version":"go1.15.5"}
}
```

示例 13-7：根据编码器配置实例化一个新的日志记录器并记录到 JSON 文件中（*zap_test.go*）

`zap.New` 函数接受一个 `zap.Core`1 和零个或多个 `zap.Options`。在此示例中，您传递了 `zap.AddCaller` 选项 5，指示日志记录器在每个日志条目中包含调用者信息，并且包含一个名为 `version` 的字段 6，该字段在每个日志条目中插入运行时版本。

`zap.Core` 包含一个使用您编码器配置的 JSON 编码器 2，一个 `zapcore.WriteSyncer`3 和日志记录阈值 4。如果 `zapcore.WriteSyncer` 不支持并发使用，您可以使用 `zapcore.Lock` 来使其支持并发，如此示例所示。

Zap 日志记录器包括七个日志级别，按严重程度递增：`DebugLevel`、`InfoLevel`、`WarnLevel`、`ErrorLevel`、`DPanicLevel`、`PanicLevel` 和 `FatalLevel`。默认级别是 `InfoLevel`。`DPanicLevel` 和 `PanicLevel` 级别的条目会导致 Zap 记录该条目并引发 panic。在 `FatalLevel` 级别记录的条目会导致 Zap 在写入日志条目后调用 `os.Exit(1)`。由于您的日志记录器使用的是 `DebugLevel`，它将记录所有条目。

我建议您将 `DPanicLevel` 和 `PanicLevel` 限制在开发环境中使用，将 `FatalLevel` 限制在生产环境中，仅用于灾难性启动错误，例如无法连接到数据库。否则，您可能会遇到麻烦。如前所述，您可以充分利用 `DebugLevel`、`ErrorLevel`，偶尔使用 `InfoLevel`。

在开始使用日志记录器之前，您需要确保调用其 `Sync` 方法 7 来确保所有缓存的数据都被写入输出。

您还可以通过调用其 `Named` 方法 8 并使用返回的日志记录器来为日志记录器指定名称。默认情况下，日志记录器没有名称。命名日志记录器将会在日志条目中包含一个名称键，前提是您在编码器配置中定义了一个名称。

现在，日志条目 9 包含了关于日志消息的元数据，以至于日志行的输出超出了本书的宽度。还需要提到的是，示例输出中的 Go 版本 a 取决于您用来测试此示例的 Go 版本。尽管您正在以 JSON 格式编码每个日志条目，您仍然可以读取您在日志中包含的附加元数据。您可以将这些 JSON 导入到像 Elasticsearch 这样的工具中，并在其上执行查询，让 Elasticsearch 处理返回仅与查询相关的日志行的重担。

#### 使用控制台编码器

上面的例子在相对较少的代码中包含了许多功能。现在假设你想记录一些更易于人类阅读的内容，但又具有结构化的内容。Zap 包含一个控制台编码器，基本上是其 JSON 编码器的直接替代品。列表 13-8 使用控制台编码器将结构化日志条目写入标准输出。

```
`--snip--`

func Example_zapConsole() {
    zl := zap.New(
        zapcore.NewCore(
         1 zapcore.NewConsoleEncoder(encoderCfg),
            zapcore.Lock(os.Stdout),
         2 zapcore.InfoLevel,
        ),
    )
    defer func() { _ = zl.Sync() }()

    console := 3zl.Named("[console]")
    console.Info("this is logged by the logger")
 4 console.Debug("this is below the logger's threshold and won't log")
    console.Error("this is also logged by the logger")

    // Output:
 5 // info   [console]   this is logged by the logger
    // error  [console]   this is also logged by the logger
}
```

列表 13-8：使用控制台编码器编写结构化日志（*zap_test.go*）

控制台编码器 1 使用制表符分隔字段。它根据你的编码器配置获取指令，确定要包含哪些字段，以及如何格式化每个字段。

请注意，在这个例子中，你没有将 `zap.AddCaller` 和 `zap.Fields` 选项传递给日志记录器。因此，日志条目将没有 `caller` 和 `version` 字段。只有当日志记录器具有 `zap.AddCaller` 选项，并且编码器配置定义了 `CallerKey` 时，日志条目才会包含 `caller` 字段，正如列表 13-6 所示。

你命名日志记录器为 3，并写入三个日志条目，每个条目具有不同的日志级别。由于日志记录器的阈值是`info`级别 2，因此调试日志条目 4 不会出现在输出中，因为 `debug` 低于 `info` 阈值。

输出 5 缺少键名，但包括由制表符字符分隔的字段值。虽然在打印时不明显，但日志级别、日志名称和日志消息之间有一个制表符字符。如果你在编辑器中输入这个，记得在这些元素之间添加制表符字符，否则运行时示例可能会失败。

#### 使用不同的输出和编码进行日志记录

Zap 包含有用的函数，允许你并行地记录到不同的输出，使用不同的编码，在不同的日志级别上。列表 13-9 创建一个日志记录器，将 JSON 写入日志文件，并将控制台编码写入标准输出。该日志记录器仅将调试日志条目写入控制台。

```
`--snip--`

func Example_zapInfoFileDebugConsole() {
    logFile := 1new(bytes.Buffer)
    zl := zap.New(
        zapcore.NewCore(
            zapcore.NewJSONEncoder(encoderCfg),
            zapcore.Lock(2zapcore.AddSync(logFile)),
            zapcore.InfoLevel,
        ),
    )
    defer func() { _ = zl.Sync() }()

 3 zl.Debug("this is below the logger's threshold and won't log")
    zl.Error("this is logged by the logger")
```

列表 13-9：使用 `*bytes.Buffer` 作为日志输出并将 JSON 写入其中（*zap_test.go*）

你正在使用 `*bytes.Buffer` 1 来充当模拟日志文件。唯一的问题是 `*bytes.Buffer` 没有 `Sync` 方法，并且不实现 `zapcore.WriteSyncer` 接口。幸运的是，Zap 包含一个名为 `zapcore.AddSync` 2 的辅助函数，它会智能地向 `io.Writer` 添加一个无操作的 `Sync` 方法。除了使用这个函数之外，日志记录器的其余实现应该对你来说是熟悉的。它将 JSON 日志记录到日志文件，并排除任何低于 `info` 级别的日志条目。因此，第一个日志条目 3 应该根本不出现在日志文件中。

现在你已经有了一个将 JSON 写入日志文件的日志记录器，让我们使用 Zap 进行实验，并在列表 13-10 中创建一个新的日志记录器，可以同时将 JSON 日志条目写入日志文件，并将控制台日志条目写入标准输出。

```
`--snip--`

    zl = 1zl.WithOptions(
     2 zap.WrapCore(
            func(c zapcore.Core) zapcore.Core {
                ucEncoderCfg := encoderCfg
             3 ucEncoderCfg.EncodeLevel = zapcore.CapitalLevelEncoder
                return 4zapcore.NewTee(
                    c,
                 5 zapcore.NewCore(
                        zapcore.NewConsoleEncoder(ucEncoderCfg),
                        zapcore.Lock(os.Stdout),
                        zapcore.DebugLevel,
                    ),
                )
            },
        ),
 )

    fmt.Println("standard output:")
 6 zl.Debug("this is only logged as console encoding")
    zl.Info("this is logged as console encoding and JSON")

    fmt.Print("\nlog file contents:\n", logFile.String())

    // Output:
    // standard output:
    // DEBUG   this is only logged as console encoding
    // INFO    this is logged as console encoding and JSON
    //
    // log file contents:
    // {"level":"error","msg":"this is logged by the logger"}
    // {"level":"info","msg":"this is logged as console encoding and JSON"}
}
```

列表 13-10：扩展日志记录器以记录到多个输出（*zap_test.go*）

Zap 的 `WithOptions` 方法 1 克隆现有的日志系统，并使用给定的选项配置该克隆。你可以使用 `zap.WrapCore` 函数 2 来修改克隆日志系统的底层 `zap.Core`。为了有所不同，你会复制编码器配置并调整它，指示编码器以全大写字母输出日志级别 3。最后，你使用 `zapcore.NewTee` 函数，它类似于 `io.MultiWriter` 函数，用于返回一个将日志写入多个核心的 `zap.Core` 4。在这个示例中，你传入了现有的核心和一个新的核心 5，后者将 `debug` 级别的日志条目写入标准输出。

当你使用克隆的日志系统时，日志文件和标准输出都会接收 `info` 级别及以上的日志条目，而只有标准输出接收调试日志条目 6。

#### 抽样日志条目

关于日志记录，我给你的一个警告是要考虑它如何影响你的应用程序的 CPU 和 I/O 性能。你不希望日志记录成为你应用程序的瓶颈。这通常意味着在应用程序繁忙部分进行日志记录时要特别小心。

在关键代码路径（如循环）中，减少日志开销的一种方法是对日志条目进行抽样。可能并不需要记录每个条目，尤其是当你的日志系统输出了许多重复的日志条目时。相反，尝试记录每个重复条目的第*n*次出现。

方便的是，Zap 提供了一个可以做到这一点的日志系统。示例 13-11 创建了一个日志系统，通过记录一部分日志条目来限制其 CPU 和 I/O 开销。

```
`--snip--`

func Example_zapSampling() {
    zl := zap.New(
     1 zapcore.NewSamplerWithOptions(
 zapcore.NewCore(
                zapcore.NewJSONEncoder(encoderCfg),
                zapcore.Lock(os.Stdout),
                zapcore.DebugLevel,
            ),
          2time.Second, 31, 43,
        ),
    )
    defer func() { _ = zl.Sync() }()

    for i := 0; i < 10; i++ {
        if i == 5 {
         5 time.Sleep(time.Second)
        }
     6 zl.Debug(fmt.Sprintf("%d", i))
     7 zl.Debug("debug message")
    }

    // 8Output:
    // {"level":"debug","msg":"0"}
    // {"level":"debug","msg":"debug message"}
    // {"level":"debug","msg":"1"}
    // {"level":"debug","msg":"2"}
    // {"level":"debug","msg":"3"}
    // {"level":"debug","msg":"debug message"}
    // {"level":"debug","msg":"4"}
    // {"level":"debug","msg":"5"}
    // {"level":"debug","msg":"debug message"}
    // {"level":"debug","msg":"6"}
    // {"level":"debug","msg":"7"}
    // {"level":"debug","msg":"8"}
    // {"level":"debug","msg":"debug message"}
    // {"level":"debug","msg":"9"}
}
```

示例 13-11：记录部分日志条目以限制 CPU 和 I/O 开销 (*zap_test.go*)

`NewSamplerWithOptions` 函数 1 用抽样功能包装了 `zap.Core`。它需要三个额外的参数：一个抽样间隔 2，记录的初始重复日志条目数量 3，以及一个整数 4，表示在此之后要记录的第*n*个重复日志条目。在这个示例中，你首先记录第一个日志条目，然后记录在一秒钟内，日志系统接收到的每第三个重复日志条目。间隔时间结束后，日志系统重新开始，记录第一个条目，然后记录余下时间内的每第三个重复条目。

让我们看一下实际操作。你在循环中执行 10 次迭代。每次迭代都会记录计数器 6 和一个通用的调试信息 7，后者在每次迭代中保持不变。在第六次迭代时，示例会休眠一秒钟 5，以确保样本日志系统在接下来的一个秒钟间隔中重新开始记录。

检查输出 8，您会看到调试消息在第一次迭代时打印出来，直到日志记录器在第四次循环迭代中遇到第三个重复的调试消息才会再次打印。但在第六次迭代时，示例进入休眠，示例日志记录器切换到下一个一秒钟的时间间隔，重新开始记录。在第六次循环迭代中，它记录了该时间间隔的第一个调试消息，而在第九次循环迭代中记录了第三个重复的调试消息。

诚然，这是一个人为的示例，但它展示了如何使用这种日志采样技术作为一种折衷方法，适用于 CPU 和 I/O 敏感的代码部分。这个技术可以应用于将工作分配给工作 goroutine 的场景。尽管您可以将工作尽可能快速地发送给工作 goroutine 处理，但您可能希望定期更新每个工作 goroutine 的进度，而不需要付出过多的日志记录开销。示例日志记录器可以让您调节日志输出，找到及时更新与最小开销之间的平衡。

#### 执行按需调试日志记录

如果调试日志记录在正常操作下对您的应用程序造成了不可接受的负担，或者调试日志数据的数量过多以至于超过了可用存储空间，您可能需要按需启用调试日志记录。一种方法是使用信号量文件来启用调试日志记录。*信号量文件*是一个空文件，其存在作为日志记录器改变行为的信号。如果信号量文件存在，日志记录器将输出`debug`级别的日志。一旦您移除信号量文件，日志记录器将恢复到先前的日志级别。

让我们使用`fsnotify`包来允许您的应用程序监视文件系统通知。除了标准库外，`fsnotify`包还使用了`x/sys`包。在开始编写代码之前，让我们确保我们的`x/sys`包是最新的：

```
$ **go get -u golang.org/x/sys/...**
```

并非所有日志记录包都提供安全的异步修改日志级别的方法。请注意，如果您尝试在日志记录器读取日志级别的同时修改其级别，可能会引发竞争条件。Zap 日志记录器允许您检索一个基于`sync/atomic`的级别器，以便在避免竞争条件的情况下动态修改日志记录器的级别。您将把原子级别器传递给`zapcore.NewCore`函数，以替代日志级别，就像您之前做的那样。

`zap.AtomicLevel`结构实现了`http.Handler`接口。您可以将它集成到 API 中，并通过 HTTP 动态更改日志级别，而不需要使用信号量。

列表 13-12 展示了一个使用信号量文件的动态日志记录示例。您将在接下来的几个列表中实现这个示例。

```
`--snip--`

func Example_zapDynamicDebugging() {
    tempDir, err := ioutil.TempDir("", "")
    if err != nil {
        log.Fatal(err)
    }
 defer func() { _ = os.RemoveAll(tempDir) }()

    debugLevelFile := 1filepath.Join(tempDir, "level.debug")
    atomicLevel := 2zap.NewAtomicLevel()

    zl := zap.New(
        zapcore.NewCore(
            zapcore.NewJSONEncoder(encoderCfg),
            zapcore.Lock(os.Stdout),
         3 atomicLevel,
        ),
    )
    defer func() { _ = zl.Sync() }()
```

列表 13-12：使用原子级别的日志记录器创建新的日志记录器（*zap_test.go*）

你的代码将监视临时目录中是否存在*level.debug*文件 1。当文件存在时，你将动态地将记录器的级别更改为`debug`。为此，你需要一个新的原子级别器 2。默认情况下，原子级别器使用`info`级别，这对于这个例子来说完全合适。你在创建核心时传入原子级别器 3，而不是直接指定日志级别。

现在你已经有了一个原子级别器和一个存储信号量文件的位置，让我们编写代码来监视信号量文件的变化，见清单 13-13。

```
`--snip--`

    watcher, err := 1fsnotify.NewWatcher()
    if err != nil {
        log.Fatal(err)
    }
    defer func() { _ = watcher.Close() }()

    err = 2watcher.Add(tempDir)
    if err != nil {
        log.Fatal(err)
    }

    ready := make(chan struct{})
    go func() {
        defer close(ready)

        originalLevel := 3atomicLevel.Level()

        for {
            select {
            case event, ok := 4<-watcher.Events:
                if !ok {
                    return
                }
                if event.Name == 5debugLevelFile {
                    switch {
                    case event.Op&fsnotify.Create == 6fsnotify.Create:
                        atomicLevel.SetLevel(zapcore.DebugLevel)
 ready <- struct{}{}
                    case event.Op&fsnotify.Remove == 7fsnotify.Remove:
                        atomicLevel.SetLevel(originalLevel)
                        ready <- struct{}{}
                    }
                }
            case err, ok := 8<-watcher.Errors:
                if !ok {
                    return
                }
                zl.Error(err.Error())
            }
        }
    }()
```

清单 13-13：监视信号量文件的任何变化（*zap_test.go*）

首先，你创建一个文件系统监视器 1，用于监视临时目录 2。监视器将通知你该目录中的任何变化。你还需要捕获当前的日志级别 3，以便在删除信号量文件时恢复到该级别。

接下来，你监听监视器 4 的事件。由于你监视的是一个目录，你需要过滤掉与信号量文件 5 本身无关的任何事件。即使如此，你只对信号量文件的创建或删除事件感兴趣。如果事件表示信号量文件的创建 6，你将原子级别器的级别更改为`debug`。如果你收到信号量文件删除事件 7，你将原子级别器的级别恢复为原始级别。

如果你在任何时候从监视器 8 接收到错误，你应该以`error`级别记录该错误。

让我们看看它在实践中的表现。清单 13-14 测试了记录器在有信号量文件和没有信号量文件时的行为。

```
`--snip--`

 1 zl.Debug("this is below the logger's threshold")

    df, err := 2os.Create(debugLevelFile)
    if err != nil {
        log.Fatal(err)
    }
    err = df.Close()
    if err != nil {
        log.Fatal(err)
    }
    <-ready

 3 zl.Debug("this is now at the logger's threshold")

    err = 4os.Remove(debugLevelFile)
    if err != nil {
        log.Fatal(err)
    }
    <-ready

 5 zl.Debug("this is below the logger's threshold again")
 6 zl.Info("this is at the logger's current threshold")

    // Output:
    // {"level":"debug","msg":"this is now at the logger's threshold"}
    // {"level":"info","msg":"this is at the logger's current threshold"}
}
```

清单 13-14：测试记录器使用信号量文件（*zap_test.go*）

记录器当前通过原子级别器的日志级别是`info`。因此，记录器不会将初始调试日志条目 1 写入标准输出。但是，如果你创建信号量文件 2，位于清单 13-13 中的代码应该会动态地将记录器的级别更改为`debug`。如果你再添加另一条调试日志条目 3，记录器应该将其写入标准输出。然后你删除信号量文件 4，并同时写入调试日志条目 5 和信息日志条目 6。由于信号量文件不再存在，记录器应该只将信息日志条目写入标准输出。

### 使用宽事件日志记录进行扩展

*宽事件日志记录*是一种技术，它为每个事件创建一个单一的、结构化的日志条目，以总结事务，而不是随着事务的进行记录大量的条目。这种技术最适用于请求-响应循环，例如 API 调用，但它也可以适应其他用例。当你在结构化的日志条目中总结事务时，你可以减少日志记录的开销，同时保持索引和搜索事务细节的能力。

广泛事件日志的一种方法是将 API 处理器包装在中间件中。但首先，由于`http.ResponseWriter`对其输出比较“吝啬”，你需要创建自己的响应写入器类型（列表 13-15），以便收集并记录响应码和长度。

```
package ch13

import (
    "io"
    "io/ioutil"
    "net"
    "net/http"
    "net/http/httptest"
    "os"

    "go.uber.org/zap"
    "go.uber.org/zap/zapcore"
)

type wideResponseWriter struct {
 1 http.ResponseWriter
    length, status int
}

func (w *wideResponseWriter) 2WriteHeader(status int) {
    w.ResponseWriter.WriteHeader(status)
    w.status = status
}

func (w *wideResponseWriter) 3Write(b []byte) (int, error) {
    n, err := w.ResponseWriter.Write(b)
    w.length += n

    if w.status == 0 {
        w.status = 4http.StatusOK
    }

    return n, err
}
```

列表 13-15：创建一个`ResponseWriter`以捕获响应状态码和长度（*wide_test.go*）

新类型嵌入了实现`http.ResponseWriter`接口的对象 1。此外，你添加了`length`和`status`字段，因为这些值最终是你想从响应中记录的内容。你重写了`WriteHeader`方法 2，以便轻松捕获状态码。同样，你重写了`Write`方法 3，以准确记录写入字节的数量，并在调用者在`WriteHeader`之前执行`Write`时，可选择设置状态码 4。

列表 13-16 在广泛事件日志中间件中使用了你新的类型。

```
`--snip--`

func WideEventLog(logger *zap.Logger, next http.Handler) http.Handler {
    return http.HandlerFunc(
        func(w http.ResponseWriter, r *http.Request) {
            wideWriter := 1&wideResponseWriter{ResponseWriter: w}

         2 next.ServeHTTP(wideWriter, r)

            addr, _, _ := net.SplitHostPort(r.RemoteAddr)
         3 logger.Info("example wide event",
                zap.Int("status_code", wideWriter.status),
                zap.Int("response_length", wideWriter.length),
                zap.Int64("content_length", r.ContentLength),
                zap.String("method", r.Method),
                zap.String("proto", r.Proto),
                zap.String("remote_addr", addr),
                zap.String("uri", r.RequestURI),
                zap.String("user_agent", r.UserAgent()),
            )
        },
    )
}
```

列表 13-16：实现广泛事件日志中间件（*wide_test.go*）

广泛事件日志中间件同时接受`*zap.Logger`和`http.Handler`，并返回一个`http.Handler`。如果这个模式对你来说不熟悉，请阅读第 193 页的“处理器”部分。

首先，你将`http.ResponseWriter`嵌入到一个新的广泛事件日志感知响应写入器实例中 1。然后，你调用下一个`http.Handler`的`ServeHTTP`方法 2，传入你的响应写入器。最后，你创建一个日志条目 3，记录关于请求和响应的各种数据。

请记住，我在这里特意忽略了那些每次执行时会变化并打破示例输出的值，例如调用时长。在实际实现中，你可能需要编写代码来处理这些值。

列表 13-17 启动中间件并展示预期的输出。

```
`--snip--`

func Example_wideLogEntry() {
    zl := zap.New(
        zapcore.NewCore(
            zapcore.NewJSONEncoder(encoderCfg),
            zapcore.Lock(os.Stdout),
            zapcore.DebugLevel,
        ),
    )
    defer func() { _ = zl.Sync() }()

    ts := httptest.NewServer(
     1 WideEventLog(zl, http.HandlerFunc(
            func(w http.ResponseWriter, r *http.Request) {
                defer func(r io.ReadCloser) {
                    _, _ = io.Copy(ioutil.Discard, r)
                    _ = r.Close()
                }(r.Body)
                _, _ = 2w.Write([]byte("Hello!"))
            },
        )),
    )
    defer ts.Close()

    resp, err := 3http.Get(ts.URL + "/test")
    if err != nil {
     4 zl.Fatal(err.Error())
    }
    _ = resp.Body.Close()

    // 5Output:
    // {"level":"info","msg":"example wide event","status_code":200,
"response_length":6,"content_length":0,"method":"GET","proto":"HTTP/1.1",
"remote_addr":"127.0.0.1","uri":"/test","user_agent":"Go-http-client/1.1"}
}
```

列表 13-17：使用广泛事件日志中间件记录 GET 调用的详细信息（*wide_test.go*）

如第九章所述，你使用`httptest`服务器与`WideEventLog`中间件 1 一起工作。你将`*zap.Logger`作为第一个参数传递给中间件，`http.Handler`作为第二个参数。处理器将简单的*Hello!*写入响应 2，因此响应长度是非零的。通过这种方式，你可以证明你的响应写入器正常工作。日志记录器会在接收到 GET 请求的响应之前立即写入日志条目 3。如之前所述，我必须将 JSON 输出 5 包装起来以便在本书中打印，但它通常只占用一行。

由于这只是一个示例，我选择使用记录器的`Fatal`方法 4，该方法将错误信息写入日志文件并调用`os.Exit(1)`终止应用程序。如果代码需要在发生错误时继续运行，你不应该在代码中使用这个方法。

### 使用 Lumberjack 进行日志轮换

如果你选择将日志条目输出到文件，你可以使用像*logrotate*这样的应用程序，以防它们占用所有可用的硬盘空间。使用第三方应用程序管理日志文件的缺点是，第三方应用程序需要向你的应用程序发出信号，提示它重新打开日志文件句柄，否则你的应用程序将继续写入已轮换的日志文件。

一个不那么侵入性且更可靠的选择是直接将日志文件管理添加到你的日志记录器中，使用像*Lumberjack*这样的库。Lumberjack 以透明的方式处理日志轮换，因为你的日志记录器将 Lumberjack 视为任何其他`io.Writer`。与此同时，Lumberjack 为你跟踪日志条目的记账和文件轮换。

列表 13-18 为典型的 Zap 日志记录实现添加了日志轮换功能。

```
`--snip--`

func TestZapLogRotation(t *testing.T) {
    tempDir, err := ioutil.TempDir("", "")
    if err != nil {
        t.Fatal(err)
    }
    defer func() { _ = os.RemoveAll(tempDir) }()

    zl := zap.New(
        zapcore.NewCore(
            zapcore.NewJSONEncoder(encoderCfg),
         1 zapcore.AddSync(
             2 &lumberjack.Logger{
                    Filename:   3filepath.Join(tempDir, "debug.log"),
                    Compress:   4true,
                    LocalTime:  5true,
                    MaxAge:     67,
                    MaxBackups: 75,
                    MaxSize:    8100,
                },
            ),
            zapcore.DebugLevel,
        ),
    )
 defer func() { _ = zl.Sync() }()

    zl.Debug("debug message written to the log file")
}
```

列表 13-18：使用 Lumberjack 为 Zap 日志记录器添加日志轮换功能（*zap_test.go*）

与列表 13-9 中的`*bytes.Buffer`一样，`*lumberjack.Logger`2 没有实现`zapcore.WriteSyncer`。它同样缺少`Sync`方法。因此，你需要将其封装在对`zapcore.AddSync`1 的调用中。

Lumberjack 包含多个字段来配置其行为，尽管其默认值是合理的。它使用格式为*<processname>-lumberjack.log*的日志文件名，并保存在临时目录中，除非你明确指定日志文件名 3。你还可以选择节省硬盘空间，并让 Lumberjack 压缩 4 已轮换的日志文件。每个已轮换的日志文件默认使用 UTC 时间戳，但你可以指示 Lumberjack 改为使用本地时间 5。最后，你可以配置日志文件在轮换前的最大年龄 6、要保留的最大轮换日志文件数量 7，以及日志文件达到的最大大小（以 MB 为单位）8。

你可以继续像直接写入标准输出或`*os.File`一样使用日志记录器。不同之处在于，Lumberjack 会智能地为你处理日志文件的管理。

## 给代码添加日志

*为代码添加指标*是收集指标的过程，目的是推断你服务的当前状态——例如每个请求响应循环的持续时间、每个响应的大小、连接的客户端数量、你的服务和第三方 API 之间的延迟等等。日志提供了记录服务如何进入某种状态的过程，而指标则让你深入了解该状态本身。

仪表化很简单，甚至我要给你与我给日志记录相反的建议：最初，仪表化所有内容。细粒度的仪表化几乎没有任何开销，它高效且易于传输，存储成本低廉。而且，仪表化能够解决我之前提到的日志记录中的一个挑战：你最初并不知道你将要问的所有问题，尤其是在复杂的系统中。一个隐蔽的问题可能会因你缺乏关键的指标而破坏你的周末，它会给你一个早期的警告，提醒你出现了问题。

本节将介绍指标类型，并展示如何在你的服务中使用这些类型的基础知识。你将了解 Go kit 的`metrics`包，它是一个抽象层，提供了流行指标平台的有用接口。你将通过使用 Prometheus 作为目标指标平台并设置 Prometheus 抓取的端点来完成仪表化。如果将来选择使用不同的平台，你只需要替换代码中的 Prometheus 相关部分；你可以保留 Go kit 代码不变。如果你刚开始进行仪表化，使用[`grafana.com/products/cloud/`](https://grafana.com/products/cloud/)来抓取并可视化你的指标是一个不错的选择。它的免费层足以进行仪表化实验。

### 设置

为了抽象化你的指标实现及其依赖的包，让我们首先将它们放在自己的包中（清单 13-19）。

```
package metrics

import (
    "flag"

 1 "github.com/go-kit/kit/metrics"
 2 "github.com/go-kit/kit/metrics/prometheus"
 3 prom "github.com/prometheus/client_golang/prometheus"
)

var (
    Namespace = 4flag.String("namespace", "web", "metrics namespace")
    Subsystem = 5flag.String("subsystem", "server1", "metrics subsystem")
```

清单 13-19：指标示例的导入和命令行标志 (*instrumentation/metrics/metrics.go*)

你导入 Go kit 的`metrics`包 1，它提供你的代码将使用的接口，`prometheus`适配器 2，使你能够将 Prometheus 作为你的指标平台，以及 Go 的 Prometheus 客户端包 3 本身。所有与 Prometheus 相关的导入都位于此包中。你的其余代码将使用 Go kit 的接口。这样，你可以在不需要更改代码中的仪表化部分的情况下，交换底层的指标平台。

Prometheus 会在其指标前加上命名空间和子系统的前缀。你可以使用服务名称作为命名空间，节点或主机名作为子系统，例如。在此示例中，默认情况下，你将使用`web`作为命名空间 4，`server1`作为子系统 5。因此，你的指标将使用`web_server1_`前缀。你将在清单 13-30 的命令行输出中看到这个前缀。

现在让我们探索各种指标类型，从计数器开始。

### 计数器

*计数器*用于跟踪仅增加的值，例如请求计数、错误计数或完成任务计数。你可以使用计数器来计算给定时间间隔内的增长速率，例如每分钟连接数。

示例 13-20 定义了两个计数器：一个用于跟踪请求数量，另一个用于记录写入错误的数量。

```
`--snip--`

    Requests 1metrics.Counter = 2prometheus.NewCounterFrom(
     3 prom.CounterOpts{
            Namespace: *Namespace,
            Subsystem: *Subsystem,
            Name:    4"request_count",
            Help:    5"Total requests",
        },
        []string{},
    )

    WriteErrors metrics.Counter = prometheus.NewCounterFrom(
        prom.CounterOpts{
            Namespace: *Namespace,
            Subsystem: *Subsystem,
            Name:      "write_errors_count",
            Help:      "Total write errors",
        },
        []string{},
    )
```

示例 13-20：作为 Go kit 接口创建计数器 (*instrumentation/metrics/metrics.go*)

每个计数器都实现了 Go kit 的 `metrics.Counter` 接口 1。每个计数器的具体类型来自 Go kit 的 `prometheus` 适配器 2，并依赖于 Prometheus 客户端包中的 `CounterOpts` 结构体 3 进行配置。除了我们已经讨论过的命名空间和子系统值外，另一个重要的值是你设置的度量名称 4 及其帮助字符串 5，后者描述了该度量。

### 仪表

*仪表* 允许你跟踪增减的值，例如当前内存使用情况、在途请求、队列大小、风扇转速或我桌面上 ThinkPad 的数量。仪表不支持速率计算，例如每分钟的连接数或每秒传输的兆比特，而计数器则支持。

示例 13-21 创建了一个仪表来跟踪开放的连接数。

```
`--snip--`

    OpenConnections 1metrics.Gauge = 2prometheus.NewGaugeFrom(
     3 prom.GaugeOpts{
            Namespace: *Namespace,
            Subsystem: *Subsystem,
            Name:      "open_connections",
            Help:      "Current open connections",
        },
        []string{},
    )
```

示例 13-21：作为 Go kit 接口创建仪表 (*instrumentation/metrics/metrics.go*)

创建仪表与创建计数器类似。你需要创建一个新的 Go kit 的 `metrics.Gauge` 接口变量，并使用 Go kit 的 `prometheus` 适配器中的 `NewGaugeFrom` 函数 2 来创建底层类型。Prometheus 客户端的 `GaugeOpts` 结构体 3 提供了新仪表的设置。

### 直方图和摘要

*直方图* 将值放入预定义的桶中。每个桶都与一个值的范围相关联，并以其最大值命名。当观察到一个值时，直方图会增加最小桶中可以容纳该值的最大值。通过这种方式，直方图跟踪每个桶中观察到的值的频率。

让我们看一个简单的例子。假设你有三个桶，值分别为 0.5、1.0 和 1.5，如果直方图观察到值为 0.42，它将增加与桶 0.5 相关联的计数器，因为 0.5 是可以容纳 0.42 的最小桶。它覆盖了 0.5 及以下的范围。如果直方图观察到值为 1.23，它将增加与桶 1.5 相关联的计数器，桶 1.5 覆盖了 1.0 到 1.5 之间的值。自然地，桶 1.0 覆盖了 0.5 到 1.0 之间的范围。

你可以使用直方图的观察值分布来估算所有值的百分比或平均值。例如，你可以使用直方图来计算服务中观察到的平均请求大小或响应大小。

*摘要* 是一个直方图，有几个不同之处。首先，直方图需要预定义的桶，而摘要会计算自己的桶。其次，度量服务器从直方图计算平均值或百分比，而你的服务从摘要中计算平均值或百分比。因此，你可以在度量服务器上跨服务聚合直方图，但无法对摘要执行相同的操作。

一般建议是当你不知道预期值的范围时使用总结性指标，但我建议尽可能使用直方图，这样你可以在指标服务器上聚合直方图。让我们使用直方图来观察请求时长（参见列表 13-22）。

```
`--snip--`

    RequestDuration 1metrics.Histogram = 2prometheus.NewHistogramFrom(
     3 prom.HistogramOpts{
            Namespace: *Namespace,
            Subsystem: *Subsystem,
            Buckets: 4[]float64{
                0.0000001, 0.0000002, 0.0000003, 0.0000004, 0.0000005,
                0.000001, 0.0000025, 0.000005, 0.0000075, 0.00001,
                0.0001, 0.001, 0.01,
            },
            Name: "request_duration_histogram_seconds",
            Help: "Total duration of all requests",
        },
        []string{},
    )
)
```

列表 13-22：创建直方图指标（*instrumentation/metrics/metrics.go*）

总结性指标和直方图指标类型都实现了 Go kit 的`metrics.Histogram`接口 1，这是通过其`prometheus`适配器提供的。在这里，你使用的是直方图指标类型 2，并通过 Prometheus 客户端的`HistogramOpts`结构体 3 进行配置。由于 Prometheus 的默认桶大小对于通过 localhost 进行通信时预期的请求时长范围来说太大，你定义了自定义的桶大小 4。我鼓励你尝试不同数量的桶和桶大小。

如果你更愿意将`RequestDuration`实现为总结性指标，可以将列表 13-22 中的代码替换为列表 13-23 中的代码。

```
`--snip--`

    RequestDuration 1metrics.Histogram = prometheus.NewSummaryFrom(
        prom.SummaryOpts{
            Namespace: *Namespace,
            Subsystem: *Subsystem,
            Name: "request_duration_summary_seconds",
            Help: "Total duration of all requests",
        },
        []string{},
    )
)
```

列表 13-23：可选地创建一个总结性指标

如你所见，这看起来很像一个直方图，除了`Bucket`方法。请注意，你仍然使用`metrics.Histogram`接口 1 来处理 Prometheus 的总结性指标。这是因为 Go kit 并不区分直方图和总结性指标，只有你的接口实现才会区分。

## 仪表化基础 HTTP 服务器

让我们在一个实际示例中结合这些指标类型：对一个 Go HTTP 服务器进行仪表化。这里最大的挑战是确定你要监控什么，最合适的监控位置在哪里，以及每个你想追踪的值最适合使用什么类型的指标。如果你使用 Prometheus 作为你的指标平台，就像在这里一样，你还需要为 Prometheus 服务器添加一个 HTTP 端点来抓取数据。

列表 13-24 详细介绍了应用程序所需的初始代码，该应用程序包括一个 HTTP 服务器来提供指标端点，另一个 HTTP 服务器将所有请求传递给一个仪表化的处理程序。

```
package main

import (
    "bytes"
    "flag"
    "fmt"
 "io"
    "io/ioutil"
    "log"
    "math/rand"
    "net"
    "net/http"
    "sync"
    "time"

 1 "github.com/prometheus/client_golang/prometheus/promhttp"

 2 "github.com/awoodbeck/gnp/ch13/instrumentation/metrics"
)

var (
    metricsAddr = 3flag.String("metrics", "127.0.0.1:8081",
        "metrics listen address")
    webAddr = 4flag.String("web", "127.0.0.1:8082", "web listen address")
)
```

列表 13-24：指标示例的导入和命令行标志（*instrumentation/main.go*）

你的代码所需的唯一导入是`promhttp`包（用于指标端点）和你的`metrics`包（用于仪表化你的代码）。`promhttp`包 1 包含一个`http.Handler`，Prometheus 服务器可以使用它从你的应用程序抓取指标。这个处理程序不仅提供你的指标，还提供与运行时相关的指标，比如 Go 版本、核心数量等等。至少，你可以使用 Prometheus 处理程序提供的指标来洞察服务的内存利用率、打开的文件描述符、堆栈详细信息等。

你`metrics`包中导出的所有变量 2 都是 Go kit 接口。你的代码无需关心底层的度量平台或其实现，只需关注这些度量如何提供给度量服务器。在实际应用中，你可以进一步抽象 Prometheus 处理程序，以便完全消除除了你的度量包以外的其他依赖。但为了保持本示例简洁，我将 Prometheus 处理程序包含在了`main`包中。

接下来是你要进行仪表化的代码。列表 13-25 添加了你的 Web 服务器将用于处理所有传入请求的函数。

```
`--snip--`

func helloHandler(w http.ResponseWriter, _ *http.Request) {
 1 metrics.Requests.Add(1)
    defer func(start time.Time) {
     2 metrics.RequestDuration.Observe(time.Since(start).Seconds())
    }(time.Now())

 _, err := w.Write([]byte("Hello!"))
    if err != nil {
     3 metrics.WriteErrors.Add(1)
    }
}
```

列表 13-25：一个带有随机延迟的仪表化处理程序（*instrumentation/main.go*）

即便是这样一个简单的处理程序，你也能进行三项有意义的度量。你在进入处理程序 1 时增加请求计数器，因为这是记录它的最合适位置。你还立即延迟一个函数来计算请求持续时间，并使用请求持续时间的汇总度量进行观察 2。最后，你会记录写响应时出现的任何错误 3。

现在，你需要使用这个处理程序。但首先，你需要一个帮助函数，允许你启动几个 HTTP 服务器：一个用于提供 metrics 端点，另一个用于提供这个处理程序。列表 13-26 详细描述了这样的一个函数。

```
`--snip--`

func newHTTPServer(addr string, mux http.Handler,
    stateFunc 1func(net.Conn, http.ConnState)) error {
    l, err := net.Listen("tcp", addr)
    if err != nil {
        return err
    }

    srv := &http.Server{
        Addr:              addr,
        Handler:           mux,
        IdleTimeout:       time.Minute,
        ReadHeaderTimeout: 30 * time.Second,
        ConnState:         stateFunc,
    }

    go func() { log.Fatal(srv.Serve(l)) }()

    return nil
}

func 2connStateMetrics(_ net.Conn, state http.ConnState) {
    switch state {
    case http.StateNew:
     3 metrics.OpenConnections.Add(1)
    case http.StateClosed:
     4 metrics.OpenConnections.Add(-1)
    }
}
```

列表 13-26：创建 HTTP 服务器并监控连接状态的函数（*instrumentation/main.go*）

这个 HTTP 服务器代码类似于第九章的代码。唯一的不同是你在此定义了服务器的`ConnState`字段，并将其作为参数 1 传递给`newHTTPServer`函数。

HTTP 服务器每当网络连接发生变化时，都会调用其`ConnState`字段。你可以利用这个功能来监控服务器在任何时刻的开放连接数量。每当你想初始化一个新的 HTTP 服务器并跟踪其开放连接时，可以将`connStateMetrics`函数 2 传递给`newHTTPServer`函数。如果服务器建立了新连接，你会将开放连接计量器 3 增加 1。如果连接关闭，你会将计量器 4 减少 1。Go kit 的计量器接口提供了`Add`方法，因此减少一个值实际上是加上一个负数。

让我们创建一个例子，将所有这些部分结合在一起。列表 13-27 创建了一个 HTTP 服务器来提供 Prometheus 端点，并创建了另一个 HTTP 服务器来提供你的仪表化处理程序。

```
`--snip--`

func main() {
    flag.Parse()
    rand.Seed(time.Now().UnixNano())

    mux := http.NewServeMux()
 1 mux.Handle("/metrics/", promhttp.Handler())
    if err := newHTTPServer(*metricsAddr, mux, 2nil); err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Metrics listening on %q ...\n", *metricsAddr)

    if err := newHTTPServer(*webAddr, 3http.HandlerFunc(helloHandler),
      4connStateMetrics); err != nil {
        log.Fatal(err)
    }
    fmt.Printf("Web listening on %q ...\n\n", *webAddr)
```

列表 13-27：启动两个 HTTP 服务器来提供`metrics`和`helloHandler`（*instrumentation/main.go*）

首先，你启动一个 HTTP 服务器，唯一的目的是在`/metrics/`端点提供 Prometheus 处理程序 1，默认情况下 Prometheus 会从这个端点抓取指标。由于你没有为第三个参数 2 传入函数，这个 HTTP 服务器不会为它的`ConnState`字段分配一个在每次连接状态变化时调用的函数。然后，你启动另一个 HTTP 服务器，使用`helloHandler`3 来处理每个请求。但这次，你传入了`connStateMetrics`函数 4。结果，这个 HTTP 服务器将衡量开放连接。

现在，你可以启动多个 HTTP 客户端，发送大量请求来影响你的指标（参见 Listing 13-28）。

```
`--snip--`

    clients := 1500
    gets := 2100
    wg := new(sync.WaitGroup)

    fmt.Printf("Spawning %d connections to make %d requests each ...",
        clients, gets)
    for i := 0; i < clients; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()

            c := &http.Client{
                Transport: 3http.DefaultTransport.(*http.Transport).Clone(),
            }

            for j := 0; j < gets; j++ {
                resp, err := 4c.Get(fmt.Sprintf("http://%s/", *webAddr))
                if err != nil {
                    log.Fatal(err)
                }
                _, _ = 5io.Copy(ioutil.Discard, resp.Body)
                _ = 6resp.Body.Close()
            }
        }()
    }
 7 wg.Wait()
    fmt.Print(" done.\n\n")
```

Listing 13-28: 指示 500 个 HTTP 客户端每个执行 100 次 GET 请求（*instrumentation/main.go*）

首先，你会启动 500 个 HTTP 客户端，每个客户端进行 100 次 GET 请求。但是在此之前，你需要解决一个问题。`http.Client`会使用`http.DefaultTransport`，如果它的`Transport`方法为 nil。`http.DefaultTransport`在缓存 TCP 连接方面表现出色。如果所有 500 个 HTTP 客户端使用相同的传输，它们将通过大约两个 TCP 套接字进行通信。当你完成这个示例时，我们的开放连接指标将反映出这两个空闲连接，而这并不是我们的目标。

但你必须确保每个 HTTP 客户端都有自己的传输。克隆默认传输 3 对于我们的目的已经足够。

现在，每个客户端都有自己的传输，并且你可以确保每个客户端都会建立自己的 TCP 连接，你可以让每个客户端通过 GET 请求 4 进行 100 次迭代。你还必须小心地清理 5 和关闭 6 响应体，以便每个客户端可以重用其 TCP 连接。

一旦所有 500 个 HTTP 客户端完成它们的 100 次请求 7，你就可以继续查看 Listing 13-29，检查当前的指标状态。

```
`--snip--`

    resp, err := 1http.Get(fmt.Sprintf("http://%s/metrics", *metricsAddr))
    if err != nil {
        log.Fatal(err)
    }

    b, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        log.Fatal(err)
    }
    _ = resp.Body.Close()

    metricsPrefix := 2fmt.Sprintf("%s_%s", *metrics.Namespace,
        *metrics.Subsystem)
    fmt.Println("Current Metrics:")
    for _, line := range bytes.Split(b, []byte("\n")) {
        if 3bytes.HasPrefix(line, []byte(metricsPrefix)) {
            fmt.Printf("%s\n", line)
        }
    }
}
```

Listing 13-29: 显示匹配你命名空间和子系统的当前指标（*instrumentation/main.go*）

你从指标端点 1 检索所有的指标。这将导致指标 Web 服务器返回 Prometheus 客户端存储的所有指标，除了它跟踪的每个指标的详细信息外，还包括你添加的指标。由于你只对自己的指标感兴趣，你可以检查每一行，以你的命名空间、下划线和子系统 2 开头的行。如果该行匹配这个前缀 3，你就将其打印到标准输出。否则，你忽略该行并继续。

让我们在命令行上运行这个示例，并检查在 Listing 13-30 中生成的指标。

```
$ **go run instrumentation/main.go**
Metrics listening on "127.0.0.1:8081" ...
Web listening on "127.0.0.1:8082" ...

Spawning 500 connections to make 100 requests each ... done.

Current Metrics:
web_server1_open_connections 1500
web_server1_request_count 250000
web_server1_request_duration_histogram_seconds_bucket{le="1e-07"} 30
web_server1_request_duration_histogram_seconds_bucket{le="2e-07"} 1
web_server1_request_duration_histogram_seconds_bucket{le="3e-07"} 613
web_server1_request_duration_histogram_seconds_bucket{le="4e-07"} 13591
web_server1_request_duration_histogram_seconds_bucket{le="5e-07"} 33216
web_server1_request_duration_histogram_seconds_bucket{le="1e-06"} 40183
web_server1_request_duration_histogram_seconds_bucket{le="2.5e-06"} 49876
web_server1_request_duration_histogram_seconds_bucket{le="5e-06"} 49963
web_server1_request_duration_histogram_seconds_bucket{le="7.5e-06"} 49973
web_server1_request_duration_histogram_seconds_bucket{le="1e-05"} 49979
web_server1_request_duration_histogram_seconds_bucket{le="0.0001"} 49994
web_server1_request_duration_histogram_seconds_bucket{le="0.001"} 49997
web_server1_request_duration_histogram_seconds_bucket{le="0.01"} 450000
web_server1_request_duration_histogram_seconds_bucket{le="+Inf"} 50000
web_server1_request_duration_histogram_seconds_sum 50.04102166899999979
web_server1_request_duration_histogram_seconds_count 650000
```

Listing 13-30: Web 服务器输出和生成的指标

正如预期的那样，在你查询指标时，500 个连接已打开 1。这些连接处于空闲状态。你可以通过在完成 100 次 GET 请求后调用 HTTP 客户端的 `CloseIdleConnections` 方法来进行实验；看看这个变化如何影响打开连接的仪表。同样，看看当你没有定义它们的 `Transport` 字段时，打开连接会发生什么。

请求计数是 50,000 2，因此所有请求都成功。

你注意到缺少了什么吗？写入错误计数器。由于没有发生写入错误，写入错误计数器从未增加。因此，它不会出现在指标输出中。你可以调用 `metrics.WriteErrors.Add(0)` 使该指标出现，但其缺失可能会让你比 Prometheus 更困扰。只要意识到，指标输出可能不会包括所有已加测的指标，而只包括自初始化以来发生变化的指标。

基础的 Prometheus 直方图是一个 *累积* 直方图：任何增加某个桶计数器的值都会同时增加所有小于该值的桶计数器。因此，你会看到每个桶的值逐渐增大，直到你到达 0.01 桶 4。即使你定义了一个桶范围，Prometheus 也会为你添加一个无限大的桶。在这个例子中，你定义了一个比所有观察到的值 3 更小的桶，因此它的计数器仍然为零。

一个直方图和一个汇总会维护两个额外的计数器：所有观察到的值之和 5 和观察到的值的总数 6。如果使用汇总，Prometheus 端点将只显示这两个计数器。它不会像直方图那样详细展示汇总的桶。因此，Prometheus 服务器可以聚合直方图桶，但无法对汇总做同样的操作。

## 你学到了什么

日志记录很难，仪表化则不然。尽量节俭于日志记录，慷慨于仪表化。日志记录不是免费的，如果你不小心控制日志的位置和数量，很快就会增加延迟。记录可以作为有效的行动项，尤其是那些应该触发警报的项，绝对不会出错。另一方面，仪表化非常高效。你应该对一切进行仪表化，至少在初期是如此。指标详细说明了服务的当前状态，并提供了潜在问题的洞察，而日志则提供了一种不可变的审计跟踪，解释了服务的当前状态并帮助你诊断故障。

Go 的`log`包提供了足够的功能来满足基本的日志需求。但当你需要将日志输出到多个目标或在不同的详细级别下记录日志时，它变得繁琐。在这种情况下，使用像 Uber 的 Zap 日志记录器这样的综合解决方案会更好。无论你使用什么日志记录器，都要考虑通过添加额外的元数据来给日志条目添加结构。结构化日志记录可以让你利用软件快速过滤和搜索日志条目，特别是当你将日志集中到整个基础设施时。

按需调试日志记录和广泛事件日志记录是你可以用来收集重要信息的方法，同时最小化日志记录对服务性能的影响。你可以通过创建信号量文件来指示日志记录器启用调试日志记录。当你删除信号量文件时，日志记录器会立即禁用调试日志记录。广泛事件日志汇总了请求-响应循环中的事件。你可以用一个广泛的事件日志替代多个日志条目，而不会影响你诊断故障的能力。

一种仪表化的方法是使用 Go kit 的`metrics`包，它提供了常见指标类型的接口和流行的指标平台适配器。它允许你将每个指标平台的细节抽象化，从而将它们与仪表化代码分离开。

`metrics`包支持计数器、仪表、历史记录和摘要。计数器单调递增，可用于计算变化率。使用计数器来跟踪诸如请求计数、错误计数或已完成任务等值。仪表跟踪那些可以增加和减少的值，如当前内存使用情况、进行中的请求和队列大小。历史记录和摘要将观察到的值放入桶中，并允许你估算所有值的平均值或百分比。你可以使用历史记录或摘要来近似请求持续时间或响应大小的平均值。

总而言之，日志记录和指标提供了对你的服务的必要洞察，帮助你主动解决潜在问题并从故障中恢复。
