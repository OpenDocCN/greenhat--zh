## 第二章：查找回文短语**

![image](img/common01.jpg)

雷达、皮划艇、回转器、性别。这些词有什么共同点？它们是*回文*，即正着读和反着读一样的词。更棒的是*回文短语*，即整个短语也具有相同的特性。拿破仑就是最著名回文短语的作者。当他第一次看到厄尔巴岛时，作为流放地，他说：“Able was I ere I saw Elba。”

2011 年，DC Comics 出版了一个有趣的故事，巧妙地利用了回文短语。超级英雄女巫扎塔娜被下了诅咒，只有通过回文方式才能施展魔法。她想出了一些足够用的两字短语，比如*nurses run*、*stack cats*和*puff up*，成功击败了持剑的攻击者。这让我想知道：到底有多少“具有攻击性”的回文短语？是否有更好的选择适合扎塔娜？

在本章中，你将从互联网上加载词典文件，并使用 Python 首先发现回文，然后在这些文件中发现更复杂的回文短语。接着，你将使用一个名为`cProfile`的工具分析回文短语代码，从而使其更加高效。最后，你将筛选回文短语，看看有多少个具有“攻击性”特征。

### **查找和打开词典**

本章的所有项目都需要一个以文本文件格式列出的单词表，通常称为*词典文件*，所以我们从学习如何加载一个词典文件开始。

尽管名为“词典文件”，这些文件仅包含单词——没有发音、音节数量、定义等信息。这是个好消息，因为这些内容反而会妨碍我们。更棒的是，词典文件可以在线免费获取。

你可以在表 2-1 中列出的地址找到合适的词典文件。下载其中一个文件，或者如果文件直接打开，复制并粘贴内容到文本编辑器（如记事本或 WordPad，macOS 上的 TextEdit）中，并将其保存为*.txt*文件。将词典文件与 Python 代码保存在同一文件夹中。我使用了*2of4brif.txt*文件来准备这个项目。它可以在表 2-1 中列出的第一个网站上下载的*12dicts-6.0.2.zip*文件中找到。

**表 2-1：** 可下载的词典文件

