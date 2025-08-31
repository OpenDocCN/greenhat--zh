## 第十八章 目录和链接

在本章中，我们通过研究目录和链接来结束对文件相关主题的讨论。在概述其实现之后，我们描述用于创建和删除目录及链接的系统调用。接着，我们研究允许程序扫描单个目录内容并遍历（即检查目录树中每个文件）目录树的库函数。

每个进程都有两个与目录相关的属性：根目录，它决定了绝对路径名的解释起点；当前工作目录，它决定了相对路径名的解释起点。我们将研究允许进程更改这两个属性的系统调用。

本章的结尾将讨论用于解析路径名并将其拆解为目录和文件名组件的库函数。

## 目录和（硬）链接

*目录*在文件系统中的存储方式与常规文件类似。有两点将目录与常规文件区分开：

+   目录在其 i 节点条目中标记为不同的文件类型（I 节点）。

+   目录是具有特殊组织方式的文件。本质上，它是一个由文件名和 i 节点号组成的表。

在大多数原生 Linux 文件系统中，文件名最长可以为 255 个字符。目录与 i 节点之间的关系如图 18-1 所示，图中展示了文件系统 i 节点表的部分内容以及为示例文件(`/etc/passwd`)维护的相关目录文件。

### 注意

尽管一个进程可以打开一个目录，但它不能使用*read()*读取目录的内容。要检索目录内容，进程必须使用本章后面讨论的系统调用和库函数。（在某些 UNIX 实现中，可以对目录执行*read()*，但这并不具有移植性。）进程也不能直接通过*write()*更改目录的内容；它只能通过系统调用间接（即请求内核）更改内容，如*open()*（创建新文件）、*link()*、*mkdir()*、*symlink()*、*unlink()*和*rmdir()*。（这些系统调用将在本章后面描述，除了*open()*，它在第 4.3 节中已描述。）

i 节点表是从 1 开始编号的，而不是从 0 开始，因为目录条目中的 i 节点字段为 0 表示该条目未使用。i 节点 1 用于记录文件系统中的坏块。文件系统的根目录（/）总是存储在 i 节点条目 2 中（如 图 18-1 所示），这样内核就知道从哪里开始解析路径名。

![文件 /etc/passwd 的 i 节点与目录结构的关系](img/18-1_DIRS_LINKS-inodes-dirs.png.jpg)图 18-1. 文件 `/etc/passwd` 的 i 节点与目录结构的关系

如果我们回顾存储在文件 i 节点中的信息列表（I 节点），会发现 i 节点不包含文件名；文件名仅在目录列表中的映射定义了文件的名称。这有一个有用的结果：我们可以在同一目录或不同目录中创建多个名称，每个名称都指向相同的 i 节点。这些多个名称被称为 *链接*，有时为了与符号链接区分开来，也称为 *硬链接*，后者将在稍后讨论。

### 注意

所有本地 Linux 和 UNIX 文件系统都支持硬链接。然而，许多非 UNIX 文件系统（例如，微软的 VFAT）不支持硬链接。（微软的 NTFS 文件系统支持硬链接。）

在 shell 中，我们可以使用 *ln* 命令为现有文件创建新的硬链接，如以下 shell 会话日志所示：

```
$ `echo -n 'It is good to collect things,' > abc`
$ ls -li abc
 122232 -rw-r--r--   1 mtk      users          29 Jun 15 17:07 abc
$ `ln abc xyz`
$ `echo ' but it is better to go on walks.' >> xyz`
$ `cat abc`
It is good to collect things, but it is better to go on walks.
$ `ls -li abc xyz`
 122232 -rw-r--r--   2 mtk      users          63 Jun 15 17:07 abc
 122232 -rw-r--r--   2 mtk      users          63 Jun 15 17:07 xyz
```

*ls -li* 命令显示的 i 节点号（作为第一列）确认了从 *cat* 命令输出中已经明确的内容：`abc` 和 `xyz` 指向相同的 i 节点条目，因此指向相同的文件。在 *ls -li* 命令显示的第三列中，我们可以看到该 i 节点的链接计数。在执行 *ln abc xyz* 命令后，`abc` 所指向的 i 节点的链接计数增加到了 2，因为现在有两个名称指向同一个文件。（对于文件 `xyz` 显示相同的链接计数，因为它指向相同的 i 节点。）

如果其中一个文件名被删除，另一个文件名和文件本身仍然存在：

```
$ `rm abc`
$ `ls -li xyz`
 122232 -rw-r--r--   1 mtk      users          63 Jun 15 17:07 xyz
```

文件的 i 节点条目和数据块仅在 i 节点的链接计数降到 0 时被删除（释放），即当文件的所有名称都被删除时。总结一下：*rm* 命令从目录列表中删除一个文件名，减少对应 i 节点的链接计数 1，并且如果链接计数因此降到 0，则释放该 i 节点及其指向的数据块。

文件的所有名称（链接）是等价的——没有任何名称（例如，第一个）优先于其他名称。正如我们在上面的例子中看到的那样，在文件的第一个名称被删除后，物理文件仍然存在，但它现在只能通过其他名称访问。

在线论坛中经常有人提问：“如何找到与程序中的文件描述符 X 关联的文件名？”简短的回答是我们无法做到——至少不能以可移植和明确的方式做到——因为文件描述符指向的是一个 i-node，而多个文件名（甚至如在创建和删除（硬）链接：*link*()和*unlink*()和 unlink()")中描述的那样，甚至没有文件名）可能指向这个 i-node。

### 注意

在 Linux 上，我们可以通过使用*readdir()*（读取目录：*opendir()*和*readdir()*和 readdir()")）扫描 Linux 特有的`/proc/`*PID*`/fd`目录的内容，来查看当前进程打开了哪些文件，该目录包含了进程当前打开的每个文件描述符的符号链接。*lsof(1)*和*fuser(1)*工具也已移植到许多 UNIX 系统上，在这方面也非常有用。

硬链接有两个限制，而这两个限制都可以通过使用符号链接来绕过：

+   由于目录条目（硬链接）是通过仅使用 i-node 编号来引用文件的，而 i-node 编号仅在文件系统内是唯一的，因此硬链接必须与它所指向的文件位于同一文件系统中。

+   硬链接不能创建到目录上。这是为了防止创建循环链接，这种链接会混淆许多系统程序。

### 注意

早期的 UNIX 实现允许超级用户创建指向目录的硬链接。这是必要的，因为这些实现没有提供*mkdir()*系统调用。相反，目录是通过*mknod()*创建的，然后为`.`和`..`条目创建了链接（[Vahalia, 1996]）。虽然这个特性现在不再需要，但一些现代 UNIX 实现保留了它以保持向后兼容。

使用绑定挂载（绑定挂载）可以实现类似于硬链接对目录的效果。

## 符号（软）链接

*符号链接*，有时也叫做*软链接*，是一种特殊的文件类型，其数据是另一个文件的名称。图 18-2 展示了两个硬链接，`/home/erena/this`和`/home/allyn/that`，它们指向同一个文件，而一个符号链接`/home/kiran/other`则指向名称`/home/erena/this`。

从 shell 中，可以使用*ln -s*命令创建符号链接。*ls -F*命令会在符号链接的末尾显示一个`@`字符。

符号链接所指向的路径可以是绝对路径，也可以是相对路径。相对符号链接是相对于链接本身的位置进行解释的。

符号链接不像硬链接那样具有相同的地位。特别是，符号链接不包括在它所指向的文件的链接计数中。（因此，图 18-2 中 i 节点 61 的链接计数为 2，而不是 3。）因此，如果符号链接所指向的文件名被删除，符号链接本身仍然存在，尽管它不能再被解引用（跟随）。我们称其为*悬空链接*。甚至可以创建指向在创建时不存在的文件名的符号链接。

### 注意

符号链接是由 4.2BSD 引入的。尽管它们没有包含在 POSIX.1-1990 中，但随后被纳入 SUSv1，并因此出现在 SUSv3 中。

![硬链接和符号链接的表示](img/18-2_DIRS_LINKS-links.png.jpg)图 18-2. 硬链接和符号链接的表示

由于符号链接是指向文件名，而不是 i 节点编号，因此可以用来链接到不同文件系统中的文件。符号链接也没有硬链接的其他限制：我们可以创建指向目录的符号链接。像*find*和*tar*这样的工具能够区分硬链接和符号链接，它们通常不会跟随符号链接，或者避免陷入由符号链接创建的循环引用。

可以链接符号链接（例如，`a`是指向`b`的符号链接，`b`是指向`c`的符号链接）。当在各种与文件相关的系统调用中指定符号链接时，内核会解引用这系列链接，以最终访问目标文件。

SUSv3 要求实现至少允许对路径名中的每个符号链接组件进行`_POSIX_SYMLOOP_MAX`次解引用。指定的`_POSIX_SYMLOOP_MAX`值为 8。然而，在内核版本 2.6.18 之前，Linux 在跟随符号链接链时，限制了解引用次数为 5 次。从内核版本 2.6.18 开始，Linux 实现了 SUSv3 指定的最小值 8 次解引用。Linux 还对整个路径名的解引用总数施加了 40 次限制。这些限制是为了防止极长的符号链接链和符号链接环路导致内核代码中的栈溢出问题，内核代码负责解析符号链接。

### 注意

一些 UNIX 文件系统执行了一项优化，主文中未提及，且未在图 18-2 中显示。当构成符号链接内容的字符串总长度足够小，能够适应通常用于数据指针的 i 节点部分时，该链接字符串会直接存储在那里。这样可以避免分配一个磁盘块，并且加速符号链接信息的访问，因为它与文件的 i 节点一起被检索。例如，*ext2*、*ext3*和*ext4*采用这种技术，将短符号链接字符串存储在通常用于数据块指针的 60 个字节内。实际上，这是一项非常有效的优化。在作者检查的一个系统中的 20,700 个符号链接中，97%的链接大小都在 60 字节以内。

