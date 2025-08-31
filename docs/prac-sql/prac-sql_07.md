# 第七章：在关系数据库中连接表

![](img/chapterart.png)

在第二章中，我介绍了*关系数据库*的概念，这是一种支持数据跨多个相关表存储的应用程序。在关系模型中，每张表通常保存一个单独实体的数据——如学生、汽车、购买、房屋——表中的每一行描述该实体之一。一个被称为*表连接*的过程，允许我们将一张表中的行与其他表中的行进行连接。

关系数据库的概念来源于英国计算机科学家埃德加·F·科德（Edgar F. Codd）。1970 年，在为 IBM 工作时，他发布了一篇名为《大型共享数据银行的数据关系模型》（A Relational Model of Data for Large Shared Data Banks）的论文。他的观点彻底改变了数据库设计，并推动了 SQL 的发展。通过关系模型，你可以构建消除重复数据、更易维护并且在编写查询时提供更高灵活性的表，从而精确地获取你需要的数据。

## 使用 JOIN 链接表

要在查询中连接表，我们使用`JOIN ... ON`结构（或者本章我将介绍的其他`JOIN`变体）。`JOIN`是 ANSI SQL 标准的一部分，它通过`ON`子句中的*布尔*值表达式将一张表与另一张表连接。常用的语法测试相等性，通常采用如下形式：

```
SELECT * 
FROM `table_a` JOIN `table_b`
ON `table_a.key_column` = `table_b.foreign_key_column`
```

这与你已经学习的基本`SELECT`类似，但我们不再在`FROM`子句中指定一个表，而是命名一个表，给出`JOIN`关键字，然后再命名第二个表。接着是`ON`子句，我们在这里使用等式比较操作符放置一个表达式。当查询执行时，它会返回两个表中`ON`子句表达式为`true`的行，意味着指定列中的值相等。

你可以使用任何评估为*布尔*结果`true`或`false`的表达式。例如，你可以匹配一个列中的值是否大于或等于另一个列中的值：

```
ON `table_a.key_column` >= `table_b.foreign_key_column`
```

这种情况很少见，但如果你的分析需要，还是可以选择这种方式。

## 使用关键列关联表

这是一个使用关键列关联表的例子：假设你是一个数据分析师，任务是检查一个公共机构按部门划分的薪资支出。你向该机构提交了一份信息自由法案（Freedom of Information Act）请求，期望收到一份简单的电子表格，列出每个员工及其薪水，格式如下：

```
dept    location    first_name    last_name    salary
----    --------    ----------    ---------    ------
IT      Boston      Julia         Reyes        115300
IT      Boston      Janet         King          98000
Tax     Atlanta     Arthur        Pappas        72700
Tax     Atlanta     Michael       Taylor        89500
```

但实际上并不是这样。相反，机构向你发送了来自其薪资系统的数据转储：十几个 CSV 文件，每个文件代表数据库中的一张表。你阅读了文档，了解了数据布局（一定要记得请求这份文档！），并开始理解每张表中的列。两张表特别突出：一张名为`employees`，另一张名为`departments`。

使用 列表 7-1 中的代码，让我们创建这些表的版本，插入行，并检查如何连接两个表中的数据。使用你为这些练习创建的 `analysis` 数据库，运行所有代码，然后通过使用基本的 `SELECT` 语句查看数据，或通过点击 pgAdmin 中的表名并选择 **查看/编辑数据**▶**所有行**。

```
CREATE TABLE departments (
    dept_id integer,
    dept text,
    city text,
  1 CONSTRAINT dept_key PRIMARY KEY (dept_id),
  2 CONSTRAINT dept_city_unique UNIQUE (dept, city)
);

CREATE TABLE employees (
    emp_id integer,
    first_name text,
    last_name text,
    salary numeric(10,2),
  3 dept_id integer REFERENCES departments (dept_id),
  4 CONSTRAINT emp_key PRIMARY KEY (emp_id)
);

INSERT INTO departments
VALUES
    (1, 'Tax', 'Atlanta'),
    (2, 'IT', 'Boston');

INSERT INTO employees
VALUES
    (1, 'Julia', 'Reyes', 115300, 1),
    (2, 'Janet', 'King', 98000, 1),
    (3, 'Arthur', 'Pappas', 72700, 2),
    (4, 'Michael', 'Taylor', 89500, 2);
```

列表 7-1：创建 `departments` 和 `employees` 表

这两个表遵循 Codd 的关系模型，因为每个表描述了一个单一实体的属性：机构的部门和员工。在 `departments` 表中，你应该看到以下内容：

```
dept_id    dept    city
-------    ----    -------
      1    Tax     Atlanta
      2    IT      Boston
```

`dept_id` 列是表的主键。*主键*是一个列或列的集合，其值唯一标识表中的每一行。有效的主键列会强制执行某些约束：

+   列或列的集合必须为每一行提供唯一值。

+   列或列的集合不能有缺失值。

你使用 `CONSTRAINT` 关键字为 `departments` 1 和 `employees` 4 定义了主键，我将在第八章中详细介绍其他约束类型。`dept_id` 中的值唯一标识 `departments` 中的每一行，尽管这个例子只包含了部门名称和城市，但这个表可能还会包括其他信息，比如地址或联系信息。

`employees` 表应该包含以下内容：

```
emp_id first_name last_name  salary   dept_id
------ ---------- --------- --------- -------
     1 Julia      Reyes     115300.00       1
     2 Janet      King       98000.00       1
     3 Arthur     Pappas     72700.00       2
     4 Michael    Taylor     89500.00       2
```

