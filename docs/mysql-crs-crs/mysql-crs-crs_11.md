# 第九章：插入、更新和删除数据

![](img/chapterart.png)

在本章中，你将学习如何插入、更新和删除表中的数据。你将练习将数据从一个表插入到另一个表，使用查询来更新或删除表中的数据，并创建一个在插入行时自动递增数字值的表。

## 插入数据

到目前为止，你一直是在查询表中的数据。那么，这些数据是如何最初进入表中的呢？通常，你是通过`insert`语句来插入数据的。

使用`insert`语句向表中添加行称为*填充*表格。你需要指定表的名称、你要插入值的列名，以及你想要插入的值。

在这里，你向`arena`表插入了一行数据，其中包含关于不同竞技场名称、位置和容量的信息：

```
❶ insert into arena
    (
  ❷ arena_id,
    arena_name,
    location,
    seating_capacity
    )
❸ values
    (
    1,
  ❹ 'Madison Square Garden',
    'New York',
    20000
    );
```

首先，你需要指定你要将一行数据插入到`arena`表中❶，并且你的数据将填入`arena_id`、`arena_name`、`location`和`seating_capacity`这几列❷。然后，你在`values`关键字下列出你想插入的值，顺序与列名一致❸。你需要将`Madison Square Garden`和`New York`这两个值用引号括起来，因为它们是字符串❹。

当你运行此`insert`语句时，MySQL 会返回`1 row(s) affected`的信息，告诉你表中已插入了一行数据。

然后，你可以查询你的`arena`表，确认新插入的行符合预期：

```
select * from arena;
```

结果是：

```
arena_id  arena_name             location  seating_capacity
--------  ---------------------  --------  ----------------
    1     Madison Square Garden  New York       20000
```

行已成功插入，列及其值如你所预期的那样显示。

### 插入`null`值

当你想要插入一个`null`值到某列时，你有两个选择。首先，你可以列出该列名，并使用`null`关键字作为要插入的值。例如，如果你想向`arena`表中添加一行`Dean Smith Center`的数据，但不知道它的座位容量，你可以像这样编写`insert`语句：

```
insert into arena
    (
    arena_id,
 arena_name,
    location,
 **seating_capacity**
    )
values
    (
    2,
    'Dean Smith Center',
    'North Carolina',
    **null**
    );
```

第二种选择是完全省略列名。作为前面`insert`语句的替代方案，你可以将`seating_capacity`列从列名列表中省略，并且在值列表中不为该列提供任何值：

```
insert into arena
    (
    arena_id,
    arena_name,
    location
    )
values
    (
    2,
    'Dean Smith Center',
    'North Carolina'
    );
```

由于你没有向`seating_capacity`列插入任何值，MySQL 将默认将其设置为`null`。你可以通过以下查询查看插入的行：

```
select  *
from    arena
where   arena_id = 2;
```

结果是：

```
arena_id  arena_name         location        seating_capacity
--------  -----------------  --------        ----------------
    2     Dean Smith Center  North Carolina         null
```

无论你采用哪种方法，`seating_capacity`列的值都会被设置为`null`。

如果在创建表时，`seating_capacity`列已被定义为`not null`，则无论采用哪种方法，你都不允许插入`null`值（参见第二章）。

### 一次插入多行数据

当你想要插入多行数据时，你可以选择一次插入一行，或者将它们作为一组插入。我们先从第一种方法开始。以下是如何通过单独的`insert`语句向`arena`表插入三条数据：

```
insert into arena (arena_id, arena_name, location, seating_capacity)
values (3, 'Philippine Arena', 'Bocaue', 55000);

insert into arena (arena_id, arena_name, location, seating_capacity)
values (4, 'Sportpaleis', 'Antwerp', 23359);

insert into arena (arena_id, arena_name, location, seating_capacity)
values (5, 'Bell Centre', 'Montreal', 22114);
```

你也可以通过将所有三行合并成一个 `insert` 语句来达到相同的效果：

```
insert into arena (arena_id, arena_name, location, seating_capacity)
values (3, 'Philippine Arena', 'Bocaue', 55000),
       (4, 'Sportpaleis', 'Antwerp', 23359),
       (5, 'Bell Centre', 'Montreal', 22114);
```

若要一次插入多行，需将每行的值用括号括起来，并在每组值之间使用逗号。MySQL 将会把所有三行插入到表中，并给出消息 `3 row(s) affected`，表示所有三行已成功插入。

### 不列出列名的插入

你也可以在不指定列名的情况下向表中插入数据。由于你要插入四个值，而 `arena` 表只有四列，你可以用不列出列名的 `insert` 语句替代列出列名的语句：

```
insert into arena
values (6, 'Staples Center', 'Los Angeles', 19060);
```

