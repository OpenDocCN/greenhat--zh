# 第二十二章：项目：技能分享网站

`技能分享`会议是一个人们聚集在一起，围绕共同兴趣进行小型非正式演讲的活动。在一次园艺技能分享会议上，有人可能会讲解如何种植芹菜。或者在一次编程技能分享小组中，你可以随意来告诉大家关于`Node.js`的信息。

在本项目的最后一章中，我们的目标是建立一个用于管理技能分享会议上讲座的网站。想象一下，一小群人定期在其中一位成员的办公室聚会，讨论独轮车。之前的会议组织者已搬到另一个城市，没有人主动承担这一任务。我们希望有一个系统，让参与者在没有积极组织者的情况下，自主提议和讨论讲座。

项目的完整代码可以从[`eloquentjavascript.net/code/skillsharing.zip`](https://eloquentjavascript.net/code/skillsharing.zip)下载。

### 设计

该项目包含一个为`Node.js`编写的`服务器`部分和一个为浏览器编写的`客户端`部分。服务器存储系统的数据并将其提供给客户端。同时，服务器还提供实现客户端系统的文件。

服务器保持下次会议提议的讲座列表，客户端显示此列表。每个讲座都有一个演讲者姓名、标题、摘要以及与之相关的评论数组。客户端允许用户提议新讲座（将其添加到列表中）、删除讲座并对现有讲座进行评论。每当用户进行这样的更改时，客户端会发送`HTTP`请求以告知服务器。

![图片](img/f0356-01.jpg)

该应用程序将设置为显示当前提议的讲座及其评论的`实时`视图。每当有人在任何地方提交新讲座或添加评论时，所有在浏览器中打开该页面的人应立即看到变化。这带来了一些挑战——因为没有办法让网络服务器打开与客户端的连接，也没有好的方法来知道当前哪些客户端正在查看特定网站。

解决此问题的一个常见方法称为`长轮询`，这恰好是`Node`设计的动机之一。

### 长轮询

为了能够立即通知客户某些内容发生了变化，我们需要与该客户建立连接。由于网络浏览器通常不接受连接，并且客户常常处于会阻止此类连接的路由器后面，因此让服务器发起此连接并不实用。

我们可以安排客户端打开连接并保持连接，以便服务器在需要时可以使用它发送信息。但`HTTP`请求仅允许简单的信息流：客户端发送请求，服务器返回单个响应，便结束了。一个名为`WebSockets`的技术使得可以为任意数据交换打开连接，但正确使用这些套接字有些棘手。

在本章中，我们使用一种更简单的技术，即`长轮询`，客户端通过常规的`HTTP`请求持续向服务器请求新信息，而服务器在没有新信息时会延迟响应。

只要客户端确保始终保持一个轮询请求开放，它将能够在信息可用后快速接收来自服务器的信息。例如，如果`Fatma`在浏览器中打开了我们的技能共享应用程序，那么该浏览器将已经发出更新请求，并等待对此请求的响应。当`Iman`提交关于极限单轮车的讨论时，服务器将注意到`Fatma`在等待更新，并将包含新讨论的响应发送给她的挂起请求。`Fatma`的浏览器将接收到数据并更新屏幕以显示讨论内容。

为了防止连接超时（因缺乏活动而被中止），长轮询技术通常会为每个请求设置最大时间，超过该时间后，即使服务器没有任何报告，仍会做出响应。然后客户端可以启动新的请求。定期重新启动请求也使得这种技术更加稳健，使客户端能够从临时的连接故障或服务器问题中恢复。

一个繁忙的服务器如果使用长轮询，可能会有数千个等待请求，因此会保持许多`TCP`连接。`Node.js`可以轻松管理多个连接，而无需为每个连接创建单独的控制线程，这使其非常适合这种系统。

### `HTTP`接口

在我们开始设计服务器或客户端之前，让我们先思考它们交互的点：用于通信的`HTTP`接口。

我们将使用`JSON`作为请求和响应主体的格式。就像在第二十章的文件服务器中，我们将尽量充分利用`HTTP`方法和头部。接口围绕`/talks`路径展开。以`/talks`开头的路径将用于提供静态文件——客户端系统的`HTML`和`JavaScript`代码。

对`/talks`的`GET`请求将返回如下`JSON`文档：

```js
[{"title": "Unituning",
  "presenter": "Jamal",
  "summary": "Modifying your cycle for extra style",
  "comments": []}]
```

创建新讨论可以通过向类似`/talks/Unituning`的`URL`发起`PUT`请求来实现，其中第二个斜杠后的部分是讨论的标题。`PUT`请求的主体应包含一个具有`presenter`和`summary`属性的`JSON`对象。

由于讨论标题可能包含空格和其他在`URL`中通常不会出现的字符，因此在构建此类`URL`时，标题字符串必须使用`encodeURIComponent`函数进行编码。

```js
console.log("/talks/" + encodeURIComponent("How to Idle"));
// → /talks/How%20to%20Idle
```

创建关于闲置的讲座的请求可能看起来像这样：

```js
PUT /talks/How%20to%20Idle HTTP/1.1
Content-Type: application/json
Content-Length: 92

{"presenter": "Maureen",
 "summary": "Standing still on a unicycle"}
```

此类`URL`也支持`GET`请求以检索讲座的`JSON`表示和`DELETE`请求以删除讲座。

向讲座添加评论是通过向类似`/talks/Unituning/comments`的URL发送`POST`请求来完成的，`JSON`正文中包含`author`和`message`属性。

```js
POST /talks/Unituning/comments HTTP/1.1
Content-Type: application/json
Content-Length: 72

{"author": "Iman",
 "message": "Will you talk about raising a cycle?"}
```

为了支持长轮询，对`/talks`的`GET`请求可以包含额外的头部，告知服务器在没有新信息可用时延迟响应。我们将使用一对通常用于管理缓存的头部：`ETag`和`If-None-Match`。

服务器可能在响应中包含`ETag`（“实体标签”）头部。其值是一个字符串，用于标识资源的当前版本。当客户端稍后再次请求该资源时，可以通过包含`If-None-Match`头部并将其值设置为该字符串来进行`条件请求`。如果资源未发生变化，服务器将以状态码`304`响应，这意味着“未修改”，告知客户端其缓存版本仍然是最新的。当标签不匹配时，服务器则正常响应。

我们需要类似这样的功能，客户端可以告知服务器它拥有的讲座列表的版本，服务器仅在该列表发生变化时作出响应。但服务器不应立即返回`304`响应，而是应延迟响应，仅在有新信息可用或经过一定时间后才返回。为了将长轮询请求与普通条件请求区分开，我们为它们提供了另一个头部，`Prefer: wait=90`，这告诉服务器客户端愿意等待最长`90`秒的时间以获取响应。

服务器将保持一个版本号，每次讲座发生变化时更新该版本号，并将其用作`ETag`值。客户端可以发出这样的请求，以便在讲座发生变化时得到通知：

```js
GET /talks HTTP/1.1
If-None-Match: "4"
Prefer: wait=90

(time passes)

HTTP/1.1 200 OK
Content-Type: application/json
ETag: "5"
Content-Length: 295

--snip--
```

此处描述的协议不进行任何访问控制。每个人都可以评论、修改讲座，甚至删除它们。（由于互联网上充满了流氓，将这样的系统在线放置而没有进一步的保护可能不会有好的结果。）

### `服务器`

让我们开始构建程序的服务器端部分。本节中的代码在`Node.js`上运行。

#### `路由`

我们的服务器将使用`Node`的`createServer`来启动一个`HTTP`服务器。在处理新请求的函数中，我们必须区分我们支持的各种请求类型（由方法和路径决定）。这可以通过一长串`if`语句来完成，但还有更好的方法。

`路由器`是一个组件，帮助将请求分发到可以处理它的函数。你可以告诉路由器，例如，`PUT`请求的路径匹配正则表达式`/^\/talks\/([^\/]+)$/`（`talks/`后跟讲座标题）可以由特定函数处理。此外，它还可以帮助提取路径中的有意义部分（在此情况下为讲座标题），这些部分用正则表达式中的括号包裹，并将它们传递给处理函数。

`NPM`上有许多优秀的路由器包，但在这里我们将自己编写一个以说明原理。

这是`router.mjs`，我们稍后将从服务器模块中导入它：

```js
export class Router {
  constructor() {
    this.routes = [];
  }
 add(method, url, handler) {
    this.routes.push({method, url, handler});
  }
  async resolve(request, context) {
    let {pathname} = new URL(request.url, "http://d");
    for (let {method, url, handler} of this.routes) {
      let match = url.exec(pathname);
      if (!match || request.method != method) continue;
      let parts = match.slice(1).map(decodeURIComponent);
      return handler(context, ...parts, request);
    }
  }
}
```

该模块导出了`Router`类。路由器对象允许你使用其`add`方法注册特定方法和`URL`模式的处理程序。当使用`resolve`方法解析请求时，路由器会调用与请求的方法和`URL`匹配的处理程序，并返回其结果。

处理函数在给定的上下文值下调用`resolve`。我们将利用这一点使它们能够访问我们的服务器状态。此外，它们接收其正则表达式中定义的任何组的匹配字符串，以及请求对象。这些字符串必须进行`URL`解码，因为原始`URL`可能包含`%20`样式的编码。

#### `服务文件`

当请求与我们路由器中定义的请求类型都不匹配时，服务器必须将其解释为对`public`目录中某个文件的请求。可以使用第二十章中定义的文件服务器来提供此类文件，但我们既不需要也不想在文件上支持`PUT`和`DELETE`请求，并且我们希望具备支持缓存等高级功能。让我们使用`NPM`中一个稳健且经过充分测试的静态文件服务器。

我选择了`serve-static`。这不是`NPM`上唯一的此类服务器，但它工作良好，符合我们的目的。`serve-static`包导出一个可以用根目录调用的函数，以生成请求处理函数。处理函数接受服务器从`node:http`提供的请求和响应参数，以及第三个参数，如果没有文件与请求匹配，它将调用的一个函数。我们希望我们的服务器首先检查应该特别处理的请求，如路由器中定义的那样，因此我们将其包装在另一个函数中。

```js
import {createServer} from "node:http";
import serveStatic from "serve-static";

function notFound(request, response) {
  response.writeHead(404, "Not found");
  response.end("<h1>Not found</h1>");
}
class SkillShareServer {
  constructor(talks) {
    this.talks = talks;
    this.version = 0;
    this.waiting = [];

    let fileServer = serveStatic("./public");
    this.server = createServer((request, response) => {
      serveFromRouter(this, request, response, () => {
        fileServer(request, response,
                   () => notFound(request, response));
      });
    });
  }
  start(port) {
    this.server.listen(port);
  }
  stop() {
    this.server.close();
  }
}
```

`serveFromRouter`函数具有与`fileServer`相同的接口，接受`(request, response, next)`参数。我们可以利用这一点“链接”多个请求处理程序，允许每个处理程序处理请求或将责任传递给下一个处理程序。最终处理程序`notFound`仅仅响应一个“未找到”错误。

我们的`serveFromRouter`函数使用与前一章文件服务器类似的约定来处理响应——路由器中的处理程序返回的承诺解析为描述响应的对象。

```js
import {Router} from "./router.mjs";

const router = new Router();
const defaultHeaders = {"Content-Type": "text/plain"};

async function serveFromRouter(server, request,
                               response, next) {
  let resolved = await router.resolve(request, server)
    .catch(error => {
      if (error.status != null) return error;
      return {body: String(err), status: 500};
    });
  if (!resolved) return next();
  let {body, status = 200, headers = defaultHeaders} =
    await resolved; 
 response.writeHead(status, headers);
  response.end(body);
}
```

#### `讲座作为资源`

已提议的演讲存储在服务器的`talks`属性中，这是一个对象，其属性名称为演讲标题。我们将为我们的路由器添加一些处理程序，将其作为 HTTP 资源公开，路径为`/talks/<title>`。

处理`GET`单个演讲请求的处理程序必须查找该演讲，并以该演讲的 JSON 数据或 404 错误响应进行回应。

```js
const talkPath = /^\/talks\/([^\/]+)$/;

router.add("GET", talkPath, async (server, title) => {
  if (Object.hasOwn(server.talks, title)) {
    return {body: JSON.stringify(server.talks[title]),
            headers: {"Content-Type": "application/json"}};
  } else {
    return {status: 404, body: `No talk '${title}' found`};
  }
});
```

删除演讲是通过将其从`talks`对象中移除来完成的。

```js
router.add("DELETE", talkPath, async (server, title) => {
  if (Object.hasOwn(server.talks, title)) {
    delete server.talks[title];
    server.updated();
  }
  return {status: 204};
});
```

`updated`方法（稍后我们将定义）会通知等待的长轮询请求有关更改的信息。

需要读取请求主体的一个处理程序是`PUT`处理程序，它用于创建新的演讲。它必须检查提供的数据是否具有字符串类型的`presenter`和`summary`属性。来自系统外部的任何数据可能都是无意义的，我们不想损坏内部数据模型或在出现错误请求时崩溃。

如果数据看起来有效，处理程序将一个表示新演讲的对象存储在`talks`对象中，可能会覆盖具有相同标题的现有演讲，并再次调用`updated`。

为了从请求流中读取主体，我们将使用来自`node:stream/consumers`的`json`函数，该函数收集流中的数据并将其解析为 JSON。这个包中还有类似的导出，称为`text`（用于将内容读取为字符串）和`buffer`（用于将其读取为二进制数据）。由于`json`是一个非常通用的名称，因此我们将其导入重命名为`readJSON`，以避免混淆。

```js
import {json as readJSON} from "node:stream/consumers";

router.add("PUT", talkPath,
           async (server, title, request) => {
  let talk = await readJSON(request);
  if (!talk ||
      typeof talk.presenter != "string" ||
      typeof talk.summary != "string") {
    return {status: 400, body: "Bad talk data"};
  }
  server.talks[title] = {
    title,
    presenter: talk.presenter,
    summary: talk.summary,
    comments: []
  };
  server.updated();
  return {status: 204};
});
```

向演讲添加评论的过程类似。我们使用`readJSON`获取请求的内容，验证结果数据，并在数据有效时将其存储为评论。

```js
router.add("POST", /^\/talks\/([^\/]+)\/comments$/,
           async (server, title, request) => {
  let comment = await readJSON(request);
  if (!comment ||
      typeof comment.author != "string" ||
      typeof comment.message != "string") {
    return {status: 400, body: "Bad comment data"};
  } else if (Object.hasOwn(server.talks, title)) {
    server.talks[title].comments.push(comment);
    server.updated();
    return {status: 204};
  } else {
    return {status: 404, body: `No talk '${title}' found`};
  }
});
```

尝试向一个不存在的演讲添加评论将返回 404 错误。

#### `长轮询支持`

服务器最有趣的部分是处理长轮询的部分。当对`/talks`发起`GET`请求时，它可能是一个常规请求或一个长轮询请求。

我们将有多个地方需要向客户端发送一个演讲数组，因此我们首先定义一个帮助方法来构建这样的数组，并在响应中包含一个`ETag`头。

```js
SkillShareServer.prototype.talkResponse = function() {
  let talks = Object.keys(this.talks)
    .map(title => this.talks[title]);
  return {
    body: JSON.stringify(talks),
    headers: {"Content-Type": "application/json",
              "ETag": `"${this.version}"`,
              "Cache-Control": "no-store"}
  };
};
```

处理程序本身需要查看请求头，以确认是否存在`If-None-Match`和`Prefer`头。Node 将头信息以不区分大小写的名称存储为小写形式。

```js
router.add("GET", /^\/talks$/, async (server, request) => {
  let tag = /"(.*)"/.exec(request.headers["if-none-match"]);
  let wait = /\bwait=(\d+)/.exec(request.headers["prefer"]);
  if (!tag || tag[1] != server.version) {
    return server.talkResponse();
  } else if (!wait) {
    return {status: 304};
  } else {
    return server.waitForChanges(Number(wait[1]));
  }
});
```

如果没有提供标签，或者提供的标签与服务器当前版本不匹配，处理程序将响应演讲列表。如果请求是条件性的且演讲没有改变，我们会查看`Prefer`头，以决定是否延迟响应或立即回应。

延迟请求的回调函数存储在服务器的等待数组中，以便在发生某些事情时能够通知它们。`waitForChanges`方法还会立即设置一个定时器，在请求等待足够长的时间后以 304 状态进行响应。

```js
SkillShareServer.prototype.waitForChanges = function(time) {
  return new Promise(resolve => {
    this.waiting.push(resolve);
    setTimeout(() => {
      if (!this.waiting.includes(resolve)) return;
      this.waiting = this.waiting.filter(r => r != resolve);
 resolve({status: 304});
    }, time * 1000);
  });
};
```

使用`updated`注册更改会增加版本属性并唤醒所有等待的请求。

```js
SkillShareServer.prototype.updated = function() {
  this.version++;
  let response = this.talkResponse();
  this.waiting.forEach(resolve => resolve(response));
  this.waiting = [];
};
```

这就是服务器代码的全部内容。如果我们创建`SkillShare Server`的实例并在 8000 端口启动它，生成的 HTTP 服务器将从`public`子目录提供文件，并在`/talks` URL下提供讲座管理界面。

```js
new SkillShareServer({}).start(8000);
```

### 客户端

技能共享网站的客户端部分由三个文件组成：一个小的 HTML 页面、一个样式表和一个 JavaScript 文件。

#### `HTML`

当直接请求与目录对应的路径时，Web 服务器通常会尝试提供名为`index.xhtml`的文件。我们使用的文件服务器模块`serve-static`支持这一约定。当对路径`/`发出请求时，服务器会查找文件`./public/index.xhtml`（`./public`是我们指定的根目录），并在找到时返回该文件。

因此，如果我们希望在浏览器指向我们的服务器时显示一个页面，我们应该将其放在`public/index.xhtml`中。这是我们的索引文件：

```js
<!doctype html>
<meta charset="utf-8">
<title>Skill Sharing</title>
<link rel="stylesheet" href="skillsharing.css">

<h1>Skill Sharing</h1>

<script src="skillsharing_client.js"></script>
```

它定义了文档标题，并包含一个样式表，该样式表定义了一些样式，以确保讲座之间有一些空间。然后，它在页面顶部添加了一个标题，并加载包含客户端应用程序的脚本。

#### `操作`

应用程序状态包括讲座列表和用户的名字，我们将其存储在一个`{talks, user}`对象中。我们不允许用户界面直接操作状态或发送 HTTP 请求。相反，它可以发出描述用户尝试执行的操作的`动作`。

`handleAction`函数接受这样的操作并使其生效。由于我们的状态更新非常简单，状态更改在同一函数中处理。

```js
function handleAction(state, action) {
  if (action.type == "setUser") {
    localStorage.setItem("userName", action.user);
    return {...state, user: action.user};
  } else if (action.type == "setTalks") {
    return {...state, talks: action.talks};
  } else if (action.type == "newTalk") {
    fetchOK(talkURL(action.title), {
      method: "PUT",
      headers: {"Content-Type": "application/json"},
      body: JSON.stringify({
        presenter: state.user,
        summary: action.summary
      })
    }).catch(reportError);
  } else if (action.type == "deleteTalk") {
    fetchOK(talkURL(action.talk), {method: "DELETE"})
      .catch(reportError);
  } else if (action.type == "newComment") {
    fetchOK(talkURL(action.talk) + "/comments", {
      method: "POST",
      headers: {"Content-Type": "application/json"},
      body: JSON.stringify({
        author: state.user,
        message: action.message
      })
    }).catch(reportError);
  }
  return state;
}
```

我们将用户的名字存储在`localStorage`中，以便在页面加载时能够恢复。

需要与服务器交互的操作会使用`fetch`发起网络请求，访问前面描述的 HTTP 接口。我们使用一个包装函数`fetchOK`，以确保当服务器返回错误代码时，返回的 Promise 被拒绝。

```js
function fetchOK(url, options) {
  return fetch(url, options).then(response => {
    if (response.status < 400) return response;
    else throw new Error(response.statusText);
  });
}
```

这个辅助函数用于构建具有给定标题的讲座的 URL。

```js
function talkURL(title) {
  return "talks/" + encodeURIComponent(title);
}
```

当请求失败时，我们不希望页面就这样静止不动而没有解释。我们使用的名为`reportError`的函数作为捕获处理程序，向用户显示一个简单的对话框，告诉他们出了点问题。

```js
function reportError(error) {
  alert(String(error));
}
```

#### `渲染组件`

我们将采用类似于在第十九章中看到的方法，将应用程序分为组件。然而，由于某些组件要么从不需要更新，要么在更新时总是完全重绘，因此我们将那些定义为函数，而不是类，直接返回一个`DOM`节点。例如，下面是一个显示用户可以输入其姓名的字段的组件：

```js
function renderUserField(name, dispatch) {
  return elt("label", {}, "Your name: ", elt("input", {
    type: "text",
    value: name,
    onchange(event) {
      dispatch({type: "setUser", user: event.target.value});
    }
  }));
}
```

`elt`函数用于构建`DOM`元素，这是我们在第十九章中使用的函数。

一个类似的函数用于渲染讲座，其中包括评论列表和添加新评论的表单。

```js
function renderTalk(talk, dispatch) {
  return elt(
    "section", {className: "talk"},
    elt("h2", null, talk.title, " ", elt("button", {
      type: "button",
      onclick() {
        dispatch({type: "deleteTalk", talk: talk.title});
      }
    }, "Delete")),
    elt("div", null, "by ",
        elt("strong", null, talk.presenter)),
    elt("p", null, talk.summary),
    ...talk.comments.map(renderComment),
    elt("form", {
      onsubmit(event) {
        event.preventDefault();
        let form = event.target;
        dispatch({type: "newComment",
                  talk: talk.title,
                  message: form.elements.comment.value});
        form.reset();
      }
    }, elt("input", {type: "text", name: "comment"}), " ",
       elt("button", {type: "submit"}, "Add comment")));
}
```

“提交”事件处理程序在创建`newComment`操作后调用`form.reset`以清除表单内容。

当创建中等复杂度的`DOM`元素时，这种编程风格开始显得相当混乱。为避免这种情况，人们通常使用`模板语言`，它允许你将界面作为一个`HTML`文件编写，并使用一些特殊标记来指示动态元素的位置。或者他们使用`JSX`，这是一种非标准的`JavaScript`方言，允许你在程序中编写非常接近`HTML`标签的内容，就好像它们是`JavaScript`表达式。上述两种方法都使用额外的工具在运行之前预处理代码，这一章我们将避免使用这些工具。

评论的渲染非常简单。

```js
function renderComment(comment) {
  return elt("p", {className: "comment"},
             elt("strong", null, comment.author),
             ": ", comment.message);
}
```

最后，用户可以用来创建新讲座的表单渲染如下：

```js
function renderTalkForm(dispatch) {
  let title = elt("input", {type: "text"});
  let summary = elt("input", {type: "text"});
  return elt("form", {
    onsubmit(event) {
      event.preventDefault();
      dispatch({type: "newTalk",
                title: title.value,
                summary: summary.value});
      event.target.reset();
    }
  }, elt("h3", null, "Submit a Talk"),
     elt("label", null, "Title: ", title),
     elt("label", null, "Summary: ", summary),
     elt("button", {type: "submit"}, "Submit"));
}
```

#### `轮询`

要启动应用程序，我们需要当前的讲座列表。由于初始加载与长轮询过程密切相关——加载时的`ETag`必须在轮询时使用——我们将编写一个函数，该函数持续向服务器轮询`/talks`，并在有新的讲座集可用时调用回调函数。

```js
async function pollTalks(update) {
  let tag = undefined;
  for (;;) {
    let response;
    try {
      response = await fetchOK("/talks", {
        headers: tag && {"If-None-Match": tag,
                         "Prefer": "wait=90"}
      });
    } catch (e) {
      console.log("Request failed: " + e);
      await new Promise(resolve => setTimeout(resolve, 500));
      continue;
    }
    if (response.status == 304) continue;
    tag = response.headers.get("ETag");
    update(await response.json());
  }
}
```

这是一个异步函数，以便于循环和等待请求。它运行一个无限循环，在每次迭代中检索讲座列表——要么正常检索，要么如果这不是第一次请求，则包含使其成为长轮询请求的头部信息。

当请求失败时，函数会等待片刻然后重试。这样，如果你的网络连接暂时中断后又恢复，应用程序可以恢复并继续更新。通过`setTimeout`解决的promise是一种强制异步函数等待的方法。

当服务器返回`304`响应时，意味着长轮询请求超时，因此函数应立即开始下一个请求。如果响应是正常的`200`响应，则其主体将被读取为`JSON`并传递给回调函数，其`ETag`头部值将被存储以供下次迭代使用。

#### `应用程序`

以下组件将整个用户界面串联在一起：

```js
class SkillShareApp {
  constructor(state, dispatch) {
    this.dispatch = dispatch;
    this.talkDOM = elt("div", {className: "talks"});
    this.dom = elt("div", null,
                   renderUserField(state.user, dispatch),
                   this.talkDOM,
                   renderTalkForm(dispatch));
    this.syncState(state);
  }

  syncState(state) {
    if (state.talks != this.talks) {
      this.talkDOM.textContent = "";
      for (let talk of state.talks) {
        this.talkDOM.appendChild(
          renderTalk(talk, this.dispatch));
      }
      this.talks = state.talks;
    }
  }
}
```

当讲座发生变化时，该组件会重新绘制所有讲座。这很简单，但也很浪费。我们将在练习中回到这个问题。

我们可以这样启动应用程序：

```js
function runApp() {
  let user = localStorage.getItem("userName") || "Anon";
  let state, app;
  function dispatch(action) {
    state = handleAction(state, action);
 app.syncState(state);
  }

  pollTalks(talks => {
    if (!app) {
      state = {user, talks};
      app = new SkillShareApp(state, dispatch);
      document.body.appendChild(app.dom);
    } else {
      dispatch({type: "setTalks", talks});
    }
  }).catch(reportError);
}

runApp();
```

如果你运行服务器并在两个浏览器窗口中打开`http://localhost:8000`，你会看到你在一个窗口中执行的操作会立即在另一个窗口中显示。

### 练习

以下练习将涉及修改本章定义的系统。为了解决这些问题，请确保你已下载代码（`[eloquentjavascript.net/code/skillsharing.zip](https://eloquentjavascript.net/code/skillsharing.zip)`），安装了`Node`（`[nodejs.org](https://nodejs.org)`），并使用`npm install`安装项目依赖。

#### `磁盘持久性`

技能分享服务器将其数据完全保存在内存中。这意味着当它崩溃或因任何原因重新启动时，所有讲座和评论都将丢失。

扩展服务器，使其将谈话数据存储到磁盘，并在重启时自动重新加载数据。不要担心效率——只需做最简单有效的事情。

#### `评论字段重置`

对谈话进行全面重绘效果不错，因为通常你无法区分一个`DOM`节点和它的相同替代品。但也有例外。如果你在一个浏览器窗口的评论字段中开始输入内容，然后在另一个窗口中给该谈话添加评论，第一个窗口中的字段将被重绘，移除其内容和焦点。

当多个人同时添加评论时，这会很烦人。你能想出解决办法吗？

`重大优化来自于对高层设计的精炼，而不是单个例程。`

—史蒂夫·麦康奈尔，`《代码大全》`

![图片](img/f0372-01.jpg)
