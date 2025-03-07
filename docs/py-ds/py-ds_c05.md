## 第五章：使用数据库

![](img/chapterart.png)

*数据库*是一个有组织的数据集合，可以方便地访问、管理和更新。即使你的项目初期架构中没有数据库，流经你应用的数据在某个时刻也很可能会接触到一个或多个数据库。

本章延续上一章关于将数据导入 Python 应用程序的讨论，讲解如何处理数据库中的数据。这里的示例将展示如何访问和操作存储在不同类型数据库中的数据，包括将 SQL 作为主要工具来处理数据的数据库以及不使用 SQL 的数据库。你将学习如何使用 Python 与多种流行数据库进行交互，包括 MySQL、Regis 和 MongoDB。

数据库提供了许多优势。首先，借助数据库，你可以在脚本的多次调用之间持久化数据，并高效地在不同应用程序之间共享数据。此外，数据库语言可以帮助你系统地组织数据并回答关于数据的问题。更重要的是，许多数据库系统允许你在数据库内部实现编程代码，这可以提高应用程序的性能、模块化和可重用性。例如，你可以在数据库中存储一个*触发器*；这是一段代码，每当发生某个特定事件时自动触发，比如每次向某个特定表格插入新行时。 

数据库分为两类：关系型数据库和非关系型（NoSQL）数据库。关系型数据库有一个固定的结构，以数据的架构形式实现。这种方法有助于确保数据的完整性、一致性和总体准确性。然而，关系型数据库的主要缺点是随着数据量的增加，它们的扩展性较差。相比之下，NoSQL 数据库对数据结构没有限制，从而提供了更多的灵活性、适应性和可扩展性。本章将涵盖在关系型和非关系型数据库中存储和检索数据。

## 关系型数据库

*关系型数据库*，也叫做*行列式数据库*，是当今最常用的数据库类型。它们提供了一种结构化的方式来存储数据。就像亚马逊上的书籍列表有一套结构来存储信息，包括书名、作者、简介、评分等字段，存储在关系型数据库中的数据必须符合预定义的形式化架构。使用关系型数据库的工作从设计这个形式化架构开始：你定义一组表格，每个表格由一组字段或列组成，并且你指定每个字段将存储的数据类型。你还要建立表格之间的关系。然后，你就可以将数据存入数据库、从数据库中检索数据或根据需要更新数据。

关系型数据库旨在高效地插入、更新和/或删除少量到大量的结构化数据。在许多应用中，这种类型的数据库都能发挥重要作用。特别地，关系型数据库非常适合*在线事务处理（OLTP）*应用程序，这些应用程序处理大量用户的高交易量。

一些常见的关系型数据库系统包括 MySQL、MariaDB 和 PostgreSQL。本节将重点介绍 MySQL，毫无疑问是全球最流行的开源数据库，来说明如何与数据库交互。你将学习如何设置 MySQL、创建新数据库、定义其结构，并编写 Python 脚本来存储和检索数据。

### 理解 SQL 语句

*SQL*，或称*结构化查询语言*，是与关系型数据库进行交互的主要工具。尽管我们这里的重点是使用 Python 与数据库进行交互，但 Python 代码本身必须包含 SQL 语句才能实现这一点。全面了解 SQL 超出了本书的范围，但简要介绍这一查询语言仍然是必要的。

SQL 语句是数据库引擎（如 MySQL）识别并执行的文本命令。例如，这条 SQL 语句要求数据库检索`orders`表中`status`字段设置为`Shipped`的所有行：

```py
SELECT * FROM orders WHERE status = 'Shipped';
```

SQL 语句通常包含三个主要组成部分：一个*操作*，一个*目标*以及一个*条件*，后者用以限定操作的范围。在前面的例子中，`SELECT`是 SQL 操作，意味着我们正在从数据库中访问行。`orders`表是操作的目标，由`FROM`子句定义，条件则在语句的`WHERE`子句中指定。例如，这条语句缺少条件，因此它将检索`orders`表中的所有行：

```py
SELECT * FROM orders;
```

你还可以精炼 SQL 语句，使其仅影响表中的某些列。以下是如何仅检索`orders`表中所有行的`pono`和`date`列：

```py
SELECT pono, date FROM orders;
```

按约定，SQL 中的保留字，如`SELECT`和`FROM`，通常采用全大写字母书写。然而，SQL 是一个大小写不敏感的语言，因此这种大写并非严格必要。每个 SQL 语句应以分号结束。

像刚才展示的`SELECT`操作就是*数据操作语言（DML）* *语句*的例子，DML 语句是一类用于访问和操作数据库数据的 SQL 语句。其他 DML 操作包括`INSERT`、`UPDATE`和`DELETE`，分别用于向数据库中添加、修改和删除记录。*数据定义语言（DDL）* *语句*是另一类常见的 SQL 语句。你使用这些语句来实际定义数据库结构。典型的 DDL 操作包括`CREATE`来创建、`ALTER`来修改、`DROP`来删除数据容器，无论是列、表还是整个数据库。

### MySQL 入门

