## 第九章. 进程凭证

每个进程都有一组关联的数字用户标识符（UIDs）和组标识符（GIDs）。有时，这些被称为进程凭证。它们的定义如下：

+   实际用户 ID 和组 ID；

+   有效用户 ID 和组 ID；

+   保存的设置用户 ID 和保存的设置组 ID；

+   文件系统用户 ID 和组 ID（Linux 特有）；以及

+   补充组 ID。

本章中，我们详细探讨这些进程标识符的作用，并描述可以用来检索和更改它们的系统调用和库函数。我们还讨论了特权进程和非特权进程的概念，以及使用设置用户 ID 和设置组 ID 机制，允许创建具有指定用户或组权限的程序。

## 实际用户 ID 和实际组 ID

实际的用户 ID 和组 ID 标识了进程所属的用户和组。作为登录过程的一部分，登录 Shell 从用户密码记录的第三和第四字段中获取其实际的用户 ID 和组 ID，这些记录存储在`/etc/passwd`文件中（密码文件：`/etc/passwd`）。当一个新进程被创建时（例如，当 Shell 执行一个程序时），它会从其父进程继承这些标识符。

## 有效用户 ID 和有效组 ID

在大多数 UNIX 实现中（Linux 有所不同，如在文件系统用户 ID 和文件系统组 ID 中所述），有效用户 ID 和组 ID，与补充组 ID 一起，用于确定当进程尝试执行各种操作（即系统调用）时赋予它的权限。例如，这些标识符决定了当进程访问文件和 System V 进程间通信（IPC）对象等资源时赋予它的权限，这些资源本身也有与之相关联的用户和组 ID，决定了它们属于谁。正如我们在发送信号：*kill()*")中看到的，有效用户 ID 还被内核用来判断一个进程是否可以向另一个进程发送信号。

一个有效用户 ID 为 0（即*root*用户 ID）的进程拥有超级用户的所有权限。这样的进程被称为*特权进程*。某些系统调用只能由特权进程执行。

### 注意

在第三十九章中，我们描述了 Linux 对能力的实现，这是一种将超级用户的权限划分为多个独立单元的机制，这些单元可以单独启用或禁用。

通常，进程的有效用户 ID 和组 ID 与相应的真实 ID 值相同，但有两种方式可以使有效 ID 具有不同的值。一种方式是通过我们在 9.7 节中讨论的系统调用。第二种方式是通过执行设置用户 ID 和设置组 ID 的程序。

## 设置用户 ID 和设置组 ID 程序

设置用户 ID 的程序允许进程通过将进程的有效用户 ID 设置为与可执行文件的用户 ID（所有者）相同的值，来获得通常无法拥有的权限。设置组 ID 的程序为进程的有效组 ID 执行类似的任务。（术语*设置用户 ID 程序*和*设置组 ID 程序*有时简写为*set-UID 程序*和*set-GID 程序*。）

与其他文件一样，可执行程序文件具有一个关联的用户 ID 和组 ID，用于定义文件的所有权。此外，可执行文件有两个特殊的权限位：设置用户 ID 和设置组 ID 位。（实际上，每个文件都有这两个权限位，但我们在这里关注的是它们与可执行文件的使用。）这些权限位是通过*chmod*命令设置的。一个非特权用户可以为他们拥有的文件设置这些位。一个特权用户（`CAP_FOWNER`）可以为任何文件设置这些位。下面是一个例子：

```
$ `su`

Password:
# `ls -l prog`

-rwxr-xr-x    1 root     root       302585 Jun 26 15:05 prog
# `chmod u+s prog`                        *Turn on set-user-ID permission bit*

# `chmod g+s prog`
                        *Turn on set-group-ID permission bit*
```

如本示例所示，程序可以同时设置这两个位，尽管这种情况不常见。当使用*ls -l*列出具有设置用户 ID 或设置组 ID 权限位的程序的权限时，通常用于表示已设置执行权限的*x*会被*s*替代：

```
# `ls -l prog`

-rwsr-sr-x    1 root     root       302585 Jun 26 15:05 prog
```

当运行设置用户 ID 的程序时（即通过*exec()*将程序加载到进程的内存中），内核会将进程的有效用户 ID 设置为与可执行文件的用户 ID 相同。运行设置组 ID 的程序对进程的有效组 ID 有类似的影响。以这种方式更改有效用户或组 ID 赋予进程（换句话说，执行程序的用户）通常无法拥有的权限。例如，如果一个可执行文件由*root*（超级用户）所有，并且启用了设置用户 ID 权限位，则运行该程序时进程将获得超级用户权限。

设置用户 ID 和设置组 ID 的程序也可以设计成将进程的有效 ID 更改为其他值，而不是*root*。例如，为了提供对受保护文件（或其他系统资源）的访问，可能只需创建一个特殊用途的用户（组）ID，该 ID 拥有访问文件所需的权限，并创建一个设置用户 ID（设置组 ID）程序，将进程的有效用户（组）ID 更改为该 ID。这使得程序可以访问文件，而不授予其超级用户的所有权限。

有时，我们会使用术语设置用户 ID-*root*，以区分由*root*拥有的设置用户 ID 程序和由其他用户拥有的程序，后者仅授予进程该用户所拥有的特权。

### 注意

