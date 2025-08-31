## 第三十九章. 能力

本章介绍了 Linux 权限能力方案，它将传统的全有或全无的 UNIX 权限方案分解为可以独立启用或禁用的各个能力。使用能力可以让程序执行某些特权操作，同时防止其执行其他操作。

## 能力的基本原理

传统的 UNIX 权限方案将进程分为两类：有效用户 ID 为 0（超级用户）的进程，绕过所有权限检查；以及所有其他进程，这些进程根据其用户和组 ID 进行权限检查。

这一方案的粗粒度是一个问题。如果我们希望允许一个进程执行一些仅超级用户才允许的操作——例如，修改系统时间——那么我们必须使用有效用户 ID 为 0 来运行该进程。（如果没有特权的用户需要执行这些操作，通常会使用一个设置了用户 ID 为 *root* 的程序来实现。）然而，这也会授予该进程执行许多其他操作的权限——例如，绕过访问文件时的所有权限检查——从而为一系列安全漏洞打开了大门，如果程序以意外方式行为（这可能是不可预见的情况所致，或是恶意用户故意操作的结果）。解决这个问题的传统方法在第三十八章中有所概述：我们放弃有效特权（即从有效用户 ID 0 变更，同时保持保存的用户 ID 为 0），并且只有在需要时才临时重新获取这些特权。

Linux 权限能力方案改进了对这一问题的处理方式。与其在内核中执行安全检查时使用单一的特权（即有效用户 ID 为 0），不如将超级用户特权分为不同的单元，称为*能力*。每个特权操作都与特定的能力相关联，只有当进程具有相应的能力时，才能执行该操作（无论其有效用户 ID 为何）。换句话说，在本书中我们讨论的所有 Linux 上的特权进程，实际上指的是那些具有执行特定操作所需能力的进程。

大多数情况下，Linux 权限能力方案对我们是不可见的。原因在于，当一个不知道能力的应用程序假设其有效用户 ID 为 0 时，内核会授予该进程完整的能力范围。

