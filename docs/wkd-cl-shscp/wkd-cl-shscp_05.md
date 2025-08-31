## 第五章：**调整 Unix**

![image](img/common4.jpg)

外行可能会把 Unix 想象成在许多不同系统上都有一个良好、统一的命令行体验，这得益于它们遵循 POSIX 标准。但任何曾使用过不止一个 Unix 系统的人都知道它们在这些广泛参数内部有多么不同。例如，你很难找到一个 Unix 或 Linux 系统，它没有`ls`作为标准命令，但你的版本是否支持`--color`标志？你的 Bourne shell 版本是否支持变量切片（如`${var:0:2}`）？

或许 Shell 脚本最有价值的用途之一是调整你特定版本的 Unix 系统，使其更像其他系统。尽管大多数现代 GNU 实用工具在非 Linux Unix 上都能正常运行（例如，你可以用更新的 GNU `tar`替换古老的`tar`），但通常在调整 Unix 系统时不需要如此激进的系统更新，避免向支持的系统添加新二进制文件带来的潜在问题。相反，Shell 脚本可以用来将流行的标志映射到其本地等效项，利用核心 Unix 功能创建现有命令的智能版本，甚至解决某些功能长期缺失的问题。

### #27 显示带行号的文件

添加行号到显示文件有几种方法，其中许多方法都非常简短。例如，这里是一个使用`awk`的解决方案：

```
awk '{ print NR": "$0 }' < inputfile
```

在某些 Unix 实现中，`cat`命令有一个`-n`标志，而在其他系统上，`more`（或`less`、`pg`）分页器有一个指定每行输出应编号的标志。但在某些 Unix 版本中，这些方法都行不通，这时可以使用列表 4-1 中的简单脚本。

#### *代码*

```
   #!/bin/bash

   # numberlines--A simple alternative to cat -n, etc.

   for filename in "$@"
   do
     linecount="1"
➊   while IFS="\n" read line
     do
       echo "${linecount}: $line"
➋     linecount="$(( $linecount + 1 ))"
➌   done < $filename
   done
   exit 0
```

*列表 4-1：* `*numberlines*` *脚本*

#### *工作原理*

这个程序的主循环有一个技巧：它看起来像一个普通的`while`循环，但实际上重要的部分是`done < $filename` ➌。事实证明，每个主要的块结构都作为自己的虚拟子 shell，因此这种文件重定向不仅有效，而且是一种轻松的方法，可以让循环逐行迭代`$filename`的内容。再结合➊处的`read`语句——一个内部循环，逐次将每行加载到`line`变量中——输出带有行号的行和增加`linecount`变量 ➋ 就变得很容易了。

#### *运行脚本*

你可以将任意数量的文件名输入到这个脚本中。但你不能通过管道输入，不过如果没有提供起始参数，通过调用`cat -`序列来修复这个问题也不是很难。

#### *结果*

列表 4-2 展示了使用`numberlines`脚本对《爱丽丝梦游仙境》摘录进行行号标记的文件。

```
$ numberlines alice.txt
1: Alice was beginning to get very tired of sitting by her sister on the
2: bank, and of having nothing to do: once or twice she had peeped into the
3: book her sister was reading, but it had no pictures or conversations in
4: it, 'and what is the use of a book,' thought Alice 'without pictures or
5: conversations?'
6:
7: So she was considering in her own mind (as well as she could, for the
8: hot day made her feel very sleepy and stupid), whether the pleasure
9: of making a daisy-chain would be worth the trouble of getting up and
10: picking the daisies, when suddenly a White Rabbit with pink eyes ran
11: close by her.
```

*列表 4-2：* 在《爱丽丝梦游仙境》摘录上测试* `*numberlines*` *脚本*

#### *黑客脚本*

一旦你有了一个带有行号的文件，你可以像这样将文件中的所有行的顺序反转：

```
cat -n filename | sort -rn | cut -c8-
```

这在支持 `cat` 的 `-n` 标志的系统中有效。例如，这种方法可能在哪些地方有用呢？一个明显的场景是按从新到旧的顺序显示日志文件。

### #28 只换行长行

`fmt` 命令及其对应的 shell 脚本版本的一个限制是，它们会换行并填充每一行，不管这样做是否合理。例如，这可能会搞乱电子邮件（例如，换行你的 `.signature` 是不好的）以及任何需要保留行分隔符的输入文件格式。

