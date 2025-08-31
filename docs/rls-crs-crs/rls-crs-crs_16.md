# 第十四章 Web API

最终，你可能希望将应用程序的范围扩展到网站之外。流行的 Web 应用程序通常还拥有一个本地移动客户端，有时甚至还有桌面客户端。你可能还希望将应用程序中的数据与其他网站和应用程序集成。

一个网页*应用程序编程接口*（或*API*）使这一切成为可能。可以将 API 看作是应用程序之间进行通信的语言。在 Web 上，API 通常是使用 JavaScript 对象表示法（JSON）消息的 REST 协议。

在本章中，我们将探讨 GitHub API，了解如何访问有关用户和仓库的详细信息。在讨论 GitHub 的 API 后，你将构建自己的 API。在此过程中，我将讲解 JSON、超文本传输协议（HTTP）和基于令牌的身份验证等细节。

# GitHub API

GitHub 代码托管服务拥有一个广泛的 API。它的许多功能甚至无需身份验证即可使用。如果你在本章的示例讲解后，想继续探索 GitHub API，完整的文档可以在线查阅，地址为*[`developer.github.com/`](https://developer.github.com/)*。

GitHub API 提供了对有关用户、组织、仓库和其他站点功能数据的便捷访问。例如，访问*[`api.github.com/orgs/rails/`](https://api.github.com/orgs/rails/)*，你可以在浏览器中查看 GitHub 上的 Rails 组织：

```
 {
   "login": "rails",
➊  "id": 4223,
➋  "url": "https://api.github.com/orgs/rails",
   "repos_url": "https://api.github.com/orgs/rails/repos",
   "events_url": "https://api.github.com/orgs/rails/events",
   "members_url": "https://api.github.com/orgs/rails/me...",
   "public_members_url": "https://api.github.com/orgs/r...",
   "avatar_url": "https://avatars.githubusercontent.com...",
   "name": "Ruby on Rails",
   "company": null,
   "blog": "http://weblog.rubyonrails.org/",
   "location": null,
   "email": null,
   "public_repos": 73,
   "public_gists": 3,
   "followers": 2,
   "following": 0,
   "html_url": "https://github.com/rails",
➌  "created_at": "2008-04-02T01:59:25Z",
➍  "updated_at": "2014-04-13T20:24:49Z",
   "type": "Organization"
}
```

返回的数据对于任何与 Rails 模型打过交道的人来说，应该至少部分是熟悉的。你将看到`id` ➊、`created_at` ➌和`updated_at` ➍等字段，这些字段在你迄今为止创建的所有模型中都有。GitHub API 还包括几个`url`字段 ➋，你可以使用它们来访问有关组织的更多数据。

例如，访问`repos_url`（*[`api.github.com/orgs/rails/repos/`](https://api.github.com/orgs/rails/repos/)）查看属于 Rails 组织的源代码仓库列表。从那里，你可以通过访问其`url`来查看单个仓库的详细信息，例如*[`api.github.com/repos/rails/rails/`](https://api.github.com/repos/rails/rails/)*。

访问*[`api.github.com/users/username/`](https://api.github.com/users/username/)*可以获取有关单个用户的信息。要查看我的 GitHub 账户，请在浏览器中访问*[`api.github.com/users/anthonylewis/`](https://api.github.com/users/anthonylewis/)*。

### 注意

*这些请求返回的数据是* JavaScript 对象表示法（JSON）*格式的，它基于 JavaScript 编程语言的一个子集。在 JSON 格式中，大括号之间的数据是一个包含多个命名属性的 JavaScript 对象。每个属性由名称、后跟冒号以及属性值组成。此格式与 Ruby 中的哈希结构非常相似。*

除了你目前为止所做的简单数据请求外，GitHub API 还支持使用适当的请求创建和更新对象。当然，这些操作需要身份验证。但在我讲解 API 身份验证之前，我需要先给你多讲一点 HTTP 的内容。

# HTTP

HTTP 是 Web 的语言。Web 服务器和浏览器使用该协议进行通信。我已经讨论了一些 HTTP 的方面，例如 HTTP 动词（GET、POST、PATCH 和 DELETE），同时在第四章中讲解了 REST 架构。

除了你目前看到的数据外，HTTP 响应还包含一个包含更多详细信息的头部。你可能熟悉 HTTP 响应头部中的部分数据。任何在 Web 上呆过一段时间的人可能都见过来自 Web 服务器的 404 或 500 响应。像这样的状态码会包含在 Web 服务器的每个响应中。

## 状态码

每个响应的第一行都会包含一个 HTTP 状态码。这个三位数字代码告诉客户端应该期待什么类型的响应。

状态码根据其首位数字分为五个类别：

+   1*xx* 信息性

+   2*xx* 成功

+   3*xx* 重定向

+   4*xx* 客户端错误

+   5*xx* 服务器错误

在处理 API 时，你不应该遇到 1*xx* 范围的状态码。原始的 HTTP 1.0 规范并没有定义此范围的任何状态码，在我的经验中，这些状态码很少使用。

2*xx* 范围内的状态码表示请求成功。希望你能遇到许多这样的状态码。常见的代码包括 *200 OK*，表示成功的响应，通常是针对 GET 请求的；*201 Created*，当对象在服务器上被创建以响应 POST 请求时返回；以及 *204 No Content*，表示请求成功，但响应中没有附加数据。

3*xx* 范围的状态码表示重定向到其他地址。每当你在应用程序中使用 `redirect_to` 时，Rails 会返回 *302 Found* 响应。要查看这个过程，登录到你的应用程序并查看重定向日志。

4*xx* 范围的状态码表示某种客户端错误。换句话说，用户犯了错误。*401 Unauthorized* 会在请求需要身份验证的 URL 时返回。*403 Forbidden* 状态码与 401 相似，只是即使客户端成功进行身份验证，服务器也不会完成请求。*404 Not Found* 会在客户端尝试访问一个不存在的 URL 时发送。当你与 API 打交道时，可能会遇到 *406 Not Acceptable* 状态码，这表示请求无效，或者遇到 *422 Unprocessable Entity* 状态码，这表示请求有效，但附带的数据无法处理。

5*xx*状态码范围表示服务器错误。*500 内部服务器错误*代码是最常用的。这是一个通用消息，不提供任何额外的数据。*503 服务不可用*状态码表示服务器的临时问题。

要查看这些代码，您需要检查随响应一起发送的 HTTP 头部。这些通常不会在网页浏览器中显示。幸运的是，有一些工具可以轻松检查 HTTP 头部。其中最受欢迎的工具之一是命令行程序 Curl。

## Curl

Curl 是一个免费的命令行网络通信工具。Curl 包含在 Mac OS X 和 Linux 中，Windows 用户可以从*[`curl.haxx.se/`](http://curl.haxx.se/)*下载该工具。Curl 使用 URL 语法，因此是测试 Web API 的理想工具。

打开终端窗口并尝试一些`curl`命令。我们从刚才查看过的 GitHub API 开始。

```
$ **curl https://api.github.com/users/anthonylewis**
{
  "login": "anthonylewis",
  "id": 301,
  *--snip--*
}
```

这个示例展示了如何从 GitHub 获取特定用户账户的信息。默认情况下，Curl 只显示响应数据；输入**`curl -i`**以将 HTTP 头部包含在响应中：

```
  $ **curl -i https://api.github.com/users/anthonylewis**
➊ HTTP/1.1 200 OK
  Server: GitHub.com
  Date: Thu, 17 Apr 2014 00:36:29 GMT
  Content-Type: application/json; charset=utf-8
  Status: 200 OK
➋ X-RateLimit-Limit: 60
  X-RateLimit-Remaining: 58
➌ X-RateLimit-Reset: 1397696651
  *--snip--*

  {
   "login": "anthonylewis",
   "id": 301,
   *--snip--*
  }
```

响应头部以`200 OK`状态码开始➊。还要注意，GitHub API 请求是有速率限制的。`X-RateLimit-Limit: 60`这一行➋表示您在一定时间内最多可以发送 60 次请求。下一行显示您还剩余 58 次请求。您的速率限制将在`X-RateLimit-Reset: 1397696651`这一行➌指定的时间自动重置。

### 注意

*数字`1397696651`是一个 Unix 时间戳。您可以通过在 IRB 会话或 Rails 控制台中输入`Time.at 1397696651`来将其转换为正常时间。*

# 身份验证

到目前为止，您只从 GitHub API 读取了公共数据。您还可以使用 GitHub API 读取有关用户和仓库的私人数据，以及创建或更新信息，但这些操作需要身份验证。

我在第九章中讲解了用户身份验证。用户期望一次登录后就能在一段时间内浏览网站。您通过会话保持用户的登录状态，会话存储在一个浏览器会自动随每个请求一起发送的 cookie 中。

API 请求不会保持会话状态。访问 API 的应用程序需要在每次请求中提供身份验证凭据。一个常见的 API 请求身份验证方法是*基于令牌的身份验证*。在基于令牌的身份验证中，用户在每次请求中都包含一个唯一的 API 令牌。

您可以使用`curl`命令测试 GitHub 上的基于令牌的身份验证。首先，您需要在 GitHub 的应用设置页面生成一个个人访问令牌。如果需要，请登录 GitHub，并访问 *[`github.com/settings/applications/`](https://github.com/settings/applications/)*。在该页面，点击**生成新令牌**按钮。接下来，您为该令牌提供一个描述；类似于 API 测试的名称就可以。最后，确认“repo”和“user”旁边的复选框已选中，然后点击**生成令牌**按钮。

GitHub 应该将您带回应用设置页面，并展示一个新的 40 位十六进制令牌。复制您的新令牌并粘贴到文本文件中，以便随时查看。正如屏幕上的信息所说，您将无法再次看到它！

为了验证您的令牌是否有效，请在终端中输入以下`curl`命令。将***`token`***替换为您在所有请求中实际使用的令牌：

```
$ **curl -H "Authorization: Token *token*" https://api.github.com/user**
{
  "login": "anthonylewis",
  "id": 301,
  *--snip--*
```

在这里，我使用了`-H`参数传递给`curl`，用于向服务器传递自定义头数据，在这种情况下，数据是`Authorization: Token`头后跟我的令牌。

即使您没有指定用户名，您也应该看到关于您自己账户的信息。GitHub 使用您的个人访问令牌来验证请求。

您现在可以使用该令牌访问私密信息，例如与您的账户关联的 Git 仓库列表。

```
$ **curl -H "Authorization: Token *token*" https://api.github.com/user/repos**

  {
    "id": 6289476,
    "name": "blog",
    "full_name": "anthonylewis/blog",
    "owner": {
      "login": "anthonylewis",
      "id": 301,
      *--snip--*
```

GitHub 应该返回一个包含您账户创建的仓库的数组。根据您创建的仓库数量，这可能会是大量数据。

既然您已经有了令牌，您还可以通过 POST 请求将另一个仓库添加到您的账户中。正如您在[第四章中所学到的，POST 在 REST 中意味着*创建*。

```
➊ $ **curl -i -d '{"name":"API Test"}' \**
         **-H "Authorization: Token *token*" \**
         **https://api.github.com/user/repos**
➋ HTTP/1.1 201 Created
  Server: GitHub.com
  Date: Mon, 21 Apr 2014 23:47:59 GMT
  Content-Type: application/json; charset=utf-8
  Status: 201 Created
  *--snip---*
➌ {
    "id": 18862420,
    "name": "API-Test",
    "full_name": "anthonylewis/API-Test",
    "owner": {
      "login": "anthonylewis",
    "id": 301,
  *--snip--*
```

`-d`选项用于`curl`指定请求中包含的数据。在这里，您发送一个 JSON 字符串，其中包含名称为`"API Test"`的新仓库 ➊。因为您正在发送数据，`curl`会自动使用 POST 请求。GitHub 会对请求作出响应，返回指示 HTTP 状态`201 Created`的头信息 ➋，随后返回关于新创建仓库的信息 ➌。

现在您对现有的 API 有了一些经验，我们来为我们的社交应用创建一个自己的 API。

# 您的个人 API

您可能还记得在第四章中，Rails 的脚手架生成器在`PostsController`中使用了`respond_to`方法，根据请求类型返回不同的数据。这种方法对于某些应用程序是可以的，但当您在应用中加入用户身份验证和会话管理时，就会出现问题。

现有的控制器通过在每个动作之前调用`authenticate_user!`方法来进行用户身份验证。您的 API 将使用不同的方法来支持基于令牌的身份验证。现有的控制器还根据`current_user`的值展示数据，如帖子。您的 API 将在请求时展示所有帖子。

与其使用相同的控制器来处理应用程序和 API，你可以为每个构建单独的控制器。由于你的应用程序主要是关于帖子的，所以你可以从构建 API 的帖子控制器开始。

## API 路由

首先添加 API 请求的路由。GitHub API 使用子域来处理 API 请求。由于你尚未设置自己的域名，因此你将使用一个单独的路径来处理 API 请求。打开文件*config/routes.rb*，并在文件末尾添加以下代码块：

```
  Social::Application.routes.draw do
  *--snip--*

➊   **namespace :api do**
      **resources :posts**
    **end**
  end
```

`namespace :api`块 ➊ 表示为其包含的资源创建的所有路由路径都以*api/*开头。此外，这些资源的控制器文件应该位于一个名为*api*的目录中，并且控制器类应该在一个名为`Api`的模块内。

你可以在终端中输入`bin/rake routes`命令来查看新创建的路由。

## API 控制器

现在你已经定义了路由，接下来需要创建一个控制器来处理这些动作。首先，创建一个目录来存放 API 控制器，方法是输入以下命令：

```
$ **mkdir app/controllers/api**
```

然后创建一个名为*app/controllers/api/posts_controller.rb*的新文件，并添加 API `PostsController`的代码，如下所示：

```
  **module Api**
    **class PostsController < ApplicationController**
➊   **respond_to :json**

➋   **def index**
      **@posts = Post.all**
➌     **respond_with @posts**
    **end**
  **end**
**end**
```

该文件以`module Api`开始，表示该类属于 API 命名空间。在`PostsController`类内部，有一个对`respond_to`类方法的调用。调用`respond_to :json`表示该控制器中的动作返回 JSON 数据 ➊。

该类接着定义了`index`动作 ➋。`index`动作检索所有帖子，然后使用`respond_with`方法将其发送到客户端 ➌。`respond_with`方法会根据请求中使用的格式和 HTTP 动词自动格式化数据。在这种情况下，它应该在响应 GET 请求时返回 JSON 数据，用于`index`动作。

保存该文件后，如果 Rails 服务器尚未启动，请启动它。然后，你可以使用`curl`命令通过输入以下命令来测试你的 API：

```
$ **curl http://localhost:3000/api/posts**
[{"id":1,"title":"First Post","body":"Hello, World!"...
```

API 将返回一个帖子的数组，以响应帖子`index`动作。

数据是紧凑的并且在一行中，这可能难以阅读，但有几个免费的工具可以帮你格式化 JSON 数据。例如，jq 是一个 JSON 处理器，可以格式化 JSON 数据并添加语法高亮。你可以从*[`stedolan.github.io/jq/`](http://stedolan.github.io/jq/)*下载 jq。安装后，你可以通过在命令的末尾添加`| jq '.'`将输出通过 jq 的基本过滤器来格式化：

```
$ **curl http://localhost:3000/api/posts | jq '.'**
[
  {
   "id": 1,
   "title": "First Post",
   "body": "Hello, World!",
   "url":null,
   "user_id":1,
   *--snip--*
```

本章剩余的示例采用了漂亮打印格式。为了简洁起见，我省略了 `| jq '.'`，但如果你想让输出与书中所见相同，应该包括它。你也可以在浏览器中查看 JSON 输出。在浏览器中输入 *http://localhost:3000/api/posts* 会引发 `ActionController::UnknownFormat` 错误。如果你查看终端中的服务器输出，会看到这是一个 *406 Not Acceptable* 错误，正如本章之前讨论的那样。发生此错误是因为控制器仅响应 JSON 请求，而浏览器默认请求 HTML。

通过在地址栏的 URL 中添加扩展名来指定不同的内容类型。浏览 *http://localhost:3000/api/posts.json* 会按预期返回一个 JSON 数组。

## 自定义 JSON 输出

到目前为止，你的 API 返回了与每个帖子相关的所有数据。你可能希望为每条记录包含额外的数据，某些情况下，可能希望排除某些字段的数据。例如，包含每个帖子的作者数据是有帮助的，但你不想包含用户的 `password_digest` 或 `api_token`。

你可以通过几种方式自定义内置于 Rails 中的 API 输出。你使用哪种方法取决于你需要多少自定义和个人偏好。

### as_json

因为这个 API 返回的是 JSON 数据，所以你可以通过更改 Rails 将模型转换为 JSON 的方式来轻松自定义输出。Rails 首先调用模型上的 `as_json` 方法将其转换为哈希，然后再将哈希转换为 JSON 字符串。

你可以在 `Post` 模型中重写 `as_json` 方法，以自定义每个帖子的返回数据。打开文件 *app/models/post.rb* 并添加如下所示的 `as_json` 方法，强制该方法仅显示每个帖子的 `id` 和 `title`：

```
  class Post < ActiveRecord::Base
    *--snip--*

➊   **def as_json(options={})**
➋     **super(only: [:id, :title])**
    **end**

    *--snip--*
  end
```

确保包括默认值为 `{}` 的 `options` 参数 ➊，因为原始的 `as_json` 包括了它。你虽然没有使用 `options` 参数，但因为你正在重写现有方法，所以你的定义必须与原始方法匹配。你的 `as_json` 方法调用 `super`，这将调用 Active Record 中定义的原始 `as_json` 方法，并传递参数 `only: [:id, :title]` ➋。

使用此方法后，你的 API 应该仅返回每个帖子的 `id` 和 `title`。使用 `curl` 命令验证此更改：

```
$ **curl http://localhost:3000/api/posts**
[
  {"id": 1, "title": "First Post"},
  {"id": 2, "title": "Google Search"}
]
```

`as_json` 方法支持若干附加选项。你可以使用 `:except` 选项来排除字段，而不是像 `:only` 一样指定包含的字段。你还可以使用 `:include` 选项来包含关联的模型。例如，更新 `as_json` 方法，如下所示，排除 `user_id` 字段并包含帖子关联的 `user` 模型：

```
def as_json(options={})
  **super(except: [:user_id], include: :user)**
end
```

`:methods` 选项调用方法列表，并将其返回值包括在输出中。例如，你可以使用此选项调用你在第十二章中添加的 `cached_comment_count` 方法：

```
def as_json(options={})
  **super(except: [:user_id], include: :user,**
    **methods: :cached_comment_count)**
end
```

这个选项将包括与该帖子关联的评论数（缓存）的信息。

重写`as_json`当然有效，但根据定制需求的不同，这可能会变得有些杂乱。幸运的是，Rails 提供了一种完全定制 API 返回 JSON 数据的方式。删除`Post`模型中的`as_json`方法，让我们来学习 jbuilder。

### Jbuilder

Jbuilder 是一个专门用于生成 JSON 输出的领域特定语言。`jbuilder` gem 默认包含在`rails new`命令生成的*Gemfile*中。使用 jbuilder，你可以为每个 API 操作创建视图，就像使用 ERB 为 Web 操作创建视图一样。

和其他视图一样，你需要为 jbuilder 视图创建一个目录。视图目录必须与控制器名称匹配。输入以下命令来为 API 视图创建一个目录，并为`PostsController`视图创建子目录：

```
$ **mkdir app/views/api**
$ **mkdir app/views/api/posts**
```

在这些目录设置好后，你可以创建第一个 jbuilder 视图。创建一个新文件，命名为*app/views/api/posts/index.json.jbuilder*，并在编辑器中打开它。添加这一行代码并保存文件：

```
**json.array! @posts**
```

`json.array!`方法告诉 jbuilder 将`@posts`的值呈现为 JSON 数组。使用 Curl 检查索引操作的输出：

```
$ **curl http://localhost:3000/api/posts**

  {
    "id": 1,
    "title": "First Post",
    "body": "Hello, World!",
    "url":null,
    "user_id":1,
    *--snip--*
```

输出与开始时相同。现在让我们看看如何定制这个输出。

`json.array!`方法也接受一个块。在块内，你可以访问数组中的每个记录。然后，你可以使用`json.extract!`方法仅包含帖子中的特定字段：

```
**json.array! @posts do |post|**
  **json.extract! post, :id, :title, :body, :url**
**end**
```

这个示例将每篇文章的`id`、`title`、`body`和`url`字段呈现为 JSON 格式。

所有常见的视图辅助方法在 jbuilder 视图中也可以使用。例如，你可以使用`api_post_url`辅助方法为每篇文章包含一个 URL：

```
  json.array! @posts do |post|
    json.extract! post, :id, :title, :body, :url
➊   **json.post_url api_post_url(post)**
end
```

方法调用的输出，如`api_post_url(post)` ➊，会自动转换为 JSON 格式。下一个示例添加了每篇文章的作者数据：

```
json.array! @posts do |post|
  json.extract! post, :id, :title, :body, :url
  json.post_url api_post_url(post)

  **json.user do**
    **json.extract! post.user, :id, :name, :email**
  **end**
end
```

在这里，我再次使用了`json.extract!`方法，只包含每个用户的特定字段。你不希望公开 API 中暴露`password_digest`字段。

## 基于令牌的认证

现在，让我们添加认证，以便你也可以通过 API 创建帖子。你将添加基于令牌的认证，就像你之前访问 GitHub API 时使用的那样。

### 生成令牌

首先，通过生成数据库迁移，为`User`模型添加一个`api_token`字符串字段：

```
$ **bin/rails g migration add_api_token_to_users api_token:string**
```

记得在生成此迁移后，输入`bin/rake db:migrate`命令来更新数据库。

现在通过在编辑器中打开*app/models/user.rb*文件，更新`User`模型，添加对`api_token`字段的验证，并添加`before_validation`回调来生成 API 令牌：

```
   class User < ActiveRecord::Base
     *--snip--*

➊    **validates :api_token, presence: true, uniqueness: true**

➋    **before_validation :generate_api_token**

     *--snip--*
```

首先，你需要验证`api_token`是否存在且唯一 ➊。因为你将使用这个值进行认证，所以两个用户不能拥有相同的`api_token`。

接下来，使用`before_validation`回调调用一个方法，如果`api_token`不存在，则生成它➋。在`User`模型的底部添加`generate_api_token`方法，如下所示：

```
  class User < ActiveRecord::Base

    *--snip--*

    **def generate_api_token**
➊     **return if api_token.present?**

      **loop do**
➋       **self.api_token = SecureRandom.hex**
➌       **break unless User.exists? api_token: api_token**
      **end**
    **end**

  end
```

如果`api_token`已经有值，`generate_api_token`方法会立即返回➊。如果`api_token`没有值，方法会在一个无尽的`loop`中调用`SecureRandom.hex`生成一个值➋。`SecureRandom`类使用计算机上最安全的随机数生成器来生成值。在 Unix 计算机上，它使用`/dev/urandom`设备；在 Windows 上，它使用 Win32 加密 API。`SecureRandom`类还包括几种格式化随机值的方法。`hex`方法返回一个随机的 32 字符十六进制值。最后，如果没有用户拥有该`api_token`，则跳出循环➌。

现在打开 Rails 控制台并更新现有用户：

```
➊ irb(main):001:0> **user = User.first**
    User Load (0.2ms) SELECT "users".* ...
  => #<User id: 1, ... api_token: nil>
➋ irb(main):002:0> **user.save**
     (0.1ms) begin transaction
    User Exists (0.2ms) SELECT 1 AS one FROM ...
    User Exists (0.1ms) SELECT 1 AS one FROM ...
    User Exists (0.1ms) SELECT 1 AS one FROM ...
    SQL (1.3ms) UPDATE "users" SET "api_token" ...
     (1.7ms)  commit transaction
  => true
```

由于`generate_api_token`方法是通过`before_validation`回调自动调用的，您只需要将用户加载到变量中➊，然后将其保存到数据库中➋进行更新。对每个用户执行此操作。如果有任何用户没有`api_token`值，它将被创建。

现在更新用户的`show`视图，以便在用户查看自己的账户时显示`api_token`。按照下面所示更新*app/views/users/show.html.erb*：

```
  <div class="page-header">
    <h1>User</h1>
  </div>

  <p class="lead"><%= @user.email %></p>

➊ **<% if @user == current_user %>**
    **<p class="lead">API Token: <%= @user.api_token %></p>**
  **<% end %>**

*--snip--*
```

由于 API 令牌本质上是密码，您需要通过仅在显示的用户等于`current_user`时才显示它们，从而保护它们➊。

### 身份验证请求

现在所有用户都有了 API 令牌，让我们开始使用这些令牌。使用令牌进行身份验证的过程类似于您已经创建的用户名和密码身份验证。因为您的 API 可能有多个控制器，您应该将身份验证方法包含在`ApplicationController`中，它是所有其他控制器的父类。

首先，您需要一个方法来使用`api_token`进行身份验证。幸运的是，Rails 提供了一个名为`authenticate_or_request_with_http_token`的内建方法，可以处理这些细节。打开文件*app/controllers/application_controller.rb*，并添加以下方法来查看它是如何工作的：

```
  class ApplicationController < ActionController::Base
    # Prevent CSRF attacks by raising an exception.
    # For APIs, you may want to use :null_session instead.
    protect_from_forgery with: :exception

    private

    **def authenticate_token!**
      **authenticate_or_request_with_http_token do |token, options|**
➊       **@api_user = User.find_by(api_token: token)**
      **end**
    **end**

    *--snip--*
```

该方法命名为`authenticate_token!`，与您在[第九章中添加的`authenticate_user!`方法匹配。`authenticate_or_request_with_http_token`从请求的 Authorization 头部获取令牌，并将其传递给一个代码块。在代码块中，您尝试使用给定的令牌在数据库中查找用户➊。`find_by`方法如果找到匹配的用户，则返回一个`User`对象，否则返回`nil`。此值将赋给`@api_user`实例变量，并从代码块中返回。如果代码块返回一个假值，如`nil`，则方法知道身份验证失败，并向客户端发送*401 未授权*响应。

你为访问经过身份验证的用户写了一个辅助方法 `current_user`，该方法出现在 第九章 中。对于 API 请求，经过身份验证的用户已经分配给 `@api_user` 实例变量，因此你可以使用这个变量。

你的基于令牌的身份验证解决方案已经准备好了。让我们尝试通过 API 添加创建文本帖子的功能。

### 使用基于令牌的身份验证

首先，你需要为文本帖子添加路由，因此打开 *config/routes.rb* 并在 `:api` 命名空间中添加 `text_posts` 资源：

```
Social::Application.routes.draw do
*--snip--*

  namespace :api do u
    resources :posts
    **resources :text_posts**
  end
end
```

现在你需要为文本帖子创建一个控制器。记住，它需要位于 *api/* 目录中，因为路由位于 `:api` 命名空间中。创建一个名为 *app/controllers/api/text_posts_controller.rb* 的文件，并添加以下代码：

```
  **module Api**
    **class TextPostsController < ApplicationController**
      **respond_to :json**
➊     **before_action :authenticate_token!**

    **end**
  **end**
```

这个控制器的起始方式与 API 帖子控制器相同。`TextPostsController` 类必须位于名为 `Api` 的模块内。它还包括 `respond_to :json`。第一个变化是添加了 `before_action :authenticate_token!` ➊。控制器在每个操作之前都会调用 `authenticate_token!` 方法。

你想创建文本帖子，因此添加 `create` 方法：

```
  module Api
    class TextPostsController < ApplicationController
      respond_to :json
      before_action :authenticate_token!
➊   **def create**
      **@text_post = @api_user.text_posts.create(text_post_params)**
      **respond_with @text_post**
    **end**
  end
end
```

`create` 方法使用在 `authenticate_token!` 中设置的 `@api_user` 实例变量来创建一个新的文本帖子 ➊。然后你使用 `respond_with` 将新的文本帖子发送回客户端。请注意，你没有检查文本帖子是否真正创建。`respond_with` 方法会自动在 `@text_post` 包含错误时发送适当的错误响应。

因为你还想指定允许的参数值，所以你的最终添加是一个 `text_post_params` 方法：

```
  module Api
    class TextPostsController < ApplicationController
      before_action :authenticate_token!

      respond_to :json

      def create
        @text_post = @api_user.text_posts.build(text_post_params)
        respond_with @text_post
      end

     **private**

➊    **def text_post_params**
       **params.require(:text_post).permit(:title, :body)**
     **end**
  end
end
```

`text_post_params` 方法允许在一个嵌套的哈希中使用 `:title` 和 `:body` 数据，哈希的键是 `:text_post` ➊。这与处理 Web 请求时控制器中的 `text_post_params` 方法相同。

输入 `curl` 命令以尝试新的 API。运行命令时，确保将 `Content-Type` 头设置为 `application/json`，这样 Rails 就会自动解析请求中包含的 JSON 数据。将 ***`token`*** 替换为你应用程序某个用户的实际 `api_token`。

```
  **$ curl -i \**
       **-d '{"text_post":{"title":"Test","body":"Hello"}}' \**
       **-H "Content-Type: application/json" \**
       **-H "Authorization: Token *token*" \**
       **http://localhost:3000/api/text_posts**
1 HTTP/1.1 422 Unprocessable Entity
*--snip--*
```

出现了问题：状态码 *422 Unprocessable Entity* ➊ 表示客户端传递给服务器的数据无效。请检查终端中的服务器输出以获取更多信息。

```
  Started POST "/api/text_posts" for 127.0.0.1 at 2014-04-23 19:39:09 -0500
  Processing by Api::TextPostsController#create as */*
    Parameters: {"text_post"=>{"title"=>"Test", "body"=>"Hello"}}
➊ Can't verify CSRF token authenticity
  Completed 422 Unprocessable Entity in 1ms

--*snip*--
```

传递给服务器的数据有效，但未包含 CSRF 令牌 ➊。请记住，这个令牌与 API 令牌不同。CSRF 令牌是另一个唯一的令牌，当你在应用程序中提交表单数据时，它会自动发送。因为你没有提交表单，所以无法知道正确的 CSRF 令牌。

当你之前更新`ApplicationController`时，可能注意到了类顶部的一条有用的注释。Rails 通常通过引发异常来防止 CSRF 攻击。这对于 Web 应用程序来说很有用，但对 API 无效。你可以通过清除用户的会话数据来防止 CSRF 攻击，而不是引发异常。现在，每当应用程序收到一个不包含 CSRF 令牌的数据时，它会清除用户的会话，从而有效地将用户从应用程序中登出并防止攻击。

幸运的是，API 客户端在每次请求时都包括正确的 API 令牌，而不是将认证数据存储在会话中。因此，API 请求在空会话下应该可以正常工作。打开*app/controllers/application_controller.rb*文件并进行以下更新： 

```
  class ApplicationController < ActionController::Base
    # Prevent CSRF attacks by raising an exception.
    # For APIs, you may want to use :null_session instead.
➊   **protect_from_forgery with: :null_session**

    *--snip--*
```

在`protect_from_forgery`方法调用 ➊ 中，将`:with`选项的值更改为`:null_session`，然后使用`curl`再次尝试相同的请求：

```
  **$ curl -i \**
         **-d '{"text_post":{"title":"Test","body":"Hello"}}' \**
         **-H "Content-Type: application/json" \**
         **-H "Authorization: Token *token*" \**
         **http://localhost:3000/api/text_posts**
➊ HTTP/1.1 201 Created
  *--snip--*
➋ {
    "id":5,
    "title":"Test",
    "body":"Hello",
    "url":null,
    "user_id":1,
    "created_at":"2014-04-24T00:33:35.874Z",
    "updated_at":"2014-04-24T00:33:35.874Z"
  }
```

状态码现在是*201 Created*，表示成功 ➊。HTTP 头部后面是新文本帖子的 JSON 表示 ➋。因为你没有为这个动作创建 jbuilder 视图，所以使用了默认的 JSON 表示。

你也可以在浏览器中打开`posts`索引页面，或者使用命令`curl http://localhost:3000/api/posts`发出请求来验证文本帖子是否成功创建。

# 总结

Web API 可以让你的应用程序与客户和第三方应用程序进行协作。通过有效的 API，你还可以为你的应用程序构建本地移动或桌面客户端。你甚至可以使用另一个应用程序的 API 将其数据集成到你的应用程序中。

在本章中，我们讨论了 GitHub API，并使用它访问有关用户和仓库的详细数据。在介绍了超文本传输协议和基于令牌的认证后，你为你的社交网络应用程序构建了自己的 API。

在下一章，你将学习如何设置自己的服务器来托管 Rails 应用程序，并使用 Capistrano 远程服务器自动化工具来部署和维护你的应用程序。 

# 练习

| 问： | 1\. 通过发送一个带有假令牌的 POST 请求，验证你的基于令牌的认证是否真的有效。使用`curl`命令发送请求，并确保检查头部中的状态码和响应体。 |
| --- | --- |
| 问： | 2\. 尝试使用无效数据创建文本帖子，看看会发生什么。你可以在*app/models/text_post.rb*中检查文本帖子的验证。再次使用`curl`命令发送请求，并确保检查头部和响应体中的状态码。 |
| 问： | 3\. 通过在帖子控制器中添加一个`show`动作来扩展 API。这个动作应该使用`params[:id]`查找正确的帖子，然后使用`respond_with`方法将帖子返回给客户端。因为这是一个 GET 请求，你可以使用`curl`或在你的浏览器中检查它。 |