MySQL 能够确定将值插入到哪些列中，因为你提供的数据顺序与表中的列顺序相同。

虽然省略列名可以减少一些打字工作，但最佳实践是列出它们。将来你可能会向 `arena` 表中添加一个第五列。如果不列出列名，进行该更改时会破坏你的 `insert` 语句，因为你会试图将四个值插入到一个有五个列的表中。

### 插入数字序列

你可能想将连续的数字插入到表的某个列中，比如在 `arena` 表中，`arena_id` 列的第一行应该为 `1`，第二行应该为 `2`，第三行应该为 `3`，以此类推。MySQL 提供了一种简单的方法，让你通过定义带有 `auto_increment` 属性的列来实现这一点。`auto_increment` 属性特别适用于主键列——即唯一标识表中行的列。

我们来看它是如何工作的。从你到目前为止创建的 `arena` 表中选择所有内容：

```
select * from arena;
```

结果是：

```
arena_id  arena_name             location        seating_capacity
--------  ---------------------  --------------  ----------------
    1     Madison Square Garden  New York             20000
    2     Dean Smith Center      North Carolina        null
    3     Philippine Arena       Bocaue               55000
    4     Sportpaleis            Antwerp              23359
    5     Bell Centre            Montreal             22114
    6     Staples Center         Los Angeles          19060
```

你可以看到每个竞技场都有自己的 `arena_id`，它比之前插入的竞技场的 `arena_id` 大 1。

当你在 `arena_id` 列中插入值时，你需要先找到表中已存在的最大 `arena_id`，然后在插入下一行时将其加 1。例如，当你为 `Staples Center` 插入行时，你硬编码了 `arena_id` 为 `6`，因为前一个 `arena_id` 是 `5`：

```
insert into arena (arena_id, arena_name, location, seating_capacity)
values (**6**, 'Staples Center', 'Los Angeles', 19060);
```

这种方法在实际的生产数据库中效果不好，因为在生产环境下，很多新的行会迅速被创建。一个更好的方法是让 MySQL 通过在创建表时定义带有 `auto_increment` 的 `arena_id` 列来为你处理这项工作。我们来试试吧。

删除 `arena` 表，并使用 `auto_increment` 重新创建它以适配 `arena_id` 列：

```
drop table arena;

create table arena (
    arena_id          int            primary key       **auto_increment**,
    arena_name        varchar(100),
    location          varchar(100),
    seating_capacity  int
);
```

现在，当你向表中插入行时，你就不需要再处理 `arena_id` 列的数据插入了。你只需要插入其他列的数据，MySQL 会自动为每个新插入的行递增 `arena_id` 列。你的 `insert` 语句应该是这样的：

```
insert into arena (arena_name, location, seating_capacity)
values ('Madison Square Garden', 'New York', 20000);

insert into arena (arena_name, location, seating_capacity)
values ('Dean Smith Center', 'North Carolina', null);

insert into arena (arena_name, location, seating_capacity)
values ('Philippine Arena', 'Bocaue', 55000);

insert into arena (arena_name, location, seating_capacity)
values ('Sportpaleis', 'Antwerp', 23359);

insert into arena (arena_name, location, seating_capacity)
values ('Bell Centre', 'Montreal', 22114);

insert into arena (arena_name, location, seating_capacity)
values ('Staples Center', 'Los Angeles', 19060);
```

你没有在列的列表中列出`arena_id`作为一列，也没有在值的列表中提供`arena_id`的值。看看在 MySQL 运行你的`insert`语句后表中的行：

```
select * from arena;
```

结果如下：

```
arena_id  arena_name             location        seating_capacity
--------  ---------------------  --------------  ----------------
    1     Madison Square Garden  New York             20000
    2     Dean Smith Center      North Carolina        null
    3     Philippine Arena       Bocaue               55000
    4     Sportpaleis            Antwerp              23359
    5     Bell Centre            Montreal             22114
    6     Staples Center         Los Angeles          19060
```

如你所见，MySQL 自动为`arena_id`列的值进行了递增。

每个表格只能定义一个`auto_increment`列，并且该列必须是主键列（或主键的一部分）。

当向一个定义了`auto_increment`的列插入值时，MySQL 会始终插入一个更大的数字，但这些数字之间可能会有间隙。例如，你的表格可能会出现`arena_id`为 22、23，然后是 29 的情况。造成这种情况的原因与数据库使用的存储引擎、MySQL 服务器的配置以及其他超出本书范围的因素有关，因此请记住，定义为`auto_increment`的列始终会生成递增的数字列表。

### 使用查询插入数据

你可以基于查询返回的值将数据插入到表格中。例如，假设`large_building`表中有你想添加到`arena`表的数据。`large_building`表是使用以下数据类型创建的：

