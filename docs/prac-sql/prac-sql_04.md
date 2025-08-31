# 第四章：理解数据类型

![](img/chapterart.png)

理解数据类型非常重要，因为以合适的格式存储数据是构建可用数据库和进行准确分析的基础。每当我深入研究一个新数据库时，我都会检查每个表中每一列指定的 *数据类型*。如果幸运的话，我能找到一个 *数据字典*：这是一份列出每个列、指定它是数字、字符还是其他类型，并解释列值的文档。不幸的是，许多组织没有创建和维护良好的文档，因此常常会听到，“我们没有数据字典。”在这种情况下，我会在 pgAdmin 中检查表格结构，尽可能多地了解信息。

数据类型是一个编程概念，适用于不仅仅是 SQL。你将在本章中探讨的概念也适用于你可能想学习的其他编程语言。

在 SQL 数据库中，表格中的每一列只能存储一种数据类型，且必须在 `CREATE TABLE` 语句中通过在列名后声明数据类型来定义。在以下简单的示例表格中——你可以查看但无需创建——你会看到三个不同数据类型的列：一个日期、一个整数和一个文本。

```
CREATE TABLE eagle_watch (
    observation_date date,
    eagles_seen integer,
    notes text
);
```

在名为 `eagle_watch` 的表格中（假设是关于秃鹰的库存），我们通过在列名后添加 `date` 类型声明来声明 `observation_date` 列存储日期值。同样，我们通过 `integer` 类型声明将 `eagles_seen` 设置为存储整数，并通过 `text` 类型声明将 `notes` 设置为存储字符。

这些数据类型属于你最常遇到的三种类别：

1.  字符 任何字符或符号

1.  数字包括整数和分数

1.  日期和时间 时间信息

