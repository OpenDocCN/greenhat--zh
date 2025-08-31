## 第十一章：附加外设设备的取证痕迹**

![图片](img/common01.jpg)

在本章中，外设设备指的是诸如存储设备、摄像头、网络摄像头、打印机、扫描仪、移动设备等外部连接的硬件。我们将通过日志和配置文件中的痕迹尝试识别和分析这些附加设备。从取证的角度来看，我们试图尽可能多地了解这些设备，特别是任何独特的识别信息和使用证据。了解哪些设备连接到系统以及它们是如何被使用的，有助于重建过去的事件和活动。

你可能会注意到本章中缺少了蓝牙设备。它们也被视为外设，但它们与其他无线分析主题一起在第八章中讨论。

### Linux 外设设备

用于连接外部外设设备的最常见接口是 USB 和 Thunderbolt。USB 设备占据了外部连接设备的绝大多数，远远超过任何其他外部接口。Thunderbolt 的物理接口现在使用 USB3C，并提供连接 PCI Express 设备的能力。此外，Fibre Channel（FC）和串行附加 SCSI（SAS）PCI 板卡提供的外部接口主要出现在企业环境中。

#### *Linux 设备管理*

如第二章中所述，Unix 最初开发时，一个核心理念（Linux 也采纳了这一理念）是“任何事物都是文件”。这一革命性的思想使得可以通过与内核交互的特殊文件来访问硬件设备。

设备文件可以分为两种类型（块设备或字符设备），它们具有关联的编号（主设备号和次设备号），用于指定设备的类别和实例。字符设备是按顺序访问（或流式传输）一个字节一个字节地访问，通常用于键盘、视频、打印机和其他串行设备。块设备则按块大小进行访问，可以进行缓存或随机访问，通常用于存储设备。

