

## 第七章：7 MONGODB 和 MONGOOSE



![](img/Drop-image.jpg)

大多数应用程序依赖于数据库管理系统，简称*数据库*，来组织和授予对数据集集合的访问权限。在本章中，你将使用 MongoDB 非关系型数据库和 Mongoose 作为其附带的对象映射工具。

因为 MongoDB 以 JSON 格式返回数据，并使用 JavaScript 进行数据库查询，它为全栈 JavaScript 开发人员提供了自然的选择。在接下来的章节中，你将学习如何创建一个 Mongoose 模型，通过它你可以查询数据库，简化与 MongoDB 的交互，并编写中间件将前端与后端数据库连接起来。你还将编写服务函数来实现对数据库的四个 CRUD 操作。

在 第七章练习（第 125 页）中，你将为你在第六章中创建的 GraphQL API 添加一个数据库，替换当前的静态数据存储。

### 应用程序如何使用数据库和对象关系映射器

一个应用程序需要数据库来存储和操作数据。在本书的前面部分，我们的应用程序的 API 仅返回了预定义的数据集，这些数据集存储在文件中，且无法更改。我们使用请求中的参数来添加到数据集中，但不能在不同的 API 调用之间存储数据（这被称为*数据持久化*）。例如，如果我们想要更新应用程序的天气信息，我们需要一个数据库来持久化数据，以便下一个 API 调用可以读取它。在全栈开发中，我们通常使用数据库来存储与用户相关的数据。另一个数据库的例子是你的电子邮件客户端用来存储你消息的数据库。

为了使用数据库，我们首先需要连接到它并进行身份验证。一旦我们获得了数据访问权限，就可以执行查询来请求特定的数据集。查询返回的结果包含数据，我们的应用程序可以展示这些数据或以其他方式使用它。每一步如何实现，取决于具体使用的数据库。

使用数据库的 API 查询数据往往会显得笨拙，因为它通常需要大量的样板代码，即便只是建立和维护连接。因此，我们通常使用*对象关系映射器*或*对象数据建模工具*，通过抽象一些细节来简化与数据库的交互。例如，MongoDB 的 Mongoose 对象数据建模工具为我们处理数据库连接，避免了我们在每次交互时都需要检查数据库连接是否开启。

Mongoose 还简化了 MongoDB 在独立数据库服务器上运行的处理方式。使用分布式系统需要进行异步调用，这点你在第二章中已经学过。使用 Mongoose，我们可以通过面向对象的 async/await 接口来访问数据，而不需要使用繁琐的回调函数。

此外，MongoDB 是无模式的；它不要求我们预定义并严格遵守模式。虽然这种灵活性很方便，但它也是常见错误的来源，尤其是在大型应用程序或开发者团队不断变动的项目中。在第三章中，我们讨论了通过使用 TypeScript 为 JavaScript 添加类型的好处。Mongoose 通过类似的方式对 MongoDB 的数据模型进行类型化并验证其完整性，正如你将在“定义 Mongoose 模型”（第 118 页）中发现的那样。

### 关系型与非关系型数据库

数据库可以以多种方式组织数据，这些方式主要分为两大类：关系型和非关系型。*关系型数据库*，如 MySQL 和 PostgreSQL，数据存储在一个或多个表中。你可以把这些数据库想象成类似于 Excel 电子表格。与 Excel 类似，每个表都有一个唯一名称，并包含列和行。列定义所有存储在该列中的数据的属性，如数据类型，而行包含实际的数据集，每行都有一个唯一 ID。关系型数据库使用某种变体的结构化查询语言（SQL）来进行数据库操作。

MongoDB 是一个*非关系型数据库*。与传统的关系型数据库不同，它以 JSON 文档的形式存储数据，而不是以表格形式存储，并且不使用 SQL。非关系型数据库有时被称为*NoSQL*，它们可以以多种不同格式存储数据。例如，流行的 NoSQL 数据库 Redis 和 Memcached 使用键值存储，这使它们具有高性能和易于扩展的特点。因此，它们常被用作内存缓存。另一个 NoSQL 数据库，Neo4j，是一个*图形数据库*，它使用图论将数据存储为节点，这个概念我们在第六章中提到过。这些只是非关系型数据库的一些例子。

