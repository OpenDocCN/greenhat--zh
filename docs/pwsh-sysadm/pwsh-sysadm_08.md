## 第七章：探索模块

![图片](img/common.jpg)

在前一章节中，你学习了函数。函数将脚本拆分成可管理的单元，使你的代码更加高效、可读。但没有理由认为一个好的函数只能存在于某个脚本或单一会话中。在本章中，你将学习关于*模块*的内容，它是将一组相似的函数打包在一起，并分发供其他人在多个脚本中使用。

从最基本的形式来看，PowerShell 模块就是一个*.psm1*文件扩展名的文本文件，并包含一些可选的附加元数据。其他类型的模块，如不符合这个描述的模块，被称为*二进制模块*和*动态模块*，但它们超出了本书的讨论范围。

任何没有显式放入你会话中的命令，几乎可以肯定都来自一个模块。在本书中，你使用的许多命令都属于微软内置的 PowerShell 模块，但也有第三方模块以及你自己创建的模块。要使用模块，你首先需要安装它。然后，当需要使用模块中的命令时，必须将该模块导入到你的会话中；从 PowerShell v3 开始，PowerShell 会在引用命令时自动导入模块。

本章的开始，你将查看已经安装在你系统中的模块。然后，你将拆解一个模块，了解其不同部分，最后你将学习如何从 PowerShell Gallery 下载并安装 PowerShell 模块。

### 探索默认模块

PowerShell 默认安装了许多模块。在本节中，你将看到如何从会话中发现并导入模块。

#### 在会话中查找模块

你可以通过使用`Get-Module` cmdlet（它本身也是一个模块的一部分）来查看导入到当前会话中的模块。`Get-Module` cmdlet 是一个命令，允许你查看系统上所有可在当前会话中使用的模块。

启动一个全新的 PowerShell 会话并运行`Get-Module`，如示例 7-1 所示。

```
PS> Get-Module

ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   3.1.0.0    Microsoft.PowerShell.Management     {Add-Computer, Add-Content...
--snip--
```

*示例 7-1：使用`Get-Module`命令查看导入的模块*

你在`Get-Module`输出中看到的每一行都是已经导入到当前会话中的模块，这意味着该模块中的所有命令都可以立即使用。`Microsoft.PowerShell.Management`和`Microsoft.PowerShell.Utility`模块是 PowerShell 会话中默认导入的模块。

注意示例 7-1 中的`ExportedCommands`列。这些是你可以从模块中使用的命令。你可以通过使用`Get-Command`并指定模块名称，轻松找到所有这些命令。让我们看看示例 7-2 中`Microsoft.PowerShell.Management`模块中的所有导出命令。

```
PS> Get-Command -Module Microsoft.PowerShell.Management

CommandType     Name                 Version    Source
-----------     ----                 -------    ------
Cmdlet          Add-Computer         3.1.0.0    Microsoft.PowerShell.Management
Cmdlet          Add-Content          3.1.0.0    Microsoft.PowerShell.Management 
--snip--
```

*示例 7-2：查看 PowerShell 模块中的命令*

这些是从该模块导出的所有命令；它们是可以从模块外部显式调用的命令。某些模块作者选择在模块中包含用户无法使用的函数。任何未导出给用户，并且仅在脚本或模块内部执行的函数，称为 *私有函数*，或一些开发者所说的 *助手函数*。

如果不带任何参数使用 `Get-Module`，它会返回所有已导入的模块，但对于那些已安装但未导入的模块，应该怎么办呢？

#### 在计算机上查找模块

要获取所有已安装且可以导入到会话中的模块列表，你可以使用带有 `ListAvailable` 参数的 `Get-Module`，如 清单 7-3 所示。

```
PS> Get-Module –ListAvailable
   Directory: C:\Program Files\WindowsPowerShell\Modules

ModuleType Version    Name              ExportedCommands
---------- -------    ----              ----------------
Script     1.2        PSReadline        {Get-PSReadlineKeyHandler,Set-PSReadlineKeyHandler...

   Directory:\Modules

ModuleType Version    Name              ExportedCommands
---------- -------    ----              ----------------
Manifest   1.0.0.0    ActiveDirectory   {Add-ADCentralAccessPolicyMember...
Manifest   1.0.0.0    AppBackgroundTask {Disable-AppBackgroundTaskDiagnosticLog...
--snip--
```

