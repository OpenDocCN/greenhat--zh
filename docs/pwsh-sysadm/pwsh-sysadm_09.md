## 第八章：远程运行脚本

![图片](img/common.jpg)

如果你是一个小型组织中的唯一 IT 人员，可能有多台服务器需要管理。如果你有一个脚本需要运行，你可以登录到每台服务器，打开 PowerShell 控制台并在那里运行你的脚本。但如果你运行一个脚本，执行每台服务器上的特定任务，你可以节省很多时间。在本章中，你将学习如何使用 PowerShell 远程管理来远程运行命令。

*PowerShell 远程管理*是一个功能，允许用户在一个或多个计算机上的会话中远程执行命令。*会话*，或更具体地说，`PSSession`，是 PowerShell 远程管理的术语，指的是在远程计算机上运行 PowerShell 的环境，从中可以执行命令。尽管执行方式不同，但微软的 Sysinternals 工具`psexec`与此概念相同：你编写在本地计算机上工作的代码，将其发送到远程计算机，并像坐在它前面一样执行该代码。

本章的大部分内容将集中在会话上——它们是什么，如何使用它们，以及完成后如何处理它们——但首先，你需要了解一些关于脚本块的内容。

**注意**

*微软在 PowerShell v2 中引入了 PowerShell 远程管理，它是建立在* Windows 远程管理(WinRM) *服务之上的。因此，偶尔你可能会看到 WinRM 一词，用来指代 PowerShell 远程管理。*

### 使用脚本块

PowerShell 远程管理广泛使用*脚本块*，它们像函数一样，是封装成一个可执行单元的代码。但它们与函数有几个关键区别：它们是匿名的——或者说没有名称——并且可以赋值给变量。

为了检验这种差异，我们来看一个例子。我们定义一个名为`New-Thing`的函数，它调用`Write-Host`在控制台中显示一些文本（见列表 8-1）。

```
function New-Thing {
    param()
    Write-Host "Hi! I am in New-Thing"
}

New-Thing
```

*列表 8-1：定义`New-Thing`函数，在控制台窗口显示文本*

如果你运行这个脚本，你应该看到它将文本`"Hi! I am in New-Thing!"`返回到控制台。但请注意，为了得到这个结果，你必须调用`New-Thing`函数才能执行。

你可以通过首先将脚本块赋值给一个变量来复制`New-Thing`函数调用的结果，如列表 8-2 所示。

```
PS> $newThing = { Write-Host "Hi! I am in a scriptblock!" }
```

*列表 8-2：创建脚本块并将其赋值给名为`$newThing`的变量*

要构建一个脚本块，将你想执行的代码放在花括号之间。你将我们的脚本块存储在变量`$newThing`中，你可能认为要执行这个脚本块，只需调用该变量，如列表 8-3 所示。

```
PS> $newThing = { Write-Host "Hi! I am in a scriptblock!" }
PS> $newThing
 Write-Host "Hi! I am in a scriptblock!"
```

*列表 8-3：创建并执行脚本块*

但正如你所看到的，PowerShell 会字面地读取`$newThing`的内容。它没有意识到`Write-Host`是一个应该执行的命令，而是显示了脚本块的值。

要告诉 PowerShell 运行内部的代码，你需要使用&符号（`&`）后跟变量名。示例 8-4 展示了这种语法。

```
PS> & $newThing
Hi! I am in a scriptblock!
```

*示例 8-4：执行脚本块*

&符号告诉 PowerShell，花括号中的内容是它应该执行的代码。&符号是执行代码块的一种方式；然而，它不允许像命令那样进行定制，而你在使用 PowerShell 远程操作时，通常需要这种定制功能来操作远程计算机。下一节将介绍另一种执行脚本块的方法。

#### 使用`Invoke-Command`在远程系统上执行代码

在使用 PowerShell 远程操作时，你将使用两个主要命令：`Invoke-Command`和`New-PSSession`。在本节中，你将学习`Invoke-Command`；下一节将介绍`New-PSSession`命令。

`Invoke-Command`可能是你在使用 PowerShell 远程操作时最常用的命令。有两种主要的使用方式。第一种是当你运行我所说的*临时命令*——你希望执行的小型、一次性的表达式。第二种是使用交互式会话。本章将介绍这两种方式。

