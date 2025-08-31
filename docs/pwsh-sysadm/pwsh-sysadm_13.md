## 第十一章：自动化 Active Directory

![图片](img/common.jpg)

使用 PowerShell 自动化的最佳产品之一就是微软的 Active Directory（AD）。员工不断地进出并在组织中调动。需要一个动态系统来跟踪员工的不断变化，而这正是 AD 的作用。IT 专业人员在 AD 中执行重复且相似的任务，这使得它成为自动化的理想场所。

在本章中，我们将演示如何使用 PowerShell 自动化处理一些涉及 AD 的场景。虽然可以使用 PowerShell 操作许多 AD 对象，但我们只会涉及三种最常见的对象：用户账户、计算机账户和组。这些对象是 AD 管理员日常工作中最常遇到的。

### 前提条件

当你跟随本章的示例进行操作时，我假设你的计算机环境符合一些基本条件。

第一个要求是你正在使用一台已经是 Active Directory 域成员的 Windows 计算机。虽然有方法可以通过使用备用凭据从工作组计算机操作 AD，但这超出了本章的范围。

第二个要求是你将使用与你的计算机属于同一域的环境。复杂的跨域和林信任问题也超出了本章的范围。

最后，你需要确保使用的是具有适当权限的 AD 账户登录到计算机，以便读取、修改和创建常见的 AD 对象，如用户、计算机、组和组织单位。我是在一个属于域管理员组的账户下进行这些练习的——这意味着我对我的域中的所有内容都有控制权限。虽然这不是完全必要的，通常也不推荐在生产环境中使用，但这使我可以在不担心对象权限的情况下演示各种主题，而对象权限超出了本书的范围。

### 安装 ActiveDirectory PowerShell 模块

正如你现在所知道的，使用 PowerShell 完成任务有不止一种方式。同样，当你可以利用现有工具来构建更大、更好的工具时，就没有必要重新发明轮子。在本章中，你将只使用一个模块：`ActiveDirectory`。尽管它有一些不足之处——不太直观的参数、奇怪的过滤语法、异常的错误行为——但它无疑是管理 AD 最全面的模块。

`ActiveDirectory`模块随*远程服务器管理工具*软件包提供。该软件包包含许多工具，而且不幸的是，在撰写本文时，这是获取`ActiveDirectory`模块的唯一方式。在继续阅读本章之前，我建议你下载并安装此软件包。安装后，你将拥有`ActiveDirectory`模块。

为了确认你已经安装了`ActiveDirectory`，可以使用`Get-Module`命令：

```
PS> Get-Module -Name ActiveDirectory -List  
Directory: C:\WINDOWS\system32\WindowsPowerShell\v1.0\Modules

ModuleType  Version  Name             ExportedCommands
----------  -------  ----             ----------------
Manifest    1.0.0.0  ActiveDirectory  {Add-ADCentralAccessPolicyMember,...
```

如果你看到此输出，说明`ActiveDirectory`已经安装。

### 查询和过滤 AD 对象

一旦你确保已经满足所有前提条件并安装了 `ActiveDirectory` 模块，你就可以开始了。

适应新的 PowerShell 模块的最佳方法之一是查找所有以 `Get` 为动词的命令。以 *Get* 开头的命令仅用于读取信息，因此你意外更改某些内容的风险较小。我们将采取这种方法，使用 `ActiveDirectory` 模块，查找与本章中将要操作的对象相关的命令。Listing 11-1 展示了如何仅检索那些以 *Get* 开头并且动词部分包含 *computer* 的 `ActiveDirectory` 命令。

```
PS> Get-Command -Module ActiveDirectory -Verb Get -Noun *computer*

CommandType     Name                               Version    Source
-----------     ----                               -------    ------
Cmdlet          Get-ADComputer                     1.0.0.0    ActiveDirectory
Cmdlet          Get-ADComputerServiceAccount       1.0.0.0    ActiveDirectory

PS> Get-Command -Module ActiveDirectory -Verb Get -Noun *user*

CommandType     Name                               Version    Source
-----------     ----                               -------    ------
Cmdlet          Get-ADUser                         1.0.0.0    ActiveDirectory
Cmdlet          Get-ADUserResultantPasswordPolicy  1.0.0.0    ActiveDirectory

PS> Get-Command -Module ActiveDirectory -Verb Get -Noun *group*

CommandType     Name                               Version    Source
-----------     ----                               -------    ------
Cmdlet          Get-ADAccountAuthorizationGroup    1.0.0.0    ActiveDirectory
Cmdlet          Get-ADGroup                        1.0.0.0    ActiveDirectory
Cmdlet          Get-ADGroupMember                  1.0.0.0    ActiveDirectory
Cmdlet          Get-ADPrincipalGroupMembership     1.0.0.0    ActiveDirectory
```

*Listing 11-1: `ActiveDirectory` 模块 `Get` 命令*

你可以看到一些看起来很有趣的命令。在本章中，你将使用 `Get-ADComputer`、`Get-ADUser`、`Get-ADGroupm` 和 `Get-ADGroupMember` 命令。

#### 过滤对象

