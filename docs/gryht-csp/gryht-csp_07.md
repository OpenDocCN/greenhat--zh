# 第八章

8

自动化 Cuckoo Sandbox

![](img/00010.jpg)

Cuckoo Sandbox 是一个开源项目，允许你在虚拟机的安全环境中运行恶意软件样本，然后分析并报告恶意软件在虚拟沙箱中的行为，而不必担心恶意软件会感染你的真实机器。Cuckoo Sandbox 是用 Python 编写的，它还提供了一个 REST API，允许程序员使用任何语言来自动化许多 Cuckoo 的功能，例如启动沙箱、运行恶意软件和获取报告。在本章中，我们将使用易于使用的 C# 库和类来完成这一切。但是，在我们开始使用 C# 测试和运行恶意软件样本之前，还需要完成许多工作，比如设置 Cuckoo 使用的虚拟环境。你可以在 [`www.cuckoosandbox.org/`](https://www.cuckoosandbox.org/) 找到更多关于 Cuckoo Sandbox 的信息并进行下载。

设置 Cuckoo Sandbox

本章不会介绍如何设置 Cuckoo Sandbox，因为不同操作系统之间的安装步骤差异较大，甚至在使用哪个版本的 Windows 作为虚拟机沙箱时也会有所不同。本章假设你已经正确设置了带有 Windows 客户机的 Cuckoo Sandbox，并且 Cuckoo 已完全功能正常。请确保遵循 Cuckoo Sandbox 官方网站上的说明（[`docs.cuckoosandbox.org/en/latest/installation/`](http://docs.cuckoosandbox.org/en/latest/installation/)），该网站提供了关于软件安装和配置的最新且详细的文档。

在 Cuckoo Sandbox 附带的 conf/cuckoo.conf 文件中，我建议你在开始使用 API 之前，调整默认的超时配置，使其更短（我将它设置为 15 秒）。这将使测试过程更加简单和快速。在你的 cuckoo.conf 文件中，你会看到底部有一个类似于 Listing 8-1 的部分。

> [timeouts]
> 
> # 设置默认的分析超时时间，单位为秒。这个值将会是
> # 
> # 用来定义分析将在多少秒后终止，除非
> # 
> # 除非在提交时另有说明。
> # 
> 默认 = ➊120

Listing 8-1: cuckoo.conf 文件中的默认超时配置部分

Cuckoo 测试的默认超时设置为 120 秒 ➊。较长的超时可能会让你在调试时变得有些焦急，因为你必须等到超时达到之后才能看到报告，但是将该值设置在 15 到 30 秒之间应该对我们的目的来说足够了。

手动运行 Cuckoo Sandbox API

类似于 Nessus，Cuckoo Sandbox 遵循 REST 模式（如果你需要复习 REST，请参见 第五章 的描述）。然而，Cuckoo Sandbox 的 API 比 Nessus API 简单得多，因为我们只需要与几个 API 端点进行通信。为此，我们将继续使用 session/manager 模式，首先实现 CuckooSession 类，涵盖我们将如何与 Cuckoo Sandbox API 进行通信。在开始编写代码之前，让我们检查一下你是否正确设置了 Cuckoo Sandbox。

启动 API

成功安装 Cuckoo Sandbox 后，你应该能够通过命令 ./cuckoo.py 在本地启动它，如 清单 8-2 所示。如果收到错误信息，请确保你用于测试的虚拟机正在运行。

> $ ./cuckoo.py
> 
> eeee e e eeee e e eeeee eeeee
> 
> 8 8 8 8 8 8 8 8 8 88 8 88
> 
> 8e 8e 8 8e 8eee8e 8 8 8 8
> 
> 88 88 8 88 88 8 8 8 8 8
> 
> 88e8 88ee8 88e8 88 8 8eee8 8eee8
> 
> Cuckoo Sandbox 2.0-rc2
> 
> www.cuckoosandbox.org
> 
> 版权所有 (c) 2010-2015
> 
> 检查更新中...
> 
> 好的！你已经拥有最新版本。
> 
> 2016-05-19 16:17:06,146 [lib.cuckoo.core.scheduler] 信息：使用 "virtualbox" 作为机器管理器
> 
> 2016-05-19 16:17:07,484 [lib.cuckoo.core.scheduler] 信息：已加载 1 台机器
> 
> 2016-05-19 16:17:07,495 [lib.cuckoo.core.scheduler] 信息：等待分析任务...

清单 8-2：启动 Cuckoo Sandbox 管理器

成功启动 Cuckoo 后，应该会显示一个有趣的 ASCII 艺术横幅，随后是一些快速信息，显示已加载了多少虚拟机。启动主 Cuckoo 脚本后，你需要启动我们将要进行通信的 API。这两个 Python 脚本必须同时运行！cuckoo.py Python 脚本是 Cuckoo Sandbox 的引擎。如果我们在没有启动 cuckoo.py 脚本的情况下启动 api.py 脚本，如 清单 8-3 所示，那么我们的 API 请求将不会执行任何操作。为了通过 API 使用 Cuckoo Sandbox，cuckoo.py 和 api.py 必须同时运行。默认情况下，Cuckoo Sandbox API 监听 8090 端口，如 清单 8-3 所示。

> $ utils/api.py ➊-H 0.0.0.0
> 
> * 正在运行在 ➋http://0.0.0.0:8090/（按 CTRL+C 退出）

清单 8-3：运行 Cuckoo Sandbox 的 HTTP API

要指定监听的 IP 地址（默认是 localhost），你可以通过 utils/api.py 脚本传递 -H 参数 ➊，该参数告诉 API 在监听 API 请求时使用哪个 IP 地址。在此案例中，我们将 0.0.0.0 设置为监听的 IP 地址，这意味着所有网络接口（包括系统的内部和外部 IP 地址）都将有 8090 端口可用进行通信，因为我们使用的是默认端口。Cuckoo API 监听的 URL 在启动后也会打印到屏幕上 ➋。这个 URL 是我们与 API 通信，驱动 Cuckoo Sandbox 进行后续操作的方式。

检查 Cuckoo 的状态

我们可以使用 curl 命令行工具测试 API 是否正确设置，就像我们在前几章中为其他 API 做的一样。在本章后面，我们会发出类似的 API 请求来创建任务，观察任务直到完成，并报告文件以查看它在运行时的行为。但在开始时，列表 8-4 展示了如何使用 curl 通过 HTTP API 以 JSON 格式获取 Cuckoo Sandbox 状态信息。

> $ curl http://127.0.0.1:8090/cuckoo/status
> 
> {
> 
> "cpuload": [
> 
> 0.0,
> 
> 0.02,
> 
> 0.05
> 
> ],
> 
> "diskspace": {
> 
> "analyses": {
> 
> "free": 342228357120,
> 
> "total": 486836101120,
> 
> "used": 144607744000
> 
> },
> 
> "binaries": {
> 
> "free": 342228357120,
> 
> "total": 486836101120,
> 
> "used": 144607744000
> 
> }
> 
> },
> 
> "hostname": "fdsa-E7450",
> 
> ➊"machines": {
> 
> "available": 1,
> 
> "total": 1
> 
> },
> 
> "memory": 82.06295645686164,
> 
> ➋"tasks": {
> 
> "completed": 0,
> 
> "pending": 0,
> 
> "reported": 3,
> 
> "running": 0,
> 
> "total": 13
> 
> },
> 
> ➌"version": "2.0-rc2"
> 
> }

列表 8-4：使用 curl 通过 HTTP API 获取 Cuckoo Sandbox 状态

状态信息非常有用，详细描述了 Cuckoo Sandbox 系统的多个方面。值得注意的是汇总任务信息➋，其中列出了 Cuckoo 已运行或正在运行的任务数量，按状态分类。任务可能是分析正在运行的文件，或者是打开带有 URL 的网页，尽管本章只会介绍提交文件进行分析。你还可以看到用于分析的虚拟机数量➊和当前 Cuckoo 的版本➌。

很棒，API 已启动并运行！我们稍后会使用相同的状态 API 端点来测试我们编写的代码，并更详细地讨论它返回的 JSON 数据。目前，我们只需要确认 API 已经启动并运行。

创建 CuckooSession 类

现在我们知道 API 工作正常，可以发送 HTTP 请求并获取 JSON 响应，接下来我们可以开始编写代码来以编程方式驱动 Cuckoo Sandbox。一旦构建了基础类，我们就可以提交一个文件进行分析，分析文件运行时的行为并报告结果。我们从 CuckooSession 类开始，代码见列表 8-5。

> public class ➊CuckooSession
> 
> {
> 
> public CuckooSession➋(string host, int port)
> 
> {
> 
> this.Host = host;
> 
> this.Port = port;
> 
> }
> 
> public string ➌Host { get; set; }
> 
> public int ➍Port { get; set; }

列表 8-5：启动 CuckooSession 类

为了简单起见，我们首先创建 CuckooSession 类➊以及 CuckooSession 构造函数。构造函数接受两个参数➋，第一个是要连接的主机，第二个是主机上 API 监听的端口。在构造函数中，传入的两个参数值被分配给相应的属性 Host ➌和 Port ➍，这些属性在构造函数下方定义。接下来，我们需要实现 CuckooSession 类中的可用方法。

编写 ExecuteCommand()方法以处理 HTTP 请求

Cuckoo 期望在 API 请求时收到两种 HTTP 请求：一种是传统的 HTTP 请求，另一种是用于将文件发送到 Cuckoo 进行分析的更复杂的 HTTP 多部分表单请求。我们将实现两个 ExecuteCommand() 方法来涵盖这些请求类型：首先，我们将使用一个简单的 ExecuteCommand() 方法，接受两个参数用于传统请求，然后我们将通过重载它，创建一个接受三个参数用于多部分请求的 ExecuteCommand() 方法。在 C# 中，允许创建具有相同名称但不同参数的方法，也就是方法重载。这是一个典型的例子，展示了在方法重载时的应用场景，而不是使用一个接受可选参数的单一方法，因为尽管方法名相同，但每种请求的方法相对不同。更简单的 ExecuteCommand() 方法在 Listing 8-6 中有详细说明。

> public JObject ➊ExecuteCommand(string uri, string method)
> 
> {
> 
> HttpWebRequest req = (HttpWebRequest)WebRequest
> 
> .➋Create("http://" + this.Host + ":" + this.Port + uri);
> 
> req.➌Method = method;
> 
> string resp = string.Empty;
> 
> using (Stream str = req.GetResponse().GetResponseStream())
> 
> using (StreamReader rdr = new StreamReader(str))
> 
> resp = rdr.➍ReadToEnd();
> 
> JObject obj = JObject.➎Parse(resp);
> 
> return obj;
> 
> }

Listing 8-6: 更简单的 ExecuteCommand() 方法，它仅接受 URI 和 HTTP 方法作为参数

第一个 ExecuteCommand() 方法 ➊ 接受两个参数：请求的 URI 和要使用的 HTTP 方法（如 GET、POST、PUT 等）。在使用 Create() ➋ 构建新的 HTTP 请求并设置请求的 Method 属性 ➌ 后，我们发出 HTTP 请求并读取 ➍ 响应到一个字符串中。最后，我们将返回的字符串解析 ➎ 为 JSON，并返回新的 JSON 对象。

重载的 ExecuteCommand() 方法接受三个参数：请求的 URI、HTTP 方法和一个字典，字典包含将通过 HTTP 多部分请求发送的参数。多部分请求允许你发送更复杂的数据，如二进制文件以及其他 HTTP 参数到 Web 服务器，这正是我们将要使用的方式。一个完整的多部分请求将在 Listing 8-9 中展示。如何发送这种类型的请求将在 Listing 8-7 中详细说明。

> public JObject ➊ExecuteCommand(string uri, string method, IDictionary<string, object> parms)
> 
> {
> 
> HttpWebRequest req = (HttpWebRequest)WebRequest
> 
> .➋Create("http://" + this.Host + ":" + this.Port + uri);
> 
> req.➌Method = method;
> 
> string boundary = ➍String.Format("----------{0:N}", Guid.NewGuid());
> 
> byte[] data = ➎GetMultipartFormData(parms, boundary);
> 
> req.ContentLength = data.Length;
> 
> req.ContentType = ➏"multipart/form-data; boundary=" + boundary;
> 
> using (Stream parmStream = req.GetRequestStream())
> 
> parmStream.➐Write(data, 0, data.Length);
> 
> string resp = string.Empty;
> 
> using (Stream str = req.GetResponse().GetResponseStream())
> 
> using (StreamReader rdr = new StreamReader(str))
> 
> resp = rdr.➑ReadToEnd();
> 
> JObject obj = JObject.➒Parse(resp);
> 
> return obj;
> 
> }

示例 8-7：重载的 `ExecuteCommand()` 方法，它发起一个 multipart/form-data HTTP 请求

第二个，更复杂的 `ExecuteCommand()` 方法 ➊ 接受三个参数，如前所述。实例化一个新的请求 ➋ 并设置 HTTP 方法 ➌ 后，我们创建一个边界，边界将用于在多部分表单请求中分隔 HTTP 参数，使用 `String.Format()`  ➍。一旦边界创建完成，我们调用 `GetMultipartFormData()` ➎（我们稍后将实现）来将作为第三个参数传递的参数字典转换为一个带有新边界的多部分 HTTP 表单。

在构建完多部分 HTTP 数据后，我们需要通过设置基于多部分 HTTP 数据的 `ContentLength` 和 `ContentType` 请求属性来设置 HTTP 请求。对于 `ContentType` 属性，我们还需要附加用于分隔 HTTP 参数的边界 ➏。最后，我们可以将 ➐ 多部分表单数据写入 HTTP 请求流并读取 ➑ 来自服务器的响应。通过从服务器获取最终响应后，我们将响应解析 ➒ 为 JSON，然后返回 JSON 对象。

这两个 `ExecuteCommand()` 方法将用于执行针对 Cuckoo Sandbox API 的 API 调用。但在我们开始调用 API 端点之前，我们需要再写一些代码。

使用 `GetMultipartFormData()` 方法创建多部分 HTTP 数据

尽管 `GetMultipartFormData()` 方法是与 Cuckoo Sandbox 通信的核心，但我不会逐行讲解它。这个方法实际上是 C# 核心库中一个小缺陷的好例子，因为制作一个多部分 HTTP 请求不应该这么复杂。不幸的是，目前没有一个易于使用的类可以帮助我们完成这个操作，所以我们需要创建这个方法，从头开始构建 HTTP 多部分请求。构建多部分 HTTP 请求的技术细节有些超出了我们要实现的目标，所以我只会简单概述这个方法的基本流程。完整的方法（见示例 8-8，去除了内联注释）是由 Brian Grinstead 编写的^(1)，他的工作后来被纳入了 RestSharp 客户端 ([`restsharp.org/`](http://restsharp.org/))。

> private byte[] ➊GetMultipartFormData(IDictionary<string, object> postParameters, string boundary)
> 
> {
> 
> System.Text.Encoding encoding = System.Text.Encoding.ASCII;
> 
> Stream formDataStream = new System.IO.MemoryStream();
> 
> bool needsCLRF = false;
> 
> foreach (var param in postParameters)
> 
> {
> 
> if (needsCLRF)
> 
> formDataStream.Write(encoding.GetBytes("\r\n"), 0, encoding.GetByteCount("\r\n"));
> 
> needsCLRF = true;
> 
> if (param.Value is FileParameter)
> 
> {
> 
> FileParameter fileToUpload = (FileParameter)param.Value;
> 
> string header = string.Format("--{0}\r\nContent-Disposition: form-data; name=\"{1}\";" +
> 
> "filename=\"{2}\";\r\nContent-Type: {3}\r\n\r\n",
> 
> boundary,
> 
> param.Key,
> 
> fileToUpload.FileName ?? param.Key,
> 
> fileToUpload.ContentType ?? "application/octet-stream");
> 
> formDataStream.Write(encoding.GetBytes(header), 0, encoding.GetByteCount(header));
> 
> formDataStream.Write(fileToUpload.File, 0, fileToUpload.File.Length);
> 
> }
> 
> else
> 
> {
> 
> string postData = string.Format("--{0}\r\nContent-Disposition: form-data;" +
> 
> "name=\"{1}\"\r\n\r\n{2}",
> 
> boundary,
> 
> param.Key,
> 
> param.Value);
> 
> formDataStream.Write(encoding.GetBytes(postData), 0, encoding.GetByteCount(postData));
> 
> }
> 
> }
> 
> string footer = "\r\n--" + boundary + "--\r\n";
> 
> formDataStream.Write(encoding.GetBytes(footer), 0, encoding.GetByteCount(footer));
> 
> formDataStream.Position = 0;
> 
> byte[] formData = new byte[formDataStream.Length];
> 
> formDataStream.Read(formData, 0, formData.Length);
> 
> formDataStream.Close();
> 
> return formData;
> 
> }
> 
> }

Listing 8-8: The  GetMultipartFormData()  method

在`GetMultipartFormData()`方法➊中，我们首先接受两个参数：第一个是参数及其各自值的字典，我们将把这些转换为一个 multipart 表单；第二个是用于分隔请求中文件参数的字符串，以便它们可以被解析。这个第二个参数叫做`boundary`，我们用它告诉 API 使用这个边界分隔 HTTP 请求体，然后将每个部分作为请求中的独立参数和值。这个过程可能难以想象，所以示例 8-9 详细介绍了一个示例 HTTP 多部分表单请求。

> POST / HTTP/1.1
> 
> Host: localhost:8000
> 
> User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:29.0) Gecko/20100101 Firefox/29.0
> 
> Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
> 
> Accept-Language: en-US,en;q=0.5
> 
> Accept-Encoding: gzip, deflate
> 
> Connection: keep-alive
> 
> Content-Type: ➊multipart/form-data;
> 
> boundary➋=------------------------9051914041544843365972754266
> 
> Content-Length: 554
> 
> --------------------------9051914041544843365972754266➌
> 
> Content-Disposition: form-data; ➍name="text"
> 
> text default➎
> 
> --------------------------9051914041544843365972754266➏
> 
> Content-Disposition: form-data; name="file1"; filename="a.txt"
> 
> Content-Type: text/plain
> 
> Content of a.txt.
> 
> --------------------------9051914041544843365972754266➐
> 
> Content-Disposition: form-data; name="file2"; filename="a.html"
> 
> Content-Type: text/html
> 
> <!DOCTYPE html><title>Content of a.html.</title>
> 
> --------------------------9051914041544843365972754266--➑

