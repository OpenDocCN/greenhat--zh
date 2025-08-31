## 第三十七章 守护进程

本章讨论守护进程的特点，并介绍将进程转变为守护进程所需的步骤。我们还将探讨如何使用*syslog*功能记录守护进程的消息。

## 概述

*守护进程*是具有以下特点的进程：

+   它是长期运行的。通常，守护进程在系统启动时创建，并在系统关闭时停止。

+   它在后台运行，没有控制终端。没有控制终端确保内核不会自动为守护进程生成任何作业控制或终端相关的信号（如`SIGINT, SIGTSTP`和`SIGHUP`）。

守护进程是为执行特定任务而编写的，以下是几个例子：

+   *cron*：一个在预定时间执行命令的守护进程。

+   *sshd*：安全外壳守护进程，允许使用安全通信协议从远程主机登录。

+   *httpd*：HTTP 服务器守护进程（Apache），负责提供网页服务。

+   *inetd*：互联网超级服务器守护进程（详见 The *inetd* (Internet Superserver) Daemon Daemon")），它监听指定的 TCP/IP 端口上的传入网络连接，并启动相应的服务器程序来处理这些连接。

许多标准守护进程作为特权进程运行（即有效用户 ID 为 0），因此应按照第三十八章中提供的指南进行编写。

守护进程的命名惯例（虽然不是普遍遵守）通常以字母*d*结尾。

### 注意

在 Linux 中，某些守护进程作为*内核线程*运行。这些守护进程的代码是内核的一部分，通常在系统启动时创建。当使用*ps(1)*列出时，这些守护进程的名称会被方括号（`[]`）括起来。一个内核线程的例子是*pdflush*，它定期将脏页面（例如，缓冲区缓存中的页面）刷新到磁盘。

## 创建守护进程

要成为守护进程，一个程序需要执行以下步骤：

1.  执行*fork()*，然后父进程退出，子进程继续运行。（因此，守护进程成为*init*进程的子进程。）此步骤有两个原因：

    +   假设守护进程是从命令行启动的，那么父进程的终止会被 shell 察觉，随后显示另一个 shell 提示符，并让子进程继续在后台运行。

    +   子进程保证不成为进程组的领导者，因为它从父进程继承了进程组 ID，并获得了自己的唯一进程 ID，这与继承的进程组 ID 不同。这样才能顺利执行下一步。

1.  子进程调用*setsid()*（会话）来启动一个新会话，并与控制终端断开关联。

1.  如果守护进程此后永远不会打开任何终端设备，那么我们不需要担心守护进程重新获取控制终端。如果守护进程可能在之后打开终端设备，那么我们必须采取措施确保该设备不会成为控制终端。我们可以通过两种方式来实现这一点：

    +   在任何可能应用于终端设备的*open()*操作中，指定`O_NOCTTY`标志。

    +   或者，更简单地，在*setsid()*调用后执行第二次*fork()*，并且再次让父进程退出，(孙)子进程继续执行。这可以确保子进程不是会话领导进程，因此，根据 System V 关于获取控制终端的约定（Linux 遵循该约定），该进程永远无法重新获取控制终端（控制终端和控制进程）。

    ### 注意

    在遵循 BSD 约定的实现中，进程只能通过显式的*ioctl()* `TIOCSCTTY`操作来获取控制终端，因此第二次*fork()*不会影响控制终端的获取，但多余的*fork()*不会造成任何 harm。

1.  清除进程的 umask（进程文件模式创建掩码：*umask()*")），以确保当守护进程创建文件和目录时，它们具有所请求的权限。

1.  更改进程的当前工作目录，通常是根目录(`/`)。这是必要的，因为守护进程通常会一直运行直到系统关机；如果守护进程的当前工作目录位于一个不同于包含`/`的文件系统中，那么该文件系统无法被卸载（卸载文件系统：*umount()* 和 *umount2()* 和 umount2()")）。或者，守护进程可以将其工作目录更改为它执行工作的地方或其配置文件中定义的某个位置，只要我们知道包含该目录的文件系统不需要被卸载。例如，*cron* 将自己放置在`/var/spool/cron`。

1.  关闭守护进程从其父进程继承的所有打开的文件描述符。（守护进程可能需要保持某些继承的文件描述符打开，因此此步骤是可选的，或者可以有所变化。）这样做有多种原因。由于守护进程已经失去了控制终端并在后台运行，所以如果文件描述符 0、1 和 2 指向终端，守护进程保持这些文件描述符打开是没有意义的。此外，我们无法卸载守护进程保持打开文件的文件系统。而且，通常情况下，我们应该关闭不再使用的打开文件描述符，因为文件描述符是有限的资源。

    ### 注意

    一些 UNIX 实现（例如，Solaris 9 和一些近期的 BSD 版本）提供一个名为 *closefrom(n)*（或类似）的函数，它关闭所有大于或等于 *n* 的文件描述符。这个函数在 Linux 上不可用。