设备文件通常位于*/dev/*目录下，并由 udev 守护进程（systemd-udevd）动态创建。*/dev/*目录是一个伪文件系统，由正在运行的内核在内存中提供。因此，本目录中的设备文件在尸检取证检查中是不存在的。^(1) 设备文件不一定必须位于*/dev/*目录下，也可以通过`mknod`命令或`mknod`系统调用在任何地方创建。然而，任何位于*/dev/*目录之外的设备文件都是可疑的，值得进一步检查。

systemd-udevd 守护进程会注意到设备何时被内核附加或移除，并使用在规则文件中指定的 udev 规则设置相应的设备文件。软件包可能会在*/usr/lib/udev/rules.d/*目录中创建 udev 规则文件，而系统管理员则在*/etc/udev/rules.d/*目录中创建自定义的 udev 规则文件。以下是一个 udev 规则文件的示例：

```
$ cat /etc/udev/rules.d/nitrokey.rules
ATTRS{idVendor}=="20a0", ATTRS{idProduct}=="4108", MODE="660", GROUP="sam", TAG+="systemd"
```

系统所有者（`sam`）为一个 USB 设备 ID 为 20a0:4108 的 Nitrokey 身份验证设备创建了一个规则，定义了如何设置权限和组所有权。

检查*/etc/udev/rules.d/*将显示任何由系统所有者调整或创建的文件。有关 udev 的更多信息，请参见 udev(7)手册页。

#### *识别连接的 USB 设备*

USB 索引 USB 设备被创建用以整合和替代老化的外部外设接口，如 RS-232、并行打印机接口、PS/2 键盘和鼠标以及其他专有 PC 接口。它旨在支持多功能应用，如磁盘、键盘、鼠标、声音、网络连接、打印和扫描，以及连接小型设备（如手机）。越来越多的物联网设备可以通过 USB 连接到 PC，并且可能包含作为法医证据有用的数据。

在法医检查过程中，创建连接的 USB 设备列表将有助于回答与调查相关的问题，提供诸如以下信息：

+   人员接近的指示

+   在某一时刻的活动

+   其他设备的查找与分析

+   将特定设备与分析中的系统关联

在法医调查的背景下，我们特别关注唯一标识符和时间戳。唯一标识符将把特定设备与事件或犯罪中的特定计算机关联起来。USB 唯一标识符可能包括存储在设备固件或设备内存中的硬件序列号或 UUID。在尝试识别 USB 设备时，我们可以检查日志文件、配置文件和其他持久化数据。

USB 设备会出现在内核日志中，如下所示：

```
Dec 30 09:13:20 pc1 kernel: usb 5-3.2: new full-speed USB device number 36 using xhci_hcd
Dec 30 09:13:20 pc1 kernel: usb 5-3.2: New USB device found, idVendor=05ac, idProduct=1393,
bcdDevice= 1.05
Dec 30 09:13:20 pc1 kernel: usb 5-3.2: New USB device strings: Mfr=1, Product=2, SerialNumber=3
Dec 30 09:13:20 pc1 kernel: usb 5-3.2: Product: AirPod Case
Dec 30 09:13:20 pc1 kernel: usb 5-3.2: Manufacturer: Apple Inc.
Dec 30 09:13:20 pc1 kernel: usb 5-3.2: SerialNumber: GX3CFW4PLKKT
...
Dec 30 09:13:20 pc1 kernel: usbcore: registered new device driver apple-mfi-fastcharge
...
Dec 30 09:16:00 pc1 kernel: usb 5-3.2: USB disconnect, device number 36
```

这个例子显示了一个 Apple AirPod 充电盒在 12 月 30 日上午 9:13（`09:13:20`）连接。序列号提供了唯一的标识。断开连接的日志条目显示，AirPod 充电盒几分钟后被拔出。在分析存储设备日志时，设备号和 USB 端口（在这个例子中是`36`和`5-3.2`）是从内核日志中移除设备时唯一显示的信息。这些提供了与其他包含更详细设备信息（如制造商、产品、序列号等）的日志条目的关联。

从取证的角度来看，插入和移除的时间戳是有意义的。它们提供了一个指示，表明某人在设备插入和拔出时处于计算机的物理接近位置，并且可以推测使用时长。在得出确切的使用结论之前，可能需要其他日志和信息来验证这些时间戳。USB 设备插入的端口表示使用了哪个物理连接器。这些信息可能很有用，例如，如果 USB 设备被插入到机架一排中的服务器中，那么前端或后端的位置可能与数据中心的 CCTV 监控录像中的活动相吻合。

视频会议最近变得越来越流行，Linux 支持 Zoom、Teams、Jitsi 等视频会议软件。这些软件依赖于 USB 网络摄像头和麦克风（笔记本电脑为内置，台式机为外置）。这些设备可以以与本节描述的其他设备相同的方式进行查找，但 Linux 通过 Video4Linux（V4L）框架来管理视频设备，该框架是 Linux 媒体子系统的一部分。当视频设备连接到 Linux 系统时，内核会检测到它并创建一个*/dev/video0*设备（多个摄像头将显示为*/dev/video1*、*/dev/video2*，依此类推）。典型的视频设备包括网络摄像头、数码摄像机、电视调谐器和视频采集卡。以下是一个示例：

```
Dec 30 03:45:56 pc1 kernel: usb 6-3.4: new SuperSpeed Gen 1 USB device number 3 using xhci_hcd
Dec 30 03:45:56 pc1 kernel: usb 6-3.4: New USB device found, idVendor=046d, idProduct=0893,
bcdDevice= 3.17
Dec 30 03:45:56 pc1 kernel: usb 6-3.4: New USB device strings: Mfr=0, Product=2, SerialNumber=3
Dec 30 03:45:56 pc1 kernel: usb 6-3.4: Product: Logitech StreamCam
Dec 30 03:45:56 pc1 kernel: usb 6-3.4: SerialNumber: 32B24605
Dec 30 03:45:56 pc1 kernel: hid-generic 0003:046D:0893.0005: hiddev1,hidraw4: USB HID v1.11
 Device [Logitech StreamCam] on usb-0000:0f:00.3-3.4/input5
...
Dec 30 03:45:56 pc1 kernel: mc: Linux media interface: v0.10
Dec 30 03:45:56 pc1 kernel: videodev: Linux video capture interface: v2.00
Dec 30 03:45:56 pc1 kernel: usbcore: registered new interface driver snd-usb-audio
Dec 30 03:45:56 pc1 kernel: uvcvideo: Found UVC 1.00 device Logitech StreamCam (046d:0893)
Dec 30 03:45:56 pc1 kernel: input: Logitech StreamCam as
/devices/pci0000:00/0000:00:08.1/0000:0f:00.3/usb6/6-3/6-3.4/6-3.4:1.0/input/input25
Dec 30 03:45:56 pc1 kernel: usbcore: registered new interface driver uvcvideo
Dec 30 03:45:56 pc1 kernel: USB Video Class driver (1.1.1)
Dec 30 03:45:56 pc1 systemd[587]: Reached target Sound Card.
```