我们现在开始在两种不同的意义上使用术语*特权*。一种是前面定义的意义：具有有效用户 ID 为 0 的进程，拥有授予*root*的所有特权。然而，当我们谈论由非*root*用户拥有的设置用户 ID 程序时，我们有时会提到一个进程获得了授予设置用户 ID 程序的用户 ID 的特权。每次我们使用*特权*这个术语时，具体指哪种意义应该从上下文中可以清楚地辨别。

出于我们在执行程序时需小心中解释的原因，设置用户 ID 和设置组 ID 权限位对 Linux 上的 shell 脚本没有任何影响。

在 Linux 上常用的设置用户 ID 程序的示例包括：*passwd(1)*，用于更改用户密码；*mount(8)* 和 *umount(8)*，用于挂载和卸载文件系统；以及 *su(1)*，允许用户以不同的用户 ID 运行一个 shell。一个设置组 ID 程序的示例是 *wall(1)*，它将消息写入所有属于*tty*组的终端（通常每个终端都由该组拥有）。

在密码加密与用户认证中，我们提到过示例 8-2 中的程序需要从*root*登录运行，以便访问 `/etc/shadow` 文件。我们可以通过将该程序设为设置用户 ID-*root*程序，使其可以由任何用户运行，具体如下：

```
$ `su`

Password:
# `chown root check_password`             *Make this program owned by root*

# `chmod u+s check_password`              *With the set-user-ID bit enabled*
# `ls -l check_password`

-rwsr-xr-x    1 root   users    18150 Oct 28 10:49 check_password
# `exit`

$ `whoami`                                *This is an unprivileged login*

mtk
$ `./check_password`                      *But we can now access the shadow*

Username: `avr`                           *password file using this program*

Password:
Successfully authenticated: UID=1001
```

设置用户 ID/设置组 ID 技术是一个有用且强大的工具，但如果应用程序设计不当，可能会导致安全漏洞。在第三十八章中，我们列出了编写设置用户 ID 和设置组 ID 程序时应该遵循的一系列良好实践。

## 已保存的设置用户 ID 和已保存的设置组 ID

已保存的设置用户 ID 和已保存的设置组 ID 是为设置用户 ID 和设置组 ID 程序设计的。当程序执行时，以下步骤（以及许多其他步骤）会发生：

1.  如果可执行文件上启用了设置用户 ID（设置组 ID）权限位，那么进程的有效用户（组）ID 将与可执行文件的所有者相同。如果没有设置设置用户 ID（设置组 ID）位，则不会更改进程的有效用户（组）ID。

1.  已保存的设置用户 ID 和已保存的设置组 ID 的值将从相应的有效 ID 复制。这种复制发生在不管文件是否设置了设置用户 ID 或设置组 ID 位的情况下。

作为上述步骤效果的示例，假设一个进程，其实际用户 ID、有效用户 ID 和保存的设置用户 ID 均为 1000，并执行一个由*root*（用户 ID 0）拥有的设置用户 ID 程序。执行后，进程的用户 ID 将按如下方式更改：

```
real=1000 effective=0 saved=0
```

各种系统调用允许设置用户 ID 的程序在实际用户 ID 和保存的设置用户 ID 之间切换其有效用户 ID。类似的系统调用允许设置组 ID 的程序修改其有效组 ID。通过这种方式，程序可以暂时丢失并恢复与执行的文件的用户（组）ID 相关的任何特权。（换句话说，程序可以在潜在特权状态和实际操作特权状态之间切换。）正如我们将在以最小特权操作中详细说明的那样，对于设置用户 ID 和设置组 ID 的程序，在程序不实际需要执行与特权（即保存的设置）ID 相关的任何操作时，在不具特权的（即实际）ID 下操作是一种安全的编程实践。

### 注意

保存的设置用户 ID 和保存的设置组 ID 有时被同义地称为*保存的用户 ID*和*保存的组 ID*。

保存的设置 ID 是 System V 发明的，并被 POSIX 采用。在 BSD 的 4.4 之前的版本中没有提供这些 ID。最初的 POSIX.1 标准将对这些 ID 的支持设为可选，但后来的标准（从 1988 年 FIPS 151-1 开始）将支持设置为强制性。

## 文件系统用户 ID 和文件系统组 ID

在 Linux 上，使用的是文件系统的用户和组 ID，而不是有效的用户和组 ID（与附加组 ID 结合使用），用于在执行文件系统操作时（如打开文件、修改文件所有权和修改文件权限）确定权限。（有效 ID 仍然用于其他目的，正如在其他 UNIX 实现中所描述的那样。）

通常，文件系统的用户和组 ID 与相应的有效 ID 具有相同的值（因此通常与相应的实际 ID 相同）。此外，每当有效用户 ID 或组 ID 发生变化时，无论是通过系统调用还是执行设置用户 ID 或设置组 ID 的程序，相应的文件系统 ID 也会更改为相同的值。由于文件系统 ID 以这种方式跟随有效 ID，这意味着 Linux 在检查权限和权限时实际上表现得与任何其他 UNIX 实现相同。只有当我们使用两个 Linux 特定的系统调用，*setfsuid()*和*setfsgid()*，显式地使它们不同，Linux 才与其他 UNIX 实现有所不同。