一个临时命令的例子是，当你运行`Start-Service`命令以启动远程计算机上的服务时。当你使用`Invoke-Command`执行临时命令时，PowerShell 会在后台创建一个会话，并在命令执行完成后立即拆除该会话。这限制了你只能用`Invoke-Command`做的事情，这就是为什么在下一节你将看到如何创建自己的会话。

但现在，让我们看看`Invoke-Command`如何与临时命令一起工作。打开你的 PowerShell 控制台，键入`Invoke-Command`并按 ENTER，正如在示例 8-5 中所示。

```
PS> Invoke-Command

cmdlet Invoke-Command at command pipeline position 1
Supply values for the following parameters:
ScriptBlock:
```

*示例 8-5：无参数运行`Invoke-Command`*

你的控制台应该立即要求你提供一个脚本块。你将提供`hostname`命令，它将返回执行命令的计算机的主机名。

要将`hostname`脚本块传递给`Invoke-Command`，你需要使用必需的参数`ComputerName`，它告诉`Invoke-Command`在哪台远程计算机上运行此命令，正如你在示例 8-6 中看到的那样。（注意，要使其正常工作，我的计算机和远程计算机`WEBSRV1`必须是同一 Active Directory（AD）域的一部分，并且我的计算机需要在`WEBSRV1`上拥有管理员权限。）

```
PS> Invoke-Command -ScriptBlock { hostname } -ComputerName WEBSRV1
WEBSRV1
```

*示例 8-6：运行一个简单的`Invoke-Command`示例*

请注意，`hostname`的输出现在是远程计算机的名称——在我的系统中，远程计算机名为`WEBSRV1`。你现在已经成功执行了你的第一个远程命令！

**注意**

*如果您在运行 Windows Server 2012 R2 之前版本的操作系统的远程计算机上尝试此操作，可能无法按预期工作。如果是这种情况，您首先需要启用 PowerShell 远程处理。从 Server 2012 R2 开始，PowerShell 远程处理默认启用，WinRM 服务正在运行，并且所有必要的防火墙端口已打开并设置了访问权限。但如果您运行的是较早版本的 Windows，则必须手动执行此操作，因此在尝试对旧版本的服务器运行 Invoke-Command 之前，首先需要在远程计算机上以提升权限的控制台会话运行 Enable-PSRemoting。您还可以使用 Test-WSMan 命令确认 PowerShell 远程处理是否已配置并可用。*

#### 在远程计算机上运行本地脚本

在上一节中，您在远程计算机上执行了脚本块。您还可以使用 `Invoke-Command` 执行整个脚本。与使用 `Scriptblock` 参数不同，您可以使用 `FilePath` 参数并提供本地计算机上的脚本路径。使用 `FilePath` 参数时，`Invoke-Command` 会在本地读取脚本内容，然后在远程计算机上执行这些代码。与普遍认知相反，脚本本身并不会在远程计算机上执行。

举个例子，假设您在本地计算机的 *C:\* 根目录下有一个名为 *GetHostName.ps1* 的脚本。该脚本包含一行：`hostname`。您希望在远程计算机上运行这个脚本以返回计算机的主机名。请注意，虽然我们保持脚本极其简单，但 `Invoke-Command` 并不关心脚本中的内容。它会高兴地执行其中的任何内容。

要运行脚本，您将脚本文件传递给 `Invoke-Command` 的 `FilePath` 参数，如 清单 8-7 所示。

```
PS> Invoke-Command -ComputerName WEBSRV1 -FilePath C:\GetHostName.ps1
WEBSRV1
```

*清单 8-7：在远程计算机上运行本地脚本*

`Invoke-Command` 会在 `WEBSRV1` 计算机上运行 *GetHostName.ps1* 中的代码，并将输出返回到您的本地会话。

#### 在远程使用本地变量

尽管 PowerShell 远程处理解决了许多问题，但在使用本地变量时仍需小心。假设您在远程计算机上有一个文件路径 *C:\File.txt*。由于这个文件路径可能会在某个时候发生变化，您可能决定将该路径分配为一个变量，例如 `$serverFilePath`：

