## 第十章：**重建用户桌面和登录活动**

![Image](img/common01.jpg)

重建用户登录活动通常是必要的，以了解某人何时登录系统，如何登录，他们在做什么，以及何时最终注销。本章解释了 shell 和桌面用户登录的各个方面，并描述了从数字取证角度来看有趣的各种证据。

我们主要关注*人类*与计算机的交互。其他系统“用户”正在运行守护进程或启动程序，但它们是正常系统操作的一部分，并且在本书的其他部分有所涉及。人类使用外设，如打印机、外部硬盘等，也会在第十一章中单独讨论。

### Linux 登录和会话分析

在早期的 Unix 系统中，用户通过物理终端或由 PC 仿真出的终端登录，二者都通过 RS232 串行线连接。远程连接可以通过拨号或本地电话公司租用的模拟调制解调器来实现。随着 TCP/IP 的流行，用户通过 telnet 或 rlogin 在网络上登录。用户输入登录名和密码，如果正确，系统会运行脚本来设置环境并提供命令行提示符。当用户完成操作时，他们会注销，终端会被重置，以准备下次登录。

现在，人们通过本地控制台或通过网络安全地登录。登录 Linux 系统的最常见方式有：

+   通过本地 *显示管理器* 的图形登录（通常用于工作站和笔记本电脑）

+   在本地虚拟控制台上的 shell 登录（通常是物理服务器访问）

+   使用安全外壳（SSH）远程登录（通常是远程服务器访问）

+   通过本地串行线的 shell 登录（通常用于嵌入式系统或基于 Linux 的物联网设备）

图 10-1 显示了这些用户登录方式的简化概览。

![Image](img/ch10fig01.jpg)

*图 10-1：系统初始化和用户登录过程*

上述列出的前三种登录方式主要用于人工交互。最后一种登录方式主要用作配置、固件更新或诊断程序的接口，它可能直接使用电路板上的内部引脚。串行线登录在嵌入式设备和物联网设备的取证分析中非常有用，因为这些设备的存储不能像普通计算机那样被移除和镜像。

像 VNC 这样的远程桌面连接在此没有列出，因为它们通常连接到已经登录的桌面或远程访问显示管理器。在这种情况下，远程桌面可以像本地图形登录一样进行分析。远程桌面访问将在本章末尾解释。

以下部分描述了登录会话的工作原理，并识别可能存在的有趣数字取证证据。

#### *席位和会话*

要分析 Linux 系统上的人类用户活动，我们必须理解座席、用户和会话的概念。

*座席*通常由一个或多个屏幕、键盘和鼠标（除非是触摸屏）、音频设备、摄像头及其他人机交互外设组成，这些设备连接到本地工作站。默认座席名为 `seat0`，并在系统启动时被识别。我们可以在 systemd 日志中查看它：

```
Jul 23 13:06:11 pc1 systemd-logind[316]: New seat seat0.
```

当一台 PC 配备了多个键盘和显示器供多人使用时（尽管这种情况比较少见），Linux 系统可以配置为拥有额外的座席。

我们可以通过登录 `loginctl seat-status seat0` 来查看座席的设备组件，但在事后取证调查中，这些信息是不可用的，必须从日志中推断或重建。有关座席的更多信息，请参阅 sd-login(3) 手册。

*用户*一词可以指人或进程。人类用户是指拥有计算机用户账户的人，这对应于传统 Unix 的用户名和数字用户 ID（UID）。系统进程（不是人类）也在指定的用户名和 UID 下运行。在对系统进行取证分析时，区分人类用户和系统用户活动非常重要。人类用户通常通过座席或远程 SSH 或其他远程访问方法登录。非人类（系统进程）用户通常是由 systemd 或其他系统用户启动的守护进程。

*会话*是用户登录的持续时间，可以发生在物理座席上，也可以通过网络连接（如 SSH）进行。成功登录后，用户会获得一个会话 ID，且会话在注销时会被干净地终止。会话由 systemd-logind 记录和管理。Systemd 和显示管理器一起，还可以实现*快速用户切换*。这意味着多个用户可以同时登录到同一座席，并能安全地在用户之间切换控制权。

**注意**

*“会话”一词在计算机中有多重含义。包括系统登录会话、桌面会话、应用登录会话、浏览器会话、TCP 会话、SSL/TLS 会话等。在进行取证分析和编写取证报告时，请确保清楚理解会话一词的使用。*

早期的 Unix 系统价格昂贵，开发了会计日志来便于对用户或部门进行计费。管理员需要了解用户何时登录、何时注销，以及其他可能的使用信息。在现代 Linux 系统中，这主要由 systemd 管理，但一些传统文件仍记录用户登录会话的状态和历史：

| ***/var/log/wtmp*** | 成功登录和注销的历史 |
| --- | --- |
| ***/var/log/btmp*** | 失败的登录尝试历史 |
| ***/var/log/lastlog*** | 最近的用户登录记录 |
| ***/var/run/utmp*** | 当前登录的用户（仅在运行的系统上） |

在进行现代 Linux 系统的事后法医分析时，任何存储在伪文件系统上的临时数据将无法获得（伪文件系统存储在内存中）。除非从内存镜像中恢复，否则无法分析 */var/run/utmp*。

`utmpdump`^(1) 工具可用于查看 *wtmp* 和 *btmp*（以及运行系统中的 *utmp*）的原始内容。以下是一些示例条目：

```
[1] [00000] [~~  ] [shutdown] [~          ] [5.7.9-arch1-1   ]
 [0.0.0.0        ] [2020-07-23T07:54:31,091222+00:00]
[2] [00000] [~~  ] [reboot  ] [~          ] [5.7.9-arch1-1   ]
 [0.0.0.0        ] [2020-07-23T07:59:19,330505+00:00]
[5] [00392] [tty1] [        ] [/dev/tty1  ] [                ]
 [0.0.0.0        ] [2020-07-23T07:59:21,363253+00:00]
[6] [00392] [tty1] [LOGIN   ] [tty1       ] [                ]
 [0.0.0.0        ] [2020-07-23T07:59:21,363253+00:00]
[7] [00392] [tty1] [sam     ] [tty1       ] [                ]
 [0.0.0.0        ] [2020-07-23T07:59:31,017548+00:00]
[7] [14071] [s/11] [sam     ] [pts/11     ] [10.0.1.30       ]
 [10.0.1.30      ] [2020-07-24T01:44:54,513510+00:00]
[6] [32537] [    ] [ftpuser ] [ssh:notty  ] [122.224.217.42  ]
 [122.224.217.42 ] [2020-07-25T05:46:17,000000+00:00]
```

输出字段（从左到右，换行后的字段）如下所示，带有描述：^(2)

***type*** 记录类型（见下方的类型列表）

***pid*** 登录进程的 PID（agetty、sshd，或重启和关机时为 0）

***id*** 终端名称后缀（tty 的最后四个字符；如果没有则为空或为波浪符）

***user*** 用户名（失败或成功）或操作（关机、重启等）

***line*** 终端设备名称（如果没有则为波浪符）

***host*** 主机名或 IP 地址字符串（或某些类型的内核信息）

***addr*** IP 地址（IPv4 或 IPv6，如果可用）

***time*** 记录的时间戳

根据记录类型和写入 *wtmp* 或 *btmp* 的程序，字段的内容可能用于不同的信息。例如，类型 1 或 2 时，*user* 字段用于记录关机或重启，*host* 字段记录内核版本。同时注意，*id* 和 *line* 相似，*host* 和 *address* 也类似。任何程序都可以写入 *wtmp* 或 *btmp* 并选择它要使用的字段。虽然这看起来有些冗余，但它增加了不同程序所保存的日志信息量。

以下记录类型编号存储在 *wtmp* 和 *btmp*（以及 */var/run/utmp*）中：

**0**   无效数据

**1**   运行级别或等效的 systemd 目标发生变化

**2**   启动时间

**3**   时钟变更前的时间戳

**4**   时钟变更后的时间戳

**5**   由 init 启动的进程

**6**   提供登录提示

**7**   用户登录成功

**8**   进程终止（注销）

欲了解更多信息，请参阅 utmp(5) 手册页。

**NOTE**

*在法医检查中，查找可能的密码在* btmp *文件中。如果用户在登录提示时不小心输入了密码，它将被记录在这里。*

