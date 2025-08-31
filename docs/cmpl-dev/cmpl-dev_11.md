

## 第十章：10 使用 Docker 进行容器化



![](img/Drop-image.jpg)

专业的全栈开发者经常使用 Docker，广义上也使用容器。*Docker* 作为一个开源容器化平台，解决了三个常见问题。

首先，它使我们能够为每个项目运行某个特定版本的软件，如 Node.js。其次，它将开发环境与本地机器解耦，并创建一种可复现的方式来运行应用程序。第三，与传统的虚拟机不同，Docker 容器运行在共享主机上。因此，它们的体积更小，消耗的内存比传统虚拟机要少，后者需要模拟完整的系统，且通常依赖特定硬件。因此，基于容器的应用程序轻量且易于扩展。这些优势使得 Docker 成为近年来最受欢迎的开发平台。

本章将介绍 Docker 的基础知识。我们首先演示如何通过创建一个运行最新 Node.js 版本并在容器内提供应用程序的 Docker 容器，将我们的 Next.js 应用容器化。接着，我们将探讨微服务架构的概念，并使用 Docker 创建两个微服务。

### 容器化架构

在日常工作中，开发者必须频繁地在需要不同版本同一库的应用程序之间切换。例如，专注于 JavaScript 的开发者可能需要为每个项目使用不同版本的 Node.js 或 TypeScript。当然，他们可以使用像 nvm 这样的工具，在本地机器上切换已安装的 Node.js 版本，每当需要切换到不同的项目时。但与其采取这种粗糙的方式，他们可以选择一个更优雅的解决方案。

使用 Docker，我们可以将应用程序或其服务分离到独立的容器中，每个容器都提供特定服务的环境。这些容器运行在我们选择的操作系统上（通常是 Debian、Ubuntu 或 Alpine），并且只包含此特定应用程序所需的依赖项。容器之间是隔离的，并通过定义的 API 进行通信。

当我们在开发过程中使用 Docker 容器时，我们便于应用程序的后续部署。毕竟，容器提供了一个与平台无关的版本，这意味着无论在哪个环境中，应用程序都能正常运行。因此，我们已经知道我们的应用程序能够与已安装的依赖项配合使用，不需要解决冲突或执行额外的安装步骤。与其设置一个远程服务器，安装所需的软件，然后再部署和测试我们的应用程序，我们可以直接将 Docker 容器移到服务器上，并在那里启动它。

在需要迁移到不同服务器、扩展应用程序、添加额外数据库服务器或将实例分布到多个位置时，Docker 使我们能够通过相同的简便流程部署应用程序。我们无需管理不同的主机和配置，就能有效地构建一个与平台无关的应用程序，并在任何地方运行相同的容器。

### 安装 Docker

