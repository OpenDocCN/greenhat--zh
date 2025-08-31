## 第三章：**改进用户命令**

![image](img/common4.jpg)

一个典型的 Unix 或 Linux 系统默认包括数百个命令，当你考虑到标志和将命令与管道组合的不同方式时，就会产生数百万种不同的命令行工作方式。

在我们深入之前，清单 2-1 展示了一个附加脚本，它会告诉你 `PATH` 中有多少个命令。

```
#!/bin/bash

# How many commands: a simple script to count how many executable
#   commands are in your current PATH

IFS=":"
count=0 ; nonex=0
for directory in $PATH ;  do
  if [ -d "$directory" ] ; then
    for command in "$directory"/* ; do
      if [ -x "$command" ] ; then
        count="$(( $count + 1 ))"
      else
        nonex="$(( $nonex + 1 ))"
      fi
    done
  fi
done

echo "$count commands, and $nonex entries that weren't executable"

exit 0
```

*清单 2-1：计算当前`*PATH*`中可执行文件和非可执行文件的数量*

这个脚本计算的是可执行文件的数量，而不仅仅是文件的数量，它可以用来揭示许多流行操作系统默认 `PATH` 变量中有多少命令和非可执行文件（见表 2-1）。

**表 2-1：** 各操作系统的典型命令数量

| **操作系统** | **命令数** | **非可执行文件数** |
| --- | --- | --- |
| Ubuntu 15.04（包括所有开发者库） | 3,156 | 5 |
| OS X 10.11（安装了开发者选项） | 1,663 | 11 |
| FreeBSD 10.2 | 954 | 4 |
| Solaris 11.2 | 2,003 | 15 |

显然，不同版本的 Linux 和 Unix 提供了大量的命令和可执行脚本。为什么会有这么多？答案基于 Unix 的基本哲学：命令应该做一件事，并且做得好。具有拼写检查、文件查找和电子邮件功能的文字处理器在 Windows 和 Mac 环境中可能运作良好，但在命令行中，每个功能都应该是独立且离散的。

这种设计哲学有很多优点，最重要的一点是每个功能都可以单独修改和扩展，从而让所有使用它的应用程序都能获得这些新功能。无论你想在 Unix 上执行什么任务，通常都能轻松组合出一些能解决问题的东西，无论是通过下载添加新功能的实用工具、创建一些别名，还是稍微接触一下 shell 脚本的世界。

本书中的脚本不仅有帮助，而且是 Unix 哲学的逻辑延伸。毕竟，比起为自己的安装构建复杂且不兼容的命令版本，扩展和扩展现有功能要好得多。

本章探讨的脚本与清单 2-1 中的脚本相似，它们添加了有趣或有用的功能和特性，同时保持较低的复杂度。一些脚本接受不同的命令标志，以提供更大的灵活性，而一些脚本还展示了如何将一个 shell 脚本作为 *包装器*，一个插入程序，允许用户以一种通用的表示法指定命令或命令标志，然后将这些标志转换成实际 Unix 命令所需的正确格式和语法。

### #14 格式化长行

如果幸运的话，你的 Unix 系统已经包含了 `fmt` 命令，这对于经常处理文本的用户来说是一个非常有用的程序。从重新格式化电子邮件到使行文本充满文档中所有可用宽度，`fmt` 是一个值得了解的实用工具。

然而，一些 Unix 系统并不包含 `fmt`。这在老旧系统中尤其常见，这些系统通常有着相对简化的实现。

事实证明，`nroff` 命令自 Unix 初期便已存在，并且本身就是一个 shell 脚本封装器，可以在短小的 shell 脚本中使用，用于包装长行并填充短行以平衡行长，如在列表 2-2 中所示。

#### *代码部分*

```
   #!/bin/bash

   # fmt--Text formatting utility that acts as a wrapper for nroff
   #   Adds two useful flags: -w X for line width
   #   and -h to enable hyphenation for better fills
➊ while getopts "hw:" opt; do
     case $opt in
       h ) hyph=1              ;;
       w ) width="$OPTARG"     ;;
     esac
   done
➋ shift $(($OPTIND - 1))

➌ nroff << EOF
➍ .ll ${width:-72}
   .na
   .hy ${hyph:-0}
   .pl 1
➎ $(cat "$@")
   EOF

   exit 0
```

*列表 2-2：用于良好格式化长文本的 `*fmt*` shell 脚本*

#### *工作原理*

这个简洁的脚本提供了两个不同的命令标志：`-w X` 用于指定当行长度超过 `X` 字符时进行换行（默认值为 72），`-h` 用于启用跨行的连字符断词。请注意，在➊处检查标志。`while` 循环使用 `getopts` 一次读取传递给脚本的每个选项，内层的 `case` 块决定如何处理这些选项。一旦选项被解析，脚本会在➋调用 `shift` 丢弃所有选项标志，使用 `$OPTIND`（它保存着 `getopts` 要读取的下一个参数的索引），并将剩余的参数继续处理。

这个脚本还使用了 *here document*（在脚本 #9 中有讨论，在第 34 页），这是一种可以向命令提供多行输入的代码块类型。通过这种书写便捷方式，脚本在 ➌ 处将所有必要的命令传递给 `nroff`，以实现预期输出。在本文档中，我们使用了一种 bash 语法替代了一个未定义的变量 ➍，以便为用户未指定参数时提供一个合理的默认值。最后，脚本调用了 `cat` 命令，处理请求的文件名。为了完成任务，`cat` 命令的输出也会直接传递给 `nroff` ➎。这是一种在本书中会频繁出现的技巧。

#### *运行脚本*

这个脚本可以直接从命令行调用，但更可能作为外部管道的一部分，从 `vi` 或 `vim` 这样的编辑器内调用（例如，`!}fmt`）来格式化一段文本。

#### *结果*

列表 2-3 启用了连字符处理，并指定了最大宽度为 50 字符。

```
$ fmt -h -w 50 014-ragged.txt
So she sat on, with closed eyes, and half believed
herself in Wonderland, though she knew she had but
to open them again, and all would change to dull
reality--the grass would be only rustling in the
wind, and the pool rippling to the waving of the
reeds--the rattling teacups would change to tin-
kling sheep-bells, and the Queen's shrill cries
to the voice of the shepherd boy--and the sneeze
of the baby, the shriek of the Gryphon, and all
the other queer noises, would change (she knew) to
the confused clamour of the busy farm-yard--while
the lowing of the cattle in the distance would
take the place of the Mock Turtle's heavy sobs.
```

*列表 2-3：使用 `*fmt*` 脚本按 50 字符的宽度换行并进行连字符处理*