Listing 8-9: A sample HTTP multipart form request

这个 HTTP 请求看起来与我们正在尝试构建的请求非常相似，因此让我们指出在 GetMultipartFormData() 中提到的重要部分。首先，注意 Content-Type 头部是 multipart/form-data ➊，并且有一个边界 ➋，就像我们在示例 8-7 中设置的那样。这个边界将在整个 HTTP 请求中使用（➌、➏、➐、➑）来分隔每个 HTTP 参数。每个参数也都有一个参数名 ➍ 和一个参数值 ➎。GetMultipartFormData() 方法接受我们在字典参数中传递的参数名和值，以及边界，然后使用给定的边界将它们转换为类似的 HTTP 请求，以分隔每个参数。

使用 FileParameter 类处理文件数据

为了将我们想要分析的文件或恶意软件发送给 Cuckoo，我们需要创建一个类来存储文件的数据，例如文件类型、文件名和文件的实际内容。简单的 FileParameter 类封装了我们需要为 GetMultipartFormData() 方法提供的信息的一部分。它在示例 8-10 中展示。

> public class ➊FileParameter
> 
> {
> 
> public byte[] File { get; set; }
> 
> public string FileName { get; set; }
> 
> public string ContentType { get; set; }
> 
> public ➋FileParameter(byte[] file, string filename, string contenttype)
> 
> {
> 
> ➌File = file;
> 
> ➍FileName = filename;
> 
> ➎ContentType = contenttype;
> 
> }
> 
> }

