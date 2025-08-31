## 第四十六章 系统 V 消息队列

本章描述了 System V 消息队列。消息队列允许进程以消息的形式交换数据。尽管消息队列在某些方面与管道和 FIFO 相似，但它们在许多重要方面也有所不同：

+   用于引用消息队列的句柄是通过调用*msgget()*返回的标识符。这些标识符与大多数 UNIX 系统中用于其他 I/O 形式的文件描述符不同。

+   通过消息队列进行通信是面向消息的；也就是说，读取者接收到完整的消息，正如写入者所写的那样。不能读取消息的一部分并将其余部分留在队列中，也不能一次读取多条消息。这与管道不同，管道提供的是不区分字节流（即，使用管道时，读取者可以一次读取任意数量的字节，无论写入者写入的数据块的大小）。

+   每条消息除了包含数据外，还有一个整数*type*。消息可以按照先进先出顺序从队列中获取，或者按类型获取。

本章的最后部分（System V 消息队列的缺点）总结了 System V 消息队列的若干局限性。这些局限性使我们得出结论：在可能的情况下，新的应用程序应避免使用 System V 消息队列，转而选择其他 IPC 机制，如 FIFO、POSIX 消息队列和套接字。然而，当初设计消息队列时，这些替代机制尚不可用或在 UNIX 实现中并不广泛。因此，仍有许多现有应用程序使用消息队列，这也是我们描述它们的主要动机之一。

## 创建或打开消息队列

*msgget()*系统调用用于创建一个新的消息队列或获取现有队列的标识符。

```
#include <sys/types.h>        /* For portability */
#include <sys/msg.h>

int `msgget`(key_t *key*, int *msgflg*);
```

### 注意

成功时返回消息队列标识符，出错时返回 -1

*key*参数是通过 IPC 键（即通常是值`IPC_PRIVATE`或*ftok()*返回的键）生成的键。*msgflg*参数是一个位掩码，指定新消息队列的权限（表 15-4，在常规文件权限中）。此外，可以在*msgflg*中按位或(|)一个或多个以下标志，以控制*msgget()*的操作：

`IPC_CREAT`

如果不存在使用指定*key*的消息队列，则创建一个新队列。

`IPC_EXCL`

如果还指定了`IPC_CREAT`，并且使用指定的*key*的队列已经存在，则返回错误`EEXIST`。

这些标志在第 45.1 节中有更详细的描述。

*msgget()* 系统调用首先会在所有现有的消息队列中搜索具有指定键的队列。如果找到匹配的队列，则返回该队列的标识符（除非在 *msgflg* 中同时指定了 `IPC_CREAT` 和 `IPC_EXCL`，此时将返回错误）。如果未找到匹配的队列并且在 *msgflg* 中指定了 `IPC_CREAT`，则会创建一个新队列并返回其标识符。

示例 46-1") 中的程序提供了一个命令行接口，用于调用 *msgget()* 系统调用。该程序允许使用命令行选项和参数来指定 *key* 和 *msgflg* 参数的所有可能值。该程序接受的命令格式的详细信息显示在 *usageError()* 函数中。成功创建队列后，程序会打印队列标识符。我们在接收消息中演示了该程序的使用。

示例 46-1. 使用 *msgget()*

```
`svmsg/svmsg_create.c`
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
#include <sys/stat.h>
#include "tlpi_hdr.h"

static void             /* Print usage info, then exit */
usageError(const char *progName, const char *msg)
{
    if (msg != NULL)
        fprintf(stderr, "%s", msg);
    fprintf(stderr, "Usage: %s [-cx] {-f pathname | -k key | -p} "
                            "[octal-perms]\n", progName);
    fprintf(stderr, "    -c           Use IPC_CREAT flag\n");
    fprintf(stderr, "    -x           Use IPC_EXCL flag\n");
    fprintf(stderr, "    -f pathname  Generate key using ftok()\n");
    fprintf(stderr, "    -k key       Use 'key' as key\n");
    fprintf(stderr, "    -p           Use IPC_PRIVATE key\n");
    exit(EXIT_FAILURE);
}

int
main(int argc, char *argv[])
{
    int numKeyFlags;            /* Counts -f, -k, and -p options */
    int flags, msqid, opt;
    unsigned int perms;
    long lkey;
    key_t key;

    /* Parse command-line options and arguments */

    numKeyFlags = 0;
    flags = 0;

    while ((opt = getopt(argc, argv, "cf:k:px")) != -1) {
        switch (opt) {
        case 'c':
            flags |= IPC_CREAT;
            break;

        case 'f':               /* -f pathname */
            key = ftok(optarg, 1);
            if (key == -1)
                errExit("ftok");
            numKeyFlags++;
            break;

        case 'k':               /* -k key (octal, decimal or hexadecimal) */
            if (sscanf(optarg, "%li", &lkey) != 1)
                cmdLineErr("-k option requires a numeric argument\n");
            key = lkey;
            numKeyFlags++;
            break;

        case 'p':
            key = IPC_PRIVATE;
            numKeyFlags++;
            break;

        case 'x':
            flags |= IPC_EXCL;
            break;

        default:
            usageError(argv[0], "Bad option\n");
        }
    }

    if (numKeyFlags != 1)
        usageError(argv[0], "Exactly one of the options -f, -k, "
                            "or -p must be supplied\n");

    perms = (optind == argc) ? (S_IRUSR | S_IWUSR) :
                getInt(argv[optind], GN_BASE_8, "octal-perms");

    msqid = msgget(key, flags | perms);
    if (msqid == -1)
        errExit("msgget");

    printf("%d\n", msqid);
    exit(EXIT_SUCCESS);
}
      `svmsg/svmsg_create.c`
```

## 交换消息

*msgsnd()* 和 *msgrcv()* 系统调用执行消息队列上的 I/O 操作。两个系统调用的第一个参数 (*msqid*) 是消息队列的标识符。第二个参数，*msgp*，是指向程序员定义的结构体的指针，该结构体用于保存正在发送或接收的消息。该结构体的通用形式如下：

```
struct mymsg {
    long mtype;                 /* Message type */
    char mtext[];               /* Message body */
}
```

这个定义实际上只是简写，表示消息的第一部分包含消息类型，类型为长整型，而消息的其余部分是程序员定义的任意长度和内容的结构体；它不一定是字符数组。因此，*mgsp* 参数被类型化为 *void **，允许它指向任何类型的结构体。

允许零长度的 *mtext* 字段，如果要传递的信息仅能通过消息类型编码，或者消息的存在本身就是接收进程所需的信息，零长度的 *mtext* 字段有时会很有用。

### 发送消息

*msgsnd()* 系统调用将消息写入消息队列。

```
#include <sys/types.h>        /* For portability */
#include <sys/msg.h>

int `msgsnd`(int *msqid*, const void **msgp*, size_t *msgsz*, int *msgflg*);
```

### 注意

成功时返回 0，出错时返回 -1

