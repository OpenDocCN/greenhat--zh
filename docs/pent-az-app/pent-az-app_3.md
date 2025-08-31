## 第四章：**检查存储**

![image](img/00015.jpeg)

在接下来的几章中，我们将深入探讨特定的 Azure 服务以及每种服务独特的渗透测试技术和工具。我们将从 Azure 存储账户开始，这些账户被多个 Azure 服务用于存储从日志到虚拟机“硬盘”映像的所有内容。客户还使用存储账户进行文档共享和备份，基本上是本地文件服务器的云端替代方案。当然，将所有这些数据集中存储在一个地方会成为攻击者诱人的目标。

除了其数据潜在价值之外，存储账户出于多种原因是理想的攻击目标；其中最重要的是每个存储账户都有两个密钥，可以完全控制其数据。这些密钥被所有使用该存储账户的服务和所有账户管理员共享。更糟糕的是，大多数客户从不更改它们。

这些做法会导致否认、授权和补救问题（如果确实发生攻击）。存储账户密钥还可能存在用户自身引发的弱点：因为许多应用程序需要存储访问权限，开发人员经常在其代码或配置文件中嵌入存储密钥，而未考虑可能的安全后果。

在本章中，我们首先讨论 Azure 存储中可用的不同身份验证方法。然后，我们将看看如何在源代码中找到这些凭据，接着看看每个常用工具用于访问和管理 Azure 存储的方法以及如何从中窃取凭据。这非常重要，因为您事先不知道将在开发人员系统中遇到哪些实用程序。最后，我们将研究如何从存储账户中检索不同形式的数据。这有两个目的：首先，向客户证明未经适当保护的云存储存在重大数据泄露的风险；其次，账户中的数据有时可以用于获取对环境的额外访问权限。

### 最佳实践：存储安全

不当配置的云存储在 2016 年至 2018 年间已在超过二十起公开披露的数据泄露事件中被提及。通常，当开发人员编写程序化访问云存储容器的代码，并将访问密钥嵌入源代码中并提交到源代码管理系统时，会出现问题。由于许多公司使用像 GitHub 这样的服务来托管他们的代码，开发人员可能没有意识到他们提交密码的代码库是公开可访问的。偶尔，泄露也发生在存储账户被配置为任何人都可以读取，无需密码的情况下。由于恶意行为者定期扫描公共代码库，寻找密码和存储账户网址，试图获取访问权限，因此错误发生与泄露之间的时间可能非常短。但即使对代码库的访问是有限的，能够访问代码的人数通常也高于被授权访问密钥的人数。此外，密钥和密码绝不应以明文形式存储，即使是临时存储也不行。

作为管理员，您可以采取若干措施来防范这些问题。首先，定期进行存储账户的“滚动”或重置访问密钥，并记录需要更新密钥的地方。这样，如果发生实际事件，您可以在不担心破坏依赖服务的情况下开始修复工作。

接下来，尽可能启用云存储的数据传输和静态数据加密。从 2017 年底开始，Azure 默认加密 Azure 存储中的所有静态数据，使用的是自动管理的密钥。如果需要，管理员可以通过 Azure 门户的存储账户设置提供自己的加密密钥。然而，尽管此设置保护了存储介质上的数据，但它并没有保护数据在上传或下载过程中。为此，存储账户必须配置为仅允许通过 HTTPS 协议连接。可以在 Azure 门户的存储账户配置设置中通过启用“要求安全传输”选项来实现。也可以通过 PowerShell 启用此功能：

```
PS C:\> Set-AzureRmStorageAccount -Name "StorageName" -ResourceGroupName "
GroupName" -EnableHttpsTrafficOnly $True
```

为了确保存储账户不会被超过预期的人访问，请定期检查存储容器的访问类型设置。除非您打算允许匿名访问，否则它应设置为私有。此外，您可以使用共享访问签名（SAS）访问令牌来指定存储账户内更细粒度的权限，包括限制访问的时间段和 IP 范围。有关这些权限的更多信息，请参阅 *[`docs.microsoft.com/en-us/azure/storage/blobs/storage-manage-access-to-resources/`](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-manage-access-to-resources/)*。

最后，定期进行代码审查，查找开发人员将密钥检查到源代码中的实例。你甚至可以考虑使用代码分析工具，在每次提交新代码时自动检查是否包含密码。这不仅对查找存储帐户密钥有帮助，还能帮助发现其他凭据。

### 访问存储帐户

可以通过存储帐户密钥、用户凭据和*共享访问签名（SAS）*令牌访问 Azure 存储，这些令牌是带有嵌入式访问密钥的 URL，通常提供对文件子集的访问权限，并可能有其他限制。每种凭据类型有不同的用途，有些对渗透测试人员比其他类型更有用。让我们逐一检查这些凭据。

#### *存储帐户密钥*

