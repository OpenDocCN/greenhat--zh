# 附录

其他 PostgreSQL 资源

![](img/chapterart.png)

本附录包含了帮助你了解 PostgreSQL 发展、寻找其他软件并获得帮助的资源。由于软件资源可能会发生变化，我将在包含本书所有资源的 GitHub 仓库中维护一份本附录的副本。你可以通过 [`nostarch.com/practical-sql-2nd-edition/`](https://nostarch.com/practical-sql-2nd-edition/) 找到指向 GitHub 的链接。

## PostgreSQL 开发环境

在本书中，我们使用了图形用户界面 pgAdmin 来连接 PostgreSQL、执行查询并查看数据库对象。尽管 pgAdmin 是免费的、开源的且非常流行，但它并不是你使用 PostgreSQL 的唯一选择。维基条目“PostgreSQL 客户端”在 [`wiki.postgresql.org/wiki/PostgreSQL_Clients`](https://wiki.postgresql.org/wiki/PostgreSQL_Clients) 中列出了许多替代方案。

以下列表展示了我尝试过的几个工具，包括免费和付费选项。免费的工具适合一般的分析工作。如果你深入数据库开发，可能需要升级到付费选项，后者通常提供高级功能和支持。

1.  Beekeeper Studio 是一个免费的开源 GUI，支持 PostgreSQL、MySQL、Microsoft SQL Server、SQLite 和其他平台。Beekeeper 可在 Windows、macOS 和 Linux 上运行，并且在数据库 GUI 中拥有更精致的应用设计之一（见 [`www.beekeeperstudio.io/`](https://www.beekeeperstudio.io/)）。

1.  DBeaver 被描述为一个“通用数据库工具”，支持 PostgreSQL、MySQL 以及其他许多数据库。DBeaver 包含一个可视化查询构建器、代码自动补全和其他高级功能。它有 Windows、macOS 和 Linux 的付费版和免费版（见 [`dbeaver.com/`](https://dbeaver.com/)）。

1.  DataGrip 是一个 SQL 开发环境，提供代码补全、错误检测以及简化代码的建议等许多功能。它是付费产品，但公司 JetBrains 为学生、教育工作者和非营利组织提供折扣和免费版本（见 [`www.jetbrains.com/datagrip/`](https://www.jetbrains.com/datagrip/)）。

1.  Navicat 是一个功能丰富的 SQL 开发环境，支持 PostgreSQL 以及 MySQL、Oracle、MongoDB 和 Microsoft SQL Server 等其他数据库的版本。Navicat 没有免费版本，但公司提供 14 天的免费试用（见 [`www.navicat.com/`](https://www.navicat.com/)）。

1.  Postbird 是一个简单的跨平台 PostgreSQL GUI，用于编写查询和查看对象。免费且开源（见 [`github.com/Paxa/postbird/`](https://github.com/Paxa/postbird/)）。

1.  Postico 是一个仅限 macOS 的客户端，由 Postgres.app 的开发者制作，灵感来源于 Apple 设计。完整版是付费的，但可以使用功能受限的版本，且没有时间限制（见 [`eggerapps.at/postico/`](https://eggerapps.at/postico/)）。

一个试用版本可以帮助你决定该产品是否适合你。

## PostgreSQL 实用工具、工具和扩展

你可以通过许多第三方工具、实用程序和扩展来扩展 PostgreSQL 的功能。这些工具包括额外的备份和导入/导出选项、改进的命令行格式以及强大的统计包。你可以在线找到一个精选列表，网址为 [`github.com/dhamaniasad/awesome-postgres/`](https://github.com/dhamaniasad/awesome-postgres/)，但这里有几个值得关注的：

1.  Devart Excel 插件 for PostgreSQL 一个 Excel 插件，允许你直接在 Excel 工作簿中加载和编辑 PostgreSQL 数据（详见 [`www.devart.com/excel-addins/postgresql.html`](https://www.devart.com/excel-addins/postgresql.html)）。

1.  MADlib 一个为大数据集设计的机器学习和分析库，集成了 PostgreSQL（详见 [`madlib.apache.org/`](https://madlib.apache.org/)）。

1.  **pgAgent** 一个作业管理器，让你能够在预定时间运行查询及执行其他任务（详见 [`www.pgadmin.org/docs/pgadmin4/latest/pgagent.html`](https://www.pgadmin.org/docs/pgadmin4/latest/pgagent.html)）。

1.  pgBackRest 一个先进的数据库备份与恢复管理工具（详见 [`pgbackrest.org/`](https://pgbackrest.org/)）。

1.  `pgcli` 一个替代 `psql` 的命令行界面，包含自动补全和语法高亮功能（详见 [`github.com/dbcli/pgcli/`](https://github.com/dbcli/pgcli/)）。

1.  pgRouting 使支持 PostGIS 的 PostgreSQL 数据库能够执行网络分析任务，例如沿道路计算行车距离（详见 [`pgrouting.org/`](https://pgrouting.org/)）。

1.  PL/R 一个可加载的过程性语言，提供在 PostgreSQL 函数和触发器中使用 R 统计编程语言的能力（详见 [`www.joeconway.com/plr.html`](https://www.joeconway.com/plr.html)）。

1.  `pspg` 将 `psql` 的输出格式化为可排序、可滚动的表格，支持多种颜色主题（详见 [`github.com/okbob/pspg/`](https://github.com/okbob/pspg/)）。

## PostgreSQL 新闻与社区

现在你已经是一个真正的 PostgreSQL 用户，保持对社区新闻的关注是明智的。PostgreSQL 开发团队定期更新软件，更新可能会影响你写的代码或使用的工具。你甚至可能会发现新的分析机会。

以下是一些在线资源，帮助你保持信息更新：

1.  Crunchy Data 博客 来自 Crunchy Data 团队的帖子，Crunchy Data 提供企业级 PostgreSQL 支持和解决方案（详见 [`blog.crunchydata.com/blog/`](https://blog.crunchydata.com/blog/)）。

1.  **EDB 博客** 来自 EDB 团队的帖子，EDB 是一家提供 PostgreSQL 服务的公司，提供本书中提到的 Windows 安装程序，并主导 pgAdmin 的开发（详见 [`www.enterprisedb.com/blog/`](https://www.enterprisedb.com/blog/)）。

1.  **Planet PostgreSQL** 汇集了数据库社区的博客文章和公告（请参见[`planet.postgresql.org/`](https://planet.postgresql.org/)）。

1.  Postgres Weekly 一份电子邮件通讯，汇总了公告、博客文章和产品发布（请参见[`postgresweekly.com/`](https://postgresweekly.com/)）。

1.  PostgreSQL 邮件列表 这些列表对于向社区专家提问非常有用。pgsql-novice 和 pgsql-general 列表特别适合初学者，尽管需要注意邮件量可能较大（请参见[`www.postgresql.org/list/`](https://www.postgresql.org/list/)）。

1.  **PostgreSQL 新闻档案** 来自 PostgreSQL 团队的官方新闻（请参见[`www.postgresql.org/about/newsarchive/`](https://www.postgresql.org/about/newsarchive/)）。

1.  PostgreSQL 非营利组织 与 PostgreSQL 相关的慈善组织包括美国 PostgreSQL 协会和 PostgreSQL 欧洲。两者都提供有关该产品的教育、活动和倡导（请参见[`postgresql.us/`](https://postgresql.us/)和[`www.postgresql.eu/`](https://www.postgresql.eu/)）。

1.  PostgreSQL 用户组 一个列出提供聚会和其他活动的社区小组的列表（请参见[`www.postgresql.org/community/user-groups/`](https://www.postgresql.org/community/user-groups/)）。

1.  PostGIS 博客 关于 PostGIS 扩展的公告和更新（请参见[`postgis.net/blog/`](https://postgis.net/blog/)）。

此外，我建议你关注你使用的任何与 PostgreSQL 相关软件的开发者注释，例如 pgAdmin。

## 文档

在本书中，我经常引用 PostgreSQL 官方文档中的页面。你可以在主页[`www.postgresql.org/docs/`](https://www.postgresql.org/docs/)找到每个版本的文档以及常见问题解答和维基。随着你对某个主题（例如索引）了解得越来越多，阅读手册的相关部分是很有价值的，或者你可以查找函数的所有选项。特别是，“前言”、“教程”和“SQL 语言”部分涵盖了本书章节中介绍的许多内容。

其他有价值的文档资源包括 Postgres Guide（请参见[`postgresguide.com/`](http://postgresguide.com/)）和 Stack Overflow，在那里你可以找到由开发者发布的相关问题和答案（请参见[`stackoverflow.com/questions/tagged/postgresql/`](https://stackoverflow.com/questions/tagged/postgresql/)）。你还可以查看 PostGIS 的问答网站（请参见[`gis.stackexchange.com/questions/tagged/postgis/`](https://gis.stackexchange.com/questions/tagged/postgis/)）。
