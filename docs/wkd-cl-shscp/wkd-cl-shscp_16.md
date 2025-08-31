## 第十六章：**日期与星期**

![image](img/common4.jpg)

计算日期数学是棘手的，无论是要确定某一年是否为闰年，距离圣诞节还有多少天，还是你已经活了多少天。在这一点上，Unix 系统（如 OS X）和基于 GNU 的 Linux 系统之间存在巨大差距。David MacKenzie 为 GNU 版本的 Linux 重写的 `date` 工具在功能上远远优于其他工具。

如果你使用的是 OS X 或其他系统，其中 `date --version` 会生成错误信息，你可以下载一组核心工具，它们会提供 GNU `date` 作为新的命令行选项（可能会以 `gdate` 安装）。对于 OS X，你可以使用 `brew` 包管理器（默认未安装，但可以轻松安装，以备未来使用）：

```
$ brew install coreutils
```

一旦安装了 GNU `date`，例如计算某一年是否为闰年，可以由程序自动处理，而不需要你去操作那些关于能被 4 整除但不能被 100 整除等复杂规则。

```
if [ $( date 12/31/$year +%j ) -eq 366 ]
```

换句话说，如果一年中的最后一天是第 366 天，那一定是闰年。

另一个使 GNU `date` 优越的特点是它能够回溯很久以前的时间。标准的 Unix `date` 命令是以 1970 年 1 月 1 日 00:00:00 UTC 作为“时间零”或纪元日期构建的。如果你想了解 1965 年发生的事情？那可难了。幸运的是，借助本章的三个巧妙脚本，你可以利用 GNU `date` 的优势。

### #99 查找过去特定日期的星期几

快问：你出生那天是星期几？尼尔·阿姆斯特朗和巴兹·奥尔德林第一次登上月球时是星期几？清单 15-1 中的脚本可以帮助你快速回答这些经典问题，并展示 GNU `date` 的强大功能。

#### *代码*

```
   #!/bin/bash
   # dayinpast--Given a date, reports what day of the week it was

   if [ $# -ne 3 ] ; then
     echo "Usage: $(basename $0) mon day year" >&2
     echo "  with just numerical values (ex: 7 7 1776)" >&2
     exit 1
   fi

   date --version > /dev/null 2>&1    # Discard error, if any.
   baddate="$?"                       # Just look at return code.

   if [ ! $baddate ] ; then
➊   date -d $1/$2/$3 +"That was a %A."
   else

     if [ $2 -lt 10 ] ; then
       pattern=" $2[⁰-9]"
     else
       pattern="$2[⁰-9]"
     fi

     dayofweek="$(➋ncal $1 $3 | grep "$pattern" | cut -c1-2)"

     case $dayofweek in
       Su ) echo "That was a Sunday.";        ;;
       Mo ) echo "That was a Monday.";        ;;
       Tu ) echo "That was a Tuesday.";       ;;
       We ) echo "That was a Wednesday.";     ;;
       Th ) echo "That was a Thursday.";      ;;
       Fr ) echo "That was a Friday.";        ;;
       Sa ) echo "That was a Saturday.";      ;;
     esac
   fi
   exit 0
```

*清单 15-1：* `*dayinpast*` *脚本*

#### *它是如何工作的*

你知道我们一直在推崇 GNU `date` 吧？这就是原因。这个脚本最终只需要在 ➊ 处执行一次。

简单得不可思议。

如果该版本的 `date` 不可用，脚本会使用 `ncal` ➋，这是一个简单的 `cal` 程序的变体，以一种独特但有用的格式呈现指定月份的日历：

```
$ ncal 8 1990
    August 1990
Mo     6 13 20 27
Tu     7 14 21 28
We  1  8 15 22 29
Th  2  9 16 23 30
Fr  3 10 17 24 31
Sa  4 11 18 25
Su  5 12 19 26
```

有了这些信息，确定星期几变得非常简单，只需找到对应日期的行，并将两字母的星期缩写翻译成完整的名称。

#### *运行脚本*

尼尔·阿姆斯特朗和巴兹·奥尔德林于 1969 年 7 月 20 日登陆宁静海基地，清单 15-2 显示那天是星期天。

```
$ dayinpast 7 20 1969
That was a Sunday.
```