1.  在关闭了文件描述符 0、1 和 2 后，守护进程通常会打开 `/dev/null`，并使用 *dup2()*（或类似）将所有这些描述符指向这个设备。这么做有两个原因：

    +   它确保如果守护进程调用执行 I/O 操作的库函数，这些函数不会因为描述符的问题而意外失败。

    +   它防止守护进程稍后打开一个文件，使用描述符 1 或 2，这样就会被一个库函数写入，从而导致这些描述符的标准输出和标准错误被破坏。

    ### 注意

    `/dev/null` 是一个虚拟设备，它总是丢弃写入的数据。当我们想要消除一个 shell 命令的标准输出或错误时，可以将其重定向到这个文件。对这个设备的读取总是返回文件末尾。

下面展示了一个函数的实现，*becomeDaemon()*，它执行上面描述的步骤，将调用者转换为守护进程。

```
#include <syslog.h>

int `becomeDaemon`(int *flags*);
```

### 注意

成功时返回 0，错误时返回 -1

*becomeDaemon()* 函数接受一个位掩码参数，*flags*，允许调用者选择性地禁止一些步骤，具体如 示例 37-1 中头文件的注释所述。

示例 37-1. `become_daemon.c` 的头文件

```
`daemons/become_daemon.h`
#ifndef BECOME_DAEMON_H             /* Prevent double inclusion */
#define BECOME_DAEMON_H

/* Bit-mask values for 'flags' argument of becomeDaemon() */

#define BD_NO_CHDIR           01    /* Don't chdir("/") */
#define BD_NO_CLOSE_FILES     02    /* Don't close all open files */
#define BD_NO_REOPEN_STD_FDS  04    /* Don't reopen stdin, stdout, and
                                      stderr to /dev/null */
#define BD_NO_UMASK0         010    /* Don't do a umask(0) */

#define BD_MAX_CLOSE  8192          /* Maximum file descriptors to close if
                                       sysconf(_SC_OPEN_MAX) is indeterminate */

int becomeDaemon(int flags);

#endif
      `daemons/become_daemon.h`
```

*becomeDaemon()* 函数的实现见 示例 37-2。

### 注意

GNU C 库提供了一个非标准函数，*daemon()*，它将调用者转换为守护进程。*glibc daemon()* 函数没有相当于我们 *becomeDaemon()* 函数中的 *flags* 参数。

示例 37-2. 创建守护进程

```
`daemons/become_daemon.c`
#include <sys/stat.h>
#include <fcntl.h>
#include "become_daemon.h"
#include "tlpi_hdr.h"

int                             /* Returns 0 on success, -1 on error */
becomeDaemon(int flags)
{
    int maxfd, fd;

    switch (fork()) {                   /* Become background process */
    case -1: return -1;
    case 0:  break;                     /* Child falls through... */
    default: _exit(EXIT_SUCCESS);       /* while parent terminates */
    }

    if (setsid() == -1)                 /* Become leader of new session */
        return -1;

    switch (fork()) {                   /* Ensure we are not session leader */
    case -1: return -1;
    case 0:  break;
    default: _exit(EXIT_SUCCESS);
    }

    if (!(flags & BD_NO_UMASK0))
        umask(0);                       /* Clear file mode creation mask */

    if (!(flags & BD_NO_CHDIR))
        chdir("/");                     /* Change to root directory */

    if (!(flags & BD_NO_CLOSE_FILES)) { /* Close all open files */
        maxfd = sysconf(_SC_OPEN_MAX);
        if (maxfd == -1)                /* Limit is indeterminate... */
            maxfd = BD_MAX_CLOSE;       /* so take a guess */

        for (fd = 0; fd < maxfd; fd++)
            close(fd);
    }

    if (!(flags & BD_NO_REOPEN_STD_FDS)) {
        close(STDIN_FILENO);            /* Reopen standard fd's to /dev/null */

        fd = open("/dev/null", O_RDWR);

        if (fd != STDIN_FILENO)         /* 'fd' should be 0 */
            return -1;
        if (dup2(STDIN_FILENO, STDOUT_FILENO) != STDOUT_FILENO)
            return -1;
        if (dup2(STDIN_FILENO, STDERR_FILENO) != STDERR_FILENO)
            return -1;
    }

    return 0;
}
      `daemons/become_daemon.c`
```