将列表 2-3（注意第 6 行和第 7 行突出显示的已连字符化的单词 `tinkling`）与使用默认宽度且没有连字符处理的列表 2-4 的输出进行比较。

```
$ fmt 014-ragged.txt
So she sat on, with closed eyes, and half believed herself in
Wonderland, though she knew she had but to open them again, and all
would change to dull reality--the grass would be only rustling in the
wind, and the pool rippling to the waving of the reeds--the rattling
teacups would change to tinkling sheep-bells, and the Queen's shrill
cries to the voice of the shepherd boy--and the sneeze of the baby, the
shriek of the Gryphon, and all the other queer noises, would change (she
knew) to the confused clamour of the busy farm-yard--while the lowing of
the cattle in the distance would take the place of the Mock Turtle's
heavy sobs.
```

*列表 2-4：没有连字符处理的 `*fmt*` 脚本的默认格式化*

### #15 备份文件在删除时

Unix 用户最常遇到的问题之一是，没有简单的办法恢复一个不小心删除的文件或文件夹。没有像 Undelete 360、WinUndelete 或 OS X 工具那样的用户友好应用，可以让你轻松浏览和恢复已删除的文件，只需按一个按钮。一旦你按下回车键，输入`rm *filename*`，文件就永远消失了。

解决这个问题的一种方法是将文件和目录秘密且自动地归档到一个*.deleted-files*归档中。通过脚本中的一些巧妙操作（如清单 2-5 所示），这个过程几乎可以对用户完全隐形。

#### *代码*

```
   #!/bin/bash

   # newrm--A replacement for the existing rm command.
   #   This script provides a rudimentary unremove capability by creating and
   #   utilizing a new directory within the user's home directory. It can handle
   #   directories of content as well as individual files. If the user specifies
   #   the -f flag, files are removed and NOT archived.

   # Big Important Warning: You'll want a cron job or something similar to keep
   #   the trash directories tamed. Otherwise, nothing will ever actually
   #   be deleted from the system, and you'll run out of disk space!

   archivedir="$HOME/.deleted-files"
   realrm="$(which rm)"
   copy="$(which cp) -R"

   if [ $# -eq 0 ] ; then            # Let 'rm' output the usage error.
     exec $realrm                    # Our shell is replaced by /bin/rm.
   fi

   # Parse all options looking for '-f'

   flags=""

   while getopts "dfiPRrvW" opt
   do
     case $opt in
       f ) exec $realrm "$@"     ;;  # exec lets us exit this script directly.
       * ) flags="$flags -$opt"  ;;  # Other flags are for rm, not us.
     esac
   done
   shift $(( $OPTIND - 1 ))

   # BEGIN MAIN SCRIPT
   # =================

   # Make sure that the $archivedir exists.

➊ if [ ! -d $archivedir] ; then
     if [ ! -w $HOME ] ; then
       echo "$0 failed: can't create $archivedir in $HOME" >&2
       exit 1
     fi
     mkdir $archivedir
➋   chmod 700 $archivedir           # A little bit of privacy, please.
   fi

   for arg
   do
➌   newname="$archivedir/$(date "+%S.%M.%H.%d.%m").$(basename "$arg")"
     if [ -f "$arg" -o -d "$arg" ] ; then
       $copy "$arg" "$newname"
     fi
   done

➍ exec $realrm $flags "$@"          # Our shell is replaced by realrm.
```

*清单 2-5：* `*newrm*` *shell 脚本，它在文件从磁盘中删除之前进行备份*

#### *工作原理*

这个脚本中有许多值得注意的地方，其中最显著的是它确保用户不会意识到它的存在。例如，当脚本无法工作时，它不会生成错误信息；它只是让`realrm`生成错误信息，通常是通过调用可能包含错误参数的*/bin/rm*。对`realrm`的调用是通过`exec`命令完成的，该命令用指定的新进程替换当前进程。一旦`exec`调用`realrm` ➍，它实际上就退出了这个脚本，并且`realrm`进程的返回码会传递给调用的 shell。

因为这个脚本会在用户的主目录中秘密创建一个目录 ➊，所以它需要确保该目录中的文件不会因为不正确设置的`umask`值而突然对其他人可读。（`umask`值定义了新创建的文件或目录的默认权限。）为了避免这种过度共享，脚本在 ➋ 处使用`chmod`来确保该目录设置为对用户可读/写/执行，并且对其他人关闭权限。

最终在 ➌ 处，脚本使用`basename`去除文件路径中的任何目录信息，并且为每个已删除的文件添加一个日期和时间戳，格式为*秒.分钟.小时.天.月.文件名*：

```
newname="$archivedir/$(date "+"%S.%M.%H.%d.%m").$(basename "$arg")"
```

请注意在同一替换中使用多个`$( )`元素。虽然这可能有些复杂，但仍然很有帮助。记住，任何在`$(`和`)`之间的内容都会被送入子 shell 中执行，然后整个表达式会被该命令的结果替代。

那么为什么还要使用时间戳呢？是为了支持存储多个具有相同名称的已删除文件。一旦文件被归档，脚本不再区分*/home/oops.txt*和*/home/subdir/oops.txt*，除了它们被删除的时间。如果多个同名文件同时被删除（或者在同一秒内删除），先被归档的文件将被覆盖。解决这个问题的一种方法是将原始文件的绝对路径添加到归档文件名中。

#### *运行脚本*

要安装这个脚本，可以添加一个别名，使得当你输入 `rm` 时，实际上运行的是这个脚本，而不是 `/bin/rm` 命令。一个 bash 或 ksh 的别名可能是这样的：

```
alias rm=yourpath/newrm
```

#### *结果*

运行这个脚本的结果是故意隐藏的（正如列表 2-6 所示），所以我们一路上要关注 *.deleted-files* 目录。

```
$ ls ~/.deleted-files
ls: /Users/taylor/.deleted-files/: No such file or directory
$ newrm file-to-keep-forever
$ ls ~/.deleted-files/
51.36.16.25.03.file-to-keep-forever
```

*列表 2-6：测试* `*newrm*` *shell 脚本*

完全正确。虽然文件已从本地目录中删除，但它的副本被秘密地存放在 *.deleted-files* 目录中。时间戳允许其他同名的已删除文件存储在同一目录中，而不会互相覆盖。

#### *破解脚本*

一个有用的改动是更改时间戳，使其按逆时间顺序排列，从而按照时间顺序显示 `ls` 的文件列表。下面是修改脚本的代码：

```
newname="$archivedir/$(date "+"%S.%M.%H.%d.%m").$(basename "$arg")"
```

