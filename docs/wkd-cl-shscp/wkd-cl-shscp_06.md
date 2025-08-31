## 第六章：**系统管理：管理用户**

![image](img/common4.jpg)

无论是 Windows、OS X 还是 Unix，任何复杂的操作系统都无法在没有人工干预的情况下无限期运行。如果你使用的是多用户的 Linux 系统，肯定会有人在执行必要的系统管理任务。你可能忽略了那个“幕后的人”，他负责管理和维护一切，或者你可能正是那个“大魔法师”自己，掌控着一切操作，保持系统的正常运行。如果你使用的是单用户系统，那么你应该定期执行一些系统管理任务。

幸运的是，简化 Linux 系统管理员的工作（本章目标）是 Shell 脚本最常见的用途之一。实际上，许多 Linux 命令实际上就是 Shell 脚本，许多最基本的任务，如添加用户、分析磁盘使用情况以及管理访客帐户的文件空间，都可以通过简短的脚本更高效地完成。

令人惊讶的是，许多系统管理脚本的长度也不过是 20 到 30 行。事实上，你可以使用 Linux 命令来识别脚本，并通过管道命令来查找每个脚本的行数。以下是 */usr/bin/* 中 15 个最短的脚本：

```
$ file /usr/bin/* | grep "shell script" | cut -d: -f1 | xargs wc -l \
| sort -n | head -15
    3 zcmp
    3 zegrep
    3 zfgrep
    4 mkfontdir
    5 pydoc
    7 sgmlwhich
    8 batch
    8 ps2pdf12
    8 ps2pdf13
    8 ps2pdf14
    8 timed-read
    9 timed-run
   10 c89
   10 c99
   10 neqn
```

*/usr/bin/* 目录中最短的 15 个脚本都不超过 10 行。而在 10 行中，方程式格式化脚本 `neqn` 是一个很好的例子，展示了一个小的 Shell 脚本如何真正改善用户体验：

```
#!/bin/bash
# Provision of this shell script should not be taken to imply that use of
#   GNU eqn with groff -Tascii|-Tlatin1|-Tutf8|-Tcp1047 is supported.

: ${GROFF_BIN_PATH=/usr/bin}
PATH=$GROFF_BIN_PATH:$PATH
export PATH
exec eqn -Tascii ${1+"$@"}

# eof
```

与 `neqn` 类似，本章介绍的脚本简短而实用，提供了一系列的管理功能，包括简单的系统备份；用户及其数据的创建、管理和删除；一个易于使用的 `date` 命令前端，用于更改当前的日期和时间；以及一个有用的工具来验证 *crontab* 文件。

### #35 分析磁盘使用情况

即便是大容量硬盘的出现以及它们价格的持续下降，系统管理员似乎仍然不断被要求监控磁盘使用情况，以防共享磁盘被填满。

最常见的监控技术是查看 */usr* 或 */home* 目录，使用 `du` 命令确定所有子目录的磁盘使用情况，并报告前 5 或前 10 个用户。然而，这种方法的问题在于，它没有考虑硬盘（或多个硬盘）上其他地方的空间使用情况。如果某些用户在第二个硬盘上有额外的归档空间，或者你有些偷偷摸摸的用户在 */tmp* 的点目录或 *ftp* 区域中的未使用目录中存放 MPEG 文件，那么这些使用情况将无法被检测到。此外，如果你的 home 目录分布在多个硬盘上，逐个搜索每个 */home* 目录未必是最优的做法。

更好的解决方案是直接从 */etc/passwd* 文件中获取所有帐户名，然后在文件系统中搜索每个帐户拥有的文件，如 列表 5-1 所示。

#### *代码*

```
   #!/bin/bash

   # fquota--Disk quota analysis tool for Unix; assumes all user
   #   accounts are >= UID 100

   MAXDISKUSAGE=20000   # In megabytes

   for name in $(cut -d: -f1,3 /etc/passwd | awk -F: '$2 > 99 {print $1}')
   do
     /bin/echo -n "User $name exceeds disk quota. Disk usage is: "
     # You might need to modify the following list of directories to match
     #   the layout of your disk. The most likely change is from /Users to /home.
➊   find / /usr /var /Users -xdev -user $name -type f -ls | \
       awk '{ sum += $7 } END { print sum / (1024*1024) " Mbytes" }'

➋ done | awk "\$9 > $MAXDISKUSAGE { print \$0 }"

   exit 0
```

*列表 5-1：* `*fquota*` *脚本*

#### *工作原理*

按惯例，用户 ID 从 1 到 99 用于系统守护进程和管理任务，而 100 及以上则用于用户账户。由于 Linux 管理员通常比较有条理，这个脚本跳过了所有 UID 小于 100 的账户。

`-xdev` 参数用于 `find` 命令 ➊，确保 `find` 不会遍历所有文件系统。换句话说，这个参数防止命令在系统区域、只读源目录、可移动设备、正在运行的进程的 */proc* 目录（在 Linux 中）以及类似区域中耗时。正因如此，我们明确指定了如 */usr*、*/var* 和 */home* 这样的目录。这些目录通常在各自独立的文件系统中，用于备份和管理目的。即使它们与根文件系统位于同一文件系统中，加入这些目录并不意味着它们会被重复搜索。

刚开始看，可能会觉得这个脚本对每个账户都输出 `超过磁盘配额` 的消息，但在循环之后的 `awk` 语句 ➋ 只会在账户的使用量大于预定义的 `MAXDISKUSAGE` 时报告此消息。

#### *运行脚本*

这个脚本没有参数，应该以 root 用户身份运行，以确保它有权限访问所有目录和文件系统。聪明的做法是使用有用的 `sudo` 命令（在终端运行命令 `man sudo` 以获取更多细节）。为什么 `sudo` 有用？因为它允许你以 root 用户身份执行*一个*命令，之后你会恢复为普通用户身份。每次你想运行一个管理命令时，都必须有意识地使用 `sudo`。与此相反，使用 `su - root` 会让你一直以 root 身份执行后续的所有命令，直到退出子 Shell，而一旦你分心，很容易忘记自己是 root，执行一些可能导致灾难的操作。

**注意**

*你需要修改* `*find*` *命令中列出的目录* ➊ *，使其与自己磁盘拓扑中的对应目录匹配。*