如果我们编写一个程序，调用 *becomeDaemon(0)* 后再睡一段时间，我们可以使用 *ps(1)* 查看结果进程的一些属性：

```
$ `./test_become_daemon`
$ `ps -C test_become_daemon -o "pid ppid pgid sid tty command"`
  PID  PPID  PGID   SID TT       COMMAND
24731     1 24730 24730 ?        ./test_become_daemon
```

### 注意

我们没有展示 `daemons/test_become_daemon.c` 的源代码，因为它很简单，但该程序已包含在本书的源代码分发包中。

在 *ps* 的输出中，*TT* 头下的 `?` 表示该进程没有控制终端。从进程 ID 与会话 ID (SID) 不同这一点，我们还可以看出该进程不是其会话的领导者，因此即使它打开一个终端设备，也不会重新获得控制终端。对于守护进程来说，情况应当如此。

## 编写守护进程的指南

如前所述，守护进程通常只有在系统关闭时才会终止。许多标准守护进程在系统关闭时会被由特定应用执行的脚本停止。那些没有以这种方式终止的守护进程会收到`SIGTERM`信号，这是*init*进程在系统关闭时发送给其所有子进程的信号。默认情况下，`SIGTERM`会终止一个进程。如果守护进程在终止之前需要执行任何清理工作，它应该通过为该信号建立一个处理程序来实现。这个处理程序必须设计得能够快速完成清理，因为*init*在 5 秒后会跟进发送`SIGKILL`信号。（这并不意味着守护进程可以执行 5 秒的 CPU 工作；*init*会同时向系统中的所有进程发送信号，它们可能都会在这 5 秒内尝试进行清理。）

由于守护进程的生命周期较长，我们必须特别小心可能出现的内存泄漏（*malloc()* 和 *free()* 的实现 and free()")）和文件描述符泄漏（应用程序未能关闭其打开的所有文件描述符）。如果这些 bug 影响到守护进程，唯一的解决办法就是杀死该进程并在修复 bug 后重新启动它。

许多守护进程需要确保一次只有一个实例在运行。例如，让两个*cron*守护进程都尝试执行定时任务是没有意义的。在只运行一个程序实例中，我们探讨了一种实现这一目标的技术。

## 使用`SIGHUP`重新初始化守护进程

许多守护进程需要持续运行，这就提出了几个编程上的难题：

+   通常，守护进程在启动时会从关联的配置文件中读取操作参数。有时，可能希望能够在运行时动态改变这些参数，而无需停止和重新启动守护进程。

+   一些守护进程会生成日志文件。如果守护进程从不关闭日志文件，那么日志文件可能会不断增长，最终导致文件系统堵塞。（在创建和删除（硬）链接：*link*() 和 *unlink*() Links: link() and unlink()")中，我们提到过，即使我们删除文件的最后一个名称，只要有任何进程打开该文件，文件仍然会存在。）我们需要的是一种方法来告诉守护进程关闭日志文件并打开一个新文件，这样我们就可以根据需要旋转日志文件。

这两个问题的解决方案是让守护进程为`SIGHUP`信号建立一个处理程序，并在接收到该信号后执行所需的步骤。在控制终端和控制进程中，我们提到，`SIGHUP`信号是在控制终端断开连接时生成的，因为守护进程没有控制终端，所以内核不会为守护进程生成此信号。因此，守护进程可以将`SIGHUP`用于这里描述的目的。

### 注

*logrotate*程序可用于自动轮转守护进程的日志文件。有关详细信息，请参阅*logrotate(8)*手册页。

示例 37-3 提供了一个守护进程如何使用`SIGHUP`的示例。这个程序建立了一个`SIGHUP`信号的处理程序 ![](img/U002.png)，然后成为守护进程 ![](img/U003.png)，打开日志文件 ![](img/U004.png)，并读取其配置文件 ![](img/U005.png)。`SIGHUP`信号处理程序 ![](img/U001.png) 仅设置了一个全局标志变量，*hupReceived*，该变量由主程序检查。主程序在一个循环中运行，每 15 秒打印一条信息到日志文件 ![](img/U008.png)。循环中的*sleep()*调用 ![](img/U006.png) 用于模拟真实应用程序的某些处理。每次从*sleep()*返回时，程序都会检查*hupReceived*是否已设置 ![](img/U007.png)；如果是，它会重新打开日志文件，重新读取配置文件，并清除*hupReceived*标志。

为了简洁起见，*logOpen()*, *logClose()*, *logMessage()*和*readConfigFile()*函数在示例 37-3 中被省略，但它们随本书的源代码分发。前三个函数的功能符合它们的名称。*readConfigFile()*函数只是读取配置文件中的一行并将其回显到日志文件中。

### 注

