## 计数俳句诗歌的音节**

![image](img/common01.jpg)

诗歌可能是文学的最高形式。正如柯尔律治所说，“最好的词语按最好的顺序排列。”诗人必须以极简的语言讲述故事，传播思想，描绘场景，或唤起强烈的情感，同时遵循严格的节奏和押韵规则、风格和结构。

计算机喜欢规则和结构，甚至有潜力激发情感。在 1996 年出版的《虚拟缪斯：计算机诗歌实验》一书中，作者查尔斯·哈特曼描述了早期尝试编写能够模仿人类诗歌的算法。引用哈特曼的话说：“诗歌互动的复杂性，诗人、文本和读者之间微妙的舞蹈，导致了一场犹豫的博弈。在这场博弈中，经过适当编程的计算机有机会插入一些有趣的动作。”

哈特曼描述的早期程序，充其量只能产生糟糕的嬉皮诗。那时的目标是“将计算出的机械化无序引入语言，将结果放回语言存在的具体世界，看看尘土如何落定。”正如我们在几个章节中提到的，上下文是编程中弱的一环，比如专有名词的字谜和空白密码。为了写出能够通过文学“至高”测试的计算机诗歌，你不能忽视上下文。

让计算机模拟这一最具人类特质的活动是一个引人入胜的挑战——而且无疑是我们无法放弃的挑战。在本章及下一章中，你将教你的计算机生成一种传统的日本诗歌形式——*俳句*。

### **日本俳句**

俳句由三行组成，分别为五个、七个和五个音节。这些诗歌很少押韵，题材通常直接或间接地涉及自然世界——主要是季节。如果做得好，一首俳句能够让你沉浸在场景中，就像唤起一段记忆一样。

我这里提供了三首俳句范例。第一首是大师布孙（1715–1783）创作的，第二首是大师一茶（1763–1828）创作的，第三首是我自己创作的，基于儿时公路旅行的回忆。

黄昏时静立

听啊。。。在遥远的地方

青蛙的歌声！

*—布孙*

好朋友蝉蜕

你会当看护者吗

为我的小墓地吗？

*—一茶*

远方的云层

我让自己假装

远山的景象

*—沃恩*

由于其唤起性特征，每一首俳句都内置了程序员可以“利用的空隙”。这一点彼得·贝伦森在他 1955 年出版的书《日本俳句》中有一句话总结得很好：“俳句并不总是期待成为一个完整的，甚至是清晰的陈述。读者应该将自己的联想和意象添加到文字中，从而成为自己在诗中愉悦的共同创作者。”哈特曼补充道：“读者的大脑在稀疏的材料上最为活跃。我们从最少的星星中画出最清晰的星座。因此，对于一小组可以赋予意象意义的词语来说，胡说八道的因素很低。”简单来说，搞砸一首短诗更为困难。读者总是认为诗人有某种意图，并且如果他们找不到，就会自己构造一个。

尽管有这个优势，训练计算机写诗并非易事，你将需要两个完整的章节才能完成。在本章中，你将编写一个程序来计算单词和短语中的音节数，以便你能够遵循俳句的音节结构。在第九章，你将使用一种叫做*马尔可夫链分析*的技术来捕捉俳句的本质——那种难以捉摸的唤起性成分——并将现有的诗歌转变为新的，有时甚至是更好的作品。

### **项目 #15：音节计数**

计算英语音节是困难的。问题在于，正如查尔斯·哈特曼所说，英语的拼写古怪且语言历史错综复杂。例如，一个词像*aged*（年老的）根据它是描述一个人还是奶酪，可能是一个音节，也可能是两个音节。一个程序如何在不沦为无休止的特殊情况列表的情况下，准确地计算音节呢？

答案是它无法做到，至少没有“备忘单”。幸运的是，借助于一门叫做*自然语言处理*（*NLP*）的科学分支，这些“备忘单”存在了，它处理的是计算机的精确结构化语言与人类使用的含糊且常常模糊的“自然”语言之间的互动。NLP 的示例应用包括机器翻译、垃圾邮件检测、搜索引擎问题理解和手机用户的预测文本识别。NLP 的最大影响还未到来：即挖掘大量以前无法使用的、结构不良的数据，并与我们的计算机“霸主”进行无缝对话。

