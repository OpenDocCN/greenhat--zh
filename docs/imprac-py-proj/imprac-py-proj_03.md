## 第四章：**解决字谜**

![image](img/common01.jpg)

*字谜* 是通过重新排列另一个单词的字母形成的单词。例如，*Elvis* 产生了令人毛骨悚然的三重奏 *evils*、*lives* 和 *veils*。这是否意味着 Elvis 仍然活着，但隐藏了他邪恶的存在？在《哈利·波特与密室》一书中，“I am Lord Voldemort” 是邪恶巫师真名 Tom Marvolo Riddle 的字谜。 “Lord Earldom Vomit” 也是 Tom Marvolo Riddle 的字谜，但作者 J.K. Rowling 很有眼光地放弃了这个版本。

在本章中，你将首先找到给定单词或姓名的所有字谜。然后，你将编写一个程序，让用户可以交互式地从自己的名字构建字谜短语。最后，你将成为计算机巫师，看看如何从 “Tom Marvolo Riddle” 中提取 “I am Lord Voldemort”。

### **项目 #4：查找单词字谜**

你将从分析简单的单词字谜开始，并搞清楚如何通过编程来识别它们。完成这个任务后，你就准备好在接下来的章节中处理字谜短语了。

**目标**

使用 Python 和字典文件查找给定英语单词或单一姓名的所有单词字谜。你可以在 第二章 开头阅读查找和加载字典文件的说明。

#### ***策略与伪代码***

超过 600 家报纸和 100 个网站都有一个叫做 *Jumble* 的字谜游戏。该游戏创建于 1954 年，现在是世界上最受欢迎的字谜游戏。*Jumble* 可能会非常让人沮丧，但找字谜几乎像找回文一样简单——你只需要知道所有字谜的共同特点：它们必须包含相同数量的相同字母。

##### **识别字谜**

Python 没有内建的字谜运算符，但你可以很容易地编写一个。在本章的项目中，你将从 第二章 加载字典文件作为字符串列表。因此，程序需要验证两个字符串是否为彼此的字谜。

让我们来看一个例子。*Pots* 是 *stop* 的字谜，你可以通过 `len()` 函数验证 *stop* 和 *pots* 的字母数相同。但 Python 无法知道两个字符串是否有相同数量的单个字符——至少在不将字符串转换为其他数据结构或使用计数函数的情况下无法知道。所以，与其简单地将这两个单词视为字符串，不如将它们表示为包含单个字符字符串的两个列表。你可以在 shell 中（例如 IDLE）创建这些列表，并将它们命名为 `word` 和 `anagram`，正如我在这里所做的那样：

```py
>>> word = list('stop')
>>> word
['s', 't', 'o', 'p']
>>> anagram = list('pots')
>>> anagram
['p', 'o', 't', 's']
```

这两个列表符合我们对字谜对的描述；也就是说，它们包含相同数量的相同字母。但如果你尝试用比较运算符 `==` 将它们相等，结果会是 `False`。

```py
>>> anagram == word
False
```

问题在于运算符（`==`）仅在两个列表具有相同数量的相同项且这些项按相同顺序排列时，才会认为它们是等价的。你可以通过内置函数`sorted()`轻松解决这个问题，它可以接受一个列表作为参数，并按字母顺序重新排序其内容。因此，如果你分别对这两个列表调用`sorted()`，然后比较排序后的列表，它们将是等价的。换句话说，`==`会返回`True`。

```py
>>> word = sorted(word)
>>> word
['o', 'p', 's', 't']
>>> anagram = sorted(anagram)
>>> anagram
['o', 'p', 's', 't']
>>> anagram == word
True
```

你也可以将一个字符串传递给`sorted()`函数来创建一个排序后的列表，类似于前面的代码片段。这对将字典文件中的单词转换为排序后的单字符字符串列表非常有用。

现在你已经知道如何验证是否找到一个变位词，让我们设计整个脚本——从加载字典和提示用户输入单词（或名字）到搜索并打印所有的变位词。

##### **使用伪代码**

记住，使用伪代码进行规划可以帮助你发现潜在的问题，早期发现问题可以节省时间。以下的伪代码应有助于你更好地理解我们将在下一节中编写的脚本*anagrams.py*。

```py
Load digital dictionary file as a list of words
Accept a word from user
Create an empty list to hold anagrams
Sort the user-word
Loop through each word in the word list:
    Sort the word
    if word sorted is equal to user-word sorted:
        Append word to anagrams list
Print anagrams list
```

该脚本将首先从字典文件中加载单词到一个字符串列表中。在开始遍历字典查找变位词之前，你需要知道你想要查找变位词的单词，并且需要一个地方来存储找到的变位词。所以，首先要求用户输入一个单词，然后创建一个空列表来存储变位词。一旦程序遍历完字典中的每个单词，它将打印出变位词列表。

#### ***变位词查找器代码***

