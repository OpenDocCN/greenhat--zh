# 第十二章：数据序列化

![](img/chapterart.png)

作为开发者，我们的工作很大一部分是将我们的网络服务与现有服务进行集成，包括用其他编程语言实现的遗留服务或第三方服务。这些服务必须通过交换数据字节进行通信，这种通信方式对发送方和接收方都能理解，尽管他们使用的是不同的编程语言。为此，发送方使用标准格式将数据转换为字节，并通过网络将字节传输给接收方。如果接收方理解发送方使用的格式，它就能将字节转换回结构化数据。这个将结构化数据转化为连续字节的过程称为*数据序列化*。

服务可以使用数据序列化将结构化数据转换为适合在网络上传输或持久化到存储的字节序列。无论序列化的数据来自网络还是磁盘，任何理解该序列化格式的代码都应该能够反序列化这些数据，恢复出原始对象的副本。

在写这章时，我一开始很难解释数据序列化的概念。后来我意识到，我们在说话时其实就是在序列化数据。大脑中的电信号形成单词，然后大脑指示声带将这些单词*序列化*为声波，这些声波穿过空气到达你的耳朵。声波使你的耳膜振动，进而将振动传递到内耳。内耳中的类毛结构将这些振动反序列化为电信号，这些电信号被你的大脑解释为我大脑中形成的原始单词。我们刚刚使用英语这种序列化格式进行了交流，因为这是我们都能理解的格式。

你在 Go 代码中也已经有了一些序列化数据的经验。你在第四章学到的类型-长度-值二进制编码，以及在第八章通过 HTTP 发送的 JavaScript 对象表示法（JSON），都是将对象转化为众所周知的数据序列化格式的例子。在第十一章中，我们还对证书和私钥进行了 PEM 编码，以便将其持久化到磁盘中。

本章将深入探讨如何使用数据序列化来存储数据或在系统之间传输数据，这使得数据可以被用其他语言编写的服务访问。我们可以讨论许多数据序列化格式，但我们将重点讨论 Go 网络编程中最常用的三种格式：JSON、协议缓冲区（protocol buffers）和 Gob。我们还将花一些时间介绍如何使用一个名为 gRPC 的框架在远程机器上执行代码。到本章结束时，你将知道如何序列化数据以便存储或传输，并能够将这些数据解码成有意义的数据结构。你应该能够使用本章中的技术构建可以通过网络交换复杂数据的服务，或者编写代码与现有的网络服务进行通信。

## 序列化对象

对象或结构化数据不能直接通过网络连接传输。换句话说，不能将对象传递给`net.Conn`的`Write`方法，因为它只接受字节切片。因此，你需要将对象序列化为字节切片，然后才能传递给`Write`方法。幸运的是，Go 让这个过程变得很简单。

Go 的标准库在其`encoding`包中对流行的数据序列化格式提供了出色的支持。你已经使用了`encoding/binary`将数字序列化为字节序列，使用`encoding/json`将对象序列化为 JSON 以便通过 HTTP 提交，使用`encoding/pem`将 TLS 证书和私钥序列化为文件。（每当你遇到一个函数或方法，其名称中包含*encode*或*marshal*时，它可能会进行数据序列化。同样，*decode*和*unmarshal*与反序列化数据是同义词。）

本节将构建一个应用程序，将数据序列化为三种二进制编码格式：JSON、协议缓冲区和 Gob。由于我常常难以记住家务，所以这个应用程序将记录需要做的家务。应用程序的状态将在执行之间持久化，因为你不希望它在退出时忘记家务。你将把每个任务序列化到文件中，并使用你的应用程序根据需要更新它。

为了保持程序简洁，你需要一个家务的描述，并且有一种方法来确定它是否已完成。列表 12-1 定义了一个新的包，其中包含一个表示家庭家务的类型。

```
package housework

type Chore struct {
    Complete    bool
    Description string
}
```

列表 12-1：表示家庭家务的类型（*housework/housework.go*）

Go 的 JSON 和 Gob 编码包只能序列化导出的结构体字段，因此你将`Chore`定义为一个结构体，并确保其字段是导出的。如果你已经完成了家务，`Complete`字段的值将为`true`。`Description`字段是家务的可读描述。

如果需要，你可以使用结构体标签来指示编码器如何处理每个字段。例如，你可以在`Complete`字段上添加结构体标签`` `json:"-"` ``，告诉 Go 的 JSON 编码器忽略该字段，而不是对其进行编码。由于你完全愿意传递所有字段的值，因此可以省略结构体标签。

一旦你定义了家务结构体，你可以在一个应用程序中使用它，跟踪命令行上的家务。这个应用程序应该显示家务的列表及其状态，允许你将家务添加到列表中，并标记家务为已完成。列表 12-2 包括该家务应用程序的初始代码，其中包括其命令行使用细节。

