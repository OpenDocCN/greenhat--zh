## 第四十章 登录账户管理

登录账户管理主要关注记录哪些用户当前登录系统，以及记录过去的登录和登出信息。本章讨论登录账户管理文件以及用于检索和更新这些文件内容的库函数。我们还描述了提供登录服务的应用程序在用户登录和登出时，应该执行哪些步骤来更新这些文件。

## `utmp` 和 `wtmp` 文件概述

UNIX 系统维护着两个数据文件，记录有关用户登录和登出的信息：

+   `utmp` 文件记录了当前登录系统的用户信息（以及稍后会描述的其他信息）。每当有用户登录时，都会在 `utmp` 文件中写入一条记录。记录中的一个字段 *ut_user* 记录了该用户的登录名。用户登出时，这条记录会被删除。诸如 *who(1)* 之类的程序会使用 `utmp` 文件中的信息来显示当前登录的用户列表。

+   `wtmp` 文件记录了所有用户的登录和登出日志（以及一些稍后会描述的其他信息）。每次登录时，包含与写入 `utmp` 文件相同信息的记录都会被追加到 `wtmp` 文件中。登出时，另一个记录会被追加到文件中。这个记录包含相同的信息，除了 *ut_user* 字段被清零。可以使用 *last(1)* 命令来显示和过滤 `wtmp` 文件的内容。

在 Linux 系统中，`utmp` 文件位于 `/var/run/utmp`，`wtmp` 文件位于 `/var/log/wtmp`。一般来说，应用程序不需要知道这些路径，因为它们已经被编译进了 *glibc* 中。需要引用这些文件位置的程序应该使用 `_PATH_UTMP` 和 `_PATH_WTMP` 路径常量，这些常量定义在 `<paths.h>`（和 `<utmpx.h>`）中，而不是将路径硬编码到程序中。

### 注意

SUSv3 并没有为 `utmp` 和 `wtmp` 文件的路径标准化任何符号名称。`_PATH_UTMP` 和 `_PATH_WTMP` 这些名称在 Linux 和 BSD 系统中使用。许多其他 UNIX 实现则定义了常量 `UTMP_FILE` 和 `WTMP_FILE` 来表示这些路径。Linux 也在 `<utmp.h>` 中定义了这些名称，但并未在 `<utmpx.h>` 或 `<paths.h>` 中定义它们。

## *utmpx* API

`utmp` 和 `wtmp` 文件自 UNIX 系统早期以来就存在，但随着不同 UNIX 实现的演变，尤其是 BSD 与 System V 的差异，它们经历了持续的发展和分化。System V Release 4 极大扩展了 API，在此过程中创建了一个新的（平行的）*utmpx* 结构以及相关的 `utmpx` 和 `wtmpx` 文件。字母 *x* 同样被包含在头文件和处理这些新文件的附加函数的名称中。许多其他 UNIX 实现也对 API 添加了自己的扩展。

在本章中，我们将介绍 Linux 的 *utmpx* API，它是 BSD 和 System V 实现的混合体。Linux 并没有像 System V 那样创建并行的 `utmpx` 和 `wtmpx` 文件；相反，`utmp` 和 `wtmp` 文件包含所有所需的信息。然而，为了与其他 UNIX 实现兼容，Linux 提供了传统的 *utmp* 和基于 System V 的 *utmpx* API 来访问这些文件的内容。在 Linux 上，这两个 API 返回完全相同的信息。（这两个 API 之间为数不多的差异之一是，*utmp* API 包含一些函数的可重入版本，而 *utmpx* API 则没有。）然而，我们将讨论限定于 utmpx 接口，因为它是 SUSv3 规范中指定的 API，因此更适合于移植到其他 UNIX 实现。

SUSv3 规范没有涵盖 `utmpx` API 的所有方面（例如，`utmp` 和 `wtmp` 文件的位置未指定）。登录记账文件的精确内容在不同实现中有所不同，而且各种实现提供了未在 SUSv3 中指定的附加登录记账功能。

### 注意

第十七章 [Frisch, 2002] 总结了不同 UNIX 实现中 `wtmp` 和 `utmp` 文件的存储位置及使用上的一些变体。它还描述了 *ac(1)* 命令的使用，该命令可用于总结 `wtmp` 文件中的登录信息。

## *utmpx* 结构体

`utmp` 和 `wtmp` 文件由 *utmpx* 记录组成。*utmpx* 结构体在 `<utmpx.h>` 中定义，如 示例 40-1 所示。

### 注意

SUSv3 规范中的 *utmpx* 结构体不包括 *ut_host*、*ut_exit*、*ut_session* 或 *ut_addr_v6* 字段。*ut_host* 和 *ut_exit* 字段在大多数其他实现中都有；*ut_session* 字段在一些其他实现中存在；*ut_addr_v6* 是 Linux 特有的。SUSv3 规范中指定了 *ut_line* 和 *ut_user* 字段，但未指定其长度。