*清单 15-2：运行* `*dayinpast*` *脚本，日期为阿姆斯特朗和奥尔德林登月的日期*

诺曼底盟军大规模登陆的 D 日是 1944 年 6 月 6 日：

```
$ dayinpast 6 6 1944
That was a Tuesday.
```

还有一个，美国独立宣言签署的日期是 1776 年 7 月 4 日：

```
$ dayinpast 7 4 1776
That was a Thursday.
```

#### *破解脚本*

本章中的所有脚本都使用相同的 `*month day year*` 输入格式，但如果能让用户指定更熟悉的格式，比如 `*month*/*day*/ *year*`，那会更好。幸运的是，这并不难实现，且脚本 #3 在第 17 页是一个很好的起点。

### #100 计算两个日期之间的天数

你已经活了多少天？自从你父母相遇以来已经过了多少天？有很多类似的问题与经过的时间有关，而答案通常很难计算。然而，GNU `date` 使得这件事变得更简单。

脚本 #100 和脚本 #101 都基于通过计算起始年和结束年之间的天数差异以及每个年份中间的天数来计算两个日期之间的天数的概念。你可以使用这种方法来计算某个过去的日期距离现在有多少天（这个脚本），以及某个未来的日期还有多少天（脚本 #101）。

示例 15-3 相当复杂。准备好了吗？

#### *代码*

```
   #!/bin/bash
   # daysago--Given a date in the form month/day/year, calculates how many
   #   days in the past that was, factoring in leap years, etc.

   # If you are on Linux, this should only be 'which date'.
   #   If you are on OS X, install coreutils with brew or from source for gdate.
   date="$(which gdate)"

   function  daysInMonth
   {
     case $1 in
       1|3|5|7|8|10|12 ) dim=31 ;;  # Most common value
       4|6|9|11        ) dim=30 ;;
       2               ) dim=29 ;;  # Depending on whether it's a leap year
       *               ) dim=-1 ;;  # Unknown month
     esac
   }

➊ function isleap
   {
     # Returns nonzero value for $leapyear if $1 was a leap year
       leapyear=$($date -d 12/31/$1 +%j | grep 366)
   }

   #######################
   #### MAIN BLOCK
   #######################

   if [ $# -ne 3 ] ; then
     echo "Usage: $(basename $0) mon day year"
     echo "  with just numerical values (ex: 7 7 1776)"
     exit 1
   fi

➋ $date --version > /dev/null 2>&1         # Discard error, if any.

   if [ $? -ne 0 ] ; then
     echo "Sorry, but $(basename $0) can't run without GNU date." >&2
     exit 1
   fi

   eval $($date "+thismon=%m;thisday=%d;thisyear=%Y;dayofyear=%j")

   startmon=$1; startday=$2; startyear=$3

   daysInMonth $startmon # Sets global var dim.

   if [ $startday -lt 0 -o $startday -gt $dim ] ; then
     echo "Invalid: Month #$startmon only has $dim days." >&2
     exit 1
   fi

   if [ $startmon -eq 2 -a $startday -eq 29 ] ; then
     isleap $startyear
     if [ -z "$leapyear" ] ; then
       echo "Invalid: $startyear wasn't a leap year; February had 28 days." >&2
       exit 1
     fi
   fi

   #######################
   #### CALCULATING DAYS
   #######################

   #### DAYS LEFT IN START YEAR

   # Calculate the date string format for the specified starting date.

   startdatefmt="$startmon/$startday/$startyear"

➌ calculate="$((10#$($date -d "12/31/$startyear" +%j))) \
     -$((10#$($date -d $startdatefmt +%j)))"

   daysleftinyear=$(( $calculate ))

   #### DAYS IN INTERVENING YEARS

   daysbetweenyears=0
   tempyear=$(( $startyear + 1 ))

   while [ $tempyear -lt $thisyear ] ; do
     daysbetweenyears=$(($daysbetweenyears + \
     $((10#$($date -d "12/31/$tempyear" +%j)))))
     tempyear=$(( $tempyear + 1 ))
   done

   #### DAYS IN CURRENT YEAR

➍ dayofyear=$($date +%j) # That's easy!

   #### NOW ADD IT ALL UP

   totaldays=$(( $((10#$daysleftinyear)) + \
     $((10#$daysbetweenyears)) + \
     $((10#$dayofyear)) ))

   /bin/echo -n "$totaldays days have elapsed between "
   /bin/echo -n "$startmon/$startday/$startyear "
   echo "and today, day $dayofyear of $thisyear."
   exit 0
```

