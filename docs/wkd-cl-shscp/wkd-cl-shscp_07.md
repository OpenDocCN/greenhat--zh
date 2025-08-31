## 第七章：**系统管理：系统维护**

![image](img/common4.jpg)

shell 脚本最常见的用途是帮助 Unix 或 Linux 系统管理。对此当然有明显的原因：管理员通常是系统中最了解的人，他们也负责确保系统平稳运行。但系统管理领域中强调 shell 脚本的原因可能还有另一个。我们的猜测是：系统管理员和其他高级用户是最有可能享受与系统互动的人，而在 Unix 环境下开发 shell 脚本非常有趣！

接下来，让我们继续探索 shell 脚本如何帮助你进行系统管理任务。

### #45 跟踪设置用户 ID 应用程序

不论黑客是否拥有账户，都有很多方式可以侵入 Linux 系统，其中最简单的一种是找到没有正确保护的 `setuid` 或 `setgid` 命令。如前几章所讨论的，这些命令会改变它们调用的任何子命令的有效用户身份，具体由配置文件指定，因此普通用户可能会运行一个脚本，其中脚本中的命令以 root 或超级用户身份执行。糟糕。危险！

例如，在一个 `setuid` shell 脚本中，添加以下代码可以为坏人创建一个 `setuid` root shell，一旦代码被一个不知情的管理员作为 root 登录时触发。

```
if [ "${USER:-$LOGNAME}" = "root" ] ; then # REMOVEME
  cp /bin/sh /tmp/.rootshell               # REMOVEME
  chown root /tmp/.rootshell               # REMOVEME
  chmod -f 4777 /tmp/.rootshell            # REMOVEME
  grep -v "# REMOVEME" $0 > /tmp/junk      # REMOVEME
  mv /tmp/junk  $0                         # REMOVEME
fi # REMOVEME
```

一旦这个脚本被 root 不小心运行，一份 */bin/sh* 会偷偷地复制到 */tmp* 目录，并命名为 *.rootshell*，并且被设置为 `setuid` root，供黑客随意利用。然后，脚本会导致自己被重写，删除条件代码（因此每行末尾有 `# REMOVEME`），几乎没有留下黑客所做的痕迹。

如上所示的代码片段也可以在任何以 root 身份有效用户 ID 运行的脚本或命令中被利用；因此，确保你了解并批准系统中所有 `setuid` root 命令的必要性是至关重要的。当然，出于这个原因，你绝不应该让脚本拥有任何形式的 `setuid` 或 `setgid` 权限，但保持警觉始终是明智的。

然而，比起向你展示如何破解系统，更有用的是展示如何识别系统中所有标记为 `setuid` 或 `setgid` 的 shell 脚本！清单 6-1 详细说明了如何实现这一点。

#### *代码*

```
   #!/bin/bash

   # findsuid--Checks all SUID files or programs to see if they're writeable,
   #   and outputs the matches in a friendly and useful format

   mtime="7"            # How far back (in days) to check for modified cmds.
   verbose=0            # By default, let's be quiet about things.

   if [ "$1" = "-v" ] ; then
     verbose=1          # User specified findsuid -v, so let's be verbose.
   fi

   # find -perm looks at the permissions of the file: 4000 and above
   #   are setuid/setgid.

➊ find / -type f -perm +4000 -print0 | while read -d '' -r match
   do
     if [ -x "$match" ] ; then

       # Let's split file owner and permissions from the ls -ld output.

       owner="$(ls -ld $match | awk '{print $3}')"
       perms="$(ls -ld $match | cut -c5-10 | grep 'w')"

       if [ ! -z $perms ] ; then
         echo "**** $match (writeable and setuid $owner)"
       elif [ ! -z $(find $match -mtime -$mtime -print) ] ; then
         echo "**** $match (modified within $mtime days and setuid $owner)"
       elif [ $verbose -eq 1 ] ; then
         # By default, only dangerous scripts are listed. If verbose, show all.
         lastmod="$(ls -ld $match | awk '{print $6, $7, $8}')"
         echo "     $match (setuid $owner, last modified $lastmod)"
       fi
     fi
   done

   exit 0
```

*清单 6-1：* `*findsuid*` *脚本*

#### *原理*

这个脚本检查系统中所有 `setuid` 命令，查看它们是否对组或全体用户可写，并且检查它们是否在过去的 `$mtime` 天内被修改。为此，我们使用 `find` 命令 ➊，并指定搜索文件权限类型的参数。如果用户请求详细输出，所有具有 `setuid` 权限的脚本将被列出，无论其读/写权限和修改日期如何。

#### *运行脚本*

这个脚本有一个可选参数：`-v` 会生成详细输出，列出脚本遇到的每个 `setuid` 程序。这个脚本应该以 root 身份运行，但任何用户都可以运行，因为每个人都应该对关键目录有基本访问权限。

#### *结果*

我们在系统中某个地方放置了一个漏洞脚本。让我们看看 `findsuid` 能否在列表 6-2 中找到它。

```
$ findsuid
**** /var/tmp/.sneaky/editme (writeable and setuid root)
```

*列表 6-2：运行* `*findsuid*` *shell 脚本并找到反向门 shell 脚本*

就在这里（列表 6-3）！

```
$ ls -l /var/tmp/.sneaky/editme
-rwsrwxrwx  1 root  wheel  25988 Jul 13 11:50 /var/tmp/.sneaky/editme
```

*列表 6-3：* `*ls*` *反向门的输出，显示权限中的* `*s*`，这意味着它是* `*setuid*`*

这是一个巨大的漏洞，随时等待有人来利用。很高兴我们找到了它！

### #46 设置系统日期

简洁性是 Linux 及其 Unix 前身的核心，这一特点对 Linux 的发展产生了深远影响。但在某些领域，这种简洁性会让系统管理员抓狂。最常见的烦恼之一就是重置系统日期所需的格式，如`date`命令所示：

```
usage: date [[[[[cc]yy]mm]dd]hh]mm[.ss]
```

尝试弄清楚所有方括号可能让人困惑，更不用说你需要或不需要指定什么了。我们来解释一下：你可以只输入分钟；或者分钟和秒；或者小时、分钟和秒；或者加上月份再加所有这些——或者你也可以加上年份，甚至是世纪。是的，疯狂！与其费劲去弄清楚这些，不如使用像列表 6-4 中的脚本，它会提示每个相关字段的输入，然后构建压缩日期字符串。这真的是一个保持理智的好帮手。