要使用 *msgsnd()* 发送消息，必须将消息结构体的 *mtype* 字段设置为大于 0 的值（我们将在下一节讨论 *msgrcv()* 时看到该值的使用），并将所需的信息复制到程序员定义的 *mtext* 字段中。*msgsz* 参数指定 *mtext* 字段中包含的字节数。

### 注意

使用 *msgsnd()* 发送消息时，不存在像 *write()* 那样的部分写入概念。这就是为什么成功的 *msgsnd()* 只需要返回 0，而不是发送的字节数。

最后的参数 *msgflg* 是一个位掩码，控制 *msgsnd()* 的操作。定义了一个这样的标志：

`IPC_NOWAIT`

执行非阻塞发送。通常，如果消息队列已满，*msgsnd()*会阻塞，直到有足够的空间可以将消息放入队列中。然而，如果指定了此标志，则*msgsnd()*会立即返回错误`EAGAIN`。

如果由于队列已满而阻塞的*msgsnd()*调用可能会被信号处理程序中断。在这种情况下，*msgsnd()*将始终因错误`EINTR`而失败。（如在系统调用的中断与重启中所述，*msgsnd()*是那些无论在设置`SA_RESTART`标志时如何，都不会自动重启的系统调用之一。）

向消息队列写入消息需要具有队列的写权限。

示例 46-2 发送消息")中的程序为*msgsnd()*系统调用提供了一个命令行界面。该程序接受的命令行格式如*usageError()*函数中所示。请注意，这个程序并没有使用*msgget()*系统调用。（我们在第 45.1 节中提到，进程不需要使用*get*调用就能访问 IPC 对象。）相反，我们通过提供其标识符作为命令行参数来指定消息队列。我们在接收消息中演示了此程序的使用。

示例 46-2. 使用*msgsnd()*发送消息

```
`svmsg/svmsg_send.c`
#include <sys/types.h>
#include <sys/msg.h>
#include "tlpi_hdr.h"

#define MAX_MTEXT 1024

struct mbuf {
    long mtype;                         /* Message type */
    char mtext[MAX_MTEXT];              /* Message body */
};

static void             /* Print (optional) message, then usage description */
usageError(const char *progName, const char *msg)
{
    if (msg != NULL)
        fprintf(stderr, "%s", msg);
    fprintf(stderr, "Usage: %s [-n] msqid msg-type [msg-text]\n", progName);
    fprintf(stderr, "    -n       Use IPC_NOWAIT flag\n");
    exit(EXIT_FAILURE);
}

int
main(int argc, char *argv[])
{
    int msqid, flags, msgLen;
    struct mbuf msg;                    /* Message buffer for msgsnd() */
    int opt;                            /* Option character from getopt() */

    /* Parse command-line options and arguments */

    flags = 0;
    while ((opt = getopt(argc, argv, "n")) != -1) {
        if (opt == 'n')
            flags |= IPC_NOWAIT;
        else
            usageError(argv[0], NULL);
    }

    if (argc < optind + 2 || argc > optind + 3)
        usageError(argv[0], "Wrong number of arguments\n");

    msqid = getInt(argv[optind], 0, "msqid");
    msg.mtype = getInt(argv[optind + 1], 0, "msg-type");

    if (argc > optind + 2) {            /* 'msg-text' was supplied */
        msgLen = strlen(argv[optind + 2]) + 1;
        if (msgLen > MAX_MTEXT)
            cmdLineErr("msg-text too long (max: %d characters)\n", MAX_MTEXT);

        memcpy(msg.mtext, argv[optind + 2], msgLen);

    } else {                            /* No 'msg-text' ==> zero-length msg */
        msgLen = 0;
    }

    /* Send message */

    if (msgsnd(msqid, &msg, msgLen, flags) == -1)
        errExit("msgsnd");

    exit(EXIT_SUCCESS);
}
      `svmsg/svmsg_send.c`
```

### 接收消息

*msgrcv()*系统调用从消息队列中读取（并移除）一条消息，并将其内容复制到由*msgp*指向的缓冲区中。

```
#include <sys/types.h>        /* For portability */
#include <sys/msg.h>

ssize_t `msgrcv`(int *msqid*, void **msgp*, size_t *maxmsgsz*, long
 *msgtyp*, int *msgflg*);
```

### 注意

返回复制到*mtext*字段的字节数，若出错则返回-1

*msgp*缓冲区中*mtext*字段可用的最大空间由参数*maxmsgsz*指定。如果要从队列中移除的消息体超过*maxmsgsz*字节，则不会从队列中移除消息，且*msgrcv()*将因错误`E2BIG`而失败。（这个默认行为可以通过稍后描述的`MSG_NOERROR`标志进行更改。）

消息不必按发送的顺序读取。相反，我们可以根据*mtype*字段中的值选择消息。此选择由*msgtyp*参数控制，如下所示：

+   如果*msgtyp*等于 0，则从队列中移除并返回第一个消息给调用进程。

+   如果*msgtyp*大于 0，则从队列中移除并返回第一个*mtype*等于*msgtyp*的消息给调用进程。通过为*msgtyp*指定不同的值，多个进程可以从消息队列中读取，而不必竞争读取相同的消息。一种有用的技术是让每个进程选择与其进程 ID 匹配的消息。

+   如果*msgtyp*小于 0，则将待处理消息视为*优先队列*。首先移除并返回绝对值小于或等于*msgtyp*的最低*mtype*的消息。

一个示例有助于阐明当*msgtyp*小于 0 时的行为。假设我们有一个包含图 46-1 所示消息序列的消息队列，并且我们执行一系列如下形式的*msgrcv()*调用：

```
msgrcv(id, &msg, maxmsgsz, -300, 0);
```

这些*msgrcv()*调用将按顺序检索消息：2（类型 100）、5（类型 100）、3（类型 200）和 1（类型 300）。进一步的调用将会阻塞，因为剩余消息的类型（400）超过了 300。

*msgflg*参数是一个位掩码，通过按位或组合以下标志中的一个或多个形成：

`IPC_NOWAIT`

执行非阻塞接收。通常，如果队列中没有与*msgtyp*匹配的消息，*msgrcv()*将阻塞，直到有匹配的消息可用。指定`IPC_NOWAIT`标志会使*msgrcv()*立即返回，并显示错误`ENOMSG`。（错误`EAGAIN`会更为一致，就像在非阻塞*msgsnd()*或非阻塞从 FIFO 读取时发生的那样。然而，返回`ENOMSG`是历史行为，并且 SUSv3 要求如此。）

`MSG_EXCEPT`

该标志仅在*msgtyp*大于 0 时有效，在这种情况下，它强制执行常规操作的补充操作；也就是说，队列中第一个*mtype*不等于*msgtyp*的消息将从队列中移除并返回给调用者。此标志是 Linux 特有的，只有在定义了`_GNU_SOURCE`时才会从`<sys/msg.h>`中提供。对图 46-1 所示的消息队列执行一系列形式为*msgrcv(id, &msg, maxmsgsz, 100, MSG_EXCEPT)*的调用，将按顺序检索消息：1、3、4，然后阻塞。

