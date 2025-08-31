# 第三章：使用 SELECT 开始数据探索

![](img/chapterart.png)

对我来说，深入挖掘数据的最好部分不是收集、加载或清洗数据的前提工作，而是当我真正开始 *访谈* 数据时。那些时刻我会发现数据是否干净、是否完整，最重要的是，它能够讲述什么样的故事。可以把访谈数据看作是类似于面试求职者的过程。你想问一些问题，揭示出他们的实际专业水平是否与简历匹配。

访谈数据令人兴奋，因为你会发现真相。例如，你可能会发现一半的受访者忘记填写问卷中的电子邮件字段，或者市长过去五年没有缴纳物业税。或者你可能会发现你的数据是脏的：名字拼写不一致，日期不正确，或者数字与预期不符。这些发现会成为数据故事的一部分。

在 SQL 中，访问数据从 `SELECT` 关键字开始，它从数据库中的一个或多个表中检索行和列。一个 `SELECT` 语句可以很简单，检索一个表中的所有内容，也可以足够复杂，连接数十个表，处理多个计算，并按精确条件进行过滤。

我们将从简单的 `SELECT` 语句开始，然后深入了解 `SELECT` 可以做的更强大操作。

## 基本的 SELECT 语法

这里是一个 `SELECT` 语句，它从一个名为 `my_table` 的表中提取每一行和每一列：

```
SELECT * FROM my_table;
```

这一行代码展示了 SQL 查询的最基本形式。`SELECT` 关键字后面的星号是一个 *通配符*，它像是一个占位符：它不代表特定的任何东西，而是代表那个值可能是的所有内容。在这里，它是“选择所有列”的简写。如果你给出了列名而不是通配符，这个命令会选择该列中的值。`FROM` 关键字表示你希望查询从某个特定的表中返回数据。表名后的分号告诉 PostgreSQL 这是查询语句的结束。

让我们使用这个带有星号通配符的 `SELECT` 语句，针对你在第二章创建的 `teachers` 表。再次打开 pgAdmin，选择 `analysis` 数据库，打开查询工具。然后执行 Listing 3-1 中显示的语句。记住，作为输入这些语句的替代方式，你也可以通过点击 **打开文件** 并导航到保存你从 GitHub 下载的代码的位置来运行代码。如果你看到代码被 `--snip--` 截断了，务必使用这种方法。对于本章，你应该打开 *Chapter_03.sql* 并在点击 **执行/刷新** 图标之前高亮每个语句。

```
SELECT * FROM teachers;
```

Listing 3-1：从 `teachers` 表查询所有行和列

一旦执行查询，查询工具的输出面板中将显示你在第二章中插入到`teachers`表中的所有行和列。行的顺序可能不会总是这样显示，但这没关系。

```
id    first_name    last_name    school                 hire_date     salary
--    ----------    ---------    -------------------    ----------    ------
1     Janet         Smith        F.D. Roosevelt HS      2011-10-30    36200
2     Lee           Reynolds     F.D. Roosevelt HS      1993-05-22    65000
3     Samuel        Cole         Myers Middle School    2005-08-01    43500
4     Samantha      Bush         Myers Middle School    2011-10-30    36200
5     Betty         Diaz         Myers Middle School    2005-08-30    43500
6     Kathleen      Roush        F.D. Roosevelt HS      2010-10-22    38500
```

请注意，`id`列（类型为`bigserial`）会自动填充顺序整数，即使你没有显式插入它们。这非常方便。这个自动递增的整数充当唯一标识符或键，不仅确保表中的每一行都是唯一的，还能让我们稍后将此表与数据库中的其他表连接起来。

在继续之前，注意你有两种其他方式可以查看表中的所有行。使用 pgAdmin，你可以右键点击对象树中的`teachers`表，然后选择**查看/编辑数据**▶**所有行**。或者，你可以使用一种鲜为人知的标准 SQL 方法：

```
TABLE teachers;
```

这两者提供的结果与清单 3-1 中的代码相同。现在，让我们优化这个查询，使其更具针对性。

### 查询列的子集

通常，限制查询返回的列是更实用的，特别是在处理大数据库时，这样你就不必浏览过多的信息。你可以通过在`SELECT`关键字后列出列名并用逗号分隔来实现。以下是一个示例：

```
SELECT some_column, another_column, amazing_column FROM table_name;
```

