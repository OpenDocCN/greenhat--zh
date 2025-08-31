# 第四章：使用 sudo 控制用户命令

![](img/chapterart.png)

到目前为止，你已经通过公钥和双因素认证确保了对虚拟机的访问。你还通过使用组权限控制了对特定文件和目录的访问。接下来的基础步骤是允许用户在虚拟机上运行提升权限的命令。用户通常需要访问一些可能需要管理员权限的命令，比如重启服务或安装缺失的软件包。作为管理员，你希望严格控制谁可以运行哪些命令。在 Linux 操作系统上，`sudo`（超级用户执行）命令允许用户以*root*或其他用户的身份执行特定命令，同时保持事件的审计痕迹。

在本章中，你将使用 Ansible 安装一个简单的 Python Flask web 应用程序。你还将使用 Ansible 创建一个*sudoers*安全策略，这个策略由一个文件配置，决定了用户在调用`sudo`命令时拥有的权限。这个策略将允许*developers*组的成员使用`sudo`命令启动、停止、重启以及编辑示例 web 应用程序。虽然这是一个假设的例子，但它遵循了软件工程师应熟悉的典型发布工作流程。在本章结束时，你将对如何自动化应用程序部署并通过*sudoers*策略进行控制有一个清晰的理解。

## 什么是 sudo？

如果你是`sudo`的新手，它是大多数 Unix 操作系统中的一个命令行工具，允许用户或用户组以其他用户的身份执行命令。例如，一个软件工程师可能需要重启由*root*用户拥有的 Nginx web 服务器，或者系统管理员可能需要提升权限来安装一些软件包。如果你在 Linux 上待得够久，你可能已经使用过`sudo`来执行需要提升权限的命令。通常，你不会允许任何人拥有这种权限，因为这会涉及到各种安全问题。无论你的使用场景如何，用户都需要一种安全且可追溯的方式来访问特权命令，以完成他们的工作。

`sudo`的最佳特性之一是它能够留下审计痕迹。如果有人使用`sudo`执行命令，你可以查看日志，看看是谁执行了什么命令。如果没有`sudo`，如果你盲目允许人们切换到其他用户去执行命令，就没有任何责任追踪。

你还可以通过插件增强`sudo`。实际上，`sudo`自带一个名为*sudoers*的默认安全策略插件，它决定了用户在调用`sudo`命令时拥有的权限。你将为用户*bender*实现这一策略。

### 规划 sudoers 安全策略

当你规划一个*sudoers*策略时，少即是多。你希望一个用户或一组用户在主机上拥有恰到好处的权限。如果有一个用户能够在管理公司网站时运行许多特权命令，那么如果这个用户被攻击者利用，你将面临严重的问题。这是因为任何攻击者都会继承被攻破用户的相同访问权限。

话虽如此，认为你可以完全锁定主机并仍然能够完成工作是天真的。试想一个软件交付工作流，其中应用程序在每次部署后需要重新启动。如果没有适当的用户权限，你将无法为该应用程序实现持续交付自动化。

在本章中，你将设置的示例安全策略是，*developers*组中的每个人都能够访问示例 Web 应用程序。他们还将能够停止、启动和编辑主应用程序文件。

## 安装问候 Web 应用程序

我提供的示例 Python Web 应用程序巧妙地（也是懒惰地）命名为*Greeting*。这个简单的 Web 应用程序在你访问虚拟机上的*http://localhost:5000*时，会热情地回应“Greetings！”我提供这个应用程序是为了让你集中精力学习自动化和配置；在这里我不会讲解它的代码。

你将使用 Ansible 任务安装运行 Web 应用程序所需的库和文件。你还将安装一个*systemd*单元文件，它是管理 Linux 主机上进程和服务的标准服务管理器，便于启动和停止 Web 应用程序。

