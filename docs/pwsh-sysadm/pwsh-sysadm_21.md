## 第十八章：创建和配置 SQL 服务器

![图片](img/common.jpg)

到目前为止，你已经创建了一个可以创建虚拟机、附加 VHD、安装 Windows 并创建（和填充）活动目录森林的模块。让我们再增加一项：部署 SQL 服务器。有了一个虚拟机、安装了操作系统并设置了域控制器，你已经完成了大部分的繁重工作！现在你只需利用现有的功能，通过少许调整，就能安装 SQL 服务器。

### 先决条件

在本章中，我假设你已经在第三部分中跟随操作并创建了至少一个名为 LABDC 的虚拟机，该虚拟机正在你的 Hyper-V 主机上运行。这个虚拟机将作为域控制器运行，由于你将通过 PowerShell Direct 再次连接到多个虚拟机，因此你需要将域凭据保存到 Hyper-V 主机（查看第十七章以了解我们是如何做到这一点的）。

你将使用一个名为*ManuallyCreatingASqlServer.ps1*的脚本（可以在本章资源中找到）来解释如何正确地自动化部署 SQL 服务器。这个脚本包含了本章中介绍的所有基本步骤，是你在完成本章过程中一个很好的参考资源。

和往常一样，请运行本章附带的先决条件测试脚本，以确保你满足所有预期的先决条件。

### 创建虚拟机

当你想到*SQL Server*时，你可能会想到数据库、作业和表等内容。但在你能够处理这些内容之前，必须完成大量的后台工作：首先，每个 SQL 数据库都必须存在于服务器上，每个服务器需要一个操作系统，每个操作系统需要一个物理或虚拟机来安装。幸运的是，你在过去的几章中已经设置了创建 SQL 服务器所需的确切环境。

一位优秀的自动化工程师会从分解所有必要的依赖项开始每个项目。他们围绕这些依赖项进行自动化，然后再基于它们进行扩展。这个过程会导致一个模块化、解耦的架构，具有随时相对轻松地进行更改的灵活性。

最终你需要的是一个函数，它使用标准配置启动任意数量的 SQL 服务器。但要实现这一点，你必须分层思考这个项目。第一层是虚拟机。我们先处理这个。

既然你已经在 PowerLab 模块中有一个构建虚拟机的函数，你就可以使用它。因为你构建的所有实验室环境都将是相同的，而且你已经将创建新虚拟机所需的许多参数定义为`New-PowerLabVM`函数的默认参数值，所以你唯一需要传递给这个函数的就是虚拟机的名称：

```
PS> New-PowerLabVm -Name 'SQLSRV'
```

### 安装操作系统

就这样，你有了一个准备好的虚拟机。那还真是简单。我们再做一次。使用你在第十六章中编写的命令在虚拟机上安装 Windows：

```
PS> Install-PowerLabOperatingSystem -VmName 'SQLSRV'
Get-Item : Cannot find path 'C:\Program Files\WindowsPowerShell\Modules\
powerlab\SQLSRV.xml' because it does not exist.
At C:\Program Files\WindowsPowerShell\Modules\powerlab\PowerLab.psm1:138 char:16
+     $answerFile = Get-Item -Path "$PSScriptRoot\$VMName.xml"
+                   ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (C:\Program File...rlab\SQLSRV
                              .xml:String) [Get-Item], ItemNotFoundException
```

哎呀！你使用了 PowerLab 模块中现有的`Install-PowerLabOperatingSystem`函数来安装即将成为 SQL 服务器的操作系统，但它失败了，因为它引用了模块文件夹中的一个名为*SQLSRV.xml*的文件。当你构建这个函数时，你假设模块文件夹中会有一个*.xml*文件。在构建像这样的庞大自动化项目时，路径不一致和文件不存在等问题是常见的。你会有很多依赖项需要处理。解决这些错误的唯一方法就是尽可能多地执行代码，尽可能多地测试不同场景。

### 添加一个 Windows 无人值守应答文件

