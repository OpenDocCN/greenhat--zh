# 第二十四章：API 黑客技术

![](img/chapterart.png)

*应用程序编程接口（**APIs**）* 是程序之间进行通信的一种方式，它们为各种应用程序提供动力。随着应用程序变得越来越复杂，开发者越来越多地使用 API 来组合应用程序的组件或属于同一组织的多个应用程序。并且，越来越多的 API 具备执行重要操作或传输敏感信息的能力。

在本章中，我们将讨论什么是 API，API 如何工作，以及你如何发现和利用 API 漏洞。

## 什么是 API？

简单来说，API 是一组规则，允许一个应用程序与另一个应用程序进行通信。它们使得应用程序能够以受控的方式共享数据。通过使用 API，互联网上的应用程序可以利用其他应用程序的资源，构建更复杂的功能。

例如，考虑一下 Twitter 的 API（[`developer.twitter.com/en/docs/twitter-api/`](https://developer.twitter.com/en/docs/twitter-api/)）。这个公共 API 允许外部开发者访问 Twitter 的数据和操作。例如，如果开发者希望他们的代码从 Twitter 的数据库中检索推文内容，他们可以使用一个 Twitter API 端点，通过向位于 *api.twitter.com* 的 Twitter API 服务器发送 GET 请求来返回推文信息：

```
GET /1.1/statuses/show.json?id=210462857140252672
Host: api.twitter.com
```

这个 URL 表示开发者正在使用 Twitter 的 API 版本 1.1，并请求一个名为 `statuses`（即 Twitter 的推文）的资源，ID 为 210462857140252672。URL 中的 `id` 字段是 API 端点所要求的请求参数。API 端点通常需要某些参数来确定返回哪个资源。

然后，Twitter 的 API 服务器会以 JSON 格式将数据返回给请求的应用程序（这个示例取自 Twitter 的公共 API 文档）：

```
1 {2 "created_at": "Wed Oct 10 20:19:24 +0000 2018", "id": 1050118621198921728, "id_str": "1050118621198921728", "text": "To make room for more expression, we will now count all emojis 
as equal—including those with gender... and skin t... https://t.co/MkGjXf9aXm", "truncated": true, "entities": { 3 "hashtags": [], "symbols": [], "user_mentions": [], "urls": [ { "url": "https://t.co/MkGjXf9aXm", "expanded_url": "https://twitter.com/i/web/status/1050118621198921728", "display_url": "twitter.com/i/web/status/1...", "indices": [ 117, 140 ] } ] }, 4 "user": { "id": 6253282, "id_str": "6253282", "name": "Twitter API", "screen_name": "TwitterAPI", "location": "San Francisco, CA", "description": "The Real Twitter API. Tweets about API changes, service issues and our Developer Platform. 
Don't get an answer? It's on my website.", [...]1 }
```

API 通常以 JSON 或 XML 格式返回数据。JSON 是一种以明文形式表示数据的方式，通常用于在 web 消息中传输数据。当你在测试应用程序时，经常会看到 JSON 消息，因此学会阅读它们非常有用。

JSON 对象以大括号 1 开始和结束。在这些大括号内，表示对象的属性以键值对的形式存储。例如，在前面的数据块中，表示一条推文，`created_at` 属性的值为 `Wed Oct 10 20:19:24 +0000 2018`。这表示该推文是在 2018 年 10 月 10 日星期三晚上 8:19 创建的 2。

JSON 对象还可以包含列表或其他对象。大括号表示对象。前面的推文包含一个 `user` 对象，表示创建该推文的用户 4。列表用方括号表示。Twitter 在前面的 JSON 块中返回了一个空的标签列表，这意味着该推文中没有使用任何标签 3。

你可能会想知道 API 服务器是如何决定谁可以访问数据或执行操作的。API 通常要求用户在访问其服务之前进行身份验证。通常，用户会在 API 请求中包含访问令牌以证明他们的身份。其他时候，用户需要使用特殊的认证头或 Cookies。服务器随后会使用请求中提供的凭证来决定用户应访问哪些资源和操作。

### REST API

有多种类型的 API。此处讨论的 Twitter API 被称为*表述性状态转移（**REST**）* API。REST 是最常用的 API 结构之一。大多数情况下，REST API 返回 JSON 或纯文本格式的数据。REST API 用户向特定的资源端点发送请求以访问该资源。在 Twitter 的案例中，您可以向 *https://api.twitter.com/1.1/statuses/show/* 发送 GET 请求以检索推文信息，向 *https://api.twitter.com/1.1/users/show/* 发送 GET 请求以检索用户信息。

REST API 通常为查询定义了结构，方便用户预测应将请求发送到哪个具体的端点。例如，要通过 Twitter API 删除推文，用户可以向 *https://api.twitter.com/1.1/statuses/destroy/* 发送 POST 请求，要转发推文，用户可以向 *https://api.twitter.com/1.1/statuses/retweet/* 发送 POST 请求。你可以看到，Twitter 的所有 API 端点都采用相同的结构（*https://api.twitter.com/1.1/RESOURCE/ACTION*）：

```
https://api.twitter.com/1.1/users/show
https://api.twitter.com/1.1/statuses/show
https://api.twitter.com/1.1/statuses/destroy
https://api.twitter.com/1.1/statuses/retweet
```

REST API 也可以使用多种 HTTP 方法。例如，GET 通常用于检索资源，POST 用于更新或创建资源，PUT 用于更新资源，DELETE 用于删除资源。

### SOAP API

*SOAP* 是一种在现代应用中较少使用的 API 架构，但许多老旧应用和物联网应用仍然使用 SOAP API。SOAP API 使用 XML 来传输数据，且其消息包含头部和主体。一个简单的 SOAP 请求如下所示：

```
DELETE / HTTPS/1.1
Host: example.s3.amazonaws.com
<DeleteBucket xmlns="http://doc.s3.amazonaws.com/2006-03-01"> <Bucket>quotes</Bucket> <AWSAccessKeyId> AKIAIOSFODNN7EXAMPLE</AWSAccessKeyId> <Timestamp>2006-03-01T12:00:00.183Z</Timestamp> <Signature>Iuyz3d3P0aTou39dzbqaEXAMPLE=</Signature> </DeleteBucket>
```

这个示例请求来自 Amazon S3 的 SOAP API 文档。它删除了一个名为 *quotes* 的 S3 桶。如你所见，API 请求参数作为 XML 文档中的标签传递给服务器。

SOAP 响应如下所示：

```
<DeleteBucketResponse xmlns="http://s3.amazonaws.com/doc/2006-03-01"> <DeleteBucketResponse> <Code>204</Code> <Description>No Content</Description> </DeleteBucketResponse>
</DeleteBucketResponse>
```

该响应表示桶已成功删除，并且不再可用。

SOAP API 有一个名为 *Web 服务描述语言（**WSDL**）* 的服务，用于描述 API 的结构及如何访问它。如果你能找到一个 SOAP API 的 WSDL 文件，你就可以在对其进行破解前了解该 API。你通常可以通过在 API 端点后面添加 *.wsdl* 或 *?wsdl* 来找到 WSDL 文件，或者通过搜索包含 *wsdl* 词汇的 URL 端点。在 WSDL 中，你将能够找到可以测试的 API 端点列表。

### GraphQL API

*GraphQL* 是一种较新的 API 技术，允许开发者请求所需的精确资源字段，并通过一次 API 调用获取多个资源。由于这些优点，GraphQL 正变得越来越普及。

GraphQL API 使用自定义查询语言和单一端点来处理所有 API 功能。这些端点通常位于 /graphql、*/gql* 或 */g*。GraphQL 主要有两种操作类型：查询和变更。*查询* 用于获取数据，就像 REST API 中的 GET 请求一样。*变更* 用于创建、更新和删除数据，就像 REST API 中的 POST、PUT 和 DELETE 请求一样。

举个例子，看看以下 Shopify 的 GraphQL API 请求。Shopify 是一个电子商务平台，允许用户通过 GraphQL API 与他们的在线商店进行互动。要访问 Shopify 的 GraphQL API，开发者需要向端点 *https://SHOPNAME.myshopify.com/admin/api/API_VERSION/graphql.json* 发送 POST 请求，并将 GraphQL 查询放在 POST 请求体中。要检索关于你的商店的信息，你可以发送以下请求：

```
query { shop { name primaryDomain { url host } } }
```

这个 GraphQL 查询表示我们想要检索商店的名称和 `primaryDomain`，并且我们只需要 `primaryDomain` 的 URL 和主机属性。

Shopify 的服务器将以 JSON 格式返回请求的信息：

```
{ "data": { "shop": { "name": "example", "primaryDomain": { "url": "https://example.myshopify.com", "host": "example.myshopify.com" } } }
}
```

注意，响应中并不包含对象的所有字段，而是仅包含用户请求的精确字段。根据需求，你可以请求相同数据对象的更多或更少字段。以下是一个请求更少字段的例子：

```
query { shop { name } }
```

你还可以请求资源属性的精确子字段和其他嵌套属性。例如，在这里，你只请求商店的 `primaryDomain` 的 URL：

```
query { shop { primaryDomain { url } } }
```

这些查询都是用来检索数据的。

变更操作用于编辑数据，可以带有参数并返回值。让我们来看一个来自 *graphql.org* 的变更示例。这个变更操作创建了一个新的客户记录，并需要三个输入参数：`firstName`、`lastName` 和 `email`。然后它会返回新创建客户的 ID：

```
mutation { customerCreate( input: { firstName: "John", lastName: "Tate", email: "john@johns-apparel.com" }) { customer { id } }
}
```

GraphQL 独特的语法可能一开始会让测试变得困难，但一旦你理解了它，你可以像测试其他类型的 API 一样测试这些 API。要了解更多关于 GraphQL 语法的信息，请访问 [`graphql.org/`](https://graphql.org/)。

GraphQL API 还包括一个非常好的侦察工具，供漏洞猎人使用：一种叫做 *introspection* 的功能，允许 API 用户向 GraphQL 系统请求有关其自身的信息。换句话说，它们是返回如何使用 API 的信息的查询。例如，`__schema` 是一个特殊的字段，将返回 API 中所有可用的类型；以下查询将返回系统中的所有类型名称。你可以用它来查找可以查询的数据类型：

```
{ __schema { types { name } }
}
```

你还可以使用 `__type` 查询来查找特定类型的相关字段：

```
{ __type(name: "customer") { name fields { name } }
}
```

你将会得到如下的类型字段返回。你可以使用这些信息来查询 API：

```
{ "data": { "__type": { "name": "customer", "fields": [ { "name": "id", }, { "name": "firstName", }, { "name": "lastName", }, { "name": "email", } ] } }
}
```

内省使得 API 黑客的侦察变得轻松。为了防止恶意攻击者枚举其 API，许多组织在其 GraphQL API 中禁用了内省功能。

### API 驱动的应用程序

越来越多的 API 不再仅仅作为与外部开发者共享数据的机制。你还会遇到*API 驱动的应用程序*，即通过 API 构建的应用程序。API 驱动的应用程序不再从服务器获取完整的 HTML 文档，而是由客户端组件通过 API 调用向服务器请求并呈现数据。

例如，当用户查看 Facebook 帖子时，Facebook 的移动应用使用 API 调用从服务器检索关于这些帖子的数据显示，而不是获取包含嵌入数据的完整 HTML 文档。然后，应用程序在客户端渲染这些数据，形成网页。

许多移动应用程序都是以这种方式构建的。当公司已经有了一个 Web 应用时，使用 API 驱动的方法来构建移动应用可以节省时间。API 允许开发者将应用程序的渲染和数据传输任务分开：开发者可以使用 API 调用来传输数据，然后为移动设备构建一个独立的渲染机制，而不是重新实现相同的功能。

然而，API 驱动的应用程序的兴起意味着公司和应用程序通过 API 暴露出越来越多的数据和功能。API 常常泄露敏感数据和托管应用程序的应用逻辑。如你所见，这使得 API 漏洞成为普遍的安全漏洞来源，并成为漏洞猎人重要的攻击目标。

## 寻找 API 漏洞

让我们探索一些影响 API 的漏洞，以及你可以采取的步骤来发现它们。API 漏洞与影响非 API Web 应用程序的漏洞类似，因此请确保你对我们至今讨论的漏洞有充分的理解。话虽如此，在测试 API 时，你应该把测试重点放在本节中列出的漏洞上，因为它们在 API 实现中非常普遍。

在我们深入探讨之前，有许多开源 API 开发和测试工具可以帮助你提高 API 测试过程的效率。Postman（[`www.postman.com/`](https://www.postman.com/)）是一个非常方便的工具，可以帮助你测试 API。你可以使用 Postman 从头开始创建复杂的 API 请求，并管理你将发送的大量测试请求。GraphQL Playground（[`github.com/graphql/graphql-playground/`](https://github.com/graphql/graphql-playground/)）是一个用来编写 GraphQL 查询的 IDE，具有自动补全和错误高亮功能。

ZAP 有一个 GraphQL 插件（[`www.zaproxy.org/blog/2020-08-28-introducing-the-graphql-add-on-for-zap/`](https://www.zaproxy.org/blog/2020-08-28-introducing-the-graphql-add-on-for-zap/)），可以自动化 GraphQL 自省和测试查询生成。Clairvoyance（[`github.com/nikitastupin/clairvoyance/`](https://github.com/nikitastupin/clairvoyance/)）帮助你在自省被禁用时深入了解 GraphQL API 的结构。

### 执行侦察

首先，寻找 API 漏洞与寻找常规 Web 应用程序的漏洞非常相似，都需要进行侦察。API 测试中最困难的方面是了解应用程序的预期，然后调整有效载荷以操控其功能。

如果你在攻击 GraphQL API，你可以通过发送自省查询来了解 API 的结构。如果你正在测试 SOAP API，可以先寻找 WSDL 文件。如果你攻击的是 REST 或 SOAP API，或者你攻击的 GraphQL API 禁用了自省功能，首先从枚举 API 开始。*API 枚举*是指识别尽可能多的 API 端点，以便你可以测试尽可能多的端点的过程。

要枚举 API，首先阅读 API 的公共文档（如果有的话）。拥有公共 API 的公司通常会发布关于 API 端点及其参数的详细文档。你可以通过在互联网上搜索*公司名称 API* 或 *公司名称 开发者文档* 来找到公共 API 文档。这些文档是枚举 API 端点的一个良好起点，但不要被误导，以为官方文档包含了你可以测试的所有端点！API 通常有公共和私有端点，只有公共端点会出现在这些开发者指南中。

尝试使用 Swagger（[`swagger.io/`](https://swagger.io/)），这是开发人员用于开发 API 的工具包。Swagger 包含一个用于生成和维护 API 文档的工具，开发人员通常用它来内部记录 API。有时候公司不会公开发布 API 文档，但忘记了锁定托管在 Swagger 上的内部文档。在这种情况下，你可以通过搜索互联网中的*公司名称 inurl:swagger*来找到文档。这些文档通常包括所有 API 端点、它们的输入参数以及示例响应。

接下来，你可以浏览所有应用程序的工作流，以捕捉 API 调用。你可以通过使用拦截代理记录后台 HTTP 流量来浏览公司的应用程序。在应用程序的工作流中，你可能会发现一些没有出现在公共文档中的 API 调用。

使用你找到的端点，你可以尝试推测其他端点。例如，REST API 通常具有可预测的结构，因此你可以通过研究现有端点来推测新的端点。如果 */posts/POST_ID/read* 和 /*posts/POST_ID/delete* 都存在，那么是否也有一个名为 */posts/POST_ID/edit* 的端点？类似地，如果你发现博客文章位于 */posts/1234* 和 */posts/1236*，那么 /*posts/1235* 是否也存在？

接下来，使用第五章中的侦查技术，寻找其他 API 端点，例如研究 JavaScript 源代码或公司的公开 GitHub 仓库。你还可以尝试生成错误消息，希望 API 泄露关于它本身的信息。例如，尝试向 API 端点提供意外的数据类型或格式不正确的 JSON 代码。模糊测试技术也可以通过使用单词列表帮助你找到其他 API 端点。许多在线单词列表专门用于模糊测试 API 端点；一个示例单词列表可以在[`gist.github.com/yassineaboukir/8e12adefbd505ef704674ad6ad48743d/`](https://gist.github.com/yassineaboukir/8e12adefbd505ef704674ad6ad48743d/)找到。我们将在第二十五章中进一步讨论如何模糊测试端点。

还要注意，API 经常会更新。虽然应用程序可能不再积极使用旧版本的 API，但这些版本可能仍会向服务器发出响应。对于你在 API 后续版本中发现的每个端点，你应该测试旧版本的端点是否可用。例如，如果 */api/****v2****/user_emails/52603991338963203244* 端点存在，那么 */api/****v1****/user_emails/52603991338963203244* 是否也存在？ API 的旧版本通常包含已在新版本中修复的漏洞，因此请确保将查找旧版本的 API 端点纳入你的侦查策略中。

最后，花时间理解每个 API 端点的功能、参数和查询结构。你对 API 工作原理的了解越深入，就越能理解如何攻击它。识别所有可能的用户数据输入位置，以便未来测试。注意任何认证机制，包括以下内容：

+   需要哪些访问令牌？

+   哪些端点需要令牌，哪些不需要？

+   访问令牌是如何生成的？

+   用户可以在不登录的情况下使用 API 生成有效的令牌吗？

+   更新或重置密码时，访问令牌会过期吗？

在整个侦查过程中，一定要记下大量笔记。记录你找到的端点及其参数。

### 测试访问控制漏洞和信息泄露

在侦查后，我喜欢从测试访问控制问题和信息泄露开始。大多数 API 使用访问令牌来确定客户端的权限；它们为每个 API 客户端颁发访问令牌，客户端使用这些令牌来执行操作或检索数据。如果这些 API 令牌没有正确发放和验证，攻击者可能绕过认证，非法访问数据。

例如，有时 API 令牌在服务器接收后没有进行验证。其他时候，API 令牌没有随机生成，可以被预测。最后，一些 API 令牌没有定期失效，因此攻击者在窃取令牌后可以无限期地访问系统。

另一个问题是资源或功能级别的访问控制问题。有时，API 端点并没有与主应用程序相同的访问控制机制。例如，假设一个拥有有效 API 密钥的用户可以检索关于自己的数据。他们是否也能读取其他用户的数据？或者他们能否通过 API 代表他人执行操作？最后，没有管理员权限的普通用户是否可以读取仅限管理员访问的端点数据？与 REST 或 SOAP API 不同，应用程序的 GraphQL API 可能有自己独立的授权机制和配置。这意味着，即使应用程序的 Web 或 REST API 是安全的，你也可以在 GraphQL 端点上测试访问控制问题。这些问题类似于第十章中讨论的 IDOR 漏洞。

另外，有时一个 API 提供多种方式来执行相同的操作，而访问控制并未在所有方式中实现。例如，假设一个 REST API 有两种方式来删除博客文章：发送 POST 请求到*/posts/POST_ID/delete*，或者发送 DELETE 请求到*/posts/POST_ID*。你应该问自己：这两个端点是否受到相同的访问控制？

另一个常见的 API 漏洞是信息泄露。API 端点常常返回比必要的更多的信息，或者返回不需要的内容来渲染网页。例如，我曾经发现一个 API 端点，用于填充用户的个人资料页面。当我访问其他人的个人资料页面时，API 调用返回了该资料主人的信息。乍一看，个人资料页面似乎没有泄露任何敏感信息，但实际上，获取用户数据的 API 响应却返回了该资料主人的私人 API 令牌！当攻击者通过访问受害者的个人资料页面窃取其 API 令牌后，他们可以使用该访问令牌冒充受害者。

制作一份应该由某种形式的访问控制限制的端点列表。对于每个端点，创建两个不同权限级别的用户帐户：一个应该能够访问该功能，另一个不应该。测试是否能够使用低权限帐户访问受限功能。

如果权限较低的用户无法访问受限功能，可以尝试移除访问令牌，或者添加额外的参数，如在 API 调用中加入 cookie `admin=1`。你还可以切换 HTTP 请求方法，包括 GET、POST、PUT、PATCH 和 DELETE，看看是否在所有方法中都正确实施了访问控制。例如，如果你不能通过 POST 请求到 API 端点来编辑其他用户的博客文章，那么你是否能通过使用 PUT 请求绕过保护？

尝试通过更换用户 ID 或 API 调用中找到的其他用户识别参数，查看、修改和删除其他用户的信息。如果用于标识用户和资源的 ID 不可预测，尝试通过其他端点的信息泄露来泄露 ID。例如，我曾经找到一个 API 端点，它返回了用户信息，暴露了用户的 ID 以及所有用户朋友的 ID。通过获取用户和朋友的 ID，我能够访问两者之间发送的消息。通过结合两个信息泄露，仅使用用户 ID，我成功读取了用户的私人消息！

在 GraphQL 中，一个常见的错误配置是允许权限较低的用户通过变更请求修改他们不应该修改的数据。尝试捕捉一个用户账户允许的 GraphQL 查询，并查看你是否可以通过另一个不应拥有权限的用户发送相同的查询并获得相同的结果。

在寻找访问控制问题时，仔细研究服务器返回的数据。不要仅仅查看结果 HTML 页面；深入分析原始 API 响应，因为 API 通常返回未显示在网页上的数据。你可能会在响应体中发现敏感信息泄露。API 端点是否返回了任何私人用户信息，或者关于组织的敏感信息？返回的信息是否应该对当前用户可用？返回的信息是否对公司构成安全风险？

### 测试速率限制问题

API 通常缺乏速率限制；换句话说，API 服务器没有限制客户端或用户账户在短时间内可以发送的请求数量。缺乏速率限制本身是一个低严重性的漏洞，除非它被证明能够被攻击者利用。但是，在关键的端点上，缺乏速率限制意味着恶意用户可以向服务器发送大量请求，从而窃取数据库信息或暴力破解凭证。

在没有速率限制的情况下，可能会很危险的端点包括认证端点、没有访问控制保护的端点以及返回大量敏感数据的端点。例如，我曾经遇到过一个 API 端点，允许用户通过电子邮件 ID 检索他们的电子邮件，如下所示：

```
GET /api/v2/user_emails/**52603991338963203244**
```

这个端点没有受到任何访问控制的保护。由于这个端点没有速率限制，攻击者实际上可以通过发送大量请求来猜测电子邮件 ID 字段。一旦猜到有效的 ID，他们就能访问另一个用户的私人电子邮件。

测试速率限制问题时，可以向端点发送大量请求。你可以使用 Burp Intruder 或`curl`在短时间内发送 100 到 200 个请求。确保在不同的认证阶段重复测试，因为不同权限级别的用户可能会受到不同的速率限制。

在测试速率限制问题时要格外小心，因为很容易因为请求过多而意外地对应用程序发起 DoS 攻击。在进行速率限制测试之前，应该获得书面许可，并根据公司的政策按时限来控制你的请求。

还要记住，应用程序可能设置的速率限制高于你的测试工具的能力。例如，应用程序可能设置每秒 400 个请求的速率限制，而你的工具可能无法达到这一限制。

### 技术漏洞测试

书中我们讨论过的许多漏洞——如 SQL 注入、反序列化问题、XXE、模板注入、SSRF 和 RCE——都是由于输入验证不当造成的。有时，开发人员会忘记为 API 实现适当的输入验证机制。

因此，API 也容易受到许多影响常规 Web 应用程序的其他漏洞的攻击。由于 API 是应用程序接受用户输入的另一种方式，它们成为攻击者将恶意输入传递到应用程序工作流中的另一种途径。

如果 API 端点可以访问外部 URL，则可能存在 SSRF 漏洞，因此你应该检查其对内部 URL 的访问是否受到限制。API 中也可能发生竞态条件。如果你能够利用 API 端点访问受竞态条件影响的应用程序功能，这些端点可能成为触发竞态条件的另一种方式。

其他漏洞，如路径遍历、文件包含、不安全的反序列化问题、XXE 和 XSS 也可能发生。如果 API 端点通过文件路径返回内部资源，攻击者可能利用该端点读取存储在服务器上的敏感文件。如果用于文件上传的 API 端点没有限制用户可以上传的数据类型，攻击者可能会上传恶意文件，如 Web Shell 或其他恶意软件到服务器。API 通常还会接受以 XML 等序列化格式提供的用户输入。在这种情况下，不安全的反序列化或 XXE 问题可能会发生。通过文件上传或 XXE 的 RCE 通常在 API 端点中见到。最后，如果 API 的 URL 参数在响应中被反射，攻击者可以利用该 API 端点触发受害者浏览器上的反射型 XSS 攻击。

测试这些问题的过程将类似于常规 Web 应用程序的测试。你只需要以 API 的形式将有效载荷提供给应用程序。

例如，对于路径遍历和文件包含攻击等漏洞，注意 API 端点中的绝对路径和相对路径，并尝试修改路径参数。如果 API 端点接受 XML 输入，尝试将 XXE 有效载荷插入请求中。如果该端点的 URL 参数在响应中被反射，查看是否可以通过在 URL 中放置有效载荷来触发反射型 XSS。

你还可以利用模糊测试技术（我们将在第二十五章讨论），来发现这些漏洞。

应用程序越来越依赖于 API，即使 API 的保护措施往往不如其 Web 应用程序的同类那样完善。注意观察你目标使用的 API，你可能会发现一些在主应用程序中不存在的问题。如果你有兴趣了解更多关于 API 和 Web 应用程序的黑客攻击方法，OWASP Web 安全测试指南（[`github.com/OWASP/wstg/`](https://github.com/OWASP/wstg/)）是一个很好的学习资源。
