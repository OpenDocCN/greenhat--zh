# 第八章：调用内建 MySQL 函数

![](img/chapterart.png)

MySQL 有数百个预写的函数，执行各种任务。在本章中，您将回顾一些常见的函数，并学习如何从查询中调用它们。您将使用聚合函数，这些函数基于数据库中许多行数据返回一个单一的值汇总，以及帮助执行数学计算、处理字符串、处理日期等的函数。

在第十一章中，您将学习如何创建自己的函数，但目前您将专注于调用 MySQL 最有用的内建函数。关于所有内建函数的最新列表，最好的来源是 MySQL 参考手册。在线搜索“MySQL 内建函数与操作符参考”，并将网页添加到您的浏览器书签中。

## 什么是函数？

*函数* 是一组已保存的 SQL 语句，用于执行某些任务并返回一个值。例如，`pi()` 函数确定圆周率的值并返回它。以下是一个调用 `pi()` 函数的简单查询：

```
select pi();
```

到目前为止，您看到的大多数查询都包含 `from` 子句，指定要使用的表。在这个查询中，您并没有从任何表中选择数据，因此您可以在没有 `from` 的情况下调用该函数。它返回以下结果：

```
pi()
----------
3.141593
```

对于这种常见任务，使用 MySQL 的内建函数比每次都记住该值更为合理。

## 向函数传递参数

正如您刚刚看到的，函数返回一个值。有些函数还允许您传递值给它们。当您调用函数时，可以指定它应该使用的值。传递给函数的值称为 *参数*。

为了了解参数是如何工作的，您将调用 `upper()` 函数，该函数允许您接受一个参数：一个字符串值。该函数确定该字符串的大写等效形式并返回它。以下查询调用 `upper()` 并指定一个参数为文本 `rofl`：

```
select upper('rofl');
```

结果如下：

```
upper('rofl')
--------------
ROFL
```

该函数将每个字母转换为大写并返回 `ROFL`。

在某些函数中，您可以指定多个参数。例如，`datediff()` 允许您指定两个日期作为参数，然后返回它们之间的天数差异。在这里，您调用 `datediff()` 来查找 2024 年圣诞节与感恩节之间有多少天：

```
select datediff('2024-12-25', '2024-11-28');
```

结果是：

```
datediff('2024-12-25', '2024-11-28')
27
```

当您调用 `datediff()` 函数时，您指定了两个参数，圣诞节的日期和感恩节的日期，并用逗号将它们分开。该函数计算出天数差并返回该值（`27`）。

函数接受不同数量和类型的值。例如，`upper()` 接受一个字符串值，而 `datediff()` 接受两个 `date` 类型的值。正如您将在本章中看到的，其他函数接受整数、布尔值或其他数据类型的值。

## 可选参数

一些函数接受一个可选参数，在调用函数时，你可以提供另一个值来获取更具体的结果。例如，`round()`函数，它用于四舍五入小数，接受一个必须提供的参数和一个可选的第二个参数。如果你只传入一个要四舍五入的数字作为唯一参数，它会将数字四舍五入到零位。尝试使用`2.71828`作为唯一参数调用`round()`函数：

```
select round(2.71828);
```

`round()`函数将返回四舍五入后的数字，去掉小数点后的位数：

```
round(2.71828)
--------------
       3
```

如果你为`round()`提供了可选参数，你可以指定四舍五入小数点后保留的位数。尝试使用`2.71828`作为第一个参数，`2`作为第二个参数，并用逗号分隔参数调用`round()`：

```
select round(2.71828, 2);
```

现在结果是：

```
round(2.71828)
--------------
      2.72
```

这次，`round()`返回一个四舍五入到小数点后两位的数字。

## 在函数内调用函数

你可以通过包装或嵌套函数来将一个函数的结果用作另一个函数的参数。

假设你想获取 pi 的四舍五入值。你可以将对`pi()`函数的调用包装在对`round()`函数的调用中：

```
select round(pi());
```

结果是：

```
round(pi())
-----------
     3
```

最内层的函数首先执行，结果传递给外层函数。对`pi()`函数的调用返回`3.141593`，该值作为参数传递给`round()`函数，`round()`返回`3`。

你可以修改查询，通过指定`round()`函数可选的第二个参数，将 pi 四舍五入到两位小数，如下所示：

```
select round(pi(), 2);
```

结果是：

```
round(pi(), 2)
-------------
     3.14
```

对`pi()`函数的调用返回`3.141593`，这个值作为函数的第一个参数传递给`round()`。这个语句的计算为`round(3.141593,2)`，返回`3.14`。

## 从查询的不同部分调用函数

你可以在查询的`select`列表中调用函数，也可以在`where`子句中调用。例如，看看`movie`表，它包含以下关于电影的数据：

```
movie_name         star_rating  release_date
-----------------  -----------  ------------
Exciting Thriller  4.72         2024-09-27
Bad Comedy         1.2          2025-01-02
OK Horror          3.1789       2024-10-01
```

`star_rating`列保存了观众给电影打的平均星级，评分范围是 1 到 5。你被要求写一个查询，显示评分超过 3 星且发布年份为 2024 年的电影。你还需要将电影名称转换为大写并四舍五入星级评分：

```
select upper(movie_name),
       round(star_rating)
from   movie
where  star_rating > 3
and    year(release_date) = 2024;
```

首先，在查询的`select`列表中，你使用`upper()`和`round()`函数。你将电影名称值包裹在`upper()`函数中，并将星级评分值包裹在`round()`函数中。然后你指定从`movie`表中提取数据。

在`where`子句中，你调用`year()`函数并指定一个参数：`movie`表中的`release_date`。`year()`函数返回电影的发布年份，你将其与`2024`进行比较（`=`），从而只显示发布年份为 2024 年的电影。

结果是：

```
upper(movie_name)  round(star_rating)
-----------------  ------------------
EXCITING THRILLER           5
OK HORROR                   3
```

## 聚合函数

*聚合*函数是一种基于数据库中多个值返回单一值的函数类型。常见的聚合函数包括`count()`、`max()`、`min()`、`sum()`和`avg()`。在本节中，你将看到如何使用以下`continent`表来调用这些函数：

```
continent_id  continent_name  population
------------  --------------  ----------
1             Asia            4641054775
2             Africa          1340598147
3             Europe          747636026
4             North America   592072212
5             South America   430759766
6             Australia       43111704
7             Antarctica      0
```

### count()

`count()`函数返回查询结果中的行数，可以帮助回答有关数据的问题，例如“你有多少顾客？”或“你今年收到了多少投诉？”

你可以使用`count()`函数来确定`continent`表中有多少行，像这样：

```
select  count(*)
from    continent;
```