一些守护进程在接收到`SIGHUP`信号时使用另一种方法来重新初始化自己：它们关闭所有文件，然后通过*exec()*重新启动自己。

以下是运行程序时可能看到的示例，来自示例 37-3。我们首先创建一个虚拟的配置文件，然后启动守护进程：

```
$ `echo START > /tmp/ds.conf`
$ `./daemon_SIGHUP`
$ `cat /tmp/ds.log`                                     *View log file*
2011-01-17 11:18:34: Opened log file
2011-01-17 11:18:34: Read config file: START
```

现在我们修改配置文件并重命名日志文件，然后向守护进程发送`SIGHUP`信号：

```
$ `echo CHANGED > /tmp/ds.conf`
$ `date +'%F %X'; mv /tmp/ds.log /tmp/old_ds.log`
2011-01-17 11:19:03 AM
$ `date +'%F %X'; killall -HUP daemon_SIGHUP`
2011-01-17 11:19:23 AM
$ `ls /tmp/*ds.log`                                     *Log file was reopened*
/tmp/ds.log  /tmp/old_ds.log
$ `cat /tmp/old_ds.log`                                 *View old log file*
2011-01-17 11:18:34: Opened log file
2011-01-17 11:18:34: Read config file: START
2011-01-17 11:18:49: Main: 1
2011-01-17 11:19:04: Main: 2
2011-01-17 11:19:19: Main: 3
2011-01-17 11:19:23: Closing log file
```

*ls* 命令的输出显示我们有一个旧日志文件和一个新日志文件。当我们使用 *cat* 查看旧日志文件的内容时，我们发现即使在使用 *mv* 命令重命名文件后，守护进程仍然继续在该文件中记录日志消息。此时，如果不再需要该旧日志文件，我们可以删除它。当我们查看新日志文件时，我们看到配置文件已经重新读取：

```
$ `cat /tmp/ds.log`
2011-01-17 11:19:23: Opened log file
2011-01-17 11:19:23: Read config file: CHANGED
2011-01-17 11:19:34: Main: 4
$ `killall daemon_SIGHUP`                               *Kill our daemon*
```

请注意，守护进程的日志和配置文件通常放置在标准目录中，而不是像示例 37-3 中的程序那样放在 `/tmp` 目录中。按照惯例，配置文件放置在 `/etc` 或其子目录中，而日志文件通常放在 `/var/log` 中。守护进程程序通常提供命令行选项，以便指定替代位置，而不是使用默认位置。

示例 37-3. 使用 `SIGHUP` 重新初始化守护进程

```
`daemons/daemon_SIGHUP.c`
    #include <sys/stat.h>
    #include <signal.h>
    #include "become_daemon.h"
    #include "tlpi_hdr.h"

    static const char *LOG_FILE = "/tmp/ds.log";
    static const char *CONFIG_FILE = "/tmp/ds.conf";

    /* Definitions of logMessage(), logOpen(), logClose(), and
       readConfigFile() are omitted from this listing */

    static volatile sig_atomic_t hupReceived = 0;
                                        /* Set nonzero on receipt of SIGHUP */
     from
    static void
    sighupHandler(int sig)
    {
    hupReceived = 1;
    }

    int
    main(int argc, char *argv[])
    {
        const int SLEEP_TIME = 15;      /* Time to sleep between messages */
        int count = 0;                  /* Number of completed SLEEP_TIME intervals */
        int unslept;                    /* Time remaining in sleep interval */
        struct sigaction sa;

        sigemptyset(&sa.sa_mask);
        sa.sa_flags = SA_RESTART;
        sa.sa_handler = sighupHandler;
    if (sigaction(SIGHUP, &sa, NULL) == -1)
                   errExit("sigaction");
    if (becomeDaemon(0) == -1)
            errExit("becomeDaemon");

    logOpen(LOG_FILE);
    readConfigFile(CONFIG_FILE);

        unslept = SLEEP_TIME;

        for (;;) {
        unslept = sleep(unslept);       /* Returns > 0 if interrupted */
        if (hupReceived) {              /* If we got SIGHUP... */
                logClose();
                    logOpen(LOG_FILE);
                readConfigFile(CONFIG_FILE);
                hupReceived = 0;            /* Get ready for next SIGHUP */
            }

            if (unslept == 0) {             /* On completed interval */
                count++;
            logMessage("Main: %d", count);
                unslept = SLEEP_TIME;       /* Reset interval */
            }
        }
    }
          `daemons/daemon_SIGHUP.c`