如果你有一个文档，你只想将长行换行，而保持其他部分不变呢？对于 Unix 用户来说，默认的命令集只有一种方法可以完成这项任务：在编辑器中逐行处理每一行，将长行单独传给 `fmt`。（你可以在 `vi` 中通过将光标移到目标行并使用 `!$fmt` 来完成此操作。）

清单 4-3 中的脚本自动化了这个任务，利用了 shell 的 `${#*varname*}` 结构，它返回存储在变量 `*varname*` 中的数据的长度。

#### *代码*

```
   #!/bin/bash
   # toolong--Feeds the fmt command only those lines in the input stream
   #   that are longer than the specified length

   width=72

   if [ ! -r "$1" ] ; then
     echo "Cannot read file $1" >&2
     echo "Usage: $0 filename" >&2
     exit 1
   fi

➊ while read input
   do
     if [ ${#input} -gt $width ] ; then
       echo "$input" | fmt
     else
       echo "$input"
     fi
➋ done < $1

   exit 0
```

*清单 4-3：* `*toolong*` *脚本*

#### *工作原理*

请注意，文件通过简单的 `< $1` 被传递给 `while` 循环，并与循环的结束位置 ➋ 关联，然后每一行通过 `read input` ➊ 被读取，从而一行一行地将文件内容分配给 `input` 变量。

如果你的 shell 不支持 `${#*var*}` 这种表示法，你可以通过超实用的“字数统计”命令 `wc` 来模拟它的行为：

```
varlength="$(echo "$var" | wc -c)"
```

然而，`wc` 有个恼人的习惯，它会在输出前加上空格以使数值对齐。为了避免这个麻烦，需要稍微修改，确保只通过最终的管道步骤传递数字，如下所示：

```
varlength="$(echo "$var" | wc -c | sed 's/[^[:digit:]]//g')"
```

#### *运行脚本*

这个脚本接受正好一个文件名作为输入，正如 清单 4-4 所示。

#### *结果*

```
$ toolong ragged.txt
So she sat on, with closed eyes, and half believed herself in
Wonderland, though she knew she had but to open them again, and
all would change to dull reality--the grass would be only rustling
in the wind, and the pool rippling to the waving of the reeds--the
rattling teacups would change to tinkling sheep-bells, and the
Queen's shrill cries to the voice of the shepherd boy--and the
sneeze
of the baby, the shriek of the Gryphon, and all the other queer
noises, would change (she knew) to the confused clamour of the busy
farm-yard--while the lowing of the cattle in the distance would
take the place of the Mock Turtle's heavy sobs.
```

*清单 4-4：测试* `*toolong*` *脚本*

请注意，与标准的 `fmt` 调用不同，`toolong` 在可能的情况下保留了换行，因此在输入文件中单独一行的单词 *sneeze* 在输出中也会单独占一行。

### #29 显示包含附加信息的文件

许多最常用的 Unix 和 Linux 命令最初是为缓慢、几乎没有交互的输出环境设计的（我们谈过 Unix 是一个古老的操作系统，对吧？），因此它们提供的输出和交互性非常有限。例如，`cat`：当用来查看一个短文件时，它不会提供太多有用的信息。不过，假如我们想知道文件的更多信息，那就来获取它吧！清单 4-5 详细介绍了 `showfile` 命令，这是 `cat` 的一种替代方法。

#### *代码*

```
   #!/bin/bash
   # showfile--Shows the contents of a file, including additional useful info

   width=72

   for input
   do
     lines="$(wc -l < $input | sed 's/ //g')"
     chars="$(wc -c < $input | sed 's/ //g')"
     owner="$(ls -ld $input | awk '{print $3}')"
     echo "-----------------------------------------------------------------"
     echo "File $input ($lines lines, $chars characters, owned by $owner):"
     echo "-----------------------------------------------------------------"
     while read line
     do
       if [ ${#line} -gt $width ] ; then
         echo "$line" | fmt | sed -e '1s/^/ /' -e '2,$s/^/+ /'
       else
         echo "  $line"
       fi
➊   done < $input

     echo "-----------------------------------------------------------------"

➋ done | ${PAGER:more}

   exit 0
```