*清单 7-3：使用 `Get-Module` 查看所有可用模块*

`ListAvailable` 参数告诉 PowerShell 检查几个文件夹，查找其中包含 *.psm1* 文件的子文件夹。然后，PowerShell 会从文件系统读取这些模块，并返回每个模块的名称、一些元数据，以及可以从该模块中使用的所有功能。

PowerShell 会根据模块的类型，在几个默认位置查找磁盘上的模块：

**系统模块** 几乎所有默认安装的 PowerShell 模块都会位于 *C:\Windows\System32\WindowsPowerShell\1.0\Modules*。这个模块路径通常仅用于内部 PowerShell 模块。严格来说，你可以将模块放在这个文件夹里，但不建议这样做。

**所有用户模块** 模块也存储在 *C:\Program Files\WindowsPowerShell\Modules*。这个路径通常被称为 *所有用户* 模块路径，这是你可以放置任何希望所有登录计算机的用户都能使用的模块的地方。

**当前用户模块** 最后，你可以将模块存储在 *C:\Users\<LoggedInUser>\Documents\WindowsPowerShell\Modules*。在这个文件夹中，你会找到所有由你创建或下载的仅对当前用户可用的模块。将模块放在这个路径中，可以实现一些分离，以防多个具有不同需求的用户登录计算机。

当调用 `Get-Module -ListAvailable` 时，PowerShell 会读取所有这些文件夹路径，并返回每个路径中的所有模块。但是，这些并不是唯一可能的模块路径，只是默认路径。

你可以通过使用 `$PSModulePath` 环境变量来告诉 PowerShell 添加一个新的模块路径，该变量定义了每个模块文件夹，并用分号分隔，如 清单 7-4 所示。

```
PS> $env:PSModulePath
C:\Users\Adam\Documents\WindowsPowerShell\Modules;
C:\Program Files\WindowsPowerShell\Modules\Modules;
C:\Program Files (x86)\Microsoft SQL Server\140\Tools\PowerShell\Modules\
```

*清单 7-4：`PSModulePath` 环境变量*

你可以通过对字符串进行解析，向 `PSModulePath` 环境变量添加文件夹，尽管这种技术可能有点高级。下面是一个简短的命令：

```
PS> $env:PSModulePath + ';C;\MyNewModulePath'.
```

然而，要注意，这种更改只会在当前会话中生效。为了使更改持久化，你需要在`Environment` .NET 类上使用`SetEnvironmentVariable()`方法，如下所示：

```
PS> $CurrentValue = [Environment]::GetEnvironmentVariable("PSModulePath", "Machine")
PS> [Environment]::SetEnvironmentVariable("PSModulePath", $CurrentValue + ";C:\
MyNewModulePath", "Machine")
```

现在让我们看看如何通过导入模块来使用你已有的模块。

#### 导入模块

一旦模块文件夹路径被添加到`PSModulePath`环境变量中，你就必须将模块导入到当前会话中。如今，由于 PowerShell 的自动导入功能，如果你安装了一个模块，通常可以先调用你想要的函数，PowerShell 会自动导入该函数所属的模块。不过，理解导入机制仍然很重要。

让我们使用一个默认的 PowerShell 模块，叫做`Microsoft.PowerShell.Management`。在清单 7-5 中，你将运行`Get-Module`两次：第一次是在一个新的 PowerShell 会话中，第二次是在使用`cd`命令之后，`cd`是`Set-Location`的别名，它是`Microsoft.PowerShell.Management`模块中的一个命令。看看会发生什么：

```
PS> Get-Module

ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   3.1.0.0    Microsoft.PowerShell.Utility        {Add-Member, Add-Type...
Script     1.2        PSReadline                          {Get-PSReadlineKeyHandler... 

PS> cd\
PS> Get-Module

ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   3.1.0.0    Microsoft.PowerShell.Management     {Add-Computer, Add-Content...
Manifest   3.1.0.0    Microsoft.PowerShell.Utility        {Add-Member, Add-Type...
Script     1.2        PSReadline                          {Get-PSReadlineKeyHandler....
```

*清单 7-5：使用`cd`后 PowerShell 自动导入`Microsoft.PowerShell.Management`*

