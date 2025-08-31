## 第八章 用户与组

每个用户都有一个唯一的登录名和一个关联的数字用户标识符（UID）。用户可以属于一个或多个组。每个组也有一个唯一的名称和一个组标识符（GID）。

用户和组 ID 的主要目的是确定各种系统资源的所有权，并控制访问这些资源的进程所授予的权限。例如，每个文件属于一个特定的用户和组，每个进程都有一个或多个用户和组 ID，这些 ID 决定了进程的所有者是谁，以及它在访问文件时拥有哪些权限（有关详细信息，请参见第九章）。

在本章中，我们将介绍用于定义系统中用户和组的系统文件，然后描述用于从这些文件中检索信息的库函数。最后，我们将讨论用于加密和验证登录密码的*crypt()*函数。

## 密码文件：`/etc/passwd`

系统的*密码文件*，`/etc/passwd`，包含系统中每个用户帐户的一行。每一行由七个字段组成，这些字段由冒号（`:`）分隔，如下例所示：

```
mtk:x:1000:100:Michael Kerrisk:/home/mtk:/bin/bash
```

依次排列，这些字段如下：

+   *登录名*：这是用户必须输入的唯一名称才能登录。通常，这也称为用户名。我们也可以将登录名视为与数字用户标识符（稍后描述）对应的人类可读（符号）标识符。当程序如*ls(1)*被要求显示文件的所有者时，它会显示此名称，而不是与文件相关联的数字用户 ID（例如，在*ls -l*中）。

+   *加密密码*：此字段包含一个 13 个字符的加密密码，我们将在 8.5 节中详细描述。如果密码字段包含任何其他字符串——特别是其他长度为 13 以外的字符串——则该帐户的登录将被禁用，因为这样的字符串不能表示有效的加密密码。需要注意的是，如果启用了影子密码（这是典型的情况），则此字段将被忽略。在这种情况下，`/etc/passwd`中的密码字段通常包含字母*x*（尽管可以出现任何非空字符字符串），而加密密码则存储在影子密码文件中（影子密码文件：`/etc/shadow`）。如果`/etc/passwd`中的密码字段为空，则登录此帐户时不需要密码（即使启用了影子密码，这也成立）。

### 注意

在这里，我们假设密码是使用数据加密标准（DES）加密的，这是一种历史悠久且仍广泛使用的 UNIX 密码加密方案。也可以用其他方案替代 DES，如 MD5，它生成一个 128 位的*消息摘要*（一种哈希值）。这个值以 34 个字符的字符串形式存储在密码（或影子密码）文件中。

+   *用户 ID*（UID）：这是该用户的数字 ID。如果此字段的值为 0，则该账户具有超级用户权限。通常，只有一个这样的账户，登录名为 *root*。在 Linux 2.2 及之前版本中，用户 ID 作为 16 位值存储，允许的范围是 0 到 65,535；在 Linux 2.4 及之后版本中，它们以 32 位存储，允许更大的范围。

### 注意

可以（但不常见）在密码文件中有多个具有相同用户 ID 的记录，从而允许同一用户 ID 有多个登录名。这使得多个用户可以使用不同的密码访问相同的资源（例如文件）。不同的登录名可以与不同的组 ID 集关联。

+   *组 ID*（GID）：这是该用户所属的第一个组的数字 ID。该用户的进一步组成员关系定义在系统组文件中。

+   *注释*：此字段包含有关用户的文本。各种程序，如 *finger(1)*，会显示此文本。

+   *主目录*：这是用户登录后进入的初始目录。此字段成为 `HOME` 环境变量的值。

+   *登录 shell*：这是用户登录后控制权转交给的程序。通常，这是其中一个 shell，例如 *bash*，但它可以是任何程序。如果此字段为空，则登录 shell 默认为 `/bin/sh`，即 Bourne shell。此字段成为 `SHELL` 环境变量的值。

在独立系统上，所有密码信息存储在文件 `/etc/passwd` 中。然而，如果我们使用类似网络信息系统（NIS）或轻量级目录访问协议（LDAP）这样的系统来在网络环境中分发密码，那么部分或全部信息将存储在远程系统上。只要访问密码信息的程序采用本章后面描述的函数（*getpwnam()*、*getpwuid()* 等），NIS 或 LDAP 的使用对于应用程序是透明的。类似的评论适用于后面讨论的影子密码和组文件。