你将使用的许多 `Get` AD 命令都有一个名为 `Filter` 的公共参数。`Filter` 类似于 PowerShell 的 `Where-Object` 命令，因为它过滤每个命令返回的内容，但在实现这一任务的方式上有所不同。

`Filter` 参数使用它自己的语法，并且在使用复杂的过滤器时可能会很难理解。要详细了解 `Filter` 参数的语法，你可以运行 `Get-Help` `about`_`ActiveDirectory_Filter`。

在本章中，我们将保持简单，避免使用任何高级过滤。首先，让我们使用 `Filter` 参数和 `Get-ADUser` 命令返回域中的所有用户，如 Listing 11-2 所示。不过要小心：如果你的域中有大量用户账户，可能需要等待一段时间。

```
PS> Get-ADUser -Filter *

DistinguishedName : CN=adam,CN=Users,DC=lab,DC=local
Enabled           : True
GivenName         :
Name              : adam
ObjectClass       : user
ObjectGUID        : 5e53c562-4fd8-4620-950b-aad8fbaa84db
SamAccountName    : adam
SID               : S-1-5-21-930245869-402111599-3553179568-500
Surname           :
UserPrincipalName :
--snip--
```

*Listing 11-2: 查找域中的所有用户账户*

如你所见，`Filter` 参数接受一个字符串值通配符字符 `*`。单独使用时，这个字符告诉（大多数）`Get` 命令返回它们找到的所有内容。尽管这种做法偶尔会有用，但大多数时候你并不想要*所有*可能的对象。不过，如果正确使用，通配符字符是一个强大的工具。

假设你想在 AD 中查找所有以字母 *C* 开头的计算机账户。你可以通过运行 `Get-ADComputer -Filter 'Name -like "C*"'` 来实现，其中 `C*` 代表所有以 *C* 开头的字符。你也可以反过来操作；假设你想查找姓氏以 *son* 结尾的人。你可以运行命令 `Get-ADComputer -Filter 'Name -like "*son"'`。

如果你想找到所有姓*Jones*的用户，可以运行`Get-ADUser -Filter "surName -eq 'Jones'"`；如果你想根据名字和姓氏找到一个用户，可以运行`Get-ADUser -Filter "surName -eq 'Jones' -and givenName -eq 'Joe'"`。`Filter`参数允许你使用各种 PowerShell 操作符，如`like`和`eq`，构建一个仅返回你所需要结果的过滤器。Active Directory 属性以小驼峰命名法存储在 AD 数据库中，因此在过滤器中使用的是这种格式，尽管从技术上讲，这并不是必须的。

另一个用于过滤 AD 对象的有用命令是`Search-ADAccount`命令。该命令内置了对常见过滤场景的支持，比如查找密码已过期的所有用户、查找被锁定的用户，以及查找已启用的计算机。查看`Search-ADAccount` cmdlet 的帮助文档，了解所有参数。

大多数情况下，`Search-ADAccount`语法是自解释的。各种切换参数，包括`PasswordNeverExpires`、`AccountDisabled`和`AccountExpired`，不需要其他参数即可使用。

除了这些高级参数，`Search-ADAccount`还具有一些需要额外输入的参数——例如，指示日期时间属性的年龄，或者如果你需要按特定对象类型（例如，用户或计算机）限制结果。

让我们以`AccountInactive`参数为例。假设你想查找 90 天内没有使用其账户的所有用户。这是`Search-ADAccount`的一个很好的查询。通过使用示例 11-3 中的语法，使用`–`UsersOnly`来过滤对象类型，并使用`–`TimeSpan`来过滤过去 90 天内未活跃的对象，你可以快速找到所有符合要求的用户。

```
PS> Search-ADAccount -AccountInactive -TimeSpan 90.00:00:00 -UsersOnly
```

*示例 11-3：使用`Search-ADAccount`*

`Search-ADAccount` cmdlet 返回的对象类型是`Microsoft.ActiveDirectory.Management.ADUser`。这是与`Get-ADUser`和`Get-ADComputer`等命令返回的对象类型相同的类型。当你使用`Get`命令并感到卡住，不知道该如何编写`Filter`参数的语法时，`Search-ADAccount`可以作为一个很好的快捷方式。

#### 返回单个对象

有时你知道自己要查找的确切 AD 对象，因此根本不需要使用`Filter`。在这种情况下，你可以使用`Identity`参数。

`Identity`是一个灵活的参数，允许你指定使 AD 对象唯一的属性；因此，它只会返回一个对象。每个用户帐户都有一个唯一的属性，叫做`samAccountName`。你可以使用`Filter`参数查找所有具有特定`samAccountName`的用户，语法如下：

```
Get-ADUser -Filter "samAccountName -eq 'jjones'"
```

但是使用`Identity`参数会更加简洁：

```
Get-ADUser -Identity jjones
```

#### 项目 4：查找 30 天内没有更改密码的用户帐户

现在你对如何查询 AD 对象有了基本了解，接下来让我们创建一个小脚本并将这些知识付诸实践。场景是这样的：你在一家公司工作，公司即将实施新的密码过期政策，而你的工作是找到过去 30 天内未更改密码的所有账户。

