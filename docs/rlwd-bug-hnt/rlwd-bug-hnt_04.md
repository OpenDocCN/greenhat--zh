## 第四章：跨站请求伪造（**CSRF**）

![Image](img/common.jpg)

当攻击者能够使目标的浏览器向另一个网站发送 HTTP 请求时，就会发生*跨站请求伪造（CSRF）*攻击。该网站随后会执行一个操作，仿佛该请求是由目标发送且有效的。这种攻击通常依赖于目标已经在易受攻击的网站上进行身份验证，并且在目标不知情的情况下发生。当 CSRF 攻击成功时，攻击者能够修改服务器端信息，甚至可能接管用户账户。下面是一个基本示例，我们稍后会详细讲解：

1.  Bob 登录银行网站查看他的余额。

1.  完成后，Bob 检查他在另一个域名下的电子邮件账户。

1.  Bob 收到一封电子邮件，邮件中有一个链接指向一个不熟悉的网站，他点击该链接查看它的目的地。

1.  加载完成后，那个不熟悉的网站指示 Bob 的浏览器向 Bob 的银行网站发起 HTTP 请求，要求将他的账户中的钱转移到攻击者的账户。

1.  Bob 的银行网站接收到了来自不熟悉（且恶意）网站发起的 HTTP 请求。但由于银行网站没有任何 CSRF 防护，它处理了该转账请求。

### 身份验证

如我刚才描述的那样，CSRF 攻击利用了网站在验证请求时存在的漏洞。当您访问一个需要登录的网站时，通常会使用用户名和密码进行身份验证。网站会将您的身份验证信息存储在浏览器中，这样您在访问该网站的其他页面时就不需要每次都重新登录。它可以通过两种方式存储认证信息：使用基础认证协议或使用 cookie。

当 HTTP 请求中包含如下所示的头部时，您可以识别出一个使用基础认证的网站：`Authorization: Basic QWxhZGRpbjpPcGVuU2VzYW1l`。这个看似随机的字符串是经过 base64 编码的用户名和密码，中间用冒号分隔。在这种情况下，`QWxhZGRpbjpPcGVuU2VzYW1l` 解码为 `Aladdin:OpenSesame`。我们在本章不会专门讲解基础认证，但您可以使用本章介绍的许多技术来利用使用基础认证的 CSRF 漏洞。

*Cookies* 是网站创建并存储在用户浏览器中的小文件。网站使用 cookies 来实现多种目的，例如存储用户偏好或用户访问网站的历史记录。Cookies 有一些 *属性*，这些是标准化的信息。这些细节告诉浏览器如何处理 cookies。一些 cookie 属性可能包括 `domain`、`expires`、`max-age`、`secure` 和 `httponly`，你将在本章后面学习到这些属性。除了属性之外，cookies 还可以包含 *名称/值对*，该对由标识符和与之关联的值组成，并将其传递给网站（cookie 的 `domain` 属性定义了将此信息传递给哪个网站）。

浏览器定义了一个网站可以设置的 cookie 数量。但通常情况下，单个网站在常见浏览器中可以设置从 50 到 150 个 cookie，有些报告显示支持多达 600 个 cookie。浏览器通常允许每个 cookie 使用最多 4KB 的空间。对于 cookie 的名称和值没有标准：网站可以自由选择自己的名称/值对及用途。例如，一个网站可以使用名为 `sessionId` 的 cookie 来记住用户身份，而不需要用户在每次访问页面或执行操作时都输入用户名和密码。（回想一下，HTTP 请求是无状态的，正如在第一章中所描述的。无状态意味着每次 HTTP 请求时，网站都不知道用户是谁，因此必须为每次请求重新验证该用户。）

举个例子，cookie 中的一个名称/值对可能是 `sessionId=9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08`，该 cookie 的 `domain` 可能是 `.site.com`。因此，`sessionId` cookie 将会被发送到用户访问的每个 *.<site>.com* 网站，如 *foo.<site>.com*、*bar.<site>.com*、*www.<site>.com* 等等。