#### 系统调用对符号链接的解释

许多系统调用会解引用（跟随）符号链接，因此它们作用于链接所指向的文件。一些系统调用则不会解引用符号链接，而是直接作用于链接文件本身。在每个系统调用的介绍中，我们都会描述它在符号链接方面的行为。该行为也在表 18-1 中进行了总结。

在少数情况下，当需要对符号链接所指向的文件和符号链接本身执行类似操作时，提供了替代系统调用：一个会解引用链接，另一个不会，后者以字母*l*为前缀；例如，*stat()*和*lstat()*。

一般来说，有一点适用：路径名中目录部分的符号链接（即，所有位于最后一个斜杠前的组件）总是会被解引用。因此，在路径名`/somedir/somesubdir/file`中，如果`somedir`和`somesubdir`是符号链接，它们将始终被解引用，而`file`则可能会被解引用，具体取决于传递路径名的系统调用。

### 注意

在相对于目录文件描述符的操作中，我们描述了一组在 Linux 2.6.16 中新增的系统调用，它们扩展了表 18-1 中所示的一些接口的功能。对于这些系统调用中的某些，跟随符号链接的行为可以通过调用中的*flags*参数进行控制。

#### 符号链接的文件权限和所有权

符号链接的所有权和权限对于大多数操作都被忽略（符号链接始终创建时会启用所有权限）。相反，链接所指向的文件的所有权和权限会在确定操作是否允许时起作用。符号链接的所有权只有在链接本身在具有粘性权限位的目录中被移除或重命名时才相关（用户标识符、组标识符和粘性位）。

## 创建和移除（硬）链接：*link*()和*unlink*()

*link()*和*unlink()*系统调用创建和移除硬链接。

```
#include <unistd.h>

int `link`(const char **oldpath*, const char **newpath*);
```

### 注意

成功时返回 0，出错时返回-1

表 18-1。 各种函数对符号链接的解释

| 函数 | 是否跟随链接？ | 备注 |
| --- | --- | --- |
| *access()* | • |   |
| *acct()* | • |   |
| *bind()* | • | UNIX 域套接字具有路径名 |
| *chdir()* | • |   |
| *chmod()* | • |   |
| *chown()* | • |   |
| *chroot()* | • |   |
| *creat()* | • |   |
| *exec()* | • |   |
| *getxattr()* | • |   |
| *lchown()* |   |   |
| *lgetxattr()* |   |   |
| *link()* |   | 见创建和移除（硬）链接：*link*()和*unlink*()和 unlink()") |
| *listxattr()* | • |   |
| *llistxattr()* |   |   |
| *lremovexattr()* |   |   |
| *lsetxattr()* |   |   |
| *lstat()* |   |   |
| *lutimes()* |   |   |
| *open()* | • | 除非指定了 O_NOFOLLOW 或 O_EXCL &#124; O_CREAT |
| *opendir()* | • |   |
| *pathconf()* | • |   |
| *pivot_root()* | • |   |
| *quotactl()* | • |   |
| *readlink()* |   |   |
| *removexattr()* | • |   |
| *rename()* |   | 链接在任一参数中都不会被跟随 |
| *rmdir()* |   | 如果参数是符号链接，则会失败并返回 ENOTDIR |
| *setxattr()* | • |   |
| *stat()* | • |   |
| *statfs(), statvfs()* | • |   |
| *swapon(), swapoff()* | • |   |
| *truncate()* | • |   |
| *unlink()* |   |   |
| *uselib()* | • |   |
| *utime(), utimes()* | • |   |

给定*oldpath*中的现有文件路径名，*link()*系统调用会使用*newpath*中指定的路径名创建一个新链接。如果*newpath*已经存在，则不会被覆盖；而是会返回一个错误（`EEXIST`）。

在 Linux 中，*link()* 系统调用不会解引用符号链接。如果 *oldpath* 是符号链接，则 *newpath* 会作为指向相同符号链接文件的新硬链接创建。（换句话说，*newpath* 也是指向与 *oldpath* 相同文件的符号链接。）这种行为不符合 SUSv3，后者规定所有执行路径名解析的函数应该解引用符号链接，除非另有说明（并且没有为 *link()* 指定任何例外）。大多数其他 UNIX 实现遵循 SUSv3 规定的行为。一个显著的例外是 Solaris，它默认提供与 Linux 相同的行为，但如果使用适当的编译器选项，则可以提供符合 SUSv3 的行为。这种实现之间的不一致的结果是，便携式应用程序应避免为 *oldpath* 参数指定符号链接。

### 注意

SUSv4 认识到现有实现之间的不一致，并规定是否 *link()* 解引用符号链接的选择由实现定义。SUSv4 还新增了 *linkat()* 的规范，它执行与 *link()* 相同的任务，但具有一个 *flags* 参数，可以用来控制调用是否解引用符号链接。有关更多细节，请参见 相对于目录文件描述符的操作。

```
#include <unistd.h>

int `unlink`(const char **pathname*);
```

### 注意

成功时返回 0，出错时返回 -1。

*unlink()* 系统调用删除一个链接（删除文件名），并且如果这是指向该文件的最后一个链接，它也会删除该文件本身。如果 *pathname* 中指定的链接不存在，则 *unlink()* 会因错误 `ENOENT` 而失败。

我们不能使用 *unlink()* 删除目录；这个任务需要 *rmdir()* 或 *remove()*，我们将在第 18.6 节中讨论它们。

### 注意

SUSv3 规定，如果 *pathname* 指定的是一个目录，那么 *unlink()* 应该因错误 `EPERM` 而失败。然而，在 Linux 中，在这种情况下，*unlink()* 会因错误 `EISDIR` 而失败。（LSB 明确允许这种偏离 SUSv3 的行为。）便携式应用程序应该准备好处理此情况的任何一个错误值。

*unlink()* 系统调用不会解引用符号链接。如果 *pathname* 是符号链接，则会删除该链接本身，而不是它指向的名称。

#### 只有当所有文件描述符都被关闭时，打开的文件才会被删除。

除了为每个 i-node 维护链接计数外，内核还会统计文件的打开文件描述符（参见图 5-2，以及复制文件描述符）。如果删除了文件的最后一个链接，而任何进程仍持有指向该文件的打开描述符，则文件在所有描述符关闭之前不会真正被删除。这是一个有用的特性，因为它允许我们在不需要担心其他进程是否打开该文件的情况下删除文件。（但是，我们不能将一个链接计数已降至 0 的打开文件重新附加到一个文件名。）此外，我们可以执行一些技巧，比如创建并打开一个临时文件，立即删除它，然后在程序中继续使用它，依赖于文件只有在我们关闭文件描述符时才会被销毁——无论是显式地，还是在程序退出时隐式地关闭。（这就是创建临时文件中描述的 *tmpfile()* 函数所做的。）

示例 18-1 删除链接")中的程序演示了即使删除了文件的最后一个链接，文件也只有在所有指向它的打开文件描述符关闭时才会被删除。

示例 18-1. 使用 *unlink()* 删除链接

```
`dirs_links/t_unlink.c`
#include <sys/stat.h>
#include <fcntl.h>
#include "tlpi_hdr.h"

#define CMD_SIZE 200
#define BUF_SIZE 1024

int
main(int argc, char *argv[])
{
    int fd, j, numBlocks;
    char shellCmd[CMD_SIZE];            /* Command to be passed to system() */
    char buf[BUF_SIZE];                 /* Random bytes to write to file */

    if (argc < 2 || strcmp(argv[1], "--help") == 0)
        usageErr("%s temp-file [num-1kB-blocks] \n", argv[0]);

    numBlocks = (argc > 2) ? getInt(argv[2], GN_GT_0, "num-1kB-blocks")
                           : 100000;

    fd = open(argv[1], O_WRONLY | O_CREAT | O_EXCL, S_IRUSR | S_IWUSR);
    if (fd == -1)
        errExit("open");

    if (unlink(argv[1]) == -1)          /* Remove filename */
        errExit("unlink");

    for (j = 0; j < numBlocks; j++)     /* Write lots of junk to file */
        if (write(fd, buf, BUF_SIZE) != BUF_SIZE)
            fatal("partial/failed write");

    snprintf(shellCmd, CMD_SIZE, "df -k `dirname %s`", argv[1]);
    system(shellCmd);                   /* View space used in file system */

    if (close(fd) == -1)                /* File is now destroyed */
        errExit("close");
    printf("********** Closed file descriptor\n");

    system(shellCmd);                   /* Review space used in file system */
    exit(EXIT_SUCCESS);
}
     `dirs_links/t_unlink.c`
```

示例 18-1 删除链接")中的程序接受两个命令行参数。第一个参数指定程序应该创建的文件名。程序打开此文件后立即删除该文件名。虽然文件名消失了，但文件本身仍然存在。程序接着向文件写入随机数据块。这些数据块的数量由程序的可选第二个命令行参数指定。此时，程序使用 *df(1)* 命令显示文件系统上已使用的空间。程序随后关闭文件描述符，此时文件被删除，并再次使用 *df(1)* 显示已使用的磁盘空间减少。以下的 shell 会话演示了示例 18-1 删除链接")中程序的使用：

```
`$ ./t_unlink /tmp/tfile 1000000`
Filesystem           1K-blocks      Used Available Use% Mounted on
/dev/sda10             5245020   3204044   2040976  62% /
********** Closed file descriptor
Filesystem           1K-blocks      Used Available Use% Mounted on
/dev/sda10             5245020   2201128   3043892  42% /
```

### 注意

在示例 18-1 删除链接")中，我们使用 *system()* 函数执行一个 shell 命令。我们在第 27.6 节中详细描述了 *system()*。

