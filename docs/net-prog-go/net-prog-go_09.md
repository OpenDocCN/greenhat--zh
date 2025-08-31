# 第七章：Unix 域套接字

![](img/chapterart.png)

到目前为止，在本书中，我们已经讨论了网络节点之间的通信。但并非所有网络编程都专门在不同节点之间进行。您的应用程序有时可能需要与在同一节点上托管的服务（例如数据库）进行通信。

将应用程序连接到在同一系统上运行的数据库的一种方法是将数据发送到节点的 IP 地址或本地主机地址（通常为 127.0.0.1）以及数据库的端口号。但是，还有另一种方法：使用 Unix 域套接字。Unix 域套接字是一种使用文件系统确定数据包目的地地址的通信方法，允许在同一节点上运行的服务之间交换数据，这个过程称为进程间通信（IPC）。

本章首先精确定义了 Unix 域套接字是什么，以及如何控制对它们的读写访问。接下来，您将通过 Go 的`net`包探索三种 Unix 域套接字的类型，并在每种类型中编写一个简单的回显服务器。最后，您将编写一个使用 Unix 域套接字的服务，根据客户端的用户和组 ID 信息进行身份验证。

## 什么是 Unix 域套接字？

在第二章中，我将网络套接字定义为 IP 地址和端口号。套接字定址允许同一节点上的各个服务监听传入的流量。为了说明套接字定址的重要性，想象一下在大公司只有一条电话线的低效率。如果您想要与某人交谈，最好希望电话没有被占线。这就是为什么为了缓解拥塞，大多数公司为每个员工分配一个分机号。这使您可以通过拨打公司的电话号码（类似于节点的 IP 地址）后接员工的分机号（类似于端口号）来联系您想要交谈的人。正如电话号码和分机号允许您单独呼叫公司中的每个人一样，套接字地址的 IP 地址和端口号允许您与节点上每个套接字地址监听的每个服务通信。

*Unix 域套接字*将套接字寻址原理应用于文件系统：每个 Unix 域套接字在文件系统上都有一个关联的文件，该文件对应于网络套接字的 IP 地址和端口号。你可以通过读取和写入这个文件与监听该套接字的服务进行通信。同样，你可以利用文件系统的所有权和权限来控制对该套接字的读写访问。Unix 域套接字通过绕过操作系统的网络栈来提高效率，消除了流量路由的开销。出于同样的原因，在使用 Unix 域套接字时，你不需要担心分片或数据包排序问题。如果你选择放弃 Unix 域套接字，专门使用网络套接字与本地服务进行通信（例如，将应用程序连接到本地数据库、内存缓存等），你将忽视显著的安全优势和性能提升。

尽管这一系统带来了明显的优势，但它也有一个警告：Unix 域套接字仅限于使用它们的节点，因此你不能像使用网络套接字那样与其他节点进行通信。因此，如果你预见到将服务迁移到其他节点，或者需要最大程度的应用程序可移植性，Unix 域套接字可能并不适合你。为了保持通信，你必须首先迁移到网络套接字。

## 绑定到 Unix 域套接字文件

当你的代码尝试通过使用`net.Listen`、`net.ListenUnix`或`net.ListenPacket`函数绑定到一个未使用的 Unix 域套接字地址时，将会创建一个 Unix 域套接字文件。如果该地址的套接字文件已经存在，操作系统会返回一个错误，指示该地址正在使用。大多数情况下，简单地删除现有的 Unix 域套接字文件就足以解决错误。然而，你应首先确保套接字文件存在，并非因为某个进程正在使用该地址，而是因为你没有正确清理来自一个已经终止的进程的文件。

如果你希望重用一个套接字文件，可以使用`net`包的`FileListener`函数来绑定到一个已存在的套接字文件。这个函数超出了本书的范围，但我鼓励你阅读它的文档。

### 更改套接字文件的所有权和权限

一旦服务绑定到套接字文件，你可以使用 Go 的`os`包修改文件的所有权和读写权限。具体来说，`os.Chown`函数允许你修改文件的用户和组所有者。Windows 不支持此函数，但在 Windows Subsystem for Linux（WSL）、Linux、macOS 等平台上支持此函数，这些都超出了本书的范围。我们现在将查看更改文件所有权和权限的代码行，但将在本章稍后在上下文中进行讲解。

以下命令指示操作系统更新给定文件的用户和组所有权：

```
err := os.Chown("/path/to/socket/file", 1-1, 2100)
```

`os.Chown` 函数接受三个参数：文件路径、所有者的用户 ID 和所有者的组 ID。如果用户或组 ID 为 `-1`，则告诉 Go 保持当前的用户或组 ID。在此示例中，你希望保持套接字文件的当前用户 ID，但将其组 ID 设置为 100，这里假定它是 */etc/group* 文件中的有效组 ID。