当你调用`count()`函数时，你在括号中使用星号（或通配符）来计算所有行。星号选择表中的所有行，包括每行的所有列值。

结果是：

```
count(*)
--------
   7
```

使用`where`子句选择所有人口超过 10 亿的大陆：

```
select  count(*)
from    continent
where   population > 1000000000;
```

结果是：

```
count(*)
--------
   2
```

查询返回`2`，因为只有亚洲和非洲这两个大陆的人口超过了 10 亿。

### max()

`max()`函数返回一组值中的最大值，可以帮助回答诸如“最高的年度通货膨胀率是多少？”或“哪个销售员本月卖出了最多的车？”等问题。

这里你使用`max()`函数来查找表中任何大陆的最大人口：

```
select max(population)
from   continent;
```

结果是：

```
max(population)
---------------
  4641054775
```

当你调用`max()`函数时，它返回人口最多的大陆的居民数。表中人口最多的大陆是亚洲，人口为 4,641,054,775。

像`max()`这样的聚合函数在子查询中尤其有用。暂时离开`continent`表，将注意力转向`train`表：

```
train            mile
---------------  ----
The Chief        8000
Flying Scotsman  6500
Golden Arrow     2133
```

这里你将使用`max()`来帮助确定`train`表中行驶里程最多的火车：

```
select   *
from     train
where    mile =
(
  select max(mile)
  from   train
);
```

在内部查询中，你选择表中任何火车所行驶的最大里程数。在外部查询中，你显示所有行驶了该里程数的火车的列。

结果是：

```
train_name  mile
----------  ----
The Chief   8000
```

### min()

`min()`函数返回一组值中的最小值，可以帮助回答诸如“市区汽油的最低价格是多少？”或“哪种金属的熔点最低？”等问题。

让我们回到`continent`表。使用`min()`函数来查找人口最少的大陆：

```
select min(population)
from   continent;
```

当你调用`min()`函数时，它返回表中的最小人口值：

```
min(population)
---------------
       0
```

表中人口最少的行是南极洲，人口为`0`。

### sum()

`sum()`函数计算一组数字的总和，并帮助回答诸如“中国有多少辆自行车？”或“你今年的总销售额是多少？”等问题。

使用`sum()`函数来计算所有大陆的总人口，像这样：

```
select sum(population)
from   continent;
```

当你调用`sum()`函数时，它返回所有大陆人口的总和。

结果是：

```
max(population)
---------------
   7795232630
```

### avg()

`avg()`函数根据一组数字返回平均值，能够帮助回答“威斯康星州的平均降雪量是多少？”或者“医生的平均薪水是多少？”等问题。

使用`avg()`函数来找出大洲的平均人口：

```
select avg(population)
from   continent;
```

当你调用`avg()`函数时，它会返回表中大洲的平均人口值：

```
avg(population)
---------------
1113604661.4286
```

MySQL 通过将每个大洲的人口总和（7,795,232,630）除以大洲数量（7）得出了 1,113,604,661.4286。

现在，使用`avg()`函数在子查询中显示人口少于平均大洲人口的所有大洲：

```
select    *
from      continent
where     population <
(
  select  avg(population)
  from    continent
);
```

内部查询选择所有大洲的平均人口数量：1,113,604,661.4286 人。外部查询选择人口少于该值的大洲的所有列。

结果是：

```
continent_id  continent_name  population
------------  --------------  ----------
     3        Europe           747636026
     4        North America    592072212
     5        South America    430759766
     6        Australia         43111704
     7        Antarctica               0
```

### group by

`group by`子句告诉 MySQL 你希望如何对结果进行分组，并且只能在包含聚合函数的查询中使用。要查看`group by`如何工作，可以查看`sale`表，它存储了公司的销售记录：

```
sale_id  customer_name  salesperson  amount
-------  -------------  -----------  ------
1        Bill McKenna   Sally        12.34
2        Carlos Souza   Sally        28.28
3        Bill McKenna   Tom           9.72
4        Bill McKenna   Sally        17.54
5        Jane Bird      Tom          34.44
```

你可以使用`sum()`聚合函数来添加销售金额，但你是想计算所有销售的总额，按客户汇总金额，按销售员汇总金额，还是计算每个销售员向每个客户销售的总额？

要显示按客户汇总的金额，你需要在`customer_name`列上使用`group by`，如示例 8-1 所示。

```
select sum(amount)
from   sale
group by customer_name;
```

示例 8-1：按客户汇总金额的查询

结果如下：

```
sum(amount)
-----------
      39.60
      28.28
      34.44
```

客户 Bill McKenna 消费的总金额为$39.60；Carlos Souza 为$28.28；Jane Bird 为$34.44。结果按客户的名字字母顺序排序。

另外，你可能想查看每个销售员的汇总金额。示例 8-2 展示了如何在`salesperson_name`列上使用`group by`。

```
select sum(amount)
from   sale
group by salesperson_name;
```

示例 8-2：按销售员汇总金额的查询

你的结果是：

```
sum(amount)
-----------
      58.16
      44.16
```

Sally 的总销售额为$58.16，Tom 的为$44.16。

因为`sum()`是一个聚合函数，它可以对任意数量的行进行操作，并返回一个值。`group by`语句告诉 MySQL 你希望`sum()`作用于哪些行，因此语法`group by salesperson_name`会对每个销售员的金额进行求和。

假设你只想查看一行，其中包含表中所有`amount`的总和。在这种情况下，你不需要使用`group by`，因为你并不是按任何分组来求和。你的查询应该如下所示：

```
select  sum(amount)
from    sale;
```

结果应为：

```
sum(amount)
-----------
     102.32
```

`group by`子句适用于所有聚合函数。例如，你可以将`group by`与`count()`一起使用，返回每个销售员的销售数量，如示例 8-3 所示。

```
select count(*)
from   sale
group by salesperson_name;
```

示例 8-3：按销售员统计行数的查询

结果是：

```
count(*)
--------
   3
   2
```

查询统计了`sales`表中 Sally 的三行和 Tom 的两行。

或者，你可以使用`avg()`来获取平均销售额，并根据`salesperson_name`进行分组，返回每个销售人员的平均销售额，如清单 8-4 所示。

```
select   avg(amount)
from     sale
group by salesperson_name;
```

清单 8-4：获取每个销售人员平均销售额的查询

结果是：

```
avg(amount)
-----------
  19.386667
  22.080000
```

结果显示，Sally 每笔销售的平均金额为$19.386667，而 Tom 每笔销售的平均金额为$22.08。

然而，查看这些结果时，尚不清楚哪个销售人员的平均值是$19.386667，哪个销售人员的是$22.08。为了澄清这一点，让我们修改查询以显示更多的信息。在清单 8-5 中，你也选择了销售人员的名字。