```
PS> $serverFilePath = 'C:\File.txt'
```

现在，您可能需要在远程脚本块中引用 *C:\File.txt* 路径。在 清单 8-8 中，您可以看到当您尝试直接引用该变量时会发生什么情况。

```
PS> Invoke-Command -ComputerName WEBSRV1 -ScriptBlock { Write-Host "The value
of foo is $serverFilePath" }
The value of foo is
```

*清单 8-8：本地变量在远程会话中不起作用。*

请注意，`$serverFilePath` 变量没有值，因为在远程计算机上执行的脚本块中，该变量并不存在！当你在脚本或控制台中定义一个变量时，该变量会存储在一个特定的 *运行空间* 中，这是 PowerShell 用来存储会话信息的容器。如果你尝试同时打开两个 PowerShell 控制台并（未能）在另一个控制台中使用其中的变量，你可能已经遇到过运行空间的概念。

默认情况下，变量、函数和其他构造不能跨多个运行空间传播。然而，你可以使用几种方法在不同的运行空间中使用变量、函数等。有两种主要方法可以将变量传递到远程计算机。

##### 使用 `ArgumentList` 参数传递变量

要将变量的值传递到远程脚本块中，你可以在 `Invoke-Command` 上使用 `ArgumentList` 参数。这个参数允许你将本地值的数组传递到脚本块中，称为 `$args`，你可以在脚本块的代码中使用它。为了演示这个过程，在 列表 8-9 中，你将传递包含文件路径 *C:\File.txt* 的 `$serverFilePath` 变量到远程脚本块，并通过 `$args` 数组进行引用。

```
PS> Invoke-Command -ComputerName WEBSRV1 -ScriptBlock { Write-Host "The value
of foo is $($args[0])" } -ArgumentList $serverFilePath
The value of foo is C:\File.txt
```

*列表 8-9：使用 `$args` 数组将本地变量传递到远程会话*

正如你所看到的，变量的值 *C:\File.txt* 现在已在脚本块中。这是因为你将 `$serverFilePath` 传递到 `ArgumentList` 中，并且将脚本块内的 `$serverFilePath` 引用替换为 `$args[0]`。如果你想传递多个变量到脚本块中，你可以在 `ArgumentList` 参数值中添加另一个值，并在需要引用新变量的地方将 `$args` 引用加 1。

##### 使用 `$Using` 语句传递变量值

将本地变量的值传递给远程脚本块的另一种方法是使用 `$using` 语句。通过在任何本地变量名前加上 `$using`，你可以避免使用 `ArgumentList` 参数。在 PowerShell 将脚本块发送到远程计算机之前，它会查找 `$using` 语句，并展开脚本块中的所有本地变量。

在 列表 8-10 中，你将重写 列表 8-9，使用 `$using:serverFilePath` 来代替 `ArgumentList`。

```
PS> Invoke-Command -ComputerName WEBSRV1 -ScriptBlock { Write-Host "The value
of foo is $using:serverFilePath" }
The value of foo is C:\File.txt
```

*列表 8-10：使用 `$using` 来引用远程会话中的本地变量*

正如你所看到的，列表 8-9 和 8-10 的结果是相同的。

`$using`语句需要的工作更少，也更直观，但将来当你开始编写 Pester 测试脚本时，你会发现可能需要回退到使用`ArgumentList`参数：当使用`$using`选项时，Pester 无法评估`$using`变量中的值。而当使用`ArgumentList`参数时，传递给远程会话的变量是在本地定义的，Pester 可以解释和理解这些变量。如果现在这不太清楚，等你读到第九章时你就会明白了。现在，`$using`语句已经非常优秀了！

现在你对`Invoke-Command` cmdlet 有了基本的了解，让我们学习更多关于会话的内容。

### 使用会话

如前所述，PowerShell 远程操作使用了一个叫做*会话*的概念。当你在远程创建会话时，PowerShell 会在远程计算机上打开一个*本地会话*，你可以使用这个会话在那里执行命令。你不需要了解太多会话的技术细节。你需要知道的是，你可以创建会话、连接到会话、断开会话，并且会话将保持你离开时的状态。会话在你删除它之前不会结束。