## 更改文件名：*rename()*

*rename()* 系统调用既可以用来重命名文件，也可以将文件移动到同一文件系统的其他目录中。

```
#include <stdio.h>

int `rename`(const char **oldpath*, const char **newpath*);
```

### 注意

成功时返回 0，错误时返回 -1

*oldpath*参数是一个现有路径名，它被重命名为*newpath*中给定的路径名。

*rename()*调用仅操作目录条目；它不会移动文件数据。重命名文件不会影响其他硬链接，也不会影响任何已打开文件描述符的进程，因为这些描述符指向的是打开文件描述，它们（在*open()*调用之后）与文件名没有关联。

使用*rename()*时适用以下规则：

+   如果*newpath*已经存在，则会被覆盖。

+   如果*newpath*和*oldpath*指向同一个文件，则不会进行任何更改（且调用成功）。这一点有点反直觉。根据前一点，我们通常期望如果两个文件名`x`和`y`存在，那么*rename*（`x`，`y`）调用会删除`x`。但如果`x`和`y`是指向同一文件的链接，情况则不同。

### 注意

这一规则的理由来自最初的 BSD 实现，可能是为了简化内核必须执行的检查，以保证像*rename(“x”, “x”)*、*rename(“x”, “./x”)*和*rename(“x”, “somedir/../x”)*这样的调用不会删除文件。

+   *rename()*系统调用不会取消引用其任何参数中的符号链接。如果*oldpath*是符号链接，则重命名的是符号链接。如果*newpath*是符号链接，则它被视为一个普通的路径名，将*oldpath*重命名为该路径名（即，现有的*newpath*符号链接会被删除）。

+   如果*oldpath*指向的是一个非目录文件，则*newpath*不能指定目录路径名（错误为`EISDIR`）。要将文件重命名为目录中的位置（即，将文件移到另一个目录），*newpath*必须包含新的文件名。以下调用会将文件移到不同的目录并更改其名称：

    ```
    rename("sub1/x", "sub2/y");
    ```

+   在*oldpath*中指定目录名称允许我们重命名该目录。在这种情况下，*newpath*要么不存在，要么必须是一个空目录的名称。如果*newpath*是一个现有文件或一个已存在且非空的目录，则会出现错误（分别为`ENOTDIR`和`ENOTEMPTY`）。

+   如果*oldpath*是一个目录，则*newpath*不能包含与*oldpath*相同的目录前缀。例如，我们不能将`/home/mtk`重命名为`/home/mtk/bin`（错误为`EINVAL`）。

+   *oldpath*和*newpath*所引用的文件必须位于同一文件系统中。这是因为目录是指向同一文件系统中 i 节点的硬链接列表。如前所述，*rename()*只是操作目录列表的内容。尝试将文件重命名到不同的文件系统会导致错误`EXDEV`。（要实现预期结果，我们必须将文件内容从一个文件系统复制到另一个文件系统，然后删除旧文件。这正是*mv*命令在这种情况下的操作。）

## 使用符号链接：*symlink()*和*readlink()*

我们现在来看看用于创建符号链接的系统调用，并检查它们的内容。

*symlink()* 系统调用创建一个新的符号链接 *linkpath*，指向 *filepath* 中指定的路径名。（要删除符号链接，我们使用 *unlink()*。）

```
#include <unistd.h>

int `symlink`(const char **filepath*, const char **linkpath*);
```

### 注意

成功时返回 0，失败时返回 -1。

如果 *linkpath* 中给定的路径名已经存在，则调用失败（并且 *errno* 设置为 `EEXIST`）。*filepath* 中指定的路径名可以是绝对路径或相对路径。

在调用时，*filepath* 中指定的文件或目录不需要存在。即使它在调用时存在，也没有任何东西能阻止它稍后被删除。在这种情况下，*linkpath* 变成了一个 *悬挂链接*，在其他系统调用中尝试解除引用它会导致错误（通常是 `ENOENT`）。

如果我们将符号链接指定为 *open()* 的 *pathname* 参数，它将打开链接所指向的文件。有时，我们更希望检索链接本身的内容——即它所指向的路径名。*readlink()* 系统调用执行此任务，将符号链接字符串的副本放入 *buffer* 指向的字符数组中。

```
#include <unistd.h>

ssize_t `readlink`(const char **pathname*, char **buffer*, size_t *bufsiz*);
```

### 注意

成功时返回放入 *buffer* 中的字节数，失败时返回 -1。

*bufsiz* 参数是一个整数，用来告诉 *readlink()* *buffer* 中可用的字节数。

如果没有发生错误，*readlink()* 将返回实际放入 *buffer* 中的字节数。如果链接的长度超过 *bufsiz*，则会将截断的字符串放入 *buffer* 中（并且 *readlink()* 返回该字符串的大小——即 *bufsiz*）。

因为 *buffer* 的末尾不会放置终止的空字节，所以无法区分 *readlink()* 返回的是被截断的字符串还是返回了恰好填满 *buffer* 的字符串。检查后一种情况的方法是重新分配更大的 *buffer* 数组并再次调用 *readlink()*。或者，我们可以使用 `PATH_MAX` 常量（在 系统限制 中描述）来确定 *pathname* 的大小，该常量定义了程序应能容纳的最长路径名长度。

我们在 示例 18-4 中演示了 *readlink()* 的使用。

### 注意

SUSv3 定义了一个新的限制 `SYMLINK_MAX`，实现应该定义该限制，以指示可以存储在符号链接中的最大字节数。该限制要求至少为 255 字节。截至本文写作时，Linux 并未定义此限制。正文中，我们建议使用 `PATH_MAX`，因为该限制应至少与 `SYMLINK_MAX` 一样大。

在 SUSv2 中，*readlink()* 的返回类型被指定为 *int*，许多当前的实现（以及 Linux 上较旧的 *glibc* 版本）遵循该规范。SUSv3 将返回类型更改为 *ssize_t*。

## 创建和删除目录：*mkdir()* 和 *rmdir()*

*mkdir()* 系统调用用于创建新目录。

```
#include <sys/stat.h>

int `mkdir`(const char **pathname*, mode_t *mode*);
```

### 注意

成功时返回 0，出错时返回-1

*pathname* 参数指定新目录的路径名。该路径名可以是相对路径或绝对路径。如果该路径名对应的文件已经存在，则调用将失败，并返回错误 `EEXIST`。

新目录的所有权将根据新文件的所有权中描述的规则进行设置。

*mode* 参数指定新目录的权限。（我们在新文件的所有权、更改文件所有权：*chown()*、*fchown()* 和 *lchown()*, fchown(), 和 lchown()")以及设置用户 ID、设置组 ID 和粘滞位中描述了目录权限位的含义。）该位掩码值可以通过按位或（`|`）将表 15-4 中的常量组合来指定，也可以像*open()*一样指定为八进制数。给定的 *mode* 值会与进程的 umask 进行按位与运算（进程文件模式创建掩码：*umask()*")）。此外，设置用户 ID 位（`S_ISUID`）总是被关闭，因为它对目录没有意义。

如果在 *mode* 中设置了粘滞位（`S_ISVTX`），则该位也会设置在新目录上。

在 *mode* 中设置组 ID 位（`S_ISGID`）将被忽略。相反，如果父目录上设置了组 ID 位，则新创建的目录也会设置该位。在新文件的所有权中，我们提到，在目录上设置组 ID 权限位会使得在该目录中创建的新文件继承目录的组 ID，而不是进程的有效组 ID。*mkdir()* 系统调用以此方式传播组 ID 权限位，以确保目录下的所有子目录共享相同的行为。

SUSv3 明确指出，*mkdir()* 如何处理设置用户 ID、设置组 ID 和粘滞位是由实现定义的。在某些 UNIX 实现中，这三个位在新目录上总是关闭的。

新创建的目录包含两个条目：`.`（点），它是指向该目录本身的链接，和`..`（点点），它是指向父目录的链接。

### 注意

SUSv3 不要求目录必须包含 `.` 和 `..` 项。它只要求在路径名中出现 `.` 和 `..` 时，正确解释它们。一个便携式应用程序不应依赖目录中存在这些项。  

*mkdir()* 系统调用仅创建 *pathname* 的最后一个组件。换句话说，调用 *mkdir(“aaa/bbb/ccc”, mode)* 仅在目录 `aaa` 和 `aaa/bbb` 已经存在时才会成功。（这与 *mkdir(1)* 命令的默认操作相对应，但 *mkdir(1)* 还提供了 *-p* 选项，以便在目录不存在时创建所有中间的目录名称。）  

### 注意  

GNU C 库提供了 *mkdtemp(template)* 函数，它是 *mkstemp()* 函数的目录类模拟。它创建一个唯一命名的目录，为所有者启用读、写和执行权限，且不允许其他用户拥有任何权限。与返回文件描述符不同，*mkdtemp()* 返回指向修改后的字符串的指针，该字符串包含实际的目录名称，存储在 *template* 中。SUSv3 未指定此函数，且并非所有 UNIX 实现都支持它；它在 SUSv4 中有所规定。  

*rmdir()* 系统调用删除指定的 *pathname* 目录，*pathname* 可以是绝对路径或相对路径。  

```
#include <unistd.h>

int `rmdir`(const char **pathname*);
```

### 注意  

成功时返回 0，出错时返回 -1  

为了使 *rmdir()* 成功，目录必须为空。如果 *pathname* 的最后一个组件是符号链接，则不会取消引用；相反，会返回错误 `ENOTDIR`。  

## 删除文件或目录：*remove()*  

*remove()* 库函数用于删除文件或空目录。  

```
#include <stdio.h>

int `remove`(const char **pathname*);
```

### 注意  

成功时返回 0，出错时返回 -1  

