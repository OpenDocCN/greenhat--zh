## **12**

**管理关系型数据库**

![image](../images/common01.jpg)

应用程序几乎总是需要存储某种类型的数据，开发者通常会将关系型数据库管理系统（RDBMS）与某种类型的对象关系映射工具（ORM）结合使用。RDBMS和ORM可能会比较棘手，许多开发者对此话题并不感兴趣，但迟早必须处理这些问题。

### **RDBMS、ORM及何时使用它们**

RDBMS是存储应用程序关系数据的数据库。开发者将使用类似SQL（结构化查询语言）这样的语言来处理关系代数，也就是说，这类语言负责数据管理及数据之间的关系。它们配合使用，可以让你既存储数据，又能高效查询这些数据以获取特定的信息。深入理解关系数据库结构，比如如何正确使用规范化或不同类型的可序列化，可能会帮助你避免许多陷阱。显然，这些主题值得一本书来详细探讨，本章不会涵盖所有内容，而是集中讲解如何通过其常用编程语言SQL来使用数据库。

开发者可能不想花时间学习一门全新的编程语言来与RDBMS交互。如果是这样，他们往往会完全避免编写SQL查询，而是依赖于某个库来为他们完成这项工作。ORM库通常可以在编程语言生态系统中找到，Python也不例外。

ORM的目的是通过抽象创建查询的过程，使数据库系统更加易于访问：它会生成SQL，因此你不需要手动编写。不幸的是，这一抽象层可能会妨碍你执行一些更具体或更底层的任务，这些任务ORM根本无法完成，比如编写复杂的查询。

使用ORM时，在面向对象程序中有一组特定的难题，它们普遍存在，以至于被统称为*对象关系阻抗不匹配*。这种阻抗不匹配的出现，是因为关系型数据库和面向对象程序在数据表示上有所不同，导致无法正确映射：无论你怎么做，将SQL表映射到Python类，都无法得到最佳结果。

理解SQL和RDBMS将使你能够编写自己的查询，而无需完全依赖抽象层来处理所有事情。

但这并不是说你应该完全避免使用ORM。ORM库在快速原型化应用模型时非常有用，有些库甚至提供了像模式升级和降级这样的有用工具。需要理解的是，使用ORM并不能代替真正理解RDBMS的过程：许多开发者试图用他们选择的语言解决问题，而不是使用其模型API，结果得出的解决方案充其量只是勉强应付。

**注意**

*本章假设你已经了解基本的SQL。介绍SQL查询和讨论表格如何工作的内容超出了本书的范围。如果你是SQL新手，建议在继续之前先学习基本知识。Anthony DeBarros的《Practical SQL》（No Starch Press, 2018）是一个不错的入门选择。*

让我们看一个示例，演示为什么理解RDBMS（关系数据库管理系统）有助于你编写更好的代码。假设你有一个SQL表格用于跟踪消息。这个表格有一个名为id的单列，表示消息发送者的ID，这是主键，还有一个包含消息内容的字符串，如下所示：

CREATE TABLE message (

id serial PRIMARY KEY,

content text

);

