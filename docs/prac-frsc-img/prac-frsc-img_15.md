## 第十五章：脚注

### 第零章：数字法医概述

1. Gary Palmer, “数字法医研究路线图。” 数字法医研究研讨会 (DFRWS)，2001 年。技术报告 DTR-T0010-01，纽约尤蒂卡。2. *[`www.dfrws.org/about-us/`](http://www.dfrws.org/about-us/)*3. *[`utica.edu/academic/institutes/ecii/publications/articles/A04BC142-F4C3-EB2B-462CCC0C887B3CBE.pdf`](https://utica.edu/academic/institutes/ecii/publications/articles/A04BC142-F4C3-EB2B-462CCC0C887B3CBE.pdf)*4. “硬盘驱动器的法医成像—我们曾经认为我们知道的”，法医焦点，2012 年 1 月 27 日，*[`articles.forensicfocus.com/2012/01/27/forensic-imaging-of-hard-disk-drives-what-we-thought-we-knew-2/`](http://articles.forensicfocus.com/2012/01/27/forensic-imaging-of-hard-disk-drives-what-we-thought-we-knew-2/)*。

### 第一章：存储介质概述

1. Jeff Hedlesky, “固态硬盘法医取证的进展” (报告, CEIC2014, 拉斯维加斯, NV, 2014 年 5 月 19 日至 22 日).2. Bruce Nikkel, *数字调查* 16 (2016): 38-45, doi:10.1016/j.diin.2016.01.001.3. LAN/WAN 层次网络通过开放系统互联（OSI）模型的七层抽象来描述网络通信。4. 磁盘驱动器的 AT 附加接口，ANSI X3.221-199x，修订版 4c，X3T10，1994 年。5. ATA/ATAPI 命令集-2（ACS-2），修订版 2。6. Mayank R. Gupta, Michael D. Hoeschele 和 Marcus K. Rogers, “隐藏磁盘区域：HPA 和 DCO,” *国际数字证据期刊* 5, 第 1 期 (2006), *[`www.utica.edu/academic/institutes/ecii/publications/articles/EFE36584-D13F-2962-67BEB146864A2671.pdf`](https://www.utica.edu/academic/institutes/ecii/publications/articles/EFE36584-D13F-2962-67BEB146864A2671.pdf)*。

### 第二章：Linux 作为法医采集平台

1. Erin Kenneally, “Gatekeeping Out of the Box: Open Source Software as a Mechanism to Assess Reliability for Digital Evidence,” *弗吉尼亚法律与技术期刊* 6, 第 13 期 (2001).2. Brian Carrier, “Open Source Digital Forensic Tools: The Legal Argument” [技术报告] (Atstake 公司, 2002 年 10 月).3. FUSE 是一个用户空间文件系统实现（参见 *[`en.wikipedia.org/wiki/Filesystem_in_Userspace`](https://en.wikipedia.org/wiki/Filesystem_in_Userspace)*）。4. 关于将*GNU*与*Linux*合并的命名争议，参见 *[`en.wikipedia.org/wiki/GNU/Linux_naming_controversy`](https://en.wikipedia.org/wiki/GNU/Linux_naming_controversy)*。

### 第三章：法医图像格式

1。Philip Turner，“来自不同来源的数字证据统一（数字证据包）”（论文发表于数字取证研究研讨会[DFRWS]，路易斯安那州新奥尔良，2005 年 8 月 18 日）。*[`dfrws.org/2005/proceedings/turner_evidencebags.pdf`](http://dfrws.org/2005/proceedings/turner_evidencebags.pdf)*。2。M.I. Cohen，Simson Garfinkel 和 Bradley Schatz，“扩展高级取证文件格式，以容纳多个数据源、逻辑证据、任意信息和法医工作流程，”*数字调查* 6（2009）：S57–S68。3。在基于 Debian 的系统上，使用 `apt-get install squashfs-tools` 安装此软件包。

### 第四章：规划与准备

1。此示例中的点可以解释为正则表达式。为了简化起见，这里忽略了这一点。2。GNU `cp` 命令还允许在复制过程中创建稀疏文件。3。在典型的 i7 PC 上使用两个 SATA3 硬盘，使用 dd 测试。4。法医采集还涉及数据的完整性和保存完整性。5。“Tableau Bridge Query—技术文档”，访问日期：2005 年 12 月 8 日，之前可以下载。欲了解更多信息，请联系 Guidance Software。

### 第五章：将目标媒体附加到采集主机

1. 从 `lsusb -v` 输出中，`Linux Foundation...root hub` 设备中的 `iSerial` 设备描述符将指向 USB 控制器的 PCI 设备地址。2. 可用的 SMART 统计数据和日志因硬盘厂商而异。3. 关于 HPA 和 DCO 区域的取证论文，见 Mayank R. Gupta、Michael D. Hoeschele 和 Marcus K. Rogers，“隐藏磁盘区域：HPA 和 DCO”，*国际数字证据杂志* 第 5 卷，第 1 期（2006）。4. 一些主板要求在 BIOS 中配置 SATA 端口以支持热插拔。5. 关于在服务扇区中隐藏数据的研究，见 Ariel Berkman，“在硬盘的服务区中隐藏数据”，Recover Information Technologies LTD，2013 年 2 月 14 日，* [`www.recover.co.il/SA-cover/SA-cover.pdf`](http://www.recover.co.il/SA-cover/SA-cover.pdf) *。6. Todd G. Shipley 和 Bryan Door，“硬盘驱动器的取证影像：我们曾经认为我们知道的”，*取证焦点*，2012 年 1 月 27 日，* [`articles.forensicfocus.com/2012/01/27/forensic-imaging-of-hard-disk-drives-what-we-thought-we-knew-2/`](http://articles.forensicfocus.com/2012/01/27/forensic-imaging-of-hard-disk-drives-what-we-thought-we-knew-2/) *。7. 联合测试行动小组（JTAG）定义了用于访问电子组件的标准化调试接口。8. 在撰写本文时，我没有测试访问任何支持多个命名空间的 NVME 驱动器。这些结论基于对标准和文档的阅读。

### 第六章：取证影像采集

1. 假设授权人已安装 GnuPG 并安全地生成了密钥对。2. PEM 最初在隐私增强邮件标准中定义，今天通常指用于存储 X.509 证书的文件格式。3. 也可以使用不同人提供的多个签名来降低密钥被窃取或某人恶意篡改的风险。4. 在某些系统中，这是一个 Perl 脚本，位于 */usr/lib/ssl/misc*。5. 在数据恢复行业中，这被称为 *捐赠盘*。6. Andrew S. Tanenbaum 的名言在这里是适用的：“永远不要低估一辆装满磁带的旅行车飞速驶过公路的带宽。”7. cdparanoia 是在 CD 驱动器的质量问题比今天的驱动器更多时开发的。8. Heinz Mauelshagen，“dmraid - 设备映射 RAID 工具：通过通用 Linux 设备映射器支持 ATARAID 设备。” 论文在 2005 年 7 月 20-23 日的渥太华 Linux 研讨会上发布。9. Linux md 驱动程序最初意味着 *镜像设备*，一些操作系统称之为 *元设备*。

### 第七章：取证影像管理

1。截至本文写作时，ewfacquire 的最新版本暂时禁用了 bzip2 支持（请参阅 libewf 软件包的*ChangeLog*文件中的 20160404 节）。2。也可以使用 DCO 来复制扇区大小。3。我是加拿大人，因此偏爱 RCMP 方法。:-)

### 第八章：特殊镜像访问主题

1。术语*slices*来源于 BSD UNIX，并且它是 UNIX 世界中常见的分区方案。
