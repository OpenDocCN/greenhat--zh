## 第十一章 系统限制和选项

每个 UNIX 实现都对各种系统功能和资源设置了限制，并根据不同的标准提供（或选择不提供）选项。以下是一些示例：

+   一个进程最多可以同时打开多少个文件？

+   系统是否支持实时信号？

+   *int* 类型的变量能存储的最大值是多少？

+   程序能拥有多大的参数列表？

+   路径名的最大长度是多少？

虽然我们可以将假定的限制和选项硬编码到应用程序中，但这样做会减少可移植性，因为限制和选项可能会有所不同：

+   *跨 UNIX 实现*：尽管某些限制和选项在个别实现中可能是固定的，但它们在不同的 UNIX 实现之间可能会有所不同。可以存储在 *int* 类型中的最大值就是这种限制的一个例子。

+   *在特定实现的运行时*：例如，内核可能已经重新配置以更改某个限制。或者，应用程序可能是在一个系统上编译的，但在另一个具有不同限制和选项的系统上运行。

+   *从一个文件系统到另一个文件系统*：例如，传统的 System V 文件系统允许文件名最多为 14 字节，而传统的 BSD 文件系统和大多数本地 Linux 文件系统允许文件名最多为 255 字节。

由于系统限制和选项会影响应用程序的行为，因此便携式应用程序需要有方法来确定限制值以及选项是否受支持。C 编程语言标准和 SUSv3 提供了两种主要的途径供应用程序获取此类信息：

+   一些限制和选项可以在编译时确定。例如，*int* 的最大值由硬件架构和编译器设计选择决定。这类限制可以在头文件中记录。

+   其他限制和选项可能在运行时发生变化。对于这种情况，SUSv3 定义了三个函数——*sysconf()*、*pathconf()* 和 *fpathconf()*——应用程序可以调用它们来检查这些实现限制和选项。

SUSv3 指定了一系列可由符合规范的实现强制执行的限制，以及一组选项，每个选项可能由特定系统提供，也可能不提供。我们将在本章中描述其中的一些限制和选项，并在后续章节的相关部分描述其他选项和限制。

## 系统限制

对于每个它指定的限制，SUSv3 要求所有实现都支持该限制的*最小值*。在大多数情况下，这个最小值会在 `<limits.h>` 中定义为一个常量，名称以字符串 `_POSIX_` 开头，并且（通常）包含字符串 `_MAX`；因此，名称的形式为 `_POSIX_XXX_MAX`。

如果一个应用程序将自己限制在 SUSv3 为每个限制所要求的最小值上，那么它将能够在所有符合标准的实现中移植。然而，这样做会阻止应用程序利用提供更高限制的实现。因此，通常更建议通过`<limits.h>`、*sysconf()*或*pathconf()*来确定特定系统的限制。

### 注意

SUSv3 定义的限制名称中使用字符串`_MAX`可能会让人感到困惑，因为它们被描述为*最小*值。当我们考虑到每个常量定义了某个资源或特性上的上限，并且标准规定这个上限必须具有某个最小值时，名称的理由就变得清晰了。

在某些情况下，*最大值*被提供为某个限制，这些值的名称包含字符串`_MIN`。对于这些常量，情况恰恰相反：它们表示某个资源的下限，标准规定，在符合要求的实现中，这个下限不能大于某个值。例如，`FLT_MIN`限制（`1E-37`）定义了一个实现可能设置的最小浮点数的最大值，所有符合标准的实现都能够表示至少这么小的浮点数。

每个限制都有一个*名称*，该名称对应于上述描述的*最小值名称*，但没有`_POSIX_`前缀。实现*可以*在`<limits.h>`中定义一个常量，表示该实现对应的限制。如果定义了这个常量，那么这个限制将始终至少等于上述描述的最小值（即，`XXX_MAX >= _POSIX_XXX_MAX`）。

SUSv3 将其指定的限制分为三类：*运行时不变值*、*路径名可变值*和*运行时可增加值*。在接下来的段落中，我们将描述这些类别并提供一些示例。

