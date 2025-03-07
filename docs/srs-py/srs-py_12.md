## 第十三章：**管理关系型数据库**

![image](img/common01.jpg)

应用程序几乎总是需要存储某种数据，开发者通常会将关系型数据库管理系统（RDBMS）与某种类型的对象关系映射工具（ORM）结合使用。RDBMS 和 ORM 可能很复杂，并且对于许多开发者来说并不是他们最喜欢的话题，但迟早需要面对。

### **关系型数据库管理系统（RDBMS）、ORM 以及何时使用它们**

RDBMS 是存储应用程序关系数据的数据库。开发者将使用类似 SQL（结构化查询语言）这样的语言来处理关系代数，也就是说，像这样的语言处理数据管理和数据之间的关系。两者结合使用，可以让你既能存储数据，又能尽可能高效地查询数据以获取特定的信息。对关系型数据库结构有良好的理解，例如如何正确使用规范化或不同类型的可串行性，可能会帮助你避免掉入许多陷阱。显然，这些主题值得写一本完整的书籍，而不会在本章中完全涵盖；相反，我们将专注于通过其常用的编程语言 SQL 来使用数据库。

开发者可能不愿意投入时间学习一门全新的编程语言来与 RDBMS 交互。如果是这样，他们通常会完全避免编写 SQL 查询，而是依赖于某个库来为他们完成这项工作。ORM 库在编程语言生态系统中非常常见，Python 也不例外。

ORM 的目的是通过抽象化创建查询的过程，使数据库系统更容易访问：它生成 SQL，这样你就不需要手动写了。不幸的是，这种抽象层可能会妨碍你执行一些更具体或底层的任务，这些任务是 ORM 根本无法做到的，比如编写复杂的查询。

在面向对象程序中使用 ORM 也有一组特定的难题，这些问题非常常见，通常被统称为 *对象关系阻抗不匹配*。这种阻抗不匹配发生在关系型数据库和面向对象程序对数据的表示不同，彼此之间无法正确映射：无论你怎么做，将 SQL 表映射到 Python 类都无法得到最佳结果。

理解 SQL 和 RDBMS 将使你能够编写自己的查询，而无需依赖于抽象层来做所有事情。

但这并不是说你应该完全避免使用 ORM。ORM 库可以帮助你快速原型化应用程序模型，一些库甚至提供有用的工具，比如架构升级和降级。重要的是要理解，使用 ORM 并不能替代你对 RDBMS 的真正理解：许多开发者尝试用自己选择的语言来解决问题，而不是使用它们的模型 API，而他们提出的解决方案充其量也只是勉强凑合。

**注意**

*本章假设你了解基本的 SQL。引入 SQL 查询并讨论表的工作原理超出了本书的范围。如果你是 SQL 新手，建议在继续之前先学习基本内容。Anthony DeBarros 的《实用 SQL》（No Starch Press，2018）是一个很好的起点。*

让我们看一个例子，演示为什么理解关系型数据库管理系统（RDBMS）可以帮助你编写更好的代码。假设你有一个用于跟踪消息的 SQL 表。这张表有一个名为`id`的列，表示消息发送者的 ID，这是主键，还有一个包含消息内容的字符串，像这样：

```py
CREATE TABLE message (
  id serial PRIMARY KEY,
  content text
);
```

我们希望检测任何重复的消息并将其从数据库中排除。为此，典型的开发者可能会使用 ORM 编写 SQL，如列表 12-1 所示。

```py
if query.select(Message).filter(Message.id == some_id):
    # We already have the message, it's a duplicate, ignore and raise
    raise DuplicateMessage(message)
else:
    # Insert the message
    query.insert(message)
```

*列表 12-1：使用 ORM 检测并排除重复消息*

这段代码适用于大多数情况，但它有一些主要的缺点：

+   重复约束已经在 SQL 模式中表达，因此这里存在一种代码重复：使用`PRIMARY KEY`隐式定义了`id`字段的唯一性。

+   如果消息尚未在数据库中，这段代码会执行两条 SQL 查询：一条`SELECT`语句，然后是`INSERT`语句。执行 SQL 查询可能需要很长时间，并且可能需要与 SQL 服务器进行来回通信，造成额外的延迟。

+   这段代码没有考虑到在我们调用`select_by_id()`之后、调用`insert()`之前，其他人可能会插入重复的消息，这样会导致程序抛出异常。这个漏洞被称为*竞态条件*。

