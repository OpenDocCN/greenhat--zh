## 第十八章：B

**附加脚本**

![image](img/common4.jpg)

因为我们无法拒绝这些珍品！在我们开发第二版时，最终我们写了几个备份脚本。结果我们并不需要这些备用脚本，但我们不想把我们的秘密武器藏着不让读者知道。

前两个附加脚本是为系统管理员设计的，他们需要管理大量文件的迁移或处理。最后一个脚本是为那些总是在寻找下一个即将被转化为 shell 脚本的 web 服务的 web 用户准备的；我们将抓取一个帮助我们跟踪月亮各个阶段的网站！

### #102 批量重命名文件

系统管理员经常需要将许多文件从一个系统移动到另一个系统，而且在新系统中，文件通常需要完全不同的命名方案。对于一些文件，手动重命名很简单，但当需要重命名数百或数千个文件时，这立即成为一个更适合用 shell 脚本完成的任务。

#### *代码*

Listing B-1 中的简单脚本接受两个参数用于匹配和替换的文本，以及一个指定要重命名的文件的参数列表（这些文件可以通过通配符方便地使用）。

```
   #!/bin/bash
   # bulkrename--Renames specified files by replacing text in the filename

➊ printHelp()
   {
     echo "Usage: $0 -f find -r replace FILES_TO_RENAME*"
     echo -e "\t-f The text to find in the filename"
     echo -e "\t-r The replacement text for the new filename"
     exit 1
   }

➋ while getopts "f:r:" opt
   do
     case "$opt" in
       r ) replace="$OPTARG"    ;;
       f ) match="$OPTARG"      ;;
       ? ) printHelp            ;;
     esac
   done

   shift $(( $OPTIND - 1 ))

   if [ -z $replace➌ ] || [ -z $match➍ ]
   then
     echo "You need to supply a string to find and a string to replace";
     printHelp
   fi

➎ for i in $@
   do
     newname=$(echo $i | ➏sed "s/$match/$replace/")
     mv $i $newname
     && echo "Renamed file $i to $newname"
   done
```

*Listing B-1: `*bulkrename*` 脚本*

#### *工作原理*

我们首先定义一个`printHelp()`函数 ➊，它将打印所需的参数和脚本的目的，然后退出。在定义了新函数后，代码使用`getopts` ➋（如同之前的脚本中一样）迭代脚本传入的参数，当指定了参数时，将值赋给`replace`和`match`变量。

脚本接着检查我们是否为稍后使用的变量提供了值。如果`replace` ➌和`match` ➍变量的长度为零，脚本会打印错误消息，告诉用户他们需要提供一个查找字符串和一个替换字符串。然后脚本打印`printHelp`文本并退出。

在验证`match`和`replace`有值之后，脚本开始迭代其余指定的参数 ➎，这些参数应该是需要重命名的文件。我们使用`sed` ➏将文件名中的`match`字符串替换为`replace`字符串，并将新文件名存储在一个 bash 变量中。存储了新文件名后，我们使用`mv`命令将文件移动到新文件名，并打印一条消息告诉用户文件已经被重命名。

#### *运行脚本*

`bulkrename` shell 脚本接受两个字符串参数和要重命名的文件（这些文件可以通过通配符方便地使用；否则，必须逐个列出）。如果指定了无效的参数，将打印一条友好的帮助消息，如 Listing B-2 所示。

#### *结果*

```
   $ ls ~/tmp/bulk
   1_dave  2_dave  3_dave  4_dave
   $ bulkrename
   You need to supply a string to find and a string to replace
   Usage: bulkrename -f find -r replace FILES_TO_RENAME*
     -f The text to find in the filename
     -r The replacement text for the new filename
➊ $ bulkrename -f dave -r brandon ~/tmp/bulk/*
   Renamed file /Users/bperry/tmp/bulk/1_dave to /Users/bperry/tmp/bulk/1_brandon
   Renamed file /Users/bperry/tmp/bulk/2_dave to /Users/bperry/tmp/bulk/2_brandon
   Renamed file /Users/bperry/tmp/bulk/3_dave to /Users/bperry/tmp/bulk/3_brandon
   Renamed file /Users/bperry/tmp/bulk/4_dave to /Users/bperry/tmp/bulk/4_brandon
   $ ls ~/tmp/bulk
   1_brandon  2_brandon  3_brandon  4_brandon
```