MySQL 可用于大多数现代操作系统，包括 Linux、Unix、Windows 和 macOS。提供免费和商业版。对于本章内容，你可以使用 MySQL 社区版（[`www.mysql.com/products/community`](https://www.mysql.com/products/community)），这是 MySQL 的免费下载版，适用于 GPL 许可证。有关你操作系统的 MySQL 安装详细说明，请参考 MySQL 最新版本的参考手册，网址为[`dev.mysql.com/doc`](https://dev.mysql.com/doc)。

在安装后，要启动 MySQL 服务器，你需要使用安装指南中为你的操作系统指定的命令。然后，你可以从系统终端使用*mysql*客户端程序连接到 MySQL 服务器：

```py
$ **mysql -uroot -p**
```

系统会要求你输入在 MySQL 服务器安装过程中设置的密码。之后，你将看到 MySQL 提示符：

```py
mysql>
```

如果你愿意，可以使用以下 SQL 命令为 root 用户选择一个新密码：

```py
mysql> **ALTER USER 'root'@'localhost' IDENTIFIED BY '**`your_new_pswd`**';**
```

现在，你可以创建你的应用程序所需的数据库。在`mysql>`提示符下输入以下命令：

```py
mysql> **CREATE DATABASE sampledb;**
Query OK, 1 row affected (0.01 sec)
```

这将创建一个名为`sampledb`的数据库。接下来，你必须选择该数据库进行使用：

```py
mysql> **USE sampledb;**
Database changed
```

现在，任何后续命令都将作用于你的`sampledb`数据库。

### 定义数据库结构

关系型数据库的结构来自其组成表格的构成以及这些表格之间的连接。链接不同表格的字段称为*键*。有两种类型：*主键*和*外键*。主键唯一标识表格中的一条记录。外键是另一个表中的字段，指向第一个表中的主键。通常，主键和对应的外键在两个表中共享相同的名称。

现在你已经创建了`sampledb`数据库，可以开始创建一些表格并定义它们的结构。为了演示目的，这些表格将具有与你在第三章中使用的一些 pandas 数据框相同的结构。以下是需要在数据库中实现的三个表格数据结构：

```py
emps

empno  empname         job
----------------------------
9001   Jeff Russell    sales
9002   Jane Boorman    sales
9003   Tom Heints      sales

salary

empno     salary
----------------
9001      3000
9002      2800
9003      2500

orders

pono   empno  total
-------------------
2608   9001   35
2617   9001   35
2620   9001   139
2621   9002   95
2626   9002   218
```

要查看这些结构之间可以建立哪些关系，请回顾第三章中的图 3-4 和图 3-6。如图 3-4 所示，`emps`表和`salary`表中的行之间具有一对一的关系。这个关系是通过`empno`字段建立的。`emps`表和`orders`表之间也通过`empno`字段建立了关系。这是一个一对多的关系，如图 3-6 所示。

你可以使用`mysql>`提示符通过 SQL 命令将这些数据结构添加到关系型数据库中。首先，创建`emps`表：

```py
mysql> **CREATE TABLE emps (**
 **empno INT NOT NULL,**
 **empname VARCHAR(50),**
 **job VARCHAR(30),**
 **PRIMARY KEY (empno)**
 **);**
```

你使用`CREATE TABLE`命令创建表，指定每一列及其数据类型，并可选择性地指定可存储数据的大小。例如，`empno`列用于存储整数（类型为`INT`），而对其应用的`NOT NULL`约束确保你无法插入一个`empno`字段为空的行。与此同时，`empname`列可以存储最长为 50 个字符的字符串（类型为`VARCHAR`），而`job`列可以存储最多 30 个字符的字符串。你还指定`empno`是该表的主键列，意味着它在表中不应有重复值。

在成功执行该命令后，你将看到以下消息：

```py
Query OK, 0 rows affected (0.03 sec)
```

同样，下面是如何创建`salary`表的示例：

```py
mysql> **CREATE TABLE salary (**
 **empno INT NOT NULL,**
 **salary INT,**
 **PRIMARY KEY (empno)**
 **);**

Query OK, 0 rows affected (0.05 sec)
```

接下来，你将在`salary`表的`empno`列中添加外键约束，引用`emps`表中的`empno`列：

```py
mysql> **ALTER TABLE salary ADD FOREIGN KEY (empno) REFERENCES emps (empno);**
```

该命令创建了`salary`表和`empno`表之间的关系。它规定了`salary`表中的员工编号必须与`emps`表中的员工编号匹配。这个约束确保如果`emps`表中没有对应的行，你将无法向`salary`表插入新行。

由于`salary`表目前没有任何行，因此`ALTER TABLE`操作不会影响任何行，可以从返回的消息中看到这一点：

```py
Query OK, 0 rows affected (0.14 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

最后，创建`orders`表：

```py
mysql> **CREATE TABLE orders (**
 **pono INT NOT NULL,**
 **empno INT NOT NULL,**
 **total INT,**
 **PRIMARY KEY (pono),**
 **FOREIGN KEY (empno) REFERENCES emps (empno)**
 **);**

Query OK, 0 rows affected (0.13 sec)
```

这次你在`CREATE TABLE`命令中添加了一个外键约束，从而在创建表时立即定义外键。

### 向数据库插入数据

现在你已经准备好向新创建的表中插入数据。虽然你可以使用`mysql>`提示符执行此操作，但通常这种操作是通过应用程序进行的。你将在 Python 代码中通过 MySQL Connector/Python 驱动与数据库进行交互。你可以通过`pip`安装它，如下所示：

```py
$ **pip install mysql-connector-python**
```

运行以下脚本，将数据填充到数据库表中：

```py
import mysql.connector

try:
❶ cnx = mysql.connector.connect(user='root', password='`your_pswd`',
                                host='127.0.0.1',
                                database='sampledb')
❷ cursor = cnx.cursor()
  # defining employee rows
❸ emps = [
   (9001, "Jeff Russell", "sales"),
   (9002, "Jane Boorman", "sales"),
   (9003, "Tom Heints", "sales")
  ]
  # defining the query
❹ query_add_emp = ("""INSERT INTO emps (empno, empname, job)
                      VALUES (%s, %s, %s)""")
  # inserting the employee rows
  for emp in emps:
  ❺ cursor.execute(query_add_emp, emp)
  # defining and inserting salaries
  salary = [
    (9001, 3000),
    (9002, 2800),
    (9003, 2500)
  ]
  query_add_salary = ("""INSERT INTO salary (empno, salary)
                         VALUES (%s, %s)""")
  for sal in salary:
    cursor.execute(query_add_salary, sal)
  # defining and inserting orders
  orders = [
    (2608, 9001, 35),
    (2617, 9001, 35),
    (2620, 9001, 139),
    (2621, 9002, 95),
    (2626, 9002, 218)
  ]
 query_add_order = ("""INSERT INTO orders(pono, empno, total)
                        VALUES (%s, %s, %s)""")
  for order in orders:
    cursor.execute(query_add_order, order)
  # making the insertions permanent in the database
❻ cnx.commit()
❼ except mysql.connector.Error as err:
  print("Error-Code:", err.errno)
  print("Error-Message: {}".format(err.msg))
❽ finally:
  cursor.close()
  cnx.close()
```

在这个脚本中，你将 MySQL Connector/Python 驱动导入为`mysql.connector`。然后你打开一个`try/except`块，它为你在脚本中执行任何与数据库相关的操作提供了一个模板。你将在`try`块中编写操作的代码，如果操作执行时发生错误，执行将转到`except`块。

在`try`块中，首先建立与数据库的连接，指定你的用户名和密码、主机 IP 地址（在这种情况下是本地主机）以及数据库名称❶。然后，获取与此连接相关的`cursor`对象❷。`cursor`对象提供了执行语句和获取结果的接口。

你将`emps`表的行定义为一个元组列表❸。然后，定义要执行的 SQL 语句，以便将这些行插入表中❹。在此`INSERT`语句中，你指定了要填充数据的字段，并使用`%s`占位符将这些字段与每个元组的成员相对应。在循环中，你执行该语句，使用`cursor.execute()`方法逐行插入数据❺。类似地，你也将数据插入`salary`和`orders`表中。在`try`块的末尾，你使用连接的`commit()`方法使所有插入的数据永久生效❻。

如果任何与数据库相关的操作失败，`try`块中的其余部分将被跳过，`except`子句会执行❼，并打印出由 MySQL 服务器生成的错误代码以及相应的错误信息。

`finally`子句无论如何都会执行❽。在此子句中，你显式地关闭`cursor`，然后关闭连接。

### 查询数据库数据

现在你已经用数据填充了表格，可以查询这些数据以供 Python 代码使用。假设你想检索`emps`表中`empno`大于`9001`的所有行。为此，可以参考前一节的脚本，只需更改`try`块如下：

```py
`--snip--`
try:
  cnx = mysql.connector.connect(user='root', password='`your_pswd`',
                                host='127.0.0.1',
                                database='sampledb')
  cursor = cnx.cursor()
 query = ("SELECT ❶ * FROM emps WHERE ❷ empno > %s")
❸ empno = 9001
❹ cursor.execute(query, (empno,))
❺ for (empno, empname, job) in cursor:
    print("{}, {}, {}".format(
      empno, empname, job))
`--snip--`
```

与插入操作不同，选择行不需要在循环中对每一行执行多个`cursor.execute()`操作。相反，你编写一个查询，指定要选择的行的条件，然后通过一次`cursor.execute()`操作一次性获取所有行。

在构成查询的`SELECT`语句中，你指定了星号符号（`*`），这意味着你想查看检索到的行中的所有字段❶。在`WHERE`子句中，你指定了选择行必须满足的条件。在这里，你表明只有当`empno`大于绑定到`%s`占位符的变量的值时，行才会被选中❷。在执行过程中，变量`empno`会绑定到占位符❸。当你使用`cursor.execute()`执行查询时，你将绑定变量作为第二个参数传递，并以元组的形式传递❹。即使只需要传递一个变量，`execute()`方法也要求将绑定变量作为元组或字典传递。

你通过`cursor`对象访问检索到的行，并在循环中遍历它。每一行作为元组访问，元组的项表示该行字段的值❺。在这里，你只是打印字段的值，逐行输出结果，如下所示：

```py
9002, Jane Boorman, sales
9003, Tom Heints, sales
```

您还可以编写`SELECT`语句，连接来自不同表的行。连接关系型数据库表类似于连接 pandas DataFrame 的过程，如第三章所讨论的那样。您通常通过在设置数据库时定义的外键关系来连接表。

例如，假设您想要连接`emps`和`salary`表，并保持`empno`大于`9001`的条件。您可以通过它们共享的`empno`列来实现这一点，因为您在`salary`表中将`empno`定义为外键，引用了`emps`表中的`empno`。您可以通过修改脚本中的`try`块来实现此连接：

```py
`--snip--`
try:
  cnx = mysql.connector.connect(user='root', password='`your_pswd`',
                                host='127.0.0.1',
                                database='sampledb')
  cursor = cnx.cursor()
  query = ("""SELECT ❶ e.empno, e.empname, e.job, s.salary
              FROM ❷ emps e JOIN salary s ON ❸ e.empno = s.empno
              WHERE ❹ e.empno > %s""")

  empno = 9001
 cursor.execute(query, (empno,))
  for (empno, empname, job, salary) in cursor:
    print("{}, {}, {}, {}".format(
      empno, empname, job, salary))
`--snip--`
```

这次，`query`包含一个`SELECT`语句，连接了`emps`和`salary`表。在`SELECT`列表中，您指定了要在连接中包含的两个表中的列 ❶。在`FROM`子句中，您指定了这两个表，并通过`JOIN`关键字连接它们，以及别名`e`和`s`，这对于区分两个表中同名的列是必要的 ❷。在`ON`子句中，您定义了连接条件，表示两个表中的`empno`列的值应该匹配 ❸。在`WHERE`子句中，与之前的例子一样，您使用`%s`占位符设置最小的`empno`值 ❹。

脚本输出以下行，每个员工的薪资与其在`emps`表中的记录连接：

```py
9002, Jane Boorman, sales, 2800
9003, Tom Heints, sales, 2500
```

### 使用数据库分析工具

在 MySQL 中持久化数据时，您可以利用数据库内置的分析工具，如分析 SQL，显著减少应用程序和数据库之间传输的数据量。*分析 SQL*是一组额外的 SQL 命令，旨在实际分析存储在数据库中的数据，而不仅仅是存储、检索和更新数据。例如，假设您只想导入与那些股票价格在一定期间内未跌破前一天价格 1%的公司相关的股市数据。您可以通过分析 SQL 进行初步分析，这样您就无需将整个股票价格数据集从数据库加载到 Python 脚本中。

为了了解这个过程，您将通过第三章介绍的 yfinance 库获取股票数据并将其存储到数据库表中。然后，您将从 Python 脚本中查询该表，只加载满足指定条件的股票数据部分。首先，您需要在`sampledb`数据库中创建一个表来存储股票数据。该表应有三列：`ticker`、`date`和`price`。在`mysql>`提示符下输入以下命令：

```py
mysql> **CREATE TABLE stocks(**
 **ticker VARCHAR(10),**
 **date VARCHAR(10),**
 **price DECIMAL(15,2)**
 **);** 
```

现在使用此脚本通过 yfinance 获取一些股票数据：

```py
import yfinance as yf
❶ data = []
❷ tickers = ['TSLA', 'FB', 'ORCL', 'AMZN']
for ticker in tickers:
❸ tkr = yf.Ticker(ticker)
  hist = tkr.history(period='5d')
  ❹ .reset_index()
❺ records = hist[['Date','Close']].to_records(index=False)❻ records = list(records)
  records = [(ticker, ❼ str(elem[0])[:10], round(elem[1],2)) for elem in records]
❽ data = data + records
```

首先，定义一个名为 `data` 的空列表，将填充股票数据❶。正如本章前面所见，`cursor.execute()` 方法在执行 `INSERT` 语句时期望数据以列表对象的形式传入。接下来，定义一个股票代码列表，你希望获取这些股票的数据❷。然后，在一个循环中，将 `tickers` 列表中的每个股票代码传递给 yfinance 的 `Ticker()` 函数❸。该函数返回一个 `Ticker` 对象，其 `history()` 方法提供与对应股票代码相关的数据。在此示例中，你获取了每个股票代码过去五个交易日的股票数据（`period='5d'`）。

`history()` 方法将股票数据返回为 pandas DataFrame，并以 `Date` 列作为索引。最终，你需要将该 DataFrame 转换为元组列表，以便插入数据库。由于需要将 `Date` 列包含在数据集中，你可以通过 DataFrame 的 `reset_index()` 方法将其从索引中移除，从而将 `Date` 转换为普通列❹。然后，从获取的 DataFrame 中仅提取 `Date` 和 `Close` 列，其中 `Close` 包含当天的股票收盘价，并将其转换为 NumPy 记录数组，这是将输入数据转换的一个中间步骤❺。接下来，将数据转换为元组列表❻。之后，还需要重新格式化每个元组，以便将其作为一行插入到 `stocks` 数据库表中。特别地，每个 `Date` 字段包含大量多余信息（小时、分钟、秒等）。通过提取每个元组中第一个字段的前 10 个字符，你仅保留年、月和日，这是你分析所需的全部信息❼。例如，`2022-01-06T00:00:00.000000000` 将变为 `2022-01-06`。最后，仍在循环内，将与股票代码相关的元组追加到 `data` 列表中❽。

因此，`data` 元组列表的内容可能如下所示：

```py
[
 ('TSLA', '2022-01-06', 1064.7),
 ('TSLA', '2022-01-07', 1026.96),
 ('TSLA', '2022-01-10', 1058.12),
 ('TSLA', '2022-01-11', 1064.4),
 ('TSLA', '2022-01-12', 1106.22),
 ('FB', '2022-01-06', 332.46),
 ('FB', '2022-01-07', 331.79),
 ('FB', '2022-01-10', 328.07),
 ('FB', '2022-01-11', 334.37),
 ('FB', '2022-01-12', 333.26),
 ('ORCL', '2022-01-06', 86.34),
 ('ORCL', '2022-01-07', 87.51),
 ('ORCL', '2022-01-10', 89.28),
 ('ORCL', '2022-01-11', 88.48),
 ('ORCL', '2022-01-12', 88.31),
 ('AMZN', '2022-01-06', 3265.08),
 ('AMZN', '2022-01-07', 3251.08),
 ('AMZN', '2022-01-10', 3229.72),
 ('AMZN', '2022-01-11', 3307.24),
 ('AMZN', '2022-01-12', 3304.14)
]
```

若要将此数据集作为一组行插入 `stocks` 表，请将以下代码附加到之前的脚本并重新执行：

```py
import mysql.connector
from mysql.connector import errorcode
try:
  cnx = mysql.connector.connect(user='root', password='`your_pswd`',
                                host='127.0.0.1',
                                database='sampledb')
  cursor = cnx.cursor()
  # defining the query
  query_add_stocks = ("""INSERT INTO stocks (ticker, date, price)
                         VALUES (%s, %s, %s)""")
  # adding the stock price rows
❶ cursor.executemany(query_add_stocks, data)
  cnx.commit()
except mysql.connector.Error as err:
  print("Error-Code:", err.errno)
  print("Error-Message: {}".format(err.msg))
finally:
  cursor.close()
  cnx.close()
```

这段代码遵循你之前用于将数据插入数据库的相同模型。不过，这次你使用 `cursor.executemany()` 方法，它允许你高效地多次执行 `INSERT` 语句，每次执行时使用 `data` 元组列表中的一个元组❶。

现在，你已经将数据存入数据库，可以使用分析 SQL 查询来进行操作，尝试回答问题。例如，为了筛选出股价比前一天低于 1% 的股票，如本节开始时所建议的，你需要一个能够分析同一股票在多天内股价的查询。作为第一步，以下查询生成一个数据集，其中包含当前股价和前一天的股价，并将它们放在同一行中。在 `mysql>` 提示符下试试这个查询：

```py
SELECT
  date,
  ticker,
  price,
  LAG(price) OVER(PARTITION BY ticker ORDER BY date) AS prev_price
FROM stocks;
```

`SELECT`列表中的`LAG()`函数是一个分析 SQL 函数。它让你从当前行访问上一行的数据。`OVER`子句中的`PARTITION BY`子句将数据集分成多个组，每个组对应一个 ticker。`LAG()`函数在每个组内单独应用，确保数据不会从一个 ticker 传递到另一个 ticker。查询生成的结果看起来大概是这样的：

```py
+------------+--------+---------+------------+
| date       | ticker | price   | prev_price |
+------------+--------+---------+------------+

| 2022-01-06 | AMZN   | 3265.08 |       NULL |
| 2022-01-07 | AMZN   | 3251.08 |    3265.08 |
| 2022-01-10 | AMZN   | 3229.72 |    3251.08 |
| 2022-01-11 | AMZN   | 3307.24 |    3229.72 |
| 2022-01-12 | AMZN   | 3304.14 |    3307.24 |
| 2022-01-06 | FB     |  332.46 |       NULL |
| 2022-01-07 | FB     |  331.79 |     332.46 |
| 2022-01-10 | FB     |  328.07 |     331.79 |
| 2022-01-11 | FB     |  334.37 |     328.07 |
| 2022-01-12 | FB     |  333.26 |     334.37 |
| 2022-01-06 | ORCL   |   86.34 |       NULL |
| 2022-01-07 | ORCL   |   87.51 |      86.34 |
| 2022-01-10 | ORCL   |   89.28 |      87.51 |
| 2022-01-11 | ORCL   |   88.48 |      89.28 |
| 2022-01-12 | ORCL   |   88.31 |      88.48 |
| 2022-01-06 | TSLA   | 1064.70 |       NULL |
| 2022-01-07 | TSLA   | 1026.96 |    1064.70 |
| 2022-01-10 | TSLA   | 1058.12 |    1026.96 |
| 2022-01-11 | TSLA   | 1064.40 |    1058.12 |
| 2022-01-12 | TSLA   | 1106.22 |    1064.40 |
+------------+--------+---------+------------+
20 rows in set (0.00 sec)
```

查询生成了一个新的列`prev_price`，包含前一天的股票价格。如你所见，`LAG()`基本上让你在同一行中访问两行数据，这意味着你可以在同一个数学表达式中操作这两行数据作为查询的一部分。例如，你可以将一个价格除以另一个价格来计算每日的百分比变化。考虑到这一点，下面是一个查询，满足最初的要求，仅选择那些价格在指定期间内没有跌幅超过 1%的代码的行：

```py
❶ SELECT s.* FROM stocks AS s
LEFT JOIN
❷ (SELECT DISTINCT(ticker) FROM
  ❸ (SELECT
     ❹ price/LAG(price) OVER(PARTITION BY ticker ORDER BY date) AS dif,
       ticker 
     FROM stocks) AS b
❺ WHERE dif <0.99) AS a
❻ ON a.ticker = s.ticker
❼ WHERE a.ticker IS NULL;
```

这个 SQL 语句是对同一表`stocks`发出的两个不同查询的连接。连接的第一个查询检索了`stocks`表的所有行❶，而第二个查询仅检索了那些价格至少下降了 1%或更多的代码，这种下降发生在分析期的至少一天❷。这个连接的第二个查询结构较为复杂：它从子查询中选择数据，而不是直接从`stocks`表中选择。该子查询从❸开始，检索那些在`price`字段中的值比前一行低至少 1%的行。你通过将`price`除以`LAG(price)`❹并检查结果是否小于`0.99`❺来确定这一点。然后，在主查询的`SELECT`列表中，你对`ticker`字段应用`DISTINCT()`函数，以去除结果集中重复的代码名❷。

你在`ticker`列上连接了查询❻。在`WHERE`子句中，你指示连接只检索那些`a.ticker`字段（价格跌幅超过 1%的代码）和`s.ticker`字段（所有代码）之间没有对应关系的行❼。由于使用了左连接，只有来自第一个查询的匹配行会被检索。结果是，连接返回了所有`tickers`不在第二个查询返回的代码中的`stocks`表行。

根据前面展示的股票数据，查询生成的结果集如下：

```py
+--------+------------+---------+
| ticker | date       | price   |
+--------+------------+---------+

| ORCL   | 2022-01-06 |   86.34 |
| ORCL   | 2022-01-07 |   87.51 |
| ORCL   | 2022-01-10 |   89.28 |
| ORCL   | 2022-01-11 |   88.48 |
| ORCL   | 2022-01-12 |   88.31 |
| AMZN   | 2022-01-06 | 3265.08 |
| AMZN   | 2022-01-07 | 3251.08 |
| AMZN   | 2022-01-10 | 3229.72 |
| AMZN   | 2022-01-11 | 3307.24 |
| AMZN   | 2022-01-12 | 3304.14 |
+--------+------------+--------+
10 rows in set (0.00 sec)
```

如你所见，并非所有来自`stocks`表的行都已被检索。特别是，你不会找到与 FB 和 TSLA 代码相关的行。例如，后者被排除是因为在之前查询生成的输出中发现了以下这一行：

```py
+------------+--------+---------+------------+
| date       | ticker | price   | prev_price |
+------------+--------+---------+------------+
  ...
  2022-01-07 | TSLA   | 1026.96 |    1064.70 |
  ...
```

该行显示了 3.54%的跌幅，超出了 1%的阈值。

在以下脚本中，你在 Python 代码中发出相同的查询，并将结果提取到一个 pandas DataFrame 中：

```py
import pandas as pd
import mysql.connector
from mysql.connector import errorcode
try:
  cnx = mysql.connector.connect(user='root', password='`your_pswd`',
                                host='127.0.0.1',
                                database='sampledb')
  query = ("""
    SELECT s.* FROM stocks AS s
    LEFT JOIN
     (SELECT DISTINCT(ticker) FROM
       (SELECT
         price/LAG(price) OVER(PARTITION BY ticker ORDER BY date) AS dif,
         ticker
        FROM stocks) AS b
      WHERE dif <0.99) AS a
    ON a.ticker = s.ticker
    WHERE a.ticker IS NULL""")
❶ df_stocks = pd.read_sql(query, con=cnx)
❷ df_stocks = df_stocks.set_index(['ticker','date'])
except mysql.connector.Error as err:
  print("Error-Code:", err.errno)
  print("Error-Message: {}".format(err.msg))
finally:
  cnx.close()
```

这个脚本与本章前面展示的那些大体相似，主要区别是你直接将数据库数据加载到 pandas DataFrame 中。为此，你使用 pandas 的 `read_sql()` 方法，该方法的第一个参数是 SQL 查询语句（作为字符串），第二个参数是数据库连接对象 ❶。然后，你将 `ticker` 和 `date` 列设置为 DataFrame 的索引 ❷。

给定之前展示的股票数据，生成的 `df_stocks` DataFrame 将如下所示：

```py
 price
ticker date               
ORCL   2022-01-06    86.34
       2022-01-07    87.51
       2022-01-10    89.28
       2022-01-11    88.48
       2022-01-12    88.31
AMZN   2022-01-06  3265.08
       2022-01-07  3251.08
       2022-01-10  3229.72
       2022-01-11  3307.24
       2022-01-12  3304.14
```

现在你已经将数据加载到 DataFrame 中，可以在 Python 中继续进行进一步分析。例如，你可能想计算每个股票符号在一定时间段内的平均价格。在下一章中，你将看到如何通过在 DataFrame 中按组级别应用适当的聚合函数来解决这类问题。

## NoSQL 数据库

*NoSQL 数据库*，或称 *非关系型数据库*，不要求存储数据时有预先定义的组织架构，也不支持像连接（joins）这样的标准关系型数据库操作。相反，它们提供了更加灵活的结构化数据存储方式，使得处理大量数据变得更加容易。例如，一种类型的 NoSQL 数据库是键值存储（key-value store），它允许你将数据存储为键值对，例如时间-事件对。另一种类型的 NoSQL 数据库是面向文档的数据库，它们被设计为处理灵活结构的数据容器，如 JSON 文档。这使得你可以将与特定对象相关的所有信息作为数据库中的一个条目进行存储，而不是像关系型数据库中那样将信息拆分到多个表中。

尽管 NoSQL 数据库的出现时间不如关系型数据库那样久远，但由于它们允许开发人员以简单直观的格式存储数据，并且无需高级技术就能访问和操作数据，因此它们迅速获得了普及。它们的灵活性使其特别适用于实时和大数据应用，例如 Google Gmail 或 LinkedIn。

### 键值存储

*键值存储* 是一种持有键值对的数据库，类似于 Python 字典。一个典型的键值存储示例是 Redis，它代表远程字典服务（Remote Dictionary Service）。Redis 支持诸如 `GET`、`SET` 和 `DEL` 等命令，用于访问和操作键值对，以下是一个简单的示例：

```py
$ **redis-cli**
127.0.0.1:6379> **SET emp1 "Maya Silver"**
OK
127.0.0.1:6379> **GET emp1**
"Maya Silver"
```

在这里，你使用 `SET` 命令创建键 `emp1`，并为其赋值 `Maya Silver`，然后使用 `GET` 命令通过键获取值。

#### 设置 Redis

要自行探索 Redis，你需要安装它。你可以在[`redis.io/topics/quickstart`](https://redis.io/topics/quickstart)上找到 Redis 快速入门页面的详细信息。安装完 Redis 服务器后，你还需要安装 redis-py，这是一个让你能够通过 Python 代码与 Redis 交互的 Python 库。你可以使用`pip`命令来完成安装：

```py
$ **pip install redis**
```

然后，你可以通过命令`import redis`将 redis-py 导入到你的脚本中。

#### 使用 Python 访问 Redis

以下是一个通过 redis-py 库从 Python 访问 Redis 服务器的简单示例：

```py
> **import redis**
❶ > **r = redis.Redis()**
❷ > **r.mset({"emp1": "Maya Silver", "emp2": "John Jamison"})**
True
❸ > **r.get("emp1")**
b'Maya Silver'
```

你使用`redis.Redis()`方法建立与 Redis 服务器的连接 ❶。由于方法的参数被省略，默认值将被采用，这假设服务器运行在你的本地机器上：`host='localhost'`，`port=6379`，以及`db=0`。

在建立连接后，你使用`mset()`方法设置多个键值对 ❷（*m*代表*multiple*）。当数据成功存储时，服务器返回`True`。然后，你可以使用`get()`方法获取任何已存储键的值 ❸。

像任何其他数据库一样，Redis 允许你持久化插入的数据，这样你就可以在另一个 Python 会话或脚本中通过键来获取值。Redis 还允许你在设置键值对时为键设置*过期标志*，指定它应该保留多长时间。这在实时应用程序中尤为有用，因为输入数据在一定时间后会变得无关紧要。例如，如果你的应用程序是一个出租车服务，你可能想要存储每辆出租车的可用性数据。由于这些数据会经常变化，你可能希望它们在短时间后过期。以下是这种情况的示例：

```py
`--snip--`
> **from datetime import timedelta**
> **r.setex("cab26", timedelta(minutes=1), value="in the area now")**
True
```

你使用`setex()`方法设置一个键值对，该键值对将在指定的时间后自动从数据库中删除。在这里，你将过期时间指定为一个`timedelta`对象。或者，你可以将其指定为秒数。

到目前为止，我们只讨论了简单的键值对，但你还可以使用 Redis 存储与同一对象相关的多个信息，正如下例所示：

```py
> **cabDict = {"ID": "cab48", "Driver": "Dan Varsky", "Brand": "Volvo"}**
> **r.hmset("cab48", cabDict)**
> **r.hgetall("cab48")**
{'Cab': 'cab48', 'Driver': 'Dan Varsky', 'Brand': 'Volvo'}
```

你首先定义一个 Python 字典，可以包含任意数量的键值对。然后，你将整个字典发送到数据库，并通过`hmset()`将其存储在`cab48`键下（*h*代表*hash*）。接着，你使用`hgetall()`函数来检索存储在`cab48`键下的所有键值对。

### 文档导向数据库

*文档导向数据库* 将每条记录存储为一个单独的文档。与关系型数据库表的字段必须遵循预定义的模式不同，文档导向数据库中的每个文档可以具有自己的结构。这种灵活性使得文档导向数据库成为最受欢迎的 NoSQL 数据库类型，而在文档导向数据库中，MongoDB 无疑是领先者。MongoDB 旨在管理 JSON 类文档的集合。在本节中，我们将探讨如何使用 MongoDB。

#### 设置 MongoDB

你可以尝试使用 MongoDB 的几种方式。其中一种是将 MongoDB 数据库安装到你的系统上。详情请参考 MongoDB 文档：[`docs.mongodb.com/manual/installation`](https://docs.mongodb.com/manual/installation)。另一种无需安装的方式是使用 MongoDB Atlas 创建一个免费的托管 MongoDB 数据库。你需要在 [`www.mongodb.com/cloud/atlas/register`](https://www.mongodb.com/cloud/atlas/register) 注册账号。

在你开始从 Python 与 MongoDB 数据库交互之前，你需要安装 PyMongo，这是 MongoDB 的官方 Python 驱动程序。你可以使用 `pip` 命令来安装：

```py
$ **pip install pymongo**
```

#### 使用 Python 访问 MongoDB

使用 Python 操作 MongoDB 的第一步是通过 PyMongo 的 `MongoClient` 对象建立与数据库服务器的连接，如下所示：

```py
> **from pymongo import MongoClient**
> **client = MongoClient('**`connection_string`**')**
```

连接字符串可以是 MongoDB 连接 URI，例如 *mongodb://localhost:27017*。这个连接字符串假设你已在本地系统上安装了 MongoDB。如果你使用的是 MongoDB Atlas，你需要使用 Atlas 提供的连接字符串。有关详细信息，请参考 Atlas 文档中的“通过驱动程序连接”页面：[`docs.atlas.mongodb.com/driver-connection`](https://docs.atlas.mongodb.com/driver-connection)。你还可以查看 MongoDB 文档中的“连接字符串 URI 格式”页面：[`docs.mongodb.com/manual/reference/connection-string`](https://docs.mongodb.com/manual/reference/connection-string)。

除了使用连接字符串外，你还可以将主机和端口作为 `MongoClient()` 构造函数的单独参数来指定：

```py
> **client = MongoClient('localhost', 27017)**
```

一个 MongoDB 实例可以支持多个数据库，因此一旦与服务器建立连接，你需要指定要操作的数据库。MongoDB 并未提供单独的命令来创建数据库，因此你使用相同的语法来创建新数据库或访问现有数据库。例如，要创建一个名为 `sampledb` 的数据库（如果已存在，则访问它），你可以使用以下类似字典的语法：

```py
> **db = client['sampledb']**
```

或者使用属性访问语法：

```py
> **db = client.sampledb**
```

与关系型数据库不同，MongoDB 不将数据存储在表格中。相反，文档被分组到 *集合* 中。创建或访问集合类似于创建或访问数据库：

```py
> **emps_collection = db['emps']**
```

此命令将在 `sampledb` 数据库中创建 `emps` 集合，如果该集合尚未创建。然后，你可以使用 `insert_one()` 方法将文档插入该集合。在这个例子中，你将插入一个格式为字典的 `emp` 文档：

```py
> **emp = {"empno": 9001,**
...         **"empname": "Jeff Russell",**
...         **"orders": [2608, 2617, 2620]}**
> **result = emps_collection.insert_one(emp)**
```

在文档插入后，会自动为其添加一个 `"_id"` 字段。该字段的值在集合中是唯一的。你可以通过 `insert_one()` 返回的对象的 `inserted_id` 字段访问该 ID：

```py
> **result.inserted_id**
ObjectId('69y67385ei0b650d867ef236')
```

现在你已经在数据库中插入了一些数据，如何查询它呢？最常见的查询类型是使用 `find_one()`，它返回一个与搜索条件匹配的单一文档：

```py
> **emp = emps_collection.find_one({"empno": 9001})**
> **print(emp)**
```

如你所见，`find_one()` 不需要使用文档的 ID，该 ID 在插入时会自动添加。相反，你可以查询特定的元素，只要返回的文档与之匹配。

结果将类似于以下内容：

```py
{
 u'empno': 9001,
 u'_id': ObjectId('69y67385ei0b650d867ef236'),
 u'empname': u'Jeff Russell',
 u'orders': [2608, 2617, 2620]
}
```

## 总结

在本章中，你看到了将数据从不同类型的数据库（包括关系型数据库和 NoSQL 数据库）进行迁移的示例。你使用了 MySQL，这是最受欢迎的关系型数据库之一。接着，你了解了 Redis，一种 NoSQL 解决方案，它允许你高效地存储和检索键值对。你还探讨了 MongoDB，可以说是今天最受欢迎的 NoSQL 数据库，它允许你以对 Python 友好的方式处理类似 JSON 的文档。