为什么 Linux 提供文件系统 ID，在哪些情况下我们希望有效 ID 和文件系统 ID 不同？原因主要是历史性的。文件系统 ID 首次出现在 Linux 1.2 中。在那个内核版本中，如果发送者的有效用户 ID 与目标进程的真实用户 ID 或有效用户 ID 匹配，一个进程可以向另一个进程发送信号。这影响了某些程序，比如 Linux NFS（网络文件系统）服务器程序，它需要能够以相应客户端进程的有效 ID 来访问文件。然而，如果 NFS 服务器改变了其有效用户 ID，它就会面临来自非特权用户进程的信号攻击。为防止这种情况，设计了单独的文件系统用户 ID 和组 ID。通过保持其有效 ID 不变，但更改其文件系统 ID，NFS 服务器可以伪装成另一个用户来访问文件，而不易受到用户进程的信号攻击。

从 2.0 内核版本开始，Linux 采用了 SUSv3 规定的发送信号权限规则，这些规则不涉及目标进程的有效用户 ID（参考发送信号：*kill()*")）。因此，文件系统 ID 功能不再严格必要（如今，进程可以通过合理使用本章后续描述的系统调用来更改有效用户 ID 的值，达到所需的效果），但它仍然保留以兼容现有的软件。

由于文件系统 ID 是一种特殊的存在，且它们通常与相应的有效 ID 具有相同的值，在本书的其余部分，我们将通常以进程的有效 ID 来描述各种文件权限检查以及新文件的所有权设置。尽管在 Linux 上，进程的文件系统 ID 确实用于这些目的，但在实践中，它们的存在很少产生实质性的影响。

## 补充组 ID

补充组 ID 是一组附加的组，表示一个进程所属的组。一个新进程会从其父进程继承这些 ID。登录 shell 会从系统组文件中获取其补充组 ID。如上所述，这些 ID 会与有效 ID 和文件系统 ID 一起使用，用于确定访问文件、System V IPC 对象以及其他系统资源的权限。

## 检索和修改进程凭证

Linux 提供了一系列系统调用和库函数，用于检索和更改本章中描述的各种用户和组 ID。只有其中一些 API 在 SUSv3 中有明确规定，其余的有些在其他 UNIX 实现中广泛可用，有些是 Linux 特有的。我们在描述每个接口时会指出可移植性问题。在本章的最后，表 9-1 总结了所有用于更改进程凭证的接口的操作。

作为使用以下页面中描述的系统调用的替代方法，可以通过检查 Linux 特有的`/proc/`*`PID`*`/status`文件中的`Uid`、`Gid`和`Groups`行来查找任何进程的凭证。`Uid`和`Gid`行按顺序列出标识符，包括真实、有效、保存集合和文件系统。

在以下章节中，我们使用传统的特权进程定义，即有效用户 ID 为 0 的进程。然而，Linux 将超级用户权限划分为不同的能力，如第三十九章所述。两项能力与我们讨论所有用于更改进程用户和组 ID 的系统调用相关：

+   `CAP_SETUID`能力允许进程对其用户 ID 进行任意修改。

+   `CAP_SETGID`能力允许进程对其组 ID 进行任意修改。

### 检索和修改真实、有效和保存的 ID 集合

在接下来的段落中，我们将描述检索和修改真实、有效和保存的 ID 集合的系统调用。有几个系统调用执行这些任务，在某些情况下，它们的功能重叠，反映了这些系统调用起源于不同的 UNIX 实现。

#### 检索真实和有效的 ID

*getuid()*和*getgid()*系统调用分别返回调用进程的真实用户 ID 和真实组 ID。*geteuid()*和*getegid()*系统调用执行对应的任务，用于有效 ID。这些系统调用始终成功。

```
#include <unistd.h>

uid_t `getuid`(void);
```

### 注意

返回调用进程的真实用户 ID

```
uid_t `geteuid`(void);
```

### 注意

返回调用进程的有效用户 ID

```
gid_t `getgid`(void);
```

### 注意

返回调用进程的真实组 ID

```
gid_t `getegid`(void);
```

### 注意

返回调用进程的有效组 ID

#### 修改有效 ID

*setuid()*系统调用将调用进程的有效用户 ID—以及可能的真实用户 ID 和保存的用户 ID 集合—修改为*uid*参数给定的值。*setgid()*系统调用执行对应的任务，用于修改组 ID。

```
#include <unistd.h>

int `setuid`(uid_t *uid*);
int `setgid`(gid_t *gid*);
```

### 注意

成功时返回 0，出错时返回-1

使用*setuid()*和*setgid()*修改进程凭证的规则取决于进程是否具有特权（即，其有效用户 ID 为 0）。以下规则适用于*setuid()*：

1.  当一个非特权进程调用*setuid()*时，仅进程的有效用户 ID 会被更改。此外，它只能更改为与真实用户 ID 或保存的设置用户 ID 相同的值。（尝试违反此限制会产生错误`EPERM`。）这意味着，对于非特权用户而言，这个调用仅在执行一个设置用户 ID 的程序时有用，因为对于正常程序的执行，进程的真实用户 ID、有效用户 ID 和保存的设置用户 ID 都具有相同的值。在某些 BSD 衍生的实现中，非特权进程调用*setuid()*或*setgid()*的语义与其他 UNIX 实现不同：这些调用会更改真实、有效和保存的设置 ID（更改为当前真实或有效 ID 的值）。