## 影子密码文件：`/etc/shadow`

历史上，UNIX 系统将所有用户信息，包括加密密码，都保存在`/etc/passwd`中。这带来了安全问题。由于各种无特权的系统工具需要读取密码文件中的其他信息，因此该文件必须对所有用户可读。这为密码破解程序打开了大门，这些程序尝试加密可能的密码列表（例如，标准字典单词或人名），以查看它们是否与用户的加密密码匹配。*影子密码文件*`/etc/shadow`的设计目的是防止此类攻击。其思路是，所有不敏感的用户信息保存在公开可读的密码文件中，而加密密码则保存在影子密码文件中，只有特权程序才能读取该文件。

除了提供与密码文件中相应记录匹配的登录名和加密密码外，影子密码文件还包含其他许多与安全相关的字段。有关这些字段的详细信息，请参阅*shadow(5)*手册页。我们主要关心的是加密密码字段，我们将在第 8.5 节进一步讨论此字段，并查看*crypt()*库函数。

SUSv3 没有指定影子密码。并非所有 UNIX 实现都提供此功能，在提供此功能的实现中，文件位置和 API 的细节有所不同。

## 组文件：`/etc/group`

出于各种管理目的，特别是控制对文件和其他系统资源的访问，组织用户到*组*中是非常有用的。

用户所属的组集由用户密码条目中的组 ID 字段和组文件中列出的用户所属组的组合定义。这种将信息分散到两个文件中的做法源于历史。在早期的 UNIX 实现中，用户一次只能属于一个组。用户在登录时的初始组成员身份是由密码文件中的组 ID 字段确定的，之后可以使用*newgrp(1)*命令进行更改，前提是用户需要提供组密码（如果该组受密码保护）。4.2BSD 引入了多个同时组成员身份的概念，后来在 POSIX.1-1990 中进行了标准化。在这种方案下，组文件列出了每个用户的额外组成员身份。（*groups(1)* 命令显示 shell 进程所属的组，或者，如果在命令行参数中提供一个或多个用户名，则显示这些用户的组成员身份。）

*组文件*`/etc/group`包含系统中每个组的一行。每一行由四个由冒号分隔的字段组成，如下例所示：

```
users:x:100:
jambit:x:106:claus,felli,frank,harti,markus,martin,mtk,paul
```

按顺序，这些字段如下：

+   *组名*：这是组的名称。与密码文件中的登录名一样，我们可以将其视为与数字组标识符对应的人类可读（符号）标识符。

+   *加密密码*：此字段包含一个可选的密码，用于该组。随着多个组成员身份的出现，UNIX 系统上现在很少使用组密码。尽管如此，仍然可以给组设置密码（特权用户可以使用 *gpasswd* 命令做到这一点）。如果用户不是该组的成员，*newgrp(1)* 在启动新 shell 之前会要求输入此密码，而该 shell 的组成员包括该组。如果启用了密码影像功能，则此字段会被忽略（在这种情况下，通常只包含字母 *x*，但也可以出现任何字符串，包括空字符串），加密后的密码实际上保存在 *shadow group file* 文件中，即 `/etc/gshadow`，该文件只能由特权用户和程序访问。组密码的加密方式与用户密码类似（参见密码加密与用户认证）。

+   *组 ID*（GID）：这是该组的数字 ID。通常有一个组定义为组 ID 0，名为 *root*（与用户 ID 为 0 的 `/etc/passwd` 记录类似，但与用户 ID 0 不同，该组没有特殊权限）。在 Linux 2.2 及更早版本中，组 ID 被作为 16 位值维护，范围从 0 到 65,535；在 Linux 2.4 及更高版本中，它们使用 32 位存储。

+   *用户列表*：这是一个由逗号分隔的用户名单，列出了此组的成员。（此列表包含用户名，而不是用户 ID，因为，如前所述，用户 ID 在密码文件中不一定是唯一的。）

为了记录用户 *avr* 是 *users*、*staff* 和 *teach* 组的成员，我们会在密码文件中看到以下记录：

```
avr:x:1001:100:Anthony Robins:/home/avr:/bin/bash
```