```
select   salesperson_name,
         avg(amount)
from     sale
group by salesperson_name;
```

清单 8-5：显示销售人员姓名及其平均销售额的查询

修改后的查询结果是：

```
salesperson_name  avg(amount)
----------------  -----------
Sally               19.386667
Tom                 22.080000
```

你的平均值显示了相同的数值，但现在销售人员的名字也显示在其旁边。添加这些额外的信息使得结果更加易于理解。

在你编写了多个使用聚合函数和`group by`的查询后，你可能会注意到，你通常会对查询中选择的相同列进行分组。例如，在清单 8-5 中，你选择了`salesperson_name`列，并且也根据`salesperson_name`列进行了分组。

为了帮助你确定应分组的列，查看*选择列表*，即查询中`select`和`from`之间的部分。选择列表包含你希望从数据库表中选择的项；你几乎总是希望按这个相同的列表进行分组。选择列表中唯一不应该出现在`group by`语句中的部分是调用的聚合函数。

例如，看看这个 `theme_park` 表，它包含了六个不同主题公园的数据，包括它们的国家、州以及所在城市：

```
country  state           city                park
-------  ------------    ------------------  -----------------
USA      Florida         Orlando             Disney World
USA      Florida         Orlando             Universal Studios
USA      Florida         Orlando             SeaWorld
USA      Florida         Tampa               Busch Gardens
Brazil   Santa Catarina  Balneario Camboriu  Unipraias Park
Brazil   Santa Catarina  Florianopolis       Show Water Park
```

假设你想选择国家、州以及这些国家和州的公园数量。你可能会开始像这样编写 SQL 语句：

```
select country,
       state,
       count(*)
from   theme_park;
```

然而，这个查询是不完整的，运行它会返回错误信息或错误的结果，具体取决于你的配置设置。

你应该对所有选中的*非聚合函数*列进行分组。在这个查询中，你选择的列`country`和`state`不是聚合函数，所以你将使用`group by`来对它们进行分组：

```
select   country,
         state,
         count(*)
from     theme_park
**group by country,**
 **state;**
```

结果如下：

```
country state           count(*)
------  --------------  --------
USA     Florida            4
Brazil  Santa Catarina     2
```

如你所见，查询现在返回了正确的结果。

## 字符串函数

MySQL 提供了多个函数来帮助你处理字符字符串，执行诸如比较、格式化和组合字符串等任务。让我们来看看一些最有用的字符串函数。

### concat()

`concat()` 函数*连接*两个或更多字符串。例如，假设你有以下的 `phone_book` 表：

```
first_name  last_name
----------  ----------
Jennifer    Perez
Richard     Johnson
John        Moore
```

你可以编写一个查询，将名字和姓氏一起显示，并用空格字符分隔：

```
select  concat(first_name, ' ', last_name)
from    phone_book;
```

结果应如下所示：

```
concat(first_name, ' ', last_name)
----------------------------------
Jennifer Perez
Richard Johnson
John Moore
```

名字作为一个字符串显示，以空格分隔。

### format()

`format()` 函数通过添加逗号和显示所请求的小数位数来格式化数字。例如，我们重新访问 `continent` 表并选择亚洲人口，如下所示：

```
select  population
from    continent
where   continent_name = 'Asia';
```

结果是：

```
population
----------
4641054775
```

很难判断亚洲人口是约 46 亿人还是 4.64 亿人。为了使结果更易读，你可以使用 `format()` 函数为 `population` 列添加逗号格式，如下所示：

```
select format(population, 0)
from   continent;
```

`format()` 函数接受两个参数：一个数字来格式化和显示小数点后位数的数量。你使用了两个参数调用 `format()`：`population` 列和数字 `0`。

现在 `population` 列已经用逗号格式化，结果中清楚地显示亚洲大约有 46 亿人口：

```
population
-------------
4,641,054,775
```

现在调用 `format()` 函数将数字 1234567.89 格式化为小数点后五位：

```
select format(1234567.89, 5);
```

结果是：

```
format(1234567.89, 5)
---------------------
    1,234,567.89000
```

`format()` 函数接受 `1234567.89` 作为第一个参数中的数字进行格式化，添加逗号，并且添加尾随零，使得结果显示五位小数。

### left()

`left()` 函数从值的左侧返回若干字符。考虑以下 `taxpayer` 表：

```
last_name  soc_sec_no
---------  ------------
Jagger     478-555-7598
McCartney  478-555-1974
Hendrix    478-555-3555
```

要从 `taxpayer` 表中选择姓氏，并且还要选择 `last_name` 列的前三个字符，你可以写如下查询：

```
select  last_name,
        left(last_name, 3)
from    taxpayer;
```

结果是：

```
last_name   left(last_name, 3)
----------  -----------------
Jagger      Jag
McCartney   McC
Hendrix     Hen
```

`left()` 函数在你想忽略右侧字符的情况下非常有用。

### right()

`right()` 函数从值的右侧返回若干字符。继续使用 `taxpayer` 表选择税务员社会安全号码的最后四位数字：

```
select  right(soc_sec_no, 4)
from    taxpayer;
```

结果是：

```
right(soc_sec_no, 4)
--------------------
        7598
        1974
        3555
```

`right()` 函数选择最右边的字符，忽略左边的字符。

### lower()

`lower()` 函数返回字符串的小写版本。选择税务员的姓氏并将其转换为小写：

```
select  lower(last_name)
from    taxpayer;
```

结果是：

```
lower(last_name)
----------------
jagger
mccartney
hendrix
```

### upper()

`upper()` 函数返回字符串的大写版本。选择税务员的姓氏并将其转换为大写：

```
select  upper(last_name)
from    taxpayer;
```

结果是：

```
upper(last_name)
----------------
JAGGER
MCCARTNEY
HENDRIX
```

### substring()

`substring()` 函数返回字符串的一部分，接受三个参数：一个字符串、你想要的子字符串的起始字符位置和结束字符位置。

你可以通过以下查询从字符串 `gumbo` 中提取子字符串 `gum`：

```
select substring('gumbo', 1, 3);
```

结果是：

```
substring('gumbo', 1, 3)
------------------------
          gum
```

在 `gumbo` 中，`g` 是第一个字符，`u` 是第二个字符，`m` 是第三个字符。从第 1 个字符开始，选择到第 3 个字符，会返回这前三个字符。

`substring()` 函数的第二个参数可以接受负数。如果你传递负数，它会从字符串的末尾向回计数来确定子字符串的起始位置。例如：

```
select substring('gumbo', -3, 2);
```

结果是：