*清单 4-5：* `*showfile*` *脚本*

#### *工作原理*

为了同时逐行读取输入并添加头部和尾部信息，这个脚本使用了一个方便的 shell 技巧：在脚本的末尾，它通过 `done < $input` ➊ 将输入重定向到 `while` 循环中。然而，可能这个脚本中最复杂的元素是对于超过指定长度的行，调用 `sed` 进行处理：

```
echo "$line" | fmt | sed -e '1s/^/ /' -e '2,$s/^/+ /'
```

超过最大允许长度的行会通过 `fmt`（或其 shell 脚本替代品，脚本 #14 在第 53 页）进行换行处理。为了直观地标示哪些行是续行，哪些行是从原文件中保留的，过长行的第一行输出会有通常的两个空格缩进，但后续行则以加号和一个空格作为前缀。最后，将输出通过管道传送到 `${PAGER:more}` 会使用系统变量 `$PAGER` 设置的分页程序显示文件，或者如果未设置该变量，则使用 `more` 程序 ➋。

#### *运行脚本*

你可以通过在调用程序时指定一个或多个文件名来运行 `showfile`，正如清单 4-6 所示。

#### *结果*

```
$ showfile ragged.txt
-----------------------------------------------------------------
File ragged.txt (7 lines, 639 characters, owned by taylor):
-----------------------------------------------------------------
  So she sat on, with closed eyes, and half believed herself in
  Wonderland, though she knew she had but to open them again, and
  all would change to dull reality--the grass would be only rustling
+ in the wind, and the pool rippling to the waving of the reeds--the
  rattling teacups would change to tinkling sheep-bells, and the
  Queen's shrill cries to the voice of the shepherd boy--and the
  sneeze
  of the baby, the shriek of the Gryphon, and all the other queer
+ noises, would change (she knew) to the confused clamour of the busy
+ farm-yard--while the lowing of the cattle in the distance would
+ take the place of the Mock Turtle's heavy sobs.
```

*清单 4-6：测试* `*showfile*` *脚本*

### #30 模拟 GNU 样式标志与 quota

各种 Unix 和 Linux 系统中命令标志的不一致性是一个长期存在的问题，这给那些在主要版本之间切换的用户带来了很多麻烦，尤其是从商业 Unix 系统（如 SunOS/Solaris、HP-UX 等）切换到开源 Linux 系统时。一个展示此问题的命令是 `quota`，它在某些 Unix 系统上支持全名标志，但在其他系统上仅接受单字母标志。

一个简洁的 shell 脚本（如清单 4-7 所示）通过将任何指定的全名标志映射到相应的单字母替代标志来解决问题。

#### *代码*

```
   #!/bin/bash
   # newquota--A frontend to quota that works with full-word flags a la GNU

   # quota has three possible flags, -g, -v, and -q, but this script
   #   allows them to be '--group', '--verbose', and '--quiet' too.

   flags=""
   realquota="$(which quota)"

   while [ $# -gt 0 ]
   do
     case $1
     in
       --help)      echo "Usage: $0 [--group --verbose --quiet -gvq]" >&2
                          exit 1 ;;
       --group)     flags="$flags -g";   shift ;;
       --verbose)   flags="$flags -v";   shift ;;
       --quiet)     flags="$flags -q";   shift ;;
       --)          shift;               break ;;
       *)           break;          # Done with 'while' loop!
     esac

   done

➊ exec $realquota $flags "$@"
```

*清单 4-7：* `*newquota*` *脚本*

#### *工作原理*

这个脚本实际上归结为一个 `while` 语句，它遍历传递给脚本的每个参数，识别任何匹配的全名标志，并将相关的单字母标志添加到 `flags` 变量中。完成后，它会简单地调用原始的 quota 程序 ➊，并根据需要添加用户指定的标志。

#### *运行脚本*

有几种方法可以将这种包装程序集成到你的系统中。最明显的方法是将此脚本重命名为`quota`，然后将该脚本放置在本地目录（例如，*/usr/local/bin*）中，并确保用户的默认`PATH`在查找标准 Linux 二进制目录（*/bin*和*/usr/bin*）之前会先查找这个目录。另一种方法是添加系统范围的别名，使得输入`quota`的用户实际上调用的是`newquota`脚本。（一些 Linux 发行版附带了管理系统别名的工具，如 Debian 的`alternatives`系统。）然而，最后一种策略可能存在风险，因为如果用户在自己的脚本中使用带有新标志的`quota`，如果这些脚本没有使用用户的交互式登录 shell，它们可能看不到指定的别名，最终会调用基础的`quota`命令，而不是`newquota`。

