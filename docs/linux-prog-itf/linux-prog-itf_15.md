## 第十五章 文件属性

在本章中，我们研究文件的各种属性（文件元数据）。我们首先介绍*stat()*系统调用，它返回一个包含许多这些属性的结构体，包括文件时间戳、文件所有权和文件权限。接下来，我们讨论用于更改这些属性的各种系统调用。（关于文件权限的讨论将在第十七章中继续，那里我们将讨论访问控制列表。）本章的最后，我们讨论 i 节点标志（也称为*ext2*扩展文件属性），它们控制内核对文件的处理方式。

## 获取文件信息：*stat()*

*stat()*、*lstat()*和*fstat()*系统调用获取关于文件的信息，主要来自文件的 i 节点。

```
#include <sys/stat.h>

int `stat`(const char **pathname*, struct stat **statbuf*);
int `lstat`(const char **pathname*, struct stat **statbuf*);
int `fstat`(int *fd*, struct stat **statbuf*);
```

### 注意

所有函数成功时返回 0，出错时返回-1。

这三种系统调用仅在指定文件的方式上有所不同：

+   *stat()*返回一个命名文件的信息；

+   *lstat()*与*stat()*类似，不同之处在于如果命名的文件是符号链接，则返回关于链接本身的信息，而不是链接指向的文件的信息；

+   *fstat()*返回由打开的文件描述符引用的文件的信息。

*stat()*和*lstat()*系统调用不需要对文件本身有权限。然而，*pathname*中指定的所有父目录都需要有执行（搜索）权限。*fstat()*系统调用在提供有效的文件描述符时始终成功。

所有这些系统调用都在由*statbuf*指向的缓冲区中返回一个*stat*结构体。这个结构体的形式如下：

```
struct stat {
    dev_t     st_dev;         /* IDs of device on which file resides */
    ino_t     st_ino;         /* I-node number of file */
    mode_t    st_mode;        /* File type and permissions */
    nlink_t   st_nlink;       /* Number of (hard) links to file */
    uid_t     st_uid;         /* User ID of file owner */
    gid_t     st_gid;         /* Group ID of file owner */
    dev_t     st_rdev;        /* IDs for device special files */
    off_t     st_size;        /* Total file size (bytes) */
    blksize_t st_blksize;     /* Optimal block size for I/O (bytes) */
    blkcnt_t  st_blocks;      /* Number of (512B) blocks allocated */
    time_t    st_atime;       /* Time of last file access */
    time_t    st_mtime;       /* Time of last file modification */
    time_t    st_ctime;       /* Time of last status change */
};
```

用于定义*stat*结构字段的数据类型在 SUSv3 中都有明确规定。有关这些类型的更多信息，请参见系统数据类型。

### 注意

根据 SUSv3 标准，当*lstat()*应用于符号链接时，它只需要在*st_size*字段和*st_mode*字段中的文件类型组件（稍后描述）返回有效信息。其他字段（例如时间字段）不需要包含有效信息。这使得实现可以自由选择不维护这些字段，出于效率考虑可能会这样做。特别地，早期 UNIX 标准的意图是允许符号链接作为 i 节点或目录条目实现。在后一种实现中，无法实现*stat*结构要求的所有字段。（在所有主要的现代 UNIX 实现中，符号链接作为 i 节点实现。有关更多细节，请参见符号（软）链接。）在 Linux 中，*lstat()*在应用于符号链接时返回所有*stat*字段的信息。

在接下来的页面中，我们将更详细地查看一些 *stat* 结构字段，并以一个示例程序结束，显示整个 *stat* 结构。

#### 设备 ID 和 i-node 号

*st_dev* 字段标识文件所在的设备。*st_ino* 字段包含文件的 i-node 号。*st_dev* 和 *st_ino* 的组合唯一标识一个文件，在所有文件系统中都有效。*dev_t* 类型记录设备的主次设备 ID（设备特殊文件（设备））。

如果这是一个设备的 i-node，那么 *st_rdev* 字段包含该设备的主次设备 ID。

可以使用两个宏：`major()` 和 `minor()` 来提取 *dev_t* 值的主次设备 ID。获取这两个宏声明的头文件在不同的 UNIX 实现中有所不同。在 Linux 中，如果定义了 `_BSD_SOURCE` 宏，则它们由 `<sys/types.h>` 提供。

`major()` 和 `minor()` 返回的整数值在不同的 UNIX 实现中大小不同。为了移植性，我们始终将返回值强制转换为 *long* 类型进行打印（参见 系统数据类型）。

#### 文件所有权

*st_uid* 和 *st_gid* 字段分别标识文件的所有者（用户 ID）和所属组（组 ID）。

#### 链接计数

*st_nlink* 字段是指向该文件的（硬）链接数。我们在 第十八章 中详细描述了链接。

#### 文件类型和权限

*st_mode* 字段是一个位掩码，具有双重功能：识别文件类型和指定文件权限。该字段的位布局如 图 15-1 所示。

![st_mode 位掩码的布局](img/15-1_FILES-st_mode.png.jpg)图 15-1. *st_mode* 位掩码的布局

文件类型可以通过与常量 `S_IFMT` 进行与（`&`）运算从该字段提取出来。（在 Linux 上，*st_mode* 字段的文件类型部分使用 4 位。然而，由于 SUSv3 没有规定文件类型的表示方式，因此此细节可能在不同实现中有所不同。）然后，可以将得到的值与一系列常量进行比较，以确定文件类型，如下所示：

```
if ((statbuf.st_mode & S_IFMT) == S_IFREG)
    printf("regular file\n");
```

由于这是一个常见操作，提供了标准宏来简化上述内容，简化为以下内容：

```
if (S_ISREG(statbuf.st_mode))
    printf("regular file\n");
```

完整的文件类型宏集合（定义在`<sys/stat.h>`中）见表 15-1。所有在表 15-1 中的文件类型宏都在 SUSv3 中有规定，并且在 Linux 上出现。一些其他的 UNIX 实现定义了额外的文件类型（例如，`S_IFDOOR`，用于 Solaris 上的门文件）。类型`S_IFLNK`仅通过调用*lstat()*返回，因为调用*stat()*总是会遵循符号链接。

原始的 POSIX.1 标准并没有指定表 15-1 中第一列所示的常量，尽管它们大多数在大多数 UNIX 实现中都有出现。SUSv3 要求这些常量。

### 注意

为了从`<sys/stat.h>`中获取`S_IFSOCK`和`S_ISSOCK()`的定义，我们必须定义`_BSD_SOURCE`特性测试宏，或者定义`_XOPEN_SOURCE`并设置其值大于或等于 500。（这些规则在不同的*glibc*版本中略有不同：在某些情况下，`_XOPEN_SOURCE`必须定义为 600 或更大值。）

表 15-1. 用于检查*stat*结构体的*st_mode*字段中文件类型的宏

| 常量 | 测试宏 | 文件类型 |
| --- | --- | --- |
| `S_IFREG` | `S_ISREG()` | 常规文件 |
| `S_IFDIR` | `S_ISDIR()` | 目录 |
| `S_IFCHR` | `S_ISCHR()` | 字符设备 |
| `S_IFBLK` | `S_ISBLK()` | 块设备 |
| `S_IFIFO` | `S_ISFIFO()` | FIFO 或管道 |
| `S_IFSOCK` | `S_ISSOCK()` | 套接字 |
| `S_IFLNK` | `S_ISLNK()` | 符号链接 |

*st_mode*字段的最低 12 位定义了文件的权限。我们在第 15.4 节中描述了文件权限位。目前，我们只需注意，权限位中的 9 个位是每个类别（所有者、组和其他）的读、写和执行权限。

#### 文件大小、分配的块数和最佳 I/O 块大小

对于常规文件，*st_size*字段是文件的总大小，以字节为单位。对于符号链接，该字段包含链接指向的路径名的长度（以字节为单位）。对于共享内存对象（第五十四章），该字段包含对象的大小。

*st_blocks*字段表示分配给文件的总块数，单位为 512 字节的块。这个总数包括为指针块分配的空间（参见图 14-2, 在 I 节点和*ext2*中的数据块指针）。选择 512 字节作为度量单位是出于历史原因——这是 UNIX 下所有实现的文件系统中最小的块大小。现代文件系统使用更大的逻辑块大小。例如，在*ext2*下，*st_blocks*中的值始终是 2、4 或 8 的倍数，具体取决于*ext2*逻辑块大小是 1024、2048 还是 4096 字节。

### 注意

SUSv3 没有定义*st_blocks*的度量单位，这使得实现可能使用除 512 字节以外的其他单位。大多数 UNIX 实现使用 512 字节单位，但 HP-UX 11 使用特定于文件系统的单位（例如，在某些情况下为 1024 字节）。

*st_blocks*字段记录了实际分配的磁盘块数量。如果文件包含空洞（更改文件偏移量：*lseek()*")），那么此值将小于文件中对应字节数*(st_size)*的预期值。（磁盘使用命令，*du -k file*，显示文件实际分配的空间，以千字节为单位；也就是说，这是根据文件的*st_blocks*值计算出来的，而不是*st_size*值。）

*st_blksize*字段的名称有些误导。它不是底层文件系统的块大小，而是该文件系统上文件 I/O 的最佳块大小（以字节为单位）。I/O 操作如果使用比此值更小的块，将效率较低（参考内核文件 I/O 缓冲：缓冲区缓存）。*st_blksize*返回的典型值为 4096。

