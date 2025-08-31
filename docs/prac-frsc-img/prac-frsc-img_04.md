## 第四章：**取证镜像格式**

![image](img/common-01.jpg)

本章概述了今天常用的各种采集工具、证据容器和取证镜像格式。取证镜像格式和证据容器是存储取证采集镜像及其附加案件数据（如采集时间和持续时间、采集方式、大小、错误、哈希等）的结构。取证格式的附加功能通常包括压缩文件和加密。本章展示了使用几种取证格式的命令行取证任务。

你可以在数字取证研究研讨会（DFRWS）网站上找到一篇描述各种取证格式的介绍性论文，网址是 *[`www.dfrws.org/CDESF/survey-dfrws-cdesf-diskimg-01.pdf`](http://www.dfrws.org/CDESF/survey-dfrws-cdesf-diskimg-01.pdf)*。

你可以通过使用 Sleuth Kit 命令 `img_stat` 来识别本章中描述的常用取证格式：

```
# img_stat -i list
Supported image format types:
        raw (Single or split raw file (dd))
        aff (Advanced Forensic Format)
        afd (AFF Multiple File)
        afm (AFF with external metadata)
        afflib (All AFFLIB image formats (including beta ones))
        ewf (Expert Witness format (encase))
```

除了这些格式，本章还介绍了一种临时方法，使用 SquashFS 作为与标准取证工具配合使用的实际取证容器。

**注意**

*关于取证镜像的一个重要概念是，它们不是复制文件，而是复制磁盘扇区，从扇区 0 到磁盘上最后一个可访问的扇区。原始镜像的大小始终等于整个磁盘的大小，而与磁盘文件系统上存在的文件数量无关。*

### 原始镜像

原始镜像本身并不是一种格式，而是从证据源获取的一段原始数据。原始镜像除了包含有关镜像文件本身（如名称、大小、时间戳和其他信息）的元数据外，不包含任何额外的元数据。

提取原始镜像在技术上是直接的：它只是将字节序列从源设备传输到目标文件。这通常是没有任何转换或翻译的。

磁盘块复制工具，如 dd 和变种工具，最常用来提取原始镜像。以下各节将讨论这些工具。

#### *传统 dd*

创建原始镜像的最简单工具，也是最古老的工具，就是原始的 Unix dd 工具。它并非为证据收集而设计，但其简单的逐字节传输对于成像磁盘设备非常有用，因为它会对磁盘的每个扇区进行完整的低级别复制（保留文件系统结构、文件、目录和元数据）。然而，像日志记录、错误处理和哈希等功能要么不足，要么根本不存在；当没有更好的替代工具时，可以使用 dd。计算机取证工具测试（CFTT）项目测试了几个标准的 dd 版本。你可以在 CFTT 网站上找到测试结果，网址是 *[`www.cftt.nist.gov/disk_imaging.htm`](http://www.cftt.nist.gov/disk_imaging.htm)*。

dd 工具最初在 1970 年代的早期 UNIX 系统上创建，主要用于字节顺序转换和块复制。最初是为了将主机世界中的 EBCDIC 编码数据转换为 ASCII 编码，这在 UNIX 环境中更为适用。该程序简单地从源位置获取数据块，任选执行转换或变换，然后将这些数据块放置到指定的目标位置（另一设备或文件中）。现代版本的 dd 做了改进，使其能够用于从设备（如磁盘和磁带）中进行取证数据采集。

#### *取证 dd 变种*

由于原始的 dd 工具并非为取证环境设计，因此缺少某些功能。随后，基于 dd 的工具应运而生，加入了取证所需的功能，如：

• 加密哈希

• 改进的错误处理

• 日志记录

• 性能提升

• 验证检查

• 进度监控（取证影像可能需要数小时）

两个最常用的 dd 工具变种分别是 dcfldd，2002 年由尼古拉斯·哈伯在美国国防部计算机取证实验室（DCFL）创建，以及 dc3dd，2007 年由杰西·科恩布鲁姆在美国国防部网络犯罪中心（DC3）创建。

dcfldd 工具基于 GNU dd，新增了许多功能，如哈希、改进的日志记录和输出文件分割等。尽管自 2006 年以来没有更新，但该工具至今仍在使用。亚历山大·杜劳诺伊创建了一个 dcfldd 的补丁版本，包含了一些 Debian 的 bug 修复，你可以在* [`github.com/adulau/`](https://github.com/adulau/)*找到它。

更新版的 dc3dd 工具作为一个补丁实现，并且更容易跟随 GNU dd 的代码变化。该工具目前仍在维护，并且已做过最近的更新。它包含与 dcfldd 类似的取证功能，并实现了改进的日志记录和错误处理。

dcfldd 和 dc3dd 都源自传统的 dd，并具有相似的功能。尽管这两个工具都不内置支持写入取证格式（如 FTK、Encase、AFF）、压缩或图像加密，但你可以使用命令管道和重定向来完成这些任务。书中全程展示了这两个工具的使用示例。CFTT 的测试报告已经存在，涵盖了 dcfldd 和 dc3dd。

#### *数据恢复工具*

有几种数据恢复工具值得一提，因为它们具有强大的错误处理能力和积极的恢复方法。尽管这些工具并非专为取证设计，但在其他取证工具未能从严重损坏的介质中恢复数据时，它们可以派上用场。

GNU ddrescue 和 dd_rescue 名称相似，但它们是不同的工具，独立开发。截至本文撰写时，这两款工具仍在积极开发中，各自具有不同的有用功能。尽管它们的名字中都提到了 dd，但它们都没有使用`dd`命令的语法。

GNU ddrescue 由 Antonio Diaz Diaz 于 2004 年创建，并且以*“gddrescue”*的包名在 Debian 中打包。它采用激进和持续的方法来尝试恢复磁盘上的坏区。

dd_rescue 工具由 Kurt Garloff 于 1999 年创建，拥有一个复杂的插件系统，支持压缩、加密、哈希以及其他插件。

其他类似的存储介质恢复工具包括 myrescue 和 safecopy。这些工具中的一些将在第六章和第七章中演示。

### 取证格式

对于原始镜像的几个问题导致了取证文件格式的创建。当将存储介质作为证据进行成像时，会包含关于调查、调查员、驱动器详细信息、日志/时间戳、加密哈希等的元数据。除了元数据外，通常还需要对获取的镜像进行压缩或加密。专用的取证格式有助于实现这些功能，本文描述了最常见的格式。

取证文件格式有时被称为*证据容器*。一些研究工作还概述了数字证据袋的概念。^(1) 用于将数据采集到取证格式中的工具将在第六章中演示。

#### *EnCase EWF*

Guidance Software，作为最古老的取证软件公司之一，生产其旗舰产品 EnCase 取证软件套件，该套件使用专家证人格式（EWF）。EWF 格式支持元数据、压缩、加密、哈希、拆分文件等功能。一个反向工程的开源库和工具——libewf，由 Joachim Metz 于 2006 年创建，并且支持可以编译进 Sleuth Kit 中。

#### *FTK SMART*

AccessData 的 FTK SMART 格式是 EnCase EWF 的直接竞争对手。这是一种专有格式，也包括元数据、压缩、加密、哈希、拆分文件等功能。命令行工具 ftkimager（免费但不是开源的）可以从 AccessData 获得，并将在第六章和第七章中进行演示。

#### *AFF*

高级取证格式（AFF）由 Simson Garfinkel 创建，作为一种开放的、同行评审的、公开发布的格式。它包括所有取证格式的预期功能，还包括使用标准 X.509 证书的额外加密和签名功能。AFFlib 软件包包含许多用于转换和管理 AFF 格式的工具。

AFF 版本 3 由* [`github.com/sshock/AFFLIBv3/`](http://github.com/sshock/AFFLIBv3/)*单独维护。2009 年，关于 AFF 版本 4 的论文发表了。^(2) 当前的 AFF 版本 4 网站可以在* [`www.aff4.org/`](http://www.aff4.org/)*找到。高级取证格式 4 工作组（AFF4 WG）在 2016 年夏季宣布，并于 DFRWS 会议上举行了第一次会议，地点在西雅图。

### SquashFS 作为取证证据容器

在本书中，我将演示一种创建混合法医容器的技巧，该容器结合了简单的原始成像，并允许以类似于更高级法医格式的方式存储支持案件信息。该技巧使用 SquashFS 作为法医证据容器，并配合一个小型 shell 脚本 sfsimage 来管理容器的各个方面。此方法将一个压缩镜像与成像日志、磁盘设备信息以及任何其他信息（如照片、证据链表格等）结合成一个包。文件被包含在只读的 SquashFS 文件系统中，你可以在没有特殊法医工具的情况下访问它们。

#### *SquashFS 背景*

SquashFS 是一个高压缩率的只读文件系统，专为 Linux 设计。它由 Phillip Lougher 于 2002 年创建，并于 2009 年合并到 Linux 内核树中，从 2.6.29 版本的内核开始支持。

SquashFS 最初是为可启动 CD 和嵌入式系统设计的，但它有许多特性使其成为法医证据容器的有吸引力选择：

• SquashFS 是一个高压缩率的文件系统。

• 它是只读的；可以添加项目，但不能移除或修改。

• 它存储调查员的 uid/gid 和创建时间戳。

• 它支持非常大的文件大小（理论上可达 16EiB）。

• 它已包含在 Linux 内核中，挂载为只读文件系统非常简单。

• 该文件系统是一个开放标准（Windows、OS X 等平台已有工具支持）。

• mksquashfs 工具使用所有可用的 CPU 来创建容器。

将 SquashFS 作为法医证据容器的使用是使用其他法医格式的实际替代方案，因为它便于管理使用 dd 获取的压缩原始镜像。接下来的 sfsimage 工具提供了你需要的功能来管理 SquashFS 法医证据容器。

#### *SquashFS 法医证据容器*

现代 Linux 内核默认支持 SquashFS 文件系统。无需额外的内核模块或重新编译即可挂载和访问 SquashFS 文件系统。然而，要创建文件、附加文件或列出 SquashFS 文件的内容，必须安装 squashfs-tools 包。^(3) 另外，依据你偏好的成像工具，可能还需要额外的法医软件包（如 dcfldd、dc3dd、ewfacquire）。

我的 sfsimage shell 脚本可以在*[`digitalforensics.ch/sfsimage/`](http://digitalforensics.ch/sfsimage/)*上找到。运行 sfsimage 而不带任何选项会显示一些帮助文本，描述其用法：

```
$ sfsimage
Sfsimage: a script to manage forensic evidence containers with squashfs
Version: Sfsimage Version 0.8
Usage:
       sfsimage -i diskimage container.sfs
       sfsimage -a file ... container.sfs
       sfsimage -l container.sfs ...
       sfsimage -m container.sfs ...
       sfsimage -m
       sfsimage -u container.sfs ...
Where:
diskimage is a disk device, regular file, or "-" for stdin
container.sfs is a squashfs forensic evidence container
file is a regular file to be added to a container
and the arguments are as follows:
  -i images a disk into a newly created *.sfs container
  -a adds a file to an existing *.sfs container
  -l lists the contents of an existing *.sfs container
  -m mounts an *.sfs container in the current directory
  -m without options shows all mounted sfs containers
  -u umounts an *.sfs container
```

要配置 sfsimage，你可以编辑脚本或为脚本创建单独的*sfsimage.conf*文件。*config*文件包含了注释和示例，允许你定义以下参数：

• 优选的成像/采集命令（`dd`、`dcfldd`、`dc3dd` 等）

• 查询设备的优选命令（`hdparm`、`tableu-parm` 等）

• 默认目录用于挂载证据容器（当前工作目录是默认的）

• 如何管理特权命令（`sudo`，`su`等）

• 创建文件的权限和 uid/gid

sfsimage 脚本使用**.sfs*作为 SquashFS 取证证据容器的命名约定。脚本附带了 sfsimage(1)手册页，提供了更多详细信息。

要将磁盘镜像到 SquashFS 取证证据容器中，请使用`-i`标志、磁盘设备和证据容器名称运行 sfsimage 命令。将创建一个包含图像和设备初步元数据的证据容器。在此示例中，sfsimage 已配置为使用 dc3dd 作为成像工具：

```
$ sfsimage -i /dev/sde kingston.sfs
Started: 2016-05-14T20:44:12
Sfsimage version: Sfsimage Version 0.8
Sfsimage command: /usr/bin/sfsimage -i /dev/sde
Current working directory: /home/holmes
Forensic evidence source: if=/dev/sde
Destination squashfs container: kingston.sfs
Image filename inside container: image.raw
Acquisition command: sudo dc3dd if=/dev/sde log=errorlog.txt hlog=hashlog.txt
    hash=md5 2>/dev/null | pv -s 7918845952
7.38GiB 0:01:19 [95.4MiB/s] [========================================>] 100%
Completed: 2016-05-14T20:45:31
```

在这里，创建了一个 SquashFS 容器，并在其中生成了一个常规的原始镜像。还可以创建其他日志和信息，或者单独添加。

你可以使用带有`-a`标志的 sfsimage 命令将额外的证据添加到容器中。例如，如果你需要将物理磁盘的照片添加到先前创建的取证证据容器中，可以使用以下命令完成此任务：

```
$ sfsimage -a photo.jpg kingston.sfs
Appending to existing 4.0 filesystem on kingston.sfs, block size 131072
```

要列出 SquashFS 取证证据容器的内容，请使用`-l`标志运行 sfsimage 脚本，如下所示：

```
$ sfsimage -l kingston.sfs
Contents of kingston.sfs:
drwxrwxrwx holmes/holmes        135 2016-05-14 20:46 squashfs-root
-r--r--r-- holmes/holmes        548 2016-05-14 20:45 squashfs-root/errorlog.txt
-r--r--r-- holmes/holmes        307 2016-05-14 20:45 squashfs-root/hashlog.txt
-r--r--r-- holmes/holmes 7918845952 2016-05-14 20:44 squashfs-root/image.raw
-rw-r----- holmes/holmes     366592 2016-05-14 20:45 squashfs-root/photo.jpg
-r--r--r-- holmes/holmes        431 2016-05-14 20:45 squashfs-root/sfsimagelog.txt
```

此命令输出显示了**.sfs*容器的内容（未挂载）。还显示了文件创建或添加的正确时间。错误日志、哈希日志和 sfsimage 日志包含有关活动和错误的文档。*photo.jpg*是随后添加到容器中的照片。

通过挂载**.sfs*文件，你可以访问在 SquashFS 容器中的获取的镜像和附加的元数据文件。内容变得像文件系统中的常规部分一样可访问。因为 SquashFS 文件系统是只读的，所以内容不会被修改。

在以下示例中，**.sfs*文件使用`-m`标志挂载，并且使用常规取证工具（此示例中的 sleuthkit mmls）对获取的镜像进行操作：

```
$ sfsimage -m kingston.sfs
kingston.sfs.d mount created
$ mmls kingston.sfs.d/image.raw
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

      Slot      Start        End          Length       Description
000:  Meta      0000000000   0000000000   0000000001   Primary Table (#0)
001:  -------   0000000000   0000002047   0000002048   Unallocated
002:  000:000   0000002048   0015466495   0015464448   Linux (0x83)
```

注意，挂载的**.sfs*容器（默认情况下）会显示为**.sfs.d*目录。挂载后，你可以使用常规操作系统工具或取证工具访问目录中的文件，甚至可以通过网络将文件作为共享驱动器导出。

当**.sfs.d*挂载不再需要时，使用`-u`标志卸载，如下所示：

```
$ sfsimage -u kingston.sfs.d
kingston.sfs.d unmounted
```

在没有挂载点的情况下运行`sfsimage -m`将列出所有挂载的 SquashFS 容器。你也可以在单个系统上挂载多个容器。

磁盘镜像文件的大小在取证环境中一直是一个难题。较大的磁盘大小会带来空间问题和后勤障碍。像 SquashFS 这样的实用压缩方法有助于解决这个问题。为了说明使用压缩文件系统的实用性，sfsimage 被用来对一个 8TB 的目标磁盘（*bonkers*）进行镜像，该磁盘位于一个只有 2TB 硬盘空间的调查员系统上。整个获取过程花费了超过 16 小时，最终生成的压缩 SquashFS 文件仅为 1TB。挂载的 SquashFS 文件提供对整个 8TB 的访问，作为一个原始镜像文件。该镜像文件在运行时进行压缩，不需要任何临时文件。**.sfs**文件和镜像文件的大小如下所示：

```
$ ls -l bonkers.sfs bonkers.sfs.d/bonkers.raw
-rw-r----- 1 holmes root 1042820382720 Jun 28 13:06 bonkers.sfs
-r--r--r-- 1 root root 8001562156544 Jun 27 20:19 bonkers.sfs.d/bonkers.raw
```

使用 SquashFS 是一种实用且有效的解决方案，可以以压缩的方式使用原始文件，并提供了另一种取证证据容器的选择。

### 结束语

本章介绍了各种取证镜像格式。我提供了关于不同工具的简要概述和历史，这些工具可以用来取证性地获取硬盘。你还了解了 SquashFS 文件系统以及用于创建和管理 SquashFS 取证证据容器的 sfsimage 脚本。本章中介绍的工具和格式将在本书其余部分的示例中使用。