#### *结果*

因为这个脚本会跨文件系统进行搜索，所以它需要一些时间才能运行，这一点应该不足为奇。在大型系统上，运行时间可能会介于喝一杯茶和和你的伴侣共进午餐之间。列表 5-2 详细说明了结果。

```
$ sudo fquota
User taylor exceeds disk quota. Disk usage is: 21799.4 Mbytes
```

*列表 5-2：测试* `*fquota*` *脚本*

你可以看到 `taylor` 在磁盘使用方面完全失控！他使用的 21GB 明显超过了每个用户 20GB 的配额。

#### *破解脚本*

这种类型的完整脚本应该具有某种自动发送电子邮件的功能，用来警告那些占用过多磁盘空间的人。这个增强功能在下一个脚本中得到了演示。

### #36 报告磁盘占用者

大多数系统管理员都希望以最简单的方式解决问题，而管理磁盘配额的最简单方法是扩展`fquota`（见脚本 #35，第 119 页），直接向消耗过多空间的用户发出电子邮件警告，见列表 5-3。

#### *代码*

```
   #!/bin/bash

   # diskhogs--Disk quota analysis tool for Unix; assumes all user
   #   accounts are >= UID 100\. Emails a message to each violating user
   #   and reports a summary to the screen.

   MAXDISKUSAGE=500
➊ violators="/tmp/diskhogs0.$$"

➋ trap "$(which rm) -f $violators" 0

➌ for name in $(cut -d: -f1,3 /etc/passwd | awk -F: '$2 > 99 { print $1 }')
   do
➍   /bin/echo -n "$name "
     # You might need to modify the following list of directories to match the
     #   layout of your disk. The most likely change is from /Users to /home.
     find / /usr /var /Users -xdev -user $name -type f -ls | \
       awk '{ sum += $7 } END { print sum / (1024*1024) }'

   done | awk "\$2 > $MAXDISKUSAGE { print \$0 }" > $violators

➎ if [ ! -s $violators ] ; then
     echo "No users exceed the disk quota of ${MAXDISKUSAGE}MB"
     cat $violators
     exit 0
   fi

   while read account usage ; do

➏   cat << EOF | fmt | mail -s "Warning: $account Exceeds Quota" $account
     Your disk usage is ${usage}MB, but you have been allocated only
     ${MAXDISKUSAGE}MB. This means that you need to delete some of your
     files, compress your files (see 'gzip' or 'bzip2' for powerful and
     easy-to-use compression programs), or talk with us about increasing
     your disk allocation.

     Thanks for your cooperation in this matter.

     Your afriendly neighborhood sysadmin
     EOF

     echo "Account $account has $usage MB of disk space. User notified."

   done < $violators

   exit 0
```

*列表 5-3：*`*diskhogs*`*脚本*

#### *它是如何工作的*

该脚本使用脚本 #35 作为基础，变更标记在➊、➋、➍、➎和➏处。请注意在邮件管道中添加了`fmt`命令，见➏。

这个巧妙的技巧改善了自动生成的电子邮件的外观，当字段的长度未知时，例如`$account`，它被嵌入到文本中。在此脚本中，`for`循环的逻辑在第➌处稍有不同于脚本 #35 中的`for`循环：因为该脚本中循环的输出仅用于脚本的第二部分，在每个循环中，脚本只报告账户名和磁盘使用情况，而不是`磁盘配额超限`的错误信息。

#### *运行脚本*

这个脚本没有起始参数，应该以 root 身份运行以获得准确的结果。最安全的做法是使用`sudo`命令，如列表 5-4 所示。

#### *结果*

```
$ sudo diskhogs
Account ashley has 539.7MB of disk space. User notified.
Account taylor has 91799.4MB of disk space. User notified.
```

*列表 5-4：测试*`*diskhogs*`*脚本*

如果我们现在查看`ashley`账户的邮箱，我们会看到来自脚本的一条消息已被送达，见列表 5-5。

```
Subject: Warning: ashley Exceeds Quota

Your disk usage is 539.7MB, but you have been allocated only 500MB. This means
that you need to delete some of your files, compress your files (see 'gzip' or
'bzip2' for powerful and easy-to-use compression programs), or talk with us
about increasing your disk allocation.

Thanks for your cooperation in this matter.

Your friendly neighborhood sysadmin
```

*列表 5-5：因超用磁盘而向`*ashley*`用户发送的电子邮件*

#### *修改脚本*

这个脚本的一个有用的改进是允许某些用户拥有比其他用户更大的配额。这可以通过创建一个单独的文件来定义每个用户的磁盘配额来轻松实现，并在脚本中为未出现在文件中的用户设置默认配额。可以使用`grep`扫描包含账户名和配额对的文件，并通过调用`cut -f2`提取第二个字段。

### #37 改善 df 输出的可读性

`df`工具的输出可能会让人费解，但我们可以提高其可读性。列表 5-6 中的脚本将`df`报告的字节数转换为更易懂的单位。

#### *代码*

```
   #!/bin/bash

   # newdf--A friendlier version of df

   awkscript="/tmp/newdf.$$"

   trap "rm -f $awkscript" EXIT

   cat << 'EOF' > $awkscript
   function showunit(size)
➊ { mb = size / 1024; prettymb=(int(mb * 100)) / 100;
➋   gb = mb / 1024; prettygb=(int(gb * 100)) / 100;

     if ( substr(size,1,1) !~ "[0-9]" ||
          substr(size,2,1) !~ "[0-9]" ) { return size }
     else if ( mb < 1) { return size "K" }
     else if ( gb < 1) { return prettymb "M" }
     else              { return prettygb "G" }
   }

   BEGIN {
     printf "%-37s %10s %7s %7s %8s %-s\n",
           "Filesystem", "Size", "Used", "Avail", "Capacity", "Mounted"
   }

   !/Filesystem/ {

     size=showunit($2);
     used=showunit($3);
     avail=showunit($4);

     printf "%-37s %10s %7s %7s %8s %-s\n",
           $1, size, used, avail, $5, $6
   }

   EOF

➌ df -k | awk -f $awkscript

   exit 0
```

*列表 5-6：*`*newdf*`*脚本，包装*`*df*`*以便更易于使用*

