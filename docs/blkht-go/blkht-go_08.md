## 第八章：原始数据包处理

![Image](img/common.jpg)

在本章中，你将学习如何捕获和处理网络数据包。你可以将数据包处理用于多种目的，包括捕获明文认证凭证、修改数据包的应用程序功能，或者伪造和投毒流量。你还可以将其用于 SYN 扫描和通过 SYN 洪水保护进行端口扫描等。

我们将向你介绍 Google 的优秀 `gopacket` 包，它将帮助你解码数据包并重新组装流量流。该包允许你使用伯克利数据包过滤器（BPF，亦称为 tcpdump 语法）过滤流量；读取和写入 *.pcap* 文件；检查各种层和数据；以及操作数据包。

我们将通过几个示例演示如何识别设备、过滤结果，并创建一个能够绕过 SYN 洪水保护的端口扫描器。

### 设置你的开发环境

在本章的代码示例之前，你需要设置好你的环境。首先，输入以下命令安装 `gopacket`：

```
$ go get github.com/google/gopacket
```

现在，`gopacket` 依赖于外部库和驱动程序来绕过操作系统的协议栈。如果你打算为 Linux 或 macOS 编译本章中的示例，你需要安装 `libpcap-dev`。你可以使用大多数包管理工具，如 `apt`、`yum` 或 `brew` 来完成安装。以下是使用 `apt` 安装它的方法（其他两个选项的安装过程类似）：

```
$ sudo apt-get install libpcap-dev
```

