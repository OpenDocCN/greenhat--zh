## 第四章 无线网络简易设置

![无线网络简易设置](img/httpatomoreillycomsourcenostarchimages2127149.png.jpg)

很容易说，在 BSD 系统，尤其是 OpenBSD 中，不需要“简化无线网络设置”，因为它本身就已经很简单。让无线网络运行并不比让有线网络运行更复杂，但确实有一些问题是因为我们处理的是无线电波而不是电缆。我们将在继续讲解创建可用设置的实用步骤之前，简要探讨一些问题。

一旦我们掌握了设置无线网络的基本方法，我们将转向一些选项，使你的无线网络更加有趣且更难以被破解。

## IEEE 802.11 背景介绍

设置任何网络接口，原则上是一个两步过程：首先建立连接，然后配置接口以支持 TCP/IP 流量。

对于有线以太网类型的接口，建立连接通常只是插入电缆并看到连接指示灯亮起。然而，一些接口需要额外的步骤。例如，拨号连接的网络设置需要电话步骤，比如拨打一个号码以获取载波信号。

对于 IEEE 802.11 风格的无线网络，获取载波信号涉及底层的几个步骤。首先，你需要在分配的频谱中选择正确的频道。一旦找到信号，你需要设置一些链路级网络识别参数。最后，如果你想连接的站点使用某种链路级加密，你需要设置正确的加密类型，并可能需要协商一些额外的参数。

幸运的是，在 OpenBSD 系统上，所有无线网络设备的配置都通过 `ifconfig` 命令和选项进行，就像配置任何其他网络接口一样。虽然其他 BSD 系统也通过 `ifconfig` 配置大多数网络设置，但在一些系统上，特定功能可能需要其他配置。^([20])不过，由于我们在这里介绍无线网络，所以我们需要从这个新视角来审视网络堆栈中各个层次的安全性。

基本上，有三种流行且简单的 IEEE 802.11 隐私机制，我们将在接下来的章节中简要讨论它们。

### 注意

