## 第二十章：创建与配置 IIS Web 服务器

![图片](img/common.jpg)

你已经完成了自动化过程的最后一步：Web 服务器。在本章中，你将使用 *IIS*，一个内置的 Windows 服务，提供 Web 服务给客户端。IIS 是你在进行 IT 工作时常常遇到的服务器类型——换句话说，它是一个非常适合自动化的领域！与前几章一样，你首先将从零部署一个 IIS Web 服务器；然后你将专注于安装服务并应用一些基本配置。

### 前提条件

到现在为止，你应该已经熟悉如何创建和设置一个新的虚拟机，因此我们不会再重复这些步骤。我假设你已经有一个安装了 Windows Server 的虚拟机。如果没有，你可以通过运行以下命令，利用我们在 PowerLab 模块中现有的工作：

```
PS> New-PowerLabServer -ServerType Generic 
-DomainCredential (Import-Clixml -Path C:\PowerLab\DomainCredential.xml)
-VMCredential (Import-Clixml -Path C:\PowerLab\VMCredential.xml) -Name WEBSRV
```

注意，这次你指定了一个 `Generic` 服务器类型；这是因为你还没有为 Web 服务器提供完全的支持（这就是本章的任务！）。

### 安装与设置

创建虚拟机后，就该设置 IIS 了。IIS 是一个 Windows 功能，幸运的是，PowerShell 提供了一个内置命令来安装 Windows 功能，叫做 `Add-WindowsFeature`。如果你只是做一次性测试，你 *可以* 使用一行代码来安装 IIS，但既然你正在将这个自动化集成到一个更大的项目中，你将像安装 SQL 一样通过创建一个函数来安装 IIS。我们将其命名为 `Install-PowerLabWebServer`。

你将让这个函数遵循你之前创建的 `Install-PowerLabSqlServer` 函数的模型。当你开始为这个项目增加更多服务器支持时，你会发现，即使只是为一行代码创建一个函数，也能让使用和修改模块变得更加容易！

最简单的方式是尽可能地模仿 `Install-PowerLabSqlServer` 函数，去掉任何 SQL Server 特定的代码。通常，我会建议重用现有的函数而不是再创建一个新的，但在这个案例中，你有一个完全不同的“对象”：SQL Server 与 IIS 服务器。拥有一个不同的函数更为合理。在 清单 20-1 中，你只需复制 `Install-PowerLabSqlServer` 函数，去掉其“核心”部分，同时保留所有公共参数（你需要排除 `AnswerFilePath` 和 `IsoFilePath` 参数，因为 IIS 不需要这些参数）。

```
function Install-PowerLabWebServer {
    param
    (

        [Parameter(Mandatory)]
        [string]$ComputerName,

        [Parameter(Mandatory)]
        [pscredential]$DomainCredential
    )

    $session = New-PSSession -VMName $ComputerName -Credential $DomainCredential

    $session | Remove-PSSession
}
```

*清单 20-1： “框架” `Install-PowerLabWebServer` 函数*

至于如何设置 IIS 服务，那简直是小菜一碟：你只需要运行一个命令来安装 `Web-Server` 功能。赶紧将这一行添加到你的 `Install-PowerLabWebServer` 函数中（清单 20-2）。

```
function Install-PowerLabWebServer {
    param
    (

        [Parameter(Mandatory)]
        [string]$ComputerName,

        [Parameter(Mandatory)]
        [pscredential]$DomainCredential
    )

    $session = New-PSSession -VMName $ComputerName -Credential $DomainCredential

    $null = Invoke-Command -Session $session -ScriptBlock { Add-WindowsFeature -Name 'Web-Server' }

    $session | Remove-PSSession
}
```

*清单 20-2： `Install-PowerLabWebServer` 函数*

你的 `Install-PowerLabWebServer` 函数的开头部分已经完成！接下来我们添加更多代码。

### 从零构建 Web 服务器