首先，让我们考虑使用什么命令。你可能首先想到的是本章前面学到的`Search-ADAccount`命令。`Search-ADAccount`有很多用途，用于搜索和筛选各种对象，但你无法创建自定义筛选器。为了更精细地进行搜索，你需要使用`Get-ADUser`命令来构建自己的筛选器。

一旦你知道将使用什么命令，下一步就是确定要筛选的内容。你知道你要筛选出过去 30 天内没有更改密码的账户，但如果你只查找这一点，你会找到比实际需要更多的账户。为什么？如果你不筛选出`Enabled`账户，你可能会得到一些已经不再重要的旧账户（例如离开公司或失去计算机权限的人）。因此，你需要查找那些过去 30 天内未更改密码的启用计算机账户。

让我们从筛选启用的用户账户开始。你可以通过使用`–Filter "Enabled -eq 'True'"`来做到这一点。很简单。下一步是找出当用户的密码设置时存储的属性。

默认情况下，`Get-ADUser`不会返回用户的所有属性。通过使用`Properties`参数，你可以指定希望查看的属性；在这里，你将使用`name`和`passwordlastset`。请注意，有些用户没有`passwordlastset`属性，这是因为他们从未设置过自己的密码。

```
PS> Get-AdUser -Filter * -Properties passwordlastset  | select name,passwordlastset

name           passwordlastset
----           ---------------
adam           2/22/2019 6:45:40 AM
Guest
DefaultAccount
krbtgt         2/22/2019 3:03:32 PM
Non-Priv User  2/22/2019 3:12:38 PM
abertram
abertram2
fbar
--snip--
```

现在你已经有了属性名称，你需要为其构建一个筛选器。记住，你只想筛选那些在过去 30 天内更改了密码的账户。为了找到日期差，你需要两个日期：最早的日期（30 天前）和最新的日期（今天）。你可以通过使用`Get-Date`命令轻松获得今天的日期。然后可以使用`AddDays`方法来计算 30 天前的日期。你会将两个日期存储在变量中，方便以后访问。

```
PS> $today = Get-Date
PS> $30DaysAgo = $today.AddDays(-30)
```

现在你已经有了日期，可以在筛选器中使用它们：

```
PS> Get-ADUser -Filter "passwordlastset -lt '$30DaysAgo'"
```

剩下的就是将你的`Enabled`条件添加到过滤器中。列表 11-4 展示了执行此操作的步骤。

```
$today = Get-Date
$30DaysAgo = $today.AddDays(-30)
Get-ADUser -Filter "Enabled -eq 'True' -and passwordlastset –lt
'$30DaysAgo'"
```

*列表 11-4：查找过去 30 天内未更改密码的启用用户账户*

现在你已经编写了一些代码，用于查找所有在过去 30 天内已设置密码的启用 Active Directory 用户。

### 创建和更改 AD 对象

现在你已经知道如何查找现有的 AD 对象，让我们来学习如何更改和创建它们。本节分为两部分：一部分涉及用户和计算机，另一部分涉及组。

#### 用户和计算机

要更改用户和计算机账户，你将使用 `Set` 命令：`Set-ADUser` 或 `Set-ADComputer`。这些命令可以更改对象的任何属性。通常，你会希望将从 `Get` 命令（如上一节中介绍的命令）获取的对象传递给它们。

作为一个例子，假设一名员工名叫 Jane Jones，她结婚了，你需要更改她用户账户的姓氏。如果你不知道此用户账户的身份属性，你可以在 `Get-ADUser` 上使用 `Filter` 参数来查找它。但首先，你需要发现 AD 是如何存储每个用户的名字和姓氏的。然后，你可以使用这些属性的值传递给 `Filter` 参数。

查找存储在 AD 中的所有可用属性的一种方法是使用一些.NET 代码。通过使用模式对象，你可以找到用户类并枚举其所有属性：

```
$schema =[DirectoryServices.ActiveDirectory.ActiveDirectorySchema]::GetCurrentSchema()
$userClass = $schema.FindClass('user')
$userClass.GetAllProperties().Name
```

通过查看可用属性列表，你会找到 `givenName` 和 `surName` 属性，这些属性可以与 `Get-ADUser` 命令中的 `Filter` 参数一起使用，找到用户账户。接下来，你可以将该对象传递给 `Set-ADUser`，如 列表 11-5 所示。

```
PS> Get-ADUser -Filter "givenName -eq 'Jane' -and surName –eq
'Jones'" | Set-ADUser -Surname 'Smith'
PS> Get-ADUser -Filter "givenName -eq 'Jane' -and surName –eq
'Smith'"

DistinguishedName : CN=jjones,CN=Users,DC=lab,DC=local
Enabled           : False
GivenName         : Jane
Name              : jjones
ObjectClass       : user
ObjectGUID        : fbddbd77-ac35-4664-899c-0683c6ce8457
SamAccountName    : jjones
SID               : S-1-5-21-930245869-402111599-3553179568-3103
Surname           : Smith
UserPrincipalName :
```

*列表 11-5：使用 `Set-ADUser` 更改 AD 对象属性*

