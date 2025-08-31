# 第七章

7

自动化 OpenVAS

![](img/00010.jpg)

在本章中，我将向你介绍 OpenVAS 和 OpenVAS 管理协议（OMP），这是一种免费的开源漏洞管理系统，源自 Nessus 最后的开源版本。第五章 和 第六章 中，我们分别讨论了自动化专有漏洞扫描器 Nessus 和 Nexpose。虽然 OpenVAS 功能类似，但它是你武器库中的另一款强大工具。

我将向你展示如何通过核心的 C# 库和一些自定义类来驱动 OpenVAS 扫描并报告你网络中主机的漏洞。在你读完本章后，你应该能够使用 OpenVAS 和 C# 来评估任何网络连接主机的漏洞。

安装 OpenVAS

安装 OpenVAS 的最简单方法是从 [`www.openvas.org/`](http://www.openvas.org/) 下载预构建的 OpenVAS 演示虚拟设备。你下载的文件是一个 .ova 文件（开放虚拟化档案），可以在虚拟化工具如 VirtualBox 或 VMware 中运行。先在你的系统上安装 VirtualBox 或 VMware，然后打开下载的 .ova 文件并在你选择的虚拟化工具中运行它。（为了提升性能，建议给 OVA 设备分配至少 4GB 的内存。）虚拟设备的 root 密码是 root。你在用最新漏洞数据更新设备时应使用 root 用户。

登录后，使用 示例 7-1 中显示的命令更新 OpenVAS，以获取最新的漏洞信息。

> # openvas-nvt-sync
> # 
> # openvas-scapdata-sync
> # 
> # openvas-certdata-sync
> # 
> # openvasmd --update  示例 7-1：用于更新 OpenVAS 的命令

根据你的网络连接，更新可能需要一段时间才能完成。一旦完成，尝试连接到端口 9390 上的 openvasmd 进程，然后运行如示例 7-2 所示的测试命令。

> $ openssl s_client <ip 地址>:9390
> 
> [...SSL 协商...]
> 
> <get_version />
> 
> <get_version_response status="200" status_text="OK"><version>6.0</version></get_version_response> 示例 7-2：连接到 openvasmd

如果一切正常，你应该能在输出的末尾看到状态消息中的 OK。

构建类

与 Nexpose API 类似，OpenVAS 通过 XML 格式将数据传输到服务器。为了自动化 OpenVAS 扫描，我们将使用前面章节中讨论的 Session 类和 Manager 类的组合。OpenVASSession 类将负责我们如何与 OpenVAS 通信，并处理认证问题。OpenVASManager 类将封装 API 中的常见功能，使得程序员使用该 API 更加简单。

OpenVASSession 类

我们将使用 OpenVASSession 类与 OpenVAS 进行通信。示例 7-3 展示了构造函数和属性，标志着 OpenVASSession 类的开始。

> public class OpenVASSession : IDisposable
> 
> {
> 
> private SslStream _stream = null;
> 
> public OpenVASSession(string user, string pass, string host, int port = ➊9390)
> 
> {
> 
> this.ServerIPAddress = ➋IPAddress.Parse(host);
> 
> this.ServerPort = port;
> 
> this.Authenticate(username, password);
> 
> }
> 
> public string Username { get; set; }
> 
> public string Password { get; set; }
> 
> public IPAddress ServerIPAddress { get; set; }
> 
> public int ServerPort { get; set; }
> 
> public SslStream Stream
> 
> {
> 
> ➌get
> 
> {
> 
> 如果 (_stream == null)
> 
> GetStream();
> 
> return _stream;
> 
> }
> 
> ➍set { _stream = value; }
> 
> }

列表 7-3：OpenVASSession 类的构造函数和属性

OpenVASSession 构造函数最多接受四个参数：用于与 OpenVAS 进行身份验证的用户名和密码（在虚拟设备中默认是 admin:admin）；要连接的主机；以及可选的连接主机时使用的端口，默认值为 9390 ➊。

我们将主机参数传递给 IPAddress.Parse() ➋，并将结果赋值给 ServerIPAddress 属性。接下来，我们将端口变量的值赋给 ServerPort 属性，并在身份验证成功时将用户名和密码传递给 Authenticate() 方法（如下一节所讨论）。ServerIPAddress 和 ServerPort 属性在构造函数中被赋值，并在类中使用。

Stream 属性使用 get ➌ 检查私有的 _stream 成员变量是否为 null。如果是，它调用 GetStream()，该方法将 _stream 设置为与 OpenVAS 服务器的连接，然后返回 _stream 变量。

与 OpenVAS 服务器进行身份验证

为了尝试与 OpenVAS 服务器进行身份验证，我们发送一个包含用户名和密码的 XML 文档到 OpenVAS，然后读取响应，如 列表 7-4 所示。如果身份验证成功，我们应该能够调用更高权限的命令来指定扫描目标、获取报告等。

> public XDocument ➊Authenticate(string username, string password)
> 
> {
> 
> XDocument authXML = new XDocument(
> 
> new XElement("authenticate",
> 
> new XElement("credentials",
> 
> new XElement("username", ➋username),
> 
> new XElement("password", ➌password))));
> 
> XDocument response = this.➍ExecuteCommand(authXML);
> 
> 如果 response.Root.Attribute(➎"status").Value != "200"
> 
> throw new Exception("身份验证失败");
> 
> this.Username = username;
> 
> this.Password = password;
> 
> return response;
> 
> }

列表 7-4：OpenVASSession 构造函数的 Authenticate() 方法

Authenticate() 方法 ➊ 首先接受两个参数：用于与 OpenVAS 进行身份验证的用户名和密码。我们创建一个新的身份验证 XML 命令，使用提供的用户名 ➋ 和密码 ➌ 作为凭据；然后我们通过 ExecuteCommand() ➍ 发送身份验证请求，并存储响应，以确保身份验证成功并获取身份验证令牌。

如果服务器返回的根 XML 元素的状态属性 ➎ 为 200，则说明身份验证成功。我们将分配用户名属性、密码属性以及方法的任何参数，然后返回身份验证响应。

创建一个执行 OpenVAS 命令的方法

Listing 7-5 显示了 ExecuteCommand() 方法，它接受一个任意的 OpenVAS 命令，发送到 OpenVAS 并返回结果。

> public XDocument ExecuteCommand(XDocument doc)
> 
> {
> 
> ASCIIEncoding enc = new ASCIIEncoding();
> 
> string xml = doc.ToString();
> 
> this.Stream.➊Write(enc.GetBytes(xml), 0, xml.Length);
> 
> return ReadMessage(this.Stream);
> 
> }

Listing 7-5: ExecuteCommand() 方法用于 OpenVAS

为了使用 OpenVAS 管理协议执行命令，我们通过 TCP 套接字发送 XML 到服务器并接收 XML 响应。ExecuteCommand() 方法只接受一个参数：要发送的 XML 文档。我们在 XML 文档上调用 ToString()，保存结果，然后使用 Stream 属性的 Write() 方法 ➊ 将 XML 写入流中。

读取服务器消息

我们使用 Listing 7-6 中显示的 ReadMessage() 方法来读取服务器返回的消息。

> private XDocument ReadMessage(SslStream ➊sslStream)
> 
> {
> 
> using (var stream = new ➋MemoryStream())
> 
> {
> 
> int bytesRead = 0;
> 
> ➌do
> 
> {
> 
> byte[] buffer = new byte[2048];
> 
> bytesRead = sslStream.➍Read(buffer, 0, buffer.Length);
> 
> stream.Write(buffer, 0, bytesRead);
> 
> if (bytesRead < buffer.Length)
> 
> {
> 
> ➎try
> 
> {
> 
> string xml = System.Text.Encoding.ASCII.GetString(stream.ToArray());
> 
> return XDocument.Parse(xml);
> 
> }
> 
> catch
> 
> {
> 
> ➏continue;
> 
> }
> 
> }
> 
> }
> 
> while (bytesRead > 0);
> 
> }
> 
> return null;
> 
> }

Listing 7-6: ReadMessage() 方法用于 OpenVAS

这个方法从 TCP 流中分块读取 XML 文档，并将文档（或 null）返回给调用者。在将 sslStream ➊ 传递给方法后，我们声明一个 MemoryStream ➋，它允许我们动态存储从服务器接收的数据。接着，我们声明一个整数来存储读取的字节数，并使用 do/while 循环 ➌ 来创建一个 2048 字节的缓冲区以读取数据。然后，我们在 SslStream 上调用 Read() ➍ 方法，将缓冲区填充从流中读取的字节数，之后我们使用 Write() 方法将来自 OpenVAS 的数据复制到 MemoryStream 中，以便后续解析成 XML。

如果服务器返回的数据少于缓冲区能够容纳的内容，我们需要检查是否从服务器读取了有效的 XML 文档。为此，我们在 try/catch 块 ➎ 中使用 GetString() 将存储在 MemoryStream 中的字节转换为可解析的字符串，并尝试解析 XML，因为如果 XML 无效，解析将抛出异常。如果没有抛出异常，我们返回 XML 文档。如果抛出异常，我们知道我们还没有读取完流的数据，因此调用 continue ➏ 以读取更多数据。如果我们已经完成了从流中读取字节，但仍未返回有效的 XML 文档，我们返回 null。这是一种防御性措施，以防与 OpenVAS 的通信中断，并且无法读取完整的 API 响应。返回 null 允许我们稍后检查来自 OpenVAS 的响应是否有效，因为只有在无法读取完整的 XML 响应时，才会返回 null。

设置 TCP 流以发送和接收命令

清单 7-7 显示了首先出现在 清单 7-3 中的 GetStream() 方法。它建立了与 OpenVAS 服务器的实际 TCP 连接，我们将使用该连接来发送和接收命令。

> private void GetStream()
> 
> {
> 
> if (_stream == null || !_stream.CanRead)
> 
> {
> 
> TcpClient client = new ➊TcpClient(this.ServerIPAddress.ToString(), this.ServerPort);
> 
> _stream = new ➋SslStream(client.GetStream(), false,
> 
> new RemoteCertificateValidationCallback (ValidateServerCertificate),
> 
> (sender, targetHost, localCertificates, remoteCertificate, acceptableIssuers) => null);
> 
> _stream.➌AuthenticateAsClient("OpenVAS", null, SslProtocols.Tls, false);
> 
> }
> 
> }

清单 7-7：OpenVASSession 构造函数的 GetStream() 方法

GetStream() 方法为与 OpenVAS 通信时其余类中的其他方法设置了 TCP 流。为此，我们通过将 ServerIPAddress 和 ServerPort 属性传递给 TcpClient 来实例化一个新的 TcpClient ➊，如果流无效。然后我们将流包装在一个不验证 SSL 证书的 SslStream ➋ 中，因为 OpenVAS 使用的 SSL 证书是自签名的，会抛出错误；接着，我们通过调用 AuthenticateAsClient() ➌ 执行 SSL 握手。现在，OpenVAS 服务器的 TCP 流可以被其余方法使用，当我们开始发送命令和接收响应时。

证书验证和垃圾回收

清单 7-8 显示了用于验证 SSL 证书的方法（由于 OpenVAS 默认使用的是自签名的 SSL 证书）并且在完成后清理我们的会话。

> private bool ValidateServerCertificate(object sender, X509Certificate certificate,
> 
> X509Chain chain, SslPolicyErrors sslPolicyErrors)
> 
> {
> 
> return ➊true;
> 
> }
> 
> public void Dispose()
> 
> {
> 
> if (_stream != null)
> 
> ➋_stream.Dispose();
> 
> }

清单 7-8：ValidateServerCertificate() 和 Dispose() 方法

返回 true ➊ 通常不是一个好的实践，但由于在我们的例子中，OpenVAS 使用的是自签名 SSL 证书，否则该证书无法验证，因此我们必须允许所有证书。与之前的示例一样，我们创建 Dispose() 方法，以便在处理网络或文件流后清理资源。如果 OpenVASSession 类中的流不为 null，我们将释放用于与 OpenVAS 通信的内部流 ➋。

获取 OpenVAS 版本

我们现在可以通过 OpenVAS 启动命令并获取响应，如 Listing 7-9 所示。例如，我们可以运行类似 get_version 的命令，该命令返回 OpenVAS 实例的版本信息。我们稍后会在 OpenVASManager 类中封装类似的功能。

> class MainClass
> 
> {
> 
> public static void Main(string[] args)
> 
> {
> 
> using (OpenVASSession session = new ➊OpenVASSession("admin", "admin", "192.168.1.19"))
> 
> {
> 
> XDocument doc = session.➋ExecuteCommand(
> 
> XDocument.Parse("<get_version />"));
> 
> Console.WriteLine(doc.ToString());
> 
> }
> 
> }
> 
> }

Listing 7-9: Main() 方法驱动 OpenVAS 获取当前版本

我们通过传入用户名、密码和主机来创建一个新的 OpenVASSession ➊。接下来，我们将一个请求 OpenVAS 版本的 XDocument 传递给 ExecuteCommand() ➋，将结果存储在一个新的 XDocument 中，然后将其输出到屏幕上。Listing 7-9 的输出应类似于 Listing 7-10。

> <get_version_response status="200" status_text="OK">
> 
> <version>6.0</version>
> 
> </get_version_response> Listing 7-10: OpenVAS 对 <get_version /> 的响应

OpenVASManager 类

我们将使用 OpenVASManager 类（如 Listing 7-11 中所示）来封装 API 调用，以启动扫描、监控扫描并获取扫描结果。

> public class OpenVASManager : IDisposable
> 
> {
> 
> private OpenVASSession _session;
> 
> public OpenVASManager(OpenVASSession ➊session)
> 
> {
> 
> if (session != null)
> 
> _session = session;
> 
> else
> 
> throw new ArgumentNullException("session");
> 
> }
> 
> public XDocument ➋GetVersion()
> 
> {
> 
> return _session.ExecuteCommand(XDocument.Parse("<get_version />"));
> 
> }
> 
> private void Dispose()
> 
> {
> 
> _session.Dispose();
> 
> }
> 
> }

Listing 7-11: OpenVASManager 构造函数和 GetVersion() 方法

OpenVASManager 类的构造函数接受一个参数，即 OpenVASSession ➊。如果传入的 session 参数为 null，我们会抛出异常，因为没有有效的 session 我们无法与 OpenVAS 通信。否则，我们将该 session 分配给一个本地类变量，以便在类中的方法中使用，如 GetVersion()。然后，我们实现 GetVersion() ➋ 方法来获取 OpenVAS 的版本（如 Listing 7-9 中所示）以及 Dispose() 方法。

我们现在可以用 OpenVASManager 替换 Main() 方法中调用 ExecuteCommand() 的代码，以获取 OpenVAS 版本，如 Listing 7-12 所示。

> public static void Main(string[] args)
> 
> {
> 
> using (OpenVASSession session = new OpenVASSession("admin", "admin", "192.168.1.19"))
> 
> {
> 
> using (OpenVASManager manager = new OpenVASManager(session))
> 
> {
> 
> XDocument version = manager.GetVersion();
> 
> Console.WriteLine(version);
> 
> }
> 
> }
> 
> }

清单 7-12：Main() 方法通过 OpenVASManager 类获取 OpenVAS 版本

程序员不再需要记住获取版本信息所需的 XML，因为它已经通过一个方便的方法调用进行了抽象。我们可以遵循这个模式来调用 API 中的其他命令。

获取扫描配置和创建目标

清单 7-13 展示了我们如何在 OpenVASManager 中添加命令，创建新目标并获取扫描配置。

> public XDocument GetScanConfigurations()
> 
> {
> 
> return _session.ExecuteCommand(XDocument.Parse(➊"<get_configs />"));
> 
> }
> 
> public XDocument CreateSimpleTarget(string cidrRange, string targetName)
> 
> {
> 
> XDocument createTargetXML = new XDocument(
> 
> new XElement(➋"create_target",
> 
> new XElement("name", targetName),
> 
> new XElement("hosts", cidrRange)));
> 
> return _session.ExecuteCommand(createTargetXML);
> 
> }

清单 7-13：OpenVAS GetScanConfigurations() 和 CreateSimpleTarget() 方法

GetScanConfigurations() 方法将 <get_configs /> 命令 ➊ 传递给 OpenVAS 并返回响应。CreateSimpleTarget() 方法接受 IP 地址或 CIDR 范围（例如 192.168.1.0/24）和目标名称作为参数，我们使用这些信息通过 XDocument 和 XElement 构建一个 XML 文档。第一个 XElement 创建一个名为 create_target 的根 XML 节点 ➋。其余的两个包含目标的名称和主机信息。清单 7-14 展示了生成的 XML 文档。

> <create_target>
> 
> <name>家庭网络</name>
> 
> <hosts>192.168.1.0/24</hosts>
> 
> </create_target> 清单 7-14：OpenVAS create_target 命令 XML

清单 7-15 展示了我们如何创建目标并对其进行扫描，以获取 Discovery 扫描配置，该配置执行基本的端口扫描和其他基本的网络测试。

> XDocument target = manager.➊CreateSimpleTarget("192.168.1.31", Guid.NewGuid().ToString());
> 
> string targetID = target.Root.Attribute("id").➋Value;
> 
> XDocument configs = manager.GetScanConfigurations();
> 
> string discoveryConfigID = string.Empty;
> 
> foreach (XElement node in configs.Descendants("name"))
> 
> {
> 
> if (node.Value == ➌"Discovery")
> 
> {
> 
> discoveryConfigID = node.Parent.Attribute("id").Value;
> 
> break;
> 
> }
> 
> }
> 
> Console.➍WriteLine("正在创建目标 " + targetID + " 的扫描，使用的扫描配置是 " +
> 
> discoveryConfigID); 清单 7-15：创建 OpenVAS 目标并获取扫描配置 ID

首先，我们通过调用 CreateSimpleTarget() ➊ 来创建一个要扫描的目标，传入要扫描的 IP 地址和一个新的 Guid 作为目标名称。为了自动化，我们不需要目标的可读名称，因此我们只生成一个 Guid 作为名称。

注意

> 将来，你可能想将目标命名为 Databases 或 Workstations，以便区分网络上的特定机器进行扫描。你也可以指定像这样的可读名称，但每个目标的名称必须是唯一的。)

以下是成功创建目标时响应的样子：<create_target_response status="201" status_text="OK, resource created"

id="254cd3ef-bbe1-4d58-859d-21b8d0c046c6"/> 创建目标后，我们从 XML 响应中获取 id 属性的值 ➋，并将其存储，以便在需要获取扫描状态时使用。接着，我们调用 GetScanConfigurations() 获取所有可用的扫描配置，将它们存储并遍历，找到名称为 Discovery ➌ 的配置。最后，我们使用 WriteLine() ➍ 将一条消息打印到屏幕，告诉用户将使用哪个目标和扫描配置 ID 进行扫描。

创建并启动任务

Listing 7-16 展示了如何使用 OpenVASManager 类创建并启动扫描。

> public XDocument ➊CreateSimpleTask(string name, string comment, Guid configID, Guid targetID)
> 
> {
> 
> XDocument createTaskXML = new XDocument(
> 
> new XElement(➋"create_task",
> 
> new XElement("name", name),
> 
> new XElement("comment", comment),
> 
> new XElement("config",
> 
> new XAttribute(➌"id", configID.ToString())),
> 
> new XElement("target",
> 
> new XAttribute("id", targetID.ToString()))));
> 
> return _session.ExecuteCommand(createTaskXML);
> 
> }
> 
> public XDocument ➍StartTask(Guid taskID)
> 
> {
> 
> XDocument startTaskXML = new XDocument(
> 
> new XElement(➎"start_task",
> 
> new XAttribute("task_id", taskID.ToString())));
> 
> return _session.ExecuteCommand(startTaskXML);
> 
> }

Listing 7-16：OpenVAS 方法，用于创建并启动任务

CreateSimpleTask() 方法 ➊ 创建一个带有少量基本信息的新任务。可以创建非常复杂的任务配置。为了进行基本的漏洞扫描，我们构建了一个简单的 XML 文档，根元素是 create_task ➋，并包含一些子元素用于存储配置的相关信息。前两个子元素是任务的名称和注释（或描述）。接下来是扫描配置和目标元素，值作为 id 属性 ➌ 存储。在设置好 XML 后，我们将 create_task 命令发送给 OpenVAS，并返回响应。

StartTask() 方法 ➍ 接受一个参数：要启动的任务 ID。我们首先创建一个名为 start_task ➎ 的 XML 元素，并为其添加 task_id 属性。

Listing 7-17 展示了如何将这两个方法添加到 Main() 中。

> XDocument task = manager.CreateSimpleTask(Guid.NewGuid().ToString(),
> 
> string.Empty, new Guid(discoveryConfigID), new Guid(targetID));
> 
> Guid taskID = new Guid(task.Root.➊Attribute("id").Value);
> 
> manager.➋StartTask(taskID); Listing 7-17：创建并启动一个 OpenVAS 任务

要调用 CreateSimpleTask()，我们传入一个新的 Guid 作为任务名称，一个空字符串作为评论，以及扫描配置 ID 和目标 ID 作为参数。我们从返回的 XML 文档的根节点提取 id 属性 ➊，这是任务 ID；然后我们将其传递给 StartTask() ➋ 来启动 OpenVAS 扫描。

监控扫描并获取扫描结果

为了监控扫描，我们实现了 GetTasks() 和 GetTaskResults() 方法，如 列表 7-18 所示。GetTasks() 方法（先实现）返回一个任务及其状态的列表，这样我们就可以监控扫描直到完成。GetTaskResults() 方法返回给定任务的扫描结果，这样我们就能看到 OpenVAS 找到的任何漏洞。

> public XDocument GetTasks(Guid? taskID = ➊null)
> 
> {
> 
> if (taskID != null)
> 
> return _session.ExecuteCommand(new XDocument(
> 
> new XElement("get_tasks",
> 
> new ➋XAttribute("task_id", taskID.ToString()))));
> 
> return _session.ExecuteCommand(➌XDocument.Parse("<get_tasks />"));
> 
> }
> 
> public XDocument GetTaskResults(Guid taskID)
> 
> {
> 
> XDocument getTaskResultsXML = new XDocument(
> 
> new ➍XElement("get_results",
> 
> new XAttribute("task_id", taskID.ToString())));
> 
> return _session.ExecuteCommand(getTaskResultsXML);
> 
> }

列表 7-18：OpenVASManager 方法，用于获取当前任务列表并检索给定任务的结果

GetTasks() 方法有一个单一的可选参数，默认为 null ➊。GetTasks() 方法将返回所有当前任务，或者仅返回单个任务，具体取决于传入的 taskID 参数是否为 null。如果传入的任务 ID 不为 null，我们会创建一个名为 get_tasks 的新的 XML 元素，并为其添加一个 task_id 属性 ➋，该属性为传入的任务 ID；然后我们将 get_tasks 命令发送给 OpenVAS 并返回响应。如果 ID 为 null，我们会使用 XDocument.Parse() 方法 ➌ 创建一个没有特定 ID 的新的 get_tasks 元素，以便获取任务；然后我们执行命令并返回结果。

GetTaskResults() 方法的工作方式与 GetTasks() 类似，不同之处在于它的唯一参数不是可选的。我们使用传入的 ID 作为参数，创建一个带有 task_id 属性的 get_results XML 节点 ➍。将此 XML 节点传递给 ExecuteCommand() 后，我们返回响应。

完成自动化

列表 7-19 显示了我们如何监控扫描并通过我们刚刚实现的方法获取其结果。在驱动 Session/Manager 类的 Main() 方法中，我们可以添加以下代码来完善我们的自动化。

> XDocument status = manager.➊GetTasks(taskID);
> 
> while (status.➋Descendants("status").First().Value != "Done")
> 
> {
> 
> Thread.Sleep(5000);
> 
> Console.Clear();
> 
> string percentComplete = status.➌Descendants("progress").First().Nodes()
> 
> .OfType<XText>().First().Value;
> 
> Console.WriteLine("扫描已完成 " + percentComplete + "%。");
> 
> status = manager.➍GetTasks(taskID);
> 
> }
> 
> XDocument results = manager.➎GetTaskResults(taskID);
> 
> Console.WriteLine(results.ToString()); 示例 7-19：监视 OpenVAS 扫描直到完成，然后获取扫描结果并打印

我们通过传入之前保存的任务 ID 调用 GetTasks() ➊，然后将结果保存在 status 变量中。接着，我们使用 LINQ to XML 方法 Descendants() ➋来查看 XML 文档中的状态节点是否等于 Done，这意味着扫描已经完成。如果扫描没有完成，我们调用 Sleep()等待五秒钟，然后清空控制台屏幕。然后，我们使用 Descendants() ➌获取进度节点来获取扫描的完成百分比，打印出该百分比，再次通过 GetTasks() ➍请求 OpenVAS 的当前状态，直到扫描报告完成。

一旦扫描完成，我们通过传入任务 ID 调用 GetTaskResults() ➎，然后保存并打印包含扫描结果的 XML 文档到控制台屏幕。该文档包括一系列有用信息，包括检测到的主机和开放端口、扫描主机上已知的活动服务，以及已知的漏洞，如软件的旧版本。

运行自动化

扫描可能需要一段时间，这取决于运行 OpenVAS 的机器和网络速度。在扫描过程中，我们的自动化将显示一条友好的消息，让用户了解当前扫描的状态。成功的输出应该类似于示例 7-20 中展示的简化报告。

> 扫描已完成 1%。
> 
> 扫描已完成 8%。
> 
> 扫描已完成 8%。
> 
> 扫描已完成 46%。
> 
> 扫描已完成 50%。
> 
> 扫描已完成 58%。
> 
> 扫描已完成 72%。
> 
> 扫描已完成 84%。
> 
> 扫描已完成 94%。
> 
> 扫描已完成 98%。
> 
> <get_results_response status="200" status_text="OK">
> 
> <result id="57e9d1fa-7ad9-4649-914d-4591321d061a">
> 
> <owner>
> 
> <name>admin</name>
> 
> </owner>
> 
> --省略--
> 
> </result>
> 
> </get_results_response> 示例 7-20：OpenVAS 自动化的示例输出

结论

本章展示了如何使用 C#内置的网络类来自动化 OpenVAS。你学会了如何与 OpenVAS 建立 SSL 连接以及如何使用基于 XML 的 OMP 进行通信。你学会了如何创建扫描目标，检索可用的扫描配置，并启动针对目标的特定扫描。你还学会了如何监视扫描进度并以 XML 报告的形式检索扫描结果。

有了这些基本模块，我们可以开始修复网络中的漏洞，然后运行新的扫描以确保漏洞不再被报告。OpenVAS 扫描器是一个非常强大的工具，我们仅仅是初步了解它。OpenVAS 不断更新漏洞数据，并且可以作为一个有效的漏洞管理解决方案。

下一步，你可能需要考虑管理通过 SSH 进行认证的漏洞扫描凭据，或创建自定义扫描配置以检查特定的策略配置。通过 OpenVAS，这一切都可以实现，甚至更多。
