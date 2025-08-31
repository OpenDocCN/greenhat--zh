## 第十三章：**SHELL 脚本趣味与游戏**

![image](img/common4.jpg)

到目前为止，我们关注的是 Shell 脚本的严肃应用，旨在改善你与系统的互动，并使系统更具灵活性和强大功能。但是，Shell 脚本还有另一个值得探索的方面：游戏。

别担心——我们并不是提议你把*Fallout 4*写成 Shell 脚本。只是有一些简单的游戏，非常适合用 Shell 脚本来编写，并且具有很高的可读性。你不希望通过一些有趣的脚本来学习如何调试 Shell 脚本，而不是通过一些暂停用户帐户或分析 Apache 错误日志的工具吗？

对于其中的一些脚本，你需要从书籍资源中获取文件，文件可以在*[`www.nostarch.com/wcss2/`](http://www.nostarch.com/wcss2/)*找到，如果你还没有下载文件，现在就去下载。

**两个快速技巧**

这里有两个快速示例，向你展示我们的意思。首先，老派的 Usenet 用户知道*rot13*，这是一种简单的机制，通过这种机制，黄色笑话和猥亵文字会被模糊化，使它们稍微不那么容易被读取。这是一种*替代加密*，在 Unix 中非常容易实现。

要对某个内容进行 rot13 编码，可以通过`tr`命令来处理。

```
tr '[a-zA-Z]' '[n-za-mN-ZA-M]'
```

这是一个例子：

```
$ echo "So two people walk into a bar..." | tr '[a-zA-Z]' '[n-za-mN-ZA-M]'
Fb gjb crbcyr jnyx vagb n one...
```

要解开它，应用相同的转换：

```
$ echo 'Fb gjb crbcyr jnyx vagb n one...' | tr '[a-zA-Z]' '[n-za-mN-ZA-M]'
So two people walk into a bar...
```

这种著名的替代加密方法与电影*2001 太空漫游*有关。还记得计算机的名字吗？来看一下：

```
$ echo HAL | tr '[a-zA-Z]' '[b-zaB-ZA]'
IBM
```

另一个简短的示例是回文检测器。输入你认为是回文的内容，代码将对其进行测试。

```
testit="$(echo $@ | sed 's/[^[:alpha:]]//g' | tr '[:upper:]' '[:lower:]')"
backward="$(echo $testit | rev)"

if [ "$testit" = "$backward" ] ; then
  echo "$@ is a palindrome"
else
  echo "$@ is not a palindrome"
fi
```

回文是一个前后相同的单词，因此第一步是去除所有非字母字符，并确保所有字母都是小写字母。然后，Unix 工具`rev`会反转输入行中的字母。如果正向和反向版本相同，那么就是回文；如果不同，则不是回文。

本章中的游戏只是稍微复杂一点，但都非常有趣，值得添加到你的系统中。

### #83 解谜：文字游戏

这是一个基本的字谜游戏。如果你曾在报纸上玩过*Jumble*游戏，或者玩过任何文字游戏，你应该熟悉这个概念：随机挑选一个单词并将其打乱。你的任务是在最少的回合内找出原始单词。这个游戏的完整脚本在清单 12-1 中，但要获取单词列表，你还需要从书籍资源中下载*long-words.txt*文件，网址是*[`www.nostarch.com/wcss2/`](http://www.nostarch.com/wcss2/)*，并将其保存在目录*/usr/lib/games*中。

#### *代码*

```
   #!/bin/bash
   # unscramble--Picks a word, scrambles it, and asks the user to guess
   #   what the original word (or phrase) was

   wordlib="/usr/lib/games/long-words.txt"

   scrambleword()
   {
     # Pick a word randomly from the wordlib and scramble it.
     #   Original word is $match, and scrambled word is $scrambled.

     match="$(➊randomquote $wordlib)"

     echo "Picked out a word!"

     len=${#match}
     scrambled=""; lastval=1

     for (( val=1; $val < $len ; ))
     do
➋     if [ $(($RANDOM % 2)) -eq 1 ] ; then
         scrambled=$scrambled$(echo $match | cut -c$val)
       else
         scrambled=$(echo $match | cut -c$val)$scrambled
       fi
       val=$(( $val + 1 ))
     done
   }

   if [ ! -r $wordlib ] ; then
     echo "$0: Missing word library $wordlib" >&2
     echo "(online: http://www.intuitive.com/wicked/examples/long-words.txt" >&2
     echo "save the file as $wordlib and you're ready to play!)" >&2
     exit 1
   fi

   newgame=""; guesses=0; correct=0; total=0

➌ until [ "$guess" = "quit" ] ; do

     scrambleword
 echo ""
     echo "You need to unscramble: $scrambled"

     guess="??" ; guesses=0
     total=$(( $total + 1 ))

➍ while [ "$guess" != "$match" -a "$guess" != "quit" -a "$guess" != "next" ]
     do
       echo ""
       /bin/echo -n "Your guess (quit|next) : "
       read guess

       if [ "$guess" = "$match" ] ; then
         guesses=$(( $guesses + 1 ))
         echo ""
         echo "*** You got it with tries = ${guesses}! Well done!! ***"
         echo ""
         correct=$(( $correct + 1 ))
       elif [ "$guess" = "next" -o "$guess" = "quit" ] ; then
         echo "The unscrambled word was \"$match\". Your tries: $guesses"
       else
         echo "Nope. That's not the unscrambled word. Try again."
         guesses=$(( $guesses + 1 ))
       fi
     done
   done

   echo "Done. You correctly figured out $correct out of $total scrambled words."

   exit 0
```

*清单 12-1：* `*unscramble*` *Shell 脚本游戏*

#### *工作原理*

要从文件中随机选取一行，脚本使用了 `randomquote` （参见 Script #68 ，第 213 页） ➊，即使该脚本最初是为处理网页而编写的（就像许多优秀的 Unix 工具一样，事实上，它在其他场景中也非常有用）。

这个脚本最难的部分是弄清楚如何打乱单词。虽然没有直接可用的 Unix 工具，但事实证明，如果我们按字母逐个检查正确拼写的单词，并随机将每个后续字母加到打乱序列的开头或结尾，那么我们每次都能以不同且不可预测的方式打乱单词 ➋。

注意 `$scrambled` 在两行中的位置：在第一行中，添加的字母被追加，而在第二行中，它被放置在开头。

否则，游戏的主要逻辑应该很容易理解：外层的 `until` 循环 ➌ 会一直运行，直到用户输入 `quit` 作为猜测，而内层的 `while` 循环 ➍ 会一直运行，直到用户猜出单词或输入 `next` 跳到下一个单词。

#### *运行脚本*

这个脚本没有参数或选项，所以只需输入名称即可开始游戏！

#### *结果*

运行后，脚本会将各种长度的打乱单词呈现给用户，并跟踪用户成功解开的单词数量，如 Listing 12-2 所示。

```
$ unscramble
Picked out a word!

You need to unscramble: ninrenoccg

Your guess (quit|next) : concerning

*** You got it with tries = 1! Well done!! ***

Picked out a word!

You need to unscramble: esivrmipod

Your guess (quit|next) : quit
The unscrambled word was "improvised". Your tries: 0
Done. You correctly figured out 1 out of 2 scrambled words.
```

*Listing 12-2: 运行* `*unscramble*` *shell 脚本游戏*

显然第一个猜测非常有灵感！

#### *破解脚本*

提供某种线索的方式会让这个游戏更加有趣，如果能有一个提示最小可接受单词长度的标志，那就更好了。为了实现前者，或许可以将未打乱单词的前 *n* 个字母展示出来，作为一种在得分中扣除的惩罚；每次请求提示时，都会展示一个额外的字母。对于后者，你需要一个扩展的单词字典，因为脚本中包含的字典的最小单词长度为 10 个字母——这有点难！

### #84 刽子手：在为时已晚之前猜出单词

**“刽子手”**是一个带有恐怖隐喻的文字游戏，尽管如此，它依然是一个令人愉快的经典游戏。在这个游戏中，你需要猜测隐藏单词中的字母，每次猜错时，吊在绞刑架上的人就会多出一部分身体。如果你猜错太多次，那个“人”就会被完全画出，这样不仅你会失败，嗯，你大概也会死。后果相当严苛！

然而，游戏本身很有趣，且将其编写为 shell 脚本证明意外地简单，如 Listing 12-3 所示。对于这个脚本，你仍然需要我们在 Script #83 中使用的单词列表，该文件位于 第 275 页：将书中的 *long-words.txt* 文件保存在目录 */usr/lib/games* 中。

#### *代码*

```
   #!/bin/bash
   # hangman--A simple version of the hangman game. Instead of showing a
   #   gradually embodied hanging man, this simply has a bad-guess countdown.
   #   You can optionally indicate the initial distance from the gallows as
   #   the only argument.

   wordlib="/usr/lib/games/long-words.txt"
   empty="\."      # We need something for the sed [set] when $guessed="".
   games=0

   # Start by testing for our word library datafile.

   if [ ! -r "$wordlib" ] ; then
     echo "$0: Missing word library $wordlib" >&2
     echo "(online: http://www.intuitive.com/wicked/examples/long-words.txt" >&2
     echo "save the file as $wordlib and you're ready to play!)" >&2
     exit 1
   fi

   # The big while loop. This is where everything happens.

   while [ "$guess" != "quit" ] ; do
     match="$(randomquote $wordlib)"      # Pick a new word from the library.

     if [ $games -gt 0 ] ; then
       echo ""
       echo "*** New Game! ***"
     fi

     games="$(( $games + 1 ))"
     guessed="" ; guess="" ; bad=${1:-6}
     partial="$(echo $match | sed "s/[^$empty${guessed}]/-/g")"

     # The guess > analyze > show results > loop happens in this block.

     while [ "$guess" != "$match" -a "$guess" != "quit" ] ; do

       echo ""
       if [ ! -z "$guessed" ] ; then # Remember, ! –z means "is not empty".
         /bin/echo -n "guessed: $guessed, "
       fi
       echo "steps from gallows: $bad, word so far: $partial"

       /bin/echo -n "Guess a letter: "
       read guess
       echo ""
       if [ "$guess" = "$match" ] ; then   # Got it!
         echo "You got it!"
       elif [ "$guess" = "quit" ] ; then   # You're out? Okay.
         exit 0
       # Now we need to validate the guess with various filters.
➊     elif [ $(echo $guess | wc -c | sed 's/[^[:digit:]]//g') -ne 2 ] ; then
         echo "Uh oh: You can only guess a single letter at a time"
➋     elif [ ! -z "$(echo $guess | sed 's/[[:lower:]]//g')" ] ; then
         echo "Uh oh: Please only use lowercase letters for your guesses"
➌     elif [ -z "$(echo $guess | sed "s/[$empty$guessed]//g")" ] ; then
         echo "Uh oh: You have already tried $guess"
       # Now we can actually see if the letter appears in the word.
➍     elif [ "$(echo $match | sed "s/$guess/-/g")" != "$match" ] ; then
         guessed="$guessed$guess"
➎     partial="$(echo $match | sed "s/[^$empty${guessed}]/-/g")"
         if [ "$partial" = "$match" ] ; then
           echo "** You've been pardoned!! Well done! The word was \"$match\"."
           guess="$match"
         else
           echo "* Great! The letter \"$guess\" appears in the word!"
         fi
       elif [ $bad -eq 1 ] ; then
         echo "** Uh oh: you've run out of steps. You're on the platform..."
         echo "** The word you were trying to guess was \"$match\""
         guess="$match"
       else
         echo "* Nope, \"$guess\" does not appear in the word."
         guessed="$guessed$guess"
         bad=$(( $bad - 1 ))
       fi
     done
   done
   exit 0
```

*Listing 12-3: The* `*hangman*` *shell 脚本游戏*

#### *工作原理*

这个脚本中的测试都很有趣，值得仔细检查。考虑一下在 ➊ 处的测试，它检查玩家是否输入了多个字母作为猜测。

为什么测试值是 2 而不是 1？因为输入的值包含了用户按下 ENTER 键时产生的回车符（即字符 `\n`），如果正确输入，它将有两个字母，而不是一个。这个语句中的 `sed` 会去除所有非数字字符，当然是为了避免与 `wc` 喜欢输出的前导制表符产生混淆。

测试小写字母是否正确非常简单 ➋。去除`guess`中的所有小写字母，看看结果是否为零（空）。

最后，为了检查用户是否已经猜过某个字母，将猜测转换为：将`guess`中与`guessed`变量中已出现的字母去除。结果是零（空）还是其他 ➌？

除了这些测试，成功让 `hangman` 游戏运行的关键在于：将原始单词中每个已猜字母的位置替换为短横线，然后将结果与原始单词进行比较，原始单词中没有任何字母被替换成短横线 ➍。如果它们不同（即单词中的一个或多个字母现在变成了短横线），则猜测的字母在单词中。举个例子，当单词是*cat*时，猜测字母*a*，`guessed`变量的值将是‘-a-’。

编写“猜单词”游戏的关键思想之一是，每次玩家做出正确猜测时，显示给玩家的部分填充单词变量`partial`都会被重建。由于变量`guessed`会累积玩家猜测的每个字母，`sed`转换将原单词中不在`guessed`字符串中的字母替换为短横线，就能完成这个操作 ➎。

#### *运行脚本*

“猜单词”游戏有一个可选参数：如果指定一个数字值作为参数，代码将使用该值作为允许的错误猜测次数，而不是默认的 6 次。Listing 12-4 显示了没有参数的情况下运行 `hangman` 脚本。

#### *结果*

```
$ hangman

steps from gallows: 6, word so far: -------------
Guess a letter: e

* Great! The letter "e" appears in the word!

guessed: e, steps from gallows: 6, word so far: -e--e--------
Guess a letter: i

* Great! The letter "i" appears in the word!

guessed: ei, steps from gallows: 6, word so far: -e--e--i-----
Guess a letter: o

* Great! The letter "o" appears in the word!

guessed: eio, steps from gallows: 6, word so far: -e--e--io----
Guess a letter: u

* Great! The letter "u" appears in the word!

guessed: eiou, steps from gallows: 6, word so far: -e--e--iou---
Guess a letter: m

* Nope, "m" does not appear in the word.

guessed: eioum, steps from gallows: 5, word so far: -e--e--iou---
Guess a letter: n

* Great! The letter "n" appears in the word!

guessed: eioumn, steps from gallows: 5, word so far: -en-en-iou---
Guess a letter: r

* Nope, "r" does not appear in the word.

guessed: eioumnr, steps from gallows: 4, word so far: -en-en-iou---
Guess a letter: s

* Great! The letter "s" appears in the word!

guessed: eioumnrs, steps from gallows: 4, word so far: sen-en-ious--
Guess a letter: t

* Great! The letter "t" appears in the word!

guessed: eioumnrst, steps from gallows: 4, word so far: sententious--
Guess a letter: l

* Great! The letter "l" appears in the word!

guessed: eioumnrstl, steps from gallows: 4, word so far: sententiousl-
Guess a letter: y

** You've been pardoned!! Well done! The word was "sententiously".

*** New Game! ***

steps from gallows: 6, word so far: ----------
Guess a letter: quit
```

*Listing 12-4：玩* `*hangman*` *shell 脚本游戏*

#### *破解脚本*

显然，使用 shell 脚本很难展示悬挂图形，所以我们采用了另一种方式，即计算“到达绞刑架的步骤”。不过，如果你有足够的动力，你可以预定义一系列“文本”图形，每一步一个，然后随着游戏进行逐步输出。或者，你也可以选择某种非暴力的替代方式！

注意，尽管可以选择两次相同的单词，但由于默认的单词列表包含 2,882 个不同的单词，发生这种情况的几率不大。不过，如果这是一个问题，选择单词的那一行也可以将所有以前的单词保存在一个变量中，并进行筛选，以确保没有重复的单词。

最后，如果你有动力的话，把猜测的字母列表按字母顺序排序会更好。有几种方法可以实现，但我们会使用 `sed|sort`。

### #85 州府问答游戏

一旦你有了从文件中随机选择一行的工具，就没有限制可以编写什么类型的问答游戏了。我们已经整理了美国所有 50 个州的州府列表，可以从 *[`www.nostarch.com/wcss2/`](http://www.nostarch.com/wcss2/)* 下载。将文件 *state.capitals.txt* 保存到你的 */usr/lib/games* 目录中。列表 12-5 中的脚本会从文件中随机选择一行，显示州名，然后要求用户输入匹配的首府。

#### *代码*

```
   #!/bin/bash
   # states--A state capital guessing game. Requires the state capitals
   #   data file state.capitals.txt.

   db="/usr/lib/games/state.capitals.txt"     # Format is State[tab]City.

   if [ ! -r "$db" ] ; then
     echo "$0: Can't open $db for reading." >&2
     echo "(get state.capitals.txt" >&2
     echo "save the file as $db and you're ready to play!)" >&2
     exit 1
   fi

   guesses=0; correct=0; total=0

   while [ "$guess" != "quit" ] ; do

     thiskey="$(randomquote $db)"

     # $thiskey is the selected line. Now let's grab state and city info, and
     #   then also have "match" as the all-lowercase version of the city name.

➊   state="$(echo $thiskey | cut -d\   -f1 | sed 's/-/ /g')"
     city="$(echo $thiskey | cut -d\   -f2 | sed 's/-/ /g')"
     match="$(echo $city | tr '[:upper:]' '[:lower:]')"

     guess="??" ; total=$(( $total + 1 )) ;

     echo ""
     echo "What city is the capital of $state?"

     # Main loop where all the action takes place. Script loops until
     #   city is correctly guessed or the user types "next" to
     #   skip this one or "quit" to quit the game.

     while [ "$guess" != "$match" -a "$guess" != "next" -a "$guess" != "quit" ]
     do
       /bin/echo -n "Answer: "
       read guess
       if [ "$guess" = "$match" -o "$guess" = "$city" ] ; then
         echo ""
         echo "*** Absolutely correct! Well done! ***"
         correct=$(( $correct + 1 ))
         guess=$match
       elif [ "$guess" = "next" -o "$guess" = "quit" ] ; then
         echo ""
         echo "$city is the capital of $state." # What you SHOULD have known :)
       else
         echo "I'm afraid that's not correct."
       fi
     done

   done

   echo "You got $correct out of $total presented."
   exit 0
```

*列表 12-5：* `*states*` *问答游戏脚本*

#### *工作原理*

对于这样一个有趣的游戏，`states` 只涉及非常简单的脚本编写。数据文件包含州名/首府对，州名和首府名称中的所有空格都被破折号替换，两个字段之间由一个空格分隔。因此，从数据中提取城市和州名非常简单 ➊。

每次猜测都会与城市名的小写版本（`match`）和正确大写的城市名进行比较，看看是否正确。如果不正确，则该猜测会与两个命令字 `next` 和 `quit` 进行比较。如果其中一个匹配，脚本会显示答案并根据需要提示下一个州或退出。如果都不匹配，猜测将被认为是错误的。

#### *运行脚本*

这个脚本没有参数或命令标志。只需启动它并开始游戏！

#### *结果*

准备好挑战自己，测试州府知识了吗？列表 12-6 展示了我们的州府知识技能！

```
$ states

What city is the capital of Indiana?
Answer: Bloomington
I'm afraid that's not correct.
Answer: Indianapolis

*** Absolutely correct! Well done! ***

What city is the capital of Massachusetts?
Answer: Boston

*** Absolutely correct! Well done! ***

What city is the capital of West Virginia?
Answer: Charleston

*** Absolutely correct! Well done! ***

What city is the capital of Alaska?
Answer: Fairbanks
I'm afraid that's not correct.
Answer: Anchorage
I'm afraid that's not correct.
Answer: Nome
I'm afraid that's not correct.
Answer: Juneau

*** Absolutely correct! Well done! ***

What city is the capital of Oregon?
Answer: quit

Salem is the capital of Oregon.
You got 4 out of 5 presented.
```

*列表 12-6：运行 `*states*` 问答游戏脚本*

幸运的是，这个游戏只跟踪最终正确的猜测，而不是你猜错了多少次，或者你是否跳到 Google 查找答案！

#### *破解脚本*

这个游戏最大的弱点可能就是它对拼写非常挑剔。一个有用的修改是添加代码来允许模糊匹配，例如，用户输入 `Juneu` 时能匹配到 Juneau。这可以通过修改过的 *Soundex 算法* 实现，在该算法中元音被移除，重复的字母被压缩成一个字母（例如，Annapolis 会变成 `npls`）。这可能对你来说有些过于宽容，但这个概念值得考虑。

和其他游戏一样，提供提示功能也很有用。也许在请求时，提示功能会显示正确答案的第一个字母，并在游戏进行过程中记录使用的提示次数。

尽管这个游戏是为州首府设计的，但修改脚本以处理任何类型的配对数据文件将是微不足道的。例如，使用不同的文件，你可以创建一个意大利词汇测验、一个国家/货币配对测试，或者一个政治家/政党配对测试。正如我们在 Unix 中反复看到的，编写一些合理通用的程序可以让它以有用的甚至是意想不到的方式被重复使用。

### #86 这个数字是质数吗？

质数是只能被自身整除的数字，例如 7。另一方面，6 和 8 不是质数。识别单一数字的质数很简单，但当我们处理更大的数字时，情况就变得复杂起来。

有多种数学方法可以判断一个数字是否是质数，但我们还是坚持使用暴力法，尝试所有可能的除数，看看是否有余数为零，正如列表 12-7 所示。

#### *代码*

```
   #!/bin/bash
   # isprime--Given a number, ascertain whether it's a prime. This uses what's
   #   known as trial division: simply check whether any number from 2 to (n/2)
   #   divides into the number without a remainder.

     counter=2
   remainder=1

   if [ $# -eq 0 ] ; then
     echo "Usage: isprime NUMBER" >&2
     exit 1
   fi

   number=$1

   # 3 and 2 are primes, 1 is not.

   if [ $number -lt 2 ] ; then
     echo "No, $number is not a prime"
     exit 0
   fi

   # Now let's run some calculations.

➊ while [ $counter -le $(expr $number / 2) -a $remainder -ne 0 ]
   do
     remainder=$(expr $number % $counter)  # '/' is divide, '%' is remainder
     # echo "  for counter $counter, remainder = $remainder"
     counter=$(expr $counter + 1)
   done

   if [ $remainder -eq 0 ] ; then
     echo "No, $number is not a prime"
   else
     echo "Yes, $number is a prime"
   fi
   exit 0
```

*列表 12-7：* `*isprime*` *脚本*

#### *它是如何工作的*

这个脚本的核心在于`while`循环，所以请更仔细地查看它在➊的位置。如果我们尝试的`number`是 77，那么条件语句将测试以下内容：

```
while [ 2 -le 38 -a 1 -ne 0 ]
```

很显然这是错误的：77 不能被 2 整除。每次代码测试一个潜在的除数（`$counter`），如果发现它不能整除，就会计算余数（`$number % $counter`），并将`$counter`递增 1。脚本按部就班地继续执行。

#### *运行脚本*

让我们选几个看起来像是质数的数字，在列表 12-8 中进行测试。

```
$ isprime 77
No, 77 is not a prime
$ isprime 771
No, 771 is not a prime
$ isprime 701
Yes, 701 is a prime
```

*列表 12-8：运行* `*isprime*` *Shell 脚本并对一些数字进行测试*

如果你感兴趣，可以在`while`循环中取消注释`echo`语句，查看计算过程，并感受脚本在找出一个能整除该数字且没有余数的除数时的速度——是多快还是多慢。事实上，我们就做这个测试，看看 77 的情况，正如在列表 12-9 中所示。

#### *结果*

```
$ isprime 77
  for counter 2, remainder = 1
  for counter 3, remainder = 2
  for counter 4, remainder = 1
  for counter 5, remainder = 2
  for counter 6, remainder = 5
  for counter 7, remainder = 0
No, 77 is not a prime
```

*列表 12-9：运行* `*isprime*` *脚本并取消注释调试行*

#### *破解脚本*

这个脚本中实现数学公式的方式有一些低效的地方，导致它执行得非常慢。例如，考虑`while`循环的条件。我们一直在计算`$(expr $number / 2)`，而实际上可以只计算一次这个值，并在每次后续迭代中使用计算出来的结果，避免每次都启动一个子 shell 并调用`expr`来得出与上次迭代相同的结果。

还有一些更智能的算法可以用来测试质数，这些算法值得探索，包括那种非常有趣的埃拉托斯特尼筛法，以及更现代的筛法如桑达拉姆筛法和更复杂的阿特金筛法。可以在线查看它们，并测试一下你的电话号码（没有破折号！）是否是质数。

### #87 骰子游戏

这是一个对任何喜欢桌面游戏的人来说都很有用的脚本，特别是像*龙与地下城*这样的角色扮演游戏。

这些游戏的普遍看法是它们只是不断地掷骰子，实际上这个看法是正确的。这一切都与概率有关，因此有时你会掷一个 20 面骰子，其他时候你会掷六个 6 面骰子。骰子是如此简单的随机数生成器，以至于很多游戏都使用它们，不管是一个骰子、两个（想想*大富翁*或*麻烦*），还是更多。

它们都很容易建模，这正是清单 12-10 中脚本的作用，允许用户指定需要多少什么样的骰子，然后“掷”出它们，并提供一个总和。

#### *代码*

```
   #!/bin/bash
   # rolldice--Parse requested dice to roll and simulate those rolls.
   #   Examples: d6 = one 6-sided die
   #             2d12 = two 12-sided dice
   #             d4 3d8 2d20 = one 4-side die, three 8-sided, and two 20-sided dice

   rolldie()
   {
     dice=$1
     dicecount=1
     sum=0

     # First step: break down arg into MdN.

➊   if [ -z "$(echo $dice | grep 'd')" ] ; then
       quantity=1
       sides=$dice
     else
       quantity=$(echo $dice | ➋cut -dd -f1)
       if [ -z "$quantity" ] ; then       # User specified dN, not just N.
         quantity=1
       fi
       sides=$(echo $dice | cut -dd -f2)
     fi
     echo "" ; echo "rolling $quantity $sides-sided die"
     # Now roll the dice...

     while [ $dicecount -le $quantity ] ; do
➌     roll=$(( ( $RANDOM % $sides ) + 1 ))
       sum=$(( $sum + $roll ))
       echo " roll #$dicecount = $roll"
       dicecount=$(( $dicecount + 1 ))
     done

     echo I rolled $dice and it added up to $sum
   }

   while [ $# -gt 0 ] ; do
     rolldie $1
     sumtotal=$(( $sumtotal + $sum ))
     shift
   done

   echo ""
   echo "In total, all of those dice add up to $sumtotal"
   echo ""
   exit 0
```

*清单 12-10：* `*rolldice*` *脚本*

#### *它是如何工作的*

这个脚本围绕一行简单的代码展开，它通过引用`$RANDOM` ➌来调用 bash 的随机数生成器。这是关键行；其他的只是点缀。

另一个有趣的部分是骰子描述被拆解的地方 ➊，因为脚本支持这三种表示法：`3d8`、`d6` 和 `20`。这是标准的游戏表示法，为了方便：骰子的数量 + *d* + 骰子应有的面数。例如，`2d6`意味着两个 6 面骰子。看看你能否弄明白每种是如何处理的。

对于这么一个简单的脚本，输出还挺多的。你可能想根据自己的喜好调整它，但在这里你可以看到这个语句只是一个方便的方式来验证它是否正确解析了骰子或骰子请求。

哦，还有那个`cut`调用 ➋？记住，`-d`表示字段分隔符，因此`-dd`只是告诉它使用字母*d*作为分隔符，这是该骰子表示法所需的。

#### *运行脚本*

让我们从简单的开始：在清单 12-11 中，我们将使用两个 6 面骰子，就像我们在玩*大富翁*一样。

```
$ rolldice 2d6
rolling 2 6-sided die
  roll #1 = 6
  roll #2 = 2
I rolled 2d6 and it added up to 8
In total, all of those dice add up to 8
$ rolldice 2d6
rolling 2 6-sided die
  roll #1 = 4
  roll #2 = 2
I rolled 2d6 and it added up to 6
In total, all of those dice add up to 6
```

*清单 12-11：用一对六面骰子测试* `*rolldice*` *脚本*

注意到第一次“掷”这两个骰子时，它们分别掷出了 6 和 2，但第二次却是 4 和 2。

怎么样，来一局快速的*雅兹*掷骰吗？够简单的。我们将在清单 12-12 中掷五个六面骰子。

```
$ rolldice 5d6
rolling 5 6-sided die
  roll #1 = 2
  roll #2 = 1
  roll #3 = 3
  roll #4 = 5
  roll #5 = 2
I rolled 5d6 and it added up to 13
In total, all of those dice add up to 13
```

*清单 12-12：用五个六面骰子测试* `*rolldice*` *脚本*

不算很好的掷骰结果：1、2、2、3、5。如果我们在玩*雅兹*，我们会保留一对 2，然后重新掷其他所有的。

当你需要掷一组更复杂的骰子时，事情变得更加有趣。在清单 12-13 中，让我们尝试两个 18 面骰子，一个 37 面骰子和一个 3 面骰子（因为我们不必担心 3D 几何形状的限制）。

```
$ rolldice 2d18 1d37 1d3
rolling 2 18-sided die
  roll #1 = 16
  roll #2 = 14
I rolled 2d18 and it added up to 30
rolling 1 37-sided die
  roll #1 = 29
I rolled 1d37 and it added up to 29
rolling 1 3-sided die
  roll #1 = 2
I rolled 1d3 and it added up to 2
In total, all of those dice add up to 61
```

*清单 12-13：用各种骰子类型运行* `*rolldice*` *脚本*

很酷吧？几次掷骰子后，这一堆杂七杂八的骰子分别掷出了 22、49 和 47。现在你知道了，玩家们！

#### *破解脚本*

这个脚本中没有太多可以修改的地方，因为任务本身非常简单。我们唯一的建议是微调程序输出的量。例如，像`5d6: 2 3 1 3 7 = 16`这样的表示方式会更节省空间。

### #88 Acey Deucey

在本章的最后一个脚本中，我们将创建纸牌游戏 Acey Deucey，这意味着我们需要弄清楚如何创建并“洗牌”一副扑克牌，以得到随机化的结果。这很棘手，但你为这个游戏写的函数将为你提供一个通用的解决方案，可以用来制作像 21 点、或者甚至是打扑克和“捉鱼”之类的更复杂游戏。

这个游戏很简单：发两张牌，然后赌下一张翻出来的牌是否在这两张牌的之间。花色无关紧要；只看牌面大小，平局算输。因此，如果你翻出来的是一张红桃 6 和一张黑桃 9，而第三张牌是方块 6，那就是失败。黑桃 4 也是失败。但梅花 7 则是胜利。

所以这里有两个任务：整个牌组的模拟和游戏本身的逻辑，包括询问用户是否要下注。哦，还有一件事：如果发出的两张牌是相同的牌面大小，那就没有意义下注，因为你无法获胜。

这将是一个有趣的脚本。准备好了吗？那么请访问列表 12-14。

#### *代码*

```
   #!/bin/bash
   # aceyduecey: Dealer flips over two cards, and you guess whether the
   #   next card from the deck will rank between the two. For example,
   #   with a 6 and an 8, a 7 is between the two, but a 9 is not.

   function initializeDeck
   {
       # Start by creating the deck of cards.

       card=1
       while [ $card –le 52 ]         # 52 cards in a deck. You knew that, right?
       do
➊       deck[$card]=$card
         card=$(( $card + 1 ))
       done
   }

   function shuffleDeck
   {
 # It's not really a shuffle. It's a random extraction of card values
       #   from the 'deck' array, creating newdeck[] as the "shuffled" deck.

       count=1

       while [ $count != 53 ]
       do
         pickCard
➋       newdeck[$count]=$picked
         count=$(( $count + 1 ))
       done
   }

➌ function pickCard
   {
       # This is the most interesting function: pick a random card from
       #   the deck. Uses the deck[] array to find an available card slot.

       local errcount randomcard

       threshold=10      # Max guesses for a card before we fall through
       errcount=0

       # Randomly pick a card that hasn't already been pulled from the deck
       #   a max of $threshold times. Fall through on fail (to avoid a possible
       #   infinite loop where it keeps guessing the same already dealt card).

➍   while [ $errcount -lt $threshold ]
       do
         randomcard=$(( ( $RANDOM % 52 ) + 1 ))
         errcount=$(( $errcount + 1 ))

         if [ ${deck[$randomcard]} -ne 0 ] ; then
           picked=${deck[$randomcard]}
           deck[$picked]=0    # Picked--remove it.
           return $picked
         fi
       done

       # If we get here, we've been unable to randomly pick a card, so we'll
       #   just step through the array until we find an available card.

       randomcard=1

➎   while [ ${newdeck[$randomcard]} -eq 0 ]
       do
         randomcard=$(( $randomcard + 1 ))
       done

       picked=$randomcard
       deck[$picked]=0      # Picked--remove it.

       return $picked
   }

 function showCard
   {
      # This uses a div and a mod to figure out suit and rank, though
      #   in this game, only rank matters. Still, presentation is
      #   important, so this helps make things pretty.

      card=$1

      if [ $card -lt 1 -o $card -gt 52 ] ; then
        echo "Bad card value: $card"
        exit 1
      fi

      # div and mod -- see, all that math in school wasn't wasted!

➏    suit="$(( ( ( $card - 1) / 13 ) + 1))"
      rank="$(( $card % 13))"

      case $suit in
        1 ) suit="Hearts"   ;;
        2 ) suit="Clubs"    ;;
        3 ) suit="Spades"   ;;
        4 ) suit="Diamonds" ;;
        * ) echo "Bad suit value: $suit"
            exit 1

      esac

      case $rank in
        0 ) rank="King"    ;;
        1 ) rank="Ace"     ;;
        11) rank="Jack"    ;;
        12) rank="Queen"   ;;
      esac

      cardname="$rank of $suit"
   }

➐ function dealCards
   {
       # Acey Deucey has two cards flipped up...

       card1=${newdeck[1]}    # Since deck is shuffled, we take
       card2=${newdeck[2]}    #   the top two cards from the deck
       card3=${newdeck[3]}    #   and pick card #3 secretly.

       rank1=$(( ${newdeck[1]} % 13 ))  # And let's get the rank values
       rank2=$(( ${newdeck[2]} % 13 ))  #   to make subsequent calculations easy.
       rank3=$(( ${newdeck[3]} % 13 ))

       # Fix to make the king: default rank = 0, make rank = 13.

       if [ $rank1 -eq 0 ] ; then
         rank1=13;
       fi
 if [ $rank2 -eq 0 ] ; then
         rank2=13;
       fi
       if [ $rank3 -eq 0 ] ; then
         rank3=13;
       fi

       # Now let's organize them so that card1 is always lower than card2.

➑     if [ $rank1 -gt $rank2 ] ; then
         temp=$card1; card1=$card2; card2=$temp
         temp=$rank1; rank1=$rank2; rank2=$temp
       fi

       showCard $card1 ; cardname1=$cardname
       showCard $card2 ; cardname2=$cardname

       showCard $card3 ; cardname3=$cardname # Shhh, it's a secret for now.

➒     echo "I've dealt:" ; echo "   $cardname1" ; echo "   $cardname2"
   }

   function introblurb
   {
   cat << EOF

   Welcome to Acey Deucey. The goal of this game is for you to correctly guess
   whether the third card is going to be between the two cards I'll pull from
   the deck. For example, if I flip up a 5 of hearts and a jack of diamonds,
   you'd bet on whether the next card will have a higher rank than a 5 AND a
   lower rank than a jack (that is, a 6, 7, 8, 9, or 10 of any suit).

   Ready? Let's go!

   EOF
   }

   games=0
   won=0

   if [ $# -gt 0 ] ; then    # Helpful info if a parameter is specified
     introblurb
   fi

   while [ /bin/true ] ; do

     initializeDeck
     shuffleDeck
     dealCards

     splitValue=$(( $rank2 - $rank1 ))

     if [ $splitValue -eq 0 ] ; then
       echo "No point in betting when they're the same rank!"
       continue
     fi

     /bin/echo -n "The spread is $splitValue. Do you think the next card will "
     /bin/echo -n "be between them? (y/n/q) "
     read answer

     if [ "$answer" = "q" ] ; then
       echo ""
       echo "You played $games games and won $won times."
       exit 0
     fi

     echo "I picked: $cardname3"

     # Is it between the values? Let's test. Remember, equal rank = lose.

➓   if [ $rank3 -gt $rank1 -a $rank3 -lt $rank2 ] ; then # Winner!
       winner=1
     else
       winner=0
     fi

     if [ $winner -eq 1 -a "$answer" = "y" ] ; then
       echo "You bet that it would be between the two, and it is. WIN!"
       won=$(( $won + 1 ))
     elif [ $winner -eq 0 -a "$answer" = "n" ] ; then
       echo "You bet that it would not be between the two, and it isn't. WIN!"
       won=$(( $won + 1 ))
     else
       echo "Bad betting strategy. You lose."
     fi

     games=$(( $games + 1 )) # How many times do you play?

   done

   exit 0
```

*列表 12-14：* `*aceydeucey*` *脚本游戏*

#### *它是如何工作的*

模拟一副洗牌后的扑克牌并不容易。问题在于如何呈现这些牌本身，以及如何“洗牌”或者将原本整齐有序的牌组随机排序。

为了解决这个问题，我们创建了两个包含 52 个元素的数组：`deck[]` ➊ 和 `newdeck[]` ➋。前者是一个有序的卡牌数组，每个值在“被选中”并放入`newdeck[]`的随机位置时都会被替换为`-1`。然后，`newdeck[]`数组就是“洗牌”后的牌组。虽然在这个游戏中我们只会使用前三张牌，但相较于特定的解法，一般的解法更值得探讨。

这意味着这个脚本有些大材小用。不过，嘿，它很有趣。 ![image](img/common1.jpg)

让我们一步步查看这些函数，了解它们是如何工作的。首先，初始化牌组非常简单，正如你翻回去查看`initializeDeck`函数时看到的那样。

同样，`shuffleDeck`出奇地简单，因为所有的工作实质上都是在`pickCard`函数中完成的。`shuffleDeck`仅仅是遍历`deck[]`中的 52 个位置，随机选择一个尚未被选中的值，并将其保存到`newdeck[]`的*n*位置。

我们来看一下 `pickCard` ➌，因为这部分是洗牌的关键。这个函数分为两个块：第一个尝试随机选择一张可用的牌，并给它 `$threshold` 次机会成功。随着函数的反复调用，最初的调用总是会成功，但在过程中，一旦 50 张牌已经移入 `newdeck[]`，可能会出现 10 次随机猜测都失败的情况。这就是 ➍ 处的 `while` 代码块。

一旦 `$errcount` 等于 `$threshold`，为了提高性能，我们基本上放弃了这个策略，转而使用第二块代码：逐张检查牌堆，直到找到一张可用的牌。这就是 ➎ 处的代码块。

如果你考虑这个策略的含义，你会意识到，阈值设置得越低，`newdeck` 的顺序性就越高，特别是在牌堆后期。极端情况下，`threshold = 1` 将会得到一个有序的牌堆，其中 `newdeck[]` = `deck[]`。10 是正确的值吗？这有点超出了本书的范围，但如果有人想通过实验找出最合适的随机性与性能平衡，我们欢迎他们通过邮件联系我们！

`showCard` 函数很长，但其中大部分行其实只是为了让结果更漂亮。整个牌堆模拟的核心部分包含在 ➏ 处的两行代码中。

对于这个游戏，花色无关紧要，但你可以看到，对于每一张牌的数值，等级会是 0–12，花色会是 0–3。牌的属性只是需要映射到易于用户理解的值上。为了方便调试，梅花 6 的等级是 6，王牌的等级是 1。国王的默认等级是 0，但我们将其调整为等级 13，这样计算才能正确。

`dealCards` 函数 ➐ 是实际的 Acey Deucey 游戏逻辑所在：之前的所有函数都致力于为任何扑克牌游戏实现有用的功能集。`dealCards` 函数发出游戏所需的所有三张牌，尽管第三张牌在玩家下注之前是隐藏的。这只是为了简化操作——并不是为了让计算机作弊！在这里，你也可以看到为国王 = 13 的场景单独存储的等级值（`$rank1`，`$rank2` 和 `$rank3`）。为了简化操作，前两张牌会被排序，使得较低等级的牌总是排在前面。这就是 ➑ 处的 `if` 代码块。

在 ➒，是时候展示已经发出的牌了。最后一步是展示牌，检查排名是否匹配（如果匹配，我们会跳过提示，让用户决定是否下注），然后测试第三张牌是否在前两张牌之间。这项测试在 ➓ 的代码块中完成。

最后，下注结果是棘手的。如果你赌抽到的牌会在前两张牌之间，结果确实如此，或者你赌它不会在两张牌之间且它没有，那么你就是赢家。否则，你就输了。这个结果会在最后的代码块中得出。

#### *运行脚本*

指定任何起始参数，游戏会给你一个简单的玩法说明。否则，你只需直接跳入游戏。

让我们看看示例 12-15 中的介绍。

#### *结果*

```
$ aceydeucey intro

Welcome to Acey Deucey. The goal of this game is for you to correctly guess
whether the third card is going to be between the two cards I'll pull from
the deck. For example, if I flip up a 5 of hearts and a jack of diamonds,
you'd bet on whether the next card will have a higher rank than a 5 AND a
lower rank than a jack (that is, a 6, 7, 8, 9, or 10 of any suit).

Ready? Let's go!

I've dealt:
   3 of Hearts
   King of Diamonds
The spread is 10\. Do you think the next card will be between them? (y/n/q) y
I picked: 4 of Hearts
You bet that it would be between the two, and it is. WIN!

I've dealt:
   8 of Clubs
   10 of Hearts
The spread is 2\. Do you think the next card will be between them? (y/n/q) n
I picked: 6 of Diamonds
You bet that it would not be between the two, and it isn't. WIN!

I've dealt:
   3 of Clubs
   10 of Spades
The spread is 7\. Do you think the next card will be between them? (y/n/q) y
I picked: 5 of Clubs
You bet that it would be between the two, and it is. WIN!

I've dealt:
   5 of Diamonds
   Queen of Spades
The spread is 7\. Do you think the next card will be between them? (y/n/q) q

You played 3 games and won 3 times.
```

*示例 12-15：玩* `*aceydeucey*` *脚本游戏*

#### *破解脚本*

关于是否以 10 为阈值足够充分地洗牌这个问题依然存在疑问；这是一个可以明确改进的地方。另一个不确定的地方是是否显示分差（两张卡牌的排名差异）是有益的。当然，在真正的游戏中你是不会这样做的；玩家需要自己弄清楚。

另外，你也可以朝相反的方向进行，计算两张任意卡牌之间的概率。我们来思考一下：任意一张卡牌被抽到的概率是 1/52。如果牌堆中剩下 50 张卡，因为已经发了两张牌，那么任意一张卡被抽到的概率是 1/50。由于花色无关，所以任意不同排名的卡牌出现的机会是 4/50。因此，某一特定分差的概率是（该分差中的卡牌数量 × 4）/50。如果发了 5 和 10，那么分差为 4，因为可能的获胜牌是 6、7、8 或 9。所以获胜的概率是 4 × 4 / 50。明白我们的意思了吗？

最后，像所有基于命令行的游戏一样，界面也可以做得更好。我们将这部分留给你来处理。我们还会留给你一个问题：你可以探索这个方便的扑克牌功能库来开发其他哪些游戏。