安装 Web 应用程序的 Ansible 任务（以及本章的所有其他任务）位于*ansible/chapter4/*目录中。你应该导航到该目录并在你喜欢的编辑器中打开名为*web_application.yml*的任务文件。

这个文件包含四个独立的任务，名称如下：

1.  安装`python3-flask`、`gunicorn3`和`nginx`

1.  复制 Flask 示例应用程序

1.  复制*Systemd*单元文件以启动问候应用程序

1.  启动并启用问候应用程序

我将逐个讲解这些任务，首先从安装 Web 应用程序依赖项的任务开始：`python3-flask`、`gunicorn3`和`nginx`。这是文件顶部的第一个任务，应该如下所示：

```
- name: Install python3-flask, gunicorn3, and nginx
  apt:
    name:
      - python3-flask
      - gunicorn3
      - nginx
    update_cache: yes
```

任务`name`描述了它的意图，即`安装`一些软件包。`apt`模块再次被用来从 Ubuntu 存储库中在虚拟机上安装`python3-flask`、`gunicorn3`和`nginx`包。然而这次，`apt`模块使用了一些语法糖：YAML 列表。这一特性允许你在一个任务中安装多个软件包（或卸载它们），而不需要为每个要安装的软件包创建单独的任务。

从顶部开始的第二个任务将示例问候应用程序复制到虚拟机上。你需要两个文件来让问候 Web 应用程序正常运行，任务应该如下所示：

```
- name: Copy Flask Sample Application
  copy:
    src: "../ansible/chapter4/{{ item }}"
    dest: "/opt/engineering/{{ item }}"
  group: developers
  mode: '0750'
  loop:
    - greeting.py
    - wsgi.py
```

`copy` 模块将两个文件从提供的仓库复制到虚拟机（VM）。`src` 和 `dest` 行是模板化的（使用双大括号），并由 `loop` 模块的值替换。在这里，`loop` 模块通过名称引用了两个文件：*greeting.py* 和 *wsgi.py*。*greeting.py* 文件是实际的 Python Flask 代码，而 *wsgi.py* 文件包含了 HTTP 服务器的应用程序对象。在此任务的运行时，`{{ item }}` 占位符会被 `loop` 中的这两个文件名之一替换。例如，`src` 行在 `loop` 第一次遍历时会变成 `"../ansible/chapter4/greeting.py"`。`mode` 行将两个文件的权限设置为任何 *developers* 组的成员都可以读取和执行。

接下来，让我们看看复制 *systemd* 单元文件到虚拟机的任务。此任务位于从顶部数起的第三个位置，应如下所示：

```
- name: Copy Systemd Unit file for Greeting
  copy:
    src: "../ansible/chapter4/greeting.service"
    dest: "/etc/systemd/system/greeting.service"
```

该任务首先使用描述性的 `name`，如往常一样。然后，熟悉的 Ansible `copy` 模块将一个文件从本地主机复制到虚拟机。在这种情况下，它将 *greeting.service* 文件复制到虚拟机上 `systemd` 可以找到的位置：*/etc/systemd/system*。

让我们回顾一下 *system service* 文件。这类文件可以有很多选项和设置，但在本例中，我提供了一个简单的文件来控制 Greeting Web 应用程序的生命周期。

在编辑器中打开 *ansible/chapter4/greeting.service* 文件。它应如下所示：

```
[Unit]
Description=The Highly Complicated Greeting Application
After=network.target

[Service]
Group=developers
**WorkingDirectory=/opt/engineering** 
**ExecStart=/usr/bin/gunicorn3 --bind 0.0.0.0:5000 --access-logfile - --error-logfile - wsgi:app** 
ExecReload=/bin/kill -s HUP $MAINPID
KillMode=mixed

[Install]
WantedBy=multi-user.target
```

`WorkingDirectory` 和 `ExecStart` 行是此文件中最重要的部分。第一行将工作目录设置为 */opt/engineering*，因为这是应用程序代码所在的目录。在 `ExecStart` 行中，`gunicorn3` 应用程序调用 *wsgi.py* 文件来启动 Web 应用程序。你还会告诉 `gunicorn3` 将 STDOUT（`--access-logfile -`）和 STDERR（`--error-logfile -`）日志记录到 *systemd* 日志中，默认情况下，这些日志会转发到 */var/log/syslog* 文件。现在关闭 *greeting.service* 文件。

*web_application.yml* 文件中的最后一个任务确保 Greeting Web 应用程序已启动，并且每次执行配置时都会重新加载 `systemd` 守护进程。它应如下所示：