MongoDB 是最广泛使用的*文档数据库*；它不是通过表格、行和列来组织数据，而是通过集合、文档和字段。*字段*是数据库中最小的单位，它定义数据类型和其他属性，并包含实际数据。你可以将其视为 SQL 表中的列的粗略等价物。*文档*由字段构成，类似于 SQL 表中的行。我们有时称它们为记录，MongoDB 使用 *BSON*，即 JSON 对象的二进制表示，来存储它们。*集合*大致等同于 SQL 表，但它不是由行和列组成，而是聚合了文档。

由于非关系型数据库可以以不同格式存储数据，因此每个数据库使用特定的、优化过的查询语言进行 CRUD 操作。这些低级 API 关注的是访问和操作数据，而不一定是开发者体验。相比之下，面向对象的关系映射工具提供了高级抽象，拥有清晰简化的查询语言接口。因此，虽然 MongoDB 有 MongoDB 查询语言（MQL），我们将使用 Mongoose 来访问它。

### 设置 MongoDB 和 Mongoose

在开始使用 MongoDB 和 Mongoose 之前，必须将它们添加到您的示例项目中。为了简化，我们将使用 MongoDB 的内存实现，而不是在机器上安装和维护真实的数据库服务器。这对于测试本章示例是合适的，但不适用于部署实际的应用程序，因为它在重启时不会持久化数据。当您在第二部分构建食品查找应用时，您将获得设置真实 MongoDB 服务器的经验。第十一章将展示如何使用预构建的 Docker 容器，该容器包含 MongoDB 服务器。

在第六章的重构版 Next.js 应用的根目录下运行此命令：

```
$ **npm install mongodb-memory-server mongoose**
```

然后，在根目录中创建两个新文件夹，位于 *package.json* 文件旁边：一个用于 Mongoose 代码，命名为 *mongoose*，并在其中创建子文件夹 *weather*；另一个命名为 *middleware*，用于存放所需的中间件。

### 定义 Mongoose 模型

为了验证我们数据的完整性，我们必须创建一个基于架构的 Mongoose *模型*，它充当与数据库中 MongoDB 集合的直接接口。所有与数据库的交互将通过该模型进行。然而，在创建模型之前，我们需要先创建架构本身，架构定义了数据库数据的结构，并将 Mongoose 实例映射到集合中的文档。

我们的 Mongoose 架构将与第六章中为 GraphQL API 创建的架构相匹配。这是因为我们将在第 125 页的练习 7 中将 GraphQL API 连接到数据库，从而允许我们用从数据库查询的数据集替换静态 JSON 对象。

#### 接口

在用 TypeScript 编写 Mongoose 模型和架构之前，让我们先声明一个 TypeScript 接口。如果没有匹配的接口，我们将无法为 TSC 类型化模型或架构，代码也无法编译。将列表 7-1 中显示的代码粘贴到 *mongoose/weather/interface.ts* 文件中。

```
export declare interface WeatherInterface {
    zip: string;
    weather: string;
    tempC: string;
    tempF: string;
    friends: string[];
}; 
```

列表 7-1：Mongoose 天气模型的接口

这段代码是一个常规的 TypeScript 接口，属性与 GraphQL 和 Mongoose 架构相匹配。

#### 架构

列表 7-2 展示了 Mongoose 架构。它的顶层属性代表文档中的字段。每个字段都有一个类型和一个标志，指示该字段是否是必需的。字段还可以具有其他可选属性，例如自定义或内建的验证器。这里我们使用了内建的 required 验证器；其他常见的内建验证器包括用于字符串的 minlength 和 maxlength，以及用于数字的 min 和 max。将代码添加到 *mongoose/weather/schema.ts* 文件中。

```
import {Schema} from "mongoose";
import {WeatherInterface} from "./interface";

export const WeatherSchema = new Schema<WeatherInterface>({
    zip: {
        type: "String",
        required: true,
    },
    weather: {
        type: "String",
        required: true,
    },
    tempC: {
        type: "String",
        required: true,
    },
    tempF: {
        type: "String",
        required: true,
    },
    friends: {
        type: ["String"],
        required: true,
    },
}); 
```

列表 7-2：Mongoose 天气模型的架构

我们使用传递给架构构造函数的对象来创建架构，并将 WeatherInterface 设置为其 SchemaType。因此，我们从 *mongoose* 包中导入 Schema 函数，并导入之前创建的接口。

类似于 TypeScript 为 JavaScript 添加自定义类型，Mongoose 会将每个属性转换为其关联的 *SchemaType*，该类型提供模型的配置。可用的类型包括内建的 JavaScript 类型，如 Array、Boolean、Date、Number 和 String，以及自定义类型，如 Buffer 和 ObjectId，后者指的是 Mongoose 在创建每个文档时添加的默认唯一 _id 属性。这类似于你可能知道的关系数据库中的主键。