#### *结果*

清单 4-8 详细说明了如何使用`--verbose`和`--quiet`参数运行`newquota`。

```
$ newquota --verbose
Disk quotas for user dtint (uid 24810):
     Filesystem   usage   quota   limit   grace   files   quota   limit   grace
           /usr  338262  614400  675840           10703  120000  126000
$ newquota --quiet
```

*清单 4-8：测试* `*newquota*` *脚本*

`--quiet`模式只在用户超出配额时才会输出信息。从最后的结果中可以看到，这正正常工作，因为我们没有超出配额。呼——！

### #31 让 sftp 更像 ftp

文件传输协议`ftp`的安全版本包含在`ssh`（安全 Shell 包）中，但对于从老旧的`ftp`客户端切换过来的用户来说，它的界面可能有些让人困惑。基本问题在于，`ftp`是以`ftp remotehost`的形式调用的，然后它会提示输入帐户和密码信息。相比之下，`sftp`希望在命令行中指定帐户和远程主机，如果只指定主机，它就无法正常工作（或者无法按预期工作）。

为了解决这个问题，清单 4-9 中详细介绍的简单包装脚本允许用户像调用`ftp`程序一样调用`mysftp`，并提示输入必要的字段。

#### *代码*

```
   #!/bin/bash

   # mysftp--Makes sftp start up more like ftp

   /bin/echo -n "User account: "
   read account

   if [ -z $account ] ; then
     exit 0;       # Changed their mind, presumably
   fi

   if [ -z "$1" ] ; then
     /bin/echo -n "Remote host: "
     read host
     if [ -z $host ] ; then
       exit 0
     fi
   else
     host=$1
   fi

   # End by switching to sftp. The -C flag enables compression here.

➊ exec sftp -C $account@$host
```

*清单 4-9：* `*mysftp*` *脚本，`*sftp*`的更友好版本*

#### *工作原理*

这个脚本中有一个值得一提的技巧。实际上，这是我们在之前的脚本中做过的，只不过之前没有特别强调：最后一行是一个`exec`调用➊。它的作用是*替换*当前运行的 shell，执行指定的应用程序。因为你知道，在调用`sftp`命令后，已经没有其他操作需要做了，这种结束脚本的方法比让 shell 等待`sftp`完成并使用一个单独的子 shell 要更高效——如果我们直接调用`sftp`的话，情况就会是这样。

#### *运行脚本*

和`ftp`客户端一样，如果用户省略了远程主机，脚本会继续并提示输入远程主机。如果脚本以`mysftp remotehost`的形式调用，则使用提供的`remotehost`。

#### *结果*

让我们看看当你在没有任何参数的情况下调用这个脚本时，和在没有任何参数的情况下调用`sftp`时会发生什么。清单 4-10 展示了运行`sftp`的情况。

```
$ sftp
usage: sftp [-1246Cpqrv] [-B buffer_size] [-b batchfile] [-c cipher]
          [-D sftp_server_path] [-F ssh_config] [-i identity_file] [-l limit]
          [-o ssh_option] [-P port] [-R num_requests] [-S program]
          [-s subsystem | sftp_server] host
       sftp [user@]host[:file ...]
       sftp [user@]host[:dir[/]]
       sftp -b batchfile [user@]host
```

*清单 4-10：运行`*sftp*`工具时不带参数会产生非常难以理解的帮助输出。*

这很有用，但也很混淆。相比之下，通过`mysftp`脚本，你可以继续进行实际连接，正如清单 4-11 所示。

```
$ mysftp
User account: taylor
Remote host: intuitive.com
Connecting to intuitive.com...
taylor@intuitive.com's password:
sftp> quit
```

*清单 4-11：运行`*mysftp*`脚本时不带参数更为清晰。*

像调用`ftp`会话一样调用脚本，提供远程主机，它将提示输入远程帐户名（在清单 4-12 中详细说明），然后悄悄地调用`sftp`。

