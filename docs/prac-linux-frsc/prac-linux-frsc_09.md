# 第九章：**时间与位置的取证分析**

![Image](img/common01.jpg)

本章解释了与 Linux 时间、区域设置和位置相关的数字取证概念。探讨了取证时间线，包括如何从 Linux 系统构建取证时间线。还描述了国际配置，如区域设置、键盘和语言。最后一节涵盖了地理定位技术以及如何重建 Linux 系统的地理位置历史。

### Linux 时间配置分析

数字取证的一个重要部分是重建过去的事件。这个*数字考古学*依赖于理解时间的概念，特别是在 Linux 环境中的应用。

#### *时间格式*

Linux 中时间的标准表示方式源自 Unix。最初的 Unix 开发者需要一种紧凑的方式来表示当前的时间和日期。他们选择了 1970 年 1 月 1 日 00:00:00 UTC 作为时间的起点（恰好与 Unix 的命名时间相符），从这个时刻起的秒数表示特定的时间和日期。这个日期也被称为*Unix 纪元*，这种格式使得时间和日期能够以 32 位数值存储。

我们将指定的时间点称为*时间戳*。以下示例显示了使用 Linux `date` 命令的秒级时间：

```
$ date +%s
1608832258
```

这个时间戳是以文本格式给出的，但也可以以大端或小端格式存储为二进制格式。这个相同的字符串用十六进制表示是一个四字节字符串：0x5fe4d502。

使用 32 位纪元时间表示的一个问题是，在时钟重置为零之前的最大秒数。这个重置将发生在 2038 年 1 月 18 日，类似于 Y2K（2000 年 1 月 1 日的重置）事件。Linux 内核开发者已经意识到这个问题，并已实现对 64 位时间戳的支持。

原始 Unix 时间表示的另一个问题是其精度，仅限于一秒。这种限制对于早期计算机的较慢速度来说足够了，但现代系统需要更高的精度。常见的表示秒的分数单位有：

**毫秒** 一千分之一秒（0.001）

**微秒** 一百万分之一秒（0.000001）

**纳秒** 十亿分之一秒（0.000000001）

以下示例显示了从纪元以来的秒数，并具有纳秒级精度：

```
$ date +%s.%N
1608832478.606373616
```

为了保持向后兼容性，一些文件系统在时间戳中添加了一个额外的字节。这个字节中的各个位被分配给解决 2038 问题和提供更高精度的功能。

**注意**

*当你在进行取证分析工作时，训练自己注意那些可能是时间戳的数字字符串。例如，如果你看到一个以 16 开头的 10 位数字（16*XXXXXXXX*），它可能是一个时间戳（2020 年 9 月到 2023 年 11 月）。*

显示人类可读时间的格式是可定制的。该格式可以是长格式、短格式、数字格式，或这三者的组合。地区差异也可能导致混淆。例如，1/2/2020 可能是 2 月 1 日或 1 月 2 日，具体取决于地区。即使分隔符也因地区或风格而异（“.” 或 “/” 或 “-”）。

1988 年，ISO 制定了一个全球标准的日期格式，定义了年份、月份和日期的顺序：2020-01-02。如果你的取证工具支持这一格式（它可能支持），我建议使用这种格式。图 9-1 中的 XKCD 漫画可能帮助你记住这一点。

![Image](img/ch09fig01.jpg)