用于定义 *utmpx* 结构体中 *ut_addr_v6* 字段的 *int32_t* 数据类型是一个 32 位整数。

示例 40-1. *utmpx* 结构体定义

```
#define _GNU_SOURCE             /* Without _GNU_SOURCE the two field
struct exit_status {               names below are prepended by "__" */
    short e_termination;        /* Process termination status (signal) */
    short e_exit;               /* Process exit status */
};

#define __UT_LINESIZE    32
#define __UT_NAMESIZE    32
#define __UT_HOSTSIZE   256

struct utmpx {
    short ut_type;                      /* Type of record */
    pid_t ut_pid;                       /* PID of login process */
    char  ut_line[__UT_LINESIZE];       /* Terminal device name */
    char  ut_id[4];                     /* Suffix from terminal name, or
                                           ID field from inittab(5) */
    char  ut_user[__UT_NAMESIZE];       /* Username */
    char  ut_host[__UT_HOSTSIZE];       /* Hostname for remote login, or kernel
                                           version for run-level messages */
    struct exit_status ut_exit;         /* Exit status of process marked
                                           as DEAD_PROCESS (not filled
                                           in by init(8) on Linux) */
    long  ut_session;                   /* Session ID */
    struct timeval ut_tv;               /* Time when entry was made */
    int32_t ut_addr_v6[4];              /* IP address of remote host (IPv4
                                           address uses just ut_addr_v6[0],
                                           with other elements set to 0) */
    char __unused[20];                  /* Reserved for future use */
};
```

*utmpx* 结构体中的每个字符串字段都以空字符结尾，除非它完全填充了相应的数组。

对于登录进程，存储在 *ut_line* 和 *ut_id* 字段中的信息来源于终端设备的名称。*ut_line* 字段包含终端设备的完整文件名。*ut_id* 字段包含文件名的后缀部分——即 *tty*、*pts* 或 *pty* 后面的字符串（后两个是用于 System-V 和 BSD 风格的伪终端）。因此，对于终端 `/dev/tty2`，*ut_line* 将是 *tty2*，而 *ut_id* 将是 *2*。

在窗口环境中，一些终端仿真器使用 *ut_session* 字段记录终端窗口的会话 ID。（有关会话 ID 的解释，请参见 Sessions。）

*ut_type* 字段是一个整数，用于定义写入文件的记录类型。以下常量集合（其对应的数字值见括号内）可作为该字段的值：

`EMPTY` (0)

该记录不包含有效的记账信息。

`RUN_LVL` (1)

该记录表示系统在启动或关机过程中运行级别的变化。（有关运行级别的信息，请参见 *init(8)* 手册页面。）必须定义 `_GNU_SOURCE` 特性测试宏，才能从 `<utmpx.h>` 中获取该常量的定义。

`BOOT_TIME` (2)

该记录包含系统启动时间，记录在 *ut_tv* 字段中。`RUN_LVL` 和 `BOOT_TIME` 记录通常由 *init* 生成。这些记录会写入 `utmp` 文件和 `wtmp` 文件。

`NEW_TIME` (3)

该记录包含系统时钟更改后的新时间，记录在 *ut_tv* 字段中。

`OLD_TIME` (4)

该记录包含系统时钟更改前的旧时间，记录在 *ut_tv* 字段中。`OLD_TIME` 和 `NEW_TIME` 类型的记录由 NTP（或类似）守护进程在更改系统时钟时写入 `utmp` 和 `wtmp` 文件。

`INIT_PROCESS` (5)

这是一个由 *init* 生成的进程记录，例如 *getty* 进程。有关详细信息，请参见 *inittab(5)* 手册页面。

`LOGIN_PROCESS` (6)

这是一个用户登录的会话领导进程记录，如 *login(1)* 进程。

`USER_PROCESS` (7)

这是一个用户进程记录，通常是一个登录会话，用户名会出现在 *ut_user* 字段中。该登录会话可能是由 *login(1)* 启动的，也可能是由某个提供远程登录功能的应用程序启动的，比如 *ftp* 或 *ssh*。

`DEAD_PROCESS` (8)

该记录标识一个已退出的进程。

我们展示这些常量的数字值，因为不同的应用程序依赖于这些常量的数字顺序。例如，在 *agetty* 程序的源代码中，我们可以看到类似以下的检查：

```
utp->ut_type >= INIT_PROCESS && utp->ut_type <= DEAD_PROCESS
```

`INIT_PROCESS` 类型的记录通常对应于 *getty(8)*（或类似程序，如 *agetty(8)* 或 *mingetty(8)*）的调用。在系统启动时，*init* 进程为每个终端行和虚拟控制台创建一个子进程，每个子进程执行 *getty* 程序。*getty* 程序打开终端，提示用户输入登录名，然后执行 *login(1)*。在成功验证用户并执行其他各种操作后，*login* 会生成一个子进程，该进程执行用户的登录 shell。一个完整的登录会话生命周期会通过以下四条记录写入 `wtmp` 文件，顺序如下：

