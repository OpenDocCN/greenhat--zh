## 第十一章：实现与攻击加密技术

![Image](img/common.jpg)

讨论安全性时，如果不涉及*加密学*，那就不完整。当组织使用加密实践时，它们可以帮助保持信息和系统的完整性、机密性和真实性。作为工具开发者，你可能需要实现加密功能，或许是为了 SSL/TLS 通信、互相认证、对称密钥加密或密码哈希。然而，开发者经常以不安全的方式实现加密功能，这意味着攻击者可以利用这些弱点来破坏敏感的、具有价值的数据，如社会保障号码或信用卡号码。

本章展示了在 Go 语言中实现加密技术的各种方式，并讨论了你可以利用的常见弱点。尽管我们提供了不同加密函数和代码块的介绍信息，但我们并不打算深入探讨加密算法的细节或其数学基础。坦率地说，这远远超出了我们对加密学的兴趣（或知识）。正如我们之前所说，未经所有者明确许可，请勿在任何资源或资产上尝试本章中的内容。我们提供这些讨论是为了学习目的，而不是为了协助非法活动。

### 回顾基本的加密学概念

在我们探索 Go 中的加密技术之前，先来讨论一些基本的加密学概念。我们会简要介绍，避免让你陷入深度睡眠。

首先，加密（为了保持机密性）只是加密学的一项任务。*加密*，一般来说，是一个双向功能，你可以将数据加密并随后解密以恢复原始输入。加密数据的过程会使数据变得无意义，直到它被解密为止。

加密和解密都涉及将数据和一个附带的密钥传入加密函数。该函数输出加密数据（称为*密文*）或原始的、可读的数据（称为*明文*）。有多种算法可以实现这一过程。*对称*算法在加密和解密过程中使用相同的密钥，而*非对称*算法在加密和解密过程中使用不同的密钥。你可能会使用加密来保护传输中的数据，或者存储敏感信息，例如信用卡号码，以便稍后解密，可能是为了将来购物时的便利，或进行欺诈监控。

另一方面，*哈希*是一种单向过程，用于在数学上对数据进行混淆。你可以将敏感信息传入哈希函数，以产生固定长度的输出。当你使用强大的算法时，例如 SHA-2 系列算法，不同输入产生相同输出的概率极低。也就是说，发生*碰撞*的可能性很小。由于哈希是不可逆的，它们通常被用作替代方案，用于在数据库中存储明文密码，或者进行完整性检查，以确定数据是否已被更改。如果你需要模糊或随机化两个相同输入的输出，可以使用*盐*，它是一个随机值，用来在哈希过程中区分两个相同的输入。盐通常用于密码存储，因为它们允许多个恰好使用相同密码的用户仍然具有不同的哈希值。

加密学还提供了一种认证消息的手段。*消息认证码（MAC）*是从一个特殊的单向加密函数产生的输出。该函数会使用数据本身、一个秘密密钥和一个初始化向量，并产生一个输出，这个输出不太可能发生碰撞。消息的发送者执行该函数以生成 MAC，并将 MAC 作为消息的一部分发送出去。接收方在本地计算 MAC 并与接收到的 MAC 进行比较。如果匹配，则表示发送者拥有正确的秘密密钥（即发送者是可信的），并且消息没有被更改（即完整性得到了保持）。

好的！现在你应该对加密学有足够的了解，能够理解本章的内容。必要时，我们会讨论与特定主题相关的更多细节。让我们先来看一下 Go 的标准加密库。

### 理解标准加密库

在 Go 中实现加密的美妙之处在于，你很可能会使用的大多数加密功能都是标准库的一部分。而其他语言通常依赖于 OpenSSL 或其他第三方库，Go 的加密功能则是官方库的一部分。这使得实现加密相对简单，因为你不需要安装繁琐的依赖项，从而避免污染开发环境。这里有两个独立的库。

自包含的`crypto`包包含用于最常见的加密任务和算法的多种子包。例如，你可以使用`aes`、`des`和`rc4`子包来实现对称密钥算法；使用`dsa`和`rsa`子包来进行非对称加密；使用`md5`、`sha1`、`sha256`和`sha512`子包进行哈希。这不是一个详尽无遗的列表，还有其他子包可用于其他加密功能。

除了标准的 `crypto` 包，Go 还提供了一个官方的扩展包，其中包含了多种附加的加密功能：`golang.org/x/crypto`。该包中的功能包括额外的哈希算法、加密算法和工具。例如，该包包含了一个用于 *bcrypt 哈希* 的子包（这是一个更好的、更安全的哈希密码和敏感数据的替代方法），`acme/autocert` 用于生成合法的证书，还有用于 SSH 协议通信的 SSH 子包。

内置的 `crypto` 包和附加的 `golang.org/x/crypto` 包之间唯一的真正区别是，`crypto` 包遵循更严格的兼容性要求。此外，如果你希望使用任何 `golang.org/x/crypto` 子包，你需要先通过以下命令安装该包：

```
$ go get -u golang.org/x/crypto/bcrypt
```

