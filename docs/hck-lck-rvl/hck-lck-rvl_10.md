

## 第七章：7 Python 入门



你在前几章中学到的技能对于调查泄漏的数据集非常有用，但拥有基本的编程知识更为强大。通过使用 Python 或其他编程语言，你可以给计算机下达精确的指令，执行现有工具或 Shell 脚本无法完成的任务。例如，你可以编写一个 Python 脚本，搜索一百万条视频元数据，确定视频拍摄的地点。根据我的经验，Python 比 Shell 脚本更简单、更易理解，且错误更少。

本章提供了 Python 编程基础的速成课程。你将学习编写和执行 Python 脚本，并使用交互式 Python 解释器。你还将使用 Python 做数学运算、定义变量、处理字符串和布尔逻辑、循环遍历列表中的项，并使用函数。未来的章节将依赖于你对这些基础技能的理解。

### 练习 7-1：安装 Python

一些操作系统，包括大多数版本的 Linux 和 macOS，预装了 Python，并且通常会安装多个版本的 Python。本书使用 Python 3。在本练习中，根据你的操作系统按照 Python 安装说明进行操作后，你应该能够使用 python3（适用于 Linux 和 Mac）或 python（适用于 Windows）命令运行 Python 脚本。

#### Windows

从 [*https://<wbr>www<wbr>.python<wbr>.org*](https://www.python.org) 下载并安装最新版本的 Python 3（适用于 Windows）。在安装过程中，勾选 **Add Python 3.*****x*** **to PATH**（其中 **3.*****x*** 是最新的 Python 3 版本），这将允许你在 PowerShell 中运行 python 命令，而无需使用 Python 程序的绝对路径。

本章要求你打开终端时，请使用 PowerShell 而不是 Ubuntu 终端。你也可以通过遵循本章中的 Linux 指南，在 Ubuntu 中使用 WSL 学习 Python，但直接在 Windows 中运行 Python 可以更快地读取和写入 Windows 格式的 USB 磁盘数据。

Windows 用户在运行本章中的示例代码时，应将所有的 python3 替换为 python。

#### Linux

打开终端并使用此 apt 命令确保已安装 python3、python3-pip 和 python3-venv 软件包：

```
**sudo apt install python3 python3-pip python3-venv**
```

该命令会安装 Ubuntu 软件库中可用的最新版本 Python 3（以及本章中需要的其他相关软件包），或者如果这些软件包已安装，则不会执行任何操作。

#### macOS

打开终端并运行以下 Homebrew 命令以确保已安装 python3：

```
**brew install python3**
```

该命令会安装 Homebrew 中可用的最新版本 Python 3，或者如果已经安装，则不会执行任何操作。

### 练习 7-2：编写你的第一个 Python 脚本

现在你已经下载了 Python，接下来你将编写并运行一个简单的 Python 脚本，在终端中显示一些文本。

在文本编辑器中，创建一个名为 *exercise-7-2.py* 的新文件（所有 Python 脚本以 *.py* 结尾）。第一次在 VS Code 中打开 Python 脚本时，它会询问你是否要安装 Python 扩展。我建议安装该扩展，以便在输入时启用 VS Code 提供建议。该扩展还具有高亮语法错误和帮助你格式化代码等功能。

输入以下代码（或从 [*https://<wbr>github<wbr>.com<wbr>/micahflee<wbr>/hacks<wbr>-leaks<wbr>-and<wbr>-revelations<wbr>/blob<wbr>/main<wbr>/chapter<wbr>-7<wbr>/exercise<wbr>-7<wbr>-2<wbr>.py*](https://github.com/micahflee/hacks-leaks-and-revelations/blob/main/chapter-7/exercise-7-2.py) 复制并粘贴），然后保存文件：

```
print("hacks")

print("leaks")

revelations = "revelations".upper()

print(revelations)
```

与 shell 脚本一样，Python 脚本逐行执行指令，从顶部开始。当你运行这段代码时，print("hacks") 调用了一个名为 print() 的函数，并将字符串 hacks 传递给它，在终端窗口中显示 hacks。第二行同样会显示 leaks。（我将在第 172 页的《Python 基础》一节中更详细地解释字符串，第 192 页的《函数》一节中解释函数。）

接下来，脚本定义了一个名为 revelations 的变量，并将其值设置为字符串 revelations 的大写版本。为了获取该字符串的大写版本，程序调用了 upper() 方法，这是一种函数类型。最后一行则显示了存储在 revelations 变量中的内容： REVELATIONS。

> 注意

*我对重新输入书中的代码片段有着美好的记忆。当我还是青少年时，我通过阅读编程书籍并将其中的代码示例输入到我自己的编辑器中，自己学习了网页和视频游戏开发。我发现，真正重新输入代码，而不是直接复制粘贴，能帮助我更好地理解概念，所以我建议你也这样做，完成本书中的练习。*

在终端中，切换到你的 *exercises* 文件夹，并运行你刚刚创建的脚本，使用以下命令（Windows 用户记得将 python3 替换为 python）：

```
micah@trapdoor chapter-7 % **python3 exercise-7-2.py**
```

此命令中的参数是你要运行的脚本的路径，*exercise-7-2.py*。你应该得到如下输出：

```
hacks

leaks

REVELATIONS
```

尝试对你的脚本进行以下更改，并在每次更改后运行脚本查看结果：

+   更改 print() 函数中的文本。

+   添加新的 print() 函数来显示更多文本。

+   使用字符串方法 lower() 和 capitalize() 来代替 upper()。

### Python 基础

在本节中，你将学习如何在交互式 Python 解释器中编写代码，如何注释代码，如何在 Python 中做简单的数学运算，以及如何使用字符串和列表。这本书将通过对 Python 语法的温和介绍，让你在深入学习更高级的内容之前，能迅速开始动手尝试代码。

在阅读时，不要害羞，可以上网查找关于 Python 的问题，特别是那些本书没有覆盖的部分。我经常在像 Stack Overflow 这样的网站上找到解决 Python 问题的方法，Stack Overflow 是一个论坛，大家可以在上面提问技术问题，其他人则可以回答。

#### 交互式 Python 解释器

*Python 解释器* 是一个命令行程序，它允许你实时运行 Python 代码，无需先编写脚本，从而可以快速测试命令。要打开 Python 解释器，你只需运行 python3 命令，无需任何参数，如下所示：

```
micah@trapdoor ~ % **python3**

`--snip--`

Type "help", "copyright", "credits" or "license" for more information.

>>>
```

解释器首先会告诉你正在使用哪个版本的 Python。类似于命令行界面，它会显示提示符 >>>，并等待你输入 Python 命令。

运行以下命令：

```
>>> **print("Hello World!")**

Hello World!

>>>
```

输入 print("Hello World!") 并按下 ENTER 键，应该会立即运行你的代码，并在下一行显示 Hello World!。通过运行 exit() 或按下 CTRL-D 退出解释器并返回到 shell。

在本书的后续章节中，如果我的示例中包含 >>> 提示符，表示它们正在 Python 解释器中运行。请在自己的解释器中运行相同的代码，并跟着一起学习。

#### 注释

编写代码对于即使是经验丰富的程序员来说也可能感到困惑，因此，*注释*代码总是个好主意：为自己或其他可能阅读你程序的人添加内联注释。如果你用简单的英语（或你所说的任何语言）描述代码特定部分的目的，那么将来查看这段代码的人就能一目了然地理解它的作用。

如果一行代码以井号（#）开头，则整行都是注释。你也可以在某些代码之后添加井号，并写上注释。例如，运行以下几行代码：

```
>>> **# This is a comment**

>>> **x = 10 # This sets the variable x to the value 10**

>>> **print(x)**

10
```

这与你在第三章中学习到的 shell 脚本中的注释完全相同。Python 会忽略注释，因为它们是给人看的。

#### 在 Python 中的数学运算

计算机，作为技术上复杂的计算器，非常擅长进行数学运算。虽然这可能并不立刻显现出来，但研究数据集意味着不断进行基本的数学操作：计算磁盘空间、统计文件数量、搜索关键词以及排序列表。以下是几个基本数学运算在 Python 中的实现方法：

**运算符**

加法（+）、减法（−）、乘法（×）和除法（/）的算术运算符在 Python 中大致相同：+、- 和 /，乘法则使用星号 *。

**变量**

在数学中，变量是占位符，通常是像 *x* 这样的字母。数学中的变量通常代表未知数，你的任务是解出它，但 Python 中的变量永远不是未知的——它们始终有一个值。给你的 Python 变量起一个有描述性的名字，例如 price 或 number_of_retweets，而不是没有明确意义的单个字母。正如你在本章后面看到的，Python 中的变量不仅仅代表数字。

**表达式**

表达式有点像由数字、变量和运算符组成的句子。例如，以下是一些表达式：

```
1 + 1

100 / 5

x * 3 + 5
```

像句子一样，表达式需要有正确的语法。就像“potato the inside”不是一个有效的句子，1 1 + 也不是一个有效的表达式。你可以在 Python 解释器中输入以下表达式，看看它如何评估这些表达式：

```
>>> **1** **+** **1**

2

>>> **100 / 5**

20.0

>>> **3.14 * 2**

6.28
```

就像计算器一样，Python 遵循运算顺序。它还支持使用括号：

```
>>> **100 - 12 * 2**

76

>>> **(100 - 12) * 2**

176
```

和其他数学运算一样，Python 不允许你除以零：

```
>>> **15 / 0**

Traceback (most recent call last):

  File "<stdin>", line 1, in <module>

ZeroDivisionError: division by zero
```

在 Python 中，你通过将一个值赋给变量来定义它，使用等号 (=)。尝试定义 price 和 sales_tax 变量，然后在表达式中使用它们：

```
>>> **price = 100**

>>> **sales_tax** **= .05**

>>> **total = price + (price * sales_tax)**

>>> **print(total)**

105.0
```

你不能使用尚未定义的变量。例如，如果你在表达式中使用一个未定义的变量 x，你会遇到错误：

```
>>> **x * 10**

Traceback (most recent call last):

  File "<stdin>", line 1, in <module>

NameError: name 'x' is not defined
```

你不仅仅是将一个变量赋值为某个值，通常你会希望通过某个量来修改其现有值。例如，如果你在 total 变量中记录购物车中商品的总价格，并希望在总额上加 10 美元，你可以这样定义这个变量：

```
total = total + 10
```

Python 的 += 运算符执行相同的操作：

```
total += 10
```

+= 运算符将右侧的数字加到左侧的变量上。Python 的运算符 -=、*= 和 /= 以相同的方式工作。在你的 Python 解释器中，定义一个变量，然后尝试使用这些运算符修改它的值。

#### 字符串

*字符串* 是一系列字符。任何时候你需要加载、修改或显示文本时，都可以将其存储在字符串中。如果你将一个文本文件的内容加载到 Python 变量中（例如，包含附件的 5MB EML 文件），那就是一个字符串。但字符串通常也很短：在练习 7-2 中，你使用了字符串 "hacks"、"leaks" 和 "revelations"。

在 Python 中，字符串必须用单引号（'）或双引号（"）括起来。运行以下示例，演示如何使用每种类型的引号。这里是一个使用双引号的字符串：

```
>>> **"apple"** 

'apple'
```

这是使用单引号的相同字符串：

```
>>> **'apple'** # The same string with single quotes

'apple'
```

如果字符串中包含单引号，可以使用双引号：

```
>>> **"She's finished!"**

"She's finished!"
```

如果字符串中包含双引号，可以使用单引号：

```
>>> **'She said, "Hello" '**

'She said, "Hello" '
```

在第三章中你学到的处理字符串的技巧，同样适用于 Python 中的字符串。如果你的字符串使用了双引号，可以像这样转义它们：

```
>>> **"She said, \"Hello\" "**
```

你也可以在单引号字符串中使用转义单引号：

```
>>> **'She\'s finished!'**
```

和数字一样，字符串也可以存储在变量中。运行以下代码来定义 first_name 和 last_name 变量，将我的名字替换为你的名字：

```
>>> **first_name = "****`Micah`****"**

>>> **last_name = "****`Lee`****"**
```

在 Python 中，*f-strings* 是包含变量的字符串。要使用 f-string，在引号前加上字母 f，然后将变量名放在大括号中（{ 和 }）。例如，运行以下命令来显示你刚刚定义的变量的值：

```
>>> **print(f"{first_name} {last_name}")**

Micah Lee

>>> **full_name = f"{first_name} {last_name}"**

>>> **print(f"{first_name}'s full name is {full_name}, but he goes by {first_name}")**

Micah's full name is Micah Lee, but he goes by Micah
```

将表达式放入 f-strings 中进行求值：

```
>>> **print(f"1 + 2 + 3 + 4 + 5 = {1 + 2** **+ 3 + 4 + 5}")**

1 + 2 + 3 + 4 + 5 = 15
```

Python 会为你计算表达式，在此情况下为 1 + 2 + 3 + 4 + 5，并直接打印结果，即 15。

### 练习 7-3：编写一个包含变量、数学和字符串的 Python 脚本

在这个练习中，你将通过编写一个简单的 Python 脚本来练习迄今为止学到的概念，该脚本使用变量和一些基本的数学表达式并打印一些字符串。该脚本根据姓名和年龄（以年为单位）计算一个人以月、日、小时、分钟和秒为单位的年龄，然后显示这些信息。在文本编辑器中，创建一个名为 *exercise-7-3.py* 的新文件，并定义这两个变量：

```
**name** **= "****`Micah`****"**

**age_years =** **`38`**
```

将 name 和 age_years 的值替换为你自己的姓名和年龄。

接下来，定义一些表示不同单位年龄的变量：月、日、小时、分钟和秒。先从月开始：

```
**age_months = age_years * 12**
```

添加一个天数变量：

```
**age_days =** **age_years * 365**
```

最后，定义小时、分钟和秒的变量：

```
**age_hours = age_days * 24**

**age_minutes = age_hours * 60**

**age_seconds = age_minutes * 60**
```

现在你已经定义了变量，可以将它们显示给用户。由于本练习中的数字会变得很大，你将包括逗号以使其更容易阅读。例如，在解释器中运行以下代码，以使用 f-string 显示带有逗号的 number 变量，方法是在变量名的花括号内添加 :,：

```
>>> **number = 1000000**

>>> **print(f"the number is: {number}")**

the number is: 1000000

>>> **print(f"the number is: {number:,}")**

the number is: 1,000,000
```

返回 Python 脚本中，添加代码以显示所有这些值，像这样：

```
print(f"{name} is {age_years:,} years old")

print(f"That would be {age_months:,} months old")

print(f"Which is {age_days:,} days old")

print(f"Which is {age_hours:,} hours old")

print(f"Which is {age_minutes:,} minutes old")

print(f"Which is {age_seconds:,} seconds old")
```

这段代码使用 {name} 来显示姓名变量的值。该变量是一个字符串，因此尝试用逗号分隔它是没有意义的。然而，其它变量是数字，所以代码在所有这些变量的花括号内包括了 :,，以便在输出中加入逗号。（age_years 的值不需要逗号，除非你超过了 1,000 岁，但使用 :, 语法也没有坏处——它只会在需要时添加逗号。）

在文本编辑器中保存文件。（脚本的完整副本可在[*https://<wbr>github<wbr>.com<wbr>/micahflee<wbr>/hacks<wbr>-leaks<wbr>-and<wbr>-revelations<wbr>/blob<wbr>/main<wbr>/chapter<wbr>-7<wbr>/exercise<wbr>-7<wbr>-3<wbr>.py*](https://github.com/micahflee/hacks-leaks-and-revelations/blob/main/chapter-7/exercise-7-3.py)找到。）在终端中，切换到此练习的*exercises*文件夹并运行脚本。以下是我运行时的结果：

```
micah@trapdoor chapter-7 % **python3 exercise-7-3.py**

Micah is 38 years old

That would be 456 months old

Which is 13,870 days old

Which is 332,880 hours old

Which is 19,972,800 minutes old

Which is 1,198,368,000 seconds old
```

当你运行带有姓名和年龄的脚本时，尝试更改年龄并重新运行，看看数字是如何变化的。

### 列表和循环

在研究数据集时，你经常需要管理列表。例如，你可能会处理文件名的列表或电子表格中的行。在这一节中，你将学习如何将列表存储为变量，并循环遍历这些列表，以便对每个列表项运行相同的代码。在第四章中，你已经在 shell 中使用过for循环，今天你将使用 Python 来处理。

#### 定义和打印列表

在 Python 中，列表是用括号（[和]）定义的，列表中的每一项由逗号（,）分隔。你可能会有一个数字列表：

```
[1, 2, 3]
```

或者是字符串：

```
["one", "two", "three"]
```

或者是一个空列表：

```
[]
```

就像变量可以包含数字或字符串一样，它们也可以包含列表。使用以下代码行将希伯来字母表的字母列表（使用拉丁字母拼写）存储在hebrew_letters变量中：

```
>>> **hebrew_letters = ["aleph", "bet", "gimel", "dalet", "he", "vav", "zayin",**

**"chet", "tet", "yod", "kaf", "lamed", "mem", "nun", "samech", "ayin", "pe",**

**"tsadi", "qof", "resh", "shin", "tav"]**
```

现在使用print()函数来显示hebrew_letters变量中的项：

```
>>> **print(hebrew_letters)**

['aleph', 'bet', 'gimel', 'dalet', 'he', 'vav', 'zayin', 'chet', 'tet', 'yod',

'kaf', 'lamed', 'mem', 'nun', 'samech', 'ayin', 'pe', 'tsadi', 'qof', 'resh',

'shin', 'tav']
```

你可以通过将列表中的每一项单独输入一行并缩进，使长列表更易于阅读，如下所示：

```
hebrew_letters = [

    "aleph",

`--snip--`

    "tav"

]
```

列表中的每一项都有一个*索引*，即表示该项在列表中位置的数字。第一个项的索引是 0，第二个是 1，第三个是 2，依此类推。要选择列表项，你需要将该项的索引放入括号中并附加到列表的末尾。例如，要选择hebrew_letters列表中的第一个字母，可以使用hebrew_letters[0]：

```
>>> **print(hebrew_letters[0])**

aleph

>>> **print(hebrew_letters[1])**

bet
```

第一行代码使用print()函数显示hebrew_letters列表中索引为 0 的项（aleph），第二行显示索引为 1 的项（bet）。

现在可以使用负数来从列表的末尾开始选择项目，如下所示：

```
>>> **print(hebrew_letters[-1])**

tav

>>> **print(hebrew_letters[-2])**

shin
```

你可以使用len()函数来计算列表中项的数量。例如，运行以下代码来获取hebrew_letters列表中的项数：

```
>>> **print(len(hebrew_letters))**

22
```

这段代码使用print()函数显示len()函数的输出。你也可以将len()函数的输出存储到一个变量中，以得到相同的结果：

```
>>> **length_of_hebrew_alphabet = len(hebrew_letters)**

>>> **print(length_of_hebrew_alphabet)**

22
```

第一行代码运行len(hebrew_letters)并将结果存储在length_of_hebrew_alphabet变量中。第二行使用print()函数显示该结果。

你不必将列表存储在变量中就能从中选择项。例如，运行这段代码来显示列表[1,2,3]中的第二项（索引为 1）：

```
>>> **print([1,2,3][1])**

2
```

append()方法允许你向列表中添加项目。例如，运行以下代码将一个新颜色添加到收藏列表中：

```
>>> **favorite_colors = ["red", "green", "blue"]**

>>> **favorite_colors.append("black")**

>>> **print(favorite_colors)**

['red', 'green', 'blue', 'black']
```

这段代码将变量favorite_colors定义为一个包含red、green和blue的字符串列表。然后，它通过使用append()方法将另一个字符串black添加到列表中，最后使用print()函数显示favorite_colors变量的值。

在编写分析数据集的代码时，你通常会创建一个空列表，然后将项目添加到该列表中，以便更容易处理数据。例如，在第十三章中，你将了解到我在调查美国前线医生（一家反疫苗组织）时编写的代码。为了正确分析包含患者信息的数十万个文件的数据集，我编写了代码来创建一个空列表，打开每个文件，并将相关的患者数据附加到该列表中。

#### 运行 for 循环

在第四章中，你使用了一个for循环来解压每个 BlueLeaks ZIP 文件。Python 也有for循环，它们的工作方式与 Shell 脚本中的 for 循环相同：通过对列表中的每一项运行一个代码片段（称为*代码块*）。一个for循环有以下语法：

```
for `variable_name` in `list_name`:
```

这个语法后面跟着一个缩进的代码块。一旦你选择了一个新的变量名variable_name，你就可以在代码块中使用它。

例如，运行以下代码循环遍历hebrew_letters列表，将每个项存储在变量letter中，然后显示该项：

```
>>> **for letter in hebrew_letters:**

...     **print(letter)**

...
```

在你输入 for 循环后，它以冒号（:）结束，Python 解释器会将提示符从 >>> 改为 …，并等待你输入将在每个项上运行的代码块。你需要在代码块的每一行前缩进相同数量的空格，然后用一个空行结束你的代码块。在这个例子中，运行的代码块只有一行： print(letter)。

代码应该返回以下输出：

```
aleph

bet

`--snip--`

shin

tav
```

在这个例子中，for 循环执行了 22 次，每次遍历列表中的每个项，并将项存储在变量 letter 中。第一次循环时，letter 的值是 aleph。第二次时，值是 bet，依此类推。

> 注意

*缩进告诉 Python 哪些代码行是代码块的一部分。如果某些行用四个空格缩进，但其他行用两个或三个空格，Python 代码就会无法运行。为了保持简单，我建议总是使用四个空格进行缩进。当在 VS Code 中编写脚本时，你可以通过鼠标选择多行代码，然后按* *TAB* *（它会为你缩进四个空格），或者按 *SHIFT-TAB* *取消缩进*。*

以下是一个稍微复杂一点的例子，使用 len() 函数来计算字符串中的字符数量，而不是列表中的项目数：

```
>>> **for letter in hebrew_letters:**

...     **count = len(letter)**

...     **print(f"The letter {letter} has {count} characters")**

...

The letter aleph has 4 characters

The letter bet has 3 characters

The letter gimel has 5 characters

`--snip--`

The letter resh has 4 characters

The letter shin has 4 characters

The letter tav has 3 characters
```

这段代码会告诉你每个希伯来字母在拉丁字母中拼写该单词时使用了多少个字符。

你也可以使用 for 循环来遍历字符串，因为字符串本质上是字符的列表：

```
>>> **word = "hola"**

>>> **for character in word:**

...     **print(character)**

...

h

o

l

a
```

你可以根据需要运行一个单独的 for 循环，针对你正在处理的数据集。例如，在第九章中，你将编写代码来打开 BlueLeaks 数据集中的每个数百个电子表格，并使用 for 循环在每一行上运行你的代码块。

在接下来的章节中，你将学会通过确定哪些代码块在什么情况下运行，使你的程序更加动态和有用。

### 控制流

Python 脚本从顶部开始，逐行执行代码，但它们并不总是按顺序执行这些代码行。例如，在 for 循环中，同一段代码可能会反复执行，直到循环完成，程序才继续到下一行。代码执行的顺序就是你程序的 *控制流程*。

当你开始编写代码时，你会经常通过告诉计算机在不同情况下做不同的事情来改变控制流程。例如，如果你编写一个程序来遍历数据集中的文件列表，你可能希望在程序处理 PDF 文档时运行不同的代码，而不是遇到 MP4 视频时。

本节将教你如何在特定条件下运行某些代码块。为此，你将学习如何比较值，基于这些比较使用 if 语句，并使用布尔逻辑表达任意复杂的条件，这一切都可以让你控制程序的流程。当你编写搜索数据集中特定内容的代码并根据找到的内容做出响应时，你将需要这种逻辑。

#### 比较运算符

如本章前面所述，使用算术运算符 +、-、/ 和 * 的表达式通常会评估为数字：例如，1 + 1 会评估为 2。Python 中的表达式还使用以下 *比较运算符* 来比较项：

< 小于

<= 小于或等于

> 大于

>= 大于或等于

== 等于（与单个等号（=）定义变量不同）

!= 不等于

*布尔值*是一种变量类型，它的值要么是 True 要么是 False。使用比较运算符的表达式会评估为布尔值，而不是数字，如以下示例所示：

```
>>> **100 > 5**

True

>>> **100 < 5**

False

>>> **100 > 100**

False

>>> **100 >= 100**

True

>>> **0.5 <** **1**

True

>>> **0.999999 == 1**

False
```

你也可以使用这些相同的运算符来比较字符串。在 Python 中，说一个字符串小于另一个字符串意味着前者在字母顺序中排在后者之前，如以下示例所示：

```
>>> **"Alice" == "Bob"**

False

>>> **"Alice" != "Bob"**

True

>>> **"Alice" < "Bob"**

True

>>> **"Alice" > "Bob"**

False
```

字符串是区分大小写的。如果你不在意大小写，只想看看字符串是否由相同的单词组成，可以在比较之前将它们都转换为小写：

```
>>> **name1 = "Vladimir Putin"**

>>> **name2 = "vladimir putin"**

>>> **name1 == name2**

False

>>> **name1.lower() == name2.lower()**

True
```

这项技术使你能够判断正在评估的数据是否满足给定条件。例如，在 第十一章 中，你将编写代码分析上传到极右翼社交网络 Parler 的超过百万个视频的元数据。通过使用比较运算符，你将判断哪些视频是在 2021 年 1 月 6 日，在特朗普失去 2020 年大选后，华盛顿特区暴动期间拍摄的。

#### if 语句

你使用 if 语句来告诉代码在某些条件下执行某些操作，而在其他条件下不执行。if 语句的语法是 if 表达式:，然后跟着一个缩进的代码块。如果表达式的结果为 True，那么代码块就会执行。如果表达式的结果为 False，代码就不会执行，流程将转到下一行。

例如，运行以下代码：

```
>>> **password = "letmein"**

>>> **if password == "letmein":**

...     **print("ACCESS GRANTED")**

...     **print("Welcome")**

...

ACCESS GRANTED

Welcome

>>>
```

这段代码将 password 变量的值设置为 letmein。这意味着 if 语句中的表达式（password == "letmein"）的结果为 True，于是代码块被执行，显示 ACCESS GRANTED 和 Welcome。

现在尝试在你的 if 语句中包含错误的密码：

```
>>> **password = "yourefired"**

>>> **if password == "letmein":**

...     **print("ACCESS GRANTED")**

...     **print("Welcome")**

...

>>>
```

这次，由于你将密码设置为 "yourefired"，表达式 password == "letmein" 的结果为 False，因此 Python 不会执行 if 语句中的代码块。

一个 if 语句可以选择性地包含一个 else 语句块，这样如果条件为真，则运行一个代码块；如果条件为假，则运行另一个代码块：

```
if password == "letmein":

    print("ACCESS GRANTED")

    print("Welcome")

else:

    print("ACCESS DENIED")
```

你也可以加入 elif 语句块，它是“else if”的缩写。这样可以在第一次比较为假时进行另一次比较，正如 Listing 7-1 所示。

```
if password == "letmein":

    print("ACCESS GRANTED")

    print("Welcome")

elif password == "open sesame":

    print("SECRET AREA ACCESS GRANTED")

else:

    print("ACCESS DENIED")
```

Listing 7-1: 比较 if, elif, 和 else 语句

在这段代码中，if 语句评估 password == "letmein" 表达式。如果它的值为 True，代码块运行并显示 ACCESS GRANTED 和 Welcome 消息。如果表达式的值为 False，程序将跳转到 elif 语句块，它评估 password == "open sesame" 表达式。如果该表达式的值为 True，则运行显示 SECRET AREA ACCESS GRANTED 的代码块。如果表达式的值为 False，程序将跳转到 else 代码块，显示 ACCESS DENIED。

#### 嵌套代码块

你还可以通过多个 if 语句并且不使用 elif 来实现 Listing 7-1 的结果，使用*嵌套*的代码块，或者在其他缩进的代码块内嵌套缩进的代码块：

```
if password == "letmein":

    print("ACCESS GRANTED")

    print("Welcome.")

else:

    if password == "open sesame":

        print("SECRET AREA ACCESS GRANTED")

    else:

        print("ACCESS DENIED")
```

这段代码的功能与 Listing 7-1 相同。

代码越复杂，嵌套代码块可能越有用。你可能会在 if 语句的代码块中包含 for 循环，或者在 for 循环中包含 if 语句，甚至在 for 循环中再嵌套 for 循环。

出于可读性考虑，你可能会更倾向于使用 elif 语句，而不是嵌套的 if 语句：使用 100 个 elif 语句比因为有 100 个嵌套的 if 语句而导致代码缩进 100 次要更易读和编写。

#### 搜索列表

Python 中的 in 运算符可以告诉你某个项是否出现在列表中，它在处理列表时非常有用。例如，要检查数字 42 是否出现在数字列表中，你可以按如下方式使用 in：

```
favorite_numbers = [7, 13, 42, 101]

if 42 in favorite_numbers:

    print("life, the universe, and everything")
```

在 in 运算符的左侧是列表中的潜在项，右侧是列表。如果该项在列表中，则表达式的值为 True。如果不在，则值为 False。

你还可以使用 not in 来检查某个项是否*不在*列表中：

```
if 1337 not in favorite_numbers:

    print("mess with the best, die like the rest")
```

此外，你还可以使用 in 来搜索较小的字符串是否出现在较大的字符串中：

```
sentence = "What happens in the coming hours will decide how bad the Ukraine

crisis gets for the vulnerable democracy in Russian President Vladimir Putin's

sights but also its potentially huge impact on Americans and an already deeply

unstable world."

if "putin" in sentence.lower():

    print("Putin is mentioned")
```

这段代码定义了变量 sentence，然后检查字符串 putin 是否出现在该句子的所有小写字母版本中。

#### 逻辑运算符

无论情景多么复杂，都可以使用 *逻辑运算符* and、or 和 not 来描述。像比较运算符一样，逻辑运算符的值也为 True 或 False，并且它们允许你组合多个比较。

例如，假设你喜欢天文学，并且想知道现在是不是适合观星的好时机。我们可以将这个情况设置为一个逻辑表达式：如果（（天黑了）**并且**（没有下雨）**并且**（没有多云））**或**（你有詹姆斯·韦布太空望远镜的使用权限），那么是的。否则，否。逻辑运算符允许你在 Python 代码中定义这种逻辑。

像其他运算符一样，and 和 or 运算符将左边的表达式与右边的表达式进行比较。使用 and 时，如果两边都为真，整个表达式为真。如果任一边为假，整个表达式为假。例如：

True and True == True

True and False == False

False and True == False

False and False == False

使用 or 时，如果任一表达式为真，整个表达式为真。只有当两个表达式都为假时，整个表达式才为假。例如：

True or True == True

True or False == True

False or True == True

False or False == False

not 表达式与其他表达式不同，它不使用左侧的表达式，而只使用右侧的表达式。它将真值转换为假，假值转换为真。例如：

not True == False

not False == True

总结来说，使用 and 来判断两个条件是否都为真，使用 or 来判断两个条件中至少有一个为真，使用 not 来将真值转换为假，反之亦然。例如，考虑下面的代码：

```
if country == "US" **and** age >= 21:

    print("You can legally drink alcohol")

else:

    if country != "US":

        print("I don't know about your country")

    else:

        print("You're too young to legally drink alcohol")
```

第一个 if 语句包含一个表达式，比较两个其他表达式，country == "US" 和 age >= 21。如果 country 是 US 并且 age 大于或等于 21，该表达式将简化为 True and True。由于两个布尔值都为真，这个表达式的值为 True，因此 if 语句后的代码块将执行，屏幕上会打印出 You can legally drink alcohol。

第一个else代码块决定了当该表达式求值为False时会发生什么。例如，如果country是Italy，但age是30，则表达式简化为False and True。由于至少有一个布尔值为假，因此结果为False，所以else后面的代码块会运行。同样，如果country是US，但age是18，那么表达式简化为True and False。这同样求值为False，因此else后面的代码块会运行。

在第二个else代码块中是一个简单的if语句，没有布尔逻辑：如果country不是US，则屏幕显示I don't know about your country。否则（意味着country是US），则显示You're too young to legally drink alcohol。

就像数学中一样，您可以在if语句中使用括号来比较多个表达式。例如，美国的法定饮酒年龄是 21 岁，而意大利的法定饮酒年龄是 18 岁。让我们在这个程序中加入意大利，这次使用or运算符：

```
if (country == "US" and age >= 21) **or** (country == "Italy" and age >= 18):

    print("You can legally drink alcohol")

else:

    if country not in ["US", "Italy"]:

        print("I don't know about your country")

    else:

        print("You're too young to legally drink alcohol")
```

用简单的英语来说，第一个 if 语句告诉程序：如果你的国家是美国且你至少 21 岁，*或者* 如果你的国家是意大利且你至少 18 岁，那么你可以合法饮酒。在这两种情况下，if 语句中的整个表达式为真，程序将打印 You can legally drink alcohol。如果其中只有一个条件为真，另一个条件为假（例如，如果你是一个 19 岁的意大利人），整个语句仍然为真。这就是 or 的意思：如果你比较的两个条件中任意一个为真，那么整个表达式为真。

使用运算符 not 可以将 True 转换为 False，或者将 False 转换为 True。例如：

```
if country == "US" and **not** age >= 21:

    print("Sorry, the drinking age in the US is 21")
```

你可以将 not age >= 21 替换为 age < 21，效果相同。

#### 异常处理

Python 程序可能会因为一个错误而突然退出，这个错误被称为 *异常*。这通常被称为“抛出异常”。*异常处理* 确保你的 Python 代码在捕获异常时会运行另一个代码块，而不是因错误退出。

在本章中，你已经看过几个异常的例子，比如尝试除以零（这是数学中不允许的）或者使用未定义的变量：

```
>>> **15 / 0**

Traceback (most recent call last):

  File "<stdin>", line 1, in <module>

ZeroDivisionError: division by zero

>>> **x * 10**

Traceback (most recent call last):

  File "<stdin>", line 1, in <module>

NameError: name 'x' is not defined
```

在这些情况下，Python 分别抛出了 ZeroDivisionError 异常和 NameError 异常。

你可以编写捕获异常的代码，当异常被抛出时允许你优雅地处理它们。例如，假设你有一个名为 names 的名字列表，并且你想显示列表中的第一个名字：

```
>>> **names = ["Alice", "Bob", "Charlie"]**

>>> **print(f"The first name is {names[0]}")**

The first name is Alice
```

这段代码显示 names[0] 的值，即 names 列表中的第一个项目。如果列表中有几个名字，这段代码按预期工作。但是，如果 names 为空呢？

```
>>> **names = []**

>>> **print(f"The first name is {names[0]}")**

Traceback (most recent call last):

  File "<stdin>", line 1, in <module>

IndexError: list index out of range
```

在这种情况下，由于索引 0 不存在（因为列表为空），Python 抛出了一个IndexError异常。

你可以使用try和except语句来捕获这个异常，像这样：

```
try:

    print(f"The first name is {names[0]}")

except:

    print("The list of names is empty")
```

这段代码首先运行一个try语句，接着是一个代码块。它尝试运行代码块中的代码，如果没有遇到异常，它会继续执行except块之后的下一行代码。然而，如果遇到异常，它会先运行except块中的代码，然后再继续。

当没有异常时，代码的表现如下：

```
>>> **names = ["Alice", "Bob", "Charlie"]**

>>> **try:**

...     **print(f"The first name is {names[0]}")**

... **except:**

...     **print("The list of names is empty")**

...

The first name is Alice
```

在这种情况下，try语句后的代码块成功运行，因此控制流跳过了except块。

当抛出异常，但代码捕获并优雅地处理它时，代码的表现如下：

```
>>> **names = []**

>>> **try:**

...     **print(f"The first name is {names[0]}")**

... **except:**

...     **print("The list of names is empty")**

...

The list of names is empty
```

try语句后的代码块运行了，但是当 Python 在评估names[0]时抛出了IndexError异常。该代码没有崩溃或显示错误，而是捕获了异常并执行了except块。在这种情况下，如果在try块中抛出任何异常，except语句都会执行，但你可以使用不同的except语句来处理不同类型的异常。请考虑以下示例：

```
try:

    `--snip--`

except ZeroDivisionError:

    # This catches ZeroDivisionError exception

    `--snip--`

except NameError:

    # This catches NameError exceptions

    `--snip--`

except IndexError:

    # This catches IndexError exceptions

    `--snip--`

except:

    # This catches any other exceptions that haven't been caught yet

    `--snip--`
```

通过使用except Exception:，你可以将Exception替换为你感兴趣捕获的具体异常，编写不同的代码来处理不同类型的异常。在第十章中，你将学习如何处理 JSON 数据时重新回顾异常处理，而在第十四章的案例研究中，你将学习如何处理新纳粹聊天日志。

现在你已经了解了 Python 中的控制流如何工作，接下来你将练习一些基本的 Python 语法，并在下一个练习中使用if语句和布尔逻辑进行比较。

### 练习 7-4：练习循环和控制流

在社交媒体俚语中，一种常见的讽刺方式是使用 *交替大小写*，即在引用别人时从大写字母切换到小写字母，再切换回大写字母。例如，这是来自现在被暂停的 Twitter 账户 @BigWangTheoryy 的一条病毒性推文的文本：

> *不及格的课程*我：“我可以加个学分吗？”教授：“cAn i GEt SomE eXtRa creDiT？”

在这个练习中，你将编写一个 Python 脚本，使用你在上一节中学到的控制流概念，从一些文本开始，并将其转换为交替大小写风格。

在你的文本编辑器中，创建一个名为 *exercise-7-4.py* 的新文件，并像这样定义变量 text：

```
text = "One does not simply walk into Mordor"
```

编写这个脚本的最简单方法是从一个空字符串开始，叫做 alternating_caps_text，然后循环遍历 text 中的字符，一次添加一个字符到 alternating_caps_text 中，并在添加过程中交替改变它们的大小写。像这样在脚本中添加第二行来定义该变量：

```
alternating_caps_text = " "
```

接下来，你将定义一个名为 should_be_capital 的布尔变量。每次循环遍历 text 中的字符时，你将使用这个布尔变量来跟踪当前字符是否应该是大写或小写。对于这个例子，先从大写字母开始：

```
should_be_capital = True
```

在那行下面，添加脚本的主要部分：

```
for character in text:

    if should_be_capital:

        alternating_caps_text += character.upper()

         should_be_capital = False

    else:

         alternating_caps_text += character.lower()

         should_be_capital = True
```

使用 for 循环，这段代码遍历 text 中的字符，将每个字符存储在 character 变量中。然后，它将这些字符添加到 alternating_caps_text 中，并在大写和小写之间切换。

在每次执行 for 循环时，character 是 text 中的另一个字符，text 变量包含了字符串 "One does not simply walk into Mordor"。代码第一次循环时，character 是 O。当代码执行到 if 语句时，should_be_capital 对于这个字符的值为 True，因此执行代码块。+= 运算符将 character.upper()（即 character 的大写版本）添加到 alternating_caps_text 中。由于代码一开始是添加大写字母，因此接下来希望添加小写字母，因此将 should_be_capital 设置为 False。代码块结束后，代码开始进入第二次循环。

在第二次迭代中，character 为 n，而 should_be_capital 的值为 False。当代码执行到 if 语句时，表达式的值为 False，因此执行 else 块。这个过程与另一个块类似，不同的是，它将字符的小写版本 character.lower() 附加到 alternative_caps_text 中，并将 should_be_capital 重新设置为 True。到目前为止，alternating_caps_text 的值是 On。

在第三次迭代时，character是e，并且should_be_capital的值为True。当代码进入if语句时，表达式的值为True，因此该代码块再次执行，将大写字母E添加到alternating_caps_text中，并将should_be_capital重新设置为False。代码以这种方式继续处理text中的其余字符。请注意，空格字符的大小写版本，" ".upper()和" ".lower()是相同的。upper()和lower()方法也不会改变像,、.、!等标点符号的字符。

当这个for循环完成时，你只需要通过在脚本中添加这一行来显示alternating_caps_text的值：

```
print(alternating_caps_text)
```

你的 Python 脚本已经完成（你也可以在[*https://<wbr>github<wbr>.com<wbr>/micahflee<wbr>/hacks<wbr>-leaks<wbr>-and<wbr>-revelations<wbr>/blob<wbr>/main<wbr>/chapter<wbr>-7<wbr>/exercise<wbr>-7<wbr>-4<wbr>.py*](https://github.com/micahflee/hacks-leaks-and-revelations/blob/main/chapter-7/exercise-7-4.py)找到完整的副本）。运行你的脚本。以下是我得到的输出：

```
micah@trapdoor chapter-7 % **python3 exercise-7-4.py**

OnE DoEs nOt sImPlY WaLk iNtO MoRdOr
```

现在改变text的值并重新运行脚本。例如，我将值更改为"There are very fine people on both sides"：

```
micah@trapdoor chapter-7 % **python3 exercise-7-4.py**

ThErE ArE VeRy fInE PeOpLe oN BoTh sIdEs
```

你已经掌握了使用列表、循环和控制执行流程的基础知识。本章将以另一项基本编程技能结束：使用函数将你的代码拆分成更简单的模块。

### 函数

程序越复杂，你越需要将要解决的问题分解成更小的部分，并分别处理它们。这可以让你专注于更大的整体，通过使用这些小块代码作为构建块来实现。在本节中，你将学习如何通过函数来实现这一点。

*函数*，作为编程的基本构建块，是可重复使用的代码块。它们接受*参数*—你传递给函数的变量—作为输入，并在运行完成后可以*返回*一个值。你已经使用过一些 Python 提供的函数，如 print() 和 len()，但你也可以定义自己的函数，并且可以多次使用它，而不需要重新编写那段代码。在本节中，你将学习如何做到这一点。

#### def 关键字

你可以使用 def 关键字来定义一个新函数。例如，下面的代码定义了一个名为 test() 的函数，它会在终端打印一串字符串：

```
>>> **def test():**

...     **print("this is a test function")**

...

>>> **test()**

this is a test function
```

函数定义行以冒号结束，后面跟着一个缩进的代码块，定义函数的具体行为：在这个例子中，它显示字符串 this is a test function。这个 test() 函数没有任何参数，这意味着每次你运行它时，它都会做完全相同的事情。

Listing 7-2 定义了一个稍微复杂一点的函数 sum()，它将两个数字相加。

```
def sum(a, b):

    return a + b
```

Listing 7-2: 定义一个示例函数

这个新函数接受 a 和 b 作为参数，并返回这两个变量的和。对于任何接受多个参数的函数，如本例，你需要用逗号 (,) 分隔这些参数。

每个变量都有一个*作用域*，它描述了代码中的哪些部分可以使用该变量。一个函数的参数（在这个例子中是 a 和 b），以及在函数内部定义的任何变量，具有只能在该函数代码块内访问的作用域。换句话说，你只能在 sum() 函数内部使用这些 a 和 b 变量，它们不会在该代码块外部定义。

你可以把定义一个函数看作是告诉 Python：“我正在创建一个新函数，命名为这个，并且这是它的功能。”然而，函数本身不会执行，直到你*调用*它。考虑以下 Python 脚本：

```
def sum(a, b):

    return a + b

red_apples = 10

green_apples = 6

total_apples = sum(red_apples, green_apples)

print(f"There are {total_apples} apples")
```

首先，代码定义了一个名为sum()的函数，该函数只是一个包含return语句的代码块。这个函数尚未执行。然后，代码定义了red_apples变量，并将其值设置为10，还定义了green_apples变量，并将其值设置为6。

下一行以total_apples =开始，但在 Python 能够设置该变量的值之前，它需要知道该值应为多少。为此，代码首先调用了sum()函数，传入red_apples和green_apples作为a和b。现在代码终于调用了这个函数，return a + b会运行。在这个函数调用中，a是red_apples，b是green_apples。函数返回a + b，即16。现在sum()函数已经返回，代码定义了一个名为total_apples的变量，并将其值设置为sum()函数的返回值，即16。

最后，代码调用了print()函数，并传入一个 f-string 作为参数，显示total_apples变量。它会显示消息There are 16 apples。

#### 默认参数

函数定义也可以有*默认参数*，这意味着定义它们的值是可选的。如果在调用函数时没有传入这些参数的值，那么默认值将会被使用。

例如，考虑这个函数，它接受一个数字，并可选地接受感叹号和问号的数量，使用这些参数打印一个问候语：

```
def greet(name, num_exclamations=3, num_questions=2):

    exclamations = "!" * num_exclamations

    questions = "?" * num_questions

    print(f"Hello {name}{exclamations}{questions}")
```

参数 name 是*位置参数*，这意味着在调用函数时，你传入的第一个参数必须始终是 name。然而，num_exclamations 和 num_questions 是默认参数，所以传入这些参数的值是可选的。greet() 函数定义了字符串 exclamations 和 questions，并将它们设置为一系列感叹号和问号。（在 Python 中，当你将一个字符串乘以一个数字时，结果是将原字符串重复多次；例如，"A" * 3 结果是字符串 AAA。）然后代码会显示 Hello，接着是 name 的值，再后面是传入函数的感叹号和问号的数量。

这个函数有一个位置参数（name）和两个默认参数（num_exclamations 和 num_questions）。你可以只传入 name，而不传入默认参数的值，这样它们会自动设置为 3 和 2，分别对应感叹号和问号的数量：

```
>>> **greet("Alice")**

Hello Alice!!!??
```

你也可以保留其中一个默认参数的默认值，但为另一个选择一个新的值。当你手动为默认参数选择值时，你就是在使用*关键字参数*。例如：

```
>>> **greet("Bob", num_exclamations****=5, num_questions=5)**

Hello Bob!!!!!?????

>>> **greet("Charlie", num_questions=0)**

Hello Charlie!!!

>>> **greet("Eve", num_exclamations=0)**

Hello Eve??
```

第一个函数调用使用了关键字参数来传递 num_exclamation 和 num_questions；第二个函数调用仅使用了 num_questions 的关键字参数，并且对 num_exclamations 使用了默认参数；第三个函数调用则使用了 num_exclamations 的关键字参数，而对 num_questions 使用了默认参数。

#### 返回值

函数在接受输入、进行计算并返回一个值时变得更加有用，这个返回的值被称为 *返回值*。之前描述的 greet() 函数只是显示输出，但它不会返回一个我可以保存在变量中或传递给其他函数的值。然而，之前你使用的 len() 函数接受输入（一个列表或字符串），进行计算（计算列表或字符串的长度），并返回一个值（该长度）。

这是一个示例函数，它接受一个字符串 s 作为参数，并返回该字符串中元音字母的数量：

```
def count_vowels(s):

    number_of_vowels = 0

    vowels = "aeiouAEIOU"

    for c in s:

        if c in vowels:

            number_of_vowels += 1

    return number_of_vowels
```

这个函数综合了本章迄今为止讲解的许多概念：它将变量 number_of_vowels 定义为 0，然后将变量 vowels 定义为包含大小写英语元音字母的字符串。接下来，它使用 for 循环遍历传入函数的字符串 s 中的每个字符。

在每次循环中，代码使用 if 语句检查字符是否为元音字母（由于 vowels 包含大小写字母，因此此代码将 a 和 A 都视为元音字母）。如果字符是元音字母，代码会将 number_of_vowels 变量加一。最后，它返回 number_of_vowels，其值等于在 s 中计算出的元音字母数量。

下面是一些调用此函数并传入不同字符串的示例：

```
>>> **count_vowels("THINK")**

1

>>> **count_vowels("lizard")**

2

>>> **count_vowels("zzzzzzz")**

0

>>>
```

当你定义一个变量时，可以通过将该变量设置为函数调用的返回值来初始化它：

```
>>> **num_vowels_think = count_vowels("THINK")**

>>> **num_vowels_lizard = count_vowels("lizard")**
```

这段代码定义了变量 num_vowels_think，并将其值设置为 count_vowels("THINK") 的返回值，或者说是字符串 THINK 中的元音字母数量。它还定义了变量 num_vowels_lizard，并将其值设置为 count_vowels("lizard") 的返回值。

然后，你可以使用这些变量来定义新变量：

```
>>> **total_vowels = num_vowels_think + num_vowels_lizard**

>>> **print(total_vowels)**

3
```

这段代码将这两个变量相加，并将它们的和保存在一个名为total_vowels的新变量中。然后，它会将total_vowels的值打印到终端。

当return语句执行时，函数会立即结束，因此return在你想提前停止函数时也非常有用。例如，下面的is_exciting()函数会循环遍历字符串s中的所有字符，以检查该字符是否为感叹号：

```
def is_exciting(s):

    for character in s:

        if character == "!":

            return True

    return False
```

如果函数找到感叹号，它会返回True，并立即停止函数。如果检查每个字符时没有找到感叹号，它将返回False。例如，如果你调用这个函数并传入字符串!@#$，函数将在循环的第一次迭代中返回True并立即结束——它甚至不会进入第二次迭代。如果你传入字符串hello!，它直到循环的最后一次迭代才返回True，因为它要到字符串的末尾才能找到!。如果你传入字符串goodbye，它会循环遍历整个字符串，但找不到感叹号，所以会返回False。

#### 文档字符串

在*自文档化*代码中，文档作为文档字符串被定义为代码的一部分，而不是单独的文档。*文档字符串*是被三重双引号（"""）或三重单引号（'''）包围的字符串，位于函数定义后的第一行代码中。当你运行函数时，程序会忽略文档字符串，但 Python 可以在需要时使用它来提取有关函数的文档。文档字符串是可选的，但它们可以帮助其他人理解你的代码。

例如，下面是如何使用文档字符串定义<code class="SANS_TheSansMonoCd_W5Regular_11">sum()</code>函数的方式：

```
>>> **def sum(a, b):**

...     **"""This function returns the sum of a and b"""**

...     **return a + b**
```

这与在示例 7-2 中定义的sum()函数完全相同，只不过它包含了一个文档字符串。

如果你运行help()函数，并将一个函数的名称（不带参数）作为参数传入，Python 解释器将显示该函数的文档。例如，运行help(sum)将显示以下输出：

```
Help on function sum in module __main__:

sum(a, b)

    This function returns the sum of a and b
```

help()函数适用于任何函数，但只有在编写该函数的程序员包含了文档字符串时才有用。在本例中，它告诉你，它正在显示名为sum()的函数的帮助文档，该函数位于__main__模块中。你将在第八章中了解更多关于模块的内容，但它们本质上是你自己编写的函数。尝试运行help(print)或help(len)，查看print()和len()函数的文档字符串。

按 Q 退出帮助界面，返回到 Python 解释器。

### 练习 7-5：练习编写函数

在本练习中，你将把在练习 7-4 中编写的脚本转换为一个函数。然后，你可以多次调用此函数，传入文本，以便它每次都返回该文本的交替大小写版本。

在你的文本编辑器中，创建一个名为*exercise-7-5.py*的新文件，并创建一个名为alternating_caps()的新函数，该函数接收参数text，如下所示：

```
def alternating_caps(text):

    """Returns an aLtErNaTiNg cApS version of text"""
```

接下来，复制练习 7-4 中的代码并将其粘贴到此函数中，确保将其缩进，使其与文档字符串对齐。删除定义text值的那一行；改为将text作为参数传递给函数来定义它。同时，将练习 7-4 代码中的最后一行从print(alternating_caps_text)更改为return alternating_caps_text。这个函数不应该显示字符串的交替大小写版本；它应该创建一个包含该版本的变量并返回它。

你的完整函数应该如下所示（你也可以在[*https://<wbr>github<wbr>.com<wbr>/micahflee<wbr>/hacks<wbr>-leaks<wbr>-and<wbr>-revelations<wbr>/blob<wbr>/main<wbr>/chapter<wbr>-7<wbr>/exercise<wbr>-7<wbr>-5<wbr>.py*](https://github.com/micahflee/hacks-leaks-and-revelations/blob/main/chapter-7/exercise-7-5.py)找到一个副本）：

```
def alternating_caps(text):

    """Returns an aLtErNaTiNg cApS version of text"""

    alternating_caps_text = " "

    should_be_capital = True

    for character in text:

        if should_be_capital:

            alternating_caps_text += character.upper()

            should_be_capital = False

        else:

            alternating_caps_text += character.lower()

            should_be_capital = True

    return alternating_caps_text
```

现在你有了一个函数——一个可重用的代码块——你可以根据需要调用它多次。像这样调用这个函数几次，并记得使用print()函数显示它的返回值：

```
print("Hacks, Leaks, and Revelations")

print(alternating_caps("This book is amazing"))

print(alternating_caps("I'm learning so much"))
```

你可以更改传递给alternating_caps()函数调用的文本，修改成你想要的任何内容。

这是我运行这个脚本时的效果：

```
micah@trapdoor chapter-7 % **python3 exercise-7-5.py**

Hacks, Leaks, and Revelations

ThIs bOoK Is aMaZiNg

I'M LeArNiNg sO MuCh
```

虽然这个脚本的输出是以嘲讽的语气显示的，但我希望这个情感对你来说是真实的！

### 总结

本章涵盖了你在未来研究中将依赖的几个基本的 Python 编程概念。你学会了编写包含语言主要特性的简单 Python 脚本，包括变量、if语句、for循环和函数。你已经准备好在下一章继续你的 Python 编程之旅，这次将编写代码直接调查数据集。