在上一节中，当你运行`Invoke-Command`时，它会启动一个新的会话，运行代码，并且一气呵成地结束会话。在本节中，你将看到如何创建我所称的*完整会话*，即你可以直接向其中输入命令的会话。使用`Invoke-Command`执行一次性的临时命令效果很好，但当你需要运行很多不能全部放入单个脚本块的命令时，它就不那么高效了。例如，如果你正在编写一个大型脚本，这个脚本需要本地执行一些工作、从另一个源获取信息、在远程会话中使用这些信息、从远程会话获取信息并返回本地计算机，你就必须创建一个脚本，反复运行`Invoke-Command`。更麻烦的是，如果你需要在远程会话中设置一个变量并在之后再次使用它，使用目前的`Invoke-Command`方法是无法实现的——你需要一个在你离开后仍然保持活动的会话。

#### 创建新会话

要在远程计算机上创建一个半永久的会话，进行 PowerShell 远程操作，你必须显式地通过使用`New-PSSession`命令来创建一个完整会话，这会在远程计算机上创建一个会话，并在本地计算机上创建该会话的引用。

要创建一个新的 `PSSession`，请使用带有 `ComputerName` 参数的 `New-PSSession`，就像在 清单 8-11 中所示。在此示例中，我运行此命令的计算机与 `WEBSRV1` 处于同一的 Active Directory 域中，并且我以域用户的身份登录 `WEBSRV1`，具有管理员权限。要通过使用 `ComputerName` 参数进行连接（就像我在 清单 8-11 中所做的那样），用户必须是本地管理员或者至少是远程管理用户组中的成员。如果您不在 AD 域中，可以在 `New-PSSession` 上使用 `Credential` 参数，传递一个包含用于身份验证到远程计算机的备用凭据的 `PSCredential` 对象。

```
PS> New-PSSession -ComputerName WEBSRV1

 Id Name        ComputerName   ComputerType    State    ConfigurationName      Availability
 -- ----        ------------   ------------    -----    -----------------      ------------
  3 WinRM3      WEBSRV1        RemoteMachine   Opened   Microsoft.PowerShell   Available
```

*清单 8-11：创建一个新的 `PSSession`*

如您所见，`New-PSSession` 返回一个会话。一旦建立了会话，您可以通过 `Invoke-Command` 跳入和跳出会话；与使用临时命令时不同，您将必须使用 `Session` 参数。

您需要使用 `Session` 参数并提供一个会话对象。您可以使用 `Get-PSSession` 命令查看所有当前会话。在 清单 8-12 中，您将会将 `Get-PSSession` 的输出存储在一个变量中。

```
PS> $session = Get-PSSession
PS> $session

 Id    Name     ComputerName   ComputerType    State    ConfigurationName      Availability
 --    ----     ------------   ------------    -----    -----------------      ------------
  6    WinRM6   WEBSRV1        RemoteMachine   Opened   Microsoft.PowerShell   Available
```

*清单 8-12：查找在本地计算机上创建的会话*

因为您只运行了一次 `New-PSSession`，所以在 清单 8-12 中只创建了一个 `PSSession`。如果您有多个会话，可以通过使用 `Get-PSSession` 命令的 `Id` 参数来选择 `Invoke-Command` 要使用的会话。

#### 在会话中调用命令

现在您有了一个存储在变量中的会话，可以将该变量传递给 `Invoke-Command` 并在会话中运行一些代码，就像在 清单 8-13 中所示。

```
PS> Invoke-Command -Session $session -ScriptBlock { hostname }
WEBSRV1
```

*清单 8-13：使用现有会话在远程计算机上调用命令*

您应该注意到，此命令的运行速度比您传递命令时要快得多。这是因为 `Invoke-Command` 不需要创建和拆除一个新的会话。当您创建完整会话时，不仅速度更快，而且还可以访问更多功能。例如，正如您在 清单 8-14 中所见，您可以在远程会话中设置变量，并且返回到会话时不会丢失这些变量。

```
PS> Invoke-Command -Session $session -ScriptBlock { $foo = 'Please be here next time' }
PS> Invoke-Command -Session $session -ScriptBlock { $foo }
Please be here next time
```