+   由 *init* 写入的 `INIT_PROCESS` 记录；

+   由 *getty* 写入的 `LOGIN_PROCESS` 记录；

+   一个由*login*写入的`USER_PROCESS`记录；以及

+   一个由*init*写入的`DEAD_PROCESS`记录，当它检测到子进程*login*（用户注销时发生）终止时。

有关*getty*和*login*在用户登录期间操作的更多细节，请参见第九章，[Stevens & Rago, 2005]。

### 注意

某些版本的*init*在更新`wtmp`文件之前启动*getty*进程。因此，*init*和*getty*会相互竞争更新`wtmp`文件，结果是`INIT_PROCESS`和`LOGIN_PROCESS`记录的写入顺序可能与正文中描述的顺序相反。

## 从`utmp`和`wtmp`文件中检索信息

本节中描述的函数从包含*utmpx*格式记录的文件中检索记录。默认情况下，这些函数使用标准的`utmp`文件，但可以使用*utmpxname()*函数（如下所述）进行更改。

这些函数采用文件中的*当前位置*的概念来检索记录。每个函数都会更新该位置。

*setutxent()*函数将`utmp`文件回到文件开头。

```
#include <utmpx.h>

void `setutxent`(void);
```

通常，在使用任何*getutx*()函数（下面描述）之前，我们应该调用*setutxent()*。这样可以防止如果我们调用的某些第三方函数之前已经使用了这些函数而导致的可能混乱。根据执行的任务，可能还需要在程序的适当位置再次调用*setutxent()*。

*setutxent()*函数和*getutx*()函数在`utmp`文件尚未打开时，会打开该文件。当我们完成使用该文件后，可以通过*endutxent()*函数关闭它。

```
#include <utmpx.h>

void `endutxent`(void);
```

*getutxent()*、*getutxid()*和*getutxline()*函数从`utmp`文件中读取一条记录，并返回指向（静态分配的）*utmpx*结构的指针。

```
#include <utmpx.h>

struct utmpx *`getutxent`(void);
struct utmpx *`getutxid`(const struct utmpx **ut*);
struct utmpx *`getutxline`(const struct utmpx **ut*);
```

### 注意

所有函数都返回指向静态分配的*utmpx*结构的指针，或者如果没有匹配的记录或遇到 EOF，则返回`NULL`。

*getutxent()*函数从`utmp`文件中检索下一个顺序记录。*getutxid()*和*getutxline()*函数从当前位置开始搜索，与*utmpx*结构中由*ut*参数指向的条件匹配的记录。

*getutxid()*函数根据*ut*参数中*ut_type*和*ut_id*字段指定的值，在`utmp`文件中搜索记录：

+   如果*ut_type*字段是`RUN_LVL`、`BOOT_TIME`、`NEW_TIME`或`OLD_TIME`，则*getutxid()*会查找下一个其*ut_type*字段与指定值匹配的记录。（这些类型的记录与用户登录无关。）这允许查找系统时间和运行级别变化的记录。

+   如果 *ut_type* 字段是剩余的有效值之一（`INIT_PROCESS`、`LOGIN_PROCESS`、`USER_PROCESS` 或 `DEAD_PROCESS`），则 *getutxent()* 会查找下一个记录，其 *ut_type* 字段匹配 *任何* 这些值，并且其 *ut_id* 字段与 *ut* 参数中指定的值匹配。这允许扫描文件中与特定终端相关的记录。

*getutxline()* 函数会向前搜索一个记录，其 *ut_type* 字段为 `LOGIN_PROCESS` 或 `USER_PROCESS`，并且其 *ut_line* 字段与 *ut* 参数中指定的值匹配。这对于查找与用户登录相关的记录非常有用。  

如果搜索失败（即在遇到文件结束符时未找到匹配记录），则 *getutxid()* 和 *getutxline()* 都会返回 `NULL`。  

在某些 UNIX 实现中，*getutxline()* 和 *getutxid()* 将用于返回 *utmpx* 结构的静态区域视为一种缓存。如果它们确定由先前的 *getutx*() 调用放入此缓存的记录与 *ut* 中指定的标准匹配，则不会执行文件读取操作；调用仅会再次返回相同的记录（SUSv3 允许这种行为）。因此，为了防止在循环中多次调用 *getutxline()* 和 *getutxid()* 时重复返回相同的记录，我们必须清除这个静态数据结构，可以使用如下代码：  

```
struct utmpx *res = NULL;

/* Other code omitted */

if (res != NULL)            /* If 'res' was set via a previous call */
    memset(res, 0, sizeof(struct utmpx));
res = getutxline(&ut);
```

