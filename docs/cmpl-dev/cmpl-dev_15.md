

## 第十三章：13 构建 GraphQL API



![](img/Drop-image.jpg)

在本章中，你将通过定义其模式以及每个查询和突变的解析器，为中间件添加一个 GraphQL API。这些解析器将补充在 第十二章 中创建的 Mongoose 服务。查询将是公开的；然而，我们将通过添加授权层（通过 OAuth）将突变暴露为受保护的 API。

与 第六章 中的 GraphQL API 不同，我们将遵循模块化模式来实现这些模式和解析器。我们将不再将所有内容写在一个大文件中，而是将各个元素分拆到不同的文件中。就像在现代 JavaScript 中使用模块一样，这种方法的好处是将代码拆解为更小的逻辑单元，每个单元都有明确的焦点。这些单元提高了代码的可读性和可维护性。

### 设置

我们将使用 Apollo 服务器创建 API 的单一入口点 */api/graphql*，并通过 *@as-integrations/next* 包将其集成到 Next.js 中。首先，从 npm 注册表安装 GraphQL 设置所需的包：

```
$ **docker exec -it foodfinder-application npm install @apollo/server graphql graphql-tag**
**@as-integrations/next \** 
```

安装完成后，在应用程序根目录下创建 *graphql/locations* 文件夹，紧挨着 *middleware* 文件夹。

### 模式

编写模式的第一步是定义查询和突变的 typedef，以及我们为模式使用的任何自定义类型。为此，我们将在 *graphql/locations* 文件夹中将模式拆分为三个文件：*custom.gql.ts*、*queries.gql.ts* 和 *mutations.gql.ts*。然后，我们将使用普通的模板字面量将它们合并到最终的模式定义中。

#### 自定义类型和指令

将 清单 13-1 中的代码添加到 *custom.gql.ts* 文件，以定义 GraphQL 查询的模式。

```
export default `
    directive @cacheControl(maxAge: Int) on FIELD_DEFINITION | OBJECT
    type Location @cacheControl(maxAge: 86400) {
        address: String
        street: String
        zipcode: String
        borough: String
        cuisine: String
        grade: String
        name: String
        on_wishlist: [String] @cacheControl(maxAge: 60)
        location_id: String
    }
`; 
```

清单 13-1：graphql/locations/custom.gql.ts 文件

GraphQL API 将从 Mongoose 模式返回位置对象。因此，我们必须定义一个自定义类型来表示这些位置对象。创建一个自定义的 Location 类型。为了指示服务器缓存检索到的值，为整个自定义类型设置 @cacheControl 指令，并为 on_wishlist 属性设置一个更短的缓存指令，因为我们预期该属性会频繁变化。

#### 查询模式

现在将 清单 13-2 中的代码添加到 *queries.gql.ts* 文件，以定义查询的模式。

```
export default `
    allLocations: [Location]!
    locationsById(location_ids: [String]!): [Location]!
    onUserWishlist(user_id: String!): [Location]!
`; 
```

清单 13-2：graphql/locations/queries.gql.ts 文件

我们定义了一个包含三个 GraphQL 查询的模板字面量，所有查询都是我们在第十二章中为 Mongoose 位置模型实现的服务的入口点。这些查询的名称和参数与服务中的类似，并且查询遵循你在第六章中学到的 GraphQL 语法。

#### 变更操作架构

要定义变更操作架构，请将列表 13-3 中的代码粘贴到*mutations.gql.ts*文件中。

```
export default `
    addWishlist(location_id: String!, user_id: String!): Location!
    removeWishlist(location_id: String!, user_id: String!): Location!
`; 
```

列表 13-3：graphql/locations/mutations.gql.ts 文件

我们使用 GraphQL 语法创建了两个变更操作作为模板字面量：一个用于将项目添加到用户的愿望清单中，另一个用于将其移除。两者都会使用我们在位置服务中实现的updateWishlist函数，因此它们需要location_id和user_id作为参数。

### 将类型定义合并到最终的架构中

我们已将位置架构拆分为两个文件，一个用于查询，一个用于变更操作，并将它们的自定义类型放置在第三个文件中；然而，为了启动 Apollo 服务器，我们需要一个统一的架构。幸运的是，类型定义不过是模板字面量而已，如果我们使用模板字面量占位符，解析器就能将其插入成一个完整的字符串。为此，创建一个新文件*schema.ts*，放入*graphql*文件夹，并添加列表 13-4 中的代码。

