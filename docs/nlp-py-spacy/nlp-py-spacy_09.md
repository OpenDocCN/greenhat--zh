## **9

将用户输入存储在数据库中**

![Image](../Images/comm1.jpg)

许多为商业设计的应用程序需要在某个时刻将它们处理的数据传输到数据库中。例如，一个食品订购聊天机器人可能在通过与客户的对话提取信息后填写订单表单并保存该表单。一旦订单出现在数据库中，它就可以进行进一步处理，最终产品将被发送给客户。

本章讨论了如何将从提交的文本中提取的信息转换为结构化格式，以便将其存储并在*关系型*（行列）数据库中进行操作。通过示例，你将学习一个聊天机器人如何将输入文本拆解成小片段，并从中构建一个准备好存入数据库的结构。

### **将非结构化数据转换为结构化数据**

*结构化数据*是使用预定义的数据架构在格式化的存储库中组织的。如果你之前使用过关系数据库，你会知道必须首先将任何你要输入数据库的数据转换为结构化格式，这样它才能适应表格或一组相关表格。

问题在于，应用程序从用户接收到的自然语言输入是*非结构化*的，这意味着它没有预定义的组织架构。非结构化数据的典型示例包括文本和多媒体内容，如电子邮件、网页、商业文档、视频、照片等。尽管你仍然可以将非结构化数据存储在数据库中，但通常在插入时需要进行一些预处理。例如，你可能需要为照片加标签，以便数据库能够对其进行分类，或者为文本文件分配ID，以便数据库能够区分它们。

