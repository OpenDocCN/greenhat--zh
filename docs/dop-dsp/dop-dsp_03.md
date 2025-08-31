# 第二章：使用 Ansible 管理密码、用户和组

![](img/chapterart.png)

现在你已经构建好了虚拟机（VM），让我们开始进行一些管理任务，比如用户管理。DevOps 实践中的自动化是构建和管理资源的关键。要管理任何 Linux 主机，你需要对密码、用户和组的工作原理有基本的理解。用户和密码是身份管理的基础，而组则使你能够管理一组用户并控制对文件、目录和命令的访问。通过在用户和组之间划分责任，可能会决定是否允许未经授权的访问。

在本章中，你将继续学习如何使用 Ansible，并且还将配置你刚刚创建的虚拟机，以改善你的基本安全策略。你将使用本书提供的 Ansible 任务来强制执行复杂密码、管理用户和组，以及控制对共享目录和文件的访问。一旦你掌握了这些安全基础，你就能将它们作为每个 playbook 的基础。

## 强制执行复杂密码

让用户决定什么是强密码可能会导致灾难，因此你需要在每个用户可以访问的主机上强制执行复杂密码。由于自动化是我们的指导原则之一，你将使用代码来强制执行所有用户的强密码。为此，你可以使用一个 Ansible 任务来安装一个 *可插拔认证模块* *(PAM)* 插件，这是大多数 Linux 发行版使用的用户认证框架。提供复杂密码的插件叫做 `pam_pwquality`。该模块根据你设置的标准验证密码。

### 安装 libpam-pwquality

`pwquality` PAM 模块可以在 Ubuntu 软件仓库中找到，名称为 `libpam-pwquality`。你将使用本书提供的 Ansible 任务来安装和配置此软件包。记住，目标是尽可能地自动化所有内容，任务提供了执行管理工作的机制。这些任务位于你从简介中克隆的仓库中。导航到 *ansible/chapter2/* 目录，并在你喜欢的编辑器中打开 *pam_pwquality.yml* 文件。该文件包含两个任务：`安装 libpam-pwquality` 和 `配置 pam_pwquality`。

让我们专注于第一个任务，使用 Ansible 的 `package` 模块在虚拟机上安装 `libpam-pwquality`。文件顶部的安装任务应该像这样：

```
---
- name: **Install libpam-pwquality**
  package:
    name: "libpam-pwquality"
    state: present
`--snip--`
```

每个 Ansible 任务应以 `name` 声明开始，用于定义其目标。在本例中，`name` 为 `Install libpam-pwquality`。接下来，Ansible 的 `package` 模块执行软件安装。`package` 模块要求你设置两个参数：`name` 和 `state`。在本例中，软件包名称（可在 Ubuntu 仓库中找到）应为 `libpam-pwquality`，而 `state` 应设置为 `present`。要删除软件包，将 `state` 设置为 `absent`。这是声明性指令的一个很好的示例，因为你告诉 Ansible 确保安装了这个软件包。你无需关心它是如何被安装的，只要确保它被安装即可。如果你安装了该软件包（`present`），然后从 Ansible 中删除该任务，下次提供时，软件包仍将被安装。如果你希望主机保持所需状态，必须显式将软件包设置为 `absent`。

如第一章所述，Ansible 模块（如上所示）在操作系统上执行常见操作，例如启用防火墙、管理用户或（在本例中）安装软件。Ansible 使得你的操作能够是*幂等*的，这意味着你可以反复执行某个特定操作，并且结果将与上次执行时相同。正因如此，你应该尽可能自动化所有工作！这样你不仅能节省时间，还能避免因手动操作疲劳导致的错误。试想一下，如果你每天需要配置 1,000 台机器，没了自动化，几乎是不可能完成的！

### 配置 pam_pwquality 以强制执行更严格的密码策略

在默认的 Ubuntu 系统中，密码复杂度并不像它应该的那样强大。它要求密码的最小长度为六个字符，并且仅执行一些基本的复杂度检查。要加强复杂度，你需要配置 `pam_pwquality` 来设置更严格的密码策略。

一个名为 */etc/pam.d/common-password* 的文件负责配置 `pam_pwquality` 模块。这个文件是 Ansible 任务用来对密码进行验证并做出必要更改的地方。你所需要做的就是修改该文件中的一行。使用 Ansible 编辑文件中的某一行的常见方法是使用 `lineinfile` 模块，该模块可以更改文件中的一行或检查某一行是否存在。