#### *它是如何工作的*

该脚本的大部分工作都在`awk`脚本中完成，而且完全可以将整个脚本用`awk`编写，而不是使用 Shell，利用`system()`函数直接调用`df`。 (实际上，这个脚本是用 Perl 重写的理想候选者，但这超出了本书的范围。)

这个脚本中也有一个老派的技巧，在➊和➋处，来自于 BASIC 编程。

在处理任意精度的数字值时，一种快速限制小数点后位数的方法是将值乘以 10 的幂次，将其转换为整数（去掉小数部分），然后再除以相同的 10 的幂次：`prettymb=(int(mb * 100)) / 100;`。使用这段代码，像 7.085344324 这样的值会变得更加简洁，变成 7.08。

**注意**

*某些版本的* `*df*` *具有一个* `*-h*` *标志，提供类似于此脚本输出格式的输出。然而，正如本书中许多脚本所示，这个脚本可以让你在每个 Unix 或 Linux 系统上实现更友好、更有意义的输出，无论你使用的是什么版本的* `*df*`*。*

#### *运行脚本*

这个脚本没有参数，任何人都可以运行，包括 root 用户和普通用户。为了避免报告你不感兴趣的设备的磁盘使用情况，可以在调用 `df` 后使用 `grep -v` 来过滤。

#### *结果*

常规的 `df` 报告难以理解，正如在 清单 5-7 中所示。

```
$ df
Filesystem                        512-blocks Used      Available Capacity Mounted on
/dev/disk0s2                      935761728  628835600 306414128 68%      /
devfs                             375        375       0         100%     /dev
map -hosts                        0          0         0         100%     /net
map auto_home                     0          0         0         100%     /home
localhost:/mNhtYYw9t5GR1SlUmkgN1E 935761728  935761728 0         100%     /Volumes/MobileBackups
```

*清单 5-7：* `*df*` *的默认输出复杂且令人困惑。*

新脚本利用 `awk` 改善了可读性，并且知道如何将 512 字节的块转换为更易读的千兆字节格式，正如在 清单 5-8 中所示。

```
$ newdf
Filesystem                         Size    Used     Avail    Capacity  Mounted
/dev/disk0s2                       446.2G  299.86G  146.09G  68%       /
devfs                              187K    187K     0        100%      /dev
map -hosts                         0       0        0        100%
map auto_home                      0       0        0        100%
localhost:/mNhtYYw9t5GR1SlUmkgN1E  446.2G  446.2G   0        100%      /Volumes/MobileBackups
```

*清单 5-8：* `*newdf*` *的更易读且易理解的输出*

#### *修改脚本*

这个脚本有许多坑，最不容忽视的一点是，现在很多版本的 `df` 会包括 `inode` 使用情况，许多版本还会包括处理器内部信息，尽管这些信息实际上完全无关紧要（例如，上面例子中的两个 `map` 条目）。实际上，如果我们去除这些内容，这个脚本会更有用。因此，你可以做的第一个修改是，在脚本最后调用 `df` 时使用 `-P` 标志，以去除 `inode` 使用信息。（你也可以将其作为一个新列添加，但那样输出会变得更宽，格式也更难处理。）至于去除 `map` 数据，这个很容易用 `grep` 解决，对吧？只需在 ➊ 之后添加 `|grep -v "^map"`，你就能永远屏蔽它们。

### #38 计算可用磁盘空间

虽然 脚本 #37 简化了 `df` 输出，使其更易读和理解，但如何在系统中查看可用磁盘空间的基本问题可以通过一个 shell 脚本来解决。`df` 命令按磁盘报告磁盘使用情况，但输出可能有些令人困惑：

```
$ df
Filesystem          1K-blocks  Used     Available  Use%  Mounted on
/dev/hdb2           25695892   1871048  22519564   8%    /
/dev/hdb1           101089     6218     89652      7%    /boot
none                127744     0        127744     0%    /dev/shm
```

一个更有用的 `df` 版本会将第四列中的“可用容量”值求和，并以人类可读的格式展示总和。这是一个可以轻松通过脚本使用 `awk` 命令来完成的任务，正如在 清单 5-9 中所示。

#### *代码*

```
   #!/bin/bash

   # diskspace--Summarizes available disk space and presents it in a logical
   #   and readable fashion

   tempfile="/tmp/available.$$"
   trap "rm -f $tempfile" EXIT

   cat << 'EOF' > $tempfile
       { sum += $4 }
   END { mb = sum / 1024
         gb = mb / 1024
         printf "%.0f MB (%.2fGB) of available disk space\n", mb, gb
       }
   EOF

➊ df -k | awk -f $tempfile

   exit 0
```

*清单 5-9：* `*diskspace*` *脚本，这是一个具有更友好输出的实用封装器，替代了* `*df*`

#### *工作原理*

`diskspace` 脚本主要依赖于一个临时的 `awk` 脚本，该脚本写入到 */tmp* 目录中。这个 `awk` 脚本使用传入的数据计算剩余的总磁盘空间，并以用户友好的格式输出结果。然后，`df` 的输出通过 `awk` ➊ 被传递，`awk` 执行该脚本中的操作。当脚本执行完毕后，临时的 `awk` 脚本会由于脚本开头运行的 `trap` 命令而从 */tmp* 目录中被删除。

#### *运行脚本*

这个脚本可以作为任何用户运行，输出一个简洁的可用磁盘空间的单行摘要。

#### *结果*

对于生成先前 `df` 输出的相同系统，这个脚本的输出与 清单 5-10 中显示的类似。

```
$ diskspace
96199 MB (93.94GB) of available disk space
```

*清单 5-10：测试* `*diskspace*` *脚本*

#### *破解脚本*

如果您的系统有很多多 TB 的磁盘空间，您可以扩展此脚本，使其在需要时自动返回以太字节为单位的值。如果磁盘空间不足，看到仅剩 0.03GB 的可用磁盘空间无疑会让人沮丧——但这也是使用 脚本 #36 进行清理的一个好动力，对吧？