`emp_id` 中的值唯一标识 `employees` 表中的每一行。为了确定每个员工所属的部门，表中包含了一个 `dept_id` 列。该列中的值引用 `departments` 表的主键中的值。我们称之为*外键*，你在创建表时作为约束 3 添加。外键约束要求其值在它所引用的列中已经存在。通常，这些值是另一个表的主键，但它也可以引用任何具有唯一值的列。因此，`employees` 表中的 `dept_id` 值必须在 `departments` 表中的 `dept_id` 存在；否则，你不能添加它们。这有助于维护数据的完整性。与主键不同，外键列可以为空，并且可以包含重复值。

在这个例子中，与员工 `Julia Reyes` 关联的 `dept_id` 为 `1`；这指的是 `departments` 表中主键 `dept_id` 的值 `1`。这告诉我们 `Julia Reyes` 是位于 `Atlanta` 的 `Tax` 部门的一员。

`departments`表还包括一个`UNIQUE`约束，我将在下一章的“UNIQUE 约束”中更详细地讨论。简而言之，它保证某一列中的值，或多列值的组合，是唯一的。在这里，它要求每一行在`dept`和`city`列中都有一对唯一的值，这有助于避免重复数据——例如，表中不会有两个名为`Tax`的亚特兰大部门。通常，你可以使用这种唯一的组合来创建一个*自然键*作为主键，我们将在下一章进一步讨论。

你可能会问：将数据拆分成这样组成的好处是什么？那么，考虑一下如果你按照最初的想法，将所有数据都放在一个表中，数据会是什么样子：

```
dept    location    first_name    last_name    salary
----    --------    ----------    ---------    ------
IT      Boston      Julia         Reyes        115300
IT      Boston      Janet         King          98000
Tax     Atlanta     Arthur        Pappas        72700
Tax     Atlanta     Michael       Taylor        89500
```

首先，当你将来自不同实体的数据组合到一个表中时，不可避免地需要重复信息。在这里，部门名称和位置会为每个员工重复显示。当表仅包含四行数据时，或者即使是 4,000 行时，这种重复是可以接受的。但当表中有数百万行时，重复的冗长字符串既冗余又浪费宝贵的空间。

其次，将所有数据压缩到一个表中会使得数据管理变得困难。如果营销部门的名字更改为品牌营销（Brand Marketing）怎么办？表中的每一行都需要更新，如果有人不小心只更新了一部分行而不是所有行，就可能会引入错误。在这种模型中，更新部门名称要简单得多——只需更改表中的一行。

最后，信息被组织或*规范化*在多个表中，并不会妨碍我们将其作为一个整体来查看。我们总是可以通过`JOIN`查询数据，将多个表的列合并在一起。

现在你已经了解了表如何关联的基本知识，接下来我们来看看如何在查询中连接它们。

## 使用`JOIN`查询多个表

当你在查询中连接表时，数据库会在两个表中连接你指定用于连接的列，只有当这些列的值使得`ON`子句的表达式返回`true`时，才会连接。查询结果会包含来自两个表的列，前提是你在查询中要求它们。你还可以使用连接表中的列，通过`WHERE`子句过滤结果。

连接表的查询在语法上与基本的`SELECT`语句类似。不同之处在于查询还指定了以下内容：

+   要连接的表和列，使用 SQL `JOIN ... ON`结构

+   使用`JOIN`关键字的变体来指定要执行的连接类型

让我们先看一下`JOIN ... ON`结构的语法，然后再探索不同类型的连接。为了连接示例中的`employees`和`departments`表，并查看两个表的所有相关数据，可以从编写类似 Listing 7-2 的查询开始。

```
1 SELECT *
2 FROM employees JOIN departments
3 ON employees.dept_id = departments.dept_id
ORDER BY employees.dept_id;
```

Listing 7-2: 连接`employees`和`departments`表

在这个示例中，你在`SELECT`语句中使用了一个星号通配符，以便从查询中使用的所有表中包含所有列。接下来，在`FROM`子句中，你将`JOIN`关键字放置在你想要连接的两个表之间。最后，你使用`ON`子句指定要评估的表达式。对于每个表，你提供表名、一个句点和包含关键值的列名。两个表和列名之间用等号连接。

当你运行查询时，结果将包括两个表中所有在`dept_id`列中匹配的值。事实上，甚至`dept_id`列也会出现两次，因为你选择了两个表的所有列：

```
emp_id   first_name   last_name   salary      dept_id   dept_id   dept   city
------   ----------   ---------   ---------   -------   -------   ----   -------
     1   Julia        Reyes       115300.00         1         1   Tax    Atlanta
     2   Janet        King         98000.00         1         1   Tax    Atlanta
     3   Arthur       Pappas       72700.00         2         2   IT     Boston
     4   Michael      Taylor       89500.00         2         2   IT     Boston
```

因此，即使数据存在于两个表中，每个表都有一组专注的列，你仍然可以查询这些表，将相关数据汇总到一起。在本章稍后的“在连接中选择特定列”一节中，我将向你展示如何从两个表中只检索你需要的列。

## 理解 JOIN 类型

在 SQL 中有多种连接表的方法，使用哪种连接取决于你希望如何检索数据。以下列表描述了不同类型的连接。在回顾每种连接时，思考将两个表并排放置，一个在`JOIN`关键字的左边，另一个在右边会很有帮助。每种连接的具体数据驱动示例如下所示：

1.  `JOIN` 返回两个表中在连接列中找到匹配值的行。另一种语法是`INNER JOIN`。