#### 运行时不变值（可能是未确定的）

运行时不变值是一个限制，如果在`<limits.h>`中定义，则其值对于该实现是固定的。然而，值可能是未确定的（可能因为它依赖于可用的内存空间），因此会被省略在`<limits.h>`中。在这种情况下（即使该限制也在`<limits.h>`中定义），应用程序仍然可以使用*sysconf()*在运行时确定该值。

`MQ_PRIO_MAX` 限制是运行时不变值的一个示例。如在 发送消息 中所述，POSIX 消息队列中的消息优先级存在限制。SUSv3 定义了常量 `_POSIX_MQ_PRIO_MAX`，其值为 32，作为所有符合要求的实现必须提供的此限制的最小值。这意味着我们可以确定，所有符合要求的实现都将允许从 0 到至少 31 的消息优先级。UNIX 实现可以设置一个高于此值的限制，在 `<limits.h>` 中定义常量 `MQ_PRIO_MAX`，并给出其限制值。例如，在 Linux 上，`MQ_PRIO_MAX` 的值为 32,768。此值也可以在运行时通过以下调用来确定：

```
lim = sysconf(_SC_MQ_PRIO_MAX);
```

#### 路径名变量值

路径名变量值是与路径名（文件、目录、终端等）相关的限制。每个限制可能对实现是常量，或者可能因文件系统的不同而有所不同。在限制可能因路径名而有所变化的情况下，应用程序可以通过 *pathconf()* 或 *fpathconf()* 来确定其值。

`NAME_MAX` 限制是路径名变量值的一个示例。此限制定义了特定文件系统上文件名的最大大小。SUSv3 定义了常量 `_POSIX_NAME_MAX`，其值为 14（旧的 System V 文件系统限制），作为实现必须允许的最小值。实现可以定义一个高于此值的 `NAME_MAX` 限制，并/或通过以下形式的调用提供有关特定文件系统的信息：

```
lim = pathconf(directory_path, _PC_NAME_MAX)
```

*directory_path* 是感兴趣的文件系统中目录的路径名。

#### 运行时可增加值

运行时可增加值是指对于特定实现具有固定最小值的限制，所有运行该实现的系统都将提供至少此最小值。然而，特定系统可以在运行时增加此限制，应用程序可以使用 *sysconf()* 来查找系统上支持的实际值。

一个运行时可增加值的示例是 `NGROUPS_MAX`，它定义了进程可以同时拥有的最大补充组 ID 数量（补充组 ID）。SUSv3 定义了相应的最小值 `_POSIX_NGROUPS_MAX`，其值为 8。在运行时，应用程序可以通过调用 *sysconf(_SC_NGROUPS_MAX)* 来获取此限制。

#### 选定的 SUSv3 限制总结

表 11-1 列出了与本书相关的部分 SUSv3 定义的限制（其他限制将在后续章节中介绍）。

表 11-1. 选定的 SUSv3 限制

| 限制名称 (`<limits.h>`) | 最小值 | *sysconf()/pathconf()* 名称 (`<unistd.h>`) | 描述 |
| --- | --- | --- | --- |
| `ARG_MAX` | `4096` | `_SC_ARG_MAX` | 可传递给*exec()*的参数（*argv*）加上环境（*environ*）的最大字节数（环境列表和将调用者的环境传递给新程序） |
| none | none | `_SC_CLK_TCK` | *times()* 的计量单位 |
| `LOGIN_NAME_MAX` | `9` | `_SC_LOGIN_NAME_MAX` | 登录名的最大长度（包括终止的空字节） |
| `OPEN_MAX` | `20` | `_SC_OPEN_MAX` | 进程一次可以打开的文件描述符的最大数量，比最大可用描述符数量多一个（进程资源限制） |
| `NGROUPS_MAX` | `8` | `_SC_NGROUPS_MAX` | 进程可以作为成员的最大补充组数（检索和修改补充组 ID） |
| none | `1` | `_SC_PAGESIZE` | 虚拟内存页面的大小（`_SC_PAGE_SIZE`是其同义词） |
| `RTSIG_MAX` | `8` | `_SC_RTSIG_MAX` | 最大的不同实时信号数（实时信号） |
| `SIGQUEUE_MAX` | `32` | `_SC_SIGQUEUE_MAX` | 最大排队的实时信号数（实时信号） |
| `STREAM_MAX` | `8` | `_SC_STREAM_MAX` | 一次可以打开的*stdio*流的最大数量 |
| `NAME_MAX` | `14` | `_PC_NAME_MAX` | 文件名的最大字节数，*不包括*终止空字节 |
| `PATH_MAX` | `256` | `_PC_PATH_MAX` | 路径名中的最大字节数，*包括*终止空字节 |
| `PIPE_BUF` | `512` | `_PC_PIPE_BUF` | 可以原子写入管道或 FIFO 的最大字节数（概述） |