如果 *pathname* 是一个文件，*remove()* 会调用 *unlink()*；如果 *pathname* 是一个目录，*remove()* 会调用 *rmdir()*。  

像 *unlink()* 和 *rmdir()* 一样，*remove()* 不会取消引用符号链接。如果 *pathname* 是一个符号链接，*remove()* 删除的是符号链接本身，而不是它所指向的文件。  

如果我们希望删除一个文件以便准备创建一个同名的新文件，那么使用 *remove()* 比检查路径名是文件还是目录并调用 *unlink()* 或 *rmdir()* 更为简单。  

### 注意  

*remove()* 函数是为标准 C 库发明的，该库在 UNIX 和非 UNIX 系统上都有实现。大多数非 UNIX 系统不支持硬链接，因此使用名为 *unlink()* 的函数来删除文件是没有意义的。

## 读取目录：*opendir()* 和 *readdir()*  

本节描述的库函数可以用于打开目录并逐一检索其中包含的文件名称。  

### 注意  

用于读取目录的库函数是建立在*getdents()*系统调用之上的（该调用不是 SUSv3 的一部分），但提供了一个更易于使用的接口。Linux 还提供了*readdir(2)*系统调用（与此处描述的*readdir(3)*库函数不同），其执行的任务类似，但已被*getdents()*取代。

*opendir()*函数打开一个目录并返回一个句柄，该句柄可以在后续调用中用于引用该目录。

```
#include <dirent.h>

DIR *`opendir`(const char **dirpath*);
```

### 注意

返回目录流句柄，出错时返回`NULL`

*opendir()*函数打开由*dirpath*指定的目录，并返回指向*DIR*类型结构的指针。该结构是所谓的*目录流*，它是一个句柄，调用者将其传递给下面描述的其他函数。*opendir()*返回时，目录流位于目录列表中的第一个条目。

*fdopendir()*函数类似于*opendir()*，不同之处在于为创建目录流指定的目录是通过打开的文件描述符*fd*来指定的。

```
#include <dirent.h>

DIR *`fdopendir`(int fd);
```

### 注意

返回目录流句柄，出错时返回`NULL`

提供*fdopendir()*函数是为了让应用程序避免在操作相对于目录文件描述符中描述的竞争条件。

成功调用*fdopendir()*后，文件描述符由系统控制，程序不应以任何方式访问它，除非使用本节其余部分描述的函数。

*fdopendir()*函数在 SUSv4 中被指定（但在 SUSv3 中没有）。

*readdir()*函数从目录流中读取连续的条目。

```
#include <dirent.h>

struct dirent *`readdir`(DIR **dirp*);
```

### 注意

返回指向静态分配结构的指针，描述下一个目录条目，或者在目录结束或出错时返回`NULL`

每次调用*readdir()*时，都会从由*dirp*引用的目录流中读取下一个目录条目，并返回指向静态分配的*dirent*类型结构的指针，该结构包含有关该条目的以下信息：

```
struct dirent {
    ino_t d_ino;          /* File i-node number */
    char  d_name[];       /* Null-terminated name of file */
};
```

该结构在每次调用*readdir()*时会被覆盖。

### 注意

我们已在上述定义中省略了 Linux *dirent*结构中的各种非标准字段，因为它们的使用会使应用程序变得不可移植。这些非标准字段中最有趣的是*d_type*，它在 BSD 衍生版本中也存在，但在其他 UNIX 实现中不存在。该字段保存一个值，表示在*d_name*中命名的文件类型，例如`DT_REG`（常规文件）、`DT_DIR`（目录）、`DT_LNK`（符号链接）或`DT_FIFO`（FIFO）。 （这些名称类似于表 15-1 中的宏，参见文件大小、分配的块和最佳 I/O 块大小。）使用该字段中的信息可以节省调用*lstat()*以发现文件类型的成本。请注意，然而，在撰写本文时，该字段仅在*Btrfs*、*ext2*、*ext3*和*ext4*上完全支持。

可以通过调用*lstat()*（如果需要取消符号链接，则调用*stat()*）来获取有关由*d_name*引用的文件的更多信息，方法是使用指定给*opendir()*的*dirpath*参数构造的路径名，并与（一个斜杠和）*d_name*字段中返回的值连接起来。

*readdir()*返回的文件名不是按排序顺序排列的，而是按它们在目录中出现的顺序排列（这取决于文件系统向目录添加文件的顺序，以及在删除文件后如何填充目录列表中的空白）。 （命令*ls -f*按与*readdir()*检索文件时相同的无序顺序列出文件。）

### 注意

我们可以使用函数*scandir(3)*来检索符合程序员定义的条件的已排序文件列表；有关详细信息，请参阅手册页。尽管 SUSv3 中未指定，*scandir()*在大多数 UNIX 实现中都提供。SUSv4 为*scandir()*添加了规范。

在目录结束或出错时，*readdir()*返回`NULL`，在后者的情况下，会设置*errno*以指示错误。为了区分这两种情况，我们可以编写如下代码：

```
errno = 0;
direntp = readdir(dirp);
if (direntp == `NULL`) {
    if (errno != 0) {
        /* Handle error */
    } else {
        /* We reached end-of-directory */
    }
}
```

如果目录的内容在程序使用*readdir()*扫描时发生变化，程序可能看不到这些变化。SUSv3 明确指出，未指定*readdir()*是否会返回自上次调用*opendir()*或*rewinddir()*以来已添加或删除的文件名。所有自上次此类调用以来既未添加也未删除的文件名都保证会被返回。

*rewinddir()*函数将目录流移动回起始位置，以便下次调用*readdir()*时，从目录中的第一个文件开始。

```
#include <dirent.h>

void `rewinddir`(DIR **dirp*);
```

*closedir()*函数关闭由*dirp*引用的已打开的目录流，释放该流使用的资源。

```
#include <dirent.h>

int `closedir`(DIR **dirp*);
```

### 注意

成功时返回 0，出错时返回 -1

还有两个函数，*telldir()* 和 *seekdir()*，它们也在 SUSv3 中有指定，允许在目录流中进行随机访问。有关这些函数的更多信息，请参阅手册页。

#### 目录流和文件描述符

目录流有一个相关联的文件描述符。*dirfd()* 函数返回与 *dirp* 引用的目录流相关联的文件描述符。

```
#define _BSD_SOURCE             /* Or: #define _SVID_SOURCE */
#include <dirent.h>

int `dirfd`(DIR **dirp*);
```

### 注意

成功时返回文件描述符，出错时返回 -1

例如，我们可以将 *dirfd()* 返回的文件描述符传递给 *fchdir()*（进程的当前工作目录），以将进程的当前工作目录更改为相应的目录。或者，我们也可以将文件描述符作为 *dirfd* 参数传递给 相对于目录文件描述符操作 中描述的函数之一。

*dirfd()* 函数也出现在 BSD 系统中，但在其他实现中较少出现。它在 SUSv3 中没有指定，但在 SUSv4 中有指定。

在这一点上，值得一提的是，*opendir()* 会自动为与目录流关联的文件描述符设置关闭执行标志（`FD_CLOEXEC`）。这确保了在执行 *exec()* 时，文件描述符会自动关闭。（SUSv3 要求这种行为。）我们在第 27.4 节中描述了关闭执行标志。

#### 示例程序

示例 18-2 使用 *opendir()*、*readdir()* 和 *closedir()* 来列出命令行中指定的每个目录（如果没有提供参数，则列出当前工作目录的内容）。下面是使用此程序的示例：

```
`$ mkdir sub`                             *Create a test directory*
`$ touch sub/a sub/b`                     *Make some files in the test directory*
`$ ./list_files sub`                      *List contents of directory*
sub/a
sub/b
```

示例 18-2. 扫描目录

```
`dirs_links/list_files.c`
#include <dirent.h>
#include "tlpi_hdr.h"

static void             /* List all files in directory 'dirPath' */
listFiles(const char *dirpath)
{
    DIR *dirp;
    struct dirent *dp;
    Boolean isCurrent;          /* True if 'dirpath' is "." */

    isCurrent = strcmp(dirpath, ".") == 0;

    dirp = opendir(dirpath);
    if (dirp  == NULL) {
        errMsg("opendir failed on '%s'", dirpath);
        return;
    }

    /* For each entry in this directory, print directory + filename */

    for (;;) {
        errno = 0;              /* To distinguish error from end-of-directory */
        dp = readdir(dirp);
        if (dp == NULL)
            break;

        if (strcmp(dp->d_name, ".") == 0 || strcmp(dp->d_name, "..") == 0)
            continue;           /* Skip . and .. */

        if (!isCurrent)
            printf("%s/", dirpath);
        printf("%s\n", dp->d_name);
    }

    if (errno != 0)
        errExit("readdir");

    if (closedir(dirp) == -1)
        errMsg("closedir");
}

int
main(int argc, char *argv[])
{
    if (argc > 1 && strcmp(argv[1], "--help") == 0)
        usageErr("%s [dir...]\n", argv[0]);

    if (argc == 1)              /* No arguments - use current directory */
        listFiles(".");
    else
        for (argv++; *argv; argv++)
            listFiles(*argv);

    exit(EXIT_SUCCESS);
}
      `dirs_links/list_files.c`
```

#### *readdir_r()* 函数

*readdir_r()* 函数是 *readdir()* 的一种变体。*readdir_r()* 和 *readdir()* 之间的关键语义差异在于，前者是可重入的，而后者不是。这是因为 *readdir_r()* 通过调用方分配的 *entry* 参数返回文件项，而 *readdir()* 则通过指向静态分配结构的指针返回信息。我们在 可重入和异步信号安全函数 和 线程安全（以及可重入性复习） 中讨论了可重入性。

```
#include <dirent.h>

int `readdir_r`(DIR **dirp*, struct dirent **entry*, struct dirent ***result*);
```

