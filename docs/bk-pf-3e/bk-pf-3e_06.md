## 第六章：为主动防御翻盘

![为主动防御翻盘](img/httpatomoreillycomsourcenostarchimages2127149.png.jpg)

在上一章中，你了解了如何确保即使在严格的包过滤规则下，你希望提供的服务仍然可以使用，并且需要花费大量时间和精力来完成。现在，在你的工作设置完成后，你很快就会注意到，一些服务可能比其他服务更容易吸引不必要的注意。

这里是一个场景：你有一个网络，配备了与你站点需求相匹配的包过滤，其中包括一些需要向外部用户提供访问的服务。不幸的是，当这些服务可用时，就存在某些人会试图利用它们进行恶作剧的风险。

你几乎肯定会通过 SSH（安全外壳协议）进行远程登录，并且你的网络上会运行 SMTP 邮件服务——这两者都是诱人的攻击目标。在本章中，我们将探讨如何通过 SSH 增加未经授权访问的难度，然后转向一些更有效的方法来阻止垃圾邮件发送者使用你的服务器。

## 拒绝暴力破解者

安全外壳服务（通常称为 SSH）对于 Unix 系统管理员来说是一个非常重要的服务。它通常是与机器交互的主要接口，也是脚本小子攻击的常见目标。

### SSH 暴力破解攻击

如果你运行一个可以通过互联网访问的 SSH 登录服务，你可能在身份验证日志中看到了类似这样的条目：

```
Sep 26 03:12:34 skapet sshd[25771]: Failed password for root from 200.72.41.31 port 40992 ssh2
Sep 26 03:12:34 skapet sshd[5279]: Failed password for root from 200.72.41.31 port 40992 ssh2
Sep 26 03:12:35 skapet sshd[5279]: Received disconnect from 200.72.41.31: 11: Bye Bye
Sep 26 03:12:44 skapet sshd[29635]: Invalid user admin from 200.72.41.31
Sep 26 03:12:44 skapet sshd[24703]: input_userauth_request: invalid user admin
Sep 26 03:12:44 skapet sshd[24703]: Failed password for invalid user admin from 200.72.41.31
port 41484 ssh2
Sep 26 03:12:44 skapet sshd[29635]: Failed password for invalid user admin from 200.72.41.31
port 41484 ssh2
Sep 26 03:12:45 skapet sshd[24703]: Connection closed by 200.72.41.31
Sep 26 03:13:10 skapet sshd[11459]: Failed password for root from 200.72.41.31 port 43344 ssh2
Sep 26 03:13:10 skapet sshd[7635]: Failed password for root from 200.72.41.31 port 43344 ssh2
Sep 26 03:13:10 skapet sshd[11459]: Received disconnect from 200.72.41.31: 11: Bye Bye
Sep 26 03:13:15 skapet sshd[31357]: Invalid user admin from 200.72.41.31
Sep 26 03:13:15 skapet sshd[10543]: input_userauth_request: invalid user admin
Sep 26 03:13:15 skapet sshd[10543]: Failed password for invalid user admin from 200.72.41.31
port 43811 ssh2
Sep 26 03:13:15 skapet sshd[31357]: Failed password for invalid user admin from 200.72.41.31
port 43811 ssh2
Sep 26 03:13:15 skapet sshd[10543]: Received disconnect from 200.72.41.31: 11: Bye Bye
Sep 26 03:13:25 skapet sshd[6526]: Connection closed by 200.72.41.31
```

这就是一个*暴力破解攻击*的样子。某人或某物正试图通过暴力破解找到一个用户名和密码组合，从而让他们进入你的系统。

最简单的应对方式是编写一个*pf.conf*规则，阻止所有访问，但这会导致另一类问题，包括如何让合法用户访问系统。将你的`sshd`配置为只接受基于密钥的身份验证有帮助，但很可能无法阻止小白攻击者尝试。你可以考虑将服务移到另一个端口，但这又会带来问题，可能会导致那些在 22 端口向你发起攻击的攻击者通过扫描轻松找到 22222 端口，再次进行攻击。^([30])

从 OpenBSD 3.7（及其等效版本）开始，PF 提供了一个稍微更优雅的解决方案。

### 设置自适应防火墙

为了防止暴力破解攻击，你可以编写你的`pass`规则，限制连接主机的某些行为。为了更好地防范，你可以将违规者驱逐到一个地址表中，拒绝对这些地址的某些或全部访问。你甚至可以选择断开所有来自超出限制机器的现有连接。要启用此功能，首先通过在任何过滤规则之前将以下行添加到配置中来设置表：

```
table <bruteforce> persist
```

然后，在规则集的前面，`block`暴力破解者，如下所示：

```
block quick from <bruteforce>
```

最后，添加你的`pass`规则：

```
pass proto tcp to $localnet port $tcp_services \
     keep state (max-src-conn 100, max-src-conn-rate 15/5, \
         overload <bruteforce> flush global)
```

这个规则与你之前看到的例子非常相似。此处有趣的部分是括号内的内容，称为*状态跟踪选项*：

+   `max-src-conn`是允许来自单一主机的最大并发连接数。在这个例子中，它被设置为`100`。你可能需要根据网络的流量模式稍微调高或降低这个值。

+   `max-src-conn-rate`是允许来自任何单一主机的新连接速率。在此，它被设置为`15`个连接每 5 秒，表示为`15/5`。选择一个适合你配置的速率。

+   `overload <bruteforce>`意味着任何超出前述限制的主机地址将被加入到`bruteforce`表中。我们的规则集会阻止所有来自`bruteforce`表中地址的流量。一旦主机超过这些限制并被放入溢出表中，规则将不再匹配来自该主机的流量。确保溢出者被处理，即使只是通过默认的阻止规则或类似的方式。

+   `flush global`表示当主机达到限制时，它的所有连接状态将被终止（清除）。`global`选项意味着为了保险起见，`flush`适用于该主机所有由流量创建的状态，无论是哪条规则创建了该状态。

如你所想，这个小小的规则集修改会产生显著效果。在尝试几次后，暴力破解者会进入`bruteforce`表中。这意味着它们所有现有的连接会被终止（清除），任何新的尝试将被阻止，最可能在它们的端显示`Fatal: timeout before authentication`消息。你已经创建了一个*自适应防火墙*，它会根据网络中的条件自动调整并对不良活动作出反应。

### 注意

*这些自适应规则仅对防御传统的、快速的暴力破解攻击有效。2008 年首次被识别并一直以来不断重复的低强度、分布式密码猜测攻击（又名*The Hail Mary Cloud*^([31]))，不会产生符合这些规则的流量。*