Linux 能力的实现基于 POSIX 1003.1e 草案标准（[`wt.tuxomania.net/publications/posix.1e/`](http://wt.tuxomania.net/publications/posix.1e/)）。该标准化工作在 1990 年代末期未能完成，但各种能力实现仍然基于该草案标准。（表格 39-1 中列出的一些能力是在 POSIX.1e 草案中定义的，但许多是 Linux 的扩展。）

### 注

一些其他 UNIX 实现也提供了能力方案，例如 Sun 的 Solaris 10 及其早期版本的受信任 Solaris、SGI 的受信任 Irix，以及作为 TrustedBSD 项目一部分的 FreeBSD（[Watson, 2000]）。其他操作系统中也存在类似的方案，例如 Digital 的 VMS 系统中的特权机制。

## Linux 能力

表格 39-1 列出了 Linux 的能力，并提供了简略（且不完整）的操作指南，说明这些能力适用于哪些操作。

## 进程和文件能力

每个进程都有三个相关的能力集——称为*许可*、*有效*和*可继承*——这些能力集可以包含表格 39-1 中列出的零个或多个能力。每个文件也可以有三个相关的能力集，名称相同。（出于某些原因，文件的有效能力集实际上只是一个单一的位，能够开启或关闭。）我们将在接下来的章节中详细介绍这些能力集。

### 进程能力

对于每个进程，内核维护三个能力集（实现为位掩码），其中零个或多个表格 39-1 中指定的能力被启用。这三个能力集如下：

+   *许可*: 这些是进程*可能*使用的能力。许可集是可以添加到有效集和可继承集中的能力的限制超集。如果进程从其许可集中删除某个能力，则永远无法重新获取该能力（除非它执行一个程序，该程序再次赋予该能力）。

+   *有效*: 这些是内核用来执行进程权限检查的能力。只要进程在其许可集中保持某个能力，它可以通过将其从有效集移除，临时禁用该能力，然后稍后再将其恢复到有效集中。

+   *可继承*: 这些是可以在程序被此进程执行时传递到许可集中的能力。

我们可以通过 Linux 特定的 `/proc/`*PID*`/status` 文件中的 `CapInh`、`CapPrm` 和 `CapEff` 这三个字段，查看任何进程的三个能力集的十六进制表示。

### 注意

*getpcap* 程序（是 程序化更改进程能力 中描述的 *libcap* 包的一部分）可以用来以更易读的格式显示进程的能力。

通过 *fork()* 创建的子进程会继承父进程的能力集副本。我们在第 39.5 节中描述了 *exec()* 过程中能力集的处理。

### 注意

实际上，能力是每个线程的属性，可以独立地调整进程中每个线程的能力。多线程进程中某个特定线程的能力显示在 `/proc/`*PID*`/task/`*TID*`/status` 文件中。`/proc/`*PID*`/status` 文件显示的是主线程的能力。

在内核版本 2.6.25 之前，Linux 使用 32 位表示能力集。内核 2.6.25 中新增的能力集需要使用 64 位集来表示。

### 文件能力

如果文件具有相关的能力集，那么这些能力集将用于确定当进程执行该文件时授予其的能力。文件有三种能力集：

+   *允许的*：这是一个可以在 *exec()* 过程中添加到进程允许集中的能力集，无论进程当前的能力集是什么。

+   *有效的*：这只是一个单一的位。如果启用，那么在 *exec()* 过程中，进程的新允许集中的启用能力也将在进程的新有效集中启用。如果文件的有效位被禁用，那么在 *exec()* 之后，进程的新有效集将最初为空。

+   *可继承的*：该集合与进程的可继承集合进行掩码操作，以确定在 *exec()* 后应启用进程的允许集中的哪些能力。

*exec()*期间进程能力的转化") 提供了在 *exec()* 过程中如何使用文件能力的详细信息。

### 注意

允许的和可继承的文件能力以前被称为 *强制* 和 *允许*。这些术语现在已经过时，但仍然具有参考意义。允许的文件能力是在 *exec()* 过程中被 *强制* 添加到进程的允许集中的能力，无论进程当前的能力集如何。可继承的文件能力是在 *exec()* 过程中由文件 *允许* 添加到进程的允许集中的能力，如果这些能力也在进程的可继承能力集中启用的话。

文件关联的能力存储在名为 *security.capability* 的 *security* 扩展属性中（概述）。更新此扩展属性需要 `CAP_SETFCAP` 能力。

表 39-1. 每个 Linux 能力允许的操作

| 能力 | 允许进程执行 |
| --- | --- |
| `CAP_AUDIT_CONTROL` | （自 Linux 2.6.11）启用和禁用内核审计日志；更改审计的过滤规则；检索审计状态和过滤规则 |
| `CAP_AUDIT_WRITE` | （自 Linux 2.6.11）将记录写入内核审计日志 |
| `CAP_CHOWN` | 更改文件的用户 ID（所有者）或将文件的组 ID 更改为进程不属于的组（*chown()*) |
| `CAP_DAC_OVERRIDE` | 绕过文件读取、写入和执行权限检查（DAC 是“自主访问控制”的缩写）；读取 `/proc/`*PID* 中的 `cwd`、`exe` 和 `root` 符号链接的内容 |
| `CAP_DAC_READ_SEARCH` | 绕过文件读取权限检查和目录读取与执行（搜索）权限检查 |
| `CAP_FOWNER` | 通常忽略需要进程的文件系统用户 ID 与文件的用户 ID 匹配的操作的权限检查（*chmod()*, *utime()*)；在任意文件上设置 i-node 标志；在任意文件上设置和修改 ACL；忽略删除文件时目录粘性位的影响（*unlink()*, *rmdir()*, *rename()*)；为任意文件指定 `O_NOATIME` 标志，使用 *open()* 和 *fcntl(F_SETFL)* |
| `CAP_FSETID` | 修改文件而不让内核关闭设置用户 ID 和设置组 ID 位（*write()*, *truncate()*)；为一个文件启用设置组 ID 位，即使文件的组 ID 与进程的文件系统组 ID 或附加组 ID 不匹配（*chmod()*) |
| `CAP_IPC_LOCK` | 覆盖内存锁定限制（*mlock()*, *mlockall()*, *shmctl(SHM_LOCK)*, *shmctl(SHM_UNLOCK)*）；使用 *shmget()* `SHM_HUGETLB` 标志和 *mmap()* `MAP_HUGETLB` 标志 |
| `CAP_IPC_OWNER` | 绕过对 System V IPC 对象操作的权限检查 |
| `CAP_KILL` | 绕过发送信号（*kill()*, *sigqueue()*）的权限检查 |
| `CAP_LEASE` | （自 Linux 2.4）在任意文件上建立租约（*fcntl(F_SETLEASE)*) |
| `CAP_LINUX_IMMUTABLE` | 设置附加和不可变的 i-node 标志 |
| `CAP_MAC_ADMIN` | （自 Linux 2.6.25）配置或修改强制访问控制（MAC）的状态（由某些 Linux 安全模块实现） |
| `CAP_MAC_OVERRIDE` | （自 Linux 2.6.25）覆盖 MAC（由某些 Linux 安全模块实现） |
| `CAP_MKNOD` | （自 Linux 2.4）使用 *mknod()* 创建设备 |
| `CAP_NET_ADMIN` | 执行各种与网络相关的操作（例如，设置特权套接字选项、启用多播、配置网络接口和修改路由表） |
| `CAP_NET_BIND_SERVICE` | 绑定到特权套接字端口 |
| `CAP_NET_BROADCAST` | （未使用）执行套接字广播并监听多播 |
| `CAP_NET_RAW` | 使用原始套接字和数据包套接字 |
| `CAP_SETGID` | 对进程组 ID 进行任意修改（*setgid()*, *setegid()*, *setregid()*, *setresgid()*, *setfsgid()*, *setgroups()*, *initgroups()*）；通过 UNIX 域套接字 (`SCM_CREDENTIALS`) 传递凭据时伪造组 ID |
| `CAP_SETFCAP` | （自 Linux 2.6.24 起）设置文件能力 |
| `CAP_SETPCAP` | 如果文件能力不受支持，授予和移除进程允许集中的能力，或者从任何其他进程（包括自身）中添加或删除能力；如果文件能力受支持，将进程的能力边界集中的任何能力添加到其可继承集，删除能力边界集中的能力，并修改 *securebits* 标志 |
| `CAP_SETUID` | 对进程用户 ID 进行任意修改（*setuid()*, *seteuid()*, *setreuid()*, *setresuid()*, *setfsuid()*）；通过 UNIX 域套接字 (`SCM_CREDENTIALS`) 传递凭据时伪造用户 ID |
| `CAP_SYS_ADMIN` | 在打开文件的系统调用中超过 `/proc/sys/fs/file-max` 限制（例如，*open()*, *shm_open()*, *pipe()*, *socket()*, *accept()*, *exec()*, *acct()*, *epoll_create()*）；执行各种系统管理操作，包括 *quotactl()*（控制磁盘配额）、*mount()* 和 *umount()*, *swapon()* 和 *swapoff()*, *pivot_root()*, *sethostname()* 和 *setdomainname()*；执行各种 *syslog(2)* 操作；覆盖 `RLIMIT_NPROC` 资源限制（*fork()*）；调用 *lookup_dcookie()*；设置 *trusted* 和 *security* 扩展属性；对任意 System V IPC 对象执行 `IPC_SET` 和 `IPC_RMID` 操作；通过 UNIX 域套接字 (`SCM_CREDENTIALS`) 传递凭据时伪造进程 ID；使用 *ioprio_set()* 分配 `IOPRIO_CLASS_RT` 调度类；使用 *TIOCCONS ioctl()*；与 *clone()* 和 *unshare()* 一起使用 `CLONE_NEWNS` 标志；执行 `KEYCTL_CHOWN` 和 `KEYCTL_SETPERM` *keyctl()* 操作；管理 *random(4)* 设备；各种设备特定操作 |
| `CAP_SYS_BOOT` | 使用 *reboot()* 重启系统；调用 *kexec_load()* |
| `CAP_SYS_CHROOT` | 使用 *chroot()* 设置进程根目录 |
| `CAP_SYS_MODULE` | 加载和卸载内核模块（*init_module()*, *delete_module()*, *create_module()*) |
| `CAP_SYS_NICE` | 提高优先级值（*nice()*, *setpriority()*）；改变任意进程的优先级值（*setpriority()*）；为调用进程设置 `SCHED_RR` 和 `SCHED_FIFO` 实时调度策略；重置 `SCHED_RESET_ON_FORK` 标志；为任意进程设置调度策略和优先级（*sched_setscheduler()*, *sched_setparam()*）；为任意进程设置 I/O 调度类和优先级（*ioprio_set()*）；为任意进程设置 CPU 亲和性（*sched_setaffinity()*）；使用 *migrate_pages()* 迁移任意进程，并允许进程迁移到任意节点；对任意进程应用 *move_pages()*；使用 `MPOL_MF_MOVE_ALL` 标志与 *mbind()* 和 *move_pages()* |
| `CAP_SYS_PACCT` | 使用 *acct()* 启用或禁用进程会计 |
| `CAP_SYS_PTRACE` | 使用 *ptrace()* 跟踪任意进程；访问 `/proc/`*PID*`/environ` 以获取任意进程的环境变量；对任意进程应用 *get_robust_list()* |
| `CAP_SYS_RAWIO` | 使用 *iopl()* 和 *ioperm()* 对 I/O 端口进行操作；访问 `/proc/kcore`；打开 `/dev/mem` 和 `/dev/kmem` |
| `CAP_SYS_RESOURCE` | 使用文件系统上的保留空间；进行 *ioctl()* 调用以控制 *ext3* 日志；覆盖磁盘配额限制；增加硬资源限制 (*setrlimit()*); 覆盖 `RLIMIT_NPROC` 资源限制 (*fork()*); 提高 System V 消息队列中 *msg_qbytes* 的限制；绕过 `/proc/sys/kernel/msgmnb` 中定义的各种 POSIX 消息队列限制；绕过 `/proc/sys/fs/mqueue` 下文件定义的消息队列限制 |
| `CAP_SYS_TIME` | 修改系统时钟 (*settimeofday()*, *stime()*, *adjtime()*, *adjtimex()*); 设置硬件时钟 |
| `CAP_SYS_TTY_CONFIG` | 使用 *vhangup()* 执行终端或伪终端的虚拟挂断 |