*glibc* 实现不执行这种类型的缓存，但为了可移植性，我们仍然应该采用这种技术。  

### 注意  

因为 *getutx*() 函数返回一个指向静态分配结构的指针，所以它们是不可重入的。GNU C 库提供了传统的 *utmp* 函数的重入版本（*getutent_r()*、*getutid_r()* 和 *getutline_r()*），但没有提供它们的 *utmpx* 对应函数的重入版本。（SUSv3 并未指定重入版本。）  

默认情况下，所有 *getutx*() 函数都作用于标准的 `utmp` 文件。如果我们想使用其他文件，如 `wtmp` 文件，则必须首先调用 *utmpxname()*，指定所需的路径名。  

```
#define _GNU_SOURCE
#include <utmpx.h>

int `utmpxname`(const char **file*);
```

### 注意  

成功时返回 0，错误时返回 -1  

*utmpxname()* 函数仅记录给定路径名的副本。它不会打开文件，但会关闭任何之前由其他调用打开的文件。这意味着，如果指定了无效的路径名，*utmpxname()* 不会返回错误。相反，当稍后调用某个 *getutx*() 函数时，如果未能打开文件，它将返回错误（即 `NULL`，并将 *errno* 设置为 `ENOENT`）。  

### 注意  

尽管 SUSv3 中未指定，但大多数 UNIX 实现都提供 *utmpxname()* 或类似的 *utmpname()* 函数。  

#### 示例程序  

示例 40-2 中的程序使用了本节中描述的一些函数来转储 *utmpx* 格式文件的内容。以下 shell 会话日志演示了当我们使用该程序转储 `/var/run/utmp`（如果未调用 *utmpxname()*，这些函数使用的默认值）内容时的结果：

```
$ `./dump_utmpx`
user     type        PID line   id  host      date/time
LOGIN    LOGIN_PR   1761 tty1   1             Sat Oct 23 09:29:37 2010
LOGIN    LOGIN_PR   1762 tty2   2             Sat Oct 23 09:29:37 2010
lynley   USER_PR   10482 tty3   3             Sat Oct 23 10:19:43 2010
david    USER_PR    9664 tty4   4             Sat Oct 23 10:07:50 2010
liz      USER_PR    1985 tty5   5             Sat Oct 23 10:50:12 2010
mtk      USER_PR   10111 pts/0  /0            Sat Oct 23 09:30:57 2010
```

为了简洁起见，我们编辑掉了程序生成的大部分输出。匹配 `tty1` 到 `tty5` 的行是虚拟控制台（`/dev/tty[1-6]`）上的登录记录。最后一行输出是伪终端上的 *xterm* 会话。

以下通过转储 `/var/log/wtmp` 所生成的输出显示了当用户登录和注销时，两个记录会被写入 `wtmp` 文件。（我们编辑掉了程序生成的其他输出。）通过顺序搜索 `wtmp` 文件（使用 *getutxline()*），这些记录可以通过 *ut_line* 字段匹配。

```
$ `./dump_utmpx /var/log/wtmp`
user     type        PID line   id  host      date/time
lynley   USER_PR   10482 tty3   3             Sat Oct 23 10:19:43 2010
         DEAD_PR   10482 tty3   3   2.4.20-4G Sat Oct 23 10:32:54 2010
```

示例 40-2. 显示 *utmpx* 格式文件的内容

```
`loginacct/dump_utmpx.c`
#define _GNU_SOURCE
#include <time.h>
#include <utmpx.h>
#include <paths.h>
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    struct utmpx *ut;

    if (argc > 1 && strcmp(argv[1], "--help") == 0)
        usageErr("%s [utmp-pathname]\n", argv[0]);

    if (argc > 1)               /* Use alternate file if supplied */
        if (utmpxname(argv[1]) == -1)
            errExit("utmpxname");

    setutxent();

    printf("user     type        PID line   id  host      date/time\n");

    while ((ut = getutxent()) != NULL) {        /* Sequential scan to EOF */
        printf("%-8s ", ut->ut_user);
        printf("%-9.9s ",
                (ut->ut_type == EMPTY) ?         "EMPTY" :
                (ut->ut_type == RUN_LVL) ?       "RUN_LVL" :
                (ut->ut_type == BOOT_TIME) ?     "BOOT_TIME" :
                (ut->ut_type == NEW_TIME) ?      "NEW_TIME" :
                (ut->ut_type == OLD_TIME) ?      "OLD_TIME" :
                (ut->ut_type == INIT_PROCESS) ?  "INIT_PR" :
                (ut->ut_type == LOGIN_PROCESS) ? "LOGIN_PR" :
                (ut->ut_type == USER_PROCESS) ?  "USER_PR" :
                (ut->ut_type == DEAD_PROCESS) ?  "DEAD_PR" : "???");
        printf("%5ld %-6.6s %-3.5s %-9.9s ", (long) ut->ut_pid,
                ut->ut_line, ut->ut_id, ut->ut_host);
        printf("%s", ctime((time_t *) &(ut->ut_tv.tv_sec)));
    }

    endutxent();
    exit(EXIT_SUCCESS);
}
     `loginacct/dump_utmpx.c`
```

