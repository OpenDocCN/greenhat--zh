# 第八章：编写 HTTP 客户端

![](img/chapterart.png)

*超文本传输协议**(HTTP)* 是一种应用层协议，被万维网使用。在 HTTP 通信中，Web 客户端向 Web 服务器发送一个 *统一资源定位符**(URL)*，然后 Web 服务器返回相应的媒体资源。在这个上下文中，*资源* 可以是图像、样式表、HTML 文档、JavaScript 文件等。例如，如果您的 Web 浏览器向 Google 的 Web 服务器发送了 URL *www.google.com*，服务器将返回 Google 的主页。我们中的大多数人每天都会进行这种 Web 交易，无论它们是来自我们的手机、电脑，还是物联网（IoT）设备，如门铃、温控器或烤面包机（是的，真的）。

本章将向您介绍 Go 的 HTTP 客户端。首先，您将学习 HTTP 的基础知识，包括请求方法和响应代码。接下来，您将探索 Go 的 HTTP 客户端，向 Web 服务器请求资源，并注意过程中可能遇到的问题。然后，您将进入标准库代码，学习实现 HTTP 客户端与服务器之间请求响应通信的相关内容。最后，您将了解使用 Go 的 HTTP 客户端与 Web 服务器交互时常见的问题。

本章将为您提供与服务进行 HTTP 交互的基础知识。您需要掌握这些基础，才能在下一章中了解如何从服务器的角度处理请求。

## 理解 HTTP 的基础知识

HTTP 是一种无状态的客户端-服务器协议，在该协议中，客户端向服务器发起请求，服务器对客户端进行响应。HTTP 是一种应用层协议，作为 Web 上通信的基础。它使用 TCP 作为底层传输层协议。

本章假设您使用的是 HTTP 1.1 版本（HTTP/1.1）。我们还将介绍 HTTP 2.0 版本（HTTP/2）中引入的功能。幸运的是，Go 对这些协议之间的差异进行了抽象处理，因此我们可以使用相同的代码轻松地使用任意一个协议。

### 统一资源定位符

*URL*是客户端用来定位网页服务器并识别请求资源的一种地址。它由五个部分组成：一个必需的*scheme*，表示用于连接的协议，一个可选的*authority*，表示资源的权限，一个表示资源路径的*path*，一个可选的*query*，以及一个可选的*fragment*。冒号（:）后跟两个斜杠（//）将 scheme 和 authority 分开。authority 包括一个可选的由冒号分隔的用户名和密码，后接一个@符号，一个主机名，以及一个可选的端口号，端口号前面有冒号。path 是由斜杠分隔的一系列段。问号（?）表示 query 的开始，通常由键值对组成，键值对之间用&符号分隔。井号（#）前置 fragment，它是资源的子部分标识符。综合来看，URL 遵循如下模式：

```
`scheme`://`user`:`password`@`host`:`port`/`path`?`key1`=`value1`&`key2`=`value2`#`table_of_contents`
```

你在互联网上使用的典型 URL 至少包括一个 scheme 和一个 hostname。例如，如果你想查找地松鼠的图片，你可以通过在浏览器地址栏中输入以下 URL 来访问 Google 的图像搜索，然后在图像搜索标签中搜索*gophers*：

```
1https://2images.google.com3/
```

scheme 1 告知浏览器你希望使用 HTTPS 协议连接到地址*images.google.com*2，并且你希望获取默认资源 3。如果你指定了网页服务器的地址而没有明确指定资源，网页服务器将响应一个默认资源。就像大公司在你没有输入分机号时会将电话转接给接待员一样，网页服务器在你没有指定想要的资源时也会提供一个默认资源。谷歌接收到你的请求并返回图像搜索页面。当你在搜索框中输入*gophers*并提交表单时，浏览器会发送一个像这样的网址请求，以下是简化后的示例：

```
https://www.google.com/1search2?3q=gophers&tbm=isch . . .
```

这个 URL 向谷歌请求一个名为*search*1 的资源，并包含一个*query string*。query string 由问号 2 开始，包含由&符号分隔的参数，这些参数由网页服务器定义，并对其有意义。在这个例子中，`q`参数 3 的值是你的搜索查询，`gophers`。`tbm`参数的值是`isch`，告诉谷歌你正在进行图像搜索。谷歌定义了这些参数及其值，你将它们作为请求的一部分传递给谷歌的网页服务器。实际上，浏览器地址栏中的 URL 要长得多，并包含谷歌为了满足你的请求所需的其他详细信息。

如果我的妻子让我使用 HTTP 去购物，她可能会给我这样的 URL：

```
automobile://the.grocery.store/purchase?butter=irish&eggs=12&coffee=dark_roast
```

这告诉我，我应该开车去杂货店，买爱尔兰黄油、一打鸡蛋和深烘焙咖啡。值得提到的是，scheme 仅与使用它的上下文相关。我的网页浏览器不知道该如何处理*automobile*这个 scheme，但为了我的婚姻，我可得知道怎么做。

### 客户端资源请求

*HTTP 请求* 是从客户端发送到 Web 服务器的一条消息，要求服务器返回特定的资源。请求包括方法、目标资源、头部信息和正文。*方法* 告诉服务器你希望它如何处理目标资源。例如，`GET` 方法后跟 *robots.txt* 告诉服务器你希望它将 *robots.txt* 文件发给你，而 `DELETE` 方法则表示你希望服务器删除该资源。