你还可以一次更改多个属性。结果发现 Jane 也调动了部门并且得到了晋升，这两个变动都需要更新。没问题，你只需要使用与 AD 属性相匹配的参数：

```
PS> Get-ADUser -Filter "givenName -eq 'Jane' -and surname –eq
'Smith'" | Set-ADUser -Department 'HR' -Title Director
PS> Get-ADUser -Filter "givenName -eq 'Jane' -and surname –eq
'Smith'" -Properties GivenName,SurName,Department,Title

Department        : HR
DistinguishedName : CN=jjones,CN=Users,DC=lab,DC=local
Enabled           : False
GivenName         : Jane
Name              : jjones
ObjectClass       : user
ObjectGUID        : fbddbd77-ac35-4664-899c-0683c6ce8457
SamAccountName    : jjones
SID               : S-1-5-21-930245869-402111599-3553179568-3103
Surname           : Smith
Title             : Director
UserPrincipalName :
```

最后，你可以使用 `New-AD*` 命令创建 AD 对象。创建新的 AD 对象与更改现有对象类似，但在这里你无法使用 `Identity` 参数。创建一个新的 AD 计算机账户就像运行 `New-ADComputer -Name FOO` 一样简单；同样，可以通过使用 `New-ADUser -Name adam` 创建一个 AD 用户。你会发现 `New-AD*` 命令也有与 AD 属性相关的参数，和 `Set-AD*` 命令一样。

#### 组

*组* 比用户和计算机更复杂。你可以把组看作是许多 AD 对象的容器。从这个意义上来说，组就是一堆东西。但同时，它仍然是一个*单一*的容器，意味着像用户和计算机一样，组是一个单一的 AD 对象。这也意味着你可以像查询、创建和更改用户和计算机一样查询、创建和更改组，尽管会有一些细微的差别。

也许你的组织创建了一个新的部门，叫做 AdamBertramLovers，它正在快速扩张，吸引了很多新员工。现在你需要创建一个名为该部门的组。列表 11-6 显示了如何创建这样的组的示例。你使用 `Description` 参数传入一个字符串（组的描述），并使用 `GroupScope` 参数来确保创建的组具有 `DomainLocal` 范围。如果需要的话，你也可以选择 `Global` 或 `Universal`。

```
PS> New-ADGroup -Name 'AdamBertramLovers' 
-Description 'All Adam Bertram lovers in the company' 
-GroupScope DomainLocal
```

*列表 11-6：创建一个 AD 组*

一旦组存在，你可以像修改用户或计算机一样修改它。例如，如果要更改描述，你可以这样做：

```
PS> Get-ADGroup -Identity AdamBertramLovers | 
Set-ADGroup -Description 'More Adam Bertram lovers'
```

当然，组和用户/计算机之间的关键区别是，组可以包含用户和计算机。当一个计算机或用户账户被包含在一个组中时，我们说它是该组的*成员*。但是，要添加和更改组成员，你不能使用之前使用的命令。相反，你需要使用`Add-ADGroupMember`和`Remove-ADGroupMember`。

例如，要将 Jane 添加到我们的组中，可以使用`Add-ADGroupMember`命令。如果 Jane 想要离开该组，可以使用`Remove-ADGroupMember`命令将她移除。当你尝试运行这个命令时，你会发现运行`Remove-ADGroupMember`命令时会弹出一个提示，要求你确认是否要移除该成员：

```
PS> Get-ADGroup -Identity AdamBertramLovers | Add-ADGroupMember Members 'jjones'
PS> Get-ADGroup -Identity AdamBertramLovers | Remove-ADGroupMember-Members 'jjones'

    Confirm
Are you sure you want to perform this action?
Performing the operation "Set" on target
"CN=AdamBertramLovers,CN=Users,DC=lab,DC=local".
[Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  
[?]
Help (default is "Y"): a
```

如果你希望跳过此检查，可以添加`Force`参数，但请注意，得到这个确认提示可能会在某一天拯救你！

#### 项目 5：创建员工入职脚本

让我们把这一切汇总起来，解决另一个实际场景。你的公司雇佣了一名新员工。作为系统管理员，你现在需要执行一系列操作：创建 AD 用户、创建他们的计算机账户，以及将他们添加到特定的组中。你将编写一个脚本来自动化整个过程。

但在你开始这个项目之前——其实在开始任何项目之前——重要的是要弄清楚脚本的功能，并写下一个非正式的定义。对于这个脚本，你需要创建 AD 用户，具体包括：

+   根据名字和姓氏动态创建用户名

+   创建并分配一个随机密码给用户

+   强制用户在登录时更改密码

+   根据给定的部门设置部门属性

+   为用户分配一个内部员工编号

接下来，将用户账户添加到一个与部门名称相同的组中。最后，将用户账户添加到一个与员工所在部门名称相同的组织单位中。

现在，明确了这些需求，我们来构建脚本。完成的脚本将命名为*New-Employee.ps1*，并可以在书籍资源中找到。