#### *代码*

```
   #!/bin/bash
   # setdate--Friendly frontend to the date command
   # Date wants: [[[[[cc]yy]mm]dd]hh]mm[.ss]

   # To make things user-friendly, this function prompts for a specific date
   #   value, displaying the default in [] based on the current date and time.

   . library.sh   # Source our library of bash functions to get echon().

➊ askvalue()
   {
     # $1 = field name, $2 = default value, $3 = max value,
     # $4 = required char/digit length

     echon "$1 [$2] : "
     read answer

     if [ ${answer:=$2} -gt $3 ] ; then
       echo "$0: $1 $answer is invalid"
       exit 0
     elif [ "$(( $(echo $answer | wc -c) - 1 ))" -lt $4 ] ; then
       echo "$0: $1 $answer is too short: please specify $4 digits"
       exit 0
     fi
     eval $1=$answer   # Reload the given variable with the specified value.
   }

➋ eval $(date "+nyear=%Y nmon=%m nday=%d nhr=%H nmin=%M")

   askvalue year $nyear 3000 4
   askvalue month $nmon 12 2
   askvalue day $nday 31 2
   askvalue hour $nhr 24 2
   askvalue minute $nmin 59 2

   squished="$year$month$day$hour$minute"

   # Or, if you're running a Linux system:
➌ #   squished="$month$day$hour$minute$year"
   #   Yes, Linux and OS X/BSD systems use different formats. Helpful, eh?

   echo "Setting date to $squished. You might need to enter your sudo password:"
   sudo date $squished

   exit 0
```

*列表 6-4：* `*setdate*` *脚本*

#### *工作原理*

为了尽可能简洁地编写这个脚本，我们在 ➋ 使用了 `eval` 函数来完成两件事。首先，这一行使用 `date` 格式字符串设置当前的日期和时间值。其次，它设置了变量 `nyear`、`nmon`、`nday`、`nhr` 和 `nmin` 的值，这些值随后会被用于简单的 `askvalue()` 函数 ➊ 来提示和测试输入的值。通过使用 `eval` 函数为变量赋值，还可以避免在多次调用 `askvalue()` 函数之间日期发生变化或溢出的潜在问题，否则脚本中的数据将不一致。例如，如果 `askvalue` 在 23:59.59 获取了月份和日期的值，然后在 0:00:02 获取了小时和分钟的值，那么系统日期实际上会回滚 24 小时——这显然不是我们希望的结果。

我们还需要确保使用适合我们系统的正确日期格式字符串，因为例如，OS X 在设置日期时要求使用特定的格式，而 Linux 则要求使用稍微不同的格式。默认情况下，这个脚本使用的是 OS X 日期格式，但请注意，在注释中还提供了一个适用于 Linux 的格式字符串，见 ➌。

这是使用 `date` 命令时遇到的一个微妙问题。使用这个脚本时，如果你在提示过程中指定了确切的时间，但随后必须输入 `sudo` 密码，你可能会将系统时间设置为过去几秒钟的时间。这可能不会造成问题，但这也是为什么网络连接的系统应该使用网络时间协议（NTP）工具来与官方时间服务器同步的原因之一。你可以通过查阅你 Linux 或 Unix 系统上的 `timed(8)` 来开始了解网络时间同步。

#### *运行脚本*

请注意，脚本中使用 `sudo` 命令以 root 身份运行实际的日期重置操作，正如 Listing 6-5 所示。通过输入错误的 `sudo` 密码，你可以在不担心出现奇怪结果的情况下尝试这个脚本。

#### *结果*

```
$ setdate
year [2017] :
month [05] :
day [07] :
hour [16] : 14
minute [53] : 50
Setting date to 201705071450\. You might need to enter your sudo password:
passwd:
$
```

*Listing 6-5: 测试交互式 `*setdate*` 脚本*

### #47 根据名称终止进程

Linux 和一些 Unix 系统有一个有用的命令 `killall`，它允许你终止所有与指定模式匹配的运行中的应用程序。当你需要终止九个 `mingetty` 守护进程，或者仅仅是想给 `xinetd` 发送一个 `SIGHUP` 信号来促使它重新读取配置文件时，这个命令非常有用。没有 `killall` 的系统可以通过基于 `ps` 来识别匹配的进程，并通过 `kill` 发送指定信号来模拟该命令。

脚本中最棘手的部分是 `ps` 的输出格式在不同操作系统之间差异显著。例如，考虑一下 FreeBSD、Red Hat Linux 和 OS X 在默认的 `ps` 输出中如何显示运行中的进程。首先看一下 FreeBSD 的输出：

```
BSD $ ps
 PID TT  STAT    TIME COMMAND
 792  0  Ss   0:00.02 -sh (sh)
4468  0  R+   0:00.01 ps
```

将这个输出与 Red Hat Linux 的输出进行比较：

```
RHL $ ps
  PID TTY          TIME CMD
 8065 pts/4    00:00:00 bash
12619 pts/4    00:00:00 ps
```

最后，比较一下 OS X 的输出：

```
OSX $ ps
  PID TTY           TIME CMD
37055 ttys000    0:00.01 -bash
26881 ttys001    0:00.08 -bash
```

更糟糕的是，GNU 的 `ps` 命令没有像典型的 Unix 命令那样模仿 `ps`，它接受 BSD 风格的标志、SYSV 风格的标志、*以及* GNU 风格的标志。真是一个混乱的局面！

幸运的是，通过使用 `cu` 标志，一些这些不一致的问题在这个特定脚本中可以被规避，它会生成更一致的输出，包括进程的拥有者、完整的命令名称以及——我们真正感兴趣的——进程 ID。

这也是第一个我们真正使用 `getopts` 命令的脚本，它让我们可以处理许多不同的命令行选项，甚至传入可选值。 Listing 6-6 中的脚本有四个起始标志，其中三个需要参数：`-s *SIGNAL*`，`-u *USER*`，`-t *TTY*` 和 `-n`。你将在代码的第一部分看到它们。

#### *代码*