`secure` 和 `httponly` 属性告诉浏览器何时以及如何发送和读取 cookies。这些属性不包含值；相反，它们作为标志，可能出现在 cookie 中，也可能不出现。当一个 cookie 包含 `secure` 属性时，浏览器只有在访问 HTTPS 网站时才会发送该 cookie。例如，如果你访问 *http://www.<site>.com/*（一个 HTTP 网站），并且该网站有一个安全的 cookie，你的浏览器不会将该 cookie 发送到该网站。原因是为了保护你的隐私，因为 HTTPS 连接是加密的，而 HTTP 连接不是。`httponly` 属性将在你学习跨站脚本攻击（XSS）时变得重要，参见第七章，它指示浏览器只通过 HTTP 和 HTTPS 请求来读取 cookie。因此，浏览器不会允许任何脚本语言（如 JavaScript）读取该 cookie 的值。当 `secure` 和 `httponly` 属性没有在 cookie 中设置时，这些 cookie 可能会被合法发送，但被恶意读取。没有 `secure` 属性的 cookie 可以发送到非 HTTPS 网站；同样，未设置 `httponly` 的 cookie 可以被 JavaScript 读取。

`expires` 和 `max-age` 属性指示 cookie 何时过期以及浏览器何时销毁它。`expires` 属性简单地告诉浏览器在特定日期销毁一个 cookie。例如，cookie 可以设置属性为 `expires=Wed, 18 Dec 2019 12:00:00 UTC`。与此相对，`max-age` 是一个整数，表示 cookie 过期的秒数（例如，`max-age=300`）。

总结一下，如果鲍勃访问的银行网站使用了 cookies，该网站将通过以下过程存储他的身份验证信息。一旦鲍勃访问网站并登录，银行将以 HTTP 响应的形式回应他的 HTTP 请求，响应中包含一个标识鲍勃的 cookie。反过来，鲍勃的浏览器会自动将该 cookie 与所有其他 HTTP 请求一起发送到银行网站。

完成银行事务后，鲍勃没有在离开银行网站时登出。请注意这个重要细节，因为当你从一个网站登出时，该网站通常会以 HTTP 响应的形式回应，令你的 cookie 失效。结果，当你重新访问该网站时，你需要再次登录。

当鲍勃查看电子邮件并点击链接访问未知网站时，他无意中访问了一个恶意网站。该网站的设计目的是通过指示鲍勃的浏览器向银行网站发送请求来执行 CSRF 攻击。此请求也会从他的浏览器发送 cookies。

### 使用 GET 请求的 CSRF

恶意网站如何利用 Bob 的银行网站，取决于银行是否接受通过 `GET` 或 `POST` 请求进行转账。如果 Bob 的银行网站接受通过 `GET` 请求进行转账，恶意网站将通过隐藏表单或 `<img>` 标签发送 HTTP 请求。`GET` 和 `POST` 方法都依赖 HTML 使浏览器发送所需的 HTTP 请求，且两种方法都可以使用隐藏表单技术，但只有 `GET` 方法能够使用 `<img>` 标签技术。在本节中，我们将讨论使用 `GET` 请求方法时，攻击如何通过 HTML `<img>` 标签技术来实现，而关于隐藏表单技术的内容将在下一节 “使用 `POST` 请求的 CSRF” 中讨论。

攻击者需要在任何转账 HTTP 请求中包含 Bob 的 cookies。但由于攻击者无法读取 Bob 的 cookies，攻击者不能仅仅创建一个 HTTP 请求并将其发送到银行网站。相反，攻击者可以使用 HTML `<img>` 标签来创建一个 `GET` 请求，该请求也包含 Bob 的 cookies。`<img>` 标签在网页上渲染图像，并包含一个 `src` 属性，指示浏览器图像文件的位置。当浏览器渲染 `<img>` 标签时，它会发出一个 HTTP `GET` 请求到标签中的 `src` 属性，并在该请求中包含任何现有的 cookies。所以，假设恶意网站使用如下的 URL 将 $500 从 Bob 转账到 Joe：

```
https://www.bank.com/transfer?from=bob&to=joe&amount=500
```

然后，恶意的 `<img>` 标签会使用这个 URL 作为其源值，如下所示：

```
<img src="https://www.bank.com/transfer?from=bob&to=joe&amount=500">
```

因此，当 Bob 访问攻击者控制的网站时，该网站在 HTTP 响应中包含 `<img>` 标签，浏览器随后会发出 HTTP `GET` 请求到银行。浏览器发送 Bob 的认证 cookies，试图获取它认为应该是图像的内容。但实际上，银行收到请求后，会处理标签中 `src` 属性的 URL，并创建转账请求。

