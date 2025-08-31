# 第七章：比较值

![](img/chapterart.png)

本章讨论了在 MySQL 中比较值。你将练习检查值是否相等，某个值是否大于或小于另一个值，值是否在特定范围内，或者是否匹配模式。你还将学习如何检查查询中至少满足一个条件。

在多种场景下，比较值非常有用。例如，你可能想要检查员工是否工作了 40 小时或更多，航班状态是否未取消，或者度假目的地的平均温度是否在 70 到 95 华氏度之间。

## 比较运算符

你可以使用 MySQL 的比较运算符（见表 7-1）来比较查询中的值。

表 7-1：MySQL 比较运算符

| **符号或关键字** | **描述** |
| --- | --- |
| `=` | 相等 |
| `!=, <>` | 不等于 |
| `>` | 大于 |
| `>=` | 大于或等于 |
| `<` | 小于 |
| `<=` | 小于或等于 |
| `is null` | 空值 |
| `is not null` | 非空值 |
| `in` | 匹配列表中的值 |
| `not in` | 不匹配列表中的值 |
| `between` | 在范围内 |
| `not between` | 不在范围内 |
| `like` | 匹配模式 |
| `not like` | 不匹配模式 |

这些运算符让你能够将数据库中的值与其他值进行比较。如果某些数据符合你使用这些比较运算符定义的标准，你可以选择将其选出来。我们将深入讨论它们，并以不同的数据库为例。

### 相等

等号运算符，在第五章中介绍过，可以检查值是否相等以实现特定的结果。例如，这里你使用`=`与第六章中的`wine`数据库表：

```
select  *
from    country
where   **country_id = 3**;
```

该查询从`country`表中选择所有`country_id`等于`3`的国家。

在以下查询中，你使用`=`与字符串而不是数字进行比较：

```
select  *
from    wine_type
where   **wine_type_name = 'Merlot'**;
```

该查询从`wine_type`表中选择所有名称为 Merlot 的葡萄酒，即`wine_type_name`等于`Merlot`。

以下查询类似于你在第五章中学习如何连接两个表时看到的内容。这里你使用`=`来比较来自两个表的具有共同列名的值：

```
select  c.country_name
from    country c
join    region r
  on    **c.country_id = r.country_id**;
```

该查询连接了`region`和`country`表中`country_id`列的所有相等值。

在这些例子中，`=`语法检查运算符左侧的值是否与右侧的值相同。你还可以将`=`与返回一行的子查询一起使用：

```
select *
from   region
where  **country_id =**
**(**
 **select country_id**
 **from   country**
 **where  country_name = 'USA'**
**);**
```

通过这种方式使用`=`，你在外部查询中检查`region`表的`country_id`列是否与整个子查询的结果匹配。

### 不相等

不等于使用 `<>` 或 `!=` 符号表示，其中 `<` 符号表示 *小于*，`>` 符号表示 *大于*（所以 `<>` 意味着小于或大于），而 `!` 符号表示 *不*（所以 `!=` 意味着不等于）。`!=` 和 `<>` 操作符执行相同的操作，因此使用哪种语法都可以。

不等于操作符对于排除某些数据非常有用。例如，也许你是一个班卓琴演奏者，正在寻找志同道合的音乐人组建乐队。因为你弹奏班卓琴，你可以从你想查看的乐器列表中排除它：

```
select  *
from    musical_instrument
where   **instrument != 'banjo'**;
```

在这里，你在 `musical_instrument` 表上使用了不等于操作符，排除了班卓琴在返回的乐器列表中。

假设你正在计划一场婚礼，并且在 2024 年 2 月 11 日有一个先前的安排，所以你需要排除这个日期：

```
select  *
from    possible_wedding_date
where   **wedding_date <> '2024-02-11'**;
```

现在你已经从 `possible_wedding_date` 表中排除了 2024 年 2 月 11 日作为潜在婚礼日期。

### 大于

大于操作符检查左侧的值是否大于右侧的值。它使用 `>` 符号表示。假设你正在寻找那些 `salary` 大于 100,000 美元且 `start_date` 在 2024 年 1 月 20 日之后的工作，你可以使用以下查询从 `job` 表中选择符合这些条件的工作：

```
select  *
from    job
where   **salary > 100000**
and     **start_date > '2024-01-20'**;
```

在这个查询中，只有满足两个条件的工作才会被返回。

### 大于或等于

大于或等于使用 `>=` 符号表示。例如，你可以编辑之前的查询，选择所有 `salary` 为 100,000 美元或更高且 `start_date` 为 2024 年 1 月 20 日或之后的工作：

