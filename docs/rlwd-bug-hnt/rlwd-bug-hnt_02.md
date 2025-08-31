## 第二章：开放重定向**

![Image](img/common.jpg)

我们将从 *开放重定向* 漏洞开始讨论，当目标访问一个网站时，该网站将他们的浏览器重定向到一个不同的 URL，可能是一个独立的域名。开放重定向利用给定域名的信任，将目标引导到恶意网站。钓鱼攻击也可以伴随重定向，欺骗用户以为他们正在向一个可信站点提交信息，而实际上他们的信息正在被发送到恶意站点。当与其他攻击结合时，开放重定向还可以使攻击者从恶意站点分发恶意软件，或窃取 OAuth 令牌（我们将在 第十七章 中进一步探讨）。

由于开放重定向只是将用户重定向，因此它们有时被认为是低影响的，不值得奖励。例如，Google 的漏洞奖励计划通常认为开放重定向的风险过低，不值得奖励。专注于应用程序安全并整理 Web 应用程序中最关键安全漏洞列表的开放 Web 应用程序安全项目（OWASP）也在其 2017 年的十大漏洞列表中移除了开放重定向。

尽管开放重定向是低影响的漏洞，但它们对于学习浏览器如何处理重定向非常有帮助。在本章中，你将学习如何利用开放重定向漏洞，并通过三个漏洞报告的实例来识别关键参数。

### 开放重定向是如何工作的

开放重定向发生在开发者不信任攻击者控制的输入以进行重定向到另一个站点时，通常通过 URL 参数、HTML `<meta>` 刷新标签或 DOM window.location 属性。

许多网站故意通过在原始 URL 中放置目标 URL 作为参数，将用户重定向到其他站点。该应用程序使用此参数告诉浏览器向目标 URL 发送 `GET` 请求。例如，假设 Google 具有将用户重定向到 Gmail 的功能，只需访问以下 URL：

```
https://www.google.com/?redirect_to=https://www.gmail.com
```

