

# 第八章：8 本地信息收集



![](img/opener.jpg)

在前两章中，我们已经在多个主机上获得了初步立足点。在这一章中，我们将进行本地侦察，识别感兴趣的资产，在征服网络中其他主机的道路上，不留下任何遗漏。

一旦成功控制了主机，知道在哪里找到敏感信息是一项关键技能。我们将专注于你可以收集的关键信息类别：身份（如用户和组）、文件（包括日志和配置文件）、网络信息、自动化工作流、已安装的软件和固件、正在运行的进程以及安全机制。在第九章中，当我们讨论特权提升技术时，还会涉及其他信息，如凭证。

在现实生活场景中，后渗透阶段也是你被防守方抓住的几率增加的阶段，因为你收集的信息可能会留下痕迹。出于这个原因，我们会尽可能默认使用本地的 Linux 工具和文件来收集信息，尝试做到*利用现有资源*：利用主机上已有的工具，而避免使用外部工具，这些工具可能会触发警报。

尝试在你目前已控制的所有主机上运行本章介绍的 shell 命令，以及在继续阅读过程中你所控制的任何新机器。你甚至可以根据这些命令编写脚本，在所有机器上轻松执行相同的命令。

## 文件系统层次结构标准

感兴趣的数据可能分布在 Linux 文件系统的多个区域。为了高效探索你获得 shell 访问权限的系统，参考*文件系统层次结构标准（FHS）*，它描述了 Linux 系统中文件夹的结构及其位置。这一层次结构标准使得用户和程序能够更容易地搜索感兴趣的文件，如日志文件或配置文件。

Linux 文件系统的层次结构从根目录（*/*）开始，这是进入文件系统目录树结构的起点。表 8-1 展示了根目录下的主要子目录及其主要用途。

表 8-1：文件系统层次结构标准目录布局

| 目录 | 描述 |
| --- | --- |
| / | 主目录，也叫根目录。 |
| /var | 用于非静态（可变）文件的目录。通常包含/var/log 目录下的应用日志文件，或包含处理过的任务，如定时任务和打印作业，存放在/var/spool 目录下。它也可能包含/var/cache 下的缓存文件，以及/var/run 下的系统相关运行时数据。 |
| /etc | 存放配置文件的目录。安装在系统上的应用程序将其专用的配置文件保存在此目录中（通常以 *.conf 后缀）。该目录还包含诸如 /etc/passwd、/etc/group 和 /etc/shadow 等文件，分别存储用户账户、组信息和密码哈希。 |
| /bin | 存放二进制工具的目录。通常用于存放与系统任务相关的二进制文件，如导航命令（cd）、文件复制（cp）、目录创建（mkdir）或文件创建（touch）。 |
| /sbin | 存放系统二进制文件的目录，例如用于系统调试、磁盘操作和服务管理的工具，主要供系统管理员使用。 |
| /dev | 表示并提供访问设备文件（如磁盘分区、U 盘和外部硬盘驱动器）的目录。 |
| /boot | 存放引导加载程序、内核文件和初始随机存取内存（RAM）磁盘（initrd）的目录。 |
| /home | 存放本地系统用户账户的家目录的目录。活动的系统用户账户通常会有一个子目录作为其分配的家目录。 |
| /root | 存放 root 用户家目录的目录。 |
| /tmp | 存放临时文件和目录的目录。/var/tmp 是另一个常用于临时文件的临时目录。 |
| /proc | 存放进程和内核数据的虚拟文件系统。在系统启动时自动创建。 |
| /usr | 存放用户二进制文件、手册页、内核源代码、头文件等的目录（过去还包括游戏）。 |
| /run | 存放运行时数据的目录。描述自上次启动以来系统的状态。 |
| /opt | 存放软件应用程序的目录。通常用于存放与第三方软件安装相关的数据。 |
| /mnt | 用于挂载网络共享或其他网络设备的目录，主要用于将设备挂载到本地文件系统，可以是临时的也可以是永久的。 |
| /media | 存放可移动设备的目录，例如 CD 驱动器。作为挂载点使用。 |
| /lib, /lib32, /lib64 | 存放启动系统和运行命令所需共享库的目录。 |
| /srv | 存放常见网络服务数据的目录，例如 Web 服务器和文件服务器的数据。 |

生产系统可能有成千上万个文件分布在各个位置，因此了解需要搜索的敏感数据以及搜索位置非常重要。

虽然 FHS 旨在标准化文件系统的布局，但系统可以偏离标准。此外，系统管理员可以将应用程序文件存储在任何位置。例如，系统管理员完全可以将整个 Web 服务器内容服务于像*/mywebsite* 这样的目录，并将日志写入像*/data/logs* 这样的目录。

## Shell 环境

从信息收集的角度来看，shell 环境非常重要，因为它可以揭示系统查找可执行文件的路径。自定义应用程序可能会向 PATH 环境变量中添加新的目录路径，以便应用程序能够从非标准位置运行自定义库和可执行文件。你也可能会在这些自定义配置中发现凭证和其他机密信息。

### 环境变量

当入侵主机时，使用 env 或 printenv 命令转储其环境变量通常是很有用的。管理员往往将凭证存储在环境变量中，以避免将凭证写入磁盘文件。交付系统可以通过这些环境变量将凭证注入到应用程序的运行时，应用程序然后读取这些凭证。此外，你还可能在环境变量中找到其他重要信息，如相邻服务器的地址和运行时配置。

### bash 配置文件中的敏感信息

在第二章中，我们使用了*~/.bashrc*文件和 bash 别名来设置命令的快捷方式。系统管理员可以轻松地在像*~/.bashrc*这样的 shell 脚本中包含凭证，以避免在命令行上手动输入凭证，因此总是要仔细检查是否做了任何自定义设置；你可能会找到凭证或用于管理目的的命令。以下是一些常见的配置文件，可以检查一下：*/etc/profile*、*/etc/bashrc*、*~/.bashrc*、*~/.profile*、*~/.bash_profile*、*~/.env*、*~/.bash_login*和*~/.bash_logout*。

除了 bash 外，系统上也可能存在其他 shell，如 Z Shell。在这种情况下，你可能需要查看像*/etc/zprofile*、*/etc/zshrc*、*~/.zprofile*和*~/.zshrc*这样的文件。

使用 man 命令来了解各种 shell 的环境和配置文件。例如，运行 man bash 可以查看 bash shell，man zsh 可以查看 Z Shell，man csh 可以查看 C Shell。

## 用户和组

你应该收集系统中各种用户和组的信息。系统可以为人类操作员配置用户账户，但你也可能会遇到除了 Linux 机器的默认账户外没有其他账户的系统。尤其是在容器化环境中，主机可能每天会频繁创建和销毁。短生命周期的服务器通常不会使用本地系统账户进行管理；相反，编排和配置工具会自动化整个部署、升级、降级、扩展等过程。

### 本地账户

Linux 系统有多个默认的用户和组。你可以在*/etc/passwd*中找到用户账户，在*/etc/group*中找到组信息，即使是权限较低的用户也应该能够读取这些文件。这些文件不包含敏感数据，但可以帮助你找到其他目录和文件，因为 Linux 系统中的一切都有用户和组的所有权。

> 注意

*黑客通常会攻击* /etc/passwd *和* /etc/group *文件，因此，安全防御者需要通过适当的监控来观察是否有任何读取或写入这些文件的行为。*

让我们查看被攻陷主机上的 */etc/passwd* 文件。在 *p-web-01*（172.16.10.10）、*p-web-02*（172.16.10.12）和 *p-jumpbox-01*（172.16.10.13）上运行清单 8-1 中的命令，查看用户列表及其属性。

```
$ **cat /etc/passwd**

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
`--snip--`
messagebus:x:100:101::/nonexistent:/usr/sbin/nologin
systemd-resolve:x:996:996:systemd Resolver:/:/usr/sbin/nologin
jmartinez:x:1001:1001::/home/jmartinez:/bin/bash
`--snip--` 
```

清单 8-1：查看系统上的用户

如你所见，我们得到一个由冒号（:）分隔的值列表。每一行都是一个唯一的用户账户，每个字段表示该账户的特定信息。我们特别关注输出中的第一行，这一行表示有一个 *root* 用户账户。表格 8-2 将这一行拆分成其组成字段。

表格 8-2：/etc/passwd 文件的字段

| 账户 | 密码 | 用户 ID | 组 ID | 注释 | 主目录 | 默认 shell |
| --- | --- | --- | --- | --- | --- | --- |
| root | x | 0 | 0 | root | /root | /bin/bash |

第一个字段是账户的用户名，第二个字段中的 *x* 表示密码。你可以在名为 */etc/shadow* 的单独文件中找到相应的密码哈希，我们将在后面的章节中讨论凭证访问时提及该文件。第三和第四个字段分别表示用户的用户 ID（UID）和组 ID（GID）。第五个字段是注释字段，可以包含有关用户的详细信息（例如全名、位置和员工 ID）。第六个字段表示用户的主目录（在本例中为 */root*），第七个字段表示其默认的 shell 环境（在本例中为 */bin/bash*）。