另一个需要考虑的问题是，了解所有设备上的可用磁盘空间是否更有用，包括那些无法扩展的分区，如 */boot*，还是仅报告用户卷的磁盘空间就足够了。如果是后者，您可以通过在 `df` 调用后 ➊ 立即调用 `grep` 来改进此脚本。使用 `grep` 结合所需的设备名称，仅包括特定设备，或者使用 `grep -v` 后跟不需要的设备名称，筛选掉您不希望包含的设备。

### #39 实现一个安全的 locate

`locate` 脚本，脚本 #19 在 第 68 页，是有用的，但存在安全问题：如果构建过程以 root 身份运行，它会构建一个包含整个系统所有文件和目录的列表，而不考虑所有者，允许用户查看他们本不允许访问的目录和文件名。构建过程可以以普通用户身份运行（如 OS X 所做的，运行 `mklocatedb` 时使用 `nobody` 用户），但这样也不对，因为您希望能够在目录树中的任何位置找到文件匹配项，无论用户 `nobody` 是否有权限访问这些特定的文件和目录。

解决这个难题的一种方法是增加 `locate` 数据库中保存的数据，使得每个条目都附带所有者、组和权限字符串。但随后 `mklocatedb` 数据库本身仍然不安全，除非 `locate` 脚本以 `setuid` 或 `setgid` 脚本运行，而出于系统安全的考虑，这种做法是应该避免的。

一种折中的方法是为每个用户单独保存一个 *.locatedb* 文件。这并不是一个坏选择，因为只有实际使用 `locate` 命令的用户才需要个人数据库。一旦调用，系统会在用户的主目录中创建一个 *.locatedb* 文件，`cron` 任务可以每日更新现有的 *.locatedb* 文件，以保持同步。在第一次运行安全的 `slocate` 脚本时，它会输出一条警告消息，提醒用户他们可能只会看到公共访问的文件的匹配结果。从第二天开始（取决于 `cron` 的计划），用户将看到个性化的结果。

#### *代码*

安全的 `locate` 需要两个脚本：数据库构建器 `mkslocatedb`（如 列表 5-11 所示）和实际的搜索工具 `slocate`（如 列表 5-12 所示）。

```
   #!/bin/bash

   # mkslocatedb--Builds the central, public locate database as user nobody
   #   and simultaneously steps through each user's home directory to find
   #   those that contain a .slocatedb file. If found, an additional, private
   #   version of the locate database will be created for that user.

   locatedb="/var/locate.db"
   slocatedb=".slocatedb"

   if [ "$(id -nu)" != "root" ] ; then
     echo "$0: Error: You must be root to run this command." >&2
     exit 1
   fi

   if [ "$(grep '^nobody:' /etc/passwd)" = "" ] ; then
     echo "$0: Error: you must have an account for user 'nobody'" >&2
     echo "to create the default slocate database." >&2
     exit 1
   fi

   cd /            # Sidestep post-su pwd permission problems.

   # First create or update the public database.
➊ su -fm nobody -c "find / -print" > $locatedb 2>/dev/null
   echo "building default slocate database (user = nobody)"
   echo ... result is $(wc -l < $locatedb) lines long.

   # Now step through the user accounts on the system to see who has
   #   a .slocatedb file in their home directory.
   for account in $(cut -d: -f1 /etc/passwd)
   do
     homedir="$(grep "^${account}:" /etc/passwd | cut -d: -f6)"

     if [ "$homedir" = "/" ] ; then
       continue # Refuse to build one for root dir.
     elif [ -e $homedir/$slocatedb ] ; then
       echo "building slocate database for user $account"
       su -m $account -c "find / -print" > $homedir/$slocatedb \
        2>/dev/null
       chmod 600 $homedir/$slocatedb
       chown $account $homedir/$slocatedb
       echo ... result is $(wc -l < $homedir/$slocatedb) lines long.
     fi
   done

   exit 0
```

*列表 5-11：* `*mkslocatedb*` *脚本*

`slocate` 脚本本身（如 列表 5-12 所示）是与 `slocate` 数据库的用户接口。

```
#!/bin/bash
# slocate--Tries to search the user's own secure locatedb database for the
#   specified pattern. If the pattern doesn't match, it means no database
#   exists, so it outputs a warning and creates one. If personal .slocatedb
#   is empty, it uses system database instead.

locatedb="/var/locate.db"
slocatedb="$HOME/.slocatedb"

if [ ! -e $slocatedb -o "$1" = "--explain" ] ; then
  cat << "EOF" >&2
Warning: Secure locate keeps a private database for each user, and your
database hasn't yet been created. Until it is (probably late tonight),
I'll just use the public locate database, which will show you all
publicly accessible matches rather than those explicitly available to
account ${USER:-$LOGNAME}.
EOF
  if [ "$1" = "--explain" ] ; then
    exit 0
  fi

  # Before we go, create a .slocatedb file so that cron will fill it
  # the next time the mkslocatedb script is run.

  touch $slocatedb      # mkslocatedb will build it next time through.
  chmod 600 $slocatedb  # Start on the right foot with permissions.

elif [ -s $slocatedb ] ; then
  locatedb=$slocatedb
else
  echo "Warning: using public database. Use \"$0 --explain\" for details." >&2
fi

if [ -z "$1" ] ; then
  echo "Usage: $0 pattern" >&2
  exit 1
fi

exec grep -i "$1" $locatedb
```

*列表 5-12：* `*slocate*` *脚本，* `*mkslocatedb*` *脚本的配套脚本*

#### *工作原理*

`mkslocatedb` 脚本的核心思想是，一个以 root 身份运行的进程可以通过使用 `su -fm *user*` ➊ 临时变更为由其他用户 ID 所拥有。然后，它可以作为该用户在每个用户的文件系统上运行 `find`，以便创建一个特定于用户的文件名数据库。然而，在这个脚本中使用 `su` 命令有一定的难度，因为默认情况下，`su` 不仅希望更改有效用户 ID，还希望导入指定账户的环境。最终的结果是在几乎所有的 Unix 系统上都会出现奇怪且令人困惑的错误信息，除非指定了 `-m` 标志，这可以防止导入用户环境。`-f` 标志是额外的保障，绕过任何 `csh` 或 `tcsh` 用户的 *.cshrc* 文件。

另一个不寻常的符号 ➊ 是 `2>/dev/null,`，它将所有错误信息直接重定向到所谓的“位桶”：任何重定向到 */dev/null* 的内容都会无声无息地消失。这是一种跳过每次调用 `find` 函数时不可避免的 `permission denied` 错误信息的简便方法。