我们在第六章中创建的天气 API 返回了一个包含四个属性的对象：zip、weather、tempC和tempF，每个属性的值都是字符串。此外，friends 属性中包含一个字符串数组。在这个架构中，我们定义了相同的属性，然后导出该架构。

#### 模型

既然我们已经有了一个模式，现在可以创建 Mongoose 模型了。这个模式的封装器将提供对集合中 MongoDB 文档的访问，以执行所有的 CRUD 操作。我们在 *mongoose/weather/model.ts* 文件中编写模型，其代码位于清单 7-3。请记住，我们还没有将其连接到服务器上的 MongoDB 数据库。

```
import mongoose, {model} from "mongoose";
import {WeatherInterface} from "./interface";
import {WeatherSchema} from "./schema";

export default mongoose.models.Weather ||
    model<WeatherInterface>("Weather", WeatherSchema); 
```

清单 7-3：Mongoose 天气模型

首先，我们导入 Mongoose 模块和来自 *mongoose* 包的模型构造函数，以及我们之前创建的接口和模式。然后，我们设置 Weather 模型，使用 WeatherInterface 来为其指定类型。我们传入两个参数：模型的名称 Weather 和定义模型内部数据结构的模式。Mongoose 会将新创建的模型绑定到我们 MongoDB 实例的集合上。Weathers 集合位于 *Weather* 数据库中，两个都会由 Mongoose 创建。请注意，在创建新模型之前，我们需要检查 mongoose.models 上是否已存在 Weather 模型；否则，Mongoose 将抛出错误。我们导出该模型，以便在后续的模块中使用它。

#### 数据库连接中间件

本书到目前为止，我们多次提到全栈开发涵盖了应用程序的前端、后端和中间件，后者通常也被称为“应用程序粘合剂”。现在是时候创建我们的第一个专用中间件了。

这个中间件将打开与数据库的连接，然后使用 Mongoose 的异步辅助函数保持该连接。接下来，它将把 Mongoose 的模型映射到 MongoDB 集合，以便我们通过 Mongoose 访问它们。方便的是，连接助手将缓冲操作，并在必要时重新连接到数据库，因此我们不需要自己处理连接问题。将代码从清单 7-4 粘贴到 *middleware/db-connect.ts* 文件中。

```
import mongoose from "mongoose";
import {MongoMemoryServer} from "mongodb-memory-server";

async function dbConnect(): Promise<any | String> {
    const mongoServer = await MongoMemoryServer.create();
    const MONGOIO_URI = mongoServer.getUri();
    await mongoose.disconnect();
    await mongoose.connect(MONGOIO_URI, {
        dbName: "Weather"
    });
}

export default dbConnect; 
```

清单 7-4：Mongoose 中间件

我们导入 *mongoose* 包和 *mongodb-memory-server* 数据库。我们定义并导出的异步函数 dbConnect 通过 mongoose.connect 函数管理与数据库服务器的连接。我们创建一个 MongoMemoryServer 实例，将数据保存在内存中，而不是使用真实的数据库服务器，如前所述。然后，我们将连接字符串存储在常量 MONGOIO_URI 中。由于我们使用的是内存服务器，这个字符串是动态的，但对于远程数据库，它将是一个表示数据库服务器地址的静态字符串。接着，我们关闭所有现有的连接，并使用 Mongoose 打开一个新连接。Mongoose 模型已经映射并可用，因此我们已经准备好执行我们的第一个查询。

### 查询数据库

现在是编写数据库查询的时候了。你应该将这些查询提取为服务，而不是在应用程序代码中随意分散这些查询或直接在 GraphQL 解析器中编写它们。

*服务*是执行实际 CRUD 操作并返回结果的函数。每个 GraphQL 解析器可以调用一个服务函数，所有的数据库访问都应通过这些函数进行。此外，每个服务应只负责一个特定的 CRUD 操作。Mongoose 会自动排队命令并执行它们，保持连接，并在与数据库建立连接后立即处理队列。

本节介绍了服务功能和基本的 Mongoose 命令。然而，这并不是一个完整的参考。当你开始在自己的项目中使用 Mongoose 时，请查阅 Mongoose 文档以获取所有需要的功能。

#### 创建文档

第一个也是最基本的操作是“创建”操作。它被方便地称为 mongoose.create，幸运的是，我们可以用它来创建和更新数据集。这是因为如果数据条目尚不存在，Mongoose 会自动创建一个新的数据库条目或文档。因此，我们无需先检查数据集是否存在，然后再有条件地创建它再进行更新。