#### 文件时间戳

*st_atime*、*st_mtime*和*st_ctime*字段分别包含最后访问文件、最后修改文件和最后状态更改的时间。这些字段的类型是*time_t*，即 UNIX 标准时间格式，表示自纪元以来的秒数。我们将在 15.2 节中进一步介绍这些字段。

#### 示例程序

示例 15-1 中的程序使用 *stat()* 来检索命令行中指定的文件信息。如果指定了 *-l* 命令行选项，则程序改用 *lstat()*，这样我们可以检索符号链接的信息，而不是它所指向的文件信息。程序打印返回的 *stat* 结构的所有字段。（关于为何将 *st_size* 和 *st_blocks* 字段强制转换为 *long long* 类型的解释，请参见 大文件的 I/O）。该程序使用的 *filePermStr()* 函数展示于 示例 15-4，在 目录权限中也有展示。

下面是该程序的使用示例：

```
$ `echo 'All operating systems provide services for programs they run' > apue`
$ `chmod g+s apue`        *Turn on set-group-ID bit; affects last status change time*
$ `cat apue`              *Affects last file access time*
All operating systems provide services for programs they run
$ `./t_stat apue`
File type:                regular file
Device containing i-node: major=3   minor=11
I-node number:            234363
Mode:                     102644 (rw-r--r--)
    special bits set:     set-GID
Number of (hard) links:   1
Ownership:                UID=1000   GID=100
File size:                61 bytes
Optimal I/O block size:   4096 bytes
512B blocks allocated:    8
Last file access:         Mon Jun  8 09:40:07 2011
Last file modification:   Mon Jun  8 09:39:25 2011
Last status change:       Mon Jun  8 09:39:51 2011
```

示例 15-1. 获取和解释文件 *stat* 信息

```
`files/t_stat.c`
#define _BSD_SOURCE     /* Get major() and minor() from <sys/types.h> */
#include <sys/types.h>
#include <sys/stat.h>
#include <time.h>
#include "file_perms.h"
#include "tlpi_hdr.h"

static void
displayStatInfo(const struct stat *sb)
{
    printf("File type:                ");

    switch (sb->st_mode & S_IFMT) {
    case S_IFREG:  printf("regular file\n");            break;
    case S_IFDIR:  printf("directory\n");               break;
    case S_IFCHR:  printf("character device\n");        break;
    case S_IFBLK:  printf("block device\n");            break;
    case S_IFLNK:  printf("symbolic (soft) link\n");    break;
    case S_IFIFO:  printf("FIFO or pipe\n");            break;
    case S_IFSOCK: printf("socket\n");                  break;
    default:       printf("unknown file type?\n");      break;
    }

    printf("Device containing i-node: major=%ld   minor=%ld\n",
                (long) major(sb->st_dev), (long) minor(sb->st_dev));

    printf("I-node number:            %ld\n", (long) sb->st_ino);

    printf("Mode:                     %lo (%s)\n",
            (unsigned long) sb->st_mode, filePermStr(sb->st_mode, 0));

    if (sb->st_mode & (S_ISUID | S_ISGID | S_ISVTX))
        printf("    special bits set:     %s%s%s\n",
                (sb->st_mode & S_ISUID) ? "set-UID " : "",
                (sb->st_mode & S_ISGID) ? "set-GID " : "",
                (sb->st_mode & S_ISVTX) ? "sticky " : "");

    printf("Number of (hard) links:   %ld\n", (long) sb->st_nlink);

    printf("Ownership:                UID=%ld   GID=%ld\n",
            (long) sb->st_uid, (long) sb->st_gid);

    if (S_ISCHR(sb->st_mode) || S_ISBLK(sb->st_mode))
        printf("Device number (st_rdev):  major=%ld; minor=%ld\n",
                (long) major(sb->st_rdev), (long) minor(sb->st_rdev));

    printf("File size:                %lld bytes\n", (long long) sb->st_size);
    printf("Optimal I/O block size:   %ld bytes\n", (long) sb->st_blksize);
    printf("512B blocks allocated:    %lld\n", (long long) sb->st_blocks);

    printf("Last file access:         %s", ctime(&sb->st_atime));
    printf("Last file modification:   %s", ctime(&sb->st_mtime));
    printf("Last status change:       %s", ctime(&sb->st_ctime));
}

int
main(int argc, char *argv[])
{
    struct stat sb;
    Boolean statLink;           /* True if "-l" specified (i.e., use lstat) */
    int fname;                  /* Location of filename argument in argv[] */

    statLink = (argc > 1) && strcmp(argv[1], "-l") == 0;
                                /* Simple parsing for "-l" */
    fname = statLink ? 2 : 1;

    if (fname >= argc || (argc > 1 && strcmp(argv[1], "--help") == 0))
        usageErr("%s [-l] file\n"
                "        -l = use lstat() instead of stat()\n", argv[0]);

    if (statLink) {
        if (lstat(argv[fname], &sb) == -1)
            errExit("lstat");
    } else {
        if (stat(argv[fname], &sb) == -1)
            errExit("stat");
    }

    displayStatInfo(&sb);

    exit(EXIT_SUCCESS);
}
      `files/t_stat.c`
```

## 文件时间戳

*stat* 结构中的 *st_atime*、*st_mtime* 和 *st_ctime* 字段包含文件时间戳。这些字段分别记录了文件的最后访问时间、最后修改时间和最后状态更改时间（即文件的 i-node 信息最后一次修改时间）。时间戳以自纪元（1970 年 1 月 1 日；参见 日历时间）以来的秒数记录。

大多数本地 Linux 和 UNIX 文件系统都支持所有时间戳字段，但一些非 UNIX 文件系统可能不支持。

表 15-2 总结了哪些时间戳字段（以及在某些情况下，父目录中的类似字段）会被本书中描述的各种系统调用和库函数所更改。在此表的标题中，*a*、*m* 和 *c* 分别代表 *st_atime*、*st_mtime* 和 *st_ctime* 字段。在大多数情况下，相关时间戳会被系统调用设置为当前时间。例外情况是 *utime()* 和类似的调用（讨论见 纳秒时间戳 和 使用 *utime()* 和 *utimes()* 更改文件时间戳 和 utimes() 更改文件时间戳")），这些调用可以用来显式地将文件的最后访问时间和修改时间设置为任意值。

表 15-2. 各种函数对文件时间戳的影响

| 函数 | 文件或目录 | 父目录 | 备注 |
| --- | --- | --- | --- |
| a | m | c | a | m | c |
| --- | --- | --- | --- | --- | --- |
| *chmod()* |   |   | • |   |   |   | 同样适用于 *fchmod()* |
| *chown()* |   |   | • |   |   |   | 同样适用于 *lchown()* 和 *fchown()* |
| *exec()* | • |   |   |   |   |   |   |
| *link()* |   |   | • |   | • | • | 影响第二个参数的父目录 |
| *mkdir()* | • | • | • |   | • | • |   |
| *mkfifo()* | • | • | • |   | • | • |   |
| *mknod()* | • | • | • |   | • | • |   |
| *mmap()* | • | • | • |   |   |   | 只有在更新 `MAP_SHARED` 映射时，*st_mtime* 和 *st_ctime* 才会改变 |
| *msync()* |   | • | • |   |   |   | 只有在文件被修改时才会发生变化 |
| *open()*, *creat()* | • | • | • |   | • | • | 创建新文件时 |
| *open()*, *creat()* |   | • | • |   |   |   | 当截断现有文件时 |
| *pipe()* | • | • | • |   |   |   |   |
| *read()* | • |   |   |   |   |   | 同样适用于 *readv()*, *pread()*, 和 *preadv()* |
| *readdir()* | • |   |   |   |   |   | readdir() 可能会缓存目录条目；只有在读取目录时才会更新时间戳 |
| *removexattr()* |   |   | • |   |   |   | 同样适用于 *fremovexattr()* 和 *lremovexattr()* |
| *rename()* |   |   | • |   | • | • | 影响父目录中的时间戳；SUSv3 没有指定文件 *st_ctime* 变化，但指出某些实现会这么做 |
| *rmdir()* |   |   |   |   | • | • | 同样适用于 *remove(directory)* |
| *sendfile()* | • |   |   |   |   |   | 输入文件的时间戳已更改 |
| *setxattr()* |   |   | • |   |   |   | 同样适用于 *fsetxattr()* 和 *lsetxattr()* |
| *symlink()* | • | • | • |   | • | • | 设置的是符号链接的时间戳（而不是目标文件） |
| *truncate()* |   | • | • |   |   |   | 同样适用于 *ftruncate()*；只有在文件大小发生变化时，时间戳才会更改 |
| *unlink()* |   |   | • |   | • | • | 同样适用于 *remove(file)*；如果先前的链接计数大于 1，则文件 *st_ctime* 会发生变化 |
| *utime()* | • | • | • |   |   |   | 同样适用于 *utimes()*, *futimes()*, *futimens()*, *lutimes()*, 和 *utimensat()* |
| *write()* |   | • | • |   |   |   | 同样适用于 *writev()*, *pwrite()*, 和 *pwritev()* |

在 挂载文件系统: *mount()*") 和 I 节点标志 (*ext2* 扩展文件属性)") 中，我们描述了 *mount(2)* 选项和每个文件标志，这些选项和标志防止更新文件的最后访问时间。 文件描述符号由 *open()* 返回 返回") 中描述的 *open()* `O_NOATIME` 标志也有类似的作用。在一些应用中，这对性能有帮助，因为它减少了文件访问时所需的磁盘操作次数。

