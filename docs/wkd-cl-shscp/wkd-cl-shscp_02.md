## 第二章：**缺失的代码库**

![image](img/common4.jpg)

Unix 的最大优势之一在于，它允许你通过以新颖的方式将旧命令组合起来，创建新的命令。尽管 Unix 包括数百个命令，并且有成千上万种组合它们的方法，但你仍然会遇到一些情况，没有任何一个命令能完全满足需求。本章将重点介绍一些垫脚石，帮助你在 shell 脚本的世界中创建更智能、更复杂的程序。

还有一件事我们应该提前说明：shell 脚本编程环境并不像真实的编程环境那么复杂。Perl、Python、Ruby 甚至 C 语言都有提供扩展功能的结构和库，但 shell 脚本更像是一个“自创”的世界。本章中的脚本将帮助你在这个世界中找到自己的路。它们是构建块，帮助你编写本书后面将介绍的酷炫 shell 脚本。

编写脚本的挑战之一，来自于不同版本的 Unix 和不同的 GNU/Linux 发行版之间的微妙差异。尽管 IEEE 的 POSIX 标准应该为 Unix 的实现提供一个共同的功能基础，但在 Red Hat GNU/Linux 环境中使用了一年之后，再使用 OS X 系统可能会感到困惑。命令不同，位置不同，且它们的命令标志也常常有所不同。这些差异可能使得编写 shell 脚本变得棘手，但我们将学习一些技巧，帮助你应对这些变化。

### 什么是 POSIX？

Unix 的早期就像是西部荒野，许多公司在创新并将操作系统带向不同的方向，同时还向客户保证所有这些新版本彼此兼容，并且与其他 Unix 系统没有区别。电气和电子工程师协会（IEEE）介入，并在所有主要 Unix 供应商的巨大努力下，创建了一个 Unix 的标准定义，称为可移植操作系统接口（Portable Operating System Interface），简称*POSIX*，所有商业和开源的 Unix 实现都以此为衡量标准。你不能单纯地购买一个 POSIX 操作系统，但你运行的 Unix 或 GNU/Linux 通常是 POSIX 兼容的（尽管是否需要 POSIX 标准一直存在争议，特别是当 GNU/Linux 本身已成为事实上的标准时）。

与此同时，即使是 POSIX 兼容的 Unix 实现也可能有所不同。章节后面将提到的一个例子涉及`echo`命令。某些版本的`echo`支持`-n`选项，该选项禁用命令执行时的尾随换行符。其他版本的`echo`支持`\c`转义序列作为特殊的“不包含换行符”标记，而还有一些版本无法避免输出末尾的换行符。更有趣的是，一些 Unix 系统具有内置的`echo`函数，它忽略`-n`和`\c`选项，同时也有独立的二进制文件`/bin/echo`，可以理解这些选项。这使得在 Shell 脚本中提示输入变得棘手，因为脚本应该尽可能在多个 Unix 系统中表现一致。因此，对于功能性的脚本，必须对`echo`命令进行规范化，以确保它在不同系统上表现相同。稍后在本章的脚本#8 中，位于第 33 页，我们将看到如何将`echo`包装在 Shell 脚本中，以创建这种规范化版本的命令。

**注意**

*本书中的一些脚本利用了 bash 风格的特性，这些特性可能并非所有 POSIX 兼容的 Shell 都支持。*

话不多说——让我们开始查看可以加入我们 Shell 脚本库的脚本吧！

### #1 在 PATH 中查找程序

使用环境变量（如`MAILER`和`PAGER`）的 Shell 脚本存在一个潜在的危险：它们的某些设置可能指向不存在的程序。如果你之前没有接触过这些环境变量，`MAILER`应该设置为你偏好的电子邮件程序（如`/usr/bin/mailx`），而`PAGER`应该设置为你用来逐页查看长文档的程序。例如，如果你决定通过使用`PAGER`设置来显示脚本输出，而不是使用系统默认的分页程序（常见的值是`more`或`less`程序），你如何确保`PAGER`环境变量设置为有效的程序呢？

第一个脚本解决了如何测试给定程序是否可以在用户的`PATH`中找到的问题。它同时展示了多种 Shell 脚本技巧，包括脚本函数和变量切片。清单 1-1 展示了如何验证路径是否有效。

#### *代码*

```
   #!/bin/bash
   # inpath--Verifies that a specified program is either valid as is
   #   or can be found in the PATH directory list

   in_path()
   {
     # Given a command and the PATH, tries to find the command. Returns 0 if
     #   found and executable; 1 if not. Note that this temporarily modifies
     #   the IFS (internal field separator) but restores it upon completion.

     cmd=$1        ourpath=$2         result=1
     oldIFS=$IFS   IFS=":"

     for directory in "$ourpath"
     do
       if [ -x $directory/$cmd ] ; then
         result=0      # If we're here, we found the command.
       fi
     done

     IFS=$oldIFS
     return $result
   }

   checkForCmdInPath()
   {
     var=$1

     if [ "$var" != "" ] ; then
➊     if [ "${var:0:1}" = "/" ] ; then
➋       if [ !  -x $var ] ; then
           return 1
         fi
➌     elif !  in_path $var "$PATH" ; then
         return 2
       fi
     fi
   }
```

*清单 1-1：* `*inpath*` *Shell 脚本功能*

如第零章所述，我们建议你在主目录中创建一个名为*scripts*的新目录，并将该完全限定的目录名添加到你的`PATH`变量中。使用`echo $PATH`来查看当前的`PATH`，并编辑你的登录脚本（*login*、*profile*、*bashrc*或*bash_profile*，具体取决于所用的 Shell），以适当地修改`PATH`。更多详细信息请参见“配置登录脚本”，详见第 4 页。

**注意**

*如果你在终端使用* `*ls*` *命令列出文件，一些特殊文件，如* .bashrc *或* .bash_profile*，可能一开始不会显示。这是因为以点号开头的文件（如* .bashrc*）被文件系统视为“隐藏”文件。（这实际上是 Unix 初期出现的一个 bug，但后来变成了特性。）要列出目录中的所有文件，包括隐藏文件，可以使用* `*-a*` *参数和* `*ls*`。

再次强调一下，我们假设你使用的是 bash 作为所有这些脚本的 shell。请注意，脚本明确设置了第一行（称为 *shebang*）来调用 `/bin/bash`。许多系统也支持 `/usr/bin/env bash` 作为脚本的运行时环境。

**关于注释的说明**

我们曾经纠结是否要包含每个脚本如何工作的详细解释。在某些情况下，我们会在代码后提供对一些复杂编码片段的解释，但通常我们会使用代码注释在上下文中解释发生了什么。请注意以 `#` 符号开头的行，或者有时是代码行中 `#` 后面的任何内容。

由于你很可能会阅读其他人的脚本（当然不是我们的脚本！），因此练习通过阅读注释来弄清楚脚本中的内容非常有用。注释也是编写自己脚本时的一个好习惯，可以帮助你定义在特定代码块中要完成的任务。

#### *工作原理*

使 `checkForCmdInPath` 正常工作的关键是能够区分仅包含程序名称（如 `echo`）的变量和包含完整目录路径加文件名（如 `/bin/echo`）的变量。它通过检查给定值的第一个字符是否为 `/` 来实现这一点；因此，我们需要将第一个字符与变量值的其余部分隔离开来。

请注意，变量切片语法 `${var:0:1}` 在 ➊ 处是一个简写表示法，允许你在字符串中指定子字符串，从偏移量开始并持续到给定的长度（如果没有提供长度，则返回字符串的其余部分）。例如，表达式 `${var:10}` 将返回从第 10 个字符开始的 `$var` 的剩余值，而 `${var:10:5}` 将子字符串限制为第 10 到第 15 个字符之间的字符（包括 10 和 15）。你可以通过以下方式理解我们的意思：

```
$ var="something wicked this way comes..."
$ echo ${var:10}
wicked this way comes...
$ echo ${var:10:6}
wicked
$
```

在列表 1-1 中，语法仅用于查看指定路径是否有前导斜杠。一旦我们确定传递给脚本的路径是否以斜杠开头，我们就检查是否可以在文件系统中找到该路径。如果路径以`/`开头，我们假设给定的路径是绝对路径，并使用`-x` bash 运算符 ➋ 检查它是否存在。否则，我们将该值传递给`inpath`函数 ➌，看看它是否能在默认的`PATH`中设置的任何目录中找到。

#### *运行脚本*

要将此脚本作为独立程序运行，我们首先需要在文件的末尾添加一小段命令。这些命令将执行获取用户输入并将其传递给我们编写的函数的基本工作，如下所示。

```
if [ $# -ne 1 ] ; then
  echo "Usage: $0 command" >&2
  exit 1
fi

checkForCmdInPath "$1"
case $? in
  0 ) echo "$1 found in PATH"                   ;;
  1 ) echo "$1 not found or not executable"     ;;
  2 ) echo "$1 not found in PATH"               ;;
esac

exit 0
```

一旦你添加了代码，就可以直接调用脚本，如下所示的“结果”部分。但是，在完成脚本后，确保删除或注释掉这段附加代码，这样它就可以作为库函数在以后使用，而不会搞乱其他内容。

#### *结果*

为了测试脚本，让我们用三种程序的名称来调用`inpath`：一个存在的程序，一个存在但不在`PATH`中的程序，以及一个不存在但有完整文件名和路径的程序。列表 1-2 展示了脚本的示例测试。

