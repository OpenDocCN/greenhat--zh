# 第八章：为你设计的表格

![](img/chapterart.png)

对秩序和细节的执着有时是件好事。当你匆忙出门时，看到钥匙挂在你*总是*放的地方的钩子上会让你安心。数据库设计同样如此。当你需要从数十个表和数百万行中挖掘出一条信息时，你会感谢这种对细节的执着。有了经过精心组织、命名得当的表格，分析过程变得更加可控。

在本章中，我将在第七章的基础上，介绍如何组织和加速 SQL 数据库的最佳实践，无论是你自己的，还是你继承来进行分析的数据库。我们将深入探讨表格设计，探索命名规则和约定、如何维护数据完整性，以及如何为表格添加索引以加速查询。

## 遵循命名约定

编程语言往往有自己的风格模式，甚至不同的 SQL 编码者群体在命名表格、列和其他对象（称为*标识符*）时也偏好某些约定。有些人喜欢使用*驼峰式命名法*，如 `berrySmoothie`，其中单词连写，且每个单词的首字母大写（除了第一个单词）。*帕斯卡式命名法*，如 `BerrySmoothie`，遵循类似的模式，但首字母也大写。使用*蛇形命名法*，如 `berry_smoothie`，所有单词都小写，并用下划线分隔。

你会发现每种命名约定都有热情的支持者，有些偏好与特定的数据库应用程序或编程语言有关。例如，Microsoft 在其 SQL Server 数据库文档中使用帕斯卡式命名法。在本书中，出于 PostgreSQL 相关的原因，我稍后会解释，我们使用蛇形命名法，如 `us_counties_pop_est_2019`。无论你喜欢哪种约定，或者你被要求使用哪种约定，重要的是要始终如一地应用它。一定要检查你的组织是否有风格指南，或者主动提议一起制定，并严格遵守。

混合风格或不遵循任何约定通常会导致混乱。例如，想象一下连接到数据库并发现以下一组表格：

1.  `Customers`

1.  `customers`

1.  `custBackup`

1.  `customer_analysis`

1.  `customer_test2`

1.  `customer_testMarch2012`

1.  `customeranalysis`

你可能会有疑问。例如，哪张表格实际上包含客户的当前数据？一个混乱的命名方案——以及缺乏条理性——使得别人很难进入你的数据，也让你自己在接着上次的工作时感到困难。

让我们探索与命名标识符相关的注意事项以及最佳实践建议。

### 引号标识符启用混合大小写

无论你提供什么样的大小写，PostgreSQL 都将标识符视为小写，除非你将双引号括住标识符。请看以下 PostgreSQL 的 `CREATE TABLE` 语句：

```
CREATE TABLE customers (
    customer_id text,
 `--snip--`
);

CREATE TABLE Customers (
    customer_id text,
 `--snip--`
);
```

当你按顺序执行这些语句时，第一个命令创建了一个名为`customers`的表。第二个语句并不是创建一个名为`Customers`的单独表，而是会抛出一个错误：`relation "customers" already exists`。因为你没有给标识符加引号，PostgreSQL 将`customers`和`Customers`视为相同的标识符，不区分大小写。为了保留大写字母并创建一个名为`Customers`的独立表，你必须将标识符用引号括起来，像这样：

```
CREATE TABLE "Customers" (
    customer_id serial,
 `--snip--`
);
```

然而，因为这要求你在`SELECT`语句中查询`Customers`而不是`customers`时，必须对其名称加引号：

```
SELECT * FROM "Customers";
```

这可能很难记住，也容易让用户混淆。确保你的表名清晰，并且与数据库中的其他表区分开。

### 引用标识符的陷阱

引用标识符还允许你使用其他情况下不被允许的字符，包括空格。对于某些人来说，这可能是一个吸引人的特性，但也有一些负面影响。你可能希望在重新造林数据库中将`"trees planted"`作为列名，但是这样所有用户在引用该列时都必须加上引号。如果在查询中省略引号，数据库会报错，认为`trees`和`planted`是不同的列，并提示`trees`列不存在。一个更具可读性和可靠性的选择是使用蛇形命名法，例如`trees_planted`。

引号还允许你使用 SQL 的*保留关键字*，即在 SQL 中具有特殊意义的单词。你已经遇到过一些，比如`TABLE`、`WHERE`或`SELECT`。大多数数据库开发人员不推荐将保留关键字用作标识符。至少这会造成混淆，最糟糕的是，忽略或忘记稍后引用该关键字时，可能会导致错误，因为数据库会将该单词解释为命令，而不是标识符。

### 命名标识符的指南

考虑到引用标识符的额外负担及其潜在问题，最好保持标识符名称简单、不加引号且一致。以下是我的建议：

1.  使用蛇形命名法（snake case）。蛇形命名法具有良好的可读性和可靠性，如前面提到的`trees_planted`示例所示。它在官方 PostgreSQL 文档中被广泛使用，并帮助使多单词的名称更容易理解：`video_on_demand`比`videoondemand`一目了然。

1.  使名称易于理解，避免使用晦涩的缩写。如果你正在构建一个与旅游相关的数据库，`arrival_time`比`arv_tm`更容易理解。

1.  对于表名，使用复数形式。表格包含行，每一行代表实体的一个实例。因此，表名应该使用复数形式，如`teachers`、`vehicles`或`departments`。有时我会做例外。例如，为了保留导入的 CSV 文件名，我会将它们作为表名，特别是在它们是一次性导入的情况下。

1.  注意长度。不同数据库应用程序允许的标识符名称的最大字符数不同：SQL 标准为 128 个字符，但 PostgreSQL 限制为 63 个字符，而旧版 Oracle 系统的最大值为 30。如果你编写的代码可能在其他数据库系统中重用，建议使用较短的标识符名称。

