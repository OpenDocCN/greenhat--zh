# 第三章：Web API 的解剖

![](img/chapterart.png)

大多数普通用户对 Web 应用程序的了解来自于他们在 Web 浏览器的图形用户界面（GUI）中能看到和点击的内容。在幕后，API 执行大部分工作。特别是，Web API 提供了一种应用程序可以通过 HTTP 使用其他应用程序的功能和数据，以便将图像、文本和视频传递到 Web 应用程序 GUI 的方式。

本章涵盖常见的 API 术语、类型、数据交换格式和认证方法，并通过示例将这些信息联系在一起：观察与 Twitter API 交互期间交换的请求和响应。

## Web API 的工作原理

与 Web 应用程序类似，Web API 依赖于 HTTP，在 API 的主机（*提供者*）与发出 API 请求的系统或个人（*消费者*）之间建立客户端/服务器关系。

API 消费者可以从*API 端点*请求资源，这是与 API 的某一部分进行交互的 URL。以下每个示例都是不同的 API 端点：

1.  *https://example.com/api/v3/users/*

1.  *https://example.com/api/v3/customers/*

1.  *https://example.com/api/updated_on/*

1.  *https://example.com/api/state/1/*

*资源*是被请求的数据。*单例*资源是一个唯一的对象，例如*/api/user/{user_id}*。*集合*是一组资源，例如*/api/profiles/users*。*子集合*指的是特定资源内的集合。例如，*/api/user/{user_id}/settings* 是访问特定（单例）用户的*设置*子集合的端点。

当消费者从提供者请求资源时，请求会经过一个*API 网关*，这是一个作为 Web 应用入口点的 API 管理组件。例如，如图 Figure 2-1 所示，最终用户可以使用众多设备访问应用程序的服务，所有这些设备都经过 API 网关过滤。然后，API 网关将请求分发到需要满足每个请求的微服务。

API 网关会过滤错误请求，监控传入流量，并将每个请求路由到适当的服务或微服务。API 网关还可以处理诸如身份验证、授权、传输中的 SSL 加密、速率限制和负载平衡等安全控制。

![显示 API 网关在中心，左侧是应用程序，右侧是微服务的图示。箭头从移动应用程序、Web 应用程序和 IoT 应用程序指向 API 网关，从 API 网关指向微服务 1、微服务 2 和微服务 3。](img/F02001.png)

Figure 2-1: 一个示例微服务架构和 API 网关

*微服务*是一个 Web 应用程序的模块化部分，处理特定的功能。微服务使用 API 来传输数据并触发操作。例如，一个带有支付网关的 Web 应用程序可能在一个单独的网页上具有几个不同的功能：账单功能、记录客户账户信息的功能以及在购买后发送电子邮件收据的功能。该应用程序的后端设计可以是单体架构，意味着所有服务都存在于一个单一的应用程序中，或者它可以有一个微服务架构，其中每个服务作为独立的应用程序运行。

API 使用者无法看到后端设计，他们只能看到可以交互的端点和可以访问的资源。这些内容在 API 的*契约*中列出，契约是人类可读的文档，描述了如何使用 API 以及你可以期望它的行为。API 文档在不同的组织之间有所不同，但通常包括认证要求、用户权限级别、API 端点和所需的请求参数的描述。它可能还包括使用示例。从 API 黑客的角度来看，文档可以揭示调用客户数据的端点，成为管理员所需的 API 密钥，甚至业务逻辑的缺陷。