*清单 8-14：变量值在后续会话连接中保持不变。*

只要会话保持打开状态，您可以在远程会话中执行任何您需要的操作，会话的状态将保持不变。但是，这仅适用于当前的本地会话。如果启动另一个 PowerShell 进程，您不能继续之前的操作。远程会话仍然处于活动状态，但是本地计算机对该远程会话的引用将会丢失。在这种情况下，`PSSession` 将进入断开连接状态（您将在后面的部分看到）。

#### 打开交互式会话

Listing 8-14 使用 `Invoke-Command` 向远程计算机发送命令并接收响应。像这样运行远程命令就像运行一个没有监控的脚本。它不是交互式的，就像你在 PowerShell 控制台中输入按键一样。如果你想为远程计算机上运行的会话打开一个交互式控制台——例如进行故障排除——你可以使用 `Enter-PSSession` 命令。

`Enter-PSSession` 命令允许用户以交互方式操作会话。它可以创建自己的会话，也可以依赖于通过 `New-PSSession` 创建的现有会话。如果没有指定要进入的会话，`Enter-PSSession` 将创建一个新的会话并等待进一步输入，如 Listing 8-15 所示。

```
PS> Enter-PSSession -ComputerName WEBSRV1
[WEBSRV1]: PS C:\Users\Adam\Documents>
```

*Listing 8-15: 进入交互式会话*

请注意，你的 PowerShell 提示符已经变为 `[WEBSRV1]: PS`。这个提示符表示你不再在本地运行命令，而是在那个远程会话中。此时，你可以像在远程计算机的控制台上一样运行任何命令。像这样交互式地操作会话是避免使用 *远程桌面协议*（*RDP*）应用程序启动交互式 GUI 来执行任务（例如远程计算机的故障排除）的好方法。

#### 断开和重新连接会话

如果你关闭了 PowerShell 控制台，然后重新打开它，再尝试在之前工作的会话中使用 `Invoke-Command`，你将收到一条错误消息，如 Listing 8-16 所示。

```
PS> $session = Get-PSSession -ComputerName websrv1
PS> Invoke-Command -Session $session -ScriptBlock { $foo }
Invoke-Command : Because the session state for session WinRM6, a617c702-ed92
-4de6-8800-40bbd4e1b20c, websrv1 is not equal to Open, you cannot run a
command in the session. The session state is Disconnected.
At line:1 char:1
+ Invoke-Command -Session $session -ScriptBlock { $foo }
--snip--
```

*Listing 8-16: 尝试在断开的会话中运行命令*

PowerShell 能够在远程计算机上找到 `PSSession`，但在本地计算机上找不到该引用，这表明会话已断开。如果没有正确断开本地会话对远程 `PSSession` 的引用，就会发生这种情况。

你可以使用 `Disconnect-PSSession` 命令断开现有会话。你可以通过 `Get-PSSession` 检索之前创建的会话，然后将这些会话传递给 `Disconnect-PSSession` 命令进行清理（参见 Listing 8-17）。或者，您还可以使用 `Disconnect-PSSession` 的 `Session` 参数一次断开一个会话。

```
PS> Get-PSSession | Disconnect-PSSession

Id Name          ComputerName   ComputerType    State          ConfigurationName    Availability
-- ----          ------------   ------------    -----          -----------------    ------------
 4 WinRM4        WEBSRV1        RemoteMachine   Disconnected   Microsoft.PowerShell None
```

*Listing 8-17: 断开 `PSSession` 连接*

要正确地断开会话，你可以通过显式调用 `Disconnect-PSSession -Session` 会话名称，或通过 `Get-PSSession` 将现有会话传递给命令来将远程会话名称传递给 `Session` 参数，如 Listing 8-17 所示。

如果你想在稍后重新连接会话，在使用`Disconnect-PSSession`断开连接后，关闭你的 PowerShell 控制台，然后使用`Connect-PSSession`命令，如示例 8-18 所示。请注意，你只能看到并连接到已断开的会话，这些会话必须是你账户之前创建的。你将无法看到其他用户创建的会话。