#### *运行脚本*

`mkslocatedb` 脚本的特殊之处在于，它不仅必须以 root 身份运行，而且使用 `sudo` 是不够的。你需要以 root 用户身份登录，或者使用更强大的 `su` 命令来成为 root，才能运行该脚本。这是因为 `su` 实际上会将你切换为 root 用户以运行脚本，而 `sudo` 只是简单地赋予当前用户 root 权限。`sudo` 可能会导致文件上的权限与 `su` 不同。当然，`slocate` 脚本没有这样的要求。

#### *结果*

在 Linux 系统上为 `nobody`（公共数据库）和用户 `taylor` 构建 `slocate` 数据库时，输出结果如 列表 5-13 所示。

```
# mkslocatedb
building default slocate database (user = nobody)
... result is 99809 lines long.
building slocate database for user taylor
... result is 99808 lines long.
```

*列表 5-13：* 以 root 身份运行 `*mkslocatedb*` *脚本*

要查找匹配给定模式的特定文件或文件集，首先让我们以` tintin `用户身份尝试（该用户没有*.slocatedb*文件）：

```
tintin $ slocate Taylor-Self-Assess.doc
Warning: using public database. Use "slocate --explain" for details.
$
```

现在我们将以` taylor `用户身份输入相同的命令，该用户拥有要查找的文件：

```
taylor $ slocate Taylor-Self-Assess.doc
/Users/taylor/Documents/Merrick/Taylor-Self-Assess.doc
```

#### *破解脚本*

如果你有一个非常大的文件系统，这种方法可能会占用相当数量的空间。解决此问题的一种方法是确保单个*.slocatedb*数据库文件不包含在中央数据库中也出现的条目。这需要更多的前期处理（将两者都`sort`并使用`diff`，或在搜索单个用户文件时直接跳过*/usr*和*/bin*），但从节省空间的角度来看，这可能会有所收获。

另一种节省空间的技巧是构建仅包含自上次更新以来已访问文件引用的单个*.slocatedb*文件。如果`mkslocatedb`脚本每周运行一次而不是每天运行，这种方法效果更好；否则，每到周一，所有用户都会回到原点，因为他们不太可能在周末运行`slocate`命令。

最后，另一个节省空间的简单方法是将*.slocatedb*文件压缩，并在通过`slocate`进行搜索时即时解压。有关如何实现这一点的灵感，请参见脚本 #33 中的`zgrep`命令，以及第 109 页的说明。

### #40 将用户添加到系统

如果你负责管理 Unix 或 Linux 系统的网络，你已经体验到不同操作系统之间微妙的不兼容性所带来的沮丧。一些最基本的管理任务在不同的 Unix 版本中被证明是最不兼容的，其中最为重要的任务就是用户帐户管理。与其让所有 Linux 版本的命令行界面保持 100%的一致性，每个厂商都开发了自己的图形界面，以处理其系统的特殊性。

简单网络管理协议（SNMP）本应帮助规范此类问题，但管理用户帐户现在依然像十年前一样困难，特别是在异构计算环境中。因此，对于系统管理员而言，一套非常有用的脚本包括可以根据特定需求自定义的`adduser`、`suspenduser`和`deleteuser`版本，并且可以轻松地移植到所有 Unix 系统。我们将在这里展示`adduser`，并将在接下来的两个脚本中介绍`suspenduser`和`deleteuser`。

**注意**

*OS X 是这一规则的例外，它依赖于单独的用户帐户数据库。为了保持理智，直接使用 Mac 版本的这些命令，不要试图弄清楚它们给予管理用户的那些复杂命令行访问。*

在 Linux 系统中，账户是通过向 */etc/passwd* 文件中添加一个唯一条目来创建的，该条目包括一个 1 到 8 个字符的账户名、一个唯一的用户 ID、一个组 ID、一个主目录和该用户的登录 Shell。现代系统将加密的密码值存储在 */etc/shadow* 中，因此也必须向该文件添加一个新的用户条目。最后，账户需要列出在 */etc/group* 文件中，用户可以是他们自己的组（这是此脚本中实现的策略），也可以是现有组的一部分。Listing 5-14 展示了我们如何完成所有这些步骤。

#### *代码*

```
   #!/bin/bash

   # adduser--Adds a new user to the system, including building their
   #   home directory, copying in default config data, etc.
   #   For a standard Unix/Linux system, not OS X.

   pwfile="/etc/passwd"
   shadowfile="/etc/shadow"
   gfile="/etc/group"
   hdir="/home"

   if [ "$(id -un)" != "root" ] ; then
     echo "Error: You must be root to run this command." >&2
     exit 1
   fi

   echo "Add new user account to $(hostname)"
   /bin/echo -n "login: "     ; read login

   # The next line sets the highest possible user ID value at 5000,
   #   but you should adjust this number to match the top end
   #   of your user ID range.
➊ uid="$(awk -F: '{ if (big < $3 && $3 < 5000) big=$3 } END { print big + 1 }'\
          $pwfile)"
   homedir=$hdir/$login

   # We are giving each user their own group.
   gid=$uid

   /bin/echo -n "full name: " ; read fullname
   /bin/echo -n "shell: "     ; read shell

   echo "Setting up account $login for $fullname..."

   echo ${login}:x:${uid}:${gid}:${fullname}:${homedir}:$shell >> $pwfile
   echo ${login}:*:11647:0:99999:7::: >> $shadowfile

   echo "${login}:x:${gid}:$login" >> $gfile

   mkdir $homedir
   cp -R /etc/skel/.[a-zA-Z]* $homedir
   chmod 755 $homedir
   chown -R ${login}:${login} $homedir

   # Setting an initial password
   aexec passwd $login
```

*Listing 5-14：* `*adduser*` *脚本*

#### *它是如何工作的*

这个脚本中最酷的单行代码位于 ➊。它扫描 */etc/passwd* 文件，找出当前正在使用的最大用户 ID，该 ID 小于允许的最大用户账户值（此脚本使用的是 5000，但你应该根据自己的配置进行调整），然后在其基础上加 1，作为新账户的用户 ID。这可以避免管理员记住下一个可用的 ID，同时，在用户社区发展和变化的过程中，它还可以提供高度一致的账户信息。