列表 7-5 展示了一个基本的服务函数实现，该函数将数据集存储到数据库中。将代码放入 *mongoose/weather/services.ts* 文件中。

```
import WeatherModel from "./model";
import {WeatherInterface} from "./interface";

export async function storeDocument(doc: WeatherInterface): Promise<boolean> {
    try {
        await WeatherModel.create(doc);
    } catch (error) {
        return false;
    }
    return true;
} 
```

列表 7-5：通过 Mongoose 创建文档

为了存储文档，我们创建并导出异步函数 storeDocument，该函数以数据集作为参数。这里我们将其类型设为 WeatherInterface。然后，我们在模型上调用 create 函数，并将数据集传递给它。该函数将创建并插入文档到 WeatherModel 中，该模型是 MongoDB 实例中的天气集合。最后，它返回一个布尔值，表示操作的状态。

#### 读取文档

为了实现“读取”操作，我们通过 Mongoose 的 findOne 函数查询 MongoDB。它接受一个参数——一个包含要查找属性的对象，并返回第一个匹配项。通过 清单 7-6 中的代码，扩展 *mongoose/weather/services.ts* 文件。它定义了一个 findByZip 函数，用于查找并返回 Weathers 集合中第一个 zip 属性与传递给函数的 ZIP 代码匹配的文档。

```
export async function findByZip(
    paramZip: string
): Promise<Array<WeatherInterface> | null> {
    try {
        return await WeatherModel.findOne({zip: paramZip});
    } catch (err) {
        console.log(err);
    }
    return [];
} 
```

清单 7-6：通过 Mongoose 读取数据

我们向 *services.ts* 文件中的服务添加并导出异步函数 readByZip。该函数接受一个字符串参数——ZIP 代码，并返回一个包含文档的数组或一个空数组。在新的服务函数内部，我们在模型上调用 Mongoose 的 findOne 函数，并传递一个过滤对象，查找其 zip 字段与参数值匹配的文档。最后，函数返回结果或 null。

#### 更新文档

我们提到过，可以使用 create 函数来更新文档。然而，也有一个专门用于此任务的 API：updateOne。它接受两个参数。第一个是过滤对象，类似于我们在 findOne 中使用的过滤器，第二个是包含新值的对象。你可以将 updateOne 看作是 “find” 和 “create” 函数的结合。通过 清单 7-7 中的代码，扩展 *mongoose/weather/services.ts* 文件。

```
export async function updateByZip(
    paramZip: string,
    newData: WeatherInterface
): Promise<boolean> {
    try {
        await WeatherModel.updateOne({zip: paramZip}, newData);
        return true;
    } catch (err) {
        console.log(err);
    }
    return false;
} 
```

清单 7-7：通过 Mongoose 更新数据

我们添加到服务中的 updateByZip 函数接受两个参数。第一个是字符串 paramZip，它是我们用来查询要更新的文档的邮政编码。第二个参数是新的数据集，我们将其类型定义为 WeatherInterface。我们在模型上调用 Mongoose 的 updateOne 函数，传入一个过滤器对象和最新的数据。该函数应返回一个布尔值，表示操作状态。

#### 删除文档

我们需要实现的最后一个 CRUD 操作是一个删除文档的服务。为此，我们使用 Mongoose 的 deleteOne 函数，并将 Listing 7-8 中的代码添加到 *mongoose/weather/services.ts* 文件中。它与 findOne 函数类似，不同之处在于它直接删除查询结果。Mongoose 会排队执行这些操作，并在连接建立后自动从数据库中删除文档。

```
export async function deleteByZip(
        paramZip: string
    ): Promise<boolean> {
    try {
        await WeatherModel.deleteOne({zip: paramZip});
        return true;
    } catch (err) {
        console.log(err);
    }
    return false;
} 
```

Listing 7-8: 通过 Mongoose 删除数据

异步函数 deleteByZip 接受一个字符串参数 zip。我们使用它查询模型，找到要删除的文档，并将过滤器传递给 Mongoose 的 deleteOne 函数。该函数应返回一个布尔值。

### 创建一个端到端查询

在全栈开发中，*端到端* 通常指的是数据能够从应用程序的前端（或它的某个 API）一路传递，通过中间件到达后端，然后再回到它的原始来源。为了练习，让我们使用 REST API 的 */zipcode* 端点创建一个简单的端到端示例。