## 获取登录名：*getlogin()*

*getlogin()* 函数返回在调用进程的控制终端上登录的用户的用户名。该函数使用保存在 `utmp` 文件中的信息。

```
#include <unistd.h>

char *`getlogin`(void);
```

### 注意

返回指向用户名字符串的指针，或在发生错误时返回 `NULL`

*getlogin()* 函数调用 *ttyname()* (终端标识) 来查找与调用进程的标准输入相关联的终端的名称。然后，它在 `utmp` 文件中搜索一个记录，该记录的 *ut_line* 值与该终端名称匹配。如果找到匹配的记录，*getlogin()* 会返回该记录中的 *ut_user* 字符串。

如果未找到匹配项或发生错误，*getlogin()* 会返回 `NULL` 并设置 *errno* 以指示错误。*getlogin()* 失败的一个原因可能是进程没有与标准输入相关联的终端（`ENOTTY`），可能是因为它是守护进程。另一个可能的原因是该终端会话未记录在 `utmp` 中；例如，一些软件终端模拟器不会在 `utmp` 文件中创建条目。

即使在用户 ID 在 `/etc/passwd` 中有多个登录名的（不常见）情况下，*getlogin()* 仍然能够返回实际用于在该终端登录的用户名，因为它依赖于 `utmp` 文件。相比之下，使用 *getpwuid(getuid())* 总是从 `/etc/passwd` 中检索到第一个匹配的记录，而不管登录时使用的是什么名字。

### 注意

*getlogin()* 的可重入版本由 SUSv3 规定，形式为 *getlogin_r()*，该函数由 *glibc* 提供。

`LOGNAME` 环境变量也可以用来查找用户的登录名。然而，该变量的值可以被用户更改，这意味着它不能用于安全地标识用户。

## 更新登录会话的 `utmp` 和 `wtmp` 文件

在编写创建登录会话的应用程序时（例如像 *login* 或 *sshd* 这样的程序），我们应当按照如下方式更新 `utmp` 和 `wtmp` 文件：

+   在登录时，应向 `utmp` 文件写入一条记录，表示该用户已登录。应用程序必须检查该终端的记录是否已经存在于 `utmp` 文件中。如果存在先前的记录，则覆盖该记录；否则，追加一条新记录到文件中。通常，调用 *pututxline()*（稍后将描述）足以确保这些步骤正确执行（有关示例，请参见 Example 40-3）。输出的 *utmpx* 记录应至少填写 *ut_type*、*ut_user*、*ut_tv*、*ut_pid*、*ut_id* 和 *ut_line* 字段。*ut_type* 字段应设置为 `USER_PROCESS`。*ut_id* 字段应包含设备名称的后缀（即用户登录的终端或伪终端的名称），而 *ut_line* 字段应包含登录设备的名称，并去掉前缀 `/dev/` 字符串。（这些字段内容的示例可以参见程序在 Example 40-2 中的示例运行。）一条包含完全相同信息的记录会追加到 `wtmp` 文件中。

    ### 注意

    终端名称（通过 *ut_line* 和 *ut_id* 字段）充当 `utmp` 文件中记录的唯一标识符。

+   在登出时，之前写入 `utmp` 文件的记录应该被删除。这是通过创建一条 *ut_type* 设置为 `DEAD_PROCESS` 的记录来完成的，记录中的 *ut_id* 和 *ut_line* 值与登录时写入的记录相同，但 *ut_user* 字段被置零。这条记录将覆盖先前的记录。相同的记录副本会追加到 `wtmp` 文件中。

    ### 注意

    如果我们未能在登出时清理 *utmp* 记录，可能是因为程序崩溃，那么在下次重启时，*init* 会自动清理该记录，将其 *ut_type* 设置为 `DEAD_PROCESS`，并将记录的其他字段置零。

`utmp` 和 `wtmp` 文件通常是受保护的，只有特权用户才能对这些文件进行更新。*getlogin()* 的准确性取决于 `utmp` 文件的完整性。因此，出于这个原因以及其他原因，`utmp` 和 `wtmp` 文件的权限绝不能设置为允许非特权用户写入。

什么算作登录会话？正如我们所料，*login*、*telnet* 和 *ssh* 登录会话会被记录在登录账务文件中。大多数 *ftp* 实现也会创建登录账务记录。然而，系统上每个启动的终端窗口，或者像 *su* 这样的调用，是否也会创建登录账务记录呢？这个问题的答案在不同的 UNIX 实现中有所不同。