### 注意

成功时返回 0，出错时返回一个正的错误编号

给定 *dirp*，这是一个之前通过 *opendir()* 打开的目录流，*readdir_r()* 将下一个目录项的信息放入 *entry* 所引用的 *dirent* 结构中。此外，指向此结构的指针被放入 *result* 中。如果已到达目录流的末尾，则 *result* 中将放入 `NULL`（此时 *readdir_r()* 返回 0）。如果发生错误，*readdir_r()* 不会返回 -1，而是返回一个对应于某个 *errno* 值的正整数。

在 Linux 中，*dirent* 结构的 *d_name* 字段被定义为一个 256 字节的数组，这足够容纳最大的文件名。尽管其他几个 UNIX 实现也为 *d_name* 定义了相同的大小，但 SUSv3 没有规定这一点，一些 UNIX 实现将该字段定义为 1 字节数组，导致调用程序需要负责分配正确大小的结构体。在这种情况下，我们应该将 *d_name* 字段的大小设置为常量 `NAME_MAX` 的值加一（用于终止的空字节）。因此，便携式应用程序应按以下方式分配 *dirent* 结构：

```
struct dirent *entryp;
size_t len;

len = offsetof(struct dirent, d_name) + `NAME_MAX` + 1;
entryp = malloc(len);
if (entryp == NULL)
    errExit("malloc");
```

使用 `offsetof()` 宏（定义在 `<stddef.h>` 中）可以避免对 *dirent* 结构中在 *d_name* 字段之前的字段数量和大小的任何实现特定依赖（*d_name* 字段总是结构中的最后一个字段）。

### 注意

`offsetof()` 宏接受两个参数——一个结构体类型和该结构体中某个字段的名称——并返回一个 *size_t* 类型的值，表示该字段相对于结构体开始位置的字节偏移量。这个宏是必要的，因为编译器可能会在结构体中插入填充字节，以满足诸如 *int* 之类类型的对齐要求，从而导致字段在结构体中的偏移量可能大于前面字段大小的总和。

## 文件树遍历：*nftw()*

*nftw()* 函数允许程序递归遍历整个目录子树，并对子树中的每个文件执行某些操作（即调用某些程序员定义的函数）。

### 注意

*nftw()* 函数是对旧版 *ftw()* 函数的增强，后者执行类似的任务。新应用程序应使用 *nftw()*（*new ftw*），因为它提供了更多功能，并且能预测性地处理符号链接（SUSv3 允许 *ftw()* 要么跟随符号链接，要么不跟随符号链接）。SUSv3 同时指定了 *nftw()* 和 *ftw()*，但后者在 SUSv4 中被标记为过时。

GNU C 库还提供了基于 BSD 衍生的 *fts* API（*fts_open()*, *fts_read()*, *fts_children()*, *fts_set()*, 和 *fts_close()*）。这些函数执行与 *ftw()* 和 *nftw()* 相似的任务，但为应用程序遍历树结构提供了更大的灵活性。然而，这个 API 并没有标准化，除了 BSD 后代外，其他 UNIX 实现中很少提供，因此我们在这里省略对其的讨论。

*nftw()* 函数会遍历由 *dirpath* 指定的目录树，并对目录树中的每个文件调用程序员定义的函数 *func* 一次。

```
#define _XOPEN_SOURCE 500
#include <ftw.h>

int `nftw`(const char **dirpath*,
         int (**func*) (const char **pathname*, const struct stat **statbuf*,
                      int *typeflag*, struct FTW **ftwbuf*),
         int *nopenfd*, int *flags*);
```

### 注意

成功遍历整个树后返回 0，出错时返回 -1，或者返回 *func* 调用时的第一个非零值。

默认情况下，*nftw()* 对给定的树执行无序的先序遍历，先处理每个目录，再处理该目录下的文件和子目录。

在遍历目录树时，*nftw()* 每一层目录最多只打开一个文件描述符。*nopenfd* 参数指定 *nftw()* 可以使用的最大文件描述符数。如果目录树的深度超过此最大值，*nftw()* 会进行一些记录，并关闭和重新打开描述符，以避免同时打开超过 *nopenfd* 个描述符（从而导致性能下降）。在旧的 UNIX 实现中对这个参数的需求更大，因为其中一些系统对每个进程有 20 个打开文件描述符的限制。现代 UNIX 实现允许进程打开大量文件描述符，因此我们可以在此指定一个较大的数字（比如 10 或更多）。

*nftw()* 的 *flags* 参数是通过按位或（`|`）运算将以下常量之一或多个组合起来的，这些常量会修改该函数的操作：

`FTW_CHDIR`

在处理每个目录的内容之前，先进行 *chdir()* 操作。这在 *func* 设计为在包含其 *pathname* 参数指定的文件的目录中执行某些操作时非常有用。

`FTW_DEPTH`

对目录树进行后序遍历。这意味着 *nftw()* 会在执行 *func* 操作目录本身之前，对目录中的所有文件（和子目录）调用 *func*。（这个标志的名称有些误导——*nftw()* 始终是深度优先遍历目录树，而不是广度优先遍历。这个标志的作用只是将遍历方式从先序转换为后序。）

`FTW_MOUNT`

不要跨越到另一个文件系统。因此，如果树的某个子目录是挂载点，则不会进行遍历。

`FTW_PHYS`

默认情况下，*nftw()* 会取消符号链接的引用。这个标志指示它不要这么做。相反，符号链接会传递给 *func*，并且 *typeflag* 值为 `FTW_SL`，如下所述。

对于每个文件，*nftw()*在调用*func*时传递四个参数。其中第一个参数，*pathname*，是文件的路径名。如果*dirpath*指定为绝对路径名，则此路径名可以是绝对的；如果*dirpath*以相对路径名表示，则此路径名相对于调用*nftw()*时调用进程的当前工作目录。第二个参数，*statbuf*，是指向*stat*结构体的指针（检索文件信息：*stat()*")），该结构体包含该文件的信息。第三个参数，*typeflag*，提供关于文件的进一步信息，并具有以下符号值之一：

`FTW_D`

这是一个目录。

`FTW_DNR`

这是一个无法读取的目录（因此*nftw()*不会遍历其任何子目录）。

`FTW_DP`

我们正在对一个目录进行后序遍历（`FTW_DEPTH`），当前项是一个目录，其文件和子目录已被处理。

`FTW_F`

这是除目录或符号链接外的任何类型的文件。

`FTW_NS`

对该文件调用*stat()*失败，可能是由于权限限制。*statbuf*中的值未定义。

`FTW_SL`

这是一个符号链接。只有在使用`FTW_PHYS`标志调用*nftw()*时，才会返回此值。

`FTW_SLN`

这是一个悬空的符号链接。只有在*flags*参数中未指定`FTW_PHYS`时，才会出现此值。

*func*的第四个参数，*ftwbuf*，是指向以下定义结构体的指针：

```
struct FTW {
    int base;       /* Offset to basename part of pathname */
    int level;      /* Depth of file within tree traversal */
};
```

该结构的*base*字段是*func*的*pathname*参数中文件名部分（即最后一个`/`之后的部分）的整数偏移量。*level*字段是此项相对于遍历起点的深度（起点为 0 级）。

每次调用时，*func*必须返回一个整数值，并且*nftw()*会解释该值。返回 0 表示*nftw()*继续树形遍历，如果所有对*func*的调用都返回 0，则*nftw()*本身会返回 0 给调用者。返回非零值表示*nftw()*立即停止树形遍历，在这种情况下，*nftw()*返回相同的非零值作为返回值。

因为*nftw()*使用动态分配的数据结构，程序唯一应提前终止目录树遍历的方式是通过从*func*返回一个非零值。使用*longjmp()*（执行非局部跳转：*setjmp()* 和 *longjmp()* 和 longjmp()")）从*func*退出可能导致不可预测的结果——至少会导致程序内存泄漏。

#### 示例程序

示例 18-3*遍历目录树")演示了如何使用*nftw()*。

示例 18-3. 使用*nftw()*遍历目录树

```
`dirs_links/nftw_dir_tree.c`
#define _XOPEN_SOURCE 600       /* Get nftw() and S_IFSOCK declarations */
#include <ftw.h>
#include "tlpi_hdr.h"

static void
usageError(const char *progName, const char *msg)
{
    if (msg != NULL)
        fprintf(stderr, "%s\n", msg);
    fprintf(stderr, "Usage: %s [-d] [-m] [-p] [directory-path]\n", progName);
    fprintf(stderr, "\t-d Use FTW_DEPTH flag\n");
    fprintf(stderr, "\t-m Use FTW_MOUNT flag\n");
    fprintf(stderr, "\t-p Use FTW_PHYS flag\n");
    exit(EXIT_FAILURE);
}

static int                      /* Function called by nftw() */
dirTree(const char *pathname, const struct stat *sbuf, int type,
        struct FTW *ftwb)
{
    switch (sbuf->st_mode & S_IFMT) {       /* Print file type */
    case S_IFREG:  printf("-"); break;
    case S_IFDIR:  printf("d"); break;
    case S_IFCHR:  printf("c"); break;
    case S_IFBLK:  printf("b"); break;
    case S_IFLNK:  printf("l"); break;
    case S_IFIFO:  printf("p"); break;
    case S_IFSOCK: printf("s"); break;
    default:       printf("?"); break;      /* Should never happen (on Linux) */
    }

    printf(" %s  ",
            (type == FTW_D)  ? "D  " : (type == FTW_DNR) ? "DNR" :
            (type == FTW_DP) ? "DP " : (type == FTW_F)   ? "F  " :
            (type == FTW_SL) ? "SL " : (type == FTW_SLN) ? "SLN" :
            (type == FTW_NS) ? "NS " : "  ");

    if (type != FTW_NS)
        printf("%7ld ", (long) sbuf->st_ino);
    else
        printf("        ");

    printf(" %*s", 4 * ftwb->level, "");        /* Indent suitably */
    printf("%s\n",  &pathname[ftwb->base]);     /* Print basename */
    return 0;                                   /* Tell nftw() to continue */
}

int
main(int argc, char *argv[])
{
    int flags, opt;

    flags = 0;
    while ((opt = getopt(argc, argv, "dmp")) != -1) {
        switch (opt) {
        case 'd': flags |= FTW_DEPTH;   break;
        case 'm': flags |= FTW_MOUNT;   break;
        case 'p': flags |= FTW_PHYS;    break;
        default:  usageError(argv[0], NULL);
        }
    }

    if (argc > optind + 1)
        usageError(argv[0], NULL);

    if (nftw((argc > optind) ? argv[optind] : ".", dirTree, 10, flags) == -1) {
        perror("nftw");
        exit(EXIT_FAILURE);
    }
    exit(EXIT_SUCCESS);
}
      `dirs_links/nftw_dir_tree.c`
```