示例 8-10: FileParameter 类

FileParameter 类 ➊ 表示我们需要构建一个 HTTP 参数，该参数将包含要分析的文件。该类的构造函数 ➋ 接受三个参数：包含文件内容的字节数组、文件名和内容类型。每个参数随后会被分配给相应的类属性（➌、➍、➎）。

测试 CuckooSession 和支持类

我们可以使用一个简短且简单的 Main() 方法来测试到目前为止所写的内容，该方法通过 API 请求 Cuckoo Sandbox 的状态。我们之前在 “检查 Cuckoo 状态”（第 149 页）中手动做过这件事。示例 8-11 展示了我们如何使用新的 CuckooSession 类来做到这一点。

> public static void ➊Main(string[] args)
> 
> {
> 
> CuckooSession session = new ➋CuckooSession("127.0.0.1", 8090);
> 
> JObject response = session.➌ExecuteCommand("/cuckoo/status", "GET");
> 
> Console.➍WriteLine(response.ToString());
> 
> }

示例 8-11: 用于检索 Cuckoo Sandbox 状态的 Main() 方法

使用新的 Main() 方法 ➊，我们首先通过传递 Cuckoo Sandbox 运行的 IP 地址和端口来创建一个 CuckooSession 对象 ➋。如果 API 运行在本地机器上，IP 地址应该可以使用 127.0.0.1。IP 地址和端口（默认端口为 8090）应该在我们启动 API 时在示例 8-3 中已经设置过了。使用新的会话，我们调用 ExecuteCommand() 方法 ➌，传入 URI /cuckoo/status 作为第一个参数，HTTP 方法 GET 作为第二个参数。然后，响应通过 WriteLine() ➍ 打印到屏幕上。