### 进程允许和有效能力集的目的

*进程允许*能力集定义了进程*可以*使用的能力。*进程有效*能力集定义了当前对进程生效的能力——即内核在检查进程是否有必要权限以执行某个操作时所使用的能力集。

允许能力集对有效能力集施加上限。只有当某项能力在允许集中的时候，进程才可以在其有效集里*提升*该能力。（术语 *add* 到和 *set* 有时与 *raise* 同义。相反操作是 *drop*，或者同义的 *remove* 或 *clear*。）

### 注意

有效能力集与允许能力集之间的关系类似于有效用户 ID 与保存的 set-user-ID（针对 set-user-ID-*root* 程序）的关系。将某项能力从有效集移除类似于暂时移除有效用户 ID 为 0，同时保持 0 在保存的 set-user-ID 中。将能力从有效集和允许集都移除类似于通过将有效用户 ID 和保存的 set-user-ID 都设置为非零值，从而永久地移除超级用户权限。

### 文件允许和有效能力集的目的

*文件允许*能力集提供了一种机制，通过它可执行文件可以向进程授予能力。它指定了一组能力，这些能力将在 *exec()* 调用时分配到进程的允许能力集。

*文件有效*能力集是一个单一标志（位），可以启用或禁用。为了理解为何这个集合只有一个位，我们需要考虑当程序被 exec 时发生的两种情况：

+   程序可能是*能力盲*的，意味着它不知道能力（即，它被设计为传统的设置用户 ID-*root*程序）。这样的程序不会知道它需要在其有效能力集中提升能力，以便能够执行特权操作。对于这样的程序，*exec()*应该具有这样的效果：进程的所有新许可能力自动也会分配到其有效能力集中。通过启用文件有效位来实现这一结果。

+   程序可能是*能力感知*的，意味着它在设计时考虑了能力框架，并会通过适当的系统调用（稍后讨论）在其有效能力集中提升和丢弃能力。对于这样的程序，最小特权原则意味着，在*exec()*调用后，进程的有效能力集中的所有能力应该最初被禁用。通过禁用文件有效能力位来实现这一结果。

### 进程和文件可继承集的目的

乍一看，使用进程和文件的许可集和有效集似乎是能力系统的足够框架。然而，在某些情况下，它们并不充分。例如，如果一个进程在执行*exec()*时希望保留其当前的一些能力怎么办？看起来，能力实现可以通过简单地保留进程的许可能力来提供这个功能。然而，这种方法无法处理以下情况：

+   执行*exec()*可能需要某些特权（例如，`CAP_DAC_OVERRIDE`），而这些特权我们不希望在*exec()*后保留。

+   假设我们显式地丢弃了一些不想在*exec()*后保留的许可能力，但*exec()*调用失败了。在这种情况下，程序可能需要一些它已经（不可恢复地）丢弃的许可能力。

由于这些原因，进程的许可能力不会在*exec()*调用后保留。相反，另一个能力集被引入：*可继承集*。可继承集提供了一种机制，使得进程可以在*exec()*调用后保留其部分能力。

*进程可继承*能力集指定了一组能力，这些能力可能在*exec()*调用过程中被分配到进程的许可能力集中。对应的*文件可继承*能力集会与进程继承的能力集进行掩码（按位与运算），以确定实际添加到进程许可能力集中的能力。

### 注意

不仅仅保留进程许可功能集跨越*exec()*调用，还有一个进一步的哲学原因。功能系统的理念是，所有赋予进程的特权都由进程执行的文件授予或控制。尽管进程可继承集指定了在*exec()*中传递的功能，但这些功能会被文件的可继承集屏蔽。