```
$ mysftp intuitive.com
User account: taylor
Connecting to intuitive.com...
taylor@intuitive.com's password:
sftp> quit
```

*清单 4-12：运行`*mysftp*`脚本时提供单个参数：要连接的主机*

#### *破解脚本*

当你有这样的脚本时，始终要思考的一件事是，它是否可以作为自动备份或同步工具的基础，而`mysftp`就是一个完美的候选者。所以一个很好的技巧就是在你的系统上指定一个目录，例如，然后写一个包装器来创建关键文件的 ZIP 归档，并使用`mysftp`将它们复制到服务器或云存储系统。实际上，我们将在本书稍后通过脚本 #72 在第 229 页来做这个。

### #32 修复 grep

一些版本的`grep`提供了丰富的功能，包括特别有用的显示匹配行上下文（上下各一两行）的能力。此外，一些版本的`grep`还可以高亮显示匹配指定模式的行中的区域（至少对于简单模式）。你可能已经拥有这样的版本的`grep`，但也可能没有。

幸运的是，这两个功能都可以通过 Shell 脚本来模拟，因此即使你使用的是旧版商业 Unix 系统，且`grep`命令相对原始，仍然可以使用它们。要指定匹配指定模式的行上下文的行数，可以使用`-c *value*`，后面跟上要匹配的模式。这个脚本（见清单 4-13）还借用了 ANSI 颜色脚本，脚本 #11 在第 40 页中，来进行区域高亮。

#### *代码*

```
   #!/bin/bash

   # cgrep--grep with context display and highlighted pattern matches

   context=0
   esc="^["
   boldon="${esc}[1m" boldoff="${esc}[22m"
   sedscript="/tmp/cgrep.sed.$$"
   tempout="/tmp/cgrep.$$"

   function showMatches
   {
     matches=0

➊   echo "s/$pattern/${boldon}$pattern${boldoff}/g" > $sedscript

➋   for lineno in $(grep -n "$pattern" $1 | cut -d: -f1)
     do
       if [ $context -gt 0 ] ; then
➌       prev="$(( $lineno - $context ))"

         if [ $prev -lt 1 ] ; then
           # This results in "invalid usage of line address 0."
           prev="1"
         fi
➍       next="$(( $lineno + $context ))"

         if [ $matches -gt 0 ] ; then
           echo "${prev}i\\" >> $sedscript
           echo "----" >> $sedscript
         fi
         echo "${prev},${next}p" >> $sedscript
       else
         echo "${lineno}p" >> $sedscript
       fi
       matches="$(( $matches + 1 ))"
     done

     if [ $matches -gt 0 ] ; then
       sed -n -f $sedscript $1 | uniq | more
     fi
   }

➎ trap "$(which rm) -f $tempout $sedscript" EXIT

   if [ -z "$1" ] ; then
     echo "Usage: $0 [-c X] pattern {filename}" >&2
     exit 0
   fi

   if [ "$1" = "-c" ] ; then
     context="$2"
     shift; shift
   elif [ "$(echo $1|cut -c1-2)" = "-c" ] ; then
     context="$(echo $1 | cut -c3-)"
     shift
   fi

   pattern="$1"; shift

   if [ $# -gt 0 ] ; then
     for filename ; do
       echo "----- $filename -----"
       showMatches $filename
     done
   else
     cat - > $tempout      # Save stream to a temp file.
     showMatches $tempout
   fi

   exit 0
```

*清单 4-13：`*cgrep*`脚本*

#### *它是如何工作的*

这个脚本使用`grep -n`来获取文件中所有匹配行的行号➋，然后，使用指定的上下文行数，确定显示每个匹配项的起始➌和结束➍行。这些行会写入在➊定义的临时`sed`脚本中，该脚本执行一个单词替换命令，将指定的模式包装在加粗开关 ANSI 序列中。这就是脚本的 90%，简而言之。

这个脚本中值得一提的另一个点是有用的 `trap` 命令 ➎，它让你将事件与 shell 脚本执行系统本身关联起来。第一个参数是你希望调用的命令或命令序列，所有后续参数是具体的信号（事件）。在这个案例中，我们告诉 shell 当脚本退出时，调用 `rm` 删除两个临时文件。