1.  `LEFT JOIN` 返回左侧表中的每一行。当 SQL 在右侧表中找到匹配的值时，这一行的值会包含在结果中。否则，右侧表的值不会显示。

1.  `RIGHT JOIN` 返回右侧表中的每一行。当 SQL 在左侧表中找到匹配的值时，这一行的值会包含在结果中。否则，左侧表的值不会显示。

1.  `FULL OUTER JOIN` 返回两个表中的每一行，并在连接列中的值匹配时合并这些行。如果左侧或右侧表中的某个值没有匹配项，则查询结果中不会显示该表的任何值。

1.  `CROSS JOIN` 返回两个表中所有可能的行组合。

让我们通过数据来看看这些连接是如何运作的。假设你有两个简单的表，分别存储一个学区的学校名称，学区计划未来的入学情况：`district_2020` 和 `district_2035`。`district_2020` 中有四行数据：

```
id          school_2020
--    ------------------------
 1    Oak Street School
 2    Roosevelt High School
 5    Dover Middle School
 6    Webutuck High School
```

`district_2035` 中有五行数据：

```
id         school_2035
--    ---------------------
 1    Oak Street School
 2    Roosevelt High School
 3    Morrison Elementary
 4    Chase Magnet Academy
 6    Webutuck High School
```

请注意，学区预计会随着时间发生变化。只有`id`为`1`、`2`和`6`的学校同时出现在两个表中，而其他学校仅出现在其中一个表中。这种情况很常见，也是数据分析师的一个常见初始任务——特别是当你处理的表比这两个表有更多行时——你可以使用 SQL 来识别哪些学校同时出现在两个表中。使用不同的连接可以帮助你找到这些学校以及其他相关信息。

再次使用你的`analysis`数据库，运行 Listing 7-3 中的代码，来构建和填充这两个表。

```
CREATE TABLE district_2020 (
  1 id integer CONSTRAINT id_key_2020 PRIMARY KEY,
    school_2020 text
);

CREATE TABLE district_2035 (
  2 id integer CONSTRAINT id_key_2035 PRIMARY KEY,
    school_2035 text
);

3 INSERT INTO district_2020 VALUES
    (1, 'Oak Street School'),
    (2, 'Roosevelt High School'),
    (5, 'Dover Middle School'),
    (6, 'Webutuck High School');

INSERT INTO district_2035 VALUES
    (1, 'Oak Street School'),
    (2, 'Roosevelt High School'),
    (3, 'Morrison Elementary'),
    (4, 'Chase Magnet Academy'),
    (6, 'Webutuck High School');
```

Listing 7-3: 创建两个表以探索`JOIN`类型

我们创建并填充了两个表：这些声明现在应该看起来很熟悉，但有一个新的元素：我们为每个表添加了主键。在`district_2020`的`id`列 1 和`district_2035`的`id`列 2 之后，关键字`CONSTRAINT` `key_name` `PRIMARY KEY`表明这些列将作为它们各自表的主键。这意味着每个表中的每一行，`id`列都必须填充，并且包含该表中每一行唯一的值。最后，我们使用熟悉的`INSERT`语句 3 将数据添加到表中。

### JOIN

当我们希望返回仅包含在两个表中匹配列值的行时，我们使用`JOIN`或`INNER JOIN`。要查看这个示例，请运行 Listing 7-4 中的代码，它连接了你刚刚创建的两个表。

```
SELECT *
FROM district_2020 JOIN district_2035
ON district_2020.id = district_2035.id
ORDER BY district_2020.id;
```

Listing 7-4: 使用`JOIN`

类似于我们在 Listing 7-2 中使用的方法，我们在`JOIN`关键字两侧指定要连接的两个表。然后，在`ON`子句中，我们指定用于连接的表达式，在这个例子中是两个表的`id`列的相等性。两个表中都有三个学校 ID，因此查询只返回这三个 ID 匹配的行。仅存在于其中一个表中的学校不会出现在结果中。还要注意，`JOIN`关键字左侧表中的列会显示在结果表的左侧：

```
id      school_2020      id      school_2035
-- --------------------- -- ---------------------
 1 Oak Street School      1 Oak Street School
 2 Roosevelt High School  2 Roosevelt High School
 6 Webutuck High School   6 Webutuck High School
```

什么时候应该使用`JOIN`？通常，当你处理结构良好、维护良好的数据集时，并且需要找到在所有连接的表中都存在的行。因为`JOIN`不会返回仅在一个表中存在的行，所以如果你想查看一个或多个表中的所有数据，请使用其他类型的连接。

#### 使用 USING 的 JOIN

如果你在`JOIN`的`ON`子句中使用相同名称的列，你可以通过用`USING`子句替代`ON`子句，减少冗余输出并简化查询语法，如在 Listing 7-5 中所示。

```
SELECT *
FROM district_2020 JOIN district_2035
1 USING (id)
ORDER BY district_2020.id;
```

Listing 7-5: 使用`USING`的`JOIN`

在指定要连接的表后，我们添加`USING` 1，后面跟着括号中的列名，这个列名是两个表用于连接的列——在这个例子中是`id`。如果我们要在多个列上进行连接，我们将在括号内用逗号分隔它们。运行查询后，你应该看到以下结果：

```
id      school_2020           school_2035
-- --------------------- ---------------------
 1 Oak Street School     Oak Street School
 2 Roosevelt High School Roosevelt High School
 6 Webutuck High School  Webutuck High School
```