1.  当一个特权进程执行*setuid()*并传入非零参数时，真实用户 ID、有效用户 ID 和保存的设置用户 ID 都会被设置为*uid*参数中指定的值。这是一个单向操作，一旦特权进程以这种方式更改了其标识符，它将失去所有特权，因此无法随后使用*setuid()*将标识符重置为 0。如果不希望发生这种情况，则应使用*seteuid()*或*setreuid()*（我们稍后将描述），而不是*setuid()*。

使用*setgid()*更改组 ID 的规则与使用*setuid()*更改用户 ID 的规则相似，只是将*setuid()*替换为*setgid()*，并将*user*替换为*group*。在这些更改下，规则 1 完全适用。规则 2 中，由于更改组 ID 不会导致进程丧失特权（特权由有效的*user* ID 决定），特权程序可以使用*setgid()*自由更改组 ID 为任何期望的值。

以下调用是设置用户 ID 为*root*的程序的首选方法，其有效用户 ID 当前为 0，以不可逆方式丢弃所有特权（通过将有效用户 ID 和保存的设置用户 ID 都设置为与真实用户 ID 相同的值）：

```
if (setuid(getuid()) == -1)
    errExit("setuid");
```

由非*root*用户拥有的设置用户 ID 程序可以使用*setuid()*在真实用户 ID 和保存的设置用户 ID 之间切换有效用户 ID，出于第 9.4 节中描述的安全原因。然而，*seteuid()*更适用于此目的，因为它具有相同的效果，无论该设置用户 ID 程序是否由*root*拥有。

一个进程可以使用*seteuid()*更改其有效用户 ID（更改为由*euid*指定的值），并使用*setegid()*更改其有效组 ID（更改为由*egid*指定的值）。

```
#include <unistd.h>

int `seteuid`(uid_t *euid*);
int `setegid`(gid_t *egid*);
```

### 注意

成功时两者均返回 0，出错时返回 -1

以下规则规范了进程通过*seteuid()*和*setegid()*更改其有效 ID 的方式：

1.  非特权进程只能将有效 ID 更改为与相应的实际 ID 或保存的 ID 相同的值。（换句话说，对于非特权进程，*seteuid()*和*setegid()*的作用与*setuid()*和*setgid()*相同，除了前面提到的 BSD 可移植性问题。）

1.  特权进程可以将有效 ID 更改为任何值。如果特权进程使用*seteuid()*将其有效用户 ID 更改为非零值，那么它将不再是特权进程（但可以通过前述规则恢复特权）。

使用*seteuid()*是设置用户 ID 和组 ID 程序临时丧失特权然后再恢复特权的首选方法。以下是一个示例：

```
euid = geteuid();               /* Save initial effective user ID (which
                                   is same as saved set-user-ID) */
if (seteuid(getuid()) == -1)    /* Drop privileges */
    errExit("seteuid");
if (seteuid(euid) == -1)        /* Regain privileges */
    errExit("seteuid");
```

*seteuid()*和*setegid()*最初源自 BSD，现在已在 SUSv3 中指定，并且在大多数 UNIX 实现中都有提供。

### 注意

在较旧版本的 GNU C 库（*glibc* 2.0 及更早版本）中，*seteuid(euid)*的实现方式是*setreuid(-1, euid)*。在现代版本的*glibc*中，*seteuid(euid)*的实现方式是*setresuid(-1, euid, -1)*。（我们稍后会描述*setreuid()*、*setresuid()*及其组 ID 的类似方法。）这两种实现都允许我们将*euid*指定为与当前有效用户 ID 相同的值（即不做更改）。然而，SUSv3 并未对*seteuid()*规定此行为，并且在某些其他 UNIX 实现中是不可行的。通常情况下，这种实现差异并不明显，因为在正常情况下，实际用户 ID 和保存的用户 ID 与有效用户 ID 的值是相同的。（在 Linux 中，唯一能够使有效用户 ID 与实际用户 ID 和保存的用户 ID 不同的方式是通过使用非标准的*setresuid()*系统调用。）

在所有版本的*glibc*（包括现代版本）中，*setegid(egid)*的实现方式是*setregid(-1, egid)*。与*seteuid()*类似，这意味着我们可以将*egid*指定为与当前有效组 ID 相同的值，尽管这种行为在 SUSv3 中并没有明文规定。这也意味着，如果将有效组 ID 设置为与当前实际组 ID 不同的值，*setegid()*将会更改保存的组 ID。（对于使用*setreuid()*实现的旧版*seteuid()*，也有类似的说明。）同样，这种行为在 SUSv3 中并没有明确规定。

#### 修改实际和有效 ID

*setreuid()*系统调用允许调用进程独立地更改其实际和有效用户 ID 的值。*setregid()*系统调用则执行类似的任务，用于实际和有效组 ID。

```
#include <unistd.h>

int `setreuid`(uid_t *ruid*, uid_t *euid*);
int `setregid`(gid_t *rgid*, gid_t *egid*);
```

### 注意

成功时返回 0，错误时返回-1

每个系统调用的第一个参数是新的实际 ID，第二个参数是新的有效 ID。如果我们只想更改其中一个标识符，那么可以为另一个参数指定-1。