`MSG_NOERROR`

默认情况下，如果消息的*mtext*字段的大小超过了可用空间（由*maxmsgsz*参数定义），*msgrcv()*将失败。如果指定了`MSG_NOERROR`标志，则*msgrcv()*将从队列中移除该消息，将其*mtext*字段截断为*maxmsgsz*字节，并返回给调用者。截断的数据将丢失。

成功完成后，*msgrcv()*返回接收到的消息的*mtext*字段的大小；如果发生错误，则返回-1。

![不同类型消息的消息队列示例](img/46-1_SVMSG-queue-example-scale90.png.jpg)图 46-1. 包含不同类型消息的消息队列示例

与*msgsnd()*一样，如果被信号处理程序中断的阻塞*msgrcv()*调用发生，调用将失败并返回错误`EINTR`，无论在设置信号处理程序时`SA_RESTART`标志的设置如何。

从消息队列中读取消息需要对队列具有读取权限。

#### 示例程序

示例 46-3 读取消息")中的程序提供了一个命令行接口，用于调用*msgrcv()*系统调用。该程序接受的命令行格式显示在*usageError()*函数中。与示例 46-2 发送消息")中的程序一样，该程序演示了如何使用*msgsnd()*，但没有使用*msgget()*系统调用，而是期望将消息队列标识符作为命令行参数。

以下 shell 会话演示了示例 46-1")、示例 46-2 发送消息")和示例 46-3 读取消息")程序的使用。我们首先使用`IPC_PRIVATE`键创建一个消息队列，然后将三条不同类型的消息写入队列：

```
$ `./svmsg_create -p`
32769                                               *ID of message queue*
$ `./svmsg_send 32769 20 "I hear and I forget."`
$ `./svmsg_send 32769 10 "I see and I remember."`
$ `./svmsg_send 32769 30 "I do and I understand."`
```

然后，我们使用示例 46-3 读取消息")中的程序从队列中读取类型小于或等于 20 的消息：

```
$ `./svmsg_receive -t -20 32769`
Received: type=10; length=22; body=I see and I remember.
$ `./svmsg_receive -t -20 32769`
Received: type=20; length=21; body=I hear and I forget.
$ `./svmsg_receive -t -20 32769`
```

上述命令中的最后一个被阻塞，因为队列中没有类型小于或等于 20 的消息。所以，我们继续通过键入*Control-C*来终止命令，然后执行一个从队列中读取任何类型消息的命令：

```
*Type Control-C to terminate program*
$ `./svmsg_receive 32769`
Received: type=30; length=23; body=I do and I understand.
```

示例 46-3. 使用*msgrcv()*读取消息

```
`svmsg/svmsg_receive.c`
#define _GNU_SOURCE     /* Get definition of MSG_EXCEPT */
#include <sys/types.h>
#include <sys/msg.h>
#include "tlpi_hdr.h"

#define MAX_MTEXT 1024

struct mbuf {
    long mtype;                 /* Message type */
    char mtext[MAX_MTEXT];      /* Message body */
};

static void
usageError(const char *progName, const char *msg)
{
    if (msg != NULL)
        fprintf(stderr, "%s", msg);
    fprintf(stderr, "Usage: %s [options] msqid [max-bytes]\n", progName);
    fprintf(stderr, "Permitted options are:\n");
    fprintf(stderr, "    -e       Use MSG_NOERROR flag\n");
    fprintf(stderr, "    -t type  Select message of given type\n");
    fprintf(stderr, "    -n       Use IPC_NOWAIT flag\n");
#ifdef MSG_EXCEPT
    fprintf(stderr, "    -x       Use MSG_EXCEPT flag\n");
#endif
    exit(EXIT_FAILURE);
}

int
main(int argc, char *argv[])
{
    int msqid, flags, type;
    ssize_t msgLen;
    size_t maxBytes;
    struct mbuf msg;            /* Message buffer for msgrcv() */
    int opt;                    /* Option character from getopt() */

    /* Parse command-line options and arguments */

    flags = 0;
    type = 0;
    while ((opt = getopt(argc, argv, "ent:x")) != -1) {
        switch (opt) {
        case 'e':       flags |= MSG_NOERROR;   break;
        case 'n':       flags |= IPC_NOWAIT;    break;
        case 't':       type = atoi(optarg);    break;
#ifdef MSG_EXCEPT
        case 'x':       flags |= MSG_EXCEPT;    break;
#endif
        default:        usageError(argv[0], NULL);
        }
    }

    if (argc < optind + 1 || argc > optind + 2)
        usageError(argv[0], "Wrong number of arguments\n");

    msqid = getInt(argv[optind], 0, "msqid");
    maxBytes = (argc > optind + 1) ?
                getInt(argv[optind + 1], 0, "max-bytes") : MAX_MTEXT;

    /* Get message and display on stdout */

    msgLen = msgrcv(msqid, &msg, maxBytes, type, flags);
    if (msgLen == -1)
        errExit("msgrcv");

    printf("Received: type=%ld; length=%ld", msg.mtype, (long) msgLen);
    if (msgLen > 0)
        printf("; body=%s", msg.mtext);
    printf("\n");

    exit(EXIT_SUCCESS);
}
      `svmsg/svmsg_receive.c`
```

## 消息队列控制操作

*msgctl()* 系统调用对由*msqid*标识的消息队列执行控制操作。

```
#include <sys/types.h>        /* For portability */
#include <sys/msg.h>

int `msgctl`(int *msqid*, int *cmd*, struct msqid_ds **buf*);
```

### 注意

成功时返回 0，出错时返回 -1

*cmd*参数指定要对队列执行的操作。它可以是以下之一：

`IPC_RMID`

立即删除消息队列对象及其关联的*msqid_ds*数据结构。队列中剩余的所有消息将丢失，任何被阻塞的读写进程将立即被唤醒，*msgsnd()*或*msgrcv()*调用将因错误`EIDRM`而失败。*msgctl()*的第三个参数对于此操作被忽略。

`IPC_STAT`

将与此消息队列相关的*msqid_ds*数据结构的副本放入指向*buf*的缓冲区中。我们在第 46.4 节中描述了*msqid_ds*结构。

`IPC_SET`

使用指向*buf*的缓冲区中提供的值，更新与此消息队列相关的*msqid_ds*数据结构的选定字段。

这些操作的详细信息，包括调用进程所需的特权和权限，详见第 45.3 节。我们在第 46.6 节中描述了*cmd*的其他一些值。

示例 46-4 中的程序展示了使用 *msgctl()* 删除消息队列的过程。

示例 46-4. 删除 System V 消息队列

```
`svmsg/svmsg_rm.c`
#include <sys/types.h>
#include <sys/msg.h>
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    int j;

    if (argc > 1 && strcmp(argv[1], "--help") == 0)
        usageErr("%s [msqid...]\n", argv[0]);

    for (j = 1; j < argc; j++)
        if (msgctl(getInt(argv[j], 0, "msqid"), IPC_RMID, NULL) == -1)
            errExit("msgctl %s", argv[j]);

    exit(EXIT_SUCCESS);
}
      `svmsg/svmsg_rm.c`
```

