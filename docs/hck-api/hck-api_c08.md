# 第九章：攻击认证

![](img/chapterart.png)

在测试认证时，你会发现许多困扰 Web 应用程序数十年的缺陷已经被移植到了 API 中：糟糕的密码和密码要求、默认凭证、冗长的错误信息以及糟糕的密码重置流程。

此外，API 中常见的一些弱点比传统的 Web 应用程序更为普遍。API 认证漏洞有多种形式。你可能会遇到完全没有认证、没有对认证尝试进行速率限制、为所有请求使用相同的令牌或密钥、使用熵不足的令牌以及一些 JSON Web Token（JWT）配置漏洞。

本章将引导你了解经典的认证攻击，如暴力破解攻击和密码喷洒攻击，之后我们将讨论特定于 API 的令牌攻击，例如令牌伪造和 JWT 攻击。一般来说，这些攻击都有一个共同的目标，即获得未授权访问，无论是从无访问状态到未授权访问状态，从其他用户的资源中获得访问权限，还是从有限的 API 访问状态升级为特权访问状态。

## 经典认证攻击

在第二章中，我们介绍了 API 中使用的最简单的认证方式：基本认证。要使用此方法进行认证，消费者会发送一个包含用户名和密码的请求。如我们所知，RESTful API 不保持状态，因此，如果 API 在整个 API 中使用基本认证，每个请求都必须带上用户名和密码。因此，提供者通常只在注册过程中使用基本认证。然后，在用户成功认证后，提供者会发放 API 密钥或令牌。提供者接着会检查用户名和密码是否与存储的认证信息匹配。如果凭证匹配，提供者会返回成功响应。如果不匹配，API 可能会返回几种响应之一。提供者可能会对所有错误的认证尝试发送一个通用响应：“用户名或密码错误。”这会告诉我们最少的信息，但有时提供者会倾向于方便消费者，提供更多有用的信息。提供者可能会特别告诉我们某个用户名不存在。那么我们就会得到一个可以帮助我们发现并验证用户名的响应。

### 密码暴力破解攻击

获得 API 访问权限的一种更直接的方法是进行暴力破解攻击。暴力破解 API 的身份验证与其他暴力破解攻击没有太大区别，只不过你会将请求发送到 API 端点，负载通常是 JSON 格式，并且身份验证值可能是 base64 编码的。暴力破解攻击通常会发出很大的噪音，耗时且粗暴，但如果 API 缺乏防止暴力破解攻击的安全控制，我们不应该避免利用这一点来获得优势。

微调暴力破解攻击的最佳方法之一是生成针对目标的密码。为此，你可以利用在数据泄露漏洞中揭示的信息，比如你在实验室 #4 中发现的漏洞，来编制用户名和密码列表。泄露的数据可能揭示有关用户帐户的技术细节，例如用户是否使用了多因素认证，是否使用了默认密码，以及帐户是否已激活。如果泄露的数据涉及用户信息，你可以将其输入到能够生成大型、有针对性的密码列表的工具中，以进行暴力破解攻击。有关创建针对性密码列表的更多信息，请查看 Mentalist 应用程序（[`github.com/sc0tfree/mentalist`](https://github.com/sc0tfree/mentalist)）或常见用户密码分析工具（[`github.com/Mebus/cupp`](https://github.com/Mebus/cupp)）。

一旦你有了合适的字典来进行暴力破解攻击，你可以使用像 Burp Suite 的暴力破解工具或第四章介绍的 Wfuzz 等工具。以下示例使用 Wfuzz 和一个旧的、广为人知的密码列表 *rockyou.txt*：

```
$ **wfuzz -d '{"email":"a@email.com","password":"FUZZ"}' --hc 405 -H 'Content-Type: application/json' -z file,/home/hapihacker/rockyou.txt http://192.168.195.130:8888/api/v2/auth**
==================================================================
ID             Response   Lines    Word       Chars       Payload
==================================================================
000000007:   200        0 L      1 W        225 Ch      "Password1!"
000000005:   400        0 L      34 W       474 Ch      "win"
```

`-d` 选项允许你模糊化发送在 POST 请求体中的内容。后面的花括号包含了 POST 请求体。为了发现此示例中使用的请求格式，我尝试使用浏览器进行身份验证，然后捕获了身份验证请求并在这里复制了它的结构。在这个例子中，Web 应用发出了包含 `"email"` 和 `"password"` 参数的 POST 请求。每个 API 的请求体结构会有所不同。在此示例中，你可以看到我们指定了一个已知的电子邮件并将 `FUZZ` 参数用作密码。

`--hc` 选项用于隐藏带有特定响应代码的响应。如果你经常在许多请求中收到相同的状态代码、字长和字符数，这个选项很有用。如果你知道目标的典型失败响应是什么样的，那么就没有必要看到数百或数千个相同的响应。`–hc` 选项可以帮助你过滤掉不想看到的响应。

在测试实例中，典型的失败请求会返回 405 状态码，但这在不同的 API 中可能有所不同。接下来，`-H` 选项允许你在请求中添加一个头部。如果你在发送 JSON 数据时没有包含 `Content-Type:application/json` 头部，某些 API 提供者可能会返回 HTTP 415 不支持的媒体类型错误码。