你可能希望在规则集中有一定的灵活性，为某些服务允许更多的连接，但在 SSH 方面可能想要更加严格。在这种情况下，你可以在规则集的前面补充一个类似下面的通用`pass`规则：

```
pass quick proto { tcp, udp } to port ssh \
     keep state (max-src-conn 15, max-src-conn-rate 5/3, \
        overload <bruteforce> flush global)
```

你应该通过阅读相关的手册页和*PF 用户指南*（见附录 A），找到最适合你情况的参数集。

### 注意

*记住，这些示例规则仅作为说明，可能你的网络需求更适合其他规则。将同时连接数或连接速率设置得过低，可能会阻止合法流量。当配置中有许多主机在一个公共 NAT 网关后面，而 NAT 的主机上的用户有合法的业务需要穿越一个有严格`overload`规则的网关时，可能会造成自我引发的拒绝服务风险。*

状态跟踪选项和`overload`机制并不需要仅仅应用于 SSH 服务，而且并不总是希望阻止所有来自违规者的流量。例如，你可以使用如下规则：

```
pass proto { tcp, udp } to port $mail_services \
     keep state (max 1500, max-src-conn 100)
```

这里，`max`指定了每条规则可以创建的最大状态数，用于防止邮件或 Web 服务接收超过其处理能力的连接（请记住，加载的规则数取决于`$mail_services`宏的扩展内容）。一旦达到`max`限制，新的连接将不再匹配该规则，直到旧连接终止。或者，你可以移除`max`限制，给规则添加`overload`部分，并将违规者分配到一个带有最小带宽分配的队列中（有关设置队列的详细信息，请参见第七章中的流量整形讨论）。

一些站点使用`overload`实现多层次系统，其中触发一个`overload`规则的主机会被转移到一个或多个中间的“试用”表中，进行特别处理。在 Web 上下文中，不直接阻止来自`overload`表中的主机的流量，而是将这些主机的所有 HTTP 请求重定向到特定的网页，可能会很有用（就像第四章末尾的`authpf`示例一样）。

### 使用 pfctl 整理你的表

在上一节中设置了`overload`规则后，你现在拥有了一个自适应防火墙，可以自动检测不良行为并将违规者的 IP 地址添加到表中。观察日志和表格在短期内可能很有趣，但由于这些规则只会向表中添加内容，我们面临下一个挑战：保持表格内容的最新和相关性。

当你使用适应性规则集运行配置一段时间后，你会发现，上周由于暴力破解攻击而被`overload`规则阻止的某个 IP 地址，实际上是一个动态分配的地址，而这个地址现在被分配给了另一个 ISP 客户，该客户有合法的理由与你网络中的主机通信。^([32]) 如果你的适应性规则捕获了大量网络流量，你还可能发现，随着时间的推移，`overload`表将不断增长，占用越来越多的内存。

解决方案是 *过期* 表格条目——在一定时间后移除条目。在 OpenBSD 4.1 中，`pfctl` 获得了根据统计信息上次重置的时间来过期表格条目的能力。^([33])（在几乎所有情况下，这个重置时间等于表格条目被添加的时间。）关键词是 `expire`，并且表格条目的年龄以秒为单位指定。以下是一个示例：

```
$ **sudo pfctl -t bruteforce -T expire 86400**
```

此命令将移除 `bruteforce` 表格中，统计信息重置时间超过 86,400 秒（24 小时）以上的条目。

### 注

*选择 24 小时作为过期时间是一个相当随意的决定。你应该选择一个你认为合理的时间值，以便任何问题在另一端能够被发现并修复。如果你已经设定了自适应规则，建议设置 `crontab` 条目以定期运行表格过期命令，类似于前述的命令，以确保你的表格保持最新。*

## 给垃圾邮件发送者制造麻烦，使用 spamd

电子邮件是一个相当重要的服务，需要特别关注，因为它每天都会处理大量的垃圾邮件（*spam*）。当恶意软件开发者发现通过电子邮件传播蠕虫有效并开始使用电子邮件传播恶意载荷时，未经请求的商业邮件数量已经成为一个痛苦的问题。在 2000 年代初，垃圾邮件和通过电子邮件传播的恶意软件的数量已经增加到，若没有某种反垃圾邮件措施，运行一个 SMTP 邮件服务几乎变得不可想象。

反垃圾邮件措施几乎与垃圾邮件问题本身一样古老。早期的工作主要集中在分析邮件内容（称为 *内容过滤*），并在一定程度上对邮件的可伪造的头部进行解读，例如所谓的发件人地址（`From:`）或在 `Received:` 头部记录的中间传递的存储与转发路径。

当 OpenBSD 团队设计其反垃圾邮件解决方案 `spamd` 时，首次在 2003 年的 OpenBSD 3.3 中引入，开发者专注于网络层面以及 SMTP 会话中的直接通信伙伴，并结合任何关于尝试发送邮件的主机的可用信息。开发者的目标是创建一个小巧、简单且安全的程序。早期的实现几乎完全依赖于创意使用 PF 表格，并结合来自可信外部来源的数据。

### 注

