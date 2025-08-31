# 第七章：开放重定向

![](img/chapterart.png)

网站通常使用 HTTP 或 URL 参数将用户重定向到指定的 URL，而无需用户任何操作。虽然这种行为可能有用，但也可能导致 *开放重定向*，即攻击者能够操纵该参数的值，将用户重定向到其他网站。让我们讨论一下这个常见的漏洞，为什么它是个问题，以及如何利用它来升级你发现的其他漏洞。

## 机制

网站经常需要自动重定向用户。例如，当未认证的用户尝试访问需要登录的页面时，这种情况通常会发生。网站通常会将这些用户重定向到登录页面，认证后再将他们返回到原来的位置。例如，当这些用户访问他们的账户仪表板 *https://example.com/dashboard* 时，应用程序可能会将他们重定向到登录页面 *https://example.com/login*。

为了稍后将用户重定向回他们之前的位置，网站需要记住他们在被重定向到登录页面之前打算访问的页面。因此，网站会使用某种形式的重定向 URL 参数，将其附加到 URL 中，以跟踪用户的原始位置。这个参数决定了在登录后将用户重定向到哪里。例如，URL *https://example.com/login?redirect=https://example.com/dashboard* 会在登录后将用户重定向到他们的仪表板，位于 *https://example.com/dashboard*。或者，如果用户最初试图浏览他们的账户设置页面，网站将在登录后将用户重定向到设置页面，URL 将如下所示：*https://example.com/login?redirect=https://example.com/settings*。自动重定向用户节省了时间并改善了他们的体验，因此你会发现许多应用程序都实现了这一功能。

在开放重定向攻击中，攻击者通过提供来自合法站点的 URL，将用户引导到外部网站，就像这样：*https://example.com/login?redirect=https://attacker.com*。像这样的 URL 可能会欺骗受害者点击链接，因为他们会相信它会带他们到合法站点 *example.com* 的页面。但实际上，这个页面会自动重定向到一个恶意页面。攻击者随后可以发起社交工程攻击，诱骗用户在攻击者网站上输入他们的 *example.com* 凭证。在网络安全领域，*社交工程* 是指通过欺骗手段攻击受害者。使用社交工程窃取凭证和私人信息的攻击被称为 *钓鱼*。

另一种常见的开放重定向技术是基于 referer 的开放重定向。*referer*是一个 HTTP 请求头，浏览器会自动包括它。它告诉服务器请求来自哪里。Referer 头是确定用户原始位置的常用方法，因为它包含指向当前页面的链接 URL。因此，一些网站在用户执行某些操作（如登录或注销）后，会自动重定向到页面的 referer URL。在这种情况下，攻击者可以托管一个链接到受害者网站的页面，以设置请求的 referer 头，HTML 代码如下所示：

```
<html> <a href="https://example.com/login">Click here to log in to example.com</a>
</html>
```

这个 HTML 页面包含一个`<a>`标签，该标签将文本链接到另一个位置。这个页面包含一个链接，文本为`Click here to log in to example.com`。当用户点击该链接时，他们将被重定向到`<a>`标签的`href`属性指定的位置，在这个例子中是*https://example.com/login*。

图 7-1 显示了在浏览器中渲染后的页面效果。

![f07001](img/f07001.png)

图 7-1：我们的示例渲染 HTML 页面

如果*example.com*使用基于 referer 的重定向系统，那么在用户访问*example.com*后，用户的浏览器会将其重定向到攻击者的网站，因为浏览器是通过攻击者的页面访问*example.com*的。

## 预防

为了防止开放重定向，服务器需要确保不会将用户重定向到恶意位置。网站通常会实现*URL 验证器*来确保用户提供的重定向 URL 指向合法位置。这些验证器使用黑名单或白名单来进行过滤。

当验证器实施黑名单时，它会检查重定向 URL 是否包含某些恶意重定向的标志，然后相应地阻止这些请求。例如，一个网站可能会将已知的恶意主机名或开放重定向攻击中常用的特殊 URL 字符列入黑名单。当验证器实施白名单时，它会检查 URL 中的主机名部分，确保它与预先确定的允许主机列表匹配。如果 URL 中的主机名部分与允许的主机名匹配，重定向会继续进行。否则，服务器会阻止该重定向。

这些防御机制听起来简单明了，但实际上，解析和解码 URL 是一个很难做到准确的任务。验证器通常很难识别 URL 中的主机名部分。这使得开放重定向成为现代 Web 应用中最常见的漏洞之一。我们将在本章稍后讨论攻击者如何利用 URL 验证问题绕过开放重定向保护。