## 消息队列关联数据结构

每个消息队列都有一个关联的 *msqid_ds* 数据结构，其形式如下：

```
struct msqid_ds {
    struct ipc_perm msg_perm;           /* Ownership and permissions */
    time_t          msg_stime;          /* Time of last msgsnd() */
    time_t          msg_rtime;          /* Time of last msgrcv() */
    time_t          msg_ctime;          /* Time of last change */
    unsigned long   __msg_cbytes;       /* Number of bytes in queue */
    msgqnum_t       msg_qnum;           /* Number of messages in queue */
    msglen_t        msg_qbytes;         /* Maximum bytes in queue */
    pid_t           msg_lspid;          /* PID of last msgsnd() */
    pid_t           msg_lrpid;          /* PID of last msgrcv() */
};
```

### 注意

*msqid_ds* 中的简写 *msq* 目的是为了让程序员感到困惑。这是唯一使用此拼写的消息队列接口。

*msgqnum_t* 和 *msglen_t* 数据类型——用于定义 *msg_qnum* 和 *msg_qbytes* 字段——是无符号整数类型，规定于 SUSv3 中。

*msqid_ds* 结构的字段会在各类消息队列系统调用中隐式更新，并且某些字段可以通过 *msgctl()* `IPC_SET` 操作显式更新。详细信息如下：

*msg_perm*

当消息队列被创建时，此子结构的字段按第 45.3 节所述初始化。*uid*、*gid* 和 *mode* 子字段可以通过 `IPC_SET` 更新。

*msg_stime*

当队列创建时，此字段被设置为 0；每次成功的 *msgsnd()* 会将此字段设置为当前时间。此字段和 *msqid_ds* 结构中的其他时间戳字段被定义为 *time_t* 类型；它们存储的是自纪元以来的秒数。

*msg_rtime*

此字段在消息队列创建时被设置为 0，然后在每次成功的 *msgrcv()* 后设置为当前时间。

*msg_ctime*

此字段在消息队列创建时以及每次成功执行 `IPC_SET` 操作时，都会被设置为当前时间。

*__msg_cbytes*

此字段在消息队列创建时被设置为 0，然后在每次成功的 *msgsnd()* 和 *msgrcv()* 后调整，以反映队列中所有消息的 *mtext* 字段所包含的总字节数。

*msg_qnum*

当消息队列创建时，此字段被设置为 0。随后，每次成功的 *msgsnd()* 会使其加一，每次成功的 *msgrcv()* 会使其减一，以反映队列中消息的总数。

*msg_qbytes*

该字段中的值定义了消息队列中所有消息的 *mtext* 字段的字节数上限。此字段在队列创建时会初始化为 `MSGMNB` 限制值。一个具有特权的（`CAP_SYS_RESOURCE`）进程可以使用 `IPC_SET` 操作将 *msg_qbytes* 调整为 0 到 `INT_MAX`（在 32 位平台上为 2,147,483,647）字节的任意值。一个没有特权的进程可以将 *msg_qbytes* 调整为 0 到 `MSGMNB` 的任何值。一个特权用户可以修改 Linux 特定的 `/proc/sys/kernel/msgmnb` 文件中的值，以更改所有后续创建的消息队列的初始 *msg_qbytes* 设置，以及未特权进程对 *msg_qbytes* 的后续更改的上限。我们将在第 46.5 节进一步讨论消息队列的限制。

*msg_lspid*

当队列被创建时，该字段被设置为 0，并在每次成功执行*msgsnd()*时被设置为调用进程的进程 ID。

*msg_lrpid*

当消息队列被创建时，该字段被设置为 0，并在每次成功执行*msgrcv()*时被设置为调用进程的进程 ID。

上述所有字段由 SUSv3 指定，__*msg_cbytes*__ 除外。不过，大多数 UNIX 实现提供了相当于 __*msg_cbytes*__ 字段的实现。

示例 46-5 中的程序演示了如何使用`IPC_STAT`和`IPC_SET`操作来修改消息队列的*msg_qbytes*设置。

示例 46-5. 修改 System V 消息队列的*msg_qbytes*设置

```
`svmsg/svmsg_chqbytes.c`
#include <sys/types.h>
#include <sys/msg.h>
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    struct msqid_ds ds;
    int msqid;
    if (argc != 3 || strcmp(argv[1], "--help") == 0)
        usageErr("%s msqid max-bytes\n", argv[0]);

    /* Retrieve copy of associated data structure from kernel */

    msqid = getInt(argv[1], 0, "msqid");
    if (msgctl(msqid, IPC_STAT, &ds) == -1)
        errExit("msgctl");

    ds.msg_qbytes = getInt(argv[2], 0, "max-bytes");

    /* Update associated data structure in kernel */

    if (msgctl(msqid, IPC_SET, &ds) == -1)
        errExit("msgctl");

    exit(EXIT_SUCCESS);
}
      `svmsg/svmsg_chqbytes.c`
```

## 消息队列限制

大多数 UNIX 实现对 System V 消息队列的操作施加了各种限制。这里，我们描述了 Linux 下的限制，并指出与其他 UNIX 实现的若干差异。

以下限制在 Linux 上强制执行。受限的系统调用以及如果达到限制时产生的错误会在括号中注明。

`MSGMNI`

这是一个系统级别的限制，规定了可以创建的消息队列标识符数量（换句话说，消息队列的数量）。(*msgget()*, `ENOSPC`)

`MSGMAX`

这是一个系统级别的限制，指定了可以在单个消息中写入的最大(*mtext*)字节数。(*msgsnd()*, `EINVAL`)

`MSGMNB`

这是消息队列中一次可以容纳的最大(*mtext*)字节数。这个限制是一个系统级别的参数，用来初始化与该消息队列关联的*msqid_ds*数据结构中的*msg_qbytes*字段。随后，可以根据每个队列的需要修改*msg_qbytes*值，如第 46.4 节所述。如果队列的*msg_qbytes*限制被达到，则*msgsnd()*会被阻塞，或者如果设置了`IPC_NOWAIT`，则返回错误`EAGAIN`。

一些 UNIX 实现还定义了以下进一步的限制：

`MSGTQL`

这是一个系统级别的限制，规定了可以放置在系统所有消息队列上的消息数量。

`MSGPOOL`

这是一个系统级别的限制，规定了用于保存系统中所有消息队列数据的缓冲池的大小。

虽然 Linux 不强加上述限制，但它限制了单个队列中消息的数量，限制值由队列的*msg_qbytes*设置指定。这个限制仅在我们向队列写入零长度消息时相关。它的作用是，零长度消息的数量限制与可以写入队列的 1 字节消息的数量限制相同。这是为了防止写入无限数量的零长度消息。尽管这些消息不包含数据，但每个零长度消息会占用一小部分内存用于系统的账本开销。