你可以反转该格式化请求中令牌的顺序，使得原始文件名排在前，日期排在备份文件名的后面。然而，由于我们的时间粒度是秒，你可能会在同一秒内删除多个版本的同名文件（例如，`rm test testdir/test`），这会导致两个同名文件。因此，另一个有用的修改是将文件的存储位置加入到归档副本中。例如，这会生成 *timestamp.test* 和 *timestamp.testdir.test*，它们显然是两个不同的文件。

### #16 处理已删除文件归档

现在，已删除文件的目录隐藏在用户的主目录中，一个让用户在不同版本的已删除文件之间选择的脚本将非常有用。然而，要处理所有可能的情况是相当复杂的，从完全找不到指定文件，到找到多个匹配给定条件的已删除文件。例如，如果有多个匹配项，脚本应该自动选择最新的文件来恢复吗？抛出一个错误，指示有多少个匹配项？还是展示不同版本并让用户选择？让我们看看列表 2-7，它详细介绍了 `unrm` 脚本。

#### *代码*

```
   #!/bin/bash

   # unrm--Searches the deleted files archive for the specified file or
   #   directory. If there is more than one matching result, it shows a list
   #   of results ordered by timestamp and lets the user specify which one
   #   to restore.

   archivedir="$HOME/.deleted-files"
   realrm="$(which rm)"
   move="$(which mv)"

   dest=$(pwd)

   if [ ! -d $archivedir ] ; then
     echo "$0: No deleted files directory: nothing to unrm" >&2
     exit 1
   fi
 cd $archivedir

   # If given no arguments, just show a listing of the deleted files.
➊ if [ $# -eq 0 ] ; then
     echo "Contents of your deleted files archive (sorted by date):"
➋   ls -FC | sed -e 's/\([[:digit:]][[:digit:]]\.\)\{5\}//g' \
       -e 's/^/ /'
     exit 0
   fi

   # Otherwise, we must have a user-specified pattern to work with.
   #   Let's see if the pattern matches more than one file or directory
   #   in the archive.

➌ matches="$(ls -d *"$1" 2> /dev/null | wc -l)"

   if [ $matches -eq 0 ] ; then
     echo "No match for \"$1\" in the deleted file archive." >&2
     exit 1
   fi

➍ if [ $matches -gt 1 ] ; then
     echo "More than one file or directory match in the archive:"
     index=1
     for name in $(ls -td *"$1")
     do
       datetime="$(echo $name | cut -c1-14| \
➎       awk -F. '{ print $5"/"$4" at "$3":"$2":"$1 }')"
       filename="$(echo $name | cut -c16-)"
       if [ -d $name ] ; then
➏      filecount="$(ls $name | wc -l | sed 's/[^[:digit:]]//g')"
         echo " $index) $filename (contents = ${filecount} items," \
              " deleted = $datetime)"
       else
➐       size="$(ls -sdk1 $name | awk '{print $1}')"
         echo " $index) $filename (size = ${size}Kb, deleted = $datetime)"
       fi
       index=$(( $index + 1))
     done
     echo ""
     /bin/echo -n "Which version of $1 should I restore ('0' to quit)? [1] : "
     read desired
     if [ ! -z "$(echo $desired | sed 's/[[:digit:]]//g')" ] ; then
       echo "$0: Restore canceled by user: invalid input." >&2
       exit 1
     fi

     if [ ${desired:=1} -ge $index ] ; then
       echo "$0: Restore canceled by user: index value too big." >&2
       exit 1
     fi

 if [ $desired -lt 1 ] ; then
       echo "$0: Restore canceled by user." >&2
       exit 1
     fi

➑   restore="$(ls -td1 *"$1" | sed -n "${desired}p")"

➒   if [ -e "$dest/$1" ] ; then
       echo "\"$1\" already exists in this directory. Cannot overwrite." >&2
       exit 1
     fi

     /bin/echo -n "Restoring file \"$1\" ..."
     $move "$restore" "$dest/$1"
     echo "done."

➓   /bin/echo -n "Delete the additional copies of this file? [y] "
     read answer

     if [ ${answer:=y} = "y" ] ; then
       $realrm -rf *"$1"
       echo "Deleted."
     else
       echo "Additional copies retained."
     fi
   else
     if [ -e "$dest/$1" ] ; then
       echo "\"$1\" already exists in this directory. Cannot overwrite." >&2
       exit 1
     fi

     restore="$(ls -d *"$1")"

     /bin/echo -n "Restoring file \"$1\" ... "
     $move "$restore" "$dest/$1"
     echo "Done."
   fi

   exit 0
```

*列表 2-7：恢复备份文件的* `*unrm*` *shell 脚本*

#### *原理*

在➊处的第一段代码，`if [$# -eq 0]` 条件块，会在没有指定参数时执行，显示已删除文件的归档内容。然而，这里有一个问题：我们不希望向用户展示我们添加到文件名中的时间戳数据，因为这些数据仅供脚本内部使用，展示出来会让输出显得杂乱。为了以更吸引人的格式显示这些数据，➋处的 `sed` 语句会删除 `ls` 输出中前五个 *数字 数字 点* 的出现。

用户可以通过指定文件或目录的名称作为参数来恢复该文件或目录。接下来的步骤在➌处是确定所提供名称的匹配项数量。

这一行中嵌套双引号的特殊用法（围绕`$1`）是为了确保`ls`匹配包含空格的文件名，同时`*`通配符将匹配扩展到包括任何前置的时间戳。`2> /dev/null`序列用于丢弃命令产生的任何错误，而不是将其显示给用户。被丢弃的错误很可能是*没有此文件或目录*，当指定的文件名未找到时会出现此错误。

如果给定的文件或目录名有多个匹配项，则脚本中最复杂的部分，即在➍处的`if [ $matches -gt 1 ]`块将被执行，并显示所有结果。主`for`循环中使用`ls`命令的`-t`标志，使得归档文件按从最新到最旧的顺序显示，而在➎处，通过简洁地调用`awk`命令，将文件名中的时间戳部分转换为括号中的删除日期和时间。在➐处的大小计算中，通过给`ls`命令添加`-k`标志，强制文件大小以千字节为单位表示。

脚本并不显示匹配目录条目的大小，而是显示每个匹配目录中包含的文件数量，这是一个更有用的统计信息。计算目录中条目的数量很容易。在➏处，我们只需要计算`ls`给出的行数，并将`wc`的输出中的空格去掉。