注意，`id`在这个`JOIN`中出现在两个表中，并且具有相同的值，因此它只显示一次。这是一个简单且方便的简写方式。

### LEFT JOIN 和 RIGHT JOIN

与`JOIN`相比，`LEFT JOIN`和`RIGHT JOIN`关键字分别返回一个表的所有行，并且当另一个表中存在匹配值的行时，会将该行的值包括在结果中。否则，另一个表中的值不会显示。

让我们先看看 `LEFT JOIN` 的实际操作。执行 列表 7-6 中的代码。

```
SELECT *
FROM district_2020 LEFT JOIN district_2035
ON district_2020.id = district_2035.id
ORDER BY district_2020.id;
```

列表 7-6：使用 `LEFT JOIN`

查询的结果显示了来自 `district_2020` 的所有四行，这些数据位于连接的左侧，以及 `district_2035` 中与 `id` 列值匹配的三行。因为 `district_2035` 的 `id` 列中没有值为 `5` 的数据，所以没有匹配项，因此 `LEFT` `JOIN` 会在右侧返回一行空数据，而不是像 `JOIN` 那样完全省略左表中的行。最后，`district_2035` 中没有与 `district_2020` 匹配的任何值的行会从结果中省略：

```
id      school_2020      id      school_2035
-- --------------------- -- ---------------------
 1 Oak Street School      1 Oak Street School
 2 Roosevelt High School  2 Roosevelt High School
 5 Dover Middle School
 6 Webutuck High School   6 Webutuck High School
```

通过运行 `RIGHT JOIN`，我们可以看到类似但相反的行为，正如 列表 7-7 中所示。

```
SELECT *
FROM district_2020 RIGHT JOIN district_2035
ON district_2020.id = district_2035.id
ORDER BY district_2035.id;
```

列表 7-7：使用 `RIGHT JOIN`

这次，查询返回了来自 `district_2035` 的所有行，这些数据位于连接的右侧，以及 `district_2020` 中 `id` 列值匹配的行。查询结果会省略没有与 `district_2035` 在 `id` 上匹配的 `district_2020` 行：

```
id      school_2020      id      school_2035
-- --------------------- -- ---------------------
 1 Oak Street School      1 Oak Street School
 2 Roosevelt High School  2 Roosevelt High School
                          3 Morrison Elementary
                          4 Chase Magnet Academy
 6 Webutuck High School   6 Webutuck High School
```

在以下几种情况下，你可以使用这些连接类型之一：

+   你希望查询结果包含其中一个表的所有行。

+   你希望查找一个表中的缺失值。例如，在比较表示两个不同时间段的实体数据时。

+   当你知道某些连接表中的行没有匹配的值时。

和 `JOIN` 一样，如果表满足条件，你可以用 `USING` 子句替换 `ON` 子句。

### FULL OUTER JOIN

当你希望在连接中查看两个表的所有行时，无论是否匹配任何数据，都可以使用 `FULL OUTER JOIN` 选项。要查看其效果，请运行 列表 7-8。

```
SELECT *
FROM district_2020 FULL OUTER JOIN district_2035
ON district_2020.id = district_2035.id
ORDER BY district_2020.id;
```

列表 7-8：使用 `FULL OUTER JOIN`

结果返回了左表中的每一行，包括匹配的行以及右表中缺失行的空白数据，后面跟着右表中剩余的缺失行：

```
id      school_2020      id      school_2035
-- --------------------- -- ---------------------
 1 Oak Street School      1 Oak Street School
 2 Roosevelt High School  2 Roosevelt High School
 5 Dover Middle School
 6 Webutuck High School   6 Webutuck High School
                          3 Morrison Elementary
                          4 Chase Magnet Academy
```

虽然 `FULL OUTER JOIN` 明确来说不如内连接以及左连接或右连接那样常用且有用，但你仍然可以在一些任务中使用它：比如连接两个部分重叠的数据源，或者可视化表格之间共享匹配值的程度。

### CROSS JOIN

在 `CROSS JOIN` 查询中，结果（也称为 *笛卡尔积*）将左表中的每一行与右表中的每一行配对，展示所有可能的行组合。列表 7-9 显示了 `CROSS JOIN` 的语法；因为该连接不需要在关键列之间查找匹配项，因此无需提供 `ON` 子句。

```
SELECT *
FROM district_2020 CROSS JOIN district_2035
ORDER BY district_2020.id, district_2035.id;
```

列表 7-9：使用 `CROSS JOIN`

结果有 20 行——即左表中的四行与右表中的五行的乘积：

```
id      school_2020      id      school_2035      
-- --------------------- -- ---------------------
 1 Oak Street School      1 Oak Street School
 1 Oak Street School      2 Roosevelt High School
 1 Oak Street School      3 Morrison Elementary
 1 Oak Street School      4 Chase Magnet Academy
 1 Oak Street School      6 Webutuck High School
 2 Roosevelt High School  1 Oak Street School
 2 Roosevelt High School  2 Roosevelt High School
 2 Roosevelt High School  3 Morrison Elementary
 2 Roosevelt High School  4 Chase Magnet Academy
 2 Roosevelt High School  6 Webutuck High School
 5 Dover Middle School    1 Oak Street School
 5 Dover Middle School    2 Roosevelt High School
 5 Dover Middle School    3 Morrison Elementary
 5 Dover Middle School    4 Chase Magnet Academy
 5 Dover Middle School    6 Webutuck High School
 6 Webutuck High School   1 Oak Street School
 6 Webutuck High School   2 Roosevelt High School
 6 Webutuck High School   3 Morrison Elementary
 6 Webutuck High School   4 Chase Magnet Academy
 6 Webutuck High School   6 Webutuck High School
```

