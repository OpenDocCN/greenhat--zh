## 第十六章：不安全的直接对象引用（IDOR）**

![Image](img/common.jpg)

*不安全的直接对象引用（IDOR）* 漏洞发生在攻击者能够访问或修改他们本不应访问的对象引用时，例如文件、数据库记录、账户等。例如，假设网站 *www.<example>.com* 上有私人用户资料，只有资料拥有者能够通过 URL *www.<example>.com/user?id=1* 访问该资料。`id` 参数决定了您查看的是哪个资料。如果您能够通过更改 `id` 参数为 2 来访问其他人的资料，那么这就是一个 IDOR 漏洞。

### 查找简单的 IDOR 漏洞

一些 IDOR 漏洞比其他的更容易发现。您会发现最容易的 IDOR 漏洞类似于前面的例子：其中标识符是一个简单的整数，并且随着新记录的创建而自动递增。要测试这种类型的 IDOR，您只需对 `id` 参数加 1 或减 1，并确认您是否能够访问本不应访问的记录。

您可以使用 Web 代理工具 Burp Suite 进行此测试，详细信息请参见 附录 A。*Web 代理* 捕获浏览器发送到网站的流量。Burp 允许您监视 HTTP 请求，实时修改它们，并重放请求。要测试 IDOR，您可以将请求发送到 Burp 的 Intruder，设置 `id` 参数的有效载荷，并选择一个数值有效载荷来递增或递减。

启动 Burp Intruder 攻击后，您可以通过检查 Burp 接收到的内容长度和 HTTP 响应代码来判断是否能够访问数据。例如，如果您测试的站点始终返回状态码 403 且所有响应的内容长度相同，那么该站点可能没有漏洞。状态码 403 表示访问被拒绝，因此一致的内容长度表明您收到的是标准的访问拒绝信息。但如果您收到状态码 200 响应并且内容长度可变，那么您可能已经访问了私人记录。

### 查找更复杂的 IDOR 漏洞

复杂的 IDOR 漏洞可能发生在 `id` 参数被隐藏在 `POST` 请求体中，或者通过参数名称无法轻易识别的情况下。您可能会遇到不明显的参数，例如 `ref`、`user` 或 `column`，它们被用作 ID。即使您无法通过参数名称轻易识别出 ID，当参数取整数值时，您可能能够识别出该参数。找到一个取整数值的参数后，测试它，看看在修改 ID 时站点的行为如何变化。同样，您可以使用 Burp 来简化这一过程，通过拦截 HTTP 请求、更改 ID，并使用 Repeater 工具重放请求。

当站点使用随机化标识符时，IDOR 漏洞更难识别，比如 *通用唯一标识符（UUID）*。UUID 是由 36 个字符组成的字母数字字符串，没有固定的模式。如果你发现一个使用 UUID 的站点，通过测试随机值几乎不可能找到有效的记录或对象。相反，你可以创建两个记录，并在测试过程中在它们之间切换。例如，假设你正在尝试访问使用 UUID 标识的用户资料。首先使用用户 A 创建个人资料；然后以用户 B 登录，尝试使用用户 A 的 UUID 访问其个人资料。

在某些情况下，你可能能够访问使用 UUID 的对象。但站点可能不会认为这是一种漏洞，因为 UUID 被设计为不可猜测。在这种情况下，你需要寻找站点透露该随机标识符的机会。假设你在一个基于团队的网站上，用户是通过 UUID 进行标识的。当你邀请某个用户加入你的团队时，邀请的 HTTP 响应可能会泄露他们的 UUID。在其他情况下，你可能能够在网站上搜索记录，并返回包括 UUID 的结果。如果你无法找到显而易见的地方泄露了 UUID，那么请检查 HTTP 响应中包含的 HTML 页面源代码，这可能会透露一些在网站上不易察觉的信息。你可以通过在 Burp 中监控请求，或通过右键点击浏览器并选择“查看页面源代码”来进行此操作。

即使你找不到泄露的 UUID，一些站点也会奖励这种漏洞，如果该信息是敏感的，并且明显违反了它们的权限模型。你有责任向公司解释为什么你认为发现了他们应该处理的问题，以及你确定漏洞的影响。以下示例展示了找到 IDOR 漏洞的难度范围。

### [Binary.com](http://Binary.com) 权限提升

**难度:** 低

