

# 第三十三章：B 数据库设置



![](img/opener.jpg)

本书的第 VI 部分概述了如何使用 PHP 与 MySQL 和 SQLite 数据库进行交互。本附录涵盖了如何确保这些数据库管理系统在您的本地计算机上已正确设置。

## MySQL

MySQL 有多个版本可供选择。对于本书的目的，免费版的 MySQL Community Server 足够使用。我们将讨论如何为您选择的操作系统安装 MySQL。

### macOS 和 Windows

要在 macOS 或 Windows 上安装 MySQL Community Server，请访问 *[`dev.mysql.com/downloads/mysql/`](https://dev.mysql.com/downloads/mysql/)*。该网站应该能够自动检测您的操作系统，因此您只需下载适合您系统的最新版本安装程序即可。对于 macOS，我推荐使用其中一个 DMG 压缩文件：适用于 M 系列机器的 ARM 安装程序，或适用于基于 Intel 的机器的 x86 安装程序。对于 Windows，我建议使用 Microsoft 软件安装程序 (MSI)。

下载适合您系统的安装程序后，运行它并接受默认选项。您需要特别注意的部分是当系统要求您为 MySQL 服务器的 root 用户输入密码时。选择一个您能够记住的密码，因为在您与数据库服务器通信的 PHP 脚本中，您需要提供此密码。

完成安装过程后，MySQL 服务器应该可以与您的 PHP 应用程序一起使用。默认安装会将服务器配置为每次重新启动系统时自动启动并在后台运行，因此在使用 MySQL 之前，您不需要手动启动服务器。

### Linux

如果您是 Linux 用户，您需要安装 PDO 和 MySQL 服务器扩展包，以便使用 PDO 库让 PHP 与 MySQL 数据库进行通信。使用以下命令：

```
$ **sudo apt-get install php-mysql**
$ **sudo apt-get install mysql-server**
```

数据库服务器在安装完成后应该已经启动，您可以通过以下命令检查其状态：

```
$ **sudo ss -tap | grep mysql**
LISTEN 0      70         127.0.0.1:33060        0.0.0.0:*
users:(("mysqld",pid=21486,fd=21))
LISTEN 0      151        127.0.0.1:mysql        0.0.0.0:*
users:(("mysqld",pid=21486,fd=23))
```

这表示服务器正在运行，并且运行在端口 33060 上。如果您需要重新启动 MySQL 服务器，可以使用以下命令：

```
$ **sudo service mysql restart**
```

如果您愿意，您可以为 root MySQL 用户设置密码，方法如下（将密码替换为您喜欢的任何内容）：

```
$ **sudo mysql**
mysql> **ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '**`**password**`**';**
mysql> **exit**
Bye
```

现在，您可以将 MySQL 数据库用于您的 PHP 项目。

## SQLite

在 macOS 上通过 Homebrew 安装 PHP 时，SQLite 应该默认启用。在 Windows 上，只要在您的 INI 文件中启用了 pdo_sqlite 扩展，SQLite 就会可用。我们在附录 A 中讨论了如何验证这一点。

在 Linux 上，使用以下命令启用 PHP 与 SQLite 数据库进行通信：

```
$ **sudo apt install php-sqlite3**
```

截至本文写作时，SQLite 的最新稳定版本是版本 3。

## 确认 MySQL 和 SQLite 扩展

你可以随时通过创建一个调用 phpinfo()函数的*index.php*脚本来检查当前激活的 PHP 数据库扩展。正如在第一章中讨论的那样，这个函数会打印出关于你 PHP 安装的详细报告。列表 B-1 展示了你需要的*index.php*文件。

```
<?php
phpinfo();
```

列表 B-1：用于查看 PHP 设置的 index.php 脚本

通过在命令行输入 php -S localhost:8000 来提供此脚本，然后在浏览器中打开*localhost:8000*。在生成的页面中搜索**PDO**以查看 PDO 数据库扩展的列表。如果一切正常，你应该能看到 MySQL 和 SQLite 都已启用。
