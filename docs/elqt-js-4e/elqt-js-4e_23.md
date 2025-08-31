# 第二十一章：`NODE.JS`

到目前为止，我们只在一个环境中使用了JavaScript语言：浏览器。本章和下一章将简要介绍Node.js，这是一个允许你在浏览器外应用JavaScript技能的程序。通过它，你可以构建从小型命令行工具到支持动态网站的HTTP服务器的各种应用。

这些章节旨在教你Node.js使用的主要概念，并给你足够的信息来编写有用的程序。它们并不试图全面或深入地介绍该平台。

如果你想跟着本章的代码运行，你需要安装Node.js版本18或更高版本。要做到这一点，请访问`[nodejs.org](https://nodejs.org)`并按照你操作系统的安装说明进行操作。你还可以在那里找到Node.js的进一步文档。

### 背景

在构建通过网络通信的系统时，你管理输入和输出的方式——即从网络和硬盘读取和写入数据的方式——会对系统对用户或网络请求的响应速度产生很大影响。

在这样的程序中，异步编程通常是有帮助的。它允许程序同时与多个设备发送和接收数据，而无需复杂的线程管理和同步。

Node最初的构想是为了使异步编程变得简单方便。JavaScript非常适合像Node这样的系统。它是少数几种没有内置输入输出方式的编程语言之一。因此，JavaScript能够很好地适应Node对网络和文件系统编程的相当奇特的方法，而不会导致两个不一致的接口。在2009年，当Node被设计时，人们已经在浏览器中进行基于回调的编程，因此围绕该语言的社区已经习惯了异步编程风格。

### `node`命令

当Node.js安装在系统上时，它提供了一个名为`node`的程序，用于运行JavaScript文件。假设你有一个名为`hello.js`的文件，包含以下代码：

```js
let message = "Hello world";
console.log(message);
```

然后你可以像这样从命令行运行`node`以执行程序：

```js
$ node hello.js
Hello world
```

Node中的`console.log`方法与浏览器中的作用类似。它输出一段文本。但在Node中，这段文本会发送到进程的标准输出流，而不是浏览器的JavaScript控制台。当从命令行运行`node`时，这意味着你会在终端中看到记录的值。

如果你运行`node`而不指定文件，它会给你一个提示，你可以在此输入JavaScript代码并立即看到结果。

```js
$ node
> 1 + 1
2
> [-1, -2, -3].map(Math.abs)
[1, 2, 3]
> process.exit(0)
$
```

进程绑定，就像控制台绑定一样，在Node中是全局可用的。它提供了多种方式来检查和操作当前程序。`exit`方法结束进程，并可以指定一个退出状态代码，这告诉启动Node的程序（在此情况下为命令行shell）程序是否成功完成（代码零）或遇到错误（任何其他代码）。

要查找传递给你脚本的命令行参数，你可以读取`process.argv`，它是一个字符串数组。请注意，它还包括`node`命令的名称和你的脚本名称，因此实际参数从索引2开始。如果`showargv.js`包含语句`console.log(process.argv)`，你可以这样运行它：

```js
$ node showargv.js one --and two
["node", "/tmp/showargv.js", "one", "--and", "two"]
```

所有标准的JavaScript全局绑定，如`Array`、`Math`和`JSON`，在Node的环境中也存在。与浏览器相关的功能，如`document`或`prompt`，则不存在。

### 模块

除了我提到的绑定，如`console`和`process`，Node在全局作用域中几乎没有其他绑定。如果你想访问内置功能，你必须请求模块系统。

Node最初使用基于`require`函数的CommonJS模块系统，我们在第十章中看到过。当你加载`*.js`文件时，它仍然会默认使用此系统。

但今天，Node也支持更现代的ES模块系统。当脚本的文件名以`*.mjs`结尾时，它被视为这样的模块，你可以在其中使用`import`和`export`（但不能使用`require`）。我们将在本章中使用ES模块。

当导入一个模块时——无论是使用`require`还是`import`——Node需要将给定的字符串解析为一个实际可以加载的文件。以`/`、`./`或`../`开头的名称将相对于当前模块的路径解析为文件。在这里，`.`表示当前目录，`../`表示上一级目录，`/`表示文件系统的根目录。如果你从文件`/tmp/robot/robot.mjs`请求`"./graph.mjs"`，Node将尝试加载文件`/tmp/robot/graph.mjs`。