运行 Main() 方法应该会在屏幕上打印一个 JSON 字典，包含 Cuckoo 的状态信息，具体细节见 Listing 8-12。  

> $ ./ch8_automating_cuckoo.exe  
> 
> {  
> 
> "cpuload": [  
> 
> 0.0,  
> 
> 0.03,  
> 
> 0.05  
> 
> ],  
> 
> "diskspace": {  
> 
> "analyses": {  
> 
> "free": 342524416000,  
> 
> "total": 486836101120,  
> 
> "used": 144311685120  
> 
> },  
> 
> "binaries": {  
> 
> "free": 342524416000,  
> 
> "total": 486836101120,  
> 
> "used": 144311685120  
> 
> }  
> 
> },  
> 
> "hostname": "fdsa-E7450",  
> 
> "machines": {  
> 
> "available": 1,  
> 
> "total": 1  
> 
> },  
> 
> "memory": 85.542549616647932,  
> 
> "tasks": {  
> 
> "completed": 0,  
> 
> "pending": 0,  
> 
> "reported": 2,  
> 
> "running": 0,  
> 
> "total": 12  
> 
> },  
> 
> "version": "2.0-rc2"  
> 
> }  

Listing 8-12: 测试 CuckooSession 类以打印 Cuckoo Sandbox 的当前状态信息  

