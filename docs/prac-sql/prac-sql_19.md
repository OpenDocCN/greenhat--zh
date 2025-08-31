# 第十九章：维护你的数据库

![](img/chapterart.png)

在我们对 SQL 的探索结束时，我们将了解关键的数据库维护任务以及定制 PostgreSQL 的选项。本章中，你将学习如何跟踪和节省数据库空间，如何更改系统设置，以及如何备份和恢复数据库。你需要执行这些任务的频率取决于你当前的角色和兴趣。如果你想成为*数据库管理员*或*后端开发者*，这里所涵盖的主题至关重要。

值得注意的是，数据库维护和性能调优是足够庞大的话题，常常占据整本书的篇幅，而本章主要作为一些基本概念的介绍。如果你想深入了解，可以从附录中的资源开始。

让我们从 PostgreSQL 的`VACUUM`功能开始，它通过移除未使用的行来缩小表的大小。

## 使用 VACUUM 回收未使用的空间

PostgreSQL 的`VACUUM`命令帮助管理数据库的大小，正如第十章《更新大表时提升性能》一文中所讨论的，数据库的大小可能因日常操作而增长。

例如，当你更新行值时，数据库会创建该行的新版本，并保留（但隐藏）旧版本。PostgreSQL 文档将这些你看不见的行称为*死元组*，其中*元组*是 PostgreSQL 数据库中行的内部实现方式的名称。删除行时也会发生相同的情况。尽管该行对你不可见，但它仍作为死行存在于表中。

这是经过设计的，目的是让数据库能够在多个事务发生的环境中提供某些功能，旧版本的行可能会被当前事务之外的其他事务所需要。

`VACUUM`命令清理这些死行。单独运行`VACUUM`将死行所占的空间标记为可供数据库再次使用（假设任何使用该行的事务已经完成）。在大多数情况下，`VACUUM`并不会将空间返还给系统的磁盘，它只是将该空间标记为可以用于新数据。要真正缩小数据文件的大小，你可以运行`VACUUM FULL`，它会将表重新写成一个不包含死行空间的新版本，并删除旧版本。

尽管`VACUUM FULL`可以释放系统磁盘上的空间，但仍有几个注意事项需要记住。首先，`VACUUM FULL`的完成时间比`VACUUM`更长。其次，它在重写表时必须独占访问表，这意味着在操作期间没有人可以更新数据。常规的`VACUUM`命令可以在更新和其他操作进行时运行。最后，表中的并非所有死空间都是坏的。在许多情况下，拥有可用空间来存放新元组，而不是需要向操作系统请求更多磁盘空间，可以提高性能。

你可以根据需要运行`VACUUM`或`VACUUM FULL`，但 PostgreSQL 默认会运行一个*自动清理*后台进程，它会监控数据库并在需要时运行`VACUUM`。在本章的后面，我将展示如何监控自动清理并手动运行`VACUUM`命令。但首先，让我们来看一下表在更新后的增长情况，以及如何跟踪这种增长。

### 跟踪表大小