```
- name: Start and enable Greeting Application
  systemd:
    name: greeting.service
    daemon_reload: yes
    state: started
    enabled: yes
```

在这里，`systemd` Ansible 模块启动 Greeting Web 应用程序。该模块要求你设置 `name` 和 `state`，在此情况下分别为 `greeting.service` 和 `started`。`enabled` 参数告诉 `systemd` 在启动时自动启动该服务。使用 `daemon_reload` 参数还强制 `systemd` 重新加载所有服务文件，并在执行其他操作之前发现 *greeting.service* 文件。这相当于运行 `systemctl daemon-reload`。`daemon_reload` 参数在主机的首次配置时非常有用，以确保 `systemd` 知道该服务。务必使用 `daemon_reload` 参数，以确保 `systemd` 始终知道服务文件的任何更改。

## sudoers 文件的结构

*sudoers* 文件是配置安全策略（针对用户和组），用于调用 `sudo` 命令的地方。此类安全文件由名为 `Defaults`、`User Specifications` 和 `Aliases` 的部分组成。*sudoers* 文件是从上到下读取的，规则按此顺序应用，因此最后匹配的规则总是会生效。

`Defaults` 语法允许你在运行时覆盖一些 *sudoers* 选项，例如设置用户在运行 `sudo` 时可以访问的环境变量。`User Specifications` 部分决定了用户可以运行哪些命令，以及可以在哪些主机上运行这些命令。例如，你可以授予 *bender* 用户在所有 Web 服务器主机上运行 `apt install` 命令的权限。`Aliases` 语法引用文件中的其他对象，这对于在有很多重复内容时保持配置清晰简洁非常有用。

你可以混合使用的四个别名如下：

1.  `Host_Alias` 指定一个主机或一组主机

1.  `Runas_Alias` 指定一个命令可以以哪些用户或组的身份运行

1.  `Cmnd_Alias` 指定一个或多个命令

1.  `User_Alias` 指定一个用户或一组用户

在本例中，你只会在 *sudoers* 文件中使用 `Cmnd_Alias` 和 `Host_Alias`。

### 创建 *sudoers* 文件