*示例 15-3：* `*daysago*` *脚本*

#### *工作原理*

这是一个长脚本，但其原理并不复杂。闰年函数 ➊ 很简单——我们只需检查该年份是否有 366 天。

有一个有趣的测试，确保在脚本继续之前，GNU 版本的 `date` 是可用的 ➋。

重定向会丢弃任何错误信息或输出，返回码会被检查以确定是否为非零值，如果是非零值，则表示解析`--version`参数时出错。例如，在 OS X 上，`date` 命令是最简化的，并没有`--version`或许多其他功能。

现在只是基础的日期计算。`%j` 返回年份中的第几天，因此它使得计算当前年份剩余的天数变得非常简单 ➌。介于两年之间的天数在 `while` 循环中计算，其中进度通过 `tempyear` 变量来跟踪。

最后，当前年份已经过去了多少天？这在 ➍ 很容易算出来。

```
dayofyear=$($date +%j)
```

然后只需将天数相加就能得到结果！

#### *运行脚本*

让我们再看一下示例 15-4 中的那些历史日期。

```
$ daysago 7 20 1969
17106 days have elapsed between 7/20/1969 and today, day 141 of 2016.

$ daysago 6 6 1944
26281 days have elapsed between 6/6/1944 and today, day 141 of 2016.

$ daysago 1 1 2010
2331 days have elapsed between 1/1/2010 and today, day 141 of 2016.
```

*示例 15-4：使用不同日期运行* `*daysago*` *脚本*

这些都是运行在... 好吧，让我们让 `date` 来告诉我们：

```
$ date
Fri May 20 13:30:49 UTC 2016
```

#### *破解脚本*

脚本没有捕捉到一些额外的错误情况，特别是在过去的日期距离现在只有几天，甚至是未来几天的边界情况。会发生什么？你怎么修复它？（提示：看看脚本 #101，了解你可以对这个脚本应用的更多测试。）

### #101 计算直到指定日期的天数

脚本 #100 的逻辑伙伴`daysago`是另一个脚本`daysuntil`。这个脚本本质上执行相同的计算，但修改了逻辑，以计算当前年份剩余的天数、跨年年份的天数以及目标年份指定日期之前的天数，正如列表 15-5 所示。

#### *代码*