```
substring('gumbo', -3, 2)
------------------------
          mb
```

字符串`gumbo`包含五个字符。你要求`substring()`从字符串末尾减去三个字符位置开始子字符串，即位置 3。你的第三个参数是 2，因此子字符串会从第三个字符开始，并取两个字符，得到`mb`子字符串。

`substring()`函数的第三个参数是可选的。你只需提供前两个参数——一个字符串和起始字符位置——即可返回从起始位置到字符串末尾的字符集：

```
select substring('MySQL', 3);
```

结果是：

```
substring('MySQL', 3)
------------------------
          SQL
```

`substring()`函数返回了从字符串`MySQL`的第三个字符开始，直到字符串末尾的所有字符，结果是`SQL`。

MySQL 提供了一种替代语法用于`substring()`，使用`from`和`for`关键字。例如，要选择单词`gumbo`的前三个字符，可以使用以下语法：

```
select substring('gumbo' from 1 for 3);
```

这个子字符串从第一个字符开始，持续三个字符。结果如下：

```
substring('gumbo' from 1 for 3)
------------------------------
             gum
```

这个结果与第一个子字符串示例相同，但你可能会觉得这种语法更容易阅读。

### `trim()`

`trim()`函数会去除字符串中的任意数量的前导或尾随字符。你可以指定要移除的字符，以及是否要移除前导字符、尾随字符或两者。

例如，如果你有字符串`**instructions**`，你可以使用`trim()`来返回去掉星号后的字符串，像这样：

```
select trim(leading  '*' from '**instructions**') as column1,
       trim(trailing '*' from '**instructions**') as column2,
       trim(both     '*' from '**instructions**') as column3,
       trim(         '*' from '**instructions**') as column4;
```

在`column1`中，你去除前导的星号。在`column2`中，你去除尾随的星号。在`column3`中，你去除前导和尾随的星号。当你没有指定`leading`、`trailing`或`both`时，如在`column4`中，MySQL 默认为去除两端的空格。

结果如下：

```
column1         column2         column3       column4
--------------  --------------  ------------  ------------
instructions**  **instructions  instructions  instructions
```

默认情况下，`trim()`会移除空格字符。这意味着，如果字符串两侧有空格字符，你可以直接使用`trim()`，无需指定要移除的字符：

```
select trim('   asteroid   ');
```

结果是字符串`asteroid`，两侧都没有空格：

```
trim('   asteroid   ')
----------------------
asteroid
```

`trim()`函数默认会移除字符串两侧的空格。

### `ltrim()`

`ltrim()`函数用于移除字符串左侧的前导空格：

```
select ltrim('   asteroid   ');
```

结果是字符串`asteroid`，左侧没有空格：

```
ltrim('   asteroid   ')
----------------------
asteroid
```

右侧的空格不会受到影响。

### `rtrim()`

`rtrim()`函数用于移除字符串右侧的尾随空格：

```
select rtrim('   asteroid   ');
```

结果是字符串`asteroid`，右侧没有空格：

```
rtrim('   asteroid   ')
----------------------
   asteroid
```

左侧的空格不会受到影响。

## 日期和时间函数

MySQL 提供了与日期相关的函数，帮助你执行获取当前日期和时间、选择日期的某一部分以及计算两个日期之间相差多少天等任务。

如您在第四章中所见，MySQL 提供了 `date`、`time` 和 `datetime` 数据类型，其中 `date` 包含月、日和年；`time` 包含小时、分钟和秒；`datetime` 则包含这些所有部分，因为它既包括日期又包括时间。这些是 MySQL 用于返回许多函数结果的格式。

### curdate()

`curdate()` 函数以 `date` 格式返回当前日期：

```
select curdate();
```

您的结果应类似于以下内容：

```
curdate()
----------
2024-12-14
```

`current_date()` 和 `current_date` 都是 `curdate()` 的同义词，并会产生相同的结果。

### curtime()

`curtime()` 函数返回当前时间，格式为 `time`：

```
select curtime();
```

您的结果应类似于以下内容：

```
curtime()
---------
09:02:41
```

对我来说，当前时间是上午 9:02 和 41 秒。`current_time()` 和 `current_time` 都是 `curtime()` 的同义词，并会产生相同的结果。

### now()

`now()` 函数以 `datetime` 格式返回当前的日期和时间：

```
select now();
```

您的结果应类似于以下内容：

```
now()
-------------------
2024-12-14 09:02:18
```

`current_timestamp()` 和 `current_timestamp` 都是 `now()` 的同义词，并会产生相同的结果。

### date_add()

`date_add()` 函数将一定量的时间加到 `date` 值上。要对日期值进行加（或减）操作，需要使用 *间隔*，这是一种可以用于执行日期和时间计算的值。使用间隔时，您可以提供一个数字和一个时间单位，例如 `5 day`、`4 hour` 或 `2 week`。请看以下名为 `event` 的表：

```
event_id  eclipse_datetime
--------  -------------------
   374    2024-10-25 11:01:20
```

要从 `event` 表中选择 `eclipse_datetime` 日期并加上 5 天、4 小时和 2 周，您可以使用带有 `interval` 的 `date_add()`，如下所示：

```
select  eclipse_datetime,
        date_add(eclipse_datetime, interval 5 day)  as add_5_days,
        date_add(eclipse_datetime, interval 4 hour) as add_4_hours,
        date_add(eclipse_datetime, interval 2 week) as add_2_weeks
from    event
where   event_id = 374;
```

您的结果应类似于此：

```
eclipse_datetime     add_5_days           add_4_hours          add_2_weeks
-------------------  -------------------  -------------------  -------------------
2024-10-25 11:01:20  2024-10-30 11:01:20  2024-10-25 15:01:20  2024-11-08 11:01:20
```

结果显示，5 天、4 小时和 2 周的时间间隔已加到日全食的日期和时间，并列出了您指定的列。

### date_sub()

`date_sub()` 函数从 `date` 值中减去一个时间间隔。例如，在这里，您从 `event` 表的 `eclipse_datetime` 列中减去与前面示例相同的时间间隔：

```
select  eclipse_datetime,
        date_sub(eclipse_datetime, interval 5 day)  as sub_5_days,
        date_sub(eclipse_datetime, interval 4 hour) as sub_4_hours,
        date_sub(eclipse_datetime, interval 2 week) as sub_2_weeks
from    event
where   event_id = 374;
```

结果如下：

```
eclipse_datetime     sub_5_days           sub_4_hours          sub_2_weeks
-------------------  -------------------  -------------------  -------------------
2024-10-25 11:01:20  2024-10-20 11:01:20  2024-10-25 07:01:20  2024-10-11 11:01:20
```