在仍然打开的 `pam_pwquality` 任务文件中，让我们回顾一下从顶部开始的第二个任务。它应如下所示：

```
`--snip--`
- name: **Configure pam_pwquality**
  lineinfile:
    path: "/etc/pam.d/common-password"
    regexp: "pam_pwquality.so"

    line: "password required pam_pwquality.so minlen=12 lcredit=-1 ucredit=-1 
           dcredit=-1 ocredit=-1 retry=3 enforce_for_root”
    state: present
`--snip--`
```

再次，任务以名称 `Configure pam_pwquality` 开始，描述其目的。然后，它告诉 Ansible 使用 `lineinfile` 模块来编辑 PAM 密码文件。`lineinfile` 模块要求提供文件的 `path`，以便对其进行更改。在此案例中，它是 PAM 密码文件 */etc/pam.d/common-password*。使用正则表达式（*regexp*）来找到要更改的文件行。正则表达式定位到包含 `pam_pwquality.so` 的行，并用新的一行替换它。替换的 `line` 参数包含 `pwquality` 配置更改，这些更改强制执行更复杂的密码要求。上面提供的选项强制执行以下要求：

+   密码最小长度为 12 个字符

+   一个小写字母

+   一个大写字母

+   一个数字字符

+   一个非字母数字字符

+   三次重试

+   禁用 root 覆盖

添加这些要求将加强 Ubuntu 默认的密码策略。任何新密码都需要满足或超过这些要求，从而使攻击者暴力破解用户密码变得更加困难。

关闭 *pam_pwquality.yml* 文件，这样你就可以继续使用 Ansible 模块来创建用户。

## Linux 用户类型

说到 Linux，用户可以分为三种类型：普通用户、系统用户和 root 用户。你可以将*普通用户*看作是人类账户，接下来你将创建一个这样的账户。每个普通用户通常都会关联一个密码、一个组和一个用户名。将*系统用户*看作是非人类账户，比如 Nginx 运行的用户。事实上，系统用户与普通用户几乎相同，但它位于不同的用户 ID（UID）范围内，出于隔离的考虑。*root 用户*（或*超级用户*）账户对操作系统有无限制的访问权限。你可以通过 UID 来辨别 root 用户，它的 UID 始终是零。与所有的配置一样，当涉及到创建和配置用户时，你将使用 Ansible 模块来进行重任操作。

### Ansible 用户模块入门

Ansible 配有 `user` 模块，使得管理用户变得非常简单。它处理账户的所有繁琐细节，比如 shell、密钥、组和主目录。你将使用 `user` 模块创建一个名为 *bender* 的新用户。如果你愿意，可以取一个别的名字，但由于本书中的示例将继续使用 *bender* 这个用户名，记得在以后的章节中也将名字更改为 *bender*。