### 注意

尽管大多数 UNIX 系统不会记录文件的创建时间，但在最近的 BSD 系统中，这个时间会记录在一个名为 *st_birthtime* 的 *stat* 字段中。

#### 纳秒时间戳

从版本 2.6 开始，Linux 支持 *stat* 结构中三个时间戳字段的纳秒分辨率。纳秒分辨率提高了需要基于文件时间戳相对顺序做出决策的程序的准确性（例如，*make(1)*）。

SUSv3 并没有为 *stat* 结构指定纳秒时间戳，但 SUSv4 添加了这一规范。

并非所有文件系统都支持纳秒时间戳。*JFS*、*XFS*、*ext4* 和 *Btrfs* 支持，但 *ext2*、*ext3* 和 *Reiserfs* 不支持。

在 *glibc* API（自 2.3 版本起）下，时间戳字段被定义为 *timespec* 结构（当我们讨论 *utimensat()* 时会介绍此结构），该结构表示秒和纳秒组件的时间。合适的宏定义使得这些结构的秒组件可以通过传统的字段名称（*st_atime*、*st_mtime* 和 *st_ctime*）显示。纳秒组件可以通过如 *st_atim.tv_nsec* 这样的字段名访问，用于表示最后文件访问时间的纳秒部分。

### 使用 *utime()* 和 *utimes()* 修改文件时间戳

存储在文件 i 节点中的最后文件访问和修改时间戳可以通过 *utime()* 或一组相关的系统调用显式更改。像 *tar(1)* 和 *unzip(1)* 这样的程序使用这些系统调用在解压档案时重置文件时间戳。

```
#include <utime.h>

int `utime`(const char **pathname*, const struct utimbuf **buf*);
```

### 注

成功时返回 0，出错时返回 -1

*pathname* 参数用于指定我们希望修改时间的文件。如果 *pathname* 是一个符号链接，它会被解除引用。*buf* 参数可以是 `NULL` 或指向 *utimbuf* 结构的指针：

```
struct utimbuf {
    time_t actime;      /* Access time */
    time_t modtime;     /* Modification time */
};
```

该结构中的字段以自纪元（日历时间）以来的秒数来表示时间。

两种不同的情况决定了 *utime()* 的工作方式：

+   如果 *buf* 被指定为 `NULL`，则将最后的访问时间和最后的修改时间都设置为当前时间。在这种情况下，进程的有效用户 ID 必须与文件的用户 ID（所有者）匹配，进程必须具有文件的写权限（这是合乎逻辑的，因为一个拥有文件写权限的进程可以使用其他系统调用来间接修改这些文件时间戳），或者进程必须具备特权（`CAP_FOWNER` 或 `CAP_DAC_OVERRIDE`）。 （准确来说，在 Linux 上，检查的是进程的文件系统用户 ID，而不是有效用户 ID，如第 9.5 节所述。）

+   如果 *buf* 被指定为指向 *utimbuf* 结构的指针，则使用该结构的相应字段更新最后的文件访问时间和修改时间。在这种情况下，进程的有效用户 ID 必须与文件的用户 ID 匹配（仅具有写权限是不够的），或者调用者必须具备特权（`CAP_FOWNER`）。

若要仅更改一个文件时间戳，首先使用 *stat()* 获取两个时间，然后用其中一个时间初始化 *utimbuf* 结构，再按需要设置另一个时间。以下代码演示了这一过程，它使文件的最后修改时间与最后访问时间相同：

```
struct stat sb;
struct utimbuf utb;

if (stat(pathname, &sb) == -1)
    errExit("stat");
utb.actime = sb.st_atime;       /* Leave access time unchanged */
utb.modtime = sb.st_atime;
if (utime(pathname, &utb) == -1)
    errExit("utime");
```

成功调用 *utime()* 后，总是将最后的状态变化时间设置为当前时间。

Linux 还提供了 BSD 衍生的*utimes()*系统调用，它执行与*utime()*相似的任务。

```
#include <sys/time.h>

int `utimes`(const char **pathname*, const struct timeval *tv[2]*);
```

### 注意

成功时返回 0，出错时返回-1

*utime()*和*utimes()*之间最显著的区别在于，*utimes()*允许使用微秒精度指定时间值（*timeval*结构在日历时间中描述）。这为 Linux 2.6 提供的纳秒精度文件时间戳提供了（部分）访问。新的文件访问时间在*tv[0]*中指定，新的修改时间在*tv[1]*中指定。

### 注意

使用*utimes()*的示例在本书的源代码分发中的`files/t_utimes.c`文件中提供。

*futimes()*和*lutimes()*库函数执行与*utimes()*相似的任务。它们与*utimes()*的不同之处在于用于指定要更改时间戳的文件的参数。

```
#include <sys/time.h>

int `futimes`(int *fd*, const struct timeval *tv[2]*);
int `lutimes`(const char **pathname*, const struct timeval *tv[2]*);
```

### 注意

成功时返回 0，出错时返回-1

使用*futimes()*时，通过打开的文件描述符*fd*指定文件。

使用*lutimes()*时，通过路径名指定文件，区别在于与*utimes()*不同的是，如果路径名指向符号链接，则不会取消引用该链接；相反，链接本身的时间戳会被更改。

*futimes()*函数自*glibc* 2.3 起支持。*lutimes()*函数自*glibc* 2.6 起支持。

### 使用*utimensat()*和*futimens()*更改文件时间戳

*utimensat()*系统调用（自内核 2.6.22 起支持）和*futimens()*库函数（自*glibc* 2.6 起支持）提供了设置文件最后访问时间和最后修改时间戳的扩展功能。以下是这些接口的一些优点：

+   我们可以以纳秒精度设置时间戳。这比*utimes()*提供的微秒精度更精确。

+   可以独立设置时间戳（即一次设置一个）。如前所述，要使用旧的接口只更改一个时间戳，我们必须首先调用*stat()*以检索另一个时间戳的值，然后指定检索到的值以及我们要更改的时间戳的值。（如果另一个进程在这两个步骤之间执行了一个更新时间戳的操作，这可能会导致竞态条件。）

+   我们可以独立设置任何一个时间戳为当前时间。要使用旧的接口仅将一个时间戳更改为当前时间，我们需要调用*stat()*以检索我们希望保留不变的时间戳的设置，并调用*gettimeofday()*来获取当前时间。

这些接口未在 SUSv3 中指定，但在 SUSv4 中包含。

*utimensat()*系统调用将指定路径*pathname*的文件时间戳更新为*times*数组中指定的值。

```
#define _XOPEN_SOURCE 700     /* Or define _POSIX_C_SOURCE >= 200809 */
#include <sys/stat.h>

int `utimensat`(int *dirfd*, const char **pathname*,
              const struct timespec *times[2]*, int *flags*);
```

### 注意

成功时返回 0，出错时返回-1

如果 *times* 被指定为 `NULL`，则两个文件时间戳都会更新为当前时间。如果 *times* 不是 `NULL`，则新的最后访问时间戳在 *times[0]* 中指定，新的最后修改时间戳在 *times[1]* 中指定。数组 *times* 的每个元素都是以下形式的结构：

```
struct timespec {
    time_t tv_sec;     /* Seconds ('time_t' is an integer type) */
    long   tv_nsec;    /* Nanoseconds */
};
```

该结构中的字段指定自纪元以来的秒数和纳秒数（见 日历时间）。

要将其中一个时间戳设置为当前时间，我们在相应的 *tv_nsec* 字段中指定特殊值 `UTIME_NOW`。要保持其中一个时间戳不变，我们在相应的 *tv_nsec* 字段中指定特殊值 `UTIME_OMIT`。在这两种情况下，相应的 *tv_sec* 字段中的值将被忽略。

*dirfd* 参数可以指定 `AT_FDCWD`，此时 *pathname* 参数的解释方式与 *utimes()* 相同，或者它可以指定一个指向目录的文件描述符。后一种选择的目的在于 相对于目录文件描述符操作 中有所描述。

*flags* 参数可以是 0，或者是 `AT_SYMLINK_NOFOLLOW`，表示如果 *pathname* 是符号链接，则不应取消引用它（即，应更改符号链接本身的时间戳）。与此相对，*utimes()* 总是取消引用符号链接。

以下代码段将最后的访问时间设置为当前时间，而不改变最后的修改时间：

```
struct timespec times[2];

times[0].tv_sec = 0;
times[0].tv_nsec = UTIME_NOW;
times[1].tv_sec = 0;
times[1].tv_nsec = UTIME_OMIT;
if (utimensat(AT_FDCWD, "myfile", times, 0) == -1)
    errExit("utimensat");
```

使用 *utimensat()*（和 *futimens()*）更改时间戳的权限规则与旧的 API 相似，详细信息请参见 *utimensat(2)* 手册页面。

*futimens()* 库函数更新由打开文件描述符 *fd* 引用的文件的时间戳。

```
#include _GNU_SOURCE
#include <sys/stat.h>

int `futimens`(int *fd*, const struct timespec *times[2]*);
```

### 注意

成功时返回 0，出错时返回 -1。

*futimens()* 的 *times* 参数使用方式与 *utimensat()* 相同。

## 文件所有权