## 寻找开放重定向

我们先从寻找一个简单的开放重定向开始。你可以通过使用一些侦察技巧来发现易受攻击的端点，并手动确认开放重定向。

### 步骤 1：查找重定向参数

从查找用于重定向的参数开始。这些参数通常会作为 URL 参数出现，如这里加粗的内容：

```
https://example.com/login?**redirect=https://example.com/dashboard**
https://example.com/login?**redir=https://example.com/dashboard**
https://example.com/login?**next=https://example.com/dashboard**
https://example.com/login?**next=/dashboard**
```

在浏览网站时打开你的代理。然后，在你的 HTTP 历史记录中，查找任何包含绝对或相对 URL 的参数。*绝对 URL* 是完整的，包含了定位其指向的资源所需的所有组件，如*https://example.com/login*。绝对 URL 至少包含 URL 协议、主机名和资源路径。*相对 URL* 必须通过服务器与另一个 URL 连接才能使用。它们通常仅包含 URL 的路径组件，如*/login*。某些重定向 URL 甚至会省略相对 URL 的第一个斜杠（`/`），如*https://example.com/login?next=dashboard*。

注意，并非所有的重定向参数都有像`redirect`或`redir`这样直接的名称。例如，我见过名为`RelayState`、`next`、`u`、`n`和`forward`的重定向参数。无论参数名称是什么，你都应该记录下所有看起来是用于重定向的参数。

此外，请注意那些 URL 中没有重定向参数，但仍然会自动重定向用户的页面。这些页面是基于 referer 的开放重定向的候选对象。要查找这些页面，你可以留意像 3*XX* 的响应代码，如 301 和 302。这些响应代码表示重定向。

### 步骤 2：使用 Google Dork 查找额外的重定向参数

Google Dork 技巧是一种高效的查找重定向参数的方法。要使用 Google Dork 查找目标网站上的重定向参数，首先将`site`搜索词设置为你的目标站点：

```
site:example.com
```

然后，查找包含 URL 参数中 URL 的页面，利用`%3D`，即等号（=）的 URL 编码版本。通过在搜索词中添加`%3D`，你可以搜索像`=http`和`=https`这样的词，它们是 URL 参数中 URL 的指示符。以下搜索会查找包含绝对 URL 的 URL 参数：

```
inurl:%3Dhttp site:example.com
```

这个搜索词可能会找到如下页面：

```
https://example.com/login?next=https://example.com/dashboard
https://example.com/login?u=http://example.com/settings
```

也可以尝试使用`%2F`，即斜杠（/）的 URL 编码版本。以下搜索词会查找包含`=/`的 URL，因此返回包含相对 URL 的 URL 参数：

```
inurl:%3D%2F site:example.com
```

这个搜索词将查找如下的 URL：

```
https://example.com/login?n=/dashboard
```

或者，你可以搜索常见 URL 重定向参数的名称。以下是一些搜索词，可能会揭示用于重定向的参数：

```
inurl:redir site:example.com
inurl:redirect site:example.cominurl:redirecturi site:example.com
inurl:redirect_uri site:example.com
inurl:redirecturl site:example.com
inurl:redirect_uri site:example.com
inurl:return site:example.com
inurl:returnurl site:example.com
inurl:relaystate site:example.com
inurl:forward site:example.com
inurl:forwardurl site:example.com
inurl:forward_url site:example.com
inurl:url site:example.com
inurl:uri site:example.com
inurl:dest site:example.com
inurl:destination site:example.com
inurl:next site:example.com
```

这些搜索词将查找如下的 URL：

```
https://example.com/logout?dest=/
https://example.com/login?RelayState=https://example.com/home
https://example.com/logout?forward=home
https://example.com/login?return=home/settings
```

请注意你发现的新参数，以及在步骤 1 中找到的那些参数。

### 步骤 3：测试基于参数的开放重定向

接下来，注意你找到的每个重定向参数的功能，并对每个进行开放重定向的测试。将一个随机的主机名，或者你拥有的主机名，插入重定向参数中；然后查看该站点是否自动重定向到你指定的站点：

```
https://example.com/login?n=http://google.com
https://example.com/login?n=http://attacker.com
```

一些网站会在你访问 URL 后立即重定向到目标站点，无需任何用户互动。但对于许多页面，重定向通常会在用户执行某些操作后才发生，比如注册、登录或注销。在这些情况下，确保在检查重定向之前，先执行所需的用户互动。

### 第 4 步：测试基于 Referer 的开放重定向

