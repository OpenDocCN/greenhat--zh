# 第九章：通过分组和汇总提取信息

![](img/chapterart.png)

每个数据集都在讲述一个故事，而数据分析师的工作就是找到这个故事。在第三章中，你学习了如何通过使用 `SELECT` 语句对列进行排序、查找不同的值以及筛选结果来“采访”数据。你还学到了 SQL 数学、数据类型、表格设计和连接表格的基础知识。有了这些工具，你已经准备好通过使用 *分组* 和 *聚合函数* 来总结数据，获取更多的见解。

通过汇总数据，我们可以识别出仅通过扫描表格的行无法看到的有用信息。在本章中，我们将使用你当地图书馆这一大家熟悉的机构作为示例。

图书馆仍然是全球社区的重要组成部分，但互联网和图书馆技术的进步已经改变了我们使用图书馆的方式。例如，电子书和在线访问数字资料现在与书籍和期刊一起，在图书馆中占据了永久位置。

在美国，博物馆与图书馆服务研究所（IMLS）将图书馆活动作为其年度公共图书馆调查的一部分进行测量。该调查收集来自大约 9,000 个图书馆行政机构的数据，调查定义这些机构为提供图书馆服务给特定地区的单位。一些机构是县级图书馆系统，其他一些则是学校区的一部分。每个机构的数据包括分馆数量、员工数、书籍数量、每年开放小时数等。自 1988 年以来，IMLS 每年都会收集数据，并涵盖所有 50 个州的公共图书馆机构，包括哥伦比亚特区和美国领土，如美属萨摩亚。（阅读更多关于该项目的信息：[`www.imls.gov/research-evaluation/data-collection/public-libraries-survey/`](https://www.imls.gov/research-evaluation/data-collection/public-libraries-survey/)）

对于本次练习，我们将假设自己是一个分析师，刚刚收到了一份新的图书馆数据集，目的是根据数据生成描述趋势的报告。我们将创建三个表格，分别保存 2018 年、2017 年和 2016 年调查的数据。（通常，评估多个年份的数据有助于发现趋势。）接着，我们会对每个表格中更有趣的数据进行汇总，并将这些表格连接起来，看看各项指标如何随时间变化。

## 创建图书馆调查表

让我们创建三个图书馆调查表并导入数据。我们将为每个列使用适当的数据类型和约束，并在适当的位置添加索引。代码和三个 CSV 文件可以在本书的资源中找到。

### 创建 2018 年图书馆数据表

我们将从创建 2018 年图书馆数据的表开始。使用`CREATE TABLE`语句，列表 9-1 构建了`pls_fy2018_libraries`，这是来自公共图书馆调查的 2018 财年公共图书馆系统数据文件。公共图书馆系统数据文件总结了在机构级别的数据，统计所有机构网点的活动，包括中央图书馆、分馆和流动图书馆。年度调查生成两个额外的文件我们不会使用：一个总结州级数据，另一个包含各个网点的数据。对于本练习来说，这些文件是冗余的，但你可以在[`www.imls.gov/sites/default/files/2018_pls_data_file_documentation.pdf`](https://www.imls.gov/sites/default/files/2018_pls_data_file_documentation.pdf)阅读它们包含的数据。

为了方便，我为表格创建了命名方案：`pls`代表调查标题，`fy2018`是数据覆盖的财年，`libraries`是调查中某个特定文件的名称。为了简便起见，我从原始调查文件中的 166 列中选择了 47 个更相关的列填充`pls_fy2018_libraries`表，排除了诸如解释个别响应来源的代码等数据。当某个图书馆没有提供数据时，机构使用其他方法推导数据，但我们在本次练习中不需要这些信息。

列表 9-1 为了方便进行了缩写，代码中标注了`--snip--`，但完整版本已经包含在本书的资源中。

```
CREATE TABLE pls_fy2018_libraries (
    stabr text NOT NULL,
    1 fscskey text CONSTRAINT fscskey_2018_pkey PRIMARY KEY,
    libid text NOT NULL,
    libname text NOT NULL,
    address text NOT NULL,
    city text NOT NULL,
    zip text NOT NULL,
 `--snip--`
    longitude numeric(10,7) NOT NULL,
    latitude numeric(10,7) NOT NULL
);

2 COPY pls_fy2018_libraries
FROM '`C:\YourDirectory\`pls_fy2018_libraries.csv'
WITH (FORMAT CSV, HEADER);

3 CREATE INDEX libname_2018_idx ON pls_fy2018_libraries (libname);
```

列表 9-1：创建并填充 2018 年公共图书馆调查表

找到列表 9-1 的代码和数据文件后，连接到你的`analysis`数据库，在 pgAdmin 中运行它。确保记得将`C:\YourDirectory\`改为你保存*pls_fy2018_libraries.csv*文件的路径。

首先，代码通过`CREATE TABLE`语句创建表。我们将一个名为`fscskey`的列赋予主键约束 1，这是数据字典中为每个图书馆分配的唯一代码。因为它是唯一的、出现在每一行，并且不太可能改变，所以可以作为自然主键。

每个列的定义包括适当的数据类型和`NOT NULL`约束条件，其中列没有缺失值。`startdate`和`enddate`列包含日期，但我们在代码中将其数据类型设置为`text`；在 CSV 文件中，这些列包含非日期值，如果我们尝试使用`date`数据类型导入，导入将失败。在第十章，你将学习如何清理像这样的情况。目前，这些列的设置已经可以使用。

创建表后，`COPY`语句 2 从名为*pls_fy2018_libraries.csv*的 CSV 文件中导入数据，使用你提供的文件路径。我们为`libname`列添加了索引 3，以便在搜索特定图书馆时提供更快的结果。

### 创建 2017 年和 2016 年图书馆数据表

创建 2017 年和 2016 年图书馆调查的表格遵循类似的步骤。我将创建和填充这两个表格的代码合并在了清单 9-2 中。再次提醒，所示的清单是截断的，但完整的代码可以在本书的资源页面找到：[`nostarch.com/practical-sql-2nd-edition/`](https://nostarch.com/practical-sql-2nd-edition/)。

更新`COPY`语句中的文件路径以导入数据，并执行代码。

```
CREATE TABLE pls_fy2017_libraries (
    stabr text NOT NULL,
    1 fscskey text CONSTRAINT fscskey_17_pkey PRIMARY KEY,
 libid text NOT NULL,
    libname text NOT NULL,
    address text NOT NULL,
    city text NOT NULL,
    zip text NOT NULL,
 `--snip--`
    longitude numeric(10,7) NOT NULL,
    latitude numeric(10,7) NOT NULL
);

CREATE TABLE pls_fy2016_libraries (
    stabr text NOT NULL,
    fscskey text CONSTRAINT fscskey_16_pkey PRIMARY KEY,
    libid text NOT NULL,
    libname text NOT NULL,
    address text NOT NULL,
    city text NOT NULL,
    zip text NOT NULL,
 `--snip--`
    longitude numeric(10,7) NOT NULL,
    latitude numeric(10,7) NOT NULL
);

2 COPY pls_fy2017_libraries
FROM '*C:\YourDirectory\*pls_fy2017_libraries.csv'
WITH (FORMAT CSV, HEADER);

COPY pls_fy2016_libraries
FROM '*C:\YourDirectory\*pls_fy2016_libraries.csv'
WITH (FORMAT CSV, HEADER);

3 CREATE INDEX libname_2017_idx ON pls_fy2017_libraries (libname);
CREATE INDEX libname_2016_idx ON pls_fy2016_libraries (libname);
```

清单 9-2：创建并填充 2017 年和 2016 年公共图书馆调查表格

我们首先创建两个表，在这两个表中，我们再次使用`fscskey` 1 作为主键。接下来，我们运行`COPY`命令导入 CSV 文件到表中，最后，我们在两个表的`libname`列上创建索引。

在查看代码时，你会注意到这三个表格具有相同的结构。大多数正在进行的调查每年都会有一些变化，因为调查的制定者要么提出新的问题，要么修改现有问题，但我为这三个表格选择的列是一致的。调查年份的文档可以在[`www.imls.gov/research-evaluation/data-collection/public-libraries-survey/`](https://www.imls.gov/research-evaluation/data-collection/public-libraries-survey/)找到。现在，让我们挖掘这些数据，发现它们的故事。

## 使用聚合函数探索图书馆数据

聚合函数将来自多行的值组合在一起，对这些值执行操作，并返回单一的结果。例如，你可能会使用`avg()`聚合函数返回值的平均值，正如你在第六章中学到的那样。一些聚合函数是 SQL 标准的一部分，而其他一些则特定于 PostgreSQL 和其他数据库管理系统。本章中使用的大多数聚合函数都是 SQL 标准的一部分（PostgreSQL 聚合函数的完整列表请参见：[`www.postgresql.org/docs/current/functions-aggregate.html`](https://www.postgresql.org/docs/current/functions-aggregate.html)）。

在本节中，我们将通过对单列和多列使用聚合函数来处理图书馆数据，然后探索如何通过将返回结果与其他列的值分组，来扩展这些聚合函数的应用。

### 使用 count()函数计数行数和数值

在导入数据集后，合理的第一步是确保表格中有预期的行数。IMLS 文档中说明我们导入的 2018 年数据文件有 9,261 行；2017 年有 9,245 行；2016 年有 9,252 行。这个差异很可能反映了图书馆的开设、关闭或合并。当我们计算这些表格中的行数时，结果应该与这些数字匹配。

`count()`聚合函数是 ANSI SQL 标准的一部分，它使得统计行数和执行其他计数任务变得非常简单。如果我们提供一个星号作为输入，例如`count(*)`，那么星号将充当通配符，因此该函数返回表格行数，无论这些行是否包含`NULL`值。在清单 9-3 中的三个语句都是这样做的。

```
SELECT count(*)
FROM pls_fy2018_libraries;

SELECT count(*)
FROM pls_fy2017_libraries;

SELECT count(*)
FROM pls_fy2016_libraries;
```

清单 9-3：使用`count(``)`统计表格行数

分别运行清单 9-3 中的每个命令，以查看表格行数。对于`pls_fy2018_libraries`，结果应如下：

```
count
-----
 9261
```

对于`pls_fy2017_libraries`，你应该看到以下结果：

```
count
-----
 9245
```

最终，`pls_fy2016_libraries`的结果应如下所示：

```
count
-----
 9252
```

三个结果都与我们预期的行数相符。这是一个良好的第一步，因为它可以提醒我们是否存在缺失行或可能导入了错误的文件等问题。

#### 统计列中出现的值

如果我们向`count()`提供一个列名而不是星号，它将返回非`NULL`行的数量。例如，我们可以使用`count()`来统计`pls_fy2018_libraries`表格中`phone`列的非`NULL`值数量，如清单 9-4 所示。

```
SELECT count(phone)
FROM pls_fy2018_libraries;
```

清单 9-4：使用`count(``)`统计列中值的数量

结果显示有 9,261 行在`phone`列中有值，这与我们之前找到的总行数相同。

```
count
-----
 9261
```

这意味着`phone`列中的每一行都有值。你可能已经猜到这一点，因为该列在`CREATE TABLE`语句中有`NOT NULL`约束。但进行这个检查是值得的，因为值的缺失可能会影响你是否继续进行分析的决定。为了全面验证数据，通常检查领域专家并深入挖掘数据是一个好主意；我建议将寻求专家意见作为更广泛分析方法的一部分（有关此主题的更多内容，请参见第二十章）。

#### 统计列中不同值的数量

在第三章中，我讲解了`DISTINCT`关键字——这是 SQL 标准的一部分——它与`SELECT`一起使用时返回唯一值的列表。我们可以用它来查看单列中的唯一值，或者查看多列中值的唯一组合。我们也可以将`DISTINCT`添加到`count()`函数中，来返回某列中不同值的计数。

清单 9-5 展示了两个查询。第一个查询统计 2018 表格中`libname`列的所有值。第二个查询也做相同的事情，但在列名前加上了`DISTINCT`。请分别运行这两个查询。

```
SELECT count(libname)
FROM pls_fy2018_libraries;

SELECT count(DISTINCT libname)
FROM pls_fy2018_libraries;
```

清单 9-5：使用`count(``)`统计列中不同值的数量

第一个查询返回的行数与我们通过清单 9-3 找到的表格行数相匹配：

```
count
-----
 9261
```

这很好。我们预计每一行都会列出图书馆机构的名称。但是第二个查询返回的结果更小：

```
count
-----
 8478
```

使用 `DISTINCT` 去除重复项，将图书馆名称的数量减少到唯一的 8,478 个。对数据的进一步检查显示，2018 年调查中有 526 个图书馆机构与一个或多个其他机构共享名称。十个图书馆机构都被命名为 `OXFORD PUBLIC LIBRARY`，每个都位于不同州的名为 Oxford 的城市或城镇，包括阿拉巴马州、康涅狄格州、堪萨斯州和宾夕法尼亚州等。我们将编写查询，以查看“使用 `GROUP BY` 聚合数据”部分中的不同值组合。

### 使用 `max()` 和 `min()` 查找最大值和最小值

`max()` 和 `min()` 函数分别给我们返回一列中的最大值和最小值，这些函数有几个用途。首先，它们帮助我们了解报告值的范围。其次，这些函数能揭示数据中的意外问题，正如你现在将看到的。

`max()` 和 `min()` 都是以列名作为输入，工作方式相同。 列表 9-6 在 2018 表中使用 `max()` 和 `min()`，它使用 `visits` 列，记录了每年对图书馆机构及其所有分支的访问次数。运行代码。

```
SELECT max(visits), min(visits)
FROM pls_fy2018_libraries;
```

列表 9-6：使用 `max()` 和 `min()` 查找最多和最少的访问量

查询返回以下结果：

```
max         min
--------    ---
16686945     -3
```

哦，这很有趣。超过 1,660 万的最大值对一个大城市的图书馆系统来说是合理的，但最小值为 `-3` 呢？表面上看，这个结果似乎是一个错误，但事实证明，图书馆调查的创建者采用了一种常见但可能存在问题的数据收集惯例，即在某列中放入负数或某个人为的高值，以表示某种特定条件。

在这种情况下，数字列中的负值表示以下内容：

1.  `-1` 的值表示对该问题的“未响应”。

1.  `-3` 的值表示“不可应用”，当一个图书馆机构暂时或永久关闭时，会使用这个值。

在探索数据时，我们需要考虑并排除负值，因为将负值包含在列的求和中会导致总和错误。我们可以使用 `WHERE` 子句来过滤这些负值。这也提醒我们，在深入分析前，始终先阅读数据的文档，以提前发现问题，而不是在投入大量时间分析后才后退修正！

### 使用 `GROUP BY` 聚合数据

当你使用带有聚合函数的 `GROUP BY` 子句时，可以根据一个或多个列中的值对结果进行分组。这使我们可以对表中的每个州或每种图书馆机构类型执行如 `sum()` 或 `count()` 等操作。

让我们探讨一下如何使用带有聚合函数的 `GROUP BY`。单独使用 `GROUP BY`（这也是标准 ANSI SQL 的一部分）会从结果中去除重复值，类似于 `DISTINCT`。 列表 9-7 显示了 `GROUP BY` 子句的实际应用。

```
SELECT stabr
FROM pls_fy2018_libraries
1 GROUP BY stabr
ORDER BY stabr;
```

列表 9-7：对 `stabr` 列使用 `GROUP BY`

我们在`FROM`子句后添加`GROUP BY`子句 1，并包括要分组的列名。在这种情况下，我们选择`stabr`，它包含州的缩写，并按该列进行分组。然后，我们使用`ORDER BY` `stabr`，这样分组后的结果将按字母顺序排列。这样就会得到 2018 年表格中唯一的州缩写。以下是部分结果：

```` ``` stabr  -----  AK  AL  AR  AS  AZ  CA  `--snip--`  WV  WY ```    Notice that there are no duplicates in the 55 rows returned. These standard two-letter postal abbreviations include the 50 states plus Washington, DC, and several US territories, such as Guam and the US Virgin Islands.    You’re not limited to grouping just one column. In Listing 9-8, we use the `GROUP BY` clause on the 2018 data to specify the `city` and `stabr` columns for grouping.    ``` SELECT city, stabr  FROM pls_fy2018_libraries  GROUP BY city, stabr  ORDER BY city, stabr; ```    Listing 9-8: Using `GROUP BY` on the `city` and `stabr` columns    The results get sorted by city and then by state, and the output shows unique combinations in that order:    ``` city          stabr  ----------    -----  ABBEVILLE     AL  ABBEVILLE     LA  ABBEVILLE     SC  ABBOTSFORD    WI  ABERDEEN      ID  ABERDEEN      SD  ABERNATHY     TX  `--snip--` ```    This grouping returns 9,013 rows, 248 fewer than the total table rows. The result indicates that the file includes multiple instances where there’s more than one library agency for a particular city and state combination.    #### Combining GROUP BY with count()    If we combine `GROUP BY` with an aggregate function, such as `count()`, we can pull more descriptive information from our data. For example, we know 9,261 library agencies are in the 2018 table. We can get a count of agencies by state and sort them to see which states have the most. Listing 9-9 shows how to do this.    ``` 1 SELECT stabr, count(*)  FROM pls_fy2018_libraries  2 GROUP BY stabr  3 ORDER BY count(*) DESC; ```    Listing 9-9: Using `GROUP BY` with `count(``)` on the `stabr` column    We’re now asking for the values in the `stabr` column and a count of how many rows have a given `stabr` value. In the list of columns to query 1, we specify `stabr` and `count()` with an asterisk as its input, which will cause `count()` to include `NULL` values. Also, when we select individual columns along with an aggregate function, we must include the columns in a `GROUP BY` clause 2. If we don’t, the database will return an error telling us to do so, because you can’t group values by aggregating and have ungrouped column values in the same query.    To sort the results and have the state with the largest number of agencies at the top, we can use an `ORDER BY` clause 3 that includes the `count()` function and the `DESC` keyword.    Run the code in Listing 9-9. The results show New York, Illinois, and Texas as the states with the greatest number of library agencies in 2018:    ``` stabr    count  -----    -----  NY         756  IL         623  TX         560  IA         544  PA         451  MI         398  WI         381  MA         369  `--snip--` ```    Remember that our table represents library agencies that serve a locality. Just because New York, Illinois, and Texas have the greatest number of library agencies doesn’t mean they have the greatest number of outlets where you can walk in and peruse the shelves. An agency might have one central library only, or it might have no central libraries but 23 branches spread around a county. To count outlets, each row in the table also has values in the columns `centlib` and `branlib`, which record the number of central and branch libraries, respectively. To find totals, we would use the `sum()` aggregate function on both columns.    #### Using GROUP BY on Multiple Columns with count()    We can glean yet more information from our data by combining `GROUP BY` with `count()` and multiple columns. For example, the `stataddr` column in all three tables contains a code indicating whether the agency’s address changed in the last year. The values in `stataddr` are as follows:    1.  00 No change from last year 2.  07 Moved to a new location 3.  15 Minor address change    Listing 9-10 shows the code for counting the number of agencies in each state that moved, had a minor address change, or had no change using `GROUP BY` with `stabr` and `stataddr` and adding `count()`.    ``` 1 SELECT stabr, stataddr, count(*)  FROM pls_fy2018_libraries  2 GROUP BY stabr, stataddr  3 ORDER BY stabr, stataddr; ```    Listing 9-10: Using `GROUP BY` with `count(``)` of the `stabr` and `stataddr` columns    The key sections of the query are the column names and the `count()` function after `SELECT` 1, and making sure both columns are reflected in the `GROUP BY` clause 2 to ensure that `count()` will show the number of unique combinations of `stabr` and `stataddr`.    To make the output easier to read, let’s sort first by the state and address status codes in ascending order 3. Here are the results:    ``` stabr    stataddr    count  -----    --------    -----  AK       00          82  AL       00         220  AL       07           3  AL       15           1  AR       00          58  AR       07           1  AR       15           1  AS       00           1  `--snip--` ```    The first few rows show that code `00` (no change in address) is the most common value for each state. We’d expect that because it’s likely there are more library agencies that haven’t changed address than those that have. The result helps assure us that we’re analyzing the data in a sound way. If code `07` (moved to a new location) was the most frequent in each state, that would raise a question about whether we’ve written the query correctly or whether there’s an issue with the data.    #### Revisiting sum() to Examine Library Activity    Now let’s expand our techniques to include grouping and aggregating across joined tables using the 2018, 2017, and 2016 libraries data. Our goal is to identify trends in library visits spanning that three-year period. To do this, we need to calculate totals using the `sum()` aggregate function.    Before we dig into these queries, let’s address the values `-3` and `-1`, which indicate “not applicable” and “nonresponse.” To prevent these negative numbers from affecting the analysis, we’ll filter them out using a `WHERE` clause to limit the queries to rows where values in `visits` are zero or greater.    Let’s start by calculating the sum of annual visits to libraries from the individual tables. Run each `SELECT` statement in Listing 9-11 separately.    ``` SELECT sum(visits) AS visits_2018  FROM pls_fy2018_libraries  WHERE visits >= 0;    SELECT sum(visits) AS visits_2017  FROM pls_fy2017_libraries  WHERE visits >= 0;    SELECT sum(visits) AS visits_2016  FROM pls_fy2016_libraries  WHERE visits >= 0; ```    Listing 9-11: Using the `sum(``)` aggregate function to total visits to libraries in 2016, 2017, and 2018    For 2018, visits totaled approximately 1.29 billion:    ``` visits_2018  -----------   1292348697 ```    For 2017, visits totaled approximately 1.32 billion:    ``` visits_2017  -----------   1319803999 ```    And for 2016, visits totaled approximately 1.36 billion:    ``` visits_2016  -----------   1355648987 ```    We’re onto something here, but it may not be good news for libraries. The trend seems to point downward with visits dropping about 5 percent from 2016 to 2018.    Let’s refine this approach. These queries sum visits recorded in each table. But from the row counts we ran earlier in the chapter, we know that each table contains a different number of library agencies: 9,261 in 2018; 9,245 in 2017; and 9,252 in 2016\. The differences are likely due to agencies opening, closing, or merging. So, let’s determine how the sum of visits will differ if we limit the analysis to library agencies that exist in all three tables and have a non-negative value for `visits`. We can do that by joining the tables, as shown in Listing 9-12.    ``` 1 SELECT sum(pls18.visits) AS visits_2018,         sum(pls17.visits) AS visits_2017,         sum(pls16.visits) AS visits_2016  2 FROM pls_fy2018_libraries pls18         JOIN pls_fy2017_libraries pls17 ON pls18.fscskey = pls17.fscskey         JOIN pls_fy2016_libraries pls16 ON pls18.fscskey = pls16.fscskey  3 WHERE pls18.visits >= 0         AND pls17.visits >= 0         AND pls16.visits >= 0; ```    Listing 9-12: Using `sum(``)` to total visits on joined 2018, 2017, and 2016 tables    This query pulls together a few concepts we covered in earlier chapters, including table joins. At the top, we use the `sum()` aggregate function 1 to total the `visits` columns from each of the three tables. When we join the tables on the tables’ primary keys, we’re declaring table aliases 2 as we explored in Chapter 7—and here, we’re omitting the optional `AS` keyword in front of each alias. For example, we declare `pls18` as the alias for the 2018 table to avoid having to write its lengthier full name throughout the query.    Note that we use a standard `JOIN`, also known as an `INNER JOIN`, meaning the query results will only include rows where the values in the `fscskey` primary key match in all three tables.    As we did in Listing 9-11, we specify with a `WHERE` clause 3 that the result should include only those rows where `visits` are greater than or equal to 0 in the tables. This will prevent the artificial negative values from impacting the sums.    Run the query. The results should look like this:    ``` visits_2018   visits_2017   visits_2016  -----------   -----------   -----------   1278148838    1319325387    1355078384 ```    The results are similar to what we found by querying the tables separately, although these totals are as much as 14 million smaller in 2018\. Still, the downward trend holds.    For a full picture of how library use is changing, we’d want to run a similar query on all of the columns that contain performance indicators to chronicle the trend in each. For example, the column `wifisess` shows how many times users connected to the library’s wireless internet. If we use `wifisess` instead of `visits` in Listing 9-11, we get this result:    ``` wifi_2018  wifi_2017  wifi_2016  ---------  ---------  ---------  349767271  311336231  234926102 ```    So, though visits were down, libraries saw a sharp increase in Wi-Fi network use. That provides a keen insight into how the role of libraries is changing.    #### Grouping Visit Sums by State    Now that we know library visits dropped for the United States as a whole between 2016 and 2018, you might ask yourself, “Did every part of the country see a decrease, or did the degree of the trend vary by region?” We can answer this question by modifying our preceding query to group by the state code. Let’s also use a percent-change calculation to compare the trend by state. Listing 9-13 contains the full code.    ``` 1 SELECT pls18.stabr,         sum(pls18.visits) AS visits_2018,         sum(pls17.visits) AS visits_2017,         sum(pls16.visits) AS visits_2016,         round( (sum(pls18.visits::numeric) - sum(pls17.visits)) /              2 sum(pls17.visits) * 100, 1 ) AS chg_2018_17,         round( (sum(pls17.visits::numeric) - sum(pls16.visits)) /              sum(pls16.visits) * 100, 1 ) AS chg_2017_16  FROM pls_fy2018_libraries pls18         JOIN pls_fy2017_libraries pls17 ON pls18.fscskey = pls17.fscskey         JOIN pls_fy2016_libraries pls16 ON pls18.fscskey = pls16.fscskey  WHERE pls18.visits >= 0         AND pls17.visits >= 0         AND pls16.visits >= 0  3 GROUP BY pls18.stabr  4 ORDER BY chg_2018_17 DESC; ```    Listing 9-13: Using `GROUP BY` to track percent change in library visits by state    We follow the `SELECT` keyword with the `stabr` column 1 from the 2018 table; that same column appears in the `GROUP BY` clause 3. It doesn’t matter which table’s `stabr` column we use because we’re only querying agencies that appear in all three tables. After the `visits` columns, we include the now-familiar percent-change calculation you learned in Chapter 6. We use this twice, giving the aliases `chg_2018_17` 2 and `chg_2017_16` for clarity. We end the query with an `ORDER BY` clause 4, sorting by the `chg_2018_17` column alias.    When you run the query, the top of the results shows 10 states with an increase in visits from 2017 to 2018\. The rest of the results show a decline. American Samoa, at the bottom of the ranking, had a 28 percent drop!    ``` stabr visits_2018 visits_2017 visits_2016 chg_2018_17 chg_2017_16  ----- ----------- ----------- ----------- ----------- -----------  SD        3824804     3699212     3722376         3.4        -0.6  MT        4332900     4215484     4298268         2.8        -1.9  FL       68423689    66697122    70991029         2.6        -6.0  ND        2216377     2162189     2201730         2.5        -1.8  ID        8179077     8029503     8597955         1.9        -6.6  DC        3632539     3593201     3930763         1.1        -8.6  ME        6746380     6731768     6811441         0.2        -1.2  NH        7045010     7028800     7236567         0.2        -2.9  UT       15326963    15295494    16096911         0.2        -5.0  DE        4122181     4117904     4125899         0.1        -0.2  OK       13399265    13491194    13112511        -0.7         2.9  WY        3338772     3367413     3536788        -0.9        -4.8  MA       39926583    40453003    40427356        -1.3         0.1  WA       37338635    37916034    38634499        -1.5        -1.9  MN       22952388    23326303    24033731        -1.6        -2.9  `--snip--`  GA       26835701    28816233    27987249        -6.9         3.0  AR        9551686    10358181    10596035        -7.8        -2.2  GU          75119       81572       71813        -7.9        13.6  MS        7602710     8581994     8915406       -11.4        -3.7  HI        3456131     4135229     4490320       -16.4        -7.9  AS          48828       67848       63166       -28.0         7.4 ```    It’s helpful, for context, to also see the percent change in `visits` from 2016 to 2017\. Many of the states, such as Minnesota, show consecutive declines. Others, including several at the top of the list, show gains after substantial decreases the year prior.    This is when it’s a good idea investigate what’s driving the changes. Data analysis can sometimes raise as many questions as it answers, but that’s part of the process. It’s always worth a phone call to a person who works closely with the data to review your findings. Sometimes, they’ll have a good explanation. Other times, an expert will say, “That doesn’t sound right.” That answer might send you back to the keeper of the data or the documentation to find out if you overlooked a code or a nuance with the data.    #### Filtering an Aggregate Query Using HAVING    To refine our analysis, we can examine a subset of states and territories that share similar characteristics. With percent change in visits, it makes sense to separate large states from small states. In a small state like Rhode Island, a single library closing for six months for repairs could have a significant effect. A single closure in California might be scarcely noticed in a statewide count. To look at states with a similar volume in visits, we could sort the results by either of the `visits` columns, but it would be cleaner to get a smaller result set by filtering our query.    To filter the results of aggregate functions, we need to use the `HAVING` clause that’s part of standard ANSI SQL. You’re already familiar with using `WHERE` for filtering, but aggregate functions, such as `sum()`, can’t be used within a `WHERE` clause because they operate at the row level, and aggregate functions work across rows. The `HAVING` clause places conditions on groups created by aggregating. The code in Listing 9-14 modifies the query in Listing 9-13 by inserting the `HAVING` clause after `GROUP BY`.    ``` SELECT pls18.stabr,         sum(pls18.visits) AS visits_2018,         sum(pls17.visits) AS visits_2017,         sum(pls16.visits) AS visits_2016,         round( (sum(pls18.visits::numeric) - sum(pls17.visits)) /   sum(pls17.visits) * 100, 1 ) AS chg_2018_17,         round( (sum(pls17.visits::numeric) - sum(pls16.visits)) /              sum(pls16.visits) * 100, 1 ) AS chg_2017_16  FROM pls_fy2018_libraries pls18         JOIN pls_fy2017_libraries pls17 ON pls18.fscskey = pls17.fscskey         JOIN pls_fy2016_libraries pls16 ON pls18.fscskey = pls16.fscskey  WHERE pls18.visits >= 0         AND pls17.visits >= 0         AND pls16.visits >= 0  GROUP BY pls18.stabr  1 HAVING sum(pls18.visits) > 50000000  ORDER BY chg_2018_17 DESC; ```    Listing 9-14: Using a `HAVING` clause to filter the results of an aggregate query    In this case, we’ve set our query results to include only rows with a sum of visits in 2018 greater than 50 million. That’s an arbitrary value I chose to show only the very largest states. Adding the `HAVING` clause 1 reduces the number of rows in the output to just six. In practice, you might experiment with various values. Here are the results:    ``` stabr visits_2018 visits_2017 visits_2016 chg_2018_17 chg_2017_16  ----- ----------- ----------- ----------- ----------- -----------  FL       68423689    66697122    70991029         2.6        -6.0  NY       97921323   100012193   103081304        -2.1        -3.0  CA      146656984   151056672   155613529        -2.9        -2.9  IL       63466887    66166082    67336230        -4.1        -1.7  OH       68176967    71895854    74119719        -5.2        -3.0  TX       66168387    70514138    70975901        -6.2        -0.7 ```    All but one of the six states experienced a decline in visits, but notice that the percent-change variation isn’t as wide as in the full set of states and territories. Depending on what we learn from library experts, looking at the states with the most activity as a group might be helpful in describing trends, as would looking at other groupings. Think of a sentence you might write that would say, “Among states with the most library visits, Florida was the only one to see an increase in activity between 2017 and 2018; the rest saw visits decrease between 2 percent and 6 percent.” You could write similar sentences about medium-sized states and small states.    ## Wrapping Up    If you’re now inspired to visit your local library and check out a couple of books, ask a librarian whether their branch has seen a rise or drop in visits over the last few years. You can probably guess the answer. In this chapter, you learned how to use standard SQL techniques to summarize data in a table by grouping values and using a handful of aggregate functions. By joining datasets, you were able to identify some interesting trends.    You also learned that data doesn’t always come perfectly packaged. The presence of negative values in columns, used as an indicator rather than as an actual numeric value, forced us to filter out those rows. Unfortunately, those sorts of challenges are part of the data analyst’s everyday world, so we’ll spend the next chapter learning how to clean up a dataset that has a number of issues. Later in the book, you’ll also discover more aggregate functions to help you find the stories in your data. ````