### 从 Shell 分配和查看文件功能

*setcap(8)*和*getcap(8)*命令包含在*libcap*包中，该包在程序化地更改进程功能中有所描述，能够操作文件功能集。我们通过一个简短的示例演示这些命令的使用，使用的是标准的*date(1)*程序。（根据文件许可和有效功能集的目的的定义，该程序是一个功能缺失的应用示例。）当以特权运行时，*date(1)*可以用来更改系统时间。*date*程序没有设置用户 ID 为*root*，所以通常只有成为超级用户才能以特权运行它。

我们首先显示当前的系统时间，然后尝试作为普通用户更改时间：

```
$ `date`
Tue Dec 28 15:54:08 CET 2010
$ `date -s '2018-02-01 21:39'`
date: cannot set date: Operation not permitted
Thu Feb  1 21:39:00 CET 2018
```

如上所示，我们看到*date*命令未能更改系统时间，但仍然以标准格式显示了其参数。

接下来，我们成为超级用户，这样就能成功更改系统时间：

```
$ `sudo date -s '2018-02-01 21:39'`
root's password:
Thu Feb  1 21:39:00 CET 2018
$ `date`
Thu Feb  1 21:39:02 CET 2018
```

我们现在复制*date*程序，并为其分配所需的功能：

```
$ `whereis -b date`                           *Find location of* *date* *binary*
date: /bin/date
$ `cp /bin/date .`
$ `sudo setcap "cap_sys_time=pe" date`
root's password:
$ `getcap date`
date = cap_sys_time+ep
```

上面显示的*setcap*命令将*CAP_SYS_TIME*功能分配给可执行文件的许可（*p*）和有效（*e*）功能集。然后我们使用*getcap*命令验证分配给文件的功能。（*setcap*和*getcap*用于表示功能集的语法可在*libcap*包提供的*cap_from_text(3)*手册页中找到。）

我们的*date*程序副本的文件功能允许普通用户使用该程序设置系统时间：

```
$ `./date -s '2010-12-28 15:55'`
Tue Dec 28 15:55:00 CET 2010
$ `date`
Tue Dec 28 15:55:02 CET 2010
```

## 现代功能实现

完整的功能实现需要以下内容：

+   对于每个特权操作，内核应该检查进程是否具有相关功能，而不是检查有效（或文件系统）用户 ID 是否为 0。

+   内核必须提供系统调用，允许进程检索和修改其功能。

+   内核必须支持将能力附加到可执行文件的概念，这样进程在执行该文件时会获得相关的能力。这类似于设置用户 ID 位，但允许独立地指定可执行文件上的所有能力。此外，系统必须提供一组编程接口和命令，用于设置和查看附加到可执行文件的能力。

直到内核版本 2.6.23，Linux 仅满足这两个要求中的前两个。从内核 2.6.24 开始，Linux 支持将能力附加到文件上。在内核 2.6.25 和 2.6.26 中，添加了其他功能以完善能力的实现。

在我们对能力的讨论中，我们将专注于现代实现。在旧内核和没有文件能力的系统中，我们将讨论文件能力引入之前实现的不同之处。此外，文件能力在现代内核中是一个可选的内核组件，但在本部分讨论中，我们假设该组件已启用。稍后，我们将描述如果未启用文件能力时的不同之处。（在几个方面，其行为类似于 Linux 2.6.24 之前的内核版本，在这些版本中并未实现文件能力。）

在接下来的章节中，我们将更详细地介绍 Linux 能力的实现。

## 在*exec()*过程中，进程能力的转变

在*exec()*过程中，内核会根据进程当前的能力以及正在执行文件的能力集为进程设置新的能力。内核通过以下规则计算进程的新能力：

```
P'(permitted) = (P(inheritable) & F(inheritable)) | (F(permitted) & cap_bset)

P'(effective) = F(effective) ? P'(permitted) : 0

P'(inheritable) = P(inheritable)
```

在上述规则中，*P*表示*exec()*之前能力集的值，*P’*表示*exec()*之后能力集的值，而*F*表示文件能力集的值。标识符*cap_bset*表示能力边界集的值。请注意，*exec()*不会改变进程的可继承能力集。

### 能力边界集

能力边界集是一种安全机制，用于限制进程在*exec()*过程中能够获得的能力。该集的使用方式如下：

+   在*exec()*过程中，能力边界集与文件允许的能力集进行与操作，以确定授予新程序的允许能力。换句话说，如果某个能力不在边界集中，执行文件的允许能力集不能授予该能力给进程。

+   能力边界集是一个限制性的超集，用于限制可以添加到进程的可继承集中的能力。这意味着，除非能力位于边界集内，否则进程不能将其允许的能力添加到其可继承集，并且通过上面描述的第一个能力转换规则，无法在执行具有可继承集能力的文件时，将该能力保留在其允许集内。

能力边界集是一个进程级别的属性，通过*fork()*创建的子进程会继承该属性，并且在*exec()*中得以保留。在支持文件能力的内核中，*init*（所有进程的祖先）开始时具有包含所有能力的能力边界集。

如果进程具有`CAP_SETPCAP`能力，则它可以使用*prctl()*的`PR_CAPBSET_DROP`操作（不可逆地）从其边界集中删除能力。（从边界集中删除能力不会影响进程的允许、有效和可继承能力集。）进程可以使用*prctl()*的`PR_CAPBSET_READ`操作来确定某个能力是否在其边界集中。

### 注意

更准确地说，能力边界集是每个线程的属性。从 Linux 2.6.26 开始，该属性显示为 Linux 特有的`/proc/`*PID*`/task/`*TID*`/status`文件中的`CapBnd`字段。`/proc/`*PID*`/status`文件显示进程主线程的边界集。

### 保留*root*语义

为了在执行文件时保持传统的*root*用户语义（即*root*拥有所有权限），与文件相关的任何能力集都会被忽略。相反，为了在执行过程能力转换")算法中，文件能力集在*exec()*期间会被假定为如下定义：