*Listing B-2: 运行`*bulkrename*` 脚本*

你可以单独列出要重命名的文件，或者使用文件路径中的星号（`*`）进行通配符匹配，就像我们在 ➊ 中所做的那样。每个重命名的文件在被移动后都会显示其新名称，以确保用户文件已按预期重命名。

#### *破解脚本*

有时，将文件名中的文本替换为特殊字符串（如今天的日期或时间戳）可能会很有用。这样，你就能知道文件是什么时候重命名的，而不需要在`-r`参数中指定今天的日期。你可以通过在脚本中添加特殊标记来实现这一点，这些标记在文件重命名时会被替换。例如，你可以有一个`replace`字符串，其中包含`%d`或`%t`，它们在文件重命名时分别被今天的日期或时间戳替换。

这样的特殊标记可以使文件移动以备份变得更容易。你可以添加一个`cron`作业来移动某些文件，这样脚本就会自动更新文件名中的动态标记，而不必在想要更改文件名中的日期时更新`cron`作业。

### #103 在多处理器机器上批量运行命令

本书首次出版时，除非你从事服务器或大型主机相关工作，否则拥有多核或多处理器的机器是非常罕见的。如今，大多数笔记本电脑和台式机都有多个核心，使得计算机可以同时处理更多的任务。但有时你想要运行的程序无法充分利用这种处理能力的增加，可能一次只能使用一个核心；要利用更多的核心，你需要同时运行多个程序实例。

假设你有一个将图像文件从一种格式转换为另一种格式的程序，并且有大量的文件需要转换！让一个进程依次串行转换每个文件（一个接一个，而不是并行转换）可能需要很长时间。将文件分配到多个进程中并行处理会更快。

清单 B-3 中的脚本详细介绍了如何将给定的命令并行化，以便一次运行多个进程。

**注意**

*如果你*没有*多核的计算机，或者程序因为其他原因（如硬盘访问瓶颈）而变慢，运行多个并行实例可能会对性能产生不利影响。启动过多进程可能会使系统负担过重，因此要小心。幸运的是，即便是树莓派现在也有多个核心了！*

#### *代码*

```
   #!/bin/bash
   # bulkrun--Iterates over a directory of files, running a number of
   #   concurrent processes that will process the files in parallel

   printHelp()
   {
     echo "Usage: $0 -p 3 -i inputDirectory/ -x \"command -to run/\""
➊   echo -e "\t-p The maximum number of processes to start concurrently"
➋   echo -e "\t-i The directory containing the files to run the command on"
➌   echo -e "\t-x The command to run on the chosen files"
     exit 1
   }

➍ while getopts "p:x:i:" opt
   do
     case "$opt" in
       p ) procs="$OPTARG"    ;;
       x ) command="$OPTARG"  ;;
       i ) inputdir="$OPTARG" ;;
       ? ) printHelp          ;;
     esac
   done

   if [[ -z $procs || -z $command || -z $inputdir ]]
   then
➎   echo "Invalid arguments"
     printHelp
   fi

   total=➏$(ls $inputdir | wc -l)
   files="$(ls -Sr $inputdir)"

➐ for k in $(seq 1 $procs $total)
   do
➑   for i in $(seq 0 $procs)
     do
       if [[ $((i+k)) -gt $total ]]
       then
         wait
         exit 0
       fi

       file=➒$(echo "$files" | sed $(expr $i + $k)"q;d")
       echo "Running $command $inputdir/$file"
       $command "$inputdir/$file"&
     done

➓ wait
   done
```

*清单 B-3: The* `*bulkrun*` *脚本*

#### *工作原理*