你希望这个脚本是可重用的。理想情况下，每当有新员工时，你都可以使用这个脚本。这意味着你需要找到一种智能的方式来处理脚本的输入。通过查看需求，你知道你需要提供一个名字、姓氏、部门和员工编号。列表 11-7 提供了一个脚本大纲，定义了所有参数，并且有一个`try/catch`块来捕获可能遇到的任何终止错误。`#requires`语句设置在顶部，以确保每次运行这个脚本时，它都会检查机器上是否已安装`ActiveDirectory`模块。

```
#requires -Module ActiveDirectory

[CmdletBinding()]
param (
    [Parameter(Mandatory)]
    [string]$FirstName,

    [Parameter(Mandatory)]
    [string]$LastName,

    [Parameter(Mandatory)]
    [string]$Department,

 [Parameter(Mandatory)]
    [int]$EmployeeNumber
)

try {

} catch {
    Write-Error -Message $_.Exception.Message
}
```

*列表 11-7：基础* New-Employee.ps1 *脚本*

现在，你已经创建了基础结构，我们来填充`try`块。

首先，你需要根据我们非正式定义中列出的要求创建一个 AD 用户。你必须*动态创建*一个用户名。实现这一点的方法有很多：一些组织喜欢用户名是名字的首字母加姓氏，有些喜欢名字和姓氏组合，还有些完全有不同的做法。假设你的公司使用名字首字母加姓氏。如果该用户名已被占用，则会继续从名字中添加下一个字符，直到找到一个唯一的用户名。

我们先处理基本情况。你将对每个字符串对象使用内置的`Substring`方法来获取名字的首字母。然后，你将姓氏与首字母连接在一起。你将通过*字符串格式化*来完成这一步，字符串格式化允许你在字符串中定义多个表达式的占位符，并在运行时用值替换这些占位符，示例如下：

```
$userName = '{0}{1}' -f $FirstName.Substring(0, 1), $LastName
```

创建初始用户名后，你需要查询 AD，使用`Get-ADUser`检查该用户名是否已被占用。

```
Get-ADUser -Filter "samAccountName -eq '$userName'"
```

如果该命令返回任何内容，则用户名已被占用，你需要尝试下一个用户名。这意味着你需要找到一种方法来动态生成新用户名，并始终为用户名已被占用的情况做好准备。检查不同用户名的一个好方法是使用`while`循环，条件是你之前对`Get-ADUser`的调用结果。但你还需要另一个条件来应对如果名字中的字母用完的情况。你不希望循环永远运行下去，所以你会添加另一个条件，`$userName –notlike "$FirstName*" `，来停止循环。

`while` 条件看起来像这样：

```
(Get-ADUser -Filter "samAccountName -eq '$userName'") –and
($userName -notlike "$FirstName*")
```

创建了`while`条件后，你可以完成循环的其余部分：

```
$i = 2
while ((Get-ADUser -Filter "samAccountName -eq '$userName'") –and
($userName -notlike "$FirstName*")) {
    Write-Warning -Message "The username [$($userName)] already exists. Trying another..."
    $userName = '{0}{1}' -f $FirstName.Substring(0, $i), $LastName
    Start-Sleep -Seconds 1
    $i++
}
```

对于循环的每次迭代，你通过获取从 0 到`i`的子字符串，将第一个名字中的一个额外字符添加到建议的用户名中，其中`$i`是一个计数器变量，它从 2（字符串中的下一个位置）开始，并在每次循环运行时增加。到这个`while`循环结束时，它要么找到了一个唯一的用户名，要么已经耗尽了所有选项。

如果没有找到现有用户名，你就可以创建你想要的用户名。如果找到了一个用户名，你还需要检查其他事项。你需要检查你将用户账户放入的*组织单位*（OU）和组是否存在：

```
if (-not ($ou = Get-ADOrganizationalUnit -Filter "Name –eq '$Department'")) {
    throw "The Active Directory OU for department [$($Department)] could not be found."
} elseif (-not (Get-ADGroup -Filter "Name -eq '$Department'")) {
    throw "The group [$($Department)] does not exist."
}
```

一旦完成所有检查，你需要创建用户账户。再一次，你需要参考我们的非正式定义：*创建并分配一个* *随机密码*给用户。你希望每次运行这个脚本时生成一个随机密码。生成安全密码的一种简单方法是使用`System.Web.Security.Membership`对象上的`GeneratePassword`静态方法，如下所示：

```
Add-Type -AssemblyName 'System.Web'
$password = [System.Web.Security.Membership]::GeneratePassword(
    (Get-Random Minimum 20 -Maximum 32), 3)
$secPw = ConvertTo-SecureString -String $password -AsPlainText -Force
```

我选择生成一个至少 20 个字符、最多 32 个字符的密码，但这是完全可以配置的。如果需要，你还可以通过运行`Get-ADDefaultDomainPasswordPolicy | Select-object -expand minPasswordLength`来查看 AD 的最低密码要求。这个方法甚至允许你指定新密码的长度和复杂度。

现在你已经将密码作为安全字符串获取，你拥有了根据我之前列出的要求创建用户所需的所有参数值。