打开位于 *ansible/chapter2/* 目录下的 *user_and_group.yml* 文件。该文件包含以下五个任务：

1.  确保 *developers* 组存在。

1.  创建用户 *bender*。

1.  将 *bender* 分配给 *developers* 组。

1.  创建一个名为 *engineering* 的目录。

1.  在工程目录中创建一个文件。

这些任务将创建一个组和一个用户，将用户分配到组中，并创建一个共享目录和文件。

尽管这有些违反直觉，我们先从列表中的第二个任务开始，即创建用户*bender*。（我们将在下一页的“Linux 群组”部分讨论第一个任务。）它应该如下所示：

```
`--snip--`
- name: **Create the user 'bender'**
  user:
    name: bender
    shell: /bin/bash
    password: $6$...(truncated)
`--snip--`
```

这个任务，像其他任务一样，以描述其功能的`name`开头。在这个例子中，`name`是`Create the user 'bender'`（创建用户'bender'）。你将使用 Ansible 的`user`模块来创建用户。`user`模块有许多选项，但只有`name`参数是必需的。在本例中，`name`被设置为`bender`。在配置时设置用户密码是有用的，因此可以将可选的`password`参数设置为已知的密码哈希值（稍后会详细介绍）。以`$6`开头的`password`值是 Linux 支持的加密哈希。我已经提供了*bender*的密码哈希示例，展示如何自动化此步骤。在下一部分，我将详细介绍我生成密码哈希的过程。

### 生成复杂密码

你可以使用多种不同的方法生成密码，以匹配你在`pam_pwquality`中设置的复杂度要求。如前所述，我提供了一个密码哈希值，符合这一阈值，以节省时间。我使用了两个命令行应用程序，`pwgen`和`mkpasswd`，来创建复杂密码。`pwgen`命令可以生成安全密码，而`mkpasswd`命令可以使用不同的哈希算法生成密码。`pwgen`应用程序由`pwgen`包提供，`mkpasswd`应用程序由名为`whois`的包提供。这些工具结合在一起，可以生成 Ansible 和 Linux 所期望的哈希值。

Linux 将密码哈希值存储在名为*shadow*的文件中。在 Ubuntu 系统中，默认的密码哈希算法是 SHA-512。要为 Ansible 的用户模块创建自己的 SHA-512 哈希，请在 Ubuntu 主机上使用以下命令：

```
$ **sudo apt update**
$ **sudo apt install pwgen whois**
$ **pass=`pwgen --secure --capitalize --numerals --symbols 12 1`**
$**echo $pass | mkpasswd --stdin --method=sha-512; echo $pass**
```

由于这些软件包默认没有安装，你需要先使用 APT 包管理器安装它们。`pwgen`命令生成符合`pwquality`要求的复杂密码，并将其保存在一个名为`pass`的变量中。接下来，将`pass`变量的内容通过管道传输到`mkpasswd`中，使用`sha-512`算法进行哈希处理。最终输出应包含两行。第一行是 SHA-512 哈希值，第二行是新密码。你可以将哈希字符串拿来，并在用户创建任务中设置`password`值以更改密码。尽管如此，尽情尝试吧！

## Linux 群组

Linux 群组允许你在主机上管理多个用户。创建群组也是限制访问主机资源的一种高效方式。对群组进行管理比对成百上千的用户逐个管理要容易得多。在下一个示例中，我提供了一个 Ansible 任务，创建一个名为*developers*的群组，你将使用它来限制对某个目录和文件的访问。

### 开始使用 Ansible 群组模块

与`user`模块类似，Ansible 还有一个`group`模块，可以管理创建和删除组。与其他 Ansible 模块相比，`group`模块非常简洁；它只能创建或删除组。

在您的编辑器中打开*user_and_group.yml*文件，以查看组创建任务。文件中的第一个任务应该是这样的：

```
- name: **Ensure group 'developers' exists**
  group:
    name: developers
    state: present
`--snip--`
```

任务的`name`字段表明您希望确保组存在。使用`group`模块创建组。此模块要求您设置`name`参数，在此处设置为`developers`。`state`参数设置为`present`，因此如果组不存在，则会创建该组。

文件中的第一个任务是创建组，这并非偶然。在执行任何其他任务之前，您需要创建**开发者**组。任务按顺序运行，因此您需要确保组首先存在。如果在创建组之前尝试引用该组，则会收到错误消息，指出**开发者**组不存在，且配置将失败。理解 Ansible 任务操作顺序对执行更复杂的操作至关重要。

继续查看其他任务时，请保持*user_and_group.yml*文件打开状态。

### 分配用户到组

要使用 Ansible 将用户添加到组中，您将再次利用`user`模块。在*user_and_group.yml*文件中，找到将*bender*分配给**开发者**组的任务（文件中从顶部算起第三个任务）。它应该看起来像这样：

```
`--snip--`
- name: **Assign 'bender' to the 'developers' group**
  user:
    name: bender
    **groups: developers**
    append: yes
`--snip--`
```

任务的`name`字段描述了其意图。`user`模块将*bender*追加到**开发者**组。`groups`选项可以接受逗号分隔的多个组。通过使用`append`选项，您保留了*bender*之前的所有组，并仅添加了**开发者**组。如果省略`append`选项，则*bender*将从除其主要组和`groups`参数中列出的组之外的所有组中移除。

### 创建受保护的资源

确定了*bender*的组关联后，让我们来看看*user_and_group.yml*文件中的最后两个任务，这些任务涉及在虚拟机上创建一个目录（*/opt/engineering/*）和一个文件（*/opt/engineering/private.txt*）。稍后您将使用该目录和文件来测试*bender*的用户访问权限。