表 11-1 的第一列给出了限制的名称，可以在`<limits.h>`中定义为常量，以表示特定实现的限制。第二列是 SUSv3 定义的该限制的最小值（也在`<limits.h>`中定义）。在大多数情况下，每个最小值都被定义为一个以`_POSIX_`为前缀的常量。例如，常量`_POSIX_RTSIG_MAX`（定义为值 8）指定了 SUSv3 所要求的最小值，对应于`RTSIG_MAX`实现常量。第三列指定了可以在运行时传递给*sysconf()*或*pathconf()*以检索实现限制的常量名称。以`_SC_`开头的常量用于*sysconf()*；以`_PC_`开头的常量用于*pathconf()*和*fpathconf()*。

请注意以下信息，作为对 表 11-1 中所示内容的补充：

+   *getdtablesize()* 函数是一个过时的替代方法，用于确定进程文件描述符的限制（`OPEN_MAX`）。该函数在 SUSv2 中被指定（标记为 LEGACY），但在 SUSv3 中被移除。

+   *getpagesize()* 函数是一个过时的替代方法，用于确定系统页面大小（`_SC_PAGESIZE`）。该函数在 SUSv2 中被指定（标记为 LEGACY），但在 SUSv3 中被移除。

+   常量 `FOPEN_MAX`，在 `<stdio.h>` 中定义，与 `STREAM_MAX` 同义。

+   `NAME_MAX` 不包括终止的空字节，而 `PATH_MAX` 包括它。这个不一致修复了 POSIX.1 标准中的早期不一致，该标准未明确说明 `PATH_MAX` 是否包含终止的空字节。将 `PATH_MAX` 定义为包括终止符意味着，分配了正好 `PATH_MAX` 字节作为路径名的应用程序仍然符合标准。

#### 从 shell 确定限制和选项：*getconf*

从 shell 中，我们可以使用 *getconf* 命令来获取特定 UNIX 实现所实现的限制和选项。此命令的一般形式如下：

```
$ `getconf` ``*`variable-name`*`` [ ``*`pathname`*`` ]
```

*variable-name* 标识了感兴趣的限制，它是 SUSV3 标准限制名称之一，如 `ARG_MAX` 或 `NAME_MAX`。当限制与路径名相关时，我们必须指定路径名作为命令的第二个参数，如下列第二个示例所示。

```
$ `getconf ARG_MAX`
131072
$ `getconf NAME_MAX /boot`
255
```

## 在运行时检索系统限制（和选项）

*sysconf()* 函数允许应用程序在运行时获取系统限制的值。

```
#include <unistd.h>

long `sysconf`(int *name*);
```

### 注意

返回由 *name* 指定的限制值，如果限制无法确定或发生错误，则返回 -1。

*name* 参数是 `<unistd.h>` 中定义的 `_SC_*` 常量之一，其中一些常量列在 表 11-1 中。限制的值作为函数结果返回。

如果无法确定某个限制，*sysconf()* 返回 -1。如果发生错误，它也可能返回 -1。（唯一指定的错误是 `EINVAL`，表示 *name* 无效。）为了区分限制不确定的情况和错误，我们必须在调用之前将 *errno* 设置为 0；如果调用返回 -1 且调用后 *errno* 被设置，则发生了错误。