一旦用户指定了一个可能的匹配文件或目录，具体的文件将在➑处被识别。这条语句使用了稍微不同的`sed`用法。指定`-n`标志并使用行号（`${desired}`）后跟`p`（打印）命令，是从输入流中快速提取指定行的方式。想只看第 37 行？命令`sed -n 37p`就是这么做的。

然后，在➒处进行测试，以确保`unrm`不会覆盖现有的文件副本，并通过调用`/bin/mv`来恢复文件或目录。完成后，用户将有机会删除额外的（可能是多余的）文件副本➓，脚本执行完毕。

请注意，使用 `ls` 配合 `*"$1"` 可以匹配任何以 `$1` 中的值结尾的文件名，因此多个“匹配文件”的列表可能包含不仅仅是用户想要恢复的文件。例如，如果删除的文件目录中包含文件 *11.txt* 和 *111.txt*，运行 `unrm 11.txt` 将提示找到多个匹配项，并返回 *11.txt* 和 *111.txt* 的列表。虽然这可能没问题，但一旦用户选择恢复正确的文件（*11.txt*），接受提示删除其他副本时，也会删除 *111.txt*。因此，在这种情况下默认删除可能并不是最优选择。然而，如果你像脚本 #15 中所示的那样保持相同的时间戳格式，改用 `??.??.??.??.??."$1"` 模式就可以轻松解决这个问题，如第 55 页所示。

#### *运行脚本*

有两种方式可以运行这个脚本。没有任何参数时，脚本会显示用户删除的文件归档中所有文件和目录的列表。当提供一个文件名作为参数时，脚本会尝试恢复该文件或目录（如果只有一个匹配项），或者会显示候选恢复文件的列表，并允许用户指定要恢复的删除文件或目录的版本。

#### *结果*

在没有指定任何参数的情况下，脚本会显示删除文件归档中的内容，如清单 2-8 所示。

```
$ unrm
Contents of your deleted files archive (sorted by date):
  detritus            this is a test
  detritus            garbage
```

*清单 2-8：运行没有参数的* `*unrm*` *shell 脚本列出当前可恢复的文件*

当指定了文件名时，如果有多个同名文件，脚本会显示更多关于该文件的信息，如清单 2-9 所示。

```
$ unrm detritus
More than one file or directory match in the archive:
 1)   detritus (size = 7688Kb, deleted = 11/29 at 10:00:12)
 2)   detritus  (size = 4Kb, deleted = 11/29 at 09:59:51)

Which version of detritus should I restore ('0' to quit)? [1] : 0
unrm: Restore canceled by user.
```

*清单 2-9：运行带有单个参数的* `*unrm*` *shell 脚本尝试恢复文件*

#### *破解脚本*

如果你使用这个脚本，请注意，由于没有任何控制或限制，删除文件归档中的文件和目录将无限增长。为了避免这种情况，可以在 `cron` 作业中调用 `find` 来修剪删除文件归档，使用 `-mtime` 标志来识别那些几周没有被触碰的文件。对于大多数用户来说，14 天的归档时间应该足够，并且可以防止归档脚本占用过多的磁盘空间。

在我们讨论这些时，实际上有一些改进可以使这个脚本更加用户友好。可以考虑添加像 `-l` 来恢复最新文件，或者 `-D` 来删除多余副本等启动标志。你会添加哪些标志？它们会如何简化处理流程？

### #17 记录文件删除

如果你不想归档已删除的文件，你也许只想追踪系统上发生的删除事件。在 Listing 2-10 中，使用 `rm` 命令删除的文件会被记录到一个单独的文件中，而不会通知用户。这可以通过使用脚本作为封装器来实现。封装器的基本理念是，它们位于实际的 Unix 命令和用户之间，提供原始命令无法单独提供的有用功能。

**注意**

*封装器是一个非常强大的概念，随着你深入本书，你会发现它们反复出现。*

#### *代码*

```
   #!/bin/bash
   # logrm--Logs all file deletion requests unless the -s flag is used

   removelog="/var/log/remove.log"

➊ if [ $# -eq 0 ] ; then
     echo "Usage: $0 [-s] list of files or directories" >&2
     exit 1
   fi

➋ if [ "$1" = "-s" ] ; then
     # Silent operation requested ... don't log.
     shift
   else
➌   echo "$(date): ${USER}: $@" >> $removelog
   fi

➍ /bin/rm "$@"

   exit 0
```

*Listing 2-10: *`*logrm*`* *shell 脚本*

#### *工作原理*

第一部分 ➊ 测试用户输入，如果没有给定参数，则生成一个简单的文件列表。然后在 ➋，脚本测试参数 `1` 是否为 `-s`；如果是，它会跳过删除请求的日志记录。最后，时间戳、用户和命令会被添加到 *$removelog* 文件中 ➌，并且用户的命令会被静默地传递给真正的 */bin/rm* 程序 ➍。

#### *运行脚本*

与其给这个脚本命名为 `logrm`，一个典型的封装程序安装方式是重命名它所封装的底层命令，然后使用原始命令的旧名称来安装封装器。然而，如果你选择这种方式，请确保封装器调用的是新重命名的程序，而不是它自己！例如，如果你将 */bin/rm* 重命名为 */bin/rm.old*，并将这个脚本命名为 */bin/rm*，那么脚本的最后几行需要进行更改，以便它调用的是 */bin/rm.old* 而不是它自己。

另外，你可以使用别名将标准的 `rm` 命令替换为这个命令：

```
alias rm=logrm
```

在任何情况下，你都需要对 */var/log* 目录具有写入和执行权限，这可能不是你系统上的默认配置。

#### *结果*

让我们创建几个文件，删除它们，然后查看删除日志，如 Listing 2-11 所示。

```
$ touch unused.file ciao.c /tmp/junkit
$ logrm unused.file /tmp/junkit
$ logrm ciao.c
$ cat /var/log/remove.log
Thu Apr  6 11:32:05 MDT 2017: susan: /tmp/central.log
Fri Apr  7 14:25:11 MDT 2017: taylor: unused.file /tmp/junkit
Fri Apr  7 14:25:14 MDT 2017: taylor: ciao.c
```

*Listing 2-11: 测试* `*logrm*` *shell 脚本*

啊哈！注意到周四，用户 Susan 删除了文件 */tmp/central.log*。

#### *破解脚本*