每个文件都有一个关联的用户 ID（UID）和组 ID（GID）。这些 ID 决定了文件属于哪个用户和组。接下来我们将讨论确定新文件所有权的规则，并描述用于更改文件所有权的系统调用。

### 新文件的所有权

当新文件创建时，它的用户 ID 来自进程的有效用户 ID。新文件的组 ID 可以来自进程的有效组 ID（相当于 System V 的默认行为），或者来自父目录的组 ID（BSD 行为）。后一种情况对于创建项目目录很有用，在这些目录中所有文件都属于一个特定组，并且该组的成员可以访问这些文件。哪个值被用作新文件的组 ID 由多个因素决定，包括新文件所在的文件系统类型。我们从描述 *ext2* 及其他一些文件系统的规则开始。

### 注意

准确地说，在 Linux 上，本节中所有使用的 *有效用户* 或 *组 ID* 术语实际上应该是 *文件系统用户* 或 *组 ID*（文件系统用户 ID 和文件系统组 ID）。

当挂载 *ext2* 文件系统时，可以通过 *mount* 命令指定 *-o grpid*（或同义的 *-o bsdgroups*）选项，或者 *-o nogrpid*（或同义的 *-o sysvgroups*）选项。（如果没有指定任何选项，默认使用 *-o nogrpid*。）如果指定了 *-o grpid*，则新文件总是从父目录继承其组 ID。如果指定了 *-o nogrpid*，则默认情况下，新文件从进程的有效组 ID 中获取其组 ID。然而，如果目录启用了设置组 ID 位（通过 *chmod g+s*），则文件的组 ID 会从父目录继承。这些规则在 表 15-3 中进行了总结。

### 注意

在 创建和删除目录：*mkdir()* 和 *rmdir()* 和 rmdir()") 中，我们将看到，当目录设置了设置组 ID 位时，创建该目录内的新子目录时，组 ID 位也会被设置。这样，主文中描述的设置组 ID 的行为会在整个目录树中传播。

表 15-3. 确定新创建文件的组所有权的规则

| 文件系统挂载选项 | 父目录是否启用设置组 ID 位？ | 新文件的组所有权来源于 |
| --- | --- | --- |
| *-o grpid*、*-o bsdgroups* | （忽略） | 父目录组 ID |
| *-o nogrpid*、*-o sysvgroups*（*默认*） | 否 | 进程有效组 ID |
| 是 | 父目录组 ID |

在写作时，唯一支持 *grpid* 和 *nogrpid* 挂载选项的文件系统是 *ext2*、*ext3*、*ext4* 和（自 Linux 2.6.14 起）*XFS*。其他文件系统遵循 *nogrpid* 规则。

### 更改文件所有权：*chown()*、*fchown()* 和 *lchown()*

*chown()*、*lchown()* 和 *fchown()* 系统调用会更改文件的所有者（用户 ID）和组（组 ID）。

```
#include <unistd.h>

int `chown`(const char **pathname*, uid_t *owner*, gid_t *group*);
int `lchown`(const char **pathname*, uid_t *owner*, gid_t *group*);
int `fchown`(int *fd*, uid_t *owner*, gid_t *group*);
```

### 注意

成功时返回 0，错误时返回 -1

这三个系统调用之间的区别类似于 *stat()* 系列系统调用：

+   *chown()* 会更改 *pathname* 参数指定的文件的所有权；

+   *lchown()* 执行相同的操作，不同的是如果 *pathname* 是符号链接，则会更改链接文件的所有权，而不是链接所指向的文件的所有权；并且

+   *fchown()* 会更改由打开文件描述符 *fd* 引用的文件的所有权。

*owner* 参数指定文件的新用户 ID，*group* 参数指定文件的新组 ID。要只更改其中一个 ID，我们可以为另一个参数指定 -1，以保持该 ID 不变。

### 注意

在 Linux 2.2 之前，*chown()* 不会解除符号链接的引用。Linux 2.2 更改了 *chown()* 的语义，并新增了 *lchown()* 系统调用，以提供旧版 *chown()* 系统调用的行为。

只有特权 (`CAP_CHOWN`) 进程才能使用 *chown()* 更改文件的用户 ID。非特权进程可以使用 *chown()* 将其拥有的文件的组 ID 更改为它们是成员的任何组（即进程的有效用户 ID 与文件的用户 ID 匹配）。特权进程可以将文件的组 ID 更改为任何值。

如果更改了文件的所有者或组，则会关闭设置用户 ID 和设置组 ID 权限位。这是一种安全预防措施，以确保普通用户不能在可执行文件上启用设置用户 ID（或设置组 ID）位，然后通过某种方式使其成为特权用户（或组）的所有，进而在执行该文件时获得该特权身份。

### 注意

SUSv3 没有规定当超级用户更改可执行文件的所有者或组时，是否应关闭设置用户 ID 和设置组 ID 权限位。Linux 2.0 在这种情况下确实关闭了这些位，而早期的 2.2 内核（直到 2.2.12）则没有。之后的 2.2 内核恢复了 2.0 的行为，其中超级用户的更改与普通用户的行为相同，并且这种行为在随后的内核版本中得到了保持。（然而，如果我们在 *root* 登录下使用 *chown(1)* 命令更改文件的所有权，那么，在调用 *chown(2)* 后，*chown* 命令会使用 *chmod()* 系统调用重新启用设置用户 ID 和设置组 ID 权限位。）

当更改文件的所有者或组时，如果组执行权限位已经关闭，或者我们正在更改目录的所有权，则不会关闭设置组 ID 权限位。在这两种情况下，设置组 ID 位用于其他目的，而非创建设置组 ID 程序，因此不希望关闭该位。设置组 ID 位的其他用途如下：

+   如果关闭了组执行权限位，则设置组 ID 权限位将用于启用强制文件锁定（在强制锁定中讨论）。

+   在目录的情况下，设置组 ID 位用于控制在该目录中创建的新文件的所有权（新文件的所有权）。

*chown()* 的使用示例见于示例 15-2, 这是一个允许用户更改任意数量文件的所有者和组的程序，文件通过命令行参数指定。（该程序使用示例 8-1 中的 *userIdFromName()* 和 *groupIdFromName()* 函数，来自示例程序，将用户和组名转换为相应的数字 ID。）

示例 15-2. 更改文件的所有者和组

```
`files/t_chown.c`
#include <pwd.h>
#include <grp.h>
#include "ugid_functions.h"             /* Declarations of userIdFromName()
                                           and groupIdFromName() */
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    uid_t uid;
    gid_t gid;
    int j;
    Boolean errFnd;

    if (argc < 3 || strcmp(argv[1], "--help") == 0)
        usageErr("%s owner group [file...]\n"
                "        owner or group can be '-', "
                "meaning leave unchanged\n", argv[0]);

    if (strcmp(argv[1], "-") == 0) {            /* "-" ==> don't change owner */
        uid = -1;
    } else {                                    /* Turn user name into UID */
        uid = userIdFromName(argv[1]);
        if (uid == -1)
            fatal("No such user (%s)", argv[1]);
    }

    if (strcmp(argv[2], "-") == 0) {            /* "-" ==> don't change group */
        gid = -1;
    } else {                                    /* Turn group name into GID */
        gid = groupIdFromName(argv[2]);
        if (gid == -1)
            fatal("No group user (%s)", argv[1]);
    }

    /* Change ownership of all files named in remaining arguments */

    errFnd = FALSE;
    for (j = 3; j < argc; j++) {
        if (chown(argv[j], uid, gid) == -1) {
            errMsg("chown: %s", argv[j]);
            errFnd = TRUE;
        }
    }

    exit(errFnd ? EXIT_FAILURE : EXIT_SUCCESS);
}
     `files/t_chown.c`
```

## 文件权限

在本节中，我们描述了应用于文件和目录的权限方案。虽然我们主要讨论文件和目录的权限，但我们描述的规则适用于所有类型的文件，包括设备、FIFO 和 UNIX 域套接字。此外，System V 和 POSIX 进程间通信对象（共享内存、信号量和消息队列）也有权限掩码，适用于这些对象的规则与文件类似。

### 常规文件的权限

如检索文件信息：*stat()*")所述，*stat* 结构的 *st_mode* 字段的底部 12 位定义了文件的权限。这些位中的前三位是特殊位，称为设置用户 ID、设置组 ID 和粘滞位（分别在图 15-1 中标记为 U、G 和 T）。我们将在设置用户 ID、设置组 ID 和粘滞位中进一步讨论这些位。剩余的 9 位形成了定义授予不同类别用户访问文件权限的掩码。文件权限掩码将世界分为三类：

+   *所有者*（也称为 *用户*）：授予文件所有者的权限。

    ### 注意

    *用户* 这一术语由 *chmod(1)* 等命令使用，其中 *u* 是该权限类别的缩写。

+   *组*：授予文件所属组成员的权限。

+   *其他*：授予其他所有人的权限。

每个用户类别可以授予三种权限：

+   *读取*：文件的内容可以被读取。

+   *写入*：文件的内容可能会被更改。

+   *执行*：文件可以被执行（即，它是一个程序或脚本）。为了执行脚本文件（例如，*bash* 脚本），需要同时拥有读取和执行权限。

可以使用命令 *ls -l* 查看文件的权限和所有权，如以下示例所示：

```
$ `ls -l myscript.sh`
-`rwxr-x---`    1 mtk      users        1667 Jan 15 09:22 myscript.sh
```