*setreuid()*和*setregid()*最初源自 BSD，现在已在 SUSv3 中指定，并且在大多数 UNIX 实现中都有提供。

与本节描述的其他系统调用一样，*setreuid()*和*setregid()*的变化也受到规则的限制。我们从*setreuid()*的角度描述这些规则，理解*setregid()*是类似的，除了已提到的事项：

1.  一个非特权进程只能将实际用户 ID 设置为当前实际用户 ID 的值（即不更改）或有效用户 ID 的值。有效用户 ID 只能设置为当前实际用户 ID 的值、有效用户 ID 的值（即不更改）或保存的设置用户 ID 的值。

    ### 注意

    SUSv3 表示，是否允许一个非特权进程使用*setreuid()*将实际用户 ID 的值更改为当前实际用户 ID、有效用户 ID 或保存的设置用户 ID，尚未指定，而实际用户 ID 的具体更改在不同实现中可能会有所不同。

    SUSv3 对*setregid()*描述了略有不同的行为：一个非特权进程可以将实际组 ID 设置为当前保存的设置组 ID 的值，或将有效组 ID 设置为当前实际组 ID 或保存的设置组 ID 的值。同样，具体可以进行哪些更改的细节在不同实现中有所不同。

1.  一个特权进程可以对 ID 进行任何更改。

1.  对于特权进程和非特权进程，保存的设置用户 ID 如果符合以下任一条件，也将设置为与（新的）有效用户 ID 相同的值：

    1.  *ruid* 不是 -1（即正在设置实际用户 ID，即使其值与已有值相同），或者

    1.  有效用户 ID 被设置为不同于调用前实际用户 ID 的值。

    换句话说，如果一个进程仅使用*setreuid()*将有效用户 ID 更改为与当前实际用户 ID 相同的值，则保存的设置用户 ID 保持不变，稍后的*setreuid()*（或*seteuid()*）调用可以将有效用户 ID 恢复为保存的设置用户 ID 的值。（SUSv3 未明确指定*setreuid()*和*setregid()*对保存的设置 ID 的影响，但 SUSv4 明确了这里描述的行为。）

第三条规则提供了一种方法，允许设置用户 ID 的程序永久放弃其特权，方法如下所示：

```
setreuid(getuid(), getuid());
```

一个希望将其用户和组凭证都更改为任意值的设置用户 ID-*root*进程，应首先调用*setregid()*，然后再调用*setreuid()*。如果顺序相反调用，那么*setregid()*调用将会失败，因为在调用*setreuid()*之后，程序将不再拥有特权。如果我们使用*setresuid()*和*setresgid()*（下面会描述）来实现这个目的，也应遵循类似的规则。

### 注意

直到包括 4.3BSD 在内的 BSD 版本没有保存的用户 ID 和保存的组 ID（这些如今是 SUSv3 强制要求的）。相反，在 BSD 中，*setreuid()* 和 *setregid()* 允许进程通过来回交换实际 ID 和有效 ID 的值来丢弃并恢复特权。这会产生一个不希望出现的副作用，即为了更改有效用户 ID，必须更改实际用户 ID。

#### 检索实际、有效和保存的设置 ID

在大多数 UNIX 实现中，进程不能直接检索（或更新）其保存的用户 ID 和保存的组 ID。然而，Linux 提供了两个（非标准的）系统调用，允许我们做到这一点：*getresuid()* 和 *getresgid()*。

```
#define _GNU_SOURCE
#include <unistd.h>

int `getresuid`(uid_t **ruid*, uid_t **euid*, uid_t **suid*);
int `getresgid`(gid_t **rgid*, gid_t **egid*, gid_t **sgid*);
```

### 注意

两者在成功时返回 0，错误时返回 -1

*getresuid()* 系统调用返回调用进程当前的实际用户 ID、有效用户 ID 和保存的用户 ID 的值，存储在其三个参数所指向的位置中。*getresgid()* 系统调用对相应的组 ID 执行相同操作。

#### 修改实际、有效和保存的设置 ID

*setresuid()* 系统调用允许调用进程独立更改其所有三个用户 ID 的值。每个用户 ID 的新值由系统调用的三个参数指定。*setresgid()* 系统调用对组 ID 执行类似的操作。

```
#define _GNU_SOURCE
#include <unistd.h>

int `setresuid`(uid_t *ruid*, uid_t *euid*, uid_t *suid*);
int `setresgid`(gid_t *rgid*, gid_t *egid*, gid_t *sgid*);
```

### 注意

两者在成功时返回 0，错误时返回 -1

如果我们不想更改所有标识符，那么为参数指定 -1 将使相应的标识符保持不变。例如，以下调用等价于 *seteuid(x)*：

```
setresuid(-1, x, -1);
```

关于 *setresuid()*（*setresgid()* 类似）可以更改的内容，规则如下：

1.  非特权进程可以将其实际用户 ID、有效用户 ID 和保存的用户 ID 设置为当前实际用户 ID、有效用户 ID 或保存的用户 ID 中的任何值。

1.  特权进程可以对其实际用户 ID、有效用户 ID 和保存的用户 ID 进行任意更改。