我们希望检测到任何重复的消息并将其排除在数据库之外。为此，一个典型的开发者可能会使用ORM编写SQL，如[清单12-1](ch12.xhtml#ch12list1)所示。

if query.select(Message).filter(Message.id == some_id):

# 我们已经有这条消息，它是重复的，忽略并抛出

raise DuplicateMessage(message)

else:

# 插入消息

query.insert(message)

*清单12-1：使用ORM检测并排除重复消息*

这段代码适用于大多数情况，但它有一些主要缺点：

+   重复约束已经在SQL模式中表达，因此存在某种代码重复：使用PRIMARY KEY隐式定义了id字段的唯一性。

+   如果消息尚未存入数据库，这段代码会执行两个SQL查询：首先是一个SELECT语句，然后是一个INSERT语句。执行SQL查询可能需要较长时间，并且需要与SQL服务器来回交互，从而引入额外的延迟。

+   这段代码没有考虑到可能会有其他人在我们调用select_by_id()后，但在我们调用insert()之前插入一条重复消息，这将导致程序抛出异常。这种漏洞被称为*竞态条件*。

这里有一种更好的方法来编写这段代码，但它需要与RDBMS服务器合作。我们可以直接插入消息，并使用try...except块来捕获重复冲突，而不是先检查消息是否存在再插入：

try:

# 插入消息

message_table.insert(message)

except UniqueViolationError:

# 重复

raise DuplicateMessage(message)

在这种情况下，如果消息尚未存在，直接将其插入表格可以完美运行。如果已经存在，ORM会抛出一个异常，表示违反了唯一性约束。这种方法实现了与[清单12-1](ch12.xhtml#ch12list1)相同的效果，但更加高效，并且没有任何竞态条件。这是一个非常简单的模式，并且与任何ORM都不会发生冲突。问题在于，开发者倾向于把SQL数据库当作傻瓜存储，而不是当作他们可以用来实现数据完整性和一致性的工具；因此，他们可能会在控制器代码中重复编写SQL中已经存在的约束，而不是在模型中编写。

将 SQL 后端视为模型 API 是有效利用它的好方法。你可以通过在其自身的过程语言中编写简单的函数调用来操作存储在 RDBMS 中的数据。

### **数据库后端**

ORM 支持多种数据库后端。没有任何 ORM 库能够完全抽象所有关系型数据库管理系统（RDBMS）的特性，将代码简化到最基本的 RDBMS 会使得无法使用任何高级 RDBMS 功能，且不破坏抽象层。即使是 SQL 中没有标准化的简单操作，如处理时间戳操作，使用 ORM 时也会很麻烦。如果你的代码与特定的 RDBMS 无关，那么这一点尤为明显。选择应用程序的 RDBMS 时，务必牢记这一点。

隔离 ORM 库（如在 “[外部库](ch02.xhtml#lev1sec10)” 中描述的 [第 22 页](ch02.xhtml#page_22)）有助于缓解潜在问题。这种方法使你可以轻松地在需要时将 ORM 库替换为其他库，并通过识别查询效率低的地方来优化 SQL 使用，从而绕过大部分 ORM 样板代码。

例如，你可以在应用程序的某个模块中使用 ORM，比如 myapp.storage，从而轻松实现这种隔离。这个模块应该只导出允许你在高级抽象层次上操作数据的函数和方法。ORM 应该仅从该模块中使用。任何时候，你都能通过提供相同 API 的任何模块来替换 myapp.storage。

Python 中最常用的 ORM 库（也可以说是事实上的标准）是 sqlalchemy。这个库支持大量的后端，并为大多数常见操作提供抽象。架构升级可以通过 alembic 等第三方包来处理（*[https://pypi.python.org/pypi/alembic/](https://pypi.python.org/pypi/alembic/)*）。

一些框架，如 Django (*[https://www.djangoproject.com](https://www.djangoproject.com)*), 提供了自己的 ORM 库。如果你选择使用框架，最好使用内置的库，因为它通常会比外部库更好地与框架集成。

**警告**

*大多数框架所依赖的模块视图控制器（MVC）架构很容易被误用。这些框架直接在其模型中实现（或使实现变得容易）ORM，但并没有充分抽象化：任何在视图和控制器中使用模型的代码，也将直接使用 ORM。你需要避免这种情况。你应该编写一个包含 ORM 库的数据模型，而不是由它构成。这样做可以提供更好的可测试性和隔离性，并且使得将 ORM 替换为其他存储技术变得更加容易。*

### **使用 Flask 和 PostgreSQL 进行数据流处理**

在这里，我将展示如何使用 *PostgreSQL* 的高级功能来构建一个 HTTP 事件流系统，帮助掌握你的数据存储。

#### ***编写数据流应用程序***

[Listing 12-2](ch12.xhtml#ch12list2) 中的微应用程序的目的是将消息存储在 SQL 表中，并通过 HTTP REST API 提供对这些消息的访问。每条消息包括一个频道号、一个来源字符串和一个内容字符串。

CREATE TABLE message (

id SERIAL PRIMARY KEY,

channel INTEGER NOT NULL,

source TEXT NOT NULL,

content TEXT NOT NULL

);

*Listing 12-2：用于存储消息的 SQL 表模式*

我们还希望将这些消息流式传输到客户端，以便它可以实时处理它们。为此，我们将使用 PostgreSQL 的 LISTEN 和 NOTIFY 功能。这些功能允许我们监听由我们提供的函数发送的消息，而 PostgreSQL 会执行这些函数：

➊ CREATE OR REPLACE FUNCTION notify_on_insert() RETURNS trigger AS $$

➋ BEGIN

PERFORM pg_notify('channel_' || NEW.channel,

CAST(row_to_json(NEW) AS TEXT));

RETURN NULL;

END;

$$ LANGUAGE plpgsql;

这段代码创建了一个用 pl/pgsql 编写的触发器函数，这是一种只有 PostgreSQL 理解的语言。请注意，我们也可以用其他语言编写此函数，例如 Python 本身，因为 PostgreSQL 内嵌了 Python 解释器来提供 pl/python 语言。我们在这里执行的单一简单操作并不需要使用 Python，因此坚持使用 pl/pgsql 是一个明智的选择。

函数 notify_on_insert() ➊ 调用了 pg_notify() ➋，这是实际发送通知的函数。第一个参数是表示 *频道* 的字符串，而第二个参数是携带实际 *载荷* 的字符串。我们根据行中的频道列的值动态定义频道。在这种情况下，载荷将是整个行的 JSON 格式。是的，PostgreSQL 原生知道如何将一行转换为 JSON！

接下来，我们希望在每次在消息表中执行 INSERT 操作时发送通知消息，因此我们需要在此类事件上触发此函数：

CREATE TRIGGER notify_on_message_insert AFTER INSERT ON message

FOR EACH ROW EXECUTE PROCEDURE notify_on_insert();

该函数现在已插入，并将在每次在消息表中执行成功的 INSERT 操作时被执行。

我们可以通过在 psql 中使用 LISTEN 操作来检查它是否正常工作：

$ psql

psql (9.3rc1)

SSL 连接（加密算法：DHE-RSA-AES256-SHA，位数：256）

输入 "help" 获取帮助。

mydatabase=> LISTEN channel_1;

LISTEN

mydatabase=> INSERT INTO message(channel, source, content)

mydatabase-> VALUES(1, 'jd', 'hello world');

INSERT 0 1

异步通知 "channel_1" 载荷

"{"id":1,"channel":1,"source":"jd","content":"hello world"}"

从服务器进程（PID 26393）接收。

一旦行被插入，通知就会发送，我们可以通过 PostgreSQL 客户端接收到它。现在我们只需构建一个 Python 应用程序来流式传输这个事件，如 [Listing 12-3](ch12.xhtml#ch12list3) 所示。

import psycopg2

import psycopg2.extensions

import select

conn = psycopg2.connect(database='mydatabase', user='myuser',

password='idkfa', host='localhost')

conn.set_isolation_level(

psycopg2.extensions.ISOLATION_LEVEL_AUTOCOMMIT)

curs = conn.cursor()

curs.execute("LISTEN channel_1;")

while True:

select.select([conn], [], [])

conn.poll()

while conn.notifies:

notify = conn.notifies.pop()

print("收到 NOTIFY:", notify.pid, notify.channel,

notify.payload)

*示例 12-3：监听并接收通知流*

[示例 12-3](ch12.xhtml#ch12list3) 使用 psycopg2 库连接到 PostgreSQL。psycopg2 库是一个 Python 模块，实现在 PostgreSQL 网络协议，并允许我们连接到 PostgreSQL 服务器以发送 SQL 请求并接收结果。我们本可以使用提供抽象层的库，例如 sqlalchemy，但抽象库不提供 PostgreSQL 的 LISTEN 和 NOTIFY 功能。需要注意的是，即使使用像 sqlalchemy 这样的库，仍然可以访问底层数据库连接来执行代码，但对于这个示例来说这样做没有意义，因为我们不需要 ORM 库提供的其他功能。

程序监听 channel_1，一旦接收到通知，就将其打印到屏幕上。如果我们运行程序并在消息表中插入一行数据，我们将看到以下输出：

$ python listen.py

收到 NOTIFY: 28797 channel_1

{"id":10,"channel":1,"source":"jd","content":"hello world"}

一旦我们插入行数据，PostgreSQL 会触发触发器并发送通知。我们的程序接收它并打印通知负载；在这里，就是将行数据序列化为 JSON。现在我们具备了接收数据库中插入数据的基本能力，而无需执行额外的请求或操作。

#### ***构建应用程序***

接下来，我们将使用*Flask*，一个简单的 HTTP 微框架，来构建我们的应用程序。我们将构建一个 HTTP 服务器，利用 HTML5 定义的 *Server-Sent Events* 消息协议来流式传输插入的数据。另一种选择是使用 HTTP/1.1 定义的 *Transfer-Encoding: chunked*：

import flask

import psycopg2

import psycopg2.extensions

import select

app = flask.Flask(__name__)

def stream_messages(channel):

conn = psycopg2.connect(database='mydatabase', user='mydatabase',

password='mydatabase', host='localhost')

conn.set_isolation_level(

psycopg2.extensions.ISOLATION_LEVEL_AUTOCOMMIT)

curs = conn.cursor()

curs.execute("LISTEN channel_%d;" % int(channel))

while True:

select.select([conn], [], [])

conn.poll()

while conn.notifies:

notify = conn.notifies.pop()

yield "data: " + notify.payload + "\n\n"

@app.route("/message/<channel>", methods=['GET'])

def get_messages(channel):

return flask.Response(stream_messages(channel),

mimetype='text/event-stream')

if __name__ == "__main__":

app.run()

这个应用程序足够简单，支持流式传输，但不支持其他任何数据检索操作。我们使用 Flask 来路由 HTTP 请求 GET /message/channel 到我们的流式代码。一旦代码被调用，应用程序会返回一个 mimetype 为 text/event-stream 的响应，并返回一个生成器函数，而不是字符串。Flask 会调用这个函数，并在每次生成器产生数据时发送结果。

生成器`stream_messages()`重用了我们之前写的代码，用于监听 PostgreSQL 通知。它将频道标识符作为参数，监听该频道，然后生成有效载荷。记住我们在触发函数中使用了 PostgreSQL 的 JSON 编码功能，所以我们已经从 PostgreSQL 收到 JSON 数据。我们不需要对数据进行转码，因为将 JSON 数据发送到 HTTP 客户端是没问题的。

**注意**

*为了简单起见，这个示例应用程序被写在一个文件中。如果这是一个真实的应用程序，我会将存储处理实现移到自己的 Python 模块中。*

我们现在可以运行服务器了：

$ python listen+http.py

* 正在运行于 http://127.0.0.1:5000/

在另一个终端，我们可以连接并检索正在输入的事件。连接时没有收到任何数据，连接保持打开状态：

$ curl -v http://127.0.0.1:5000/message/1

* 即将连接()到 127.0.0.1 端口 5000 (#0)

*   尝试连接 127.0.0.1...

* 正在添加句柄：conn: 0x1d46e90

* 正在添加句柄：send: 0

* 正在添加句柄：recv: 0

* Curl_addHandleToPipeline: 长度: 1

* - Conn 0 (0x1d46e90) send_pipe: 1, recv_pipe: 0

* 已连接到 127.0.0.1 (127.0.0.1) 端口 5000 (#0)

> GET /message/1 HTTP/1.1
> 
> 用户代理：curl/7.32.0
> 
> 主机：127.0.0.1:5000
> 
> 接受：*/*

但是，一旦我们在消息表中插入一些行，我们就会开始在运行 curl 的终端看到数据进入。在第三个终端中，我们向数据库插入一条消息：

mydatabase=> INSERT INTO message(channel, source, content)

mydatabase-> VALUES(1, 'jd', 'hello world');

INSERT 0 1

mydatabase=> INSERT INTO message(channel, source, content) mydatabase-> VALUES(1, 'jd', 'it works');

INSERT 0 1

以下是数据输出：

数据：{"id":71,"channel":1,"source":"jd","content":"hello world"}

数据：{"id":72,"channel":1,"source":"jd","content":"it works"}

这些数据会打印到运行 curl 的终端。这样，curl 会保持与 HTTP 服务器的连接，同时等待下一波消息流。在这里，我们创建了一个流式服务，而没有进行任何类型的轮询，建立了一个完全基于 *推送* 的系统，信息在不同点之间无缝流动。

一个天真且可以说更具可移植性的应用程序实现会反复循环执行 SELECT 语句，通过轮询表中插入的新数据。这对于任何不支持发布-订阅模式的存储系统（像这个例子一样）也能起作用。

### **Dimitri Fontaine 关于数据库的讨论**

Dimitri是一个熟练的PostgreSQL主要贡献者，现工作于Citus Data，并在*pgsql-hackers*邮件列表中与其他数据库专家进行辩论。我们分享了许多开源项目，他也很友好地回答了一些关于处理数据库时应该做什么的问题。

**你会给使用RDBMS作为存储后端的开发者什么建议？**

关系数据库管理系统（RDBMS）是在70年代发明的，旨在解决当时每个应用程序开发者所面临的一些常见问题，而RDBMS所实现的主要服务不仅仅是数据存储。

关系数据库管理系统（RDBMS）提供的主要服务实际上是以下内容：

+   **并发：** 你可以通过任意多个并发执行线程来访问数据进行读取或写入——RDBMS会正确处理这些操作。这是你希望RDBMS提供的主要特性。

+   **并发语义：** 使用关系数据库管理系统时的并发行为细节是通过一个高级规范来提出的，通常涉及原子性和隔离性，这可能是ACID（原子性、一致性、隔离性、持久性）中最关键的部分。*原子性*是指在你开始一个事务（BEGIN）到你完成它（无论是提交COMMIT还是回滚ROLLBACK）之间，系统不允许其他并发活动知道你正在做什么——不管是什么。使用合适的RDBMS时，还应包括数据定义语言（DDL），例如CREATE TABLE或ALTER TABLE。*隔离性*则涉及在自己的事务中，你允许观察到的系统并发活动的程度。SQL标准定义了四个隔离级别，具体内容请参见PostgreSQL文档中的描述 (*[http://www.postgresql.org/docs/9.2/static/transaction-iso.html](http://www.postgresql.org/docs/9.2/static/transaction-iso.html)*).

关系数据库管理系统（RDBMS）对你的数据负全责。因此，它允许开发者描述自己的一致性规则，然后会在关键时刻检查这些规则是否有效，比如在事务提交时或语句边界时，具体取决于约束声明的可推迟性。

你可以对数据施加的第一个约束是其预期的输入和输出格式，使用适当的数据类型。关系数据库管理系统（RDBMS）将知道如何处理比文本、数字和日期更多的数据类型，并且能够正确处理今天实际使用中的日历日期。

数据类型不仅仅与输入和输出格式有关。它们还实现了行为和某种程度的多态性，因为我们都期望基本的相等性测试是特定于数据类型的：我们不会将文本与数字、日期与IP地址、数组与范围等进行同样的比较。

保护你的数据也意味着，关系数据库管理系统（RDBMS）唯一的选择是主动拒绝不符合一致性规则的数据，其中第一个规则就是你选择的数据类型。如果你认为处理一个如 0000-00-00 这样的日期（在日历中从未存在过）是可以接受的，那么你需要重新考虑。

一致性保证的另一部分通过约束的形式来表达，例如 CHECK 约束、NOT NULL 约束和约束触发器，其中之一就是外键。所有这些可以被看作是数据类型定义和行为的用户级扩展，主要区别在于，你可以选择将这些约束的检查推迟执行，从每个语句的结束推迟到当前事务的结束。

RDBMS 的关系部分专注于建模你的数据，并保证所有在关系中找到的元组共享一套共同的规则：结构和约束。在强制执行这些时，我们实际上是在强制使用适当的显式模式来处理我们的数据。

为你的数据设计合适的模式称为 *规范化*，在设计中，你可以目标多个微妙不同的规范形式。然而，有时你需要比规范化过程的结果所提供的更大的灵活性。常见的做法是首先规范化你的数据模式，然后再修改它以恢复一些灵活性。很有可能你其实并不需要更多的灵活性。

当你需要更多的灵活性时，可以使用 PostgreSQL 来尝试多种去规范化选项：复合类型、记录、数组、H-Store、JSON 或 XML 等。

然而，去规范化有一个非常重要的缺点，那就是我们接下来要讨论的查询语言是为处理规范化数据而设计的。当然，在 PostgreSQL 中，查询语言已经扩展，以支持尽可能多的去规范化，当使用复合类型、数组或 H-Store，甚至在最近的版本中支持 JSON 时。

RDBMS 了解你的数据，能够帮助你实现非常细粒度的安全模型，如果你需要的话。访问模式在关系和列级别进行管理，PostgreSQL 还实现了 SECURITY DEFINER 存储过程，允许你以非常受控的方式提供对敏感数据的访问，类似于使用保存的用户 ID（SUID）程序。

RDBMS 提供通过 SQL 访问你的数据，SQL 在 80 年代成为事实标准，现在由一个委员会推动。在 PostgreSQL 的情况下，很多扩展被添加进来，每次主要发布都允许你访问一个非常丰富的领域特定语言（DSL）。所有查询规划和优化的工作都由 RDBMS 为你完成，这样你就可以专注于声明式查询，描述你希望从现有数据中获得的结果。

这也是为什么你需要密切关注这里的NoSQL产品，因为这些时髦的产品实际上并不仅仅是去除了SQL，而是去除了你所习惯的许多其他基础设施。

**你会给使用关系型数据库管理系统（RDBMS）作为存储后端的开发者什么建议？**

我的建议是，记住*存储后端*和RDBMS之间的区别。这是两种非常不同的服务，如果你需要的仅仅是存储后端，也许考虑使用其他非RDBMS的服务。

然而，大多数时候，你真正需要的是一个功能齐全的RDBMS。在这种情况下，你最好的选择是PostgreSQL。去阅读它的文档 (*[https://www.postgresql.org/docs/](https://www.postgresql.org/docs/)*); 查看它提供的数据类型、运算符、函数、特性和扩展列表。阅读一些博客上的使用示例。

然后，将PostgreSQL视为你可以在开发中利用的工具，并将其纳入你的应用架构中。你需要实现的部分服务最好由RDBMS层提供，而PostgreSQL在作为整个实现中的可靠部分方面表现出色。

**使用或不使用ORM的最佳方式是什么？**

ORM最适合用于*CRUD*应用：创建、读取、更新和删除。读取部分应仅限于一个非常简单的SELECT语句，目标是单个表，因为检索不必要的更多列会对查询性能和资源使用产生显著影响。

从RDBMS中检索的任何列，如果最终没有使用，都是对宝贵资源的纯粹浪费，是首要的可扩展性杀手。即使你的ORM能够仅提取你请求的数据，你仍然必须在每种情况下管理精确的列列表，而不能使用一个简单的抽象方法来自动为你计算字段列表。

创建、更新和删除查询是简单的INSERT、UPDATE和DELETE语句。许多RDBMS提供了ORM无法利用的优化，例如在INSERT之后返回数据。

此外，在一般情况下，关系要么是一个表，要么是任何查询的结果。在使用ORM时，通常会在已定义的表和某些模型类之间建立关系映射，或者与其他辅助存根之间建立映射。

如果你考虑SQL语义的整体，那么关系映射器应该能够将任何查询映射到一个类。你可能需要为你想要执行的每个查询构建一个新的类。

这个思想应用到我们的案例中就是，你信任你的ORM比你自己更能高效地编写SQL查询，即使你没有给它足够的信息来计算出你感兴趣的确切数据集。

诚然，有时SQL会变得相当复杂，但通过使用一个你无法控制的API-to-SQL生成器，你绝不会接近简单。

然而，有两种情况你可以放松并使用ORM，前提是你愿意接受以下妥协：稍后你可能需要将ORM的使用从代码库中移除。

+   **上市时间：** 当你急需推出产品并尽快获得市场份额时，唯一的办法就是发布应用程序和创意的第一个版本。如果你的团队在使用ORM方面比手写SQL查询更熟练，那就直接这么做吧。不过，你必须意识到，一旦你的应用成功了，首先要解决的扩展性问题之一就是ORM生成的查询非常糟糕。此外，你使用ORM的方式会让你陷入困境，并导致糟糕的代码设计决策。但如果你到了那个阶段，说明你已经足够成功，可以花一些重构的费用来去除对ORM的依赖，对吧？

+   **CRUD应用：** 这是真正的应用场景，你只会一次编辑一个元组，而不太关心性能问题，例如在基本的管理应用界面中。

**在使用Python时，选择PostgreSQL而非其他数据库的优点是什么？**

以下是我选择PostgreSQL作为开发者的主要理由：

+   **社区支持：** PostgreSQL社区庞大且热情欢迎新用户，大家通常会花时间提供最佳答案。邮件列表仍然是与社区沟通的最佳方式。

+   **数据完整性和持久性：** 你发送到PostgreSQL的任何数据都是安全的，数据的定义和你之后再获取它的能力都能得到保证。

+   **数据类型、函数、运算符、数组和范围：** PostgreSQL提供了丰富的数据类型，附带许多运算符和函数。即便使用数组或JSON数据类型进行非规范化，仍然能够编写高级查询，包括对它们进行JOIN操作。

+   **查询规划器和优化器：** 值得花时间理解这些组件有多复杂和强大。

+   **事务性DDL：** 你几乎可以回滚任何命令。现在试试看：打开你的psql终端，连接到一个数据库，然后输入BEGIN; DROP TABLE foo; ROLLBACK;，其中将foo替换为本地实例中存在的表名。不可思议吧？

+   **PL/Python（以及其他如C、SQL、Javascript或Lua）：** 你可以在服务器上运行自己的Python代码，直接在数据所在的位置处理它，这样就不需要通过网络获取数据来处理，然后再通过查询将结果发送回来执行下一层的JOIN操作。

+   **特定索引（GiST、GIN、SP-GiST、部分和函数索引）：** 你可以创建Python函数来处理PostgreSQL中的数据，然后对调用该函数的结果进行索引。当你发出带有WHERE子句的查询时，该函数只会对查询中的数据调用一次；然后，它直接与索引的内容进行匹配。