示例 18-3 遍历目录树")中的程序显示了目录树中文件名的缩进层级，每个文件一行，以及文件类型和 i 节点号。可以使用命令行选项来指定调用*nftw()*时使用的*flags*参数设置。以下 shell 会话展示了我们运行此程序时的输出示例。我们首先创建一个新的空子目录，然后向其中添加各种类型的文件：

```
`$ mkdir dir`
`$ touch dir/a dir/b`                 *Create some plain files*
`$ ln -s a dir/sl`                    *and a symbolic link*
`$ ln -s x dir/dsl`                   *and a dangling symbolic link*
`$ mkdir dir/sub`                     *and a subdirectory*
`$ touch dir/sub/x`                   *with a file of its own*
`$ mkdir dir/sub2`                    *and another subdirectory*
`$ chmod 0 dir/sub2`                  *that is not readable*
```

我们然后使用我们的程序调用*nftw()*，并传入一个*flags*参数为 0：

```
`$ ./nftw_dir_tree dir`
d D    2327983  dir
- F    2327984      a
- F    2327985      b
- F    2327984      sl              *The symbolic link sl was resolved to  a*
l SLN  2327987      dsl
d D    2327988      sub
- F    2327989          x
d DNR  2327994      sub2
```

在上述输出中，我们可以看到符号链接`s1`已被解析。

我们然后使用我们的程序调用*nftw()*，并传入一个包含`FTW_PHYS`和`FTW_DEPTH`的*flags*参数：

```
`$ ./nftw_dir_tree -p -d dir`
- F    2327984      a
- F    2327985      b
l SL   2327986      sl              *The symbolic link sl was not resolved*
l SL   2327987      dsl
- F    2327989          x
d DP   2327988      sub
d DNR  2327994      sub2
d DP   2327983  dir
```

从上述输出中，我们可以看到符号链接`s1`没有被解析。

#### *nftw()*中的`FTW_ACTIONRETVAL`标志

从版本 2.3.3 开始，*glibc*允许在*flags*中指定一个额外的非标准标志。这个标志`FTW_ACTIONRETVAL`改变了*nftw()*对调用*func()*时返回值的解释。当指定此标志时，*func()*应该返回以下值之一：

`FTW_CONTINUE`

继续处理目录树中的条目，类似于*func()*返回的传统 0 值。

`FTW_SKIP_SIBLINGS`

不再处理当前目录中的其他条目；处理将在父目录中继续。

`FTW_SKIP_SUBTREE`

如果*pathname*是一个目录（即，*typeflag*是`FTW_D`），则不要对该目录下的条目调用*func()*。处理将从该目录的下一个兄弟目录继续。

`FTW_STOP`

不再处理目录树中的其他条目，类似于*func()*返回的传统非零值。`FTW_STOP`值将返回给*nftw()*的调用者。

必须定义`_GNU_SOURCE`功能测试宏，以便从`<ftw.h>`中获取`FTW_ACTIONRETVAL`的定义。

## 进程的当前工作目录

进程的*当前工作目录*定义了相对路径名解析的起始点，进程引用的相对路径名将从此目录开始解析。新进程继承其父进程的当前工作目录。

#### 检索当前工作目录

进程可以使用*getcwd()*检索其当前工作目录。

```
#include <unistd.h>

char *`getcwd`(char **cwdbuf*, size_t *size*);
```

### 注意

成功时返回*cwdbuf*，错误时返回`NULL`

*getcwd()*函数将当前工作目录的绝对路径名放入由*cwdbuf*指向的已分配缓冲区中，路径名以空字符结尾。调用者必须为*cwdbuf*分配至少*size*字节的内存空间。（通常，我们会使用`PATH_MAX`常量来设置*cwdbuf*的大小。）

成功时，*getcwd()*将*cwdbuf*的指针作为函数返回值。如果当前工作目录的路径名超出*size*字节，则*getcwd()*返回`NULL`，并将*errno*设置为`ERANGE`。

在 Linux/x86-32 上，*getcwd()* 返回最大 4096 字节（`PATH_MAX`）。如果当前工作目录（以及 *cwdbuf* 和 *size*）超过此限制，则路径名会被静默截断，删除字符串*开头*的完整目录前缀（仍然是以 null 结尾）。换句话说，当当前工作目录的绝对路径名长度超过此限制时，我们不能可靠地使用 *getcwd()*。

### 注意

实际上，Linux 的 *getcwd()* 系统调用在内部为返回的路径名分配了一个虚拟内存页面。在 x86-32 架构上，页面大小为 4096 字节，但在页面大小更大的架构上（例如，Alpha 架构的页面大小为 8192 字节），*getcwd()* 可以返回更大的路径名。

如果 *cwdbuf* 参数为 `NULL` 且 *size* 为 0，则 *glibc* 为 *getcwd()* 的包装函数分配足够大的缓冲区，并将指向该缓冲区的指针作为函数结果返回。为了避免内存泄漏，调用者必须稍后使用 *free()* 释放此缓冲区。应避免在可移植应用程序中依赖此功能。大多数其他实现提供了对 SUSv3 规范的更简单扩展：如果 *cwdbuf* 为 `NULL`，则 *getcwd()* 会分配 *size* 字节，并使用此缓冲区将结果返回给调用者。*glibc getcwd()* 实现也提供了此功能。

### 注意

GNU C 库还提供了另外两个获取当前工作目录的函数。BSD 衍生的 *getwd(path)* 函数易受到缓冲区溢出的影响，因为它没有提供指定返回路径名大小上限的方法。*get_current_dir_name()* 函数返回一个包含当前工作目录名称的字符串作为其函数结果。此函数易于使用，但不可移植。为了安全性和可移植性，*getcwd()* 比这两个函数更受推荐（只要我们避免使用 GNU 扩展）。

具有适当权限（大致上，我们拥有进程或具有`CAP_SYS_PTRACE`能力）时，可以通过读取 (*readlink()*) Linux 特定的 `/proc/`*PID*`/cwd` 符号链接的内容，确定任何进程的当前工作目录。

#### 更改当前工作目录

*chdir()* 系统调用将调用进程的当前工作目录更改为 *pathname* 中指定的相对或绝对路径名（如果是符号链接，则会解引用该路径名）。

```
#include <unistd.h>

int `chdir`(const char **pathname*);
```

### 注意

成功时返回 0，失败时返回 -1

*fchdir()* 系统调用与 *chdir()* 执行相同的操作，不同之处在于，目录是通过先前通过 *open()* 打开目录而获得的文件描述符来指定的。

```
#include <unistd.h>

int `fchdir`(int *fd*);
```

### 注意

成功时返回 0，失败时返回 -1

我们可以使用 *fchdir()* 将进程的当前工作目录更改为另一个位置，然后稍后返回原始位置，如下所示：

```
int fd;

fd = open(".", O_RDONLY);       /* Remember where we are */
chdir(somepath);                /* Go somewhere else */
fchdir(fd);                     /* Return to original directory */
close(fd);
```

使用 *chdir()* 等效的操作如下：

```
char buf[PATH_MAX];

getcwd(buf, PATH_MAX);          /* Remember where we are */
chdir(somepath);                /* Go somewhere else */
chdir(buf);                     /* Return to original directory */
```

## 操作相对于目录文件描述符

从内核版本 2.6.16 开始，Linux 提供了一系列新的系统调用，这些系统调用执行与传统系统调用相似的任务，但提供了额外的功能，这对某些应用程序非常有用。这些系统调用在表 18-2 中进行了总结。我们在本章中描述这些系统调用，因为它们对进程当前工作目录的传统语义提供了变化。

表 18-2. 使用目录文件描述符解释相对路径名的系统调用

| 新接口 | 传统类比 | 注释 |
| --- | --- | --- |
| *faccessat()* | *access()* | 支持 `AT_EACCESS` 和 `AT_SYMLINK_NOFOLLOW` 标志 |
| *fchmodat()* | *chmod()* |   |
| *fchownat()* | *chown()* | 支持 `AT_SYMLINK_NOFOLLOW` 标志 |
| *fstatat()* | *stat()* | 支持 `AT_SYMLINK_NOFOLLOW` 标志 |
| *linkat()* | *link()* | 支持（从 Linux 2.6.18 开始）`AT_SYMLINK_FOLLOW` 标志 |
| *mkdirat()* | *mkdir()* |   |
| *mkfifoat()* | *mkfifo()* | 该库函数是在 `mknodat()` 之上封装的 |
| *mknodat()* | *mknod()* |   |
| *openat()* | *open()* |   |
| *readlinkat()* | *readlink()* |   |
| *renameat()* | *rename()* |   |
| *symlinkat()* | *symlink()* |   |
| *unlinkat()* | *unlink()* | 支持 `AT_REMOVEDIR` 标志 |
| *utimensat()* | *utimes()* | 支持 `AT_SYMLINK_NOFOLLOW` 标志 |