为了避免这种漏洞，开发者应该避免使用 HTTP `GET` 请求来执行任何会修改后端数据的操作，如转账。但是，任何只读请求应该是安全的。许多用于构建网站的常见 Web 框架，如 Ruby on Rails、Django 等，都会期望开发者遵循这一原则，因此它们会自动为 `POST` 请求添加 CSRF 保护，而不为 `GET` 请求添加保护。

### 使用 POST 请求的 CSRF

如果银行通过 `POST` 请求执行转账操作，你将需要使用不同的方法来创建 CSRF 攻击。攻击者不能使用 `<img>` 标签，因为 `<img>` 标签无法触发 `POST` 请求。相反，攻击者的策略将依赖于 `POST` 请求的内容。

最简单的情况是`POST`请求使用`application/x-www-form-urlencoded`或`text/plain`的内容类型。内容类型是浏览器在发送 HTTP 请求时可能包含的头部。这个头部告诉接收方 HTTP 请求正文是如何编码的。以下是一个`text/plain`内容类型请求的示例：

```
   POST / HTTP/1.1

   Host: www.google.ca

   User-Agent: Mozilla/5.0 (Windows NT 6.1; rv:50.0) Gecko/20100101 Firefox/50.0

   Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8

   Content-Length: 5

➊ Content-Type: text/plain;charset=UTF-8

   DNT: 1

   Connection: close

   hello
```

内容类型➊已标明，并列出了请求的类型以及字符编码。内容类型非常重要，因为浏览器对不同的类型有不同的处理方式（稍后我会详细说明）。

在这种情况下，恶意网站有可能创建一个隐藏的 HTML 表单，并在目标不知情的情况下悄悄地将其提交到易受攻击的网站。该表单可以提交一个`POST`或`GET`请求到一个 URL，甚至可以提交参数值。以下是恶意链接将 Bob 引导到的网站中的一些有害代码示例：

```
➊ <iframe style="display:none" name="csrf-frame"></iframe>

➋ <form method='POST' action='http://bank.com/transfer' target="csrf-frame"

   id="csrf-form">

  ➌ <input type='hidden' name='from' value='Bob'>

     <input type='hidden' name='to' value='Joe'>

     <input type='hidden' name='amount' value='500'>

     <input type='submit' value='submit'>

   </form>

➍ <script>document.getElementById("csrf-form").submit()</script>
```

在这里，我们正在向 Bob 的银行发送一个 HTTP `POST`请求➋，请求携带一个表单（由`<form>`标签中的`action`属性表示）。由于攻击者不希望 Bob 看到该表单，因此每个`<input>`元素➌的类型设置为`'hidden'`，使其在 Bob 看到的网页上不可见。最后，攻击者在`<script>`标签中包含一些 JavaScript 代码，自动提交表单，当页面加载时执行 ➍。JavaScript 通过调用 HTML 文档中的`getElementByID()`方法，并传入我们在第二行➋设置的表单 ID（`"csrf-form"`）作为参数来完成这一过程。与`GET`请求类似，一旦表单提交，浏览器会发送 HTTP `POST`请求，将 Bob 的 Cookies 发送到银行网站，从而触发转账。由于`POST`请求会将 HTTP 响应返回给浏览器，攻击者通过使用`display:none`属性➊将响应隐藏在 iFrame 中。结果，Bob 看不见这个响应，也没意识到发生了什么。

在其他情况下，网站可能期望`POST`请求使用`application/json`的内容类型提交。某些情况下，`application/json`类型的请求会包含一个*CSRF 令牌*。这个令牌是与 HTTP 请求一起提交的值，目的是让合法的网站验证该请求是来源于自身，而不是来自其他恶意网站。有时`POST`请求的 HTTP 正文中会包含令牌，但在其他情况下，`POST`请求会带有一个类似`X-CSRF-TOKEN`的自定义头。浏览器向网站发送`application/json POST`请求时，它会在`POST`请求之前先发送一个`OPTIONS` HTTP 请求。网站随后会返回一个响应，告知哪些类型的 HTTP 请求被接受，并指明哪些可信来源是被允许的。这被称为预检`OPTIONS`调用。浏览器读取这个响应后，再发送适当的 HTTP 请求，在我们银行的例子中，这将是一个用于转账的`POST`请求。

