

## 第一章：1 NODE.JS



![](img/Drop-image.jpg)

Node.js 是一个开源的运行时环境，用于在 Web 浏览器外执行 JavaScript 代码。例如，你可以将它用作脚本语言，执行各种任务，如删除和移动文件、在服务器端记录数据，甚至创建自己的 Web 服务器（如我们在本章练习中将要做的那样）。

学会使用 Node.js 其实并不是了解单个命令或包的使用，因为它基于标准的 JavaScript，你可以参考文档来了解其语法和参数。相反，所有开发者应该努力理解 Node.js 生态系统，并利用它来发挥自己的优势。本章将向你介绍这一点。

### 安装 Node.js

首先通过在命令行中运行 node 命令来检查本地机器上是否已经安装了 Node.js。版本标志 (-v) 应该返回当前的 Node.js 版本：

```
$ **node -v**
```

如果你看到带有版本号的输出，说明 Node.js 已经安装。如果没有，或者版本低于 [*https://<wbr>nodejs<wbr>.org*](https://nodejs.org) 上列出的当前推荐稳定版本，你应该安装这个稳定版本。

若要在本地安装 Node.js，请访问 [*https://<wbr>nodejs<wbr>.org<wbr>/en<wbr>/download*](https://nodejs.org/en/download)，并选择适合你操作系统的安装程序。我建议安装 Node.js 的长期支持 (LTS) 版本，因为许多 Node.js 模块要求使用该版本。运行 Node.js LTS 和 npm 的安装包，然后再次检查版本号。它应该与刚才安装的版本相匹配。

接下来，我们将回顾 Node.js 运行时环境的基本命令和功能。如果你不想安装 Node.js，你可以在在线代码编辑器中运行 Node.js 命令行示例和 JavaScript 代码，地址是 [*https://<wbr>codesandbox<wbr>.io<wbr>/s<wbr>/new*](https://codesandbox.io/s/new) 和 [*https://<wbr>stackblitz<wbr>.com*](https://stackblitz.com)。

### 使用 npm

Node.js 的默认包管理器是 npm。你可以在这里找到各种任务的模块，这些模块来自于在线注册表 [*https://<wbr>www<wbr>.npmjs<wbr>.com*](https://www.npmjs.com)。通过在命令行中运行以下命令，确认你的本地机器上是否安装了 npm：

```
$ **npm -v**
```

如果没有列出版本，或者版本低于当前发布的版本，请安装最新的 Node.js LTS 版本，包括 npm。

请注意，[*https://<wbr>www<wbr>.npmjs<wbr>.com*](https://www.npmjs.com) 上没有审核过程或质量控制。任何人都可以发布包，网站依赖社区报告任何恶意或损坏的包。

运行以下命令会显示可用命令列表：

```
$ **npm**
```

> 注意
> 
> *npm 的最流行替代品是 yarn，它也使用* [`<wbr>www<wbr>.npmjs<wbr>.com`](https://www.npmjs.com) *注册表，并且与 npm 完全兼容。*

### package.json 文件

*package.json* 文件是每个基于 Node.js 的项目中的关键元素。虽然 *node_modules* 文件夹包含实际的代码，但 *package.json* 文件保存了关于项目的所有元数据。它位于项目的根目录，必须包含项目的名称和版本；此外，它还可以包含可选数据，例如项目描述、许可证、脚本以及更多详细信息。

让我们来看看你将在 练习 1 中创建的网页服务器的 *package.json* 文件，位于第 13 页。它应该与 列表 1-1 中展示的类似。

```
{
    "name": "sample-express",
    "version": "1.0.0",
    "description": "sample express server",
    "main": "index.js",
    "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1",
        "run": "node index.js"
    },
    "author": "",
    "license": "ISC",
    "dependencies": {
        "express":"⁴.18.2"
    }
} 
```

列表 1-1：用于 Express.js 服务器项目的 package.json 文件，见 练习 1

*package.json* 文件包含其他人需要在他们的机器上安装所需模块并运行应用程序的所有信息。因此，你不需要将 *node_modules* 文件夹包含在代码库中，这可以最小化代码库的大小。我们来详细看看 *package.json* 文件。

#### 必填字段

*package.json* 文件必须包含 name 字段和 version 字段。所有其他字段都是可选的。name 字段包含包的名称，必须是一个小写字母单词，但可以包含连字符和下划线。

version 字段必须遵循语义版本控制指南，建议使用以下格式：*major.minor.patch*；例如，*1.2.3*。我们称之为*语义*版本控制，因为每个数字都有特定含义。*major* 版本引入不兼容的 API 更改。通常，切换到另一个 major 版本时要非常小心，因为你无法预期你的应用程序能完美运行。*minor* 版本更改以向后兼容的方式添加新功能，因此一般不会对你的应用程序造成问题。*patch* 版本修复向后兼容的 bug，且你应该始终保持它的最新版本。

> 注意

*你可以阅读更多关于语义版本控制和如何定义不同版本范围的信息，访问* [`<wbr>semver<wbr>.org`](https://semver.org)*。*

#### 依赖关系

最重要的可选字段指定了依赖关系和开发依赖关系。dependencies 字段列出了运行项目所需的所有依赖项及其所需的版本范围，遵循语义化版本控制语法。默认情况下，npm 只要求指定主版本，并保持次版本和修订版本范围的灵活性。这样，npm 就能始终使用最新的兼容版本初始化你的项目。

这些依赖项是你打包应用的一部分。当你在新机器上安装一个项目时，*package.json* 文件中列出的所有依赖项将被安装，并放置在 *node_modules* 文件夹中，紧邻 *package.json* 文件。

你的应用可能需要各种依赖项，例如框架和辅助模块。例如，我们将在第二部分中构建的 Food Finder 应用必须至少包含 Next.js 作为单页应用框架，以及 Mongoose 和 MongoDB 作为数据库层。

#### 开发依赖关系

devDependencies 字段列出了开发项目所需的所有依赖项及其版本。再次强调，只有主版本是固定的。这些依赖项仅在开发时需要，并不用于运行应用程序。因此，它们会被打包脚本忽略，并不包含在部署的应用中。当你在新机器上安装项目时，*package.json* 文件中列出的所有开发依赖项将被安装并放置在 *node_modules* 文件夹中，紧邻 *package.json* 文件。对于我们的 Food Finder 应用，我们的开发依赖项将包括 TypeScript 的类型定义。其他常见的依赖项包括测试框架、代码检查工具和构建工具，例如 webpack 和 Babel。

### package-lock.json 文件

npm 包管理器会为每个项目自动生成 *package-lock.json* 文件。这个锁文件解决了使用语义化版本控制来管理依赖时所引入的问题。如前所述，npm 默认只定义主版本，并使用最新的次版本和修订版本。虽然这样可以确保应用包含最新的 bug 修复，但它也引入了一个新问题：没有确切的版本时，构建无法复现。由于 npm 注册表没有质量控制，即使是修订版或次版本更新，也可能引入不兼容的 API 更改，而这种更改本应该是主版本更新。因此，版本之间的轻微偏差可能导致构建失败。

*package-lock.json* 文件通过跟踪每个包及其依赖项的确切版本来解决这个问题。这个文件通常相当大，但它列出的与你将在本章末尾创建的 Web 服务器相关的条目将类似于 列表 1-2。

```
{
    "name": "sample-express",
    "lockfileVersion": 2,
    "requires": true,
    "packages": {
        "": {
            "dependencies": {
                "express": "⁴.18.2"
            }
        },
        "node_modules/accepts": {
            "version": "1.3.8",
            "resolved": "https://registry.npmjs.org/accepts/-/accepts-1.3.8.tgz",
            "integrity": "sha512-PYAthTa2m2VKxuvSD3DPC/Gy+U+sOA1LAuT8mkmRuvw+NACSaeXEhosdQ==",
            `--snip--`
        },
        `--snip--`
        "node_modules/express": {
            "version": "4.18.2",
            "resolved": "https://registry.npmjs.org/express/-/express-4.18.2.tgz",
            "integrity": "sha512-5/PsL6iGPdfQ/lKM1UuielYgv3BUoJfz1aUwU9vHZ+J7gyvwdQXFEBIEI==",
            "dependencies": {
                "accepts": "~1.3.8",
                `--snip--`
                "vary": "~1.1.2"
            },
            "engines": {
                "node": ">= 0.10.0"
            }
        },
        `--snip--`
        "vary": {
            "version": "1.1.2",
            "resolved": "https://registry.npmjs.org/vary/-/vary-1.1.2.tgz",
            "integrity": "sha512-BNGbWLfd0eUPabhkXUVm0j8uuvREyTh5ovRa/dyow/BqAbZJyC+bfhskkh=="
        }
    }
} 
```

列表 1-2：练习 1 的 package-lock.json 文件

锁定文件包含对项目的引用，并列出来自相应 *package.json* 文件的信息。然后，它列出所有项目的依赖项；对我们来说，唯一的依赖项是 Express.js，并且版本是固定的。（我们将在 练习 1 中讲解 Express.js。）此外，该文件列出了正在使用的 Express.js 版本的所有依赖项，在本例中是 *accept* 和 *vary* 包。存储的工件的 SHA 哈希使得 npm 在下载资源后能够验证其完整性。

现在，所有模块版本已被锁定，每次运行 `npm install` 命令都会创建与原始设置完全相同的克隆。像 *package.json* 一样，*package-lock.json* 文件也是代码仓库的一部分。

### 创建项目

让我们来看看日常工作中最重要的命令，按照你在创建和维护项目时逻辑上会使用它们的顺序。在执行这些步骤之后，你将拥有一个 *package.json* 文件和一个包含已安装包 Express.js 的生产就绪项目文件夹。

#### 初始化新模块或项目

要启动一个新项目，运行 `npm init`，它会初始化一个新模块。这将启动一个交互式向导，您可以根据自己的输入填写项目的 *package.json* 文件：

```
$ **mkdir sample-express**
$ **cd sample-express**
$ **npm init**
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sensible defaults.
`--snip--`
Is this OK? **(yes)** 
```

在每个项目的开始，你需要在一个空文件夹中初始化一个新的 Node.js 设置（这里通过 `mkdir sample-express` 创建）并使用 `npm init`。为了简便起见，在这里保持默认建议。助手将在你的项目文件夹中创建一个基本的 *package.json* 文件。它应该类似于 列表 1-3。

```
{
    "name": " sample-express",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "scripts": {
      "test": "echo \"Error: no test specified\" && exit 1"
    },
    "author": "",
    "license": "ISC"
} 
```

列表 1-3：默认的 package.json 文件

当我们将这个文件与 列表 1-1 中显示的文件进行比较时，我们可以看到它们非常相似，除了依赖项和开发依赖项不同。准备好 *package.json* 文件后，我们现在可以使用 `npm install` 安装这些依赖项。

#### 安装依赖项

Node.js 提供了用于执行任务的模块，例如访问文件系统的输入输出、使用网络协议（如 DNS、HTTP、TCP、TLS/SSL 和 UDP）以及处理二进制数据。它还提供了加密模块、用于处理数据流的接口等。

运行 npm install <package> 会下载并将特定的包放置在 *node_modules* 文件夹中，紧邻你的 *package.json* 文件，并将其添加到 *package.json* 中的依赖列表中。每当你需要添加运行应用程序所需的新模块时，应使用此命令。

假设你想创建一个基于 Express.js 的新服务器。你需要从 [*https://<wbr>npmjs<wbr>.com*](https://npmjs.com) 安装 Express.js 包。在这里，我们安装一个特定版本，但如果要安装最新版本，可以省略版本号，改用 npm install express：

```
$ **npm install express@4.18.2**
added 57 packages, and audited 58 packages in 1s
found 0 vulnerabilities 
```

现在，*node_modules* 文件夹包含一个 *express* 文件夹和其他一些依赖文件夹。此外，Express.js 被列为 *package.json* 中的一个依赖项，如 清单 1-4 所示。

```
{
    "name": " sample-express",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "scripts": {
      "test": "echo \"Error: no test specified\" && exit 1"
    },
    "author": "",
    "license": "ISC",
    "dependencies": {
      "express": "⁴.18.2"
    }
} 
```

清单 1-4：默认的 package.json 文件，其中 Express.js 是一个依赖项

我们已成功将 Express.js 作为一个依赖项添加。

#### 安装开发依赖

假设你现在想使用一个叫做 *karma* 的包来进行服务器的端到端测试。与 Express.js 不同，这个包仅在开发过程中使用，并且不需要在实际应用运行时使用。

在这种情况下，你应该运行 npm install --save-dev package 来下载该包，并将其添加到本地 *package.json* 文件中的 devDependencies 列表中：

```
$ **npm install --save-dev karma@5.0.0**
added 128 packages, and audited 186 packages in 3m
9 vulnerabilities (1 moderate, 4 high, 4 critical)

To address issues that do not require attention, run:
  npm audit fix

To address all issues (including breaking changes), run:
  npm audit fix --force

Run `npm audit` for details. 
```

请注意，在安装 *karma* 包后，npm 表示该版本存在已知漏洞。尽管如此，它仍被添加到 *node_modules* 文件夹，并作为 devDependency 列出在 *package.json* 中。稍后我们将按照建议修复这些问题。

#### 审核 package.json 文件

在安装过程中，npm 提示 *karma* 存在一个漏洞，我们来验证一下。npm audit 命令会检查本地的 *package.json* 文件是否有已知的漏洞：

```
$ **npm audit**

# npm audit report
`--snip--`
karma  <=6.3.15
Severity: high
Open redirect in karma - https://github.com/advisories/GHSA-rc3x-jf5g-xvc5
Cross-site Scripting in karma - https://github.com/advisories/GHSA-7x7c-qm48-pq9c
Depends on vulnerable versions of log4js
Depends on vulnerable versions of ua-parser-js
fix available via `npm audit fix --force`
Will install karma@6.4.1, which is a breaking change
`--snip--`
9 vulnerabilities (1 moderate, 4 high, 4 critical)

To address issues that do not require attention, run:
  npm audit fix

To address all issues (including breaking changes), run:
  npm audit fix --force 
```

运行该命令后，会返回一个关于每个问题包的版本和严重性详细报告，以及当前安装的 Node.js 模块中所有问题的总结。

npm 包管理器还表示，问题可以通过npm audit fix自动修复。可惜的是，它警告我们最新的*karma*版本存在破坏性变更。为了适应这些变化，我们需要使用--force标志。我建议每隔几个月使用一次npm audit，并结合使用npm update，以避免使用过时的依赖并造成安全风险：

```
$ **npm audit fix --force**
added 13 packages, removed 41 packages, changed 27 packages, and audited 158 packages in 5s 
```

现在我们看到*package.json*中的<sup class="SANS_TheSansMonoCd_W5Regular_11">devDependencies</sup>列表已经包含了最新的*karma*版本，并且再次运行npm audit报告显示已安装的软件包没有已知漏洞。

#### 清理 node_modules 文件夹

运行npm prune会检查本地*package.json*文件，将其与本地*node_modules*文件夹进行比较，并移除所有不必要的包。你应该在开发过程中使用它，尤其是在添加或移除包后，或者进行常规清理工作时。

让我们检查一下我们刚刚执行的审计是否安装了不必要的包：

```
$ **npm prune**
up to date, audited 136 packages in 1s

found 0 vulnerabilities 
```

输出看起来没问题；我们的包没有问题。

#### 更新所有包

运行npm update会将所有已安装的包更新到最新的可接受版本。你应该经常使用此命令，以避免过时的依赖和安全风险：

```
$ **npm update**
added 1 package, removed 1 package, changed 1 package, and audited 158 packages in 8s

found 0 vulnerabilities 
```

如你所见，npm update会显示更新摘要。

#### 移除依赖

运行npm uninstall package会从本地*node_modules*文件夹和*package.json*文件中移除该包及其依赖项。你应该在删除不再需要的模块时使用此命令。比如，你决定不再需要与*karma*的端到端测试：

```
$ **npm uninstall karma**
removed 71 packages, and audited 138 packages in 3s

found 0 vulnerabilities 
```

该命令的输出显示了对*node_modules*文件夹所做的更改。该软件包也已从*package.json*中移除。

#### 安装依赖

运行 npm install 会从 npm 仓库下载所有依赖项和 devDependencies，并将它们放置在 *node_modules* 文件夹中。使用此命令可以在新机器上安装现有项目。例如，要在新文件夹中安装 Express.js 项目的副本，您可以创建一个新的空文件夹，只将 *package.json* 和 *package-lock.json* 文件复制到其中。然后，您可以在该文件夹中运行 npm install 命令：

```
$ **npm install**
added 137 packages, and audited 138 packages in 3s

found 0 vulnerabilities 
```

每当您克隆仓库或从 *package.json* 文件创建新项目时，运行 npm install。与所有以前的命令一样，npm 会显示一个状态报告，列出任何漏洞。

#### 仅使用 npx 执行一次脚本

当您安装 Node.js 时，您也安装了 npx，它代表 *node package execute*。该工具使您能够在不预先安装的情况下执行注册表中的任何包。当您只需要执行某些代码一次时，这非常有用。例如，您可能会使用一个脚手架脚本来初始化一个项目，但它既不是依赖项，也不是开发依赖项。

npx 工具通过检查您尝试运行的可执行文件是否通过 $PATH 环境变量或本地项目的二进制文件可用来工作。如果不可用，npx 会将包安装到中央缓存中，而不是您的本地 *node_modules* 文件夹中。假设您想检查包 JSON 是否有语法错误。为此，您可以使用 *jsonlint* 包。由于该包既不需要运行项目，也不是您开发过程的一部分，因此您不希望将其安装到 *node_modules* 文件夹中：

```
$ **npx jsonlint package.json**
Need to install the following packages:
  jsonlint
Ok to proceed? (y) **y**
{
    "name": " sample-express",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
    },
    "author": "",
    "license": "ISC",
    "dependencies": {
        "express": "⁴.18.2"
    }
} 
```

这会调用 *jsonlint* 来通过 npx 验证我们的 *package.json* 文件。首先，npx 会将包安装到全局缓存文件夹中，然后运行 *jsonlint*。它会打印我们的 *package.json* 文件内容，并报告没有错误。检查您的 *node_modules* 文件夹；*jsonlint* 不应该被安装。然而，在随后的每次调用 npx 时，您会发现 *jsonlint* 可用。

练习 1：构建一个“Hello World” Express.js 服务器

Express.js 是一个免费且开源的后端框架，建立在 Node.js 之上。它旨在构建 web 应用程序和 API，是 Node.js 生态系统中事实上的标准服务器框架，也是全栈 web 开发的基础。

Express.js 提供了 HTTP 服务器常用的中间件，用于任务如缓存、内容协商、Cookie 处理、跨域请求处理、重定向等。

> 注意

*Next.js 使用自己内置的服务器，该服务器大量借鉴了 Express.js。在本书的第二部分中，你将构建一个“食物查找器”应用程序，Next.js 将成为你所使用的中间件的基础。由于 Next.js 为你抽象了这个中间件，你将不会直接与服务器进行交互。*

让我们构建一个基于 Express.js 的简单 Node.js 服务器，以练习你的 Node.js 技能。

#### 设置

如果你在跟随本章的过程中已经创建了*sample-express*文件夹和*package.json*文件，那么可以跳过此设置。否则，创建并切换到一个名为*sample-express*的新文件夹。然后，在命令行中运行 npm init 来初始化一个新的 Node.js 项目。交互式向导会要求你提供一些细节，例如应用程序的名称和版本。现在可以接受默认设置。

接下来，你需要使用 Express.js 包作为服务器的基础。运行 npm install express@4 来安装主要版本 4 的最新发布版。你会看到*package.json*文件现在将*express*作为依赖项。

#### 编写服务器代码

在*sample-express*文件夹中创建一个*index.js*文件，并添加列表 1-5 中的代码。

```
const express = require('express');
const server = express();
const port = 3000;

server.get('/hello', function (req, res) {
    res.send('Hello World!');
});

server.listen(port, function () {
    console.log('Listening on ' + port);
}); 
```

列表 1-5：一个基本的 Express.js 服务器

首先，我们将*express*包加载到文件中，实例化应用程序，并定义一个常量来指定要使用的端口。然后，我们为服务器创建一个路由，使其能够响应每一个发送到*/hello*基本 URL 的 GET 请求，并返回 Hello World!。我们使用 Express.js 的 get 方法，并将 /hello 作为第一个参数，回调函数作为第二个参数。现在，每次发送到*/hello*端点的 GET 请求，服务器都会运行回调函数并返回 Hello World! 作为响应。最后，我们使用 Express.js 的 listen 方法启动 Web 服务器，并告诉它在 3000 端口监听。

从命令行启动服务器：

```
$ **node index.js**
Listening on 3000 
```

现在，在浏览器中访问*http://localhost:3000/hello*。你应该会看到 Hello World! 消息。恭喜你！你刚刚用 JavaScript 写了你的第一个 Node.js Web 服务器。

### 总结

本章教会了你如何使用 Node.js 及其模块生态系统在浏览器外运行 JavaScript 代码。你学习了如何在全栈应用中使用、添加和移除模块，掌握了 npm 命令的使用方法，以及如何读取和使用*package.json*和*package-lock.json*文件。最后，你对 Express.js 进行了初步了解，它是全栈开发的事实标准服务器，并使用它通过几行代码构建了一个示例 Node.js 服务器。

本章仅仅触及了 Node.js 的表面。如果你想深入探索它的全部潜力，我推荐 W3Schools 的 Node.js 教程，网址是[*https://<wbr>www<wbr>.w3schools<wbr>.com<wbr>/nodejs<wbr>/*](https://www.w3schools.com/nodejs/)以及 Udemy 上免费的 ExpressJS 基础课程，网址是[*https://<wbr>www<wbr>.udemy<wbr>.com<wbr>/course<wbr>/expressjs<wbr>-fundamentals<wbr>/*](https://www.udemy.com/course/expressjs-fundamentals/)。

在下一章，你将了解 ES.Next，这是 JavaScript 的最新版本，并掌握它为开发带来的现代特性。
