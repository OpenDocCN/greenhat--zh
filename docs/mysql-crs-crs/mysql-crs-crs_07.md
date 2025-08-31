# 第五章：连接数据库表

> 一条 SQL 查询走进酒吧，走向两个表，问道：“我能加入你们吗？”
> 
> —历史上最糟糕的数据库笑话

![](img/chapterart.png)

现在你已经学会了如何使用 SQL 从表中选择和过滤数据，接下来你将学习如何连接数据库表。*连接*表意味着从多个表中选择数据，并将它们合并到一个结果集中。MySQL 提供了不同类型连接的语法，比如内连接和外连接。在本章中，你将学习如何使用每种类型。

## 从多个表中选择数据

你想从数据库中检索的数据通常会存储在多个表中，且你需要将它们作为一个数据集返回，以便一次性查看所有数据。

我们来看一个例子。这个表，叫做`subway_system`，包含了世界上每个地铁系统的数据：

```
subway_system              city              country_code
------------------------   ----------------  ------------
Buenos Aires Underground   Buenos Aires      AR
Sydney Metro               Sydney            AU
Vienna U-Bahn              Vienna            AT
Montreal Metro             Montreal          CA
Shanghai Metro             Shanghai          CN
London Underground         London            GB
MBTA                       Boston            US
Chicago L                  Chicago           US
BART                       San Francisco     US
Washington Metro           Washington, D.C.  US
Caracas Metro              Caracas           VE
`--snip--`
```

前两列，`subway_system`和`city`，分别包含地铁的名称和它所在的城市。第三列，`country_code`，存储了两位字符的 ISO 国家代码。`AR`代表阿根廷，`CN`代表中国，等等。

第二个表，叫做`country`，有两列，`country_code`和`country`：

```
country_code   country
------------   -----------
AR             Argentina
AT             Austria
AU             Australia
BD             Bangladesh
BE             Belgium
`--snip--`
```

假设你想获取地铁系统的列表，并包括完整的城市和国家名称。这些数据分布在两个表中，因此你需要将它们连接起来，以便得到你想要的结果集。每个表都有相同的`country_code`列，所以你将使用它作为连接，编写一个 SQL 查询来连接这两个表（见 Listing 5-1）。

```
select subway_system.subway_system,
       subway_system.city,
       country.country
from   subway_system
inner join country
on     subway_system.country_code = country.country_code;
```

Listing 5-1：连接`subway_system`和`country`表

在`country`表中，`country_code`列是主键。在`subway_system`表中，`country_code`列是外键。回想一下，主键唯一标识表中的每一行，而外键用于与另一个表的主键进行连接。你使用`=`（等号）符号来指定要连接`subway_system`和`country`表中`country_code`列的所有相等值。

由于在这个查询中你从两个表中选择数据，因此每次引用列时最好指定该列所在的表，尤其是当两个表中有相同列名时。这样做有两个原因。首先，它能使 SQL 语句更易于维护，因为在 SQL 查询中，哪个列来自哪个表会立刻显而易见。其次，因为两个表都有一个名为`country_code`的列，如果不指定表名，MySQL 就不知道你要使用哪个列，并会返回错误信息。为避免这种情况，在`select`语句中，键入表名、一个点号，再加上列名。例如，在 Listing 5-1 中，`subway_system.city`指的是`subway_system`表中的`city`列。

当你运行这个查询时，它会返回所有地铁系统，并从`country`表中获取对应的国家名称：

```
subway_system               city                country
------------------------    ----------------    --------------
Buenos Aires Underground    Buenos Aires        Argentina
Sydney Metro                Sydney              Australia
Vienna U-Bahn               Vienna              Austria
Montreal Metro              Montreal            Canada
Shanghai Metro              Shanghai            China
London Underground          London              United Kingdom
MBTA                        Boston              United States
Chicago L                   Chicago             United States
BART                        San Francisco       United States
Washington Metro            Washington, D.C.    United States
Caracas Metro               Caracas             Venezuela
`--snip--`
```

请注意，`country_code` 列没有出现在结果连接中。这是因为您在查询中只选择了 `subway_system`、`city` 和 `country` 列。

## 表别名

为了节省编写 SQL 的时间，您可以为表名声明别名。*表别名*是表的短暂名称。以下查询返回与列表 5-1 相同的结果集：