如果正确实施，预检 `OPTIONS` 请求可以防止一些 CSRF 漏洞：恶意网站不会被服务器列为受信任的网站，浏览器仅允许特定的网站（即*白名单网站*）读取 HTTP `OPTIONS` 响应。因此，由于恶意网站无法读取 `OPTIONS` 响应，浏览器也不会发送恶意的 `POST` 请求。

定义网站何时以及如何相互读取响应的规则集称为*跨域资源共享（CORS）*。CORS 限制了资源访问，包括来自外部域（即未提供文件或未被测试网站允许的域）的 JSON 响应访问。换句话说，当开发者使用 CORS 来保护网站时，你无法提交 `application/json` 请求去调用被测试的应用程序，读取响应并再次调用，除非被测试的网站允许这么做。在某些情况下，你可以通过将 `content-type` 头更改为 `application/x-www-form-urlencoded`、`multipart/form-data` 或 `text/plain` 来绕过这些保护。当使用这些三种内容类型发送 `POST` 请求时，浏览器不会发送预检的 `OPTIONS` 请求，因此 CSRF 请求可能会成功。如果没有成功，可以查看服务器 HTTP 响应中的 `Access-Control-Allow-Origin` 头，仔细检查服务器是否信任任意来源。如果该响应头在来自任意来源的请求发送时发生变化，则该站点可能存在更大的问题，因为它允许任何来源读取其服务器的响应。这不仅会导致 CSRF 漏洞，还可能允许恶意攻击者读取服务器 HTTP 响应中返回的任何敏感数据。

### 防御 CSRF 攻击

你可以通过多种方式缓解 CSRF 漏洞。防御 CSRF 攻击的最常见方法之一是使用 CSRF token。受保护的网站在提交可能会修改数据的请求时（即 `POST` 请求），会要求使用 CSRF token。在这种情况下，像 Bob 的银行这样的 Web 应用会生成一个包含两部分的 token：一部分 Bob 会收到，另一部分应用会保留。当 Bob 尝试进行转账请求时，他必须提交自己的 token，银行会将其与服务器端的 token 进行验证。这些 token 的设计使得它们无法被猜测，并且只能由分配给特定用户（如 Bob）的人访问。此外，它们的命名并不总是显而易见，但一些可能的命名示例包括 `X-CSRF-TOKEN`、`lia-token`、`rt` 或 `form-id`。这些 token 可以包含在 HTTP 请求头中，HTTP `POST` 请求体中，或作为隐藏字段，如以下示例所示：

```
<form method='POST' action='http://bank.com/transfer'>

  <input type='text' name='from' value='Bob'>

  <input type='text' name='to' value='Joe'>

  <input type='text' name='amount' value='500'>

  <input type='hidden' name='csrf' value='lHt7DDDyUNKoHCC66BsPB8aN4p24hxNu6ZuJA+8l+YA='>

  <input type='submit' value='submit'>

</form>
```

在这个示例中，网站可以从 Cookie、嵌入的脚本或作为网站内容的一部分获取 CSRF 令牌。无论使用哪种方法，只有目标的网页浏览器才能知道并读取该值。由于攻击者无法提交令牌，他们将无法成功提交 `POST` 请求，也无法执行 CSRF 攻击。然而，仅仅因为网站使用了 CSRF 令牌，并不意味着在寻找漏洞时就会走入死胡同。尝试移除令牌、修改其值等操作，以确认令牌是否已正确实现。

网站保护自己的一种方式是使用 CORS；然而，这并非万无一失，因为它依赖于浏览器的安全性，并确保适当的 CORS 配置，以确定何时第三方网站可以访问响应。攻击者有时可以通过将内容类型从 `application/json` 更改为 `application/x-www-form-urlencoded` 或使用 `GET` 请求而非 `POST` 请求来绕过 CORS，因为服务器端的配置可能存在问题。绕过之所以有效，是因为当内容类型为 `application/json` 时，浏览器会自动发送 `OPTIONS HTTP` 请求，但如果是 `GET` 请求或内容类型为 `application/x-www-form-urlencoded`，浏览器则不会自动发送 `OPTIONS HTTP` 请求。