```

## 使用 *syslog* 记录消息和错误

在编写守护进程时，我们遇到的一个问题是如何显示错误消息。由于守护进程在后台运行，我们无法像其他程序那样在关联的终端上显示消息。一个可能的替代方法是将消息写入特定于应用程序的日志文件，就像在示例 37-3 中的程序那样。这个方法的主要问题是，系统管理员很难管理多个应用程序的日志文件并同时监控这些日志中的错误消息。*syslog* 功能就是为了解决这个问题而设计的。

#### 概述

*syslog* 功能提供了一个单一的、集中式的日志记录功能，供系统上的所有应用程序使用。有关该功能的概述请参见图 37-1。

![系统日志概述](img/37-1_DAEMON-syslog-scale90.png.jpg)图 37-1. 系统日志概述

*syslog* 功能有两个主要组件：*syslogd* 守护进程和 *syslog(3)* 库函数。

*系统日志* 守护进程，*syslogd*，接受来自两个不同来源的日志消息：一个是 UNIX 域套接字 `/dev/log`，用于存储本地生成的消息，另一个是（如果启用）互联网域套接字（UDP 端口 514），用于存储通过 TCP/IP 网络发送的消息。（在其他某些 UNIX 实现中，*syslog* 套接字位于 `/var/run/log`。）

每条由*syslogd*处理的消息都有若干属性，包括一个*facility*，它指定生成该消息的程序类型，以及一个*level*，它指定消息的严重性（优先级）。*syslogd*守护进程检查每条消息的*facility*和*level*，然后根据相关配置文件`/etc/syslog.conf`的规定将其传递给多个可能的目的地。可能的目的地包括终端或虚拟控制台、磁盘文件、FIFO、一个或多个（或所有）登录的用户，或通过 TCP/IP 网络连接的另一系统上的进程（通常是另一个*syslogd*守护进程）。将消息发送到另一个系统上的进程对于通过将来自多个系统的消息集中到一个位置来减少管理开销非常有用。一条消息可以被发送到多个目的地（或完全不发送），并且具有不同*facility*和*level*组合的消息可以定向到不同的目的地或不同实例的目的地（即，不同的控制台、不同的磁盘文件等）。

### 注意

通过 TCP/IP 网络将*syslog*消息发送到另一个系统，也有助于检测系统入侵。入侵通常会在系统日志中留下痕迹，但攻击者通常会通过删除日志记录来掩盖他们的活动。使用远程日志记录，攻击者需要侵入另一个系统才能做到这一点。

*syslog(3)*库函数可以被任何进程用来记录消息。这个函数，我们稍后会详细描述，使用其提供的参数构造一个标准格式的消息，然后将其放置在`/dev/log`套接字中，供*syslogd*读取。

另一种将消息放置到`/dev/log`的来源是*内核日志*守护进程*klogd*，它收集内核日志消息（由内核使用*printk()*函数生成）。这些消息通过两种等效的 Linux 特有接口收集——`/proc/kmsg`文件和*syslog(2)*系统调用——然后使用*syslog(3)*库函数将它们放置到`/dev/log`。

### 注意

尽管*syslog(2)*和*syslog(3)*共享相同的名称，但它们执行的任务是完全不同的。*syslog(2)*的接口在*glibc*中提供，名为*klogctl()*。除非另有明确说明，本节中提到的*syslog()*指的是*syslog(3)*。

*syslog*功能最初出现在 4.2BSD 中，但现在在大多数 UNIX 实现中都可以找到。SUSv3 已对*syslog(3)*及相关函数进行了标准化，但未指定*syslogd*的实现和操作方式，以及`syslog.conf`文件的格式。Linux 对*syslogd*的实现与原始 BSD 功能有所不同，允许在`syslog.conf`中指定某些消息处理规则的扩展。

#### `syslog` API

*syslog* API 由三个主要函数组成：

+   *openlog()* 函数建立默认设置，这些设置适用于后续对 *syslog()* 的调用。使用 *openlog()* 是可选的。如果省略它，则在第一次调用 *syslog()* 时，连接日志设施并使用默认设置。

+   *syslog()* 函数记录一条消息。

+   *closelog()* 函数在完成日志记录后调用，用于断开与日志的连接。

这些函数都不返回状态值。部分原因是系统日志应该始终可用（系统管理员很快会注意到如果它不可用）。此外，如果系统日志发生错误，应用程序通常也无力有效地报告它。

### 注意

GNU C 库还提供了 *void vsyslog(int priority, const char *format, va_list args)* 函数。这个函数执行与 *syslog()* 相同的任务，但接受一个由 *stdarg(3)* API 之前处理过的参数列表。（因此，*vsyslog()* 对 *syslog()* 就像 *vprintf()* 对 *printf()*。）SUSv3 没有指定 *vsyslog()*，并且并非所有 UNIX 实现都提供此函数。

#### 建立与系统日志的连接

*openlog()* 函数可选择性地建立与系统日志设施的连接，并设置适用于后续 *syslog()* 调用的默认值。

```
#include <syslog.h>