在上述示例中，文件权限显示为 `rwxr-x---`（该字符串前的初始连字符表示文件类型：常规文件）。为了理解这个字符串，我们将这 9 个字符分成 3 组，每组分别表示是否启用了读、写和执行权限。第一组表示所有者的权限，所有者具有读、写和执行权限。接下来的组表示组的权限，组具有读和执行权限，但没有写权限。最后一组表示其他用户的权限，其他用户没有任何权限。

`<sys/stat.h>` 头文件定义了常量，可以与 *stat* 结构的 *st_mode* 使用 AND 运算符（`&`）进行比较，以检查是否设置了特定的权限位。（这些常量也通过包含 `<fcntl.h>` 被定义，`<fcntl.h>` 中原型化了 *open()* 系统调用。）这些常量显示在 表 15-4 中。

表 15-4. 文件权限位常量

| 常量 | 八进制值 | 权限位 |
| --- | --- | --- |
| `S_ISUID` | `04000` | 设置用户标识位 |
| `S_ISGID` | `02000` | 设置组标识位 |
| `S_ISVTX` | `01000` | Sticky 位 |
| `S_IRUSR` | `0400` | 用户可读 |
| `S_IWUSR` | `0200` | 用户可写 |
| `S_IXUSR` | `0100` | 用户可执行 |
| `S_IRGRP` | `040` | 组可读 |
| `S_IWGRP` | `020` | 组可写 |
| `S_IXGRP` | `010` | 组可执行 |
| `S_IROTH` | `04` | 其他用户可读 |
| `S_IWOTH` | `02` | 其他用户可写 |
| `S_IXOTH` | `01` | 其他用户可执行 |

除了 表 15-4 中显示的常量外，还定义了三个常量，用于表示所有者、组和其他用户的所有三种权限掩码：`S_IRWXU`（0700）、`S_IRWXG`（070）和 `S_IRWXO`（07）。

示例 15-3 中的头文件声明了一个函数，*filePermStr()*，该函数接收一个文件权限掩码，返回该掩码的静态分配字符串表示，格式与 *ls(1)* 中使用的格式相同。

示例 15-3. `file_perms.c` 的头文件

```
`files/file_perms.h`
#ifndef FILE_PERMS_H
#define FILE_PERMS_H

#include <sys/types.h>

#define FP_SPECIAL 1            /* Include set-user-ID, set-group-ID, and sticky
                                   bit information in returned string */

char *filePermStr(mode_t perm, int flags);

#endif
      `files/file_perms.h`
```

如果在 *filePermStr()* 函数的 *flags* 参数中设置了 `FP_SPECIAL` 标志，那么返回的字符串将包含设置的用户标识位、组标识位和 Sticky 位，再次以 *ls(1)* 的样式显示。

*filePermStr()* 函数的实现显示在 示例 15-4 中。我们在 示例 15-1 的程序中使用了这个函数。

示例 15-4. 将文件权限掩码转换为字符串

```
`files/file_perms.c`
#include <sys/stat.h>
#include <stdio.h>
#include "file_perms.h"                 /* Interface for this implementation */

#define STR_SIZE sizeof("rwxrwxrwx")

char *          /* Return ls(1)-style string for file permissions mask */
filePermStr(mode_t perm, int flags)
{
    static char str[STR_SIZE];

    snprintf(str, STR_SIZE, "%c%c%c%c%c%c%c%c%c",
        (perm & S_IRUSR) ? 'r' : '-', (perm & S_IWUSR) ? 'w' : '-',
        (perm & S_IXUSR) ?
            (((perm & S_ISUID) && (flags & FP_SPECIAL)) ? 's' : 'x') :
            (((perm & S_ISUID) && (flags & FP_SPECIAL)) ? 'S' : '-'),
        (perm & S_IRGRP) ? 'r' : '-', (perm & S_IWGRP) ? 'w' : '-',
        (perm & S_IXGRP) ?
            (((perm & S_ISGID) && (flags & FP_SPECIAL)) ? 's' : 'x') :
            (((perm & S_ISGID) && (flags & FP_SPECIAL)) ? 'S' : '-'),
        (perm & S_IROTH) ? 'r' : '-', (perm & S_IWOTH) ? 'w' : '-',
        (perm & S_IXOTH) ?
            (((perm & S_ISVTX) && (flags & FP_SPECIAL)) ? 't' : 'x') :
            (((perm & S_ISVTX) && (flags & FP_SPECIAL)) ? 'T' : '-'));

    return str;
}
      `files/file_perms.c`
```

### 目录权限

目录的权限结构与文件相同。然而，三种权限的含义不同：

+   *读取*: 可以列出目录的内容（即文件名列表）（例如，通过*ls*命令）。

    ### 注意

    如果实验验证目录读取权限位的操作，请注意，一些 Linux 发行版会将*ls*命令别名化，包含一些需要访问目录中文件 i 节点信息的标志（例如，*-F*），而这需要目录的执行权限。为了确保我们使用的是未经修改的`ls`命令，可以指定命令的完整路径名`(/bin/ls)`。

+   *写入*: 可以在目录中创建和删除文件。请注意，要删除文件本身，并不需要对文件本身拥有任何权限。

+   *执行*: 可以访问目录中的文件。目录上的执行权限有时也称为*搜索*权限。

访问文件时，需要对路径名中列出的所有目录具有执行权限。例如，读取文件`/home/mtk/x`需要对`/`、`/home`和`/home/mtk`具有执行权限（同时对文件`x`本身需要有读取权限）。如果当前工作目录是`/home/mtk/sub1`，我们访问相对路径`../sub2/x`时，需要对`/home/mtk`和`/home/mtk/sub2`具有执行权限（但不需要对`/`或`/home`有执行权限）。

对目录的读取权限只允许我们查看目录中文件名的列表。要访问目录中文件的内容或 i 节点信息，我们必须对目录具有执行权限。

反之，如果我们对目录具有执行权限，但没有读取权限，那么如果知道文件名，我们可以访问该目录中的文件，但无法列出目录中的其他文件名。这是控制公共目录内容访问的一种简单且常用的技术。

要在目录中添加或删除文件，我们需要对目录同时拥有执行和写入权限。

### 权限检查算法

每当我们在系统调用中指定路径名以访问文件或目录时，内核会检查文件权限。当系统调用的路径名包括目录前缀时，内核除了检查文件本身的所需权限外，还会检查该前缀中每个目录的执行权限。权限检查是使用进程的有效用户 ID、有效组 ID 和附加组 ID 进行的。（严格来说，在 Linux 中进行文件权限检查时，使用的是文件系统的用户和组 ID，而不是相应的有效 ID，如第 9.5 节所述。）

### 注意

一旦文件通过*open()*打开，之后的系统调用（如*read()*, *write()*, *fstat()*, *fcntl()*, 和 *mmap()*)不会对返回的文件描述符进行权限检查。

内核在检查权限时应用的规则如下：

1.  如果进程具有特权，则授予所有访问权限。

1.  如果进程的有效用户 ID 与文件的用户 ID（所有者）相同，则根据文件上的*所有者*权限授予访问权限。例如，如果文件权限掩码中的所有者读取权限位被启用，则授予读取访问权限；否则，拒绝读取访问权限。

1.  如果进程的有效组 ID 或任何进程附加组 ID 与文件的组 ID（组所有者）匹配，则根据文件上的*组*权限授予访问权限。

1.  否则，访问权限将根据文件上的*其他*权限授予。

### 注意

在内核代码中，上述测试实际上是这样构造的：只有在进程未通过其他测试获得所需权限时，才会执行检查该进程是否具有特权的测试。这是为了避免不必要地设置 `ASU` 进程会计标志，该标志表示进程使用了超级用户特权（进程会计）。

对所有者、组和其他权限的检查是按顺序进行的，一旦找到适用的规则，检查便停止。这可能会产生意想不到的后果：例如，如果组的权限超过所有者的权限，那么所有者实际上在文件上的权限会少于文件组的成员，以下例子说明了这一点：

```
$ `echo 'Hello world' > a.txt`
$ `ls -l a.txt`
-rw-r--r--   1 mtk    users    12 Jun 18 12:26 a.txt
$ `chmod u-rw a.txt`               *Remove read and write permission from owner*
$ `ls -l a.txt`
----r--r--   1 mtk    users    12 Jun 18 12:26 a.txt
$ `cat a.txt`
cat: a.txt: Permission denied    *Owner can no longer read file*
$ `su avr`                         *Become someone else...*
Password:
$ `groups`                         *who is in the group owning the file...*
users staff teach cs
$ `cat a.txt`                      *and thus can read the file*
Hello world
```

如果其他权限比所有者或组更多，类似的说明也适用。

由于文件权限和所有权信息保存在文件的 i 节点中，因此所有指向同一 i 节点的文件名（链接）共享这些信息。

Linux 2.6 提供了访问控制列表（ACLs），使得可以基于每个用户和每个组定义文件权限。如果文件具有 ACL，则使用修改后的算法来检查权限。我们在第十七章中描述了 ACL。

#### 对特权进程的权限检查

上述中，我们说过如果进程具有特权，则在检查权限时会授予所有访问权限。我们需要为这个说法添加一个前提条件。对于非目录文件，只有当文件的某个权限类别被授予执行权限时，Linux 才会授予特权进程执行权限。在某些其他 UNIX 实现中，即使没有任何权限类别授予执行权限，特权进程也能执行文件。当访问目录时，特权进程始终会被授予执行（搜索）权限。