Go 的 `os/user` 包包含帮助你在用户和组名称与 ID 之间进行转换的函数。例如，这行代码使用 `LookupGroup` 函数查找 *users* 组的组 ID：

```
grp, err := user.LookupGroup("users")
```

如果提供的 `user.LookupGroup` 没有返回错误，`grp` 变量的 `Gid` 字段将包含 *users* 组的组 ID。

`os.Chmod` 函数更改文件的模式以及 Unix 兼容的权限位的数字表示法。这些权限位通知操作系统文件的模式、文件的用户读/写/执行权限、文件的组读/写/执行权限，以及对于任何不在文件组中的用户的读/写/执行权限：

```
err := os.Chmod("/path/to/socket/file", os.ModeSocket|0660)
```

`os.Chmod` 函数接受一个文件路径和一个 `os.FileMode`，后者表示文件模式、用户权限、组权限和非组用户权限。由于你正在处理一个套接字文件，应该始终在文件上设置 `os.ModeSocket` 模式。你可以通过 `os.ModeSocket` 与数字文件权限符号之间的按位或操作来做到这一点。在这里，你传递的是八进制的 `0660`，这意味着用户和组有读写权限，但阻止组外的任何人读取或写入套接字。你可以在 Go 的文档中阅读更多关于 `os.FileMode` 的内容，访问 [`golang.org/pkg/os/#FileMode`](https://golang.org/pkg/os/#FileMode)，并在 [`en.wikipedia.org/wiki/File_system_permissions#Numeric_notation`](https://en.wikipedia.org/wiki/File_system_permissions#Numeric_notation) 上熟悉文件系统权限的数字表示法。

### 理解 Unix 域套接字类型

Unix 域套接字有三种类型：*流套接字*，其工作方式类似于 TCP；*数据报套接字*，其工作方式类似于 UDP；以及 *序列数据包套接字*，其结合了前两者的特点。Go 分别将这些类型指定为 `unix`、`unixgram` 和 `unixpacket`。在本节中，我们将编写与每种类型配合使用的回显服务器。

`net.Conn` 接口允许你一次编写代码并在多种网络类型中使用它。它抽象了 TCP、UDP 和 Unix 域套接字所使用的网络套接字之间的许多差异，这意味着你可以将为 TCP 通信编写的代码拿来直接用于 Unix 域套接字，只需更改地址和网络类型即可。

#### Unix 流套接字

流式 Unix 域套接字的工作原理类似于 TCP，但没有 TCP 的确认、校验和、流控制等开销。操作系统负责通过 Unix 域套接字实现流式进程间通信，代替了 TCP。

为了说明这种 Unix 域套接字，让我们编写一个函数来创建一个通用的基于流的回显服务器（示例 7-1）。你将能够使用这个函数与任何流式网络类型一起使用。这意味着你可以用它来创建一个到不同节点的 TCP 连接，但你也可以将它与`unix`类型一起使用，来与 Unix 套接字地址通信。

```
package echo

import (
    "context"
    "net"
)

func 1streamingEchoServer(ctx context.Context, network string,
 addr string) (net.Addr, error) {
    s, err := 2net.Listen(network, addr)
    if err != nil {
        return nil, err
    }
```

示例 7-1：创建流式回显服务器函数 (*echo.go*)

`streamingEchoServer`函数 1 接受一个表示基于流的网络的字符串和一个表示地址的字符串，并返回一个地址对象和一个`error`接口。你应该从本书前面的内容中认识到这些参数和返回类型。

由于你通过接受上下文、网络字符串和地址字符串使回显服务器变得更加通用，你可以将任何基于流的网络类型传递给它，如`tcp`、`unix`或`unixpacket`。地址需要与网络类型相对应。上下文用于向服务器发出关闭信号。如果网络类型是`tcp`，地址字符串必须是 IP 地址和端口的组合，如 127.0.0.1:80。如果网络类型是`unix`或`unixpacket`，地址必须是指向一个不存在文件的路径。套接字文件将在回显服务器绑定到该地址时创建 2。然后服务器会开始监听传入的连接。

示例 7-2 完成了流式回显服务器的实现。

```
`--snip--`

    go func() {
        go func() {
          1<-ctx.Done()
            _ = s.Close()
        }()

        for {
            conn, err := 2s.Accept()
            if err != nil {
                return
            }

            go func() {
                defer func() { _ = conn.Close() }()

                for {
                    buf := make([]byte, 1024)
                    n, err := 3conn.Read(buf)
                    if err != nil {
                        return
                    }

                    _, err = 4conn.Write(buf[:n])
                    if err != nil {
                        return
                    }
                }
            }()
 }
    }()

    return s.Addr(), nil
}
```

示例 7-2：基于流的回显服务器 (*echo.go*)

使用`net.Listen`或`net.ListenUnix`创建的监听器在退出时会自动删除套接字文件。你可以通过`net.UnixListener`的`SetUnlinkOnClose`方法来修改这种行为，尽管默认行为对大多数使用案例来说是理想的。使用`net.ListenPacket`创建的 Unix 域套接字文件在监听器退出时不会自动删除，稍后在本章中你会看到这一点。

如前所述，你将回显服务器放入自己的 goroutine 中，以便它能够异步接受连接。一旦服务器接受了连接 2，你就会启动一个 goroutine 来回显接收到的消息。由于你使用的是`net.Conn`接口，你可以使用其`Read`3 和`Write`4 方法与客户端进行通信，无论服务器是通过网络套接字还是 Unix 域套接字进行通信。一旦调用者取消了上下文 1，服务器就会关闭。

示例 7-3 测试了通过`unix`网络类型使用 Unix 域套接字的流式回显服务器。

```
package echo

import (
    "bytes"
    "context"
    "fmt"
    "io/ioutil"
    "net"
    "os"
    "path/filepath"
    "testing"
)

func TestEchoServerUnix(t *testing.T) {
    dir, err := 1ioutil.TempDir("", "echo_unix")
    if err != nil {
        t.Fatal(err)
    }
    defer func() {
        if rErr := 2os.RemoveAll(dir); rErr != nil {
            t.Error(rErr)
        }
    }()

    ctx, cancel := context.WithCancel(context.Background())
    3 socket := filepath.Join(dir, fmt.Sprintf("%d.sock", os.Getpid()))
    rAddr, err := streamingEchoServer(ctx, "unix", socket)
    if err != nil {
 t.Fatal(err)
    }

    err = 4os.Chmod(socket, os.ModeSocket|0666)
    if err != nil {
        t.Fatal(err)
    }
```

示例 7-3：通过`unix`域套接字设置回显服务器测试 (*echo_test.go*)

你在操作系统的临时目录中创建一个名为 *echo_unix*1 的子目录，该子目录将包含回显服务器的套接字文件。对 `os.RemoveAll` 的延迟调用会在测试完成时清理服务器 2，通过删除临时子目录。你将一个名为 *#.sock*3 的套接字文件传递给 `streamingEchoServer` 函数，其中 *#* 是服务器的进程 ID，保存于临时子目录（*/tmp/echo_unix/123.sock*）中。最后，你确保所有人都能对套接字具有读写权限 4。

列表 7-4 连接到流回显服务器并发送测试。

```
`--snip--`

    conn, err := net.Dial("unix", 1rAddr.String())
    if err != nil {
        t.Fatal(err)
    }
    defer func() { _ = conn.Close() }()

    msg := []byte("ping")
    2 for i := 0; i < 3; i++ { // write 3 "ping" messages
        _, err = conn.Write(msg)
        if err != nil {
            t.Fatal(err)
        }
    }

    buf := make([]byte, 1024)
    n, err := 3conn.Read(buf) // read once from the server
    if err != nil {
        t.Fatal(err)
    }

    expected := 4bytes.Repeat(msg, 3)
    if !bytes.Equal(expected, buf[:n]) {
        t.Fatalf("expected reply %q; actual reply %q", expected,
            buf[:n])
    }

    _ = closer.Close()
    <-done
}
```

列表 7-4：通过 Unix 域套接字传输流数据 (*echo_test.go*)

你通过使用熟悉的 `net.Dial` 函数来拨号连接服务器。它接受 `unix` 网络类型和服务器地址，即 Unix 域套接字文件的完整路径 1。

在读取第一个响应之前，你向回显服务器发送了三条 ping 消息 2。当你稍后在本章中探索 `unixpacket` 类型时，发送三条独立的 ping 消息的原因会变得很清楚。当你使用足够大的缓冲区读取第一个响应 3 来存储刚刚发送的三条消息时，你会以字符串 `pingpingping` 的形式接收到所有三条 ping 消息 4。请记住，基于流的连接并不区分消息。你需要自己确定当从服务器读取一系列字节时，一条消息的结束和另一条消息的开始。

#### unixgram 数据报套接字

接下来，让我们创建一个使用基于数据报的网络类型（如 `udp` 和 `unixgram`）进行通信的回显服务器。无论你是通过 UDP 还是 `unixgram` 套接字进行通信，你编写的服务器基本上是相同的。不同之处在于，你需要使用 `unixgram` 监听器清理套接字文件，正如你将在列表 7-5 中看到的那样。

```
`--snip--`

func datagramEchoServer(ctx context.Context, network string,
    addr string) (net.Addr, error) {
    s, err := 1net.ListenPacket(network, addr)
    if err != nil {
        return nil, err
    }

    go func() {
        go func() {
            <-ctx.Done()
            _ = s.Close()
            if network == "unixgram" {
                _ = 2os.Remove(addr)
            }
        }()

        buf := make([]byte, 1024)
        for {
            n, clientAddr, err := s.ReadFrom(buf)
            if err != nil {
                return
            }

            _, err = s.WriteTo(buf[:n], clientAddr)
            if err != nil {
                return
 }
        }
    }()

    return s.LocalAddr(), nil
}
```

列表 7-5：基于数据报的回显服务器 (*echo.go*)

你调用 `net.ListenPacket`1，它返回一个 `net.PacketConn`。如本章前面所述，由于你没有使用 `net.Listen` 或 `net.ListenUnix` 创建监听器，Go 在服务器完成后不会为你清理套接字文件。你必须确保自己移除套接字文件 2，否则后续尝试绑定到现有套接字文件时会失败。

由于 `unixgram` 网络类型在 Windows 上无法工作，列表 7-6 使用构建约束确保这段代码不会在 Windows 上运行，并导入必要的包。

```
// +build darwin linux

package echo

import (
    "bytes"
    "context"
    "fmt"
    "io/ioutil"
    "net"
    "os"
    "path/filepath"
    "testing"
)
```

列表 7-6：适用于 macOS 和 Linux 的构建约束和导入 (*echo_posix_test.go*)

构建约束告诉 Go 仅在 macOS 或 Linux 操作系统上运行时包含这段代码。诚然，Go 支持其他操作系统，其中许多可能提供 `unixgram` 支持，但这些操作系统超出了本书的范围。此构建约束没有考虑到这些其他操作系统，因此我鼓励你在目标操作系统上测试这段代码。

在设置构建约束后，您可以在清单 7-7 中添加测试。

```
`--snip--`

func TestEchoServerUnixDatagram(t *testing.T) {
    dir, err := ioutil.TempDir("", "echo_unixgram")
    if err != nil {
        t.Fatal(err)
    }
    defer func() {
        if rErr := os.RemoveAll(dir); rErr != nil {
            t.Error(rErr)
 }
    }()

    ctx, cancel := context.WithCancel(context.Background())
    1 sSocket := filepath.Join(dir, fmt.Sprintf("s%d.sock", os.Getpid()))
    serverAddr, err := datagramEchoServer(ctx, "unixgram", sSocket)
    if err != nil {
        t.Fatal(err)
    }
    defer cancel()

    err = os.Chmod(sSocket, os.ModeSocket|0622)
    if err != nil {
        t.Fatal(err)
    }
```

清单 7-7：实例化基于数据报的回显服务器（*echo_posix_test.go*）

就像 UDP 连接一样，服务器和客户端都必须绑定到一个地址，以便它们可以发送和接收数据报。服务器有自己的套接字文件 1，该文件与客户端在清单 7-8 中的套接字文件是分开的。

```
`--snip--`

    1 cSocket := filepath.Join(dir, fmt.Sprintf("c%d.sock", os.Getpid()))
    client, err := net.ListenPacket("unixgram", cSocket)
    if err != nil {
        t.Fatal(err)
    }
    2 defer func() { _ = client.Close() }()

    err = 3os.Chmod(cSocket, os.ModeSocket|0622)
    if err != nil {
        t.Fatal(err)
    }
```

清单 7-8：实例化基于数据报的客户端（*echo_posix_test.go*）

在清单 7-5 的 `datagramEchoServer` 函数中调用 `os.Remove` 来清理服务器关闭时的套接字文件。客户端有一些额外的清理工作，因此当客户端完成监听后，您需要让客户端清理自己的套接字文件 1。幸运的是，在清单 7-7 中通过调用 `os.RemoveAll` 来删除临时子目录，已经为您处理了这个问题。否则，您需要在 `defer`2 中添加一个调用 `os.Remove` 来删除客户端的套接字文件。此外，服务器应该能够写入客户端的套接字文件以及它自己的套接字文件，否则服务器将无法回复消息。在这个例子中，您设置了非常宽松的权限，以便所有用户都可以写入套接字 3。

现在服务器和客户端都已实例化，清单 7-9 测试了流式回显服务器和数据报回显服务器之间的区别。

```
`--snip--`

    msg := []byte("ping")
    for i := 0; i < 3; i++ { // write 3 "ping" messages
        _, err = 1client.WriteTo(msg, serverAddr)
        if err != nil {
            t.Fatal(err)
        }
    }

    buf := make([]byte, 1024)
    for i := 0; i < 3; i++ { // read 3 "ping" messages
        n, addr, err := 2client.ReadFrom(buf)
        if err != nil {
            t.Fatal(err)
        }

        if addr.String() != serverAddr.String() {
            t.Fatalf("received reply from %q instead of %q", 
                addr, serverAddr)
        }

        if !bytes.Equal(msg, buf[:n]) {
            t.Fatalf("expected reply %q; actual reply %q", msg,
                buf[:n])
        }
    }
}
```

清单 7-9：使用 unixgram 套接字回显消息（*echo_posix_test.go*）

您在读取第一个数据报之前向服务器 1 发送三条 ping 消息。然后，您执行三次读取 2，使用一个足够大的缓冲区来容纳三条 ping 消息。如预期，`unixgram` 套接字保持消息之间的划分；您发送了三条消息并读取了三条回复。与清单 7-3 和 7-4 中的 `unix` 套接字类型相比，您发送了三条消息并通过连接一次读取收到了三条回复。

#### unixpacket 序列数据包套接字

*序列数据包套接字* 类型是一个混合类型，它结合了 TCP 的面向会话的连接和可靠性，以及 UDP 清晰划分的数据报。然而，序列数据包套接字会丢弃每个数据报中的未请求数据。例如，如果您读取了一个 50 字节数据报中的 32 字节，操作系统会丢弃剩余的 18 个未请求字节。

在三种 Unix 域套接字类型中，`unixpacket` 具有最少的跨平台支持。再加上 `unixpacket` 的混合行为和丢弃未请求数据的特点，`unix` 或 `unixgram` 更适合大多数应用。您不太可能在互联网上找到序列数据包套接字的应用。它主要用于旧的 X.25 电信网络、某些类型的金融交易以及业余无线电中使用的 AX.25。

清单 7-10 中的测试代码展示了 `unixpacket` 套接字的演示。

```
package echo

import (
    "bytes"
    "context"
    "fmt"
    "io/ioutil"
    "net"
    "os"
    "path/filepath"
    "testing"
)

func TestEchoServerUnixPacket(t *testing.T) {
    dir, err := ioutil.TempDir("", "echo_unixpacket")
    if err != nil {
        t.Fatal(err)
    }
    defer func() {
        if rErr := os.RemoveAll(dir); rErr != nil {
            t.Error(rErr)
        }
    }()

    ctx, cancel := context.WithCancel(context.Background())
    socket := filepath.Join(dir, fmt.Sprintf("%d.sock", os.Getpid()))
    rAddr, err := streamingEchoServer(ctx, "unixpacket", socket)
    if err != nil {
        t.Fatal(err)
    }
    defer cancel()

    err = os.Chmod(socket, os.ModeSocket|0666)
    if err != nil {
        t.Fatal(err)
    }
```

清单 7-10：实例化基于数据包的流式回显服务器（*echo_linux_test.go*）

首先注意，你将此代码保存在一个名为 *echo_linux_test.go* 的文件中。* _linux_test.go* 后缀是一个构建约束，告知 Go 仅在 Linux 上运行测试时包含此文件。

列表 7-11 拨打回声服务器并发送一系列 ping 消息。

```
`--snip--`

    conn, err := 1net.Dial("unixpacket", rAddr.String())
    if err != nil {
 t.Fatal(err)
    }
    defer func() { _ = conn.Close() }()

    msg := []byte("ping")
    2 for i := 0; i < 3; i++ { // write 3 "ping" messages
        _, err = conn.Write(msg)
        if err != nil {
            t.Fatal(err)
        }
    }

    buf := make([]byte, 1024)
    3 for i := 0; i < 3; i++ { // read 3 times from the server
        n, err := conn.Read(buf)
        if err != nil {
            t.Fatal(err)
        }

        if !bytes.Equal(msg, buf[:n]) {
            t.Errorf("expected reply %q; actual reply %q", msg, buf[:n])
        }
    }
```

列表 7-11：使用 `unixpacket` 套接字回显消息 (*echo_linux_test.go*)

由于 `unixpacket` 是面向会话的，你使用 `net.Dial`1 来启动与服务器的连接。你不会像在数据报式网络类型下那样直接向服务器地址写入。

你可以通过向服务器 2 写入三条 ping 消息，在读取第一个回复之前，看到 `unix` 和 `unixpacket` 套接字类型之间的区别。而 `unix` 套接字类型会将所有三条 ping 消息合并为一次读取返回，而 `unixpacket` 就像其他基于数据报的网络类型一样，每次读取返回一条消息 3。

列表 7-12 说明了 `unixpacket` 如何在每个数据报中丢弃未请求的数据。

```
`--snip--`

    for i := 0; i < 3; i++ { // write 3 more "ping" messages
        _, err = conn.Write(msg)
        if err != nil {
            t.Fatal(err)
        }
    }

    1 buf = make([]byte, 2)    // only read the first 2 bytes of each reply
    for i := 0; i < 3; i++ { // read 3 times from the server
        n, err := conn.Read(buf)
        if err != nil {
            t.Fatal(err)
        }

        if !bytes.Equal(2msg[:2], buf[:n]) {
 t.Errorf("expected reply %q; actual reply %q", msg[:2],
                buf[:n])
        }
    }
}
```

列表 7-12：丢弃未读取的字节 (*echo_linux_test.go*)

这一次，你将缓冲区大小减少为 2 字节 1，并读取每个数据报的前 2 字节。如果你使用的是类似 `tcp` 或 `unix` 的流式网络类型，你会期待第一次读取时得到 `pi`，第二次读取时得到 `ng`。但 `unixpacket` 会丢弃 `ping` 消息中的 `ng` 部分，因为你只请求了前 2 字节—`pi`。因此，你确保每次读取时都只接收数据报的前 2 字节 2。

## 编写一个验证客户端身份的服务

在 Linux 系统上，Unix 域套接字允许你通过接收对端操作系统的凭证，获取有关对端进程的详细信息。你可以使用这些信息来验证 Unix 域套接字另一端的对端身份，并在对端凭证不符合你的标准时拒绝访问。例如，如果用户 *davefromaccounting* 通过 Unix 域套接字连接到你的管理服务，对端凭证可能表明你应该拒绝访问；Dave 应该在处理数字，而不是向你的管理服务发送数据。

你可以创建一个仅允许特定用户或在 * /etc/groups* 文件中找到的特定组中的任何用户连接的服务。* /etc/groups* 文件中的每个命名组都有一个对应的组 ID。当客户端连接到你的 Unix 域套接字时，你可以请求对端凭证，并将客户端的组 ID 与任何允许组的组 ID 进行比较。如果客户端的组 ID 与允许的组 ID 匹配，则可以认为客户端已通过身份验证。Go 的标准库对于处理 Linux 组提供了有用的支持，你将在“编写服务”一章中使用这些内容（第 156 页）。

### 请求对端凭证

请求对等凭证的过程并不完全简单。你不能仅仅从连接对象本身请求对等凭证。相反，你需要使用 `golang.org/x/sys/unix` 包从操作系统请求对等凭证，可以使用以下命令获取：

```
go get -u golang.org/x/sys/unix
```

列表 7-13 显示了一个函数，该函数接受一个 Unix 域套接字连接，并在对等方不是特定组的成员时拒绝访问。

```
package auth

import (
    "log"
    "net"
    "golang.org/x/sys/unix"
)

func Allowed(conn *net.UnixConn, groups map[string]struct{}) bool {
    if conn == nil || groups == nil || len(groups) == 0 {
        return false
    }

    file, _ := 1conn.File()
    defer func(){ _ = file.Close() }()

    var (
        err   error
        ucred *unix.Ucred
    )

    for {
        ucred, err = 2unix.GetsockoptUcred(int(3file.Fd()), unix.SOL_SOCKET,
            unix.SO_PEERCRED)
        if err == unix.EINTR {
            continue // syscall interrupted, try again
        }
        if err != nil {
            log.Println(err)
            return false
        }

        break
    }

    u, err := 4user.LookupId(string(ucred.Uid))
    if err != nil {
        log.Println(err)
        return false
    }

    gids, err := 5u.GroupIds()
    if err != nil {
        log.Println(err)
        return false
    }

    for _, gid := range gids {
        if _, ok := 6groups[gid]; ok {
 return true
        }
    }

    return false
}
```

列表 7-13：获取套接字连接的对等凭证 (*creds/auth/allowed_linux.go*)

要检索对等方的 Unix 凭证，你首先需要从 `net.UnixConn`1 中获取底层文件对象，它表示你这一方的 Unix 域套接字连接。它类似于 Go 中 TCP 连接的 `net.TCPConn`。由于你需要从连接中提取文件描述符的详细信息，因此不能仅仅依赖从监听器的 `Accept` 方法收到的 `net.Conn` 接口。相反，你的 `Allowed` 函数需要调用者传递指向底层 `net.UnixConn` 对象的指针，通常这是从监听器的 `AcceptUnix` 方法返回的。你将在下一节中看到这个方法的实际应用。

然后，你可以将文件对象的描述符 3、协议级别的 `unix.SOL_SOCKET` 和选项名称 `unix.SO_PEERCRED` 传递给 `unix.GetsockoptUcred` 函数 2。要从 Linux 内核检索套接字选项，必须指定你想要的选项以及选项所在的级别。`unix.SOL_SOCKET` 告诉 Linux 内核你需要一个套接字级别的选项，而不是比如说 `unix.SOL_TCP`，它表示 TCP 级别的选项。`unix.SO_PEERCRED` 常量告诉 Linux 内核你需要对等凭证选项。如果 Linux 内核在 Unix 域套接字级别找到对等凭证选项，`unix.GetsockoptUcred` 将返回指向有效的 `unix.Ucred` 对象的指针。

`unix.Ucred` 对象包含对等方的进程、用户和组 ID。你将对等方的用户 ID 传递给 `user.LookupId` 函数 4。如果成功，你将从用户对象中获取一组组 ID 5。用户可以属于多个组，你需要考虑每个组的访问权限。最后，你检查每个组 ID 是否存在于允许的组映射中 6。如果对等方的任何一个组 ID 在你的映射中，你就返回 `true`，允许对等方连接。

这个例子主要是教学性的。你可以通过将套接字文件的组所有权分配给相应的组来实现类似的结果，正如我们在第 143 页“更改套接字文件的所有权和权限”中讨论的那样。然而，了解组成员身份可以用于应用程序中的访问控制和其他安全决策。

### 编写服务

现在，让我们在一个可以从命令行运行的服务中使用此函数。此服务将接受 Linux 操作系统中 */etc/group* 文件中找到的一个或多个组名作为命令行参数，并开始监听 Unix 域套接字文件。只有当客户端是命令行中指定的某个组的成员时，服务才会允许客户端连接。客户端随后可以建立与该服务的 Unix 域套接字连接。服务将检索客户端的对等方凭证，并根据客户端是否是允许的组成员，决定是否允许其保持连接，或者立即断开未授权的客户端。该服务仅进行客户端组 ID 的身份验证，不执行其他操作。

在列表 7-14 中，你指定了所需的导入，并为该服务创建了一个有意义的使用消息。

```
package main

import (
    "flag"
    "fmt"
    "log"
    "net"
    "os"
    "os/signal"
    "os/user"
    "path/filepath"

    "github.com/awoodbeck/gnp/ch07/creds/auth"
)

func init() {
    flag.Usage = func() {
        _, _ = fmt.Fprintf(flag.CommandLine.Output(),
            "Usage:\n\t%s 1<group names>\n", filepath.Base(os.Args[0]))
        flag.PrintDefaults()
    }
}
```

列表 7-14：期望在命令行中传入组名（*creds/creds.go*）

我们的应用程序期望传入一系列组名作为参数 1。你将把每个组名的组 ID 添加到允许的组映射中。列表 7-15 中的代码解析了这些组名。

```
`--snip--`

func parseGroupNames(args []string) map[string]struct{} {
    groups := make(map[string]struct{})

    for _, arg := range args {
        grp, err := 1user.LookupGroup(arg)
        if err != nil {
            log.Println(err)
            continue
        }

        groups[2grp.Gid] = struct{}{}
    }

    return groups
}
```

列表 7-15：将组名解析为组 ID（*creds/creds.go*）

`parseGroupNames` 函数接受一个包含组名的字符串切片，获取每个组名的组信息 1，并将每个组的 ID 插入到 `groups` 映射 2 中。

列表 7-16 将最后几项列表结合成一个可以从命令行连接的服务。

```
`--snip--`

func main() {
    flag.Parse()

    groups := parseGroupNames(flag.Args())
    socket := filepath.Join(os.TempDir(), "creds.sock")
    addr, err := net.ResolveUnixAddr("unix", socket)
    if err != nil {
        log.Fatal(err)
    }

    s, err := net.ListenUnix("unix", addr)
    if err != nil {
        log.Fatal(err)
    }

    c := make(chan os.Signal, 1)
    signal.Notify(c, 1os.Interrupt)
    2 go func() {
        <-c
        _ = s.Close()
    }()

    fmt.Printf("Listening on %s ...\n", socket)

    for {
        conn, err := 3s.AcceptUnix()
        if err != nil {
            break
        }
        if 4auth.Allowed(conn, groups) {
            _, err = conn.Write([]byte("Welcome\n"))
            if err == nil {
                // handle the connection in a goroutine here
                continue
            }
        } else {
            _, err = conn.Write([]byte("Access denied\n"))
        }
        if err != nil {
            log.Println(err)
        }
        _ = conn.Close()
    }
}
```

列表 7-16：基于凭证授权对等方（*creds/creds.go* 续）

你首先解析命令行参数，以创建允许的组 ID 映射。然后，你在 */tmp/creds.sock* 套接字上创建一个监听器。监听器通过 `AcceptUnix`3 接受连接，因此会返回一个 `*net.UnixConn`，而不是通常的 `net.Conn`，因为你的 `auth.Allowed` 函数需要一个 `*net.UnixConn` 类型作为第一个参数。接着，你确定对等方的凭证是否被允许 4。允许的对等方保持连接，不允许的对等方会立即断开连接。

由于你将在命令行中执行此服务，因此可以通过发送中断信号来停止该服务，通常是使用 ctrl-C 键组合。然而，这个信号会在 Go 有机会清理套接字文件之前突然终止服务，尽管你已经小心使用了 `net.ListenUnix`。因此，你需要监听这个信号 1，并在接收到信号后启动一个 goroutine，在其中优雅地关闭监听器 2。这将确保 Go 正确地清理套接字文件。

### 使用 Netcat 测试服务

Netcat 是一个流行的命令行工具，它允许你建立 TCP、UDP 和 Unix 域套接字连接。你将使用它从命令行测试该服务。你可以在 Linux 发行版的包管理器中找到 Netcat。例如，在 Debian 10 上安装 Netcat 的 OpenBSD 重写版本，运行以下命令：

```
$ **sudo apt install netcat-openbsd**
```

该命令使用`sudo`命令行工具，以伪装成`root`用户的身份运行`apt install netcat-openbsd`。CentOS 8.1 提供了 Nmap 的 Netcat 替代品。运行此命令来安装它：

```
$ **sudo dnf install nmap-ncat**
```

安装完成后，你应该能在`PATH`环境变量中找到*nc*二进制文件。

在你可以连接到凭证检查服务之前，你需要运行该服务，使其绑定到一个套接字文件：

```
$ **cd $GOPATH/src/github.com/awoodbeck/gnp/ch07/creds**
$ **go run . -- users staff**
Listening on /tmp/creds.sock …
```

在这个例子中，你允许来自`users`或`staff`组的任何对等体连接。服务将拒绝任何不属于这两个组中的至少一个的对等体访问。如果这些组在你的 Linux 发行版中不存在，可以选择`/etc/groups`文件中的任何组。该服务监听`/tmp/creds.sock`套接字文件，这是你提供给 Netcat 的地址。

接下来，你需要一种方法来更改你的组 ID，以便测试服务是否拒绝访问未被允许的客户端。当前，服务是以你的用户和组 ID 运行的，因为你启动了该服务。因此，它会接受所有连接，因为服务允许其自己的组（即我们的组）进行身份验证，依据 Listing 7-15 中的`groups`映射。要在启动与服务的套接字连接时更改组，可以使用`sudo`命令行工具。

由于使用`sudo`需要提升权限，通常在尝试执行该命令时，系统会提示输入密码。我已省略以下示例中的密码提示，但在第一次调用`sudo`时，系统会提示输入密码：

```
$ **sudo -g staff -- nc -U /tmp/creds.sock**
Welcome
**^C**
$
```

使用`sudo`，你通过将组名传递给`-g`标志来修改你的组。在这种情况下，你将组设置为`staff`。然后，你执行`nc`命令。`-U`标志告诉 Netcat 建立与`/tmp/creds.sock`文件的 Unix 域套接字连接。

由于`staff`组是允许的组之一，连接后你会收到`Welcome`消息。按下 ctrl-C 终止你的连接。

如果你使用一个不允许的组重复测试，你应该会收到相反的结果：

```
$ **sudo -g nogroup -- nc -U /tmp/creds.sock**
Access denied
$
```

这次，你使用`nogroup`组，而该服务不允许该组。正如预期的那样，你立即收到`Access denied`消息，并且套接字的服务器端终止了你的连接。

## 你学到的内容

本章开始时，我们介绍了 Unix 域套接字。Unix 域套接字是一种基于文件的通信方法，用于同一节点上运行的进程之间的通信。两个或多个进程，比如本地数据库服务器和客户端，可以通过 Unix 域套接字发送和接收数据。由于 Unix 域套接字依赖文件系统进行寻址，因此你可以利用文件系统的所有权和权限来控制通过 Unix 域套接字进行通信的进程的访问。

你接着学习了 Go 支持的 Unix 域套接字类型：`unix`、`unixgram` 和 `unixpacket`。Go 使得通过 Unix 域套接字进行通信变得相对轻松，并且处理了许多细节，特别是如果你坚持使用 `net` 包的接口。例如，针对基于流的 TCP 网络编写的代码也能在 `unix` 域套接字（仅限本地进程通信）上工作，几乎不需要修改。同样，针对 UDP 网络编写的代码可以通过 `unixgram` 域套接字来使用。你还了解了混合型的 Unix 域套接字类型 `unixpacket`，并学到了它的缺点在大多数应用中并不超过其优点，特别是在跨平台支持方面。对于大多数使用场景，另外两种 Unix 域套接字类型是更好的选择。

本章介绍了对等凭据，并展示了如何使用它们来验证客户端连接。你可以超越基于文件的 Unix 域套接字访问限制，获取 Unix 域套接字另一端客户端的详细信息。

现在你应该具备了判断 Unix 域套接字在网络栈中最佳应用位置的能力。
