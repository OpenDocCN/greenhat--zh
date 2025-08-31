## 第十章：**提取法证图像的子集**

![image](img/common-01.jpg)

本章介绍如何从附加驱动器或法证获取的镜像文件中选择性提取数据区域。你将学习如何提取整个分区、已删除或部分覆盖的分区、分区间隙以及各种卷和文件空闲区。此外，你将看到如何提取一些特殊区域，如统一可扩展固件接口（UEFI）分区、DCO 或 HPA 隐藏的扇区，以及休眠分区，如英特尔快速启动技术。

最后的部分演示了如何从已分配和未分配（可能已删除）区域提取数据，以便进一步检查，并使用偏移量手动提取扇区。我们从确定驱动器的分区布局开始。

### 评估分区布局和文件系统

一旦你将磁盘连接到系统或获取了镜像文件，你可以对磁盘的分区方案进行分析。本节解释如何识别文件系统、分区表和常用的磁盘分区方案。

磁盘布局，或称为 *分区方案*，指的是用于组织硬盘上的 *分区*（或 *切片*）的方法。你在消费级计算中最常见的分区方案有 DOS、GPT、BSD 和 APM（Apple Partition Map，有时称为 *mac*）。我们将从识别磁盘上使用的分区方案开始。

#### *分区方案*

磁盘上的每个分区或切片包含一个独立的文件系统，或者用于某些特殊目的。磁盘的一小部分（通常只是第一个扇区）通过指定每个分区的起始扇区、分区大小、分区类型、标签等信息来定义磁盘的布局。

要确定磁盘的分区方案，你可以检查磁盘的初始扇区以寻找指示标志。分区方案没有官方的“分配号码”指定（大约只有六种或更多）。不要将其与 DOS MBR 分区类型或 ID 混淆，后者列出了最多 255 种可能的文件系统和其他格式，这些格式可能存在于 DOS 分区中。当你将目标磁盘连接到工作站时，Linux 内核将尝试检测并解释所使用的分区方案，并为它找到的每个分区创建设备。

你可以使用 Sleuth Kit 的 `mmstat` 命令来识别最常见的分区方案。支持的分区方案列表如下：

```
# mmstat -t list
Supported partition types:
        dos (DOS Partition Table)
        mac (MAC Partition Map)
        bsd (BSD Disk Label)
        sun (Sun Volume Table of Contents (Solaris))
        gpt (GUID Partition Table (EFI))
```

运行 `mmstat` 将输出所使用的方案名称：

```
# mmstat image.raw
dos
```

或者，你可以使用 disktype 工具来识别分区方案。disktype 工具提供更详细的信息，支持分区、文件系统以及文件和归档容器。以下示例显示了 disktype 的输出：

```
$ sudo disktype /dev/sda

--- /dev/sda
Block device, size 27.96 GiB (30016659456 bytes)
DOS/MBR partition map
Partition 1: 27.95 GiB (30015610880 bytes, 58624240 sectors from 2048)
  Type 0x83 (Linux)
```