使用 `trap` 工作的特别好的一点是，无论你从脚本的哪里退出，它都会起作用，而不仅仅是在脚本的最底部。在后续的脚本中，你将看到 `trap` 可以绑定到各种信号，而不仅仅是 `SIGEXIT`（或 `EXIT`，或 `SIGEXIT` 的数值等价物，`0`）。事实上，你可以将不同的 `trap` 命令与不同的信号关联，因此，如果有人向脚本发送 `SIGQUIT`（CTRL-C），你可能会输出“已清理临时文件”的消息，而在常规的 (`SIGEXIT`) 事件中则不会显示该消息。

#### *运行脚本*

这个脚本可以处理输入流，在这种情况下它会将输入保存到临时文件中，然后像处理命令行指定的文件一样处理该临时文件，或者处理命令行中的一个或多个文件列表。清单 4-14 显示了通过命令行传递单个文件的示例。

#### *结果*

```
$ cgrep -c 1 teacup ragged.txt
----- ragged.txt -----
in the wind, and the pool rippling to the waving of the reeds--the
rattling teacups would change to tinkling sheep-bells, and the
Queen's shrill cries to the voice of the shepherd boy--and the
```

*清单 4-14：测试* `*cgrep*` *脚本*

#### *破解脚本*

对这个脚本的一个有用改进是返回匹配行的行号。

### #33 处理压缩文件

多年来的 Unix 开发中，很少有程序像`compress`那样被反复考虑和重新开发。在大多数 Linux 系统上，有三种明显不同的压缩程序可供使用：`compress`、`gzip` 和 `bzip2`。每种程序使用不同的后缀（分别是 *.z*、*.gz* 和 *.bz2*），并且压缩程度可能会根据文件中数据的布局而有所不同。

无论压缩级别如何，也无论你安装了哪个压缩程序，在许多 Unix 系统上，处理压缩文件都需要手动解压，完成所需任务后再重新压缩。这是一个繁琐的过程，因此非常适合用 shell 脚本来处理！清单 4-15 中详细的脚本作为一个方便的压缩/解压包装器，适用于你经常需要在压缩文件上使用的三个功能：`cat`、`more` 和 `grep`。

#### *代码*

```
   #!/bin/bash

   # zcat, zmore, and zgrep--This script should be either symbolically
   #   linked or hard linked to all three names. It allows users to work with
   #   compressed files transparently.

    Z="compress";  unZ="uncompress"  ;  Zlist=""
   gz="gzip"    ; ungz="gunzip"      ; gzlist=""
   bz="bzip2"   ; unbz="bunzip2"     ; bzlist=""

   # First step is to try to isolate the filenames in the command line.
   #   We'll do this lazily by stepping through each argument, testing to
   #   see whether it's a filename. If it is and it has a compression
   #   suffix, we'll decompress the file, rewrite the filename, and proceed.
   #   When done, we'll recompress everything that was decompressed.

   for arg
   do
     if [ -f "$arg" ] ; then
       case "$arg" in
          *.Z) $unZ "$arg"
               arg="$(echo $arg | sed 's/\.Z$//')"
               Zlist="$Zlist \"$arg\""
               ;;

         *.gz) $ungz "$arg"
               arg="$(echo $arg | sed 's/\.gz$//')"
               gzlist="$gzlist \"$arg\""
               ;;

        *.bz2) $unbz "$arg"
               arg="$(echo $arg | sed 's/\.bz2$//')"
               bzlist="$bzlist \"$arg\""
               ;;
       esac
     fi
     newargs="${newargs:-""} \"$arg\""
   done

   case $0 in
      *zcat* ) eval cat $newargs                   ;;
     *zmore* ) eval more $newargs                  ;;
     *zgrep* ) eval grep $newargs                  ;;
           * ) echo "$0: unknown base name. Can't proceed." >&2
               exit 1
   esac

   # Now recompress everything.

   if [ ! -z "$Zlist"  ] ; then
➊   eval $Z $Zlist
   fi
   if [ ! -z "$gzlist"] ; then
➋   eval $gz $gzlist
   fi
   if [ ! -z "$bzlist" ] ; then
➌   eval $bz $bzlist
   fi

   # And done!

   exit 0
```

*清单 4-15：* `*zcat*`*/*`*zmore*`*/*`*zgrep*` *脚本*

#### *它是如何工作的*