除非你想要一个超长的咖啡休息时间，否则建议避免在大型表上使用`CROSS JOIN`查询。两张表，每张有 25 万条记录，将生成 625 亿行的结果集，这会给即便是最强大的服务器也带来巨大压力。一个更实际的用途是生成数据来创建一个清单，比如为商店中的几种衬衫款式提供的所有颜色。

## 使用`NULL`查找缺失值的行

每当你连接表时，明智的做法是检查一个表中的关键值是否出现在另一个表中，以及是否有任何缺失的值。差异发生的原因有很多种。有些数据可能随着时间变化。例如，一个新产品的表格可能包含在旧产品表格中不存在的代码。或者可能存在问题，如文书错误或数据库输出不完整。这些都是进行正确数据推断时的重要背景信息。

当你只有少量行时，通过眼睛观察数据是一种简单的方式来查找缺失数据的行，正如我们在前面的连接示例中所做的那样。对于大型表格，你需要更好的策略：筛选出所有没有匹配的行。为此，我们使用关键字`NULL`。

在 SQL 中，`NULL`是一个特殊值，表示没有数据或数据因为没有包含而未知。例如，如果一个人填写地址表单时跳过了“中间名首字母”字段，我们就不会在数据库中存储空字符串，而是使用`NULL`来表示未知值。需要记住的是，`NULL`与`0`或空字符串（你可以使用两个引号（`''`）表示）是不同的。这两者可能有一些意图不明确的含义，容易被误解，因此我们使用`NULL`来表示值是未知的。与`0`或空字符串不同，你可以在不同的数据类型中使用`NULL`。

当 SQL 连接返回其中一个表的空行时，这些列不会返回为空，而是返回值`NULL`。在列表 7-10 中，我们将通过添加`WHERE`子句，使用`IS NULL`来筛选`district_2035`表的`id`列，以找到那些行。如果我们想查看有数据的列，我们会使用`IS NOT NULL`。

```
SELECT *
FROM district_2020 LEFT JOIN district_2035
ON district_2020.id = district_2035.id
WHERE district_2035.id IS NULL;
```

列表 7-10：使用`IS NULL`筛选出缺失值

现在，连接的结果仅显示左侧表中没有在右侧表中匹配的那一行。这通常被称为*反连接（anti-join）*。

```
id     school_2020       id       school_2035 
-- ------------------- ------ ---------------------
 5 Dover Middle School
```

很容易反转输出，以查看右侧表中与左侧表没有匹配的行。你只需将查询改为使用`RIGHT JOIN`，并修改`WHERE`子句，筛选出`district_2020.id IS NULL`。

## 理解三种表关系类型

连接表的一部分科学（或者说艺术，一些人可能会这么说）涉及到理解数据库设计者如何设计表之间的关系，这也被称为数据库的*关系模型*。表关系有三种类型：一对一、一对多和多对多。

### 一对一关系

在我们例子 7-4 中的`JOIN`操作里，两个表中的`id`值没有重复：`district_2020`表中只有一行`id`为`1`，而`district_2035`表中只有一行`id`为`1`。这意味着在任意表中的`id`在另一个表中最多只能找到一个匹配项。在数据库术语中，这称为*一对一*关系。考虑另一个例子：连接两个包含州级人口普查数据的表。一个表可能包含家庭收入数据，另一个表则是关于教育水平的数据。两个表都会有 51 行（每个州加上华盛顿特区），如果我们根据州名、州缩写或标准地理代码来连接它们，那么每个表中的每个键值只会有一个匹配项。

### 一对多关系

在*一对多*关系中，一个表中的关键值将在另一个表的连接列中有多个匹配值。考虑一个追踪汽车的数据库。一个表会包含制造商数据，每个制造商（例如福特、本田、特斯拉等）占一行。另一个表则包含模型名称，例如野马、思域、Model 3 和雅阁，每个制造商表中的行会与这些行匹配。

### 多对多关系

*多对多*关系出现在一个表中的多个项可以与另一个表中的多个项相关联，反之亦然。例如，在一个棒球联赛中，每个球员可以被分配到多个位置，而每个位置也可以由多个球员担任。由于这种复杂性，多对多关系通常会涉及一个第三个中间表。在棒球联赛的例子中，数据库可能会有一个`players`表、一个`positions`表，以及一个第三个表`players_positions`，它有两个列来支持多对多关系：来自`players`表的`id`和来自`positions`表的`id`。

理解这些关系至关重要，因为它帮助我们判断查询结果是否准确反映了数据库的结构。

## 在连接中选择特定列

到目前为止，我们使用了星号通配符来选择两个表中的所有列。这对于快速检查数据是可以的，但更常见的情况是你需要指定一个列的子集。你可以专注于你想要的数据，并避免在有人向表中添加新列时不小心改变查询结果。

正如你在单表查询中所学到的，选择特定列时，需要使用`SELECT`关键字，后面跟上所需的列名。在连接多个表时，最佳实践是将表名与列名一起写明。原因是，多个表可能包含相同名称的列，这在我们当前的连接表中是完全正确的。

考虑以下查询，它尝试在没有指定表名的情况下获取`id`列：

```
SELECT id 
FROM district_2020 LEFT JOIN district_2035
ON district_2020.id = district_2035.id;
```