使用这种语法，查询将仅从这三列中检索所有行。

让我们将此应用于`teachers`表。也许在你的分析中，你想重点关注教师的姓名和薪水。在这种情况下，你只需选择相关的列，如清单 3-2 所示。请注意，查询中列的顺序与表中的顺序不同：你可以按照任何你喜欢的顺序检索列。

```
SELECT last_name, first_name, salary FROM teachers;
```

清单 3-2：查询列的子集

现在，在结果集里，你已将列限制为三列：

```
last_name    first_name    salary
---------    ----------    ------
Smith        Janet         36200
Reynolds     Lee           65000
Cole         Samuel        43500
Bush         Samantha      36200
Diaz         Betty         43500
Roush        Kathleen      38500
```

尽管这些示例很基础，但它们展示了开始数据集分析的一个好策略。通常，开始分析时最好先检查你的数据是否存在并且格式符合预期，这是`SELECT`非常适合完成的任务。日期是否按正确格式输入，包含了月、日和年，还是像我曾经遗憾地观察到的那样，只输入了月和年作为文本？每行是否在所有列中都有值？是否没有以*M*字母开头的姓氏？所有这些问题都指示着潜在的风险，从缺失数据到某个环节的记录不当。

我们只处理一个包含六行的表，但当你面对一个有成千上万甚至百万行的表时，快速了解数据质量和它包含的值范围就显得至关重要。为此，让我们更深入地挖掘，并添加几个 SQL 关键字。

## 使用 ORDER BY 排序数据

当数据按顺序排列时，通常更容易理解，并且可能更容易揭示出模式，而不是随意混乱地排列。

在 SQL 中，我们使用包含关键字`ORDER BY`的子句对查询结果进行排序，后面跟上要排序的列名。应用这个子句不会改变原始表格，只会改变查询的结果。列表 3-3 展示了一个使用`teachers`表的示例。

```
SELECT first_name, last_name, salary
FROM teachers
ORDER BY salary DESC;
```

列表 3-3：使用`ORDER BY`对一列进行排序

默认情况下，`ORDER BY`按升序排列值，但在这里我通过添加`DESC`关键字进行降序排序。（可选的`ASC`关键字指定升序排序。）现在，通过按从高到低的顺序排列`salary`列，我可以确定哪些教师收入最高：

```
first_name    last_name    salary
----------    ---------    ------
Lee           Reynolds     65000
Samuel        Cole         43500
Betty         Diaz         43500
Kathleen      Roush        38500
Janet         Smith        36200
Samantha      Bush         36200
```

`ORDER BY`子句也接受数字而非列名，数字根据其在`SELECT`子句中的位置来确定排序的列。因此，你可以这样重写列表 3-3，使用`3`来引用`SELECT`子句中的第三列`salary`：

```
SELECT first_name, last_name, salary
FROM teachers
ORDER BY 3 DESC;
```

在查询中排序的能力为我们提供了极大的灵活性，帮助我们以不同方式查看和展示数据。例如，我们不必局限于仅对一列进行排序。输入列表 3-4 中的语句。

```
SELECT last_name, school, hire_date
FROM teachers
1 ORDER BY school ASC, hire_date DESC;
```

列表 3-4：使用`ORDER BY`对多个列进行排序

在这种情况下，我们检索教师的姓氏、学校和聘用日期。通过按升序排序`school`列和按降序排序`hire_date`，我们创建了一个按学校分组的教师列表，其中最新聘用的教师排在前面。这可以让我们看到每所学校的最新教师。结果集应该如下所示：

```
last_name    school                 hire_date
---------    -------------------    ----------
Smith        F.D. Roosevelt HS      2011-10-30
Roush        F.D. Roosevelt HS      2010-10-22
Reynolds     F.D. Roosevelt HS      1993-05-22
Bush         Myers Middle School    2011-10-30
Diaz         Myers Middle School    2005-08-30
Cole         Myers Middle School    2005-08-01
```

你可以在两个以上的列上使用`ORDER BY`，但很快你会发现效果开始递减，几乎难以察觉。如果你在`ORDER BY`子句中加入关于教师最高学历、所教年级和出生日期的列，那么在输出中一次性理解各种排序方向将变得非常困难，更不用说将其传达给他人了。数据的消化最容易发生在结果专注于回答特定问题时；因此，更好的策略是将查询中的列数限制为最重要的列，并进行多个查询来回答每个问题。