该脚本使用这个用户 ID 创建一个账户。然后，它会创建该账户的主目录，并将 */etc/skel* 目录中的内容复制到该目录中。按惯例，*/etc/skel* 目录中存放着主 *.cshrc*、*.login*、*.bashrc* 和 *.profile* 文件，如果站点上有提供 `~account` 服务的 Web 服务器，还会将像 */etc/skel/public_html* 这样的目录复制到新的主目录中。如果你的组织为工程师或开发人员配置 Linux 工作站或账户并带有特定的 bash 配置，这非常有用。

#### *运行脚本*

该脚本必须由 root 用户运行，并且没有起始参数。

#### *结果*

我们的系统已经有了一个名为 `tintin` 的账户，因此我们还将确保 `snowy`^(1) 也有自己的账户（见 Listing 5-15）。

```
$ sudo adduser
Add new user account to aurora
login: snowy
full name: Snowy the Dog
shell: /bin/bash
Setting up account snowy for Snowy the Dog...
Changing password for user snowy.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

*Listing 5-15：测试* `*adduser*` *脚本*

#### *破解脚本*

使用自定义 `adduser` 脚本的一个显著优势是，你可以添加代码并更改某些操作的逻辑，而不必担心操作系统升级时覆盖这些修改。可能的修改包括自动发送欢迎电子邮件，概述使用指南和在线帮助选项；自动打印出账户信息表，并将其分发给用户；在邮件 *aliases* 文件中添加 `firstname_lastname` 或 `firstname.lastname` 别名；甚至将一组文件复制到账户中，使得账户所有者可以立即开始进行团队项目。

### #41 挂起用户账户

无论是因为工业间谍行为被护送出门，学生放暑假，还是承包商暂时休假，有时禁用账户而不删除它是非常有用的。

这可以通过将用户的密码更改为一个他们不知道的新值来简单完成，但如果用户此时已经登录，那么还需要确保将其登出，并关闭系统中其他账户对该主目录的访问权限。当账户被暂停时，很有可能该用户需要立即离开系统——而不是等到他们自己觉得合适的时候。

Listing 5-16 中的大部分脚本都集中在确定用户是否已登录，通知用户他们即将被登出，并将用户踢出系统。

#### *代码*

```
   #!/bin/bash

   # suspenduser--Suspends a user account for the indefinite future

   homedir="/home"         # Home directory for users
   secs=10                 # Seconds before user is logged out

   if [ -z $1 ] ; then
     echo "Usage: $0 account" >&2
     exit 1
   elif [ "$(id -un)" != "root" ] ; then
     echo "Error. You must be 'root' to run this command." >&2
     exit 1
   fi

   echo "Please change the password for account $1 to something new."
   passwd $1

   # Now let's see if they're logged in and, if so, boot 'em.
   if who|grep "$1" > /dev/null ; then

     for tty in $(who | grep $1 | awk '{print $2}'); do

       cat << "EOF" > /dev/$tty

   ******************************************************************************
   URGENT NOTICE FROM THE ADMINISTRATOR:

   This account is being suspended, and you are going to be logged out
   in $secs seconds. Please immediately shut down any processes you
   have running and log out.

   If you have any questions, please contact your supervisor or
   John Doe, Director of Information Technology.
   ******************************************************************************
   EOF
     done

     echo "(Warned $1, now sleeping $secs seconds)"

     sleep $secs

     jobs=$(ps -u $1 | cut -d\ -f1)

➊   kill -s HUP $jobs                  # Send hangup sig to their processes.
     sleep 1                            # Give it a second...
➋   kill -s KILL $jobs > /dev/null 2>1 # and kill anything left.

     echo "$1 was logged in. Just logged them out."
   fi

   # Finally, let's close off their home directory from prying eyes.
   chmod 000 $homedir/$1

   echo "Account $1 has been suspended."

   exit 0
```

*Listing 5-16: `*suspenduser*` 脚本*

#### *工作原理*

该脚本将用户的密码更改为一个用户不知道的值，然后关闭用户的主目录。如果用户已登录，我们会给出几秒钟的警告，然后通过终止他们所有正在运行的进程将用户登出。

请注意，脚本如何向每个正在运行的进程发送`SIGHUP`（`HUP`）挂起信号 ➊，然后等待一秒钟，再发送更具攻击性的`SIGKILL`（`KILL`）信号 ➋。`SIGHUP`信号会退出正在运行的应用程序——但并非*总是如此*，它不会杀死登录的 shell。然而，`SIGKILL`信号无法被忽视或阻止，因此它能确保 100%有效。尽管如此，它并不是首选方法，因为它不给应用程序任何时间来清理临时文件，刷新文件缓冲区以确保更改写入磁盘，等等。

取消暂停用户是一个简单的两步过程：首先通过`chmod 700`重新打开用户的主目录，然后通过`passwd`将密码重置为已知的值。

#### *运行脚本*

此脚本必须以 root 身份运行，并且有一个参数：要暂停的账户名称。

#### *结果*

结果证明，`snowy`已经在滥用他的账户。让我们按照 Listing 5-17 中的方式暂停他的账户。

```
$ sudo suspenduser snowy
Please change the password for account snowy to something new.
Changing password for user snowy.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
(Warned snowy, now sleeping 10 seconds)
snowy was logged in. Just logged them out.
Account snowy has been suspended.
```

*Listing 5-17: 在用户`*snowy*`上测试`*suspenduser*`脚本*

由于`snowy`当时已登录，Listing 5-18 展示了他在被踢出系统前几秒钟看到的屏幕内容。

```
******************************************************************************
URGENT NOTICE FROM THE ADMINISTRATOR:

This account is being suspended, and you are going to be logged out
in 10 seconds. Please immediately shut down any processes you
have running and log out.