一旦请求发送完毕，你可以在命令行中查看结果。如果你的 `–hc` Wfuzz 选项生效，那么结果应该相对容易阅读。否则，200 到 300 范围的状态码通常表明你已经成功破解了凭证。

### 密码重置和多因素认证暴力破解攻击

虽然你可以将暴力破解技术直接应用于认证请求，但你也可以将其用于密码重置和多因素认证（MFA）功能。如果密码重置过程包括安全问题，并且没有对请求进行速率限制，那么我们就可以在这种攻击中加以利用。

与 GUI Web 应用程序类似，API 通常会使用 SMS 恢复码或一次性密码（OTP）来验证想要重置密码的用户身份。此外，提供者可能会在成功的身份验证尝试中部署 MFA，因此你需要绕过该过程才能访问账户。在后台，API 通常会通过一个服务来实现此功能，向与账户关联的电话号码或电子邮件发送一个四到六位数的验证码。如果我们没有被速率限制阻止，我们应该能够通过暴力破解这些验证码来访问目标账户。

首先捕获相关过程的请求，例如密码重置过程。在以下请求中，你可以看到消费者在请求体中包括了一个 OTP，以及用户名和新密码。因此，要重置用户的密码，我们需要猜测 OTP。

```
POST /identity/api/auth/v3/check-otp HTTP/1.1
Host: 192.168.195.130:8888
User-Agent: Mozilla/5.0 (x11; Linux x86_64; rv: 78.0) Gecko/20100101
Accept: */*
Accept -Language: en-US, en;q=0.5
Accept-Encoding: gzip,deflate
Referer: http://192.168.195.130:8888/forgot-password
Content-Type: application/json
Origin: http://192.168.195.130:8888
Content-Length: 62
Connection: close

{
"email":"a@email.com",
**"otp":"1234"**,
"password": "Newpassword"
}
```

在这个示例中，我们将利用 Burp Suite 中的暴力破解负载类型，但你也可以使用 Wfuzz 配置并运行一个等效的攻击，使用暴力破解选项。一旦你在 Burp Suite 中捕获到密码重置请求，突出显示 OTP 并添加第四章讨论的攻击位置标记，将该值转化为变量。接下来，选择 **Payloads** 选项卡，并将负载类型设置为 **brute forcer**（参见 图 8-1）。

![Burp Suite Payloads 选项卡截图，"Payload set" 字段设置为“1”，"Payload type" 字段设置为“Brute forcer”，"Character set" 字段设置为“0123456789”，Min length 和 Max length 字段都设置为“4”](img/F08001.png)

图 8-1：配置 Burp Suite Intruder，设置暴力破解负载类型

如果你正确配置了负载设置，它们应该与图 8-1 中的设置匹配。在字符集字段中，只包含数字和用于一次性密码（OTP）的字符。在其详细错误信息中，API 提供者可能会指明它期望的值。你通常可以通过启动你自己账户的密码重置并查看 OTP 的组成来测试这一点。例如，如果 API 使用的是四位数字验证码，那么将数字 0 到 9 添加到字符集中。然后将验证码的最小和最大长度设置为**4**。

对密码重置代码进行暴力破解绝对值得一试。然而，许多 Web 应用会同时强制实施速率限制，并限制你猜测 OTP 的次数。如果速率限制阻碍了你，你可以尝试第十三章中的某些规避技巧。

### 密码喷射攻击

许多安全控制措施可能会阻止你成功暴力破解 API 的身份验证。一种叫做*密码喷射攻击*的技巧能够通过将一个长用户列表与一个短目标密码列表相结合，避开这些控制。假设你知道某个 API 的身份验证过程有一个锁定策略，并且只允许 10 次登录尝试。你可以制作一个包含九个最可能密码的列表（比限制少一个密码），然后用这些密码尝试登录许多用户账户。

当你进行密码喷射攻击时，像*rockyou.txt*这样的庞大且过时的词库将不起作用。这样的文件中包含了太多不太可能的密码，无法取得任何成功。相反，你应该根据 API 提供者的密码策略约束（你可以在侦察过程中发现这些约束），制作一个简短的可能密码列表。大多数密码策略可能要求最小字符长度、大写字母和小写字母，可能还会要求数字或特殊字符。

尝试将你的密码喷射列表与两种*低阻力路径（POS）*密码混合，或者是那些足够简单以供猜测，但又足够复杂以满足基本密码要求的密码（通常要求至少八个字符、一个符号、大小写字母和一个数字）。第一种类型包括明显的密码，如 QWER!@#$、Password1!，以及公式*季节+年份+符号*（例如 Winter2021!、Spring2021?、Fall2021!和 Autumn2021?）。第二种类型包括更为复杂的密码，这些密码通常直接与目标相关，通常包含一个大写字母、一个数字、组织的某个细节和一个符号。如果我要攻击 Twitter 员工的端点，以下是我可能生成的一个简短密码喷射列表：

1.  Winter2021!

1.  Spring2021!

1.  QWER!@#$

1.  Password1!

1.  March212006!

1.  July152006!

1.  Twitter@2022

1.  JPD1976!

1.  Dorsey@2021

密码喷射攻击的关键在于最大化你的用户列表。你包含的用户名越多，获得访问权限的机会就越大。在进行侦察工作时，或者通过发现过度的数据暴露漏洞来构建用户列表。