```
import gql from "graphql-tag";

import locationTypeDefsCustom from "graphql/locations/custom.gql";
import locationTypeDefsQueries from "graphql/locations/queries.gql";
import locationTypeDefsMutations from "graphql/locations/mutations.gql";

export const typeDefs = gql`

    ${locationTypeDefsCustom}

    type Query {
      ${locationTypeDefsQueries}
    }

    type Mutation {
        ${locationTypeDefsMutations}
    }

`; 
```

列表 13-4：graphql/schema.ts 文件

我们从*graphql-tag*包中导入gql标签。虽然在使用 Apollo 服务器时这一步是可选的，但我们仍然在标记模板前保留gql标签，以确保与所有其他 GraphQL 实现的兼容性。这也能在 IDE 中产生正确的语法高亮，IDE 会静态分析类型定义作为 GraphQL 标签。

接下来，我们导入将用于实现统一架构的依赖项和模式片段。最后，我们使用gql函数创建一个标记模板字面量，使用模板字面量占位符将模式片段合并到架构骨架中。我们添加自定义的Location类型，然后将查询的类型定义合并到Query对象中，将变更操作合并到Mutations对象中，并将架构const作为类型定义导出。

### GraphQL 解析器

现在我们有了架构，我们将转向解析器。我们将采用类似的开发模式，分别编写查询和变更操作文件，然后将它们合并到 Apollo 服务器所需的单个文件中。首先，在 *graphql/locations* 文件夹中创建 *queries.ts* 和 *mutations.ts* 文件，然后将列表 13-5 中的代码添加到 *queries.ts*。

```
import {
    findAllLocations,
    findLocationsById,
    onUserWishlist,
} from "mongoose/locations/services";

export const locationQueries = {
    allLocations: async (_: any) => {
        return await findAllLocations();
    },
    locationsById: async (_: any, param: {location_ids: string[]}) => {
        return await findLocationsById(param.location_ids);
    },
    onUserWishlist: async (_: any, param: {user_id: string}) => {
        return await onUserWishlist(param.user_id);
    },
}; 
```

列表 13-5：graphql/locations/queries.ts 文件

我们从 Mongoose 文件夹导入服务，然后创建并导出位置查询对象。每个查询的结构遵循在第六章中讨论的结构。我们为每个服务创建一个查询，并且它们的参数与服务中的参数相匹配。

对于变更操作，将列表 13-6 中的代码添加到 *mutations.ts* 文件中。

```
import {updateWishlist} from "mongoose/locations/services";

interface UpdateWishlistInterface {
    user_id: string;
    location_id: string;
}

export const locationMutations = {
    removeWishlist: async (
        _: any,
        param: UpdateWishlistInterface,
        context: {}
    ) => {
        return await updateWishlist(param.location_id, param.user_id,
            "remove"
        );
    },
    addWishlist: async (_: any, param: UpdateWishlistInterface, context: {}) => {
        return await updateWishlist(param.location_id, param.user_id, "add");
    },
}; 
```

列表 13-6：graphql/locations/mutations.ts 文件

在这里，我们只从服务中导入 `updateWishlist` 函数。这是因为我们将其定义为更新文档的单一入口点，并且我们选择使用第三个参数，其值为 `add` 或 `remove`，来区分变更操作应该执行的两个动作。我们还创建了 `UpdateWishlistInterface`，但我们并未导出它。相反，我们将在此文件中使用它，以避免在为函数的 `param` 参数定义接口时重复代码。

作为变更操作，我们在 `locationMutations` 对象中创建了两个函数，一个用于从用户的愿望清单中添加项目，另一个用于移除它。两个函数都使用 `updateWishlist` 服务，并提供对应用户操作的 `value` 参数。这两个变更操作，`removeWishlist` 和 `addWishlist`，还接受一个名为 `context` 的第三个对象。目前，它是一个空对象，但在第十五章中，我们将用验证执行操作的用户身份所需的会话信息替换它。

创建最终的解析器文件 *resolvers.ts*，并将列表 13-7 中的代码添加到该文件中。此代码将合并变更和查询定义。

