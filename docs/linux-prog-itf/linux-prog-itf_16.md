## 第十六章. 扩展属性

本章描述了扩展属性（EA），它允许将任意的元数据（以名称-值对的形式）与文件 i 节点关联。EA 是在 Linux 2.6 版本中加入的。

## 概述

EA 用于实现访问控制列表（第十七章）和文件能力（第三十九章）。然而，EA 的设计足够通用，也允许它们用于其他目的。例如，EA 可以用来记录文件版本号、文件的 MIME 类型或字符集信息，或者（指向）一个图标。

EA 在 SUSv3 中没有被规范化。然而，类似的功能在一些其他 UNIX 实现中提供，特别是现代 BSD 系统（参见*extattr(2)*)和 Solaris 9 及以后版本（参见*fsattr(5)*）。

EA 需要底层文件系统的支持。*Btrfs*、*ext2*、*ext3*、*ext4*、*JFS*、*Reiserfs*和*XFS*都提供了这种支持。

### 注意

对于每个文件系统，EA 的支持是可选的，且由*文件系统*菜单下的内核配置选项控制。从 Linux 2.6.7 开始，*Reiserfs*支持 EA。

#### EA 命名空间

EA 的名称形式为*命名空间.名称*。*命名空间*组件用于将 EA 划分为不同功能的类别。*名称*组件在给定的*命名空间*内唯一标识一个 EA。

支持四种*命名空间*的值：*user*、*trusted*、*system*和*security*。这四种 EA 类型的使用方式如下：

+   *User* EA 可以被非特权进程操作，但需要进行文件权限检查：检索*user* EA 的值需要对文件具有读取权限；更改*user* EA 的值需要对文件具有写入权限。（缺少所需权限会导致`EACCES`错误。）为了将*user* EA 与*ext2*、*ext3*、*ext4*或*Reiserfs*文件系统中的文件关联，必须使用*user_xattr*选项挂载底层文件系统：

    ```
    $ `mount -o user_xattr` ``*`device directory`*``
    ```

+   *Trusted* EA 与*user* EA 类似，可以被用户进程操作。不同之处在于，进程必须具备特权（`CAP_SYS_ADMIN`）才能操作*trusted* EA。

+   *System* EA 由内核使用，用于将系统对象与文件关联。目前，唯一受支持的对象类型是访问控制列表（第十七章）。