If you have any questions, please contact your supervisor or
John Doe, Director of Information Technology.
******************************************************************************
```

*Listing 5-18: 用户被暂停前显示的警告信息*

### #42 删除用户账户

删除账户比暂停账户稍微复杂一些，因为脚本需要在从*/etc/passwd*和*/etc/shadow*中移除账户信息之前，检查整个文件系统中是否有该用户拥有的文件。Listing 5-19 确保用户及其数据被完全从系统中删除。它假设之前的`suspenduser`脚本已存在于当前的`PATH`中。

#### *代码*

```
   #!/bin/bash

   # deleteuser--Deletes a user account without a trace.
   #   Not for use with OS X.

   homedir="/home"
   pwfile="/etc/passwd"
   shadow="/etc/shadow"
   newpwfile="/etc/passwd.new"
   newshadow="/etc/shadow.new"
   suspend="$(which suspenduser)"
   locker="/etc/passwd.lock"

   if [ -z $1 ] ; then
     echo "Usage: $0 account" >&2
     exit 1
   elif [ "$(whoami)" != "root" ] ; then
     echo "Error: you must be 'root' to run this command.">&2
     exit 1
   fi

   $suspend $1    # Suspend their account while we do the dirty work.

   uid="$(grep -E "^${1}:" $pwfile | cut -d: -f3)"

   if [ -z $uid ] ; then
     echo "Error: no account $1 found in $pwfile" >&2
     exit 1
   fi

   # Remove the user from the password and shadow files.
   grep -vE "^${1}:" $pwfile > $newpwfile
   grep -vE "^${1}:" $shadow > $newshadow

   lockcmd="$(which lockfile)"             # Find lockfile app in the path.
➊ if [ ! -z $lockcmd ] ; then             # Let's use the system lockfile.
     eval $lockcmd -r 15 $locker
   else                                    # Ulp, let's do it ourselves.
➋   while [ -e $locker ] ; do
       echo "waiting for the password file" ; sleep 1
     done
➌   touch $locker                         # Create a file-based lock.
   fi

   mv $newpwfile $pwfile
   mv $newshadow $shadow
➍ rm -f $locker                           # Click! Unlocked again.

   chmod 644 $pwfile
   chmod 400 $shadow

   # Now remove home directory and list anything left.
   rm -rf $homedir/$1

   echo "Files still left to remove (if any):"
   find / -uid $uid -print 2>/dev/null | sed 's/^/ /'

   echo ""
   echo "Account $1 (uid $uid) has been deleted, and their home directory "
   echo "($homedir/$1) has been removed."

   exit 0
```

*Listing 5-19: `*deleteuser*` 脚本*

#### *工作原理*

为了避免脚本运行时目标用户账户的任何更改，`deleteuser`执行的第一个任务是通过调用`suspenduser`来挂起用户账户。

在修改密码文件之前，如果`lockfile`程序可用，脚本会先使用它锁定文件 ➊。或者，在 Linux 上，你也可以考虑使用`flock`工具来创建文件锁。如果没有，脚本会退回到一个相对原始的信号量锁定机制，通过创建文件*/etc/passwd.lock*来实现。如果锁文件已经存在 ➋，该脚本将等待另一个程序删除它；一旦它被删除，`deleteuser`会立即创建该锁文件并继续执行 ➌，执行完成后再删除它 ➍。

#### *运行脚本*

该脚本必须以 root 身份运行（使用`sudo`），并且需要提供要删除的账户名作为命令参数。清单 5-20 展示了脚本在用户`snowy`上运行的示例。

**警告**

*这个脚本是不可逆的，且会导致许多文件消失，因此如果你想实验它，请小心！*

#### *结果*

```
$ sudo deleteuser snowy
Please change the password for account snowy to something new.
Changing password for user snowy.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
Account snowy has been suspended.
Files still left to remove (if any):
  /var/log/dogbone.avi

Account snowy (uid 502) has been deleted, and their home directory
(/home/snowy) has been removed.
```

*清单 5-20：测试* `*deleteuser*` *脚本，目标用户为* `*snowy*`

那个狡猾的`snowy`在*/var/log*中隐藏了一个 AVI 文件（*dogbone.avi*）。幸运的是我们发现了它——谁知道它是什么呢？

#### *黑客脚本*

这个`deleteuser`脚本故意不完整。你应该决定采取什么额外的步骤——无论是压缩并归档账户文件的最终副本，将其写入磁带，备份到云服务，刻录到 DVD-ROM，还是直接邮寄给 FBI（希望我们在最后一点开玩笑）。此外，还需要从*/etc/group*文件中删除该账户。如果有用户主目录之外的孤立文件，`find`命令会帮助找到它们，但仍然需要系统管理员检查并根据情况删除每一个文件。

这个脚本的一个有用补充是干运行模式，它可以让你在实际删除用户之前，先查看该脚本将从系统中删除哪些内容。

### #43 验证用户环境

由于人们会将登录、配置文件以及其他 Shell 环境的定制从一个系统迁移到另一个系统，因此这些设置逐渐衰退是常见现象；最终，`PATH`可能包括一些系统上不存在的目录，`PAGER`可能指向一个不存在的二进制文件，等等。

解决这个问题的高级方案是，首先检查`PATH`，确保它只包含系统上有效的目录，然后检查每个关键帮助程序设置，确保它们要么指向一个存在的完全限定文件，要么指定一个在`PATH`中的二进制文件。这个过程在清单 5-21 中有详细说明。

#### *代码*

```
   #!/bin/bash
   # validator--Ensures that the PATH contains only valid directories
   #   and then checks that all environment variables are valid.
   #   Looks at SHELL, HOME, PATH, EDITOR, MAIL, and PAGER.

   errors=0

➊ source library.sh   # This contains Script #1, the in_path() function.

➋ validate()
   {
     varname=$1
     varvalue=$2

     if [ ! -z $varvalue ] ; then
➌     if [ "${varvalue%${varvalue#?}}" = "/" ] ; then
         if [ ! -x $varvalue ] ; then
           echo "** $varname set to $varvalue, but I cannot find executable."
           (( errors++ ))
         fi
       else
         if in_path $varvalue $PATH ; then
           echo "** $varname set to $varvalue, but I cannot find it in PATH."
           errors=$(( $errors + 1 ))
         fi
       fi
     fi
   }

   # BEGIN MAIN SCRIPT
   # =================