### 注意

我们可以用两个 Linux 进程能力来重新表述特权进程：`CAP_DAC_READ_SEARCH` 和 `CAP_DAC_OVERRIDE`（Linux 能力）。具有 `CAP_DAC_READ_SEARCH` 能力的进程始终对任何类型的文件具有读取权限，并且始终对目录具有读取和执行权限（即始终可以访问目录中的文件并读取目录中的文件列表）。具有 `CAP_DAC_OVERRIDE` 能力的进程始终对任何类型的文件具有读取和写入权限，如果文件是目录，或者文件的某个权限类别被授予执行权限，则还具有执行权限。

### 检查文件可访问性：*access()*

正如在 权限检查算法中所述，*有效*用户 ID 和组 ID，以及附加组 ID，决定了进程访问文件时的权限。程序（例如，设置用户 ID 或设置组 ID 的程序）也可以根据进程的*真实*用户 ID 和组 ID 来检查文件的可访问性。

*access()* 系统调用根据进程的真实用户 ID 和组 ID（以及附加组 ID）检查指定的*pathname*文件的可访问性。

```
#include <unistd.h>

int `access`(const char **pathname*, int *mode*);
```

### 注意

如果所有权限都被授予，则返回 0；否则返回 -1。

如果*pathname*是符号链接，*access()* 会取消引用它。

*mode* 参数是一个位掩码，包含一个或多个常量，参见 表 15-5 的模式常量")，通过 OR 运算（`|`）组合。如果在*pathname*上授予*mode*指定的所有权限，则*access()* 返回 0；如果至少有一个请求的权限不可用（或发生错误），则*access()* 返回 -1。

表 15-5. *access()*的*mode*常量

| 常量 | 描述 |
| --- | --- |
| `F_OK` | 文件是否存在？ |
| `R_OK` | 文件是否可以读取？ |
| `W_OK` | 文件是否可以写入？ |
| `X_OK` | 文件是否可以执行？ |

从*access()*调用到随后的文件操作之间的时间间隔意味着，*access()*返回的信息不能保证在稍后的操作时仍然有效（无论间隔多短）。这种情况可能在某些应用程序设计中导致安全漏洞。

假设例如我们有一个设置用户 ID 为*root*的程序，该程序使用*access()*检查文件是否对程序的真实用户 ID 可访问，如果可以，则对文件执行操作（例如，*open()* 或 *exec()*）。

问题在于，如果传递给*access()*的路径名是符号链接，并且恶意用户在第二步之前成功地修改了该链接，使其指向一个不同的文件，那么设置用户 ID-*root*的程序可能会在一个真实用户 ID 没有权限的文件上操作。（这是第 38.6 节中描述的检查时与使用时条件竞争的一个例子。）因此，推荐的做法是完全避免使用*access()*（例如，参见[Borisov, 2005]）。在刚才给出的例子中，我们可以通过临时更改设置用户 ID 进程的有效（或文件系统）用户 ID，尝试执行所需的操作（例如，*open()*或*exec()*），然后检查返回值和*errno*来判断操作是否因权限问题而失败。

### 注意

GNU C 库提供了一个类似的非标准函数，*euidaccess()*（或同义词，*eaccess()*），它使用进程的有效用户 ID 来检查文件访问权限。

### 设置用户 ID、设置组 ID 和粘滞位

除了用于所有者、组和其他权限的 9 位外，文件权限掩码还包含 3 个位，这些位分别是*设置用户 ID*（位 04000）、*设置组 ID*（位 02000）和*粘滞*（位 01000）位。我们已经在第 9.3 节中讨论了使用设置用户 ID 和设置组 ID 权限位来创建特权程序的情况。设置组 ID 位还具有两个其他用途，我们将在其他地方描述：控制在挂载了*nogrpid*选项的目录中新创建文件的组所有权（新文件的所有权），以及启用文件的强制锁定（强制锁定）。在本节的其余部分，我们将讨论*粘滞位*的使用。

在旧版本的 UNIX 实现中，粘滞位被用作一种使常用程序运行更快的方式。如果在程序文件上设置了粘滞位，那么程序第一次执行时，程序文本会被保存在交换区——因此它在交换区中*粘滞*，并且在随后的执行中加载更快。现代 UNIX 实现具有更复杂的内存管理系统，已使得粘滞权限位的这一用途过时。

### 注意

表 15-4 中显示的粘滞权限位常量的名称`S_ISVTX`，源自粘滞位的另一种名称：*保存文本*位。

在现代 UNIX 实现（包括 Linux）中，粘滞权限位有一个完全不同的用途。对于目录，粘滞位作为 *受限删除* 标志。将这个位设置在目录上意味着，未经授权的进程只能在具有目录写权限 *且* 拥有文件或目录的情况下，才能在目录中执行 unlink（*unlink()*, *rmdir()*）和重命名（*rename()*）操作。（具有 `CAP_FOWNER` 能力的进程可以绕过后者的所有权检查。）这使得可以创建一个多个用户共享的目录，每个用户都可以在该目录中创建和删除自己的文件，但不能删除其他用户拥有的文件。出于这个原因，粘滞权限位通常会被设置在 `/tmp` 目录上。

文件的粘滞权限位是通过 *chmod* 命令（*chmod +t file*）或 *chmod()* 系统调用设置的。如果一个文件的粘滞位被设置，*ls -l* 会在其他执行权限字段中显示一个小写或大写字母 *T*，具体取决于其他执行权限位是否开启，如下所示：

```
$ `touch tfile`
$ `ls -l tfile`
-rw-r--r--   1 mtk    users     0 Jun 23 14:44 tfile
$ `chmod +t tfile`
$ `ls -l tfile`
-rw-r--r-T   1 mtk    users     0 Jun 23 14:44 tfile
$ `chmod o+x tfile`
$ `ls -l tfile`
-rw-r--r-t   1 mtk    users     0 Jun 23 14:44 tfile
```

### 进程文件模式创建掩码：*umask()*

现在我们更详细地讨论新创建的文件或目录上的权限。对于新文件，内核使用 *open()* 或 *creat()* 中的 *mode* 参数指定的权限。对于新目录，权限是根据 *mkdir()* 的 *mode* 参数设置的。然而，这些设置会被文件模式创建掩码修改，也就是通常所说的 *umask*。umask 是一个进程属性，指定在进程创建新文件或目录时，哪些权限位应该始终被关闭。

通常，一个进程只使用它从父 shell 继承的 umask，结果是（通常是期望的）用户可以通过 shell 内建命令*umask*来控制从 shell 执行的程序的 umask，而这个命令会改变 shell 进程的 umask。

大多数 shell 的初始化文件将默认 umask 设置为八进制值 022（`----w--w-`）。这个值指定应该始终关闭组和其他用户的写权限。因此，假设在调用 *open()* 时 *mode* 参数为 0666（即所有用户均可读写，这是典型值），则新文件会被创建为所有者具有读写权限，而其他用户仅具有读权限（*ls -l* 显示为 `rw-r--r--`）。相应地，假设对 *mkdir()* 的 *mode* 参数指定为 0777（即为所有用户授予所有权限），则新目录会被创建为所有者具有所有权限，而组和其他用户仅具有读和执行权限（即 `rwxr-xr-x`）。

*umask()* 系统调用将进程的 umask 更改为 *mask* 中指定的值。

```
#include <sys/stat.h>

mode_t `umask`(mode_t *mask*);
```

### 注意

总是成功地返回之前进程的 umask

*mask* 参数可以作为八进制数字指定，或者通过按位或（`|`）将表 15-4 中列出的常量结合在一起。

对 *umask()* 的调用总是成功，并返回先前的 umask 值。

示例 15-5")展示了如何将 *umask()* 与 *open()* 和 *mkdir()* 一起使用。当我们运行该程序时，我们会看到以下结果：

```
$ `./t_umask`
Requested file perms: rw-rw----             *This is what we asked for*
Process umask:        ----wx-wx             *This is what we are denied*
Actual file perms:    rw-r-----             *So this is what we end up with*

Requested dir. perms: rwxrwxrwx
Process umask:        ----wx-wx
Actual dir. perms:    rwxr--r--
```

### 注意

在示例 15-5")中，我们使用 *mkdir()* 和 *rmdir()* 系统调用来创建和删除目录，并使用 *unlink()* 系统调用来删除文件。我们在第十八章中描述了这些系统调用。

示例 15-5. 使用 *umask()*