### 注意

*sysconf()* 返回的限制值（以及 *pathconf()* 和 *fpathconf()*）始终是（*long*）整数。在 *sysconf()* 的原理说明中，SUSv3 提到字符串曾被考虑作为可能的返回值，但由于实现和使用的复杂性而被拒绝。

示例 11-1") 演示了使用 *sysconf()* 来显示各种系统限制。在一台 Linux 2.6.31/x86-32 系统上运行此程序的输出如下：

```
$ `./t_sysconf`
_SC_ARG_MAX:         2097152
_SC_LOGIN_NAME_MAX:  256
_SC_OPEN_MAX:        1024
_SC_NGROUPS_MAX:     65536
_SC_PAGESIZE:        4096
_SC_RTSIG_MAX:       32
```

示例 11-1. 使用 *sysconf()*

```
`syslim/t_sysconf.c`
#include "tlpi_hdr.h"

static void             /* Print 'msg' plus sysconf() value for 'name' */
sysconfPrint(const char *msg, int name)
{
    long lim;

    errno = 0;
    lim = sysconf(name);
    if (lim != -1) {        /* Call succeeded, limit determinate */
        printf("%s %ld\n", msg, lim);
    } else {
        if (errno == 0)     /* Call succeeded, limit indeterminate */
            printf("%s (indeterminate)\n", msg);
        else                /* Call failed */
            errExit("sysconf %s", msg);
    }
}

int
main(int argc, char *argv[])
{
    sysconfPrint("_SC_ARG_MAX:        ", _SC_ARG_MAX);
    sysconfPrint("_SC_LOGIN_NAME_MAX: ", _SC_LOGIN_NAME_MAX);
    sysconfPrint("_SC_OPEN_MAX:       ", _SC_OPEN_MAX);
    sysconfPrint("_SC_NGROUPS_MAX:    ", _SC_NGROUPS_MAX);
    sysconfPrint("_SC_PAGESIZE:       ", _SC_PAGESIZE);
    sysconfPrint("_SC_RTSIG_MAX:      ", _SC_RTSIG_MAX);
    exit(EXIT_SUCCESS);
}
      `syslim/t_sysconf.c`
```

SUSv3 要求 *sysconf()* 返回的特定限制值在调用进程的生命周期内保持不变。例如，我们可以假设在进程运行期间，返回的 `_SC_PAGESIZE` 值不会发生变化。

### 注意

在 Linux 上，对于限制值在进程生命周期内应保持不变的说法，存在一些（合理的）例外。进程可以使用 *setrlimit()*（进程资源限制）来更改影响 *sysconf()* 返回的限制值的各种进程资源限制：`RLIMIT_NOFILE`，决定进程可以打开的文件数量（`_SC_OPEN_MAX`）；`RLIMIT_NPROC`（SUSv3 中未实际指定的资源限制），即此进程可以创建的最大子进程数（`_SC_CHILD_MAX`）；以及 `RLIMIT_STACK`，自 Linux 2.6.23 起，决定允许进程的命令行参数和环境变量的空间限制（`_SC_ARG_MAX`；有关详细信息，请参见 *execve(2)* 手册页）。

## 运行时获取与文件相关的限制（和选项）

*pathconf()* 和 *fpathconf()* 函数允许应用程序在运行时获取与文件相关的限制值。

```
#include <unistd.h>

long `pathconf`(const char **pathname*, int *name*);
long `fpathconf`(int *fd*, int *name*);
```

### 注意

两者返回由 *name* 指定的限制值，若限制无法确定或发生错误，则返回 -1。

*pathconf()* 和 *fpathconf()* 之间的唯一区别在于指定文件或目录的方式。对于 *pathconf()*，通过路径名指定；对于 *fpathconf()*，通过（已打开的）文件描述符指定。

`*name*` 参数是 `<unistd.h>` 中定义的 `_PC_*` 常量之一，其中一些常量列在表 11-1 中。表 11-2 _PC_* 名称的详细信息")提供了有关在表 11-1 中显示的 `_PC_*` 常量的更多细节。