使用 bash，我们可以解析 */etc/passwd* 的输出，提取特定的字段。例如，要提取每个用户的用户名（第一个字段）、主目录（第六个字段）和默认 shell（第七个字段），请运行清单 8-2 中的命令。

```
$ **awk -F':' '{print $1, $6, $7}' /etc/passwd | sed 's/ /,/g'**

root,/root,/bin/bash
daemon,/usr/sbin,/usr/sbin/nologin
bin,/bin,/usr/sbin/nologin
sys,/dev,/usr/sbin/nologin
sync,/bin,/bin/sync
`--snip--` 
```

清单 8-2：从 /etc/passwd 中提取关键信息

由于字段是由冒号分隔的，我们可以轻松使用 awk 和 sed 来提取感兴趣的字段。

### 本地组

接下来，运行清单 8-3 中的命令，查看本地组列表。

```
$ **cat /etc/group**

root:x:0:
daemon:x:1:
bin:x:2:
sys:x:3:
**adm:x:4:ubuntu**
tty:x:5:
disk:x:6:
lp:x:7:
`--snip--` 
```

清单 8-3：查看系统上的组

*/etc/group* 文件的格式如下：第一个字段是表示组名的唯一值，第二个字段表示密码，第三个字段是 GID，最后一个字段是每个组成员的列表，成员之间由逗号分隔。正如你在输出的加粗部分所看到的，*ubuntu* 用户账户是 *adm* 组的一部分，*adm* 组用于系统管理任务，如查看日志。

### 主文件夹访问

默认情况下，只有用户或超级用户（如 *root* 用户）可以访问该用户的主目录。在清单 8-4 中运行命令，列出所有用户的主目录及其权限。

```
$ **ls -l /home/**

total 20
drwxr-x--- 2 arodriguez arodriguez 4096 May 19 02:28 arodriguez
drwxr-x--- 2 dbrown     dbrown     4096 May 19 02:28 dbrown
drwxr-x--- 2 jmartinez  jmartinez  4096 May 19 02:28 jmartinez
drwxr-x--- 2 ogarcia    ogarcia    4096 May 19 02:28 ogarcia
drwxr-x--- 2 ubuntu     ubuntu     4096 Apr 20 13:44 ubuntu 
```

列表 8-4：查看 home 目录和权限

正如你所看到的，每个 home 目录都归其所属的用户所有。我们将在第九章中更详细地讨论目录权限。

让我们写一个小的 bash 脚本来检查是否能够访问用户的 home 目录。这是非常有用的，因为权限可能会因错误而被搞乱，例如在权限被递归修改时，或者在涉及大量用户账户的大型系统中。

> 注意