你可以在*[`disktype.sourceforge.net/`](http://disktype.sourceforge.net/)*找到原始的 disktype 软件包。另外，你可以在*[`github.com/kamwoods/disktype/`](https://github.com/kamwoods/disktype/)*、*[`github.com/Pardus-Linux/Packages/tree/master/system/base/disktype/files/`](https://github.com/Pardus-Linux/Packages/tree/master/system/base/disktype/files/)* 和 *[`github.com/ericpaulbishop/gargoyle/tree/master/package/disktype/patches/`](https://github.com/ericpaulbishop/gargoyle/tree/master/package/disktype/patches/)*找到 disktype 的分支和多个补丁。

存储介质不需要分区表甚至文件系统。二进制数据可以直接写入原始磁盘，并且可以被任何能够理解它的程序访问（例如，一些数据库可以直接使用原始磁盘）。也有可能存在没有分区方案的磁盘。在这种情况下，文件系统从第零扇区开始，直到磁盘的末尾（即整个磁盘就是一个分区）。这在一些旧的 USB 闪存盘和软盘中很常见。在这种情况下，分区分析工具将无效，并且通常报告虚假的或不存在的分区表。如果工具无法检测到分区类型，值得检查是否将文件系统直接写入了原始设备。在这个例子中，`mmstat`找不到任何内容，但`fsstat`确实识别出了文件系统：

```
# mmls /dev/sdj
Cannot determine partition type
# disktype /dev/sdj

--- /dev/sdj
Block device, size 1.406 MiB (1474560 bytes)
FAT12 file system (hints score 5 of 5)
  Volume size 1.390 MiB (1457664 bytes, 2847 clusters of 512 bytes)

# mmstat /dev/sdj
Cannot determine partition type
# fsstat /dev/sdj
FILE SYSTEM INFORMATION
--------------------------------------------
File System Type: FAT12
...
```

一些加密卷试图隐藏其存在或有关使用的文件系统的信息，它们不会使用可识别的分区方案。

#### *分区表*

分区方案会有一个磁盘块或一组块，描述其如何组织。这些叫做*分区表*（或者 BSD 系统中的*磁盘标签*），你可以使用各种工具查询它们。

你可以使用 Sleuth Kit 的`mmls`命令列出磁盘或法医获取的映像上的分区表。在这个例子中，`mmls`找到了一个常规的 DOS 分区方案，包含一个 FAT32 分区：

```
# mmls image.raw
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

     Slot    Start        End          Length       Description
00:  Meta    0000000000   0000000000   0000000001   Primary Table (#0)
01:  -----   0000000000   0000000062   0000000063   Unallocated
02:  00:00   0000000063   0005028344   0005028282   Win95 FAT32 (0x0b)
03:  -----   0005028345   0005033951   0000005607   Unallocated
```

传统的 DOS 分区方案无法处理大于 2TB 的磁盘。GPT 分区方案的创建是为了允许更大的磁盘通过更多的分区进行组织。GPT 支持 128 个分区，而 DOS 只支持 4 个分区（不包括扩展分区）。我曾写过一篇关于 GPT 磁盘和 GUID 分区表的法医分析论文，你可以在这里找到：*[`dx.doi.org/10.1016/j.diin.2009.07.001`](http://dx.doi.org/10.1016/j.diin.2009.07.001)*。

现在大多数新的 PC 系统都使用 GPT 分区。以下是一个 Windows 8 系统的分区表示例：

```
# mmls lenovo.raw
GUID Partition Table (EFI)
Offset Sector: 0
Units are in 512-byte sectors

     Slot    Start        End          Length       Description
00:  Meta    0000000000   0000000000   0000000001   Safety Table
01:  -----   0000000000   0000002047   0000002048   Unallocated
02:  Meta    0000000001   0000000001   0000000001   GPT Header
03:  Meta    0000000002   0000000033   0000000032   Partition Table
04:  00      0000002048   0002050047   0002048000
05:  01      0002050048   0002582527   0000532480   EFI system partition
06:  02      0002582528   0003606527   0001024000
07:  03      0003606528   0003868671   0000262144   Microsoft reserved partition
08:  04      0003868672   1902323711   1898455040   Basic data partition
09:  05      1902323712   1953523711   0051200000
```

Gary Kessler 提供了几种分区表解析工具，这些工具可以提供更详细的信息。你可以在*[`www.garykessler.net/software/index.html`](http://www.garykessler.net/software/index.html)*找到这些工具。

为了说明 Kessler 的解析工具提供的详细程度，以下是使用 gptparser.pl 工具生成的前面示例中的分区表部分输出：

```
$ gptparser.pl -i lenovo.raw

GPT Parser V1.4 beta - Gary C. Kessler (14 March 2013)

Source file = /exam/lenovo.raw
Input file length = 17408 bytes.

***** LBA 0: Protective/Legacy MBR *****

000:  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00   ................
016:  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00   ................
...
=== Partition Table #5 (LBA 3, bytes 0:127) ===
000-015  Partition type GUID: 0xA2-A0-D0-EB-E5-B9-33-44-87-C0-68-B6-B7-26-99-C7
         GUID: EBD0A0A2-B9E5-4433-87C0-68B6B72699C7
         Type: Data partition (Linux *or* Windows)
016-031  Partition GUID: 0x64-12-FF-80-A7-F7-72-42-B6-46-25-33-6D-96-13-B5
         GUID: 80FF1264-F7A7-4272-B646-25336D9613B5
032-039  First LBA: 0x00-08-3B-00-00-00-00-00 [3,868,672]
040-047  Last LBA: 0xFF-27-63-71-00-00-00-00 [1,902,323,711]
048-055  Partition attributes: 0x00-00-00-00-00-00-00-00
056-127  Partition name --
056:  42 00 61 00 73 00 69 00 63 00 20 00 64 00 61 00   B.a.s.i.c. .d.a.
072:  74 00 61 00 20 00 70 00 61 00 72 00 74 00 69 00   t.a. .p.a.r.t.i.
088:  74 00 69 00 6F 00 6E 00 00 00 00 00 00 00 00 00   t.i.o.n.........
104:  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00   ................
120:  00 00 00 00 00 00 00 00                           ........
      Name: Basic data partition
...
```

该工具提供了关于每个 128 个 GPT 分区的详细信息。

#### *文件系统识别*

已在 “分区方案” 中介绍过的 disktype 工具（第 260 页）允许你识别分区方案和分区内的文件系统。Sleuth Kit 的 fsstat 工具提供关于文件系统的更全面信息。fsstat 工具可以直接在分区设备上操作，或者如果你指定了扇区偏移量，可以在取证获取的镜像上操作。

在之前的示例中，*lenovo.raw* 镜像文件上 Windows 卷的扇区偏移量是 3868672。你可以通过 `-o` 标志将此扇区偏移量提供给 fsstat 工具，以分析文件系统元数据：

```
# fsstat -o 3868672 lenovo.raw
FILE SYSTEM INFORMATION
--------------------------------------------
File System Type: NTFS
Volume Serial Number: 4038B39F38B39300
OEM Name: NTFS
Volume Name: Windows8_OS
Version: Windows XP

METADATA INFORMATION
--------------------------------------------
First Cluster of MFT: 786432
...
```

如果驱动器直接连接到你的工作站，Linux 内核将尝试解析分区表，并在 */dev* 中提供磁盘和分区设备，你可以直接访问它们。

然而，如果你正在检查一个原始镜像文件（*.raw*、*.ewf* 等），该镜像将没有设备文件。内核不会解析分区表，也不会创建熟悉的分区设备（*/dev/sda1*、*/dev/sda2* 等）。在访问镜像文件中的分区时，你必须指定偏移量。

最好依赖取证工具来确定分区详情，而不是依赖内核。如果磁盘损坏或损坏，内核可能会拒绝创建分区设备，或者创建错误的设备。你在本节看到的示例始终指定了偏移量，而不是使用内核。在涉及恶意软件、防取证技术或其他恶意误导的情况下，应该优先使用取证工具，而不是内核。

### 分区提取

本节介绍了单个分区的提取、分区间隙和磁盘上其他区域（如 DCO 和 HPA）。我们从提取常规分区的一些基本示例开始。

#### *提取单个分区*

为了访问和提取单独的分区而非整个硬盘，你可以使用多种技术。我将展示一些使用直接连接的驱动器、分区设备、分区映射设备以及由 Sleuth Kit 的 mmcat 和 dd 风格工具操作的镜像文件进行分区提取的示例。

如果磁盘作为附加设备可访问，则获取分区的过程类似于使用原始驱动器设备进行完整获取，但使用的是分区设备。以下示例中，*/dev/sda* 的第一个分区被提取到文件中：

```
# dcfldd if=/dev/sda1 of=partition.raw
```

提取分区需要一定的容量规划，因为分区会占用磁盘空间（可能还会占用完整的驱动器镜像）。如果你仅需要临时访问一个获取的镜像文件中的分区，可以将其作为环回设备附加并进行访问。以下步骤演示了这种方法。

首先，使用 mmls 工具识别要作为环回设备附加的分区，方法如下：

```
# mmls lenovo.raw
GUID Partition Table (EFI)
Offset Sector: 0
Units are in 512-byte sectors
...
05:  01      0002050048   0002582527   0000532480   EFI system partition
...
```

然后使用 Bash 数学扩展将扇区偏移量和扇区长度转换为字节偏移量和字节长度：

```
# echo $((2050048*512))
1049624576
# echo $((532480*512))
272629760
```

计算出的字节偏移量和字节长度会传递给 losetup 来创建一个环回设备，方法如下：

```
# losetup --read-only --find --show --offset 1049624576 --sizelimit 272629760
    lenovo.raw
/dev/loop2
```

你可以像访问附加磁盘的分区设备一样，使用法医工具访问这个结果环回设备。这里是使用 Sleuth Kit fls 的一个示例：

```
# fls /dev/loop2
r/r 3:  SYSTEM_DRV (Volume Label Entry)
d/d 4:  EFI
d/d 5:  BOOT
d/d * 7:        MSIa11f8.tmp
d/d * 8:        _SI2DBB4.TMP
d/d * 9:        _190875_
...
```

如果你需要将已获取的镜像中的某个分区提取到一个单独的文件中，可以使用 dd 工具或 Sleuth Kit 的 `mmcat` 命令。

提取已获取镜像中的分区，初步步骤是识别分区和扇区的详细信息。在以下示例中，从获取的磁盘镜像中提取的分区表显示了要提取的分区：

```
# mmls image.raw
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

     Slot    Start        End          Length       Description
...
02:  00:00   0000000063   0078124094   0078124032   Linux (0x83)
...
```

使用 dcfldd 或 dd 从已获取的镜像文件中提取分区时，需要添加 `skip`（dc3dd 使用 `iskip`）和 `count` 参数，这些参数使得命令跳过（skip）分区的起始位置，并仅获取分区的大小：

```
$ dcfldd if=image.raw of=partition.raw bs=512 skip=63 count=78124032
```

在此命令中，块大小设置为 512 字节，以匹配扇区大小，分区起始位置在扇区 63，应该提取 78124032 个扇区。通过进行一些额外的计算，你可以通过将 512 字节块大小更改为更大的值来提高此命令的性能（但如果这样做，不要忘记调整 `skip` 和 `count` 参数）。

在 Sleuth Kit 3.0 及以后版本中，你可以使用 mmcat 工具轻松提取分区。要使用 mmcat 恢复前面示例中的第一个分区，你必须指定 mmls 插槽编号（而不是 DOS 分区编号）。在此示例中，第一个分区位于 mmls 插槽编号 2，可以按如下方式提取：

```
$ mmcat image.raw 2 > partition.raw
```

mmcat 工具会简单地将输出通过管道传递到标准输出（stdout），因此你必须将其重定向到文件或将其传递到其他程序中。

#### *查找并提取已删除的分区*

为了全面搜索法医获取的镜像中部分覆盖或已删除的分区，可以使用多种方法。Sleuth Kit 提供了一个名为 sigfind 的基本工具来搜索二进制签名字符串。gpart 和 testdisk 是两种用于全面分区搜索的有用工具。它们实现了文件系统识别算法，通过更智能的猜测来识别丢失的分区。

运行 gpart 命令而不加任何选项时，它会开始扫描分区，跳过已标记为已分配的区域。例如：

```
# gpart lenovo.raw

Begin scan...
Possible partition(Windows NT/W2K FS), size(1000mb), offset(1mb)
Possible partition(Windows NT/W2K FS), size(3mb), offset(1030mb)
Possible partition(Windows NT/W2K FS), size(3mb), offset(1494mb)
Possible partition(Windows NT/W2K FS), size(926980mb), offset(1889mb)
Possible partition(Windows NT/W2K FS), size(25000mb), offset(928869mb)
End scan.
...
Guessed primary partition table:
Primary partition(1)
   type: 007(0x07)(OS/2 HPFS, NTFS, QNX or Advanced UNIX)
   size: 1000mb #s(2048000) s(2048-2050047)
   chs:  (0/32/33)-(406/60/28)d (0/32/33)-(406/60/28)r
...
```

添加`-f`标志会告诉 gpart 执行更全面的扫描，检查整个磁盘的每个扇区，即使在没有预期分区的区域也会进行检查。与没有标志的默认 gpart 扫描相比，这将花费更多时间。

testdisk 工具（* [`www.cgsecurity.org/`](http://www.cgsecurity.org/)*，由 Christophe Grenier 编写，他还编写了 photorec 数据恢复工具）提供了多个分区搜索功能。Testdisk 提供交互式界面，支持多种磁盘布局（DOS、GPT、BSD 等），检测多种分区类型，生成活动日志，并可以将发现的分区提取到文件中。你可以在设备、原始镜像文件甚至**.e01*文件上使用 testdisk。

小心使用 testdisk 工具。此工具是为修复和恢复分区设计的，使用不当可能会轻易修改证据。在运行此工具之前，确保在附加的目标磁盘上使用写保护器。

工具还包括一个全面的用户交互菜单系统，用于定义选项和操作。这里展示的是一个在附加磁盘上运行的批处理模式示例：

```
# testdisk /list /dev/sdb
TestDisk 7.0, Data Recovery Utility, April 2015
Christophe GRENIER <grenier@cgsecurity.org>
http://www.cgsecurity.org
Please wait...
Disk /dev/sdb - 15 GB / 14 GiB - CHS 14663 64 32
Sector size:512
Model: SanDisk Ultra Fit, FW:1.00

Disk /dev/sdb - 15 GB / 14 GiB - CHS 14663 64 32
     Partition                  Start        End    Size in sectors
 1 P FAT32 LBA                0   1  1 14663  44 18   30031218 [NO NAME]
     FAT32, blocksize=16384
```

你可以进行一定量的手动分析，以搜索已删除的分区。如果分区表显示磁盘上有大片未分配空间，请检查该区域以确定是否存在分区。在以下示例中，mmls 显示在一个 U 盘的末尾有接近 2.5GB（4863378 个扇区）的空闲空间：

```
# mmls /dev/sdb
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

      Slot      Start        End          Length       Description
000:  Meta      0000000000   0000000000   0000000001   Primary Table (#0)
001:  -------   0000000000   0000002047   0000002048   Unallocated
002:  000:000   0000002048   0025167871   0025165824   Win95 FAT32 (0x0c)
003:  -------   0025167872   0030031249   0004863378   Unallocated
```

这个未分配空间可能是一个已删除的分区。在此示例中，通过使用空闲空间的偏移量运行 fsstat 可以发现一个有效的文件系统：

```
# fsstat -o 25167872 /dev/sdb
FILE SYSTEM INFORMATION
--------------------------------------------
File System Type: Ext3
Volume Name:
Volume ID: 74a2f1b777ae52bc9748c3dbca837a80

Last Written at: 2016-05-21 15:42:54 (CEST)
Last Checked at: 2016-05-21 15:42:54 (CEST)
...
```

如果你检测到有效的文件系统，可以使用其元数据来确定分区的可能大小。知道了大小和起始偏移后，你可以提取发现的分区或进一步分析它。你可以使用 dd 风格的工具提取它，或者使用 mmcat 更容易地提取，如下所示：

```
# mmcat /dev/sdb 3 > deleted_partition.raw
```

在这里，从 mmls 槽 003 中发现的已删除分区的 mmcat 输出被发送到名为*deleted_partition.raw*的文件中。

#### *识别并提取分区间空隙*

在某些情况下，分区之间可能会有意外的空隙，或者由于相邻分区在柱面或块边界上相遇所造成的空隙。也可能是为了隐藏数据而故意创建的空隙。你可以像提取分区一样识别并恢复这些分区间的空隙。使用 mmls 确定空隙的大小和扇区偏移，然后使用 dd 或 mmcat 来提取它。

这里展示了一个分区表的 mmls 输出。磁盘包含两个分区，并且它们之间存在空隙：

```
# mmls /dev/sdb
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

      Slot      Start        End          Length       Description
000:  Meta      0000000000   0000000000   0000000001   Primary Table (#0)
001:  -------   0000000000   0000002047   0000002048   Unallocated
002:  000:000   0000002048   0015626236   0015624189   Linux (0x83)
003:  -------   0015626237   0015626239   0000000003   Unallocated
004:  000:001   0015626240   0030031249   0014405010   Linux (0x83)
```

在此示例中，第一个分区结束于扇区 15626236，但相邻分区从扇区 15626240 开始，表明它们之间有一个三扇区的空隙。虽然你可以使用 dd 提取这个分区间空隙，但使用 mmcat 更简单：

```
# mmcat /dev/sdb 3 > gap.raw
# ls -l gap.raw
-rw-r----- 1 root root 1536 May 21 16:11 gap.raw
```

生成的文件大小为三个扇区，包含两个分区之间的空白区域。分区之间的较大空隙，如果包含部分重写、损坏或可识别的文件系统碎片，可以通过如 foremost 这样的雕刻工具进行分析。

最后一个分区与磁盘末尾之间的空隙也可能是值得关注的地方。它可能包含一些遗留物，比如先前重写的分区的内容、GPT 分区的备份副本，甚至是恶意软件试图隐藏二进制代码片段。

#### *提取 HPA 和 DCO 扇区范围*

你已经学会了如何识别并移除 HPA 和 DCO 限制。一旦移除，这些磁盘区域可以被提取出来进行单独分析。

在这个示例中，hdparm 显示存在 HPA，而 mmls 输出显示三个插槽，其中一个是 Linux 分区：

```
# hdparm -N /dev/sdh

/dev/sdh:
 max sectors   = 234441648/976773168, HPA is enabled
# mmls /dev/sdh
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

     Slot    Start        End          Length       Description
00:  Meta    0000000000   0000000000   0000000001   Primary Table (#0)
01:  -----   0000000000   0000002047   0000002048   Unallocated
02:  00:00   0000002048   0234441647   0234439600   Linux (0x83)
```

成功移除 HPA 后（并告诉内核重新扫描 SCSI 总线），再次运行相同的命令会产生不同的输出，如下所示：

```
# hdparm -N p976773168 /dev/sdh

/dev/sdh:
 setting max visible sectors to 976773168 (permanent)
 max sectors   = 976773168/976773168, HPA is disabled
# mmls /dev/sdh
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

     Slot    Start        End          Length       Description
00:  Meta    0000000000   0000000000   0000000001   Primary Table (#0)
01:  -----   0000000000   0000002047   0000002048   Unallocated
02:  00:00   0000002048   0234441647   0234439600   Linux (0x83)
03:  -----   0234441648   0976773167   0742331520   Unallocated
```

现在，hdparm 指示 HPA 已被禁用，而 mmls 输出显示一个额外的输出行（插槽 03），表示之前被 HPA 隐藏的扇区。

使用 `mmcat` 命令与分区插槽 03 结合，将提取来自 HPA 的数据，如下所示：

```
# mmcat /dev/sdh 3 > hpa.raw
```

本示例使用的是连接到获取主机的实时磁盘。当从移除 HPA 的磁盘中获取映像文件时，mmls 将看到这个隐藏的区域。

提取由 DCO 隐藏的扇区的方法与这里展示的 HPA 方法完全相同。首先使用 hdparm 显示 DCO 保护的扇区，然后使用 dd 或 mmcat 提取这些扇区。这个过程不需要通过额外的示例来重复，专门演示 DCO 扇区的提取。

### 其他分段数据提取

在本节的最后，我将描述各种其他的分段数据提取示例。本节内容（实际上是本章的大部分内容）与法医文件系统分析有些重叠，这不是本书的预期范围。因此，这些示例的描述略显简略。

#### *提取文件系统松散空间*

*松散空间* 是一个传统的数字取证概念，指的是位于磁盘扇区、文件系统块或文件系统末尾的已分配但未使用的数据（分别为 RAM 松散、文件松散和分区松散）。

为了可视化松散空间，假设这本书就是一个硬盘，其中段落是扇区，章节是文件，正文部分是分区。请注意，段落不会精确地在一行的末尾结束，章节不会精确地在一页的末尾结束，书的结尾可能会有几页空白。这些空白区域就是这本书的“松散空间”。对于存储介质，如果操作系统或物理驱动器没有明确地将零写入这些区域，它们可能仍然包含先前写入文件的数据。

历史上，提取和分析空闲空间在取证调查中一直很有用。然而，由于多个因素，空闲空间的价值正在下降：

• SSD 使用 TRIM 命令将未分配的块置零。

• 现代操作系统正在将零写回未使用的扇区和块的部分。

• 原生 4K 扇区的磁盘与文件系统块大小对齐。

• 操作系统创建的分区和文件系统对齐到块边界。

作为取证过程的一部分，获取和分析潜在的空闲区域仍然是需要认真完成的步骤。

要提取给定镜像的所有空闲空间，可以使用 Sleuth Kit 的`blkls`命令。空闲空间是文件系统特定的，因此必须在每个文件系统上单独提取空闲空间（不能仅使用整个原始磁盘）。在这个示例中，通过 mmls 找到了已获取镜像的文件系统偏移量，并提取了每个文件系统的空闲空间：

```
# mmls lenovo.raw
GUID Partition Table (EFI)
Offset Sector: 0
Units are in 512-byte sectors

     Slot    Start        End          Length       Description
04:  00      0000002048   0002050047   0002048000
05:  01      0002050048   0002582527   0000532480   EFI system partition
06:  02      0002582528   0003606527   0001024000
...
08:  04      0003868672   1902323711   1898455040   Basic data partition
...
# blkls -o 2048 -s lenovo.raw > slack.04
# blkls -o 2050048 -s lenovo.raw > slack.05
# blkls -o 2582528 -s lenovo.raw > slack.06
# blkls -o 3868672 -s lenovo.raw > slack.08
```

每个已识别文件系统的空闲空间被保存到文件中。`blkls`命令的`-s`标志提取所有空闲空间（仅空闲空间）。重要的是要理解，空闲空间并不指未分配的块或扇区。空闲空间是文件系统中*已分配*块和扇区内未使用的区域。

#### *提取文件系统未分配的块*

下面的示例将从已获取的镜像中的文件系统收集所有未分配的块。未分配的块是文件系统特定的，因此需要对每个识别的文件系统分别执行此操作。

在这里，`mmls`命令再次用于确定每个文件系统的偏移量，并使用`blkls`命令提取未分配的块：

```
# blkls -o 2048 lenovo.raw > unalloc.04
# blkls -o 2050048 lenovo.raw > unalloc.05
# blkls -o 2582528 lenovo.raw > unalloc.06
# blkls -o 3868672 lenovo.raw > unalloc.08
```

提取未分配块的正确`blkls`标志是`-A`，但是由于这是默认命令行为，因此可以省略它。

您还可以执行提取所有（仅限）分配块的反向操作，使用`blkls -a`命令。

#### *使用偏移量进行手动提取*

在某些情况下，您可能会使用 hex 编辑器浏览、搜索或手动分析磁盘或已获取的磁盘镜像的内容。hex 编辑器可能提供字节偏移量、扇区偏移量或两者。

这个示例使用基于控制台的 hexedit 工具来分析磁盘：

```
# hexedit -s /dev/sda
```

hexedit 工具允许您直接编辑块设备文件，并编辑非常大的镜像文件（无需加载到内存或临时文件），并提供扇区模式（显示整个扇区和扇区偏移量）。

在以下示例中，扇区偏移量为 2048（NTFS 分区的起始位置），字节偏移量为 0x100181，显示整个扇区（注意：hexedit 假设 512 字节扇区）：

```
00100000   EB 52 90 4E 54 46 53 20  20 20 20 00 02 08 00 00  .R.NTFS    .....
00100010   00 00 00 00 00 F8 00 00  3F 00 FF 00 00 08 00 00  ........?.......
00100020   00 00 00 00 80 00 80 00  01 48 00 00 00 00 00 00  .........H......
00100030   04 00 00 00 00 00 00 00  80 04 00 00 00 00 00 00  ................
00100040   F6 00 00 00 01 00 00 00  22 90 FD 7E 2E 42 12 09  ........"..~.B..
00100050   00 00 00 00 FA 33 C0 8E  D0 BC 00 7C FB 68 C0 07  .....3.....|.h..
00100060   1F 1E 68 66 00 CB 88 16  0E 00 66 81 3E 03 00 4E  ..hf......f.>..N
00100070   54 46 53 75 15 B4 41 BB  AA 55 CD 13 72 0C 81 FB  TFSu..A..U..r...
00100080   55 AA 75 06 F7 C1 01 00  75 03 E9 D2 00 1E 83 EC  U.u.....u.......
00100090   18 68 1A 00 B4 48 8A 16  0E 00 8B F4 16 1F CD 13  .h...H..........
001000A0   9F 83 C4 18 9E 58 1F 72  E1 3B 06 0B 00 75 DB A3  .....X.r.;...u..
001000B0   0F 00 C1 2E 0F 00 04 1E  5A 33 DB B9 00 20 2B C8  ........Z3... +.
001000C0   66 FF 06 11 00 03 16 0F  00 8E C2 FF 06 16 00 E8  f...............
001000D0   40 00 2B C8 77 EF B8 00  BB CD 1A 66 23 C0 75 2D  @.+.w......f#.u-
001000E0   66 81 FB 54 43 50 41 75  24 81 F9 02 01 72 1E 16  f..TCPAu$....r..
001000F0   68 07 BB 16 68 70 0E 16  68 09 00 66 53 66 53 66  h...hp..h..fSfSf
00100100   55 16 16 16 68 B8 01 66  61 0E 07 CD 1A E9 6A 01  U...h..fa.....j.
00100110   90 90 66 60 1E 06 66 A1  11 00 66 03 06 1C 00 1E  ..f`..f...f.....
00100120   66 68 00 00 00 00 66 50  06 53 68 01 00 68 10 00  fh....fP.Sh..h..
00100130   B4 42 8A 16 0E 00 16 1F  8B F4 CD 13 66 59 5B 5A  .B..........fY[Z
00100140   66 59 66 59 1F 0F 82 16  00 66 FF 06 11 00 03 16  fYfY.....f......
00100150   0F 00 8E C2 FF 0E 16 00  75 BC 07 1F 66 61 C3 A0  ........u...fa..
00100160   F8 01 E8 08 00 A0 FB 01  E8 02 00 EB FE B4 01 8B  ................
00100170   F0 AC 3C 00 74 09 B4 0E  BB 07 00 CD 10 EB F2 C3  ..<.t...........
00100180   0D 0A 41 20 64 69 73 6B  20 72 65 61 64 20 65 72  ..A disk read er
00100190   72 6F 72 20 6F 63 63 75  72 72 65 64 00 0D 0A 42  ror occurred...B
001001A0   4F 4F 54 4D 47 52 20 69  73 20 6D 69 73 73 69 6E  OOTMGR is missin
001001B0   67 00 0D 0A 42 4F 4F 54  4D 47 52 20 69 73 20 63  g...BOOTMGR is c
001001C0   6F 6D 70 72 65 73 73 65  64 00 0D 0A 50 72 65 73  ompressed...Pres
001001D0   73 20 43 74 72 6C 2B 41  6C 74 2B 44 65 6C 20 74  s Ctrl+Alt+Del t
001001E0   6F 20 72 65 73 74 61 72  74 0D 0A 00 00 00 00 00  o restart.......
001001F0   00 00 00 00 00 00 00 00  80 9D B2 CA 00 00 55 AA  ..............U.

---  sda       --0x100181/0x6FD21E000--sector 2048---------------------------
```

从字节或扇区偏移量，可以构造 dd 命令来提取在 hex 编辑器中找到的内容。

以下示例使用 512 字节的扇区大小、扇区偏移量和扇区计数从镜像中提取一范围的数据（四个 512 字节的扇区）：

```
# dd if=/dev/sda of=sectors.raw skip=2048 bs=512 count=4
```

下一个示例使用字节偏移量提取相同范围的数据。`skip`命令使用 Bash 数学扩展将十六进制转换为十进制，这是`dd`命令所需要的；块大小为 1 字节；count 为所需字节数。

```
# dd if=/dev/sda of=bytes.raw skip=$((0x100000)) bs=1 count=2048
```

前面的两个示例提取了相同的块（四个扇区或 2048 字节）数据。请注意，在提取磁盘区域时，确保你有扇区或块对齐的偏移量是明智的（即扇区大小或块大小的倍数）。

在需要提取一系列文件系统块的情况下，可以使用 Sleuth Kit 的`blkcat`命令。以下示例从文件系统中提取从块 100 开始的 25 个块：

```
# blkcat /dev/sda1 100 25 > blocks.raw
```

文件系统的块大小应由工具检测。

本章最后部分的示例展示了如何访问镜像；使用偏移量；并提取字节、扇区或块的范围。你还可以使用其他 Sleuth Kit 命令将扇区映射到块，将块映射到索引节点和文件名。这些任务是特定于文件系统的，涉及到文件系统取证分析的范畴。

### 结语

在本章的最后，你学习了如何提取驱动器和取证镜像的子集。本章重点介绍了如何提取镜像的不同部分，例如由 HPA 或 DCO 隐藏的扇区、已删除的分区和分区间隙。你还看到了如何手动提取指定的扇区和块，包括未分配的块和闲置空间。本章接近取证分析，讨论了如何识别分区方案、理解分区表和识别文件系统。由于本书的主题是取证采集而非取证分析，因此这也是一个合适的结尾章节。