以下记录会出现在组文件中：

```
users:x:100:
staff:x:101:mtk,avr,martinl
teach:x:104:avr,rlb,alc
```

密码记录的第四个字段，包含组 ID 100，指定了 *users* 组的成员身份。其余的组成员身份通过在组文件中的相关记录中一次列出 *avr* 来表示。

## 检索用户和组信息

在本节中，我们将介绍一些库函数，它们允许我们从密码文件、影像密码文件和组文件中检索单个记录，并扫描每个文件中的所有记录。

#### 从密码文件中检索记录

*getpwnam()* 和 *getpwuid()* 函数从密码文件中检索记录。

```
#include <pwd.h>

struct passwd *`getpwnam`(const char **name*);
struct passwd *`getpwuid`(uid_t *uid*);
```

### 注释

两者在成功时返回一个指针，在出错时返回 `NULL`；有关“未找到”情况的描述，请参见正文

给定登录名 *name*，*getpwnam()* 函数返回一个指向以下类型结构的指针，该结构包含来自密码记录的相应信息：

```
struct passwd {
    char *pw_name;      /* Login name (username) */
    char *pw_passwd;    /* Encrypted password */
    uid_t pw_uid;       /* User ID */
    gid_t pw_gid;       /* Group ID */
    char *pw_gecos;     /* Comment (user information) */
    char *pw_dir;       /* Initial working (home) directory */
    char *pw_shell;     /* Login shell */
};
```

*passwd* 结构体的 *pw_gecos* 和 *pw_passwd* 字段在 SUSv3 中没有定义，但在所有 UNIX 实现中都可用。如果没有启用密码阴影，则 *pw_passwd* 字段包含有效信息。（从编程角度来看，确定是否启用了密码阴影的最简单方法是，在成功调用 *getpwnam()* 后，调用 *getspnam()*（稍后描述），查看是否返回相同用户名的阴影密码记录。）一些其他实现提供了此结构体中的额外非标准字段。

### 注意

*pw_gecos* 字段的名字来源于早期的 UNIX 实现，当时这个字段包含用于与运行通用电气综合操作系统（GECOS）的机器进行通信的信息。尽管这种用法早已过时，但字段名称得以保留，并且该字段用于记录有关用户的信息。

*getpwuid()* 函数返回与 *getpwnam()* 完全相同的信息，但通过传入的 *uid* 数字用户 ID 进行查找。

*getpwnam()* 和 *getpwuid()* 都返回指向静态分配结构体的指针。每次调用这些函数（或者下面描述的 *getpwent()* 函数）时，都会覆盖该结构体。

### 注意

由于它们返回指向静态分配内存的指针，*getpwnam()* 和 *getpwuid()* 不是可重入的。实际上，情况更为复杂，因为返回的 *passwd* 结构体包含指向其他信息的指针（例如，*pw_name* 字段），这些信息也是静态分配的。（我们在可重入和异步信号安全函数中解释了可重入性。）类似的说法适用于 *getgrnam()* 和 *getgrgid()* 函数（稍后描述）。

SUSv3 指定了一组等效的可重入函数——*getpwnam_r()*、*getpwuid_r()*、*getgrnam_r()* 和 *getgrgid_r()*——这些函数的参数既包括 *passwd*（或 *group*）结构体，又包括一个缓冲区，用于保存其他结构体，这些结构体是 *passwd*（*group*）结构体字段指向的内容。可以使用调用 *sysconf(_SC_GETPW_R_SIZE_MAX)*（或者在与组相关的函数中使用 *sysconf(_SC_GETGR_R_SIZE_MAX)*）来获取此附加缓冲区所需的字节数。有关这些函数的详细信息，请参见手册页。

根据 SUSv3，如果无法找到匹配的 *passwd* 记录，则 *getpwnam()* 和 *getpwuid()* 应返回 `NULL` 并保持 *errno* 不变。这意味着我们应该能够通过如下代码区分错误情况和“未找到”情况：

```
struct passwd *pwd;

errno = 0;
pwd = getpwnam(name);
if (pwd == NULL) {
    if (errno == 0)
        /* Not found */;
    else
        /* Error */;
 }
```