我们将创建一个小的测试表，并在向表中填充数据并执行更新时监控其增长。与本书中的所有资源一样，本练习的代码可以在[`nostarch.com/practical-sql-2nd-edition/`](https://nostarch.com/practical-sql-2nd-edition/)找到。

#### 创建一个表并检查其大小

清单 19-1 创建了一个`vacuum_test`表，该表只有一个列用来存储整数。运行代码后，我们将测量该表的大小。

```
CREATE TABLE vacuum_test (
    integer_column integer
);
```

清单 19-1：创建一个表以测试清理

在我们向表中填充测试数据之前，让我们检查一下它在磁盘上占用了多少空间，以便建立一个参考点。我们可以通过两种方式来做到这一点：通过 pgAdmin 界面检查表的属性，或者使用 PostgreSQL 管理函数运行查询。在 pgAdmin 中，单击一次表格以突出显示它，然后点击**统计**选项卡。表的大小是列表中的二十多个指标之一。

我将在这里重点介绍运行查询的技术，因为了解这些查询在 pgAdmin 不可用或使用其他图形用户界面（GUI）时非常有用。清单 19-2 展示了如何使用 PostgreSQL 函数检查`vacuum_test`表的大小。

```
SELECT 1pg_size_pretty(
           2pg_total_relation_size('vacuum_test')
       );
```

清单 19-2：确定`vacuum_test`的大小

最外层的函数`pg_size_pretty()` 1 将字节转换为更易于理解的格式，如千字节、兆字节或千兆字节。内部嵌套的`pg_total_relation_size()`函数 2 报告了一个表、它的索引以及任何离线压缩数据在磁盘上占用了多少字节。由于此时表为空，运行 pgAdmin 中的代码应该返回`0 字节`，如下所示：

```
 pg_size_pretty
----------------
 0 bytes
```

你也可以使用命令行获取相同的信息。启动`psql`，如第十八章所学。然后，在提示符下输入元命令`\dt+ vacuum_test`，它应该显示包括表大小在内的以下信息（为了节省空间，我省略了一列）：

```
 List of relations
 Schema |    Name     | Type  |  Owner   | Persistence |  Size
--------+-------------+-------+----------+-------------+---------
 public | vacuum_test | table | postgres | permanent   | 0 bytes
```

再次检查，当前的 `vacuum_test` 表的大小应显示为 `0 字节`。

#### 添加新数据后检查表的大小

让我们向表中添加一些数据，然后再次检查其大小。我们将使用第十二章中介绍的 `generate_series()` 函数，将 500,000 行数据填充到表的 `integer_column` 列中。运行 列表 19-3 中的代码来完成此操作。

```
INSERT INTO vacuum_test
SELECT * FROM generate_series(1,500000);
```

列表 19-3：向 `vacuum_test` 表插入 500,000 行数据

这个标准的 `INSERT INTO` 语句将 `generate_series()` 的结果（即从 1 到 500,000 的一系列值）作为行插入表中。查询完成后，再次运行 列表 19-2 中的查询来检查表的大小。你应该看到以下输出：

```
 pg_size_pretty
----------------
 17 MB
```

查询报告显示，`vacuum_test` 表现在有一列 500,000 个整数，使用了 17MB 的磁盘空间。

#### 更新后检查表的大小

现在，让我们更新数据，看看这如何影响表的大小。我们将使用 列表 19-4 中的代码，通过将 `integer_column` 的每个值加上 `1` 来更新 `vacuum_test` 表中的每一行，将现有值替换为更大的数字。

```
UPDATE vacuum_test
SET integer_column = integer_column + 1;
```

列表 19-4：更新 `vacuum_test` 表中的所有行

运行代码，然后再次测试表的大小。

```
 pg_size_pretty
----------------
 35 MB
```

表的大小从 17MB 翻倍到 35MB！增加的幅度看起来过大，因为 `UPDATE` 只是用相似大小的值替换了现有的数字。正如你可能猜到的那样，表大小增加的原因是，PostgreSQL 为每个更新的值创建了一个新行，而旧行依然保留在表中。即使你只看到了 500,000 行，表中实际有双倍数量的行。这种行为可能会给不监控磁盘空间的数据库所有者带来意外。

在查看 `VACUUM` 和 `VACUUM FULL` 如何影响表的磁盘大小之前，让我们回顾一下自动运行 `VACUUM` 的过程，以及如何检查与表清理相关的统计信息。

### 监控自动清理进程

PostgreSQL 的自动清理（autovacuum）进程监控数据库，并在检测到表中有大量死行时自动启动 `VACUUM`。尽管默认启用自动清理，你可以通过稍后在“更改服务器设置”一节中讲解的设置来开启或关闭它，并进行配置。由于自动清理在后台运行，你不会看到它正在工作的任何明显迹象，但你可以通过查询 PostgreSQL 收集的关于系统性能的数据来检查它的活动。

PostgreSQL 有自己的 *统计收集器*，用于跟踪数据库活动和使用情况。你可以通过查询系统提供的多个视图之一来查看统计信息。（查看 PostgreSQL 文档中“统计收集器”下的完整视图列表： [`www.postgresql.org/docs/current/monitoring-stats.html`](https://www.postgresql.org/docs/current/monitoring-stats.html)。）要检查自动清理的活动，我们查询 `pg_stat_all_tables` 视图，如 Listing 19-5 中所示。

```
SELECT 1relname,
       2last_vacuum,
       3last_autovacuum,
       4vacuum_count,
       5autovacuum_count
FROM pg_stat_all_tables
WHERE relname = 'vacuum_test';
```

Listing 19-5: 查看 `vacuum_test` 的自动清理统计信息

正如你在第十七章中学到的，视图提供了一个存储查询的结果。视图 `pg_stat_all_tables` 存储的查询返回一个名为 `relname` 的列，该列是表的名称，并且还有与索引扫描、插入和删除的行数以及其他数据相关的统计列。对于此查询，我们关心的是 `last_vacuum` 和 `last_autovacuum`，它们分别包含表格手动和自动清理的最后时间。我们还请求了 `vacuum_count` 和 `autovacuum_count`，它们显示手动和自动运行清理的次数。

默认情况下，自动清理每分钟检查一次表格。因此，如果自上次更新 `vacuum_test` 已经过去了一分钟，你应该会在运行 Listing 19-5 中的查询时看到清理活动的详细信息。以下是我的系统显示的内容（请注意，我已将时间中的秒数去除，以节省空间）：

```
 relname   | last_vacuum | last_autovacuum  | vacuum_count | autovacuum_count
-------------+-------------+------------------+--------------+------------------
 vacuum_test |             | 2021-09-02 14:46 |            0 |                1
```

该表显示了最后一次自动清理的日期和时间，`autovacuum_count` 列显示了一次出现的记录。这个结果表明，自动清理在该表上执行了 `VACUUM` 命令一次。然而，由于我们没有手动执行清理，`last_vacuum` 列为空，`vacuum_count` 为 `0`。

回想一下，`VACUUM` 会将死掉的行标记为可供数据库重新使用，但通常不会减少表格在磁盘上的大小。你可以通过重新运行 Listing 17-2 中的代码来确认这一点，它显示即使在自动清理之后，表格仍然保持在 35MB。

### 手动运行 VACUUM

要手动运行 `VACUUM`，可以使用 Listing 19-6 中的单行代码。

```
VACUUM vacuum_test;
```

Listing 19-6: 手动运行 `VACUUM`

该命令应该从服务器返回 `VACUUM` 的消息。现在，当你再次使用 Listing 17-5 中的查询提取统计信息时，你应该看到 `last_vacuum` 列反映了你刚刚运行的手动清理的日期和时间，并且 `vacuum_count` 列中的数字应该增加一个。

在这个例子中，我们对测试表执行了 `VACUUM`，但是你也可以通过省略表名来对整个数据库执行 `VACUUM`。此外，你可以添加 `VERBOSE` 关键字，以返回例如表格中找到的行数和删除的行数等信息。

### 使用 `VACUUM FULL` 减小表大小

接下来，我们将使用`FULL`选项运行`VACUUM`，该选项实际上会将被删除的元组所占的空间归还给磁盘。它通过创建一个新的表版本并丢弃死行来实现这一点。

要查看`VACUUM FULL`的工作原理，请运行清单 19-7 中的命令。

```
VACUUM FULL vacuum_test;
```

清单 19-7：使用`VACUUM FULL`来回收磁盘空间

命令执行后，再次测试表的大小。它应该已经恢复到 17MB，这是我们第一次插入数据时的大小。

永远不要让磁盘空间耗尽，因此，关注数据库文件的大小以及整体系统空间是一个值得建立的常规习惯。使用`VACUUM`来防止数据库文件比必要时更大，是一个不错的开始。

## 更改服务器设置

你可以通过编辑*postgresql.conf*中的值来更改 PostgreSQL 服务器的设置，这是控制服务器设置的几个配置文本文件之一。其他文件包括*pg_hba.conf*，它控制与服务器的连接，以及*pg_ident.conf*，数据库管理员可以使用它将网络上的用户名映射到 PostgreSQL 中的用户名。有关这些文件的详细信息，请参阅 PostgreSQL 文档；在这里，我们将只介绍*postgresql.conf*，因为它包含了你可能希望更改的设置。文件中的大多数值都设置为默认值，你可能永远不需要调整它们，但还是值得探索，以防你在微调系统时需要修改。让我们从基础开始。

### 定位和编辑 postgresql.conf

*postgresql.conf*的位置取决于你的操作系统和安装方式。你可以运行清单 19-8 中的命令来定位该文件。

```
SHOW config_file;
```

清单 19-8：显示*postgresql.conf*的位置

当我在 macOS 上运行该命令时，它显示文件的路径，如下所示：

```
/Users/anthony/Library/Application Support/Postgres/var-13/postgresql.conf
```

要编辑*postgresql.conf*，请在文件系统中导航到`SHOW config_file;`显示的目录，并使用文本编辑器打开该文件。不要使用像 Microsoft Word 这样的富文本编辑器，因为它可能会向文件中添加额外的格式。

当你打开文件时，前几行应该是这样的：

```
# -----------------------------
# PostgreSQL configuration file
# -----------------------------
#
# This file consists of lines of the form:
#
#   name = value
`--snip--`
```

*postgresql.conf*文件被组织成多个部分，指定文件位置、安全性、信息日志记录和其他进程的设置。许多行以井号符号（`#`）开始，这表示该行被注释掉，显示的设置是有效的默认值。

例如，在*postgresql.conf*文件的“自动清理参数”部分，默认情况下启用了自动清理（这是一个很好的标准做法）。行前的井号符号（`#`）表示该行被注释掉，默认值仍然生效：

```
#autovacuum = on               # Enable autovacuum subprocess?  'on'
```

要更改此项或其他默认设置，你需要去掉井号，调整设置值，并保存 *postgresql.conf*。某些更改（如内存分配的更改）需要重启服务器；这些更改会在 *postgresql.conf* 中注明。其他更改只需要重新加载设置文件。你可以通过在具有超级用户权限的账户下运行 `pg_reload_conf()` 函数，或者执行 `pg_ctl` 命令来重新加载设置文件，具体内容我们将在下一节介绍。

清单 19-9 显示了你可能想要更改的设置，摘自 *postgresql.conf* 中的“客户端连接默认值”部分。使用文本编辑器搜索文件以查找以下内容。

```
1 datestyle = 'iso, mdy'

2 timezone = 'America/New_York'

3 default_text_search_config = 'pg_catalog.english'
```

清单 19-9：示例 *postgresql.conf* 设置

你可以使用 `datestyle` 设置 1 来指定 PostgreSQL 在查询结果中显示日期的方式。此设置包含两个用逗号分隔的参数：输出格式和月份、日期和年份的排序。输出格式的默认值是 ISO 格式 `YYYY-MM-DD`，这是本书中始终使用的格式，我推荐使用它，因为它具有跨国便携性。不过，你也可以使用传统的 SQL 格式 `MM/DD/YYYY`，扩展的 Postgres 格式 `Sun Nov 12 22:30:00 2023 EST`，或德式格式 `DD.MM.YYYY`，在日期、月份和年份之间用点号分隔（如 `12.11.2023`）。要使用第二个参数指定格式，可以按你喜欢的顺序排列 `m`、`d` 和 `y`。

`timezone` 2 参数设置服务器的时区。清单 19-9 显示了值 `America/New_York`，它反映了我在安装 PostgreSQL 时机器上的时区。你的值应根据你所在的位置而有所不同。当将 PostgreSQL 设置为数据库应用程序的后端或在网络上使用时，管理员通常将此值设置为 `UTC`，并将其作为多个位置机器的标准。

`default_text_search_config` 3 值设置全文搜索操作使用的语言。这里，我的值设置为 `english`。根据需要，你可以将其设置为 `spanish`、`german`、`russian` 或你选择的其他语言。

这三个示例只是可调节设置的一小部分。除非你深入进行系统调优，否则通常不需要做太多其他调整。此外，在更改多个用户或应用程序使用的网络服务器上的设置时要谨慎；这些更改可能会带来意想不到的后果，因此最好先与同事沟通。

接下来，让我们看一下如何使用 `pg_ctl` 使更改生效。

### 使用 pg_ctl 重新加载设置

命令行工具 `pg_ctl` 允许你对 PostgreSQL 服务器执行操作，如启动、停止以及检查其状态。在这里，我们将使用该工具重新加载设置文件，以便我们所做的更改能够生效。运行此命令将一次性重新加载所有设置文件。

你需要像第十八章学习如何设置和使用`psql`时那样打开并配置命令行提示符。启动命令提示符后，使用以下命令之一重新加载，替换路径为 PostgreSQL 数据目录的路径：

1.  在 Windows 上，使用`pg_ctl reload -D "``C:\path\to\data\directory\``"`。

1.  在 macOS 或 Linux 上，使用`pg_ctl reload -D '``/path/to/data/directory/``'`。

要找到 PostgreSQL 数据目录的位置，请运行示例 19-10 中的查询。

```
SHOW data_directory;
```

示例 19-10：显示数据目录的位置

将路径放在`-D`参数后面，在 Windows 上使用双引号，在 macOS 或 Linux 上使用单引号。你需要在系统的命令提示符下运行此命令，而不是在`psql`应用程序内。输入命令并按回车键，它应该会返回`server signaled`的消息。设置文件将重新加载，并且更改应该生效。

如果你更改了需要重启服务器的设置，请将示例 19-10 中的`reload`替换为`restart`。

## 数据库的备份与恢复

你可能希望备份整个数据库，无论是为了保存数据还是为了将数据迁移到新服务器或升级后的服务器。PostgreSQL 提供了命令行工具，使备份和恢复操作变得简单。接下来的几个部分展示了如何将数据库或单个表的数据导出到文件，以及如何从导出文件恢复数据。

### 使用`pg_dump`导出数据库或表

PostgreSQL 命令行工具`pg_dump`创建一个包含你数据库所有数据的输出文件；包括重新创建表、视图、函数和其他数据库对象的 SQL 命令，以及将数据加载到表中的命令。你还可以使用`pg_dump`仅保存数据库中的特定表。默认情况下，`pg_dump`输出为文本文件；我将首先讨论一种自定义压缩格式，然后再讨论其他选项。

要将我们在练习中使用的`analysis`数据库导出到文件，请在系统的命令提示符下（不是在`psql`中）运行示例 19-11 中的命令。

```
pg_dump -d analysis -U `user_name` -Fc -v -f analysis_backup.dump
```

示例 19-11：使用`pg_dump`导出`analysis`数据库

在这里，我们以`pg_dump`开始命令，并使用与`psql`相似的连接参数。我们通过`-d`参数指定要导出的数据库，接着是`-U`参数和你的用户名。然后，我们使用`-Fc`参数指定我们希望以自定义 PostgreSQL 压缩格式生成此导出文件，并使用`-v`参数生成详细输出。接着，使用`-f`参数将`pg_dump`的输出导向名为*analysis_backup.dump*的文本文件。如果你希望将文件放在当前终端提示符所在目录之外的目录中，可以在文件名之前指定完整的目录路径。

执行命令时，取决于你的安装方式，你可能会看到一个密码提示。如果有提示，请填写密码。然后，根据数据库的大小，该命令可能需要几分钟才能完成。你将看到一系列关于命令正在读取和输出的对象的消息。当它完成时，它应该会返回一个新的命令提示符，并且你应该能在当前目录中看到一个名为*analysis_backup.dump*的文件。

若要将导出限制为匹配特定名称的一个或多个表，请使用`-t`参数，后跟单引号中的表名。例如，要仅备份`train_rides`表，可以使用以下命令：

```
pg_dump -t 'train_rides' -d analysis -U `user_name` -Fc -v -f train_backup.dump
```

现在让我们来看一下如何从导出文件中恢复数据，然后我们将探讨更多的`pg_dump`选项。

### 使用`pg_restore`恢复数据库导出

`pg_restore`工具从导出的数据库文件中恢复数据。你可能需要在将数据迁移到新服务器或升级到新的 PostgreSQL 主版本时恢复数据库。要恢复`analysis`数据库（假设你所在的服务器上没有`analysis`数据库），在命令提示符下，运行 Listing 19-12 中的命令。

```
pg_restore -C -v -d postgres -U `user_name` analysis_backup.dump
```

Listing 19-12: 使用`pg_restore`恢复`analysis`数据库

在`pg_restore`后，你添加`-C`参数，这告诉工具在服务器上创建`analysis`数据库。（它从导出文件中获取数据库名称。）然后，如前所述，`-v`参数提供详细输出，`-d`指定要连接的数据库名称，后跟`-U`参数和你的用户名。按回车键，恢复过程将开始。当它完成时，你应该能够通过`psql`或 pgAdmin 查看恢复的数据库。

### 探索更多的备份和恢复选项

你可以使用多个选项配置`pg_dump`，以包括或排除某些数据库对象，例如匹配名称模式的表，或指定输出格式。例如，当我们备份`analysis`数据库时，我们使用`-Fc`参数与`pg_dump`一起生成自定义的 PostgreSQL 压缩格式的备份。通过省略`-Fc`参数，工具将以纯文本格式输出，你可以使用文本编辑器查看备份内容。有关详细信息，请查看完整的`pg_dump`文档：[`www.postgresql.org/docs/current/app-pgdump.html`](https://www.postgresql.org/docs/current/app-pgdump.html)。有关相应的恢复选项，请查看`pg_restore`文档：[`www.postgresql.org/docs/current/app-pgrestore.html`](https://www.postgresql.org/docs/current/app-pgrestore.html)。

你还可以探索`pg_basebackup`命令，它可以备份在 PostgreSQL 服务器上运行的多个数据库。详情请见[`www.postgresql.org/docs/current/app-pgbasebackup.html`](https://www.postgresql.org/docs/current/app-pgbasebackup.html)。一个更为强大的备份解决方案是 pgBackRest（[`pgbackrest.org/`](https://pgbackrest.org/)），这是一款免费的开源应用，提供如云存储集成等选项，并且支持创建完整备份、增量备份或差异备份。

## 总结

在本章中，你学习了如何使用 PostgreSQL 中的`VACUUM`功能来跟踪和节省数据库空间。你还学会了如何更改系统设置，以及如何使用其他命令行工具备份和恢复数据库。你可能不需要每天都执行这些任务，但你在这里学到的维护技巧可以帮助提升数据库的性能。请注意，这不是该主题的全面概述；更多关于数据库维护的资源请参阅附录。

在本书的下一章也是最后一章，我将分享如何识别隐藏的趋势并利用数据讲述有效故事的指南。