使用存储帐户密钥和存储帐户名称是最理想且最常用的攻击方法，因为它们允许完全访问整个存储帐户，而无需 2FA。存储帐户只有两个密钥——一个主密钥和一个次密钥——所有存储帐户用户共享这些密钥。这些密钥不会自动过期，但可以被更新。与可以由用户选择的密码不同，存储密钥是自动生成的 64 字节值，以 base64 编码表示，这使它们在源代码或配置文件中容易被识别。

存储密钥也受到每个 Azure 存储工具和与存储相关的 API 的支持，因此它们非常通用。此外，它们是开发人员最常使用的凭据，并且不经常更改，因此获取有效密钥的机会较大。

#### *用户凭据*

获取用户凭据是下一个最佳的访问方式。尽管基于角色的权限可以限制用户帐户对存储帐户执行某些操作的能力，但实际上，这种细粒度的权限很少被实施。依赖这些凭据的最大缺点是可能遇到双因素认证（2FA）。如果用户帐户启用了 2FA，那么在没有使用在“遇到双因素认证”中讨论过的某些方法的情况下，无法冒充该用户。这些方法增加了攻击的复杂性，降低了成功的概率。在使用用户凭据时，另一个障碍是工具的支持缺乏。我们将在本章后面讨论的许多 Azure 存储工具仅接受存储密钥，因此您可能需要使用用户凭据登录到 Azure 门户并复制存储密钥以供使用。

#### *SAS 令牌*

SAS 令牌是仅授予对存储帐户中某些对象的访问权限的密钥。例如，SAS 令牌用于启用 OneDrive、SharePoint Online、Office 365、Dropbox 和类似服务中的“共享文件”选项。

