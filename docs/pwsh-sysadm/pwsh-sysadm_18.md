## 第十五章：配置虚拟环境

![图片](img/common.jpg)

*PowerLab* 是一个最终的大型项目，涵盖了你所学的所有概念以及更多内容。它是一个自动化 Hyper-V 虚拟机（VM）配置的项目，包括安装和配置服务，如 SQL 和 IIS。试想一下，只需运行一个命令，如 `New-PowerLabSqlServer`、`New-PowerLabIISServer`，甚至是 `New-PowerLab`，等待几分钟，就能获得一个完全配置好的机器（或多台机器）。如果你跟着我完成本书的剩余部分，这就是你将得到的成果。

PowerLab 项目的目的是消除创建测试环境或实验室时所需的所有重复、耗时的任务。完成后，你只需少数几个命令就能从一个 Hyper-V 主机和几个 ISO 文件构建一个完整的 Active Directory 林。

我故意没有在第一部分和第二部分中涵盖 PowerLab 中的*所有*内容。相反，我挑战你注意这些领域并自行想出独特的解决方案。毕竟，在编程中，总是有很多方法可以完成同一任务。如果你遇到困难，请随时通过 Twitter @adbertram 联系我。

通过构建一个如此规模的项目，你不仅可以覆盖数百个 PowerShell 主题，还能看到脚本语言的强大功能，并获得一个显著节省时间的工具。

在本章中，你将通过创建基础的 `PowerLab` 模块来启动 PowerLab。然后，你将添加自动化创建虚拟交换机、虚拟机和虚拟硬盘（VHD）的功能。

### PowerLab 模块先决条件

为了跟上第三部分中所有的代码示例，你需要满足一些先决条件。每一章都会有一个“先决条件”部分，确保你始终知道该期待什么。

本章的项目需要一个配置如下的 Hyper-V 主机：

+   一个网络适配器

+   IP: 10.0.0.5（可选，但为了完全按照示例进行，你需要此 IP）

+   子网掩码：255.255.255.0

+   一个工作组

+   至少 100GB 的可用存储

+   带有完整图形用户界面的 Windows Server 2016