让我们深入了解每种数据类型；我会标注它们是否属于标准 ANSI SQL，或者是 PostgreSQL 特有的。关于 PostgreSQL 与 SQL 标准的差异，可以在 [`wiki.postgresql.org/wiki/PostgreSQL_vs_SQL_Standard`](https://wiki.postgresql.org/wiki/PostgreSQL_vs_SQL_Standard) 找到全面的深入分析。

## 理解字符

*字符字符串类型* 是通用类型，适用于任何文本、数字和符号的组合。字符类型包括以下几种：

**`char(``n``)`**

一个固定长度的列，其中字符长度由 `n` 指定。设置为 `char(20)` 的列每行存储 20 个字符，无论插入多少字符。如果某行插入的字符少于 20 个，PostgreSQL 会用空格填充该列的剩余部分。此类型是标准 SQL 的一部分，也可以使用更长的名称 `character(``n``)` 来指定。如今，`char(``n``)` 使用较少，主要是遗留计算机系统的产物。

**`varchar(``n``)`**

一个可变长度的列，其*最大*长度由 `n` 指定。如果插入的字符少于最大值，PostgreSQL 将不会存储额外的空格。例如，字符串 `blue` 将占用四个空间，而字符串 `123` 将占用三个空间。在大型数据库中，这种做法节省了大量空间。这种类型，标准 SQL 中也有，包括用更长的名称 `character varying(``n``)` 来指定。

**`text`**

一个长度不受限制的可变长度列。（根据 PostgreSQL 文档，你可以存储的最长字符串大约为 1 GB。）`text` 类型不是 SQL 标准的一部分，但你会在其他数据库系统中找到类似的实现，包括 Microsoft SQL Server 和 MySQL。

根据 PostgreSQL 文档，[`www.postgresql.org/docs/current/datatype-character.html`](https://www.postgresql.org/docs/current/datatype-character.html)，这三种类型在性能上没有实质性的区别。如果你使用的是其他数据库管理器，情况可能有所不同，因此最好检查相关文档。`varchar` 和 `text` 的灵活性及其潜在的空间节省似乎给它们带来了优势。但是，如果你在线查阅讨论，某些用户建议使用 `char` 来定义一个始终包含相同字符数的列，这样可以很好地指示它应包含的数据。例如，`char(2)` 可能用于美国州的邮政缩写。

要查看这三种字符类型的实际应用，可以运行 Listing 4-1 中显示的脚本。这个脚本将构建并加载一个简单的表格，然后将数据导出到你电脑上的一个文本文件。

```
CREATE TABLE char_data_types (
    1 char_column char(10),
    varchar_column varchar(10),
    text_column text
);

2 INSERT INTO char_data_types
VALUES
    ('abc', 'abc', 'abc'),
    ('defghi', 'defghi', 'defghi');

3 COPY char_data_types TO '`C:\YourDirectory\`typetest.txt'
4 WITH (FORMAT CSV, HEADER, DELIMITER '|');
```

Listing 4-1: 字符数据类型示例

我们定义了三种不同类型的字符列，并将相同的字符串插入到每一列的两行中。与第二章中学到的 `INSERT INTO` 语句不同，这里我们没有指定列的名称。如果 `VALUES` 语句中的值与表中列的数量相匹配，数据库将假定你按照列定义的顺序插入值。

接下来，我们使用 PostgreSQL 的 `COPY` 关键字将数据导出到一个名为 *typetest.txt* 的文本文件中，文件保存在你指定的目录里。你需要将 *C:\YourDirectory\* 替换为你电脑上要保存该文件的目录的完整路径。本书中的示例使用的是 Windows 格式——它在文件夹和文件名之间使用反斜杠——并且路径指向位于 C: 驱动器上的名为 *YourDirectory* 的目录。Windows 用户必须根据第一章中“从 GitHub 下载代码和数据”部分的说明设置目标文件夹的权限。

Linux 和 macOS 的文件路径格式不同，文件夹和文件名之间使用正斜杠。例如，在我的 Mac 上，桌面文件的路径是 */Users/anthony/Desktop/*。目录必须已经存在；PostgreSQL 不会为你创建它。

在 PostgreSQL 中，`COPY` `table_name` `FROM` 是导入功能，`COPY` `table_name` `TO` 是导出功能。我将在第五章中详细讲解它们；目前，你只需要知道的是，`WITH` 关键字选项 4 会将文件中的数据格式化，每列由一个 *管道符* (`|`) 分隔。这样，你可以轻松看到 `char` 列中空白部分填充的位置。

要查看输出，使用你在第一章中安装的文本编辑器打开 *typetest.txt*（而不是 Word、Excel 或其他电子表格应用程序）。其内容应如下所示：

```
char_column|varchar_column|text_column
abc       |abc|abc
defghi    |defghi|defghi
```

即使你为 `char` 和 `varchar` 列都指定了 10 个字符，只有 `char` 列会在两行中输出 10 个字符，并用空格填充未使用的字符。`varchar` 和 `text` 列仅存储你插入的字符。

再次强调，三种类型之间没有实际的性能差异，尽管这个例子显示了 `char` 可能会比实际需要的空间更多。每个列中的几个未使用的空格看似微不足道，但如果在数百万行的数十个表中进行相同的操作，很快你就会希望自己当初更节省空间。

我倾向于在所有字符列中使用 `text`。这样可以避免为多个 `varchar` 列配置最大长度，也意味着如果字符列的要求发生变化，我以后不需要修改表。

## 理解数字

数字列包含各种类型的（你猜对了）数字，但不仅仅如此：它们还允许你对这些数字进行计算。这与将数字存储为字符列中的字符串有所不同，因为字符列中的字符串无法进行加法、乘法、除法或执行任何其他数学操作。此外，作为字符存储的数字排序方式不同于作为数字存储的数字，因此，如果你要进行数学运算或数字顺序很重要，应该使用数字类型。

SQL 数字类型包括以下几种：

1.  整数 既包括正数，也包括负数的整数

1.  定点数和浮点数 两种表示小数的格式

我们将分别查看每种类型。

### 使用整数

整数数据类型是你在 SQL 数据库中最常见的数字类型。这些是 *整数*，包括正数、负数和零。想想生活中所有出现整数的地方：你的街道或公寓号码、冰箱上的序列号、彩票上的号码。

SQL 标准提供了三种整数类型：`smallint`、`integer` 和 `bigint`。这三种类型的区别在于它们能够存储的数字的最大大小。表 4-1 显示了每种类型的上下限，以及它们各自所需的存储空间（以字节为单位）。

表 4-1：整数数据类型

| **数据类型** | **存储大小** | **范围** |
| --- | --- | --- |
| `smallint` | 2 字节 | −32768 到 +32767 |
| `integer` | 4 字节 | −2147483648 到 +2147483647 |
| `bigint` | 8 字节 | −9223372036854775808 到 +9223372036854775807 |

`bigint`类型几乎可以满足你所有的数字列需求，尽管它占用的存储空间最大。如果你处理的是超过约 21 亿的数字，它是必须使用的类型，但你也可以轻松将其设为默认类型，永远不必担心数字无法存入该列。另一方面，如果你确定数字将保持在`integer`的限制范围内，那么选择该类型是一个不错的选择，因为它不像`bigint`那样消耗空间（尤其在处理百万级数据行时，这是一个重要的考量）。

当你知道值会保持在某个范围内时，`smallint`是合适的选择：例如月份的天数或年份。`smallint`类型的存储空间是`integer`的一半，因此如果列的值始终适合其范围，选择它是一个明智的数据库设计决策。

如果你尝试向这些列中插入超出其范围的数字，数据库将停止操作并返回`超出范围`的错误。

### 自增整数

有时候，创建一个每次向表中添加行时都会*自动递增*的整数列是很有帮助的。例如，你可以使用自增列为表中的每一行创建一个唯一的 ID 号码，也称为*主键*。这样每一行都会有自己的 ID，其他表可以引用该 ID，这一概念我将在第七章中讲解。

在 PostgreSQL 中，你可以通过两种方式实现整数列的自动递增。一种是*serial*数据类型，这是 PostgreSQL 特有的实现，符合 ANSI SQL 标准中的自动编号*标识符列*。另一种是 ANSI SQL 标准中的`IDENTITY`关键字。我们先从 serial 开始。

#### 使用 serial 实现自增

在第二章中，当你创建`teachers`表时，你创建了一个`id`列并声明了`bigserial`类型：这个和它的兄弟类型`smallserial`、`serial`并不完全是独立的数据类型，而是对应的`smallint`、`integer`和`bigint`类型的特殊*实现*。当你添加一个自增列时，PostgreSQL 会在每次插入行时自动递增该列的值，从 1 开始，直到每个整数类型的最大值。

表 4-2 展示了自增类型及其覆盖的范围。

表 4-2：序列数据类型

| **数据类型** | **存储大小** | **范围** |
| --- | --- | --- |
| `smallserial` | 2 字节 | 1 到 32767 |
| `serial` | 4 字节 | 1 到 2147483647 |
| `bigserial` | 8 字节 | 1 到 9223372036854775807 |

要在列上使用序列类型，可以像声明整数类型一样，在`CREATE TABLE`语句中声明它。例如，你可以创建一个名为`people`的表，其中有一个`id`列，大小与`integer`数据类型相当：

```
CREATE TABLE people (
    id serial,
    person_name varchar(100)
);
```

每次向表中添加一行具有`person_name`的记录时，`id`列的值将递增 1。

#### 使用 IDENTITY 实现自增

从版本 10 开始，PostgreSQL 支持`IDENTITY`，这是标准 SQL 中用于自动递增整数的实现。`IDENTITY`语法较为冗长，但一些数据库用户更倾向于使用它，因为它与其他数据库系统（如 Oracle）具有跨平台兼容性，并且还提供了防止用户意外插入自动递增列值的选项（而序列类型则允许这种操作）。

你可以通过两种方式指定`IDENTITY`：

1.  `GENERATED ALWAYS AS IDENTITY`告诉数据库始终用自动递增的值填充该列。用户不能插入一个值到`id`列中，除非手动覆盖该设置。详细信息请参见 PostgreSQL `INSERT`文档中的`OVERRIDING SYSTEM VALUE`部分，链接地址：[`www.postgresql.org/docs/current/sql-insert.html`](https://www.postgresql.org/docs/current/sql-insert.html)。

1.  `GENERATED BY DEFAULT AS IDENTITY`告诉数据库，如果用户未提供值，则默认用自动递增的值填充该列。此选项允许出现重复值，这可能会使其在创建键列时变得具有问题。我将在第七章详细讨论这一点。

目前，我们将坚持使用第一种方式，即使用`ALWAYS`。要创建一个名为`people`的表，并通过`IDENTITY`填充`id`列，你可以使用以下语法：

```
CREATE TABLE people (
    id integer GENERATED ALWAYS AS IDENTITY,
    person_name varchar(100)
);
```

对于`id`数据类型，我们使用`integer`，后跟关键词`GENERATED ALWAYS AS IDENTITY`。现在，每次我们将`person_name`值插入表中时，数据库会自动为`id`列填充递增的值。

由于它与 ANSI SQL 标准的兼容性，我将在本书的其余部分使用`IDENTITY`。

### 使用十进制数

*十进制*表示一个整数加上一个整数的分数部分；该分数部分由*小数点*后的数字表示。在 SQL 数据库中，它们通过*定点*和*浮点*数据类型进行处理。例如，从我家到最近的杂货店的距离是 6.7 英里；我可以将 6.7 插入定点或浮点列，PostgreSQL 都不会有任何投诉。唯一的区别是计算机存储数据的方式。稍后，你将看到这有重要的含义。

#### 理解定点数

定点类型，也叫做*任意精度*类型，表示为`numeric(``precision``,``scale``)`。你需要将参数`precision`指定为小数点左右的最大位数，参数`scale`则表示小数点右侧允许的位数。或者，你也可以使用`decimal(``precision``,``scale``)`来指定这种类型。两者都是 ANSI SQL 标准的一部分。如果你省略了`scale`值的指定，默认会设置为零；实际上，这样会创建一个整数。如果你省略了`precision`和`scale`的指定，数据库将存储任何精度和范围的值，直到最大值为止。（根据 PostgreSQL 文档[`www.postgresql.org/docs/current/datatype-numeric.html`](https://www.postgresql.org/docs/current/datatype-numeric.html)，这最大可以是小数点前 131,072 位，后面 16,383 位。）

例如，假设你正在收集来自几个当地机场的降水数据——这并不是一个不常见的数据分析任务。美国国家气象局提供的数据通常将降水量测量到小数点后两位。（如果你像我一样，可能还记得小学数学老师讲解过小数点后两位是百分位。）

要在数据库中记录降水量，总共使用五位数字（精度）并且小数点后最多两位（范围），你可以指定为`numeric(5,2)`。即使你输入的数字没有包含两位小数，如 1.47、1.00 和 121.50，数据库也会始终返回小数点后两位。

#### 理解浮动类型

两种浮动类型是`real`和`double precision`，这两者都属于 SQL 标准的一部分。它们的区别在于存储的数据量。`real`类型允许六位小数精度，`double precision`则可以达到 15 位小数精度，二者都包括小数点两侧的位数。这些浮动类型也叫做*可变精度*类型。数据库将数字存储为表示位数的部分和一个指数—即小数点的位置。因此，与`numeric`类型中我们指定固定精度和范围不同，在给定列中小数点可以根据数字的不同而“浮动”。

#### 使用定点和浮动类型

每种类型对总位数（即精度）有不同的限制，如表 4-3 所示。

表 4-3：定点和浮动数据类型

| **数据类型** | **存储大小** | **存储类型** | **范围** |
| --- | --- | --- | --- |
| `numeric`, `decimal` | 可变 | 定点类型 | 小数点前最多 131,072 位；小数点后最多 16,383 位 |
| `real` | 4 字节 | 浮动类型 | 6 位小数精度 |
| `double precision` | 8 字节 | 浮动类型 | 15 位小数精度 |

为了查看这三种数据类型如何处理相同的数字，创建一个小表格并插入各种测试用例，如清单 4-2 所示。

```
CREATE TABLE number_data_types (
    1 numeric_column numeric(20,5),
    real_column real,
    double_column double precision
);

2 INSERT INTO number_data_types
VALUES
    (.7, .7, .7),
    (2.13579, 2.13579, 2.13579),
    (2.1357987654, 2.1357987654, 2.1357987654);

SELECT * FROM number_data_types;
```

清单 4-2：数字数据类型的应用

我们创建了一个表格，每个分数数据类型都有一列，并将三行数据加载到表格中。每一行在所有三列中重复相同的数字。当脚本的最后一行执行并选择表格中的所有内容时，我们得到以下结果：

```
numeric_column    real_column    double_column
--------------    -----------    -------------
       0.70000            0.7              0.7
       2.13579        2.13579          2.13579
       2.13580      2.1357987     2.1357987654
```

注意发生了什么。设置为五位刻度的`numeric`列，无论你插入多少位小数，总是保留五位小数。如果小于五位，它会用零填充；如果大于五位，它会进行四舍五入——例如第三行的小数点后有十位数字。

`real`和`double precision`列没有填充。在第三行，你会看到 PostgreSQL 在这两列中的默认行为，它会输出浮点数的最简精确十进制表示，而不是显示完整的值。请注意，较旧版本的 PostgreSQL 可能会显示稍有不同的结果。

#### 遇到浮点数学问题

如果你在想，“嗯，作为浮点数存储的数字看起来和作为定点数存储的数字一样”，那就要小心了。计算机存储浮点数的方式可能导致意外的数学错误。看看当我们对这些数字进行一些计算时会发生什么。运行清单 4-3 中的脚本。

```
SELECT
    1 numeric_column * 10000000 AS fixed,
    real_column * 10000000 AS floating
FROM number_data_types
2 WHERE numeric_column = .7;
```

清单 4-3：浮动列的舍入问题

这里，我们将`numeric_column`和`real_column`分别乘以一千万，并使用`WHERE`子句筛选出第一行。我们应该得到相同的结果，对吧？下面是查询返回的结果：

```
fixed             floating
-------------     ----------------
7000000.00000     6999999.88079071
```

你好！难怪浮点类型被称为“不精确”。好在我没有用这些数学来发射火星任务或计算联邦预算赤字。

浮点数学会产生这些错误的原因是，计算机试图将大量信息压缩到有限的位数中。这个话题已经有很多讨论，超出了本书的范围，但如果你感兴趣，可以在[`www.nostarch.com/practical-sql-2nd-edition/`](https://www.nostarch.com/practical-sql-2nd-edition/)找到一个很好的摘要链接。

`numeric`数据类型所需的存储空间是可变的，具体取决于指定的精度和刻度，`numeric`可能比浮点类型消耗更多的空间。如果你正在处理数百万行数据，值得考虑是否能接受相对不精确的浮点数学。

### 选择你的数字数据类型

目前，在处理数字数据类型时，请考虑以下三条指南：

+   尽可能使用整数。除非你的数据使用小数，否则请坚持使用整数类型。

+   如果你在处理十进制数据并需要精确计算（例如处理货币），请选择 `numeric` 或其等效类型 `decimal`。浮动类型可以节省空间，但浮点数计算的不精确性在许多应用中是无法接受的。只有在精度不那么重要时，才使用它们。

+   选择一个足够大的数字类型。除非你在设计一个包含数百万行数据的数据库，否则最好选择更大的数据类型。当使用 `numeric` 或 `decimal` 时，确保精度足够大，以容纳小数点两边的数字。对于整数，除非你确定列值会被限制在较小的 `integer` 或 `smallint` 类型中，否则使用 `bigint`。

## 理解日期和时间

每当你在搜索表单中输入日期时，你正在享受数据库对当前时间的感知（从服务器接收的）以及能够处理日期、时间格式和日历的细节（如闰年和时区）。这对于通过数据讲故事至关重要，因为关于*何时*发生某事的问题通常和“谁”、“什么”或“多少人”一样有价值。

PostgreSQL 的日期和时间支持包括 表 4-4 所示的四种主要数据类型。

表 4-4：日期和时间数据类型

| **数据类型** | **存储大小** | **描述** | **范围** |
| --- | --- | --- | --- |
| `timestamp` | 8 字节 | 日期和时间 | 公元前 4713 年到公元 294276 年 |
| `date` | 4 字节 | 仅日期（无时间） | 公元前 4713 年到公元 5874897 年 |
| `time` | 8 字节 | 时间（无日期） | 00:00:00 到 24:00:00 |
| `interval` | 16 字节 | 时间间隔 | +/− 1.78 亿年 |

以下是 PostgreSQL 中日期和时间数据类型的概述：

1.  `timestamp` 记录日期和时间，适用于你可能跟踪的一系列情况：乘客航班的起降时间、大联盟棒球比赛的时间表，或时间线上的事件。你几乎总是希望在事件发生的时间后面加上 `with time zone` 关键字，以确保记录的时间包括发生地点的时区。否则，不同地区记录的时间将无法进行比较。`timestamp with time zone` 格式是 SQL 标准的一部分；在 PostgreSQL 中，你可以使用 `timestamptz` 来指定相同的数据类型。

1.  `date` 仅记录日期，是 SQL 标准的一部分。

1.  `time` 仅记录时间，是 SQL 标准的一部分。虽然你可以添加 `with time zone` 关键字，但没有日期时，时区将没有意义。

1.  `i``nterval` 表示一个时间单位的值，采用`数量 单位`的格式。它不记录时间段的开始或结束，只记录持续时间。示例包括`12 天`或`8 小时`。（PostgreSQL 文档在[`www.postgresql.org/docs/current/datatype-datetime.html`](https://www.postgresql.org/docs/current/datatype-datetime.html)中列出了从`微秒`到`千年`的单位值。）你通常会使用此类型进行计算或过滤其他日期和时间列。它也是 SQL 标准的一部分，尽管 PostgreSQL 特有的语法提供了更多选项。

让我们关注一下`timestamp with time zone`和`interval`类型。要查看这些类型的实际应用，请运行清单 4-4 中的脚本。

```
1 CREATE TABLE date_time_types (
    timestamp_column timestamp with time zone,
    interval_column interval
);

2 INSERT INTO date_time_types
VALUES
    ('2022-12-31 01:00 EST','2 days'),
    ('2022-12-31 01:00 -8','1 month'),
    ('2022-12-31 01:00 Australia/Melbourne','1 century'),
    3 (now(),'1 week');

SELECT * FROM date_time_types;
```

清单 4-4：`timestamp`和`interval`类型的实际应用

在这里，我们创建一个包含这两种类型列的表，并插入四行数据。对于前三行，我们的`timestamp_column`插入使用相同的日期和时间（2022 年 12 月 31 日凌晨 1 点），采用国际标准化组织（ISO）日期和时间格式：`YYYY``-``MM``-``DD HH``:``MM``:``SS`。SQL 支持其他日期格式（如`MM/DD/YYYY`），但建议使用 ISO 格式，以确保全球的可移植性。

在指定时间之后，我们指定了时区，但在前三行中使用了不同的格式：在第一行中，我们使用了`EST`缩写，表示美国东部标准时间。

在第二行中，我们设置时区为`-8`。该值表示与世界协调时间（UTC）之间的小时差，或称为*偏移量*。UTC 的值为+/−00:00，因此`-8`表示比 UTC 时间晚 8 小时。在美国，夏令时生效时，`-8`是阿拉斯加时区的值。从 11 月到次年 3 月初，美国恢复标准时间时，这个值指的是太平洋时区。（关于 UTC 时区的地图，参见[`en.wikipedia.org/wiki/Coordinated_Universal_Time#/media/File:Standard_World_Time_Zones.tif`](https://en.wikipedia.org/wiki/Coordinated_Universal_Time#/media/File:Standard_World_Time_Zones.tif)。）

对于第三行，我们使用地区和位置的名称来指定时区：`Australia/Melbourne`。该格式使用了一个标准时区数据库中的值，通常在计算机编程中使用。你可以在[`en.wikipedia.org/wiki/Tz_database`](https://en.wikipedia.org/wiki/Tz_database)了解更多关于时区数据库的信息。

在第四行中，脚本没有指定具体的日期、时间和时区，而是使用 PostgreSQL 的`now()`函数，3 该函数从你的硬件捕获当前事务时间。

脚本运行后，输出应该类似（但不完全相同）如下：

```
timestamp_column                 interval_column
-----------------------------    ---------------
2022-12-31 01:00:00-05           2 days
2022-12-31 04:00:00-05           1 mon
2022-12-30 09:00:00-05           100 years
2020-05-31 21:31:15.716063-05    7 days
```

尽管我们在`timestamp_column`的前三行中提供了相同的日期和时间，但每行的输出却不同。原因是 pgAdmin 根据我的时区报告日期和时间，在显示的结果中，每个时间戳的末尾都标出了`-05`的 UTC 偏移量。UTC 偏移量`-05`表示比 UTC 时间晚五小时，相当于美国东部时区在秋冬季节采用标准时间时的时间。如果你生活在不同的时区，你可能会看到不同的偏移量；时间和日期也可能与你看到的有所不同。我们可以更改 PostgreSQL 报告这些时间戳值的方式，我将在第十二章中介绍如何操作以及处理日期和时间的其他技巧。

最后，`interval_column`显示了你输入的值。PostgreSQL 将`1 century`转换为`100 years`，并将`1 week`转换为`7 days`，这是因为它在区间显示的首选默认设置。阅读 PostgreSQL 文档中的“Interval Input”部分，了解更多有关区间的选项，网址是[`www.postgresql.org/docs/current/datatype-datetime.html`](https://www.postgresql.org/docs/current/datatype-datetime.html)。

## 在计算中使用`interval`数据类型

`interval`数据类型对于日期和时间数据的简易计算非常有用。例如，假设你有一列保存了客户签署合同的日期。使用区间数据，你可以在每个合同日期上加上 90 天，以确定何时与客户跟进。

要查看`interval`数据类型如何工作，我们将使用刚刚创建的`date_time_types`表，如示例 4-5 所示。

```
SELECT
    timestamp_column,
    interval_column,
    1 timestamp_column - interval_column AS new_date
FROM date_time_types;
```

示例 4-5：使用`interval`数据类型

这是一条典型的`SELECT`语句，只是我们会计算一个名为`new_date`的列，该列包含`timestamp_column`减去`interval_column`的结果。（计算列称为*表达式*；我们将经常使用这种技巧。）在每一行中，我们从日期中减去`interval`数据类型指示的时间单位。这样会产生以下结果：

```
timestamp_column                 interval_column    new_date
-----------------------------    ---------------    -----------------------------
2022-12-31 01:00:00-05           2 days             2022-12-29 01:00:00-05
2022-12-31 04:00:00-05           1 mon              2022-11-30 04:00:00-05
2022-12-30 09:00:00-05           100 years          1922-12-30 09:00:00-05
2020-05-31 21:31:15.716063-05    7 days             2020-05-24 21:31:15.716063-05
```

请注意，`new_date`列默认格式化为`timestamp with time zone`类型，允许在区间值使用时显示时间值以及日期。（你可以在 pgAdmin 的结果网格中看到数据类型，显示在列名下方。）再次提醒，根据你的时区，输出可能会有所不同。

## 理解 JSON 和 JSONB

JSON 是*JavaScript 对象表示法*的缩写，是一种用于存储数据和在计算机系统之间交换数据的结构化数据格式。所有主要的编程语言都支持以 JSON 格式读取和写入数据，这种格式将信息组织为*键/值*对和数值列表。以下是一个简单的例子：

```
{
  "business_name": "Old Ebbitt Grill",
  "business_type": "Restaurant",
  "employees": 300,
  "address": {
    "street": "675 15th St NW",
    "city": "Washington",
    "state": "DC",
    "zip_code": "20005"
  }
}
```

这段 JSON 代码展示了该格式的基本结构。例如，*键*（key）`business_name`与*值*（value）`Old Ebbitt Grill`相关联。键的值也可以是一个包含额外键值对的集合，如`address`所示。JSON 标准对格式有严格要求，例如用冒号分隔键和值，并将键名用双引号括起来。你可以使用在线工具如[`jsonlint.com/`](https://jsonlint.com/)检查 JSON 对象是否具有有效的格式。

PostgreSQL 当前提供两种 JSON 数据类型，它们都强制执行有效的 JSON 格式，并支持处理该格式数据的函数：

1.  `json` 存储 JSON 文本的精确副本

1.  `jsonb` 以二进制格式存储 JSON 文本

这两者之间有显著的差异。例如，`jsonb`支持索引，这可以提高处理速度。

JSON 在 2016 年成为 SQL 标准的一部分，但 PostgreSQL 早在几年之前就已支持，从版本 9.2 开始。PostgreSQL 目前实现了 SQL 标准中的多个函数，并提供了自己的一些额外的 JSON 函数和操作符。我们将在第十六章中更详细地介绍这些类型和功能。

## 使用杂项类型

字符、数字和日期/时间类型可能是你在使用 SQL 时处理的主要类型。但 PostgreSQL 支持许多其他类型，包括但不限于以下几种：

1.  *布尔*类型，存储`true`或`false`的值

1.  *几何类型*，包括点、线、圆及其他二维对象

1.  *PostgreSQL 全文搜索引擎的文本搜索类型*

1.  *网络地址类型*，如 IP 地址或 MAC 地址

1.  *通用唯一标识符*（*UUID*）类型，有时用作表中的唯一键值

1.  *范围*类型，允许你指定值的范围，如整数或时间戳

1.  存储*二进制*数据的类型

1.  *XML*数据类型，用于存储这种结构化格式的信息

我将在本书中根据需要介绍这些类型。

## 使用 CAST 转换值的类型

有时，你可能需要将一个值从其存储的数据类型转换为另一种类型。例如，你可能想将一个数字作为字符检索，以便与文本结合使用，或者你可能需要将存储为字符的日期转换为实际的日期类型，以便按照日期顺序排序或进行时间间隔计算。你可以使用`CAST()`函数来执行这些转换。

`CAST()`函数仅在目标数据类型能够容纳原始值时成功。例如，将整数转换为文本是可能的，因为字符类型可以包含数字。而将包含字母的文本转换为数字则不行。

列表 4-6 展示了使用我们刚创建的三张数据类型表的三个示例。前两个示例能正常工作，但第三个示例将尝试执行无效的类型转换，这样你就可以看到类型转换错误是什么样子的。

```
1 SELECT timestamp_column, CAST(timestamp_column AS varchar(10))
FROM date_time_types;

2 SELECT numeric_column,
       CAST(numeric_column AS integer),
       CAST(numeric_column AS text)
FROM number_data_types;

3 SELECT CAST(char_column AS integer) FROM char_data_types;
```

列表 4-6：三个`CAST()`示例

第一个`SELECT`语句 1 将`timestamp_column`的值作为`varchar`返回，你应该记得`varchar`是可变长度字符列。在这种情况下，我已将字符长度设置为 10，这意味着转换为字符字符串时，只保留前 10 个字符。在这种情况下，这很方便，因为它只保留了列的日期部分，排除了时间。当然，也有更好的方法来从时间戳中去除时间，我将在第十二章的“提取时间戳值的组成部分”中讲解。

第二个`SELECT`语句 2 返回`numeric_column`的值三次：一次是原始形式，接着是整数形式，最后是`text`形式。在转换为整数时，PostgreSQL 会将值四舍五入为整数。但在转换为`text`时，不会发生四舍五入。

最后的`SELECT`3 不起作用：它返回错误`invalid input syntax for type integer`，因为字母不能转换为整数！

## 使用 CAST 快捷符号

最好编写别人也能阅读的 SQL，这样当其他人后来接手时，他们能明白你写的代码。`CAST()`的写法使你在使用它时的意图相当明显。然而，PostgreSQL 也提供了一种不那么显而易见的快捷符号，节省空间：*双冒号*。

在列名和你想要转换成的数据类型之间插入双冒号。例如，这两个语句将`timestamp_column`转换为`varchar`：

```
SELECT timestamp_column, CAST(timestamp_column AS varchar(10))
FROM date_time_types;

SELECT timestamp_column::varchar(10)
FROM date_time_types;
```

使用你觉得合适的方式，但要注意，双冒号是 PostgreSQL 特有的实现，其他 SQL 变种中没有，因此不能移植。

## 总结

现在，你已经具备了更好地理解你在深入研究数据库时遇到的数据格式的能力。如果你遇到作为浮动点数存储的货币值，你一定要在进行任何数学计算之前将其转换为十进制。而且，你还知道如何使用正确的文本列类型，以防止数据库过大。

接下来，我将继续讲解 SQL 基础，并向你展示如何将外部数据导入到你的数据库中。