*请求头部* 包含关于你发送的请求内容的元数据。例如，`Content-Length` 头部指定请求体的大小（以字节为单位）。*请求体* 是请求的有效负载。如果你上传一张新的个人资料图片到 Web 服务器，请求体将包含以适合通过网络传输的格式编码的图像，而 `Content-Length` 头部的值则会被设置为图像在请求体中的字节大小。并非所有的请求方法都需要请求体。

列表 8-1 详细介绍了通过 Netcat 向 Google 的 Web 服务器发送简单的 `GET` 请求以获取 *robots.txt* 文件的过程。《使用 Netcat 测试服务》在第 159 页会引导你安装 Netcat。

```
$ **nc www.google.com 80**
1**GET /robots.txt HTTP/1.1**

2 HTTP/1.1 200 OK
3 Accept-Ranges: none
Vary: Accept-Encoding
Content-Type: text/plain
Date: Mon, 02 Jan 2006 15:04:05 MST
Expires: Mon, 02 Jan 2006 15:04:05 MST
Cache-Control: private, max-age=0
Last-Modified: Mon, 02 Jan 2006 15:04:05 MST
X-Content-Type-Options: nosniff
Server: sffe
X-XSS-Protection: 0
Transfer-Encoding: chunked

4User-agent: *
Disallow: /search
Allow: /search/about
Allow: /search/static
Allow: /search/howsearchworks
`--snip--` 
```

列表 8-1：请求 Google 的 *robots.txt* 文件并接收其内容的响应

`GET` 请求 1 告诉 Google 的 Web 服务器，你想要通过 HTTP/1.1 获取 */robots.txt* 文件。请求之后，你按下回车键两次以发送请求，后面跟着一个空行。Web 服务器会迅速响应，返回状态行 2、一系列头部信息 3、一个空行用来分隔头部和响应体，最后是 *robots.txt* 文件的内容在响应体中 4。你将在本章稍后学习关于服务器响应的内容。

使用 Go 的 `net/http` 包，你可以只使用 HTTP 方法和 URL 来创建一个请求。`net/http` 包包括最常用的 RFC 7231 和 RFC 5789 请求方法的常量。这些 RFC 中包含了大量关于请求方法的术语。以下描述了如何在实践中使用这些方法：

1.  `GET` 就像之前的例子一样，`GET` 方法指示服务器将目标资源发送给你。服务器将在响应的正文中提供目标资源。需要注意的是，目标资源不一定是文件；响应也可以返回动态生成的内容，比如前面讨论的 *gophers* 图片搜索结果。服务器在收到 `GET` 请求后不应该修改或删除资源。

1.  `HEAD` `HEAD` 方法类似于 `GET`，不过它告诉服务器在响应中排除目标资源。服务器只会发送响应代码和存储在响应头中的各种元数据。你可以使用这种方法来检索有关资源的有意义的细节，例如资源的大小，以便决定是否要获取该资源。（该资源可能比你预期的要大。）

1.  `POST` `POST` 请求是一种将请求体中的数据上传到 Web 服务器的方式。`POST` 方法告诉服务器你正在发送数据，以便与目标资源关联。例如，你可以发布一个新的评论到新闻故事中，在这种情况下，新闻故事就是目标资源。简单来说，可以将 `POST` 方法视为在服务器上创建新资源的方法。

1.  `PUT` 与 `POST` 类似，你可以使用 `PUT` 请求将数据上传到 Web 服务器。实际上，`PUT` 方法通常用于更新或完全替换现有资源。你可以使用 `PUT` 编辑你已发布到新闻故事中的评论。

1.  `PATCH` `PATCH` 方法指定对现有资源的部分修改，其他部分保持不变。这样，它就像是一个*差异比较*。假设你正在为你生命中那个特别的人购买一只河狸毛绒玩具，并且已经完成了结账流程中的配送地址步骤，这时你发现自己在街道地址中打了个错别字。你跳回到配送地址表单并纠正了这个错误。现在，你可以再次使用 `POST` 提交表单并将所有内容发送到服务器。但 `PATCH` 请求会更高效，因为你只做了一个小修正。你很可能会在 API 中遇到 `PATCH` 方法，而不是在 HTML 表单中。

1.  `DELETE` `DELETE` 方法指示服务器删除目标资源。假设你在新闻故事中的评论太过争议性，现在邻居们避免和你进行眼神交流。你可以向服务器发送 `DELETE` 请求，删除你的评论并恢复你的社交地位。

1.  `OPTIONS` 你可以使用 `OPTIONS` 方法向服务器询问目标资源支持哪些方法。例如，你可以发送一个 `OPTIONS` 请求，目标资源是你的新闻评论，结果得知 `DELETE` 方法并不是服务器支持的操作，这意味着你现在最好的选择是另找地方住并结交新邻居。

1.  `CONNECT` 客户端使用 `CONNECT` 请求 Web 服务器执行*HTTP 隧道*，或者与目标地址建立 TCP 会话并在客户端和目标之间代理数据。

1.  `TRACE` `TRACE` 方法指示 Web 服务器将请求回显给你，而不是处理它。此方法允许你查看是否有任何中间节点在请求到达 Web 服务器之前修改了请求。

需要注意的是，Web 服务器没有义务实现这些请求方法。此外，您可能会发现一些 Web 服务器未能正确实现它们。相信，但要验证。

### 服务器响应

