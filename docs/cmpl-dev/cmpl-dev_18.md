

## 第十六章：16 在 DOCKER 中运行自动化测试



![](img/Drop-image.jpg)

在这短短的最后一章中，您将编写几个自动化测试来验证 Food Finder 应用程序的状态。然后，您将配置一个 Docker 服务来持续运行这些测试。

我们将重点评估应用程序的头部，使用快照测试并模拟用户会话。我们不会为其他组件、我们的中间件、服务或 API 创建测试。然而，我鼓励您自行构建这些。可以尝试使用基于浏览器的端到端测试，借助像 Cypress 或 Playwright 这样的专用框架来测试整个页面。您可以在 [*https://<wbr>nextjs<wbr>.org<wbr>/docs<wbr>/testing*](https://nextjs.org/docs/testing) 上找到这两个框架的安装说明和示例。

### 将 Jest 添加到项目中

使用 npm 安装 Jest 库：

```
$ **docker exec -it foodfinder-application npm install --save-dev jest \**
**jest-environment-jsdom @testing-library/react @testing-library/jest-dom** 
```

接下来，通过创建一个名为 *jest.config.js* 的新文件并包含列表 16-1 中的代码，配置 Jest 与我们的 Next.js 设置兼容。将文件保存在应用程序的根文件夹中。

```
const nextJest = require("next/jest");

const createJestConfig = nextJest({
    dir: "./",
});

const customJestConfig = {
    moduleDirectories: ["node_modules", "<rootDir>/"],
    testEnvironment: "jest-environment-jsdom",
};

module.exports = createJestConfig(customJestConfig); 
```

列表 16-1：jest.config.js 文件

我们利用内置的 Next.js Jest 配置，因此需要将项目的基本目录配置为加载 *config* 和 *.env* 文件到测试环境中。然后设置模块目录的位置和全局测试环境。这里使用全局设置，因为我们的快照测试将需要一个 DOM 环境。

现在我们希望能够使用 npm 命令运行测试。因此，将列表 16-2 中的两个命令添加到项目的 scripts 属性的 *package.json* 文件中。

```
 "test": "jest ",
    "testWatch": "jest --watchAll" 
```

列表 16-2：添加到 package.json 文件的 scripts 属性中的两个命令

第一个命令一次性执行所有可用的测试，第二个命令则持续监视文件更改，并在检测到更改时重新运行测试。

### 设置 Docker

要使用 Docker 运行测试，请向 *docker-compose.yml* 中添加另一个使用 Node.js 镜像的服务。在启动时，此服务将运行 npm run testWatch，这是我们刚刚定义的命令。通过这种方式，我们将持续运行测试，并即时获取有关应用程序状态的反馈。修改文件以匹配列表 16-3 中的代码。

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

    application:
        container_name: foodfinder-application
        image: node:lts-alpine
        working_dir: /home/node/code/foodfinder-application
        ports:
            - "3000:3000"
        volumes:
            - ./code:/home/node/code
        depends_on:
            - backend
        environment:
            - HOST=0.0.0.0
            - CHOKIDAR_USEPOLLING=true
            - CHOKIDAR_INTERVAL=100
        tty: true
        command: "npm run dev"

    **jest:**
 **container_name: foodfinder-jest**
 **image: node:lts-alpine**
 **working_dir: /home/node/code/foodfinder-application**
 **volumes:**
 **- ./code:/home/node/code**
 **depends_on:**
 **- backend**
 **- application**
 **environment:**
 **- NODE_ENV=test**
 **tty: true**
 **command: "npm run testWatch"**

volumes:
    mongodb_data_container: 
```

列表 16-3：包含 jest 服务的修改后的 docker-compose.yml 文件

我们的小服务，名为*jest*，使用了我们之前用过的官方 Node.js Alpine 镜像。我们设置了工作目录，并使用 volumes 属性将我们的代码也提供给这个容器。与应用程序服务不同，*jest* 服务将 Node.js 环境设置为 test，并运行 testWatch 命令。

重启 Docker 容器；控制台应显示 Jest 正在监视我们的文件。

### 为 Header 元素编写快照测试

如同在第八章中一样，在应用程序的根目录中创建 *__tests__* 文件夹来存放我们的测试文件。然后添加包含列表 16-4 中代码的 *header.snapshot.test.tsx* 文件。

```
import {act, render} from "@testing-library/react";
import {useSession} from "next-auth/react";
import Header from "components/header";

jest.mock("next-auth/react");
describe("The Header component", () => {
    it("renders unchanged when logged out", async () => {
        (useSession as jest.Mock).mockReturnValueOnce({
            data: {user: {}},
            status: "unauthenticated",
        });
        let container: HTMLElement | undefined = undefined;
        await act(async () => {
            container = render(<Header />).container;
        });
        expect(container).toMatchSnapshot();
    });

    it("renders unchanged when logged in", async () => {
        (useSession as jest.Mock).mockReturnValueOnce({
            data: {
                user: {
                    name: "test user",
                    fdlst_private_userId: "rndmusr",
                },
            },
            status: "authenticated",
        });
        let container: HTMLElement | undefined = undefined;
        await act(async () => {
            container = render(<Header />).container;
        });
        expect(container).toMatchSnapshot();
    });
}); 
```

列表 16-4：__tests__/header.snapshot.test.tsx 文件

这个测试应该类似于你在第八章中编写的那些。注意，我们从*next-auth/react*导入了 useSession 钩子，然后使用 jest.mock 在每个测试的*安排*步骤中替换它。通过用返回状态的模拟会话替换原会话，我们可以验证标题组件在已登录和未登录用户状态下的行为是否符合预期。我们通过使用安排、执行和断言模式来描述 Header 组件的测试套件，并验证渲染的组件是否与存储的快照匹配。

第一个测试用例使用空会话和*未经验证*的状态来呈现未登录状态下的标题。第二个测试用例使用包含最少数据的会话，并将用户状态设置为*已验证*。这样我们就可以验证，现有会话显示的用户界面与空会话显示的界面不同。

如果你编写了额外的测试，请确保将它们添加到*__tests__* 文件夹中。

### 总结

你已成功添加了一些简单的快照测试，以验证 Food Finder 应用程序按预期工作。通过添加额外的 Docker 服务，你可以持续验证后续开发不会破坏应用程序。

恭喜！你已经成功地创建了第一个全栈应用程序，使用了 TypeScript、React、Next.js、Mongoose 和 MongoDB。你还使用 Docker 将应用程序容器化，并用 Jest 进行测试。通过本书及其练习，你为自己作为全栈开发者的职业生涯奠定了基础。
