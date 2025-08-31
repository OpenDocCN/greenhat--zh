## 第三部分

构建你自己的模块

到目前为止，你应该已经对 PowerShell 的特点有了清晰的了解。我们已经讨论了语言的语法，以及一些你在日常自动化工作中可能会使用的特定模块。但直到前一章为止，我们所做的事情都是零散的：这里一点语法，那里一点语法，没有什么重要的内容。在第十四章中，使用服务器库存脚本，你第一次尝试了进行一个较长时间的 PowerShell 项目。在第三部分中，我们将做得更大：你将构建自己的 PowerShell 模块。

### PowerLab

*PowerLab* 是一个单独的 PowerShell 模块，包含了你从零开始配置 Windows 服务器所需的功能。你将一步一步构建 PowerLab；如果你想看到最终结果，可以在这个 GitHub 仓库中找到：[`github.com/adbertram/PowerLab`](https://github.com/adbertram/PowerLab)。

从零开始配置 Windows 服务器的过程大致如下：

+   创建虚拟机。

+   安装 Windows 操作系统。

+   安装服务器服务（Active Directory、SQL Server 或 IIS）。

这意味着你需要 PowerLab 模块来完成五个任务：

+   创建 Hyper-V 虚拟机

+   安装 Windows 服务器

+   创建 Active Directory 林

+   配置 SQL 服务器

+   配置 IIS Web 服务器

为了完成这些任务，你将使用三个主要命令：

+   `New-PowerLabActiveDirectoryForest`

+   `New-PowerLabSqlServer`

+   `New-PowerLabWebServer`

当然，你会使用超过三个命令。你将通过多个辅助命令构建每个命令，这些辅助命令将处理后台功能，包括创建虚拟机和安装操作系统。但我们将在接下来的章节中详细介绍这一切。

### 先决条件

构建 PowerLab 你需要一些东西：

+   一台 Windows 10 专业版客户端计算机，位于工作组中。已加入域的 Windows 10 计算机可能也能使用，但未经过测试。

+   一台运行 Windows Server 2012 R2（至少）并与客户端处于同一网络中的 Hyper-V 主机—尽管主机也可以加入域，但此场景未经过测试。

+   位于 Hyper-V 主机上的 Windows Server 2016 的 ISO 文件。未测试 Windows Server 2019。你可以从 [*https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016?filetype=ISO*](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016?filetype=ISO) 下载 Windows Server 的评估版本。

+   在客户端计算机上启用远程服务器管理工具（RSAT）（从 [*https://www.microsoft.com/en-us/download/details.aspx?id=45520*](https://www.microsoft.com/en-us/download/details.aspx?id=45520) 下载）。

+   在你的客户端计算机上安装最新版本的 Pester PowerShell 模块。

你还需要以本地管理员组成员身份登录客户端计算机，并将 PowerShell 执行策略设置为 unrestricted。（你可以运行 `Set-ExecutionPolicy Unrestricted` 来更改执行策略，但我建议在实验室设置完成后将其更改回 `AllSigned` 或 `RemoteSigned`。）

### 设置 PowerLab

当像 PowerLab 这样的工具提供给消费者时，你希望让设置过程尽可能无痛。一种方法是提供一个脚本，处理模块的安装和配置，并且需要尽量少的用户输入。

我已经编写好了 PowerLab 的安装脚本。它可以在 PowerLab 的 GitHub 仓库中找到：[*https://raw.githubusercontent.com/adbertram/PowerLab/master/Install-PowerLab.ps1*](https://raw.githubusercontent.com/adbertram/PowerLab/master/Install-PowerLab.ps1)。该链接将提供脚本的原始源代码。你可以将其复制并粘贴到一个新的文本文件中，并保存为 *Install-PowerLab.ps1*，不过这是一本 PowerShell 书籍，因此我们可以尝试运行以下命令：

```
PS> Invoke-WebRequest -Uri 'http://bit.ly/powerlabinstaller' -OutFile 'C:\Install-PowerLab.ps1'
```

请注意：当你运行脚本时，你需要回答一些问题。你需要提供 Hyper-V 主机的主机名、Hyper-V 主机的 IP 地址、Hyper-V 主机的本地管理员用户名和密码，以及每个要安装的操作系统的产品密钥（如果没有使用 Windows Server 评估版）。

一旦你拥有所有必要的信息，就可以使用以下命令运行安装脚本：

```
PS> C:\Install-PowerLab.ps1

Name of your HYPERV host: HYPERVSRV
IP address of your HYPERV host: 192.168.0.200
Enabling PS remoting on local computer...
Adding server to trusted computers...
PS remoting is already enabled on [HYPERVSRV]
Setting firewall rules on Hyper-V host...
Adding the ANONYMOUS LOGON user to the local machine and host server
Distributed COM Users group for Hyper-V manager
Enabling applicable firewall rules on local machine...
Adding saved credential on local computer for Hyper-V host...
Ensure all values in the PowerLab configuration file are valid and close the
ISE when complete.
Enabling the Microsoft-Hyper-V-Tools-All features...
Lab setup is now complete.
```

如果你想检查这个脚本的具体内容，可以通过书籍的资源下载并查看它。不过，请知道这个脚本是为了让我们都能到达相同的基础架构，而不一定是为了展示脚本的具体操作；此时它可能超出了你的理解范围。这个脚本的目的是帮助你跟上我的进度。

### 演示代码

你在接下来的章节中编写的所有代码都可以在 *[`github.com/adbertram/PowerShellForSysadmins/tree/master/Part%20III`](https://github.com/adbertram/PowerShellForSysadmins/tree/master/Part%20III)* 找到。除了所有的 PowerLab 代码，你还会找到必要的数据文件以及 Pester 脚本，用于测试模块并验证你的环境是否满足所有预期的前提条件。在开始每一章之前，我*强烈*建议你使用 `Invoke-Pester` 命令来运行每章文件中找到的 *Prerequisites.Tests.ps1* Pester 脚本。这样做可以避免后续出现许多令人头痛的 bug。

### 摘要

你应该已经准备好开始构建 PowerLab。接下来的章节将涉及很多内容，并且会涉及 PowerShell 的多个领域，所以如果你看到不认识的东西不要感到惊讶。网络上有很多资源可以帮助你解决复杂的语法问题，如果你不理解某些内容，也可以随时在 Twitter 上联系我，用户名是 @adbertram，或者向互联网中的其他人求助。

既然如此，让我们开始吧！
