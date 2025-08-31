# 第十一章：使用 TLS 保护通信

![](img/chapterart.png)

在五年前，揭露者爱德华·斯诺登向我们展示了我们是多么地理所当然地认为电子隐私是理所应当的，作家兼活动家科里·多克特罗就曾写道：“我们应该像对待武器级铀一样对待个人电子数据——它是危险的、持久的，一旦泄露，就无法找回。”

在 2013 年之前，大多数人通过使用明文在互联网上进行通信。社会安全号码、信用卡信息、密码、敏感电子邮件和其他可能令人尴尬的信息在互联网上传播，容易被恶意行为者拦截。大多数流行的网站默认使用 HTTP；谷歌是为数不多的支持 HTTPS 的大型科技公司之一。

今天，找一个不支持 HTTPS 的网站已经变得不寻常，尤其是现在 Let’s Encrypt 提供免费的 TLS 证书给你的域名。我们对传输中的信息的处理就像处理武器级铀一样，帮助确保我们共享信息的隐私性和完整性。我们的网络应用程序也应该如此。我们应该努力认证我们的通信，并在适当的时候使用加密，特别是当信息有可能通过不安全的网络泄露时。

到目前为止，我们只是在代码中将 TLS 当作事后的补充来使用。这部分是因为 Go 的 `net/http` 库使得它的使用相对轻松，但也因为我们没有充分探索 TLS 协议以及使它成为可能的基础设施。要编写安全的软件，你应该在开发开始之前仔细规划安全性，然后在编写代码时使用良好的安全实践。TLS 是提高软件安全性的绝佳方式，它通过保护传输中的数据来改善软件的安全态势。

本章将从程序员的角度介绍 TLS 的基础知识。你将了解客户端和服务器之间的握手过程，以及使这一过程得以实现的固有信任。然后我们将讨论即使你使用了 TLS，事情如何（以及如何）出错。最后，我们将看一些实际的例子，了解如何将 TLS 集成到你的应用程序中，包括客户端和服务器的相互认证。

## 深入了解传输层安全性

TLS 协议为客户端和服务器之间提供安全通信。它允许客户端认证服务器，并可选择性地允许服务器认证客户端。客户端使用 TLS 来加密与服务器的通信，防止第三方拦截和篡改。

TLS 使用握手过程来建立 TLS 会话的某些标准。如果客户端发起了 TLS 1.3 与服务器的握手，它将大致如下：

1.  客户端 Hello *google.com*。我希望使用 TLS 版本 1.3 与你进行通信。以下是我希望用来加密我们消息的密码套件列表，按我的偏好顺序排列。我专门为这次会话生成了一对公钥和私钥。这是我的公钥。

1.  服务器问候，客户端。TLS 1.3 版本非常适合我。根据你的密码套件列表，我决定使用**高级加密标准与 Galois/计数器模式（AES-GCM）**密码套件。我也为这次会话创建了新的密钥对。这里是我的公钥和证书，你可以证明我确实是*google.com*。我还发送了一个 32 字节的值，它对应于你要求我使用的 TLS 版本。最后，我还包括了一个签名和一个*消息认证码*（*MAC*），这是通过使用你的公钥对我们迄今讨论的所有内容进行衍生的，这样你就可以在收到我的回复时验证其完整性。

1.  客户端（自言自语）一个我信任的认证机构签署了服务器的证书，所以我确信我正在与*google.com*通信。我通过使用我的私钥从服务器的签名中衍生出了这次会话的对称密钥。使用这个对称密钥，我验证了 MAC，并确保没有人篡改服务器的回复。回复中的 32 个字节对应于 TLS 版本 1.3，因此没有人试图欺骗服务器使用更旧、更弱的 TLS 版本。我现在拥有了一切所需的东西，可以与服务器进行安全通信。

1.  客户端（发送给服务器）*这是一些加密数据。*

服务器的 Hello 消息中的 32 字节值防止了*降级攻击*，即攻击者拦截客户端的 Hello 消息并修改它，要求使用更旧、更弱的 TLS 版本。如果客户端请求的是 TLS v1.3，但攻击者将客户端的 Hello 消息改为请求 TLS v1.1，那么服务器的 Hello 消息中的 32 字节值将对应于 TLS v1.1。当客户端接收到服务器的 Hello 消息时，它会注意到该值指示了错误的 TLS 版本，从而中止握手。

从现在开始，客户端和服务器使用 AES-GCM 对称密钥加密（在这个假设的例子中）。客户端和服务器都会在传输层之前将应用层负载封装到 TLS 记录中，然后将负载传递给传输层。

尽管名称中带有“传输”，TLS 并不是一个传输层协议。相反，它位于 TCP/IP 协议栈的传输层和应用层之间。TLS 在将应用层协议的负载传递到传输层之前会先对负载进行加密。一旦负载到达目的地，TLS 会从传输层接收负载，对其解密，并将负载传递给应用层协议。

### 前向保密

我们假设对话中的握手方法是 TLS v1.3 中使用的 Diffie-Hellman (DH) 密钥交换的一个例子。*DH 密钥交换*要求生成新的客户端和服务器密钥对，以及一个新的对称密钥，所有这些密钥仅在会话期间有效。一旦会话结束，客户端和服务器将丢弃会话密钥。

使用每会话密钥意味着 TLS v1.3 提供了 *前向保密*；如果攻击者破解了你的会话密钥，他们只能破解该会话期间交换的数据。攻击者无法使用这些密钥解密其他会话期间交换的数据。

### 我们信任证书颁发机构

我的父亲和我在我开始写这本书之前不久去了一趟爱尔兰。为了准备这次旅行，我需要办理一本新护照，因为我的旧护照早就过期了。这个过程很简单。我填写了申请表，收集了重要的个人记录，拍了一张丑陋的照片，并把这些材料以及申请费交给了当地的美国邮政局分局。我还向公证人证明了我就是我自己。几周后，我收到了新办理的美国护照。

当我们到达爱尔兰时，一位可爱的海关官员迎接了我们，并要求出示护照。她在电脑验证我们的身份的同时，询问了关于我们假期的一些问题。不到三分钟，她就把护照还给我们，欢迎我们来到爱尔兰。

我的护照代表了美国政府对我身份的认证，证明我是**亚当·伍德贝克**。但它的有效性仅取决于爱尔兰对美国政府验证我身份能力的信任。如果爱尔兰不信任美国，它将不会相信美国说我就是我，并很可能拒绝让我进入该国。（说实话，我并没有足够的魅力让海关人员仅凭我自己的话就放行我。）

