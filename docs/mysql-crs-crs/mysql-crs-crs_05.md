# 第三章：SQL 简介

![](img/chapterart.png)

要从 MySQL 数据库中选择数据，你将使用*结构化查询语言*（*SQL*）。SQL 是查询和管理像 MySQL 这样的关系数据库管理系统（RDBMS）中的数据的标准语言。

SQL 命令可以分为*数据定义语言*（*DDL*）语句和*数据操作语言*（*DML*）语句。到目前为止，你一直在使用 DDL 命令，如 `create database`、`create table` 和 `drop table` 来*定义*你的数据库和表格。

DML 命令则用于*操作*现有数据库和表中的数据。在本章中，你将使用 DML `select` 命令从表中检索数据。你还将学习如何为 MySQL 指定排序顺序，以及如何处理表列中的空值。

## 从表中查询数据

*查询*是从数据库表或一组表中请求信息。要指定你希望从表中检索的信息，使用 `select` 命令，如 列表 3-1 中所示。

```
**select** continent_id,
       continent_name,
       population
from   continent;
```

列表 3-1: 使用 `select` 显示 `continent` 表中的数据

这里你正在查询 `continent` 表（由 `from` 关键字指示），该表包含每个洲的名称和人口信息。通过使用 `select` 命令，你指定了要从 `continent_id`、`continent_name` 和 `population` 列返回数据。这被称为 `select` 语句。

列表 3-2 显示了执行 `select` 语句的结果。

```
continent_id  continent_name  population
------------  --------------  ----------
      1       Asia            4641054775
      2       Africa          1340598147
      3       Europe           747636026
      4       North America    592072212
      5       South America    430759766
      6       Australia         43111704
      7       Antarctica               0
```

列表 3-2: 执行 `select` 语句的结果

查询返回了所有七大洲的列表，显示了每个洲的 ID、名称和人口。

为了只显示一个洲的数据——例如亚洲，你可以在之前的代码末尾添加一个 `where` 子句：

```
select continent_id,
       continent_name,
       population
from   continent
**where  continent_name = 'Asia';**
```

`where` 子句通过对 `select` 语句应用条件来过滤结果。此查询查找表中 `continent_name` 列的值等于 `Asia` 的唯一一行，并显示以下结果：

```
continent_id  continent_name  population
------------  --------------  ----------
      1       Asia            4641054775
```

现在将 `select` 语句更改为仅选择 `population` 列：

```
select **population**
from   continent
where  continent_name = 'Asia';
```

查询现在返回一列（`population`）和一行（`Asia`）：

```
population
----------
4641054775
```

`continent_id` 和 `continent_name` 的值未出现在你的结果集中，因为你在 SQL 查询中没有选择它们。

## 使用通配符字符

SQL 中的星号通配符字符（`*`）允许你选择表中的所有列，而不必在查询中输入所有列名：

```
**select ***
from   continent;
```

此查询返回 `continent` 表中的所有三列。结果与 列表 3-1 相同，其中你单独列出了三列名称。

## 排序行

当你从数据库查询数据时，通常希望按特定顺序查看结果。为此，在 SQL 查询中添加 `order by` 子句：

```
select continent_id,
       continent_name,
 population
from   continent
**order by continent_name;**
```

在这里，你选择了`continent`表中的所有列，并根据`continent_name`列中的值按字母顺序排列结果。

结果如下：

```
continent_id  continent_name  population
------------  --------------  ----------
      2       Africa          1340598147
      7       Antarctica               0
      1       Asia            4641054775
      6       Australia         43111704
      3       Europe           747636026
      4       North America    592072212
      5       South America    430759766
```

添加`order by` `continent_name`将按字母顺序列出结果，而不管`continent_id`或`population`列的值是什么。MySQL 按字母顺序排序行，因为`continent_name`被定义为存储字母数字字符的列。

MySQL 也可以对整数数据类型的列进行排序。你可以通过`asc`和`desc`关键字指定结果是按升序（从低到高）还是降序（从高到低）排序：

```
select continent_id,
       continent_name,
       population
from   continent;
**order by population desc;**
```

在这个例子中，你让 MySQL 根据`population`排序，并按降序（`desc`）排列值。

结果如下：

```
continent_id  continent_name  population
------------  --------------  ----------
      1       Asia            4641054775
      2       Africa          1340598147
      3       Europe           747636026
      4       North America    592072212
      5       South America    430759766
      6       Australia         43111704
      7       Antarctica               0
```

