

## 第十一章：11 设置 Docker 环境



![](img/Drop-image.jpg)

在本书的这一部分，你将通过运用迄今为止学到的知识，从零开始构建一个全栈应用程序。虽然前面的章节已经解释了部分技术栈，但剩下的章节将更详细地聚焦于代码部分。

本章描述了你将构建的应用程序，并引导你通过使用 Docker 配置环境。虽然我建议在开始编写代码之前阅读前面的章节，但唯一的真正要求是，在继续之前，你必须确保已经安装并运行 Docker。有关安装 Docker 的说明，请参考第十章。

> 注意

*你可以从* [`<wbr>www<wbr>.usemodernfullstack<wbr>.dev<wbr>/downloads<wbr>/food<wbr>-finder`](http://www.usemodernfullstack.dev/downloads/food-finder) *下载 Food Finder 应用程序的完整源代码，以及从* [`<wbr>www<wbr>.usemodernfullstack<wbr>.dev<wbr>/downloads<wbr>/assets`](http://www.usemodernfullstack.dev/downloads/assets) *下载仅包含所需资源的 ZIP 文件。*

### Food Finder 应用程序

Food Finder 应用程序展示了一系列餐馆及其位置。用户可以点击这些餐馆，以查看每个位置的更多细节。此外，用户还可以通过 OAuth 使用他们的 GitHub 账户登录该应用，以便维护一个位置的愿望清单。

在幕后，我们将使用 TypeScript 编写这个简单的单页应用程序。在设置本地环境后，我们将使用 Next.js、Mongoose 和 MongoDB 构建后端和中间件，并为其预填充初始数据。然后，我们将添加 GraphQL，以通过 API 层访问用户的愿望清单。为了构建前端，我们将运用对 React 组件、Next.js 页面和路由的知识。我们还将使用 *next-auth* 添加 OAuth 授权流，以便用户可以通过 GitHub 登录。最后，我们将使用 Jest 编写自动化测试，以验证应用程序的完整性和稳定性。

### 使用 Docker 构建本地环境

Docker 将开发环境与我们的本地机器解耦。我们将使用它为应用程序的每个部分创建自包含的服务。在 *docker-compose* 文件中，我们将添加一个服务来提供 MongoDB 数据库的后端，另一个服务来运行托管前端和中间件的 Next.js 应用程序。

要开始开发，创建一个新的空文件夹 *code*。该文件夹将作为应用程序的根目录，并包含 Food Finder 应用程序的所有代码。在本章后面，我们将使用 create-next-app 辅助命令向其中添加文件。

接下来，在这个根目录下创建一个空的*docker-compose.yml*文件和一个*.docker*文件夹。在文件中，我们将定义环境的两个服务，并存储我们创建容器所需的种子数据。

#### 后端容器

后端容器只提供应用的 MongoDB 实例。因此，我们可以使用官方的 MongoDB 镜像，Docker 可以自动从 Docker 注册表下载该镜像，而无需创建自定义的 Dockerfile。

##### 种植数据库

我们希望 MongoDB 以一个预填充的数据库启动，该数据库包含一组有效的初始数据集。这个过程称为数据库的种植，我们可以通过将种植脚本*seed-mongodb.js*复制到容器的*/docker-entrypoint-initdb.d/*目录中来自动化这个过程。MongoDB 镜像会在容器的*/data/db*目录没有数据时，在启动时执行这个文件夹中的脚本，并将其应用到<MONGO_INITDB_DATABASE>环境变量中定义的数据库上。

在*.docker*文件夹中创建一个新的文件夹*foodfinder-backend*，然后将之前下载的*assets.zip*文件中的*seed-mongodb.js*文件复制到新创建的文件夹中。种子文件的内容应该类似于列表 11-1。

```
db.locations.insert([
    {
        address: "6220 Avenue U",
        zipcode: "NY 11234",
        borough: "Brooklyn",
        cuisine: "Cafe",
        grade: "A",
        name: "The Roasted Bean",
        on_wishlist: [],
        location_id: "56018",
    },
`--snip--`
    {
        address: "405 Lexington Avenue",
        zipcode: "NY 10174",
        borough: "Manhattan",
        cuisine: "American",
        grade: "A",
        name: "The Diner At The Corner",
        on_wishlist: [],
        location_id: "63426",
    }
]); 
```

列表 11-1：seed-mongodb.js 文件

你可以看到，这个脚本直接与我们将在下一节中设置的 MongoDB 实例中的一个集合进行交互。我们使用 MongoDB 的 insert 方法，将文档填充到数据库的 location 集合中。请注意，我们使用的是*原生* MongoDB 驱动程序来插入文档，而不是使用 Mongoose。我们之所以这样做，是因为默认的 MongoDB Docker 镜像中没有安装 Mongoose，而插入文档是一个相对简单的任务。尽管我们没有使用 Mongoose 来种植数据库，但我们插入的文档需要与我们稍后用 Mongoose 定义的架构相匹配。

##### 创建后端服务

现在，我们可以在 Docker 设置中定义后端服务。将列表 11-2 中的代码添加到我们之前创建的空的*docker-compose.yml*文件中。

```
version: "3.0"
services:
    backend:
        container_name: foodfinder-backend
        image: mongo:latest
        restart: always
 environment:
            DB_NAME: foodfinder
            MONGO_INITDB_DATABASE: foodfinder
        ports:
            - 27017:27017
        volumes:
            - "./.docker/foodfinder-backend/seed-mongodb.js:
/docker-entrypoint-initdb.d/seed-mongodb.js"
            - mongodb_data_container:/data/db

volumes:
    mongodb_data_container: 
```

列表 11-2：带有后端服务的 docker-compose.yml 文件

我们首先定义容器的名称，以便后续可以轻松引用它。如前所述，我们使用官方 MongoDB 镜像的最新版本，并指定如果容器停止，它应始终重新启动。接下来，我们使用环境变量来定义我们将与 MongoDB 一起使用的集合。我们定义了两个变量：DB_NAME指向我们将与 Mongoose 一起使用的集合，MONGO_INITDB_DATABASE指向种子脚本。*/docker-entrypoint-initdb.d/*中的脚本默认使用这个后者集合。

我们希望脚本填充应用程序的数据库，因此我们将两个变量设置为相同的名称foodfinder，从而为我们的 Mongoose 模型提供了一个预填充的数据库。

然后我们将容器的内部端口 27017 映射并暴露到主机的端口 27017，以便 MongoDB 实例可以通过应用程序访问，地址为*mongodb://backend:27017/foodfinder*。请注意，连接字符串中包含了服务名称、端口和数据库。稍后，我们将这个连接字符串存储在环境变量中，并用它从中间件连接到数据库。最后，我们将种子脚本映射并复制到设置位置，并将数据库数据从*/data/db*保存到 Docker 卷*mongodb_data_container*中。因为我们希望将字符串拆分到两行，所以需要根据 YAML 约定将其包裹在双引号中(")。

现在使用docker compose up完成 Docker 设置：

```
$ **docker compose up**
[+] Running 2/2
 ⠿ Network foodfinder_default                      Created                 0.1s
 ⠿ Container foodfinder-backend                    Created                 0.3s
Attaching to foodfinder-backend

foodfinder-backend  | /usr/local/bin/docker-entrypoint.sh: running /docker
                    /entrypoint-initdb.d/seed-mongodb.js 
```

输出显示 Docker 守护进程成功创建了foodfinder-backend容器，并且在启动期间执行了种子脚本。我们通过在*docker-compose*文件中添加几行代码，将 MongoDB 添加到我们的项目中，而不必经历安装和维护 MongoDB 的麻烦，或寻找免费的或低成本的云实例。

使用 CRTL-C 停止容器，并通过docker compose down将其移除：

```
$ **docker compose down**
[+] Running 2/2
 ⠿ Container foodfinder-backend                     Removed                 0.0s
 ⠿ Network foodfinder_default                       Removed 
```

现在我们可以添加前端容器了。

#### 前端容器

现在我们将创建前端和中间件的容器化基础设施。我们的做法是使用create-next-app来搭建 Next.js 应用程序，正如我们在第五章中所做的那样，依赖官方的 Node.js Docker 镜像，将应用程序与任何本地 Node.js 安装解耦。

由于我们将所有与 Node.js 相关的命令都在该容器内执行，从技术上讲，我们甚至不需要在本地机器上安装 Node.js；也不必确保我们使用的 Node.js 版本符合 Next.js 的要求。此外，npm 可能会安装优化过的操作系统相关的包，因此通过在容器内使用 npm，我们确保了 npm 安装适用于 Linux 的正确版本。

尽管如此，我们仍然希望 Docker 同步 Node.js *modules* 文件夹到我们的本地系统。这将允许我们的 IDE 自动使用已安装的依赖项，例如 TypeScript 编译器和 ESLint。让我们从创建一个最小的 Dockerfile 开始。

##### 创建应用程序服务

我们通过将 列表 11-3 中的代码添加到项目的 *docker-compose.yml* 文件的 services 属性中，将前端和中间件服务结合到我们的 Docker 设置中。

```
`--snip--`
services:

    application:
        container_name: foodfinder-application
        image: node:lts-alpine
        ports:
            - "3000:3000"
        volumes:
            - ./code:/home/node/code
        working_dir: /home/node/code/
        depends_on:
            - backend
        environment:
            - HOST=0.0.0.0
            - CHOKIDAR_USEPOLLING=true
            - CHOKIDAR_INTERVAL=100
        tty: true
    backend:
`--snip--` 
```

列表 11-3：带有后端和应用程序服务的 docker-compose.yml 文件

Food Finder 应用程序的服务结构与后端服务的结构相同。首先，我们设置容器的名称。然后，我们定义为该特定服务使用的镜像。虽然后端服务使用了官方的 MongoDB 镜像，但我们现在使用的是官方的 Node.js 镜像，并且运行的是当前 LTS 版本，基于 Alpine Linux 的轻量级 Linux 发行版，这种发行版比基于 Debian 的镜像消耗更少的内存。

然后，我们暴露并映射 3000 端口，使应用程序可以通过 *http://localhost:3000* 访问，并将本地应用程序的代码目录映射到容器中。接下来，我们将工作目录设置为 *code* 目录。我们指定容器需要一个正在运行的后端服务，因为 Next.js 应用程序需要与 MongoDB 实例保持有效连接。此外，我们还添加了环境变量。特别地，chokidar 支持 Next.js 代码的热重载。最后，将 tty 属性设置为 true 使容器提供交互式 shell，而不是关闭容器。我们需要这个 shell 来在容器内执行命令。

##### 安装 Next.js

在这两个服务都就绪后，我们现在可以在容器内安装 Next.js。为此，我们需要使用 docker compose up 启动容器：

```
$ **docker compose up**

[+] Running 3/3
 ⠿ Network foodfinder_default                      Created                 0.1s
 ⠿ Container foodfinder-backend                    Created                 0.3s
 ⠿ Container foodfinder-application                Created                 0.3s
Attaching to foodfinder-application, foodfinder-backend
`--snip--`
foodfinder-application  | Welcome to Node.js ...
`--snip--` 
```

将这个命令行输出与之前的 docker compose up 输出进行对比。你应该能看到应用程序容器已成功启动，并运行一个 Node.js 交互式 shell。

现在我们可以使用 docker exec 在正在运行的容器内执行命令。这样做有两个主要优点。首先，我们在本地机器上不需要任何特定版本的 Node.js（甚至不需要任何版本）。其次，我们在 Node.js Linux Alpine 镜像中运行 Node.js 应用程序和 npm 命令，这样依赖项就会针对 Alpine 优化，而不是针对我们的主机系统。

要在容器内运行 npm 命令，可以使用 docker exec -it foodfinder-application 后跟要运行的命令。Docker 守护进程会连接到容器内的终端，并在应用程序容器的工作目录 */home/node/code* 中执行提供的命令，这个目录是我们之前设置的。让我们使用在 第五章 中讨论的 npx 命令在那里安装 Next.js 应用程序：

```
/home/node/code# **docker exec -it foodfinder-application \**
**npx create-next-app@latest foodfinder-application \**
**--typescript --use-npm**
Need to install the following packages:
  create-next-app
Ok to proceed? (y)
✔ Would you like to use ESLint with this project? ... No / Yes
Creating a new Next.js app in /home/node/code/foodfinder-application.

Success! Created foodfinder-application at /home/node/code/foodfinder-application 
```

我们将项目名称设置为 *foodfinder-application* 并接受默认设置。其余的输出应该对你来说是熟悉的。

一旦脚手架搭建完成，我们可以使用 npm run dev 启动 Next.js 应用程序。如果你在浏览器中访问 *http://localhost:3000*，应该能看到熟悉的 Next.js 启动画面。*foodfinder-application* 文件夹应映射到本地的 *code* 文件夹，这样我们就可以在本地编辑与 Next.js 相关的文件。

##### 调整应用程序服务以支持重启

目前，连接到应用程序容器需要在每次通过 docker compose up 重启后运行 docker exec，然后手动调用 npm run dev。让我们对应用程序服务进行两项小调整，以实现更便捷的设置。修改文件，使其与 示例 11-4 匹配。

```
`--snip--`
services:
`--snip--`
    **application:**
`--snip--`
        volumes:
            - ./code:/home/node/code
        working_dir: /home/node/code/**foodfinder-application**
`--snip--`
 **command: "npm run dev"**
`--snip--` 
```

示例 11-4：用于自动启动 Next.js 的 docker-compose.yml 文件

首先，修改 working_dir 属性。因为我们正在处理 Next.js，所以我们将其设置为 Next.js 应用程序的根文件夹 */home/node/code/foodfinder-application*，该文件夹包含 *package.json* 文件。然后，我们添加 command 属性，值为 npm run dev。通过这两个修改，每次调用 docker compose up 时，Next.js 应用程序应立即启动。尝试使用 docker compose up 启动容器；控制台输出应显示 Next.js 正在运行，并且可以通过 *http://localhost:3000* 访问：

```
$ **docker compose up**
[+] Running 3/3
 ⠿ Network foodfinder_default                      Created    0.1s
 ⠿ Container foodfinder-backend                    Created    0.3s
 ⠿ Container foodfinder-application                Created    0.3s
Attaching to foodfinder-application, foodfinder-backend
foodfinder-application  |
foodfinder-application  | > foodfinder-application@0.1.0 dev
foodfinder-application  | > next dev
foodfinder-application  |
foodfinder-application  | ready - started server on 0.0.0.0:3000,
foodfinder-application  | url: foodfinder-application  | http://localhost:3000
foodfinder-application  | info  - Loaded env from /home/node/code/foodfinder-
foodfinder-application  | application/.env.local 
```

如果你在浏览器中访问 *http://localhost:3000*，你应该会看到 Next.js 启动画面，而不需要手动启动 Next.js 应用程序。

请注意，如果你在 Linux 或 macOS 上使用非管理员或 root 用户，你需要调整应用服务和启动命令。因为 Docker 守护进程默认以 root 用户身份运行，它创建的所有文件都需要 root 权限。你的常规用户没有这些权限，无法访问这些文件。为避免这些问题，请修改设置，使得 Docker 守护进程将所有权转移给你的用户。首先，将 列表 11-5 中的代码添加到 *docker-compose* 文件中的应用服务。

```
services:
`--snip--`
    **application:**
`--snip--`
 **user: ${MY_USER}**
`--snip--` 
```

列表 11-5：带有 user 属性的 docker-compose.yml 文件

我们将 user 属性添加到 application 服务，并使用环境变量 MY_USER 作为该属性的值。然后我们修改 docker compose 命令，使得在启动时将当前用户的用户 ID 和组 ID 添加到该环境变量中。我们使用以下代码，而不是直接调用 docker compose up：

```
MY_USER=$(id -u):$(id -g) docker compose up
```

我们使用 id 辅助程序将用户 ID 和组 ID 以 userid:groupid 格式保存到我们的环境变量中，*docker-compose* 文件随后会读取这个变量。-u 标志返回用户 ID，-g 标志返回组 ID。

### 总结

我们已经使用 Docker 容器设置好了本地开发环境。通过我们在本章中创建的 *docker-compose.yml* 文件，我们将应用程序开发与本地主机系统解耦。现在我们可以更换主机系统，并确保 Food Finder 应用始终使用相同的 Node.js 版本。此外，我们还添加了一个运行 MongoDB 服务器的容器，在下一章我们将连接该容器并实现应用程序的中间件。