由于`id`在`district_2020`和`district_2035`中都存在，服务器会抛出一个错误，在 pgAdmin 的结果面板中显示：`column reference "id" is ambiguous`（列引用`id`不明确）。目前无法判断`id`属于哪个表。

为了修正错误，我们需要在每个查询的列前面加上表名，正如我们在`ON`子句中所做的那样。示例 7-11 展示了语法，指定我们想要从`district_2020`中获取`id`列。我们还从两个表中获取学校名称。

```
SELECT district_2020.id,
       district_2020.school_2020,
       district_2035.school_2035
FROM district_2020 LEFT JOIN district_2035
ON district_2020.id = district_2035.id
ORDER BY district_2020.id;
```

示例 7-11：在连接查询中特定列的查询

我们只需要在每个列名前加上它所在的表名，其余查询语法保持不变。结果会返回来自每个表的请求列：

```
id      school_2020           school_2035      
-- --------------------- ----------------------
 1 Oak Street School     Oak Street School
 2 Roosevelt High School Roosevelt High School
 5 Dover Middle School   
 6 Webutuck High School  Webutuck High School
```

我们还可以添加之前在普查数据中使用过的`AS`关键字，使结果中明确标识`id`列来自`district_2020`。语法如下所示：

```
SELECT district_2020.id AS d20_id, ...
```

这样，`district_2020 id`列的名称将以`d20_id`的形式显示在结果中。

## 使用表别名简化 JOIN 语法

为列指定表名其实不难，但对于多个列重复书写冗长的表名会让你的代码显得杂乱。为同事写可读性强的代码是最好的方式，而这通常不应该让他们在超过 25 列的代码中重复查找表名！编写更简洁代码的一种方法是使用一种叫做*表别名*的简写方式。

创建表别名时，我们在`FROM`子句中声明表时，表名后加一个或两个字符作为别名。（你可以为别名使用多个字符，但如果目标是简化代码，就不要过多使用。）这些字符将作为别名，我们可以在代码中任何引用表的位置使用它。示例 7-12 展示了这种方式的使用。

```
SELECT d20.id,
       d20.school_2020,
       d35.school_2035
1 FROM district_2020 AS d20 LEFT JOIN district_2035 AS d35
ON d20.id = d35.id
ORDER BY d20.id;
```

示例 7-12：使用表别名简化代码

在`FROM`子句中，我们使用`AS`关键字声明了`district_2020`的别名`d20`，以及`district_2035`的别名`d35`。这两个别名比表名更短，但依然有意义。一旦这样做，我们就可以在代码的其他地方使用别名代替完整的表名。我们的 SQL 立刻变得更简洁，这是理想的做法。请注意，`AS`关键字在这里是可选的；你可以在声明表名和列名的别名时省略它。

## 连接多个表

当然，SQL 连接不仅限于两个表。只要我们有匹配的列值可以进行连接，我们就可以继续将表添加到查询中。假设我们获得了另外两个与学校相关的表，并且想要在三表连接中将它们与 `district_2020` 连接。`district_2020_enrollment` 表包含每个学校的学生人数：

```
id    enrollment
--    ----------
 1           360
 2          1001
 5           450
 6           927
```

`district_2020_grades` 表包含每个楼宇中所在的年级：

```
id    grades
--    ------
 1    K-3
 2    9-12
 5    6-8
 6    9-12
```

为了编写查询，我们将使用列表 7-13 来创建表，加载数据，并运行查询将它们连接到 `district_2020`。

```
CREATE TABLE district_2020_enrollment (
    id integer,
    enrollment integer
);

CREATE TABLE district_2020_grades (
    id integer,
    grades varchar(10)
);

INSERT INTO district_2020_enrollment
VALUES
    (1, 360),
    (2, 1001),
    (5, 450),
    (6, 927);

INSERT INTO district_2020_grades
VALUES
    (1, 'K-3'),
    (2, '9-12'),
    (5, '6-8'),
    (6, '9-12');

SELECT d20.id,
       d20.school_2020,
       en.enrollment,
       gr.grades
1 FROM district_2020 AS d20 JOIN district_2020_enrollment AS en
    ON d20.id = en.id
2 JOIN district_2020_grades AS gr
    ON d20.id = gr.id
ORDER BY d20.id;
```

列表 7-13：连接多个表

在执行 `CREATE TABLE` 和 `INSERT` 部分的脚本后，我们得到了新的 `district_2020_enrollment` 和 `district_2020_grades` 表，每个表都包含与本章前面提到的 `district_2020` 相关的记录。然后，我们将所有三个表连接起来。

在 `SELECT` 查询中，我们通过表的 `id` 列将 `district_2020` 与 `district_2020_enrollment` 连接 1。我们还声明了表别名，以保持代码简洁。接下来，查询将 `district_2020` 与 `district_2020_grades` 连接，再次使用 `id` 列 2。

我们的结果现在包括来自所有三个表的列：

```
id      school_2020      enrollment grades 
-- --------------------- ---------- ------
 1 Oak Street School            360 K-3
 2 Roosevelt High School       1001 9-12
 5 Dover Middle School          450 6-8
 6 Webutuck High School         927 9-12
```

如果需要，你还可以通过额外的连接将更多表添加到查询中。你也可以根据表之间的关系，在不同的列上进行连接。虽然 SQL 中并没有硬性限制一个查询中可以连接的表数量，但某些数据库系统可能会设置限制。请查阅相关文档。

## 使用集合运算符合并查询结果