```
   #!/bin/bash

   # killall--Sends the specified signal to all processes that match a
   #   specific process name

   # By default it kills only processes owned by the same user, unless you're
   #   root. Use -s SIGNAL to specify a signal to send to the process, -u USER
   #   to specify the user, -t TTY to specify a tty, and -n to only report what
   #   should be done, rather than doing it.

   signal="-INT"      # Default signal is an interrupt.
   user=""   tty=""   donothing=0

   while getopts "s:u:t:n" opt; do
     case "$opt" in
           # Note the trick below: the actual kill command wants -SIGNAL
           #   but we want SIGNAL, so we'll just prepend the "-" below.
       s ) signal="-$OPTARG";              ;;
       u ) if [ ! -z "$tty" ] ; then
             # Logic error: you can't specify a user and a TTY device
             echo "$0: error: -u and -t are mutually exclusive." >&2
             exit 1
           fi
           user=$OPTARG;                  ;;
       t ) if [ ! -z "$user" ] ; then
              echo "$0: error: -u and -t are mutually exclusive." >&2
              exit 1
           fi
           tty=$2;                        ;;
       n ) donothing=1;                   ;;
       ? ) echo "Usage: $0 [-s signal] [-u user|-t tty] [-n] pattern" >&2
           exit 1
     esac
   done

   # Done with processing all the starting flags with getopts...
   shift $(( $OPTIND - 1 ))

   # If the user doesn't specify any starting arguments (earlier test is for -?)
   if [ $# -eq 0 ] ; then
     echo "Usage: $0 [-s signal] [-u user|-t tty] [-n] pattern" >&2
     exit 1
   fi

   # Now we need to generate a list of matching process IDs, either based on
   #   the specified TTY device, the specified user, or the current user.

   if [ ! -z "$tty" ] ; then
➊   pids=$(ps cu -t $tty | awk "/ $1$/ { print \$2 }")
   elif [ ! -z "$user" ] ; then
➋   pids=$(ps cu -U $user | awk "/ $1$/ { print \$2 }")
   else
➌   pids=$(ps cu -U ${USER:-LOGNAME} | awk "/ $1$/ { print \$2 }")
   fi

   # No matches? That was easy!
   if [ -z "$pids" ] ; then
     echo "$0: no processes match pattern $1" >&2
     exit 1
   fi

   for pid in $pids
   do
     # Sending signal $signal to process id $pid: kill might still complain
     #   if the process has finished, the user doesn't have permission to kill
     #   the specific process, etc., but that's okay. Our job, at least, is done.
     if [ $donothing -eq 1 ] ; then
       echo "kill $signal $pid" # The -n flag: "show me, but don't do it"
     else
       kill $signal $pid
     fi
   done

   exit 0
```

*Listing 6-6: `*killall*` 脚本*

#### *工作原理*

由于这个脚本非常强大且潜在危险，我们做了额外的努力来最小化错误的模式匹配，以防像`sh`这样的模式匹配到`ps`输出中的`bash`或`vi crashtest.c`等值。这是通过在`awk`命令上加上模式匹配前缀来实现的（➊，➋，➌）。

*左根模式* `$1`，前面加一个空格并且*右根模式* 后面加上`$`，使得脚本能够在`ps`输出中将指定模式`'sh'`匹配为`' sh$'`。

#### *运行脚本*

这个脚本有多种启动标志，可以让你修改它的行为。`-s *SIGNAL*`标志允许你指定一个不同于默认中断信号`SIGINT`的信号，发送到匹配的进程。`-u *USER*`和`-t *TTY*`标志主要对 root 用户有用，分别用于杀死与指定用户或 TTY 设备相关的所有进程。而`-n`标志则让你选择是否仅报告脚本将要执行的操作，而不实际发送任何信号。最后，必须指定一个进程名称模式。

#### *结果*

要在 OS X 上杀死所有`csmount`进程，你现在可以使用`killall`脚本，如清单 6-7 所示。

```
$ ./killall -n csmount
kill -INT 1292
kill -INT 1296
kill -INT 1306
kill -INT 1310
kill -INT 1318
```

*清单 6-7：在任何`*csmount*`进程上运行`*killall*`脚本*

#### *破解脚本*

虽然不太可能，但在运行这个脚本时可能会出现一个不太可能的错误。为了仅匹配指定的模式，`awk`调用会输出匹配该模式的进程 ID，并且输入行的末尾会有一个前置空格。但理论上有可能有两个进程同时运行——例如，一个叫`bash`，另一个叫`emulate bash`。如果用`bash`作为模式调用`killall`，这两个进程都会被匹配，尽管只有前者是真正的匹配。解决这个问题并确保跨平台的一致性结果会非常棘手。

如果你有动力，你还可以编写一个基于`killall`脚本的脚本，允许你通过名称而不仅仅是通过进程 ID 来`renice`任务。唯一需要更改的就是调用`renice`而不是`kill`。调用`renice`可以让你改变程序的相对优先级，比如，你可以降低长时间文件传输的优先级，同时提高老板正在运行的视频编辑器的优先级。

### #48 验证用户 crontab 条目

在 Linux 世界中，最有用的工具之一就是`cron`，它能够在将来的任意时间安排任务，或者让任务每分钟、每几个小时、每月甚至每年自动运行。每个优秀的系统管理员都有一把来自`crontab`文件的瑞士军刀式的脚本。

然而，输入`cron`规范的格式有点复杂，而`cron`字段具有数字值、范围、集合，甚至是星期几或月份的助记名称。更糟糕的是，当`crontab`程序遇到用户或系统`cron`文件中的问题时，会生成难以理解的错误信息。

例如，如果你指定一个有拼写错误的星期几，`crontab`会报告类似下面的错误：

```
"/tmp/crontab.Dj7Tr4vw6R":9: bad day-of-week
crontab: errors in crontab file, can't install
```

实际上，示例输入文件的第 12 行存在第二个错误，但由于`crontab`的错误检查代码很差，它将迫使我们在脚本中采取较长的方式来查找该错误。

与其按照`crontab`要求的方式进行错误检查，不如使用一个稍长的 Shell 脚本（见列表 6-8），它可以逐步检查`crontab`文件，检查语法并确保值在合理的范围内。这种验证能够在 Shell 脚本中实现的原因之一是，集合和范围可以作为单独的值来处理。因此，要测试`3-11`或`4`、`6`和`9`是否是某个字段的有效值，只需要测试前者的`3`和`11`，以及后者的`4`、`6`和`9`。

#### *代码*

