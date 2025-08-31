# 第五章：自动化和测试基于主机的防火墙

![](img/chapterart.png)

对于一台生产服务器，特别是暴露在互联网上的服务器，不过滤其网络流量是很危险的。作为软件或 DevOps 工程师，我们会为像 SSH 或 Web 服务器这样的服务开放端口，这是一种必要的、被接受的风险。然而，这并不意味着我们可以忽略所有其他目的地为我们主机的流量。为了最小化风险，我们需要过滤所有其他流量，并在允许进出的流量上做出务实的决策。因此，我们使用*防火墙*来监控网络或主机上的进出数据包。防火墙有两种类型。*网络防火墙* 通常是一个设备，所有流量都通过它从一个网络流向另一个网络，而 *基于主机的* *防火墙* 控制进出单一主机的数据包。

在本章中，你将专注于基于主机的防火墙。你将学习如何使用 Ansible、一些提供的任务和一个叫做 Uncomplicated Firewall (UFW) 的软件应用来自动化配置基于主机的防火墙。这个防火墙将阻止所有传入流量，除了 SSH 连接和你在第四章中安装的 Greeting Web 应用。到本章结束时，你将理解如何自动化基本的基于主机的防火墙，并能够审计来自防火墙的日志事件。

## 规划防火墙规则

防火墙规则需要非常明确地指定允许哪些流量，拒绝哪些流量。如果你不小心屏蔽了一个端口（或更糟，留下了一个暴露的端口），结果将会非常不理想。

你可以将防火墙的流量分为三个默认部分，称为*链*。可以将链看作是一个门，数据包必须通过这个门。每个门当正确路由的数据包到达时，都会引导到特定的地方。以下是 UFW 中你可以访问的三个默认链的简要描述：

1.  输入链 过滤目标主机的数据包

1.  输出链 过滤源自主机的数据包

1.  转发链 过滤正在通过主机路由的数据包