| **文件** | **单词数量** |
| --- | --- |
| *[`wordlist.aspell.net/12dicts/`](http://wordlist.aspell.net/12dicts/)* | 60,388 |
| *[`inventwithpython.com/dictionary.txt`](https://inventwithpython.com/dictionary.txt)* | 45,000 |
| *[`www-personal.umich.edu/~jlawler/wordlist.html`](http://www-personal.umich.edu/~jlawler/wordlist.html)* | 69,903 |
| *[`greenteapress.com/thinkpython2/code/words.txt`](http://greenteapress.com/thinkpython2/code/words.txt)* | 113,809 |

除了表 2-1 中的文件外，Unix 及类 Unix 操作系统通常还会附带一个以换行符分隔的包含超过 20 万个单词的大型字典文件。该文件通常存储在*/usr/share/dict/words*或*/usr/dict/words*中。在 Debian GNU/Linux 中，单词列表位于*/usr/share/opendict/dictionaries*。macOS 字典通常位于*/Library/Dictionaries*，并且包含非英语字典。如果你想使用这些文件，你可能需要在线搜索你的操作系统和版本，找到确切的目录路径。

一些字典文件会排除*a*和*I*作为单词。其他字典可能将每个字母作为一个单独的“头词”包括在字典中（例如以*d*开头的单词会以*d*作为头词）。在这些项目中，我们将忽略一字母回文，因此这些问题不应成为困扰。

#### ***处理打开文件时的异常***

每当你加载外部文件时，你的程序应该自动检查输入/输出问题，比如缺失文件或文件名错误，并在出现问题时通知你。

使用以下`try`和`except`语句来捕获和处理在执行过程中检测到的*异常*：

```py
➊ try:
    ➋ with open(file) as in_file:
           do something
   except IOError➌ as e:
    ➍ print("{}\nError opening {}. Terminating program.".format(e, file),
              file=sys.stderr)   
    ➎ sys.exit(1)
```

`try`语句首先执行 ➊。`with`语句会在嵌套代码块执行完毕后自动关闭文件，不管该块是如何退出的 ➋。在结束进程前关闭文件是一种良好的实践。如果不关闭文件，可能会用完文件描述符（这通常是长时间运行的大脚本的问题）、在 Windows 中锁定文件使其无法进一步访问、损坏文件，或者在写入文件时丢失数据。

如果出现问题，且错误类型与`except`关键字后面指定的异常类型匹配 ➌，则跳过剩余的`try`语句，执行`except`语句 ➍。如果没有出现问题，则执行`try`语句并跳过`except`语句。`except`语句中的`print`语句会让你知道出现了问题，而`file=sys.stderr`参数会在 IDLE 解释器窗口中将错误信息显示为红色。

`sys.exit(1)` ➎语句用于终止程序。`sys.exit(1)`中的`1`表示程序遇到错误并未成功关闭。

如果发生一个*未命名的异常*，并且它不匹配`except`语句中的命名异常，它会被传递给任何外部的`try`语句或主程序执行。如果没有找到处理程序，*未处理的异常*将导致程序停止并显示标准的“回溯”错误信息。

#### ***加载字典文件***

列表 2-1 将字典文件作为列表加载。你可以手动输入这个脚本，或者从*[`nostarch.com/impracticalpython/`](https://nostarch.com/impracticalpython/)*下载它作为*load_dictionary.py*。

你可以将这个文件作为模块导入到其他程序中，并通过一行语句运行它。记住，模块只是一个可以在另一个 Python 程序中使用的 Python 程序。正如你可能已经知道的，模块代表了一种*抽象*。抽象意味着你不必担心所有的编码细节。抽象的一个原则是*封装*，即隐藏细节。我们将文件加载代码封装在一个模块中，这样你就不需要看到或担心其他程序中的详细代码。

*load_dictionary.py*

```py
   """Load a text file as a list.

   Arguments:
   -text file name (and directory path, if needed)

   Exceptions:
   -IOError if filename not found.

   Returns:
   -A list of all words in a text file in lower case.

   Requires-import sys

   """
➊ import sys

➋ def load(file):
       """Open a text file & return a list of lowercase strings."""
       try:
           with open(file) as in_file:
            ➌ loaded_txt = in_file.read().strip().split('\n')
            ➍ loaded_txt = [x.lower() for x in loaded_txt]
               return loaded_txt
       except IOError as e:
        ➎ print("{}\nError opening {}. Terminating program.".format(e, file),
               file=sys.stderr)
           sys.exit(1)
```

*清单 2-1：将字典文件加载为列表的模块*

在文档字符串之后，我们导入了系统函数`sys`，以便我们的错误处理代码可以正常工作 ➊。接下来的代码块定义了一个基于之前文件打开讨论的函数 ➋。该函数以文件名作为参数。

如果没有抛出异常，文本文件的空白字符会被去除，其内容将被拆分成单独的行并添加到列表中 ➌。我们希望每个单词在返回之前都成为列表中的一个独立项。而且由于大小写对 Python 具有重要意义，列表中的单词会通过*列表推导式* ➍转换为小写。列表推导式是一种简写方式，用于将列表或其他可迭代对象转换为另一个列表。在这种情况下，它替代了`for`循环。

如果遇到 I/O 错误，程序会显示标准的错误信息，由`e`表示，并附带描述事件的消息，告知用户程序即将结束 ➎。然后，`sys.exit(1)`命令终止程序。

这个代码示例是为了说明这些步骤如何一起工作。通常情况下，你不会在模块中调用`sys.exit()`，因为你可能希望你的程序在终止之前做一些事情——比如写入日志文件。在后面的章节中，我们会将`try`-`except`块和`sys.exit()`都移到`main()`函数中，以便更清晰地控制。

### **项目 #2：寻找回文**

你将从在字典中找到单个单词的回文开始，然后转向更困难的回文短语。

**目标**

使用 Python 在英文词典文件中搜索回文。

#### ***策略与伪代码***

在你进入代码之前，先退后一步，思考一下你想要做的事情。从概念上讲，识别回文是很简单的：只需将一个单词与其反向切片后的结果进行比较。以下是一个将单词从前到后切片然后再从后到前切片的例子：

```py
>>> word = 'NURSES'
>>> word[:]
'NURSES'
>>> word[::-1]
'SESRUN'
```

如果你在切片字符串（或任何可切片类型）时不提供值，默认会使用字符串的起始位置、结束位置和步长为`1`。

图 2-1 展示了反向切片过程。我提供了起始位置为`2`，步长为`-1`。由于没有提供结束索引（冒号之间没有索引或空格），意味着要反向切片（因为索引步长是`-1`），直到没有更多字符可用。

![image](img/f0023-01.jpg)

*图 2-1：负切片的示例，针对* word = 'NURSES'

负切片的行为与正切片不完全相同，正负位置值和端点是不对称的。这可能会导致混淆，因此让我们将负切片限制为简单的 `[::-1]` 格式。

在词典中查找回文所需的代码行数比加载词典文件还少！这是伪代码：

```py
Load digital dictionary file as a list of words
Create an empty list to hold palindromes
Loop through each word in the word list:
    If word sliced forward is the same as word sliced backward:
        Append word to palindrome list
Print palindrome list
```

#### ***回文代码***

清单 2-2，*palindromes.py*，读取英文词典文件，识别哪些单词是回文，将它们保存到列表中，并将列表以堆叠的形式打印出来。你可以从本书的资源页面下载此代码，网址为 *[`www.nostarch.com/impracticalpython/`](https://www.nostarch.com/impracticalpython/)*。你还需要 *load_dictionary.py* 和一个词典文件；将所有三个文件保存在同一文件夹中。

*palindromes.py*

```py
   """Find palindromes (letter palingrams) in a dictionary file."""
➊ import load_dictionary
➋ word_list = load_dictionary.load('2of4brif.txt')
➌ pali_list = []

➍ for word in word_list:
       if len(word) > 1 and word == word[::-1]:
           pali_list.append(word)

   print("\nNumber of palindromes found = {}\n".format(len(pali_list)))
➎ print(*pali_list, sep='\n')
```

*清单 2-2：查找加载的词典文件中的回文*

首先，通过将 *load_dictionary.py* 作为模块导入 ➊。注意，导入时不使用 *.py* 扩展名。此外，由于该模块与此脚本在同一文件夹中，因此我们无需指定模块的目录路径。并且由于模块已经包含了所需的 `import sys` 语句，我们不需要在此重复它。

为了用词典中的单词填充我们的单词列表，可以通过点符号 ➋ 调用 `load()` 函数，来自 `load_dictionary` 模块。将外部词典文件的名称传递给它。如果词典文件与 Python 脚本在同一文件夹中，则不需要指定路径。你使用的文件名可能会根据你下载的词典有所不同。

接下来，创建一个空列表来保存回文 ➌，并开始循环遍历 `word_list` 中的每个单词 ➍，将正向切片与反向切片进行比较。如果这两个切片相同，则将该单词添加到 `pali_list`。注意，只有长度大于一的单词才被允许（`len(word) > 1`），这符合回文的最严格定义。

最后，以一种美观的方式打印回文——堆叠显示且没有引号或逗号 ➎。你可以通过循环遍历列表中的每个单词来完成这项工作，但有一种更高效的方式。你可以使用 *拆包* 操作符（由 `*` 表示），它将列表作为输入，并将其扩展为函数调用中的位置参数。最后一个参数是多个列表值之间的分隔符。默认分隔符是空格（`sep=' '`），但你可以改为将每个项目打印在新的一行上（`sep='\n'`）。

单词回文很少，至少在英语中是这样的。使用一个包含 60,000 个单词的词典文件，你最多可能会找到约 60 个回文单词，约占所有单词的 0.1%。尽管它们很稀少，但使用 Python 查找它们还是很容易的。那么，让我们继续更有趣、更复杂的回文组合。

### **项目 #3：寻找回文组合**

查找回文词对比查找单词回文需要更多的努力。在本节中，我们将规划并编写代码来查找词对回文。

**目标**

使用 Python 搜索英语词典中的双词回文。使用 cProfile 工具分析和优化回文代码。

#### ***策略与伪代码***

例子词对回文有*nurses run*和*stir grits*。（如果你在想，grits 是一种玉米早餐食品，类似于意大利的玉米粥。）

像回文一样，回文词对正反读都相同。我喜欢把这些看作是一个*核心*词，比如*nurses*，从中衍生出一个*回文序列*和*反向单词*（见图 2-2）。

![image](img/f0025-01.jpg)

*图 2-2：解析词对回文图*

我们的程序将分析核心词。基于图 2-2，我们可以对核心词做出以下推断：

1.  它可以是奇数或偶数个字母。

1.  单词的一个连续部分在反向读取时拼写出一个实际单词。

1.  这个连续部分可以占据核心词的一部分或全部。

1.  另一个连续部分包含一个回文序列的字母。

1.  回文序列可以占据核心词的一部分或全部。

1.  回文序列不必是实际的单词（除非它占据了*整个*单词）。

1.  这两部分不能重叠或共享字母。

1.  该序列是可逆的。

**注意**

*如果反向单词占据了整个核心词并且不是回文，它被称为*反向回文（semordnilap）*。反向回文与回文相似，除了一个关键的区别：它不是在反向读取时拼写出*相同*的单词，而是拼写出*不同*的单词。例子有*bats*和*stab*，还有*wolf*和*flow*。顺便说一下，semordnilap 就是回文反过来的拼写。*

图 2-3 表示一个任意的六个字母的单词。Xs 代表当单词反向阅读时，*可能*形成实际单词的部分（例如*run*在*nurses*中）。Os 代表*可能的*回文序列（例如*nurses*中的*ses*）。图 2-3 左栏中表示的单词像图 2-2 中的*nurses*一样，反向单词位于开头。右栏中的单词像*grits*一样，反向单词位于结尾。请注意，每一列的组合数是单词中字母的总数加一；还要注意，上下行表示的是相同的情况。

每列的顶部行表示一个反向词（semordnilap）。每列的底部行表示一个回文。它们都是反向单词，只是*类型*不同的反向单词。因此，它们被视为同一个实体，并且可以通过单行代码在单个循环中识别。

![image](img/f0026-01.jpg)

*图 2-3：六个字母核心单词中反向单词（X）和回文序列（O）的可能位置*

要查看图示的实际操作，考虑 图 2-4，其中展示了 palingrams *devils lived* 和 *retro porter*。*Devils* 和 *porter* 是两个核心单词，并且在回文序列和反向单词方面相互镜像。将其与倒序词 *evil* 和回文 *kayak* 进行比较。

![image](img/f0027-01.jpg)

*图 2-4：单词中的反向单词（X）和回文序列（O），倒序词和回文。*

回文既是反向单词 *也是* 回文序列。由于它们具有与倒序词相同的 X 模式，因此可以使用与倒序词相同的代码进行处理。

从策略角度来看，你需要遍历字典中的每个单词，并根据 图 2-3 中的 *所有组合* 进行评估。假设字典包含 60,000 个单词，程序大约需要进行 500,000 次循环。

要理解循环，看看 图 2-5 中的核心单词 *stack cats*。你的程序需要从单词的结尾字母开始，逐步加入字母。在寻找像 *stack cats* 这样的 palingram 时，它将同时评估单词末尾是否存在回文序列 *stack*，并且在开始处是否有反向单词。注意， 图 2-5 中的第一个循环会成功，因为单个字母 (*k*) 在这种情况下可以作为回文。

![image](img/f0027-02.jpg)

*图 2-5：示例循环遍历核心单词，同时寻找回文和反向单词*

但你还没有完成。为了捕捉 图 2-3 中的“镜像”行为，你需要反向运行循环，寻找单词开头的回文序列和结尾的反向单词。这将使你能够找到像 *stir grits* 这样的 palingram。

下面是一个找到 palingram 的伪代码：

```py
Load digital dictionary as a list of words
Start an empty list to hold palingrams
For word in word list:
    Get length of word
    If length > 1:
        Loop through the letters in the word:
            If reversed word fragment at front of word is in word list and letters
            after form a palindromic sequence:
               Append word and reversed word to palingram list
            If reversed word fragment at end of word is in word list and letters
            before form a palindromic sequence:
               Append reversed word and word to palingram list
Sort palingram list alphabetically
Print word-pair palingrams from palingram list
```

#### ***Palingrams 代码***

清单 2-3，*palingrams.py*，遍历单词列表，识别哪些单词形成单词对 palingram，保存这些对到一个列表中，并将列表作为堆叠项目打印出来。你可以从 *[`www.nostarch.com/impracticalpython/`](https://www.nostarch.com/impracticalpython/)* 下载代码。我建议你使用 *2of4brif.txt* 字典文件开始，这样你的结果会和我的一致。将字典文件和 *load_dictionary.py* 存放在与 palingrams 脚本相同的文件夹中。

*palingrams.py*

```py
   """Find all word-pair palingrams in a dictionary file."""

   import load_dictionary

   word_list = load_dictionary.load('2of4brif.txt')

   # find word-pair palingrams

➊ def find_palingrams():

       """Find dictionary palingrams."""

       pali_list = []

       for word in word_list:

        ➋ end = len(word)

        ➌ rev_word = word[::-1]

        ➍ if end > 1:

            ➎ for i in range(end):

                ➏ if word[i:] == rev_word[:end-i] and rev_word[end-i:] in word_list:

                       pali_list.append((word, rev_word[end-i:]))

                ➐ if word[:i] == rev_word[end-i:] and rev_word[:end-i] in word_list:

                       pali_list.append((rev_word[:end-i], word))

    ➑ return pali_list

➒ palingrams = find_palingrams()

   # sort palingrams on first word

   palingrams_sorted = sorted(palingrams)

   # display list of palingrams

➓ print("\nNumber of palingrams = {}\n".format(len(palingrams_sorted)))

   for first, second in palingrams_sorted:

       print("{} {}".format(first, second))
```

*清单 2-3：在加载的字典中查找并打印单词对 palingram*

在使用*palindromes.py*代码加载词典文件后，定义一个函数来查找回文组合 ➊。使用函数可以让你在之后将代码隔离开来，并计算处理词典中所有单词所需的时间。

立即启动一个名为`pali_list`的列表，用于存储程序发现的所有回文组合。接下来，启动一个`for`循环来评估`word_list`中的单词。对于每个单词，找到它的长度并将其赋值给变量`end` ➋。单词的长度决定了程序用来切片单词的索引，从而查找每一个可能的反转单词-回文序列组合，如图 2-3 所示。

接下来，反向切片单词并将结果赋值给变量`rev_word` ➌。`word[::-1]`的替代方法是`''.join(reversed(word))`，一些人认为这种方法更具可读性。

由于你正在寻找单词对的回文组合，排除掉单字母单词 ➍。然后嵌套另一个`for`语句，遍历当前单词中的字母 ➎。

现在，运行一个条件判断，要求单词的后半部分是回文，并且前半部分是单词列表中反转的单词（换句话说，是真正的单词） ➏。如果单词通过测试，它将被添加到回文组合列表中，并紧接着是反转后的单词。

根据图 2-3，你知道需要重复条件判断，但要改变切片方向和单词顺序，以反转输出。换句话说，你必须捕获单词开头的回文序列，而不是结尾 ➐。返回回文组合列表以完成函数 ➑。

定义了函数后，调用它 ➒。由于在循环过程中添加到回文组合列表中的词典单词的顺序发生变化，回文组合将不会按字母顺序排列。因此，排序该列表，以确保词对中的第一个单词按字母顺序排列。打印列表的长度 ➓，然后将每个单词对单独打印在一行上。

按照现有的写法，*palingrams.py*在约 60,000 个单词的词典文件上运行大约需要三分钟。在接下来的部分中，我们将探讨这个长时间运行的原因，并看看我们能做些什么来解决这个问题。

#### ***回文组合分析***

*分析*是一个分析过程，它在程序执行时收集程序行为的统计数据——例如函数调用的数量和持续时间。分析是优化过程中的关键部分。它能告诉你程序中哪些部分占用了最多的时间或内存。这样，你就知道应该在哪些地方集中精力以提高性能。

##### **使用 cProfile 进行分析**

*配置文件*是一个度量输出——记录程序各部分执行的时长和频率。Python 标准库提供了一个方便的分析接口`cProfile`，它是一个适用于分析长时间运行程序的 C 扩展。

`find_palingrams()`函数中的某些内容可能导致*palingrams.py*程序的相对较长运行时间。为了确认，我们运行`cProfile`。

将以下代码复制到一个新文件中，命名为*cprofile_test.py*，并将其保存在与*palingrams.py*和字典文件相同的文件夹中。该代码导入了`cProfile`和 palingrams 程序，并对`find_palingrams()`函数进行`cProfile`分析——通过点符号调用。再次提醒，你无需指定*.py*扩展名。

```py
import cProfile
import palingrams
cProfile.run('palingrams.find_palingrams()')
```

运行*cprofile_test.py*，在它完成后（你会看到解释器窗口中的`>>>`），你应该会看到类似下面的内容：

```py
         62622 function calls in 199.452 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000  199.451  199.451 <string>:1(<module>)
        1  199.433  199.433  199.451  199.451 palingrams.py:7(find_palingrams)
        1    0.000    0.000  199.452  199.452 {built-in method builtins.exec}
    60388    0.018    0.000    0.018    0.000 {built-in method builtins.len}
     2230    0.001    0.000    0.001    0.000 {method 'append' of 'list' objects}
```

所有的循环、切片和搜索在我的机器上花费了 199.452 秒，但当然你的时间可能与我的不同。你还会获得一些内置函数的额外信息，且由于每个 palingram 都调用了内置的`append()`函数，你甚至可以看到找到的 palingrams 数量（2,230）。

**注意**

*最常见的运行* cProfile *方式是在解释器中直接运行。这可以让你将输出转储到文本文件，并通过网页查看器查看。更多信息，请访问* [`docs.python.org/3/library/profile.html`](https://docs.python.org/3/library/profile.html)。

##### **使用 time 进行性能分析**

另一种对程序进行计时的方法是使用`time.time()`，它返回一个*纪元时间戳*——自 1970 年 1 月 1 日 UTC 午夜 12 点以来的秒数（*Unix 纪元*）。将*palingrams.py*复制到一个新文件中，保存为*palingrams_timed.py*，并在文件顶部插入以下代码：

```py
import time
start_time = time.time()
```

现在去文件的末尾，添加以下代码：

```py
end_time = time.time()
print("Runtime for this program was {} seconds.".format(end_time - start_time))
```

保存并运行文件。你应该在解释器窗口的底部看到以下反馈——大约几秒钟：

```py
Runtime for this program was 222.73954558372498 seconds.
```

运行时间比之前长，因为你现在在评估整个程序，包括打印输出，而不仅仅是`find_palingrams()`函数。

与`cProfile`不同，`time`不提供详细的统计数据，但像`cProfile`一样，它可以在单个代码组件上运行。编辑你刚才运行的文件，将开始和结束时间语句（如下所示的粗体部分）移到我们长时间运行的`find_palingrams()`函数两侧。保持文件顶部和底部的`import`和`print`语句不变。

```py
start_time = time.time()
palingrams = find_palingrams()
end_time = time.time()
```

保存并运行文件。你应该在解释器窗口的底部看到以下反馈：

```py
Runtime for this program was 199.42786622047424 seconds.
```

这现在与使用`cProfile`时的初始结果相匹配。如果重新运行程序或使用不同的计时器，你不会得到完全相同的时间，但不要纠结于此。重要的是*相对*时间，用于指导代码优化。

#### ***Palingram 优化***

抱歉，三分钟的等待时间对我来说太长了。根据我们的性能分析结果，我们知道 `find_palingrams()` 函数占用了大部分处理时间。这可能与读取和写入列表、对列表进行切片或在列表中查找有关。使用其他数据结构（如元组、集合或字典）可能会加速该函数。特别是，使用 `in` 关键字时，集合比列表要快得多。集合使用哈希表进行非常快速的查找。通过哈希，文本字符串会被转换为独特的数字，这些数字比引用的文本要小得多，查找也更高效。而列表则需要对每个项进行线性搜索。

想象一下，如果你在家里找丢失的手机，你可以通过查看每个房间来模拟一个列表，直到在（俗话说的）最后一个地方找到它。但如果你模拟一个集合，你基本上可以从另一部电话拨打你的手机号码，听到铃声后直接走到正确的房间。

使用集合的一个缺点是，集合中项目的顺序不可控，并且不允许重复值。使用列表时，顺序被保留且允许重复，但查找速度较慢。幸运的是，我们不关心顺序或重复，因此集合是最佳选择！

Listing 2-4 是原始 *palingrams.py* 程序中的 `find_palingrams()` 函数，经过编辑，改用一个单词集合而不是一个单词列表。你可以在一个名为 *palingrams_optimized.py* 的新程序中找到它，你可以从 *[`www.nostarch.com/impracticalpython/`](https://www.nostarch.com/impracticalpython/)* 下载，或者如果你想自己检查新的运行时间，也可以直接对你的 *palingrams_timed.py* 版本做这些更改。

*palingrams_optimized.py*

```py
def find_palingrams():

    """Find dictionary palingrams."""

    pali_list = []

 ➊ words = set(word_list)

 ➋ for word in words:

        end = len(word)

        rev_word = word[::-1]

        if end > 1:

            for i in range(end):

             ➌ if word[i:] == rev_word[:end-i] and rev_word[end-i:] in words:

                    pali_list.append((word, rev_word[end-i:]))

             ➍ if word[:i] == rev_word[end-i:] and rev_word[:end-i] in words:

                    pali_list.append((rev_word[:end-i], word))

    return pali_list
```

*Listing 2-4: 使用集合优化的* find_palingrams() *函数*

只有四行代码需要更改。定义一个新的变量 `words`，它是 `word_list` 的集合 ➊。然后遍历集合 ➋，查找单词切片在这个集合中的成员资格 ➌➍，而不是像以前那样在列表中查找。

这是 *palingrams_optimized.py* 中 `find_palingrams()` 函数的新运行时间：

```py
Runtime for this program was 0.4858267307281494 seconds.
```

哇！从三分钟缩短到不到一秒！*这就是*优化！差异就在于数据结构。验证一个单词是否属于*列表*是让我们头疼的地方。

为什么我先展示了“错误”的做法？因为这就是现实世界中的情况。你先让代码运行起来，然后再进行优化。这是一个简单的例子，经验丰富的程序员从一开始就能做对，但它代表了优化的总体概念：先让它尽可能地工作，再去改进它。

### **dnE ehT**

你已经编写了用于查找回文和回文字符串的代码，使用 `cProfile` 对代码进行了性能分析，并通过使用适合任务的数据结构优化了代码。那么，我们在 Zatanna 面前表现如何？她有机会获胜吗？

在这里，我列出了一些在 *2of4brif* 字典文件中找到的“更具攻击性”的回文字符串——从意外的 *相同灌肠* 到严厉的 *躯干腐烂*，再到作为地质学家的我最喜欢的 *风化矿石*。

| 倾倒泥土 | 昏昏欲睡剑 | 相同灌肠 |
| --- | --- | --- |
| 腿部凝胶 | 牛仔裤挖掘 | 麻风病人排斥 |
| 冰雹鳗鱼 | 乳制品袭击 | 猛击哺乳动物 |
| 风化矿石 | 升起先生 | 锅不停 |
| 滑行放屁 | 躯干腐烂 | 天鹅啃咬 |
| 狼流 | 伙伴陷阱 | 坚果震惊 |
| 拍打伙伴 | 拍打小腿 | 旋钮撞击 |

### **进一步阅读**

《*Think Python, 2nd Edition*》（O’Reilly，2015），由 Allen Downey 编写，简洁明了地描述了哈希表及其为何如此高效。这也是一本很棒的 Python 参考书。

### **实践项目：字典清理**

互联网上可用的数据文件并不总是“即插即用”。你可能会发现，在将数据应用到项目之前，你需要稍微处理一下数据。如前所述，一些在线字典文件将每个字母作为一个单词。若要允许在回文字符串中使用一字母单词（例如 *acidic a*），这些字母将会造成问题。你可以通过直接编辑字典文本文件来删除它们，但这很繁琐且不划算。相反，编写一个简短的脚本，在字典加载到 Python 后删除这些单词。为了测试它是否有效，可以编辑你的字典文件，加入一些像 *b* 和 *c* 这样的单字母单词。有关解决方案，请参见附录，或在网上找到副本 (*dictionary_cleanup_practice.py*)，网址为 *[`www.nostarch.com/impracticalpython/`](https://www.nostarch.com/impracticalpython/)*。

### **挑战项目：递归方法**

在 Python 中，通常有不止一种方法解决问题。请查看 Khan Academy 网站上的讨论和伪代码（*[`www.khanacademy.org/computing/computer-science/algorithms/recursive-algorithms/a/using-recursion-to-determine-whether-a-word-is-a-palindrome/`](https://www.khanacademy.org/computing/computer-science/algorithms/recursive-algorithms/a/using-recursion-to-determine-whether-a-word-is-a-palindrome/)*）。然后重写 *palindrome.py* 程序，使其使用递归来识别回文。