有一种更好的方式来编写这段代码，但它需要与 RDBMS 服务器协作。我们可以直接插入消息，并使用`try...except`块来捕捉重复冲突，而不是先检查消息是否存在然后再插入：

```py
try:
    # Insert the message
    message_table.insert(message)
except UniqueViolationError:
    # Duplicate
    raise DuplicateMessage(message)
```

在这种情况下，如果消息尚不存在，直接将消息插入表中可以完美运行。如果已存在，ORM 会抛出一个异常，指示违反了唯一性约束。此方法与列表 12-1 达到相同的效果，但更加高效且没有任何竞态条件。这是一个非常简单的模式，并且它与任何 ORM 都不会冲突。问题在于开发者倾向于将 SQL 数据库视为“愚蠢的存储”而不是一个可以用来获得数据完整性和一致性的工具；因此，他们可能会在控制器代码中而不是在模型中重复写 SQL 中的约束。

将你的 SQL 后端当作模型 API 来使用是高效利用它的好方法。你可以通过用其自身的过程式语言编写简单的函数调用来操作存储在 RDBMS 中的数据。

### **数据库后端**

ORM 支持多个数据库后端。没有哪个 ORM 库能够完全抽象所有 RDBMS 的特性，而将代码简化到最基础的 RDBMS 功能将使得在不打破抽象层的情况下使用任何高级 RDBMS 功能变得不可能。即使是一些在 SQL 中没有标准化的简单操作，比如处理时间戳操作，在使用 ORM 时也会变得很麻烦。如果你的代码是 RDBMS 无关的，这一点尤其如此。在选择应用程序的 RDBMS 时，记住这一点非常重要。

隔离 ORM 库（如在《外部库》中描述的，第 22 页）有助于减轻潜在问题。这个方法允许你在需要时轻松更换 ORM 库，并通过识别低效查询的地方来优化 SQL 使用，从而避免大部分 ORM 的样板代码。

例如，你可以在应用程序的一个模块中使用 ORM，比如`myapp.storage`，轻松实现这种隔离。该模块应该只导出允许你在高抽象级别操作数据的函数和方法。ORM 应仅在该模块中使用。任何时候，你都可以插入任何提供相同 API 的模块来替代`myapp.storage`。