为了描述这些系统调用，我们将使用一个特定的例子：*openat()*。

```
#define _XOPEN_SOURCE 700     /* Or define _POSIX_C_SOURCE >= 200809 */
#include <fcntl.h>

int `openat`(int *dirfd*, const char **pathname*, int
 *flags*, ... /* mode_t  *mode* */);
```

### 注

成功时返回文件描述符，出错时返回 -1

*openat()* 系统调用类似于传统的 *open()* 系统调用，但增加了一个参数 *dirfd*，该参数的使用方式如下：

+   如果 *pathname* 指定了相对路径名，那么它会相对于由打开的文件描述符 *dirfd* 所指示的目录进行解释，而不是相对于进程的当前工作目录。

+   如果 *pathname* 指定了一个相对路径名，并且 *dirfd* 包含特殊值 `AT_FDCWD`，那么 *pathname* 会相对于进程的当前工作目录进行解释（即，与 *open(2)* 相同的行为）。

+   如果 *pathname* 指定了绝对路径名，则 *dirfd* 被忽略。

*openat()* 的 *flags* 参数与 *open()* 中的作用相同。然而，表 18-2 中列出的一些系统调用支持 *flags* 参数，而传统系统调用中没有该参数，且该参数的目的是修改调用的语义。最常见的标志是 `AT_SYMLINK_NOFOLLOW`，它指定如果 *pathname* 是符号链接，那么系统调用应对链接进行操作，而不是对其指向的文件进行操作。（*linkat()* 系统调用提供了 `AT_SYMLINK_FOLLOW` 标志，执行相反的操作，改变 *linkat()* 的默认行为，使其在 *oldpath* 是符号链接时进行解引用。）有关其他标志的详细信息，请参阅相应的手册页。

表 18-2 中列出的系统调用有两个原因被支持（我们再次以 *openat()* 为例解释）：

+   使用 *openat()* 可以让应用程序避免使用 *open()* 在当前工作目录之外的其他位置打开文件时可能发生的某些竞态条件。这些竞态条件可能发生，因为*pathname*的某些目录前缀组件可能在与*open()*调用并行时发生变化。通过为目标目录打开一个文件描述符，并将该描述符传递给 *openat()*，可以避免这种竞态条件。

+   在第二十九章中，我们将看到工作目录是一个进程属性，所有线程共享。对于某些应用程序，不同的线程拥有不同的“虚拟”工作目录是有用的。应用程序可以通过将 *openat()* 与应用程序维护的目录文件描述符结合使用来模拟此功能。

这些系统调用在 SUSv3 中没有标准化，但在 SUSv4 中被包含。为了暴露每个系统调用的声明，必须在包含适当的头文件（例如，`<fcntl.h>` 用于 *open()*) 之前定义宏`_XOPEN_SOURCE`，并且其值大于或等于 700。或者，也可以定义宏`_POSIX_C_SOURCE`，并且其值大于或等于 200809。（在*glibc* 2.10 之前，需要定义宏`_ATFILE_SOURCE`来暴露这些系统调用的声明。）

### 注意

Solaris 9 及更高版本提供了表 18-2 中列出的一些接口的版本，具有略微不同的语义。

## 改变进程的根目录：*chroot()*

每个进程都有一个 *根目录*，这是解释绝对路径名（即以 / 开头的路径名）的起点。默认情况下，这是文件系统的真实根目录。（新进程继承其父进程的根目录。）偶尔，一个进程更改其根目录是有用的，并且有特权（`CAP_SYS_CHROOT`）的进程可以使用 *chroot()* 系统调用来实现这一点。

```
#define _BSD_SOURCE
#include <unistd.h>

int `chroot`(const char **pathname*);
```

### 注意

成功返回 0，错误返回 -1

*chroot()* 系统调用将进程的根目录更改为由 *pathname* 指定的目录（如果它是符号链接，则进行解引用）。此后，所有绝对路径名都被解释为从文件系统中的该位置开始。有时这被称为设置 *chroot* 监狱，因为程序随后被限制在文件系统的特定区域内。

SUSv2 包含了 *chroot()* 的规范（标记为 LEGACY），但在 SUSv3 中已移除。尽管如此，*chroot()* 出现在大多数 UNIX 实现中。

### 注意

*chroot()* 系统调用由 *chroot* 命令使用，它使我们能够在 *chroot* 监狱中执行 shell 命令。

任何进程的根目录可以通过读取（*readlink()*）Linux 特定的 `/proc/`*PID*`/root` 符号链接的内容找到。

*chroot()* 的经典应用示例是在 *ftp* 程序中。作为安全措施，当用户以匿名方式登录 FTP 时，*ftp* 程序使用 *chroot()* 将新进程的根目录设置为专门为匿名登录保留的目录。在执行 chroot() 调用后，用户被限制在其新根目录下的文件系统子树中，因此不能在整个文件系统中漫游。（这依赖于根目录是其自身的父目录的事实；也就是说，`/..` 是指向 `/` 的链接，因此改变到 `/` 然后尝试 *cd ..* 将会让用户留在同一目录。）

### 注意

一些 UNIX 实现（但不包括 Linux）允许一个目录有多个硬链接，因此可以在子目录中创建一个硬链接到其父目录（或进一步移除的祖先）。在允许此操作的实现中，存在一条硬链接超出监狱目录树的情况会破坏监狱的安全性。指向监狱外部目录的符号链接不会构成问题——因为它们在进程新根目录的框架内解释，它们无法达到 *chroot* 监狱之外。

通常情况下，我们不能在 *chroot* 监狱中执行任意程序。这是因为大多数程序是动态链接到共享库的。因此，我们必须限制自己只能执行静态链接的程序，或者在监狱中复制包含共享库的标准系统目录集合（例如 `/lib` 和 `/usr/lib`）（在这方面，文中描述的绑定挂载功能在 绑定挂载 中可能是有用的）。

*chroot()*系统调用最初并不是作为一个完全安全的监狱机制设计的。首先，有多种方式，特权程序可以利用进一步的*chroot()*调用突破监狱。例如，一个特权程序（`CAP_MKNOD`）可以使用*mknod()*创建一个内存设备文件（类似于`/dev/mem`），从而访问 RAM 的内容，从那时起，一切皆有可能。一般来说，建议不要在*chroot*监狱文件系统中包含 set-user-ID-*root*程序。

即使是非特权程序，我们也必须小心防止以下可能的路径导致突破*chroot*监狱：

+   调用*chroot()*不会改变进程的当前工作目录。因此，*chroot()*调用通常会前后紧跟着*chdir()*调用（例如，*chdir*(“/”)在*chroot()*调用之后）。如果没有执行此操作，进程可以使用相对路径名访问监狱外的文件和目录。（一些 BSD 派生系统防止了这种可能性——如果当前工作目录位于新根目录树之外，则*chroot()*调用会将其更改为与根目录相同。）

+   如果一个进程持有指向监狱外目录的打开文件描述符，则可以结合使用*fchdir()*和*chroot()*来突破监狱，如以下代码示例所示：

    ```
    int fd;

    fd = open("/", O_RDONLY);
    chroot("/home/mtk");            /* Jailed */
    fchdir(fd);
    chroot(".");                    /* Out of jail */
    ```

    为了防止这种情况，我们必须关闭所有指向监狱外部目录的打开文件描述符。（其他一些 UNIX 实现提供了*fchroot()*系统调用，可以用来实现与上述代码片段相似的效果。）

+   即便防止了前述的可能性，也不足以阻止一个任意的非特权程序（即我们无法控制其操作的程序）从监狱中逃脱。被监禁的进程仍然可以使用 UNIX 域套接字接收一个文件描述符（来自另一个进程），该描述符指向监狱外的目录。（我们将在传递文件描述符中简要解释通过套接字在进程间传递文件描述符的概念。）通过在对*fchdir()*的调用中指定该文件描述符，程序可以将当前工作目录设置到监狱外，并使用相对路径名访问任意文件和目录。

### 注意

一些 BSD 派生系统提供了一个*jail()*系统调用，它解决了上面描述的问题以及其他一些问题，可以创建一个即使对特权进程也安全的监狱环境。

## 解析路径名：*realpath()*

*realpath()*库函数会解引用*pathname*（一个以空字符结尾的字符串）中的所有符号链接，并解析所有对/.和/..的引用，以生成一个包含相应绝对路径名的以空字符结尾的字符串。

```
#include <stdlib.h>

char *`realpath`(const char **pathname*, char **resolved_path*);
```

### 注意

成功时返回指向解析后的路径名的指针，出错时返回`NULL`。

结果字符串将被放置在 *resolved_path* 指向的缓冲区中，该缓冲区应至少为 `PATH_MAX` 字节的字符数组。在成功的情况下，*realpath()* 还会返回指向该解析字符串的指针。

*glibc* 实现的 *realpath()* 允许调用者将 *resolved_path* 指定为 `NULL`。在这种情况下，*realpath()* 为解析后的路径名分配最多 `PATH_MAX` 字节的缓冲区，并将该缓冲区的指针作为函数结果返回。（调用者必须使用 *free()* 释放该缓冲区。）SUSv3 没有指定这一扩展，但在 SUSv4 中有所规定。

示例 18-4 中的程序使用 *readlink()* 和 *realpath()* 读取符号链接的内容，并将链接解析为绝对路径名。以下是此程序使用的示例：

```
`$ pwd`                                       *Where are we?*
/home/mtk
`$ touch x`                                   *Make a file*
`$ ln -s x y`                                 *and a symbolic link to it*
`$ ./view_symlink y`
readlink: y --> x
realpath: y --> /home/mtk/x
```

