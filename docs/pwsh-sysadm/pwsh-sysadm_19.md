## 第十六章：安装操作系统

![Images](img/common.jpg)

在前一章中，你已设置好 PowerLab 模块，准备好开始。现在，你将迈出自动化旅程的下一步：学习自动化操作系统的安装。既然你已经创建了一个带有 VHD 的虚拟机，接下来需要安装 Windows。为此，你将使用 Windows Server ISO 文件、*Convert-WindowsImage.ps1* PowerShell 脚本，以及大量脚本来创建一个完全自动化的 Windows 部署！

### 前提条件

我假设你已经跟随前一章的内容，并满足了所有的前提条件。在这里，你将需要一些额外的工具以便继续操作。首先，由于你将要部署操作系统，你需要一个 Windows Server 2016 ISO 文件。你可以通过登录免费的 Microsoft 账户，在[*http://bit.ly/2r5TPRP*](http://bit.ly/2r5TPRP) 下载一个免费试用版。

从前一章开始，我期望你在 Hyper-V 服务器上创建了一个*C:\PowerLab*文件夹。现在，你应该在其中创建一个 ISOs 子文件夹，*C:\PowerLab\ISOs*，并将你的 Windows Server 2016 ISO 文件放入其中。撰写本文时，ISO 文件名为*en_windows_server_2016_x64_dvd_9718492.iso*。你将在脚本中使用此文件路径，因此如果你的路径不同，请确保相应地更新脚本代码。

你还需要在 PowerLab 模块文件夹中有*Convert-WindowsImage.ps1* PowerShell 脚本。如果你下载了本书的资源，这个脚本将与本章的资源一起提供。

还有一些事情：我期望你已经在 Hyper-V 服务器上创建了前一章中的 LABDC 虚拟机。你将使用它作为关联新创建的虚拟磁盘的地方。

最后，你需要一个无人值守的 XML 答案文件（也可以通过本章的可下载资源获取），名为*LABDC.xml*，位于 PowerLab 模块文件夹中。

和往常一样，运行本章附带的*Prerequisites.Tests**.ps1* Pester 测试脚本，以确保你事先满足所有的前提条件。

### 操作系统部署

在自动化操作系统部署时，你将使用三个基本组件：

+   一个包含操作系统位的 ISO 文件

+   一个提供所有通常在安装时手动输入的答案文件

+   微软的 PowerShell 脚本，用于将 ISO 文件转换为 VHDX

你的任务是找出一种方法，将所有这些组件组合在一起。大部分繁重的工作是由答案文件和 ISO 转换脚本完成的。你需要做的是创建一个小脚本，确保转换脚本使用适当的参数调用，并将新创建的 VHD 附加到相应的虚拟机。

你可以通过在下载的资源中找到名为*Install-LABDCOperatingSystem.ps1*的脚本来跟着操作。

#### 创建 VHDX

LABDC 虚拟机将拥有一个 40GB 动态 VHDX 磁盘，分区为 GUID 分区表（GPT），运行 Windows Server 2016 Standard Core。转换脚本需要这些信息。它还需要知道源 ISO 文件的路径以及无人值守答案文件的路径。

首先，定义 ISO 文件和预填充答案文件的路径：

```
$isoFilePath = 'C:\PowerLab\ISOs\en_windows_server_2016_x64_dvd_9718492.iso'
$answerFilePath = 'C:\PowerShellForSysAdmins\PartII\Automating Operating System Installs\LABDC.xml'
```

接下来，你将构建转换脚本的所有参数。使用 PowerShell 的 splatting 技术，你可以创建一个单一的哈希表并将所有这些参数作为一个整体定义。这种定义和传递参数的方式比在一行中键入每个参数要更清晰：

```
$convertParams = @{
    SourcePath        = $isoFilePath
    SizeBytes         = 40GB
    Edition           = 'ServerStandardCore'
    VHDFormat         = 'VHDX'
    VHDPath           = 'C:\PowerLab\VHDs\LABDC.vhdx'
    VHDType           = 'Dynamic'
    VHDPartitionStyle = 'GPT'
    UnattendPath      = $answerFilePath
}
```

一旦为转换脚本定义了所有参数，你将对 *Convert-WindowsImage.ps1* 脚本进行点源（dot source）。你不想直接调用这个转换脚本，因为它包含一个名为 `Convert-WindowsImage` 的函数。如果你只是执行 *Convert-WindowsImage.ps1* 脚本，什么也不会发生，因为它只会加载脚本中的函数。

*点源*是一种将函数加载到内存中以供后续使用的方法；它会加载脚本中定义的所有函数到当前会话中，但不会实际执行它们。以下是如何点源 *Convert-WindowsImage.ps1* 脚本：

```
. "$PSScriptRoot\Convert-WindowsImage.ps1"
```

看看这段代码。这里有一个新变量：`$PSScriptRoot`。这是一个自动变量，表示脚本所在文件夹的路径。在这个例子中，由于*Convert-WindowsImage.ps1*脚本与 PowerLab 模块位于同一文件夹，所以你在 PowerLab 模块中引用了该脚本。

一旦转换脚本被点源到会话中，你就可以调用其中的函数，包括 `Convert-WindowsImage`。这个函数会为你完成所有繁重的工作：它会打开 ISO 文件，适当格式化新的虚拟磁盘，设置启动卷，注入你提供的答案文件，最终生成一个可以启动的新 Windows 系统的 VHDX 文件！

```
Convert-WindowsImage @convertParams

Windows(R) Image to Virtual Hard Disk Converter for Windows(R) 10
Copyright (C) Microsoft Corporation.  All rights reserved.
Version 10.0.9000.0.amd64fre.fbl_core1_hyp_dev(mikekol).141224-3000 Beta

INFO   : Opening ISO en_windows_server_2016_x64_dvd_9718492.iso...
INFO   : Looking for E:\sources\install.wim...
INFO   : Image 1 selected (ServerStandardCore)...
INFO   : Creating sparse disk...
INFO   : Attaching VHDX...
INFO   : Disk initialized with GPT...
INFO   : Disk partitioned
INFO   : System Partition created
INFO   : Boot Partition created
INFO   : System Volume formatted (with DiskPart)...
INFO   : Boot Volume formatted (with Format-Volume)...
INFO   : Access path (F:\) has been assigned to the System Volume...
INFO   : Access path (G:\) has been assigned to the Boot Volume...
INFO   : Applying image to VHDX. This could take a while...
INFO   : Applying unattend file (LABDC.xml)...
INFO   : Signing disk...
INFO   : Image applied. Making image bootable...
INFO   : Drive is bootable. Cleaning up...
INFO   : Closing VHDX...

INFO   : Closing Windows image...
INFO   : Closing ISO...

INFO   : Done.
```

使用社区脚本，如 *Convert-WindowsImage.ps1*，是加速开发的好方法。这个脚本节省了大量时间，而且由于它是由 Microsoft 创建的，你可以信任它。如果你对这个脚本做了什么感到好奇，随时可以打开它。它做了很多事情，我个人很高兴我们有这样的资源来自动化操作系统安装。

#### 附加虚拟机

当转换脚本完成时，你应该在 *C:\PowerLab\VHDs* 目录下找到一个准备启动的 *LABDC.vhdx* 文件。但你还没有完成。按现有状态，这个虚拟磁盘并没有附加到虚拟机。你必须将这个虚拟磁盘附加到一个现有的虚拟机（你将使用之前创建的 LABDC 虚拟机）。

就像你在前一章节中做的那样，使用 `Add-VmHardDiskDrive` 函数将虚拟磁盘附加到你的虚拟机：

```
$vm = Get-Vm -Name 'LABDC'
Add-VMHardDiskDrive -VMName 'LABDC' -Path 'C:\PowerLab\VHDs\LABDC.vhdx'
```

你需要从这个磁盘启动，所以让我们确保它在正确的启动顺序中。你可以使用`Get-VMFirmware`命令并查看`BootOrder`属性来发现现有的启动顺序：

```
$bootOrder = (Get-VMFirmware -VMName 'LABDC').Bootorder
```

注意，启动顺序中的第一个启动设备是网络启动。这不是你想要的。你希望虚拟机从你刚创建的磁盘启动。

```
$bootOrder.BootType

BootType
------
Network
```

要将你刚创建的 VHDX 设置为第一个启动设备，使用`Set-VMFirmware`命令和`FirstBootDevice`参数：

```
$vm | Set-VMFirmware -FirstBootDevice $vm.HardDrives[0]
```

到此为止，你应该已经拥有一个名为 LABDC 的虚拟机，并附加了一个将启动到 Windows 的虚拟磁盘。使用`Start-VM -Name LABDC`启动虚拟机，并确保它成功启动到 Windows。如果是这样，那么你完成了！

### 自动化操作系统部署

到目前为止，你已经成功创建了一个名为 LABDC 的虚拟机，它可以启动 Windows。现在需要意识到，你正在使用的脚本是专门为你的单个虚拟机量身定制的。在实际工作中，你很少能享有这种奢侈。一个好的脚本是可重用和可移植的，这意味着它不需要针对每个特定的输入进行更改，而是围绕一组不断变化的参数值进行工作。

让我们来看一下 PowerLab 模块中的`Install-PowerLabOperatingSystem`函数，它可以在本章的可下载资源中找到。这个函数很好地展示了如何将你正在使用的*Install-LABDCOperatingSystem.ps1*脚本转换为一个可以跨多个虚拟磁盘部署操作系统的脚本，只需简单地更改参数值。

在这一节中，我不会覆盖整个脚本，因为我们在上一节中已经讲解了大部分功能，但我确实想指出一些不同之处。首先，注意你使用了更多的变量。变量让你的脚本更具灵活性。它们为值提供了占位符，而不是将内容直接硬编码到代码中。

另外，还要注意脚本中的条件逻辑。查看 Listing 16-1 中的代码。这是一个`switch`语句，根据操作系统名称查找 ISO 文件路径。在之前的脚本中不需要这个，因为所有内容都是硬编码到脚本中的。

因为`Install-PowerLabOperatingSystem`函数有一个`OperatingSystem`参数，所以你可以灵活地安装不同的操作系统。你只需要找到一种方法来处理所有这些操作系统。一个很好的方法是使用`switch`语句，这样你可以轻松地添加更多条件。

```
switch ($OperatingSystem) {
    'Server 2016' {
        $isoFilePath = "$IsoBaseFolderPath\en_windows_server_2016_x64_dvd_9718492.iso"
    }
    default {
 throw "Unrecognized input: [$_]"
    }
}
```

*Listing 16-1: 使用 PowerShell switch 逻辑*

你可以看到，你已经将硬编码的值转移到了参数中。我不能再强调这一点：参数是构建可重用脚本的关键。尽量避免硬编码，并时刻关注那些你需要在运行时更改的值（然后使用参数来处理它们！）。但是，你可能会想，如果你只想偶尔更改某个值怎么办？接下来，你可以看到多个参数都有默认值。这允许你静态地设置“典型”值，然后根据需要进行覆盖。

```
param
(
    [Parameter(Mandatory)]
    [string]$VmName,

    [Parameter()]
    [string]$OperatingSystem = 'Server 2016',

    [Parameter()]
    [ValidateSet('ServerStandardCore')]
    [string]$OperatingSystemEdition = 'ServerStandardCore',

    [Parameter()]
    [string]$DiskSize = 40GB,

    [Parameter()]
    [string]$VhdFormat = 'VHDX',

    [Parameter()]
    [string]$VhdType = 'Dynamic',

    [Parameter()]
    [string]$VhdPartitionStyle = 'GPT',

    [Parameter()]
    [string]$VhdBaseFolderPath = 'C:\PowerLab\VHDs',

    [Parameter()]
    [string]$IsoBaseFolderPath = 'C:\PowerLab\ISOs',

    [Parameter()]
    [string]$VhdPath
)
```

使用 `Install-PowerLabOperatingSystem` 函数，你可以将所有这些内容变成一行代码，支持数十种配置。现在，你有了一块完整的、连贯的代码单元，可以用多种方式调用它，而不需要更改脚本中的任何一行！

### 将加密的凭据存储到磁盘

你很快就会完成项目的这一阶段，但在继续之前，你需要稍微绕个弯。这是因为你即将使用 PowerShell 执行一些需要凭据的操作。在脚本编写中，常常会把敏感信息（例如，用户名/密码组合）以明文形式存储在脚本中。类似地，可能会认为如果在测试环境中进行操作也无妨——但这为未来的工作埋下了危险的伏笔。即使在测试过程中，也要时刻关注安全措施，这样才能在从测试环境转向生产环境时养成良好的安全习惯。

避免在脚本中存储明文密码的一种简单方法是将其加密到文件中。当需要时，脚本可以解密并使用这些密码。幸运的是，PowerShell 提供了一种原生的方式来实现这一点：Windows 数据保护 API。该 API 在 `Get-Credential` 命令的底层被使用，这个命令会返回一个 `PSCredential` 对象。

`Get-Credential` 会创建一个被称为 *安全字符串* 的加密密码形式。一旦转换为安全字符串格式，整个凭据对象就可以通过 `Export-CliXml` 命令保存到磁盘；反之，使用 `Import-CliXml` 命令可以读取 `PSCredential` 对象。这些命令提供了一个便捷的密码管理系统。

在 PowerShell 中处理凭据时，你需要存储 `PSCredential` 对象，这些对象是大多数 `Credential` 参数接受的对象类型。在前面的章节中，你要么是交互式地输入用户名和密码，要么是以明文形式存储它们。但现在你已经入门了，让我们真正开始吧，为你的凭据添加保护。

将 `PSCredential` 对象以加密格式保存到磁盘需要使用 `Export-CliXml` 命令。使用 `Get-Credential` 命令，你可以创建一个用户名和密码的提示，并将结果传递给 `Export-CliXml`，后者接受保存 XML 文件的路径，如 列表 16-2 所示。

```
Get-Credential | Export-CliXml  -Path C:\DomainCredential.xml
```

*列表 16-2：将凭据导出到文件*

如果你打开 XML 文件，它应该像这样：

```
<TN RefId="0">
  <T>System.Management.Automation.PSCredential</T>
  <T>System.Object</T>
  </TN>
  <ToString>System.Management.Automation.PSCredential</ToString>
  <Props>
  <S N="UserName">userhere</S>
  <SS N="Password">ENCRYPTEDTEXTHERE</SS>
  </Props>
  </Obj>
</Objs>
```

现在凭证已经保存到磁盘上，让我们看看如何在 PowerShell 中获取它。使用 `Import-CliXml` 命令来解析 XML 文件并创建 `PSCredential` 对象：

```
$cred = Import-Clixml -Path C:\DomainCredential.xml
$cred | Get-Member

   TypeName: System.Management.Automation.PSCredential

Name                 MemberType Definition
----                 ---------- ----------
Equals               Method     bool Equals(System.Object obj)
GetHashCode          Method     int GetHashCode()
GetNetworkCredential Method     System.Net.NetworkCredential
                                GetNetworkCredential()
GetObjectData        Method     void GetObjectData(System.Runtime...
GetType              Method     type GetType()
ToString             Method     string ToString()
Password             Property   securestring Password {get;}
UserName             Property   string UserName {get;}
```

你将代码设置为只需将 `$cred` 传递给命令中的任何 `Credential` 参数。现在，代码将像你交互式输入一样工作。这种方法简洁明了，但通常你不会在生产环境中使用它，因为加密文本的用户必须也是解密者（这不是加密的本意！）。这种单一用户的要求并不适合大规模应用。但是，话虽如此，在测试环境中，它表现得非常好！

### PowerShell 直接连接

现在，回到我们的项目。通常，当你在 PowerShell 中对远程计算机执行命令时，你需要使用 PowerShell 远程处理。这显然依赖于本地主机和远程主机之间的网络连接。如果你可以简化这个设置，完全不需要担心网络连接，岂不是很好？嗯，你可以！

因为你在 Windows Server 2016 Hyper-V 主机上运行所有自动化操作，你有一个非常有用的功能可以使用：PowerShell 直接连接。*PowerShell 直接连接*是 PowerShell 的一个较新功能，允许你在没有网络连接的情况下在 Hyper-V 服务器上托管的任何虚拟机上运行命令。你不需要提前为虚拟机设置网络适配器（尽管你已经通过无人值守的 XML 文件做了这件事）。

为了方便起见，你将大量使用 PowerShell 直接连接，而不是使用完整的网络堆栈。如果你不这样做，因为你处于工作组环境中，你必须在工作组环境中配置 PowerShell 远程处理——这不是件容易的事（请参见 *[`bit.ly/2D3deUX`](http://bit.ly/2D3deUX)*）。在 PowerShell 中，选择战斗总是一个好主意，而在这里，我选择了最简单的方式！

PowerShell 直接连接与 PowerShell 远程处理几乎相同。它是一种在远程计算机上运行命令的方法。通常，这需要网络连接，但使用 PowerShell 直接连接时，不再需要网络连接。要通过 PowerShell 远程处理启动远程计算机上的命令，你通常会使用带有 `ComputerName` 和 `ScriptBlock` 参数的 `Invoke-Command`：

```
Invoke-Command -ComputerName LABDC -ScriptBlock { hostname }
```

然而，在使用 PowerShell 直接连接时，`ComputerName` 参数会变成 `VMName`，并且添加了一个 `Credential` 参数。通过 PowerShell 直接连接，完全可以像之前的代码一样运行相同的命令，但仅限于 Hyper-V 主机本身。为了简化操作，让我们先将 `PSCredential` 对象保存在磁盘上，这样以后就不需要反复提示输入凭证了。

对于这个例子，使用用户名`powerlabuser`和密码`P@$$w0rd12`：

```
Get-Credential | Export-CliXml -Path C:\PowerLab\VMCredential.xml
```

现在你已经将凭证保存到磁盘，你将解密它并传递给 `Invoke-Command`。让我们读取保存在 *VMCredential.xml* 中的凭证，然后使用该凭证在 LABDC 虚拟机上执行代码：

```
$cred = Import-CliXml -Path C:\PowerLab\VMCredential.xml
Invoke-Command -VMName LABDC -ScriptBlock { hostname } -Credential $cred
```

为了让 PowerShell Direct 正常工作，背后有许多更复杂的操作，但我在这里不会深入探讨这些细节。如果你想全面了解 PowerShell Direct 是如何工作的，我推荐你查看 Microsoft 博客中宣布该功能的文章（[*https://docs.microsoft.com/en-us/virtualization**/hyper-v-on-windows/user-guide/powershell-direct*](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/powershell-direct)）。

### Pester 测试

现在是本章最重要的部分：让我们通过 Pester 测试把一切整合起来！你将遵循与上一章相同的模式，但在这里我想指出测试中的一个关键部分。在本章的 Pester 测试中，你将使用 `BeforeAll` 和 `AfterAll` 块（清单 16-3）。

正如其名称所示，`BeforeAll` 块包含在所有测试之前执行的代码，而 `AfterAll` 块则包含在所有测试之后执行的代码。你使用这些块是因为你需要通过 PowerShell Direct 多次连接到 LABDC 服务器。PowerShell 远程处理和 PowerShell Direct 都支持会话的概念，你在 第一部分（第八章）中学到过。与其让 `Invoke-Command` 创建和销毁多个会话，不如提前定义一个会话并重复使用它。

```
BeforeAll {
    $cred = Import-CliXml -Path C:\PowerLab\VMCredential.xml
    $session = New-PSSession -VMName 'LABDC' -Credential $cred
}

AfterAll {
    $session | Remove-PSSession
}
```

*清单 16-3：* Tests.ps1——BeforeAll *和* AfterAll *块*

你会注意到你是在 `BeforeAll` 块中解密从磁盘保存的凭证。一旦创建了凭证，你将其与虚拟机的名称一起传递给 `New-PSSession` 命令。这是与 第一部分（第八章）中介绍的相同的 `New-PSSession`，但在这里你可以看到，你不是使用 `ComputerName` 作为参数，而是使用 `VMName`。

这将创建一个单一的远程会话，你可以在整个测试过程中重用它。所有测试完成后，Pester 会查看 `AfterAll` 块并移除该会话。这种方法比反复创建会话要高效得多，尤其是当你需要执行数十个或数百个远程执行代码的测试时。

本章资源中的其余脚本内容很简单，遵循了你一直在使用的模式。如你所见，所有 Pester 测试都通过了，这意味着你仍然在正确的轨道上！

```
PS> Invoke-Pester 'C:\PowerShellForSysadmins\Part II\Automating Operating
System Installs\Automating Operating System Installs.Tests.ps1'
Describing Automating Operating System Installs
   Context Virtual Disk
    [+] created a VHDX called LABDC in the expected location 305ms
    [+] attached the virtual disk to the expected VM 164ms
    [+] creates the expected VHDX format 79ms
    [+] creates the expected VHDX partition style 373ms
    [+] creates the expected VHDX type 114ms
    [+] creates the VHDDX of the expected size 104ms
   Context Operating System
    [+] sets the expected IP defined in the unattend XML file 1.07s
    [+] deploys the expected Windows version 65ms
Tests completed in 2.28s
Passed: 8 Failed: 0 Skipped: 0 Pending: 0 Inconclusive: 0
```

### 总结

在本章中，你深入了解了我们的实际项目。你使用了在上一章中构建的现有虚拟机，并且通过手动和自动方式为其部署了操作系统。到目前为止，你已经拥有了一台完全功能的 Windows 虚拟机，准备进入你旅程的下一阶段。

在下一章中，你将为你的 LABDC 虚拟机设置 Active Directory（AD）。设置 AD 将创建一个新的 AD 林和域，在本节结束时，你将有更多的服务器加入该域。