在 Burp Suite 的 Intruder 中，你可以以类似标准暴力破解攻击的方式设置此攻击，唯一的区别是你将同时使用用户列表和密码列表。选择 cluster bomb 攻击类型，并将攻击位置设置在用户名和密码周围，如图 8-2 所示。

![Burp Suite 有效载荷位置屏幕截图，攻击类型设置为“Cluster bomb”，电子邮件和密码字段设置为 POST 请求体中的有效载荷位置。](img/F08002.png)

图 8-2：使用 Intruder 进行凭证喷射攻击的示例

请注意，第一个攻击位置设置为替换 *@email.com* 前的用户名，如果你只针对特定电子邮件域名的用户进行测试，可以这样做。

接下来，将收集的用户列表添加为第一个有效载荷集，将一个简短的密码列表作为第二个有效载荷集。一旦你的有效载荷如图 8-3 所示配置好，你就可以开始执行密码喷射攻击。

![Burp Suite 有效载荷集和有效载荷选项屏幕的两张截图。第一张的“有效载荷集”字段设置为 1，并且列出了一些名称作为有效载荷选项。第二张的“有效载荷集”字段设置为 2，并列出了一些常见的密码作为有效载荷选项。](img/F08003.png)

图 8-3：Burp Suite Intruder 的 cluster bomb 攻击示例有效载荷

当你分析结果时，如果你大致了解标准成功登录的样子，会有所帮助。如果不确定，可以通过检查返回的长度和响应代码中的异常来寻找线索。大多数 Web 应用程序在成功登录时返回的 HTTP 状态代码位于 200 或 300 系列。在图 8-4 中，你可以看到一次成功的密码喷射攻击，其中有两个异常特征：状态代码为 200，响应长度为 682。

![Burp Suite Intruder 截图，显示已提交的有效载荷列表、响应中的状态代码以及每个响应的长度。大多数的状态代码为 500，长度为 479，但有一个状态代码为 200，长度为 682。](img/F08004.png)

图 8-4：使用 Intruder 进行成功密码喷射攻击的示例

为了帮助通过 Intruder 识别异常，你可以按状态代码或响应长度对结果进行排序。

### 在暴力破解攻击中包含 Base64 身份验证

一些 API 会对 API 请求中发送的身份验证有效载荷进行 Base64 编码。这样做有很多原因，但需要知道的是，安全性并不是其中之一。你可以轻松绕过这个小小的不便。

如果你测试身份验证尝试并注意到一个 API 正在进行 base64 编码，它很可能是在后台将数据与 base64 编码的凭据进行比较。这意味着你应该调整你的模糊测试攻击，包括使用 Burp Suite Intruder 进行 base64 有效负载，它可以同时进行 base64 的编码和解码。例如，图 8-5 中的密码和邮箱值是 base64 编码的。你可以通过高亮显示有效负载，右键单击并选择**Base64 解码**（或快捷键 ctrl-shift-B）来解码它们。这将揭示有效负载内容，让你可以看到它的格式。

比如，要使用 base64 编码执行密码喷洒攻击，首先选择攻击位置。在本例中，我们将选择图 8-5 中的 base64 编码密码。接下来，添加有效负载集；我们将使用上一节列出的密码。

现在，为了在请求发送之前对每个密码进行编码，我们必须使用一个有效负载处理规则。在“有效负载”标签下有一个选项可以添加这样的规则。选择**添加**▶**编码**▶**Base64 编码**，然后点击**确定**。你的有效负载处理窗口应该如下所示：图 8-6。

![Burp Suite 中一个 HTTP 请求的截图，突出显示并右键单击一个值，菜单中选择“转换选择，Base64，Base64 解码”](img/F08005.png)

图 8-5：使用 Burp Suite Intruder 解码 base64

![Burp Suite 的“添加有效负载处理规则”菜单截图。下拉菜单中选择了“编码”和“Base64 编码”。](img/F08006.png)

图 8-6：在 Burp Suite Intruder 中添加有效负载处理规则

现在，你的 base64 编码的密码喷洒攻击已经准备好启动。

## 伪造令牌

如果正确实现，令牌可以成为 API 验证用户并授权他们访问资源的优秀方法。然而，如果在生成、处理或处理令牌时出现问题，它们就会成为我们进入系统的钥匙。

令牌的问题在于它们可能被窃取、泄露或伪造。我们已经在第六章中介绍了如何窃取和查找泄露的令牌。在本节中，我将指导你如何在令牌生成过程中存在漏洞时伪造自己的令牌。这首先需要分析 API 提供方的令牌生成过程是否具有可预测性。如果我们能发现提供的令牌中存在任何模式，可能就能够伪造自己的令牌或劫持其他用户的令牌。

API 通常会使用令牌作为授权方法。消费者可能首先需要使用用户名和密码组合进行身份验证，但之后，提供方会生成一个令牌并将该令牌提供给消费者，用于他们的 API 请求。如果令牌生成过程存在缺陷，我们就能够分析令牌，劫持其他用户的令牌，然后使用它们访问受影响用户的资源和额外的 API 功能。

Burp Suite 的 Sequencer 提供两种令牌分析方法：手动分析提供的文本文件中的令牌和执行实时捕获以自动生成令牌。我将引导你完成这两个过程。

### 手动加载分析