最后，测试在第 1 步中找到的任何页面，看看是否存在基于 referer 的开放重定向，即使这些页面没有包含重定向 URL 参数。要测试这些，可以在你拥有的域名上设置一个页面，并托管此 HTML 页面：

```
<html> <a href="https://example.com/login">Click on this link!</a>
</html>
```

替换链接 URL 为目标页面。然后重新加载并访问你的 HTML 页面。点击链接，看看是否会在自动或需要用户互动后重定向到你的站点。

## 绕过开放重定向保护

作为一个漏洞奖励猎人，我在攻击的几乎所有网站目标中都能发现开放重定向漏洞。为什么开放重定向在今天的 web 应用中仍然如此普遍？网站通过验证用于重定向用户的 URL 来防止开放重定向，开放重定向的根本原因是 URL 验证失败。不幸的是，URL 验证非常难以做到完全正确。

这里你可以看到一个 URL 的组成部分。浏览器如何重定向用户取决于浏览器如何区分这些组成部分：

```
scheme://userinfo@hostname:port/path?query#fragment
```

URL 验证器需要预测浏览器如何重定向用户，并拒绝那些会导致站外重定向的 URL。浏览器会将用户重定向到 URL 中主机名部分所指示的位置。然而，URL 不总是严格遵循此示例中所示的格式。它们可能会格式错误，组成部分顺序混乱，包含浏览器无法解码的字符，或缺少或多余的组成部分。例如，浏览器会如何重定向这个 URL？

https://user:password:8080/example.com@attacker.com

当你在不同的浏览器中访问这个链接时，你会发现不同的浏览器对这个 URL 的处理方式不同。有时验证器没有考虑到所有可能导致浏览器异常行为的边缘情况。在这种情况下，你可以尝试通过几种策略来绕过保护，我将在本节中讲解这些策略。

### 使用浏览器自动更正功能

首先，你可以使用浏览器自动更正功能来构造替代 URL，从而实现站外重定向。现代浏览器通常会自动更正那些没有正确组成部分的 URL，以纠正由于用户输入错误导致的 URL 错误。例如，Chrome 会将以下所有 URL 解释为指向 *https://attacker.com*：

```
https:attacker.com
https;attacker.com
https:\/\/attacker.com
https:/\/\attacker.com
```

这些小细节可以帮助你绕过基于黑名单的 URL 验证。例如，如果验证器拒绝任何包含 `https://` 或 `http://` 的重定向 URL，你可以使用一个替代字符串，如 `https;`，来实现相同的效果。

大多数现代浏览器还会自动将反斜杠（\）更正为斜杠（/），这意味着它们会将这些 URL 视为相同：

```
https:\\example.com
https://example.com
```

如果验证器未能识别这种行为，不一致性可能导致漏洞。例如，下面的 URL 可能存在问题：

```
https://attacker.com\@example.com
```

除非验证器将反斜杠视为路径分隔符，否则它会将主机名解释为 *example.com*，并将 *attacker.com\* 视为 URL 的用户名部分。但如果浏览器自动将反斜杠修正为正斜杠，它会将用户重定向到 *attacker.com*，并将 *@example.com* 视为 URL 的路径部分，从而形成以下有效的 URL：

```
https://attacker.com/@example.com
```

### 利用有缺陷的验证器逻辑

你还可以通过利用验证器逻辑中的漏洞绕过开放重定向验证器。例如，作为一种常见的防止开放重定向的措施，URL 验证器通常会检查重定向 URL 是否以站点的域名开始、包含或结束。你可以通过创建一个带有目标域名的子域或目录来绕过这种保护：

```
https://example.com/login?redir=**http://example.com.attacker.com**
https://example.com/login?redir=**http://attacker.com/example.com**
```

为了防止类似这些攻击的成功，验证器可能只接受那些既以允许列表上的域名开始又以该域名结束的 URL。然而，也有可能构造一个满足这两个规则的 URL。看看这个例子：

```
https://example.com/login?redir=**https://example.com.attacker.com/example.com**
```

这个 URL 会重定向到 *attacker.com*，尽管它的开始和结束都包含目标域名。浏览器会将第一个 *example.com* 解释为子域名，而将第二个解释为文件路径。

或者你可以使用 @ 符号，让第一个 *example.com* 成为 URL 的用户名部分：

```
https://example.com/login?redir=**https://example.com@attacker.com/example.com**
```

定制的 URL 验证器容易受到类似这些攻击的影响，因为开发人员常常没有考虑到所有的边缘情况。