```
`files/t_umask.c`
#include <sys/stat.h>
#include <fcntl.h>
#include "file_perms.h"
#include "tlpi_hdr.h"

#define MYFILE "myfile"
#define MYDIR  "mydir"
#define FILE_PERMS    (S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP)
#define DIR_PERMS     (S_IRWXU | S_IRWXG | S_IRWXO)
#define UMASK_SETTING (S_IWGRP | S_IXGRP | S_IWOTH | S_IXOTH)

int
main(int argc, char *argv[])
{
    int fd;
    struct stat sb;
    mode_t u;

    umask(UMASK_SETTING);

    fd = open(MYFILE, O_RDWR | O_CREAT | O_EXCL, FILE_PERMS);
    if (fd == -1)
        errExit("open-%s", MYFILE);
    if (mkdir(MYDIR, DIR_PERMS) == -1)
        errExit("mkdir-%s", MYDIR);

    u = umask(0);               /* Retrieves (and clears) umask value */

    if (stat(MYFILE, &sb) == -1)
        errExit("stat-%s", MYFILE);
    printf("Requested file perms: %s\n", filePermStr(FILE_PERMS, 0));
    printf("Process umask:        %s\n", filePermStr(u, 0));
    printf("Actual file perms:    %s\n\n", filePermStr(sb.st_mode, 0));

    if (stat(MYDIR, &sb) == -1)
        errExit("stat-%s", MYDIR);
    printf("Requested dir. perms: %s\n", filePermStr(DIR_PERMS, 0));
    printf("Process umask:        %s\n", filePermStr(u, 0));
    printf("Actual dir. perms:    %s\n", filePermStr(sb.st_mode, 0));

    if (unlink(MYFILE) == -1)
        errMsg("unlink-%s", MYFILE);
    if (rmdir(MYDIR) == -1)
        errMsg("rmdir-%s", MYDIR);
    exit(EXIT_SUCCESS);
}
     `files/t_umask.c`
```

### 更改文件权限：*chmod()* 和 *fchmod()*

*chmod()* 和 *fchmod()* 系统调用更改文件的权限。

```
#include <sys/stat.h>

int `chmod`(const char **pathname*, mode_t *mode*);
int `fchmod`(int *fd*, mode_t *mode*);
```

### 注意

两者在成功时返回 0，出错时返回 -1。

*chmod()* 系统调用更改在 *pathname* 中指定的文件的权限。如果该参数是符号链接，*chmod()* 更改其指向的文件的权限，而不是链接本身的权限。（符号链接始终以所有用户具有读、写和执行权限的方式创建，这些权限无法更改。在取消引用链接时，这些权限会被忽略。）

*fchmod()* 系统调用更改由打开文件描述符 *fd* 引用的文件的权限。

*mode* 参数指定文件的新权限，可以是数字（八进制）或通过按位或（|）运算得到的权限位掩码，这些位列在表 15-4 中。为了更改文件的权限，进程必须具备相应的权限（`CAP_FOWNER`），或者其有效用户 ID 必须与文件的所有者（用户 ID）匹配。（严格来说，在 Linux 上，对于非特权进程，必须是进程的文件系统用户 ID，而不是有效用户 ID，必须与文件的用户 ID 匹配，详情请见第 9.5 节。）

要设置文件的权限，使得所有用户仅被授予读取权限，我们可以使用以下调用：

```
if (chmod("myfile", S_IRUSR | S_IRGRP | S_IROTH) == -1)
    errExit("chmod");
/* Or equivalently: chmod("myfile", 0444); */
```

为了修改文件权限的特定位，我们首先使用 *stat()* 检索现有的权限，调整我们想要更改的位，然后使用 *chmod()* 更新权限：

```
struct stat sb;
mode_t mode;

if (stat("myfile", &sb) == -1)
    errExit("stat");
mode = (sb.st_mode | S_IWUSR) & ~S_IROTH;
       /* owner-write on, other-read off, remaining bits unchanged */
if (chmod("myfile", mode) == -1)
    errExit("chmod");
```

上述操作等同于以下 shell 命令：

```
$ `chmod u+w,o-r myfile`
```

在 新文件的所有权 中，我们指出，如果一个目录位于使用 *-o bsdgroups* 选项挂载的 *ext2* 系统上，或者位于使用 *-o sysvgroups* 选项挂载的系统上，并且该目录的设置组 ID 权限位已开启，那么在该目录中新创建的文件将继承父目录的所有权，而不是创建进程的有效组 ID。可能出现这样的情况：该文件的组 ID 与创建进程的任何组 ID 不匹配。为此，当一个非特权进程（没有 `CAP_FSETID` 权限）对一个组 ID 不等于其有效组 ID 或任何补充组 ID 的文件调用 *chmod()*（或 *fchmod()*）时，内核会始终清除设置组 ID 权限位。这是一项安全措施，旨在防止用户为自己不属于的组创建一个设置组 ID 的程序。以下的 shell 命令展示了这种措施所阻止的尝试性漏洞攻击：

```
$ `mount | grep test`                *Hmmm,* /test *is mounted with -o bsdgroups*
/dev/sda9 on /test type ext3 (rw,bsdgroups)
$ `ls -ld /test`                     *Directory has GID root, writable by anyone*
drwxrwxrwx   3 root   root    4096 Jun 30 20:11 /test
$ `id`                               *I'm an ordinary user, not part of root group*
uid=1000(mtk) gid=100(users) groups=100(users),101(staff),104(teach)
$ `cd /test`
$ `cp ~/myprog .`                    *Copy some mischievous program here*
$ `ls -l myprog`                     *Hey! It's in the root group!*
-rwxr-xr-x   1 mtk    root   19684 Jun 30 20:43 myprog
$ `chmod g+s myprog`                 *Can I make it set-group-ID to root?*
$ `ls -l myprog`                     *Hmm, no...*
-rwxr-xr-x   1 mtk    root   19684 Jun 30 20:43 myprog
```

## i-node 标志（*ext2* 扩展文件属性）

一些 Linux 文件系统允许在文件和目录上设置各种 *i-node 标志*。这个功能是一个非标准的 Linux 扩展。

### 注意

现代 BSD 系统提供了类似于 i-node 标志的功能，通过使用 *chflags(1)* 和 *chflags(2)* 设置文件标志。

第一个支持 i-node 标志的 Linux 文件系统是 *ext2*，这些标志有时被称为 *ext2 扩展文件属性*。随后，i-node 标志的支持被添加到其他文件系统中，包括 *Btrfs*、*ext3*、*ext4*、*Reiserfs*（自 Linux 2.4.19 起）、*XFS*（自 Linux 2.4.25 和 2.6 起）、*JFS*（自 Linux 2.6.17 起）。

### 注意

各个文件系统支持的 i-node 标志范围有所不同。为了在 *Reiserfs* 文件系统上使用 i-node 标志，我们必须在挂载文件系统时使用 *mount -o attrs* 选项。

从 shell 中，可以使用 *chattr* 和 *lsattr* 命令设置和查看 i-node 标志，如下例所示：

```
$ `lsattr myfile`
-------- myfile
$ `chattr +ai myfile`               *Turn on Append Only and Immutable flags*
$ `lsattr myfile`
----ia-- myfile
```

在程序中，可以使用 *ioctl()* 系统调用来检索和修改 i-node 标志，稍后将详细介绍。

i-node 标志可以设置在常规文件和目录上。大多数 i-node 标志是用于常规文件的，尽管其中一些也（或仅）对目录有意义。表 15-6 总结了可用的 i-node 标志范围，显示了相应的标志名称（在 `<linux/fs.h>` 中定义），该名称在程序中的 *ioctl()* 调用中使用，以及与 *chattr* 命令一起使用的选项字母。

### 注意

在 Linux 2.6.19 之前，表 15-6 中显示的 `FS_*` 常量未在 `<linux/fs.h>` 中定义。相反，有一组特定于文件系统的头文件定义了特定于文件系统的常量名称，所有这些常量的值相同。因此，*ext2* 有 `EXT2_APPEND_FL`，在 `<linux/ext2_fs.h>` 中定义；*Reiserfs* 有 `REISERFS_APPEND_FL`，在 `<linux/reiser_fs.h>` 中定义，值相同；依此类推。由于每个头文件都以相同的值定义了相应的常量，在较旧的系统中，如果没有提供 `<linux/fs.h>` 中的定义，可以包含任何头文件并使用特定于文件系统的名称。

表 15-6. I 节点标志

| 常量 | *chattr* 选项 | 目的 |
| --- | --- | --- |
| `FS_APPEND_FL` | `a` | 仅追加（需要特权） |
| `FS_COMPR_FL` | `c` | 启用文件压缩（未实现） |
| `FS_DIRSYNC_FL` | `D` | 同步目录更新（自 Linux 2.6 起） |
| `FS_IMMUTABLE_FL` | `i` | 不可变（需要特权） |
| `FS_JOURNAL_DATA_FL` | `j` | 启用数据日志（需要特权） |
| `FS_NOATIME_FL` | `A` | 不更新文件最后访问时间 |
| `FS_NODUMP_FL` | `d` | 不进行转储 |
| `FS_NOTAIL_FL` | `t` | 不进行尾部打包 |
| `FS_SECRM_FL` | `s` | 安全删除（未实现） |
| `FS_SYNC_FL` | `S` | 同步文件（和目录）更新 |
| `FS_TOPDIR_FL` | `T` | 将其视为 Orlov 的顶级目录（自 Linux 2.6 起） |
| `FS_UNRM_FL` | `u` | 文件可以恢复删除（未实现） |

各种 `FL_*` 标志及其含义如下：

`FS_APPEND_FL`

只有在指定了 `O_APPEND` 标志的情况下，文件才能以写入模式打开（从而强制所有文件更新都追加到文件的末尾）。例如，可以为日志文件使用此标志。只有具有特权的 (`CAP_LINUX_IMMUTABLE`) 进程才能设置此标志。

`FS_COMPR_FL`

将文件以压缩格式存储在磁盘上。这个功能不是任何主要本地 Linux 文件系统的标准部分。（有些软件包实现了该功能，可以用于 *ext2* 和 *ext3*。）考虑到磁盘存储成本较低，压缩和解压缩所需的 CPU 开销，以及压缩文件意味着文件内容无法简单地通过 *lseek()* 随机访问，文件压缩对于许多应用来说是不可取的。