`bulkrun` 脚本接受三个参数：同时运行的最大进程数 ➊，包含待处理文件的目录 ➋，以及要执行的命令（后缀为要处理的文件名） ➌。通过 `getopts` 解析用户提供的参数 ➍ 后，脚本检查用户是否提供了这三个参数。如果在处理用户参数后，`procs`、`command` 或 `inputdir` 变量未定义，脚本将打印错误信息 ➎ 和帮助文本，然后退出。

一旦我们确定了运行并行进程所需的变量，脚本的真正工作就可以开始了。首先，脚本确定要处理的文件数量 ➏ 并保存文件列表，以备后用。然后，脚本开始一个 `for` 循环，用来跟踪到目前为止处理了多少文件。这个 `for` 循环使用 `seq` 命令 ➐ 从 1 到指定的文件总数迭代，并使用将并行运行的进程数作为增量步长。

在其中还有一个 `for` 循环 ➑ 用于跟踪在给定时间启动的进程数量。这个内部的 `for` 循环也使用 `seq` 命令从 0 迭代到指定的进程数，默认增量步长为 1。在每次内部 `for` 循环的迭代中，脚本从文件列表 ➒ 中提取一个新文件，使用 `sed` 打印出我们需要的文件，并在后台使用 `&` 符号运行提供的命令。

当最大进程数在后台启动后，`wait` 命令 ➓ 会告诉脚本休眠，直到后台所有命令完成处理。`wait` 完成后，整个工作流将重新开始，继续处理更多文件。这类似于我们在脚本 `bestcompress` 中快速实现最佳压缩的方法（脚本 #34 在 第 113 页）。

#### *运行脚本*

使用 `bulkrun` 脚本非常简单。它接受的三个参数分别是同时运行的最大进程数、要处理的文件目录和要在文件上执行的命令。例如，如果你想并行地运行 ImageMagick 工具 `mogrify` 来调整图像目录中的图片大小，你可以运行类似于 列表 B-4 的命令。

#### *结果*

```
$ bulkrun -p 3 -i tmp/ -x "mogrify -resize 50%"
Running mogrify -resize 50% tmp//1024-2006_1011_093752.jpg
Running mogrify -resize 50% tmp//069750a6-660e-11e6-80d1-001c42daa3a7.jpg
Running mogrify -resize 50% tmp//06970ce0-660e-11e6-8a4a-001c42daa3a7.jpg
Running mogrify -resize 50% tmp//0696cf00-660e-11e6-8d38-001c42daa3a7.jpg
Running mogrify -resize 50% tmp//0696cf00-660e-11e6-8d38-001c42daa3a7.jpg
--snip--
```

*列表 B-4：运行* `*bulkrun*` *命令并行化* `*mogrify*` *ImageMagick 命令*

#### *修改脚本*

能够在命令中指定文件名，或者使用类似于在 `bulkrename` 脚本中提到的令牌（脚本 #102 在 第 346 页）是非常有用的：这些特殊字符串在运行时被动态值替换（例如 `%d`，它被当前日期替换，或者 `%t`，它被时间戳替换）。更新脚本，使其能够在命令或文件名中替换类似日期或时间戳的特殊令牌，在处理文件时会非常有帮助。

另一个有用的技巧可能是使用 `time` 工具跟踪所有处理所需的时间。如果脚本能打印统计信息，显示将处理多少文件，已处理多少文件，还剩多少文件，那么在处理一项真正庞大的工作时，了解这些信息是非常有价值的。

### #104 查找月相

无论你是狼人、女巫，还是单纯对农历感兴趣，跟踪月相并了解盈亏和凸月（月亮几乎与长臂猿无关）是非常有用且具有教育意义的。

让事情变得复杂的是，月亮的轨道周期为 27.32 天，并且它的月相实际上取决于你在地球上的位置。不过，给定一个特定日期，还是可以计算出月亮的相位。