最后，还有两种额外的、较少见的 CSRF 缓解策略。首先，网站可以检查提交的 HTTP 请求中的`Origin`或`Referer`头的值，并确保其包含预期的值。例如，在某些情况下，Twitter 会检查`Origin`头，如果该头未包含，会检查`Referer`头。之所以有效，是因为浏览器控制这些头部，攻击者无法远程设置或更改它们（显然，这不包括利用浏览器或浏览器插件中的漏洞来允许攻击者控制这些头部）。第二，浏览器现在开始实现对一种名为`samesite`的新 cookie 属性的支持。该属性可以设置为`strict`或`lax`。当设置为`strict`时，浏览器不会发送任何来自非本站点的 HTTP 请求的 cookie。这甚至包括简单的 HTTP `GET`请求。例如，如果你已登录 Amazon 并使用`strict samesite` cookies，当你从另一个网站点击链接时，浏览器不会提交你的 cookie。此时，Amazon 不会识别你为已登录状态，直到你访问另一个 Amazon 网页并提交了 cookie。相反，将`samesite`属性设置为`lax`指示浏览器在初始的`GET`请求中发送 cookie。这支持`GET`请求不应改变服务器端数据的设计原则。在这种情况下，如果你已登录 Amazon 并使用`lax samesite` cookies，当你从另一个网站被重定向到 Amazon 时，浏览器会提交你的 cookie，Amazon 会识别你为已登录状态。

### Shopify Twitter 断开连接

**难度：** 低

**URL：** *https://twitter-commerce.shopifyapps.com/auth/twitter/disconnect/*