如你所见，`Microsoft.PowerShell.Management`会在你使用`cd`后自动导入。自动导入功能通常是有效的。但如果你期望一个模块中的命令可用，而它却不可用，可能是模块本身的问题导致命令未能导入。

要手动导入一个模块，使用`Import-Module`命令，如清单 7-6 所示。

```
PS> Import-Module -Name Microsoft.PowerShell.Management
PS> Import-Module -Name Microsoft.PowerShell.Management -Force
PS> Remove-Module -Name Microsoft.PowerShell.Management
```

*清单 7-6：手动导入模块、重新导入模块和移除模块*

你会注意到这个清单还使用了`Force`参数和`Remove-Module`命令。如果模块已经发生了变化（比如你修改了一个自定义模块），你可以使用带有`Force`参数的`Import-Module`命令来卸载并重新导入该模块。`Remove-Module`会将一个模块从会话中卸载，尽管这个命令并不常用。

### PowerShell 模块的组成部分

现在你已经学会了如何使用 PowerShell 模块，让我们看看它们的具体样子。

#### .psm1 文件

任何带有.*psm1*文件扩展名的文本文件都可以是 PowerShell 模块。为了让这个文件有用，它必须包含函数。虽然不是严格要求，所有模块中的函数最好围绕相同的概念来构建。例如，清单 7-7 展示了一些与软件安装相关的函数。

```
function Get-Software {
    param()
}

function Install-Software {
    param()
}

function Remove-Software {
    param()
}
```

*清单 7-7：处理软件安装的函数*

请注意，每个命令名称中的名词保持不变，只有动词发生变化。这是构建模块时的最佳实践。如果你发现自己需要更改名词，那么你应该考虑将一个模块拆分为多个模块。

#### 模块清单

除了包含函数的 *.psm1* 文件外，你还会有一个模块清单，或者一个 *.psd1* 文件。*模块清单* 是一个可选但推荐的文本文件，以 PowerShell 哈希表的形式编写。这个哈希表包含描述模块元数据的元素。

虽然可以从头开始创建一个模块清单，但 PowerShell 提供了一个 `New-ModuleManifest` 命令，可以为你生成一个模板。让我们使用 `New-ModuleManifest` 为我们的软件包构建一个模块清单，如 清单 7-8 所示。

```
PS> New-ModuleManifest -Path 'C:\Program Files\WindowsPowerShell\Modules\Software\Software.psd1' 
-Author 'Adam Bertram' -RootModule Software.psm1 
-Description 'This module helps in deploying software.'
```

*清单 7-8：使用 `New-ModuleManifest` 来构建模块清单*

此命令会创建一个 *.psd1* 文件，内容如下：

```
#
# Module manifest for module 'Software'
#
# Generated by: Adam Bertram
#
# Generated on: 11/4/2019
#

@{

# Script module or binary module file associated with this manifest.
RootModule = 'Software.psm1'

# Version number of this module.
ModuleVersion = '1.0'

# Supported PSEditions
# CompatiblePSEditions = @()

# ID used to uniquely identify this module
GUID = 'c9f51fa4-8a20-4d35-a9e8-1a960566483e'

# Author of this module
Author = 'Adam Bertram'

# Company or vendor of this module
CompanyName = 'Unknown'

# Copyright statement for this module
Copyright = '(c) 2019 Adam Bertram. All rights reserved.'

# Description of the functionality provided by this module
Description = 'This modules helps in deploying software.'

# Minimum version of the Windows PowerShell engine required by this module
# PowerShellVersion = ''

# Name of the Windows PowerShell host required by this module
# PowerShellHostName = ''
--snip--
}
```

正如你在运行命令时看到的，我没有为许多字段提供参数。我们不会深入讨论模块清单。现在，只需要知道，至少要定义 `RootModule`、`Author`、`Description`，以及可能的 `version`。所有这些属性都是可选的，但最好养成尽可能多地向模块清单添加信息的习惯。

现在你已经了解了模块的结构，接下来我们来看一下如何下载并安装一个模块。

### 使用自定义模块

到目前为止，你一直在使用 PowerShell 默认安装的模块。在本节中，你将学习如何查找、安装和卸载自定义模块。

#### 查找模块