查询返回所有七行数据，因为你没有使用`where`子句来过滤结果。现在数据按`population`列的降序排列，而不是按`continent_name`列的字母顺序排列。

## SQL 代码格式化

到目前为止，你看到的 SQL 格式很好，容易阅读：

```
select continent_id,
       continent_name,
       population
from   continent;
```

注意列名和表名是如何垂直对齐的。像这样以整洁、可维护的格式编写 SQL 语句是个好主意，但 MySQL 也允许你以不太规范的方式编写 SQL 语句。例如，你可以将示例 3-1 中的代码写成一行：

```
select continent_id, continent_name, population from continent;
```

或者你也可以将`select`和`from`语句分开写，如下所示：

```
select continent_id, continent_name, population
from continent;
```

两种选项返回与示例 3-1 相同的结果，尽管你的 SQL 可能对于其他人来说稍微难以理解。

可读的代码对于代码库的可维护性非常重要，即使 MySQL 会正常运行不太可读的代码。虽然可能会有诱惑，只是让代码运行起来然后继续做下一个任务，但编写代码仅仅是你的工作的一部分。花时间让代码更易读，你的未来自己（或将来会维护代码的人）会感谢你。

让我们来看一下你可能会遇到的其他 SQL 代码约定。

### 大写关键字

一些开发者使用大写字母书写 MySQL 关键字。例如，他们可能会像这样将示例 3-1 中的`select`和`from`写成大写：

```
SELECT continent_id,
       continent_name,
       population
FROM   continent;
```

类似地，一些开发者可能会将`create table`语句中的多个词组写成大写：

```
CREATE TABLE dog
(
    dog_id            int,
    dog_name          varchar(50) UNIQUE,
    owner_id          int,
    breed_id          int,
    veterinarian_id   int,
    PRIMARY KEY (dog_id),
    FOREIGN KEY (owner_id) REFERENCES owner(owner_id),
    FOREIGN KEY (breed_id) REFERENCES breed(breed_id),
    FOREIGN KEY (veterinarian_id) REFERENCES veterinarian(veterinarian_id)
);
```

在这里，`create` `table`、`unique`、`primary` `key`、`foreign` `key`和`references`都已被大写化以提高可读性。一些 MySQL 开发者也会将数据类型`int`和`varchar`大写。如果你觉得使用大写字母对关键字有帮助，可以随意这样做。

如果你正在处理现有的代码库，最好保持一致，并遵循已有的编码风格。如果你在一家公司工作，且公司有正式的编码风格规范，你应当遵循这些规范。否则，选择最适合你的方式。无论如何，你都会得到相同的结果。

### 反引号

如果你维护其他开发者编写的 SQL 代码，你可能会遇到使用反引号（`` ` ``）的 SQL 语句：

```
select `continent_id`,
       `continent_name`,
       `population`
from   `continent`;
```

这个查询选择了`continent`表中的所有列，并将列名和表名用反引号括起来。在这个例子中，即使没有反引号，语句也可以正常运行。

反引号允许你绕过 MySQL 在命名表和列时的一些规则。例如，你可能注意到，当列名包含多个单词时，我使用了下划线连接这些单词，而不是空格，比如`continent_id`。然而，如果你将列名用反引号括起来，你就不需要使用下划线了；你可以将列命名为`continent id`，而不是`continent_id`。

通常，如果你将一个表或列命名为`select`，你会收到一个错误信息，因为`select`是 MySQL 的*保留字*；也就是说，它在 SQL 中有一个专门的含义。然而，如果你将`select`用反引号括起来，查询将不会报错：

```
select * from `select`;
```

在这个`select * from`语句中，你正在选择`select`表中的所有列。

尽管 MySQL 会运行像这样的代码，但我建议避免使用反引号，因为没有它们你的代码会更易于维护且更易于输入。未来，其他需要更改此查询的开发者可能会被名为`select`的表或表名中带有空格的名称所困惑。你的目标应该始终是编写简单且结构良好的代码。

### 代码注释

注释是你可以添加到代码中的解释性文本，以帮助理解代码。它们能帮助你或其他开发者在未来维护代码。通常，注释用于阐明复杂的 SQL 语句，或者指出表或数据中异常的部分。

要添加单行注释，请使用两个连字符后跟一个空格（`--`）。这种语法告诉 MySQL 该行的其余部分是注释。

这个 SQL 查询在顶部包含了一条单行注释：

```
**-- This SQL statement shows the highest-populated continents at the top**
select continent_id,
       continent_name,
       population