`Install-PowerLabOperatingSystem`函数假设 PowerLab 模块文件夹中总是会有一个名为*.xml*的文件。这意味着，在部署新服务器之前，你必须先确保将该文件放在正确的位置。幸运的是，现在你已经创建了 LABDC 无人值守应答文件，这应该很容易。你首先需要做的是复制现有的*LABDC.xml*文件，并将其命名为*SQLSRV.xml*：

```
PS> Copy-Item -Path 'C:\Program Files\WindowsPowerShell\Modules\PowerLab\LABDC.xml' -Destination
'C:\Program Files\WindowsPowerShell\Modules\PowerLab\SQLSRV.xml'
```

一旦你复制了，接下来你需要做一些调整：主机名和 IP 地址。由于你没有部署 DHCP 服务器，所以你将使用静态 IP 地址并必须更改它们（否则你只需要更改服务器名称）。

打开 *C:\Program Files\WindowsPowerShell\Modules\SQLSRV.xml*，并查找定义主机名的部分。一旦找到它，修改`ComputerName`值。它应该类似于下面这样：

```
<component name="Microsoft-Windows-Shell-Setup" processorArchitecture="amd64"
publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" 
    xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <ComputerName>SQLSRV</ComputerName>
    <ProductKey>XXXXXXXXXXXXX</ProductKey>
</component>
```

接下来，查找`UnicastIPAddress`节点。它看起来像以下代码。请注意，我使用的是一个 10.0.0.0/24 的网络，并选择让我的 SQL 服务器的 IP 地址为 10.0.0.101：

```
<UnicastIpAddresses>
    <IpAddress wcm:action="add" wcm:keyValue="1">10.0.0.101</IpAddress>
</UnicastIpAddresses>
```

保存*SQLSRV.xml*文件，并再次尝试运行`Install-PowerLabOperatingSystem`命令。此时，你的命令应该能够成功运行，并将 Windows Server 2016 部署到你的 SQLSRV 虚拟机上。

### 将 SQL Server 添加到域

你刚安装了操作系统，现在需要启动虚拟机。使用`Start-VM` cmdlet 很容易做到：

```
PS> Start-VM -Name SQLSRV
```

现在你必须等待虚拟机上线——这可能需要一些时间。需要多久？这取决于很多变量。你可以做的一件事是使用`while`循环不断检查是否能够连接到虚拟机。

让我们看看如何操作。在 Listing 18-1 中，你获取了虚拟机的本地保存凭据。一旦你有了凭据，你可以创建一个`while`循环，持续执行`Invoke-Command`直到返回结果。

注意你正在为`ErrorAction`参数使用`Ignore`值。你必须这么做，因为如果没有它，当`Invoke-Command`无法连接到计算机时，它会返回一个非终止错误信息。为了避免控制台被预期中的错误信息填满（因为你知道可能无法连接，并且对此没问题），你正在忽略这些错误信息。

```
$vmCred = Import-CliXml -Path 'C:\PowerLab\VMCredential.xml'
while (-not (Invoke-Command -VmName SQLSRV -ScriptBlock { 1 } -Credential
$vmCred -ErrorAction Ignore)) {
    Start-Sleep -Seconds 10
    Write-Host 'Waiting for SQLSRV to come up...'
}
```

*清单 18-1：检查服务器是否在线，并忽略错误信息*

一旦虚拟机终于启动，就可以将其添加到你在上一章创建的域中。添加计算机到域的命令是 `Add-Computer`。由于你是在 Hyper-V 主机上运行所有命令，而不依赖于网络连接，因此需要将 `Add-Computer` 命令包裹在脚本块中，并通过 PowerShell Direct 执行它，直接在 SQLSRV 上运行。

请注意，在清单 18-2 中，你需要同时使用虚拟机的本地用户帐户和域帐户。为此，你首先通过 `Invoke-Command` 连接到 SQLSRV 服务器本身。连接后，你会将域凭证传递给域控制器以进行身份验证，这样就可以将计算机帐户添加到域中。