在系统启动时，消息队列限制会设置为默认值。这些默认值在不同的内核版本之间有所不同。（一些发行版的内核设置的默认值与原生内核提供的有所不同。）在 Linux 上，可以通过 `/proc` 文件系统中的文件查看或更改这些限制。表 46-1 显示了与每个限制对应的 `/proc` 文件。例如，以下是我们在一台 x86-32 系统上看到的 Linux 2.6.31 的默认限制：

```
$ `cd /proc/sys/kernel`
$ `cat msgmni`
748
$ `cat msgmax`
8192
$ `cat msgmnb`
16384
```

表 46-1. 系统 V 消息队列限制

| 限制 | 上限值（x86-32） | 对应的 `/proc/sys/kernel` 中的文件 |
| --- | --- | --- |
| `MSGMNI` | 32768 (`IPCMNI`) | `msgmni` |
| `MSGMAX` | 取决于可用内存 | `msgmax` |
| `MSGMNB` | 2147483647 (`INT_MAX`) | `msgmnb` |

表 46-1 中的上限值列显示了在 x86-32 架构上每个限制可以提升的最大值。请注意，尽管 `MSGMNB` 限制可以提高到 `INT_MAX`，但在消息队列加载如此多数据之前，其他一些限制（例如，内存不足）会先达到。

Linux 特定的 *msgctl()* `IPC_INFO` 操作检索类型为 *msginfo* 的结构，其中包含各种消息队列限制的值：

```
struct msginfo buf;

msgctl(0, IPC_INFO, (struct msqid_ds *) &buf);
```

有关 `IPC_INFO` 和 *msginfo* 结构的详细信息，请参见 *msgctl(2)* 手册页。

## 显示系统上的所有消息队列

在 获取所有 IPC 对象的列表中，我们查看了通过 `/proc` 文件系统中的一组文件来获取系统上所有 IPC 对象列表的一个方法。现在我们来看获取相同信息的第二种方法：通过一组 Linux 特定的 IPC *ctl*（*msgctl()*、*semctl()* 和 *shmctl()*）操作。（*ipcs* 程序使用这些操作。）这些操作如下：

+   `MSG_INFO`、`SEM_INFO` 和 `SHM_INFO`：`MSG_INFO` 操作有两个目的。首先，它返回一个结构，详细说明系统上所有消息队列消耗的资源。其次，作为 *ctl* 调用的函数结果，它返回指向消息队列对象数据结构的 *entries* 数组中最大项的索引（参见 图 45-1，以及 系统 V IPC *get* 调用所采用的算法）。`SEM_INFO` 和 `SHM_INFO` 操作分别为信号量集和共享内存段执行类似的任务。我们必须定义 `_GNU_SOURCE` 特性测试宏，以便从相应的系统 V IPC 头文件中获取这三个常量的定义。

    ### 注意

    一个示例展示了如何使用 `MSG_INFO` 获取一个 *msginfo* 结构体，包含有关所有消息队列对象使用的资源信息，示例代码位于本书源代码分发包中的 `svmsg/svmsg_info.c` 文件。

+   `MSG_STAT`、`SEM_STAT` 和 `SHM_STAT`：与 `IPC_STAT` 操作类似，这些操作用于检索 IPC 对象的相关数据结构。它们的区别在于两个方面。首先，这些操作期望的是 *entries* 数组中的索引，而不是 IPC 标识符作为 *ctl* 调用的第一个参数。其次，如果操作成功，那么作为函数结果，*ctl* 调用将返回与该索引对应的 IPC 对象的标识符。我们必须定义 `_GNU_SOURCE` 功能测试宏，以从相应的 System V IPC 头文件中获取这三个常量的定义。

要列出系统上所有的消息队列，我们可以执行以下操作：

1.  使用 `MSG_INFO` 操作来查找消息队列的 *entries* 数组的最大索引（*maxind*）。

1.  对从 0 到包括 *maxind* 的所有值执行循环，为每个值使用 `MSG_STAT` 操作。在这个循环过程中，我们忽略可能出现的错误，比如如果 *entries* 数组中的某个项为空（`EINVAL`），或者我们没有权限访问它所指向的对象（`EACCES`）。

示例 46-6 提供了上述消息队列操作步骤的实现。以下是该程序使用的 shell 会话日志：

```
$ `./svmsg_ls`
maxind: 4

index     ID       key      messages
   2    98306  0x00000000       0
   4   163844  0x000004d2       2
$ `ipcs -q`                               *Check above against output of ipcs*

------ Message Queues --------
key        msqid      owner    perms    used-bytes   messages
0x00000000 98306      mtk      600      0            0
0x000004d2 163844     mtk      600      12           2
```

示例 46-6. 显示系统上所有 System V 消息队列

```
`svmsg/svmsg_ls.c`
#define _GNU_SOURCE
#include <sys/types.h>
#include <sys/msg.h>
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    int maxind, ind, msqid;
    struct msqid_ds ds;
    struct msginfo msginfo;

    /* Obtain size of kernel 'entries' array */

    maxind = msgctl(0, MSG_INFO, (struct msqid_ds *) &msginfo);
    if (maxind == -1)
        errExit("msgctl-MSG_INFO");

    printf("maxind: %d\n\n", maxind);
    printf("index     id       key      messages\n");

    /* Retrieve and display information from each element of 'entries' array */

    for (ind = 0; ind <= maxind; ind++) {
        msqid = msgctl(ind, MSG_STAT, &ds);
        if (msqid == -1) {
            if (errno != EINVAL && errno != EACCES)
                errMsg("msgctl-MSG_STAT");              /* Unexpected error */
            continue;                                   /* Ignore this item */
        }

        printf("%4d %8d  0x%08lx %7ld\n", ind, msqid,
                (unsigned long) ds.msg_perm.__key, (long) ds.msg_qnum);
    }

    exit(EXIT_SUCCESS);
}
      `svmsg/svmsg_ls.c`
```

## 使用消息队列进行客户端-服务器编程

在本节中，我们考虑了使用 System V 消息队列实现客户端-服务器应用程序的两种可能设计：

+   在服务器和客户端之间使用单一的消息队列进行双向消息交换。

+   为服务器和每个客户端使用单独的消息队列。服务器的队列用于接收来自客户端的请求，而响应通过单独的客户端队列发送给客户端。

我们选择哪种方法取决于应用程序的需求。接下来，我们将考虑一些可能影响我们选择的因素。

#### 为服务器和客户端使用单一的消息队列

当服务器和客户端之间交换的消息较小时，使用单一的消息队列可能是合适的。然而，需要注意以下几点：

+   由于多个进程可能同时尝试读取消息，我们必须使用消息类型（*mtype*）字段来允许每个进程只选择那些针对它的消息。实现这一点的一种方法是使用客户端的进程 ID 作为从服务器发送到客户端的消息的消息类型。客户端可以将其进程 ID 作为其消息的一部分发送给服务器。此外，发送到服务器的消息必须也通过一个唯一的消息类型来区分。为此，我们可以使用数字 1，作为始终运行的*init*进程的进程 ID，它永远不会是客户端进程的进程 ID。（另一种方法是使用服务器的进程 ID 作为消息类型；然而，客户端很难获得此信息。）这种编号方案如图 46-2 所示。