1.  无论调用是否对其他 ID 做出更改，文件系统用户 ID 始终设置为与（可能是新的）有效用户 ID 相同的值。

对 *setresuid()* 和 *setresgid()* 的调用具有“全有或全无”的效果。要么所有请求的标识符都成功更改，要么都没有更改。（对于本章描述的更改多个标识符的其他系统调用，也适用相同的评论。）

尽管 *setresuid()* 和 *setresgid()* 提供了最直接的 API 来更改进程凭据，但我们不能在应用程序中便捷地使用它们；它们没有在 SUSv3 中规定，并且仅在少数其他 UNIX 实现中可用。

### 检索和修改文件系统 ID

所有之前描述的更改进程有效用户或组 ID 的系统调用也始终会更改相应的文件系统 ID。要独立于有效 ID 更改文件系统 ID，我们必须使用两个 Linux 特有的系统调用：*setfsuid()*和*setfsgid()*。

```
#include <sys/fsuid.h>

int `setfsuid`(uid_t *fsuid*);
```

### 注意

始终返回先前的文件系统用户 ID

```
int `setfsgid`(gid_t *fsgid*);
```

### 注意

始终返回先前的文件系统组 ID

*setfsuid()*系统调用将进程的文件系统用户 ID 更改为由*fsuid*指定的值。*setfsgid()*系统调用将文件系统组 ID 更改为由*fsgid*指定的值。

再次强调，关于可以进行的更改有一些规则。*setfsgid()*的规则类似于*setfsuid()*的规则，具体如下：

1.  无特权进程可以将文件系统用户 ID 设置为实际用户 ID、有效用户 ID、文件系统用户 ID（即不变）或保存的设置用户 ID 的当前值。

1.  特权进程可以将文件系统用户 ID 设置为任何值。

这些调用的实现有些粗糙。首先，没有相应的系统调用来检索当前的文件系统 ID 值。此外，系统调用不进行错误检查；如果一个无特权进程试图将其文件系统 ID 设置为不可接受的值，该尝试会被悄悄忽略。每个系统调用的返回值是相应文件系统 ID 的先前值，无论调用是否成功。因此，我们确实有一种方式可以找到当前文件系统 ID 的值，但只有在我们尝试（无论是成功还是失败）更改它们时才能知道。

在 Linux 上，不再需要使用*setfsuid()*和*setfsgid()*系统调用，并且应该避免在设计为移植到其他 UNIX 实现的应用程序中使用这些调用。

### 检索和修改补充组 ID

*getgroups()*系统调用返回当前调用进程成员的组的集合，这些组保存在指向*grouplist*的数组中。

```
#include <unistd.h>

int `getgroups`(int *gidsetsize*, gid_t *grouplist*[]);
```

### 注意

返回成功时放置在*grouplist*中的组 ID 数量，出错时返回-1

在 Linux 上，和大多数 UNIX 实现一样，*getgroups()*仅返回调用进程的补充组 ID。然而，SUSv3 也允许实现将调用进程的有效组 ID 包含在返回的*grouplist*中。

调用程序必须分配*grouplist*数组并在参数*gidsetsize*中指定其长度。成功完成后，*getgroups()*返回放置在*grouplist*中的组 ID 数量。

如果进程所属的组数超过了 *gidsetsize*，*getgroups()* 将返回错误（`EINVAL`）。为了避免这种情况，我们可以将 *grouplist* 数组的大小设置为比常量 `NGROUPS_MAX`（在 `<limits.h>` 中定义）大 1（便于便携地允许可能包含有效组 ID），该常量定义了进程可作为成员的最大附加组数。因此，我们可以按照以下方式声明 *grouplist*：

```
gid_t grouplist[NGROUPS_MAX + 1];
```

在 2.6.4 之前的 Linux 内核中，`NGROUPS_MAX` 的值为 32。从 2.6.4 内核版本开始，`NGROUPS_MAX` 的值为 65,536。

应用程序还可以通过以下方式在运行时确定 `NGROUPS_MAX` 限制：

+   调用 *sysconf(_SC_NGROUPS_MAX)*。（我们将在第 11.2 节解释 *sysconf ()* 的使用。）

+   从只读的、Linux 特有的 `/proc/sys/kernel/ngroups_max` 文件中读取限制。自 2.6.4 内核版本以来提供了该文件。

或者，应用程序可以调用 *getgroups()*，并将 *gidsetsize* 指定为 0。在这种情况下，*grouplist* 不会被修改，但调用的返回值会给出进程所属的组数。

通过任何这些运行时技术获得的值可以用于动态分配一个 *grouplist* 数组，以供未来的 *getgroups()* 调用使用。

一个特权进程可以使用 *setgroups()* 和 *initgroups()* 来更改其附加组 ID 集合。

```
#define _BSD_SOURCE
#include <grp.h>

int `setgroups`(size_t *gidsetsize*, const gid_t **grouplist*);
int `initgroups`(const char **user*, gid_t *group*);
```

### 注意

两者在成功时返回 0，在出错时返回 -1

*setgroups()* 系统调用将调用进程的附加组 ID 替换为 *grouplist* 数组中给定的组集合。*gidsetsize* 参数指定 *grouplist* 数组中的组 ID 数量。