你可以看到，这里打印的 JSON 信息与我们之前手动运行 API 命令检查 Cuckoo 状态时得到的相同。  

编写 CuckooManager 类  

在实现了 CuckooSession 类和其他支持类之后，我们可以继续编写 CuckooManager 类，它将封装一些简单的 API 调用。要开始 CuckooManager 类，我们需要构造函数，如 Listing 8-13 所示。  

> public class ➊CuckooManager : ➋IDisposable  
> 
> {  
> 
> CuckooSession ➌_session = null;  
> 
> public ➍CuckooManager(CuckooSession session)  
> 
> {  
> 
> ➎_session = session;  
> 
> }  

Listing 8-13: 启动 CuckooManager 类  

CuckooManager 类 ➊ 首先实现 IDisposable 接口 ➋，我们将利用该接口来处理私有的 _session 变量 ➌，当我们完成 CuckooManager 类的使用时。类的构造函数 ➍ 仅接受一个参数：与 Cuckoo Sandbox 实例进行通信时使用的会话。私有的 _session 变量会赋值为传递给构造函数的参数 ➎，这样我们接下来编写的方法就可以使用该会话来进行特定的 API 调用。  

编写 CreateTask() 方法  

CuckooManager 类中的第一个方法是 CreateTask()，它是我们编写的最复杂的管理方法。CreateTask() 方法实现了 HTTP 调用，该调用会根据我们要创建的任务类型，确定并执行正确的 HTTP 调用，具体见 Listing 8-14。  

> public int ➊CreateTask(Task task)  
> 
> {  
> 
> string param = null, uri = "/tasks/create/";  
> 
> object val = null;  
> 
> if ➋(task is FileTask)  
> 
> {  
> 
> byte[] 数据;  
> 
> using (FileStream str = new ➌FileStream((task as FileTask).Filepath,  
> 
> FileMode.Open,  
> 
> FileAccess.Read))  
> 
> {  
> 
> data = new byte[str.Length];  
> 
> str.➍Read(data, 0, data.Length);  
> 
> }  
> 
> param = "file";  
> 
> uri += param;  
> 
> val = new ➎FileParameter(data, (task as FileTask).Filepath,  
> 
> "application/binary");
> 
> }  
> 
> IDictionary<string, object> ➏parms = new Dictionary<string, object>();  
> 
> parms.Add(param, val);  
> 
> parms.Add("package", task.Package);  
> 
> parms.Add("timeout", task.Timeout.ToString());  
> 
> parms.Add("options", task.Options);  
> 
> parms.Add("machine", ➐task.Machine);  
> 
> parms.Add("platform", task.Platform);  
> 
> parms.Add("custom", task.Custom);  
> 
> parms.Add("memory", task.EnableMemoryDump.ToString());  
> 
> parms.Add("enforce_timeout", task.EnableEnforceTimeout.ToString());
> 
> JObject resp = _session.➑ExecuteCommand(uri, "POST", parms);
> 
> return ➒(int)resp["task_id"];
> 
> }

列表 8-14：CreateTask() 方法

CreateTask() 方法 ➊ 首先检查传入的任务是否为 FileTask 类 ➋（用于描述要分析的文件或恶意软件的类）。由于 Cuckoo Sandbox 不仅支持分析文件（例如 URL），因此 CreateTask() 方法可以很容易地扩展为创建不同类型的任务。如果任务是 FileTask，我们会用新的 FileStream() ➌ 打开要发送到 Cuckoo Sandbox 的文件，并将文件读取到字节数组中。文件读取完成后 ➍，我们使用新的 FileParameter 类 ➎ 来创建文件名、文件字节和内容类型为 application/binary 的参数。

然后，我们在新的 Dictionary ➏ 中设置将发送到 Cuckoo Sandbox 的 HTTP 参数。这些 HTTP 参数在 Cuckoo Sandbox API 文档中有说明，并应包含创建任务所需的信息。这些参数允许我们更改默认配置项，例如选择使用哪个虚拟机 ➐。最后，我们通过调用 ExecuteCommand() ➑ 并使用字典中的参数来创建新任务，然后返回 ➒ 新的任务 ID。

任务详情和报告方法

为了能够提交我们的文件进行分析和报告，还需要支持更多的 API 调用，但它们比 CreateTask() 要简单得多，如列表 8-15 所述。我们只需要创建一个方法来显示任务详情，两个方法来报告我们的任务，还有一个方法来清理我们的会话。

> public Task ➊GetTaskDetails(int id)
> 
> {
> 
> string uri = ➋"/tasks/view/" + id;
> 
> JObject resp = _session.➌ExecuteCommand(uri, "GET");
> 
> ➍return TaskFactory.CreateTask(resp["task"]);
> 
> }
> 
> public JObject ➎GetTaskReport(int id)
> 
> {
> 
> return GetTaskReport(id, ➏"json");
> 
> }
> 
> public JObject ➐GetTaskReport(int id, string type)
> 
> {
> 
> string uri = ➑"/tasks/report/" + id + "/" + type;
> 
> return _session.➒ExecuteCommand(uri, "GET");
> 
> }
> 
> public void ➓Dispose()
> 
> {
> 
> _session = null;
> 
> }
> 
> }