当导入的字符串看起来不是相对路径或绝对路径时，假定它指的是内置模块或安装在`node_modules`目录中的模块。例如，从`node:fs`导入将为你提供Node的内置文件系统模块。导入`robot`可能会尝试加载在`node_modules/robot/`中找到的库。通常会使用NPM安装这些库，我们稍后会回到这个话题。

让我们建立一个由两个文件组成的小项目。第一个文件叫做`main.mjs`，它定义了一个可以从命令行调用的脚本，用于反转字符串。

```js
import {reverse} from "./reverse.mjs";

// Index 2 holds the first actual command line argument
let argument = process.argv[2];

console.log(reverse(argument));
```

文件`reverse.mjs`定义了一个用于反转字符串的库，可以被这个命令行工具和需要直接访问字符串反转功能的其他脚本使用。

```js
export function reverse(string) {
  return Array.from(string).reverse().join("");
}
```

请记住，`export`用于声明一个绑定是模块接口的一部分。这允许`main.mjs`导入并使用该函数。

现在我们可以这样调用我们的工具：

```js
$ node main.mjs JavaScript
tpircSavaJ
```

### 使用NPM安装

NPM在第十章中介绍，是一个在线JavaScript模块库，其中许多模块是专门为Node编写的。当你在计算机上安装Node时，你还会得到`npm`命令，可以用来与这个库进行交互。

NPM的主要用途是下载包。我们在第十章中看到了`ini`包。我们可以使用NPM在我们的计算机上获取并安装该包。

```js
$ npm install ini
added 1 package in 723ms

$ node
> const {parse} = require("ini");
> parse("x = 1\ny = 2");
{ x: '1', y: '2' }
```

运行`npm install`后，NPM将创建一个名为`node_modules`的目录。该目录下将包含一个`ini`目录，其中包含库。你可以打开它并查看代码。当我们导入`ini`时，该库被加载，我们可以调用它的解析属性来解析配置文件。

默认情况下，NPM会在当前目录下安装包，而不是在中心位置。如果你习惯于其他包管理器，这可能看起来不寻常，但它有其优势——它让每个应用程序完全控制其安装的包，并更容易管理版本以及在删除应用程序时进行清理。

#### `包文件`

在运行`npm install`安装某个包后，你会发现当前目录下不仅有一个`node_modules`目录，还有一个名为`package.json`的文件。建议每个项目都有这样的文件。你可以手动创建它，或运行`npm init`。此文件包含关于项目的信息，例如其名称和版本，并列出其依赖项。

从第七章的机器人模拟，作为第十章练习中的模块化，可能会有一个这样的`package.json`文件：

```js
{
  "author": "Marijn Haverbeke",
  "name": "eloquent-javascript-robot",
  "description": "Simulation of a package-delivery robot",
  "version": "1.0.0",
  "main": "run.mjs",
  "dependencies": {
    "dijkstrajs": "¹.0.1",
    "random-item": "¹.0.0"
  },
  "license": "ISC"
}
```

当你运行`npm install`而不指定要安装的包时，NPM将安装`package.json`中列出的依赖项。当你安装一个未在依赖项中列出的特定包时，NPM会将其添加到`package.json`中。

#### `版本`

`package.json`文件列出了程序自身的版本以及其依赖项的版本。版本是处理包独立演变的方式，编写的代码在某个时间点与包的版本兼容，可能在后来的修改版本中不再兼容。

NPM要求其包遵循一种称为`语义版本控制`的模式，该模式在版本号中编码了一些关于哪些版本是`兼容`（不破坏旧接口）的信息。语义版本由三个用句点分隔的数字组成，例如`2.3.0`。每次添加新功能时，中间的数字必须增加。每当破坏兼容性时，即现有代码使用的包可能与新版本不兼容，首个数字必须增加。

在`package.json`中，依赖项版本号前的插入符号（`^`）表示可以安装与给定数字兼容的任何版本。例如，“`².3.0`”意味着允许任何大于或等于`2.3.0`且小于`3.0.0`的版本。