### 注意

在某些终端仿真程序（例如 *xterm*）下，可以使用命令行选项或其他机制来确定程序是否更新登录记账文件。

*pututxline()* 函数将 *utmpx* 结构（由 *ut* 指向）写入 `/var/run/utmp` 文件（如果之前调用了 *utmpxname()*，则写入替代文件）。

```
#include <utmpx.h>

struct utmpx *`pututxline`(const struct utmpx **ut*);
```

### 注意

成功时返回指向已成功更新记录的指针，出错时返回 `NULL`

在写入记录之前，*pututxline()* 首先使用 *getutxid()* 向前查找可能被覆盖的记录。如果找到这样的记录，它将被覆盖；否则，一个新记录将追加到文件的末尾。在许多情况下，应用程序会在调用 *pututxline()* 之前调用其中一个 *getutx*() 函数，该函数将当前文件位置设置为正确的记录——即匹配 *getutxid()* 风格条件的 *utmpx* 结构，该结构由 *ut* 指向。如果 *pututxline()* 确定这种情况发生，它就不会调用 *getutxid()*。

### 注意

如果 *pututxline()* 内部调用了 *getutxid()*，此调用不会改变 *getutx*() 函数返回的 *utmpx* 结构所使用的静态区域。SUSv3 要求实现必须具备这种行为。

更新 `wtmp` 文件时，我们只需打开文件并追加记录。由于这是一个标准操作，*glibc* 将其封装在 *updwtmpx()* 函数中。

```
#define _GNU_SOURCE
#include <utmpx.h>

void `updwtmpx`(char **wtmpx_file*, struct utmpx **ut*);
```

*updwtmpx()* 函数将 *ut* 指向的 *utmpx* 记录追加到 *wtmpx_file* 指定的文件中。

SUSv3 没有指定 *updwtmpx()*，它只出现在一些其他 UNIX 实现中。其他实现提供相关函数——*login(3)*、*logout(3)* 和 *logwtmp(3)*——这些函数也包含在 *glibc* 中，并在手册页中描述。如果没有这些函数，我们需要编写自己的等效函数。（这些函数的实现并不复杂。）

#### 示例程序

示例 40-3 使用本节描述的函数来更新 `utmp` 和 `wtmp` 文件。此程序执行所需的更新操作，以便在命令行上登录指定的用户，然后在休眠几秒钟后将其注销。通常，这样的操作与用户登录会话的创建和终止相关联。此程序使用 *ttyname()* 来检索与文件描述符关联的终端设备的名称。我们在 终端识别 中描述了 *ttyname()*。

以下 shell 会话日志演示了 示例 40-3 中程序的操作。我们假设具备权限以便能够更新登录记账文件，然后使用该程序为用户 *mtk* 创建一个记录：

```
$ `su`
Password:
# `./utmpx_login mtk`
Creating login entries in utmp and wtmp
        using pid 1471, line pts/7, id /7
*Type Control-Z to suspend program*
[1]+  Stopped                 ./utmpx_login mtk
```

当 *utmpx_login* 程序正在休眠时，我们按下了 *Control-Z*，将程序挂起并推入后台。接下来，我们使用 示例 40-2 中的程序来检查 `utmp` 文件的内容：

```
# `./dump_utmpx /var/run/utmp`
user     type        PID line   id  host      date/time
cecilia  USER_PR     249 tty1   1             Fri Feb  1 21:39:07 2008
mtk      USER_PR    1471 pts/7  /7            Fri Feb  1 22:08:06 2008
# `who`
cecilia  tty1     Feb  1 21:39
mtk      pts/7    Feb  1 22:08
```

上面，我们使用 *who(1)* 命令显示 *who* 的输出来自 *utmp*。

接下来，我们使用我们的程序来检查 `wtmp` 文件的内容：

```
# `./dump_utmpx /var/log/wtmp`
user     type        PID line   id  host      date/time
cecilia  USER_PR     249 tty1   1             Fri Feb  1 21:39:07 2008
mtk      USER_PR    1471 pts/7  /7            Fri Feb  1 22:08:06 2008
# `last mtk`
mtk      pts/7                      Fri Feb  1 22:08   still logged in
```

上面，我们使用了 *last(1)* 命令来显示 `last` 的输出来自 `wtmp`。（为了简洁，我们已编辑此 shell 会话日志中的 *dump_utmpx* 和 *last* 命令的输出，去除了与我们讨论无关的输出行。）

接下来，我们使用 *fg* 命令将 *utmpx_login* 程序恢复到前台。随后，它将注销记录写入 `utmp` 和 `wtmp` 文件。

```
# `fg`
./utmpx_login mtk
Creating logout entries in utmp and wtmp
```

然后我们再次检查 `utmp` 文件的内容。我们看到 `utmp` 记录被覆盖了：

