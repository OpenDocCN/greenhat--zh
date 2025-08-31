## 第十四章：创建服务器清单脚本

![Images](img/common.jpg)

到目前为止，在本书中，你已经专注于学习 PowerShell 作为一种语言，熟悉其语法和命令。但 PowerShell 不仅仅是一个语言，它还是一个工具。既然你已经掌握了 PowerShell 的基本知识，现在是时候进行更有趣的部分了！

PowerShell 的真正力量在于它的工具制作能力。在这个上下文中，*工具*指的是一个 PowerShell 脚本、一个模块、一个函数或任何有助于你执行管理任务的东西。无论任务是创建报告、收集计算机信息、创建公司用户账户，还是更复杂的任务，你都将学习如何使用 PowerShell 自动化这些任务。

在本章中，我将向你展示如何使用 PowerShell 收集数据，以便做出更明智的决策。具体来说，你将构建一个服务器清单项目。你将学习如何创建一个带有参数的脚本，输入服务器名称，并发现大量的信息供你浏览：操作系统规格以及硬件信息，包括存储大小、空闲存储、内存等。

### 先决条件

在开始本章之前，你需要一台已加入域的 Windows 计算机、对 Active Directory 计算机对象的读取权限、一个包含计算机账户的 Active Directory 组织单位（OU），以及可以从 [*https://www.microsoft.com/en-us/download/details.aspx?id=45520*](https://www.microsoft.com/en-us/download/details.aspx?id=45520) 下载的远程服务器管理工具包（RSAT）。

### 创建项目脚本

由于你将在本章中构建脚本，而不仅仅是在控制台中执行代码，首先你需要创建一个新的 PowerShell 脚本。创建一个名为 *Get-ServerInformation.ps1* 的脚本。我把我的脚本放在 *C:\* 目录下。你将在本章中不断地往这个脚本中添加代码。

### 定义最终输出

在你开始编写代码之前，制定一个“草图”计划，确定完成后输出应该是什么样子，这是一个良好的实践。这个简单的草图是衡量进度的一个好方法，尤其是在构建大型脚本时。

对于这个服务器清单脚本，我们假设在脚本结束时，你希望在 PowerShell 控制台中看到如下输出：

```
ServerName  IPAddress  OperatingSystem  AvailableDriveSpace (GB)  Memory (GB)  UserProfilesSize (MB)  StoppedServices
MYSERVER    x.x.x.x    Windows....      10                        4            50.4                   service1,service2,service3
```

现在你知道你想看到的内容，让我们开始实现它。

### 发现与脚本输入

第一步是决定如何告诉你的脚本查询内容。你将从多个服务器收集信息。如在“先决条件”部分所述，你将使用 Active Directory 来查找服务器名称。

当然，你可以从文本文件中查询服务器名称，从存储在 PowerShell 脚本中的服务器名称数组中查询，从注册表中查询，从 Windows 管理工具 (WMI) 库中查询，或者从数据库中查询——这都没关系。只要你的脚本最终获得一个表示服务器名称的字符串数组，你就可以继续进行。不过，在这个项目中，你将使用来自 Active Directory 的服务器。

在这个示例中，所有的服务器都位于同一个 OU。如果你自己尝试时发现它们不在同一个 OU 中，也没关系；你只需要遍历你的 OU，并读取每个 OU 中的计算机对象即可。但在这里，你的第一个任务是读取 OU 中的所有计算机对象。在这个环境中，所有的服务器都位于`Servers` OU。你的域名是`powerlab.local`。要从 AD 中检索计算机对象，使用`Get-ADComputer`命令，如 Listing 14-1 所示。这个命令应该会返回你感兴趣的所有 AD 计算机对象。

```
PS> $serversOuPath = 'OU=Servers,DC=powerlab,DC=local'
PS> $servers = Get-ADComputer -SearchBase $serversOuPath -Filter *
PS> $servers

DistinguishedName : CN=SQLSRV1,OU=Servers,DC=Powerlab,DC=local
DNSHostName       : SQLSRV1.Powerlab.local
Enabled           : True
Name              : SQLSRV1
ObjectClass       : computer
ObjectGUID        : c288d6c1-56d4-4405-ab03-80142ac04b40
SamAccountName    : SQLSRV1$
SID               : S-1-5-21-763434571-1107771424-1976677938-1105
UserPrincipalName :

DistinguishedName : CN=WEBSRV1,OU=Servers,DC=Powerlab,DC=local
DNSHostName       : WEBSRV1.Powerlab.local
Enabled           : True
Name              : WEBSRV1
ObjectClass       : computer
ObjectGUID        : 3bd2da11-4abb-4eb6-9c71-7f2c58594a98
SamAccountName    : WEBSRV1$
SID               : S-1-5-21-763434571-1107771424-1976677938-1106
UserPrincipalName :
```

*Listing 14-1：使用`Get-AdComputer`返回服务器数据*

注意，在这里你不是直接设置`SearchBase`参数的值，而是定义了一个变量。你应该习惯这样做。事实上，每当你遇到类似的具体配置时，把它放到一个变量中总是一个好主意，因为你永远不知道什么时候你还需要再次使用这个值。你还将`Get-ADComputer`的输出返回到一个变量中。由于你稍后还会处理这些服务器，因此你希望能够引用它们的名称。

`Get-ADComputer`命令返回的是整个 AD 对象，但你只需要服务器名称。你可以通过使用`Select-Object`来缩小范围，仅返回`Name`属性：

```
PS> $servers = Get-ADComputer -SearchBase $serversOuPath -Filter * |
Select-Object -ExpandProperty Name
PS> $servers
SQLSRV1
WEBSRV1
```

现在你已经有了查询单个服务器的基本思路，让我们来看一下如何查询所有服务器。

### 查询每台服务器

要查询每台服务器，你需要创建一个循环，这样可以确保每台服务器在你的数组中只被查询一次。

假设你的代码会立即工作通常并不是一个好主意（它通常不会）。相反，我喜欢在构建过程中慢慢进行，并在每个步骤中进行测试。在这种情况下，不要尝试一次性完成所有任务，而是使用`Write-Host`确保脚本返回你预期的服务器名称：

```
foreach ($server in $servers) {
    Write-Host $server
}
```

到现在为止，你应该已经有了一个名为*Get-ServerInformation.ps1*的脚本，内容如 Listing 14-2 所示。

```
$serversOuPath = 'OU=Servers,DC=powerlab,DC=local'
$servers = Get-ADComputer -SearchBase $serversOuPath -Filter * | Select-Object -ExpandProperty Name
foreach ($server in $servers) {
    Write-Host $server
}
```

*Listing 14-2：到目前为止你的脚本*

一旦你运行脚本，你将获得一些服务器名称。根据你使用的服务器不同，输出可能会有所不同：

```
PS> C:\Get-ServerInformation.ps1
SQLSRV1
WEBSRV1
```

很好！你已经设置了一个循环，它会遍历你数组中的每个服务器名称。你的第一个任务已经完成。

### 提前思考：结合不同类型的信息

PowerShell 成功的关键之一是良好的规划和组织。部分内容就是了解预期结果。对于许多初学者来说，他们没有太多关于 PowerShell 可能返回的结果的经验，这是一个问题：他们知道自己希望发生什么（希望如此），但他们不知道*可能*发生什么。因此，他们编写的脚本会在数据源之间“之”字形地穿梭，从一个获取数据，再到另一个，接着是第一个，然后是第三个，将它们连接起来，再做一遍。其实有更简单的方式，我如果不暂停来解释这些，反而会对你造成不利影响。

查看 Listing 14-1 中的输出，你可以看到，你将需要一些命令来从不同的来源提取信息（WMI、文件系统、Windows 服务）。每个来源将返回不同类型的对象，如果你不加思考地将它们合并，你会得到一团糟。

稍微提前一点，让我们看一下，如果你尝试在没有任何格式化或输出关注的情况下提取服务名称和内存，输出会是什么样子。你可能会看到类似这样的内容：

```
Status   Name               DisplayName
------   ----               -----------
Running  wuauserv           Windows Update

__GENUS              : 2
__CLASS              : Win32_PhysicalMemory
__SUPERCLASS         : CIM_PhysicalMemory
__DYNASTY            : CIM_ManagedSystemElement
__RELPATH            : Win32_PhysicalMemory.Tag="Physical Memory 0"
__PROPERTY_COUNT     : 30
__DERIVATION         : {CIM_PhysicalMemory, CIM_Chip, CIM_PhysicalComponent, CIM_PhysicalElement...}
__SERVER             : DC
__NAMESPACE          : root\cimv2
__PATH               : \\DC\root\cimv2:Win32_PhysicalMemory.Tag="Physical Memory 0"
```

在这里，你正在查询一个服务，并同时尝试从服务器获取内存。这些对象不同，这些对象上的属性也不同，如果你将所有输出合并并直接输出，看起来会很糟糕。

让我们看看如何避免这种输出。由于你将组合不同类型的输出，并且你需要符合我们确切规范的内容，因此你必须创建自己类型的输出。别担心，这不像你想象的那么复杂。在第二章中，你学会了如何创建`PSCustomObject`类型。PowerShell 中的这些通用对象允许你添加自己的属性—非常适合你在这里做的事情。

你知道所需输出的标题（并且，正如我相信你现在已经知道的，这些“标题”将始终是对象属性）。让我们创建一个自定义对象，并将你希望在输出中看到的属性放进去。出于明显的原因，我将这个对象命名为`$output`；你在填充它的属性后将返回它：

```
$output = [pscustomobject]@{
    'ServerName'                  = $null
    'IPAddress'                   = $null
    'OperatingSystem'             = $null
    'AvailableDriveSpace (GB)'    = $null
    'Memory (GB)'                 = $null
    'UserProfilesSize (MB)'       = $null
    'StoppedServices'             = $null
}
```

你会注意到哈希表的键被单引号包围。如果键中没有空格，这是不强制的。然而，由于我在一些键名中使用了空格，我决定在所有键上统一使用单引号。通常不推荐在对象属性名称中使用空格，除非使用自定义格式化，但这超出了本书的范围。有关自定义格式化的更多信息，请参阅*about_Format.ps1xml*帮助主题。

如果你将其复制到控制台，并通过格式化命令`Format-Table`返回它，你将看到你所需要的标题：

```
PS> $output | Format-Table -AutoSize

ServerName IPAddress OperatingSystem AvailableDriveSpace (GB) Memory (GB) UserProfilesSize (MB) StoppedServices
```

`Format-Table`命令是 PowerShell 中少数几个格式化命令之一，旨在作为管道中的最后一个命令使用。它们会转换当前的输出并以不同的方式显示它。在这种情况下，你正在告诉 PowerShell 将对象输出转换为表格格式，并根据控制台的宽度自动调整行的大小。

一旦定义了自定义输出对象，你可以返回到循环中，确保每个服务器都以这种格式返回。由于你已经知道服务器名称，可以立即设置该属性，如 Listing 14-3 所示。

```
$serversOuPath = 'OU=Servers,DC=powerlab,DC=local'
$servers = Get-ADComputer -SearchBase $serversOuPath -Filter * | Select-Object -ExpandProperty Name
foreach ($server in $servers) {
    $output = @{
        'ServerName'                  = $server
        'IPAddress'                   = $null
        'OperatingSystem'             = $null
        'AvailableDriveSpace (GB)'    = $null
        'Memory (GB)'                 = $null
        'UserProfilesSize (MB)'       = $null
        'StoppedServices'             = $null
    }
    [pscustomobject]$output
}
```

*Listing 14-3: 将你的`output`对象放入循环中并设置服务器名称*

请注意，你在填充数据后才将 `output` 创建为哈希表并将其转换为 `PSCustomObject`。之所以这样做，是因为将属性值保存在哈希表中比保存在 `PSCustomObject` 中更简单；你只有在输出时才关心 `output` 是该类型的对象，以便当你引入其他信息源时，它们都将是相同的对象类型。

你可以通过以下代码查看你的 `PSCustomObject` 所有属性的名称，以及你正在查询的服务器名称：

```
PS> C:\Get-ServerInformation.ps1 | Format-Table -AutoSize

ServerName UserProfilesSize (MB) AvailableDriveSpace (GB) OperatingSystem StoppedServices IPAddress Memory (GB)
---------- --------------------- ------------------------ --------------- --------------- --------- -----------
SQLSRV1
WEBSRV1
```

如你所见，你已经有了数据。它可能看起来不多，但你已经走在了正确的道路上！

### 查询远程文件

现在你已经知道如何存储数据，接下来只需要获取数据。这意味着需要从每个服务器中提取所需的信息，并仅返回你关心的属性。让我们从 `UserProfileSize`（MB）的值开始。为此，让我们想办法找出每个服务器的 *C:\Users* 文件夹中所有这些配置文件占用了多少空间。

由于你设置了循环的方式，你需要弄清楚如何仅为一个服务器执行此操作。既然你知道文件夹路径是 *C:\Users*，那么让我们先看看你是否能查询到所有服务器的用户配置文件夹下的所有文件。

当你运行 `Get-ChildItem -Path \\WEBSRV1\c$\Users -Recurse -File` 并且有权限访问该文件共享时，你会看到它返回了所有用户配置文件中的所有文件和文件夹，但没有看到任何与大小相关的信息。让我们将输出通过管道传递给 `Select-Object`，以返回所有属性：

```
PS> Get-ChildItem -Path \\WEBSRV1\c$\Users -Recurse -File | Select-Object -Property *

PSPath            : Microsoft.PowerShell.Core\FileSystem::\WEBSRV1\c$\Users\Adam\file.log
PSParentPath      : Microsoft.PowerShell.Core\FileSystem::\\WEBSRV1\c$\Users\Adam
PSChildName       : file.log
PSProvider        : Microsoft.PowerShell.Core\FileSystem
PSIsContainer     : False
Mode              : -a----
VersionInfo       : File:             \\WEBSRV1\c$\Users\Adam\file.log
                    InternalName:
                    OriginalFilename:
                    FileVersion:
                    FileDescription:
                    Product:
                    ProductVersion:
                    Debug:            False
                    Patched:          False
                    PreRelease:       False
                    PrivateBuild:     False
                    SpecialBuild:     False
                    Language:
BaseName          : file
Target            :
LinkType          :
Name              : file.log
Length            : 8926
DirectoryName     : \\WEBSRV1\c$\Users\Adam
--snip--
```

`Length` 属性显示文件的大小（以字节为单位）。知道这一点后，你需要计算服务器 *C:\Users* 文件夹中每个文件的 `Length` 值的总和。幸运的是，PowerShell 通过其中一个 cmdlet `Measure-Object` 使这一过程变得简单。这个 cmdlet 接受来自管道的输入，并自动将特定属性的值加总起来：

```
PS> Get-ChildItem -Path '\\WEBSRV1\c$\Users\' -File -Recurse | Measure-Object -Property Length -Sum

Count    : 15
Average  :
Sum      : 600554
Maximum  :
Minimum  :
Property : Length
```

现在你有了一个属性（`Sum`），可以用来表示输出中总的用户配置文件大小。此时，只需将代码整合到循环中，并在 `$output` 哈希表中设置适当的属性。由于你只需要从 `Measure-Object` 返回的对象中获取 `Sum` 属性，因此你会将命令括在括号中，并像 清单 14-4 中那样引用 `Sum` 属性。

```
Get-ServerInformation.ps1
-------------------
$serversOuPath = 'OU=Servers,DC=powerlab,DC=local'
$servers = Get-ADComputer -SearchBase $serversOuPath -Filter * | Select-Object -ExpandProperty Name
foreach ($server in $servers) {
    $output = @{
        'ServerName'                  = $null
        'IPAddress'                   = $null
        'OperatingSystem'             = $null
        'AvailableDriveSpace (GB)'    = $null
        'Memory (GB)'                 = $null
        'UserProfileSize (MB)'        = $null
        'StoppedServices'             = $null
    }
    $output.ServerName = $server
    $output.'UserProfileSize (MB)' = (Get-ChildItem -Path '\\WEBSRV1\c$\Users\' -File -Recurse |
    Measure-Object -Property Length -Sum).Sum
    [pscustomobject]$output
}
```

*清单 14-4：更新脚本以存储 `UserProfilesSize`*

如果你运行该脚本，结果如下：

```
PS> C:\Get-ServerInformation.ps1 | Format-Table -AutoSize

ServerName UserProfilesSize (MB) AvailableDriveSpace (GB) OperatingSystem StoppedServices IPAddress Memory (GB)
---------- --------------------- ------------------------ --------------- --------------- --------- -----------
SQLSRV1                   636245
WEBSRV1                   600554
```

如你所见，你现在得到了用户配置文件的总大小——但它还不是以兆字节为单位。你计算了 `Length` 的总和，而 `Length` 是以字节为单位的。PowerShell 使得这种转换变得简单：只需将数字除以 `1MB`，就可以得到结果。你可能会看到结果以小数点形式表示。你可以采取最后一步，将输出转换为整数，以确保你得到的是一个整数，这样就可以将数字“四舍五入”到一个完整的兆字节值：

```
$userProfileSize = (Get-ChildItem -Path "\\$server\c$\Users\" -File |
Measure-Object -Property Length -Sum).Sum
$output.'UserProfilesSize (MB)' = int
```

### 查询 Windows 管理工具

你还有五个值需要填充。对于其中四个，你将使用一个名为*Windows 管理工具（WMI）*的微软内置功能。WMI 基于行业标准的通用信息模型（CIM），是一个包含与操作系统及其运行硬件相关的数千个属性的实时信息库。这些信息被分隔成不同的命名空间、类和属性。如果你正在寻找有关计算机的信息，你很可能会经常使用 WMI。

对于这个特定的脚本，你将提取硬盘空间、操作系统版本、服务器的 IP 地址，以及服务器包含的内存量的信息。

PowerShell 有两个命令用于查询 WMI：`Get-WmiObject`和`Get-CimInstance`。`Get-WmiObject`是较旧的命令，灵活性不如`Get-CimInstance`（如果你想了解技术细节：这主要是因为`Get-WmiObject`只使用 DCOM 来连接远程计算机，而`Get-CimInstance`默认使用 WSMAN，也可以选择使用 DCOM）。目前，微软似乎将所有精力都投入到`Get-CimInstance`中，所以你将使用这个命令。关于 CIM 与 WMI 的详细对比，可以参考这篇博客：[*https://blogs.technet.microsoft.com/heyscriptingguy/2016/02/08/should-i-use-cim-or-wmi-with-windows-powershell/*](https://blogs.technet.microsoft.com/heyscriptingguy/2016/02/08/should-i-use-cim-or-wmi-with-windows-powershell/)。

查询 WMI 的最难部分是弄清楚你想要的信息藏在哪里。通常，你需要自己做这个研究（我鼓励你在这里尝试），但为了节省时间，让我为你提供这个脚本的答案：所有存储资源使用情况都在`Win32_LogicalDisk`中，操作系统的信息在`Win32_OperatingSystem`中，Windows 服务都在`Win32_Service`中，任何网络适配器的信息都在`Win32_NetworkAdapterConfiguration`中，内存信息则在`Win32_PhysicalMemory`中。

现在让我们看看如何使用`Get-CimInstance`查询这些 WMI 类，获取你需要的属性。

#### 磁盘剩余空间

我们从可用的硬盘空间开始，这些信息存储在`Win32_LogicalDisk`中。像处理`UserProfilesSize`一样，你将从一台服务器开始，然后在循环中进行泛化。在这里，你很幸运；你甚至不需要使用`Select-Object`来挖掘所有的属性——`FreeSpace`就在这里：

```
PS> Get-CimInstance -ComputerName sqlsrv1 -ClassName Win32_LogicalDisk

DeviceID DriveType ProviderName VolumeName Size        FreeSpace   PSComputerName
-------- --------- ------------ ---------- ----        ---------   --------------
C:       3                                 42708496384 34145906688 sqlsrv1
```

了解到`Get-CimInstance`返回的是一个对象后，你只需访问所需的属性，就能获取到剩余空间的数值：

```
PS> (Get-CimInstance -ComputerName sqlsrv1 -ClassName Win32_LogicalDisk).FreeSpace
34145906688
```

你已经获得了数值，但是像上次一样，它是以字节为单位的（这是 WMI 中的常见情况）。你可以像之前一样进行转换，只不过这次你需要的是千兆字节，所以你要将其除以`1GB`。当你更新脚本，通过将`FreeSpace`属性除以`1GB`时，输出结果大概是这样：

```
PS> C:\Get-ServerInformation.ps1 | Format-Table -AutoSize

ServerName UserProfilesSize (MB) AvailableDriveSpace (GB) OperatingSystem StoppedServices IPAddress Memory (GB)
---------- --------------------- ------------------------ --------------- --------------- --------- -----------
SQLSRV1                   636245          31.800853729248
WEBSRV1                   603942         34.5973815917969
```

你不需要看到 12 位数字的空闲空间，因此可以通过使用`[Math]`类的`Round()`方法进行四舍五入，使输出看起来更好：

```
$output.'AvailableDriveSpace (GB)' = [Math]::Round(((Get-CimInstance -ComputerName $server
-ClassName Win32_LogicalDisk).FreeSpace / 1GB),1)

ServerName UserProfilesSize (MB) AvailableDriveSpace (GB) OperatingSystem StoppedServices IPAddress Memory (GB)
---------- --------------------- ------------------------ --------------- --------------- --------- -----------
SQLSRV1                   636245                     31.8
WEBSRV1                   603942                     34.6
```

现在这些值更容易阅读了。三个已经完成，剩下四个。

#### 操作系统信息

到现在你应该能看到一般的模式：查询单台服务器，找到合适的属性，然后将查询添加到你的`foreach`循环中。

从现在开始，你只需在`foreach`循环中添加行。缩小类、类属性和属性值的过程对于你从 WMI 查询的任何值都是一样的。只需遵循相同的一般模式：

```
$output.'PropertyName' = (Get-CimInstance -ComputerName ServerName 
-ClassName WMIClassName).WMIClassPropertyName
```

添加下一个值后，你的脚本看起来像示例 14-5。

```
Get-ServerInformation.ps1
-------------------
$serversOuPath = 'OU=Servers,DC=powerlab,DC=local'
$servers = Get-ADComputer -SearchBase $serversOuPath -Filter * |
Select-Object -ExpandProperty Name
foreach ($server in $servers) {
    $output = @{
        'ServerName'                  = $null
        'IPAddress'                   = $null
        'OperatingSystem'             = $null
        'AvailableDriveSpace (GB)'    = $null
        'Memory (GB)'                 = $null
        'UserProfilesSize (MB)'       = $null
        'StoppedServices'             = $null
    }
    $output.ServerName = $server
    $output.'UserProfilesSize (MB)' = (Get-ChildItem -Path "\\$server\c$\
    Users\" -File | Measure-Object -Property Length -Sum).Sum / 1MB
    $output.'AvailableDriveSpace (GB)' = [Math]::Round(((Get-CimInstance
    -ComputerName $server -ClassName Win32_LogicalDisk).FreeSpace / 1GB),1)
    $output.'OperatingSystem' = (Get-CimInstance -ComputerName $server
    -ClassName Win32_OperatingSystem).Caption
    [pscustomobject]$output
}
```

*示例 14-5：更新后的脚本，包含`OperatingSystem`查询*

现在运行你的脚本：

```
PS> C:\Get-ServerInformation.ps1 | Format-Table -AutoSize

ServerName UserProfilesSize (MB) AvailableDriveSpace (GB) OperatingSystem                           StoppedServices IPAddress Memory (GB)
---------- --------------------- ------------------------ ---------------                           --------------- --------- -----------
SQLSRV1                   636245         31.8005790710449 Microsoft Windows Server 2016 Standard
WEBSRV1                   603942         34.5973815917969 Microsoft Windows Server 2012 R2 Standard
```

你已经获得了一些有用的操作系统信息。让我们迈出下一步，看看如何查询内存信息。

#### 内存

接下来是收集下一条信息（`Memory`），你将使用`Win32_PhysicalMemory`类。再次在单台服务器上测试你的查询，能够获得你需要的信息。在这种情况下，你需要的内存信息存储在`Capacity`中：

```
PS> Get-CimInstance -ComputerName sqlsrv1 -ClassName Win32_PhysicalMemory

Caption              : Physical Memory
Description          : Physical Memory
InstallDate          :
Name                 : Physical Memory
Status               :
CreationClassName    : Win32_PhysicalMemory
Manufacturer         : Microsoft Corporation
Model                :
OtherIdentifyingInfo :
--snip--
Capacity             : 2147483648
--snip--
```

`Win32_PhysicalMemory`下的每个实例代表一个*内存条*。你可以把内存条看作服务器中的一根物理内存条。恰好我的 SQLSRV1 服务器只有一根内存条。然而，你肯定会找到有更多内存条的服务器。

由于你需要查询服务器的总内存，你必须按照获取配置文件大小时使用的相同步骤进行操作。你需要将所有实例中的`Capacity`值加起来。幸运的是，`Measure-Object` cmdlet 可以跨任何数量的对象类型工作。只要属性是数字，它就能将它们加起来。

同样，由于`Capacity`是以字节表示的，你需要将其转换为适当的标签：

```
PS> (Get-CimInstance -ComputerName sqlsrv1 -ClassName Win32_PhysicalMemory |
Measure-Object -Property Capacity -Sum).Sum /1GB
2
```

正如你在示例 14-6 中看到的，你的脚本越来越长！

```
Get-ServerInformation.ps1
-------------------
$serversOuPath = 'OU=Servers,DC=powerlab,DC=local'
$servers = Get-ADComputer -SearchBase $serversOuPath -Filter * | Select-Object
-ExpandProperty Name
foreach ($server in $servers) {
    $output = @{
        'ServerName'                  = $null
        'IPAddress'                   = $null
        'OperatingSystem'             = $null
        'AvailableDriveSpace (GB)'    = $null
        'Memory (GB)'                 = $null
        'UserProfilesSize (MB)'       = $null
        'StoppedServices'             = $null
    }
 $output.ServerName = $server
    $output.'UserProfilesSize (MB)' = (Get-ChildItem -Path "\\$server\c$\
    Users\" -File | Measure-Object -Property Length -Sum).Sum / 1MB
    $output.'AvailableDriveSpace (GB)' = [Math]::Round(((Get-CimInstance
    -ComputerName $server -ClassName Win32_LogicalDisk).FreeSpace / 1GB),1)
    $output.'OperatingSystem' = (Get-CimInstance -ComputerName $server
    -ClassName Win32_OperatingSystem).Caption
    $output.'Memory (GB)' = (Get-CimInstance -ComputerName $server -ClassName
    Win32_PhysicalMemory | Measure-Object -Property Capacity -Sum).Sum /1GB
    [pscustomobject]$output
}
```

*示例 14-6：包含`Memory`查询的脚本*

让我们看看到目前为止的输出：

```
PS> C:\Get-ServerInformation.ps1 | Format-Table -AutoSize

ServerName UserProfilesSize (MB) AvailableDriveSpace (GB) OperatingSystem                           StoppedServices IPAddress Memory (GB)
---------- --------------------- ------------------------ ---------------                           --------------- --------- -----------
SQLSRV1                   636245                     31.8 Microsoft Windows Server 2016 Standard                                        2
WEBSRV1                   603942                     34.6 Microsoft Windows Server 2012 R2 Standard                                     2
```

到此为止，你只剩下两个字段需要填写！

#### 网络信息

最后一项 WMI 信息是 IP 地址，它来自`Win32_NetworkAdapterConfiguration`。我把找 IP 地址的任务放到最后，因为与其他数据条目不同，找到服务器的 IP 地址不像找到一个值然后将其添加到`$output`哈希表那样简单。你需要做一些筛选操作来缩小范围。

让我们首先看看使用到目前为止的方法输出是什么样的：

```
PS> Get-CimInstance -ComputerName SQLSRV1 -ClassName Win32_NetworkAdapterConfiguration

ServiceName    DHCPEnabled    Index    Description   PSComputerName
-----------    -----------    -----    -----------   --------------
kdnic          True           0        Microsoft...  SQLSRV1
netvsc         False          1        Microsoft...  SQLSRV1
tunnel         False          2        Microsoft...  SQLSRV1
```

你会立刻发现默认输出不显示 IP 地址，不过这并没有阻止你。但是，更棘手的是，这个命令并没有返回一个实例。这个服务器上有三个网络适配器。你如何选择包含你要查找的 IP 地址的那个呢？

首先，你需要通过使用 `Select-Object` 查看所有属性。使用 `Get-CimInstance -ComputerName SQLSRV1 -ClassName Win32_NetworkAdapterConfiguration | Select-Object -Property *`，你可以浏览（大量的）输出。根据服务器上安装的网络适配器，你可能会注意到某些字段在 `IPAddress` 属性上没有任何内容。这是很常见的，因为某些网络适配器没有 IP 地址。然而，当你找到绑定有 IP 地址的适配器时，它应该类似于以下代码，你可以看到 `IPAddress` 属性 ❶ 在这个例子中有一个 IPv4 地址 192.168.0.40 和几个 IPv6 地址：

```
   DHCPLeaseExpires             :
   Index                        : 1
   Description                  : Microsoft Hyper-V Network Adapter
   DHCPEnabled                  : False
   DHCPLeaseObtained            :
   DHCPServer                   :
   DNSDomain                    : Powerlab.local
   DNSDomainSuffixSearchOrder   : {Powerlab.local}
   DNSEnabledForWINSResolution  : False
   DNSHostName                  : SQLSRV1
   DNSServerSearchOrder         : {192.168.0.100}
   DomainDNSRegistrationEnabled : True
   FullDNSRegistrationEnabled   : True
❶ IPAddress                     : {192.168.0.40... 
   IPConnectionMetric           : 20
   IPEnabled                    : True
   IPFilterSecurityEnabled      : False
   --snip--
```

这个脚本需要动态并支持多种网络适配器配置。确保脚本能够处理除你正在使用的 `Microsoft Hyper-V Network Adapter` 之外的其他类型的网络适配器非常重要。你需要找到一个标准的过滤标准，这样它就能适用于所有服务器。

`IPEnabled` 属性是关键。当这个属性设置为 `True` 时，TCP/IP 协议已绑定到这个网络适配器，这是拥有 IP 地址的前提。如果你能缩小到那个 `IPEnabled` 属性设置为 `True` 的网卡，那么你就找到了你需要的适配器。

在过滤 WMI 实例时，最好使用 `Get-CimInstance` 上的 `Filter` 参数。PowerShell 社区有句名言：*filter left*。基本意思是，如果可以的话，总是尽可能早地过滤输出——也就是说，尽早过滤，这样你就不会把不必要的对象送入管道。除非必须，否则不要使用 `Where-Object`。如果管道中没有不需要的对象，性能会更快。

`Get-CimInstance` 上的 `Filter` 参数使用的是 *Windows 查询语言（WQL）*，这是 *结构化查询语言（SQL）* 的一个子集。`Filter` 参数接受与 WQL 相同的 `WHERE` 子句语法。举个例子：如果在 WQL 中，你希望所有 `Win32_NetworkAdapterConfiguration` 类实例的 `IPEnabled` 属性设置为 `True`，你可以使用 `SELECT *` `FROM Win32_NetworkAdapterConfiguration WHERE IPEnabled = 'True'`。由于你已经在 `Get-CimInstance` 中为 `ClassName` 参数指定了类名，你需要为 `Filter` 指定 `IPEnabled = 'True'`：

```
Get-CimInstance -ComputerName SQLSRV1 -ClassName Win32_NetworkAdapterConfiguration
-Filter "IPEnabled = 'True'" | Select-Object -Property *
```

这应该只返回那些 `IPEnabled`（意味着它们有 IP 地址）的网络适配器。

现在你已经有了一个单一的 WMI 实例，并且知道你要找的属性是 `IPAddress`，我们来看看在查询单个服务器时它是什么样子的。你将使用你一直在使用的 object.property 语法：

```
PS> (Get-CimInstance -ComputerName SQLSRV1 -ClassName Win32_NetworkAdapterConfiguration
-Filter "IPEnabled = 'True'").IPAddress

192.168.0.40
fe80::e4e1:c511:e38b:4f05
2607:fcc8:acd9:1f00:e4e1:c511:e38b:4f05
```

哎呀！看起来里面有 IPv4 和 IPv6 的引用。你需要过滤更多的元素。由于 WQL 无法对属性值进行更深层次的过滤，你需要解析出 IPv4 地址。

经过一些调查，你可以看到所有地址都被花括号包围，并且由逗号分隔：

```
IPAddress : {192.168.0.40, fe80::e4e1:c511:e38b:4f05, 2607:fcc8:acd9:1f00:e4e1:c511:e38b:4f05}
```

这表明该属性不是作为一个大字符串存储，而是作为一个数组。为了确认它是一个数组，你可以尝试使用索引来查看是否能只获取 IPv4 地址：

```
PS> (Get-CimInstance -ComputerName SQLSRV1 -ClassName Win32_NetworkAdapterConfiguration
-Filter "IPEnabled = 'True'").IPAddress[0]

192.168.0.40
```

你真幸运！`IPAddress`属性*确实*是一个数组。此时，你已经得到了值，可以将完整的命令添加到脚本中，如列表 14-7 所示。

```
Get-ServerInformation.ps1
-------------------
$serversOuPath = 'OU=Servers,DC=powerlab,DC=local'
$servers = Get-ADComputer -SearchBase $serversOuPath -Filter * |
Select-Object -ExpandProperty Name
foreach ($server in $servers) {
    $output = @{
        'ServerName'                  = $null
        'IPAddress'                   = $null
 'OperatingSystem'             = $null
        'AvailableDriveSpace (GB)'    = $null
        'Memory (GB)'                 = $null
        'UserProfilesSize (MB)'       = $null
        'StoppedServices'             = $null
    }
    $output.ServerName = $server
    $output.'UserProfilesSize (MB)' = (Get-ChildItem -Path "\\$server\c$\
    Users\" -File | Measure-Object -Property Length -Sum).Sum / 1MB
    $output.'AvailableDriveSpace (GB)' = [Math]::Round(((Get-CimInstance
    -ComputerName $server -ClassName Win32_LogicalDisk).FreeSpace / 1GB),1)
    $output.'OperatingSystem' = (Get-CimInstance -ComputerName $server
    -ClassName Win32_OperatingSystem).Caption
    $output.'Memory (GB)' = (Get-CimInstance -ComputerName $server -ClassName
    Win32_PhysicalMemory | Measure-Object -Property Capacity -Sum).Sum /1GB
    $output.'IPAddress' = (Get-CimInstance -ComputerName $server -ClassName
    Win32_NetworkAdapterConfiguration -Filter "IPEnabled = 'True'").IPAddress[0]
    [pscustomobject]$output
}
```

*列表 14-7：更新后的代码，现在可以处理`IPAddress`*

现在你运行这个：

```
PS> C:\Get-ServerInformation.ps1 | Format-Table -AutoSize

ServerName UserProfilesSize (MB) AvailableDriveSpace (GB) OperatingSystem                          StoppedServices IPAddress     Memory (GB)
---------- --------------------- ------------------------ ---------------                          --------------- ---------     -----------
SQLSRV1                   636245                     31.8 Microsoft Windows Server 2016 Standard                   192.168.0.40  2
WEBSRV1                   603942                     34.6 Microsoft Windows Server 2012 R2 Standard                192.168.0.70  2
```

现在你已经收集到所有所需的 WMI 信息，只剩下最后一件事了。

### Windows 服务

收集的最后一项数据是服务器上已停止的服务列表。你将按照我们的基本算法，首先在单台服务器上进行测试。为此，你将使用`Get-Service`命令在服务器上运行，这将返回所有正在使用的服务。然后，你将把输出通过管道传递给`Where-Object`命令，仅筛选出状态为`Stopped`的服务。总的来说，命令将如下所示：`Get-Service -ComputerName sqlsrv1 | Where-Object { $_.Status -eq 'Stopped' }`。

这个命令返回的是包含所有属性的完整对象。但你只是想要服务名称，所以你将使用你一直在使用的技巧——引用属性名称——并只返回服务名称的列表。

```
PS> (Get-Service -ComputerName sqlsrv1 | Where-Object { $_.Status -eq 'Stopped' }).DisplayName
Application Identity
Application Management
AppX Deployment Service (AppXSVC)
--snip--
```

将这一部分添加到你的脚本中，你将得到列表 14-8。

```
Get-ServerInformation.ps1
-------------------
$serversOuPath = 'OU=Servers,DC=powerlab,DC=local'
$servers = Get-ADComputer -SearchBase $serversOuPath -Filter * |
Select-Object -ExpandProperty Name
foreach ($server in $servers) {
    $output = @{
        'ServerName'                  = $null
        'IPAddress'                   = $null
        'OperatingSystem'             = $null
        'AvailableDriveSpace (GB)'    = $null
        'Memory (GB)'                 = $null
        'UserProfilesSize (MB)'       = $null
        'StoppedServices'             = $null
    }
    $output.ServerName = $server
    $output.'UserProfilesSize (MB)' = (Get-ChildItem -Path "\\$server\c$\
    Users\" -File | Measure-Object -Property Length -Sum).Sum / 1MB
    $output.'AvailableDriveSpace (GB)' = [Math]::Round(((Get-CimInstance
    -ComputerName $server -ClassName Win32_LogicalDisk).FreeSpace / 1GB),1)
    $output.'OperatingSystem' = (Get-CimInstance -ComputerName $server
    -ClassName Win32_OperatingSystem).Caption
    $output.'Memory (GB)' = (Get-CimInstance -ComputerName $server -ClassName
    Win32_PhysicalMemory | Measure-Object -Property Capacity -Sum).Sum /1GB
    $output.'IPAddress' = (Get-CimInstance -ComputerName $server -ClassName
    Win32_NetworkAdapterConfiguration -Filter "IPEnabled = 'True'").IPAddress[0]
    $output.StoppedServices = (Get-Service -ComputerName $server |
    Where-Object { $_.Status -eq 'Stopped' }).DisplayName
    [pscustomobject]$output
}
```

*列表 14-8：更新并使用你的脚本打印已停止的服务*

运行以下代码来测试你的脚本：

```
PS> C:\Get-ServerInformation.ps1 | Format-Table -AutoSize

ServerName UserProfilesSize (MB) AvailableDriveSpace (GB) OperatingSystem                           StoppedServices
---------- --------------------- ------------------------ ---------------                           ---------------
SQLSRV1                   636245                     31.8 Microsoft Windows Server 2016 Standard    {Application Identity,
                                                                                                    Application Management,
                                                                                                    AppX Deployment Servi...
WEBSRV1                   603942                     34.6 Microsoft Windows Server 2012 R2 Standard {Application Experience,
                                                                                                    Application Management,
                                                                                                    Background Intellig...
```

关于已停止的服务，一切看起来都正常——但是其他的属性去哪儿了？此时，控制台窗口已经没有空间了。移除`Format-Table`引用可以让你看到所有的值：

```
PS> C:\Get-ServerInformation.ps1 

ServerName               : SQLSRV1
UserProfilesSize (MB)    : 636245
AvailableDriveSpace (GB) : 31.8
OperatingSystem          : Microsoft Windows Server 2016 Standard
StoppedServices          : {Application Identity, Applic... 
IPAddress                : 192.168.0.40
Memory (GB)              : 2

ServerName               : WEBSRV1
UserProfilesSize (MB)    : 603942
AvailableDriveSpace (GB) : 34.6
OperatingSystem          : Microsoft Windows Server 2012 R2 Standard
StoppedServices          : {Application Experience, Application Management, 
                           Background Intelligent Transfer Service, Computer 
                           Browser...}
IPAddress                : 192.168.0.70
Memory (GB)              : 2
```

看起来不错！

### 脚本清理与优化

在宣布胜利并继续之前，让我们稍微反思一下。编写代码是一个迭代过程。你完全有可能一开始设定了目标，达成了目标，但最终还是写出了糟糕的代码——优秀的程序不仅仅是完成需要做的事。脚本现在确实完成了你想要的功能，但你可以用更好的方式来实现。如何做呢？

回想一下 DRY 原则：*不要重复自己*。你可以看到这个脚本中有很多重复的地方。你有许多`Get-CimInstance`的引用，里面使用了相同的参数。你还在为同一个服务器多次调用 WMI。这些地方看起来是让代码更高效的好机会。

首先，CIM cmdlets 有一个`CimSession`参数。这个参数允许你创建一个 CIM 会话并在之后重用它。与其创建一个临时会话，使用它，然后销毁它，不如创建一个会话，随时使用，然后销毁它，正如 Listing 14-9 中所示。这个概念类似于我们在第八章中介绍的`Invoke-Command`命令的`Session`参数。

```
Get-ServerInformation.ps1
-------------------
$serversOuPath = 'OU=Servers,DC=powerlab,DC=local'
$servers = Get-ADComputer -SearchBase $serversOuPath -Filter * |
Select-Object -ExpandProperty Name
foreach ($server in $servers) {
 $output = @{
        'ServerName'                  = $null
        'IPAddress'                   = $null
        'OperatingSystem'             = $null
        'AvailableDriveSpace (GB)'    = $null
        'Memory (GB)'                 = $null
        'UserProfilesSize (MB)'       = $null
        'StoppedServices'             = $null
    }
    $cimSession = New-CimSession -ComputerName $server
    $output.ServerName = $server
    $output.'UserProfilesSize (MB)' = (Get-ChildItem -Path "\\$server\c$\
    Users\" -File | Measure-Object -Property Length -Sum).Sum
    $output.'AvailableDriveSpace (GB)' = [Math]::Round(((Get-CimInstance
    -CimSession $cimSession -ClassName Win32_LogicalDisk).FreeSpace / 1GB),1)
    $output.'OperatingSystem' = (Get-CimInstance -CimSession $cimSession
    -ClassName Win32_OperatingSystem).Caption
    $output.'Memory (GB)' = (Get-CimInstance -CimSession $cimSession
    -ClassName Win32_PhysicalMemory | Measure-Object -Property Capacity -Sum)
    .Sum /1GB
    $output.'IPAddress' = (Get-CimInstance -CimSession $cimSession -ClassName
    Win32_NetworkAdapterConfiguration -Filter "IPEnabled = 'True'").IPAddress[0]
    $output.StoppedServices = (Get-Service -ComputerName $server |
    Where-Object { $_.Status -eq 'Stopped' }).DisplayName
    Remove-CimSession -CimSession $cimSession
    [pscustomobject]$output
}
```

*Listing 14-9: 更新你的代码以创建并重用单个会话*

现在你正在重用单个 CIM 会话，而不是多个会话。但你仍然在不同命令的参数中多次引用它。为了更好地优化，你可以创建一个哈希表，并为其分配一个名为`CIMSession`的键，将你刚创建的 CIM 会话作为值。一旦你在哈希表中保存了一个通用的参数集，就可以在所有`Get-CimInstance`引用中重用它。

这种技巧被称为*splatting*，你可以通过在调用每个`Get-CimInstance`引用时，使用`@`符号后跟哈希表名称来实现，如 Listing 14-10 所示。

```
Get-ServerInformation.ps1
-------------------
$serversOuPath = 'OU=Servers,DC=powerlab,DC=local'
$servers = Get-ADComputer -SearchBase $serversOuPath -Filter * |
Select-Object -ExpandProperty Name
foreach ($server in $servers) {
    $output = @{
        'ServerName'                  = $null
        'IPAddress'                   = $null
        'OperatingSystem'             = $null
        'AvailableDriveSpace (GB)'    = $null
        'Memory (GB)'                 = $null
        'UserProfilesSize (MB)'       = $null
        'StoppedServices'             = $null
 }
    $getCimInstParams = @{
        CimSession = New-CimSession -ComputerName $server
    }
    $output.ServerName = $server
    $output.'UserProfilesSize (MB)' = (Get-ChildItem -Path "\\$server\c$\
    Users\" -File | Measure-Object -Property Length -Sum).Sum
    $output.'AvailableDriveSpace (GB)' = [Math]::Round(((Get-CimInstance
    @getCimInstParams -ClassName Win32_LogicalDisk).FreeSpace / 1GB),1)
    $output.'OperatingSystem' = (Get-CimInstance @getCimInstParams -ClassName
    Win32_OperatingSystem).Caption
    $output.'Memory (GB)' = (Get-CimInstance @getCimInstParams -ClassName
    Win32_PhysicalMemory | Measure-Object -Property Capacity -Sum).Sum /1GB
    $output.'IPAddress' = (Get-CimInstance @getCimInstParams -ClassName
    Win32_NetworkAdapterConfiguration -Filter "IPEnabled = 'True'").IPAddress[0]
    $output.StoppedServices = (Get-Service -ComputerName $server |
    Where-Object { $_.Status -eq 'Stopped' }).DisplayName
    Remove-CimSession -CimSession $getCimInstParams.CimSession
    [pscustomobject]$output
}
```

*Listing 14-10: 创建`CIMSession`参数以供重用*

到此为止，你可能已经习惯于以`dash<`参数名称`>` `<`参数值`>`的格式将参数传递给命令。这种方式有效，但如果你反复将相同的参数传递给命令，它会变得低效。相反，你可以像这里一样使用 splatting，通过创建一个哈希表，然后将该哈希表传递给每个需要相同参数的命令。

现在你已经完全删除了`$cimSession`变量。

### 总结

在本章中，你从所有前面的章节中提取了关键信息，并将其应用到你在现实世界中可能遇到的情境中。我通常推荐创建的一种脚本类型是查询信息的脚本。它教会你很多 PowerShell 的知识，而且出错的几率很小！

你在本章中进行了逐步迭代，从目标到解决方案，再到更好的解决方案。这是你在使用 PowerShell 时将反复遵循的过程。定义目标，从小处着手，搭建框架（在本例中是`foreach`循环），然后逐步添加代码，一步步克服障碍，直到一切都达成。

一旦你完成了脚本，记住，你实际上并未完全完成，直到你回顾你的代码：看看如何使它更高效、使用更少的资源并提升速度。经验会让优化变得更容易。你将建立起必要的视角，直到优化成为一种自然而然的行为。当你完成优化后，坐下来，享受成功的喜悦，准备好开始下一个项目吧！