```
$ inpath echo
echo found in PATH
$ inpath MrEcho
MrEcho not found in PATH
$ inpath /usr/bin/MrEcho
/usr/bin/MrEcho not found or not executable
```

*列表 1-2：测试* `*inpath*` *脚本*

我们添加的最后一块代码将`in_path`函数的结果转换为更易读的格式，现在我们可以轻松地看到，三种情况都按预期处理了。

#### *破解脚本*

如果你想成为这里第一个脚本的代码忍者，可以将表达式`${var:0:1}`换成它更复杂的版本：`${var%${var#?}}`。这就是 POSIX 变量切片方法。看似复杂的语法实际上是两个嵌套的字符串切片操作。内层调用`${var#?}`提取除了`var`的第一个字符之外的所有内容，其中`#`表示删除给定模式的第一个实例，而`?`是一个正则表达式，匹配恰好一个字符。

接下来，调用`${var%*pattern*}`会生成一个子字符串，去掉指定模式后的剩余部分。在这种情况下，删除的模式是内层调用的结果，因此剩下的就是字符串的第一个字符。

如果这种 POSIX 表示法对你来说太复杂，大多数 shell（包括 bash、ksh 和 zsh）都支持另一种变量切片方法，`${*varname*:*start*:*size*}`，这在脚本中也有使用。

当然，如果你不喜欢这些提取第一个字符的技术，你还可以使用系统调用：`$(echo $var | cut -c1)`。在 bash 编程中，通常会有多种方式来解决一个给定的问题，无论是提取、转换，还是以不同的方式从系统加载数据。重要的是要意识到并理解，“多种方式解题”并不意味着某一种方式比其他方式更好。

同样，如果你想创建一个版本的脚本，能够区分它是在独立运行还是从另一个脚本中调用，考虑在开头添加一个条件测试，正如这里所示：

```
if [ "$BASH_SOURCE" = "$0" ]
```

我们将把剩下的代码片段留给你，亲爱的读者，通过一些实验来完成！

**注意**

*脚本 #47 在 第 150 页 是一个与此脚本紧密相关的有用脚本。它验证了`*PATH*`中的两个目录以及用户登录环境中的环境变量。*

### #2 验证输入：仅限字母数字

用户经常忽视指示，输入不一致、格式不正确或语法错误的数据。作为一个 Shell 脚本开发者，你需要在这些问题变成麻烦之前，识别并标记这些错误。

一个典型的情况涉及文件名或数据库键。你的程序提示用户输入一个字符串，应该是*字母数字*的，只包含大写字母、小写字母和数字——没有标点符号，没有特殊字符，没有空格。用户输入的是有效字符串吗？这就是清单 1-3 中测试的内容。

#### *代码*

```
   #!/bin/bash
   # validAlphaNum--Ensures that input consists only of alphabetical
   #   and numeric characters

   validAlphaNum()
   {
     # Validate arg: returns 0 if all upper+lower+digits; 1 otherwise

     # Remove all unacceptable chars.
➊   validchars="$(echo $1 | sed -e 's/[^[:alnum:]]//g')"

➋   if [ "$validchars" = "$1" ] ; then
       return 0
     else
       return 1
     fi
   }

   # BEGIN MAIN SCRIPT--DELETE OR COMMENT OUT EVERYTHING BELOW THIS LINE IF
   #   YOU WANT TO INCLUDE THIS IN OTHER SCRIPTS.
   # =================
   /bin/echo -n "Enter input: "
   read input

   # Input validation
   if ! validAlphaNum "$input" ; then
     echo "Please enter only letters and numbers." >&2
     exit 1
   else
     echo "Input is valid."
   fi

   exit 0
```

*清单 1-3：`*validalnum*` 脚本*

#### *工作原理*

这个脚本的逻辑很简单。首先，使用基于`sed`的转换创建输入信息的新版本，去除所有无效字符 ➊。然后，将新版本与原始版本进行比较 ➋。若两者相同，表示一切正常。如果不同，说明转换过程中丢失了不属于可接受字符集（字母加数字）的数据，输入无效。

之所以有效，是因为`sed`替换会去除所有不在`[:alnum:]`集合中的字符，这是 POSIX 正则表达式中表示所有字母数字字符的简写。如果这个转换后的值与之前输入的原始值不匹配，就揭示了输入字符串中存在非字母数字字符，从而表示输入无效。该函数返回非零结果以指示问题。请记住，我们只期望 ASCII 文本。

#### *运行脚本*

这个脚本是自包含的。它会提示用户输入，然后告知输入是否有效。然而，这个函数的更典型使用方式是将其复制并粘贴到另一个 shell 脚本的顶部，或将其作为库的一部分引用，如在脚本 #12 中展示的第 42 页。

`validalnum` 也是一个很好的通用 shell 脚本编程技巧示例。编写函数后再进行测试，然后再将它们集成到更大、更复杂的脚本中。这样做，你将避免很多麻烦。

#### *结果*

`validalnum` shell 脚本很容易使用，它会提示用户输入一个字符串进行验证。列表 1-4 展示了脚本如何处理有效和无效的输入。

```
$ validalnum
Enter input: valid123SAMPLE
Input is valid.
$ validalnum
Enter input: this is most assuredly NOT valid, 12345
Please enter only letters and numbers.
```

*列表 1-4：测试* `*validalnum*` *脚本*

#### *黑客脚本*

这种“去除有效字符，看看剩下什么”的方法很好，因为它很灵活，特别是当你记得将输入变量和匹配模式（或根本不使用模式）都用双引号括起来，以避免空输入错误时。空模式在脚本编写中是一个常见问题，因为它会将有效的条件判断转变为一个无效的语句，产生错误信息。始终记住，零字符的带引号短语与空白短语是不同的，这一点是非常有益的。如果你想要求大写字母，同时允许空格、逗号和句号，只需将➊处的替换模式更改为这里显示的代码：

```
sed 's/[^[:upper:] ,.]//g'
```

你也可以使用以下简单的测试来验证电话号码输入（允许整数值、空格、括号和破折号，但不允许前导空格或连续多个空格）：

```
sed 's/[^- [:digit:]\(\)]//g'
```

但如果你想将输入限制为整数值，你必须小心一个陷阱。例如，你可能会想尝试这样做：

```
sed 's/[^[:digit:]]//g'
```

这段代码适用于正数，但如果你想允许负数输入呢？如果你只是将负号添加到有效字符集里，`-3-4` 就会变成有效输入，尽管它显然不是一个合法的整数。脚本 #5 在第 23 页中讨论了如何处理负数。

### #3 规范化日期格式

Shell 脚本开发中的一个问题是数据格式的不一致性；将它们规范化可能从有点棘手到非常困难。日期格式是最难处理的，因为日期可以有很多不同的表示方式。即使你提示输入一个特定的格式，比如月-日-年，你也很可能会得到不一致的输入：例如数字表示月份而不是月份名称，月份名称的缩写，甚至是全部大写的月份名称。由于这个原因，一个规范化日期的函数，尽管它本身很基础，但将为后续脚本工作提供非常有用的构建块，特别是脚本 #7 中在第 29 页展示的。

#### *代码*

清单 1-5 中的脚本规范化符合相对简单条件集的日期格式：月份必须以名称或 1 到 12 之间的数字表示，年份必须以四位数字表示。规范化后的日期包括月份名称（以三字母缩写表示），接着是日期，再接着是四位数字的年份。

```
   #!/bin/bash
   # normdate--Normalizes month field in date specification to three letters,
   #   first letter capitalized. A helper function for Script #7, valid-date.
   #   Exits with 0 if no error.

   monthNumToName()
   {
     # Sets the 'month' variable to the appropriate value.
     case $1 in
       1 ) month="Jan"    ;;  2 ) month="Feb"    ;;
       3 ) month="Mar"    ;;  4 ) month="Apr"    ;;
       5 ) month="May"    ;;  6 ) month="Jun"    ;;
       7 ) month="Jul"    ;;  8 ) month="Aug"    ;;
       9 ) month="Sep"    ;;  10) month="Oct"    ;;
       11) month="Nov"    ;;  12) month="Dec"    ;;
       * ) echo "$0: Unknown month value $1" >&2
           exit 1
     esac
     return 0
   }

   # BEGIN MAIN SCRIPT--DELETE OR COMMENT OUT EVERYTHING BELOW THIS LINE IF
   #   YOU WANT TO INCLUDE THIS IN OTHER SCRIPTS.
   # =================
   # Input validation
   if [ $# -ne 3 ] ; then
     echo "Usage: $0 month day year" >&2
     echo "Formats are August 3 1962 and 8 3 1962" >&2
     exit 1
   fi
   if [ $3 -le 99 ] ; then
     echo "$0: expected 4-digit year value." >&2
     exit 1
   fi

   # Is the month input format a number?
➊ if [ -z $(echo $1|sed 's/[[:digit:]]//g') ]; then
     monthNumToName $1
   else
   # Normalize to first 3 letters, first upper- and then lowercase.
➋   month="$(echo $1|cut -c1|tr '[:lower:]' '[:upper:]')"
➌   month="$month$(echo $1|cut -c2-3 | tr '[:upper:]' '[:lower:]')"
   fi

   echo $month $2 $3

   exit 0
```