```
# `./dump_utmpx /var/run/utmp`
user     type        PID line   id  host      date/time
cecilia  USER_PR     249 tty1   1             Fri Feb  1 21:39:07 2008
         DEAD_PR    1471 pts/7  /7            Fri Feb  1 22:09:09 2008
# `who`
cecilia  tty1     Feb  1 21:39
```

最后一行输出显示 *who* 忽略了 `DEAD_PROCESS` 记录。

当我们检查 `wtmp` 文件时，我们看到 `wtmp` 记录被替代了：

```
# `./dump_utmpx /var/log/wtmp`
user     type        PID line   id  host      date/time
cecilia  USER_PR     249 tty1   1             Fri Feb  1 21:39:07 2008
mtk      USER_PR    1471 pts/7  /7            Fri Feb  1 22:08:06 2008
         DEAD_PR    1471 pts/7  /7            Fri Feb  1 22:09:09 2008
# `last mtk`
mtk      pts/7                      Fri Feb  1 22:08 - 22:09  (00:01)
```

上面的最后一行输出演示了 *last* 如何匹配 `wtmp` 中的登录和注销记录，以显示已完成登录会话的开始和结束时间。

示例 40-3. 更新 `utmp` 和 `wtmp` 文件

```
`loginacct/utmpx_login.c`
#define _GNU_SOURCE
#include <time.h>
#include <utmpx.h>
#include <paths.h>              /* Definitions of _PATH_UTMP and _PATH_WTMP */
#include "tlpi_hdr.h"
int
main(int argc, char *argv[])
{
    struct utmpx ut;
    char *devName;

    if (argc < 2 || strcmp(argv[1], "--help") == 0)
        usageErr("%s username [sleep-time]\n", argv[0]);

    /* Initialize login record for utmp and wtmp files */

    memset(&ut, 0, sizeof(struct utmpx));
    ut.ut_type = USER_PROCESS;          /* This is a user login */
    strncpy(ut.ut_user, argv[1], sizeof(ut.ut_user));
    if (time((time_t *) &ut.ut_tv.tv_sec) == -1)
        errExit("time");                /* Stamp with current time */
    ut.ut_pid = getpid();

    /* Set ut_line and ut_id based on the terminal associated with
       'stdin'. This code assumes terminals named "/dev/[pt]t[sy]*".
       The "/dev/" dirname is 5 characters; the "[pt]t[sy]" filename
       prefix is 3 characters (making 8 characters in all). */

    devName = ttyname(STDIN_FILENO);
    if (devName == NULL)
        errExit("ttyname");
    if (strlen(devName) <= 8)           /* Should never happen */
        fatal("Terminal name is too short: %s", devName);

    strncpy(ut.ut_line, devName + 5, sizeof(ut.ut_line));
    strncpy(ut.ut_id, devName + 8, sizeof(ut.ut_id));

    printf("Creating login entries in utmp and wtmp\n");
    printf("        using pid %ld, line %.*s, id %.*s\n",
            (long) ut.ut_pid, (int) sizeof(ut.ut_line), ut.ut_line,
            (int) sizeof(ut.ut_id), ut.ut_id);

    setutxent();                        /* Rewind to start of utmp file */
    if (pututxline(&ut) == NULL)        /* Write login record to utmp */
        errExit("pututxline");
    updwtmpx(_PATH_WTMP, &ut);          /* Append login record to wtmp */

    /* Sleep a while, so we can examine utmp and wtmp files */

    sleep((argc > 2) ? getInt(argv[2], GN_NONNEG, "sleep-time") : 15);

    /* Now do a "logout"; use values from previously initialized 'ut',
       except for changes below */

    ut.ut_type = DEAD_PROCESS;          /* Required for logout record */
    time((time_t *) &ut.ut_tv.tv_sec);  /* Stamp with logout time */
    memset(&ut.ut_user, 0, sizeof(ut.ut_user));
                                        /* Logout record has null username */
    printf("Creating logout entries in utmp and wtmp\n");
    setutxent();                        /* Rewind to start of utmp file */
    if (pututxline(&ut) == NULL)        /* Overwrite previous utmp record */
        errExit("pututxline");
    updwtmpx(_PATH_WTMP, &ut);          /* Append logout record to wtmp */

    endutxent();
    exit(EXIT_SUCCESS);
}
      `loginacct/utmpx_login.c`
```

## `lastlog` 文件

`lastlog` 文件记录了每个用户最后一次登录系统的时间。（这与 `wtmp` 文件不同，后者记录了所有用户的所有登录和注销。）其中，`lastlog` 文件使得 *login* 程序能够在用户开始新登录会话时通知他们最后一次登录的时间。除了更新 `utmp` 和 `wtmp` 文件外，提供登录服务的应用程序还应该更新 `lastlog`。