这里可能会遇到日志文件的所有权权限问题。要么 *remove.log* 文件对所有人可写，在这种情况下，用户可以使用类似 `cat /dev/null > /var/log/remove.log` 的命令清空其内容，要么该文件对所有人不可写，在这种情况下，脚本无法记录事件。你可以使用 `setuid` 权限——脚本以 root 用户身份运行——这样脚本就会与日志文件具有相同的权限。然而，这种方法有两个问题。首先，这是一个非常糟糕的主意！绝对不要在 `setuid` 下运行 shell 脚本！通过使用 `setuid` 以特定用户身份运行命令，无论是谁执行该命令，都可能会给系统带来安全隐患。其次，可能会出现用户可以删除自己的文件，但脚本却无法删除的情况，因为 `setuid` 设置的有效用户 ID 会被 `rm` 命令继承，导致系统出错。当用户甚至无法删除自己的文件时，系统将陷入混乱！

如果你使用的是 ext2、ext3 或 ext4 文件系统（通常是 Linux 系统），另一种解决方案是使用 `chattr` 命令在日志文件上设置一个特定的仅追加文件权限，然后让所有人都可以写入而不会有任何危险。另一种解决方案是将日志信息写入 `syslog`，使用便捷的 `logger` 命令。使用 `logger` 记录 `rm` 命令是非常简单的，下面是示例：

```
logger -t logrm "${USER:-LOGNAME}: $*"
```

这会向 `syslog` 数据流中添加一个条目，普通用户无法触及，该条目标记为 `logrm`，包括用户名和指定的命令。

**注意**

*如果你选择使用* `*logger*` *，你需要检查* `*syslogd(8)*` *以确保你的配置不会丢弃* `*user.notice*` *优先级的日志事件。它几乎总是会在* /etc/syslogd.conf *文件中指定。*

### #18 显示目录内容

`ls` 命令的一个方面一直让人觉得毫无意义：当列出一个目录时，`ls` 要么逐个列出目录中的文件，要么显示目录数据所需的 1,024 字节块数。`ls -l` 输出中的一个典型条目可能是这样的：

```
drwxrwxr-x    2 taylor   taylor        4096 Oct 28 19:07 bin
```

但这并不太有用！我们真正想知道的是目录中有多少个文件。这就是 Listing 2-12 中脚本的作用。它生成了一个多列文件和目录的清单，显示文件的大小以及包含的文件数。

#### *代码*

```
   #!/bin/bash

   # formatdir--Outputs a directory listing in a friendly and useful format

   # Note that you need to ensure "scriptbc" (Script #9) is in your current path
   #   because it's invoked within the script more than once.

   scriptbc=$(which scriptbc)

   # Function to format sizes in KB to KB, MB, or GB for more readable output
➊ readablesize()
   {

     if [ $1 -ge 1048576 ] ; then
       echo "$($scriptbc -p 2 $1 / 1048576)GB"
     elif [ $1 -ge 1024 ] ; then
       echo "$($scriptbc -p 2 $1 / 1024)MB"
     else
       echo "${1}KB"
     fi
   }

   #################
   ## MAIN CODE

   if [ $# -gt 1 ] ; then
     echo "Usage: $0 [dirname]" >&2
     exit 1
➋ elif [ $# -eq 1 ] ; then   # Specified a directory other than the current one?
     cd "$@"                  # Then let's change to that one.
     if [ $? -ne 0 ] ; then   # Or quit if the directory doesn't exist.
       exit 1
     fi
   fi

   for file in *
   do
     if [ -d "$file" ] ; then
➌     size=$(ls "$file" | wc -l | sed 's/[^[:digit:]]//g')
       if [ $size -eq 1 ] ; then
         echo "$file ($size entry)|"
       else
         echo "$file ($size entries)|"
       fi
     else
       size="$(ls -sk "$file" | awk '{print $1}')"
➍     echo "$file ($(readablesize $size))|"
     fi
   done | \
➎   sed 's/ /^^^/g' | \
     xargs -n 2 | \
     sed 's/\^\^\^/ /g' | \
➏   awk -F\| '{ printf "%-39s %-39s\n", $1, $2 }'

   exit 0
```

*Listing 2-12: 更具可读性的目录列出脚本* `*formatdir*`

#### *原理*

这个脚本最有趣的部分之一是 `readablesize` 函数 ➊，它接受以千字节为单位的数字，并根据最合适的单位输出其值，可能是千字节、兆字节或吉字节。例如，代替将一个非常大的文件的大小显示为 2,083,364KB，该函数会将其显示为 2.08GB。请注意，`readablesize` 是通过 `$( )` 语法 ➍ 调用的：

```
echo "$file ($(readablesize $size))|"
```

由于子 shell 会自动继承运行中的 shell 中定义的所有函数，因此通过 `$()` 语法创建的子 shell 可以访问 `readablesize` 函数，非常方便。

在脚本的顶部 ➋，还有一个快捷方式，允许用户指定一个不同于当前目录的目录，然后通过使用 `cd` 命令将运行中的 shell 脚本的当前工作目录更改为所需位置。

这个脚本的主要逻辑是将输出组织成两列整齐对齐的形式。需要处理的一个问题是，不能简单地将空格替换为换行符，因为文件和目录的名称中可能包含空格。为了解决这个问题，脚本在 ➎ 首先将每个空格替换为三个插入符号（`^^^`）的序列。然后，它使用 `xargs` 命令合并配对的行，使得每一对行变成一行，并通过一个真实的、预期的空格分隔。最后，在 ➏ 它使用 `awk` 命令输出列并正确对齐。

注意，通过在 ➌ 使用 `wc` 快速调用并结合 `sed` 命令清理输出，可以轻松计算出目录中（非隐藏）条目的数量：

```
size=$(ls "$file" | wc -l | sed 's/[^[:digit:]]//g')
```

#### *运行脚本*

要列出当前目录，执行没有参数的命令，正如 列表 2-13 所示。若要查看其他目录的内容，只需指定目录名作为唯一的命令行参数。

#### *结果*

```
$ formatdir ~
Applications (0 entries)                Classes (4KB)
DEMO (5 entries)                        Desktop (8 entries)
Documents (38 entries)                  Incomplete (9 entries)
IntermediateHTML (3 entries)            Library (38 entries)
Movies (1 entry)                        Music (1 entry)
NetInfo (9 entries)                     Pictures (38 entries)
Public (1 entry)                        RedHat 7.2 (2.08GB)
Shared (4 entries)                      Synchronize! Volume ID (4KB)
X Desktop (4KB)                         automatic-updates.txt (4KB)
bin (31 entries)                        cal-liability.tar.gz (104KB)
cbhma.tar.gz (376KB)                    errata (2 entries)
fire aliases (4KB)                      games (3 entries)
junk (4KB)                              leftside navbar (39 entries)
mail (2 entries)                        perinatal.org (0 entries)
scripts.old (46 entries)                test.sh (4KB)
testfeatures.sh (4KB)                   topcheck (3 entries)
tweakmktargs.c (4KB)                    websites.tar.gz (18.85MB)
```

