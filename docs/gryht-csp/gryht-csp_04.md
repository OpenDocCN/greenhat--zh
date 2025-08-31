# 第五章

5

自动化 Nessus

![](img/00010.jpg)

Nessus 是一个流行且强大的漏洞扫描器，它使用已知漏洞的数据库来评估网络中给定系统是否缺少任何补丁，或者是否易受已知漏洞的攻击。在本章中，我将向你展示如何编写类与 Nessus API 交互，以自动化、配置和执行漏洞扫描。

Nessus 最初作为一个开源漏洞扫描器开发，但在 2005 年被 Tenable Network Security 收购后变为闭源。截至本文写作时，Tenable 提供了一个为期七天的 Nessus Professional 试用版，并且还有一个叫 Nessus Home 的有限版。两者之间最大的区别是，Nessus Home 一次最多只能扫描 16 个 IP 地址，但 Home 版本足以让你运行本章中的示例并熟悉该程序。Nessus 在帮助扫描和管理其他公司网络的专业人士中尤其受欢迎。请按照 Tenable 网站上的说明[`www.tenable.com/products/nessus-home/`](https://www.tenable.com/products/nessus-home/)安装和配置 Nessus Home。

许多组织要求定期进行漏洞和补丁扫描，以便管理和识别其网络上的风险，并满足合规性要求。我们将使用 Nessus 来实现这些目标，通过构建类来帮助我们对网络上的主机执行无认证的漏洞扫描。

REST 与 Nessus API

Web 应用程序和 API 的出现催生了一种叫做 REST API 的架构。REST（表述性状态转移）是一种通过 HTTP 等协议访问和交互资源（如用户账户或漏洞扫描）的方法，通常使用多种 HTTP 方法（GET、POST、DELETE 和 PUT）。HTTP 方法描述了我们发起 HTTP 请求时的意图（例如，我们是想创建资源还是修改资源？），有点像数据库中的 CRUD（创建、读取、更新、删除）操作。

例如，看看以下简单的 GET HTTP 请求，它类似于数据库的读取操作（如 SELECT * FROM users WHERE id = 1）：GET /users/➊1 HTTP/1.0

主机：192.168.0.11

在这个例子中，我们请求 ID 为 1 的用户信息。如果要获取其他用户 ID 的信息，可以将 URI 末尾的 1 ➊替换为该用户的 ID。

要更新第一个用户的信息，HTTP 请求可能如下所示：POST /users/1 HTTP/1.0

主机：192.168.0.11

内容类型：application/json

内容长度：24

{"name": "Brandon Perry"}

在我们假设的 RESTful API 中，上面的 POST 请求会将第一个用户的名称更新为 Brandon Perry。通常，POST 请求用于更新 Web 服务器上的资源。

要完全删除账户，可以使用 DELETE，例如：DELETE /users/1 HTTP/1.0

主机：192.168.0.11

Nessus API 的行为也类似。在使用 API 时，我们将向服务器发送 JSON 并从服务器接收 JSON，如这些示例所示。本章中我们将编写的类旨在处理与 REST API 交互的方式。

一旦你安装了 Nessus，你可以在 https://<IP 地址>:8834/api 找到 Nessus REST API 文档。我们将仅讨论一些用于驱动 Nessus 进行漏洞扫描的核心 API 调用。

NessusSession 类

为了自动化发送命令并接收来自 Nessus 的响应，我们将使用 NessusSession 类创建会话并执行 API 命令，如清单 5-1 所示。

> public class NessusSession : ➊IDisposable
> 
> {
> 
> public ➋NessusSession(string host, string username, string password)
> 
> {
> 
> ServicePointManager.ServerCertificateValidationCallback =
> 
> (Object obj, X509Certificate certificate, X509Chain chain, SslPolicyErrors errors) => true;
> 
> this.Host = ➌host;
> 
> if (➍!Authenticate(username, password))
> 
> throw new Exception("身份验证失败");
> 
> }
> 
> public bool ➎Authenticate(string username, string password)
> 
> {
> 
> JObject obj = ➏new JObject();
> 
> obj["username"] = username;
> 
> obj["password"] = password;
> 
> JObject ret = ➐MakeRequest(WebRequestMethods.Http.Post, "/session", obj);
> 
> if (ret ["token"] == null)
> 
> return false;
> 
> this.➑Token = ret["token"].Value<string>();
> 
> this.Authenticated = true;
> 
> return true;
> 
> }

清单 5-1：NessusSession 类的开头，显示了构造函数和 Authenticate()方法

如清单 5-1 所示，这个类实现了 IDisposable 接口➊，以便我们可以在 using 语句中使用 NessusSession 类。正如你在前面的章节中可能记得的，IDisposable 接口允许我们通过调用 Dispose()方法在垃圾回收时自动清理与 Nessus 的会话，我们将在稍后实现该方法。

在➌处，我们将 Host 属性赋值为传递给 NessusSession 构造函数➋的 host 参数的值，然后我们尝试进行身份验证➍，因为后续的所有 API 调用都需要已认证的会话。如果身份验证失败，我们将抛出异常并打印警告“身份验证失败”。如果身份验证成功，我们将存储 API 密钥以备后用。

在 Authenticate()方法➎中，我们创建了一个 JObject➏来保存作为参数传入的凭证。我们将使用这些凭证尝试进行身份验证，然后调用 MakeRequest()方法➐（接下来讨论）并传递 HTTP 方法、目标主机的 URI 和 JObject。如果身份验证成功，MakeRequest()应该返回一个包含身份验证令牌的 JObject；如果身份验证失败，则返回一个空的 JObject。

当我们收到认证令牌时，我们将其值赋给 Token 属性 ➑，将 Authenticated 属性设置为 true，并返回 true 给调用方法，告诉程序员认证成功。如果认证失败，我们返回 false。

发起 HTTP 请求

MakeRequest() 方法执行实际的 HTTP 请求，并返回响应，如 列表 5-2 所示。

> public JObject MakeRequest(string method, string uri, ➊JObject data = null, string token = null)
> 
> {
> 
> string url = ➋"https://" + this.Host + ":8834" + uri;
> 
> HttpWebRequest request = (HttpWebRequest)WebRequest.Create(url);
> 
> request.➌Method = method;
> 
> 如果 (!string.IsNullOrEmpty(token))
> 
> request.Headers ["X-Cookie"] = ➍"token=" + token;
> 
> request.➎ContentType = "application/json";
> 
> 如果 (data != null)
> 
> {
> 
> byte[] bytes = System.Text.Encoding.ASCII.➏GetBytes(data.ToString());
> 
> request.ContentLength = bytes.Length;
> 
> 使用 (Stream requestStream = request.GetRequestStream())
> 
> requestStream.➐Write(bytes, 0, bytes.Length);
> 
> }
> 
> 否则
> 
> request.ContentLength = 0;
> 
> string response = string.Empty;
> 
> 尝试 ➑
> 
> {
> 
> 使用 (StreamReader reader = new ➒StreamReader(request.GetResponse().GetResponseStream()))
> 
> response = reader.ReadToEnd();
> 
> }
> 
> 捕获
> 
> {
> 
> 返回新的 JObject();
> 
> }
> 
> 如果 (string.IsNullOrEmpty(response))
> 
> 返回新的 JObject();
> 
> 返回 JObject.➓Parse(response);
> 
> }

列表 5-2：来自 NessusSession 类的 MakeRequest() 方法

MakeRequest() 方法有两个必需的参数（HTTP 和 URI）和两个可选参数（JObject 和认证令牌）。每个参数的默认值为 null。

为了创建 MakeRequest() 方法，我们首先通过将主机和 URI 参数结合起来并将结果作为第二个参数传递来创建 API 调用的基本 URL ➋；然后我们使用 HttpWebRequest 构建 HTTP 请求，并将 HttpWebRequest 的 Method 属性 ➌ 设置为传递给 MakeRequest() 方法的 method 变量的值。接下来，我们测试用户是否在 JObject 中提供了认证令牌。如果提供了，我们将 HTTP 请求头 X-Cookie 设置为 token 参数的值 ➍，这是 Nessus 在认证时会查找的内容。我们将 HTTP 请求的 ContentType 属性 ➎ 设置为 application/json，以确保 API 服务器知道如何处理我们在请求体中发送的数据（否则，它将拒绝接受请求）。

如果一个 JObject 被传递给 MakeRequest() 作为第三个参数 ➊，我们会使用 GetBytes() ➏ 将其转换为字节数组，因为 Write() 方法只能写入字节。我们将 ContentLength 属性设置为数组的大小，然后使用 Write() ➐ 将 JSON 写入请求流。如果传递给 MakeRequest() 的 JObject 为 null，我们仅将 ContentLength 设置为 0，然后继续，因为我们不会在请求体中放入任何数据。

在声明了一个空字符串来保存服务器的响应后，我们在 ➑ 处开始一个 try/catch 块来接收响应。在 using 语句中，我们创建一个 StreamReader ➒ 来读取 HTTP 响应，通过将服务器的 HTTP 响应流传递给 StreamReader 构造函数；然后我们调用 ReadToEnd() 来读取完整的响应体到我们的空字符串中。如果读取响应时发生异常，我们可以预期响应体为空，因此我们捕获异常并返回一个空的 JObject 到 ReadToEnd()。否则，我们将响应传递给 Parse() ➓ 并返回结果 JObject。

注销并清理

为了完成 NessusSession 类，我们将创建 LogOut() 方法以注销服务器，并创建 Dispose() 方法来实现 IDisposable 接口，如 Listing 5-3 所示。

> public void ➊LogOut()
> 
> {
> 
> if (this.Authenticated)
> 
> {
> 
> MakeRequest("DELETE", "/session", null, this.Token);
> 
> this.Authenticated = false;
> 
> }
> 
> }
> 
> public void ➋Dispose()
> 
> {
> 
> if (this.Authenticated)
> 
> this.LogOut();
> 
> }
> 
> public string Host { get; set; }
> 
> public bool Authenticated { get; private set; }
> 
> public string Token { get; private set; }
> 
> }

Listing 5-3：NessusSession 类的最后两个方法，以及 Host、Authenticated 和 Token 属性

LogOut() 方法 ➊ 会检查我们是否已通过 Nessus 服务器认证。如果已认证，我们调用 MakeRequest()，并将 DELETE 作为 HTTP 方法；/session 作为 URI；以及认证令牌，这会向 Nessus 服务器发送 DELETE HTTP 请求，从而有效地注销我们。一旦请求完成，我们将 Authenticated 属性设置为 false。为了实现 IDisposable 接口，我们创建 Dispose() ➋ 方法，如果已认证，则注销我们。

测试 NessusSession 类

我们可以通过一个小的 Main() 方法轻松测试 NessusSession 类，如 Listing 5-4 中所示。

> public static void ➊Main(string[] args)
> 
> {
> 
> ➋using (NessusSession session = new ➌NessusSession("192.168.1.14", "admin", "password"))
> 
> {
> 
> Console.➍WriteLine("您的认证令牌是：" + session.Token);
> 
> }
> 
> }

Listing 5-4：测试 NessusSession 类以便与 NessusManager 进行认证

在 Main() 方法 ➊ 中，我们创建一个新的 NessusSession ➌ 并传递 Nessus 主机的 IP 地址、用户名和 Nessus 密码作为参数。通过认证的会话，我们打印出 Nessus 成功认证时给我们的认证令牌 ➍，然后退出。

注意

> NessusSession 是在使用语句 ➋的上下文中创建的，因此我们在 NessusSession 类中实现的 Dispose() 方法将在 using 块结束时自动调用。这会注销 NessusSession，失效我们从 Nessus 获取的认证令牌。

运行此代码应该会打印出一个类似于 Listing 5-5 中的认证令牌。

> $ mono ./ch5_automating_nessus.exe
> 
> 您的认证令牌是：19daad2f2fca99b2a2d48febb2424966a99727c19252966a
> 
> $

Listing 5-5: 运行 NessusSession 测试代码以打印认证令牌

NessusManager 类

Listing 5-6 展示了我们需要在 NessusManager 类中实现的方法，这些方法将为 Nessus 的常见 API 调用和功能提供易于使用的方法，我们稍后可以调用它们。

> public class NessusManager : ➊IDisposable
> 
> {
> 
> NessusSession _session;
> 
> public NessusManager(NessusSession session)
> 
> {
> 
> _session = ➋session;
> 
> }
> 
> public JObject GetScanPolicies()
> 
> {
> 
> return _session.➌MakeRequest("GET", "/editor/policy/templates", null, _session.Token);
> 
> }
> 
> public JObject CreateScan(string policyID, string cidr, string name, string description)
> 
> {
> 
> JObject data = ➍new JObject();
> 
> data["uuid"] = policyID;
> 
> data["settings"] = new JObject();
> 
> data["settings"]["name"] = name;
> 
> data["settings"]["text_targets"] = cidr;
> 
> data["settings"]["description"] = description;
> 
> return _session.➎MakeRequest("POST", "/scans", data, _session.Token);
> 
> }
> 
> public JObject StartScan(int scanID)
> 
> {
> 
> return _session.MakeRequest("POST", "/scans/" + scanID + "/launch", null, _session.Token);
> 
> }
> 
> public JObject ➏GetScan(int scanID)
> 
> {
> 
> return _session.MakeRequest("GET", "/scans/" + scanID, null, _session.Token);
> 
> }
> 
> public void Dispose()
> 
> {
> 
> if (_session.Authenticated)
> 
> _session.➐LogOut();
> 
> _session = null;
> 
> }
> 
> }

Listing 5-6: NessusManager 类

NessusManager 类实现了 IDisposable ➊，这样我们就可以使用 NessusSession 与 Nessus API 进行交互，并在必要时自动注销。NessusManager 的构造函数接受一个参数——一个 NessusSession，并将其分配给私有的 _session 变量 ➋，NessusManager 中的任何方法都可以访问该变量。

Nessus 预配置了几种不同的扫描策略。我们将使用 GetScanPolicies()和 MakeRequest() ➌来从/editor/policy/templates URI 中检索策略及其 ID 的列表。CreateScan()的第一个参数是扫描策略 ID，第二个参数是要扫描的 CIDR 范围。（你也可以在此参数中输入一个以换行符分隔的 IP 地址字符串。）第三个和第四个参数可以分别用于存储扫描的名称和描述。由于我们的扫描仅用于测试目的，我们将为每个名称使用唯一的 Guid（全球唯一标识符，长串唯一的字母和数字），但随着你构建更复杂的自动化流程，可能需要采用一种命名扫描的系统，以便更容易跟踪它们。我们使用传递给 CreateScan()的参数创建一个新的 JObject ➍，该对象包含要创建的扫描的设置。然后我们将这个 JObject 传递给 MakeRequest() ➎，它将向/scans URI 发送一个 POST 请求，并返回关于特定扫描的所有相关信息，显示我们成功创建了（但并未启动！）一个扫描。我们可以使用扫描 ID 来报告扫描的状态。

一旦我们使用 CreateScan()创建了扫描，我们将把它的 ID 传递给 StartScan()方法，该方法会创建一个 POST 请求到/scans/<scanID>/launch URI，并返回 JSON 响应，告诉我们扫描是否已启动。我们可以使用 GetScan() ➏来监控扫描。

为了完成 NessusManager 的实现，我们实现 Dispose()方法以注销会话 ➐，然后通过将 _session 变量设置为 null 来清理资源。

执行 Nessus 扫描

Listing 5-7 显示了如何开始使用 NessusSession 和 NessusManager 来运行扫描并打印结果。

> public static void Main(string[] args)
> 
> {
> 
> ServicePointManager.➊ServerCertificateValidationCallback =
> 
> (Object obj, X509Certificate certificate, X509Chain chain, SslPolicyErrors errors) => true;
> 
> using (NessusSession session = ➋new NessusSession("192.168.1.14", "admin", "password"))
> 
> {
> 
> using (NessusManager manager = new NessusManager(session))
> 
> {
> 
> JObject policies = manager.➌GetScanPolicies();
> 
> string discoveryPolicyID = string.Empty;
> 
> foreach (JObject template in policies["templates"])
> 
> {
> 
> if (template ["name"].Value<string>() == ➍"basic")
> 
> discoveryPolicyID = template ["uuid"].Value<string>();
> 
> }

Listing 5-7: 获取扫描策略列表，以便我们使用正确的扫描策略开始扫描

我们通过首先禁用 SSL 证书验证来开始自动化（因为 Nessus 服务器的 SSL 密钥是自签名的，所以它们会验证失败），方法是将一个仅返回 true 的匿名方法分配给 ServerCertificateValidationCallback ➊。此回调由 HTTP 网络库用于验证 SSL 证书。仅返回 true 会导致接受任何 SSL 证书。接下来，我们创建一个 NessusSession ➋并传入 Nessus 服务器的 IP 地址以及 Nessus API 的用户名和密码。如果认证成功，我们将新的会话传递给另一个 NessusManager。

一旦我们获得了认证会话和管理器，就可以开始与 Nessus 服务器进行交互。我们首先通过 GetScanPolicies() ➌获取可用的扫描策略列表，然后使用 string.Empty 创建一个空字符串来存储基础扫描策略的扫描策略 ID，并遍历扫描策略模板。在遍历扫描策略时，我们检查当前扫描策略的名称是否等于字符串 basic ➍；这是一个很好的起点，用于执行一组小规模的未认证检查，针对网络中的主机。我们将基础扫描策略的 ID 存储起来，以便稍后使用。

现在，使用基础扫描策略 ID 创建并启动扫描，如 Listing 5-8 所示。

> JObject scan = manager.➊CreateScan(discoveryPolicyID, "192.168.1.31",
> 
> "Network Scan", "对单个 IP 地址进行简单扫描。");
> 
> int scanID = ➋scan["scan"]["id"].Value<int>();
> 
> manager.➌StartScan(scanID);
> 
> JObject scanStatus = manager.GetScan(scanID);
> 
> while (scanStatus["info"]["status"].Value<string>() != ➍"completed")
> 
> {
> 
> Console.WriteLine("扫描状态： " + scanStatus["info"]
> 
> ["status"].Value<string>());
> 
> Thread.Sleep(5000);
> 
> scanStatus = manager.➎GetScan(scanID);
> 
> }
> 
> foreach (JObject vuln in scanStatus["vulnerabilities"])
> 
> Console.WriteLine(vuln.ToString());
> 
> }
> 
> }

清单 5-8：Nessus 自动化 Main() 方法的后半部分

在 ➊ 处，我们调用 CreateScan()，传入策略 ID、IP 地址、名称和方法描述，并将其响应存储在 JObject 中。然后，我们从 JObject 中提取扫描 ID ➋，以便将扫描 ID 传递给 StartScan() ➌ 开始扫描。

我们使用 GetScan() 来监控扫描，传入扫描 ID，将结果存储在 JObject 中，并使用 while 循环不断检查当前扫描状态是否已完成 ➍。如果扫描未完成，我们打印其状态，等待五秒钟，然后再次调用 GetScan() ➎。该循环将重复，直到扫描报告完成，此时我们会遍历并打印 GetScan() 返回的每个漏洞，使用 foreach 循环，这可能类似于 清单 5-9。根据你的计算机和网络速度，扫描可能需要几分钟才能完成。

> $ mono ch5_automating_nessus.exe
> 
> 扫描状态：运行中
> 
> 扫描状态：运行中
> 
> 扫描状态：运行中
> 
> --省略--
> 
> {
> 
> "count": 1,
> 
> "plugin_name": ➊"SSL 版本 2 和 3 协议检测",
> 
> "vuln_index": 62,
> 
> "severity": 2,
> 
> "plugin_id": 20007,
> 
> "severity_index": 30,
> 
> "plugin_family": "服务检测"
> 
> }
> 
> {
> 
> "count": 1,
> 
> "plugin_name": ➋"SSL 自签名证书",
> 
> "vuln_index": 61,
> 
> "severity": 2,
> 
> "plugin_id": 57582,
> 
> "severity_index": 31,
> 
> "plugin_family": "通用"
> 
> }
> 
> {
> 
> "count": 1,
> 
> "plugin_name": "SSL 证书无法信任",
> 
> "vuln_index": 56,
> 
> "severity": 2,
> 
> "plugin_id": 51192,
> 
> "severity_index": 32,
> 
> "plugin_family": "通用"
> 
> }

清单 5-9：使用 Nessus 漏洞扫描器进行自动化扫描的部分输出

扫描结果告诉我们，目标使用了弱的 SSL 模式（协议 2 和 3） ➊，以及在开放端口上使用了自签名的 SSL 证书 ➋。我们现在可以确保服务器的 SSL 配置使用了完全最新的 SSL 模式，然后禁用弱模式（或完全禁用该服务）。完成后，我们可以重新运行自动化扫描，确保 Nessus 不再报告使用任何弱 SSL 模式。

结论

本章向你展示了如何自动化 Nessus API 的各个方面，以完成对网络连接设备的无认证扫描。为了实现这一点，我们需要能够向 Nessus HTTP 服务器发送 API 请求。为此，我们创建了 NessusSession 类；然后，一旦能够与 Nessus 进行认证，我们创建了 NessusManager 类来创建、运行并报告扫描结果。我们用代码封装了一切，使用这些类基于用户提供的信息自动驱动 Nessus API。

这并不是 Nessus 所提供功能的全部，你可以在 Nessus API 文档中找到更多详细信息。许多组织需要对网络上的主机执行认证扫描，以获取完整的补丁列表，从而判断主机的健康状况，升级我们的自动化以处理这一需求将是一个很好的练习。