+   *Security* EA 用于存储操作系统安全模块的文件安全标签，并将能力与可执行文件关联（文件能力）。*Security* EA 最初是为了支持安全增强 Linux（SELinux，[`www.nsa.gov/research/selinux/`](http://www.nsa.gov/research/selinux/)）而设计的。

一个 i-node 可能有多个关联的 EA，可能在同一命名空间内，也可能在不同命名空间内。每个命名空间中的 EA 名称是独立的集合。在 *user* 和 *trusted* 命名空间中，EA 名称可以是任意字符串。在 *system* 命名空间中，仅允许内核明确允许的名称（例如，用于访问控制列表的名称）。

### 注意

*JFS* 支持另一个命名空间 *os2*，该命名空间在其他文件系统中未实现。*os2* 命名空间用于支持旧版 OS/2 文件系统的 EA。进程不需要具有特权就能创建 *os2* EA。

#### 从 shell 创建和查看 EA

从 shell 中，我们可以使用 *setfattr(1)* 和 *getfattr(1)* 命令来设置和查看文件上的 EA：

```
`$ touch tfile`
`$ setfattr -n user.x -v "The past is not dead." tfile`
`$ setfattr -n user.y -v "In fact, it's not even past." tfile`
`$ getfattr -n user.x tfile`          *Retrieve value of a single EA*
# file: tfile                       *Informational message from getfattr*
user.x="The past is not dead."      *The getfattr command prints a blank*
                                    *line after each file’s attributes*
$ getfattr -d tfile                 *Dump values of all user EAs*
# file: tfile
user.x="The past is not dead."
user.y="In fact, it's not even past."

`$ setfattr -n user.x tfile`          *Change value of EA to be an empty string*
`$ getfattr -d tfile`
# file: tfile
user.x
user.y="In fact, it's not even past."

`$ setfattr -x user.y tfile`          *Remove an EA*
`$ getfattr -d tfile`
# file: tfile
user.x
```

前面 shell 会话所展示的一点是，EA（扩展属性）的值可以是空字符串，这与未定义的 EA 不同。（在 shell 会话结束时，*user.x* 的值是空字符串，而 *user.y* 是未定义的。）

默认情况下，*getfattr* 只列出 *user* EA 的值。可以使用 *-m* 选项指定一个正则表达式模式，选择要显示的 EA 名称：

```
`$ getfattr -m '```*`pattern`*```'` ``*`file`*``
```

*pattern* 的默认值是 `^user\.`。我们可以使用以下命令列出文件上的所有 EA：

```
$ ``getfattr -m - *`file`*``
```

## 扩展属性实现细节

在本节中，我们扩展了前面一节的概述，填补了一些扩展属性实现的细节。

#### *user* 扩展属性的限制

只能将 *user* EA 放置在文件和目录上。其他文件类型因以下原因被排除：

+   对于符号链接，所有用户的权限都被启用，并且这些权限无法更改。（符号链接的权限在 Linux 上没有意义，详见第 18.2 节。）这意味着权限无法用来防止任意用户在符号链接上放置 *user* EA。解决该问题的方法是禁止所有用户在符号链接上创建 *user* EA。

+   对于设备文件、套接字和 FIFO，权限控制用户在执行 I/O 操作时对底层对象的访问。修改这些权限以控制 *user* EA 的创建将与该目的相冲突。

此外，如果目录上设置了粘滞位（Set-User-ID, Set-Group-ID, and Sticky Bits），则无特权进程无法将 *user* EA 放置在其他用户拥有的目录上。这防止了任意用户将 EA 附加到像 `/tmp` 这样的公共可写目录（这样可能允许任意用户操控该目录上的 EA），但该目录已设置粘滞位，防止用户删除其他用户拥有的文件。

#### 实现限制

Linux VFS 对所有文件系统中的 EA 施加了以下限制：

+   EA 名称的长度限制为 255 个字符。

+   EA 值的大小限制为 64 KB。

此外，一些文件系统对与文件关联的 EA 的大小和数量施加了更严格的限制：

+   在 *ext2, ext3* 和 *ext4* 上，文件上所有 EA 的名称和值的总字节数限制为单个逻辑磁盘块的大小（文件系统）：1024、2048 或 4096 字节。

+   在 *JFS* 上，所有 EA 的名称和值在文件上的总字节数有 128 KB 的上限。

## 操作扩展属性的系统调用

在本节中，我们将查看用于更新、检索和删除 EA 的系统调用。

#### 创建和修改扩展属性

*setxattr(), lsetxattr()* 和 *fsetxattr()* 系统调用用于设置文件某个扩展属性的值。

```
#include <sys/xattr.h>

int `setxattr`(const char **pathname*, const char **name*, const void **value*,
              size_t *size*, int *f lags*);
int `lsetxattr`(const char **pathname*, const char **name*, const void **value*,
              size_t *size*, int *f lags*);
int `fsetxattr`(int *fd*, const char **name*, const void **value*,
              size_t *size*, int *f lags*);
```

### 注意

所有成功时返回 0，失败时返回 -1

这三者之间的区别类似于 *stat()*、*lstat()* 和 *fstat()* 之间的区别（获取文件信息：*stat()*")）：

+   *setxattr()* 通过 *pathname* 来识别文件，并在文件是符号链接时解引用文件名；

+   *lsetxattr()* 通过 *pathname* 来识别文件，但不解引用符号链接；

+   *fsetxattr()* 通过打开的文件描述符 *fd* 来识别文件。

同样的区别适用于本节剩余部分中描述的其他系统调用组。

*name* 参数是一个以空字符结尾的字符串，定义了 EA 的名称。*value* 参数是一个指向缓冲区的指针，定义了 EA 的新值。*size* 参数指定了该缓冲区的长度。

默认情况下，这些系统调用会创建一个新的 EA，如果给定的 *name* 对应的 EA 不存在，或者如果已经存在，则替换该 EA 的值。*flags* 参数提供了对这种行为的更精细控制。可以指定为 0 来获取默认行为，或者指定为以下常量之一：

`XATTR_CREATE`

如果给定的 *name* 对应的 EA 已经存在，则失败（`EEXIST`）。

`XATTR_REPLACE`

如果给定的 *name* 对应的 EA 不存在，则失败（`ENODATA`）。

这是使用 *setxattr()* 创建 *user* EA 的示例：

```
char *value;

value = "The past is not dead.";

if (setxattr(pathname, "user.x", value, strlen(value), 0) == -1)
    errExit("setxattr");
```

#### 获取 EA 的值

*getxattr()*、*lgetxattr()* 和 *fgetxattr()* 系统调用用于获取 EA 的值。

```
#include <sys/xattr.h>

ssize_t `getxattr`(const char **pathname*, const char **name*, void **value*,
                  size_t *size*);
ssize_t `lgetxattr`(const char **pathname*, const char **name*, void **value*,
                  size_t *size*);
ssize_t `fgetxattr`(int *fd*, const char **name*, void **value*,
                  size_t *size*);
```

### 注意

所有返回（非负）EA 值的大小，成功时返回，失败时返回 -1

*name* 参数是一个以空字符结尾的字符串，用于标识我们想要获取值的扩展属性（EA）。EA 的值将保存在 *value* 所指向的缓冲区中。这个缓冲区必须由调用者分配，并且其长度必须在 *size* 中指定。成功时，这些系统调用返回复制到 *value* 中的字节数。

如果文件没有给定 *name* 的属性，这些系统调用将失败并返回错误 `ENODATA`。如果 *size* 太小，这些系统调用将失败并返回错误 `ERANGE`。

可以将*size*指定为 0，在这种情况下，*value*会被忽略，但系统调用仍会返回 EA 值的大小。这提供了一种机制，用于确定为后续调用实际检索 EA 值所需的*value*缓冲区的大小。然而，请注意，我们不能保证返回的大小在后续尝试检索值时足够大。另一个进程可能在此期间为该属性分配了更大的值，或完全删除了该属性。

#### 删除 EA

*removexattr()*、*lremovexattr()*和*fremovexattr()*系统调用从文件中删除 EA。

```
#include <sys/xattr.h>

int `removexattr`(const char **pathname*, const char **name*);
int `lremovexattr`(const char **pathname*, const char **name*);
int `fremovexattr`(int *fd*, const char **name*);
```

### 注意

成功时返回 0，错误时返回-1。

在*name*中给定的以空字符终止的字符串标识要删除的 EA。尝试删除一个不存在的 EA 会失败，并返回错误`ENODATA`。

#### 检索与文件关联的所有 EA 名称。

*listxattr()*、*llistxattr()*和*flistxattr()*系统调用返回包含所有与文件关联的 EA 名称的列表。

```
#include <sys/xattr.h>

ssize_t `listxattr`(const char **pathname*, char **list*, size_t *size*);
ssize_t `llistxattr`(const char **pathname*, char **list*, size_t *size*);
ssize_t `flistxattr`(int *fd*, char **list*, size_t *size*);
```

### 注意

成功时返回复制到 list 中的字节数，错误时返回-1。

EA 名称列表以一系列以空字符终止的字符串形式返回，这些字符串存储在由*list*指向的缓冲区中。此缓冲区的大小必须在*size*中指定。成功时，这些系统调用返回复制到*list*中的字节数。

与*getxattr()*类似，可以将*size*指定为 0，在这种情况下，*list*会被忽略，但系统调用仍会返回为后续调用实际检索 EA 名称列表所需的缓冲区大小（假设列表未改变）。

要检索与文件关联的 EA 名称列表，只需要确保文件可访问（即，我们对*pathname*中包含的所有目录具有执行权限）。文件本身不需要任何权限。

出于安全原因，*list*中返回的 EA 名称可能会排除调用进程没有访问权限的属性。例如，大多数文件系统会在无特权进程调用*listxattr()*时省略*trusted*属性。但请注意前面句子中的“可能”，表明文件系统实现并不强制要求这样做。因此，我们需要考虑到使用*list*中返回的 EA 名称后续调用*getxattr()*时，可能由于进程没有获取该 EA 值所需的权限而失败。（如果另一个进程在*listxattr()*和*getxattr()*之间删除了一个属性，也可能发生类似的失败。）

#### 示例程序

示例 16-1 中的程序检索并显示命令行中列出文件的所有 EA 的名称和值。对于每个文件，程序使用*listxattr()*来检索与文件关联的所有 EA 名称，然后执行一个循环，每次调用*getxattr()*来检索相应的值。默认情况下，属性值以纯文本形式显示。如果提供了*-x*选项，则属性值将以十六进制字符串显示。以下是该程序使用的 Shell 会话日志：

```
`$ setfattr -n user.x -v "The past is not dead." tfile`
`$ setfattr -n user.y -v "In fact, it's not even past." tfile`
`$ ./xattr_view tfile`
tfile:
        name=user.x; value=The past is not dead.
        name=user.y; value=In fact, it's not even past.
```

示例 16-1. 显示文件扩展属性

```
`xattr/xattr_view.c`
#include <sys/xattr.h>
#include "tlpi_hdr.h"

#define XATTR_SIZE 10000

static void
usageError(char *progName)
{
    fprintf(stderr, "Usage: %s [-x] file...\n", progName);
    exit(EXIT_FAILURE);
}

int
main(int argc, char *argv[])
{
    char list[XATTR_SIZE], value[XATTR_SIZE];
    ssize_t listLen, valueLen;
    int ns, j, k, opt;
    Boolean hexDisplay;

    hexDisplay = 0;
    while ((opt = getopt(argc, argv, "x")) != -1) {
        switch (opt) {
        case 'x': hexDisplay = 1;       break;
        case '?': usageError(argv[0]);
        }
    }

    if (optind >= argc + 2)
        usageError(argv[0]);
    for (j = optind; j < argc; j++) {
        listLen = listxattr(argv[j], list, XATTR_SIZE);
        if (listLen == -1)
            errExit("listxattr");

        printf("%s:\n", argv[j]);

        /* Loop through all EA names, displaying name + value */

        for (ns = 0; ns < listLen; ns += strlen(&list[ns]) + 1) {
            printf("        name=%s; ", &list[ns]);

            valueLen = getxattr(argv[j], &list[ns], value, XATTR_SIZE);
            if (valueLen == -1) {
                printf("couldn't get value");
            } else if (!hexDisplay) {
                printf("value=%.*s", (int) valueLen, value);
            } else {
                printf("value=");
                for (k = 0; k < valueLen; k++)
                    printf("%02x ", (unsigned int) value[k]);
            }

            printf("\n");
        }

        printf("\n");
    }

    exit(EXIT_SUCCESS);
}
      `xattr/xattr_view.c`
```

## 总结

从版本 2.6 开始，Linux 支持扩展属性，允许将任意元数据与文件关联，形式为名称-值对。

## 练习

1.  编写一个程序，用于创建或修改文件的*用户*EA（即，*setfattr(1)*的简单版本）。文件名、EA 名称和值应作为命令行参数传递给程序。