TLS 的证书工作原理与我的护照非常相似。如果我需要为 *woodbeck.net* 获取一个新的 TLS 证书，我会向证书颁发机构（如 Let’s Encrypt）提出请求。证书颁发机构会验证我是否是 *woodbeck.net* 的合法拥有者。一旦确认无误，证书颁发机构会为 *woodbeck.net* 签发新的证书，并使用其证书进行加密签名。我的服务器可以向客户端展示这个证书，客户端通过验证证书颁发机构的签名来认证我的服务器，从而确认他们与真正的 *woodbeck.net* 通信，而不是与冒名顶替者通信。

证书颁发机构为*woodbeck.net*签发签名证书类似于美国政府为我签发护照。它们都是由受信任的机构签发，证明其主体的真实性。就像爱尔兰对美国的信任一样，客户端只有在信任签发证书的证书颁发机构时，才会信任*woodbeck.net*证书。我可以像创建一个宣称是我护照的文件一样，创建自己的证书颁发机构并自签证书。但爱尔兰宁可承认 Jack Daniel’s 田纳西威士忌优于 Jameson 爱尔兰威士忌，也不可能信任我自签发的护照，世界上没有任何操作系统或网页浏览器会信任我自签的证书。

### 如何妥协 TLS

2013 年 12 月 24 日，谷歌发现土耳其的 Turktrust 证书颁发机构错误地签发了一个证书，使恶意行为者能够伪装成*google.com*。这意味着攻击者可以欺骗你的网页浏览器，使其认为自己与谷歌通过 TLS 连接进行通信，并诱使你泄露凭证。谷歌迅速注意到这一错误，并采取了补救措施。

Turktrust 的失误削弱了它的权威并破坏了我们的信任。但即使证书颁发机构运作正常，攻击者也可以将目标转向个人。如果攻击者能够在你的操作系统受信任的证书存储中安装自己的 CA 证书，你的计算机会信任他签发的任何证书。这意味着攻击者可以妥协你所有的 TLS 流量。

大多数人不会得到这种特别的关注。相反，攻击者更可能妥协服务器。一旦服务器被攻破，攻击者就能捕获所有 TLS 流量及其对应的会话密钥。

你不太可能遇到这些场景，但了解它们是可能发生的非常重要。总体而言，TLS 1.3 提供了极好的安全性，并且很难被妥协，因为它有完整的握手签名、降级保护、前向保密性和强加密。

## 保护传输中的数据

无论是你自己的数据还是他人的数据，确保你在网络上传输的数据的完整性应该是你的主要关注点。Go 使得使用 TLS 变得如此简单，以至于你几乎无法为不使用它辩解。在本节中，你将学习如何为客户端和服务器添加 TLS 支持。你还将看到 TLS 如何在 TCP 上工作，以及如何通过证书固定技术来减轻恶意证书的威胁。

### 客户端 TLS

客户端在握手过程中的主要关注点是通过使用证书验证服务器。如果客户端无法信任服务器，就无法认为与服务器的通信是安全的。`net/http/httptest`包提供了方便的构造，能够轻松演示 Go 的 HTTP-over-TLS 支持（参见 Listing 11-1）。

```
package ch11

import (
    "crypto/tls"
    "net"
    "net/http"
    "net/http/httptest"
    "strings"
    "testing"
    "time"

    "golang.org/x/net/http2"
)

func TestClientTLS(t *testing.T) {
    ts := 1httptest.NewTLSServer(
        http.HandlerFunc(
            func(w http.ResponseWriter, r *http.Request) {
                if 2r.TLS == nil {
                    u := "https://" + r.Host + r.RequestURI
                    http.Redirect(w, r, u, http.StatusMovedPermanently)
                    return
                }

                w.WriteHeader(http.StatusOK)
            },
        ),
    )
    defer ts.Close()

    resp, err := 3ts.Client().Get(ts.URL)
    if err != nil {
        t.Fatal(err)
    }

 if resp.StatusCode != http.StatusOK {
        t.Errorf("expected status %d; actual status %d",
            http.StatusOK, resp.StatusCode)
    }
```

清单 11-1：测试 HTTPS 客户端和服务器支持 (*tls_client_test.go*)

`httptest.NewTLSServer`函数返回一个 HTTPS 服务器 1。除了函数名之外，这段代码看起来与我们在第八章中使用的`httptest`完全相同。在这里，`httptest.NewTLSServer`函数处理了 HTTPS 服务器的 TLS 配置细节，包括创建一个新的证书。此证书没有受到任何受信任机构的签名，因此没有辨别能力的 HTTPS 客户端会信任它。你将很快看到如何通过使用预配置的客户端来绕过这个细节。

如果服务器通过 HTTP 接收到客户端的请求，则请求的`TLS`字段将为`nil`。你可以检查这种情况 2，并相应地将客户端重定向到 HTTPS 端点。

出于测试目的，服务器的`Client`方法 3 返回一个新的`*http.Client`，该客户端固有信任服务器的证书。你可以使用此客户端测试处理程序中的 TLS 特定代码。

让我们看看在清单 11-2 中，当你尝试使用一个新的客户端与相同的服务器进行通信时，且该客户端没有固有信任服务器证书时会发生什么。

```
`--snip--`

    tp := &http.Transport{
        TLSClientConfig: &tls.Config{
            CurvePreferences: []tls.CurveID{1tls.CurveP256},
            MinVersion:       tls.VersionTLS12,
        },
    }

    err = 2http2.ConfigureTransport(tp)
    if err != nil {
        t.Fatal(err)
    }

    client2 := &http.Client{Transport: tp}

    _, err = client2.Get(ts.URL)
    if err == nil || !strings.Contains(err.Error(),
        "certificate signed by unknown authority") {
        t.Fatalf("expected unknown authority error; actual: %q", err)
    }

 3 tp.TLSClientConfig.InsecureSkipVerify = true

    resp, err = client2.Get(ts.URL)
    if err != nil {
        t.Fatal(err)
    }

 if resp.StatusCode != http.StatusOK {
        t.Errorf("expected status %d; actual status %d",
            http.StatusOK, resp.StatusCode)
    }
}
```

清单 11-2：使用有辨别能力的客户端测试 HTTPS 服务器 (*tls_client_test.go*)

你通过创建一个新的传输，定义其 TLS 配置，并配置`http2`使用该传输，来覆盖客户端传输中的默认 TLS 配置。好的做法是将客户端的曲线首选项限制为 P-256 曲线 1，并避免使用 P-384 和 P-521。P-256 对时间攻击具有免疫性，而 P-384 和 P-521 则没有。此外，客户端将协商最低 TLS 1.2\。