```
select  *
from    job
where   **salary >= 100000**
and     **start_date >= '2024-01-20'**;
```

`>` 和 `>=` 之间的区别在于，`>=` 会将列出的值包含在其结果中。在前面的示例中，`>=` 会返回 `salary` 为 *恰好* 100,000 美元的工作，但 `>` 不会返回此类工作。

### 小于

小于使用 `<` 符号表示。例如，要查看所有在晚上 10 点之前开始的比赛，你可以执行以下查询：

```
select *
from   team_schedule
where  **game_time < '22:00'**;
```

在 MySQL 中，时间是以军用格式表示的，使用 24 小时制。

### 小于或等于

*小于或等于* 使用 `<=` 符号表示。你可以扩展之前的查询，选择所有 `game_time` 为晚上 10 点或更早的行：

```
select *
from   team_schedule
where  **game_time <= '22:00'**;
```

如果 `game_time` 恰好为 22:00（晚上 10 点），当你使用 `<=` 时将返回该行，但使用 `<` 时则不会返回。

### is null

如第二章和第三章所讨论的，`null` 是一个特殊值，表示数据不可用或不适用。`is null` 语法允许你指定只返回 `null` 值的记录。例如，假设你想查询 `employee` 表，查看那些没有退休或没有设置退休日期的员工：

```
select  *
from    employee
where   **retirement_date** **is null**;
```

现在只返回那些 `retirement_date` 为 `null` 的行：

```
emp_name   retirement_date
--------   ---------------
Nancy      null
Chuck      null
Mitch      null
```

只有使用 `is null` 比较操作符才能检查 `null` 值。例如，使用 `= null` 是无效的：

```
select *
from   employee
where  retirement_date = null;
```

即使表中有 null 值，这个语法也不会返回任何行。在这种情况下，MySQL 不会抛出错误，因此你可能没有意识到返回的是错误的数据。

### is not null

你可以使用 `is not null` 来检查值是否*不是* null。尝试反转之前示例的逻辑，检查已经退休或设定退休日期的员工：

```
select *
from   employee
where  **retirement_date** **is not null**;
```

现在，查询返回 `retirement_date` 不为 `null` 的行：

```
emp_name   retirement_date
--------   ---------------
Alfred     2034-01-08
Latasha    2029-11-17
```

与 `is null` 一样，你必须使用 `is not null` 语法进行此类查询。使用其他语法，如 `!= null` 或 `<> null`，将不会产生正确的结果：

```
select *
from   employee
where  retirement_date != null;
```

正如你之前看到的使用 `= null`，当你尝试使用 `!= null` 语法时，MySQL 不会返回任何行，也不会给出错误提示。

### in

你可以使用 `in` 关键字指定一个多个值的列表，以便查询返回这些值。例如，让我们重新查看 `wine` 数据库，返回 `wine_type` 表中特定的酒：

```
select  *
from    wine_type
where   **wine_type_name in ('Chardonnay', 'Riesling')**;
```

这将返回 `wine_type_name` 为 Chardonnay 或 Riesling 的行。

你还可以使用 `in` 和子查询，从另一个表中选择一组酒类类型：

```
select  *
from    wine_type
where   **wine_type_name in**
 **(**
 **select  wine_type_name**
 **from    cheap_wine**
 **)**;
```

你可以选择不提供硬编码的酒类类型列表，而是从 `cheap_wine` 表中选择所有酒类类型。

### not in

要反转前一个示例的逻辑并排除某些酒类类型，你可以使用 `not in`：

```
select  *
from    wine_type
where   **wine_type_name not in ('Chardonnay', 'Riesling')**;
```

这将返回所有 `wine_type_name` 不是 Chardonnay 或 Riesling 的行。

要选择不在 `cheap_wine` 表中的酒，你可以使用 `not in` 与子查询，如下所示：

```
select  *
from    wine_type
where   **wine_type_name not in**
 **(**
 **select  wine_type_name**
 **from    cheap_wine**
 **)**;
```

这个查询排除了 `cheap_wine` 表中的酒类类型。

### between

你可以使用 `between` 运算符检查某个值是否在指定范围内。例如，要列出 `customer` 表中的千禧一代，可以查找出生在 1981 年到 1996 年之间的人：

```
select  *
from    customer
where   **birthyear between 1981 and 1996**;
```

`between` 关键字是*包含*的。这意味着它会检查范围内的每个 `birthyear`，*包括*1981 年和 1996 年。

### not between