from   continent
order by population desc;
```

你可以使用相同的语法在 SQL 语句的末尾添加注释：

```
select continent_id,
       continent_name, **-- Continent names are displayed in English**
       population
from   continent
order by population desc;
```

在这段代码中，`continent_name`列的注释让开发者知道列中的名称是用英语显示的。

要添加多行注释，请在注释的开头使用`/*`，在结尾使用`*/`：

```
**/***
**This query retrieves data for all the continents in the world.**
**The population of each continent is updated in this table yearly.**
***/**
select ***** from continent;
```

这个两行的注释解释了查询并说明了表的更新频率。

内联注释的语法类似：

```
select 3.14 **/* The value of pi */** * 81;
```

内联注释有一些特殊用途。例如，如果你维护由他人编写的代码，你可能会注意到看起来像是神秘的内联注释：

```
select **/*+ no_index(employee idx1) */**
       employee_name
from   employee;
```

第一行中的`/*+ no_index(employee idx1) */`是一个*优化器提示*，它使用了带加号的内联注释语法`/*`。

当你运行查询时，MySQL 的查询优化器会尝试确定执行查询的最快方法。例如，如果`employee`表上有索引，使用索引来访问数据会更快，还是因为表中行数太少，使用索引反而会变得更慢？

查询优化器通常能很好地生成查询计划，比较它们，然后执行最快的计划。但有时你可能希望给出自己的指示——提示——来指定执行查询的最有效方法。

前面的示例中的提示告诉优化器不要在`employee`表上使用`idx1`索引。

查询优化是一个庞大的话题，我们仅仅触及了表面，但如果你遇到`/*+` . . . `*/`语法，只需要知道它允许你向 MySQL 提供提示。

正如你所看到的，一个恰当的位置、描述性的注释将节省时间并减少烦恼。对你为什么使用某种方法的简短解释，可以避免其他开发者重复研究相同的问题，或者在你自己维护代码时帮助你回忆起相关内容。然而，要避免添加显而易见的注释；如果某个注释不会让 SQL 更加易懂，就不应该添加它。同时，随着代码的更新，也要更新注释。不再相关且未更新的注释没有任何作用，可能会让其他开发者或将来的你产生困惑。

## 空值

如第二章中所讨论的，`null`表示缺失或未知的值。MySQL 有特殊的语法，包括`is null`和`is not null`，用于处理数据中的 null 值。

假设有一个名为`unemployed`的表，它有两列：`region_id`和`unemployed`。每一行表示一个地区，告诉你该地区有多少人失业。使用`select * from`查看完整表格，如下所示：

```
select *
from   unemployed;
```

结果如下：

```
region_id   unemployed
---------   ----------
    1          2218457
    2           137455
    3             null
```

区域 1 和区域 2 已报告其失业人数，但区域 3 尚未报告，因此区域 3 的`unemployed`列被设置为`null`值。你不会想在这里使用`0`，因为那样意味着区域 3 没有失业的人。

要仅显示那些`unemployed`值为`null`的地区的行，可以在`where`子句中使用`is null`：

```
select *
from   unemployed
**where  unemployed is null;**
```

结果是：

```
region   unemployed
------   ----------
   3           null
```

另一方面，如果你想要*排除*那些`unemployed`值为`null`的行，只查看已经报告的数据，可以在`where`子句中将`is null`替换为`is not null`，如下面所示：

```
select *
from   unemployed
where  unemployed **is not null;**
```

结果如下：

```
region   unemployed
------   ----------
   1        2218457
   2         137455
```

使用此语法与 null 值结合，可以帮助你筛选表格数据，从而让 MySQL 仅返回最有意义的结果。

## 总结

在本章中，你学习了如何使用`select`语句和通配符来从表格中检索数据，并且你看到 MySQL 可以按照你指定的顺序返回结果。你还学习了如何格式化代码以提高可读性和清晰度，包括在 SQL 语句中添加注释以便于代码的维护。最后，你还了解了如何处理数据中的 null 值。

第四章讲述的是 MySQL 数据类型。到目前为止，你创建的表主要使用`int`来接受整数数据，或者使用`varchar`来接受字符数据。接下来，你将学习更多关于 MySQL 数据类型的内容，包括数值型和字符型数据类型，以及日期类型和非常大的值类型。