1.  在复制表时，请使用有助于以后管理的名称。一种方法是在创建副本时将`_YYYY_MM_DD`日期附加到表名中，例如`vehicle_parts_2021_04_08`。另一个好处是，表名将按日期排序。

## 使用约束控制列值

你可以通过使用某些约束进一步控制列接受的数据。列的数据类型广泛定义了它将接受的数据类型，例如整数与字符。额外的约束让我们根据规则和逻辑测试进一步指定可接受的值。通过约束，我们可以避免“垃圾进，垃圾出”的现象，即当低质量数据导致分析结果不准确或不完整时。设计良好的约束有助于维护数据质量，并确保表之间关系的完整性。

在第七章中，你学习了*主键*和*外键*，它们是最常用的两种约束。SQL 还具有以下约束类型：

1.  `CHECK` 仅允许布尔表达式求值为`true`的行

1.  `UNIQUE` 确保某一列或列组合中的值在每一行中都是唯一的

1.  `NOT NULL` 防止列中出现`NULL`值

我们可以通过两种方式添加约束：作为*列约束*或*表约束*。列约束仅适用于该列。在`CREATE TABLE`语句中，我们用列名和数据类型声明它，每次修改该列时都会进行检查。而表约束则适用于一个或多个列。在`CREATE TABLE`语句中，我们在定义完所有表列后立即声明它，并且每次修改表中行时都会进行检查。

让我们探索这些约束、它们的语法以及它们在表设计中的实用性。

### 主键：自然主键与代理主键

如第七章所述，*主键*是一个列或多个列的集合，其值唯一标识表中的每一行。主键是一种约束，它对组成主键的列或列集合施加两条规则：

+   每行的值必须是唯一的。

+   任何列都不能有缺失值。

在存储在仓库中的产品表中，主键可以是一个包含唯一产品代码的列。在第七章“通过键列关联表格”的简单主键示例中，我们的表格有一个由我们（用户）插入的整数构成的单一 ID 列的主键。通常，数据会暗示最佳路径，并帮助我们决定是否使用*自然键*或*替代键*作为主键。

#### 使用现有列作为自然键

自然键使用表格中现有的一列或多列，这些列符合主键的标准：每行唯一且永不为空。列中的值可以更改，只要新值不违反约束。

一个自然键可能是由地方车辆管理部门发放的驾驶证号码。在美国这样的一个政府管辖区内，我们可以合理地预期所有司机会在他们的驾驶证上获得一个唯一的 ID，我们可以将其存储为`driver_id`。然而，如果我们正在编制一个全国性的驾驶证数据库，我们可能无法做出这样的假设；多个州可能会独立地发放相同的 ID 代码。在这种情况下，`driver_id`列可能没有唯一值，不能作为自然键使用。作为解决方案，我们可以通过将`driver_id`与存储州名的列结合来创建一个*复合主键*，这将为每一行提供唯一的组合。例如，表中的这两行有一个唯一的`driver_id`和`st`列组合：

```
driver_id   st  first_name  last_name
----------  --  ----------  ---------
10302019    NY  Patrick     Corbin
10302019    FL  Howard      Kendrick
```

本章将探讨这两种方法，随着你处理数据时，留意那些适合自然键的值。部件号、序列号或书籍的 ISBN 号都是很好的例子。

#### 引入替代键列

*替代*键是一个单独的列，你可以用人工生成的值填充它；当一个表没有支持创建自然主键的数据时，我们可能会使用它。替代键可能是数据库自动生成的一个顺序号。我们已经通过序列数据类型和`IDENTITY`语法（在第四章的“自动递增整数”部分中介绍）做了这种操作。使用自动生成整数作为替代键的表可能如下所示：

```
id  first_name  last_name
--  ----------  ---------
 1  Patrick     Corbin
 2  Howard      Kendrick
 3  David       Martinez
```

一些开发人员喜欢使用*全局唯一标识符（UUID）*，它由 32 个十六进制数字组成，按连字符分组。UUID 通常用于标识计算机硬件或软件，格式如下所示：

```
2911d8a8-6dea-4a46-af23-d64175a08237
```