我们将修改 API，以便从 URL 中获取查询参数，查找数据库中对应邮政编码的 weather 对象，然后返回它，实际上是用动态查询结果替换了静态的 JSON 响应。修改文件 *pages/api/v1/weather/[zipcode].ts* 以匹配 Listing 7-9。

```
import type {NextApiRequest, NextApiResponse} from "next";
**import {findByZip} from "./../../../../mongoose/weather/services";**
import dbConnect from "./../../../..//middleware/db-connect";
**dbConnect();**

export default async function handler(
    req: NextApiRequest,
    res: NextApiResponse
): Promise<NextApiResponse<WeatherDetailType> | void> {
 **let data** **= await findByZip(req.query.zipcode as string);**
    return res.status(200).json(**data**);
} 
```

Listing 7-9: 完整的 REST API

注意修改后的 API 处理程序。我们对它做了两个主要修改。首先，我们调用了 dbConnect 来连接数据库。然后，我们使用导入的 findByZip 服务，并将查询参数转换为字符串类型传递给它。与之前使用静态 JSON 对象不同，我们现在返回从服务函数接收到的动态 data。

在我们能够接收 API 调用响应数据之前，还需要执行一步：*初始化数据*，即向数据库中添加初始数据集。为了简化操作，我们使用 storeDocuments 服务，并直接在 dbConnect 函数中进行初始化。修改 *middleware/db-connect.ts* 文件，使其与 列表 7-10 中的代码一致，该代码导入了 storeDocument 服务，并在建立数据库连接后添加数据集。

```
import mongoose from "mongoose";
import {MongoMemoryServer} from "mongodb-memory-server";
**import {storeDocument} from** **"****../mongoose/weather/services****"****;**

async function dbConnect(): Promise<any | String> {
    const mongoServer = await MongoMemoryServer.create();
    const MONGOIO_URI = mongoServer.getUri();
    await mongoose.disconnect();

    let db = await mongoose.connect(MONGOIO_URI, {
        dbName: "Weather"
    });

    **await storeDocument({**
    **zip:** **"****96815****"****,**
    **weather:** **"****sunny****"****,**
    **tempC:** **"****25C****"****,**
      **tempF:** **"****70F****"****,**
    **    friends: [****"****96814****"****,** **"****96826****"****]**
    **});**
    **await storeDocument({**
    **zip:** **"****96814****"****,**
    **weather:** **"****rainy****"****,**
    **tempC:** **"****20C****"****,**
    **tempF:** **"****68F****"****,**
    **    friends: [****"****96815****"****,** **"****96826****"****]**
    **});**
    **await storeDocument({**
        **zip:** **"****96826****"****,**
    **weather:** **"****rainy****"****,**
      **tempC:** **"****30C****"****,**
      **tempF:** **"****86F****"****,**
      **friends: [****"****96815****"****,** **"****96814****"****]**
    **});**

}
export default dbConnect; 
```

列表 7-10：在 dbConnect 函数中的简单数据初始化

现在我们可以执行端到端请求。在浏览器中访问 REST API 端点 *http://localhost:3000/api/v1/weather/96815*。你应该能看到来自 MongoDB 数据库的数据集作为 API 响应。尝试在 URL 中调整查询参数为另一个有效的邮政编码。你应该会在响应中获得另一个数据集。

练习 7：将 GraphQL API 连接到数据库

让我们重新设计天气应用的 GraphQL API，使其从数据库读取响应数据，而不是从静态的 JSON 文件中读取。代码看起来会很熟悉，因为我们将使用与前一节 REST API 示例相同的模式。

首先，验证是否已将 MongoDB 内存实现和 Mongoose 添加到你的项目中。如果没有，请按照第 117 页的《设置 MongoDB 和 Mongoose》中的说明进行添加。接下来，检查是否已创建本章中描述的 *middleware* 和 *mongoose* 文件夹中的文件，并确保它们包含从 列表 7-1 到 7-10 的代码。

现在，为了将 GraphQL API 连接到数据库，我们需要做两件事：实现数据库连接，并重构 GraphQL 解析器以使用其数据集。

#### 连接到数据库

要通过 GraphQL API 查询数据库，我们需要连接到数据库。正如你在第六章中所学到的，所有 API 调用都有相同的端点，*/graphql*。这一点现在对我们来说非常方便；因为所有请求都使用相同的入口点，我们只需要处理一次数据库连接。因此，我们打开文件 *api/graphql.ts*，并将其修改为与清单 7-11 中的代码相匹配。