### 使用数据 URL

你还可以操控 URL 的方案部分来欺骗验证器。如第六章所述，数据 URL 使用 `data:` 方案将小文件嵌入到 URL 中。它们的构造格式如下：

```
data:`MEDIA_TYPE`[;base64],`DATA`
```

例如，你可以发送一个带有数据方案的纯文本消息，格式如下：

```
data:text/plain,hello!
```

可选的 Base64 规范允许你发送 Base64 编码的消息。例如，这是前述消息的 Base64 编码版本：

```
data:text/plain;base64,aGVsbG8h
```

你可以使用 `data:` 方案构造一个 Base64 编码的重定向 URL，绕过验证器。例如，下面的 URL 会重定向到 *example.com*：

```
data:text/html;base64,
PHNjcmlwdD5sb2NhdGlvbj0iaHR0cHM6Ly9leGFtcGxlLmNvbSI8L3NjcmlwdD4=
```

该 URL 中编码的数据，*PHNjcmlwdD5sb2NhdGlvbj0iaHR0cHM6Ly9leGFtcGxlLmNvbSI8L3NjcmlwdD4=*，是该脚本的 Base64 编码版本：

```
<script>location="https://example.com"</script>
```

这是一个 JavaScript 代码片段，包裹在 HTML `<script>` 标签之间。它将浏览器的位置设置为 *https://example.com*，强制浏览器重定向到该位置。你可以将这个数据 URL 插入重定向参数中，从而绕过黑名单：

```
https://example.com/login?redir=data:text/html;base64,
PHNjcmlwdD5sb2NhdGlvbj0iaHR0cHM6Ly9leGFtcGxlLmNvbSI8L3NjcmlwdD4=
```

### 利用 URL 解码

通过互联网发送的 URL 只能包含 *ASCII* *字符*，这些字符包括常用于英语中的一组字符以及一些特殊字符。但由于 URL 经常需要包含特殊字符或其他语言的字符，人们通过使用 URL 编码对字符进行编码。URL 编码将字符转换为百分号，后跟两个十六进制数字；例如，`%2f`。这是斜杠字符 (`/`) 的 URL 编码版本。

当验证器验证 URL 或浏览器重定向用户时，它们必须首先通过解码任何 URL 编码的字符来确定 URL 中包含的内容。如果验证器和浏览器解码 URL 的方式存在任何不一致，你可以利用这一点来获得优势。

#### 双重编码

首先，尝试在你的 payload 中对某些特殊字符进行双重或三重 URL 编码。例如，你可以对 *https://example.com/@attacker.com* 中的斜杠字符进行 URL 编码。这里是一个带有 URL 编码斜杠的 URL：

```
https://example.com%2f@attacker.com
```

这是一个双重 URL 编码的斜杠的 URL：

```
https://example.com%252f@attacker.com
```

最后，这是一个三重 URL 编码的斜杠：

```
https://example.com%25252f@attacker.com
```

当验证器和浏览器在解码这些特殊字符时存在不匹配时，你可以利用这种不匹配来引发开放重定向。例如，一些验证器可能会完全解码这些 URL，然后假设该 URL 会重定向到*example.com*，因为 *@attacker.com* 位于 URL 的路径部分。然而，浏览器可能会解码不完全，而将 *example.com%25252f* 视为 URL 的用户名部分。

另一方面，如果验证器不对 URL 进行双重解码，而浏览器会，那么你可以使用像这样的 payload：

```
https://attacker.com%252f@example.com
```

验证器会将 *example.com* 视为主机名。但浏览器会重定向到 *attacker.com*，因为 *@example.com* 成为 URL 的路径部分，像这样：

```
https://attacker.com/@example.com
```

#### 非 ASCII 字符

你有时可以利用验证器和浏览器解码非 ASCII 字符时的不一致。例如，假设这个 URL 已通过 URL 验证：

```
https://attacker.com%ff.example.com
```

`%ff` 是字符 ÿ，这是一个非 ASCII 字符。验证器已确定 *example.com* 是域名，而 *attacker.comÿ* 是子域名。可能发生几种情况。有时浏览器会将非 ASCII 字符解码为问号。在这种情况下，*example.com* 将成为 URL 查询的一部分，而不是主机名，浏览器会导航到 *attacker.com*：

```
https://attacker.com?.example.com
```

另一个常见的情况是，浏览器会尝试查找“最相似”的字符。例如，如果字符 ╱ (`%E2%95%B1`) 出现在像这样的 URL 中，验证器可能会确定主机名是 *example.com*：