某些情况下，我们需要重新排序数据，以使得来自不同表的列不是像连接那样并排返回，而是聚集在一个结果中。示例包括基于 JavaScript 的数据可视化或使用 R 和 Python 编程语言库进行分析的输入格式要求。实现这种数据操作的一种方法是使用 ANSI 标准 SQL *集合运算符* `UNION`、`INTERSECT` 和 `EXCEPT`。集合运算符将多个 `SELECT` 查询的结果合并。下面是每个运算符的简要介绍：

1.  `UNION` 给定两个查询，它将第二个查询的结果行附加到第一个查询返回的行，并删除重复项，生成一个包含唯一行的组合集合。将语法修改为 `UNION ALL` 会返回所有行，包括重复项。

1.  `INTERSECT` 只返回同时存在于两个查询结果中的行，并删除重复项。

1.  `EXCEPT` 返回仅存在于第一个查询结果中，但不在第二个查询结果中的行。重复项会被移除。

对于这些操作，两个查询必须生成相同数量的列，并且来自两个查询的结果列必须具有兼容的数据类型。让我们继续使用学校区表，简要展示它们的工作原理。

### UNION 和 UNION ALL

在列表 7-14 中，我们使用 `UNION` 来组合检索 `district_2020` 和 `district_2035` 中所有行的查询。

```
SELECT * FROM district_2020
1 UNION
SELECT * FROM district_2035
2 ORDER BY id;
```

示例 7-14：使用 `UNION` 合并查询结果

该查询由两个完整的`SELECT`语句组成，中间用`UNION`关键字 1 连接。`ORDER BY` 2 位于`id`列上，发生在集合操作之后，因此不能作为每个`SELECT`的一部分列出。从我们已经处理的数据来看，你知道这些查询将返回两个表中完全相同的几行。但是通过将查询合并为`UNION`，我们的结果消除了重复项：

```
id      school_2020      
-- ---------------------
 1 Oak Street School
 2 Roosevelt High School
 3 Morrison Elementary
 4 Chase Magnet Academy
 5 Dover Middle School
 6 Webutuck High School
```

注意，学校的名称在`school_2020`列中，这是第一个查询结果的一部分。第二个查询中来自`district_2035`表的`school_2035`列的学校名称被简单地附加到第一个查询的结果中。因此，第二个查询中的列必须与第一个查询中的列匹配，并且具有兼容的数据类型。

如果我们希望结果包含重复行，我们可以在查询中将`UNION`替换为`UNION ALL`，就像在示例 7-15 中一样。

```
SELECT * FROM district_2020
UNION ALL
SELECT * FROM district_2035
ORDER BY id;
```

示例 7-15：使用 `UNION ALL` 合并查询结果

这将产生所有行，包含重复项：

```
id      school_2020      
-- ---------------------
 1 Oak Street School
 1 Oak Street School
 2 Roosevelt High School
 2 Roosevelt High School
 3 Morrison Elementary
 4 Chase Magnet Academy
 5 Dover Middle School
 6 Webutuck High School
 6 Webutuck High School
```

最后，定制合并结果通常是有帮助的。例如，你可能想知道每一行的表值来自哪里，或者你可能想包含或排除某些列。示例 7-16 展示了使用`UNION ALL`的一个例子。

```
1 SELECT '2020' AS year,
     2 school_2020 AS school
FROM district_2020

UNION ALL

SELECT '2035' AS year,
       school_2035
FROM district_2035
ORDER BY school, year;
```

示例 7-16：自定义 `UNION` 查询

在第一个查询的`SELECT`语句 1 中，我们将字符串`2020`指定为填充名为`year`的列的值。我们在第二个查询中也使用`2035`作为字符串。这个方法与第五章“导入时向列添加值”部分中使用的技术类似。然后，我们将`school_2020`列 2 重命名为`school`，因为它将显示来自两个年份的学校。

执行查询以查看结果：

```
year        school         
---- --------------------
2035 Chase Magnet Academy
2020 Dover Middle School
2035 Morrison Elementary
2020 Oak Street School
2035 Oak Street School
2020 Roosevelt High School
2035 Roosevelt High School
2020 Webutuck High School
2035 Webutuck High School
```

现在我们的查询为每所学校生成一个年份标识，例如，我们可以看到 Dover 中学的那一行来自查询`district_2020`表的结果。

### INTERSECT 和 EXCEPT

现在你已经知道如何使用`UNION`，你可以将相同的概念应用到`INTERSECT`和`EXCEPT`。示例 7-17 展示了两者，你可以分别运行它们以查看结果的差异。

```
SELECT * FROM district_2020
1 INTERSECT
SELECT * FROM district_2035
ORDER BY id;

SELECT * FROM district_2020
2 EXCEPT
SELECT * FROM district_2035
ORDER BY id;
```

示例 7-17：使用 `INTERSECT` 和 `EXCEPT` 合并查询结果

使用`INTERSECT` 1 的查询仅返回在两个查询结果中都存在的行，并消除重复项：

```
id  school_2020      
-- --------------
 1 Oak Street School
 2 Roosevelt High School
 6 Webutuck High School
```

使用`EXCEPT` 2 的查询返回在第一个查询中存在但在第二个查询中不存在的行，并消除可能存在的重复项：

```
id     school_2020     
-- -------------------
 5 Dover Middle School
```

与`UNION`一起，使用`INTERSECT`和`EXCEPT`的查询为你提供了充足的能力来安排和检查数据。

最后，让我们简要回到连接，看看如何对不同表中的数字进行计算。

## 对连接表列执行数学运算