*除了 OpenBSD 的垃圾邮件延迟守护进程，基于内容过滤的反垃圾邮件软件包 SpamAssassin* ([`spamassassin.apache.org/`](http://spamassassin.apache.org/)) *还包含一个名为 `spamd` 的程序。两个程序都是为了帮助抵抗垃圾邮件，但它们解决基础问题的方法完全不同，并且不会直接互操作。然而，当这两个程序都被正确配置并运行时，它们能够很好地互补。*

### 网络层行为分析与黑名单

原始的 `spamd` 设计基于以下观察：垃圾邮件发送者发送大量邮件，而你成为第一个接收到某一特定邮件的人的可能性极小。此外，垃圾邮件通过一些垃圾邮件友好的网络和大量被劫持的机器发送。无论是单个邮件还是发送它们的机器都会很快报告给黑名单维护者，而由已知垃圾邮件发送者的 IP 地址构成的黑名单数据是 `spamd` 处理的基础。

当处理黑名单中的主机时，`spamd` 使用一种叫做 *tarpitting* 的方法。当守护进程接收到一个 SMTP 连接时，它会展示其横幅并立即切换到一个模式，在该模式下，它以每秒 1 字节的速度响应 SMTP 流量，使用一小部分 SMTP 命令确保邮件永远不会被投递，而是在邮件头被传输后被拒绝并返回到发送方的队列中。目的是尽可能浪费发送端的时间，同时对接收方几乎没有任何成本。这个特定的 tarpitting 实现，即每秒 1 字节的 SMTP 回复，通常被称为 *stuttering*（颤抖）。基于黑名单的 tarpitting 和 stuttering 是 `spamd` 在 OpenBSD 4.0 及之前版本的默认模式。

### 注意

*在 FreeBSD 和 NetBSD 上，spamd 不是基础系统的一部分，但可以通过 ports 和 packages 以* mail/spamd* 的形式获取。如果你在 FreeBSD 或 NetBSD 上运行 PF，你需要先安装该 port 或 package，然后再按照接下来的页面中的指示进行操作。*

#### 在黑名单模式下设置 spamd

要将 `spamd` 设置为传统的仅黑名单模式，你首先需要在 *pf.conf* 中添加一个专用的表格和相应的重定向，然后再关注 `spamd` 自身的 *spamd.conf* 文件。之后，`spamd` 通过该表格和重定向接入 PF 规则集。

以下是此配置的 *pf.conf* 配置行：

```
table <spamd> persist
pass in on $ext_if inet proto tcp from <spamd> to \
      { $ext_if, $localnet } port smtp rdr-to 127.0.0.1 port 8025
```

以下是 OpenBSD 4.7 之前的语法：

```
table <spamd> persist
rdr pass on $ext_if inet proto tcp from <spamd> to \
         { $ext_if, $localnet } port smtp -> 127.0.0.1 port 8025
```

表格 `<spamd>` 用于存储你从可信的黑名单源导入的 IP 地址。重定向负责处理所有来自黑名单中已存在主机的 SMTP 尝试。`spamd` 监听端口 8025，并对它接收到的所有 SMTP 连接进行慢速响应（每秒 1 字节），这是重定向导致的。稍后，在规则集的其他部分，你将会有一个规则，确保合法的 SMTP 流量通过并传送到邮件服务器。*spamd.conf* 是你指定黑名单数据源以及任何例外或本地覆盖设置的地方。

### 注意

*在 OpenBSD 4.0 及之前版本（以及基于 OpenBSD 4.1 之前版本的 ports）中，* spamd.conf *位于* /etc*。从 OpenBSD 4.1 开始，* spamd.conf *位于* /etc/mail*。FreeBSD port 会在* /usr/local/etc/spamd/spamd.conf.sample* 中安装一个示例配置文件。*

在*spamd.conf*的开头附近，你会看到一行没有`#`注释符号，看起来像是`all:\`。这一行指定了你将使用的黑名单。以下是一个示例：

```
all:\
:uatraps:whitelist:
```

将你想使用的所有黑名单添加到`all:\`行下，每个黑名单之间用冒号（**`:`**）分隔。如果要使用白名单从黑名单中减去地址，则在每个黑名单名称后面立即添加白名单的名称，如`：blacklist:whitelist:`。

接下来是黑名单定义：

```
uatraps:\
        :black:\
        :msg="SPAM. Your address %A has sent spam within the last 24 hours":\
        :method=http:\
        :file=www.openbsd.org/spamd/traplist.gz
```

紧随名称（`uatraps`）之后，第一个数据字段指定了列表类型——在这种情况下是`black`。`msg`字段包含在 SMTP 对话过程中要显示给黑名单发件人的消息。`method`字段指定了`spamd-setup`如何获取列表数据——在这种情况下是通过 HTTP。其他可能的方式包括通过 FTP（`ftp`）、从挂载的文件系统中的文件（`file`），或通过执行外部程序（`exec`）获取。最后，`file`字段指定了`spamd`期望接收的文件名。

白名单的定义遵循相似的模式，但省略了消息参数：

```
whitelist:\
        :white:\
        :method=file:\
        :file=/etc/mail/whitelist.txt
```

### 注意

*当前默认的 spamd.conf 中的建议黑名单是积极维护的，几乎没有出现假阳性。然而，早期版本的该文件也建议使用了一些黑名单，这些黑名单排除了互联网上的大块区域，包括几个声称覆盖整个国家的地址范围。如果你的网站预计会与这些国家交换合法邮件，那么这些黑名单可能不适合你的配置。其他流行的黑名单已知会将整个`/16`地址范围列为垃圾邮件来源，因此，在将黑名单投入生产之前，查看该黑名单的维护政策非常值得。*

将`spamd`的启动行和你希望的启动参数放入 OpenBSD 中的*/etc/rc.conf.local*，或者在 FreeBSD 或 NetBSD 中放入*/etc/rc.conf*。以下是一个示例：

```
spamd_flags="-v -b" # for normal use: "" and see spamd-setup(8)
```

在这里，我们启用`spamd`并设置它以黑名单模式运行，使用`-b`标志。此外，`-v`标志启用详细日志记录，这对于调试`spamd`的活动非常有用。

在 FreeBSD 上，控制`spamd`行为的*/etc/rc.conf*设置包括`obspamd_enable`，它应该设置为`"YES"`以启用`spamd`，以及`obspamd_flags`，你可以在这里填入任何`spamd`的命令行选项：

```
obspamd_enable="YES"
obspamd_flags="-v -b" # for normal use: "" and see spamd-setup(8)
```

### 注意

*要使 spamd 在 OpenBSD 4.1 或更高版本中以纯黑名单模式运行，你可以通过将 spamd_black 变量设置为“YES”并重新启动 spamd 来实现相同的效果。*

完成配置编辑后，使用所需的选项启动`spamd`，并使用`spamd-setup`完成配置。最后，创建一个`cron`作业，定期调用`spamd-setup`以更新黑名单。在纯黑名单模式下，你可以使用`pfctl`表命令查看和操作表内容。

#### spamd 日志

默认情况下，`spamd`记录到您的一般系统日志中。要将`spamd`日志消息发送到单独的日志文件，请向*syslog.conf*添加类似以下条目：

```
!!spamd
daemon.err;daemon.warn;daemon.info;daemon.debug         /var/log/spamd
```

一旦您确信`spamd`正在运行并且正在执行其预期的操作，您可能希望将`spamd`日志文件添加到您的日志轮换中。在运行`spamd-setup`并填充表之后，您可以使用`pfctl`查看表内容。

### 注意

*在本节开头的* pf.conf *片段示例中，重定向*（rdr-to）*规则也是一个通过规则。如果您选择使用匹配规则（或者如果您使用较旧的 PF 版本并选择编写不包括通过部分的 rdr 规则），请确保设置一个通过规则以允许流量通过到您的重定向。您可能还需要设置规则以允许合法的电子邮件通过。但是，如果您已经在网络上运行电子邮件服务，您可能可以继续使用旧的 SMTP 通过规则。

给定一组可靠且维护良好的黑名单，`spamd`在纯黑名单模式下可以很好地减少垃圾邮件。然而，使用纯黑名单，您只能捕获那些已经尝试在其他地方投递垃圾邮件的主机的流量，并且您需要信任外部数据源来确定哪些主机应该被陷阱。对于提供对网络级行为更快速响应并在垃圾邮件预防方面提供一些真正收益的设置，请考虑*灰名单*，这是现代`spamd`工作的一个关键部分。

### 灰名单：我的管理员告诉我不要和陌生人说话

灰名单主要包括解释当前 SMTP 标准并添加一点善意谎言以使生活更轻松。

垃圾邮件发送者倾向于使用他人的设备发送他们的消息，他们未经合法所有者许可安装的软件需要相对轻量级才能在不被检测的情况下运行。与合法的邮件发送者不同，垃圾邮件发送者通常不认为他们发送的任何单个消息很重要。综合起来，这意味着典型的垃圾邮件和恶意软件发送者软件没有设置正确解释 SMTP 状态代码的功能。这是一个我们可以利用的事实，正如埃文·哈里斯在他 2003 年的论文《垃圾邮件控制战的下一步：灰名单》中提出的那样。

正如哈里斯所指出的，当被篡改的机器用于发送垃圾邮件时，发送应用程序往往只尝试一次投递，而不检查任何结果或返回代码。真正的 SMTP 实现会解释 SMTP 返回代码并根据其行动，如果初始尝试失败并出现任何临时错误，真正的邮件服务器会重试。

在他的论文中，哈里斯概述了一个实用的方法：

+   在与先前未知的通信伙伴的第一次 SMTP 联系中，*不要*在第一次投递尝试时接收电子邮件，而是用指示临时本地问题的状态代码回复，并存储发件人 IP 地址以供将来参考。

+   如果发送者立即重试，则如之前一样以临时失败状态代码回复。

+   如果发送者在设定的最小等待时间（例如 1 小时）后重试，但不超过最大等待时间（例如 4 小时），则接受消息并将发送者的 IP 地址记录在白名单中。

这就是灰名单的精髓。幸运的是，你可以在配备 PF 的网关上设置和维护一个灰名单 `spamd`。

#### 配置灰名单模式下的 spamd

OpenBSD 的 `spamd` 在 OpenBSD 3.5 中获得了灰名单功能。从 OpenBSD 4.1 开始，`spamd` 默认在灰名单模式下运行。

在默认的灰名单模式下，用于黑名单的 `spamd` 表（如前一部分所述）变得多余。你仍然可以使用黑名单，但 `spamd` 会使用私有数据结构和 `spamdb` 数据库的组合来存储与灰名单相关的数据。默认模式下的 `spamd` 规则集通常如下所示：

```
table <spamd-white> persist
table <nospamd> persist file "/etc/mail/nospamd"
pass in log on egress proto tcp to port smtp \
            rdr-to 127.0.0.1 port spamd
pass in log on egress proto tcp from <nospamd> to port smtp
pass in log on egress proto tcp from <spamd-white> to port smtp
pass out log on egress proto tcp to port smtp
```

这包括必要的 `pass` 规则，以便让合法的电子邮件从你自己的网络流向预定的目的地。`<spamd-white>` 表是由 `spamd` 维护的白名单。`<spamd-white>` 表中的主机已经通过灰名单验证，来自这些机器的邮件被允许通过，送到实际的邮件服务器或其内容过滤前端。此外，`nospamd` 表可以让你加载不希望暴露给 `spamd` 处理的主机地址，匹配的 `pass` 规则确保这些主机的 SMTP 流量通过。

在你的网络中，你可能希望收紧这些规则，仅允许通过 SMTP 从允许发送和接收电子邮件的主机传输 SMTP 流量。我们将在 处理不适合灰名单的站点 中再次讨论 `nospamd` 表。

以下是在 OpenBSD 4.7 之前语法的等效规则：

```
table <spamd-white> persist
table <nospamd> persist file "/etc/mail/nospamd"
rdr pass in log on egress proto tcp to port smtp \
            -> 127.0.0.1 port spamd
pass in log on egress proto tcp from <nospamd> to port smtp
pass in log on egress proto tcp from <spamd-white> to port smtp
pass out log on egress proto tcp to port smtp
```

在 FreeBSD 上，为了在灰名单模式下使用 `spamd`，你需要一个文件描述符文件系统（参见 `man 5 fdescfs`），并将其挂载到 */dev/fd/*。要实现这一点，将以下行添加到 */etc/fstab* 文件中，并确保 `fdescfs` 代码已包含在你的内核中，或者通过适当的 `kldload` 命令加载模块。

```
fdescfs /dev/fd fdescfs rw 0 0
```

要开始配置 `spamd`，将 `spamd` 和你希望的启动参数放入 */etc/rc.conf.local* 文件中。这里是一个示例：

```
spamd_flags="-v -G 2:4:864" # for normal use: "" and see spamd-setup(8)
```

在 FreeBSD 上，相应的行应该放在 */etc/rc.conf* 中：

```
obspamd_flags="-v -G 2:4:864" # for normal use: "" and see spamd-setup(8)
```

你可以通过 `spamd` 命令行参数，使用 `-G` 选项后面的参数来微调与灰名单相关的多个参数。

为什么灰名单有效

在设计和开发过程中，已投入大量工作使得一些关键服务（例如 SMTP 邮件传输）具有容错性。实际上，这意味着，像 SMTP 这样的服务能够尽最大努力实现接近完美的邮件投递记录。这就是为什么我们可以依赖灰名单来最终接收来自合法邮件服务器的邮件。

当前的互联网邮件传输标准定义在 RFC 5321 中。^([35]) 以下是第 4.5.4.1 节“发送策略”的几段摘录：

> “在一个典型的系统中，组成邮件的程序有某种方法来请求立即关注一封新的外发邮件，而那些无法立即传输的邮件必须被排队并由发件人定期重试……”
> 
> “发件人 *MUST* 在一次尝试失败后延迟重试某个特定的目标。通常，重试间隔 *SHOULD* 至少为 30 分钟；然而，当 SMTP 客户端能够确定未能传输的原因时，更复杂和多变的策略会更有益。”
> 
> “重试会继续，直到消息传输成功或发件人放弃；放弃时间通常需要至少 4 到 5 天。”

电子邮件的传递是一个协作性的最佳努力过程，RFC 明确指出，如果你试图发送邮件的站点报告它暂时无法接收邮件，那么你的责任（必需要求）是稍后再试，给接收服务器一个从问题中恢复的机会。

灰名单的巧妙之处在于它是一种方便的小谎言。当我们声称有一个临时的本地问题时，那个问题其实就是等同于“我的管理员告诉我不要与陌生人交谈。” 规范的发件人会再次尝试，但垃圾邮件发送者不会等机会重试，因为这样会增加他们发送邮件的成本。这就是为什么灰名单依然有效，而且由于它严格遵循已接受的标准，^([36]) 错误的假阳性很少见。

使用冒号分隔的列表 `2:4:864` 表示 `passtime`、`greyexp` 和 `whiteexp` 的值：

+   `passtime` 表示 `spamd` 考虑的合理重试前的最短时间，默认值为 25 分钟，但在这里我们将其调整为 2 分钟。

+   `greyexp` 是一个条目在灰名单状态下保留的小时数，直到它从数据库中移除。

+   `whiteexp` 决定了一个白名单条目被保留的时间。`greyexp` 和 `whiteexp` 的默认值分别为 4 小时和 864 小时（大约 1 个月）。

#### 实践中的灰名单

实施灰名单的站点的用户和管理员普遍认为，灰名单有效地清除了大多数垃圾邮件，并显著减少了它们邮件内容过滤系统的负载。我们将首先查看`spamd`的灰名单日志记录的样子，然后返回一些数据。

如果你使用`-v`命令行选项启动`spamd`以进行详细日志记录，那么日志将包含一些额外的信息，除了 IP 地址之外。启用详细日志记录后，典型的日志摘录如下所示：

```
Oct 2 19:53:21 delilah spamd[26905]: 65.210.185.131: connected (1/1), lists: spews1
Oct 2 19:55:04 delilah spamd[26905]: 83.23.213.115: connected (2/1)
Oct 2 19:55:05 delilah spamd[26905]: (GREY) 83.23.213.115: <gilbert@keyholes.net> ->
<wkitp98zpu.fsf@datadok.no>
Oct 2 19:55:05 delilah spamd[26905]: 83.23.213.115: disconnected after 0 seconds.
Oct 2 19:55:05 delilah spamd[26905]: 83.23.213.115: connected (2/1)
Oct 2 19:55:06 delilah spamd[26905]: (GREY) 83.23.213.115: <gilbert@keyholes.net> ->
<wkitp98zpu.fsf@datadok.no>
Oct 2 19:55:06 delilah spamd[26905]: 83.23.213.115: disconnected after 1 seconds.
Oct 2 19:57:07 delilah spamd[26905]: (BLACK) 65.210.185.131: <bounce-3C7E40A4B3@branch15.
summer-bargainz.com> -> <adm@dataped.no>
Oct 2 19:58:50 delilah spamd[26905]: 65.210.185.131: From: Auto lnsurance Savings <noreply@
branch15.summer-bargainz.com>
Oct 2 19:58:50 delilah spamd[26905]: 65.210.185.131: Subject: Start SAVlNG M0NEY on Auto
lnsurance
Oct 2 19:58:50 delilah spamd[26905]: 65.210.185.131: To: adm@dataped.no
Oct 2 20:00:05 delilah spamd[26905]: 65.210.185.131: disconnected after 404 seconds. lists:
spews1
Oct 2 20:03:48 delilah spamd[26905]: 222.240.6.118: connected (1/0)
Oct 2 20:03:48 delilah spamd[26905]: 222.240.6.118: disconnected after 0 seconds.
Oct 2 20:06:51 delilah spamd[26905]: 24.71.110.10: connected (1/1), lists: spews1
Oct 2 20:07:00 delilah spamd[26905]: 221.196.37.249: connected (2/1)
Oct 2 20:07:00 delilah spamd[26905]: 221.196.37.249: disconnected after 0 seconds.
Oct 2 20:07:12 delilah spamd[26905]: 24.71.110.10: disconnected after 21 seconds. lists:
spews1
```

第一行是来自`spews1`黑名单中机器的连接开始。接下来的六行显示了来自另一台机器的两次连接尝试的完整记录，每次它都作为第二个活跃连接进行连接。这台机器还没有被列入任何黑名单，因此被列为灰名单。请注意，灰名单机器尝试投递的邮件中有一个相当奇怪的收件地址（*wkitp98zpu.fsf@datadok.no*)。这里有一个有用的技巧，我们将在灰名单陷阱中讨论。地址前面的`(GREY)`和`(BLACK)`标记表示灰名单或黑名单状态。接着，我们看到来自黑名单主机的更多活动，稍后我们看到，在 404 秒（即 6 分钟 44 秒）后，黑名单主机放弃了投递。

其余的几行显示了一些非常短的连接，包括一台已经被列入黑名单的机器。不过这次，机器断开连接太快，无法在 SMTP 对话的开头看到任何`(BLACK)`标记，但我们在末尾看到了对列表名称（`spews1`）的引用。

粗略估计，大约 400 秒是天真被列入黑名单的垃圾邮件发送者停留的时间（根据来自不同网站的数据），也是完成`EHLO ...`对话直到`spamd`拒绝邮件所需的时间（按每秒 1 字节的速率）。然而，在查看日志时，你可能会发现一些垃圾邮件发送者停留的时间明显更长。例如，在我们办公室网关的数据中，有一个日志条目格外引人注目：

```
Dec 11 23:57:24 delilah spamd[32048]: 69.6.40.26: connected (1/1), lists:
spamhaus spews1 spews2
Dec 12 00:30:08 delilah spamd[32048]: 69.6.40.26: disconnected after 1964
seconds. lists: spamhaus spews1 spews2
```

这台特定的机器在 12 月 9 日至 12 月 12 日之间进行过 13 次投递尝试时，已经被列入了多个黑名单。最后一次尝试持续了 32 分钟 44 秒，仍未完成投递。相对聪明的垃圾邮件发送者通常会在前几秒钟内就断开连接，就像第一个日志片段中的那些一样。其他一些则在大约 400 秒后放弃。有些则坚持几个小时。（我们记录的最极端情况是坚持了 42,673 秒，接近 12 小时。）

### 跟踪你实际的邮件连接：spamlogd

在幕后，鲜为人知且几乎没有文档记录的，是`spamd`最重要的辅助程序之一：`spamlogd`白名单更新程序。顾名思义，`spamlogd`在后台安静地工作，记录进出你的邮件服务器的连接，以保持白名单的更新。其目的是确保在你与常联系的主机之间发送的有效邮件能够顺利通过，且不出现太多麻烦。

### 注意

*如果你已经跟随讨论到此为止，`spamlogd`可能已经被自动启动了。然而，如果你的初始`spamd`配置没有包含灰名单功能，`spamlogd`可能没有启动，你可能会遇到一些奇怪的现象，比如灰名单和白名单没有被正确更新。在启用灰名单后重新启动`spamd`应该能确保`spamlogd`被加载并可用。*

为了正确执行其工作，`spamlogd`需要你记录进出邮件服务器的 SMTP 连接，就像我们在第五章的示例规则集中所做的那样：

```
emailserver = "192.0.2.225"
pass log proto tcp to $emailserver port $email
pass log proto tcp from $emailserver to port smtp
```

在 OpenBSD 4.1 及更高版本（以及等效版本）上，你可以创建多个`pflog`接口，并指定规则应该记录到哪里。以下是如何将`spamlogd`需要读取的数据与其他 PF 日志分开的方法：

1.  使用`ifconfig pflog1 create`创建一个单独的`pflog1`接口，或者创建一个仅包含`up`这一行的*hostname.pflog1*文件。

1.  将规则更改为以下内容：

    ```
    pass log (to pflog1) proto tcp to $emailserver port $email
    pass log (to pflog1) proto tcp from $emailserver to port smtp
    ```

1.  将`-l pflog1`添加到`spamlogd`的启动参数中。

这将`spamd`相关的日志记录与其他日志分开。（有关日志记录的更多信息，请参见第九章。）

在前面的规则生效后，`spamlogd`将把接收你发送的邮件的 IP 地址添加到白名单中。这并不是一个铁定保证回复邮件会立即通过，但在大多数配置下，它能显著加速邮件处理过程。

### 灰色拦截

我们知道，垃圾邮件发送者很少使用完全合规的 SMTP 实现来发送他们的邮件，这也是灰名单技术有效的原因之一。我们还知道，垃圾邮件发送者很少检查他们传给被劫持机器的地址是否实际上是可投递的。结合这两个事实，你会发现，如果一个被灰名单列入的机器尝试向你域中的无效地址发送邮件，那么该邮件很有可能是垃圾邮件或恶意软件。

这一认识促使`spamd`开发的下一个进化步骤——一种被称为*灰色拦截*的技术。当一个被灰名单列入的主机尝试向我们域中的已知无效地址发送邮件时，该主机将被添加到本地维护的黑名单`spamd-greytrap`中。`spamd-greytrap`列表中的成员将像其他黑名单成员一样，遭遇每秒 1 字节的延迟。

`spamd`中实现的灰色陷阱简单而优雅。你需要的主要启动条件是`spamd`处于灰名单模式。另一个关键组件是你服务器处理邮件的域中所有地址的列表，但只能包含那些你确定永远不会接收合法邮件的地址。列表中的地址数量不重要，但必须至少有一个，最多的限制主要由你希望添加的地址数量决定。

接下来，你使用`spamdb`将你的列表提供给灰色陷阱功能，并坐下来观看。首先，发送者尝试向你灰名单中的地址发送邮件，并被简单地加入灰名单，就像任何你之前没有交换过邮件的发送者一样。如果同一台机器再次尝试，无论是发送到同一个无效地址，还是另一个在灰名单中的地址，灰色陷阱将被触发，违规者会被放入`spamd-greytrap`，持续 24 小时。在接下来的 24 小时内，来自被灰色陷阱地址的任何 SMTP 流量将被延迟，每次只回复 1 个字节。

那个 24 小时的时间段足够短，不会对合法流量造成严重干扰，因为真实的 SMTP 实现会至少持续几天尝试发送。大规模实施该技术的经验表明，它很少产生误报。在 24 小时后继续发送垃圾邮件的机器很快就会重新进入陷阱。

要设置你的陷阱列表，请使用`spamdb`的`-T`选项。在我的例子中，之前在实践中的灰名单中提到的那个奇怪地址^([37])是一个自然的候选地址：

```
$ **sudo spamdb -T -a wkitp98zpu.fsf@datadok.no**
```

我实际输入的命令是`$ sudo spamdb -T -a "<wkitp98zpu.fsf@datadok.no>"`。在 OpenBSD 4.1 及更高版本中，`spamdb`不需要尖括号或引号，但它仍然可以接受这些符号。

你可以添加任意多的地址。我通常通过在灰名单和邮件服务器日志中查找尝试发送失败的报告到我域中不存在的地址来找到新的垃圾邮件陷阱地址（是的，听起来确实像疯了一样）。 

### 警告

*确保你添加到垃圾邮件陷阱列表中的地址是无效的，并且将始终保持无效。没有什么比发现你将一个有效地址变成垃圾邮件陷阱更让人尴尬的了，哪怕它是暂时的。*

以下日志片段显示了一台发送垃圾邮件的机器如何在第一次联系时被加入灰名单，然后它试图笨拙地向我添加到陷阱列表中的地址发送邮件，几分钟后最终被加入`spamd-greytrap`黑名单。我们知道它接下来大约 20 个小时会做什么。

```
Nov 6 09:50:25 delilah spamd[23576]: 210.214.12.57: connected (1/0)
Nov 6 09:50:32 delilah spamd[23576]: 210.214.12.57: connected (2/0)
Nov 6 09:50:40 delilah spamd[23576]: (GREY) 210.214.12.57: <gilbert@keyholes.net> ->
<wkitp98zpu.fsf@datadok.no>
Nov 6 09:50:40 delilah spamd[23576]: 210.214.12.57: disconnected after 15 seconds.
Nov 6 09:50:42 delilah spamd[23576]: 210.214.12.57: connected (2/0)
Nov 6 09:50:45 delilah spamd[23576]: (GREY) 210.214.12.57: <bounce-3C7E40A4B3@branch15.summerbargainz.
com> -> <adm@dataped.no>
Nov 6 09:50:45 delilah spamd[23576]: 210.214.12.57: disconnected after 13 seconds.
Nov 6 09:50:50 delilah spamd[23576]: 210.214.12.57: connected (2/0)
Nov 6 09:51:00 delilah spamd[23576]: (GREY) 210.214.12.57: <gilbert@keyholes.net> ->
<wkitp98zpu.fsf@datadok.no>
Nov 6 09:51:00 delilah spamd[23576]: 210.214.12.57: disconnected after 18 seconds.
Nov 6 09:51:02 delilah spamd[23576]: 210.214.12.57: connected (2/0)
Nov 6 09:51:02 delilah spamd[23576]: 210.214.12.57: disconnected after 12 seconds.
Nov 6 09:51:02 delilah spamd[23576]: 210.214.12.57: connected (2/0)
Nov 6 09:51:18 delilah spamd[23576]: (GREY) 210.214.12.57: <gilbert@keyholes.net> ->
<wkitp98zpu.fsf@datadok.no>
Nov 6 09:51:18 delilah spamd[23576]: 210.214.12.57: disconnected after 16 seconds.
Nov 6 09:51:18 delilah spamd[23576]: (GREY) 210.214.12.57: <bounce-3C7E40A4B3@branch15.summerbargainz.
com> -> <adm@dataped.no>
Nov 6 09:51:18 delilah spamd[23576]: 210.214.12.57: disconnected after 16 seconds.
Nov 6 09:51:20 delilah spamd[23576]: 210.214.12.57: connected (1/1), lists: spamd-greytrap
Nov 6 09:51:23 delilah spamd[23576]: 210.214.12.57: connected (2/2), lists: spamd-greytrap
Nov 6 09:55:33 delilah spamd[23576]: (BLACK) 210.214.12.57: <gilbert@keyholes.net> ->
<wkitp98zpu.fsf@datadok.no>
Nov 6 09:55:34 delilah spamd[23576]: (BLACK) 210.214.12.57: <bounce-3C7E40A4B3@branch15.
summer-bargainz.com> -> <adm@dataped.no>
```

顺便说一下，尽管垃圾邮件发送者换了机器发送邮件，但`From:`和`To:`地址保持不变。事实证明，他仍然尝试发送到一个从未能成功传递的地址，这强烈表明这个垃圾邮件发送者并没有频繁检查自己的列表。

### 使用 spamdb 管理列表

有时你可能需要查看或更改黑名单、白名单和灰名单的内容。这些记录位于*/var/db/spamdb*数据库中，管理员管理这些列表的主要接口是`spamdb`。

早期版本的`spamdb`只是提供了将白名单条目添加到数据库或更新现有条目的选项（`spamdb -a` *nn*`.`*mm*`.`*nn*`.`*mm*）。你可以删除白名单条目（`spamdb -d` *nn*`.`*mm*`.`*nn*`.`*mm*），以弥补黑名单的缺陷或灰名单算法的影响。近期版本的`spamdb`提供了一些有趣的功能来支持灰陷阱。

#### 更新列表

如果你运行`spamdb`而不带任何参数，它将列出你的`spamdb`数据库的内容，并允许你添加或删除垃圾邮件陷阱地址和陷阱列表条目。你还可以动态地添加白名单条目。

如果你想将主机添加到白名单中，而不将其添加到永久的*nospamd*文件中，并且不重新加载规则集或表格，你可以直接通过命令行执行此操作，方法如下：

```
$ sudo spamdb -a 213.187.179.198
```

如果一个垃圾邮件发送者尽管你做出了最大努力，仍然成功地发送了消息，你可以通过将该垃圾邮件发送者添加到`spamd-greytrap`列表来纠正这一情况，方法如下：

```
$ sudo spamdb -a -t 192.168.2.128
```

添加新的陷阱地址同样简单：

```
$ sudo spamdb -a -T _-medvetsky@ehtrib.org
```

如果你想撤销其中任何一个决定，你只需将这两个命令中的`-a`选项替换为`-d`选项即可。

#### 保持 spamdb 灰名单同步

从 OpenBSD 4.1 开始，`spamd`可以在任意数量的协作灰名单网关之间保持灰名单数据库的同步。其实现是通过一组`spamd`命令行选项：

+   `-Y`选项指定一个*同步目标*，即你希望通知其灰名单信息更新的其他`spamd`运行网关的 IP 地址。

+   在接收端，`-y`选项指定一个*同步监听器*，即该`spamd`实例准备接收来自其他主机的灰名单更新的地址或接口。

例如，我们的主要`spamd`网关`mainoffice-gw.example.com`可能会在其启动命令行中添加以下选项，分别用于建立同步目标和同步监听器：

```
-Y minorbranch-gw.example.com -y mainoffice-gw.example.com
```

相反，位于分支办公室的`minorbranch-gw.example.com`的主机名会是反转的：

```
-Y mainoffice-gw.example.com -y minorbranch-gw.example.com
```

`spamd`守护进程还支持同步伙伴之间的共享密钥认证。具体来说，如果你创建了文件*/etc/mail/spamd.key*并将其分发给所有同步伙伴，它将用于计算认证所需的校验和。*spamd.key*文件本身可以是任何类型的数据，比如从*/dev/arandom*中采集的随机数据，正如`spamd`的手册页所建议的那样。

### 注意

*在无法直接同步与`spamd`相关的数据的情况下，或者如果你只是想与他人共享你的`spamd-greytrap`，将本地陷阱垃圾邮件发送者的列表导出到文本文件中可能是可取的。`spamd-setup`期望的列表格式是每行一个地址，评论行可以选择性地以一个或多个#字符开头。以可用格式导出当前被拦截地址的列表，可以像拼凑一个`spamdb`、`grep`和一些想象力的单行命令一样简单。*

### 检测无序 MX 使用

OpenBSD 4.1 为`spamd`增加了检测无序 MX 使用的功能。首先联系辅助邮件交换服务器而不是主邮件交换服务器是垃圾邮件发送者常用的技巧，且这种做法与我们期望的普通电子邮件传输代理行为相违背。换句话说，如果有人以错误的顺序尝试邮件交换服务器，我们可以相当确定他们是在尝试发送垃圾邮件。

对于我们的*example.com*域，主邮件服务器为 192.0.2.225，备份邮件服务器为 192.0.2.224，在`spamd`的启动选项中添加`-M 192.0.2.224`意味着任何在联系主邮件服务器 192.0.2.225 之前尝试通过 SMTP 联系 192.0.2.224 的主机将在接下来的 24 小时内被加入到本地的`spamdgreytrap`列表中。

### 处理与灰名单不兼容的站点

不幸的是，有些情况你需要为其他站点的电子邮件设置的特殊性进行补偿。

从任何没有与你联系过的站点发送的第一封电子邮件，将会被延迟一段随机时间，这段时间主要取决于发件人的重试间隔。有时候，即使是最小的延迟也不可接受。例如，如果你有一些不常联系的客户，他们要求你在联系时立即并紧急地处理他们的事务，那么初次交付的延迟可能长达几个小时，这样的延迟可能并不理想。此外，你还可能遇到配置错误的邮件服务器，它们可能根本不重试，或者重试得太快，甚至可能在只尝试一次后就停止重试。

此外，一些站点足够大，拥有多个外发 SMTP 服务器，它们与灰名单不兼容，因为它们不能保证会从之前尝试交付时使用的相同 IP 地址重试交付。尽管这些站点遵守了重试要求，但显然这是灰名单的几个剩余缺点之一。

补偿这种情况的一种方法是定义一个本地白名单表，从文件中加载，以应对重启情况。为了确保来自表中地址的 SMTP 流量不被传递到`spamd`，可以添加一个`pass`规则，允许该流量通过：

```
table <nospamd> persist file "/etc/mail/nospamd"
pass in log on egress proto tcp from <nospamd> to port smtp
```

在 OpenBSD 4.7 之前的语法中，需在重定向块的顶部添加一个`no rdr`规则，并添加一个匹配的`pass`规则，以便让来自`nospamd`表中主机的 SMTP 流量通过，如下所示：

```
no rdr proto tcp from <nospamd> to $mailservers port smtp
pass in log on egress proto tcp from <nospamd> to port smtp
```

在对规则集进行这些更改后，将需要保护免受重定向的地址输入到*/etc/mail/nospamd*文件中。然后使用`pfctl -f /etc/pf.conf`重新加载规则集。接下来，你可以对`<nospamd>`表使用所有预期的表技巧，包括在编辑*nospamd*文件后替换其内容。事实上，这种方法在`spamd`的 man 页面和样本配置文件中都有强烈提示。

至少一些具有多个外发 SMTP 服务器的网站会发布有关哪些主机被允许通过发送者策略框架（SPF）记录为其域发送电子邮件的信息，作为域 DNS 信息的一部分。^([38]) 要检索*example.com*域的 SPF 记录，可以使用`host`命令的`-ttxt`选项，如下所示：

```
$ **host -ttxt example.com**
```

此命令将产生类似以下的答案：

```
example.com descriptive text "v=spf1 ip4:192.0.2.128/25 -all"
```

这里，引用中的文本是*example.com*域的 SPF 记录。如果你希望来自*example.com*的电子邮件快速到达，并且你相信那里的人员不会发送或转发垃圾邮件，可以选择 SPF 记录中的地址范围，将其添加到*nospamd*文件中，并从更新的文件重新加载`<nospamd>`表内容。

## 反垃圾邮件技巧

当选择性地使用时，结合`spamd`的黑名单是强大、精确且高效的反垃圾邮件工具。`spamd`机器的负载非常小。另一方面，`spamd`的表现永远不会好于其最弱的数据源，这意味着你需要监控日志，并在必要时使用白名单。

也可以在纯灰名单模式下运行`spamd`，不使用黑名单。事实上，一些用户报告称，纯灰名单的`spamd`配置与使用黑名单的配置在反垃圾邮件方面同样有效，有时比内容过滤更加有效。一项发布在*openbsd-misc*上的报告称，纯灰名单配置立即使该公司消除了大约 95%的垃圾邮件负载。（这份报告可以通过*[`marc.info/`](http://marc.info/)*等地方访问；搜索主题“Followup – spamd greylisting results。”）

我推荐两个非常好的黑名单。一个是基于“过去的新闻组帖子幽灵”的 Bob Beck 的陷阱列表。由阿尔伯塔大学运行`spamd`的计算机自动生成，Bob 的设置是一个定期自动删除被困地址的`spamd`系统，这意味着您会获得极低数量的误报。主机数量差异很大，最高曾达到 67 万。虽然仍处于测试阶段，但该列表于 2006 年 1 月公开。该列表可从*[`www.openbsd.org/spamd/traplist.gz`](http://www.openbsd.org/spamd/traplist.gz)*获取。它是最近示例*spamd.conf*文件中的`uatraps`黑名单的一部分。

我推荐的另一个列表是*heise.de*的`nixspam`，它具有 12 小时的自动过期时间和极高的准确性。它也在示例*spamd.conf*文件中。有关此列表的详细信息可从*[`www.heise.de/ix/nixspam/dnsbl_en/`](http://www.heise.de/ix/nixspam/dnsbl_en/)*获取。

一旦您对您的设置感到满意，请尝试引入本地灰色陷阱。这可能会捕捉到更多不受欢迎的人，并且这是一种有趣、干净的乐趣。一些有限的实验—在撰写本章时进行（在*[`bsdly.blogspot.com/`](http://bsdly.blogspot.com/)*中以*[`bsdly.blogspot.com/2007/07/hey-spammer-heres-list-for-you.html`](http://bsdly.blogspot.com/2007/07/hey-spammer-heres-list-for-you.html)*开头的条目中记录）—甚至表明，从您的邮件服务器日志、`spamd`日志或直接从您的灰名单中收集垃圾邮件使用的无效地址放入您的陷阱列表是非常有效的。将列表发布在一个适度可见的网页上似乎可以确保您放在那里的地址将一遍又一遍地被地址收集机器人记录，并且将为您提供更好的灰色陷阱材料，因为它们更有可能被保留在垃圾邮件发送者已知良好地址列表上。

* * *

^([30]) 在撰写本章时，这仅仅是理论性的；我还没有收到任何可靠的报告表明这种情况正在发生。这在 2012 年发生了变化，当可靠的消息来源开始报告在奇怪的端口出现暴力序列。更多信息请参见*[`bsdly.blogspot.com/2013/02/theres-no-protection-in-high-ports.html`](http://bsdly.blogspot.com/2013/02/theres-no-protection-in-high-ports.html)*。

^([31]) 要了解“Hail Mary Cloud”暴力尝试序列的概述，请参阅文章“Hail Mary Cloud and the Lessons Learned” *[`bsdly.blogspot.com/2013/10/the-hail-mary-cloud-and-lessons-learned.html`](http://bsdly.blogspot.com/2013/10/the-hail-mary-cloud-and-lessons-learned.html)*。更多资源在那里和附录 A 中有引用。

^([32]) 从长期来看，整个网络和更大范围的 IP 地址被重新分配给新所有者是相当正常的，通常是响应于物理世界或商业领域的事件。

^([33]) 在 `pfctl` 获取过期表项的能力之前，表项过期通常由专用工具 `expiretable` 处理。如果你的 `pfctl` 没有 `expire` 选项，你应该认真考虑升级到更新的系统。如果升级由于某种原因不切实际，可以在你的包管理系统中寻找 `expiretable`。

^([34]) 原始的 Harris 论文以及其他一些有用的文章和资源可以在 *[`www.greylisting.org/`](http://www.greylisting.org/)* 找到。

^([35]) RFC 5321 的相关部分与 RFC 2821 的相应部分相同，后者已废弃。我们中的一些人对于 IETF 没有澄清这些文本内容而感到有些失望，尽管该标准现已向前推进。我的反应（实际上，这是一次相当激烈的抱怨）可以在 *[`bsdly.blogspot.com/2008/10/ietf-failed-to-account-for-greylisting.html`](http://bsdly.blogspot.com/2008/10/ietf-failed-to-account-for-greylisting.html)* 中看到。

^([36]) 相关的 RFC 主要是 RFC 1123 和 RFC 5321，它们废弃了早期的 RFC 2821。请记住，临时拒绝是 SMTP 容错特性的一部分。

^([37]) 当然，这个地址完全是假的。它看起来像是 GNUS 邮件和新闻客户端生成的消息 ID，可能是从新闻存档或某个不幸的恶意软件受害者的邮箱中窃取的。

^([38]) SPF 记录以 TXT 记录的形式存储在 DNS 区域中。详情请参见 *[`www.openspf.org/`](http://www.openspf.org/)*。