*清单 1-5：`*normdate*` Shell 脚本*

#### *它是如何工作的*

请注意脚本中的第三个条件判断，位于➊。它会从第一个输入字段中剥离所有数字，然后使用`-z`测试检查结果是否为空。如果结果为空，说明输入仅包含数字，因此可以直接通过`monthNumToName`映射为一个月份名称，并验证该数字是否代表有效的月份。否则，我们假设第一个输入是一个月份字符串，并使用复杂的`cut`和`tr`管道结合两个子壳调用（即被`$(`和`)`括起来的命令序列，在这种情况下，命令会被调用并用其输出替代）对其进行规范化。

第一个子壳序列位于➋，它提取输入的第一个字符并使用`tr`将其转换为大写（尽管`echo $1|cut -c1`序列也可以写成`${1%${1#?}}`，如之前在 POSIX 中所见）。第二个序列位于➌，它提取第二和第三个字符，并强制将其转换为小写，最终得到一个大写的三字母缩写形式的`month`。注意，这种字符串操作方法并不会检查输入是否实际是一个有效的月份，与传入数字的月份不同。

#### *运行脚本*

为了确保未来涉及`normdate`功能的脚本具有最大灵活性，本脚本设计为接受命令行输入的三个字段，如清单 1-6 所示。如果你只打算交互式使用此脚本，应该提示用户输入这三个字段，但这会使得从其他脚本调用`normdate`变得更加困难。

#### *结果*

```
$ normdate 8 3 62
normdate: expected 4-digit year value.
$ normdate 8 3 1962
Aug 3 1962
$ normdate AUGUST 03 1962
Aug 03 1962
```

*清单 1-6：测试`*normdate*`脚本*

请注意，这个脚本只规范化月份表示方式；日期格式（例如带前导零的日期）和年份保持不变。

#### *破解脚本*

在你为这个脚本能添加的众多扩展感到兴奋之前，先查看一下脚本 #7，它使用`normdate`来验证输入的日期，具体内容见第 29 页。

然而，你可以做一个修改，允许脚本接受 MM/DD/YYYY 或 MM-DD-YYYY 格式的日期，方法是将以下代码添加到第一个条件判断之前。

```
if [ $# -eq 1 ] ; then # To compensate for / or - formats
  set -- $(echo $1 | sed 's/[\/\-]/ /g')
fi
```

通过这个修改，你可以输入并规范化以下常见格式：

```
$ normdate 6-10-2000
Jun 10 2000
$ normdate March-11-1911
Mar 11 1911
$ normdate 8/3/1962
Aug 3 1962
```

如果仔细阅读代码，你会意识到，通过采用更复杂的方法验证指定日期中的年份，脚本将会得到改进，更不用说考虑到各种国际日期格式了。这些作为练习留给你去探索！

### #4 以吸引人的方式展示大数字

程序员常犯的一个错误是，在将计算结果展示给用户之前，没有先对其进行格式化。用户很难判断`43245435`是否属于百万级别，除非他们从右到左数，并在每三个数字处 mentally 插入一个逗号。清单 1-7 中的脚本会很好地格式化你的数字。

#### *代码*

```
   #!/bin/bash
   # nicenumber--Given a number, shows it in comma-separated form. Expects DD
   #   (decimal point delimiter) and TD (thousands delimiter) to be instantiated.
   #   Instantiates nicenum or, if a second arg is specified, the output is
   #   echoed to stdout.

   nicenumber()
   {
     # Note that we assume that '.' is the decimal separator in the INPUT value
     #   to this script. The decimal separator in the output value is '.' unless
     #   specified by the user with the -d flag.

➊   integer=$(echo $1 | cut -d. -f1)        # Left of the decimal
➋   decimal=$(echo $1 | cut -d. -f2)        # Right of the decimal
     # Check if number has more than the integer part.
     if [ "$decimal" != "$1" ]; then
       # There's a fractional part, so let's include it.
       result="${DD:= '.'}$decimal"
     fi

     thousands=$integer

➌   while [ $thousands -gt 999 ]; do
➍     remainder=$(($thousands % 1000))    # Three least significant digits

       # We need 'remainder' to be three digits. Do we need to add zeros?
       while [ ${#remainder} -lt 3 ] ; do  # Force leading zeros
         remainder="0$remainder"
       done

➎     result="${TD:=","}${remainder}${result}"    # Builds right to left
➏     thousands=$(($thousands / 1000))    # To left of remainder, if any
     done

     nicenum="${thousands}${result}"
     if [ ! -z $2 ] ; then
       echo $nicenum
     fi
   }

   DD="."  # Decimal point delimiter, to separate whole and fractional values
   TD=","  # Thousands delimiter, to separate every three digits

   # BEGIN MAIN SCRIPT
   # =================

➐ while getopts "d:t:" opt; do
     case $opt in
       d ) DD="$OPTARG"   ;;
       t ) TD="$OPTARG"   ;;
     esac
   done
   shift $(($OPTIND - 1))

   # Input validation
   if [ $# -eq 0 ] ; then
     echo "Usage: $(basename $0) [-d c] [-t c] number"
     echo "  -d specifies the decimal point delimiter"
     echo "  -t specifies the thousands delimiter"
     exit 0
   fi

➑ nicenumber $1 1    # Second arg forces nicenumber to 'echo' output.

   exit 0
```

*清单 1-7: `*nicenumber*` 脚本将长数字格式化，使其更易于阅读。*

#### *工作原理*

这个脚本的核心是`nicenumber()`函数中的`while`循环 ➌，它通过迭代不断从存储在变量`thousands`中的数值中移除后三位，并将这些数字附加到正在构建的漂亮数字版本 ➎。然后，循环会减少存储在`thousands`中的数字 ➏，如果需要，再次将其输入循环。`nicenumber()`函数完成后，主脚本逻辑开始。首先，它解析传递给脚本的任何选项，使用`getopts` ➐，然后最后调用`nicenumber()`函数 ➑，并将用户指定的最后一个参数传递给它。

#### *运行脚本*

要运行这个脚本，只需指定一个非常大的数值。脚本会根据需要添加小数点和分隔符，使用默认值或通过标志指定的字符。

结果可以纳入输出消息中，如下所示：

```
echo "Do you really want to pay \$$(nicenumber $price)?"
```

#### *结果*

`nicenumber`脚本易于使用，但也可以接受一些高级选项。清单 1-8 演示了使用脚本格式化一些数字。

```
$ nicenumber 5894625
5,894,625
$ nicenumber 589462532.433
589,462,532.433
$ nicenumber -d, -t. 589462532.433
589.462.532,433
```

*清单 1-8: 测试 `*nicenumber*` 脚本*

#### *修改脚本*

不同国家使用不同的字符作为千位和小数点分隔符，因此我们可以为这个脚本添加灵活的调用标志。例如，德国人和意大利人使用`-d "."`和`-t ","`，法国人使用`-d ","`和`-t " "`，而瑞士有四种官方语言，他们使用`-d "."`和`-t "'"`。这是一个很好的例子，说明灵活性优于硬编码，使得该工具对尽可能广泛的用户群体都很有用。

另一方面，我们确实硬编码了 `"."` 作为输入值的小数分隔符，因此如果你预计会使用不同的分隔符来处理带小数的输入值，可以修改在 ➊ 和 ➋ 处调用的 `cut` 命令，这里目前指定了 `"."` 作为小数分隔符。

以下代码展示了一种解决方案：

```
integer=$(echo $1 | cut "-d$DD" -f1)         # Left of the decimal
decimal=$(echo $1 | cut "-d$DD" -f2)         # Right of the decimal
```

这段代码有效，除非输入中的小数分隔符与输出中指定的分隔符不同，在这种情况下，脚本会静默中断。一个更复杂的解决方案是在这两行之前加入一个测试，确保输入的小数分隔符与用户请求的相同。我们可以通过使用脚本 #2 中展示的相同技巧来实现这一测试，如第 15 页所示：将所有数字去掉，看看剩下什么，就像下面的代码一样。

```
separator="$(echo $1 | sed 's/[[:digit:]]//g')"
if [ ! -z "$separator" -a "$separator" != "$DD" ] ; then
  echo "$0: Unknown decimal separator $separator encountered." >&2
  exit 1
fi
```

### #5 验证整数输入

正如你在脚本 #2 中看到的那样，验证整数输入看起来很简单，直到你希望确保负值也能被接受。问题在于，每个数字值只能有一个负号，而且负号必须出现在值的最前面。列表 1-9 中的验证例程确保负数格式正确，并且更广泛地，它可以检查值是否在用户指定的范围内。

#### *代码*

```
   #!/bin/bash
   # validint--Validates integer input, allowing negative integers too

   validint()
   {
     # Validate first field and test that value against min value $2 and/or
     #   max value $3 if they are supplied. If the value isn't within range
     #   or it's not composed of just digits, fail.

     number="$1";      min="$2";      max="$3"

➊   if [ -z $number ] ; then
       echo "You didn't enter anything. Please enter a number." >&2
       return 1
     fi

     # Is the first character a '-' sign?
➋   if [ "${number%${number#?}}" = "-" ] ; then
       testvalue="${number#?}" # Grab all but the first character to test.
     else
       testvalue="$number"
     fi

     # Create a version of the number that has no digits for testing.
➌   nodigits="$(echo $testvalue | sed 's/[[:digit:]]//g')"

     # Check for nondigit characters.
     if [ ! -z $nodigits ] ; then
       echo "Invalid number format! Only digits, no commas, spaces, etc." >&2
       return 1
     fi

➍   if [ ! -z $min ] ; then
       # Is the input less than the minimum value?
       if [ "$number" -lt "$min" ] ; then
         echo "Your value is too small: smallest acceptable value is $min." >&2
         return 1
       fi
     fi
     if [ ! -z $max ] ; then
       # Is the input greater than the maximum value?
       if [ "$number" -gt "$max" ] ; then
         echo "Your value is too big: largest acceptable value is $max." >&2
         return 1
       fi
     fi
     return 0
   }
```

