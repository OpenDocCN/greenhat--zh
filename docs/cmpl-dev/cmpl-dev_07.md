

## 第六章：6 REST 和 GRAPHQL API



![](img/Drop-image.jpg)

*API* 是一种通用模式，用于连接计算机或计算机程序。与用户界面不同，它的设计目的是让软件而非用户来访问。API 的一个目的是隐藏系统内部工作细节，同时暴露出一个标准化的网关，供访问系统的数据或功能。

作为全栈开发人员，你通常会与两种类型的 API 进行交互或*使用*：内部 API 和第三方 API。在查询内部 API 时，你是从自己的系统中获取数据，通常来自自己的数据库或服务。私有 API 不对外部方开放。例如，你的银行可能使用私有 API 来检查其内部系统中的信用评分或账户余额，并将这些信息显示在你的在线银行账户页面上。

第三方 API 提供对外部系统数据的访问。例如，你将在第二部分中实现的 OAuth 登录就使用了一个 API。你还可以使用 API 从外部提供者获取社交媒体动态或天气信息，并在网站上展示它们。由于外部 API 对公众开放，你可以通过公共 URL 访问它们，它们会在*API 合同*中记录你应使用的访问数据的约定。这个合同定义了通信格式、API 期望的参数以及每个请求可能收到的响应。我们在第三章中简要讨论了 API 和功能合同，并解释了为什么你应该为它们指定类型。

全栈 web 开发主要使用两种类型的 API，分别是 REST 和 GraphQL，它们都通过 HTTP 传输数据。本章将介绍这两种 API。

### REST API

REST 是一种用于设计 RESTful web API 的架构模式。这些 API 本质上是一组 URL，每个 URL 提供对单一资源的访问。它们依赖于使用 HTTP 方法和标准 HTTP 状态码来传输数据，并接受 URL 编码或请求头参数。通常，它们会以 JSON 或纯文本的形式响应请求的数据。

实际上，你已经构建了第一个 REST API。回想一下你在第五部分练习中创建的 Next.js 服务器，它提供了*api/weather/:zipcode*端点。到目前为止，我们已使用这个端点来玩转 Next.js 的路由，理解动态 URL，并学习如何访问查询参数。然而，很快你就会发现，这个 API 遵循了 REST 的约定：为了访问它，我们使用了 HTTP 的 GET 方法来访问 URL 端点，并收到了一个带有 HTTP 状态码*200: OK*的 JSON 响应。常见的状态码范围是*2XX*表示请求成功，*3XX*表示重定向。如果请求失败，我们会看到*4XX*范围表示与客户端相关的错误，例如*401: Unauthorized*，以及*5XX*表示服务器错误，通常是通用的*500: Internal Server Error*。

作为全栈开发人员，我们有时可能会创建自己的 API；但更多的时候，我们会使用第三方 API。为什么我们会使用第三方的天气 API 呢？好吧，想象一下，我们希望我们的应用在多个远程地点显示当前天气。与其自行设置并维护多个天气站，然后从传感器中读取数据，这不仅需要为每个天气站提供和消费 API，不如直接从现有天气服务提供的第三方 API 中获取数据。我们的代码可能会调用该 API，传递一个邮政编码作为参数，并以预定的格式接收该地点的天气数据。然后，我们会在网站上显示这些数据。

RESTful API 使我们能够与数据交互，而无需了解数据是如何存储的或由何种底层技术提供的。如果你遵循 API 的规范，即使底层技术或架构发生变化，你也应该能够收到请求的数据。除此之外，还有一些要求，API 才能被认为是 RESTful 的。

#### URL

一个独特的 URL 为 RESTful API 提供了接口。每个提供者的 API 通常都有相同的基本 URL，称为*根入口点*，例如*http://localhost:3000/api*。你可以把它看作是 API 的姓氏。通常，你会看到根入口点后面加上版本号，因为提供者可能有多个 API 版本。例如，可能会有旧版的*http://localhost:3000/api/v1*和新版的*http://localhost:3000/api/v2*。为了遵循这个模式，你可以在*api*文件夹中创建一个*v1*文件夹，并将 REST API 代码移动到该文件夹中。

> 注意

*API 版本控制的其他常见方式包括自定义头部和查询字符串。在第一种情况下，客户端会使用自定义的 Accept-Version 头部请求 API，并收到匹配的 Content-Version 头部。在第二种情况下，API 请求会在 URL 中使用* ?version=1.0.0 *作为查询参数。*

API URL 的下一部分是路径，通常被称为*端点*。它指定我们想要查询的资源（例如天气 API）。API 规范通常只提到端点本身，如*/v1/weather*，并暗示根入口点。URL 通常还接受参数。这些可以是路径参数，它们是 URL 的一部分，例如我们的邮政编码 API 端点*/v1/weather/{zipcode}*，也可以是查询参数，它们作为编码的键值对在初始问号后添加，如*/v1/weather?zipcode*=<*zipcode*>。按照惯例，路径参数通常用于表示一个或多个资源，而查询参数用于对返回的数据执行操作，如排序或过滤。

#### 规范

