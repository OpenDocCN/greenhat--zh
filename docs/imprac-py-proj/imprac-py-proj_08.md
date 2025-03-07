## **8

**为俳句诗歌计算音节**

![image](../images/common01.jpg)

诗歌可能是文学的最高形式。正如柯尔律治所说：“最好的词汇排列在最佳的顺序中。”诗人必须以极其简练的方式讲述一个故事，提出一个观点，描述一个场景，或激发强烈的情感，同时遵循严格的节奏和韵律、风格和结构的规则。

计算机喜欢规则和结构，甚至有潜力唤起情感。在他1996年的书《虚拟缪斯：计算机诗歌实验》中，作者查尔斯·哈特曼描述了早期编写能够模仿人类诗歌的算法尝试。引用哈特曼的话：“诗歌互动的复杂性，诗人、文本和读者之间微妙的舞蹈，造成了一种犹豫不决的游戏。在这个游戏中，经过恰当编程的计算机有机会插入一些有趣的动作。”

哈特曼描述的早期程序最多只能生成糟糕的“嬉皮”诗歌。那个时候的目标是“将计算出的机械化无序引入语言，将结果重新放回语言生活的偶然世界，看看灰尘如何沉淀。”正如我们在几章中提到的，编程中像是人名字谜和空白密码之类的情境问题是薄弱环节。要写出符合文学“至高”标准的计算机诗歌，你不能忽视情境。

让计算机模拟这种最具人类特征的行为是一个令人着迷的挑战——而且显然是我们不能放弃的。在本章和下一章中，你将教会你的计算机如何生成一种传统的日本诗歌形式，称为*俳句*。

### **日本俳句**

俳句由三行组成，分别为五、七、五个音节。诗歌很少押韵，主题通常直接或间接地涉及自然界——主要是四季。如果做得恰当，俳句能让你沉浸在场景中，仿佛唤起了记忆。

我在这里提供了三个俳句的例子。第一个是大师芭蕉（1715–1783）的作品，第二个是大师一茶（1763–1828）的作品，第三个则是我自己创作的，基于童年旅行的回忆。

傍晚时分站立

听吧……在遥远的地方

小青蛙的歌声！

*—芭蕉*

好朋友蝗虫

你会当那守墓人吗

为我的小坟墓？

*—一茶*

远方的云层

我让自己假装

远方的山脉

*—沃恩*

由于其唤起性的特质，每首俳句都有一个内建的“可利用空白”供程序员使用。彼得·贝伦森在他1955年出版的《*日本俳句*》一书中很好地总结了这一点：“俳句并不要求总是一个完整甚至清晰的陈述。读者应该将自己的联想和形象加到这些词语中，从而成为自己在诗歌中享受的共同创作者。”哈特曼补充道：“读者的大脑在稀缺材料上工作最为活跃。我们从最少的星星中画出最清晰的星座。因此，对于一小段能够赋予形象意义的词语，无意义的因素很少。”简单来说，短诗更难出错。读者总是假设诗人有意图，如果找不到，他们会自己编造一个。