```
https://attacker.com╱.example.com
```

但浏览器将斜杠相似字符转换为实际的斜杠，使 *attacker.com* 成为主机名：

```
https://attacker.com/.example.com
```

浏览器通常会以这种方式标准化 URL，旨在提高用户友好性。除了使用类似的符号外，你还可以使用其他语言的字符集来绕过过滤器。*Unicode* 标准是一组为在计算机上表示世界所有语言而开发的代码集。你可以在[`www.unicode.org/charts/`](http://www.unicode.org/charts/)上找到 Unicode 字符的列表。利用 Unicode 图表查找相似字符并将其插入 URL 中，以绕过过滤器。*西里尔*字符集特别有用，因为它包含许多与 ASCII 字符相似的字符。

### 结合利用技巧

为了破解更复杂的 URL 验证器，可以结合多种策略来绕过层层防御。我发现以下载荷非常有用：

```
https://example.com%252f@attacker.com/example.com
```

这个 URL 绕过了仅检查 URL 是否包含、以某个允许列出的主机名开始或结束的保护，方法是使 URL 同时以*example.com*开头和结尾。大多数浏览器会将*example.com%252f*解释为 URL 的用户名部分。但如果验证器对 URL 进行过度解码，它会将*example.com*误认为是主机名部分：

```
https://example.com/@attacker.com/example.com
```

你可以使用更多方法来破解 URL 验证器。在本节中，我提供了最常见的一些方法概述。尝试每一种方法，检查你正在测试的验证器是否存在弱点。如果你有时间，可以尝试构造新的 URL，发明绕过 URL 验证器的新的方式。例如，尝试在 URL 中插入随机的非 ASCII 字符，或者故意搞乱它的不同部分，看看浏览器如何解释它。

## 升级攻击

攻击者可以通过自己使用开放重定向来使他们的网络钓鱼攻击更加可信。例如，他们可以将此 URL 发送到用户的电子邮件中：*https://example.com/login?next=https://attacker.com/fake_login.html*。

尽管这个 URL 最初会将用户引导到合法网站，但登录后会将他们重定向到攻击者的网站。攻击者可以在恶意网站上托管一个虚假的登录页面，模仿合法站点的登录页面，并通过类似下面这样的消息提示用户重新登录：

> 对不起！您提供的密码不正确。请重新输入您的用户名和密码。

用户可能会认为自己输入了错误的密码，从而将自己的凭据提供给攻击者的站点。这时，攻击者的站点甚至可以将用户重定向回合法站点，避免受害者意识到他们的凭据已被盗。

由于组织无法完全防止网络钓鱼攻击（因为这些攻击依赖于人为判断），安全团队通常会将开放重定向视为微不足道的漏洞，如果仅仅是单独报告时。然而，开放重定向往往能作为一系列漏洞中的一环，发挥更大的作用。例如，开放重定向可以帮助绕过 URL 阻止列表和白名单。例如，考虑这个 URL：

```
https://example.com/?next=https://attacker.com/
```

这个 URL 即使是经过精心实现的 URL 验证器也能通过，因为从技术上讲，URL 仍然在合法的网站上。因此，开放重定向可以帮助你最大化诸如服务器端请求伪造（SSRF）等漏洞的影响，我将在第十三章讨论。如果一个网站利用白名单来防止 SSRF，并只允许请求访问预定义的 URL 列表，攻击者可以利用在这些白名单页面中的开放重定向，将请求重定向到任何地方。

你还可以利用开放重定向来窃取凭证和 OAuth 令牌。通常，当页面重定向到另一个网站时，浏览器会将原始 URL 作为 referer HTTP 请求头包含在内。当原始 URL 包含敏感信息，如认证令牌时，攻击者可以通过诱导开放重定向来通过 referer 头窃取令牌。（即使在敏感端点没有开放重定向，也有方法通过使用开放重定向链将令牌转移到站外。关于这些攻击如何工作的详细信息，我将在第二十章讲解。）

## 寻找你的第一个开放重定向！

你已准备好寻找第一个开放重定向。按照本章介绍的步骤测试你的目标应用：

1.  搜索重定向 URL 参数。这些参数可能会对基于参数的开放重定向存在漏洞。

1.  搜索执行基于 referer 的重定向的页面。这些页面是基于 referer 的开放重定向的候选者。

1.  测试你找到的页面和参数，查看是否存在开放重定向。

1.  如果服务器阻止了开放重定向，尝试本章提到的保护绕过技术。

1.  思考如何在其他漏洞链中使用开放重定向！