➍ if [ ! -x ${SHELL:?"Cannot proceed without SHELL being defined."} ] ; then
     echo "** SHELL set to $SHELL, but I cannot find that executable."
     errors=$(( $errors + 1 ))
   fi
   if [ ! -d ${HOME:?"You need to have your HOME set to your home directory"} ]
   then
     echo "** HOME set to $HOME, but it's not a directory."
     errors=$(( $errors + 1 ))
   fi

   # Our first interesting test: Are all the paths in PATH valid?

➎ oldIFS=$IFS; IFS=":"     # IFS is the field separator. We'll change to ':'.

➏ for directory in $PATH
   do
     if [ ! -d $directory ] ; then
       echo "** PATH contains invalid directory $directory."
       errors=$(( $errors + 1 ))
     fi
   done

   IFS=$oldIFS             # Restore value for rest of script.

   # The following variables should each be a fully qualified path,
   #   but they may be either undefined or a progname. Add additional
   #   variables as necessary for your site and user community.

   validate "EDITOR" $EDITOR
   validate "MAILER" $MAILER
   validate "PAGER"  $PAGER

   # And, finally, a different ending depending on whether errors > 0

   if [ $errors -gt 0 ] ; then
     echo "Errors encountered. Please notify sysadmin for help."
   else
     echo "Your environment checks out fine."
   fi

   exit 0
```

*清单 5-21：* `*validator*` *脚本*

#### *工作原理*

这个脚本执行的测试并不复杂。为了检查 `PATH` 中的所有目录是否有效，代码会逐个检查每个目录，确保它存在 ➏。注意，在 ➎ 处需要将内部字段分隔符（`IFS`）更改为冒号，这样脚本才能正确地逐个检查所有的 `PATH` 目录。按照惯例，`PATH` 变量使用冒号来分隔每个目录：

```
$ echo $PATH
/bin/:/sbin:/usr/bin:/sw/bin:/usr/X11R6/bin:/usr/local/mybin
```

为了验证环境变量值的有效性，`validate()` 函数 ➋ 首先检查每个值是否以 `/` 开头。如果是，它将检查该变量是否可执行。如果不是以 `/` 开头，脚本会调用我们从库中引入的 `in_path()` 函数（在 脚本 #1 的 第 11 页 中有提到） ➊ 来检查该程序是否能在当前 `PATH` 中的某个目录下找到。

这个脚本最不寻常的地方是它在某些条件语句中使用默认值以及变量切片。它在条件语句中使用默认值的例子可以在 ➍ 处看到。符号 `${*varname*:?"*errorMessage*"}` 可以解释为：“如果 `*varname*` 存在，则替换它的值；否则，返回错误信息 `*errorMessage*`。”

在 ➌ 处使用的变量切片符号 `${varvalue%${varvalue#?}}` 是 POSIX 子字符串函数，它只提取变量 `varvalue` 的第一个字符。在这个脚本中，它用于判断一个环境变量是否拥有一个完全限定的文件名（即以 `/` 开头并指定二进制文件路径的文件名）。

如果你使用的 Unix/Linux 版本不支持这些符号，它们可以通过简单的方式替代。例如，代替 `${SHELL:?No Shell}`，你可以使用以下代码：

```
if [ -z "$SHELL" ] ; then
  echo "No Shell" >&2; exit 1
fi
```

如果你不想使用 `{varvalue%${varvalue#?}}`，可以用以下代码来实现相同的结果：

```
$(echo $varvalue | cut -c1)
```

#### *运行脚本*

这是用户可以运行的代码来检查自己的环境。如 列表 5-22 所示，脚本没有传入任何参数。

#### *结果*

```
$ validator
** PATH contains invalid directory /usr/local/mybin.
** MAILER set to /usr/local/bin/elm, but I cannot find executable.
Errors encountered. Please notify sysadmin for help.
```

*列表 5-22：测试`*validator*`脚本*

### #44 客人离开后的清理工作

尽管许多网站出于安全原因禁用了 `guest` 用户，其他网站仍然有访客账户（通常设置了一个容易猜到的密码），以便让客户或其他部门的人访问网络。这个账户很有用，但也有一个大问题：多个用户共享同一个账户，容易导致下一个用户使用时遇到麻烦——也许他们在试验命令、编辑 *.rc* 文件、添加子目录，等等。

清单 5-23 中的这个脚本通过在每次用户注销访客账户时清理账户空间来解决问题。它删除任何新创建的文件或子目录，移除所有点文件，并重建官方账户文件，这些文件的副本存储在访客账户的*.template*目录中的只读存档里。

#### *代码*

```
#!/bin/bash

# fixguest--Cleans up the guest account during the logout process

# Don't trust environment variables: reference read-only sources.

iam=$(id -un)
myhome="$(grep "^${iam}:" /etc/passwd | cut -d: -f6)"

# *** Do NOT run this script on a regular user account!

if [ "$iam" != "guest" ] ; then
  echo "Error: you really don't want to run fixguest on this account." >&2
  exit 1
fi

if [ ! -d $myhome/..template ] ; then
  echo "$0: no template directory found for rebuilding." >&2
  exit 1
fi

# Remove all files and directories in the home account.

cd $myhome

rm -rf * $(find . -name ".[a-zA-Z0-9]*" -print)

# Now the only thing present should be the ..template directory.

cp -Rp ..template/* .
exit 0
```

*清单 5-23：* `*fixguest*` *脚本*

#### *工作原理*

为了确保此脚本正确运行，你需要在访客主目录中创建一个模板文件和目录的主集，并将其放入一个名为*..template*的新目录中。将*..template*目录的权限设置为只读，并确保*..template*目录中的所有文件和目录对用户`guest`具有正确的所有权和权限。

#### *运行脚本*

一个合理的执行`fixguest`脚本的时间是在注销时，可以在*.logout*文件中调用它（这适用于大多数 shell，但并非全部）。此外，如果`login`脚本输出如下信息，肯定会为你节省很多来自用户的投诉：

```
Notice: All files are purged from the guest account immediately
upon logout, so please don't save anything here you need. If you
want to save something, email it to your main account instead.
You've been warned!
```

然而，由于一些访客用户可能足够聪明，能够修改*.logout*文件，因此值得通过`cron`调用`fixguest`脚本。只要确保在脚本运行时没有人登录该账户！

#### *结果*

运行此程序没有明显的结果，除了`guest`主目录恢复为与*..template*目录中的布局和文件相一致的状态。