列表 8-15：用于检索任务信息和报告的辅助方法

我们实现的第一个方法是 GetTaskDetails() 方法 ➊，它以任务 ID 作为唯一参数传入变量 id。我们首先通过将 ID 参数附加到 /tasks/view ➋ 来创建将进行 HTTP 请求的 URI，然后使用新的 URI 调用 ExecuteCommand() ➌。此端点返回有关任务的一些信息，例如运行任务的虚拟机名称和任务的当前状态，我们可以用这些信息来监控任务直到它完成。最后，我们使用 TaskFactory.CreateTask() 方法 ➍ 将 API 返回的 JSON 任务转换为 C# 的 Task 类，我们将在下一节中创建该类。

第二个方法是一个简单的便利方法 ➎。由于 Cuckoo Sandbox 支持多种报告类型（JSON、XML 等），因此有两个 GetTaskReport() 方法，第一个只用于 JSON 报告。它只接受要获取报告的任务 ID 作为参数，并调用其重载方法，传入相同的 ID，并指定第二个参数表明应该返回 JSON ➏ 报告。在第二个 GetTaskReport() 方法 ➐ 中，任务 ID 和报告类型作为参数传递，然后用于构建将在 API 调用中请求的 URI ➑。新的 URI 被传递给 ExecuteCommand() 方法 ➒，并返回来自 Cuckoo Sandbox 的报告。

最后，实现了 Dispose() 方法 ➓，它完成了 IDisposable 接口。该方法清理了我们与 API 通信时使用的会话，并将私有的 _session 变量赋值为 null。

创建任务抽象类

支持 CuckooSession 和 CuckooManager 类的是 Task 类，它是一个抽象类，存储了给定任务的大部分相关信息，以便可以作为属性轻松访问。清单 8-16 详细介绍了抽象的 Task 类。

> public abstract class ➊Task
> 
> {
> 
> protected ➋Task(JToken token)
> 
> {
> 
> if (token != null)
> 
> {
> 
> this.AddedOn = ➌DateTime.Parse((string)token["added_on"]);
> 
> if (token["completed_on"].Type != JTokenType.Null)
> 
> this.CompletedOn = ➍DateTime.Parse(token["completed_on"].ToObject<string>());
> 
> this.Machine = (string)token["machine"];
> 
> this.Errors = token["errors"].ToObject<ArrayList>();
> 
> this.Custom = (string)token["custom"];
> 
> this.EnableEnforceTimeout = (bool)token["enforce_timeout"];
> 
> this.EnableMemoryDump = (bool)token["memory"];
> 
> this.Guest = token["guest"];
> 
> this.ID = (int)token["id"];
> 
> this.Options = token["options"].ToString();
> 
> this.Package = (string)token["package"];
> 
> this.Platform = (string)token["platform"];
> 
> this.Priority = (int)token["priority"];
> 
> this.SampleID = (int)token["sample_id"];
> 
> this.Status = (string)token["status"];
> 
> this.Target = (string)token["target"];
> 
> this.Timeout = (int)token["timeout"];
> 
> }
> 
> }
> 
> public string Package { get; set; }
> 
> public int Timeout { get; set; }
> 
> public string Options { get; set; }
> 
> public string Machine { get; set; }
> 
> public string Platform { get; set; }
> 
> public string Custom { get; set; }
> 
> public bool EnableMemoryDump { get; set; }
> 
> public bool EnableEnforceTimeout { get; set; }
> 
> public ArrayList Errors { get; set; }
> 
> public string Target { get; set; }
> 
> public int SampleID { get; set; }
> 
> public JToken Guest { get; set; }
> 
> public int Priority { get; set; }
> 
> public string Status { get; set;}
> 
> public int ID { get; set; }
> 
> public DateTime AddedOn { get; set; }
> 
> public DateTime CompletedOn { get; set; }
> 
> }

清单 8-16：抽象的 Task 类

尽管抽象的 Task 类 ➊ 起初看起来很复杂，但该类实际上只有一个构造函数和十几个属性。构造函数 ➋ 接受一个 JToken 作为参数，这是一个特殊的 JSON 类，如 JObject。JToken 用于将来自 JSON 的所有任务细节分配给类中的 C#属性。在构造函数中，第一个赋值的属性是 AddedOn 属性。使用 DateTime.Parse() ➌，任务创建时的时间戳将从字符串解析为 DateTime 类，并分配给 AddedOn 属性。如果任务已完成，CompletedOn 属性也会使用 DateTime.Parse() ➍进行同样的操作。其余的属性则直接使用传递给构造函数的 JSON 中的值进行赋值。

排序和创建不同的类类型

Cuckoo Sandbox 支持多种类型的任务，尽管我们只实现了其中一种（文件分析任务）。FileTask 类将从抽象的 Task 类继承，但它添加了一个新属性，用于存储我们希望发送给 Cuckoo 分析的文件路径。Cuckoo 支持的另一种任务是 URL 任务，它会在 web 浏览器中打开给定的 URL 并分析发生了什么（以防该网站存在 drive-by 攻击或其他恶意软件）。

