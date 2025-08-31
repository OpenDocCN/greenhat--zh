# 第十七章：通过视图、函数和触发器节省时间

![](img/chapterart.png)

使用编程语言的一个优势是我们可以自动化重复的、枯燥的任务。这就是本章的内容：将你可能一遍又一遍执行的查询或步骤转化为可重用的数据库对象，你只需编写一次代码，之后可以调用它们让数据库完成工作。程序员称之为 DRY 原则：不要重复自己。

你将首先学习如何将查询存储为可重用的数据库*视图*。接下来，你将探索如何创建数据库函数，像使用`round()`和`upper()`这样的内置函数一样操作数据。然后，你将设置*触发器*，当表上发生特定事件时，自动运行这些函数。所有这些技巧不仅有助于减少重复工作，还能确保数据完整性。

我们将在前面章节中的示例上练习这些技巧。本章的所有代码都可以通过[`nostarch.com/practical-sql-2nd-edition/`](https://nostarch.com/practical-sql-2nd-edition/)与本书的资源一起下载。

## 使用视图简化查询

*视图*本质上是一个存储的查询，带有一个名称，你可以像操作表一样使用它。例如，一个视图可能存储一个计算每个州总人口的查询。与表一样，你可以查询这个视图，将视图与表（或其他视图）连接，并使用视图更新或插入数据到它所依赖的表，尽管有一些限制。视图中的存储查询可以很简单，只引用一个表，或者很复杂，涉及多个表的连接。

视图在以下情况下尤其有用：

+   **避免重复劳动：** 它们让你能够只写一次复杂的查询，并在需要时访问结果。

+   **减少杂乱：** 通过只显示与你需求相关的列，它们可以减少你需要浏览的信息量。

+   **提供安全性：** 视图可以限制对表中某些列的访问。

在本节中，我们将介绍两种类型的视图。第一种——标准视图——包含与 ANSI SQL 标准大致一致的 PostgreSQL 语法。每次访问标准视图时，存储的查询都会运行并生成一组临时结果。第二种是*物化视图*，这是 PostgreSQL、Oracle 以及少数其他数据库系统特有的。当你创建物化视图时，它的查询返回的数据会像表一样永久存储在数据库中；如果需要，你可以刷新视图以更新存储的数据。

视图易于创建和维护。让我们通过几个示例来看看它们是如何工作的。

### 创建和查询视图

在本节中，我们将返回到你在第五章导入的人口普查估计表`us_counties_pop_est_2019`。清单 17-1 创建了一个标准视图，仅返回内华达州县的总人口。原始表有 16 列；该视图将只返回其中的 4 列。当我们经常引用数据或在应用程序中使用这些数据时，这样做会方便我们快速访问内华达州的人口普查数据的子集。

```
1 CREATE OR REPLACE VIEW nevada_counties_pop_2019 AS
  2 SELECT county_name,
 state_fips,
           county_fips,
           pop_est_2019
    FROM us_counties_pop_est_2019
    WHERE state_name = 'Nevada';
```

清单 17-1：创建一个显示内华达州 2019 年县数据的视图

我们使用关键字`CREATE OR REPLACE VIEW`1 定义视图，后面是视图的名称`nevada_counties_pop_2019`，然后是`AS`。（我们可以根据需要命名视图；我更倾向于给视图起一个描述性名称。）接下来，我们使用标准 SQL 的`SELECT`2 查询`us_counties_pop_est_2019`表中每个内华达州县的 2019 年人口估计（`pop_est_2019`列）。

注意`CREATE`后面的`OR REPLACE`关键字。这些是可选的，表示如果已经存在同名的视图，则用新定义替换它。如果你在迭代创建视图并希望完善查询时，加入这些关键字会很有帮助。有一个警告：如果你要替换现有视图，新查询 2 必须生成相同的列名，且数据类型和顺序必须与要替换的视图一致。你可以添加列，但它们必须放在列列表的末尾。如果尝试做其他操作，数据库会返回错误消息。

使用 pgAdmin 运行清单 17-1 中的代码。数据库应返回`CREATE VIEW`消息。要找到新视图，在 pgAdmin 的对象浏览器中，右键点击`analysis`数据库并点击**刷新**。选择**Schemas**▶**public**▶**Views**来查看所有视图。当你右键点击新视图并点击**属性**时，应该能在弹出的对话框中的“代码”标签页看到查询的详细版本（表名会添加到每个列名之前）。这是检查数据库中可能存在的视图的一种方便方式。

这种类型的视图——一个非物化的视图——此时不包含任何数据；相反，它包含的`SELECT`查询将在你从另一个查询访问该视图时执行。例如，清单 17-2 中的代码返回视图中的所有列。与典型的`SELECT`查询一样，我们可以使用`ORDER BY`对结果进行排序，这次使用的是县的联邦信息处理标准（FIPS）代码——美国人口普查局和其他联邦机构用来指定每个县和州的标准标识符。我们还添加了一个`LIMIT`子句，只显示五行。

```
SELECT *
FROM nevada_counties_pop_2019
ORDER BY county_fips
LIMIT 5;
```

清单 17-2：查询`nevada_counties_pop_2010`视图

除了五行的限制外，结果应该与运行清单 17-1 中用于创建视图的`SELECT`查询相同：

```
 geo_name     | state_fips | county_fips | pop_2010
------------------+------------+-------------+----------
 Churchill County | 32         | 001         |   24909
 Clark County     | 32         | 003         | 2266715
 Douglas County   | 32         | 005         |   48905
 Elko County      | 32         | 007         |   52778
 Esmeralda County | 32         | 009         |     873
```

这个简单的示例除非你需要频繁列出内华达州的县人口，否则没有什么实际用处。那么，让我们想象一个政治研究组织中的数据分析师可能经常会问的问题：2010 年到 2019 年期间，每个内华达州（或其他州）县的人口百分比变化是多少？

我们在第七章写过一个查询来回答这个问题，虽然创建这个查询并不繁琐，但它确实需要在两列上进行表连接，并使用包含四舍五入和类型转换的百分比变化公式。为了避免重复这项工作，我们可以创建一个视图，将类似于第七章中的查询作为视图存储，如清单 17-3 所示。

```
1 CREATE OR REPLACE VIEW county_pop_change_2019_2010 AS
  2 SELECT c2019.county_name,
           c2019.state_name,
           c2019.state_fips,
           c2019.county_fips,
           c2019.pop_est_2019 AS pop_2019,
           c2010.estimates_base_2010 AS pop_2010,
         3 round( (c2019.pop_est_2019::numeric - c2010.estimates_base_2010)
               / c2010.estimates_base_2010 * 100, 1 ) AS pct_change_2019_2010
  4 FROM us_counties_pop_est_2019 AS c2019
        JOIN us_counties_pop_est_2010 AS c2010
    ON c2019.state_fips = c2010.state_fips
       AND c2019.county_fips = c2010.county_fips;
```

清单 17-3：创建一个显示美国县人口变化的视图

我们从`CREATE OR REPLACE VIEW` 1 开始定义视图，接着是视图的名称和`AS`。`SELECT` 查询 2 从人口普查表中选择列，并包括一个百分比变化计算的列定义 3，这是你在第六章学习过的内容。然后，我们使用州和县的 FIPS 代码连接 2019 年和 2010 年的人口普查表 4。运行代码后，数据库应该再次返回`CREATE VIEW`。

现在我们已经创建了视图，可以使用清单 17-4 中的代码，通过新的视图运行一个简单查询，检索内华达州县的数据。

```
SELECT county_name,
       state_name,
       pop_2019,
     1 pct_change_2019_2010
FROM county_pop_change_2019_2010
2 WHERE state_name = 'Nevada'
ORDER BY county_fips
LIMIT 5;
```

清单 17-4：从`county_pop_change_2019_2010`视图中选择列

在清单 17-2 中，引用我们`nevada_counties_pop_2019`视图的查询中，我们通过在`SELECT`后使用星号通配符，检索了视图中的所有列。清单 17-4 显示了与查询表格一样，我们在查询视图时可以指定具体的列。这里，我们指定了`county_pop_change_2019_2010`视图的七列中的四列。其中一列是`pct_change_2019_2010` 1，它返回我们需要的百分比变化计算结果。如你所见，像这样写列名比写整个公式要简单得多。我们还通过`WHERE`子句 2 对结果进行了过滤，这与我们过滤任何查询的方式类似。

查询视图中的四列后，结果应该是这样的：

```
 county_name     state_name  pop_2019  pct_change_2019_2010
----------------  ----------  --------  --------------------
Churchill County  Nevada         24909                   0.1
Clark County      Nevada       2266715                  16.2
Douglas County    Nevada         48905                   4.1
Elko County       Nevada         52778                   7.8
Esmeralda County  Nevada           873                  11.4
```

现在我们可以根据需要随时重新查看这个视图，以便提取数据进行演示，或回答关于 2010 年到 2019 年间任何美国县人口百分比变化的问题。

仅看这五行数据，你可以看到几个有趣的故事：克拉克县的持续快速增长，这里包括了拉斯维加斯市；以及埃斯梅拉达县的强劲百分比增长，埃斯梅拉达县是美国最小的县之一，还是几个鬼镇的所在地。

### 创建和刷新物化视图

物化视图与标准视图的不同之处在于，在创建时，物化视图的存储查询会被执行，并且它生成的结果会保存在数据库中。实际上，这相当于创建了一个新表。视图保留其存储的查询，因此你可以通过发出刷新视图的命令来更新存储的数据。物化视图的一个好用途是预处理需要较长时间运行的复杂查询，并使这些结果可供更快查询。

让我们删除`nevada_counties_pop_2019`视图，并使用清单 17-5 中的代码重新创建它作为物化视图。

```
1 DROP VIEW nevada_counties_pop_2019;

2 CREATE MATERIALIZED VIEW nevada_counties_pop_2019 AS
    SELECT county_name,
 state_fips,
           county_fips,
           pop_est_2019
    FROM us_counties_pop_est_2019
    WHERE state_name = 'Nevada';
```

清单 17-5：创建物化视图

首先，我们使用`DROP VIEW`语句删除数据库中的`nevada_counties_pop_2019`视图。然后，我们运行`CREATE MATERIALIZED VIEW`语句来创建视图。请注意，语法与创建标准视图相同，只是在添加了`MATERIALIZED`关键字，并且省略了`OR REPLACE`，因为物化视图语法中不支持该选项。运行该语句后，数据库应该会响应`SELECT 17`消息，告诉你视图的查询生成了 17 行数据，将被存储在视图中。现在，我们可以像使用标准视图一样查询这些数据。

假设存储在`us_counties_pop_est_2019`中的人口估算值已经被修订。要更新存储在物化视图中的数据，我们可以使用`REFRESH`关键字，如清单 17-6 所示。

```
REFRESH MATERIALIZED VIEW nevada_counties_pop_2019;
```

清单 17-6：刷新物化视图

执行该语句会重新运行存储在`nevada_counties_pop_2019`视图中的查询；服务器将响应`REFRESH MATERIALIZED VIEW`消息。该视图将反映视图查询引用的任何数据更新。当你有一个需要一定时间来运行的查询时，可以通过将其结果存储在定期刷新的物化视图中节省时间，从而让用户快速访问存储的数据，而不是运行一个冗长的查询。

要删除一个物化视图，我们使用`DROP MATERIALIZED VIEW`语句。另外，请注意，物化视图会出现在 pgAdmin 对象浏览器的不同部分，位于**Schemas**▶**public**▶**Materialized Views**下。

### 使用视图插入、更新和删除数据

对于非物化视图，只要视图满足某些条件，你可以更新或插入被查询的底层表中的数据。一个要求是，视图必须引用单个表或可更新的视图。如果视图的查询连接了多个表，如我们在上一节中构建的人口变化视图，则不能直接对原始表执行插入或更新操作。另外，视图的查询不能包含`DISTINCT`、`WITH`、`GROUP BY`或其他子句。（有关限制的完整列表，请参见[`www.postgresql.org/docs/current/sql-createview.html`](https://www.postgresql.org/docs/current/sql-createview.html)。）

你已经知道如何直接在表中插入和更新数据，那么为什么要通过视图来操作呢？其中一个原因是，视图是控制用户可以更新哪些数据的一种方式。让我们通过一个例子来看看如何操作。

#### 创建员工视图

在第七章的联接课程中，我们创建并填充了`departments`和`employees`表，包含了关于员工和他们工作地点的四行数据（如果你跳过了那部分内容，可以回顾 Listing 7-1）。运行一个快速的`SELECT * FROM employees ORDER BY emp_id;`查询，可以查看表的内容，如下所示：

```
emp_id first_name last_name  salary   dept_id
------ ---------- --------- --------- -------
     1 Julia      Reyes     115300.00       1
     2 Janet      King       98000.00       1
     3 Arthur     Pappas     72700.00       2
     4 Michael    Taylor     89500.00       2
```

假设我们希望通过视图使税务部门的用户（其`dept_id`为`1`）能够添加、删除或更新他们员工的姓名，但不允许他们更改薪资信息或其他部门员工的数据。为此，我们可以使用 Listing 17-7 中的视图定义来实现。

```
CREATE OR REPLACE VIEW employees_tax_dept WITH (security_barrier)1 AS
     SELECT emp_id,
            first_name,
            last_name,
            dept_id
     FROM employees
   2 WHERE dept_id = 1
   3 WITH LOCAL CHECK OPTION;
```

Listing 17-7: 在`employees`表上创建视图

这个视图与我们之前创建的其他视图类似，但有一些额外的功能。首先，在`CREATE OR REPLACE VIEW`语句中，我们添加了关键字`WITH (security_barrier)` 1。这为数据库增加了一层安全性，防止恶意用户绕过视图对行和列的限制。（有关如何防止用户绕过视图安全性的详细信息，请参见[`www.postgresql.org/docs/current/rules-privileges.html`](https://www.postgresql.org/docs/current/rules-privileges.html)。）

在视图的`SELECT`查询中，我们从`employees`表中选择要显示的列，并使用`WHERE`条件过滤出`dept_id = 1`的数据，只列出税务部门的员工。视图本身会限制对符合`WHERE`条件的行进行更新或删除。添加关键字`WITH LOCAL CHECK OPTION` 3 也会限制插入操作，只允许用户添加新的税务部门员工（如果视图定义中没有这些关键字，用户还可以插入`dept_id`为`3`的行）。`LOCAL CHECK OPTION`还会防止用户将员工的`dept_id`更改为`1`以外的值。

通过运行 Listing 17-7 中的代码来创建`employees_tax_dept`视图。然后运行`SELECT * FROM employees_tax_dept ORDER BY emp_id;`，应该会返回以下两行数据：

```
emp_id first_name last_name dept_id
------ ---------- --------- -------
     1 Julia      Reyes           1
     2 Janet      King            1
```

查询结果显示了在税务部门工作的员工；他们是整个`employees`表中四行数据中的两行。

现在，让我们来看一下通过这个视图如何进行插入和更新操作。

#### 使用`employees_tax_dept`视图插入行

我们可以使用视图来插入或更新数据，但在`INSERT`或`UPDATE`语句中，我们不使用表名，而是使用视图名作为替代。在我们通过视图添加或更改数据后，变化会应用到底层表中，这里是`employees`。视图然后通过它所运行的查询来反映这些变化。

列表 17-8 展示了两个通过`employees_tax_dept`视图尝试添加新员工记录的例子。第一个成功，第二个失败。

```
1 INSERT INTO employees_tax_dept (emp_id, first_name, last_name, dept_id)
VALUES (5, 'Suzanne', 'Legere', 1);

2 INSERT INTO employees_tax_dept (emp_id, first_name, last_name, dept_id)
VALUES (6, 'Jamil', 'White', 2);

3 SELECT * FROM employees_tax_dept ORDER BY emp_id;

4 SELECT * FROM employees ORDER BY emp_id;
```

列表 17-8：通过`employees_tax_dept`视图成功和拒绝的插入

在第一个`INSERT` 1 中，我们使用了在第二章中学到的插入语法，提供了 Suzanne Legere 的名字和姓氏，以及她的`emp_id`和`dept_id`。由于新行符合视图中的`LOCAL CHECK`—它包含相同的列且`dept_id`为`1`—因此插入在执行时成功。

但是，当我们运行第二个`INSERT` 2，尝试使用`dept_id`为`2`的 Jamil White 来添加员工时，操作失败，出现错误信息`new row violates check option for view "employees_tax_dept"`。原因是当我们创建视图时，使用了`WHERE`子句来仅返回`dept_id = 1`的行。`dept_id`为`2`的行没有通过`LOCAL CHECK`，因此被阻止插入。

运行`SELECT`语句 3，查看 Suzanne Legere 是否成功添加：

```
emp_id first_name last_name dept_id
------ ---------- --------- -------
     1 Julia      Reyes           1
     2 Janet      King            1
     5 Suzanne    Legere          1
```

我们还查询了`employees`表 4，确实发现 Suzanne Legere 已经被添加到完整的表中。每次访问视图时，它都会查询`employees`表。

```
emp_id first_name last_name  salary   dept_id
------ ---------- --------- --------- -------
     1 Julia      Reyes     115300.00       1
     2 Janet      King       98000.00       1
     3 Arthur     Pappas     72700.00       2
     4 Michael    Taylor     89500.00       2
     5 Suzanne    Legere                    1
```

如你从 Suzanne Legere 的添加中看到的那样，我们通过视图添加的数据也会添加到底层表中。然而，由于视图不包含`salary`列，所以她行中的值为`NULL`。如果你尝试通过该视图插入薪资值，将会收到错误信息`column "salary" of relation "employees_tax_dept" does not exist`。原因是即使`salary`列在底层的`employees`表中存在，它在视图中并未被引用。同样，这是限制对敏感数据访问的一种方式。如果你打算承担数据库管理员的责任，可以查看我在“使用视图简化查询”一节中的笔记中提供的链接，了解更多关于授权用户和添加`WITH (security_barrier)`的内容。

#### 使用`employees_tax_dept`视图更新行

在使用`employees_tax_dept`视图更新数据时，访问底层表数据的相同限制适用。列表 17-9 展示了一个标准查询，通过`UPDATE`更正 Suzanne 姓氏的拼写（作为一个姓氏中有多个大写字母的人，我可以确认这种更正并不罕见）。

```
UPDATE employees_tax_dept
SET last_name = 'Le Gere'
WHERE emp_id = 5;

SELECT * FROM employees_tax_dept ORDER BY emp_id;
```

列表 17-9：通过`employees_tax_dept`视图更新行

运行代码后，`SELECT`查询的结果应显示更新后的姓氏，这会反映在底层的`employees`表中：

```
emp_id first_name last_name dept_id
------ ---------- --------- -------
     1 Julia      Reyes           1
     2 Janet      King            1
     5 Suzanne    Le Gere         1
```

Suzanne 的姓氏现在正确拼写为 Le Gere，而不是 Legere。

然而，如果我们尝试更新一位不在税务部门的员工的姓名，查询会像在示例 17-8 中尝试插入 Jamil White 时一样失败。即使是在税务部门的员工，尝试通过此视图更新薪水也会失败。如果视图没有引用基础表中的某个列，你就无法通过视图访问该列。同样，视图上的更新受到这种限制，提供了保护和隐藏某些数据的方式。

#### 使用 `employees_tax_dept` 视图删除行

现在，让我们探讨如何使用视图删除行。这里也会有对哪些数据可以影响的限制。例如，如果 Suzanne Le Gere 从另一家公司获得了更好的报价并决定离开，你可以通过 `employees_tax_dept` 视图将她从 `employees` 中移除。示例 17-10 显示了标准 `DELETE` 语法中的查询。

```
DELETE FROM employees_tax_dept
WHERE emp_id = 5;
```

示例 17-10：通过 `employees_tax_dept` 视图删除一行

运行查询时，PostgreSQL 应该会返回 `DELETE 1`。但是，当你尝试删除一个不在税务部门的员工所在行时，PostgreSQL 会拒绝操作，并返回 `DELETE 0`。

总结来说，视图不仅可以让你控制数据的访问，还能为你提供处理数据的快捷方式。接下来，让我们探讨如何使用函数来节省击键次数和时间。

## 创建你自己的函数和过程

在本书中，你已经使用过函数，例如使用 `upper()` 转换字母为大写，或使用 `sum()` 计算总和。这些函数背后有大量（有时复杂的）编程代码，它执行一系列操作，可能根据函数的功能返回响应。我们在这里避免使用复杂的代码，但会构建一些基本函数，作为你自己想法的跳板。即使是简单的函数也能帮助你避免重复代码。

本节中的大部分语法是 PostgreSQL 特有的，它支持用户自定义的函数和*过程*（两者之间的区别微妙，我会给出两者的例子）。你可以使用普通 SQL 来定义函数和过程，但你也可以选择其他选项。一个选项是 PostgreSQL 特有的*过程语言* PL/pgSQL，它增加了一些标准 SQL 中没有的特性，例如逻辑控制结构（`IF ... THEN ... ELSE`）。其他选项包括 PL/Python 和 PL/R，分别用于 Python 和 R 编程语言。

请注意，主要的数据库系统（包括 Microsoft SQL Server、Oracle 和 MySQL）都实现了自己变种的函数和过程。如果你使用的是其他数据库管理系统，本节将帮助你理解与函数相关的概念，但你需要查看数据库文档以了解其对函数的具体实现。

### 创建 `percent_change()` 函数

函数处理数据并返回一个值。作为例子，让我们编写一个函数，简化数据分析中的常见任务：计算两个值之间的百分比变化。在第六章中，你学习了我们如何表达百分比变化公式：

```
percent change = (New Number – Old Number) / Old Number
```

我们不必每次都写这个公式，可以创建一个名为 `percent_change()` 的函数，接受新的和旧的数字作为输入，并返回结果，四舍五入到用户指定的小数位数。让我们通过 Listing 17-11 中的代码来了解如何声明一个使用 SQL 的简单函数。

```
1 CREATE OR REPLACE FUNCTION
2 percent_change(new_value numeric,
               old_value numeric,
               decimal_places integer 3DEFAULT 1)
4 RETURNS numeric AS
5 'SELECT round(
       ((new_value - old_value) / old_value) * 100, decimal_places
);'
6 LANGUAGE SQL
7 IMMUTABLE
8 RETURNS NULL ON NULL INPUT;
```

Listing 17-11: 创建 `percent_change()` 函数

这段代码发生了很多事情，但它并不像看起来那么复杂。我们从命令 `CREATE OR REPLACE FUNCTION` 1 开始。与创建视图的语法一样，`OR REPLACE` 关键字是可选的。接着，我们给出函数的名称 2，并在括号中列出确定函数输入的 *参数*。每个参数都作为函数的输入，具有名称和数据类型。例如，`new_value` 和 `old_value` 是 `numeric` 类型，要求函数用户提供匹配该类型的输入值，而 `decimal_places`（指定四舍五入结果的小数位数）是 `integer` 类型。对于 `decimal_places`，我们指定 `1` 作为 `DEFAULT` 3 值——这使得该参数是可选的，如果用户省略了该参数，默认值将设置为 `1`。

然后，我们使用关键字 `RETURNS numeric AS` 4 来告诉函数将其计算结果作为 `numeric` 类型返回。如果这是一个用于连接字符串的函数，我们可能会返回 `text` 类型。

接下来，我们编写函数的核心部分，执行计算。在单引号内，我们放入一个 `SELECT` 查询 5，其中包含嵌套在 `round()` 函数中的百分比变化计算。在公式中，我们使用函数的参数名而不是数字。

然后，我们提供一系列定义函数属性和行为的关键字。`LANGUAGE` 6 关键字指定我们使用的是普通 SQL，而不是 PostgreSQL 支持的其他编程语言来创建函数。接下来，`IMMUTABLE` 关键字 7 表示该函数不能修改数据库，并且对于给定的一组参数，它始终返回相同的结果。`RETURNS NULL ON NULL INPUT` 8 这一行保证了如果任何默认未提供的输入为 `NULL`，函数将返回 `NULL`。

使用 pgAdmin 运行代码以创建 `percent_change()` 函数。服务器应返回 `CREATE FUNCTION` 消息。

### 使用 `percent_change()` 函数

为了测试新的 `percent_change()` 函数，可以像 Listing 17-12 中所示，单独运行它，使用 `SELECT`。

```
SELECT percent_change(110, 108, 2);
```

Listing 17-12: 测试 `percent_change()` 函数

这个例子使用 `110` 作为新值，`108` 作为旧值，`2` 作为四舍五入结果的小数位数。

运行代码；结果应如下所示：

```
 percent_change
----------------
           1.85
```

结果告诉我们，108 和 110 之间的百分比增加为 1.85%。你可以尝试使用其他数字，看看结果如何变化。还可以尝试将`decimal_places`参数更改为包括`0`在内的值，或者省略它，看看这如何影响输出。你应该看到结果的小数点后面有更多或更少的数字，具体取决于你的输入。

我们创建了这个函数，以避免在查询中编写完整的百分比变化公式。让我们使用它来计算百分比变化，使用我们在第七章编写的普查估计人口变化查询的版本，如清单 17-13 所示。

```
SELECT c2019.county_name,
       c2019.state_name,
       c2019.pop_est_2019 AS pop_2019,
 1 percent_change(c2019.pop_est_2019,
                      c2010.estimates_base_2010) AS pct_chg_func,
     2 round( (c2019.pop_est_2019::numeric - c2010.estimates_base_2010)
           / c2010.estimates_base_2010 * 100, 1 ) AS pct_change_formula
FROM us_counties_pop_est_2019 AS c2019
    JOIN us_counties_pop_est_2010 AS c2010
ON c2019.state_fips = c2010.state_fips
   AND c2019.county_fips = c2010.county_fips
ORDER BY pct_chg_func DESC
LIMIT 5;
```

清单 17-13：在普查数据上测试`percent_change()`函数

清单 17-13 修改了第七章中的原始查询，在`SELECT`中添加了`percent_change()`函数 1 作为一列。我们还包括了明确的百分比变化公式 2，以便我们可以比较结果。作为输入，我们使用 2019 年人口估算列（`c2019.pop_est_2019`）作为新数字，使用 2010 年估算基数作为旧数字（`c2010.estimates_base_2010`）。

查询结果应显示人口变化百分比最大的五个县，并且函数的结果应与直接输入查询中的公式结果相匹配。请注意，`pct_chg_func`列中的每个值都有一个小数位，这是函数的默认值，因为我们没有提供可选的第三个参数。以下是同时使用函数和公式的结果：

```
 county_name    state_name  pop_2019 pct_chg_func pct_chg_formula
--------------- ------------ -------- ------------ ---------------
McKenzie County North Dakota    15024        136.3           136.3
Loving County   Texas             169        106.1           106.1
Williams County North Dakota    37589         67.8            67.8
Hays County     Texas          230191         46.5            46.5
Wasatch County  Utah            34091         44.9            44.9
```

现在我们知道函数按预期工作，我们可以在任何需要解决该计算时使用`percent_change()`——这比编写公式要快得多！

### 使用过程更新数据

在 PostgreSQL 中实现的*过程*与函数非常相似，尽管有一些显著的不同之处。过程和函数都可以执行不返回值的数据操作，例如更新。然而，过程没有返回值的子句，而函数有。此外，过程可以包含我们在第十章中讲解的事务命令，例如`COMMIT`和`ROLLBACK`，而函数不能。许多数据库管理系统实现了过程，通常称为*存储过程*。PostgreSQL 从版本 11 开始添加了过程，它们是 SQL 标准的一部分，但 PostgreSQL 的语法并不完全兼容。

我们可以使用过程简化常规的数据更新。在本节中，我们将编写一个过程，根据教师的雇佣日期以来的时间，更新教师的个人休假天数（除了假期天数）。

在本练习中，我们将返回到第二章第一节的`teachers`表。如果你跳过了该章节中的“创建表格”部分，现在使用清单 2-2 和 2-3 中的示例代码创建`teachers`表并插入数据。

让我们向`teachers`表添加一列，用于存储教师的个人假期，使用列表 17-14 中的代码。新列在我们稍后使用过程填充之前将为空。

```
ALTER TABLE teachers ADD COLUMN personal_days integer;

SELECT first_name,
       last_name,
       hire_date,
       personal_days
FROM teachers;
```

列表 17-14：向`teachers`表添加一列并查看数据

列表 17-14 使用`ALTER`更新教师表，并通过`ADD COLUMN`关键字添加`personal_days`列。然后，我们运行`SELECT`语句查看数据，同时也包括每个教师的姓名和聘用日期。当两个查询完成后，你应该看到以下六行：

```
first_name last_name hire_date  personal_days
---------- --------- ---------- -------------
Janet      Smith     2011-10-30
Lee        Reynolds  1993-05-22
Samuel     Cole      2005-08-01
Samantha   Bush      2011-10-30
Betty      Diaz      2005-08-30
Kathleen   Roush     2010-10-22
```

`personal_days`列目前只包含`NULL`值，因为我们尚未插入任何内容。

现在，让我们创建一个名为`update_personal_days()`的过程，该过程将根据获得的个人假期（除了假期天数）填充`personal_days`列。我们将使用以下标准：

+   自聘用之日起不到 10 年：3 天个人假期

+   自聘用之日起 10 年至 15 年：4 天个人假期

+   自聘用之日起 15 年至 20 年：5 天个人假期

+   自聘用之日起 20 年至 25 年：6 天个人假期

+   自聘用之日起 25 年或以上：7 天个人假期

列表 17-15 中的代码创建了一个过程。这一次，我们不仅使用纯 SQL，还结合了 PL/pgSQL 过程语言的元素，这是 PostgreSQL 支持的一种额外语言，用于编写函数。让我们来看看一些不同之处。

```
CREATE OR REPLACE PROCEDURE update_personal_days()
AS 1$$
2 BEGIN
    UPDATE teachers
    SET personal_days =
      3 CASE WHEN (now() - hire_date) >= '10 years'::interval
                  AND (now() - hire_date) < '15 years'::interval THEN 4
             WHEN (now() - hire_date) >= '15 years'::interval
                  AND (now() - hire_date) < '20 years'::interval THEN 5
             WHEN (now() - hire_date) >= '20 years'::interval
                  AND (now() - hire_date) < '25 years'::interval THEN 6
             WHEN (now() - hire_date) >= '25 years'::interval THEN 7
             ELSE 3
        END;
  4 RAISE NOTICE 'personal_days updated!';
END;
5 $$
6 LANGUAGE plpgsql;
```

列表 17-15：创建`update_personal_days()`函数

我们以`CREATE OR REPLACE PROCEDURE`开始，并为过程指定一个名称。这一次，我们没有提供参数，因为不需要用户输入——该过程操作的是预定的列，并具有用于计算间隔的固定值。

在编写基于 PL/pgSQL 的函数时，PostgreSQL 的约定通常是使用非 ANSI SQL 标准的美元引号（`$$`）来标记包含所有函数命令的字符串的开始和结束（就像之前的`percent_change()` SQL 函数一样，你可以使用单引号来包围字符串，但那样字符串中的任何单引号就需要加倍，这不仅看起来凌乱，还可能引起混淆）。所以，`$$`之间的所有内容就是执行工作的代码。你也可以在美元符号之间添加一些文本，比如`$namestring$`，以创建一对独特的起始和结束引号。这在某些情况下很有用，比如你需要在函数内部引用一个查询时。

紧接着第一个 `$$` 后，我们开始一个 `BEGIN ... END;` 2 块。这是 PL/pgSQL 的约定，用于标识函数或过程中的代码段的开始和结束；与美元符号引号一样，可以将一个 `BEGIN ... END;` 块嵌套在另一个块内，以便逻辑分组代码。在这个块内，我们放置了一个 `UPDATE` 语句，该语句使用 `CASE` 语句 3 来确定每位教师的个人假期天数。我们通过 `now()` 函数从服务器获取当前日期，并用其减去 `hire_date`。根据 `now() - hire_date` 所在的时间范围，`CASE` 语句返回对应的个人假期天数。我们使用 PL/pgSQL 关键字 `RAISE NOTICE` 4 来显示过程完成的消息。最后，我们使用 `LANGUAGE` 关键字 6 使数据库知道我们编写的代码需要按照 PL/pgSQL 特定的语法进行解释。

运行 示例 17-15 中的代码来创建 `update_personal_days()` 过程。要调用该过程，我们使用 `CALL` 命令，这是 ANSI SQL 标准的一部分：

```
CALL update_personal_days();
```

当过程运行时，服务器会响应并显示它所引发的通知，内容为 `personal_days updated!`。

当您重新运行 示例 17-14 中的 `SELECT` 语句时，您应该会看到 `personal_days` 列的每一行都填充了相应的值。请注意，结果会有所不同，具体取决于您运行此函数的时间，因为使用 `now()` 的计算会随时间变化而变化。

```
first_name last_name hire_date  personal_days
---------- --------- ---------- -------------
Janet      Smith     2011-10-30             3
Lee        Reynolds  1993-05-22             7
Samuel     Cole      2005-08-01             5
Samantha   Bush      2011-10-30             3
Betty      Diaz      2005-08-30             5
Kathleen   Roush     2010-10-22             4
```

您可以在执行某些任务后手动使用 `update_personal_days()` 函数定期更新数据，或者可以使用任务调度程序（如 pgAgent，一个独立的开源工具）自动运行它。您可以在附录的《PostgreSQL 实用工具、工具和扩展》中了解有关 pgAgent 和其他工具的更多信息。

### 在函数中使用 Python 语言

之前，我提到过 PL/pgSQL 是 PostgreSQL 中的默认过程语言，但数据库还支持使用开源语言（如 Python 和 R）创建函数。这种支持使您能够在创建的函数中利用这些语言的特性和模块。例如，使用 Python，您可以使用 `pandas` 库进行分析。有关 PostgreSQL 中包含的语言的详细信息，请参考 [`www.postgresql.org/docs/current/server-programming.html`](https://www.postgresql.org/docs/current/server-programming.html) 上的文档，但在这里我将展示一个使用 Python 的简单函数。

要启用 PL/Python，您必须使用 示例 17-16 中的代码创建扩展。

```
CREATE EXTENSION plpython3u;
```

示例 17-16：启用 PL/Python 过程语言

如果你收到错误信息，如`image not found`，那意味着系统上未安装 PL/Python 扩展。根据操作系统的不同，PL/Python 的安装通常需要安装 Python 并进行一些基本 PostgreSQL 安装之外的额外配置。有关详细信息，请参考第一章中针对你操作系统的安装说明。

启用扩展后，我们可以使用类似你之前尝试过的语法创建一个函数，但在函数体中使用 Python。Listing 17-17 展示了如何使用 PL/Python 创建一个名为`trim_county()`的函数，该函数移除字符串末尾的*County*。我们将使用这个函数来清理人口普查数据中的县名。

```
CREATE OR REPLACE FUNCTION trim_county(input_string text)
1 RETURNS text AS $$
    import re2
  3 cleaned = re.sub(r' County', '', input_string)
    return cleaned
$$
4 LANGUAGE plpython3u;
```

Listing 17-17: 使用 PL/Python 创建`trim_county()`函数

结构看起来应该很熟悉。为函数命名并定义其文本输入后，我们使用`RETURNS`关键字 1 来指定该函数将返回文本。在开头的`$$`符号之后，我们直接写入 Python 代码，从导入 Python 正则表达式模块`re` 2 开始。即使你不太了解 Python，你也许可以推测接下来的两行代码 3 是设置一个变量`cleaned`，它保存了 Python 正则表达式函数`sub()`的结果。该函数在`input_string`中查找一个空格后跟单词*County*的模式，并将其替换为空字符串（由两个撇号表示）。然后该函数返回`cleaned`变量的内容。最后，我们指定`LANGUAGE plpython3u` 4 来标明我们使用 PL/Python 编写函数。

运行代码以创建函数，然后执行 Listing 17-18 中的`SELECT`语句，查看函数执行效果。

```
SELECT county_name,
       trim_county(county_name)
FROM us_counties_pop_est_2019
ORDER BY state_fips, county_fips
LIMIT 5;
```

Listing 17-18: 测试`trim_county()`函数

我们使用`county_name`列作为输入，传递给`us_counties_pop_est_2019`表中的`trim_county()`函数。那应该返回以下结果：

```
 county_name     trim_county
----------------  -------------
 Autauga County   Autauga
 Baldwin County   Baldwin
 Barbour County   Barbour
 Bibb County      Bibb
 Blount County    Blount
```

如你所见，`trim_county()`函数检查了`county_name`列中的每个值，并在存在时删除了空格和单词*County*。虽然这是一个简单的例子，但它展示了如何轻松地在函数中使用 Python（或其他支持的过程语言）。

接下来，你将学习如何使用触发器来自动化数据库操作。

## 使用触发器自动化数据库操作

数据库中的*触发器*在每次发生指定事件（例如`INSERT`、`UPDATE`或`DELETE`）时都会执行一个函数。你可以设置触发器在事件发生前、后，或代替事件触发，并且可以设置它仅对每一行受到事件影响的记录触发一次，或者每次操作只触发一次。例如，假设你从一个表中删除了 20 行数据。你可以设置触发器，在每删除一行时都触发一次，或者只触发一次。

我们将通过两个例子进行演示。第一个例子记录学校成绩的变化日志。第二个例子每次收集温度读数时，自动对温度进行分类。

### 将成绩更新记录到表格中

假设我们想要自动跟踪我们学校数据库中学生 `grades` 表的变化。每次更新一行数据时，我们想记录旧成绩、新成绩以及更改发生的时间（在线搜索 *David Lightman and grades*，你会理解为什么这值得跟踪）。为了自动处理这个任务，我们需要三项内容：

+   一个 `grades_history` 表，用来记录 `grades` 表中成绩的变化

+   一个触发器，每次 `grades` 表发生变化时都会运行一个函数，我们将其命名为 `grades_update`

+   触发器将执行的函数，我们称之为 `record_if_grade_changed()`

#### 创建表格以跟踪成绩和更新

我们从创建所需的表格开始。清单 17-19 包含了首先创建并填充 `grades` 表，然后创建 `grades_history` 表的代码。

```
1 CREATE TABLE grades (
    student_id bigint,
    course_id bigint,
    course text NOT NULL,
    grade text NOT NULL,
PRIMARY KEY (student_id, course_id)
);

2 INSERT INTO grades
VALUES
    (1, 1, 'Biology 2', 'F'),
    (1, 2, 'English 11B', 'D'),
    (1, 3, 'World History 11B', 'C'),
    (1, 4, 'Trig 2', 'B');

3 CREATE TABLE grades_history (
    student_id bigint NOT NULL,
    course_id bigint NOT NULL,
    change_time timestamp with time zone NOT NULL,
    course text NOT NULL,
    old_grade text NOT NULL,
    new_grade text NOT NULL,
PRIMARY KEY (student_id, course_id, change_time)
);
```

清单 17-19：创建 `grades` 和 `grades_history` 表

这些命令很简单。我们使用 `CREATE` 创建一个 `grades` 表 1，并使用 `INSERT` 2 添加四行数据，每一行表示一名学生在某门课程中的成绩。接着，我们使用 `CREATE TABLE` 创建 `grades_history` 表 3，用于记录每次现有成绩被更改时的日志。`grades_history` 表有新成绩、旧成绩和更改时间的列。运行代码以创建这些表格并填充 `grades` 表。在这里我们不往 `grades_history` 表插入任何数据，因为触发器会处理这项任务。

#### 创建函数和触发器

接下来，我们编写触发器将执行的 `record_if_grade_changed()` 函数（注意，PostgreSQL 文档中将此类函数称为 *触发器过程*）。我们必须在触发器中引用该函数之前先编写它。让我们查看 清单 17-20 中的代码。

```
CREATE OR REPLACE FUNCTION record_if_grade_changed()
  1 RETURNS trigger AS
$$
BEGIN
  2 IF NEW.grade <> OLD.grade THEN
    INSERT INTO grades_history (
        student_id,
        course_id,
        change_time,
        course,
        old_grade,
        new_grade)
    VALUES
        (OLD.student_id,
         OLD.course_id,
         now(),
         OLD.course,
       3 OLD.grade,
       4 NEW.grade);
    END IF;
  5 RETURN NULL;
END;
$$ LANGUAGE plpgsql;
```

清单 17-20：创建 `record_if_grade_changed()` 函数

`record_if_grade_changed()` 函数遵循早期示例的模式，但在与触发器配合使用时有一些特定的区别。首先，我们指定 `RETURNS trigger` 1，而不是数据类型。我们使用美元引号来分隔函数的代码部分，并且由于 `record_if_grade_changed()` 是一个 PL/pgSQL 函数，我们还将执行代码放在 `BEGIN ... END;` 块中。接下来，我们使用 `IF ... THEN` 语句 2 启动过程，这是 PL/pgSQL 提供的控制结构之一。我们在这里使用它来仅在更新的成绩与旧成绩不同的情况下运行 `INSERT` 语句，我们通过 `<>` 运算符来进行检查。

当 `grades` 表发生变化时，将执行我们接下来要创建的触发器。对于每一行的更改，触发器将把两个数据集传递到 `record_if_grade_changed()` 中。第一个是更改之前的行值，以 `OLD` 前缀标记。第二个是更改之后的行值，以 `NEW` 前缀标记。函数可以访问原始行值和更新后的行值，用于比较。如果 `IF ... THEN` 语句评估为 `true`，表明旧的和新的 `grade` 值不同，我们使用 `INSERT` 将包含 `OLD.grade` 3 和 `NEW.grade` 4 的行添加到 `grades_history`。最后，我们包含一个带有 `NULL` 值的 `RETURN` 语句 5；触发器过程执行数据库 `INSERT`，因此我们不需要返回值。

运行 列表 17-20 中的代码来创建函数。然后，使用 列表 17-21 将 `grades_update` 触发器添加到 `grades` 表中。

```
1 CREATE TRIGGER grades_update
2 AFTER UPDATE
  ON grades
3 FOR EACH ROW
4 EXECUTE PROCEDURE record_if_grade_changed();
```

列表 17-21：创建`grades_update`触发器

在 PostgreSQL 中，创建触发器的语法遵循 ANSI SQL 标准（尽管文档中并不支持标准的所有方面，详情请见 [`www.postgresql.org/docs/current/sql-createtrigger.html`](https://www.postgresql.org/docs/current/sql-createtrigger.html)）。代码以 `CREATE TRIGGER` 1 语句开头，接着是控制触发器何时运行以及如何运行的子句。我们使用 `AFTER UPDATE` 2 来指定我们希望触发器在 `grades` 行更新后执行。根据需要，我们还可以使用 `BEFORE` 或 `INSTEAD OF` 关键字。

我们使用 `FOR EACH ROW` 3 来告诉触发器在更新表中的每一行时执行该过程。例如，如果某人运行的更新影响了三行，该过程将运行三次。另一种（也是默认的）是 `FOR EACH STATEMENT`，它只运行一次过程。如果我们不关心捕获每一行的更改，只想记录在特定时间内更改了成绩，我们可以使用该选项。最后，我们使用 `EXECUTE PROCEDURE` 4 来指定触发器应运行的函数为 `record_if_grade_changed()`。

在 pgAdmin 中运行 列表 17-21 中的代码来创建触发器。数据库应该会响应 `CREATE TRIGGER` 消息。

#### 测试触发器

现在我们已经创建了触发器和函数，当`grades`表中的数据发生变化时，它应该会运行；让我们看看这个过程是如何运行的。首先，让我们检查我们数据的当前状态。当你运行 `SELECT * FROM grades_history;` 时，你会看到表是空的，因为我们还没有对 `grades` 表进行任何更改，没有需要跟踪的内容。接下来，当你运行 `SELECT * FROM grades ORDER BY student_id, course_id;` 时，你应该会看到你在 列表 17-19 中插入的成绩数据，如下所示：

```
student_id course_id      course       grade
---------- --------- ----------------- -----
         1         1 Biology 2         F
         1         2 English 11B       D
         1         3 World History 11B C
         1         4 Trig 2            B
```

那个生物学 2 等级看起来不怎么好。让我们使用 列表 17-22 中的代码进行更新。

```
UPDATE grades
SET grade = 'C'
WHERE student_id = 1 AND course_id = 1;
```

列表 17-22: 测试 `grades_update` 触发器

当您运行 `UPDATE` 后，pgAdmin 不会显示任何内容来告知您后台执行了触发器。它只会报告 `UPDATE 1`，表示已更新一行。但我们的触发器确实运行了，我们可以通过检查使用此 `SELECT` 查询查看 `grades_history` 表中的列来确认：

```
SELECT student_id,
       change_time,
       course,
       old_grade,
       new_grade
FROM grades_history;
```

当您运行此查询时，您应该看到 `grades_history` 表中包含了一行变更：

```
student_id          change_time           course      old_grade new_grade
---------- ----------------------------- ---------    --------- ---------
         1 2023-09-01 15:50:43.291164-04 Biology 2    F         C
```

此行显示了旧的生物学 2 等级 `F`，新值 `C`，以及 `change_time`，显示了更新的时间（你的结果应反映出你的日期和时间）。请注意，将此行添加到 `grades_history` 是在没有更新者知情的情况下背景进行的。但是表格上的 `UPDATE` 事件触发了触发器执行了 `record_if_grade_changed()` 函数。

如果您曾经使用过内容管理系统，例如 WordPress 或 Drupal，这种修订跟踪可能会很熟悉。它提供了一个有用的记录内容变更的方式，用于参考、审计，以及偶尔的责备。无论如何，自动触发数据库操作的能力让您对数据有了更多的控制。

### 自动分类温度

在第十三章中，我们使用 SQL 的 `CASE` 语句将温度读数重新分类为描述性类别。`CASE` 语句也是 PL/pgSQL 过程化语言的一部分，我们可以利用它的能力为变量赋值，以便每次添加温度读数时自动将这些类别名称存储在表格中。如果我们经常收集温度读数，使用这种技术自动化分类可以避免手动处理任务。

我们将按照记录成绩变化的相同步骤进行操作：首先创建一个函数来分类温度，然后创建一个触发器，在每次更新表格时运行该函数。使用 列表 17-23 来创建一个名为 `temperature_test` 的表格以供练习使用。

```
CREATE TABLE temperature_test (
    station_name text,
    observation_date date,
    max_temp integer,
    min_temp integer,
    max_temp_group text,
PRIMARY KEY (station_name, observation_date)
);
```

列表 17-23: 创建一个名为 `temperature_test` 的表格

`temperature_test` 表包含用于保存站点名称和温度观测日期的列。假设我们有某个过程，每天插入一行数据，提供该位置的最高和最低温度，并且我们需要填写 `max_temp_group` 列以提供天气预报的描述性分类的文本。

为此，我们首先创建一个名为 `classify_max_temp()` 的函数，如 列表 17-24 所示。

```
CREATE OR REPLACE FUNCTION classify_max_temp()
    RETURNS trigger AS
$$
BEGIN
  1 CASE
       WHEN NEW.max_temp >= 90 THEN
           NEW.max_temp_group := 'Hot';
       WHEN NEW.max_temp >= 70 AND NEW.max_temp < 90 THEN
           NEW.max_temp_group := 'Warm';
       WHEN NEW.max_temp >= 50 AND NEW.max_temp < 70 THEN
           NEW.max_temp_group := 'Pleasant';
       WHEN NEW.max_temp >= 33 AND NEW.max_temp < 50 THEN
           NEW.max_temp_group := 'Cold';
       WHEN NEW.max_temp >= 20 AND NEW.max_temp < 33 THEN
           NEW.max_temp_group := 'Frigid';
       WHEN NEW.max_temp < 20 THEN
           NEW.max_temp_group := 'Inhumane';
       ELSE NEW.max_temp_group := 'No reading';
    END CASE;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

列表 17-24: 创建 `classify_max_temp()` 函数

到现在为止，这些函数应该看起来很熟悉。这里的新内容是`CASE`语法的 PL/pgSQL 版本 1，它与 SQL 语法稍有不同。PL/pgSQL 语法在每个`WHEN ... THEN`子句后面都包括一个分号。另一个新内容是*赋值运算符* `:=`，我们用它根据`CASE`函数的结果为`NEW.max_temp_group`列赋予描述性名称。例如，语句`NEW.max_temp_group := 'Cold'`会在温度值大于或等于 33 度但小于 50 度华氏度时，将字符串`'Cold'`赋给`NEW.max_temp_group`。当函数将`NEW`行返回以插入表中时，`NEW.max_temp_group`列会包含字符串值`Cold`。运行代码以创建该函数。

接下来，使用清单 17-25 中的代码，创建一个触发器，每次向`temperature_test`添加一行数据时执行该函数。

```
CREATE TRIGGER temperature_insert
  1 BEFORE INSERT
    ON temperature_test
  2 FOR EACH ROW
  3 EXECUTE PROCEDURE classify_max_temp();
```

清单 17-25：创建`temperature_insert`触发器

在这个例子中，我们在将行插入表之前对`max_temp`进行分类，并为`max_temp_group`创建一个值。这样做比在插入后执行单独的更新操作更高效。为了指定这种行为，我们将`temperature_insert`触发器设置为在`BEFORE INSERT` 1 时触发。

我们还希望触发器以`FOR EACH ROW` 2 触发，因为我们希望表中记录的每个`max_temp`都能获得一个描述性的分类。最终的`EXECUTE PROCEDURE`语句指定了我们刚创建的`classify_max_temp()`函数 3。运行`CREATE TRIGGER`语句在 pgAdmin 中创建触发器，然后使用清单 17-26 测试设置。

```
INSERT INTO temperature_test
VALUES
    ('North Station', '1/19/2023', 10, -3),
    ('North Station', '3/20/2023', 28, 19),
    ('North Station', '5/2/2023', 65, 42),
    ('North Station', '8/9/2023', 93, 74),
    ('North Station', '12/14/2023', NULL, NULL);

SELECT * FROM temperature_test ORDER BY observation_date;
```

清单 17-26：插入行以测试`temperature_insert`触发器

在这里，我们向`temperature_test`插入了五行数据，并且我们期望每行都会触发`temperature_insert`触发器——而它确实做到了！清单中的`SELECT`语句应显示这些结果：

```
station_name  observation_date max_temp min_temp max_temp_group
------------- ---------------- -------- -------- --------------
North Station 2023-01-19             10       -3 Inhumane
North Station 2023-03-20             28       19 Frigid
North Station 2023-05-02             65       42 Pleasant
North Station 2023-08-09             93       74 Hot
North Station 2023-12-14                         No reading
```

感谢触发器和函数的帮助，每个插入的`max_temp`都会自动在`max_temp_group`列中获得适当的分类——即使该值没有读取。请注意，触发器对该列的更新将在插入过程中覆盖任何用户提供的值。

这个温度示例和之前的成绩变更审计示例虽然比较基础，但它们让你初步了解了触发器和函数在简化数据维护方面的强大作用。

## 总结

尽管你在本章中学到的技术开始与数据库管理员的技能重合，但你可以应用这些概念来减少重复某些任务所花费的时间。我希望这些方法能帮助你腾出更多时间，从数据中发现有趣的故事。

本章总结了我们对分析技术和 SQL 语言的讨论。接下来的两章将提供一些工作流程技巧，帮助你提升对 PostgreSQL 的掌握。内容包括如何从计算机的命令行连接数据库并运行查询，以及如何维护你的数据库。