要执行手动加载分析，请选择**Sequencer**模块并选择**Manual Load**标签。点击**Load**并提供要分析的令牌列表。样本中的令牌越多，结果会越好。Sequencer 至少需要 100 个令牌才能执行基本分析，其中包括*位级*分析，或将令牌转换为比特集合后进行的自动化分析。这些比特集合会通过一系列压缩、相关性和频谱测试，同时还包括基于联邦信息处理标准（FIPS）140-2 安全要求的四个测试。

完整的分析还包括*字符级*分析，这是在原始令牌形式中对每个字符在特定位置进行的一系列测试。然后，令牌会经过字符计数分析和字符过渡分析，这两个测试分别分析字符在令牌中的分布以及令牌之间的差异。要执行完整分析，Sequencer 可能需要成千上万的令牌，具体取决于每个令牌的大小和复杂性。

令牌加载完成后，你应该能看到加载的令牌总数、最短的令牌和最长的令牌，如图 8-7 所示。

![Burp Suite Sequencer 截图，显示加载到手动加载字段中的令牌值。](img/F08007.png)

图 8-7：Burp Suite Sequencer 中手动加载的令牌

现在你可以点击**Analyze Now**开始分析。Burp Suite 会生成一份报告（见图 8-8）。

![Burp Suite Sequencer 摘要标签截图，显示“总体结果”和“有效熵”部分，描述分析结果](img/F08008.png)

图 8-8：Sequencer 提供的令牌分析报告的摘要标签

令牌分析报告以结果摘要开始。总体结果包括令牌样本中随机性的质量。在图 8-8 中，你可以看到随机性质量极差，表明我们很可能能够通过暴力破解其他现有的令牌。

为了最小化暴力破解令牌所需的工作量，我们希望确定令牌中哪些部分不变，哪些部分经常变化。使用字符位置分析来确定哪些字符应进行暴力破解（见图 8-9）。你可以在字符级分析标签下的字符集选项中找到此功能。

如你所见，令牌的字符位置变化不大，除了最后三个字符；字符串`Ab4dt0k3n`在整个采样过程中保持不变。现在我们知道应该对最后三个字符进行暴力破解，而不触动令牌的其余部分。

![Burp Suite Sequencer “字符集”菜单的截图。显示了每个位置使用的字符集大小的图表。大多数位置的字符集小于 5，但第 10 个位置的字符集包含超过 25 个字符。](img/F08009.png)

图 8-9：Sequencer 的字符级分析中找到的字符位置图表

### 实时令牌捕获分析

Burp Suite 的 Sequencer 可以自动请求 API 提供者生成 20,000 个令牌进行分析。为此，我们只需拦截提供者的令牌生成过程，然后配置 Sequencer。Burp Suite 将重复令牌生成过程，最多达到 20,000 次，以分析令牌之间的相似性。

在 Burp Suite 中，拦截启动令牌生成过程的请求。选择**操作**（或右键点击请求），然后将其转发到 Sequencer。在 Sequencer 中，确保选中了实时捕获标签，然后在**响应中的令牌位置**下，选择**为自定义位置配置**选项。如图 8-10 所示，选中生成的令牌并点击**确定**。

选择**开始实时捕获**。Burp Sequencer 现在将开始捕获令牌进行分析。如果你选中“自动分析”复选框，Sequencer 将在不同的里程碑处显示有效的熵结果。

除了执行熵分析，Burp Suite 还会为你提供大量的令牌，这些令牌在绕过安全控制时可能会非常有用（我们在第十三章中探讨这一话题）。如果某个 API 在创建新令牌后没有使旧令牌失效，并且安全控制使用令牌作为身份验证方法，那么你现在就有最多 20,000 个身份，可以帮助你避免被检测到。

如果某些令牌字符位置的熵较低，你可以尝试对这些字符位置进行暴力攻击。审查熵低的令牌可能会揭示出你可以利用的某些模式。例如，如果你发现某些位置的字符只包含小写字母，或某个范围内的数字，你将能够通过减少请求尝试的次数来增强暴力破解攻击。

![Burp Suite Sequencer 的“定义自定义令牌位置”菜单截图。选中了“定义起始和结束”，为“起始位置后表达式”和“结束位置分隔符”字段设置了表达式，并在 HTTP 响应中显示了高亮的令牌值。](img/F08010.png)

图 8-10：选中的 API 提供者令牌响应进行分析

### 暴力破解可预测的令牌

让我们回到手动负载分析中发现的坏令牌（其最后三个字符是唯一变化的部分），并对可能的字母和数字组合进行暴力破解，以找到其他有效的令牌。一旦发现有效令牌，我们可以测试对 API 的访问并找出我们被授权执行的操作。

当你通过字母和数字的组合进行暴力破解时，最好尽量减少变量的数量。字符级分析已经告诉我们，令牌`Ab4dt0k3n`的前九个字符保持静态。最后三个字符是变量，根据样本，我们可以看到它们遵循*字母 1* + *字母 2* + *数字*的模式。此外，令牌样本告诉我们，*字母 1*仅由*a*到*d*之间的字母组成。像这样的观察将帮助我们减少所需的暴力破解总量。

使用 Burp Suite Intruder 或 Wfuzz 对弱令牌进行暴力破解。在 Burp Suite 中，捕获一个需要令牌的 API 端点请求。在图 8-11 中，我们使用对 */identity/api/v2/user/dashboard* 端点的 GET 请求，并将令牌作为头信息。将捕获的请求发送到 Intruder，并在 Intruder 有效载荷位置选项卡下选择攻击位置。