结果显示，5 天、4 小时和 2 周的时间间隔已从日全食的日期和时间中减去，并列出了您指定的列。

### extract()

`extract()` 函数提取指定的 `date` 或 `datetime` 值的部分。它使用与 `date_add()` 和 `date_sub()` 相同的时间单位，如 `day`、`hour` 和 `week`。

在此示例中，您选择了 `eclipse_datetime` 列的部分内容：

```
select  eclipse_datetime,
        extract(year from eclipse_datetime)   as year,
        extract(month from eclipse_datetime)  as month,
        extract(day from eclipse_datetime)    as day,
        extract(week from eclipse_datetime)   as week,
        extract(second from eclipse_datetime) as second
from    event
where   event_id = 374;
```

`extract()` 函数从 `event` 表中的 `eclipse_datetime` 值中提取并显示您指定列名所请求的各个部分。结果如下：

```
eclipse_datetime     year  month  day  week  second
-------------------  ----  -----  ---  ----  ------
2024-10-25 11:01:20  2024     10   25    43      20
```

MySQL 还提供了其他函数，您可以用来与 `extract()` 达到相同的目的，包括 `year()`、`month()`、`day()`、`week()`、`hour()`、`minute()` 和 `second()`。该查询与前一个查询产生相同的结果：

```
select  eclipse_datetime,
        year(eclipse_datetime)   as year,
        month(eclipse_datetime)  as month,
        day(eclipse_datetime)    as day,
        week(eclipse_datetime)   as week,
        second(eclipse_datetime) as second
from    event
where   event_id = 374;
```

你也可以使用 `date()` 和 `time()` 函数，只选择 `datetime` 值中的 `date` 或 `time` 部分：

```
select  eclipse_datetime,
        date(eclipse_datetime)   as date,
        time(eclipse_datetime)   as time
from    event
where   event_id = 374;
```

结果是：

```
eclipse_datetime     date        time
-------------------  ----------  --------
2024-10-25 11:01:20  2024-10-25  11:01:20
```

如你所见，`date()` 和 `time()` 函数提供了一种快速方式，从 `datetime` 值中提取出日期或时间。

### datediff()

`datediff()` 函数返回两个日期之间的天数。假设你想检查 2024 年新年和 Cinco de Mayo 之间有多少天：

```
select datediff('2024-05-05', '2024-01-01');
```

结果是 125 天：

```
datediff('2024-05-05', '2024-01-01')
------------------------------------
                 125
```

如果左边的日期参数比右边的日期参数更新，`datediff()` 会返回一个正数。如果右边的日期更晚，`datediff()` 会返回一个负数。如果两个日期相同，返回 `0`。

### date_format()

`date_format()` 函数根据你指定的格式字符串格式化日期。格式字符串由你添加的字符和以百分号开头的*格式符*组成。最常见的格式符列在表 8-1 中。

表 8-1：常见的格式化符号

| **格式符** | **描述** |
| --- | --- |
| `%a` | 缩写的星期名称（`Sun`–`Sat`） |
| `%b` | 缩写的月份名称（`Jan`–`Dec`） |
| `%c` | 数字表示的月份（`1`–`12`） |
| `%D` | 带后缀的日期（`1st`，`2nd`，`3rd`，...） |
| `%d` | 日期（两位数字，适用时带前导零，范围为`01`–`31`） |
| `%e` | 日期（`1`–`31`） |
| `%H` | 小时，适用时带前导零（`00`–`23`） |
| `%h` | 小时（`01`–`12`） |
| `%i` | 分钟（`00`–`59`） |
| `%k` | 小时（`0`–`23`） |
| `%l` | 小时（`1`–`12`） |
| `%M` | 月份名称（`January`–`December`） |
| `%m` | 月份（`00`–`12`） |
| `%p` | `AM` 或 `PM` |
| `%r` | 时间，12 小时制（`hh:mm:ss` 后跟 `AM` 或 `PM`） |
| `%s` | 秒（`00`–`59`） |
| `%T` | 时间，24 小时制（`hh:mm:ss`） |
| `%W` | 星期几的名称（`Sunday`–`Saturday`） |
| `%w` | 星期几（`0` = 星期天，`6` = 星期六） |
| `%Y` | 四位数字年份 |
| `%y` | 两位数字的年份 |

`2024-02-02 01:02:03` 代表 2024 年 2 月 2 日凌晨 1:02:03。试试为该 `datetime` 使用不同的格式：

```
select  date_format('2024-02-02 01:02:03', '%r') as format1,
        date_format('2024-02-02 01:02:03', '%m') as format2,
        date_format('2024-02-02 01:02:03', '%M') as format3,
        date_format('2024-02-02 01:02:03', '%Y') as format4,
        date_format('2024-02-02 01:02:03', '%y') as format5,
        date_format('2024-02-02 01:02:03', '%W, %M %D at %T') as format6;
```

结果是：

```
format1      format2  format3   format4  format5  format6
-----------  -------  --------  -------  -------  -----------------------------------
01:02:03 AM     02    February    2024      24    Friday, February 2nd at 01:02:03
```

你将列别名为 `format6` 显示了如何将格式化符号组合在一起。在该格式字符串中，除了为日期和时间添加四个格式符号外，你还添加了一个逗号和单词 `at`。

### str_to_date()

`str_to_date()` 函数根据你提供的格式将字符串值转换为日期。你使用的格式符与 `date_format()` 中的相同，但这两个函数的作用正好相反：`date_format()` 将日期转换为字符串，而 `str_to_date()` 将字符串转换为日期。

根据你提供的格式，`str_to_date()` 可以将字符串转换为 `date`、`time` 或 `datetime`：

```
select str_to_date('2024-02-02 01:02:03', '%Y-%m-%d')          as date_format,
       str_to_date('2024-02-02 01:02:03', '%Y-%m-%d %H:%i:%s') as datetime_format,
       str_to_date('01:02:03', '%H:%i:%s')                     as time_format;
```

结果是：

```
date_format  datetime_format      time_format
-----------  -------------------  -----------
2024-02-02   2024-02-02 01:02:03    01:02:03
```

最后一列 `time_format` 也可以通过同名函数进行转换。接下来我们将讨论这个。

### time_format()

正如其名称所示，`time_format()`函数用于格式化时间。你可以像`date_format()`那样使用相同的格式说明符来格式化`time_format()`。例如，以下是获取当前时间并以不同方式格式化的示例：

```
select  time_format(curtime(), '%H:%i:%s')                            as format1,
        time_format(curtime(), '%h:%i %p')                            as format2,
        time_format(curtime(), '%l:%i %p')                            as format3,
        time_format(curtime(), '%H hours, %i minutes and %s seconds') as format4,
        time_format(curtime(), '%r')                                  as format5,
        time_format(curtime(), '%T')                                  as format6;
```