*列表 2-13：测试* `*formatdir*` *shell 脚本*

#### *破解脚本*

一个值得考虑的问题是，是否有用户喜欢在文件名中使用三个插入符号（^）。这种命名规范相当不太可能出现——我们在一次对 116,696 个文件的 Linux 安装的测试中，发现它的文件名中甚至没有一个插入符号——但如果确实发生了，你将得到一些令人困惑的输出。如果你担心，可以通过将空格转化为另一种字符序列来解决这个潜在的问题，这样的字符序列在用户文件名中发生的可能性会更小。四个插入符号？五个？

### #19 通过文件名定位文件

在 Linux 系统中有一个非常有用的命令，但并不是所有 Unix 系统都具备，它就是 `locate`，它可以在一个预构建的文件名数据库中搜索用户指定的正则表达式。曾经想快速找到主 `*.cshrc*` 文件的位置吗？这就是使用 `locate` 来完成的方式：

![image](img/f0068-01.jpg)

你可以看到，主*.cshrc*文件位于这台 OS X 系统的*/private/etc*目录中。我们将要构建的`locate`版本在构建内部文件索引时，能够看到磁盘上的每一个文件，无论该文件是否在回收站队列中，是否在单独的卷中，甚至是否是隐藏的点文件。这既是一个优势，也可能是一个劣势，我们将稍后讨论。

#### *代码*

这种查找文件的方法简单易行，分为两个脚本。第一个（见列表 2-14）通过调用`find`来构建所有文件名的数据库，第二个（见列表 2-15）则是对新数据库进行简单的`grep`查找。

```
   #!/bin/bash

   # mklocatedb--Builds the locate database using find. User must be root
   #   to run this script.

   locatedb="/var/locate.db"

➊ if [ "$(whoami)" != "root" ] ; then
     echo "Must be root to run this command." >&2
     exit 1
   fi

   find / -print > $locatedb

   exit 0
```

*列表 2-14：`*mklocatedb*`* shell 脚本*

第二个脚本甚至更简短。

```
#!/bin/sh

# locate--Searches the locate database for the specified pattern

locatedb="/var/locate.db"

exec grep -i "$@" $locatedb
```

*列表 2-15：`*locate*`* shell 脚本*

#### *工作原理*

必须以 root 用户身份运行`mklocatedb`脚本，以确保它能够看到系统中的所有文件，因此在➊处通过调用`whoami`进行了检查。然而，以 root 身份运行任何脚本都是一个安全问题，因为如果某个目录对特定用户的访问被关闭，`locate`数据库就不应存储该目录及其内容的信息。这个问题将在第五章中通过一个新的、更安全的`locate`脚本来解决，该脚本考虑到了隐私和安全问题（详见脚本 #39，位于第 127 页）。不过，目前这个脚本完美模拟了标准 Linux、OS X 及其他发行版中`locate`命令的行为。

如果`mklocatedb`运行需要几分钟或更长时间，不必惊讶；它正在遍历整个文件系统，即使是中等大小的系统也可能需要一些时间。结果也可能非常庞大。在我们测试的一台 OS X 系统上，*locate.db*文件有超过 150 万个条目，占用了 1874.5MB 的磁盘空间。

一旦数据库构建完成，`locate`脚本本身就非常简单；它只是调用`grep`命令，带上用户指定的任何参数。

#### *运行脚本*

要运行`locate`脚本，首先需要运行`mklocatedb`。完成之后，`locate`调用几乎可以瞬间找到系统中所有符合指定模式的文件。

#### *结果*

`mklocatedb`脚本没有任何参数或输出，如列表 2-16 所示。

```
$ sudo mklocatedb
Password:
...
Much time passes
...
$
```

*列表 2-16：以 root 身份运行`*mklocatedb*`* shell 脚本，并使用`*sudo*`命令*

我们可以通过快速的`ls`命令检查数据库的大小，如下所示：

```
$ ls -l /var/locate.db
-rw-r--r--  1 root  wheel  174088165 Mar 26 10:02 /var/locate.db
```

现在，我们准备开始使用`locate`查找系统中的文件：

```
$ locate -i solitaire
/Users/taylor/Documents/AskDaveTaylor image folders/0-blog-pics/vista-search-
solitaire.png
/Users/taylor/Documents/AskDaveTaylor image folders/8-blog-pics/windows-play-
solitaire-1.png
/usr/share/emacs/22.1/lisp/play/solitaire.el.gz
/usr/share/emacs/22.1/lisp/play/solitaire.elc
/Volumes/MobileBackups/Backups.backupdb/Dave's MBP/2014-04-03-163622/BigHD/
Users/taylor/Documents/AskDaveTaylor image folders/0-blog-pics/vista-search-
solitaire.png
/Volumes/MobileBackups/Backups.backupdb/Dave's MBP/2014-04-03-163622/BigHD/
Users/taylor/Documents/AskDaveTaylor image folders/8-blog-pics/windows-play-
solitaire-3.png
```

这个脚本还可以让你获得关于系统的其他有趣统计信息，比如你有多少个 C 源文件，像这样：

```
$ locate '\.c$' | wc -l
  1479
```

**注意**

*请注意这里的正则表达式。* `*grep*` *命令要求我们转义点号（*`.`*），否则它会匹配任何单一字符。另外，* `*$*` *表示行尾，或者在这个情况下，表示文件名的结束。*

再做一点小调整，我们可以将每个 C 源文件都传递给 `wc` 命令，从而计算系统中 C 代码的总行数，但，嗯，这样做是不是有点傻呢？

#### *修改脚本*

为了保持数据库的合理更新，调度 `mklocatedb` 每周在深夜通过 `cron` 运行就非常简单——正如大多数内置 `locate` 命令的系统所做的那样——甚至可以根据本地的使用模式更频繁地运行。和任何由 root 用户执行的脚本一样，要确保该脚本本身不能被非 root 用户修改。

这个脚本的一个潜在改进是让 `locate` 检查其调用，如果没有指定模式或 *locate.db* 文件不存在，则失败并显示有意义的错误信息。现在按目前的写法，脚本只会输出一个标准的 `grep` 错误，这并没有多大用处。更重要的是，正如我们之前讨论过的，允许用户访问系统中所有文件名的列表，包括那些他们通常无法看到的文件，是一个重大的安全问题。对这个脚本的安全性改进可以参见 脚本 #39 和 第 127 页。