**来源：** *[`www.hackerone.com/reports/111216/`](https://www.hackerone.com/reports/111216/)*

**报告日期：** 2016 年 1 月 17 日

**奖励支付：** $500

在寻找潜在的 CSRF 漏洞时，要留意那些会修改服务器端数据的`GET`请求。例如，一名黑客发现了一个 Shopify 功能的漏洞，该功能将 Twitter 集成到站点中，允许商店所有者发布关于其产品的推文。该功能还允许用户断开与连接商店的 Twitter 帐户的关联。断开 Twitter 帐户的 URL 如下所示：

```
https://twitter-commerce.shopifyapps.com/auth/twitter/disconnect/
```

事实证明，访问此 URL 会发送一个`GET`请求以断开账户，如下所示：

```
GET /auth/twitter/disconnect HTTP/1.1

Host: twitter-commerce.shopifyapps.com

User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.11; rv:43.0) Gecko/20100101 Firefox/43.0

Accept: text/html, application/xhtml+xml, application/xml

Accept-Language: en-US,en;q=0.5

Accept-Encoding: gzip, deflate

Referer: https://twitter-commerce.shopifyapps.com/account

Cookie: _twitter-commerce_session=REDACTED

Connection: keep-alive
```

此外，当该链接最初实施时，Shopify 并未验证发送给它的`GET`请求的合法性，这使得该 URL 容易受到 CSRF 攻击。

提交报告的黑客 WeSecureApp 提供了以下概念验证 HTML 文档：

```
<html>

  <body>

  ➊ <img src="https://twitter-commerce.shopifyapps.com/auth/twitter/disconnect">

  </body>

</html>
```

当打开时，这个 HTML 文档会导致浏览器通过 `<img>` 标签的 `src` 属性 ➊ 向 *https://twitter-commerce.shopifyapps.com* 发送一个 HTTP `GET` 请求。如果有一个连接到 Shopify 的 Twitter 账户访问了包含这个 `<img>` 标签的网页，他们的 Twitter 账户将会与 Shopify 断开连接。

#### *关键点*

留意那些执行某些服务器端操作的 HTTP 请求，例如通过 `GET` 请求断开 Twitter 账户。如前所述，`GET` 请求永远不应该修改服务器上的任何数据。在这种情况下，你可以通过使用代理服务器，例如 Burp 或 OWASP 的 ZAP，来监控发送到 Shopify 的 HTTP 请求，发现这个漏洞。

### 更改用户 Instacart 区域

**难度：** 低

**网址：** *https://admin.instacart.com/api/v2/zones/*

**来源：** *[`hackerone.com/reports/157993/`](https://hackerone.com/reports/157993/)*

**报告日期：** 2015 年 8 月 9 日

**奖励金额：** $100

当你查看攻击面时，记得考虑网站的 API 端点以及它的网页。Instacart 是一个食品配送应用程序，允许配送员定义他们工作的区域。该网站通过向 Instacart 管理子域发送 `POST` 请求来更新这些区域。一名黑客发现该子域上的区域端点存在 CSRF 漏洞。例如，你可以通过以下代码修改目标的区域：

```
<html>

  <body>

  ➊ <form action="https://admin.instacart.com/api/v2/zones" method="POST">

    ➋ <input type="hidden" name="zip" value="10001" />

    ➌ <input type="hidden" name="override" value="true" />

    ➍ <input type="submit" value="Submit request" />

    </form>

  </body>

</html>
```

在这个例子中，黑客创建了一个 HTML 表单，向 `/api/v2/zones` 端点发送了一个 HTTP `POST` 请求 ➊。黑客包括了两个隐藏输入框：一个用来将用户的新区域设置为邮政编码 `10001` ➋，另一个将 API 的 `override` 参数设置为 `true` ➌，这样用户当前的 `zip` 值就被黑客提交的值替换了。此外，黑客还包括了一个提交按钮来发起 `POST` 请求 ➍，不同于 Shopify 示例中使用的自动提交的 JavaScript 函数。

尽管这个例子依然成功，但黑客可以通过使用之前描述的技巧来改善这个漏洞利用方式，比如使用隐藏的 iFrame 来代表目标自动提交请求。这将向 Instacart 的漏洞奖励评审员展示攻击者如何使用这种漏洞，减少目标的操作；完全由攻击者控制的漏洞，比那些需要目标操作的漏洞更容易成功利用。

#### *关键点*

在寻找漏洞时，要拓宽攻击范围，不仅要关注网站页面，还要包括它的 API 端点，这些端点常常潜藏着巨大的漏洞风险。有时，开发者会忘记黑客可以发现并利用 API 端点，因为它们不像网页那样容易被发现。例如，移动应用程序通常会向 API 端点发送 HTTP 请求，你可以使用 Burp 或 ZAP 监控这些请求，就像监控网站一样。

### Badoo 完整账户接管

**难度：** 中

**网址：** *[`www.badoo.com/`](https://www.badoo.com/)*

**来源：** *[`hackerone.com/reports/127703/`](https://hackerone.com/reports/127703/)*

**报告日期：** 2016 年 4 月 1 日

**奖励支付：** $852

尽管开发者通常使用 CSRF 令牌来防止 CSRF 漏洞，但在某些情况下，攻击者可以窃取令牌，正如你在这个漏洞中看到的那样。如果你浏览社交网站 *[`www.badoo.com/`](https://www.badoo.com/)*，你会看到它使用了 CSRF 令牌。更具体地说，它使用了一个名为`rt`的 URL 参数，该参数对每个用户都是唯一的。当 Badoo 的漏洞奖励计划在 HackerOne 上线时，我无法找到利用它的方法。然而，黑客 Mahmoud Jamal 做到了。

Jamal 识别了`rt`参数及其重要性。他还注意到该参数几乎在所有的 JSON 响应中都会返回。不幸的是，这并没有提供帮助，因为 CORS 保护了 Badoo，防止攻击者读取这些响应，因为它们以`application/json`内容类型进行编码。但 Jamal 仍然继续深入挖掘。

Jamal 最终找到了包含名为`url_stats`的变量的 JavaScript 文件 *[`eu1.badoo.com/worker-scope/chrome-service-worker.js`](https://eu1.badoo.com/worker-scope/chrome-service-worker.js)*，该变量被设置为以下值：

```
var url_stats = 'https://eu1.badoo.com/chrome-push-stats?ws=1&rt=<➊rt_param_value>';
```

`url_stats`变量存储了一个 URL，其中包含用户唯一的`rt`值作为参数，当用户的浏览器访问该 JavaScript 文件时➊。更好的是，为了获取用户的`rt`值，攻击者只需要让目标访问一个恶意网页，该网页将访问该 JavaScript 文件。CORS 不会阻止这种情况，因为浏览器允许读取和嵌入来自外部源的远程 JavaScript 文件。攻击者随后可以使用`rt`值将任何社交媒体账户与用户的 Badoo 账户关联起来。因此，攻击者可以发起 HTTP `POST`请求来修改目标的账户。以下是 Jamal 用来实现这个漏洞的 HTML 页面：

```
<html>

  <head>

    <title>Badoo account take over</title>

  ➊ <script src=https://eu1.badoo.com/worker-scope/chrome-service-worker.

    js?ws=1></script>

  </head>

  <body>

    <script>

    ➋ function getCSRFcode(str) {

        return str.split('=')[2];

      }

    ➌ window.onload = function(){

      ➍ var csrf_code = getCSRFcode(url_stats);

      ➎ csrf_url = 'https://eu1.badoo.com/google/verify.phtml?code=4/nprfspM3y

        fn2SFUBear08KQaXo609JkArgoju1gZ6Pc&authuser=3&session_state=7cb85df679

        219ce71044666c7be3e037ff54b560..a810&prompt=none&rt='+ csrf_code;

      ➏ window.location = csrf_url;

      };

    </script>

  </body>

</html>
```

当目标加载此页面时，页面将通过在`<script>`标签的`src`属性中引用 Badoo 的 JavaScript 来加载它➊。加载了脚本后，网页会调用 JavaScript 函数`window.onload`，该函数定义了一个匿名 JavaScript 函数➌。浏览器在网页加载时会调用`onload`事件处理程序；由于 Jamal 定义的函数位于`window.onload`处理程序中，他的函数将在页面加载时始终被调用。

接下来，Jamal 创建了一个`csrf_code`变量➍，并将其赋值为他在➋处定义的一个名为`getCSRFcode`的函数的返回值。`getCSRFcode`函数接受一个字符串，并在每个`'='`字符处将其拆分成字符串数组。然后它返回数组的第三个成员。当该函数解析 Badoo 的漏洞 JavaScript 文件中的`url_stats`变量时，它将字符串拆分成以下数组值：

```
https://eu1.badoo.com/chrome-push-stats?ws,1&rt,<rt_param_value>
```

然后该函数返回数组的第三个成员，即`rt`值，并将其赋值给`csrf_code`。

一旦获得了 CSRF 令牌，Jamal 创建了 `csrf_url` 变量，该变量存储了指向 Badoo 的 */google/verify.phtml* 网页的 URL。该网页将他的 Google 账户与目标的 Badoo 账户链接起来 ➎。此页面需要一些参数，这些参数被硬编码到 URL 字符串中。我在这里不会详细讨论它们，因为它们是 Badoo 特有的。然而，请注意最后的 `rt` 参数，它没有硬编码的值。相反，`csrf_code` 被拼接到 URL 字符串的末尾，因此它作为 `rt` 参数的值被传递。然后，Jamal 通过调用 `window.location` ➏ 发起一个 HTTP 请求，并将其赋值给 `csrf_url`，从而将访问用户的浏览器重定向到 ➎ 处的 URL。这会导致向 Badoo 发起一个 `GET` 请求，Badoo 会验证 `rt` 参数并处理该请求，将目标的 Badoo 账户与 Jamal 的 Google 账户关联，从而完成账户接管。

#### *重点总结*

哪里有烟，哪里就有火。Jamal 注意到 `rt` 参数在不同的位置被返回，特别是在 JSON 响应中。因此，他正确地猜测 `rt` 可能会出现在攻击者可以访问并利用的位置，在这种情况下就是一个 JavaScript 文件。如果你觉得一个站点可能存在漏洞，继续深入检查。在这种情况下，我觉得奇怪的是 CSRF 令牌居然只有五个数字，并且包含在 URL 中。通常，令牌要长得多，这样更难猜测，而且应该包含在 HTTP `POST` 请求的主体中，而不是 URL 中。使用代理并检查访问站点或应用时调用的所有资源。Burp 允许你搜索代理历史记录中的特定术语或值，这可以帮助你发现这里的 JavaScript 文件中包含的 `rt` 值。你可能会发现敏感数据泄漏，例如 CSRF 令牌。

### 总结

CSRF 漏洞代表了攻击者可以在目标用户不知情或没有主动执行任何操作的情况下利用的另一种攻击向量。发现 CSRF 漏洞可能需要一些独创性和测试站点上所有功能的意愿。

通常，应用框架，如 Ruby on Rails，正在越来越多地保护 Web 表单，尤其是当站点执行 `POST` 请求时；但是，`GET` 请求并不在保护范围之内。因此，一定要注意任何可能更改服务器端用户数据的 `GET` HTTP 调用（例如断开 Twitter 账户连接）。另外，虽然我没有提供这个例子，但如果你看到站点通过 `POST` 请求发送 CSRF 令牌，你可以尝试更改 CSRF 令牌的值或完全删除它，以确保服务器在验证其存在性。