*列表 1-9：* `*validint*` *脚本*

#### *原理解释*

验证整数是相对直接的，因为值要么只是数字（0 到 9）的序列，要么可能带有一个只能出现一次的负号。如果调用`validint()`函数并传入最小值或最大值，或者两者，它还会检查这些值，以确保输入的值在范围内。

函数在➊处确保用户没有完全跳过输入（这里另一个关键点是需要预见到可能出现空字符串的情况，使用引号来确保不会生成错误信息）。接着，在➋处，它检查负号，并在➌处创建一个去掉所有数字的输入值版本。如果该值的长度不为零，则表示存在问题，测试失败。

如果值有效，用户输入的数字会与最小值和最大值进行比较 ➍。最后，函数返回 1 表示出错，返回 0 表示成功。

#### *运行脚本*

整个脚本是一个函数，可以复制到其他 Shell 脚本中或作为库文件包含。要将其转为命令，只需将列表 1-10 中的代码附加到脚本的底部。

```
# Input validation
if validint "$1" "$2" "$3" ; then
  echo "Input is a valid integer within your constraints."
fi
```

*列表 1-10：为* `*validint*` *添加支持，以使其作为命令运行*

#### *结果*

将列表 1-10 放入脚本中后，你应该能够像列表 1-11 所示那样使用它：

```
$ validint 1234.3
Invalid number format! Only digits, no commas, spaces, etc.
$ validint 103 1 100
Your value is too big: largest acceptable value is 100.
$ validint -17 0 25
Your value is too small: smallest acceptable value is 0.
$ validint -17 -20 25
Input is a valid integer within your constraints.
```

*列表 1-11：测试* `*validint*` *脚本*

#### *破解脚本*

请注意，在➋处的测试检查数字的第一个字符是否为负号：

```
if [ "${number%${number#?}}" = "-" ] ; then
```

如果第一个字符是负号，`testvalue`将被赋值为整数值的数字部分。然后，这个非负值会去掉数字并进一步测试。

你可能会想使用逻辑与（`-a`）来连接表达式并缩减一些嵌套的`if`语句。例如，看起来这段代码应该是有效的：

```
if [ ! -z $min -a "$number" -lt "$min" ] ; then
  echo "Your value is too small: smallest acceptable value is $min." >&2
  exit 1
fi
```

然而，实际并不是这样，因为即使一个 AND 表达式的第一个条件为假，你也不能保证第二个条件不会被测试（这与大多数其他编程语言不同）。这意味着，如果你尝试这样做，你可能会遇到各种无效或意外的比较值所导致的错误。这本不应该是这样，但这就是 shell 脚本的特性。

### #6 验证浮点输入

初看之下，验证浮点（或“实数”）值的过程在 shell 脚本的范围和能力内可能看起来令人畏惧，但请考虑到，浮点数仅仅是两个整数通过小数点分隔开。结合这个洞察力，再加上能够内联引用不同脚本（`validint`）的能力，你会发现浮点数验证测试竟然可以出奇的简短。清单 1-12 中的脚本假设它是从与`validint`脚本相同的目录下运行的。

#### *代码*

```
   #!/bin/bash

   # validfloat--Tests whether a number is a valid floating-point value.
   #   Note that this script cannot accept scientific (1.304e5) notation.

   # To test whether an entered value is a valid floating-point number,
   #   we need to split the value into two parts: the integer portion
   #   and the fractional portion. We test the first part to see whether
   #   it's a valid integer, and then we test whether the second part is a
   #   valid >=0 integer. So -30.5 evaluates as valid, but -30.-8 doesn't.

   # To include another shell script as part of this one, use the "." source
   #   notation. Easy enough.

   . validint

   validfloat()
   {
     fvalue="$1"

     # Check whether the input number has a decimal point.
➊   if [ ! -z $(echo $fvalue | sed 's/[^.]//g') ] ; then

       # Extract the part before the decimal point.
➋     decimalPart="$(echo $fvalue | cut -d. -f1)"

       # Extract the digits after the decimal point.
➌     fractionalPart="${fvalue#*\.}"

       # Start by testing the decimal part, which is everything
       #   to the left of the decimal point.

➍     if [ ! -z $decimalPart ] ; then
         # "!" reverses test logic, so the following is
         #   "if NOT a valid integer"
         if ! validint "$decimalPart" "" "" ; then
           return 1
         fi
       fi

       # Now let's test the fractional value.

       # To start, you can't have a negative sign after the decimal point
       #   like 33.-11, so let's test for the '-' sign in the decimal.
➎     if [ "${fractionalPart%${fractionalPart#?}}" = "-" ] ; then
         echo "Invalid floating-point number: '-' not allowed \
           after decimal point." >&2
         return 1
       fi
       if [ "$fractionalPart" != "" ] ; then
         # If the fractional part is NOT a valid integer...
         if ! validint "$fractionalPart" "0" "" ; then
           return 1
         fi
       fi

   else
     # If the entire value is just "-", that's not good either.
➏   if [ "$fvalue" = "-" ] ; then
       echo "Invalid floating-point format." >&2
       return 1
     fi

     # Finally, check that the remaining digits are actually
     #   valid as integers.
     if ! validint "$fvalue" "" "" ; then
       return 1
     fi
   fi

     return 0
   }
```

*清单 1-12：* `*validfloat*` *脚本*

#### *工作原理*

脚本首先检查输入值是否包含小数点 ➊。如果没有，它就不是浮点数。接下来，小数 ➋ 和分数 ➌ 部分的值会被切割出来进行分析。然后在 ➍，脚本检查小数部分（小数点*左侧*的数字）是否是一个有效的整数。接下来的检查较为复杂，因为我们需要在 ➎ 检查是否没有额外的负号（避免出现像 17\. –30 这样的奇怪情况），然后再次确保分数部分（小数点*右侧*的数字）是一个有效的整数。

最后的检查在 ➏，是检查用户是否仅指定了负号和小数点（这会很奇怪，必须承认）。

一切正常吗？如果是，那么脚本返回 0，表示用户输入了一个有效的浮点数。

#### *运行脚本*

如果调用该函数时没有产生错误信息，返回代码为 0，并且指定的数字是一个有效的浮点值。你可以通过在代码末尾添加以下几行来测试这个脚本：

```
if validfloat $1 ; then
  echo "$1 is a valid floating-point value."
fi

exit 0
```

如果`validint`产生了错误，确保它作为一个独立的函数在`PATH`中可以被脚本访问，或者直接将它复制粘贴到脚本文件中。

#### *结果*

`validfloat` shell 脚本仅接受一个参数来进行验证。清单 1-13 使用`validfloat`脚本验证几个输入。

```
$ validfloat 1234.56
1234.56 is a valid floating-point value.
$ validfloat -1234.56
-1234.56 is a valid floating-point value.
$ validfloat -.75
-.75 is a valid floating-point value.
$ validfloat -11.-12
Invalid floating-point number: '-' not allowed after decimal point.
$ validfloat 1.0344e22
Invalid number format! Only digits, no commas, spaces, etc.
```

*清单 1-13：测试* `*validfloat*` *脚本*

如果你在此时看到额外的输出，可能是因为你之前为了测试 `validint` 添加了一些行，但在切换到这个脚本时忘记删除它们。只需返回到脚本 #5 的第 23 页，确保那些让你以独立方式运行函数的最后几行已经被注释掉或删除。

#### *破解脚本*

一个很酷的附加技巧是扩展这个函数以允许科学记数法，如最后一个例子所示。这并不难。你可以检测是否存在 `'e'` 或 `'E'`，然后将结果分为三个部分：小数部分（始终是一个数字），分数部分和 10 的幂。然后你只需要确保每部分都是一个 `validint`。

如果你不想要求小数点前有前导零，你也可以修改列表 1-12 中的条件测试。在处理奇怪格式时要小心。

### #7 验证日期格式

最具挑战性的验证任务之一，但对于处理日期的 Shell 脚本至关重要的是，确保指定的日期在日历上实际存在。如果我们忽略闰年，这项任务不算太难，因为每年的日历是恒定的。在这种情况下，我们只需要一个包含每个月最大天数的表格，来与指定的日期进行比较。为了考虑闰年，你需要向脚本中添加一些额外的逻辑，这也使得问题变得更加复杂。

判断某一年是否为闰年的一组规则如下：

• 不能被 4 整除的年份*不是*闰年。

• 能被 4 和 400 整除的年份*是*闰年。

• 能被 4 整除，但不能被 400 整除的年份，以及能被 100 整除的年份*不是*闰年。

• 所有其他能被 4 整除的年份*是*闰年。

当你浏览源代码列表 1-14 时，注意这个脚本如何利用 `normdate` 来确保在继续之前日期格式一致。