```
   #!/bin/bash
   # verifycron--Checks a crontab file to ensure that it's formatted properly.
   #   Expects standard cron notation of min hr dom mon dow CMD, where min is
   #   0-59, hr is 0-23, dom is 1-31, mon is 1-12 (or names), and dow is 0-7
   #   (or names). Fields can be ranges (a-e) or lists separated by commas
   #   (a,c,z) or an asterisk. Note that the step value notation of Vixie cron
   #   (e.g., 2-6/2) is not supported by this script in its current version.

   validNum()
   {
     # Return 0 if the number given is a valid integer and 1 if not.
     #   Specify both number and maxvalue as args to the function.
     num=$1   max=$2

 # Asterisk values in fields are rewritten as "X" for simplicity,
     #   so any number in the form "X" is de facto valid.

     if [ "$num" = "X" ] ; then
       return 0
     elif [ ! -z $(echo $num | sed 's/[[:digit:]]//g') ] ; then
       # Stripped out all the digits, and the remainder isn't empty? No good.
       return 1
     elif [ $num -gt $max ] ; then
       # Number is bigger than the maximum value allowed.
       return 1
     else
       return 0
     fi
   }

   validDay()
   {
     # Return 0 if the value passed to this function is a valid day name;
     #   1 otherwise.

     case $(echo $1 | tr '[:upper:]' '[:lower:]') in
       sun*|mon*|tue*|wed*|thu*|fri*|sat*) return 0 ;;
       X) return 0 ;;         # Special case, it's a rewritten "*"
       *) return 1
     esac
   }

   validMon()
   {
     # This function returns 0 if given a valid month name; 1 otherwise.

     case $(echo $1 | tr '[:upper:]' '[:lower:]') in
       jan*|feb*|mar*|apr*|may|jun*|jul*|aug*) return 0           ;;
       sep*|oct*|nov*|dec*)                    return 0           ;;
       X) return 0 ;; # Special case, it's a rewritten "*"
       *) return 1        ;;
     esac
   }

➊ fixvars()
   {
     # Translate all '*' into 'X' to bypass shell expansion hassles.
     #   Save original input as "sourceline" for error messages.

     sourceline="$min $hour $dom $mon $dow $command"
       min=$(echo "$min" | tr '*' 'X')      # Minute
       hour=$(echo "$hour" | tr '*' 'X')    # Hour
       dom=$(echo "$dom" | tr '*' 'X')      # Day of month
       mon=$(echo "$mon" | tr '*' 'X')      # Month
       dow=$(echo "$dow" | tr '*' 'X')      # Day of week
   }

 if [ $# -ne 1 ] || [ ! -r $1 ] ; then
     # If no crontab filename is given or if it's not readable by the script, fail.
     echo "Usage: $0 usercrontabfile" >&2
     exit 1
   fi

   lines=0  entries=0  totalerrors=0

   # Go through the crontab file line by line, checking each one.

   while read min hour dom mon dow command
   do
     lines="$(( $lines + 1 ))"
     errors=0

     if [ -z "$min" -o "${min%${min#?}}" = "#" ] ; then
       # If it's a blank line or the first character of the line is "#", skip it.
       continue    # Nothing to check
     fi

     ((entries++))

     fixvars

     # At this point, all the fields in the current line are split out into
     #   separate variables, with all asterisks replaced by "X" for convenience,
     #   so let's check the validity of input fields...

     # Minute check

➋   for minslice in $(echo "$min" | sed 's/[,-]/ /g') ; do
       if ! validNum $minslice 60 ; then
         echo "Line ${lines}: Invalid minute value \"$minslice\""
         errors=1
       fi
     done

     # Hour check

➌   for hrslice in $(echo "$hour" | sed 's/[,-]/ /g') ; do
       if ! validNum $hrslice 24 ; then
         echo "Line ${lines}: Invalid hour value \"$hrslice\""
         errors=1
       fi
     done

     # Day of month check

➍   for domslice in $(echo $dom | sed 's/[,-]/ /g') ; do
       if ! validNum $domslice 31 ; then
         echo "Line ${lines}: Invalid day of month value \"$domslice\""
         errors=1
       fi
     done

 # Month check: Has to check for numeric values and names both.
     #   Remember that a conditional like "if ! cond" means that it's
     #   testing whether the specified condition is FALSE, not true.

➎   for monslice in $(echo "$mon" | sed 's/[,-]/ /g') ; do
       if ! validNum $monslice 12 ; then
         if ! validMon "$monslice" ; then
           echo "Line ${lines}: Invalid month value \"$monslice\""
           errors=1
         fi
       fi
     done

     # Day of week check: Again, name or number is possible.

➏   for dowslice in $(echo "$dow" | sed 's/[,-]/ /g') ; do
       if ! validNum $dowslice 7 ; then
         if ! validDay $dowslice ; then
           echo "Line ${lines}: Invalid day of week value \"$dowslice\""
           errors=1
         fi
       fi
     done

     if [ $errors -gt 0 ] ; then
       echo ">>>> ${lines}: $sourceline"
       echo ""
       totalerrors="$(( $totalerrors + 1 ))"
     fi
   done < $1 # read the crontab passed as an argument to the script

   # Notice that it's here, at the very end of the while loop, that we
   #   redirect the input so that the user-specified filename can be
   #   examined by the script!

   echo "Done. Found $totalerrors errors in $entries crontab entries."

   exit 0
```

*列表 6-8：* `*verifycron*` *脚本*

#### *工作原理*

使这个脚本正常工作面临的最大挑战是避免由于 Shell 扩展星号字段值（`*`）而产生的问题。在`cron`条目中，星号是完全可以接受的，实际上也非常常见，但如果你通过`$( )`序列或管道将其传递给子 Shell，Shell 会自动将其扩展为当前目录中的文件列表——显然这不是我们想要的结果。与其为了解决这个问题而纠结于单引号和双引号的组合，不如将每个星号替换为一个`X`，这就是`fixvars`函数➊所做的，它会将内容分割成独立的变量，供后续测试使用。

另外值得注意的是，处理以逗号和破折号分隔的值列表的简单解决方案。标点符号会被空格替换，每个值会像独立的数字值一样进行测试。这就是在➋、➌、➍、➎和➏的`for`循环中，`$( )`序列所做的事情：

```
$(echo "$dow" | sed 's/[,-]/ /g')
```

这使得逐一检查所有数字值变得简单，确保每个值都是有效的，并且在该特定`crontab`字段参数的范围内。

#### *运行脚本*

这个脚本非常易于运行：只需将`crontab`文件的名称作为唯一参数指定即可。要处理现有的`crontab`文件，请参见列表 6-9。

```
$ crontab -l > my.crontab
$ verifycron my.crontab
$ rm my.crontab
```

