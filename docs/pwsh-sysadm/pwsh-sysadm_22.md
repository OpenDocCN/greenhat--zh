## 第十九章：重构你的代码

![图片](img/common.jpg)

在前一章中，你使用现有的虚拟化管理程序、操作系统 ISO 文件和少量代码构建了一个运行 SQL 服务器的虚拟机。这样做意味着你将前几章中创建的多个函数链接在一起。在这里，你将做一些不同的事情：你不再向 PowerLab 模块添加新功能，而是深入研究代码，看看是否可以让你的模块更模块化。

当我说*模块化*时，我指的是将代码的功能拆分为可重用的函数，这些函数能够处理多种情况。代码越模块化，它的通用性就越强。而代码的通用性越强，它就越有用。通过模块化代码，你可以重用像`New-PowerLabVM`或`Install-PowerLabOperatingSystem`这样的函数来安装多种类型的服务器（你将在下一章中看到）。

### 再看一下`New-PowerLabSqlServer`

在第十八章中，你创建了两个主要函数：`New-PowerLabSqlServer`和`Install-PowerLabSqlServer`。你这样做的目的是为了设置一个 SQL 服务器。但如果你想让你的函数更具通用性呢？毕竟，不同的服务器与 SQL 服务器有很多相同的组件：虚拟机、虚拟磁盘、Windows 操作系统等等。你可以简单地复制你已有的函数，然后将所有特定的 SQL 引用替换为你想要的服务器类型的引用。

但我必须建议你不要这样做。没有必要写那么多额外的代码。相反，你只需要重构现有的代码。*重构*指的是在不改变功能的情况下，改变代码内部结构；换句话说，重构是为你，程序员，所做的事情。它帮助代码变得更易读，并确保你在扩展项目时不会遇到太多让人头疼的组织问题。

让我们首先看看你创建的`New-PowerLabSqlServer`函数，见清单 19-1。

```
function New-PowerLabSqlServer { 
    [CmdletBinding()] 
 ❶ param 
    ( 
        [Parameter(Mandatory)] 
        [string]$Name, 

        [Parameter(Mandatory)] 
        [pscredential]$DomainCredential, 

        [Parameter(Mandatory)] 
        [pscredential]$VMCredential, 

        [Parameter()] 
        [string]$VMPath = 'C:\PowerLab\VMs', 

        [Parameter()] 
        [int64]$Memory = 4GB, 

        [Parameter()] 
        [string]$Switch = 'PowerLab', 

        [Parameter()] 
        [int]$Generation = 2, 

        [Parameter()] 
        [string]$DomainName = 'powerlab.local', 

        [Parameter()] 
     ❷ [string]$AnswerFilePath = "C:\Program Files\WindowsPowerShell\Modules
           \PowerLab\SqlServer.ini"
    ) 

 ❸ ## Build the VM 
    $vmparams = @{  
        Name       = $Name 
        Path       = $VmPath 
        Memory     = $Memory 
        Switch     = $Switch 
        Generation = $Generation 
    } 
    New-PowerLabVm @vmParams 

    Install-PowerLabOperatingSystem -VmName $Name 
    Start-VM -Name $Name 

    Wait-Server -Name $Name -Status Online -Credential $VMCredential 

  $addParams = @{ 
        DomainName = $DomainName 
        Credential = $DomainCredential 
        Restart    = $true 
        Force      = $true 
    } 
    Invoke-Command -VMName $Name -ScriptBlock { Add-Computer
    @using:addParams } -Credential $VMCredential 

    Wait-Server -Name $Name -Status Offline -Credential $VMCredential 

 ❹ Wait-Server -Name $Name -Status Online -Credential $DomainCredential 

    $tempFile = Copy-Item -Path $AnswerFilePath -Destination "C:\Program
    Files\WindowsPowerShell\Modules\PowerLab\temp.ini" -PassThru 

    Install-PowerLabSqlServer -ComputerName $Name -AnswerFilePath $tempFile
    .FullName -DomainCredential $DomainCredential 
}
```