**网址:** *[www.binary.com](http://www.binary.com)*

**来源:** *[`hackerone.com/reports/98247/`](https://hackerone.com/reports/98247/)*

**报告日期:** 2015 年 11 月 6 日

**奖励金额:** $300

当你在测试使用账户的 web 应用程序时，应该注册两个不同的账户并同时进行测试。这样可以帮助你测试两个你控制的账户之间的 IDOR（不正确的对象引用）漏洞，并了解预期的结果。这就是 Mahmoud Gamal 在发现 *[binary.com](http://binary.com)* 中的 IDOR 漏洞时所采取的方法。

网站 *[binary.com](http://binary.com)* 是一个交易平台，允许用户交易货币、指数、股票和商品。在本报告发布时，URL *www.binary.com/cashier* 会呈现一个 iFrame，其`src`属性引用了子域 *cashier.binary.com* 并将如`pin`、`password`和`secret`等 URL 参数传递给该网站。这些参数可能是用于验证用户身份的。由于浏览器正在访问 *www.binary.com/cashier*，因此传递给 *cashier.binary.com* 的信息在不查看网站发送的 HTTP 请求的情况下是不可见的。

Gamal 注意到`pin`参数被用作帐户标识符，并且它似乎是一个容易猜测的递增数字整数。使用两个不同的帐户，我们称之为帐户 A 和帐户 B，他访问了帐户 A 的*/cashier*路径，记下了`pin`参数，然后登录到帐户 B。当他修改帐户 B 的 iFrame 以使用帐户 A 的 pin 时，他能够访问帐户 A 的信息并在以帐户 B 身份验证的情况下请求提款。

*[binary.com](http://binary.com)* 的团队在收到报告后的当天就解决了该问题。他们声称他们手动审查并批准提款，因此会注意到可疑的活动。

#### *重点总结*

在这种情况下，一名黑客通过在登录为不同帐户时使用一个帐户的客户 PIN 轻松地手动测试了漏洞。你还可以使用 Burp 插件，如 Autorize 和 Authmatrix，来自动化这种类型的测试。

但是，发现模糊的 IDOR 漏洞可能更困难。该站点使用了 iFrame，这可能导致易于忽略易受攻击的 URL 及其参数，因为如果不查看 HTML 页面源代码，你是看不到它们的。跟踪 iFrame 和多个 URL 可能通过单个网页访问的情况的最佳方法是使用像 Burp 这样的代理。Burp 将记录任何对其他 URL 的`GET`请求，例如*cashier.binary.com*，并将其保存在代理历史中，使你更容易捕捉到这些请求。

### Moneybird 应用创建

**难度：** 中等

**网址：** *[`moneybird.com/user/applications/`](https://moneybird.com/user/applications/)*

**来源：** *[`hackerone.com/reports/135989/`](https://hackerone.com/reports/135989/)*

**报告日期：** 2016 年 5 月 3 日

**奖励支付：** $100

2016 年 5 月，我开始对 Moneybird 进行漏洞测试，重点关注其用户帐户权限。为此，我创建了一个帐户 A 的企业，并邀请了一个第二个用户帐户 B，以有限的权限加入。Moneybird 定义了它为新增用户分配的权限，例如使用发票、估算等的权限。

拥有完全权限的用户可以创建应用并启用 API 访问。例如，用户可以提交一个`POST`请求来创建一个具有完全权限的应用，格式如下：

```
POST /user/applications HTTP/1.1

Host: moneybird.com

User-Agent: Mozilla/5.0 (Windows NT 6.1; rv:45.0) Gecko/20100101 Firefox/45.0

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8

Accept-Language: en-US,en;q=0.5

Accept-Encoding: gzip, deflate, br

DNT: 1

Referer: https://moneybird.com/user/applications/new

Cookie: _moneybird_session=REDACTED; trusted_computer=

Connection: close

Content-Type: application/x-www-form-urlencoded

Content-Length: 397

utf8=%E2%9C%93&authenticity_token=REDACTED&doorkeeper_application%5Bname%5D=TW

DApp&token_type=access_token&➊administration_id=ABCDEFGHIJKLMNOP&scopes%5B%5D

=sales_invoices&scopes%5B%5D=documents&scopes%5B%5D=estimates&scopes%5B%5D=ban

k&scopes%5B%5D=settings&doorkeeper_application%5Bredirect_uri%5D=&commit=Save
```

如你所见，`POST`请求体中包含了`administration_id` ➊参数。这是用户被添加到的账户 ID。尽管 ID 的长度和随机性使其难以猜测，但当用户访问邀请他们的账户时，ID 会立即被泄露。例如，当账户 B 登录并访问账户 A 时，他们会被重定向到 URL *https://moneybird.com/ABCDEFGHIJKLMNOP/*，其中`ABCDEFGHIJKLMNOP`就是账户 A 的`administration_id`。

我测试了账户 B 是否能在没有适当权限的情况下，为账户 A 的业务创建应用程序。我以账户 B 登录并创建了一个第二个业务，这个业务只有账户 B 是唯一成员。这样，账户 B 就会对第二个业务拥有完全权限，即使账户 B 本应只对账户 A 拥有有限权限，且无法为其创建应用程序。

接下来，我访问了账户 B 的设置页面，创建了一个应用，并使用 Burp Suite 拦截了`POST`调用，将`administration_id`替换为账户 A 的 ID。转发修改后的请求确认了该漏洞有效。作为账户 B，我拥有了一个完全权限的应用，能够绕过账户 B 的权限限制，使用新创建的应用执行原本无法访问的任何操作。

#### *要点*

寻找可能包含 ID 值的参数，例如任何名称中包含`id`字符的参数。特别注意那些仅包含数字的参数值，因为这些 ID 可能是以某种可猜测的方式生成的。如果无法猜测一个 ID，确定它是否在某处被泄露。我注意到`administrator_id`因为它名称中有 ID 的引用。尽管 ID 值没有遵循可猜测的模式，但每当用户被邀请到公司时，该值会通过 URL 泄露。

### Twitter Mopub API 令牌盗窃

**难度：** 中等

**URL：** *https://mopub.com/api/v3/organizations/ID/mopub/activate/*

**来源：** *[`hackerone.com/reports/95552/`](https://hackerone.com/reports/95552/)*

**报告日期：** 2015 年 10 月 24 日

**奖金支付：** $5,040

发现漏洞后，请务必考虑如果攻击者利用此漏洞，会产生什么影响。2015 年 10 月，Akhil Reni 报告了 Twitter 的 Mopub 应用（2013 年收购）存在 IDOR 漏洞，会在`POST`响应中泄露 API 密钥和密钥。但是几周后，Reni 意识到漏洞比他最初报告的要严重，并提交了更新。幸运的是，他在 Twitter 为此漏洞支付奖金之前做了更新。

当 Reni 最初提交报告时，他发现 Mopub 的一个端点没有正确授权用户，并且会在`POST`响应中泄露账户的 API 密钥和`build_secret`。以下是`POST`请求的样子：

```
POST /api/v3/organizations/5460d2394b793294df01104a/mopub/activate HTTP/1.1

Host: fabric.io

User-Agent: Mozilla/5.0 (Windows NT 6.3; WOW64; rv:41.0) Gecko/20100101

Firefox/41.0

Accept: */*

Accept-Language: en-US,en;q=0.5

Accept-Encoding: gzip, deflate

X-CSRF-Token: 0jGxOZOgvkmucYubALnlQyoIlsSUBJ1VQxjw0qjp73A=

Content-Type: application/x-www-form-urlencoded; charset=UTF-8

X-CRASHLYTICS-DEVELOPER-TOKEN: 0bb5ea45eb53fa71fa5758290be5a7d5bb867e77

X-Requested-With: XMLHttpRequest

Referer: https://fabric.io/img-srcx-onerrorprompt15/android/apps/app

.myapplication/mopub

Content-Length: 235

Cookie: <redacted>

Connection: keep-alive

Pragma: no-cache

Cache-Control: no-cache

company_name=dragoncompany&address1=123 street&address2=123&city=hollywood&

state=california&zip_code=90210&country_code=US&link=false
```

对该请求的响应如下：

```
{"mopub_identity":{"id":"5496c76e8b15dabe9c0006d7","confirmed":true,"primary":

false,"service":"mopub","token":"35592"},➊"organization":{"id":"5460d2394b793

294df01104a","name":"test","alias":"test2",➋"api_key":"8590313c7382375063c2fe

279a4487a98387767a","enrollments":{"beta_distribution":"true"},"accounts

_count":3,"apps_counts":{"android":2},"sdk_organization":true,➌"build

_secret":"5ef0323f62d71c475611a635ea09a3132f037557d801503573b643ef8ad82054",

"mopub_id":"33525"}}
```

Mopub 的`POST`响应提供了`api_key` ➋和`build_secret` ➌，这些信息 Reni 在初次报告中已经报告给了 Twitter。但访问这些信息还需要知道一个`organization_id` ➊，这是一个无法猜测的 24 位数字字符串。Reni 注意到用户可以通过一个 URL 公开分享应用崩溃问题，例如 *http://crashes.to/s/<11 个字符>*。访问这些 URL 中的任何一个将会在响应体中返回无法猜测的`organization_id`。Reni 通过使用 Google dork *site:http://crashes.to/s/*访问这些 URL，枚举了`organization_id`值。通过`api_key`、`build_secret`和`organization_id`，攻击者可以窃取 API 令牌。

Twitter 解决了该漏洞，并要求 Reni 确认他是否仍然能够访问这些敏感信息。正是在这一点上，Reni 发现 HTTP 响应中返回的`build_secret`也被用在了 URL *https://app.mopub.com/complete/htsdk/?code=*<*BUILDSECRET>&amp;next=%2d*。这个 URL 验证了用户身份，并将他们重定向到关联的 Mopub 账户，这使得恶意用户能够登录任何其他用户的账户。恶意用户将能够访问目标账户在 Twitter 移动开发平台上的应用和组织。Twitter 回应了 Reni 的评论，要求提供更多的信息和复现攻击的步骤，Reni 也提供了这些信息。

#### *要点*

始终确保确认你所发现漏洞的全部影响，尤其是当涉及到 IDOR 时。在这个案例中，Reni 发现他可以通过访问`POST`请求并使用一个简单的 Google dork 来获取敏感的秘密值。Reni 最初报告说 Twitter 泄露了敏感信息，但他后来才意识到这些值在平台上的使用方式。如果 Reni 在提交报告后没有提供额外的信息，Twitter 可能不会意识到他们容易受到账户接管的威胁，赏金也许会更少。

### ACME 客户信息泄露

**难度：** 高

**URL：** *https://www.<acme>.com/customer_summary?customer_id=abeZMloJyUovapiXqrHyi0DshH*

**来源：** 不适用

**报告日期：** 2017 年 2 月 20 日

**支付赏金：** $3,000

这个漏洞是 HackerOne 上的一个私有项目的一部分。此漏洞未公开，所有信息已被匿名化。

一家公司，我在这里称之为 ACME 公司，为管理员创建用户并分配权限的软件。当我开始测试该软件的漏洞时，我使用管理员账户创建了一个没有权限的第二个用户。使用第二个用户账户时，我开始访问管理员本应有权限访问，但第二个用户不该访问的 URL。

使用我的无权限账户，我通过 URL 访问了一个客户详情页，URL 为*www.<acme>.com/customization/customer_summary?customer_id=abeZMloJyUovapiXqrHyi0DshH*。这个 URL 根据传递给`customer_id`参数的 ID 返回客户信息。我惊讶地发现，第二个用户账户能够看到客户详情。

尽管`customer_id`看似无法猜测，但它可能会在网站某处被错误地泄露。或者，如果一个用户被撤销了权限，只要知道`customer_id`，他们仍然能够访问客户信息。我以这个理由报告了这个漏洞。事后想来，我应该在报告之前就寻找泄露的`customer_id`。

该程序将我的报告标记为信息性报告，理由是`customer_id`无法猜测。信息性报告不会带来奖励，并可能对你的 HackerOne 统计数据产生负面影响。尽管如此，我仍然开始寻找可能泄露 ID 的地方，通过测试我能找到的所有端点。两天后，我发现了一个漏洞。

我开始通过一个仅有权限搜索订单的用户访问 URL，该用户本不应能访问客户或产品信息。但我发现订单搜索的响应中返回了以下 JSON：

```
{

  "select": "(*,hits.(data.(order_no, customer_info, product_items.(product_

id,item_text), status, creation_date, order_total, currency)))",

  "_type": "order_search_result",

  "count": 1,

  "start": 0,

  "hits": [{

    "data": {

      "order_no": "00000001",

      "product_items": [{

        "_type": "product_item",

        "product_id": "test1231234",

        "item_text": "test"

      }],

      "_type": "order",

      "creation_date": "2017-02-25T02:31Z",

      "customer_info": {

        "customer_no": "00006001",

        "_type": "customer_info",

        "customer_name": "pete test",

        "customer_id": "abeZMloJyUovapiXqHyi0DshH",

        "email": "test@gmail.com"

      }

    }

  }]

}--snip--
```

请注意，JSON 中包括了`customer_id` ➊，这个 ID 与在 URL 中用于显示客户信息的 ID 相同。这意味着客户 ID 正在泄露，无权限的用户可能会找到并访问他们本不应该看到的客户信息。

除了找到`customer_id`，我还继续调查漏洞的范围。我发现了其他 ID，这些 ID 也可以在 URL 中使用，返回本应无法访问的信息。我的第二份报告被接受，并获得了奖励。

#### *总结*

当你发现一个漏洞时，确保了解攻击者能够利用它的范围。尝试寻找泄露的标识符或其他可能存在类似漏洞的 ID。此外，如果一个程序不同意你的报告，不要灰心。你可以继续寻找其他可能利用该漏洞的地方，如果发现任何进一步的信息，可以提交另一个报告。

### 概述

IDOR（不安全直接对象引用）发生在攻击者可以访问或修改他们不应该能够访问的对象引用时。IDOR 可以很简单：它们可能只需要通过加减 1 来利用数值递增的整数。对于更复杂的 IDOR，使用 UUID 或随机标识符时，可能需要彻底测试平台以查找泄漏。你可以在多个地方检查泄漏，例如在 JSON 响应中、HTML 内容中、通过 Google dorks 或通过 URLs 中。当你报告时，务必详细说明攻击者如何滥用此漏洞。例如，攻击者能够绕过平台权限的漏洞，其悬赏金额会低于导致完整账户接管的漏洞的悬赏金额。