示例 18-4. 读取并解析符号链接

```
`dirs_links/view_symlink.c`
#include <sys/stat.h>
#include <limits.h>             /* For definition of PATH_MAX */
#include "tlpi_hdr.h"

#define BUF_SIZE PATH_MAX

int
main(int argc, char *argv[])
{
    struct stat statbuf;
    char buf[BUF_SIZE];
    ssize_t numBytes;

    if (argc != 2 || strcmp(argv[1], "--help") == 0)
        usageErr("%s pathname\n", argv[0]);

    if (lstat(argv[1], &statbuf) == -1)
        errExit("lstat");

    if (!S_ISLNK(statbuf.st_mode))
        fatal("%s is not a symbolic link", argv[1]);

    numBytes = readlink(argv[1], buf, BUF_SIZE - 1);
    if (numBytes == -1)
        errExit("readlink");
    buf[numBytes] = '\0';                       /* Add terminating null byte */
    printf("readlink: %s --> %s\n", argv[1], buf);

    if (realpath(argv[1], buf) == NULL)
        errExit("realpath");
    printf("realpath: %s --> %s\n", argv[1], buf);

    exit(EXIT_SUCCESS);
}
      `dirs_links/view_symlink.c`
```

## 解析路径名字符串：*dirname()* 和 *basename()*

*dirname()* 和 *basename()* 函数将路径名字符串分解为目录和文件名部分。（这些函数执行的任务与 *dirname(1)* 和 *basename(1)* 命令类似。）

```
#include <libgen.h>

char *`dirname`(char **pathname*);
char *`basename`(char **pathname*);
```

### 注意

两者都返回指向空字符终止（且可能是静态分配的）字符串的指针。

例如，给定路径名 `/home/britta/prog.c`，*dirname()* 返回 `/home/britta`，而 *basename()* 返回 `prog.c`。将 *dirname()* 返回的字符串、一个斜杠（`/`）和 *basename()* 返回的字符串连接起来，即可得到完整的路径名。

请注意以下有关 *dirname()* 和 *basename()* 操作的几点：

+   *pathname* 中的尾部斜杠字符会被忽略。

+   如果 *pathname* 不包含斜杠，则 *dirname()* 返回字符串 `.`（点），而 *basename()* 返回 *pathname*。

+   如果 *pathname* 仅由一个斜杠组成，则 *dirname()* 和 *basename()* 都返回字符串 `/`。按照上述的连接规则，用这些返回的字符串创建路径名将得到字符串 `///`。这*是*一个有效的路径名。因为多个连续的斜杠等价于一个斜杠，所以路径名 `///` 等同于路径名 `/`。

+   如果 *pathname* 是 `NULL` 指针或空字符串，则 *dirname()* 和 *basename()* 都返回字符串 `.`（点）。（将这些字符串连接起来得到路径名 `./.`，其等价于当前目录 `.`。）

表 18-3* 和 *basename()* 返回的字符串示例") 显示了对于各种示例路径名，*dirname()* 和 *basename()* 返回的字符串。

表 18-3. *dirname()* 和 *basename()* 返回的字符串示例

| 路径名字符串 | *dirname()* | *basename()* |
| --- | --- | --- |
| `/` | `/` | `/` |
| `/usr/bin/zip` | `/usr/bin` | `zip` |
| `/etc/passwd////` | `/etc` | `passwd` |
| `/etc////passwd` | `/etc` | `passwd` |
| `etc/passwd` | `etc` | `passwd` |
| `passwd` | `.` | `passwd` |
| `passwd/` | `.` | `passwd` |
| `..` | `.` | `..` |
| `NULL` | `.` | `.` |

示例 18-5. 使用 *dirname()* 和 *basename()*

```
`dirs_links/t_dirbasename.c`
#include <libgen.h>
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    char *t1, *t2;
    int j;

    for (j = 1; j < argc; j++)  {
        t1 = strdup(argv[j]);
        if (t1 == NULL)
            errExit("strdup");
        t2 = strdup(argv[j]);
        if (t2 == NULL)
            errExit("strdup");

        printf("%s ==> %s + %s\n", argv[j], dirname(t1), basename(t2));

        free(t1);
        free(t2);
    }

    exit(EXIT_SUCCESS);
}
      `dirs_links/t_dirbasename.c`
```

*dirname()* 和 *basename()* 可能会修改 *pathname* 所指向的字符串。因此，如果我们希望保留一个路径名字符串，我们必须将其副本传递给 *dirname()* 和 *basename()*，如 示例 18-5 和 basename()")（第 371 页）所示。此程序使用 *strdup()*（它调用 *malloc()*）来制作要传递给 *dirname()* 和 *basename()* 的字符串副本，然后使用 *free()* 来释放这些副本字符串。

最后，请注意 *dirname()* 和 *basename()* 都可以返回指向静态分配字符串的指针，这些字符串可能会被未来对同一函数的调用修改。

## 总结

一个 i-node 不包含文件名。相反，文件通过目录中的条目来分配名称，这些条目是列出文件名和 i-node 编号对应关系的表格。这些目录条目被称为（硬）链接。一个文件可以有多个链接，所有这些链接具有相同的地位。链接是通过 *link()* 和 *unlink()* 创建和删除的。一个文件可以使用 *rename()* 系统调用进行重命名。

使用 *symlink()* 创建符号（或软）链接。符号链接在某些方面类似于硬链接，但不同之处在于，符号链接可以跨越文件系统边界并可以引用目录。符号链接只是一个包含另一个文件名称的文件；这个名称可以通过 *readlink()* 获取。符号链接不包含在（目标）i-node 的链接计数中，如果它所指向的文件名被删除，符号链接可能会变成悬挂状态。某些系统调用会自动解引用（跟随）符号链接；而其他系统调用则不会。在某些情况下，提供了两种版本的系统调用：一种解引用符号链接，另一种则不解引用。示例包括 *stat()* 和 *lstat()*。

目录通过 *mkdir()* 创建，使用 *rmdir()* 删除。要扫描目录内容，我们可以使用 *opendir()*、*readdir()* 和相关函数。*nftw()* 函数允许程序遍历整个目录树，调用程序定义的函数对树中的每个文件进行操作。

*remove()* 函数可用于删除文件（即，链接）或空目录。

每个进程都有一个根目录，它决定了绝对路径名的解释起点，还有一个当前工作目录，它决定了相对路径名的解释起点。*chroot()* 和 *chdir()* 系统调用用于更改这些属性。*getcwd()* 函数返回进程的当前工作目录。

Linux 提供了一组系统调用（例如，*openat()*），它们的行为类似于传统的对等函数（例如，*open()*），但相对路径名可以根据传递给调用的文件描述符指定的目录进行解释（而不是使用进程的当前工作目录）。这对于避免某些类型的竞争条件以及实现每个线程的虚拟工作目录非常有用。

*realpath()*函数解析一个路径名——取消所有符号链接并解析所有对`.`和`..`的引用为相应的目录——从而生成一个对应的绝对路径名。*dirname()*和*basename()*函数可以用来将路径名解析为目录和文件名组件。

## 练习

1.  在《*open() 标志* 参数》 flags Argument")中，我们注意到，如果文件正在被执行，则无法打开它进行写入（*open()*返回-1，并且*errno*被设置为`ETXTBSY`）。然而，从 shell 中可以执行以下操作：

    ```
    $ `cc -o longrunner longrunner.c`
    $ `./longrunner &`                        *Leave running in background*
    $ `vi longrunner.c`                       *Make some changes to the source code*
    $ `cc -o longrunner longrunner.c`
    ```

    最后一条命令覆盖了同名的现有可执行文件。怎么做到的呢？（提示：使用*ls -li*查看每次编译后可执行文件的 i 节点号。）

1.  为什么下面代码中的*chmod()*调用会失败？

    ```
    mkdir("test", S_IRUSR | S_IWUSR | S_IXUSR);
    chdir("test");
    fd = open("myfile", O_RDWR | O_CREAT, S_IRUSR | S_IWUSR);
    symlink("myfile", "../mylink");
    chmod("../mylink", S_IRUSR);
    ```

1.  实现*realpath()*。

1.  修改示例 18-2（`list_files.c`）中的程序，使用*readdir_r()*代替*readdir()*。

1.  实现一个功能，执行与*getcwd()*等效的操作。解决此问题的一个有用提示是，你可以通过使用*opendir()*和*readdir()*遍历父目录（`..`）中的每个条目，找到与当前工作目录具有相同 i 节点和设备号的条目（即分别是*stat()*和*lstat()*返回的*st_ino*和*st_dev*字段）。因此，可以通过逐步向上走目录树（*chdir(`..`)*）并进行这种扫描来构建目录路径。当父目录与当前工作目录相同（回想一下，`/..`与`/`是相同的）时，可以结束扫描。调用者应保持在其启动时所在的目录，不管你的*getcwd()*函数是否成功或失败（*open()*加上*fchdir()*对于此目的非常有用）。

1.  修改示例 18-3 遍历目录树")（`nftw_dir_tree.c`）中的程序，使用`FTW_DEPTH`标志。注意目录树遍历顺序的不同。

1.  编写一个程序，使用*nftw()*遍历目录树，最后打印出树中各种类型（常规文件、目录、符号链接等）的文件数量和百分比。

1.  实现*nftw()*。（这将需要使用*opendir()*、*readdir()*、*closedir()*、*stat()*等系统调用。）

1.  在进程的当前工作目录中，我们展示了两种不同的技术（分别使用*fchdir()*和*chdir()*）来在将当前工作目录更改为另一个位置后返回到之前的当前工作目录。假设我们正在重复执行这样的操作。你认为哪种方法效率更高？为什么？编写一个程序以确认你的答案。