*清单 19-1: `New-PowerLabSqlServer`函数*

你打算如何重构这段代码？首先，你知道每个服务器都需要一个虚拟机、一个虚拟磁盘和一个操作系统；你在❸和❹之间的代码块中处理了这些需求。

然而，如果你查看这段代码，你会发现你不能简单地将其提取出来并粘贴到一个新函数中。在`New-PowerLabSqlServer`函数❶中定义的参数在这些行中被使用。请注意，这里唯一特定于 SQL 的参数是`AnswerFilePath`❷。

现在你已经找出了那些与 SQL 无关的代码，让我们将其提取出来并用它来创建新的函数`New-PowerLabServer`（清单 19-2）。

```
function New-PowerLabServer { 
    [CmdletBinding()] 
    param 
    ( 
        [Parameter(Mandatory)] 
        [string]$Name, 

 [Parameter(Mandatory)] 
        [pscredential]$DomainCredential, 

        [Parameter(Mandatory)] 
        [pscredential]$VMCredential, 

        [Parameter()] 
        [string]$VMPath = 'C:\PowerLab\VMs', 

        [Parameter()] 
        [int64]$Memory = 4GB, 

        [Parameter()] 
        [string]$Switch = 'PowerLab', 

        [Parameter()] 
        [int]$Generation = 2, 

        [Parameter()] 
        [string]$DomainName = 'powerlab.local' 
    ) 

    ## Build the VM 
    $vmparams = @{  
        Name       = $Name 
        Path       = $VmPath 
        Memory     = $Memory 
        Switch     = $Switch 
        Generation = $Generation 
    } 
    New-PowerLabVm @vmParams 

    Install-PowerLabOperatingSystem -VmName $Name 
    Start-VM -Name $Name 

    Wait-Server -Name $Name -Status Online -Credential $VMCredential 

    $addParams = @{ 
        DomainName = $DomainName 
        Credential = $DomainCredential 
        Restart    = $true 
        Force      = $true 
    } 
    Invoke-Command -VMName $Name
    -ScriptBlock { Add-Computer @using:addParams } -Credential $VMCredential 

    Wait-Server -Name $Name -Status Offline -Credential $VMCredential 

    Wait-Server -Name $Name -Status Online -Credential $DomainCredential 
}
```

*清单 19-2: 更通用的`New-PowerLabServer`函数*

此时，你有了一个通用的服务器配置函数，但没有办法指明你要创建的是哪种服务器。让我们通过使用另一个名为`ServerType`的参数来解决这个问题：

```
[Parameter(Mandatory)] 
[ValidateSet('SQL', 'Web', 'Generic')] 
[string]$ServerType
```

注意新的`ValidateSet`参数。我将在本章稍后深入解释它的作用；现在，你只需要知道的是，它确保用户只能传入此集合中的服务器类型。

现在你有了这个参数，让我们来使用它。在函数的末尾插入一个`switch`语句，根据用户输入的服务器类型执行不同的代码：

```
switch ($ServerType) { 
    'Web' { 
        Write-Host 'Web server deployments are not supported at this time' 
        break 
    } 
    'SQL' { 
        $tempFile = Copy-Item -Path $AnswerFilePath -Destination "C:\Program
        Files\WindowsPowerShell\Modules\PowerLab\temp.ini" -PassThru 
        Install-PowerLabSqlServer -ComputerName $Name -AnswerFilePath
        $tempFile.FullName -DomainCredential $DomainCredential 
        break 
    } 
    'Generic' { 
        break 
    } 
 ❶ default { 
        throw "Unrecognized server type: [$_]" 
    } 
}
```

如你所见，你处理了三种类型的服务器输入（并使用`default`情况来处理任何异常❶）。但这里有个问题。为了填写 SQL 代码，你从`New-PowerLabSqlServer`函数中复制并粘贴了代码，而现在你使用了你没有的东西：`AnswerFilePath`变量。回想一下，当你将通用代码移到新函数时，你将这个变量留下了，这意味着你无法在这里使用它……还是可以吗？