![Burp Suite 有效载荷位置菜单，攻击类型设置为“Cluster bomb”，令牌的最后三个字符每个都设置为一个有效载荷位置](img/F08011.png)

图 8-11：Burp Suite Intruder 中的 Cluster Bomb 攻击

由于我们仅暴力破解最后三个字符，因此创建三个攻击位置：一个用于倒数第三个字符，一个用于倒数第二个字符，一个用于最后一个字符。将攻击类型更新为**cluster bomb**，这样 Intruder 就会遍历每个可能的组合。接下来，配置有效载荷，如图 8-12 所示。

![Burp Suite 有效载荷选项卡的截图，设置为“Payload set”为“1”，“Payload type”为“Brute forcer”，“Character set”为“abcd”，“Min length”和“Max length”字段都设置为“1”](img/F08012.png)

图 8-12：Burp Suite 的 Intruder 中的有效载荷选项卡

选择**Payload Set**编号，它代表特定的攻击位置，并将有效载荷类型设置为**brute forcer**。在字符集字段中，包含要在该位置测试的所有数字和字母。由于前两个有效载荷是字母，我们将尝试从*a*到*d*的所有字母。对于有效载荷集 3，字符集应包括数字 0 到 9。将最小长度和最大长度都设置为**1**，因为每个攻击位置只有一个字符。启动攻击后，Burp Suite 将发送所有 160 种令牌可能性请求到该端点。

Burp Suite CE 限制了 Intruder 请求的速度。作为更快且免费的替代方案，你可能希望使用 Wfuzz，方法如下：

```
$ **wfuzz -u vulnexample.com/api/v2/user/dashboard –hc 404 -H "token: Ab4dt0k3nFUZZFUZ2ZFUZ3Z1" -z list,a-b-c-d -z list,a-b-c-d -z range,0-9**
============================================================================
ID           Response   Lines    Word            Chars       Payload
============================================================================
000000117:   200        1 L      10 W            345 Ch      " Ab4dt0k3nca1"
000000118:   200        1 L      10 W            345 Ch      " Ab4dt0k3ncb2"
000000119:   200        1 L      10 W            345 Ch      " Ab4dt0k3ncc3"
000000120:   200        1 L      10 W            345 Ch      " Ab4dt0k3ncd4"
000000121:   200        1 L      10 W            345 Ch      " Ab4dt0k3nce5"
```

在请求中使用`-H`包含头部令牌。要指定三个有效载荷位置，将第一个标记为`FUZZ`，第二个标记为`FUZ2Z`，第三个标记为`FUZ3Z`。在`-z`后列出有效载荷。我们使用`-z list,a-b-c-d`来循环遍历前两个有效载荷位置的字母*a*到*d*，并使用`-z range,0-9`来循环遍历最后一个有效载荷位置的数字。

拥有有效令牌的列表后，可以在 API 请求中利用它们，以了解它们所具备的权限。如果你有 Postman 中的一组请求，可以尝试简单地将令牌变量更新为捕获到的令牌，并使用 Postman Runner 快速测试集合中的所有请求。这应该能帮助你大致了解给定令牌的能力。

## JSON Web Token 滥用

我在第二章中介绍了 JSON Web Token（JWT）。它们是较为常见的 API 令牌类型之一，因为它们可以在多种编程语言中使用，包括 Python、Java、Node.js 和 Ruby。虽然上一节描述的策略同样适用于 JWT，但这些令牌可能容易受到几种额外的攻击。本节将指导你进行一些可以用来测试和攻破实现不当的 JWT 的攻击。这些攻击可能会让你获得基本的未经授权的访问权限，甚至是 API 的管理员权限。

如果你捕获了另一个用户的 JWT，可以尝试将其发送给提供方，并冒充为你自己的令牌。令牌仍然有效的可能性是存在的，你可以像有效载荷中指定的用户一样访问 API。不过，更常见的是，你会在 API 注册后，提供方会响应一个 JWT。一旦你获得了 JWT，你将需要在所有后续请求中包括它。如果你使用浏览器，这个过程会自动发生。

### 识别与分析 JWT

