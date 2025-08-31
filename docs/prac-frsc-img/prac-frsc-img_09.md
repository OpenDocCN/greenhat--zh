## 第九章：**特殊镜像访问主题**

![image](img/common-01.jpg)

本章展示了如何获取磁盘镜像文件的信息，并将其作为块设备和挂载目录使其可访问。你将学习如何设置循环设备，并使用设备映射工具创建逻辑设备。你还将探索映射或转换软件加密磁盘镜像的方法，使其能够被取证工具访问。这些方法在图像内容无法直接访问且需要进行主动翻译或解密时非常有用。此类图像的示例包括加密文件系统、虚拟机（VM）镜像以及其他取证工具不直接支持的镜像文件格式。

每个部分还包括将（只读）镜像文件安全挂载为常规文件系统的示例，这些镜像文件将被挂载到取证采集主机上。然后，你可以使用常见程序轻松浏览和访问文件系统，如文件管理器、办公套件、文件查看器、媒体播放器等。

### 取证采集的镜像文件

本部分中许多方法和示例的基础是 Linux 循环设备（不要与环回设备混淆，环回设备是网络接口）。*循环设备*是一种伪设备，可以与常规文件关联，使得该文件可以作为块设备在*/dev*中访问。

Linux 系统通常默认创建八个循环设备，这对于取证采集主机可能不够，但你可以手动或自动在启动时增加这个数量。要在启动时创建 32 个循环设备，可以在*/etc/default/grub*文件的`GRUB_CMDLINE_LINUX_DEFAULT=`行中添加`max_loop=32`；重启后，应有 32 个未使用的循环设备可用。sfsimage 脚本使用循环设备挂载 SquashFS 取证证据容器。

本章将介绍来自常见虚拟机系统（如 QEMU、VirtualBox、VMWare 和 Microsoft Virtual PC）的不同虚拟机镜像。我还将描述如何访问操作系统加密的文件系统，包括微软的 BitLocker、苹果的 FileVault、Linux LUKS 和 VeraCrypt（TrueCrypt 的一个分支）。但让我们从最简单的镜像类型开始：通过 dd 风格的采集工具获取的原始磁盘镜像。

#### *使用循环设备的原始镜像文件*

最简单的循环设备演示可以使用原始镜像文件（可能是通过简单的`dd`命令采集的）。`losetup`命令用于在 Linux 系统中附加和分离循环设备。此示例为*image.raw*文件创建一个块设备：

```
# losetup --read-only --find --show image.raw
/dev/loop0
```

在这里，标志指定循环设备应为只读（`--read-only`），并且应使用下一个可用的循环设备（`--find`），并在完成后显示（`--show`）。指定的文件名（*image.raw*）随后将作为附加的块设备可用。

运行没有参数的`losetup`命令将显示所有已配置循环设备的状态。这里我们可以看到刚刚创建的一个：

```
# losetup
NAME       SIZELIMIT OFFSET AUTOCLEAR RO BACK-FILE
/dev/loop0         0      0         0  1 /exam/image.raw
```

现在，*/dev/loop0*设备指向*/exam/image.raw*，你可以使用任何操作块设备的工具访问它。例如，这里使用 Sleuth Kit 的`mmls`命令，通过循环设备查看*image.raw*文件中的分区表：

```
# mmls /dev/loop0
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

     Slot    Start        End          Length       Description
00:  Meta    0000000000   0000000000   0000000001   Primary Table (#0)
01:  -----   0000000000   0000002047   0000002048   Unallocated
02:  00:00   0000002048   0058597375   0058595328   Linux (0x83)
03:  00:01   0058597376   0078129151   0019531776   Linux Swap / Solaris x86 (0x82)
04:  00:02   0078129152   0078231551   0000102400   NTFS (0x07)
05:  00:03   0078231552   0234441647   0156210096   Mac OS X HFS (0xaf)
```

当你不再需要循环设备时，只需按照以下方式分离它：

```
# losetup --detach /dev/loop0
```

循环设备是灵活且可配置的。在前面的 mmls 示例中，文件系统从第 2048 扇区开始。每次运行取证工具时，都可以指定偏移量，但为每个分区创建一个单独的设备（类似于*/dev/sda1*）会更容易。你可以通过指定正确的偏移量标志（`--offset`）和大小标志（`--sizelimit`）来仅为该分区创建一个单独的循环设备。然而，通常接受的做法是使用设备映射器。

你也可以手动使用 dmsetup 和映射表，如在《RAID 和多磁盘系统》的第 178 页中所描述的那样。然而，kpartx 工具自动化了为特定镜像文件创建分区设备的过程。以下示例中使用了一个具有四个分区的取证获取的镜像文件，展示了 kpartx 工具如何为每个分区创建映射设备：

```
# kpartx -r -a -v image.raw
add map loop0p1 (252:0): 0 58595328 linear /dev/loop0 2048
add map loop0p2 (252:1): 0 19531776 linear /dev/loop0 58597376
add map loop0p3 (252:2): 0 102400 linear /dev/loop0 78129152
add map loop0p4 (252:3): 0 156210096 linear /dev/loop0 78231552
```

在这里，kpartx 工具读取磁盘或镜像文件中的分区表，创建一个用于整个镜像的循环设备，然后为每个分区创建映射设备。`-r`标志确保驱动器循环和分区映射为只读，`-a`标志指示 kpartx 映射它找到的所有内容。使用详细标志`-v`来记录命令输出，并指示刚才映射了什么。

在这个示例中，创建了一个循环设备（*/dev/loop0*）用于整个镜像文件，并作为原始块设备可以访问。此外，分区设备现在可以在*/dev/mapper*目录中找到，你可以使用操作分区的取证工具访问它们，而无需指定任何偏移量。以下是一些用于某些分区的 Sleuth Kit 命令示例：

```
# fsstat /dev/mapper/loop0p1
FILE SYSTEM INFORMATION
--------------------------------------------
File System Type: Ext4
Volume Name:
Volume ID: d4605b95ec13fcb43646de38f7f49680
...
# fls /dev/mapper/loop0p3
r/r 4-128-1:    $AttrDef
r/r 8-128-2:    $BadClus
r/r 8-128-1:    $BadClus:$Bad
r/r 6-128-1:    $Bitmap
r/r 7-128-1:    $Boot
d/d 11-144-2:   $Extend
r/r 2-128-1:    $LogFile
r/r 0-128-1:    $MFT
...
# fsstat /dev/mapper/loop0p4
FILE SYSTEM INFORMATION
--------------------------------------------
File System Type: HFS+
File System Version: HFS+
...
```

从镜像文件映射到设备的文件系统可以安全地以只读模式挂载。这将允许你通过标准文件管理器、应用程序和其他文件分析工具访问它。你可以像示例中所示那样挂载和卸载循环分区：

```
# mkdir p3
# mount --read-only /dev/mapper/loop0p3 p3
# mc ./p3
...
# umount p3
# rmdir p3
```

在这里，创建了一个目录，*p3*，表示该分区，并且该目录与原始镜像文件位于同一目录下。然后，*p3*被用作挂载点（选择的挂载点可以是检查主机文件系统中的任何位置）。Midnight Commander（`mc`）是一个基于文本的文件管理器（Norton Commander 的克隆版），在本示例中用于查看挂载分区中的文件。当不再需要挂载点时，`umount`命令（此命令只有一个*n*）卸载文件系统，rmdir 则移除挂载点目录。这是传统的 Unix 方式，用于在主机系统上挂载和卸载文件系统。