```
$domainCred = Import-CliXml -Path 'C:\PowerLab\DomainCredential.xml'
$addParams = @{
    DomainName = 'powerlab.local'
    Credential = $domainCred
    Restart    = $true
    Force      = $true
}
Invoke-Command -VMName SQLSRV -ScriptBlock { Add-Computer ❶@using:addParams } -Credential $vmCred
```

*清单 18-2：获取凭证并将计算机添加到域*

请注意，你正在使用 `$using` 关键字 ❶。该关键字允许你将本地变量 `$addParams` 传递到 SQLSRV 服务器的远程会话中。

由于你在 `Add-Computer` 中使用了 `Restart` 开关参数，虚拟机将在添加到域后立即重启。同样，由于你还有进一步的工作要做，你需要等待这一过程发生。然而，这一次，你需要等它先关闭 *然后* 再重启（见清单 18-3），因为脚本非常快速，如果你不等待它先关闭，脚本可能会继续运行，因为它检测到服务器已经启动，但实际上它并没有关闭！

```
❶ while (Invoke-Command -VmName SQLSRV -ScriptBlock { 1 } -Credential $vmCred 
   -ErrorAction Ignore) {
    ❷ Start-Sleep -Seconds 10
    ❸ Write-Host 'Waiting for SQLSRV to go down...'
}

❶ while (-not (Invoke-Command -VmName SQLSRV -ScriptBlock { 1 } -Credential 
   $domainCred -ErrorAction Ignore)) {
    ❷ Start-Sleep -Seconds 10
    ❸ Write-Host 'Waiting for SQLSRV to come up...'
}
```

*清单 18-3：等待服务器重启*

首先，你通过在 SQLSRV ❶ 上返回数字 1 来检查 SQLSRV 是否已关闭。如果返回输出，这意味着 PowerShell 远程访问可用，因此 SQLSRV 尚未关闭。如果有输出返回，接下来你需要暂停 10 秒 ❷，在屏幕上写一条消息 ❸，然后再试一次。

当测试 SQLSRV 何时重新启动时，你会采取相反的操作。一旦脚本释放了控制台，SQLSRV 应该已经启动并被添加到你的 Active Directory 域中。

#### 安装 SQL Server

现在你已经创建了一个带有 Windows Server 2016 的虚拟机，你可以在其上安装 SQL Server 2016。这是新代码！直到现在，你一直在利用现有的代码；现在你又在开辟新天地。

通过 PowerShell 安装 SQL Server 包括几个步骤：

1.  复制并调整 SQL Server 答案文件

1.  复制 SQL Server ISO 文件到即将成为 SQL 服务器的虚拟机

1.  挂载即将成为 SQL 服务器的 ISO 文件

1.  运行 SQL Server 安装程序

1.  卸载 ISO 文件

1.  清理 SQL 服务器上的任何临时复制文件

#### 复制文件到 SQL 服务器

根据我们的计划，第一步是将一些文件复制到即将成为 SQL 服务器的计算机上。你需要 SQL Server 安装程序需要的无人参与回答文件，还需要包含 SQL Server 安装内容的 ISO 文件。由于我们假设 Hyper-V 主机与虚拟机之间没有网络连接，因此你将再次使用 PowerShell Direct 来复制这些文件。要使用 PowerShell Direct 复制文件，你首先需要在远程虚拟机上创建一个会话。在下面的代码中，你使用 `Credential` 参数来验证 SQLSRV。如果你的服务器与当前操作的计算机在同一个 Active Directory 域中，那么就不需要 `Credential` 参数。

```
$session = New-PSSession -VMName 'SQLSRV' -Credential $domainCred
```

接下来，复制 PowerLab 模块中找到的模板 *SQLServer.ini* 文件：