*列表 6-9：在导出当前* `*cron*` *文件后运行* `*verifycron*` *脚本*

#### *结果*

使用一个包含两个错误和大量注释的示例`crontab`文件，脚本将生成在列表 6-10 中显示的结果。

```
$ verifycron sample.crontab
Line 10: Invalid day of week value "Mou"
>>>> 10: 06 22 * * Mou /home/ACeSystem/bin/del_old_ACinventories.pl

Line 12: Invalid minute value "99"
>>>> 12: 99 22 * * 1-3,6 /home/ACeSystem/bin/dump_cust_part_no.pl

Done. Found 2 errors in 13 crontab entries.
```

*列表 6-10：在含有无效条目的* `*cron*` *文件上运行* `*verifycron*` *脚本*

包含两个错误的示例`crontab`文件，以及本书中探讨的所有 shell 脚本，可以在*[`www.nostarch.com/wcss2/`](http://www.nostarch.com/wcss2/)*上找到。

#### *破解脚本*

有一些增强功能可能值得添加到这个脚本中。验证月份和日期组合的兼容性可以确保用户不会调度`cron`作业在例如 2 月 31 日运行。检查被调用的命令是否能实际找到也是有用的，但这需要解析和处理`PATH`变量（即用于查找脚本中指定的命令的目录列表），该变量可以在`crontab`文件中显式设置。这可能会非常棘手……最后，你还可以添加对诸如`@hourly`或`@reboot`等特殊值的支持，它们是`cron`中用于表示常见脚本运行时间的特殊值。

### #49 确保系统 cron 作业被执行

直到最近，Linux 系统都设计为作为服务器运行——全天候 24 小时，每周 7 天，永远不停。你可以在`cron`工具的设计中看到这种隐性预期：如果系统每天晚上 6 点关机，那么在每周四凌晨 2:17 调度任务就没有意义。

然而，许多现代 Unix 和 Linux 用户正在使用桌面电脑和笔记本电脑，因此他们在一天结束时会关闭系统。例如，对于 OS X 用户来说，系统过夜运行是相当陌生的，更别提在周末或假期中运行了。

这对于用户的`crontab`条目来说不是大问题，因为那些由于关机计划未能执行的条目可以调整，以确保最终能够被调用。问题出现在系统中日常、每周和每月的`cron`作业上，它们是底层系统的一部分，但未能在指定的时间执行。

这就是示例 6-11 中脚本的目的：允许管理员根据需要直接从命令行调用日常、每周或每月作业。

#### *代码*

```
   #!/bin/bash

   # docron--Runs the daily, weekly, and monthly system cron jobs on a system
   #   that's likely to be shut down during the usual time of day when the system
   #   cron jobs would otherwise be scheduled to run.

   rootcron="/etc/crontab"   # This is going to vary significantly based on
                             # which version of Unix or Linux you've got.

   if [ $# -ne 1 ] ; then
     echo "Usage: $0 [daily|weekly|monthly]" >&2
     exit 1
   fi

   # If this script isn't being run by the administrator, fail out.
   #   In earlier scripts, you saw USER and LOGNAME being tested, but in
   #   this situation, we'll check the user ID value directly. Root = 0.

   if [ "$(id -u)" -ne 0 ] ; then
     # Or you can use $(whoami) != "root" here, as needed.
     echo "$0: Command must be run as 'root'" >&2
     exit 1
   fi

   # We assume that the root cron has entries for 'daily', 'weekly', and
   #   'monthly' jobs. If we can't find a match for the one specified, well,
   #   that's an error. But first, we'll try to get the command if there is
   #   a match (which is what we expect).

➊ job="$(awk "NF > 6 && /$1/ { for (i=7;i<=NF;i++) print \$i }" $rootcron)"

   if [ -z "$job" ] ; then   # No job? Weird. Okay, that's an error.
     echo "$0: Error: no $1 job found in $rootcron" >&2
     exit 1
   fi

   SHELL=$(which sh)        # To be consistent with cron's default

➋ eval $job                # We'll exit once the job is finished.
```

*示例 6-11: `*docron*` 脚本*

#### *它是如何工作的*

位于*/etc/daily*、*/etc/weekly*和*/etc/monthly*（或*/etc/cron.daily*、*/etc/cron.weekly*和*/etc/cron.monthly*）中的`cron`作业与用户的`crontab`文件设置方式完全不同：每个目录包含一组脚本，每个作业对应一个脚本，这些脚本由`crontab`工具根据*/etc/crontab*文件中的设置执行。更让人困惑的是，*/etc/crontab*文件的格式也不同，因为它添加了一个额外的字段，用于指示应该由哪个有效的用户 ID 来执行作业。

*/etc/crontab*文件指定了每天、每周和每月作业的运行时间（在下列输出的第二列中），其格式与作为普通 Linux 用户所见的完全不同，如下所示：

```
$ egrep '(daily|weekly|monthly)' /etc/crontab
# Run daily/weekly/monthly jobs.
15      3       *       *       *       root    periodic daily
30      4       *       *       6       root    periodic weekly
30      5       1       *       *       root    periodic monthly
```

如果这个系统没有在每天凌晨 3:15、每周六早晨 4:30、以及每月 1 日早晨 5:30 运行，那么日常、每周和每月的任务会发生什么？什么也不发生。它们就是不会运行。

与其强行让`cron`运行任务，我们编写的脚本会在此文件中识别任务 ➊，并通过最后一行的`eval`直接运行它们 ➋。从这个脚本调用任务和作为`cron`任务调用任务的唯一区别在于，当任务从`cron`运行时，它们的输出流会自动转为电子邮件消息，而这个脚本则会在屏幕上显示输出流。

当然，你也可以通过如下方式调用脚本，复制`cron`的电子邮件行为：

```
./docron weekly | mail -E -s "weekly cron job" admin
```

#### *运行脚本*

这个脚本必须以 root 身份运行，并且有一个参数——`daily`、`weekly`或`monthly`——用于指示你想运行哪个组的系统`cron`任务。像往常一样，我们强烈推荐使用`sudo`来以 root 身份运行任何脚本。

#### *结果*

这个脚本基本没有直接输出，只会显示在`crontab`中运行的脚本结果，除非在脚本本身或由`cron`脚本启动的某个任务中遇到错误。

#### *破解脚本*

有些任务不应该每周或每月运行超过一次，因此确实应该有某种检查机制来确保它们不会更频繁地运行。此外，有时系统的定期任务可能会从`cron`运行，所以我们不能笼统地假设如果`docron`没有运行，任务就没有运行。

一种解决方案是创建三个空的时间戳文件，分别用于日常、每周和每月任务，然后在*/etc/daily*、*/etc/weekly*和*/etc/monthly*目录中添加新条目，通过`touch`更新每个时间戳文件的最后修改日期。这将解决一半的问题：`docron`可以检查上次调用定期`cron`任务的时间，并在经过的时间不足以证明任务应该重新运行时退出。

这个解决方案没有处理的情况是：在上次每月`cron`任务运行后的六周，管理员运行`docron`来调用每月任务。然后四天后，有人忘记关闭电脑，每月的`cron`任务被再次调用。那时，如何确保这个任务知道不再需要执行每月任务呢？

可以将两个脚本添加到适当的目录中。第一个脚本必须首先通过`run-script`或`periodic`运行（这是调用`cron`任务的标准方式），然后可以关闭目录中所有其他脚本的可执行权限，除了其配对脚本外，后者会在`run-script`或`periodic`扫描并确认没有需要执行的任务后，将可执行权限恢复。这种方法并不是一个理想的解决方案，因为无法保证脚本的执行顺序，如果我们不能确保新脚本的执行顺序，整个解决方案就会失败。

事实上，可能没有一个完美的解决方案来解决这个困境。或者它可能涉及编写一个`run-script`或`periodic`的包装器，能够管理时间戳，确保任务不会执行得过于频繁。也许我们只是担心了一些从大局来看并不那么严重的事情。![image](img/common1.jpg)

### #50 轮转日志文件

对于没有太多 Linux 经验的用户来说，系统日志文件中记录事件的命令、工具和守护进程的数量可能会让他们感到惊讶。即使是在磁盘空间充足的计算机上，也需要注意这些文件的大小——当然，也需要关注它们的内容。

因此，许多系统管理员会在其日志文件分析工具的顶部放置一组指令，类似于这里展示的命令：

```
mv $log.2 $log.3
mv $log.1 $log.2
mv $log $log.1
touch $log
```

如果每周运行一次，这将生成一个滚动的一个月日志文件信息归档，将数据分成按周划分的部分。然而，同样容易创建一个脚本，可以一次性处理*/var/log*目录中的所有日志文件，从而减轻任何日志文件分析脚本的负担，并且在管理员没有进行任何分析的月份也能管理日志。

列表 6-12 中的脚本逐一处理*/var/log*目录中符合特定标准的每个文件，检查每个匹配文件的轮转计划和最后修改日期，以确定是否需要轮转该文件。如果需要轮转，脚本会执行轮转操作。

#### *代码*

```
#!/bin/bash
# rotatelogs--Rolls logfiles in /var/log for archival purposes and to ensure
#   that the files don't get unmanageably large. This script uses a config
#   file to allow customization of how frequently each log should be rolled.
#   The config file is in logfilename=duration format, where duration is
#   in days. If, in the config file, an entry is missing for a particular
#   logfilename, rotatelogs won't rotate the file more frequently than every
#   seven days. If duration is set to zero, the script will ignore that
#   particular set of log files.

logdir="/var/log"             # Your logfile directory could vary.
config="$logdir/rotatelogs.conf"
mv="/bin/mv"

default_duration=7     # We'll default to a 7-day rotation schedule.
count=0

duration=$default_duration

if [ ! -f $config ] ; then
  # No config file for this script? We're out. You could also safely remove
  #   this test and simply ignore customizations when the config file is
  #   missing.
  echo "$0: no config file found. Can't proceed." >&2
  exit 1
fi

if [ ! -w $logdir -o ! -x $logdir ] ; then
  # -w is write permission and -x is execute. You need both to create new
  #   files in a Unix or Linux directory. If you don't have 'em, we fail.
  echo "$0: you don't have the appropriate permissions in $logdir" >&2
  exit 1
fi

cd $logdir

# While we'd like to use a standardized set notation like :digit: with
#   the find, many versions of find don't support POSIX character class
#   identifiers--hence [0-9].

# This is a pretty gnarly find statement that's explained in the prose
#   further in this section. Keep reading if you're curious!

for name in $(➊find . -maxdepth 1 -type f -size +0c ! -name '*[0-9]*' \
     ! -name '\.*' ! -name '*conf' -print | sed 's/^\.\///')
do

  count=$(( $count + 1 ))
  # Grab the matching entry from the config file for this particular log file.

  duration="$(grep "^${name}=" $config|cut -d= -f2)"

  if [ -z "$duration" ] ; then
    duration=$default_duration   # If there isn't a match, use the default.
  elif [ "$duration" = "0" ] ; then
    echo "Duration set to zero: skipping $name"
    continue
  fi

  # Set up the rotation filenames. Easy enough:

  back1="${name}.1"; back2="${name}.2";
  back3="${name}.3"; back4="${name}.4";

  # If the most recently rolled log file (back1) has been modified within
  #   the specific quantum, then it's not time to rotate it. This can be
  #   found with the -mtime modification time test to find.
 if [ -f "$back1" ] ; then
    if [ -z "$(find \"$back1\" -mtime +$duration -print 2>/dev/null)" ]
    then
      /bin/echo -n "$name's most recent backup is more recent than $duration "
      echo "days: skipping" ;   continue
    fi
  fi

  echo "Rotating log $name (using a $duration day schedule)"

  # Rotate, starting with the oldest log, but be careful in case one
  #   or more files simply don't exist yet.

  if [ -f "$back3" ] ; then
    echo "... $back3 -> $back4" ; $mv -f "$back3" "$back4"
  fi
  if [ -f "$back2" ] ; then
    echo "... $back2 -> $back3" ; $mv -f "$back2" "$back3"
  fi
  if [ -f "$back1" ] ; then
    echo "... $back1 -> $back2" ; $mv -f "$back1" "$back2"
  fi
  if [ -f "$name" ] ; then
    echo "... $name -> $back1" ; $mv -f "$name" "$back1"
  fi
  touch "$name"
  chmod 0600 "$name"    # Last step: Change file to rw------- for privacy
done

if [ $count -eq 0 ] ; then
  echo "Nothing to do: no log files big enough or old enough to rotate"
fi

exit 0
```

*列表 6-12：`*rotatelogs*` 脚本*

为了最大程度地发挥作用，脚本与一个配置文件一起工作，该配置文件位于*/var/log*中，允许管理员为不同的日志文件指定不同的轮转计划。一个典型配置文件的内容如列表 6-13 所示。

```
# Configuration file for the log rotation script: Format is name=duration,
#   where name can be any filename that appears in the /var/log directory.
#   Duration is measured in days.

ftp.log=30
lastlog=14
lookupd.log=7
lpr.log=30
mail.log=7
netinfo.log=7
secure.log=7
statistics=7
system.log=14
# Anything with a duration of zero is not rotated.
wtmp=0
```

*列表 6-13：`*rotatelogs*` 脚本的示例配置文件*

#### *工作原理*

该脚本的核心部分，也是最棘手的部分，是 ➊ 处的 `find` 语句。`find` 语句创建了一个循环，返回所有在 */var/log* 目录下，大小大于零字符的文件，这些文件名中不包含数字、不以点号开头（特别是 OS X 会在该目录下产生许多奇怪命名的日志文件——这些都需要跳过），并且不以 *conf* 结尾（显而易见，我们不想轮转 *rotatelogs.conf* 文件）。`maxdepth 1` 确保 `find` 不会进入子目录，最后的 `sed` 调用则移除了匹配结果中的任何前导 `./` 序列。

**注意**

*懒是好事！* `*rotatelogs*` *脚本展示了 Shell 脚本编程中的一个基本概念：避免重复工作的价值。与其让每个日志分析脚本单独轮转日志，不如由一个单一的日志轮转脚本来集中处理这一任务，从而使修改变得更简单。*

#### *运行脚本*

该脚本不接受任何参数，但它会打印出正在轮转哪些日志以及为什么要这样做的信息。它也应以 root 身份运行。

#### *结果*

`rotatelogs` 脚本易于使用，如 列表 6-14 所示，但请注意，根据文件权限，它可能需要以 root 身份运行。

```
$ sudo rotatelogs
ftp.log's most recent backup is more recent than 30 days: skipping
Rotating log lastlog (using a 14 day schedule)
... lastlog -> lastlog.1
lpr.log's most recent backup is more recent than 30 days: skipping
```

*列表 6-14：以 root 身份运行 `*rotatelogs*` 脚本以轮转 /var/log 中的日志。*

请注意，在此调用中，只有三个日志文件符合指定的 `find` 条件。根据配置文件中的持续时间值，其中只有 `lastlog` 文件最近没有得到足够的备份。然而，再次运行 `rotatelogs` 后，什么也没有发生，如 列表 6-15 所示。

```
$ sudo rotatelogs
ftp.log's most recent backup is more recent than 30 days: skipping
lastlog's most recent backup is more recent than 14 days: skipping
lpr.log's most recent backup is more recent than 30 days: skipping
```

*列表 6-15：再次运行 `*rotatelogs*` 显示无需再轮转其他日志。*

#### *破解脚本*

让这个脚本更有用的一种方法是，在 `$back4` 文件被 `mv` 命令覆盖之前，将最旧的存档文件通过电子邮件发送或复制到云存储网站上。对于电子邮件的简单情况，脚本可能就像这样：

```
echo "... $back3 -> $back4" ; $mv -f "$back3" "$back4"
```

对 `rotatelogs` 的另一个有用增强是将所有轮转的日志压缩，以进一步节省磁盘空间；这将要求脚本在执行时能够识别并正确处理压缩文件。

### #51 管理备份

管理系统备份是所有系统管理员都熟悉的任务，而且这是一个几乎得不到任何感谢的工作。没有人会说：“嘿，那个备份正常工作——干得好！”即便是在单用户的 Linux 计算机上，也需要某种备份计划。不幸的是，通常只有在数据和文件丢失后，你才会意识到定期备份的重要性。许多 Linux 系统忽视备份的原因之一是许多备份工具原始且难以理解。

一个 shell 脚本可以解决这个问题！清单 6-16 中的脚本备份指定的一组目录，可以选择增量备份（即，仅备份自上次备份以来发生变化的文件）或完整备份（所有文件）。备份会实时压缩，以最小化磁盘空间的使用，脚本输出可以定向到文件、磁带设备、远程挂载的 NFS 分区、云备份服务（例如我们在书中稍后设置的服务），甚至是 DVD。

#### *代码*

```
   #!/bin/bash

   # backup--Creates either a full or incremental backup of a set of defined
   #   directories on the system. By default, the output file is compressed and
   #   saved in /tmp with a timestamped filename. Otherwise, specify an output
   #   device (another disk, a removable storage device, or whatever else floats
   #   your boat).

 compress="bzip2"                 # Change to your favorite compression app.
    inclist="/tmp/backup.inclist.$(date +%d%m%y)"
     output="/tmp/backup.$(date +%d%m%y).bz2"
     tsfile="$HOME/.backup.timestamp"
      btype="incremental"           # Default to an incremental backup.
      noinc=0                       # And here's an update of the timestamp.

   trap "/bin/rm -f $inclist" EXIT

   usageQuit()
   {
     cat << "EOF" >&2
   Usage: $0 [-o output] [-i|-f] [-n]
     -o lets you specify an alternative backup file/device,
     -i is an incremental, -f is a full backup, and -n prevents
     updating the timestamp when an incremental backup is done.
   EOF
     exit 1
   }

   ########## Main code section begins here ###########

   while getopts "o:ifn" arg; do
     case "$opt" in
       o ) output="$OPTARG";       ;;   # getopts automatically manages OPTARG.
       i ) btype="incremental";    ;;
       f ) btype="full";           ;;
       n ) noinc=1;                ;;
       ? ) usageQuit               ;;
     esac
   done

   shift $(( $OPTIND - 1 ))

   echo "Doing $btype backup, saving output to $output"

   timestamp="$(date +'%m%d%I%M')"  # Grab month, day, hour, minute from date.
                                    # Curious about date formats? "man strftime"

   if [ "$btype" = "incremental" ] ; then
     if [ ! -f $tsfile ] ; then
       echo "Error: can't do an incremental backup: no timestamp file" >&2
       exit 1
     fi
     find $HOME -depth -type f -newer $tsfile -user ${USER:-LOGNAME} | \
➊   pax -w -x tar | $compress > $output
     failure="$?"
   else
     find $HOME -depth -type f -user ${USER:-LOGNAME} | \
➋   pax -w -x tar | $compress > $output
     failure="$?"
   fi

   if [ "$noinc" = "0" -a "$failure" = "0" ] ; then
     touch -t $timestamp $tsfile
   fi
   exit 0
```

*清单 6-16：* `*backup*` *脚本*

#### *工作原理*

对于完整的系统备份，➊ 和 ➋ 中的 `pax` 命令完成所有工作，将其输出通过管道传递给压缩程序（默认是 `bzip2`），然后输出到文件或设备。增量备份则稍微复杂一些，因为标准版本的 `tar` 不包括任何修改时间测试，这与 GNU 版本的 `tar` 不同。自上次备份以来修改过的文件列表是通过 `find` 构建的，并保存在 `inclist` 临时文件中。该文件模仿 `tar` 的输出格式，以提高便携性，然后直接传递给 `pax`。

选择何时标记备份的时间戳是许多备份程序容易出错的地方，通常将“最后备份时间”标记为程序完成备份时，而不是开始备份时。如果将时间戳设置为备份完成时的时间，当备份过程中有文件被修改时可能会出现问题，这种情况随着单个备份完成时间的延长而变得更有可能。因为在这种情况下修改过的文件，其最后修改时间会比时间戳日期更早，所以下次进行增量备份时，它们将不会被备份，这将是一个问题。

但请注意，因为将时间戳设置为*备份发生之前*也是错误的：如果备份因某种原因失败，无法撤销更新时间戳。

通过在备份开始之前（在 `timestamp` 变量中）保存日期和时间，并等待在备份成功后再通过 `-t` 标志将 `$timestamp` 应用到 `$tsfile`（使用 `touch`）来解决这两个问题。微妙吧？

#### *运行脚本*

该脚本有多个选项，所有选项都可以忽略，从而执行基于自上次脚本运行以来修改过的文件的默认增量备份（即，自上次增量备份的时间戳以来）。启动参数允许你指定不同的输出文件或设备（`-o output`），选择完整备份（`-f`），即使默认是增量备份，也可以主动选择增量备份（`-i`），或在进行增量备份时防止更新时间戳文件（`-n`）。

#### *结果*

`backup` 脚本无需任何参数，运行起来很简单，具体细节参见清单 6-17。

```
$ backup
Doing incremental backup, saving output to /tmp/backup.140703.bz2
```

*清单 6-17：运行* `*backup*` *脚本无需任何参数，并将结果输出到屏幕上。*

正如你所预期的那样，备份程序的输出并不十分引人注目。但生成的压缩文件足够大，足以显示里面有大量数据，正如你在清单 6-18 中所看到的。

```
$ ls -l /tmp/backup*
-rw-r--r--  1 taylor  wheel  621739008 Jul 14 07:31 backup.140703.bz2
```

*清单 6-18：使用* `*ls*` *命令显示已备份文件*

### #52 备份目录

与备份整个文件系统的任务相关的是一个以用户为中心的任务，即为特定目录或目录树拍摄快照。清单 6-19 中的简单脚本允许用户创建一个指定目录的压缩 `tar` 归档，以便备份或共享。

#### *代码*

```
   #!/bin/bash

   # archivedir--Creates a compressed archive of the specified directory

   maxarchivedir=10           # Size, in blocks, of big directory.
   compress=gzip              # Change to your favorite compress app.
   progname=$(basename $0)    # Nicer output format for error messages.

   if [ $# -eq 0 ] ; then     # No args? That's a problem.
     echo "Usage: $progname directory" >&2
     exit 1
   fi

   if [ ! -d $1 ] ; then
     echo "${progname}: can't find directory $1 to archive." >&2
     exit 1
   fi

   if [ "$(basename $1)" != "$1" -o "$1" = "." ] ; then
     echo "${progname}: You must specify a subdirectory" >&2
     exit 1
   fi

➊ if [ ! -w . ] ; then
     echo "${progname}: cannot write archive file to current directory." >&2
     exit 1
   fi

   # Is the resultant archive going to be dangerously big? Let's check...

   dirsize="$(du -s $1 | awk '{print $1}')"

   if [ $dirsize -gt $maxarchivedir ] ; then
     /bin/echo -n "Warning: directory $1 is $dirsize blocks. Proceed? [n] "
     read answer
     answer="$(echo $answer | tr '[:upper:]' '[:lower:]' | cut -c1)"
     if [ "$answer" != "y" ] ; then
       echo "${progname}: archive of directory $1 canceled." >&2
       exit 0
     fi
   fi

   archivename="$1.tgz"

   if ➋tar cf - $1 | $compress > $archivename ; then
     echo "Directory $1 archived as $archivename"
   else
     echo "Warning: tar encountered errors archiving $1"
   fi

   exit 0
```

*清单 6-19：* `*archivedir*` *脚本*

#### *工作原理*

该脚本几乎完全由错误检查代码组成，以确保它永远不会造成数据丢失或生成不正确的快照。除了使用典型的测试来验证起始参数的存在性和适当性外，该脚本还强制用户位于要压缩和归档的子目录的父目录中，确保归档文件在完成时保存到正确的位置。测试 `if [ ! -w . ]` ➊ 用于验证用户是否对当前目录具有写权限。即使在归档前，该脚本也会在备份文件异常大的情况下发出警告。

最终，实际执行归档指定目录的命令是 `tar` ➋。此命令的返回码会被测试，以确保在发生任何错误时脚本不会删除该目录。

#### *运行脚本*

该脚本应当以要归档的目录名作为唯一参数来调用。为了确保脚本不会尝试归档自身，它要求指定当前目录的一个子目录作为参数，而不是`.`，正如清单 6-20 所示。

#### *结果*

```
$ archivedir scripts
Warning: directory scripts is 2224 blocks. Proceed? [n] n
archivedir: archive of directory scripts canceled.
```

*清单 6-20：在* `*archivedir*` *脚本中运行* `scripts` *目录，但取消操作*

看起来这可能是一个较大的归档，我们犹豫是否创建它，但在深思熟虑之后，我们决定没有理由不继续执行。

```
$ archivedir scripts
Warning: directory scripts is 2224 blocks. Proceed? [n] y
Directory scripts archived as scripts.tgz
```

以下是结果：

```
$ ls -l scripts.tgz
-rw-r--r--  1 taylor  staff  325648 Jul 14 08:01 scripts.tgz
```

**注意**

*这是一个开发者的小贴士：在积极进行项目开发时，将* `*archivedir*` *脚本用于* `*cron*` *任务，每晚自动为你的工作代码拍摄快照以备份。*