在本章中，你将使用一个 NLP 数据集来帮助计算单词或短语中的音节数量。你还将编写代码，找到缺失于此数据集的单词，并帮助你建立一个辅助字典。最后，你将编写一个程序，帮助你检查你的音节计数代码。在第九章，你将把这个音节计数算法作为模块，嵌入到一个程序中，帮助你在计算上创作文学中的最高成就：诗歌。

**目标**

编写一个 Python 程序，统计英语单词或短语中的音节数。

### **策略**

对于你和我来说，数音节很简单。将手背放在下巴下方开始说话，每次下巴碰到手时，你就说了一个音节。电脑没有手或下巴，但每个元音音素代表一个音节——而计算机可以数元音音素。然而，这并不容易，因为没有简单的规则来执行这一操作。有些书面语言中的元音是哑音，比如 *like* 中的 *e*，而有些元音组合成一个声音，比如 *moo* 中的 *oo*。幸运的是，英语中的单词数量不是无限的。已经有相当详尽的单词列表，其中包含你需要的很多信息。

*语料库* 是文本的一个高级名称。在第九章中，你将使用一个由俳句组成的*训练语料库*，它教会 Python 如何写新的俳句。在本章中，你将使用这个相同的语料库来提取音节计数。

你的音节计数器应评估短语和单个单词，因为最终你将使用它来统计俳句中整个*行*的音节数。该程序将接收一些文本作为输入，统计每个单词的音节数，并返回总音节数。你还需要处理标点符号、空格和缺失的单词等问题。

你需要遵循的主要步骤是：

1.  下载一个包含音节计数信息的大型语料库。

1.  将音节计数语料库与俳句训练语料库进行比较，找出所有缺失在音节计数语料库中的单词。

1.  构建一个缺失单词及其音节数的字典。

1.  编写一个程序，使用音节计数语料库和缺失单词字典来统计训练语料库中的音节数。

1.  编写一个程序，将音节计数程序与更新后的训练语料库进行比对。

#### ***使用语料库***