*椭圆曲线*是一种平面曲线，曲线上的所有点都满足相同的多项式方程。与使用大素数来生成密钥的第一代加密技术（如 RSA）不同，椭圆曲线加密使用椭圆曲线上的点来生成密钥。P-256、P-384 和 P-521 是在美国国家标准与技术研究院（NIST）数字签名标准中定义的特定椭圆曲线。你可以在《联邦信息处理标准》（FIPS）出版物 186-4 中找到更多细节（[`nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf`](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf)）。

由于你的传输不再依赖于默认的 TLS 配置，客户端不再具备固有的 HTTP/2 支持。如果你想使用 HTTP/2，你需要显式地为你的传输添加 HTTP/2 支持 2。当然，这个测试并不依赖于 HTTP/2，但如果你没有意识到覆盖传输的 TLS 配置会移除 HTTP/2 支持，那么这个实现细节可能会让你陷入困境。

你的客户端使用操作系统的受信证书存储，因为你没有明确告诉它信任哪些证书。第一次调用测试服务器时会出现错误，因为你的客户端不信任服务器证书的签发者。你可以绕过这个问题，通过将 `InsecureSkipVerify` 字段设置为 `true`3 来配置客户端的传输跳过验证服务器的证书。我不建议你在除调试之外的任何情况下启用 `InsecureSkipVerify`。我认为启用该选项并发布代码是一个代码气味。你将在本章稍后学习更好的替代方案，当我们讨论 *证书钉扎* 概念时。正如字段名所示，启用此选项会使你的客户端本质上不安全，容易受到中间人攻击，因为它现在会盲目信任服务器提供的任何证书。如果你使用新配置的天真客户端进行相同的调用，你会发现它会愉快地与服务器协商 TLS。

### TCP 上的 TLS

TLS 是有状态的；客户端和服务器在初始握手时协商会话参数，一旦达成一致，它们就会在会话期间交换加密的 TLS 记录。由于 TCP 本身也是有状态的，它是实现 TLS 的理想传输层协议，因为你可以利用 TCP 的可靠性保证来维持你的 TLS 会话。

让我们暂时不考虑应用协议，学习如何通过 TCP 建立 TLS 连接。清单 11-3 展示了如何使用 `crypto/tls` 包，通过几行代码发起 TLS 连接。

```
`--snip--`

func TestClientTLSGoogle(t *testing.T) {
    conn, err := 1tls.DialWithDialer(
        &net.Dialer{Timeout: 30 * time.Second},
        "tcp",
        "www.google.com:443",
        &tls.Config{
            CurvePreferences: []tls.CurveID{tls.CurveP256},
            MinVersion:       tls.VersionTLS12,
        },
    )
    if err != nil {
        t.Fatal(err)
    }

    state := 2conn.ConnectionState()
    t.Logf("TLS 1.%d", state.Version-tls.VersionTLS10)
    t.Log(tls.CipherSuiteName(state.CipherSuite))
    t.Log(state.VerifiedChains[0][0].Issuer.Organization[0])

    _ = conn.Close()
}
```

清单 11-3：启动与 *www.google.com* 的 TLS 连接（*tls_client_test.go*）

`tls.DialWithDialer` 函数 1 接受一个 `*net.Dialer`、网络类型、地址和一个 `*tls.Config`。在这里，你为拨号器设置了 30 秒的超时，并指定了推荐的 TLS 设置。如果成功，你可以检查连接的状态 2，以获取关于 TLS 连接的详细信息。

清单 11-4 显示了 清单 11-3 测试的输出。

```
$ **go test -race -run TestClientTLSGoogle -v ./...**
=== RUN   TestClientTLSGoogle
    TestClientTLSGoogle: tls_client_test.go:89: TLS 1.3
    TestClientTLSGoogle: tls_client_test.go:90: TLS_AES_128_GCM_SHA256
    TestClientTLSGoogle: tls_client_test.go:91: Google Trust Services
--- PASS: TestClientTLSGoogle (0.31s)
PASS
```

清单 11-4：运行 `TestClientTLSGoogle` 测试

你的 TLS 客户端使用的是 TLS_AES_128_GCM_SHA256 密码套件，基于 TLS 1.3 版本。注意，`tls.DialWithDialer` 并没有对服务器的证书提出异议。底层 TLS 客户端使用操作系统的受信证书存储，并确认 *www.google.com* 的证书由受信的证书颁发机构签发——在本例中是 Google Trust Services。

### 服务器端 TLS