```
$newUserParams = @{
    GivenName             = $FirstName
    EmployeeNumber        = $EmployeeNumber
    Surname               = $LastName
    Name                  = $userName
    AccountPassword       = $secPw
    ChangePasswordAtLogon = $true
    Enabled               = $true
    Department            = $Department
    Path                  = $ou.DistinguishedName
    Confirm               = $false
}
New-ADUser @newUserParams
```

在你创建用户之后，剩下的就是将他们添加到部门组中，你可以使用一个简单的`Add-ADGroupMember`命令来完成：

```
Add-ADGroupMember -Identity $Department -Members $userName
```

一定要查看书中资源中的*New-Employee.ps1*脚本，以获取此脚本的完整实现版本。

### 从其他数据源同步

活动目录，特别是在大企业中使用时，可能包含数百万个对象，这些对象每天都被几十个人创建和修改。随着所有这些活动和输入，问题肯定会出现。你将遇到的最大问题之一是保持 AD 数据库与组织的其他部分同步。

公司的 AD 应该与公司组织结构相匹配。这可能意味着每个部门都有自己的 AD 组，每个物理办公室有自己的 OU，等等。无论如何，作为系统管理员，我们的困难任务是确保 AD 始终与组织的其他部分保持同步。这是 PowerShell 的一个重要任务。

使用 PowerShell，你可以将 AD 与几乎任何其他信息源“链接”，这意味着你可以让 PowerShell 不断地读取外部数据源，并根据需要对 AD 进行适当的修改，以创建一个同步过程。

当触发该同步过程时，通常包括以下六个步骤：

1.  查询外部数据源（SQL 数据库、CSV 文件等）。

1.  从 AD 中检索对象。

1.  在源中查找每个对象，AD 具有一个唯一的属性来进行匹配。这个属性通常称为*ID*。ID 可以是员工 ID，甚至是用户名。唯一重要的是该属性是唯一的。如果找不到匹配项，可以根据源选择性地在 AD 中创建或删除该对象。

1.  查找一个匹配的单一对象。

1.  将所有外部数据源映射到 AD 对象属性。

1.  修改现有的 AD 对象或创建新的对象。

你将在下一节中实施这个计划。

#### 项目 6：创建同步脚本

在本节中，你将学习如何构建一个脚本，将员工信息从 CSV 文件同步到 AD。为此，你需要使用你在第十章中学到的一些命令，以及你在本章之前课程中刚学到的命令。在我们开始之前，建议你浏览一下书中资源中的*Employees.csv*和*Invoke-AdCsvSync.ps1*，并熟悉项目文件。

构建一个优秀的 AD 同步工具的关键是相似性。我的意思并不是说数据源应该是相同的——因为从技术上讲，它们永远不会相同——而是你需要创建一个脚本，能够以相同的方式查询每个数据存储，并且让每个数据存储返回相同类型的对象。这个难点出现在当你有两个使用不同模式的数据源时。在这种情况下，你可能需要通过将一个字段名映射到另一个字段名来开始做一些转换（正如你将在本章后面所做的那样）。

请考虑以下情况：你已经知道 AD 中每个用户账户都有一些常见的属性——例如名字、姓氏和部门，我们称之为*属性模式*。然而，可能源数据存储中用于同步的属性永远不会完全相同。即使它们有相同的属性，它们的名称也可能不同。为了解决这个问题，你必须在两个数据存储之间建立映射。

#### 映射数据源属性

创建这种映射的一个简单有效方法是使用哈希表，其中键是第一个数据存储中的属性名称，值是第二个数据存储中的属性名称。为了查看这一过程的实际操作，假设你在一家名为 Acme 的公司工作。Acme 想要将员工记录从 CSV 文件同步到 AD。具体来说，他们想要同步*Employees.csv*，你可以在本书的资源中找到该文件，或者在这里找到：

```
"fname","lname","dept"
"Adam","Bertram","IT"
"Barack","Obama","Executive Office"
"Miranda","Bertram","Executive Office"
"Michelle","Obama","Executive Office"
```

既然你知道 CSV 的表头和 AD 中的属性名称，你可以构建一个映射哈希表，将 CSV 字段的值作为键，AD 属性名称作为值：

```
$syncFieldMap = @{   
    fname = 'GivenName'
    lname = 'Surname'   
    dept = 'Department'
}
```

这将处理两个数据存储模式之间的转换。但你还需要为每个员工创建一个唯一的 ID。到目前为止，CSV 的每一行中没有可以匹配到 AD 对象的唯一 ID。例如，你可能会遇到多个名字叫 Adam 的人，多个 IT 部门的员工，或者多个姓 Bertram 的人。这意味着你必须生成自己的唯一 ID。为了简化问题，假设没有两个员工的名字和姓氏相同。否则，创建 ID 的方式可能会依赖于你自己组织的模式。在此假设下，你可以简单地将每个数据存储的名字和姓氏字段连接起来，创建一个临时的唯一 ID。

你将在另一个哈希表中表示这个唯一的 ID。虽然你还没有处理连接操作，但你已经设置好了执行此操作的基础设施：

```
$fieldMatchIds = @{
    AD = @('givenName','surName')
    CSV = @('fname','lname')
}
```