`utmpdump` 的替代工具有 `utmpr`^(3)（GitHub 上的 *[`github.com/m9/lastlog/`](https://github.com/m9/lastlog/)*) 和一个一行的 Perl 脚本来转储 *wtmp* 文件（* [`www.hcidata.info/wtmp.htm`](https://www.hcidata.info/wtmp.htm)*）。

此外，*/var/log/lastlog* 文件包含系统中每个用户的最近登录信息。这是一个稀疏的二进制文件，可以通过 `lastlog` 命令在运行的系统上读取。在单独的 Linux 检查主机上运行 `lastlog` 将产生不正确的结果，因为它读取本地密码文件，因此必须使用离线法医工具。

以下三行 Perl 脚本 (*lastlog.pl*) 解析嫌疑 Linux 系统的离线 *lastlog* 文件：

```
#!/bin/perl -w
$U=0;$/=\292;while(<>){($T,$L,$H)=unpack(IZ32Z256,$_);if($T!=0)
{printf("%5d %s %s %s\n",$U,scalar(gmtime($T)),$L,$H);}$U++;};
```

在离线检查机器上运行时，输出类似于以下内容：

```
$ ./lastlog.pl lastlog
    0 Sun Jul 26 09:35:06 2020 tty3
 1000 Sun Jul 26 08:48:19 2020 pts/2 10.0.0.35
 1001 Mon Mar 30 05:41:18 2020 pts/0 10.0.0.35
```

输出从数字 UID 开始，后跟时间戳。最后两列是使用的行（或终端）和主机名或 IP 地址（如果存在）。相同的信息也包含在 *wtmp* 日志中，并应匹配。

`lslogins` 工具将关于 *wtmp*、*btmp* 和 *lastlog* 的信息转储到一个表格中（使用 `--output-all` 标志）。也可以指定在分析机器上使用哪些离线文件副本。然而，运行此命令仍然会读取本地分析机器上的 */etc/passwd* 和 */etc/shadow*，从而生成不正确的输出。

**注意**

*在分析机器上运行针对实时系统的工具时要小心。在许多情况下，生成的数据将不会关于嫌疑驱动器，而是来自您自己的分析机器。*

一些机器会有一个 */var/log/tallylog* 文件。该文件维护 `pam_tally` 的状态，这是一个 PAM 模块，用于计算实时系统上的登录尝试，可能会在登录失败次数过多时进行阻止。有关更多信息，请参见 pam_tally2(8) 手册页。

#### *Shell 登录*

用户可以通过本地控制台^(4)登录到 Linux 系统，或者通过 SSH 远程登录。成功验证和授权后，启动一个名为 shell 的程序，用户可以与系统进行交互。这个 shell 程序解释并执行用户输入的命令，或从文本文件中读取并作为 shell 脚本运行。

Linux 系统上最常见的 shell 程序是 Bash；然而，zsh 和 fish 也有活跃的用户社区。默认 shell 定义在用户的 */etc/passwd* 条目的最后一个字段中。本节重点介绍 Bash，但取证检查原理应适用于任何 shell（请参考特定 shell 的手册页获取帮助）。

Shell 可以是 *交互式*（针对用户）或 *非交互式*（针对脚本）。当作为 *登录* shell 启动时（通常是登录时的第一个 shell），会运行几个额外的启动脚本。图 10-1 在本章前面显示了获取登录 shell 的典型过程。

本地 Linux 控制台是通过 PC 显示器和键盘提供的文本模式接口。通过这个物理接口，可以使用多个“虚拟控制台”，可以通过热键（ALT-FN 或 CTRL-ALT-FN）或 `chvt` 程序进行切换。

当虚拟控制台变为活动状态时，Systemd-logind 启动 agetty^(5) 程序。agetty 守护进程设置终端并显示登录提示。输入用户名后，传递给登录程序，要求输入密码。如果用户名和密码正确且用户已授权，则在用户的 UID 和组 ID（GID）下启动 shell。

通过网络登录到 shell 从 telnet 和 rlogin 引入网络协议以来就一直是可能的。如今，远程登录通常使用更安全的替代方案，如 SSH。

默认情况下，SSH 守护进程（sshd）监听 TCP 端口 22。当收到传入的网络连接时，建立加密通道，进行用户认证，并启动一个 shell。关于分析 SSH 的更多细节将在本章稍后提供，但之前给出的 图 10-1 提供了网络登录的概述。

Linux 系统使用 PAM 库进行多种登录活动。PAM 模块检查密码、验证用户、确定授权并执行其他登录前检查。现代 Linux 系统中的一个重要功能是启动 systemd 用户实例（如果尚未启动）。登录成功后，PAM 将会话注册到 systemd-logind，进而启动 systemd 用户实例。systemd 用户实例有一个 `default` `.target`，在用户最终获得 shell 命令提示符之前，会启动多个单元文件（用户守护进程，例如 D-Bus）。

可以通过日志查看 shell 登录活动。此示例展示了一个 SSH 登录，随后是注销：

```
Aug 16 20:38:45 pc1 sshd[75355]: Accepted password for sam from 10.0.11.1 port 53254 ssh2
Aug 16 20:38:45 pc1 sshd[75355]: pam_unix(sshd:session): session opened for user sam by (uid=0)
Aug 16 20:38:45 pc1 systemd-logind[374]: New session 56 of user sam.
Aug 16 20:38:45 pc1 systemd[1]: Started Session 56 of user sam.
...
Aug 16 20:39:02 pc1 sshd[75357]: Received disconnect from 10.0.11.1 port 53254:11: disconnected
 by user
Aug 16 20:39:02 pc1 sshd[75357]: Disconnected from user sam 10.0.11.1 port 53254
Aug 16 20:39:02 pc1 sshd[75355]: pam_unix(sshd:session): session closed for user sam
Aug 16 20:39:02 pc1 systemd[1]: session-56.scope: Succeeded.
Aug 16 20:39:02 pc1 systemd-logind[374]: Session 56 logged out. Waiting for processes to exit.
```

注意，在前面三行中，SSH 守护进程接收连接并启动 `pam`，然后涉及 systemd。SSH 登录也可以在 syslog 文件中找到，如 */var/log/auth.log*，或者在其他传统 Unix 位置。

##### Shell 启动文件

成功登录后，shell 启动并运行多个脚本来设置环境。一些系统脚本由系统管理员配置并由每个用户运行，但用户也可以在自己的主目录中创建和修改其他脚本。Shell 启动脚本（以 Bash 为例）通常包括以下内容：

+   */etc/profile*

+   */etc/profile.d/**

+   *~/.bash_profile*

+   */etc/bash.bashrc*

+   *~/.bashrc*

配置文件脚本仅在登录 shell 中运行（通常是用户登录时的第一个 shell）。其他脚本（**rc*）在每次调用 shell 时运行。

退出或注销时，会运行额外的脚本，通常包括以下内容：

+   */etc/bash.bash_logout*

+   *~/.bash_logout*

应检查这些文件是否有偏离默认设置的变化。特别是，用户在主目录中的自定义可能会很有趣。在系统范围内被攻击的情况下，恶意修改也可能发生在 */etc/* 文件中。

环境变量，特别是那些显式设置的变量，可能很有趣，并且可能透露使用的程序或自定义配置。`PATH` 变量可能指向一个附加目录，其中包含用户自己的脚本和二进制文件。`VISUAL` 和 `EDITOR` 变量表示使用的默认编辑器，并且根据编辑器的不同，可能指向关于已编辑文件的附加缓存和历史信息。

Systemd 和 PAM 提供了额外的位置来设置登录时的环境变量：

+   */etc/security/pam_env.conf*

+   */etc/environment*

+   */etc/environment.d/*.conf*

+   */usr/lib/environment.d/*.conf*

+   *~/.config/environment.d/*.conf*

你可以在 environment.d(5) 和 pam_env.conf(5) 的手册页中找到更多信息。存储在 */run/* 中或在正在运行的系统的内存中修改的变量在事后取证分析中不可用。

##### Shell 历史

大多数 shell 可以保存输入的命令历史，以便用户可以搜索并调用这些命令，而不是重新输入它们。从调查角度来看，这个命令历史特别有趣，因为这些命令是由人类用户显式输入的。然而，在登录被破坏的情况下，命令历史也可能来自恶意脚本。

Shell 历史通过环境变量（以 `HIST*` 开头）进行配置，这些变量指定了使用的文件、保存的命令数量、时间戳格式以及特定 shell 提供的其他历史功能。默认的 Bash 历史文件是 *~/.bash_history*。该文件包含一个简单的命令列表。希望提高取证准备的组织可以在 Bash 中设置 `HISTTIMEFORMAT` 变量，以在历史记录中包含时间戳。每个用户（包括 root）都可能有一个 shell 历史文件。

检查 shell 历史记录可以提供有关用户活动和行为的洞察。可以在 shell 历史记录中观察到或寻找的项目、活动和行为包括：

+   技能水平（简单命令或表明初学者的错误）

+   揭示的文件名（来自创建、编辑或删除的文件）

+   修改系统配置的命令

+   手动设置隧道、代理或 VPN

+   挂载本地或远程文件系统或加密容器

+   测试本地守护进程或远程主机上的功能

+   输入的密码（无论是偶然输入还是作为命令行参数）

+   通过运行 `ping`、`nslookup`、`ssh` 或其他网络工具泄露的其他 IP 地址或主机名

+   意外复制/粘贴到终端窗口中的文本信息

+   任何揭示意图或思维过程的命令序列

输入的命令会存储在内存中，并在 shell 退出时写入历史文件。历史文件可能包含来自多个不同时间退出的 shell 实例的命令，因此保存的命令可能不是按时间顺序排列的。

如果历史文件被显式禁用、删除、清空或符号链接到 */dev/null*，则表明怀疑用户或攻击者具有一定的安全意识或更高的技能水平。有关 Bash 历史取证的精彩 SANS 演讲，请参见 *[`youtu.be/wv1xqOV2RyE/`](https://youtu.be/wv1xqOV2RyE/)*。

#### *X11 和 Wayland*

X11 窗口系统曾是 Unix 系统的事实标准图形界面，也是 Linux 社区的自然选择。如今，Linux 上最流行的 X11 实现是 [X.Org](http://X.Org)，自从从 XFree86 项目分叉以来，已经加入了许多新的扩展和增强功能。

[X.Org](http://X.Org)将应用程序连接到输入设备（如键盘、鼠标、触摸屏等）以及输出设备，如显卡和显示器。除了[X.Org](http://X.Org)之外，还需要一个单独的窗口管理器来管理窗口（放置、装饰、调整大小、移动等）。在窗口管理器之上，桌面环境通常会提供额外的“外观和感觉”或甚至完全独立的图形外壳。这些组件和子组件中的每一个都可能存储在数字取证环境中有用的信息。

大部分的[X.Org](http://X.Org)配置是自动完成的；然而，手动调整和定制通常可以在*/etc/X11/xorg.conf*或*/etc/X11/xorg.conf.d/*目录下的文件中找到。默认情况下，会创建[X.Org](http://X.Org)活动日志，并写入到*/var/log/Xorg.0.log*（在某些情况下，可能位于用户的*.local/share/xorg/Xorg.0.log*中）。该文件的内容描述了图形硬件、显示器、输入设备、默认屏幕分辨率等信息。以下是从此类日志中提取的一些示例：

```
...
[  31.701] (II) NVIDIA(0): NVIDIA GPU GeForce GTX 1050 Ti (GP107-A) at PCI:1:0:0 (GPU-0)
[  31.701] (--) NVIDIA(0): Memory: 4194304 kBytes
[  31.701] (--) NVIDIA(0): VideoBIOS: 86.07.59.00.24
...
[  31.702] (--) NVIDIA(GPU-0): LG Electronics LG ULTRAWIDE (DFP-2): connected
...
[  31.707] (II) NVIDIA(0): Virtual screen size determined to be 3840 x 1600
...
[  31.968] (II) config/udev: Adding input device Logitech M280/320/275 (/dev/input/event5)
...
[  31.978] (II) XINPUT: Adding extended input device "LEOPOLD Mini Keyboard" (type: KEYBOARD,
 id 12)
...
```

日志中可能还存在其他实例，如*/var/log/Xorg.1.log*文件。与旋转日志文件不同，这不是旧版本，而是表示已记录的显示（0、1，依此类推）。日志的旧版本也可能存在，并且具有*.old*文件扩展名。

*Xorg*日志文件包含用于描述日志条目的“标记”：

(–)     探测到

(**)     来自配置文件

(==)     默认设置

(++)     来自命令行

(!!)     注意

(II)     信息

(WW)     警告

(EE)     错误

(NI)     未实现

(??)     未知

如果用户曾使用 X11 并后来切换到 Wayland，这个日志可能仍然存在，并提供从早期时间点的信息。你可以在 Xorg(1)手册页中找到更多关于[X.Org](http://X.Org)的信息。

图 10-2 展示了 X11 的基本架构。桌面计算的发展使得许多 X11 原始设计决策变得过时，因此需要一种更现代的窗口系统。Wayland 被设计为替代品，并且大多数 Linux 发行版正朝着基于 Wayland 的桌面系统发展。

![Image](img/ch10fig02.jpg)

*图 10-2：X11 架构*

窗口管理器用于 X11 环境中管理窗口。从功能上讲，窗口管理器只是另一个 X11 客户端。大多数 Linux 发行版和图形环境都有默认的窗口管理器。一些流行的 X11 窗口管理器包括：

+   Mutter（GNOME 默认）

+   KWin（KDE 默认）

+   Xfwm4（Xfce 默认）

+   Openbox（LXDE 默认）

+   Fluxbox、FVWM 以及像 i3 这样的平铺窗口管理器

每个窗口管理器都会有自己的配置和日志文件。有关更多信息，请参阅相关文档。

Wayland 使用与 X11 不同的模型，将窗口管理、合成和其他功能结合在一起。图 10-3 显示了 Wayland 的架构。通过比较 X11 和 Wayland 的架构，可以看到它们的差异。顺便提一下，Wayland 并不专属于 Linux，还用于其他操作系统，如 BSD。

![Image](img/ch10fig03.jpg)

*图 10-3：Wayland 架构*

关于 X11 和 Wayland 之间架构差异的更多信息，请访问 *[`wayland.freedesktop.org/architecture.html`](https://wayland.freedesktop.org/architecture.html)*。

Wayland 合成器越来越受欢迎。Mutter 和 KWin 都支持 Wayland（除了 X11），而高级用户则使用像 Sway（Wayland 的 i3 克隆）或 Hikari（最初为 FreeBSD 开发）这样的专业合成器。每个合成器都有可以检查的配置和日志功能，但对个别合成器的法医分析超出了本书的范围。

#### *桌面登录*

典型的 Linux 桌面和笔记本系统有一个图形登录屏幕。这个屏幕有时被称为 *greeter*，由一个名为 *display manager* 的守护进程提供。显示管理器设置本地机器的图形并提供登录前的选项（例如语言、屏幕亮度、辅助功能等）。

显示管理器与使用的图形环境无关，它可能允许用户在登录后选择想要使用的图形环境。目前最流行的显示管理器是 GDM（GNOME 默认）和 SDDM（KDE Plasma 默认）。

你可以通过检查 systemd *display-manager.service* 单元文件来确定使用的桌面管理器，该文件是实际显示管理器的符号链接。在以下示例中，默认目标被符号链接到图形目标，并指定（`Wants=`）显示管理器服务：

```
default.target -> /lib/systemd/system/graphical.target
Wants=display-manager.service
```

显示管理器服务是符号链接到 GDM 服务，该服务启动（`ExecStart=`）GDM 守护进程：

```
display-manager.service -> /usr/lib/systemd/system/gdm.service
ExecStart=/usr/bin/gdm
```

根据配置，GDM 显示管理器可能会将日志保存在 */var/log/gdm/* 中，或者在 systemd 日志中留下痕迹。

SDDM 显示管理器可能将日志保存在 */var/log/sddm.log* 中，并且还会在 systemd 日志中记录活动（搜索 `sddm`）。成功登录后，SDDM 显示管理器将会话日志存储在用户的主目录中，供检查：

```
$ ls -l /home/sam/.local/share/sddm
total 24
-rw------- 1 sam sam 20026 Jun 14 12:35 wayland-session.log
-rw------- 1 sam sam 2514 Jun 14 15:38 xorg-session.log
```

可能存在与桌面环境相关的其他日志，这些日志与 Wayland 或 X11 会话相关。

通过显示管理器成功登录后，多个进程将被启动。例如：

+   系统用户实例（`systemd --user`）

+   桌面会话管理器（`gnome-session`、`plasma_session`、`xfce4-session` 等）

+   窗口管理器（如果运行 X11）

+   系统用户单元

+   XDG 会话自动启动项（**.desktop** 文件）

+   D-Bus 会话实例

+   代理（polkit、gpg、ssh 等）

+   桌面或图形外壳

+   支持桌面环境的守护进程（设置、Pulseaudio 或 PipeWire、蓝牙等）

每个组件将在用户的 UID 下运行。配置、日志、缓存和其他相关数据通常可以在用户的 XDG 目录中找到。（参考图 10-1 了解图形登录过程的概述。）

systemd *用户*实例（与 systemd *系统*实例不同）负责激活启动和监督登录会话所需的单元。当 PAM 将会话注册到 systemd-logind 时，启动 systemd 用户实例。用户单元文件位于这里：

+   */usr/lib/systemd/user/**

+   */etc/systemd/user/**

+   *~/.config/systemd/user/**

每个目录都会覆盖前一个目录。前两个目录是供应商和系统管理员的默认值。最后一个目录包含用户主目录中的自定义配置。在取证检查中，可以检查这些目录以发现与预期默认设置的偏差，或系统管理员、用户或恶意行为者添加的任何自定义内容。systemd 用户实例的系统范围配置文件位于*/etc/systemd/user.conf*和*/etc/systemd/logind.conf*文件中。

除了 systemd 用户实例，桌面会话管理器还将通过自己的启动文件启动用户的登录环境。XDG 桌面定义文件（**.desktop**）提供启动用户桌面环境所需的信息。XDG 标准还定义了存储桌面配置文件的常用位置。这些文件位于自动启动目录中，并且特定于桌面环境的文件会被读取并启动相应的应用程序。系统默认和用户自定义的目录位置（用户创建的文件优先）位于这里：

+   */etc/xdg/autostart/**

+   *~/.config/autostart/**

窗口管理器和桌面外壳也可能有自己的*autostart*目录，其中包含启动相关组件的桌面文件。XDG 桌面条目规范可以在*[`specifications.freedesktop.org/desktop-entry-spec/`](https://specifications.freedesktop.org/desktop-entry-spec/)*找到。

定义文件具有**.desktop**扩展名，并描述桌面组件应如何启动。以下示例展示了几个条目，说明定义文件的内容：

```
$ cat gnome-keyring-secrets.desktop
[Desktop Entry]
Type=Application
...
Name[en_GB]=Secret Storage Service
...
Comment[de]=GNOME-SchlÃsselbunddienst: Sicherheitsdienst
...
Exec=/usr/bin/gnome-keyring-daemon --start --components=secrets
OnlyShowIn=GNOME;Unity;MATE;Cinnamon;
...
```

这里描述了一个应用程序（稍后在本章讨论的 GNOME Keyring）。文件包含名称和注释的多语言内容，并指定定义文件有效的上下文。同时也定义了要执行的程序和标志。

Systemd 和 XDG 都提供了类似的功能来设置桌面环境。由于 XDG 被广泛使用，并且主要发行版承诺保持兼容性，两个系统都可以进行检查。许多桌面环境正在将 XDG 桌面启动活动迁移到 systemd，这一过程复杂，需要与 XDG **.desktop** 文件保持向后兼容。如果 **.desktop** 文件包含 `X-GNOME-Hidden UnderSystemd=true` 这一行，表示 GNOME 会话管理器应忽略该文件，因为它是由 systemd 启动的。^(6)

一些会话管理器可以保存和恢复桌面状态。这些文件可以检查，以确定之前保存状态中哪些窗口可能已经打开。保存的会话信息的位置因桌面环境而异，常见的位置包括：

+   *~/.cache/sessions/*

+   *~/.config/session/*

+   *~/.config/gnome-session/saved-session/*

+   *~/.config/ksmserverrc*

会话可能会在退出时自动保存，或者由用户明确请求。根据桌面环境及其配置，会话管理器可能保存一个简单的打开程序列表，并包含窗口的大小和屏幕上的位置。

##### 快速用户切换

通过在不同的虚拟控制台中启动会话，多个用户可以同时登录到不同的图形环境中。用户之间的切换可以通过热键（CTRL-ALT-FN）、`chvt` 命令，或当前图形环境中的*切换用户*选项来完成。用户切换可能意味着多个人在同一台机器上使用，或者一个人在同一台机器上使用多个身份。

也称为*快速用户切换*，通常在图形环境中提供一个菜单选项（如果有多个用户），该选项会锁定屏幕并跳转到显示管理器，在那里可以验证另一个用户的身份。根据显示管理器的不同，这一过渡可能会出现在日志中。以下示例日志显示，由于用户切换，启动了一个新的 GDM 会话（登录屏幕），并在不到一分钟后，在第二个用户成功验证后结束：

```
Jul 03 15:05:42 pc1 systemd-logind[401]: New session 26 of user gdm.
Jul 03 15:05:42 pc1 systemd[1]: Started Session 26 of user gdm.
...
Jul 03 15:06:20 pc1 systemd-logind[401]: Session 26 logged out. Waiting for
 processes to exit.
Jul 03 15:06:20 pc1 systemd-logind[401]: Removed session 26.
```

当显示管理器在没有用户注销的情况下启动时，表明可能发生了用户切换。这些信息提供了一个起点，可以检查周围的日志和文件系统时间戳，以确定切换前后哪些用户是活跃的。用户切换也可以由另一位用户在锁定屏幕时发起。

### 身份验证与授权

当用户想要访问 Linux 系统时，在授予访问权限之前会进行一系列检查。系统会寻找一个指示，确认此人确实是他们所声称的身份，并且他们确实有权访问所需的资源。如今，这通常通过 PAM 来完成。PAM 可以在用户和系统之间提供身份验证和授权控制，无论是在登录时还是在整个用户登录会话期间。

PAM 配置文件位于 *pam.conf* 文件和 */etc/pam.d/* 目录下。PAM 还记录尝试进行身份验证和授权的成功与失败。以下是几个示例：

```
➊ Dec 26 19:31:00 pc1 sshd[76857]: pam_unix(sshd:session): session opened for
    user sam(uid=1000) by (uid=0)
   Dec 26 19:31:20 pc1 sshd[76857]: pam_unix(sshd:session): session closed for
    user sam
   ...
➋ Dec 26 19:26:50 pc1 login[76823]: pam_unix(login:session): session opened for
    user sam(uid=1000) by LOGIN(uid=0)
   Dec 26 19:28:04 pc1 login[76823]: pam_unix(login:session): session closed for
    user sam
   ...
➌ Dec 26 19:45:40 pc1 gdm-password][6257]: pam_unix(gdm-password:session):
    session opened for user sam(uid=1000) by (uid=0)
   Dec 26 19:46:46 pc1 gdm-password][6257]: pam_unix(gdm-password:session):
    session closed for user sam
```

前两行 ➊ 显示的是通过网络进行的 SSH 登录和登出的日志。接下来的两行 ➋ 显示的是在本地虚拟控制台上（文本登录提示）进行的登录和登出的日志。最后两行 ➌ 显示的是使用 GDM（典型的图形登录界面）进行的登录和登出。

#### *用户、组和密码文件*

Linux 采用了 Unix 的用户名和组的概念和实现。传统上，这些用户名和组会列在 */etc/* 目录中的几个文件里。^(7) 密码文件 */etc/passwd*（不再包含密码）列出了系统中定义的用户及一些附加信息。影子文件 */etc/shadow* 包含每个（已启用）用户的哈希密码。组文件 */etc/group* 列出了组及其成员。每个用户都有一个默认的组（通常以用户名命名），并可以被添加到其他组以便访问文件和资源。

*passwd*、*shadow* 和 *group* 文件的格式在 passwd(5)、^(8) shadow(5) 和 group(5) 手册页中有描述。这些文件是纯文本格式，每行一个用户/组，每行有多个字段。以下是 *passwd* 文件中的一些摘录：

```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
sam:x:1000:1000:Samantha Samuel:/home/sam:/bin/bash
```

*passwd* 文件的字段（用冒号分隔）如下：

+   登录名

+   密码字段（`x` 表示密码存储在 */etc/shadow* 中；`!` 表示密码访问被锁定；空字段表示不需要密码，应用程序可以选择允许访问）

+   数字用户 ID

+   数字组 ID

+   注释字段（通常是用户的全名）

+   用户的主目录

+   用户的 Shell 程序（`nologin` 程序会直接拒绝登录尝试）

*/etc/passwd* 文件在早期 Unix 系统中曾是盗窃的主要目标。任何窃取此文件的人都可以获得一个用户列表和加密/哈希密码，这些密码可能会被破解。这一弱点促使了影子密码文件的开发。

*/etc/shadow* 文件不能被普通用户读取，因为它包含（加密的）密码和其他可能敏感的信息。以下是一些来自影子文件的示例：

```
daemon:*:17212:0:99999:7:::
...
sam:$6$6QKDnXEBlVofOhFC$iGGPk2h1160ERjIkI7GrHKPpcLFn1mL2hPDrhX4cXyYa8SbdrbxVt.h
nwZ4MK1fp2yGPIdvD8M8CxUdnItDSk1:18491:0:99999:7:::
```

*shadow* 文件的字段（用冒号分隔）如下：

+   登录名

+   加密密码（如果不是有效的密码字符串，则会阻止密码访问）

+   最后一次密码更改的日期（自 1970 年 1 月 1 日以来的天数）

+   用户被允许更改密码的天数（如果为空，则用户可以随时更改密码）

+   用户必须更改密码的天数（如果为空，则用户永远不必更改密码）

+   密码警告期（密码过期前的天数）

+   密码宽限期（用户过期后可以更改密码的天数）

+   帐号过期日期（自 1970 年 1 月 1 日以来的天数）

+   未使用的字段预留给未来使用

在构建用户活动的取证时间线时，最后一次密码更改日期可能非常重要。

加密的密码字段包含三个由美元符号（$）分隔的字段。这些字段分别是使用的加密算法、加密盐值（用来增加破解难度）和加密的密码字符串。加密算法包括：

| **1** | MD5 |
| --- | --- |
| **2a** | Blowfish |
| **5** | SHA-256 |
| **6** | SHA-512 |

请参阅 crypt(3) 手册页获取更多信息。

*/etc/group* 文件存储关于 Unix 组的信息，包括组成员列表。以下是一个典型 *group* 文件的部分摘录：

```
root:x:0:
daemon:x:1:
...
sudo:x:27:sam,fred,anne
```

组文件的字段（由冒号分隔）如下：

+   组名

+   密码（如果使用，则密码信息存储在 gshadow 文件中）

+   数字组 ID

+   以逗号分隔的成员列表

每个用户的默认组定义在 */etc/passwd* 文件中。*/etc/group* 文件可以提供额外的组配置。例如，注意 *sudo* 组列出了可以使用 `sudo` 程序的用户。

用户和组只是映射到数字的可读名称：用户 ID（UID）和组 ID（GID）。*passwd* 和 *group* 文件定义了名称到数字的分配。^(9) 对于特定的 UID 或 GID 数字，没有要求必须分配用户或组名称。为说明这一点，观察以下命令序列：

```
# touch example.txt
# chown 5555:6666 example.txt
# ls -l example.txt
-rw-r----- 1 5555 6666 0 5\. Mar 19:33 example.txt
#
```

在此示例中，文件是使用 `touch` 命令创建的。然后，使用 `chown` 命令将用户和组更改为在密码或组文件中未定义的数字值。从目录列表中可以看到，未知用户为 `5555`，未知组为 `6666`。从取证角度来看，具有未分配用户和组的文件很有趣，因为它们可能表示一个先前删除的用户/组或隐藏恶意活动的尝试。

如何找到没有分配 UID 或 GID 的文件？在活动系统中，`find` 命令具有 `-nouser` 和 `-nogroup` 选项，可用于扫描没有分配现有用户或组的文件。在事后检查的磁盘镜像中，取证软件可能具备识别此类文件的能力（例如使用 EnCase EnScript）。已识别的文件和目录可以进一步分析，以回答某些问题：

+   文件是如何以及为什么创建的？

+   原始用户和组发生了什么？

+   文件的时间戳是否有趣或相关？

+   这个 UID 或 GID 是否出现在任何日志中？

+   在运行的系统中，是否有任何进程具有相同的 UID 和 GID？

有几种方法可以创建和删除用户和组。系统管理员可以手动编辑 *passwd*、*shadow* 和 *group* 文件，添加或删除分配的 UID 或 GID。也可以使用 `useradd` 或 `groupadd` 等命令行工具。发行版可能还提供图形配置工具来添加用户和组。

当用户或组被创建或修改时，一些工具会制作密码、组、shadow 文件等的备份副本。备份副本的名称与原文件相同，只是末尾会加上连字符（-），如下所示：

+   */etc/passwd-*

+   */etc/shadow-*

+   */etc/gshadow-*

+   */etc/group-*

+   */etc/subuid-*

这些备份副本通常与原始文件相同。如果文件不同，则可能是手动修改的，或者使用了不支持该备份约定的其他工具。检查差异可能揭示之前已删除、添加或修改的用户。

*passwd* 文件包含了人类用户和系统用户。在分析人类用户活动时，了解二者的区别非常重要。*passwd* 和 *group* 中的数字 ID 字段可以帮助在法医调查中做出区分。以下列表描述了一些标准用户、组和分配的数字范围：

**0** root（LSB 必需）

**1** daemon（LSB 必需）

**2** bin（LSB 必需）

**0–100** 系统分配

**101–999** 应用程序分配

**1000–6000** 普通（人类）用户帐户

**65534** nobody

偏离这些标准的 UID 和 GID 范围对于法医调查员来说很有趣，因为它们可能表明用户和组的手动修改或非标准创建。

大多数 Linux 发行版在创建新用户时会分配 UID 从 1000 开始，并为默认组分配相同的 GID。然而，用户的 UID 和 GID 不必是相同的数字。如果用户的 UID 与 GID 不同，则意味着可能手动创建了一个额外的组。

创建新用户或组的操作可能会出现在 root 用户的 shell 历史记录中（例如 `useradd fred`），或者普通用户的 shell 历史记录中（例如 `sudo useradd fred`）。如果用户是通过图形界面工具创建的，则可能在日志中显示如下：

```
Aug 17 20:21:57 pc1 accounts-daemon[966]: request by system-bus-name::1.294
 [gnome-control-center pid:7908 uid:1000]: create user 'fred'
Aug 17 20:21:57 pc1 groupadd[10437]: group added to /etc/group: name=fred,
 GID=1002
Aug 17 20:21:57 pc1 groupadd[10437]: group added to /etc/gshadow: name=fred
Aug 17 20:21:57 pc1 groupadd[10437]: new group: name=fred, GID=1002
Aug 17 20:21:57 pc1 useradd[10441]: new user: name=fred, UID=1002, GID=1002,
 home=/home/fred, shell=/bin/bash
```

在这个例子中，GNOME 的 `gnome-control-center`（设置程序）请求 `accounts-daemon`（属于 AccountsService； *[`www.freedesktop.org/wiki/Software/AccountsService/`](https://www.freedesktop.org/wiki/Software/AccountsService/)*) 创建一个用户（`fred`）。这个 D-Bus 服务通过系统工具，如 `useradd` 或 `groupadd`，访问并配置本地用户帐户。AccountService 最初为 GNOME 开发，但可以被任何发行版使用。

删除用户仅意味着在 shadow、password 和 group 文件中定义的用户和 ID 记录已被删除。以下是从日志中找到的示例（从前面的示例中删除 `fred`）：

```
Aug 17 20:27:22 pc1 accounts-daemon[966]: request by system-bus-name::1.294
 [gnome-control-center pid:7908 uid:1000]: delete user 'fred' (1002)
Aug 17 20:27:22 pc1 userdel[10752]: delete user 'fred'
```

删除用户或组不会自动删除由这些用户拥有的文件。除非明确删除，否则这些文件仍然存在，并且会以已删除用户的原始数字 ID 显示。

一些取证程序或密码恢复工具能够尝试恢复存储在 shadow 文件中的加密密码。以下是 John the Ripper 从为用户 *sam* 提取的 */etc/shadow* 条目中恢复密码的示例：

```
# cat sam.txt
sam:$6$CxWwj5nHL9G9tsJZ$KCIUnMpd6v8W1fEu5sfXMo9/K5ZgjbX3ZSPFhthkf5DfWbyzGL3DxH
NkYBGs4eFJPvqw1NAEQcveD5rCZ18j7/:18746:0:99999:7:::
# john sam.txt
Created directory: /root/.john
Warning: detected hash type "sha512crypt", but the string is also recognized
as "sha512crypt-opencl"
...
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 128/128 AVX 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 8 OpenMP threads
...
Proceeding with wordlist:/usr/share/john/password.lst, rules:Wordlist
canada           (sam)
...
```

在这里，`john` 破解工具通过字典或基于词汇表的攻击发现密码是 *canada*。John the Ripper 会在运行它的用户的 *~/.john/* 目录中留下密码破解活动的痕迹，包括以前恢复的密码。

#### *提升的权限*

在 Linux 系统中，典型的用户帐户预计具有足够的权限来进行“正常工作”，但没有足够的权限去破坏系统、干扰其他用户或访问本应保密的文件。只有一个用户，即 root（UID 0），拥有执行所有操作的权限。若干机制允许普通用户提升权限，以执行某些授权任务。

传统的 Unix `su`（替代用户）命令允许以另一个用户或组的权限执行命令（如果未指定，则默认为 root）。`su` 命令的失败和成功使用记录会出现在系统日志中，如下所示：

```
Aug 20 09:00:13 pc1 su[29188]: pam_unix(su:auth): authentication failure;
 logname= uid=1000 euid=0 tty=pts/4 ruser=sam rhost= user=root
Aug 20 09:00:15 pc1 su[29188]: FAILED SU (to root) sam on pts/4
...
Aug 20 09:01:20 pc1 su[29214]: (to root) sam on pts/4
Aug 20 09:01:20 pc1 su[29214]: pam_unix(su:session): session opened for user
 root by (uid=1000)
```

默认情况下，所有用户都被允许使用 `su` 命令。有关更多信息，请参阅 su(1) 手册页。

`sudo` 命令比 `su` 提供了更精细的权限管理，并可以配置为仅允许某些用户执行特定命令。`sudo` 配置可以在 */etc/sudoers* 文件中找到，或在 */etc/sudoers.d/* 目录中的文件中。*sudo* 组中也可能包含授权用户的列表。

授权用户使用 `sudo` 命令的失败和成功记录如下所示：

```
Aug 20 09:21:22 pc1 sudo[18120]: pam_unix(sudo:auth): authentication failure;
logname=sam uid=1000 euid=0 tty=/dev/pts/0 ruser=sam rhost= user=sam
...
Aug 20 09:21:29 pc1 sudo[18120]:   sam : TTY=pts/0 ; PWD=/home/sam ; USER=
root ; COMMAND=/bin/mount /dev/sdb1 /mnt
Aug 20 09:21:29 pc1 sudo[18120]: pam_unix(sudo:session): session opened for
user root by sam(uid=0)
```

未经授权的用户（即那些不被视为“管理员”的用户）尝试使用 `sudo` 的操作也会出现在系统日志中：

```
Aug 20 09:24:19 pc1 sudo[18380]:   sam : user NOT in sudoers ; TTY=pts/0 ;
PWD=/home/sam ; USER=root ; COMMAND=/bin/ls
```

对 `sudo` 活动的搜索可以揭示有关系统被攻破或普通用户滥用的信息，包括尝试执行的特权命令。

当用户第一次运行 `sudo` 时，可能会出现警告信息或关于风险和责任的“讲解”：

```
$ sudo ls

We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for sam:
```

如果 sudo 配置为仅显示一次信息（默认），则会在 */var/db/sudo/lectured/* 目录中创建一个以用户命名的零长度文件。该文件的创建时间戳表示用户首次运行 `sudo` 命令的时间。有关更多信息，请参阅 sudo(8) 和 sudoers(5) 手册页。

另一种提升权限的方法是使用可执行文件上的 `setuid` 标志，表示该程序应以文件所有者的 UID 执行。使用此标志不会记录日志（尽管 `setuid` 程序本身可能会生成日志）。`setuid` 程序的 `ls -l` 权限信息中会有一个“s”：

```
$ ls -l /usr/bin/su
-rwsr-xr-x 1 root root 67552 23\. Jul 20:39 /usr/bin/su
```

在取证调查中，可以搜索所有 *setuid* 文件。特别是，*setuid* 文件如果不是任何官方发行版软件包的一部分，可能会很有趣；例如：

```
$ find /usr -perm -4000
/usr/bin/sudo
...
/usr/bin/passwd
...
/tmp/Uyo6Keid
...
```

在这个例子中，一个可疑的 *setuid* 文件被发现在 */tmp/* 目录下，应进一步检查。

所有 *setuid* 文件都对系统构成风险，如果它们包含漏洞，可能被利用。如果非特权用户能够利用 `setuid` 程序，他们可能会获得未授权访问权限，或者以其他用户（例如 root）身份执行任意代码。文件还可以设置 `setgid` 标志，导致程序以文件的组身份运行。

polkit（也称为 PolicyKit）框架提供的 API 也可以通过 D-Bus 提升权限。polkit 守护进程（`polkitd`）监听请求并采取适当的行动。授权操作通过位于 */etc/polkit-1/* 或 */usr/share/polkit-1/* 目录中的 *.rules* 和 *.policy* 文件进行配置。在做出授权决策时，`polkitd` 会检查这些规则和政策，并将活动记录到日志中，如下所示：

```
Aug 20 10:41:21 pc1 polkitd[373]: Operator of unix-process:102176:33910959 FAILED
to authenticate to gain authorization for action org.freedesktop.login1.reboot-
multiple-sessions for system-bus-name::1.2975 [<unknown>] (owned by unix-user:sam)
```

在此示例中，用户尝试重启系统，polkit 请求认证，用户未能提供认证。

`pkexec` 命令行工具是 polkit 软件包的一部分，其功能类似于 `sudo`。有关通过 D-Bus 使用 polkit 的更多信息，请参阅 polkit(8) 和 polkitd(8) 的手册页。

Linux 内核还提供了 *capabilities*，可以在更细粒度的级别上扩展和减少用户的权限。Systemd 具有在单元文件中定义能力的选项。有关更多信息，请参阅 capabilities(7) 和 systemd.unit(5) 的手册页。

#### *GNOME 密钥环*

GNOME 桌面环境有一个名为 GNOME 密钥环的凭证存储机制。用户可以创建多个密钥环，每个密钥环可以存储多个密码。前端工具与后台守护进程交互，后台守护进程创建和管理包含密码的文件。

密钥环文件的默认位置是 *~/.local/share/keyrings/*（以前是 *~/.gnome2/keyrings/*）。文件名与密钥环名称相同，空格被替换为下划线。如果存在多个密钥环并指定了默认密钥环，则名为 *default* 的文件将包含默认密钥环的名称。图 10-4 显示了 GNOME 密钥环的概述。

![Image](img/ch10fig04.jpg)

*图 10-4：GNOME 密钥环数据流*

在某些安装中，`pam_gnome_keyring` PAM 模块可能会使用一个密钥环进行登录。在这种情况下，登录密码与默认的`gnome-keyring`密码相同。如果在创建时没有为密钥环设置密码，则密钥环文件将以未加密的形式存储，密码和其他信息将以可读的纯文本文件格式显示。

**.keyring** 文件可以复制到另一个系统进行分析。解密后的密钥环文件从取证角度来看包含有趣的数据，包括密钥环的创建时间戳、每个密码条目的创建和修改时间戳，以及每个密码条目的描述和密码。

如果您没有密码，可以使用支持 GNOME Keyring 格式的恢复工具进行暴力破解尝试。如果您拥有解锁密钥环的密码，则有几种方法可以提取信息。

查看所有信息的最简单方法是为密钥环设置一个空白密码，这样生成的密钥环文件内容将以未加密的形式保存。提取信息的另一种方法是使用 `dump-keyring0` `-format`，该工具包含在 GNOME Keyring 源代码中，^(10)，如下面所示：

```
$ dump-keyring0-format ~/.local/share/keyrings/Launch_Codes.keyring
Password:
#version: 0.0 / crypto: 0 / hash: 0

[keyring]
display-name=Launch Codes
ctime=0
mtime=1583299936
lock-on-idle=false
lock-after=false
lock-timeout=0
x-hash-iterations=1953
x-salt=8/Ylw/XF+98=
x-num-items=1
x-crypto-size=128

[1]
item-type=2
display-name=WOPR
secret=topsecretpassword
ctime=1583300127
mtime=1583419166

[1:attribute0]
name=xdg:schema
type=0
value=org.gnome.keyring.Note
```

使用此方法，您可以查看密钥环和各个条目的信息。密码条目包含密码、创建时间和最后修改时间。

*Seahorse* 是 GNOME 桌面环境中用于管理密码和密钥的主要图形工具。Seahorse 可以创建和管理密码密钥环（通过 gnome-keyring-daemon），还可以创建和管理其他密钥，如 SSH 和 GNU 隐私保护（GPG）。对 PKCS11 证书的支持正在开发中，使用的是 *user.keystore* 文件。图 10-5 显示了 Seahorse 的截图。

![Image](img/ch10fig05.jpg)

*图 10-5：Seahorse 密码和密钥管理工具*

#### *KDE 钱包管理器*

KDE 桌面环境有一个名为 KWallet 的凭证存储机制，用户可以在其中存储多个密码和网页表单数据。钱包受一个独立的密码保护。集成了 KDE 的应用程序能够使用 KWallet 存储密码和其他敏感信息。

使用 KWallet 管理的钱包通过 `kwalletd` 守护进程操作，该守护进程由钱包管理器按需启动。钱包可以使用 Blowfish 算法或用户的 GPG 密钥进行加密。图 10-6 在下一页显示了 KDE 钱包系统的概览。

钱包文件的默认位置是 *~/.local/share/kwalletd/*，文件名与钱包相同。每个钱包有两个文件：一个是 **.kwl** 扩展名的文件，包含加密数据；另一个是 **.salt** 扩展名的文件，包含盐数据，用于加强对密码破解的防护。**.kwl** 文件有一个头部，用于确定钱包文件的版本和类型。

钱包文件的前 12 个字节始终相同，表示它是一个 KDE 钱包：

```
4B 57 41 4C 4C 45 54 0A 0D 00 0D 0A 00 01 02 00 KWALLET.........
```

第 13 和 14 个字节是主版本号和次版本号，第 15 和 16 个字节分别指定加密算法和哈希算法（更多信息请参见*[`github.com/KDE/kwallet/blob/master/src/runtime/kwalletd/backend/backendpersisthandler.cpp`](https://github.com/KDE/kwallet/blob/master/src/runtime/kwalletd/backend/backendpersisthandler.cpp)*）。如果**.kwl**文件的第 15 个字节是 0x02，则表示 GPG；如果第 15 个字节是 0x00 或 0x03，则表示 Blowfish 的版本。

![Image](img/ch10fig06.jpg)

*图 10-6：KWallet 数据流*

一些 Linux 发行版创建了一个名为*kdewallet*的默认钱包，用户可以使用前端工具如`kwallet-query`或`kwalletmanager5`创建和管理额外的钱包，如图 10-7 所示。

![Image](img/ch10fig07.jpg)

*图 10-7：KWallet 管理工具*

您可以将这些文件复制到另一台 Linux 机器上，使用相同的钱包管理工具进行分析。

如果密码恢复工具支持 KWallet 文件的 Blowfish 格式，可以尝试暴力破解密码。

在某些情况下，登录密码和 KWallet 密码可能是相同的——例如，当使用`pam_kwallet`时。如果使用 GPG，则 KWallet 密码与用户的 GPG 密钥密码相同。还可以检查`kwalletd5`或`kwalletmanager5`的日志，因为在使用钱包管理器时，有时会出现错误信息，提供与使用证据相关的时间戳。

#### *生物识别指纹身份验证*

最近版本的 Linux 桌面提供生物识别指纹身份验证功能，如果机器具有兼容的硬件。fprint 项目（*[`fprint.freedesktop.org/`](https://fprint.freedesktop.org/)*)为各种指纹识别设备提供 Linux 支持，可用于身份验证。

用户必须先注册指纹，才能使用它们。注册过程将指纹信息保存到文件中（每个手指一个文件）。这些文件位于*/var/lib/fprint/*目录中，如下所示：

```
$ sudo ls /var/lib/fprint/sam/synaptics/45823e114e26
1 2 7 8
```

该目录路径由用户名（`sam`）、指纹识别设备的制造商（`synaptics`）和 USB 设备编号或序列号（`45823e114e26`）构成。已注册手指的文件名保存为数字。每个手指的相关数字如下：

1    左拇指

2    左食指

3    左中指

4    左无名指

5    左小指

6    右拇指

7    右食指

8    右中指

9    右无名指

10    右小指

指纹对象的结构已在项目团队的网站上进行了文档化，其中包含了法医学检查的有用信息。

指纹文件包含有关指纹读取器、用户名、注册日期以及可能的扫描指纹数据的信息。根据指纹读取器硬件的不同，这些文件可能会有所不同。有些读取器会将指纹数据存储在设备本身，而只在文件中保存元数据。

一个 PAM 模块（*pam_fprintd*）和 PAM 配置文件（例如*gdm-fingerprint*）便于进行指纹扫描以实现认证。该 PAM 模块还会记录成功的指纹认证，如下所示：

```
Dec 26 20:59:33 pc1 gdm-fingerprint][6241]: pam_unix(gdm-fingerprint:session):
session opened for user sam(uid=1000) by (uid=0)
```

在这里，使用生物识别认证通过 GDM 登录到机器。

从取证角度来看，生物识别认证尤其有趣。它通过识别个人的身体特征来进行身份验证，而不是依赖可能被盗用或共享的密码。然而，生物识别认证也可能被迫（胁迫、敲诈、暴力或其他威胁）或在某人睡觉或失去知觉时“被盗”。其他使用指纹复制品在某些材料上的方法已被证明在一些指纹读取器上有效。^(11)

#### *GnuPG*

1991 年，Philip Zimmermann 创建了“相当好的隐私”（PGP），为公众提供了一种简单的强加密工具来保护文件和消息。最初它是免费的并且是开源的，但后来变成了商业产品。关于专利和商业化的担忧促使了 OpenPGP 标准的诞生，最初在 RFC 2440 中描述（目前是 RFC 4880 和 RFC 5581）。1999 年，一个独立的 OpenPGP 实现以 GNU Privacy Guard（GnuPG 或 GPG）的名义开发，这个软件项目至今仍在积极开发。

GPG 是一种流行的加密方式，广泛应用于电子邮件程序、办公程序、软件包完整性验证工具、密码管理器、^(12)以及其他需要可互操作加密的程序中。

大多数 Linux 发行版默认包含 GPG 软件，目的是验证软件包的签名。像 Seahorse 和 KGpg 这样的前端工具使得 Linux 用户轻松生成和管理 GPG 密钥。解密 GPG 加密文件是取证调查员常面临的挑战之一，同时也面临其他加密挑战。

`gpg`程序使用默认选项进行编译，但会查找系统范围的配置文件（*/etc/gnupg/gpgconf.conf*）以及用户配置文件的默认位置（*~/.gnupg/gpg.conf*）。图 10-8 提供了 GPG 的概述。

![Image](img/ch10fig08.jpg)

*图 10-8：GnuPG 数据流*

关键文件由属于用户的公钥和私钥对以及已添加到公钥环中的其他密钥组成。在较新的系统中，用户的公钥位于*~/.gnupg/pubring.kbx*（早期版本存储在*~/.gnupg/pubring.gpg*中）。

除了私钥外，检查添加到密钥环中的公钥也可能具有兴趣。这个文件可以在没有秘密密钥的情况下读取，并且可能包含取证相关的信息。例如，任何用户添加的公钥都会显示出来，并伴有创建日期、姓名、电子邮件地址和其他信息。

`gpg` 可执行文件没有指定使用哪个文件的选项，但可以通过设置 `GNUPGHOME` 环境变量指向 *.gnupg* 目录的副本（如果你已将文件移动到单独的分析机器上），如下面的示例所示：

```
$ GNUPGHOME=/evidence-extracted/home/sam/.gnupg gpg --list-public-keys
/home/sam/extract/.gnupg/pubring.kbx
------------------------------------
...
pub  rsa2048 2011-04-26 [SC]
   FCF986EA15E6E293A5644F10B4322F04D67658D8
   uid           [ unknown] FFmpeg release signing key <ffmpeg-devel@ffmpeg.org>
   sub   rsa2048 2011-04-26 [E]
```

其他用于列出或提取密钥和信息的 GPG 命令也可以以类似方式使用。更多详细信息请参阅 gpg(1) 手册页。

一些取证程序或密码恢复工具能够尝试恢复 GPG 私钥。John the Ripper 也支持暴力破解 GPG 加密文件。

### Linux 桌面取证

与对 Windows 或 Macintosh 计算机的取证检查类似，Linux 桌面系统对于取证调查员来说也具有重要意义。分析来自各种图形组件的数字痕迹可以帮助重建过去的活动和用户行为。本节内容重点介绍在图形化 Linux 系统中寻找有用的取证线索。

#### *桌面设置和配置*

如今大多数桌面系统使用数据库来存储配置数据。任何应用程序都可以使用这个数据库，配置设置可以在不同的程序之间共享。

##### GNOME 配置

基于 GNOME 3 和 GNOME 40^(13) 的桌面环境使用 GSettings API 存储设置和配置数据，GSettings API 进一步使用 dconf 配置系统。每当应用程序或桌面组件想要修改配置设置时，`dconf-service` 程序会通过 D-Bus 激活（为了性能考虑，读取设置时直接从文件中进行，而不通过 D-Bus）。Dconf 在概念上类似于 Windows 注册表，其中数据以层次树的形式存储，包含键和值。

像 GNOME 控制中心（请参见下一页的 图 10-9）或 GNOME Tweaks 这样的桌面配置工具会读取并写入设置到 dconf 系统中（`dconf-editor` 工具可以用来查看所有设置）。任何使用 glib 库构建的应用程序也能够利用 dconf 系统来存储配置信息。

![Image](img/ch10fig09.jpg)

*图 10-9：GNOME 控制中心*

由于查看 dconf 配置的典型工具（如 GNOME 控制中心、Gnome Tweaks、`gsettings`、`dconf-editor`）也都通过 D-Bus 在实时系统上操作，因此它们不适用于事后取证分析。因此，我们必须检查文件系统中存储配置数据的文件。所有偏离默认设置的 dconf 设置（即用户或应用程序所做的更改）都存储在一个单独的文件中：*~/.config/dconf/user*。

这个文件使用一种称为 GNOME 变体（gvdb）的二进制数据库格式。可以使用离线阅读器提取数据库，阅读器可以在这里找到：*[`github.com/chbarts/gvdb/`](https://github.com/chbarts/gvdb/)*。`reader`工具可以提取任何 gvdb 文件的内容，包括 GNOME 配置数据库；例如：

```
$ reader /home/sam/.config/dconf/user
/home/sam/.config/dconf/user
...
   /org/gnome/shell/favorite-apps
       ['org.gnome.Calendar.desktop', 'org.gnome.Music.desktop',
       'org.gnome.Photos.desktop', 'org.gnome.Nautilus.desktop',
       'org.gnome.Software.desktop', 'termite.desktop',
       'firefox.desktop'] ➊ 
...
   /org/gnome/cheese/camera
       'HD Webcam C525' ➋ 
...
   /org/gnome/desktop/background/picture-uri
       'file:///home/sam/Pictures/Webcam/2020-10-11-085405.jpg' ➌ 
...
   /org/blueman/plugins/recentconns/recent-connections
       [{'adapter': 'B4:6B:FC:56:BA:70',
       'address': '38:01:95:99:4E:31',
       'alias': '[Samsung] R3', 'icon': 'audio-card', 'name': 'Auto connect profiles',
       'uuid': '00000000-0000-0000-0000-000000000000', 'time': '1597938017.9869914',
       'device': '', 'mitem': ''}] ➍ 
...
   /org/gnome/epiphany/search-engines
       [('DuckDuckGo', 'https://duckduckgo.com/?q=%s&t=epiphany', '!ddg')] ➎ 
...
   /system/proxy/socks/port
       8008 ➏ 
...
   /system/proxy/socks/host
       'proxy.example.com' ➏ 
...
```

在这个示例中，我们看到各种可能在 dconf 数据库文件中找到的桌面配置信息。配置的层次结构可以在路径（*/org/gnome/. . .*）和下面的内容中看到。从这个示例中，从取证的角度看，比较有趣的配置包括：

+   GNOME 仪表盘上列出的最爱应用（通过点击活动显示的任务栏） ➊

+   被奶酪程序使用的网络摄像头（奶酪是一个网络摄像头照片应用：* [`wiki.gnome.org/Apps/Cheese`](https://wiki.gnome.org/Apps/Cheese) *）➋

+   桌面背景图片的文件位置（很可能是通过网络摄像头拍摄的）➌

+   最近的蓝牙设备，包括 MAC 地址、设备描述和时间戳 ➍

+   用户配置的默认搜索引擎（DuckDuckGo）在 Epiphany 网页浏览器中 ➎

+   用户定义的代理设置，包括协议（SOCKS）、TCP 端口号和代理主机 ➏

任何应用程序都可以通过 GSettings API 保存设置，并且它们将存储在 dconf 数据库文件中。除了 gvdb *user* 文件外，系统范围的 dconf 数据库也可以在*/etc/dconf/db/**中找到。用户定义的配置数据优先于系统配置或其他配置数据库（配置文件）。

保存的配置信息取决于应用程序开发者。如前面的示例所示，配置信息可以包括任何持久化信息，包括已打开文件的历史记录、书签、各种事件的时间戳、远程服务器和帐户名称、以前连接的设备、以前的日历通知，以及更多可能对取证调查有用的信息。有关更多信息，请参阅 dconf(7)手册页。

##### KDE 配置

KDE 桌面通过 KConfig 模块（KCMs）管理用户的配置更改。^(14)这些配置更改以纯文本文件形式存储在用户的*.config/*目录中，文件名通常以*rc*结尾。以下是一些示例：

```
$ ls .config/*rc
 .config/akregatorrc                .config/kmixrc
 .config/baloofilerc                .config/konsolerc
 .config/gtkrc                      .config/kscreenlockerrc
 .config/gwenviewrc                 .config/ksmserverrc
 .config/kactivitymanagerdrc        .config/ktimezonedrc
 .config/kactivitymanagerd-statsrc  .config/kwinrc
 .config/kateschemarc               .config/kwinrulesrc
 .config/kcminputrc                 .config/kxkbrc
...
```

在这个示例中，用户的 KDE/Plasma 配置更改与系统默认设置不同，这些更改被写入文件。这些文件可以来自与 KDE/Plasma 集成的任何应用程序。

这些文件采用了基本的*ini*风格格式，易于理解，如下所示：

```
$ cat ~/.config/kcookiejarrc
[Cookie Policy]
AcceptSessionCookies=true
CookieDomainAdvice=evil.com:Reject,evil.org:Reject
CookieGlobalAdvice=Accept
Cookies=true
RejectCrossDomainCookies=true
```

在这里，用户配置了一个个人的 Cookie 策略，其中明确拒绝来自某些网站的 Cookie。

##### 其他桌面配置

基于 GNOME 2 的桌面环境和应用程序使用 GConf 系统存储设置和配置信息。虽然 GConf 现在已被弃用，但一些应用程序仍可能使用它。配置信息以可读的 XML 格式文本文件存储。用户定义的 gconf 文件位于*~/.config/gconf/**，而系统范围的文件则位于*/etc/gconf/**。

其他桌面环境、窗口管理器和图形组件可能会将配置信息存储在文件或数据库中，这些文件位于用户 XDG 标准目录（*~/.config/*、*~/.local/share/*）或作为隐藏文件存储在主目录（*~/.**）中。仔细检查用户主目录可能会发现一些特定于桌面环境或未遵循 XDG 基础目录标准的组件的额外配置。

#### *桌面剪贴板数据*

早期的 X11 系统有非常简单的复制/粘贴机制，选中的文本可以通过中键粘贴到任何获得焦点的窗口中（选中的文本不会被保存）。《客户端间通信规范手册》（ICCCM）标准将其称为“PRIMARY”选择，并添加了一个额外的“CLIPBOARD”用于保存到内存中的文本，可以随时粘贴。

现代桌面环境引入了用于管理多个项目的剪贴板管理系统，这些项目在不同的登录会话中会持续保存。这些剪贴板管理器通常作为用户守护进程、插件或托盘小程序来实现，它们协调文本的复制和粘贴选择。

大多数桌面环境都有默认的剪贴板管理器，但用户可以选择安装其他独立的剪贴板管理器程序。本节描述了从最常见的剪贴板管理器中分析和提取剪贴板数据的方法。

KDE 桌面提供了 Klipper 剪贴板管理器。默认情况下，最后七个复制的项目会被记住并保存到文件*~/.local/share/klipper/history2.lst*中。该文件有一个简短的头部，剪贴板条目由`string`一词分隔。

该文件可以通过十六进制编辑器或支持 16 位字符宽度的文本编辑器查看。以下`sed`命令可以快速列出保存的剪贴板条目：

```
$ sed 's/s.t.r.i.n.g...../\n/g' .local/share/klipper/history2.lst
.P^ÃĞ5.18.2

apropos clipboard

xclip - command line interface to X selections

UUID=514d2d84-e25d-41dd-b013-36d3a4575c0a

MyUncrackableSuperPassword!1234

https://www.opensuse.org/searchPage
```

头部以版本号结尾，之后的行是复制到剪贴板中的项目历史记录。你也可以使用`strings`命令（可能需要`-el`参数），但列表将以未经格式化的方式显示。

使用 GNOME 桌面环境的发行版可能会有不同的剪贴板管理器。这些管理器作为插件或独立程序提供，有些发行版默认并不安装剪贴板管理器。以下示例展示了 GNOME 的剪贴板指示器扩展。默认历史记录大小为 15 个项目，这些项目存储在*~/.cache/clipboard-indicator@tudmotu.com/registry.txt*文件中，如下所示：

```
$ cat .cache/clipboard-indicator@tudmotu.com/registry.txt
[{"contents":"GNOME Shell Extension","favorite":false},{
"contents":"https://www.debian.org/","favorite":false},{
"contents":"https://www.gnome.org/gnome-3/","favorite":false}]
```

这是一个简单的 JSON 文件，可以使用任何文本编辑器打开。

Clipman 是 Xfce 面板的一个插件，它嵌入在桌面顶部或底部的面板栏中。默认情况下，10 个项目存储在*~/.cache/xfce4/clipman/textsrc*文件中。这些项目以可读的格式存储，每个项目之间用分号分隔：

```
$ cat .cache/xfce4/clipman/textsrc
[texts]
texts=1584351829;MyAWeSoMeUnCrackablePassword!1234;This paragraph has\nmultiple
lines\nof text to demonstrate\nhow it looks in the\nclipboard history;
```

`texts=`中的内容是一行文本。复制的多行文本由换行符`\n`分隔。

另一个例子是 Lubuntu，它默认使用 Qlipper 并将剪贴板数据存储在*~/.config/Qlipper/qlipper.ini*文件中。

Linux 有许多剪贴板管理器。每个发行版都会自行决定使用哪个管理器，你需要确定正在使用哪个剪贴板系统，并了解数据可能存储的位置。

#### *桌面垃圾桶*

计算机桌面隐喻还引入了垃圾桶的概念，允许用户轻松恢复被丢弃的文件。[freedesktop.org](http://freedesktop.org)为在 Linux 桌面系统上实现垃圾桶定义了一个标准。^(15) 该标准将文件移入垃圾桶称为*丢弃*，从文件系统中断开链接则称为*删除*。遵循这一标准的责任不仅仅在于发行版或桌面环境，主要还在于文件管理器。

桌面或文件管理器可以显示一个垃圾桶图标，用户可以在其中查看丢弃的文件，恢复它们，或者从文件系统中删除它们（即清空垃圾桶）。根据存储介质和文件系统的类型，从垃圾桶中删除的文件可能仍然可以通过取证工具恢复。

GNOME、KDE、Xfce 和 LXDE 的默认文件管理器分别是 Nautilus、Dolphin、Thunar 和 PCManFM。这些文件管理器（以及其他文件管理器）遵循垃圾桶规范。当文件和目录被移入垃圾桶时，它们会被移动到文件系统中的另一个位置，并保存恢复它们所需的信息。垃圾桶的典型位置是用户主目录下的*~/.local/share/Trash/*，该位置包含以下内容：

***files/*** 存放丢弃文件和目录的目录。除非整个目录被丢弃，否则*files/*目录是平面的，没有额外的结构。

***info/*** 一个包含**.trashinfo*文件的目录，每个已删除的文件或目录都有一个对应的文件。这些文件包含被丢弃项目的原始位置及其被移动到垃圾桶的时间戳。

***directorysizes*** 当一个目录被丢弃时，一些文件管理器会更新*directorysizes*文件，记录被丢弃目录的名称、大小以及移动的时间戳（Unix 纪元）。

***expunged/*** GNOME gvfs 可能会创建一个 expunged 目录，用于从垃圾桶中删除文件。这不是标准的一部分，且并不总是出现。

以下示例展示了一个典型的垃圾文件夹结构，其中包含一个被丢弃的文件（*helloworld.c*）和一个被丢弃的目录（*Secret_Docs/*）：

```
$ find .local/share/Trash/
.local/share/Trash/
.local/share/Trash/files
.local/share/Trash/files/Secret_Docs
.local/share/Trash/files/Secret_Docs/mypasswords.odt
.local/share/Trash/files/helloworld.c
.local/share/Trash/info
.local/share/Trash/info/Secret_Docs.trashinfo
.local/share/Trash/info/helloworld.c.trashinfo
.local/share/Trash/directorysizes
```

**.trashinfo**和*directorysizes*文件的内容是可读的纯文本。*directorysizes*文件为每个已删除的目录包含一行（除了**.trashinfo**文件）。

垃圾桶目录的内容不会保存其他元信息——只保存文件大小。*.trashinfo*和*directorysizes*文件内容如下：

```
$ cat .local/share/Trash/info/helloworld.c.trashinfo
[Trash Info]
Path=/home/sam/helloworld.c
DeletionDate=2020-03-16T15:55:04
$ cat .local/share/Trash/info/Secret_Docs.trashinfo
[Trash Info]
Path=/home/sam/Secret_Docs
DeletionDate=2020-03-16T21:14:14
$ cat .local/share/Trash/directorysizes
8293 1584389654463 Secret_Docs
```

垃圾桶文件夹（除了用户主目录中的垃圾桶）可以存在于可移动存储（如 USB 闪存盘）、挂载的网络共享和其他位置，使用*.Trash/*或*.Trash-UID/*目录（其中 UID 是用户的数字 ID），该目录位于挂载目录的顶部。垃圾桶规范不要求系统支持此功能，但许多文件管理器都支持。

在任何操作系统上分析垃圾桶文件夹是法医调查中的常规操作。当进行删除尝试时，会存在删除时间戳，并且会揭示出一个原始位置，可能会找到更多相关的文件。

#### *桌面书签和最近的文件*

在桌面上识别书签（有时称为“收藏夹”）和最近使用的项目是法医检查的典型部分。在 Linux 桌面上，书签和最近使用的文件，或称“最近的”，是通过相同的机制进行管理的。最近的文档也可以被认为是动态创建的书签。

*xbel*文件格式指的是 XML 书签交换语言（请参见*[`pyxml.sourceforge.net/topics/xbel/`](http://pyxml.sourceforge.net/topics/xbel/)*和*[`www.freedesktop.org/wiki/Specifications/desktop-bookmark-spec/`](https://www.freedesktop.org/wiki/Specifications/desktop-bookmark-spec/)*）。这些文件不仅限于办公文档和图片；它们还可能包含应用程序或文件管理器打开的其他文件（例如 zip 文件）。

在 Linux 系统中，书签和关于最近文件的信息可以在多个标准位置找到，文件扩展名为*.xbel*。例如，*.local/share/recently-used.xbel*和*.local/user-places.xbel*文件就存储在用户的主目录中。这些文件可能还有备份副本（**.bak**），其中包含之前书签的项目。

以下显示的是最近使用的文件中的单个条目（可能有多个条目）：

```
$ cat ~/.local/share/recently-used.xbel
  <bookmark href="file:///tmp/mozilla_sam0/Conference.pdf" added="2020-11-03T06
  :47:20.501705Z" modified="2020-11-03T06:47:20.501738Z" visited="2020-11-03T06
  :47:20.501708Z">
    <info>
      <metadata owner="http://freedesktop.org">
        <mime:mime-type type="application/pdf"/>
        <bookmark:applications>
          <bookmark:application name="Thunderbird" exec="&apos;thunderbird
          %u&apos;" modified="2020-11-03T06:47:20.501717Z" count="1"/>
        </bookmark:applications>
      </metadata>
    </info>
  </bookmark>
...
```

这里，*Conference.pdf*文件是由 Thunderbird 邮件客户端保存到临时位置的。文件类型和时间戳的信息也被保存。

这个示例展示了*user-places.xbel*文件中的一项条目：

```
$ cat ~/.local/user-places.xbel
 <bookmark href="file:///home/sam/KEEPOUT">
 <title>KEEPOUT</title>
 <info>
  <metadata owner="http://freedesktop.org">
   <bookmark:icon name="/usr/share/pixmaps/electron.png"/>
  </metadata>
  <metadata owner="http://www.kde.org">
   <ID>1609154297/4</ID>
  </metadata>
 </info>
</bookmark>
```

这里，文件夹*/home/sam/KEEPOUT*在 KDE 的 Dolphin 文件管理器中被书签（“添加到位置”）。时间戳指的是添加的日期或书签属性更改（如名称、图标等）的日期。

一些最近的文件数据存储在**.desktop**文件中，位于*.local/share/Recent Documents/*目录下；例如：

```
$ cat PFI_cover-front-FINAL.png.desktop
[Desktop Entry]
Icon=image-png
Name=PFI_cover-front-FINAL.png
Type=Link
URL[$e]=file:$HOME/publish/pfi-book/nostarch/COVER/PFI_cover-front-FINAL.png
X-KDE-LastOpenedWith=ristretto
```

这里，*PFI_cover-front-FINAL.png*图像文件（我的最后一本书的封面艺术）是最近由 Ristretto 应用程序打开的。这些桌面文件不包含时间戳，文件系统时间戳可能会指示创建日期。

上述书签方法是为了跨应用程序共享而设计的，但单个应用程序可能有自己的实现来存储书签和最近的文档。在取证检查中，已安装程序的列表可能会被分析，以寻找特定于应用程序的痕迹。这些痕迹通常存储在用户的 *.cache/* 目录中。

#### *桌面缩略图图像*

当 Linux 桌面环境开始流行时，图形应用程序开始开发各自管理缩略图图像（原始图像的缩小版本）的方法，以便快速预览。今天，这一标准由[freedesktop.org](http://freedesktop.org)规范化，并被大多数需要缩略图功能的现代应用程序使用。这意味着由一个应用程序创建的缩略图可以被另一个应用程序重用，因为它们都存储在同一个位置，并采用相同的格式。Linux 桌面缩略图的规范可以在 *[`www.freedesktop.org/wiki/Specifications/thumbnails/`](https://www.freedesktop.org/wiki/Specifications/thumbnails/)* 中找到。

缩略图通常存储在 *~/.cache/thumbnails/* 目录中的多个子目录中。三个可能的子目录用于存储缩略图图像：*large/*、*normal/* 和 *fail/*。这些目录包含不同尺寸（通常为 256×256 或 128×128）的缩略图图像，还包括创建缩略图时的失败尝试。

标准规定，所有的缩略图文件必须以 PNG 格式保存，并包含关于原始文件的元数据。缩略图图像中可能存储的元数据包括：

Thumb::URI    原始文件的 URI（必填）

Thumb::MTime    原始文件的修改时间（必填）

Thumb::Size    原始文件的大小

Thumb::Mimetype    文件的 MIME 类型

描述    关于缩略图内容的描述性文字

Software    创建缩略图的软件信息

Thumb::Image::Width    原始图像的宽度（像素）

Thumb::Image::Height    原始图像的高度（像素）

Thumb::Document::Pages    原始文档的页数

Thumb::Movie::Length    原始视频的时长（秒）

date:create    缩略图文件的创建时间戳

date:modify    缩略图文件的修改日期（如果原始文件发生变化，则更新）

缩略图文件名是通过原始文件位置的 URI 的 MD5 哈希值生成的（不包含尾随换行符）。例如，如果原始文件的 URI 是 *file:///home/username/cats.jpg*，则缩略图文件名将是 *14993c875146cb2df70672a60447ea31.png*。

失败的缩略图按失败的程序进行排序，并包含一个空白的 PNG 文件，尽可能包含关于原始文件的所有元数据。保存在失败目录中的 PNG 文件的时间戳是失败的时间。

以下示例展示了在用户的 *~/.cache/* 目录中找到的缩略图：

```
$ ls .cache/thumbnails/normal/
a13c5980c0774f2a19bc68716c63c3d0.png d02efb099973698e2bc7364cb37bd5f4.png
a26075bbbc1eec31ae2e152eb9864976.png d677a23a98437d33c7a7fb5cddf0a5b0.png
a3afe6c3e7e614d06093ce4c71cf5a43.png dc1455eab0c0e77bf2b2041fe99b960e.png
a4a457a6738615c9bfe80dafc8abb17d.png e06e9ae1a831b3903d9a368ddd653778.png
...
```

使用任何 PNG 分析工具可以揭示这些文件中的更多信息。

在这个例子中，使用 ImageMagick 的`identify`工具从其中一个文件中提取元数据：

```
$ identify -verbose a13c5980c0774f2a19bc68716c63c3d0.png
Image: a13c5980c0774f2a19bc68716c63c3d0.png
  Format: PNG (Portable Network Graphics)
...
 Properties:
    date:create: 2020-03-15T08:27:17+00:00
    date:modify: 2020-03-15T08:27:17+00:00
...
    Software: KDE Thumbnail Generator Images (GIF, PNG, BMP, ...)
    Thumb::Mimetype: image/png
    Thumb::MTime: 1465579499
    Thumb::Size: 750162
    Thumb::URI: file:///tmp/Practical_Forensic_Imaging.png
...
```

前两个时间戳分别指的是缩略图 PNG 的创建时间和最后修改时间（如果原始图像更改，它将被更新）。`Thumb::MTime:`属性是原始文件的最后修改时间戳（以 Unix 纪元格式表示）。`Software:`属性是创建缩略图的程序。在这种情况下，它来自 KDE，使用的是 Dolphin 文件管理器。`Thumb::Mimetype:`、`Thumb::Size:`和`Thumb::URI:`属性显示了原始文件的图像类型、大小和位置。缩略图是原始图像的较小版本，如图 10-10 所示。

![Image](img/ch10fig10.jpg)

*图 10-10：恢复的缩略图示例*

缩略图文件的删除是尽最大努力进行的。一些文件管理器可能会在原始文件删除时删除缩略图。一些“清理”工具可以删除缓存文件。用户也可以手动删除缓存。

一些较老的应用程序可能会使用*~/.thumbnails*目录来存储缩略图文件。

#### *完全集成的桌面应用程序*

在 X11 窗口管理器的早期，标准小部件库被用来创建跨窗口统一的外观（相同的按钮样式、滚动条样式等）。桌面环境将这种统一的“外观和感觉”进一步扩展，包括紧密集成的应用程序。这些应用程序不仅外观相似，而且行为也相似，能够相互通信（通常通过 D-Bus），并可以共享配置。这些应用程序有时被称为*完全集成*应用程序，并作为桌面环境项目的一部分进行开发。以下是一些项目团队的示例以及他们的应用程序列表链接：

+   GNOME: *[`wiki.gnome.org/Apps/`](https://wiki.gnome.org/Apps/)*

+   KDE: *[`apps.kde.org/`](https://apps.kde.org/)*

+   Xfce: *[`gitlab.xfce.org/apps/`](https://gitlab.xfce.org/apps/)*

+   LXDE: *[`wiki.lxde.org/`](https://wiki.lxde.org/)*

典型的集成应用程序通常是文本编辑器、图像和文档查看器、文件管理器、音乐和视频播放器等。

其他集成的“配件”应用程序可能包括屏幕截图工具、配置工具、热键管理器、主题等。更大的桌面环境甚至可能包括自己的电子邮件客户端（例如 GNOME 的 Evolution 或 KDE 的 Kmail）或 Web 浏览器。像 Firefox、Thunderbird、LibreOffice 等大型跨平台应用程序可能以更通用的方式进行集成（使用 D-Bus 进行通信）。

完全集成的应用程序从取证的角度来看很有趣，因为它们倾向于将数据记录、共享、配置和存储在同一个地方，并以相同的方式进行处理，从而使得取证分析更加容易。

小部件库和集成应用程序的使用不是强制性的。可以在单一系统上同时安装 GNOME、KDE、Xfce 和 LXDE 应用程序，甚至可以与各种小部件库（如 Athena 或 Motif 等）一起使用旧的非集成 X11 应用程序。

##### 文件管理器

文件管理器应用程序对取证人员尤为重要。文件管理器在本地系统中的作用类似于网页浏览器在互联网中的作用。对文件管理器的分析可以帮助了解本地机器上文件的管理方式。

Linux 上有数十种文件管理器，既有图形界面的，也有基于文本控制台的。每种桌面环境都偏爱某个特定的文件管理器，发行版可能选择某个文件管理器作为默认管理器。

文件管理器通常是 Linux 爱好者的个人偏好，用户可能会安装自己喜欢的文件管理器，覆盖掉默认的发行版设置。

总体而言，这些文件管理器不局限于特定的桌面环境，且可以在任何环境中使用（前提是已安装所需的库）。

不同桌面环境的默认文件管理器（KDE Dolphin、GNOME Nautilus、XFCE Thunar 和 LXDE PCManFM）可能会被其他集成良好的应用程序调用，并留下过去活动的遗留物，这些在调查中可能很有用。

从取证的角度来看，文件管理器和其他集成应用程序的分析可能包括以下内容：

+   最近打开的文档

+   垃圾桶/回收站

+   图像缩略图

+   搜索索引和查询

+   已书签的文件和目录

+   标签和文件管理器元数据

+   挂载设备和网络共享的历史

+   配置和插件

这些遗留物可能会在集成良好的应用程序之间创建并共享，帮助重建过去的活动。每个应用程序可能存储不同的信息，并且存储在不同的位置。在进行取证分析时，需要寻找每个应用程序使用的缓存和数据文件。

#### *其他桌面取证遗留物*

大多数 Linux 系统上可以找到各种其他桌面遗留物，这里进行了描述。

##### 截图

Linux 桌面上的截图功能可以作为扩展、某个特定环境的捆绑工具或独立应用程序来实现。截图工具通常将截图保存到剪贴板或文件系统。

当保存到文件系统时，截图通常会保存在用户的 *~/Pictures/* 目录下，命名约定通常包括截图时的时间戳，示例如下：

```
$ ls -l /home/sam/Pictures/
total 3040
-rw-r----- 1 sam sam 1679862 Oct 11 09:18 'Screenshot from 2020-10-11 09-18-47.png'
-rw-r----- 1 sam sam 1426161 Oct 11 09:20 'Screenshot from 2020-10-11 09-20-52.png'
```

Wayland 的安全架构阻止了基于 X11 的截图程序按预期工作，但替代工具可与各种 Wayland 合成器兼容使用。

##### 桌面搜索

桌面搜索引擎是寻找法医证据的一个有趣地方。在这里，我们并不是在寻找搜索的关键词（它们通常不会被保存），而是查找包含文件名和其他数据的搜索索引。大多数发行版都包含本地搜索引擎，它们可以对文件名或文件内容进行索引。

##### GNOME 桌面搜索

GNOME 的本地搜索引擎叫做 Tracker，它使用被称为矿工（Miners）的守护进程来对文件系统进行索引，并提取 Tracker 数据库的元数据。Tracker 使用基于 SQLite 的 SPARQL 数据库。这些数据库文件可以在 *.cache/tracker/* 或 *.cache/tracker3/* 目录中找到。

更新版本的 Tracker 将数据库分为多个文件，每个文件对应一个搜索矿工（例如图片、文档、文件系统等）。这些数据库文件（**.db**）可以使用 `sqlite` 命令进行导出，并以文本形式查看或导入 SQLite 法医工具进行分析。例如，以下是使用 `sqlite` 命令导出 Tracker 数据库的示例：

```
$ sqlite3 ~/.cache/tracker3/files/http%3A%2F%2Ftracker.api.gnome.org%2Fontology
%2Fv3%2Ftracker%23FileSystem.db .dump
...
INSERT INTO "nfo:FileDataObject" VALUES(100086,1602069522,NULL,275303,NULL,
'Fintech_Forensics_Nikkel.pdf',NULL,NULL,1593928895,'9f3e4118b613f560ccdebc
ee36846f09695c584997fa626eb72d556f8470697f');
...
INSERT INTO "nie:DataObject" VALUES(100086,'file:///home/sam/Downloads/
Fintech_Forensics_Nikkel.pdf', 275303,NULL,NULL,100081);
...
```

在这个示例中，文件系统中的一个文件由两行表示（通过记录号 `100086` 连接）。有一个路径和文件名（`file:///home/sam/` `Downloads/Fintech_Forensics_Nikkel.pdf`），文件大小（`275303`），文件创建时间戳（`1593928895`），以及文件加入数据库的时间戳（`1602069522`）。

这些数据库可能包含法医镜像中通常找不到的额外信息，可能是关于已删除文件的信息。

##### KDE 桌面搜索

KDE 有两个本地搜索引擎：一个用于本地文件系统，称为 Baloo；另一个用于联系人、日历和电子邮件，内置于 Akonadi 中，这是 KDE 的个人信息管理（PIM）框架。

Baloo 数据库是一个位于用户主目录中的单一文件（*~/.local/share/baloo/index*），如图所示：

```
$ ls -lh ~/.local/share/baloo/
total 13G
-rw-r----- 1 sam sam 13G 4\. Okt 19:07 index
-rw-r----- 1 sam sam 8.0K 11\. Dez 10:48 index-lock
```

随着时间的推移，索引的大小可能会变得很大，因为 Baloo 似乎会摄取大量的内容数据。截止目前，没有工具可以在单独的分析机器上进行离线法医分析 Baloo 索引文件。分析可以使用 `strings`、十六进制编辑器和法医切割工具。还有几个 Baloo 工具可以用于从正在运行的系统中搜索和提取数据。

KDE 的另一个索引活动是通过 Akonadi 完成的。这个框架存储并索引电子邮件、联系人、日历条目、便签和 KDE Kontact PIM 套件中的其他信息。数据本身存储在 MySQL 数据库中，搜索索引使用 Xapian 数据库文件（**.glass**）。所有内容都位于用户的主目录中（*~/.local/share/akonadi/*）。

```
$ ls ~/.local/share/akonadi/
Akonadi.error db_data db_misc file_db_data mysql.conf search_db
socket-localhost.localdomain-default
$ ls ~/.local/share/akonadi/search_db/
calendars collections contacts email emailContacts notes
$ ls ~/.local/share/akonadi/search_db/email
docdata.glass flintlock iamglass postlist.glass termlist.glass
```

这个示例展示了 Akonadi 目录结构的部分内容。*/search_db/* 目录包含每个数据类别的 Xapian 数据库。其他目录则包含 MySQL 数据库，存储实际数据。可以使用标准的 MySQL 和 Xapian 工具提取数据库内容。

##### 其他搜索索引

Xfce 桌面环境使用 Catfish 搜索工具。Catfish 不会索引文件，而是按需搜索文件。

一个名为 *mlocate* 的系统范围搜索包用于索引文件名。一些发行版可能默认安装了它（例如 Ubuntu）。更新数据库的工具会定期从 cron 或通过 systemd 定时器运行。该索引仅包含文件和目录名称，而不包含内容。配置文件是 */etc/updatedb.conf*，数据库位于 */var/lib/mlocate/mlocate.db*。mlocate.db(5) 手册页描述了数据库格式。该数据库包含每个目录的最后修改/变更时间戳，还列出了该目录所属的文件（但单个文件没有时间戳）。一个用于转储此数据库的工具可以在这里找到： *[`github.com/halpomeranz/dfis/blob/master/mlocate-time/`](https://github.com/halpomeranz/dfis/blob/master/mlocate-time/)*。

本节描述的搜索数据库可能包含已删除文件的证据、文件的先前时间戳，甚至可能包含在取证调查中有用的文档和文件内容。

### 用户网络访问

本节描述了通过网络访问 Linux 系统的方式。远程访问可以从两个角度来理解：用户从 Linux 系统发起连接到远程系统，以及 Linux 系统接受来自远程系统用户的连接。远程访问通常以远程 shell 或远程桌面的形式存在。

网络共享和云访问从最终用户或客户端的角度进行考虑。客户端活动的本地取证分析有涉及，尽管本书不涉及网络服务器应用程序的分析。

#### *安全 Shell 访问*

对 Unix 机器的远程访问始于使用模拟电话调制解调器将物理终端连接到远程系统的串行端口（tty）。一旦机器连接到互联网，像 telnet 和 rlogin 这样的协议就被创建出来，它们通过 TCP/IP 访问远程系统上的伪终端（pty 或 pts）。这些早期的协议安全性较差，SSH 被开发出来作为一个安全的替代方案，它使用加密身份验证和保护。今天，OpenSSH (*[`www.openssh.com/`](https://www.openssh.com/)*) 已成为安全远程访问的事实标准。

配有 SSH 服务器（默认 TCP 端口 22）的机器直接暴露在互联网上，将会遭遇持续的扫描、探测和暴力破解尝试，这些都将在日志中可见。在取证检查中，必须将来自互联网的随机机会性“噪声”与正在调查的目标攻击区分开。

图 10-11 提供了 OpenSSH 客户端的基本概览图。

![图片](img/ch10fig11.jpg)

*图 10-11：SSH 客户端概览*

`ssh` 客户端用于访问 shell 或向远程机器发送命令，`scp` 客户端用于复制文件（基于 BSD 的`rcp`），而`sftp` 客户端也以交互方式复制文件，类似于`ftp`。这三个客户端程序使用相同的配置文件和密钥，这些文件存储在用户的*~/.ssh/*目录中。

SSH 客户端可以使用密码、密钥文件或其他安全密钥提供者（例如智能卡）来对远程机器进行身份验证。默认情况下，密钥文件（如果使用）是通过`ssh-keygen`工具手动创建的，并存储在以*id_**开头的文件中。这些文件的命名基于所使用的算法，公钥文件以**.pub*扩展名结尾。

私钥文件可以使用密码短语进行加密，也可以以明文存储（通常用于自动化远程系统管理任务）。检查密钥文件是否加密的最简单方法是尝试更改密码（使用`ssh-keygen -p`）。如果提示`Enter old` `passphrase:`，说明它已加密。如果提示`Enter new passphrase` `(empty for no passphrase):`，说明它以明文存储。也可以尝试暴力破解加密的 SSH 密钥文件。

在调查中，搜索整个系统以查找可能没有加密的 SSH 密钥文件是很有用的。有时系统用户是为了备份或运行自动化的系统管理工具（例如 Ansible 或 Nagios）而创建的。无论密钥是否加密，SSH 私钥的头部和尾部都是相同的，以下示例可用于在取证工具中创建搜索字符串：^(17)

```
-----BEGIN OPENSSH PRIVATE KEY-----
...
-----END OPENSSH PRIVATE KEY-----
```

公钥文件以一个注释字段结尾，这个字段可能很有趣。它可能包含用户名、电子邮件地址、主机名或与密钥相关的其他描述信息。这个公钥可以通过*authorized_keys*密钥文件提供身份验证。以下是一个公钥的示例：

```
ssh-rsa AAAAB3NzaC1yc2EAAA ... /uzXGy1Wf172aUzlpvV3mHws= sam@example.com
```

请注意，公钥字符串在注释区域包含了用户的电子邮件地址。默认情况下，SSH 客户端不会在本地记录任何内容，因此重建过去的 SSH 活动可能会很困难。从取证角度来看，*.ssh/known_hosts* 文件是很有趣的，因为它包含了过去访问过的主机列表。当建立 SSH 连接时，新的主机会自动添加到该列表中。*.ssh/known_hosts* 文件包含主机名和/或 IP 地址、使用的加密算法以及远程机器的公钥。这个列表可以用来识别与调查可能相关的其他机器、主机名、域名和 IP 地址。

公钥信息也很有趣，因为它可以与外部收集的情报数据进行关联，比如 SSH 公钥扫描（例如 Shodan），并且有可能识别出使用相同密钥的其他主机（例如重用或复制的虚拟机）。以下是来自*.ssh/known_hosts*文件的示例行：

```
sdf.lonestar.org,205.166.94.16 ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJJk3a190w/1
TZkzVKORvz/kwyKmFY144lVeDFm80p17
```

另一个可以查找过去安全外壳客户端活动痕迹的地方是用户的 shell 历史记录。这些历史文件可以搜索 `ssh`、`scp` 或 `sftp` 命令。

即使 SSH 客户端默认不记录活动，仍可能会有日志条目指示之前的使用情况。例如，当客户端脚本或程序失败（或成功）时，可能会找到 SSH 连接尝试的证据。

SSH 客户端配置可以在多个位置找到：*/etc/ssh/ssh_config*、*/etc/ssh/ssh_config.d/* 和 *~/.ssh/config*，尽管其中一些是可选的。在这里，添加的自定义配置可能指向其他基础设施（例如，Host、Match 和 ProxyJump 命令）。此外，可能会暴露端口转发和代理使用情况（例如，RemoteForward、ProxyCommand 和 Tunnel）。SSH 提供了高度灵活的端口转发和代理功能，可以用于绕过防火墙规则和现有的外围安全系统。远程主机、远程用户名、端口转发和代理的证据可能会在配置文件中或通过 shell 历史中的命令找到。

在取证检查中，检查其他（非 OpenSSH）程序与 SSH 的交互（例如，密码管理器或代理）或 SSH 的替代实现（例如 PuTTY）。SSH 代理将提供密钥认证，OpenSSH 默认包括这一功能，但也可以使用其他代理。前面已经描述过一些替代代理或密码管理器（如 GNOME 密钥链、GPG 或 KDEWallet）。可以搜索 `SSH_AUTH_SOCK` 变量设置的存在，这表示使用了替代代理进行 SSH 认证。

文件复制程序 `scp` 和 `sftp` 经常作为较大应用程序（如办公套件、文件管理器等）的后台，用于与远程服务器交换文件。还有一个额外的软件包叫做 sshfs，它用于创建一个远程 sftp 登录的 FUSE 挂载文件系统。

请参考 ssh(1)、scp(1)、sftp(1)、ssh-keygen(1) 和 ssh_config(5) 手册页，了解有关安全外壳客户端的详细信息。

#### *远程桌面访问*

对于服务器环境来说，能够复制文件并获取远程 shell 通常足够满足用户（尤其是管理员）的需求，而 SSH 足以满足这一需求。但对于桌面环境来说，远程图形桌面是可能的，并且通常是所需的。

传统的 Unix 和 Linux 机器不需要远程桌面软件，因为远程访问桌面已经内置在 X11 协议中。此功能要求本地和远程机器都运行 X11，但这并不总是成立（例如，Windows 或 Mac 客户端访问远程 Linux 桌面）。这促使了远程桌面的使用。

虚拟网络计算（VNC）是 Linux 最流行的远程桌面客户端。当 Linux 桌面安装并运行 VNC 服务器时，VNC 服务器通常会监听 TCP 端口 5900。

Wayland 是在更注重安全性的基础上开发的，防止客户端窗口互相访问。因此，大多数基于 X11 的远程访问软件在 Wayland 桌面上无法工作（X11 屏幕截图工具或热键管理器也无法工作）。因此，Wayland 桌面必须将远程桌面功能构建到合成器中，或使用其他方法来访问桌面。

VNC 服务器的一个问题是日志记录不良。在某些情况下，可能没有日志记录远程桌面连接。在其他情况下，连接可能被记录，但没有 IP 地址。以下是来自 Ubuntu 机器的一个示例：

```
Dec 29 10:52:43 pc1 vino-server[371755]: 29/12/2020 10:52:43 [IPv4] Got connection from
 pc2.example.com
...
Dec 29 10:53:12 pc1 vino-server[371755]: 29/12/2020 10:53:12 Client pc2.example.com gone
```

在这里，VNC 连接已连接到 `vino-server` 守护进程，并随后终止。通过反向 DNS 查找记录了一个主机名，但没有记录 IP 地址。

**注意**

*如果某个人或组织为源 IP 范围（**.in-addr .arpa *区域）运行自己的 DNS，他们可以伪造或欺骗任何他们想要的 DNS 反向查找，从而导致日志信息失真。永远不要完全相信来自反向 DNS 查找的主机名。*

还有其他用于远程桌面访问的客户端协议。远程桌面协议（RDP）在 Windows 环境中很流行，并且在 Linux 上也有一些支持。Spice 协议主要为 Linux 桌面开发，具有 TLS 加密、USB 端口重定向、音频和智能卡支持等功能。许多视频会议应用程序（例如 Jitsi、Zoom、Microsoft Teams 和 Skype）提供屏幕共享功能，用于支持和演示。

许多企业环境正在实施虚拟桌面环境（VDE），作为硬件桌面或笔记本系统的替代方案。VDE 是在云中运行的完整桌面环境。类似于虚拟服务器，它是一个虚拟桌面 PC，可以通过远程桌面访问方法访问。

#### *网络共享与云服务*

网络挂载文件系统（也称为网络共享）可以在内核中管理，也可以通过 FUSE 在用户空间中管理。如果为系统范围的使用而挂载，这些网络文件系统可能会与本地硬盘一起在 */etc/fstab* 文件中进行配置。网络文件系统也可以通过命令行手动挂载，相关证据可能出现在 Shell 历史记录中。挂载的证据也可能出现在日志中。

网络文件系统（NFS）是传统的 Unix 协议，由 Sun Microsystems 开发，用于将远程文件系统挂载到本地机器上。NFS 共享像普通驱动器一样挂载，但在 fstab 条目的第一个字段前会加上主机名（例如 `hostname.example.com:/home`）。

与其他网络文件系统相比，NFS 更为复杂，需要多个协议和 RPC 服务（如 mountd）、用于管理锁定、身份验证、共享导出等的进程。NFS 通常在企业环境中使用，极少出现在消费者的家庭环境中。有关更多信息，请参阅 nfs(5) 手册页。支持的协议在近十个不同的 RFC 中有所定义。

公共互联网文件系统（CIFS）和/或服务器消息块（SMB）最初由 IBM 开发，后来由 Microsoft 开发，用于在本地计算机上挂载远程网络文件系统。Linux 在内核中实现了客户端，挂载可以作为 */etc/fstab* 中的一个条目（类似于 NFS）。最常见的服务器端实现是 Samba，它为其他 SMB 客户端提供网络共享。有关详细信息，请参阅 mount.smb(3) 手册页。

Webdav 是一种基于 Web 的规范，用于通过 HTTP 协议挂载共享。Linux 下的文件系统实现称为 davfs。Webdav 在挂载云服务（如 NextCloud）时非常流行。Webdav 协议的变种包括 caldav 和 carddav，用于访问远程日历和联系人数据库。有关挂载 Webdav 共享的更多信息，请参见 mount.davfs(8) 手册页。

FUSE 允许在不需要内核实现的情况下挂载文件系统。FUSE 文件系统还允许非特权用户挂载文件系统（例如 USB 存储）。FUSE 可以为访问文件系统中任意数据集创建文件系统抽象（如远程 FTP 服务器、本地归档文件或包含数据的特殊硬件设备）。

在桌面计算机上，可以使用桌面环境提供的 GUI 工具配置各种云帐户。GNOME 提供 GOA（GNOME 在线帐户）来配置云帐户。图 10-12 显示了 GOA 配置面板。

![Image](img/ch10fig12.jpg)

*图 10-12：GNOME 在线帐户面板*

用户可以添加和配置各种商业和开源云服务。

配置的帐户可以在用户的主目录中的 *~/.config/goa-1.0/accounts.conf* 文件中找到。以下展示了两个配置的云帐户示例：

```
   $ cat ~/.config/goa-1.0/accounts.conf
➊ [Account account_1581875544_0]
➋ Provider=exchange
   Identity=sam
   PresentationIdentity=sam@example.com
   MailEnabled=true
   CalendarEnabled=true
   ContactsEnabled=true
   Host=example.com
   AcceptSslErrors=false

➌ [Account account_1581875887_1]
➍ Provider=imap_smtp
   Identity=sam@example.com
   PresentationIdentity=sam@example.com
   Enabled=true
   EmailAddress=sam@example.com
   Name=Samantha Samuel
   ImapHost=example.com
   ImapUserName=sam
   ImapUseSsl=false
   ImapUseTls=true
   ImapAcceptSslErrors=false
   SmtpHost=example.com
   SmtpUseAuth=true
   SmtpUserName=sam
   SmtpAuthLogin=false
   SmtpAuthPlain=true
   SmtpUseSsl=false
   SmtpUseTls=true
   SmtpAcceptSslErrors=false
```

这里配置了 Microsoft Exchange ➋ 和 Imap ➍ 帐户。文件中 ➊ 和 ➌ 的帐户标识符各自包含一个数字时间戳，指示帐户条目的创建时间。密码存储在 GNOME 密钥环中。

可以在此处找到可能的 GOA 帐户部分列表： *[`gitlab.gnome.org/GNOME/gnome-online-accounts/raw/master/doc/goa-sections.txt`](https://gitlab.gnome.org/GNOME/gnome-online-accounts/raw/master/doc/goa-sections.txt)*。

KDE 将云帐户信息存储在用户的 *~/.config/libaccounts-glib/* 目录中。这是一个 SQLite 3 数据库，可以按以下方式访问（转储）：

```
$ sqlite3 ~/.config/libaccounts-glib/accounts.db .dump
...
INSERT INTO Accounts VALUES(1,'sam','nextcloud',1);
...
INSERT INTO Settings VALUES(1,0,'dav/storagePath','s','''/remote.php/dav/files/sam''');
INSERT INTO Settings VALUES(1,0,'dav/contactsPath','s','''/remote.php/dav/addressbooks/users/sam
''');
INSERT INTO Settings VALUES(1,0,'dav/host','s','''example.com''');
INSERT INTO Settings VALUES(1,0,'auth/mechanism','s','''password''');
INSERT INTO Settings VALUES(1,0,'username','s','''sam''');
INSERT INTO Settings VALUES(1,0,'name','s','''sam''');
INSERT INTO Settings VALUES(1,0,'CredentialsId','u','1');
INSERT INTO Settings VALUES(1,0,'server','s','''https://example.com/cloud/''');
...
```

这表明为用户 `sam` 配置了一个 NextCloud 帐户。密码存储在 KDE 钱包中，并由 libaccounts 客户端请求。

在某些情况下，Linux 系统可能已安装用于访问云资源的“胖客户端”软件。这可以是免费的开源软件，如 NextCloud 客户端，或是专有客户端软件，如 Microsoft Teams。

能够重建对云服务的访问可以支持调查，并可能导致恢复存储在远程服务器上的其他证据。

### *摘要*

本章对于来自 Windows 或 Mac 取证背景的读者来说，可能是最熟悉的。这里涉及的几乎所有用户和桌面相关的文档都在概念上类似。你现在应该知道如何查找和分析用户凭据和密码的位置，以及如何存储指纹扫描数据。你还探索了窗口和桌面系统及其提供的文档。你应该具备了在桌面上重建用户活动、远程访问和云连接的坚实基础。