服务器端的代码与之前学习的差别不大。主要的不同之处在于，服务器需要在握手过程中向客户端提供证书。你可以使用 Go 的 *src/crypto/tls* 子目录中的 *generate_cert.go* 文件来创建证书。对于生产环境，最好使用 Let’s Encrypt 或其他证书颁发机构的证书。你可以使用 LEGO 库（[`github.com/go-acme/lego/`](https://github.com/go-acme/lego/)）将证书管理功能添加到你的服务中。生成一个新的证书和私钥，像这样：

```
$ **go run $GOROOT/src/crypto/tls/generate_cert.go -host localhost -ecdsa-curve P256**
```

该命令会创建一个名为 *cert.pem* 的证书，主机名为 *localhost*，并创建一个名为 *key.pem* 的私钥。接下来的代码假设这两个文件已经存在于当前目录中。

延续前几章的传统，列表 11-5 包含了一个仅支持 TLS 的回显服务器的第一个代码片段。

```
package ch11

import (
    "context"
    "crypto/tls"
    "fmt"
    "net"
    "time"
)

func NewTLSServer(ctx context.Context, address string,
    maxIdle time.Duration, tlsConfig *tls.Config) *Server {
    return &Server{
        ctx:       ctx,
        ready:     make(chan struct{}),
        addr:      address,
        maxIdle:   maxIdle,
        tlsConfig: tlsConfig,
    }
}

type Server struct {
    ctx   context.Context
    ready chan struct{}

    addr      string
    maxIdle   time.Duration
    tlsConfig *tls.Config
}

func (s *Server) 1Ready() {
    if s.ready != nil {
        <-s.ready
    }
}
```

列表 11-5：服务器结构体类型和构造函数（*tls_echo.go*）

`Server` 结构体有一些字段，用于记录服务器的设置、TLS 配置和一个信号通道，用于指示服务器何时准备好接受传入的连接。稍后你会写一个测试用例，并使用 `Ready` 方法 1 来阻塞，直到服务器准备好接受连接。

`NewTLSServer` 函数接受一个用于停止服务器的上下文、一个地址、服务器允许连接空闲的最长时间以及 TLS 配置。尽管控制空闲客户端与 TLS 无关，但你将使用最大空闲时间来推进套接字的截止时间，就像在第三章中一样。

你在前几章使用的服务器依赖于监听和服务这两个独立的概念。通常，你会调用一个辅助函数来同时执行这两个操作，比如 `net/http` 服务器的 `ListenAndServe` 方法。列表 11-6 向回显服务器添加了一个类似的方法。

```
`--snip--`

func (s *Server) ListenAndServeTLS(certFn, keyFn string) error {
    if s.addr == "" {
        s.addr = "localhost:443"
    }

    l, err := net.Listen("tcp", s.addr)
    if err != nil {
        return fmt.Errorf("binding to tcp %s: %w", s.addr, err)
    }

    if s.ctx != nil {
        go func() {
            <-s.ctx.Done()
            _ = l.Close()
        }()
    }

    return s.ServeTLS(l, certFn, keyFn)
}
```

列表 11-6：为监听和服务方法添加信号，指示服务器已准备好接收连接（*tls_echo.go*）

`ListenAndServe` 方法接受证书和私钥的完整路径，并返回错误。它创建一个绑定到服务器地址的 `net.Listener`，然后启动一个 goroutine，在你取消上下文时关闭监听器。最后，方法将监听器、证书路径和密钥路径传递给服务器的 `ServeTLS` 方法。

列表 11-7 通过 `ServeTLS` 方法完善了回显服务器的实现。

```
`--snip--`

func (s Server) ServeTLS(l net.Listener, certFn, keyFn string) error {
    if s.tlsConfig == nil {
        s.tlsConfig = &tls.Config{
 CurvePreferences:         []tls.CurveID{tls.CurveP256},
            MinVersion:               tls.VersionTLS12,
         1 PreferServerCipherSuites: true,
        }
    }

    if len(s.tlsConfig.Certificates) == 0 &&
        s.tlsConfig.GetCertificate == nil {
        cert, err := 2tls.LoadX509KeyPair(certFn, keyFn)
        if err != nil {
            return fmt.Errorf("loading key pair: %v", err)
        }

        s.tlsConfig.Certificates = []tls.Certificate{cert}
    }

    tlsListener := 3tls.NewListener(l, s.tlsConfig)
    if s.ready != nil {
        close(s.ready)
    }
```

列表 11-7：为 `net.Listener` 添加 TLS 支持（*tls_echo.go*）

`ServeTLS` 方法首先检查服务器的 TLS 配置。如果配置为 `nil`，它将添加一个默认配置，并将 `PreferServerCipherSuites` 设置为 `true`。`PreferServerCipherSuites` 对服务器有意义，它使服务器使用自己首选的密码套件，而不是听从客户端的偏好。

如果服务器的 TLS 配置没有至少一个证书，或者其`GetCertificate`方法为`nil`，你可以通过从文件系统读取证书和私钥文件来创建一个新的`tls.Certificate` 2。

在这段代码中，服务器拥有一个 TLS 配置，其中至少包含一个证书，准备向客户端展示。剩下的就是通过将 TLS 配置和监听器传递给`tls.NewListener`函数来为`net.Listener`添加 TLS 支持 3。`tls.NewListener`函数充当中间件，它增强了监听器，使其从`Accept`方法返回 TLS 支持的连接对象。

列表 11-8 通过从监听器接受连接并在单独的 goroutine 中处理它们来完成`ServeTLS`方法。

```
`--snip--`

    for {
        conn, err := 1tlsListener.Accept()
        if err != nil {
            return fmt.Errorf("accept: %v", err)
        }

        go func() {
            defer func() { _ = 2conn.Close() }()

            for {
                if s.maxIdle > 0 {
 err := 3conn.SetDeadline(time.Now().Add(s.maxIdle))
                    if err != nil {
                        return
                    }
                }

                buf := make([]byte, 1024)
                n, err := 4conn.Read(buf)
                if err != nil {
                    return
                }

                _, err = conn.Write(buf[:n])
                if err != nil {
                    return
                }
            }
        }()
    }
}
```

列表 11-8：从监听器接受 TLS 支持的连接（*tls_echo.go*）

这个模式与前面章节中你见过的类似。你使用一个无限的`for`循环，不断阻塞在监听器的`Accept`方法 1 上，当客户端成功连接时，它会返回一个新的`net.Conn`对象。由于你使用的是一个支持 TLS 的监听器，它返回的是具有 TLS 支持的连接对象。你与这些连接对象的交互方式与以往相同。Go 在这一点上将 TLS 的细节抽象给你。然后你会将这个连接分配到一个新的 goroutine 中，从那时起处理该连接。

服务器以相同的方式处理每个连接。它首先有条件地将套接字截止日期设置为服务器的最大空闲时长 3，然后等待客户端发送数据。如果服务器在达到截止日期之前没有从套接字读取到任何数据，连接的`Read`方法 4 会返回*I/O 超时*错误，最终导致连接关闭 2。

如果相反，服务器从连接读取数据，它会将相同的负载写回给客户端。控制循环回绕以重置截止日期，然后等待来自客户端的下一个负载。

### 证书钉扎

在本章前面，我们讨论了破坏 TLS 信任的方式，无论是证书颁发机构颁发伪造证书，还是攻击者将恶意证书注入到计算机的可信证书存储中。你可以通过使用证书钉扎（certificate pinning）来减轻这两种攻击。

*证书钉扎*是指放弃使用操作系统的可信证书存储，并在应用程序中显式定义一个或多个可信证书。你的应用程序将只信任来自展示钉扎证书或由钉扎证书签名的证书的主机。如果你计划在零信任环境中部署客户端，并且需要与服务器安全通信，请考虑将服务器的证书钉扎到每个客户端。

假设前面一节介绍的服务器使用了您为*localhost*主机名生成的*cert.pem*和*key.pem*文件，所有客户端将在服务器展示证书后立即中止 TLS 连接。客户端不会信任服务器的证书，因为没有受信任的证书机构签署它。

您可以将`tls.Config`的`InsecureSkipVerify`字段设置为`true`，但由于此方法不安全，我不建议您将其视为一个实际选择。相反，我们明确告诉客户端它可以通过将服务器证书固定到客户端来信任服务器的证书。列表 11-9 展示了这个过程的开始。

```
package ch11

import (
    "bytes"
    "context"
    "crypto/tls"
    "crypto/x509"
    "io"
    "io/ioutil"
    "strings"
    "testing"
    "time"
)

func TestEchoServerTLS(t *testing.T) {
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()

    serverAddress := "localhost:34443"
    maxIdle := time.Second
    server := NewTLSServer(ctx, serverAddress, maxIdle, nil)
    done := make(chan struct{})

    go func() {
        err := 1server.ListenAndServeTLS("cert.pem", "key.pem")
        if err != nil && !strings.Contains(err.Error(),
            "use of closed network connection") {
            t.Error(err)
            return
        }
        done <- struct{}{}
    }()
2     server.Ready()
```

列表 11-9：创建一个新的 TLS 回显服务器并在后台启动它（*tls_echo_test.go*）

由于*cert.pem*中的主机名是*localhost*，您创建一个新的 TLS 回显服务器，监听*localhost*端口 34443。这里端口不重要，但客户端期望服务器可以通过与证书中显示的相同主机名进行访问。您通过使用*cert.pem*和*key.pem*文件 1 启动服务器并阻塞，直到它准备好接受连接 2。

列表 11-10 从我们停下的地方继续，通过创建一个带有明确信任服务器证书的客户端 TLS 配置。

```
`--snip--`

    cert, err := ioutil.ReadFile("cert.pem")
    if err != nil {
        t.Fatal(err)
    }

    certPool := 1x509.NewCertPool()
    if ok := certPool.AppendCertsFromPEM(cert); !ok {
        t.Fatal("failed to append certificate to pool")
    }

    tlsConfig := &tls.Config{
        CurvePreferences: []tls.CurveID{tls.CurveP256},
        MinVersion:       tls.VersionTLS12,
     2 RootCAs:          certPool,
    }
```

列表 11-10：将服务器证书固定到客户端（*tls_echo_test.go*）

将服务器证书固定到客户端是直接的。首先，您需要读取*cert.pem*文件。然后，您创建一个新的证书池 1，并将证书添加到池中。最后，您将证书池添加到`tls.Config`的`RootCAs`字段 2。如名称所示，您可以将多个受信任的证书添加到证书池中。这在您迁移到新证书，但尚未完全淘汰旧证书时非常有用。

使用此配置的客户端将仅认证展示*cert.pem*证书或任何由其签署的证书的服务器。我们将在测试的其余部分确认这一行为（见列表 11-11）。

```
`--snip--`

    conn, err := 1tls.Dial("tcp", serverAddress, tlsConfig)
    if err != nil {
        t.Fatal(err)
    }

    hello := []byte("hello")
    _, err = conn.Write(hello)
    if err != nil {
        t.Fatal(err)
    }

    b := make([]byte, 1024)
    n, err := conn.Read(b)
    if err != nil {
 t.Fatal(err)
    }

    if actual := b[:n]; !bytes.Equal(hello, actual) {
        t.Fatalf("expected %q; actual %q", hello, actual)
    }

 2 time.Sleep(2 * maxIdle)
    _, err = conn.Read(b)
    if err != 3io.EOF {
        t.Fatal(err)
    }

    err = conn.Close()
    if err != nil {
        t.Fatal(err)
    }

    cancel()
    <-done
}
```

列表 11-11：通过使用固定的证书进行服务器认证（*tls_echo_test.go*）

您将包含固定服务器证书 1 的`tls.Config`传递给`tls.Dial`。您的 TLS 客户端将在不需要使用`InsecureSkipVerify`及其带来的所有不安全性的情况下验证服务器的证书。

现在，您已经与服务器建立了受信任的连接，尽管服务器展示了一个未签名的证书，我们还是来确保服务器按预期工作。它应该回显您发送的任何消息。如果您长时间空闲 2，您会发现与套接字的下一次交互会导致错误 3，显示服务器关闭了套接字。

## 相互 TLS 认证

在前一节中，你学习了客户端如何通过使用服务器的证书和受信任的第三方证书，或者通过配置客户端显式信任服务器的证书来验证服务器身份。服务器也可以以相同的方式验证客户端身份。这在零信任网络架构中尤为重要，在这种架构下，客户端和服务器都必须证明自己的身份。例如，你可能有一个位于你网络外部的客户端，该客户端必须向代理服务器出示证书，代理服务器才会允许该客户端访问你的受信任网络资源。同样，客户端也会验证代理服务器出示的证书，以确保它与代理服务器通信，而不是与由恶意行为者控制的代理服务器通信。

你可以指示服务器只与已认证的客户端建立 TLS 会话。那些客户端必须出示由受信任的证书颁发机构签名或已固定到服务器的证书。在你查看示例代码之前，客户端需要一个可以向服务器出示的证书用于身份验证。然而，客户端不能使用在 *$GOROOT/src/crypto/tls/generate_cert.go* 中生成的证书进行客户端身份验证。相反，你需要创建自己的证书和私钥。

### 为身份验证生成证书

Go 的标准库包含了生成你自己证书所需的所有内容，使用的是椭圆曲线数字签名算法（ECDSA）和 P-256 椭圆曲线。列表 11-12 展示了一个命令行工具的初步实现，正是用来做这个的。在你阅读它时，记住它可能不完全适合你的使用场景。例如，它生成的是 10 年的证书，并且使用我的名字作为证书的主体，这可能不是你在代码中想使用的（不过如果你愿意使用，我会感到受宠若惊）。根据需要进行调整。

```
package main

import (
    "crypto/ecdsa"
    "crypto/elliptic"
    "crypto/rand"
    "crypto/x509"
    "crypto/x509/pkix"
    "encoding/pem"
    "flag"
    "log"
    "math/big"
    "net"
    "os"
    "strings"
    "time"
)

var (
    host = flag.String("host", "localhost",
        "Certificate's comma-separated host names and IPs")
    certFn = flag.String("cert", "cert.pem", "certificate file name")
    keyFn  = flag.String("key", "key.pem", "private key file name")
)

func main() {
    flag.Parse()

    serial, err := 1rand.Int(rand.Reader, new(big.Int).Lsh(big.NewInt(1), 
        128))
    if err != nil {
        log.Fatal(err)
    }

    notBefore := time.Now()
    template := x509.Certificate{
        SerialNumber: serial,
 Subject: pkix.Name{
            Organization: []string{"Adam Woodbeck"},
        },
        NotBefore: notBefore,
        NotAfter:  notBefore.Add(10 * 356 * 24 * time.Hour),
        KeyUsage: x509.KeyUsageKeyEncipherment |
            x509.KeyUsageDigitalSignature |
            x509.KeyUsageCertSign,
        ExtKeyUsage: []x509.ExtKeyUsage{
            x509.ExtKeyUsageServerAuth,
          2x509.ExtKeyUsageClientAuth,
        },
        BasicConstraintsValid: true,
        IsCA:                  true,
    }
```

列表 11-12：创建 X.509 证书模板 (*cert/generate.go*)

该命令行工具接受一个用逗号分隔的主机名和 IP 地址列表，这些主机名和 IP 地址将使用该证书。它还允许你指定证书和私钥的文件名，但默认使用我们熟悉的 *cert.pem* 和 *key.pem* 文件名。

生成证书和私钥的过程包括在代码中构建一个模板，然后将其编码为 X.509 格式。每个证书都需要一个序列号，通常由证书颁发机构分配。由于你正在生成自己的自签名证书，你使用加密随机的、无符号的 128 位整数 1 来生成自己的序列号。接着，你创建一个 `x509.Certificate` 对象，表示一个 X.509 格式的证书，并设置各种值，如序列号、证书的主题、有效期和该证书的各种用途。由于你希望使用此证书进行客户端身份验证，你必须包括 `x509.ExtKeyUsageClientAuth` 值 2。如果遗漏此值，服务器将无法在客户端提供证书时验证该证书。

模板几乎准备好了。你只需要在生成证书之前添加主机名和 IP 地址（参见 清单 11-13）。

```
`--snip--`

    for _, h := range 1strings.Split(*host, ",") {
        if ip := net.ParseIP(h); ip != nil {
         2 template.IPAddresses = append(template.IPAddresses, ip)
        } else {
         3 template.DNSNames = append(template.DNSNames, h)
        }
    }

    priv, err := 4ecdsa.GenerateKey(elliptic.P256(), rand.Reader)
    if err != nil {
        log.Fatal(err)
    }

 der, err := 5x509.CreateCertificate(rand.Reader, &template,
        &template, &priv.PublicKey, priv)
    if err != nil {
        log.Fatal(err)
    }

    cert, err := os.Create(*certFn)
    if err != nil {
        log.Fatal(err)
    }

    err = 6pem.Encode(cert, &pem.Block{Type: "CERTIFICATE", Bytes: der})
    if err != nil {
        log.Fatal(err)
    }

    if err := cert.Close(); err != nil {
        log.Fatal(err)
    }
    log.Println("wrote", *certFn)
```

清单 11-13：编写隐私增强邮件（PEM）编码的证书（*cert/generate.go*）

你遍历以逗号分隔的主机名和 IP 地址列表 1，将每个地址分配到模板中的相应切片。如果主机名是 IP 地址，你将其分配到 `IPAddresses` 切片 2。否则，你将主机名分配到 `DNSNames` 切片 3。Go 的 TLS 客户端使用这些值来验证服务器。例如，如果客户端连接到 *https://www.google.com*，但服务器证书中的常见名称或备用名称与 *www.google.com* 的主机名或解析后的 IP 地址不匹配，客户端将无法验证服务器。

接下来，你使用 P-256 椭圆曲线生成一个新的 ECDSA 私钥 4。此时，你已经拥有生成证书所需的一切。`x509.CreateCertificate` 函数 5 接受一个熵源（`crypto/rand` 的 `Reader` 是理想选择）、新证书的模板、父证书、公钥和相应的私钥。然后，它返回一个字节切片，包含按区分编码规则（DER）编码的证书。你使用模板作为父证书，因为生成的证书将自签名。接下来要做的就是创建一个新文件，生成一个新的 `pem.Block`，并将 DER 编码的字节切片进行 PEM 编码，保存到新文件中 6。你不需要担心各种编码。Go 非常适合在磁盘上使用 PEM 编码的证书。

现在你已经在磁盘上生成了新的证书，我们来编写相应的私钥，参见 清单 11-14。

```
`--snip--`

    key, err := os.OpenFile(*keyFn, os.O_WRONLY|os.O_CREATE|os.O_TRUNC,
      10600)
    if err != nil {
        log.Fatal(err)
 }

    privKey, err := 2x509.MarshalPKCS8PrivateKey(priv)
    if err != nil {
        log.Fatal(err)
    }

    err = 3pem.Encode(key, &pem.Block{Type: "EC PRIVATE KEY",
    Bytes: privKey})
    if err != nil {
        log.Fatal(err)
    }

    if err := key.Close(); err != nil {
        log.Fatal(err)
    }
    log.Println("wrote", *keyFn)
}
```

清单 11-14：编写 PEM 编码的私钥（*cert/generate.go*）

证书是为了公开共享，而私钥则完全相反：是私密的。你应该小心地为它分配最小的权限。在这里，你只给用户读写私钥文件的权限 1，并移除其他所有人的访问权限。我们将私钥编组为字节切片 2，并且类似地，将其分配给新的`pem.Block`，然后将 PEM 编码的输出写入私钥文件 3。

Listing 11-15 使用前面的代码生成了服务器和客户端的证书和密钥对。

```
$ **go run cert/generate.go -cert serverCert.pem -key serverKey.pem -host localhost**
2006/01/02 15:04:05 wrote serverCert.pem
2006/01/02 15:04:05 wrote serverKey.pem
$ **go run cert/generate.go -cert clientCert.pem -key clientKey.pem -host localhost**
2006/01/02 15:04:05 wrote clientCert.pem
2006/01/02 15:04:05 wrote clientKey.pem
```

Listing 11-15: 为服务器和客户端生成证书和私钥对

由于服务器绑定到*localhost*，而客户端从*localhost*连接到服务器，因此这个值适用于客户端和服务器证书。如果你想将客户端移动到不同的主机名或将服务器绑定到 IP 地址，例如，你需要相应地更改*host*标志。

### 实现互相 TLS 认证

现在你已经为服务器和客户端生成了证书和私钥对，你可以开始编写它们的代码了。让我们编写一个测试，实现我们的回显服务器和客户端之间的互相 TLS 认证，从 Listing 11-16 开始。

```
package ch11

import (
    "bytes"
    "context"
    "crypto/tls"
    "crypto/x509"
    "errors"
    "io/ioutil"
    "strings"
    "testing"
)

func caCertPool(caCertFn string) (*x509.CertPool, error) {
    caCert, err := 1ioutil.ReadFile(caCertFn)
    if err != nil {
        return nil, err
    }

    certPool := x509.NewCertPool()
    if ok := 2certPool.AppendCertsFromPEM(caCert); !ok {
        return nil, errors.New("failed to add certificate to pool")
    }

    return certPool, nil
}
```

Listing 11-16: 创建证书池以提供 CA 证书(*tls_mutual_test.go*)

客户端和服务器都使用`caCertPool`函数来创建一个新的 X.509 证书池。该函数接受一个 PEM 编码证书的文件路径，你在第 1 步中读取该证书并将其附加到新的证书池 2 中。证书池作为受信任证书的来源。客户端将服务器的证书放入其证书池中，反之亦然。

Listing 11-17 详细介绍了初步的测试代码，用于演示客户端和服务器之间的互相 TLS 认证。

```
`--snip--`

func TestMutualTLSAuthentication(t *testing.T) {
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()

    serverPool, err := caCertPool(1"clientCert.pem")
    if err != nil {
        t.Fatal(err)
    }

    cert, err := 2tls.LoadX509KeyPair("serverCert.pem", "serverKey.pem")
    if err != nil {
        t.Fatalf("loading key pair: %v", err)
    }
```

Listing 11-17: 实例化 CA 证书池和服务器证书(*tls_mutual_test.go*)

在创建服务器之前，你需要先用客户端的证书 1 填充一个新的 CA 证书池。此时，你还需要加载服务器的证书 2，而不是像之前的列表中那样依赖服务器的`ServeTLS`方法来为你加载。你为什么现在需要服务器的证书，看到 Listing 11-18 中的 TLS 配置更改后就会明白。

```
`--snip--`

    serverConfig := &tls.Config{
        Certificates: []tls.Certificate{cert},
     1 GetConfigForClient: func(hello *tls.ClientHelloInfo) (*tls.Config,
            error) {
            return &tls.Config{
                Certificates:             []tls.Certificate{2cert},
             3 ClientAuth:               tls.RequireAndVerifyClientCert,
             4 ClientCAs:                serverPool,
                CurvePreferences:         []tls.CurveID{tls.CurveP256},
                MinVersion:             5tls.VersionTLS13,
                PreferServerCipherSuites: true,
```

Listing 11-18: 使用 GetConfigForClient 访问客户端的 hello 信息(*tls_mutual_test.go*)

请记住，在 Listing 11-13 中，你定义了生成客户端证书时使用的模板中的`IPAddresses`和`DNSNames`切片。这些值填充了客户端证书的公共名称和备用名称部分。你已经了解到 Go 的 TLS 客户端使用这些值来验证服务器。但是服务器并不使用这些来自客户端证书的值来验证客户端。

由于你正在实现双向 TLS 认证，你需要对服务器的证书验证过程进行一些更改，以便它根据客户端证书的公共名称和备用名称来验证客户端的 IP 地址或主机名。为此，服务器至少需要知道客户端的 IP 地址。在证书验证之前获取客户端连接信息的唯一方法是定义`tls.Config`的`GetConfigForClient`方法 1。这个方法允许你定义一个函数，该函数接收在 TLS 握手过程中与客户端创建的`*tls.ClientHelloInfo`对象。通过这个，你可以获取客户端的 IP 地址。但首先，你需要返回一个适当的 TLS 配置。

你将服务器的证书添加到 TLS 配置中 2，并将服务器池添加到 TLS 配置的`ClientCAs`字段 4 中。这个字段是服务器的等效项，类似于客户端 TLS 配置中的`RootCAs`字段。你还需要告诉服务器，每个客户端在完成 TLS 握手过程之前必须提供有效的证书 3。由于你控制客户端和服务器，指定一个最小的 TLS 协议版本为 1.35。

这个函数为每个客户端连接返回相同的 TLS 配置。如前所述，你使用`GetConfigForClient`方法的唯一原因是为了从客户端的 hello 信息中获取客户端的 IP 地址。示例 11-19 实现了通过使用客户端的 IP 地址以及证书的公共名称和备用名称来验证客户端的过程。

```
`--snip--`

             1 VerifyPeerCertificate: func(rawCerts [][]byte,
                    verifiedChains [][]*x509.Certificate) error {

                    opts := x509.VerifyOptions{
                        KeyUsages: []x509.ExtKeyUsage{
                          2x509.ExtKeyUsageClientAuth,
                        },
                        Roots: 3serverPool,
                    }

                    ip := strings.Split(hello.Conn.RemoteAddr().String(),
                        ":")[0]
                    hostnames, err := 4net.LookupAddr(ip)
                    if err != nil {
                        t.Errorf("PTR lookup: %v", err)
                    }
                    hostnames = append(hostnames, ip)

                    for _, chain := range verifiedChains {
                        opts.Intermediates = x509.NewCertPool()
                        for _, cert := range 5chain[1:] {
                            opts.Intermediates.AddCert(cert)
                        }

                        for _, hostname := range hostnames {
                            opts.DNSName = 6hostname
                            _, err = chain[0].Verify(opts)
                            if err == nil {
                                return nil
                            }
                        }
                    }

                    return errors.New("client authentication failed")
                },
            }, nil
        },
    }
```

示例 11-19：使服务器验证客户端的 IP 地址和主机名（*tls_mutual_test.go*）

由于你想在服务器上增强常规证书验证过程，你定义了一个合适的函数并将其分配给 TLS 配置的`VerifyPeerCertificate`方法 1。服务器在正常的证书验证检查后调用此方法。你执行的唯一额外检查是通过叶证书验证客户端的主机名。

*叶证书*是客户端提供给服务器的证书链中的最后一张证书。叶证书包含客户端的公钥。链中的所有其他证书都是中间证书，用于验证叶证书的真实性，并最终以证书颁发机构的证书作为结束。你可以在每个`verifiedChains`切片中的索引 0 找到每个叶证书。换句话说，你可以在`verifiedChains[0][0]`中找到第一个链的叶证书。如果服务器调用你分配给`VerifyPeerCertificate`方法的函数，至少可以找到第一个链中的叶证书。

创建一个新的 `x509.VerifyOptions` 对象，并修改 `KeyUsages` 方法以指示您希望执行客户端身份验证 2。然后，将服务器池分配给 `Roots` 方法 3。服务器在验证期间使用此池作为其可信证书源。

现在，从传递到 第 11-18 行 的 `GetConfigForClient` 方法的 `*tls.ClientHelloInfo` 对象 `hello` 中提取客户端的 IP 地址。使用该 IP 地址执行反向 DNS 查找 4，以考虑分配给客户端 IP 地址的任何主机名。如果此查找失败或返回空切片，则如何处理这种情况由您决定。如果您依赖客户端的主机名进行身份验证，并且反向查找失败，则无法对客户端进行身份验证。但如果您仅在证书的通用名称或备用名称中使用客户端的 IP 地址，则反向查找失败无关紧要。为了演示目的，我们将考虑失败的反向查找等同于失败的测试。最少，您应将客户端的 IP 地址附加到 `hostnames` 切片中。

唯一剩下的事情就是循环遍历每个验证的链条，将一个新的中间证书池分配给 `opts.Intermediates`，将所有证书但叶子证书添加到中间证书池 5，并尝试验证客户端 6。如果验证返回 `nil` 错误，则您已经成功认证了客户端。如果未能验证每个叶证书的每个主机名，则返回错误以指示客户端认证失败。客户端将收到一个错误，并且服务器将终止连接。

现在服务器的 TLS 配置正确验证客户端证书，接着继续在 第 11-20 行 中完成服务器实现。

```
`--snip--`

    serverAddress := "localhost:44443"
    server := NewTLSServer(ctx, serverAddress, 0, 1serverConfig)
    done := make(chan struct{})

    go func() {
        err := server.ListenAndServeTLS("serverCert.pem", "serverKey.pem")
        if err != nil &&!strings.Contains(err.Error(),
            "use of closed network connection") {
            t.Error(err)
            return
        }
 done <- struct{}{}
    }()
 2 server.Ready()
```

第 11-20 行：启动 TLS 服务器 (*tls_mutual_test.go*)

创建一个新的 TLS 服务器实例，确保传递刚刚创建的 TLS 配置 1。在一个 goroutine 中调用其 `ListenAndServeTLS` 方法，并确保在继续之前等待服务器准备好连接 2。

现在，服务器实现准备就绪，让我们继续进行测试的客户端部分。第 11-21 行 实现了一个 TLS 客户端，可以在服务器要求时提交 *clientCert.pem*。

```
`--snip--`

    clientPool, err := caCertPool(1"serverCert.pem")
    if err != nil {
        t.Fatal(err)
    }

    clientCert, err := tls.LoadX509KeyPair("clientCert.pem", "clientKey.pem")
    if err != nil {
        t.Fatal(err)
    }

    conn, err := tls.Dial("tcp", serverAddress, &tls.Config{
     2 Certificates:     []tls.Certificate{clientCert},
        CurvePreferences: []tls.CurveID{tls.CurveP256},
        MinVersion:       tls.VersionTLS13,
     3 RootCAs:          clientPool,
    })
    if err != nil {
        t.Fatal(err)
    }
```

第 11-21 行：将服务器证书固定到客户端 (*tls_mutual_test.go*)

客户端检索一个新的证书池，其中包含服务器的证书 1。然后客户端在其 TLS 配置的 `RootCAs` 字段中使用此证书池 3，这意味着客户端仅信任由 *serverCert.pem* 签名的服务器证书。您还配置客户端使用其自己的证书 2，在服务器请求时提交给服务器。

值得注意的是，客户端和服务器尚未初始化 TLS 会话。它们还没有完成 TLS 握手。如果 `tls.Dial` 返回错误，这并非因为认证问题，而更可能是由于 TCP 连接问题。让我们继续查看客户端代码以启动握手（参见 Listing 11-22）。

```
`--snip--`

    hello := []byte("hello")
    _, err = conn.Write(hello)
 if err != nil {
        t.Fatal(err)
    }

    b := make([]byte, 1024)
    n, err := 1conn.Read(b)
    if err != nil {
        t.Fatal(err)
    }

    if actual := b[:n]; !bytes.Equal(hello, actual) {
        t.Fatalf("expected %q; actual %q", hello, actual)
    }

    err = conn.Close()
    if err != nil {
        t.Fatal(err)
    }

    cancel()
    <-done
}
```

Listing 11-22: TLS 握手在你与连接互动时完成（*tls_mutual_test.go*）

从套接字连接中进行的首次读取或写入操作会自动启动客户端与服务器之间的握手过程。如果服务器拒绝客户端证书，读取调用将返回一个*坏证书*错误。但如果你创建了合适的证书并正确地进行了固定，那么客户端和服务器都能正常工作，测试也会通过。

## 你所学到的

传输层安全性（TLS）为客户端与服务器之间提供认证和加密通信。服务器向客户端呈现一个由证书颁发机构签署的证书，作为 TLS 握手过程的一部分。客户端验证证书的签署者。如果证书被客户端信任的第三方签署，那么服务器在客户端眼中就是可信的。从那时起，客户端和服务器将使用对称密钥加密通信。

默认情况下，Go 的 TLS 配置使用操作系统的可信证书存储区。该存储区通常包含来自全球领先的受信任证书颁发机构的证书。然而，我们可以修改 TLS 配置以信任特定的密钥，这一过程被称为密钥固定（key pinning）。

我们还可以修改服务器的 TLS 配置，要求客户端提供证书。然后，服务器会使用该证书以与客户端认证服务器相同的方式来认证客户端。这一过程被称为双向 TLS 认证。

TLS 1.3 为客户端和服务器之间的所有通信提供前向保密性。这意味着，一次会话的泄露不会影响其他任何会话。客户端和服务器为每个会话生成公钥和私钥对。它们还会在握手过程中交换一个临时共享密钥。一旦会话结束，客户端和服务器将删除共享密钥和它们的临时密钥对。一个能够捕获共享密钥和会话流量的攻击者只能解密该会话的流量。攻击者无法使用一个会话的共享密钥来解密其他会话的流量。

尽管 TLS 无处不在，并保护着世界上大量的数字通信，但攻击者仍然能够攻破它。证书颁发机构的部分职责是验证请求特定域名证书的实体是否拥有该域名。如果攻击者欺骗了证书颁发机构，或者证书颁发机构犯了错误并颁发了伪造的证书，那么伪造证书的持有者就可能伪装成谷歌（Google）等，并诱使人们泄露敏感信息。

另一种攻击途径是欺骗客户端将攻击者的证书添加到客户端的受信证书存储中。攻击者随后可以签发并签署任何他们想要的证书，而客户端会天然地信任攻击者所声明的身份。

攻击者还可能攻破服务器，拦截 TLS 会话密钥和机密信息，甚至在服务器解密后捕获应用层的流量。

然而，整体来说，这些攻击是罕见的，TLS 在实现身份验证和加密通信的目标方面是成功的。