要创建一个 Hyper-V 服务器，你需要在计划使用的 Windows 服务器上安装 Hyper-V 角色。你可以通过下载并运行书中资源中的 Hyper-V *Setup.ps1* 脚本来加快设置过程，网址为 *[`github.com/adbertram/PowerShellForSysadmins/`](https://github.com/adbertram/PowerShellForSysadmins/)*。这将设置 Hyper-V 并创建一些必要的文件夹。

**注意**

*如果你打算逐字跟随，请运行关联章节的 Pester 前提脚本* (Prerequisites.Tests.ps1) *以确认你的 Hyper-V 服务器已按预期设置。这些测试将确认你的实验环境与我的设置完全一致。运行* Invoke-Pester*，并传递前提脚本，像 列表 15-1 中那样。书中的其余代码将直接在 Hyper-V 主机上执行。*

```
PS> Invoke-Pester -Path 'C:\PowerShellForSysadmins\Part III\Automating Hyper-V\Prerequisites
.Tests.ps1'

Describing Automating Hyper-V Chapter Prerequisites
 [+] Hyper-V host server should have the Hyper-V Windows feature installed 2.23s
 [+] Hyper-V host server is Windows Server 2016 147ms
 [+] Hyper-V host server should have at least 100GB of available storage 96ms
 [+] has a PowerLab folder at the root of C 130ms
 [+] has a PowerLab\VMs folder at the root of C 41ms
 [+] has a PowerLab\VHDs folder at the root of C 47ms
Tests completed in 2.69s
Passed: 5 Failed: 0 Skipped: 0 Pending: 0 Inconclusive: 0
```

*列表 15-1：运行 Pester 前提检查以确保 Hyper-V 工作正常*

如果你成功设置了环境，输出应该会确认五个测试通过。确认环境已准备好后，你可以开始项目！

### 创建模块

因为你知道自己将需要自动化多个彼此相关的任务，所以你应该创建一个 PowerShell 模块。正如你在 第七章 中所学，PowerShell 模块是将多个相似功能合并为一个单元的好方法；这样，你可以轻松管理执行特定任务所需的所有代码。PowerLab 也不例外。没有必要一次性考虑所有内容，所以从小处着手——添加功能，测试，并重复。

#### 创建空模块

首先，你需要创建一个空模块。为此，请远程桌面连接到即将成为 Hyper-V 主机的计算机，并以本地管理员身份登录——或以任何本地管理员组的帐户登录。你将直接在 Hyper-V 主机上构建这个模块，以便简化虚拟机的创建和管理。这意味着你将使用 RDP 会话连接到 Hyper-V 主机的控制台会话。然后，你将创建模块文件夹、模块本身（*.psm1* 文件）和可选的清单（*.psd1* 文件）。

由于你是通过本地管理员帐户登录，并且将来可能允许其他人使用你的 PowerLab 模块，建议将模块创建在 *C:\ProgramFiles\WindowsPowerShell\Modules* 目录下。这样，无论何时作为任何管理员用户登录主机，你都可以访问该模块。

接下来，打开 PowerShell 控制台并选择 **以管理员身份运行**。然后，使用以下命令创建 PowerLab 模块文件夹：

```
PS> New-Item -Path C:\Program Files\WindowsPowerShell\Modules\PowerLab -ItemType Directory
```

接下来，创建一个名为 *PowerLab.psm1* 的空文本文件。使用 `New-Item` 命令：

```
PS> New-Item -Path 'C:\Program Files\WindowsPowerShell\Modules\PowerLab\PowerLab.psm1'
```

#### 创建模块清单

现在，创建一个模块清单。要创建模块清单，使用便捷的 `New-ModuleManifest` 命令。此命令创建一个模板清单，你可以在初始文件构建后用文本编辑器打开并根据需要进行调整。以下是我用来构建模板清单的参数：

```
PS> New-ModuleManifest -Path 'C:\Program Files\WindowsPowerShell\Modules\PowerLab\PowerLab.psd1' 
-Author 'Adam Bertram' 
-CompanyName 'Adam the Automator, LLC' 
-RootModule 'PowerLab.psm1' 
-Description 'This module automates all tasks to provision entire environments of a domain
controller, SQL server and IIS web server from scratch.'
```

随意修改参数值以满足你的需求。

#### 使用内置前缀命名函数

函数不一定需要特定的名称。然而，当你构建一个通常由相关函数组成的模块时，最好在函数名的名词部分前加上相同的标签。例如，你的项目名为*PowerLab*。在这个项目中，你将构建与该共同主题相关的函数。为了将 PowerLab 中的函数与其他模块中的函数区分开来，你可以在函数名的实际名词部分前加上模块名。这意味着大多数函数的名词将以*PowerLab*为开头。

然而，并不是所有的函数都将以模块名开头。例如，一些仅协助其他函数且永远不会被用户调用的辅助函数。

当你确定要让所有函数名的名词都使用相同的前缀，而不必在函数名中明确指定时，模块清单中有一个选项叫做`DefaultCommandPrefix`。这个选项将强制 PowerShell 在名词前加上特定的字符串。例如，如果你在清单中定义了`DefaultCommandPrefix`键，并在模块中创建了一个名为`New-Switch`的函数，那么当模块被导入时，这个函数将无法作为`New-Switch`使用，而是作为`New-PowerLabSwitch`：

```
# Default prefix for commands exported from this modul...
# DefaultCommandPrefix = ''
```

我倾向于*不*采用这种方式，因为它会强制在模块中的*所有*函数名的名词部分前加上这个字符串。

#### 导入新模块

现在你已经构建了清单，接下来可以检查它是否成功导入。由于你还没有编写任何函数，模块不会执行任何操作，但重要的是检查 PowerShell 是否能够识别该模块。如果你看到以下结果，那么你就可以继续了。

```
PS> Get-Module -Name PowerLab –ListAvailable

    Directory: C:\Program Files\WindowsPowerShell\Modules

ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Script     1.0        PowerLab
```

如果 PowerLab 模块没有出现在输出的底部，请返回前面的步骤检查。此外，确保在 *C:\Program Files\WindowsPowerShell\Modules* 下已创建 PowerLab 文件夹，并且其中包含 *PowerLab.psm1* 和 *PowerLab.psd1* 文件。

### 自动化虚拟环境的配置

现在你已经构建了模块的结构，可以开始向其中添加功能了。由于创建一个服务器（如 SQL 或 IIS）的任务包含多个相互依赖的步骤，你将首先自动化虚拟交换机、虚拟机和虚拟磁盘的创建。接着你将自动化操作系统部署到这些虚拟机上，最后在这些虚拟机上安装 SQL Server 和 IIS。

#### 虚拟交换机

在你开始自动化创建虚拟机之前，需要确保 Hyper-V 主机上已设置虚拟交换机。*虚拟交换机*使虚拟机能够与客户端计算机及在同一主机上创建的其他虚拟机进行通信。

##### 手动创建虚拟交换机

你的虚拟交换机将是一个*外部*交换机，名为`PowerLab`。这个名字的交换机可能在 Hyper-V 主机上并不存在，但为了确保无误，列出主机上的所有虚拟交换机。你永远不会后悔先检查一遍。

要查看在 Hyper-V 主机上设置的所有交换机，使用 Get-VmSwitch 命令。确认 PowerLab 交换机不存在后，使用 `New-VmSwitch` 命令创建一个新的虚拟交换机，指定名称（`PowerLab`）和交换机类型：

```
PS> New-VMSwitch -Name PowerLab -SwitchType External
```

由于你需要让虚拟机能够与 Hyper-V 外部的主机通信，因此你将值 `External` 传递给 `SwitchType` 参数。无论你与谁分享这个项目，他们也需要创建一个外部交换机。

交换机创建完成后，现在是时候创建 PowerLab 模块的第一个函数了。

##### 自动化虚拟机交换机创建

第一个 PowerLab 功能，称为 `New-PowerLabSwitch`，用于创建 Hyper-V 交换机。这个功能并不复杂。事实上，如果没有它，你只需要在命令行中执行一个简单的命令——也就是 `New-VmSwitch`。但是，如果你将这个 Hyper-V 命令包装成一个自定义函数，你将能够执行其他工作：例如，为交换机添加任何类型的默认配置。

我是 *幂等性* 的忠实粉丝，这个词的意思是“无论命令执行的状态如何，它每次都会执行相同的任务。”在这个例子中，如果创建交换机的任务不是幂等的，那么如果交换机已存在，运行 `New-VmSwitch` 就会导致错误。

为了去除手动检查交换机是否创建的要求，你可以使用 `Get-VmSwitch` 命令。这个命令会检查交换机是否已创建。然后，只有当交换机不存在时，你才会尝试创建新的交换机。这使得你可以在任何环境中运行 `New-PowerLabSwitch`，并且知道它将始终创建虚拟交换机，而不会返回错误——无论 Hyper-V 主机的状态如何。

打开 *C:\Program Files\WindowsPowerShell\Modules\PowerLab\PowerLab.psm1* 文件并创建 `New-PowerLabSwitch` 函数，如 Listing 15-2 所示。

```
function New-PowerLabSwitch {
    param(
        [Parameter()]
        [string]$SwitchName = 'PowerLab',

        [Parameter()]
        [string]$SwitchType = 'External'
    )

    if (-not (Get-VmSwitch -Name $SwitchName -SwitchType $SwitchType -ErrorAction
    SilentlyContinue)) { ❶
        $null = New-VMSwitch -Name $SwitchName -SwitchType $SwitchType ❷
    } else {
        Write-Verbose -Message "The switch [$($SwitchName)] has already been created." ❸
    }
}
```

*Listing 15-2: `New-PowerLabSwitch` 函数在 `PowerLab` 模块中的实现*

该函数首先检查交换机是否已经创建 ❶。如果没有，函数会创建它 ❷。如果交换机已经创建，函数会向控制台返回一条详细信息 ❸。

保存模块，然后通过使用 Import-Module -Name PowerLab -Force 命令强制重新导入。

当你向模块添加新功能时，必须重新导入模块。如果模块已经导入，你必须使用 `Force` 参数与 `Import-Module` 一起强制 PowerShell 重新导入它。否则，PowerShell 会看到模块已经被导入，并跳过它。

一旦你重新导入该模块，`New-PowerLabSwitch` 函数应该就可以使用了。运行以下命令：

```
PS> New-PowerLabSwitch –Verbose
VERBOSE: The switch [PowerLab] has already been created.
```

注意到你没有收到错误信息，而是收到了一个有用的详细信息，说明开关已经创建。这是因为你将可选的`Verbose`参数传递给了函数。由于`SwitchName`和`SwitchType`参数的默认值通常相同，所以这两个参数选择了默认值。

#### 创建虚拟机

现在你已经设置了虚拟交换机，接下来是创建虚拟机。对于这个演示，你将创建一个二代虚拟机，命名为 LABDC，分配 2GB 内存，并连接到你刚刚在 Hyper-V 主机的*C:\PowerLab\VMs*文件夹中创建的虚拟交换机。我选择*LABDC*作为名称，因为这将最终成为我们的 Active Directory 域控制器。这个虚拟机最终将成为你完全构建的实验室中的域控制器。

首先，查看所有现有的虚拟机，确保没有同名的虚拟机已经存在。因为你已经知道要创建的虚拟机的名称，所以将该值传递给`Get-Vm`命令的`Name`参数：

```
PS> Get-Vm -Name LABDC
Get-Vm : A parameter is invalid. Hyper-V was unable to find a virtual machine with name LABDC.
At line:1 char:1
+ Get-Vm -Name LABDC
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidArgument: (LABDC:String) [Get-VM],
                              VirtualizationInvalidArgumentException
    + FullyQualifiedErrorId : InvalidParameter,Microsoft.HyperV.PowerShell.Commands.GetVMCommand
```

当`Get-Vm`命令找不到指定名称的虚拟机时，它会返回一个错误。由于你只是检查虚拟机是否存在，且此时我们并不关心它是否存在，因此可以使用`ErrorAction`参数并设置为`SilentlyContinue`，以确保命令在虚拟机不存在时返回空值。为了简化，这里没有使用`try/catch`。

此技术仅在命令返回的是非终止错误时有效。如果命令返回终止错误，你将需要查看是否能返回所有对象并使用`Where-Object`进行过滤，或将命令包含在`try/catch`块中。

##### 手动创建虚拟机

该虚拟机不存在，这意味着你需要创建它。要创建虚拟机，你需要运行`Get-Vm`命令，并传递在本节开始时定义的值。

```
PS> New-VM -Name 'LABDC' -Path 'C:\PowerLab\VMs' 
-MemoryStartupBytes 2GB -Switch 'PowerLab' -Generation 2

Name   State CPUUsage(%) MemoryAssigned(M) Uptime   Status             Version
----   ----- ----------- ----------------- ------   ------             -------
LABDC  Off   0           0                 00:00:00 Operating normally 8.0
```

现在你应该已经有了一台虚拟机，但请通过再次运行 Get-Vm 来确认这一点。

##### 自动化虚拟机创建

要自动化创建一个简单的虚拟机，你需要再添加一个函数。这个函数将遵循与创建新虚拟交换机时相同的模式：编写一个幂等函数，无论 Hyper-V 主机的状态如何，都能执行任务。

将`New-PowerLabVm`函数，如清单 15-3 所示，输入到你的*PowerLab.psm1*模块中。

```
function New-PowerLabVm {
    param(
        [Parameter(Mandatory)]
        [string]$Name,

        [Parameter()]
        [string]$Path = 'C:\PowerLab\VMs',

        [Parameter()]
        [string]$Memory = 4GB,

        [Parameter()]
        [string]$Switch = 'PowerLab',

        [Parameter()]
        [ValidateRange(1, 2)]
        [int]$Generation = 2
    )

 ❶ if (-not (Get-Vm -Name $Name -ErrorAction SilentlyContinue)) {
     ❷ $null = New-VM -Name $Name -Path $Path -MemoryStartupBytes $Memory
        -Switch $Switch -Generation $Generation
    } else {
     ❸ Write-Verbose -Message "The VM [$($Name)] has already been created."
    }
}
```

*清单 15-3：`New-PowerLabVm`函数，位于`PowerLab`模块中*

该函数检查虚拟机是否已经存在❶。如果不存在，函数将创建一个虚拟机❷。如果已存在，函数将向控制台显示一条详细信息❸。

保存*PowerLab.psm1*并在命令提示符下执行你刚创建的新函数：

```
PS> New-PowerLabVm -Name 'LABDC' –Verbose
VERBOSE: The VM [LABDC] has already been created.
```

再次运行此命令时，你可以使用指定的参数值创建一个虚拟机——无论该虚拟机是否已经存在（在你强制模块重新导入之后）。

#### 虚拟硬盘

你现在已经将虚拟机附加到交换机，但没有存储的虚拟机是没有用的。为了解决这个问题，你需要创建一个本地虚拟硬盘（VHD）并将其连接到虚拟机。

**注意**

*在第十六章中，你将使用一个社区脚本将 ISO 文件转换为 VHD。因此，你无需创建 VHD。但如果你不打算自动化操作系统部署，或者你需要将 VHD 创建作为其他脚本的一部分自动化，我仍然建议你完成这一部分。*

##### 手动创建 VHD

要创建 VHD 文件，你只需要一个命令：`New-Vhd`。在本节中，你将创建一个可以增长到 50GB 大小的 VHD；为了节省空间，你会将 VHD 设置为动态调整大小。

你首先需要在 Hyper-V 主机的*C:\PowerLab\VHDs*路径下创建一个文件夹来存放 VHD。确保为你的 VHD 命名时使用与你打算附加的虚拟机相同的名称，以保持简洁。

使用`New-Vhd`命令创建 VHD：

```
PS> New-Vhd ❶-Path 'C:\PowerLab\VHDs\MYVM.vhdx' ❷-SizeBytes 50GB ❸–Dynamic

ComputerName            : HYPERVSRV
Path                    : C:\PowerLab\VHDs\MYVM.vhdx
VhdFormat               : VHDX
VhdType                 : Dynamic
FileSize                : 4194304
Size                    : 53687091200
MinimumSize             :
LogicalSectorSize       : 512
PhysicalSectorSize      : 4096
BlockSize               : 33554432
ParentPath              :
DiskIdentifier          : 3FB5153D-055D-463D-89F3-BB733B9E69BC
FragmentationPercentage : 0
Alignment               : 1
Attached                : False
DiskNumber              :
Number                  :
```

你需要传递给`New-Vhd`路径❶和 VHD 大小❷，最后，指定你希望它动态调整大小❸。

使用`Test-Path`命令确认你是否成功在 Hyper-V 主机上创建了 VHD。如果`Test-Path`返回`True`，说明成功：

```
PS> Test-Path -Path 'C:\PowerLab\VHDs\MYVM.vhdx'
True
```

现在你需要将 VHD 附加到之前创建的虚拟机。为此，你需要使用`Add-VMHardDiskDrive`命令。但因为你*不会*将 VHD 附加到 LABDC——操作系统部署自动化将在第十六章中完成这项工作——所以你需要创建一个名为 MYVM 的虚拟机来附加 VHD：

```
PS> New-PowerLabVm -Name 'MYVM'
PS> ❶Get-VM -Name MYVM | Add-VMHardDiskDrive -Path 'C:\PowerLab\VHDs\MYVM.vhdx'
PS> ❷Get-VM -Name MYVM | Get-VMHardDiskDrive

VMName ControllerType ControllerNumber ControllerLocation DiskNumber Path
------ -------------- ---------------- ------------------ ---------- ----
MYVM   SCSI           0                0                             C:\PowerLab\VHDs\MYVM.vhdx
```

`Add-VMHardDiskDrive`命令接受`Get-VM`命令为其管道输入返回的对象类型，因此你可以直接从`Get-VM`将虚拟机传递给`Add-VMHardDiskDrive`——并指定 Hyper-V 主机上 VHD 的路径❶。

紧接着，使用`Get-VMHardDiskDrive`命令确认 VHDX 是否已成功添加❷。

##### 自动化 VHD 创建

你可以向模块中添加另一个函数来自动化创建 VHD 并将其附加到虚拟机的过程。在创建脚本或函数时，考虑各种配置非常重要。

列表 15-4 定义了`New-PowerLabVhd`函数，该函数创建 VHD 并将虚拟机附加到它上面。

```
function New-PowerLabVhd {
    param
    (
        [Parameter(Mandatory)]
        [string]$Name,

        [Parameter()]
        [string]$AttachToVm,

        [Parameter()]
        [ValidateRange(512MB, 1TB)]
        [int64]$Size = 50GB,

 [Parameter()]
        [ValidateSet('Dynamic', 'Fixed')]
        [string]$Sizing = 'Dynamic',

        [Parameter()]
        [string]$Path = 'C:\PowerLab\VHDs'
    )

    $vhdxFileName = "$Name.vhdx"
    $vhdxFilePath = Join-Path -Path $Path -ChildPath "$Name.vhdx"

    ### Ensure we don't try to create a VHD when there's already one there
    if (-not (Test-Path -Path $vhdxFilePath -PathType Leaf)) { ❶
        $params = @{
            SizeBytes = $Size
            Path      = $vhdxFilePath
        }
        if ($Sizing -eq 'Dynamic') { ❷
            $params.Dynamic = $true
        } elseif ($Sizing -eq 'Fixed') {
            $params.Fixed = $true
        }

        New-VHD @params
        Write-Verbose -Message "Created new VHD at path [$($vhdxFilePath)]"
    }

    if ($PSBoundParameters.ContainsKey('AttachToVm')) {
        if (-not ($vm = Get-VM -Name $AttachToVm -ErrorAction SilentlyContinue)) { ❸
            Write-Warning -Message "The VM [$($AttachToVm)] does not exist. Unable to attach VHD."
        } elseif (-not ($vm | Get-VMHardDiskDrive | Where-Object { $_.Path -eq $vhdxFilePath })) { ❹
            $vm | Add-VMHardDiskDrive -Path $vhdxFilePath
            Write-Verbose -Message "Attached VHDX [$($vhdxFilePath)] to VM [$($AttachToVM)]."
        } else { ❺
            Write-Verbose -Message "VHDX [$($vhdxFilePath)] already attached to VM [$($AttachToVM)]."
        }
    }
}
```

*列表 15-4：`New-PowerLabVhd`函数在`PowerLab`模块中的实现*

该函数支持动态和固定大小❷，并且考虑到四种不同的状态：

+   VHD 已经存在❶。

+   要附加 VHD 的虚拟机不存在❸。

+   要附加 VHD 的虚拟机已经存在，但 VHD 尚未连接❹。

+   要附加 VHD 的虚拟机已经存在，并且 VHD 已经附加❺。

函数设计是一个完全不同的领域。要能够创建一个在多种场景下都能正常运行的脚本或函数，需要多年的编码和实践。这是一门艺术，至今尚未完全完善，但如果你能预想到问题可能出现的多种方式，并在一开始就考虑到这些情况，你的函数就会更好。然而，不要*过度*投入，花费几个小时在一个函数或脚本上，确保*每个*细节都被覆盖！这只是代码，你可以在以后进行修改。

##### 执行了 New-PowerLabVhd 函数

你可以在不同的状态下执行这段代码，并考虑每种状态。让我们测试多种状态，确保这个自动化脚本在每种情况下都能正常工作：

```
PS> New-PowerLabVhd -Name MYVM -Verbose -AttachToVm MYVM

VERBOSE: VHDX [C:\PowerLab\VHDs\MYVM.vhdx] already attached to VM [MYVM].

PS> Get-VM -Name MYVM | Get-VMHardDiskDrive | Remove-VMHardDiskDrive
PS> New-PowerLabVhd -Name MYVM -Verbose -AttachToVm MYVM

VERBOSE: Attached VHDX [C:\PowerLab\VHDs\MYVM.vhdx] to VM [MYVM].
PS> New-PowerLabVhd -Name MYVM -Verbose -AttachToVm NOEXIST

WARNING: The VM [NOEXIST] does not exist. Unable to attach VHD.
```

在这里，你并不是以正式的方式进行测试。相反，你通过强制让你的新函数运行你定义的每条代码路径，来测试它的表现。

### 使用 Pester 测试新函数

现在你应该能够自动化创建 Hyper-V 虚拟机了，但你应该始终为你创建的每个功能编写 Pester 测试，以确保一切按预期工作，并且随着时间的推移监控你的自动化。在本书的其余部分，你将为所有工作编写 Pester 测试。你可以在本书的资源中找到这些 Pester 测试，网址是[*https://github.com/adbertram/PowerShellForSysadmins/*](https://github.com/adbertram/PowerShellForSysadmins/)。

在这一章中，你完成了四个任务：

+   创建了一个虚拟交换机

+   创建了一个虚拟机

+   创建了一个 VHDX

+   将 VHDX 附加到虚拟机

我把这一章的 Pester 测试分成了几个部分，每部分对应四个成果。像这样将测试分阶段有助于保持测试的条理性。

让我们运行这个测试，验证你在这一章编写的代码。要运行测试脚本，确保你已经从本书的资源中下载了*Automating-Hyper-V.Tests.ps1*脚本。在以下代码中，测试脚本位于*C:\*的根目录，但你的路径可能不同，具体取决于你下载资源文件的位置。

```
PS> Invoke-Pester 'C:\Automating-Hyper-V.Tests.ps1'
Describing Automating Hyper-V Chapter Demo Work
   Context Virtual Switch
    [+] created a virtual switch called PowerLab 195ms
   Context Virtual Machine
    [+] created a virtual machine called MYVM 62ms
   Context Virtual Hard Disk
    [+] created a VHDX called MYVM at C:\PowerLab\VHDs 231ms
    [+] attached the MYVM VHDX to the MYVM VM 194ms
Tests completed in 683ms
Passed: 4 Failed: 0 Skipped: 0 Pending: 0 Inconclusive: 0
```

所有四个测试都通过了，所以你可以继续进行下一章了。

### 总结

你已经为第一个真正的 PowerShell 自动化项目奠定了基础！希望你已经看到，通过 PowerShell 自动化可以节省多少时间！通过使用微软提供的免费 PowerShell 模块，你能够快速运行几个命令，轻松创建虚拟交换机、虚拟机和磁盘驱动器。微软给你了命令，但最终还是你自己搭建了周围的逻辑，使一切无缝衔接。

现在你可能已经意识到，可以即时编写有效的脚本，但通过提前思考并添加条件逻辑，你的脚本可以应对更多情况。

在下一章中，你将使用刚刚创建的虚拟机，自动化部署操作系统，几乎只需要一个 ISO 文件。