```
   #!/bin/bash
   # daysuntil--Basically, this is the daysago script backward, where the
   #   desired date is set as the current date and the current date is used
   #   as the basis of the daysago calculation.
 # As in the previous script, use 'which gdate' if you are on OS X.
   #   If you are on Linux, use 'which date'.
   date="$(which gdate)"

   function daysInMonth
   {
     case $1 in
       1|3|5|7|8|10|12 ) dim=31 ;;  # Most common value
       4|6|9|11        ) dim=30 ;;
       2               ) dim=29 ;;  # Depending on whether it's a leap year
       *               ) dim=-1 ;;  # Unknown month
     esac
   }

   function isleap
   {
     # If specified year is a leap year, returns nonzero value for $leapyear

     leapyear=$($date -d 12/31/$1 +%j | grep 366)
   }

   #######################
   #### MAIN BLOCK
   #######################

   if [ $# -ne 3 ] ; then
     echo "Usage: $(basename $0) mon day year"
     echo "  with just numerical values (ex: 1 1 2020)"
     exit 1
   fi

   $date --version > /dev/null 2>&1         # Discard error, if any.

   if [ $? -ne 0 ] ; then
     echo "Sorry, but $(basename $0) can't run without GNU date." >&2
     exit 1
   fi

   eval $($date "+thismon=%m;thisday=%d;thisyear=%Y;dayofyear=%j")

   endmon=$1; endday=$2; endyear=$3

   # Lots of parameter checks needed...

   daysInMonth $endmon    # Sets $dim variable
   if [ $endday -lt 0 -o $endday -gt $dim ] ; then
     echo "Invalid: Month #$endmon only has $dim days." >&2
     exit 1
   fi

   if [ $endmon -eq 2 -a $endday -eq 29 ] ; then
     isleap $endyear
 if [ -z "$leapyear" ] ; then
       echo "Invalid: $endyear wasn't a leapyear; February had 28 days." >&2
       exit 1
     fi
   fi

   if [ $endyear -lt $thisyear ] ; then
     echo "Invalid: $endmon/$endday/$endyear is prior to the current year." >&2
     exit 1
   fi

   if [ $endyear -eq $thisyear -a $endmon -lt $thismon ] ; then
     echo "Invalid: $endmon/$endday/$endyear is prior to the current month." >&2
     exit 1
   fi

   if [ $endyear -eq $thisyear -a $endmon -eq $thismon -a $endday -lt $thisday ]
   then
     echo "Invalid: $endmon/$endday/$endyear is prior to the current date." >&2
     exit 1
   fi

➊ if [ $endyear -eq $thisyear -a $endmon -eq $thismon -a $endday -eq $thisday ]
   then
     echo "There are zero days between $endmon/$endday/$endyear and today." >&2
     exit 0
   fi

   #### If we're working with the same year, the calculation is a bit different.

   if [ $endyear -eq $thisyear ] ; then

     totaldays=$(( $($date -d "$endmon/$endday/$endyear" +%j) - $($date +%j) ))

   else

     #### Calculate this in chunks, starting with days left in this year.

     #### DAYS LEFT IN START YEAR

     # Calculate the date string format for the specified starting date.

     thisdatefmt="$thismon/$thisday/$thisyear"

     calculate="$($date -d "12/31/$thisyear" +%j) - $($date -d $thisdatefmt +%j)"

     daysleftinyear=$(( $calculate ))

     #### DAYS IN INTERVENING YEARS

     daysbetweenyears=0
     tempyear=$(( $thisyear + 1 ))
     while [ $tempyear -lt $endyear ] ; do
       daysbetweenyears=$(( $daysbetweenyears + \
         $($date -d "12/31/$tempyear" +%j) ))
       tempyear=$(( $tempyear + 1 ))
     done

     #### DAYS IN END YEAR

     dayofyear=$($date --date $endmon/$endday/$endyear +%j)    # That's easy!

     #### NOW ADD IT ALL UP

     totaldays=$(( $daysleftinyear + $daysbetweenyears + $dayofyear ))
   fi

   echo "There are $totaldays days until the date $endmon/$endday/$endyear."
   exit 0
```

*列表 15-5：*`*daysuntil*`* 脚本*

#### *它是如何工作的*

如我们所说，`daysago` 脚本和这个脚本之间有很多重叠，足以让你将它们合并为一个脚本，并通过条件判断来测试用户请求的是过去的日期还是未来的日期。这里的大部分数学运算实际上是`daysago`脚本中的数学运算的逆操作，是向未来看而不是向过去看。

然而，这个脚本稍微干净一些，因为它在执行实际计算之前考虑了更多的错误条件。例如，我们最喜欢的测试，见 ➊。

如果有人试图通过指定今天的日期来欺骗脚本，这个条件判断会捕捉到这一点并返回“零天”作为计算结果。

#### *运行脚本*

离 2020 年 1 月 1 日还有多少天？列表 15-6 给出了答案。

```
$ daysuntil 1 1 2020
There are 1321 days until the date 1/1/2020.
```

*列表 15-6：运行*`*daysuntil*`* 脚本，使用 2020 年的第一天*

离 2025 年圣诞节还有多少天？

```
$ daysuntil 12 25 2025
There are 3506 days until the date 12/25/2025.
```

准备迎接美国的三百周年纪念了吗？这里是你剩余的天数：

```
$ daysuntil 7 4 2076
There are 21960 days until the date 7/4/2076.
```

最后，考虑到以下情况，我们很可能不会活到第三个千年：

```
$ daysuntil 1 1 3000
There are 359259 days until the date 1/1/3000.
```

#### *黑客脚本*

在脚本 #99 中，我们能够确定给定日期是星期几。将这个功能与`daysago`和`daysuntil`脚本的功能结合在一起，一次性获取所有相关信息将非常有用。