```
select  **s**.subway_system,
        **s**.city,
        **c**.country
from    subway_system **s**
inner join country **c**
on      **s**.country_code = **c**.country_code;
```

您声明 `s` 为 `subway_system` 表的别名，`c` 为 `country` 表的别名。然后，在查询的其他部分引用列名时，您可以输入 `s` 或 `c` 来代替完整的表名。请记住，表别名仅对当前查询有效。

您还可以使用 `as` 来定义表别名：

```
select  s.subway_system,
        s.city,
        c.country
from    subway_system **as** s
inner join country **as** c
on      s.country_code = c.country_code;
```

无论是否使用 `as`，查询返回的结果是相同的，但不使用它可以减少输入量。

## 连接类型

MySQL 有多种不同类型的连接，每种连接都有自己的语法，概述如下表 5-1。

**表 5-1:** MySQL 连接类型

| **连接类型** | **描述** | **语法** |
| --- | --- | --- |
| 内连接 | 返回两个表中有匹配值的行。 | `inner join` `join` |

| 外连接 | 返回一个表中的所有行和第二个表中匹配的行。左连接返回左表中的所有行，右连接返回右表中的所有行。 | `left outer join` `left join`

`right outer join`

`right join` |

| 自然连接 | 基于两个表中相同的列名返回行。 | `natural join` |
| --- | --- | --- |
| 交叉连接 | 将一个表中的所有行与另一个表中的所有行匹配，并返回笛卡尔积。 | `cross join` |

让我们更深入地了解每种连接类型。

### 内连接

内连接是最常用的连接类型。在内连接中，只有两个表中都有匹配的数据时，才能检索到数据。

您在列表 5-1 中对 `subway_system` 和 `country` 表执行了内连接。返回的列表中没有孟加拉国和比利时的行。这些国家不在 `subway_system` 表中，因为它们没有地铁；因此，两个表中没有匹配的数据。

请注意，当您在查询中指定`inner join`时，`inner`这个词是可选的，因为这是默认的连接类型。以下查询执行内连接，并产生与列表 5-1 相同的结果：

```
select  s.subway_system,
        s.city,
 c.country
from    subway_system s
**join**    country c
on      s.country_code = c.country_code;
```

您可能会遇到使用 `inner join` 的 MySQL 查询，也有使用 `join` 的查询。如果您有现有的代码库或书面标准，最好遵循其中概述的做法。如果没有，我建议为清晰起见，包含 `inner` 这个词。

### 外连接

外连接显示一个表中的所有行以及第二个表中任何匹配的行。在列表 5-2 中，您选择所有国家，并显示这些国家的地铁系统（如果有的话）。

```
select  c.country,
        s.city,
        s.subway_system
from    subway_system s **right outer join** country c
on      s.country_code = c.country_code;
```

列表 5-2：执行右外连接

在这个查询中，`subway_system`表被认为是左表，因为它位于`outer join`语法的左侧，而`country`表是右表。由于这是一个*右*外连接，即使在`subway_system`表中没有匹配的行，这个查询仍然会返回`country`表中的所有行。因此，所有国家都会出现在结果集中，无论它们是否拥有地铁系统：

```
country                 city            subway_system
--------------------    ------------    ------------------------
United Arab Emirates    Dubai           Dubai Metro
Afghanistan             null            null
Albania                 null            null
Armenia                 Yerevan         Yerevan Metro
Angola                  null            null
Antarctica              null            null
Argentina               Buenos Aires    Buenos Aires Underground
`--snip--`
```

对于没有与`subway_system`表中的匹配行的国家，`city`和`subway_system`列将显示为 null 值。

与内连接一样，`outer`这个词是可选的；使用`left join`和`right join`将产生与其较长的等价语句相同的结果。

以下外连接返回的结果与列表 5-2 中的相同，但使用了`left outer join`语法：

```
select  c.country,
        s.city,
        s.subway_system
from    country c  **left outer join**  subway_system s
on      s.country_code = c.country_code;
```

在这个查询中，表的顺序与列表 5-2 中的顺序不同。`subway_system`表现在被列为最后一个表，成为右表。语法`country c left outer join subway_system s`等价于列表 5-2 中的`subway_system s right outer join country c`。无论使用哪种连接方式，只要表的顺序正确，就没有问题。

### 自然连接