按军用时间格式表示，我现在的时间是`21:09:55`，即晚上 9:09 分 55 秒。你的结果应如下所示：

```
format1   format2   format3  format4                              format5      format6
--------  --------  -------  -----------------------------------  -----------  --------
21:09:55  09:09 PM  9:09 PM  21 hours, 09 minutes and 55 seconds  09:09:55 PM  21:09:55
```

你将别名为`format2`的列显示了带有前导`0`的小时，因为你使用了`%H`格式说明符，而`format3`列则没有，因为你使用了`%h`格式说明符。在列 1–3 中，你向格式字符串中添加了冒号字符。在`format4`中，你添加了单词`hours`，逗号，单词`minutes`，单词`and`，以及单词`seconds`。

## 数学运算符和函数

MySQL 提供了许多函数来进行计算。也提供了一些算术运算符，如`+`表示加法，`-`表示减法，`*`表示乘法，`/`和`div`表示除法，`%`和`mod`表示模运算。你将开始查看一些使用这些运算符的查询，然后使用括号来控制运算顺序。之后，你将使用数学函数来执行各种任务，包括求一个数的幂、计算标准差以及对数字进行四舍五入和截断。

### 数学运算符

你将首先使用`payroll`表中的数据进行一些数学计算：

```
employee   salary    deduction   bonus    tax_rate
--------  ---------  ---------  --------  --------
Max Bain   80000.00    5000.00  10000.00      0.24
Lola Joy   60000.00       0.00    800.00      0.18
Zoe Ball  110000.00    2000.00  30000.00      0.35
```

尝试以下的一些算术运算符：

```
select  employee,
        salary - deduction,
        salary + bonus,
        salary * tax_rate,
        salary / 12,
        salary div 12
from    payroll;
```

在这个例子中，你使用数学运算符计算员工的工资减去扣款，再加上奖金，乘以税率，最后通过将年薪除以 12 来计算月薪。

结果如下：

```
employee salary - deduction  salary + bonus   salary * tax_rate  salary / 12  salary div 12
-------- ------------------  --------------  ------------------  -----------  -------------
Max Bain           75000.00        90000.00   9199.999570846558  6666.666667           6666
Lola Joy           60000.00        60800.00  10800.000429153442  5000.000000           5000
Zoe Ball          108000.00       140000.00   38499.99934434891  9166.666667           9166
```

请注意，在右侧的两列中，`salary / 12`和`salary div 12`使用`/`和`div`运算符时，得到的结果不同。这是因为`div`会舍弃任何小数部分，而`/`则不会。

#### 模运算

MySQL 提供了两个模运算符：百分号（`%`）和`mod`运算符。*模运算*接受一个数字，将其除以另一个数字，并返回余数。考虑一个名为`roulette_winning_number`的表：

```
winning_number
--------------
       21
        8
       13
```

你可以使用模运算来判断一个数字是奇数还是偶数，通过将其除以 2 并检查余数，如下所示：

```
select  winning_number,
        winning_number % 2
from    roulette_winning_number;
```

任何余数为 1 的数字都是奇数。结果如下：

```
winning_number  winning_number % 2
--------------  ------------------
       21              1
        8              0
       13              1
```

结果显示奇数的余数是`1`，偶数的余数是`0`。在第一行，`21 % 2`的结果是`1`，因为 21 除以 2 得到商 10，余数为 1。

使用`mod`或`%`运算符会得到相同的结果。模运算也可以通过`mod()`函数来实现。这些查询都会返回相同的结果：

```
select winning_number % 2     from roulette_winning_number;
select winning_number mod 2   from roulette_winning_number;
select mod(winning_number, 2) from roulette_winning_number;
```

#### 运算符优先级

当数学表达式中使用多个算术运算符时，`*`、`/`、`div`、`%`和`mod`会先被计算，`+`和`-`会最后计算。这被称为*运算符优先级*。以下查询（使用`payroll`表）是为了计算员工根据薪水、奖金和税率应支付的税款，但该查询返回了错误的税额：

```
select  employee,
        salary,
        bonus,
        tax_rate,
        salary + bonus * tax_rate
from    payroll;
```

结果是：

```
employee  salary     bonus     tax_rate  salary + bonus * tax_rate
--------  ---------  --------  --------  -------------------------
Max Bain   80000.00  10000.00      0.24                 82400.0000
Lola Joy   60000.00    800.00      0.18                 60144.0000
Zoe Ball  110000.00  30000.00      0.35                120500.0000
```

右侧的列应表示员工需要支付的税款，但似乎太高了。如果 Max Bain 的薪水是 80,000 美元，奖金是 10,000 美元，那么要求他支付 82,400 美元的税款似乎不合理。

查询返回了错误的值，因为你期望 MySQL 首先将`salary`和`bonus`相加，然后将结果乘以`tax_rate`。然而，MySQL 先将`bonus`乘以`tax_rate`，然后再加上`salary`。这是因为乘法的优先级高于加法。

为了修正这个问题，使用括号来告诉 MySQL 将`salary +` `bonus`作为一个整体处理：

```
select  employee,
        salary,
        bonus,
        tax_rate,
        **(**salary + bonus**)** * tax_rate
from    payroll;
```

结果是：

```
employee  salary     bonus     tax_rate  salary + bonus * tax_rate
--------  ---------  --------  --------  -------------------------
Max Bain   80000.00  10000.00      0.24                 21600.0000
Lola Joy   60000.00    800.00      0.18                 10944.0000
Zoe Ball  110000.00  30000.00      0.35                 49000.0000
```

现在，查询返回了 Max Bain 的 21,600 美元，这就是正确的值。你在进行计算时应该经常使用括号——不仅因为它能帮助你控制运算顺序，还因为它让你的 SQL 更加易读和易懂。

### 数学函数

MySQL 提供了许多数学函数，可以帮助处理诸如四舍五入、获取数字的绝对值、处理指数等任务，以及计算余弦、对数和弧度。

#### abs()

`abs()`函数获取一个数字的绝对值。一个数字的绝对值总是正数。例如，5 的绝对值是 5，–5 的绝对值是 5。

假设你举办了一个比赛，猜测罐子里有多少颗果冻豆。写一个查询，看看谁的猜测最接近实际数字 300：

```
select  guesser,
        guess,
        300          as actual,
        300 – guess  as difference
from    jelly_bean;
```

这里你从`jelly_bean`表中选择了猜测者的姓名和他们的猜测值。你选择了`300`并将该列别名为`actual`，这样它将在结果中显示该标题。然后你从 300 中减去猜测值，并将该列别名为`difference`。结果是：

