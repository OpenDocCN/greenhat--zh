# 第十三章：高级查询技巧

![](img/chapterart.png)

有时候，数据分析需要一些超越表连接或基础 `SELECT` 查询的高级 SQL 技巧。在本章中，我们将介绍一些技巧，包括编写查询，利用其他查询的结果作为输入，并在计数之前将数值重新分类为不同的类别。

在这些练习中，我将介绍一个包含美国部分城市温度数据集，并回顾你在前几章中创建的数据集。本书的练习代码及所有资源可通过 [`nostarch.com/practical-sql-2nd-edition/`](https://nostarch.com/practical-sql-2nd-edition/) 获得。你将继续使用你已经建立的 `analysis` 数据库。让我们开始吧。

## 使用子查询

*子查询* 是嵌套在另一个查询内部的查询。通常，它执行计算或逻辑测试，或生成行以传递给外部的主查询。子查询是标准 ANSI SQL 的一部分，语法并不特别：我们只是将查询括在括号内。例如，我们可以编写一个返回多行的子查询，并将这些结果当作 `FROM` 子句中的表来使用。或者，我们可以创建一个 *标量子查询*，它返回一个单一值，并将其作为 *表达式* 的一部分，用于通过 `WHERE`、`IN` 和 `HAVING` 子句过滤行。*相关子查询* 是指依赖外部查询中的某个值或表名来执行的子查询。相反，*非相关子查询* 则不引用主查询中的任何对象。

通过操作数据来理解这些概念会更容易，因此让我们回顾一些早期章节中的数据集，包括 `us_counties_pop_est_2019` 人口普查县级估算表和 `cbp_naics_72_establishments` 商业模式表。

### 在 `WHERE` 子句中过滤使用子查询

`WHERE` 子句允许你根据提供的标准过滤查询结果，使用像 `WHERE quantity > 1000` 这样的表达式。但这要求你已经知道要用来比较的值。如果你不知道该怎么做呢？这时子查询就派上用场了：它让你编写一个查询，生成一个或多个值，用作 `WHERE` 子句中的表达式的一部分。

#### 为查询表达式生成值

假设你想编写一个查询，显示哪些美国县的人口位于或超过第 90 百分位数，或者是前 10%的县。你不必编写两个独立的查询——一个计算 90 百分位数，另一个查找人口位于或高于这个数值的县——你可以一次性完成这两个任务，使用子查询作为 `WHERE` 子句的一部分，如列表 13-1 所示。

```
SELECT county_name,
       state_name,
       pop_est_2019
FROM us_counties_pop_est_2019
1 WHERE pop_est_2019 >= (
    SELECT percentile_cont(.9) WITHIN GROUP (ORDER BY pop_est_2019)
    FROM us_counties_pop_est_2019
    )
ORDER BY pop_est_2019 DESC;
```

列表 13-1：在 `WHERE` 子句中使用子查询

`WHERE`子句 1 通过总人口列`pop_est_2019`进行筛选，但没有像通常那样指定一个值。相反，在`>=`比较操作符之后，我们提供了一个括号中的子查询。这个子查询使用`percentile_cont()`函数生成一个值：`pop_est_2019`列中的第 90 百分位数临界点。

这是一个无关子查询的示例。它不依赖于外部查询中的任何值，并且只会执行一次以生成所请求的值。如果你只运行子查询部分，通过在 pgAdmin 中高亮它，它将执行，你应该会看到`213707.3`的结果。但在运行整个查询示例 13-1 时，你看不到这个数字，因为子查询的结果会直接传递到外部查询的`WHERE`子句中。

整个查询应该返回 315 行，约占`us_counties_pop_est_2019`表中 3,142 行的 10%。

```
 county_name            state_name      pop_est_2019
----------------------- -------------------- ------------
Los Angeles County      California               10039107
Cook County             Illinois                  5150233
Harris County           Texas                     4713325
Maricopa County         Arizona                   4485414
San Diego County        California                3338330
`--snip--`
Cabarrus County         North Carolina             216453
Yuma County             Arizona                    213787
```

结果包括所有人口大于或等于`213707.3`的县，这是子查询生成的值。

#### 使用子查询来识别要删除的行

我们可以在`DELETE`语句中使用相同的子查询来指定从表中删除的内容。在示例 13-2 中，我们使用你在第十章学到的方法创建了一个人口普查表的副本，然后从该备份中删除除前 10%人口的 315 个县以外的所有数据。

```
CREATE TABLE us_counties_2019_top10 AS
SELECT * FROM us_counties_pop_est_2019;

DELETE FROM us_counties_2019_top10
WHERE pop_est_2019 < (
    SELECT percentile_cont(.9) WITHIN GROUP (ORDER BY pop_est_2019)
    FROM us_counties_2019_top10
    );
```

示例 13-2：在`WHERE`子句中使用子查询与`DELETE`

执行示例 13-2 中的代码，然后执行`SELECT count(*) FROM us_counties_2019_top10;`来计算剩余的行数。结果应该是 315 行，即原始的 3,142 行减去子查询所识别的 2,827 行。

### 使用子查询创建派生表

如果你的子查询返回行和列，你可以将其放入`FROM`子句中，创建一个新的表，称为*派生表*，你可以像查询常规表一样查询或与其他表进行连接。这是另一个无关子查询的示例。

让我们看一个简单的例子。在第六章中，你学习了平均数和中位数的区别。中位数通常更能准确表示数据集的中心值，因为少数几个极大或极小的值（或异常值）会偏移平均值。因此，我常常比较二者。如果它们接近，数据更可能符合*正态分布*（熟悉的钟形曲线），此时平均值可以很好地代表中心值。如果平均值和中位数差距很大，则可能存在异常值的影响，或者数据分布是偏斜的，非正态的。

查找美国县的平均人口和中位数人口以及二者之间的差异是一个两步过程。我们需要计算平均值和中位数，然后将二者相减。我们可以通过在`FROM`子句中使用子查询一次性完成这两个操作，如示例 13-3 所示。

```
SELECT round(calcs.average, 0) AS average,
       calcs.median,
       round(calcs.average - calcs.median, 0) AS median_average_diff
FROM (
   1 SELECT avg(pop_est_2019) AS average,
            percentile_cont(.5)
                WITHIN GROUP (ORDER BY pop_est_2019)::numeric AS median
     FROM us_counties_pop_est_2019
     )
2 AS calcs;
```

示例 13-3：作为派生表的子查询用于`FROM`子句

产生派生表的子查询 1 很直接。我们使用`avg()`和`percentile_cont()`函数来计算人口普查表中`pop_est_2019`列的平均值和中位数，并为每一列命名一个别名。然后，我们将派生表命名为`calcs` 2，这样我们就可以在主查询中引用它。

在主查询中，我们将子查询返回的`median`与`average`相减。结果会被四舍五入并标记为别名`median_average_diff`。执行查询后，结果应该如下所示：

```
average    median     median_average_diff
-------    -------    -------------------
 104468      25726                  78742
```

中位数和平均数之间的差异 78,742 几乎是中位数的三倍。这表明我们有一些高人口的县城拉高了平均数。

### 连接派生表

连接多个派生表可以让你在主查询的最终计算之前执行多个预处理步骤。例如，在第十一章中，我们计算了每千人中与旅游相关的企业数量。假设我们想在州一级进行相同的计算。在我们计算这个比率之前，我们需要知道每个州的旅游企业数量和每个州的人口。 示例 13-4 展示了如何为这两个任务编写子查询，并将它们连接起来计算整体比率。

```
SELECT census.state_name AS st,
       census.pop_est_2018,
       est.establishment_count,
     1 round((est.establishment_count/census.pop_est_2018::numeric) * 1000, 1)
           AS estabs_per_thousand
FROM
    (
      2 SELECT st,
               sum(establishments) AS establishment_count
        FROM cbp_naics_72_establishments
        GROUP BY st
    )
    AS est
JOIN
    (
      3 SELECT state_name,
               sum(pop_est_2018) AS pop_est_2018
        FROM us_counties_pop_est_2019
        GROUP BY state_name
    )
    AS census
4 ON est.st = census.state_name
ORDER BY estabs_per_thousand DESC;
```

示例 13-4：连接两个派生表

你在第十一章中学过如何计算比率，所以外部查询中寻找`estabs_per_thousand` 1 的数学运算和语法应该是熟悉的。我们将企业数量除以人口，然后将商数乘以千。对于输入，我们使用从两个派生表生成的值。

第一个 2 使用`sum()`聚合函数找出每个州的企业数量。我们将这个派生表命名为`est`，以便在查询的主部分引用。第二个 3 使用`sum()`计算每个州的 2018 年估算人口，基于`pop_est_2018`列。我们将这个派生表命名为`census`。

接下来，我们通过将`est`中的`st`列与`census`中的`state_name`列连接起来，连接这两个派生表 4。然后，我们根据比率按降序列出结果。以下是 51 行中的一个样本，展示了最高和最低的比率：

```
 st          pop_est_2018 establishment_count estabs_per_thousand
-------------------- ------------ ------------------- -------------------
District of Columbia       701547                2754                 3.9
Montana                   1060665                3569                 3.4
Vermont                    624358                1991                 3.2
Maine                     1339057                4282                 3.2
Wyoming                    577601                1808                 3.1
`--snip--`
Arizona                   7158024               13288                 1.9
Alabama                   4887681                9140                 1.9
Utah                      3153550                6062                 1.9
Mississippi               2981020                5645                 1.9
Kentucky                  4461153                8251                 1.8
```

排在第一位的是华盛顿特区，这并不令人意外，因为国会大厦、博物馆、纪念碑以及其他旅游景点产生了大量游客活动。蒙大拿州排第二似乎令人惊讶，但它是一个人口较少的州，拥有主要的旅游目的地，包括冰川国家公园和黄石国家公园。密西西比州和肯塔基州是每千人中旅游相关企业最少的州。

### 使用子查询生成列

你也可以在`SELECT`后面的列列表中放置子查询，以生成查询结果中该列的值。子查询必须只生成一行数据。例如，清单 13-5 中的查询从`us_counties_pop_est_2019`中选择地理和人口信息，然后添加一个无关的子查询，将所有县的中位数添加到新列`us_median`的每一行中。

```
SELECT county_name,
       state_name AS st,
       pop_est_2019,
       (SELECT percentile_cont(.5) WITHIN GROUP (ORDER BY pop_est_2019)
        FROM us_counties_pop_est_2019) AS us_median
FROM us_counties_pop_est_2019;
```

清单 13-5：向列列表中添加子查询

结果集的前几行应该如下所示：

```
 county_name                     st          pop_est_2019 us_median
--------------------------------- -------------------- ------------ ---------
Autauga County                    Alabama                     55869     25726
Baldwin County                    Alabama                    223234     25726
Barbour County                    Alabama                     24686     25726
Bibb County                       Alabama                     22394     25726
Blount County                     Alabama                     57826     25726
`--snip--`
```

单独来看，重复的`us_median`值并不是特别有用。更有趣和实用的是生成一些值，表示每个县的人口与中位数值的偏差。让我们看看如何使用相同的子查询技巧来实现这一点。清单 13-6 在清单 13-5 的基础上，替换了一个子查询，计算每个县人口与中位数之间的差异。

```
SELECT county_name,
       state_name AS st,
       pop_est_2019,
       pop_est_2019 - (SELECT percentile_cont(.5) WITHIN GROUP (ORDER BY pop_est_2019) 1
                       FROM us_counties_pop_est_2019) AS diff_from_median
FROM us_counties_pop_est_2019
WHERE (pop_est_2019 - (SELECT percentile_cont(.5) WITHIN GROUP (ORDER BY pop_est_2019) 2
                       FROM us_counties_pop_est_2019))
       BETWEEN -1000 AND 1000;
```

清单 13-6：在计算中使用子查询

子查询 1 现在成为一个计算的一部分，该计算将子查询的结果从`pop_est_2019`（总人口）中减去，为列指定了别名`diff_from_median`。为了使这个查询更加有用，我们可以过滤结果，显示人口接近中位数的县。为此，我们在`WHERE`子句 2 中重复子查询计算，并使用`BETWEEN -1000 AND 1000`表达式过滤结果。

结果应该会显示 78 个县。以下是前五行：

```
 county_name             st       pop_est_2019 diff_from_median
----------------------- -------------- ------------ ----------------
Cherokee County         Alabama               26196              470
Geneva County           Alabama               26271              545
Cleburne County         Arkansas              24919             -807
Johnson County          Arkansas              26578              852
St. Francis County      Arkansas              24994             -732
`--snip--`
```

请记住，子查询可能会增加整体查询执行时间。在清单 13-6 中，我从清单 13-5 中移除了显示列`us_median`的子查询，以避免第三次重复子查询。对于我们的数据集，影响是最小的；如果我们处理的是数百万行数据，剔除一些不必要的子查询可能会显著提高速度。

### 理解子查询表达式

你也可以使用子查询通过评估条件是否为`true`或`false`来过滤行。为此，我们可以使用*子查询表达式*，这是一种将关键字与子查询结合的表达式，通常用于`WHERE`子句中，根据另一个表中是否存在某些值来过滤行。

PostgreSQL 文档在[`www.postgresql.org/docs/current/functions-subquery.html`](https://www.postgresql.org/docs/current/functions-subquery.html)中列出了可用的子查询表达式，但在这里我们将检查两种最常用的子查询语法：`IN`和`EXISTS`。在此之前，运行清单 13-7 中的代码，创建一个名为`retirees`的小表，我们将与第七章中构建的`employees`表一起查询。我们假设已经从供应商处收到数据，列出了申请退休福利的人。

```
CREATE TABLE retirees (
    id int,
    first_name text,
    last_name text
);

INSERT INTO retirees
VALUES (2, 'Janet', 'King'),
       (4, 'Michael', 'Taylor');
```

清单 13-7：创建并填充`retirees`表

现在让我们在一些子查询表达式中使用这个表。

#### 为 `IN` 运算符生成值

子查询表达式 `IN (``subquery``)` 的工作方式与第三章中 `IN` 运算符的例子类似，只不过我们使用一个子查询来提供检查的值列表，而不是手动输入一个。在清单 13-8 中，我们使用一个不相关的子查询，它将执行一次，生成来自 `retirees` 表的 `id` 值。它返回的值成为 `WHERE` 子句中 `IN` 运算符的列表。这使我们能够找到也出现在退休人员表中的员工。

```
SELECT first_name, last_name
FROM employees
WHERE emp_id IN (
    SELECT id
    FROM retirees)
ORDER BY emp_id;
```

清单 13-8：生成 `IN` 运算符的值

运行查询时，输出将显示 `employees` 中的两个人，他们的 `emp_id` 在 `retirees` 表中有匹配的 `id`：

```
first_name last_name
---------- ---------
Janet      King
Michael    Taylor
```

#### 检查值是否存在

子查询表达式 `EXISTS (``subquery``)` 如果括号中的子查询返回至少一行，则返回 `true`，如果没有返回任何行，则 `EXISTS` 评估为 `false`。

在清单 13-9 中的 `EXISTS` 子查询表达式展示了一个关联子查询的例子——它在 `WHERE` 子句中包含一个需要外部查询数据的表达式。此外，由于子查询是关联的，它会对外部查询返回的每一行执行一次，每次检查 `retirees` 中是否有一个与 `employees` 中的 `emp_id` 匹配的 `id`。如果有匹配，`EXISTS` 表达式返回 `true`。

```
SELECT first_name, last_name
FROM employees
WHERE EXISTS (
    SELECT id
    FROM retirees
    WHERE id = employees.emp_id);
```

清单 13-9：使用 `WHERE EXISTS` 的关联子查询

当你运行代码时，它应该返回与清单 13-8 中相同的结果。使用这种方法特别有帮助，特别是当你需要连接多个列时，这是 `IN` 表达式无法做到的。你还可以在 `EXISTS` 中添加 `NOT` 关键字来执行相反的操作，查找员工表中没有与 `retirees` 表中的记录相对应的行，如清单 13-10 所示。

```
SELECT first_name, last_name
FROM employees
WHERE NOT EXISTS (
    SELECT id
    FROM retirees
    WHERE id = employees.emp_id);
```

清单 13-10：使用 `WHERE NOT EXISTS` 的关联子查询

这应该会产生以下结果：

```
first_name last_name
---------- ---------
Julia      Reyes
Arthur     Pappas
```

使用 `NOT` 和 `EXISTS` 的技巧对于查找缺失值或评估数据集是否完整非常有用。

### 使用 `LATERAL` 的子查询

在 `FROM` 子句中放置 `LATERAL` 关键字在子查询之前，增加了几个功能，这有助于简化本来复杂的查询。

#### 使用 `LATERAL` 与 `FROM`

首先，前面加上 `LATERAL` 的子查询可以引用在 `FROM` 子句中出现在它之前的表和其他子查询，这可以通过使计算易于重用来减少冗余代码。

清单 13-11 通过两种方式计算从 2018 年到 2019 年县人口的变化：原始变化数和百分比变化。

```
SELECT county_name,
       state_name,
       pop_est_2018,
       pop_est_2019,
       raw_chg,
       round(pct_chg * 100, 2) AS pct_chg
FROM us_counties_pop_est_2019,
     1 LATERAL (SELECT pop_est_2019 - pop_est_2018 AS raw_chg) rc,
     2 LATERAL (SELECT raw_chg / pop_est_2018::numeric AS pct_chg) pc
ORDER BY pct_chg DESC;
```

清单 13-11：在 `FROM` 子句中使用 `LATERAL` 子查询

在`FROM`子句中，在指定`us_counties_pop_est_2019`表后，我们添加了第一个`LATERAL`子查询 1。在括号内，我们编写一个查询，计算 2018 年人口估算与 2019 年估算的差值，并将结果别名为`raw_chg`。由于`LATERAL`子查询可以引用`FROM`子句中之前列出的表，而无需指定其名称，因此我们可以省略`us_counties_pop_est_2019`表在子查询中的引用。`FROM`中的子查询必须有别名，因此我们将其标记为`rc`。

第二个`LATERAL`子查询 2 计算 2018 年到 2019 年人口的百分比变化。为了找到百分比变化，我们必须知道原始变化量。我们可以通过引用前一个子查询中的`raw_chg`值来避免重新计算它。这有助于使我们的代码更简洁、更易读。

查询结果应如下所示：

```
 county_name     state_name  pop_est_2018 pop_est_2019 raw_chg pct_chg
---------------- ------------ ------------ ------------ ------- -------
Loving County    Texas                 148          169      21   14.19
McKenzie County  North Dakota        13594        15024    1430   10.52
Loup County      Nebraska              617          664      47    7.62
Kaufman County   Texas              128279       136154    7875    6.14
Williams County  North Dakota        35469        37589    2120    5.98
`--snip--`
```

#### `LATERAL`与`JOIN`

将`LATERAL`与`JOIN`结合使用，创建了类似于编程语言中*for 循环*的功能：对于`LATERAL`连接前由查询生成的每一行，在`LATERAL`连接之后的子查询或函数将被评估一次。

我们将重新使用第二章中的`teachers`表，并创建一个新表记录每次教师刷卡解锁实验室门的时间。我们的任务是找到教师访问实验室的两个最近时间。Listing 13-12 展示了代码。

```
1 ALTER TABLE teachers ADD CONSTRAINT id_key PRIMARY KEY (id);

2 CREATE TABLE teachers_lab_access (
    access_id bigint PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
    access_time timestamp with time zone,
    lab_name text,
    teacher_id bigint REFERENCES teachers (id)
);

3 INSERT INTO teachers_lab_access (access_time, lab_name, teacher_id)
VALUES ('2022-11-30 08:59:00-05', 'Science A', 2),
       ('2022-12-01 08:58:00-05', 'Chemistry B', 2),
       ('2022-12-21 09:01:00-05', 'Chemistry A', 2),
       ('2022-12-02 11:01:00-05', 'Science B', 6),
       ('2022-12-07 10:02:00-05', 'Science A', 6),
       ('2022-12-17 16:00:00-05', 'Science B', 6);

SELECT t.first_name, t.last_name, a.access_time, a.lab_name
FROM teachers t
4 LEFT JOIN LATERAL (SELECT *
                   FROM teachers_lab_access
                 5 WHERE teacher_id = t.id
                   ORDER BY access_time DESC
                   LIMIT 2)6 a
7 ON true
ORDER BY t.id;
```

Listing 13-12: 使用子查询与`LATERAL`连接

首先，我们使用`ALTER TABLE`为`teachers`表添加主键 1（在第二章中我们没有在此表上设置约束，因为我们只是介绍了创建表的基础知识）。接下来，我们创建一个简单的`teachers_lab_access`表 2，包含记录实验室名称和访问时间戳的列。该表有一个替代主键`access_id`，并且外键`teacher_id`引用了`teachers`表中的`id`。最后，我们使用`INSERT`3 语句向表中添加了六行数据。

现在我们准备查询数据。在我们的`SELECT`语句中，我们通过`LEFT JOIN`将`teachers`连接到一个子查询。我们添加了`LATERAL`4 关键字，这意味着对于从`teachers`返回的每一行，子查询将执行，返回该教师访问的两个最近的实验室及其访问时间。使用`LEFT JOIN`将返回所有来自`teachers`的行，无论子查询是否找到匹配的教师在`teachers_lab_access`中。

在`WHERE`子句中，子查询通过外键`teacher_lab_access`引用外部查询。此`LATERAL`连接语法要求子查询具有别名 6，这里为`a`，并且在`JOIN`子句的`ON`部分需要设置`true`的值 7。在这种情况下，`true`使我们能够创建连接，而不需要指定连接的特定列。

执行查询后，结果应如下所示：

```
first_name last_name      access_time         lab_name
---------- --------- ----------------------  ------------
Janet      Smith
Lee        Reynolds  2022-12-21 09:01:00-05  Chemistry A
Lee        Reynolds  2022-12-01 08:58:00-05  Chemistry B
Samuel     Cole
Samantha   Bush
Betty      Diaz
Kathleen   Roush     2022-12-17 16:00:00-05  Science B
Kathleen   Roush     2022-12-07 10:02:00-05  Science A
```

访问表中两个教师的 ID 显示了他们最近的两次实验室访问时间。没有访问实验室的教师显示 `NULL` 值；如果我们希望从结果中删除这些教师，可以用 `INNER JOIN`（或直接使用 `JOIN`）代替 `LEFT JOIN`。

接下来，让我们探索另一种处理子查询的语法。

## 使用公用表表达式

*公用表表达式*（*CTE*）是标准 SQL 中的一个相对较新的功能，它允许你使用一个或多个 `SELECT` 查询预定义临时表，并在主查询中根据需要反复引用这些临时表。CTE 非正式地称为 `WITH` 查询，因为它们是通过 `WITH ... AS` 语句定义的。以下示例展示了使用 CTE 的一些优势，包括代码更简洁、冗余更少。

列表 13-13 显示了一个基于我们普查估算数据的简单公用表表达式（CTE）。该代码确定了每个州中有多少县的人口达到 100,000 或以上。我们来逐步分析这个例子。

```
1 WITH large_counties (county_name, state_name, pop_est_2019)
AS (
  2 SELECT county_name, state_name, pop_est_2019
    FROM us_counties_pop_est_2019
    WHERE pop_est_2019 >= 100000
   )
3 SELECT state_name, count(*)
FROM large_counties
GROUP BY state_name
ORDER BY count(*) DESC;
```

列表 13-13：使用简单的 CTE 计算大县

`WITH ... AS` 语句 1 定义了临时表 `large_counties`。在 `WITH` 后，我们命名表并在括号中列出其列名。与 `CREATE TABLE` 语句中的列定义不同，我们不需要提供数据类型，因为临时表继承了来自子查询 2 的数据类型，子查询用括号括起来并位于 `AS` 后面。子查询必须返回与临时表中定义的列数相同的列，但列名不需要匹配。如果不重新命名列，则列列表是可选的；这里我列出了它以便你看到语法。

主查询 3 按 `state_name` 对 `large_counties` 中的行进行计数和分组，然后按降序排列计数。结果的前六行应该是这样的：

```
 state_name      count
-------------------- -----
Texas                   40
Florida                 36
California              35
Pennsylvania            31
New York                28
North Carolina          28
`--snip--`
```

德克萨斯州、佛罗里达州和加利福尼亚州是人口在 100,000 以上的县最多的州之一。

列表 13-14 使用 CTE 将 列表 13-4 中派生表的连接（查找每个州每千人中的旅游相关企业比率）改写成更易读的格式。

```
WITH
  1 counties (st, pop_est_2018) AS
    (SELECT state_name, sum(pop_est_2018)
     FROM us_counties_pop_est_2019
     GROUP BY state_name),

  2 establishments (st, establishment_count) AS
    (SELECT st, sum(establishments) AS establishment_count
     FROM cbp_naics_72_establishments
     GROUP BY st)

SELECT counties.st,
       pop_est_2018,
       establishment_count,
       round((establishments.establishment_count /
              counties.pop_est_2018::numeric(10,1)) * 1000, 1)
           AS estabs_per_thousand
3 FROM counties JOIN establishments
ON counties.st = establishments.st
ORDER BY estabs_per_thousand DESC;
```

列表 13-14：在表连接中使用 CTE

在 `WITH` 关键字之后，我们使用子查询定义了两个表。第一个子查询 `counties` 1 返回每个州的 2018 年人口。第二个子查询 `establishments` 2 返回每个州的旅游相关企业数量。定义了这些表之后，我们通过每个表中的 `st` 列将它们连接 3，并计算每千人的比率。结果与 列表 13-4 中连接的派生表相同，但 列表 13-14 更容易理解。

另一个例子是，你可以使用 CTE 简化那些包含冗余代码的查询。例如，在示例 13-6 中，我们在两个位置使用了带有`percentile_cont()`函数的子查询来查找中位数县人口。在示例 13-15 中，我们可以将这个子查询作为 CTE 只写一次。

```
1 WITH us_median AS
    (SELECT percentile_cont(.5)
     WITHIN GROUP (ORDER BY pop_est_2019) AS us_median_pop
     FROM us_counties_pop_est_2019)

SELECT county_name,
       state_name AS st,
       pop_est_2019,
     2 us_median_pop,
     3 pop_est_2019 - us_median_pop AS diff_from_median
4 FROM us_counties_pop_est_2019 CROSS JOIN us_median
5 WHERE (pop_est_2019 - us_median_pop)
       BETWEEN -1000 AND 1000;
```

示例 13-15：使用 CTE 来减少冗余代码

在`WITH`关键字之后，我们定义了`us_median` 1，作为示例 13-6 中相同子查询的结果，该子查询使用`percentile_cont()`查找中位数人口。然后，我们单独引用`us_median_pop`列 2，作为计算列 3 的一部分，并在`WHERE`子句 5 中使用它。为了在`SELECT`过程中将该值提供给`us_counties_pop_est_2019`表中的每一行，我们使用了第七章中学到的`CROSS JOIN` 4。

这个查询返回的结果与示例 13-6 中的结果相同，但我们只需要写一次查找中位数的子查询。另一个优点是，你可以更容易地修改查询。例如，要查找人口接近 90 百分位的县，只需要在`percentile_cont()`的输入中将`.5`替换为`.9`，并且只需在一个地方进行修改。

可读性强的代码、更少的冗余和更容易的修改通常是使用 CTE 的理由。另一个超出本书范围的理由是能够添加`RECURSIVE`关键字，使 CTE 可以在 CTE 内部循环查询结果——当处理按层次组织的数据时，这个功能特别有用。一个例子是公司的人员列表，可能你想找出所有向某位高管汇报的人员。递归 CTE 会从该高管开始，然后向下循环遍历各行，找到她的直接下属，再找到这些下属的下属。你可以通过 PostgreSQL 文档了解更多关于递归查询语法的信息，网址是[`www.postgresql.org/docs/current/queries-with.html`](https://www.postgresql.org/docs/current/queries-with.html)。

## 执行交叉表

*交叉表*提供了一种简单的方法，通过将变量以表格布局或矩阵的形式展示，从而总结和比较它们。矩阵中的行代表一个变量，列代表另一个变量，而每个行列交点处的单元格则包含一个值，例如计数或百分比。

你经常会看到交叉表格，也叫做*透视表*或*交叉表*，它们用于报告调查结果的汇总或比较成对的变量。一个常见的例子发生在选举期间，当候选人的选票按照地理位置进行统计时：

```
candidate    ward 1    ward 2    ward 3
---------    ------    ------    ------
Collins         602     1,799     2,112
Banks           599     1,398     1,616
Rutherford      911       902     1,114
```

在这种情况下，候选人的名字是一个变量，选区（或城市区）是另一个变量，而交点处的单元格则保存该候选人在该选区的得票总数。让我们看看如何生成交叉表格。

### 安装 crosstab()函数

标准 ANSI SQL 没有交叉表函数，但 PostgreSQL 作为一个你可以轻松安装的 *模块* 提供了此功能。模块是 PostgreSQL 的附加功能，不是核心应用程序的一部分；它们包括与安全性、文本搜索等相关的函数。你可以在 [`www.postgresql.org/docs/current/contrib.html`](https://www.postgresql.org/docs/current/contrib.html) 查找 PostgreSQL 模块的列表。

PostgreSQL 的 `crosstab()` 函数是 `tablefunc` 模块的一部分。要安装 `tablefunc`，请在 pgAdmin 中执行以下命令：

```
CREATE EXTENSION tablefunc;
```

PostgreSQL 应该返回消息 `CREATE EXTENSION`。（如果你使用的是其他数据库管理系统，请查阅其文档以获取类似功能。例如，Microsoft SQL Server 有 `PIVOT` 命令。）

接下来，我们将创建一个基本的交叉表，以便你可以学习语法，然后我们将处理一个更复杂的情况。

### 汇总调查结果

假设你的公司需要一个有趣的员工活动，因此你协调了在三个办公室举办的冰淇淋聚会。问题是，人们对冰淇淋口味很挑剔。为了选择每个办公室人们喜欢的口味，你决定进行一项调查。

CSV 文件 *ice_cream_survey.csv* 包含了对你调查的 200 个回应。你可以在 [`nostarch.com/practical-sql-2nd-edition/`](https://nostarch.com/practical-sql-2nd-edition/) 下载此文件以及本书的所有资源。每一行包括一个 `response_id`、`office` 和 `flavor`。你需要计算每个办公室每种口味的选择人数，并以可读的方式共享结果。

在你的 `analysis` 数据库中，使用 示例 13-16 中的代码创建表并加载数据。确保更改文件路径为你在计算机上保存 CSV 文件的位置。

```
CREATE TABLE ice_cream_survey (
    response_id integer PRIMARY KEY,
    office text,
    flavor text
);

COPY ice_cream_survey
FROM '`C:\YourDirectory\`ice_cream_survey.csv'
WITH (FORMAT CSV, HEADER);
```

示例 13-16：创建并填充 `ice_cream_survey` 表

如果你想检查数据，运行以下命令查看前五行：

```
SELECT *
FROM ice_cream_survey
ORDER BY response_id
LIMIT 5;
```

数据应该像这样：

```
response_id    office      flavor
-----------    --------    ----------
          1    Uptown      Chocolate
          2    Midtown     Chocolate
          3    Downtown    Strawberry
          4    Uptown      Chocolate
          5    Midtown     Chocolate
```

看起来巧克力口味领先！但让我们通过使用 示例 13-17 中的代码生成交叉表来确认这一选择。

```
SELECT *
1 FROM crosstab('SELECT 2 office,
                      3 flavor,
                      4 count(*)
               FROM ice_cream_survey
               GROUP BY office, flavor
               ORDER BY office',

            5 'SELECT flavor
               FROM ice_cream_survey
               GROUP BY flavor
               ORDER BY flavor')

6 AS (office text,
    chocolate bigint,
    strawberry bigint,
    vanilla bigint);
```

示例 13-17：生成冰淇淋调查交叉表

查询以 `SELECT *` 语句开始，该语句从 `crosstab()` 函数的内容中选择所有内容。我们将两个查询作为参数传递给 `crosstab()` 函数；请注意，由于这些查询是参数，我们将它们放在单引号内。第一个查询生成交叉表的数据，并包含三个必需的列。第一列，`office`，提供交叉表的行名称。第二列，`flavor`，提供与第三列中提供的值相关联的类别（或列）名称。这些值将显示在表中行与列交叉的每个单元格中。在这种情况下，我们希望交叉的单元格显示每个办公室选择的每种口味的 `count()`。这个第一个查询本身创建了一个简单的聚合列表。

第二个查询参数 5 生成列的类别名称。`crosstab()` 函数要求第二个子查询只返回一列，因此我们使用 `SELECT` 来检索 `flavor`，并使用 `GROUP BY` 来返回该列的唯一值。

然后，我们在 `AS` 关键字后指定交叉表输出列的名称和数据类型 6。该列表必须与查询生成的行和列的名称顺序一致。例如，由于提供类别列的第二个查询按字母顺序排列口味，输出列列表也必须如此。

当我们运行代码时，数据将以干净、易读的交叉表形式显示：

```
office      chocolate    strawberry    vanilla
--------    ---------    ----------    -------
Downtown           23            32         19
Midtown            41                       23
Uptown             22            17         23
```

一目了然，Midtown 办公室偏爱巧克力，但对草莓毫无兴趣，草莓的得票数为 `NULL`，表示草莓没有收到任何投票。而草莓是 Downtown 的首选，Uptown 办公室则在三种口味之间较为均衡。

### 城市温度读数汇总

让我们创建另一个交叉表，这次我们将使用真实数据。*temperature_readings.csv* 文件也可以在本书的所有资源中找到，网址为 [`nostarch.com/practical-sql-2nd-edition/`](https://nostarch.com/practical-sql-2nd-edition/)，其中包含来自美国三个观测站点（芝加哥、西雅图和威基基——位于檀香山市区南岸的一个社区）的为期一年的每日温度读数。数据来自美国国家海洋和大气管理局（NOAA），网址为 [`www.ncdc.noaa.gov/cdo-web/datatools/findstation/`](https://www.ncdc.noaa.gov/cdo-web/datatools/findstation/)。

CSV 文件中的每一行包含四个值：站点名称、日期以及当天的最高和最低温度。所有温度均为华氏度。对于每个城市的每个月，我们希望使用中位数最高温度来比较气候。Listing 13-18 中的代码用于创建 `temperature_readings` 表并导入 CSV 文件。

```
CREATE TABLE temperature_readings (
    station_name text,
    observation_date date,
    max_temp integer,
    min_temp integer,
    CONSTRAINT temp_key PRIMARY KEY (station_name, observation_date)
);

COPY temperature_readings
FROM '`C:\YourDirectory\`temperature_readings.csv'
WITH (FORMAT CSV, HEADER);
```

Listing 13-18：创建并填充 `temperature_readings` 表

表格包含来自 CSV 文件的四列；我们使用站点名称和观测日期添加了一个自然主键。快速计数应返回 1,077 行数据。现在，让我们看看使用 Listing 13-19 交叉汇总数据会产生什么效果。

```
SELECT *
FROM crosstab('SELECT
                1 station_name,
                2 date_part(''month'', observation_date),
                3 percentile_cont(.5)
                      WITHIN GROUP (ORDER BY max_temp)
               FROM temperature_readings
               GROUP BY station_name,
                        date_part(''month'', observation_date)
               ORDER BY station_name',

              'SELECT month
               FROM 4 generate_series(1,12) month')

AS (station text,
    jan numeric(3,0),
    feb numeric(3,0),
    mar numeric(3,0),
    apr numeric(3,0),
    may numeric(3,0),
    jun numeric(3,0),
    jul numeric(3,0),
    aug numeric(3,0),
    sep numeric(3,0),
    oct numeric(3,0),
    nov numeric(3,0),
    dec numeric(3,0)
);
```

Listing 13-19：生成温度读数交叉表

交叉表的结构与 清单 13-18 中相同。`crosstab()` 内部的第一个子查询生成交叉表的数据，查找每个月的中位数最高温度。它提供了三个必需的列。第一个列 `station_name` 1 命名了行。第二列使用了第十二章中的 `date_part()` 函数 2，从 `observation_date` 中提取月份，作为交叉表的列。然后我们使用 `percentile_cont(.5)` 3 来查找 `max_temp` 的第 50 个百分位数，即中位数。我们按照站点名称和月份进行分组，以便获得每个站点每个月的中位数 `max_temp`。

如 清单 13-18 中所示，第二个子查询生成列的类别名称集合。我使用一个叫做 `generate_series()` 的函数，按照官方 PostgreSQL 文档中的方式创建从 1 到 12 的数字列表，这些数字与 `date_part()` 从 `observation_date` 提取的月份数字相匹配。

在 `AS` 后，我们提供了交叉表输出列的名称和数据类型。每列都是 `numeric` 类型，匹配分位数函数的输出。以下输出几乎如同诗篇：

```
station                          jan  feb  mar  apr  may  jun  jul  aug  sep  oct  nov  dec
------------------------------   ---  ---  ---  ---  ---  ---  ---  ---  ---  ---  ---  ---
CHICAGO NORTHERLY ISLAND IL US    34   36   46   50   66   77   81   80   77   65   57   35
SEATTLE BOEING FIELD WA US        50   54   56   64   66   71   76   77   69   62   55   42
WAIKIKI 717.2 HI US               83   84   84   86   87   87   88   87   87   86   84   82
```

我们将一组原始的每日读数转化为一个简洁的表格，展示每个站点每月的中位数最高温度。一眼看去，我们可以发现怀基基的温度始终如春，而芝加哥的中位数最高温度则从接近冰点到相当宜人不等。西雅图的温度介于两者之间。

设置交叉表确实需要时间，但将数据集以矩阵的形式展示，往往比以垂直列表的形式查看相同数据更容易进行比较。请记住，`crosstab()` 函数是资源密集型的，因此在查询包含百万或十亿行的集合时要小心。

## 使用 `CASE` 重新分类值

ANSI 标准 SQL 的 `CASE` 语句是一个 *条件表达式*，意味着它允许你在查询中添加“如果这样，则……”的逻辑。你可以以多种方式使用 `CASE`，但对于数据分析而言，它非常方便，用于将值重新分类为不同类别。你可以根据数据中的范围创建类别，并根据这些类别对值进行分类。

`CASE` 语法遵循以下模式：

```
1 CASE WHEN `condition` THEN `result`
   2 WHEN `another_condition` THEN `result`
   3 ELSE `result`
4 END
```

我们给 `CASE` 关键字一个值 1，然后提供至少一个 `WHEN` `condition` `THEN` `result` 子句，其中 `condition` 是任何数据库可以评估为 `true` 或 `false` 的表达式，例如 `county = 'Dutchess County'` 或 `date > '1995-08-09'`。如果条件为 `true`，则 `CASE` 语句返回 `result` 并停止检查任何进一步的条件。结果可以是任何有效的数据类型。如果条件为 `false`，则数据库继续评估下一个条件。

为了评估更多条件，我们可以添加可选的 `WHEN ... THEN` 子句 2。我们还可以提供一个可选的 `ELSE` 子句 3，以在没有条件为 `true` 时返回一个结果。如果没有 `ELSE` 子句，当没有条件为 `true` 时，语句将返回 `NULL`。语句以 `END` 关键字 4 结束。

清单 13-20 显示了如何使用 `CASE` 语句将温度读数重新分类为描述性分组（这些分组根据我自己对寒冷天气的偏见来命名）。

```
SELECT max_temp,
       CASE WHEN max_temp >= 90 THEN 'Hot'
            WHEN max_temp >= 70 AND max_temp < 90 THEN 'Warm'
            WHEN max_temp >= 50 AND max_temp < 70 THEN 'Pleasant'
            WHEN max_temp >= 33 AND max_temp < 50 THEN 'Cold'
            WHEN max_temp >= 20 AND max_temp < 33 THEN 'Frigid'
            WHEN max_temp < 20 THEN 'Inhumane'
            ELSE 'No reading'
        END AS temperature_group
FROM temperature_readings
ORDER BY station_name, observation_date;
```

清单 13-20：使用 `CASE` 重新分类温度数据

我们为 `temperature_readings` 中的 `max_temp` 列创建了六个范围，通过比较运算符来定义。`CASE` 语句会评估每个值，检查六个表达式中是否有任何一个为 `true`。如果是，语句会输出相应的文本。请注意，这些范围涵盖了列中的所有可能值，没有任何遗漏。如果没有任何条件为 `true`，则 `ELSE` 子句将值分配到 `No reading` 类别。

运行代码；输出的前五行应该如下所示：

```
max_temp    temperature_group
--------    -----------------
      31    Frigid
      34    Cold
      32    Frigid
      32    Frigid
      34    Cold
      `--snip--`
```

现在我们已经将数据集压缩成六个类别，让我们用这些类别来比较表中三个城市的气候。

## 在公用表表达式中使用 `CASE`

我们在上一节中使用 `CASE` 对温度数据进行的操作是一个很好的示例，展示了你可以在 CTE 中使用的预处理步骤。现在我们已经将温度分组，接下来让我们在 CTE 中按城市统计这些分组，看看每个温度类别包含了多少天。

清单 13-21 显示了重新分类日最高温度的代码，重新生成 `temps_collapsed` CTE 并将其用于分析。

```
1 WITH temps_collapsed (station_name, max_temperature_group) AS
    (SELECT station_name,
           CASE WHEN max_temp >= 90 THEN 'Hot'
                WHEN max_temp >= 70 AND max_temp < 90 THEN 'Warm'
                WHEN max_temp >= 50 AND max_temp < 70 THEN 'Pleasant'
                WHEN max_temp >= 33 AND max_temp < 50 THEN 'Cold'
                WHEN max_temp >= 20 AND max_temp < 33 THEN 'Frigid'
                WHEN max_temp < 20 THEN 'Inhumane'
                ELSE 'No reading'
            END
     FROM temperature_readings)

2 SELECT station_name, max_temperature_group, count(*)
FROM temps_collapsed
GROUP BY station_name, max_temperature_group
ORDER BY station_name, count(*) DESC;
```

清单 13-21：在 CTE 中使用 `CASE`

这段代码重新分类了温度，然后按站点名称进行计数和分组，以找到每个城市的一般气候分类。`WITH` 关键字定义了 `temps_collapsed` CTE 1，它有两列：`station_name` 和 `max_temperature_group`。然后我们在 CTE 2 上运行 `SELECT` 查询，执行简单的 `count(*)` 和 `GROUP BY` 操作。结果应该如下所示：

```
station_name                      max_temperature_group    count
------------------------------    ---------------------    -----
CHICAGO NORTHERLY ISLAND IL US    Warm                       133
CHICAGO NORTHERLY ISLAND IL US    Cold                        92
CHICAGO NORTHERLY ISLAND IL US    Pleasant                    91
CHICAGO NORTHERLY ISLAND IL US    Frigid                      30
CHICAGO NORTHERLY ISLAND IL US    Inhumane                     8
CHICAGO NORTHERLY ISLAND IL US    Hot                          8
SEATTLE BOEING FIELD WA US        Pleasant                   198
SEATTLE BOEING FIELD WA US        Warm                        98
SEATTLE BOEING FIELD WA US        Cold                        50
SEATTLE BOEING FIELD WA US        Hot                          3
WAIKIKI 717.2 HI US               Warm                       361
WAIKIKI 717.2 HI US               Hot                          5
```

使用这一分类方案，令人惊讶的一致的威基基天气，361 天的 `Warm` 最高气温，证明了它作为度假胜地的吸引力。从温度角度来看，西雅图也不错，几乎有 300 天的 `Pleasant` 或 `Warm` 高温（尽管这与西雅图传奇般的降雨量相矛盾）。芝加哥有 30 天的 `Frigid` 最高气温和 8 天的 `Inhumane` 最高气温，可能不适合我。

## 总结

在本章中，你学习了如何让查询更好地为你服务。你现在可以在多个位置添加子查询，从而在分析主查询之前对数据进行更精细的过滤或预处理。你还可以使用交叉制表法将数据可视化为矩阵，并将数据重新分类为不同的组；这两种技术都为你提供了更多使用数据来发现和讲述故事的方法。干得不错！

在接下来的章节中，我们将深入探讨更多专门针对 PostgreSQL 的 SQL 技巧。我们将从处理和搜索文本以及字符串开始。