### 使用参数集

在像前面这样的情况下，当你有一个参数决定需要哪个其他参数时，PowerShell 有一个非常方便的功能叫做*参数集*。你可以将参数集视为允许你使用条件逻辑来控制用户输入哪些参数。

在这个示例中，你将使用三个参数集：一个用于配置 SQL 服务器，一个用于配置 Web 服务器，以及一个默认集。

你可以通过使用`ParameterSetName`属性并跟上一个名称来定义参数集。以下是一个示例：

```
[Parameter(Mandatory)] 
[ValidateSet('SQL', 'Web', 'Generic')] 
[string]$ServerType, 

[Parameter(ParameterSetName = 'SQL')] 
[string]$AnswerFilePath = "C:\Program Files\WindowsPowerShell\Modules\PowerLab\SqlServer.ini", 

[Parameter(ParameterSetName = 'Web')] 
[switch]$NoDefaultWebsite
```

注意你没有为`ServerType`分配参数集。未属于任何参数集的参数可以与任何集一起使用。因此，你可以将`ServerType`与`AnswerFilePath`或你将用于 Web 服务器配置的新增参数`CreateDefaultWebsite`一起使用。

你可以看到这里大部分参数保持不变，但你根据为`ServerType`传入的值添加了一个最终的参数：

```
PS> New-PowerLabServer -Name WEBSRV -DomainCredential CredentialHere -VMCredential CredentialHere -ServerType 'Web' -NoDefaultWebsite 
PS> New-PowerLabServer -Name SQLSRV -DomainCredential CredentialHere -VMCredential CredentialHere -ServerType 'SQL' -AnswerFilePath 'C:\OverridingTheDefaultPath\SqlServer.ini'
```

如果你尝试混合并匹配，同时使用两个不同参数集中的参数，你将会失败：

```
PS> New-PowerLabServer -Name SQLSRV -DomainCredential CredentialHere -VMCredential CredentialHere -ServerType 'SQL' -NoDefaultWebsite -AnswerFilePath 'C:\OverridingTheDefaultPath\SqlServer.ini'

New-PowerLabServer : Parameter set cannot be resolved using the specified named parameters. 
At line:1 char:1 
+ New-PowerLabServer -Name SQLSRV -ServerType 'SQL' -NoDefaultWebsite - ... 
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 
    + CategoryInfo          : InvalidArgument: (:) [New-PowerLabServer], ParameterBindingException 
    + FullyQualifiedErrorId : AmbiguousParameterSet,New-PowerLabServer
```

如果你做相反的操作，既不使用`NoDefaultWebsite`参数也不使用`AnswerFilePath`参数，会发生什么呢？

```
PS> New-PowerLabServer -Name SQLSRV -DomainCredential CredentialHere -VMCredential CredentialHere
-ServerType 'SQL' 
New-PowerLabServer : Parameter set cannot be resolved using the specified named parameters. 
At line:1 char:1 
+ New-PowerLabServer -Name SQLSRV -DomainCredential $credential... 
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 
    + CategoryInfo          : InvalidArgument: (:) [New-PowerLabServer], ParameterBindingException
    + FullyQualifiedErrorId : AmbiguousParameterSet,New-PowerLabServer
PS> New-PowerLabServer -Name WEBSRV -DomainCredential CredentialHere -VMCredential CredentialHere -ServerType 'Web'
New-PowerLabServer : Parameter set cannot be resolved using the specified named parameters. 
At line:1 char:1 
+ New-PowerLabServer -Name WEBSRV -DomainCredential $credential... 
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 
    + CategoryInfo          : InvalidArgument: (:) [New-PowerLabServer], ParameterBindingException
    + FullyQualifiedErrorId : AmbiguousParameterSet,New-PowerLabServer
```