限制值作为函数结果返回。我们可以像对待 *sysconf()* 一样区分不确定返回和错误返回。

与 *sysconf()* 不同，SUSv3 不要求 *pathconf()* 和 *fpathconf()* 返回的值在进程生命周期内保持不变，因为例如，文件系统可能在进程运行时被卸载并重新挂载，且具有不同的特性。

表 11-2. 选定的 *pathconf()* `_PC_*` 名称的详细信息

| 常量 | 注释 |
| --- | --- |
| `_PC_NAME_MAX` | 对于目录，此值表示目录中文件的最大数量。其他文件类型的行为未指定。 |
| `_PC_PATH_MAX` | 对于目录，此值表示从该目录到相对路径名的最大长度。其他文件类型的行为未指定。 |
| `_PC_PIPE_BUF` | 对于 FIFO 或管道，它会返回一个适用于被引用文件的值。对于目录，该值适用于在该目录中创建的 FIFO。其他文件类型的行为未指定。 |

示例 11-2")展示了使用*fpathconf()*来检索通过其标准输入引用的文件的各种限制。当我们运行该程序并指定标准输入为*ext2*文件系统上的一个目录时，我们看到如下输出：

```
$ `./t_fpathconf < .`
_PC_NAME_MAX:  255
_PC_PATH_MAX:  4096
_PC_PIPE_BUF:  4096
```

示例 11-2. 使用*fpathconf()*

```
`syslim/t_fpathconf.c`
#include "tlpi_hdr.h"

static void             /* Print 'msg' plus value of fpathconf(fd, name) */
fpathconfPrint(const char *msg, int fd, int name)
{
    long lim;

    errno = 0;
    lim = fpathconf(fd, name);
    if (lim != -1) {        /* Call succeeded, limit determinate */
        printf("%s %ld\n", msg, lim);
    } else {
        if (errno == 0)     /* Call succeeded, limit indeterminate */
            printf("%s (indeterminate)\n", msg);
        else                /* Call failed */
            errExit("fpathconf %s", msg);
    }
}

int
main(int argc, char *argv[])
{
    fpathconfPrint("_PC_NAME_MAX: ", STDIN_FILENO, _PC_NAME_MAX);
    fpathconfPrint("_PC_PATH_MAX: ", STDIN_FILENO, _PC_PATH_MAX);
    fpathconfPrint("_PC_PIPE_BUF: ", STDIN_FILENO, _PC_PIPE_BUF);
    exit(EXIT_SUCCESS);
}
     `syslim/t_fpathconf.c`
```

## 不确定的限制

有时，我们可能会发现某些系统限制没有由实现限制常量（例如，`PATH_MAX`）定义，并且*sysconf()*或*pathconf()*告诉我们该限制（例如，`_PC_PATH_MAX`）是不可确定的。在这种情况下，我们可以采用以下策略之一：

+   在编写一个可移植的应用程序以适应多个 UNIX 实现时，我们可以选择使用 SUSv3 中指定的最小限制值。这些常量的名称形式为`_POSIX_*_MAX`，在第 11.1 节中有描述。有时，这种方法可能不可行，因为限制值不切实际地低，像是`_POSIX_PATH_MAX`和`_POSIX_OPEN_MAX`的情况。

+   在某些情况下，实际的解决方案可能是忽略限制的检查，而是直接执行相关的系统或库函数调用。（类似的论点也适用于第 11.5 节中描述的一些 SUSv3 选项。）如果调用失败并且*errno*表明错误发生是由于某个系统限制被超出，那么我们可以重试，根据需要修改应用程序的行为。例如，大多数 UNIX 实现对可以排队到进程的实时信号数有限制。一旦达到了此限制，尝试发送更多信号（使用*sigqueue()*）将失败，并显示错误`EAGAIN`。在这种情况下，发送信号的进程可以简单地重试，可能是在某个延迟间隔后。同样，尝试打开一个文件名过长的文件将产生错误`ENAMETOOLONG`，应用程序可以通过尝试使用较短的名称来处理这种情况。