当您不再需要驱动环回和分区映射时，可以通过使用 kpartx 删除（`-d`）标志和镜像文件名来删除它们，如下所示：

```
# kpartx -d image.raw
loop deleted : /dev/loop0
```

请注意，这个“删除”操作不会影响磁盘镜像的内容。删除的是环回设备和映射，而不是驱动镜像，驱动镜像并没有被修改。

如果原始镜像具有损坏或被覆盖的分区表，您可以扫描镜像文件查找文件系统，并使用 dmsetup 手动将文件系统映射为设备（使用 dmsetup 表格）。

创建、挂载、卸载或分离环回设备时需要 root 权限。操作*/dev/loopX*设备时也需要 root 权限，尤其是使用取证工具时。本节中的示例是作为 root 用户运行的，以简化命令行的复杂性，使其更易理解。通过在命令前加`sudo`，可以作为非 root 用户执行特权命令。

#### *取证格式镜像文件*

ewflib 软件包包括一个名为 ewfmount 的工具，用于“挂载”取证镜像的内容，使其可以作为常规原始镜像文件访问。

以下示例展示了一组**.e01*文件。通过`mkdir`创建了一个挂载点，在本例中为`raw`，该挂载点将包含原始镜像文件：

```
# ls
image.E01  image.E02  image.E03  image.E04  image.E05
# mkdir raw
```

ewfmount 工具创建了一个 FUSE 文件系统，其中包含一个来自一个或多个 EWF 文件的虚拟原始镜像。您可以使用第一个 EnCase EWF 文件和挂载点运行`ewfmount`命令来访问一个原始镜像文件，如下所示：

```
# ewfmount image.E01 raw
ewfmount 20160424

# ls -l raw
total 0
-r--r--r-- 1 root root 16001269760 May 17 21:20 ewf1
```

然后，您可以使用不直接支持 EWF 格式的工具来操作这个虚拟原始镜像文件。在下面的例子中，使用十六进制编辑器（不支持 EWF）在扇区模式下分析原始镜像：

```
# hexedit -s raw/ewf1
...
```

kpartx 工具再次用于识别分区并创建相应的环回设备，从而使可以使用操作块设备的工具，并允许挂载文件系统进行常规浏览。以下是使用 ewfmount 挂载的**.e01*文件的 kpartx 输出：

```
# kpartx -r -a -v raw/ewf1
add map loop0p1 (252:0): 0 29848707 linear /dev/loop0 63
add map loop0p2 (252:1): 0 2 linear /dev/loop0 29848770
add map loop0p5 : 0 1397592 linear /dev/loop0 29848833
```

让我们继续使用这个例子来为一个分区创建挂载点，并挂载和访问文件系统：

```
# mkdir p1
# mount --read-only /dev/mapper/loop0p1 p1
# ls p1
cdrom  home/       lib32/       media/  proc/  selinux/  tmp/  vmlinuz
bin/     dev/   initrd.img  lib64        mnt/    root/  srv/      usr/
boot/    etc/   lib/        lost+found/  opt/    sbin/  sys/      var/
...
```

在此示例中，创建了一个与分区对应的挂载点并将分区设备挂载到该点，然后通过`ls`访问文件系统。如果可能，避免在挂载证据文件和容器时使用*/mnt*或其他共享挂载目录。当镜像的挂载点与其他相关案件文件位于同一工作目录时，取证工作会更容易进行。

如之前所述，当工作完成后，您需要清理挂载点和虚拟文件。这一步依旧按照反向顺序进行：

```
# umount p1
# kpartx -d raw/ewf1
loop deleted : /dev/loop0
# fusermount -u raw
# rmdir p1 raw
```

本示例中展示了`fusermount`命令，但标准的 Linux `umount`命令也可以使用。确保您的当前工作目录不在挂载点内，并且没有程序在挂载点内打开文件。如果满足这两个条件，清理步骤将会失败。

使用 SquashFS 法医证据容器时，您可以通过使用 `sfsimage -m` 挂载 **.sfs** 文件，创建分区设备，然后挂载所需的分区来访问原始镜像。然后，您可以在目标镜像的文件系统上执行常规命令。完整的示例如下：

```
# sfsimage -m image.sfs
image.sfs.d mount created
# kpartx -r -a -v image.sfs.d/image.raw
add map loop1p1 (252:0): 0 29848707 linear /dev/loop1 63
add map loop1p2 (252:1): 0 2 linear /dev/loop1 29848770
add map loop1p5 : 0 1397592 linear /dev/loop1 29848833
# mkdir p1
# mount /dev/mapper/loop1p1 p1
mount: /dev/mapper/loop1p1 is write-protected, mounting read-only
# ls -l
...
```

完成访问原始镜像及其文件系统后，SquashFS 法医证据容器的清理过程也需要反向操作。`sfsimage -u` 命令卸载 SquashFS 文件系统，如本示例所示：

```
# umount p1
# kpartx -d image.sfs.d/image.raw
loop deleted : /dev/loop1
# sfsimage -u image.sfs.d/
image.sfs.d unmounted
```

本节演示了几种访问法医格式内容的方法，既可以作为块设备，也可以作为常规文件系统。ewfmount 工具也适用于 FTK SMART 文件。Afflib 有一个类似的工具叫做 affuse，用于挂载 **.aff** 文件。ewfmount 和 affuse 都可以在各自格式的单个文件或分割文件上操作。

请注意，许多法医工具（例如 Sleuth Kit）能够直接操作法医格式，而无需原始块设备或原始文件。

#### *使用 xmount 准备启动镜像*

法医调查员通常希望使用非法医工具（如文件管理器、办公套件、应用程序或其他文件查看工具）检查目标驱动器镜像。这可以通过使驱动器内容通过只读挂载安全地提供给本地考官机器访问来实现。

在某些情况下，启动目标驱动器到虚拟机中进行观察和直接与目标环境互动是有用的。这允许您查看目标桌面并使用目标计算机上安装的程序。为此，您可以使用本节中描述的多种工具。

xmount（发音为“crossmount”）工具创建一个虚拟磁盘镜像，您可以使用虚拟机软件（如 VirtualBox 或 kvmqemu）启动。xmount 工具允许您模拟一个读写驱动器，使虚拟机认为磁盘是可写的，但它仍然保持镜像为只读状态。提供多种虚拟机输出格式，包括 raw、DMG、VDI、VHD、VMDK 和 VMDKS。

输入格式包括法医获取的镜像文件，如 **.raw**、EnCase **.ewf** 和 AFFlib **.aff** 文件。

这是一个使用 xmount 设置的原始镜像（*image.raw*），作为 VirtualBox **.vdi** 文件的示例：

```
$ mkdir virtual
$ xmount --cache xmount.cache --in raw image.raw --out vdi virtual
$ ls virtual/
image.info  image.vdi
$ cat virtual/image.info
------> The following values are supplied by the used input library(ies) <------

--> image.raw <--
RAW image assembled of 1 piece(s)
30016659456 bytes in total (27.955 GiB)

------> The following values are supplied by the used morphing library <------

None
$ virtualbox
```