```
guesser  guess actual  difference
-------  ----- ------  ----------
Ruth       275    300          25
Henry      350    300         -50
Ike        305    300          -5
```

`difference`列显示了猜测与实际值 300 之间的偏差，但结果有点难以理解。当猜测高于实际值 300 时，`difference`列显示为负数；当猜测低于实际值时，`difference`列显示为正数。对于你的比赛，你不关心猜测是高于还是低于 300，你只关心哪个猜测最接近 300。

你可以使用`abs()`函数从`difference`列中移除负数：

```
select  guesser,
        guess,
        300 as actual,
 **abs(**300 – guess**)** as difference
from    jelly_bean;
```

结果是：

```
guesser  guess actual  difference
-------  ----- ------  ----------
Ruth       275    300          25
Henry      350    300          50
Ike        305    300           5
```

现在你可以轻松地看到，Ike 赢得了你的比赛，因为他在`difference`列中的值是最小的。

#### ceiling()

`ceiling()`函数返回大于或等于参数的最小整数。如果你支付了$3.29 的油费，并想将该数字四舍五入到下一个整数，你可以写下以下查询：

```
select ceiling(3.29);
```

结果是：

```
ceiling(3.29)
-------------
      4
```

`ceiling()`函数有一个同义词`ceil()`，它返回相同的结果。

#### floor()

`floor()`函数返回小于或等于参数的最大整数。要将$3.29 四舍五入到最接近的整数，你可以写下以下查询：

```
select floor(3.29);
```

结果是：

```
floor(3.29)
-----------
      3
```

如果参数已经是整数，那么`ceiling()`和`floor()`函数都会返回该整数。例如，`ceiling(33)`和`floor(33)`都会返回`33`。

#### pi()

`pi()`函数返回 pi 的值，如本章开头所示。

#### degrees()

`degrees()`函数将弧度转换为角度。你可以通过这个查询将 pi 转换为角度：

```
select degrees(pi());
```

结果是：

```
degrees(pi())
-------------
     180
```

你通过将`pi()`函数包装在`degrees()`函数中得到了答案。

#### radians()

`radians()`函数将角度转换为弧度。你可以使用以下查询将 180 转换为弧度：

```
select radians(180);
```

你的结果是：

```
radians(180)
-----------------
3.141592653589793
```

该函数接收到参数`180`并返回了 pi 的值。

#### exp()

`exp()`函数返回自然对数底数*e*（2.718281828459）被你提供的数字（例如 2）作为指数时的结果：

```
select exp(2);
```

结果是：

```
7.38905609893065
```

该函数返回`7.38905609893065`，即*e*（2.718281828459）的平方。

#### log()

`log()`函数返回你提供的数字的自然对数：

```
select log(2);
```

结果是：

```
0.6931471805599453
```

MySQL 还提供了`log10()`函数，它返回以 10 为底的对数，以及`log2()`函数，它返回以 2 为底的对数。

`log()`函数可以接受两个参数：一个是数字的底数，另一个是该数字本身。例如，要计算 log2;
```

结果是：

```
log(2, 8)
--------
    3
```

该函数接收到两个参数，`2`和`8`，并返回值`3`。

#### mod()

`mod()`函数，如你之前看到的，是取模函数。它接受一个数字，将其除以另一个数字，并返回余数。

```
select mod(7, 2);
```

结果是：

```
mod(7, 2)
--------
    1
```

`mod(7,2)`函数的结果为`1`，因为 7 除以 2 的商为 3，余数为 1。取模运算也可以通过`%`运算符和`mod`运算符实现。

#### pow()

`pow()`函数返回一个数值的幂。要将 5 的 3 次方计算出来，你可以写下这个查询：

```
select pow(5, 3);
```

结果是：

```
pow(5, 3)
--------
   125
```

`pow()`函数有一个同义词`power()`，它返回相同的结果。

#### round()

本章前面介绍的`round()`函数用于四舍五入小数。要将数字 9.87654321 四舍五入到小数点后 3 位，可以使用以下查询：

```
select round(9.87654321, 3);
```

结果是：

```
round(9.87654321, 3)
-------------------
       9.877
```

要四舍五入所有小数数值，可以只用一个参数调用`round()`：

```
select round(9.87654321);
```

结果是：

```
round(9.87654321)
-------------------
         10
```

如果调用`round()`时没有提供可选的第二个参数，它会默认四舍五入到小数点后 0 位。

#### truncate()

`truncate()` 函数将数字截断到指定的小数位数。要将数字 9.87654321 截断到小数点后三位，请使用以下查询：

```
select truncate(9.87654321, 3);
```

结果是：

```
truncate(9.87654321, 3)
-----------------------
       9.876
```

要截断所有小数部分，可以将 `truncate()` 函数的第二个参数设为 `0`：

```
select truncate(9.87654321, 0);
```

结果是：

```
truncate(9.87654321, 0)
----------------------
          9
```

`truncate()` 函数通过移除数字来将数字转换为小数点后的指定位数。这与 `round()` 函数不同，后者在去除数字之前会四舍五入。

#### sin()

`sin()` 函数返回一个以弧度表示的数的正弦值。你可以使用这个查询来得到 2 的正弦值：

```
select sin(2);
```

结果是：

```
sin(2)
------------------
0.9092974268256817
```

函数接收到 `2` 作为参数，并返回值 `0.9092974268256817`。

#### cos()

`cos()` 函数返回一个以弧度表示的数的余弦值。使用以下查询可以得到 2 的余弦值：

```
select cos(2);
```

结果是：

```
cos(2)
-------------------
-0.4161468365471424
```

函数接收到 `2` 作为参数，并返回值 `-0.4161468365471424`。

#### sqrt()

`sqrt()` 函数返回一个数的平方根。你可以像这样计算 16 的平方根：

```
sqrt(16)
--------
   4
```

函数接收到 `16` 作为参数，并返回值 `4`。

#### stddev_pop()

`stddev_pop()` 函数返回提供的数值的总体标准差。*总体标准差*是考虑数据集中所有值时的标准差。例如，查看包含你所有考试成绩的 `test_score` 表：

```
score
-----
  70
  82
  97