自然语言工具包（NLTK）是一个流行的程序和库套件，专门用于处理 Python 中的自然语言数据。它于 2001 年作为宾夕法尼亚大学计算机与信息科学系的一门计算语言学课程的一部分创建。随着众多贡献者的帮助，它不断发展和扩展。要了解更多信息，请访问官方 NLTK 网站 *[`www.nltk.org/`](http://www.nltk.org/)*。

在这个项目中，你将使用 NLTK 访问卡内基梅隆大学发音词典（CMUdict）。该语料库包含近 125,000 个单词及其发音映射。它是机器可读的，适用于语音识别等任务。

#### ***安装 NLTK***

你可以在 *[`www.nltk.org/install.html`](http://www.nltk.org/install.html)* 上找到安装 NLTK 的说明，适用于 Unix、Windows 和 macOS。如果你使用的是 Windows，我建议你先打开 Windows 命令提示符或 PowerShell，并尝试使用 `pip` 进行安装：

```py
python -m pip install nltk
```

你可以通过打开 Python 交互式 shell 并输入以下内容来检查安装：

```py
>>> import nltk
>>>
```

如果没有遇到错误，那么你就可以继续了。否则，请按照网站上提供的说明操作。

#### ***下载 CMUdict***

要访问 CMUdict（或任何其他 NLTK 语料库），你需要下载它。你可以使用便捷的 NLTK 下载器来实现这一点。安装 NLTK 后，在 Python shell 中输入以下内容：

```py
>>> import nltk
>>> nltk.download()
```

NLTK 下载器窗口（图 8-1）应该已经打开。点击顶部的 **Corpora** 标签，然后在标识符列中点击 **cmudict**。接着，滚动到窗口底部并设置下载目录；我使用了默认目录，*C:\nltk_data*。最后，点击 **下载** 按钮加载 CMUdict。

![image](img/f0149-01.jpg)

*图 8-1：NLTK 下载器窗口，已选择 cmudict 进行下载*

当 CMUdict 下载完成后，退出下载器并在 Python 交互式 shell 中输入以下内容：

```py
>>> from nltk.corpus import cmudict
>>>
```

如果没有遇到错误，则表示语料库已成功下载。

#### ***计算声音而不是音节***

CMUdict 语料库将单词分解为一组 *音素* ——在特定语言中感知上不同的 *声音单元* ——并通过数字（0、1 和 2）标记元音的词汇重音。CMUdict 语料库为每个元音标记一个且仅一个这些数字，因此你可以使用这些数字来识别单词中的元音。

将单词视为音素集合将帮助你避免一些问题。首先，CMUdict 不会包括书写中未发音的元音。例如，下面是 CMUdict 如何表示单词 *scarecrow*：

```py
[['S', 'K', 'AE1', 'R', 'K', 'R', 'OW0']]
```

每个带有数字后缀的项目表示一个发音元音。注意，*scare* 结尾的静音 *e* 被正确省略了。

其次，有时多个连续的书写元音会被发音为一个单一音素。例如，CMUdict 是如何表示 *house* 的：

```py
[['HH', 'AW1', 'S']]
```

注意语料库如何将书写中的双元音 *ou* 作为一个单一元音 `'AW1'` 来处理，用于发音目的。

#### ***处理有多重发音的词语***

正如我在介绍中提到的，某些单词有多种不同的发音；*aged* 和 *learned* 就是两个例子：

```py
[['EY1', 'JH', 'D'], ['EY1', 'JH', 'IH0', 'D']]
[['L', 'ER1', 'N', 'D'], ['L', 'ER1', 'N', 'IH0', 'D']]
```

注意嵌套列表。语料库识别到这两个词可以用一个或两个音节发音。这意味着它会返回多个音节数，对于某些词，你需要在代码中考虑这一点。

### **管理缺失的词语**

CMUdict 非常有用，但在语料库中，一个词要么存在，要么不存在。仅仅几秒钟，我就找到了 50 多个词——比如 *dewdrop*、*bathwater*、*dusky*、*ridgeline*、*storks*、*dragonfly*、*beggar* 和 *archways*——在一个 1500 词的测试用例中缺失。因此，你的策略之一应该是检查 CMUdict 中缺失的词，并通过为你的语料库创建一个语料库来解决这些遗漏！

#### ***训练语料库***

在第九章，你将使用数百首俳句作为训练语料库来“教”你的程序如何创作新的俳句。但是你不能指望 CMUdict 包含这个语料库中的所有单词，因为一些单词是日语词汇，比如*sake*。正如你已经看到的那样，甚至一些常见的英语单词在 CMUdict 中也缺失。

所以，首要任务是检查训练语料库中的所有单词是否存在于 CMUdict 中。为此，你需要从 *[`www.nostarch.com/impracticalpython/`](https://www.nostarch.com/impracticalpython/)* 下载训练语料库，名为*train.txt*。将它和本章中的所有 Python 程序保存在同一个文件夹里。该文件包含略低于 300 首俳句，并且为了确保训练集的可靠性，这些俳句被随机重复了大约 20 次。

一旦你找到不在 CMUdict 中的单词，你将编写一个脚本，帮助你准备一个 Python 字典，将单词作为键，音节计数作为值；然后你将把这个字典保存到一个文件中，以便在音节计数程序中支持 CMUdict。

#### ***缺失单词的代码***

本节中的代码将查找 CMUdict 中缺失的单词，帮助你准备一个包含单词及其音节计数的字典，并将字典保存到文件中。你可以从 *[`nostarch.com/impracticalpython/`](https://nostarch.com/impracticalpython/)* 下载代码，文件名为 *missing_words_finder.py*。

##### **导入模块、加载 CMUdict 并定义 main()函数**

清单 8-1 导入模块，加载 CMUdict，并定义运行程序的`main()`函数。

*missing_words_finder.py,* 第一部分

```py
   import sys
   from string import punctuation
➊ import pprint
   import json
   from nltk.corpus import cmudict

➋ cmudict = cmudict.dict()  # Carnegie Mellon University Pronouncing Dictionary

➌ def main():
    ➍ haiku = load_haiku('train.txt')
    ➎ exceptions = cmudict_missing(haiku)
    ➏ build_dict = input("\nManually build an exceptions dictionary (y/n)? \n")
       if build_dict.lower() == 'n':
           sys.exit()
       else:
        ➐ missing_words_dict = make_exceptions_dict(exceptions)
           save_exceptions(missing_words_dict)
```

*清单 8-1：导入模块，加载 CMUdict，并定义* main()

你从一些熟悉的导入开始，并添加几个新的模块。`pprint`模块允许你以易于阅读的格式“美化打印”缺失单词的字典 ➊。你将使用 JavaScript 对象表示法（`json`）将这个字典写出作为持久化数据，`json`是一种基于文本的计算机数据交换方式，能够很好地与 Python 数据结构配合使用；它是标准库的一部分，在多个编程语言中都得到标准化，数据安全且易于阅读。最后，导入 CMUdict 语料库。

接下来，调用`cmudict`模块的`dict()`方法，将语料库转化为一个字典，字典的键是单词，值是它们的音素 ➋。

定义`main()`函数，该函数将调用其他函数来加载训练语料库，查找 CMUdict 中缺失的单词，构建包含单词及其音节计数的字典，并保存结果 ➌。你将在定义`main()`后定义这些函数。

调用该函数加载俳句训练语料库，并将返回的集合赋值给名为`haiku`的变量 ➍。然后调用一个函数，查找缺失的单词并将它们作为集合返回 ➎。使用集合可以去除不需要的重复单词。`cmudict_missing()`函数还会显示缺失单词的数量和一些其他统计信息。

现在，询问用户是否想手动构建一个字典以处理缺失的单词，并将他们的输入赋值给`build_dict`变量 ➏。如果他们想停止，退出程序；否则，调用一个函数来构建字典 ➐，然后再调用另一个函数保存字典。请注意，用户并不一定非得按`y`键继续，尽管这是提示。

##### **加载训练语料库并寻找缺失的单词**

清单 8-2 加载并准备训练语料库，将其内容与 CMUdict 进行比较，并跟踪差异。这些任务由两个函数分工完成。

*missing_words_finder.py*，第二部分

```py
➊ def load_haiku(filename):
       """Open and return training corpus of haiku as a set."""
       with open(filename) as in_file:
     ➋ haiku = set(in_file.read().replace('-', ' ').split())
     ➌ return haiku

   def cmudict_missing(word_set):
       """Find and return words in word set missing from cmudict."""
    ➍ exceptions = set()
       for word in word_set:
           word = word.lower().strip(punctuation)
           if word.endswith("'s") or word.endswith("’s"):
               word = word[:-2]
        ➎ if word not in cmudict:
               exceptions.add(word)
       print("\nexceptions:")
       print(*exceptions, sep='\n')
    ➏ print("\nNumber of unique words in haiku corpus = {}"
             .format(len(word_set)))
       print("Number of words in corpus not in cmudict = {}"
             .format(len(exceptions)))
       membership = (1 - (len(exceptions) / len(word_set))) * 100
    ➐ print("cmudict membership = {:.1f}{}".format(membership, '%'))
       return exceptions
```

*清单 8-2: 定义加载语料库的函数并查找缺失的单词*

定义一个函数来读取来自 haiku 训练语料库中的单词 ➊。*train.txt*中的 haiku 已经被重复多次，且原始 haiku 包含了重复的单词，如*moon*、*mountain*和*the*。没有必要多次评估一个单词，因此将这些单词加载为集合来去重 ➋。你还需要用空格替换连字符。连字符在 haiku 中很常见，但你需要将它们两侧的单词分开，以便在 CMUdict 中进行检查。函数结束时返回`haiku`集合 ➌。

现在是时候找出缺失的单词了。定义一个函数`cmudict_missing()`，它以一个序列作为参数——在这种情况下，是`load_haiku()`函数返回的单词集。创建一个空的`exceptions`集合来保存任何缺失的单词 ➍。遍历每个 haiku 单词集中的单词，将其转换为小写，并去除任何前后标点。请注意，你不想去除除了连字符之外的内部标点，因为 CMUdict 识别诸如*wouldn’t*之类的单词。拥有所有格的单词通常不在语料库中，因此去掉尾部的*’s*，因为这不会影响音节数的统计。

**注意**

*小心使用文本处理软件生成的弯引号（’）。这些与简单文本编辑器和命令行中使用的直引号（*`'`*）不同，CMUdict 可能无法识别这些弯引号。如果你向训练文件或 JSON 文件中添加新单词，请务必使用直引号来表示缩写或所有格名词。*

如果在 CMUdict 中找不到该单词，将其添加到`exceptions` ➎。打印这些单词作为检查，并提供一些基本信息 ➏，如有多少个唯一单词、缺失单词的数量以及训练语料库中有多少百分比的单词是 CMUdict 的成员。将百分比的精度设置为小数点后一位 ➐。函数结束时返回异常单词集。

##### **构建缺失词字典**

列表 8-3 继续了 *missing_words_finder.py* 代码，现在通过将音节数作为值分配给缺失的单词，来补充 CMUdict 数据。由于缺失的单词数量应该相对较小，用户可以手动分配这些音节数，因此编写代码帮助用户与程序交互。

*missing_words_finder.py*，第三部分

```py
➊ def make_exceptions_dict(exceptions_set):
       """Return dictionary of words & syllable counts from a set of words."""
    ➋ missing_words = {}
       print("Input # syllables in word. Mistakes can be corrected at end. \n")
       for word in exceptions_set:
           while True:
            ➌ num_sylls = input("Enter number syllables in {}: ".format(word))
            ➍ if num_sylls.isdigit():
                   break
               else:
                   print("                 Not a valid answer!", file=sys.stderr)
        ➎ missing_words[word] = int(num_sylls)
       print()
    ➏ pprint.pprint(missing_words, width=1)

    ➐ print("\nMake Changes to Dictionary Before Saving?")
       print("""
       0 - Exit & Save
       1 – Add a Word or Change a Syllable Count
       2 - Remove a Word
       """)

    ➑ while True:
           choice = input("\nEnter choice: ")
           if choice == '0':
               break
           elif choice == '1':
               word = input("\nWord to add or change: ")
               missing_words[word] = int(input("Enter number syllables in {}: "
                                               .format(word)))
           elif choice == '2':
               word = input("\nEnter word to delete: ")
            ➒ missing_words.pop(word, None)

       print("\nNew words or syllable changes:")
    ➓ pprint.pprint(missing_words, width=1)

       return missing_words
```

*列表 8-3：允许用户手动计算音节并构建字典*

从定义一个函数开始，该函数接受由 `cmudict_missing()` 函数返回的异常集合作为参数 ➊。立即将一个空字典赋值给名为 `missing_words` 的变量 ➋。让用户知道如果他们犯了错误，会有机会稍后修正；然后，使用 `for` 和 `while` 循环遍历缺失的单词集，向用户展示每个单词，询问音节数作为输入。单词将是字典的键，`num_sylls` 变量将成为它的值 ➌。若输入是数字 ➍，则跳出循环。否则，警告用户并让 `while` 循环再次请求输入。如果输入通过，便将该值作为整数添加到字典中 ➎。

使用 `pprint` 来单独显示每个键/值对，作为检查。`width` 参数作为换行符号 ➏。

给用户提供机会在保存 `missing_words` 字典为文件之前进行最后修改 ➐。使用三引号展示选项菜单，然后用 `while` 循环保持选项活跃，直到用户准备好保存 ➑。三个选项是退出，触发 `break` 命令；添加新单词或更改现有单词的音节数，需要输入单词和音节数；以及删除条目，使用字典的 `pop()` 函数 ➒。将 `None` 参数添加到 `pop()` 中意味着如果用户输入的单词不在字典中，程序不会抛出 `KeyError`。

最后，给用户一个最后查看字典的机会，以防有更改 ➓，然后返回字典。

##### **保存缺失单词字典**

*持久数据* 是在程序终止后仍然保存的数据。为了在后面章节中编写的 *count_syllables.py* 程序中使用 `missing_words` 字典，你需要将它保存到文件中。列表 8-4 就是完成这一操作的代码。

*missing_words_finder.py*，第四部分

```py
➊ def save_exceptions(missing_words):
       """Save exceptions dictionary as json file."""
    ➋ json_string = json.dumps(missing_words)
    ➌ f = open('missing_words.json', 'w')
       f.write(json_string)
       f.close()
    ➍ print("\nFile saved as missing_words.json")

➎ if __name__ == '__main__':
       main()
```

*列表 8-4：将缺失单词字典保存到文件并调用* main()

使用 `json` 来保存字典。定义一个新函数，接受缺失单词集合作为参数 ➊。将 `missing_words` 字典赋值给一个名为 `json_string` 的新变量 ➋；然后，打开一个以 *.json* 为扩展名的文件 ➌，写入 `json` 变量，并关闭文件。提醒用户文件名 ➍。最后，编写代码，让程序可以作为模块或独立模式运行 ➎。

`json.dumps()`方法将`missing_words`字典序列化为一个字符串。*序列化*是将数据转换为更易传输或存储格式的过程。例如：

```py
>>> import json
>>> d = {'scarecrow': 2, 'moon': 1, 'sake': 2}
>>> json.dumps(d)
'{"sake": 2, "scarecrow": 2, "moon": 1}'
```

请注意，序列化的字典被单引号包围，使其成为一个字符串。

我在这里提供了*missing_words_finder.py*的部分输出。顶部的缺失单词列表和底部的手动音节计数都已缩短，以便简洁。

```py
--snip--
froglings
scatters
paperweights
hibiscus
cumulus
nightingales

Number of unique words in haiku corpus = 1523
Number of words in corpus not in cmudict = 58
cmudict membership = 96.2%

Manually build an exceptions dictionary (y/n)?
y
Enter number syllables in woodcutter: 3
Enter number syllables in morningglory: 4
Enter number syllables in cumulus: 3
--snip--
```

别担心——你不需要为所有单词分配音节计数。*missing_words.json*文件已经完成，并且在需要时可以下载。

**注意**

*对于有多种发音的单词，比如* jagged *或* our*，你可以通过手动打开*missing_words.json*文件并在任意位置添加键/值对，强制程序使用你喜欢的发音。我就是通过这种方式调整了单词*sake*，让它使用两音节的日语发音。因为该文件首先会检查单词的成员资格，它会覆盖 CMUdict 中的值。*

现在你已经解决了 CMUdict 中的漏洞，你可以编写计数字节数的代码了。在第九章，你将把这段代码作为模块在*markov_haiku.py*程序中使用。

### **计数字节数的代码**

本节包含了*count_syllables.py*程序的代码。你还需要在上一节中创建的*missing_words.json*文件。你可以从*[`www.nostarch.com/impracticalpython/`](https://www.nostarch.com/impracticalpython/)*下载这两个文件。将它们保存在同一个文件夹中。

#### ***准备、加载和计数***

清单 8-5 导入必要的模块，加载 CMUdict 和缺失单词字典，并定义一个函数来计数字节数。

*count_syllables.py*，第一部分

```py
   import sys
   from string import punctuation
   import json
   from nltk.corpus import cmudict

   # load dictionary of words in haiku corpus but not in cmudict
   with open('missing_words.json') as f:
       missing_words = json.load(f)

➊ cmudict = cmudict.dict()

➋ def count_syllables(words):
       """Use corpora to count syllables in English word or phrase."""
       # prep words for cmudict corpus
       words = words.replace('-', ' ')
       words = words.lower().split()
    ➌ num_sylls = 0
    ➍ for word in words:
           word = word.strip(punctuation)
           if word.endswith("'s") or word.endswith("’s"):
               word = word[:-2]
        ➎ if word in missing_words:
               num_sylls += missing_words[word]
           else:
            ➏ for phonemes in cmudict[word][0]:
                   for phoneme in phonemes:
                    ➐ if phoneme[-1].isdigit():
                           num_sylls += 1

    ➑ return num_sylls
```

*清单 8-5：导入模块、加载词典并计数字节数*

在导入一些熟悉的模块后，加载包含所有 CMUdict 中缺失的单词和音节计数的*missing_words.json*文件。使用`json.load()`恢复存储为字符串的字典。接下来，使用`dict()`方法将 CMUdict 语料库转化为字典➊。

定义一个名为`count_syllables()`的函数来计数字节数。它应该能够接受单词*和*短语，因为你最终会希望将它传递给俳句的每一行。按照之前在*missing_words_finder.py*程序中做的那样准备单词➋。

分配一个`num_sylls`变量来保存音节计数，并将其初始化为`0` ➌。然后开始循环输入的单词，去掉末尾的标点符号和*'s*。注意，撇号的格式可能会让你感到困扰，因此提供了两种版本：一种是直撇号，另一种是弯撇号 ➍。接下来，检查单词是否是缺失单词的小字典中的成员。如果找到了该单词，则将该单词在字典中的值加到`num_sylls`中 ➎。否则，开始查看音素，这些音素在 CMUdict 中表示一个值；对于每个音素，查看它所组成的字符串 ➏。如果在字符串的末尾找到数字，那么你就知道这个音素是元音。以单词*aged*为例，只有第一个字符串（在此处以灰色突出显示）末尾有数字，因此该单词包含一个元音：

```py
[['EY1', 'JH', 'D'], ['EY1', 'JH', 'IH0', 'D']]
```

请注意，如果有多个发音，你使用第一个值（`[0]`）；记住，CMUdict 将每个发音表示为一个嵌套列表。这可能会偶尔导致错误，因为正确的选择取决于上下文。

检查音素的末尾是否有数字，如果有，则将`1`加到`num_sylls`中 ➐。最后，返回单词或短语的总音节数 ➑。

#### ***定义 main() 函数***

在完成程序后，清单 8-6 定义并运行了 `main()` 函数。程序在以独立模式运行时会调用此函数——例如，用于检查单词或短语——但如果你将 `syllable_counter` 作为模块导入，则不会调用此函数。

*count_syllables.py,* 第二部分

```py
   def main():
    ➊ while True:
           print("Syllable Counter")
        ➋ word = input("Enter word or phrase; else press Enter to Exit: ")
        ➌ if word == '':
               sys.exit()
        ➍ try:
               num_syllables = count_syllables(word)
               print("number of syllables in {} is: {}"
                     .format(word, num_syllables))
               print()
           except KeyError:
               print("Word not found.  Try again.\n", file=sys.stderr)
➎ if __name__ == '__main__':
       main()
```

*清单 8-6：定义并调用* main() *函数*

定义`main()`函数并开始`while`循环 ➊。提示用户输入一个单词或短语 ➋。如果用户按下 ENTER 键但没有输入任何内容，程序将退出 ➌。否则，启动一个`try`-`except`块，以防用户输入的单词在字典中找不到而导致程序崩溃 ➍。只有在独立模式下，才应引发异常，因为你已经准备好让程序在没有异常的情况下运行在 haiku-training 语料库上。在此块内，调用`count_syllables()`函数并传递输入，然后在交互式命令行中显示结果。最后使用标准代码，使程序能够作为独立程序运行或作为其他程序中的模块 ➎。

### **检查你的程序**

你已经精心调整了音节计数程序，确保它能与训练语料库配合使用。在继续进行 haiku 程序时，你可能想要将一两首诗加入到这个语料库中，但添加新的 haiku 可能会引入一个新的单词，而这个单词不在 CMUdict 或你的异常字典中。在你回去重新构建异常字典之前，请检查是否真的需要这样做。

清单 8-7 将自动计算训练语料库中每个单词的音节数，并显示失败的单词。你可以从* [`www.nostarch.com/impracticalpython/`](https://www.nostarch.com/impracticalpython/)*下载这个程序，文件名为*test_count_syllables_w_full_corpus.py*。将其保存在与*count_syllables.py*、*train.txt*和*missing_words.json*相同的文件夹中。

*test_count_syllables_w_full_corpus.py*

```py
   import sys
   import count_syllables

   with open('train.txt.') as in_file:
    ➊ words = set(in_file.read().split())

➋ missing = []

➌ for word in words:
       try:
           num_syllables = count_syllables.count_syllables(word)
           ##print(word, num_syllables, end='\n') # uncomment to see word counts
    ➍ except KeyError:
           missing.append(word)

➎ print("Missing words:", missing, file=sys.stderr)
```

*清单 8-7：尝试计算训练语料库中单词的音节数，并列出所有失败的情况*

打开更新后的*train.txt*训练语料库，并将其加载为集合，以删除重复项 ➊。创建一个空列表，命名为`missing`，用于存储无法计数音节的新单词 ➋。`missing`中的单词将不会出现在 CMUdict 或你的`missing_words`字典中。

遍历新训练语料库中的单词 ➌，并使用`try-except`块处理当*count_syllables.py*找不到单词时抛出的`KeyError` ➍。将该单词追加到`missing`列表中，然后显示该列表 ➎。

如果程序显示一个空列表，则说明新哈库中的所有单词已经存在于 CMUdict 或*missing_words.json*中，因此不需要进行任何调整。否则，你可以选择手动将单词添加到*missing_words.json*文件中，或者重新运行*missing_words_finder.py*来重建*missing_words.json*。

### **总结**

在本章中，你已经学习了如何下载 NLTK 并使用其中一个数据集——卡内基梅隆发音词典（CMUdict）。你将 CMUdict 数据集与哈库训练语料库进行了比对，并为任何缺失的单词构建了一个支持的 Python 字典。你使用 JavaScript 对象表示法（JSON）将这个 Python 字典保存为持久数据。最后，你编写了一个可以计算音节的程序。在第九章中，你将使用你的音节计数程序帮助生成新的哈库诗。

### **进一步阅读**

*虚拟缪斯：计算机诗歌实验*（卫斯理大学出版社，1996 年）由查尔斯·O·哈特曼（Charles O. Hartman）编写，是一本生动有趣的书，探讨了人类与计算机在早期合作写诗的过程。

《使用 Python 进行自然语言处理：使用自然语言工具包分析文本》（O'Reilly，2009）由史蒂文·伯德（Steven Bird）、厄恩·克莱因（Ewan Klein）和爱德华·洛珀（Edward Loper）编写，是一本易于入门的 NLP 书籍，使用 Python 进行操作，包含大量练习和有用的与 NLTK 网站的集成。该书的新版已更新至 Python 3 和 NLTK 3，并可在线访问，网址为* [`www.nltk.org/book/`](http://www.nltk.org/book/)*。

斯蒂芬·F·德安吉利斯（Stephen F. DeAngelis）的《自然语言处理的重要性日益增长》是*Wired*杂志上一篇关于自然语言处理（NLP）在大数据中日益扩展角色的文章。在线版本可以在* [`www.wired.com/insights/2014/02/growing-importance-natural-language-processing/`](https://www.wired.com/insights/2014/02/growing-importance-natural-language-processing/)*上查看。

### **实践项目：音节计数器与词典文件**

编写一个 Python 程序，允许你测试 *count_syllables.py*（或任何其他音节计数 Python 代码）与词典文件的对比。在允许用户指定要检查多少个单词后，随机选择单词并在不同的行上显示每个单词及其音节数。输出应类似于以下打印结果：

```py
ululation 4
intimated 4
sand 1
worms 1
leatherneck 3
contenting 3
scandals 2
livelihoods 3
intertwining 4
beaming 2
untruthful 3
advice 2
accompanying 5
deathly 2
hallos 2
```

可下载的词典文件列在表格 2-1 中，位于第 20 页。你可以在附录中找到解决方案，并且可以从 *[`www.nostarch.com/impracticalpython/`](https://www.nostarch.com/impracticalpython/)* 下载，文件名为 *test_count_syllables_w_dict.py*。
