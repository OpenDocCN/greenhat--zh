## 第十四章. 文件系统

在第四章、第五章和第十三章中，我们探讨了文件 I/O，特别关注了常规（即磁盘）文件。在本章及接下来的章节中，我们将详细讨论一系列与文件相关的主题：

+   本章介绍了文件系统。

+   第十五章描述了与文件相关的各种属性，包括时间戳、所有权和权限。

+   第十六章和第十七章探讨了 Linux 2.6 的两个新特性：扩展属性和访问控制列表（ACLs）。扩展属性是一种将任意元数据与文件关联的方法。ACLs 是对传统 UNIX 文件权限模型的扩展。

+   第十八章探讨了目录和链接。

本章的大部分内容涉及文件系统，文件系统是组织文件和目录的集合。我们解释了一系列文件系统的概念，有时以传统的 Linux *ext2*文件系统作为具体示例。我们还简要描述了 Linux 上可用的一些日志文件系统。

本章最后讨论了用于挂载和卸载文件系统的系统调用，以及用于获取已挂载文件系统信息的库函数。

## 设备特殊文件（设备）

本章频繁提到磁盘设备，因此我们首先简要概述一下设备文件的概念。

设备特殊文件对应系统中的设备。在内核中，每种设备类型都有一个相应的设备驱动程序，负责处理该设备的所有 I/O 请求。*设备驱动程序*是内核代码的一部分，实现了一组操作，这些操作通常对应于与关联硬件进行输入输出操作。设备驱动程序提供的 API 是固定的，包括与系统调用*open()*、*close()*、*read()*、*write()*、*mmap()*和*ioctl()*对应的操作。每个设备驱动程序提供一致的接口，隐藏了各个设备操作的差异，从而实现了*I/O 的通用性*（I/O 的通用性）。

一些设备是真实的，例如鼠标、磁盘和磁带驱动器。其他设备是*虚拟的*，意味着没有对应的硬件；相反，内核通过设备驱动程序提供一个抽象设备，其 API 与真实设备相同。

设备可以分为两种类型：

+   *字符设备*按字符逐个处理数据。终端和键盘是字符设备的例子。

+   *块设备*一次处理一个数据块。块的大小取决于设备类型，但通常是 512 字节的倍数。块设备的例子包括磁盘和磁带驱动器。

设备文件出现在文件系统中，就像其他文件一样，通常位于`/dev`目录下。超级用户可以使用*mknod*命令创建设备文件，同样的任务也可以在具有特权的（`CAP_MKNOD`）程序中使用*mknod()*系统调用来完成。

### 注意

我们不会详细描述*mknod()*（“创建文件系统 i-node”）系统调用，因为它的使用非常直接，而且如今唯一需要它的目的就是创建设备文件，这并不是一个常见的应用需求。我们也可以使用*mknod()*来创建 FIFO（FIFO），但在这项任务中，*mkfifo()*函数更为推荐。从历史上看，一些 UNIX 实现也使用*mknod()*来创建目录，但现在这一用法已被*mkdir()*系统调用所取代。然而，一些 UNIX 实现——但不是 Linux——为了向后兼容，仍保留了*mknod()*中的此功能。有关详细信息，请参见*mknod(2)*手册页。

在早期版本的 Linux 中，`/dev`包含系统上所有可能设备的条目，即使这些设备实际上并未连接到系统。这意味着`/dev`可能包含成千上万的未使用条目，从而减慢了需要扫描该目录内容的程序的任务，并且使得无法将目录内容作为发现哪些设备实际存在于系统中的手段。在 Linux 2.6 中，这些问题通过*udev*程序得到解决。*udev*程序依赖于*sysfs*文件系统，它通过挂载在`/sys`下的伪文件系统将关于设备和其他内核对象的信息导出到用户空间。

### 注意

[Kroah-Hartman, 2003]提供了关于*udev*的概述，并概述了它为何被认为优于*devfs*，即 Linux 2.4 解决相同问题的方案。有关*sysfs*文件系统的信息可以在 Linux 2.6 内核源文件`Documentation/filesystems/sysfs.txt`中找到，并在[Mochel, 2005]中提供。

#### 设备 ID

每个设备文件都有一个*主 ID 数字*和一个*次 ID 数字*。主 ID 标识设备的大类，并由内核用于查找该类型设备的适当驱动程序。次 ID 唯一标识大类中的特定设备。设备文件的主 ID 和次 ID 可以通过*ls -l*命令显示。

设备的主 ID 和次 ID 会记录在设备文件的 i-node 中。（我们在第 14.4 节描述了 i-node。）每个设备驱动程序都会注册其与特定主设备 ID 的关联，而这种关联提供了设备特殊文件与设备驱动程序之间的连接。当内核查找设备驱动程序时，设备文件的名称并不重要。