*图 9-1: XKCD 时间格式 (* [`xkcd.com/1179/`](https://xkcd.com/1179/)*)*

有两个标准对于理解时间格式很有用：ISO 8601 (*[`www.iso.org/iso-8601-date-and-time-format.html`](https://www.iso.org/iso-8601-date-and-time-format.html)*) 和 RFC 3339 (*[`datatracker.ietf.org/doc/html/rfc3339/`](https://datatracker.ietf.org/doc/html/rfc3339/)*)。在进行数字取证时，尤其是日志文件分析时，请确保你理解所使用的时间格式。

#### *时区*

地球被划分为 24 个主要时区，相差一小时。^(1) 时区表示一个地理区域及其与协调世界时（UTC）的时间偏差。时区可以应用于系统或用户，如果用户远程登录，这些时区不一定相同。

当系统首次安装时，系统所有者会指定一个时区。此设置是 */etc/localtime* 的符号链接（symlink），指向位于 */usr/share/zoneinfo/* 的 *tzdata* 文件。确定系统配置的时区，只需识别该文件的链接位置。在以下示例中，系统配置为欧洲地区和苏黎世市：

```
$ ls -l /etc/localtime
lrwxrwxrwx 1 root root 33 Jun 1 08:50 /etc/localtime -> /usr/share/zoneinfo/Europe/Zurich
```

此配置提供了机器物理位置的指示（或者至少是地区）。系统时区与用户登录时的时区之间的差异是值得注意的，因为它表明系统所有者的潜在位置（使用远程安装/管理的系统）。

对于像桌面 PC 和服务器这样的固定位置系统，配置的时区通常是静态的。经常更改时区的笔记本电脑表明用户可能正在旅行。时区的变化（无论是手动还是自动）可以在日志中看到：

```
Dec 23 03:44:54 pc1 systemd-timedated[3126]: Changed time zone to 'America/Winnipeg' (CDT).
...
Dec 23 10:49:31 pc1 systemd-timedated[3371]: Changed time zone to 'Europe/Zurich' (CEST).
```

这些日志展示了使用 GNOME 日期和时间图形界面更改时区的示例。请求 `systemd-timedated` 守护进程更改时区并更新 */etc/localtime* 的符号链接。如果设置为自动更改，系统将查询 GeoClue 以获取位置。GeoClue 是 Linux 的地理位置服务（稍后在本章中将描述）。

个别用户也可以指定与系统时区不同的登录时区——例如在多个全球用户通过安全外壳（SSH）远程登录的服务器上。要识别个别用户的时区，可以查找`TZ`环境变量的赋值。`TZ`变量可能出现在 shell 启动文件中（如*.bash_login*、*.profile*等），或者作为由 SSH 程序传递的变量。要确定 SSH 是否传递了`TZ`变量，请检查 SSH 服务器配置（*sshd_config*）是否显式允许`TZ`，通过`AcceptEnv`参数，或者检查客户端配置（*ssh_config*或*./ssh/config*）是否显式传递了`TZ`，通过`SendEnv`参数。

TZ 变量是一个 POSIX 标准，并通过 GNU C 库在 Linux 中实现。TZ 变量有三种格式，以下是一些示例：

**时区和偏移** CET+1

**带夏令时的时区和偏移** EST+5EDT

**时区文件名** Europe/London

你可以在*[`www.gnu.org/software/libc/manual/html_node/TZ-Variable.html`](https://www.gnu.org/software/libc/manual/html_node/TZ-Variable.html)*找到关于 TZ 变量的更详细描述。

在 Fedora 和 SUSE 系统中，一些软件包和脚本可能会读取*/etc/sysconfig/clock*文件（如果该文件存在）。该文件描述了硬件时钟（如果是 UTC，时区等）。

在使用取证工具分析时间戳时，工具可能要求指定时区。例如，在使用 The Sleuth Kit 时，带有时区信息的命令可以使用`-z`标志来指定时区。

#### *夏令时与闰时间*

夏令时是将时钟在春季提前一小时，在秋季延后一个小时（“春天提前，秋天延后”）的做法，目的是在冬季提供更早的日光，并在夏季提供更晚的日光。这一做法由各地区政府决定，并不是全球标准。一些地区（例如 2014 年的俄罗斯和 2021 年的欧洲）已经废除或正在废除夏令时的调整。

在对受影响地区的系统进行取证分析时，了解夏令时的变化是很重要的。增加或减少的小时数会影响取证时间线的重建和过去事件的解释。取证工具通常支持夏令时调整，如果指定了地理区域。UTC 时间不会因为夏令时而改变。

上一节中描述的*tzdata*文件包含了夏令时信息。要提取某一时区的时间间隔列表（包括历史和未来的），可以在 Linux 机器上使用`zdump`工具，如下所示：

```
$ zdump -v Europe/Paris |less 
...
Europe/Paris Sun Mar 31 00:59:59 2019 UT = Sun Mar 31 01:59:59 2019 CET isdst=0 gmtoff=3600
Europe/Paris Sun Mar 31 01:00:00 2019 UT = Sun Mar 31 03:00:00 2019 CEST isdst=1 gmtoff=7200
Europe/Paris Sun Oct 27 00:59:59 2019 UT = Sun Oct 27 02:59:59 2019 CEST isdst=1 gmtoff=7200
Europe/Paris Sun Oct 27 01:00:00 2019 UT = Sun Oct 27 02:00:00 2019 CET isdst=0 gmtoff=3600
Europe/Paris Sun Mar 29 00:59:59 2020 UT = Sun Mar 29 01:59:59 2020 CET isdst=0 gmtoff=3600
Europe/Paris Sun Mar 29 01:00:00 2020 UT = Sun Mar 29 03:00:00 2020 CEST isdst=1 gmtoff=7200
Europe/Paris Sun Oct 25 00:59:59 2020 UT = Sun Oct 25 02:59:59 2020 CEST isdst=1 gmtoff=7200
Europe/Paris Sun Oct 25 01:00:00 2020 UT = Sun Oct 25 02:00:00 2020 CET isdst=0 gmtoff=3600
Europe/Paris Sun Mar 28 00:59:59 2021 UT = Sun Mar 28 01:59:59 2021 CET isdst=0 gmtoff=3600
Europe/Paris Sun Mar 28 01:00:00 2021 UT = Sun Mar 28 03:00:00 2021 CEST isdst=1 gmtoff=7200
Europe/Paris Sun Oct 31 00:59:59 2021 UT = Sun Oct 31 02:59:59 2021 CEST isdst=1 gmtoff=7200
Europe/Paris Sun Oct 31 01:00:00 2021 UT = Sun Oct 31 02:00:00 2021 CET isdst=0 gmtoff=3600
Europe/Paris Sun Mar 27 00:59:59 2022 UT = Sun Mar 27 01:59:59 2022 CET isdst=0 gmtoff=3600
Europe/Paris Sun Mar 27 01:00:00 2022 UT = Sun Mar 27 03:00:00 2022 CEST isdst=1 gmtoff=7200
Europe/Paris Sun Oct 30 00:59:59 2022 UT = Sun Oct 30 02:59:59 2022 CEST isdst=1 gmtoff=7200
Europe/Paris Sun Oct 30 01:00:00 2022 UT = Sun Oct 30 02:00:00 2022 CET isdst=0 gmtoff=3600
...
```

这里显示了过渡时间、时区缩写（CET 或 CEST）、当前夏令时标志（`isdst=`）和与 UTC 的偏移量（以秒为单位的`gmtoff=`）。

有趣的是，那些放弃夏令时的地区，*tzdata* 文件中的最后一项是该地区最后一次变更的日期和时间。

有关 *tzdata* 文件的更多信息，请参见 tzfile(5) 手册页。时区数据的权威来源是互联网号码分配局 (IANA)，tz 数据库文件可以在 IANA 网站上找到 (*[`www.iana.org/time-zones/`](https://www.iana.org/time-zones/)*).

闰年和闰秒也是 Linux 时间管理中的一个因素，并且在法医分析中也是一个挑战。闰年是每四年增加一天，即 2 月 29 日（每个世纪会有一次闰年规则的例外）。闰秒更难预测，通常是由于地球自转速度减慢引起的。国际地球自转服务 (IERS) 决定何时添加闰秒，并提前半年发布该决定（通常计划在年底或年中）。自 Unix 纪元以来的闰秒列表（截至目前已有 28 次）可以在 IERS 网站上找到 (*[`hpiers.obspm.fr/iers/bul/bulc/ntp/leap-seconds.list`](https://hpiers.obspm.fr/iers/bul/bulc/ntp/leap-seconds.list)*). 使用外部时间同步的 Linux 系统会自动添加闰秒。闰年是可以预测的，Linux 系统设计为每四年自动增加 2 月 29 日。

在进行法医分析时，了解闰年和闰秒非常重要。额外的那一天和那一秒可能会影响过去事件的重建和法医时间线的创建。

#### *时间同步*

从数字取证的角度来看，了解配置的时间同步非常重要，原因有几个。它有助于确定系统何时同步或不同步，从而提供更准确的系统时间线分析。当时钟因恶意原因被故意更改或篡改时，它也有助于调查。

为了在正常系统操作期间保持正确的时间，会使用外部时间源。外部时间源的例子包括：

**网络时间协议 (NTP)** 基于网络的时间同步协议 (RFC 5905)

**DCF77** 德国长波广播时间信号，来自法兰克福附近（在欧洲广泛使用）

**全球定位系统 (GPS)** 从卫星网络接收的时间

大多数 Linux 系统在启动时会检查并设置日期，在网络正常工作的情况下使用 NTP。

在 Linux 系统中最常用的 NTP 软件包有：

**ntp** 原始的 NTP 参考实现 (*[`ntp.org/`](https://ntp.org/)*)

**openntpd** 由 OpenBSD 社区设计，注重简洁性和安全性

**chrony** 旨在在各种条件下表现良好

**systemd-timesyncd** 内置于 systemd 的时间同步

要确定使用了哪种 ntp 机制，请检查已安装的软件包，如 ntp、openntpd 或 chrony（systemd-timesync 是作为 systemd 的一部分安装的）。然后，检查通过查看*/etc/systemd/system/*.wants/*目录中的符号链接，来查看哪个服务单元文件已启用。常见的单元文件包括*ntp.service*、*ntpd.service*、*chrony.service*和*openntpd.service*。

Systemd 的 timesyncd 会创建符号链接，如*/etc/systemd/system/dbus-org.freedesktop.timesync1.service*和*/etc/systemd/system/sysinit.target.wants/systemd-timesyncd.service*。在实时系统中，`timedatectl`命令用于查询和管理这些文件。

单元文件的内容提供了关于配置的信息。通常，时间守护进程会有一个单独的配置文件位于*/etc/*（例如*ntp.conf*或*ntpd.conf*），该文件定义了守护进程的行为并指定了使用的时间服务器。systemd-timesyncd 的配置文件位于*/etc/systemd/timesyncd.conf*。

与时间守护进程相关的日志提供了有关启动、关闭、时间同步变化和错误的信息。这些日志可以在 systemd 日志、syslog 日志以及*/var/log/**中的独立日志文件中找到。

以下示例展示了来自 openntpd、chrony 和 systemd-timesyncd 的日志条目，其中时间被更改：

```
Aug 01 08:13:14 pc1 ntpd[114535]: adjusting local clock by -57.442957s
...
Aug 01 08:27:27 pc1 chronyd[114841]: System clock wrong by -140.497787 seconds,
adjustment started
...
Aug 01 08:41:00 pc1 chronyd[114841]: Backward time jump detected!
...
Aug 01 09:58:39 pc1 systemd-timesyncd[121741]: Initial synchronization to
time server 162.23.41.10:123 (ntp.metas.ch).
```

系统通常会配置一个服务器列表用于时间同步。在某些情况下，系统可能会有一个本地附加的时间源（如 DCF77、GPS 等），并且在配置文件中可能会以 127.*x*.*x*.*x* IP 地址的形式出现作为服务器。你可以在软件包的手册页或开发者网站上找到有关时间守护进程和配置文件的更多信息。

如果连接了 GPS 设备，请查找 gpsd (*[`gpsd.io/`](https://gpsd.io/))*软件包及其相关配置（*/etc/gpsd/**或*/etc/default/gpsd*）。

时钟同步是典型的，但并非必须，某些情况下可能不会找到 NTP 配置。例如：

+   信任主机时钟的虚拟机（例如，具有准虚拟化硬件时钟的虚拟机）

+   用户手动设置时钟的机器

+   在启动时（或定期）运行`ntpdate`命令来设置时钟的机器

在这种情况下，虚拟机的主机同步或主板上硬件时钟的时间变得非常重要。

大多数 PC 主板有一个小电池，可以在系统关闭时保持时钟运行。Linux 内核的实时时钟（RTC）驱动程序使时钟通过*/dev/rtc*设备（通常是指向*/dev/rtc0*的符号链接）可访问。时间同步软件会相应地保持硬件时钟的更新。

系统的硬件时钟可以设置为本地时间或 UTC 时间（推荐使用 UTC）。有关更多信息，请参阅 hwclock(8)手册页。

##### 树莓派时钟

树莓派没有时钟电池，启动时时间戳从零开始（1970 年 1 月 1 日 00:00:00）。在树莓派的时间与标准时间同步之前生成的任何日志都会有不正确的时间戳。在分析带有时间戳的内容时，了解系统时间同步何时建立了正确的时间非常重要。

树莓派和其他嵌入式系统可能会在关机时保存时间戳，以便在早期启动时设置一个更合理的时间（直到时间同步）。这是通过使用*fake-hwclock*软件包来实现的。时间存储在一个文件中，如以下示例所示：

```
# cat /etc/fake-hwclock.data
2020-03-24 07:17:01
```

存储在*fake-hwclock.data*文件中的时间可能是 UTC 格式，并与相应的文件系统时间戳（最后修改和更改）匹配。定期的 cron 作业可能会更新写入文件的时间，以防出现意外崩溃或电源丢失。有关更多信息，请参见 fake-hwclock(8) 手册页。

#### *时间戳与取证时间线*

时间戳指的是一个特定的时间点，通常与某个操作或活动相关，并且该操作或活动有数字证据。使用时间戳进行取证有助于重建过去事件的顺序。然而，使用和信任从数字数据源提取的时间戳存在挑战。一些影响时间戳准确性的风险包括：

+   无时间同步的机器上的时钟漂移或偏移

+   非实时操作系统的延迟和时延

+   未知时区的时间戳

+   反取证或恶意修改时间戳（例如使用`timestomp`）

涉及多个时区、多个设备的全球性调查在时间戳受到这些风险影响时变得更加复杂。

大多数取证工具都意识到这些问题，并包括调整时间的功能。例如，Sleuth Kit 具有帮助调整时间的标志：

-s 秒     调整 +/- 秒

-z 区域     指定一个时区（例如 CET）

永远不要完全信任时间戳。错误、故障或反取证活动是始终可能发生的，因此尽量与其他设备上的时间戳或其他证据来源进行核对。

取证时间线是基于与调查相关的时间戳重建事件的过程。最早的数字取证时间线是从文件系统元数据的时间戳（最后访问、修改、改变等）创建的。如今，调查人员将来自多个来源的时间戳数据汇集到一个单一的*超级时间线*中，可能包含任何相关的时间戳，例如：

+   文件系统时间戳（MACB）

+   日志（系统日志、systemd 日志和应用程序日志）

+   浏览器历史记录、Cookies、缓存和书签

+   包含时间戳的配置数据

+   回收/垃圾数据

+   电子邮件及附件（mbox、maildir）

+   办公文档元数据（PDF、LibreOffice 等）

+   EXIF 数据（来自照片或视频的元数据）

+   易失性输出文件（内存取证）

+   捕获的网络流量（PCAP 文件）

+   CCTV 摄像头和建筑物门禁系统（刷卡读卡器）

+   电话、聊天和其他通信记录

+   备份档案（tar *.snar* 文件和备份索引）

+   其他时间戳来源（手机、物联网设备或云端）

一个流行的超时间轴框架是 log2timeline/plaso，它使用自由和开源工具从各种来源组装时间戳。你可以访问该项目网站 (*[`github.com/log2timeline/plaso/`](https://github.com/log2timeline/plaso/)*) 获取更多信息。

每个 Linux 镜像的取证时间线包含几个重要的时间节点：

+   Unix 纪元

+   安装前存在的文件（发行版提供的文件）

+   原始系统安装时间

+   正常操作期间观察到的最后时间戳

+   取证采集时间

取证采集后不应出现任何时间戳。如果有时间戳，它们可能表明驱动器镜像被篡改或修改。采集后的日期也可能是通过反取证活动故意创建（伪造）的。

构建和解释时间线会面临一些挑战。在大型技术数据集中，可用的时间戳数量可能会难以处理（尤其是手动处理时）。许多时间戳描述的是琐碎或不相关的事件。有时，多个时间戳的集合描述的是单一的整体事件。

另一个挑战是确定某个事件是由用户还是机器引起的。特别是对于文件系统取证而言，需要注意的是，我们回溯时间线时，往往会发现信息越来越少。随着时间的推移，扇区被覆盖，文件系统时间戳被更新，其他信息在正常系统操作过程中丢失。

### 国际化

Linux 系统的国际化包括配置区域设置、语言、键盘和其他地区特定信息。涉及人员识别（也称为归属）的全球性调查可以大大受益于了解 Linux 系统上的本地区域工件。

Linux 国际化指的是对多语言和文化设置的支持。*国际化*一词有时会缩写为 *i18n*，因为在 *i* 和 *n* 之间有 18 个字符。

在基于 Fedora 和 SUSE 的系统中，一些包和脚本可能会读取 */etc/sysconfig/* 目录下的 i18n、键盘、控制台和语言文件（如果存在）。基于 Debian 的系统在 */etc/default/* 目录中有类似的键盘、硬件时钟、控制台设置和语言区域文件。

这些文件可以在取证调查中进行检查，但它们已经部分被这里描述的 systemd 等效物所取代。

#### *区域设置和语言设置*

Linux 的大部分国际化配置通过定义语言环境设置来完成。语言环境是 glibc 的一部分，可以被任何支持语言环境的软件使用来控制语言、格式和其他区域设置。这些设置定义在*/etc/locale.conf*文件中，该文件可能不存在（如果系统使用其他默认设置），可能只包含一行（例如，语言设置），或者包含详细的语言环境配置：

```
$ cat /etc/locale.conf
LANG="en_CA.UTF-8"
```

在这里，语言设置为加拿大英语（Unicode）。语言环境定义文件描述了日期格式、货币和其他本地信息。可用语言环境的定义可以在*/usr/share/i18n/locales*中找到，并存储为可读的文本文件。

在某些系统上，locale-gen 程序会生成*/etc/locale.gen*中指定的所有语言环境，并将其安装在*/usr/lib/locale/locale-archive*中，供系统上的任何用户使用。`localedef`工具可以列出文件中的语言环境：

```
$ localedef --list-archive -i /usr/lib/locale/locale-archive
de_CH.utf8
en_CA.utf8
en_GB.utf8
en_US.utf8
fr_CH.utf8
```

输出应与*/etc/locale.gen*文件中的配置相对应。该文件可以复制到单独的检查机器上进行离线分析（使用`-i`标志）。

从用户的角度看，语言环境是定义其本地或区域偏好的变量集合。在运行中的系统上，`locale`命令列出了这些变量：

```
$ locale
LANG=en_US.UTF-8
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
LC_ALL=
```

这些变量决定了语言、数字格式（例如，使用逗号代替句点）、时间（24 小时制与 AM/PM 制）、货币、纸张大小、姓名和地址格式、测量单位等。有些变量由 POSIX 定义，其他则由 Linux 社区添加。在事后法医检查中，我们可以通过配置文件重建这些偏好设置。

有关这些变量的更多信息，请参阅 locale(5)手册页（有三个 locale 手册页，分别位于不同的章节：locale(1)、locale(5)和 locale(7)，因此请确保查阅正确的版本）。

用户还可以创建一个混合语言环境，该语言环境由多个已安装的语言环境的变量组成（例如，北美英语语言与欧洲时间设置的结合）。

如果用户没有在（shell 启动脚本中）定义任何变量，则使用系统范围的默认语言环境，该默认语言环境定义在*/etc/locale.conf*文件中。Systemd 使用`localectl`工具来管理本地化，并在系统启动时读取*locale.conf*。系统管理员和用户明确设置的任何本地化设置都是有价值的，可能对调查有所帮助。例如，一些设置的混合可能表明某个人讲某种语言，但居住在另一个国家。

大多数国际化软件项目都包括对多种语言的支持，用于互动消息、错误消息、帮助页面、文档以及传达给用户的其他信息。当软件包提供单独的语言文件时，这些文件存储在*/usr/share/locale/*目录下，并根据配置的语言动态选择。`LANG=`变量指定要使用的语言，可以是系统默认语言，也可以为每个用户单独配置。

图形环境可能有额外的或单独的语言信息和配置设置（例如，KDE 的`KDE_LANG`变量或 GNOME 的 dconf 数据库中的设置）。XDG **.desktop** 文件通常会在文件中定义语言翻译字符串。有些应用程序需要单独安装语言包（例如，字典、办公程序和手册页）。

#### *物理键盘布局*

物理系统连接的键盘很有趣，因为它可以告诉我们使用者的一些信息。键盘的国家和语言暗示了用户的文化背景（不过，许多非英语的 Linux 程序员和爱好者选择使用美国英语键盘）。键盘的设计也可能提供关于用户如何使用机器的信息。有游戏键盘、程序员/系统管理员键盘、人体工学键盘、触摸屏键盘、收藏键盘和其他异国情调的键盘设计。这些物理键盘特征在法医学检查中可能是有用的背景信息。

分析键盘的第一步是识别物理连接的设备。USB 键盘的制造商和产品信息可以在内核日志中找到：

```
Aug 01 23:30:02 pc1 kernel: usb 1-6.3: New USB device found, idVendor=0853,
idProduct=0134, bcdDevice= 0.01
Aug 01 23:30:02 pc1 kernel: usb 1-6.3: New USB device strings: Mfr=1,
Product=2, SerialNumber=0
Aug 01 23:30:02 pc1 kernel: usb 1-6.3: Product: Mini Keyboard
Aug 01 23:30:02 pc1 kernel: usb 1-6.3: Manufacturer: LEOPOLD
```

这里，`idVendor`是`0853`，表示 Topre（参见 *[`www.linux-usb.org/usb-ids.html`](http://www.linux-usb.org/usb-ids.html)*），`Manufacturer`是`LEOPOLD`，产品（`0134`）被描述为`迷你键盘`。

虚拟机没有物理键盘（除非将物理 USB 键盘直接传递到虚拟机），虚拟键盘可能会显示为 PS/2 设备：

```
[    0.931940] i8042: PNP: PS/2 Controller [PNP0303:KBD,PNP0f13:MOU]
at 0x60,0x64 irq
[    0.934092] serio: i8042 KBD port at 0x60,0x64 irq 1
[    0.934597] input: AT Translated Set 2 keyboard as
/devices/platform/i8042/serio0/input/input0
```

键盘的电子/数字硬件接口是通用的，与语言无关。Linux 系统必须手动配置，以映射物理按键帽上显示的特定语言布局和符号。这个配置可以为控制台环境和图形环境分别进行。

物理键盘生成的低级扫描码会被内核转换为键码。这些键码在用户空间（无论是控制台环境还是图形环境）中被映射为键符（keysyms），即人类语言中的字符（字形）。可用的字符集存储在*/usr/share/i18n/charmaps/*目录下，格式为压缩文本文件。系统范围的字符集可以定义为默认字符集，用户也可以在登录时选择自己的字符集。

Linux 系统将早期的 Unix 串行端口替换为虚拟控制台，在这些控制台上连接键盘、鼠标和显示器。这些控制台是没有启动图形环境时可用的文本接口，通常在启动时或在服务器系统上看到。控制台键盘（和字体）可以在*/etc/vconsole.conf*中通过`KEYMAP=`选项进行配置。

如果使用图形环境，键盘配置描述了模型、语言和其他选项。KDE 将此信息存储在用户主目录下的*.config/kxkbrc*文件中。例如：

```
[Layout]
DisplayNames=,
LayoutList=us,ch
LayoutLoopCount=-1
Model=hhk
Options=caps:ctrl_modifier
...
```

在这里，使用的是 Happy Hacking Keyboard（`hhk`），可用的语言布局是`us`和`ch`（瑞士），并且其他选项已被指定（CAPS LOCK 被重新映射为 CTRL 键）。

GNOME 将键盘信息存储在 dconf 数据库中的*org.gnome.libgnomekbgd*键下。有关如何分析 dconf 数据库，请参见第十章。

如果使用了 systemd 或`localectl`命令（无论是手动还是在脚本中）来设置配置，键盘配置将保存在*/etc/X11/xorg.conf.d/00-keyboard.conf*文件中：

```
$ cat /etc/X11/xorg.conf.d/00-keyboard.conf
# Written by systemd-localed(8), read by systemd-localed and Xorg. It's
# probably wise not to edit this file manually. Use localectl(1) to
# instruct systemd-localed to update it.
Section "InputClass"
        Identifier "system-keyboard"
        MatchIsKeyboard "on"
        Option "XkbLayout" "ch"
        Option "XkbModel" "hhk"
        Option "XkbVariant" "ctrl:nocaps,altwin:swap_lalt_lwin"
EndSection
```

在这里，另一款 Happy Hacking Keyboard（`hhk`）配置为瑞士（`ch`）布局。

其他窗口管理器和图形环境也可能使用 dconf 或拥有自己的配置文件。基于 Debian 的系统可能将这些信息作为变量存储在*/etc/default/keyboard*文件中，格式如下：

```
$ cat /etc/default/keyboard
# KEYBOARD CONFIGURATION FILE

# Consult the keyboard(5) manual page.

XKBMODEL="pc105"
XKBLAYOUT="us"
XKBVARIANT=""
XKBOPTIONS="ctrl:nocaps"
```

XKB 指的是来自 X11 规范的*X 键盘扩展*。有关键盘模型、布局和选项的列表，请参见 xkeyboard-config(7)手册页。一些 Wayland 合成器也将使用这些`XKB*`变量来配置键盘（例如，Sway WM）。

### Linux 与地理位置

在法医调查中，回答地理上的“哪里？”问题需要重建 Linux 设备随时间变化的物理位置。如果一台设备被盗或丢失后被找回，那么在这段时间内它的位置在哪里？如果设备被扣押或隔离以供调查，那么与事件相关的设备位置历史是什么？我们可以通过地理位置分析来尝试回答这些问题。

手持移动设备因其位置感知功能而广为人知，主要得益于硬件中实现的 GPS。Linux 系统通常安装在没有内建 GPS 的通用 PC 上。然而，仍然可以找到指示地理位置的法医证据。在某些情况下，地理位置数据也可能通过其他来源（外部于所检查的法医镜像）推导或推测得出。

对于位置的引用可能有几种不同的上下文，包括：

**全球上下文** 纬度和经度（GPS 坐标）

**区域上下文** 文化或政治区域（区域设置，键盘）

**组织上下文** 校园、建筑、办公室或桌面（IT 资产）

这些位置参考信息可能是通过对系统或系统所在基础设施的法医分析确定或推断出来的。

#### *地理位置历史*

位置历史是物体在一段时间内改变空间位置的记录。为了重建位置历史，我们需要物理位置数据以及时间戳。知道一个物理位置发生变化的时间帮助我们建立位置时间轴。这里描述的许多思想不仅限于 Linux 系统，也可能适用于其他操作系统。

键盘、语言和其他区域设置提供了一个广泛的地区位置指示。例如，知道默认的纸张大小是美国信纸或 A4，可以判断系统是否来自北美地区。如果一个系统有瑞士键盘和德国语言，这意味着它来自瑞士的德语区。如果纸张大小或键盘在某个（已知的）时间发生了变化，这可能表明地区发生了变化。

时间和时区的变化可能是旅行的潜在指标。如果一个系统突然改变了它的时区设置（如之前日志中所示），这表明位置发生了变化。改变的时区数量也可能很有意思，因为它可能暗示了某种旅行方式（飞机与汽车）。

对时区切换前后的时间戳进行分析也可能很有意思。时区变化前，时间戳活动是否有显著间隔？还是时间戳显示该人在时区变化期间一直在工作？

在某种程度上，IP 地址可以提供大致的地理位置。这种确定位置的方法有时称为 *IP 地理定位* 或 *Geo-IP* 查找。IP 范围分配给区域互联网注册机构（RIRs），它们将这些范围委托给指定区域使用。五个 RIR（及其成立日期）是：

+   RIPE NCC, RIPE 网络协调中心（1992）

+   APNIC，亚太网络信息中心（1993）

+   ARIN，美国互联网号码注册局（1997）

+   LACNIC，拉丁美洲和加勒比互联网地址注册局（1999）

+   AfriNIC，非洲网络信息中心（2004）

国家互联网注册局（NIRs）和本地互联网注册局（LIRs）可能会进一步将 IP 范围分配给地理区域。像 MaxMind 这样的公司（* [`www.maxmind.com/`](https://www.maxmind.com/) *）可能会从互联网注册局、互联网服务提供商（ISPs）和其他分析来源收集数据，制作 IP 查找数据库，并将其作为产品和服务出售。

**注意**

*使用隧道、转发、匿名化、移动网络、国际非公开网络或私人 IP 范围（RFC 1918）的设备的 IP 地理定位可能无法提供准确的结果。*

每当法医检查发现与时间戳关联的 IP 地址时，它就是位置历史时间线上的一个点。来自组织内部网络的 IP 地址可能提供更准确的位置信息（如网络配置文档、IT 库存数据库等）。

在链路层中，日志中发现的周围 MAC 地址可能是位置的指示器。网络段上本地路由器或其他固定位置设备的 MAC 地址可能有助于确定位置。例如，企业 IT 环境可能拥有基础设施 MAC 地址的清单，这些地址分配给物理建筑或办公室。存储在本地机器上的 Wi-Fi 基础设施（BSSID）日志或缓存也可能是地理位置的指示器。

在某些情况下，机器的 MAC 地址或其他唯一标识符可能会在无线基础设施提供商处被记录（例如，WWAN 移动设备连接到基站塔或 WLAN 无线接口连接到公共 Wi-Fi 热点）。

与固定蓝牙设备的连接可能表明一个物理位置（例如，证据显示笔记本电脑使用蓝牙与桌面电脑、家庭音响、键盘或打印机在已知地点连接）。与其他具有地理定位信息的移动设备的蓝牙连接可能有助于重建位置历史（例如，笔记本电脑连接到存储了 GPS 位置信息的手机或汽车）。

应用数据可能提供漫游 Linux 系统的过去位置。例如，许多提供商会在有人访问他们的网站时，存储包含地理定位信息的 cookies。此外，任何连接到远程服务的操作可能会在服务器日志中保留位置信息（假设这些日志可以可靠地与正在检查的机器关联）。在某些情况下，这些信息可以通过正式请求获取（例如传票或其他合法请求）。

地理定位信息常常出现在文件的元数据中（例如照片）。然而，这不一定表示 PC 的实际位置，而是指最初拍摄照片的设备的位置。

如果一台 Linux 系统配备了 GPS 设备，它很可能正在使用 gpsd 软件包。任何使用 gpsd 的程序或应用可能会有日志或缓存的位置信息。

台式电脑通常位于固定的物理位置。如果被扣押，位置是明确已知的（显然）。在法医报告中，其他信息可能也很重要，比如建筑物地址、房间号或开放式办公室中特定的桌子。在企业环境中，机器的物理位置可能会随着时间变化，位置历史可以通过 IT 库存中的变化来重建（如果存在并且追踪系统位置的变化）。

在某种程度上，我们还可以进入物理世界来确定某个电子设备的位置。例如，有些人收集贴纸并将它们贴在笔记本电脑的盖子上。人们这样做的原因有很多：便于识别自己的笔记本，防止盗窃，或宣传喜欢的产品、项目、会议或其他事物。笔记本电脑盖子上的贴纸创造了一个独特的视觉标识符，可以与 CCTV 摄像头录像或包含该笔记本的照片的地理位置标签相匹配。它们也可能与特定的会议和活动匹配，这些会议和活动上曾分发过这些贴纸。

#### *GeoClue 地理定位服务*

GeoClue 软件项目的初衷是通过 D-Bus 为位置感知应用程序提供位置信息。正如其官方网站上所记录的（*[`gitlab.freedesktop.org/geoclue/geoclue/`](https://gitlab.freedesktop.org/geoclue/geoclue/)*），它从以下途径获取位置信息：

+   基于 Wi-Fi 的地理定位，使用 Mozilla Location Service（精度为码/米）

+   GPS(A) 接收器（精度为英寸/厘米）

+   本地网络上其他设备的 GPS，例如智能手机（精度为英寸/厘米）

+   3G 调制解调器（精度为英里/公里，除非调制解调器具备 GPS 功能）

+   GeoIP（城市级别精度）

GeoClue 最初是为 GNOME 应用程序编写的，但它是一个 D-Bus 服务，任何在 GeoClue 配置文件中授权的应用程序都可以使用它。

GeoClue 的配置文件定义了使用哪些位置源，以及哪些本地应用程序被允许请求位置信息：

```
$ cat /etc/geoclue/geoclue.conf
# Configuration file for Geoclue
...
# Modem GPS source configuration options
[modem-gps]

# Enable Modem-GPS source
enable=true

# WiFi source configuration options
[wifi]

# Enable WiFi source
enable=true
...
[org.gnome.Shell]
allowed=true
system=true
users=
...
[firefox]
allowed=true
system=false
users=
```

守护进程本身不会记录位置信息；然而，使用它的应用程序可能会记录或存储这些信息。

使用位置服务的偏好存储在用户的 dconf 数据库中（*org.gnome.system.location.enabled*）。这个偏好设置与 `geoclue` 服务是否正在运行无关。如果用户在图形界面设置中禁用了位置服务，`geoclue` 服务不会被全局禁用。要确定 GeoClue 是否已启用，需要检查 systemd 的 *geoclue.service* 文件是否存在。

### 摘要

本章描述了如何分析 Linux 系统中的时间相关元素。它探讨了 Linux 的国际化功能及其在取证调查中的应用。它还考虑了在 Linux 取证分析中的地理定位问题。本章已涉及用户活动和行为，下一章将更深入地探讨这个话题。