*本章节的脚本可以在* [`github.com/dolevf/Black-Hat-Bash/blob/master/ch08`](https://github.com/dolevf/Black-Hat-Bash/blob/master/ch08) 获取。

列表 8-5 中的脚本将执行以下步骤：检查运行用户是否可以读取 */etc/passwd*，如果可以，则读取其内容；提取每个用户账户的默认 home 目录路径；检查当前用户是否可以读取每个 home 目录；并打印结果。

home_dir _access_check.sh

```
#!/bin/bash

if [[! -r "/etc/passwd"]]; then
  echo "/etc/passwd must exist and be readable to be able to continue."
  exit 1
fi

❶ while read -r line; do
❷ account=$(echo "${line}" | awk -F':' '{print $1}')
❸ home_dir=$(echo "${line}" | awk -F':' '{print $6}')

  # Target only home directories under /home.
❹ if echo "${home_dir}" | grep -q "^/home"; then
  ❺ if [[-r "${home_dir}"]]; then
      echo "Home directory ${home_dir} of ${account} is accessible!"
    else
      echo "Home directory ${home_dir} of ${account} is NOT accessible!"
    fi
  fi
done < <(cat "/etc/passwd") 
```

列表 8-5：尝试访问用户的 home 目录

在 while 循环中，我们逐行读取 */etc/passwd* 文件 ❶。在 ❷ 和 ❸ 处，我们分别将账户和 home_dir 变量赋值为每行的第一个和第六个字段。接着，我们使用插入符号 (^) 字符和 grep -q（安静模式）选项检查 home 目录是否以 /home 开头，确保命令的输出不会打印到标准输出流中。在 ❺ 处，如果我们之前的检查成功，我们会检查 home 目录是否可读，并将结果打印到屏幕上。

### 有效的 Shell

我们提到过 */etc/passwd* 的第七个字段是用户的默认 shell。然而，系统管理员可以为用户分配一个无效的 shell 作为安全加固措施。对于黑客来说，具有真实 shell（例如 */bin/bash*）的账户应该表明两种可能性之一：账户属于一个需要登录的真实用户或服务，或者账户存在配置错误。

当系统管理员使用 useradd 或 adduser 命令向 Linux 机器添加账户时，默认的 shell 由 */etc/default/useradd* 文件中的 SHELL 设置或 */etc/adduser.conf* 中的 DSHELL 设置决定，正如你在这里看到的：

```
$ **grep -e "#DSHELL" /etc/adduser.conf**
#DSHELL=/bin/bash

$ **grep -e "SHELL=" /etc/default/useradd**
SHELL=/bin/sh 
```

通过一些高级 bash 和 awk，我们可以筛选出包含有效 shell（例如 */bin/bash* 或 */bin/sh*）的行，然后将后续工作集中在这些账户上（列表 8-6）。

```
$ **awk -F':' '{if ($7=="/bin/sh" || $7=="/bin/bash") {print $1,$7}}' /etc/passwd**

root /bin/bash
ubuntu /bin/bash
jmartinez /bin/bash
dbrown /bin/bash
ogarcia /bin/bash
arodriguez /bin/bash 
```

列表 8-6：使用高级 awk 语法查找具有活动 shell 的账户

我们故意让这个命令比必要的稍微复杂一些，以便让你看到 awk 在解析方面的强大功能。在列表 8-6 中，awk 使用内建的 if 条件和 OR 操作符 (||) 来检查文件的第七个字段是否等于 */bin/sh* 或 */bin/bash*。如果表达式为真，它会打印第一个和第七个字段。

就像在 bash 中做任何事一样，你也可以通过一个更简单的命令来实现相同的目标（列表 8-7）。

```
$ **grep -e "/bin/bash" -e "/bin/sh" /etc/passwd**
```

列表 8-7：使用 grep 查找具有活动 shell 的账户

然而，这个更简单的 grep 命令更容易出错，因为它会打印包含这两个字符串的任何字段（而不仅仅是第七个字段，其中定义了默认的 shell）。

## 进程

枚举运行中的进程是成功侦察的重要步骤。进程帮助我们识别系统正在运行的所有代码，从而使我们能够集中精力针对特定应用程序。进程还很重要，因为它们帮助我们了解主机的防御系统。

### 查看进程文件

每个 Linux 主机上的进程都有一个专门的目录，在 */proc* 下，目录名称与其进程标识符（PID）相同，PID 是一个数值。让我们运行一个简单的 ls 命令（使用 -1 选项每行列出一个文件），然后使用带有特殊正则表达式的 grep 来列出该目录中所有名称为数字的文件（列表 8-8）。

```
$ **ls -1 /proc/ | grep -E '^[0-9]+$'**

1
33
34
7 
```

列表 8-8：在 /proc 目录中筛选 PID

因为新进程经常会生成并随之终止，所以你可能会看到与此输出中的 PID 不同的 PID 数字（除了 1，通常称为 *init 进程*，它应该始终存在）。让我们探索 init 进程的文件夹中可用的信息：

```
$ **ls -1 /proc/1/**

arch_status
attr
autogroup
auxv
cgroup
clear_refs
cmdline
comm
coredump_filter
cpu_resctrl_groups
cpuset
cwd
environ
exe
fd
`--snip--` 
```

该文件夹包含许多文件，其中一些对渗透测试人员更为有趣。例如，以下文件包含有用的信息：

***/proc/<pid>/cmdline*** 包含启动进程时使用的完整命令。

***/proc/<pid>/cwd*** 指向进程的工作目录。

***/proc/<pid>/environ*** 包含进程启动时的环境变量。

***/proc/<pid>/exe*** 指向启动该进程的二进制文件。

***/proc/<pid>/task*** 包含进程启动的每个线程的子目录。

***/proc/<pid>/status*** 包含关于进程的信息，例如其状态、虚拟内存大小、线程数量、线程 ID 和进程的 *umask*（一个四位数值，用于确定新创建文件的权限）。

***/proc/<pid>/fd*** 包含正在使用的 *文件描述符*。文件描述符是进程用于描述打开文件的非负整数（无符号整数）。

让我们探索这些文件，看看它们能告诉我们关于系统上 PID 1 的信息。在 *p-web-01*（172.16.10.10）上，运行以下命令：

```
$ **cat /proc/1/cmdline**

python3-mflaskrun--host=0.0.0.0--port=8081 
```

如你所见，启动此进程的是一个 python3 命令。由于其元素由空字节分隔，输出有些难以阅读。我们可以使用以下命令将空字节替换为空格，使其更易读：

```
$ **cat /proc/1/cmdline | tr '\000' ' '**

python3 -m flask run --host=0.0.0.0 --port=8081 
```

接下来，查看符号链接 */proc/1/cwd*，通过运行以下 ls 命令来确定进程 1 的工作目录：

```
$ **ls -ld /proc/1/cwd**

lrwxrwxrwx 1 root 0 May  4 01:26 /proc/1/cwd -> /app 
```

输出中的第一个字符是 l，表示符号链接。你还可以看到从*/proc/1/cwd*到*/app*有一个箭头（->），表明*cwd*符号链接指向*/app*目录。

我们鼓励你发现*/proc*目录下的任何其他文件及其用途。你可以在 proc 手册页中找到对这些文件的详细解释（通过运行 man proc）。

### 运行 ps

类似 ps 这样的工具使我们能够探索进程，而无需手动浏览/*proc*目录。运行以下命令以查看进程列表：

```
$ **ps aux**

USER  PID %CPU %MEM    VSZ   RSS TTY    STAT START   TIME COMMAND
root    1  0.0  0.7  36884 30204 ?      Ss   01:12   0:00 python3 -m flask run --host=0.0.0...
root    7  0.0  0.0   4508  3900 pts/0  Ss   01:12   0:00 /bin/bash
root   92  0.0  0.0   8204  3888 pts/0  R+   02:05   0:00 ps aux 
```

输出是轻量级的，因为实验室运行在容器上，而容器旨在尽可能使用最少的资源。在运行非容器化服务器的生产系统上，你可能会看到更多的进程。你可以在你的 Kali 主机上运行相同的命令，查看输出的差异。

ps 命令使用*/proc*虚拟文件系统以更易于理解的方式显示进程信息。让我们利用它的一些内置过滤功能，从输出中提取关键信息，如运行用户、PID 和执行的命令：

```
$ **ps x -o user -o pid -o cmd**

USER         PID CMD
root           1 python3 -m flask run --host=0.0.0.0 --port=8081
root           7 /bin/bash
root         137 ps x -o user -o pid -o cmd 
```

对我们目前已攻破的所有机器运行相同的命令，并记录你的结果。

### 检查 Root 进程

进程的所有权也是一个需要考虑的重要因素。以 root 身份运行的进程如果编写不安全，可能会导致权限提升漏洞。例如，当我们攻破*p-web-01*网站服务器（172.16.10.10）时，由于*root*用户初始化并启动了应用程序，我们进入了 shell 并以*root*用户身份登录。

以超级用户身份运行应用程序通常被认为是不好的做法，但这使得我们作为渗透测试者的工作变得更加轻松。如果应用程序是以自定义应用用户启动的，我们就得寻找权限提升的机会。正如你可能记得的那样，当我们攻破*p-web-02*（172.16.10.12）机器时，我们作为*www-data*用户登录，而不是 root 用户。

作为另一个使用*root*用户运行应用程序时不推荐的例子，假设有一个 bash 脚本每 10 分钟由 root 用户以后台任务的方式执行一个名为*/tmp/update.sh*的文件，并且假设该文件也可以被其他系统用户写入。在这个例子中，某人可以在文件中写入一条指令，授予自己额外的权限，由于该进程是以 root 身份运行的，执行*update.sh*文件时也会以*root*用户的身份执行。

## 操作系统

Linux 操作系统有如此多的变种，以至于像*[`distrowatch.com`](https://distrowatch.com)*这样的网站专门用于跟踪它们。你如何确定你刚刚攻破的主机上运行的到底是哪种操作系统？

操作系统可能会将其相关信息存放在不同的位置，但大多数情况下，你会在*/etc*目录下找到。检查以下位置：*/etc/os-release*、*/etc/issue*、*/usr/lib/os-release*、*/proc/version*、*/etc/*-release*和*/etc/*-version*。例如，在基于 Ubuntu 的*p-web-01*机器（172.16.10.10）上，你应该能在*/etc/os-release*中找到操作系统的信息。

除了文件，一些工具也能帮助你识别操作系统。试试运行`uname -o`或`uname -a`、`lsb_release`、`hostnamectl`和`hostname`。尽管像`hostname`和`hostnamectl`这样的命令并不是为了显示操作系统信息而设计的，但如果系统管理员将机器的主机名设置为包含操作类型（如*ubuntu-prod-01*），它们也可能揭示操作系统信息。同样，内置的环境变量$HOSTNAME 也保存着主机名的值。

练习 12：编写一个 Linux 操作系统检测脚本

尝试编写一个脚本，能够识别任何基于 Linux 的操作系统的操作系统类型（例如 Ubuntu、Debian 或其他）。为此，脚本应查找感兴趣的特定文件并从中提取信息。此外，因为任何人都应该能够在任何 Linux 系统上运行该脚本并期望它能够优雅地失败，你需要考虑如何处理错误。

以下是脚本应采取的步骤：

1.  脚本应使用一个或多个可用的方法，通过命令或文件收集我们之前强调的与操作系统相关的信息。你也可以进行自己的研究，以实现其他本地操作系统发现方法。

2.  如果你没有找到操作系统检测方法，脚本需要处理这种情况，并向用户指示。

3.  脚本应在运行结果正确的状态码下退出。

本书 GitHub 仓库中的脚本*os_detect.sh*是一个操作系统检测脚本的示例。

## 登录会话和用户活动

当用户登录系统或打开一个新的终端会话时，系统会记录下这些信息。无论用户是本地登录（例如在笔记本电脑上）还是通过如 SSH 或 Telnet 等协议远程登录，都会发生这种情况。

这些信息非常有价值，因为它可以告诉你有关以前连接的情况，包括用于连接的源 IP 地址。例如，如果系统管理员使用专用的管理服务器连接到其他服务器，收集登录会话将揭示管理服务器的 IP 地址。

### 收集用户会话

要查看系统上的当前用户，可以使用`w`或`who`命令：

```
$ **w**
$ **who** 
```

这些命令显示的信息包括用户的用户名、登录时间以及当前进程的命令。命令从*/var/run/utmp*文件中读取这些信息。

last 命令显示来自 */var/log/wtmp* 文件的历史登录记录，该文件包含当前和过去的用户会话：

```
$ **last**
```

在通过 SSH 使用备份用户登录后，尝试在 *p-jumpbox-01* 机器（172.16.10.13）上运行这些命令。

另一个有用的命令是 lastb（last bad）。该命令显示一个失败登录尝试的列表，数据来源于 */var/log/btmp* 文件（如果该文件存在于文件系统中）。

像 */var/run/utmp* 和 */var/log/wtmp* 这样的文件是二进制文件。如果您尝试使用 cat 命令读取它们，输出将会是乱码。某些系统可能提供 utmpdump 命令，它可以将这些文件作为参数，并以正确的格式将内容输出到屏幕上。

### 调查已执行的命令

当用户开始在 shell 中执行命令时，系统会捕获这些信息并将其写入 *历史文件*，这些通常是隐藏文件（以点开头的文件），存储在用户的主目录中。例如，*root* 用户的历史文件位于 */root/.bash_history*。对于普通用户，历史文件通常保存在 */home/<user>/.bash_history*。不同的 shell 可能会以不同的方式命名历史文件。例如，Z Shell 的历史文件命名为 *.zsh_history*。

历史文件很有趣，因为它们本质上是用户在命令行上的操作摘要。如果某人使用凭证运行 curl 命令来认证远程网站，命令及其凭证将被记录在历史文件中。要查看当前用户的历史文件，可以运行以下命令：

```
$ **history**
```

使用 bash 一行命令和 find 可以帮助我们搜索带有 *_history* 后缀的隐藏文件（列表 8-9）。

```
$ **find / -name ".*_history" -type f**
```

列表 8-9：搜索 shell 命令历史文件

该命令从根目录（*/*）开始搜索，并对文件进行区分大小写的搜索（-type f），查找文件名以字符串 *_history* 结尾的文件。

## 网络

网络信息是收集系统数据中最重要的部分之一。在渗透测试中，您可能只知道一个网络（例如，如果您在现场参与测试时，您可能只知道您物理连接的网络），但这并不意味着这是唯一可用的网络。如果您恰好入侵了一个 *multi-homed* 主机，即一台具有多个网络接口并连接到不同网络的机器，您可能会发现新网络。

### 网络接口和路由

在被入侵的主机上，一种获取所有网络接口的简单方法是查看 */sys/class/net* 目录下的文件。继续尝试在被入侵的机器上列出文件。以下示例来自 *p-web-01* 主机（172.16.10.10）：

```
$ **ls -l /sys/class/net/**

total 0
lrwxrwxrwx 1 root root 0 May 10 03:13 eth0 -> ../../devices/virtual/net/eth0
lrwxrwxrwx 1 root root 0 May 10 03:13 lo -> ../../devices/virtual/net/lo 
```

每个文件都是一个符号链接，包含一个网络接口的名称，每个链接指向 */sys/devices/virtual/net/* 目录下的一个子目录：

```
$ **ls -l /sys/devices/virtual/net/**
total 0
drwxr-xr-x 5 root root 0 May 10 03:13 eth0
drwxr-xr-x 5 root root 0 May 10 03:13 lo 
```

你还可以使用网络接口分析来判断一个网络设备是物理的还是虚拟的。值得注意的是，管理员可以更改网络接口的名称，因此这些并不是可靠的指标。不过，物理网络设备在你列出 */sys/devices/virtual/net* 下的文件时应该会有不同的表现。你可以在 Kali 机器上运行之前的命令，应该会看到类似以下的输出：

```
lrwxrwxrwx 1 root root 0 Sep 25 16:15 br_corporate -> ../../devices/virtual/net/br_corporate
lrwxrwxrwx 1 root root 0 Sep 25 16:15 br_public -> ../../devices/virtual/net/br_public
lrwxrwxrwx 1 root root 0 Sep 19 21:41 docker0 -> ../../devices/virtual/net/docker0
lrwxrwxrwx 1 root root 0 Sep 19 21:41 eth0 -> ../../devices/**pci0000:00/0000:00:03.0**/net/eth0
lrwxrwxrwx 1 root root 0 Sep 19 21:41 lo -> ../../devices/virtual/net/lo 
```

如你所见，除了 eth0 之外，所有设备都是虚拟的，eth0 有一个外设组件互联总线标识符 pci0000:00/0000:00:03.0。在你的机器上，根据你使用的网卡，这可能会有所不同。

> 注意

*要确定目标是物理服务器还是虚拟服务器，需要使用多种启发式方法。网络收集可能会产生假阳性。*

另一种不使用特殊网络工具打印所有网络接口的方法是检查 */proc/net/route* 文件，该文件包含有关网络路由的信息。在强化的主机或轻量级 Linux 容器中手动检查此文件非常有用，因为你可能无法访问常见的网络工具，如 ifconfig、ip、netstat 或 ss（套接字统计）：

```
$ **cat /proc/net/route**

Iface Destination Gateway Flags RefCnt Use Metric Mask MTU Window IRTT
eth0 00000000 010A10AC 0003 0 0 0 00000000 0 0 0
eth0 000A10AC 00000000 0001 0 0 0 00FFFFFF 0 0 0 
```

文件的第一行是列标题行，每一行之后的内容对应一个网络路由、其网络接口以及以十六进制格式表示的其他路由相关信息。例如，在第一行中，Gateway 下的值 010A10AC 表示网络接口的网关 IP 地址。如果你将每个字节转换为十进制值，你应该得到如下结果：

011

0A10

1016

AC172

这是 172.16.10.1，eth0 接口的网关 IP 地址，以小端格式表示。你可以使用 *[`ascii.cl/conversion.htm`](https://ascii.cl/conversion.htm)* 来将值从十六进制转换为十进制，或者使用 bash 进行转换：

```
$ **echo $((16#AC))**
172 
```

使用算术运算符 $(()) 和字符序列 16#，表示十六进制（或 *base16*），你可以将任何十六进制值转换为十进制数。

*/proc/net/route* 文件没有给我们主机上网络接口的 IP 地址。然而，我们可以通过查看 */proc/net/fib_trie* 文件来获取这些信息。这个文件包含类似这样的数据：

```
Main:
  +-- 0.0.0.0/0 3 0 5
`--snip--`
           |-- 127.0.0.1
              /32 host LOCAL
        |-- 127.255.255.255
           /32 link BROADCAST
     +-- 172.16.10.0/24 2 0 2
        +-- 172.16.10.0/28 2 0 2
           |-- 172.16.10.0
              /24 link UNICAST
           |-- 172.16.10.10
`--snip--`
Local:
  +-- 0.0.0.0/0 3 0 5
     |-- 0.0.0.0
        /0 universe UNICAST
     +-- 127.0.0.0/8 2 0 2
        +-- 127.0.0.0/31 1 0 0
 |-- 127.0.0.0
              /8 host LOCAL
           |-- 127.0.0.1
`--snip--` 
```

要解析此输出并仅获取网络接口的 IP 地址，我们可以使用 列表 8-10 中的 bash 脚本。

```
$ **awk '/32 host/ {print f} {f=$2}' /proc/net/fib_trie | sort | uniq**

127.0.0.1
172.16.10.10 
```

列表 8-10：提取网络接口的 IP 地址

那么，MAC 地址，网络接口的物理地址呢？我们也可以通过 */sys* 虚拟文件系统获取这些信息：

```
$ **cat /sys/class/net/eth0/address**

02:42:ac:10:0a:0a 
```

在非强化主机上，你可能可以访问如 ifconfig 这样的网络工具，这是一个在 Linux 主机上非常流行的命令。这个命令可以让你以更易于理解的方式查看所有必要的网络信息：

```
$ **ifconfig**

eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.16.10.10  netmask 255.255.255.0  broadcast 172.16.10.255
        ether 02:42:ac:10:0a:0a  txqueuelen 0  (Ethernet)
        RX packets 97  bytes 211107 (211.1 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 83  bytes 5641 (5.6 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0 
```

你应该会收到每个接口的信息，如 MAC 地址、子网掩码和广播地址，以及一些网络统计数据，如发送和接收数据包的字节数。默认情况下，ifconfig 只会显示处于“up”状态的网络接口；使用 -a 标志可以显示所有接口。

ifconfig 的替代命令是 ip，它显示相同类型的信息，包括路由详情。运行 ip addr 显示所有网络接口，运行 ip route 显示所有网络路由。

尝试在其余的机器（*p-web-02* 和 *p-jumpbox-01*）上运行这些命令；你应该会注意到其中一台机器连接到了另一个内部网络，地址为 10.1.0.0/24。这意味着其中一台受损主机拥有进入另一个网络的网络接口！

### 连接和邻居

网络是非常活跃的；数据包不断进出系统。提供服务的主机很少闲置，你可以通过收集连接信息，主动了解它们的环境，而无需发送网络数据包。

尝试直接通过使用 */proc* 虚拟文件系统从 */proc/net/tcp* 文件收集这些信息：

```
$ **cat /proc/net/tcp**

 sl  local_address rem_address   st tx_queue rx_queue tr tm->when retrnsmt   uid  timeout inode
  0: 0B00007F:A0F1 00000000:0000 0A 00000000:00000000 00:00000000 00000000     0        0 4...
  1: 00000000:1F91 00000000:0000 0A 00000000:00000000 00:00000000 00000000     0        0 4... 
```

该文件的输出是一个 *TCP 套接字表*，每一行代表两个地址之间的连接：本地地址（local_address）和远程地址（rem_address）。数据是十六进制的，因此我们必须再次将其转换为十进制，以便理解每个连接背后的 IP 地址和端口：

```
$ **awk '{print $2,$3}' /proc/net/tcp | tail -n +2**

0B00007F:A0F1 00000000:0000
00000000:1F91 00000000:0000 
```

我们使用 awk 只打印第二列和第三列，然后将其通过管道传递给 tail -n +2 命令，以去除输出中的表头。随着更多连接在受损主机与其他客户端和服务器之间建立，这个表格将会增长。

你也可以使用 Netstat 来打印网络连接。Netstat 美化了每个连接的输出，并帮助突出显示当前哪些连接是活动的，哪些已经超时，以及它们与哪个 PID 和程序名称相关联。在 *p-web-01*（172.16.10.10）上运行以下命令：

```
$ **netstat -atnup**

Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      PID/Program name
tcp        0      0 127.0.0.11:41201        0.0.0.0:*               LISTEN     -
tcp        0      0 0.0.0.0:8081            0.0.0.0:*               LISTEN     1/python3
udp        0      0 127.0.0.11:45965        0.0.0.0:*                          - 
```

让我们关注对我们最有价值的列。第一列表示协议（例如，TCP 或 UDP），第四列是本地地址和端口，第五列是 *外部地址*（连接的远程地址），第六列是程序名称和 PID。请注意，当 Netstat 以非 root 用户身份执行时，PID 列可能没有填充 PID 和程序名称等信息。

当我们执行 Netstat 命令时，没有连接到 web 应用程序。让我们模拟一个传入连接，看看套接字表如何变化。在你的 Kali 主机上，运行以下 Netcat 命令：

```
$ **nc -v 172.16.10.10 8081**
```

接下来，在受损的 *p-web-01* 主机（172.16.10.10）上运行我们之前展示的 Netstat 命令：

```
$ **netstat -atnup**

Proto Recv-Q Send-Q Local Address     Foreign Address           State         PID/Program name
tcp        0      0 172.16.10.10:8081 **172.16.10.1:56520**         ESTABLISHED   1/python3 
```

如你所见，连接表中添加了一行，表示客户端通过 8081 端口连接时的远程 IP 地址。这个远程地址属于你运行 Netcat 的主机（在此案例中是 Kali）。

### 防火墙规则

主机的 *防火墙规则* 也是网络信息的来源。防火墙表可能包含一些规则，阻止某些网络或单个 IP 地址与主机通信。这些信息可以帮助我们了解其他邻近的网络、服务器或客户端。

在 Linux 服务器上常见的主机防火墙是 iptables。让我们运行以下 iptables 命令来查看 *p-web-01*（172.16.10.10）上的配置规则：

```
$ **iptables -L --line-numbers  -v**

Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source      destination
 **1      0     0 DROP       all  --  any    any     10.1.0.0/24 anywhere    /* Block Network */**

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source      destination

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source      destination 
```

如你所见，一条规则阻止了网络 10.1.0.0/24 连接到 *p-web-01* 主机；这再次表明 10.1.0.0/24 网络是存在的。请注意，使用 iptables 命令查看规则表通常需要提升权限。

### 网络接口配置文件

网络接口可能有专门的配置文件，例如，为特定接口静态配置网络 IP 地址，或确保网络卡默认在启动时启用。Linux 发行版可以将其网络配置放在不同的位置，但通常你会在以下位置找到它们：*/etc/network/interfaces*，*/etc/network/interfaces.d/*，*/etc/netplan/*，*/lib/netplan/*，*/run/netplan/*，以及 */etc/sysconfig/network-scripts/*。

如果是静态配置，网络接口可以揭示正在使用的 DNS 服务器。网络接口还可以提供诸如 IP 方案、网关地址等信息。以下是后续版本的基于 Ubuntu 的 Linux 系统中的静态网络配置文件：

```
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
     dhcp4: no
     addresses: [172.16.10.0/24]
     gateway4: 172.16.10.1
     nameservers:
       addresses: [8.8.8.8,8.8.4.4] 
```

该文件配置了 eth0 网络接口，默认网关为 172.16.10.1，并且配置了 Google DNS 服务器 8.8.8.8 和 8.8.4.4。

### 域名解析器

主机通常被配置为使用 DNS 将域名（如 *example.com*）转换为 IP 地址。DNS 服务器可以托管在本地网络中，也可以托管在其他地方，如公共云实例中。无论它们运行在哪里，都可能存在安全漏洞。

你可以在 Linux 操作系统的多个位置找到 DNS 服务器的配置，包括在 */etc/resolv.conf* 文件中，使用 nameserver 条目，格式如下：

```
$ **cat /etc/resolv.conf**

nameserver 127.0.0.11 
```

DNS 服务器还可以在 */etc/hosts* 配置文件中进行配置，如此处为 *p-web-01*（172.16.10.10）所示。这个 */etc/hosts* 文件可能包括一个你可以目标的其他网络和主机列表：

```
$ **cat /etc/hosts**
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.16.10.10    p-web-01.acme-hyper-branding.com p-web-01 
```

DNS 服务器也可以在单独的网络接口文件中进行配置，如前一部分所讨论的。

DNS 服务器还可以通过使用 *动态主机配置协议*（DHCP）服务器进行自动配置，DHCP 是一个负责动态分配网络配置的网络服务，在这种情况下，DNS 服务器不会在任何配置文件中明确设置。

## 软件安装

维护不当的操作系统映像通常会受到各种漏洞的影响，特别是如果它们默认安装了许多软件包。我们应该调查与操作系统捆绑的软件，因为它可以引导我们发现有趣的漏洞，帮助我们提升权限或获取未经授权的信息。

调查已安装软件的一种方法是使用包管理器。您将在 Linux 操作系统上找到几种常用的包管理器：在 Debian 和 Ubuntu 等系统上是高级软件包工具（APT），在 Red Hat、CentOS 和 Fedora 等系统上是 Yellowdog Updater Modified，以及在基于容器的操作系统（如 Alpine Linux）上是 Alpine Package Keeper。

尝试在任何受损主机上运行以下 apt 命令以列出已安装的软件包：

```
$ **apt list --installed**

Listing... Done
adduser/lunar,now 3.129ubuntu1 all [installed,automatic]
apt/lunar,now 2.6.0 amd64 [installed]
base-files/lunar,now 12.3ubuntu2 amd64 [installed]
base-passwd/lunar,now 3.6.1 amd64 [installed]
`--snip--` 
```

您可以通过使用 dpkg 获得稍微更好的输出。请注意，此命令主要在基于 Ubuntu 或 Debian 的 Linux 系统中找到：

```
$ **dpkg -l**

`--snip--`
ii  adduser            3.129ubuntu1           all        add and remove users and groups
ii  apt                2.6.0                  amd64      commandline package manager
ii  base-files         12.3ubuntu2            amd64      Debian base system miscellaneous files
`--snip--` 
```

要使用其他软件管理器获取软件包列表，您可以尝试以下任何一条命令：

yum list installed

apk list --installed

rpm -qa

我们可以使用 bash 解析这些软件包列表并获取软件的名称和版本，还可以进行一些聪明的搜索。要仅列出软件包名称，请运行此命令：

```
$ **apt list --installed | awk -F'/' '{print $1}'**
```

使用以下内容仅列出软件包版本：

```
$ **apt list --installed | awk '{print $2}'**
```

如果我们想搜索特定软件包并通过精确匹配搜索打印其版本，该怎么办？我们可以使用 awk 来实现：

```
$ **apt list --installed | awk -F'[/]' '$1 == "openssl" {print $3}'**
```

我们使用一个 awk 分隔符（-F），由斜杠和空格组成，并用方括号[/]括起来定义多个分隔符。然后检查第一个字段是否等于 openssl；如果是，则打印第三个字段，即版本字段。

我们甚至可以使用 awk 部分匹配软件包名称：

```
$ **apt list --installed | awk -F'[/]' '$1 ~ /openssl/ {print $3}'**
```

要查看安装的软件包总数，请运行 apt list 并将其管道传输到 wc（word count）命令：

```
$ **apt list --installed | wc -l**

341 
```

您可以使用这些软件包名称和版本作为查找漏洞数据来源的网站上的查询，例如国家漏洞数据库（*[`nvd.nist.gov`](https://nvd.nist.gov)*）或 MITRE 公共漏洞和暴露（CVE）数据库（*[`cve.mitre.org`](https://cve.mitre.org)*）。

请注意，包管理器可能不会列出服务器上安装的所有软件。例如，服务器可能直接从源安装 Java，而不使用包管理工具，在这种情况下，它不会显示在软件包列表中。

## 存储

从安全角度看，服务器存储有几个有趣的原因。多个服务器可以共享同一存储系统或使用它与最终用户共享文件。如果您可以写入存储系统，则可能能够在相邻服务器上实现代码执行，如果它们从受损的存储系统中获取文件（例如 shell 脚本）。

服务器存储可以是虚拟的或物理的，服务器可以运行在单个本地磁盘或多个本地磁盘上。服务器还可以使用多个磁盘来形成冗余阵列的廉价磁盘系统，这可以提高冗余性和性能，并且能够备份关键数据。

Linux 系统可以将远程存储系统挂载为本地目录（通常在 */mnt* 目录下）。这些可以作为操作系统的重要组成部分。你将看到远程存储通过网络附加存储或存储区域网络设备和协议（如网络文件系统或公共互联网文件系统）实现。

远程存储非常有用，因为系统可以将其用于多种目的：作为数据备份位置、集中式安全日志存储、远程文件共享，甚至存储远程用户的主文件夹。应用程序日志通常写入到远程存储设备的 */mnt/log_storage/* 文件夹中，该文件夹可能物理连接到完全不同的服务器。

让我们探索如何在受损主机上识别磁盘、分区和挂载点。

### 块设备

首先，让我们使用命令 lsblk 来查看存在哪些块设备。*块设备* 是数据存储设备，如光盘、软盘和硬盘。以下输出来自 *p-web-01*（172.16.10.10）：

```
$ **lsblk**

NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sr0     11:0    1 1024M  0 rom
vda    254:0    0   40G  0 disk
|-vda1 254:1    0   39G  0 part /etc/hosts
|                               /etc/hostname
|                               /etc/resolv.conf
|                               /mnt/scripts
|-vda2 254:2    0    1K  0 part
`-vda5 254:5    0  975M  0 part [SWAP] 
```

如你所见，我们有两个主要设备：sr0 和 vda。sr0 设备是 ROM 类型，vda 设备是磁盘类型。你在列表中看到的其他名称，如 vda1、vda2 和 vda5，都是 vda 磁盘的分区。对你可以访问的其余受损机器运行相同的命令，并记录下结果。

另一种查看分区列表的方法是读取 */proc/partitions*：

```
$ **cat /proc/partitions**

major minor  #blocks  name

 254        0   41943040 vda
 254        1   40941568 vda1
 254        2          1 vda2
 254        5     998400 vda5
`--snip--` 
```

*/proc* 文件系统还暴露了一个名为 */proc/mounts* 的文件，它提供了所有挂载点的列表、挂载选项以及挂载点的其他属性：

```
$ **cat /proc/mounts**

`--snip--`
shm /dev/shm tmpfs rw,nosuid,nodev,noexec,relatime,size=65536k,inode64 0 0
/dev/vda1 /mnt/scripts ext4 rw,relatime,errors=remount-ro 0 0
/dev/vda1 /etc/resolv.conf ext4 rw,relatime,errors=remount-ro 0 0
/dev/vda1 /etc/hostname ext4 rw,relatime,errors=remount-ro 0 0
/dev/vda1 /etc/hosts ext4 rw,relatime,errors=remount-ro 0 0 
```

或者，你也可以直接调用 mount 命令来获取这些信息：

```
$ **mount**

`--snip--`
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev type tmpfs (rw,nosuid,size=65536k,mode=755,inode64)
/dev/vda1 on /mnt/scripts type ext4 (rw,relatime,errors=remount-ro)
`--snip--` 
```

获取各种挂载文件系统视图的快速方法是使用 df 命令，该命令还会显示每个文件系统的可用和总磁盘大小：

```
$ **df -h -T**

Filesystem     Type     Size  Used Avail Use% Mounted on
overlay        overlay   39G   20G   18G  53% /
tmpfs          tmpfs     64M     0   64M   0% /dev
shm            tmpfs     64M     0   64M   0% /dev/shm
/dev/vda1      ext4      39G   20G   18G  53% /mnt/scripts 
```

-h 和 -T 标志将分别输出人类可读的版本和文件系统类型。

你可能已经注意到 *p-web-01*（172.16.10.10）上的挂载点 */mnt/scripts*。请记下这一点，因为它将在后续章节中派上用场。

### 文件系统标签文件

*/etc/fstab* 文件是一个静态配置文件，控制设备和分区的挂载。如果在没有必要的安全措施的情况下挂载设备和分区，可能会导致文件系统级的漏洞。

你可以使用特殊选项将设备或分区挂载到特定的文件系统位置，这些选项控制在挂载点上可以或不能做什么。例如，你可以配置一个来自远程存储系统的卷，在系统启动时挂载到 */mnt/external_storage* 上。你还可以将其配置为只读文件系统，这样就不允许写入，或者移除执行选项，以使用户无法从中运行二进制文件。

以下是渗透测试人员可能需要了解的一些有用的挂载选项：

dev 解释特殊块设备，例如设备文件。

nodev 与 dev 相反；不会解释特殊块设备。

noexec 禁止执行二进制文件。像 bash 这样的脚本仍然允许执行。

suid 允许使用设置了 setuid 标志的程序，用户可以使用文件的用户或组所有者的权限执行该程序。

nosuid 与 suid 选项相反；不允许使用设置了 setuid 标志的程序。

exec 允许执行二进制文件和其他类型的文件。

ro 禁止写入文件系统；换句话说，创建只读文件系统。

rw 允许对文件系统进行读写操作。

nosymfollow 限制跟踪文件系统上创建的符号链接。此选项仍然允许创建符号链接。

defaults 使用以下挂载选项：rw、suid、dev、exec 等。

如果你返回到前面显示的挂载命令输出，你将看到每个挂载点上已设置的挂载选项（如果已定义）。

## 日志

应用程序通常会生成某种类型的运行时输出，这些输出有时会写入日志文件。这些日志文件的内容会根据应用程序的不同而有所变化，但通常会指示一切是否正常工作，或者是否发生了错误。

某些日志文件是 Linux 操作系统的一部分，而其他则与第三方应用程序如 Web 服务器和数据库相关。此外，你还可能会找到由你进行渗透测试的公司编写的自定义应用程序日志。

在 Linux 系统上，系统和应用程序日志通常会写入到 */var/log* 目录。自定义应用程序可以将日志写入任何位置，但通常也会将其写入到 */var* 目录下的文件中。以下是一个示例查找命令，可以用于搜索日志文件：

```
$ **find / -name "*.log" -o -name "*.txt" -o -name "*.out" -type f 2> /dev/null**
```

这个命令用于查找扩展名为 *.log* 和 *.out* 的文件。

### 系统日志

以下是 Linux 系统上常见的系统日志列表：

*/var/log/auth.log                     /var/log/faillog*

*/var/log/secure                       /var/log/lastlog*

*/var/log/audit/audit.log         /var/log/dpkg*

*/var/log/dmesg                      /var/log/boot.log*

*/var/log/messages                 /var/log/cron*

*/var/log/syslog*

特别感兴趣的是诸如*/var/log/auth.log*、*/var/log/secure*和*/var/log/lastlog*等文件，它们与认证有关，可能包含有关连接到服务器的客户端的重要信息。*/var/log/audit/audit.log* 文件由审计系统如 Auditd 使用，用于记录命令行活动、认证尝试和一般系统调用等事件。

### 应用程序日志

应用程序日志还可能包含对渗透测试人员有趣的信息。例如，如果服务器运行网站，则 Web 引擎可能会生成关于连接到其的客户端以及请求的 Web 路径的日志。这可能会显示出网络上其他客户端和服务器。

像 Apache 和 nginx 这样的 Web 服务器通常将其日志写入*/var/log/apache2/*、*/var/log/httpd/*或*/var/log/nginx/*等目录。其他类型的应用程序，如代理、电子邮件服务器、打印服务器、文件传输服务器、关系数据库、消息队列和缓存数据库，也会生成您需要注意的日志。表 8-3 列出了您可能遇到的常见应用程序日志的位置。

表 8-3: 日志位置

| 日志类型 | 日志文件 |
| --- | --- |

| Web 服务器 | /var/log/apache2/access.log /var/log/httpd/access.log

/var/log/nginx/access.log

/var/log/lighttpd/access.log |

| 数据库 | /var/log/mysql/mysql.log /var/log/postgresql

/var/log/redis

/var/log/mongodb/mongod.log

/var/log/elasticsearch/elasticsearch.log |

| 打印服务器 | /var/log/cups |
| --- | --- |
| 文件传输服务器 | /var/log/vsftpd /var/log/proftpd |

| 监控系统 | /var/log/icinga2 /var/log/zabbix

/var/log/logstash

/var/log/nagios/nagios.log

/var/log/cacti |

注意，由于其敏感性，一些日志将需要提升的权限。

练习 13: 递归搜索可读取的日志文件

在此练习中，您将编写一个查找日志文件的脚本。它应执行以下操作：

1.  接受路径作为命令行输入。如果未指定参数，默认应使用*/var/log*。

2.  递归遍历路径以查找可读文件。

3.  将这些文件复制到您选择的中心化目录中。

4.  使用 tar 命令压缩文件夹。

为了帮助您编写脚本，我们建议您查看 find 命令，它具有许多强大的内置功能，可以按用户和组所有权进行搜索。

您可以在该书的 GitHub 仓库中找到一个完整的解决方案，*recursive_file_search.sh*。

## 内核和引导加载程序

操作系统如 Linux 的主要组件称为*内核*。内核负责核心功能，如进程和内存管理、驱动程序、安全性等。它是一个非常复杂的软件组件，因此容易受到漏洞的影响。一个内核漏洞的例子是*Dirty COW 漏洞*（CVE-2016-5195），它允许远程执行并获得 root 访问权限而不留下系统痕迹。

发现系统上运行的内核版本可能帮助你通过内核漏洞提升权限。要检查内核版本，请使用以下命令：

```
$ **uname -r**
```

由于实验室机器基于 Docker，它们共享主机（Kali）的内核，运行 `uname` 命令将会显示 Kali 的内核版本。

一个 Linux 系统可能会安装多个内核版本，以便在系统故障时进行回滚。内核文件位于 */boot* 目录下。你还可以通过运行以下命令来查看安装了哪些内核：

```
$ **rpm -qa | grep kernel**
$ **ls -l /boot | grep "vmlinuz-"** 
```

确保使用正确的包管理器命令来适应主机系统。

不稳定的内核漏洞利用程序非常危险，如果没有经过充分测试，它们可能会导致服务器崩溃。我们建议在尝试运行这些类型的漏洞时，先获得明确的授权。

## 配置文件

在本章中，我们已经提到了几种类型的配置文件。虽然这些文件高度依赖于具体的应用程序，但它们通常包含敏感数据。在本地侦察过程中，你需要特别关注它们，尤其是那些与 Web 应用程序相关的文件，因为 Web 应用程序通常依赖许多服务来完成日常操作。Web 应用程序需要连接这些服务，通常需要某种形式的身份验证，因此你可能会在附近找到凭据。

配置文件通常位于 */etc* 目录下，可能有也可能没有关联的文件扩展名，如 **.conf**、**.cfg**、**.ini**、*.cnf* 和 **.cf**。你也可能会在用户的隐藏目录下找到配置文件，例如 */home/user/.config/* 或 */home/user/.local*。要执行广泛的配置文件搜索，可以使用以下命令：

```
$ **find / -name "*.conf" -o -name "*.cf" -o -name "*.ini" -o -name "*.cfg" -type f 2> /dev/null**
```

要搜索特定文件夹，可以将命令中的 `find /` 部分更改为另一个目录，例如 `find /etc`。你甚至可以将多个目录连接在一起，如下所示：

```
$ **find /etc /usr /var/www-name "*.conf" -o -name "*.cf" -o -name "*.ini" -o –****name "*.cfg"**
**-type f 2> /dev/null** 
```

第三方软件通常也会包含自定义的配置，这可能很有趣。例如，WordPress 通常使用数据库来存储与博客相关的数据，其配置文件 *wp-config.php* 通常包含与数据库（如 MySQL）相关的凭据信息：

```
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'database_name_here');

/** MySQL database username */
define('DB_USER', 'username_here');

/** MySQL database password */
define('DB_PASSWORD', 'password_here'); 
```

这个文件的位置取决于 WordPress 的安装位置，因为它通常位于应用程序的根目录下，例如 */var/www/html/wp-config.php*。正如你所看到的，它有一个 .*php* 扩展名，因为 WordPress 是用 PHP 语言编写的。我们之前使用的搜索方法不会捕捉到这个文件，但我们可以调整命令，搜索包含 *config* 的文件：

```
$ **find / -name "*config*" 2> /dev/null**
```

我们已经知道 *p-web-02* 服务器（172.16.10.12）运行着 WordPress；你能找到它的配置文件吗？提示：它与应用程序一起位于 Web 根目录下。

了解常见的配置文件及其位置对于识别主机上正在运行的感兴趣服务很有帮助。表 8-4 列出了几个示例。

表 8-4：常见配置文件位置

| 服务器类型 | 文件位置 |
| --- | --- |

| Web 服务器 | /etc/httpd/httpd.conf /etc/httpd/conf/httpd.conf

/etc/apache2/apach2.conf

/etc/lighttpd/lighttpd.conf

/etc/nginx/nginx.conf |

| 文件共享和文件传输服务器 | /etc/vsftpd/vsftpd.conf /etc/protftpd.conf /usr/local

/etc/proftpd.conf

/etc/samba/smb.conf |

| 数据库 | /etc/mysql/my.cnf /etc/my.cnf

/etc/redis/redis.conf

/etc/mongo.conf

/etc/cassandra |

| 域名服务器 | /etc/bind/named.conf /etc/dnsmasq.conf |
| --- | --- |

| 邮件服务器 | /etc/postfix/main.cf /etc/mail/sendmail.cf

/etc/dovecot/dovecot.conf |

| 虚拟专用网络服务器 | /etc/openvpn /etc/ipsec.conf |
| --- | --- |

这个表格并不全面，但它应该能让你了解常见的网络服务器通常将其配置文件存储在哪里。

## 定时任务

*定时任务* 允许你为系统指定一个命令或脚本，系统将在指定的时间间隔自动运行。它们在渗透测试中非常有趣，因为它们经常以可以引发权限提升的方式编写。

例如，一个任务可能会读取并执行来自全局可写文件的指令，如果恶意用户能够将恶意指令写入这些文件，系统可能会以提升的权限执行它们。用户可能会采取恶意行为，例如创建特权用户、修改类似 */root* 这样的受保护文件夹的权限、为现有用户添加权限、启动自定义恶意进程，或删除或覆盖文件中的敏感信息。

在 Linux 中，我们有两种常见的任务调度机制：Cron 和 At。

### Cron

让我们编写一个小脚本，它创建一个文件并将当前的日期和时间附加到该文件中（清单 8-11）。

```
#!/bin/bash
job_name="my_scheduled_job"

echo "The time now is $(date)" >> "/tmp/${job_name}"

exit 0 
```

清单 8-11：一个简单的 Cron 任务

保存该文件并命名为 *cron_task.sh*。确保它是可执行的，使用 chmod u+x cron_task.sh 命令。

接下来，我们将使用 Cron 每分钟运行该脚本。运行以下命令以打开文本编辑器：

```
$ **crontab -e**
```

现在将以下内容附加到 */etc/crontab* 文件的末尾并保存。确保你更改路径为你保存脚本的位置：

```
* * * * * bash /path/to/cron_task.sh
```

你可能会问，为什么有五个星号（*）呢？Cron 有一种特殊的语法来描述它的执行计划。其格式如下：

```
Minutes (0-59), Hours (0-23), Days of the month (1-31), Month (1-12), Days of the week (0-6)
```

例如，以下语法描述了一个回显任务，它将在每天晚上 11:30 执行：

```
30 23 * * * echo "It is 23:30!" >> /tmp/cron.log
```

Cron 进程应该执行该脚本。为了确认它是否生效，可以在 */tmp* 文件夹中运行 ls。你应该会看到 */tmp/my_scheduled_job* 文件，其中包含关于时间的更新：

```
$ **cat /tmp/my_scheduled_job**

The time now is Mon May 22 03:11:01
The time now is Mon May 22 03:12:01
The time now is Mon May 22 03:13:01 
```

在渗透测试的背景下，Cron 任务可能不安全。例如，一个任务可能会将敏感文件复制到全局可读的路径，这样不信任的本地用户就可以访问这些文件。以下是一个备份任务的示例，如果它以 *root* 用户身份运行，则非常不安全：

```
30 23 1 * * tar czvf /home/backup.tar.gz /etc /var
```

像这样的 Cron 作业将会把敏感目录 */etc* 和 */var* 复制到 */home* 目录中。由于 */home* 目录对所有本地用户可访问，任何具有读取权限的用户都可以复制或查看这些文件。

表 8-5 列出了 Cron 在运行时使用的其他文件。

表 8-5: Cron 文件

| 目的 | 文件 |
| --- | --- |
| Cron 日志 | /var/spool/cron /var/spool/cron/crontab |

| 作业配置 | /etc/crontab /etc/cron.d |

/etc/cron.hourly

/etc/cron.daily

/etc/cron.weekly

/etc/cron.monthly |

| Cron 安全 | /etc/cron.deny /etc/cron.allow |
| --- | --- |

用户的 cron 作业通常存储在 */var/spool/cron/crontab/USER* 中，系统范围的 cron 作业定义在 */etc/crontab* 中。像 */etc/cron.hourly*、*/etc/cron.daily*、*/etc/cron.weekly* 和 */etc/cron.monthly* 这样的目录包含由 Cron 进程执行的 shell 脚本，而 */etc/crontab* 文件定义了这些目录中的脚本执行的时间间隔。

系统管理员可以限制用户创建 cron 作业。两个访问控制文件定义了谁可以运行 crontab 命令：*/etc/cron.allow* 和 */etc/cron.deny*。如果 */etc/cron.allow* 文件存在，列在该文件中的用户将能够使用 Cron 调度任务。如果该文件不存在，除非在 */etc/cron.deny* 中列出，否则所有用户都可以调度任务。如果两个文件都不存在，只有特权用户可以调度任务。如果一个用户同时出现在允许和拒绝文件中，该用户仍然可以调度任务。

### At

*At* 是 Linux 中的另一种作业调度工具，尽管它比 Cron 少见，且采用更简单的方式。它通过在 at 提示符中指定 shell 命令，或通过使用 | 将命令作为标准输入传递给 at 来工作。以下示例使用 at 提示符安排一个任务：

```
$ **at now + 1 minute**

warning: commands will be executed using /bin/sh
at Sat May 27 22:15:00
at> **rm -rf /home/user/.bash_history** 
```

我们首先指定调度，使用 now + 1 minute 告诉 At 在 1 分钟后运行命令。At 还支持其他格式的调度语法。以下是一些调度定义的示例：

```
$ **at 22:00**
$ **at 11pm + 3 days**
$ **at tomorrow**
$ **at Sunday**
$ **at May 27 2050** 
```

第一个示例安排命令在军用时间晚上 10 点运行。第二个示例安排在三天后的晚上 11 点运行。第三个示例安排在明天的当前时间运行，第四个示例安排在周日的当前时间运行。最后一个示例安排在 2050 年 5 月 27 日运行。

指定时间后，At 会将您的 shell 提示符切换到专用的命令行（at>），您可以逐行输入 shell 命令。要保存作业，使用 CTRL-D。

at 命令还提供了一种查看作业队列的方式（使用 atq）以及移除作业的方式（使用 atrm）。要列出所有排队的 At 作业，运行以下命令：

```
$ **atq**

1 Sun May 28 22:20:00 a root
2 Sun May 29 23:20:00 a root 
```

每个作业都有一个 ID（此例中为 1 和 2），它们的执行时间和调度该作业的用户。在提交作业后，您通常可以在 */var/spool/cron/atjobs* 下找到作业定义：

```
$ **ls -l /var/spool/cron/atjobs/**
total 8
-rwx------ 1 root daemon 2405 May 28 02:32 a0000101ac9454
-rwx------ 1 root daemon 2405 May 28 02:32 a0000201ac9454 
```

默认情况下，普通用户无法读取此目录。其他可能的 At 作业目录包括 */var/spool/cron/atspool*、*/var/spool/at* 和 */var/spool/at/spool*。  

你可以使用 `atrm` 命令后跟作业 ID 来删除排队的作业：  

```
$ **atrm 1**
```

和 Cron 一样，At 使用 deny（*/etc/at.deny*）和 allow（*/etc/at.allow*）文件来确定哪些用户可以调度作业。  

练习 14：编写 Cron 作业脚本以查找凭据  

这项练习的目标是编写一个监控 cron 作业脚本。这个脚本应该定期搜索系统中的文件，查找包含凭据的文件。创建一个 cron 作业来执行以下操作：  

1. 每 10 分钟运行一次，每周每天，全年无休。  

2. 在 */tmp* 目录下查找包含 *username* 或 *password* 字样的文件。  

3. 当找到这样的文件时，运行 grep 命令查找包含字符串的行，并将这些字符串写入你选择的可写位置。  

要测试你的脚本，你可以创建一个包含字符串 `username=administrator` 或 `password=12345` 的假文件，并将其保存到 */tmp* 目录。如果你的 cron 作业按预期工作，你应该能够在目标目录中看到这两个字符串。  

## 硬件  

你可以收集与硬件相关的信息，例如内存分配详情、CPU 和核心数量，以及硬件组件的制造商，例如主板、网卡和其他外设。要收集这些信息，你可以使用如 lshw、dmidecode 和 hwinfo 等命令。  

这些命令在使用非特权用户运行时可能只显示部分信息，因为它们通常从仅 *root* 用户可访问的系统文件中读取。它们也可能不是默认安装的，因此你可能需要手动通过查看 */proc*、*/dev* 和 */sys* 下的特定文件和目录来收集硬件信息。  

让我们来看一下在其中一台实验室机器上运行 lshw 命令时得到的输出，例如 *p-web-01*（172.16.10.10）：  

```
$ **lshw**
```

请记住，我们的实验室是虚拟的，因此输出可能无法准确报告底层物理硬件的信息，如内存大小、主板厂商和声卡。  

lshw 命令带有一个 -class (-C) 参数，允许你查看特定类别的硬件，例如磁盘（-C disk）、处理器（-C cpu）和网络（-C network）：  

```
$ **lshw -C disk**

  *-disk
       description: ATA Disk
       product: VBOX HARDDISK
       vendor: VirtualBox
       size: 80GiB (86GB)
`--snip--` 
```

在这个磁盘示例中，你可以看到厂商名称是 VirtualBox，这暗示我们在虚拟机中运行了这个命令。  

硬件实用工具从各种文件中收集信息。表 8-6 汇总了这些工具从中聚合硬件信息的一些文件和目录。  

表 8-6：文件系统中的硬件信息位置  

| 虚拟文件系统 | 文件和目录 |   |
| --- | --- | --- |
| --- | --- |   |

| /proc | /proc/bus/usb/devices /proc/dma  

/proc/interrupts  

/proc/partitions  

/proc/modules  

/proc/cpuinfo  

/proc/devices-tree  

/proc/devices  

/proc/efi/systab

/proc/ide  

/proc/kcore  

/proc/mounts  

/proc/net/dev  

/proc/scsi  

/proc/sys  

/proc/sys/abi  

/proc/sys/dev/sensors |  

| /sys | /sys/bus /sys/class

/sys/devices

/sys/firmware

/sys/firmware/dmi/tables/DMI |

| /dev | /dev/cdrom /dev/input

/dev/fb*

/dev/machines

/dev/snd

/dev/mem

/dev/scsi* |

## 虚拟化

管理员可以直接在物理服务器上安装操作系统，或者运行一个虚拟化程序（如 VirtualBox、Microsoft Hyper-V 或 VMware ESXi）在同一硬件上托管多个虚拟机。或者，他们可能使用容器技术将虚拟服务器作为容器运行。

确定一个环境是虚拟的还是物理的，通常在防御规避的背景下很重要。例如，恶意软件常常会检查虚拟环境，以便规避逆向工程的尝试，因为分析人员通常会在这种虚拟环境中分析恶意软件。

和之前的场景一样，我们可以使用专用工具以及“利用现有资源”方法来获取这些信息。我们将探索这两种方法。

### 使用专用工具

像 virt-who 和 virt-what 这样的工具可以检查系统，以确定它是物理系统还是虚拟系统。以下是 virt-what 在 VirtualBox 上的 Kali 系统中的输出结果：

```
$ **sudo apt install -y virt-what**
$ **sudo virt-what**

virtualbox
kvm 
```

另一个有用的工具 systemd-detect-virt 提供了一份详尽的虚拟环境枚举技术列表，用于 systemd 系统。它能够识别多个虚拟机监控程序和容器运行环境，您可以在这里找到相关列表：*[`www.freedesktop.org/software/systemd/man/systemd-detect-virt.html`](https://www.freedesktop.org/software/systemd/man/systemd-detect-virt.html)*。

尝试在实验室的任意一台机器上运行 systemd-detect-virt 查看输出结果：

```
$ **systemd-detect-virt**

docker 
```

使用 dmesg 命令，您还可以从内核环形缓冲日志中读取虚拟化信息：

```
$ **dmesg | grep  "Detected virtualization"**

[1075720.226245] systemd[1]: Detected virtualization oracle. 
```

在这个例子中，oracle 是虚拟化软件，因为我们正在运行由 Oracle 开发和维护的 VirtualBox。

### 利用现有资源

让我们强调几种可以确定系统是否在虚拟环境中运行的方式。

桌面管理接口（DMI）是一个用于系统硬件和软件管理跟踪的框架。在*/sys/class/dmi/id*目录下，一些与 DMI 相关的文件可能会泄露有关各种虚拟化供应商的信息。这些文件包括*product_name*、*sys_vendor*、*board_vendor*、*bios_vendor*和*product_version*。查看它们的内容：

```
$ **cat /sys/class/dmi/id/product_name**
VirtualBox

$ **cat /sys/class/dmi/id/board_vendor**
Oracle Corporation 
```

文件*/sys/hypervisor/type*也可能暗示底层的虚拟化程序。例如，Xen 虚拟化程序可能会在该文件中插入值 xen，而 Microsoft Hyper-V 则会使用 Hyper-V。

另一个只有*root*用户可以访问的文件，*/proc/1/environ*，可能包含一个名为 container= 的环境变量，其中包含相关信息。例如，Linux 容器可能会使用 container=lxc，而 Podman 容器可能会使用 container=podman。

一些容器技术，包括 Podman 和 Docker，使用放置在特定位置的*env*文件。任何一个文件的存在都表明这是一个容器环境：

*/run/.containerenv*

*/.dockerenv*

在 systemd 系统上，可能存在 */run/systemd/container* 文件：

```
$ **cat /run/systemd/container**

Docker 
```

尝试在你能访问的任何实验机上运行此命令。

## 使用 LinEnum 自动化信息收集

到现在，你应该已经意识到，操作系统上任何地方都可能存在有价值的信息。为了高效地覆盖某些基础领域，包括用户和组、cron 任务、进程等，我们可以运行信息收集脚本，这些脚本依赖于文件位置的可预测性和常见的搜索模式。

*LinEnum* 是一个本地信息收集的 shell 脚本，用于自动从主机收集数据。它覆盖了系统信息、用户信息、服务和进程、版本和权限等收集领域。

让我们使用 LinEnum 以自动化方式在本地收集文件。首先，我们需要将 LinEnum 放到被攻破的机器上。由于它是一个单一的 shell 脚本文件，我们可以简单地将其复制并粘贴到机器上的新文件中。复制 */home/kali/tools/LinEnum.sh* 的内容，并将该文件保存为 *LinEnum.sh*，然后放到被攻破的机器上。

现在运行 LinEnum 时，使用 -t（彻底收集）和 -r（报告）选项来指定一个文件，将输出发送到该文件：

```
$ **chmod u+x LinEnum.sh**
$ **./LinEnum.sh -t -r report.txt**

#########################################################
# Local Linux Enumeration & Privilege Escalation Script #
#########################################################
`--snip--`
[-] Debug Info
[+] Report name = report.txt
[+] Thorough tests = Disabled
`--snip--` 
```

阅读收集到的结果，以查看收集了哪些信息。在接下来的练习中，你将阅读 LinEnum 的代码，构建新功能，并根据自己的需求进行定制。

练习 15：为 LinEnum 添加自定义功能

在渗透测试过程中，你可能会发现自己需要重新使用概念验证漏洞代码和脚本来适应特定的用例。这是一个非常重要的技能，因为如果你能避免从头编写脚本，就能节省大量时间。

在本练习中，你的目标是修改 LinEnum 的源代码，为其添加新功能：

1.  仔细阅读 LinEnum 脚本的源代码。虽然它大约包含 1300 行代码，但应该相对容易理解，因为它遵循一致的模式，例如执行命令并将输出保存到变量中。

2.  修改源代码，收集你感兴趣的文件内容，或者是 LinEnum 尚未收集的文件内容。或者，实现你自己的新功能的想法。

3.  为 LinEnum 添加另一个命令行选项，使用 tar 命令将报告压缩为 *tar.gz* 文件（-c 选项）。

阅读外部代码与编写代码同样重要。每个人都有自己编写代码的风格和实现逻辑的方式，你可以从中学到很多关于工具内部结构以及如何根据自己的需求定制它们的知识。

## 总结

在本章中，我们强调了你可以在被攻破的主机上进行的数据收集的主要类别，例如操作系统和内核、相邻网络和连接、正在运行的进程和用户活动会话、环境数据、用户和组身份、系统和第三方日志文件以及配置文件。此外，我们使用 Cron 和 At 来调度执行 shell 脚本。

随着你阅读本书的过程，你将继续收集数据，以帮助特权升级、凭证访问和其他不端黑客活动。