```
PS> Connect-PSSession -ComputerName websrv1
[WEBSRV1]: PS>
```

*示例 8-18：重新连接到`PSSession`*

现在你应该能够像从未关闭过控制台一样，在远程计算机上运行代码。

如果你仍然收到错误信息，可能是 PowerShell 版本不匹配。断开会话仅在本地计算机和远程服务器的 PowerShell 版本相同的情况下有效。例如，如果本地计算机上运行 PowerShell 5.1，而你连接的远程服务器正在运行一个不支持断开会话的版本（例如 PowerShell v2 或更早版本），则断开会话将无法正常工作。始终确保本地计算机和远程服务器使用相同版本的 PowerShell。

要检查本地计算机的 PowerShell 版本是否与远程计算机的版本匹配，请检查`$PSVersionTable`变量的值，该变量包含版本信息（见示例 8-19）。

```
PS> $PSVersionTable

Name                           Value
----                           -----
PSVersion                      5.1.15063.674
PSEdition                      Desktop
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}
BuildVersion                   10.0.15063.674
CLRVersion                     4.0.30319.42000
WSManStackVersion              3.0
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1
```

*示例 8-19：检查本地计算机上的 PowerShell 版本*

要检查远程计算机上的版本，可以在该计算机上运行`Invoke-Command`，并传递`$PSVersionTable`变量，如示例 8-20 所示。

```
PS> Invoke-Command -ComputerName WEBSRV1 -ScriptBlock { $PSVersionTable }

Name                           Value
----                           -----
PSRemotingProtocolVersion      2.2
BuildVersion                   6.3.9600.16394
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0}
PSVersion                      4.0
CLRVersion                     4.0.30319.34014
WSManStackVersion              3.0
SerializationVersion           1.1.0.1
```

*示例 8-20：检查远程计算机上的 PowerShell 版本*

我建议，在断开连接之前，你检查一下版本是否匹配；这样，你就能避免在远程系统上丢失宝贵的工作。

#### 使用 Remove-PSSession 删除会话

每当`New-PSSession`命令创建一个新会话时，该会话会同时存在于远程服务器和本地计算机上。你也可以同时在多个服务器上打开多个会话，如果其中一些会话不再使用，你可能最终需要清理它们。你可以使用`Remove-PSSession`命令来执行此操作，该命令会访问远程计算机，关闭该会话，并在存在的情况下，移除本地的`PSSession`引用。示例 8-21 就是一个例子：

```
PS> Get-PSSession | Remove-PSSession
PS> Get-PSSession
```

*示例 8-21：删除`PSSession`*

这里，你再次运行`Get-PSSession`，但没有返回任何结果。这意味着本地计算机上没有会话。

### 理解 PowerShell 远程身份验证

到目前为止，我一直没有涉及身份验证的问题。默认情况下，如果本地计算机和远程计算机都在同一个域中，并且都启用了 PowerShell 远程功能，你无需显式身份验证。但是，如果它们不在同一域中，你需要以某种方式进行身份验证。

使用 PowerShell 远程连接到远程计算机的两种最常见身份验证方式是通过 Kerberos 或 CredSSP。如果你处于一个 Active Directory 域中，你很可能已经在使用 Kerberos 票证系统，无论你是否意识到这一点。Active Directory 和一些 Linux 系统使用 Kerberos *领域*，它们向客户端颁发票证。这些票证随后会被提交给资源并与域控制器（在 Active Directory 中）进行比对。

另一方面，CredSSP 不需要 Active Directory。CredSSP 从 Windows Vista 开始就已被引入，它使用客户端凭据服务提供程序（CSP）来使应用程序能够将用户凭据委托给远程计算机。CredSSP 不需要像域控制器这样的外部系统来进行两台系统的身份验证。

在 Active Directory 环境中，PowerShell 远程使用 Kerberos 网络身份验证协议向 Active Directory 发起调用，所有身份验证操作都在后台完成。PowerShell 使用你本地登录的账户作为远程计算机的用户身份进行身份验证——就像许多其他服务一样。这就是单点登录的优势所在。