对于任何给定的后缀，都需要三个步骤：解压文件、重命名文件以去除后缀，并将其添加到脚本末尾重新压缩的文件列表中。通过保持三个单独的列表，每个压缩程序一个，这个脚本还让你可以轻松地在使用不同压缩工具压缩的文件之间进行 `grep` 搜索。

最重要的技巧是在重新压缩文件时使用`eval`指令 ➊➋➌。这对于确保带空格的文件名被正确处理是必要的。当`Zlist`、`gzlist`和`bzlist`变量被实例化时，每个参数都会被引号括起来，因此一个典型的值可能是`""sample.c" "test.pl" "penny.jar""`。由于列表中有嵌套的引号，调用类似`cat $Zlist`的命令会导致`cat`抱怨找不到文件`"sample.c"`。为了强制 shell 执行命令，仿佛命令是在命令行中输入的（引号在`arg`解析后会被去除），使用`eval`，这样一切就能按预期工作。

#### *运行脚本*

为了正确运行，该脚本应该有三个名称。如何在 Linux 中实现这一点？简单：链接。您可以使用符号链接，它是存储链接目标名称的特殊文件，或者使用硬链接，硬链接实际上会被分配与被链接文件相同的 inode。我们更倾向于使用符号链接。这些链接可以很容易地创建（这里脚本已经被命名为`zcat`），如清单 4-16 所示。

```
$ ln -s zcat zmore
$ ln -s zcat zgrep
```

*清单 4-16：符号链接* `*zcat*` *脚本到* `*zmore*` *和* `*zgrep*` *命令*

完成后，您将有三个新的命令，它们具有相同的实际（共享的）内容，每个命令都接受一个文件列表，根据需要处理文件，完成后解压缩并重新压缩它们。

#### *结果*

无处不在的`compress`工具可以快速压缩*ragged.txt*并为其添加*.z*后缀：

```
$ compress ragged.txt
```

在*ragged.txt*的压缩状态下，我们可以使用`zcat`查看文件，具体细节见清单 4-17。

```
$ zcat ragged.txt.Z
So she sat on, with closed eyes, and half believed herself in
Wonderland, though she knew she had but to open them again, and
all would change to dull reality--the grass would be only rustling
in the wind, and the pool rippling to the waving of the reeds--the
rattling teacups would change to tinkling sheep-bells, and the
Queen's shrill cries to the voice of the shepherd boy--and the
sneeze of the baby, the shriek of the Gryphon, and all the other
queer noises, would change (she knew) to the confused clamour of
the busy farm-yard--while the lowing of the cattle in the distance
would take the place of the Mock Turtle's heavy sobs.
```

*清单 4-17：使用* `*zcat*` *打印压缩的文本文件*

然后再次搜索*teacup*。

```
$ zgrep teacup ragged.txt.Z
rattling teacups would change to tinkling sheep-bells, and the
```

同时，文件保持在其原始的压缩状态，起始和结束状态都如清单 4-18 所示。

```
$ ls -l ragged.txt*
-rw-r--r-- 1 taylor staff 443 Jul 7 16:07 ragged.txt.Z
```

*清单 4-18：`*ls*`*的结果，只显示压缩文件存在*

#### *破解脚本*

这个脚本可能最大的问题是，如果在中途取消，文件不能保证会重新压缩。一个不错的改进是使用`trap`功能智能地解决这个问题，并增加一个带有错误检查的重新压缩函数。

### #34 确保最大压缩的文件

正如脚本 #33 在第 109 页中强调的那样，大多数 Linux 实现都包括多种压缩方法，但用户需要自己弄清楚哪种方法对给定文件压缩效果最好。因此，用户通常只学会使用一种压缩程序，而没有意识到他们可以通过使用其他程序获得更好的效果。更让人困惑的是，某些文件使用一种算法压缩效果更好，而使用另一种则较差，而没有实验就无法知道哪种更好。

逻辑上的解决方案是使用一个脚本来利用每个工具压缩文件，然后选择最小的文件作为最佳结果。这正是`bestcompress`所做的，见清单 4-19！

#### *代码*