```
create table large_building
       (
       building_type      varchar(50),
 building_name      varchar(100),
       building_location  varchar(100),
       building_capacity  int,
       active_flag        bool
);
```

它包含以下数据：

```
building_type  building_name      building_location  building_capacity  active_flag
-------------  -----------------  -----------------  -----------------  -----------
Hotel          Wanda Inn          Cape Cod                  125             1
Arena          Yamada Green Dome  Japan                    20000            1
Arena          Oracle Arena       Oakland                  19596            1
```

对你来说，你并不关心表格中的第一行数据，因为`Wanda Inn`是一个酒店，而不是一个竞技场。你可以编写查询，从`large_building`表中的其他行返回竞技场的数据，如下所示：

```
select  building_name,
        building_location,
        building_capacity
from    large_building
where   building_type = 'Arena'
and     active_flag is true;
```

结果如下：

```
building_name      building_location  building_capacity
-----------------  -----------------  -----------------
Yamada Green Dome  Japan                   20000
Oracle Arena       Oakland                 19596
```

然后，你可以使用该查询作为`insert`语句的基础，将这些行数据插入到`arena`表中：

```
insert into arena (
        arena_name,
        location,
        seating_capacity
)
select  building_name,
        building_location,
        building_capacity
from    large_building
where   building_type = 'Arena'
and     active_flag is true;
```

MySQL 将从查询中返回的两行数据插入到`arena`表中。你可以查询`arena`表以查看新插入的行：

```
select * from arena;
```

这是包含新行的结果：

```
arena_id  arena_name             location        seating_capacity
--------  ---------------------  --------------  ----------------
    1     Madison Square Garden  New York             20000
    2     Dean Smith Center      North Carolina        null
    3     Philippine Arena       Bocaue               55000
    4     Sportpaleis            Antwerp              23359
    5     Bell Centre            Montreal             22114
    6     Staples Center         Los Angeles          19060
    7     Yamada Green Dome      Japan                20000
    8     Oracle Arena           Oakland              19596
```

`insert`语句将竞技场`7`和`8`添加到`arena`表中的现有数据中。

### 使用查询创建并填充新表

`create table as`语法允许你在一步操作中创建并填充表格。在这里，你创建了一个名为`new_arena`的新表，并同时插入行数据：

```
create table new_arena as
select  building_name,
        building_location,
        building_capacity
from    large_building
where   building_type = 'Arena'
and     active_flag is true;
```

该语句根据前面的`large_building`查询结果创建了一个名为`new_arena`的表格。现在查询新表：

```
select * from new_arena;
```

结果如下：

```
building_name      building_location  building_capacity
-----------------  -----------------  -----------------
Yamada Green Dome  Japan                    20000
Oracle Arena       Oakland                  19596
```

`new_arena`表与`large_building`表具有相同的列名和数据类型。你可以使用`desc`关键字描述表格，以确认数据类型：

```
desc new_arena;
```

结果如下：

```
Field              Type          Null  Key  Default  Extra
-----------------  ------------  ----  ---  -------  -----
building_name      varchar(100)  YES          null
building_location  varchar(100)  YES          null
building_capacity  int           YES          null
```

你还可以使用`create table`复制一个表格。例如，你可以通过复制`arena`表并将新表命名为`arena_`，后面加上当前日期来保存`arena`表的当前状态，如下所示：

```
create table arena_20241125 as
select * from arena;
```

在你添加或删除`arena`表的列之前，你可能希望先确保你已将原始数据保存在第二个表格中。当你即将对表格进行重大更改时，这一点尤其有用，但如果表格非常大，可能不切实际去复制整个表格。

## 更新数据

一旦你的表中有了数据，你可能会想要随着时间推移对其进行修改。MySQL 的`update`语句允许你修改现有数据。

场馆因名称变更而臭名昭著，你表中的场馆也不例外。在这里，你通过`update`语句将`arena_id 6`的`arena_name`值从`Staples Center`更改为`Crypto.com Arena`：

```
update  arena
set     arena_name = 'Crypto.com Arena'
where   arena_id = 6;
```

首先，你使用`set`关键字来设置表中列的值。在这里，你将`arena_name`列的值设置为`Crypto.com Arena`。

接下来，你在`where`子句中指定要更新的行。在这种情况下，你选择根据`arena_id`列值为`6`来更新行，但你也可以根据其他列来更新相同的行。例如，你可以根据`arena_name`列来更新这一行：

```
update  arena
set     arena_name = 'Crypto.com Arena'
**where   arena_name = 'Staples Center'**;
```

或者，由于你在洛杉矶只列出了一个场馆，你可以使用`location`列来更新这一行：

```
update  arena
set     arena_name = 'Crypto.com Arena'
**where**   **location = 'Los Angeles';**
```