MySQL 中的自然连接会在两个表有相同名称的列时自动连接它们。以下是基于两个表中都存在的列自动连接的语法：

```
select  *
from    subway_system s
**natural join** country c;
```

使用自然连接时，你可以避免内连接所需的额外语法。在列表 5-2 中，你需要包含`on s.country_code = c.country_code`来基于它们共同的`country_code`列连接表，但使用自然连接时，这个操作是自动完成的。这个查询的结果如下：

```
country_code    subway_system               city                country
------------    ------------------------    ------------        --------------
AR              Buenos Aires Underground    Buenos Aires        Argentina
AU              Sydney Metro                Sydney              Australia
AT              Vienna U-Bahn               Vienna              Austria
CA              Montreal Metro              Montreal            Canada
CN              Shanghai Metro              Shanghai            China
GB              London Underground          London              United Kingdom
US              MBTA                        Boston              United States
US              Chicago L                   Chicago             United States
US              BART                        San Francisco       United States
US              Washington Metro            Washington, D.C.    United States
VE              Caracas Metro               Caracas             Venezuela
`--snip--`
```

请注意，你使用`select *`通配符选择了所有表中的列。另外，尽管两个表都有`country_code`列，但 MySQL 的自然连接足够智能，仅在结果集中显示该列一次。

### 笛卡尔连接

MySQL 的`cross join`语法可以用来获取两个表的笛卡尔积。*笛卡尔积*是一个列出每一行与第二个表中每一行匹配的结果。例如，假设有一个餐厅的数据库，其中有两个表：`main_dish`和`side_dish`。每个表有三行和一列。

`main_dish`表如下所示：

```
main_item
---------
steak
chicken
ham
```

而`side_dish`表看起来像这样：

```
side_item
----------
french fries
rice
potato chips
```

这两个表的笛卡尔积将是所有主菜和配菜的可能组合的列表，可以使用`cross join`语法来检索：

```
select     m.main_item,
           s.side_item
from       main_dish m
**cross join** side_dish s;
```

这个查询与之前看到的查询不同，它没有基于列来连接表。没有使用主键或外键。以下是该查询的结果：

```
main_item   side_item
---------   ----------
ham         french fries
chicken     french fries
steak       french fries
ham         rice
chicken     rice
steak       rice
ham         potato chips
chicken     potato chips
steak       potato chips
```

由于`main_dish`表有三行，`side_dish`表也有三行，因此可能的组合总数为九个。

### 自连接

有时，将表与其自身连接是有益的，这称为自连接。与之前使用的特殊语法不同，您通过将相同的表名列出两次，并使用两个不同的表别名来执行自连接。

例如，以下表格名为`music_preference`，列出了音乐迷及其喜欢的音乐类型：

```
music_fan   favorite_genre
---------   --------------
Bob         Reggae
Earl        Bluegrass
Ella        Jazz
Peter       Reggae
Benny       Jazz
Bunny       Reggae
Sierra      Bluegrass
Billie      Jazz
```

为了将喜欢相同音乐类型的音乐迷配对，您将`music_preference`表与其自身连接，如列表 5-3 所示。

```
select a.music_fan,
       b.music_fan
from   **music_preference** **a**
inner join **music_preference** **b**
on (a.favorite_genre = b.favorite_genre)
where  a.music_fan **!=** b.music_fan
order by a.music_fan;
```

列表 5-3：`music_preference`表的自连接

`music_preference`表在查询中列出了两次，一次作为表`a`，一次作为表`b`。然后，MySQL 会将表`a`和表`b`连接起来，仿佛它们是不同的表。

在这个查询中，您在`where`子句中使用`!=`（不等于）语法，确保表`a`中`music_fan`列的值与表`b`中`music_fan`列的值不同。（请记住在第三章中，您可以在`select`语句中使用`where`子句，通过应用某些条件来筛选结果。）这样，音乐迷就不会与自己配对了。

列表 5-3 产生如下结果集：

```
music_fan  music_fan
---------  ---------
Benny      Ella
Benny      Billie
Billie     Ella
Billie     Benny
Bob        Peter
Bob        Bunny
Bunny      Bob
Bunny      Peter
Earl       Sierra
Ella       Benny
Ella       Billie
Peter      Bob
Peter      Bunny
Sierra     Earl
```