与 `utmp` 和 `wtmp` 文件一样，`lastlog` 文件的位置和格式有所不同。（一些 UNIX 实现没有提供此文件。）在 Linux 上，该文件位于 `/var/log/lastlog`，并且在 `<paths.h>` 中定义了一个常量 `_PATH_LASTLOG` 来指向此位置。与 `utmp` 和 `wtmp` 文件一样，`lastlog` 文件通常是受保护的，使得所有用户可以读取，但只有特权进程才能更新。

`lastlog` 文件中的记录具有以下格式（定义在 `<lastlog.h>` 中）：

```
#define UT_NAMESIZE           32
#define UT_HOSTSIZE          256

struct lastlog {
    time_t ll_time;                     /* Time of last login */
    char   ll_line[UT_NAMESIZE];        /* Terminal for remote login */
    char   ll_host[UT_HOSTSIZE];        /* Hostname for remote login */
};
```

请注意，这些记录不包括用户名或用户 ID。相反，`lastlog` 文件由一系列按用户 ID 索引的记录组成。因此，要找到用户 ID 为 1000 的 `lastlog` 记录，我们需要定位到文件的第 *(1000 * sizeof(struct lastlog))* 字节。这在 示例 40-4 中演示了，程序允许我们查看命令行上列出的用户的 `lastlog` 记录。这与 *lastlog(1)* 命令提供的功能相似。以下是运行此程序时输出的示例：

```
$ `./view_lastlog annie paulh`
annie    tty2                        Mon Jan 17 11:00:12 2011
paulh    pts/11                      Sat Aug 14 09:22:14 2010
```

更新 `lastlog` 文件的方法与打开文件、定位到正确位置并进行写入相似。

### 注释

由于 `lastlog` 文件是按用户 ID 索引的，因此无法区分具有相同用户 ID 的不同用户名的登录。（在 密码文件：`/etc/passwd` 中，我们提到过，虽然不常见，但可以有多个登录名对应相同的用户 ID。）

示例 40-4. 从 `lastlog` 文件中显示信息

```
`loginacct/view_lastlog.c`
#include <time.h>
#include <lastlog.h>
#include <paths.h>                      /* Definition of _PATH_LASTLOG */
#include <fcntl.h>
#include "ugid_functions.h"             /* Declaration of userIdFromName() */
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    struct lastlog llog;
    int fd, j;
    uid_t uid;

    if (argc > 1 && strcmp(argv[1], "--help") == 0)
        usageErr("%s [username...]\n", argv[0]);

    fd = open(_PATH_LASTLOG, O_RDONLY);
    if (fd == -1)
        errExit("open");

    for (j = 1; j < argc; j++) {
        uid = userIdFromName(argv[j]);
        if (uid == -1) {
            printf("No such user: %s\n", argv[j]);
            continue;
        }

        if (lseek(fd, uid * sizeof(struct lastlog), SEEK_SET) == -1)
            errExit("lseek");

        if (read(fd, &llog, sizeof(struct lastlog)) <= 0) {
            printf("read failed for %s\n", argv[j]);    /* EOF or error */
            continue;
        }

        printf("%-8.8s %-6.6s %-20.20s %s", argv[j], llog.ll_line,
                llog.ll_host, ctime((time_t *) &llog.ll_time));
    }

    close(fd);
    exit(EXIT_SUCCESS);
}
      `loginacct/view_lastlog.c`
```

## 总结

登录记录会记录当前登录的用户以及所有过去的登录信息。这些信息保存在三个文件中：`utmp` 文件，记录所有当前登录的用户；`wtmp` 文件，是所有登录和登出的审计跟踪；以及 `lastlog` 文件，记录每个用户的最后登录时间。各种命令，如 *who* 和 *last*，都使用这些文件中的信息。

C 库提供了用于检索和更新登录记录文件信息的函数。提供登录服务的应用程序应使用这些函数来更新登录记录文件，以便依赖这些信息的命令能够正确运行。

#### 更多信息

除了 *utmp(5)* 手册页面，关于登录记录功能的更多信息，最有用的地方是那些使用这些功能的各种应用程序的源代码。例如，可以查看 *mingetty*（或 *agetty*）、*login*、*init*、*telnet*、*ssh* 和 *ftp* 的源代码。

## 练习

1.  实现 *getlogin()*。如在 获取登录名：*getlogin()*") 中所述，*getlogin()* 可能无法正确工作，尤其是在某些软件终端模拟器下运行的进程中；在这种情况下，请尝试从虚拟控制台进行测试。

1.  修改 示例 40-3 (`utmpx_login.c`) 中的程序，使其除了更新 `utmp` 和 `wtmp` 文件外，还更新 `lastlog` 文件。

1.  阅读 *login(3)*、*logout(3)* 和 *logwtmp(3)* 的手册页面。实现这些功能。

1.  实现一个简单版本的 *who(1)* 命令。