+   消息队列具有有限的容量，这可能导致几个问题。其中之一是多个同时客户端可能会填满消息队列，导致死锁情况，在这种情况下，无法提交新的客户端请求，服务器也无法写入任何响应。另一个问题是一个表现不良或故意恶意的客户端可能无法读取来自服务器的响应。这会导致队列被未读消息堵塞，阻止客户端与服务器之间的任何通信。（使用两个队列——一个用于客户端到服务器的消息，另一个用于服务器到客户端的消息——可以解决第一个问题，但无法解决第二个问题。）

![使用单一消息队列进行客户端-服务器 IPC](img/46-2_SVMSG-cs-single-queue.png.jpg)图 46-2. 使用单一消息队列进行客户端-服务器 IPC

#### 每个客户端使用一个消息队列

每个客户端使用一个消息队列（以及一个服务器的消息队列）更为可取，尤其是在需要交换大消息或使用单一消息队列时可能出现上述问题的情况下。关于这种方法，请注意以下几点：

+   每个客户端必须创建自己的消息队列（通常使用`IPC_PRIVATE`键），并通过将队列标识符作为客户端消息的一部分传递给服务器来告知服务器该队列的标识符。

+   系统中有一个关于消息队列数量的系统限制（`MSGMNI`），在一些系统上，该限制的默认值非常低。如果我们预计会有大量的同时客户端连接，可能需要提高这个限制。

+   服务器应该考虑到客户端的消息队列可能不再存在（可能是因为客户端提前删除了它）。

我们将在下一节中进一步讨论每个客户端使用一个消息队列的方式。

## 使用消息队列的文件服务器应用程序

在本节中，我们描述了一个使用每个客户端一个消息队列的客户端-服务器应用程序。该应用程序是一个简单的文件服务器。客户端向服务器的消息队列发送请求消息，要求获取指定文件的内容。服务器通过将文件内容作为一系列消息返回到客户端的私有消息队列来进行响应。图 46-3 提供了该应用程序的概述。

由于服务器不对客户端进行认证，任何能够运行客户端的用户都可以访问服务器上任何可以访问的文件。更为复杂的服务器会在为客户端提供请求的文件之前，要求进行某种形式的客户端认证。

![每个客户端使用一个消息队列的客户端-服务器 IPC](img/46-3_SVMSG-cs-multi-queue-scale90.png.jpg)图 46-3. 每个客户端使用一个消息队列的客户端-服务器 IPC

#### 公共头文件

示例 46-7 是服务器和客户端都包含的头文件。此头文件定义了用于服务器消息队列的知名密钥（`SERVER_KEY`），并定义了客户端与服务器之间传递消息的格式。

*requestMsg* 结构体定义了从客户端发送到服务器的请求格式。在这个结构体中，*mtext* 组件包含两个字段：客户端消息队列的标识符和客户端请求的文件的路径名。常量 `REQ_MSG_SIZE` 等于这两个字段的总大小，并作为 *msgsz* 参数，在使用此结构体的 *msgsnd()* 调用中传递。

*responseMsg* 结构体定义了从服务器返回给客户端的响应消息的格式。*mtype* 字段用于响应消息中，提供有关消息内容的信息，如 `RESP_MT_*` 常量所定义。

示例 46-7. `svmsg_file_server.c` 和 `svmsg_file_client.c` 的头文件

```
`svmsg/svmsg_file.h`
#include <sys/types.h>
#include <sys/msg.h>
#include <sys/stat.h>
#include <stddef.h>                       /* For definition of offsetof() */
#include <limits.h>
#include <fcntl.h>
#include <signal.h>
#include <sys/wait.h>
#include "tlpi_hdr.h"

#define SERVER_KEY 0x1aaaaaa1             /* Key for server's message queue */

struct requestMsg {                       /* Requests (client to server) */
    long mtype;                           /* Unused */
    int  clientId;                        /* ID of client's message queue */
    char pathname[PATH_MAX];              /* File to be returned */
};

/* REQ_MSG_SIZE computes size of 'mtext' part of 'requestMsg' structure.
   We use offsetof() to handle the possibility that there are padding
   bytes between the 'clientId' and 'pathname' fields. */

#define REQ_MSG_SIZE (offsetof(struct requestMsg, pathname) - \
                      offsetof(struct requestMsg, clientId) + PATH_MAX)

#define RESP_MSG_SIZE 8192

struct responseMsg {                      /* Responses (server to client) */
    long mtype;                           /* One of RESP_MT_* values below */
    char data[RESP_MSG_SIZE];             /* File content / response message */
};

/* Types for response messages sent from server to client */

#define RESP_MT_FAILURE 1                 /* File couldn't be opened */
#define RESP_MT_DATA    2                 /* Message contains file data */
#define RESP_MT_END     3                 /* File data complete */
     `svmsg/svmsg_file.h`
```

#### 服务器程序

示例 46-8 是该应用程序的服务器程序。请注意以下几点关于服务器的内容：

+   服务器设计为并发处理请求。与在示例 44-7（第 912 页）中使用的迭代设计相比，并发服务器设计更为优越，因为我们希望避免客户端请求大文件时，导致其他所有客户端请求被阻塞的情况。

+   每个客户端请求通过创建一个子进程来处理，该进程为请求的文件提供服务 ![](img/U008.png)。与此同时，主服务器进程会等待进一步的客户端请求。请注意以下关于服务器子进程的几点：

    +   由于通过*fork()*创建的子进程继承了父进程的堆栈副本，因此它获得了主服务器进程读取的请求消息副本。

    +   服务器子进程在处理完其关联的客户端请求后终止！[](figs/web/U009.png)。

+   为了避免产生僵尸进程（孤儿进程和僵尸进程），服务器为`SIGCHLD`建立了一个处理程序！[](figs/web/U006.png)，并在该处理程序内调用*waitpid()*！[](figs/web/U001.png)。

+   父服务器进程中的*msgrcv()*调用可能会阻塞，因此可能会被`SIGCHLD`处理程序中断。为了处理这种情况，使用了一个循环，当调用失败并返回`EINTR`错误时，会重新启动该调用！[](figs/web/U007.png)。

+   服务器子进程执行*serveRequest()*函数！[](figs/web/U002.png)，该函数向客户端发送三种类型的消息。*mtype*为`RESP_MT_FAILURE`的请求表示服务器无法打开请求的文件！[](figs/web/U003.png)；`RESP_MT_DATA`用于一系列包含文件数据的消息！[](figs/web/U004.png)；`RESP_MT_END`（数据字段为空）用于表示文件数据传输已完成！[](figs/web/U005.png)。

我们考虑了多种方法来改进和扩展练习 46-4 中的服务器程序。