现在，您已经有了一个 IIS 安装功能，是时候更新您的`New-PowerLabServer`函数了。回想一下在第十九章中，当您在重构`New-PowerLabServer`函数时，由于缺乏所需功能，您不得不使用占位符代码来处理 Web 服务器部分。您使用了这一行`Write-Host 'Web server deployments are not` `supported` `at this` `time'`作为填充代码。现在，让我们将这段文本替换为调用您新创建的`Install-PowerLabWebServer`函数：

```
PS> Install-PowerLabWebServer –ComputerName $Name –DomainCredential $DomainCredential
```

完成此操作后，您可以像处理 SQL 服务器一样启动 Web 服务器！

### WebAdministration 模块

一旦 Web 服务器启动并运行，您需要对其进行操作。当`Web-Server`功能在服务器上启用时，会安装一个名为`WebAdministration`的 PowerShell 模块。此模块包含了处理 IIS 对象所需的多个命令。`Web-Server`功能还会创建一个名为 IIS 的 PowerShell 驱动程序，允许您管理常见的 IIS 对象（如网站、应用程序池等）。

*PowerShell 驱动程序*使您能够像操作文件系统一样浏览数据源。接下来，您将看到，您可以像操作文件和文件夹一样，使用常见的 cmdlet（如`Get-Item`、`Set-Item`和`Remove-Item`）来操作网站、应用程序池以及其他许多 IIS 对象。

要使 IIS 驱动程序可用，您首先需要导入`WebAdministration`模块。让我们远程连接到您新创建的 Web 服务器，并稍微操作一下该模块，看看您能做些什么。

首先，您将创建一个新的 PowerShell Direct 会话，并以交互模式进入。之前，您主要使用`Invoke-Command`将命令发送到虚拟机。现在，由于您只是在调查 IIS 的可能性，您使用`Enter-PSSession`以交互方式在会话中工作：

```
PS> $session = New-PSSession -VMName WEBSRV 
-Credential (Import-Clixml -Path C:\PowerLab\DomainCredential.xml)
PS> Enter-PSSession -Session $session
[WEBSRV]: PS> Import-Module WebAdministration
```

注意最终提示符前的`[WEBSRV]`。这表明您现在正在操作 WEBSRV 主机，并且可以导入`WebAdministration`模块。一旦模块被导入到会话中，您可以通过运行`Get-PSDrive`来验证 IIS 驱动程序是否已创建：

```
[WEBSRV]: PS> Get-PSDrive -Name IIS | Format-Table -AutoSize

Name Used (GB) Free (GB) Provider          Root     CurrentLocation
---- --------- --------- --------          ----     ---------------
IIS                      WebAdministration \\WEBSRV
```

您可以像使用任何其他 PowerShell 驱动程序一样浏览此驱动程序：通过将其视为文件系统，使用`Get-ChildItem`列出驱动程序中的项，使用`New-Item`创建新项，以及使用`Set-Item`修改项。但执行这些操作并不等于自动化；这只是通过命令行管理 IIS。而您是来进行自动化的！我之所以现在提到 IIS 驱动程序，是因为它在后续的自动化任务中会派上用场，而且了解如何手动操作总是好事，万一自动化出问题，您可以进行故障排除。

#### 网站和应用程序池

`WebAdministration`模块中的命令几乎可以管理和自动化 IIS 的每个方面。你将首先了解如何处理网站和应用程序，因为网站和应用程序池是系统管理员在现实世界中最常操作的两个常见组件。

##### 网站

你将从一个简单的命令开始：`Get-Website`，它允许你查询 IIS 并返回当前在 Web 服务器上存在的所有网站：

```
[WEBSRV]: PS> Get-Website -Name 'Default Web Site'

Name             ID   State      Physical Path                  Bindings
----             --   -----      -------------                  --------
Default Web Site 1    Started    %SystemDrive%\inetpub\wwwroot  http *:80:
```

你会注意到你已经创建了一个网站。这是因为 IIS 在安装时会有一个名为“Default Web Site”的默认网站。但假设你不想要这个默认网站，而是想创建你自己的网站，你可以通过将`Get-Website`命令的输出管道传递给`Remove-Website`来删除它：