void `openlog`(const char **ident*, int *log_options*, int *facility*);
```

*ident* 参数是指向字符串的指针，该字符串包含在 *syslog()* 写入的每条消息中；通常，这个参数指定程序名称。请注意，*openlog()* 仅仅复制这个指针的值。只要它继续调用 *syslog()*，应用程序应确保引用的字符串在后续不会被修改。

### 注意

如果 *ident* 被指定为 `NULL`，则与一些其他实现类似，*glibc syslog* 实现会自动使用程序名称作为 *ident* 值。然而，SUSv3 并未要求此功能，且某些实现不提供此功能。便携式应用程序应避免依赖此功能。

*openlog()* 的 *log_options* 参数是通过将以下常量进行或运算生成的位掩码：

`LOG_CONS`

如果发送到系统日志程序时发生错误，则将消息写入系统控制台（`/dev/console`）。

`LOG_NDELAY`

立即打开与日志系统的连接（即，底层的 UNIX 域套接字 `/dev/log`）。默认情况下（`LOG_ODELAY`），只有在第一次使用 *syslog()* 记录消息时才会打开连接。如果需要精确控制何时分配 `/dev/log` 的文件描述符，`O_NDELAY` 标志很有用。一种需要此功能的情况是调用 *chroot()* 的程序。在 *chroot()* 调用后，`/dev/log` 路径将不再可见，因此必须在 *chroot()* 之前执行指定 `LOG_NDELAY` 的 *openlog()* 调用。*tftpd*（Trivial File Transfer）守护进程就是使用 `LOG_NDELAY` 达到这一目的的程序之一。

`LOG_NOWAIT`

不要*wait()*任何可能已经创建的子进程以记录消息。在需要为日志消息创建子进程的实现中，如果调用者同时创建并等待子进程，则需要`LOG_NOWAIT`，以避免*syslog()*试图等待一个已经被调用者收割的子进程。在 Linux 上，`LOG_NOWAIT`没有效果，因为在记录消息时不会创建任何子进程。

`LOG_ODELAY`

该标志是`LOG_NDELAY`的反义操作——连接到日志系统的过程会延迟，直到记录了第一条消息。这是默认行为，无需显式指定。

`LOG_PERROR`

将消息写入标准错误，并同时写入系统日志。通常，守护进程会关闭标准错误或将其重定向到`/dev/null`，在这种情况下，`LOG_PERROR`就没有用处。

`LOG_PID`

将调用者的进程 ID 与每条消息一起记录。将`LOG_PID`应用于一个产生多个子进程的服务器时，可以帮助我们区分哪个进程记录了特定的消息。

上述所有常量都在 SUSv3 中进行了规定，除了`LOG_PERROR`，该常量出现在许多（但不是所有）其他 UNIX 实现中。

*openlog()*的*facility*参数指定在后续的*syslog()*调用中要使用的默认*facility*值。此参数的可能值列在表 37-1 and the priority argument of syslog()")中。

表 37-1 and the priority argument of syslog()")中的大多数*facility*值出现在 SUSv3 中，如表格中的*SUSv3*列所示。例外情况是`LOG_AUTHPRIV`和`LOG_FTP`，它们只出现在少数其他 UNIX 实现中，而`LOG_SYSLOG`则出现在大多数实现中。`LOG_AUTHPRIV`值对于将包含密码或其他敏感信息的日志消息记录到不同的位置（而不是`LOG_AUTH`）非常有用。

`LOG_KERN` *facility*值用于内核消息。此类消息无法由用户空间程序生成。`LOG_KERN`常量的值为 0。如果在*syslog()*调用中使用，它的 0 会转换为“使用默认级别”。

表 37-1. *openlog()*的*facility*值和*syslog()*的*priority*参数

| 值 | 描述 | SUSv3 |
| --- | --- | --- |
| `LOG_AUTH` | 安全性和授权消息（例如，*su*） | • |
| `LOG_AUTHPRIV` | 私密安全和授权消息 |   |
| `LOG_CRON` | 来自*cron*和*at*守护进程的消息 | • |
| `LOG_DAEMON` | 来自其他系统守护进程的消息 | • |
| `LOG_FTP` | 来自*ftp*守护进程（*ftpd*）的消息 |   |
| `LOG_KERN` | 内核消息（无法由用户进程生成） | • |
| `LOG_LOCAL0` | 保留用于本地使用（同样适用于`LOG_LOCAL1`至`LOG_LOCAL7`） | • |
| `LOG_LPR` | 来自行式打印机系统（*lpr*，*lpd*，*lpc*）的消息 | • |
| `LOG_MAIL` | 来自邮件系统的消息 | • |
| `LOG_NEWS` | 与 Usenet 网络新闻相关的消息 | • |
| `LOG_SYSLOG` | 来自*syslogd*守护进程的内部消息 |   |
| `LOG_USER` | 用户进程生成的消息（默认） | • |
| `LOG_UUCP` | 来自 UUCP 系统的消息 | • |

#### 记录消息

要写入日志消息，我们调用*syslog()*。

```
#include <syslog.h>