```
import {locationQueries} from "graphql/locations/queries";
import {locationMutations} from "graphql/locations/mutations";

export const resolvers = {
    Query: {
...locationQueries,
    },
    Mutation: {
...locationMutations,
    },
}; 
```

列表 13-7：graphql/resolvers.ts 文件

除了模式外，我们还必须将一个包含所有解析器的对象传递给 Apollo 服务器，正如我们在第六章中讨论的那样。为了做到这一点，我们必须导入查询和变更。然后，我们使用扩展运算符将导入的对象合并到<sup class="SANS_TheSansMonoCd_W5Regular_11">resolvers</sup>对象中，并导出它。现在，模式和<sup class="SANS_TheSansMonoCd_W5Regular_11">resolvers</sup>对象都可以使用，我们可以创建 API 端点并实例化 Apollo 服务器。

### 将 API 端点添加到 Next.js

在我们讨论 REST 和 GraphQL API 之间的差异时，我们指出，与每个 REST API 都有自己独立的端点不同，GraphQL 只提供一个端点，通常暴露为*/graphql*。为了创建这个端点，我们将使用 Apollo 服务器的 Next.js 集成，就像我们在第六章中做的那样。

在*pages/api*文件夹中创建*graphql.ts*文件，并复制清单 13-8 中的代码，该代码定义了 API 处理程序及其唯一的入口点。

```
import {ApolloServer, BaseContext} from "@apollo/server";
import {startServerAndCreateNextHandler} from "@as-integrations/next";

import {resolvers} from "graphql/resolvers";
import {typeDefs} from "graphql/schema";
import dbConnect from "middleware/db-connect";

import {NextApiHandler, NextApiRequest, NextApiResponse} from "next";

❶ const server = new ApolloServer<BaseContext>({
    resolvers,
    typeDefs,
});

❷ const handler = startServerAndCreateNextHandler(server, {
    context: async () => {
        const token = {};
        return {token};
    },
});

❸ const allowCors =
    (fn: NextApiHandler) =>
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

❹ const connectDB =
    (fn: NextApiHandler) =>
    async (req: NextApiRequest, res: NextApiResponse) => {
        await dbConnect();
        return await fn(req, res);
    };

export default connectDB(allowCors(handler)); 
```

清单 13-8：pages/api/graphql.ts 文件

我们导入了创建 API 处理程序所需的所有元素：Apollo 服务器、Apollo-Next.js 集成的辅助工具、我们的解析器、GraphQL 模式文件、用于连接数据库的函数以及 Next.js 的 API 辅助工具。

我们使用解析器和模式❶创建一个新的 Apollo 服务器。然后，我们使用 Next.js 集成辅助工具❷来启动 Apollo 服务器并返回一个 Next.js 处理程序。集成辅助工具使用无服务器 Apollo 设置，顺利地与 Next.js 自定义服务器集成，而不是创建它自己的服务器。此外，我们将带有空<sup class="SANS_TheSansMonoCd_W5Regular_11">token</sup>的<sup class="SANS_TheSansMonoCd_W5Regular_11">context</sup>传递给处理程序。这就是我们如何访问在 OAuth 流程中收到的 JWT，并稍后将其传递给解析器的方法。

接下来，我们创建在第六章中讨论的包装函数，以添加 CORS 头❸并确保在每个 API 调用中都有数据库连接❹。我们可以安全地这样做，因为我们已经以返回现有缓存连接的方式设置了数据库连接。最后，我们导出返回的异步包装处理程序。

访问 Apollo 沙箱，地址是*http:/localhost:3000/api/graphql*，并运行一些查询以测试 GraphQL API，然后再进入下一章。如果你看到的是天气查询和变更而不是 Food Finder 的查询，请清除浏览器缓存并进行强制刷新。

### 摘要

我们已成功将 GraphQL API 添加到中间件中。通过本章中的代码，我们现在可以使用 Apollo 沙盒来读取和更新数据库中的值。我们还为认证准备了 Apollo 处理程序，并为其提供了一个空的令牌。现在，我们准备使用在 第十五章 中从 OAuth 流程中获得的 JWT 令牌来保护 API 的变更操作。在我们添加认证之前，让我们先构建前端。