而客户端的请求始终指定一个方法和目标资源，Web 服务器的响应始终包括一个状态码，以通知客户端其请求的状态。成功的请求会导致响应中包含 200 类状态码。

如果客户端发出的请求需要客户端进一步操作，服务器将返回 300 类状态码。例如，如果客户端请求一个自上次请求以来没有变化的资源，服务器可能返回 304 状态码，通知客户端应该从缓存中渲染该资源。

如果由于客户端的请求发生错误，服务器将在响应中返回 400 类状态码。此情形中最常见的例子是客户端请求一个不存在的目标资源，在这种情况下，服务器将返回 404 状态码，通知客户端未能找到该资源。

500 类状态码通知客户端服务器端发生了错误，导致服务器无法完成请求。假设您的请求需要 Web 服务器从上游服务器获取资源以满足请求，但上游服务器未能响应。Web 服务器将返回 504 状态码，表示在与上游服务器的通信过程中发生了超时。

在 HTTP/1.1 中存在一些 100 类状态码，用于给客户端提供指示。例如，客户端在发送 `POST` 请求时可以向服务器请求指导。为此，客户端将发送 `POST` 方法、目标资源和请求头到服务器，其中之一告诉服务器客户端希望获得继续发送请求体的许可。服务器可以返回 100 状态码，表示客户端可以继续请求并发送请求体。