```
$sqlServerAnswerFilePath = "C:\Program Files\WindowsPowerShell\Modules\
PowerLab\SqlServer.ini"
$tempFile = Copy-Item -Path $sqlServerAnswerFilePath -Destination "C:\Program
Files\WindowsPowerShell\Modules\PowerLab\temp.ini" -PassThru
```

完成之后，你将修改文件以匹配所需的配置。回想一下，之前当你需要更改某些值时，你手动打开了无人参与的 XML 文件。这比你需要做的更多工作——信不信由你，你也可以自动化这一步！

在清单 18-4 中，你正在读取复制的模板文件内容，查找字符串 `SQLSVCACCOUNT=`, `SQLSVCPASSWORD=`, 和 `SQLSYSADMINACCOUNTS=`，并用特定值替换这些字符串。当你完成后，将修改后的字符串写回复制的模板文件。

```
$configContents = Get-Content -Path $tempFile.FullName -Raw
$configContents = $configContents.Replace('SQLSVCACCOUNT=""', 'SQLSVCACCOUNT="PowerLabUser"')
$configContents = $configContents.Replace('SQLSVCPASSWORD=""', 'SQLSVCPASSWORD="P@$$w0rd12"')
$configContents = $configContents.Replace('SQLSYSADMINACCOUNTS=""', 'SQLSYSADMINACCOUNTS=
"PowerLabUser"')
Set-Content -Path $tempFile.FullName -Value $configContents
```

*清单 18-4：替换字符串*

一旦你有了回答文件，并将该文件和 SQL Server ISO 文件复制到即将成为 SQL 服务器的计算机上，安装程序就准备好了：

```
$copyParams = @{
    Path        = $tempFile.FullName
    Destination = 'C:\'
    ToSession   = $session
}
Copy-Item @copyParams
Remove-Item -Path $tempFile.FullName -ErrorAction Ignore
Copy-Item -Path 'C:\PowerLab\ISOs\en_sql_server_2016_standard_x64_dvd_8701871.iso' 
-Destination 'C:\' -Force -ToSession $session
```

#### 运行 SQL Server 安装程序

现在你终于准备好安装 SQL Server。清单 18-5 包含了安装 SQL Server 的代码：

```
$icmParams = @{
    Session      = $session
    ArgumentList = $tempFile.Name
    ScriptBlock  = {
        $image = Mount-DiskImage -ImagePath 'C:\en_sql_server_2016_standard_x64_dvd_8701871
        .iso' -PassThru ❶
        $installerPath = "$(($image | Get-Volume).DriveLetter):"
        $null = & "$installerPath\setup.exe" "/CONFIGURATIONFILE=C:\$($using:tempFile.Name)" ❷
        $image | Dismount-DiskImage ❸
    }
}
Invoke-Command @icmParams
```

*清单 18-5：使用 `Invoke-Command` 挂载、安装和卸载映像*

首先，你在远程机器上挂载复制的 ISO 文件 ❶；然后你执行安装程序，将输出赋值给 `$null` ❷，因为你不需要它；最后，完成后，你卸载该映像 ❸。在清单 18-5 中，你使用 `Invoke-Command` 和 PowerShell Direct 来远程执行这些命令。

安装完 SQL Server 后，进行一些清理工作，确保删除所有临时复制的文件，如清单 18-6 所示。

```
$scriptBlock = { Remove-Item -Path 'C:\en_sql_server_2016_standard_x64_dvd
_8701871.iso', "C:\$($using:tempFile.Name)" -Recurse -ErrorAction Ignore }
Invoke-Command -ScriptBlock $scriptBlock -Session $session
$session | Remove-PSSession
```

*清单 18-6：清理临时文件*

到此为止，SQL Server 已经设置完成并准备就绪！仅用 64 行 PowerShell，你就从一个 Hyper-V 主机创建了一个 Microsoft SQL Server。这是一个很大的进展，但你可以做得更好。

### 自动化 SQL Server