有时，你可能需要对非结构化文本内容进行更为彻底的转换，例如从中提取信息片段，然后将这些片段按格式组织成结构。例如，商业聊天机器人通常需要解析客户的言语，以填写某个表单。另一个应用程序可能会从网页中提取某些特定元素，对这些元素加标签，然后将信息转换为表格，如[图9-1](../Text/ch09.xhtml#ch09fig01)所示。

![image](../Images/fig9-1.jpg)

*图9-1：将非结构化内容转换为结构化数据的示例*

类似spaCy这样的工具通过为句子中的每个标记添加语言学注释，揭示了文本的内部结构。这一预处理使你能够从文本中提取特定元素，通常是通过检查文本的句法依赖标签。[图9-2](../Text/ch09.xhtml#ch09fig02)描述了食品订购聊天机器人如何通过依赖于spaCy为每个标记分配的句法依赖标签，在你对文本应用处理管道时，识别并提取用户话语中的必要元素。

![image](../Images/fig9-2.jpg)

*图9-2：原始文本如何转换为行列数据的高层次视图。*

接下来，你将看到一旦提取了这些元素，如何将它们构建并插入到数据库表中作为一行数据。

#### ***提取数据到交换格式***

许多当前的关系型数据库本身就支持多种常见的数据交换格式。例如，MySQL本身就支持XML和JSON，这两种是网页上最常见的数据交换格式。

你的数据格式选择会影响你决定如何收集数据。例如，如果你使用的数据库支持JSON格式，你可以直接将数据提取到JSON对象中，然后将其发送到数据库进行进一步处理。JSON对象是一种键值对数据格式，围绕着花括号。它的形式如下：

{"product": "pizza", "type": "Chicago", "qty": 1}

除了基本值，如字符串和数字，JSON还支持复杂值，如数组和其他JSON对象。你将看到这一点在《[构建数据库驱动的聊天机器人](../Text/ch09.xhtml#lev122)》中，[第132页](../Text/ch09.xhtml#page_132)有详细说明。

实际上，使用JSON格式显著简化了在Python脚本中为数据库构建数据结构的过程。首先，你无需准备符合较少使用格式的结构，这使得你的代码不那么依赖于特定的数据库类型。其次，JSON对象中的元素可以按任何顺序排列，这对如何组织从输入文本中提取和确定必要元素的过程限制更少。

[图9-3](../Text/ch09.xhtml#ch09fig03)展示了食品订购聊天机器人应用如何使用JSON与底层数据库进行交互。

![image](../Images/fig9-3.jpg)

*图9-3：一个食品订购聊天机器人应用的工作流*

在第1步，用户向聊天机器人提交一个请求，要求订购一份希腊比萨。在第2步，聊天机器人使用spaCy处理提交的语言，生成一个包含订购信息的JSON对象。在第3步，代表订单表单的JSON对象被提交到数据库，数据库存储该表单并生成关于它的响应供聊天机器人使用。在第4步，聊天机器人通知用户订单是否已成功下单。

#### ***将应用程序逻辑移动到数据库***

请注意，在[图9-3](../Text/ch09.xhtml#ch09fig03)中的聊天机器人应用中，数据库不仅存储了提交的JSON对象，还生成了关于订单保存是否成功的响应。原因是数据库运行了一部分应用程序逻辑。

数据库驱动的应用程序通常将与数据处理相关的应用程序逻辑保留在数据库中。这种方法可以减少应用程序逻辑层与底层数据库之间的数据移动，消除冗余，提高数据处理效率，并维护数据安全。

[图 9-4](../Text/ch09.xhtml#ch09fig04) 详细描述了聊天机器人应用程序中数据库部分，如 [图 9-3](../Text/ch09.xhtml#ch09fig03) 所示。

![image](../Images/fig9-4.jpg)

*图 9-4：聊天机器人应用程序中数据库的更详细视图，如 [图 9-3](../Text/ch09.xhtml#ch09fig03) 所示*

在此应用程序中，数据库将输入的 JSON 对象转换为关系型数据，并以保证数据插入正确和完整的方式存储在关系表中。如果某个字段的值缺失，客户将收到一条消息，告知他们应该提供什么信息。

在将输入数据移动到表格之前，你可以借助存储过程、SQL 语句中的 ON ERROR 子句或在数据表上定义的触发器检查每个字段的值。关于 SQL 的更详细讨论超出了本书的范围。但是在 “[准备你的数据库环境](../Text/ch09.xhtml#lev125)” 第 [135 页](../Text/ch09.xhtml#page_135)，你将看到一个使用 SQL 创建数据库基础设施并与 Python 交互的示例。

**注意**

*如果你使用的数据库不支持将 JSON 数据转换为关系型数据等功能，你需要在 Python 中自行实现检查数据完整性的逻辑；然而，这一讨论超出了本章的范围。*

### **构建一个基于数据库的聊天机器人**

现在你已经有了实现一个基于数据库的聊天机器人应用程序的基本概念，让我们为 [图 9-3](../Text/ch09.xhtml#ch09fig03) 中显示的应用程序创建一个简单的版本。该应用程序应处理用户的发言，提取必要的信息以填写订单表格，例如产品名称、产品类型和数量。然后，这些信息将被打包成一个 JSON 对象，并发送到底层数据库。数据库应将 JSON 对象拆解为关系型数据，并根据数据的完整性向应用程序发送响应。

#### ***收集数据并构建 JSON 对象***

我们将从开发应用程序的逻辑层开始，使用 Python 构建一个 JSON 对象，然后将其发送到任何数据库类型。以下代码展示了这种实现可能的样子：

import spacy

nlp = spacy.load('en')

doc = nlp(u'I want a Greek pizza.')

➊ orderdict ={}

➋ for token in doc:

➌ if token.dep_ == 'dobj':

dobj = token

➍ orderdict.update(product = dobj.lemma_)

➎ for child in dobj.lefts:

➏ if child.dep_ == 'amod' or child.dep_ == 'compound':

orderdict.update(ptype = child.text )

➐ elif child.dep_ == 'det':

orderdict.update(qty = 1 )

➑ elif child.dep_ == 'nummod':

orderdict.update(qty = child.text)

break

我们将 `orderdict` 字典定义为创建的 JSON 对象的容器 ➊。稍后我们将能够轻松地将这个字典转换为 JSON 字符串。

然后我们遍历话语的标记 ➋，寻找直接宾语 ➌。我们可能想要一个比萨，或者可能要求别人给我们做一个比萨。无论哪种情况，“pizza” 都将是我们话语中的直接宾语，因此我们在这里寻找直接宾语。当然，真正的实现会进行更多的检查。

一旦找到，我们在 `orderdict` 字典中定义一个键值对，将产品作为键，直接宾语的词根作为值 ➍。我们使用词形还原将产品名称的可能词形简化为其基本形式（在大多数情况下，从复数形式转为单数形式）。

接下来，我们遍历直接宾语的句法左孩子 ➎，因为我们期望在那里找到关于请求的产品类型的信息。在句法依赖标签方面，产品类型可以是复合词或形容词修饰语（amod） ➏。例如，spaCy 会将短语 “a Greek pizza” 中的单词 “Greek” 视为形容词修饰语，而将短语 “a Chicago pizza” 中的单词 “Chicago” 视为复合词。

现在我们检查修饰词或复合词的孩子中是否有限定词。限定词“a”意味着客户要求单个单位的产品 ➐。相反，带有依赖标签 nummod 的单词表示特定数量的单位 ➑。

使用以下命令打印 `orderdict` 字典：

print(orderdict)

这应该会给你以下结果：

{'product': 'pizza', 'ptype': 'Greek', 'qty': 1}

现在我们得到了一个 JSON 字符串，可以将其发送到底层数据库进行进一步处理。

#### ***将数字单词转换为数字***

在将 JSON 字符串发送到数据库的代码之前，考虑一下当用户明确指定产品数量时的情况，如以下话语所示：

我想要两个希腊风味的比萨。

如果你将此添加到前面的脚本中，你将得到以下结果：

{'product': 'pizza', 'ptype': 'Greek', 'qty': two}

在第一个示例句子中，'qty' 键的值是一个数字。在第二个示例中，它是一个拼写出来的数字单词。在这个阶段，这种差异看起来不成问题。但问题在于，当我们创建一个关系表时，必须为每一列定义数据类型。尝试将其他类型的数据插入该列会失败。

你应该为聊天机器人做好准备，客户可能会以任何他们喜欢的方式指定产品的数量。为了解决这个问题，你需要将表示数字单词的字符串转换为相应的整数值。

为此，定义一个包含用单词拼写的数字并按升序排列的列表；然后遍历该列表找到正确的数字。在此示例中，我们定义了一个数字单词列表，范围从“zero”到“twenty”，假设我们不认为客户在一次交易中会订购超过二十个相同的产品。

我们需要将这个转换场景实现为一个函数，该函数接收一个数字词或一个数字（在后者情况下，无需转换），并返回一个数字。然后，我们应该使用这个函数修改前面部分脚本的代码。以下是该函数实现的样子：

➊ def word2int(numword):

num = 0

➋ try:

➌ num = int(numword)

➍ return num

except ValueError:

➎ pass

➏ words = ["零", "一", "二", "三", "四", "五", "六", "七",

"八", "九", "十", "十一", "十二", "十三", "十四",

"十五", "十六", "十七", "十八", "十九", "二十"]

➐ for idx, word in enumerate(words):

➑ if word in numword:

num = idx

➒ return num

`word2int()` 函数接受一个参数：要转换为对应数字的数字词，或者已经是数字的数字，在后一种情况下我们不需要进行转换 ➊。该函数必须处理这两种情况，因为我们无法确定在顾客的陈述中会出现哪一种。

我们使用 try 和 except 块来处理不需要转换的情况 ➋。我们检查输入是否为整数 ➌，如果是，所有我们需要做的就是原样返回输入的数字 ➍。否则，我们忽略因尝试将非数字值当作整数处理而引发的错误，并继续将其转换为数字 ➎。

我们定义了一个数字词列表，从零开始，按递增顺序列出 ➏。然后，我们使用 `enumerate()` 方法遍历这个列表 ➐，寻找函数接收到的输入词 ➑。当找到匹配时，我们返回迭代的编号（该词在列表中的索引），作为输入数字词的数字表示 ➒。

将 `word2int()` 函数定义添加到前面的脚本中。然后转到脚本的末尾，找到以下代码行：

elif child.dep_ == 'nummod':

orderdict.update(qty = child.text)

按照以下方式更改它们，使用本节中定义的 `word2int()` 函数：

elif child.dep_ == 'nummod':

orderdict.update(qty = word2int(child.text))

现在让我们看看脚本如何处理这个句子：

我想要两个希腊式披萨。

这次，你应该得到以下结果：

{'product': 'pizza', 'ptype': 'Greek', 'qty': 2}

'qty' 字段的值现在是一个数字，我们有了一个一致的格式，可以发送到数据库。

#### ***准备你的数据库环境***

要准备你的数据库环境，你需要安装或获得对数据库的访问权限；在数据库中创建你需要的组件，例如数据库模式、表格等；并安装一个允许你与数据库交互的 Python 模块。

尽管你可以选择任何能够接收和处理 JSON 数据的数据库，例如 Oracle 数据库，本节使用的是 MySQL。MySQL 数据库有着长期支持最流行的数据交换格式的历史，包括 XML 和 JSON。而且，MySQL 是全球最受欢迎的开源数据库，支持大多数现代操作系统，包括 Linux、Windows、Unix 和 macOS。MySQL 提供免费可下载版本，并且还有商业版来满足特定的业务需求。

对于本章内容，你可以使用 MySQL Community Edition——这个可以免费下载安装的版本，适用于 GPL 许可。要了解更多关于 MySQL Community Edition 的信息，请访问其官方网站 *https://www.mysql.com/products/community/*。

你首先需要在系统上安装 MySQL。在撰写本文时，MySQL 8.0 是最新版本。请参考 MySQL 8.0 参考手册中的“安装和升级 MySQL”章节 *[https://dev.mysql.com/doc/refman/8.0/en/installing.html](https://dev.mysql.com/doc/refman/8.0/en/installing.html)*，或者将来 MySQL 版本的等效章节。在这里，你可以找到针对操作系统的详细安装说明。

安装完成后，你可以使用安装指南为你的操作系统指定的命令来启动 MySQL 服务器。在开始使用数据库之前，你需要获取安装过程中为 mysql 超级用户（‘root’@‘localhost’）生成的密码。你可以在安装错误日志文件中找到该密码。

拥有超级用户密码后，你可以通过系统终端使用以下命令连接到 MySQL 服务器：

$ mysql -uroot -p

输入密码：******

mysql>

如果你更喜欢使用图形界面，可以利用 MySQL Workbench (*[https://www.mysql.com/products/workbench/](https://www.mysql.com/products/workbench/)*)，它是一个统一的可视化工具，旨在建模和管理 MySQL 数据库。

连接到服务器后，你的第一步是为 root 用户选择一个新密码，替换安装过程中生成的随机密码。使用以下命令来进行操作：

ALTER USER 'root'@'localhost' IDENTIFIED BY 'Your-pswd';

现在你可以开始为应用程序开发所需的服务器基础设施了。你将首先创建一个数据库，将它作为其他对象的容器，这些对象是应用程序需要交互的。要创建数据库，请在 mysql> 提示符下输入以下命令：

mysql> CREATE DATABASE mybot;

查询 OK, 1 行受影响 (0.03 秒)

然后选择刚创建的数据库进行使用，方法如下：

mysql> USE mybot;

数据库已更改

你已经准备好开始创建数据库的结构了。对于这个例子，你需要创建一个包含以下命令的单个表：

CREATE TABLE orders (

id INT NOT NULL AUTO_INCREMENT,

product VARCHAR(30),

ptype VARCHAR(30),

qty INT,

PRIMARY KEY (id)

);

在设置好这个数据库基础设施后，你需要安装MySQL Connector/Python驱动程序，它允许你的Python代码与该基础设施进行交互。在任何操作系统上，你都可以通过pip安装Connector/Python，方法如下：

pip install mysql-connector-python

要获取有关如何安装此驱动程序的详细信息，请查看文档 *[https://dev.mysql.com/doc/connector-python/en/](https://dev.mysql.com/doc/connector-python/en/)。*

使用以下简单脚本，确保你已安装了Connector/Python：

import mysql.connector

cnx = mysql.connector.connect(user='root', password='Your_pswd',

host='127.0.0.1',

database='mybot')

cnx.close()

如果安装成功，您应该看不到任何错误信息。

#### ***将数据发送到底层数据库***

让我们回到[第134页](../Text/ch09.xhtml#page_134)的脚本。以下代码连接到你的数据库并将订单数据传递给orders表。将此代码附加到脚本中：

import json

➊ json_str = json.dumps(orderdict)

import mysql.connector

from mysql.connector import errorcode

try:

➋ cnx = mysql.connector.connect(user='root', password='Your_pswd',

host='127.0.0.1',

database='mybot')

➌ query = ("""INSERT INTO orders (product, ptype, qty)

SELECT product, ptype, qty FROM

JSON_TABLE(

➍ %s,

"$" COLUMNS(

qty    INT PATH '$.qty',

product   VARCHAR(30) PATH "$.product",

ptype     VARCHAR(30) PATH "$.ptype"

)

) AS jt1""")

➎ cursor = cnx.cursor()

➏ cursor.execute(query, ➐(json_str,))

➑ cnx.commit()

➒ except mysql.connector.Error as err:

print("错误代码:", err.errno)

print("错误消息: {}".format(err.msg))

finally:

cursor.close()

cnx.close()

我们首先将orderdict字典转换为JSON字符串 ➊。接下来，我们连接到数据库 ➋，并定义一个插入SQL语句，以便将其传递给数据库进行处理 ➌。注意语句中使用了占位符（称为*绑定变量*） ➍。使用占位符可以让我们编写在运行时接受输入的SQL语句。

在执行语句之前，我们首先创建一个mysql.connector的cursor对象 ➎，它使我们能够在连接到的数据库中对对象进行操作。然后我们可以执行INSERT语句 ➏，将我们在代码片段开头获得的JSON字符串 ➐ 绑定到语句中的占位符。请注意使用JSON_TABLE函数，它将提交的JSON数据拆解为表格数据，使其适合插入到关系表中。

在执行INSERT语句之后，我们需要显式地使用commit()方法提交语句的更改 ➑。否则，当连接关闭时（无论是通过cnx.close()显式关闭，还是脚本执行完成后），插入操作将会回滚。

如果数据库端发生错误，except代码块将开始执行 ➒。在接下来的部分中，你将学习如何在传递的JSON字符串没有包含所有字段时利用这个功能。

现在执行脚本。如果没有看到任何错误消息，返回到你在前一部分使用的 mysql 提示符并输入以下 SELECT 语句：

mysql> SELECT * FROM orders;

ID   PRODUCT     PTYPE    QTY

----  -----------  --------  ---

1    pizza       Greek    2

如果你能看到这个输出，说明你的 Python 脚本按预期工作。

#### ***当用户的请求不包含足够的信息时***

有时候，用户的请求可能没有提供足够的信息来填写订单表单中的所有字段。例如，考虑以下话句：

我要两个披萨。

[表 9-1](../Text/ch09.xhtml#ch09tab01) 显示了应用程序根据这个句子生成的订单表单。

**表 9-1:** 缺失信息的订单表单

| **product** | **ptype** | **quantity** |
| --- | --- | --- |
| pizza |  | 2 |

由于用户没有指定他们想要的披萨类型，ptype 字段的值缺失。为了解决这个问题，按照以下方式增强之前脚本中的 INSERT 语句：

query = ("""INSERT INTO orders (product, ptype, qty)

SELECT product, ptype, qty FROM

JSON_TABLE(

%s,

"$"

COLUMNS(

qty    INT PATH '$.qty' ➊ERROR ON EMPTY,

product   VARCHAR(30) PATH "$.product" ➊ERROR ON EMPTY,

ptype     VARCHAR(30) PATH "$.ptype" ➊ERROR ON EMPTY

)

) AS jt1""");

我们在每个 JSON_TABLE 列中添加了 ERROR ON EMPTY 选项 ➊。这个选项允许我们处理因尝试插入一个未包含所有必需字段的 JSON 字符串而导致的错误。

现在，当你执行包含“我要两个披萨。”的示例句子时，你应该会看到以下输出：

错误代码：3665

错误消息：缺少 JSON_TABLE 列 'ptype' 的值

我们可以扩展脚本，在这种情况下，聊天机器人会询问客户使用以下问题来澄清他们的订单：

你想要什么类型的披萨？

答案可能是这样的：

我想要希腊式的。

我们应该收到的答案句子的结构与原始句子的结构相似。因此，我们可以使用与分析原始句子时相同的代码来分析这个答案。当然，这种方法是基于对用户回应的假设。真实的实现将从这种方法开始，然后在必要时，转向其他可能的回应结构。例如，用户的回应可能只是一个词，“希腊”。在这种情况下，我们只需要检查它是否包含在我们的披萨类型列表中。

#### ***试试这个***

错误消息会告诉你缺少了哪个特定字段。但你仍然需要从消息中提取这个字段名，以便你可以要求客户澄清他们订单中的特定部分。一种方法是查看消息中介词的宾语。例如，在消息 Error-Message: Missing value for JSON_TABLE column 'ptype' 中，介词的宾语是 ptype。

### **总结**

在本章中，你学习了如何将原始文本切割成碎片，并将其插入到关系型数据库中。你使用了 JSON 格式与能够处理 JSON 输入的数据库进行交互，并将其提取为关系型数据。你还学会了在数据库中实现一些应用逻辑，借助纯 SQL，使得数据处理更加贴近数据本身。为了实现更复杂的场景，你可能需要使用触发器和存储过程——具体细节可以参考你所使用的数据库文档。