清单 3-1 加载一个字典文件，接受程序中*指定的*单词或名字，并查找该单词或名字在字典文件中的所有变位词。你还需要在第二章中获取字典加载的代码。你可以从*[`www.nostarch.com/impracticalpython/`](https://www.nostarch.com/impracticalpython/)*下载这两个文件，分别为*anagrams.py*和*load_dictionary.py*。请将这两个文件保存在同一文件夹内。你可以使用在第二章中使用的相同字典文件，或者下载另一个文件（有关建议，请参见表格 2-1，位于第 20 页）。

*anagrams.py*

```py
➊ import load_dictionary

➋ word_list = load_dictionary.load('2of4brif.txt')

➌ anagram_list = []

   # input a SINGLE word or SINGLE name below to find its anagram(s):
➍ name = 'Foster'
   print("Input name = {}".format (name))
➎ name = name.lower()
   print("Using name = {}".format(name))

   # sort name & find anagrams
➏ name_sorted = sorted(name)
➐ for word in word_list:
       word = word.lower()
       if word != name:
           if sorted(word) == name_sorted:
               anagram_list.append(word)

   # print out list of anagrams
   print()
➑ if len(anagram_list) == 0:
       print("You need a larger dictionary or a new name!")
   else:
    ➒ print("Anagrams =", *anagram_list, sep='\n')
```

*清单 3-1：给定一个单词（或名字）和一个字典文件，该程序会搜索并打印出一个包含所有变位词的列表。*

你从导入你在第二章中创建的`load_dictionary`模块开始 ➊。这个模块将打开一个字典文本文件，并通过其`load()`函数将所有单词加载到一个列表中 ➋。你使用的**.txt**文件可能不同，具体取决于你下载了哪个字典文件（参见“查找并打开字典”，位于第 20 页）。

接下来，创建一个空列表，命名为`anagram_list`，用于存储找到的任何字谜 ➌。让用户添加一个*单独的*单词，例如他们的名字 ➍。这不一定是一个正式的名字，但我们将在代码中称其为`name`，以便与字典中的`word`区分开。打印这个名字，以便用户能看到输入了什么内容。

下一行预见到了一个可能出现的问题用户行为。人们往往会输入以大写字母开头的名字，但字典文件可能不包括大写字母，而这对 Python 来说是有影响的。因此，首先使用`.lower()`字符串方法将所有字母转换为小写 ➎。

现在对名字进行排序 ➏。如前所述，你可以将`sorted()`传递给一个字符串，而不仅仅是一个列表。

输入按字母顺序排列在列表中后，接下来是查找字谜。开始遍历字典单词列表中的每个单词 ➐。为了安全起见，将单词转换为小写字母，因为比较操作是区分大小写的。转换后，将单词与未排序的名称进行比较，因为一个单词不能是它自己的字谜。接下来，对字典单词进行排序，并与排序后的名称进行比较。如果匹配，则将该字典单词添加到`anagram_list`中。

现在显示结果。首先，检查字谜列表是否为空。如果为空，打印一个俏皮的回复，这样就不会让用户空等 ➑。如果程序找到了至少一个字谜，则使用拆包（`*`）操作符打印列表。记住在第二章中提到的，拆包让你将列表中的每个成员打印在单独的一行 ➒。

以下是该程序的示例输出，使用输入名字*Foster*：

```py
Input name = Foster
Using name = foster

Anagrams =
forest
fortes
softer
```

如果你想使用其他输入，可以在源代码中更改`name`变量的值。作为练习，尝试调整代码，使程序提示用户输入名字（或单词）；你可以使用`input()`函数实现这一点。

### **项目#5：寻找短语字谜**

在上一个项目中，你将一个名字或单词的字母重新排列，找到单一单词的字谜。现在，你将从一个名字中推导出多个单词。这些*短语字谜*只包含输入名称的一部分字母，因此你需要几个单词来用尽所有字母。

**目标**

编写一个 Python 程序，让用户通过互动从名字中的字母构建字谜短语。

#### ***策略与伪代码***

最棒的短语字谜是那些描述与名字拥有者相关的著名特征或行为的。例如，“Clint Eastwood”的字母可以重新排列形成*old west action*，“Alec Guinness”可以变成*genuine class*，“Madam Curie”变为*radium came*，“George Bush”变为*he bugs Gore*，而“Statue of Liberty”则包含*built to stay free*。我自己的名字重新排列后得出*a huge navel*，这其实并不是我的特点。

到此时，你可能会看到一个战略性的挑战：计算机如何处理上下文内容？发明了 Watson 的 IBM 团队似乎知道答案，但对于我们其他人来说，这块巨石有点难以搬动。

*暴力破解法*是在线异位词生成器中常用的一种方法。这些算法接收一个名字并返回大量随机的异位词短语（通常是 100 个到 1 万多个）。大多数返回的短语是无意义的，滚动浏览这些内容可能是件繁琐的事。

另一种方法是认识到人类在处理上下文问题时最为擅长，因此编写一个程序帮助人类解决问题。计算机可以获取初始名称并提供从其中一些（或所有）字母组成的单词；然后用户可以选择一个“合适”的单词。程序随后将根据名称中剩余字母重新计算单词选项，直到每个字母都被使用或所有可能的单词选项耗尽。这个设计发挥了双方的优势。

你需要一个简单的界面，提示用户输入初始名称，展示潜在的单词选项，并显示剩余的字母。程序还需要跟踪正在生成的异位词短语，并在每个字母都被使用时通知用户。由于很可能会有许多失败的尝试，所以界面应该允许用户随时重新开始过程。

由于异位词具有相同数量的相同字母，另一种识别异位词的方法是统计每个字母的出现次数。如果你将你的名字看作一个字母集合，那么一个单词可以由你的名字构成，当且仅当（1）该单词的所有字母都出现在你的名字中，并且（2）它们的出现频率相同或更少。显然，如果*e*在一个单词中出现三次，而在你的名字中出现两次，那么这个单词不能由你的名字衍生出来。因此，如果构成一个单词的字母集合不是你名字中字母集合的子集，那么这个单词就不能是你名字的异位词。

##### **使用 Counter 来统计字母**

幸运的是，Python 自带了一个名为`collections`的模块，其中包含了几种容器数据类型。其中一种类型，`Counter`，用于统计项的出现次数。Python 将这些项存储为字典的键，将计数存储为字典的值。例如，下面的代码片段统计了在一个列表中每种盆栽树的数量。

```py
   >>> from collections import Counter
➊ >>> my_bonsai_trees = ['maple', 'oak', 'elm', 'maple', 'elm', 'elm', 'elm', 'elm']
➋ >>> count = Counter(my_bonsai_trees)
   >>> print(count)
➌ Counter({'elm': 5, 'maple': 2, 'oak': 1})
```

`my_bonsai_trees`列表包含了多种相同类型的树木 ➊。`Counter`统计这些树木 ➋，并创建一个便于引用的字典 ➌。请注意，`print()`函数是可选的，这里使用它是为了清晰展示。单独输入`count`也会显示字典内容。

你可以使用`Counter`，而不是`sorted()`方法，来查找单词异位词。输出将是两个字典，而不是两个排序后的列表，这两个字典也可以直接用`==`进行比较。下面是一个示例：

```py
   >>> name = 'foster'
   >>> word = 'forest'
   >>> name_count = Counter(name)
   >>> print(name_count)
➊ Counter({'f': 1, 't': 1, 'e': 1, 'o': 1, 'r': 1, 's': 1})
   >>> word_count = Counter(word)
   >>> print(word_count)
➋ Counter({'f': 1, 't': 1, 'o': 1, 'e': 1, 'r': 1, 's': 1})
```

`Counter` 为每个单词生成一个字典，将单词中的每个字母映射到它出现的次数➊➋。这些字典是无序的，但尽管没有排序，Python 仍能正确识别相同字母和相同计数的字典是相等的：

```py
>>> if word_count == name_count:
        print("It's a match!")

It's a match!
```

`Counter` 提供了一种很棒的方式来查找在名称中“适配”的单词。如果一个单词中每个字母的计数小于或等于该字母在名称中出现的计数，那么这个单词就可以从该名称中派生出来！

##### **伪代码**

我们现在做出了两个重要的设计决策：（1）让用户交互式地每次构建一个字谜；（2）使用 `Counter` 方法来查找字谜。这足以开始思考高层次的伪代码：

```py
Load a dictionary file
Accept a name from user
Set limit = length of name
Start empty list to hold anagram phrase
While length of phrase < limit:
    Generate list of dictionary words that fit in name
    Present words to user
    Present remaining letters to user
    Present current phrase to user
    Ask user to input word or start over
    If user input can be made from remaining letters:
        Accept choice of new word or words from user
        Remove letters in choice from letters in name
        Return choice and remaining letters in name
    If choice is not a valid selection:
        Ask user for new choice or let user start over
    Add choice to phrase and show to user
    Generate new list of words and repeat process
When phrase length equals limit value:
    Display final phrase
    Ask user to start over or to exit
```

##### **分配工作**

随着过程性代码变得越来越复杂，有必要将大部分代码封装到函数中。这使得管理输入和输出、执行递归以及阅读代码变得更加容易。

*主函数* 是程序开始执行的地方，并且使得高层次的组织变得可能，例如管理代码的各个部分，包括处理用户。在短语字谜程序中，主函数将包装所有“工作蜂”函数，处理*大部分*用户输入，跟踪逐渐生成的字谜短语，确定短语何时完成，并向用户显示结果。

用铅笔和纸草拟任务及其流程是一个很好的方法，可以帮助你理清想做什么以及在哪里做（类似于“图形伪代码”）。图 3-1 是一个流程图，突出显示了函数分配。在这种情况下，三个函数应该足够：`main()`、`find_anagrams()` 和 `process_choice()`。

`main()` 函数的主要任务是设置字母计数限制并管理负责通用短语字谜构建的 `while` 循环。`find_anagrams()` 函数将获取名称中剩余的当前字母集合，并返回可以由这些字母组成的所有可能单词。然后，单词将显示给用户，并且当前的短语由 `main()` 函数“拥有”并显示。接着，`process_choice()` 函数提示用户重新开始或选择一个字谜单词。如果用户做出选择，这个函数将判断所选字母是否可用。如果不可用，用户将被提示重新选择或重新开始。如果用户做出有效选择，所选字母将从剩余字母列表中移除，并且返回所选字和剩余字母列表。`main()` 函数将返回的选择添加到现有的短语中。如果达到限制，完成的短语字谜将显示出来，用户将被询问是否重新开始或退出。

注意，你在 *全局* 范围内请求初始名称，而不是在 `main()` 函数中。这让用户可以随时重新开始，而无需重新输入他们的名称。目前，如果用户想选择一个全新的名称，他们必须退出程序并重新开始。在第九章，你将使用一个菜单系统，允许用户完全重置他们的操作而无需退出。

![image](img/f0043-01.jpg)

*图 3-1：带有函数分配高亮的短语字谜流程图*

#### ***字谜短语代码***

本节的代码从用户那里获取一个名称，并帮助他们构建该名称的字谜短语。你可以从 *[`www.nostarch.com/impracticalpython/`](https://www.nostarch.com/impracticalpython/)* 下载整个脚本，保存为 *phrase_anagrams.py*。你还需要下载 *load_dictionary.py* 程序。将这两个文件保存在同一文件夹中。你可以使用在 “项目 #4：寻找单词字谜” 中使用的同一个字典文件，位于 第 36 页。

##### **设置和查找字谜**

清单 3-2 导入了 *phrase_anagrams.py* 使用的模块，加载了一个字典文件，询问用户输入一个名称，并定义了 `find_anagrams()` 函数，该函数完成了大部分与寻找字谜相关的工作。

*phrase_anagrams.py,* 第一部分

```py
➊ import sys
   from collections import Counter
   import load_dictionary

➋ dict_file = load_dictionary.load('2of4brif.txt')
   # ensure "a" & "I" (both lowercase) are included
   dict_file.append('a')
   dict_file.append('i')
   dict_file = sorted(dict_file)

➌ ini_name = input("Enter a name: ")

➍ def find_anagrams(name, word_list):
       """Read name & dictionary file & display all anagrams IN name."""
    ➎ name_letter_map = Counter(name)
       anagrams = []
    ➏ for word in word_list:
        ➐ test = ''
        ➑ word_letter_map = Counter(word.lower())
        ➒ for letter in word:
               if word_letter_map[letter] <= name_letter_map[letter]:
                   test += letter
           if Counter(test) == word_letter_map:
               anagrams.append(word)
    ➓ print(*anagrams, sep='\n')
       print()
       print("Remaining letters = {}".format(name))
       print("Number of remaining letters = {}".format(len(name)))
       print("Number of remaining (real word) anagrams = {}".format(len(anagrams)))
```

*清单 3-2：导入模块，加载字典，并定义* find_anagrams() *函数*

从 `import` 语句 ➊ 开始，按照推荐的顺序导入 Python 标准库、第三方模块，然后是本地开发的模块。你需要 `sys` 来在 IDLE 窗口中将特定的输出显示为红色，并让用户通过按键退出程序。你将使用 `Counter` 来帮助识别输入名称的字谜。

接下来，使用导入的模块 ➋ 加载字典文件。文件名参数应为你正在使用的字典的文件名。由于一些字典文件省略了 *a* 和 *I*，如果需要，可以将它们附加到字典中，并对列表进行排序，以便它们出现在正确的字母顺序位置，而不是出现在列表的末尾。

现在从用户那里获取一个名称，并将其分配给变量 `ini_name`（或“初始名称”） ➌。你将从这个初始名称派生出 `name` 变量，并随着用户逐步构建名称字谜，`name` 变量会不断变化。将初始名称保留为一个单独的变量将允许你在用户希望重新开始或再试一次时重置一切。

下一个代码块是 `find_anagrams()` ➍，用于在 `name` 中查找变位词的函数。该函数的参数包括一个 `name` 和一个单词列表。函数通过使用 `Counter` 来计算名字中给定字母出现的次数，并将计数分配给变量 `name_letter_map` ➎；`Counter` 使用字典结构，字母作为键，计数作为值。然后，函数创建一个空列表来保存变位词，并开始 `for` 循环遍历字典文件中的每个单词 ➏。

`for` 循环从创建一个名为 `test` 的空字符串开始 ➐。使用这个变量累积 `word` 中“适合” `name` 的所有字母。然后为当前单词创建一个 `Counter`，就像你为 `name` 所做的那样，并称其为 `word_``letter``_map` ➑。遍历 `word` 中的字母 ➒，检查每个字母的计数是否与 `name` 中的计数相同或更少。如果字母满足这个条件，则将其添加到 `test` 字符串中。由于某些字母可能会被拒绝，所以通过对 `test` 运行 `Counter` 并将其与 `word_letter_map` 进行比较来结束循环。如果它们匹配，则将该单词追加到变位词列表中。

函数以使用 `print` 和带有分散运算符的单词列表结尾，并为用户提供一些统计信息 ➓。请注意，`find_``anagrams()` 不返回任何内容。这是人际交互部分的地方。程序将继续运行，但在用户选择列表中的一个单词之前不会发生任何事情。

##### **处理用户的选择**

清单 3-3 定义了 `process_choice()` 函数，位于 *phrase_anagrams.py* 中，它接收用户选择的单词（或多个单词），检查它们与 `name` 变量中剩余的字母是否匹配，并将可接受的选择（连同任何剩余的字母）返回给 `main()` 函数。像 `main()` 一样，这个函数可以直接与用户交互。

*phrase_anagrams.py,* 第二部分

```py
➊ def process_choice(name):
       """Check user choice for validity, return choice & leftover letters."""
       while True:
        ➋ choice = input('\nMake a choice else Enter to start over or # to end: ')
           if choice == '':
               main()
           elif choice == '#':
               sys.exit()
           else:
            ➌ candidate = ''.join(choice.lower().split())
        ➍ left_over_list = list(name)
        ➎ for letter in candidate:
               if letter in left_over_list:
                   left_over_list.remove(letter)
        ➏ if len(name) - len(left_over_list) == len(candidate):
               break
           else:
               print("Won't work! Make another choice!", file=sys.stderr)
    ➐ name = ''.join(left_over_list)  # makes display more readable
    ➑ return choice, name
```

*清单 3-3：定义了* `process_choice()` *函数*

从定义只有一个名为 `name` 的参数的函数开始 ➊。第一次运行程序时，这个参数将与 `ini_name` 变量相同，即用户启动程序时输入的完整姓名。用户选择用于变位词短语的单词（或多个单词）后，它将表示姓名中剩余的字母。

使用一个 `while` 循环开始函数，直到用户做出有效选择为止，并从用户那里获取输入 ➋。用户可以选择从当前变位词列表中输入一个或多个单词，按 ENTER 重新开始，或按 # 退出。使用 # 而不是单词或字母，以避免与有效选择混淆。

如果用户做出选择，将字符串赋给变量 `candidate`，去除空白并转换为全小写 ➌。这样可以直接与 `name` 变量进行比较。之后，从 `name` 变量中构建一个列表以保存任何剩余的字母 ➍。

现在开始一个循环，减去 `candidate` ➎ 中使用的字母。如果选中的字母存在于列表中，它将被移除。

如果用户输入了一个不在显示列表中的单词，或输入了多个单词，可能会导致某些字母不在列表中。为检查这一点，可以从 `name` 中减去剩余字母，如果结果与 `candidate` 中字母的数量相等，则认为输入有效，并跳出 `while` 循环 ➏。否则，显示警告并将其标红，以便在使用 IDLE 窗口的用户看到。`while` 循环将继续提示用户，直到做出有效选择。

如果用户选择的所有字母都通过了测试，剩余字母的列表将被转换回字符串，并用于更新 `name` 变量 ➐。将列表转换为字符串并非绝对必要，但这样可以保持 `name` 变量类型一致，并且使你能够清晰地显示剩余字母，无需额外的 `print` 参数。

最后，将用户的选择和剩余字母的字符串（`name`）一起返回给 `main()` 函数 ➑。

##### **定义 main() 函数**

列出 3-4 定义了 *phrase_anagrams.py* 中的 `main()` 函数。这个函数封装了之前的函数，运行一个 `while` 循环，并决定用户何时成功创建字谜短语。

*phrase_anagrams.py，* 第三部分

```py
   def main():
       """Help user build anagram phrase from their name."""
    ➊ name = ''.join(ini_name.lower().split())
       name = name.replace('-', '')
    ➋ limit = len(name)
       phrase = ''
       running = True

    ➌ while running:
        ➍ temp_phrase = phrase.replace(' ', '')
        ➎ if len(temp_phrase) < limit:
               print("Length of anagram phrase = {}".format(len(temp_phrase)))

            ➏ find_anagrams(name, dict_file)
               print("Current anagram phrase =", end=" ")
               print(phrase, file=sys.stderr)

            ➐ choice, name = process_choice(name)
               phrase += choice + ' '

        ➑ elif len(temp_phrase) == limit:
               print("\n*****FINISHED!!!*****\n")
               print("Anagram of name =", end=" ")
               print(phrase, file=sys.stderr)
               print()
            ➒ try_again = input('\n\nTry again? (Press Enter else "n" to quit)\n ')
               if try_again.lower() == "n":
                   running = False
                   sys.exit()
               else:
                   main()

➓ if __name__ == '__main__':
       main()
```

*列出 3-4：定义并调用* main() *函数*

首要任务是将 `ini_name` 变量转化为一个连续的小写字母字符串，并且没有空格 ➊。记住，大小写对 Python 很重要，因此需要在每次出现字符串时都转换为小写；这样，比较才能按预期工作。Python 还将空格视为字符，所以在进行字母计数之前，需要移除空格和连字符。通过声明这个新的 `name` 变量，你可以保留初始名称，以防用户想要重新开始。只有 `name` 会在 `process_choice()` 函数中被修改。

接下来，获取名称 ➋ 的长度，用作 `while` 循环的限制。这将帮助你确定当字谜短语使用完名称中的所有字母时，应该结束循环。在 `while` 循环外执行此操作，以确保你使用的是完整的初始名称。然后，分配一个变量来保存字谜短语，并将 `running` 变量设置为 `True`，以控制 `while` 循环。

现在开始一个大的循环，让你遍历名称并构建字谜短语 ➌。首先，准备一个字符串来保存增长中的短语，并去除空格 ➍。空格会被计算为字母，并在与 `limit` 变量比较短语长度时引起偏差。接下来，进行比较，如果短语的长度小于限制，显示当前短语的长度，作为与用户互动的序曲 ➎。

现在是时候让其他函数发挥作用了。调用`find_anagrams()` ➏并传入姓名和字典文件，以获取姓名中的字谜列表。在显示的列表底部，向用户展示当前的短语。使用`print()`函数的`end`参数，将两个`print`语句显示在同一行。这样，你可以在 IDLE 窗口中使用红色字体显示短语，将其与显示中的其他信息区分开来。

接下来，调用`process_choice()`函数 ➐获取用户的单词选择，并将其添加到逐步完成的字谜短语中。这还会获取更新后的`name`变量，以便程序在短语未完成时可以在`while`循环中再次使用它。

如果短语的长度等于`limit`变量 ➑，那么姓名的字谜就完成了。让用户知道他们完成了，并用红色字体显示该短语。请注意，你没有为短语长度大于`limit`变量的情况设置条件。这是因为`process_choice()`函数已经处理了这种情况（选择的字母多于可用字母时会未通过验证标准）。

`main()`函数通过询问用户是否想要再试一次来结束。如果用户输入`n`，程序结束；如果按下 ENTER 键，`main()`函数将再次被调用 ➒。如前所述，用户更改初始姓名的唯一方法是退出并重新启动程序。

在`main()`函数外，结束时使用标准的两行代码来调用`main()`函数，当程序未作为模块导入时 ➓。

##### **运行示例会话**

在这一部分，我包含了一个示例互动会话，使用*phrase_anagrams.py*和姓名*Bill Bo*。粗体字体表示用户输入，斜体粗体字体表示在显示中使用红色字体的地方。

```py
Enter a name: Bill Bo

Length of anagram phrase = 0

bib

bill

blob

bob

boil

boll

i

ill

lib

lilo

lo

lob

oi

oil

Remaining letters = billbo

Number of remaining letters = 6

Number of remaining (real word) anagrams = 14

Current anagram phrase =

Make a choice else Enter to start over or # to end: ill

Length of anagram phrase = 3

bob

Remaining letters = bbo

Number of remaining letters = 3

Number of remaining (real word) anagrams = 1

Current anagram phrase = ill

Make a choice else Enter to start over or # to end: Bob

***** FINISHED!!! *****

Anagram of name = ill Bob

Try again? (Press Enter else "n" to quit)
```

找到的字谜数量取决于你使用的字典文件。如果你在构建字谜短语时遇到困难，可以尝试使用更大的字典。

### **项目 #6：寻找伏地魔：高卢策略**

你有没有想过汤姆·里德尔是怎么想出字谜“I am Lord Voldemort”的？他是不是拿起羽毛笔写在羊皮纸上，还是只是挥舞了一下魔杖？Python 的魔力是否也有所帮助？

假设你是霍格沃茨的计算机魔法教授，汤姆·里德尔，学校的学长和模范学生，来找你寻求帮助。使用你在前一部分中提到的*phrase_anagrams.py*法术，他能够在第一个字谜列表中找到*I am Lord*，令他非常高兴。但剩下的字母*tmvoordle*只生成了像*dolt*、*drool*、*looter*和*lover*这样的琐碎单词。里德尔一定不太高兴。

回头看，问题很明显：*伏地魔*是法语词，无法在任何英语词典中找到。*Vol de la mort*在法语中意味着“死亡之翼”，因此伏地魔可以 loosely 理解为“死亡飞行”。但 Riddle 是百分之百的英语名字，而到目前为止，你一直在使用英语。除非进行逆向工程，否则你没有理由突然将英语词典换成法语词典，就像你没有理由去用荷兰语、德语、意大利语或西班牙语一样。

你*可以*尝试随机打乱剩余的字母，看看能得到什么。遗憾的是，可能的组合数是字母的阶乘除以重复字母的阶乘（*o*出现两次）：9! / 2! = 181,440。如果你要浏览所有这些排列，每个排列只花一秒钟时间，那么完成整个列表将需要超过两天！如果你让 Tom Riddle 来做这件事，他可能会用你来做魂器！

到此为止，我想探讨两条逻辑路径。其一我称之为“高卢战略”，另一条是“英国蛮力法”。我们将在这里讨论第一条，第二条将在下一节讨论。

**注意**

Marvolo *显然是一个*人为创造的词*，用来使伏地魔的字谜成立。J.K. Rowling 本可以通过使用*Thomas*代替*Tom*，或者省略*Lord*或*I am*部分，获得更多的自由。在书籍翻译成非英语语言时，常常使用这样的技巧。在某些语言中，一个或两个名字可能需要更改。在法语中，字谜是“I am Voldemort”。在挪威语中是“Voldemort the Great”。在荷兰语中是“My name is Voldemort”。在其他语言中，如中文，根本无法使用字谜！*

Tom Riddle 痴迷于战胜死亡，如果你在*tmvoordle*中寻找死亡，你会发现既有古老的法语词*morte*（就像在托马斯·马洛里的著名书籍《亚瑟王之死》中的用法），也有现代法语词*mort*。去除*mort*后，剩下的就是*vodle*，五个字母，排列组合的数量非常可控。事实上，你可以轻松地在解释器窗口中找到*volde*：

```py
➊ >>> from itertools import permutations
   >>> name = 'vodle'
➋ >>> perms = [''.join(i) for i in permutations(name)]
➌ >>> print(len(perms))
   120
➍ >>> print(perms)
   ['vodle', 'vodel', 'volde', 'voled', 'voedl', 'voeld', 'vdole', 'vdoel',
   'vdloe', 'vdleo', 'vdeol', 'vdelo', 'vlode', 'vloed', 'vldoe', 'vldeo',
   'vleod', 'vledo', 'veodl', 'veold', 'vedol', 'vedlo', 'velod', 'veldo',
   'ovdle', 'ovdel', 'ovlde', 'ovled', 'ovedl', 'oveld', 'odvle', 'odvel',
   'odlve', 'odlev', 'odevl', 'odelv', 'olvde', 'olved', 'oldve', 'oldev',
   'olevd', 'oledv', 'oevdl', 'oevld', 'oedvl', 'oedlv', 'oelvd', 'oeldv',
   'dvole', 'dvoel', 'dvloe', 'dvleo', 'dveol', 'dvelo', 'dovle', 'dovel',
   'dolve', 'dolev', 'doevl', 'doelv', 'dlvoe', 'dlveo', 'dlove', 'dloev',
   'dlevo', 'dleov', 'devol', 'devlo', 'deovl', 'deolv', 'delvo', 'delov',
   'lvode', 'lvoed', 'lvdoe', 'lvdeo', 'lveod', 'lvedo', 'lovde', 'loved',
   'lodve', 'lodev', 'loevd', 'loedv', 'ldvoe', 'ldveo', 'ldove', 'ldoev',
   'ldevo', 'ldeov', 'levod', 'levdo', 'leovd', 'leodv', 'ledvo', 'ledov',
   'evodl', 'evold', 'evdol', 'evdlo', 'evlod', 'evldo', 'eovdl', 'eovld',
   'eodvl', 'eodlv', 'eolvd', 'eoldv', 'edvol', 'edvlo', 'edovl', 'edolv',
   'edlvo', 'edlov', 'elvod', 'elvdo', 'elovd', 'elodv', 'eldvo', 'eldov']
   >>>
➎ >>> print(*perms, sep='\n')
   vodle
   vodel
   volde
   voled
   voedl
   --snip--
```

从`itertools` ➊模块导入`permutations`开始。`itertools`模块是 Python 标准库中的一组函数，用于创建高效的迭代器。你通常会想到数字的排列，但`itertools`的版本是针对可迭代对象中的*元素*，其中包括字母。

输入名字或在此情况下，输入名字中的剩余字母后，使用列表推导创建名字的排列列表 ➋。将排列中的每个元素连接在一起，使得最终列表中的每一项都将是*vodle*的一个独特排列。使用`join`生成的新名字作为一个元素是，'vodle'，而不是一个难以阅读的单字符元组，`('v', 'o', 'd', 'l', 'e')`。

获取排列的长度作为一个检查点；这样你可以确认它确实是 5 的阶乘 ➌。最后，无论你以何种方式打印它 ➍➎，*volde*都很容易找到。

### **项目 #7：寻找伏地魔：英国暴力破解法**

现在假设 Tom Riddle 不擅长字谜（或者法语）。他无法识别 *mort* 或 *morte*，于是你又回到了将剩余的九个字母反复排列数千次，寻找一个他认为令人愉悦的字母组合。

从积极的一面来看，这是一个比你刚才看到的交互式解决方案更有趣的问题。你只需要通过某种形式的筛选，缩小所有排列的范围。

**目标**

将 *tmvoordle* 的字谜数量减少到一个可以管理的数字，这个数字仍然包含 *Voldemort*。

#### ***策略***

根据《牛津英语词典，第 2 版》，目前使用中的英语单词有 171,476 个，这比 *tmvoordle* 中的所有排列还要少！不论是哪种语言，你可以推测，通过 `permutations()` 函数生成的大多数字谜都是无意义的。

使用 *密码学*，即密码和加密的科学，你可以安全地排除许多无用的、无法发音的组合，例如 *ldtmvroeo*，而无需目视检查它们。密码学家长期研究语言，并编制了关于单词和字母重复模式的统计数据。我们可以使用许多密码分析技术来完成这个项目，但让我们集中于三种技术：辅音-元音映射、三元组频率和二元组频率。

##### **使用辅音-元音映射进行筛选**

一个 *辅音-元音映射*（*c-v 映射*）简单地将单词中的字母替换为 *c* 或 *v*，根据需要。例如，*Riddle* 变成 *cvcccv*。你可以编写一个程序，遍历字典文件并为每个单词创建 c-v 映射。默认情况下，不可能的组合，如 *ccccccvvv*，将被排除。你还可以通过移除那些 c-v 映射是 *可能的*，但出现频率很低的单词，进一步排除成员。

C-v 映射相当全面，但这是好事。目前，*Riddle* 的一个选择是创造一个新的专有名词，专有名词不一定需要出现在字典中。所以，刚开始的时候你不想过于*排外*。

##### **使用三元组进行筛选**

由于初始筛选器需要较宽的视野，你将需要在较低的层级再次进行筛选，以安全地从排列中移除更多的字谜。*三元组*是由三个连续字母组成的三连词。毫不奇怪，英语中最常见的三元组是单词 *the*，紧随其后的是 *and* 和 *ing*。在另一端，像 *zvq* 这样的三元组则极为罕见。

你可以在网站如 *[`norvig.com/ngrams/count_3l.txt`](http://norvig.com/ngrams/count_3l.txt)* 上找到三字母组合的出现频率统计数据。对于任何字母组合，比如 *tmvoordle*，你可以生成并使用一个最不常见的三字母组合列表，以进一步减少排列组合的数量。对于这个项目，你可以使用 *least-likely_trigrams.txt* 文件，该文件可以从 *[`www.nostarch.com/impracticalpython/`](https://www.nostarch.com/impracticalpython/)* 下载。这个文本文件包含了 *tmvoordle* 中出现频率处于英语三字母组合最低 10%的三字母组合。

##### **使用二字母组合进行过滤**

*二字母组合*（也称为 *双字母组合*）是指字母对。英语中常见的二字母组合包括 *an*、*st* 和 *er*。另一方面，像 *kg*、*vl* 或 *oq* 这样的组合很少见。你可以在以下网站上找到有关二字母组合出现频率的统计数据：* [`www.math.cornell.edu/~mec/2003-2004/cryptography/subs/digraphs.html`](https://www.math.cornell.edu/~mec/2003-2004/cryptography/subs/digraphs.html) * 和 * [`practicalcryptography.com/`](http://practicalcryptography.com/) *。

表格 3-1 是基于 *tmvoordle* 字母集合和一个包含 60,000 个单词的英语词典文件构建的。图表左侧的字母是二字母组合的起始字母，顶部的字母代表结束字母。例如，要查找 *vo*，从左侧的 *v* 开始，横向读取到 *o* 下方的列。在 *tmvoordle* 中，*vo* 只出现了 0.8% 的时间。

**表格 3-1：** 来自 60,000 词英语词典的 *tmvoordle* 字母组合的相对频率（黑色方格表示未出现）

|  | **d** | **e** | **l** | **m** | **o** | **r** | **t** | **v** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **d** |  | 3.5% | 0.5% | 0.1% | 1.7% | 0.5% | 0.0% | 0.1% |
| **e** | 6.6% |  | 2.3% | 1.4% | 0.7% | 8.9% | 2.0% | 0.6% |
| **l** | 0.4% | 4.4% |  | 0.1% | 4.2% | 0.0% | 0.4% | 0.1% |
| **m** | 0.0% | 2.2% | 0.0% |  | 2.8% | 0.0% | 0.0% | 0.0% |
| **o** | 1.5% | 0.5% | 3.7% | 3.2% | 5.3% | 7.1% | 2.4% | 1.4% |
| **r** | 0.9% | 6.0% | 0.4% | 0.7% | 5.7% |  | 1.3% | 0.3% |
| **t** | 0.0% | 6.2% | 0.6% | 0.1% | 3.6% | 2.3% |  | 0.0% |
| **v** | 0.0% | 2.5% | 0.0% | 0.0% | 0.8% | 0.0% | 0.0% |  |

假设你正在寻找“类英语”的字母组合，可以使用这样的频率图来排除不太可能出现的字母对。可以将其看作一个“二字母筛网”，只有未着色的方格才能通过。

为了安全起见，只需排除出现频率低于 0.1% 的二字母组合。我已将这些部分用黑色标记。请注意，如果裁剪过于严格，很容易会遗漏 *Voldemort* 中所需的 *vo* 组合！

你可以通过标记不太可能出现在单词开头的二元组来设计更具选择性的过滤器。例如，虽然二元组 *lm* 在单词 *内部* 中出现并不罕见（如 *almanac* 和 *balmy*），但你需要非常幸运才能找到一个以 *lm* 开头的单词。你不需要密码学来查找这些二元组；只需尝试发音它们！这些以灰色阴影显示的起始选择在 表 3-2 中。

**表 3-2：** 表 3-1 的更新，其中灰色阴影方块表示不太可能出现在单词开头的二元组

|  | **d** | **e** | **l** | **m** | **o** | **r** | **t** | **v** |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **d** |  | 3.5% | 0.5% | 0.1% | 1.7% | 0.5% | 0.0% | 0.1% |
| **e** | 6.6% |  | 2.3% | 1.4% | 0.7% | 8.9% | 2.0% | 0.6% |
| **l** | 0.4% | 4.4% |  | 0.1% | 4.2% | 0.0% | 0.4% | 0.1% |
| **m** | 0.0% | 2.2% | 0.0% |  | 2.8% | 0.0% | 0.0% | 0.0% |
| **o** | 1.5% | 0.5% | 3.7% | 3.2% | 5.3% | 7.1% | 2.4% | 1.4% |
| **r** | 0.9% | 6.0% | 0.4% | 0.7% | 5.7% |  | 1.3% | 0.3% |
| **t** | 0.0% | 6.2% | 0.6% | 0.1% | 3.6% | 2.3% |  | 0.0% |
| **v** | 0.0% | 2.5% | 0.0% | 0.0% | 0.8% | 0.0% | 0.0% |  |

现在，你可以使用三个过滤器来处理 *tmvoordle* 的 181,440 个排列：c-v 映射、三元组和二元组。作为最后的过滤器，你应该给用户选择只查看以给定字母开头的字谜的选项。这将使用户能够将剩余的字谜划分为更易管理的“块”，或者专注于那些听起来更具挑战性的字谜，比如那些以 *v* 开头的字谜！

#### ***英国暴力破解代码***

接下来的代码生成 *tmvoordle* 的所有排列，并通过刚才描述的过滤器。然后，它给用户一个选择，查看所有排列或仅查看以给定字母开头的排列。

你可以从 *[`www.nostarch.com/impracticalpython/`](https://www.nostarch.com/impracticalpython/)* 下载你需要的所有程序。本节中的代码是一个名为 *voldemort_british.py* 的脚本。你还需要同一文件夹中的 *load_dictionary.py* 程序，以及本章早些时候用于项目的相同字典文件。最后，你还需要一个名为 *least-likely_trigrams.txt* 的新文件，它是一个包含在英语中出现频率较低的三元组的文本文件。将所有这些文件下载到同一文件夹中。

##### **定义 main() 函数**

清单 3-5 导入了 *voldemort_british.py* 所需的模块，并定义了其 `main()` 函数。在 *phrase_anagrams.py* 程序中，你在代码的末尾定义了 `main()` 函数。这里我们将它放在开始处。其优点是你可以从一开始就看到函数的作用——它如何运行程序。缺点是，你还不知道任何辅助函数的作用。

*voldemort_british.py,* 第一部分

```py
➊ import sys
   from itertools import permutations
   from collections import Counter
   import load_dictionary

➋ def main():
       """Load files, run filters, allow user to view anagrams by 1st letter."""
    ➌ name = 'tmvoordle'
       name = name.lower()

    ➍ word_list_ini = load_dictionary.load('2of4brif.txt')
       trigrams_filtered = load_dictionary.load('least-likely_trigrams.txt')

    ➎ word_list = prep_words(name, word_list_ini)
       filtered_cv_map = cv_map_words(word_list)
       filter_1 = cv_map_filter(name, filtered_cv_map)
       filter_2 = trigram_filter(filter_1, trigrams_filtered)
       filter_3 = letter_pair_filter(filter_2)
       view_by_letter(name, filter_3)
```

*列表 3-5：导入模块并定义* main() *函数*

首先导入你在前面项目中使用过的模块 ➊。然后定义 `main()` 函数 ➋。`name` 变量是剩余字母的字符串 *tmvoordle* ➌。将其转换为小写字母，以防止用户输入错误。接下来，使用 `load_dictionary` 模块加载字典文件和三元组文件作为列表 ➍。你的字典文件名可能与所示的不同。

最后，按顺序调用所有各个函数 ➎。我稍后将描述这些函数的具体功能，但基本上，你需要准备词汇表，准备 c-v 图，应用三个筛选器，并让用户查看所有的字谜，或者根据字谜的首字母查看一个子集。

##### **准备词汇表**

列表 3-6 通过仅包括与 `name` 变量中字母数量相同的单词（在这个例子中是九个字母）来准备词汇表。你还应确保所有单词都转换为小写字母，以保持一致性。

*voldemort_british.py,* 第二部分

```py
➊ def prep_words(name, word_list_ini):
       """Prep word list for finding anagrams."""
    ➋ print("length initial word_list = {}".format(len(word_list_ini)))
       len_name = len(name)
    ➌ word_list = [word.lower() for word in word_list_ini
                    if len(word) == len_name]
    ➍ print("length of new word_list = {}".format(len(word_list)))
    ➎ return word_list
```

*列表 3-6：创建与* name *变量长度相等的单词列表*

定义 `prep_words()` 函数，接受一个名字字符串和字典单词列表作为参数 ➊。我建议你在单词列表经过筛选前后打印它们的长度；这样，你可以跟踪筛选器的影响。先打印字典的长度 ➋。分配一个变量来保存名字的长度，然后使用列表推导式通过遍历 `word_list_ini` 中的单词，筛选出那些长度与 `name` 中字母数量相同的单词，并将它们转换为小写字母 ➌。接下来，打印这个新单词列表的长度 ➍，最后返回这个新列表供下一个函数使用 ➎。

##### **生成 C-V 图**

你需要将准备好的词汇表转换为 c-v 图。记住，你不再关心字典中的实际单词；这些已经被审查并被拒绝。你的目标是将剩余字母打乱，直到它们形成类似专有名词的东西。

列表 3-7 定义了一个函数，用于为 `word_list` 中的每个单词生成 c-v 图。程序 *voldemort_british.py* 将使用 c-v 图来判断一个混合字母组合是否符合英语中的辅音-元音模式。

*voldemort_british.py,* 第三部分

```py
➊ def cv_map_words(word_list):
       """Map letters in words to consonants & vowels."""
    ➋ vowels = 'aeiouy'
    ➌ cv_mapped_words = []
    ➍ for word in word_list:
           temp = ''
           for letter in word:
               if letter in vowels:
                   temp += 'v'
               else:
                   temp += 'c'
           cv_mapped_words.append(temp)

       # determine number of UNIQUE c-v patterns
    ➎ total = len(set(cv_mapped_words))
       # target fraction to eliminate
    ➏ target = 0.05
       # get number of items in target fraction
    ➐ n = int(total * target)
    ➑ count_pruned = Counter(cv_mapped_words).most_common(total - n)
    ➒ filtered_cv_map = set()
       for pattern, count in count_pruned:
           filtered_cv_map.add(pattern)
       print("length filtered_cv_map = {}".format(len(filtered_cv_map)))
    ➓ return filtered_cv_map
```

*列表 3-7：从* word_list *中的单词生成 c-v 图*

定义 `cv_map_words()` 函数，接受准备好的词汇表作为参数 ➊。由于辅音和元音构成一个二进制系统，你可以使用一个字符串来定义元音 ➋。创建一个空列表来保存这些图 ➌。然后遍历单词和每个单词中的字母，将字母转换为 *c* 或 *v* ➍。使用一个名为 `temp` 的变量来积累这个图，然后将其附加到列表中。请注意，`temp` 每次循环时都会重新初始化。

你希望了解给定 c-v 映射模式（例如 *cvcv*）的出现频率，以便可以去除那些出现可能性较低的项。在计算频率之前，你需要将列表简化为唯一的 c-v 映射——目前，*cvcv* 可能会被多次重复。因此，将 `cv_mapped_words` 列表转换为集合，以去除重复项，并获取其长度 ➎。现在，你可以定义一个目标百分比来进行去除，使用分数值 ➏。从像 `0.05` 这样的低值开始——相当于 5%——这样你就不太可能去除那些可以形成有效专有名词的字谜。将这个目标值乘以 `cv_mapped_words` 集合的总长度，并将结果赋值给变量 `n` ➐。一定要将 `n` 转换为整数；因为它代表的是一个计数值，不能是浮动数。

`Counter` 模块的数据类型有一个非常实用的方法 `most_common()`，它会根据你提供的 *count* 值返回列表中最常见的项；在此例中，该值为 c-v 映射列表的长度 `total` 减去 `n`。你传给 `most_common()` 的值必须是整数。如果你传递 `most_common()` 函数的是列表的长度，它将返回列表中的所有项。如果你减去最不可能出现的 5% 的计数，你将有效地从列表中去除这些 c-v 映射 ➑。

记住，`Counter` 返回的是字典，但你只需要最终的 c-v 映射，而不是它们相关的频率计数。因此，初始化一个名为 `filtered-cv-map` 的空集合 ➒，并遍历 `count_pruned()` 中的每个键值对，只将键添加到新集合中。打印该集合的长度，这样你可以看到过滤器的影响。然后，完成并返回过滤后的 c-v 映射，以便在下一个函数中使用 ➓。

##### **定义 C-V 映射过滤器**

清单 3-8 应用了 c-v 映射过滤器：基于 `name` 变量中字母的排列生成字谜，然后程序将其转换为 c-v 映射，并将这些字谜与通过 `cv_map_words()` 函数构建的过滤后的 c-v 映射进行比较。如果一个字谜的 c-v 映射出现在 `filtered_cv_map` 中，则程序会将该字谜存储，以供下一个过滤器使用。

*voldemort_british.py*，第四部分

```py
➊ def cv_map_filter(name, filtered_cv_map):
       """Remove permutations of words based on unlikely cons-vowel combos."""
    ➋ perms = {''.join(i) for i in permutations(name)}
       print("length of initial permutations set = {}".format(len(perms)))
       vowels = 'aeiouy'
    ➌ filter_1 = set()
    ➍ for candidate in perms:
           temp = ''
           for letter in candidate:
               if letter in vowels:
                   temp += 'v'
               else:
                   temp += 'c'
        ➎ if temp in filtered_cv_map:
               filter_1.add(candidate)
       print("# choices after filter_1 = {}".format(len(filter_1)))
    ➏ return filter_1
```

*清单 3-8：定义* cv_map_filter() *函数*

定义 `cv_map_filter()` 函数，接受两个参数：名字，后面跟着 `cv_map_words()` 返回的 c-v 映射集合 ➊。使用集合推导和 `permutations` 模块生成排列的集合 ➋。我在“项目 #6：寻找伏地魔：高卢赌局”中，具体描述了这个过程，详见 第 49 页。在这里使用集合，允许后续进行集合操作，例如计算两个过滤器集合的差异。这也能去除重复项，因为 `permutations` 将每个 *o* 视为单独的项，并返回 9!，而不是 9! / 2!。请注意，`permutations` 会将 *tmv**o**ordle* 和 *tmvo**o**rdle* 视为不同的字符串。

现在初始化一个空集合来保存第一个过滤器的内容 ➌，并开始循环检查排列 ➍。使用术语 *candidate*，因为这些大多数不是单词，而只是随机字母的字符串。对于每个候选项，循环遍历字母并将它们映射为 *c* 或 *v*，就像你在 `cv_words()` 函数中做的那样。检查每个 c-v 映射 `temp` 是否存在于 `filtered_cv_map` 中。这是使用集合的一个原因：成员检查非常快速。如果候选项符合条件，则将其添加到 `filter_1` ➎。最后返回你的新字谜集合 ➏。

##### **定义三元组过滤器**

列表 3-9 定义了三元组过滤器，它移除包含不太可能的三字母组合的排列。它使用一个来源于多个加密学网站的文本文件，这些内容已经根据 *tmvoordle* 中的字母进行了调整。此函数将只返回包含这些三元组之一的排列；`main()` 函数将把新的集合传递给下一个过滤函数。

*voldemort_british.py，* 第五部分

```py
➊ def trigram_filter(filter_1, trigrams_filtered):
       """Remove unlikely trigrams from permutations."""
    ➋ filtered = set()
    ➌ for candidate in filter_1:
        ➍ for triplet in trigrams_filtered:
               triplet = triplet.lower()
               if triplet in candidate:
                   filtered.add(candidate)
    ➎ filter_2 = filter_1 - filtered
       print("# of choices after filter_2 = {}".format(len(filter_2)))
    ➏ return filter_2
```

*列表 3-9：定义了* trigram_filter() *函数*

三元组过滤器的参数包括来自 c-v 映射过滤器的输出以及外部的非可能三元组列表 `trigrams_filtered` ➊。

初始化一个空集合来保存包含禁止的三元组的排列 ➋。然后启动另一个 `for` 循环，检查通过上一个过滤器的候选项 ➌。一个嵌套的 `for` 循环查看三元组列表中的每一个三元组 ➍。如果该三元组出现在候选项中，它将被添加到过滤器中。

现在，你可以使用集合操作从 `filter_1` ➎ 中减去新的过滤器，然后返回差异，用于下一个过滤器 ➏。

##### **定义二元组过滤器**

列表 3-10 定义了二元组过滤器，它移除不太可能出现的字母对。如果它们在排列中的任何位置出现，将触发过滤器；如果它们出现在排列的开头，则仅在这种情况下触发。禁止的二元组基于 表 3-2 中的阴影单元格。该函数返回此过滤器的结果，用于最终的过滤函数。

*voldemort_british.py，* 第六部分

```py
➊ def letter_pair_filter(filter_2):
       """Remove unlikely letter-pairs from permutations."""
    ➋ filtered = set()
    ➌ rejects = ['dt', 'lr', 'md', 'ml', 'mr', 'mt', 'mv',
                  'td', 'tv', 'vd', 'vl', 'vm', 'vr', 'vt']
    ➍ first_pair_rejects = ['ld', 'lm', 'lt', 'lv', 'rd',
                             'rl', 'rm', 'rt', 'rv', 'tl', 'tm']
    ➎ for candidate in filter_2:
        ➏ for r in rejects:
               if r in candidate:
                   filtered.add(candidate)
        ➐ for fp in first_pair_rejects:
               if candidate.startswith(fp):
                   filtered.add(candidate)
    ➑ filter_3 = filter_2 - filtered
       print("# of choices after filter_3 = {}".format(len(filter_3)))
    ➒ if 'voldemort' in filter_3:
           print("Voldemort found!", file=sys.stderr)
    ➓ return filter_3
```

*列表 3-10：定义了* letter_pair_filter() *函数*

该过滤器接受前一个过滤器的结果作为参数 ➊。初始化一个空集合来保存任何被丢弃的排列 ➋。然后将两个被拒绝的对列表分别赋值给变量 `rejects` ➌ 和 `first_pair_rejects` ➍。这两个列表是手动输入的。第一个表示 表 3-2 中阴影为黑色的单元格；第二个参考阴影为灰色的单元格。任何包含第一个列表中的成员的排列——无论在哪里——都会被丢弃；以第二个列表中的成员开头的排列将不被允许。你可以向这些列表中添加或删除二元组来更改过滤器的行为。

开始循环排列——继续称这些为“候选项”，因为它们不一定是单词 ➎。一个嵌套的`for`循环遍历`rejects`中的所有对，判断是否有任何一对在`candidate`中，并将其添加到`filtered`集合中 ➏。第二个嵌套的`for`循环对`first_pair_rejects`执行相同的过程 ➐。将`filtered`从上一个函数`filter_2`返回的集合中减去 ➑。

为了增加趣味性*并*确保没有过度过滤，可以检查`filter_3` ➒中是否包含*voldemort*，并用引人注目的红色字体打印一条公告，以提醒发现此项，供 IDLE 用户使用。最后，返回最终的过滤结果集 ➓。

##### **让用户选择起始字母**

你无法预先知道过滤是否成功。你可能依然会得到成千上万的排列。提供只查看部分输出的选项并不会减少总体数量，但它会使得心理上*更容易接受*。 示例 3-11 向*voldemort_british.py*中添加了查看以特定输入字母开头的字谜列表的功能。

*voldemort_british.py,* 第七部分

```py
➊ def view_by_letter(name, filter_3):
       """Filter to anagrams starting with input letter."""
    ➋ print("Remaining letters = {}".format(name))
    ➌ first = input("select a starting letter or press Enter to see all: ")
    ➍ subset = []
    ➎ for candidate in filter_3:
           if candidate.startswith(first):
               subset.append(candidate)
    ➏ print(*sorted(subset), sep='\n')
       print("Number of choices starting with {} = {}".format(first, len(subset)))
    ➐ try_again = input("Try again? (Press Enter else any other key to Exit):")
       if try_again.lower() == '':
        ➑ view_by_letter(name, filter_3)
       else:
        ➒ sys.exit()
```

*示例 3-11：定义了* view_by_letter() *函数*

定义`view_by_letter()`函数，将`name`变量和`filter_3`作为参数 ➊。需要`name`，以便展示用户可以过滤的字母选项 ➋。获取用户输入，询问他们是否想查看所有剩余的排列，还是只查看以某个特定字母开头的排列 ➌。然后开始一个空列表，用于保存这个子集 ➍。

一个`for`循环，配合条件语句，检查候选项是否以选定字母开头，并将通过检查的字母添加到`subset` ➎。然后通过解包操作符打印这个列表 ➏。接着程序会询问用户是否希望重新尝试或退出 ➐。如果用户按下 ENTER，`view_by_letter()`会被递归调用，并从头开始重新运行 ➑。否则，程序退出 ➒。请注意，Python 默认的递归深度限制是 1,000，但在这个项目中我们会忽略它。

##### **运行 main()函数**

在全局空间中，示例 3-12 通过调用`main()`函数完成代码，如果用户以独立模式运行程序而不是将其导入到另一个程序中。

*voldemort_british.py,* 第八部分

```py
if __name__ == '__main__':
    main()
```

*示例 3-12：调用* main() *函数*

完成程序的示例输出如下所示。在程序应用第三个过滤器后，剩下 248 个排列，其中有 73 个以*v*开头，非常容易处理。我省略了排列的打印输出以简化说明。如输出所示，*voldemort*成功通过了过滤。

```py
length initial word_list = 60388
length of new word_list = 8687
length filtered_cv_map = 234
length of initial permutations set = 181440
# choices after filter_1 = 123120
# of choices after filter_2 = 674
# of choices after filter_3 = 248
Voldemort found!
Remaining letters = tmvoordle
select a starting letter or Enter to see all: v
```

有趣的是，另一个存活下来的排列是*lovedmort*。鉴于伏地魔杀害（或指使杀害）了那么多人，这可能是最合适的名字。

### **总结**

在这一章中，你首先编写了代码，找到给定单词或名字的字谜。然后你扩展了这个功能，找到了短语名称字谜，并与用户互动。最后，你采用了密码分析技术，从近 200,000 个可能的字谜中找出*伏地魔*。在此过程中，你应用了 `collections` 和 `itertools` 模块中的有用功能。

### **进一步阅读**

*Jumble* 网站是 *[`www.jumble.com/`](http://www.jumble.com/)*。

你可以在以下网站找到一些具有代表性的在线字谜生成器：

+   *[`wordsmith.org/anagram/`](http://wordsmith.org/anagram/)*

+   *[`www.dcode.fr/anagram-generator`](https://www.dcode.fr/anagram-generator)*

+   *[`www.wordplays.com/anagrammer/`](http://www.wordplays.com/anagrammer/)*

更多字谜程序可以在 Allen Downey 编写的《*Think Python, 2nd Edition*》（O'Reilly，2015）中找到。

Al Sweigart 编写的《*Cracking Codes with Python*》（No Starch Press，2017）提供了更多计算单词模式的代码，例如在 *voldemort_british.py* 程序中过滤时使用的那些模式。

### **实践项目：寻找双字谜**

你*可以*浏览加密学网站查找频率统计数据，或者你也可以自己推导这些数据。编写一个 Python 程序，找到 *tmvoordle* 中的所有双字谜，并计算它们在字典文件中的出现频率。确保在像 *volvo* 这样的单词上测试你的代码，以避免忽略同一单词中重复的双字谜。你可以在附录中找到解决方案，或者从 *[`www.nostarch.com/impracticalpython/`](https://www.nostarch.com/impracticalpython/)* 下载 *count_digrams_practice.py*。

### **挑战项目：自动字谜生成器**

查看我在“进一步阅读”中刚才提到的在线字谜生成器，并编写一个 Python 程序来模仿其中一个。你的程序应该根据输入的名字自动生成短语字谜，并显示一部分（例如前 500 个）供用户查看。