## 使用 DISTINCT 查找唯一值

在表格中，某一列包含重复值的行并不罕见。例如，在`teachers`表中，`school`列多次列出了相同的学校名称，因为每所学校有很多教师。

为了了解列中的值的范围，我们可以使用`DISTINCT`关键字，它是查询的一部分，能够消除重复项并仅显示唯一值。如列表 3-5 所示，`DISTINCT`应紧跟在`SELECT`之后使用。

```
SELECT DISTINCT school
FROM teachers
ORDER BY school;
```

列表 3-5：查询`school`列中的唯一值

结果如下所示：

```
school
-------------------
F.D. Roosevelt HS
Myers Middle School
```

尽管表中有六行数据，输出只显示 `school` 列中的两个唯一学校名称。这是评估数据质量的一个有用的第一步。例如，如果一个学校名称有多种拼写方式，这些拼写差异将很容易被发现并修正，特别是如果你对输出进行排序的话。

当你处理日期或数字时，`DISTINCT` 将帮助你突出不一致或格式错误的情况。例如，你可能会继承一个数据集，其中日期被输入到一个格式为 `text` 数据类型的列中。这种做法（你应该避免）允许格式错误的日期存在：

```
date
---------
5/30/2023
6//2023
6/1/2023
6/2/2023
```

`DISTINCT` 关键字也可以同时作用于多列。如果我们增加一列，查询将返回每对唯一值。运行 示例 3-6 中的代码。

```
SELECT DISTINCT school, salary
FROM teachers
ORDER BY school, salary;
```

示例 3-6：查询 `school` 和 `salary` 列中唯一值的配对

现在，查询返回每个学校所获得的唯一（或不同）工资。因为 Myers 中学的两位教师薪水为 $43,500，这一对只列在一行中，查询返回五行，而不是表中的六行：

```
school                 salary
-------------------    ------
F.D. Roosevelt HS      36200
F.D. Roosevelt HS      38500
F.D. Roosevelt HS      65000
Myers Middle School    36200
Myers Middle School    43500
```

这种技术让我们能够提出问题：“在表中的每个 *x*，所有的 *y* 值是什么？”例如，对于每个工厂，它生产的所有化学品是什么？对于每个选举区，所有竞选公职的候选人是谁？对于每个音乐厅，本月演出的艺术家是谁？

SQL 提供了更复杂的技术与聚合函数，允许我们计数、求和，并找到最小值和最大值。我将在第六章和第九章中详细介绍这些内容。

## 使用 WHERE 过滤行

有时，你希望限制查询返回的行，仅显示一列或多列满足特定条件的行。以 `teachers` 为例，你可能希望找到所有在某一年之前被雇佣的教师，或者所有年薪超过 $75,000 的小学教师。为此，我们使用 `WHERE` 子句。

`WHERE` 子句允许你根据通过*运算符*提供的条件查找匹配特定值、值范围或多个值的行——运算符是一个让我们执行数学、比较和逻辑操作的关键字。你也可以使用条件排除某些行。

示例 3-7 展示了一个基本示例。请注意，在标准 SQL 语法中，`WHERE` 子句位于 `FROM` 关键字之后，并且紧跟着被查询的表名。

```
SELECT last_name, school, hire_date
FROM teachers
WHERE school = 'Myers Middle School';
```

示例 3-7：使用 `WHERE` 过滤行

结果集仅显示分配给 Myers 中学的教师。

```
last_name    school                 hire_date
---------    -------------------    ----------
Cole         Myers Middle School    2005-08-01
Bush         Myers Middle School    2011-10-30
Diaz         Myers Middle School    2005-08-30
```

在这里，我使用等于比较运算符来查找完全匹配某个值的行，当然，你也可以在 `WHERE` 子句中使用其他运算符来自定义你的过滤条件。表 3-1 总结了最常用的比较运算符。根据你的数据库系统，可能还会有更多可用的运算符。

表 3-1：PostgreSQL 中的比较和匹配运算符