你已经完成了大部分繁重的工作。到目前为止，你已经有了一个可以完成所需操作的脚本。接下来，你需要做的是将所有这些功能整合到 PowerLab 模块中的几个函数里：`New-PowerLabSqlServer` 和 `Install-PowerLabOperatingSystem` 函数。

你将遵循前几章中建立的基本自动化模式：围绕所有常见操作构建函数并调用它们，而不是在许多地方使用硬编码值。最终结果将是一个用户可以调用的单一函数。在清单 18-7 中，你使用现有函数创建虚拟机和 VHD，并创建第二个`Install-PowerLabSQLServer`函数来存放安装 SQL Server 的代码：

```
function New-PowerLabSqlServer {
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
        [int64]$Memory = 2GB,

        [Parameter()]
        [string]$Switch = 'PowerLab',

        [Parameter()]
        [int]$Generation = 2,

        [Parameter()]
        [string]$DomainName = 'powerlab.local',

        [Parameter()]
        [string]$AnswerFilePath = "C:\Program Files\WindowsPowerShell\Modules\PowerLab
        \SqlServer.ini"
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
 Invoke-Command -VMName $Name -ScriptBlock { Add-Computer @using:addParams } -Credential
    $VMCredential
    Wait-Server -Name $Name -Status Offline -Credential $VMCredential
    Wait-Server -Name $Name -Status Online -Credential $DomainCredential
    $tempFile = Copy-Item -Path $AnswerFilePath
    -Destination "C:\Program Files\WindowsPowerShell\Modules\PowerLab\temp.ini" -PassThru

    Install-PowerLabSqlServer -ComputerName $Name -AnswerFilePath $tempFile.FullName
}
```

*清单 18-7：`New-PowerLabSqlServer`函数*

你应该能识别出大部分代码：这正是我们刚才讲解过的代码，只是现在它被封装成一个函数，便于重用！我使用了相同的代码主体，但不再使用硬编码值，而是将许多属性参数化，使你可以使用不同的参数安装 SQL Server，而无需修改代码本身。

将特定的脚本转化为通用函数可以保留代码的功能性，并在将来你想更改 SQL Server 部署行为时提供更大的灵活性。

让我们来看看清单 18-8 中`Install-PowerLabSqlServer`代码的重要部分。

```
function Install-PowerLabSqlServer {
    ❶ param
    (

        [Parameter(Mandatory)]
        [string]$ComputerName,

        [Parameter(Mandatory)]
        [pscredential]$DomainCredential,

        [Parameter(Mandatory)]
        [string]$AnswerFilePath,

        [Parameter()]
        [string]$IsoFilePath = 'C:\PowerLab\ISOs\en_sql_server_2016_standard
        _x64_dvd_8701871.iso'
    )

    try {
        --snip--

     ❷ ## Test to see if SQL Server is already installed
        if (Invoke-Command -Session $session
        -ScriptBlock { Get-Service -Name 'MSSQLSERVER' -ErrorAction Ignore }) {
            Write-Verbose -Message 'SQL Server is already installed'
        } else {

         ❸ PrepareSqlServerInstallConfigFile -Path $AnswerFilePath
 --snip--
    } catch {
        $PSCmdlet.ThrowTerminatingError($_)
    }
}
```

*清单 18-8：`Install-PowerLabSqlServer` PowerLab 模块函数*

你将安装 SQL Server 所需的所有输入类型进行了参数化❶，并添加了错误处理步骤❷来检查 SQL Server 是否已经安装。这使得你可以反复运行该函数；如果 SQL Server 已经安装，函数会直接跳过。

注意，你调用了一个你之前没见过的函数：`PrepareSqlServerInstallConfigFile` ❸。这是一个*辅助函数*：一个小函数，捕捉一些你可能会反复使用的功能（辅助函数通常对用户隐藏，并在后台使用）。虽然这不是必须的，但将小块功能拆分出来会使代码更具可读性。一般来说，函数应该只做一件“事”。这里的“事”当然是一个相对的概念，但你编程的越多，你就会有一种直觉，知道什么时候一个函数在做太多事情。