#### *代码*

```
   #!/bin/bash
   # valid-date--Validates a date, taking into account leap year rules

   normdate="whatever you called the normdate.sh script"

   exceedsDaysInMonth()
   {
     # Given a month name and day number in that month, this function will
     #   return 0 if the specified day value is less than or equal to the
     #   max days in the month; 1 otherwise.

➊   case $(echo $1|tr '[:upper:]' '[:lower:]') in
       jan* ) days=31    ;;  feb* ) days=28    ;;
       mar* ) days=31    ;;  apr* ) days=30    ;;
       may* ) days=31    ;;  jun* ) days=30    ;;
 jul* ) days=31    ;;  aug* ) days=31    ;;
       sep* ) days=30    ;;  oct* ) days=31    ;;
       nov* ) days=30    ;;  dec* ) days=31    ;;
          * ) echo "$0: Unknown month name $1" >&2
              exit 1
     esac
     if [ $2 -lt 1 -o $2 -gt $days ] ; then
       return 1
     else
       return 0   # The day number is valid.
     fi
   }

   isLeapYear()
   {
     # This function returns 0 if the specified year is a leap year;
     #   1 otherwise.
     # The formula for checking whether a year is a leap year is:
     #   1\. Years not divisible by 4 are not leap years.
     #   2\. Years divisible by 4 and by 400 are leap years.
     #   3\. Years divisible by 4, not divisible by 400, but divisible
     #      by 100 are not leap years.
     #   4\. All other years divisible by 4 are leap years.

     year=$1
➋   if [ "$((year % 4))" -ne 0 ] ; then
       return 1 # Nope, not a leap year.
     elif [ "$((year % 400))" -eq 0 ] ; then
       return 0 # Yes, it's a leap year.
     elif [ "$((year % 100))" -eq 0 ] ; then
       return 1
     else
       return 0
     fi
   }

   # BEGIN MAIN SCRIPT
   # =================

   if [ $# -ne 3 ] ; then
     echo "Usage: $0 month day year" >&2
     echo "Typical input formats are August 3 1962 and 8 3 1962" >&2
     exit 1
   fi

   # Normalize date and store the return value to check for errors.

➌ newdate="$($normdate "$@")"

   if [ $? -eq 1 ] ; then
     exit 1        # Error condition already reported by normdate
   fi

   # Split the normalized date format, where
   #   first word = month, second word = day, third word = year.
   month="$(echo $newdate | cut -d\  -f1)"
   day="$(echo $newdate | cut -d\  -f2)"
   year="$(echo $newdate | cut -d\  -f3)"

   # Now that we have a normalized date, let's check whether the
   #   day value is legal and valid (e.g., not Jan 36).

   if ! exceedsDaysInMonth $month "$2" ; then
     if [ "$month" = "Feb" -a "$2" -eq "29" ] ; then
       if ! isLeapYear $3 ; then
➍       echo "$0: $3 is not a leap year, so Feb doesn't have 29 days." >&2
         exit 1
       fi
     else
       echo "$0: bad day value: $month doesn't have $2 days." >&2
       exit 1
     fi
   fi

   echo "Valid date: $newdate"

   exit 0
```

*列表 1-14：* `*valid-date*` *脚本*

#### *工作原理*

这是一个有趣的脚本编写，因为它需要进行大量的智能条件测试，涉及到月份天数、闰年等内容。逻辑不仅仅是指定月份 = 1–12，日期 = 1–31 等等。为了组织性，使用了特定的函数来简化编写和理解过程。

首先，`exceedsDaysInMonth()` 解析用户的月份指定，分析非常宽松（这意味着月份名称 `JANUAR` 也能正常工作）。这一过程在 ➊ 使用一个 `case` 语句完成，该语句将其参数转换为小写字母，然后进行比较以确定该月的天数。这种方法可行，但假设二月总是 28 天。

为了解决闰年问题，第二个函数 `isLeapYear()` 使用一些基本的数学测试来确定指定的年份是否有 2 月 29 日 ➋。

在主脚本中，输入被传递到之前展示的脚本 `normdate`，以规范化输入格式 ➌，然后将其拆分为三个字段 `$month`、`$day` 和 `$year`。接着，调用 `exceedsDaysInMonth` 函数，检查指定月份的日期是否无效（例如 9 月 31 日），如果用户指定了 2 月并且日期为 29 日，则会触发特殊条件。通过 `isLeapYear` 来测试该年是否为闰年，在 ➍ 处生成适当的错误。如果用户输入通过了所有这些测试，那么它就是一个有效日期！

#### *运行脚本*

要运行脚本（如清单 1-15 所示），在命令行中输入日期，格式为月-日-年。月份可以是三字母缩写、完整单词或数字值；年份必须是四位数字。

#### *结果*

```
$ valid-date august 3 1960
Valid date: Aug 3 1960
$ valid-date 9 31 2001
valid-date: bad day value: Sep doesn't have 31 days.
$ valid-date feb 29 2004
Valid date: Feb 29 2004
$ valid-date feb 29 2014
valid-date: 2014 is not a leap year, so Feb doesn't have 29 days.
```

*清单 1-15：测试* `*valid-date*` *脚本*

#### *破解脚本*

采用类似的方法，脚本可以验证时间规格，使用 24 小时制时钟或午前/午后（AM/PM）后缀。将值按照冒号分隔，确保分钟和秒数（如果指定）在 0 到 60 之间，然后检查第一个值，如果允许 AM/PM，则应在 0 到 12 之间；如果使用 24 小时制，则应在 0 到 24 之间。幸运的是，虽然有闰秒和其他微小的时间变化来保持日历平衡，但我们在日常使用中可以安全忽略这些，因此不需要担心实现如此复杂的时间计算。

如果你在 Unix 或 GNU/Linux 实现中能够访问到 GNU `date`，测试闰年的方法会有所不同。通过指定以下命令并查看得到的结果来进行测试：

```
$ date -d 12/31/1996 +%j
```

如果你使用的是更新、更好的 `date` 版本，你会看到 `366`。在较旧的版本中，它会抱怨输入格式。现在想想从更新版 `date` 命令得到的结果，看看你是否能想出一个两行的函数来测试某个年份是否为闰年！

最后，这个脚本对于月份名称非常宽容；`febmama` 完全可以正常工作，因为 ➊ 处的 `case` 语句仅检查指定单词的前三个字母。如果你愿意，可以通过测试常见的缩写（如 `feb`）以及完全拼写的月份名称（如 `february`），甚至是常见的拼写错误（如 `febuary`），来清理和改进这一点。如果你有动机，这些都很容易实现！

### #8 绕过不良的 echo 实现

如在 “什么是 POSIX？”（第 10 页）中提到的，虽然大多数现代 Unix 和 GNU/Linux 实现都具有 `echo` 命令版本，并且知道 `-n` 标志应抑制输出中的尾随换行符，但并不是所有实现都如此。一些实现使用 `\c` 作为特殊嵌入字符来防止默认行为，而其他一些则坚持在输出中始终包括尾随换行符。

判断你的 `echo` 是否正确实现很简单：只需输入这些命令并查看发生了什么：

```
$ echo -n "The rain in Spain"; echo " falls mainly on the Plain"
```

如果你的 `echo` 支持 `-n` 标志，你会看到类似这样的输出：

```
The rain in Spain falls mainly on the Plain
```

如果没有，你会看到类似这样的输出：

```
-n The rain in Spain
falls mainly on the Plain
```

确保脚本输出按照预期呈现给用户非常重要，随着我们的脚本变得越来越互动，这一点将变得尤为重要。为此，我们将编写一个 `echo` 的替代版本，称为 `echon`，它将始终抑制尾部的换行符。这样，每次我们需要 `echo -n` 功能时，就可以可靠地调用它。

#### *代码*

解决这个奇怪的 `echo` 问题的方式和本书中的页数一样多。我们最喜欢的方式之一非常简洁；它只是简单地通过 `awk printf` 命令过滤输入，正如 示例 1-16 所示。

```
echon()
{
  echo "$*" | awk '{ printf "%s", $0 }'
}
```

*示例 1-16：一个简单的* `*echo*` *替代方案，使用* `*awk printf*` *命令*

然而，你可能希望避免调用 `awk` 命令时产生的开销。如果你有一个用户级的 `printf` 命令，你可以写一个 `echon` 函数，使用它来过滤输入，就像在 示例 1-17 中所示。

```
echon()
{
  printf "%s" "$*"
}
```

*示例 1-17：使用简单的* `*printf*` *命令的* `*echo*` *替代方案*

如果你没有 `printf`，并且不想调用 `awk`，那么可以使用 `tr` 命令去除任何最终的换行符，就像在 示例 1-18 中所示。

```
echon()
{
  echo "$*" | tr -d '\n'
}
```

*示例 1-18：使用* `*tr*` *工具的一个简单* `*echo*` *替代方案*

这种方法简单高效，并且应该具有很好的可移植性。

#### *运行脚本*

只需将脚本文件添加到你的 `PATH`，你就可以用 `echon` 替代任何 `echo -n` 调用，确保每次输出后，用户的光标都停留在行尾。

#### *结果*

`echon` shell 脚本通过接收一个参数并打印它，然后读取一些用户输入来演示 `echon` 功能。示例 1-19 展示了该测试脚本的使用。