void `syslog`(int *priority*, const char **format*, ...);
```

*priority*参数是通过将*facility*值和*level*值按位或（OR）组合起来创建的。*facility*表示记录消息的应用程序的通用类别，并作为表 37-1 and the priority argument of syslog()")中列出的值之一进行指定。如果省略，*facility*将默认为先前*openlog()*调用中指定的值，或者如果该调用被省略，则默认为`LOG_USER`。*level*值表示消息的严重性，并作为表 37-2 (from highest to lowest severity)")中的值之一进行指定。此表中列出的所有*level*值都出现在 SUSv3 中。

表 37-2. *syslog()*的*priority*参数的*level*值（从最高到最低严重性）

| 值 | 描述 |
| --- | --- |
| `LOG_EMERG` | 紧急或危急情况（系统无法使用） |
| `LOG_ALERT` | 需要立即采取行动的情况（例如，损坏的系统数据库） |
| `LOG_CRIT` | 严重条件（例如，磁盘设备错误） |
| `LOG_ERR` | 一般错误条件 |
| `LOG_WARNING` | 警告消息 |
| `LOG_NOTICE` | 可能需要特别处理的正常情况 |
| `LOG_INFO` | 信息性消息 |
| `LOG_DEBUG` | 调试消息 |

*syslog()*的其余参数是格式化字符串及其相应参数，类似于*printf()*。与*printf()*的一个不同之处在于，格式化字符串不需要包含终止的换行符。此外，格式化字符串可以包含 2 个字符的序列`%m`，它会被与当前*errno*值对应的错误字符串替换（即相当于*strerror(errno)*）。

以下代码演示了*openlog()*和*syslog()*的使用：

```
openlog(argv[0], LOG_PID | LOG_CONS | LOG_NOWAIT, LOG_LOCALO);
syslog(LOG_ERR, "Bad argument: %s", argv[1]);
syslog(LOG_USER | LOG_INFO, "Exiting");
```

由于在第一次*syslog()*调用中未指定*facility*，因此使用*openlog()*指定的默认值（`LOG_LOCAL0`）。在第二次*syslog()*调用中，显式指定`LOG_USER`会覆盖由*openlog()*建立的默认值。

### 注意

在 Shell 中，我们可以使用*logger(1)*命令将条目添加到系统日志中。该命令允许指定与日志消息关联的*level*（*priority*）和*ident*（*tag*）。有关更多详细信息，请参阅*logger(1)*手册页。*logger*命令（弱规范）在 SUSv3 中有所规定，并且该命令的版本在大多数 UNIX 实现中都可用。

使用*syslog()*以以下方式写入用户提供的字符串是一个错误：

```
syslog(priority, user_supplied_string);
```

这段代码的问题在于，它让应用程序容易受到所谓的 *格式字符串攻击*。如果用户提供的字符串包含格式说明符（例如 `%s`），那么结果是不可预测的，并且从安全角度来看，可能是危险的。（同样的观察适用于传统的 *printf()* 函数的使用。）我们应该将上面的调用重写为如下：

```
syslog(priority, "%s", user_supplied_string);
```

#### 关闭日志

当我们完成日志记录时，可以调用 *closelog()* 来释放用于 `/dev/log` 套接字的文件描述符。

```
#include <syslog.h>

void `closelog`(void);
```

由于守护进程通常会持续保持与系统日志的连接，因此通常省略调用 *closelog()*。

#### 过滤日志消息

*setlogmask()* 函数设置一个掩码，用于过滤 *syslog()* 写入的消息。

```
#include <syslog.h>

int `setlogmask`(int *mask_priority*);
```

### 注意

返回之前的日志优先级掩码

任何其 *level* 不在当前掩码设置中的消息都会被丢弃。默认的掩码值允许记录所有严重性级别的消息。

宏 `LOG_MASK()`（在 `<syslog.h>` 中定义）将 表 37-2 优先级参数的 level 值（从最高到最低严重性）") 中的 *level* 值转换为适合传递给 *setlogmask()* 的位值。例如，要丢弃除 `LOG_ERR` 及以上优先级的所有消息，我们可以进行如下调用：

```
setlogmask(LOG_MASK(LOG_EMERG) | LOG_MASK(LOG_ALERT) |
           LOG_MASK(LOG_CRIT) | LOG_MASK(LOG_ERR));