PostgreSQL 提供了 UUID 数据类型以及两个生成 UUID 的模块：`uuid-ossp`和`pgcrypto`。PostgreSQL 文档[`www.postgresql.org/docs/current/datatype-uuid.html`](https://www.postgresql.org/docs/current/datatype-uuid.html)是深入了解的一个良好起点。

#### 评估键类型的优缺点

使用任一类型主键都有充分的理由，但两者都有缺点。关于自然键需要考虑的要点包括以下几点：

+   数据已经存在于表中，因此你无需添加列来创建键。

+   由于自然键数据本身具有意义，它可以减少查询时表之间联接的需要。

+   如果你的数据发生变化，违反了键的要求——例如突然出现重复值——你将不得不更改表的设置。

这里是关于替代键需要考虑的几点：

+   因为替代键本身没有任何意义，并且其值独立于表中的数据，因此如果数据稍后发生变化，你不受键结构的限制。

+   键值保证是唯一的。

+   为替代键添加一列需要更多的空间。

在理想情况下，表应该有一个或多个列可以作为自然键，例如在产品表中使用唯一的产品代码。但现实中常常会遇到限制。例如，在员工表中，可能很难找到任何单一列，甚至多个列，能在逐行的基础上保持唯一性，作为主键。在无法重新考虑表结构的情况下，可能需要使用替代键。

#### 创建单列主键

让我们通过几个主键示例来分析。在第七章的《理解 JOIN 类型》中，你在 `district_2020` 和 `district_2035` 表上创建了主键来尝试不同的 `JOIN` 类型。实际上，这些都是替代键：在这两个表中，你创建了名为 `id` 的列作为键，并使用了关键字 `CONSTRAINT` `key_name` `PRIMARY KEY` 来声明它们为主键。

有两种方法来声明约束：作为列约束或作为表约束。在列表 8-1 中，我们尝试了这两种方法，在类似于前面提到的驾驶执照示例的表上声明主键。由于我们预期驾驶执照 ID 始终唯一，我们将使用该列作为自然键。

```
CREATE TABLE natural_key_example (
    1 license_id text CONSTRAINT license_key PRIMARY KEY,
    first_name text,
    last_name text
);

2 DROP TABLE natural_key_example;

CREATE TABLE natural_key_example (
    license_id text,
    first_name text,
    last_name text,
    3 CONSTRAINT license_key PRIMARY KEY (license_id)
);
```

列表 8-1：将单列自然键声明为主键

我们首先创建一个名为 `natural_key_example` 的表，并使用列约束语法 `CONSTRAINT` 声明 `license_id` 为主键 1，后跟约束名称和关键字 `PRIMARY KEY`。这种语法可以让你一目了然地了解哪个列被指定为主键。注意，你也可以省略 `CONSTRAINT` 关键字和主键名称，只使用 `PRIMARY KEY`：

```
license_id text PRIMARY KEY
```

在这种情况下，PostgreSQL 将自行为主键命名，采用表名后跟 `_pkey` 的命名约定。

接下来，我们使用 `DROP TABLE` 2 从数据库中删除表，以准备表约束示例。

要添加表约束，我们在列出所有列 3 后声明 `CONSTRAINT`，并在括号中列出我们想要用作主键的列。（同样，你可以省略 `CONSTRAINT` 关键字和主键名称。）在这个例子中，我们最终还是将 `license_id` 列作为主键。当你希望使用多列创建主键时，必须使用表约束语法；在这种情况下，你需要在括号中列出列，并用逗号分隔。我们稍后会详细探讨这个问题。

首先，让我们看看主键的特性——每一行唯一且没有 `NULL` 值——是如何保护数据完整性的。列表 8-2 中有两个 `INSERT` 语句。

```
INSERT INTO natural_key_example (license_id, first_name, last_name)
VALUES ('T229901', 'Gem', 'Godfrey');

INSERT INTO natural_key_example (license_id, first_name, last_name)
VALUES ('T229901', 'John', 'Mitchell');
```

列表 8-2：主键冲突示例

当你单独执行第一个 `INSERT` 语句时，服务器会顺利地将一行数据加载到 `natural_key_example` 表中。你尝试执行第二个时，服务器会返回错误：

```
ERROR:  duplicate key value violates unique constraint "license_key"
DETAIL:  Key (license_id)=(T229901) already exists.
```

在添加行之前，服务器检查表中是否已经存在 `T229901` 的 `license_id`。由于它已经存在，并且根据主键的定义，主键必须对每一行唯一，因此服务器拒绝了该操作。虚构的 DMV 规则规定，两个驾驶员不能拥有相同的驾照 ID，因此检查并拒绝重复数据是数据库执行该规则的一种方式。

#### 创建复合主键

如果单列不符合主键的要求，我们可以创建一个 *复合主键*。

我们将创建一个跟踪学生上学出勤情况的表。`student_id` 和 `school_day` 列的组合为每一行提供了一个唯一的值，记录了学生在某一天是否到校，这些信息存储在一个名为 `present` 的列中。要创建复合主键，你必须使用表约束语法，如列表 8-3 所示。

```
CREATE TABLE natural_key_composite_example (
    student_id text,
    school_day date,
    present boolean,
    CONSTRAINT student_key PRIMARY KEY (student_id, school_day)
);
```

列表 8-3：将复合主键声明为自然主键

在这里，我们传递两个（或更多）列作为参数，而不是一个。我们将通过尝试插入一行数据来模拟主键冲突，其中两个主键列 `student_id` 和 `school_day` 的值在表中并不唯一。逐个运行列表 8-4 中的 `INSERT` 语句（在 pgAdmin 中高亮显示它们后点击 **Execute/Refresh**）。

```
INSERT INTO natural_key_composite_example (student_id, school_day, present)
VALUES(775, '2022-01-22', 'Y');

INSERT INTO natural_key_composite_example (student_id, school_day, present)
VALUES(775, '2022-01-23', 'Y');

INSERT INTO natural_key_composite_example (student_id, school_day, present)
VALUES(775, '2022-01-23', 'N');
```

列表 8-4：复合主键冲突示例

前两个 `INSERT` 语句执行正常，因为在主键列的组合中没有值重复。但第三个语句会导致错误，因为它包含的 `student_id` 和 `school_day` 值与表中已存在的组合匹配：

```
ERROR:  duplicate key value violates unique constraint "student_key"
DETAIL:  Key (student_id, school_day)=(775, 2022-01-23) already exists.
```

你可以创建包含超过两列的复合主键。你可以使用的列数的限制取决于你的数据库。

#### 创建自增替代主键

正如你在第四章《自增整数》中学到的那样，PostgreSQL 数据库有两种方法可以向列添加自动增长的唯一值。第一种方法是将列设置为 PostgreSQL 特定的序列数据类型之一：`smallserial`、`serial` 和 `bigserial`。第二种方法是使用 `IDENTITY` 语法；由于它是 ANSI SQL 标准的一部分，我们将在示例中使用这种方法。

使用 `IDENTITY` 和 `smallint`、`integer`、`bigint` 等整数类型中的一种。对于主键来说，可能会诱使你通过使用 `integer` 来节省磁盘空间，它可以处理最大为 2,147,483,647 的数字。但是，许多数据库开发人员曾在深夜接到用户的紧急电话，询问为何应用程序崩溃，结果发现数据库尝试生成比数据类型的最大值更大的数字。因此，如果你的表有可能增长超过 21.47 亿行，明智的做法是使用 `bigint`，它可以接受高达 9.2 * quintillion 的数字。你可以设置并忘记它，就像在 清单 8-5 中定义的第一列那样。

```
CREATE TABLE surrogate_key_example (
  1 order_number bigint GENERATED ALWAYS AS IDENTITY,
    product_name text,
    order_time timestamp with time zone,
  2 CONSTRAINT order_number_key PRIMARY KEY (order_number)
);

3 INSERT INTO surrogate_key_example (product_name, order_time)
VALUES ('Beachball Polish', '2020-03-15 09:21-07'),
       ('Wrinkle De-Atomizer', '2017-05-22 14:00-07'),
       ('Flux Capacitor', '1985-10-26 01:18:00-07');

SELECT * FROM surrogate_key_example;
```

清单 8-5：使用 `IDENTITY` 声明 `bigint` 列作为替代键

清单 8-5 显示了如何使用 `IDENTITY` 语法声明一个自增的 `bigint` 列，名为 `order_number`，并将该列设置为主键 2。当你向表 3 中插入数据时，可以从列和值列表中省略 `order_number`。数据库将在每插入一行时为该列创建一个新值，该值将比已创建的最大值大 1。

运行 `SELECT * FROM surrogate_key_example;` 来查看该列是如何自动填充的：

```
order_number    product_name           order_time
------------ ------------------- ----------------------
           1 Beachball Polish    2020-03-15 09:21:00-07
           2 Wrinkle De-Atomizer 2017-05-22 14:00:00-07
           3 Flux Capacitor      1985-10-26 01:18:00-07
```

我们在日常购物的收据中看到这些自增的订单号。现在你知道是如何做到的了。

有几个值得注意的细节：如果你删除一行，数据库不会填补 `order_number` 序列中的空缺，也不会更改该列中的任何现有值。通常，它会将序列中最大的现有值加 1（尽管在某些操作中会有例外情况，包括从备份恢复数据库）。另外，我们使用了语法 `GENERATED ALWAYS AS IDENTITY`。正如第四章中讨论的那样，这可以防止用户在不手动覆盖设置的情况下向 `order_number` 插入值。通常，你希望防止这种干预，以避免出现问题。假设用户手动向现有的 `surrogate_key_example` 表的 `order_number` 列插入值 `4`。这个手动插入不会递增 `order_number` 列的 `IDENTITY` 序列；只有当数据库生成新值时，才会发生递增。因此，在下一行插入时，数据库也会尝试插入 `4`，因为它是序列中的下一个数字。结果将会是一个错误，因为重复值违反了主键约束。

然而，你可以通过重新启动`IDENTITY`序列来允许手动插入。你可能允许这样做，以防需要插入一个误删的行。示例 8-6 显示了如何向表中添加一个`order_number`为`4`的行，这个值是序列中的下一个值。

```
INSERT INTO surrogate_key_example
1 OVERRIDING SYSTEM VALUE
VALUES (4, 'Chicken Coop', '2021-09-03 10:33-07');

2 ALTER TABLE surrogate_key_example ALTER COLUMN order_number RESTART WITH 5;

3 INSERT INTO surrogate_key_example (product_name, order_time)
VALUES ('Aloe Plant', '2020-03-15 10:09-07');
```

示例 8-6：重新启动`IDENTITY`序列

你从一个包含关键字`OVERRIDING SYSTEM VALUE` 1 的`INSERT`语句开始。接下来，我们包括`VALUES`子句，并为`order_number`列在`VALUES`列表中指定整数`4`，这将覆盖`IDENTITY`限制。我们使用`4`，但我们也可以选择任何一个未在该列中存在的数字。

插入后，你需要重置`IDENTITY`序列，以便它从比你刚插入的`4`更大的数字开始。为此，使用`ALTER TABLE ... ALTER COLUMN`语句，其中包括关键字`RESTART WITH 5`。`ALTER TABLE`用于以各种方式修改表和列，更多内容将在第十章《检查和修改数据》中深入探讨。在这里，你用它来改变`IDENTITY`序列的起始数字；这样，下一个插入表中的行，`order_number`的值将是`5`。最后，插入一个新行并省略`order_number`的值，正如在示例 8-5 中所做的那样。

如果你再次从`surrogate_key_example`表中选择所有行，你会看到`order_number`列已经按预期填充：

```
order_number    product_name           order_time
------------ ------------------- ----------------------
           1 Beachball Polish    2020-03-15 09:21:00-07
           2 Wrinkle De-Atomizer 2017-05-22 14:00:00-07
           3 Flux Capacitor      1985-10-26 01:18:00-07
           4 Chicken Coop        2021-09-03 10:33:00-07
           5 Aloe Plant          2020-03-15 10:09:00-07
```

这个任务不一定是你需要经常处理的，但如果需要的话，知道怎么做是有帮助的。

### 外键

我们使用*外键*来建立表之间的关系。外键是一个或多个列，其值与另一个表的主键或其他唯一键中的值匹配。外键的值必须已经存在于它所引用的表的主键或其他唯一键中。如果不存在，则该值会被拒绝。通过这一约束，SQL 强制执行*参照完整性*——确保相关表中的数据不会变得无关或成为孤立数据。我们不会在一个表中得到与其他可以连接的表中的行没有关系的行。

示例 8-7 显示了一个假设数据库中的两个表，用于追踪机动车活动。

```
CREATE TABLE licenses (
    license_id text,
    first_name text,
    last_name text,
    1 CONSTRAINT licenses_key PRIMARY KEY (license_id)
);

CREATE TABLE registrations (
    registration_id text,
    registration_date timestamp with time zone,
    2 license_id text REFERENCES licenses (license_id),
    CONSTRAINT registration_key PRIMARY KEY (registration_id, license_id)
);

3 INSERT INTO licenses (license_id, first_name, last_name)
VALUES ('T229901', 'Steve', 'Rothery');

4 INSERT INTO registrations (registration_id, registration_date, license_id)
VALUES ('A203391', '2022-03-17', 'T229901');

5 INSERT INTO registrations (registration_id, registration_date, license_id)
VALUES ('A75772', '2022-03-17', 'T000001');
```

示例 8-7：外键示例

第一个表`licenses`使用驾驶员唯一的`license_id` 1 作为自然主键。第二个表`registrations`用于追踪车辆注册。一个许可证 ID 可能会与多个车辆注册相关联，因为每个持证驾驶员可以注册多辆车——这被称为*一对多关系*（第七章）。

通过 SQL，关系是这样表达的：在`registrations`表中，我们通过添加`REFERENCES`关键字，将`license_id`列指定为外键，后面跟上它引用的表名和列名。

现在，当我们向`registrations`插入一行时，数据库会检查插入到`license_id`中的值是否已经存在于`licenses`表的`license_id`主键列中。如果不存在，数据库会返回一个错误，这是非常重要的。如果`registrations`中的任何行与`licenses`中的行不对应，我们将无法编写查询来找到注册了该车辆的人。

为了查看此约束的实际效果，创建这两个表并逐一执行`INSERT`语句。第一个语句向`licenses` 3 中添加一行，其中`license_id`的值为`T229901`。第二个语句向`registrations` 4 中添加一行，其中外键包含相同的值。到目前为止，一切正常，因为该值在两个表中都存在。但是在第三次插入时，我们遇到错误，该插入尝试将一行插入到`registrations` 5 中，且其`license_id`值在`licenses`中不存在：

```
ERROR:  insert or update on table "registrations" violates foreign key constraint "registrations_license_id_fkey"
DETAIL:  Key (license_id)=(T000001) is not present in table "licenses".
```

产生的错误实际上是有帮助的：数据库通过阻止对不存在的许可证持有者进行注册来强制执行引用完整性。但它也表明了一些实际影响。首先，它影响我们插入数据的顺序。在另一个表中包含外键的表在没有相关记录之前不能添加数据，否则我们会遇到错误。在这个例子中，我们必须先创建一个驾驶执照记录，然后再插入相关的注册记录（如果你想一想，这就是你当地的机动车管理部门可能会做的事情）。

其次，当我们删除数据时，情况恰恰相反。为了保持引用完整性，外键约束阻止我们在删除`registrations`中的任何相关行之前，删除`licenses`中的一行，因为这样做会留下一个孤立的记录。我们必须先删除`registrations`中的相关行，然后再删除`licenses`中的记录。然而，ANSI SQL 提供了一种方法，通过使用`ON DELETE` `CASCADE`关键字自动处理这种操作顺序。

### 如何使用 CASCADE 自动删除相关记录

为了在删除`licenses`中的一行时自动删除`registrations`中的相关行，我们可以通过在定义外键约束时添加`ON DELETE CASCADE`来指定这种行为。

这是我们如何修改列表 8-7 中`CREATE TABLE`语句以创建`registrations`表，在`license_id`列的定义末尾添加关键字的方式：

```
CREATE TABLE registrations (
    registration_id text,
    registration_date date,
    license_id text REFERENCES licenses (license_id) ON DELETE CASCADE,
    CONSTRAINT registration_key PRIMARY KEY (registration_id, license_id)
);
```

删除`licenses`中的一行应该也会删除`registrations`中所有相关的行。这使我们能够删除驾驶执照，而不必先手动删除任何与其关联的注册记录。它还通过确保删除一个执照不会在`registrations`中留下孤立的行来维护数据完整性。

### `CHECK`约束

`CHECK`约束评估添加到列中的数据是否符合预期标准，我们通过逻辑测试来指定这些标准。如果标准不符合，数据库将返回错误。`CHECK`约束非常有价值，因为它可以防止列中加载无意义的数据。例如，棒球运动员的总击球数不应为负数，因此应该限制该数据为零或更大的值。或者，在大多数学校中，`Z`不是有效的课程成绩（尽管我那时勉强及格的代数成绩感觉像是 Z），因此我们可以插入只接受 A–F 值的约束。

与主键一样，我们可以在列级别或表级别实现`CHECK`约束。对于列约束，在`CREATE TABLE`语句中声明它，放在列名和数据类型之后：`CHECK (``logical expression``)`。作为表约束，使用语法`CONSTRAINT` `constraint_name` `CHECK (``logical expression``)`，在所有列定义之后。

Listing 8-8 展示了一个`CHECK`约束，应用于一个表中的两个列，我们可能会用这个表来追踪员工在组织中的角色和薪资。它使用了主键和`CHECK`约束的表约束语法。

```
CREATE TABLE check_constraint_example (
    user_id bigint GENERATED ALWAYS AS IDENTITY,
    user_role text,
    salary numeric(10,2),
    CONSTRAINT user_id_key PRIMARY KEY (user_id),
    1 CONSTRAINT check_role_in_list CHECK (user_role IN('Admin', 'Staff')),
    2 CONSTRAINT check_salary_not_below_zero CHECK (salary >= 0)
);
```

Listing 8-8: `CHECK`约束示例

我们创建表并将`user_id`列设置为自动递增的替代主键。第一个`CHECK` 1 测试输入到`user_role`列的值是否符合预定义的两个字符串之一，`Admin`或`Staff`，通过使用 SQL 中的`IN`运算符。第二个`CHECK` 2 测试输入到`salary`列的值是否大于或等于 0，因为负数金额是没有意义的。两个测试都是*布尔表达式*的例子，这是一种评估为真或假的语句。如果约束测试的值为`true`，则检查通过。

当插入或更新值时，数据库会将其与约束进行检查。如果任一列中的值违反了约束，或者即使违反了主键约束，数据库也会拒绝该更改。

如果我们使用表约束语法，我们还可以在一个`CHECK`语句中组合多个测试。例如，我们有一个与学生成绩相关的表。我们可以添加如下内容：

```
CONSTRAINT grad_check CHECK (credits >= 120 AND tuition = 'Paid')
```

注意，我们通过将两个逻辑测试括在括号中并用`AND`连接它们来组合这两个逻辑测试。在这里，两个布尔表达式必须都评估为`true`，整个检查才会通过。你也可以跨列进行值的测试，如下面的例子所示，我们希望确保商品的销售价格是原价的折扣，假设我们有两个列来存储这两个值：

```
CONSTRAINT sale_check CHECK (sale_price < retail_price)
```

在括号内，逻辑表达式检查销售价格是否小于零售价格。

### UNIQUE 约束

我们还可以通过使用 `UNIQUE` 约束来确保每一行中的列具有唯一的值。如果确保唯一值听起来与主键的目的相似，确实如此。但是，`UNIQUE` 有一个重要的区别：在主键中，值不能为 `NULL`，但 `UNIQUE` 约束允许列中存在多个 `NULL` 值。这在某些情况下非常有用，例如当我们并不总是拥有值，但希望确保现有值是唯一的。

为了展示 `UNIQUE` 的实用性，看看 列表 8-9 中的代码，这是一张用于跟踪联系人信息的表格。

```
CREATE TABLE unique_constraint_example (
    contact_id bigint GENERATED ALWAYS AS IDENTITY,
    first_name text,
    last_name text,
    email text,
    CONSTRAINT contact_id_key PRIMARY KEY (contact_id),
    1 CONSTRAINT email_unique UNIQUE (email)
);

INSERT INTO unique_constraint_example (first_name, last_name, email)
VALUES ('Samantha', 'Lee', 'slee@example.org');

INSERT INTO unique_constraint_example (first_name, last_name, email)
VALUES ('Betty', 'Diaz', 'bdiaz@example.org');

INSERT INTO unique_constraint_example (first_name, last_name, email)
2 VALUES ('Sasha', 'Lee', 'slee@example.org');
```

列表 8-9：`UNIQUE` 约束示例

在这个表中，`contact_id` 作为替代主键，唯一地标识每一行数据。但我们还有一个 `email` 列，这是与每个人的主要联系方式。我们希望这个列只包含唯一的电子邮件地址，但这些地址可能随着时间变化。所以，我们使用 `UNIQUE` 1 来确保每次添加或更新联系人的电子邮件时，不会重复已经存在的地址。如果我们尝试插入一个已存在的电子邮件 2，数据库将返回错误：

```
ERROR:  duplicate key value violates unique constraint "email_unique"
DETAIL:  Key (email)=(slee@example.org) already exists.
```

再次说明，错误信息表明数据库正在为我们工作。

### `NOT NULL` 约束

在第七章中，你学习了 `NULL`，它是一个特殊的 SQL 值，表示缺失数据或未知值。我们知道，主键的值不能为 `NULL`，因为主键需要唯一地标识表中的每一行。但在某些情况下，你可能希望在列中禁止空值。例如，在列出学校中每个学生的表格中，要求每一行的名字和姓氏列都必须填写是合理的。为了要求列中必须有值，SQL 提供了 `NOT NULL` 约束，它简单地禁止列接受空值。

列表 8-10 展示了 `NOT NULL` 语法。

```
CREATE TABLE not_null_example (
    student_id bigint GENERATED ALWAYS AS IDENTITY,
    first_name text NOT NULL,
    last_name text NOT NULL,
    CONSTRAINT student_id_key PRIMARY KEY (student_id)
);
```

列表 8-10：`NOT NULL` 约束示例

在这里，我们为 `first_name` 和 `last_name` 列声明了 `NOT NULL`，因为在跟踪学生信息的表格中，这些信息可能是必需的。如果我们尝试在表中执行 `INSERT` 并且没有为这些列提供值，数据库会通知我们违反了约束。

### 如何删除约束或稍后添加它们

你可以使用 `ALTER TABLE` 删除或稍后向现有表添加约束，就像你在本章“创建自增替代主键”中使用的命令来重置 `IDENTITY` 序列一样。

要删除主键、外键或 `UNIQUE` 约束，你需要编写以下格式的 `ALTER TABLE` 语句：

```
ALTER TABLE `table_name` DROP CONSTRAINT `constraint_name`;
```

要删除 `NOT NULL` 约束，语句作用于列，因此你必须使用额外的 `ALTER COLUMN` 关键字，如下所示：

```
ALTER TABLE `table_name` ALTER COLUMN `column_name` DROP NOT NULL;
```

让我们使用这些语句修改你刚刚创建的 `not_null_example` 表，如 列表 8-11 所示。

```
ALTER TABLE not_null_example DROP CONSTRAINT student_id_key;
ALTER TABLE not_null_example ADD CONSTRAINT student_id_key PRIMARY KEY (student_id);
ALTER TABLE not_null_example ALTER COLUMN first_name DROP NOT NULL;
ALTER TABLE not_null_example ALTER COLUMN first_name SET NOT NULL;
```

列表 8-11：删除和添加主键以及 `NOT NULL` 约束

一次执行一个语句。每次执行后，你可以通过在 pgAdmin 中单击表名，然后点击查询窗口上方的**SQL**标签，查看表定义的更改。（请注意，它将显示比你创建表时所使用的语法更详细的表定义语法。）

在第一个`ALTER TABLE`语句中，我们使用`DROP CONSTRAINT`移除名为`student_id_key`的主键。然后，我们使用`ADD CONSTRAINT`将主键重新添加。我们可以使用相同的语法向任何现有表添加约束。

在第三条语句中，`ALTER COLUMN`和`DROP NOT NULL`移除了`first_name`列的`NOT NULL`约束。最后，`SET NOT NULL`添加了该约束。

## 使用索引加速查询

就像一本书的索引帮助你更快找到信息一样，你也可以通过向表中的一列或多列添加*索引*—一种由数据库管理的独立数据结构—来加速查询。数据库使用索引作为快捷方式，而不是扫描每一行来查找数据。坦率地说，这只是 SQL 数据库中非平凡话题的一个简化版本。我们可以用好几章来深入探讨 SQL 索引的工作原理以及如何调优数据库性能，但在这里，我会提供关于使用索引的一般指导，并通过一个 PostgreSQL 特定的示例来展示它们的好处。

### B-树：PostgreSQL 的默认索引

你已经创建了几个索引，也许你并没有意识到。每次你添加主键或`UNIQUE`约束时，PostgreSQL（以及大多数数据库系统）会在包含该约束的列上创建一个索引。索引与表数据分开存储，并在你运行查询时自动访问（如果需要），并在每次添加、删除或更新行时更新。

在 PostgreSQL 中，默认的索引类型是*B 树索引*。它会在指定为主键或`UNIQUE`约束的列上自动创建，并且它也是使用`CREATE INDEX`语句时默认创建的索引类型。B 树（*balanced tree*的缩写）得名于其结构，因为在查找一个值时，它从树的顶部开始，通过分支向下搜索，直到找到该值。（当然，过程远比这更复杂。）B 树索引适用于可以排序并使用相等和范围运算符（如`<`、`<=`、`=`、`>=`、`>`和`BETWEEN`）进行搜索的数据。如果搜索字符串的开头没有通配符，它也适用于`LIKE`。例如：`WHERE chips LIKE 'Dorito%'`。

PostgreSQL 还支持其他索引类型，例如*广义倒排索引（GIN）*和*广义搜索树（GiST）*。每种索引有不同的用途，我将在后续章节中介绍它们，尤其是在全文搜索和使用几何类型进行查询时。

现在，让我们看看 B-tree 索引如何加速一个简单的搜索查询。在这个练习中，我们将使用一个包含超过 900,000 个纽约市街道地址的大型数据集，这些数据由 OpenAddresses 项目提供，网址为 [`openaddresses.io/`](https://openaddresses.io/)。包含数据的文件 *city_of_new_york.csv* 可以从 [`nostarch.com/practical-sql-2nd-edition/`](https://nostarch.com/practical-sql-2nd-edition/) 下载，和本书的所有资源一起提供。

下载文件后，使用 列表 8-12 中的代码创建 `new_york_addresses` 表并导入地址数据。由于 CSV 文件约为 50MB，导入过程将比您之前加载的小型数据集要慢。

```
CREATE TABLE new_york_addresses (
    longitude numeric(9,6),
    latitude numeric(9,6),
    street_number text,
    street text,
    unit text,
    postcode text,
    id integer CONSTRAINT new_york_key PRIMARY KEY
);

COPY new_york_addresses
FROM '*C:\YourDirectory\*city_of_new_york.csv'
WITH (FORMAT CSV, HEADER);
```

列表 8-12：导入纽约市地址数据

当数据加载完成后，运行一个快速的 `SELECT` 查询，目视检查是否有 940,374 行和七列。此数据的常见用途可能是搜索 `street` 列中的匹配项，因此我们将以此为例来探索索引性能。

#### 使用 EXPLAIN 基准测试查询性能

我们将通过使用 PostgreSQL 特定的 `EXPLAIN` 命令来测量添加索引前后的性能，该命令列出特定数据库查询的 *查询计划*。查询计划可能包括数据库计划如何扫描表，是否使用索引等信息。当我们添加 `ANALYZE` 关键字时，`EXPLAIN` 将执行查询并显示实际执行时间。

#### 记录一些控制执行时间

我们将使用 列表 8-13 中的三个查询来分析添加索引前后的查询性能。我们使用典型的 `SELECT` 查询，并在开始时包含带有 `WHERE` 子句的 `EXPLAIN ANALYZE`。这些关键字告诉数据库执行查询并显示查询过程的统计信息以及执行所花费的时间，而不是显示结果。

```
EXPLAIN ANALYZE SELECT * FROM new_york_addresses
WHERE street = 'BROADWAY';

EXPLAIN ANALYZE SELECT * FROM new_york_addresses
WHERE street = '52 STREET';

EXPLAIN ANALYZE SELECT * FROM new_york_addresses
WHERE street = 'ZWICKY AVENUE';
```

列表 8-13：索引性能基准查询

在我的系统上，第一个查询返回了以下统计信息，显示在 pgAdmin 输出窗格中：

```
Gather (cost=1000.00..15184.08 rows=3103 width=46) (actual time=9.000..388.448 rows=3336 loops=1)
  Workers Planned: 2
  Workers Launched: 2
  ->  Parallel Seq Scan on new_york_addresses  (cost=0.00..13873.78 1
     rows=1293 width=46) (actual time=2.362..367.258 rows=1112 loops=3)
        Filter: (street = 'BROADWAY'::text)
        Rows Removed by Filter: 312346
Planning Time: 0.401 ms
Execution Time: 389.232 ms 2
```

并非所有输出都与此相关，因此我不会解码全部内容，但有两行是相关的。第一行指出，为了找到 `street = 'BROADWAY'` 的所有行，数据库将对表进行顺序扫描 1。这是全表扫描的同义词：数据库将检查每一行并删除所有 `street` 不匹配 `BROADWAY` 的行。执行时间（在我的计算机上大约是 389 毫秒） 2 表示查询运行所需的时间。您的时间将取决于多个因素，包括您的计算机硬件。

对于测试，运行 列表 8-13 中的每个查询多次，并记录每个查询的最快执行时间。您会注意到，相同查询的执行时间在每次运行时会略有不同。这可能是由多个因素造成的，从服务器上其他进程的运行到查询在先前运行后数据被保存在内存中的效果。

#### 添加索引

现在，让我们看看添加索引如何改变查询的搜索方法和执行时间。列表 8-14 显示了使用 PostgreSQL 创建索引的 SQL 语句。

```
CREATE INDEX street_idx ON new_york_addresses (street);
```

列表 8-14：在 `new_york_addresses` 表上创建 B-tree 索引

请注意，这与创建约束的命令类似。我们给出 `CREATE INDEX` 关键字，后面跟上我们为索引选择的名称，在这个例子中是 `street_idx`。然后加上 `ON`，后面是目标表和列。

执行 `CREATE INDEX` 语句，PostgreSQL 会扫描 `street` 列中的值，并从中构建索引。我们只需要创建一次索引。当任务完成后，重新运行 列表 8-13 中的每个查询，并记录 `EXPLAIN ANALYZE` 提供的执行时间。以下是一个示例：

```
Bitmap Heap Scan on new_york_addresses  (cost=76.47..6389.39 rows=3103 width=46) (actual time=1.355..4.802 rows=3336 loops=1)
  Recheck Cond: (street = 'BROADWAY'::text)
  Heap Blocks: exact=2157
  ->  Bitmap Index Scan on street_idx  (cost=0.00..75.70 rows=3103 width=0) 1
      (actual time=0.950..0.950 rows=3336 loops=1)
        Index Cond: (street = 'BROADWAY'::text)
Planning Time: 0.109 ms
Execution Time: 5.113 ms 2
```

你注意到有什么变化吗？首先，我们看到数据库现在使用的是 `street_idx` 上的索引扫描，而不是逐行扫描。此外，查询速度明显更快了。2 表 8-1 显示了我在添加索引前后计算机上最快的执行时间（已四舍五入）。

表 8-1：衡量索引性能

| **查询筛选器** | **索引前** | **索引后** |
| --- | --- | --- |
| `WHERE street = 'BROADWAY'` | 92 毫秒 | 5 毫秒 |
| `WHERE street = '52 STREET'` | 94 毫秒 | 1 毫秒 |
| `WHERE street = 'ZWICKY AVENUE'` | 93 毫秒 | <1 毫秒 |

执行时间大大缩短，每个查询节省了近十分之一秒甚至更多。十分之一秒真的那么令人印象深刻吗？无论你是通过重复查询在数据中寻找答案，还是为成千上万的用户创建一个数据库系统，这些时间节省加起来都是可观的。

如果你需要从表中移除索引——也许是为了测试几种索引类型的性能——可以使用 `DROP INDEX` 命令，后面跟上要移除的索引名称。

### 使用索引时的考虑事项

你已经看到索引具有显著的性能优势，那么这是否意味着你应该为每个表中的列都添加索引呢？别急！索引是有价值的，但并不总是需要的。此外，索引会增大数据库，并对写入数据造成维护成本。以下是一些判断何时使用索引的建议：

+   查阅你所使用的数据库系统的文档，了解可用的索引类型以及在特定数据类型上使用哪些索引。例如，PostgreSQL 除了 B-tree 之外，还拥有五种索引类型。一个叫做 GiST 的索引类型特别适合本书后面讨论的几何数据类型。全文本搜索（你将在第十四章学习）也能从索引中受益。

+   考虑为你将在表连接中使用的列添加索引。在 PostgreSQL 中，主键默认会被索引，但相关表中的外键列则没有，这些列是添加索引的好目标。

+   外键上的索引有助于避免在级联删除时进行昂贵的顺序扫描。

+   为经常出现在查询`WHERE`子句中的列添加索引。如你所见，通过索引，搜索性能得到了显著提升。

+   使用`EXPLAIN ANALYZE`测试在不同配置下的性能。优化是一个过程！如果数据库没有使用某个索引，并且该索引不是主键或其他约束的支持索引，那么你可以删除它，以减少数据库的大小，并加速插入、更新和删除操作。

## 总结

通过本章中添加到你工具箱的工具，你已经准备好确保你构建或继承的数据库最适合你的数据收集和探索工作。至关重要的是要定义与数据和用户预期相匹配的约束，通过不允许不合理的值，确保填充所有值，以及建立表之间的正确关系。你还学会了如何让查询更快运行，并如何一致地组织你的数据库对象。这对你和其他共享你数据的人都是一种帮助。

本章结束了本书的第一部分，重点是为你提供深入 SQL 数据库所需的基本知识。接下来，我们将继续在这些基础上构建，探索更复杂的查询和数据分析策略。在下一章中，我们将使用 SQL 聚合函数评估数据集的质量，并从中获取可用信息。