*initgroups()* 函数通过扫描 `/etc/groups` 并构建一个包含命名 *user* 所属的所有组的列表，来初始化调用进程的附加组 ID。此外，*group* 中指定的组 ID 也会被添加到进程的附加组 ID 集合中。

*initgroups()* 的主要用户是创建登录会话的程序——例如，*login(1)*，它在执行用户的登录 shell 之前设置各种进程属性。此类程序通常通过从用户密码文件中的记录读取组 ID 字段来获取将用于 *group* 参数的值。这有些令人困惑，因为密码文件中的组 ID 并不是真正的附加组，它定义了登录 shell 的初始真实组 ID、有效组 ID 和保存的组 ID 集合。尽管如此，这就是 *initgroups()* 通常的使用方式。

虽然不是 SUSv3 的一部分，但 *setgroups()* 和 *initgroups()* 在所有 UNIX 实现中都可用。

### 修改进程凭据的调用摘要

表 9-1 总结了用于更改进程凭据的各种系统调用和库函数的效果。

图 9-1 提供了与 表 9-1 中给出的相同信息的图形概述。该图展示了从改变用户 ID 的调用的角度看待问题，但更改组 ID 的规则是类似的。

![凭证更改函数对进程用户 ID 的影响](img/09-1_PROCCRED-uid-functions.png.jpg)图 9-1. 凭证更改函数对进程用户 ID 的影响表 9-1. 用于更改进程凭证的接口摘要

| 接口 | 目的和效果： | 可移植性 |
| --- | --- | --- |
| 非特权进程 | 特权进程 |
| --- | --- |
| *setuid(u) setgid(g)* | 将有效 ID 更改为与当前实际或已保存的 set ID 相同的值 | 将实际、有效和已保存的 set ID 更改为任何（单一）值 | 在 SUSv3 中有规定；BSD 衍生系统有不同的语义 |
| *seteuid(e) setegid(e)* | 将有效 ID 更改为与当前实际或已保存的 set ID 相同的值 | 将有效 ID 更改为任何值 | 在 SUSv3 中有规定 |
| *setreuid(r, e) setregid(r, e)* | （独立地）将实际 ID 更改为与当前实际或有效 ID 相同的值，将有效 ID 更改为与当前实际、有效或已保存的 set ID 相同的值 | （独立地）将实际和有效 ID 更改为任何值 | 在 SUSv3 中有规定，但操作在不同实现中有所不同 |
| *setresuid(r, e, s) setresgid(r, e, s)* | （独立地）将实际、有效和已保存的 set ID 更改为与当前实际、有效或已保存的 set ID 相同的值 | （独立地）将实际、有效和已保存的 set ID 更改为任何值 | 不在 SUSv3 中，并且在其他 UNIX 实现中很少存在 |
| *setfsuid(u) setfsgid(u)* | 将文件系统 ID 更改为与当前实际、有效、文件系统或已保存的 ID 相同的值 | 将文件系统 ID 更改为任何值 | 特定于 Linux |
| *setgroups(n, l)* | 不能从非特权进程调用 | 将补充组 ID 设置为任何值 | 在 SUSv3 中没有，但在所有 UNIX 实现中都可以使用 |

请注意以下补充信息，参见 表 9-1：

+   *glibc* 对 *seteuid()*（如 *setresuid(-1, e, -1)*) 和 *setegid()*（如 *setregid(-1, e)*) 的实现也允许将有效 ID 设置为当前已拥有的相同值，但在 SUSv3 中没有指定这一点。*setegid()* 的实现还会在有效组 ID 被设置为不同于当前实际组 ID 的值时，修改已保存的组 ID。 （SUSv3 没有指定 *setegid()* 会对已保存的组 ID 进行更改。）

+   对于特权进程和非特权进程对 *setreuid()* 和 *setregid()* 的调用，如果 *r* 不是 -1，或者 *e* 在调用之前被指定为与真实 ID 不同的值，那么保存的设置用户 ID 或保存的设置组 ID 也会设置为与（新的）有效 ID 相同的值。（SUSv3 没有指定 *setreuid()* 和 *setregid()* 会改变保存的设置 ID。）

+   每当有效的用户（组）ID 被更改时，Linux 特有的文件系统用户（组）ID 也会被更改为相同的值。

+   调用 *setresuid()* 总是会将文件系统用户 ID 修改为与有效用户 ID 相同的值，无论有效用户 ID 是否被调用修改。调用 *setresgid()* 对文件系统组 ID 具有类似的效果。

### 示例：显示进程凭证

示例 9-1 中的程序使用前面页面描述的系统调用和库函数来检索进程的所有用户和组 ID，并将其显示出来。

示例 9-1. 显示所有进程用户和组 ID