你应该能够区分 JWT 与其他令牌，因为它们由三个由点分隔的部分组成：头部、有效载荷和签名。如以下 JWT 所示，头部和有效载荷通常以`ey`开头：

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJoYWNrYXBpcy5pbyIsImV4cCI6IDE1ODM2Mzc0ODgsInVzZXJuYW1lIjoiU2N1dHRsZXBoMXNoIiwic3VwZXJhZG1pbiI6dHJ1ZX0.1c514f4967142c27e4e57b612a7872003fa6cbc7257b3b74da17a8b4dc1d2ab9
```

攻击 JWT 的第一步是解码并分析它。如果在侦察过程中发现了暴露的 JWT，可以将其粘贴到解码工具中，查看 JWT 的有效载荷是否包含任何有用的信息，例如用户名和用户 ID。你可能会运气好，获得一个包含用户名和密码组合的 JWT。在 Burp Suite 的解码器中，将 JWT 粘贴到上方窗口，选择**Decode As**，然后选择**Base64**选项（见图 8-13）。

![Burp Suite 解码器的截图，长字符串被解码为一系列头部字段](img/F08013.png)

图 8-13：使用 Burp Suite 解码器解码 JWT

*头部*是一个 Base64 编码的值，包含关于令牌类型和签名所用的哈希算法的信息。解码后的头部将如下所示：

```
{
"alg": "HS256"
"typ": "JWT"
}
```

在此示例中，哈希算法是使用 SHA256 的 HMAC。HMAC 主要用于提供类似数字签名的完整性检查。SHA256 是一种由 NSA 开发并于 2001 年发布的哈希加密函数。你可能还会看到另一种常见的哈希算法 RS256，或者 RSA 使用 SHA256，它是一种非对称哈希算法。如需更多信息，请查看微软 API 文档中的加密部分：[`docs.microsoft.com/en-us/dotnet/api/system.security.cryptography`](https://docs.microsoft.com/en-us/dotnet/api/system.security.cryptography)。

当 JWT 使用对称密钥系统时，消费者和提供者都需要拥有相同的密钥。而当 JWT 使用非对称密钥系统时，提供者和消费者将使用两个不同的密钥。理解对称和非对称加密的区别，将在执行 JWT 算法绕过攻击时给你带来帮助，该攻击将在本章后面介绍。

如果算法值是`"none"`，则令牌没有使用任何哈希算法签名。稍后在本章中，我们将回到如何利用没有哈希算法的 JWT。

*负载*是令牌中包含的数据。负载中的字段根据 API 的不同而有所不同，但通常包含用于授权的信息，例如用户名、用户 ID、密码、电子邮件地址、令牌创建日期（通常称为 IAT）和权限级别。解码后的负载应如下所示：

```
{
  "userID": "1234567890",
  "name": "hAPI Hacker",
  "iat": 1516239022
}
```

最后，*签名*是用于令牌验证的 HMAC 输出，并且是通过头部指定的算法生成的。为了创建签名，API 将头部和负载进行 base64 编码，然后应用哈希算法和一个密钥。密钥可以是密码或密钥串，例如 256 位密钥。如果没有密钥，JWT 的负载将保持编码状态。

使用 HS256 的签名将如下所示：

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  thebest1)
```

为了帮助你分析 JWT，可以通过以下命令使用 JSON Web Token 工具包：

```
$ **jwt_tool eyghbocibiJIUZZINIISIRSCCI6IkpXUCJ9.eyIzdW1101IxMjMENTY3ODkwIiwibmFtZSI6ImhBuEkgSGFja2VyIiwiaWFQIjoxNTE2MjM5MDIyfQ.IX-Iz_e1CrPrkel FjArExaZpp3Y2tfawJUFQaNdftFw**
Original JWT:
Decoded Token Values:
Token header values:
[+] alg - "HS256"
[+] typ - "JWT"
Token payload values:
[+] sub = "1234567890"
[+] name - "HAPI Hacker"
[+] iat - 1516239022 = TIMESTAMP - 2021-01-17 17:30:22 (UTC)
JWT common timestamps:
iat - Issuedat
exp – Expires
nbf - NotBefore
```

如你所见，`jwt_tool`使得头部和负载值变得清晰明了。

此外，`jwt_tool`还有一个“Playbook 扫描”，可以用来定位 Web 应用程序并扫描常见的 JWT 漏洞。你可以通过以下命令运行此扫描：

```
$ **jwt_tool -t http://target-site.com/ -rc "Header: JWT_Token" -M pb**
```

要使用此命令，你需要了解 JWT 头部应该是什么。当你掌握这些信息时，将`"Header"`替换为头部的名称，`"JWT_Token"`替换为实际的令牌值。

### 无签名攻击

如果你遇到一个使用`"none"`作为算法的 JWT，那你就发现了一个轻松的漏洞。解码令牌后，你应该能清楚地看到头部、负载和签名。从这里，你可以修改负载中包含的信息，任意更改。例如，你可以将用户名更改为可能是提供者管理员账户（如 root、admin、administrator、test 或 adm）使用的名称，如下所示：

```
{
   "username": "root",
   "iat": 1516239022
}
```

一旦你编辑了有效载荷，使用 Burp Suite 的 Decoder 对有效载荷进行 base64 编码；然后将其插入到 JWT 中。重要的是，由于算法设置为`"none"`，任何存在的签名都可以被移除。换句话说，你可以移除 JWT 中第三个句点后的所有内容。将 JWT 发送给提供者并检查你是否获得了对 API 的未经授权的访问。

### 算法切换攻击

也有可能 API 提供者没有正确检查 JWT。如果是这种情况，我们可能能够欺骗提供者接受一个更改了算法的 JWT。