```
$ echon "Enter coordinates for satellite acquisition: "
Enter coordinates for satellite acquisition: 12,34
```

*示例 1-19：测试* `*echon*` *命令*

#### *修改脚本*

我们不会撒谎。事实是，某些 shell 的 `echo` 语句知道 `-n` 标志，而其他 shell 则期望使用 `\c` 作为结束符，还有一些 shell 看似根本没有避免添加换行符的能力，这对于脚本编写者来说是个巨大的痛苦。为了解决这种不一致，你可以创建一个函数，自动测试 `echo` 的输出，以确定当前使用的是哪种情况，然后相应地修改调用。例如，你可以写类似于 `echo -n hi | wc -c` 的命令，然后测试结果是两个字符（`hi`）、三个字符（`hi` 加上换行符）、四个字符（`-n hi`），还是五个字符（`-n hi` 加上换行符）。

### #9 一个任意精度的浮点计算器

在脚本编写中最常用的序列之一是`$(( ))`，它允许你使用各种基本的数学函数进行计算。这个序列非常有用，能够简化常见操作，例如递增计数器变量。它支持加法、减法、除法、余数（或取模）和乘法操作，但不支持分数或小数值。因此，以下命令返回 0，而不是 0.5：

```
echo $(( 1 / 2 ))
```

因此，当计算需要更高精度的值时，你会面临一定的挑战。目前，命令行上并没有很多优秀的计算器程序。唯一的例外是`bc`，这个少数 Unix 用户了解的奇特程序。`bc`自称是一个任意精度的计算器，回溯到 Unix 的黎明时期，带有神秘的错误信息，完全没有提示，并假设你如果使用它，已经知道该怎么做。不过，这没关系。我们可以编写一个包装器，使`bc`更加用户友好，正如清单 1-20 所示。

#### *代码*

```
   #!/bin/bash

   # scriptbc--Wrapper for 'bc' that returns the result of a calculation

➊ if ["$1" = "-p" ] ; then
     precision=$2
     shift 2
   else
➋   precision=2           # Default
   fi

➌ bc -q -l << EOF
     scale=$precision
     $*
     quit
   EOF

   exit 0
```

*清单 1-20: The* `*scriptbc*` *脚本*

#### *它是如何工作的*

➌处的`<<`表示法允许你从脚本中包含内容，并将其当作直接输入流的一部分处理，在本例中，它为将命令传递给`bc`程序提供了一个简单的机制。这被称为编写*here 文档*。在这种表示法中，紧跟在`<<`序列之后的内容将会匹配（独立成行），用以表示输入流的结束。在清单 1-20 中，它是`EOF`。

这个脚本还展示了如何使用参数来使命令更加灵活。在这里，如果脚本调用时使用`-p`标志 ➊，它允许你指定输出数字的精度。如果未指定精度，程序默认`scale=2` ➋。

在使用`bc`时，了解`length`和`scale`之间的区别至关重要。就`bc`而言，`length`指的是数字中的总位数，而`scale`则是小数点后面的数字位数。因此，10.25 的`length`为 4，`scale`为 2，而 3.14159 的`length`为 6，`scale`为 5。

默认情况下，`bc`的`length`值是可变的，但由于其`scale`为零，未做任何修改的`bc`与`$(( ))`表示法的功能完全相同。幸运的是，如果你为`bc`添加`scale`设置，你会发现它的潜力巨大，正如这个示例所示，计算了 1962 年到 2002 年（不包括闰日）之间经过了多少周：

```
$ bc
bc 1.06.95
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006 Free Software Foundation,
Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type 'warranty'.
scale=10
(2002-1962)*365
14600
14600/7
2085.7142857142
quit
```

为了允许从命令行访问 `bc` 功能，包装脚本必须屏蔽开头的版权信息（如果有的话），尽管大多数 `bc` 实现已经在其输入不是终端（`stdin`）时屏蔽了该信息。包装脚本还会将 `scale` 设置为一个合理的值，将实际的表达式传递给 `bc` 程序，然后通过 `quit` 命令退出。

#### *运行脚本*

要运行这个脚本，将一个数学表达式作为参数传递给程序，如清单 1-21 所示。

#### *结果*

```
$ scriptbc 14600/7
2085.71
$ scriptbc -p 10 14600/7
2085.7142857142
```

*清单 1-21：测试 `*scriptbc*` 脚本*

### #10 锁定文件

任何读取或追加到共享文件的脚本，比如日志文件，都需要一种可靠的方式来锁定文件，以防其他脚本实例在数据使用完之前不小心覆盖数据。一种常见的做法是为每个正在使用的文件创建一个单独的 *锁文件*。锁文件的存在作为一个 *信号量*，表示文件正在被另一个脚本使用，无法访问。请求的脚本会反复等待并重试，直到信号量锁文件被移除，表明文件可以自由编辑。

然而，锁文件是棘手的，因为许多看似万无一失的解决方案实际上并不起作用。例如，以下代码是解决这个问题的典型方法：

```
while [ -f $lockfile ] ; do
  sleep 1
done
touch $lockfile
```

看起来好像可以工作，对吧？这段代码会一直循环，直到锁文件不存在，然后创建它，以确保你拥有锁并且可以安全地修改基础文件。如果另一个具有相同循环的脚本看到你的锁文件，它也会一直循环，直到锁文件消失。然而，实际上这并不起作用。试想一下，如果在 `while` 循环退出之后，但在执行 `touch` 命令之前，这个脚本被交换出去，并重新排入处理器队列，给另一个脚本运行的机会，会发生什么。

如果你不确定我们在说什么，记住，虽然你的计算机似乎一次只做一件事，但实际上它是在同时运行多个程序，通过在每个程序之间切换，每次只做一点点。这里的问题是，在脚本完成检查锁文件和创建自己锁文件之间的这段时间，系统可能会切换到另一个脚本，而这个脚本会照常检查锁文件，发现没有锁文件并创建自己的锁文件。然后该脚本可能会被切换出去，而你的脚本可能会恢复执行 `touch` 命令。结果是两个脚本都认为它们独占了锁文件，而这正是我们想要避免的情况。

幸运的是，`procmail` 邮件过滤程序的作者 Stephen van den Berg 和 Philip Guenther 也创建了一个命令行工具 `lockfile`，该工具可以让你在 shell 脚本中安全、可靠地操作锁文件。

许多 Unix 发行版，包括 GNU/Linux 和 OS X，都已预安装`lockfile`。你可以通过输入`man 1 lockfile`来检查你的系统是否有`lockfile`。如果显示了手册页，那就表示你的运气不错！清单 1-22 中的脚本假设你已经安装了`lockfile`命令，后续脚本需要脚本 #10 中可靠的锁机制来运行，因此请确保你的系统上已安装`lockfile`命令。

#### *代码*

```
   #!/bin/bash

   # filelock--A flexible file-locking mechanism

   retries="10"            # Default number of retries
   action="lock"           # Default action
   nullcmd="'which true'"  # Null command for lockfile

➊ while getopts "lur:" opt; do
     case $opt in
       l ) action="lock"      ;;
       u ) action="unlock"    ;;
       r ) retries="$OPTARG"  ;;
     esac
   done
➋ shift $(($OPTIND - 1))

   if [ $# -eq 0 ] ; then # Output a multiline error message to stdout.
     cat << EOF >&2
   Usage: $0 [-l|-u] [-r retries] LOCKFILE
   Where -l requests a lock (the default), -u requests an unlock, -r X
   specifies a max number of retries before it fails (default = $retries).
     EOF
     exit 1
   fi

   # Ascertain if we have the lockfile command.

➌ if [ -z "$(which lockfile | grep -v '^no ')" ] ; then
     echo "$0 failed: 'lockfile' utility not found in PATH." >&2
     exit 1
   fi
➍ if [ "$action" = "lock" ] ; then
     if ! lockfile -1 -r $retries "$1" 2> /dev/null; then
       echo "$0: Failed: Couldn't create lockfile in time." >&2
       exit 1
     fi
   else    # Action = unlock.
     if [ ! -f "$1" ] ; then
       echo "$0: Warning: lockfile $1 doesn't exist to unlock." >&2
       exit 1
     fi
     rm -f "$1"
   fi

   exit 0
```

*清单 1-22：`*filelock*`脚本*

#### *它是如何工作的*

正如一个编写良好的 Shell 脚本通常会做的那样，清单 1-22 的一半内容是解析输入变量并检查错误条件。最后，它到达了`if`语句，然后尝试实际使用系统的`lockfile`命令。如果有该命令，它会指定重试次数并调用它，如果最终失败，则生成自己的错误信息。如果你请求解锁（例如，移除现有的锁），但并没有锁定文件呢？这时会产生另一个错误。否则，`lockfile`将被移除，操作完成。

更具体地说，第一个代码块➊使用强大的`getopts`函数通过`while`循环解析所有可能的用户输入标志（`-l`，`-u`，`-r`）。这是利用`getopts`的常见方式，书中会多次出现这个模式。请注意第➋步的`shift $(($OPTIND - 1 ))`语句：`OPTIND`由`getopts`设置，它使得脚本能够不断将值向下移动（例如，`$2`变成`$1`），直到处理完带有破折号的这些值。

由于这个脚本使用了系统的`lockfile`工具，因此在调用它之前确保该工具在用户的路径中是一个良好的做法。如果路径中没有该工具，它将显示错误信息。然后，在第➍步会有一个简单的条件判断，查看我们是在锁定还是解锁，并根据情况调用`lockfile`工具。