IANA 维护着 HTTP 状态码的官方列表，您可以在[`www.iana.org/assignments/http-status-codes/http-status-codes.xhtml`](https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml)找到它。如果您遇到一个相对不常见的状态码，通常可以在 RFC 7231 的[`tools.ietf.org/html/rfc7231#section-6`](https://tools.ietf.org/html/rfc7231#section-6)中找到对其的描述。

Go 定义了许多这样的状态码作为常量在其 `net/http` 包中，我建议您在代码中使用这些常量。读取 `http.StatusOK` 比记住 200 的含义要容易得多。您将遇到的最常见的 HTTP 状态码包括以下内容：

1.  `200 OK` 表示请求成功。如果请求方法是 `GET`，响应体包含目标资源。

1.  `201 Created` 当服务器成功处理请求并添加了一个新资源时返回，通常是在 `POST` 请求的情况下。

1.  `202 Accepted` 如果请求成功，但服务器尚未创建新资源时，通常会返回此状态码。尽管请求成功，资源的创建仍可能失败。

1.  `204 No Content` 如果请求成功但响应体为空，通常会返回此状态码。

1.  `304 Not Modified` 当客户端请求一个未更改的资源时返回。客户端应该使用其缓存的资源副本。一种缓存方法是使用实体标签（ETag）头部。当客户端向服务器请求资源时，响应中可能会包含一个可选的服务器派生的 ETag 头部，它对服务器有意义。如果客户端在未来再次请求相同的资源，可以将缓存的 ETag 头部及其值传递到请求中。服务器将检查客户端请求中的 ETag 值，以确定请求的资源是否已更改。如果未更改，服务器可能会返回 304 状态码和空的响应体。

1.  `400 Bad Request` 如果服务器因某种原因直接拒绝客户端的请求，则返回此状态码。这可能是由于请求格式不正确，例如请求方法中没有指定目标资源。

1.  `403 Forbidden` 如果服务器接受了你的请求，但确定你没有权限访问该资源，或者服务器本身没有权限访问请求的资源时，通常会返回此状态码。

1.  `404 Not Found` 如果你请求一个不存在的资源，服务器会返回此状态码。你也可能看到此状态码作为 *Glomar 响应*，当服务器不想确认或否认你是否有权限访问某个资源时。换句话说，网络服务器可能会对你没有权限访问的资源返回 404 状态码，而不是明确地返回 403 状态码确认你没有访问权限。试图访问你网络服务器上敏感资源的攻击者希望将精力集中在现有资源上，即使他们当前没有访问这些资源的权限。返回 404 状态码对于不存在和被禁止的资源来说是一种安全措施，防止攻击者区分这两者。这样做的缺点是，你更难调试服务器上的权限问题，因为你无法知道你请求的资源是否存在，或者你只是缺乏权限。我建议你在服务器日志中明确区分这两者。

1.  `405 Method Not Allowed` 如果你为目标资源指定了服务器不支持的请求方法，则返回此状态码。还记得你在我们讨论 `OPTIONS` 请求方法时试图删除的有争议评论吗？你会收到 405 状态码作为对该 `DELETE` 请求的响应。

1.  `426 升级要求` 返回此代码是为了指示客户端在请求目标资源之前，必须首先升级到 TLS。

1.  `500 服务器内部错误` 是一种通用的错误代码，当服务器发生错误，无法满足客户端的请求，但该错误不符合其他状态码的标准时，服务器会返回此错误代码。由于服务器端代码中的配置错误或语法错误，服务器经常返回 500 错误。如果你的服务器返回此代码，请检查日志。

1.  `502 错误网关` 当服务器在客户端与上游服务之间代理数据时，如果上游服务不可用且不接受请求，服务器将返回此错误。

1.  `503 服务不可用` 如果网络服务器无法接受请求，则会返回此代码。例如，当网络服务器进入维护模式时，它可能会为所有传入连接返回 503 状态码。

1.  `504 网关超时` 由代理服务器返回，表示上游服务已接受请求，但未能及时提供回复。

### 从请求到渲染页面

一个网页通常由各种资源组成，如图像、视频、网页浏览器的布局指令、第三方广告等。访问每个资源需要向服务器发起单独的请求。在 HTTP 1.0（HTTP/1.0）版本中，客户端必须为每个请求单独发起 TCP 连接。HTTP/1.1 消除了这一要求，减少了与多个 HTTP 请求同一服务器的请求-连接开销和延迟。相反，它允许通过同一个 TCP 连接发送多个请求和响应。（所有现代的 Web 服务器软件和网页浏览器至少支持 HTTP/1.1，因此你不太可能使用 HTTP/1.0。）

表 8-1 演示了检索 HTML 文档以及随后对该文档中所有指定资源的`GET`调用。

表 8-1：请求索引 HTML 文档后检索附加资源

| 状态 | 方法 | 域名 | 资源 | 类型 | 传输字节 | 传输时长 |
| --- | --- | --- | --- | --- | --- | --- |
| 200 | GET | *woodbeck.net* | */* | HTML | 1.83KB | 49 毫秒 |
| 200 | GET | *woodbeck.net* | *main.min.css* | CSS | 1.30KB | 20 毫秒 |
| 200 | GET | *woodbeck.net* | *code.css* | CSS | 0.99KB | 20 毫秒 |
| 304 | GET | *woodbeck.net* | *avatar.jpeg* | JPEG | 0 字节 | 0 毫秒 |
| 404 | GET | *woodbeck.net* | *favicon.ico* | IMG | 0 字节 | 0 毫秒 |

对 [`woodbeck.net/`](https://woodbeck.net/) 的初始 `GET` 请求成功地检索了由默认资源指定的 HTML 文档。这个 HTML 文档包含了渲染页面所需的附加资源的链接，因此 web 浏览器也请求了这些资源。由于这次传输使用了 HTTP/1.1，web 浏览器使用相同的 TCP 连接来检索剩余的资源。由于该资源自上次 web 浏览器接收以来没有变化，web 服务器指示浏览器使用其缓存的 *avatar.jpeg* 副本。由于 web 服务器未能找到 *favicon.ico* 文件，因此返回了 404 状态码给浏览器。

HTTP 的最新版本 HTTP/2 旨在进一步减少延迟。除了对后续请求重用相同的 TCP 连接外，HTTP/2 服务器还可以主动将资源推送到客户端。如果 表 8-1 中的对话是通过 HTTP/2 进行的，它可能是这样发生的。客户端请求了默认资源，服务器响应了默认资源。但由于服务器知道默认资源有依赖的资源，它*推送*这些资源到客户端，而无需客户端为每个资源单独发起请求。

Go 的 HTTP 客户端和服务器透明地支持 HTTP/1.0、HTTP/1.1 和 HTTP/2，这意味着你可以编写代码来检索和提供资源，同时让 Go 的 `net/http` 包中的代码来协商最佳的 HTTP 版本。然而，尽管 Go 的 HTTP/2 服务器实现可以将资源推送到客户端，但 Go 的 HTTP/2 客户端实现目前还无法接收这些服务器推送。

## 在 Go 中检索 Web 资源

就像你的 web 浏览器一样，Go 可以使用 `net/http` 包的 HTTP 客户端与 web 服务器进行通信。与 web 浏览器不同，Go 不会直接将 HTML 页面渲染到屏幕上。相反，你可以使用 Go 来从网站抓取数据（如财务股票详情）、提交表单数据，或与使用 HTTP 作为应用协议的 API 进行交互，举几个例子。

尽管在 Go 中发起 HTTP 请求很简单，但你需要处理一些客户端的陷阱。你将在接下来的内容中学习这些陷阱。首先，让我们看一个简单的示例请求。

### 使用 Go 的默认 HTTP 客户端

`net/http` 包包括一个默认客户端，允许你发起一次性的 HTTP 请求。例如，你可以使用 `http.Get` 函数向指定的 URL 发送 `GET` 请求。

列表 8-2 演示了你可以从受信任的机构 —— *time.gov* 的 web 服务器检索当前时间，并与计算机本地时间进行比较的一种方式。这将让你大致了解计算机本地时间的提前或滞后情况。你当然不想依赖这种方法来进行任何形式的取证，但这个示例用来展示通过 `HEAD` 请求和响应来演示 Go HTTP 客户端工作流。

```
package main

import (
    "net/http"
    "testing"
    "time"
)

func TestHeadTime(t *testing.T) {
    resp, err := 1http.Head("https://www.time.gov/")
    if err != nil {
        t.Fatal(err)
    }
    _ = 2resp.Body.Close() // Always close this without exception.

    now := time.Now().Round(time.Second)
    date := 3resp.Header.Get("Date")
    if date == "" {
        t.Fatal("no Date header received from time.gov")
    }

    dt, err := time.Parse(time.RFC1123, date)
    if err != nil {
        t.Fatal(err)
    }

    t.Logf("time.gov: %s (skew %s)", dt, now.Sub(dt))
}
```

示例 8-2：从 *time.gov* 获取时间戳（*time_test.go*）

`net/http` 包包含一些辅助函数，用于发起 `GET`、`HEAD` 或 `POST` 请求。在这里，我们使用 `http.Get` 函数 1 来访问 [`www.time.gov/`](https://www.time.gov/) 以获取默认资源。Go 的 HTTP 客户端会自动为你升级到 HTTPS，因为这是 URL 协议中指明的协议。虽然你没有读取响应体的内容，但你必须关闭它 2。下一节将讨论为何在每种情况下都需要关闭响应体。

现在你有了响应，你可以获取 `Date` 头部 3，这个头部指示了服务器生成响应的时间。你可以使用这个值来计算你计算机的时钟偏差。当然，由于服务器生成头部和你的代码处理它之间存在延迟，并且 `Date` 头部本身缺乏纳秒级的精度，你会失去一些准确性。

### 关闭响应体

如前所述，HTTP/1.1 允许客户端与服务器保持一个 TCP 连接，进行多个 HTTP 请求（我们称之为 *keepalive 支持*）。即便如此，当先前响应中的未读取字节仍然在网络上时，客户端无法重用 TCP 会话。Go 的 HTTP 客户端在你关闭响应体时会自动清空响应体，这使得你的代码可以重用底层的 TCP 会话，只要你始终如一地关闭每一个响应体。

让我们回顾一下来自示例 8-1 的响应，看看 Go 是如何解析响应的（示例 8-3）。

```
1HTTP/1.1 200 OK
Accept-Ranges: none
Vary: Accept-Encoding
Content-Type: text/plain
Date: Mon, 02 Jan 2006 15:04:05 MST
Expires: Mon, 02 Jan 2006 15:04:05 MST
Cache-Control: private, max-age=0
Last-Modified: Mon, 02 Jan 2006 15:04:05 MST
X-Content-Type-Options: nosniff
Server: sffe
X-XSS-Protection: 0
Transfer-Encoding: chunked

2User-agent: *
Disallow: /search
Allow: /search/about
Allow: /search/static
Allow: /search/howsearchworks
`--snip--` 
```

示例 8-3：解析 HTTP 响应

Go 的 HTTP 客户端会从网络套接字中读取响应的状态和头部 1，这些数据会立即作为响应对象的一部分提供给你的代码。然而，客户端并不会自动读取响应体 2。直到你显式地读取它，或者直到你关闭它并且 Go 隐式地清空所有未读取的字节，响应体才会被处理。

Go HTTP 客户端在关闭时隐式地清空响应体，这可能会对你造成困扰。例如，假设你发送了一个 `GET` 请求请求一个文件，并且收到了来自服务器的响应。你读取了响应中的 `Content-Length` 头部，发现文件比你预期的要大得多。如果你在没有读取任何字节的情况下关闭响应体，Go 会在清空响应体时从服务器下载整个文件。

更好的替代方法是发送一个 `HEAD` 请求来获取 `Content-Length` 头部。这样，响应体中就不会有未读取的字节，因此关闭响应体时不会产生任何额外的开销。你在示例 8-2 中已正确关闭了响应体，因此如果以后你发出更多请求，Go 的 HTTP 客户端可以重用 TCP 会话。

在极少数情况下，如果你进行 HTTP 请求并且希望显式地清空响应体，最有效的方法是使用 `io.Copy` 函数：

```
_, _ = io.Copy(ioutil.Discard, response.Body) 
_ = response.Close()
```

`io.Copy` 函数通过从 `response.Body` 中读取所有字节并将这些字节写入 `ioutil.Discard` 来清空 `response.Body`。顾名思义，`ioutil.Discard` 是一个特殊的 `io.Writer`，会丢弃所有写入它的字节。

你不必忽略 `io.Copy` 和 `response.Close` 的返回值，但这样做可以让其他开发人员知道你故意忽略了这些值。一些开发人员可能会认为这是多余的冗长，的确，在这种情况下，`io.Copy` 或 `response.Close` 很少会返回错误，但这仍然是一个好习惯。我曾遇到过代码在隐式忽略错误，可能是出于习惯，而开发人员本应处理这些错误。

底线是，无论是否读取响应体，都必须关闭它，以避免资源泄漏。

### 实现超时和取消

Go 的默认 HTTP 客户端以及通过 `http.Get`、`http.Head` 和 `http.Post` 辅助函数创建的请求不会超时。这个问题的后果可能不会立刻显现，直到你遇到以下事实（从此你将永远记住）：没有超时或截止日期意味着一个表现不正常或恶意的服务可能会导致你的代码无限期阻塞，并且不会产生错误来表明出了问题。你可能直到用户开始投诉时才会发现你的服务出了问题。

例如，示例 8-4 展示了一个简单的测试，导致 HTTP 客户端无限期阻塞。

```
package main

import (
    "context"
    "errors"
    "net/http"
    "net/http/httptest"
    "testing"
    "time"
)

func blockIndefinitely(w http.ResponseWriter, r *http.Request) {
    select {}
}

func TestBlockIndefinitely(t *testing.T) {
    ts := 1httptest.NewServer(2http.HandlerFunc(3blockIndefinitely))
    _, _ = http.Get(4ts.URL)
    t.Fatal("client did not indefinitely block")
}
```

示例 8-4：测试服务器使默认的 HTTP 客户端无限期阻塞 (*block_test.go*)。

`net/http/httptest` 包包含一个有用的 HTTP 测试服务器。`httptest.NewServer`1 函数接受一个 `http.HandlerFunc`2，后者又包装了 `blockIndefinitely` 函数 3。测试服务器将它接收到的任何请求通过其 URL 4 传递给 `http.HandlerFunc` 的 `ServeHTTP` 方法。该方法将请求和响应对象传递给 `blockIndefinitely` 函数，在该函数中控制将无限期阻塞。

因为辅助函数 `http.Get` 使用默认的 HTTP 客户端，所以这个 `GET` 请求不会超时。相反，`go test` 运行器最终会超时并停止测试，打印堆栈跟踪。

为了解决这个问题，生产代码应该使用你在第 57 页“使用带有截止日期的上下文来超时连接”中学到的技术。创建一个上下文，并用它初始化一个新的请求。然后，你可以通过使用上下文的 `cancel` 函数，或者创建一个带有截止日期或超时的上下文，手动取消该请求。

让我们通过将 示例 8-4 中的测试替换为 示例 8-5 来修复测试。新的测试将在五秒钟内没有收到服务器响应时超时。

```
`--snip--`

func TestBlockIndefinitelyWithTimeout(t *testing.T) {
    ts := httptest.NewServer(http.HandlerFunc(blockIndefinitely))

    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    req, err := 1http.NewRequestWithContext(ctx, http.MethodGet, ts.URL, nil)
    if err != nil {
        t.Fatal(err)
    }

    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        if !errors.Is(err, context.DeadlineExceeded) {
            t.Fatal(err)
        }
        return
    }
    _ = resp.Body.Close()
}
```

示例 8-5：为 GET 请求添加超时 (*block_test.go*)

首先，你通过传入上下文、请求方法、URL 和一个 `nil` 请求体来创建一个新的请求 1，因为你的请求没有负载。请记住，上下文的计时器在你初始化上下文时就开始运行。上下文控制着请求的整个生命周期。换句话说，客户端有五秒钟的时间连接到 Web 服务器，发送请求，读取响应头，并将响应传递给你的代码。接下来，你有剩余的五秒钟时间来读取响应体。如果你正在读取响应体时上下文超时，你的下一个读取操作会立即返回错误。因此，请为你的特定应用程序使用宽松的超时值。

或者，创建一个没有超时或截止时间的上下文，并仅通过使用计时器和上下文的 `cancel` 函数来控制上下文的取消，如下所示：

```
 ctx, cancel := context.WithCancel(context.Background())
    timer := time.AfterFunc(5*time.Second, 1cancel)
    // Make the HTTP request, read the response headers, etc.
    // ...
    // Add 5 more seconds before reading the response body.
    timer.Reset(5*time.Second)
```

这段代码演示了如何使用一个计时器，该计时器将在超时后调用上下文的 `cancel` 函数 1。你可以根据需要重置计时器，将 `cancel` 调用推迟到更远的未来。

### 禁用持久化 TCP 连接

默认情况下，Go 的 HTTP 客户端在读取完响应后会保持与 Web 服务器的底层 TCP 连接，除非服务器明确要求断开连接。虽然这在大多数使用场景中是期望的行为，因为它允许你在多个请求中复用同一个 TCP 连接，但你可能会不小心让计算机无法与其他 Web 服务器建立新的 TCP 连接。

这是因为计算机能够保持的活动 TCP 连接数是有限的。如果你编写了一个向多个 Web 服务器发送一次性请求的程序，你可能会发现，当所有计算机的 TCP 连接耗尽后，程序停止工作，无法打开新的连接。在这种情况下，TCP 会话重用可能会对你不利。与其在客户端禁用 TCP 会话重用，不如更灵活的做法是按请求告知客户端如何处理 TCP 套接字。

```
`--snip--`
    req, err := http.NewRequestWithContext(ctx, http.MethodGet, ts.URL, nil)
    if err != nil {
        t.Fatal(err)
    }
  1req.Close = true 
`--snip--`
```

将请求的 `Close` 字段 1 设置为 `true` 告诉 Go 的 HTTP 客户端，在读取完 Web 服务器的响应后，它应该关闭底层的 TCP 连接。如果你知道你将发送四个请求到 Web 服务器且不会再发送更多请求，你可以在第四个请求上将 `Close` 字段设置为 `true`。所有四个请求将使用同一个 TCP 会话，并且客户端将在接收到第四个响应后终止 TCP 连接。

## 通过 HTTP 发送数据

发送`POST`请求及其负载到 Web 服务器，就像你之前发出的请求一样。不同之处在于，请求体包含负载。这个负载可以是任何实现了`io.Reader`接口的对象，包括文件句柄、标准输入、HTTP 响应体或 Unix 域套接字等。但正如你将看到的，向 Web 服务器发送数据涉及比`GET`请求更多的代码，因为你必须准备请求体。

### 向 Web 服务器发布 JSON

在你能向测试服务器发送数据之前，你需要创建一个能够接收数据的处理器。列表 8-6 创建了一个名为`User`的新类型，你将其编码为 JavaScript 对象表示法（JSON）并发布到该处理器。

```
package main

import (
    "bytes"
    "context"
    "encoding/json"
    "fmt"
    "io"
    "io/ioutil"
    "mime/multipart"
    "net/http"
    "net/http/httptest"
    "os"
    "path/filepath"
    "testing"
    "time"
)

type User struct {
    First string
    Last  string
}

1 func handlePostUser(t *testing.T) func(http.ResponseWriter, *http.Request) {
    return func(w http.ResponseWriter, r *http.Request) {
        defer func(r io.ReadCloser) {
            _, _ = 2io.Copy(ioutil.Discard, r)
            _ = r.Close()
        }(r.Body)

        if r.Method != 3http.MethodPost {
          4http.Error(w, "", http.StatusMethodNotAllowed)
            return
        }

        var u User
        err := json.NewDecoder(r.Body).Decode(&u)
 if err != nil {
            t.Error(err)
            http.Error(w, "Decode Failed", http.StatusBadRequest)
            return
        }

      5w.WriteHeader(http.StatusAccepted) 
    }
}
```

列表 8-6：一个可以将 JSON 解码为`User`对象的处理器（*post_test.go*）

`handlePostUser`函数 1 返回一个处理`POST`请求的函数。如果请求方法是除`POST`外的其他任何方法 3，它将返回一个状态码，表示服务器不允许该方法 4。该函数然后尝试将请求体中的 JSON 解码为`User`对象。如果解码成功，响应的状态将设置为*Accepted*5。

与 Go HTTP 客户端不同，Go HTTP 服务器必须显式地在关闭请求体之前先排空它 2。我们将在第九章中详细讨论这个问题。

列表 8-7 中的测试将`User`对象编码为 JSON 并通过`POST`请求发送到测试服务器。

```
`--snip--`

func TestPostUser(t *testing.T) {
    ts := httptest.NewServer(http.HandlerFunc(handlePostUser(t)))
    defer ts.Close()

    resp, err := http.Get(ts.URL)
    if err != nil {
        t.Fatal(err)
    }
    if 1resp.StatusCode != http.StatusMethodNotAllowed {
        t.Fatalf("expected status %d; actual status %d",
            http.StatusMethodNotAllowed, resp.StatusCode)
    }

    buf := new(bytes.Buffer)
    u := User{First: "Adam", Last: "Woodbeck"}
 2 err = json.NewEncoder(buf).Encode(&u)
    if err != nil {
        t.Fatal(err)
    }

    resp, err = 3http.Post(ts.URL, "application/json", buf)
    if err != nil {
        t.Fatal(err)
    }
    if resp.StatusCode != 4http.StatusAccepted {
        t.Fatalf("expected status %d; actual status %d",
            http.StatusAccepted, resp.StatusCode)
 }
    _ = resp.Body.Close()
}
```

列表 8-7：将`User`对象编码为`JSON`并发布到测试服务器（*post_test.go*）

测试首先确保如果客户端发送错误类型的请求 1，测试服务器的处理器能正确响应错误。如果测试服务器收到非`POST`请求，它会响应方法不被允许错误。然后，测试会将`User`对象编码为 JSON，并将数据写入字节缓冲区 2。它向测试服务器的 URL 发送`POST`请求，并将内容类型设置为*application/json*，因为字节缓冲区（表示请求体）包含 JSON3。内容类型告知服务器的处理器请求体中预计的数据类型。如果服务器的处理器正确解码了请求体，响应的状态码将是 202 Accepted4。

### 发布带有附件文件的 Multipart 表单

向 Web 服务器发布 JSON 很简单。只需设置适当的内容类型并将 JSON 负载放入请求体中。但如何在单个`POST`请求中向 Web 服务器发送多个数据块呢？答案是：使用`mime/multipart`包。

`mime/multipart`包允许你构建多部分*多用途互联网邮件扩展（MIME）消息*，通过一个称为*边界*的字符串将你想发送的每个数据块与其他数据块分开。稍后你将在本节中看到边界的示例，尽管这通常不是你需要担心的事情。

每个 MIME 部分都包括描述内容的可选头部，以及包含实际内容的主体。例如，如果 Web 服务器解析了一个`Content-Type`头部设置为*text/plain*的 MIME 部分，它将把该部分的主体视为纯文本。

Listing 8-8 介绍了一个新的测试，带领你通过使用`mime/multipart`包构建 multipart 请求体的过程。

```
`--snip--`

func TestMultipartPost(t *testing.T) {
    reqBody := 1new(bytes.Buffer)
    w := 2multipart.NewWriter(reqBody)

    for k, v := range map[string]string{
        "date":        time.Now().Format(time.RFC3339),
        "description": "Form values with attached files",
    } {
        err := 3w.WriteField(k, v)
        if err != nil {
            t.Fatal(err)
        }
    }
```

Listing 8-8：创建一个新的请求体、multipart writer 并写入表单数据（*post_test.go*）

首先，你创建一个新的缓冲区 1 作为请求体。然后你创建一个新的 multipart writer 2，将其包装到缓冲区中。multipart writer 在初始化时会生成一个随机的边界。最后，你将表单字段写入 multipart writer 3。multipart writer 将每个表单字段分隔到独立的部分中，并为每个部分的主体写入边界、适当的头部和表单字段值。

到这时，你的请求体有两个部分，一个是*date*表单字段，一个是*description*表单字段。接下来，让我们在 Listing 8-9 中附加一些文件。

```
`--snip--`

    for i, file := range []string{
        "./files/hello.txt",
        "./files/goodbye.txt",
    } {
        filePart, err := 1w.CreateFormFile(fmt.Sprintf("file%d", i+1),
            filepath.Base(file))
        if err != nil {
            t.Fatal(err)
        }

        f, err := os.Open(file)
        if err != nil {
            t.Fatal(err)
        }

        _, err = 2io.Copy(filePart, f)
        _ = f.Close()
        if err != nil {
            t.Fatal(err)
        }
    }

    err := 3w.Close()
    if err != nil {
        t.Fatal(err)
    }
```

Listing 8-9：将两个文件写入请求体，每个文件独立为一个 MIME 部分（*post_test.go*）

将字段附加到请求体中并不像添加表单字段数据那样直接。你需要额外的步骤。首先，你需要从 Listing 8-8 中的 multipart writer 1 创建一个 multipart 部分写入器。`CreateFormField`方法接受字段名和文件名。服务器在解析 MIME 部分时使用这个文件名，它不需要与附加的文件名匹配。现在，你只需打开文件并将其内容复制到 MIME 部分写入器 2 中。

当你完成向请求体中添加各个部分后，必须关闭 multipart writer 3，这将通过附加边界来最终确定请求体。

Listing 8-10 将请求发布到一个知名的测试服务器*httpbin.org*。

```
`--snip--`

    ctx, cancel := context.WithTimeout(context.Background(),
        60*time.Second)
    defer cancel()

    req, err := http.NewRequestWithContext(ctx, http.MethodPost,
      1"https://httpbin.org/post", 2reqBody)
    if err != nil {
        t.Fatal(err)
    }
    req.Header.Set("Content-Type", 3w.FormDataContentType())

    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        t.Fatal(err)
    }
    defer func() { _ = resp.Body.Close() }()

    b, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        t.Fatal(err)
    }
    if resp.StatusCode != http.StatusOK {
        t.Fatalf("expected status %d; actual status %d",
            http.StatusOK, resp.StatusCode)
    }

    t.Logf("\n%s", b)
}
```

Listing 8-10：使用 Go 的默认 HTTP 客户端发送`POST`请求到*httpbin.org*（*post_test.go*）

首先，你创建一个新的请求，并传递一个将在 60 秒后超时的上下文。由于你是通过互联网发起这个请求，所以不像在本地测试时那样有那么高的确定性，无法确保请求会顺利到达目标地址。这个`POST`请求的目标是[`www.httpbin.org/`](https://www.httpbin.org/)1，并将在其有效载荷中发送 multipart 请求体 2。

在发送请求之前，你需要设置`Content-Type`头部，告知 Web 服务器你正在发送多个部分。multipart writer 的`FormDataContentType`方法 3 返回适当的`Content-Type`值，其中包含边界。Web 服务器使用这个头部中的边界来确定请求体的每一部分的起始和结束。

一旦你使用`-v`标志运行测试，你应该会看到像 Listing 8-11 中的 JSON 输出。

```
{
  "args": {},
  "data": "",
1"files": {
    "file1": "Hello, world!\n",
    "file2": "Goodbye, world!\n"
  },
2"form": {
    "date": "2006-01-02T15:04:05-07:00",
    "description": "Form fields with attached files"
  },
  "headers": {
    "Accept-Encoding": "gzip",
    "Content-Length": "739",
 3 "Content-Type": "multipart/form-data; boundary=e9ad4b62e0dfc8d7dc57ccfa8ba62244342f1884608e6d88018f9de8abcb",
    "Host": "httpbin.org",
    "User-Agent": "Go-http-client/1.1"
  },
  "json": null,
  "origin": "192.168.0.1",
  "url": "https://httpbin.org/post"
}
```

Listing 8-11：multipart `POST`请求的响应体

这是 *httpbin.org* 的标准 `POST` 响应，包含一些与你发送的请求无关的字段。但是，如果你查看，你会看到你附加的每个文本文件的内容 1 和你提交的表单字段 2。你还可以看到 multipart 写入器添加的 `Content-Type` 头 3。请注意，边界是一个随机字符串。在你的代码中，每次请求时，边界都会随机变化。但如果你愿意，可以通过使用 multipart 写入器的 `SetBoundary` 方法来设置一个边界。

## 你所学到的内容

HTTP 允许客户端向服务器发送请求，并从服务器接收资源，通过全球信息网进行通信。本章向你展示了如何使用 Go 构建 HTTP 请求。目标资源可以是网页、图片、视频、文档、文件、游戏等。为了检索资源，HTTP 客户端向 Web 服务器发送带有 URL 的 `GET` 请求。Web 服务器使用该 URL 定位正确的资源，并将其发送到客户端的响应中。客户端始终发起这个 HTTP 请求-响应流程。

客户端可以向服务器发送多种类型的资源请求。最常用的请求方法有 `GET`、`HEAD`、`POST`、`PUT` 和 `DELETE`。`GET` 请求要求服务器检索指定的资源。客户端可以发送 `HEAD` 请求，以获取响应头部，而不包含请求的负载。这对于确定资源是否存在，以及在检索资源之前检查响应头部非常有用。`POST` 请求允许客户端向服务器发送资源，而 `PUT` 请求通常用于更新服务器上的现有资源。客户端可以通过发送 `DELETE` 请求来请求服务器删除某个资源。

`net/http` 包提供了所有必要的类型和函数，以便通过 HTTP 与服务器进行交互。它包括一个默认的 HTTP 客户端，允许你快速发起一次性 HTTP 请求并接收响应。然而，无论你是否读取响应体的内容，你都必须认真关闭响应体，以防止资源泄露。还要注意，默认的 HTTP 客户端以及通过 `http.Get`、`http.Head` 和 `http.Post` 等助手函数发送的请求不会超时。这意味着，某个行为异常或恶意的服务可能会导致你的代码无限期阻塞。因此，重要的是通过使用上下文来自己管理请求的取消。

`mime/multipart` 包允许你轻松地向请求体中添加多个 MIME 部分。你可以高效地构建上传文件和表单内容到 Web 服务器的请求。