```
`proccred/idshow.c`
#define _GNU_SOURCE
#include <unistd.h>
#include <sys/fsuid.h>
#include <limits.h>
#include "ugid_functions.h"   /* userNameFromId() & groupNameFromId() */
#include "tlpi_hdr.h"

#define SG_SIZE (NGROUPS_MAX + 1)

int
main(int argc, char *argv[])
{
    uid_t ruid, euid, suid, fsuid;
    gid_t rgid, egid, sgid, fsgid;
    gid_t suppGroups[SG_SIZE];
    int numGroups, j;
    char *p;

    if (getresuid(&ruid, &euid, &suid) == -1)
        errExit("getresuid");
    if (getresgid(&rgid, &egid, &sgid) == -1)
        errExit("getresgid");

    /* Attempts to change the file-system IDs are always ignored
       for unprivileged processes, but even so, the following
       calls return the current file-system IDs */

    fsuid = setfsuid(0);
    fsgid = setfsgid(0);

    printf("UID: ");
    p = userNameFromId(ruid);
    printf("real=%s (%ld); ", (p == NULL) ? "???" : p, (long) ruid);
    p = userNameFromId(euid);
    printf("eff=%s (%ld); ", (p == NULL) ? "???" : p, (long) euid);
    p = userNameFromId(suid);
    printf("saved=%s (%ld); ", (p == NULL) ? "???" : p, (long) suid);
    p = userNameFromId(fsuid);
    printf("fs=%s (%ld); ", (p == NULL) ? "???" : p, (long) fsuid);
    printf("\n");

    printf("GID: ");
    p = groupNameFromId(rgid);
    printf("real=%s (%ld); ", (p == NULL) ? "???" : p, (long) rgid);
    p = groupNameFromId(egid);
    printf("eff=%s (%ld); ", (p == NULL) ? "???" : p, (long) egid);
    p = groupNameFromId(sgid);
    printf("saved=%s (%ld); ", (p == NULL) ? "???" : p, (long) sgid);
    p = groupNameFromId(fsgid);
    printf("fs=%s (%ld); ", (p == NULL) ? "???" : p, (long) fsgid);
    printf("\n");

    numGroups = getgroups(SG_SIZE, suppGroups);
    if (numGroups == -1)
        errExit("getgroups");

    printf("Supplementary groups (%d): ", numGroups);
    for (j = 0; j < numGroups; j++) {
        p = groupNameFromId(suppGroups[j]);
        printf("%s (%ld) ", (p == NULL) ? "???" : p, (long) suppGroups[j]);
    }
    printf("\n");

    exit(EXIT_SUCCESS);
}
     `proccred/idshow.c`
```

## 总结

每个进程都有一组与之相关的用户和组 ID（凭证）。真实 ID 定义了进程的所有权。在大多数 UNIX 实现中，有效 ID 用于确定进程访问资源（如文件）时的权限。然而，在 Linux 中，文件系统 ID 用于确定访问文件的权限，而有效 ID 则用于其他权限检查。（因为文件系统 ID 通常与相应的有效 ID 值相同，所以 Linux 在检查文件权限时的行为与其他 UNIX 实现相同。）进程的附加组 ID 是一组进程被视为其成员的额外组，用于权限检查。各种系统调用和库函数允许进程检索和更改其用户和组 ID。

当运行一个设置用户 ID 的程序时，进程的有效用户 ID 会被设置为文件拥有者的 ID。这个机制允许用户在运行特定程序时，假设另一个用户的身份，从而获得其特权。相应地，设置组 ID 的程序会更改运行程序的进程的有效组 ID。保存的设置用户 ID 和保存的设置组 ID 允许设置用户 ID 和设置组 ID 程序临时放弃特权，然后稍后重新获得特权。

用户 ID 0 是特殊的。通常，名为 *root* 的单一用户账户拥有此用户 ID。有效用户 ID 为 0 的进程是特权进程——即它们免除通常在进程进行各种系统调用时执行的许多权限检查（例如那些用于任意更改进程的各种用户和组 ID 的系统调用）。

## 练习

1.  假设在以下每种情况下，进程的初始用户 ID 集为*real=1000 effective=0 saved=0 file-system=0*。在执行以下调用后，用户 ID 的状态会怎样？

    1.  *setuid(2000);*

    1.  *setreuid(-1, 2000);*

    1.  *seteuid(2000);*

    1.  *setfsuid(2000);*

    1.  *setresuid(-1, 2000, 3000);*

1.  具有以下用户 ID 的进程是否有特权？解释你的答案。

    ```
    real=0 effective=1000 saved=1000 file-system=1000
    ```

1.  使用*setgroups()*和库函数实现*initgroups()*，用于从密码和组文件中获取信息（获取用户和组信息）。记住，进程必须具有特权才能调用*setgroups()*。

1.  如果一个进程的所有用户 ID 值为*X*，并执行一个用户 ID 为*Y*且非零的设置用户 ID 程序，则进程的凭证将如下所示：

    ```
    real=X effective=Y saved=Y
    ```

    （我们忽略文件系统用户 ID，因为它跟踪有效用户 ID。）展示分别用于执行以下操作的*setuid()*, *seteuid()*, *setreuid()*, 和*setresuid()*调用：

    1.  暂停并恢复设置用户 ID 身份（即，将有效用户 ID 切换为真实用户 ID 的值，然后再切换回保存的设置用户 ID）。

    1.  永久丢弃设置用户 ID 身份（即，确保有效用户 ID 和保存的设置用户 ID 被设置为真实用户 ID 的值）。

    （本练习还需要使用*getuid()*和*geteuid()*来获取进程的真实和有效用户 ID。）请注意，对于上述列出的一些系统调用，其中一些操作无法执行。

1.  对于执行一个设置用户 ID-*root*程序的进程，重复前一个练习，该进程的初始进程凭证如下：

    ```
    real=X effective=0 saved=0
    ```