清单 18-9 是`PrepareSqlServerInstallConfigFile`函数的代码。

```
function PrepareSqlServerInstallConfigFile {
    [CmdletBinding()]
    param
    (

        [Parameter(Mandatory)]
        [string]$Path,

        [Parameter()]
        [string]$ServiceAccountName = 'PowerLabUser',

        [Parameter()]
        [string]$ServiceAccountPassword = 'P@$$w0rd12',

        [Parameter()]
        [string]$SysAdminAccountName = 'PowerLabUser'
    )

    $configContents = Get-Content -Path $Path -Raw
    $configContents = $configContents.Replace('SQLSVCACCOUNT=""',
    ('SQLSVCACCOUNT="{0}"' -f $ServiceAccountName))
    $configContents = $configContents.Replace('SQLSVCPASSWORD=""',
    ('SQLSVCPASSWORD="{0}"' -f $ServiceAccountPassword))
    $configContents = $configContents.Replace('SQLSYSADMINACCOUNTS=""',
    ('SQLSYSADMINACCOUNTS="{0}"' -f $SysAdminAccountName))
    Set-Content -Path $Path -Value $configContents
}
```

*清单 18-9：`PrepareSqlServerInstallConfigFile`辅助函数*

你会从清单 18-4 中识别到这段代码；它变化不大。你添加了参数`Path`、`ServiceAccountName`、`ServiceAccountPassword`和`SysAdminAccountName`来表示每个属性，而不是之前使用的硬编码值。

现在，你已经有了所有的函数，启动一个 SQL 服务器仅需几个命令。运行以下代码即可从头开始启动 SQL 服务器！

```
PS> $vmCred = Import-CliXml -Path 'C:\PowerLab\VMCredential.xml'
PS> $domainCred = Import-CliXml -Path 'C:\PowerLab\DomainCredential.xml'
PS> New-PowerLabSqlServer -Name SQLSRV -DomainCredential $domainCred -VMCredential $vmCred
```

### 运行 Pester 测试

又到了该测试的时候了：让我们运行一些 Pester 测试来检验你实施的新更改。在本章中，你在现有的 SQLSRV 虚拟机上安装了 SQL Server。在安装时，你没有做太多配置，并接受了大部分默认安装选项，因此你只需要进行几个 Pester 测试：你需要确保 SQL Server 已经安装，并且确保在安装过程中它读取了你提供的无人值守配置文件。你可以通过验证`PowerLabUser`是否拥有服务器的 sysadmin 角色，并且 SQL Server 是否以`PowerLabUser`账户运行来完成这一点：

```
PS> Invoke-Pester 'C:\PowerShellForSysAdmins\Part II\Creating and Configuring
SQL Servers\Creating and Configuring SQL Servers.Tests.ps1'

Describing SQLSRV
   Context SQL Server installation
    [+] SQL Server is installed 4.33s
   Context SQL Server configuration
    [+] PowerLabUser holds the sysadmin role 275ms
    [+] the MSSQLSERVER is running under the PowerLabUser account 63ms
Tests completed in 6.28s
Passed: 3 Failed: 0 Skipped: 0 Pending: 0 Inconclusive: 0
```

一切都通过了检查，所以你可以继续前进了！

### 总结

在本章中，你终于看到了一个更为具体的 PowerShell 应用示例。在前几章的基础上，你添加了最终的自动化层：在“层叠”在虚拟机上的操作系统上安装软件（SQL Server）。你以类似于前几章的方式进行了操作。你使用一个示例来确定所需的代码；然后，你将这些代码封装成可重用的格式，并将其放入你的 PowerLab 模块中。现在，这一切完成了，你可以通过几行代码创建任意多的 SQL 服务器！

在下一章，你将做一些不同的事情：重新审视你已经写过的代码并进行重构。你将学习最佳编码实践，并确保在添加最终部分之前，你的模块已经处于你需要的状态，这部分内容会出现在第二十章。