现在你已经创建了一种将不同字段映射在一起的方法，可以将该代码整合到几个函数中，以“强制”两个数据存储返回相同的属性，从而实现“苹果对苹果”的比较。

#### 创建返回相似属性的函数

现在你有了哈希表，接下来需要翻译字段名称并构建唯一 ID。你可以创建一个函数来查询我们的 CSV 文件，并输出 AD 理解的属性，以及你可以用来匹配两个数据存储的属性。为此，你将创建一个名为 `Get-AcmeEmployeeFromCsv` 的函数，代码见列表 11-8。我将 `CsvFilePath` 参数的值设置为 *C:\Employees.csv*，假设我们的 CSV 文件位于该位置：

```
function Get-AcmeEmployeeFromCsv
{    
[CmdletBinding()]
    param (
        [Parameter()]
        [string]$CsvFilePath = 'C:\Employees.csv',
        [Parameter(Mandatory)]
        [hashtable]$SyncFieldMap,
 [Parameter(Mandatory)]
        [hashtable]$FieldMatchIds
    )
    try {
        ## Read each key/value pair in $SyncFieldMap to create calculated
        ## fields which we can pass to Select-Object later. This allows us to
        ## return property names that match Active Directory attributes rather
        ## than what's in the CSV file.
     ❶ $properties = $SyncFieldMap.GetEnumerator() | ForEach-Object {
            @{
                Name = $_.Value
                Expression = [scriptblock]::Create("`$_.$($_.Key)")
            }
        }
        ## Create the unique ID based on the unique fields defined in
        ## $FieldMatchIds
     ❷ $uniqueIdProperty = '"{0}{1}" -f '
        $uniqueIdProperty = $uniqueIdProperty += 
        ($FieldMatchIds.CSV | ForEach-Object { '$_.{0}' -f $_ }) – join ','
        $properties += @{
            Name = 'UniqueID'
            Expression = [scriptblock]::Create($uniqueIdProperty)
        }
        ## Read the CSV file and "transform" the CSV fields to AD attributes
        ## so we can compare apples to apples
     ❸ Import-Csv -Path $CsvFilePath | Select-Object – Property $properties
    } catch {
        Write-Error -Message $_.Exception.Message
    }
}
```

*列表 11-8：`Get-AcmeEmployeeFromCsv` 函数*

该函数的工作流程分为三个主要步骤：首先，将 CSV 的属性映射到 AD 属性 ❶；接着，创建一个唯一 ID 并将其作为属性 ❷；最后，读取 CSV，并使用 `Select-Object` 和计算属性返回你需要的属性 ❸。

如下代码所示，你可以将 `$syncFieldMap` 哈希表和 `$fieldMatchIds` 哈希表传递给你新的 `Get-AcmeEmployeeFromCsv` 函数，你可以用它来返回与 Active Directory 属性以及你新创建的唯一 ID 同步的属性名称：

```
PS> Get-AcmeEmployeeFromCsv -SyncFieldMap $syncFieldMap 
-FieldMatchIds $fieldMatchIds

GivenName Department       Surname UniqueID
--------- ----------       ------- --------
Adam      IT               Bertram AdamBertram
Barack    Executive Office Obama   BarackObama
Miranda   Executive Office Bertram MirandaBertram
Michelle  Executive Office Obama   MichelleObama
```

现在，你需要构建一个从 AD 查询的函数。幸运的是，这一次你不需要转换任何属性名称，因为 AD 的属性名称就是你的公共集合。在这个函数中，你只需要调用 `Get-ADUser`，并确保返回你需要的属性，代码见列表 11-9。

```
function Get-AcmeEmployeeFromAD
{
    [CmdletBinding()]
    param (
        [Parameter(Mandatory)]
        [hashtable]$SyncFieldMap,

        [Parameter(Mandatory)]
        [hashtable]$FieldMatchIds
    )

    try {
        $uniqueIdProperty = '"{0}{1}" -f '
        $uniqueIdProperty += ($FieldMatchIds.AD | ForEach Object { '$_.{0}' -f $_ }) -join ','

        $uniqueIdProperty = @{ ❶
            Name = 'UniqueID'
            Expression = [scriptblock]::Create($uniqueIdProperty)
        }

        Get-ADUser -Filter * -Properties @($SyncFieldMap.Values) | Select-Object *,$uniqueIdProperty ❷

    } catch {
        Write-Error -Message $_.Exception.Message
    }
}
```

*列表 11-9：`Get-AcmeEmployeeFromAD` 函数*

再次，我将重点介绍这段代码的主要步骤：首先，创建唯一 ID 来执行匹配 ❶；然后，查询 AD 用户并仅返回字段映射哈希表中的值，同时返回你之前创建的唯一 ID ❷。

当你运行这段代码时，你会看到它返回具有适当属性和唯一 ID 属性的 AD 用户帐户。

#### 在 Active Directory 中查找匹配项

现在你有了两个类似的函数，可以从数据存储中提取信息，并返回相同的属性名称。接下来的步骤是查找 CSV 和 AD 之间的所有匹配项。为了简化这个过程，你将使用列表 11-10 中的代码，创建另一个名为 `Find-UserMatch` 的函数，该函数将执行这两个函数，并收集这两个数据集。一旦获取了数据，它将查找 `UniqueID` 字段上的匹配项。

```
function Find-UserMatch {
    [OutputType()]
    [CmdletBinding()]
    param
    (
        [Parameter(Mandatory)]
        [hashtable]$SyncFieldMap,

        [Parameter(Mandatory)]
        [hashtable]$FieldMatchIds 
    )
    $adusers = Get-AcmeEmployeeFromAD -SyncFieldMap $SyncFieldMap -FieldMatchIds $FieldMatchIds ❶

    $csvUsers = Get-AcmeEmployeeFromCSV -SyncFieldMap $SyncFieldMap -FieldMatchIds $FieldMatchIds ❷

    $adUsers.foreach({
        $adUniqueId = $_.UniqueID
        if ($adUniqueId) { ❸
            $output = @{
                CSVProperties = 'NoMatch'
                ADSamAccountName = $_.samAccountName
            }
            if ($adUniqueId -in $csvUsers.UniqueId) { ❹
                $output.CSVProperties = ($csvUsers.Where({$_.UniqueId -eq $adUniqueId})) ❺
            }
            [pscustomobject]$output
        }
    })
}
```

*列表 11-10：查找用户匹配项*

让我们逐步分析这段代码。首先，从 AD 获取用户列表 ❶；然后，从我们的 CSV 获取用户列表 ❷。对于每个来自 AD 的用户，检查 `UniqueID` 属性是否已被填充 ❸。如果已填充，检查 CSV 和 AD 用户之间是否找到了匹配 ❹，如果找到了，在我们的自定义对象中创建一个名为 `CSVProperties` 的属性，包含与匹配用户相关的所有属性 ❺。

如果找到匹配项，函数将返回 AD 用户的 `samAccountName` 和所有 CSV 属性；否则，它将返回 `NoMatch`。返回 `samAccountName` 会给你一个 AD 中的唯一 ID，这样你以后就可以查找这个用户。

```
PS> Find-UserMatch -SyncFieldMap $syncFieldMap -FieldMatchIds $fieldMatchIds