要检查是否已经安装 Docker，请打开命令行并运行 docker -v。如果看到的版本号高于 20，则应该能够继续跟随本章中的示例。否则，你需要从 Docker Inc. 安装最新版本的 Docker。请访问 [*https://<wbr>www<wbr>.docker<wbr>.com<wbr>/products<wbr>/docker<wbr>-desktop<wbr>/*](https://www.docker.com/products/docker-desktop/)。然后选择适合你操作系统的 Docker 桌面安装程序并下载。执行应用程序，并在命令行中检查 Docker 版本号。它应该与你下载的版本一致。

### 创建 Docker 容器

Docker 有几个组件。运行 Docker 守护进程的物理或虚拟机器称为 *主机系统*。当你在本地开发应用程序时，主机是你的物理机器，而当你部署容器时，主机是运行应用程序的服务器。

我们在主机系统上使用 *Docker 守护进程服务* 来与 Docker 平台的所有组件进行交互。守护进程通过 API 提供 Docker 的功能，并且是安装在我们机器上的实际 Docker 应用程序。使用命令行中的 docker 命令访问守护进程。运行 docker --help 以显示所有可能的交互。

我们使用 Docker *容器* 来运行容器化的应用程序。这些容器是特定 Docker 镜像的运行实例，镜像是包含应用程序的工件。每个 Docker 镜像都依赖于一个 Dockerfile，该文件定义了 Docker 镜像的配置和内容。

#### 编写 Dockerfile

*Dockerfile* 是一个文本文件，包含我们设置 Docker 镜像所需的信息。它通常基于一些现有的基础镜像，例如一个基础的 Linux 系统，在此基础上我们安装了额外的软件或预配置的环境。例如，我们可能会使用一个包含 Node.js、MongoDB 和所有相关依赖项的 Linux 镜像。

通常，我们可以基于官方镜像进行构建。例如，清单 10-1 展示了我们用于容器化重构后的 Next.js 应用程序的基本 Dockerfile。Dockerfile 包含关键字和后续命令，我们在这里使用FROM关键字来选择官方的 Node.js Docker 镜像。在项目根目录下（与*package.json*文件相邻）创建一个名为*Dockerfile*的文件，并将清单 10-1 中的代码添加到其中。

```
FROM node:current

WORKDIR /home/node
COPY package.json package-lock.json /home/node/
EXPOSE 3000 
```

清单 10-1：用于典型 Node.js 应用程序的简单 Dockerfile

我们选择的镜像包含一个运行在 Debian 上的预配置 Node.js 系统。版本标签current提供了最新的 Node.js 版本；或者，我们可以在此处指定特定的版本号。因此，如果需要将应用程序锁定到特定的 Node.js 版本，这是实现的方法。你还可以使用更轻量级的node:current-slim镜像，它是一个精简的 Debian 发行版，仅包含运行 Node.js 所需的软件包。不过，由于我们需要 MongoDB 的内存服务器，因此我们选择了常规镜像。你可以在[*https://<wbr>hub<wbr>.docker<wbr>.com*](https://hub.docker.com)查看可用的镜像列表。在你的职业生涯中，你可能还会使用其他镜像，如 WordPress、MySQL、Redis、Apache 和 NGINX 的镜像。

最后，我们使用WORKDIR关键字将 Docker 镜像内的工作目录设置为用户的主目录。所有后续命令将会在该目录中执行。我们使用COPY关键字将*package .json*和*package-lock.json*文件添加到工作目录中。Node.js 应用程序默认运行在 3000 端口，因此我们使用EXPORT关键字选择 3000 端口用于 TCP 连接。这个连接将允许从容器外部访问应用程序。

#### 构建 Docker 镜像

要从 Dockerfile 创建 Docker 镜像，我们使用docker image build命令。在构建过程中，Docker 守护进程读取 Dockerfile 并执行其中定义的命令，下载和安装软件，复制本地文件到镜像中，并配置环境。运行以下命令来构建镜像：

```
$ **docker image build --tag nextjs:latest .**
[+] Building 11.9s (10/10) FINISHED
 => [internal] load build definition from **Dockerfile**                   0.1s
 => => transferring dockerfile: 136B                                   0.0s
 => [1/2] FROM docker.io/library/node:current-alpine@sha256:HASH 0.0s
 => [2/2] WORKDIR /home/node                                           0.0s
 => => naming to docker.io/library/ nextjs:latest 
```

--tag 标志为镜像命名为 nextjs 并将其版本设置为 latest。现在我们可以在后续的操作中轻松引用这个特定的镜像。我们在命令末尾使用一个句点 (.) 来设置构建上下文，将 docker build 命令的文件访问限制在当前目录。输出中，Docker 守护进程表示它已成功构建了标记的镜像。

现在，为了验证我们是否可以访问镜像，运行以下命令。这条命令会列出所有本地可用的 Docker 镜像：

```
$ **docker image ls**
REPOSITORY    TAG        IMAGE
nextjs        latest     98b28358e19a 
```

正如预期的那样，我们新创建的镜像有一个随机的 ID (98b28358e19a)，并被标记为 nextjs，且版本为 latest。Docker 守护进程可能还会显示额外的信息，比如镜像的大小和创建时间，暂时这些对我们来说并不重要。

Docker 提供了额外的命令来管理本地和远程镜像。你可以通过运行 docker image --help 查看所有可用的命令列表。例如，要从本地机器中删除一个现有的镜像，可以使用 docker image rm：

```
$ **docker image rm** **`<name:version or ID>`**
```

一段时间后，你会发现自己收集了许多未使用或过时的镜像版本，使用 docker image prune 删除它们以释放你机器上的空间是一种好习惯。

#### 从 Docker 容器提供应用服务

Docker 容器是 Docker 镜像的运行实例。你可以使用相同的 Docker 镜像启动多个容器，每个容器都有唯一的名称或 ID。一旦容器运行，你可以将本地文件同步到容器中。容器会监听一个暴露的 TCP 或 UDP 端口，你可以通过 SSH 连接到容器并在其中执行命令。

让我们将应用容器化。我们将从镜像启动 Docker 容器，将本地的 Next.js 文件映射到工作目录，暴露端口，最后启动 Next.js 开发服务器。我们可以通过 docker container run 完成这一切：

```
$ **docker container run \**
**--name nextjs_container \**
**--volume ~/nextjs_refactored/:/home/node/ \**
**--publish-all \**
**nextjs:latest npm run dev**
> refactored-app@0.1.0 dev
> next dev

ready - started server on 0.0.0.0:3000, url: http://localhost:3000
event - compiled client and server successfully in 10.9s (208 modules) 
```

乍一看，这个命令可能看起来很复杂，但一旦我们仔细看，你就会轻松理解它的作用。我们给它传递了几个标志，第一个是 --name，它为正在运行的容器分配一个唯一的名称。我们稍后会用这个名称来标识容器。

然后我们使用 --volume 标志来创建一个 Docker 卷。*卷*是容器之间共享数据的一种简单方式。Docker 本身管理它们，它们让我们将应用程序文件同步到容器内的 *home/node/* 目录。我们使用 source:destination 格式来定义卷，并根据你的文件结构，可能需要调整该文件夹的绝对路径。在这个例子中，我们将 */nextjs_refactored/* 从用户的主文件夹映射到容器内。

--publish-all 标志发布所有导出的端口，并将它们分配给主机系统上的随机端口。我们稍后使用 docker container ls 查看我们应用程序的端口。最后两个参数是直观的：nextjs:latest 指向我们希望用于容器的 Docker 镜像，而 npm run dev 启动 Next.js 开发服务器，像往常一样运行。控制台输出显示容器内部的 Node.js 应用程序正在运行并监听端口 3000。

#### 定位暴露的 Docker 端口

不幸的是，一旦我们尝试通过端口 3000 访问 Next.js 应用程序，浏览器会通知我们该端口不可访问；没有应用程序在该端口监听。问题是我们没有将暴露的 Docker 端口 3000 映射到主机的端口 3000。相反，我们使用了 --publish-all 标志，并将暴露的 Docker 端口分配给了一个随机端口。

让我们运行 docker container ls 查看所有运行中 Docker 容器的详细信息：

```
$ **docker container ls**
CONTAINER ID   IMAGE             PORTS                     NAMES
dff681898013   nextjs:latest     0.0.0.0:55000->3000/tcp   nextjs_container 
```

搜索我们为容器指定的名称，*nextjs_container*，并注意主机上的端口 55000 映射到 Docker 端口 3000。因此，我们可以在 *http://localhost:55000* 访问我们的应用程序。在浏览器中打开此 URL，你应该能看到 Next.js 应用程序。

如果你看一下 URL 地址栏，你会注意到我们用来访问应用程序的端口与前几章中使用的不同，因为它现在运行在 Docker 容器内部。尝试访问我们在之前章节中创建的所有页面和 API，然后再继续下一部分。

#### 与容器交互

你可以通过运行docker container --help查看所有与容器交互的 Docker 命令列表。然而，在大多数情况下，了解其中的一些命令就足够了。例如，使用exec可以在已经运行的 Docker 容器内执行命令。我们可以通过传递-it标志和 shell 路径（例如*/bin/sh*）来使用exec连接到容器内部的 shell。-i标志是--interactive的简写，而-t则启动一个伪终端。交互选项让我们能够与容器进行交互，而tty伪终端保持 Docker 容器运行，从而使我们能够与其实际互动：

```
$ **docker container exec -it** **`<container ID or name>`** **/bin/sh**
```

kill命令停止正在运行的 Docker 容器：

```
$ **docker container kill** **`<containerid or name>`**
```

我们可以通过名称或使用在本地运行容器列表中显示的容器 ID 来选择容器。

### 使用 Docker Compose 创建微服务

Docker 为我们提供了一种将应用程序分解为小而独立的单元——称为*微服务*的方法。微服务驱动的架构将应用程序拆分为一组自包含的服务，这些服务通过定义良好的 API 进行通信。这是一个相对较新的架构概念，最初在 2000 年代末至 2010 年代初期获得关注，当时 Docker 和其他可以更轻松分割和编排服务器资源的工具开始普及。这些工具构成了微服务架构的技术基础。

微服务有几个优点。首先，每个独立的服务只有一个单一的目的，这减少了其复杂性。因此，它更容易进行测试和维护。我们还可以单独部署微服务，启动同一微服务的多个实例来提高其性能，或完全替换它而不影响整个应用程序。与此对比的是传统的单体应用程序，其用户界面、中间件和数据存储都存在于一个由单一代码库构建的单一程序中。即使单体应用程序采用更模块化的方法，代码库也将它们紧密耦合，你无法轻松地替换其中的元素。

微服务的另一个特点是，专门的团队可以只负责单一服务及其代码库。这意味着他们可以根据每个服务选择适当的工具、框架和编程语言。另一方面，你通常会使用一种核心语言来编写单体应用程序。

现在你已经知道如何从零开始创建一个单一容器，我们将练习创建多个容器；每个容器将服务于应用程序的一个部分。使用微服务的一种方法是为前端创建一个服务，为后端创建另一个服务。我们将在第二部分中创建的 Food Finder 应用程序将使用这种结构。此方法的主要好处是它允许我们使用预配置的 MongoDB 镜像作为数据库。对于本章中的示例，我们将创建第二个服务来监视我们的天气服务，并在文件更改时立即重新运行其测试套件。为此，我们将使用 Docker Compose 接口，并在*docker-compose.yml*文件中定义我们的微服务架构。

#### 编写 docker-compose.yml 文件

我们在*docker-compose.yml*中定义所有服务，这是一个 YAML 格式的文本文件。该文件还为每个服务设置属性、依赖关系和卷。大多数属性类似于你在创建 Docker 镜像和容器时指定的命令行标志。创建文件并将清单 10-2 中的代码添加到应用程序的根文件夹中。

```
version: "3.0"
services:
    application:
        image:
            nextjs:latest
        ports:
            - "3000:3000"
        volumes:
            - ./:/home/node/
        command:
            "npm run dev"
    jest:
        image:
            nextjs:latest
        volumes:
            - ./:/home/node/
        command:
            "npx jest ./__tests__/mongoose/weather/services.test.ts --watchAll" 
```

清单 10-2：定义应用程序和 Jest 服务的基本 docker-compose.yml 文件

每个*docker-compose.yml*文件首先通过设置所使用的 Docker Compose 规范的版本来开始。根据版本的不同，我们可以使用不同的属性和值。然后，我们在services下定义每个服务作为单独的属性。如前所述，我们希望有两个服务：我们的 Next.js 应用程序运行在 3000 端口，和 Jest 服务，它监视我们在第八章中创建的*services .test.ts*文件，并在我们更改文件时立即重新运行测试。我们将 watch 命令限制为仅重新测试 services。这限制了练习的范围，但当然，如果你愿意，也可以重新运行所有测试。

每个服务的结构大致相同。首先，我们定义 Docker Compose 应从哪个镜像创建每个容器。这可以是官方发行版，也可以是本地构建的镜像。我们为两个服务都使用<sup class="SANS_TheSansMonoCd_W5Regular_11">nextjs</sup>镜像的<sup class="SANS_TheSansMonoCd_W5Regular_11">latest</sup>版本。然后，我们不使用<sup class="SANS_TheSansMonoCd_W5Regular_11">--publishAll</sup>标志，而是直接将<sup class="SANS_TheSansMonoCd_W5Regular_11">ports</sup>从 3000 映射到 3000。这样，我们就可以从主机的 3000 端口连接到应用程序的 3000 端口。

使用 volumes 属性，我们将主机系统中的文件和路径同步到容器中。这类似于我们在 docker run 命令中使用的映射方式，但与提供绝对路径不同，我们可以对源使用相对路径。在这里，我们将整个本地目录 *./* 映射到容器的工作目录 */home/node*。如前所述，我们可以在本地编辑 TypeScript 文件，容器中的应用程序始终使用文件的最新版本。

到目前为止，这些属性与我们在 docker run 命令中使用的命令行参数相匹配。现在我们添加 command 属性，用于指定每个容器在启动时执行的命令。对于应用服务，我们将使用常规的 npm run dev 命令启动 Next.js，而 Jest 服务则应该通过 npx 直接调用 Jest。提供测试文件的路径和 --watchAll 标志会导致 Jest 在源代码变化时重新运行测试。

#### 运行容器

使用 docker compose up 命令启动多容器应用。输出应该类似于这里所示：

```
$ **docker compose up**
 [+] Running 2/2
 ⠿ Container application-1     Created                       0.0s
 ⠿ Container jest-1  Recreated                               0.4s
Attaching to application-1, jest-1
application-1     |
application-1     | > refactored-app@0.1.0 dev
application-1     | > next dev
application-1     |
application-1     | ready - started server on 0.0.0.0:3000, URL:
application-1     | http://localhost:3000
jest-1            | PASS __tests__/mongoose/weather/services.test.ts
jest-1            |  the weather services
jest-1            |     API storeDocument
jest-1            |       ✓ returns true  (9 ms)
jest-1            |       ✓ passes the document to Model.create()  (6 ms)
jest-1            |     API findByZip
jest-1            |       ✓ returns true  (1 ms)
jest-1            |       ✓ passes the zip code to Model.findOne()  (1 ms)
jest-1            |     API updateByZip
jest-1            |       ✓ returns true  (1 ms)
jest-1            |       ✓ passes the zip code and the new data to
jest-1            |         Model.updateOne()  (1 ms)
jest-1            |     API deleteByZip
jest-1            |       ✓ returns true  (1 ms)
jest-1            |       ✓ passes the zip code Model.deleteOne()  (1 ms)
jest-1            |
jest-1            | Test Suites: 1 passed, 1 total
jest-1            | Tests:       8 passed, 8 total
jest-1            |    0 total
jest-1            | Time:        4.059 s
jest-1            | Ran all test suites matching
jest-1            |    /.\/__tests__\/mongoose\/weather\/services.test.ts/i. 
```

Docker 守护进程启动所有服务。一旦应用程序准备就绪，我们会看到来自 Express.js 服务器的状态消息，并且可以通过暴露的端口 3000 连接到它。同时，Jest 容器运行天气服务的测试并报告所有测试成功。

#### 重新运行测试

现在我们已经启动了 Docker 环境，接下来验证一下用于查找代码变化并重新运行测试的命令是否按预期工作。为此，我们需要修改源代码来触发 Jest。因此，我们打开 *mongoose/weather/service.ts* 文件，修改内容，添加一个空行，然后保存文件。Jest 应该会重新运行容器内的测试，正如你可以从 清单 10-3 的输出中看到的那样。

```
jest-1            | Ran all test suites matching
jest-1            |    /.\/__tests__\/mongoose\/weather\/services.test.ts/i.
jest-1            |
jest-1            | PASS __tests__/mongoose/weather/services.test.ts
jest-1            |   the weather services
jest-1            |     API storeDocument
jest-1            |       ✓ returns true  (9 ms)
jest-1            |       ✓ passes the document to Model.create()  (6 ms)
jest-1            |     API findByZip
jest-1            |       ✓ returns true  (1 ms)
jest-1            |       ✓ passes the zip code to Model.findOne()  (1 ms)
jest-1            |     API updateByZip
jest-1            |       ✓ returns true  (1 ms)
jest-1            |       ✓ passes the zip code and the new data to
jest-1            |         Model.updateOne()  (1 ms)
jest-1            |     API deleteByZip
jest-1            |       ✓ returns true  (1 ms)
jest-1            |       ✓ passes the zip code Model.deleteOne()  (1 ms)
jest-1            |
jest-1            | Test Suites: 1 passed, 1 total
jest-1            | Tests:       8 passed, 8 total
jest-1            |    0 total
jest-1            | Time:        7.089 s
jest-1            | Ran all test suites matching
jest-1            |    /.\/__tests__\/mongoose\/weather\/services.test.ts/i 
```

清单 10-3：使用 jest --watchAll 重新运行已更改文件的测试

所有测试仍然通过。连接到 *http://localhost:3000* 并验证你的浏览器是否仍然能够渲染应用程序。

#### 与 Docker Compose 交互

Docker Compose 提供了一个完整的接口，用于管理微服务应用程序。你可以通过运行 docker compose --help 查看可用的命令列表。以下是最重要的命令。

我们使用 docker compose ls 来获取所有本地运行的 Docker 应用程序列表，这些应用程序在 *docker-compose.yml* 文件中定义。该命令返回应用程序的名称和状态：

```
$ **docker compose ls**
```

要关闭当前目录中 *docker-compose.yml* 文件中定义的所有正在运行的服务，请运行 docker compose kill，该命令会向每个容器中的主进程发送 SIGKILL 命令：

```
$ **docker compose kill**
```

要以更优雅的 SIGTERM 命令关闭服务，请使用以下命令：

```
$ **docker compose down**
```

与强制关闭不同，这个命令会优雅地移除通过 docker compose up 创建的所有进程、容器、网络和卷。

### 总结

使用 Docker 容器化平台使得部署应用程序和使用微服务架构变得更加容易。本章介绍了 Docker 生态系统的基本构件：主机、Docker 守护进程、Dockerfile、镜像和容器。通过使用 Docker Compose 和 Docker 卷，你可以将应用程序分割成单个、独立的服务。

要充分释放 Docker 的潜力，请阅读官方教程 [*https://<wbr>docs<wbr>.docker<wbr>.com<wbr>/get<wbr>-started<wbr>/*](https://docs.docker.com/get-started/) 或者 [*https://<wbr>docker<wbr>-curriculum<wbr>.com*](https://docker-curriculum.com)。在下一章中，你将开始构建 Food Finder 应用程序。这个全栈 Web 应用程序将建立在你在之前所有章节中获得的知识基础上。