但既然有很多在线站点已经可以计算过去、现在或未来的任意日期的月相，为什么还要做这么多工作呢？在列表 B-5 中的脚本，我们将利用 Google 使用的同一站点，如果你搜索当前月相，网站是：* [`www.moongiant.com/`](http://www.moongiant.com/) *。

#### *代码*

```
   #!/bin/bash

   # moonphase--Reports the phase of the moon (really the percentage of
   #   illumination) for today or a specified date

   # Format of Moongiant.com query:
   #   http://www.moongiant.com/phase/MM/DD/YYYY

   # If no date is specified, use "today" as a special value.

   if [ $# -eq 0 ] ; then
     thedate="today"
   else
     # Date specified. Let's check whether it's in the right format.
      mon="$(echo $1 | cut -d/ -f1)"
      day="$(echo $1 | cut -d/ -f2)"
     year="$(echo $1 | cut -d/ -f3)"

➊   if [ -z "$year" -o -z "$day" ] ; then     # Zero length?
       echo "Error: valid date format is MM/DD/YYYY"
       exit 1
     fi
     thedate="$1" # No error checking = dangerous
   fi

   url="http://www.moongiant.com/phase/$thedate"
➋ pattern="Illumination:"

➌ phase="$( curl -s "$url" | grep "$pattern" | tr ',' '\
   ' | grep "$pattern" | sed 's/[⁰-9]//g')"

   # Site output format is "Illumination: <span>NN%\n<\/span>"

   if [ "$thedate" = "today" ] ; then
     echo "Today the moon is ${phase}% illuminated."
   else
     echo "On $thedate the moon = ${phase}% illuminated."
   fi

   exit 0
```

*列表 B-5：* `*moonphase*` *脚本*

#### *它是如何工作的*

与其他从网络查询中提取值的脚本类似，`moonphase` 脚本的核心是识别不同查询 URL 的格式，并从返回的 HTML 数据流中提取特定值。

对该站点的分析显示，存在两种类型的 URL：一种指定当前日期，简单结构为“phase/today”，另一种指定过去或未来的日期，格式为 MM/DD/YYYY，如“phase/08/03/2017”。

指定正确格式的日期，你就可以获得该日期的月相。但我们不能仅仅将日期附加到站点的域名上而不做错误检查，因此脚本将用户输入拆分成三部分——月份、日期和年份——然后在 ➊ 处确保日期和年份的值不为零。还可以进行更多的错误检查，我们将在“黑客脚本”一节中探讨。

任何抓取脚本中最棘手的部分就是正确识别出能够提取所需数据的模式。在`moonphase`脚本中，这一点在➋处指定。最长且最复杂的代码行出现在➌处，脚本从*moongiant.com*网站获取页面，然后使用一系列`grep`和`sed`命令提取与指定模式匹配的那一行。

然后，只需通过最终的`if`/`then`/`else`语句显示照明级别，无论是今天还是指定的日期。

#### *运行脚本*

如果没有参数，`moonphase`脚本会显示当前日期的月亮照明百分比。通过输入 MM/DD/YYYY 格式的日期来指定过去或未来的任何日期，如清单 B-6 所示。

#### *结果*

```
$ moonphase 08/03/2121
On 08/03/2121 the moon = 74% illuminated.

$ moonphase
Today the moon is 100% illuminated.

$ moonphase 12/12/1941
On 12/12/1941 the moon = 43% illuminated.
```

*清单 B-6：运行* `*moonphase*` *脚本*

**注意**

*1941 年 12 月 12 日是经典的环球恐怖电影《狼人与人》首次在电影院上映的日子。那时并不是满月。真是不可思议！*

#### *黑客破解脚本*

从内部角度来看，脚本可以通过更好的错误检查序列来大大改进，甚至仅通过在第 17 页使用脚本 #3。这将允许用户以更多格式指定日期。一个改进是用一个函数替换末尾的`if`/`then`/`else`语句，该函数将照明级别转换为更常见的月相词汇，如“盈月”、“亏月”和“凸月”。NASA 有一个你可以使用的网页，定义了不同的月相：[`starchild.gsfc.nasa.gov/docs/StarChild/solar_system_level2/moonlight.html`](http://starchild.gsfc.nasa.gov/docs/StarChild/solar_system_level2/moonlight.html)。