然而，一些 UNIX 实现未遵循 SUSv3 在这一点上的规定。如果找不到匹配的 *passwd* 记录，这些函数将返回 `NULL` 并将 *errno* 设置为非零值，如 `ENOENT` 或 `ESRCH`。在版本 2.7 之前，*glibc* 在这种情况下会产生 `ENOENT` 错误，但从版本 2.7 开始，*glibc* 遵循了 SUSv3 的要求。这个实现差异部分源于 POSIX.1-1990 并未要求这些函数在出错时设置 *errno*，而是允许它们在“未找到”情况下设置 *errno*。所有这些的结果是，使用这些函数时，实际上无法在错误和“未找到”之间进行可移植的区分。

#### 从组文件中检索记录

*getgrnam()* 和 *getgrgid()* 函数从组文件中检索记录。

```
#include <grp.h>
struct group *`getgrnam`(const char **name*);
struct group *`getgrgid`(gid_t *gid*);
```

### 注意

成功时返回指针，错误时返回 `NULL`；详见正文描述的“未找到”情况

*getgrnam()* 函数通过组名查找组信息，而 *getgrgid()* 函数通过组 ID 进行查找。两个函数都返回指向以下类型结构的指针：

```
struct group {
    char  *gr_name;     /* Group name */
    char  *gr_passwd;   /* Encrypted password (if not password shadowing) */
    gid_t  gr_gid;      /* Group ID */
    char **gr_mem;      /* NULL-terminated array of pointers to names
                           of members listed in /etc/group */
};
```

### 注意

*group* 结构中的 *gr_passwd* 字段在 SUSv3 中没有指定，但在大多数 UNIX 实现中可用。

与上述对应的密码函数一样，每次调用这些函数时，该结构都会被覆盖。

如果这些函数找不到匹配的 *group* 记录，那么它们表现出与 *getpwnam()* 和 *getpwuid()* 相同的行为变化。

#### 示例程序

本节中我们已经描述的函数的一个常见用途是将符号用户和组名称转换为数字 ID，反之亦然。示例 8-1 展示了这些转换，形式为四个函数：*userNameFromId()*、*userIdFromName()*、*groupNameFromId()* 和 *groupIdFromName()*。为了方便调用者，*userIdFromName()* 和 *groupIdFromName()* 还允许 *name* 参数为（纯粹的）数字字符串；在这种情况下，字符串会被直接转换为数字并返回给调用者。我们将在本书后面的示例程序中使用这些函数。

示例 8-1：将用户和组 ID 转换为用户和组名称的函数

```
`users_groups/ugid_functions.c`
#include <pwd.h>
#include <grp.h>
#include <ctype.h>
#include "ugid_functions.h"     /* Declares functions defined here */

char *          /* Return name corresponding to 'uid', or NULL on error */
userNameFromId(uid_t uid)
{
    struct passwd *pwd;

    pwd = getpwuid(uid);
    return (pwd == NULL) ? NULL : pwd->pw_name;
}

uid_t           /* Return UID corresponding to 'name', or -1 on error */
userIdFromName(const char *name)
{
    struct passwd *pwd;
    uid_t u;
    char *endptr;

    if (name == NULL || *name == '\0')  /* On NULL or empty string */
        return -1;                      /* return an error */

    u = strtol(name, &endptr, 10);      /* As a convenience to caller */
    if (*endptr == '\0')                /* allow a numeric string */
        return u;

    pwd = getpwnam(name);
    if (pwd == NULL)
        return -1;

    return pwd->pw_uid;
}

char *          /* Return name corresponding to 'gid', or NULL on error */
groupNameFromId(gid_t gid)
{
    struct group *grp;

    grp = getgrgid(gid);
    return (grp == NULL) ? NULL : grp->gr_name;
}

gid_t           /* Return GID corresponding to 'name', or -1 on error */
groupIdFromName(const char *name)
{
    struct group *grp;
    gid_t g;
    char *endptr;

    if (name == NULL || *name == '\0')  /* On NULL or empty string */
        return -1;                      /* return an error */

    g = strtol(name, &endptr, 10);      /* As a convenience to caller */
    if (*endptr == '\0')                /* allow a numeric string */
        return g;

    grp = getgrnam(name);
    if (grp == NULL)
        return -1;

    return grp->gr_gid;
}
     `users_groups/ugid_functions.c`
```