仍然在*user_and_group.yml*文件中，找到这两个任务。首先是目录创建任务（文件中从顶部算起第四个任务），它应该看起来像这样：

```
- name: **Create a directory named 'engineering'**
  file:
    path: /opt/engineering
    state: directory
    mode: 0750
    group: developers
```

首先，像之前一样，将 `name` 设置为匹配任务的意图。使用 `file` 模块来管理目录及其属性。`path` 参数指定了你希望创建目录的位置。在这个例子中，它被设置为 */opt/engineering/*。因为你希望创建一个目录，所以将 `state` 参数设置为你想创建的资源类型，这里是 `directory`。你还可以使用其他类型，稍后你创建文件时会看到另一个。`mode`（权限）设置为 `0750`。这个数字允许所有者（*root*）对该目录进行读取、写入和执行操作，而组成员仅允许读取和执行。执行权限是进入目录并列出其内容所必需的。Linux 使用八进制数字（此例中为 `0750`）来定义文件和组的权限。有关权限模式的更多信息，请参见 `chmod` 的手册页。最后，将目录的 `group` 所有权设置为 *developers* 组。这意味着只有 *developers* 组中的用户才能读取或列出该目录的内容。

*user_and_group.yml* 文件中的最后一个任务会在你刚创建的 */opt/engineering/* 目录内创建一个空文件。位于文件底部的任务应该像这样：

```
- name: **Create a file in the engineering directory**
  file:
    path: "/opt/engineering/private.txt"
    state: touch
    mode: 0770
    group: developers
```

将任务的 `name` 设置为你想在主机上执行的操作。再次使用 `file` 模块来创建一个文件并设置一些属性。`path` 是必填项，指定了文件在虚拟机中的位置。这个例子演示了在 */opt/engineering/* 目录中创建一个名为 *private.txt* 的文件。`state` 参数设置为 `touch`，意味着如果文件不存在，就创建一个空文件。如果你需要创建一个非空文件，可以使用 `copy` 或 `template` 这两个 Ansible 模块。更多细节请参见文档。`mode`（权限）设置为允许组中的任何用户读取、写入和执行（`0770`）。最后，将文件的 `group` 所有权设置为 *developers* 组。

理解这一点非常重要：你可以使用多种方法来保护 Linux 主机上的资源。组限制只是生产环境中更大授权体系的一小部分。我将在后续章节讨论不同的访问控制。但现在，你只需要知道，借助 Ansible 的任务和模块，你可以在整个环境中执行许多常见的系统配置任务，比如保护文件和目录。

## 更新虚拟机

到目前为止，我们一直在描述 Ansible 模块，并回顾将为虚拟机提供配置的任务。下一步实际上是使用它们。要配置虚拟机，你需要取消注释位于 *ansible/* 目录下 playbook 中的任务。*site.yml* 文件是你在 Vagrantfile 的配置器部分引用的 playbook 文件。

打开编辑器中的 *site.yml* playbook 文件，找到第二章的任务，它们看起来像这样：

```
`--snip--`
tasks:
 **#-** **import_tasks****: chapter2/****pam_pwquality.yml**
 **#-** **import_tasks****: chapter2/****user_and_group.yml**
`--snip--` 
```

它们被注释掉了。移除两行开头的哈希符号（`#`）以取消注释，这样 Ansible 才能执行这些任务。

现在，剧本应该如下所示：

```
---
- name: Provision VM
  hosts: all
  become: yes
  become_method: sudo
  remote_user: ubuntu
  tasks:
    - import_tasks: chapter2/pam_pwquality.yml
    - import_tasks: chapter2/user_and_group.yml
`--snip--`
```

第二章中的两个任务，`pam_pwquality`和`user_and_group`，现在都已取消注释，因此它们将在下次配置虚拟机时执行。暂时保存并关闭剧本文件。

你在第一章创建了虚拟机。然而，如果虚拟机未运行，请输入`vagrant up`命令重新启动它。虚拟机运行后，只需在*vagrant/*目录中运行`vagrant provision`命令来执行配置程序：

```
$ **vagrant provision**
`--snip--`
PLAY RECAP *********************************************************************
Default : ok=8    changed=7   unreachable=0   failed=0   skipped=0    rescued=0   ignored=0
```

最后一行显示 Ansible 剧本已经运行并完成了`8`个操作。可以把*操作*看作是执行的任务和其他操作。八个操作中有七个改变了虚拟机的某些状态。这一行显示配置已完成且没有错误或失败的操作。

如果你的配置出现故障，停止并尝试排查问题。再次运行`provision`命令并加上`--debug`标志，如第一章所示，以获取更多信息。你需要成功的配置才能继续进行本书中的示例。

## 测试用户和组权限

为了测试你刚刚配置的用户和组权限，你将发出`ssh`命令让`vagrant`访问虚拟机。确保你在*vagrant/*目录中，这样你才能访问 Vagrantfile。进入该目录后，在终端中输入以下命令以登录到虚拟机：

```
$ **vagrant ssh**
vagrant@dftd:~$
```

你应该以*vagrant*用户身份登录，这是 Vagrant 默认创建的用户。

接下来，为了验证用户*bender*是否已创建，你将使用`getent`命令查询*passwd*数据库中的用户信息。此命令允许你查询像*/etc/passwd*、*/etc/shadow*和*/etc/group*这样的文件中的条目。要检查*bender*是否存在，请输入以下命令：