尽管如此，将计算机训练成能够写诗并非易事，你将需要两个完整的章节来完成。在本章中，你将编写一个程序来计算单词和短语中的音节数，以便尊重俳句的音节结构。在[第9章](ch09.xhtml#ch09)中，你将使用一种叫做*马尔可夫链分析*的技术来捕捉俳句的精髓——那种难以捉摸、引人入胜的成分——并将现有的诗歌转变成新的东西，偶尔，甚至可以说更好。

### **项目 #15：音节计数**

用英语计算音节是困难的。问题在于，正如查尔斯·哈特曼所说，英语的拼写奇特且语言历史错综复杂。例如，像*aged*这样的词语，根据是描述一个人还是描述奶酪，可能是一音节或两音节。那么，如何写一个程序，在不陷入无休止的特殊情况列表的情况下准确地计算音节呢？

答案是它做不到，至少没有一个“备忘单”。幸运的是，正因为有一种叫做*自然语言处理*（*NLP*）的科学分支，这些备忘单才得以存在。自然语言处理研究计算机精确且结构化的语言与人类使用的充满细微差别和经常模糊的“自然”语言之间的互动。自然语言处理的应用示例包括机器翻译、垃圾邮件检测、搜索引擎问题的理解以及手机用户的预测文本识别。自然语言处理的最大影响还在未来：它将开采大量以前无法使用的、结构不良的数据，并与我们的计算机“统治者”进行无缝对话。

在本章中，你将使用一个自然语言处理数据集来帮助计算单词或短语中的音节数。你还将编写代码，找到这个数据集中缺失的单词，并帮助你构建一个支持字典。最后，你将编写一个程序，帮助你检查音节计数代码。在[第9章](ch09.xhtml#ch09)中，你将把这个音节计数算法作为一个模块，应用到一个帮助你计算机化创作文学最高成就——诗歌——的程序中。

**目标**

编写一个 Python 程序，计算英语单词或短语中的音节数。

### **策略**

对你和我来说，计算音节很简单。把手背放在下巴下面，然后开始说话。每次下巴碰到手背时，就说明你说出了一个音节。计算机没有手和下巴，但每个元音音素代表一个音节——而计算机可以计算元音音素。然而，这并不简单，因为没有简单的规则可以做到这一点。一些书面语言中的元音是静音的，比如 *like* 中的 *e*，而有些元音组合发出一个音，比如 *moo* 中的 *oo*。幸运的是，英语语言中的单词数量并不是无限的。可以找到相当全面的列表，涵盖了你需要的大部分信息。

*语料库* 是一组文本的高级名称。在 [第 9 章](ch09.xhtml#ch09) 中，你将使用一个 *训练语料库*——由俳句组成——教 Python 如何创作新的俳句。在这一章中，你将使用同一个语料库来提取音节计数。

你的音节计数器应评估短语和单个单词，因为你最终将用它来计算整行 *俳句* 的音节数。该程序将接受文本输入，计算每个单词的音节数，并返回总的音节数。你还需要处理标点符号、空格和缺失单词等问题。

你需要遵循的主要步骤是：

1.  下载一个包含音节计数信息的大型语料库。

1.  将音节计数语料库与俳句训练语料库进行比较，找出音节计数语料库中缺失的所有单词。

1.  构建一个字典，包含缺失单词及其音节计数。

1.  编写一个程序，利用音节计数语料库和缺失单词字典来计算训练语料库中的音节数。

1.  编写一个程序，检查音节计数程序与训练语料库的更新是否一致。

#### ***使用语料库***

自然语言工具包（NLTK）是一个流行的程序和库套件，用于在 Python 中处理人类语言数据。它于 2001 年作为宾夕法尼亚大学计算机与信息科学系的一门计算语言学课程的一部分创建。经过众多贡献者的帮助，开发和扩展持续进行。如果你想了解更多，可以访问官方 NLTK 网站 *[http://www.nltk.org/](http://www.nltk.org/)*。

在这个项目中，你将使用 NLTK 访问卡内基梅隆大学发音字典（CMUdict）。这个语料库包含了近 125,000 个单词及其对应的发音。它是机器可读的，适用于语音识别等任务。

#### ***安装 NLTK***

你可以在 *[http://www.nltk.org/install.html](http://www.nltk.org/install.html)* 找到在 Unix、Windows 和 macOS 上安装 NLTK 的说明。如果你使用的是 Windows，我建议你首先打开 Windows 命令提示符或 PowerShell，并尝试使用 pip 安装：

python -m pip install nltk

你可以通过打开 Python 交互式 shell 并输入以下命令来检查安装情况：

>>> import nltk

>>>

如果没有遇到错误，你就可以开始使用了。否则，请按照刚才引用的网页上的说明进行操作。

#### ***正在下载 CMUdict***

要访问 CMUdict（或其他任何 NLTK 语料库），你需要先下载它。你可以使用方便的 NLTK 下载器来实现这一点。一旦你安装了 NLTK，在 Python shell 中输入以下内容：

>>> import nltk

>>> nltk.download()

现在应该打开 NLTK 下载器窗口（[图 8-1](ch08.xhtml#ch08fig1)）。点击顶部附近的**Corpora**标签，然后在标识符列中点击**cmudict**。接下来，滚动到窗口底部并设置下载目录；我使用了默认目录，*C:\nltk_data*。最后，点击**Download**按钮加载 CMUdict。

![image](../images/f0149-01.jpg)

*图 8-1：带有选中 cmudict 下载项的 NLTK 下载器窗口*

当 CMUdict 下载完成后，退出下载器并在 Python 交互式 shell 中输入以下内容：

>>> from nltk.corpus import cmudict

>>>

如果没有遇到错误，那么语料库已经成功下载。

#### ***按音素计数而非音节计数***

CMUdict 语料库将单词分解为一组*音素*——在指定语言中感知上不同的*声音单位*——并使用数字（0、1 和 2）标记元音的词汇重音。CMUdict 语料库将每个元音标记为其中一个数字，而且只有一个数字，因此你可以使用这些数字来识别单词中的元音。

将单词视为一组音素将帮助你避免一些问题。首先，CMUdict 不会在书写的单词中包含不发音的元音。例如，CMUdict 如何处理单词*scarecrow*：

[['S', 'K', 'AE1', 'R', 'K', 'R', 'OW0']]

每个带有数字后缀的项目代表一个发音的元音。请注意，*scare* 结尾的静音*e*被正确省略了。

第二，有时多个连续的书写元音会被发音为单一的音素。例如，CMUdict 如何表示*house*：

[['HH', 'AW1', 'S']]

注意语料库如何将书写中的双元音*ou*当作一个元音，'AW1'，来处理发音。

#### ***处理具有多种发音的单词***

正如我在介绍中提到的，一些单词有多种不同的发音；*aged* 和 *learned* 只是两个例子：

[['EY1', 'JH', 'D'], ['EY1', 'JH', 'IH0', 'D']]

[['L', 'ER1', 'N', 'D'], ['L', 'ER1', 'N', 'IH0', 'D']]

注意嵌套的列表。语料库识别到这两个单词可以发音为一个或两个音节。这意味着它会为某些单词返回多个音节计数，代码中需要考虑这一点。

### **管理缺失的单词**

CMUdict非常有用，但对于一个语料库，单词要么在，要么不在。仅仅在一个1500字的测试案例中，就花了几秒钟找到了50多个缺失的单词——如*dewdrop*、*bathwater*、*dusky*、*ridgeline*、*storks*、*dragonfly*、*beggar*和*archways*——这些单词在CMUdict中找不到。所以，你的策略之一应该是检查CMUdict中缺失的单词，然后通过为你的语料库制作一个语料库来弥补这些遗漏！

#### ***训练语料库***

在[第9章](ch09.xhtml#ch09)中，你将使用包含几百首俳句的训练语料库来“教”你的程序如何生成新的俳句。但是你不能指望CMUdict包含这个语料库中的所有单词，因为其中有些是日语单词，比如*sake*。正如你之前所看到的，即便是一些常见的英语单词，也没有出现在CMUdict中。

所以首先要做的就是检查训练语料库中所有单词是否在CMUdict中。为此，你需要下载名为*train.txt*的训练语料库，从*[https://www.nostarch.com/impracticalpython/](https://www.nostarch.com/impracticalpython/)*获取。把它和本章所有的Python程序放在同一文件夹中。该文件包含不到300首俳句，这些俳句被随机复制了大约20次，以确保训练集的多样性。

一旦你发现CMUdict中没有的单词，你就需要编写一个脚本，帮助你准备一个以单词为键，音节数为值的Python字典；然后你将把这个字典保存到一个支持CMUdict的文件中，以便在音节计数程序中使用。

#### ***缺失单词代码***

本节中的代码将找到CMUdict中缺失的单词，帮助你准备一个包含这些单词及其音节数的字典，并将字典保存到文件中。你可以从*[https://nostarch.com/impracticalpython/](https://nostarch.com/impracticalpython/)*下载代码，文件名为*missing_words_finder.py*。

##### **导入模块，加载CMUdict，并定义main()函数**

[代码清单8-1](ch08.xhtml#ch08list1)导入模块，加载CMUdict，并定义运行程序的main()函数。

*missing_words_finder.py,* 第1部分

import sys

from string import punctuation

➊ import pprint

import json

from nltk.corpus import cmudict

➋ cmudict = cmudict.dict()  # 卡内基梅隆大学发音词典

➌ def main():

➍ haiku = load_haiku('train.txt')

➎ exceptions = cmudict_missing(haiku)

➏ build_dict = input("\n是否手动构建异常字典（y/n）？\n")

如果build_dict.lower() == 'n'：

sys.exit()

否则：

➐ missing_words_dict = make_exceptions_dict(exceptions)

save_exceptions(missing_words_dict)

*代码清单8-1：导入模块，加载CMUdict，并定义* main()

你从一些熟悉的导入开始，并增加了几个新的。pprint模块允许你以易于阅读的格式“漂亮地打印”缺失词汇的字典 ➊。你将使用JavaScript对象表示法（json）将这个字典写出为持久化数据，这是一种基于文本的计算机数据交换格式，非常适合Python数据结构；它是标准库的一部分，在多种语言中都有标准化，且数据既安全又易于人类阅读。最后，导入CMUdict语料库。

接下来，调用cmudict模块的dict()方法，将语料库转换为一个字典，词汇作为键，音标作为值 ➋。

定义main()函数，调用函数加载训练语料库，查找CMUdict中缺失的词汇，构建一个包含词汇及其音节计数的字典，并保存结果 ➌。你将在定义main()之后定义这些函数。

调用函数加载俳句训练语料库，并将返回的集合赋值给名为haiku的变量 ➍。然后调用另一个函数，找到缺失的词汇并将其返回为一个集合 ➎。使用集合可以去除不需要的重复词汇。cmudict_missing()函数还会显示缺失词汇的数量以及其他一些统计信息。

现在，询问用户是否希望手动构建一个字典以解决缺失的词汇，并将他们的输入赋值给build_dict变量 ➏。如果他们想停止，退出程序；否则，调用一个函数来构建字典 ➐，然后再调用另一个函数来保存字典。请注意，用户并不限于按y键继续，尽管这是提示信息。

##### **加载训练语料库并查找缺失的词汇**

[列表 8-2](ch08.xhtml#ch08list2)加载并准备训练语料库，将其内容与CMUdict进行比较，并跟踪差异。这些任务分为两个函数。

*missing_words_finder.py,* 第2部分

➊ def load_haiku(filename):

"""打开并返回作为集合的俳句训练语料库。"""

with open(filename) as in_file:

➋ haiku = set(in_file.read().replace('-', ' ').split())

➌ return haiku

def cmudict_missing(word_set):

"""查找并返回在词汇集内但缺失于cmudict的词汇。"""

➍ exceptions = set()

for word in word_set:

word = word.lower().strip(punctuation)

if word.endswith("'s") or word.endswith("’s"):

word = word[:-2]

➎ if word not in cmudict:

exceptions.add(word)

print("\n异常词汇:")

print(*exceptions, sep='\n')

➏ print("\n俳句语料库中的唯一词汇数量 = {}"

.format(len(word_set)))

print("语料库中不在cmudict中的词汇数量 = {}"

.format(len(exceptions)))

membership = (1 - (len(exceptions) / len(word_set))) * 100

➐ print("cmudict匹配度 = {:.1f}{}".format(membership, '%'))

return exceptions

*列表 8-2：定义了加载语料库的函数，并找到缺失的词汇*

定义一个函数来读取俳句训练语料库中的单词➊。*train.txt*中的俳句被多次重复，而且原始俳句中有重复的词汇，如*moon*、*mountain*和*the*。没有必要重复评估一个词汇，因此将这些词汇作为集合加载，以去除重复项➋。你还需要将连字符替换为空格。连字符在俳句中很常见，但你需要将连字符两侧的词分开，以便在CMUdict中进行查找。函数结束时返回俳句集合➌。

现在是时候查找缺失的词汇了。定义一个函数cmudict_missing()，该函数以一个序列作为参数——在本例中是由load_haiku()函数返回的词汇集合。创建一个空集合exceptions，用于保存任何缺失的词汇 ➍。遍历俳句集合中的每个词，将其转换为小写，并去除任何前后的标点符号。请注意，不要去除除了连字符以外的内部标点，因为CMUdict会识别像*wouldn’t*这样的词汇。所有格的词通常不在语料库中，因此需要去掉末尾的*’s*，因为这不会影响音节数。

**注意**

*小心词处理软件产生的弯引号（’）。这些与简单文本编辑器和Shell中使用的直引号（*'*）不同，可能不会被CMUdict识别。如果你在训练文件或JSON文件中添加新词汇，请确保使用直引号来表示缩写或所有格名词。*

如果在CMUdict中找不到该词，请将其添加到异常词汇表➎。打印这些词作为检查，并附上基本信息➏，例如有多少个唯一词汇，有多少个缺失的词汇，以及CMUdict中包含的训练语料库的百分比。将百分比的精度设置为小数点后一位➐。函数的结束返回异常词汇表。

##### **构建缺失词汇的字典**

[Listing 8-3](ch08.xhtml#ch08list3)继续了*missing_words_finder.py*的代码，现在通过将音节数作为值分配给缺失的词汇，补充了CMUdict。由于缺失词汇的数量应该相对较少，用户可以手动分配音节数，因此编写代码帮助他们与程序交互。

*missing_words_finder.py,* 第三部分

➊ def make_exceptions_dict(exceptions_set):

"""返回一个包含词汇和音节数的字典，来自一个词汇集合。"""

➋ missing_words = {}

print("输入词汇的音节数。错误可以在最后更正。\n")

for word in exceptions_set:

while True:

➌ num_sylls = input("请输入{}的音节数：".format(word))

➍ if num_sylls.isdigit():

break

else:

print("                 无效的答案！", file=sys.stderr)

➎ missing_words[word] = int(num_sylls)

print()

➏ pprint.pprint(missing_words, width=1)

➐ print("\n在保存之前是否要修改字典？")

print("""

0 - 退出并保存

1 – 添加词汇或更改音节数

2 - 删除词汇

""")

➑ while True:

choice = input("\n请输入选项：")

if choice == '0':

break

elif choice == '1':

word = input("\n要添加或更改的单词：")

missing_words[word] = int(input("请输入 {} 的音节数："

.format(word)))

elif choice == '2':

word = input("\n请输入要删除的单词：")

➒ missing_words.pop(word, None)

print("\n新增单词或音节更改：")

➓ pprint.pprint(missing_words, width=1)

return missing_words

*清单 8-3：允许用户手动计数音节并构建字典*

首先定义一个函数，接受由 cmudict_missing() 函数返回的异常集合作为参数 ➊。立即将一个空字典分配给名为 missing_words 的变量 ➋。告知用户如果他们犯了错，他们稍后会有机会修正；然后，使用 for 和 while 循环遍历缺失单词集，并展示每个单词给用户，要求输入音节数。单词将作为字典的键，而 num_sylls 变量将成为其值 ➌。如果输入是数字 ➍，则退出循环。否则，警告用户并让 while 循环再次请求输入。如果输入通过，则将值作为整数添加到字典中 ➎。

使用 pprint 将每个键/值对显示在单独的一行，以进行检查。width 参数充当换行符的作用 ➏。

给用户提供在保存缺失单词字典为文件之前进行最后修改的机会 ➐。使用三引号展示选项菜单，然后通过 while 循环保持选项活动，直到用户准备好保存 ➑。这三个选项分别是退出，触发 break 命令；添加新单词或更改现有单词的音节数，要求输入单词和音节数；以及删除条目，使用字典的 pop() 函数 ➒。向 pop() 添加 None 参数意味着如果用户输入的单词不在字典中，程序不会引发 KeyError。

最后，给用户一个最后查看字典的机会，以防有更改 ➓，然后返回字典。

##### **保存缺失单词字典**

*持久数据*是指程序终止后仍然保留的数据。为了让缺失单词字典能够在你稍后编写的 *count_syllables.py* 程序中使用，你需要将其保存到一个文件中。[清单 8-4](ch08.xhtml#ch08list4) 正是完成了这一操作。

*missing_words_finder.py,* 第四部分

➊ def save_exceptions(missing_words):

"""将异常字典保存为 json 文件。"""

➋ json_string = json.dumps(missing_words)

➌ f = open('missing_words.json', 'w')

f.write(json_string)

f.close()

➍ print("\n文件已保存为 missing_words.json")

➎ if __name__ == '__main__':

main()

*清单 8-4：将缺失单词字典保存为文件并调用* main()

使用json保存字典。定义一个新函数，以缺失单词的集合作为参数 ➊。将missing_words字典分配给一个名为json_string的新变量 ➋；然后，打开一个*.json*扩展名的文件 ➌，写入json变量，并关闭文件。以文件名提醒用户 ➍。最后，使用代码让程序可以作为模块或独立模式运行 ➎。

json.dumps()方法将missing_words字典序列化为字符串。*序列化*是将数据转换为更易传输或存储的格式的过程。例如：

>>> import json

>>> d = {'scarecrow': 2, 'moon': 1, 'sake': 2}

>>> json.dumps(d)

'{"sake": 2, "scarecrow": 2, "moon": 1}'

请注意，序列化字典由单引号括起来，这使其成为一个字符串。

我在这里提供了*missing_words_finder.py*的部分输出。顶部是缺失单词列表，底部是手动填写的音节数，因简洁起见都已被缩短。

--省略--

小蛙

散布

文件重

木槿花

云状云

夜莺

俳句语料库中唯一单词的数量 = 1523

语料库中不在cmudict中的单词数量 = 58

cmudict成员身份 = 96.2%

手动构建异常字典（y/n）？

y

在樵夫中输入音节数：3

在牵牛花中输入音节数：4

在云状云中输入音节数：3

--省略--

别担心——你不需要为所有音节数分配值。*missing_words.json*文件已经完成，并且在需要时可以下载。

**注意**

*对于具有多种发音的单词，例如* jagged *或* our*，你可以通过手动打开* missing_words.json *文件并在任何位置添加键值对来强制程序使用你偏好的发音。我对单词*sake*做了这个处理，以便它使用两个音节的日语发音。因为首先在这个文件中检查单词的成员身份，它将覆盖CMUdict中的值。*

现在你已经解决了CMUdict中的空白，准备编写计算音节数的代码。在[第9章](ch09.xhtml#ch09)，你将把这段代码作为模块在*markov_haiku.py*程序中使用。

### **音节计数代码**

本节包含*count_syllables.py*程序的代码。你还需要在上一节中创建的*missing_words.json*文件。你可以从*[https://www.nostarch.com/impracticalpython/](https://www.nostarch.com/impracticalpython/)*下载它们。将它们保存在同一文件夹中。

#### ***准备、加载和计数***

[清单 8-5](ch08.xhtml#ch08list5)导入所需的模块，加载CMUdict和缺失单词字典，并定义一个函数，用于计算给定单词或短语的音节数。

*count_syllables.py，* 第1部分

import sys

from string import punctuation

import json

from nltk.corpus import cmudict

# 加载不在cmudict中的俳句语料库单词字典

with open('missing_words.json') as f:

missing_words = json.load(f)

➊ cmudict = cmudict.dict()

➋ def count_syllables(words):

"""使用语料库计算英语单词或短语中的音节数。"""

# 为 cmudict 语料库准备单词

words = words.replace('-', ' ')

words = words.lower().split()

➌ num_sylls = 0

➍ 对于 words 中的每个单词：

word = word.strip(punctuation)

如果单词以 "'s" 或 "’s" 结尾：

word = word[:-2]

➎ 如果单词在 missing_words 中：

num_sylls += missing_words[word]

否则：

➏ 对于 cmudict[word][0] 中的音素：

对于每个音素：

➐ 如果音素[-1]是数字：

num_sylls += 1

➑ 返回 num_sylls

*列表 8-5：导入模块，加载字典，并计算音节数*

在一些常见的导入后，加载包含所有在 CMUdict 中缺失的单词和音节计数的 *missing_words.json* 文件。使用 json.load() 恢复存储为字符串的字典。接着，使用 dict() 方法将 CMUdict 语料库转换为字典 ➊。

定义一个名为 count_syllables() 的函数来计算音节数。它应该同时接受单词 *和* 短语，因为最终你希望将其用于处理俳句中的行。像之前在 *missing_words_finder.py* 程序中那样准备单词 ➋。

创建一个 num_sylls 变量来存储音节数，并将其初始化为 0 ➌。现在开始循环遍历输入的单词，去掉标点符号和 *’s*。请注意，可能会被撇号的格式所困扰，因此提供了两种格式：一种是直引号，另一种是弯引号 ➍。接下来，检查单词是否属于缺失单词的小字典。如果找到该单词，则将该字典中的值加到 num_sylls ➎ 中。否则，开始检查音素，它们表示 CMUdict 中的值；对于每个音素，查看组成它的字符串 ➏。 如果发现字符串末尾有数字，那么这个音素就是元音。以单词 *aged* 为例，只有第一个字符串（在此处以灰色突出显示）末尾带有数字，因此该单词包含一个元音：

[['EY1', 'JH', 'D'], ['EY1', 'JH', 'IH0', 'D']]

请注意，你使用第一个值（[0]），以防有多个发音；记住，CMUdict 将每种发音表示为一个嵌套列表。这可能会导致偶尔的错误，因为正确的选择将取决于上下文。

检查音素的结尾是否包含数字，如果包含，则将num_sylls ➐加1。最后，返回该单词或短语的总音节数 ➑。

#### ***定义 main() 函数***

完成程序后，[列表 8-6](ch08.xhtml#ch08list6) 定义并运行了 main() 函数。当程序以独立模式运行时，它将调用此函数——例如，检查某个单词或短语——但如果你将 syllable_counter 作为模块导入，它将不会被调用。

*count_syllables.py,* 第 2 部分

def main():

➊ while True:

print("音节计数器")

➋ word = input("请输入单词或短语；否则按 Enter 键退出：")

➌ 如果单词为空：

sys.exit()

➍ try:

num_syllables = count_syllables(word)

print("单词 {} 的音节数为：{}"

.format(word, num_syllables))

print()

except KeyError:

print("未找到单词。请再试一次。\n", file=sys.stderr)

➎ if __name__ == '__main__':

main()

*列出 8-6：定义并调用* main() *函数*

定义 main() 函数并启动一个 while 循环 ➊。要求用户输入一个单词或短语 ➋。如果用户按下 ENTER 键而没有输入内容，则程序退出 ➌。否则，启动一个 try-except 块，这样如果用户输入一个字典中没有的单词，程序就不会崩溃 ➍。只有在独立模式下才应抛出异常，因为你已经为该程序准备好了在 haiku 训练语料库中运行，无需任何异常。在这个块中，调用 count_syllables() 函数并传入输入，然后在交互式 shell 中显示结果。最后以标准代码结束，允许程序在独立模式或作为另一个程序的模块运行 ➎。

### **检查程序的程序**

你已仔细调整音节计数程序，确保它能够与训练语料库配合使用。在继续进行 haiku 程序时，你可能想要向此语料库添加一两首诗，但添加新的 haiku 可能会引入一个在 CMUdict 或你的例外字典中都没有的新单词。在你回去重新构建例外字典之前，请检查是否真的需要这么做。

[列出 8-7](ch08.xhtml#ch08list7) 将自动计算训练语料库中每个单词的音节数，并显示所有计算失败的单词。你可以从 *[https://www.nostarch.com/impracticalpython/](https://www.nostarch.com/impracticalpython/)* 下载此程序，命名为 *test_count_syllables_w_full_corpus.py*。将其保存在与 *count_syllables.py*、*train.txt* 和 *missing_words.json* 相同的文件夹中。

*test_count_syllables_w_full_corpus.py*

import sys

import count_syllables

with open('train.txt.') as in_file:

➊ words = set(in_file.read().split())

➋ missing = []

➌ for word in words:

try:

num_syllables = count_syllables.count_syllables(word)

##print(word, num_syllables, end='\n') # 取消注释以查看单词计数

➍ except KeyError:

missing.append(word)

➎ print("缺失的单词:", missing, file=sys.stderr)

*列出 8-7：尝试计算训练语料库中单词的音节并列出所有失败项*

打开你的更新后的 *train.txt* 训练语料库，并将其加载为一个集合，以去除重复项 ➊。启动一个空的列表，命名为 missing，用来存储任何无法计算音节的单词 ➋。missing 中的单词不会出现在 CMUdict 或你的 missing_words 字典中。

循环遍历新训练语料库中的单词 ➌，并使用 try-except 块来处理当 *count_syllables.py* 找不到单词时抛出的 KeyError ➍。将这个单词添加到 missing 列表中，然后显示该列表 ➎。

如果程序显示一个空列表，则表示新俳句中的所有单词已经出现在CMUdict或*missing_words.json*中，因此你不需要做任何调整。否则，你可以选择手动将单词添加到*missing_words.json*文件中，或者重新运行*missing_words_finder.py*来重建*missing_words.json*。

### **总结**

在本章中，你已经学会了如何下载NLTK并使用其数据集之一——卡内基梅隆发音词典（CMUdict）。你将CMUdict数据集与俳句训练语料库进行比对，并为任何缺失的单词构建了一个支持性的Python词典。你将该Python词典保存为持久数据，使用了JavaScript对象表示法（JSON）。最后，你编写了一个可以计算音节数的程序。在[第9章](ch09.xhtml#ch09)，你将使用你的音节计数程序来帮助你生成新的俳句。

### **进一步阅读**

查尔斯·O·哈特曼（Charles O. Hartman）的《虚拟缪斯：计算机诗歌实验》（Wesleyan University Press，1996年）深入探讨了人类与计算机在创作诗歌过程中的早期合作。

*Python自然语言处理：使用自然语言工具包分析文本*（O’Reilly，2009年）由Steven Bird、Ewan Klein和Edward Loper编写，是一本易于理解的关于使用Python进行NLP的入门书籍，包含大量练习并与NLTK网站有良好的集成。此书的新版本已更新为支持Python 3和NLTK 3，可在线阅读，地址为* [http://www.nltk.org/book/](http://www.nltk.org/book/)*。

斯蒂芬·F·迪安杰利斯（Stephen F. DeAngelis）的《自然语言处理日益重要性》是*Wired*杂志的一篇文章，讨论了NLP在大数据中的日益重要的角色。在线版可在* [https://www.wired.com/insights/2014/02/growing-importance-natural-language-processing/](https://www.wired.com/insights/2014/02/growing-importance-natural-language-processing/)*上查看。

### **实践项目：音节计数器与词典文件**

编写一个Python程序，允许你测试*count_syllables.py*（或任何其他音节计数Python代码）与词典文件的匹配。在允许用户指定要检查的单词数量后，随机选择单词并显示每个单词及其音节数的清单，每个单词和音节数应该显示在不同的行中。输出应类似于以下打印输出：

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

可下载的词典文件在[表2-1](ch02.xhtml#ch02tab1)中列出，位于[第20页](ch02.xhtml#page_20)。你可以在附录中找到一个解决方案，附录文件可以从* [https://www.nostarch.com/impracticalpython/](https://www.nostarch.com/impracticalpython/)*下载，文件名为*test_count_syllables_w_dict.py*。