```
[WEBSRV]: PS> Get-Website -Name 'Default Web Site' | Remove-Website
[WEBSRV]: PS> Get-Website
[WEBSRV]: PS>
```

如果你想创建一个网站，你也可以像使用`New-Website`命令那样轻松创建一个：

```
[WEBSRV]: PS> New-Website -Name PowerShellForSysAdmins
-PhysicalPath C:\inetpub\wwwroot\

Name             ID   State      Physical Path                  Bindings
----             --   -----      -------------                  --------
PowerShellForSys 1052 Stopped    C:\inetpub\wwwroot\            http *:80:
Admins           6591
```

如果网站的绑定有问题，你想要更改它们（比如你想绑定到非标准端口），你可以使用`Set-WebBinding`命令：

```
[WEBSRV]: PS> Set-WebBinding -Name 'PowerShellForSysAdmins'
-BindingInformation "*:80:" -PropertyName Port -Value 81
[WEBSRV]: PS> Get-Website -Name PowerShellForSysAdmins

Name             ID   State      Physical Path                  Bindings
----             --   -----      -------------                  --------
PowerShellForSys 1052 Started    C:\inetpub\wwwroot\            http *:81:
Admins           6591
                 05
```

你已经看到很多关于网站的操作。接下来，我们来看看应用程序池有什么可能性。

##### 应用程序池

*应用程序池*允许你将应用程序彼此隔离，即使它们运行在同一台服务器上。这样，如果一个应用程序出现错误，它不会影响其他应用程序。

应用程序池的命令与网站的命令类似，正如下面的代码所示。由于我只有一个应用程序池，所以只有`DefaultAppPool`显示。如果你在自己的 Web 服务器上运行这个命令，可能会看到更多内容：

```
[WEBSRV]: PS> Get-IISAppPool

Name                 Status       CLR Ver  Pipeline Mode  Start Mode
----                 ------       -------  -------------  ----------
DefaultAppPool       Started      v4.0     Integrated     OnDemand

[WEBSRV]: PS> Get-Command -Name *apppool*

CommandType     Name                              Version    Source
-----------     ----                              -------    ------
Cmdlet          Get-IISAppPool                    1.0.0.0    IISAdministration
Cmdlet          Get-WebAppPoolState               1.0.0.0    WebAdministration
Cmdlet          New-WebAppPool                    1.0.0.0    WebAdministration
Cmdlet          Remove-WebAppPool                 1.0.0.0    WebAdministration
Cmdlet          Restart-WebAppPool                1.0.0.0    WebAdministration
Cmdlet          Start-WebAppPool                  1.0.0.0    WebAdministration
Cmdlet          Stop-WebAppPool                   1.0.0.0    WebAdministration
```

由于你已经创建了一个网站，接下来我们来看看如何创建应用程序池并将它分配给你的网站。要创建应用程序池，请使用`New-WebAppPool`命令，如示例 20-3 所示。

```
[WEBSRV]: PS> New-WebAppPool -Name 'PowerShellForSysAdmins'

Name                     State        Applications
----                     -----        ------------
PowerShellForSysAdmins   Started
```

*示例 20-3：创建应用程序池*

不幸的是，并非所有 IIS 任务都有内置的 cmdlet。要将应用程序池分配给现有的网站，你需要使用`Set-ItemProperty`并更改 IIS 驱动器中的网站❶（如下所示）。要应用该更新，你需要停止❷并重新启动❸该网站。

```
❶ [WEBSRV]: PS> Set-ItemProperty -Path 'IIS:\Sites\PowerShellForSysAdmins'
   -Name 'ApplicationPool' -Value 'PowerShellForSysAdmins'
❷ [WEBSRV]: PS> Get-Website -Name PowerShellForSysAdmins | Stop-WebSite
❸ [WEBSRV]: PS> Get-Website -Name PowerShellForSysAdmins | Start-WebSite
   [WEBSRV]: PS> Get-Website -Name PowerShellForSysAdmins | 
      Select-Object -Property applicationPool
   applicationPool
   ---------------
   PowerShellForSysAdmins
```

