## 附录 E. 进一步的信息来源

除了本书中的内容外，还有许多关于 Linux 系统编程的其他信息来源。本附录简要介绍了其中的一些。

## 手册页

手册页可以通过 *man* 命令访问。（命令 *man man* 描述了如何使用 *man* 阅读手册页。）手册页被划分为编号的部分，将信息分类如下：

1.  *程序和 shell 命令*：用户在 shell 提示符下执行的命令。

1.  *系统调用*：Linux 系统调用。

1.  *库函数*：标准 C 库函数（以及许多其他库函数）。

1.  *特殊文件*：特殊文件，如设备文件。

1.  *文件格式*：如系统密码文件 (`/etc/passwd`) 和组文件 (`/etc/group`) 等文件的格式。

1.  *游戏*：游戏。

1.  *概述、约定、协议和杂项*：各类主题的概述，以及关于网络协议和套接字编程的各种页面。

1.  *系统管理命令*：主要供超级用户使用的命令。

在某些情况下，不同部分的手册页有相同的名称。例如，*chmod* 命令有一个第一部分的手册页，*chmod()* 系统调用有一个第二部分的手册页。为了区分具有相同名称的手册页，我们在名称后面用括号括上部分号——例如，*chmod(1)* 和 *chmod(2)*。要显示特定部分的手册页，可以在 *man* 命令中插入部分号：

```
$ `man 2 chmod`
```

系统调用和库函数的手册页被分为多个部分，通常包括以下内容：

+   *名称*：函数的名称，后面附有一行简短描述。可以使用以下命令来获取所有简短描述包含指定字符串的手册页列表：

    ```
    $ `man -k` ``*`string`*``
    ```

    如果我们记不住或不确定正在寻找哪个手册页，这会很有帮助。

+   *函数原型*：函数的 C 原型。它标明了函数参数的类型和顺序，以及函数返回值的类型。在大多数情况下，函数原型之前会列出头文件。这些头文件定义了使用此函数所需的宏和 C 类型，以及函数原型本身，并应包含在使用此函数的程序中。

+   *描述*：描述函数的功能。

+   *返回值*：描述函数返回的值的范围，包括函数如何向调用者报告错误。

+   *错误*：列出在发生错误时可能返回的 *errno* 值。

+   *符合标准*：描述函数符合的各种 UNIX 标准。这让我们了解该函数在其他 UNIX 实现中的可移植性，也能识别函数的 Linux 特性。

+   *错误*：描述无法正常工作或存在问题的事项。

    ### 注意

    尽管一些后来的商业 UNIX 实现更倾向于使用市场化的委婉语，但从早期开始，UNIX 手册页就将 bug 称为 bug。Linux 延续了这一传统。有时这些“bug”是哲学性的，仅仅描述了事物改进的方式，或者警告特殊或意外（但实际上是预期的）行为。

+   *注释*：关于函数的其他杂项附加说明。

+   *另见*：相关函数和命令的手册页列表。