`FS_DIRSYNC_FL`（自 Linux 2.6 起）

使目录更新（例如，*open(pathname, O_CREAT)*、*link()*、*unlink()*和*mkdir()*)同步。这类似于第 13.3 节中描述的同步文件更新机制。与同步文件更新一样，同步目录更新会产生性能影响。此设置只能应用于目录。（在挂载文件系统：*mount()*")中描述的`MS_DIRSYNC`挂载标志提供类似的功能，但作用范围为每个挂载。）

`FS_IMMUTABLE_FL`

使文件不可变。文件数据无法更新（如*write()*和*truncate()*），并且元数据更改被阻止（例如，*chmod()*、*chown()*、*unlink()*、*link()*、*rename()*、*rmdir()*、*utime()*、*setxattr()*和*removexattr()*）。只有具有特权的（`CAP_LINUX_IMMUTABLE`）进程可以为文件设置此标志。当此标志设置后，即使是具有特权的进程也无法更改文件内容或元数据。

`FS_JOURNAL_DATA_FL`

启用数据日志记录。此标志仅在*ext3*和*ext4*文件系统上受支持。这些文件系统提供三种日志记录级别：*journal*、*ordered*和*writeback*。所有模式都会记录文件元数据的更新，但*journal*模式还会记录文件数据的更新。在以*ordered*或*writeback*模式进行日志记录的文件系统上，具有特权的`(CAP_SYS_RESOURCE)`进程可以通过设置此标志来按文件启用数据更新的日志记录。（*mount(8)*手册页描述了*ordered*和*writeback*模式之间的区别。）

`FS_NOATIME_FL`

访问文件时不更新文件的最后访问时间。这样可以避免每次访问文件时更新文件的 i 节点，从而提高 I/O 性能（请参见挂载文件系统：*mount()*")中对`MS_NOATIME`标志的描述）。 

`FS_NODUMP_FL`

不将此文件包含在使用*dump(8)*进行的备份中。此标志的效果取决于*dump(8)*手册页中描述的*-h*选项。

`FS_NOTAIL_FL`

禁用尾部打包。此标志仅在*Reiserfs*文件系统上受支持。它禁用*Reiserfs*尾部打包功能，该功能尝试将小文件（以及较大文件的最终碎片）打包到与文件元数据相同的磁盘块中。可以通过使用*mount -o notail*选项挂载整个*Reiserfs*文件系统来禁用尾部打包。

`FS_SECRM_FL`

安全删除文件。此未实现的功能的预期目的是，在删除时，文件会被安全删除，即首先进行覆盖，以防止磁盘扫描程序读取或重新创建它。（真正安全删除的问题相当复杂：实际上可能需要在磁介质上进行多次写入才能安全地擦除之前记录的数据；请参见[Gutmann, 1996]。）

`FS_SYNC_FL`

使文件更新变为同步。当此标志应用于文件时，它会使文件的写入操作变为同步（就像在所有打开该文件的操作中指定了 `O_SYNC` 标志一样）。当应用于目录时，此标志与上述的同步目录更新标志具有相同效果。

`FS_TOPDIR_FL`（自 Linux 2.6 起）

该标志标记一个目录，在 *Orlov* 块分配策略下进行特殊处理。Orlov 策略是对 *ext2* 块分配策略的一个受 BSD 启发的修改，旨在提高相关文件（例如，同一目录中的文件）在磁盘上的靠近程度，从而提高磁盘寻址时间。详细内容参见 [Corbet, 2002] 和 [Kumar, et al. 2008]。`FS_TOPDIR_FL` 仅对 *ext2* 及其后代 *ext3* 和 *ext4* 有效。

`FS_UNRM_FL`

允许在文件被删除后恢复（未删除）该文件。此功能尚未实现，因为可以在内核外部实现文件恢复机制。

通常，当 i-node 标志应用于目录时，它们会被该目录中新创建的文件和子目录自动继承。但此规则有一些例外：

+   `FS_DIRSYNC_FL` (*chattr +D*) 标志只能应用于目录，它只会被该目录中新创建的子目录继承。

+   当 `FS_IMMUTABLE_FL` (*chattr +i*) 标志应用于目录时，它不会被该目录内创建的文件和子目录继承，因为此标志会阻止向目录中添加新条目。

在程序中，可以使用 *ioctl()* 的 `FS_IOC_GETFLAGS` 和 `FS_IOC_SETFLAGS` 操作来检索和修改 i-node 标志。（这些常量在 `<linux/fs.h>` 中定义。）以下代码展示了如何启用文件描述符 *fd* 所指向的文件的 `FS_NOATIME_FL` 标志：

```
int attr;

if (ioctl(fd, FS_IOC_GETFLAGS, &attr) == -1)    /* Fetch current flags */
    errExit("ioctl");
attr |= FS_NOATIME_FL;
if (ioctl(fd, FS_IOC_SETFLAGS, &attr) == -1)    /* Update flags */
    errExit("ioctl");
```

要更改文件的 i-node 标志，进程的有效用户 ID 必须与文件的用户 ID（所有者）匹配，或者进程必须具有特权（`CAP_FOWNER`）。 （严格来说，在 Linux 中，对于一个非特权进程，必须是进程的文件系统用户 ID，而不是其有效用户 ID，必须与文件的用户 ID 匹配，具体见第 9.5 节。）

## 总结

*stat()* 系统调用检索有关文件的信息（元数据），其中大部分来自文件的 i-node。这些信息包括文件所有权、文件权限和文件时间戳。

程序可以使用 *utime()*、*utimes()* 以及各种类似接口来更新文件的最后访问时间和最后修改时间。

每个文件都有一个关联的用户 ID（所有者）和组 ID，以及一组权限位。就权限而言，文件用户分为三类：*所有者*（也称为 *用户*）、*组* 和 *其他*。每类用户可以授予三种权限：*读取*、*写入* 和 *执行*。目录也使用相同的方案，尽管权限位的含义略有不同。*chown()* 和 *chmod()* 系统调用可以更改文件的所有权和权限。*umask()* 系统调用设置一个权限位掩码，该掩码在调用进程创建文件时始终关闭。

文件和目录有三个附加的权限位。set-user-ID 和 set-group-ID 权限位可以应用于程序文件，从而创建使执行过程通过假定不同的有效用户或组身份（即程序文件的身份）来获得特权的程序。对于使用 *nogrpid*（*sysvgroups*）选项挂载的文件系统上的目录，set-group-ID 权限位可用于控制目录中新创建的文件是继承其进程的有效组 ID 还是继承父目录的组 ID。当应用于目录时，粘滞权限位充当受限删除标志。

I-node 标志控制文件和目录的各种行为。尽管最初是为 *ext2* 定义的，但这些标志现在在多个其他文件系统中得到支持。

## 练习题

1.  文件权限包含了关于各种文件系统操作所需权限的多个声明。使用 shell 命令或编写程序来验证或回答以下问题：

    1.  移除文件所有者的所有权限会拒绝文件所有者的访问权限，即使组用户和其他用户仍然有权限访问。

    1.  在一个具有读取权限但没有执行权限的目录中，可以列出目录中的文件名，但无法访问这些文件，无论它们的权限如何。

    1.  创建新文件、打开文件进行读取、打开文件进行写入以及删除文件，需要在父目录和文件本身上具有哪些权限？重命名文件时，源目录和目标目录需要哪些权限？如果重命名操作的目标文件已存在，目标文件需要哪些权限？设置目录的粘滞权限位（*chmod +t*）如何影响重命名和删除操作？

1.  你是否期望文件的三个时间戳中的任何一个会被 *stat()* 系统调用修改？如果没有，解释原因。

1.  在运行 Linux 2.6 的系统上，修改 示例 15-1 中的程序（`t_stat.c`），使文件的时间戳能够以纳秒精度显示。

1.  *access()* 系统调用使用进程的实际用户和组 ID 来检查权限。编写一个相应的函数，根据进程的有效用户和组 ID 来执行权限检查。

1.  如进程文件模式创建掩码：*umask()*")中所述，*umask()* 始终设置进程的 umask，并同时返回旧 umask 的副本。我们如何在不改变当前进程 umask 的情况下，获取当前进程 umask 的副本？

1.  *chmod a+rX file* 命令为所有类别的用户启用读取权限，并且如果 *file* 是一个目录，或者为 *file* 的任何用户类别启用了执行权限，也会启用所有类别的用户的执行权限，具体如下所示：

    ```
    $ `ls -ld dir file prog`
    dr--------  2 mtk users    48 May  4 12:28 dir
    -r--------  1 mtk users 19794 May  4 12:22 file
    -r-x------  1 mtk users 19336 May  4 12:21 prog
    $ `chmod a+rX dir file prog`
    $ `ls -ld dir file prog`
    dr-xr-xr-x  2 mtk users    48 May  4 12:28 dir
    -r--r--r--  1 mtk users 19794 May  4 12:22 file
    -r-xr-xr-x  1 mtk users 19336 May  4 12:21 prog
    ```

    编写一个程序，使用 *stat()* 和 *chmod()* 执行等同于 *chmod a+rX* 的操作。

1.  编写一个简单版本的 *chattr(1)* 命令，用于修改文件的 i-node 标志。有关 *chattr* 命令行界面的详细信息，请参见 *chattr(1)* 手册页。（你不需要实现 *-R*、*-V* 和 *-v* 选项。）