如果你打算在 Windows 上编译并运行本章中的示例，你有几种选择，具体取决于你是否打算进行交叉编译。如果不进行交叉编译，设置开发环境会更简单，但在这种情况下，你需要在 Windows 机器上创建一个 Go 开发环境，如果你不想让其他环境变得混乱，这可能会显得不太吸引人。目前，我们假设你有一个可以用来编译 Windows 二进制文件的工作环境。在这个环境中，你需要安装 WinPcap。你可以从 *[`www.winpcap.org`](https://www.winpcap.org)* 免费下载安装程序。

### 使用 pcap 子包识别设备

在你能够捕获网络流量之前，你必须识别可以监听的可用设备。你可以使用 `gopacket/pcap` 子包轻松完成这项工作，它通过以下助手函数来获取设备：`pcap.FindAllDevs() (ifs []Interface, err error)`。列表 8-1 显示了如何使用它列出所有可用的接口。（所有位于根位置的代码列表都可以在提供的 GitHub 仓库 *[`github.com/blackhat-go/bhg/`](https://github.com/blackhat-go/bhg/)* 中找到。）

```
package main

import (
    "fmt"
    "log"

    "github.com/google/gopacket/pcap"
)

func main() {
 ❶ devices, err := pcap.FindAllDevs()
    if err != nil {
        log.Panicln(err)
    }
 ❷ for _, device := range devices {
        fmt.Println(device.Name❸)
     ❹ for _, address := range device.Addresses {
         ❺ fmt.Printf("    IP:      %s\n", address.IP)
            fmt.Printf("    Netmask: %s\n", address.Netmask)
        }  
    }
}
```

*列表 8-1：列出可用的网络设备 (*[/ch-8/identify/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-8/identify/main.go)*)*

你通过调用 `pcap.FindAllDevs()` ❶ 来枚举你的设备。然后你循环遍历找到的设备 ❷。对于每个设备，你访问各种属性，包括 `device.Name` ❸。你还可以通过 `Addresses` 属性访问它们的 IP 地址，这个属性是 `pcap.InterfaceAddress` 类型的切片。你遍历这些地址 ❹，将 IP 地址和子网掩码显示到屏幕上 ❺。

执行你的实用工具会产生类似于清单 8-2 的输出。

```
$ go run main.go
enp0s5
    IP:      10.0.1.20
    Netmask: ffffff00
    IP:      fe80::553a:14e7:92d2:114b
    Netmask: ffffffffffffffff0000000000000000
any
lo
    IP:      127.0.0.1
    Netmask: ff000000
    IP:      ::1
    Netmask: ffffffffffffffffffffffffffffffff
```

*清单 8-2：显示可用网络接口的输出*

输出列出了可用的网络接口——`enp0s5`、`any` 和 `lo`——以及它们的 IPv4 和 IPv6 地址和子网掩码。你系统上的输出可能与这些网络细节有所不同，但应该足够相似，以便你能理解这些信息。

### 实时捕获与过滤结果

现在你知道如何查询可用的设备，你可以使用 `gopacket` 的功能从网络上捕获实时数据包。在此过程中，你还将使用 BPF 语法过滤数据包。BPF 允许你限制捕获和显示的内容，从而只看到相关的流量。它通常用于按协议和端口过滤流量。例如，你可以创建一个过滤器，查看所有目标端口为 80 的 TCP 流量。你也可以按目标主机过滤流量。BPF 语法的详细讨论超出了本书的范围。如果想了解更多关于如何使用 BPF，请查看[*http://www.tcpdump.org/manpages/pcap-filter.7.html*](http://www.tcpdump.org/manpages/pcap-filter.7.html)。

清单 8-3 显示了代码，它过滤流量，以便你只捕获发送到或来自端口 80 的 TCP 流量。

```
   package main

   import (
       "fmt"
       "log"

       "github.com/google/gopacket"
       "github.com/google/gopacket/pcap"
   )

❶ var (
       iface    = "enp0s5"
       snaplen  = int32(1600)
       promisc  = false
       timeout  = pcap.BlockForever
       filter   = "tcp and port 80"
       devFound = false
   )  

   func main() {
       devices, err := pcap.FindAllDevs()❷
       if err != nil {
           log.Panicln(err)
       }

    ❸ for _, device := range devices {
           if device.Name == iface {
               devFound = true
           }
       }
       if !devFound {
           log.Panicf("Device named '%s' does not exist\n", iface)
       }

     ❹ handle, err := pcap.OpenLive(iface, snaplen, promisc, timeout)
       if err != nil {
           log.Panicln(err)
       }
       defer handle.Close()

    ❺ if err := handle.SetBPFFilter(filter); err != nil {
           log.Panicln(err)
       }

    ❻ source := gopacket.NewPacketSource(handle, handle.LinkType())
       for packet := range source.Packets()❼ {
           fmt.Println(packet)
       }
   }
```

*清单 8-3：使用 BPF 过滤器捕获特定的网络流量 (*[/ch-8/filter/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-8/filter/main.go)*)*

代码首先定义了几个必要的变量，以便设置数据包捕获 ❶。其中特别包括你想要捕获数据的接口名称、快照长度（每个帧捕获的数据量）、`promisc` 变量（决定是否启用混杂模式）以及超时时间。此外，你还定义了 BPF 过滤器：`tcp and port 80`。这将确保你只捕获符合这些条件的数据包。

在你的 `main()` 函数中，你枚举可用设备 ❷，并循环遍历它们以确定你想要的捕获接口是否存在于设备列表中 ❸。如果接口名称不存在，则会 panic，提示它无效。

剩下的 `main()` 函数部分是你的捕获逻辑。从一个高层次的角度来看，你需要首先获取或创建一个 `*pcap.Handle`，它允许你读取和注入数据包。使用这个句柄，你可以应用 BPF 过滤器，并创建一个新的数据包数据源，从中读取数据包。

你通过调用 `pcap.OpenLive()` ❹ 创建你的 `*pcap.Handle`（代码中命名为 `handle`）。此函数接受一个接口名称、一个快照长度、一个定义是否为混杂模式的布尔值以及一个超时值。这些输入变量在 `main()` 函数之前已经定义，如我们之前所述。调用 `handle.SetBPFFilter(filter)` 为你的句柄设置 BPF 过滤器 ❺，然后在调用 `gopacket.NewPacketSource(handle, handle.LinkType())` 创建新的数据包数据源 ❻ 时，将 `handle` 作为输入。第二个输入值 `handle.LinkType()` 定义了处理数据包时使用的解码器。最后，你通过在 `source.Packets()` ❼ 上进行循环来实际从网络中读取数据包，该函数返回一个通道。

正如你在本书之前的示例中可能记得，循环在通道上时，如果没有数据可供读取，循环会被阻塞。当数据包到达时，你读取它并将其内容打印到屏幕上。

输出应该类似于 列表 8-4。请注意，该程序需要提升权限，因为我们正在从网络读取原始内容。

```
$ go build -o filter && sudo ./filter
PACKET: 74 bytes, wire length 74 cap length 74 @ 2020-04-26 08:44:43.074187 -0500 CDT
- Layer 1 (14 bytes) = Ethernet   {Contents=[..14..] Payload=[..60..]
SrcMAC=00:1c:42:cf:57:11 DstMAC=90:72:40:04:33:c1 EthernetType=IPv4 Length=0}
- Layer 2 (20 bytes) = IPv4       {Contents=[..20..] Payload=[..40..] Version=4 IHL=5
TOS=0 Length=60 Id=998 Flags=DF FragOffset=0 TTL=64 Protocol=TCP Checksum=55712
SrcIP=10.0.1.20 DstIP=54.164.27.126 Options=[] Padding=[]}
- Layer 3 (40 bytes) = TCP        {Contents=[..40..] Payload=[] SrcPort=51064
DstPort=80(http) Seq=3543761149 Ack=0 DataOffset=10 FIN=false SYN=true RST=false
PSH=false ACK=false URG=false ECE=false CWR=false NS=false Window=29200
Checksum=23908 Urgent=0 Options=[..5..] Padding=[]}

PACKET: 74 bytes, wire length 74 cap length 74 @ 2020-04-26 08:44:43.086706 -0500 CDT
- Layer 1 (14 bytes) = Ethernet   {Contents=[..14..] Payload=[..60..]
SrcMAC=00:1c:42:cf:57:11 DstMAC=90:72:40:04:33:c1 EthernetType=IPv4 Length=0}
- Layer 2 (20 bytes) = IPv4       {Contents=[..20..] Payload=[..40..] Version=4 IHL=5
TOS=0 Length=60 Id=23414 Flags=DF FragOffset=0 TTL=64 Protocol=TCP Checksum=16919
SrcIP=10.0.1.20 DstIP=204.79.197.203 Options=[] Padding=[]}
- Layer 3 (40 bytes) = TCP        {Contents=[..40..] Payload=[] SrcPort=37314
DstPort=80(http) Seq=2821118056 Ack=0 DataOffset=10 FIN=false SYN=true RST=false
PSH=false ACK=false URG=false ECE=false CWR=false NS=false Window=29200
Checksum=40285 Urgent=0 Options=[..5..] Padding=[]}
```

*列表 8-4：捕获的数据包记录到 stdout*

尽管原始输出并不容易理解，但它确实很好地分离了每一层。你现在可以使用工具函数，例如 `packet.ApplicationLayer()` 和 `packet.Data()`，来检索单一层或整个数据包的原始字节。当你将输出与 `hex.Dump()` 结合使用时，你可以以更易读的格式显示内容。自己动手试试吧。

### 嗅探并显示明文用户凭据

现在让我们基于你刚刚创建的代码进行扩展。你将复制一些由其他工具提供的功能，来嗅探并显示明文的用户凭据。

现在，大多数组织都使用交换机网络，这种网络将数据直接发送到两个端点，而不是广播，使得在企业环境中被动捕获流量变得更加困难。然而，以下的明文嗅探攻击在与诸如地址解析协议（ARP）欺骗等技术结合时，可以非常有效，后者可以迫使端点与交换网络上的恶意设备进行通信，或者当你悄悄地嗅探被攻击用户工作站的出站流量时。在这个例子中，我们假设你已经入侵了一个用户工作站，并仅关注捕获使用 FTP 的流量，以保持代码简洁。

除了一些小的变化，列表 8-5 中的代码几乎与 列表 8-3 中的代码完全相同。

```
package main

import (
    "bytes"
    "fmt"
    "log"

    "github.com/google/gopacket"
    "github.com/google/gopacket/pcap"
)

var (
    iface    = "enp0s5"
    snaplen  = int32(1600)
    promisc  = false
    timeout  = pcap.BlockForever
 ❶ filter   = "tcp and dst port 21"
    devFound = false
)
 func main() {
    devices, err := pcap.FindAllDevs()
    if err != nil {
        log.Panicln(err)
    }

    for _, device := range devices {
        if device.Name == iface {
            devFound = true
        }
    }
    if !devFound {
        log.Panicf("Device named '%s' does not exist\n", iface)
    }

    handle, err := pcap.OpenLive(iface, snaplen, promisc, timeout)
    if err != nil {
        log.Panicln(err)
    }
    defer handle.Close()

    if err := handle.SetBPFFilter(filter); err != nil {
        log.Panicln(err)
    }

    source := gopacket.NewPacketSource(handle, handle.LinkType())
    for packet := range source.Packets() {
     ❷ appLayer := packet.ApplicationLayer()
        if appLayer == nil {
            continue
        }  
     ❸ payload := appLayer.Payload()
     ❹ if bytes.Contains(payload, []byte("USER")) {
            fmt.Print(string(payload))
        } else if bytes.Contains(payload, []byte("PASS")) {
            fmt.Print(string(payload))
        }  
    }
}
```

*列表 8-5：捕获 FTP 身份验证凭据 (*[/ch-8/ftp/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-8/ftp/main.go)*)*

你所做的更改仅包含大约 10 行代码。首先，你修改了 BPF 过滤器，仅捕获目标为 21 端口（常用于 FTP 流量）的流量❶。其余代码保持不变，直到你处理数据包。

要处理数据包，你首先从数据包中提取应用层，并检查它是否确实存在❷，因为应用层包含 FTP 命令和数据。你通过检查`packet.ApplicationLayer()`的响应值是否为`nil`来寻找应用层。假设数据包中存在应用层，你可以通过调用`appLayer.Payload()`来从该层提取有效负载（FTP 命令/数据）❸。（对于提取和检查其他层及数据，也有类似的方法，但你只需要应用层的有效负载。）提取了有效负载后，你接着检查该有效负载是否包含`USER`或`PASS`命令❹，这表明它是登录序列的一部分。如果包含，则将有效负载显示到屏幕上。

这是一个捕获 FTP 登录尝试的示例：

```
$ go build -o ftp && sudo ./ftp
USER someuser
PASS passw0rd
```

当然，你可以改进这段代码。在这个示例中，如果有效负载中存在`USER`或`PASS`，它们将会被显示。实际上，代码应该只搜索有效负载的开头，以消除在那些关键词出现在客户端和服务器之间传输的文件内容中，或者作为像`PASSAGE`或`ABUSER`等更长单词的一部分时所产生的误报。我们鼓励你将这些改进作为学习练习。

### 通过 SYN 洪泛保护进行端口扫描

在第二章中，你曾经演示了如何创建一个端口扫描器。通过多次迭代，你改进了代码，直到获得一个高性能的实现，能够生成准确的结果。然而，在某些情况下，那个扫描器仍然可能产生错误的结果。具体来说，当一个组织使用 SYN 洪泛保护时，通常所有端口——无论是开放、关闭还是过滤——都会产生相同的数据包交换，表明该端口是开放的。这些保护被称为 SYN *cookie*，它们防止 SYN 洪泛攻击并模糊攻击面，从而产生误报。

当目标使用 SYN Cookie 时，你如何判断一个服务是否在某个端口上监听，或者设备是否错误地显示该端口为开放状态？毕竟，在这两种情况下，TCP 三次握手都是完成的。大多数工具和扫描器（包括 Nmap）都查看这个序列（或根据你选择的扫描类型进行变体），以确定端口的状态。因此，你不能仅依赖这些工具来生成准确的结果。

然而，如果你考虑到在建立连接之后会发生什么——数据交换，可能是服务横幅的形式——你可以推断出是否有实际的服务在响应。SYN 洪水保护通常不会在初始三次握手之后交换数据包，除非有服务在监听，因此任何额外的数据包可能表明存在服务。

#### 检查 TCP 标志

为了考虑到 SYN cookies，你需要扩展你的端口扫描能力，检查三次握手后是否收到来自目标的任何额外数据包。你可以通过嗅探数据包来完成此任务，查看是否有任何数据包使用 TCP 标志值，表明有额外的合法服务通信。

*TCP 标志*表示数据包传输状态的信息。如果你查看 TCP 规范，你会发现标志存储在数据包头部的第 14 个位置的单个字节中。该字节的每一位表示一个标志值。如果该位置的位设置为 1，则标志为“开启”；如果位设置为 0，则标志为“关闭”。表 8-1 显示了根据 TCP 规范，标志在字节中的位置。

**表 8-1：** TCP 标志及其字节位置

| **位** | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **标志** | CWR | ECE | URG | ACK | PSH | RST | SYN | FIN |

一旦你知道了你关心的标志的位置，你就可以创建一个过滤器来检查它们。例如，你可以查找包含以下标志的数据包，这些标志可能表示一个正在监听的服务：

+   ACK 和 FIN

+   ACK

+   ACK 和 PSH

由于你可以通过使用`gopacket`库捕获和过滤某些数据包，你可以构建一个工具，尝试连接到远程服务，嗅探数据包，并仅显示与这些 TCP 头部通信数据包的服务。假设由于 SYN cookies，所有其他服务都是错误地“开放”的。

#### 构建 BPF 过滤器

你的 BPF 过滤器需要检查表示数据包传输的特定标志值。如果我们前面提到的标志被打开，那么标志字节将具有以下值：

+   ACK 和 FIN: 00010001 (0x11)

+   ACK: 00010000 (0x10)

+   ACK 和 PSH: 00011000 (0x18)

我们包括了二进制值的十六进制等效值，以便更清晰地理解，因为你将在过滤器中使用十六进制值。

总结来说，你需要检查 TCP 头部的第 14 个字节（基于 0 的索引为 13 偏移），仅过滤标志为 0x11、0x10 或 0x18 的数据包。以下是 BPF 过滤器的样子：

```
tcp[13] == 0x11 or tcp[13] == 0x10 or tcp[13] == 0x18
```

很好，你已经有了过滤器。

#### 编写端口扫描器

现在你将使用过滤器构建一个实用程序，该程序建立一个完整的 TCP 连接，并检查三次握手之后的包，以查看是否有其他数据包被传输，表示有实际的服务在监听。该程序如列表 8-6 所示。为了简化起见，我们选择不优化代码的效率。不过，你可以通过进行类似于我们在第二章中所做的优化，来大大改进这段代码。

```
var ( ❶
    snaplen  = int32(320)
    promisc  = true
    timeout  = pcap.BlockForever
    filter   = "tcp[13] == 0x11 or tcp[13] == 0x10 or tcp[13] == 0x18"
    devFound = false
    results  = make(map[string]int)
)

func capture(iface, target string) { ❷
    handle, err := pcap.OpenLive(iface, snaplen, promisc, timeout)
    if err != nil {
        log.Panicln(err)
    }

    defer handle.Close()

    if err := handle.SetBPFFilter(filter); err != nil {
        log.Panicln(err)
    }  

    source := gopacket.NewPacketSource(handle, handle.LinkType())
    fmt.Println("Capturing packets")
    for packet := range source.Packets() {
        networkLayer := packet.NetworkLayer() ❸
        if networkLayer == nil {
            continue
        }
        transportLayer := packet.TransportLayer()
        if transportLayer == nil {
            continue
        }

        srcHost := networkLayer.NetworkFlow().Src().String() ❹
        srcPort := transportLayer.TransportFlow().Src().String()

        if srcHost != target { ❺
            continue
        }
        results[srcPort] += 1 ❻
    }  
}

func main() {

    if len(os.Args) != 4 {
        log.Fatalln("Usage: main.go <capture_iface> <target_ip> <port1,port2,port3>")
    }  

    devices, err := pcap.FindAllDevs()
    if err != nil {
        log.Panicln(err)
    }  

    iface := os.Args[1]
    for _, device := range devices {
        if device.Name == iface {
            devFound = true
        }
    }  
    if !devFound {
        log.Panicf("Device named '%s' does not exist\n", iface)
    }  

    ip := os.Args[2]
    go capture(iface, ip) ❼
    time.Sleep(1 * time.Second)

    ports, err := explode(os.Args[3])
    if err != nil {
        log.Panicln(err)
    }  

    for _, port := range ports { ❽
        target := fmt.Sprintf("%s:%s", ip, port)
        fmt.Println("Trying", target)
        c, err := net.DialTimeout("tcp", target, 1000*time.Millisecond) ❾
        if err != nil {
            continue
        }
        c.Close()
    }
    time.Sleep(2 * time.Second)

    for port, confidence := range results { ❿
        if confidence >= 1 {
            fmt.Printf("Port %s open (confidence: %d)\n", port, confidence)
        }
    }
}

/* Extraneous code omitted for brevity */
```

*列表 8-6：带有 SYN-flood 保护的包扫描和处理（*[/ch-8/syn-flood/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-8/syn-flood/main.go)*)*

大致来说，你的代码将维护一个数据包计数，按端口分组，表示你对该端口是否开放的信心。你将使用过滤器仅选择具有适当标志设置的数据包。匹配的数据包计数越多，你对该端口上服务是否在监听的信心就越高。

你的代码首先定义了几个变量，以便在整个过程中使用❶。这些变量包括过滤器和一个名为`results`的映射，你将用它来跟踪你对端口是否开放的信心水平。你将使用目标端口作为键，并将匹配的数据包计数作为映射值。

接下来，你定义一个名为`capture()`的函数，该函数接受你正在测试的接口名称和目标 IP❷。该函数本身以与之前示例相同的方式启动数据包捕获。然而，你必须使用不同的代码来处理每个数据包。你利用`gopacket`功能来提取数据包的网络层和传输层❸。如果这些层之一缺失，你就忽略该数据包；这是因为下一步是检查数据包的源 IP 和端口❹，如果没有传输层或网络层，你就无法获取这些信息。接下来，你确认数据包的源是否与目标的 IP 地址匹配❺。如果数据包的源和 IP 地址不匹配，则跳过进一步的处理。如果数据包的源 IP 和端口与你的目标匹配，你就会增加该端口的信心水平❻。对每个后续的数据包重复此过程。每次匹配时，你的信心水平就会增加。

在你的`main()`函数中，使用一个 goroutine 调用你的`capture()`函数❼。使用 goroutine 可以确保数据包捕获和处理逻辑并发运行而不会阻塞。同时，你的`main()`函数继续解析目标端口，逐个循环并调用`net.DialTimeout`来尝试与每个端口建立 TCP 连接❾。你的 goroutine 正在运行，积极地监视这些连接尝试，寻找表示服务正在监听的数据包。

在你尝试连接每个端口后，通过只显示那些具有 1 或更高置信度的端口（意味着至少有一个数据包与该端口的过滤器匹配）❿来处理所有结果。代码中包含了若干次调用`time.Sleep()`以确保你为设置嗅探器和处理数据包留出足够的时间。

让我们来看一下程序的一个示例运行，见清单 8-7。

```
$ go build -o syn-flood && sudo ./syn-flood enp0s5 10.1.100.100
80,443,8123,65530
Capturing packets
Trying 10.1.100.100:80
Trying 10.1.100.100:443
Trying 10.1.100.100:8123
Trying 10.1.100.100:65530
Port 80 open (confidence: 1)
Port 443 open (confidence: 1)
```

*清单 8-7：带有置信度评分的端口扫描结果*

测试成功地确定了端口 80 和 443 是开放的。它还确认了在端口 8123 和 65530 上没有服务在监听。（请注意，我们已更改示例中的 IP 地址以保护无辜者。）

你可以通过几种方式改进代码。作为学习练习，我们挑战你添加以下增强功能：

1.  从`capture()`函数中移除网络层和传输层的逻辑以及源检查。改为向 BPF 过滤器添加额外的参数，以确保你只捕获来自目标 IP 和端口的数据包。

1.  将端口扫描的顺序逻辑替换为并发替代方案，类似于我们在前几章中演示的内容。这将提高效率。

1.  与其将代码限制为单一目标 IP，不如允许用户提供 IP 列表或网络块。

### 总结

我们已经完成了对数据包捕获的讨论，主要集中在被动嗅探活动上。在下一章中，我们将重点讨论漏洞开发。