在这个场景中，当你访问这个 URL 时，Google 会收到一个 `GET` HTTP 请求，并使用 `redirect_to` 参数的值来确定将浏览器重定向到哪里。完成后，Google 服务器返回一个 HTTP 响应，其中包含一个状态码，指示浏览器重定向用户。通常，状态码是 302，但在某些情况下，它可能是 301、303、307 或 308。这些 HTTP 响应码告诉浏览器某个页面已被找到；然而，该代码还告知浏览器对 `redirect_to` 参数的值发出 `GET` 请求，* [`www.gmail.com/`](https://www.gmail.com/) *，该值在 HTTP 响应的 `Location` 头中指定。`Location` 头指定了重定向 `GET` 请求的位置。

现在，假设攻击者将原始 URL 改为以下内容：

```
https://www.google.com/?redirect_to=https://www.attacker.com
```

如果 Google 没有验证 `redirect_to` 参数是否指向它自己的合法站点，攻击者就可以用自己的 URL 替换该参数。结果，HTTP 响应可能会指示浏览器发起对 *https://www.*<*attacker*>*.com/* 的 `GET` 请求。在攻击者将你引导到其恶意网站后，他们可以执行其他攻击。

在寻找这些漏洞时，留意包含某些名称的 URL 参数，如 `url=`, `redirect=`, `next=` 等，这些可能表示用户将被重定向到的 URL。同时请记住，重定向参数的名称不一定总是显而易见的；这些参数会根据网站的不同或同一网站内的不同部分而有所变化。在某些情况下，参数可能仅用单个字符表示，如 `r=` 或 `u=`。

除了基于参数的攻击外，HTML `<meta>` 标签和 JavaScript 也可以重定向浏览器。HTML `<meta>` 标签可以告诉浏览器刷新网页，并向标签的 `content` 属性中定义的 URL 发起 `GET` 请求。以下是一个可能的示例：

```
<meta http-equiv="refresh" content="0; url=https://www.google.com/">
```

`content` 属性定义了浏览器发起 HTTP 请求的两种方式。首先，`content` 属性定义了浏览器在发起 HTTP 请求到 URL 之前等待的时间；在此例中为 `0` 秒。其次，`content` 属性指定了浏览器发起 `GET` 请求时访问的网站 URL 参数；在此例中为 `https://www.google.com`。攻击者可以在他们能够控制 `<meta>` 标签的 `content` 属性或通过其他漏洞注入自己的标签时利用这种重定向行为。

攻击者还可以通过修改窗口的 `location` 属性来使用 JavaScript 重定向用户，方法是通过 *文档对象模型 (DOM)*。DOM 是用于 HTML 和 XML 文档的 API，它允许开发人员修改网页的结构、样式和内容。由于 `location` 属性指定了请求应重定向到的位置，浏览器会立即解析此 JavaScript 并重定向到指定的 URL。攻击者可以通过以下任一 JavaScript 修改窗口的 `location` 属性：

```
window.location = https://www.google.com/

window.location.href = https://www.google.com

window.location.replace(https://www.google.com)
```

通常，设置 `window.location` 值的机会仅在攻击者可以执行 JavaScript 时出现，无论是通过跨站脚本漏洞，还是在网站故意允许用户定义重定向 URL 的情况下，就像本章第 15 页 中详细介绍的 HackerOne 中继重定向漏洞一样。

当你搜索开放重定向漏洞时，通常会监控你的代理历史记录，寻找向你正在测试的网站发送的 `GET` 请求，这些请求包括指定 URL 重定向的参数。

### Shopify 主题安装开放重定向

**难度:** 低

**URL:** *https://apps.shopify.com/services/google/themes/preview/supply--blue?domain_name=<anydomain>*

**来源：** *[`www.hackerone.com/reports/101962/`](https://www.hackerone.com/reports/101962/)*

**报告日期：** 2015 年 11 月 25 日

**奖励金额：** $500

第一个开放重定向示例是在 Shopify 上发现的，Shopify 是一个允许人们创建商店销售商品的电商平台。Shopify 允许管理员通过更改主题来定制商店的外观和感觉。作为该功能的一部分，Shopify 提供了一个功能，通过将商店所有者重定向到一个 URL 来预览主题。重定向 URL 的格式如下：

```
https://app.shopify.com/services/google/themes/preview/supply--blue?domain_name=attacker.com
```

URL 末尾的 `domain_name` 参数将用户重定向到其商店域名，并在 URL 后面加上了 `/admin`。Shopify 本来预计 `domain_name` 始终会是用户的商店，并且没有验证其值是否属于 Shopify 域名的一部分。因此，攻击者可以利用该参数将目标重定向到 *http://<attacker>.com/admin/*，攻击者可以在该页面执行其他恶意攻击。

#### *总结*

不是所有的漏洞都很复杂。对于这个开放重定向，只需将 `domain_name` 参数更改为外部站点，即可将用户从 Shopify 重定向到其他网站。

### Shopify 登录开放重定向

**难度：** 低

**URL：** *[`mystore.myshopify.com/account/login/`](http://mystore.myshopify.com/account/login/)*

**来源：** *[`www.hackerone.com/reports/103772/`](https://www.hackerone.com/reports/103772/)*

**报告日期：** 2015 年 12 月 6 日

**奖励金额：** $500

第二个开放重定向示例与第一个 Shopify 示例类似，唯一不同的是，这次 Shopify 的参数并没有将用户重定向到 URL 参数指定的域名，而是将参数的值附加到 Shopify 子域名的末尾。通常，这个功能会用来将用户重定向到商店中的特定页面。然而，攻击者仍然可以通过添加字符来更改 URL 的含义，从而将浏览器从 Shopify 的子域名重定向到攻击者的网站。

在这个漏洞中，用户登录 Shopify 后，Shopify 使用 `checkout_url` 参数来重定向用户。例如，假设目标访问了这个 URL：

```
http://mystore.myshopify.com/account/login?checkout_url=.attacker.com
```

用户会被重定向到 URL *http://mystore.myshopify.com.<attacker>.com/*，而这并不是一个 Shopify 域名。

由于 URL 以 *.<attacker>.com* 结尾，并且 DNS 查询使用最右边的域名标签，因此重定向会指向 *<attacker>.com* 域名。所以当 *http://mystore.myshopify.com.<attacker>.com/* 被提交进行 DNS 查询时，它会匹配 *<attacker>.com*，而不是 Shopify 本来希望匹配的 *myshopify.com*。尽管攻击者不能随意将目标发送到任何地方，但他们可以通过向可操控的值中添加特殊字符（如句点），将用户重定向到另一个域名。

#### *总结*

如果你只能控制网站最终 URL 的一部分，添加特殊的 URL 字符可能会改变 URL 的含义，并将用户重定向到另一个域名。假设你只能控制`checkout_url`参数的值，并且你还注意到该参数与网站后端硬编码的 URL（例如商店 URL *[`mystore.myshopify.com/`](http://mystore.myshopify.com/)*）结合使用。尝试添加特殊的 URL 字符，如句点或@符号，测试是否可以控制重定向的位置。

### HackerOne 过渡重定向

**难度：** 低

**网址：** 暂无

**来源：** *[`www.hackerone.com/reports/111968/`](https://www.hackerone.com/reports/111968/)*

**报告日期：** 2016 年 1 月 20 日

**支付奖金：** $500

一些网站通过实现*过渡页面*来防止开放式重定向漏洞，过渡页面在预期内容之前显示。每当你将用户重定向到一个 URL 时，你可以显示一个过渡页面，向用户解释他们将离开当前域名。因此，如果重定向页面显示假登录页面或试图伪装成可信域名，用户将知道他们正在被重定向。这就是 HackerOne 在跟随其网站外的大多数 URL 时所采取的方法；例如，在跟随提交报告中的链接时。

尽管你可以使用过渡页面来避免重定向漏洞，但网站之间的交互复杂性可能会导致链接被篡改。HackerOne 使用 Zendesk，一个客户服务支持票务系统，来处理其*[`support.hackerone.com/`](https://support.hackerone.com/)*子域名。以前，当你跟随*[hackerone.com](http://hackerone.com)*并带有*/zendesk_session*时，浏览器会从 HackerOne 平台重定向到 HackerOne 的 Zendesk 平台，而没有过渡页面，因为包含*[hackerone.com](http://hackerone.com)*域名的 URL 被认为是可信链接。（现在，除非你通过 URL*/hc/en-us/requests/new*提交支持请求，否则 HackerOne 会将*[`support.hackerone.com`](https://support.hackerone.com)*重定向到*[docs.hackerone.com](http://docs.hackerone.com)*。）然而，任何人都可以创建自定义的 Zendesk 账户并将其传递给`/redirect_to_account?state=`参数。然后，那个自定义 Zendesk 账户可能会重定向到 Zendesk 或 HackerOne 没有拥有的其他网站。由于 Zendesk 允许在账户之间进行重定向而没有过渡页面，用户可能会在没有警告的情况下被带到不受信任的网站。作为解决方案，HackerOne 将包含`zendesk_session`的链接标识为外部链接，因此在点击时会显示过渡警告页面。

为了确认这个漏洞，黑客 Mahmoud Jamal 在 Zendesk 上创建了一个帐户，使用了子域名*[`compayn.zendesk.com`](http://compayn.zendesk.com)*。然后，他通过 Zendesk 主题编辑器将以下 JavaScript 代码添加到头文件中，该编辑器允许管理员自定义他们的 Zendesk 站点外观和感觉：

```
<script>document.location.href = «http://evil.com»;</script>
```

使用这段 JavaScript，Jamal 指示浏览器访问*[`evil.com`](http://evil.com)*。`<script>`标签表示 HTML 中的代码，而`document`指的是 Zendesk 返回的整个 HTML 文档，这是网页的内容。紧随`document`的点和名称是它的属性。属性保存信息和数值，这些信息要么描述对象，要么可以被操作以改变对象。因此，你可以使用`location`属性来控制浏览器显示的网页，并使用`href`子属性（它是`location`的一个属性）来重定向浏览器到定义的网站。访问以下链接会将目标重定向到 Jamal 的 Zendesk 子域名，这使得目标的浏览器执行 Jamal 的脚本，并将其重定向到*[`evil.com`](http://evil.com)*：

```
https://hackerone.com/zendesk_session?locale_id=1&return_to=https://support.hackerone.com/

ping/redirect_to_account?state=compayn:/
```

因为链接包含了域名*[hackerone.com](http://hackerone.com)*，所以过渡网页并没有显示出来，用户也不会知道他们访问的页面是不安全的。有趣的是，Jamal 最初向 Zendesk 报告了这个缺失的过渡页面重定向问题，但它被忽视了，并未标记为漏洞。自然，他继续深入挖掘，看看缺失的过渡页面如何被利用。最终，他找到了 JavaScript 重定向攻击，这使得 HackerOne 决定支付他赏金。

#### *关键点*

在寻找漏洞时，请注意网站使用的服务，因为每一个都代表了新的攻击路径。这个 HackerOne 漏洞正是通过结合 HackerOne 使用 Zendesk 和 HackerOne 允许的已知重定向漏洞得以实现的。

此外，当你发现漏洞时，有时阅读和回应你报告的人可能无法立即理解其中的安全影响。因此，我将在第十九章中讨论漏洞报告，其中详细介绍了你应在报告中包含的发现、如何与公司建立关系以及其他信息。如果你在前期做了一些工作，并且在报告中尊重地解释了安全影响，你的努力将有助于确保问题的顺利解决。

话虽如此，有时公司可能不同意你的看法。如果是这种情况，继续像 Jamal 一样深入挖掘，看看你是否能证明漏洞的存在，或者将其与另一个漏洞结合，展示其影响。

### 总结

开放重定向允许恶意攻击者在受害者不知情的情况下将其重定向到恶意网站。正如你从示例漏洞报告中学到的，发现这些漏洞通常需要敏锐的观察力。重定向参数有时很容易被发现，尤其是它们的名字像 `redirect_to=`、`domain_name=` 或 `checkout_url=`，如示例中所提到的。其他时候，它们可能有不那么明显的名字，例如 `r=`、`u=` 等等。

开放重定向漏洞依赖于对信任的滥用，攻击者通过让目标访问一个他们认为是熟悉的站点，实际上却是攻击者的站点。当你发现可能存在漏洞的参数时，务必彻底测试它们，并在 URL 的某些部分硬编码时加入特殊字符，如句号。

HackerOne 的过渡重定向展示了在寻找漏洞时，识别网站使用的工具和服务的重要性。记住，有时你需要保持耐心，并清楚地展示漏洞，才能说服公司接受你的发现并支付奖励。