你首先应该尝试的事情之一是发送没有包含签名的 JWT。可以通过完全删除签名并保留最后一个句点来实现，如下所示：

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJoYWNrYXBpcy5pbyIsImV4cCI6IDE1ODM2Mzc0ODgsInVzZXJuYW1lIjoiU2N1dHRsZXBoMXNoIiwic3VwZXJhZG1pbiI6dHJ1ZX0.
```

如果这不成功，尝试将算法头字段更改为`"none"`。解码 JWT，更新`"alg"`值为`"none"`，然后对头部进行 base64 编码，并将其发送给提供者。如果成功，转到 None 攻击。

```
{
"alg": "none"
"typ": "JWT"
}
```

你可以使用 JWT_Tool 创建各种令牌，算法设置为`"none"`：

```
$ **jwt_tool <JWT_Token> -X a**
```

使用此命令将自动创建多个 JWT，它们应用了不同形式的“无算法”。

比提供者接受没有算法的情况更可能的场景是他们接受多种算法。例如，如果提供者使用 RS256，但没有限制可接受的算法值，我们可以将算法更改为 HS256。这个方法很有用，因为 RS256 是非对称加密方案，这意味着我们需要提供者的私钥和公钥来准确地对 JWT 签名进行哈希。而 HS256 是对称加密，所以签名和令牌验证使用同一个密钥。如果你能发现提供者的 RS256 公钥，然后将算法从 RS256 切换为 HS256，可能有机会将 RS256 公钥作为 HS256 密钥来利用。

JWT_Tool 可以让这个攻击变得稍微简单一些。它使用以下格式`jwt_tool <JWT_Token> -X k -pk public-key.pem`，如下面所示。你需要将捕获的公钥保存为文件，存储在你的攻击机器上。

```
$ **jwt_tool eyJBeXAiOiJKV1QiLCJhbGciOiJSUZI1Ni 19.eyJpc3MiOi JodHRwOlwvxC9kZW1vLnNqb2VyZGxhbmdrzwiwZXIubmxcLyIsIm1hdCI6MTYYCJkYXRhIjp7ImhlbGxvijoid29ybGQifx0.MBZKIRF_MvG799nTKOMgdxva_S-dqsVCPPTR9N9L6q2_10152pHq2YTRafwACdgyhR1A2Wq7wEf4210929BTWsVk19_XkfyDh_Tizeszny_GGsVzdb103NCITUEjFRXURJ0-MEETROOC-TWB8n6wOTOjWA6SLCEYANSKWaJX5XvBt6HtnxjogunkVz2sVp3 VFPevfLUGGLADKYBphfumd7jkh80ca2lvs8TagkQyCnXq5VhdZsoxkETHwe_n7POBISAZYSMayihlweg -x k-pk public-key-pem**
Original JWT:
File loaded: public-key. pem
jwttool_563e386e825d299e2fc@aadaeec25269 - EXPLOIT: Key-Confusion attack (signing using the Public key as the HMAC secret)
(This will only be valid on unpatched implementations of JWT.)
[+] ey JoexAiOiJK1QiLCJhbGciOiJIUZI1NiJ9.eyJpc3MiOiJodHRwOi8vZGVtby5zam91cmRsYW5na2VtcGVyLmSsLyIsIm1hdCI6MTYyNTc4NzkzOSwizhlbGxvIjoid29ybGQifxo.gyti NhqYsSiDIn10e-6-6SfNPJle-9EZbJZjhaa30
```

一旦你运行了命令，JWT_Tool 将提供一个新的令牌，可以用于对抗 API 提供者。如果提供者存在漏洞，你将能够劫持其他令牌，因为你现在拥有签名令牌所需的密钥。尝试重复这个过程，这次基于其他 API 用户，尤其是管理员用户创建一个新令牌。

### JWT 破解攻击

JWT 破解攻击试图破解 JWT 签名哈希使用的密钥，从而让我们完全控制创建自己有效 JWT 的过程。像这样的哈希破解攻击是离线进行的，不会与提供者进行交互。因此，我们不需要担心通过向 API 提供者发送数百万请求来造成混乱。

你可以使用 JWT_Tool 或像 Hashcat 这样的工具来破解 JWT 密钥。你将向哈希破解工具输入一个单词列表，哈希破解工具会将这些单词进行哈希处理，并将结果与原始哈希签名进行比较，以确定这些单词中是否有被用作哈希密钥。如果你正在进行长期的暴力破解，尝试所有字符的可能性，可能需要使用 Hashcat 所依赖的专用 GPU，而不是 JWT_Tool。不过，JWT_Tool 仍然可以在不到一分钟的时间内测试 1200 万个密码。

使用 JWT_Tool 进行 JWT 破解攻击时，请使用以下命令：

```
$ **jwt_tool****<JWT Token>****-C -d /wordlist.txt**
```

`-C`选项表示你将进行哈希破解攻击，`-d`选项指定你将用于破解哈希的字典或单词列表。在这个例子中，我的字典名为*wordlist.txt*，但你可以指定你想要使用的任何字典的目录和名称。JWT_Tool 将针对字典中的每个值返回“CORRECT key!”或者以“key not found in dictionary”表示破解失败。

## 总结

本章介绍了各种破解 API 认证、利用令牌以及专门攻击 JSON Web Tokens 的方法。当认证机制存在时，通常是 API 的首要防御机制，因此如果你的认证攻击成功，未经授权的访问就能成为进一步攻击的立足点。

## 实验 #5：破解 crAPI JWT 签名

返回到 crAPI 认证页面，尝试攻击认证过程。我们知道，这个认证过程分为三个部分：账户注册、密码重置功能和登录操作。所有这三部分都应该经过充分测试。在本实验中，我们将重点攻击成功认证后提供的令牌。

如果你记得你的 crAPI 登录信息，可以直接登录。（如果记不得，可以注册一个新账户。）确保 Burp Suite 已打开，且 FoxyProxy 已设置为将流量代理到 Burp，这样你就可以拦截登录请求。然后将拦截到的请求转发给 crAPI 提供商。如果你正确输入了电子邮件和密码，应该会收到 HTTP 200 响应和一个 Bearer 令牌。

希望你现在已经注意到 Bearer 令牌的特别之处。没错，它被分为三个部分，且由句点分隔，前两部分都以`ey`开头。我们有了一个 JSON Web 令牌！让我们开始使用像[`jwt.io`](https://jwt.io)或 JWT_Tool 之类的站点来分析 JWT。为了方便视觉展示，图 8-14 展示了 JWT.io 调试器中的令牌。

![jwt.io 截图，"Encoded"字段中有一个长字符串，每个 JWT 部分在"Decoded"字段中被解码](img/F08014.png)

图 8-14：在 JWT.io 调试器中分析捕获到的 JWT

如您所见，JWT 头部告诉我们算法设置为 HS512，这是一种比之前介绍的更强大的哈希算法。此外，载荷包含一个 `"sub"` 值，其中有我们的电子邮件。载荷还包含两个用于令牌过期的值：`iat` 和 `exp`。最后，签名确认正在使用 HMAC+SHA512，并且需要一个密钥来签署 JWT。

接下来的自然步骤是进行 None 攻击，尝试绕过哈希算法。我会留给你自己探索这部分内容。我们不会尝试其他算法切换攻击，因为我们已经在攻击一个对称密钥加密系统，因此切换算法类型在这里不会带来好处。这就剩下执行 JWT Crack 攻击了。

要对捕获的令牌进行 Crack 攻击，请从拦截到的请求中复制令牌。打开终端并运行 JWT_Tool。作为第一次攻击，我们可以使用 *rockyou.txt* 文件作为字典：

```
$ **jwt_tool eyJhbGciOiJIUZUxMi19.eyJzdWIiOiJhQGVtYWlsLmNvbSIsImlhdCI6MTYYNTC4NzA4MywiZXhwIjoxNjI10DCzNDgzfQ. EYx8ae40nE2n9ec4yBPI6Bx0z0-BWuaUQVJg2Cjx_BD_-eT9-Rpn87IAU@QM8 -C -d rockyou.txt**
Original JWT:
[*] Tested 1 million passwords so far
[*] Tested 2 million passwords so far
[*] Tested 3 million passwords so far
[*] Tested 4 million passwords so far
[*] Tested 5 million passwords so far
[*] Tested 6 million passwords so far
[*] Tested 7 million passwords so far
[*] Tested 8 million passwords so far
[*] Tested 9 million passwords so far
[*] Tested 10 million passwords so far
[*] Tested 11 million passwords so far
[*] Tested 12 million passwords so far
[*] Tested 13 million passwords so far
[*] Tested 14 million passwords so far
[-] Key not in dictionary
```

在本章开头，我提到过 *rockyou.txt* 已经过时，因此它可能无法带来任何成功。让我们尝试集思广益，找出一些可能的秘密，并将它们保存到我们自己的 *crapi.txt* 文件中（见 表 8-1）。你也可以使用密码分析工具生成类似的列表，正如本章早些时候推荐的那样。

表 8-1: 潜在的 crAPI JWT 秘密

| Crapi2020 | OWASP | iparc2022 |
| --- | --- | --- |
| crapi2022 | owasp | iparc2023 |
| crAPI2022 | Jwt2022 | iparc2020 |
| crAPI2020 | Jwt2020 | iparc2021 |
| crAPI2021 | Jwt_2022 | iparc |
| crapi | Jwt_2020 | JWT |
| community | Owasp2021 | jwt2020 |

现在使用 JWT_Tool 运行这个针对性的哈希破解攻击：

```
$ **jwt_tool eyJhbGciOiJIUzUxMi19.eyJzdwiOiJhQGVtYWlsLmNvbSIsImlhdCI6MTYYNTC4NzA4MywiZXhwIjoxNjI10DCzNDgzfQ. EYx8ae40nE2n9ec4yBPi6Bx0z0-BWuaWQVJg2Cjx_BD_-eT9-Rp 871Au@QM8-wsTZ5aqtxEYRd4zgGR51t5PQ -C -d crapi.txt**
Original JWT:
[+] crapi is the CORRECT key!
You can tamper/fuzz the token contents (-T/-I) and sign it using:
python3 jwt_tool.py [options here] -5 HS512 -p "crapi"
```

太棒了！我们已经发现 crAPI JWT 秘密是 `"crapi"`。

这个密钥并不是特别有用，除非我们有其他有效用户的电子邮件地址，这样我们才能伪造他们的令牌。幸运的是，我们在第七章实验的结尾已经完成了这项工作。让我们看看能否获得对机器人账户的未授权访问。如图 8-15 所示，我们使用 JWT.io 为 crAPI 机器人账户生成了一个令牌。

![jwt.io 截图，显示“Encoded”字段中的长字符串，并且 JWT 的每个部分在“Decoded”字段中解码](img/F08015.png)

图 8-15: 使用 JWT.io 生成令牌

别忘了，这个令牌的算法值是 HS512，你需要将 HS512 密钥添加到签名中。令牌生成后，你可以将其复制到已保存的 Postman 请求中，或者使用 Burp Suite 的 Repeater 发送请求，然后发送到 API。如果成功，你将劫持 crAPI 机器人账户。祝贺！