+   如果正在执行一个设置了用户 ID 为*root*的程序，或者调用*exec()*的进程的真实或有效用户 ID 是 0，则文件的可继承集和允许集被定义为全 1。

+   如果正在执行一个设置了用户 ID 为*root*的程序，或者调用*exec()*的进程的有效用户 ID 是 0，则文件有效位被定义为已设置。

假设我们正在执行一个设置了用户 ID 为*root*的程序，那么这些假定的文件能力集定义意味着在执行过程能力转换")中的进程新允许和有效能力集的计算可以简化为如下：

```
P'(permitted) = P(inheritable) | cap_bset
P'(effective) = P'(permitted)
```

## 更改用户 ID 对进程能力的影响

为了保持与传统含义的兼容性，内核在更改进程用户 ID 时（使用*setuid()*等方法）会执行以下操作：

1.  如果真实用户 ID、有效用户 ID 或保存的用户 ID 之前的值为 0，并且由于用户 ID 的更改，这三个 ID 都变为非零值，则允许和有效的能力集将被清除（即，所有能力被永久丢弃）。  

1.  如果有效用户 ID 从 0 更改为非零值，则有效能力集将被清空（即，有效能力被丢弃，但允许集中的能力可以再次提升）。  

1.  如果有效用户 ID 从非零值更改为 0，则允许的能力集将被复制到有效能力集中（即，所有允许的能力变为有效）。  

1.  如果文件系统用户 ID 从 0 更改为非零值，则以下文件相关的能力将从有效能力集中清除：`CAP_CHOWN`、`CAP_DAC_OVERRIDE`、`CAP_DAC_READ_SEARCH`、`CAP_FOWNER`、`CAP_FSETID`、`CAP_LINUX_IMMUTABLE`（自 Linux 2.6.30 起）、`CAP_MAC_OVERRIDE` 和 `CAP_MKNOD`（自 Linux 2.6.30 起）。相反，如果文件系统用户 ID 从非零值更改为 0，则在允许集中启用的这些能力将被启用在有效集中。这些操作是为了保持对 Linux 特有的文件系统用户 ID 操作的传统语义。

## 以编程方式更改进程能力  

进程可以使用 *capset()* 系统调用或更推荐的 *libcap* API 来提升或丢弃其能力集中的能力，下面我们将描述这些方法。进程能力的更改遵循以下规则：  

1.  如果进程的有效能力集没有 `CAP_SETPCAP` 能力，则新的 *可继承* 集必须是现有可继承集和允许集的组合的子集。  

1.  新的 *可继承* 集必须是现有可继承集和能力边界集组合的子集。  

1.  新的 *允许* 集必须是现有允许集的子集。换句话说，进程不能授予自己没有的允许能力。换句话说，从允许集丢弃的能力不能重新获得。  

1.  新的 *有效* 集仅允许包含新允许集中的能力。  

#### *libcap* API  

直到此时，我们故意没有显示 *capset()* 系统调用的原型，或其对应的 *capget()*，后者用于检索进程的能力。因为这些系统调用应该避免使用。相反，应该使用 *libcap* 库中的函数。这些函数提供了一个符合撤回的 POSIX 1003.1e 草案标准，并带有一些 Linux 扩展的接口。  

出于空间原因，我们不会详细描述 *libcap* API。概括来说，我们指出，使用这些函数的程序通常会执行以下步骤：  

1.  使用*cap_get_proc()*函数从内核检索进程当前的能力集副本，并将其放入该函数在用户空间分配的结构中。（或者，我们可以使用*cap_init()*函数创建一个新的空能力集结构。）在*libcap* API 中，*cap_t*数据类型是一个指针，用来引用这些结构。

1.  使用*cap_set_flag()*函数更新用户空间结构，以提升（`CAP_SET`）和丢弃（`CAP_CLEAR`）从上一步中检索到的用户空间结构中的允许、有效和可继承能力集。

1.  使用*cap_set_proc()*函数将用户空间结构传回内核，以改变进程的能力。

1.  使用*cap_free()*函数释放由*libcap* API 在第一步中分配的结构。

### 注意