示例 46-8. 使用 System V 消息队列的文件服务器

```
`svmsg/svmsg_file_server.c`
    #include "svmsg_file.h"

    static void             /* SIGCHLD handler */
    grimReaper(int sig)
    {
        int savedErrno;

        savedErrno = errno;                 /* waitpid() might change 'errno' */
    while (waitpid(-1, NULL, WNOHANG) > 0)
            continue;
        errno = savedErrno;
    }

        static void             /* Executed in child process: serve a single client */
 serveRequest(const struct requestMsg *req)
    {
        int fd;
        ssize_t numRead;
        struct responseMsg resp;

        fd = open(req->pathname, O_RDONLY);
        if (fd == -1) {                     /* Open failed: send error text */
        resp.mtype = RESP_MT_FAILURE;
            snprintf(resp.data, sizeof(resp.data), "%s", "Couldn't open");
            msgsnd(req->clientId, &resp, strlen(resp.data) + 1, 0);
            exit(EXIT_FAILURE);              /* and terminate */
        }

        /* Transmit file contents in messages with type RESP_MT_DATA. We don't
           diagnose read() and msgsnd() errors since we can't notify client. */
    resp.mtype = RESP_MT_DATA;
        while ((numRead = read(fd, resp.data, RESP_MSG_SIZE)) > 0)
            if (msgsnd(req->clientId, &resp, numRead, 0) == -1)
                break;

        /* Send a message of type RESP_MT_END to signify end-of-file */
    resp.mtype = RESP_MT_END;
        msgsnd(req->clientId, &resp, 0, 0);         /* Zero-length mtext */
    }

    int
    main(int argc, char *argv[])
    {
        struct requestMsg req;
        pid_t pid;
        ssize_t msgLen;
        int serverId;
        struct sigaction sa;

        /* Create server message queue */

        serverId = msgget(SERVER_KEY, IPC_CREAT | IPC_EXCL |
                                S_IRUSR | S_IWUSR | S_IWGRP);
        if (serverId == -1)
            errExit("msgget");

        /* Establish SIGCHLD handler to reap terminated children */

        sigemptyset(&sa.sa_mask);
        sa.sa_flags = SA_RESTART;
        sa.sa_handler = grimReaper;
    if (sigaction(SIGCHLD, &sa, NULL) == -1)
            errExit("sigaction");

            /* Read requests, handle each in a separate child process */

        for (;;) {
            msgLen = msgrcv(serverId, &req, REQ_MSG_SIZE, 0, 0);
            if (msgLen == -1) {
            if (errno == EINTR)         /* Interrupted by SIGCHLD handler? */
                    continue;               /* ... then restart msgrcv() */
                errMsg("msgrcv");           /* Some other error */
                break;                      /* ... so terminate loop */
            }
        pid = fork();                   /* Create child process */
            if (pid == -1) {
                errMsg("fork");
                break;
            }

            if (pid == 0) {                 /* Child handles request */
                serveRequest(&req);
            _exit(EXIT_SUCCESS);
            }

            /* Parent loops to receive next client request */
        }

        /* If msgrcv() or fork() fails, remove server MQ and exit */

        if (msgctl(serverId, IPC_RMID, NULL) == -1)
            errExit("msgctl");
        exit(EXIT_SUCCESS);
    }
         `svmsg/svmsg_file_server.c`
```

#### 客户端程序

示例 46-9 是该应用程序的客户端程序。请注意以下内容：

+   客户端创建一个使用`IPC_PRIVATE`键的消息队列！[](figs/web/U002.png)，并使用*atexit()*！[](figs/web/U003.png)建立一个退出处理程序！[](figs/web/U001.png)，以确保客户端退出时删除该队列。

+   客户端通过请求向服务器传递其队列的标识符以及要提供的文件的路径名！[](figs/web/U004.png)。

+   客户端处理服务器发送的第一个响应消息可能是失败通知的情况（*mtype*等于`RESP_MT_FAILURE`），通过打印服务器返回的错误消息文本并退出！[](figs/web/U005.png)。

+   如果文件成功打开，客户端将进入循环！[](figs/web/U006.png)，接收一系列包含文件内容的消息（*mtype* 等于 `RESP_MT_DATA`）。当接收到结束文件的消息（*mtype* 等于 `RESP_MT_END`）时，循环终止。

这个简单的客户端没有处理服务器故障所导致的各种情况。我们将在练习 46-5 中考虑一些改进。

示例 46-9. 使用 System V 消息队列的文件服务器客户端

```
`svmsg/svmsg_file_client.c`
    #include "svmsg_file.h"

    static int clientId;

    static void
    removeQueue(void)
    {
        if (msgctl(clientId, IPC_RMID, NULL) == -1)
        errExit("msgctl");
    }

    int
    main(int argc, char *argv[])
    {
        struct requestMsg req;
        struct responseMsg resp;
        int serverId, numMsgs;
        ssize_t msgLen, totBytes;

        if (argc != 2 || strcmp(argv[1], "--help") == 0)
            usageErr("%s pathname\n", argv[0]);

        if (strlen(argv[1]) > sizeof(req.pathname) - 1)
            cmdLineErr("pathname too long (max: %ld bytes)\n",
                    (long) sizeof(req.pathname) - 1);

        /* Get server's queue identifier; create queue for response */

        serverId = msgget(SERVER_KEY, S_IWUSR);
        if (serverId == -1)
            errExit("msgget - server message queue");
    clientId = msgget(IPC_PRIVATE, S_IRUSR | S_IWUSR | S_IWGRP);
        if (clientId == -1)
            errExit("msgget - client message queue");
    if (atexit(removeQueue) != 0)
            errExit("atexit");

        /* Send message asking for file named in argv[1] */

        req.mtype = 1;                      /* Any type will do */
        req.clientId = clientId;
        strncpy(req.pathname, argv[1], sizeof(req.pathname) - 1);
        req.pathname[sizeof(req.pathname) - 1] = '\0';
                                            /* Ensure string is terminated */
    if (msgsnd(serverId, &req, REQ_MSG_SIZE, 0) == -1)
            errExit("msgsnd");
            /* Get first response, which may be failure notification */

        msgLen = msgrcv(clientId, &resp, RESP_MSG_SIZE, 0, 0);
        if (msgLen == -1)
            errExit("msgrcv");
    if (resp.mtype == RESP_MT_FAILURE) {
            printf("%s\n", resp.data);      /* Display msg from server */
            if (msgctl(clientId, IPC_RMID, NULL) == -1)
                errExit("msgctl");
            exit(EXIT_FAILURE);
        }

        /* File was opened successfully by server; process messages
           (including the one already received) containing file data */

        totBytes = msgLen;                  /* Count first message */
    for (numMsgs = 1; resp.mtype == RESP_MT_DATA; numMsgs++) {
            msgLen = msgrcv(clientId, &resp, RESP_MSG_SIZE, 0, 0);
            if (msgLen == -1)
                errExit("msgrcv");

            totBytes += msgLen;
        }

        printf("Received %ld bytes (%d messages)\n", (long) totBytes, numMsgs);

        exit(EXIT_SUCCESS);
    }
         `svmsg/svmsg_file_client.c`
```