你可以使用 `not` `between` 运算符检查某个值是否不在范围内。使用之前示例中的相同表，找到不是千禧一代的客户：

```
select  *
from    customer
where   **birthyear not between 1981 and 1996**;
```

`not between` 运算符返回的客户列表与 `between` 返回的相反，并且是*不包含*的。1981 年或 1996 年出生的客户将被此查询*排除*，因为他们属于 `between 1981 and 1996` 组。

### like

`like` 运算符允许你检查一个字符串是否匹配某种模式。例如，你可以使用 `like` 来查找来自 No Starch Press 的书籍，检查书籍的 ISBN 是否包含 No Starch 出版商代码 59327。

要指定匹配的模式，你可以使用两个通配符字符之一与 `like` 运算符：百分号（`%`）或下划线（`_`）。

#### 百分号字符

百分号通配符字符可以匹配任意数量的字符。例如，要返回姓氏以字母 *M* 开头的亿万富翁列表，你可以使用 `%` 通配符字符与 `like` 一起使用：

```
select  *
from    billionaire
where   **last_name** **like 'M%'**;
```

你的查询将找到姓氏以*M*开头，后面跟着零个或多个其他字符的亿万富翁。这意味着`like 'M%'`只会匹配字母*M*后没有字符，或者*M*后跟着几个字符（比如`Musk`），或者*M*后跟着很多字符（比如`Melnichenko`）。你的查询结果可能如下所示：

```
first_name   last_name
----------   ---------
Elon         Musk
Jacqueline   Mars
John         Mars
Andrey       Melnichenko
```

你可以使用两个`%`字符来查找位于字符串中任何位置的字符，无论是在开头、中间还是结尾。例如，以下查询查找姓氏中包含字母*e*的亿万富翁：

```
select  *
from    billionaire
where   **last_name like '%e%'**;
```

结果可能如下所示：

```
first_name   last_name
----------   ---------
Jeff         Bezos
Bill         Gates
Mark         Zuckerberg
Andrey       Melnichenko
```

虽然语法`last_name like '%e%'`很方便，但它可能导致查询的运行速度比正常情况更慢。这是因为当你在搜索模式的开头使用`%`通配符时，MySQL 无法利用`last_name`列上的任何索引。（记住，索引帮助 MySQL 优化查询；如果需要复习，请参阅第二章中的“索引”部分。）

#### _ 字符

下划线通配符字符匹配任何单个字符。例如，假设你需要找到一个联系人，但你不记得她的名字是 Jan 还是 Jen。你可以写一个查询，选择以*J*开头，后面跟着通配符字符，再后面跟着*n*的名字。

这里你使用下划线通配符来返回以*at*结尾的三字母词汇列表：

```
select  *
from    three_letter_term
where   **term** **like '_at';**
```

结果可能如下所示：

```
term
----
cat
hat
bat
```

### not like

`not like`运算符可以用于查找不匹配某个模式的字符串。它也使用`%`和`_`通配符字符。例如，要反转`like`示例的逻辑，输入以下查询：

```
select  *
from    three_letter_term
where   **term** **not like '_at'**;
```

结果是`three_letter_term`表中不以*at*结尾的单词：

```
term
----
dog
egg
ape
```

类似地，你可以使用以下查询找到那些姓氏不以字母*M*开头的亿万富翁：

```
select  *
from    billionaire
where   **last_name** **not like 'M%'**;
```

结果可能如下所示：

```
first_name   last_name
----------   ---------
Jeff         Bezos
Bill         Gates
Mark         Zuckerberg
```

### exists

`exists`运算符用于检查子查询是否返回至少一行数据。在这里，你回到`customer`表中的`not between`示例，并使用`exists`来检查该表是否至少包含一个千禧一代：

```
select 'There is at least one millennial in this table'
where **exists**
**(**
 **select  ***
 **from    customer**
 **where   birthyear between 1981 and 1996**
**)**;
```

`customer`表中有千禧一代，所以你的结果是：

```
There is at least one millennial in this table
```

如果 1981 年到 1996 年之间没有出生的客户，你的查询将不会返回任何行，并且`There is at least one millennial in this table`的文本也不会显示。

你可能会看到同一个查询使用`select 1`代替子查询中的`select *`：

```
select 'There is at least one millennial in this table'
where exists
(
    select  1
    from    customer
    where   birthyear between 1981 and 1996
);
```

在这个查询中，选择`*`还是`1`并不重要，因为你只关心至少有一个客户符合你的描述。你真正关心的是内部查询返回了*某些东西*。

## 检查布尔值

