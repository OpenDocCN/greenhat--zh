## 第七章：滥用数据库和文件系统

![Image](img/common.jpg)

现在我们已经介绍了大多数用于主动服务查询、指挥控制和其他恶意活动的常见网络协议，让我们将焦点转向另一个同样重要的话题：数据掠夺。

虽然数据掠夺可能没有初期的漏洞利用、横向网络移动或权限提升那么刺激，但它是整个攻击链中至关重要的一环。毕竟，我们通常需要数据来执行其他活动。通常，数据对攻击者来说具有实际价值。虽然攻击一个组织令人兴奋，但数据本身往往是攻击者的丰厚奖品，而对组织来说却是灾难性的损失。

根据你阅读的研究，2020 年的一次数据泄露可能会让一个组织损失约 400 万到 700 万美元。IBM 的研究估计，每条被盗记录的损失在 129 美元到 355 美元之间。天哪，黑帽黑客通过在地下市场上以每张卡 7 美元到 80 美元的价格出售信用卡，能够赚取可观的利润（[*http://online.wsj.com/public/resources/documents/secureworks_hacker_annualreport.pdf*](http://online.wsj.com/public/resources/documents/secureworks_hacker_annualreport.pdf)）。

单单是 Target 的数据泄露就导致了 4000 万张卡片的泄露。在某些情况下，Target 的卡片甚至以每张 135 美元的价格出售（*[`www.businessinsider.com/heres-what-happened-to-your-target-data-that-was-hacked-2014-10/`](http://www.businessinsider.com/heres-what-happened-to-your-target-data-that-was-hacked-2014-10/)*）。这相当可观。我们绝不提倡这种行为，但那些道德底线存疑的人通过数据掠夺可以赚大钱。

够了，关于行业和在线文章的华丽参考——让我们开始掠夺吧！在本章中，你将学习如何设置和初始化各种 SQL 和 NoSQL 数据库，并学习如何通过 Go 连接并与这些数据库进行交互。我们还将演示如何创建一个数据库和文件系统数据挖掘器，用来搜索有价值信息的关键指标。

### 使用 Docker 设置数据库

在本节中，你将安装各种数据库系统，然后用你将在本章的数据掠夺示例中使用的数据对它们进行初始化。在可能的情况下，你将使用 Docker 在 Ubuntu 18.04 虚拟机上进行操作。*Docker* 是一个软件容器平台，使得应用程序的部署和管理变得更加简便。你可以将应用程序及其依赖项打包在一起，从而简化它们的部署。容器与操作系统是隔离的，以防止污染宿主平台。这是相当巧妙的技术。

对于本章，你将使用各种预构建的 Docker 镜像来处理你将要使用的数据库。如果你还没有安装 Docker，请安装它。你可以在[*https://docs.docker.com/install/linux/docker-ce/ubuntu/*](https://docs.docker.com/install/linux/docker-ce/ubuntu/)找到 Ubuntu 的安装说明。

**注意**

*我们特意选择省略了设置 Oracle 实例的细节。虽然 Oracle 提供了可以下载并用于创建测试数据库的虚拟机镜像，但我们认为没有必要带你完成这些步骤，因为它们与下面的 MySQL 示例非常相似。我们将 Oracle 特定的实现作为一个独立的练习留给你去做。*

#### 安装并初始化 MongoDB

*MongoDB*是本章唯一使用的 NoSQL 数据库。与传统的关系型数据库不同，MongoDB 不通过 SQL 进行通信。相反，MongoDB 使用易于理解的 JSON 语法来检索和操作数据。整本书都可以用来解释 MongoDB，而全面的解释显然超出了本书的范围。现在，你将安装 Docker 镜像并用虚假数据进行初始化。

与传统的 SQL 数据库不同，MongoDB 是*无模式的*，这意味着它不遵循预定义的、僵化的规则系统来组织表数据。这也解释了为什么你在清单 7-1 中只会看到`insert`命令，而没有任何模式定义。首先，使用以下命令安装 MongoDB Docker 镜像：

```
$ docker run --name some-mongo -p 27017:27017 mongo
```

这个命令从 Docker 仓库下载名为`mongo`的镜像，启动一个名为`some-mongo`的新实例——你给实例起的名字是随意的——并将本地端口`27017`映射到容器端口`27017`。端口映射非常关键，因为它允许我们直接从操作系统访问数据库实例。如果没有这个映射，将无法访问。

通过列出所有正在运行的容器，检查容器是否已自动启动：

```
$ docker ps
```

如果你的容器没有自动启动，请运行以下命令：

```
$ docker start some-mongo
```

`start`命令应该可以启动容器。

一旦容器启动，使用`run`命令连接到 MongoDB 实例——传递 MongoDB 客户端，这样你就可以与数据库交互来初始化数据：

```
$ docker run -it --link some-mongo:mongo --rm mongo sh \
  -c 'exec mongo "$MONGO_PORT_27017_TCP_ADDR:$MONGO_PORT_27017_TCP_PORT/store"'
>
```

这个神奇的命令运行了一个临时的第二个 Docker 容器，该容器已安装 MongoDB 客户端二进制文件——因此你不必在宿主操作系统上安装该二进制文件——并使用它连接到`some-mongo` Docker 容器的 MongoDB 实例。在这个例子中，你将连接到一个名为`test`的数据库。

在清单 7-1 中，你将一组文档插入到`transactions`集合中。（所有位于根目录下的代码清单都存在于提供的 github 仓库* [`github.com/blackhat-go/bhg/`](https://github.com/blackhat-go/bhg/)* 中。）

```
> db.transactions.insert([
{
    "ccnum" : "4444333322221111",
    "date" : "2019-01-05",
    "amount" : 100.12,
    "cvv" : "1234",
    "exp" : "09/2020"
},
{
    "ccnum" : "4444123456789012",
    "date" : "2019-01-07",
    "amount" : 2400.18,
    "cvv" : "5544",
    "exp" : "02/2021"
},
{
    "ccnum" : "4465122334455667",
    "date" : "2019-01-29",
    "amount" : 1450.87,
    "cvv" : "9876",
    "exp" : "06/2020"
}
]);
```

*清单 7-1：将事务插入到 MongoDB 集合中 (*[/ch-7/db/seed-mongo.js](https://github.com/blackhat-go/bhg/blob/master/ch-7/db/seed-mongo.js)*)*

就这样！你现在已经创建了 MongoDB 数据库实例，并使用包含三条虚假文档的`transactions`集合进行了初始化，供查询使用。你稍后会进行查询部分，但首先，你应该了解如何安装和初始化传统的 SQL 数据库。

#### 安装和初始化 PostgreSQL 和 MySQL 数据库

*PostgreSQL*（也叫*Postgres*）和*MySQL*可能是最常见、最知名的企业级开源关系型数据库管理系统，并且它们都有官方的 Docker 镜像。由于它们的相似性以及安装步骤的一般重叠，我们将两者的安装步骤集中在一起进行说明。

首先，像之前 MongoDB 示例中的操作一样，下载并运行相应的 Docker 镜像：

```
$ docker run --name some-mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password -d mysql
$ docker run --name some-postgres -p 5432:5432 -e POSTGRES_PASSWORD=password -d postgres
```

在容器构建完成后，确认它们是否正在运行。如果没有运行，可以通过 `docker start name` 命令启动它们。

接下来，你可以通过适当的客户端连接到容器——再次使用 Docker 镜像以避免在主机上安装额外的文件——并继续创建和填充数据库。在 清单 7-2 中，你可以看到 MySQL 的逻辑。

```
$ docker run -it --link some-mysql:mysql --rm mysql sh -c \
'exec mysql -h "$MYSQL_PORT_3306_TCP_ADDR" -P"$MYSQL_PORT_3306_TCP_PORT" \
-uroot -p"$MYSQL_ENV_MYSQL_ROOT_PASSWORD"'
mysql> create database store;
mysql> use store;
mysql> create table transactions(ccnum varchar(32), date date, amount float(7,2),
    -> cvv char(4), exp date);
```

*清单 7-2：创建和初始化 MySQL 数据库*

该清单与接下来的清单一样，启动一个一次性 Docker shell，并执行适当的数据库客户端二进制文件。它创建并连接到名为 `store` 的数据库，然后创建一个名为 `transactions` 的表。这两个清单是相同的，唯一的区别是它们分别针对不同的数据库系统进行定制。

在 清单 7-3 中，你可以看到 Postgres 的逻辑，它在语法上与 MySQL 稍有不同。

```
$ docker run -it --rm --link some-postgres:postgres postgres psql -h postgres -U postgres
postgres=# create database store;
postgres=# \connect store
store=# create table transactions(ccnum varchar(32), date date, amount money, cvv
        char(4), exp date);
```

*清单 7-3：创建和初始化 Postgres 数据库*

在 MySQL 和 Postgres 中，插入事务的语法是相同的。例如，在 清单 7-4 中，你可以看到如何将三个文档插入到 MySQL `transactions` 集合中。

```
mysql> insert into transactions(ccnum, date, amount, cvv, exp) values
    -> ('4444333322221111', '2019-01-05', 100.12, '1234', '2020-09-01');
mysql> insert into transactions(ccnum, date, amount, cvv, exp) values
    -> ('4444123456789012', '2019-01-07', 2400.18, '5544', '2021-02-01');
mysql> insert into transactions(ccnum, date, amount, cvv, exp) values
    -> ('4465122334455667', '2019-01-29', 1450.87, '9876', '2019-06-01');
```

*清单 7-4：将事务插入 MySQL 数据库 (*[/ch-7/db/seed-pg-mysql.sql](https://github.com/blackhat-go/bhg/blob/master/ch-7/db/seed-pg-mysql.sql)*)*

尝试将相同的三个文档插入到你的 Postgres 数据库中。

#### 安装和初始化 Microsoft SQL Server 数据库

2016 年，微软开始大力推动开源一些核心技术，其中之一就是 Microsoft SQL (MSSQL) Server。值得一提的是，在展示这些曾经不可能做到的操作时，尤其是在 Linux 操作系统上安装 MSSQL Server 更显得重要。更棒的是，它有一个 Docker 镜像，你可以通过以下命令安装它：

```
$ docker run --name some-mssql -p 1433:1433 -e 'ACCEPT_EULA=Y' \
-e 'SA_PASSWORD=Password1!' -d microsoft/mssql-server-linux
```

该命令与前两节中运行的命令类似，但根据文档要求，`SA_PASSWORD` 的值必须复杂——即包含大写字母、小写字母、数字和特殊字符——否则你将无法进行身份验证。由于这是一个测试实例，前面的值只是满足了这些基本要求——这也是我们在企业网络中经常看到的情况！

安装完镜像后，启动容器，创建架构，并填充数据库，参见 清单 7-5。

```
$ docker exec -it some-mssql /opt/mssql-tools/bin/sqlcmd -S localhost \
-U sa -P 'Password1!'
> create database store;
> go
> use store;
> create table transactions(ccnum varchar(32), date date, amount decimal(7,2),
> cvv char(4), exp date);
> go
> insert into transactions(ccnum, date, amount, cvv, exp) values
> ('4444333322221111', '2019-01-05', 100.12, '1234', '2020-09-01');
> insert into transactions(ccnum, date, amount, cvv, exp) values
> ('4444123456789012', '2019-01-07', 2400.18, '5544', '2021-02-01');
> insert into transactions(ccnum, date, amount, cvv, exp) values
> ('4465122334455667', '2019-01-29', 1450.87, '9876', '2020-06-01');
> go
```

*示例 7-5：创建并填充 MSSQL 数据库*

之前的示例复现了我们之前在 MySQL 和 Postgres 中演示的逻辑。它使用 Docker 连接到服务，创建并连接到 `store` 数据库，并创建并填充 `transactions` 表。我们将其与其他 SQL 数据库分开呈现，因为它有一些 MSSQL 特定的语法。

### 在 Go 中连接并查询数据库

现在你有多种测试数据库可以使用，你可以构建逻辑来从 Go 客户端连接并查询这些数据库。我们将此讨论分为两个主题——一个是 MongoDB，一个是传统的 SQL 数据库。

#### 查询 MongoDB

尽管 Go 拥有一个优秀的标准 SQL 包，但它并没有维护一个类似的包来与 NoSQL 数据库进行交互。相反，你需要依赖第三方包来促进这种交互。我们将不检查每个第三方包的实现，而是专注于 MongoDB。我们将使用 `mgo`（发音为 *mango*）数据库驱动来进行操作。

首先，使用以下命令安装 `mgo` 驱动：

```
$ go get gopkg.in/mgo.v2
```

你现在可以建立连接并查询你的 `store` 集合（相当于表），这比我们稍后将创建的 SQL 示例代码所需的代码还要少（见 示例 7-6）。

```
package main

import (
    "fmt"
    "log"

    mgo "gopkg.in/mgo.v2"
)

type Transaction struct { ❶
    CCNum      string  `bson:"ccnum"`
    Date       string  `bson:"date"`
    Amount     float32 `bson:"amount"`
    Cvv        string  `bson:"cvv"`
    Expiration string  `bson:"exp"`
}

func main() {
    session, err := mgo.Dial("127.0.0.1") ❷
    if err != nil {
        log.Panicln(err)
    }  
    defer session.Close()

    results := make([]Transaction, 0)
    if err := session.DB("store").C("transactions").Find(nil).All(&results)❸; err != nil {
        log.Panicln(err)
    }  
    for _, txn := range results { ❹
        fmt.Println(txn.CCNum, txn.Date, txn.Amount, txn.Cvv, txn.Expiration)
    }
}
```

*示例 7-6：连接并查询 MongoDB 数据库 (*[/ch-7/db/mongo-connect/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-7/db/mongo-connect/main.go)*)*

首先，定义一个类型 `Transaction`，它将表示来自 `store` 集合的单个文档 ❶。MongoDB 中的数据表示机制是二进制 JSON。因此，使用标记来定义任何序列化指令。在这种情况下，你使用标记显式定义要在二进制 JSON 数据中使用的元素名称。

在你的 `main()` 函数 ❷ 中，调用 `mgo.Dial()` 通过建立与数据库的连接来创建会话，测试以确保没有错误发生，并延迟调用以关闭会话。然后，你使用 `session` 变量查询 `store` 数据库 ❸，从 `transactions` 集合中检索所有记录。你将结果存储在一个名为 `results` 的 `Transaction` 切片中。在底层，你的结构标签被用来将二进制 JSON 解组为你定义的类型。最后，循环遍历结果集并将其打印到屏幕上 ❹。在这种情况下和下一节中的 SQL 示例中，你的输出应类似于以下内容：

```
$ go run main.go
4444333322221111 2019-01-05 100.12 1234 09/2020
4444123456789012 2019-01-07 2400.18 5544 02/2021
4465122334455667 2019-01-29 1450.87 9876 06/2020
```

#### 查询 SQL 数据库

Go 包含一个标准包，名为`database/sql`，它定义了与 SQL 及类似 SQL 的数据库交互的接口。基本实现自动包含了连接池和事务支持等功能。遵循此接口的数据库驱动程序自动继承这些功能，并且基本上是可以互换的，因为 API 在驱动程序之间保持一致。无论你使用 Postgres、MSSQL、MySQL 还是其他驱动程序，代码中的函数调用和实现都是相同的。这使得切换后端数据库变得方便，只需要在客户端做最小的代码修改。当然，驱动程序可以实现特定于数据库的功能并使用不同的 SQL 语法，但函数调用几乎相同。

出于这个原因，我们将展示如何连接到一个 SQL 数据库——MySQL——并将其他 SQL 数据库留作练习。你首先通过以下命令安装驱动程序：

```
$ go get github.com/go-sql-driver/mysql
```

然后，你可以创建一个基本客户端，连接到数据库并从你的`transactions`表中检索信息——使用列表 7-7 中的脚本。

```
package main

import (
    "database/sql" ❶
    "fmt"
    "log"

    "github.com/go-sql-driver/mysql" ❷
)

func main() {
    db, err := sql.Open("mysql", "root:password@tcp(127.0.0.1:3306)/store")❸
    if err != nil {
        log.Panicln(err)
    }  
    defer db.Close()

    var (
        ccnum, date, cvv, exp string
        amount                float32
    )  
    rows, err := db.Query("SELECT ccnum, date, amount, cvv, exp FROM transactions") ❹
 if err != nil {
        log.Panicln(err)
    }  
    defer rows.Close()
    for rows.Next() {
        err := rows.Scan(&ccnum, &date, &amount, &cvv, &exp)❺
        if err != nil {
            log.Panicln(err)
        }
        fmt.Println(ccnum, date, amount, cvv, exp)
    }  
    if rows.Err() != nil {
        log.Panicln(err)
    }
}
```

*列表 7-7：连接和查询 MySQL 数据库（*[/ch-7/db/mysql-connect/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-7/db/mysql-connect/main.go)*)*

代码首先通过导入 Go 的`database/sql`包❶开始。这使你能够利用 Go 强大的标准 SQL 库接口与数据库交互。你还导入了你的 MySQL 数据库驱动程序❷。前导下划线表示它是匿名导入的，这意味着它的导出类型没有被包含，但驱动程序会将自己注册到`sql`包中，这样 MySQL 驱动程序本身就能处理函数调用。

接下来，你调用`sql.Open()`来建立与我们数据库的连接❸。第一个参数指定应使用哪个驱动程序——在这种情况下，驱动程序是`mysql`——第二个参数指定你的连接字符串。然后，你查询数据库，传递一个 SQL 语句来选择`transactions`表中的所有行❹，接着遍历这些行，随后将数据读取到你的变量中并打印出值❺。

这就是你查询 MySQL 数据库所需做的全部工作。使用不同的后端数据库只需要对代码做以下小的修改：

1.  导入正确的数据库驱动程序。

1.  更改传递给`sql.Open()`的参数。

1.  根据你的后端数据库的要求调整 SQL 语法。

在几个可用的数据库驱动程序中，许多是纯 Go 编写的，而少数使用`cgo`进行一些底层交互。查看可用驱动程序的列表，网址是*[`github.com/golang/go/wiki/SQLDrivers/`](https://github.com/golang/go/wiki/SQLDrivers/)。

### 构建一个数据库挖掘器

在本节中，你将创建一个工具，用来检查数据库架构（例如，列名），以确定其中的数据是否值得盗取。例如，假设你想找出密码、哈希值、社会安全号码和信用卡号码。与其构建一个单一的工具来挖掘各种后端数据库，不如为每个数据库创建独立的工具，并实现一个定义好的接口，以确保不同实现之间的一致性。虽然这个灵活性在本例中可能有些过度，但它给你提供了创建可重用和可移植代码的机会。

该接口应该是简洁的，包含一些基本类型和函数，并且只需要实现一个方法来检索数据库架构。清单 7-8，即名为*dbminer.go*的文件，定义了数据库挖掘器的接口。

```
   package dbminer

   import (
       "fmt"
       "regexp"
   )

❶ type DatabaseMiner interface {
       GetSchema() (*Schema, error)
   }

❷ type Schema struct {
       Databases []Database
   }

   type Database struct {
       Name   string
       Tables []Table
   }

   type Table struct {
       Name    string
       Columns []string
   }

❸ func Search(m DatabaseMiner) error {
    ❹ s, err := m.GetSchema()
       if err != nil {
           return err
       }

       re := getRegex()
    ❺ for _, database := range s.Databases {
           for _, table := range database.Tables {
               for _, field := range table.Columns {
                   for _, r := range re {
                       if r.MatchString(field) {
                           fmt.Println(database)
                           fmt.Printf("[+] HIT: %s\n", field)
                       }
                   }
               }
           }
       }
       return nil
   }

❻ func getRegex() []*regexp.Regexp {
       return []*regexp.Regexp{
           regexp.MustCompile(`(?i)social`),
           regexp.MustCompile(`(?i)ssn`),
           regexp.MustCompile(`(?i)pass(word)?`),
           regexp.MustCompile(`(?i)hash`),
           regexp.MustCompile(`(?i)ccnum`),
           regexp.MustCompile(`(?i)card`),
           regexp.MustCompile(`(?i)security`),
           regexp.MustCompile(`(?i)key`),
       }
   }

   /* Extranneous code omitted for brevity */
```

*清单 7-8：数据库挖掘器实现（*[/ch-7/db/dbminer/dbminer.go](https://github.com/blackhat-go/bhg/blob/master/ch-7/db/dbminer/dbminer.go)*)*

代码首先定义了一个名为`DatabaseMiner`的接口❶。任何实现该接口的类型都需要包含一个名为`GetSchema()`的方法。由于每个后端数据库可能有特定的逻辑来检索数据库架构，预期是每个特定的工具都能以独特的方式实现该逻辑，适配不同的后端数据库和驱动程序。

接下来，你定义了一个`Schema`类型，该类型由几个子类型组成，也在此定义❷。你将使用`Schema`类型来逻辑上表示数据库架构——即数据库、表和列。你可能已经注意到，在接口定义中的`GetSchema()`函数，预期实现返回一个`*Schema`。

现在，你定义了一个名为`Search()`的单一函数，其中包含大部分逻辑。`Search()`函数在调用时预期传入一个`DatabaseMiner`实例，并将矿工值存储在名为`m`的变量中❸。该函数首先调用`m.GetSchema()`来获取架构❹。然后，函数循环遍历整个架构，搜索与常见的正则表达式（regex）值匹配的列名❺。如果找到匹配项，数据库架构和匹配的字段将被打印到屏幕上。

最后，定义一个名为`getRegex()`的函数❻。该函数使用 Go 的`regexp`包编译正则表达式字符串，并返回这些值的切片。正则表达式列表由不区分大小写的字符串组成，用来匹配常见或有趣的字段名称，如`ccnum`、`ssn`和`password`。

拥有了数据库挖掘器接口之后，你可以创建特定工具的实现。让我们从 MongoDB 数据库挖掘器开始。

#### 实现 MongoDB 数据库挖掘器

Listing 7-9 中的 MongoDB 工具程序实现了 Listing 7-8 中定义的接口，并且还集成了你在 Listing 7-6 中构建的数据库连接代码。

```
   package main

   import (
       "os"

    ❶ "github.com/bhg/ch-7/db/dbminer"
       "gopkg.in/mgo.v2"
       "gopkg.in/mgo.v2/bson"
   )

❷ type MongoMiner struct {
       Host    string
       session *mgo.Session
   }

❸ func New(host string) (*MongoMiner, error) {
       m := MongoMiner{Host: host}
       err := m.connect()
       if err != nil {
           return nil, err
       }  
       return &m, nil
   }

❹ func (m *MongoMiner) connect() error {
       s, err := mgo.Dial(m.Host)
       if err != nil {
           return err
       }  
       m.session = s
       return nil
   }

❺ func (m *MongoMiner) GetSchema() (*dbminer.Schema, error) {
       var s = new(dbminer.Schema)

       dbnames, err := m.session.DatabaseNames()❻
       if err != nil {
           return nil, err
       }

       for _, dbname := range dbnames {
           db := dbminer.Database{Name: dbname, Tables: []dbminer.Table{}}
           collections, err := m.session.DB(dbname).CollectionNames()❼
           if err != nil {
               return nil, err
           }
 for _, collection := range collections {
               table := dbminer.Table{Name: collection, Columns: []string{}}

               var docRaw bson.Raw
               err := m.session.DB(dbname).C(collection).Find(nil).One(&docRaw)❽
               if err != nil {
                   return nil, err
               }

               var doc bson.RawD
               if err := docRaw.Unmarshal(&doc); err != nil {❾
                   if err != nil {
                       return nil, err
                   }
               }

               for _, f := range doc {
                   table.Columns = append(table.Columns, f.Name)
               }
               db.Tables = append(db.Tables, table)
           }
           s.Databases = append(s.Databases, db)
       }  
       return s, nil
   }

   func main() {

       mm, err := New(os.Args[1])
       if err != nil {
           panic(err)
       }  
    ❿ if err := dbminer.Search(mm); err != nil {
           panic(err)
       }
   }
```

*Listing 7-9：创建一个 MongoDB 数据库挖掘器 (*[/ch-7/db/mongo/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-7/db/mongo/main.go)*)*

你首先通过导入定义 `DatabaseMiner` 接口的 `dbminer` 包 ❶。然后你定义一个 `MongoMiner` 类型，用于实现该接口 ❷。为了方便起见，你定义了一个 `New()` 函数，它创建一个新的 `MongoMiner` 类型实例 ❸，并调用一个名为 `connect()` 的方法来建立与数据库的连接 ❹。这段逻辑实际上是初始化你的代码，以类似于 Listing 7-6 中讨论的方式连接到数据库。

代码中最有趣的部分是你实现的 `GetSchema()` 接口方法 ❺。与 Listing 7-6 中的 MongoDB 示例代码不同，现在你正在检查 MongoDB 元数据，首先获取数据库名称 ❻，然后遍历这些数据库以获取每个数据库的集合名称 ❼。最后，函数检索原始文档，不同于典型的 MongoDB 查询，它使用了懒加载解组 ❽。这使得你可以显式地将记录解组到一个通用结构中，从而检查字段名称 ❾。如果没有懒加载解组，你将不得不定义一个显式类型，可能使用 `bson` 标签属性，来指示代码如何将数据解组到你定义的结构体中。在这种情况下，你不知道（也不关心）字段的类型或结构——你只需要字段名称（而非数据）——因此你可以在不知道数据结构的情况下解组结构化数据。

你的 `main()` 函数期望接收一个 MongoDB 实例的 IP 地址作为唯一参数，调用你的 `New()` 函数来初始化所有内容，然后调用 `dbminer.Search()`，并将你的 `MongoMiner` 实例传递给它 ❿。回想一下，`dbminer.Search()` 调用接收到的 `DatabaseMiner` 实例上的 `GetSchema()`；这会调用你在 `MongoMiner` 中实现的函数，进而创建 `dbminer.Schema`，然后在 Listing 7-8 中的正则表达式列表中进行搜索。

当你运行你的工具时，你将看到以下输出：

```
$ go run main.go 127.0.0.1
[DB] = store
    [TABLE] = transactions
       [COL] = _id
       [COL] = ccnum
       [COL] = date
       [COL] = amount
       [COL] = cvv
       [COL] = exp
[+] HIT: ccnum
```

你找到匹配项了！它可能看起来不太好看，但它能完成任务——成功地定位到包含名为 `ccnum` 字段的数据库集合。

在你构建了 MongoDB 实现后，接下来的部分，你将为 MySQL 后端数据库做同样的事情。

#### 实现一个 MySQL 数据库挖掘器

为了使你的 MySQL 实现正常工作，你需要检查`information_schema.columns`表。该表维护关于所有数据库及其结构的元数据，包括表名和列名。为了使数据更容易处理，可以使用以下 SQL 查询，去除一些与你的收集工作无关的内置 MySQL 数据库的信息：

```
SELECT TABLE_SCHEMA, TABLE_NAME, COLUMN_NAME FROM columns
    WHERE TABLE_SCHEMA NOT IN ('mysql', 'information_schema', 'performance_schema', 'sys')
    ORDER BY TABLE_SCHEMA, TABLE_NAME
```

该查询产生的结果类似于以下内容：

```
+--------------+--------------+-------------+
| TABLE_SCHEMA | TABLE_NAME   | COLUMN_NAME |
+--------------+--------------+-------------+
| store        | transactions | ccnum       |
| store        | transactions | date        |
| store        | transactions | amount      |
| store        | transactions | cvv         |
| store        | transactions | exp         |
--snip--
```

尽管使用该查询来检索模式信息相对简单，但代码中的复杂性来自于在定义`GetSchema()`函数时，如何逻辑上区分和分类每一行。例如，连续的输出行可能属于同一个数据库或表，也可能不属于，因此将这些行关联到正确的`dbminer.Database`和`dbminer.Table`实例是一个相对棘手的任务。

清单 7-10 定义了该实现。

```
type MySQLMiner struct {
    Host string
    Db   sql.DB
}

func New(host string) (*MySQLMiner, error) {
    m := MySQLMiner{Host: host}
    err := m.connect()
    if err != nil {
        return nil, err
    }
    return &m, nil
}

func (m *MySQLMiner) connect() error {

    db, err := sql.Open(
        "mysql",
     ❶ fmt.Sprintf("root:password@tcp(%s:3306)/information_schema", m.Host))
    if err != nil {
        log.Panicln(err)
    }
    m.Db = *db
    return nil
}

func (m *MySQLMiner) GetSchema() (*dbminer.Schema, error) {
    var s = new(dbminer.Schema)
 ❷ sql := `SELECT TABLE_SCHEMA, TABLE_NAME, COLUMN_NAME FROM columns
    WHERE TABLE_SCHEMA NOT IN
    ('mysql', 'information_schema', 'performance_schema', 'sys')
    ORDER BY TABLE_SCHEMA, TABLE_NAME`
    schemarows, err := m.Db.Query(sql)
    if err != nil {
        return nil, err
    }
    defer schemarows.Close()

    var prevschema, prevtable string
    var db dbminer.Database
    var table dbminer.Table
 ❸ for schemarows.Next() {
        var currschema, currtable, currcol string
        if err := schemarows.Scan(&currschema, &currtable, &currcol); err != nil {
            return nil, err
        }

     ❹ if currschema != prevschema {
            if prevschema != "" {
                db.Tables = append(db.Tables, table)
                s.Databases = append(s.Databases, db)
            }
            db = dbminer.Database{Name: currschema, Tables: []dbminer.Table{}}
            prevschema = currschema
            prevtable = ""
        }

     ❺ if currtable != prevtable {
            if prevtable != "" {
                db.Tables = append(db.Tables, table)
            }
            table = dbminer.Table{Name: currtable, Columns: []string{}}
            prevtable = currtable
        }
     ❻ table.Columns = append(table.Columns, currcol)
    }
    db.Tables = append(db.Tables, table)
    s.Databases = append(s.Databases, db)
    if err := schemarows.Err(); err != nil {
        return nil, err
    }

    return s, nil
}

func main() {
    mm, err := New(os.Args[1])
    if err != nil {
        panic(err)
    }
    defer mm.Db.Close()
 if err := dbminer.Search(mm); err != nil {
        panic(err)
    }
}
```

*清单 7-10：创建一个 MySQL 数据库矿工 (*[/ch-7/db/mysql/main.go/](https://github.com/blackhat-go/bhg/blob/master/ch-7/db/mysql/main.go)*)*

快速浏览代码，你可能会发现它和前一节中的 MongoDB 示例非常相似。事实上，`main()`函数是完全相同的。

引导函数也类似——你只需改变逻辑以与 MySQL 而非 MongoDB 进行交互。注意，这段逻辑连接到了你的`information_schema`数据库 ❶，以便你检查数据库的模式。

代码的复杂性大部分体现在`GetSchema()`的实现中。虽然你能够通过使用单个数据库查询 ❷ 来检索模式信息，但之后你需要遍历结果 ❸，检查每一行，以便确定存在哪些数据库、每个数据库中有哪些表以及每个表中有哪些列。与 MongoDB 实现不同的是，你没有 JSON/BSON 属性标签来将数据编组和解组到复杂的结构中；你需要维护变量来跟踪当前行中的信息，并将其与上一行的数据进行比较，以确定是否遇到了新的数据库或表。这不是最优雅的解决方案，但它能完成工作。

接下来，你需要检查当前行的数据库名称是否与上一行不同 ❹。如果不同，你就创建一个新的`miner.Database`实例。如果这不是你循环的第一次迭代，就将表和数据库添加到你的`miner.Schema`实例中。你可以使用类似的逻辑来跟踪并将`miner.Table`实例添加到当前的`miner.Database` ❺。最后，将每一列添加到我们的`miner.Table` ❻。

现在，运行程序以检查它是否能够正常工作，方法是对你的 Docker MySQL 实例进行测试，如下所示：

```
$ go run main.go 127.0.0.1
[DB] = store
    [TABLE] = transactions
       [COL] = ccnum
       [COL] = date
       [COL] = amount
       [COL] = cvv
       [COL] = exp
[+] HIT: ccnum
```

输出应该几乎无法与您的 MongoDB 输出区分开。这是因为您的`dbminer.Schema`并没有产生任何输出——是`dbminer.Search()`函数产生了输出。这就是使用接口的强大之处。您可以为关键特性提供具体的实现，同时仍然利用单一、标准的函数以可预测、可用的方式处理您的数据。

在接下来的部分中，您将暂时离开数据库，转而专注于掠夺文件系统。

### 掠夺文件系统

在这一部分中，您将构建一个实用工具，它会递归地遍历用户提供的文件系统路径，并与您认为在后期利用练习中有用的有趣文件名列表进行匹配。这些文件可能包含个人身份信息、用户名、密码、系统登录信息和密码数据库文件等内容。

该实用工具专门关注文件名，而不是文件内容，且由于 Go 在其`path/filepath`包中包含了标准功能，使得您可以轻松地遍历目录结构，脚本因此大大简化。您可以在清单 7-11 中看到该实用工具。

```
   package main

   import (
       "fmt"
       "log"
       "os"
       "path/filepath"
       "regexp"
   )

❶ var regexes = []*regexp.Regexp{
       regexp.MustCompile(`(?i)user`),
       regexp.MustCompile(`(?i)password`),
       regexp.MustCompile(`(?i)kdb`),
       regexp.MustCompile(`(?i)login`),
   }

❷ func walkFn(path string, f os.FileInfo, err error) error {
       for _, r := range regexes {
        ❸ if r.MatchString(path) {
               fmt.Printf("[+] HIT: %s\n", path)
           }  
       }  
       return nil
   }

   func main() {
       root := os.Args[1]
    ❹ if err := filepath.Walk(root, walkFn); err != nil {
           log.Panicln(err)
       }  
   }
```

*清单 7-11：遍历和搜索文件系统 (*[/ch-7/filesystem/main.go](https://github.com/blackhat-go/bhg/blob/master/ch-7/filesystem/main.go)*)*

与您的数据库挖掘实现相比，文件系统掠夺的设置和逻辑看起来可能显得有些简单。类似于您创建数据库实现的方式，您定义了一个用于识别有趣文件名的正则表达式列表 ❶。为了保持代码简洁，我们将列表限制为仅包含少数几个项，但您可以扩展列表以适应更多实际使用场景。

接下来，您定义一个名为`walkFn()`的函数，该函数接受一个文件路径和一些附加参数 ❷。该函数会遍历您的正则表达式列表并检查匹配项 ❸，并将匹配项显示到标准输出。`walkFn()`函数 ❹ 在`main()`函数中使用，并作为参数传递给`filepath.Walk()`。`Walk()`函数期望接收两个参数——根路径和一个函数（在这种情况下是`walkFn()`）——并从提供的根路径开始递归遍历目录结构，对于每一个遇到的目录和文件都会调用`walkFn()`。

完成实用工具后，导航到您的桌面并创建以下目录结构：

```
$ tree targetpath/
targetpath/
--- anotherpath
-   --- nothing.txt
-   --- users.csv
--- file1.txt
--- yetanotherpath
    --- nada.txt
    --- passwords.xlsx

2 directories, 5 files
```

在同一`targetpath`目录下运行您的实用工具，产生以下输出，确认您的代码运行得非常顺利：

```
$ go run main.go ./somepath
[+] HIT: somepath/anotherpath/users.csv
[+] HIT: somepath/yetanotherpath/passwords.xlsx
```

这差不多就是全部内容了。你可以通过加入更多或更具体的正则表达式来改进示例代码。此外，我们建议你改进代码，将正则表达式检查仅应用于文件名，而不是目录。另一个我们建议的改进是定位并标记具有最近修改或访问时间的特定文件。这些元数据可以帮助你找到更重要的内容，包括作为关键业务流程一部分的文件。

### 总结

在这一章中，我们深入探讨了数据库交互和文件系统遍历，使用了 Go 语言的本地包和第三方库来检查数据库元数据和文件名。对于攻击者来说，这些资源通常包含宝贵的信息，我们创建了各种工具，允许我们搜索这些有价值的信息。

在下一章中，你将学习实际的包处理。具体来说，你将学习如何嗅探和操控网络数据包。