```

现在编写查询来获取考试成绩的总体标准差：

```
select  stddev_pop(score)
from    test_score;
```

结果是：

```
stddev_pop(score)
------------------
11.045361017187261
```

`std()` 和 `stddev()` 函数是 `stddev_pop()` 的同义词，会产生相同的结果。

若要获取样本值的标准差，而不是整个数据集的标准差，你可以使用 `stddev_samp()` 函数。

#### tan()

`tan()` 函数接受弧度作为参数并返回正切值。例如，你可以通过以下查询获取 3.8 的正切值：

```
select tan(3.8);
```

结果是：

```
0.7735560905031258
```

函数接收到 `3.8` 作为参数，并返回值 `0.7735560905031258`。

## 其他实用函数

其他有用的函数包括 `cast()`、`coalesce()`、`distinct()`、`database()`、`if()` 和 `version()`。

### cast()

`cast()` 函数将一个值从一种数据类型转换为另一种数据类型。调用 `cast()` 函数时，将值作为第一个参数传入 `cast()`，接着使用 `as` 关键字，指定要转换成的目标数据类型。

例如，从名为 `online_order` 的表中选择 `datetime` 列 `order_datetime`：

```
select  order_datetime
from    online_order;
```

你的结果显示以下 `datetime` 值：

```
order_datetime
-------------------
2024-12-08 11:39:09
2024-12-10 10:11:14
```

你可以通过将 `datetime` 数据类型转换为 `date` 数据类型，来选择没有时间部分的值，像这样：

```
select  cast(order_datetime as date)
from    online_order;
```

你的结果是：

```
cast(order_datetime as date)
----------------------------
         2024-12-08
         2024-12-10
```

`datetime` 的日期部分现在显示为 `date` 值。

### coalesce()

`coalesce()` 函数返回列表中第一个非空值。你可以指定空值后跟非空值，`coalesce()` 会返回非空值：

```
select coalesce(null, null, 42);
```

结果是：

```
coalesce(null, null, 42)
------------------------
            42
```

`coalesce()`函数在你想要在结果中显示某个值而不是`null`时也非常有用。例如，在以下查询中使用的`candidate`表中，`employer`列有时会存储候选人的雇主名称，其他时候该列会是`null`。为了显示`Between Jobs`而不是`null`，你可以输入以下内容：

```
select employee_name,
       coalesce(employer, 'Between Jobs')
from   candidate;
```

结果是：

```
employee_name  employer
-------------  ------------
Jim Miller     Acme Corp
Laura Garcia   Globex
Jacob Davis    Between Jobs
```

现在查询显示的是 Jacob Davis 的`Between Jobs`，而不是`null`，这更加信息丰富，特别是对于那些不理解`null`含义的非技术用户来说。

### distinct()

当你有重复的值时，可以使用`distinct()`函数使每个值只显示一次。例如，如果你想知道你的客户来自哪些国家，可以像这样查询`customer`表：

```
select country
from   customer;
```

结果是：

```
country
-------
India
USA
USA
USA
India
Peru
```

查询返回了`customer`表中每一行的`country`列值。你可以使用`distinct()`函数使结果集中每个国家只显示一次：

```
select distinct(country)
from   customer;
```

现在结果是：

```
country
-------
India
USA
Peru
```

`distinct()`函数也可以作为操作符使用。要使用它，去掉括号，如下所示：

```
select distinct country
from   customer;
```

结果集是相同的：

```
country
-------
India
USA
Peru
```

`distinct()`函数在与`count()`函数结合使用时尤其有用，用来找出你表中有多少个唯一的值。这里你写一个查询来计算表中不同国家的数量：

```
select count(distinct country)
from   customer;
```

结果是：

```
count(distinct country)
-----------------------
           3
```

你使用`distinct()`函数识别了不同的国家，并将它们包裹在`count()`函数中以获取数量。

### database()

`database()`函数告诉你当前使用的是哪个数据库。正如在第二章中所看到的，`use`命令允许你选择要使用的数据库。在你的一天中，你可能会在不同的数据库之间切换，忘记当前的数据库。你可以像这样调用`database()`函数：

```
use airport;

select database();
```

结果是：

```
database()
----------
 airport
```

如果你不在你以为自己所在的数据库中，并且你尝试查询一个表，MySQL 会给出错误，说明该表不存在。调用`database()`是一种快速检查的方式。

### if()

`if()`函数根据条件是否为`true`或`false`返回不同的值。`if()`函数接受三个参数：你要测试的条件、条件为`true`时返回的值、条件为`false`时返回的值。

让我们写一个查询，列出学生及其是否通过考试。`test_result`表包含以下数据：

```
student_name  grade
------------  -----
Lisa          98
Bart          41
Nelson        11
```

你检查每个学生是否通过考试的查询应该类似于以下内容：

```
select  student_name,
        if(grade > 59, 'pass', 'fail')
from    test_result;
```

你正在测试的条件是学生的`grade`是否大于`59`。如果是，你返回文本`pass`。如果不是，你返回文本`fail`。结果是：

```
student_name  if(grade > 59, 'pass', 'fail')
------------  ------------------------------
Lisa                       pass
Bart                       fail
Nelson                     fail
```

MySQL 还具有 `case` 运算符，它允许你执行比 `if()` 函数更复杂的逻辑。`case` 运算符允许你测试多个条件，并返回第一个满足条件的结果。在以下查询中，你根据学生的成绩选择学生姓名，并为学生添加评论：

```
select  student_name,
case
  when grade < 30 then 'Please retake this exam'
  when grade < 60 then 'Better luck next time'
  else 'Good job'
end
from test_result;
```

`case` 运算符使用匹配的 `end` 关键字来标志 `case` 语句的结束。

对于任何得分低于 30 分的学生，`case` 语句将返回 `Please retake this exam`，然后控制权传递到 `end` 语句。

得分 30 分或以上的学生不会被第一个 `when` 条件处理，因此控制权转到下一行。

如果学生的成绩为 30 分或更高，但低于 60 分，将返回 `Better luck next time`，然后控制权传递到 `end` 语句。

如果学生的成绩不符合任一 `when` 条件，即学生的分数高于 60，控制权将转到 `else` 关键字，返回 `Good job`。你可以使用 `else` 子句来捕捉任何不符合前两个条件的学生成绩。结果是：

```
student_name  case when grade < 30 then 'Please...
------------  ------------------------------------
Lisa          Good job
Bart          Better luck next time
Nelson        Please retake this exam
```

与 `if()` 函数不同——`if()` 函数在条件为 `true` 或 `false` 时返回结果——`case` 允许你检查多个条件，并根据第一个满足的条件返回结果。

### version()

`version()` 函数返回你正在使用的 MySQL 版本：

```
select version();
```

结果是：

```
version
-------
8.0.27
```

我服务器上安装的 MySQL 版本是 8.0.27。你的版本可能不同。

## 总结

在本章中，你学习了如何调用 MySQL 内置函数并向这些函数传递值，这些值被称为参数。你探索了最有用的函数，并了解了如何在需要时查找那些较不常见的函数。在下一章中，你将学习如何从 MySQL 数据库中插入、更新和删除数据。