### #20 模拟其他环境：MS-DOS

虽然你可能永远不需要它们，但创建像 `DIR` 这样的经典 MS-DOS 命令的版本作为 Unix 兼容的 shell 脚本，这既有趣又能说明一些脚本编程概念。确实，我们可以仅仅使用 shell 别名将 `DIR` 映射到 Unix 的 `ls` 命令，像这个例子中那样：

```
alias DIR=ls
```

但这种映射并不能模拟命令的实际行为；它仅仅帮助健忘的人记住新的命令名称。如果你熟悉古老的计算机方式，你会记得 `/W` 选项会产生宽格式的列出。例如，但如果你现在将 `/W` 传递给 `ls` 命令，程序会抱怨说 `/W` 目录不存在。相反，以下在 清单 2-17 中的 `DIR` 脚本可以写成使其适应斜杠风格的命令标志。

#### *代码*

```
   #!/bin/bash
   # DIR--Pretends we're the DIR command in DOS and displays the contents
   #   of the specified file, accepting some of the standard DIR flags

   function usage
   {
   cat << EOF >&2
     Usage: $0 [DOS flags] directory or directories
     Where:
      /D           sort by columns
      /H           show help for this shell script
      /N           show long listing format with filenames on right
      /OD          sort by oldest to newest
      /O-D         sort by newest to oldest
      /P           pause after each screenful of information
      /Q           show owner of the file
      /S           recursive listing
      /W           use wide listing format
   EOF
     exit 1
   }

   #####################
   ### MAIN BLOCK

   postcmd=""
   flags=""
   while [ $# -gt 0 ]
   do
     case $1 in
       /D        ) flags="$flags -x"      ;;
       /H        ) usage                  ;;
➊     /[NQW]    ) flags="$flags -l"      ;;
       /OD       ) flags="$flags -rt"     ;;
       /O-D      ) flags="$flags -t"      ;;
       /P        ) postcmd="more"         ;;
       /S        ) flags="$flags -s"      ;;
               * ) # Unknown flag: probably a DIR specifier break;
                   #   so let's get out of the while loop.
     esac
     shift         # Processed flag; let's see if there's another.
   done

   # Done processing flags; now the command itself:

   if [ ! -z "$postcmd" ] ; then
     ls $flags "$@" | $postcmd
   else
     ls $flags "$@"
   fi

   exit 0
```

*清单 2-17：* `*DIR*` *Shell 脚本，用于在 Unix 上模拟* `*DIR*` *DOS 命令*

#### *原理说明*

这个脚本突出了 shell `case` 语句中的条件测试实际上是正则表达式测试这一事实。你可以看到在 ➊ 位置，DOS 标志 `/N`、`/Q` 和 `/W` 都映射到最终调用 `ls` 命令时的同一个 `-l Unix` 标志，而所有这一切都通过一个简单的正则表达式 `/[NQW]` 实现。

#### *运行脚本*

将此脚本命名为`DIR`（并考虑创建一个系统范围的 shell 别名`dir=DIR`，因为 DOS 不区分大小写，而 Unix 确实区分大小写）。这样，每当用户在命令行输入带有典型 MS-DOS `DIR` 标志的 `DIR` 时，他们将得到有意义且有用的输出（如清单 2-18 所示），而不是一个`command not found`的错误信息。

#### *结果*

```
$ DIR /OD /S ~/Desktop
total 48320
 7720 PERP - Google SEO.pdf              28816 Thumbs.db
    0 Traffic Data                       8 desktop.ini
    8 gofatherhood-com-crawlerrors.csv   80 change-lid-close-behavior-win7-1.png
   16 top-100-errors.txt                 176 change-lid-close-behavior-win7-2.png
    0 $RECYCLE.BIN                       400 change-lid-close-behavior-win7-3.png
    0 Drive Sunshine                     264 change-lid-close-behavior-win7-4.png
   96 facebook-forcing-pay.jpg           32 change-lid-close-behavior-win7-5.png
10704 WCSS Source Files
```

*清单 2-18：测试`*DIR*` *shell 脚本列出文件*

该指定目录的列出，按从最旧到最新的顺序排序，显示文件大小（尽管目录的大小始终为 0）。

#### *修改脚本*

到这个阶段，可能很难找到一个记得 MS-DOS 命令行的人，但基本概念是强大的，值得了解。例如，你可以做的一个改进是，先显示 Unix 或 Linux 等效的命令，再执行，然后在经过一定次数的系统调用后，脚本显示翻译的命令，但实际上不执行命令。用户将被迫学习新命令，才能完成任何事情！

### #21 显示不同时间区的时间

`date`命令的最基本要求是能够在你的时区内显示日期和时间。但如果你有跨多个时区的用户呢？或者，更可能的是，如果你有分布在不同地点的朋友和同事，而你总是弄不清楚比如卡萨布兰卡、梵蒂冈或悉尼的时间呢？

事实证明，大多数现代 Unix 系统上的`date`命令是建立在一个令人惊叹的时区数据库之上的。这个数据库通常存储在目录*/usr/share/zoneinfo*中，列出了超过 600 个地区，并为每个地区详细列出了与 UTC（协调世界时，通常也称为*格林尼治标准时间*，或*GMT*）的时区偏移。`date`命令会关注`TZ`时区变量，我们可以将其设置为数据库中的任何地区，例如：

```
$ TZ="Africa/Casablanca" date
Fri Apr  7 16:31:01 WEST 2017
```

然而，大多数系统用户不习惯指定临时的环境变量设置。通过使用一个 shell 脚本，我们可以为时区数据库创建一个更加用户友好的前端。

清单 2-19 中的大部分脚本涉及在时区数据库中查找（该数据库通常分布在*zonedir*目录中的多个文件里），并尝试找到一个匹配指定模式的文件。一旦找到匹配的文件，脚本会抓取完整的时区名称（例如此例中的`TZ="Africa/Casablanca"`），并以此作为子 shell 环境设置调用`date`命令。`date`命令会检查`TZ`来查看当前的时区，并不知道它是一个一次性使用的时区还是你平时所处的时区。

#### *代码*