要创建 *sudoers* 文件，你将使用 Ansible 的 `template` 模块和一个模板文件。Ansible 的 `template` 模块对于创建需要用变量修改的文件非常有用。`template` 模块使用 Python 模板的 Jinja2 模板引擎来创建文件。你将把模板文件保存在一个名为 *ansible/templates/* 的独立目录中（稍后会详细说明）。

在 *ansible/chapter4/* 目录下，使用你喜欢的编辑器打开名为 *sudoers.yml* 的任务文件。你首先应该注意到的是，在文件顶部有一个新的 Ansible 模块，叫做 `set_fact`。这个模块允许你设置主机变量，这些变量可以在任务或整个剧本中使用。在这里，你将使用它来设置一个变量，并在模板文件中使用：

```
- set_fact:
    greeting_application_file: "/opt/engineering/greeting.py"
```

这会创建一个名为 `greeting_application_file` 的变量，并将其值设置为 */opt/engineering/greeting.py*（之前的任务会安装这个 Web 应用）。如前所述，*developers* 组中的任何人都可以在 */opt/engineering/* 目录下读取和执行文件。

接下来，找到位于 `set_fact` 模块下面的任务。这个任务为 *developers* 组创建 *sudoers* 文件，应该如下所示：

```
- name: Create sudoers file for the developers group
  template:
    src: "../ansible/templates/developers.j2"
    dest: "/etc/sudoers.d/developers"
    validate: 'visudo -cf %s'
    owner: root
    group: root
    mode: 0440
```

Ansible 的`template`模块构建了你的*sudoers*文件。它需要一个源文件（`src`）和一个目标文件（`dest`）。源文件是你本地的 Jinja2 模板（*developers.j2*），目标文件将在 VM 上创建*developers sudoers*文件。`template`模块还包含一个`validate`步骤，用于验证模板是否正确。在这种情况下，`visudo`命令以安全的方式编辑并验证你的*sudoers*文件。给`visudo`命令加上`-cf`标志可以确保*sudoers*文件合规并且没有语法错误。`%s`是`dest`参数中文件的占位符。如果`validate`命令因任何原因失败，Ansible 任务也会失败。最后，将文件的所有者、组和权限设置为`root`、`root`和`0440`（分别）。这是*sudoers*期望的正确权限。

### sudoers 模板

Ansible 的`template`模块任务引用了位于*ansible/templates/*目录中的源 Jinja2 模板文件。它包含了为*developers*组创建*sudoers*策略的基本构建块。

导航到*ansible/templates/*目录，并在编辑器中打开*developers.j2*文件。文件的*.j2*后缀告诉 Ansible 这是一个 Jinja2 模板。文件的内容应如下所示：

```
# Command alias
Cmnd_Alias	START_GREETING    = /bin/systemctl start greeting , \
				    /bin/systemctl start greeting.service
Cmnd_Alias	STOP_GREETING     = /bin/systemctl stop greeting , \
				    /bin/systemctl stop greeting.service
Cmnd_Alias	RESTART_GREETING  = /bin/systemctl restart greeting , \
				    /bin/systemctl restart greeting.service

# Host Alias
Host_Alias  LOCAL_VM = {{ hostvars[inventory_hostname]['ansible_default_ipv4']['address'] }}
# User specification
%developers LOCAL_VM = (root) NOPASSWD: START_GREETING, STOP_GREETING, \
	    	       RESTART_GREETING, \
		       sudoedit {{ greeting_application_file }} 
```

该文件以三个`Cmnd_Alias`声明开始，这些声明用于停止、启动和重启 Greeting Web 应用程序。（在`systemd`中，服务可以被称为`greeting`或`greeting.service`，所以这两种情况都会被处理。）接下来，设置一个名为`LOCAL_VM`的`Host_Alias`，它指向 VM 的私有 IP 地址。内建的 Ansible 变量`hostvars`在配置运行时动态获取 VM 的 IP 地址。如果你同时配置多个主机，这将非常有用。最后，这会为*developers*组创建一个用户规范。（`%`表示这是一个组，而不是一个用户。）用户规范规则声明，在`LOCAL_VM`上的任何*developers*组成员，都可以作为*root*用户，无需密码启动、停止、重启或编辑 Greeting Web 应用程序。请注意，发出`sudoedit`命令仅允许编辑 Web 应用程序。（稍后我会更详细地讨论`sudoedit`。）`{{ greeting_application_file }}`变量将在运行时设置，指向通过`set_fact`设置的 Greeting Web 应用程序文件。

到此为止，可以安全地关闭所有打开的文件。接下来，你将配置 VM 并测试*bender*的`sudo`权限。

## 配置 VM

要运行本章的所有任务，你需要像在前几章中一样取消注释它们。在编辑器中打开*ansible/site.yml*文件，并找到安装 Web 应用程序的任务。它应该像这样：

```
**#-** **import_tasks****: chapter4/****web_application.yml**
```

删除`#`符号以取消注释。

接下来，找到创建*developers sudoer*策略的任务：

```
**#- import_tasks: chapter4/sudoers.yml**
```

通过删除`#`符号来取消注释该行。

现在，剧本应该看起来像这样：

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
    - import_tasks: chapter3/authorized_keys.yml
    - import_tasks: chapter3/two_factor.yml
 **-** **import_tasks****: chapter4/****web_application.yml**
 **-** **import_tasks****: chapter4/****sudoers.yml**
  `--snip--`
  handlers:
    - import_tasks: handlers/restart_ssh.yml
```

第四章的 playbook 更改已经添加到第三章的更改中

现在，你将使用 Vagrant 运行 Ansible 任务。返回到 *vagrant/* 目录，其中包含你的 *Vagrant* 文件，并输入以下命令来配置虚拟机：

```
$ **vagrant** **provision**
`--snip--`
PLAY RECAP *********************************************************************
default       : ok=21  changed=6   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

配置输出中的值会有所不同，具体取决于你运行 `provision` 命令的次数，因为 Ansible 会确保你的环境一致，如果不需要，它不会做多余的工作。这里的总任务数量已增加到 `21`。你还在虚拟机上更改了以下六个内容：

+   第四章的五个新任务

+   一个任务更新了第二章中的空文件的时间戳

再次确认在继续之前没有操作失败。

## 测试权限

在虚拟机成功配置后，你现在可以通过测试 *bender* 的命令访问来检查你的安全策略。首先，你需要重新以 *bender* 用户登录虚拟机。*sudoers* 策略应该允许 *developers* 组中的任何人（在这个案例中是 *bender*）启动、停止、重启或编辑网络应用程序。

要以 *bender* 身份登录，请获取另一个 2FA 令牌。这次，找到 *ansible/chapter3/google_authenticator* 文件顶部的第二个 2FA 令牌；它应该是 `68385555`。拿到它后，在终端输入以下命令以 *bender* 用户身份登录：

```
$ **ssh** **-i ~/.ssh/dftd** **-p 2222 bender@localhost**
Enter passphrase for key '/Users/bradleyd/.ssh/dftd: `<passphrase>`
Verification code: `<68385555>`
`--snip--`
bender@dftd:~$
```

这里，你使用的是第三章中的 SSH 参数来登录虚拟机。当系统提示输入 2FA 令牌时，使用刚刚获取的第二个令牌。这个登录过程现在应该很熟悉了，如果不熟悉，请回顾第三章以获取更多信息。

### 访问网络应用程序

你需要确保网络应用程序正在运行并响应请求。你将使用 `curl` 命令来测试，它将数据传输到服务器（在此情况下是 HTTP 服务器）。Greeting 应用程序服务器在所有接口上监听 5000 端口的请求。所以，在终端输入以下命令，向 Greeting 服务器的 5000 端口发送 `HTTP GET` 请求：

```
bender@dftd:~$ **curl http://localhost:5000**
<h1 style='color:green'>Greetings!</h1>
```

输出显示，Greeting 网络应用程序在虚拟机的 `localhost` 上成功响应请求。

### 编辑 greeting.py 来测试 sudoers 策略

接下来，你将通过 `sudoedit` 对 Greeting 应用程序进行小幅修改，以测试 *bender* 的权限。你在本章早些时候设置的 *sudoers* 策略允许 *developers* 组的成员使用 `sudoedit` 命令编辑 */opt/engineering/greeting.py* 文件，`sudoedit` 让用户可以用任何编辑器编辑文件，并且在编辑前会复制一份文件，以防出错。如果没有 `sudoedit`，你可能需要为每个用户想使用的编辑器创建多个命令别名。