| **运算符** | **功能** | **示例** |
| --- | --- | --- |
| `=` | 等于 | `WHERE school = 'Baker Middle'` |
| `<>` 或 `!=` | 不等于^* | `WHERE school <> 'Baker Middle'` |
| `>` | 大于 | `WHERE salary > 20000` |
| `<` | 小于 | `WHERE salary < 60500` |
| `>=` | 大于或等于 | `WHERE salary >= 20000` |
| `<=` | 小于或等于 | `WHERE salary <= 60500` |
| `BETWEEN` | 在范围内 | `WHERE salary BETWEEN 20000 AND 40000` |
| `IN` | 匹配一组值中的一个 | `WHERE last_name IN ('Bush', 'Roush')` |
| `LIKE` | 匹配一个模式（区分大小写） | `WHERE first_name LIKE 'Sam%'` |
| `ILIKE` | 匹配一个模式（不区分大小写） | `WHERE first_name ILIKE 'sam%'` |
| `NOT` | 否定一个条件 | `WHERE first_name NOT ILIKE 'sam%'` |

以下示例展示了比较运算符的应用。首先，我们使用等于运算符查找名字为 Janet 的教师：

```
SELECT first_name, last_name, school
FROM teachers
WHERE first_name = 'Janet';
```

接下来，我们列出表格中所有学校的名称，但排除 F.D. Roosevelt HS，使用不等于运算符：

```
SELECT school
FROM teachers
WHERE school <> 'F.D. Roosevelt HS';
```

在这里，我们使用小于运算符列出 2000 年 1 月 1 日之前被聘用的教师（使用`YYYY-MM-DD`日期格式）：

```
SELECT first_name, last_name, hire_date
FROM teachers
WHERE hire_date < '2000-01-01';
```

然后，我们使用`>=`运算符查找薪水为$43,500 或以上的教师：

```
SELECT first_name, last_name, salary
FROM teachers
WHERE salary >= 43500;
```

下一个查询使用`BETWEEN`运算符查找薪水在$40,000 到$65,000 之间的教师。注意，`BETWEEN`是*包含的*，意味着结果将包括与指定的起始和结束范围匹配的值。

```
SELECT first_name, last_name, school, salary
FROM teachers
WHERE salary BETWEEN 40000 AND 65000;
```

使用`BETWEEN`时要小心，因为它的包容性可能会导致值的重复计算。例如，如果你使用`BETWEEN 10 AND 20`进行过滤，然后再使用`BETWEEN 20 AND 30`执行第二次查询，值为 20 的行会出现在两个查询结果中。你可以通过使用更明确的大于和小于运算符来定义范围，从而避免这种情况。例如，这个查询返回与之前相同的结果，但更明显地指定了范围：

```
SELECT first_name, last_name, school, salary
FROM teachers
WHERE salary >= 40000 AND salary <= 65000;
```

我们将在本书中反复回到这些运算符，因为它们将在帮助我们找到所需的数据和答案时发挥重要作用。

### 使用`LIKE`和`ILIKE`与`WHERE`

比较运算符相对简单明了，但匹配运算符`LIKE`和`ILIKE`需要额外的解释。它们都允许你查找包括与指定模式匹配的字符的多种值，如果你不完全知道自己在搜索什么，或者在找出拼写错误的单词时，它们非常有用。要使用`LIKE`和`ILIKE`，你需要使用一个或两个符号来指定匹配的模式：

1.  百分号(%) 匹配一个或多个字符的通配符

1.  下划线(_) 匹配一个字符的通配符

例如，如果你尝试查找单词`baker`，以下`LIKE`模式将匹配它：

```
LIKE 'b%'
LIKE '%ak%'
LIKE '_aker'
LIKE 'ba_er'
```

区别是什么？`LIKE`运算符是 ANSI SQL 标准的一部分，区分大小写。`ILIKE`运算符是 PostgreSQL 特有的实现，不区分大小写。清单 3-8 展示了这两个关键字如何返回不同的结果。第一个`WHERE`子句使用`LIKE` 1 来查找以`sam`开头的名称，由于它区分大小写，所以不会返回任何结果。第二个使用不区分大小写的`ILIKE` 2 会从表中返回`Samuel`和`Samantha`。

```
SELECT first_name
FROM teachers
1 WHERE first_name `LIKE` 'sam%';

SELECT first_name
FROM teachers
2 WHERE first_name `ILIKE` 'sam%';
```

清单 3-8: 使用`LIKE`和`ILIKE`进行筛选