*有关无线网络安全问题的更全面概述，请参见 Kjell Jørgen Hole 教授的文章和幻灯片，网址为* [`www.kjhole.com/`](http://www.kjhole.com/) *和* [`www.nowires.org/`](http://www.nowires.org/)*.*

### MAC 地址过滤

关于 PF 和 MAC 地址过滤的简短版本是，我们不使用它。许多消费级的现成无线接入点提供 MAC 地址过滤，但与普遍看法相反，它们并没有真正增加太多安全性。其营销成功主要是因为大多数消费者不知道，今天市场上几乎所有无线网络适配器的 MAC 地址都是可以更改的。^([21])

### 注意

*如果你真的想尝试 MAC 地址过滤，可以查阅 OpenBSD 4.7 及以后的版本中，使用`bridge(4)`功能和`ifconfig(8)`中的桥接相关规则选项。我们将在第五章中探讨桥接及其与包过滤结合的更多实用方法。请注意，你可以仅通过将一个接口添加到桥接中，而不实际运行桥接，就使用桥接过滤。*

### WEP

使用无线电波而不是电缆传输数据的一个后果是，外部人员相对更容易捕获通过无线电波传输的数据。802.11 系列无线网络标准的设计者似乎意识到了这一点，他们提出了一个解决方案，并将其推向市场，命名为*有线等效隐私*，或称*WEP*。

不幸的是，WEP 的设计者在设计有线等效加密时，并没有深入研究最新的研究成果或咨询该领域的活跃研究人员。因此，他们推荐的链路级加密方案被密码学专业人士认为是相当原始的自制产品。当 WEP 加密在首批产品发布几个月后被逆向工程并破解时，大家并不感到惊讶。

尽管你可以免费下载工具，在几分钟内破解 WEP 编码的流量，但由于多种原因，WEP 仍然被广泛支持和使用。本质上，今天所有可用的 IEEE 802.11 设备至少支持 WEP，而且惊人的是，许多设备还提供 MAC 地址过滤功能。

你应该将仅由 WEP 保护的网络流量视为比公开广播的数据稍微安全一点。不过，破解 WEP 网络所需的微小努力，可能足以吓退那些懒惰且技术水平低的攻击者。

### WPA

802.11 的设计者们很快意识到，他们的 WEP 系统并不像宣传的那样强大，因此他们提出了一个修订版且稍微更全面的解决方案，称为*Wi-Fi 保护访问*，或称*WPA*。

WPA 在纸面上看起来比 WEP 要好，但由于规范复杂，其广泛实施被推迟了。此外，WPA 因设计问题和存在的一些漏洞，偶尔会产生互操作性问题，受到了批评。再加上访问文档和硬件的常见问题，免费软件的支持程度各不相同。大多数免费系统都支持 WPA，尽管你可能会发现并非所有设备都支持，但随着时间推移，情况有所改善。如果你的项目规范包括 WPA，务必仔细查看你的操作系统和驱动文档。

当然，几乎不言而喻，为了保持数据流的机密性，你需要进一步的安全措施，如 SSH 或 SSL 加密。

### 任务所需的正确硬件

选择合适的硬件不一定是个艰巨的任务。在 BSD 系统中，以下简单命令即可查看所有包含*wireless*关键词的手册页面列表。^([22])

```
$ **apropos wireless**
```

即使在刚安装的系统上，这个命令也会给出操作系统中所有可用的无线网络驱动程序的完整列表。

下一步是阅读驱动手册页面，并将兼容设备列表与可用的硬件部分或你正在考虑的系统中内置的设备进行对比。花些时间思考你具体的需求。作为测试用途，低端的`rum`或`ural` USB 加密狗（或更新的`urtwn`和`run`）会非常有效，并且非常方便。稍后，当你准备构建一个更永久的基础设施时，可能会考虑更高端的设备，尽管你可能会发现便宜的测试设备表现得相当好。一些无线芯片组需要固件，由于法律原因，这些固件不能包含在 OpenBSD 安装介质中。在大多数情况下，只要网络连接可用，*fw_update*脚本在首次启动时会自动获取所需的固件。如果你在一个已经配置好的系统中安装这些设备，可以尝试手动运行*fw_update*。你还可能想阅读本书的附录 B，以获得更多讨论。

### 设置一个简单的无线网络

对于我们的第一个无线网络，使用上一章的基本网关配置作为起点是有意义的。在你的网络设计中，无线网络很可能并不直接连接到互联网，而是需要某种形式的网关。因此，重新使用已有的工作网关设置来配置这个无线接入点是合理的，接下来的几段会对其进行一些小的修改。毕竟，这比从头开始配置更方便。

### 注意

*我们现在处于基础设施建设模式，首先设置接入点。如果您更喜欢先查看客户端设置，请参见客户端部分。*

第一步是确保您拥有一个受支持的卡，并检查驱动程序是否加载并正确初始化该卡。启动时系统消息会在控制台上滚动显示，但它们也会记录在文件*/var/run/dmesg.boot*中。您可以查看该文件本身，或者使用`dmesg`命令查看这些消息。成功配置的 PCI 卡应该会显示类似如下内容：

```
ral0 at pci1 dev 10 function 0 "Ralink RT2561S" rev 0x00: apic 2 int 11 (irq
11), address 00:25:9c:72:cf:60
ral0: MAC/BBP RT2561C, RF RT2527
```

如果您要配置的接口是热插拔类型，例如 USB 或 PC 卡设备，您可以通过查看*/var/log/messages*文件来查看内核消息——例如，在插入设备之前，您可以运行`tail -f`命令查看该文件。

接下来，您需要配置接口：首先启用链路，最后配置系统以支持 TCP/IP。您可以通过命令行完成此操作，方法如下：

```
$ **sudo ifconfig ral0 up mediaopt hostap mode 11g chan 1 nwid unwiredbsd nwkey 0x1deadbeef9**
```

该命令一次完成多项任务。它配置了`ral0`接口，通过`up`参数启用接口，并指定该接口是无线网络的接入点，使用`mediaopt hostap`。然后，明确设置操作模式为`11g`，频道为`11`。最后，使用`nwid`参数将网络名称设置为`unwiredbsd`，WEP 密钥（`nwkey`）设置为十六进制字符串`0x1deadbeef9`。

使用`ifconfig`检查命令是否成功配置了接口：

```
**$ ifconfig ral0**
ral0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500
        lladdr 00:25:9c:72:cf:60
        priority: 4
        groups: wlan
        media: IEEE802.11 autoselect mode 11g hostap
        status: active
        ieee80211: nwid unwiredbsd chan 1 bssid 00:25:9c:72:cf:60 nwkey <not displayed> 100dBm
        inet6 fe80::225:9cff:fe72:cf60%ral0 prefixlen 64 scopeid 0x2
```

请注意`media`和`ieee80211`行的内容。这里显示的信息应与您在`ifconfig`命令行中输入的内容匹配。

在无线网络的链路部分正常工作后，您可以为接口分配一个 IP 地址。首先，设置一个 IPv4 地址：

```
$ **sudo ifconfig ral0 10.50.90.1 255.255.255.0**
```

设置 IPv6 也同样简单：

```
$ **sudo ifconfig alias ral0 2001:db8::baad:f00d:1 64**
```

在 OpenBSD 上，您可以通过创建一个*/etc/hostname.ral0*文件将这两个步骤合并为一个，大致如下：

```
up mediaopt hostap mode 11g chan 1 nwid unwiredbsd nwkey 0x1deadbeef9
inet6 alias 2001:db8::baad:f00d:1 64
```

然后，以**`sh /etc/netstart ral0`**（作为 root 用户）运行，或者耐心等待下次启动完成。

请注意，前面的配置分为几行。第一行生成一个`ifconfig`命令，用正确的参数为物理无线网络设置接口。第二行生成命令，在第一条命令完成后设置 IPv4 地址，然后为双栈配置设置 IPv6 地址。因为这是我们的接入点，所以我们显式设置频道，并通过设置`nwkey`参数启用弱 WEP 加密。

在 NetBSD 上，通常可以将所有这些参数合并为一个*rc.conf*设置：

```
ifconfig_ral0="mediaopt hostap mode 11g chan 1 nwid unwiredbsd nwkey
0x1deadbeef inet 10.50.90.1 netmask 255.255.255.0 inet6 2001:db8::baad:f00d:1
prefixlen 64 alias"
```

FreeBSD 8 及更高版本采用稍有不同的方法，将无线网络设备绑定到统一的`wlan(4)`驱动程序。根据您的内核配置，您可能需要将相关的模块加载行添加到*/boot/loader.conf*中。在我的一个测试系统中，*/boot/loader.conf*看起来是这样的：

```
if_rum_load="YES"
wlan_scan_ap_load="YES"
wlan_scan_sta_load="YES"
wlan_wep_load="YES"
wlan_ccmp_load="YES"
wlan_tkip_load="YES"
```

在加载相关模块后，设置是一个多命令的过程，最好通过一个适用于你的无线网络的*start_if.if*文件来处理。下面是一个 FreeBSD 8 中用于 WEP 接入点的*/etc/start_if.rum0*文件示例：

```
wlans_rum0="wlan0"
create_args_wlan0="wlandev rum0 wlanmode hostap"
ifconfig_wlan0="inet 10.50.90.1 netmask 255.255.255.0 ssid unwiredbsd \
wepmode on wepkey 0x1deadbeef9 mode 11g"
ifconfig_wlan0_ipv6="2001:db8::baad:f00d:1 prefixlen 64"
```

配置成功后，`ifconfig`的输出应该显示物理接口和`wlan`接口都已启动并正在运行：

```
rum0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> metric 0 mtu 2290
        ether 00:24:1d:9a:bf:67
        media: IEEE 802.11 Wireless Ethernet autoselect mode 11g <hostap>
        status: running
wlan0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> metric 0 mtu 1500
        ether 00:24:1d:9a:bf:67
        inet 10.50.90.1 netmask 0xffffff00 broadcast 10.50.90.255
        inet6 2001:db8::baad:f00d:1 prefixlen 64
        media: IEEE 802.11 Wireless Ethernet autoselect mode 11g <hostap>
        status: running
        ssid unwiredbsd channel 6 (2437 Mhz 11g) bssid 00:24:1d:9a:bf:67
        country US authmode OPEN privacy ON deftxkey UNDEF wepkey 1:40-bit
        txpower 0 scanvalid 60 protmode CTS dtimperiod 1 -dfs
```

`status: running`这一行意味着你已经至少在链路层面上启动并运行。

### 注意

*务必查看最新的`ifconfig`手册页，以获取可能更适合你配置的其他选项。*

### 一个 OpenBSD WPA 接入点

WPA 支持在 OpenBSD 4.4 中引入，并扩展到大多数无线网络驱动程序，所有基本的 WPA 密钥管理功能在 OpenBSD 4.9 中合并到`ifconfig(8)`中。

### 注意

*可能仍然有不支持 WPA 的无线网络驱动程序，所以在尝试配置网络以使用 WPA 之前，请检查驱动程序的手册页，看看是否支持 WPA。你可以通过`security/wpa_supplicant`包将 802.1*x 密钥管理与外部认证服务器结合使用“企业”模式，但为了简便起见，我们将使用共享密钥设置。*

设置带有 WPA 的接入点的过程与我们为 WEP 配置时类似。对于带有预共享密钥的 WPA 设置（有时称为*网络密码*），通常会写一个类似这样的*hostname.if*文件：

```
up media autoselect mediaopt hostap mode 11g chan 1 nwid unwiredbsd wpakey 0x1deadbeef9
inet6 alias 2001:db8::baad:f00d:1 64
```

如果你已经在运行前面描述的 WEP 设置，可以通过以下命令禁用这些设置：

```
$ sudo ifconfig ral0 -nwid -nwkey
```

然后，使用以下命令启用新的设置：

```
$ sudo sh /etc/netstart ral0
```

你可以使用`ifconfig`检查接入点是否已启动并运行：

```
$ ifconfig ral0
ral0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500
        lladdr 00:25:9c:72:cf:60
        priority: 4
        groups: wlan
        media: IEEE802.11 autoselect mode 11g hostap
        status: active
        ieee80211: nwid unwiredbsd chan 1 bssid 00:25:9c:72:cf:60 wpapsk <not displayed>
wpaprotos wpa1,wpa2 wpaakms psk wpaciphers tkip,ccmp wpagroupcipher tkip 100dBm
        inet6 fe80::225:9cff:fe72:cf60%ral0 prefixlen 64 scopeid 0x2
        inet6 2001:db8::baad:f00d:1 prefixlen 64
        inet 10.50.90.1 netmask 0xff000000 broadcast 10.255.255.255
```

请注意`status: active`的指示，以及我们没有明确设置的 WPA 选项，它们显示了合理的默认值。

### 一个 FreeBSD WPA 接入点

从我们之前配置的 WEP 接入点转到稍微安全一点的 WPA 设置是很简单的。FreeBSD 上的 WPA 支持通过`hostapd`（一个与 OpenBSD 的`hostapd`类似但不完全相同的程序）来实现。我们首先编辑*/etc/start_if.rum0*文件，以移除认证信息。编辑后的文件应该像这样：

```
wlans_rum0="wlan0"
create_args_wlan0="wlandev rum0 wlanmode hostap"
ifconfig_wlan0="inet 10.50.90.1 netmask 255.255.255.0 ssid unwiredbsd mode 11g"
ifconfig_wlan0_ipv6="2001:db8::baad:f00d:1 prefixlen 64"
```

接下来，我们在*/etc/rc.conf*中添加启用`hostapd`的行：

```
hostapd_enable="YES"
```

最后，`hostapd`需要在*/etc/hostapd.conf*中进行一些配置：

```
interface=wlan0
debug=1
ctrl_interface=/var/run/hostapd
ctrl_interface_group=wheel
ssid=unwiredbsd
wpa=1
wpa_passphrase=0x1deadbeef9
wpa_key_mgmt=WPA-PSK
wpa_pairwise=CCMP TKIP
```

这里，接口规范比较直观，而`debug`值被设置为产生最少的消息。范围是`0`到`4`，其中`0`表示没有调试消息。除非你正在开发`hostapd`，否则不需要更改`ctrl_interface*`设置。接下来的五行中的第一行设置了网络标识符。后续的行启用了 WPA 并设置了密码。最后两行指定了接受的密钥管理算法和加密方案。（有关详细信息和更新，请参阅`hostapd(8)`和`hostapd.conf(5)`的手册页。）

在成功配置后（运行 `sudo /etc/rc.d/hostapd force-start`），`ifconfig` 应该会显示类似于以下的关于两个接口的输出：

```
rum0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> metric 0 mtu 2290
        ether 00:24:1d:9a:bf:67
        media: IEEE 802.11 Wireless Ethernet autoselect mode 11g <hostap>
        status: running
wlan0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> metric 0 mtu 1500
        ether 00:24:1d:9a:bf:67
        inet 10.50.90.1 netmask 0xffffff00 broadcast 10.50.90.255
        inet6 2001:db8::baad:f00d:1 prefixlen 64
        media: IEEE 802.11 Wireless Ethernet autoselect mode 11g <hostap>
        status: running
        ssid unwiredbsd channel 6 (2437 Mhz 11g) bssid 00:24:1d:9a:bf:67
        country US authmode WPA privacy MIXED deftxkey 2 TKIP 2:128-bit
        txpower 0 scanvalid 60 protmode CTS dtimperiod 1 -dfs
```

`status: running` 这一行意味着你已经成功启动，至少在链路层级上已经运行。

### 接入点的 PF 规则集

配置好接口后，接下来是将接入点配置为数据包过滤网关。你可以从第三章复制基础网关设置开始。通过在接入点的 *sysctl.conf* 或 *rc.conf* 文件中进行适当的条目设置来启用网关功能，然后复制 *pf.conf* 文件。根据上一章中对你最有用的部分，*pf.conf* 文件可能看起来像这样：

```
ext_if = "re0" # macro for external interface - use tun0 or pppoe0 for PPPoE
int_if = "re1" # macro for internal interface
localnet = $int_if:network
# nat_address = 203.0.113.5 # Set addess for nat-to
client_out = "{ ssh, domain, pop3, auth, nntp, http,\
                https, cvspserver, 2628, 5999, 8000, 8080 }"
udp_services = "{ domain, ntp }"
icmp_types = "{ echoreq, unreach }"
# if IPv6, some ICMP6 accommodation is needed
icmp6_types = "{ echoreq unreach timex paramprob }"
# If ext_if IPv4 address is dynamic, ($ext_if) otherwise nat to specific address, ie
# match out on $ext_if inet from $localnet nat-to $nat_address
match out on $ext_if inet from $localnet nat-to ($ext_if)
block all
pass quick inet proto { tcp, udp } from $localnet to port $udp_services
pass log inet proto icmp icmp-type $icmp_types
pass inet6 proto icmp6 icmp6-type $icmp6_types
pass inet proto tcp from $localnet port $client_out
```

如果你使用的是 OpenBSD 4.6 或更早版本的 PF，`match` 规则中的 `nat-to` 将变成如下（假设外部接口只有一个地址，并且是动态分配的）：

```
nat on $ext_if from $localnet to any -> ($ext_if)
```

让接入点正常工作所需的唯一差异是 `int_if` 的定义。你必须将 `int_if` 的定义修改为匹配无线接口。在我们的例子中，这意味着这一行应该改为如下所示：

```
int_if = "ral0" # macro for internal interface
```

很可能，你还希望设置 `dhcpd` 来为与接入点关联后的 IPv4 客户端分配地址和其他相关的网络信息。对于 IPv6 网络，你可能需要设置 `rtadvd`（甚至是 DHCP6 守护进程）来帮助你的 IPv6 客户端进行自动配置。如果你阅读手册页，设置 `dhcpd` 和 `rtadvd` 是相当直接的。

就是这样。这个配置为你提供了一个功能性的 BSD 接入点，至少通过 WEP 加密或稍微强一些的链路层加密（如 WPA）提供了基本的安全性（实际上更像是一个 *禁止进入!* 的标志）。如果你需要支持 FTP，从你在第三章中设置的机器复制 `ftp-proxy` 配置，并进行类似你为其余规则集所做的更改。

### 三个或更多接口的接入点

如果你的网络设计要求接入点同时是有线局域网或多个无线网络的网关，你需要对规则集做一些小的调整。除了修改 `int_if` 宏的值外，你可能还需要为无线接口添加另一个（描述性）定义，例如如下所示：

```
air_if = "ral0"
```

你的无线接口很可能位于不同的子网中，因此为每个接口单独设置一个规则来处理任何 IPv4 NAT 配置可能会很有用。以下是 OpenBSD 4.7 及更新系统的示例：

```
match out on $ext_if from $air_if:network nat-to ($ext_if)
```

这里是关于 OpenBSD 4.7 之前的 PF 版本的内容：

```
nat on $ext_if from $air_if:network to any -> ($ext_if) static-port
```

根据你的策略，你可能还需要调整`localnet`定义，或者至少在适当的位置将`$air_if`包含在你的`pass`规则中。再一次，如果你需要支持 FTP，可能需要为无线网络设置一个单独的 pass 规则，并将流量重定向或转发到`ftp-proxy`。

### 处理 IPSec，VPN 解决方案

你可以使用内置的 IPsec 工具、OpenSSH 或其他工具来设置*虚拟专用网络（VPN）*。然而，由于无线网络的安全性普遍较差，或者出于其他原因，你可能希望设置一些额外的安全措施。

选项大致分为三类：

+   ****SSH****。如果你的 VPN 基于 SSH 隧道，基线规则集已经包含了你所需的所有过滤。你的隧道流量对于数据包过滤器来说将与其他 SSH 流量无异。

+   ****带 UDP 密钥交换的 IPsec（IKE/ISAKMP）****。几种 IPsec 变种严重依赖通过`proto udp port 500`的密钥交换，并使用`proto udp port 4500`进行*NAT 穿越（NAT-T）*。你需要允许这些流量通过，以使流建立。几乎所有实现也都严重依赖允许 ESP 协议流量（协议号 50）在主机之间传递，配置如下：

    ```
    pass proto esp from $source to $target
    ```

+   ****基于 IPsec 封装接口的过滤****。通过正确配置的 IPsec 设置，你可以设置 PF 以在封装接口`enc0`本身上进行过滤，使用如下内容：^([23])

    ```
    pass on enc0 proto ipencap from $source to $target keep state (if-bound)
    ```

请参见附录 A，其中包含一些关于该主题的有用文献。

### 客户端侧

只要你有 BSD 客户端，设置就非常简单。将 BSD 机器连接到无线网络的步骤与我们刚才设置无线接入点的步骤非常相似。在 OpenBSD 中，配置重点在于无线接口的*hostname.if*文件。在 FreeBSD 中，配置重点在于*rc.conf*，但根据具体配置，可能还需要涉及其他一些文件。

### OpenBSD 设置

以 OpenBSD 为例，为了连接到我们刚才配置的 WEP 接入点，你的 OpenBSD 客户端需要一个*hostname.if*（例如，*/etc/hostname.ral0*）配置文件，其中包含以下内容：

```
up media autoselect mode 11g chan 1 nwid unwiredbsd nwkey 0x1deadbeef9
dhcp
rtsol
```

第一行比通常需要的更详细地设置了链路层参数。只有`up`、`nwid`和`nwkey`参数是严格必要的。在几乎所有情况下，驱动程序会在适当的频道和最佳可用模式下与接入点关联。第二行要求进行 DHCP 配置，实际上会导致系统运行`dhclient`命令以获取 TCP/IP 配置信息。最后一行调用`rtsol(8)`来启动 IPv6 配置。

如果你选择 WPA 配置，文件看起来会是这样的：

```
up media autoselect mode 11g chan 1 nwid unwiredbsd wpakey 0x1deadbeef9
dhcp
rtsol
```

同样，第一行设置链路级参数，其中最关键的参数是网络选择和加密参数`nwid`和`wpakey`。你可以尝试省略`mode`和`chan`参数；在几乎所有情况下，驱动程序都会在适当的频道和最佳模式下与接入点关联。

如果你希望在将配置写入*/etc/hostname.if*文件之前先在命令行尝试配置命令，设置 WEP 网络客户端的命令如下：

```
$ sudo ifconfig ral0 up mode 11g chan 1 nwid unwiredbsd nwkey 0x1deadbeef9
```

`ifconfig`命令应该没有任何输出。然后你可以使用`ifconfig`检查接口是否已成功配置。输出应该类似于以下内容：

```
$ ifconfig ral0
ral0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500
        lladdr 00:25:9c:72:cf:60
        priority: 4
        groups: wlan
        media: IEEE802.11 autoselect (OFDM54 mode 11g)
        status: active
        ieee80211: nwid unwiredbsd chan 1 bssid 00:25:9c:72:cf:60 nwkey <not displayed> 100dBm
        inet6 fe80::225:9cff:fe72:cf60%ral0 prefixlen 64 scopeid 0x2
```

注意，`ieee80211:`行显示了网络名称和频道，以及其他一些参数。这里显示的信息应该与你在`ifconfig`命令行中输入的内容匹配。

这是配置你的 OpenBSD 客户端连接 WPA 网络的命令：

```
$ sudo ifconfig ral0 nwid unwiredbsd wpakey 0x1deadbeef9
```

命令应该在没有任何输出的情况下完成。如果你再次使用`ifconfig`检查接口状态，输出应该类似于以下内容：

```
$ ifconfig ral0
ral0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> mtu 1500
        lladdr 00:25:9c:72:cf:60
        priority: 4
        groups: wlan
        media: IEEE802.11 autoselect (OFDM54 mode 11g)
        status: active
        ieee80211: nwid unwiredbsd chan 1 bssid 00:25:9c:72:cf:60 wpapsk <not
displayed> wpaprotos wpa1,wpa2 wpaakms psk wpaciphers tkip,ccmp wpagroupcipher
tkip 100dBm
        inet6 fe80::225:9cff:fe72:cf60%ral0 prefixlen 64 scopeid 0x2
```

检查`ieee80211:`行是否显示正确的网络名称和合理的 WPA 参数。

一旦确认接口在链路层配置完毕，可以使用`dhclient`命令配置接口以支持 TCP/IP，如下所示：

```
$ sudo dhclient ral0
```

`dhclient`命令应该打印出与 DHCP 服务器的对话摘要，类似于以下内容：

```
DHCPREQUEST on ral0 to 255.255.255.255 port 67
DHCPREQUEST on ral0 to 255.255.255.255 port 67
DHCPACK from 10.50.90.1 (00:25:9c:72:cf:60)
bound to 10.50.90.11 -- renewal in 1800 seconds.
```

要初始化接口以支持 IPv6，请输入以下命令：

```
$ **sudo rtsol ral0**
```

`rtsol`命令通常会在没有任何消息的情况下完成。使用`ifconfig`检查接口配置，确认接口确实已接收到 IPv6 配置。

### FreeBSD 设置

在 FreeBSD 上，可能需要做比 OpenBSD 更多的配置工作。根据你的内核配置，可能需要将相关的模块加载行添加到*/boot/loader.conf*文件中。在我的一台测试系统中，*/boot/loader.conf*文件内容如下：

```
if_rum_load="YES"
wlan_scan_ap_load="YES"
wlan_scan_sta_load="YES"
wlan_wep_load="YES"
wlan_ccmp_load="YES"
wlan_tkip_load="YES"
```

在加载了相关模块后，你可以通过执行以下命令加入我们之前配置的 WEP 网络：

```
$ sudo ifconfig wlan create wlandev rum0 ssid unwiredbsd wepmode on wepkey 0x1deadbeef9 up
```

然后，执行此命令以获取接口的 IPv4 配置：

```
$ sudo dhclient wlan0
```

要初始化接口以支持 IPv6，请输入以下命令：

```
$ **sudo rtsol ral0**
```

`rtsol`命令通常会在没有任何消息的情况下完成。使用`ifconfig`检查接口配置，确认接口确实已接收到 IPv6 配置。

为了更永久的配置，创建一个*start_if.rum0*文件（如果物理接口名称不同，请将*rum0*替换为实际名称），文件内容如下：

```
wlans_rum0="wlan0"
create_args_wlan0="wlandev rum0 ssid unwiredbsd wepmode on wepkey 0x1deadbeef9 up"
ifconfig_wlan0="DHCP"
ifconfig_wlan0_ipv6="inet6 accept_rtadv"
```

如果你想加入 WPA 网络，需要设置`wpa_supplicant`并稍微更改网络接口设置。对于 WPA 接入点，使用以下配置连接，添加到你的*start_if.rum0*文件中：

```
wlans_rum0="wlan0"
create_args_wlan0="wlandev rum0"
ifconfig_wlan0="WPA"
```

你还需要一个包含以下内容的*/etc/wpa_supplicant.conf*文件：

```
network={
  ssid="unwiredbsd"
  psk="0x1deadbeef9"
}
```

最后，在 *rc.conf* 中添加第二行 `ifconfig_wlan0`，以确保 `dhclient` 正常运行。

```
ifconfig_wlan0="DHCP"
```

对于 IPv6 配置，将以下内容添加到 *rc.conf* 中：

```
ifconfig_wlan0_ipv6="inet6 accept_rtadv"
```

其他 WPA 网络可能需要额外的选项。在成功配置后，`ifconfig` 的输出应该类似于这样：

```
rum0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> metric 0 mtu 2290
        ether 00:24:1d:9a:bf:67
        media: IEEE 802.11 Wireless Ethernet autoselect mode 11g
        status: associated
wlan0: flags=8843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST> metric 0 mtu 1500
        ether 00:24:1d:9a:bf:67
        inet 10.50.90.16 netmask 0xffffff00 broadcast 10.50.90.255
        inet6 2001:db8::baad:f00d:1635 prefixlen 64
        media: IEEE 802.11 Wireless Ethernet OFDM/36Mbps mode 11g
        status: associated
        ssid unwiredbsd channel 1 (2412 Mhz 11g) bssid 00:25:9c:72:cf:60
        country US authmode WPA2/802.11i privacy ON deftxkey UNDEF
        TKIP 2:128-bit txpower 0 bmiss 7 scanvalid 450 bgscan bgscanintvl 300
        bgscanidle 250 roam:rssi 7 roam:rate 5 protmode CTS roaming MANUAL
```

### 使用 authpf 保护你的无线网络

安全专家普遍认为，尽管 WEP 加密提供的保护非常有限，但它勉强足够向潜在攻击者表明你不打算让任何人都能使用你的网络资源。使用 WPA 能显著提高安全性，代价是需要一些复杂的配置，特别是在需要“企业级”选项的场景下。

到目前为止，在本章中我们构建的配置是有效的。无论是 WEP 还是 WPA 配置，都会让所有合理配置的无线客户端连接，这本身可能就是一个问题，因为这种配置没有内建的真实支持来决定谁可以使用你的网络。

如前所述，MAC 地址过滤并不能真正有效地防御攻击者，因为更改 MAC 地址实在太简单了。当 Open-BSD 开发者在 OpenBSD 3.1 版本中引入 `authpf` 时，他们选择了一个截然不同的方法来解决这个问题。与其将访问权限绑定到硬件标识符（如网络卡的 MAC 地址），他们决定使用已经存在的强大且灵活的用户身份验证机制来处理此问题。用户外壳 `authpf` 允许系统根据每个用户来加载 PF 规则，实际上决定了哪个用户可以做什么。

要使用 `authpf`，你需要创建用户，并将 `authpf` 程序作为他们的 shell。为了获得网络访问权限，用户通过 SSH 登录到网关。一旦用户成功完成 SSH 身份验证，`authpf` 会加载你为该用户或相关用户类别定义的规则。

这些规则通常仅适用于用户登录时的 IP 地址，并且在用户通过 SSH 连接登录期间保持加载并有效。一旦 SSH 会话终止，这些规则就会卸载，在大多数情况下，来自用户 IP 地址的所有非 SSH 流量将被拒绝。通过合理的配置，只有经过身份验证的用户产生的流量才会被允许通过。

### 注意

*在 OpenBSD 中，`authpf` 是默认提供的登录类之一，正如你下次使用 `adduser` 创建用户时会注意到的那样。*

对于默认没有 `authpf` 登录类的系统，你可能需要将以下几行添加到你的 */etc/login.conf* 文件中：

```
authpf:\
       :welcome=/etc/motd.authpf:\
       :shell=/usr/sbin/authpf:\
       :tc=default:
```

接下来的几节内容包含了一些示例，虽然可能并不完全适用于你的情况，但我希望它们能为你提供一些启发，帮助你找到适合的解决方案。

### 一个基本的认证网关

使用 `authpf` 设置一个身份验证网关涉及创建并维护一些文件，除了基本的 *pf.conf* 文件之外。主要的新增文件是 *authpf.rules*。其他文件是相对静态的实体，一旦创建好，你不会再花太多时间在它们上面。

首先创建一个空的 */etc/authpf/authpf.conf* 文件。这个文件需要存在才能使 `authpf` 正常工作，但实际上不需要任何内容，所以用 touch 创建一个空文件即可。

接下来是其他相关的 */etc/pf.conf* 部分。首先，这里是接口宏：

```
ext_if = "re0"
int_if = "athn0"
```

此外，如果你定义了一个名为 `<authpf_users>` 的表，`authpf` 将把已认证用户的 IP 地址添加到该表中：

```
table <authpf_users> persist
```

如果需要运行 NAT，负责翻译的规则可以直接放入 *authpf.rules* 中，但在像这样简单的设置中，将其保留在 *pf.conf* 文件中不会有坏处：

```
pass out on $ext_if inet from $localnet nat-to ($ext_if)
```

以下是 OpenBSD 4.7 之前的语法：

```
nat on $ext_if inet from $localnet to any -> ($ext_if)
```

接下来，我们创建 `authpf` anchor，一旦用户进行身份验证，*authpf.rules* 中的规则就会被加载：

```
anchor "authpf/*"
```

对于 OpenBSD 4.7 之前的 `authpf` 版本，需要几个 anchor，因此相应的部分如下所示：

```
nat-anchor "authpf/*"
rdr-anchor "authpf/*"
binat-anchor "authpf/*"
anchor "authpf/*"
```

这标志着 `authpf` 设置所需的 *pf.conf* 文件部分的结束。

对于过滤部分，我们从默认的阻止所有流量开始，然后添加所需的 `pass` 规则。此时唯一必需的项是允许在内部网络上通过 SSH 流量：

```
pass quick on $int_if proto { tcp, udp } to $int_if port ssh
```

从这里开始，实际上完全由你决定。你是否希望在用户进行身份验证之前让客户端进行名称解析？如果是的话，也可以将 TCP 和 UDP 服务域的 `pass` 规则放入你的 *pf.conf* 文件中。

对于一个相对简单且平等的设置，你可以包括我们基线规则集的其余部分，将 `pass` 规则修改为允许来自 `<authpf_users>` 表中的地址的流量，而不是本地网络中的任何地址：

```
pass quick proto { tcp, udp } from <authpf_users> to port $udp_services
pass proto tcp from <authpf_users> to port $client_out
```

对于更为细分的设置，你可以将其余的规则集放入 */etc/authpf/authpf.rules* 中，或将每个用户的规则放在每个用户目录下的定制 *authpf.rules* 文件中，路径为 */etc/authpf/users/*。如果你的用户通常需要一些保护，你的通用 */etc/authpf/authpf.rules* 文件可以包含如下内容：

```
client_out = "{ ssh, domain, pop3, auth, nntp, http, https }"
udp_services = "{ domain, ntp }"
pass quick proto { tcp, udp } from $user_ip to port $udp_services
pass proto tcp from $user_ip to port $client_out
```

宏 `user_ip` 内置于 `authpf` 中，并展开为用户认证时的 IP 地址。这些规则将适用于任何在你的网关上完成身份验证的用户。

一个相对简单且容易实现的功能是为需求与普通用户群体不同的用户设置特殊规则。如果用户目录下的 */etc/authpf/users/* 中存在 *authpf.rules* 文件，那么该文件中的规则将为该用户加载。这意味着你的天真用户 Peter，只需要上网并访问在特定机器的高端口上运行的某个服务，可以通过 */etc/authpf/users/peter/authpf.rules* 文件来满足他的需求，内容如下：

```
client_out = "{ domain, http, https }"
pass inet from $user_ip to 192.168.103.84 port 9000
pass quick inet proto { tcp, udp } from $user_ip to port $client_out
```

另一方面，Peter 的同事 Christina 运行 OpenBSD，并且通常知道自己在做什么，即使她有时会产生来自奇怪端口的流量。你可以通过在 */etc/authpf/users/christina/authpf.rules* 中加入以下内容来给她完全的自由：

```
pass from $user_ip os = "OpenBSD" to any
```

这意味着 Christina 只要从她的 OpenBSD 机器进行认证，就几乎可以在 TCP/IP 上做任何她喜欢的事情。

### 看似开放，但实际上已关闭

在某些设置中，配置网络在链路层保持开放和未加密，同时通过 `authpf` 强制实施一些限制是有意义的。下一个示例非常类似于你可能在机场或其他公共场所遇到的 Wi-Fi 区域，在这些区域中，任何人都可以连接到接入点并获得 IP 地址，但任何访问 Web 的尝试都会被重定向到一个特定的网页，直到用户通过某种认证。^([24])

这个 *pf.conf* 文件再次基于我们的基础设置，加入了两个对基本 `authpf` 设置非常重要的内容——一个宏和一个重定向：

```
ext_if = "re0"
int_if = "ath0"
auth_web="192.168.27.20"
dhcp_services = "{ bootps, bootpc }" # DHCP server + client
table <authpf_users> persist
pass in quick on $int_if proto tcp from ! <authpf_users> to port http rdr-to $auth_web
match out on $ext_if from $int_if:network nat-to ($ext_if)
anchor "authpf/*"
block all
pass quick on $int_if inet proto { tcp, udp } to $int_if port $dhcp_services
pass quick inet proto { tcp, udp } from $int_if:network to any port domain
pass quick on $int_if inet proto { tcp, udp } to $int_if port ssh
For older authpf versions, use this file instead:
ext_if = "re0"
int_if = "ath0"
auth_web="192.168.27.20"
dhcp_services = "{ bootps, bootpc }" # DHCP server + client
table <authpf_users> persist
rdr pass on $int_if proto tcp from ! <authpf_users> to any port http -> $auth_web
nat on $ext_if from $localnet to any -> ($ext_if)
nat-anchor "authpf/*"
rdr-anchor "authpf/*"
binat-anchor "authpf/*"
anchor "authpf/*"
block all
pass quick on $int_if inet proto { tcp, udp } to $int_if port $dhcp_services
pass quick inet proto { tcp, udp } from $int_if:network to port domain
pass quick on $int_if inet proto { tcp, udp } to $int_if port ssh
```

`auth_web` 宏和重定向确保所有来自不在 `<authpf_users>` 表中的地址的 Web 流量都会将所有未认证的用户引导到一个特定地址。在该地址上，你可以设置一个 Web 服务器，提供你所需的内容。这可以是一个包含联系人的单页，用于获取网络访问权限，也可以是一个接受信用卡并处理用户创建的系统。

请注意，在这种设置中，名称解析会正常工作，但所有上网尝试都会最终转到 `auth_web` 地址。用户通过认证后，你可以根据需要向 *authpf.rules* 文件中添加通用规则或特定用户规则。

* * *

^([20]) 在一些系统上，旧的设备特定程序，如 `wicontrol` 和 `ancontrol` 仍然存在，但大多数情况下，它们已经被弃用，并长期被 `ifconfig` 功能所取代。在 OpenBSD 上，`ifconfig` 的整合已经完成。

^([21]) 在 OpenBSD 上快速查阅 man 页面可以告诉你，修改 `rum0` 接口的 MAC 地址的命令就是 `ifconfig rum0 lladdr 00:ba:ad:f0:0d:11`。

^([22]) 此外，您还可以在 Web 上查找 man 页面。请查看 *[`www.openbsd.org/`](http://www.openbsd.org/)* 及其他项目网站，它们提供基于关键字的 man 页面搜索。

^([23]) 在 OpenBSD 4.8 中，封装接口变成了可克隆的接口，你可以配置多个独立的 `enc` 接口。所有 `enc` 接口都会成为 `enc` 接口组的成员。

^([24]) 感谢 Vegard Engen 提供这个想法，并向我展示他的配置，虽然没有保留所有细节，但精神得以保留在这里。