但有时如果你不在 Active Directory 环境中，你就不得不稍微改变身份验证类型；例如，当你需要通过远程计算机上的本地凭据连接到远程计算机，无论是通过互联网还是本地网络时。PowerShell 支持多种 PowerShell 远程身份验证方法，但最常见的——除了使用 Kerberos 之外——是 CredSSP，它允许本地计算机将用户凭据委托给远程计算机。这个概念类似于 Kerberos，但不需要 Active Directory。

在 Active Directory 环境中工作时，通常不需要使用不同的身份验证类型，但有时会遇到这种情况，因此最好做好准备。在本节中，你将学习一个常见的身份验证问题以及如何绕过它。

#### 双跳问题

*双跳问题* 自从 Microsoft 添加了 PowerShell 远程功能以来一直存在。当你在远程会话中运行代码，然后尝试从该远程会话访问远程资源时，就会出现这个问题。例如，如果你的网络中有一个名为 DC 的域控制器，并且你想通过 `C$` 管理共享查看 *C:\* 根目录下的文件，你可以从本地计算机远程浏览该共享，毫无问题（参见 示例 8-22）。

```
PS> Get-ChildItem -Path '\\dc\c$'

    Directory: \\dc\c$

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        10/1/2019  12:05 PM                FileShare
d-----       11/24/2019   2:28 PM                inetpub
d-----       11/22/2019   6:37 PM                InstallWindowsFeature
d-----        4/16/2019   1:10 PM                Iperf
```

*示例 8-22：列举 UNC 共享上的文件*

这个问题出现在你创建了一个 `PSSession` 并尝试重新运行相同的命令时，如 示例 8-23 中所示。

```
PS> Enter-PSSession -ComputerName WEBSRV1
[WEBSRV1]: PS> Get-ChildItem -Path '\\dc\c$'
ls : Access is denied
--snip--
[WEBSRV1]: PS>
```

*示例 8-23：尝试在会话中访问网络资源*

在这种情况下，即使你知道你的用户账户有访问权限，PowerShell 仍然告诉你访问被拒绝。这是因为，当你使用默认的 Kerberos 身份验证时，PowerShell 远程操作并不会将凭据传递给其他网络资源。换句话说，它没有完成两个跳跃。出于安全原因，PowerShell 遵循 Windows 限制，拒绝委派这些凭据，结果返回“访问被拒绝”的消息。

#### 使用 CredSSP 的双跳问题

在本节中，你将学习如何解决双跳问题。我使用“解决”而不是“修复”有原因。微软已警告，使用 CredSSP 是一个安全问题，因为传递给第一个计算机的凭据会自动用于从该计算机进行的所有连接。这意味着，如果原始计算机被攻破，那么可以利用该凭据从该计算机连接到网络上的其他计算机。尽管如此，除了使用一些复杂的变通方法，如基于资源的 Kerberos 受限委派，许多用户还是选择使用 CredSSP 方法，因为它容易使用。

在实现 CredSSP 之前，你必须在客户端和服务器上都启用它，可以通过在提升的 PowerShell 会话中使用 `Enable-WsManCredSSP` 命令来实现。此命令有一个 `Role` 参数，允许你定义是启用客户端还是服务器端的 CredSSP。首先，在客户端启用 CredSSP，如 清单 8-24 所示。

**注意**