在第四章中，你学习了布尔值只有两种可能的值：`true`或`false`。你可以使用特殊语法`is true`或`is false`，只返回符合某个值的结果。在这个例子中，你通过在`employed_flag`列中使用`is true`语法，返回了`bachelor`表中所有已就业的学士学位持有者：

```
select  *
from    bachelor
where   **employed_flag is true**;
```

这个查询使 MySQL 只返回已就业的学士学位持有者的行。

要检查`employed_flag`值为`false`的学士学位持有者，可以使用`is false`：

```
select  *
from    bachelor
where   **employed_flag is false**;
```

现在 MySQL 只返回失业的学士学位持有者的行。

你也可以用其他方式检查布尔列的值。这些行是检查`true`值的等效方式：

```
employed_flag is true
employed_flag
employed_flag = true
employed_flag != false
employed_flag = 1
employed_flag != 0
```

以下几行是检查`false`值的等效方式：

```
employed_flag is false
not employed_flag
employed_flag = false
employed_flag != true
employed_flag = 0
employed_flag != 1
```

如你所见，`1`的值等同于`true`，而`0`的值等同于`false`。

## 或条件

你可以使用 MySQL 的`or`关键字来检查是否满足两个条件中的至少一个。

考虑一下名为`applicant`的表格，它包含了关于求职者的信息。

```
name          associates_degree_flag  bachelors_degree_flag  years_experience
------------  ----------------------  ---------------------  ----------------
Joe Smith               0                       1                   7
Linda Jones             1                       0                   2
Bill Wang               0                       1                   1
Sally Gooden            1                       0                   0
Katy Daly               0                       0                   0
```

`associates_degree_flag`和`bachelors_degree_flag`列是布尔值，其中`0`表示`false`，`1`表示`true`。

在以下查询中，你从`applicant`表中选择，得到一个符合要求的求职者名单，该工作要求有学士学位*或*两年或以上的工作经验：

```
select  *
from    applicant
where   **bachelors_degree_flag is true**
**or      years_experience >= 2;**
```

结果如下：

```
name          associates_degree_flag  bachelors_degree_flag  years_experience
------------  ----------------------  ---------------------  ----------------
Joe Smith               0                       1                   7
Linda Jones             1                       0                   2
Bill Wang               0                       1                   1
```

假设你需要编写一个查询，包含`and`（两个条件都必须满足）和`or`（其中一个条件满足即可）关键字。在这种情况下，你可以使用括号将条件分组，以便 MySQL 返回正确的结果。

让我们看看使用括号如何带来好处。在这里，你为需要求职者拥有两年或以上的工作经验*并且*有副学士学位*或*学士学位的新职位创建了另一个查询，查询的是`applicant`表：

```
select  *
from    applicant
**where   years_experience >= 2**
**and     associates_degree_flag is true**
**or      bachelors_degree_flag is true;**
```

这个查询的结果并非你预期的：

```
name          associates_degree_flag  bachelors_degree_flag  years_experience
------------  ----------------------  ---------------------  ----------------
Joe Smith               0                      1                   7
Linda Jones             1                      0                   2
Bill Wang               0                      1                   1
```

Bill 没有两年或以上的工作经验，那为什么他出现在你的结果集中？

查询同时使用了`and`和`or`。`and`的*运算符优先级*高于`or`，这意味着`and`会在`or`之前进行计算。这导致你的查询找到了满足以下两个条件中至少一个的求职者：

+   两年或以上的工作经验*以及*副学士学位

*或*

+   学士学位

这不是你编写查询时的意图。你可以通过使用括号将条件分组来修正问题：

```
select  *
from    applicant
where   years_experience >= 2
and     (
        associates_degree_flag is true
or      bachelors_degree_flag is true
        );
```

现在查询找到了符合这些条件的求职者：

+   两年或以上的工作经验

*和*

+   副学士学位*或*学士学位

现在你的结果应该与你的预期一致：

```
name          associates_degree_flag  bachelors_degree_flag  years_experience
------------  ----------------------  ---------------------  ----------------
Joe Smith               0                      1                   7
Linda Jones             1                      0                   2
```

## 总结

在本章中，你学习了通过比较运算符在 MySQL 中比较值的多种方式，比如检查值是否相等、是否为 null、是否在某个范围内，或是否匹配某个模式。你还学习了如何在查询中检查是否满足至少一个条件。

在下一章中，你将了解如何使用 MySQL 的内置函数，包括处理数学、日期和字符串的函数。你还将学习聚合函数以及如何在一组值中使用它们。