```

`LOG_MASK()` 宏由 SUSv3 定义。大多数 UNIX 实现（包括 Linux）还提供了未指定的宏 `LOG_UPTO()`，该宏创建一个位掩码，过滤掉某个特定*级别*及以上的所有消息。使用此宏，我们可以简化之前的 *setlogmask()* 调用，改为如下：

```
setlogmask(LOG_UPTO(LOG_ERR));
```

#### `/etc/syslog.conf` 文件

`/etc/syslog.conf` 配置文件控制 *syslogd* 守护进程的操作。该文件由规则和注释（以 `#` 字符开头）组成。规则的一般形式如下：

```
*facility.level       action*
```

*facility* 和 *level* 一起被称为 *selector*，因为它们选择应用规则的消息。这些字段是与 表 37-1 和 syslog() 的优先级参数的 facility 值") 和 表 37-2 优先级参数的 level 值（从最高到最低严重性）") 中列出的值对应的字符串。*action* 指定将匹配此 *selector* 的消息发送到何处。空格分隔规则的 *selector* 部分和 *action* 部分。以下是规则的示例：

```
*.err                           /dev/tty10
auth.notice                     root
*.debug;mail.none;news.none     -/var/log/messages
```

第一条规则表示来自所有设施（`*`）且*level*为`err`（`LOG_ERR`）或更高的消息应发送到`/dev/tty10`控制台设备。第二条规则表示来自授权设施（`LOG_AUTH`）且*level*为`notice`（`LOG_NOTICE`）或更高的消息应发送到任何*root*登录的控制台或终端。这个特定的规则允许已登录的*root*用户立即看到有关失败的*su*尝试的消息，例如。

最后一条规则展示了规则语法的多个高级特性。一条规则可以包含多个由分号分隔的选择器。第一个选择器指定*所有*消息，使用`*`通配符表示*facility*，并使用`debug`表示*level*，这意味着所有`debug`级别（最低级别）及以上的消息。（在 Linux 上，像某些其他 UNIX 实现一样，可以将*level*指定为`*`，其含义与`debug`相同。然而，并非所有*syslog*实现都支持此特性。）通常，包含多个选择器的规则会匹配与任何选择器对应的消息，但指定*level*为`none`的规则会*排除*所有属于相应*facility*的消息。因此，这条规则将所有消息（除了`mail`和`news`设施的消息）发送到文件`/var/log/messages`。文件名前的连字符（`-`）指定每次写入文件时不会同步到磁盘（参见控制内核文件 I/O 缓冲）。这意味着写入速度更快，但如果系统在写入后立即崩溃，某些数据可能会丢失。

每当我们更改`syslog.conf`文件时，必须按照常规方式要求守护进程从该文件重新初始化：

```
$ `killall -HUP syslogd`                  *Send* SIGHUP *to syslogd*
```

### 注意

`syslog.conf`规则语法的其他特性允许创建比我们展示的更强大的规则。完整的细节请参考*syslog.conf(5)*手册页。

## 总结

守护进程是一个没有控制终端（即在后台运行）的长时间运行的进程。守护进程执行特定任务，例如提供网络登录功能或提供网页服务。为了成为一个守护进程，程序执行一系列标准步骤，包括调用*fork()*和*setsid()*。

在适当的情况下，守护进程应正确处理`SIGTERM`和`SIGHUP`信号的到达。`SIGTERM`信号应导致守护进程有序关闭，而`SIGHUP`信号则提供了一种通过重新读取配置文件并重新打开可能正在使用的任何日志文件来触发守护进程重新初始化的方式。

*syslog* 功能为守护进程（以及其他应用程序）提供了一种方便的方式，将错误和其他消息记录到一个中央位置。这些消息由 *syslogd* 守护进程处理，按照 `syslogd.conf` 配置文件的指示重新分发消息。消息可以被重新分发到多个目标，包括终端、磁盘文件、已登录用户，并通过 TCP/IP 网络发送到远程主机上的其他进程（通常是其他 *syslogd* 守护进程）。

#### 更多信息

也许关于编写守护进程的最佳信息来源是各种现有守护进程的源代码。

## 练习

1.  编写一个程序（类似于 *logger(1)*），使用 *syslog(3)* 将任意消息写入系统日志文件。除了接受包含要记录消息的单个命令行参数外，程序还应该允许指定消息的 *级别* 选项。