```
   #!/bin/bash

   # timein--Shows the current time in the specified time zone or
   #   geographic zone. Without any argument, this shows UTC/GMT.
   #   Use the word "list" to see a list of known geographic regions.
   #   Note that it's possible to match zone directories (regions),
   #   but that only time zone files (cities) are valid specifications.

   # Time zone database ref: http://www.twinsun.com/tz/tz-link.htm

   zonedir="/usr/share/zoneinfo"

   if [ ! -d $zonedir ] ; then
     echo "No time zone database at $zonedir." >&2
     exit 1
   fi

   if [ -d "$zonedir/posix" ] ; then
     zonedir=$zonedir/posix        # Modern Linux systems
   fi

   if [ $# -eq 0 ] ; then
     timezone="UTC"
     mixedzone="UTC"
➊ elif [ "$1" = "list" ] ; then
     ( echo "All known time zones and regions defined on this system:"
       cd $zonedir
       find -L * -type f -print | xargs -n 2 | \
         awk '{ printf " %-38s %-38s\n", $1, $2 }'
     ) | more
     exit 0
   else

     region="$(dirname $1)"
     zone="$(basename $1)"

     # Is the given time zone a direct match? If so, we're good to go.
     #   Otherwise we need to dig around a bit to find things. Start by
     #   just counting matches.

     matchcnt="$(find -L $zonedir -name $zone -type f -print |\
           wc -l | sed 's/[^[:digit:]]//g' )"

     # Check if at least one file matches.
     if [ "$matchcnt" -gt 0 ] ; then
       # But exit if more than one file matches.
       if [ $matchcnt -gt 1 ] ; then
         echo "\"$zone\" matches more than one possible time zone record." >&2
         echo "Please use 'list' to see all known regions and time zones." >&2
         exit 1
         fi
         match="$(find -L $zonedir -name $zone -type f -print)"
         mixedzone="$zone"
       else # Maybe we can find a matching time zone region, rather than a specific
            #   time zone.
         # First letter capitalized, rest of word lowercase for region + zone
         mixedregion="$(echo ${region%${region#?}} \
                  | tr '[[:lower:]]' '[[:upper:]]')\
                  $(echo ${region#?} | tr '[[:upper:]]' '[[:lower:]]')"
       mixedzone="$(echo ${zone%${zone#?}} | tr '[[:lower:]]' '[[:upper:]]') \
                  $(echo ${zone#?} | tr '[[:upper:]]' '[[:lower:]]')"

       if [ "$mixedregion" != "." ] ; then
         # Only look for specified zone in specified region
         #   to let users specify unique matches when there's
         #   more than one possibility (e.g., "Atlantic").
         match="$(find -L $zonedir/$mixedregion -type f -name $mixedzone -print)"
       else
         match="$(find -L $zonedir -name $mixedzone -type f -print)"
       fi

       # If file exactly matched the specified pattern
       if [ -z "$match" ] ; then
         # Check if the pattern was too ambiguous.
         if [ ! -z $(find -L $zonedir -name $mixedzone -type d -print) ] ; then
➋         echo "The region \"$1\" has more than one time zone. " >&2
         else  # Or if it just didn't produce any matches at all
           echo "Can't find an exact match for \"$1\". " >&2
         fi
         echo "Please use 'list' to see all known regions and time zones." >&2
         exit 1
       fi
     fi
➌   timezone="$match"
   fi

   nicetz=$(echo $timezone | sed "s|$zonedir/||g")    # Pretty up the output.

   echo It\'s $(TZ=$timezone date '+%A, %B %e, %Y, at %l:%M %p') in $nicetz

   exit 0
```

*清单 2-19：`*timein*` *shell 脚本，用于报告特定时区的时间*

#### *它是如何工作的*

这个脚本利用了`date`命令的功能，能够显示指定时区的日期和时间，无论当前的环境设置如何。实际上，整个脚本的核心就是识别一个有效的时区名称，以便在最后调用时，`date`命令能够正常工作。

这个脚本的复杂性大部分来自于尝试预测用户输入的世界区域名称，这些名称与时区数据库中的区域名称不匹配。时区数据库是以*时区名称*和*区域/地点名称*列的形式排列的，脚本会尝试为典型的输入问题显示有用的错误消息，比如由于用户指定了像*巴西*这样的国家，而巴西有多个时区，导致找不到时区。

例如，尽管`TZ="Casablanca" date`会因为找不到匹配的区域而显示 UTC/GMT 时间，卡萨布兰卡城市确实存在于时区数据库中。问题在于，你必须使用其正确的区域名称*Africa/Casablanca*，才能使其正常工作，正如本脚本介绍中所展示的那样。

另一方面，这个脚本可以自行在非洲目录中找到卡萨布兰卡，并准确地识别该时区。然而，仅仅指定*非洲*是不够具体的，因为脚本知道非洲内有多个子区域，所以它会生成一条错误信息，表示信息不足以唯一地标识一个特定的时区➋。你还可以使用`list`列出所有时区➊，或者使用一个实际的时区名称➌（例如，UTC 或 WET），作为该脚本的参数。

**注意**

*时区数据库的一个优秀参考可以在线查找，网址为* [`www.twinsun.com/tz/tz-link.htm`](http://www.twinsun.com/tz/tz-link.htm)。

#### *运行脚本*

要查看某个区域或城市的时间，可以将区域或城市名称作为参数传递给`timein`命令。如果你知道区域和城市的名称，也可以将它们指定为`*区域*/*城市*`（例如，`Pacific/Honolulu`）。如果没有任何参数，`timein`会显示 UTC/GMT。清单 2-20 展示了`timein`脚本在多种时区下的运行情况。

#### *结果*

```
$  timein
It's Wednesday, April 5, 2017, at 4:00 PM in UTC
$ timein London
It's Wednesday, April 5, 2017, at 5:00 PM in Europe/London
$ timein Brazil
The region "Brazil" has more than one time zone. Please use 'list'
to see all known regions and time zones.
$ timein Pacific/Honolulu
It's Wednesday, April 5, 2017, at 6:00 AM in Pacific/Honolulu
$ timein WET
It's Wednesday, April 5, 2017, at 5:00 PM in WET
$ timein mycloset
Can't find an exact match for "mycloset". Please use 'list'
to see all known regions and time zones.
```

*清单 2-20：使用不同的时区测试* `*timein*` *脚本*

#### *破解脚本*

知道全球特定时区的时间是一项非常有用的技能，尤其是对于管理全球网络的系统管理员来说。但有时，你其实只是想快速了解两个时区之间的*时间差*。`timein`脚本可以被修改来实现这个功能。通过基于`timein`脚本创建一个新的脚本，可能叫做`tzdiff`，你可以接受两个参数，而不是一个。

使用这两个参数，你可以确定两个时区的当前时间，然后打印出它们之间的小时差异。然而，请记住，两个时区之间的两小时差异可能是向*前*的两小时，或者是向*后*的两小时，这之间的差异非常重要。区分两小时差异是向前还是向后，对于让这个小技巧成为一个有用的脚本至关重要。