创建 FileTask 类以执行文件分析任务

FileTask 类将用于存储启动文件分析所需的信息。如 Listing 8-17 所示，它简洁明了，因为它继承了我们刚刚实现的 Task 类的大部分属性。

> public class ➊FileTask : Task
> 
> {
> 
> public ➋FileTask() : base(null) { }
> 
> public ➌FileTask(JToken dict) : base(dict) { }
> 
> public ➍string Filepath { get; set; }
> 
> }

Listing 8-17：继承自 Task 的 FileTask 类

简单的 FileTask 类 ➊，继承自之前的 Task 类，使用了 C#中一些高级的继承技巧。该类实现了两个不同的构造函数，两个构造函数都会将参数传递给基类 Task 的构造函数。例如，第一个构造函数 ➋ 不接受任何参数，并将 null 值传递给基类的构造函数。这使得我们可以为类保留一个不需要任何参数的默认构造函数。第二个构造函数 ➌ 接受一个 JToken 类作为唯一参数，并将 JSON 参数直接传递给基类构造函数，后者将填充 FileTask 类从 Task 继承的属性。这使得使用来自 Cuckoo API 的 JSON 轻松设置 FileTask。FileTask 类中唯一的属性是 Filepath ➍，这是提交文件分析任务时才有用的属性，在通用 Task 类中没有这个属性。

使用 TaskFactory 类来确定要创建的任务类型

Java 开发者或其他熟悉面向对象编程的人可能已经知道，工厂模式是面向对象开发中常用的设计模式。这是一种灵活的方式，通过一个类来管理许多相似但最终不同类型的类的创建（通常这些类都继承自同一个基类，但它们也可以实现相同的接口）。TaskFactory 类（见清单 8-18）用于将 Cuckoo Sandbox 返回的 JSON 任务转化为我们的 C# Task 类，无论是 FileTask 还是其他类型——也就是说，如果你选择进一步实现我们为作业描述的 URL 任务！

> public static class ➊TaskFactory
> 
> {
> 
> public static Task ➋CreateTask(JToken dict)
> 
> {
> 
> Task task = null;
> 
> ➌switch((string)dict["category"])
> 
> {
> 
> case ➍"file":
> 
> task = new ➎FileTask(dict);
> 
> break;
> 
> default:
> 
> throw new Exception("未知类别: " + dict["category"]);
> 
> }
> 
> return ➏task;
> 
> }
> 
> }

清单 8-18：TaskFactory 静态类，它实现了一个常见的简单工厂模式，通常用于面向对象编程

我们要实现的最终类是 TaskFactory 静态类 ➊。这个类是将 Cuckoo Sandbox 中的 JSON 任务转换为 C# 的 FileTask 对象的关键——如果你将来选择实现其他任务类型，也可以使用 TaskFactory 来处理这些任务的创建。TaskFactory 类只有一个静态方法 CreateTask() ➋，它接受一个 JToken 作为唯一参数。在 CreateTask() 方法中，我们使用 switch 语句 ➌ 来测试任务类别的值。如果类别是文件任务 ➍，我们将 JToken 任务传递给 FileTask 构造函数 ➎，然后返回新的 C# 任务 ➏。尽管本书中我们不会使用其他文件类型，但你可以使用这个 switch 语句根据任务类别创建不同类型的任务，例如基于类别的 URL 任务，然后返回结果。

整合起来

最后，我们已经搭建好了框架，开始自动化一些恶意软件分析。清单 8-19 演示了如何使用 CuckooSession 和 CuckooManager 类来创建一个文件分析任务，监视任务直到完成，并将任务的 JSON 报告打印到控制台。

> public static void ➊Main(string[] args)
> 
> {
> 
> CuckooSession session = new ➋CuckooSession("127.0.0.1", 8090);
> 
> using (CuckooManager manager = new ➌CuckooManager(session))
> 
> {
> 
> FileTask task = new ➍FileTask();
> 
> task.➎Filepath = "/var/www/payload.exe";
> 
> int taskID = manager.➏CreateTask(task);
> 
> Console.WriteLine("创建任务: " + taskID);
> 
> task = (FileTask)manager.➐GetTaskDetails(taskID);
> 
> while(task.Status == "pending" || task.Status == "running")
> 
> {
> 
> Console.WriteLine("等待 30 秒..." + task.Status);
> 
> System.Threading.Thread.Sleep(30000);
> 
> task = (FileTask)manager.GetTaskDetails(taskID);
> 
> }
> 
> if (task.➑Status == "failure")
> 
> {
> 
> Console.Error.WriteLine("发生错误：");
> 
> foreach (var error in task.Errors)
> 
> Console.Error.WriteLine(error);
> 
> return;
> 
> }
> 
> string report = manager.➒GetTaskReport(taskID).ToString();
> 
> Console.➓WriteLine(report);
> 
> }
> 
> }

列表 8-19: Main() 方法将 CuckooSession 和 CuckooManager 类结合起来

在 Main() 方法 ➊ 中，我们首先创建一个新的 CuckooSession 实例 ➋，传入用于 API 请求的 IP 地址和端口。创建新的会话后，在 using 语句的上下文中，我们还创建一个新的 CuckooManager 对象 ➌ 和一个新的 FileTask 对象 ➍。我们还将任务的 Filepath 属性 ➎ 设置为文件系统上包含我们要分析的可执行文件的路径。为了测试，你可以使用 Metasploit 的 msfvenom 生成有效载荷（如我们在第四章中所做的那样），或者使用我们在第四章中编写的一些有效载荷。将 FileTask 设置为扫描的文件后，我们将任务传递给管理器的 CreateTask() 方法 ➏，并存储返回的 ID 以供后续使用。