```
$ **getent passwd bender**
bender:x:1002:1003::/home/bender:/bin/bash
```

你的结果应类似于上面的输出。如果用户没有被创建，命令将没有任何结果。

现在，你应该检查*developers*组是否存在，以及*bender*是否是该组成员。查询*group*数据库以获取此信息：

```
$ **getent group developers**
developers:x:1002:bender
```

结果应类似于上面的输出，显示一个*developers*组，并将用户*bender*分配给它。如果该组不存在，命令将没有任何输出结果。

最后的检查是测试只有*developers*组的成员才能访问*/opt/engineering/*目录和*private.txt*文件。为此，尝试先以*vagrant*用户身份访问该目录和文件，然后再以*bender*用户身份访问。

当以*vagrant*用户身份登录时，输入以下命令列出*/opt/engineering/*目录及其内容：

```
$ **ls -al /opt/engineering/**
ls: cannot open directory '/opt/engineering/': Permission denied
```

输出显示，当尝试以*vagrant*用户身份列出*/opt/engineering*中的文件时，访问被`denied`。这是因为*vagrant*用户不是*developers*组的成员，因此无法读取该`目录`。

现在，要测试 *vagrant* 的文件权限，使用 `cat` 命令查看 */opt/engineering/private.txt* 文件：

```
$ **cat /opt/engineering/private.txt**
cat: /opt/engineering/private.txt: Permission denied
```

由于 *vagrant* 用户没有读取文件的权限，因此发生了相同的错误。

下一步是验证 *bender* 是否能够访问相同的目录和文件。为此，必须以 *bender* 用户登录。从 *vagrant* 切换到 *bender* 用户，使用 `sudo` `su` 命令。（我将在第四章中讲解 `sudo` 命令。）

在终端中，输入以下命令来切换用户：

```
vagrant@dftd:~$ **sudo su - bender**
bender@dftd:~$
```

一旦成功切换用户，再次尝试命令列出目录：

```
$ **ls -al /opt/engineering/**
total 8
drwx`r`-`x`--- 2 root `developers`    4096 Jul  3 03:59 .
drwxr-xr-x 3 root root          4096 Jul  3 03:59 ..
-rwx`rwx`--- 1 root `developers`       0 Jul  3 04:02 private.txt
```

如您所见，您已经成功以 *bender* 访问了该目录及其内容，并且 *private.txt* 文件是可查看的。

接下来，输入以下命令检查 *bender* 是否可以读取 */opt/engineering/private.txt* 文件的内容：

```
$**cat****/opt/engineering/private.txt**
```

您再次使用 `cat` 命令查看文件内容。由于文件为空，未有输出。更重要的是，*bender* 在尝试访问文件时没有出现错误。

## 总结

在本章中，您使用以下 Ansible 模块配置了虚拟机：`package`、`lineinfile`、`user`、`group` 和 `file`。这些模块配置了主机以强制执行复杂密码、管理用户和组，以及确保文件和目录的访问安全。这些是 DevOps 工程师在典型环境中常做的任务。您不仅扩展了 Ansible 知识，还学会了如何在虚拟机上自动化基本的安全卫生工作。在下一章中，您将继续完成任务，并提高 SSH 安全性，限制对虚拟机的访问。