```
package main

import (
    "flag"
    "fmt"
    "log"
    "os"
    "path/filepath"
    "strconv"
    "strings"

    "github.com/awoodbeck/gnp/ch12/housework"
 1 storage "github.com/awoodbeck/gnp/ch12/json"
    // storage "github.com/awoodbeck/gnp/ch12/gob"
    // storage "github.com/awoodbeck/gnp/ch12/protobuf"
)

var dataFile string

func init() {
    flag.StringVar(&dataFile, "file", "housework.db", "data file")

    flag.Usage = func() {
        fmt.Fprintf(flag.CommandLine.Output(),
         2 `Usage: %s [flags] [add chore, ...|complete #]
    add         add comma-separated chores
    complete    complete designated chore

Flags:
`, filepath.Base(os.Args[0]))
        flag.PrintDefaults()
    }
}
```

列表 12-2：初始家务应用代码（*cmd/housework.go*）

这段代码设置了命令行参数及其用法 2：你可以指定`add`参数，后跟以逗号分隔的任务列表来添加任务，或者你可以传递`complete`参数和任务编号来标记任务为完成。如果没有命令行选项，应用程序将显示当前的任务列表。

由于此应用程序的最终目的是演示数据序列化，你将使用多种序列化格式来存储数据。这将向你展示如何轻松地在不同格式之间切换。为此，你包括了这些格式的`import`语句 1。这将使你以后更容易在这些格式之间切换。

让我们编写代码从存储中加载任务（参见列表 12-3）。

```
`--snip--`

func load() ([]*housework.Chore, error) {
    if _, err := os.Stat(dataFile); 1os.IsNotExist(err) {
        return make([]*housework.Chore, 0), nil
    }

    df, err := 2os.Open(dataFile)
    if err != nil {
        return nil, err
    }
    defer func() {
        if err := df.Close(); err != nil {
            fmt.Printf("closing data file: %v", err)
        }
    }()

    return 3storage.Load(df)
}
```

列表 12-3：从文件中反序列化任务 (*cmd/housework.go*)

该函数返回一个指向`housework.Chore`结构体的指针切片，来自列表 12-1。如果数据文件不存在 1，你将提前退出并返回一个空切片。这个默认情况会在你第一次运行应用程序时发生。

如果应用程序找到数据文件，你将打开它 2，并将其传递给存储的`Load`函数 3，后者期望一个`io.Reader`。你在前几章中使用了相同的接受接口并返回具体类型的模式。

列表 12-4 定义了一个函数，用于将内存中的任务刷新到存储中以便持久化。

```
`--snip--`

func flush(chores []*housework.Chore) error {
    df, err := 1os.Create(dataFile)
    if err != nil {
        return err
    }
    defer func() {
        if err := df.Close(); err != nil {
            fmt.Printf("closing data file: %v", err)
        }
    }()

    return 2storage.Flush(df, chores)
}
```

列表 12-4：将任务刷新到存储中 (*cmd/housework.go*)

在这里，你创建一个新文件或截断现有文件 1，并将文件指针和任务切片传递给存储的`Flush`函数 2。该函数接受一个`io.Writer`和你的切片。你处理现有序列化文件的方式当然还有改进的空间。但为了演示的目的，这样已经足够。

你需要一种在命令行上显示任务的方式。列表 12-5 向你的应用程序添加了这样一个函数。

```
`--snip--`

func list() error {
    chores, err := 1load()
    if err != nil {
        return err
    }

    if len(chores) == 0 {
        fmt.Println("You're all caught up!")
        return nil
    }

    fmt.Println("#\t[X]\tDescription")
    for i, chore := range chores {
        c := " "
 if chore.Complete {
            c = "X"
        }
        fmt.Printf("%d\t[%s]\t%s\n", i+1, c, chore.Description)
    }

    return nil
}
```

列表 12-5：将任务列表打印到标准输出 (*cmd/housework.go*)

首先，你从存储中加载任务列表 1。如果你的列表中没有任务，你只需将其打印到标准输出。否则，你将打印一个标题和任务列表，类似于这样（参见列表 12-6）。

```
#       [X]     Description
1       [ ]     Mop floors
2       [ ]     Clean dishes
3       [ ]     Mow the lawn
```

列表 12-6：任务列表函数的示例输出，列表中有三个任务

第一列表示任务编号。你可以参考此编号来标记任务为完成，这将在第二列的方括号之间添加一个 X。第三列描述了该任务。

列表 12-7 实现了`add`函数，允许你向列表中添加任务。

```
`--snip--`

func add(s string) error {
    chores, err := 1load()
    if err != nil {
        return err
    }

    for _, chore := range 2strings.Split(s, ",") {
        if desc := strings.TrimSpace(chore); desc != "" {
            chores = append(chores, &housework.Chore{
                Description: desc,
            })
        }
    }

    return 3flush(chores)
}
```

列表 12-7：向任务列表中添加任务 (*cmd/housework.go*)

与长时间运行的服务不同，这个应用程序的生命周期从你在命令行执行它时开始，到你要求它完成任务时结束。因此，因为你希望家务任务列表在应用程序执行之间得以持久化，你需要将家务任务的状态存储在磁盘上。换句话说，你从存储中检索家务任务，修改它们，然后将更改刷新到存储中。更改将持续存在，直到下次运行应用程序时。

你希望能够一次添加多个家务任务，因此你通过逗号将传入的家务描述进行分割，并将每个家务任务添加到切片中。当然，这样做会使你不能在单个家务描述中使用逗号，因此家庭成员必须将他们的请求简短些（我个人认为这也不全是坏事）。作为练习，想想如何解决这个限制。一个方法是使用不同的分隔符，但请记住，选择的分隔符在命令行中可能有特殊意义。另一种方法是支持包含逗号的带引号字符串。

这块拼图的最后一部分是我最喜欢的关于处理家务任务的部分：将它们标记为完成（见列表 12-8）。

```
`--snip--`

func complete(s string) error {
    i, err := strconv.Atoi(s)
    if err != nil {
        return err
    }

    chores, err := load()
    if err != nil {
        return err
    }

    if i < 1 || i > len(chores) {
        return fmt.Errorf("chore %d not found", i)
    }

 1 chores[i-1].Complete = true

    return flush(chores)
}
```

列表 12-8：标记家务任务为完成（*cmd/housework.go*）

`complete`函数接受代表你想完成的家务任务的命令行参数，并将其转换为整数。我发现如果我一个个任务地完成，效率会更高，所以我让你一次只标记一个任务完成。然后，你从存储中加载家务任务，确保整数在有效范围内。如果是，你就将该家务任务标记为完成。由于你在显示列表时是从 1 开始编号家务任务，所以需要通过减去 1 来调整切片中的位置。最后，你将家务任务刷新到存储中。

现在，让我们通过实现应用程序的`main`函数来将一切连接在一起，参见列表 12-9。

```
`--snip--`

func main() {
    flag.Parse()

    var err error

    switch strings.ToLower(flag.Arg(0)) {
    case "add":
        err = add(strings.Join(flag.Args()[1:], " "))
    case "complete":
        err = complete(flag.Arg(1))
    }

    if err != nil {
        log.Fatal(err)
    }

    err = list()
    if err != nil {
        log.Fatal(err)
    }
}
```

列表 12-9：家务应用程序的主要逻辑（*cmd/housework.go*）

你已经尽可能地将逻辑放入之前的函数中，因此这个`main`函数相当简洁。你检查第一个参数，以确定它是否是预期的子命令。如果是，你调用相应的函数。如果在考虑了可选子命令及其参数后，`err`依然为 nil，则调用`list`函数。

现在剩下的就是为 JSON、Gob 和协议缓冲区实现存储的`Load`和`Flush`函数。

### JSON

JSON 是一种常见的、人类可读的基于文本的数据序列化格式，使用键值对和数组表示复杂的数据结构。大多数现代编程语言都提供对 JSON 的官方库支持，这也是它成为 RESTful API 常用编码格式的原因之一。

JSON 的类型包括字符串、布尔值、数字、数组、键值对对象和由关键字 *null* 指定的 nil 值。JSON 数字不区分浮点数和整数。你可以在 [`blog.golang.org/json`](https://blog.golang.org/json) 阅读更多关于 Go 中 JSON 实现的内容。

让我们看看如果我们将示例 12-6 中的家务事序列化为 JSON，*housework.db* 文件的内容会是什么样子。我已经在示例 12-10 中将 JSON 格式化为便于阅读的方式，尽管你也可以使用编码器的 `SetIndent` 方法来完成此操作。

```
1[
2 {
    "Complete": false,
    "Description": "Mop floors"
  },
  {
    "Complete": false,
    "Description": "Clean dishes"
  },
  {
    "Complete": false,
    "Description": "Mow the lawn"
  }
]
```

示例 12-10：将家务事序列化为 JSON 后的 *housework.db* 文件格式化内容

如你所见，JSON 是一个包含对象的数组 1，每个对象 2 包含 `Complete` 和 `Description` 字段及其相应的值。

示例 12-11 详细介绍了使用 Go 的 `encoding/json` 包实现的 JSON 存储。

```
package json

import (
    "encoding/json"
    "io"

    "github.com/awoodbeck/gnp/ch12/housework"
)

func Load(r io.Reader) ([]*housework.Chore, error) {
    var chores []*housework.Chore

    return chores, 1json.NewDecoder(r).Decode(&chores)
}

func Flush(w io.Writer, chores []*housework.Chore) error {
    return 2json.NewEncoder(w).Encode(chores)
}
```

示例 12-11：JSON 存储实现 (*json/housework.go*)

`Load` 函数将 `io.Reader` 传递给 `json.NewDecoder` 函数 1，并返回一个解码器。然后，你调用解码器的 `Decode` 方法，并传入指向 `chores` 切片的指针。解码器从 `io.Reader` 读取 JSON，进行反序列化，并填充 `chores` 切片。

`Flush` 函数接受一个 `io.Writer` 和一个 `chores` 切片。然后它将 `io.Writer` 传递给 `json.NewEncoder` 函数 2，后者返回一个编码器。你将 `chores` 切片传递给编码器的 `Encode` 方法，该方法将 `chores` 切片序列化为 JSON 并写入 `io.Writer`。

现在你已经实现了一个可以作为应用存储的 JSON 包，我们来在示例 12-12 中试试它。

```
$ **go run cmd/housework.go**
You're all caught up!
$ **go run cmd/housework.go add Mop floors, Clean dishes, Mow the lawn**
#       [X]     Description
1       [ ]     Mop floors
2       [ ]     Clean dishes
3       [ ]     Mow the lawn
$ **go run cmd/housework.go complete 2**
#       [X]     Description
1       [ ]     Mop floors
2       [X]     Clean dishes
3       [ ]     Mow the lawn
$ **cat housework.db**
[{"Complete":false,"Description":"Mop floors"},
{"Complete":true,"Description":"Clean dishes"},
{"Complete":false,"Description":"Mow the lawn"}]
```

示例 12-12：在命令行上测试带有 JSON 存储的家务应用

你第一次运行应用时，会发现你的家务列表是空的。然后，你添加了三个逗号分隔的家务事，并完成了第二个。看起来不错。还要注意，*housework.db* 文件包含可读的 JSON（在 Windows 上查看时，使用 `type` 命令而不是 `cat`）。让我们修改这个应用，改用 Go 原生的二进制编码格式。

### Gob

*Gob*（“大量二进制数据”的缩写）是 Go 的原生二进制序列化格式。Go 团队的工程师们开发了 Gob，旨在将协议缓冲（可能是最流行的二进制序列化格式）的高效性与 JSON 的易用性结合起来。例如，协议缓冲不允许我们像在示例 12-11 中的 JSON 示例那样，简单地实例化一个新的编码器并将数据结构传递给它。另一方面，Gob 的工作方式与 JSON 编码器非常相似，它可以智能地推断对象的结构并进行序列化。

如果你有兴趣探索 Gob 的动机和细节，建议阅读 Rob Pike 的《Gobs of Data》博客文章（[`blog.golang.org/gob`](https://blog.golang.org/gob)）。与此同时，我们来实现一个基于 Gob 的存储后端（参见 Listing 12-13）。

```
package gob

import (
    "encoding/gob"
    "io"

 "github.com/awoodbeck/gnp/ch12/housework"
)

func Load(r io.Reader) ([]*housework.Chore, error) {
    var chores []*housework.Chore

    return chores, gob.NewDecoder(r).Decode(&chores)
}

func Flush(w io.Writer, chores []*housework.Chore) error {
    return gob.NewEncoder(w).Encode(chores)
}
```

Listing 12-13：Gob 存储实现 (*gob/housework.go*)

如果你查看这段代码，并注意到它将 Listing 12-11 中所有出现的 `json` 替换为 `gob`，你并没有错。在 Go 中，使用 Gob 就像使用 JSON 一样简单，因为它会根据对象本身推断出需要编码的内容。你将在下一节看到，这与协议缓冲区（protocol buffers）有何不同。

剩下的工作就是通过修改 Listing 12-2 中的导入来将 JSON 存储实现替换为 Gob 实现（参见 Listing 12-14）。

```
`--snip--`
    "github.com/awoodbeck/gnp/ch12/housework"
 1 // storage "github.com/awoodbeck/gnp/ch12/json"
 2 storage "github.com/awoodbeck/gnp/ch12/gob"
    // storage "github.com/awoodbeck/gnp/ch12/protobuf"
`--snip--`
```

Listing 12-14：将 JSON 存储包替换为 Gob 存储包 (*cmd/housework.go*)

注释掉 Listing 12-2 中的 JSON 存储包导入 1，并取消注释 Gob 存储包导入 2。

由于你当前的 *housework.db* 文件包含 JSON 格式，因此与 Gob 不兼容。因此，尝试使用 Gob 解码时，housework 应用程序将抛出错误。删除 *housework.db* 文件并重新测试应用程序（参见 Listing 12-15）。

```
$ **rm housework.db**
$ **go run cmd/housework.go**
You're all caught up!
$ **go run cmd/housework.go add Mop floors, Clean dishes, Mow the lawn**
#       [X]     Description
1       [ ]     Mop floors
2       [ ]     Clean dishes
3       [ ]     Mow the lawn
$ **go run cmd/housework.go complete 2**
#       [X]     Description
1       [ ]     Mop floors
2       [X]     Clean dishes
3       [ ]     Mow the lawn
$ **hexdump -c housework.db**
0000000  \r 377 203 002 001 002 377 204  \0 001 377 202  \0  \0   ) 377
0000010 201 003 001 002 377 202  \0 001 002 001  \b   C   o   m   p   l
0000020   e   t   e 001 002  \0 001  \v   D   e   s   c   r   i   p   t
0000030   i   o   n 001  \f  \0  \0  \0   1 377 204  \0 003 002  \n   M
0000040   o   p       f   l   o   o   r   s  \0 001 001 001  \f   C   l
0000050   e   a   n       d   i   s   h   e   s  \0 002  \f   M   o   w
0000060       t   h   e       l   a   w   n  \0                        
000006a
```

Listing 12-15：在命令行上测试使用 Gob 存储的 housework 应用程序

一切仍按预期工作。使用 `hexdump` 工具，你可以看到 *housework.db* 文件现在包含了二进制数据。虽然它不像 Listing 12-12 中的 JSON 那样可读，但 Go 会愉快地反序列化 Gob 编码的数据，尽管我们很难理解它。（我的 Windows 朋友可以在 *https://www.di-mgt.com.au/hexdump-for-windows.html* 找到 `hexdump` 的二进制文件，尽管你需要使用 `-C` 标志才能看到相同的效果。）

如果你与其他支持 Gob 的 Go 服务进行通信，我建议你使用 Gob 而不是 JSON。Go 的 `encoding/gob` 比 `encoding/json` 更高效。Gob 编码也更简洁，因为 Gob 使用比 JSON 更少的数据来表示对象。当存储或传输序列化数据时，这一点可能会带来差异。

现在你已经体验了使用 `encoding/json` 和 `encoding/gob` 来序列化数据，我们来为你的存储后端添加协议缓冲区支持。

### 协议缓冲区（Protocol Buffers）

像 Gob 一样，*协议缓冲区*使用二进制编码来存储或交换跨平台的信息。它比 Go 的 JSON 编码更快、更简洁。与 Gob 不同，像 JSON 一样，协议缓冲区是语言中立的，并且在流行的编程语言中得到了广泛支持。这使得它们非常适合在基于 Go 的服务中使用，尤其是当你希望与其他编程语言编写的服务集成时。本章假设你使用的是 *proto3* 版本的格式。

协议缓冲区使用定义文件，通常以*.proto*为后缀，来定义消息。*消息*描述了你想要序列化以便存储或传输的结构化数据。例如，表示`Chore`类型的协议缓冲区消息如下所示，在 Listing 12-16 中有定义。

```
message Chore {
  bool complete = 1;
  string description = 2;
}
```

Listing 12-16：表示`Chore`类型的协议缓冲区消息定义

你通过使用`message`关键字定义一个新的消息，后跟消息的唯一名称。惯例是使用 Pascal 大小写（*Pascal casing*），它是一种代码格式化风格，其中将单词首字母大写并连接在一起：*ThisIsPascalCasing*。然后，你可以为`Chore`消息添加字段。每个字段定义包括类型、一个蛇形命名（*snake casing*）的名称和一个该消息唯一的字段编号。蛇形命名就像 Pascal 大小写，只不过第一个单词是小写的：*thisIsSnakeCasing*。字段的类型和编号标识了二进制负载中的字段，因此一旦使用，这些字段不得更改，否则会破坏向后兼容性。然而，可以向现有的*.proto*文件中添加新的消息和消息字段。

说到向后兼容性，最佳实践是将协议缓冲区定义当作 API 来处理。如果第三方使用你的协议缓冲区定义与服务进行通信，考虑为你的定义进行版本控制；这样，你就可以在需要打破向后兼容性时创建新版本。你的开发可以继续进行最新版本，而客户端则可以继续使用之前的版本，直到他们准备好升级到最新版本。你将在本节稍后看到一种协议缓冲区定义版本控制的方法。

你需要编译*.proto*文件以生成 Go 代码。该代码允许你序列化和反序列化在*.proto*文件中定义的消息。希望与你交换消息的第三方可以使用相同的*.proto*文件为其目标编程语言生成代码。生成的代码也可以与你交换消息。因此，在你开始认真使用协议缓冲区之前，必须安装协议缓冲区编译器及其 Go 生成模块。你的操作系统的包管理器可能允许你轻松安装协议缓冲区编译器。例如，在 Debian 10 上，运行以下命令：

```
$ **sudo apt install protobuf-compiler**
```

在 macOS 上使用 Homebrew，运行以下命令：

```
$ **brew install protobuf**
```

在 Windows 上，从[`github.com/protocolbuffers/protobuf/releases/`](https://github.com/protocolbuffers/protobuf/releases/)下载最新的协议缓冲区编译器 ZIP 文件，解压后将其*bin*子目录添加到你的`PATH`中。现在你应该能够在命令行上运行*protoc*二进制文件了。

一个简单的`go get`命令将会在你的系统上安装协议缓冲区的 Go 生成器。确保你将生成的*protoc-gen-go*二进制文件添加到`PATH`中，否则*protoc*无法识别该插件：

```
$ **GO111MODULE=on go get -u github.com/golang/protobuf/protoc-gen-go**
```

现在你已经安装了协议缓冲区编译器和 Go 生成模块，让我们为你的家务应用创建一个新的*.proto*文件（见清单 12-17）。你将在*housework/v1/housework.proto*中创建这个文件。路径中的`v1`表示*版本 1*，并允许你在将来引入该包的不同版本。

```
1 syntax = "proto3";
2 package housework;

3 option go_package = "github.com/awoodbeck/gnp/ch12/housework/v1/housework";

message Chore {
  bool complete = 1;
  string description = 2;
}

message Chores {
4repeated Chore chores = 1;
}
```

清单 12-17：你家务应用的协议缓冲区定义（*housework/v1/housework.proto*）

首先，你指定使用 proto3 语法 1，并希望生成的任何代码都使用`housework`作为包名 2。接着，你添加一个`go_package`选项 3，指定生成模块的完整导入路径。然后你定义`Chore`消息，以及第二个消息`Chores`，它表示一个基于`repeated`字段类型 4 的`Chore`消息集合。

现在，让我们将*.proto*文件编译成 Go 代码：

```
$ **protoc** 1**--go_out=.** 2**--go_opt=paths=source_relative housework/v1/housework.proto**
```

你使用带有标志的*protoc*命令，表明你希望从清单 12-17 中创建的*housework/v1/housework.proto*文件生成 Go 代码，并将生成的代码输出到*.proto*文件的当前目录，使用相对路径 2。

如果你收到以下错误，提示*protoc*找不到*protoc-gen-go*二进制文件，请确保*protoc-gen-go*的路径（可能是*$GOPATH/bin*）已加入到`PATH`环境变量中：

```
protoc-gen-go: program not found or is not executable
--go_out: protoc-gen-go: Plugin failed with status code 1.
```

如果*protoc*对*.proto*文件没有问题，并成功生成了 Go 代码，你会发现一个名为*housework/v1/housework.pb.go*的新文件，虽然版本号可能有所不同。我将在 Linux/macOS 上使用`head`命令打印出前七行：

```
$ **head -n 7 housework/v1/housework.pb.go**
// Code generated by protoc-gen-go. DO NOT EDIT.
// versions:
//      protoc-gen-go v1.25.0
//      protoc        v3.6.1
// source: housework/v1/housework.proto

package housework
```

正如注释所示，你不应编辑此模块。相反，应对*.proto*文件进行必要的修改并重新编译。

现在，你已经从*.proto*文件生成了一个 Go 模块，可以通过在清单 12-18 中实现协议缓冲区存储后端来有效利用它。

```
package protobuf

import (
    "io"
    "io/ioutil"

    "google.golang.org/protobuf/proto"

 1 "github.com/awoodbeck/gnp/ch12/housework/v1"
)

func Load(r io.Reader) ([]*housework.Chore, error) {
    b, err := ioutil.ReadAll(r)
    if err != nil {
        return nil, err
    }

    var chores housework.Chores

    return chores.Chores, proto.Unmarshal(b, &chores)
}

func Flush(w io.Writer, chores []*housework.Chore) error {
    b, err := proto.Marshal(2&housework.Chores{Chores: chores})
    if err != nil {
        return err
    }

    _, err = w.Write(b)

    return err
}
```

清单 12-18：协议缓冲区存储实现（*protobuf/housework.go*）

你不再像使用 JSON 和 Gob 时那样依赖清单 12-1 中的`housework`包，而是导入*protoc*生成的版本 1 包，你也将其命名为`housework`。生成的`Chores`类型是一个结构体，包含一个`Chores`字段，`Chores`本身是`Chore`指针的切片。此外，Go 的协议缓冲区包并未实现编码器和解码器。因此，你需要手动将对象序列化为字节，写入到`io.Writer`中，并从`io.Reader`中反序列化字节。

回顾清单 12-2 中的代码，并插入协议缓冲区实现，只需做出清单 12-19 中显示的两个简单更改。

```
`--snip--`
 1 "github.com/awoodbeck/gnp/ch12/housework/v1"
    // storage "github.com/awoodbeck/gnp/ch12/json"
 // storage "github.com/awoodbeck/gnp/ch12/gob"
 2 storage "github.com/awoodbeck/gnp/ch12/protobuf"
`--snip--`
```

清单 12-19：将 JSON 存储包替换为`protobuf`存储包（*cmd/housework.go*）

你将从清单 12-1 中替换掉家务包，改为你生成的包 1，确保注释掉 `json` 和 `gob` 导入，取消注释 `protobuf` 存储导入 2。家务应用的实际功能保持不变。

## 传输序列化对象

虽然有时你可能需要序列化并本地存储对象，但你更有可能构建一个序列化数据的网络服务。例如，一个在线商店可能会有一个 web 服务，和库存、用户账户、账单、运输和通知服务进行通信，以便促进客户订单。如果这些服务都运行在同一台服务器上，你就需要购买更大的服务器，以便随着业务增长来扩展在线商店。另一种方法是将每个服务都运行在独立的服务器上，并增加服务器的数量。但这样你会面临一个新问题：当这些服务不再位于同一台服务器上，无法访问相同的内存时，你该如何在它们之间共享数据？

大型科技公司通过 *远程过程调用 (RPC)* 来简化这个过程，RPC 是一种技术，客户端可以透明地调用服务器上的子程序，就像这个调用是本地的一样。从应用程序的角度来看，RPC 服务将看似本地运行的代码分发到网络上。你的代码可能会调用一个函数，该函数透明地将消息转发到服务器。服务器会在本地执行该函数，然后返回结果，代码收到这些结果作为函数的返回值。就你的代码而言，函数调用是本地的，尽管 RPC 与服务器的交互是透明的。这种方法允许你在服务器之间扩展服务，同时将细节抽象化，不需要在代码中处理。换句话说，无论函数调用是在同一台计算机上执行，还是在网络上的另一台计算机上执行，你的代码的表现都一样。

现在大多数公司都使用 *gRPC* 来实现 RPC，这是一个跨平台框架，利用了 HTTP/2 和协议缓冲区。我们在这里使用它来构建比一个记录你尚未做的家务的应用程序更复杂的东西。你将编写一个服务，能够将任务发送给 Rosie，这个来自经典动画系列 *杰森一家* 的机器人女佣，她能接管你的家务责任。当然，她要到 2062 年才能问世，但你可以提前开始编写代码。

### 使用 gRPC 连接服务

gRPC 框架是一个库集合，抽象了许多 RPC 实现的细节。它是平台中立的，并且对编程语言无关；例如，你可以使用它将运行在 Windows 上的 Go 服务与运行在 Linux 上的 Rust 服务集成。现在你已经知道如何使用协议缓冲区，你有了向你的应用程序添加 gRPC 支持所需的基础。你将复用上一节中的 *.proto* 文件。

首先，确保你的 gRPC 包是最新的：

```
$ **go get -u google.golang.org/grpc**
```

接下来，获取生成 gRPC Go 代码所需的模块：

```
$ **go get -u google.golang.org/grpc/cmd/protoc-gen-go-grpc**
```

协议缓冲区编译器包含一个 gRPC 模块。此模块将输出 Go 代码，使你可以轻松添加 gRPC 支持。首先，你需要向*.proto*文件中添加定义。Listing 12-20 定义了一个新服务和两个额外的消息。

```
`--snip--`

service RobotMaid {
1   rpc Add (Chores) returns (Response);
  rpc Complete (CompleteRequest) returns (Response);
  rpc List (Empty) returns (Chores);
}

message CompleteRequest {
  int32 2chore_number = 1;
}

3 message Empty {}

message Response {
  string message = 1;
}
```

Listing 12-20：支持 gRPC RobotMaid 服务的附加定义（*housework/v1/housework.proto*）

该服务需要支持你在命令行中使用的相同三种调用：`add`、`complete`和`list`。你定义了一个名为`RobotMaid`的新服务，然后向其添加了三个 RPC 方法。这些 RPC 方法分别对应于 Listing 12-5、12-7 和 12-8 中定义的函数，用于命令行：分别是`list`、`add`和`complete`函数。你将不再在本地调用这些函数，而是通过 RPC 调用`RobotMaid`上的相应方法来执行这些命令。你将每个方法前缀加上`rpc`关键字，并跟随 Pascal 命名法的函数名。接下来，你在括号中编写请求消息类型，使用`returns`关键字，并在括号中编写返回消息类型 1。

`List`方法不需要任何用户输入，但与命令行应用程序中的情况一样，你仍然必须为它提供一个请求消息类型，即使它是 nil。在 gRPC 中，等同于 nil 的消息类型是一个空消息，你称之为`Empty`3。

在你有机会为机器人添加适当的人工智能（AI）之前，你需要告诉 Rosie 当前的家务活已完成，以便她可以继续下一个任务。为此，你添加了一个新消息，通知她已完成的家务活编号 2。由于你期望从 Rosie 那里获得反馈，你还添加了一个响应消息，包含一个字符串。

现在编译*.proto*文件，以使用新的服务和消息。告诉*protoc*使用*protoc-gen-go-grpc*二进制文件，它也必须位于你的`PATH`环境变量中，如下所示：

```
$ **protoc** 1**--go-grpc_out=.** 2--go-grpc_opt=paths=source_relative \
**housework/v1/housework.proto**
```

`--go-grpc_out`标志 1 调用了*protoc-gen-go-grpc*二进制文件，为生成的代码添加 gRPC 支持。该二进制文件为你生成相关的 gRPC 服务代码，并将 gRPC 特定的代码写入*housework/v1/housework_grpc.pb.go*文件，因为你告诉*protoc-gen-go-grpc*使用相对路径 2。现在，你可以使用生成的代码来构建 gRPC 服务器和客户端。

### 创建一个启用 TLS 的 gRPC 服务器

现在，让我们实现一个 gRPC 客户端和服务器。默认情况下，gRPC 需要安全连接，因此你需要为服务器添加 TLS 支持。你将使用上一章中创建的服务器的*cert.pem*和*key.pem*文件作为你的 gRPC 服务器，并将服务器的证书固定到客户端。有关详细信息，请参阅第 256 页的“生成用于身份验证的证书”部分。

你将利用通过 *.proto* 文件生成的 Go 代码来定义一个新的 `RobotMaid` 客户端和服务器，并使用客户端通过网络与服务器进行 gRPC 通信。首先，让我们创建你机器人女仆的服务器部分。由你的 *.proto* 文件生成的 `RobotMaidServer` 接口如下所示：

```
type RobotMaidServer interface {
    Add(context.Context, *Chores) (*Response, error)
    Complete(context.Context, *CompleteRequest) (*Response, error)
    List(context.Context, *empty.Empty) (*Chores, error)
    mustEmbedUnimplementedRobotMaidServer()
}
```

你将在清单 12-21 中通过创建一个名为 `Rosie` 的新类型来实现这个接口。

```
package main

import (
    "context"
    "fmt"
    "sync"

    "github.com/awoodbeck/gnp/ch12/housework/v1"
)

type Rosie struct {
    mu sync.Mutex
 1 chores []*housework.Chore
}

func (r *Rosie) Add(_ context.Context, chores *housework.Chores) (
    *housework.Response, error) {
    r.mu.Lock()
    r.chores = append(r.chores, chores.Chores...)
    r.mu.Unlock()

    return 2&housework.Response{Message: "ok"}, nil
}

func (r *Rosie) Complete(_ context.Context,
    req *housework.CompleteRequest) (*housework.Response, error) {
    r.mu.Lock()
    defer r.mu.Unlock()

    if r.chores == nil || req.ChoreNumber < 1 ||
        int(req.ChoreNumber) > len(r.chores) {
        return nil, fmt.Errorf("chore %d not found", req.ChoreNumber)
    }

    r.chores[req.ChoreNumber-1].Complete = true

    return &housework.Response{Message: "ok"}, nil
}

func (r *Rosie) List(_ context.Context, _ *housework.Empty) (
    *housework.Chores, error) {
    r.mu.Lock()
    defer r.mu.Unlock()

    if r.chores == nil {
        r.chores = make([]*housework.Chore, 0)
    }

    return &housework.Chores{Chores: r.chores}, nil
}

func (r *Rosie) Service() *housework.RobotMaidService {
    return 3&housework.RobotMaidService{
        Add:      r.Add,
        Complete: r.Complete,
        List:     r.List,
    }
}
```

清单 12-21：构建一个与 `RobotMaid` 兼容的类型，命名为 `Rosie` (*server/rosie.go*)

新的 `Rosie` 结构体将它的家务列表保存在内存中 1，并由互斥锁保护，因为多个客户端可以同时使用该服务。`Add`、`Complete` 和 `List` 方法都返回一个响应消息类型 2 或错误，两者最终都会返回给客户端。`Service` 方法返回指向一个新的 `housework.RobotMaidService` 实例的指针 3，`Rosie` 的 `Add`、`Complete` 和 `List` 方法会映射到新实例的相应方法。

现在，让我们通过使用 `Rosie` 结构体来设置一个新的 gRPC 服务器实例 (清单 12-22)。

```
package main

import (
    "crypto/tls"
    "flag"
    "fmt"
    "log"
    "net"

    "google.golang.org/grpc"

    "github.com/awoodbeck/gnp/ch12/housework/v1"
)

var addr, certFn, keyFn string

func init() {
    flag.StringVar(&addr, "address", "localhost:34443", "listen address")
    flag.StringVar(&certFn, "cert", "cert.pem", "certificate file")
    flag.StringVar(&keyFn, "key", "key.pem", "private key file")
}

func main() {
    flag.Parse()

    server := 1grpc.NewServer()
    rosie := new(Rosie)
  2housework.RegisterRobotMaidServer(server, 3rosie.Service())

    cert, err := tls.LoadX509KeyPair(certFn, keyFn)
    if err != nil {
        log.Fatal(err)
    }

    listener, err := net.Listen("tcp", addr)
    if err != nil {
        log.Fatal(err)
    }

    fmt.Printf("Listening for TLS connections on %s ...", addr)
    log.Fatal(
     4 server.Serve(
         5 tls.NewListener(
                listener,
                &tls.Config{
                    Certificates:             []tls.Certificate{cert},
                    CurvePreferences:         []tls.CurveID{tls.CurveP256},
                    MinVersion:               tls.VersionTLS12,
 PreferServerCipherSuites: true,
                },
            ),
        ),
    )
}
```

清单 12-22：使用 `Rosie` 创建一个新的 gRPC 服务器 (*server/server.go*)

首先，你获取一个新的服务器实例 1。你将它和来自 `Rosie` 的 `Service`3 方法返回的新 `*housework.RobotMaidService` 传递给生成的 gRPC 代码中的 `RegisterRobotMaidServer` 函数 2。这将 `Rosie` 的 `RobotMaidService` 实现注册到 gRPC 服务器上。在调用服务器的 `Serve` 方法 4 之前，必须执行此操作。然后，你加载服务器的密钥对并创建一个新的 TLS 监听器 5，调用 `Serve` 时将其传递给服务器。

现在你已经有了一个 gRPC 服务器实现，让我们来处理客户端部分。

### 创建一个 gRPC 客户端来测试服务器

客户端代码与你在第 270 页“对象序列化”部分编写的代码差别不大。主要的区别是你需要实例化一个新的 gRPC 客户端，并修改 `add`、`complete` 和 `list` 函数来使用它。记住，如果编程语言支持 `protobuf`，你可以用它来实现客户端部分，并且可以为目标语言从 *.proto* 文件生成代码，期望它能与清单 12-22 中的服务器无缝协作。

清单 12-23 详细说明了清单 12-2 中对家务应用程序添加 gRPC 支持所需的更改。

```
package main

import (
 1 "context"
 2 "crypto/tls"
 3 "crypto/x509"
    "flag"
    "fmt"
 4 "io/ioutil"
    "log"
    "os"
    "path/filepath"
    "strconv"
    "strings"

    "google.golang.org/grpc"
    "google.golang.org/grpc/credentials"

    "github.com/awoodbeck/gnp/ch12/housework/v1"
)

var addr, caCertFn string

func init() {
 5 flag.StringVar(&addr, "address", "localhost:34443", "server address")
 6 flag.StringVar(&caCertFn, "ca-cert", "cert.pem", "CA certificate")

    flag.Usage = func() {
        fmt.Fprintf(flag.CommandLine.Output(),
            `Usage: %s [flags] [add chore, ...|complete #]
    add         add comma-separated chores
    complete    complete designated chore

Flags:
`, filepath.Base(os.Args[0]))
        flag.PrintDefaults()
    }
}
```

清单 12-23：我们的家务应用程序的初始 gRPC 客户端代码 (*client/client.go*)

除了所有新的导入 1234，你还添加了 gRPC 服务器地址 5 和证书 6 的标志。

清单 12-24 使用 gRPC 客户端列出当前的家务。

```
`--snip--`

func list(ctx context.Context, client housework.RobotMaidClient) error {
    chores, err := client.List(ctx, 1new(housework.Empty))
    if err != nil {
        return err
    }

    if len(chores.Chores) == 0 {
        fmt.Println("You have nothing to do!")
        return nil
    }

    fmt.Println("#\t[X]\tDescription")
    for i, chore := range chores.Chores {
        c := " "
        if chore.Complete {
            c = "X"
        }
        fmt.Printf("%d\t[%s]\t%s\n", i+1, c, chore.Description)
    }

    return nil
}
```

清单 12-24：使用 gRPC 客户端列出当前家务 (*client/client.go*)

这段代码与清单 12-5 非常相似，唯一不同的是你请求 gRPC 客户端获取任务列表，客户端从服务器中检索这些任务。你需要传递一个空的消息，以让 gRPC 正常工作 1。

清单 12-25 使用 gRPC 客户端将新任务添加到 gRPC 服务器的任务列表中。

```
`--snip--`

func add(ctx context.Context, client housework.RobotMaidClient,
    s string) error {
    chores := new(housework.Chores)

    for _, chore := range strings.Split(s, ",") {
        if desc := strings.TrimSpace(chore); desc != "" {
            chores.Chores = append(chores.Chores, &housework.Chore{
                Description: desc,
            })
        }
    }

    var err error
    if len(chores.Chores) > 0 {
        _, 1err = client.Add(ctx, chores)
    }

    return err
}
```

清单 12-25：使用 gRPC 客户端添加新任务（*client/client.go*）

就像在前一节中那样，你解析了逗号分隔的任务列表。不同的是，这些任务并没有被刷新到磁盘，而是被传递给了 gRPC 客户端。gRPC 客户端透明地将它们发送到 gRPC 服务器，并将响应返回给你。由于你知道当`Add`调用失败时`Rosie`会返回一个非 nil 的错误，因此你将错误 1 作为`add`函数的返回值。

在清单 12-26 中，你编写了标记任务完成的代码。通过 gRPC 执行这项操作比清单 12-8 的代码少一些，因为大部分逻辑都在服务器端。

```
`--snip--`

func complete(ctx context.Context, client housework.RobotMaidClient,
    s string) error {
    i, err := strconv.Atoi(s)
    if err == nil {
        _, err = client.Complete(ctx,
            &housework.CompleteRequest{1ChoreNumber: int32(i)})
    }

    return err
}
```

清单 12-26：使用 gRPC 客户端标记任务完成（*client/client.go*）

注意*protoc-gen-go*模块，它将清单 12-20 中的蛇形命名的`chore_number`字段转换为 Pascal 命名法，生成 Go 代码 1。你还必须在将其赋值给完整请求消息的任务编号之前，将`strconv.Atoi`返回的`int`转换为`int32`，因为`ChoreNumber`是`int32`类型。

清单 12-27 创建了一个新的 gRPC 连接，并将服务器的证书固定到其 TLS 配置中。

```
`--snip--`

func main() {
    flag.Parse()

    caCert, err := ioutil.ReadFile(caCertFn)
    if err != nil {
        log.Fatal(err)
    }
    certPool := x509.NewCertPool()
    if ok := certPool.AppendCertsFromPEM(caCert); !ok {
        log.Fatal("failed to add certificate to pool")
    }

    conn, err := 1grpc.Dial(
        addr,
     2 grpc.WithTransportCredentials(
         3 credentials.NewTLS(
                &tls.Config{
                    CurvePreferences: []tls.CurveID{tls.CurveP256},
                    MinVersion:       tls.VersionTLS12,
                    RootCAs:          certPool,
                },
            ),
        ),
    )
    if err != nil {
        log.Fatal(err)
    }
```

清单 12-27：使用 TLS 和证书固定创建一个新的 gRPC 连接（*client/client.go*）

在客户端，你首先创建一个新的 gRPC 网络连接 1，然后使用该网络连接实例化一个新的 gRPC 客户端。对于大多数用例，你可以简单地将地址传递给`grpc.Dial`。但你希望将服务器的证书固定到客户端连接中。因此，你需要明确传递一个`grpc.DialOption`，并附上适当的 TLS 凭证。这涉及使用`grpc.WithTransportCredentials`函数 2 来返回`grpc.DialOption`，并使用`credentials.NewTLS`函数 3 来根据你的 TLS 配置创建传输凭证。结果是一个 gRPC 网络连接，它与服务器进行 TLS 通信，并通过使用固定的证书来验证服务器。

你在清单 12-28 中使用这个 gRPC 网络连接实例化了一个新的 gRPC 客户端。

```
`--snip--`

    rosie := 1housework.NewRobotMaidClient(conn)
    ctx := context.Background()

    switch strings.ToLower(flag.Arg(0)) {
    case "add":
        err = add(ctx, rosie, strings.Join(flag.Args()[1:], " "))
    case "complete":
        err = complete(ctx, rosie, flag.Arg(1))
    }

    if err != nil {
        log.Fatal(err)
    }

    err = list(ctx, rosie)
    if err != nil {
        log.Fatal(err)
    }
}
```

清单 12-28：实例化一个新的 gRPC 客户端并进行调用（*client/client.go*）

除了从 gRPC 网络连接 1 实例化一个新的 gRPC 客户端外，这段代码与清单 12-9 的差异不大。区别当然在于，任何与任务列表的交互都透明地通过 TLS 连接与 gRPC 服务器进行。

尝试一下。在一个终端中启动服务器：

```
$ **go run server/server.go server/rosie.go -cert server/cert.pem -key server/key.pem**
Listening for TLS connections on localhost:34443 ...
```

在另一个终端中，运行客户端：

```
$ **go run client/client.go -ca-cert server/cert.pem**
You have nothing to do!
$ **go run client/client.go -ca-cert server/cert.pem add Mop floors, Wash dishes**
#       [X]     Description
1       [ ]     Mop floors
2       [ ]     Wash dishes
$ **go run client/client.go -ca-cert server/cert.pem complete 2**
#       [X]     Description
1       [ ]     Mop floors
2       [X]     Wash dishes
```

当然，重启服务器会清空任务列表。我将这作为练习留给你，去实现服务器上的持久化存储。一个方法是让服务器从磁盘加载任务并将任务刷新到磁盘，就像你在本章之前所做的那样。

## 你所学到的

数据序列化允许你以平台中立和语言中立的方式交换数据。你还可以序列化数据进行长期存储，检索并反序列化数据，然后继续从应用程序上次停止的地方开始。

JSON 可以说是最流行的基于文本的数据序列化格式。现代编程语言提供对 JSON 的良好支持，这也是它在 RESTful API 中无处不在的原因之一。Go 也提供对基于二进制的数据序列化格式的良好支持，包括 Gob，它几乎可以替代 JSON。Gob 是 Go 的本地二进制数据序列化格式，旨在高效且易于使用。

如果你在寻找一个支持更广泛的二进制数据序列化格式，可以考虑协议缓冲区。Google 设计协议缓冲区是为了方便其支持的平台和编程语言之间交换序列化的二进制数据。许多现代编程语言目前都提供对协议缓冲区的支持。尽管协议缓冲区不像 Gob 对 JSON 那样是 Go 的直接替代品，但 Go 仍然对协议缓冲区提供了出色的支持。你首先需要在 *.proto* 文件中添加定义，定义你打算序列化的数据结构。然后，使用协议缓冲区编译器及其 Go 模块生成与定义的数据结构对应的 Go 代码。最后，使用生成的代码将数据结构序列化为协议缓冲区格式的二进制数据，并将该二进制数据反序列化回你的数据结构。

gRPC 框架是一个高性能、平台中立的标准，用于在网络上进行分布式函数调用。*gRPC* 中的 *RPC* 代表 *远程过程调用*，这是一种透明地在远程系统上调用函数并接收结果的技术，就像你在本地系统上执行该函数一样。gRPC 使用协议缓冲区作为其底层数据序列化格式。Go 的协议缓冲区模块允许你通过在 *.proto* 文件中定义服务并利用生成的代码，轻松地添加 gRPC 支持。这使得你能够快速高效地搭建分布式服务或与现有的 gRPC 服务进行集成。