一旦任务创建完成，我们调用 GetTaskDetails() ➐ 并传入由 CreateTask() 返回的任务 ID。当我们调用 GetTaskDetails() 时，该方法会返回一个状态。在这种情况下，我们只对两种状态感兴趣：待处理和失败。只要 GetTaskDetails() 返回待处理状态，我们就会向用户打印一个友好的消息，告知任务尚未完成，并让应用程序休眠 30 秒后再调用 GetTaskDetails() 获取任务状态。一旦状态不再是待处理状态，我们会检查状态是否为失败 ➑，以防在分析过程中出现问题。如果任务的状态是失败，我们会打印 Cuckoo Sandbox 返回的错误信息。

然而，如果状态不是失败，我们可以假设任务已成功完成分析，并且可以从 Cuckoo Sandbox 创建一个新报告，包含分析结果。我们调用 GetTaskReport() 方法 ➒，传入任务 ID 作为唯一参数，然后使用 WriteLine() ➓ 将报告打印到控制台屏幕上。

测试应用程序

通过自动化操作，我们终于可以驱动 Cuckoo Sandbox 实例运行并分析一个可能恶意的 Windows 可执行文件，然后检索运行任务的报告，如列表 8-20 所示。记得以管理员身份运行实例。

> $ ./ch8_automating_cuckoo.exe
> 
> 等待 30 秒...待处理
> 
> {
> 
> "info": {
> 
> "category": "file",
> 
> "score": 0.0,
> 
> "package": "",
> 
> "started": "2016-05-19 15:56:44",
> 
> "route": "none",
> 
> "custom": "",
> 
> "machine": {
> 
> "status": "stopped",
> 
> "name": "➊cuckoo1",
> 
> "label": "cuckoo1",
> 
> "manager": "VirtualBox",
> 
> "started_on": "2016-05-19 15:56:44",
> 
> "shutdown_on": "2016-05-19 15:57:09"
> 
> },
> 
> "ended": "2016-05-19 15:57:09",
> 
> "version": "2.0-rc2",
> 
> "platform": "",
> 
> "owner": "",
> 
> "options": "",
> 
> "id": 13,
> 
> "duration": 25
> 
> },
> 
> "signatures": [],
> 
> "target": {
> 
> "category": "file",
> 
> "file": {
> 
> "yara": [],
> 
> "sha1": "f145181e095285feeb6897c9a6bd2e5f6585f294",
> 
> "name": "bypassuac-x64.exe",
> 
> "type": "PE32+ 可执行文件（控制台） x86-64，适用于 MS Windows",
> 
> "sha256": "➋2a694038d64bc9cfcd8caf6af35b6bfb29d2cb0c95baaeffb2a11cd6e60a73d1",
> 
> "urls": [],
> 
> "crc32": "26FB5E54",
> 
> "path": "/home/bperry/tmp/cuckoo/storage/binaries/2a694038d2cb0c95baaeffb2a11cd6e60a73d1",
> 
> "ssdeep": null,
> 
> "size": 501248,
> 
> "sha512":
> 
> "4b09f243a8fcd71ec5bf146002519304fdbaf99f1276da25d8eb637ecbc9cebbc49b580c51e36c96c8548a41c38cc76
> 
> 595ad1776eb9bd0b96cac17ca109d4d88",
> 
> "md5": "46a695c9a3b93390c11c1c072cf9ef7d"
> 
> }
> 
> },
> 
> --snip--

列表 8-20：Cuckoo Sandbox 分析 JSON 报告

Cuckoo Sandbox 的分析报告非常庞大。它包含了关于在 Windows 系统上运行你的可执行文件时发生的非常详细的信息。该列表展示了有关分析的基本元数据，如执行分析的机器 ➊ 和可执行文件的常见哈希值 ➋。一旦报告输出完成，我们就可以开始了解恶意软件在被感染系统上所做的事情，并制定修复和清理计划。

请注意，这里仅包含报告的部分内容。未显示的部分包括所做的巨大数量的 Windows API 和系统调用，操作过的文件以及其他极为详细的系统信息，这些信息可以帮助你更快地确定恶意软件样本在客户端机器上可能执行了什么。更多信息可以在 Cuckoo Sandbox 官方文档网站找到，了解具体报告内容以及如何使用：[`docs.cuckoosandbox.org/en/latest/usage/results/`](http://docs.cuckoosandbox.org/en/latest/usage/results/).

作为一种练习，你可以将完整的报告保存到文件中，而不是打印到控制台屏幕上，因为输出文件可能更适合未来的恶意软件分析！

结论

Cuckoo Sandbox 是一个强大的恶意软件分析框架，借助 API 功能，它可以轻松集成到工作流程、电子邮件服务器等基础设施中，甚至是事件响应操作手册中。通过在沙箱环境中运行文件和任意网站，安全专业人员可以轻松快速地确定攻击者是否通过有效载荷或驱动器攻击渗透了网络。

在本章中，我们能够通过核心 C# 类和库编程驱动 Cuckoo Sandbox 的这一功能。我们创建了一些类与 API 进行通信，然后创建了任务，并在任务完成时报告它们。然而，我们只实现了对基于文件的恶意软件分析的支持。我们构建的类是可扩展的，因此可以添加和支持新类型的任务，例如提交一个 URL 以在 Web 浏览器中打开的任务。

有了这样一个高质量且实用的框架，所有人都可以免费使用，任何人都可以将此功能添加到其组织的安全关键基础设施中，从而轻松减少发现和修复家庭或企业网络潜在安全漏洞所需的时间。