+   我们可以编写自己的程序或函数来推断或估算该限制。在每种情况下，都会调用相关的*sysconf()*或*pathconf()*，如果该限制是不可确定的，函数将返回一个“合理的猜测”值。虽然这种解决方案并不完美，但在实践中通常是可行的。

+   我们可以使用像 GNU *Autoconf*这样的工具，它是一个可扩展的工具，可以确定各种系统特性和限制的存在及设置。Autoconf 程序基于它确定的信息生成头文件，这些文件随后可以包含在 C 程序中。有关 Autoconf 的更多信息可以在[`www.gnu.org/software/autoconf/`](http://www.gnu.org/software/autoconf/)找到。

## 系统选项

除了为各种系统资源指定限制外，SUSv3 还指定了 UNIX 实现可能支持的各种选项。这些选项包括对实时信号、POSIX 共享内存、作业控制和 POSIX 线程等特性的支持。除少数例外外，实施者不需要支持这些选项。相反，SUSv3 允许实施者在编译时和运行时指示是否支持特定特性。

实现可以通过在`<unistd.h>`中定义相应的常量，在编译时声明对特定 SUSv3 选项的支持。每个这样的常量都以指示其来源标准的前缀开始（例如，`_POSIX_` 或 `_XOPEN_`）。

如果定义了每个选项常量，它们的值将是以下之一：

+   值为 -1 意味着*该选项不受支持*。在这种情况下，实施者不需要定义与该选项相关的头文件、数据类型和函数接口。我们可能需要通过使用`#if`预处理指令来处理这种可能性。

+   值为 0 意味着*该选项可能受支持*。应用程序必须在运行时检查该选项是否受支持。

+   大于 0 的值意味着*该选项受支持*。与该选项相关的所有头文件、数据类型和函数接口都已定义并按指定方式运行。在许多情况下，SUSv3 要求该正值为 `200112L`，这是与 SUSv3 被批准为标准的年份和月份号对应的常量。（SUSv4 对应的值为 `200809L`。）

如果常量的值为 0，应用程序可以使用*sysconf()*和*pathconf()*（或*fpathconf()*）函数在运行时检查该选项是否受支持。传递给这些函数的*name*参数通常与相应的编译时常量相同，但前缀替换为`_SC_`或`_PC_`。实现必须至少提供必要的头文件、常量和函数接口，以便执行运行时检查。

### 注意

SUSv3 对于未定义的选项常量是否与定义常量为值 0（“该选项可能受支持”）或值 -1（“该选项不受支持”）有相同的含义并不明确。标准委员会随后决定该情况应表示与定义常量为值 -1 相同，SUSv4 明确声明了这一点。

表 11-3 列出了 SUSv3 中指定的一些选项。表格的第一列给出了与选项关联的编译时常量名称（在`<unistd.h>`中定义），以及相应的*sysconf()*（`_SC_*`）或*pathconf()*（`_PC_*`）*name*参数。请注意以下有关特定选项的要点：

+   某些选项是 SUSv3 要求的；也就是说，编译时常量始终评估为大于 0 的值。从历史上看，这些选项曾经是可选的，但现在它们不是了。这些选项在*备注*列中以字符`+`标记。（在 SUSv4 中，SUSv3 中可选的多个选项变为强制性选项。）

    ### 注意

    尽管这些选项是 SUSv3 要求的，但某些 UNIX 系统可能仍然以不符合规范的配置安装。因此，对于便携式应用程序，检查影响应用程序的选项是否受支持可能是值得的，无论标准是否要求该选项。

+   对于某些选项，编译时常量必须具有非-1 的值。换句话说，选项必须得到支持，或者必须能够在运行时检查是否支持这些选项。这些选项在*备注*列中以字符`*`标记。

表 11-3. 选定的 SUSv3 选项

| 选项（常量）名称 (*sysconf()* / *pathconf()* 名称) | 描述 | 备注 |
| --- | --- | --- |
| `_POSIX_ASYNCHRONOUS_IO (_SC_ASYNCHRONOUS_IO)` | *异步 I/O* |   |
| `_POSIX_CHOWN_RESTRICTED (_PC_CHOWN_RESTRICTED)` | 只有特权进程才能使用*chown()*和*fchown()*将文件的用户 ID 和组 ID 更改为任意值 (更改文件所有权：*chown()*, *fchown()* 和 *lchown()*, fchown(), and lchown()")) | `*` |
| `_POSIX_JOB_CONTROL (_SC_JOB_CONTROL)` | *作业控制* (作业控制) | `+` |
| `_POSIX_MESSAGE_PASSING (_SC_MESSAGE_PASSING)` | *POSIX 消息队列* (第五十二章) |   |
| `_POSIX_PRIORITY_SCHEDULING (_SC_PRIORITY_SCHEDULING)` | *进程调度* (实时进程调度 API) |   |
| `_POSIX_REALTIME_SIGNALS (_SC_REALTIME_SIGNALS)` | *实时信号扩展* (实时信号) |   |
| `_POSIX_SAVED_IDS` (无) | 进程具有已保存的用户 ID 和已保存的组 ID (已保存的用户 ID 和已保存的组 ID) | `+` |
| `_POSIX_SEMAPHORES (_SC_SEMAPHORES)` | *POSIX 信号量* (第五十三章) |   |
| `_POSIX_SHARED_MEMORY_OBJECTS (_SC_SHARED_MEMORY_OBJECTS)` | *POSIX 共享内存对象* (第五十四章) |   |
| `_POSIX_THREADS (_SC_THREADS)` | *POSIX 线程* |   |
| `_XOPEN_UNIX` (`_SC_XOPEN_UNIX`) | 支持 XSI 扩展 (SUSv3 和 POSIX.1-2001) |   |

## 摘要

SUSv3 规定了实现可能强制执行的限制以及实现可能支持的系统选项。

通常，不希望将关于系统限制和选项的假设硬编码到程序中，因为这些假设在不同的实现中可能会有所不同，并且在同一个实现中，也可能在运行时或不同的文件系统之间有所变化。因此，SUSv3 规定了一些方法，通过这些方法，某个实现可以公开它所支持的限制和选项。对于大多数限制，SUSv3 规定了一个所有实现必须支持的最小值。此外，每个实现可以在编译时（通过在`<limits.h>`或`<unistd.h>`中定义常量）和/或运行时（通过调用*sysconf()*、*pathconf()*或*fpathconf()*）公开其特定实现的限制和选项。这些技术也可以类似地用于查找某个实现支持的 SUSv3 选项。在某些情况下，可能无法通过这两种方法之一确定特定的限制。对于这些无法确定的限制，我们必须 resort to ad hoc 技术来确定应用程序应该遵守的限制。

#### 更多信息

第二章来自[Stevens & Rago, 2005]和第二章来自[Gallmeister, 1995]涵盖的内容与本章类似。[Lewine, 1991]也提供了许多有用的（尽管现在略显过时的）背景信息。有关 POSIX 选项以及关于*glibc*和 Linux 细节的一些信息可以在[`people.redhat.com/drepper/posix-option-groups.html`](http://people.redhat.com/drepper/posix-option-groups.html)找到。以下 Linux 手册页也与此相关：*sysconf(3)*、*pathconf(3)*、*feature_test_macros(7)*、*posixoptions(7)*和*standards(7)*。

最好的信息来源（尽管有时难以阅读）是 SUSv3 的相关部分，特别是来自基本定义（XBD）的第二章，以及*<unistd.h>*、*<limits.h>*、*sysconf()*和*fpathconf()*的规范。[Josey, 2004]提供了关于使用 SUSv3 的指导。

## 练习

1.  如果可以访问其他 UNIX 实现，尝试在其他 UNIX 实现上运行示例 11-1")中的程序。

1.  尝试在其他文件系统上运行示例 11-2")中的程序。