#### 扫描密码文件和组文件中的所有记录

*setpwent()*、*getpwent()* 和 *endpwent()* 函数用于执行密码文件记录的顺序扫描。

```
#include <pwd.h>

struct passwd *`getpwent`(void);
```

### 注意

成功时返回指针，结束流或错误时返回 `NULL`

```
void `setpwent`(void);
void `endpwent`(void);
```

*getpwent()* 函数从密码文件中逐条返回记录，当没有更多记录（或发生错误）时，返回`NULL`。在第一次调用时，*getpwent()* 会自动打开密码文件。当我们处理完文件后，调用 *endpwent()* 关闭文件。

我们可以通过以下代码遍历整个密码文件，打印登录名和用户 ID：

```
struct passwd *pwd;

while ((pwd = getpwent()) != NULL)
    printf("%-8s %5ld\n", pwd->pw_name, (long) pwd->pw_uid);

endpwent();
```

必须调用*endpwent()*，以便任何后续的*getpwent()*调用（可能在程序的其他部分或我们调用的某些库函数中）会重新打开密码文件并从头开始。另一方面，如果我们在文件中间，使用*setpwent()*函数可以从头开始重新启动。

*getgrent()*、*setgrent()*和*endgrent()*函数执行类似于组文件的任务。由于这些函数与上述密码文件函数类似，我们省略了这些函数的原型；详细信息请参见手册页。

#### 从影子密码文件中检索记录

以下函数用于从影子密码文件中检索单个记录并扫描文件中的所有记录。

```
#include <shadow.h>

struct spwd *`getspnam`(const char **name*);
```

### 注意

成功时返回指针，未找到或出错时返回`NULL`

```
struct spwd *`getspent`(void);
```

### 注意

成功时返回指针，流结束或出错时返回`NULL`

```
void `setspent`(void);
void `endspent`(void);
```

我们不会详细描述这些函数，因为它们的操作与对应的密码文件函数类似。（这些函数没有在 SUSv3 中指定，并且并非所有 UNIX 实现都包含这些函数。）

*getspnam()*和*getspent()*函数返回指向*spwd*类型结构的指针。该结构具有以下形式：

```
struct spwd {
    char *sp_namp;          /* Login name (username) */
    char *sp_pwdp;          /* Encrypted password */

    /* Remaining fields support "password aging", an optional
       feature that forces users to regularly change their
       passwords, so that even if an attacker manages to obtain
       a password, it will eventually cease to be usable. */

    long sp_lstchg;         /* Time of last password change
                               (days since 1 Jan 1970) */
    long sp_min;            /* Min. number of days between password changes */
    long sp_max;            /* Max. number of days before change required */
    long sp_warn;           /* Number of days beforehand that user is
                               warned of upcoming password expiration */
    long sp_inact;          /* Number of days after expiration that account
                               is considered inactive and locked */
    long sp_expire;         /* Date when account expires
                               (days since 1 Jan 1970) */
    unsigned long sp_flag;  /* Reserved for future use */
};
```

我们在示例 8-2 中展示了如何使用*getspnam()*。

## 密码加密与用户认证

一些应用程序要求用户进行身份验证。身份验证通常是通过用户名（登录名）和密码的形式进行的。应用程序可能会维护自己的用户名和密码数据库以供此用途。然而，有时允许用户输入在`/etc/passwd`和`/etc/shadow`中定义的标准用户名和密码是必要或方便的。（在本节的其余部分，我们假设启用了密码影藏功能，因此加密的密码存储在`/etc/shadow`中。）提供某种远程系统登录功能的网络应用程序，如*ssh*和*ftp*，就是这种程序的典型例子。这些应用程序必须以与标准*login*程序相同的方式验证用户名和密码。

出于安全考虑，UNIX 系统使用*单向加密*算法加密密码，这意味着无法从加密后的密码恢复出原始密码。因此，验证候选密码的唯一方法是使用相同的算法对其进行加密，并查看加密结果是否与`/etc/shadow`中存储的值匹配。加密算法封装在*crypt()*函数中。

```
#define _XOPEN_SOURCE
#include <unistd.h>

char *`crypt`(const char **key*, const char **salt*);
```