在真实的生产系统中，你可能不会直接在主机上编辑文件。相反，你会编辑源控版本，并允许你的自动化更新它，确保使用最新版本。然而，我之所以描述这种方法，是为了展示如何测试你的 *sudoers* 策略。

在仍然以 *bender* 身份登录的情况下，输入以下命令以编辑 *greeting.py* 文件：

```
bender@dftd:~$ **sudoedit /opt/engineering/greeting.py**
```

该命令应该将你带入 Nano 文本编辑器（Ubuntu 的默认编辑器）。进入编辑器后，在 `hello()` 函数内找到类似下面的代码行：

```
return "<h1 style='color:green'>Greetings!</h1>"
```

将 `<h1>Greetings!</h1>` 文本修改为 `<h1>Greetings and Salutations!</h1>`，使该行如下所示：

```
return "<h1 style='color:green'>**Greetings and Salutations!**</h1>"
```

保存文件并退出 Nano 文本编辑器。

### 使用 systemctl 停止和启动

为了使问候语字符串更改生效，你需要使用 `sudo` 和 `systemctl` 命令（后者是一个命令行应用程序，允许你控制由 `systemd` 管理的服务）停止并启动 Web 应用程序服务器。你在 *sudoers* 策略中的 `Cmnd_Alias` 声明允许任何属于 *developers* 组的用户执行 `/bin/systemctl stop greeting` 或 `/bin/systemctl start greeting` 命令。

要使用 `systemctl` 停止正在运行的 Greeting 应用程序，请输入以下命令：

```
bender@dftd:-$ **sudo systemctl stop greeting**
```

命令应没有输出，并且不应该提示输入密码。

接下来，重新运行 `curl` 命令，以确保 Web 应用程序已停止：