以下 shell 会话演示了在示例 46-8 和示例 46-9 中程序的使用：

```
$ `./svmsg_file_server &`                   *Run server in background*
[1] 9149
$ `wc -c /etc/services`
                     *Show size of file that client will request*
764360 /etc/services
$ `./svmsg_file_client /etc/services`
Received 764360 bytes (95 messages)       *Bytes received matches size above*
$ `kill %1`                                 *Terminate server*
[1]+  Terminated        ./svmsg_file_server
```

## System V 消息队列的缺点

UNIX 系统提供了多种机制，用于在同一系统上将数据从一个进程传输到另一个进程，数据可以是未分隔的字节流（管道、FIFO 和 UNIX 域流套接字）或分隔的消息（System V 消息队列、POSIX 消息队列和 UNIX 域数据报套接字）。

System V 消息队列的一个独特特点是能够为每条消息附加一个数字类型。这提供了两种可能性，可能对应用程序有用：读取进程可以按类型选择消息，或者它们可以采用优先级队列策略，使得优先级较高的消息（即类型值较低的消息）先被读取。

然而，System V 消息队列有一些缺点：

+   消息队列通过标识符进行引用，而不是像大多数其他 UNIX I/O 机制那样使用文件描述符。这意味着在第六十三章中描述的多种基于文件描述符的 I/O 技术（例如，*select()*, *poll()*, 和 *epoll()*）不能应用于消息队列。此外，编写同时处理来自消息队列和基于文件描述符的 I/O 机制输入的程序需要比仅处理文件描述符的程序更复杂的代码。（我们在练习 63-3 中探讨了一种结合这两种 I/O 模型的方法。）

+   使用键而不是文件名来标识消息队列会增加编程复杂性，并且还需要使用*ipcs*和*ipcrm*，而不是*ls*和*rm*。*ftok()* 函数通常会生成唯一的键，但不能保证一定生成唯一键。使用 `IPC_PRIVATE` 键可以保证唯一的队列标识符，但我们仍然需要使该标识符对其他需要它的进程可见。

+   消息队列是无连接的，内核不会像管道、FIFO 和套接字那样跟踪引用队列的进程数。因此，回答以下问题可能会很困难：

    +   应用程序何时可以安全地删除消息队列？（过早删除队列会导致数据立即丢失，无论是否有进程可能稍后想从队列中读取。）

    +   应用程序如何确保未使用的队列被删除？

+   消息队列的总数、消息的大小以及单个队列的容量是有限制的。这些限制是可配置的，但如果应用程序在默认限制范围之外运行，则在安装应用程序时需要额外的工作。

总结来说，System V 消息队列通常最好避免使用。在需要按类型选择消息的情况下，我们应该考虑其他替代方案。POSIX 消息队列（第五十二章）就是一种替代方案。作为进一步的替代，基于多个文件描述符的通信渠道解决方案可能提供类似于按类型选择消息的功能，同时允许使用第六十三章中描述的其他 I/O 模型。例如，如果我们需要传输“正常”消息和“优先”消息，可以使用一对 FIFO 或 UNIX 域套接字来表示这两种消息类型，然后使用*select()* 或 *poll()* 来监视这两个通道的文件描述符。

## 总结

System V 消息队列允许进程通过交换包含数字类型和任意数据的消息体进行通信。消息队列的显著特点是保留了消息边界，并且接收方可以按类型选择消息，而不是按先进先出顺序读取消息。

各种因素使我们得出结论，通常其他 IPC 机制比 System V 消息队列更为优选。一个主要的困难是消息队列不是通过文件描述符引用的。这意味着我们无法使用各种替代 I/O 模型与消息队列一起使用；特别是，同时监视消息队列和文件描述符以查看是否可以进行 I/O 操作是复杂的。此外，消息队列是无连接的（即，没有引用计数），这使得应用程序很难知道何时可以安全地删除队列。

## 练习

1.  在示例 46-1")（`svmsg_create.c`）、示例 46-2 发送消息")（`svmsg_send.c`）和示例 46-3 读取消息")（`svmsg_receive.c`）中进行实验，以确认你对*msgget()*、*msgsnd()* 和 *msgrcv()* 系统调用的理解。

1.  将 A Client-Server Application Using FIFOs 中的序列号客户端-服务器应用程序重新编码为使用 System V 消息队列。使用单一消息队列在客户端和服务器之间、以及服务器和客户端之间传输消息。采用第 46.8 节中描述的消息类型约定。

1.  在使用消息队列的文件服务器应用程序的客户端-服务器应用程序中，为什么客户端将其消息队列的标识符放在消息体中（*clientId*字段），而不是在消息类型（*mtype*）中？

1.  对使用消息队列的文件服务器应用程序的客户端-服务器应用程序进行以下更改：

    1.  替换服务器中硬编码的消息队列键，改为使用`IPC_PRIVATE`生成唯一标识符，然后将该标识符写入一个知名文件。客户端必须从该文件中读取标识符。如果服务器终止，应该删除此文件。

    1.  在服务器程序的*serveRequest()*函数中，系统调用错误没有被诊断。添加代码，通过*syslog()*记录错误（使用*syslog*记录消息和错误）。

    1.  向服务器添加代码，使其在启动时成为一个守护进程（创建守护进程）。

    1.  在服务器中，添加一个用于处理`SIGTERM`和`SIGINT`信号的处理程序，执行干净的退出操作。该处理程序应该移除消息队列，并且（如果前面部分的练习已实现）删除用于存放服务器消息队列标识符的文件。处理程序中应包括代码，通过取消设置处理程序来终止服务器，然后再次触发引发处理程序的相同信号（参见信号处理中的进程终止了解此任务的理由和所需步骤）。

    1.  服务器子进程没有处理客户端可能提前终止的情况，在这种情况下，服务器子进程会填满客户端的消息队列，然后无限期地阻塞。修改服务器，以处理这种情况，通过在调用*msgsnd()*时设置超时，如第 23.3 节所述。如果服务器子进程认为客户端已消失，它应该尝试删除客户端的消息队列，然后退出（可能会通过*syslog()*记录一条消息）。

1.  示例 46-9 中显示的客户端（`svmsg_file_client.c`）没有处理服务器可能出现的各种失败情况。特别是，如果服务器的消息队列已满（可能是因为服务器终止，且队列被其他客户端填满），那么*msgsnd()*调用将会无限期阻塞。同样，如果服务器未能发送响应给客户端，则*msgrcv()*调用将会无限期阻塞。请在客户端代码中添加设置超时的代码（设置阻塞操作的超时）。如果任何一个调用超时，程序应该向用户报告错误并终止。

1.  编写一个简单的聊天应用程序（类似于*talk(1)*，但没有*curses*界面），使用 System V 消息队列。为每个客户端使用一个单独的消息队列。