精心编写`where`子句非常重要，因为任何符合该子句中指定条件的行都将被更新。例如，如果有五个场馆的`location`为`Los Angeles`，那么这个`update`语句将把这五个场馆的名称全部更改为`Crypto.com Arena`，无论这是否是你原本的意图。

通常，最好根据主键列来更新行。当你创建`arena`表时，你已将`arena_id`列定义为表的主键。这意味着表中会有唯一的一行对于`arena_id`为`6`，因此如果你使用语法`where arena_id = 6`，你可以确保只更新这一行。

在`where`子句中使用主键也是最佳实践，因为主键列是已建立索引的。已建立索引的列通常在查找表中的行时比未建立索引的列要快。

### 更新多个行

要更新多个行，你可以使用匹配多行的`where`子句。在这里，你更新了所有`arena_id`大于`3`的场馆的座位容量：

```
update  arena
set     seating_capacity = 20000
where   arena_id > 3;
```

MySQL 将场馆`4`、`5`和`6`的`seating_capacity`值更新为 20,000。

如果你完全移除`where`子句，那么表中的所有行都会被更新：

```
update  arena
set     seating_capacity = 15000;
```

如果现在执行`select * from arena`，你会发现所有场馆的座位容量都是 15,000：

```
arena_id  arena_name             location        seating_capacity
--------  ---------------------  --------------  ----------------
    1     Madison Square Garden  New York             15000
    2     Dean Smith Center      North Carolina       15000
    3     Philippine Arena       Bocaue               15000
    4     Sportpaleis            Antwerp              15000
    5     Bell Centre            Montreal             15000
    6     Crypto.com Arena       Los Angeles          15000
```

在这个例子中，很明显你忘记使用`where`子句来限制更新的行数。

### 更新多个列

你可以通过用逗号分隔列名，在一个`update`语句中更新多个列：

```
update  arena
set     arena_name = 'Crypto.com Arena',
 seating_capacity = 19100
where   arena_id = 6;
```

在这里，你已更新了`arena_name`和`seating_capacity`列的值，针对的是`arena_id`为`6`的那一行。

## 删除数据

要从表中删除数据，你可以使用`delete`语句。你可以一次删除一行、多行或使用一个`delete`语句删除所有行。你使用`where`子句来指定要删除的行。在这里，你删除了`arena_id`为`2`的那一行：

```
delete from arena
where arena_id = 2;
```

在你执行完这个`delete`语句后，可以像这样从表中选择剩余的行：

```
select * from arena;
```

结果是：

```
arena_id  arena_name             location        seating_capacity
--------  ---------------------  --------------  ----------------
    1     Madison Square Garden  New York             15000
    3     Philippine Arena       Bocaue               15000
    4     Sportpaleis            Antwerp              15000
    5     Bell Centre            Montreal             15000
    6     Crypto.com Arena       Los Angeles          15000
```

你可以看到，包含`arena_id`为`2`的行已经被删除。

在第七章中，你学习了如何使用`like`进行简单的模式匹配。你可以在这里使用它删除所有名称中包含`Arena`的场馆：

```
delete from arena
where arena_name like '%Arena%';
```

从表中选择剩余的行：

```
select * from arena;
```

结果是：

```
arena_id  arena_name             location        seating_capacity
--------  ---------------------  --------------  ----------------
    1     Madison Square Garden  New York             15000
    4     Sportpaleis            Antwerp              15000
    5     Bell Centre            Montreal             15000
```

包含`Philippine Arena`和`Crypto.com Arena`的两行已不再存在于表中。

如果你编写了一个`delete`语句，并且`where`子句没有匹配任何行，那么就不会删除任何行：

```
delete from arena
where arena_id = 459237;
```

这个语句不会删除任何行，因为没有`arena_id`为`459237`的行。MySQL 不会产生错误消息，但会告诉你`0 row(s) affected`。

要删除表中的所有行，你可以使用不带`where`子句的`delete`语句：

```
delete from arena;
```

这个语句会删除表中的所有行。

## 截断和删除表格

*截断*表会删除所有行，但保留表本身。它的效果与使用不带`where`子句的`delete`相同，但通常更快。

你可以使用`truncate table`命令来截断表，如下所示：

```
truncate table arena;
```

一旦语句执行完毕，表格仍然存在，但其中将没有任何行。

如果你想删除表及其所有数据，你可以使用`drop table`命令：

```
drop table arena;
```

如果你现在尝试从`arena`表中选择数据，MySQL 会显示一条消息，说明该表不存在。

## 摘要

在这一章中，你学习了如何插入、更新和删除表中的数据。你了解了如何插入空值，并快速创建或删除整个表。在下一章，你将学习使用类似表的结构——*视图*——的好处。