Azure SAS 令牌格式为指向 Azure 存储的 URL，其中包含一长串参数和一个独特的 SHA256 哈希值，经过 base64 编码，类似这样： *[`storagerm.blob.core.windows.net/container/file.txt?st=2017-04-09T01%3A00%3A00Z&se=2017-04-20T01%3A00%3A00Z&sp=r&sip=127.0.0.1-127.0.0.100&sig=7%2BwycBOdzx8IS4zhMcKNw7AHvnZlYwk8wXIqNtLEu4s%3D`](https://storagerm.blob.core.windows.net/container/file.txt?st=2017-04-09T01%3A00%3A00Z&se=2017-04-20T01%3A00%3A00Z&sp=r&sip=127.0.0.1-127.0.0.100&sig=7%2BwycBOdzx8IS4zhMcKNw7AHvnZlYwk8wXIqNtLEu4s%3D)*。

渗透测试人员可能会发现 SAS 令牌并不特别有用，不仅因为它们通常只对一小部分文件有效，还因为它们可能已分配了某些权限（通过 SP 参数），例如只读。SAS 令牌也可以指定只能从特定的 IP 地址或范围（通过 SIP 参数）使用，因此即使你得到了一个 SAS 令牌，它也可能只对原本创建它的机器有效。SAS 令牌还可能有指定的开始和结束时间（通过 ST 和 SE 参数，分别限制令牌的有效期）。

如果这些还不够让你灰心丧气，大多数 Azure 工具并不支持 SAS 令牌。这意味着你可能只能通过 web 浏览器使用它们。而且，如果你偶然找到这些令牌的缓存，逐一查看它们将需要一些时间，从而浪费宝贵的测试时间。话虽如此，如果前两种凭证类型不可用，能够使用 SAS 令牌总比完全没有访问权限要好。

**防御者提示**

微软提供了关于选择正确存储身份验证选项、常见陷阱、可能的缓解措施以及如何从泄露凭证中恢复的详细指南，详细内容可见于 *[`docs.microsoft.com/en-us/azure/storage/storage-security-guide`](https://docs.microsoft.com/en-us/azure/storage/storage-security-guide)*。

### 查找存储凭证的位置

现在你知道了需要查找哪些类型的凭证，我们来看看它们可能最常见的几个存放位置：源代码和存储管理工具。对于源代码侦查，你需要访问开发人员的机器或他们的源代码控制系统。要从存储工具中提取密钥，你需要找到这些工具安装的位置；通常这些工具安装在开发人员的工作站上。有了对这些系统的访问权限，你就可以开始寻找密钥。

#### *在源代码中查找密钥*

查找存储密钥的最直接方法是通过使用 Azure 存储的应用程序的源代码—通常是在用于构建从 Azure 网站到自定义商业应用程序的配置文件中，这些应用程序使用云来存储数据。你有多种方法可以快速定位源代码中的存储密钥，但应选择的方式取决于你所找到的代码类型。

Microsoft 为 .NET（C# 和 Visual Basic）以及 Java 提供了库，使得访问存储和其他 Azure 功能更加简便。幸运的是，这些库中用于验证 Azure 存储的函数名称是一致的。搜索 *StorageCredentials* 类的实例，你很可能会找到任何应用程序使用存储密钥的地方。如果这不起作用，可以尝试搜索库的完整名称，比如在 .NET 中是 *Microsoft.WindowsAzure.Storage.Auth*，在 Java 中是 *com.microsoft.azure.storage.StorageCredentials*。

如果你怀疑某个存储实例可能使用了 SAS 令牌，可以在代码仓库中搜索 *.core.windows.net*，这是所有 SAS 令牌 URL 中使用的域名。（SAS 令牌中的 base64 签名应该使它们与任何其他 *windows.net* 域名引用易于区分。）

许多代码库将存储账户密钥放入配置文件中，尤其是在与 ASP.NET 和 Azure 网站结合使用时。ASP.NET 和 Azure 网站使用名为 *web.config* 的文件，而其他网站则通常使用 *app.config* 文件。配置文件中的存储账户密钥通常标记为 *StorageAccountKey*、*StorageServiceKeys* 或 *StorageConnectionString*（这是某些 Microsoft 文档示例代码中使用的名称）。

你可以通过扫描 *azure-storage.common.js* 来识别 JavaScript 文件中的 Azure 存储使用。如果你在代码中找到这个脚本引用，还需要查找 *AzureStorage.createBlobService*；你需要它来进行 Azure 身份验证。（JavaScript 库允许使用存储密钥和 SAS 令牌，但强烈鼓励使用高度限制的 SAS 令牌，因为用户可以查看 JavaScript 代码。）

#### *从开发人员的存储工具中获取密钥*

如果你无法在源代码中找到存储密钥，你可以尝试从开发人员用来传输文件到 Azure 的工具中恢复这些密钥。要找到这些密钥，首先需要访问开发人员的工作站，然后查找 Azure 存储管理应用程序。一旦访问成功，检查该应用程序是否在其用户界面中暴露了已保存的密钥，或者是否以不安全的方式保存了密钥。

在本节中，我们将查看最常用的存储账户管理工具，看看它们是否容易受到此类攻击。

**防御者提示**

在接下来的讨论中，请注意，只有 Microsoft Azure Storage Explorer 使得攻击者恢复密钥变得困难。如果你必须使用工具来管理 Azure 存储，并且系统中缓存了凭据，Microsoft Azure Storage Explorer 是最安全的选择。

##### 从 Microsoft Azure Storage Explorer 获取密钥

Azure Storage Explorer 设计良好，存储密钥保护是其显而易见的目标。它没有显示密钥的选项，一旦密钥被保存在界面中，且加密的密钥被存储在 Windows 凭据管理器中，这使得直接恢复它们变得不实际。

尽管有这些安全功能，但一切并未失去。因为 Azure Storage Explorer 需要解密密钥，以便在传输数据时将其提供给 Azure 的 API，你可以在 Storage Explorer 的代码中设置一个断点，位置在密钥解密后的下一行，然后使用内置调试器直接在内存中查看它们。

要执行此测试，请按照以下步骤进行：

1.  在目标工程师的工作站上启动 Azure Storage Explorer。

1.  选择**帮助** ▸ **切换开发者工具**。你应该能看到调试器界面。

1.  在调试窗口中，点击屏幕顶部的**源代码**选项卡，然后点击垂直省略号菜单并选择**转到文件**，如图 4-1 所示。

    ![image](img/00021.jpeg)

    *图 4-1：Azure Storage Explorer 中的源代码视图*

1.  在出现的文件列表对话框中，输入 AzureStorageUtilities.js 并点击第一个条目加载 *AzureStorageUtilities.js* 文件，该文件包含加载存储帐户密钥的逻辑。

1.  展开调试器窗口，以便查看源代码；然后找到 `loadStorageAccounts(host, key)` 函数，该函数显示在清单 4-1 中。

    ```
        /**
         * Load the stored storage accounts:
         * Get account data from localStorage
         * Combine session key and account data as user account manager key 
         * to get account key stored there.
         * @param host
         * @param key
         */
        function loadStorageAccounts(host, key) {
            --snip--
                    switch (account.connectionType) {
                        case 1 /* sasAttachedAccount */:
                            account.connectionString = confidentialData;
                            break;
                        case 3 /* key */:
                            account.accountKey = confidentialData;
                            break;
                        default:
                            // For backward compatibility reasons if the 
                            // connection type is not set
                            // we assume it is a key
                            account.accountKey = confidentialData;
                    }
                  return account;
                });
                return storageAccounts;
            });
        }
    ```

    *清单 4-1：来自 Microsoft Azure Storage Explorer 的代码片段* loadStorageAccounts() *函数*

1.  在此函数中设置一个断点，位置在帐户信息返回应用程序之前，点击窗口左侧 `return account;` 行号，如图 4-2 所示。

1.  现在，为了触发应用程序重新加载帐户信息以便触发断点，点击帐户列表上方的**刷新所有**。调试器应该会中断并暂停应用程序。查看窗口右侧的帐户：对象变量（如图 4-2 所示），点击 `account` 旁边的箭头展开它。

![image](img/00022.jpeg)

*图 4-2：在调试器中展开的帐户对象*

`account` 对象应列出第一个在 Azure Storage Explorer 中注册的存储帐户的 `accountKey` 和 `accountName`。要查看是否有多个帐户，请按 F8 继续执行。如果有更多存储帐户，调试器应立即中断并更新帐户对象，显示下一个帐户的详细信息。继续按 F8，直到你恢复了每个存储帐户的连接信息。

当最后一个存储帐户的详细信息显示后，再次按 F8 以恢复应用程序的正常操作。然后，在右侧窗格的断点列表中右键单击并选择**移除所有断点**来移除断点。最后，点击**帮助** ▸ **切换开发者工具**以关闭调试工具，然后退出应用程序。

##### 从 Redgate 的 Azure Explorer 获取密钥

Redgate 的 Azure Explorer 提供了两种方式来访问其中包含的密钥：连接编辑器对话框和每个帐户上下文菜单中的复制选项。要查看帐户密钥，启动 Redgate 的 Azure Explorer，打开帐户，然后右键单击帐户以查看其详细信息，如 图 4-3 所示。

![image](img/00023.jpeg)

*图 4-3：Redgate 的存储帐户菜单*

编辑连接详细信息选项会打开一个对话框，如 图 4-4 所示，在这个对话框中你可以更新与存储帐户关联的密钥。对话框方便地以明文显示当前的密钥。

![image](img/00024.jpeg)

*图 4-4：Redgate 的 Azure Explorer 中的存储帐户密钥*

复制连接字符串选项也很有趣。你可以用它将密钥以 SQL 连接字符串格式复制到剪贴板，连接字符串中包含密钥本身和帐户名称，还可以指示是否应使用 SSL 或未加密的连接访问存储帐户。使用此选项抓取帐户所需的所有连接信息，然后将其粘贴到一个小文档中。对每个列出的帐户重复此操作。

**注意**

*因为 Redgate 在 Azure Explorer 的设置文件 %UserProfile%\AppData\Local\Red Gate\Azure Explorer\Settings.xml 中加密了存储密钥*，你需要能够运行 Azure Explorer 来恢复密钥；你不能仅仅拿到 XML 文件。*

##### 从 ClumsyLeaf 的 CloudXplorer 获取密钥

ClumsyLeaf 软件公司开发了三款用于与云存储交互的产品：CloudXplorer、TableXplorer 和 AzureXplorer。所有这些工具不仅可以管理 Azure 存储，还可以管理其他提供商的存储服务，如亚马逊和谷歌。

CloudXplorer 处理文件和 Blob 存储，而 TableXplorer 为表格型云存储提供类似 SQL 的接口。AzureXplorer 是一个 Visual Studio 插件，旨在简化开发过程中与云内容的交互。

你可以通过在左侧窗格中右键单击存储帐户并选择 **属性** 来查看和编辑 CloudXplorer 中存储的密钥，如 图 4-5 所示。

![image](img/00025.jpeg)

*图 4-5：CloudXplorer 中的存储帐户上下文菜单*

账户窗口（参见 图 4-6）显示当前使用的是哪个 Azure 实例，是否启用了 SSL，并且应该允许你复制存储帐户的名称和密钥。

![image](img/00026.jpeg)

*图 4-6：CloudXplorer 中的帐户信息*

**注意**

*CloudXplorer 的配置* ▸ *导出选项会导出所有存储帐户连接的详细信息，但它们是加密的。你可能会发现这不太有用。*

与 Redgate 一样，ClumsyLeaf 也将其帐户信息加密存储在 XML 文件中。你可以在 *%AppData%\ClumsyLeaf Software\CloudXplorer\accounts.xml* 找到它。

##### 从 ClumsyLeaf 的 TableXplorer 获取密钥

要使用 TableXplorer 查看存储账户，点击**管理账户**，如图 4-7 所示，打开管理账户窗口。

![image](img/00027.jpeg)

*图 4-7：TableXplorer 中的管理账户按钮*

管理账户窗口应显示每个账户，如图 4-8 所示。Azure 存储账户标有 Windows 标志，而 Amazon 账户则标有橙色立方体。点击一个账户的名称并选择**编辑**。

![image](img/00028.jpeg)

*图 4-8：TableXplorer 中的账户列表*

编辑窗口将与之前在图 4-6 中显示的 CloudXplorer 窗口相同。此外，像 CloudXplorer 一样，TableXplorer 会加密其配置文件中的密钥，该文件位于*%AppData%\ClumsyLeaf Software\TableXplorer\accounts.xml*。

##### 从 Azure Storage Explorer 6 获取密钥

Azure Storage Explorer 6 可能是这个列表中最古老的工具。尽管它不再维护，但它曾是标准工具多年，你可能会在许多开发者的系统中找到它，未来几年也可能继续使用。

要通过 Azure Storage Explorer 6 查看存储账户设置，请按照以下步骤操作：

1.  启动应用程序并从下拉列表中选择一个账户。

1.  选择账户后，点击**存储账户** ▸ **查看连接字符串**，如图 4-9 所示。

    ![image](img/00029.jpeg)

    *图 4-9：Azure Storage Explorer 6 中的存储账户菜单*

1.  你应该看到一个弹出消息框，显示格式化的 SQL 连接字符串账户密钥，如图 4-10 所示。点击**确定**将值复制到剪贴板。

    ![image](img/00030.jpeg)

    *图 4-10：Azure Storage Explorer 6 中的存储账户连接字符串*

在 Azure Storage Explorer 6 版本之前，未加密的凭据存储在*%AppData%\AzureStorageExplorer\AzureStorageExplorer.config*中，因此每当你怀疑某台机器被用来管理存储账户时，这个文件是一个值得查找的文件。从版本 6 开始，这些设置被加密并移动到*%AppData%\Neudesic\AzureStorageExplorer\<Version>\AzureStorageExplorer6.dt1*。然而，由于 Azure Storage Explorer 是开源的，并且每次安装使用相同的加密密钥，所以非常容易在网上找到它用来“保护”这些文件的加密密钥，以及加密和解密代码。当然，从 GUI 恢复存储密钥更容易，但如果你无法在目标系统上启动应用程序，那么有其他选项也会很有帮助。

### 访问存储类型

一旦你访问了存储账户，就该了解可以获取何种类型的数据了。首先，你需要确定每个账户使用了哪种存储机制（Blob、表、队列和/或文件），并记住一个账户可以使用多个存储机制。务必检查每个账户的每种存储类型。

#### *识别正在使用的存储机制*

虽然你可以使用 Azure 门户检查存储账户的内容，但渗透测试者在使用这种方法时可能会面临一些挑战。首先，账户可能只有一个管理证书，这将无法提供直接的门户访问权限。其次，Azure 门户不会在一个视图中显示每种存储类型的摘要；你需要点击每个账户，点击查看该账户中的任何 Blob，然后点击查看文件的按钮，依此类推。当订阅中包含大量存储账户时，这个过程需要花费一些时间。

识别正在使用的存储类型最好的方法是使用 PowerShell。例如，下面的 PowerShell 脚本在列表 4-2 中显示，将枚举订阅中的所有存储账户，检查每种存储机制的内容，然后显示它找到的任何内容的摘要。

```
   # ASM Storage Accounts
   Write-Output ">>> ASM <<<"
➊ $storage = Get-AzureStorageAccount
   foreach($account in $storage)
   {
       $accountName = $account.StorageAccountName
       Write-Output "======= ASM Storage Account: $accountName ======="
    ➋ $key = Get-AzureStorageKey -StorageAccountName $accountName
    ➌ $context = New-AzureStorageContext -StorageAccountName `
           $accountName -StorageAccountKey $key.Primary
    ➍ $containers = Get-AzureStorageContainer -Context $context
       foreach($container in $containers)
       {
           Write-Output "----- Blobs in Container: $($container.Name) -----"
        ➎ Get-AzureStorageBlob -Context $context -Container $container.Name |
               format-table Name, Length, ContentType, LastModified -auto
       }
       Write-Output "----- Tables -----"
    ➏ Get-AzureStorageTable -Context $context | format-table Name -auto
       Write-Output "----- Queues -----"
    ➐ Get-AzureStorageQueue -Context $context |
           format-table Name, Uri, ApproximateMessageCount -auto
    ➑ $shares = Get-AzureStorageShare -Context $context
       foreach($share in $shares)
       {
           Write-Output "----- Files in Share : $($share.Name) -----"
        ➒ Get-AzureStorageFile -Context $context -ShareName $share.Name |
               format-table Name, @{label='Size';e={$_.Properties.Length}} -auto
       }
       Write-Output ""
   }
   Write-Output ""

   # ARM Storage Accounts
   Write-Output ">>> ARM <<<"
   $storage = Get-AzureRmStorageAccount
   foreach($account in $storage)
   {
       $accountName = $account.StorageAccountName
       Write-Output "======= ARM Storage Account: $accountName ======="
       $key = Get-AzureRmStorageAccountKey -StorageAccountName `
           $accountName -ResourceGroupName $account.ResourceGroupName
       $context = New-AzureStorageContext -StorageAccountName `
           $accountName -StorageAccountKey $key[0].Value
       $containers = Get-AzureStorageContainer -Context $context
       foreach($container in $containers)
       {
           Write-Output "----- Blobs in Container: $($container.Name) -----"
           Get-AzureStorageBlob -Context $context -Container $container.Name |
               format-table Name, Length, ContentType, LastModified -auto
       }
       Write-Output "----- Tables -----"
       Get-AzureStorageTable -Context $context | format-table Name -auto
       Write-Output "----- Queues -----"
       Get-AzureStorageQueue -Context $context |
           format-table Name, Uri, ApproximateMessageCount -auto
       $shares = Get-AzureStorageShare -Context $context
       foreach($share in $shares)
       {
           Write-Output "----- Files in Share : $($share.Name) -----"
           Get-AzureStorageFile -Context $context -ShareName $share.Name |
               format-table Name, @{label='Size';e={$_.Properties.Length}} -auto
       }
       Write-Output ""
   }
```

*列表 4-2：通过 PowerShell 列出存储账户使用情况*

这个脚本分为两部分：第一部分搜索 ASM 存储账户，第二部分搜索 ARM。

我们首先获取订阅中所有 ASM 存储账户的列表 ➊。对于每个账户，我们获取密钥 ➋ 然后为该存储账户创建一个 *上下文* ➌——一个包含存储账户名称和密钥的 PowerShell 对象。以后访问存储账户时，我们可以使用这个上下文。

接下来，脚本开始检查不同的存储类型，如后续章节所讨论的内容，然后对 ARM 存储账户重复该过程。

#### *访问 Blob*

Blob 是 Azure 中最基本的存储形式：它是一个无结构的比特集合，应用程序可以自由使用。Blob 最常用于存储 Azure 虚拟机的虚拟硬盘文件。

在 Azure 中，你会发现三种类型的 Blob：*页面 Blob*、*追加 Blob* 和 *块 Blob*。作为渗透测试者，了解每种 Blob 类型的主要用途可能会很有帮助，这样你可以在不必下载的情况下，对给定的 Blob 内容做出合理的猜测。在我的评估中，我发现下载一个数十 GB 的文件需要几个小时，而最终发现它并不是我预期的内容，这种情况令人非常沮丧。

+   *页面 Blob* 由一组字节组成，称为 *页*。每个页的大小为 512 字节，页面 Blob 本身的最大大小可以达到 1TB。总大小必须在创建 Blob 时设置，这意味着页面 Blob 文件可能会非常大，但其中只有一小部分是数据，其余部分可能是空的。由于页面 Blob 在随机读写操作时非常高效，因此它们是用于硬盘映像的 Blob 类型。

+   *追加块*优化了添加新数据的操作，但禁止修改现有数据。它们最大可达 195GB，特别适合日志文件。如果你正在尝试识别可能与评估相关的其他用户帐户、IP 地址或服务器，那么日志文件可能会非常有用；然而，如果你只是希望修改日志以抹去痕迹，追加块将无法让你做到这一点。

+   *块 Blob* 是默认类型。它由一个或多个字节块组成，块的大小可以最大为 100MB。一个 Blob 中最多可以包含 50,000 个块，块 Blob 可以根据需要增长。这种类型用于所有其他类型的非结构化数据。

Azure 要求用户将所有 Blob 存放在 *容器* 中，容器类似于文件目录，只是它不能嵌套。换句话说，容器可以包含 Blob，但不能包含其他容器。每个存储账户可以有无限数量的容器，每个容器中可以包含任意数量的 Blob。

清单 4-2 中的脚本通过 `Get-AzureStorageContainer` cmdlet 获取 ➍ 所有 Blob 容器的列表，然后使用 `Get-AzureStorageBlob` 为每个容器打印一张表，每个 Blob 占一行 ➎。表格包括 Blob 的名称、大小、数据类型及最后修改日期，如清单 4-3 所示。浏览此列表，查找有用的文件，忽略任何 *.status* 文件和大多数日志文件，而应关注文档、源代码和配置文件。一旦你列出了感兴趣的文件，使用 Azure 存储管理工具开始收集这些文件。

![image](img/00031.jpeg)

*清单 4-3：Blob 命令的输出*

要查看 Blob 的内容，Microsoft Azure 存储资源管理器可能是渗透测试人员的最佳选择。它是免费的，能够正确显示所有类型的 Blob，并支持打开 ASM 和 ARM 存储。或许最重要的是，它提供多种登录选项来访问存储账户，包括以下几种：

+   共享访问签名令牌

+   SQL 连接字符串格式中的存储账户密钥

+   存储账户名称和密钥

+   有权访问订阅的用户的用户名和密码

用户名和密码登录功能特别好，因为它会自动填充用户可以访问的每个订阅的存储账户。你还可以添加多个用户账户，这样你就可以同时查看每个被侵入账户的文件。

在将所有存储账户添加到 Microsoft Azure 存储资源管理器后，展开所需存储账户下的 Blob 存储部分；然后浏览容器列表，选择感兴趣的文件，点击 **下载** 按钮即可下载一份副本，如图 4-11 所示。

![image](img/00032.jpeg)

*图 4-11：从 Microsoft Azure 存储资源管理器下载 Blob*

一旦你获取了文件，一定要检查其中是否有其他凭证。我发现 Azure 存储中存储了令人惊讶数量的机密数据。这使得它成为获取更多系统或服务访问权限的绝佳途径，进一步深入目标环境。

**DEFENDER 提示**

Azure 存储 Blob 不是存储未加密机密的理想场所。由于访问密钥提供了广泛的访问权限和否认权限，机密应该存储在其他地方——或者至少要加密，并且加密密钥不应存储在同一存储帐户中。虽然 Azure Key Vault 并非完全免受攻击，正如我在第七章中所讨论的那样，但它是一个更适合存储机密的选择。

#### *访问表格*

表格提供了在 Azure 中存储表格数据的功能。它们非常适合存储半结构化数据，如 Web 服务日志或网站内容数据库，是 SQL Server 等资源密集型、成本更高的数据库解决方案的良好替代选择。

清单 4-2 调用了 `Get-AzureStorageTable` cmdlet ➏，该命令将返回提供的存储上下文中的所有表格名称，如清单 4-4 所示。你还可以使用 Azure 表格的唯一另一个 cmdlet，`Get-AzureStorageTableStoredAccessPolicy`，它显示表格的任何特殊权限。我很少看到访问策略在使用中，因此通常会跳过它。由于 PowerShell 选项有限，你需要使用独立的工具来访问表格中的数据。

```
----- Tables -----

Name
----
TestTable
TransactionAudits
SchemasTable
```

*清单 4-4：* Get-AzureStorageTable *命令的输出*

选择正确的工具很容易，因为选择不多。主要的工具有 Microsoft Azure Storage Explorer 和 ClumsyLeaf 的 TableXplorer。在这种情况下，尽管 TableXplorer 不是免费软件，我更倾向于使用它，因为它非常快速，提供导出数据的选项，并且提供如图 4-12 所示的查询选项，该选项使用普通的 SQL 语法。这个最后的功能让任何有 SQL 背景的人都能轻松识别数据。Microsoft Azure Storage Explorer 也有查询功能，但它不支持 SQL 语法，且比 TableXplorer 更慢。

在 TableXplorer 中，你可能会看到一些以 `$Metrics` 开头的表格，这些表格在使用 PowerShell 时并不会出现。Azure 会自动生成并使用这些表格来存储关于它们所处的存储帐户的详细信息。名称开头的美元符号（`$`）标记这些表格为隐藏，因此 PowerShell 不会列出它们。

![image](img/00033.jpeg)

*图 4-12：使用 TableXplorer 使用 SQL 语法查询 Azure 存储表格*

这些度量表中的数据追踪了像存储的 blob 总数以及任何具有计费影响的事务，比如数据的添加或删除。这些文件对攻击者通常没有太大价值，除非他们想查找显示他们对存储账户执行的活动的日志条目。不幸的是，你不能删除这些条目，因为度量表是只读的。

#### *访问队列*

Azure 存储队列提供了一个排队事务的地方，可以按顺序处理这些事务，当资源变得可用时。主要是软件开发人员使用队列；毕竟，除了开发人员，很少有人需要担心按顺序处理数据。

从渗透测试的角度来看，我曾经觉得队列很无聊。它们通常是空的，等待工作流入，然后在任务处理完毕后迅速被清空。然而，当我看到队列被用来向服务器发送未签名的命令以供执行时，我改变了看法。许多安全研究人员会花费数周甚至数月的时间来寻找漏洞软件，并开发*远程代码执行*漏洞——让另一台计算机上的进程执行攻击者控制的代码。在这里，它不是一个漏洞，而是一个故意的功能！

尽管那个特定的实例是极端情况，但如果开发人员不小心，队列实际上容易导致这种行为。开发人员通常将它们作为某个自定义应用程序的输入，比如订单履行系统。应用程序的开发人员可能期望队列中只包含来自他们拥有的另一个受信任系统的工作项，比如他们网站上的订单页面，因此开发人员忽视了对工作项字段进行适当验证。这意味着攻击者可以将他们自己的自定义消息注入队列，而处理它们的服务可能不会确认这些消息中的数据是否合理。如果这些字段恰好包含商品的价格、应该发送支付款项的银行账户，或者应该执行的系统命令，那么攻击者就找到了一个优先级非常高的漏洞。

**防御者提示**

如果你使用队列传输机密数据，或者发送必须来自已验证来源的命令，应该使用非对称加密来加密或签名消息，在它们被放入队列之前。然后，接收者可以解密消息或验证其签名，以确保消息是原始的，并且没有被篡改。

队列通常用作后端服务，开发人员通常使用它们来促进应用程序之间的通信，因此它们有良好的 API 支持，而且与队列的交互通常需要编写自定义应用程序。PowerShell 只有两个相关的 cmdlet 用于显示队列信息。一个是 `Get-AzureStorageQueue`，在清单 4-2 ➐中的脚本中，我用它来列出队列及其当前的消息数量，如清单 4-5 所示。第二个是 `Get-AzureStorageQueueStoredAccessPolicy`，它用于查看 SAS 令牌的权限和限制，这些权限和限制很少使用。请注意，没有 cmdlet 用于创建或查看队列中的项目。

```
----- Queues -----
Name      Uri                                               ApproximateMessageCount
----      ---                                               -----------------------
testqueue https://storeasm.queue.core.windows.net/testqueue                       0
```

*清单 4-5：* Get-AzureStorageQueue *命令的输出*

要实际查看和插入队列中的消息，你必须再次使用 Microsoft Azure Storage Explorer。在其界面中，选择一个存储帐户，展开该帐户下的队列列表，然后选择一个队列。这将打开一个视图，显示所有当前排队的消息，并允许你查看消息的内容或插入新消息。我建议在尝试插入自己的消息之前，先检查任何现有消息，以了解有效消息的格式。如果队列为空，尝试找到处理这些消息的应用程序源代码，看看它期望什么。

**警告**

*像其他编程语言中的队列数据结构一样，Azure 队列有两个与查看消息相关的功能。你可以使用* PeekMessage *查看队列中的下一个消息而不更改或移除它。另一方面，* GetMessage *会实际从队列中取出项目，并使其对任何其他使用队列的程序不可见。如果你只是使用 Microsoft Azure Storage Explorer，你不必担心这个问题，但如果你开发了一个自定义应用程序来监视队列，调用* GetMessage *可能会阻止 Azure 处理来自队列的合法请求。所以，在使用这些 API 之前，一定要完全理解它们！*

#### *访问文件*

Azure Storage 最新的服务，称为 Azure Files，是一个基于云的 SMB 文件共享服务。它允许用户创建共享目录并填充文件，就像在本地文件服务器中一样。这对于将依赖 SMB 共享的传统应用程序迁移到 Azure 非常有用。Azure Files 支持从客户端连接，前提是这些客户端支持 SMB 2.1 或 SMB 3.0 协议。

虽然 Azure Files 旨在作为现有企业文件服务器的替代品，但它确实存在一些局限性。首先，任何连接到它的客户端必须能够通过原生的 SMB 端口：TCP 445\。这可能听起来不是大问题，但一些公司网络会在双向阻止 TCP 445 流量，因为文件共享通常被视为内部资源。然而，与传统 Windows 文件服务器的最大区别是没有用户账户和权限。

在普通的 SMB 共享中，用户可以为任意数量的用户或组分配读取、修改和完全控制权限。此外，用户还可以为这些共享中的文件指定文件系统级别的权限，以进一步限制访问。

Azure Files 是不同的。按照设计，它的共享只有一个用户，并且无法配置。共享的用户是 `AZURE\`存储帐户名称，密码是该存储帐户的主密钥，再次突显了保护存储帐户密钥免受未经授权访问的重要性。因此，要完全访问名为 myshare 的 Azure Files 共享，该共享位于名为 mysa 的存储帐户中，您需要在 Windows 命令行中运行以下命令：

```
net use * \\mysa.file.core.windows.net\myshare /u:AZURE\mysa Primary_Key
```

**注意**

*从远程机器到 Azure Files 的连接仅限于支持 SMB 3.0 的 Windows 主机，因为 Linux 和早期版本的 Windows 不支持加密的 SMB 连接。Linux 和旧版 Windows 可以连接到 Azure Files，但仅在它们是运行在 Azure 中的虚拟机，并且处于同一区域时才能连接。*

要枚举共享，使用 `Get-AzureStorageShare` cmdlet，如清单 4-2 中所示的 ➑。对于每个共享，您可以使用 `Get-AzureStorageFile` cmdlet 查看该共享中的文件列表。在 清单 4-2 的 ➒ 处，我将 `Get-AzureStorageFile` 的输出通过管道传递到格式化表命令中——并使用了一些相当难看的参数——以在一行中显示每个文件，并包括文件的名称及其大小（以字节为单位）。因为文件大小被埋藏在每个文件对象的属性中（并被称为“Length”），所以需要使用 PowerShell 的哈希表语法来显示它。`-auto` 开关会自动调整表格的列宽。结果输出显示在清单 4-6 中。

```
----- Files in Share : asmshare -----

Name         Size
----         ----
testfile.txt   33
```

*清单 4-6：文件命令的输出*

除了使用 PowerShell 和 Windows 内置的 SMB 连接功能外，您还可以通过 Microsoft Azure Storage Explorer 查看 Azure Files（见图 4-13）。

![image](img/00034.jpeg)

*图 4-13：使用 Microsoft Azure Storage Explorer 访问 Azure Files*

Microsoft Azure Storage Explorer 并未提供比 PowerShell 和 Windows SMB 客户端组合更多的功能，但它通过使用 Azure 的 API 进行访问，而非直接通过 SMB 连接，从而绕过了 TCP 445 防火墙问题。它还提供了一个名为**连接虚拟机**的便捷按钮，能够自动创建并显示格式正确的 `net use` SMB 命令，帮助你通过 Windows 连接共享。

### 总结

在本章中，我们讨论了 Azure 存储身份验证设计中的一些设计限制，以及攻击者可以用来访问 Azure 存储的不同类型凭证：存储账户密钥、用户名和密码，以及共享访问签名。接下来，我们检查了攻击者通常会找到凭证的地方，如源代码、配置文件，以及存储在各种存储管理工具中的凭证。然后，我们讨论了 Azure 中可用的不同类型存储，包括 Blob、表、队列和文件，并分析了攻击者如何访问它们。利用这些信息，你可以从目标的存储账户中提取所有数据，这些数据通常包括文档、日志文件、硬盘镜像和源代码。

在下一章，我们将探讨 Azure 存储的最大用户：Azure 虚拟机。