现在，音乐迷可以在他们的名字旁边的右侧列中找到其他喜欢相同音乐类型的粉丝。

## 联接语法的变体

MySQL 允许您以不同的方式编写 SQL 查询来完成相同的结果。了解不同的语法是一个好主意，因为您可能需要修改由其他人编写的代码，而这些人可能不会像您一样编写 SQL 查询。

### 括号

在连接列时，您可以选择使用括号，或者也可以不使用。这是一个不使用括号的查询：

```
select  s.subway_system,
        s.city,
        c.country
from    subway_system as s
inner join country as c
on      s.country_code = c.country_code;
```

这与下面的查询是等价的，它的作用是：

```
select  s.subway_system,
        s.city,
        c.country
from    subway_system as s
inner join country as c
on      **(**s.country_code = c.country_code**)**;
```

这两个查询返回相同的结果。

### 传统的内部连接

这个使用 SQL 旧语法编写的查询，相当于列表 5-1：

```
select  s.subway_system,
        s.city,
        c.country
from    subway_system as s,
        country as c
where   s.country_code = c.country_code;
```

这段代码没有使用`join`这个词；相反，它在`from`语句中列出了由逗号分隔的表名。

在编写查询时，使用列表 5-1 中显示的较新语法，但请记住，MySQL 仍然支持这种较旧的样式，您今天可能会在某些遗留代码中看到它的使用。

## 列别名

您在本章中早些时候阅读过表别名；现在您将为列创建别名。

在世界的一些地方，例如法国，地铁系统被称为*地铁*。让我们从`subway_system`表中选择法国城市的地铁系统，并使用列别名将标题显示为`metro`：

```
select  s.subway_system **as metro**,
        s.city,
        c.country
from    subway_system as s
inner join country as c
on      s.country_code = c.country_code
where   c.country_code = 'FR';
```

与表别名一样，您可以在 SQL 查询中使用`as`关键字，也可以省略它。无论哪种方式，查询的结果如下，现在`subway_system`列的标题已更改为`metro`：

```
metro           city        country
-----           --------    -------
Lille Metro     Lille       France
Lyon Metro      Lyon        France
Marseille Metro Marseille   France
Paris Metro     Paris       France
Rennes Metro    Rennes      France
Toulouse Metro  Toulouse    France
```

在创建表时，尽量为列标题命名具有描述性的名称，以便查询结果一目了然。在列名不够清晰的情况下，您可以使用列别名。

## 在不同数据库中连接表

有时多个数据库中会有相同名称的表，因此您需要告诉 MySQL 使用哪个数据库。可以通过几种不同的方式来做到这一点。

在这个查询中，`use` 命令（在第二章中介绍）告诉 MySQL 使用指定的数据库来执行接下来的 SQL 语句：

```
**use subway;**

select * from subway_system;
```

在第一行中，`use` 命令将当前数据库设置为 `subway`。然后，当您在下一行选择 `subway_system` 表的所有行时，MySQL 会知道从 `subway` 数据库中的 `subway_system` 表中提取数据。

这是第二种在 `select` 语句中指定数据库名称的方法：

```
select * from **subway.subway_system**;
```

在这个语法中，表名之前加上数据库名和一个句点。`subway.subway_system` 语法告诉 MySQL，您想从 `subway` 数据库中的 `subway_system` 表中选择数据。

这两种选项产生相同的结果集：

```
subway_system       city                        country_code
-----------------   -------------------------   ------------
Buenos Aires        Underground Buenos Aires    AR
Sydney Metro        Sydney                      AU
Vienna U-Bahn       Vienna                      AT
Montreal Metro      Montreal                    CA
Shanghai Metro      Shanghai                    CN
London Underground  London                      GB
`--snip--`
```

指定数据库和表名使您能够连接位于同一 MySQL 服务器上不同数据库中的表，如下所示：

```
select  s.subway_system,
        s.city,
        c.country
from    subway.subway_system as s
inner join location.country as c
on      s.country_code = c.country_code;
```

这个查询将位于 `location` 数据库中的 `country` 表与位于 `subway` 数据库中的 `subway_system` 表连接起来。

## 总结

在本章中，您学习了如何从两个表中选择数据，并使用 MySQL 提供的各种连接将数据显示在一个结果集中。在第六章，您将通过执行涉及多个表的更复杂连接来扩展这些知识。