#### *运行脚本*

虽然`lockfile`脚本不是你通常会单独使用的脚本，但你可以通过打开两个终端窗口来进行测试。要创建一个锁，只需将你想要锁定的文件名作为`filelock`的参数指定即可。要移除锁，再次运行脚本并添加`-u`标志。

#### *结果*

首先，按照清单 1-23 所示创建一个锁定的文件。

```
$ filelock /tmp/exclusive.lck
$ ls -l /tmp/exclusive.lck
-r--r--r--  1 taylor  wheel  1 Mar 21 15:35 /tmp/exclusive.lck
```

*清单 1-23：使用`*filelock*`命令创建文件锁*

当你第二次尝试锁定文件时，`filelock`会尝试默认的次数（10 次），然后失败（如清单 1-24 所示）：

```
$ filelock /tmp/exclusive.lck
filelock : Failed: Couldn't create lockfile in time.
```

*清单 1-24：`*filelock*`命令未能创建锁文件*

当第一个进程完成文件操作后，你可以按照清单 1-25 所示释放锁。

```
$ filelock -u /tmp/exclusive.lck
```

*清单 1-25：使用`*filelock*`脚本释放文件锁*

要查看`filelock`脚本如何在两个终端中工作，可以在一个窗口中运行解锁命令，而另一个窗口则持续运行，尝试建立它自己的独占锁。

#### *破解脚本*

因为该脚本依赖于锁文件的存在来证明锁仍然有效，所以如果有一个附加参数，比如锁应该有效的最长时间长度会很有用。如果`lockfile`例程超时，则可以检查被锁定文件的最后访问时间，如果被锁定的文件比该参数值还要旧，那么它就可以安全地作为多余的文件删除，或许可以带上警告信息。

这不太可能影响你，但`lockfile`不适用于网络文件系统（NFS）挂载的网络驱动器。实际上，NFS 挂载磁盘上的可靠文件锁定机制相当复杂。一个完全避开这个问题的更好策略是只在本地磁盘上创建锁文件，或者使用一个可以跨多个系统管理锁的网络感知脚本。

### #11 ANSI 颜色序列

尽管你可能没有意识到，大多数终端应用程序都支持不同风格的文本呈现。无论你是希望在脚本中将某些单词显示为粗体，还是希望将它们显示为红色配黄色背景，都是可能的。然而，使用*ANSI（美国国家标准协会）*序列来表示这些变化可能会很困难，因为它们相当不友好。为了简化它们，清单 1-26 创建了一组变量，这些变量的值表示 ANSI 代码，可以用于开关各种颜色和格式选项。

#### *代码*

```
#!/bin/bash

# ANSI color--Use these variables to make output in different colors
#   and formats. Color names that end with 'f' are foreground colors,
#   and those ending with 'b' are background colors.

initializeANSI()
{
  esc="\033"   # If this doesn't work, enter an ESC directly.

  # Foreground colors
  blackf="${esc}30m";   redf="${esc}[31m";    greenf="${esc}[32m"
  yellowf="${esc}[33m"   bluef="${esc}[34m";   purplef="${esc}[35m"
  cyanf="${esc}[36m";    whitef="${esc}[37m"

  # Background colors
  blackb="${esc}[40m";   redb="${esc}[41m";    greenb="${esc}[42m"
  yellowb="${esc}[43m"   blueb="${esc}[44m";   purpleb="${esc}[45m"
  cyanb="${esc}[46m";    whiteb="${esc}[47m"

  # Bold, italic, underline, and inverse style toggles
  boldon="${esc}[1m";    boldoff="${esc}[22m"
  italicson="${esc}[3m"; italicsoff="${esc}[23m"
  ulon="${esc}[4m";      uloff="${esc}[24m"
  invon="${esc}[7m";     invoff="${esc}[27m"

  reset="${esc}[0m"
}
```

*清单 1-26：*`*initializeANSI*`*脚本函数*

#### *它是如何工作的*

如果你习惯了 HTML，可能会对这些序列的工作方式感到困惑。在 HTML 中，你需要以相反的顺序打开和关闭修饰符，并且你必须关闭每个打开的修饰符。因此，要在一个句子中创建一个斜体部分并显示为粗体，你将使用以下 HTML 代码：

```
<b>this is in bold and <i>this is italics</i> within the bold</b>
```

在没有关闭斜体标签的情况下关闭粗体标签会引发混乱，并可能会搞乱一些网页浏览器。但对于 ANSI 颜色序列，一些修饰符实际上会覆盖前一个修饰符，并且还有一个重置序列，它会关闭所有修饰符。使用 ANSI 序列时，你必须确保在使用颜色后输出重置序列，并且对任何你打开的功能使用`off`选项。使用此脚本中的变量定义，你可以像这样重写前面的序列：

```
${boldon}this is in bold and ${italicson}this is
italics${italicsoff}within the bold${reset}
```

#### *运行脚本*

要运行此脚本，首先调用初始化函数，然后输出一些带有不同颜色和效果组合的`echo`语句：

```
initializeANSI

cat << EOF
${yellowf}This is a phrase in yellow${redb} and red${reset}
${boldon}This is bold${ulon} this is italics${reset} bye-bye
${italicson}This is italics${italicsoff} and this is not
${ulon}This is ul${uloff} and this is not
${invon}This is inv${invoff} and this is not
${yellowf}${redb}Warning I ${yellowb}${redf}Warning II${reset}
EOF
```

#### *结果*