### 注意

返回指向静态分配的字符串的指针，成功时包含加密后的密码，出错时返回`NULL`

*crypt()* 算法采用一个最多 8 个字符的 *key*（即密码），并对其应用一种变化版的数据加密标准（DES）算法。*salt* 参数是一个 2 字符的字符串，其值用于扰动（变化）算法，这是一种旨在使破解加密密码更加困难的技术。该函数返回一个指向静态分配的 13 字符字符串的指针，该字符串即为加密后的密码。

### 注意

DES 的详细信息可以在 [`www.itl.nist.gov/fipspubs/fip46-2.htm`](http://www.itl.nist.gov/fipspubs/fip46-2.htm) 中找到。如前所述，可以使用其他算法代替 DES。例如，MD5 会生成一个以美元符号（`$`）开头的 34 字符串，这使得 *crypt()* 能区分 DES 加密的密码和 MD5 加密的密码。

在我们讨论密码加密时，使用了“加密”这个词稍微有些宽泛。准确来说，DES 使用给定的密码字符串作为加密密钥对一个固定的位串进行编码，而 MD5 是一种复杂的哈希函数类型。在这两种情况下，结果都是相同的：输入密码的不可解读且不可逆的转换。

*salt* 参数和加密密码由从 64 字符集 `[a-zA-Z0-9/.]` 中选择的字符组成。因此，2 字符的 *salt* 参数可以使加密算法在 64 * 64 = 4096 种不同的方式中变化。这意味着，破解者需要检查密码与字典的 4096 个加密版本，而不是预先加密整个字典并与字典中的所有单词进行比较。

*crypt()* 返回的加密密码包含原始 *salt* 值的副本，作为其前两个字符。这意味着，在加密候选密码时，我们可以从 `/etc/shadow` 中已存储的加密密码值中获取适当的 *salt* 值。（例如，*passwd(1)* 程序在加密新密码时会生成一个随机的 *salt* 值。）事实上，*crypt()* 函数会忽略 *salt* 字符串中超过前两个字符的任何内容。因此，我们可以将加密密码本身指定为 *salt* 参数。

为了在 Linux 上使用 *crypt()*，我们必须在编译程序时使用 *-lcrypt* 选项，以便将程序与 *crypt* 库链接。

#### 示例程序

示例 8-2 演示了如何使用 *crypt()* 对用户进行身份验证。该程序首先读取用户名，然后检索相应的密码记录以及（如果存在）shadow 密码记录。如果未找到密码记录，或者程序没有权限从 shadow 密码文件中读取（这需要超级用户权限或属于 *shadow* 组），程序将打印错误信息并退出。然后，程序使用 *getpass()* 函数读取用户密码。

```
#define _BSD_SOURCE
#include <unistd.h>
char *`getpass`(const char **prompt*);
```

### 注意

成功时返回指向静态分配的输入密码字符串的指针，出错时返回 `NULL`

*getpass()* 函数首先禁用回显以及终端特殊字符的所有处理（例如 *中断* 字符，通常是 *Control-C*）。(我们在第六十二章中解释了如何更改这些终端设置。) 然后它打印由 *prompt* 指向的字符串，并读取一行输入，将空字符终止的输入字符串返回，去掉尾随的换行符作为函数结果。（该字符串是静态分配的，因此在随后的 *getpass()* 调用中将被覆盖。）在返回之前，*getpass()* 恢复终端设置到原始状态。

在使用 *getpass()* 读取密码后，示例 8-2 中的程序接着使用 *crypt()* 对密码进行加密，并检查加密后的字符串是否与影子密码文件中记录的加密密码匹配。如果密码匹配，则显示用户的 ID，如下例所示：

```
$ `su`                            *Need privilege to read shadow password file*

Password:
# `./check_password`

Username: `mtk`

Password:                       *We type in password, which is not echoed*

Successfully authenticated: UID=1000
```

### 注意

示例 8-2 中的程序使用 *sysconf(_SC_LOGIN_NAME_MAX)* 返回的值来确定存储用户名的字符数组的大小，该值表示主机系统上用户名的最大大小。我们在第 11.2 节中解释了 *sysconf()* 的使用。

示例 8-2. 通过影子密码文件验证用户身份

```
`users_groups/check_password.c`
#define _BSD_SOURCE     /* Get getpass() declaration from <unistd.h> */
#define _XOPEN_SOURCE   /* Get crypt() declaration from <unistd.h> */
#include <unistd.h>
#include <limits.h>
#include <pwd.h>
#include <shadow.h>
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    char *username, *password, *encrypted, *p;
    struct passwd *pwd;
    struct spwd *spwd;
    Boolean authOk;
    size_t len;
    long lnmax;

    lnmax = sysconf(_SC_LOGIN_NAME_MAX);
    if (lnmax == -1)                    /* If limit is indeterminate */
        lnmax = 256;                    /* make a guess */

    username = malloc(lnmax);
    if (username == NULL)
        errExit("malloc");

    printf("Username: ");
    fflush(stdout);
    if (fgets(username, lnmax, stdin) == NULL)
        exit(EXIT_FAILURE);             /* Exit on EOF */

    len = strlen(username);
    if (username[len - 1] == '\n')
        username[len - 1] = '\0';       /* Remove trailing '\n' */

    pwd = getpwnam(username);
    if (pwd == NULL)
        fatal("couldn't get password record");
    spwd = getspnam(username);
    if (spwd == NULL && errno == EACCES)
        fatal("no permission to read shadow password file");

    if (spwd != NULL)           /* If there is a shadow password record */
        pwd->pw_passwd = spwd->sp_pwdp;     /* Use the shadow password */

    password = getpass("Password: ");

    /* Encrypt password and erase cleartext version immediately */

    encrypted = crypt(password, pwd->pw_passwd);
    for (p = password; *p != '\0'; )
        *p++ = '\0';

    if (encrypted == NULL)
        errExit("crypt");

    authOk = strcmp(encrypted, pwd->pw_passwd) == 0;
    if (!authOk) {
        printf("Incorrect password\n");
        exit(EXIT_FAILURE);
    }

    printf("Successfully authenticated: UID=%ld\n", (long) pwd->pw_uid);

    /* Now do authenticated work... */

    exit(EXIT_SUCCESS);
}
     `users_groups/check_password.c`
```

示例 8-2 说明了一个重要的安全点。读取密码的程序应立即加密该密码，并从内存中擦除未加密的版本。这可以最大限度地减少程序崩溃时产生核心转储文件并被读取以发现密码的可能性。

### 注意

还有其他可能的方式使未加密的密码暴露。例如，如果包含密码的虚拟内存页被交换出去，特权程序可能会从交换文件中读取密码。或者，具有足够权限的进程可能会尝试读取 `/dev/mem`（一个虚拟设备，它将计算机的物理内存呈现为一系列连续的字节流），以试图发现密码。

*getpass()* 函数首次出现在 SUSv2 中，该版本将其标记为遗留功能，并指出该名称具有误导性，且该函数提供的功能实际上很容易实现。*getpass()* 的规范在 SUSv3 中被删除。然而，它仍然出现在大多数 UNIX 实现中。

## 总结

每个用户都有一个唯一的登录名和相关联的数字用户 ID。用户可以属于一个或多个组，每个组也有一个唯一的名称和相关联的数字标识符。这些标识符的主要用途是建立各种系统资源（例如文件）的所有权以及访问这些资源的权限。

用户的名字和 ID 在`/etc/passwd`文件中定义，该文件还包含关于用户的其他信息。用户的组成员身份通过`/etc/passwd`和`/etc/group`文件中的字段来定义。另一个文件`/etc/shadow`只能由特权进程读取，用于将敏感的密码信息与`/etc/passwd`中的公开用户信息分开。提供了各种库函数用于从这些文件中检索信息。

*crypt()*函数以与标准*login*程序相同的方式加密密码，这对于需要认证用户的程序非常有用。

## 练习

1.  当我们执行以下代码时，我们发现它显示相同的数字两次，即使这两个用户在密码文件中有不同的 ID。为什么会这样？

    ```
    printf("%ld %ld\n", (long) (getpwnam("avr")->pw_uid),
                        (long) (getpwnam("tsr")->pw_uid));
    ```

1.  使用*getpwnam()*实现*setpwent()*、*getpwent()*和*endpwent()*。