你还可以看到，你可以通过查看运行`Get-Website`命令返回的`applicationPool`属性来确认应用程序池是否已更改。

### 配置网站的 SSL

现在你已经了解了用于操作 IIS 的命令，接下来我们回到你的 PowerLab 模块，编写一个函数，用来安装 IIS 证书并将绑定更改为端口 443。

你可以从有效的证书颁发机构获取一个“真实”的证书，或者通过使用`New-SelfSignedCertificate`函数创建一个自签名证书。因为我只是演示这个概念，所以我们现在就创建一个自签名证书并使用它。

首先，编写这个函数，并指定你需要的所有参数（见示例 20-4）。

```
function New-IISCertificate {
    param(

            [Parameter(Mandatory)]
            [string]$WebServerName,

            [Parameter(Mandatory)]
            [string]$PrivateKeyPassword,

            [Parameter()]
            [string]$CertificateSubject = 'PowerShellForSysAdmins',

            [Parameter()]
            [string]$PublicKeyLocalPath = 'C:\PublicKey.cer',

            [Parameter()]
            [string]$PrivateKeyLocalPath = 'C:\PrivateKey.pfx',

            [Parameter()]
            [string]$CertificateStore = 'Cert:\LocalMachine\My'
    )
    ## The code covered in the following text will go here

}
```

*示例 20-4：`New-IISCertificate`的开始*

这个函数需要做的第一件事是创建一个自签名证书。你可以使用 `New-SelfSignedCertificate` 命令来完成这项操作，该命令将证书导入本地计算机的 `LocalMachine` *证书存储* 中，所有计算机的证书都存放在这里。当你调用 `New-SelfSignedCertificate` 时，你可以传递一个 `Subject` 参数来存储一个字符串，该字符串会告诉你证书的相关信息。生成证书时，它也会被导入到本地计算机中。

列表 20-5 提供了你将用于生成证书的代码行，该代码行使用了传入的主题（`$CertificateSubject`）。记住，你可以使用`$null`变量来存储命令的结果，这样就不会将任何内容输出到控制台。

```
$null = New-SelfSignedCertificate -Subject $CertificateSubject
```

*列表 20-5：创建自签名证书*

一旦证书被创建，你需要做两件事：获取证书的指纹，并从证书中导出私钥。证书的 *指纹* 是一个唯一标识证书的字符串；证书的 *私钥* 用于加密和解密发送到服务器的数据（这里我不详细讲解）。

你本可以从 `New-SelfSignedCertificate` 的输出中获取指纹，但我们假设这个证书将被用于与创建它的计算机不同的计算机上，因为这更符合实际情况。为了解决这个问题，你需要先从自签名证书中导出公钥，可以使用 `Export-Certificate` 命令来完成：

```
$tempLocalCert = Get-ChildItem -Path $CertificateStore | 
    Where-Object {$_.Subject -match $CertificateSubject } 
$null = $tempLocalCert | Export-Certificate -FilePath $PublicKeyLocalPath
```

上面的命令将给你一个 *.cer* 公钥文件，你可以使用它，以及一些 .NET 魔法，暂时导入证书并检索指纹：

```
$certPrint = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2
$certPrint.Import($PublicKeyLocalPath)
$certThumbprint = $certprint.Thumbprint
```

现在你有了证书的指纹，你需要导出私钥，你将使用它来绑定到 Web 服务器上的 SSL。以下是导出私钥的命令：

```
$privKeyPw = ConvertTo-SecureString -String $PrivateKeyPassword -AsPlainText -Force
$null = $tempLocalCert | Export-PfxCertificate -FilePath $PrivateKeyLocalPath -Password $privKeyPw
```

一旦你有了私钥，就可以使用 `Import-PfxCertificate` 命令将证书导入到 Web 服务器的证书存储中。然而，在此之前，你需要检查证书是否已被导入。这就是为什么你需要先获取指纹的原因。你可以使用证书的唯一指纹来验证它是否已存在于 Web 服务器上。