Python 中最常用的 ORM 库（可以说是事实上的标准）是`sqlalchemy`。这个库支持大量的后端，并为大多数常见操作提供了抽象。模式升级可以通过第三方包来处理，例如`alembic`（*[`pypi.python.org/pypi/alembic/`](https://pypi.python.org/pypi/alembic/)*）。

一些框架，比如 Django（*[`www.djangoproject.com`](https://www.djangoproject.com)*），提供了它们自己的 ORM 库。如果你选择使用框架，最好使用内建库，因为它通常与框架的集成效果要优于外部库。

**警告**

*大多数框架依赖的模块视图控制器（MVC）架构很容易被滥用。这些框架直接在模型中实现（或方便实现）ORM，但没有足够地进行抽象：任何在视图和控制器中使用模型的代码，也将直接使用 ORM。你需要避免这种情况。你应该编写一个包含 ORM 库的数据模型，而不是由它构成。这样做可以提供更好的可测试性和隔离性，并使得用另一种存储技术替换 ORM 变得更加容易。*

### **使用 Flask 和 PostgreSQL 进行数据流处理**

在这里，我将向你展示如何利用*PostgreSQL*的高级特性构建一个 HTTP 事件流系统，帮助你掌握数据存储。

#### ***编写数据流处理应用程序***

清单 12-2 中微型应用程序的目的是将消息存储在 SQL 表中，并通过 HTTP REST API 提供对这些消息的访问。每条消息由一个频道号、一个源字符串和一个内容字符串组成。

```py
CREATE TABLE message (
  id SERIAL PRIMARY KEY,
  channel INTEGER NOT NULL,
  source TEXT NOT NULL,
  content TEXT NOT NULL
);
```

*清单 12-2：用于存储消息的 SQL 表架构*

我们还希望将这些消息流式传输到客户端，以便它可以实时处理它们。为此，我们将使用 PostgreSQL 的 `LISTEN` 和 `NOTIFY` 功能。这些功能允许我们监听 PostgreSQL 执行的函数发送的消息：

```py
➊ CREATE OR REPLACE FUNCTION notify_on_insert() RETURNS trigger AS $$
➋ BEGIN
     PERFORM pg_notify('channel_' || NEW.channel,
                       CAST(row_to_json(NEW) AS TEXT));
     RETURN NULL;
   END;
   $$ LANGUAGE plpgsql;
```

这段代码创建了一个用 pl/pgsql 编写的触发器函数，这是 PostgreSQL 独有的一种语言。请注意，我们也可以使用其他语言编写此函数，例如 Python 本身，因为 PostgreSQL 嵌入了 Python 解释器以提供 pl/python 语言。我们在这里执行的单一简单操作并不需要利用 Python，因此坚持使用 pl/pgsql 是一个明智的选择。

函数 `notify_on_insert()` ➊ 调用了 `pg_notify()` ➋，该函数实际上发送了通知。第一个参数是表示 *频道* 的字符串，而第二个参数是承载实际 *有效载荷* 的字符串。我们根据行中频道列的值动态定义频道。在这种情况下，有效载荷将是整行数据的 JSON 格式。是的，PostgreSQL 确实知道如何将一行数据转换为 JSON！

接下来，我们希望在每次在消息表中执行 `INSERT` 时发送通知消息，因此我们需要在此类事件上触发该函数：

```py
CREATE TRIGGER notify_on_message_insert AFTER INSERT ON message
FOR EACH ROW EXECUTE PROCEDURE notify_on_insert();
```

该函数现在已经连接并将在每次在消息表中执行成功的`INSERT`时被执行。

我们可以通过在 `psql` 中使用 `LISTEN` 操作来检查它是否工作正常：

```py
$ psql
psql (9.3rc1)
SSL connection (cipher: DHE-RSA-AES256-SHA, bits: 256)
Type "help" for help.

mydatabase=> LISTEN channel_1;
LISTEN
mydatabase=> INSERT INTO message(channel, source, content)
mydatabase-> VALUES(1, 'jd', 'hello world');
INSERT 0 1
Asynchronous notification "channel_1" with payload
"{"id":1,"channel":1,"source":"jd","content":"hello world"}"
received from server process with PID 26393.
```

一旦行被插入，通知便会被发送，并且我们能够通过 PostgreSQL 客户端接收到它。现在我们只需构建流式传输该事件的 Python 应用程序，如 清单 12-3 所示。

```py
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
        print("Got NOTIFY:", notify.pid, notify.channel,
notify.payload)
```

*清单 12-3：监听并接收通知流*

清单 12-3 使用 `psycopg2` 库连接到 PostgreSQL。`psycopg2` 库是一个 Python 模块，实现了 PostgreSQL 网络协议，并允许我们连接到 PostgreSQL 服务器发送 SQL 请求并接收结果。我们本可以使用提供抽象层的库，例如 `sqlalchemy`，但是抽象库并没有提供访问 PostgreSQL 的 `LISTEN` 和 `NOTIFY` 功能。需要注意的是，使用像 `sqlalchemy` 这样的库时，仍然可以访问底层数据库连接来执行代码，但对于这个示例来说这样做没有意义，因为我们不需要 ORM 库提供的其他功能。

程序监听 `channel_1`，一旦收到通知，就会将其打印到屏幕上。如果我们运行程序并在 `message` 表中插入一行数据，我们会得到以下输出：

```py
$ python listen.py
Got NOTIFY: 28797 channel_1
{"id":10,"channel":1,"source":"jd","content":"hello world"}
```

一旦我们插入行，PostgreSQL 会运行触发器并发送通知。我们的程序接收到通知并打印通知负载；在这里，通知负载就是被序列化为 JSON 的行。现在，我们具备了接收插入数据库的数据的基本能力，而无需做任何额外的请求或工作。

#### ***构建应用程序***

接下来，我们将使用 *Flask*，一个简单的 HTTP 微框架，来构建我们的应用程序。我们将构建一个 HTTP 服务器，使用 HTML5 定义的 *Server-Sent Events* 消息协议流式传输 `insert` 数据流。另一种选择是使用 HTTP/1.1 定义的 *Transfer-Encoding: chunked*：

```py
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
```

这个应用程序足够简单，支持流式传输，但不支持其他数据检索操作。我们使用 Flask 将 HTTP 请求 `GET /message/channel` 路由到我们的流式代码。一旦调用该代码，应用程序会返回一个具有 `text/event-stream` MIME 类型的响应，并返回一个生成器函数，而不是一个字符串。Flask 会调用这个函数，并在每次生成器生成内容时发送结果。

生成器 `stream_messages()` 重用了我们之前写的代码来监听 PostgreSQL 通知。它接收频道标识符作为参数，监听该频道，然后返回负载。记住，我们在触发器函数中使用了 PostgreSQL 的 JSON 编码函数，所以我们已经从 PostgreSQL 接收到了 JSON 数据。我们不需要重新编码数据，因为直接将 JSON 数据发送给 HTTP 客户端是可以的。

**注意**

*为了简化起见，这个示例应用程序是写在一个文件中的。如果这是一个真正的应用程序，我会将存储处理实现移动到它自己的 Python 模块中。*

我们现在可以运行服务器：

```py
$ python listen+http.py
 * Running on http://127.0.0.1:5000/
```

在另一个终端中，我们可以连接并检索已输入的事件。连接时，没有数据接收，并且连接保持打开状态：

```py
$ curl -v http://127.0.0.1:5000/message/1
* About to connect() to 127.0.0.1 port 5000 (#0)
*   Trying 127.0.0.1...
* Adding handle: conn: 0x1d46e90
* Adding handle: send: 0
* Adding handle: recv: 0
* Curl_addHandleToPipeline: length: 1
* - Conn 0 (0x1d46e90) send_pipe: 1, recv_pipe: 0
* Connected to 127.0.0.1 (127.0.0.1) port 5000 (#0)
> GET /message/1 HTTP/1.1
> User-Agent: curl/7.32.0
> Host: 127.0.0.1:5000
> Accept: */*
>
```

但是，一旦我们在 `message` 表中插入一些行，我们会开始通过运行 `curl` 的终端看到数据流入。在第三个终端中，我们在数据库中插入一条消息：

```py
mydatabase=> INSERT INTO message(channel, source, content)
mydatabase-> VALUES(1, 'jd', 'hello world');
INSERT 0 1
mydatabase=> INSERT INTO message(channel, source, content) mydatabase-> VALUES(1, 'jd', 'it works');
INSERT 0 1
```

这是数据输出：

```py
data: {"id":71,"channel":1,"source":"jd","content":"hello world"}
data: {"id":72,"channel":1,"source":"jd","content":"it works"}
```

这些数据会打印到运行 `curl` 的终端中。这样就保持了 `curl` 与 HTTP 服务器的连接，同时等待下一个消息流的到来。我们创建了一个流式服务，而没有做任何轮询，建立了一个完全基于 *推送* 的系统，信息在一个点到另一个点之间无缝流动。

一个天真且可能更具可移植性的实现会反复循环执行 `SELECT` 语句，以轮询表中插入的新数据。这种方式适用于任何不支持发布-订阅模式的存储系统，而不是像本例中所示的这种方式。

### **Dimitri Fontaine 论数据库**

Dimitri 是一个熟练的 PostgreSQL 主要贡献者，现任 Citus Data 的工程师，并在*pgsql-hackers*邮件列表上与其他数据库专家进行辩论。我们分享了许多开源冒险经历，他也很友善地回答了一些关于处理数据库时应该做什么的问题。

**你会给使用 RDBMS 作为存储后端的开发者什么建议？**

RDBMS 是在 70 年代发明的，用于解决当时困扰每个应用程序开发者的常见问题，RDBMS 实现的主要服务不仅仅是数据存储。

RDBMS 提供的主要服务实际上如下：

+   **并发性：** 你可以使用任意数量的并发执行线程来访问数据进行读取或写入——RDBMS 会正确处理这些操作。这正是你从 RDBMS 中希望获得的主要功能。

+   **并发语义：** 使用 RDBMS 时的并发行为细节，通过高层次的规范提出，涉及原子性和隔离性，可能是 ACID（原子性、一致性、隔离性、持久性）中最关键的部分。*原子性*是指，在你`BEGIN`一个事务和事务完成时（无论是`COMMIT`还是`ROLLBACK`）之间，系统中其他任何并发活动都不能知道你在做什么——无论是什么。当使用一个合适的 RDBMS 时，还包括数据定义语言（DDL），例如`CREATE TABLE`或`ALTER TABLE`。*隔离性*则涉及你在自己事务内可以注意到的系统并发活动的内容。SQL 标准定义了四种隔离级别，如 PostgreSQL 文档中所述（* [`www.postgresql.org/docs/9.2/static/transaction-iso.html`](http://www.postgresql.org/docs/9.2/static/transaction-iso.html)*）。

RDBMS 对你的数据负全责。因此，它允许开发者描述自己的一致性规则，然后它会在关键时刻检查这些规则是否有效，比如在事务提交或语句边界时，具体取决于约束声明的延迟性。

你可以对数据施加的第一个约束是其预期的输入和输出格式，使用正确的数据类型。关系数据库管理系统（RDBMS）能够处理比文本、数字和日期更多的内容，并且能够正确处理实际出现在今天日历中的日期。

数据类型不仅仅涉及输入和输出格式，它们还实现了行为和某种程度的多态性，因为我们都期望基本的相等性测试是特定于数据类型的：我们不会以相同的方式比较文本和数字、日期和 IP 地址、数组和范围等。

保护你的数据也意味着，RDBMS 的唯一选择是主动拒绝不符合一致性规则的数据，其中第一个规则就是你选择的数据类型。如果你认为处理像 0000-00-00 这样的日期是可以接受的，这个日期在日历中从未存在过，那么你需要重新考虑。

一致性保证的另一部分是通过约束来表达的，例如`CHECK`约束、`NOT NULL`约束和约束触发器，其中一个叫做外键。所有这些可以被看作是数据类型定义和行为的用户级扩展，主要区别在于你可以选择将约束检查的强制执行从每个语句的末尾推迟到当前事务的末尾，使用`DEFER`来实现这一点。

关系型数据库管理系统（RDBMS）的关系位主要是关于建模你的数据，并确保在关系中的所有元组共享一套共同的规则：结构和约束。当执行这些规则时，我们是在强制使用一个合适的显式模式来处理我们的数据。

为你的数据制定一个合适的模式被称为*规范化*，你可以在设计中追求几个细微不同的规范形式。然而，有时你可能需要比规范化过程的结果提供更多的灵活性。常见的做法是首先规范化你的数据模式，然后再修改它以恢复一些灵活性。实际上，你很可能并不需要更多的灵活性。

当你确实需要更多灵活性时，你可以使用 PostgreSQL 尝试多种反规范化选项：复合类型、记录、数组、H-Store、JSON 或 XML 等。

然而，反规范化有一个非常重要的缺点，那就是我们接下来要讨论的查询语言是为处理相对规范化的数据而设计的。当然，使用 PostgreSQL 时，查询语言已被扩展，以支持尽可能多的反规范化，特别是在使用复合类型、数组、H-Store，甚至最近发布的 JSON 时。

RDBMS 了解你的数据，并且可以帮助你实现一个非常细粒度的安全模型，若你需要的话。访问模式在关系和列级别进行管理，PostgreSQL 还实现了`SECURITY DEFINER`存储过程，允许你以一种非常受控的方式提供对敏感数据的访问，类似于使用保存的用户 ID（SUID）程序的方式。

RDBMS 通过 SQL 提供数据访问，这在 80 年代成为事实上的标准，并且现在由一个委员会推动。在 PostgreSQL 的情况下，增加了许多扩展，每个主要版本都会让你访问一个非常丰富的 DSL 语言。所有查询规划和优化的工作都由 RDBMS 为你完成，这样你就可以专注于一个声明式查询，在这个查询中你只需要描述你想从数据中得到的结果。

这也是为什么你需要特别关注这里的 NoSQL 产品，因为大多数这些流行的产品实际上并不是仅仅去除了 SQL，而是移除了你习惯期望的许多其他基础设施。

**你会给使用 RDBMS 作为存储后端的开发人员什么建议？**

我的建议是记住*存储后端*和 RDBMS 之间的区别。这两者是非常不同的服务，如果你所需要的仅仅是存储后端，或许可以考虑使用 RDBMS 以外的其他工具。

然而，最常见的需求是一个完整的关系数据库管理系统（RDBMS）。在这种情况下，最佳的选择是 PostgreSQL。去阅读它的文档（* [`www.postgresql.org/docs/`](https://www.postgresql.org/docs/)*）；查看它提供的数据类型、运算符、函数、特性和扩展。阅读一些博客中的使用示例。

然后，可以将 PostgreSQL 视为你在开发中可以利用的工具，并将其纳入你的应用架构中。你需要实现的服务部分最好在 RDBMS 层面提供，而 PostgreSQL 在作为整个实现中值得信赖的一部分方面表现出色。

**使用或不使用 ORM 的最佳方式是什么？**

ORM 最适合用于*CRUD*应用程序：创建、读取、更新和删除。读取部分应该限制在一个非常简单的`SELECT`语句上，目标是单一表，因为获取不必要的更多列会对查询性能和使用的资源产生重大影响。

从 RDBMS 中检索到的任何列，如果最终没有使用，都是对宝贵资源的浪费，这是一个初步的可扩展性杀手。即使你的 ORM 能够仅获取你请求的数据，你仍然需要以某种方式管理你在每种情况下需要的确切列列表，而不是使用一个简单的抽象方法，它会自动为你计算字段列表。

创建、更新和删除查询是简单的`INSERT`、`UPDATE`和`DELETE`语句。许多 RDBMS 提供了 ORM 没有利用的优化，例如在`INSERT`之后返回数据。

此外，在一般情况下，关系要么是一个表，要么是任何查询的结果。在使用 ORM 时，常见的做法是建立定义好的表与一些模型类或其他辅助代码之间的关系映射。

如果你考虑 SQL 语义的整体性，那么关系映射器实际上应该能够将任何查询映射到一个类。然后，你大概需要为每个你想运行的查询构建一个新的类。

在我们的案例中应用这个理念的方式是，你信任你的 ORM 比你自己写更高效的 SQL 查询，即使你没有提供足够的信息来确定你感兴趣的确切数据集。

的确，SQL 有时可能会变得相当复杂，但通过使用一个你无法控制的 API 到 SQL 生成器，你是无法接近简单性的。

然而，在两个情况下，你可以放松并使用 ORM，前提是你愿意接受以下妥协：稍后你可能需要从代码库中删除 ORM 的使用。

+   **市场上市时间：** 当你非常急需并希望尽快抢占市场份额时，唯一的方法就是发布应用程序和创意的第一个版本。如果你的团队更擅长使用 ORM 而非手动编写 SQL 查询，那就尽管使用它。你必须意识到，一旦你的应用成功，最先需要解决的扩展性问题之一就是 ORM 生成的查询非常糟糕。同时，你使用 ORM 的方式可能已经将你限制在了一个死胡同，导致了糟糕的代码设计决策。但如果你已经走到这一步，说明你的应用已经足够成功，足以投入一些重构的资金来移除对 ORM 的依赖，对吧？

+   **CRUD 应用：** 这是实际的操作，其中你一次只编辑一个元组，并且你并不关心性能，适用于基本的管理员应用界面。

**在使用 Python 时，为什么选择 PostgreSQL 而不是其他数据库？**

以下是我选择 PostgreSQL 作为开发者的主要原因：

+   **社区支持：** PostgreSQL 的社区非常庞大且对新用户友好，大家通常会花时间提供最佳答案。邮件列表仍然是与社区沟通的最佳方式。

+   **数据完整性和持久性：** 你发送到 PostgreSQL 的任何数据都能在其定义中安全存储，并且你能够在之后重新获取它。

+   **数据类型、函数、操作符、数组和范围：** PostgreSQL 提供了一套非常丰富的数据类型，并且伴随有许多操作符和函数。即使是使用数组或 JSON 数据类型进行反规范化，仍然可以编写高级查询，包括对这些数据进行连接。

+   **规划器和优化器：** 值得花时间了解这些组件的复杂性和强大功能。

+   **事务性 DDL：** 几乎可以 `ROLLBACK` 任何命令。现在试试：只需打开 `psql` shell，连接到你拥有的数据库，然后输入 `BEGIN; DROP TABLE foo; ROLLBACK;`，其中将 foo 替换为本地实例中存在的一个表的名称。很神奇，对吧？

+   **PL/Python（以及其他语言，如 C、SQL、JavaScript 或 Lua）：** 你可以在服务器上运行自己的 Python 代码，数据就在那里，这样就不必通过网络提取数据进行处理，再发回查询进行下一步的 `JOIN` 操作。

+   **特定索引（GiST、GIN、SP-GiST、部分和函数索引）：** 你可以在 PostgreSQL 内部创建 Python 函数来处理数据，然后索引该函数调用的结果。当你发出带有 `WHERE` 子句的查询并调用该函数时，函数仅会使用查询中的数据执行一次，然后直接与索引内容进行匹配。