这里，USB 设备通过制造商/型号/序列号信息被检测到，随后启动 Linux 视频驱动程序，使得可以使用视频设备进行录制、视频会议或观看电视。

已知的 USB 硬件 ID 列表可以在*/usr/share/hwdata/usb.ids*文件中找到，或者在* [`www.linux-usb.org/usb-ids.html`](http://www.linux-usb.org/usb-ids.html)*网站上查阅。这个列表按供应商、设备和接口名称进行格式化，由社区共同维护。

#### *识别 PCI 和 Thunderbolt 设备*

PCI Express 或 PCIe（外设组件互连快车）是一种规范（*[`pcisig.com/`](https://pcisig.com/)*)，用于为连接 PCIe 设备提供总线接口。PCIe 设备通常是插入主板上 PCIe 插槽的卡，或者是集成到主板中的设备。

在日志中查找 PCIe 设备依赖于设备的内核模块，有些模块记录的信息比其他模块更多。以下示例展示了一个内核模块记录 PCIe 设备信息：

```
Dec 29 10:37:32 pc1 kernel: pci 0000:02:00.0: [10de:1c82] type 00 class
 0x030000
...
Dec 29 10:37:32 pc1 kernel: pci 0000:02:00.0: 16.000 Gb/s available
PCIe bandwidth, limited by 2.5 GT/s PCIe x8 link at 0000:00:01.0
(capable of 126.016 Gb/s with 8.0 GT/s PCIe x16 link)
...
Dec 29 10:37:33 pc1 kernel: nouveau 0000:02:00.0: NVIDIA GP107 (137000a1)
...
Dec 29 10:37:33 pc1 kernel: nouveau 0000:02:00.0: bios: version 86.07.59.00.24
Dec 29 10:37:34 pc1 kernel: nouveau 0000:02:00.0: pmu: firmware unavailable
Dec 29 10:37:34 pc1 kernel: nouveau 0000:02:00.0: fb: 4096 MiB GDDR5
...
Dec 29 10:37:34 pc1 kernel: nouveau 0000:02:00.0: DRM: allocated 3840x2160 fb:
0x200000, bo 00000000c125ca9a
Dec 29 10:37:34 pc1 kernel: fbcon: nouveaudrmfb (fb0) is primary device
```

这里检测到一张 Nvidia GP107 PCIe 显卡插在主板的物理插槽（总线）2 中。我们可以分析描述物理 PCIe 插槽的内核日志，并将其与已检测到的 PCIe 设备关联起来。

上述示例中的字符串 `0000:02:00.0` 以 `<domain>:` `<bus>:<device>.<function>` 格式表示。此格式描述了 PCIEe 设备在系统中的位置，以及多功能设备的功能号。字符串 `[10de:1c82]` 表示设备厂商（NVIDIA）和产品（GP107）。

若要查看已知的 PCI 硬件 ID 列表，请参阅 */usr/share/hwdata/pci.ids* 文件或 *[`pci-ids.ucw.cz/`](http://pci-ids.ucw.cz/)* 网站。这些列表按厂商、设备、子厂商和子设备名称格式化，由社区共同维护。pci.ids(5) 手册页详细描述了该文件。

Thunderbolt 是由 Apple 和 Intel 联合开发的高速外部接口，用于通过单一接口连接磁盘、视频显示器和 PCIe 设备。以代号 Light Peak 开发，最初计划为光纤连接。Apple 在 Thunderbolt 的普及上起到了主导作用（主要是在 Apple 用户中），通过 Apple 硬件推广该技术。

物理接口使用 Mini DisplayPort 作为 Thunderbolt 1 和 Thunderbolt 2 的接口，Thunderbolt 3 则采用 USB Type-C 电缆和连接器。

Thunderbolt 3\. Thunderbolt 3 接口将 PCIe、DisplayPort 和 USB3 集成到一个单一接口中。Thunderbolt 1、2 和 3 分别提供 10、20 和 40Gbps 的传输速度。

以下示例显示了连接到 Linux 笔记本的 Thunderbolt 设备：

```
Dec 30 10:45:27 pc1 kernel: thunderbolt 0-3: new device found, vendor=0x1 device=0x8003
Dec 30 10:45:27 pc1 kernel: thunderbolt 0-3: Apple, Inc. Thunderbolt to Gigabit Ethernet
 Adapter
Dec 30 10:45:27 pc1 boltd[429]: [409f9f01-0200-Thunderbolt to Gigabit Ethe] parent is
 c6030000-0060...
Dec 30 10:45:27 pc1 boltd[429]: [409f9f01-0200-Thunderbolt to Gigabit Ethe] connected:
 authorized
 (/sys/devices/pci0000:00/0000:00:1d.4/0000:05:00.0/0000:06:00.0/0000:07:00.0/domain0/0-0/0-3)
Dec 30 10:45:29 pc1 kernel: tg3 0000:30:00.0 eth1: Link is up at 1000 Mbps, full duplex
Dec 30 10:45:29 pc1 kernel: tg3 0000:30:00.0 eth1: Flow control is on for TX and on for RX
Dec 30 10:45:29 pc1 kernel: tg3 0000:30:00.0 eth1: EEE is enabled
Dec 30 10:45:29 pc1 kernel: IPv6: ADDRCONF(NETDEV_CHANGE): eth1: link becomes ready
Dec 30 10:45:29 pc1 systemd-networkd[270]: eth1: Gained carrier
...
Dec 30 10:50:56 pc1 kernel: thunderbolt 0-3: device disconnected
Dec 30 10:50:56 pc1 boltd[429]: [409f9f01-0200-Thunderbolt to Gigabit Ethe] disconnected
 (/sys/devices/pci0000:00/0000:00:1d.4/0000:05:00.0/0000:06:00.0/0000:07:00.0/domain0/0-0/0-3)
Dec 30 10:50:56 pc1 systemd-networkd[270]: eth1: Lost carrier
```

日志显示，在 12 月 30 日的 10:45 插入了一个 Thunderbolt 千兆以太网适配器，并在几分钟后（10:50）拔出。在这台机器上，systemd-networkd 守护进程正在管理网络，并监视以太网链接状态（承载）。

Thunderbolt 3 引入了几项安全功能，以通过直接内存访问（DMA）来减轻未经授权的内存访问风险。^(2) `boltd` 守护进程（在前面的示例中看到）管理启用安全级别的 Thunderbolt 3 设备的授权。

### 打印机和扫描仪

打印和打印机自 Unix 计算开始以来一直是其一部分。Unix 最早的一个应用就是在贝尔实验室进行文档（专利申请）的文本格式化^(3)。

打印机和扫描仪作为数字世界与物理世界之间的桥梁。打印机和扫描仪执行相反的功能：一个将电子文件转化为纸质文档，另一个则将纸质文档转化为电子文件。两者都是标准组件。

现在的办公室中都有打印机和扫描仪，并且 Linux 系统对此有很好的支持。打印和扫描分析是法医检查的标准部分，用于识别 Linux 系统上遗留下来的文档痕迹。

#### *打印机和打印历史分析*

传统的 Unix 打印通常使用 BSD 行打印机守护进程（`lpd`）接受并排队安装打印机的打印作业。现代 Linux 系统采用了通用的 Unix 打印系统（CUPS），自从最初在基于 Unix 的 OS X 操作系统中使用以来，得到了 Apple 的显著支持和参与。对打印系统的取证分析可能会揭示过去的打印活动信息。

CUPS 软件包可以配置为使用直接连接的打印机（通常通过 USB）或通过网络连接的打印机。通过网络打印时，可以使用多种协议（如 IPP、lpr、HP JetDirect 等），其中推荐使用互联网打印协议（IPP）。`cupsd` 守护进程监听打印请求，并通过 TCP 端口 631 上的本地 web 服务器管理打印系统。

*/etc/cups/* 目录包含 CUPS 配置，单个打印机添加到 *printers.conf* 文件中（通过 CUPS 界面或由发行版提供的 GUI）。以下是一个示例 */etc/cups/printers.conf* 文件：

```
# Printer configuration file for CUPS v2.3.3op1
# Written by cupsd
# DO NOT EDIT THIS FILE WHEN CUPSD IS RUNNING
NextPrinterId 7
<Printer bro>
PrinterId 6
UUID urn:uuid:55fea3b9-7948-3f4c-75af-e18d47c02475
AuthInfoRequired none
Info Tree Killer
Location My Office
MakeModel Brother HLL2370DN for CUPS
DeviceURI ipp://bro.example.com/ipp/port1
State Idle
StateTime 1609329922
ConfigTime 1609329830
Type 8425492
Accepting Yes
Shared No
JobSheets none none
QuotaPeriod 0
PageLimit 0
KLimit 0
OpPolicy default
ErrorPolicy stop-printer
Attribute marker-colors \#000000,none
Attribute marker-levels -1,98
Attribute marker-low-levels 16
Attribute marker-high-levels 100
Attribute marker-names Black Toner Cartridge,Drum Unit
Attribute marker-types toner
Attribute marker-change-time 1609329922
</Printer>
```

打印机名称 `bro` 由 `<printer bro>` 和 `</printer>` 标签指定（这种类似 HTML 的标记方式允许在同一文件中配置多个打印机）。有关品牌和型号的信息被记录下来，并且当打印机配置或属性更改时，会更新多个时间戳。

除了打印作业，cupsd 守护进程还管理配置请求和其他本地管理任务。这些活动记录在 */var/log/cups/* 目录中，其中可能包含 *access_log*、*error_log* 和 *page_log* 文件，记录有关 CUPS 活动的信息，包括配置的打印机活动。这些日志在 cupsd-logs(5) 手册页中有文档记录。

*access_log* 文件记录管理活动以及对不同配置打印机的打印请求：

```
localhost - root [30/Dec/2020:13:46:57 +0100] "POST /admin/ HTTP/1.1"
 200 163 Pause-Printer successful-ok
localhost - root [30/Dec/2020:13:47:02 +0100] "POST /admin/ HTTP/1.1"
 200 163 Resume-Printer successful-ok
...
localhost - - [30/Dec/2020:13:48:19 +0100] "POST /printers/bro HTTP/1.1"
 200 52928 Send-Document successful-ok
```

在这里，打印机被暂停和恢复，然后打印文档。

*error_log* 文件记录各种错误和警告信息，其中可能包含关于打印机安装失败、打印问题以及其他可能与调查相关的异常事件的有趣信息，例如以下示例：

```
E [30/Apr/2020:10:46:37 +0200] [Job 46] The printer is not responding.
```

*error_log* 行以字母开头（`E` 表示错误，`W` 表示警告，依此类推）。这些错误字母在 cupsd-logs(5) 手册页中列出。

*page_log* 文件对于调查人员特别有用，因为它记录了过去打印作业和文件名的历史；例如：

```
bro sam 271 [15/Oct/2020:08:46:16 +0200] total 1 - localhost Sales receipt_35099373.pdf - -
bro sam 368 [30/Dec/2020:13:48:41 +0100] total 1 - localhost Hacking History - Part2.odt - -
...
```

显示了两个打印作业，包括打印机名称（`bro`）、打印作业的用户（`sam`）、打印时间和文件名。

这些日志文件可能会随着时间的推移而旋转，并添加数字扩展名（*error_log.1*、*page_log.2* 等）。与其他用户活动不同，用户的主目录中存储的信息不多。打印作业会传递给 CUPS 守护进程，后者作为系统范围的功能管理配置和日志记录。这些日志用于本地和网络配置的打印机。CUPS 拥有十多个手册页，因此可以从 cups(1) 手册页或 *[`www.cups.org/`](https://www.cups.org/)* 开始，获取更多信息。

除了 CUPS 日志，将 USB 打印机连接到本地机器还会在 systemd 日志中生成日志，如下所示：

```
Dec 30 14:42:41 localhost.localdomain kernel: usb 4-1.3: new high-speed USB device number 15
using ehci-pci
Dec 30 14:42:41 pc1 kernel: usb 4-1.3: New USB device found, idVendor=04f9,
idProduct=00a0, bcdDevice= 1.00
Dec 30 14:42:41 pc1 kernel: usb 4-1.3: New USB device strings: Mfr=1, Product=2,
SerialNumber=3
Dec 30 14:42:41 pc1 kernel: usb 4-1.3: Product: HL-L2370DN series
Dec 30 14:42:41 pc1 kernel: usb 4-1.3: Manufacturer: Brother
Dec 30 14:42:41 pc1 kernel: usb 4-1.3: SerialNumber: E78098H9N222411
...
Dec 30 14:42:41 localhost.localdomain kernel: usblp 4-1.3:1.0: usblp0: USB Bidirectional
printer dev 15 if 0 alt 0 proto 2 vid 0x04F9 pid 0x00A0
Dec 30 14:42:41 localhost.localdomain kernel: usbcore: registered new interface
driver usblp
...
Dec 30 14:45:19 localhost.localdomain kernel: usb 4-1.3: USB disconnect, device number 15
Dec 30 14:45:19 localhost.localdomain kernel: usblp0: removed
```

在这里，一台 Brother 打印机在下午 2:42（`14:42:41`）插入，并在几分钟后于下午 2:45（`14:45:19`）拔出。显示了型号和序列号。USB 设备（`usblp0`）也被记录，这是在多台打印机连接到单一系统时的有用信息。

#### *扫描设备和历史记录分析*

在 Linux 下进行扫描时，使用的是扫描器访问现在简单（SANE）API。一个较老的竞争系统是 TWAIN (*[`www.twain.org/`](https://www.twain.org/)*)，但现在大多数发行版都在使用 SANE。SANE 的流行部分是因为前端 GUI 和后端扫描配置驱动程序（位于 */etc/sane.d/*）的分离，以及用于网络扫描的 SANE 守护进程（`saned`）。

将 USB 扫描仪插入 Linux 机器时，会导致信息被记录：

```
Dec 30 15:04:41 pc1 kernel: usb 1-3: new high-speed USB device number 19 using xhci_hcd
Dec 30 15:04:41 pc1 kernel: usb 1-3: New USB device found, idVendor=04a9, idProduct=1905,
bcdDevice= 6.03
Dec 30 15:04:41 pc1 kernel: usb 1-3: New USB device strings: Mfr=1, Product=2, SerialNumber=0

Dec 30 15:04:41 pc1 kernel: usb 1-3: Product: CanoScan
Dec 30 15:04:41 pc1 kernel: usb 1-3: Manufacturer: Canon
...
Dec 30 15:21:32 pc1 kernel: usb 1-3: USB disconnect, device number 19
```

在这里，一台 Canon CanoScan 设备在下午 3:00 过后插入，然后在 17 分钟后被拔出。

任何前端应用程序都可以使用 SANE 后端库提供的 API。这意味着从取证角度来看，感兴趣的日志和持久数据将是特定于应用程序的。以下示例显示了默认安装在 Linux Mint 上的 simple-scan 应用程序。这些信息可以在用户的主目录中的*~/.cache/simple-scan/simple-scan.log* 文件中找到：

```
[+0.00s] DEBUG: simple-scan.vala:1720: Starting simple-scan 3.36.3, PID=172794
...
[+62.29s] DEBUG: scanner.vala:1285: sane_start (page=0, pass=0) -> SANE_STATUS_GOOD
...
[+87.07s] DEBUG: scanner.vala:1399: sane_read (15313) -> (SANE_STATUS_EOF, 0)
...
[+271.21s] DEBUG: app-window.vala:659: Saving to
 'file:///home/sam/Documents/Scanned%20Document.pdf'
```

每次使用 `simple-scan` 程序时，这个扫描日志都会被重新创建（覆盖先前的日志）。日志时间反映了程序启动以来的秒数，可以通过将这些值加到日志文件的创建时间戳来计算时间戳。在这里我们看到，程序启动后一分钟扫描了一份文档（大约用了 25 秒钟完成）。三分钟后，文档被保存到用户的 *Documents* 文件夹，文件名为 *Scanned Document.pdf*（日志中的 `%20` 代表空格）。

在涉及扫描仪的取证检查中，你需要确定使用了哪种扫描软件，然后分析该程序的遗留物（XDG 目录、日志、缓存等）。

### 外部附加存储

在许多取证调查中，特别是涉及非法材料或被盗文档的案件中，识别所有已连接到被检查计算机的存储设备非常重要。在 Linux 系统中，我们可以在多个地方找到这些信息。

外部存储通过硬件接口（如 USB 或 Thunderbolt）连接到计算机系统。计算机通过该接口使用低级协议（如 SCSI、ATA、USB BoT 等）与这些驱动器通信，以读取和写入扇区（这些扇区构成了文件系统的块）。像 USB 闪存驱动器或外部硬盘这样的存储设备，将接口电子设备和存储介质集成到一个单一设备中。然而，在某些情况下，硬盘和存储介质是分开的，被称为可移动介质设备。此类设备的例子包括 SD 卡、光盘（CD/DVD）和磁带。

#### *存储硬件识别*

当一个新的存储设备连接到 Linux 系统时，系统会配置适当的设备驱动程序并创建设备文件。设置完成后，可以挂载文件系统。挂载文件系统可以是自动的、手动的，或者在系统启动时进行。新连接的设备在内核中的设置与挂载其包含的文件系统是分开且独立的。这就是为什么我们可以在不挂载设备的情况下进行取证镜像（通过直接访问设备扇区）。

一旦内核识别出一个新的存储设备，设备文件会在 */dev/* 目录下创建（通过 udevd 的帮助），并且可以在内核的 dmesg 日志或其他系统日志中找到。以下示例来自 systemd 日志：

```
Dec 30 15:49:23 pc1 kernel: usb 1-7: new high-speed USB device number 23 using xhci_hcd
Dec 30 15:49:23 pc1 kernel: usb 1-7: New USB device found, idVendor=0781, idProduct=5567,
 bcdDevice= 1.00
Dec 30 15:49:23 pc1 kernel: usb 1-7: New USB device strings: Mfr=1, Product=2, SerialNumber=3
Dec 30 15:49:23 pc1 kernel: usb 1-7: Product: Cruzer Blade
Dec 30 15:49:23 pc1 kernel: usb 1-7: Manufacturer: SanDisk
Dec 30 15:49:23 pc1 kernel: usb 1-7: SerialNumber: 4C530001310731103142
Dec 30 15:49:23 pc1 kernel: usb-storage 1-7:1.0: USB Mass Storage device detected
Dec 30 15:49:23 pc1 kernel: scsi host5: usb-storage 1-7:1.0
...
Dec 30 15:49:24 pc1 kernel: scsi 5:0:0:0: Direct-Access   SanDisk Cruzer Blade   1.00
 PQ: 0 ANSI: 6
Dec 30 15:49:24 pc1 kernel: sd 5:0:0:0: Attached scsi generic sg2 type 0
Dec 30 15:49:24 pc1 kernel: sd 5:0:0:0: [sdc] 30031872 512-byte logical blocks:
 (15.4 GB/14.3 GiB)
...
Dec 30 15:49:24 pc1 kernel: sdc: sdc1
Dec 30 15:49:24 pc1 kernel: sd 5:0:0:0: [sdc] Attached SCSI removable disk
...
```

在这里，内核检测到一个新的 USB 设备，确定它是存储设备，并创建了 `sdc` 设备。显示了 512 字节扇区的数量，表示硬盘的大小（`30031872 512-byte logical blocks`）。关于制造商、产品和序列号的信息也被记录在日志中。使用的设备名称（此处为 `[sdc]`）可以在驱动器连接期间的其他日志中找到。

当一个存储设备从 Linux 系统中移除时，如前所述，内核不会生成太多信息：

```
Dec 30 16:02:54 pc1 kernel: usb 1-7: USB disconnect, device number 23
```

在这个例子中，USB 闪存盘在插入约 15 分钟后被移除。（有关驱动器挂载和卸载的信息将在下一节描述。）

从产品、制造商和大小上可能可以明显看出存储设备是 USB 闪存盘还是外部硬盘盒。但在某些情况下，您可能需要额外的指示器。如果一个普通的 SATA 硬盘被装入硬盘盒，且它是高级格式或 4K 原生硬盘，它可能会显示一行附加日志，内容为 `4096-byte physical blocks`。USB 闪存盘（和旧款硬盘）只会显示 512 字节的逻辑块行。以下是此附加日志的示例：

```
Dec 30 16:41:57 pc1 kernel: sd 7:0:0:0: [sde] 7814037168 512-byte logical blocks:
 (4.00 TB/3.64 TiB)
Dec 30 16:41:57 pc1 kernel: sd 7:0:0:0: [sde] 4096-byte physical blocks
```

在这里，一个外部 USB 外壳中的磁盘（一个 SATA 对接站）记录了 4096 字节的物理块（4K 本地扇区）。我的前一本书《实践取证成像》（No Starch Press，2016）中更详细地解释了高级格式和 4K 本地驱动器。

#### *挂载存储的证据*

在内核设置设备驱动程序并创建设备文件后，可以挂载文件系统。外部驱动器的挂载证据可以在多个地方找到。

在服务器上，永久附加的外部存储的文件系统会在*/etc/fstab*文件中静态配置，以便每次系统启动时自动挂载。一个*fstab*示例如下：

```
$ cat /etc/fstab
# Static information about the filesystems.
# See fstab(5) for details.

# <file system> <dir> <type> <options> <dump> <pass>
UUID=b4b80f70-1517-4637-ab5f-fa2a211bc5a3/    ext4   rw,relatime0 1

# all my cool vids
UUID=e2f063d4-e442-47f5-b4d1-b5c936b6ec7f/data    ext4   rw,relatime0 1
...
```

在这里，`/`是安装了操作系统的根文件系统，而`/data`是管理员添加的外部数据驱动器。该文件包含唯一的 UUID、挂载目录，以及可能由管理员添加的注释。其他设备识别信息可以在日志中找到（如前一节所述）。

在桌面计算机上，Linux 发行版通常会提供简单而舒适的用户体验，通常会自动挂载文件系统，并在桌面或文件管理器中显示。这是通过`udisks`程序完成的，系统在设置设备后通过 D-Bus 调用该程序。

udisks 程序会在*/media/*或*/run/media/*中创建一个临时挂载点，然后挂载驱动器。它随后会显示在用户的桌面或文件管理器中。以下示例显示了一个自动挂载驱动器的日志：

```
Dec 30 15:49:25 pc1 udisksd[773]: Mounted /dev/sdc1 at /run/media/sam/My Awesome Vids
on behalf of uid 1000
...
Dec 30 16:01:52 pc1 udisksd[773]: udisks_state_check_mounted_fs_entry: block device
/dev/sdc1 is busy, skipping cleanup
Dec 30 16:01:52 pc1 systemd[2574]: run-media-sam-My\x20Awesome\x20Vids.mount: Succeeded.
Dec 30 16:01:52 pc1 udisksd[773]: Cleaning up mount point /run/media/sam/My Awesome Vids
(device 8:33 is not mounted)
...
```

挂载的驱动器的卷名称为`My Awesome Vids`。当通过桌面上的弹出菜单项卸载驱动器时，它将在卸载后删除临时目录并记录日志：

```
Dec 30 16:01:52 pc1 udisksd[773]: Unmounted /dev/sdc1 on behalf of uid 1000
Dec 30 16:01:53 pc1 kernel: sdc: detected capacity change from 15376318464 to 0
```

然后，可以物理移除该驱动器。

手动挂载也会在系统日志文件中留下痕迹。当系统管理员通过命令行将文件系统挂载到他们选择的挂载点时，手动挂载的证据可以在日志和 shell 历史记录中找到。如果非 root 用户手动挂载文件系统，他们需要提升权限，并通常会在命令前加上`sudo`。以下是两个`mount`命令示例，一个是在 root 用户的 shell 历史记录中，另一个是在普通用户的 shell 历史记录中：

```
# mount /dev/sda1 /mnt
$ sudo mount /dev/sda1 /mnt
```

其他需要注意的指示器可能包括与存储相关的错误消息、坏道或未干净卸载的存储设备。根据使用的文件管理器，也可能会有缓存信息、历史记录或书签，表明外部存储设备的使用。

### 总结

本章已经涵盖了附加到 Linux 系统的外部外设的分析。连接和移除外设会在日志中留下痕迹，这些痕迹可以被检查。此外，本章还描述了如何分析打印子系统以及扫描过程的工作原理。现在你应该能够查找附加和移除的外设、扫描和打印的文档的证据。