你将得到与之前相同的错误，提示无法解析参数集。为什么？因为 PowerShell 不知道该使用哪个参数集！之前我说过你将使用三个集，但你只定义了两个。你需要设置一个默认的参数集。正如你之前看到的，未明确分配给参数集的参数可以与任何集中的参数一起使用。但是，如果你定义了默认的参数集，PowerShell 将在没有任何集参数被使用的情况下使用这些参数。

至于你的默认集，你可以选择定义的 SQL 或 Web 参数集作为默认值，或者你也可以简单地定义一个不特定的参数集，比如 `blah blah`，这将为所有没有明确定义集的参数创建一个默认集：

```
[CmdletBinding(DefaultParameterSetName = 'blah blah')]
```

如果你不想将某个已定义的参数集设置为默认值，可以将其设置为任何值，只要*没有使用参数集中的任何参数*，PowerShell 将会忽略这两个参数集。在这种情况下，你需要这样做；不使用已定义的参数集是完全可以的，因为你有 `ServerType` 参数来指示你是否要部署 Web 服务器或 SQL 服务器。

使用你新的参数集，`New-PowerLabServer` 函数的参数部分看起来像是 清单 19-3。

```
function New-PowerLabServer { 
    [CmdletBinding(DefaultParameterSetName = 'Generic')] 
    param 
    ( 
        [Parameter(Mandatory)] 
        [string]$Name, 

        [Parameter(Mandatory)] 
        [pscredential]$DomainCredential, 

        [Parameter(Mandatory)] 
        [pscredential]$VMCredential, 

        [Parameter()] 
        [string]$VMPath = 'C:\PowerLab\VMs', 

        [Parameter()] 
        [int64]$Memory = 4GB, 

 [Parameter()] 
        [string]$Switch = 'PowerLab', 
        [Parameter()]
        [int]$Generation = 2, 

        [Parameter()] 
        [string]$DomainName = 'powerlab.local', 

        [Parameter()] 
        [ValidateSet('SQL', 'Web')] 
        [string]$ServerType, 

        [Parameter(ParameterSetName = 'SQL')] 
        [string]$AnswerFilePath = "C:\Program Files\WindowsPowerShell\Modules
        \PowerLab\SqlServer.ini",

        [Parameter(ParameterSetName = 'Web')] 
        [switch]$NoDefaultWebsite 
    )
```

*清单 19-3：新的 `New-PowerLabServer` 函数*

请注意，你有一个对函数 `Install-PowerLabSqlServer` 的引用。这个函数看起来和将我们带入困境的函数（`New-PowerLabSqlServer`）相似。不同的是，`Install-PowerLabSqlServer` 在 `New-PowerLabServer` 完成后接管，安装 SQL 服务器软件并进行基本配置。你可能会倾向于对这个函数进行同样的重构。你可以这么做，但一旦你查看 `Install-PowerLabSqlServer` 中的代码，你会很快意识到，SQL 服务器的安装阶段与其他类型服务器的安装几乎没有共同点。这是一个独特的过程，且很难为其他服务器部署“通用化”。

### 总结

好吧，现在代码已经很好地重构了，你剩下的是一个可以……提供 SQL 服务器的函数。那么你是不是回到原点了呢？希望不是！即使你没有改变代码的功能，你已经构建了一个基础，方便你在下章中插入创建 Web 服务器的代码。

正如你在本章中看到的，重构 PowerShell 代码并不是一个简单明了的过程。了解如何重构代码，以及在当前情况下哪种方式最适合，是一种通过经验获得的技能。但只要你始终牢记程序员所说的*DRY 原则*（不要重复自己），你就会走在正确的道路上。最重要的是，遵循 DRY 原则意味着避免重复代码和冗余功能。你在本章中看到了这一点，当你选择创建一个通用函数来创建新服务器，而不是另一个 `New-PowerLab`InsertServerTypeHere`Server` 函数时。

你辛苦的工作没有白费。在下一章中，你将重新开始自动化，添加创建 IIS Web 服务器所需的代码。