你将创建的防火墙规则只会应用于输入链，因为你关注的是传入 VM 的流量。转发链和输出链超出了本书的范围，因为你正在构建一个简单的基于主机的防火墙。如果你需要屏蔽出去的端口并转发网络流量，请访问[`ubuntu.com/server/docs/security-firewall/`](https://ubuntu.com/server/docs/security-firewall/)获取更多详情。

你将实现的防火墙规则将允许两种已知端口的传入流量，同时拒绝所有其他流量。你需要为 shell 访问（SSH）和 Ansible 配置开放端口 `22`；此外，还需要为 Web 应用开放端口 `5000`。你还将对端口 `5000` 添加速率限制，以保护 Web 服务器和主机免受过度滥用。最后，你将启用防火墙日志，以便审计通过防火墙的网络流量。

## 自动化 UFW 规则

*简单防火墙 (UFW)* 是一个软件应用程序，它为 iptables 框架提供了一个简化的封装，iptables 是基于内核的 Unix 操作系统包过滤的根本所在。具体来说，iptables、Netfilter、连接跟踪和网络地址转换 (NAT) 共同构成了包过滤框架。UFW 隐藏了使用 iptables 时的复杂性。结合 Ansible，它使得基于主机的防火墙设置变得简单、易于操作并且可重复。因此，您将使用 Ansible 任务来通过 UFW 创建规则。

配置防火墙的 Ansible 任务位于 *ansible/chapter5/* 目录下。只有在配置虚拟机后，这些规则才会生效，因此在配置之前让我们先回顾一下这些规则。请导航到 *ansible/chapter5/* 目录并用您喜欢的编辑器打开名为 *firewall.yml* 的任务文件。该文件包含以下五个任务：

1.  将 `Logging` 级别设置为低。

1.  允许通过 `22` 端口的 SSH 连接。

1.  允许所有访问 `5000` 端口。

1.  限制对 `5000` 端口的过度滥用。

1.  丢弃所有其他流量。

文件顶部的第一个任务应该如下所示：

```
- name: Turn Logging level to low
  ufw:
    logging: 'low'
```

这个任务启用 `logging` 功能来记录 UFW 日志，并将日志级别设置为 `low`。Ansible 的 `ufw` 模块用于在虚拟机上创建防火墙规则和策略。您可以将 `logging` 参数设置为 `off`、`low`、`medium`、`high` 或 `full`。`low` 日志级别将记录任何未匹配默认策略的被阻止数据包，以及您添加的任何其他防火墙规则。`medium` 级别会做 `low` 级别做的所有事情，并额外记录所有未匹配默认策略的允许数据包和所有新连接。`high` 日志级别会做 `medium` 级别做的所有事情，但它还会记录所有数据包，并对日志消息进行一定的速率限制。如果您的磁盘空间充足，并且想知道关于主机上的每个数据包的所有信息，可以将日志级别设置为 `high`。任何高于 `medium` 的设置都会生成大量日志数据，并可能在繁忙的主机上快速填满磁盘，因此请小心使用这些日志设置。

接下来，让我们看一下文件顶部的第二个任务，它为 SSH 连接打开 `22` 端口。它应该如下所示：

```
- name: Allow SSH over port 22
  ufw:
    rule: allow
    port: '22'
    proto: tcp
```

在这里，Ansible 的 `ufw` 模块创建了一个 `rule`，它允许来自任何源 IP 地址的连接，使用 TCP 协议连接到虚拟机的 `22` 端口。根据您的使用场景，您可以将 `rule` 参数设置为 `deny`、`limit` 或 `reject`。例如，如果您想停止某个端口的连接，但不介意向远程主机发送拒绝回复，您应该选择 `reject`。拒绝回复将告诉远程系统您已经启用但不接受该端口上的流量。另一方面，如果您想直接丢弃传入的数据包而不向远程主机发送任何回复，则选择 `deny` 规则。这会使得扫描您的主机的人更难知道主机是否在运行。（稍后我会详细讨论 `limit` 规则。）

下一个任务是允许通过端口`5000`进行远程连接到 Greeting Web 应用程序的规则。它应该像这样：

```
- name: Allow all access to port 5000
  ufw:
    rule: allow
    port: '5000'
    proto: tcp
```

这个`rule`与之前的任务行为相同，只不过它允许通过 TCP 连接端口`5000`，而不是端口`22`。

文件中的第四个任务限制了在特定时间范围内对端口`5000`（Greeting 服务器）的连接数量。这对于你想要自动阻止某人滥用服务非常有用，无论该用户是合法的还是可疑的。它应该像这样：

```
- name: Rate limit excessive abuse on port 5000
  ufw:
    rule: limit
    port: '5000'
    proto: tcp
```

UFW 的默认速率限制功能规定，如果某个源在 30 秒内尝试进行超过六次连接，则会拒绝该连接。如果你托管像 API 或 Web 服务器这样的公共服务，这个功能非常有用。你可以利用速率限制临时阻止用户频繁访问你的服务。另一个适用的场景是限制 SSH 上的暴力破解尝试，尤其是在*堡垒主机*上，堡垒主机是系统管理员用来远程访问私有网络的强化主机。然而，使用这个默认的限制设置时要小心，因为它可能对生产环境过于严格。允许远程系统在 30 秒内连接超过六次，可能对你来说是正常的流量。你将在本章稍后测试速率限制规则。

如果你想调整默认的速率限制设置，使用`lineinfile`模块（参见第三章）创建一个新任务，以定位并更新`/etc/ufw/user.rules`中的那一行，应该像这样：

```
-A ufw-user-input -p tcp --dport 5000 -m conntrack --ctstate NEW -m recent --update --seconds 30 --hitcount 6 -j ufw-user-limit 
```

根据你的环境需要，修改`hitcount`和`seconds`选项。

本文件中的最后一个任务会丢弃所有到目前为止未匹配任何其他规则的流量。记住，Ansible 是按顺序执行任务的。丢弃规则应该像这样：

```
- name: Drop all other traffic
  ufw:
    state: enabled
 policy: deny
    direction: incoming
```

请注意，这里没有`rule`参数。这个任务将 VM 上`ufw`服务的`state`设置为启用。它还将默认的`incoming`策略设置为`deny`，这强制你将所有需要暴露的服务列入白名单。如果有人不小心配置错误并打开了主机的端口，这也能保护你。

如前所述，Ansible 按顺序从上到下读取任务，而 UFW 规则也是按相同顺序读取的。如果`drop`规则是文件中的第一个任务，它将把策略设置为丢弃所有流量，并启用防火墙。那时`drop`规则会匹配所有传入的包并丢弃它们，停止对任何其他可能匹配的规则的搜索。这样，你不仅会失去对虚拟机的访问，还会丢失 Ansible 通过 SSH 建立的连接。这意味着配置过程会失败，可能会导致机器处于不正常的状态，因此在添加或删除规则时务必记住顺序。

## 配置虚拟机

要运行本章的所有任务，你需要在剧本中取消注释它们。这与之前章节的过程相同，应该已经熟悉了。打开*ansible/site.yml*文件，在编辑器中找到安装`firewall`的任务。它应该看起来像这样：

```
**#-** **import_tasks****: chapter5/****firewall.yml**
```

移除`#`符号以取消注释。此时，剧本应该如下所示：

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
    - import_tasks: chapter4/web_application.yml
    - import_tasks: chapter4/sudoers.yml
 **- import_tasks: chapter5/firewall.****yml**
  `--snip--`
  handlers:
 **- import_tasks: handlers/restart_ssh.yml**
```

第五章对剧本的更改是在第四章的基础上添加的。

现在，到了使用 Vagrant 运行 Ansible 任务的时候了。返回到包含*Vagrantfile*的*vagrant/*目录，并输入以下命令来配置虚拟机：

```
$ **vagrant provision**
`--snip--`
PLAY RECAP *********************************************************************
default       : ok=26  changed=6   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

总任务数量已增加到`26`，虚拟机上有`6`项更改：本章的五个新任务和一个更新第二章空文件时间戳的任务。继续之前，再次确认没有任务失败。

## 测试防火墙

接下来，你需要测试基于主机的防火墙是否已启用，允许两个白名单端口，阻止所有其他端口，并对 Greeting 应用程序进行速率限制。

首先，你需要能够从本地主机访问虚拟机，因此先从虚拟机获取一个 IP 地址。在*Vagrantfile*中，你告诉 Vagrant 创建另一个接口，并让 VirtualBox 通过 DHCP 从一个范围内分配地址。

如果你不再登录虚拟机，请重新以*bender*身份登录并获取另一个 2FA 令牌（如果需要）。这次，从*ansible/chapter3/google_authenticator*文件顶部获取第三个 2FA 令牌，它应该是`52973407`。获得后，在终端中输入以下命令以*bender*身份登录：

```
$ **ssh** **-i ~/.ssh/dftd** **-p 2222 bender@localhost**
Enter passphrase for key '/Users/bradleyd/.ssh/dftd: `<passphrase>`
Verification code: `<52973407>`
`--snip--`
bender@dftd:~$
```

然后，使用`ip`命令从你指示 Vagrant 和 VirtualBox 创建的接口中获取 IP 地址。此命令主要用于列出和操作 Linux 主机上的网络路由和设备。在虚拟机终端中，输入以下命令：

```
bender@dftd:~$ **ip -4 -br addr**
lo               UNKNOWN        127.0.0.1/8
enp0s3           UP             10.0.2.15/24
enp0s8           UP             172.28.128.3/24
```

上面的输出显示`ip`命令已成功完成。`-4`标志将输出限制为仅显示 IPv4 地址。`-br`标志仅打印基本的接口信息，如 IP 地址和名称，``addr 命令告诉`ip`显示网络接口的地址信息。``

````` The output lists three devices in tabular format. The first device, named `lo`, is a loopback network interface that is created on Linux hosts (commonly referred to as `localhost`). The loopback device is not routable (accessible) from outside the VM. The second device, `enp0s3`, has an IP address of `10.0.2.15`. This is the default interface and the IP you get from Vagrant and VirtualBox when you first create the VM. This device is also not routable from outside the VM. The last interface, `enp0s8`, has an IP address of `172.28.128.3`, which was dynamically assigned by this line in the *Vagrantfile*:    ``` config.vm.network "private_network", type: "dhcp" ```    This IP address is how you’ll access the VM from your local machine. Because these IP addresses are assigned using DHCP, yours may not match exactly. The interface name may be different as well; just use whatever IP address is listed for the interface that is not a `loopback` device or the device in the `10.0.2.0/24` subnet.    Keep this terminal and connection open to the VM, as you’ll use it again in the next section.    ### Scanning Ports with Nmap    To test that the firewall is filtering traffic, you’ll use the `nmap`(network mapper) command line tool for scanning hosts and networks. Be sure to install the appropriate Nmap version for your specific OS. Visit [`nmap.org/book/install.html`](https://nmap.org/book/install.html) for instructions on installing Nmap for different OSs.    Once it’s installed, you’ll want to do a couple of scans. The first scan, which is a fast check, tests that the firewall is enabled and allowing traffic on your two ports. The other scan is a check for the services and versions running behind those open ports.    To run the first scan, enter the following command in your terminal, using the IP address of the VM you copied earlier (if you are on a Mac or Linux host, you’ll need to use `sudo` since Nmap requires elevated permissions):    ``` $ **sudo nmap -F** `<172.28.128.3>` Password: Starting Nmap 7.80 ( https://nmap.org ) at 2022-08-11 10:14 MDT Nmap scan report for 172.28.128.3 Host is up (0.00066s latency). Not shown: 98 filtered ports PORT     STATE SERVICE 22/tcp   open  ssh 5000/tcp open  upnp MAC Address: 08:00:27:FB:C3:AF (Oracle VirtualBox virtual NIC) Nmap done: 1 IP address (1 host up) scanned in 1.88 seconds ```    The `-F` flag tells `nmap` to do a fast scan, which looks for only the 100 most common ports, such as `80` (web), `22` (SSH), and `53` (DNS). As expected, the output shows `nmap` detects that ports `22` and `5000` are open. It shows the other 98 ports are *filtered*, which means `nmap` could not detect what state the ports were in because of the firewall. This tells you that the host-based firewall is enabled and filtering traffic.    The next scan you’ll do is one that bad actors do on the internet every day. They scan for hosts that are connected to the internet, looking for services and versions while hoping they can match a vulnerability to it. Once they have an exploit in hand, they can use it to try to gain access to that host.    Enter the following command from your local host’s terminal to detect your service versions:    ``` $ **sudo nmap -sV** `<172.28.128.3>` Starting Nmap 7.80 ( https://nmap.org ) at 2022-08-11 21:06 MDT Nmap scan report for 172.28.128.3 Host is up (0.00029s latency). Not shown: 998 filtered ports PORT     STATE SERVICE VERSION 22/tcp   open  ssh    OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0) 5000/tcp open  http   Gunicorn 20.0.4 MAC Address: 08:00:27:F7:33:1F (Oracle VirtualBox virtual NIC) Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel ```   ```` ``` Service detection performed. Please report any incorrect results at https://nmap.org/submit/. Nmap done: 1 IP address (1 host up) scanned in 13.13 seconds ```    The `-sV` flag tells `nmap` to attempt to extract service and version information from running services. Once again, `nmap` finds the two open ports, `22` and `5000`. Also, a service name and version are listed next to each port. For port `22`, the service name is `OpenSSH`, and the version is `8.2p1` for `Ubuntu Linux`. For port `5000`, the service name is `Gunicorn`, and the version is `20.0.4.` If you were a bad actor armed with this information, you could search the many vulnerability databases, looking for exploits for these services and versions.    Next, you’ll want to check the logs for evidence that the firewall blocked connection attempts on non-whitelisted ports.    ### Firewall Logging    All events that the firewall processes can be logged. You enabled logging and set the level to `low` for UFW in the Ansible task earlier in this chapter. The log for those events is located in the */var/log/ufw.log*file. This logfile requires *root* permissions to read it, so you’ll need a user with elevated permissions.    As an example, I have pulled out a log entry to demonstrate a block event from the *ufw.log* file. Here is what UFW logged when Nmap tried to scan port `80`:    ``` Aug 11 16:56:17 ubuntu-focal kernel: [51534.320364] 1[UFW BLOCK] 2IN=enp0s8 OUT= MAC=08:00:27:fb:c3:af:0a:00:27:00:00:00:08:00 3SRC=172.28.128.1 4DST=172.28.128.3 LEN=44 TOS=0x00 PREC=0x00 TTL=48 ID=7129 PROTO=TCP SPT=33405 5DPT=80 WINDOW=1024 RES=0x00 SYN URGP=0 ```    This log line contains a lot of information, but you’ll focus on only a few components here. The event type name 1 is a block type, so it’s named `[UFW BLOCK]`. The `IN` key-value pair 2 shows the network interface for which this packet was destined. In this case, it’s the VM interface from the earlier section. The source IP address (`SRC`) 3 is where the packet originated. In this example, it’s the source IP address from the local host where you ran the `nmap` command. This IP address was created from VirtualBox when you added the other interface in Vagrant. The destination IP address, `DST` 4, is the IP address for which the packet was destined. It should be the IP address of the second non-loopback interface on the VM. The destination port, `DPT` 5, is the port where the packet was being sent. In this log line, it’s port `80`. Since you don’t have a rule permitting any traffic on port `80`, it was blocked. This means your firewall is blocking unwanted connection attempts. Remember, Nmap’s fast scan will try 100 different ports, so there will be multiple log lines that look like this one. However, they will have different destination ports (`DPT`).    ### Rate Limiting    To test that the firewall will rate-limit excessive connection attempts (six in 30 seconds) to your Greeting web server, you’ll leverage the `curl` command again. From your local host, enter the following to access the Greeting web server six times:    ``` $ **for i in `seq 1 6` ; do curl -w "\n" http://172.28.128.3:5000 ; done** <h1 style='color:green'>Greetings!</h1> <h1 style='color:green'>Greetings!</h1> <h1 style='color:green'>Greetings!</h1> <h1 style='color:green'>Greetings!</h1> <h1 style='color:green'>Greetings!</h1>  curl: (7) Failed to connect to 172.28.128.22 port 5000: Connection refused ```    Here, a simple `for` loop in Bash iterates and executes the `curl` command six times in succession. The `curl` command uses the `-w` `"\n"` flag to write out a new line after each loop, which makes the web server’s response output more readable. As you can see, the last line shows a `Connection refused` notification after the fifth successful connection to the Greeting web server. This is because the rate limit on the firewall for port `5000` was triggered by being hit six times in less than 30 seconds.    Let’s explore the log line for this event. (Once again, I’ve grabbed the relevant log line for you.)    ``` Aug 11 17:38:48 ubuntu-focal kernel: [54085.391114] 1 [UFW LIMIT BLOCK] IN=enp0s8 OUT= MAC=08:00:27:fb:c3:af:0a:00:27:00:00:00:08:00 2SRC=172.28.128.1 3DST=172.28.128.3 LEN=64 TOS=0x00 PREC=0x00 TTL=64 ID=0 DF PROTO=TCP SPT=58634 4DPT=5000 WINDOW=65535 RES=0x00 CWR ECE SYN URGP=0 ```    The UFW event type is named `[UFW LIMIT BLOCK]` 1. This packet is coming (`SRC`) from the local host IP address 2 where you ran the `curl` command. The destination (`DST`) 3 IP address is the one for the VM. The destination port (`DPT`) 4 is `5000`, which is the Greeting web server. This temporary limit will block your local host IP address (`172.28.128.1`)2 from accessing port `5000` for about 30 seconds after the limit is reached. After that, you should be able to access it again.    ## Summary    In this chapter, you’ve learned how to implement a simple but effective host-based firewall for the VM. You can easily apply this firewall to any host you have, whether it is local or from a cloud provider. Creating firewall rules with Ansible that permit specific traffic to a VM while blocking other traffic is a typical setup a DevOps or software engineer would use. You also learned how to limit the number of connections a host can make in a given time frame. All of these techniques provide a smaller attack surface to help deter network attacks. You can do a lot more to enhance your host-based firewall, and I encourage you to explore the possibilities on your own by visiting [`help.ubuntu.com/community/UFW/`](https://help.ubuntu.com/community/UFW/).    This brings Part I to an end. You now should have a good understanding of how to provision your infrastructure and apply some basic security foundations to your environment. In Part II, we’ll move on to containers, container orchestration, and deploying modern application stacks. We’ll start with installing and understanding Docker. ```` `````