资源本身与返回给客户端的表示是分开的。换句话说，服务器可能会以 HTML、XML、JSON 等格式发送数据，无论数据是如何存储在应用程序的数据库中的。你可以在 API 的*规范*中了解 API 的响应格式，它作为 API 的手册。记录 API 的一种优秀方法是使用 OpenAPI 格式，它在行业中被广泛使用，并且是 Linux 基金会的一部分。你可以使用 Swagger 图形编辑器，访问[*https://<wbr>editor<wbr>.swagger<wbr>.io*](https://editor.swagger.io)进行实验。

例如，清单 6-1 显示了一个*v1/weather/{zipcode}*端点的规范，格式为 JSON。将代码粘贴到 Swagger 编辑器中，以更友好的方式浏览生成的文档。

```
{
    "openapi": "3.0.0",
    "info": {
        "title": "Sample Next.js - OpenAPI 3.x",
        "description": "The example APIs from our Next.js application",
        "version": "1.0.0"
    },
    "servers": [
        {"url": "https://www.usemodernfullstack.dev/api/"},
        {"url": "http://localhost:3000/api/"}
    ],
    "paths": {
        "/v1/weather/{zipcode}": {
            "get": {
                "summary": "Get weather by zip code",
                "parameters": [
                    {
                        "name": "zipcode",
                        "in": "path",
                        "description": "The zip code for the location as string.",
                        "required": true,
                        "schema": {
                          "type": "string",
                          "example": 96815
                        }
                    }
                ],
                "responses": {
                    "200": {
                        "description": "Successful operation",
                        "content": {
                            "application/json": {
                                "schema": {
                                    "$ref": "#/components/schemas/weatherDetailType"
                                }
                              }
                        }
                    }
                }
            }
          }
    },
    "components": {
        "schemas": {
            "weatherDetailType": {
                "type": "object",
                "properties": {
                    "zipcode": {
                        "type": "string",
                        "example": 96815
                    },
                    "weather": {
                        "type": "string",
                        "example": "sunny"
                    },
                    "temp": {
                        "type": "integer",
                        "format": "int64",
                        "example": 35
                    }
                }
            }
        }
    }
} 
```

清单 6-1：/v1/weather/{zipcode}端点的 OpenAPI 规范

首先我们定义一般信息，如 API 的标题和描述。这里最重要的值是 API 版本。在练习 6 中，我们将调整服务器以反映这个版本。在下一步中，我们设置服务器，即 API 的根入口点。这里我们使用 localhost，因为我们的 Next.js 应用程序现在是在本地运行的。

然后我们在paths下指定唯一的 API 端点，设置每个端点的路径、参数和响应。在这个示例中，我们为一个端点*/v1/weather/{zipcode}*指定了最小的必需数据，并明确指出它使用 GET 方法。大括号({})表示 URL 参数，但我们也在路径中显式设置了名为zipcode的参数。此外，我们定义了*schema*，即该参数的格式，应该是一个字符串。

接下来，在 responses 部分，我们设置了如果 HTTP 状态码为 *200: OK* 时，API 应该返回的响应。该内容采用 application/json 格式，是 weatherDetailType，你应该已经从之前的章节中熟悉了它。它类似于我们 *custom.d.ts* 文件中的自定义类型定义，只不过这里我们使用的是 JSON，而不是 TypeScript。

请注意，Swagger 编辑器还会根据规范生成一个交互式的“游乐场”，允许我们在运行中的服务器上测试 API 的端点。此外，我们可以直接在编辑器的界面中生成服务器和客户端。生成的服务器将提供规范中描述的 REST API，而客户端将生成一个库，我们可以在任何使用该 API 的应用程序中使用。这个交互式“游乐场”和生成的代码使得与第三方 API 的协作变得非常简单。

#### 状态与身份验证

RESTful API 是*无状态的*，这意味着它们不在服务器上存储会话信息。*会话信息*是关于之前用户交互的任何数据。例如，想象一个在线商店的购物车。在有状态设计中，应用程序会将购物车的内容存储在服务器上，并在你添加新商品时进行更新。在 RESTful 设计中，客户端会在每次请求中发送所有相关的会话数据。用户与服务器的交互是孤立理解的，不依赖于之前请求的上下文。

尽管如此，公共的 RESTful API 通常要求某种形式的身份验证。为了区分认证用户的请求和未认证用户的请求，这些 API 通常会提供一个令牌，用户应该在后续的请求中包含该令牌。消费者将此令牌作为请求数据的一部分，或者放在 HTTP Authorization 头部。我们将在第九章中提供关于授权令牌及其工作原理的更多细节。

这种无状态设计意味着，无论客户端是直接从终端服务器、代理服务器，还是负载均衡器请求数据，身份验证都能正常工作。因此，RESTful API 能够处理分层系统。无状态架构在高流量情况下也很理想，因为它们消除了因从数据库检索会话信息而导致的服务器负载。

#### HTTP 方法

在 REST 中，有四种标准方式与数据集交互：创建、读取、更新和删除。这些交互通常被称为*CRUD*操作。REST API 使用以下 HTTP 方法在请求的资源上执行这些操作：

**GET  **用于从资源中检索数据。这是最常见的请求；每当你在浏览器中访问一个网站时，实际上就是向该网站地址发送 GET 请求。

**POST  **用于向集合资源中添加新元素。多次发送相同的 POST 请求会为每个请求创建一个新的元素，导致多个具有相同内容的元素。当你发送电子邮件或提交网页表单时，你的客户端通常会在后台发送一个 POST 请求，因为你在数据库中创建了一个新资源。

**PUT  **用于覆盖或更新现有资源。多次发送相同的 PUT 请求会创建或覆盖一个元素并更新其内容。例如，当你重新上传 Instagram 或 Facebook 上的图片时，你可能会发送一个 PUT 请求。

**PATCH  **用于部分更新现有资源。与 PUT 不同，你只会发送与当前数据集不同的数据。因此，这是一项更小且性能更好的操作。例如，你在社交媒体页面上更新个人资料时，可能会使用 PATCH 请求。

**DELETE  **用于删除资源（例如，在 Instagram 上删除一张图片）。

REST API 请求面临与所有 HTTP 请求相同的性能问题。开发者必须考虑关键因素，如网络带宽、延迟和服务器负载。虽然应用程序通常无法影响网络延迟或用户带宽，但它可以通过缓存请求并返回之前缓存的结果来提高性能。

通常，推荐的做法是积极缓存请求。通过避免额外的服务器请求，我们可以显著加快应用程序的速度。不幸的是，并非所有 HTTP 请求都可以缓存。GET 请求的响应默认是可缓存的，但 PUT 和 DELETE 的响应完全不可缓存，因为它们不能保证可预测的响应。在两个类似的 PUT 请求之间，可能会有 DELETE 请求删除了资源，或者反之。理论上，POST 和 PATCH 请求的响应可以缓存，如果响应提供了<code class="SANS_TheSansMonoCd_W5Regular_11">Expire</code>头部或<code class="SANS_TheSansMonoCd_W5Regular_11">Cache-Control</code>头部，并且后续调用是对相同资源的 GET 请求。然而，服务器通常不会缓存这两种类型的请求。

### <code class="SANS_Futura_Std_Bold_B_11">与 REST 工作</code>

让我们通过查看一个虚构的天气服务来练习使用 REST。假设我们阅读了 API 文档，发现授权用户可以通过使用其公开的 REST API 接收和更新服务中的数据集。该 API 返回 JSON 数据，服务器的 URL 是*https://www.usemodernfullstack.dev*，并且*/api/v2/weather/{zipcode}*端点支持 GET 和 PUT 请求。在本节中，我们将通过 GET 请求获取特定邮政编码的当前天气数据，以及通过 PUT 请求更新存储的天气数据。

#### 读取数据

为了获取你所在位置的天气，你可能会发出一个包含邮政编码 96815 和授权令牌的 GET 请求。我们可以使用像 cURL 这样的命令行工具来发出此类 GET 请求，cURL 应该是你系统的一部分。如有必要，你可以从[*https://<wbr>curl<wbr>.se*](https://curl.se)安装它。一个典型的 cURL 请求如下所示：

```
$ **curl -i** **`url`**
```

-i标志显示我们感兴趣的头部详情。我们可以使用-X标志设置 HTTP 方法，并通过-H标志发送额外的头部。使用转义字符发送多行命令（macOS 上使用\，Windows 上使用^）。避免在转义字符后添加空格。如果你感兴趣，可以尝试使用 cURL 查询你在第五章练习第 89 页中创建的应用程序的一个 API 端点。对天气 API *v2/weather/{zipcode}* 发起 GET 请求的 cURL 调用如下所示：

```
$ **curl -i \**
    **-X GET \**
    **-H "Accept: application/json" \**
    **-H "Authorization: Bearer 83dedad0728baaef3ad3f50bd05ed030" \**
    **https://www.usemodernfullstack.dev/api/v2/weather/96815** 
```

我们向服务器上的 API 端点 *v2/weather/{zipcode}* 发出此请求，服务器的地址为 *https://www.usemodernfullstack.dev/api*。邮政编码包含在 URL 中。我们在Accept头部中设置返回格式为 JSON，并在Authorization头部中传递访问令牌。由于这是一个示例 API，它接受任何令牌；如果未提供令牌，API 将返回状态码*401*。

这是 API 对我们 GET 请求的响应示例：

```
HTTP/2 200
content-type: application/json ; charset=utf-8
access-control-allow-origin: *

{"weather":"sunny","tempC":"25","tempF":"77","friends":["96814","96826"]} 
```

API 响应的 HTTP 状态码为*200*，表示请求成功。我们要求返回 JSON 响应，并且content-type头部确认响应数据确实是该类型。

Access-Control-Allow-Origin头部，我们在第五章中讨论过，它允许访问任何域名。通过这个设置，任何客户端 JavaScript 希望访问 API 的浏览器都会允许这些请求，而不管网站的域名是什么。如果没有 CORS 头部，浏览器会阻止请求，脚本无法访问响应，反而会抛出 CORS 错误。

最后，我们看到响应体包含一个 JSON 字符串，包含 API 的响应。

#### 更新数据

现在假设你想在你的网站上添加来自你所在邻里的显示数据（邮政编码 96814）以及邻近区域（邮政编码 96826）的数据。不幸的是，这些邮政编码还没有在 API 中提供。幸运的是，由于它是开源的，我们可以连接我们自己的气象站并扩展系统。假设我们已经设置了气象传感器并将它们连接到 API。一旦天气发生变化，我们就将数据集添加到其中。

这里是我们发送的 PUT 请求，用于更新邮政编码 96814 的天气数据。PUT 请求将数据存储在请求体中；因此，我们在 cURL 命令中使用-d标志来发送编码后的 JSON：

```
$ **curl -i \**
    **-X PUT \**
    **-H "Accept: application/json" \**
    **-H "Authorization: Bearer 83dedad0728baaef3ad3f50bd05ed030" \**
    **-H "Content-Type: application/json" \**
    **-d "{\"weather\":\"sunny\",\"tempC\":\"20\",\"tempF\":\"68\",**
        **\"friends\":\"['96815','96826']\"}" \**
    **https://www.usemodernfullstack.dev/api/v2/weather/96815** 
```

我们请求相同的 API 端点，*/api/v2/weather/*，但将 GET 方法替换为 PUT，因为我们不想从数据库获取数据，而是想添加数据。我们使用Content-Type头部告诉 API 提供者请求体中的有效负载是一个 JSON 字符串。API 更新数据集并返回状态码*200*，以及包含额外状态信息的 JSON 对象：

```
HTTP/2 200
content-type: application/json ; charset=utf-8
access-control-allow-origin: *
{"status":"ok"} 
```

你可以在[*https://<wbr>restfulapi<wbr>.net*](https://restfulapi.net)了解更多关于 RESTful API 的内容，该网站涵盖了更具体的话题，比如压缩和安全模型，并指导你设计自己的 RESTful API。现在，让我们将注意力转向 GraphQL，这是一种不同且更先进的 API 类型。

### GraphQL API

与 REST 不同，GraphQL 不仅仅是一个架构模式。它是一个完整的、开源的数据查询和操作语言，专为 API 设计。它也是全栈 Web 开发中最受欢迎的 REST 替代方案，Airbnb、GitHub、PayPal 等众多公司都在使用它。事实上，据报道，前 10,000 个网站中有 10%的站点使用 GraphQL。本节仅涵盖其部分功能，但应该能让你对 GraphQL 的原理有一个扎实的理解。

> 注意

*尽管其名称中带有“Graph”，GraphQL 并不要求使用像 Neo4j 这样的图数据库。我们可以用它查询连接到 GraphQL 服务器的任何数据源，包括常见的数据库如 MySQL 和 MongoDB。*

和 REST 一样，GraphQL API 也是通过 HTTP 操作的。然而，GraphQL 的实现只暴露一个单一的 API 端点，通常称为 */graphql*，用于访问所有资源并执行所有 CRUD 操作。相比之下，REST 为每个资源提供一个专门的端点。

另一个区别是，我们只能通过 POST 请求连接到 GraphQL 服务器。与其使用 HTTP 方法来定义所需的 CRUD 操作，我们在 POST 请求体中使用查询和变更。*查询*是读取操作，而*变更*则是创建、更新和删除数据的操作。

与依赖标准 HTTP 状态码的 REST 不同，GraphQL 在操作无法执行时会返回*500*，即*内部服务器错误*。否则，即使查询或变更存在问题，响应也会使用*200*。这是因为解析器可能在遇到问题之前已经部分执行。部署 GraphQL API 到生产环境时，需牢记这一点。许多标准操作实践和工具可能需要调整，以适应这一行为。

#### 架构

GraphQL API 在其架构中定义了可用的查询和变更，这相当于 REST API 的规范。架构也叫做*typedef*，它是用架构定义语言（SDL）编写的。SDL 的核心元素是*类型*，这些类型是包含有类型的*字段*的对象，字段定义了它们的属性，还有可选的*指令*，它们可以添加额外的信息，例如，指定查询的缓存规则或标记字段为废弃。

列表 6-2 展示了我们虚构的天气 API 的 GraphQL 架构，它返回某个地点的天气数据。

```
export const typeDefs = gql`

    type LocationWeatherType {
        zip: String!
        weather: String!
        tempC: String!
        tempF: String!
        friends: [String]!
    }

    input LocationWeatherInput {
        zip: String!
        weather: String
        tempC: String
        tempF: String
        friends: [String]
    }

    type Query {
        weather(zip: String): [LocationWeatherType]!
    }

    type Mutation {
        weather(data: LocationWeatherInput): [LocationWeatherType]!
    }
`; 
```

列表 6-2：天气 API 的 GraphQL 架构

你应该注意到，架构是一个标签模板字面量，这是你在第二章中学到的内容。我们首先描述自定义的 GraphQL 对象类型。这些对象类型表示 API 返回的数据。它们类似于我们在 TypeScript 中定义的自定义类型。一个类型有一个名称，并可以实现一个接口。每个自定义对象类型包含字段，每个字段都有名称和类型。GraphQL 有内建的标量类型 Int、Float、String、Boolean 和 ID。感叹号 (!) 表示不可为空的字段，而方括号 ([]) 表示数组。

第一个自定义对象类型是 LocationWeatherType，用于描述天气查询的位置信息。这里我们使用 String! 表达式将 ZIP 字段标记为非空。由此，GraphQL 服务始终会为此字段返回一个值。我们将 friends 字段定义为一个字符串数组，表示通过 ZIP 代码关联的天气站。它也是非空的，因此在添加到返回值中时，它将始终包含一个数组（零个或多个项）。friends 数组中的 String 确保每个项都是一个字符串。

然后，我们为第一个变更定义输入类型对象。这些类型在变更中是必要的，它们代表从 API 消费者处接收到的输入。因为消费者应该仅传入他们想要更新的字段，所以我们省略了感叹号，使字段变为可选。在 GraphQL 中，我们需要分别为返回值定义输入对象和类型，使用内建类型。与 TypeScript 不同，我们不能使用通用自定义类型。

模式还定义了查询和变更函数。这些是消费者可以发送到 API 的操作。weather 查询将 ZIP 代码作为参数，并始终返回一个 WeatherLocationType 对象数组。weather 变更函数接受一个 WeatherLocationInput 参数，并始终返回修改后的 WeatherLocationType 对象。

我们的模式中没有包含任何指令，且在本章中我们不会深入探讨它们的语法。然而，使用指令的一个原因是缓存。由于 GraphQL 查询使用 POST 方法，而默认的 HTTP 缓存无法缓存 POST 请求，因此我们必须在服务器端手动实现缓存。我们可以通过在类型定义中静态配置指令 @cacheControl 或在解析器函数中动态配置 cacheControl.setCacheHint 来实现缓存。

#### 解析器

在 GraphQL 中，*解析器*是实现模式的函数。每个解析器函数映射到一个字段。查询解析器实现数据的读取，而变更解析器实现数据的创建、更新和删除。它们共同提供完整的 CRUD 功能。

为了理解解析器是如何工作的，你可以将每个 GraphQL 操作看作是一个嵌套函数调用的树。在这样的*抽象语法树（AST）*中，操作的每个部分代表一个节点。例如，考虑一个复杂的嵌套 GraphQL 查询，它请求位置的当前天气以及所有邻居的天气。我们为此示例设计的 GraphQL 架构如清单 6-3 所示。

```
export const typeDefs = gql`

    **type FriendsType {**
 **zip: String!**
 **weather: String!**
    **}**

    type LocationWeatherType {
        zip: String!
        weather: String!
        tempC: String!
        tempF: String!
        friends: [**FriendsType**]!
    }

    type Query {
        weather(zip: String): [LocationWeatherType]!
    }
`; 
```

清单 6-3：嵌套 GraphQL 查询示例的 GraphQL 架构

在这个示例的架构中，我们替换了friends数组的内容。我们希望它包含一个对象，而不是一个简单的字符串，这个对象包含邮政编码和天气信息。因此，我们定义了一个新的FriendsType并使用该类型作为数组项的类型。

清单 6-4 定义了复杂的示例查询。

```
query GetWeatherWithFriends {
    weather(zip: "96815") {
        weather
        friends {
            weather
        }
    }
} 
```

清单 6-4：嵌套的 GraphQL 查询

这个查询接受 zip 参数 96815，然后返回其 weather 属性，以及所有朋友的 weather 属性，作为字符串。但这个查询在幕后是如何工作的呢？

图 6-1 展示了解析器链和相应的 AST。GraphQL 服务器首先将查询解析成这种结构，然后将 AST 与类型定义架构进行验证，以确保查询可以在不遇到逻辑问题的情况下执行。最后，服务器执行查询。

![](img/Figure6-1.jpg)

图 6-1：查询 GraphQL AST

让我们来检查这个查询的解析器链。Query.weather 函数接受一个参数，即邮政编码，并返回该邮政编码对应的 location 对象。然后，服务器分别沿着每个分支继续执行。对于查询中的天气，它返回 location 对象的 weather 属性，即 Location.weather，此时分支结束。查询的第二部分请求所有来自 location 对象的朋友及其 weather 属性，它执行 Location.Friends 查询，然后返回每个结果的 Friends.weather 属性。每个步骤的解析器对象包含由父字段的解析器返回的结果。

让我们回到我们的天气架构并定义解析器。我们将保持这些解析器简单。在 清单 6-5 中，你可以看到它们的名称与架构中定义的名称一致。

```
export const resolvers = {
    Query: {
        weather: async (_: any, param: WeatherInterface) => {
            return [
                {
                    zip: param.zip,
                    weather: "sunny",
                    tempC: "25C",
                    tempF: "70F",
                    friends: []
                }
            ];
        },
    },
    Mutation: {
        weather: async (_: any, param: {data: WeatherInterface}) => {
            return [
                {
                    zip: param.data.zip,
                    weather: "sunny",
                    tempC: "25C",
                    tempF: "70F",
                    friends: []
                }
            ];
        }
    },
}; 
```

清单 6-5：天气 API 的 GraphQL 解析器

我们首先为查询和变更属性定义异步函数，并将对象赋值给常量 resolvers。每个函数都有两个参数。第一个参数表示解析器链中前一个解析器对象。我们没有使用嵌套或复杂的查询；因此，在这里它始终是未定义的。为此，我们使用 any 类型来避免 TypeScript 错误，并使用你在 第三章 中学到的下划线 (_) 约定将其标记为未使用。第二个参数是一个包含在调用时传递给函数的数据的对象。对于 weather 查询和 weather 变更，它是一个实现了 WeatherInterface 的对象。

目前，两个函数大部分时间忽略这个参数，仅使用 zip 属性来反映输入。此外，它们返回一个类似于我们在前面列表中创建的 REST API 的静态 JSON 对象。静态数据仅是一个占位符，稍后我们将用来自数据库查询的结果替换它。该响应遵循我们在 GraphQL 架构中定义的 API 合同，因为这些数据是包含天气位置数据集的数组。

### 将 GraphQL 与 REST 进行比较

我们已经在 Next.js 应用程序中实现了 RESTful API，正如你在本章中看到的，REST 相对简单易用。你可能会想，为什么还要考虑使用 GraphQL。其实，GraphQL 解决了 REST API 中常见的两个问题：过度获取（over-fetching）和获取不足（under-fetching）。

#### 过度获取（Over-Fetching）

当客户端查询 REST 端点时，API 总是返回该端点的完整数据集。这意味着，通常情况下，API 返回的数据比实际需要的要多，这是一个常见的性能问题，称为 *过度获取*。例如，我们的示例 RESTful 天气 API 在 */api/v2/weather/zip/96815* 返回该邮政编码的所有天气数据，即使你只需要摄氏温度。你还需要手动筛选结果。而在 GraphQL 中，API 请求明确指定它们想要返回的数据。

让我们来看一个例子，看看 GraphQL 如何让我们将 API 响应数据保持在最低限度。以下 GraphQL 查询仅返回 ZIP 码为 96815 的位置的摄氏温度：

```
query Weather {
    weather(zip: "96815") {
        tempC
    }
} 
```

在 GraphQL 中，我们通过 POST 请求的数据将查询作为 JSON 字符串发送：

```
$ **curl -i \**
 **-X POST \**
 **-H "Accept: application/json" \**
 **-H "Authorization: Bearer 83dedad0728baaef3ad3f50bd05ed030" \**
 **-H "Content-Type: application/json" \**
 **-d '{"query":"\nquery Weather  {\n  weather(zip: \"96815\") {\n    tempC  \n  }\n}"}' \**
 **https://www.usemodernfullstack.dev/api/graphql** 
```

我们使用 POST 请求访问 */api/graphql* 端点，设置 Content-Type 和 Accept 头为 JSON，明确告知 API 我们正在请求体中发送一个 JSON 对象，并期待 JSON 响应。我们像在 RESTful 请求中一样，在 Authorization 头中设置访问令牌。POST 请求体包含天气数据的查询，\n 控制字符表示 GraphQL 查询中的换行符。如合同所定义，查询期望一个参数 zip，我们传入了邮政编码 96815。另外，我们请求 API 仅返回天气节点中的 tempC 字段。

下面是来自 GraphQL API 的响应：

```
HTTP/2 200
content-type: application/json ; charset=utf-8
access-control-allow-origin: *

{"data":{"weather":[{"tempC":"25C"}]}} 
```

API 以 *200* 状态码响应。我们在请求的查询中指定，只关心 weather 对象中的 tempC 字段，所以这就是我们接收到的数据。API 不返回邮政编码、华氏温度、天气字符串或朋友数组。

#### 欠取数据

另一方面，REST 数据集可能不包含您所需的所有数据，这需要您发送后续请求。这种问题被称为 *欠取数据*。假设您的朋友也有气象站，您想获取他们邮政编码的当前天气。RESTful 天气 API 返回一个包含相关邮政编码（friends）的数组。然而，您需要为每个邮政编码发出额外的请求，以获取他们的天气信息，这可能导致性能问题。

GraphQL 将数据集视为图中的节点，并且它们之间存在关系。因此，扩展单一查询以接收相关数据非常简单。我们的示例 GraphQL 服务器的解析器已设置好，如果请求的查询包含 friends 字段，则会获取关于朋友的额外数据。我们定义 GraphQL 查询如下：

```
query Weather {
    weather(zip: "96815") {
        tempC
        friends {
            tempC
        }
    }
} 
```

以下是一个示例请求，展示如何通过 friends 数组获取所有相关节点。再次地，我们定义返回数据并仅查询 tempC 字段：

```
$ **curl -i \**
 **-X POST \**
 **-H "Accept: application/json" \**
 **-H "Authorization: Bearer 83dedad0728baaef3ad3f50bd05ed030" \**
 **-H "Content-Type: application/json" \**
 **-d '{"query":"query Weather  {\n  weather(zip: \"96815\")**
 **{\n    tempC\n    friends {\n      tempC\n    }\n  }\n}"}' \**
 **https://www.usemodernfullstack.dev/api/graphql** 
```

POST 请求体包含了针对 96815 邮政编码的天气数据查询，要求返回 tempC 字段，和之前的请求一样。为了扩展查询，我们在 friends 字段上添加了一个子选择。现在，GraphQL 会遍历相关节点及其字段，并返回与 96815 节点的 friends 数组中的邮政编码匹配的节点的 tempC 字段。

这是来自 GraphQL 服务器的响应。我们看到它包含了相关节点的数据：

```
HTTP/2 200
content-type: application/json ; charset=utf-8
access-control-allow-origin: *

{"data":{"weather":[{"tempC":"25C","friends":
[{"tempC":"20C"},{"tempC":"30C"}]}]}} 
```

正如你已经发现的，GraphQL 让我们通过调整请求中的数据轻松扩展查询。

练习 6：将 GraphQL API 添加到 Next.js

让我们重新设计天气应用程序的 API 来使用 GraphQL。为此，我们必须首先将 GraphQL 添加到项目中。GraphQL 不是一种模式，而是一个由服务器和查询语言组成的环境，我们必须将它们都添加到 Next.js 中。

我们将安装独立的 Apollo 服务器，它是最流行的 GraphQL 服务器之一，也提供了 Next.js 的集成。打开终端并导航到你在 第五章 中构建的重构应用程序。在目录的顶层，靠近 *package.json* 文件，执行以下命令：

```
$ **npm install @apollo/server @as-integrations/next graphql graphql-tag**
```

这个命令还会安装我们所需的 GraphQL 语言和 GraphQL 标签模块。

#### 创建 Schema

正如我们所讨论的，每个 GraphQL API 都是从 schema 定义开始的。在 Next.js 目录中创建一个名为 *graphql* 的文件夹，紧挨着 *pages* 文件夹。这将是我们添加所有 GraphQL 相关文件的地方。

现在创建一个名为 *schema.ts* 的文件，并粘贴你在 清单 6-2 中写的代码。我们已经定义并讨论了这里使用的类型定义。只需在文件顶部添加一行：

```
import gql from "graphql-tag";
```

这一行导入了我们用来定义 schema 的 qql 标记模板字面量。

#### 添加数据

我们希望我们的 API 根据传递给它的查询的参数和属性返回不同的数据。因此，我们需要将数据集添加到项目中。GraphQL 可以查询任何数据库，甚至是静态的 JSON 数据。所以让我们实现一个 JSON 数据集。在 *graphql* 目录中创建 *data.ts* 文件，并添加来自 清单 6-6 的代码。

```
export const db = [
    {
        zip: "96815",
        weather: "sunny",
        tempC: "25C",
        tempF: "70F",
        friends: ["96814", "96826"]
    },
    {
        zip: "96826",
        weather: "sunny",
        tempC: "30C",
        tempF: "86F",
        friends: ["96814", "96814"]
    },
    {
        zip: "96814",
        weather: "sunny",
        tempC: "20C",
        tempF: "68F",
        friends: ["96815", "96826"]
    }
]; 
```

清单 6-6：用于 GraphQL API 的 graphql/data.ts 文件

这个 JSON 定义了三个天气位置及其属性。消费者将能够查询我们的 API 获取这些数据集。

#### 实现解析器

现在我们可以定义解析器了。将文件 *resolvers.ts* 添加到 *graphql* 目录中，并粘贴 Listing 6-7 中的代码。这与我们之前讨论的代码类似，当时我们介绍了解析器，但是这次我们查询的是我们新的数据集，而不是返回相同的静态 JSON 对象给消费者。

```
import {db} from "./data";

declare interface WeatherInterface {
    zip: string;
    weather: string;
    tempC: string;
    tempF: string;
    friends: string[];
}

export const resolvers = {
    Query: {
        weather: async (_: any, param: WeatherInterface) => {
            return [db.find((item) => item.zip === param.zip)];
        }
    },
    Mutation: {
        weather: async (_: any, param: {data: WeatherInterface}) => {
            return [db.find((item) => item.zip === param.data.zip)];
        }
    }
}; 
```

Listing 6-7: GraphQL API 的 graphql/resolvers.ts 文件

我们导入之前创建的 JSON 对象数组，并为解析器定义接口。查询解析器通过使用传入的邮政编码查找对象，并将其返回给 Apollo 服务器。突变（mutation）执行相同的操作，只是参数结构稍有不同：它可以通过data属性访问。不幸的是，我们实际上无法通过使用突变来更改数据，因为数据是静态的 JSON 文件。我们在这里实现突变只是为了说明问题。

#### 创建 API 路由

Apollo GraphQL 服务器暴露一个端点，*graphql/*，我们现在将实现它。创建一个新的文件，*graphql.ts*，并将 Listing 6-8 中的代码添加到 *api* 文件夹中。此代码初始化 GraphQL 服务器，并添加 CORS 头，以便我们可以从不同的域访问 API，并稍后使用内置的 GraphQL 沙盒浏览器来玩转 GraphQL。你在之前的 cURL 响应中看到过这个头部。

```
import {ApolloServer} from "@apollo/server";
import {startServerAndCreateNextHandler} from "@as-integrations/next";
import {resolvers} from "../../graphql/resolvers";
import {typeDefs} from "../../graphql/schema";
import {NextApiHandler, NextApiRequest, NextApiResponse} from "next";

//@ts-ignore
const server = new ApolloServer({
    resolvers,
    typeDefs
});

const handler = startServerAndCreateNextHandler(server);

const allowCors =
    (fn: NextApiHandler) => async (req: NextApiRequest, res: NextApiResponse) => {
        res.setHeader("Allow", "POST");
 res.setHeader("Access-Control-Allow-Origin", "*");
        res.setHeader("Access-Control-Allow-Methods", "POST");
        res.setHeader("Access-Control-Allow-Headers", "*");
        res.setHeader("Access-Control-Allow-Credentials", "true");

        if (req.method === "OPTIONS") {
            res.status(200).end();
        }
        return await fn(req, res);
    };

export default allowCors(handler); 
```

Listing 6-8: 创建 GraphQL 的 API 入口点的 api/graphql.ts 文件

这些代码就是我们创建 GraphQL 入口点所需的全部内容。首先，我们导入必要的模块，包括我们之前创建的 GraphQL schema 和解析器。然后，我们使用 typedefs 和 resolvers 初始化一个新的 GraphQL 服务器。

我们启动服务器并继续创建 API 处理程序。为此，我们使用 Next.js 集成助手来启动服务器并返回 Next.js 处理程序。集成助手将无服务器 Apollo 实例连接到 Next.js 自定义服务器。在我们将默认导出定义为一个异步函数（该函数以 API 的请求和响应对象作为参数）之前，我们创建了一个包装器，将 CORS 头部添加到请求中。函数中的第一块代码设置了 CORS 头部，并且我们将允许的请求限制为 POST 请求。我们在这里需要 CORS 头部，以使我们的 GraphQL API 对外公开。否则，我们将无法从运行在不同域上的网站连接到 API，甚至无法使用服务器内置的 GraphQL 沙盒。

CORS 设置的一部分是在这里我们立即对任何 OPTIONS 请求返回 *200*。CORS 模式使用 OPTIONS 请求作为预检检查。在这里，浏览器仅请求头部，然后检查响应的 CORS 头部，以验证从哪个域调用 API 是否被允许访问资源，然后再发起实际请求。

然而，我们的 Apollo 服务器只允许 POST 和 GET 请求，并且会对预检的 OPTIONS 请求返回 *405: 方法不允许*。因此，我们不会将此请求传递给 Apollo 服务器，而是结束该请求并返回 *200* 及之前的 CORS 头信息。浏览器应继续按照 CORS 模式处理。最后，我们启动服务器并在所需路径 *api/graphql* 上创建 API 处理程序。

#### 使用 Apollo 沙盒

使用 npm run dev 启动你的 Next.js 服务器。你应该能看到 Next.js 应用程序正在 *http://localhost:3000* 上运行。如果你导航到 GraphQL API 地址 *http://localhost:3000/api/graphql*，你将看到 Apollo 沙盒接口，供查询 API，如 图 6-2 所示。

![](img/Figure6-2.jpg)

图 6-2：Apollo 沙盒的 API 查询界面

在左侧的文档面板中，我们看到可用的查询作为我们之前定义的查询对象的字段。如预期所示，我们在这里看到 Weather 查询，点击它后，新的查询会出现在中间的操作面板中。同时，界面发生变化，我们可以看到可用的参数和字段，点击每个字段可以查看更多信息。使用加号 (+) 按钮，我们可以向查询面板添加字段并将其应用于数据。

尝试创建一个返回 zip 和 weather 属性的 Weather 查询。此查询需要一个邮政编码作为参数；通过左侧的用户界面添加该参数，然后将邮政编码 96826 作为字符串添加到下方面板的 variables 部分中的 JSON 对象。现在点击操作面板顶部的 **Weather** 按钮来运行查询。你应该会在右侧的响应面板中看到该邮政编码的 JSON 结果。将你的屏幕与 图 6-3 进行比较。

![](img/Figure6-3.jpg)

图 6-3：从服务器返回的 GraphQL 查询和响应

尝试创建查询、访问属性，并通过无效的参数创建错误，熟悉一下 GraphQL，然后再进入下一章。

### 总结

本章探讨了 RESTful 和 GraphQL Web API 及其在全栈开发中的作用。虽然在前几章中我们使用了 REST 设计，但你现在应该已经熟悉了无状态服务器的概念，以及在 REST 中执行 CRUD 操作的五种 HTTP 方法。你还练习了使用公共 REST API 读取和更新数据，并评估其请求和响应。

GraphQL API 的实现需要更多的工作，但它们减少了 REST 中常见的过度抓取和抓取不足的问题。你学习了如何通过架构定义 API 契约，并通过解析器实现其功能。然后，你查询了一个 API 并定义了请求中返回的数据集。

最后，你通过将 Apollo 服务器添加到现有的 Next.js 应用程序中，添加了一个 GraphQL API。你现在应该能够创建自己的 GraphQL API 并消费第三方资源。想要了解更多关于 GraphQL 的信息，我推荐[*https://<wbr>www<wbr>.howtographql<wbr>.com*](https://www.howtographql.com)上的教程，以及[*https://<wbr>graphql<wbr>.org<wbr>/learn<wbr>/*](https://graphql.org/learn/)上的官方 GraphQL 介绍。

在下一章中，你将探索 MongoDB 数据库和用于存储数据的对象数据建模库 Mongoose。