模块的最佳部分之一就是共享它们：为什么要浪费时间解决已经解决的问题呢？如果你遇到问题，PowerShell Gallery 里很可能已有解决方案。[PowerShell Gallery](https://www.powershellgallery.com/) 是一个包含成千上万个 PowerShell 模块和脚本的存储库，任何有账户的人都可以自由上传或下载。这里有由个人编写的模块，也有由像 Microsoft 这样的大公司编写的模块。

幸运的是，你也可以直接使用 PowerShell 中的 Gallery。PowerShell 有一个内置模块叫做 `PowerShellGet`，提供了简单易用的命令与 PowerShell Gallery 交互。清单 7-9 使用 `Get-Command` 来列出 `PowerShellGet` 命令。

```
PS> Get-Command -Module PowerShellGet

CommandType     Name                           Version    Source
-----------     ----                           -------    ------
Function        Find-Command                   1.1.3.1    powershellget
Function        Find-DscResource               1.1.3.1    powershellget
Function        Find-Module                    1.1.3.1    powershellget
Function        Find-RoleCapability            1.1.3.1    powershellget
Function        Find-Script                    1.1.3.1    powershellget
Function        Get-InstalledModule            1.1.3.1    powershellget
Function        Get-InstalledScript            1.1.3.1    powershellget
Function        Get-PSRepository               1.1.3.1    powershellget
Function        Install-Module                 1.1.3.1    powershellget
Function        Install-Script                 1.1.3.1    powershellget
Function        New-ScriptFileInfo             1.1.3.1    powershellget
--snip--
```

*清单 7-9：`PowerShellGet` 命令*

`PowerShellGet` 模块包含用于查找、保存和安装模块的命令，还包括发布你自己的模块。你现在还没有准备好发布模块（你甚至还没创建自己的模块！），所以我们将专注于如何查找和安装来自 PowerShell Gallery 的模块。

要查找一个模块，你可以使用 `Find-Module` 命令，它允许你在 PowerShell Gallery 中搜索与特定名称匹配的模块。例如，如果你正在寻找用于管理 VMware 基础设施的模块，你可以使用通配符和 `Name` 参数来查找所有 PowerShell Gallery 中包含 *VMware* 字样的模块，如 清单 7-10 所示。

```
PS> Find-Module -Name *VMware*

Version      Name                                Repository      Description
-------      ----                                ----------      -----------
6.5.2.6...   VMware.VimAutomation.Core           PSGallery       This Windows... 
1.0.0.5...   VMware.VimAutomation.Sdk            PSGallery       This Windows...
--snip--
```

*清单 7-10：使用 `Find-Module` 查找与 VMware 相关的模块*

`Find-Module` 命令不会下载任何内容；它只会显示 PowerShell Gallery 中的内容。在接下来的部分，你将看到如何安装模块。

#### 安装模块

一旦你有了想要安装的模块，可以使用 `Install-Module` 命令来安装它。`Install-Module` 命令可以带有 `Name` 参数，但我们可以使用管道操作，直接将 `Find-Module` 返回的对象传递给 `Install-Module` 命令（见清单 7-11）。

请注意，你可能会收到关于不受信任的存储库的警告。你会收到此不受信任的警告，因为默认情况下，`Find-Module` 命令使用的是一个不受信任的 PowerShell 存储库，这意味着你必须明确告诉 PowerShell 信任该存储库中的所有包。否则，它会提示你运行 `Set-PSRepository`，如清单 7-11 中所示，以更改该存储库的安装策略。

```
PS> Find-Module -Name VMware.PowerCLI | Install-Module

Untrusted repository You are installing the modules from an untrusted repository. If you trust
this repository, change its InstallationPolicy value by running the Set-PSRepository cmdlet.
Are you sure you want to install the modules from 'https://www.powershellgallery.com/api/v2/'?
[Y] Yes [A] Yes to All [N] No [L] No to All [S] Suspend [?] Help (default is "N"): a
Installing package 'VMware.PowerCLI'
Installing dependent package 'VMware.VimAutomation.Cloud' [oooooooooooooooooooooooooooooooooooo
ooooooooooooooooooooooooo] Installing package 'VMware.VimAutomation.Cloud'
Downloaded 1003175.00 MB out of 1003175.00 MB. [ooooooooooooooooooooooooooooooooooooooooooooooo
oooooooooooooooooooooo]
```

*清单 7-11：使用 `Install-Module` 命令安装模块*

默认情况下，清单 7-11 中的命令将下载模块并将其放置在 *C:\Program Files* 中的所有用户模块路径下。要检查模块是否在该路径中，你可以使用以下命令：

```
PS> Get-Module -Name VMware.PowerCLI -ListAvailable | Select-Object –Property ModuleBase

ModuleBase
----------
C:\Program Files\WindowsPowerShell\Modules\VMware.PowerCLI\6.5.3.6870460
```

#### 卸载模块

刚接触 PowerShell 的新手常常会混淆删除和卸载模块之间的区别。如你在《导入模块》一节中看到的（见第 82 页），你可以使用 `Remove-Module` 来*移除* PowerShell 会话中的模块。但这只是将模块从会话中卸载，并不会从磁盘上删除该模块。

要从磁盘中删除模块——或*卸载*它——你必须使用 `Uninstall-Module` cmdlet。清单 7-12 卸载你刚刚安装的模块。

```
PS> Uninstall-Module -Name VMware.PowerCLI
```

*清单 7-12：卸载模块*

只有从 PowerShell Gallery 下载的模块才能通过 `Uninstall-Module` 卸载——默认模块是无法被卸载的！

### 创建你自己的模块

到目前为止，你一直在使用其他人的模块。当然，PowerShell 模块的一个惊人之处在于你可以创建自己的模块并与全世界分享。你将在本书的第三部分中构建一个真实的模块，但现在，让我们来看一下如何将你的软件模块变成一个真正的模块。

如你之前所见，典型的 PowerShell 模块由一个文件夹（*模块容器*）、一个 *.psm1* 文件（模块文件）和一个 *.psd1* 文件（模块清单）组成。如果模块文件夹位于三个位置之一（系统、所有用户或当前用户），PowerShell 将自动识别并导入它。

让我们首先创建模块文件夹。模块文件夹必须与模块本身同名。由于我通常将模块设置为系统中所有用户可用，你将把它添加到所有用户的模块路径中，像这样：

```
PS> mkdir 'C:\Program Files\WindowsPowerShell\Modules\Software'
```

一旦创建了文件夹，创建一个空白的 *.psm1* 文件，该文件最终将保存你的函数：

```
PS> Add-Content 'C:\Program Files\WindowsPowerShell\Modules\Software\Software.psm1'
```

接下来，按照你在清单 7-8 中的操作，创建模块清单：

```
PS> New-ModuleManifest -Path 'C:\Program Files\WindowsPowerShell\Modules\Software\Software.psd1' 
-Author 'Adam Bertram' -RootModule Software.psm1 
-Description 'This module helps in deploying software.'
```

到此为止，PowerShell 应该能够看到你的模块，但注意它还没有看到任何已导出的命令：

```
PS> Get-Module -Name Software -List

    Directory: C:\Program Files\WindowsPowerShell\Modules

ModuleType Version    Name                      ExportedCommands
---------- -------    ----                      ----------------
Script     1.0        Software
```

现在，让我们把你之前使用的三个函数添加到 *.psm1* 文件中，看看 PowerShell 是否能识别它们：

```
PS> Get-Module -Name Software -List

    Directory: C:\Program Files\WindowsPowerShell\Modules

ModuleType Version    Name                      ExportedCommands
---------- -------    ----                      ----------------
Script     1.0        Software                  {Get-Software...
```

PowerShell 已经导出了你模块中的所有命令，并使其可供使用。如果你想更进一步，选择哪些命令被导出，你还可以打开模块清单，找到 `FunctionsToExport` 键。在那里，你可以定义每个命令，用逗号分隔，这将决定哪些命令被导出。虽然这不是强制性的，但它提供了更细致的模块函数导出控制。

恭喜！你刚刚创建了你的第一个模块！除非你为其中的函数填充实际功能，否则它不会做太多，当然，这也是一个值得你自己完成的有趣挑战。

### 总结

在本章中，你了解了模块，这是一些志同道合的代码集合，能帮助你避免在已经解决的问题上浪费时间。你看到了模块的基本结构，以及如何安装、导入、移除和卸载它们。你甚至创建了自己的基础模块！

在第八章中，你将学习如何通过使用 PowerShell 远程操作来访问远程计算机。
