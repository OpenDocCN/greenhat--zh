# 第十章：创建视图

![](img/chapterart.png)

在本章中，你将学习如何创建和使用视图。*视图*是基于你编写的查询输出的虚拟表，用于定制结果集的显示。每次你从视图中选择数据时，MySQL 都会重新执行定义视图时的查询，返回最新的结果，作为类似表格的结构，包含行和列。

视图在你想简化复杂查询或隐藏敏感或无关数据的情况下非常有用。

## 创建新视图

你使用`create view`语法来创建一个视图。让我们看一个包含以下`course`表的示例：

```
course_name                              course_level
---------------------------------------  ------------
Introduction to Python                   beginner
Introduction to HTML                     beginner
React Full-Stack Web Development         advanced
Object-Oriented Design Patterns in Java  advanced
Practical Linux Administration           advanced
Learn JavaScript                         beginner
Advanced Hardware Security               advanced
```

在这里，你创建了一个名为`v_course_beginner`的视图，从`course`表中选择所有`course_level`为`beginner`的列：

```
create view v_course_beginner as
select *
from   course
where  level = 'beginner';
```

执行此语句将创建视图并将其保存在你的 MySQL 数据库中。现在你可以随时查询`v_course_beginner`视图，如下所示：

```
select * from v_course_beginner;
```

结果如下：

```
course_name             course_level
----------------------  ------------
Introduction to Python  beginner
Introduction to HTML    beginner
Learn JavaScript        beginner
```

由于你通过从`course`表中选择`*`（通配符字符）来定义视图，因此它具有与表相同的列名。

`v_course_beginner`视图应由初学者使用，因此你只选择了`course_level`为`beginner`的课程，隐藏了高级课程。

现在为高级学生创建第二个视图，仅包含高级课程：

```
create view v_course_advanced as
select *
from   courses
where  level = 'advanced';
```

从`v_course_advanced`视图中选择显示高级课程：

```
select * from v_course_advanced;
```

结果如下：

```
course_name                              course_level
---------------------------------------  ------------
React Full-Stack Web Development         advanced
Object-Oriented Design Patterns in Java  advanced
Practical Linux Administration           advanced
Advanced Hardware Security               advanced
```

当你定义`v_course_advanced`视图时，你提供了一个查询，该查询从`course`表中选择数据。每次使用视图时，MySQL 都会执行该查询，这意味着视图始终会显示`course`表中最新的行。在这个示例中，任何新添加到`course`表中的高级课程都会在每次从`v_course_advanced`视图中选择时显示。

这种方法允许你在`course`表中维护课程，并为初学者和高级学生提供不同的数据视图。

## 使用视图隐藏列值

在`course`表示例中，你创建了显示表中某些行并隐藏其他行的视图。你还可以创建显示不同*列*的视图。

让我们看一个使用视图隐藏敏感列数据的示例。你有两个表，`company`和`complaint`，它们帮助跟踪本地公司的投诉。

`company`表如下：

```
company_id  company_name          owner          owner_phone_number
----------  --------------------  -------------  ------------------
1           Cattywampus Cellular  Sam Shady         784-785-1245
2           Wooden Nickel Bank    Oscar Opossum     719-997-4545
3           Pitiful Pawn Shop     Frank Fishy       917-185-7911
```

这是`complaint`表：

```
complaint_id  company_id  complaint_desc
------------  ----------  ------------------------------
1                 1       Phone doesn't work
2                 1       Wi-Fi is on the blink
3                 1       Customer service is bad
4                 2       Bank closes too early
5                 3       My iguana died
6                 3       Police confiscated my purchase
```

你将首先编写一个查询来选择每个公司及其接收的投诉数量：

```
select   a.company_name,
         a.owner,
         a.owner_phone_number,
         count(*)
from     company a
join     complaint b
on       a.company_id = b.company_id
group by a.company_name,
         a.owner,
         a.owner_phone_number;
```

结果如下：

```
company_name         owner          owner_phone_number  count(*)
-------------------- -------------  ------------------  --------
Cattywampus Cellular Sam Shady         784-785-1245         3
Wooden Nickel Bank   Oscar Opossum     719-997-4545         1
Pitiful Pawn Shop    Frank Fishy       917-185-7911         2
```

要在名为`v_complaint`的视图中显示此查询的结果，只需在原始查询的第一行添加`create view`语法：

```
**create view v_complaint as**
select   a.company_name,
         a.owner,
         a.owner_phone_number,
         count(*)
from     company a
join     complaint b
on       a.company_id = b.company_id
group by a.company_name,
         a.owner,
         a.owner_phone_number;
```

现在，下次你想获取公司及其投诉计数的列表时，你可以简单地输入`select * from v_complaint`，而无需重写整个查询。

接下来，你将创建另一个隐藏所有者信息的视图。你将命名该视图为`v_complaint_public`，并允许所有数据库用户访问该视图。该视图将显示公司名称和投诉数量，但不显示所有者的姓名或电话号码：

```
create view v_complaint_public as
select   a.company_name,
         count(*)
from     company a
join     complaint b
on       a.company_id = b.company_id
group by a.company_name;
```

你可以像这样查询视图：

```
select * from v_complaint_public;
```

结果是：

```
company_name         count(*)
-------------------- --------
Cattywampus Cellular    3
Wooden Nickel Bank      1
Pitiful Pawn Shop       2
```

这是使用视图来隐藏存储在列中的数据的一个例子。虽然所有者的联系信息存储在你的数据库中，但你通过在`v_complaint_public`视图中不选择这些列来隐藏这些信息。

一旦你创建了视图，就可以像使用表一样使用它们。例如，你可以将视图与表连接，将视图与其他视图连接，并在子查询中使用视图。

## 从视图中插入、更新和删除数据

在第九章中，你学习了如何插入、更新和删除表中的行。在某些情况下，也可以通过视图修改行。例如，`v_course_beginner`视图是基于`course`表的。你可以使用以下`update`语句更新该视图：

```
update  v_course_beginner
set     course_name = 'Introduction to Python 3.1'
where   course_name = 'Introduction to Python';
```

这个`update`语句更新了`v_course_beginner`视图底层`course`表中的`course_name`列。MySQL 能够执行该更新，因为视图和表非常相似；对于`v_course_beginner`视图中的每一行，`course`表中都有一行。

现在，尝试用类似的查询更新`v_complaint`视图：

```
update  v_complaint
set     owner_phone_number = '578-982-1277'
where   owner = 'Sam Shady';
```

你会收到以下错误消息：

```
Error Code: 1288\. The target table v_complaint of the UPDATE is not updatable
```

MySQL 不允许你更新`v_complaint`视图，因为它是通过多个表和`count()`聚合函数创建的。它比`v_course_beginner`视图更复杂。关于哪些视图允许更新、插入或删除行的规则相当复杂。因此，我建议直接从表中更改数据，而避免将视图用于此目的。

## 删除视图

要删除视图，使用`drop view`命令：

```
drop view v_course_advanced;
```

虽然视图已从数据库中删除，但底层表仍然存在。

## 索引与视图

你不能为视图添加索引以加速查询，但 MySQL 可以使用底层表上的任何索引。例如，下面的查询

```
select  *
from    v_complaint
where   company_name like 'Cattywampus%';
```

可以利用`company`表中`company_name`列上的索引，因为`v_complaint`视图是基于`company`表创建的。

## 总结

在本章中，你了解了如何使用视图来提供数据的自定义表示。在下一章中，你将学习如何编写函数和过程，并向其中添加逻辑，根据数据值执行特定任务。