在撰写时，*libcap-ng*，一个改进版的能力库 API，仍在开发中。详情请访问[`freshmeat.net/projects/libcap-ng`](http://freshmeat.net/projects/libcap-ng)。

#### 示例程序

在示例 8-2 和总结中，我们展示了一个对用户名和密码进行验证的程序，验证内容是标准的密码数据库。我们提到，该程序需要特权才能读取影子密码文件，而该文件受保护，以防止除*root*或*shadow*组成员以外的用户读取。为该程序提供所需权限的传统方式是以*root*身份登录运行它，或者使其成为一个设置了用户 ID 为*root*的程序。我们现在展示一个修改版的程序，该程序使用了能力和*libcap* API。

为了作为普通用户读取影子密码文件，我们需要绕过标准的文件权限检查。在查看表 39-1 列出的能力时，我们发现适当的能力是`CAP_DAC_READ_SEARCH`。我们修改后的密码认证程序版本展示在示例 39-1 中。该程序在访问影子密码文件之前，使用*libcap* API 提升`CAP_DAC_READ_SEARCH`到其有效能力集中，然后在访问后立即丢弃该能力。为了让一个非特权用户使用该程序，我们必须在文件允许的能力集中设置此能力，具体操作可见以下的 shell 会话：

```
$ `sudo setcap "cap_dac_read_search=p" check_password_caps`
root's password:
$ `getcap check_password_caps`
check_password_caps = cap_dac_read_search+p
$ `./check_password_caps`
Username: `mtk`
Password:
Successfully authenticated: UID=1000
```

示例 39-1。一个识别用户身份的能力感知程序

```
`cap/check_password_caps.c`
#define _BSD_SOURCE         /* Get getpass() declaration from <unistd.h> */
#define _XOPEN_SOURCE       /* Get crypt() declaration from <unistd.h> */
#include <sys/capability.h>
#include <unistd.h>
#include <limits.h>
#include <pwd.h>
#include <shadow.h>
#include "tlpi_hdr.h"

/* Change setting of capability in caller's effective capabilities */

static int
modifyCap(int capability, int setting)
{
    cap_t caps;
    cap_value_t capList[1];

    /* Retrieve caller's current capabilities */

    caps = cap_get_proc();
    if (caps == NULL)
        return -1;

    /* Change setting of 'capability' in the effective set of 'caps'. The
       third argument, 1, is the number of items in the array 'capList'. */

    capList[0] = capability;
    if (cap_set_flag(caps, CAP_EFFECTIVE, 1, capList, setting) == -1) {
        cap_free(caps);
        return -1;
    }

    /* Push modified capability sets back to kernel, to change
       caller's capabilities */

    if (cap_set_proc(caps) == -1) {
        cap_free(caps);
        return -1;
    }

    /* Free the structure that was allocated by libcap */

    if (cap_free(caps) == -1)
        return -1;

    return 0;
}

static int              /* Raise capability in caller's effective set */
raiseCap(int capability)
{
    return modifyCap(capability, CAP_SET);
}

/* An analogous dropCap() (unneeded in this program), could be
   defined as: modifyCap(capability, CAP_CLEAR); */

static int              /* Drop all capabilities from all sets */
dropAllCaps(void)
{
    cap_t empty;
    int s;

    empty = cap_init();
    if (empty == NULL)
        return -1;

    s = cap_set_proc(empty);
    if (cap_free(empty) == -1)
        return -1;

    return s;
}

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
    if (lnmax == -1)                        /* If limit is indeterminate */
        lnmax = 256;                        /* make a guess */

    username = malloc(lnmax);
    if (username == NULL)
        errExit("malloc");

    printf("Username: ");
    fflush(stdout);
    if (fgets(username, lnmax, stdin) == NULL)
        exit(EXIT_FAILURE);                 /* Exit on EOF */

    len = strlen(username);
    if (username[len - 1] == '\n')
        username[len - 1] = '\0';           /* Remove trailing '\n' */

    pwd = getpwnam(username);
    if (pwd == NULL)
        fatal("couldn't get password record");

    /* Only raise CAP_DAC_READ_SEARCH for as long as we need it */

    if (raiseCap(CAP_DAC_READ_SEARCH) == -1)
        fatal("raiseCap() failed");

    spwd = getspnam(username);
    if (spwd == NULL && errno == EACCES)
        fatal("no permission to read shadow password file");

    /* At this point, we won't need any more capabilities,
       so drop all capabilities from all sets */

    if (dropAllCaps() == -1)
        fatal("dropAllCaps() failed");

    if (spwd != NULL)               /* If there is a shadow password record */
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
     `cap/check_password_caps.c`
```

## 创建仅具有能力的环境

在前面的页面中，我们描述了进程的用户 ID 为 0（*root*）时，在能力方面受到特殊对待的各种方式：

+   当一个进程的一个或多个用户 ID 等于 0 并将其所有用户 ID 设置为非零值时，它的许可和有效能力集会被清除。（参见第 39.6 节。）

+   当一个有效用户 ID 为 0 的进程将该用户 ID 更改为非零值时，它会失去其有效能力。当进行反向更改时，许可能力集会被复制到有效集。对于当进程的文件系统用户 ID 在 0 和非零值之间切换时，类似的过程也会发生。（参见第 39.6 节。）

+   如果一个具有真实或有效用户 ID 为 *root* 的进程执行一个程序，或者任何进程执行一个设置了用户 ID 为 *root* 的程序，则文件的可继承和许可集被认为是全 1。如果进程的有效用户 ID 为 0，或者它正在执行一个设置了用户 ID 为 *root* 的程序，则文件的有效位被认为是 1。（参见 保留 *root* 语义。）在通常情况下（即真实和有效用户 ID 都为 *root*，或正在执行一个设置了用户 ID 为 *root* 的程序），这意味着该进程获得所有能力在其许可和有效集中的权限。

在一个完全基于能力的系统中，内核将不需要执行对 *root* 的任何特殊处理。将不会有设置用户 ID 为 *root* 的程序，并且文件能力将用于授予程序所需的最小能力。

由于现有的应用程序并未设计成利用文件能力基础设施，内核必须保持对用户 ID 为 0 的进程的传统处理方式。尽管如此，我们可能希望某个应用程序运行在一个完全基于能力的环境中，在这个环境中，*root* 不会受到上述任何特殊对待。从内核 2.6.26 开始，如果启用了文件能力，Linux 提供了 *securebits* 机制，该机制控制一组每个进程的标志，允许或禁用对 *root* 的三种特殊处理方式。（准确来说，*securebits* 标志实际上是每个线程的属性。）

*securebits* 机制控制在 表 39-2 中显示的标志。标志作为一对对的 *base* 标志和相应的 *locked* 标志存在。每个 *base* 标志控制上述对 *root* 的一种特殊处理。设置相应的锁定标志是一个一次性操作，它会阻止对关联的 *base* 标志进一步更改——一旦设置，锁定标志就无法取消设置。

表 39-2. *securebits* 标志

| 标志 | 设置时的含义 |
| --- | --- |
| `SECBIT_KEEP_CAPS` | 当一个或多个用户 ID 为 0 的进程将其所有用户 ID 设置为非零值时，防止丢失已允许的能力。只有在未设置`SECBIT_NO_SETUID_FIXUP`的情况下，此标志才会生效。此标志会在*exec()*时被清除。 |
| `SECBIT_NO_SETUID_FIXUP` | 在有效用户 ID 或文件系统用户 ID 在 0 和非零值之间切换时，不更改能力。 |
| `SECBIT_NOROOT` | 如果一个真实或有效用户 ID 为 0 的进程执行*exec()*，或者执行了一个设置了用户 ID 为*root*的程序，则不授予它能力（除非可执行文件具有文件能力）。 |
| `SECBIT_KEEP_CAPS_LOCKED` | 锁定`SECBIT_KEEP_CAPS`。 |
| `SECBIT_NO_SETUID_FIXUP_LOCKED` | 锁定`SECBIT_NO_SETUID_FIXUP`。 |
| `SECBIT_NOROOT_LOCKED` | 锁定`SECBIT_NOROOT`。 |

*securebits*标志设置会在通过*fork()*创建的子进程中继承。所有标志设置在*exec()*过程中都会被保留，除了`SECBIT_KEEP_CAPS`，它会被清除，以确保与`PR_SET_KEEPCAPS`设置的历史兼容性，如下所述。

一个进程可以通过*prctl()* `PR_GET_SECUREBITS`操作检索*securebits*标志。如果进程具有`CAP_SETPCAP`能力，它可以通过*prctl()* `PR_SET_SECUREBITS`操作修改*securebits*标志。一个完全基于能力的应用程序可以通过以下调用不可逆地禁用对调用进程及其所有后代的*root*特殊处理：

```
if (prctl(PR_SET_SECUREBITS,
          /* SECBIT_KEEP_CAPS off */
          SECBIT_NO_SETUID_FIXUP | SECBIT_NO_SETUID_FIXUP_LOCKED |
          SECBIT_NOROOT | SECBIT_NOROOT_LOCKED)
        == -1)
    errExit("prctl");
```

在此调用之后，该进程及其后代获取能力的唯一途径是执行具有文件能力的程序。

#### `SECBIT_KEEP_CAPS`和*prctl()* `PR_SET_KEEPCAPS`操作

`SECBIT_KEEP_CAPS`标志可以防止当一个或多个用户 ID 为 0 的进程将其所有用户 ID 设置为非零值时丢失能力。大致来说，`SECBIT_KEEP_CAPS`提供了`SECBIT_NO_SETUID_FIXUP`所提供功能的一半。（如表 39-2 所述，`SECBIT_KEEP_CAPS`只有在未设置`SECBIT_NO_SETUID_FIXUP`时才会生效。）此标志的存在是为了提供一个与旧版*prctl()* `PR_SET_KEEPCAPS`操作相对应的*securebits*标志，二者控制相同的属性。（这两种机制的唯一区别是，进程不需要`CAP_SETPCAP`能力就可以使用*prctl()* `PR_SET_KEEPCAPS`操作。）

### 注意

早些时候，我们提到过，所有*securebits*标志在*exec()*过程中都会被保留，除了`SECBIT_KEEP_CAPS`。设置`SECBIT_KEEP_CAPS`标志是为了与其他*securebits*设置保持一致，反向处理*prctl()* `PR_SET_KEEPCAPS`操作设置的属性。

*prctl()* `PR_SET_KEEPCAPS`操作是为运行在不支持文件能力的旧内核上的设置了用户 ID 为*root*的程序设计的。这类程序仍然可以通过编程的方式根据需要丢弃和提升能力，从而提高其安全性（参考不支持文件能力的旧内核和系统）。

然而，即使这样一个设置了用户 ID 为*root*的程序丢弃了除了所需的能力之外的所有能力，它仍然保持着两个重要的特权：访问*root*拥有的文件的能力，以及通过执行程序恢复能力的能力（保持*root*语义）。永久丢弃这些特权的唯一方法是将进程的所有用户 ID 设置为非零值。但这样做通常会导致许可和有效能力集被清除（请参见更改用户 ID 对进程能力的影响，其中提到用户 ID 变化对能力的影响）。这与目标相悖，目标是永久丢弃用户 ID 0，同时保留一些能力。为了实现这一目标，可以使用*prctl()* `PR_SET_KEEPCAPS`操作来设置进程属性，防止在所有用户 ID 更改为非零值时清除许可能力集。（在这种情况下，进程的有效能力集始终会被清除，无论“保持能力”属性如何设置。）

## 发现程序所需的能力

假设我们有一个不关心能力的程序，并且该程序仅以二进制形式提供，或者我们有一个源代码过于庞大的程序，难以轻松阅读并确定其运行时可能需要哪些能力。如果程序需要特权，但不应当成为设置了用户 ID 为*root*的程序，那么我们如何确定应当分配给可执行文件的许可能力，并使用*setcap(8)*进行设置呢？回答这个问题有两种方法：

+   使用 *strace(1)* （附录 A）查看哪个系统调用因错误 `EPERM` 失败，该错误用于指示缺少所需的能力。通过查阅系统调用的手册页或内核源代码，我们可以推断出需要的能力。然而，这种方法并不完美，因为 `EPERM` 错误有时也会由于其他原因产生，其中一些可能与程序所需的能力要求无关。此外，程序可能合法地执行一个需要特权的系统调用，然后在确定它们没有特定操作的特权后改变行为。在确定可执行文件实际需要的能力时，有时很难区分这种“假阳性”情况。

+   使用内核探针，在内核执行能力检查时产生监控输出。如何实现这一点的示例见于 [Hallyn, 2007]，这是一篇由文件能力开发者之一撰写的文章。对于每个检查能力的请求，文章中的探针会记录被调用的内核函数、请求的能力以及请求程序的名称。尽管这种方法比使用 *strace(1)* 需要更多工作，但它也可以帮助我们更准确地确定程序所需的能力。

## 旧版内核和没有文件能力的系统

本节描述了在较旧内核中实现能力的各种差异。我们还描述了在不支持文件能力的内核上发生的差异。Linux 不支持文件能力的情况有两种：

+   在 Linux 2.6.24 之前，文件能力未实现。

+   自 Linux 2.6.24 起，如果内核在构建时未启用 `CONFIG_SECURITY_FILE_CAPABILITIES` 选项，则可以禁用文件能力。

### 注意

尽管 Linux 自 2.2 版本起引入了能力并允许它们附加到进程上，但文件能力的实现直到几年后才出现。文件能力未实现如此长时间的原因是政策问题，而非技术困难。（用于实现文件能力的扩展属性，自 2.6 版本起就已可用，详见第十六章。）内核开发人员的普遍观点是，要求系统管理员为每个特权程序设置和监控不同的能力集——其中一些后果细微但深远——会导致管理任务变得复杂且无法管理。相反，系统管理员熟悉现有的 UNIX 特权模型，知道要小心对待 set-user-ID 程序，并且可以使用简单的*find*命令定位系统中的 set-user-ID 和 set-group-ID 程序。然而，文件能力的开发者认为，文件能力可以通过行政管理得以实现，最终提供了一个足够有说服力的论据，使得文件能力被整合到内核中。

#### `CAP_SETPCAP`能力

在不支持文件能力的内核上（即，所有 2.6.24 之前的内核，以及自 2.6.24 以来禁用了文件能力的内核），`CAP_SETPCAP`能力的语义是不同的。根据类似于以编程方式更改进程能力中描述的规则，具有`CAP_SETPCAP`能力的进程可以理论上改变其他进程的能力。可以对另一个进程、指定进程组的所有成员，或除*init*进程和调用者本身外的系统上所有进程的能力进行更改。最后一个情况排除了*init*进程，因为它对于系统的操作至关重要。也排除了调用者，因为调用者可能试图从系统上每个其他进程中移除能力，而我们不希望移除调用者自身的能力。

然而，改变其他进程的能力仅仅是一个理论上的可能性。在较老的内核上，以及在禁用了文件能力支持的现代内核上，能力边界集（接下来会讨论）总是会屏蔽`CAP_SETPCAP`能力。

#### 能力边界集

自 Linux 2.6.25 版本以来，能力边界集是每个进程的属性。然而，在较老的内核上，能力边界集是一个系统范围的属性，影响系统上的所有进程。系统范围的能力边界集会初始化为始终屏蔽`CAP_SETPCAP`（如上所述）。

### 注意

在 2.6.25 之后的内核中，只有在内核启用了文件能力的情况下，才能从每个进程的边界集中移除能力。在这种情况下，*init*，作为所有进程的祖先，开始时具有包含所有能力的边界集，并且该边界集的副本会被其他在系统上创建的进程继承。如果禁用了文件能力，则由于上述`CAP_SETPCAP`语义的差异，*init*将以包含所有能力的边界集启动，除了`CAP_SETPCAP`。

在 Linux 2.6.25 及以后的版本中，能力边界集的语义发生了进一步的变化。正如之前所提到的（能力边界集），在 Linux 2.6.25 及更高版本中，每个进程的能力边界集充当着可以添加到进程可继承集合中的能力的限制超集。在 Linux 2.6.24 及更早版本中，系统范围的能力边界集没有这种掩码效果。（这并不需要，因为这些内核不支持文件能力。）

系统范围的能力边界集可以通过 Linux 特定的`/proc/sys/kernel/cap-bound`文件访问。一个进程必须具有`CAP_SYS_MODULE`能力才能更改`cap-bound`的内容。然而，只有*init*进程能够在此掩码中打开位；其他特权进程只能关闭位。这些限制的结果是，在不支持文件能力的系统上，我们永远无法将`CAP_SETPCAP`能力赋予某个进程。这是合理的，因为该能力可以用来破坏整个内核权限检查系统。（在不太可能的情况下，如果我们希望更改此限制，我们必须加载一个内核模块来更改集合中的值，修改*init*程序的源代码，或者更改内核源代码中能力边界集的初始化并进行内核重建。）

### 注意

令人困惑的是，尽管它是一个位掩码，但系统范围的`cap-bound`文件中的值以带符号的十进制数显示。例如，该文件的初始值是-257\。这是将位掩码的所有位（除了*(1 << 8)*）都打开时的二进制补码表示（即，二进制为 11111111 11111111 11111110 11111111）；`CAP_SETPCAP`的值为 8。

#### 在没有文件能力的系统上使用程序中的功能

即使在不支持文件能力的系统上，我们仍然可以利用能力来提高程序的安全性。我们通过以下方式做到这一点：

1.  以有效用户 ID 为 0 的进程（通常是 set-user-ID-*root*程序）来运行该程序。此类进程在其许可和有效集合中被授予所有能力（除了之前提到的`CAP_SETPCAP`）。

1.  在程序启动时，使用*libcap* API 从有效集合中丢弃所有能力，并从许可集合中丢弃所有不再需要的能力，除了可能稍后需要的那些。

1.  设置 `SECBIT_KEEP_CAPS` 标志（或使用 *prctl()* `PR_SET_KEEPCAPS` 操作实现相同的效果），以确保下一步不会丢失能力。

1.  设置所有用户 ID 为非零值，以防止进程访问 *root* 拥有的文件或通过执行 *exec()* 获取能力。

    ### 注意

    如果我们希望防止进程在 *exec()* 时重新获得特权，但又必须允许它访问 *root* 拥有的文件，我们可以通过设置 `SECBIT_NOROOT` 标志来替代前面两步的操作。（当然，允许访问 *root* 拥有的文件会带来某些安全漏洞的风险。）

1.  在程序生命周期的剩余部分，根据需要使用 *libcap* API 来提升和删除有效集中的剩余允许能力，以执行特权任务。

    一些为 Linux 2.6.24 之前版本的内核构建的应用程序使用了这种方法。

### 注意

在反对为可执行文件实现能力的内核开发者中，主文中描述的方法的一个优点是，应用程序开发者知道可执行文件需要哪些能力。相比之下，系统管理员可能无法轻松确定这一信息。

## 总结

Linux 能力机制将特权操作分为不同的类别，并允许一个进程获得某些能力，同时拒绝其他能力。这个机制相对于传统的全有或全无特权机制有了改进，在传统机制中，进程要么拥有执行所有操作的特权（用户 ID 为 0），要么没有任何特权（非零用户 ID）。自内核 2.6.24 版本起，Linux 支持将能力附加到文件上，这样进程可以通过执行程序来获取所选的能力。

## 练习

1.  修改 示例 35-2 中的程序（`sched_set.c`，位于 影响调度参数变化的特权和资源限制），使其使用文件能力，以便可以由非特权用户使用。