在以下框中，来自[`docs.github.com/en/rest/reference/apps`](https://docs.github.com/en/rest/reference/apps)的 GitHub API 文档，针对*/applications/{client_id}/grants/{access_token}*端点，是优质文档的示例。

该端点的文档包括 API 请求的目的描述、与 API 端点交互时使用的 HTTP 请求方法，以及端点本身，*/applications*，后面跟随变量。

缩写*CRUD*，即*创建、读取、更新、删除*，描述了与 API 交互时使用的主要操作和方法。*创建*是通过 POST 请求来创建新记录的过程。*读取*是数据检索，通过 GET 请求完成。*更新*是修改当前已存在记录而不覆盖的过程，通过 POST 或 PUT 请求完成。*删除*是删除记录的过程，可以通过 POST 或 DELETE 请求实现，如示例所示。请注意，CRUD 仅仅是最佳实践，开发人员可能会以其他方式实现他们的 API。因此，当你以后学习如何破解 API 时，我们将测试超越 CRUD 方法的内容。

按惯例，大括号表示在路径参数中需要一个特定的变量。*{client_id}* 变量必须替换为实际的客户端 ID，*{access_token}* 变量必须替换为你自己的访问令牌。令牌是 API 提供者用来识别和授权对已批准的 API 消费者的请求的工具。其他 API 文档可能使用冒号或方括号来表示变量（例如，*/api/v2/:customers/* 或 */api/:collection/:client_id*）。

“参数”部分列出了执行所述操作所需的身份验证和授权要求，包括每个参数值的名称、提供的数据类型、数据的包含位置以及对参数值的描述。

## 标准 Web API 类型

API 有标准类型，每种类型在规则、功能和目的上有所不同。通常，给定的 API 将只使用一种类型，但你可能会遇到与其他 API 格式和结构不匹配的端点，或者根本不匹配任何标准类型。能够识别典型和非典型 API 将帮助你了解作为 API 黑客需要期待什么并进行测试。记住，大多数公共 API 都是自助服务设计的，因此 API 提供者通常会告知你将与哪种类型的 API 进行交互。

本节描述了本书将重点讨论的两种主要 API 类型：RESTful API 和 GraphQL。书中的后续部分以及实验部分将专门讨论对 RESTful API 和 GraphQL 的攻击。

### RESTful API

*表现层状态转移（REST）* 是一组用于使用 HTTP 方法进行通信的应用程序架构约束。使用 REST 约束的 API 被称为 *RESTful*（或简称 REST）API。

REST 旨在改进其他较老的 API（如简单对象访问协议 SOAP）的许多低效之处。例如，它完全依赖于使用 HTTP，这使得它对最终用户更为友好。REST API 主要使用 HTTP 方法 GET、POST、PUT 和 DELETE 来实现 CRUD（如“Web API 工作原理”一节所述）。

RESTful 设计依赖于六个约束条件。这些约束是“应该”而不是“必须”，反映了 REST 本质上是一套基于 HTTP 资源架构的指南：

1.  **统一接口：**REST API 应该具有统一的接口。换句话说，客户端设备不应成为问题；无论是移动设备、物联网（IoT）设备还是笔记本电脑，都必须能够以相同的方式访问服务器。

1.  **客户端/服务器：**REST API 应该具有客户端/服务器架构。客户端是请求信息的消费者，服务器是提供信息的提供者。

1.  **无状态：** REST API 不应要求有状态的通信。REST API 在通信过程中不维护状态；每次请求就像是服务器接收到的第一个请求。因此，消费者需要提供服务器执行请求所需的所有信息。这种方式的好处是，服务器无需记住每个请求之间的消费者信息。消费者通常会提供令牌来创建类似状态的体验。

1.  **可缓存：** REST API 提供者的响应应该指示该响应是否可以缓存。*缓存*是一种通过将常请求的数据存储在客户端或服务器缓存中来提高请求吞吐量的方法。当请求被发出时，客户端首先会检查本地存储是否有请求的信息。如果找不到，它会将请求传递给服务器，服务器再检查自己的本地存储。如果数据仍然找不到，请求可能会被传递给其他服务器，如数据库服务器，去检索数据。

    正如你所想象的，如果数据存储在客户端，客户端可以立即在几乎没有服务器处理成本的情况下检索请求的数据。如果服务器已经缓存了某个请求的结果，这同样适用。请求获取数据的链条越长，资源消耗越高，所需时间也越长。通过默认使 REST API 可缓存，是提高整体 REST 性能和可扩展性的一种方式，能减少响应时间和服务器处理能力的消耗。API 通常使用头部信息来管理缓存，解释何时请求的信息会从缓存中过期。

1.  **分层系统：** 客户端应该能够从端点请求数据，而无需了解底层服务器架构。

1.  **按需代码（可选）：** 允许将代码发送到客户端执行。

REST 是一种风格，而不是协议，因此每个 RESTful API 可能会有所不同。它可能启用了 CRUD 之外的方法，拥有自己的一套认证要求，使用子域名而非路径来定义端点，具有不同的速率限制要求，等等。此外，开发者或组织可能会将他们的 API 称为“RESTful”，但并未遵循标准，这意味着你不能期望每个遇到的 API 都符合所有的 REST 限制。

列表 2-1 显示了一个典型的 REST API GET 请求，用于查找商店库存中有多少个枕头。列表 2-2 显示了提供者的响应。

```
GET /api/v3/inventory/item/pillow HTTP/1.1
HOST: rest-shop.com
User-Agent: Mozilla/5.0
Accept: application/json
```

列表 2-1：一个示例的 RESTful API 请求

```
HTTP/1.1 200 OK
Server: RESTfulServer/0.1
Cache-Control: no-store
Content-Type: application/json

{
"item": {
    "id": "00101",
    "name": "pillow",
    "count": 25
    "price": {
"currency": "USD",
"value": "19.99"
}
  },
}
```

列表 2-2：一个示例的 RESTful API 响应

这个 REST API 请求仅是一个 HTTP GET 请求，指向指定的 URL。在这个例子中，请求查询商店的枕头库存。提供者以 JSON 格式响应，显示该商品的 ID、名称和库存数量。如果请求出错，提供者会以 400 范围的 HTTP 错误代码回应，指示问题所在。

有一点需要注意：*rest-shop.com* 商店在其响应中提供了有关资源“枕头”的所有信息。如果消费者的应用程序只需要枕头的名称和值，消费者则需要过滤掉额外的信息。返回给消费者的信息量完全取决于 API 提供者如何编程其 API。

REST APIs 有一些常见的头，你应该熟悉它们。这些头与 HTTP 头是一样的，但在 REST API 请求中比在其他 API 类型中更常见，因此它们可以帮助你识别 REST APIs。（头部、命名约定和数据交换格式通常是 API 类型的最佳指示器。）以下小节详细介绍了一些你会遇到的常见 REST API 头。

#### 授权

`Authorization` 头用于将令牌或凭证传递给 API 提供者。这些头的格式是 `Authorization: <type> <token/credentials>`。例如，看看下面的授权头：

```
Authorization: Bearer Ab4dtok3n
```

有不同的授权类型*.* `Basic` 使用 base64 编码的凭证，`Bearer` 使用 API 令牌，最后，`AWS-HMAC-SHA256` 是一种 AWS 授权类型，使用访问密钥和密钥。

#### 内容类型

`Content-Type` 头用于指示正在传输的媒体类型。这些头与 `Accept` 头不同，后者表示你希望接收的媒体类型；`Content-Type` 头描述你正在发送的媒体类型。

以下是一些常见的 `Content-Type` 头用于 REST APIs：

1.  `application/json` 用于指定 JavaScript 对象表示法（JSON）作为媒体类型。JSON 是 REST APIs 中最常见的媒体类型。

1.  `application/xml` 用于指定 XML 作为媒体类型。

1.  `application/x-www-form-``urlencoded` 一种格式，其中发送的值被编码并用一个和号 (`&`) 分隔，键值对之间使用等号 (`=`)。

#### 中间件 (X) 头

`X-<anything>` 头部被称为 *中间件头部*，可以用于各种目的。它们在 API 请求之外也相当常见。`X-Response-Time` 可作为 API 响应，用于指示响应处理所花费的时间。`X-API-Key` 可作为 API 密钥的授权头部。`X-Powered-By` 可提供有关后端服务的附加信息。`X-Rate-Limit` 可告知消费者在给定时间内可以发出多少请求。`X-RateLimit-Remaining` 可告知消费者在违反速率限制之前还能发出多少个请求。（还有许多其他的，但你明白这个意思。）`X-<anything>` 中间件头部可以为 API 消费者和黑客提供大量有用信息。

### GraphQL

*Graph Query Language*（图查询语言）的简称，*GraphQL* 是一种 API 规范，允许客户端定义它们希望从服务器请求的数据结构。GraphQL 是 RESTful 的，因为它遵循 REST API 的六个约束。然而，GraphQL 也采用了 *查询中心* 的方法，因为它的结构类似于像结构化查询语言（SQL）这样的数据库查询语言。

正如你从规范的名称中可能得出的结论，GraphQL 将资源存储在图数据结构中。要访问 GraphQL API，通常需要访问其托管的 URL，并提交一个包含查询参数的授权请求作为 POST 请求的主体，类似于以下内容：

```
query {
  users {
    username
    id
    email
  }
}
```

在正确的上下文中，这个查询将为你提供所请求资源的用户名、ID 和电子邮件。对这个查询的 GraphQL 响应将如下所示：

```
{
  "data": {
    "users": {
      "username": "hapi_hacker",
      "id": 1111,
      "email": "hapihacker@email.com"
    }
  }
}
```

GraphQL 在多个方面改进了典型的 REST API。由于 REST API 是基于资源的，消费者可能需要发出多个请求才能获取他们需要的所有数据。另一方面，如果消费者只需要来自 API 提供者的特定值，消费者则需要筛选掉多余的数据。而 GraphQL 允许消费者通过单个请求获得他们想要的确切数据。这是因为与 REST API 不同，在 REST API 中，客户端会接收到服务器程序设置的返回数据，其中包括他们不需要的数据，而 GraphQL API 允许客户端从资源中请求特定字段。

GraphQL 也使用 HTTP，但通常依赖于一个使用 POST 方法的单一入口点（URL）。在一个 GraphQL 请求中，POST 请求的主体是服务提供者处理的内容。例如，查看 列表 2-3 中的 GraphQL 请求和 列表 2-4 中的响应，展示了一个检查商店显卡库存的请求。

```
POST /graphql HTTP/1.1
HOST: graphql-shop.com
Authorization: Bearer ab4dt0k3n

{query❶ {
  inventory❷ (item:"Graphics Card", id: 00101) {
name
fields❸{
price
quantity} } }
}
```

列表 2-3：一个 GraphQL 请求示例

```
HTTP/1.1 200 OK
Content-Type: application/json
Server: GraphqlServer

{
"data": { 
"inventory": { "name": "Graphics Card",
"fields":❹[
{
"price":"999.99"
"quantity": 25 } ] } }
}
```

列表 2-4：一个 GraphQL 响应示例

如你所见，请求负载在主体中指定了所需的信息。GraphQL 请求体以查询操作 ❶ 开头，等同于 GET 请求，用于从 API 获取信息。我们正在查询的 GraphQL 节点 `"inventory"` ❷ 也被称为根查询类型。节点类似于对象，由字段 ❸ 组成，类似于 REST 中的键值对。这里的主要区别在于我们可以指定具体的字段。在这个例子中，我们查询的是“价格”和“数量”字段。最后，你可以看到 GraphQL 响应只提供了所请求的显卡的字段 ❹。没有获取商品 ID、商品名称和其他冗余信息，查询只返回了所需的字段。

如果这是一个 REST API，可能需要向不同的端点发送请求，先获取显卡的数量，再获取品牌，但使用 GraphQL，你可以从一个端点构建查询，获取所需的具体信息。

GraphQL 仍然使用 CRUD 操作，刚开始可能会让人困惑，因为它依赖于 POST 请求。然而，GraphQL 在 POST 请求中使用三种操作与 GraphQL API 交互：查询（query）、变更（mutation）和订阅（subscription）。*查询*是用于检索数据（读取）。*变更*是用于提交和写入数据（创建、更新和删除）。*订阅*是用于在事件发生时发送数据（读取）。订阅是 GraphQL 客户端监听服务器实时更新的一种方式。

GraphQL 使用 *架构*，即可以通过给定服务查询的数据集合。访问 GraphQL 架构类似于访问 REST API 集合。GraphQL 架构将为你提供查询 API 所需的信息。

如果有 GraphQL IDE（如 GraphiQL），你可以通过浏览器与 GraphQL 进行交互（见 图 2-2）。

否则，你需要使用 GraphQL 客户端，如 Postman、Apollo-Client、GraphQL-Request、GraphQL-CLI 或 GraphQL-Compose。在后面的章节中，我们将使用 Postman 作为 GraphQL 客户端。

![接口截图，左侧窗口可以输入 GraphQL 查询，右侧窗口显示与 GraphQL 架构相关的信息](img/F02002.png)

图 2-2：GitHub 的 GraphiQL 接口

## REST API 规范

REST API 的多样性为其他工具和标准化留下了填补一些空白的空间。*API 规范*或描述语言是帮助组织设计其 API 的框架，自动创建一致的人类可读文档，并帮助开发者和用户了解 API 功能和结果预期的工具。没有规范，API 之间的一致性将很少甚至没有。消费者必须了解每个 API 的文档格式，并调整其应用程序以与每个 API 交互。

相反，消费者可以编写其应用程序以接受不同的规范，然后轻松地与使用给定规范的任何 API 交互。换句话说，您可以将规范视为 API 的家庭电源插座。与为每个家用电器设计唯一的电源插座不同，贯穿整个家庭使用统一的格式使您能够在任何墙壁上的插座上购买烤面包机并插入，而无需任何麻烦。

*OpenAPI Specification 3.0 (OAS)*，之前称为 Swagger，是 RESTful API 的主要规范之一。OAS 通过允许开发者描述端点、资源、操作以及认证和授权需求来帮助组织和管理 API。开发者可以创建人类和机器可读的 API 文档，格式可以是 JSON 或 YAML。一致的 API 文档对开发者和用户都是有利的。

*RESTful API 建模语言 (RAML)* 是另一种一致生成 API 文档的方法。RAML 是一种专门用于文档格式化的开放规范，完全采用 YAML。与 OAS 类似，RAML 旨在文档化、设计、构建和测试 REST API。有关 RAML 的更多信息，请访问 raml-spec GitHub 仓库（[`github.com/raml-org/raml-spec`](https://github.com/raml-org/raml-spec)）。

在后续章节中，我们将使用一个名为 Postman 的 API 客户端来导入规范，并即时访问组织 API 的功能。

## API 数据交换格式

API 使用多种格式促进数据交换。此外，规范使用这些格式来记录 API。某些 API（如 SOAP）要求特定格式，而其他 API 允许客户端在请求和响应体中指定使用的格式。本节介绍三种常见的格式：JSON、XML 和 YAML。熟悉数据交换格式将帮助您识别 API 类型、API 的功能及其如何处理数据。

### JSON

*JavaScript Object Notation (JSON)* 是本书中主要使用的数据交换格式，因为它广泛用于 API。JSON 以一种既可读又易于应用程序解析的方式组织数据；许多编程语言可以将 JSON 转换为它们可以使用的数据类型。

JSON 将对象表示为由逗号分隔的键/值对，位于一对大括号内，如下所示：

```
{
  "firstName": "James",
  "lastName": "Lovell",
  "tripsToTheMoon": 2,
  "isAstronaut": true,
  "walkedOnMoon": false,
  "comment" : "This is a comment",
  "spacecrafts": ["Gemini 7", "Gemini 12", "Apollo 8", "Apollo 13"],
  "book": [
    {
      "title": "Lost Moon",
      "genre": "Non-fiction"
    }
  ]
}
```

从第一个花括号到最后一个花括号之间的所有内容都被认为是一个对象。在该对象中，有几个键值对，例如 `"firstName": "James"`，`"lastName": "Lovell"`，和 `"tripsToTheMoon": 2`。键值对的第一个条目（左边）是 *键*，一个描述值对的字符串，第二个条目是 *值*（右边），它是某种类型的数据，表示为可接受的数据类型之一（字符串、数字、布尔值、null、数组或另一个对象）。例如，注意到 `"walkedOnMoon"` 的布尔值为 `false`，或 `"spacecrafts"` 数组被方括号包围。最后，嵌套的对象 `"book"` 包含它自己的键值对集合。表 2-1 更详细地描述了 JSON 类型。

JSON 不允许内联注释，因此任何类似注释的内容必须作为键值对存在，例如 `"comment" : "这是一个注释"`。或者，你可以在 API 文档或 HTTP 响应中找到注释。

表 2-1：JSON 类型

| **类型** | **描述** | **示例** |
| --- | --- | --- |

| 字符串 | 双引号内的任意字符组合。 | `{` `"Motto":"Hack the planet",`

`"Drink":"Jolt",`

`"User”:"Razor"`

`}` |

| 数字 | 基本整数、分数、负数和指数。请注意，多个项目之间用逗号分隔。 | `{` `"number_1" : 101,`

`"number_2" : -102,`

`"number_3" : 1.03,`

`"number_4" : 1.0E+4`

`}` |

| 布尔值 | `true` 或 `false`。 | `{` `"admin" : false,`

`"privesc" : true`

`}` |

| Null | 没有值。 | `{` `"value" : null`

`}` |

| 数组 | 有序的值集合。值集合由括号（`[]`）括起来，且值之间用逗号分隔。 | `{` `"uid" : ["1","2","3"]`

`}` |

| 对象 | 无序的值对集合，插入在花括号（`{}`）之间。一个对象可以包含多个键值对。 | `{` `"admin" : false,`

`"key" : "value",`

`"privesc" : true,`

`"uid" : 101,`

`"vulnerabilities" : "众多漏洞"`

`}` |

为了说明这些类型，看看以下在 Twitter API 响应中找到的 JSON 数据中的键值对：

```
{
"id":1278533978970976256, ❶
"id_str":"1278533978970976256", ❷
"full_text":"1984: William Gibson published his debut novel, Neuromancer. It's a cyberpunk tale about Henry Case, a washed up computer hacker who's offered a chance at redemption by a mysterious dude named Armitage. Cyberspace. Hacking. Virtual reality. The matrix. Hacktivism. A must read. https:\/\/t.co\/R9hm2LOKQi",
"truncated":false ❸
}
```

在这个例子中，你应该能够识别数字 `1278533978970976256` ❶，像 `"id_str"` 和 `"full_text"` 这样的字符串 ❷，以及 `"truncated"` 的布尔值 ❸。

### XML

*扩展标记语言 (XML)* 格式已经存在一段时间了，你可能会认识它。XML 的特点是使用描述性标签来包装数据。尽管 REST API 可以使用 XML，但它通常与 SOAP API 相关联。SOAP API 只能使用 XML 作为数据交换格式。

你刚刚看到的 Twitter JSON，如果转换成 XML 会是如下所示：

```
<?xml version="1.0" encoding="UTF-8" ?> ❶
<root> ❷
    <id>1278533978970976300</id>
  <id_str>1278533978970976256</id_str>
  <full_text>1984: William Gibson published his debut novel, Neuromancer. It&#x27;s a cyberpunk tale about Henry Case, a washed up computer hacker who&#x27;s offered a chance at redemption by a mysterious dude named Armitage. Cyberspace. Hacking. Virtual reality. The matrix. Hacktivism. A must read. https://t.co/R9hm2LOKQi </full_text>
  <truncated>false</truncated>
</root>
```

XML 总是以 *前导部分* 开始，包含有关所使用的 XML 版本和编码的信息 ❶。

接下来，*元素* 是 XML 中最基本的部分。元素是任何 XML 标签或被标签包围的信息。在之前的示例中，`<id>1278533978970976300</id>`，`<id_str>1278533978</id_str>`，`<full_text>`，`</full_text>`，和 `<truncated>false</truncated>` 都是元素。XML 必须有一个根元素，并且可以包含子元素。在这个示例中，根元素是 `<root>` ❷。子元素是 XML 属性。以下示例中，`<BookGenre>` 元素就是一个子元素的例子：

```
`<LibraryBooks>`
 `<BookGenre>SciFi</BookGenre>`
`</LibraryBooks>`
```

XML 中的注释由两个破折号包围，如下所示：`<!--XML 注释示例-->`。

XML 和 JSON 之间的主要区别是 JSON 的描述性标签、字符编码和长度：XML 需要更长时间来传达相同的信息，足足需要 565 字节。

### YAML

另一种轻量级的数据交换形式用于 API，*YAML* 是一个递归缩写，代表 *YAML Ain’t Markup Language*。它被创建为一种更易于人类和计算机阅读的数据交换格式。

与 JSON 类似，YAML 文档包含键/值对。值可以是任何 YAML 数据类型，包括数字、字符串、布尔值、空值和序列。例如，查看以下 YAML 数据：

```
---
id: 1278533978970976300
id_str: 1278533978970976256
#Comment about Neuromancer
full_text: "1984: William Gibson published his debut novel, Neuromancer. It's a cyberpunk tale about Henry Case, a washed up computer hacker who's offered a chance at redemption by a mysterious dude named Armitage. Cyberspace. Hacking. Virtual reality. The matrix. Hacktivism. A must read. https://t.co/R9hm2LOKQi"
truncated: false
...
```

你会注意到 YAML 比 JSON 更易读。YAML 文档以

```
`---`
```

并以此结束

```
`...`
```

而不是使用花括号。此外，字符串周围的引号是可选的。此外，URLs 不需要使用反斜杠进行编码。最后，YAML 使用缩进代替花括号表示嵌套，并允许以 `#` 开头的注释。

API 规范通常以 JSON 或 YAML 格式呈现，因为这些格式易于人类理解。只要掌握几个基本概念，我们就可以查看这两种格式并理解其内容；同样，机器也能轻松解析这些信息。

如果你想查看更多 YAML 的应用示例，请访问 [`yaml.org`](https://yaml.org)。整个网站以 YAML 格式呈现，YAML 是递归的，一直到最深处。

## API 身份验证

API 可能允许消费者在没有身份验证的情况下公开访问，但当 API 允许访问专有或敏感数据时，它将使用某种形式的身份验证和授权。API 的身份验证过程应该验证用户是否为其声称的身份，而授权过程应该授予他们访问所允许数据的权限。本节涵盖了多种 API 身份验证和授权方法。这些方法在复杂性和安全性上有所不同，但它们都有一个共同的原则：消费者在发起请求时必须向提供方发送某种信息，提供方必须将该信息与用户关联，然后才能授予或拒绝对资源的访问。

在深入了解 API 认证之前，了解认证的含义非常重要。认证是验证身份的过程。在 Web 应用中，认证是你向 Web 服务器证明你是该 Web 应用的有效用户的方式。通常，这是通过使用凭据完成的，凭据包括唯一的 ID（如用户名或电子邮件）和密码。客户端在发送凭据后，Web 服务器会将其与存储的凭据进行比对。如果提供的凭据与存储的凭据匹配，Web 服务器会创建一个用户会话并向客户端发放 cookie。

当 Web 应用与用户之间的会话结束时，Web 服务器会销毁会话并移除相关的客户端 cookie。

如本章早些时候所述，REST 和 GraphQL API 是无状态的，因此当消费者对这些 API 进行认证时，客户端和服务器之间不会创建会话。相反，API 消费者必须在每个发送到 API 提供者 Web 服务器的请求中证明自己的身份。

### 基本认证

最简单的 API 认证形式是 *HTTP 基本认证*，消费者将用户名和密码包含在请求的头部或正文中。API 可以将用户名和密码以明文形式传递给提供者，如 `username:password`，或者它可以使用类似 base64 的方式编码凭据来节省空间（例如，编码为 `dXNlcm5hbWU6cGFzc3dvcmQK`）。

编码并不是加密，如果 base64 编码的数据被捕获，它可以很容易地被解码。例如，你可以使用 Linux 命令行对 `username:password` 进行 base64 编码，然后解码编码结果：

```
$ **echo "username:password"|base64**
dXNlcm5hbWU6cGFzc3dvcmQK
$ **echo "dXNlcm5hbWU6cGFzc3dvcmQK"|base64 -d**
username:password
```

如你所见，基本认证本身没有内在的安全性，完全依赖于其他安全控制措施。攻击者可以通过捕获 HTTP 流量、执行中间人攻击、通过社交工程手段诱使用户提供凭据，或者进行暴力破解攻击，尝试不同的用户名和密码直到找到有效的凭据，从而突破基本认证。

由于 API 通常是无状态的，只有基本认证的 API 需要消费者在每次请求中提供凭据。通常，API 提供者会在首次请求时使用基本认证，并随后发放 API 密钥或其他令牌用于所有其他请求。

### API 密钥

*API 密钥*是由 API 提供者生成并授予的唯一字符串，用于授权批准的消费者访问。一旦 API 消费者拥有密钥，他们可以在每次按提供者要求的请求中包含它。提供者通常要求消费者在查询字符串参数、请求头、正文数据或作为 cookie 时传递该密钥。

API 密钥通常看起来像是半随机或随机的数字和字母字符串。例如，查看以下 URL 查询字符串中包含的 API 密钥：

```
/api/v1/users?apikey=**ju574n3x4mpl34p1k3y**
```

以下是作为头部包含的 API 密钥：

```
"API-Secret": **"17813fg8-46a7-5006-e235-45be7e9f2345"**
```

最后，这里是作为 cookie 传递的 API 密钥：

```
Cookie: API-Key= **4n07h3r4p1k3y**
```

获取 API 密钥的过程取决于提供者。例如，NASA 的 API 要求消费者注册 API，提供姓名、电子邮件地址和可选的应用程序 URL（如果用户正在编写一个使用 API 的应用程序），如图 2-3 所示。

![生成 API 密钥的在线表单截图。包括姓名、姓氏、电子邮件和应用程序 URL 字段。](img/F02003.png)

图 2-3：NASA 生成 API 密钥的表单

生成的密钥将类似于以下内容：

> `roS6SmRjLdxZzrNSAkxjCdb6WodSda2G9zc2Q7sK`

它必须作为每个 API 请求中的 URL 参数传递，格式如下：

> *api.nasa.gov/planetary/apod?api_key=roS6SmRjLdxZzrNSAkxjCdb6WodSda2G9zc2Q7sK*

API 密钥在多种原因下比基本身份验证更安全。当密钥足够长、复杂且随机生成时，攻击者猜测或暴力破解它们会变得极为困难。此外，提供商可以设置过期日期来限制密钥的有效时间长度。

然而，API 密钥存在一些相关的风险，我们将在本书后面利用这些风险。由于每个 API 提供商可能有自己的生成 API 密钥的系统，因此你会发现一些情况，其中 API 密钥是基于用户数据生成的。在这些情况下，API 黑客可能通过了解 API 消费者来猜测或伪造 API 密钥。API 密钥还可能暴露在互联网上的在线代码库中，留在代码注释中，或者在未加密的连接上传输时被拦截，甚至通过钓鱼攻击被盗。

### JSON Web Tokens

*JSON Web Token (JWT)* 是一种常用于 API 基于令牌的身份验证的令牌。它的使用方式是：API 消费者使用用户名和密码向 API 提供商进行身份验证。提供商生成 JWT 并将其发送回消费者。消费者将提供的 JWT 添加到所有 API 请求的`Authorization`头部。

JWT 由三部分组成，所有部分都是 base64 编码并通过句点分隔：头部、负载和签名。*头部*包含有关用于签名负载的算法的信息。*负载*是包含在令牌中的数据，例如用户名、时间戳和发行者。*签名*是用于验证令牌的编码和加密消息。

表 2-2 展示了这些部分的示例，未编码以便于阅读，以及最终的令牌。

表 2-2：JWT 组件

| **Component** | **Content** |
| --- | --- |

| Header | `{` `"alg": "HS512",`

`"typ": "JWT"`

`}` |

| Payload | `{` `"sub": "1234567890",`

`"name": "hAPI Hacker",`

`"iat": 1516239022`

`}` |

| Signature | `HMACSHA512(` `base64UrlEncode(header) + "." +`

`base64UrlEncode(payload),`

`SuperSecretPassword`

`)` |

| JWT | `eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6ImhBUEkgSGFja2VyIiwiaWF0IjoxNTE2MjM5MDIyfQ.zsUjGDbBjqI-bJbaUmvUdKaGSEvROKfNjy9K6TckK55sd97AMdPDLxUZwsneff4O1ZWQikhgPm7HHlXYn4jm0Q` |
| --- | --- |

JWT 通常是安全的，但如果实现不当，可能会危及其安全性。API 提供者可以实现不使用加密的 JWT，这意味着你只需要进行一次 base64 解码就可以看到令牌内部的内容。API 黑客可能会解码这样的令牌，篡改内容后再将其发送回提供者，从而获得访问权限，正如你将在第十章看到的那样。JWT 的密钥也可能被盗或通过暴力破解猜到。

### HMAC

*基于哈希的消息认证码（HMAC）*是 Amazon Web Services（AWS）使用的主要 API 认证方法。当使用 HMAC 时，提供者会创建一个密钥并与消费者共享。当消费者与 API 交互时，HMAC 哈希函数会应用于消费者的 API 请求数据和密钥。生成的哈希（也称为*消息摘要*）会添加到请求中并发送给提供者。提供者像消费者一样，通过将消息和密钥输入哈希函数来计算 HMAC，并将输出的哈希值与客户端提供的哈希值进行比较。如果提供者的哈希值与消费者的哈希值匹配，则消费者被授权发起请求。如果值不匹配，则可能是客户端的密钥不正确或消息被篡改。

消息摘要的安全性取决于哈希函数和密钥的加密强度。更强的哈希机制通常会生成更长的哈希值。表 2-3 显示了使用不同 HMAC 算法对相同的消息和密钥进行哈希后的结果。

表 2-3：HMAC 算法

| **算法** | **哈希输出** |
| --- | --- |
| HMAC-MD5 | f37438341e3d22aa11b4b2e838120dcf |
| HMAC-SHA1 | 4c2de361ba8958558de3d049ed1fb5c115656e65 |
| HMAC-SHA256 | be8e73ffbd9a953f2ec892f06f9a5e91e6551023d1942ec7994fa1a78a5ae6bc |
| HMAC-SHA512 | 6434a354a730f888865bc5755d9f498126d8f67d73f32ccd2b775c47c91ce26b66dfa59c25aed7f4a6bcb4786d3a3c6130f63ae08367822af3f967d3a7469e1b |

你可能会对使用 SHA1 或 MD5 有所担忧。截至本书写作时，目前尚未发现影响 HMAC-SHA1 和 HMAC-MD5 的已知漏洞，但这些函数的加密强度低于 SHA-256 和 SHA-512。然而，更安全的函数也更慢。选择使用哪种哈希函数取决于优先考虑性能还是安全性。

与之前介绍的认证方法一样，HMAC 的安全性取决于消费者和提供者保持密钥的私密性。如果密钥被泄露，攻击者可能冒充受害者，获得未授权的 API 访问权限。

### OAuth 2.0

*OAuth 2.0*，或简称*OAuth*，是一种授权标准，允许不同的服务访问彼此的数据，通常使用 API 来促进服务之间的通信。

假设你希望自动将你的 Twitter 推文分享到 LinkedIn。在 OAuth 模型中，我们会将 Twitter 视为服务提供商，将 LinkedIn 视为应用程序或客户端。为了发布你的推文，LinkedIn 需要授权访问你的 Twitter 信息。由于 Twitter 和 LinkedIn 都已经实现了 OAuth，与你每次希望跨平台共享信息时都提供凭证不同，你可以简单地进入 LinkedIn 设置并授权 Twitter。这样做会将你引导到*api.twitter.com*，授权 LinkedIn 访问你的 Twitter 帐户（见图 2-4）。

![截图显示 Twitter 网页询问是否授权 LinkedIn 访问你的帐户。包含授权应用和取消按钮。](img/F02004.png)

图 2-4：LinkedIn–Twitter OAuth 授权请求

当你授权 LinkedIn 访问你的 Twitter 帖子时，Twitter 会为 LinkedIn 生成一个有限的、基于时间的访问令牌。LinkedIn 随后将该令牌提供给 Twitter 以代表你发布内容，你无需向 LinkedIn 提供你的 Twitter 凭证。

图 2-5 展示了 OAuth 的一般过程。用户（*资源所有者*）授予应用程序（*客户端*）访问某个服务（*授权服务器*），服务创建一个令牌，然后应用程序使用该令牌与服务（也是*资源服务器*）交换数据。

在 LinkedIn–Twitter 示例中，你是资源所有者，LinkedIn 是应用程序/客户端，Twitter 是授权服务器和资源服务器。

![图示显示了 LinkedIn 和 Twitter 在 OAuth 授权过程中之间的通信。包括以下步骤：1）授权请求（从 LinkedIn 到用户），2）授权许可（从用户到 LinkedIn），3）授权许可（从 LinkedIn 到用户），4）访问令牌（从授权服务器到 LinkedIn），5）访问令牌（从 LinkedIn 到资源服务器），以及 6）受保护资源（从资源服务器到 LinkedIn）。](img/F02005.png)

图 2-5：OAuth 过程示意图

OAuth 是最受信任的 API 授权方式之一。然而，虽然它为授权过程增加了安全性，但也扩大了潜在的攻击面——尽管漏洞通常更多是与 API 提供商如何实现 OAuth 有关，而不是 OAuth 本身。实现 OAuth 不当的 API 提供商可能会暴露于各种攻击，如令牌注入、授权码重用、跨站请求伪造、无效重定向和钓鱼攻击。

### 无认证

与一般的网页应用程序一样，API 没有任何认证的情况也是有效的。如果一个 API 不处理敏感数据且只提供公共信息，提供方可以认为不需要认证。

## 实际中的 API：探索 Twitter 的 API

阅读完本章和上一章后，你应该能理解运行在网页应用程序 GUI 下的各种组件。接下来，我们通过仔细查看 Twitter 的 API，使这些概念更加具体。如果你打开浏览器并访问网址 [`twitter.com`](https://twitter.com)，初始请求会触发客户端和服务器之间的一系列通信。你的浏览器会自动协调这些数据传输，但通过使用像 Burp Suite 这样的 Web 代理（我们将在第四章中设置），你可以看到所有请求和响应的实际情况。

通信从第一章中描述的典型 HTTP 流量开始：

1.  一旦你在浏览器中输入了 URL，浏览器会自动向 *twitter.com* 的 Web 服务器提交一个 HTTP GET 请求：

    ```
    GET / HTTP/1.1
    Host: twitter.com
    User-Agent: Mozilla/5.0
    Accept: text/html
    `--snip--`
    Cookie: [...]
    ```

1.  Twitter Web 应用程序服务器收到请求后，通过发出成功的 200 OK 响应来回应 GET 请求：

    ```
    HTTP/1.1 200 OK
    cache-control: no-cache, no-store, must-revalidate
    connection: close
    content-security-policy: content-src 'self'
    content-type: text/html; charset=utf-8
    server: tsa_a
    `--snip--`
    x-powered-by: Express
    x-response-time: 56

    <!DOCTYPE html>
    <html dir="ltr" lang="en">
    `--snip--`
    ```

    此响应头包含 HTTP 连接的状态、客户端指令、中间件信息和与 cookie 相关的信息。*客户端指令* 告诉浏览器如何处理请求的信息，例如缓存数据、内容安全策略以及有关发送内容类型的指令。实际的有效载荷从 `x-response-time` 下方开始，它为浏览器提供渲染网页所需的 HTML。

    现在假设用户通过 Twitter 的搜索框查找 “hacking” 这个词。这将触发一个 POST 请求到 Twitter 的 API，如下所示。Twitter 能够利用 API 分发请求，并无缝地将请求的资源提供给多个用户。

    ```
    POST /1.1/jot/client_event.json?q=hacking HTTP/1.1
    Host: api.twitter.com
    User-Agent: Mozilla/5.0
    `--snip--`
    Authorization: Bearer AAAAAAAAAAAAAAAAA...
    `--snip--`
    ```

    这个 POST 请求是 Twitter API 查询 *api.twitter.com* 网络服务的一个例子，搜索词为 “hacking”。Twitter API 使用 JSON 格式的搜索结果回应，结果包括推文和有关每条推文的信息，如用户提及、标签和发布时间：

    ```
    "created_at": [...]
    "id":1278533978970976256
    "id_str": "1278533978970976256"
    "full-text": "1984: William Gibson published his debut novel..."
    "truncated":false,
    `--snip--`
    ```

    Twitter API 遵循 CRUD、API 命名规范、授权令牌、*application/x-www-form-urlencoded* 和 JSON 数据交换格式这一事实，清楚表明这个 API 是一个 RESTful API。

    尽管响应体格式化为可读的方式，但它是供浏览器处理并显示为人类可读的网页。浏览器使用来自 API 请求的字符串渲染搜索结果。提供者的响应随后将页面填充为搜索结果、图像以及社交媒体相关信息，如喜欢、转发、评论（见 图 2-6）。

![Screenshot of a Twitter search for the word “hacking”](img/F02006.png)

图 2-6：来自 Twitter API 搜索请求的渲染结果

从终端用户的角度来看，整个互动过程显得无缝衔接：你点击搜索框，输入查询内容，随后收到结果。

## 总结

在这一章中，我们介绍了 API 的术语、组成部分、类型以及支持架构。你了解到 API 是与 Web 应用程序交互的接口。不同类型的 API 有不同的规则、功能和目的，但它们都使用某种格式在应用程序之间交换数据。它们通常使用身份验证和授权机制，以确保消费者只能访问他们应该访问的资源。

理解这些概念将帮助你自信地深入了解构成 API 的各个组件。当你继续阅读时，如果遇到令你困惑的 API 概念，可以参考本章内容。