描述内核和 *glibc* API 的手册页可以在 [`www.kernel.org/doc/man-pages/`](http://www.kernel.org/doc/man-pages/) 在线查看。

## GNU *info* 文档

与其使用传统的手册页格式，GNU 项目使用 *info* 文档来记录其大部分软件，*info* 文档是可以通过 *info* 命令浏览的超链接文档。关于使用 *info* 的教程可以通过命令 *info info* 获取。

虽然在许多情况下，手册页和相应的 *info* 文档中的信息是相同的，但有时 C 库的 *info* 文档包含了手册页中没有的附加信息，反之亦然。

### 注意

虽然手册页和 *info* 文档存在的原因有些“宗教性质”，即 GNU 项目偏好 *info* 用户界面，因此所有文档都通过 *info* 提供。然而，UNIX 系统上的用户和程序员长期以来使用（并且在许多情况下更偏好）手册页，因此支持这种格式的动力非常强。手册页也往往包含比 *info* 文档更多的历史信息（例如，关于版本间行为变化的信息）。

## GNU C 库（*glibc*）手册

GNU C 库包含一本手册，描述了库中许多函数的使用方法。该手册可以在 [`www.gnu.org/`](http://www.gnu.org/) 上获取。它也随着大多数发行版以 HTML 格式和 *info* 格式（通过命令 *info libc*）提供。

## 书籍

本书末尾提供了大量的参考书目，但有几本书值得特别提及。

排在首位的是已故 W. Richard Stevens 的书籍。*UNIX 环境高级编程*（[Stevens, 1992]）详细介绍了 UNIX 系统编程，重点是 POSIX、System V 和 BSD。Stephen Rago 的最新修订版，[Stevens & Rago, 2005]，更新了现代标准和实现，并增加了线程相关内容以及网络编程章节。这本书是查找本书中许多话题的另一种视角的好地方。两卷本的 *UNIX 网络编程*（[Stevens et al., 2004]，[Stevens, 1999]）提供了极为详细的 UNIX 系统上的网络编程和进程间通信的内容。

### 注意

[Stevens 等人, 2004] 是 Bill Fenner 和 Andrew Rudoff 对 [Stevens, 1998] 的修订版，后者是 *UNIX 网络编程* 第 1 卷的前一版本。尽管修订版涵盖了几个新领域，但在大多数情况下，当我们引用 [Stevens 等人, 2004] 时，相同的内容也可以在 [Stevens, 1998] 中找到，尽管章节和小节的编号不同。

*高级 UNIX 编程* ([Rochkind, 1985]) 是一本简短且有时幽默的 UNIX（System V）编程入门书籍。如今，它已经有了更新和扩展的第二版 ([Rochkind, 2004])。

POSIX 线程 API 在 *编程与 POSIX 线程* ([*Butenhof*, 1996]) 中有详细描述。

*Linux 和 Unix 哲学* ([Gancarz, 2003]) 是对 Linux 和 UNIX 系统中应用设计哲学的简要介绍。

各种书籍介绍了如何阅读和修改 Linux 内核源代码，包括 *Linux 内核开发* ([Love, 2010]) 和 *理解 Linux 内核* ([Bovet & Cesati, 2005])。

关于 UNIX 内核的更一般背景，*UNIX 操作系统的设计* ([Bach, 1986]) 仍然非常易读，且包含与 Linux 相关的内容。*UNIX 内部：新前沿* ([Vahalia, 1996]) 对更现代的 UNIX 实现的内核内部进行了概述。

编写 Linux 设备驱动程序的必要参考书是 *Linux 设备驱动程序* ([Corbet 等人, 2005])。

*操作系统：设计与实现* ([Tanenbaum & Woodhull, 2006]) 通过 Minix 的例子描述了操作系统的实现。（参见 [`www.minix3.org/`](http://www.minix3.org/)）

## 现有应用程序的源代码

查看现有应用程序的源代码通常可以提供如何使用特定系统调用和库函数的良好示例。在使用 RPM 包管理器的 Linux 发行版中，我们可以通过以下方式找到包含特定程序（如 *ls*）的包：

```
$ `which ls`                      *Find pathname of* *ls* *program*
/bin/ls
$ `rpm -qf /bin/ls`
               *Find out which package created the pathname* /bin/ls
coreutils-5.0.75
```

相应的源代码包将具有类似于上述的名称，但带有后缀 `.src.rpm`。这个包将存在于发行版的安装介质中，或者可以从发行商的网站上下载。一旦我们获得该包，就可以使用 *rpm* 命令安装它，然后检查源代码，通常这些代码会被放置在 `/usr/src` 下的某个目录中。

在使用 Debian 包管理器的系统上，过程类似。我们可以通过以下命令确定创建路径名的包（以 *ls* 程序为例）：

```
$ `dpkg -S /bin/ls`
coreutils: /bin/ls
```

## Linux 文档项目

Linux 文档项目 ([`www.tldp.org/`](http://www.tldp.org/)) 提供免费的 Linux 文档，包括 HOWTO 指南和常见问题解答（FAQ），涵盖了各种系统管理和编程主题。该网站还提供了关于多个主题的更为详细的电子书。

## GNU 项目

GNU 项目 ([`www.gnu.org/`](http://www.gnu.org/)) 提供大量的软件源代码和相关文档。

## 新闻组

Usenet 新闻组通常是解答具体编程问题的好来源。以下新闻组尤其值得关注：

+   *comp.unix.programmer* 讨论一般的 UNIX 编程问题。

+   *comp.os.linux.development.apps* 讨论与应用开发特别是在 Linux 上相关的问题。

+   *comp.os.linux.development.system* 讨论 Linux 系统开发新组，重点是修改内核、开发设备驱动程序和可加载模块的问题。

+   *comp.programming.threads* 讨论使用线程进行编程，特别是 POSIX 线程。

+   *comp.protocols.tcp-ip* 讨论 TCP/IP 网络协议套件。

许多 Usenet 新闻组的常见问题解答可以在 [`www.faqs.org/`](http://www.faqs.org/) 上找到。

### 注意

在向新闻组发布问题之前，检查该组的常见问题解答（通常会定期发布在该组内部），并尝试通过网络搜索找到问题的解决方案。网站 [`groups.google.com/`](http://groups.google.com/) 提供了一个基于浏览器的接口，用于搜索旧的 Usenet 帖子。

## Linux 内核邮件列表

Linux 内核邮件列表（LKML）是 Linux 内核开发者的主要广播通信媒介。它提供了内核开发的动态，并且是提交内核错误报告和补丁的论坛。（LKML 不是系统编程问题的论坛。）要订阅 LKML，请发送电子邮件到 majordomo@vger.kernel.org，邮件正文包含以下内容作为一行：

```
subscribe linux-kernel
```

关于列表服务器的工作信息，发送包含单词“help”的消息正文到相同的地址。

要向 LKML 发送消息，使用地址 linux-kernel@vger.kernel.org。关于此邮件列表的常见问题解答和一些可搜索的档案链接，可以访问 [`www.kernel.org/`](http://www.kernel.org/)。

## 网站

以下网站特别值得关注：

+   [`www.kernel.org/`](http://www.kernel.org/)，*The Linux Kernel Archives*，包含所有版本的 Linux 内核源代码，涵盖过去和现在的版本。

+   [`www.lwn.net/`](http://www.lwn.net/)，*Linux Weekly News*，提供关于各种 Linux 相关主题的每日和每周专栏。每周的内核开发专栏总结了 LKML 的流量。

+   [`www.kernelnewbies.org/`](http://www.kernelnewbies.org/)，*Linux Kernel Newbies*，是想要了解和修改 Linux 内核的程序员的起点。

+   [`lxr.linux.no/linux/`](http://lxr.linux.no/linux/)，*Linux Cross-reference*，提供浏览器访问各种版本的 Linux 内核源代码。源文件中的每个标识符都通过超链接，方便查找该标识符的定义和用途。

## 内核源代码

如果前述的资源没有解答我们的问题，或者如果我们想确认文档中的信息是否正确，我们可以阅读内核源代码。尽管部分源代码可能难以理解，但阅读 Linux 内核源代码中某个特定系统调用的代码（或者 GNU C 库源代码中的某个库函数）通常是快速找到问题答案的有效途径。

如果 Linux 内核源代码已经安装在系统上，它通常可以在目录 `/usr/src/linux` 中找到。表 E-1 提供了该目录下部分子目录的概述信息。

表 E-1. Linux 内核源代码中的子目录

| 目录 | 内容 |
| --- | --- |
| `Documentation` | 内核各个方面的文档 |
| `arch` | 特定架构的代码，按子目录组织——例如，`alpha`、`arm`、`ia64`、`sparc` 和 `x86` |
| `drivers` | 设备驱动代码 |
| `fs` | 文件系统相关的代码，按子目录组织——例如，`btrfs`、`ext4`、`proc`（`/proc` 文件系统）和 `vfat` |
| `include` | 内核代码所需的头文件 |
| `init` | 内核的初始化代码 |
| `ipc` | 系统 V IPC 和 POSIX 消息队列的代码 |
| `kernel` | 与进程、程序执行、内核模块、信号、时间和定时器相关的代码 |
| `lib` | 内核各部分使用的通用功能函数 |
| `mm` | 内存管理代码 |
| `net` | 网络代码（TCP/IP、UNIX 和 Internet 域套接字） |
| `scripts` | 配置和构建内核的脚本 |