[清单 1-27 中的结果在本书中看起来并不太惊艳，但在支持这些颜色序列的显示器上，它们肯定会引起你的注意。

```
This is a phrase in yellow and red
This is bold this is italics bye-bye
This is italics and this is not
This is ul and this is not
This is inv and this is not
Warning I Warning II
```

*清单 1-27：如果运行 清单 1-26 中的脚本，打印出的文本*

#### *破解脚本*

使用此脚本时，你可能会看到如下输出：

```
\03333m\033[41mWarning!\033[43m\033[31mWarning!\033[0m
```

如果你这么做了，问题可能出在你的终端或窗口不支持 ANSI 颜色序列，或者它不理解重要的 `esc` 变量的 `\033` 符号。要解决后者的问题，打开脚本文件，使用 `vi` 或你喜欢的终端编辑器，删除 `\033` 序列，并通过按下 `^V`（CTRL-V）键，再按下 ESC 键，这应该会显示为 `^[`。如果屏幕上显示 `esc="^[`，一切应该正常。

另一方面，如果你的终端或窗口根本不支持 ANSI 序列，你可能需要升级，以便能够为你的其他脚本添加彩色和增强的字体输出。但在放弃当前终端之前，请检查终端的偏好设置—某些终端有一个可以启用完全 ANSI 支持的设置。

### #12 构建一个 Shell 脚本库

本章中的许多脚本都是作为函数编写的，而不是独立脚本，这样它们可以轻松地集成到其他脚本中，而不会增加系统调用的开销。虽然 shell 脚本中没有像 C 语言中的 `#include` 功能，但有一个非常重要的功能叫做 *source* 文件，它起到相同的作用，允许你像包含库函数一样包含其他脚本。

为了理解这为什么很重要，我们来考虑一下另一种情况。如果你在一个 shell 中调用一个 shell 脚本，默认情况下，该脚本会在它自己的子 shell 中运行。你可以通过实验来验证这一点：

```
$ echo "test=2" >> tinyscript.sh
$ chmod +x tinyscript.sh
$ test=1
$ ./tinyscript.sh
$ echo $test
1
```

脚本*tinyscript.sh*修改了变量`test`的值，但只是在运行该脚本的子 shell 中，所以我们 shell 环境中现有的`test`变量的值没有受到影响。如果你改用点（`.`）符号来运行脚本，这样就相当于每个脚本中的命令直接输入到当前的 shell 中：

```
$ . tinyscript.sh
$ echo $test
2
```

正如你所预期的那样，如果你 source 一个包含 `exit 0` 命令的脚本，它将退出 shell 并注销窗口，因为 `source` 操作使得被 source 的脚本成为主要运行进程。如果你有一个在子 shell 中运行的脚本，它会退出，但不会影响主脚本的执行。这是一个重要的区别，也是选择使用 `.` 或 `source` 或（如我们稍后会解释的）`exec` 来 source 脚本的原因之一。`.` 符号实际上与 bash 中的 `source` 命令是相同的；我们使用 `.` 是因为它在不同的 POSIX shell 中更具可移植性。

#### *代码*

要将本章中的函数转换为可在其他脚本中使用的库，请提取所有函数以及任何需要的全局变量或数组（即跨多个函数共享的值），并将它们合并为一个大文件。如果你将此文件命名为*library.sh*，你可以使用以下测试脚本访问我们在本章中编写的所有函数，并查看它们是否正常工作，如[列表 1-28 所示。

```
   #!/bin/bash

   # Library test script

   # Start by sourcing (reading in) the library.sh file.

➊ . library.sh

   initializeANSI  # Let's set up all those ANSI escape sequences.
   # Test validint functionality.
   echon "First off, do you have echo in your path? (1=yes, 2=no) "
   read answer
   while ! validint $answer 1 2 ; do
     echon "${boldon}Try again${boldoff}. Do you have echo "
     echon "in your path? (1=yes, 2=no) "
     read answer
   done

   # Is the command that checks what's in the path working?
   if ! checkForCmdInPath "echo" ; then
     echo "Nope, can't find the echo command."
   else
     echo "The echo command is in the PATH."
   fi

   echo ""
   echon "Enter a year you think might be a leap year: "
   read year

   # Test to see if the year specified is between 1 and 9999 by
   #   using validint with a min and max value.
   while ! validint $year 1 9999 ; do
     echon "Please enter a year in the ${boldon}correct${boldoff} format: "
     read year
   done

   # Now test whether it is indeed a leap year.
   if isLeapYear $year ; then
     echo "${greenf}You're right! $year is a leap year.${reset}"
   else
     echo "${redf}Nope, that's not a leap year.${reset}"
   fi

   exit 0
```

*列表 1-28：将先前实现的函数作为单个库源代码并调用它们*

#### *工作原理*

请注意，库文件已被引入，所有函数都会被读取并包含到脚本的运行时环境中，在➊的单行代码处。

这种处理本书中多个脚本的有用方法可以根据需要反复利用。只要确保你包含的库文件可以从`PATH`中访问，这样`.`命令就能找到它。

#### *运行脚本*

要运行测试脚本，只需像运行任何其他脚本一样从命令行调用它，就像在列表 1-29 中所示。

#### *结果*

```
$ library-test
First off, do you have echo in your PATH? (1=yes, 2=no) 1
The echo command is in the PATH.

Enter a year you think might be a leap year: 432423
Your value is too big: largest acceptable value is 9999.
Please enter a year in the correct format: 432
You're right! 432 is a leap year.
```

*列表 1-29：运行* `*library-test*` *脚本*

在你的屏幕上，值过大时的错误信息会以粗体显示。此外，正确的闰年猜测将以绿色显示。

历史上，432 年不是闰年，因为闰年直到 1752 年才出现在日历中。但是我们现在讨论的是 Shell 脚本，而不是日历技巧，所以我们就不再纠结这个问题。

### #13 调试 Shell 脚本

尽管本节没有包含真正的脚本，但我们仍然想花几页时间讨论一些调试 Shell 脚本的基础知识，因为 bug 总是不可避免地会出现！

根据我们的经验，最佳的调试策略是逐步构建脚本。一些脚本程序员对第一次就能正确运行充满乐观，但从小处开始确实能帮助推进进程。此外，你应该大量使用`echo`语句来追踪变量，并明确调用你的脚本，使用`bash -x`来显示调试输出，如下所示：

```
$ bash -x myscript.sh
```

或者，你可以提前运行`set -x`来启用调试，运行结束后使用`set +x`来停止调试，如此处所示：

```
$ set -x
$ ./myscript.sh
$ set +x
```

要查看`-x`和`+x`的效果，我们来调试一个简单的数字猜测游戏，如列表 1-30 所示。

#### *代码*

```
   #!/bin/bash
   # hilow--A simple number-guessing game

   biggest=100                   # Maximum number possible
   guess=0                       # Guessed by player
   guesses=0                     # Number of guesses made
➊ number=$(( $$ % $biggest )    # Random number, between 1 and $biggest
   echo "Guess a number between 1 and $biggest"

   while [ "$guess" -ne $number ] ; do
➋   /bin/echo -n "Guess? " ; read answer
     if [ "$guess" -lt $number ] ; then
➌     echo "... bigger!"
     elif [ "$guess" -gt $number ] ; then
➍     echo "... smaller!
     fi
     guesses=$(( $guesses + 1 ))
   done

   echo "Right!! Guessed $number in $guesses guesses."

   exit 0
```

*列表 1-30：`*hilow*` 脚本，可能包含一些需要调试的错误...*

#### *工作原理*

要理解在 ➊处的随机数部分如何工作，请记住，序列 `$$` 是运行脚本的 Shell 的处理器 ID（PID），通常是一个 5 位或 6 位的数字值。每次运行脚本时，它都会得到一个不同的 PID。`% $biggest`序列将 PID 值除以指定的最大可接受值并返回余数。换句话说，`5 % 4 = 1`，`41 % 4`也等于 1。这是一种生成 1 到 `$biggest`之间的半随机数的简单方法。

#### *运行脚本*

调试这个游戏的第一步是测试并确保生成的数字足够随机。为此，我们获取运行脚本的 Shell 的 PID，使用`$$`表示法，并通过 `%` 模运算函数 ➊ 将其缩小到一个可用的范围。要测试此函数，请将命令直接输入到 Shell 中，如下所示：

```
$ echo $(( $$ % 100 ))
5
$ echo $(( $$ % 100 ))
5
$ echo $(( $$ % 100 ))
5
```

这样是可行的，但它并不算真正的随机。稍微思考一下就能发现原因：当命令直接在命令行上运行时，PID 总是相同的；但当它在脚本中运行时，每次都会在不同的子 Shell 中运行，因此 PID 会有所不同。

生成随机数的另一种方式是通过引用环境变量`$RANDOM`。它就像魔法一样！每次引用它，你都会得到不同的值。要生成一个在 1 到`$biggest`之间的随机数，你可以在 ➊处使用`$(( $RANDOM % $biggest + 1 ))`。

下一步是添加游戏的基本逻辑。首先生成一个 1 到 100 之间的随机数 ➊；玩家进行猜测 ➋；每次猜测后，玩家会被告知猜测是太大 ➌ 还是太小 ➍，直到他们最终猜中正确的值。在输入完所有基本代码后，接下来就是运行脚本，看看效果如何。这里我们使用了清单 1-30，包括所有瑕疵：

```
$ hilow
./013-hilow.sh: line 19: unexpected EOF while looking for matching '"'
./013-hilow.sh: line 22: syntax error: unexpected end of file
```

呃，Shell 脚本开发者的噩梦：意外的文件结尾（EOF）。仅仅因为错误信息提示错误出现在第 19 行，并不意味着问题真的是出在这一行。实际上，第 19 行是完全正常的：

```
$ sed -n 19p hilow
echo "Right!! Guessed $number in $guesses guesses."
```

为了理解发生了什么，记住引号中的内容可以包含换行符。这意味着当 Shell 遇到一个未正确关闭的引号时，它会一直读取脚本，寻找匹配的引号，直到最后一个引号，才会意识到有什么地方不对劲。

因此，问题可能出现在脚本的早期部分。Shell 返回的错误信息中唯一真正有用的内容是它告诉你哪个字符不匹配，因此我们可以使用 `grep` 来提取所有包含引号的行，并过滤掉那些包含两个引号的行，如下所示：

```
$ grep '"' 013-hilow.sh | egrep -v '.*".*".*'
echo "... smaller!
```

就这样！缺少了一个闭合引号，具体是在告知用户必须猜更小数字的那一行 ➍。我们将缺失的引号加到行尾，然后再试一次：

```
$ hilow
./013-hilow.sh: line 7: unexpected EOF while looking for matching ')'
./013-hilow.sh: line 22: syntax error: unexpected end of file
```

不行。又有问题了。因为脚本中括号表达式非常少，所以我们可以直接通过目测发现随机数实例化的闭合括号被错误地截断了：

```
number=$(( $$ % $biggest )          # Random number between 1 and $biggest
```

我们可以通过在行尾加上右括号来修复这个问题，但要放在代码注释之前。现在游戏是否能正常运行了呢？让我们来看看：

```
$ hilow
Guess? 33
... bigger!
Guess? 66
... bigger!
Guess? 99
... bigger!
Guess? 100
... bigger!
Guess? ^C
```

几乎解决了。但因为 100 是最大可能值，似乎代码的逻辑有问题。这些错误特别棘手，因为没有 fancy 的`grep`或`sed`命令来帮助定位问题。回顾一下代码，看看你能不能找出问题所在。

为了调试这个问题，我们可以添加几个`echo`语句来输出用户选择的数字，并验证输入的内容是否与被测试的内容一致。相关的代码部分从➋开始，但我们在这里为了方便重新打印了这些行：

```
  /bin/echo -n "Guess? " ; read answer
  if [ "$guess" -lt $number ] ; then
```

事实上，当我们修改了`echo`语句并查看这两行时，我们意识到了错误：读取的变量是`answer`，但被测试的变量叫做`guess`。这是个明显的错误，但并不罕见（尤其是当你使用奇怪的变量名时）。要修复这个问题，我们应该把`read answer`改成`read guess`。

#### *结果*

最终，它按预期工作了，如示例 1-31 所示。

```
$ hilow
Guess? 50
... bigger!
Guess? 75
... bigger!
Guess? 88
... smaller!
Guess? 83
... smaller!
Guess? 80
... smaller!
Guess? 77
... bigger!
Guess? 79
Right!! Guessed 79 in 7 guesses.
```

*示例 1-31：* `*hilow*` *shell 脚本游戏的完整运行*

#### *脚本调试*

这个小脚本中最严重的错误是它没有验证输入。输入任何非整数的内容，脚本就会卡住并失败。包含一个基本的测试，最简单的方式是将以下代码行添加到`while`循环中：

```
if [ -z "$guess" ] ; then
  echo "Please enter a number. Use ^C to quit"; continue;
fi
```

问题在于，确认它是一个非零输入并不意味着它是一个数字，像输入`hi`这样的内容会导致`test`命令出错。为了解决这个问题，可以在第五章脚本中调用`validint`函数，见第 23 页。