*要使 CredSSP 生效，可能需要放宽本地策略。如果在尝试启用 CredSSP 时收到权限错误，请确保通过运行 gpedit.msc 并在计算机配置 ▶ 管理模板 ▶ 系统 ▶ 凭据委派下启用“允许仅使用 NTLM 服务器身份验证的保存凭据委派”设置。在该策略中，点击 **显示** 按钮并输入 **WSMAN/*** 以允许从任何端点委派凭据。*

```
PS> Enable-WSManCredSSP ❶-Role ❷Client ❸-DelegateComputer WEBSRV1 -Force

CredSSP Authentication Configuration for WS-Management
CredSSP authentication allows the user credentials on this computer to be sent
to a remote computer. If you use CredSSP authentication for a connection to
a malicious or compromised computer, that machine will have access to your
username and password. For more information, see the Enable-WSManCredSSP Help
topic.
Do you want to enable CredSSP authentication?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): y

cfg         : http://schemas.microsoft.com/wbem/wsman/1/config/client/auth
lang        : en-US
Basic       : true
Digest      : true
Kerberos    : true
Negotiate   : true
Certificate : true
CredSSP     : true
```

*清单 8-24：在客户端计算机上启用 CredSSP 支持*

通过将值 `Client` ❷ 传递给 `Role` 参数 ❶，你可以在客户端启用 CredSSP。你还需要使用必需的 `DelegateComputer` 参数 ❸，因为 PowerShell 需要知道哪些计算机被允许使用你将委派的凭据。你可以将星号（`*`）传递给 `DelegateComputer`，以允许将凭据委派给所有计算机，但出于安全原因，最好只允许你正在使用的计算机，在这种情况下是 `WEBSRV1`。

一旦在客户端启用了 CredSSP，你需要在服务器上执行相同的操作（清单 8-25）。幸运的是，你可以直接打开一个新的远程会话而不使用 CredSSP，然后在会话内启用 CredSSP，而不必使用 Microsoft 远程桌面访问服务器或亲自访问它。

```
PS> Invoke-Command -ComputerName WEBSRV1 -ScriptBlock { Enable-WSManCredSSP -Role Server }

CredSSP Authentication Configuration for WS-Management CredSSP authentication allows the server
to accept user credentials from a remote computer. If you enable CredSSP authentication on the
server, the server will have access to the username and password of the
client computer if the client computer sends them. For more information, see the Enable-WSManCredSSP Help topic.
Do you want to enable CredSSP authentication?
[Y] Yes  [N] No  [?] Help (default is "Y"): y

#text
-----
False
True
True
False
True
Relaxed
```

*清单 8-25：在服务器计算机上启用 CredSSP 支持*

这样，你就已在客户端和服务器上启用了 CredSSP：客户端允许将用户凭据委托给远程服务器，而远程服务器本身也启用了 CredSSP。现在，你可以再次尝试从该远程会话访问远程网络资源（参见清单 8-26）。请注意，如果你需要撤销启用 CredSSP，命令`Disable-WsmanCredSSP`将恢复你的更改。

```
PS> Invoke-Command -ComputerName WEBSRV1 -ScriptBlock { Get-ChildItem -Path '\\dc\c$'  } 
❶-Authentication Credssp ❷-Credential (Get-Credential)

cmdlet Get-Credential at command pipeline position 1
Supply values for the following parameters:
Credential

    Directory: \\dc\c$

Mode                LastWriteTime         Length Name                            PSComputerName
----                -------------         ------ ----                            --------------
d-----        10/1/2019  12:05 PM                FileShare                       WEBSRV1
d-----       11/24/2019   2:28 PM                inetpub                         WEBSRV1
d-----       11/22/2019   6:37 PM                InstallWindowsFeature           WEBSRV1
d-----        4/16/2019   1:10 PM                Iperf                           WEBSRV1
```

*清单 8-26：通过 CredSSP 认证的会话访问网络资源*

请注意，你必须明确告诉`Invoke-Command`（或`Enter-PSSession`）你希望使用 CredSSP 认证❶，并且无论你使用哪个命令，都需要提供凭据。你可以通过使用`Get-Credential`命令来获取凭据，而不是默认的 Kerberos 认证❷。

执行`Invoke-Command`并为`Get-Credential`提供具有访问 DC 上`c$`共享的用户名和密码后，你可以看到`Get-ChildItem`命令按预期工作！

### 总结

PowerShell 远程执行是目前最简单的在远程系统上执行代码的方法。如你在本章所学，PowerShell 远程执行功能易于使用且直观。一旦你掌握了脚本块的概念以及其中代码的执行位置，远程脚本块将成为你的第二天性。

在本书的第三部分—你将构建自己的强大 PowerShell 模块—你几乎会在每个命令中使用 PowerShell 远程执行。如果你在本章中遇到困难，请再读一遍或开始尝试实验。尝试不同的场景，打破它们，修复它们，做任何你能做的事情来理解 PowerShell 远程执行。这是你从本书中可以学习的最重要的技能之一。

第九章介绍了另一个重要技能：使用 Pester 进行测试。