我们在第六章探讨的数学函数在处理连接后的表格时同样适用。当在操作中引用某个列时，我们需要包括表格名称，就像在选择表格列时一样。如果你处理的是任何定期发布的新数据，你会发现这个概念对于将新发布的表格与旧表格连接并探索数值变化非常有用。

这正是我和许多记者每次发布新一轮人口普查数据时所做的事情。我们会加载新数据，并试图找出人口、收入、教育和其他指标的增长或下降模式。让我们通过重新访问我们在第五章创建的`us_counties_pop_est_2019`表，并加载显示 2010 年县级人口估算数据的新表来看看如何操作。为了创建表格，导入数据，并将其与 2019 年估算数据进行连接，请运行 Listing 7-18 中的代码。

```
1 CREATE TABLE us_counties_pop_est_2010 (
    state_fips text, 
    county_fips text,
    region smallint,
    state_name text,
    county_name text,
    estimates_base_2010 integer,
    CONSTRAINT counties_2010_key PRIMARY KEY (state_fips, county_fips)
);

2 COPY us_counties_pop_est_2010
FROM '*C:\YourDirectory\*us_counties_pop_est_2010.csv'
WITH (FORMAT CSV, HEADER);

3 SELECT c2019.county_name,
       c2019.state_name,
       c2019.pop_est_2019 AS pop_2019,
       c2010.estimates_base_2010 AS pop_2010,
       c2019.pop_est_2019 - c2010.estimates_base_2010 AS raw_change,
     4 round( (c2019.pop_est_2019::numeric - c2010.estimates_base_2010) 
           / c2010.estimates_base_2010 * 100, 1 ) AS pct_change
FROM us_counties_pop_est_2019 AS c2019
    JOIN us_counties_pop_est_2010 AS c2010
5 ON c2019.state_fips = c2010.state_fips
    AND c2019.county_fips = c2010.county_fips
6 ORDER BY pct_change DESC;
```

Listing 7-18: 在连接的人口普查表上进行数学运算

在这段代码中，我们在前面的基础上进行构建。我们有熟悉的`CREATE TABLE`语句 1，针对本次练习，它包含州、县和区域代码，并且有列显示州和县的名称。它还包括一个`estimates_base_2010`列，其中包含美国人口普查局为每个县估算的 2010 年人口（美国人口普查局将其每 10 年一次的完整人口普查数据进行修改，以创建一个基准数字，用于与后续年度的估算数据进行比较）。`COPY`语句 2 导入一个包含人口普查数据的 CSV 文件；你可以在[`nostarch.com/practical-sql-2nd-edition/`](https://nostarch.com/practical-sql-2nd-edition/)找到*us_counties_pop_est_2010.csv*以及本书的所有资源。下载文件后，你需要更改文件路径，以指向你保存该文件的位置。

完成导入后，你应该会得到一个名为`us_counties_pop_est_2010`的表格，包含 3,142 行数据。现在我们有了 2010 年和 2019 年的人口估算表格，计算每个县在这两年之间的人口百分比变化就显得很有意义了。哪些县在增长方面领先全国？哪些县则出现了人口下降？

我们将使用第六章中使用的百分比变化公式来得出答案。`SELECT`语句 3 包括来自 2019 年表格的县名和州名，这些数据被别名为`c2019`。接下来是 2019 年和 2010 年表格中的人口估算列，两个列都使用`AS`重命名，以简化结果中的列名。为了得到人口的原始变化，我们从 2019 年的估算数据中减去 2010 年的基准估算值，而要计算百分比变化，我们使用公式 4，并将结果四舍五入到小数点后一位。

我们通过匹配两个表中两个列的值来连接：`state_fips` 和 `county_fips` 5。之所以使用两个列而不是一个列进行连接，是因为在这两个表中，州代码和县代码的组合代表了一个唯一的县。我们使用 `AND` 逻辑运算符将这两个条件组合起来。使用该语法时，只有当两个条件都满足时，行才会被连接。最后，我们按百分比变化 6 降序排列结果，这样我们就可以看到增长最快的地区排在前面。

这需要很多工作，但这是值得的。以下是结果的前五行所显示的内容：

```
county_name       state_name    pop_2019  pop_2010  raw_change  pct_change 
---------------   ----------    --------  --------  ----------  ----------
McKenzie County   North Dakota     15024      6359        8665       136.3
Loving County     Texas              169        82          87       106.1
Williams County   North Dakota     37589     22399       15190        67.8
Hays County       Texas           230191    157103       73088        46.5
Wasatch County    Utah             34091     23525       10566        44.9
```

两个县，北达科他州的麦肯齐县和德克萨斯州的洛文县，从 2010 年到 2019 年人口增长了两倍以上，其他北达科他州和德克萨斯州的县也显示出显著的增长。这些地方每个都有自己的故事。对于麦肯齐县和北达科他州的其他县，巴肯地质层中的石油和天然气勘探热潮是人口激增的背后原因。这只是我们从这次分析中提取的一个有价值的洞见，也是理解国家人口趋势的起点。

## 总结

由于表关系是数据库架构的基础，学习如何在查询中连接表使你能够处理你将遇到的许多更复杂的数据集。在表之间实验不同类型的连接操作可以让你了解数据是如何收集的，并揭示何时存在质量问题。将尝试不同的连接方式作为你探索新数据集的一项常规任务。

继续前进，我们将继续在这些更大的概念基础上深入挖掘，寻找数据集中的信息，并处理数据类型的细节，确保数据的质量。但首先，我们将看看另一个基础元素：采用最佳实践使用 SQL 构建可靠、高效的数据库。