在此示例中，创建了一个名为 *virtual* 的目录，用于保存虚拟镜像文件（该文件将通过 FUSE 挂载）。从现有的 *image.raw* 文件中，`xmount` 命令在 *./virtual* 目录中创建一个写缓存的 VirtualBox VDI 镜像。这仅仅是镜像文件的虚拟表示；它不会被复制或转换（因此不会浪费考官机器的磁盘空间）。`--in` 和 `--out` 标志指定使用的镜像格式。输入格式必须是 raw、AFF 或 EWF。可以选择多种输出格式。

在虚拟机中启动操作系统镜像可能会遇到困难，特别是当已安装的操作系统期望的硬件配置与虚拟机提供的配置不同。通常，Linux 安装较少遇到此问题，但 Windows 和 OS X 可能会出现此问题。为了解决这个问题，创建了两个工具，opengates 和 openjobs，用于准备 Windows 和 OS X 镜像，以便在虚拟环境中安全启动目标磁盘。我不会介绍如何使用 opengates 和 openjobs，但你可以在*[`www.pinguin.lu/openjobs/`](https://www.pinguin.lu/openjobs/)*和*[`www.pinguin.lu/opengates/`](https://www.pinguin.lu/opengates/)*上找到更多相关信息。

当你不再需要虚拟机镜像时，可以通过卸载虚拟镜像并删除挂载点目录来进行清理：

```
$ fusermount -u virtual
$ ls virtual/
$ rmdir virtual
```

一个包含在使用虚拟机时写入数据的*xmount.cache*文件可能存在。如果需要继续之前的虚拟机会话，可以保存该文件，或者选择删除它。

### 虚拟机镜像

随着家庭计算机性能的提升、现代 CPU 支持的硬件虚拟化和廉价或免费的虚拟化软件的可用性，对虚拟机镜像内容的分析需求不断增加。在某些情况下，你可能会在目标 PC 上发现许多虚拟机镜像。本节将重点介绍如何访问常见的虚拟机镜像文件类型，如 QCOW2、VDI、VMDK 和 VHD。

#### *QEMU QCOW2*

QCOW2 格式是 Linux 中常见的虚拟机镜像类型，并且被 QEMU 模拟器使用。在本节中，我将展示如何将 QCOW2 镜像作为块设备提供，并安全地挂载以供浏览。

libqcow-utils 软件包（由 Joachim Metz 编写，ewflib 的作者）包含 qcowinfo 和 qcowmount 工具。你可以像使用之前示例中的 ewfinfo 和 ewfmount 工具一样使用这两个工具。但以下示例展示了使用`qemu-img`命令、nbd 内核模块和 qemu-nbd 工具的替代方法。这种方法具有性能优势，因为它在内核中运行，并且省去了几个步骤，因为你不需要使用 kpartx。

给定一个**.qcow2**文件，`qemu-img`命令可以提供该文件的概述：

```
# qemu-img info image.qcow2
image: image.qcow2
file format: qcow2
virtual size: 5.0G (5368709120 bytes)
disk size: 141M
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
    refcount bits: 16
    corrupt: false
```

若要使用 nbd 以原始镜像表示访问 QCOW 镜像，你需要加载 nbd 内核模块：

```
# modprobe nbd
# dmesg | grep nbd
[16771.003241] nbd: registered device at major 43
```

与`losetup`命令不同，设备不会自动选择。需要按如下方式指定一个*/dev/nbd*设备：

```
# qemu-nbd --read-only --connect /dev/nbd0 image.qcow2
# dmesg | grep nbd0
[16997.777839]  nbd0: p1
```

在这里，QCOW2 镜像文件已连接到内核模块，并以只读模式挂载，分区设备已自动检测。你可以像示例中所示，使用此原始设备进行取证工具分析：

```
# mmls /dev/nbd0
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

     Slot    Start        End          Length       Description
00:  Meta    0000000000   0000000000   0000000001   Primary Table (#0)
01:  -----   0000000000   0000002047   0000002048   Unallocated
02:  00:00   0000002048   0010485759   0010483712   Linux (0x83)
```

分区设备（本示例中的原始设备名称加上`p1`）也可以直接用于取证工具。为了说明这一点，以下是直接在分区设备的文件系统上操作的`fls`命令：

```
# fls /dev/nbd0p1
d/d 11: lost+found
r/r 12: hosts
d/d 327681:     $OrphanFiles
...
```

在本地挂载设备进行浏览是很简单的。创建一个本地挂载点目录，并按正常方式挂载文件系统，如下所示：

```
# mkdir p1
# mount /dev/nbd0p1 p1
mount: /dev/nbd0p1 is write-protected, mounting read-only
# ls p1
hosts  lost+found/
```

这里的清理过程类似于使用 loop 设备的示例，但步骤更少。所有进程应该关闭文件，且在卸载前需要离开挂载目录。`qemu-nbd disconnect` 命令指定设备名称，将设备从内核中注销，如下所示：

```
# umount p1
# qemu-nbd --read-only --disconnect /dev/nbd0
/dev/nbd0 disconnected
# rmdir p1
```

一个可选步骤是使用 `rmmod nbd` 移除内核模块。但如果您还会进行更多 QCOW 挂载，保留它也没有问题。您还可以通过将其添加到 */etc/modules* 文件中，在启动时自动加载 nbd 模块。

#### *VirtualBox VDI*

VirtualBox 是由 Oracle（前身为 Sun Microsystems）维护的开源项目。虽然它支持多种虚拟机镜像格式，但以下示例中使用的是 VirtualBox VDI 镜像。与之前一样，使用相同的 `qemu-nbd` 命令，但这次使用的是 OpenSolaris 镜像。

VirtualBox 软件包包括多个实用工具；这里显示的是 VBoxManage 工具，提供有关 VDI 镜像的信息：

```
# VBoxManage showhdinfo OpenSolaris.vdi
UUID:           0e2e2466-afd7-49ba-8fe8-35d73d187704
Parent UUID:    base
State:          created
Type:           normal (base)
Location:       /exam/OpenSolaris.vdi
Storage format: VDI
Format variant: dynamic default
Capacity:       16384 MBytes
Size on disk:   2803 MBytes
Encryption:     disabled
```

您可以使用 `qemu-nbd` 和 nbd 内核模块来挂载 VirtualBox 镜像（如前节中使用 QCOW2 时所见）。这里展示的 Open-Solaris 示例与 Windows 和 Linux 使用的分区方案略有不同。也展示了多个磁盘切片^(1)：

```
# qemu-nbd -c /dev/nbd0 OpenSolaris.vdi
# dmesg
...
[19646.708351]  nbd0: p1
                p1: <solaris: [s0] p5 [s1] p6 [s2] p7 [s8] p8 >
```

在本示例中，单一的 Solaris 分区（`p1`）包含多个切片（`p5`、`p6`、`p7` 和 `p8`）。

您可以使用与前面 QEMU 示例相同的方法访问原始设备和分区设备，然后将分区以只读方式挂载到本地挂载点。这里同样不需要使用 kpartx 来查找分区，因为内核会自动完成。访问完分区（或切片）后，执行清理步骤，卸载文件系统并断开 nbd 设备连接。

#### *VMWare VMDK*

*虚拟机磁盘（VMDK）*格式由 VMWare 的虚拟机软件产品使用。以下示例使用 libvmdk-utils 软件包在 Apple Lion VMDK 镜像上，该镜像被分割成多个部分：

```
# ls
lion-000001-s001.vmdk  lion-000003-s007.vmdk  lion-s009.vmdk
lion-000001-s002.vmdk  lion-000003-s008.vmdk  lion-s010.vmdk
lion-000001-s003.vmdk  lion-000003-s009.vmdk  lion-s011.vmdk
lion-000001-s004.vmdk  lion-000003-s010.vmdk  lion-s012.vmdk
lion-000001-s005.vmdk  lion-000003-s011.vmdk  lion-s013.vmdk
lion-000001-s006.vmdk  lion-000003-s012.vmdk  lion-s014.vmdk
lion-000001-s007.vmdk  lion-000003-s013.vmdk  lion-s015.vmdk
lion-000001-s008.vmdk  lion-000003-s014.vmdk  lion-s016.vmdk
...
```

您可以使用 `vmdkinfo` 获取有关已组装镜像及每个“扩展”（Extents）的信息：

```
# vmdkinfo lion.vmdk
vmdkinfo 20160119

VMware Virtual Disk (VMDK) information:
        Disk type:                      2GB extent sparse
        Media size:                     42949672960 bytes
        Content identifier:             0xadba0513
        Parent content identifier:      0xffffffff
        Number of extents:              21

Extent: 1
        Filename:                       lion-s001.vmdk
        Type:                           Sparse
        Start offset:                   0
        Size:                           2146435072 bytes
...
```

创建挂载点并挂载镜像文件后，可以将其作为原始镜像文件进行访问：

```
# mkdir lion
# vmdkmount lion.vmdk lion
vmdkmount 20160119
# ls -ls lion
total 0
0 -r--r--r-- 1 root root 42949672960 May 17 22:24 vmdk1
# mmls lion/vmdk1
GUID Partition Table (EFI)
Offset Sector: 0
Units are in 512-byte sectors

     Slot    Start        End          Length       Description
00:  Meta    0000000000   0000000000   0000000001   Safety Table
01:  -----   0000000000   0000000039   0000000040   Unallocated
02:  Meta    0000000001   0000000001   0000000001   GPT Header
03:  Meta    0000000002   0000000033   0000000032   Partition Table
04:  00      0000000040   0000409639   0000409600   EFI System Partition
05:  01      0000409640   0082616503   0082206864   Untitled
06:  02      0082616504   0083886039   0001269536   Recovery HD
07:  -----   0083886040   0083886079   0000000040   Unallocated
```

如本章前面所示，使用 kpartx 将创建关联的磁盘和分区块设备。然后，您可以直接使用取证分析工具对其进行分析，或将其挂载到本地机器上以浏览文件系统。

#### *Microsoft VHD*

有多种方法可以使 Microsoft VHD 虚拟镜像格式变得可访问。例如，您可以使用 qemu-nbd 方法或使用 libvhdi-utils 配合 vhdiinfo 和 vhdimount。

另一种方法是使用带有 Xen blktap xapi 接口的 blktap-utils。与 nbd 方法类似，blktap 需要你插入内核模块并手动分配一个设备。会启动一个 tapdisk 进程，附加到驱动程序，并指示其打开磁盘映像。blktap-utils 的手册页面不太有用，但你可以在 Xen 网站上找到相关描述，链接为 *[`wiki.xen.org/wiki/Mounting_a_.vhd_disk_image_using_blktap/tapdisk`](http://wiki.xen.org/wiki/Mounting_a_.vhd_disk_image_using_blktap/tapdisk)* 和 *[`lists.xen.org/archives/html/xen-api/2012-05/msg00149.html`](http://lists.xen.org/archives/html/xen-api/2012-05/msg00149.html)*。

为了完成这一部分，我将重复使用 libvhdi 工具设置设备的过程。为了简便起见，之前的示例使用了特权 root 用户。但接下来的示例演示了授权使用`sudo`的非特权用户。

要作为非特权用户运行 FUSE 的`mount`和`unmount`命令，你需要在*/etc/fuse.conf*中设置*user_allow_other*。

你可以使用`vhdiinfo`来查找关于映像的信息，不需要特别的权限：

```
$ vhdiinfo windows.vhd
vhdiinfo 20160111
Virtual Hard Disk (VHD) image information:
        Format:                 1.0
        Disk type:              Dynamic
        Media size:             136365211648 bytes
        Identifier:             c9f106a3-cf3f-6b42-a13f-60e349faccb5
```

你可以在没有 root 权限的情况下使用 FUSE 挂载映像，但你需要明确地指示`vhdimount`命令通过添加`-X allow_root`标志来允许 root 用户访问。这个标志也需要允许 root 通过`sudo`执行进一步的操作（例如使用 kpartx 创建块设备）：

```
$ mkdir raw
$ vhdimount -X allow_root windows.vhd raw
vhdimount 20160111

$ ls -l raw/
total 0
-r--r--r-- 1 holmes holmes 136365211648 Jan 20 08:14 vhdi1
```

原始映像现在可以在*./raw*目录中找到，你可以使用标准工具访问它。要创建循环设备和映射器设备，使用`sudo`命令运行`kpartx`。一旦设备创建完成，你可以通过`sudo`命令访问它们。所有块设备的访问都需要`sudo`命令。以下是使用 kpartx 和 fls 的示例：

```
$ sudo kpartx -r -a -v ./raw/vhdi1
add map loop0p1 (252:0): 0 266334018 linear /dev/loop0 63
$ sudo fls /dev/mapper/loop0p1
r/r 4-128-4:    $AttrDef
r/r 8-128-2:    $BadClus
r/r 8-128-1:    $BadClus:$Bad
r/r 6-128-1:    $Bitmap
r/r 7-128-1:    $Boot
d/d 11-144-4:   $Extend
r/r 2-128-1:    $LogFile
r/r 0-128-1:    $MFT
```

挂载文件系统同样需要`sudo`，并且通过明确指定`-o ro`可以将其挂载为只读。这里展示了创建挂载点、挂载上一个示例中的文件系统并通过`ls`命令访问它的示例：

```
$ mkdir p1
$ sudo mount -o ro /dev/mapper/loop0p1 p1
$ ls p1
AUTOEXEC.BAT                 IO.SYS          $RECYCLE.BIN/
...
```

清理此会话需要`sudo`来卸载原始映像并移除循环设备和映射器设备。你可以在没有 root 权限的情况下移除**.vhd**文件的 FUSE 挂载。以下是步骤：

```
$ sudo umount p1
$ sudo kpartx -d raw/vhdi1
loop deleted : /dev/loop0
$ fusermount -u raw
```

你可以通过编辑*/etc/sudoers*文件来配置`sudo`命令。本书中的许多示例为了简化命令行操作，使用了 root 用户，减少了复杂命令的数量。作为一种良好的实践，建议作为非特权用户工作，并使用诸如`sudo`之类的安全机制。

### 操作系统加密文件系统

现在让我们来看一下如何访问常见的加密文件系统。重点不在于密钥恢复（虽然我提供了一些建议），而是如何使用已知密钥访问文件系统。假设密钥或密码可以通过内存转储、企业组织中的托管/备份、依法强制提供的个人、愿意提供帮助的受害者、商业恢复服务/软件或其他来源获得。

你可以使用各种分区分析工具来确定文件系统加密的类型，这些工具能够识别头部、魔法数字以及其他特定加密文件系统类型的独特标志物。在取证环境中，你可以在 *[`encase-forensic-blog.guidancesoftware.com/2014/04/version-7-tech-tip-spotting-full-disk.html`](http://encase-forensic-blog.guidancesoftware.com/2014/04/version-7-tech-tip-spotting-full-disk.html)* 找到有关识别文件系统加密的概述。

在本节中，你将找到有关特定加密镜像的信息，这些信息用于创建一个未加密的块设备或文件，你可以使用取证工具访问，或者安全地挂载以进行本地浏览。

#### *微软 BitLocker*

微软当前的默认文件系统加密是 BitLocker。它在块级别进行加密，保护整个卷。为可移动介质设计的 BitLocker 变体称为 BitLocker-To-Go，它在常规的未加密文件系统上使用加密容器文件。本节中的示例展示了两个开源工具：dislocker 和 libbde。

由 Romain Coltel 编写，你可以在 *[`github.com/Aorimn/dislocker/`](https://github.com/Aorimn/dislocker/)* 找到 dislocker 包。它提供了处理 BitLocker 卷的各种工具，包括查看元数据、创建解密的镜像文件和 FUSE 挂载卷。

dislocker-find 工具扫描所有附加的分区设备和指定文件，以识别任何 BitLocker 卷的存在。如果在将设备附加到获取主机的过程中已经识别了目标设备，可能就不需要扫描 BitLocker 设备了。

`dislocker-metadata` 命令提供了 BitLocker 驱动器的概览。下一个示例来自一个 USB 闪存驱动器的镜像。整个驱动器都被加密，并且没有分区表。可以通过以下方式查询该镜像文件：

```
# dislocker-metadata -V bitlocker-image.raw
...
Wed Jan 20 13:46:06 2016 [INFO] BitLocker metadata found and parsed.
Wed Jan 20 13:46:06 2016 [INFO] =====[ Volume header informations ]=====
Wed Jan 20 13:46:06 2016 [INFO]   Signature: 'MSWIN4.1'
Wed Jan 20 13:46:06 2016 [INFO]   Sector size: 0x0200 (512) bytes
...
Wed Jan 20 13:46:06 2016 [INFO]   Number of sectors (64 bits): 0x0000000200000000
    (8589934592) bytes
Wed Jan 20 13:46:06 2016 [INFO]   MFT start cluster: 0x0000000000060001 (393217)
    bytes
...
Wed Jan 20 13:46:06 2016 [INFO] =====================[ BitLocker information
    structure ]=====================
Wed Jan 20 13:46:06 2016 [INFO]   Signature: '-FVE-FS-'
Wed Jan 20 13:46:06 2016 [INFO]   Total Size: 0x02f0 (752) bytes (including
    signature and data)
Wed Jan 20 13:46:06 2016 [INFO]   Version: 2
Wed Jan 20 13:46:06 2016 [INFO]   Current state: ENCRYPTED (4)
Wed Jan 20 13:46:06 2016 [INFO]   Next state: ENCRYPTED (4)
Wed Jan 20 13:46:06 2016 [INFO]   Encrypted volume size: 7918845952 bytes
    (0x1d8000000), ~7552 MB
...
```

此命令的输出提供了许多详细的加密信息，这里未展示。你可以将 `dislocker-metadata` 的输出保存为文本文件，以供文档记录使用。该命令也可以直接在附加的设备上操作。

与之前的密码和加密示例一样，假设您已经拥有密钥。有些商业工具可用于尝试密码暴力破解以恢复密钥。此外，您还可以使用波动性插件从内存镜像中提取 FVEK（*[`github.com/elceef/bitlocker/`](https://github.com/elceef/bitlocker/)*），并且可以将该工具与 inception memorydumping 工具一起使用。这里不涉及这些工具的使用。

您可以创建一个虚拟文件或块设备，在“原地”操作解密后的磁盘镜像视图。执行此过程与 “VM Images” 中的示例类似，详见 第 237 页。dislocker 软件包提供了一个工具，可以创建一个虚拟表示解密卷的 FUSE 文件系统：

```
# mkdir clear
# dislocker-fuse -u -V bitlocker-image.raw clear
Enter the user password:
# ls -l clear/
total 0
-rw-rw-rw- 1 root root 7918845952 Jan  1  1970 dislocker-file
...
```

出现在 *clear* 目录中的文件是加密文件系统的解密表示，您可以使用常规的取证工具对其进行操作。以下是使用 Sleuth Kit 的 fsstat 工具的示例：

```
# fsstat clear/dislocker-file
FILE SYSTEM INFORMATION
--------------------------------------------
File System Type: FAT32

OEM Name: MSDOS5.0
Volume ID: 0x5a08a5ba
Volume Label (Boot Sector): NO NAME
Volume Label (Root Directory): MY SECRETS
File System Type Label: FAT32
Next Free Sector (FS Info): 34304
Free Sector Count (FS Info): 15418664
...
```

您可以安全地挂载解密后的文件系统镜像以进行正常浏览。`mount` 命令具有 `loop` 选项，允许直接挂载分区镜像文件，如下所示：

```
# mkdir files
# mount -o loop,ro clear/dislocker-file files
# ls files
Penguins.jpg  private/  System Volume Information/
...
```

本示例中的清理工作很简单，只需卸载文件的挂载点，移除 FUSE 挂载，并删除挂载目录：

```
# umount files
# rmdir files
# fusermount -u clear
# rmdir clear
```

请注意，前面的示例是使用 root 权限执行的，以减少复杂性并使其更易于理解。您可以作为非特权用户执行相同的命令，如下所示：

```
$ dislocker-metadata -V bitlocker-image.raw
$ mkdir clear files
$ dislocker-fuse -u -V bitlocker-image.raw -- -o allow_root clear
$ sudo mount -o loop,ro,uid=holmes clear/dislocker-file files
...
$ sudo umount files
$ fusermount -u clear
$ rmdir clear files
```

在这里，`dislocker-fuse` 将 `-o allow_root` 传递给 FUSE 驱动程序，允许使用 `sudo` 进行挂载和卸载。`uid=holmes` 确保 Holmes 先生可以在没有 root 权限的情况下访问挂载的文件。假设 Holmes 先生是 FUSE Unix 组的成员，并且 */etc/fuse.conf* 文件中包含 *user_allow_other* 行。

使用 dislocker，您可以提供三种可能的凭证来解锁 BitLocker 容器。`-u` 标志（在前面的示例中使用）指定请求用户密码。`-p` 标志提供恢复密码（长达 48 位）。`-f` 标志指定一个密钥文件（BEK 文件）。

使用恢复密码（`-p`）而不是用户密码（`-u`）需要手动输入 48 位恢复密码，如下所示：

```
# dislocker-fuse -p -V bitlocker-image.raw clear
Enter the recovery password: XXXXXX-XXXXXX-XXXXXX-XXXXXX-XXXXXX-XXXXXX-XXXXXX-XXXXXX
Valid password format, continuing.
```

此命令的非 root 版本将标志传递给 FUSE，允许使用 `sudo` 进行挂载：

```
$ dislocker-fuse -p -V bitlocker-image.raw -- -o allow_root clear
```

您还可以解密 BitLocker 镜像，并将其单独保存为常规的文件系统镜像（仅保存指定的卷，而不是分区表或其他分区）。根据 BitLocker 镜像的大小，这将需要一些时间，因为整个镜像会被解密并写入到磁盘上的新镜像文件。您需要进行一些容量规划，因为加密和解密后的两个镜像会占用获取主机的存储空间。您可以按如下方式创建解密后的卷版本：

```
# dislocker-file -u -V bitlocker-image.raw bitlocker-image.clear
Enter the user password:
# ls -hs
total 15G
7.4G bitlocker-image.clear  7.4G bitlocker-image.raw
```

解密后的镜像文件与原始文件的大小相同，因为每个 BitLocker 块都已被解密，并将明文块写入新镜像中。此命令不需要 root 权限。

现在你可以挂载解密后的 BitLocker 镜像文件，并使用带有`loop`选项的`mount`命令将其作为分区访问：

```
# mkdir files
# mount -o loop,ro bitlocker-image.clear files
# ls files/
Penguins.jpg  private/  System Volume Information/
```

唯一不同的命令是非 root 用户使用的`mount`：

```
$ sudo mount -o loop,ro,uid=holmes bitlocker-image.clear files
```

由于 BitLocker 是主流操作系统平台上的默认文件系统加密，因此提供第二个使用不同软件包的示例是很有价值的。libbde 包（由 ewflib 的作者 Joachim Metz 编写）也提供了访问 BitLocker 镜像的库和工具。

下一个示例比前一个稍微复杂一些，因为它涉及到一个带有常规分区表的笔记本硬盘（与没有分区表的 USB 闪存驱动器不同）。在计算了来自 mmls 输出的偏移量后，展示了 bdeinfo 工具，它提供了一个紧凑的 BitLocker 容器概览。

无论是 dislocker 还是 libbde，都可以给定字节偏移量，表示 BitLocker 加密卷的起始位置。但在处理没有分区的卷/分区或设备的镜像文件时，这一点是多余的。在本示例中，获取的镜像有一个分区表，因此必须计算 BitLocker 加密卷的偏移量（以字节为单位）。

**注意**

*始终确保了解命令使用的单位。一些工具使用扇区偏移量，而另一些使用字节偏移量。区分并转换这两者非常重要。*

下一个示例演示了如何确定字节偏移量。Sleuth Kit 的`mmls`命令显示了分区表和每个分区的扇区偏移量。必须将扇区偏移量转换为字节偏移量，然后可以与解密工具一起使用：

```
# mmls image0.raw
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

     Slot    Start        End          Length       Description
00:  Meta    0000000000   0000000000   0000000001   Primary Table (#0)
01:  -----   0000000000   0000002047   0000002048   Unallocated
02:  00:00   0000002048   0004098047   0004096000   NTFS (0x07)
03:  00:01   0004098048   0625140399   0621042352   NTFS (0x07)
04:  -----   0625140400   0625142447   0000002048   Unallocated
# echo $((4098048*512))
2098200576
```

你可以通过将`mmls`显示的扇区偏移量乘以扇区大小，将其转换为字节偏移量。在命令行中，使用 Bash 数学扩展非常方便。在本示例中，扇区偏移量是 4098048，扇区大小是 512。将这两个数相乘，得到字节偏移量为 2098200576。你可以将此值用于`bdeinfo`命令，如下所示：

```
# bdeinfo -o 2098200576 image0.raw
bdeinfo 20160119

BitLocker Drive Encryption information:
        Encryption method:              AES-CBC 128-bit with Diffuser
        Volume identifier:              5f61cbf2-75b5-32e5-caef-537fce3cf412
        Creation time:                  Jan 10, 2014 17:43:50.838892200 UTC
        Description:                    Notebook System 15.01.2014
        Number of key protectors:       2

Key protector 0:
        Identifier:                     3cd1fd6c-2ecb-2dc7-c150-839ce9e710b6
        Type:                           TPM

Key protector 1:
        Identifier:                     837ef544-e1ca-65c1-a910-83acd492bc1a
        Type:                           Recovery password
...
```

`bdemount`命令的操作方式类似于`dislocker`命令，并创建一个虚拟文件，表示解密后的镜像（这里的完整密钥已简化）：

```
# mkdir raw
# bdemount -o 2098200576 -r 630641-...-154814 image.raw raw
```

文件将出现在*./raw*目录中，你可以直接分析它，或将其挂载到循环设备上进行常规浏览。挂载命令与前面的 BitLocker 示例相同，因此此处不再重复。

#### *Apple FileVault*

苹果的文件系统加密内置在 OS X 中，名为 FileVault。它也是一种块级加密系统，市面上有几种开源工具可以用来解密它。我在这里描述的两个工具是 libfvde 和 VFDecrypt。（libfvde 软件包是由 Omar Choudary 和 Joachim Metz 编写的，你可以在* [`github.com/libyal/libfvde/`](https://github.com/libyal/libfvde/) *找到它。）

在使用 libfvde 工具之前，你需要计算 FileVault 加密卷的正确字节偏移量。`mmls`命令提供了该卷的扇区偏移量，需要将其转换为字节：

```
# mmls image.raw
GUID Partition Table (EFI)
Offset Sector: 0
Units are in 512-byte sectors

     Slot    Start        End          Length       Description
00:  Meta    0000000000   0000000000   0000000001   Safety Table
01:  -----   0000000000   0000000039   0000000040   Unallocated
02:  Meta    0000000001   0000000001   0000000001   GPT Header
03:  Meta    0000000002   0000000033   0000000032   Partition Table
04:  00      0000000040   0000409639   0000409600   EFI System Partition
05:  01      0000409640   0235708599   0235298960   HDD
06:  02      0235708600   0236978135   0001269536   Recovery HD
07:  -----   0236978136   0236978175   0000000040   Unallocated
# echo $((409640*512))
209735680
```

使用简单的 Bash 数学扩展将扇区偏移量乘以扇区大小可以得到一个字节偏移量 209735680，可以用来进行 fvdeinfo 和 fvdemount 工具的操作。

fvdeinfo 工具提供了一个 FileVault 加密卷的概览：

```
# fvdeinfo -o 209735680 image.raw
fvdeinfo 20160108

Core Storage information:

Physical volume:
        Size: 120473067520             bytes
        Encryption method:             AES XTS

Logical volume:
        Size:                          120137519104 bytes
```

要解密 FileVault 卷，你需要恢复*EncryptedRoot.plist.wipekey*文件并提供用户密码或恢复密钥。你可以使用 Sleuth Kit 工具查找并提取*wipekey*文件，如下所示：

```
# fls -r -o 235708600 image.raw | grep EncryptedRoot.plist.wipekey
+++++ r/r 1036: EncryptedRoot.plist.wipekey
# icat -o 235708600 image.raw 1036 > EncryptedRoot.plist.wipekey
```

恢复 HD 分区的递归 fls 输出使用了通过 mmls 找到的扇区偏移量。输出通过 grep 查找*EncryptedRoot.plist.wipekey*文件。找到后，使用 icat 工具提取该文件（使用 inode，在这个例子中是 1036）。注意 fls 和 icat 使用的是扇区偏移，而不是字节偏移。

使用`-r`标志和现在恢复的*EncryptedRoot.plist.wipekey*文件，24 字符的恢复密钥被使用。然后，你可以使用这个密钥创建一个解密的卷的 FUSE 挂载，如下所示（恢复密钥已被简化）：

```
# mkdir clear
# fvdemount -o 209735680 -r FKZV-...-H4PD -e EncryptedRoot.plist.wipekey image.raw
    clear
fvdemount 20160108

# ls -l clear
total 0
-r--r--r-- 1 root root 120137519104 Jan 20 22:23 fvde1
...
```

你可以提供用户密码（`-p`）代替恢复密钥（`-r`），并且通过使用*EncryptedRoot.plist.wipekey*文件，你可以使用常规取证工具访问结果卷镜像。下面展示了一个使用 Sleuthkit 的 fsstat 工具在解密卷上的示例：

```
# fsstat clear/fvde1
FILE SYSTEM INFORMATION
--------------------------------------------
File System Type: HFS+
File System Version: HFS+

Volume Name: HDD
...
```

你也可以将这个解密后的卷作为常规文件系统挂载进行浏览，如下所示：

```
# mkdir files
# mount -o loop,ro clear/fvde1 files
# ls -l files
total 8212
drwxrwxr-x 1 root   80      50 Mar  2  2015 Applications/
drwxr-xr-x 1 root root      39 Jun  2  2015 bin/
drwxrwxr-t 1 root   80       2 Aug 25  2013 cores/
dr-xr-xr-x 1 root root       2 Aug 25  2013 dev/
...
```

当分析工作完成后，你需要进行一些清理工作：

```
# umount files
# rmdir files
# fusermount -u clear
# rmdir clear
```

请注意，前面的示例是在 root 权限下完成的，以减少复杂性并使其更容易理解。大多数命令可以在非 root 用户下运行，但有一些例外情况。下面展示了在非特权用户下运行时命令有所不同的示例：

```
$ fvdemount -o 209735680 -r FKZV-...-H4PD -e EncryptedRoot.plist.wipekey image.raw
    -X allow_root clear
$ sudo mount -o loop,ro clear/fvde1 files
$ sudo ls files/Users/somebody/private/directory
$ sudo umount files
```

`fvdemount`命令中的`-X allow_root`字符串允许 root 访问 FUSE 挂载目录。`sudo`命令是挂载和卸载 hfsplus 文件系统所需的。当浏览文件系统时，如果文件系统权限限制了对文件或目录的访问，你可能也需要使用`sudo`命令。

还有一些其他著名的开源工具可以用于操作 FileVault 镜像。VFDecrypt 工具也提供 FileVault 镜像的解密。最初由 Ralf-Philipp Weinmann、David Hulton 和 Jacob Appelbaum 编写，现在由 Drake Allegrini 维护。你可以在* [`github.com/andyvand/VFDecrypt/`](https://github.com/andyvand/VFDecrypt/)*找到它。它可以将镜像解密为未加密的卷镜像。

FileVault 破解软件是由与 VFDecrypt 相同的一些作者创建的，你可以在* [`openciphers.sourceforge.net/oc/vfcrack.php`](http://openciphers.sourceforge.net/oc/vfcrack.php)*找到它。

#### *Linux LUKS*

开源世界中有许多文件加密系统。像 eCryptfs 或 encfs 这样的系统是基于目录的，而像 GPG 和各种加密工具则是针对单个文件操作的。

本节主要关注 LUKS 加密系统，但也会简要介绍纯 dm-crypt 和 loop-AES。使用 cryptsetup 工具，您可以设置这三者。（您还可以使用 cryptsetup 工具管理 True-Crypt 卷，我将在接下来的部分中描述。）

以下示例操作的是一个具有 LUKS 加密文件系统的取证获取映像。我们将创建一个块设备，表示加密文件系统的解密内容，并展示如何安全地挂载文件系统结构，以便使用常规工具浏览。三个目标是获取加密信息、创建一个可以用取证工具访问的设备，并安全地挂载文件系统以供常规浏览。

第一步需要获取 LUKS 加密分区的字节偏移量。扇区偏移量可以通过 Sleuth Kit 的`mmls`命令获取映像文件的偏移信息。字节偏移量是扇区偏移量与扇区大小的乘积，使用简单的 Bash 数学扩展计算得到 1048576：

```
# mmls luks.raw
DOS Partition Table
Offset Sector: 0
Units are in 512-byte sectors

     Slot    Start        End          Length       Description
00:  Meta    0000000000   0000000000   0000000001   Primary Table (#0)
01:  -----   0000000000   0000002047   0000002048   Unallocated
02:  00:00   0000002048   0058626287   0058624240   Linux (0x83)
# echo $((2048*512))
1048576
```

您可以使用字节偏移量通过`losetup`命令创建一个加密分区的 loop 设备，如下所示：

```
# losetup --read-only --find --show -o 1048576 luks.raw
/dev/loop0
```

LUKS 加密分区现在作为块设备可以访问，cryptsetup 工具可以使用它。您可以使用 cryptsetup 的`luksDump`命令查找有关加密分区的信息：

```
# cryptsetup luksDump /dev/loop0
LUKS header information for /dev/loop0

Version:        1
Cipher name:    aes
Cipher mode:    xts-plain64
Hash spec:      sha1
Payload offset: 4096
MK bits:        256
MK digest:      8b 88 36 1e d1 a4 c9 04 0d 3f fd ba 0f be d8 4c 9b 96 fb 86
MK salt:        14 0f 0d fa 7b c3 a2 41 19 d4 6a e4 8a 16 fe 72
                88 78 a2 18 7b 0f 74 8e 26 6d 94 23 3d 11 2e aa
MK iterations:  172000
UUID:           10dae7db-f992-4ce4-89cb-61d126223f05

Key Slot 0: ENABLED
        Iterations:             680850
        Salt:                   8a 39 90 e1 f9 b6 59 e1 a6 73 30 ea 73 d6 98 5a
                                e1 d3 b6 94 a0 73 36 f7 00 68 a2 19 3f 09 62 b8
        Key material offset:    8
        AF stripes:             4000
Key Slot 1: DISABLED
Key Slot 2: DISABLED
Key Slot 3: DISABLED
Key Slot 4: DISABLED
Key Slot 5: DISABLED
Key Slot 6: DISABLED
Key Slot 7: DISABLED
```

从取证角度来看，密钥槽可能会引起关注。一个 LUKS 卷最多可以有八个密钥，意味着有可能有八个不同的密码可以尝试恢复。

使用 LUKS 加密文件系统的密码后，您可以使用 cryptsetup 的`open`命令在 loop0 设备上创建一个映射设备。该设备提供了加密映像的解密表示。在此示例中，映射设备命名为*clear*：

```
# cryptsetup -v --readonly open /dev/loop0 clear
Enter passphrase for /hyb/luks/luks.raw:
Key slot 0 unlocked.
Command successful.
```

使用`--readonly`标志打开加密的 loop 设备。还可以提供详细（`-v`）标志，以提供有关解密密钥成功的更多信息。输入成功的密钥后，将在*/dev/mapper*目录中出现一个新的（解密的）分区设备，并可以使用标准的取证工具进行操作。例如，您可以运行 Sleuth Kit 的 fsstat 工具：

```
# fsstat /dev/mapper/clear
FILE SYSTEM INFORMATION
--------------------------------------------
File System Type: Ext4
Volume Name: My Secrets
Volume ID: ba673056efcc5785f046654c00943860
...
```

您还可以将该分区设备挂载到本地机器上进行常规浏览：

```
# mkdir clear
# mount --read-only /dev/mapper/clear clear
# ls clear
lost+found/  the plan.txt
```

一旦检查工作完成，可以进行清理过程。每个步骤都按逆向顺序进行：

```
# umount clear
# rmdir clear
# cryptsetup close clear
# losetup --detach /dev/loop0
```

请注意，这是一个简化的示例，展示了单个非启动数据磁盘上的单个分区。带有可启动操作系统的 LUKS 加密磁盘可能会有一个额外的逻辑卷管理器（LVM）层。这类磁盘可能会在*/dev/mapper*目录中显示其他设备（如 root、swap 等）。您可以单独访问或挂载这些设备。在清理过程中，您需要在关闭 LVM 设备之前，使用 dmsetup 删除分区设备。

为了简便起见，本节所示的步骤是以 root 用户身份执行的。要以非 root 用户运行示例，`losetup`、`cryptsetup`、`mount`和`umount`命令需要使用`sudo`执行，任何访问*/dev/mapper*分区设备的工具也是如此。根据挂载的文件系统，可能需要其他用户选项（例如`uid=holmes`）。

使用纯 dm-crypt 和 loop-AES 加密的镜像也可以使用 cryptsetup 工具解密。这些解密过程与前面的 LUKS 示例类似。cryptsetup 的`open`命令需要通过`--type`标志指定`plain`或`loopaes`。例如：

```
# cryptsetup -v --readonly open --type plain /dev/loop0 clear
Enter passphrase:
Command successful.
```

使用`--type loopaes`也需要一个密钥文件。指定`--type luks`也是可能的，但不必要，因为它是默认选项。

您可以在*[`gitlab.com/cryptsetup/cryptsetup/wikis/home/`](https://gitlab.com/cryptsetup/cryptsetup/wikis/home/)*上找到更多关于 cryptsetup 和 LUKS 的信息。您还可以在*[`github.com/t-d-k/librecrypt/`](https://github.com/t-d-k/librecrypt/)*找到兼容的 Windows 实现。

#### *TrueCrypt 和 VeraCrypt*

在 TrueCrypt 停止开发后，出现了多个分支。目前主流的分支是 VeraCrypt。它提供了向后兼容性以及新的扩展功能。

我将提供的 VeraCrypt 示例有两个：一个是普通加密容器，另一个是隐藏容器。我使用了标准的 VeraCrypt 命令行版本，并结合常用工具使这些容器可供进一步分析。

第一个示例展示了一个简单的加密 TrueCrypt 或 VeraCrypt 容器文件。`--file-system=none`标志非常重要，因为它防止 VeraCrypt 挂载任何文件系统：

```
$ veracrypt --mount-options=readonly --filesystem=none secrets.tc
Enter password for /exam/secrets.tc:
Enter PIM for /exam/secrets.tc:
Enter keyfile [none]:
```

使用`-l`标志，您可以按插槽编号列出主机系统上所有已解密的容器。插槽编号是后续命令中使用的重要标识符。在此示例中，插槽编号为`1`，并使用了熟悉的`/dev/mapper/*`目录：

```
$ veracrypt -l
1: /exam/secrets.tc /dev/mapper/veracrypt1 -
```

提供正确的凭据后，您可以通过指定插槽编号请求容器的更多信息，如下所示：

```
$ veracrypt --volume-properties --slot=1
Slot: 1
Volume: /exam/secrets.tc
Virtual Device: /dev/mapper/veracrypt1
Mount Directory:
Size: 2.0 GB
Type: Normal
Read-Only: Yes
Hidden Volume Protected: No
Encryption Algorithm: AES
Primary Key Size: 256 bits
Secondary Key Size (XTS Mode): 256 bits
Block Size: 128 bits
Mode of Operation: XTS
PKCS-5 PRF: HMAC-SHA-512
Volume Format Version: 2
Embedded Backup Header: Yes
```

已创建两个设备。设备*/dev/loop0*作为原始镜像进行了加密（与文件系统上的文件相同）。在卷属性中显示的设备*/dev/mapper/veracrypt1*是解密后的卷，您可以直接使用取证工具对其进行操作。以下是 Sleuth Kit 检查文件系统的示例：

```
$ sudo fls /dev/mapper/veracrypt1
r/r * 4:        photo.jpg
r/r 6:  spy-photo.jpg
v/v 66969091:   $MBR
v/v 66969092:   $FAT1
v/v 66969093:   $FAT2
d/d 66969094:   $OrphanFiles
```

你也可以在本地机器上挂载映射设备，并使用常规工具浏览文件系统，如下所示：

```
$ mkdir clear
$ sudo mount -o ro,uid=holmes /dev/mapper/veracrypt1 clear
$ ls -l clear
total 360
-rwxr-x--- 1 holmes root 366592 Jan 21 23:41 spy-photo.jpg
```

显然，已删除的文件在用户挂载区不会显示；它们只会在你通过 */dev/mapper/veracrypt1* 设备使用取证工具时才会显示。

清理过程是设置过程的逆过程：

```
$ sudo umount clear
$ rmdir clear
$ veracrypt -d --slot=1
```

我将提供的第二个 VeraCrypt 示例演示了如何访问隐藏卷。TrueCrypt 和 VeraCrypt 的一个特点是可以有两个密码，分别揭示两个独立的卷。下面两个命令输出对比了同时使用这两个密码的效果。

在这里，*hidden.raw* 是一个包含隐藏卷的 VeraCrypt 驱动器。提供第一个密码会生成一个功能正常的标准 TrueCrypt 容器，容器里有文件，占用了整个 1GB 的驱动器容量，并显示 `Type: Normal`：

```
$ ls -l
total 3098104
-rw-r----- 1 holmes holmes 1024966656 Jan 22 00:07 hidden.raw
...
$ veracrypt --mount-options=readonly --filesystem=none hidden.raw
Enter password for /exam/hidden.raw: [XXXXXXXXXXX]
...
$ veracrypt --volume-properties --slot=1
Slot: 1
Volume: /exam/hidden.raw
Virtual Device: /dev/mapper/veracrypt1
Mount Directory:
Size: 977 MB
Type: Normal
Read-Only: Yes
...
$ sudo fls /dev/mapper/veracrypt1
...
r/r 20: fake secrets.pdf
...
```

如果卷被卸载后再使用隐藏卷的密码重新挂载，你会看到完全不同的一组文件。挂载卷所需的时间也不同。以之前示例中的容器为例，解锁它需要 3.5 秒，而解锁同一文件中的隐藏容器需要 29 秒。这是因为首先尝试标准卷的解密（使用所有支持的算法），如果失败，才尝试解密隐藏卷。在卷属性中，实际大小现在显示，并且 `Type: Hidden` 被标注，如下所示：

```
$ veracrypt -d --slot=1
$ veracrypt --mount-options=readonly --filesystem=none hidden.raw
Enter password for /exam/hidden.raw: [YYYYYYYYYYY]
...
$ veracrypt --volume-properties --slot=1
Slot: 1
Volume: /exam/hidden.raw
Virtual Device: /dev/mapper/veracrypt1
Mount Directory:
Size: 499 MB
Type: Hidden
Read-Only: Yes
...
$ sudo fls /dev/mapper/veracrypt1
...
r/r 19: the real hidden secrets.pdf
...
```

隐藏卷的映射设备生成一个文件系统，你可以使用取证工具直接分析它。

TrueCrypt 和 VeraCrypt 卷也可以通过较新的 cryptsetup 版本（1.6.7 及以后的版本）进行管理，为你提供类似的挂载功能。

有商业和开源的破解工具可以破解 TrueCrypt/VeraCrypt 容器，但它们的使用超出了本书的范围。

### 总结思考

在本章中，你学会了将获取的映像文件作为块设备进行使用，创建分区设备，并安全地将它们提供给常规文件系统工具使用。你还学会了使用环回设备，并更熟悉了 */dev/mapper* 设备。我展示了启动可疑映像的技巧，并演示了从各种虚拟机格式中访问虚拟机映像的方法。最后，你学会了如何将各种加密文件系统以解密形式提供访问。
