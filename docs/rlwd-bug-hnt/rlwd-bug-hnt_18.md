## 第十八章：应用逻辑和配置漏洞**

![Image](img/common.jpg)

与本书中之前介绍的漏洞不同，后者依赖于提交恶意输入的能力，应用逻辑和配置漏洞则利用了开发者的错误。*应用逻辑* 漏洞发生在开发者在编码逻辑时犯错，攻击者可以利用这个漏洞执行一些未预期的操作。*配置* 漏洞发生在开发者错误配置了工具、框架、第三方服务或其他程序或代码，从而导致了漏洞。

这两个漏洞都涉及到利用开发者在编写代码或配置网站时所做的决策中的错误。这类漏洞的影响通常是攻击者能够未经授权访问某些资源或执行某些操作。但由于这些漏洞源自编码和配置决策，因此它们可能很难描述。理解这些漏洞的最好方式是通过一个例子来分析。

2012 年 3 月，Egor Homakov 向 Ruby on Rails 团队报告了其默认配置存在不安全的问题。当时，当开发者安装一个新的 Rails 网站时，Rails 默认生成的代码会接受所有提交给控制器动作的参数，用于创建或更新数据库记录。换句话说，默认安装将允许任何人发送 HTTP 请求来更新任何用户对象的用户 ID、用户名、密码和创建日期参数，无论开发者是否希望这些参数是可更新的。这个例子通常被称为 *批量赋值* 漏洞，因为所有参数都可以用来赋值给对象记录。