要导入你的证书，你需要使用本章前面看到的几个命令：你将创建一个 PowerShell 直接会话，导入 `WebAdministration` 模块，检查证书是否存在，如果不存在则添加它。你暂时跳过最后一步，并在列表 20-6 中编写代码完成其余的操作。

```
$session = New-PSSession –VMName $WebServerName 
–Credential (Import-CliXml –Path C:\PowerLab\DomainCredential.xml)

Invoke-Command –Session $session –ScriptBlock {Import-Module –Name
WebAdministration}

if (Invoke-Command –Session $session –ScriptBlock { $using:certThumbprint –in
(Get-ChildItem –Path Cert:\LocalMachine\My).Thumbprint}) {
      Write-Warning –Message 'The Certificate has already been imported.'
} else {
      # Code for importing the certificate
}
```

*列表 20-6：检查证书是否已存在*

代码的前两行你应该已经在本章早些时候见过，但请注意，你需要使用 `Invoke-Command` 来远程导入模块。同样，由于你在 `if` 语句的脚本块中使用了本地变量，你需要使用 `$using:` 前缀来扩展远程计算机上的变量。

让我们在 Listing 20-7 中填写 `else` 语句的代码。你需要做四件事来完成 IIS 证书的设置。首先，你需要将私钥复制到 Web 服务器上。然后，你需要使用 `Import-PfxCertificate` 导入私钥。最后，你需要设置 SSL 绑定，并强制它使用私钥：

```
Copy-Item -Path $PrivateKeyLocalPath -Destination 'C:\' -ToSession $session

Invoke-Command -Session $session -ScriptBlock { Import-PfxCertificate 
-FilePath $using:PrivateKeyLocalPath -CertStoreLocation
$using:CertificateStore -Password $using:privKeyPw }

Invoke-Command -Session $session -ScriptBlock { Set-ItemProperty "IIS:\Sites
\PowerShellForSysAdmins" -Name bindings
-Value @{protocol='https';bindingInformation='*:443:*'} }

Invoke-Command -Session $session -ScriptBlock {
    $cert = Get-ChildItem -Path $CertificateStore | 
        Where-Object { $_.Subject -eq "CN=$CertificateSubject" }
    $cert | New-Item 'IIS:\SSLBindings\0.0.0.0!443' 
}
```

*Listing 20-7: 将 SSL 证书绑定到 IIS*

需要指出的是，在这段代码中，你将网站的绑定端口设置为 443，而不是 80\。这样做是为了确保网站遵循典型的 SSL 端口 443，允许 Web 浏览器理解你正在加密 Web 流量。

到目前为止，你已经完成了！你已经成功地在 Web 服务器上安装了一个自签名证书，创建了站点的 SSL 绑定，并强制 SSL 绑定使用你的证书！剩下的就是清理你所工作的会话：

```
$session | Remove-PSSession
```

在清理会话后，你可以浏览到 *https://<webservername>*，并会被提示信任该证书。所有浏览器都会这么做，因为你颁发了一个自签名证书，而不是由公共证书授权机构颁发的证书。信任该证书后，你将看到默认的 IIS 网页。

请务必查看 PowerLab 模块中的 `New-IISCertificate` 函数，了解如何在一个地方查看所有这些命令。

### 总结

本章介绍了另一种类型的服务器——Web 服务器。你学习了如何从零开始创建 Web 服务器，方法与创建 SQL 服务器完全相同。你还学习了 `WebAdministration` 模块中一些命令，该模块随 IIS 一起提供。你了解了如何使用内置命令执行许多基本任务，并查看了创建的 IIS PowerShell 驱动器。为了总结本章内容，你详细跟踪了一个真实的场景，该场景需要将之前涵盖的许多命令和技术结合起来使用。

如果你已经完成了整本书，恭喜你！我们已经覆盖了很多内容，我很高兴你坚持下来了。你学到的技能和你构建的项目应该为你解决 PowerShell 问题打下基础。把你在这里学到的内容带走，合上书本，开始编写脚本吧。只要开始，并用 PowerShell 自动化它。你真正掌握本书中涉及的概念的唯一方法就是练习。现在就是最好的时机！