```
import {ApolloServer} from "@apollo/server";
import {startServerAndCreateNextHandler} from "@as-integrations/next";
import {resolvers} from "../../graphql/resolvers";
import {typeDefs} from "../../graphql/schema";
import {NextApiHandler, NextApiRequest, NextApiResponse} from "next";
**import dbConnect from "../../middleware/db-connect";**
//@ts-ignore
const server = new ApolloServer({
    resolvers,
    typeDefs
});

const handler = startServerAndCreateNextHandler(server);

const allowCors = (fn: NextApiHandler) =>
    async (req: NextApiRequest, res: NextApiResponse) => {
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

**const connectDB =** (fn: NextApiHandler) =>
    async (req: NextApiRequest, res: NextApiResponse) => {
    **await dbConnect();**
        return await fn(req, res);
    };

export default **connectDB(**allowCors(handler)**)**; 
```

清单 7-11：包括数据库连接的 api/graphql.ts 文件

我们对文件进行了三处修改。首先，我们从中间件导入了 dbConnect 函数；然后，我们创建了一个类似于 allowCors 函数的新包装器，并使用它确保每个 API 调用都能连接到 API。我们能够安全地这样做，因为我们实现了 dbConnect 来强制保证每次只有一个数据库连接。最后，我们用新的包装器包装了处理程序，并将其作为默认导出。

#### 将服务添加到 GraphQL 解析器

现在是时候将服务添加到解析器中了。在第六章中，你已经学习到查询解析器实现了数据的读取，而突变解析器则实现了数据的创建、更新和删除。

在这里，我们还定义了两个解析器：一个返回给定 ZIP 代码的天气对象，另一个更新某个位置的天气数据。现在，我们将把在本章中创建的服务 findByZip 和 updateByZip 添加到解析器中。我们不再使用静态数据对象的简单实现，而是修改解析器通过服务查询和更新 MongoDB 文档。

清单 7-12 展示了修改后的 *graphql/resolvers.ts* 文件，其中我们重构了这两个解析器。

```
import {WeatherInterface} from "../mongoose/weather/interface";
**import {findByZip, updateByZip} from** **"****../mongoose/weather/services****"****;**

export const resolvers = {
    Query: {
        weather: async (_: any, param: WeatherInterface) => {
            let data = **await findByZip(param.zip)**;
            return [data];
        },
    },
    Mutation: {
        weather: async (_: any, param: {data: WeatherInterface}) => {
            **await updateByZip(param.data.zip, param.data**);
            let data = await findByZip(param.data.zip);
            return [data];
        },
    },
}; 
```

清单 7-12：使用服务的 graphql/resolvers.ts 文件

我们用适当的服务替换了原本简单的 array.filter 功能。为了查询数据，我们使用了 findByZip 服务，并将请求负载中的 zip 变量传递给它，然后将结果数据包装在数组中返回。对于突变操作，我们使用了 updateByZip 服务。根据类型定义，weather 突变返回更新后的数据集。为此，我们再次使用 findByZip 服务查询修改后的文档，并将结果作为数组项返回。

访问 *http://localhost:3000/api/graphql* 上的 GraphQL 沙盒，玩转 API 端点以读取和更新 MongoDB 数据库中的文档。

### 总结

在本章中，你探索了使用非关系型数据库 MongoDB 以及其 Mongoose 对象数据建模工具，Mongoose 让你能够添加和强制执行模式，并对 MongoDB 实例进行 CRUD 操作。我们讲解了关系型数据库和非关系型数据库的区别以及它们存储数据的方式。然后，你创建了一个 Mongoose 模式和一个模型，将 Mongoose 连接到 MongoDB 实例，并编写了服务以在 MongoDB 集合上执行操作。

最后，你将 REST 和 GraphQL APIs 连接到了 MongoDB 数据库。现在，所有的 API 都返回动态文档，而非静态数据集，你可以通过它们进行文档的读取和更新。

MongoDB 和 Mongoose 是功能强大的技术，拥有丰富的功能。如果你想深入了解它们，请查阅官方文档 [*https://<wbr>mongoosejs<wbr>.com*](https://mongoosejs.com) 和 [*https://<wbr>www<wbr>.geeksforgeeks<wbr>.org<wbr>/mongoose<wbr>-module<wbr>-introduction*](https://www.geeksforgeeks.org/mongoose-module-introduction)/。

下一章将介绍 Jest，这是一个现代的测试框架，用于进行单元测试、快照测试和集成测试。