在 Linux 2.4 及更早版本中，系统上的设备总数受到限制，因为设备的主次 ID 各自仅使用 8 位表示。由于主设备 ID 是固定的，并由 Linux 分配名称和编号机构（参见 [`www.lanana.org/`](http://www.lanana.org/)）集中分配，这进一步加剧了这个限制。Linux 2.6 通过使用更多的位来表示主次设备 ID（分别是 12 位和 20 位），缓解了这个限制。

## 磁盘和分区

常规文件和目录通常存储在硬盘设备上。（文件和目录也可以存储在其他设备上，如 CD-ROM、闪存卡和虚拟磁盘，但在本讨论中，我们主要关注硬盘设备。）在接下来的章节中，我们将讨论磁盘是如何组织和分区的。

#### 磁盘驱动器

硬盘驱动器是一个机械设备，由一个或多个以高速旋转的盘片组成（转速大约是每分钟几千转）。磁盘表面上磁性编码的信息通过读写头来检索或修改，读写头会沿着磁盘径向移动。从物理上讲，磁盘表面上的信息位于一组同心圆上，这些圆被称为 *磁道*。磁道本身被划分为若干个 *扇区*，每个扇区由一系列 *物理* 块组成。物理块的大小通常为 512 字节（或其倍数），它们代表了驱动器可以读取或写入的最小信息单位。

尽管现代磁盘非常快速，但读取和写入磁盘上的信息仍然需要相当长的时间。磁盘头首先必须移动到适当的磁道（寻道时间），然后驱动器必须等待直到适当的扇区转动到磁头下方（旋转延迟），最后需要的块必须被传输（传输时间）。完成此操作所需的总时间通常是以毫秒为单位的。相比之下，现代 CPU 在这个时间内能够执行数百万条指令。

#### 磁盘分区

每个磁盘被划分为一个或多个（不重叠的）*分区*。每个分区都被内核视为一个独立的设备，位于 `/dev` 目录下。

### 注意

系统管理员通过使用 *fdisk* 命令来决定磁盘的分区数量、类型和大小。命令 *fdisk -l* 会列出磁盘上的所有分区。Linux 特有的 `/proc/partitions` 文件列出了系统上每个磁盘分区的主次设备编号、大小和名称。

磁盘分区可以存储任何类型的信息，但通常包含以下之一：

+   一个 *文件系统* 用于存储常规文件和目录，如在 文件系统 中所述；

+   作为原始模式设备访问的 *数据区*，如在 绕过缓冲区缓存：直接 I/O 中所描述（一些数据库管理系统使用此技术）；或者

+   一个由内核用于内存管理的 *交换区*。

*mkswap(8)* 命令用于创建交换区。一个特权 (`CAP_SYS_ADMIN`) 进程可以使用 *swapon()* 系统调用通知内核某个磁盘分区将作为交换区使用。*swapoff()* 系统调用则执行相反的功能，告诉内核停止使用磁盘分区作为交换区。这些系统调用在 SUSv3 中并未标准化，但在许多 UNIX 实现中存在。有关更多信息，请参见 *swapon(2)* 和 *swapon(8)* 手册页。

### 注意

Linux 特有的 `/proc/swaps` 文件可以用来显示当前启用的交换区的信息。该信息包括每个交换区的大小以及已使用的部分。

## 文件系统

文件系统是由常规文件和目录组成的有组织的集合。文件系统是通过 *mkfs* 命令创建的。

Linux 的一个优点是它支持各种各样的文件系统，包括以下几种：

+   传统的 *ext2* 文件系统；

+   各种本地 UNIX 文件系统，如 Minix、System V 和 BSD 文件系统；

+   Microsoft 的 FAT、FAT32 和 NTFS 文件系统；

+   ISO 9660 CD-ROM 文件系统；

+   苹果 Macintosh 的 HFS；

+   一系列网络文件系统，包括 Sun 公司广泛使用的 NFS（关于 Linux 上 NFS 的实现信息可在 [`nfs.sourceforge.net/`](http://nfs.sourceforge.net/) 查阅），IBM 和 Microsoft 的 SMB，Novell 的 NCP，以及卡内基梅隆大学开发的 Coda 文件系统；并且

+   一系列日志文件系统，包括 *ext3*、*ext4*、*Reiserfs*、*JFS*、*XFS* 和 *Btrfs*。

当前内核已知的文件系统类型可以在 Linux 特有的 `/proc/filesystems` 文件中查看。

### 注意

Linux 2.6.14 增加了 *Filesystem in Userspace*（FUSE）功能。该机制为内核添加了钩子，允许通过用户空间程序完全实现文件系统，而无需打补丁或重新编译内核。更多详细信息，请参见 [`fuse.sourceforge.net/`](http://fuse.sourceforge.net/)。

#### *ext2* 文件系统

多年来，Linux 上最广泛使用的文件系统是 *ext2*，即 *第二扩展文件系统*，它是原始 Linux 文件系统 *ext* 的继任者。近年来，*ext2* 的使用已经减少，取而代之的是各种日志文件系统。有时，为了描述通用的文件系统概念，可能会通过特定文件系统的实现来说明，在这种情况下，我们将在本章后面的多个地方使用 *ext2* 作为示例。

### 注意

*ext2*文件系统是由 Remy Card 编写的。*ext2*的源代码很小（大约 5000 行 C 语言），并为其他几个文件系统实现提供了模型。*ext2*的主页是 [`e2fsprogs.sourceforge.net/ext2.html`](http://e2fsprogs.sourceforge.net/ext2.html)。这个网站包括一篇很好的概述文章，描述了*ext2*的实现。David Rusling 的在线书籍《*Linux Kernel*》可以在[`www.tldp.org/`](http://www.tldp.org/)上找到，也描述了*ext2*。

#### 文件系统结构

文件系统中分配空间的基本单位是*逻辑*块，它是磁盘设备中一系列连续物理块的倍数，文件系统驻留在该设备上。例如，*ext2*的逻辑块大小为 1024、2048 或 4096 字节。（逻辑块大小作为*mkfs(8)*命令的一个参数来指定，该命令用于构建文件系统。）

### 注意

一个具有特权的（`CAP_SYS_RAWIO`）程序可以使用`FIBMAP` *ioctl()* 操作来确定指定文件块的物理位置。调用的第三个参数是一个值结果整数。在调用之前，该参数应设置为逻辑块号（第一个逻辑块号为 0）；调用之后，它将被设置为存储该逻辑块的起始物理块号。

图 14-1 展示了磁盘分区与文件系统之间的关系，并展示了（通用）文件系统的各个部分。

![磁盘分区和文件系统的布局](img/14-1_FS-file-systems-scale90.png.jpg)图 14-1. 磁盘分区和文件系统的布局

一个文件系统包含以下部分：

+   *引导块*：这是文件系统中的第一个块。引导块不是由文件系统使用的，而是包含用于引导操作系统的信息。尽管操作系统只需要一个引导块，但所有文件系统都有一个引导块（其中大部分未使用）。

+   *超级块*：这是一个单独的块，紧随引导块之后，包含关于文件系统的参数信息，包括：

    +   i-节点表的大小；

    +   文件系统中逻辑块的大小；以及

    +   文件系统在逻辑块中的大小。

    不同的文件系统可以驻留在同一物理设备上，它们的类型和大小可以不同，并且有不同的参数设置（例如，块大小）。这也是将磁盘分割为多个分区的原因之一。

+   *I-节点表*：文件系统中的每个文件或目录在 i-节点表中都有一个唯一的条目。该条目记录关于文件的各种信息。i-节点将在下一节中详细讨论。i-节点表有时也被称为*i-列表*。

+   *数据块*：文件系统中大部分空间用于存储形成文件和目录的数据块，这些文件和目录驻留在文件系统中。

    ### 注意

    在 *ext2* 文件系统的具体情况下，图示比主文中描述的更为复杂。在初始引导块之后，文件系统被分为一组等大小的 *块组*。每个块组包含超级块的副本、有关块组的参数信息，然后是该块组的 i 节点表和数据块。通过尝试将文件的所有块存储在同一个块组中，*ext2* 文件系统旨在减少顺序访问文件时的寻道时间。有关更多信息，请参见 Linux 源代码文件 `Documentation/filesystems/ext2.txt`，作为 *e2fsprogs* 包的一部分的 *dumpe2fs* 程序源代码，以及[Bovet & Cesati, 2005]。

## I 节点

一个文件系统的 i 节点表包含每个文件的一个 i 节点（i 节点是*索引节点*的缩写）。I 节点通过其在 i 节点表中的顺序位置按数字标识。文件的 *i 节点号*（或简称 *i-number*）是 *ls -li* 命令显示的第一个字段。i 节点中维护的信息包括以下内容：

+   文件类型（例如，普通文件、目录、符号链接、字符设备）。

+   文件的所有者（也称为用户 ID 或 UID）。

+   文件的组（也称为组 ID 或 GID）。

+   三类用户的访问权限：*owner*（有时称为 *user*）、*group* 和 *other*（其余的世界）。文件权限提供了更多的细节。

+   三个时间戳：文件最后访问时间（通过 *ls -lu* 显示）、文件最后修改时间（*ls -l* 显示的默认时间）、最后状态更改时间（i 节点信息的最后更改时间，通过 *ls -lc* 显示）。与其他 UNIX 实现类似，值得注意的是，大多数 Linux 文件系统并不记录文件的创建时间。

+   文件的硬链接数量。

+   文件的字节大小。

+   实际分配给文件的块数，以 512 字节的块为单位进行度量。此数字与文件的字节大小之间可能没有简单的对应关系，因为文件可能包含空洞（改变文件偏移量：*lseek()*")），因此需要的分配块数可能少于按字节大小预期的块数。

+   指向文件数据块的指针。

#### *ext2*中的 I 节点和数据块指针

与大多数 UNIX 文件系统类似，*ext2*文件系统并不将文件的数据块连续存储，甚至不按顺序存储（尽管它确实尽量将它们存储在彼此接近的位置）。为了定位文件数据块，内核在 i 节点中维护一组指针。在 *ext2* 文件系统上执行此操作的系统在图 14-2 中展示。

### 注意

去除必须将文件块存储为连续块的需求，使得文件系统能够高效地使用磁盘空间。特别地，它减少了磁盘空间中 *碎片化* 的发生——即存在大量不连续的小块空闲空间，这些空闲空间过小，无法使用。反过来说，我们可以说，高效利用空闲磁盘空间的好处是通过在已填满的磁盘空间中碎片化文件来实现的。

在 *ext2* 文件系统中，每个 i-node 包含 15 个指针。这些指针中的前 12 个（编号为 0 至 11，在图 14-2 中）指向文件系统中文件的前 12 个数据块的位置。接下来的指针是一个 *指向指针块的指针*，它指向文件的第十三个及后续数据块的位置。这个块中的指针数量取决于文件系统的块大小。每个指针需要 4 字节，因此指针的数量可以从 256 个（对于 1024 字节块大小）到 1024 个（对于 4096 字节块大小）。这样可以支持相当大的文件。对于更大的文件，第十四个指针（图中的编号 13）是一个 *双重间接指针* ——它指向一块指针块，而这些指针块指向其他指针块，这些指针块又指向文件的实际数据块。如果需要处理真正庞大的文件，还可以进一步增加间接层次：i-node 中的最后一个指针是一个 *三重间接指针*。

这个看似复杂的系统旨在满足多个要求。首先，它允许 i-node 结构具有固定大小，同时也支持任意大小的文件。此外，它还允许文件系统以非连续的方式存储文件的块，同时又能通过 *lseek()* 随机访问数据；内核只需要计算应该跟随哪个指针。最后，对于大多数系统中占压倒性多数的小文件，这种方案通过 i-node 的直接指针快速访问文件数据块。

![ext2 文件系统中文件块的结构](img/14-2_FS-file-blocks.png.jpg)图 14-2. *ext2* 文件系统中文件块的结构

### 注意

作为一个例子，作者测量了一个包含略多于 150,000 个文件的系统。超过 30% 的文件小于 1000 字节，80% 的文件占用的空间小于 10,000 字节。假设块大小为 1024 字节，所有后者的文件都可以仅通过 12 个直接指针来引用，这些指针可以引用包含总共 12,288 字节的块。使用 4096 字节的块大小时，这个限制上升到 49,152 字节（系统中 95% 的文件都小于这个限制）。

这种设计还允许巨大的文件大小；对于 4096 字节的块大小，理论上的最大文件大小略大于 1024*1024*1024*4096，约为 4TB（4096 GB）。（我们说*略大*是因为直接、间接和双重间接指针指向的块。与三重间接指针能够指向的范围相比，这些块可以忽略不计。）

这种设计带来的另一个好处是文件可以有空洞，如第 4.7 节所述。文件系统可以通过在 i 节点和间接指针块中标记（值为 0）适当的指针，来指示它们不指向实际的磁盘块，而不是为文件中的空洞分配空字节块。

## 虚拟文件系统（VFS）

在 Linux 上，每个文件系统的实现细节都有所不同。这些差异包括例如文件的块是如何分配的，以及目录是如何组织的。如果每个与文件交互的程序都需要理解每个文件系统的具体细节，那么编写能够支持所有不同文件系统的程序几乎是不可能的。*虚拟文件系统*（VFS，有时也称为*虚拟文件交换机*）是一个内核特性，它通过为文件系统操作创建一个抽象层来解决这个问题（见图 14-3）。VFS 的理念很简单：

+   VFS 定义了一个通用的文件系统操作接口。所有与文件交互的程序都通过此通用接口指定它们的操作。

+   每个文件系统为 VFS 接口提供一个实现。

在这种方案下，程序只需要理解 VFS 接口，忽略各个文件系统实现的细节。

VFS 接口包含了与操作文件系统和目录的所有常用系统调用对应的操作，例如*open()*、*read()*、*write()*、*lseek()*、*close()*、*truncate()*、*stat()*、*mount()*、*umount()*、*mmap()*、*mkdir()*、*link()*、*unlink()*、*symlink()*和*rename()*。

VFS 抽象层与传统的 UNIX 文件系统模型紧密对应。自然地，一些文件系统——特别是非 UNIX 文件系统——不支持所有 VFS 操作（例如，Microsoft 的 VFAT 不支持通过*symlink()*创建的符号链接的概念）。在这种情况下，底层文件系统会将一个错误代码返回给 VFS 层，指示不支持该操作，而 VFS 又将此错误代码返回给应用程序。

![虚拟文件系统](img/14-3_FS-VFS-scale90.png.jpg)图 14-3. 虚拟文件系统

## 日志文件系统

*ext2* 文件系统是传统 UNIX 文件系统的一个典型例子，并且存在这种文件系统的经典局限性：在系统崩溃后，必须在重启时执行文件系统一致性检查（*fsck*），以确保文件系统的完整性。这是必要的，因为在系统崩溃时，文件更新可能只完成了一部分，而文件系统元数据（目录条目、i-node 信息和文件数据块指针）可能处于不一致状态，因此如果不修复这些不一致，文件系统可能会进一步损坏。文件系统一致性检查确保文件系统元数据的一致性。如果可能，进行修复；否则，丢弃无法恢复的信息（可能包括文件数据）。

问题在于，一致性检查需要检查整个文件系统。在小型文件系统中，这可能需要几秒钟到几分钟的时间。而在大型文件系统中，这可能需要几个小时，这对必须保持高可用性的系统（例如，网络服务器）来说是一个严重问题。

日志文件系统消除了系统崩溃后进行文件系统一致性检查的需要。日志文件系统在实际执行更新之前，会将所有元数据更新记录（日志）到一个特殊的磁盘日志文件中。这些更新以相关元数据更新的组（*事务*）形式进行记录。如果在事务执行过程中发生系统崩溃，系统重启时，日志可以用于快速重做任何未完成的更新，并将文件系统恢复到一致状态。（借用数据库术语，我们可以说，日志文件系统确保文件元数据事务始终以完整单元进行 *提交*。）即使是非常大的日志文件系统，在系统崩溃后通常也能在几秒钟内恢复，因此对需要高可用性的系统非常有吸引力。

日志的最显著缺点是，它会增加文件更新的时间，尽管良好的设计可以使这种开销保持较低。

### 注释

一些日志文件系统仅确保文件元数据的一致性。因为它们不记录文件数据，所以在系统崩溃时数据可能仍然会丢失。*ext3*、*ext4* 和 *Reiserfs* 文件系统提供了记录数据更新的选项，但根据工作负载的不同，这可能会导致较低的文件 I/O 性能。

可用于 Linux 的日志文件系统包括以下几种：

+   *Reiserfs* 是第一个被集成到内核中的日志文件系统（在版本 2.4.1 中）。*Reiserfs* 提供了一种名为 *尾部打包*（或 *尾部合并*）的功能：小文件（以及较大文件的最后一个片段）被打包到与文件元数据相同的磁盘块中。由于许多系统（以及一些应用程序）会创建大量小文件，这可以节省大量磁盘空间。

+   *ext3* 文件系统是一个为 *ext2* 添加日志记录功能的项目，且其对性能的影响最小。*ext2* 到 *ext3* 的迁移路径非常简单（不需要备份和恢复），也可以进行反向迁移。*ext3* 文件系统被集成到 2.4.15 版本的内核中。

+   *JFS* 是在 IBM 开发的，它被集成到 2.4.20 内核中。

+   *XFS* ([`oss.sgi.com/projects/xfs/`](http://oss.sgi.com/projects/xfs/)) 最初由硅谷图形公司（SGI）在 1990 年代初期为 Irix（其专有 UNIX 实现）开发。2001 年，*XFS* 被移植到 Linux 并作为自由软件项目发布。*XFS* 被集成到 2.4.24 内核中。

支持各种文件系统是通过在配置内核时，在*文件系统*菜单下设置内核选项来启用的。

在写作时，正在开发另外两个提供日志记录以及一系列其他高级特性的文件系统：

+   *ext4* 文件系统 ([`ext4.wiki.kernel.org/`](http://ext4.wiki.kernel.org/)) 是 *ext3* 的继任者。最初的实现部分被添加到 2.6.19 版本的内核中，之后的内核版本增加了各种特性。对于 *ext4*，计划中的（或已实现的）特性包括扩展（保留连续的存储块）以及其他旨在减少文件碎片、在线文件系统碎片整理、更快速的文件系统检查和对纳秒时间戳支持的分配特性。

+   *Btrfs*（B-tree FS，通常发音为“butter FS”；[`btrfs.wiki.kernel.org/`](http://btrfs.wiki.kernel.org/)）是一个从零开始设计的新型文件系统，提供一系列现代特性，包括扩展、可写快照（提供等同于元数据和数据日志记录的功能）、数据和元数据的校验和、在线文件系统检查、在线文件系统碎片整理、小文件的空间高效打包，以及空间高效的索引目录。它被集成到 2.6.29 版本的内核中。

## 单一目录层次结构与挂载点

在 Linux 系统上，和其他 UNIX 系统一样，所有文件都位于单一的目录树下。该树的根目录是根目录`/`（斜杠）。其他文件系统被*挂载*在根目录下，并在整个层级结构中作为子树显示。超级用户使用如下命令来挂载文件系统：

```
$ `mount` ``*`device directory`*``
```

此命令将指定的*设备*上的文件系统附加到指定的*目录*—文件系统的*挂载点*。可以改变文件系统挂载的位置—通过使用*umount*命令卸载文件系统，然后在不同的点重新挂载。

### 注意

从 Linux 2.4.19 及以后版本开始，事情变得更加复杂。内核现在支持每个进程的*挂载命名空间*。这意味着每个进程可能有自己的一组文件系统挂载点，因此可能会看到与其他进程不同的单一目录层次结构。当我们在示例程序中描述`CLONE_NEWNS`标志时，我们会进一步解释这一点。

要列出当前挂载的文件系统，我们可以使用不带任何参数的*mount*命令，如以下示例所示（输出已略微缩减）：

```
$ `mount`
/dev/sda6 on / type ext4 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,mode=0620,gid=5)
/dev/sda8 on /home type ext3 (rw,acl,user_xattr)
/dev/sda1 on /windows/C type vfat (rw,noexec,nosuid,nodev)
/dev/sda9 on /home/mtk/test type reiserfs (rw)
```

图 14-4 显示了执行上述*mount*命令的系统的部分目录和文件结构。该图显示了挂载点如何映射到目录层次结构。

![显示文件系统挂载点的示例目录层次结构](img/14-4_FS-mount.png.jpg)图 14-4. 显示文件系统挂载点的示例目录层次结构

## 挂载和卸载文件系统

*mount()*和*umount()*系统调用允许具有特权（`CAP_SYS_ADMIN`）的进程挂载和卸载文件系统。大多数 UNIX 实现提供这些系统调用的版本。然而，它们并没有被 SUSv3 标准化，而且它们的操作在不同的 UNIX 实现和不同的文件系统之间有所不同。

在查看这些系统调用之前，了解包含当前挂载或可挂载文件系统信息的三个文件是很有用的：

+   可以从 Linux 特定的`/proc/mounts`虚拟文件中读取当前挂载的文件系统列表。`/proc/mounts`是对内核数据结构的接口，因此它始终包含关于挂载文件系统的准确信息。

    ### 注意

    随着之前提到的每个进程挂载命名空间功能的出现，每个进程现在都有一个`/proc/`*PID*`/mounts`文件，列出了构成其挂载命名空间的挂载点，而`/proc/mounts`只是指向`/proc/self/mounts`的符号链接。

+   *mount(8)*和*umount(8)*命令会自动维护文件`/etc/mtab`，该文件包含类似于`/proc/mounts`中的信息，但稍微更详细。特别是，`/etc/mtab`包括传递给*mount(8)*的特定于文件系统的选项，而这些选项在`/proc/mounts`中没有显示。然而，由于*mount()*和*umount()*系统调用不会更新`/etc/mtab`，因此如果某个挂载或卸载设备的应用程序未能更新该文件，则该文件可能不准确。

+   `/etc/fstab`文件由系统管理员手动维护，包含系统上所有可用文件系统的描述，并由*mount(8)*、*umount(8)*和*fsck(8)*命令使用。

`/proc/mounts`、`/etc/mtab`和`/etc/fstab`文件共享一个共同的格式，具体描述请参见*fstab(5)*手册页。以下是来自`/proc/mounts`文件的一行示例：

```
/dev/sda9 /boot ext3 rw 0 0
```

这一行包含六个字段：

1.  已挂载设备的名称。

1.  设备的挂载点。

1.  文件系统类型。

1.  挂载标志。在上面的例子中，*rw* 表示文件系统以读写方式挂载。

1.  一个用于控制 *dump(8)* 文件系统备份操作的数字。此字段和下一个字段仅在 `/etc/fstab` 文件中使用；对于 `/proc/mounts` 和 `/etc/mtab`，这些字段始终为 0。

1.  一个用于控制 *fsck(8)* 在系统启动时检查文件系统顺序的数字。

*getfsent(3)* 和 *getmntent(3)* 手册页面记录了可以用来从这些文件中读取记录的函数。

### 挂载文件系统：*mount()*

*mount()* 系统调用将包含在由 *source* 指定的设备上的文件系统挂载到由 *target* 指定的目录（即 *挂载点*）下。

```
#include <sys/mount.h>

int `mount`(const char **source*, const char **target*, const char **fstype*,
          unsigned long *mountflags*, const void **data*);
```

### 注意

成功时返回 0，出错时返回 -1。

*source* 和 *target* 这两个名称用于前两个参数，因为 *mount()* 除了可以在目录下挂载磁盘文件系统外，还可以执行其他任务。

*fstype* 参数是一个字符串，标识设备上包含的文件系统类型，如 *ext4* 或 *btrfs*。

*mountflags* 参数是通过 OR 运算（`|`）将零个或多个标志结合成的位掩码，这些标志在 表 14-1 中列出，下面有更详细的描述。

最后的 *mount()* 参数，*data*，是一个指向信息缓冲区的指针，缓冲区内容的解释依赖于文件系统。对于大多数文件系统类型，此参数是一个由逗号分隔的选项设置组成的字符串。可以在 *mount(8)* 手册页面中找到这些选项的完整列表（如果文件系统没有在 *mount(8)* 中描述，则请查阅相关文件系统的文档）。

表 14-1. *mount()* 的 `mountflags` 值

| 标志 | 目的 |
| --- | --- |
| `MS_BIND` | 创建绑定挂载（自 Linux 2.4 起） |
| `MS_DIRSYNC` | 使目录更新同步（自 Linux 2.6 起） |
| `MS_MANDLOCK` | 允许强制锁定文件 |
| `MS_MOVE` | 原子地将挂载点移动到新位置 |
| `MS_NOATIME` | 不更新文件的最后访问时间 |
| `MS_NODEV` | 不允许访问设备 |
| `MS_NODIRATIME` | 不更新目录的最后访问时间 |
| `MS_NOEXEC` | 不允许执行程序 |
| `MS_NOSUID` | 禁用设置用户 ID 和设置组 ID 的程序 |
| `MS_RDONLY` | 只读挂载；不能创建或修改文件 |
| `MS_REC` | 递归挂载（自 Linux 2.6.20 起） |
| `MS_RELATIME` | 只有当最后访问时间晚于最后修改时间或最后状态变化时间时才更新最后访问时间（自 Linux 2.4.11 起） |
| `MS_REMOUNT` | 使用新的 *mountflags* 和 *data* 重新挂载 |
| `MS_STRICTATIME` | 始终更新最后访问时间（自 Linux 2.6.30 起） |
| `MS_SYNCHRONOUS` | 使所有文件和目录更新同步 |

*mountflags* 参数是一个位掩码，包含修改 *mount()* 操作的标志。可以在 *mountflags* 中指定以下一个或多个标志：

`MS_BIND`（自 Linux 2.4 起）

创建绑定挂载。我们在绑定挂载中描述了此功能。如果指定此标志，则 *fstype*、*mountflags* 和 *data* 参数将被忽略。

`MS_DIRSYNC`（自 Linux 2.6 起）

使目录更新同步。这类似于 *open()* 的 `O_SYNC` 标志（控制内核文件 I/O 缓存），但仅适用于目录更新。下面描述的 `MS_SYNCHRONOUS` 标志提供了 `MS_DIRSYNC` 的超集功能，确保文件和目录更新都同步执行。`MS_DIRSYNC` 标志允许应用程序确保目录更新（例如 *open(pathname, O_CREAT)*、*rename()*、*link()*、*unlink()*、*symlink()* 和 *mkdir()*）是同步的，而不需要同步所有文件更新。`FS_DIRSYNC_FL` 标志（I 节点标志（*ext2* 扩展文件属性））与 `MS_DIRSYNC` 的作用类似，但 `FS_DIRSYNC_FL` 可以应用于单个目录。此外，在 Linux 上，调用 *fsync()* 以同步引用目录的文件描述符，可以实现按目录同步更新。（这种 Linux 特有的 *fsync()* 行为未在 SUSv3 中规定。）

`MS_MANDLOCK`

允许在该文件系统中的文件上强制记录锁定。我们在第五十五章中描述了记录锁定。

`MS_MOVE`

原子性地将由 *source* 指定的现有挂载点移动到由 *target* 指定的新位置。这对应于 *mount(8)* 的 *—move* 选项。这相当于先卸载子树，然后再在不同位置重新挂载，唯一不同的是在此过程中子树从未被卸载。*source* 参数应为先前 *mount()* 调用中指定的 *target* 字符串。当指定此标志时，*fstype*、*mountflags* 和 *data* 参数将被忽略。

`MS_NOATIME`

不更新此文件系统中文件的最后访问时间。此标志的目的，与下面描述的 `MS_NODIRATIME` 标志一样，是为了消除每次访问文件时更新文件 i-node 所需的额外磁盘访问。在某些应用中，维护此时间戳并不关键，避免更新它可以显著提高性能。`MS_NOATIME` 标志与 `FS_NOATIME_FL` 标志（I-node 标志 (*ext2* 扩展文件属性)"))的作用类似，区别在于 `FS_NOATIME_FL` 可以应用于单个文件。Linux 还通过 `O_NOATIME` *open()* 标志提供了类似的功能，这个标志为单个打开的文件选择此行为（通过 *open()* 返回的文件描述符号")）。

`MS_NODEV`

不允许访问此文件系统上的块设备和字符设备。这是一个安全特性，旨在防止用户执行如插入含有设备特殊文件的可移动磁盘等操作，从而可能导致对系统的任意访问。

`MS_NODIRATIME`

不更新此文件系统中文件夹的最后访问时间。（此标志提供了 `MS_NOATIME` 的子集功能，`MS_NOATIME` 会阻止更新所有文件类型的最后访问时间。）

`MS_NOEXEC`

禁止从此文件系统执行程序（或脚本）。如果文件系统包含非 Linux 可执行文件，这个选项会很有用。

`MS_NOSUID`

禁止在此文件系统上运行 set-user-ID 和 set-group-ID 程序。这是一个安全特性，用于防止用户从可移动设备上运行 set-user-ID 和 set-group-ID 程序。

`MS_RDONLY`

以只读方式挂载文件系统，确保不能创建新文件或修改现有文件。

`MS_REC`（自 Linux 2.4.11 起）

此标志与其他标志一起使用（例如 `MS_BIND`），以递归地将挂载操作应用于子树中的所有挂载点。

`MS_RELATIME`（自 Linux 2.6.20 起）

仅当当前的最后访问时间戳小于或等于最后修改时间戳或最后状态更改时间戳时，才更新此文件系统中文件的最后访问时间戳。这提供了与 `MS_NOATIME` 类似的一些性能优势，但对于需要知道文件自上次更新以来是否被读取的程序很有用。自 Linux 2.6.30 起，`MS_RELATIME` 提供的行为为默认行为（除非指定了 `MS_NOATIME` 标志），而 `MS_STRICTATIME` 标志是获取经典行为所必需的。此外，自 Linux 2.6.30 起，如果最后访问时间戳当前的值距离现在超过 24 小时，即使当前值比最后修改和最后状态更改时间戳更新，也会始终更新该时间戳。（这对于某些系统程序很有用，这些程序监控目录，以查看文件是否最近被访问过。）

`MS_REMOUNT`

修改已挂载的文件系统的 *mountflags* 和 *data*（例如，将只读文件系统改为可写）。使用此标志时，*source* 和 *target* 参数应与原始 *mount()* 调用中的相同，*fstype* 参数会被忽略。此标志避免了卸载和重新挂载磁盘的需求，在某些情况下可能无法执行。例如，如果任何进程在文件系统中打开了文件，或其当前工作目录位于该文件系统内，我们无法卸载该文件系统（根文件系统始终如此）。另一个需要使用 `MS_REMOUNT` 的例子是 *tmpfs*（基于内存的）文件系统（虚拟内存文件系统：*tmpfs*），它们无法在不丢失内容的情况下卸载。并非所有 *mountflags* 都是可修改的；详细信息请参见 *mount(2)* 手册页。

`MS_STRICTATIME`（自 Linux 2.6.30 起）

当访问此文件系统上的文件时，总是更新最后访问时间戳。这是在 Linux 2.6.30 之前的默认行为。如果指定了 `MS_STRICTATIME`，则如果在*mountflags*中同时指定了 `MS_NOATIME` 和 `MS_RELATIME`，则这两个选项将被忽略。

`MS_SYNCHRONOUS`

使此文件系统上的所有文件和目录更新同步进行。（对于文件来说，这就像文件总是使用 *open()* `O_SYNC` 标志打开一样。）

### 注意

从内核 2.6.15 开始，Linux 提供了四个新的挂载标志以支持*共享子树*的概念。这些新标志是`MS_PRIVATE`、`MS_SHARED`、`MS_SLAVE`和`MS_UNBINDABLE`。（这些标志可以与`MS_REC`结合使用，以将它们的效果传播到挂载子树下的所有子挂载。）共享子树是为某些高级文件系统特性设计的，如每进程挂载命名空间（请参见示例程序中`CLONE_NEWNS`的描述），以及*用户空间文件系统*（FUSE）设施。共享子树设施允许文件系统挂载在挂载命名空间之间以受控方式传播。有关共享子树的详细信息，请参见内核源代码文件`Documentation/filesystems/sharedsubtree.txt`和[Viro & Pai, 2006]。

#### 示例程序

示例 14-1")中的程序提供了一个命令级接口来调用*mount(2)*系统调用。实际上，它是*mount(8)*命令的粗略版本。以下 shell 会话日志演示了该程序的使用。我们首先创建一个目录用作挂载点，并挂载文件系统：

```
$ `su`                                    *Need privilege to mount a file system*
Password:
# `mkdir /testfs`
# `./t_mount -t ext2 -o bsdgroups /dev/sda12 /testfs`
# `cat /proc/mounts | grep sda12`         *Verify the setup*
/dev/sda12 /testfs ext3 rw 0 0          *Doesn't show bsdgroups*
# `grep sda12 /etc/mtab`
```

我们发现前面的*grep*命令没有输出，因为我们的程序没有更新`/etc/mtab`。我们继续，重新挂载文件系统为只读：

```
# `./t_mount -f Rr /dev/sda12 /testfs`
# `cat /proc/mounts | grep sda12`         *Verify change*
/dev/sda12 /testfs ext3 ro 0 0
```

行中显示的字符串*ro*来自`/proc/mounts`，表示这是一个只读挂载。

最后，我们将挂载点移动到目录层次结构中的新位置：

```
# `mkdir /demo`
# `./t_mount -f m /testfs /demo`
# `cat /proc/mounts | grep sda12`         *Verify change*
/dev/sda12 /demo ext3 ro 0
```

示例 14-1. 使用*mount()*

```
`filesys/t_mount.c`
#include <sys/mount.h>
#include "tlpi_hdr.h"

static void
usageError(const char *progName, const char *msg)
{
    if (msg != NULL)
        fprintf(stderr, "%s", msg);

    fprintf(stderr, "Usage: %s [options] source target\n\n", progName);
    fprintf(stderr, "Available options:\n");
#define fpe(str) fprintf(stderr, "    " str)    /* Shorter! */
    fpe("-t fstype        [e.g., 'ext2' or 'reiserfs']\n");
    fpe("-o data          [file system-dependent options,\n");
    fpe("                 e.g., 'bsdgroups' for ext2]\n");
    fpe("-f mountflags    can include any of:\n");
#define fpe2(str) fprintf(stderr, "            " str)
    fpe2("b - MS_BIND         create a bind mount\n");
    fpe2("d - MS_DIRSYNC      synchronous directory updates\n");
    fpe2("l - MS_MANDLOCK     permit mandatory locking\n");
    fpe2("m - MS_MOVE         atomically move subtree\n");
    fpe2("A - MS_NOATIME      don't update atime (last access time)\n");
    fpe2("V - MS_NODEV        don't permit device access\n");
    fpe2("D - MS_NODIRATIME   don't update atime on directories\n");
    fpe2("E - MS_NOEXEC       don't allow executables\n");
    fpe2("S - MS_NOSUID       disable set-user/group-ID programs\n");
    fpe2("r - MS_RDONLY       read-only mount\n");
    fpe2("c - MS_REC          recursive mount\n");
    fpe2("R - MS_REMOUNT      remount\n");
    fpe2("s - MS_SYNCHRONOUS  make writes synchronous\n");
    exit(EXIT_FAILURE);
}

int
main(int argc, char *argv[])
{
    unsigned long flags;
    char *data, *fstype;
    int j, opt;

    flags = 0;
    data = NULL;
    fstype = NULL;

    while ((opt = getopt(argc, argv, "o:t:f:")) != -1) {
        switch (opt) {
        case 'o':
            data = optarg;
            break;

        case 't':
            fstype = optarg;
            break;

        case 'f':
            for (j = 0; j < strlen(optarg); j++) {
                switch (optarg[j]) {
                case 'b': flags |= MS_BIND;             break;
                case 'd': flags |= MS_DIRSYNC;          break;
                case 'l': flags |= MS_MANDLOCK;         break;
                case 'm': flags |= MS_MOVE;             break;
                case 'A': flags |= MS_NOATIME;          break;
                case 'V': flags |= MS_NODEV;            break;
                case 'D': flags |= MS_NODIRATIME;       break;
                case 'E': flags |= MS_NOEXEC;           break;
                case 'S': flags |= MS_NOSUID;           break;
                case 'r': flags |= MS_RDONLY;           break;
                case 'c': flags |= MS_REC;              break;
                case 'R': flags |= MS_REMOUNT;          break;
                case 's': flags |= MS_SYNCHRONOUS;      break;
                default:  usageError(argv[0], NULL);
                }
            }
            break;

        default:
            usageError(argv[0], NULL);
        }
    }

    if (argc != optind + 2)
        usageError(argv[0], "Wrong number of arguments\n");

    if (mount(argv[optind], argv[optind + 1], fstype, flags, data) == -1)
        errExit("mount");

    exit(EXIT_SUCCESS);
}
      `filesys/t_mount.c`
```

### 卸载文件系统：*umount()* 和 *umount2()*

*umount()*系统调用卸载已挂载的文件系统。

```
#include <sys/mount.h>

int `umount`(const char **target*);
```

### 注意

成功时返回 0，出错时返回-1

*target*参数指定要卸载的文件系统的挂载点。

### 注意

在 Linux 2.2 及之前版本中，文件系统可以通过两种方式识别：通过挂载点或通过包含文件系统的设备名称。从内核 2.4 开始，Linux 不再允许后一种方式，因为一个文件系统现在可以挂载在多个位置，因此为*target*指定文件系统会变得模糊不清。我们在在多个挂载点挂载文件系统中对此进行了更详细的解释。

无法卸载*忙碌*的文件系统；也就是说，如果文件系统上有打开的文件，或者某个进程的当前工作目录位于该文件系统中的某个位置。对忙碌的文件系统调用*umount()*会产生`EBUSY`错误。

*umount2()*系统调用是*umount()*的扩展版本。它通过*flags*参数提供对卸载操作的更精细控制。

```
#include <sys/mount.h>

int `umount2`(const char **target*, int *flags*);
```

### 注意

成功时返回 0，出错时返回-1

该*flags*位掩码参数由以下值的零个或多个通过 OR 操作组合而成：

`MNT_DETACH`（自 Linux 2.4.11 起）

执行*懒惰*卸载。挂载点会被标记，确保没有进程能访问它，但已经在使用该挂载点的进程可以继续使用。文件系统实际上会在所有进程停止使用该挂载点时卸载。

`MNT_EXPIRE`（自 Linux 2.6.8 起）

将挂载点标记为*过期*。如果在初始的*umount2()*调用中指定了此标志，并且挂载点没有被占用，则该调用会失败并显示错误`EAGAIN`，但挂载点会被标记为过期。（如果挂载点被占用，则调用会失败并显示错误`EBUSY`，挂载点不会被标记为过期。）只要没有进程随后使用该挂载点，挂载点就会保持过期状态。第二次指定`MNT_EXPIRE`的*umount2()*调用将卸载已过期的挂载点。这提供了一种卸载一段时间未使用的文件系统的机制。此标志不能与`MNT_DETACH`或`MNT_FORCE`一起使用。

`MNT_FORCE`

即使设备正在使用，也强制卸载（仅限 NFS 挂载）。使用此选项可能会导致数据丢失。

`UMOUNT_NOFOLLOW`（自 Linux 2.6.34 起）

如果*target*是符号链接，则不要取消引用它。此标志专为某些允许无权限用户执行卸载操作的 set-user-ID-*root* 程序设计，以避免在*target*是符号链接且该链接被更改为指向其他位置时可能发生的安全问题。

## 高级挂载功能

现在我们来看一些可以在挂载文件系统时使用的更高级的功能。我们通过使用*mount(8)*命令演示了大部分这些功能的使用。通过调用*mount(2)*，程序也可以实现相同的效果。

### 在多个挂载点挂载文件系统

在 2.4 版本之前的内核中，一个文件系统只能挂载到一个挂载点。从 2.4 内核开始，文件系统可以在文件系统内的多个位置挂载。由于每个挂载点显示的是相同的子树，通过一个挂载点所做的更改可以通过其他挂载点看到，正如以下 shell 会话所示：

```
$ `su`                                  *Privilege is required to use mount(8)*
Password:
# `mkdir /testfs`                       *Create two directories for mount points*
# `mkdir /demo`
# `mount /dev/sda12 /testfs`            *Mount file system at one mount point*
# `mount /dev/sda12 /demo`              *Mount file system at second mount point*
# `mount | grep sda12`                  *Verify the setup*
/dev/sda12 on /testfs type ext3 (rw)
/dev/sda12 on /demo type ext3 (rw)
# `touch /testfs/myfile`                *Make a change via first mount point*
# `ls /demo`                            *View files at second mount point*
lost+found  myfile
```

*ls*命令的输出显示，通过第一个挂载点（`/testfs`）所做的更改可以通过第二个挂载点（`/demo`）看到。

当我们在绑定挂载中描述绑定挂载时，提供了一个示例，说明为什么在多个挂载点挂载文件系统是有用的。

### 注意

由于一个设备可以在多个挂载点挂载，因此在 Linux 2.4 及以后的版本中，*umount()* 系统调用不能将设备作为参数。

### 在同一挂载点上堆叠多个挂载

在 2.4 版本之前，挂载点只能使用一次。从 2.4 版本开始，Linux 允许在单个挂载点上堆叠多个挂载。每个新的挂载都会隐藏在该挂载点上先前可见的目录子树。当堆栈顶部的挂载被卸载时，之前隐藏的挂载将再次可见，以下 shell 会话演示了这一效果：

```
$ `su`                                  *Privilege is required to use mount(8)*
Password:
# `mount /dev/sda12 /testfs`            *Create first mount on* /testfs
# `touch /testfs/myfile`                *Make a file in this subtree*
# `mount /dev/sda13 /testfs`            *Stack a second mount on* /testfs
# `mount | grep testfs`                 *Verify the setup*
/dev/sda12 on /testfs type ext3 (rw)
/dev/sda13 on /testfs type reiserfs (rw)
# `touch /testfs/newfile`               *Create a file in this subtree*
# `ls /testfs`                          *View files in this subtree*
newfile
# `umount /testfs`                      *Pop a mount from the stack*
# `mount | grep testfs`
/dev/sda12 on /testfs type ext3 (rw)
  *Now only one mount on* /testfs
# `ls /testfs`                          *Previous mount is now visible*
lost+found  myfile
```

挂载堆叠的一种用途是将新挂载堆叠到一个正在使用的现有挂载点。保持文件描述符打开、被*chroot()*限制，或当前工作目录位于旧挂载点内的进程继续在该挂载点下操作，但进行新访问的进程则使用新挂载点。结合`MNT_DETACH`卸载，这可以实现文件系统的平滑迁移，无需将系统切换到单用户模式。我们将在讨论*[tmpfs]*文件系统时看到堆叠挂载的另一种有用示例，详见 A 虚拟内存文件系统：*tmpfs*。

### 每个挂载选项的挂载标志

在 2.4 版本之前，文件系统和挂载点之间存在一一对应关系。由于这一点在 Linux 2.4 及以后的版本中不再成立，因此在挂载文件系统：*mount()*")中描述的一些*mountflags*值可以按每个挂载的基础上设置。这些标志包括`MS_NOATIME`（自 Linux 2.6.16 以来）、`MS_NODEV`、`MS_NODIRATIME`（自 Linux 2.6.16 以来）、`MS_NOEXEC`、`MS_NOSUID`、`MS_RDONLY`（自 Linux 2.6.26 以来）和`MS_RELATIME`。以下 shell 会话演示了`MS_NOEXEC`标志的效果：

```
$ `su`
Password:
# `mount /dev/sda12 /testfs`
# `mount -o noexec /dev/sda12 /demo`
# `cat /proc/mounts | grep sda12`
/dev/sda12 /testfs ext3 rw 0 0
/dev/sda12 /demo ext3 rw,noexec 0 0
# `cp /bin/echo /testfs`
# `/testfs/echo "Art is something which is well done"`
Art is something which is well done
# `/demo/echo "Art is something which is well done"`
bash: /demo/echo: Permission denied
```

### 绑定挂载

从 2.4 版本开始，Linux 允许创建绑定挂载。*绑定挂载*（使用*mount()*的`MS_BIND`标志创建）允许将目录或文件挂载到文件系统层次结构中的其他位置。这将导致该目录或文件在两个位置都可见。绑定挂载有点像硬链接，但在两个方面有所不同：

+   绑定挂载可以跨越文件系统挂载点（甚至是*chroot*监狱）。

+   可以为一个目录创建绑定挂载。

我们可以使用*—bind*选项通过 shell 创建一个绑定挂载，以下示例演示了这一点。

在第一个示例中，我们将一个目录绑定挂载到另一个位置，并展示在一个目录中创建的文件在另一个位置可见：

```
$ `su`                            *Privilege is required to use mount(8)*
Password:
# `pwd`
/testfs
# `mkdir d1`                      *Create directory to be bound at another location*
# `touch d1/x`                    *Create file in the directory*
# `mkdir d2`                      *Create mount point to which* d1 *will be bound*
# `mount --bind d1 d2`            *Create bind mount:* d1 *visible via* d2
# `ls d2`                         *Verify that we can see contents of* d1 *via* d2
x
# `touch d2/y`                    *Create second file in directory* d2
# `ls d1`                         *Verify that this change is visible via* d1
x  y
```

在第二个示例中，我们将文件绑定挂载到另一个位置，并演示通过一个挂载对文件的更改在另一个挂载下可见：

```
# `cat > f1`                      *Create file to be bound to another location*
`Chance is always powerful. Let your hook be always cast.`
*Type Control-D*
# `touch f2`                      *This is the new mount point*
# `mount --bind f1 f2`            *Bind* f1 *as* f2
# `mount | egrep '(d1|f1)'`       *See how mount points look*
/testfs/d1 on /testfs/d2 type none (rw,bind)
/testfs/f1 on /testfs/f2 type none (rw,bind)
# `cat >> f2`                     *Change* f2
`In the pool where you least expect it, will be a fish.`
# `cat f1`                        *The change is visible via original file* f1
Chance is always powerful. Let your hook be always cast.
In the pool where you least expect it, will be a fish.
# `rm f2`                         *Can't do this because it is a mount point*
rm: cannot unlink `f2': Device or resource busy
# `umount f2`                     *So unmount*
# `rm f2`                         *Now we can remove* f2
```

使用绑定挂载的一个例子是在创建*chroot*监狱时（更改进程的根目录：*chroot()*")）。我们可以通过为监狱中的这些目录（可能是只读挂载）创建绑定挂载，而不是在监狱中复制各种标准目录（如`/lib`）。

### 递归绑定挂载

默认情况下，如果我们使用`MS_BIND`为一个目录创建绑定挂载，那么只有该目录会被挂载到新位置；如果源目录下有任何子挂载，它们不会在挂载*目标*下被复制。Linux 2.4.11 添加了`MS_REC`标志，它可以与`MS_BIND`一起作为*flags*参数传递给*mount()*，以便子挂载会在挂载目标下被复制。这就是所谓的*递归绑定挂载*。*mount(8)*命令提供了`--rbind`选项，可以通过这个选项从 shell 中实现相同的效果，如下所示的 shell 会话所示。

我们首先创建一个挂载在`top`下的目录树（`src1`）。该树包含一个在`top/sub`处的子挂载（`src2`）。

```
$ `su`
Password:
# `mkdir top`                     *This is our top-level mount point*
# `mkdir src1`                    *We'll mount this under* top
# `touch src1/aaa`
# `mount --bind src1 top`         *Create a normal bind mount*
# `mkdir top/sub`                 *Create directory for a submount under* top
# `mkdir src2`                    *We'll mount this under* top/sub
# `touch src2/bbb`
# `mount --bind src2 top/sub`     *Create a normal bind mount*
# `find top`                      *Verify contents under* top *mount tree*
top
top/aaa
top/sub                         *This is the submount*
top/sub/bbb
```

现在我们使用`top`作为源，创建另一个绑定挂载（`dir1`）。由于这个新挂载是非递归的，因此子挂载不会被复制。

```
# `mkdir dir1`
# `mount --bind top dir1`         *Here we use a normal bind mount*
# `find dir1`
dir1
dir1/aaa
dir1/sub
```

在*find*命令的输出中没有`dir1/sub/bbb`，这表明子挂载`top/sub`没有被复制。

现在我们使用`top`作为源，创建一个递归绑定挂载（`dir2`）。

```
# `mkdir dir2`
# `mount --rbind top dir2`
# `find dir2`
dir2
dir2/aaa
dir2/sub
dir2/sub/bbb
```

在*find*命令的输出中存在`dir2/sub/bbb`，这表明子挂载`top/sub`已经被复制。

## 虚拟内存文件系统：*tmpfs*

到目前为止，我们在本章中描述的所有文件系统都驻留在磁盘上。然而，Linux 还支持驻留在内存中的*虚拟文件系统*的概念。对于应用程序来说，这些文件系统看起来就像任何其他文件系统——可以对这些文件系统中的文件和目录执行相同的操作（*open()*, *read()*, *write()*, *link()*, *mkdir()*等）。不过，有一个重要的区别：文件操作要快得多，因为不涉及磁盘访问。

各种基于内存的文件系统已经为 Linux 开发。到目前为止，最先进的是*tmpfs*文件系统，它首次出现在 Linux 2.4 中。*tmpfs*文件系统与其他基于内存的文件系统不同，它是一个*虚拟*内存文件系统。这意味着*tmpfs*不仅使用 RAM，如果 RAM 不足，还会使用交换空间。（尽管此处描述的*tmpfs*文件系统是 Linux 特有的，但大多数 UNIX 实现提供某种形式的基于内存的文件系统。）

### 注意

*tmpfs*文件系统是一个可选的 Linux 内核组件，可以通过`CONFIG_TMPFS`选项进行配置。

要创建一个*tmpfs*文件系统，我们使用以下形式的命令：

```
# `mount -t tmpfs` ``*`source target`*``
```

*source*可以是任何名称；它的唯一意义在于它出现在`/proc/mounts`中，并且由*mount*和*df*命令显示。如通常情况，*target*是文件系统的挂载点。请注意，不需要使用*mkfs*首先创建文件系统，因为内核会自动在*mount()*系统调用中构建一个文件系统。

作为使用*tmpfs*的示例，我们可以使用挂载堆叠（这样我们就不需要担心`/tmp`是否已经在使用）并创建一个挂载在`/tmp`的*tmpfs*文件系统，如下所示：

```
# `mount -t tmpfs newtmp /tmp`
# `cat /proc/mounts | grep tmp`
newtmp /tmp tmpfs rw 0 0
```

上述命令（或`/etc/fstab`中的等效条目）有时用于提高大量使用`/tmp`目录来创建临时文件的应用程序（例如编译器）的性能。

默认情况下，*tmpfs*文件系统的大小允许增长到 RAM 的一半，但可以使用*size=nbytes mount*选项来设置文件系统大小的不同上限，无论是在创建文件系统时还是在稍后的重新挂载时。（*tmpfs*文件系统仅消耗当前存储其文件所需的内存和交换空间。）

如果我们卸载一个*tmpfs*文件系统，或者系统崩溃，那么文件系统中的所有数据都会丢失；这就是*tmpfs*名称的由来。

除了被用户应用程序使用外，*tmpfs*文件系统还有两个特殊的用途：

+   一个由内核内部挂载的不可见*tmpfs*文件系统用于实现 System V 共享内存（第四十八章）和共享匿名内存映射（第四十九章）。

+   挂载在`/dev/shm`的*tmpfs*文件系统用于*glibc*实现的 POSIX 共享内存和 POSIX 信号量。

## 获取文件系统信息：*statvfs()*

*statvfs()*和*fstatvfs()*库函数用于获取挂载文件系统的信息。

```
#include <sys/statvfs.h>

int `statvfs`(const char **pathname*, struct statvfs **statvfsbuf*);
int `fstatvfs`(int *fd*, struct statvfs **statvfsbuf*);
```

### 注意

两者在成功时返回 0，出错时返回-1。

这两个函数之间的唯一区别在于文件系统的识别方式。对于*statvfs()*，我们使用*pathname*来指定文件系统中任何文件的名称。对于*fstatvfs()*，我们指定一个打开的文件描述符*fd*，它指向文件系统中的任何文件。这两个函数都返回一个*statvfs*结构，包含指向*statvfsbuf*的缓冲区中的文件系统信息。该结构具有以下形式：

```
struct statvfs {
    unsigned long f_bsize;     /* File-system block size (in bytes) */
    unsigned long f_frsize;    /* Fundamental file-system block size
                                  (in bytes) */
    fsblkcnt_t    f_blocks;    /* Total number of blocks in file
                                  system (in units of 'f_frsize') */
    fsblkcnt_t    f_bfree;     /* Total number of free blocks */
    fsblkcnt_t    f_bavail;    /* Number of free blocks available to
                                  unprivileged process */
    fsfilcnt_t    f_files;     /* Total number of i-nodes */
    fsfilcnt_t    f_ffree;     /* Total number of free i-nodes */
    fsfilcnt_t    f_favail;    /* Number of i-nodes available to unprivileged
                                  process (set to 'f_ffree' on Linux) */
    unsigned long f_fsid;      /* File-system ID */
    unsigned long f_flag;      /* Mount flags */
    unsigned long f_namemax;   /* Maximum length of filenames on
                                  this file system */
};
```

*statvfs*结构中大多数字段的目的如上面注释中所述。我们注意到一些字段的进一步说明：

+   *fsblkcnt_t*和*fsfilcnt_t*数据类型是由 SUSv3 指定的整数类型。

+   对于大多数 Linux 文件系统，*f_bsize* 和 *f_frsize* 的值是相同的。然而，一些文件系统支持块片段的概念，如果不需要完整的块，可以在文件末尾分配一个更小的存储单元。这样可以避免分配完整块时浪费空间。在这些文件系统中，*f_frsize* 是片段的大小，*f_bsize* 是完整块的大小。（UNIX 文件系统中的片段概念首次出现在 1980 年代初期的 4.2BSD 快速文件系统中，详见 [McKusick 等，1984]。）

+   许多原生 UNIX 和 Linux 文件系统支持为超级用户保留文件系统的一定块，以便如果文件系统已满，超级用户仍然可以登录系统并进行一些工作来解决问题。如果文件系统中有保留的块，那么 *statvfs* 结构中 *f_bfree* 和 *f_bavail* 字段的值差异告诉我们有多少块是保留的。

+   *f_flag* 字段是用于挂载文件系统的标志的位掩码；也就是说，它包含类似于传递给 *mount(2)* 的 *mountflags* 参数的信息。然而，该字段中用于位的常量的名称以 `ST_` 开头，而不是 *mountflags* 中使用的 `MS_`。SUSv3 只要求 `ST_RDONLY` 和 `ST_NOSUID` 常量，但 *glibc* 实现支持一整套常量，名称对应于 *mount() mountflags* 参数中描述的 `MS_*` 常量。

+   *f_fsid* 字段在某些 UNIX 实现中用于返回文件系统的唯一标识符——例如，基于文件系统所在设备的标识符。对于大多数 Linux 文件系统，该字段包含 0。

SUSv3 规定了 *statvfs()* 和 *fstatvfs()*。在 Linux（以及其他一些 UNIX 实现）中，这些函数是建立在类似的 *statfs()* 和 *fstatfs()* 系统调用之上的。（一些 UNIX 实现提供 *statfs()* 系统调用，但不提供 *statvfs()*。）主要区别（除了某些字段名称不同）如下：

+   *statvfs()* 和 *fstatvfs()* 函数返回 *f_flag* 字段，提供关于文件系统挂载标志的信息。（*glibc* 实现通过扫描 `/proc/mounts` 或 `/etc/mtab` 获取此信息。）

+   *statfs()* 和 *fstatfs()* 系统调用返回字段 *f_type*，提供文件系统的类型（例如，值 `0xef53` 表示这是一个 *ext2* 文件系统）。

### 注意

本书源代码分发包中的 `filesys` 子目录包含两个文件，`t_statvfs.c` 和 `t_statfs.c`，演示了 *statvfs()* 和 *statfs()* 的使用。

## 总结

设备通过 `/dev` 目录中的条目表示。每个设备都有一个相应的设备驱动程序，该驱动程序实现一套标准操作，包括与 *open()*、*read()*、*write()* 和 *close()* 系统调用对应的操作。设备可以是真实的，意味着有相应的硬件设备，或者是虚拟的，意味着没有硬件设备存在，但内核仍然提供一个设备驱动程序，实现在 API 层面与真实设备相同。

硬盘被划分为一个或多个分区，每个分区可能包含一个文件系统。文件系统是一个组织良好的常规文件和目录的集合。Linux 实现了多种文件系统，包括传统的 *ext2* 文件系统。*ext2* 文件系统在概念上类似于早期的 UNIX 文件系统，包含一个启动块、一个超级块、一个 i-node 表和一个包含文件数据块的数据区域。每个文件在文件系统的 i-node 表中都有一个条目。该条目包含有关文件的各种信息，包括其类型、大小、链接计数、所有权、权限、时间戳以及指向文件数据块的指针。

Linux 提供了一系列日志文件系统，包括 *Reiserfs*、*ext3*、*ext4*、*XFS*、*JFS* 和 *Btrfs*。日志文件系统会在实际文件更新之前，将元数据更新（在某些文件系统中可选地包括数据更新）记录到日志文件中。这意味着在系统崩溃的情况下，可以通过重放日志文件来快速将文件系统恢复到一致的状态。日志文件系统的主要优点是，它们避免了传统 UNIX 文件系统在系统崩溃后需要进行的长时间文件系统一致性检查。

Linux 系统上的所有文件系统都挂载在一个单一的目录树下，目录 `/` 为其根目录。文件系统在目录树中的挂载位置称为其挂载点。

特权进程可以使用 *mount()* 和 *umount()* 系统调用来挂载和卸载文件系统。有关已挂载文件系统的信息可以通过 *statvfs()* 获取。

#### 进一步的信息

有关设备和设备驱动程序的详细信息，请参阅 [Bovet & Cesati, 2005]，尤其是 [Corbet et al., 2005]。有关设备的一些有用信息可以在内核源文件 `Documentation/devices.txt` 中找到。

有几本书提供了有关文件系统的进一步信息。[Tanenbaum, 2007] 是一本关于文件系统结构和实现的一般介绍。[Bach, 1986] 提供了关于 UNIX 文件系统实现的介绍，主要面向 System V。[Vahalia, 1996] 和 [Goodheart & Cox, 1994] 也描述了 System V 文件系统的实现。[Love, 2010] 和 [Bovet & Cesati, 2005] 描述了 Linux VFS 的实现。

有关各种文件系统的文档可以在内核源代码子目录`Documentation/filesystems`中找到。描述 Linux 上大多数文件系统实现的个人网站也可以找到。

## 练习

1.  编写一个程序，测量在单个目录中创建并删除大量 1 字节文件所需的时间。程序应创建名称形式为`xNNNNNN`的文件，其中`NNNNNN`由一个随机的六位数字替换。文件应按照生成名称的随机顺序创建，然后按递增的数字顺序删除（即与创建时的顺序不同）。文件数量（*NF*）和文件创建目录应在命令行中指定。测量不同*NF*值（例如从 1000 到 20000 的范围）和不同文件系统（例如*ext2*、*ext3*、*XFS*）下的时间。随着*NF*的增加，你在每种文件系统上观察到什么模式？不同的文件系统如何比较？如果文件按递增数字顺序创建（`x000001`、`x000002`、`x000003`，以此类推），并按相同的顺序删除，结果是否有所不同？如果不同，你认为可能的原因是什么？同样，结果是否在不同文件系统类型之间有所变化？