```
   #!/bin/bash

   # bestcompress--Given a file, tries compressing it with all the available
   #   compression tools and keeps the compressed file that's smallest,
   #   reporting the result to the user. If -a isn't specified, bestcompress
   #   skips compressed files in the input stream.

   Z="compress"     gz="gzip"     bz="bzip2"
   Zout="/tmp/bestcompress.$$.Z"
   gzout="/tmp/bestcompress.$$.gz"
   bzout="/tmp/bestcompress.$$.bz"
   skipcompressed=1

   if [ "$1" = "-a" ] ; then
     skipcompressed=0 ; shift
   fi

   if [ $# -eq 0 ]; then
     echo "Usage: $0 [-a] file or files to optimally compress" >&2
     exit 1
   fi

   trap "/bin/rm -f $Zout $gzout $bzout" EXIT

   for name in "$@"
   do
     if [ ! -f "$name" ] ; then
       echo "$0: file $name not found. Skipped." >&2
       continue
     fi

     if [ "$(echo $name | egrep '(\.Z$|\.gz$|\.bz2$)')" != "" ] ; then
       if [ $skipcompressed -eq 1 ] ; then
         echo "Skipped file ${name}: It's already compressed."
         continue
       else
         echo "Warning: Trying to double-compress $name"
       fi
     fi

   # Try compressing all three files in parallel.
➊   $Z  < "$name" > $Zout  &
     $gz < "$name" > $gzout &
     $bz < "$name" > $bzout &

     wait  # Wait until all compressions are done.

   # Figure out which compressed best.
➋   smallest="$(ls -l "$name" $Zout $gzout $bzout | \
       awk '{print $5"="NR}' | sort -n | cut -d= -f2 | head -1)"

     case "$smallest" in
➌     1 ) echo "No space savings by compressing $name. Left as is."
           ;;
       2 ) echo Best compression is with compress. File renamed ${name}.Z
           mv $Zout "${name}.Z" ; rm -f "$name"
           ;;
       3 ) echo Best compression is with gzip. File renamed ${name}.gz
           mv $gzout "${name}.gz" ; rm -f "$name"
           ;;
       4 ) echo Best compression is with bzip2\. File renamed ${name}.bz2
           mv $bzout "${name}.bz2" ; rm -f "$name"
     esac

   done

   exit 0
```

*清单 4-19：* `*bestcompress*` *脚本*

#### *原理*

该脚本中最有趣的一行是 ➋。这一行让 `ls` 输出每个文件的大小（原始文件以及三个压缩文件，按已知顺序），然后用 `awk` 剪切出文件大小，按数字排序，最终得出最小文件的行号。如果所有压缩版本的文件都比原文件大，结果将是 `1`，并打印出相应的信息 ➌。否则，`smallest` 将指示是 `compress`、`gzip` 还是 `bzip2` 做得最好。然后，脚本只需将相应的文件移到当前目录，并删除原文件。

从 ➊ 开始的三个压缩调用也值得注意。这些调用是并行进行的，通过使用尾随的 `&` 将每个调用放入自己的子 shell 中，接着调用 `wait`，直到所有调用完成，脚本才会停止。在单处理器的情况下，这可能不会带来太大的性能提升，但在多处理器的环境下，它应该可以分摊任务，并可能更快完成。

#### *运行脚本*

该脚本应该使用文件名列表进行调用，以压缩这些文件。如果其中某些文件已经被压缩，并且你想尝试进一步压缩它们，请使用`-a`标志；否则，这些文件将被跳过。

#### *结果*

演示该脚本的最佳方式是使用一个需要压缩的文件，正如清单 4-20 所示。

```
$ ls -l alice.txt
-rw-r--r--  1 taylor  staff  154872 Dec  4  2002 alice.txt
```

*清单 4-20：显示* `*ls*` *命令输出的《爱丽丝梦游仙境》副本。请注意，文件大小为 154872 字节。*

脚本隐藏了使用三种压缩工具压缩文件的过程，而是简单地显示结果，结果见清单 4-21。

```
$ bestcompress alice.txt
Best compression is with compress. File renamed alice.txt.Z
```

*清单 4-21：运行* `*bestcompress*` *脚本来压缩* alice.txt

清单 4-22 演示了文件现在变得相当小。

```
$ ls -l alice.txt.Z
-rw-r--r--  1 taylor  wheel  66287 Jul  7 17:31 alice.txt.Z
```

*清单 4-22：演示压缩文件（66287 字节）相比于清单 4-20 的文件大小大幅减小。*
