# 第十二章

12

自动化 Arachni

![](img/00010.jpg)

Arachni 是一个强大的 Web 应用程序黑盒安全扫描工具，使用 Ruby 编写。它支持多种 Web 应用程序漏洞的检测，包括许多 OWASP 十大漏洞（如 XSS 和 SQL 注入）；具有高度可扩展的分布式架构，可以动态启动集群中的扫描器；并通过远程过程调用（RPC）接口和表现性状态转移（REST）接口实现完全自动化。在本章中，你将学习如何使用 Arachni 的 REST API，然后使用其 RPC 接口扫描给定 URL 中的 Web 应用程序漏洞。

安装 Arachni

Arachni 网站（[`www.arachni-scanner.com/`](http://www.arachni-scanner.com/)）提供了适用于多个操作系统的 Arachni 下载包。你可以使用这些安装程序在自己的系统上安装 Arachni。下载后，你可以通过运行 Arachni 来测试针对 Web 漏洞的服务器，正如 Listing 12-1 中所示。虽然此命令尚未使用 RPC 来驱动 Arachni，但你可以看到在扫描潜在的 XSS 或 SQL 注入漏洞时会得到什么样的输出。

> $ arachni --checks xss*,sql* --scope-auto-redundant 2 \
> 
> "http://demo.testfire.net/default.aspx"

Listing 12-1: 使用 Arachni 扫描一个故意易受攻击的网站

此命令使用 Arachni 检查网站 [`demo.testfire.net/default.aspx`](http://demo.testfire.net/default.aspx) 中的 XSS 和 SQL 漏洞。我们通过设置 --scope-auto-redundant 为 2 限制其跟踪的页面范围。这样，Arachni 会在继续扫描新 URL 之前，最多跟踪带有相同参数但不同参数值的 URL 两次。当有很多带有相同参数的链接指向同一页面时，Arachni 扫描的速度会更快。

注意

> 要全面了解 Arachni 中支持的漏洞检测及相关文档，请访问 Arachni 的 GitHub 页面，其中详细介绍了命令行参数：[`www.github.com/Arachni/arachni/wiki/Command-line-user-interface#checks/`](https://www.github.com/Arachni/arachni/wiki/Command-line-user-interface#checks/)。

在几分钟内（取决于你的互联网速度），Arachni 应该会报告该网站中一些 XSS 和 SQL 注入漏洞。别担心——它们是故意存在的！这个网站是专门设计为易受攻击的。稍后，在测试我们的自定义 C# 自动化时，你可以使用这个 XSS、SQL 注入和其他漏洞的列表，确保你的自动化程序返回正确的结果。

假设你想将 Arachni 自动运行在你的 web 应用的任意版本上，作为安全软件开发生命周期（SDLC）的一部分。手动运行并不高效，但我们可以轻松地自动化 Arachni，以便启动扫描任务，这样它就可以与任何持续集成系统配合使用，根据扫描结果来决定构建是否通过或失败。这就是 REST API 的作用所在。

Arachni REST API

最近，Arachni 引入了一个 REST API，可以通过简单的 HTTP 请求来驱动 Arachni。列表 12-2 展示了如何启动这个 API。

> $ arachni_rest_server
> 
> Arachni - 网络应用安全扫描框架 v2.0dev
> 
> 作者: Tasos "Zapotek" Laskos <tasos.laskos@arachni-scanner.com>
> 
> （在社区和 Arachni 团队的支持下。）
> 
> 网站: http://arachni-scanner.com
> 
> 文档: http://arachni-scanner.com/wiki
> 
> ➊[*] 正在监听 http://127.0.0.1:7331

列表 12-2: 运行 Arachni REST 服务器

当你启动服务器时，Arachni 会输出一些关于它的信息，包括它监听的 IP 地址和端口 ➊。一旦你确认服务器工作正常，就可以开始使用 API。

通过 REST API，你可以使用任何常见的 HTTP 工具，如 curl 或 netcat，启动一个简单的扫描。在本书中，我们将继续使用 curl，和之前的章节一样。我们的第一次扫描如 列表 12-3 所示。

> $ curl -X POST --data '{"url":"http://demo.testfire.net/default.aspx"}'➊ \
> 
> http://127.0.0.1:7331/scans
> 
> {"id":"b139f787f2d59800fc97c34c48863bed"}➋
> 
> $ curl http://127.0.0.1:7331/scans/b139f787f2d59800fc97c34c48863bed➌
> 
> {"status":"done","busy":false,"seed":"676fc9ded9dc44b8a32154d1458e20de",
> 
> --省略--

列表 12-3: 使用 curl 测试 REST API

要启动扫描，我们需要做的就是发送一个带有 JSON 数据的 POST 请求 ➊。我们通过 curl 的 --data 参数传递包含扫描 URL 的 JSON，发送到 /scans 端点，从而启动一个新的 Arachni 扫描。新扫描的 ID 会在 HTTP 响应中返回 ➋。创建扫描后，我们还可以通过一个简单的 HTTP GET 请求（curl 的默认请求类型）检索当前扫描的状态和结果 ➌。我们通过调用 Arachni 所监听的 IP 地址和端口，并附加在创建扫描时获得的 ID，将其添加到 /scans/ URL 端点来完成这个请求。扫描完成后，扫描日志将包含扫描过程中发现的任何漏洞，如 XSS、SQL 注入和其他常见的 web 应用漏洞。

完成此操作后，我们就能了解 REST API 的工作原理，然后可以开始编写代码，使我们能够使用 API 扫描任何有地址的网站。

创建 ArachniHTTPSession 类

正如之前章节所述，我们将实现一个会话类和一个管理器类，以便与 Arachni API 进行交互。目前，这些类相对简单，但现在将它们拆开可以在未来如果 API 需要身份验证或额外步骤时提供更大的灵活性。 Listing 12-4 详细说明了 ArachniHTTPSession 类。

> public class ArachniHTTPSession
> 
> {
> 
> public ➊ArachniHTTPSession(string host, int port)
> 
> {
> 
> this.Host = host;
> 
> this.Port = port;
> 
> }
> 
> public string Host { get; set; }
> 
> public int Port { get; set; }
> 
> public JObject ➋ExecuteRequest(string method, string uri, JObject data = null)
> 
> {
> 
> string url = "http://" + this.Host + ":" + this.Port.ToString() + uri;
> 
> HttpWebRequest request = (HttpWebRequest)WebRequest.Create(url);
> 
> request.Method = method;
> 
> if (data != null)
> 
> {
> 
> string dataString = data.ToString();
> 
> byte[] dataBytes = System.Text.Encoding.UTF8.GetBytes(dataString);
> 
> request.ContentType = "application/json";
> 
> request.ContentLength = dataBytes.Length;
> 
> request.GetRequestStream().Write(dataBytes, 0, dataBytes.Length);
> 
> }
> 
> string resp = string.Empty;
> 
> using (StreamReader reader = new StreamReader(request.GetResponse().GetResponseStream()))
> 
> resp = reader.ReadToEnd();
> 
> return JObject.Parse(resp);
> 
> }
> 
> }

Listing 12-4: ArachniHTTPSession 类

在本书的这一部分，ArachniHTTPSession 类应该比较容易理解，因此我们不会过于深入代码。我们创建了一个构造函数 ➊，该构造函数接受两个参数——要连接的主机和端口，并将这些值分配给相应的属性。然后，我们创建一个方法来执行基于传递给该方法的参数的通用 HTTP 请求 ➋。ExecuteRequest() 方法应该返回一个 JObject，其中包含由给定 API 端点返回的任何数据。由于 ExecuteRequest() 方法可以用于对 Arachni 发起任何 API 调用，我们唯一可以预期的是响应将是 JSON，可以从服务器的响应解析为 JObject。

创建 ArachniHTTPManager 类

ArachniHTTPManager 类此时应该看起来非常简单，如 Listing 12-5 所示。

> public class ArachniHTTPManager
> 
> {
> 
> ArachniHTTPSession _session;
> 
> public ➊ArachniHTTPManager(ArachniHTTPSession session)
> 
> {
> 
> _session = session;
> 
> }
> 
> public JObject ➋StartScan(string url, JObject options = ➌null)
> 
> {
> 
> JObject data = new JObject();
> 
> data["url"] = url;
> 
> data.Merge(options);
> 
> return _session.ExecuteRequest("POST", "/scans", data);
> 
> }
> 
> public JObject ➍GetScanStatus(Guid id)
> 
> {
> 
> return _session.ExecuteRequest("GET", "/scans/" + id.ToString("N"));
> 
> }
> 
> }

Listing 12-5: ArachniHTTPManager 类

我们的 ArachniHTTPManager 构造函数 ➊ 接受一个参数——用于执行请求的会话，并将该会话分配给本地私有变量，以便稍后使用。然后我们创建了两个方法：StartScan() ➋ 和 GetScanStatus() ➍。这些方法是我们创建一个扫描并报告 URL 的小工具所需的一切。

StartScan()方法接受两个参数，其中一个是可选的，默认值为 null ➌。默认情况下，您只需指定一个 URL 而不传入扫描选项，StartScan()方法会让 Arachni 仅爬取站点而不检查漏洞——这一特性可以帮助您了解 Web 应用程序的表面面积（即有多少页面和表单需要测试）。然而，我们实际上希望指定额外的参数来调整 Arachni 扫描，因此我们将这些选项合并到我们的数据 JObject 中，然后将扫描详情 POST 到 Arachni API 并返回 JSON 响应。GetScanStatus()方法通过简单的 GET 请求，使用扫描 ID 作为 URL 中的 API 参数，并返回 JSON 响应给调用者。

将 Session 和 Manager 类结合起来

在实现了这两个类后，我们就可以开始扫描了，正如 Listing 12-6 所示。

> public static void Main(string[] args)
> 
> {
> 
> ArachniHTTPSession session = new ArachniHTTPSession("127.0.0.1", 7331);
> 
> ArachniHTTPManager manager = new ArachniHTTPManager(session);
> 
> ➊JObject scanOptions = new JObject();
> 
> scanOptions["checks"] = new JArray() { "xss*", "sql*" };
> 
> scanOptions["audit"] = new JObject();
> 
> scanOptions["audit"]["elements"] = new JArray() { "links", "forms" };
> 
> string url = "http://demo.testfire.net/default.aspx";
> 
> JObject scanId = manager.➋StartScan(url, scanOptions);
> 
> Guid id = Guid.Parse(scanId["id"].ToString());
> 
> JObject scan = manager.➌GetScanStatus(id);
> 
> while (scan["status"].ToString() != "done")
> 
> {
> 
> Console.WriteLine("稍等片刻，直到扫描完成");
> 
> System.Threading.Thread.Sleep(10000);
> 
> scan = manager.GetScanStatus(id);
> 
> }
> 
> ➍Console.WriteLine(scan.ToString());
> 
> }

Listing 12-6: 使用 ArachniHTTPSession 和 ArachniHTTPManager 类驱动 Arachni

在实例化我们的会话（session）和管理器（manager）类之后，我们创建了一个新的 JObject ➊来存储我们的扫描选项。这些选项与您在运行`arachni --help`时看到的命令行选项直接相关（有很多）。通过在“checks”选项键中存储包含 xss*和 sql*值的 JArray，我们告诉 Arachni 对网站进行 XSS 和 SQL 注入测试，而不仅仅是爬取应用程序并查找所有可能的页面和表单。下面的“audit”选项键则告诉 Arachni 审核它找到的链接以及我们要求它运行的任何 HTML 表单。

在设置完扫描选项后，我们通过调用 StartScan()方法➋并传入我们的测试 URL 作为参数来启动扫描。使用 StartScan()返回的 ID，我们通过 GetScanStatus() ➌获取当前扫描状态，然后循环检查直到扫描完成，每秒检查一次新的扫描状态。扫描完成后，我们将 JSON 格式的扫描结果打印到屏幕上 ➍。

Arachni REST API 简单且易于大多数安全工程师或爱好者访问，因为它可以使用基本的命令行工具。它也非常容易通过常见的 C#库进行自动化，应该是 SDLC 或在你自己的网站上进行每周或每月扫描的一个轻松入门。为了增加一些趣味，尝试使用你的自动化工具将 Arachni 与书中已知漏洞的前 Web 应用程序（如 BadStore）一起运行。现在我们已经了解了 Arachni API，可以讨论如何自动化它的 RPC。

Arachni RPC

Arachni RPC 协议比 API 更为先进，但也更强大。虽然和 Metasploit 的 RPC 一样也由 MSGPACK 支持，Arachni 的协议却有些不同。数据有时会进行 Gzip 压缩，并且只能通过常规的 TCP 套接字进行通信，而不是 HTTP。这种复杂性有其优点：RPC 没有 HTTP 开销，因此速度极快，而且它比 API 提供了更多的扫描器管理功能，包括随时启动和停止扫描器的能力，并能够创建分布式扫描集群，从而允许多个 Arachni 实例之间进行扫描负载均衡。简而言之，RPC 非常强大，但预计 REST API 将会获得更多的开发关注和支持，因为它对大多数开发者更加易于接触。

手动运行 RPC

要启动一个 RPC 监听器，我们使用简单的脚本 arachni_rpcd，如示例 12-7 所示。

> $ arachni_rpcd
> 
> Arachni - Web 应用程序安全扫描框架 v2.0dev
> 
> 作者：Tasos "Zapotek" Laskos <tasos.laskos@arachni-scanner.com>
> 
> （在社区和 Arachni 团队的支持下。）
> 
> 网站：http://arachni-scanner.com
> 
> 文档： http://arachni-scanner.com/wiki
> 
> 我，[2016-01-16T18:23:29.000746 #18862] 信息 - 系统：RPC 服务器已启动。
> 
> 我，[2016-01-16T18:23:29.000834 #18862] 信息 - 系统：监听地址 ➊127.0.0.1:7331

示例 12-7：运行 Arachni RPC 服务器

现在我们可以使用另一个随 Arachni 一起提供的脚本来测试监听器，叫做 arachni_rpc。注意在 RPC 服务器的输出中显示的调度器 URL ➊。接下来我们需要用到它。随 Arachni 一起提供的 arachni_rpc 脚本允许你通过命令行与 RPC 监听器进行交互。在启动 arachni_rpcd 监听器后，打开另一个终端，切换到 Arachni 项目的根目录；然后使用 arachni_rpc 脚本启动扫描，如示例 12-8 所示。

> $ arachni_rpc --dispatcher-url 127.0.0.1:7331 \
> 
> "http://demo.testfire.net/default.aspx"

示例 12-8：通过 RPC 运行 Arachni 扫描同一个故意存在漏洞的网站

这个命令将驱动 Arachni 使用 MSGPACK RPC，就像我们接下来将在 C#代码中做到的那样。如果成功，你应该会看到一个基于文本的用户界面，实时更新当前扫描的状态，并在扫描结束时显示漂亮的报告，正如示例 12-9 所示。

> Arachni - Web 应用程序安全扫描框架 v2.0dev
> 
> 作者：Tasos "Zapotek" Laskos <tasos.laskos@arachni-scanner.com>
> 
> （在社区和 Arachni 团队的支持下。）
> 
> 网站： http://arachni-scanner.com
> 
> 文档： http://arachni-scanner.com/wiki
> 
> [~] 已检测到 10 个问题。
> 
> [+] 1 | 在脚本上下文中的 Cross-Site Scripting (XSS) 在
> 
> http://demo.testfire.net/search.aspx 中的表单输入 `txtSearch` 使用 GET。
> 
> [+] 2 | 在 http://demo.testfire.net/search.aspx 的 Cross-Site Scripting (XSS)
> 
> 在表单输入 `txtSearch` 中使用 GET。
> 
> [+] 3 | 在服务器的 http://demo.testfire.net/PR/ 中找到常见目录。
> 
> [+] 4 | 在服务器的 http://demo.testfire.net/default.exe 中备份文件。
> 
> [+] 5 | 在 http://demo.testfire.net/default.aspx 的服务器中缺少 'X-Frame-Options' 头。
> 
> [+] 6 | 在服务器的 http://demo.testfire.net/admin.aspx 中找到常见的管理界面。
> 
> [+] 7 | 在服务器的 http://demo.testfire.net/admin.htm 中找到常见的管理界面。
> 
> [+] 8 | 在服务器的 http://demo.testfire.net/default.aspx 收到有趣的响应。
> 
> [+] 9 | 在 http://demo.testfire.net/default.aspx 的 cookie 中有 HttpOnly cookie 和输入
> 
> `amSessionId`。
> 
> [+] 10 | 在服务器的 http://demo.testfire.net/default.aspx 中允许的 HTTP 方法。
> 
> [~] 状态：扫描中
> 
> [~] 迄今为止发现了 3 个页面。
> 
> [~] 已发送 1251 个请求。
> 
> [~] 收到并分析了 1248 个响应。
> 
> [~] 在 00:00:45
> 
> [~] 平均值：39.3732270014467 请求/秒。
> 
> [~] 当前正在审计 http://demo.testfire.net/default.aspx
> 
> [~] 突发响应时间总和 72.511066 秒
> 
> [~] 突发响应总数 97
> 
> [~] 突发平均响应时间 0.747536762886598 秒
> 
> [~] 突发平均 20.086991167522193 请求/秒
> 
> [~] 超时请求 0
> 
> [~] 原始最大并发数 20
> 
> [~] 限制的最大并发数 20
> 
> [~] ('Ctrl+C' 中止扫描并获取报告) Listing 12-9:  arachni_rpc 命令行扫描 UI

ArachniRPCSession 类

要使用 RPC 框架和 C# 运行扫描，我们将再次实现 session/manager 模式，从 Arachni RPC 会话类开始。通过 RPC 框架，您将与 Arachni 架构有更多的接触，因为您需要在更细粒度的层面上处理调度器和实例。当您首次连接到 RPC 框架时，您会连接到一个调度器。您可以与这个调度器交互来创建和管理实例，这些实例进行实际的扫描和工作，但这些扫描实例最终会动态地监听与调度器不同的端口。为了为调度器和实例提供一个易于使用的接口，我们可以创建一个会话构造函数，让我们能够稍微忽略这些区别，如 Listing 12-10 所示。

> public class ArachniRPCSession : IDisposable
> 
> {
> 
> SslStream _stream = null;
> 
> public ArachniRPCSession(➊string host, int port,
> 
> bool ➋initiateInstance = false)
> 
> {
> 
> this.Host = host;
> 
> this.Port = port;
> 
> ➌GetStream(host, port);
> 
> this.IsInstanceStream = false;
> 
> if (initiateInstance)
> 
> {
> 
> this.InstanceName = ➍Guid.NewGuid().ToString();
> 
> MessagePackObjectDictionary resp =
> 
> this.ExecuteCommand("dispatcher.dispatch"➎,
> 
> new object[] { this.InstanceName }).AsDictionary(); 列表 12-10：ArachniRPCSession 构造函数的前半部分

构造函数接受三个参数 ➊。前两个——连接的主机和主机上的端口——是必需的。第三个参数是可选的 ➋（默认为 false），它允许程序员自动创建一个新的扫描实例并连接到它，而无需通过调度器手动创建新实例。

在分别将 Host 和 Port 属性赋值为传递给构造函数的前两个参数后，我们使用 GetStream() ➌ 连接到调度器。如果第三个参数传入 true（默认为 false），实例化实例时（默认值为 false），我们使用新的 Guid 创建一个唯一的实例名称，并运行 dispatcher.dispatch ➎ RPC 命令来创建一个新的扫描器实例，该实例返回一个新的端口（如果你有多个扫描器实例集群，可能还会返回新的主机）。列表 12-11 显示了构造函数的其余部分。

> string[] url = ➊resp["url"].AsString().Split(':');
> 
> this.InstanceHost = url[0];
> 
> this.InstancePort = int.Parse(url[1]);
> 
> this.Token = ➋resp["token"].AsString();
> 
> ➌GetStream(this.InstanceHost, this.InstancePort);
> 
> bool aliveResp = this.➍ExecuteCommand("service.alive?", new object[] { },
> 
> this.Token).AsBoolean();
> 
> this.IsInstanceStream = aliveResp;
> 
> }
> 
> }
> 
> ➎public string Host { get; set; }
> 
> public int Port { get; set; }
> 
> public string Token { get; set; }
> 
> public bool IsInstanceStream { get; set; }
> 
> public string InstanceHost { get; set; }
> 
> public int InstancePort { get; set; }
> 
> public string InstanceName { get; set; }

列表 12-11：ArachniRPCSession 构造函数的后半部分及其属性

在 ➊ 处，我们将扫描器实例的 URL（例如 127.0.0.1:7331）拆分为 IP 地址和端口（分别为 127.0.0.1 和 7331）。一旦我们获取到用于实际扫描的实例主机和端口后，我们将它们分别赋值给 InstanceHost 和 InstancePort 属性。我们还会保存调度器返回的认证令牌 ➋，以便稍后对扫描器实例进行认证的 RPC 调用。该认证令牌是 Arachni RPC 在我们调度新实例时自动生成的，这样只有我们能使用这个新扫描器及其令牌。

我们使用 GetStream() ➌连接到扫描实例，它提供了对扫描实例的直接访问。如果连接成功且扫描实例处于活动状态 ➍，我们将 IsInstanceStream 属性设置为 true，这样我们就能知道自己是在驱动调度器还是扫描实例（这决定了我们稍后在实现 ArachniRPCManager 类时可以对 Arachni 进行哪些 RPC 调用，比如创建扫描器或执行扫描）。在构造函数之后，我们定义了会话类的属性 ➎，所有这些属性在构造函数中都会用到。

ExecuteCommand 的辅助方法

在我们实现 ExecuteCommand()之前，我们需要实现 ExecuteCommand()的辅助方法。我们快完成了！第 12-12 页显示了我们需要的方法，以便完成 ArachniRPCSession 类的实现。

> public byte[] 解压数据(byte[] inData)
> 
> {
> 
> using (MemoryStream outMemoryStream = new MemoryStream())
> 
> {
> 
> using (➊ZOutputStream outZStream = new ZOutputStream(outMemoryStream))
> 
> {
> 
> outZStream.Write(inData, 0, inData.Length);
> 
> return outMemoryStream.ToArray();
> 
> }
> 
> }
> 
> }
> 
> private byte[] ➋ReadMessage(SslStream sslStream)
> 
> {
> 
> byte[] sizeBytes = new byte[4];
> 
> sslStream.Read(sizeBytes, 0, sizeBytes.Length);
> 
> if (BitConverter.IsLittleEndian)
> 
> Array.Reverse(sizeBytes);
> 
> uint size = BitConverter.➌ToUInt32(sizeBytes, 0);
> 
> byte[] buffer = new byte[size];
> 
> sslStream.Read(buffer, 0, buffer.Length);
> 
> return buffer;
> 
> }
> 
> private void ➍获取流(string host, int port)
> 
> {
> 
> TcpClient client = new TcpClient(host, port);
> 
> _stream = new SslStream(client.GetStream(), false,
> 
> new RemoteCertificateValidationCallback(➎验证服务器证书),
> 
> (sender, targetHost, localCertificates,
> 
> remoteCertificate, acceptableIssuers)
> 
> => null);
> 
> _stream.AuthenticateAsClient("arachni", null, SslProtocols.Tls, false);
> 
> }
> 
> private bool 验证服务器证书(object sender, X509Certificate certificate,
> 
> X509Chain chain, SslPolicyErrors sslPolicyErrors)
> 
> {
> 
> return true;
> 
> }
> 
> public void ➏Dispose()
> 
> {
> 
> if (this.IsInstanceStream && _stream != null)
> 
> this.ExecuteCommand(➐"service.shutdown", new object[] { }, this.Token);
> 
> if (_stream != null)
> 
> _stream.Dispose();
> 
> _stream = null;
> 
> }

第 12-12 页：ArachniRPCSession 类的辅助方法

大多数 RPC 会话类的辅助方法相对简单。DecompressData()方法使用 NuGet 中提供的 zlib 库创建一个新的输出流，名为 ZOutputStream ➊。它返回解压后的数据作为字节数组。在 ReadMessage()方法 ➋中，我们从流中读取前 4 个字节，然后将这些字节转换为一个 32 位无符号整数 ➌，表示其余数据的长度。一旦知道数据长度，我们就从流中读取剩余数据，并将其作为字节数组返回。

`GetStream()`方法➍与我们在 OpenVAS 库中用于创建网络流的代码非常相似。我们创建一个新的`TcpClient`并将流包装在`SslStream`中。我们使用`ValidateServerCertificate()`方法➎通过始终返回`true`来信任所有 SSL 证书。这允许我们连接到具有自签名证书的 RPC 实例。最后，`Dispose()` ➏是`ArachniRPCSession`类实现的`IDisposable`接口所要求的。如果我们正在驱动一个扫描实例而不是调度器（在创建`ArachniRPCSession`时的构造函数中设置），我们发送一个关闭命令➐给该实例，以清理扫描实例，但保持调度器运行。

`ExecuteCommand()`方法

如清单 12-13 所示，`ExecuteCommand()`方法封装了所有必需的功能，用于发送命令并接收来自 Arachni RPC 的响应。

> `public MessagePackObject ➊ExecuteCommand(string command, object[] args,`
> 
> `string token = null)`
> 
> {
> 
> ➋`Dictionary<string, object> = new Dictionary<string, object>();`
> 
> ➌`message["message"] = command;`
> 
> `message["args"] = args;`
> 
> `if (token != null)`
> 
> ➍`message["token"] = token;`
> 
> `byte[] packed;`
> 
> `using (MemoryStream stream = new ➎MemoryStream())`
> 
> {
> 
> `Packer packer = Packer.Create(stream);`
> 
> `packer.PackMap(message);`
> 
> `packed = stream.ToArray();`
> 
> }

清单 12-13：`ArachniRPCSession`类中`ExecuteCommand()`方法的前半部分

`ExecuteCommand()`方法➊接受三个参数：要执行的命令、与命令一起使用的参数对象，以及一个可选的身份验证令牌参数（如果提供了身份验证令牌）。该方法稍后将主要由`ArachniRPCManager`类使用。我们通过创建一个新的字典来开始该方法，名为`request`，用于保存我们的命令数据（要运行的命令和 RPC 命令的参数）➋。然后，我们将字典中的`message`键➌赋值为传递给`ExecuteCommand()`方法的第一个参数，即要运行的命令。接着，我们将字典中的`args`键赋值为传递给方法的第二个参数，即要运行命令的选项。当我们发送消息时，Arachni 将查看这些键，使用给定的参数运行 RPC 命令，并返回响应。如果第三个参数（可选）不为`null`，我们将`token`键 ➍ 赋值为传递给方法的身份验证令牌。这三个字典键（`message`、`args`和`token`）是 Arachni 在接收到序列化数据时将查看的内容。

一旦我们设置好包含要发送给 Arachni 的信息的请求字典，我们就创建一个新的 MemoryStream() ➎ 并使用 Metasploit 绑定中的相同 Packer 类，在第十一章中将请求字典序列化为字节数组。现在，我们已经准备好发送数据给 Arachni 执行 RPC 命令，接下来我们需要发送数据并读取 Arachni 的响应。这发生在 ExecuteCommand() 方法的后半部分，见清单 12-14。

> byte[] packedLength = ➊BitConverter.GetBytes(packed.Length);
> 
> if (BitConverter.IsLittleEndian)
> 
> Array.Reverse(packedLength);
> 
> ➋_stream.Write(packedLength);
> 
> ➌_stream.Write(packed);
> 
> byte[] respBytes = ➍ReadMessage(_stream);
> 
> MessagePackObjectDictionary resp = null;
> 
> try
> 
> {
> 
> resp = Unpacking.UnpackObject(respBytes).Value.AsDictionary();
> 
> }
> 
> ➎catch
> 
> {
> 
> byte[] decompressed = DecompressData(respBytes);
> 
> resp = Unpacking.UnpackObject(decompressed).Value.AsDictionary();
> 
> }
> 
> return resp.ContainsKey("obj") ? resp["obj"] : resp["exception"];
> 
> }

清单 12-14：ArachniRPCSession 类中 ExecuteCommand() 方法的后半部分

由于 Arachni RPC 流使用简单协议进行通信，我们可以轻松地将 MSGPACK 数据发送给 Arachni，但我们需要向 Arachni 发送两个信息，而不仅仅是 MSGPACK 数据。我们首先需要将 MSGPACK 数据的大小作为一个 4 字节的整数发送给 Arachni，紧跟在 MSGPACK 数据之前。这个整数表示每个消息中序列化数据的长度，告诉接收方（在本例中为 Arachni）需要读取多少字节作为消息的一部分。我们需要获取数据长度的字节，因此使用 BitConverter.GetBytes() ➊ 获取 4 字节数组。数据的长度和数据本身需要按照特定顺序写入 Arachni 流中。我们首先将表示数据长度的 4 字节写入流中 ➋，然后写入完整的序列化消息 ➌。

接下来，我们需要读取来自 Arachni 的响应并将其返回给调用者。使用 ReadMessage() 方法 ➍，我们从响应中获取消息的原始字节，并尝试在 try/catch 块中将其解包成 MessagePackObjectDictionary。如果第一次尝试失败，这意味着数据使用 Gzip 压缩，因此 catch 块 ➎ 会接管。我们解压缩数据，然后将解压后的字节解包成 MessagePackObjectDictionary。最后，我们返回服务器的完整响应，或者如果发生错误，则返回异常。

ArachniRPCManager 类

ArachniRPCManager 类比 ArachniRPCSession 类要简单得多，见清单 12-15。

> public class ArachniRPCManager : IDisposable
> 
> {
> 
> ArachniRPCSession _session;
> 
> public ArachniRPCManager(➊ArachniRPCSession session)
> 
> {
> 
> if (!session.IsInstanceStream)
> 
> throw new Exception("Session 必须使用实例流");
> 
> _session = session;
> 
> }
> 
> public MessagePackObject ➋StartScan(string url, string checks = "*")
> 
> {
> 
> Dictionary<string, object>args = new Dictionary<string, object>();
> 
> args["url"] = url;
> 
> args["checks"] = checks;
> 
> args["audit"] = new Dictionary<string, object>();
> 
> ((Dictionary<string, object>)args["audit"])["elements"] = new object[] { "links", "forms" };
> 
> return _session.ExecuteCommand(➌"service.scan", new object[] { args }, _session.Token);
> 
> }
> 
> public MessagePackObject ➍GetProgress(List<uint> digests = null)
> 
> {
> 
> Dictionary<string, object>args = new Dictionary<string, object>();
> 
> args["with"] = "issues";
> 
> 如果 digests 不为空
> 
> {
> 
> args["without"] = new Dictionary<string, object>();
> 
> ((Dictionary<string, object>)args["without"])["issues"] = digests.ToArray();
> 
> }
> 
> return _session.➎ExecuteCommand("service.progress", new object[] { args }, _session.Token);
> 
> }
> 
> public MessagePackObject ➏IsBusy()
> 
> {
> 
> return _session.ExecuteCommand("service.busy?", new object[] { }, _session.Token);
> 
> }
> 
> public void Dispose()
> 
> {
> 
> ➐_session.Dispose();
> 
> }
> 
> }

示例 12-15：ArachniRPCManager 类

首先，ArachniRPCManager 构造函数接受一个 ArachniRPCSession ➊ 作为唯一参数。我们的管理器类只会实现扫描实例的方法，而不是调度器，因此，如果传入的会话不是扫描实例，我们会抛出异常。否则，我们将会话分配给本地类变量，以便在其他方法中使用。

我们在 ArachniRPCManager 类中创建的第一个方法是 StartScan() 方法 ➋，它接受两个参数。第一个参数是必需的，是 Arachni 将要扫描的 URL 字符串。第二个参数是可选的，默认会运行所有检查（例如 XSS、SQL 注入和路径遍历等），但如果用户希望在传递给 StartScan() 的选项中指定不同的检查，它可以被更改。为了确定运行哪些检查，我们通过实例化一个新的字典并使用传递给 StartScan() 方法的 url 和 checks 参数以及 Arachni 将查看的 audit 来构建一个新的消息，Arachni 根据这个消息来确定执行何种扫描。最后，我们使用 service.scan 命令 ➌ 发送该消息并将响应返回给调用者。

GetProgress() 方法 ➍ 接受一个可选的单一参数：一个整数列表，Arachni 用它来标识报告的问题。我们将在下一节中详细讨论 Arachni 如何报告问题。使用此参数，我们构建一个小字典并将其传递给 service.progress 命令 ➎，该命令将返回扫描的当前进度和状态。我们将命令发送给 Arachni，然后将结果返回给调用者。

最后一个重要的方法，IsBusy() ➏，简单地告诉我们当前的扫描器是否正在进行扫描。最后，我们通过 Dispose() ➐ 清理所有内容。

综合起来

现在我们已经有了驱动 Arachni 的 RPC 来扫描 URL 并实时报告结果的基本构件。清单 12-16 展示了如何将所有部分组合在一起，使用 RPC 扫描一个 URL。

> public static void Main(string[] args)
> 
> {
> 
> using (ArachniRPCSession session = new ➊ArachniRPCSession("127.0.0.1",
> 
> 7331, true))
> 
> {
> 
> using (ArachniRPCManager manager = new ArachniRPCManager(session))
> 
> {
> 
> Console.➋WriteLine("正在使用实例: " + session.InstanceName);
> 
> manager.StartScan("http://demo.testfire.net/default.aspx");
> 
> bool isRunning = manager.IsBusy().AsBoolean();
> 
> List<uint> issues = new List<uint>();
> 
> DateTime start = DateTime.Now;
> 
> Console.WriteLine("扫描开始时间: " + start.ToLongTimeString());
> 
> ➌while (isRunning)
> 
> {
> 
> Thread.Sleep(10000);
> 
> var progress = manager.GetProgress(issues);
> 
> foreach (MessagePackObject p in
> 
> progress.AsDictionary()["issues"].AsEnumerable())
> 
> {
> 
> MessagePackObjectDictionary dict = p.AsDictionary();
> 
> Console.➍WriteLine("发现问题: " + dict["name"].AsString());
> 
> issues.Add(dict["digest"].AsUInt32());
> 
> }
> 
> isRunning = manager.➎IsBusy().AsBoolean();
> 
> }
> 
> DateTime end = DateTime.Now;
> 
> ➏Console.WriteLine("扫描结束时间: " + end.ToLongTimeString() +
> 
> ". 扫描花费时间: " + ((end - start).ToString()) + ")。
> 
> }
> 
> }
> 
> }

清单 12-16: 使用 RPC 类驱动 Arachni

我们通过创建一个新的 ArachniRPCSession ➊来启动 Main()方法，传递 Arachni 调度器的主机和端口，并将 true 作为第三个参数自动获取一个新的扫描实例。一旦我们拥有了会话和管理器类，并且连接到 Arachni，我们打印当前的实例名称 ➋，它应该就是我们在创建扫描实例时生成的唯一 ID。接着，我们通过将测试 URL 传递给 StartScan()方法来启动扫描。

一旦扫描开始，我们可以观察它直到完成，然后打印最终报告。在创建几个变量（如一个空的列表，用来存储 Arachni 回传的报告问题）和扫描开始的时间后，我们开始一个 while 循环 ➌，它会持续运行直到 isRunning 为 false。在 while 循环中，我们调用 GetProgress()来获取扫描的当前进度，然后打印 ➍并存储自上次调用 GetProgress()以来发现的任何新问题。我们最终暂停 10 秒，然后再次调用 IsBusy() ➎。接着我们重新开始这个过程，直到扫描完成。所有步骤完成后，我们打印一个简短的总结 ➏，说明扫描花费的时间。如果你查看自动化报告的漏洞（我截断的结果见清单 12-17）以及我们在章节开始时手动执行的原始 Arachni 扫描，它们应该是一致的！

> $ mono ./ch12_automating_arachni.exe
> 
> 使用实例: 1892413b-7656-4491-b6c0-05872396b42f
> 
> 扫描开始时间: 8:58:12 AM
> 
> 发现问题: 跨站脚本（XSS）➊
> 
> 发现问题: 常见目录
> 
> 发现问题: 备份文件➋
> 
> 发现问题: 缺少 'X-Frame-Options' 头部
> 
> 发现的问题：有趣的响应
> 
> 发现的问题：允许的 HTTP 方法
> 
> 发现的问题：有趣的响应
> 
> 发现的问题：路径遍历 ➌
> 
> --snip--

列表 12-17：运行 Arachni C# 类扫描并报告示例 URL

因为我们启用了所有检查项来运行 Arachni，这个站点将报告大量的漏洞！仅在前十行左右，Arachni 就报告了一个 XSS 漏洞 ➊，一个可能包含敏感信息的备份文件 ➋，以及一个路径遍历弱点 ➌。如果你只想限制 Arachni 执行 XSS 漏洞扫描，你可以将一个第二个参数传递给 StartScan，值为 xss*（该参数的默认值是 *，表示“所有检查”），Arachni 就只会检查并报告找到的任何 XSS 漏洞。该命令最终看起来像以下这行代码：manager.StartScan("http://demo.testfire.net/default.aspx", "xss*"); Arachni 支持多种检查，包括 SQL 和命令注入，因此我鼓励你阅读文档，了解支持的检查项。

结论

Arachni 是一款非常强大且多功能的 web 应用程序扫描器，是任何认真从事安全工程或渗透测试工作的工程师工具箱中的必备工具。正如你在本章中所看到的，你可以轻松地在简单和复杂的场景中使用它。如果你只需要定期扫描单一应用程序，HTTP API 可能就足够了。然而，如果你发现自己不断扫描新的和不同的应用程序，那么随时启动扫描器的能力可能是分发扫描并避免瓶颈的最佳方式。

我们首先实现了一组简单的类，与 Arachni REST API 接口，以便启动、监控并报告扫描结果。利用我们工具集中基础的 HTTP 库，我们能够轻松构建模块化的类来驱动 Arachni。

在我们完成了更简单的 REST API 后，我们将 Arachni 推进了一步，通过 MSGPACK RPC 来驱动它。使用几个开源第三方库，我们能够使用 Arachni 的一些更强大的功能。我们利用其分布式模型，通过 RPC 调度器创建了一个新的扫描实例，然后扫描了一个 URL 并实时报告了结果。

使用这些构建块中的任何一个，你都可以将 Arachni 集成到任何 SDLC 或持续集成系统中，以确保你或你的组织使用或构建的 web 应用程序的质量和安全性。