```
bender@dftd:~$ **curl http://localhost:5000**
curl: (7) Failed to connect to localhost port 5000: Connection refused
```

在这里，`curl` 返回了一个 `Connection refused` 错误，因为服务器不再运行。

通过输入以下命令重新启动已停止的 Greeting 服务器：

```
bender@dftd:-$ **sudo systemctl start greeting**
```

如果命令成功执行，将不会有任何输出。

重新运行 `curl` 命令，检查 Web 应用程序是否正在运行并且代码已更新：

```
bender@dftd:~$ **curl http://localhost:5000**
<h1 style='color:green'>Greetings and Salutations!</h1>
```

Greeting 服务器成功响应了新的改进版问候语。如果由于某种原因，你的 Greeting 应用程序没有像这样响应，请回溯你的步骤。从检查虚拟机上的 */var/log/syslog* 文件或 */var/log/auth.log* 文件中的错误开始。

## 审计日志

如前所述，`sudo` 的一个重要特点是它会留下审计日志。这些日志中的事件通常用于监控框架，或在事件响应过程中进行取证。不管怎样，你应该确保审计数据存放在一个可访问的区域，以便你进行查看。

如果你按照本章中的测试进行操作，你会执行了三次 `sudo` 命令。那些事件被记录在 */var/log/auth.log* 文件中，因此让我们查看一些与这些 `sudo` 命令相关的日志行。我已经挑选出了一些与本示例相关的日志行，以免你被日志解析的艺术所困扰。然而，你可以随时深入探索日志文件。

在 *auth.log* 文件中，你将看到的第一行日志是关于 *bender* 使用 `sudoedit` 的：

```
Jul 23 23:17:43 ubuntu-focal sudo:   bender : TTY=pts/0 ; PWD=/home/bender ; USER=root ; COMMAND=sudoedit /opt/engineering/greeting.py
```

这行日志提供了相当多的信息，但我们将重点关注 `date/time`、`USER` 和 `COMMAND` 列。你可以看到 *bender* 在 `7 月 23 日` 的 `23:17:43` 调用了 `sudo`，执行了 `sudoedit /opt/engineering/greeting.py` 命令。这是在你修改 *greeting.py* 文件以更改问候文本时发生的。

这行日志显示了你使用 *bender* 停止 Greeting 服务器的操作：

```
Jul 23 23:18:19 ubuntu-focal sudo:   bender : TTY=pts/0 ; PWD=/home/bender ; USER=root ; COMMAND=/usr/bin/systemctl stop greeting
```

在`7 月 23 日`的`23:18:19`，*bender*以*root*用户身份使用`sudo`执行了`/bin/systemctl stop greeting`命令。

最后，这里是日志行，显示了*bender*启动了 Greeting 应用程序：

```
Jul 23 23:18:39 ubuntu-focal sudo:   bender : TTY=pts/0 ; PWD=/home/bender ; USER=root ; COMMAND=/usr/bin/systemctl start greeting
```

在`7 月 23 日`的`23:18:39`，*bender*以*root*用户身份使用`sudo`执行了`/bin/systemctl start greeting`命令。

到目前为止，我已经展示了成功且预期的日志条目。以下一行展示了*bender*执行失败的命令：

```
Jul 23 23:25:14 ubuntu-focal sudo:   bender : command not allowed ; TTY=pts/0 ; PWD=/home/bender ; USER=root ; COMMAND=/usr/bin/tail /var/log/auth.log
```

在`7 月 23 日`的`23:25:14`，*bender*尝试运行`/usr/bin/tail /var/log/auth.log`命令，但被拒绝。这些可能是你希望在警报系统中追踪的日志行，因为这可能是一个恶意行为者试图在主机上进行导航。

## 摘要

本章探讨了允许用户以提升的权限运行命令的重要性。使用 Ansible、`sudo`命令和*sudoers*文件，你可以限制命令访问并记录审计日志以确保安全。你还使用了不同的 Ansible 模块，如`template`、`systemd`和`set_fact`，这些模块使你能够自动化安装 Web 应用程序并控制其生命周期。

在下一章中，你将总结这一节关于配置和安全性的内容。你还将使用一些提供的 Ansible 任务来保护网络并为虚拟机实现防火墙。