要查看官方 Go 加密包中所有功能和子包的完整列表，请查阅官方文档：[*https://golang.org/pkg/crypto/*](https://golang.org/pkg/crypto/) 和 [*https://godoc.org/golang.org/x/crypto/*](https://godoc.org/golang.org/x/crypto/)。

接下来的章节将深入探讨各种加密实现。你将看到如何使用 Go 的加密功能做一些恶意操作，例如破解密码哈希、使用静态密钥解密敏感数据，以及暴力破解弱加密算法。你还将使用这些功能来创建使用 TLS 保护传输中的通信、检查数据的完整性和真实性，并执行互相认证的工具。

### 探索哈希

哈希，如前所述，是一种单向函数，用于根据可变长度的输入生成固定长度的、概率上唯一的输出。你无法通过反向操作哈希值来找回原始输入数据。哈希常用于存储那些不需要原始明文数据进行后续处理或用于跟踪数据完整性的信息。例如，存储明文密码是一个不好的做法，通常也是不必要的；你应当存储哈希值（理想情况下加盐，以确保不同值之间的随机性）。

为了展示 Go 中的哈希操作，我们来看两个例子。第一个例子试图通过离线字典攻击来破解给定的 MD5 或 SHA-512 哈希值。第二个例子展示了 bcrypt 的实现。如前所述，bcrypt 是一种用于哈希敏感数据（如密码）的更安全的算法。该算法还包含一个降低速度的特性，使得破解密码变得更加困难。

#### 破解 MD5 或 SHA-256 哈希

列表 11-1 展示了哈希破解的代码。（所有位于根目录 `/` 的代码列表都存在于提供的 GitHub 仓库 *[`github.com/blackhat-go/bhg/`](https://github.com/blackhat-go/bhg/)* 下。）由于哈希值不可直接反转，代码会通过生成常见词汇的哈希（这些词汇来自于一个单词列表），然后将生成的哈希值与手头的哈希值进行比较，从而尝试猜测哈希的明文值。如果两个哈希值匹配，你可能已经猜到了明文值。

```
❶ var md5hash = "77f62e3524cd583d698d51fa24fdff4f"
   var sha256hash =
   "95a5e1547df73abdd4781b6c9e55f3377c15d08884b11738c2727dbd887d4ced"

   func main() {
       f, err := os.Open("wordlist.txt")❷
       if err != nil {
           log.Fatalln(err)
       }  
       defer f.Close()

    ❸ scanner := bufio.NewScanner(f)
       for scanner.Scan() {
           password := scanner.Text()
           hash := fmt.Sprintf("%x", md5.Sum([]byte(password))❹)
        ❺ if hash == md5hash {
               fmt.Printf("[+] Password found (MD5): %s\n", password)
           }  

           hash = fmt.Sprintf("%x", sha256.Sum256([]byte(password))❻)
        ❼ if hash == sha256hash {
               fmt.Printf("[+] Password found (SHA-256): %s\n", password)
           }  
       }  

       if err := scanner.Err(); err != nil {
           log.Fatalln(err)
       }  
   }
```

*列表 11-1：破解 MD5 和 SHA-256 哈希值 (*[/ch-11/hashes/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-11/hashes/main.go)*)*

你从定义两个变量 ❶ 开始，它们分别保存目标哈希值。一个是 MD5 哈希，另一个是 SHA-256 哈希。假设你在后渗透过程中获得了这两个哈希值，你正试图确定哪些输入（明文密码）经过哈希算法处理后生成了它们。你通常可以通过检查哈希值的长度来确定使用的算法。当你找到一个与目标哈希匹配的哈希时，就可以确认你找到了正确的输入。

你将尝试的输入列表存在一个字典文件中，该文件是在之前创建的。或者，Google 搜索可以帮助你找到用于常用密码的字典文件。要检查 MD5 哈希值，你需要打开字典文件 ❷，并通过在文件描述符 ❸ 上创建一个 `bufio.Scanner` 来逐行读取它。每一行包含一个你想要检查的密码值。你将当前密码值传入一个名为 `md5.Sum(input []byte)` 的函数 ❹。该函数生成 MD5 哈希值作为原始字节，因此你使用 `fmt.Sprintf()` 函数并结合格式化字符串 `%x` 将其转换为十六进制字符串。毕竟，你的 `md5hash` 变量是目标哈希的十六进制字符串表示。转换后的值使你能够比较目标哈希和计算出的哈希值 ❺。如果这两个哈希匹配，程序将在标准输出显示成功消息。

你执行类似的过程来计算并比较 SHA-256 哈希值。实现方式与 MD5 代码非常相似。唯一的实际差异是 `sha256` 包包含额外的函数，用于计算各种 SHA 哈希长度。你不是调用 `sha256.Sum()`（一个不存在的函数），而是调用 `sha256.Sum256(input []byte)` ❻ 强制使用 SHA-256 算法计算哈希。就像在 MD5 示例中一样，你将原始字节转换为十六进制字符串，并比较 SHA-256 哈希值，以查看是否匹配 ❼。

#### 实现 bcrypt

下一个示例展示了如何使用 bcrypt 来加密和验证密码。与 SHA 和 MD5 不同，bcrypt 是专为密码哈希设计的，比 SHA 或 MD5 更适合应用程序设计人员使用。它默认包括盐值，并且有一个成本因子，使得算法执行更加资源密集。这个成本因子控制内部加密函数的迭代次数，从而增加了破解密码哈希所需的时间和努力。虽然密码仍然可以通过字典攻击或暴力破解攻击被破解，但成本（时间）显著增加，这使得在时间敏感的后期利用中，破解活动变得不划算。随着计算能力的提升，也可以逐步增加成本，以应对未来的破解攻击。这使得 bcrypt 对未来的破解攻击具有适应性。

列表 11-2 创建一个 bcrypt 哈希，并验证一个明文密码是否与给定的 bcrypt 哈希匹配。

```
   import (
       "log"
       "os"
    ❶ "golang.org/x/crypto/bcrypt"
   )

❷ var storedHash = "$2a$10$Zs3ZwsjV/nF.KuvSUE.5WuwtDrK6UVXcBpQrH84V8q3Opg1yNdWLu"

   func main() {
       var password string
       if len(os.Args) != 2 {
           log.Fatalln("Usage: bcrypt password")
       }  
       password = os.Args[1]

    ❸ hash, err := bcrypt.GenerateFromPassword(
           []byte(password),
           bcrypt.DefaultCost,
       )
       if err != nil {
           log.Fatalln(err)
       }  
       log.Printf("hash = %s\n", hash)

    ❹ err = bcrypt.CompareHashAndPassword([]byte(storedHash), []byte(password))
       if err != nil {
           log.Println("[!] Authentication failed")
           return
       }  
       log.Println("[+] Authentication successful")
   }
```

*列表 11-2：比较 bcrypt 哈希（*[/ch-11/bcrypt/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-11/bcrypt/main.go)*)*

对于本书中的大多数代码示例，我们省略了包的导入。在这个示例中，我们显式地包含了它们，以展示你正在使用辅助的 Go 包 `golang.org/x/crypto/bcrypt` ❶，因为 Go 内置的 `crypto` 包并不包含 bcrypt 功能。接下来，你会初始化一个变量 `storedHash` ❷，该变量保存一个预计算的、编码后的 bcrypt 哈希值。这是一个虚构的示例；我们没有将示例代码与数据库连接以获取值，而是为了演示目的选择了硬编码一个值。这个变量可以代表你在数据库行中找到的一个值，那个行存储了前端 Web 应用程序的用户身份验证信息。

接下来，你将从明文密码值生成一个 bcrypt 编码的哈希值。主函数将密码值作为命令行参数读取，并调用两个不同的 bcrypt 函数。第一个函数 `bcrypt.GenerateFromPassword()` ❸ 接受两个参数：一个字节切片，表示明文密码和一个成本值。在这个示例中，你将传递常量变量 `bcrypt.DefaultCost` 来使用该包的默认成本，该成本在撰写时为 10。该函数返回编码后的哈希值和可能产生的错误。

你调用的第二个 bcrypt 函数是 `bcrypt.CompareHashAndPassword()` ❹，它在后台为你进行哈希比较。它接受一个经过 bcrypt 编码的哈希值和一个明文密码，作为字节切片传入。该函数解析编码的哈希值，以确定成本和盐值。然后，它使用这些值和明文密码值生成一个 bcrypt 哈希。如果生成的哈希值与从编码的 `storedHash` 值中提取出的哈希值匹配，那么你就知道提供的密码与创建 `storedHash` 时使用的密码一致。

这与您使用的破解 SHA 和 MD5 密码的方法相同——将给定的密码通过哈希函数并将结果与存储的哈希值进行比较。在这里，您不是像破解 SHA 和 MD5 那样显式比较结果哈希，而是检查`bcrypt.CompareHashAndPassword()`是否返回错误。如果你看到错误，那就说明计算出的哈希值，进而计算这些哈希值的密码，不匹配。

以下是两个示例程序的运行情况，第一个展示了错误密码的输出：

```
$ go run main.go someWrongPassword
2020/08/25 08:44:01 hash = $2a$10$YSSanGl8ye/NC7GDyLBLUO5gE/ng51l9TnaB1zTChWq5g9i09v0AC
2020/08/25 08:44:01 [!] Authentication failed
```

第二个示例展示了正确密码的输出：

```
$ go run main.go someC0mpl3xP@ssw0rd
2020/08/25 08:39:29 hash = $2a$10$XfeUk.wKeEePNAfjQ1juXe8RaM/9EC1XZmqaJ8MoJB29hZRyuNxz.
2020/08/25 08:39:29 [+] Authentication successful
```

如果你细心观察，可能会注意到，在你成功认证时显示的哈希值与`storedHash`变量中你硬编码的值并不匹配。回想一下，你的代码调用了两个不同的函数。`GenerateFromPassword()`函数通过使用一个随机的盐值来生成编码后的哈希。由于盐值不同，即使是相同的密码也会生成不同的哈希值，这就是差异的原因。而`CompareHashAndPassword()`函数则使用与存储哈希相同的盐值和成本来执行哈希算法，因此生成的哈希值与`storedHash`变量中的值完全相同。

### 消息认证

现在让我们将焦点转向消息认证。在交换消息时，你需要验证数据的完整性以及远程服务的真实性，以确保数据是合法的且没有被篡改。消息在传输过程中是否被未经授权的源篡改过？这条消息是由授权的发送者发送的吗，还是被其他实体伪造的？

你可以通过使用 Go 的`crypto/hmac`包来解决这些问题，它实现了*密钥哈希消息认证码*（HMAC）标准。HMAC 是一种加密算法，允许我们检查消息是否被篡改，并验证来源的身份。它使用哈希函数并消耗一个共享的秘密密钥，只有被授权生成有效消息或数据的双方才应该拥有这个密钥。如果攻击者没有这个共享的秘密密钥，就无法合理地伪造有效的 HMAC 值。

在某些编程语言中实现 HMAC 可能有点棘手。例如，某些语言要求你手动逐字节地比较接收到的哈希值与计算出来的哈希值。如果开发人员在这个逐字节比较的过程中中途终止，可能会不小心引入时间差异；攻击者可以通过测量消息处理时间来推测预期的 HMAC 值。此外，开发人员有时会误以为 HMAC（它同时使用消息和密钥）与将秘密密钥附加到消息前的哈希是一样的。然而，HMAC 的内部功能与纯粹的哈希函数是不同的。如果没有显式使用 HMAC，开发人员就会暴露应用程序于长度扩展攻击，其中攻击者伪造消息并生成有效的 MAC。

幸运的是，对于我们这些 Go 程序员来说，`crypto/hmac`包使得以安全的方式实现 HMAC 功能变得相对容易。让我们来看一个实现例子。请注意，以下程序比典型的使用案例要简单得多，典型的使用场景可能涉及某种网络通信和消息传递。在大多数情况下，你会计算 HTTP 请求参数上的 HMAC，或者一些其他通过网络传输的消息。在示例 11-3 中，我们省略了客户端和服务器之间的通信，专注于 HMAC 功能本身。

```
var key = []byte("some random key") ❶

func checkMAC(message, recvMAC []byte) bool { ❷
    mac := hmac.New(sha256.New, key) ❸
    mac.Write(message)
    calcMAC := mac.Sum(nil)

    return hmac.Equal(calcMAC, recvMAC)❹
}

func main() {
    // In real implementations, we'd read the message and HMAC value from network source
    message := []byte("The red eagle flies at 10:00") ❺
    mac, _ := hex.DecodeString("69d2c7b6fbbfcaeb72a3172f4662601d1f16acfb46339639ac8c10c8da64631d") ❻
    if checkMAC(message, mac) { ❼
        fmt.Println("EQUAL")
    } else {
        fmt.Println("NOT EQUAL")
    }  
}
```

*示例 11-3：使用 HMAC 进行消息认证 (*[/ch-11/hmac/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-11/hmac/main.go)*)*

程序首先定义了你将用于 HMAC 加密功能的密钥 ❶。你在这里硬编码了这个值，但在实际的实现中，这个密钥应该得到适当的保护并且是随机的。它还应在端点之间共享，也就是说，消息发送方和接收方都在使用相同的密钥值。由于你在这里并没有实现完整的客户端-服务器功能，因此你可以假设该变量已经得到了适当的共享。

接下来，你定义了一个函数，`checkMAC()` ❷，该函数接受一个消息和接收到的 HMAC 作为参数。消息接收方会调用此函数检查他们收到的 MAC 值是否与他们本地计算的值匹配。首先，你调用`hmac.New()` ❸，并传入`sha256.New`，这是一个返回`hash.Hash`实例的函数，以及共享的密钥。在这种情况下，`hmac.New()`函数通过使用 SHA-256 算法和你的密钥来初始化 HMAC，并将结果赋值给名为`mac`的变量。然后，你使用这个变量像之前的哈希示例一样计算 HMAC 哈希值。这里，你依次调用`mac.Write(message)`和`mac.Sum(nil)`。结果是你本地计算的 HMAC，存储在名为`calcMAC`的变量中。

下一步是评估你本地计算出的 HMAC 值是否与接收到的 HMAC 值相等。为了安全地做到这一点，你调用`hmac.Equal(calcMAC, recvMAC)`❹。许多开发人员可能倾向于通过调用`bytes.Compare(calcMAC, recvMAC)`来比较字节切片。问题是，`bytes.Compare()`执行的是字典顺序比较，它会遍历并比较给定切片的每个元素，直到找到差异或到达切片的末尾。完成这个比较所需的时间会因`bytes.Compare()`在第一个元素、最后一个元素或中间某个地方遇到差异而有所不同。攻击者可以通过测量这一时间变化来确定预期的 HMAC 值，并伪造一个被合法处理的请求。`hmac.Equal()`函数通过以几乎恒定的可测时间比较切片来解决这个问题。无论函数在哪个位置发现差异，处理时间变化都非常微小，不会产生明显或可感知的模式。

`main()`函数模拟接收来自客户端的消息过程。如果你真正在接收消息，你需要从传输中读取并解析 HMAC 和消息值。由于这只是一个模拟，你将接收到的消息❺和接收到的 HMAC❻硬编码进来，并解码 HMAC 十六进制字符串，使其表示为`[]byte`。你使用`if`语句调用你的`checkMAC()`函数❼，并将接收到的消息和 HMAC 传递给它。正如前面所详细描述的，你的`checkMAC()`函数通过使用接收到的消息和共享的密钥计算 HMAC，并返回一个`bool`值，表示接收到的 HMAC 和计算出来的 HMAC 是否匹配。

虽然 HMAC 提供了真实性和完整性保证，但它并不能确保保密性。你无法完全确认消息本身是否被未经授权的资源看到。下一部分将通过探索和实现各种加密类型来解决这个问题。

### 加密数据

加密可能是最为人知的加密概念。毕竟，隐私和数据保护因高调的数据泄露事件而受到广泛关注，这些事件通常是因为组织将用户密码和其他敏感数据以未加密的格式存储所导致的。即使没有媒体的关注，加密也应该引起黑客和开发人员的兴趣。毕竟，理解基本的过程和实现可能是决定数据泄露和攻击杀链中断之间的关键。以下部分介绍了不同形式的加密，包括每种形式的有用应用和使用案例。

#### 对称密钥加密

你进入加密的旅程将从其最简单的形式——*对称密钥加密*开始。在这种形式中，加密和解密功能使用相同的密钥。Go 使对称加密变得相当简单，因为它在默认或扩展包中支持大多数常见的算法。

为了简洁起见，我们将对对称密钥加密的讨论限制为一个实际的例子。假设你已经突破了一个组织的防线，进行了必要的权限提升、横向移动和网络侦察，成功访问了一个电子商务 Web 服务器和后台数据库。数据库中包含财务交易数据；然而，这些交易中使用的信用卡号显然是加密过的。你检查了 Web 服务器上的应用程序源代码，并确定该组织正在使用高级加密标准（AES）算法。AES 支持多种操作模式，每种模式都有稍微不同的考虑和实现细节。这些模式不可互换；解密时使用的模式必须与加密时使用的模式完全相同。

在这个场景中，假设你已经确定应用程序正在使用 AES 的密码块链接（CBC）模式。那么，接下来我们来编写一个函数，解密这些信用卡信息（示例 11-4）。假设对称密钥已在应用程序中硬编码或在配置文件中静态设置。在处理这个示例时，请记住，你可能需要为其他算法或密码稍作调整，但这是一个很好的起点。

```
func unpad(buf []byte) []byte { ❶
    // Assume valid length and padding. Should add checks
    padding := int(buf[len(buf)-1])
    return buf[:len(buf)-padding]
}

func decrypt(ciphertext, key []byte) ([]byte, error) { ❷
    var (
        plaintext []byte
        iv        []byte
        block     cipher.Block
        mode      cipher.BlockMode
 err       error
    )

    if len(ciphertext) < aes.BlockSize { ❸
        return nil, errors.New("Invalid ciphertext length: too short")
    }

    if len(ciphertext)%aes.BlockSize != 0 { ❹
        return nil, errors.New("Invalid ciphertext length: not a multiple of blocksize")
    }

    iv = ciphertext[:aes.BlockSize] ❺
    ciphertext = ciphertext[aes.BlockSize:]

    if block, err = aes.NewCipher(key); err != nil { ❻
        return nil, err
    }

    mode = cipher.NewCBCDecrypter(block, iv) ❼
    plaintext = make([]byte, len(ciphertext))
    mode.CryptBlocks(plaintext, ciphertext) ❽
    plaintext = unpad(plaintext) ❾

    return plaintext, nil
}
```

*示例 11-4：AES 填充和解密 (*[/ch-11/aes/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-11/aes/main.go)*)*

该代码定义了两个函数：`unpad()` 和 `decrypt()`。`unpad()` 函数 ❶ 是一个实用函数，用来处理解密后移除填充数据的工作。这是一个必要的步骤，但超出了本讨论的范围。你可以研究一下公共密钥加密标准（PKCS）#7 填充，以获取更多信息。这是与 AES 相关的主题，因为它用于确保我们的数据具有正确的块对齐。对于这个示例，你只需要知道稍后你会用到这个函数来清理数据。该函数本身假设了一些事实，实际上你应该在真实场景中显式验证这些假设。具体来说，你应该确认填充字节的值是有效的，切片偏移量是有效的，以及结果的长度是合适的。

最有趣的逻辑存在于`decrypt()`函数❷中，该函数接受两个字节切片：需要解密的密文和你将用来解密的对称密钥。该函数执行一些验证，确认密文至少与块大小相同❸。这是必要的步骤，因为 CBC 模式加密使用初始化向量（IV）来增加随机性。这个 IV，像密码哈希中的盐值一样，并不需要保持机密。IV 的长度与单个 AES 块相同，在加密时它会被加到密文前面。如果密文的长度小于预期的块大小，你就知道要么密文存在问题，要么 IV 丢失。你还会检查密文长度是否是 AES 块大小的倍数❹。如果不是，解密将会失败，因为 CBC 模式期望密文长度是块大小的倍数。

一旦完成了验证检查，你就可以继续解密密文。正如之前提到的，IV 是加到密文前面的，因此你做的第一件事是从密文中提取 IV❺。你使用`aes.BlockSize`常量来获取 IV，然后通过`ciphertext = [aes.BlockSize:]`重新定义`ciphertext`变量，得到密文的剩余部分。现在你已将加密数据与 IV 分离开来。

接下来，你调用`aes.NewCipher()`，并传入对称密钥值❻。这会初始化 AES 块模式加密器，并将其赋值给名为`block`的变量。然后，你通过调用`cipher.NewCBCDecryptor(block, iv)`❼来指示你的 AES 加密器以 CBC 模式工作。你将结果赋值给名为`mode`的变量。（`crypto/cipher`包包含了其他 AES 模式的初始化函数，但这里你只使用 CBC 解密。）接着，你调用`mode.CryptBlocks(plaintext, ciphertext)`来解密`ciphertext`❽的内容，并将结果存储在`plaintext`字节切片中。最后，你❾通过调用`unpad()`工具函数来移除 PKCS #7 填充。你返回结果。如果一切顺利，这应该是信用卡号的明文值。

程序的示例运行会产生预期的结果：

```
$ go run main.go
key        = aca2d6b47cb5c04beafc3e483b296b20d07c32db16029a52808fde98786646c8
ciphertext = 7ff4a8272d6b60f1e7cfc5d8f5bcd047395e31e5fc83d062716082010f637c8f21150eabace62
--snip--
plaintext  = 4321123456789090
```

注意，在这段示例代码中，你并没有定义`main()`函数。为什么呢？因为在不熟悉的环境中解密数据存在许多潜在的细微差别和变数。密文和密钥值是编码过的还是原始二进制？如果它们是编码的，它们是十六进制字符串还是 Base64？数据是本地可访问的，还是需要从数据源中提取，或与硬件安全模块进行交互？重点是，解密通常不是一个简单的复制粘贴的过程，往往需要一定的算法、模式、数据库交互和数据编码的理解。因此，我们选择引导你找到答案，预期你最终会在合适的时候自己解决问题。

了解一些对称密钥加密的基本知识可以让你的渗透测试更加成功。例如，根据我们的经验，在盗取客户源代码仓库时，我们发现人们经常使用 AES 加密算法，无论是 CBC 模式还是电子密码本（ECB）模式。ECB 模式存在一些固有的弱点，而如果实现不当，CBC 模式也不会更好。加密技术可能难以理解，因此开发人员常常假设所有的加密算法和模式都是同样有效的，忽视了它们的细微差别。尽管我们不认为自己是加密专家，但我们知道足够的知识，可以在 Go 语言中安全地实现加密，并利用他人不完善的实现。

尽管对称密钥加密比非对称加密更快，但它存在固有的密钥管理挑战。毕竟，要使用它，你必须将相同的密钥分发给任何执行加密或解密操作的系统或应用程序。你必须安全地分发密钥，通常需要遵循严格的流程和审计要求。而且，单纯依赖对称密钥加密会阻止任意客户端，例如，建立与其他节点的加密通信。没有好的方法来协商秘密密钥，也没有很多常见算法和模式的身份验证或完整性保证。¹ 这意味着，任何人，无论是授权者还是恶意者，只要获得了秘密密钥，就可以使用它。

这就是非对称加密可能派上用场的地方。

#### 非对称加密

与对称密钥加密相关的许多问题都可以通过*非对称*（或*公钥*）*密码学*来解决，它使用两个独立但数学相关的密钥。一个是公开的，另一个是私密的。用私钥加密的数据只能用公钥解密，用公钥加密的数据只能用私钥解密。如果私钥得到了适当保护并保持私密，那么用公钥加密的数据依然是保密的，因为你需要严格保护的私钥才能解密它。不仅如此，你还可以用私钥来验证用户身份。比如，用户可以用私钥来签名消息，公众则可以使用公钥来解密这些消息。

那么，你可能会问：“有什么问题？如果公钥密码学提供了所有这些保证，为什么我们还需要对称密钥密码学？”好问题！公钥加密的问题在于它的速度；它比对称加密慢得多。为了兼顾两者的优点（并避免最坏的情况），你会发现很多组织采用混合方法：他们会使用非对称加密进行初始的通信协商，建立一个加密通道，通过这个通道他们创建并交换一个对称密钥（通常称为*会话密钥*）。由于会话密钥相对较小，使用公钥加密这一过程所需的开销很小。客户端和服务器随后都拥有会话密钥的副本，用它们来加快后续的通信。

让我们来看几个公钥加密的常见使用案例。具体来说，我们将讨论加密、签名验证和相互认证。

##### 加密和签名验证

在这个第一个示例中，你将使用公钥加密来加密和解密一条消息。你还将创建逻辑来签名消息并验证该签名。为了简化，你将把所有这些逻辑包含在一个单一的`main()`函数中。这样做是为了向你展示核心功能和逻辑，以便你能够实现它。在实际场景中，这个过程会稍微复杂一些，因为你可能需要有两个远程节点互相通信。这些节点必须交换公钥。幸运的是，这个交换过程不需要和交换对称密钥一样的安全保证。回想一下，任何用公钥加密的数据只能通过相关的私钥解密。所以，即使你执行中间人攻击来拦截公钥交换和未来的通信，你也无法解密任何用相同公钥加密的数据。只有私钥才能解密它。

让我们来看一下在清单 11-5 中展示的实现。我们将在回顾示例时详细阐述逻辑和加密功能。

```
func main() {
    var (
        err                                              error
        privateKey                                       *rsa.PrivateKey
        publicKey                                        *rsa.PublicKey
        message, plaintext, ciphertext, signature, label []byte
    )  

    if privateKey, err = rsa.GenerateKey(rand.Reader, 2048)❶; err != nil {
        log.Fatalln(err)
    }  
    publicKey = &privateKey.PublicKey ❷

    label = []byte("")
    message = []byte("Some super secret message, maybe a session key even")
    ciphertext, err = rsa.EncryptOAEP(sha256.New(), rand.Reader, publicKey, message, label) ❸
    if err != nil {
        log.Fatalln(err)
    }
    fmt.Printf("Ciphertext: %x\n", ciphertext)

    plaintext, err = rsa.DecryptOAEP(sha256.New(), rand.Reader, privateKey, ciphertext, label) ❹
    if err != nil {
        log.Fatalln(err)
    }  
    fmt.Printf("Plaintext: %s\n", plaintext)

    h := sha256.New()
    h.Write(message)
    signature, err = rsa.SignPSS(rand.Reader, privateKey, crypto.SHA256, h.Sum(nil), nil) ❺
    if err != nil {
        log.Fatalln(err)
    }  
    fmt.Printf("Signature: %x\n", signature)

    err = rsa.VerifyPSS(publicKey, crypto.SHA256, h.Sum(nil), signature, nil)❻
    if err != nil {
        log.Fatalln(err)
    }  
    fmt.Println("Signature verified")
}
```

*清单 11-5：非对称加密或公钥加密 (*[/ch-11/public-key/main.go/](https://github.com/blackhat-go/bhg/blob/master/ch-11/public-key/main.go)*)*

该程序演示了两个独立但相关的公钥加密功能：加密/解密和消息签名。你首先通过调用`rsa.GenerateKey()`函数 ❶生成公私钥对。你为该函数提供一个随机读取器和一个密钥长度作为输入参数。假设随机读取器和密钥长度足以生成密钥，结果将是一个`*rsa.PrivateKey`实例，该实例包含一个字段，其值为公钥。现在你有了一个有效的密钥对。为了方便起见，你将公钥分配给自己的变量 ❷。

该程序每次运行时都会生成这个密钥对。在大多数情况下，比如 SSH 通信，你只需生成一次密钥对，然后将密钥保存到磁盘上。私钥将被保密存储，而公钥将分发到各个端点。我们在这里跳过了密钥分发、保护和管理，专注于加密功能。

现在你已经创建了密钥，可以开始使用它们进行加密。你通过调用`rsa.EncryptOAEP()` ❸来实现加密，该函数接受一个哈希函数、用于填充和随机化的读取器、公钥、你希望加密的消息以及一个可选的标签。该函数返回一个错误（如果输入导致算法失败）和加密后的密文。然后，你可以将相同的哈希函数、读取器、私钥、密文和标签传递给`rsa.DecryptOAEP()` ❹函数。该函数使用你的私钥解密密文，并返回明文结果。

请注意，你正在使用公钥加密消息。这确保了只有持有私钥的人才能解密数据。接下来，你通过调用`rsa.SignPSS()` ❺来创建数字签名。你再次传递一个随机读取器、私钥、你使用的哈希函数、消息的哈希值，以及一个表示附加选项的`nil`值。该函数返回任何错误以及生成的签名值。就像人类的 DNA 或指纹一样，这个签名唯一地标识了签名者的身份（即私钥）。任何持有公钥的人都可以验证签名，不仅可以确定签名的真实性，还可以验证消息的完整性。要验证签名，你将公钥、哈希函数、哈希值、签名和附加选项传递给`rsa.VerifyPSS()` ❻。请注意，在这种情况下，你传递的是公钥，而不是私钥。希望验证签名的端点无法访问私钥，如果输入错误的密钥值，验证也将失败。`rsa.VerifyPSS()`函数在签名有效时返回`nil`，在签名无效时返回错误。

以下是程序的一个示例运行。它按预期行为操作，使用公钥加密消息，使用私钥解密，并验证签名：

```
$ go run main.go
Ciphertext: a9da77a0610bc2e5329bc324361b480ba042e09ef58e4d8eb106c8fc0b5
--snip--
Plaintext: Some super secret message, maybe a session key even
Signature: 68941bf95bbc12edc12be369f3fd0463497a1220d9a6ab741cf9223c6793
--snip--
Signature verified
```

接下来，让我们看看公钥加密的另一种应用：互相认证。

##### 互相认证

*互相认证*是客户端和服务器互相认证的过程。它们使用公钥加密技术；客户端和服务器都生成公/私钥对，交换公钥，并使用公钥来验证对方的真实性和身份。为了实现这一过程，客户端和服务器都必须做一些准备工作，设置授权，并明确指定用于验证对方的公钥值。这个过程的缺点是需要为每个节点创建唯一的密钥对，并确保服务器和客户端节点拥有正确的数据，才能顺利进行。

首先，您将处理创建密钥对的管理任务。您将以自签名的 PEM 编码证书形式存储公钥。我们使用 `openssl` 工具来创建这些文件。在您的服务器上，您将通过输入以下命令来创建服务器的私钥和证书：

```
$ openssl req -nodes -x509 -newkey rsa:4096 -keyout serverKey.pem -out serverCrt.pem -days 365
```

`openssl` 命令将提示您输入各种信息，您可以为本示例提供任意值。该命令会创建两个文件：*serverKey.pem* 和 *serverCrt.pem*。文件 *serverKey.pem* 包含您的私钥，您应当保护它。*serverCrt.pem* 文件包含服务器的公钥，您将把它分发给每个连接的客户端。

对于每个连接的客户端，您将运行类似于前面命令的操作：

```
$ openssl req -nodes -x509 -newkey rsa:4096 -keyout clientKey.pem -out clientCrt.pem -days 365
```

此命令还会生成两个文件：*clientKey.pem* 和 *clientCrt.pem*。与服务器输出类似，您应当保护客户端的私钥。*clientCrt.pem* 证书文件将被传输到您的服务器，并由您的程序加载。这将允许您配置并将客户端标识为授权的端点。对于每个额外的客户端，您必须创建、传输并配置证书，以便服务器能够识别并明确授权它们。

在 示例 11-6 中，您设置了一个 HTTPS 服务器，要求客户端提供合法的授权证书。

```
func helloHandler(w http.ResponseWriter, r *http.Request) { ❶
    fmt.Printf("Hello: %s\n", r.TLS.PeerCertificates[0].Subject.CommonName) ❷
    fmt.Fprint(w, "Authentication successful")
}

func main() {
    var (
        err        error
        clientCert []byte
        pool       *x509.CertPool
        tlsConf    *tls.Config
        server     *http.Server
    )  

    http.HandleFunc("/hello", helloHandler)

    if clientCert, err = ioutil.ReadFile("../client/clientCrt.pem")❸; err != nil {
        log.Fatalln(err)
    }  
    pool = x509.NewCertPool()
    pool.AppendCertsFromPEM(clientCert) ❹

    tlsConf = &tls.Config{ ❺
        ClientCAs:  pool,
        ClientAuth: tls.RequireAndVerifyClientCert,
    }  
    tlsConf.BuildNameToCertificate() ❻

    server = &http.Server{
        Addr:      ":9443",
        TLSConfig: tlsConf, ❼
    }  
    log.Fatalln(server.ListenAndServeTLS("serverCrt.pem", "serverKey.pem")❽)
}
```

*示例 11-6：设置互相认证服务器 (*[/ch-11/mutual-auth/cmd/server/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-11/mutual-auth/cmd/server/main.go)*)*

在`main()`函数外，程序定义了一个`helloHandler()`函数❶。正如我们在第三章和第四章中讨论过的那样，处理程序函数接受一个`http.ResponseWriter`实例和`http.Request`本身。这个处理程序相当简单，它记录了接收到的客户端证书的常用名称❷。常用名称是通过检查`http.Request`的`TLS`字段，并深入到证书的`PeerCertificates`数据中来获取的。该处理程序还会向客户端发送一条消息，指示身份验证已成功。

但是，你如何定义哪些客户端是授权的，如何对它们进行身份验证呢？这个过程相当简单。你首先从客户端之前创建的 PEM 文件中读取客户端证书❸。因为可能有多个授权的客户端证书，所以你需要创建一个证书池，并调用`pool.AppendCertsFromPEM(clientCert)`将客户端证书添加到你的池中❹。你需要对每个额外的客户端执行此步骤，以便对它们进行身份验证。

接下来，你需要创建你的 TLS 配置。你显式地将`ClientCAs`字段设置为你的`pool`，并将`ClientAuth`配置为`tls.RequireAndVerifyClientCert`❺。这个配置定义了你授权客户端的池，并要求客户端在允许继续之前正确地进行身份验证。你调用`tlsConf.BuildNameToCertificate()`，确保客户端的常用名称和主题备用名称——证书生成的域名——能够正确映射到它们指定的证书❻。你定义了你的 HTTP 服务器，显式地设置了自定义配置❼，并通过调用`server.ListenAndServeTLS()`启动服务器，传入你之前创建的服务器证书和私钥文件❽。请注意，你在服务器代码中不会使用客户端的私钥文件。正如我们之前所说，私钥保持私密；你的服务器只会使用客户端的公钥来识别和授权客户端。这就是公钥加密的精妙之处。

你可以使用`curl`验证你的服务器。如果你生成并提供一个虚假的、未经授权的客户端证书和密钥，你会收到一条详细的消息，告诉你这一点：

```
$ curl -ik -X GET --cert badCrt.pem --key badKey.pem \
  https://server.blackhat-go.local:9443/hello
curl: (35) gnutls_handshake() failed: Certificate is bad
```

你还会在服务器上收到一条更为详细的消息，内容可能如下：

```
http: TLS handshake error from 127.0.0.1:61682: remote error: tls: unknown certificate authority
```

反过来，如果你提供了有效的证书和与服务器池中配置的证书匹配的密钥，你将享受成功身份验证时那短暂的荣耀时刻：

```
$ curl -ik -X GET --cert clientCrt.pem --key clientKey.pem \
  https://server.blackhat-go.local:9443/hello
HTTP/1.1 200 OK
Date: Fri, 09 Oct 2020 16:55:52 GMT
Content-Length: 25
Content-Type: text/plain; charset=utf-8

Authentication successful
```

这条消息告诉你服务器工作正常。

现在，让我们看一下一个客户端（示例 11-7）。你可以在与服务器相同的系统上运行客户端，也可以在不同的系统上运行。如果是在不同的系统上，你需要将*clientCrt.pem*传输到服务器，并将*serverCrt.pem*传输到客户端。

```
func main() {
    var (
        err              error
        cert             tls.Certificate
        serverCert, body []byte
        pool             *x509.CertPool
        tlsConf          *tls.Config
 transport        *http.Transport
        client           *http.Client
        resp             *http.Response
    )  

    if cert, err = tls.LoadX509KeyPair("clientCrt.pem", "clientKey.pem"); err != nil { ❶
        log.Fatalln(err)
    }  

    if serverCert, err = ioutil.ReadFile("../server/serverCrt.pem"); err != nil { ❷
        log.Fatalln(err)
    }  

    pool = x509.NewCertPool()
    pool.AppendCertsFromPEM(serverCert) ❸

    tlsConf = &tls.Config{ ❹
        Certificates: []tls.Certificate{cert},
        RootCAs:      pool,
    }  
    tlsConf.BuildNameToCertificate()❺

    transport = &http.Transport{ ❻
        TLSClientConfig: tlsConf,
    }  
    client = &http.Client{ ❼
        Transport: transport,
    }  

    if resp, err = client.Get("https://server.blackhat-go.local:9443/hello"); err != nil { ❽
        log.Fatalln(err)
    }  
    if body, err = ioutil.ReadAll(resp.Body); err != nil { ❾
        log.Fatalln(err)
    }  
    defer resp.Body.Close()

    fmt.Printf("Success: %s\n", body)
}
```

*清单 11-7：相互认证客户端（*[/ch-11/mutual-auth/cmd/client/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-11/mutual-auth/cmd/client/main.go)）*

证书的准备和配置与你在服务器代码中所做的类似：创建证书池并准备主题和通用名称。由于你不会将客户端证书和密钥用作服务器，因此你需要调用 `tls.``LoadX509KeyPair("clientCrt.pem",` `"clientKey.pem")` 来加载它们，以供后续使用 ❶。你还需要读取服务器证书，并将其添加到你希望允许的证书池中 ❷。然后，你使用该证书池和客户端证书 ❸ 来构建 TLS 配置 ❹，并调用 `tlsConf.BuildNameToCertificate()` 来将域名与其各自的证书绑定 ❺。

由于你正在创建一个 HTTP 客户端，你必须定义一个传输层 ❻，并将其与 TLS 配置关联。然后，你可以使用该传输实例来创建一个 `http.Client` 结构体 ❼。正如我们在第三章和第四章中讨论的那样，你可以使用这个客户端通过 `client.Get("https://server.blackhat-go.local:9443/hello")` ❽ 发起一个 HTTP GET 请求。

到这一点时，所有的“魔法”都发生在幕后。进行的是相互认证——客户端和服务器相互认证。如果认证失败，程序将返回一个错误并退出。否则，你将读取 HTTP 响应体并将其显示到标准输出 ❾。运行你的客户端代码会产生预期的结果，具体来说，就是没有抛出错误且认证成功：

```
$ go run main.go
Success: Authentication successful
```

下面是你的服务器输出。请记住，你已经配置服务器将问候消息记录到标准输出中。该消息包含连接客户端的通用名称，从证书中提取：

```
$ go run main.go
Hello: client.blackhat-go.local
```

现在，你已经有了一个功能齐全的相互认证示例。为了进一步增强你的理解，我们鼓励你调整之前的示例，使其可以在 TCP 套接字上运行。

在下一节中，你将把精力集中在一个更阴险的目的上：暴力破解 RC2 加密算法的对称密钥。

### 暴力破解 RC2

*RC2* 是由 Ron Rivest 在 1987 年创建的对称密钥分组密码。受到政府推荐的启发，设计者使用了 40 位加密密钥，这使得该密码足够弱，以至于美国政府可以暴力破解密钥并解密通信。它为大多数通信提供了足够的保密性，但也让政府能够窥探与外国实体的谈话。例如，在 1980 年代，暴力破解密钥需要大量计算能力，只有资金雄厚的国家或专业机构才能在合理的时间内解密。然而，时至今日，普通家用电脑可以在几天或几周内暴力破解 40 位密钥。

那么，怎么说呢，咱们来暴力破解一个 40 位密钥吧。

#### 开始使用

在我们深入代码之前，先设定一下背景。首先，标准的和扩展的 Go 加密库都没有供公众使用的 RC2 包。然而，它有一个内部的 Go 包。你不能直接在外部程序中导入内部包，因此你必须找到另一种方法来使用它。

其次，为了简化操作，你将做一些通常不愿意做的假设。具体来说，你将假设明文数据的长度是 RC2 块大小（8 字节）的倍数，以避免用像处理 PKCS #5 填充这样的管理任务来干扰你的逻辑。处理填充的方式类似于你在本章之前使用 AES 时做的（见列表 11-4），但你需要更加小心地验证内容，以确保你将要处理的数据的完整性。你还将假设你的密文是一个加密的信用卡号码。你将通过验证得到的明文数据来检查潜在的密钥。在这种情况下，验证数据包括确保文本是数字，然后进行*Luhn 校验*，这是一种验证信用卡号码和其他敏感数据的方法。

接下来，你将假设你能够通过窃取文件系统数据或源代码来确定——数据使用 40 位密钥在 ECB 模式下加密，并且没有初始化向量。RC2 支持可变长度的密钥，并且作为一个分组密码，它可以在不同模式下工作。在 ECB 模式下，这是最简单的模式，数据块独立于其他数据块进行加密。这将使你的逻辑变得更加直观。最后，虽然你可以在非并发实现中破解密钥，但如果你选择并发实现，它将具有更好的性能。与其逐步构建，先展示一个非并发版本再展示一个并发版本，我们直接从并发构建开始。

现在，你将安装几个先决条件。首先，从[*https://github.com/golang/crypto/blob/master/pkcs12/internal/rc2/rc2.go*](https://github.com/golang/crypto/blob/master/pkcs12/internal/rc2/rc2.go)获取官方的 RC2 Go 实现。你需要将其安装到本地工作空间中，以便你可以将其导入到你的暴力破解器中。如前所述，该包是一个内部包，意味着默认情况下，外部包无法导入和使用它。这有点黑客手段，但它可以防止你使用第三方实现，或者——哆嗦——自己编写 RC2 密码代码。如果你将其复制到工作空间中，未导出的函数和类型将成为你开发包的一部分，从而使它们可以访问。

让我们还安装一个你将用来执行 Luhn 校验的包：

```
$ go get github.com/joeljunstrom/go-luhn
```

Luhn 检查计算信用卡号码或其他身份识别数据的校验和，以确定它们是否有效。你将使用现有的包来完成这一操作。它有着完善的文档，并且可以避免你重新发明轮子。

现在你可以编写代码了。你需要遍历整个密钥空间的每一种组合（40 位），用每个密钥解密密文，然后通过确保解密结果仅包含数字字符并通过 Luhn 检查来验证结果。你将使用生产者/消费者模型来管理工作——生产者将密钥推送到一个通道，消费者从通道读取密钥并相应执行。工作本身将是一个单一的密钥值。当你找到一个能够生成经过验证的明文的密钥（表明你找到了一个信用卡号码）时，你将通知所有 goroutine 停止工作。

这个问题的一个有趣挑战是如何遍历密钥空间。在我们的解决方案中，你通过一个 `for` 循环来遍历表示为 `uint64` 值的密钥空间。挑战在于，正如你所看到的，`uint64` 在内存中占用 64 位空间。因此，将一个 `uint64` 转换为一个 40 位（5 字节）`[]byte` 的 RC2 密钥时，必须剪切掉 24 位（3 字节）的无用数据。希望在你看过代码后，这个过程会变得更加清晰。我们会慢慢来，逐步解析程序的各个部分，逐一进行分析。Listing 11-8 开始了这段程序。

```
   import (
       "crypto/cipher"
       "encoding/binary"
       "encoding/hex"
       "fmt"
       "log"
       "regexp"
       "sync"

     ❶ luhn "github.com/joeljunstrom/go-luhn"

     ❷ "github.com/bhg/ch-11/rc2-brute/rc2"
   )

❸ var numeric = regexp.MustCompile(`^\d{8}$`)

❹ type CryptoData struct {
       block cipher.Block
       key   []byte
   }
```

*Listing 11-8: 导入 RC2 暴力破解类型 (*[/ch-11/rc2-brute/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-11/rc2-brute/main.go)*)*

我们在这里包含了 `import` 语句，以引起注意第三方 `go-luhn` 包 ❶ 的引入，以及你从内部 Go 仓库克隆的 `rc2` 包 ❷ 的引入。你还编译了一个正则表达式 ❸，用来检查结果明文块是否为 8 字节的数字数据。

请注意，你检查的是 8 字节的数据，而不是 16 字节——后者是信用卡号码的长度。你检查 8 字节是因为它是 RC2 块的长度。你将逐块解密密文，因此可以先检查你解密的第一个块，看看它是否是数字。如果这个 8 字节的块不是全数字，你可以自信地认为它不是信用卡号码，并跳过对第二块密文的解密。这个小小的性能改进将显著减少执行数百万次所需的时间。

最后，你定义了一个名为 `CryptoData` ❹ 的类型，用来存储你的密钥和一个 `cipher.Block`。你将使用这个 `struct` 来定义工作单元，生产者会创建它，消费者会对其执行操作。

#### 生成工作

我们来看一下生产者函数（Listing 11-9）。你将此函数放在前面代码列表中的类型定义后面。

```
❶ func generate(start, stop uint64, out chan <- *CryptoData,\
   done <- chan struct{}, wg *sync.WaitGroup) {
    ❷ wg.Add(1)
    ❸ go func() {
        ❹ defer wg.Done()
           var (
               block cipher.Block
               err   error
               key   []byte
               data  *CryptoData
           )
        ❺ for i := start; i <= stop; i++ {
               key = make([]byte, 8)
            ❻ select {
            ❼ case <- done:
                   return
            ❽ default:
                ❾ binary.BigEndian.PutUint64(key, i)
                   if block, err = rc2.New(key[3:], 40); err != nil {
                       log.Fatalln(err)
                   }
                   data = &CryptoData{
                       block: block,
                       key:   key[3:],
                   }
                ❿ out <- data
               }
           }
       }()

       return
   }
```

*Listing 11-9: RC2 生产者函数 (*[/ch-11/rc2-brute/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-11/rc2-brute/main.go)*)*

你的生产者函数命名为 `generate()` ❶。它接受两个 `uint64` 变量，用于定义生产者将创建工作的密钥空间段（基本上是它们将生成密钥的范围）。这允许你将密钥空间拆分，将其部分分配给每个生产者。

该函数还接受两个通道：一个用于推送工作给消费者的 `*CryptData` 写通道和一个通用的 `struct` 通道，用于接收来自消费者的信号。这个第二个通道是必要的，例如，消费者识别出正确的密钥时，可以显式地通知生产者停止生产。如果问题已经解决，就没有必要再创造更多的工作了。最后，你的函数接受一个 `WaitGroup`，用于跟踪和同步生产者的执行。对于每个并发运行的生产者，你执行 `wg.Add(1)` ❷ 来告诉 `WaitGroup` 你已经启动了一个新的生产者。

你在一个 goroutine ❸ 中填充你的工作通道，包括调用 `defer wg.Done()` ❹ 来通知你的 `WaitGroup` 当 goroutine 退出时。这将防止后续在尝试从 `main()` 函数继续执行时发生死锁。你使用 `start()` 和 `stop()` 值，通过 `for` 循环 ❺ 遍历密钥空间的一个子集。循环的每次迭代都会递增 `i` 变量，直到你到达结束偏移量。

正如我们之前提到的，你的密钥空间是 40 位，但 `i` 是 64 位。这个大小差异非常重要。你没有一个原生的 Go 类型是 40 位的。你只有 32 位或 64 位类型。由于 32 位太小，无法容纳 40 位的值，因此你需要使用 64 位类型，并且稍后处理额外的 24 位。也许你可以通过使用 `[]byte` 而不是 `uint64` 来迭代整个密钥空间，从而避免整个挑战。但这样做可能需要一些奇怪的位运算，可能会让例子变得过于复杂。所以，你将处理长度上的细微差别。

在你的循环中，你包含了一个 `select` 语句 ❻，乍一看可能显得有些愚蠢，因为它是在通道数据上操作，且不符合典型的语法。你使用它来检查 `done` 通道是否已通过 `case <- done` ❼ 关闭。如果通道已关闭，你发出 `return` 语句以跳出你的 goroutine。当 `done` 通道未关闭时，你使用 `default` 情况 ❽ 来创建定义工作的加密实例。具体而言，你调用 `binary.BigEndian.PutUint64(key, i)` ❾ 将你的 `uint64` 值（当前密钥）写入名为 `key` 的 `[]byte` 中。

尽管我们之前没有明确指出，但你将`key`初始化为一个 8 字节的切片。那么，为什么你要将切片定义为 8 字节，而实际上只处理 5 字节的密钥呢？嗯，因为`binary.BigEndian.PutUint64`接受的是一个`uint64`值，因此它需要一个 8 字节长度的目标切片，否则会抛出超出索引范围的错误。它无法将 8 字节的值存入 5 字节的切片。所以，你给它传递了一个 8 字节的切片。请注意，在其余的代码中，你只使用`key`切片的最后 5 个字节；即使前 3 个字节会是零，它们如果被包含进来，也会破坏我们加密函数的严格性。因此，你通过调用`rc2.New(key[3:], 40)`来初始化你的加密器；这样做丢弃了前 3 个不相关的字节，并且传递了密钥的长度（单位为比特）：40。你使用生成的`cipher.Block`实例和相关的密钥字节来创建一个`CryptoData`对象，并将其写入`out`工作通道 ❿。

这就是生产者代码的全部内容。注意，在这一部分，你仅仅是在初始化所需的相关关键信息。函数中并没有实际尝试解密密文。你将在消费者函数中执行这项工作。

#### 执行工作和解密数据

现在让我们回顾一下消费者函数（示例 11-10）。同样，你将把这个函数添加到与之前的代码相同的文件中。

```
❶ func decrypt(ciphertext []byte, in <- chan *CryptoData, \
   done chan struct{}, wg *sync.WaitGroup) {
       size := rc2.BlockSize
       plaintext := make([]byte, len(ciphertext))
    ❷ wg.Add(1)
       go func() {  
        ❸ defer wg.Done()
        ❹ for data := range in {
               select {
            ❺ case <- done:
                   return
            ❻ default:
                ❼ data.block.Decrypt(plaintext[:size], ciphertext[:size])
                ❽ if numeric.Match(plaintext[:size]) {
                    ❾ data.block.Decrypt(plaintext[size:], ciphertext[size:])
                    ❿ if luhn.Valid(string(plaintext)) && \
                       numeric.Match(plaintext[size:]) {
                           fmt.Printf("Card [%s] found using key [%x]\n", /
                           plaintext, data.key)
                           close(done)
                           return
                       }
                   }
               }
           }
       }()
   }
```

*示例 11-10：RC2 消费者函数（*[/ch-11/rc2-brute/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-11/rc2-brute/main.go)*)*

你的消费者函数名为`decrypt()` ❶，接受多个参数。它接收你想要解密的密文。同时，它还接受两个不同的通道：一个只读的`*CryptoData`通道，名为`in`，你将用它作为工作队列；另一个是名为`done`的通道，用于发送和接收显式的取消信号。最后，它还接受一个`*sync.WaitGroup`，名为`wg`，你将用它来管理消费者工作线程，和你的生产者实现方式类似。你通过调用`wg.Add(1)` ❷来告诉`WaitGroup`你正在启动一个工作线程。这样，你就能够跟踪并管理所有正在运行的消费者。

接下来，在你的 goroutine 中，你调用`defer wg.Done()` ❸，这样当 goroutine 函数结束时，你将更新`WaitGroup`的状态，将正在运行的工作线程数减一。这个`WaitGroup`机制对于在任意数量的工作线程之间同步程序执行是必要的。稍后，你将在`main()`函数中使用`WaitGroup`来等待所有`goroutines`完成。

消费者使用`for`循环❹从`in`通道反复读取`CryptoData`工作结构体。循环在通道关闭时停止。回想一下，生产者会填充这个通道。正如你很快会看到的那样，这个通道会在生产者迭代完它们的整个密钥空间子集并将相关的加密数据推送到工作通道后关闭。因此，消费者循环直到生产者完成生产。

和生产者代码一样，你在`for`循环中使用`select`语句来检查`done`通道是否已关闭❺，如果已关闭，你显式地通知消费者停止额外的工作。当一个有效的信用卡号码被识别时，工作者将关闭通道，正如我们稍后会讨论的那样。你的`default`分支❻执行加密相关的重任务。首先，它解密第一个密文块（8 字节）❼，检查结果明文是否是一个 8 字节的数字值❽。如果是，那么你就有了一个潜在的卡号，并继续解密第二个密文块❾。你通过访问从通道读取的`CryptoData`工作对象中的`cipher.Block`字段来调用这些解密函数。回想一下，生产者使用密钥空间中的唯一密钥值实例化了该结构。

最后，你使用 Luhn 算法验证整个明文，并验证第二个明文块是否是一个 8 字节的数字值❿。如果这些检查通过，你可以合理地确定你找到了一个有效的信用卡号码。你将卡号和密钥显示到`stdout`，并调用`close(done)`来通知其他 goroutine 你已经找到了目标。

#### 编写主函数

到这时，你已经有了生产者和消费者函数，两个函数都已准备好并具备并发执行的能力。现在，让我们在你的`main()`函数中将它们整合起来（Listing 11-11），它将出现在与之前的代码示例相同的源文件中。

```
func main() {
    var (
        err        error
        ciphertext []byte
    )

    if ciphertext, err = hex.DecodeString("0986f2cc1ebdc5c2e25d04a136fa1a6b"); err != nil { ❶
        log.Fatalln(err)
    }

    var prodWg, consWg sync.WaitGroup ❷
    var min, max, prods = uint64(0x0000000000), uint64(0xffffffffff), uint64(75)
    var step = (max - min) / prods

    done := make(chan struct{})
    work := make(chan *CryptoData, 100)
    if (step * prods) < max { ❸
        step += prods
    }
 var start, end = min, min + step
    log.Println("Starting producers...")
    for i := uint64(0); i < prods; i++ { ❹
        if end > max {
            end = max
        }
        generate(start, end, work, done, &prodWg) ❺
        end += step
        start += step
    }
    log.Println("Producers started!")
    log.Println("Starting consumers...")
    for i := 0; i < 30; i++ { ❻
        decrypt(ciphertext, work, done, &consWg) ❼
    }
    log.Println("Consumers started!")
    log.Println("Now we wait...")
    prodWg.Wait()❽
    close(work)
    consWg.Wait()❾
    log.Println("Brute-force complete")
}
```

*Listing 11-11: The RC2* main() *function (*[/ch-11/rc2-brute/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-11/rc2-brute/main.go)*)*

你的`main()`函数解码你的密文，密文以十六进制字符串表示❶。接下来，你创建几个变量❷。首先，你创建`WaitGroup`变量，用于跟踪生产者和消费者的 goroutine。你还定义了几个`uint64`类型的值，用于跟踪 40 位密钥空间中的最小值（0x0000000000）、密钥空间中的最大值（0xffffffffff），以及你打算启动的生产者数量，这里是`75`。你使用这些值来计算步长或范围，表示每个生产者将迭代的密钥数量，因为你的目标是将这些工作均匀地分配给所有生产者。你还创建了一个`*CryptoData`工作通道和一个`done`信号通道。你将这些传递给你的生产者和消费者函数。

由于你正在进行基本的整数运算来计算生产者的步长值，因此如果密钥空间的大小不是你将启动的生产者数量的倍数，就有可能丢失一些数据。为了解决这个问题——并避免在转换为浮点数时丧失精度，以便调用`math.Ceil()`——你会检查最大密钥值（`step * prods`）是否小于整个密钥空间的最大值（0xffffffffff）❸。如果是这样，密钥空间中就会有一些值没有被考虑到。你只需增加`step`值来弥补这一缺口。你初始化了两个变量，`start`和`end`，用于维护你可以用来划分密钥空间的起始和结束偏移量。

计算偏移量和步长的数学方法并不精确，这可能导致你的代码在搜索时越过最大允许的密钥空间。然而，你在`for`循环❹中修正了这一点，这个循环用于启动每个生产者。在循环中，你调整结束步长值`end`，以防该值超出最大允许的密钥空间值。每次迭代都会调用`generate()`❺，这是你的生产者函数，并将开始（`start`）和结束（`end`）的密钥空间偏移量传递给它，生产者将根据这些偏移量进行迭代。你还将`work`和`done`通道以及生产者的`WaitGroup`传递给它。调用函数后，你会调整`start`和`end`变量，以便处理下一个密钥空间范围，这个范围将传递给新的生产者。这就是如何将密钥空间分成更小、更易处理的部分，程序可以并行处理，而不会在 goroutines 之间产生重叠的工作。

在生产者启动之后，你使用`for`循环来创建你的工作线程❻。在这个例子中，你将创建 30 个工作线程。对于每次迭代，你调用`decrypt()`函数❼，并将密文、工作通道、完成通道以及消费者`WaitGroup`作为参数传递给它。这会启动你的并发消费者，它们开始在生产者生成工作时拉取并处理工作。

遍历整个密钥空间需要时间。如果你不正确处理，`main()`函数肯定会在你发现密钥或者耗尽密钥空间之前退出。因此，你需要确保生产者和消费者有足够的时间来遍历整个密钥空间或找到正确的密钥。这时你的`WaitGroups`就派上用场了。你调用`prodWg.Wait()`❽来阻塞`main()`，直到生产者完成它们的任务。回想一下，生产者完成任务的条件是它们要么耗尽了密钥空间，要么通过`done`通道显式取消了任务。任务完成后，你显式关闭`work`通道，以避免消费者在尝试从中读取时发生死锁。最后，你再次阻塞`main()`，调用`consWg.Wait()`❾，为`WaitGroup`中的消费者提供足够的时间来完成`work`通道中的任何剩余工作。

#### 运行程序

你已经完成了程序！如果你运行它，应该会看到以下输出：

```
$ go run main.go
2020/07/12 14:27:47 Starting producers...
2020/07/12 14:27:47 Producers started!
2020/07/12 14:27:47 Starting consumers...
2020/07/12 14:27:47 Consumers started!
2020/07/12 14:27:47 Now we wait...
2020/07/12 14:27:48 Card [4532651325506680] found using key [e612d0bbb6]
2020/07/12 14:27:48 Brute-force complete
```

程序启动生产者和消费者，然后等待它们执行。当找到一张卡片时，程序会显示明文卡片和用来解密该卡片的密钥。由于我们假设这个密钥是所有卡片的魔法密钥，所以我们提前中断了执行，并通过画一幅自画像（未展示）来庆祝我们的成功。

当然，根据密钥的不同，暴力破解在家庭电脑上可能需要相当长的时间——可能是几天甚至几周。对于前面的样本运行，我们通过缩小密钥空间来更快地找到密钥。然而，在 2016 款 MacBook Pro 上完全耗尽密钥空间大约需要七天时间。对于在笔记本电脑上运行的一个快速粗糙的解决方案来说，这还算不错。

### 概要

加密是安全从业人员的重要话题，尽管学习曲线可能比较陡峭。本章讲解了对称和非对称加密、哈希、使用 bcrypt 处理密码、消息认证、互认证以及暴力破解 RC2。接下来的章节，我们将深入探讨攻击 Microsoft Windows 的细节。