ADSamAccountName CSVProperties
---------------- -------------
user             NoMatch
abertram         {@{GivenName=Adam; Department=IT;
                 Surname=Bertram; UniqueID=AdamBertram}}
dbddar           NoMatch
jjones           NoMatch
BSmith           NoMatch
```

到目前为止，你已经有了一个功能，可以在 AD 数据和 CSV 数据之间找到 1:1 匹配。你现在准备好开始进行大量的 AD 更改了，这虽然令人兴奋，但也有些吓人！

#### 更改 Active Directory 属性

现在你有了一种方法，可以找出哪个 CSV 行对应哪个 AD 用户帐户。你可以使用 `Find-UserMatch` 函数通过用户的唯一 ID 查找 AD 用户，然后更新其 AD 信息，使其与 CSV 中的数据匹配，如 清单 11-11 所示。

```
## Find all of the CSV <--> AD user account matches
$positiveMatches = (Find-UserMatch -SyncFieldMap $syncFieldMap -FieldMatchIds $fieldMatchIds).where({ $_.CSVProperties -ne 'NoMatch' })
foreach ($positiveMatch in $positiveMatches) {
    ## Create the splatting parameters for Set-ADUser using
    ## the identity of the AD samAccountName
    $setADUserParams = @{
        Identity = $positiveMatch.ADSamAccountName
    }

    ## Read each property value that was in the CSV file
    $positiveMatch.CSVProperties.foreach({
        ## Add a parameter to Set-ADUser for all of the CSV
        ## properties excluding UniqueId
        ## Find all of the properties on the CSV row that are NOT UniqueId
        $_.PSObject.Properties.where({ $_.Name –ne 'UniqueID' }).foreach({
            $setADUserParams[$_.Name] = $_.Value
        })
    })
    Set-ADUser @setADUserParams
}
```

*清单 11-11：将 CSV 同步到 AD 属性*

创建一个健壮且灵活的 AD 同步脚本需要做很多工作。在这个过程中，你会遇到很多小细节和问题，尤其是当你构建更复杂的脚本时。

我们才刚刚触及与 PowerShell 同步的表面。如果你想看看你能通过这个概念做多少事情，可以查看 PowerShell Gallery 中的 `PSADSync` 模块（`Find-Module PSADSync`）。这个模块是专为我们这里的任务而构建的，但它可以处理更复杂的情况。如果在这个练习中你感觉有些迷茫，我强烈建议你重新阅读代码——多少遍都没关系。学习 PowerShell 的唯一真正方法就是实验！运行代码，看看它出错，自己修复，再试一次。

### 小结

在本章中，你熟悉了 `ActiveDirectory` PowerShell 模块。你学习了如何在 AD 中创建和更新用户、计算机和组。通过几个实际的例子，你看到了如何使用 PowerShell 自动化繁琐的 Active Directory 工作。

在接下来的两章中，我们将进入云端！我们将继续自动化所有任务，并看看如何在 Microsoft Azure 和 Amazon Web Services（AWS）中自动化一些常见任务。