`npm`命令也用于发布新包或包的新版本。如果在包含`package.json`文件的目录中运行`npm publish`，它将把JSON文件中列出的名称和版本的包发布到注册表。任何人都可以向NPM发布包——但仅限于尚未使用的包名，因为如果随机用户可以更新现有包，那就不好了。

本书不会深入探讨NPM的使用细节。请参考 *[`www.npmjs.com`](https://www.npmjs.com)* 以获取更多文档和包搜索方法。

### 文件系统模块

Node中最常用的内置模块之一是`node:fs`模块，它代表“文件系统”。它导出用于处理文件和目录的函数。

例如，名为`readFile`的函数读取一个文件，然后调用回调函数传递文件内容。

```js
import {readFile} from "node:fs";
readFile("file.txt", "utf8", (error, text) => {
  if (error) throw error;
  console.log("The file contains:", text);
});
```

`readFile`的第二个参数指示用于将文件解码为字符串的`字符编码`。文本可以以多种方式编码为二进制数据，但大多数现代系统使用`UTF-8`。除非你有理由相信使用了其他编码，否则在读取文本文件时传递“`utf8`”。如果不传递编码，Node会假设你对二进制数据感兴趣，并会返回一个`Buffer`对象，而不是字符串。这个对象类似数组，包含表示文件中字节（8位数据块）的数字。

```js
import {readFile} from "node:fs";
readFile("file.txt", (error, buffer) => {
  if (error) throw error;
  console.log("The file contained", buffer.length, "bytes.",
              "The first byte is:", buffer[0]);
});
```

一个类似的函数`writeFile`用于将文件写入磁盘。

```js
import {writeFile} from "node:fs";
writeFile("graffiti.txt", "Node was here", err => {
  if (err) console.log(`Failed to write file: ${err}`);
  else console.log("File written.");
});
```

在这里不需要指定编码——`writeFile`会假设当它接收到一个字符串而不是`Buffer`对象时，应使用其默认字符编码（`UTF-8`）将其作为文本写出。

`node:fs`模块包含许多其他有用的函数：`readdir`会将目录中的文件作为字符串数组返回，`stat`会检索文件信息，`rename`会重命名文件，`unlink`会删除文件，等等。有关具体信息，请查看 *[`nodejs.org`](https://nodejs.org)* 的文档。

这些函数中的大多数将回调函数作为最后一个参数，调用时要么带有错误（第一个参数），要么带有成功结果（第二个参数）。正如我们在`第十一章`中看到的，这种编程风格有缺点——最大的一个是错误处理变得冗长且容易出错。

`node:fs/promises`模块导出了与旧的`node:fs`模块大部分相同的函数，但使用了`Promise`而不是回调函数。

```js
import {readFile} from "node:fs/promises";
readFile("file.txt", "utf8")
  .then(text => console.log("The file contains:", text));
```

有时你并不需要异步操作，它反而会造成困扰。`node:fs`中的许多函数也有同步变体，其名称后面加上`Sync`。比如，`readFile`的同步版本叫做`readFileSync`。

```js
import {readFileSync} from "node:fs";
console.log("The file contains:",
            readFileSync("file.txt", "utf8"));
```

请注意，在执行这样的同步操作时，程序会完全停止。如果它应该响应用户或网络上的其他机器，被阻塞在同步操作上可能会造成令人烦恼的延迟。

### HTTP 模块

另一个核心模块叫做`node:http`。它提供了运行 HTTP 服务器的功能。

启动 HTTP 服务器所需的就是这些：

```js
import {createServer} from "node:http";
let server = createServer((request, response) => {
  response.writeHead(200, {"Content-Type": "text/html"});
  response.write(`
    <h1>Hello!</h1>
    <p>You asked for <code>${request.url}</code></p>`);
  response.end();
});
server.listen(8000);
console.log("Listening! (port 8000)");
```

如果你在自己的机器上运行这个脚本，你可以将你的 Web 浏览器指向`http://localhost:8000/hello`以向你的服务器发出请求。它将以一个小的 HTML 页面响应。

传递给`createServer`的函数在每次客户端连接到服务器时被调用。请求和响应绑定是表示传入和传出数据的对象。第一个包含有关请求的信息，例如它的`url`属性，告诉我们请求是发往哪个 URL 的。

当你在浏览器中打开该页面时，它会向你自己的计算机发送请求。这会导致服务器函数运行并发送回响应，你随后可以在浏览器中看到。

要向客户端发送内容，你需要在响应对象上调用方法。第一个是`writeHead`，它将写出响应头（见`第十八章`）。你需要提供状态码（在这种情况下是`200`，表示“OK”）和一个包含头部值的对象。示例将`Content-Type`头设置为通知客户端我们将返回一个 HTML 文档。

接下来，实际的响应主体（文档本身）通过`response.write`发送。如果你想逐块发送响应，可以多次调用此方法，例如，随着数据的可用性将数据流式传输给客户端。最后，`response.end`表示响应结束。

对`server.listen`的调用使服务器开始在`8000`端口等待连接。这就是为什么你必须连接到`localhost:8000`来与此服务器进行通信，而不是仅仅使用`localhost`，因为那将使用默认端口`80`。

当你运行这个脚本时，进程只是静静地等待。当一个脚本正在监听事件时——在这种情况下，是网络连接——`node`不会在达到脚本末尾时自动退出。要关闭它，请按`CTRL-C`。

一个真正的 Web 服务器通常比示例中的服务器做得更多——它查看请求的方法（`method`属性）以查看客户端正在尝试执行什么操作，并查看请求的 URL 以找出正在对哪个资源执行此操作。我们将在本章后面看到一个更高级的服务器。

`node:http`模块还提供了一个请求函数，可用于发起 HTTP 请求。然而，与我们在`第十八章`中看到的`fetch`相比，它使用起来要繁琐得多。幸运的是，`fetch`在 Node 中也作为全局绑定可用。除非你想做一些非常具体的事情，例如在数据通过网络传入时逐块处理响应文档，否则我建议使用`fetch`。

### 流

HTTP 服务器可以写入的响应对象是一个`可写流`对象的示例，这是 Node 中广泛使用的概念。这些对象具有`write`方法，可以传入一个字符串或`Buffer`对象，以将内容写入流中。它们的`end`方法关闭流，并且在关闭之前可以选择性地接收一个值以写入流。这两个方法也可以接受一个回调作为额外参数，当写入或关闭完成时会调用该回调。

可以使用来自`node:fs`模块的`createWriteStream`函数创建一个指向文件的可写流。然后，可以在生成的对象上使用`write`方法逐块写入文件，而不是像`writeFile`那样一次性写入。

`可读流`稍微复杂一些。HTTP 服务器回调的请求参数是一个可读流。从流中读取是通过事件处理程序而不是方法来完成的。

在 Node 中发出事件的对象有一个名为`on`的方法，类似于浏览器中的`addEventListener`方法。你给它一个事件名称和一个函数，它会注册该函数，以便在发生给定事件时调用。

`可读流`有`data`和`end`事件。每当数据到达时，第一种事件会被触发，而第二种事件则在流结束时被调用。该模型最适合于`流式`处理数据，即使整个文档尚不可用，也能立即处理。可以通过使用`node:fs`中的`createReadStream`函数将文件作为可读流进行读取。

这段代码创建了一个服务器，读取请求体并将其以全大写文本流回客户端：

```js
import {createServer} from "node:http";
createServer((request, response) => {
  response.writeHead(200, {"Content-Type": "text/plain"});
  request.on("data", chunk =>
    response.write(chunk.toString().toUpperCase()));
  request.on("end", () => response.end());
}).listen(8000);
```

传递给数据处理程序的`chunk`值将是一个二进制`Buffer`。我们可以通过使用其`toString`方法将其解码为 UTF-8 编码的字符，从而转换为字符串。

下面的代码在启动大写服务器时运行，将向该服务器发送请求，并输出收到的响应：

```js
fetch("http://localhost:8000/", {
  method: "POST",
  body: "Hello server"
}).then(resp => resp.text()).then(console.log);
// → HELLO SERVER
```

### `文件服务器`

让我们结合关于 HTTP 服务器和文件系统操作的新知识，创建两者之间的桥梁：一个允许远程访问文件系统的 HTTP 服务器。这样的服务器有各种用途——它允许 Web 应用程序存储和共享数据，或者可以为一组人提供对一堆文件的共享访问。

当我们将文件视为 HTTP 资源时，HTTP 方法`GET`、`PUT`和`DELETE`可以分别用于读取、写入和删除文件。我们将请求中的路径解释为请求所指向的文件的路径。

我们可能不想共享整个文件系统，因此我们将这些路径解释为从服务器的工作目录开始，这就是服务器启动时所在的目录。如果我从`/tmp/public/`（或者在 Windows 上为`C:\tmp\public\`）运行服务器，那么对`/file.txt`的请求应该指向`/tmp/public/file.txt`（或`C:\tmp\public\file.txt`）。

我们将逐步构建程序，使用一个名为`methods`的对象来存储处理各种 HTTP 方法的函数。方法处理程序是异步函数，它们将请求对象作为参数，并返回一个解析为描述响应的对象的承诺。

```js
import {createServer} from "node:http";

const methods = Object.create(null);

createServer((request, response) => {
  let handler = methods[request.method] || notAllowed;
  handler(request).catch(error => {
    if (error.status != null) return error;
    return {body: String(error), status: 500};
  }).then(({body, status = 200, type = "text/plain"}) => {
    response.writeHead(status, {"Content-Type": type});
    if (body?.pipe) body.pipe(response);
    else response.end(body);
  });
}).listen(8000);

async function notAllowed(request) {
  return {
    status: 405,
    body: `Method ${request.method} not allowed.`
  };
}
```

这启动了一个只返回`405`错误响应的服务器，该代码用于指示服务器拒绝处理给定的方法。

当请求处理程序的承诺被拒绝时，`catch`调用将错误转换为响应对象（如果还不是），以便服务器可以发送错误响应，通知客户端处理请求失败。

响应描述的状态字段可以省略，在这种情况下，它默认为`200`（OK）。类型属性中的内容类型也可以省略，在这种情况下，响应被认为是纯文本。

当`body`的值是可读流时，它将具有一个`pipe`方法，我们可以用它将所有内容从可读流转发到可写流。如果不是，则假定它是`null`（没有主体）、字符串或缓冲区，并直接传递给响应的`end`方法。

为了找出哪个文件路径对应于请求 URL，`urlPath`函数使用内置的`URL`类（在浏览器中也存在）来解析 URL。该构造函数期望一个完整的 URL，而不仅仅是以斜杠开头的部分（我们从`request.url`中获得），因此我们提供一个虚拟域名来填充。它提取其路径名，类似于`/file.txt`，对其进行解码以去除`%20`样式的转义码，并相对于程序的工作目录解析它。

```js
import {resolve, sep} from "node:path";

const baseDirectory = process.cwd();

function urlPath(url) {
  let {pathname} = new URL(url, "http://d");
  let path = resolve(decodeURIComponent(pathname).slice(1));
  if (path != baseDirectory &&
      !path.startsWith(baseDirectory + sep)) {
    throw {status: 403, body: "Forbidden"};
  }
  return path;
}
```

一旦你设置了一个程序来接受网络请求，就必须开始担心安全性。在这种情况下，如果我们不小心，很可能会意外地将整个文件系统暴露给网络。

文件路径在 Node 中是字符串。要将这样的字符串映射到实际文件，需要进行相当复杂的解释。例如，路径可能包含`../`来引用父目录。一个明显的问题来源是请求类似于`/../secret_file`的路径。

为了避免此类问题，`urlPath`使用来自`node:path`模块的`resolve`函数，该函数解析相对路径。它随后验证结果是否在工作目录`之下`。`process.cwd`函数（其中`cwd`代表“当前工作目录”）可以用来找到这个工作目录。来自`node:path`包的`sep`绑定是系统的路径分隔符——在 Windows 上是反斜杠，在大多数其他系统上是正斜杠。当路径不以基本目录开头时，该函数会抛出一个错误响应对象，使用 HTTP 状态码指示访问该资源是被禁止的。

我们将设置`GET`方法，以便在读取目录时返回文件列表，并在读取常规文件时返回文件的内容。

一个棘手的问题是我们在返回文件内容时应该设置什么样的`Content-Type`头。由于这些文件可能是任何类型，我们的服务器不能简单地为它们全部返回相同的内容类型。`NPM`在这里可以再次帮助我们。`mime-types`包（像`text/plain`这样的内容类型指示符也被称为`MIME类型`）知道许多文件扩展名的正确类型。

在服务器脚本所在的目录中，以下`npm`命令安装特定版本的`mime`：

```js
$ npm install mime-types@2.1.0
```

当请求的文件不存在时，返回的正确 HTTP 状态码是`404`。我们将使用`stat`函数，该函数查找有关文件的信息，以确定文件是否存在以及它是否是一个目录。

```js
import {createReadStream} from "node:fs";
import {stat, readdir} from "node:fs/promises";
import {lookup} from "mime-types";

methods.GET = async function(request) {
  let path = urlPath(request.url);
  let stats;
  try {
    stats = await stat(path);
  } catch (error) {
    if (error.code != "ENOENT") throw error;
    else return {status: 404, body: "File not found"};
  }
  if (stats.isDirectory()) {
    return {body: (await readdir(path)).join("\n")};
  } else {
    return {body: createReadStream(path),
            type: lookup(path)};
  }
};
```

由于它必须接触磁盘，因此可能需要一些时间，`stat`是异步的。因为我们使用的是`Promise`而不是回调风格，所以它必须从`node:fs/promises`导入，而不是直接从`node:fs`导入。

当文件不存在时，`stat`将抛出一个带有“ENOENT”代码属性的错误对象。这些有些晦涩的、受 Unix 启发的代码是识别 Node 中错误类型的方式。

`stat`返回的`stats`对象告诉我们有关文件的许多信息，例如其大小（`size`属性）和修改日期（`mtime`属性）。在这里，我们关注的问题是它是一个目录还是一个普通文件，这由`isDirectory`方法告诉我们。

我们使用`readdir`读取目录中的文件数组并将其返回给客户端。对于普通文件，我们使用`createReadStream`创建一个可读流，并将其作为主体返回，同时附上`mime`包为文件名提供的内容类型。

处理`DELETE`请求的代码稍微简单一些。

```js
import {rmdir, unlink} from "node:fs/promises";

methods.DELETE = async function(request) {
  let path = urlPath(request.url);
  let stats;
  try {
    stats = await stat(path);
 } catch (error) {
    if (error.code != "ENOENT") throw error;
    else return {status: 204};
  }
  if (stats.isDirectory()) await rmdir(path);
  else await unlink(path);
  return {status: 204};
};
```

当`HTTP`响应不包含任何数据时，可以使用状态码`204`（“无内容”）来指示这一点。由于删除的响应不需要传输除操作是否成功之外的任何信息，因此在这里返回这个是合理的。

你可能会想，为什么尝试删除一个不存在的文件会返回成功状态码而不是错误。当要删除的文件不存在时，可以说请求的目标已经实现。`HTTP`标准鼓励我们使请求`幂等`，这意味着多次发出相同请求的结果与只发出一次相同。在某种意义上，如果你尝试删除已经不存在的东西，那么你试图创造的效果已经实现——那个东西不再在那里。

这是处理`PUT`请求的处理程序：

```js
import {createWriteStream} from "node:fs";

function pipeStream(from, to) {
  return new Promise((resolve, reject) => {
    from.on("error", reject);
    to.on("error", reject);
    to.on("finish", resolve);
    from.pipe(to);
  });
}

methods.PUT = async function(request) {
  let path = urlPath(request.url);
  await pipeStream(request, createWriteStream(path));
  return {status: 204};
};
```

这次我们不需要检查文件是否存在——如果存在，我们只需覆盖它。我们再次使用管道将数据从可读流移动到可写流，在这种情况下是从请求到文件。但由于管道没有返回一个`Promise`，我们需要编写一个包装器`pipeStream`，它在调用管道的结果周围创建一个`Promise`。

当打开文件时出现问题时，`createWriteStream`仍会返回一个流，但该流会触发一个“错误”事件。请求的流也可能失败，例如，如果网络中断。因此，我们将两个流的“错误”事件连接起来，以拒绝该`promise`。当管道操作完成时，它会关闭输出流，这会触发一个“完成”事件。这时我们可以成功解析该`promise`（不返回任何内容）。

服务器的完整脚本可在`[eloquentjavascript.net/code/file_server.mjs](https://eloquentjavascript.net/code/file_server.mjs)`上找到。你可以下载它，并在安装其依赖后，使用`Node`运行它来启动自己的文件服务器。当然，你也可以修改和扩展它，以解决本章的练习或进行实验。

命令行工具`curl`广泛用于类`Unix`系统（如`macOS`和`Linux`），可用于发起`HTTP`请求。以下会话简要测试我们的服务器。`-X`选项用于设置请求的方法，`-d`选项用于包含请求主体。

```js
$ curl http://localhost:8000/file.txt
File not found
$ curl -X PUT -d CONTENT http://localhost:8000/file.txt
$ curl http://localhost:8000/file.txt
CONTENT
$ curl -X DELETE http://localhost:8000/file.txt
$ curl http://localhost:8000/file.txt
File not found
```

对`file.txt`的第一次请求失败，因为文件尚不存在。`PUT`请求创建了该文件，接着下一个请求成功检索了它。在通过`DELETE`请求删除后，文件再次消失。

### 总结

`Node`是一个精巧的小系统，使我们能够在非浏览器环境中运行`JavaScript`。它最初设计用于网络任务，充当网络中的一个节点，但它适用于各种脚本任务。如果编写`JavaScript`是你喜欢的事情，使用`Node`自动化任务可能会对你很有帮助。

`NPM`提供了你能想到的所有包（还有一些你可能从未想到的包），并允许你使用`npm`程序获取和安装这些包。`Node`内置了一些模块，包括用于处理文件系统的`node:fs`模块和用于运行`HTTP`服务器的`node:http`模块。

在`Node`中，所有的输入和输出都是异步进行的，除非你明确使用函数的同步变体，例如`readFileSync`。`Node`最初使用回调来实现异步功能，但`node:fs/promises`包提供了基于`promise`的文件系统接口。

### 练习

#### `搜索工具`

在`Unix`系统上，有一个名为`grep`的命令行工具，可以快速搜索文件中的正则表达式。

编写一个可以从命令行运行的`Node`脚本，行为类似于`grep`。它将第一个命令行参数视为正则表达式，后续参数视为要搜索的文件。它输出任何内容与正则表达式匹配的文件的名称。

当那样工作时，扩展它，使得当其中一个参数是目录时，它会搜索该目录及其子目录中的所有文件。

根据需要使用异步或同步文件系统函数。设置多个异步操作同时请求可能会加快速度，但效果有限，因为大多数文件系统一次只能读取一个文件。

#### `目录创建`

尽管我们文件服务器的`DELETE`方法能够删除目录（使用`rmdir`），但服务器目前不提供任何`创建`目录的方法。

添加对`MKCOL`方法（“创建集合”）的支持，该方法应通过调用`node:fs`模块中的`mkdir`来创建目录。`MKCOL`不是一种广泛使用的`HTTP`方法，但在`WebDAV`标准中确实存在此目的，`WebDAV`在`HTTP`上指定了一组约定，使其适合于创建文档。

#### `网络上的公共空间`

由于文件服务器可以提供任何类型的文件，并且还包含正确的 `Content-Type` 头，因此您可以用它来提供一个网站。考虑到该服务器允许所有人删除和替换文件，这将会创建一种有趣的网站：一个可以被每个花时间进行正确 HTTP 请求的人修改、改进和破坏的网站。

编写一个基本的 HTML 页面，其中包含一个简单的 JavaScript 文件。将文件放在文件服务器提供的目录中，并在浏览器中打开它们。

接下来，作为一个高级练习或周末项目，结合从本书中获得的所有知识，构建一个更用户友好的界面来修改网站——从网站的`内部`。

使用 HTML 表单编辑构成网站的文件内容，允许用户通过使用 HTTP 请求在服务器上更新它们，如第十八章所述。

首先只使单个文件可编辑。然后使用户可以选择要编辑的文件。利用我们的文件服务器在读取目录时返回文件列表的事实。

不要直接在文件服务器暴露的代码中工作，因为如果出错，您可能会损坏那里的文件。相反，将您的工作保留在公共可访问目录之外，并在测试时将其复制到那里。

`如果你有知识，让他人借此点燃他们的蜡烛。`

—`玛格丽特·富勒`

![图片](img/f0354-01.jpg)