多年来，我倾向于使用`ILIKE`和通配符运算符，以确保在搜索时不会无意间排除结果，特别是在审查数据时。我不假设输入人名、地名、产品名或其他专有名词的人总是记得正确地大写它们。如果面试数据的目标之一是了解其质量，那么使用不区分大小写的搜索将帮助你发现不同的变体。

由于`LIKE`和`ILIKE`是模式搜索，在大数据库中性能可能较慢。我们可以通过使用索引来提高性能，关于这一点我将在第八章的“通过索引加速查询”中详细介绍。

### 结合`AND`和`OR`运算符

当我们将比较运算符结合使用时，它们变得更加有用。为此，我们使用逻辑运算符`AND`和`OR`将它们连接起来，必要时还可以使用括号。

清单 3-9 中的语句展示了三种以这种方式结合运算符的示例。

```
SELECT *
FROM teachers
1 WHERE school = 'Myers Middle School'
      AND salary < 40000;

SELECT *
FROM teachers
2 WHERE last_name = 'Cole'
      OR last_name = 'Bush';

SELECT *
FROM teachers
3 WHERE school = 'F.D. Roosevelt HS'
      AND (salary < 38000 OR salary > 40000);
```

清单 3-9: 使用`AND`和`OR`结合运算符

第一个查询在`WHERE`子句 1 中使用`AND`来查找在迈尔斯中学工作且薪水低于 40,000 美元的教师。因为我们使用`AND`连接这两个条件，所以这两个条件必须同时成立，行才会满足`WHERE`子句中的条件并返回查询结果。

第二个示例使用`OR` 2 来搜索姓氏匹配 Cole 或 Bush 的任何教师。当我们使用`OR`连接条件时，只需要其中一个条件为真，行就会满足`WHERE`子句的条件。

最后的示例搜索罗斯福学校的教师，他们的薪水要么低于 38,000 美元，要么高于 40,000 美元。当我们将语句放入括号中时，这些语句会作为一组先进行评估，然后再与其他条件结合。在这种情况下，学校名称必须是`F.D. Roosevelt HS`，薪水必须低于或高于指定的值，才能使该行满足`WHERE`子句的条件。

如果我们在子句中同时使用`AND`和`OR`，但没有使用括号，数据库将首先评估`AND`条件，然后评估`OR`条件。在最后一个示例中，这意味着如果我们省略括号，结果会不同——数据库会寻找学校名称为`F.D. Roosevelt HS`且薪水低于 38,000 美元的行，或者寻找任何薪水高于 40,000 美元的学校行。在查询工具中试试看。

## 综合应用

你可以开始看到，即使是之前的简单查询，也能让我们灵活而精确地深入数据，找到我们所寻找的信息。你可以使用 `AND` 和 `OR` 关键字组合比较操作符语句，以提供多个筛选标准，并且可以包括 `ORDER BY` 子句对结果进行排序。

考虑到前面的信息，让我们将本章的概念结合成一个语句，展示它们如何结合在一起。SQL 对关键词的顺序非常讲究，因此请遵循这一惯例。

```
SELECT column_names
FROM table_name
WHERE criteria
ORDER BY column_names;
```

列表 3-10 展示了一个针对 `teachers` 表的查询，包含了所有上述内容。

```
SELECT first_name, last_name, school, hire_date, salary
FROM teachers
WHERE school LIKE '%Roos%'
ORDER BY hire_date DESC;
```

列表 3-10：一个包含 `WHERE` 和 `ORDER BY` 的 `SELECT` 语句

这个列表返回的是罗斯福高中教师的数据，按从最新雇佣到最早雇佣的顺序排列。我们可以看到教师的入职日期与其当前薪资水平之间的一些联系：

```
first_name    last_name    school               hire_date     salary
----------    ---------    -----------------    ----------    ------
Janet         Smith        F.D. Roosevelt HS    2011-10-30    36200
Kathleen      Roush        F.D. Roosevelt HS    2010-10-22    38500
Lee           Reynolds     F.D. Roosevelt HS    1993-05-22    65000
```

## 总结

现在你已经学会了几种不同 SQL 查询的基本结构，你已经为后续章节中我将介绍的许多附加技能奠定了基础。排序、筛选和从表中选择最重要的列，能够从数据中获得令人惊讶的信息，并帮助你找到数据背后的故事。

在下一章，你将学习 SQL 的另一个基础方面：数据类型。