这种行为在 Rails 社区内是众所周知的，但很少有人意识到它所带来的风险。Rails 核心开发者认为，应该由 web 开发者负责关闭这一安全漏洞，并定义站点接受哪些参数来创建和更新记录。你可以阅读一些相关讨论，链接请见 *[`github.com/rails/rails/issues/5228/`](https://github.com/rails/rails/issues/5228/)*。

Rails 核心开发者不同意 Homakov 的评估，因此 Homakov 在 GitHub（一个使用 Rails 开发的大型网站）上利用了这个漏洞。他猜测到一个可访问的参数，该参数被用来更新 GitHub 问题的创建日期。他将创建日期参数包含在 HTTP 请求中，并提交了一个创建日期设定在未来几年的问题。这本不该是 GitHub 用户能够做到的事情。他还更新了 GitHub 的 SSH 访问密钥，以获取对官方 GitHub 代码库的访问权限——这是一个严重的漏洞。

作为回应，Rails 社区重新考虑了其立场，并开始要求开发者列出白名单参数。现在，默认配置不会接受任何参数，除非开发者将其标记为安全。

GitHub 的例子结合了应用程序逻辑和配置漏洞。GitHub 开发人员本应添加安全防护措施，但由于他们使用了默认配置，结果导致了一个漏洞。

应用程序逻辑和配置漏洞可能比本书前面涉及的漏洞更难发现（并不是说其他漏洞就容易）。这是因为它们依赖于对编码和配置决策的创造性思考。你对各种框架的内部工作原理了解得越多，你就越容易发现这类漏洞。例如，Homakov 知道该网站是用 Rails 构建的，并且了解 Rails 默认如何处理用户输入。在其他例子中，我会展示报告人如何调用直接 API 请求，扫描成千上万的 IP 寻找配置错误的服务器，并发现本不应公开访问的功能。这些漏洞需要有一定的 Web 框架背景知识和调查技能，因此我会重点介绍那些能帮助你培养这些知识的报告，而不是那些奖金高的报告。

### 绕过 Shopify 管理员权限

**难度：** 低

**URL:** *<shop>.myshopify.com/admin/mobile_devices.json*

**来源：** *[`hackerone.com/reports/100938/`](https://hackerone.com/reports/100938/)*

**报告日期：** 2015 年 11 月 22 日

**奖金支付：** $500

和 GitHub 一样，Shopify 是使用 Ruby on Rails 框架构建的。Rails 很受欢迎，因为当你用它开发一个网站时，框架会处理许多常见和重复的任务，比如解析参数、路由请求、服务文件等等。但 Rails 默认并不提供权限管理。开发人员必须编写自己的权限管理代码，或者安装一个具有该功能的第三方 gem（*gems*是 Ruby 的库）。因此，在进行 Rails 应用程序的黑客攻击时，测试用户权限总是一个好主意：你可能会发现应用程序逻辑漏洞，就像寻找 IDOR 漏洞时一样。

在这个案例中，报告人 rms 注意到 Shopify 定义了一个名为 Settings 的用户权限。该权限允许管理员通过 HTML 表单在网站上提交订单时将电话号码添加到应用程序中。没有这个权限的用户，在用户界面（UI）中不会显示提交电话号码的字段。

通过使用 Burp 作为代理记录发送到 Shopify 的 HTTP 请求，rms 找到了 HTML 表单请求发送的端点。接下来，rms 登录了一个被分配了 Settings 权限的账户，添加了一个电话号码，然后删除了该号码。Burp 的历史选项卡记录了添加电话号码的 HTTP 请求，该请求发送到了*/admin/mobile_numbers.json*端点。然后 rms 从该用户账户中移除了 Settings 权限。此时，用户账户应该不能再添加电话号码了。

使用 Burp Repeater 工具，rms 绕过了 HTML 表单，并在仍然登录账户且没有设置权限的情况下，向 */admin/mobile_number.json* 发送了相同的 HTTP 请求。响应显示成功，且在 Shopify 上下单测试时确认通知已发送到手机号码。设置权限仅移除了前端 UI 元素，用户无法在界面上输入电话号码。但该权限并未阻止没有权限的用户通过站点后端提交电话号码。

#### *总结要点*

在处理 Rails 应用时，一定要测试所有用户权限，因为 Rails 默认并不处理这些功能。开发人员必须自行实现用户权限，所以他们可能会忘记添加权限检查。此外，代理流量总是一个好主意。这样，你可以轻松识别端点并重放可能无法通过网站 UI 获得的 HTTP 请求。

### 绕过 Twitter 账户保护

**难度：** 简单

**网址：** *[`twitter.com`](https://twitter.com)*

**来源：** 无

**报告日期：** 2016 年 10 月

**悬赏金额：** $560

在测试时，一定要考虑应用程序的网页版本与移动版本之间的差异。两者之间可能存在应用逻辑上的不同。当开发人员没有充分考虑这些差异时，就可能会产生漏洞，这就是本报告中所发生的情况。

2016 年秋季，Aaron Ullger 发现，当他第一次从一个未被识别的 IP 地址和浏览器登录 Twitter 时，Twitter 网站要求提供额外的信息才能进行认证。Twitter 要求的信息通常是与账户关联的电子邮件或电话号码。这个安全功能旨在确保，如果你的账户登录信息被泄露，攻击者在没有额外信息的情况下无法访问账户。

但在他的测试中，Ullger 使用手机连接到 VPN，给设备分配了一个新的 IP 地址。当从未被识别的 IP 地址和浏览器登录时，他本应被要求提供额外信息，但他在手机上从未被要求提供这些信息。这意味着，如果攻击者劫持了他的账户，他们可以通过移动应用绕过额外的安全检查。此时，攻击者还可以在应用中查看用户的电子邮件地址和电话号码，从而通过网站登录。

作为回应，Twitter 验证并修复了该问题，向 Ullger 支付了 $560。

#### *总结要点*

在通过不同方式访问应用程序时，考虑安全相关的行为是否在各个平台之间保持一致。在本例中，Ullger 只测试了应用程序的浏览器和移动版本。但其他网站可能还会使用第三方应用或 API 端点。

### HackerOne 信号操控

**难度：** 低

**URL：** *hackerone.com/reports/<X>*

**来源：** *[`hackerone.com/reports/106305`](https://hackerone.com/reports/106305)*

**报告日期：** 2015 年 12 月 21 日

**支付的悬赏金：** $500

在开发网站时，程序员通常会测试他们实现的新功能。但他们可能忽略测试罕见类型的输入，或者新功能与网站其他部分的交互方式。测试时，重点关注这些领域，尤其是边缘案例，这些是开发人员可能会无意中引入应用程序逻辑漏洞的简单方式。

2015 年底，HackerOne 在其平台上引入了一项新功能，名为 Signal，用于显示黑客根据已解决报告的平均声誉。例如，关闭为垃圾邮件的报告会获得 -10 声誉，不适用的报告获得 -5，信息性报告得 0，已解决的报告得 7。你的 Signal 越接近 7，表现就越好。

在这个案例中，报告者 Ashish Padelkar 发现，用户可以通过自闭报告来操控这一统计数据。自闭报告是一个独立的功能，允许黑客在犯错时撤回他们的报告，并将该报告的声誉设为 0。Padekar 意识到 HackerOne 正在使用自闭报告中的 0 来计算 Signal。因此，任何拥有负面 Signal 的人都可以通过自闭报告来提高他们的平均声誉。

因此，HackerOne 从 Signal 计算中移除了自闭报告，并向 Padekar 发放了 $500 的悬赏金。

#### *要点*

留意新网站功能：它代表着测试新代码的机会，并且可能会在现有功能中引发错误。在这个例子中，自闭报告与新功能 Signal 的交互产生了意想不到的后果。

### HackerOne 不正确的 S3 存储桶权限

**难度：** 中等

**URL:** *[REDACTED].s3.amazonaws.com*

**来源：** *[`hackerone.com/reports/128088/`](https://hackerone.com/reports/128088/)*

**报告日期：** 2016 年 4 月 3 日

**支付的悬赏金：** $2,500

很容易假设在你开始测试之前，应用程序中的每个漏洞都已经被发现了。但不要高估一个站点的安全性，也不要以为其他黑客已经测试过了。我在 HackerOne 上测试一个应用配置漏洞时，就不得不克服这种心态。

我注意到 Shopify 已公开了关于配置错误的 Amazon Simple Store Services (S3) 存储桶的报告，于是决定看看我能否找到类似的漏洞。S3 是 Amazon Web Services (AWS) 提供的一项文件管理服务，许多平台使用它来存储和提供静态内容，如图像。像所有 AWS 服务一样，S3 拥有复杂的权限，容易配置错误。在此报告发布时，权限包括读取、写入和读写权限。写入和读写权限意味着任何拥有 AWS 账户的人都可以修改文件，即使该文件存储在私有存储桶中。

在 HackerOne 网站上寻找漏洞时，我意识到平台从一个名为 `hackerone-profile-photos` 的 S3 桶中提供用户图片。桶名给了我一个线索，表明 HackerOne 使用了某种命名规范来命名桶。为了深入了解如何妥协 S3 桶，我开始查看之前类似漏洞的报告。不幸的是，我找到的关于配置错误的 S3 桶的报告并没有说明报告者是如何发现这些桶的，也没有说明他们是如何验证漏洞的。于是，我转向网络查找信息，发现了两篇博客文章：*[`community.rapid7.com/community/infosec/blog/2013/03/27/1951-open-s3-buckets/`](https://community.rapid7.com/community/infosec/blog/2013/03/27/1951-open-s3-buckets/)* 和 *[`digi.ninja/projects/bucket_finder.php/`](https://digi.ninja/projects/bucket_finder.php/)*。

Rapid7 文章详细介绍了他们如何使用 *模糊测试* 来发现公开可读的 S3 桶。为此，团队收集了一个有效的 S3 桶名列表，并生成了一个常见变体的词汇表，如 `backup`、`images`、`files`、`media` 等等。两个列表为他们提供了数千个桶名组合，团队使用 AWS 命令行工具测试访问这些桶。第二篇博客文章中包含了一个名为 *bucket_finder* 的脚本，它接受一个可能的桶名词汇表，并检查列表中的每个桶是否存在。如果桶确实存在，脚本会尝试使用 AWS 命令行工具读取其内容。

我为 HackerOne 创建了一个潜在的桶名列表，如 `hackerone`、`hackerone.marketing`、`hackerone.attachments`、`hackerone.users`、`hackerone.files` 等等。我将这个列表提供给 *bucket_finder* 工具，它找到了一些桶，但没有一个是公开可读的。然而，我注意到脚本没有测试它们是否是公开可写的。为了测试这一点，我创建并尝试将一个文本文件复制到我找到的第一个桶中，使用命令 `aws s3 mv test.txt s3://hackerone.marketing`。结果如下：

```
move failed: ./test.txt to s3://hackerone.marketing/test.txt A client error

(AccessDenied) occurred when calling the PutObject operation: Access Denied
```

尝试下一个桶 `aws s3 mv test.txt s3://hackerone.files`，结果是：

```
move: ./test.txt to s3://hackerone.files/test.txt
```

成功！接下来，我尝试使用命令 `aws s3 rm s3://hackerone.files/test.txt` 删除文件，并再次成功。

我能够从一个桶中写入和删除文件。理论上，攻击者可以将一个恶意文件放入这个桶中，供 HackerOne 的工作人员访问。当我在写报告时，我意识到我无法确认 HackerOne 是否拥有这个桶，因为亚马逊允许用户注册任何桶名。我不确定是否应该在没有确认所有权的情况下报告，但我想，反正试试看吧。几小时后，HackerOne 确认了报告并修复了漏洞，还发现了其他配置错误的桶。值得称赞的是，HackerOne 在发放奖金时，考虑到了额外的桶，并增加了我的奖励。

#### *要点*

HackerOne 是一个了不起的团队：拥有黑客思维的开发者们了解常见的漏洞并能够识别。但即使是最优秀的开发者也可能犯错。不要因为害怕而回避测试应用程序或功能。在测试过程中，重点关注那些容易配置错误的第三方工具。此外，如果你找到有关新概念的报告或公开报告，试着理解那些报告者是如何发现漏洞的。在这种情况下，关键是研究人们是如何发现并利用 S3 配置错误的。

### 绕过 GitLab 双因素认证

**难度：** 中等

**URL:** 不适用

**来源：** *[`hackerone.com/reports/128085/`](https://hackerone.com/reports/128085/)*

**报告日期：** 2016 年 4 月 3 日

**奖励支付：** 不适用

*双因素认证（2FA）* 是一种安全功能，它在网站登录过程中增加了第二个步骤。传统上，用户登录网站时只需输入用户名和密码进行身份验证。而在启用 2FA 的情况下，网站要求在输入密码之外进行额外的身份验证步骤。通常，网站会通过电子邮件、短信或身份验证器应用程序发送授权码，用户在提交用户名和密码后必须输入该授权码。这些系统可能很难正确实现，因此是应用逻辑漏洞测试的良好候选对象。

2016 年 4 月 3 日，Jobert Abma 在 GitLab 中发现了一个漏洞。该漏洞允许攻击者在启用 2FA 时，不知道目标用户的密码也能登录目标账户。Abma 注意到，在用户输入用户名和密码后，网站会向用户发送一个验证码。用户提交验证码后，系统会发出以下`POST`请求：

```
   POST /users/sign_in HTTP/1.1

   Host: 159.xxx.xxx.xxx

   --snip--

   ----------1881604860

   Content-Disposition: form-data; name="user[otp_attempt]"

➊ 212421

   ----------1881604860--
```

`POST`请求将包括一个 OTP 令牌➊，该令牌用于验证用户进行 2FA 的第二步。OTP 令牌仅在用户已输入用户名和密码后生成，但如果攻击者试图登录自己的账户，他们可以使用 Burp 等工具拦截请求，并将请求中的用户名替换为另一个用户名。这将改变他们登录的账户。例如，攻击者可以尝试如下方式登录名为`john`的用户账户：

```
   POST /users/sign_in HTTP/1.1

   Host: 159.xxx.xxx.xxx

   --snip--

   ----------1881604860

   Content-Disposition: form-data; name="user[otp_attempt]"

   212421

   ----------1881604860

➊ Content-Disposition: form-data; name="user[login]"

   john

   ----------1881604860--
```

`user[login]`请求告诉 GitLab 网站，用户即使没有尝试登录，也已使用其用户名和密码进行过登录尝试。无论如何，GitLab 网站都会为`john`生成一个 OTP 令牌，攻击者可以猜测并提交给网站。如果攻击者猜中了正确的 OTP 令牌，他们就能在从未知道密码的情况下登录。

这个漏洞的一个警告是，攻击者必须知道或猜测目标的有效 OTP 令牌。OTP 令牌每 30 秒变化一次，且仅在用户登录或提交 `user[login]` 请求时生成。利用这个漏洞是非常困难的。不过，GitLab 在报告后两天内确认并修复了这个漏洞。

#### *要点*

双因素认证是一个难以完美实现的系统。当你发现某个网站使用双因素认证时，一定要测试其功能，例如令牌的有效期、最大尝试次数限制等。还要检查过期的令牌是否可以重用，猜测令牌的可能性以及其他令牌漏洞。GitLab 是一个开源应用程序，Abma 很可能通过审查源代码发现了这个问题，因为他在报告中识别了代码中开发人员的错误。尽管如此，仍需留意 HTTP 响应，这些响应可能会泄露你可以在 HTTP 请求中包含的参数，就像 Abma 所做的那样。

### Yahoo! PHP 信息泄露

**难度：** 中等

**URL：** *http://nc10.n9323.mail.ne1.yahoo.com/phpinfo.php/*

**来源：** *[`blog.it-securityguard.com/bugbounty-yahoo-phpinfo-php-disclosure-2/`](https://blog.it-securityguard.com/bugbounty-yahoo-phpinfo-php-disclosure-2/)*

**报告日期：** 2014 年 10 月 16 日

**悬赏金额：** 暂无

这份报告没有像本章中的其他报告那样获得悬赏。但它展示了网络扫描和自动化在发现应用配置漏洞中的重要性。2014 年 10 月，HackerOne 的 Patrik Fehrenbach 发现了一台返回 `phpinfo` 函数内容的 Yahoo! 服务器。`phpinfo` 函数输出当前 PHP 状态的信息。这些信息包括编译选项和扩展、版本号、服务器和环境信息、HTTP 头等。由于每个系统的设置不同，`phpinfo` 通常用于检查系统上的配置设置和预定义变量。这种详细的信息不应在生产系统上公开，因为它可以让攻击者深入了解目标的基础设施。

此外，尽管费伦巴赫没有提到这一点，但请注意，`phpinfo` 会包含 `httponly` cookie 的内容。如果一个域存在 XSS 漏洞 *并且* 有一个 URL 会泄露 `phpinfo` 的内容，攻击者可以利用 XSS 发起对该 URL 的 HTTP 请求。由于 `phpinfo` 的内容被泄露，攻击者可以窃取 `httponly` cookie。这个漏洞之所以存在，是因为恶意 JavaScript 可以读取包含该值的 HTTP 响应体，即使它无法直接读取 cookie。

为了发现这个漏洞，费伦巴赫对 *[yahoo.com](http://yahoo.com)* 进行了 ping 操作，返回了 98.138.253.109。他使用了 `whois` 命令行工具查询该 IP，返回了以下记录：

```
NetRange: 98.136.0.0 - 98.139.255.255

CIDR: 98.136.0.0/14

OriginAS:

NetName: A-YAHOO-US9

NetHandle: NET-98-136-0-0-1

Parent: NET-98-0-0-0-0

NetType: Direct Allocation

RegDate: 2007-12-07

Updated: 2012-03-02

Ref: http://whois.arin.net/rest/net/NET-98-136-0-0-1
```

第一行确认了 Yahoo!拥有从 98.136.0.0 到 98.139.255.255（或 98.136.0.0/14）的一个大块 IP 地址，共计 260,000 个唯一 IP 地址。这是大量潜在的目标！使用以下简单的 bash 脚本，Fehrenbach 搜索了 IP 地址的`phpinfo`文件：

```
   #!/bin/bash

➊ for ipa in 98.13{6..9}.{0..255}.{0..255}; do

➋ wget -t 1 -T 5 http://${ipa}/phpinfo.php; done &
```

代码在➊处进入一个`for`循环，该循环遍历每对大括号中的每个范围的所有可能数字。第一个测试的 IP 将是 98.136.0.0，然后是 98.136.0.1，再然后是 98.136.0.2，依此类推，直到 98.139.255.255。每个 IP 地址都会存储在变量`ipa`中。代码在➋处使用`wget`命令行工具对正在测试的 IP 地址发出`GET`请求，通过将`${ipa}`替换为`for`循环中当前 IP 地址的值。`-t`标志表示在请求失败时应重试的次数，在此情况下为`1`。`-T`标志表示在考虑请求超时之前等待的秒数。运行他的脚本后，Fehrenbach 发现网址*http://nc10.n9323.mail.ne1.yahoo.com*启用了`phpinfo`功能。

#### *关键要点*

当你进行黑客攻击时，除非明确告诉你某个部分不在范围内，否则可以把公司的整个基础设施视为合法目标。虽然这份报告没有支付赏金，但你可以采用类似的技巧来寻找一些可观的奖金。此外，寻找自动化测试的方法。你通常需要编写脚本或使用工具来自动化过程。例如，Fehrenbach 发现的 260,000 个潜在 IP 地址，如果手动测试是不可能完成的。

### HackerOne Hacktivity 投票

**难度：** 中等

**网址：** *[`hackerone.com/hacktivity/`](https://hackerone.com/hacktivity/)*

**来源：** *[`hackerone.com/reports/137503/`](https://hackerone.com/reports/137503/)*

**报告日期：** 2016 年 5 月 10 日

**赏金支付：** 礼品

尽管这份报告从技术上讲并没有发现安全漏洞，但它是一个很好的例子，展示了如何使用 JavaScript 文件来寻找新的功能进行测试。在 2016 年春季，HackerOne 一直在开发一项功能，允许黑客对报告进行投票。这个功能在用户界面中并未启用，本不应当被使用。

HackerOne 使用 React 框架来渲染其网站，因此它的大部分功能都定义在 JavaScript 中。使用 React 构建功能的一个常见方法是根据服务器的响应启用 UI 元素。例如，网站可能会根据服务器是否识别某个用户为管理员来启用与管理员相关的功能，如删除按钮。但服务器可能不会验证通过 UI 发起的 HTTP 请求是否由合法管理员发起。根据报告，黑客 apok 测试了禁用的 UI 元素是否仍然可以用来发起 HTTP 请求。黑客修改了 HackerOne 的 HTTP 响应，将所有错误值改为正确，可能是通过像 Burp 这样的代理实现的。这样就揭示了新的 UI 按钮，点击后会发起 `POST` 请求，用于报告投票。

发现隐藏的 UI 功能的其他方法是使用浏览器开发者工具或像 Burp 这样的代理，搜索 JavaScript 文件中的 `POST` 字段，以识别该站点使用的 HTTP 请求。搜索 URL 是发现新功能的简便方法，无需浏览整个应用程序。在此情况下，JavaScript 文件包含以下内容：

```
vote: function() {

var e = this;

a.ajax({

  ➊ url: this.url() + "/votes",

    method: "POST",

    datatype: "json",

    success: function(t) {

        return e.set({

            vote_id: t.vote_id,

            vote_count: t.vote_count

        })

    }

})

},

unvote: function() {

var e = this;

a.ajax({

  ➋ url: this.url() + "/votes" + this.get("vote_id"),

    method: "DELETE":,

    datatype: "json",

    success: function(t) {

        return e.set({

            vote_id: t.void 0,

            vote_count: t.vote_count

        })

    }

})

}
```

如你所见，投票功能通过 ➊ 和 ➋ 的两个 URL 路径提供。在此报告时，你可以对这些 URL 端点执行 `POST` 请求。然后你可以投票，尽管该功能尚未可用或完成。

#### *收获*

当一个站点依赖 JavaScript，特别是像 React、AngularJS 等框架时，使用 JavaScript 文件是发现更多应用程序测试区域的好方法。使用 JavaScript 文件可以节省时间，还可能帮助你识别隐藏的端点。使用像 *[`github.com/nahamsec/JSParser`](https://github.com/nahamsec/JSParser)* 这样的工具可以让你更容易地追踪 JavaScript 文件的变化。

### 访问 PornHub 的 Memcache 安装

**难度：** 中等

**URL：** *[stage.pornhub.com](http://stage.pornhub.com)*

**来源：** *[`blog.zsec.uk/pwning-pornhub/`](https://blog.zsec.uk/pwning-pornhub/)*

**报告日期：** 2016 年 3 月 1 日

**奖励支付：** $2,500

2016 年 3 月，Andy Gill 正在参与 PornHub 的漏洞奖励计划，该计划涵盖 **.pornhub.com* 域名。这意味着该站点的所有子域名都在范围内，并有资格获得奖励。使用常见子域名的自定义列表，Gill 发现了 90 个 PornHub 子域名。

访问这些网站会非常耗时，因此正如 Fehrenbach 在前面的示例中所做的那样，Gill 使用 EyeWitness 自动化了这个过程。EyeWitness 捕获网站截图，并提供开放 80、443、8080 和 8443 端口的报告（这些是常见的 HTTP 和 HTTPS 端口）。网络和端口超出了本书的范围，但通过打开端口，服务器可以使用软件发送和接收互联网流量。

这个任务并没有揭示太多信息，所以 Gill 专注于 *[stage.pornhub.com](http://stage.pornhub.com)*，因为临时和开发服务器更容易出现配置错误。首先，他使用命令行工具 `nslookup` 获取该站点的 IP 地址。返回的记录如下：

```
   Server:     8.8.8.8

   Address:    8.8.8.8#53

   Non-authoritative answer:

   Name:       stage.pornhub.com

➊ Address:    31.192.117.70
```

地址是显著的值 ➊，因为它显示了 *[stage.pornhub.com](http://stage.pornhub.com)* 的 IP 地址。接下来，Gill 使用工具 Nmap 扫描服务器的开放端口，使用的命令是 `nmap -sV -p- 31.192.117.70 -oA stage__ph -T4`。

命令中的第一个标志（`-sV`）启用版本检测。如果发现开放端口，Nmap 会尝试确定该端口上运行的软件。`–p-` 标志指示 Nmap 扫描所有 65,535 个可能的端口（默认情况下，Nmap 仅扫描最常用的 1,000 个端口）。接下来，命令列出了要扫描的 IP：在本例中是 *[stage.pornhub.com](http://stage.pornhub.com)* (`31.192.117.70`)。然后，`-oA` 标志将扫描结果输出为三种主要输出格式，分别是普通格式、可 grep 格式和 XML 格式。此外，命令还包括了一个基础文件名 `stage__ph` 作为输出文件的名称。最后一个标志 `-T4` 让 Nmap 运行得更快。默认值是 3：值 1 最慢，值 5 最快。较慢的扫描可以避开入侵检测系统，而较快的扫描则需要更多带宽，并可能不那么准确。当 Gill 运行该命令时，他收到了以下结果：

```
   Starting Nmap 6.47 ( http://nmap.org ) at 2016-06-07 14:09 CEST

   Nmap scan report for 31.192.117.70

   Host is up (0.017s latency).

   Not shown: 65532 closed ports

   PORT    STATE    SERVICE      VERSION

   80/tcp  open     http         nginx

   443/tcp open     http         nginx

➊ 60893/tcp open   memcache

   Service detection performed. Please report any incorrect results at http://

   nmap.org/submit/.

   Nmap done: 1 IP address (1 host up) scanned in 22.73 seconds
```

报告的关键部分是端口 60893 开放，并且 Nmap 识别它为运行 `memcache` ➊。Memcache 是一种缓存服务，它使用键值对来存储任意数据。通常，它用于通过缓存加速网站内容的传递，从而提高网站的访问速度。

发现该端口开放并不是一个漏洞，但绝对是一个警示信号。原因是 Memcache 的安装指南推荐将其设置为公共不可访问，以作为安全预防措施。随后，Gill 使用命令行工具 Netcat 尝试建立连接。他没有被要求进行身份验证，这是一个应用程序配置漏洞，因此 Gill 能够运行无害的统计和版本命令来确认他的访问权限。

访问 Memcache 服务器的严重性取决于它缓存了什么信息以及应用程序如何使用这些信息。

#### *收获*

子域名和更广泛的网络配置代表了黑客攻击的巨大潜力。如果一个程序的漏洞奖励计划涵盖了广泛的范围或所有子域名，你可以枚举子域名。因此，你可能会发现别人没有测试过的攻击面。这对于查找应用程序配置漏洞特别有帮助。花时间熟悉像 EyeWitness 和 Nmap 这样的工具是值得的，它们可以为你自动化枚举过程。

### 总结

发现应用程序逻辑和配置漏洞需要你寻找与应用程序以不同方式互动的机会。Shopify 和 Twitter 的例子很好地展示了这一点。Shopify 在 HTTP 请求中没有验证权限。同样，Twitter 在其移动应用程序中省略了安全检查。两者都涉及从不同角度测试这些网站。

定位逻辑和配置漏洞的另一个技巧是寻找你可以探索的应用程序表面区域。例如，新的功能是这些漏洞的一个很好的切入点。它总是提供一个发现错误的好机会。新代码提供了测试边缘情况或新代码与现有功能交互的机会。你还可以深入挖掘一个网站的 JavaScript 源代码，发现那些在网站 UI 中无法看到的功能变化。

黑客攻击可能非常耗时，因此学习能够自动化工作流程的工具非常重要。本章中的示例包括小型 bash 脚本、Nmap、EyeWitness 和*bucket_finder*。你可以在附录 A 中找到更多工具。
