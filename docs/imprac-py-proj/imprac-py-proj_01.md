## **1

傻瓜名字生成器**

![image](../images/common01.jpg)

美国电视网曾播出一部侦探喜剧节目《*Psych*》，剧中，超观察力的业余侦探肖恩·斯宾塞在假装自己拥有超能力的同时解决案件。节目的一大特色是他总是用即兴编造的傻乎乎的名字来介绍他的搭档 Gus，比如 Galileo Humpkins、Lavender Gooms 和 Bad News Marvin Barnes。正如肖恩可能会说的，有些名字傻得足以让你脑溢血，但绝对不是好那种！

### **项目 #1：生成化名**

在这个热身项目中，你将编写一个简单的 Python 程序，通过随机组合名字和姓氏来生成有趣的名字。希望你能够创造出一堆让任何助手都引以为傲的别名。你还将回顾最佳实践的编码指南，并应用外部程序，帮助你编写符合这些指南的代码。

*Psych* 不是你的菜？可以在我的代码列表中替换掉名字，用你自己的笑话或主题来替代。你也可以轻松地把这个项目变成一个 *权力的游戏* 名字生成器，或者你想发现自己独一无二的“本尼迪克特·康伯巴奇”名字；我最喜欢的是 Bendylick Cricketbat。

**目标**

使用符合已建立风格指南的 Python 代码随机生成有趣的助手名字。

### **规划与设计项目**

规划时间从来都不是浪费时间。不论你是为了娱乐还是盈利而编程，最终你都需要相对准确地估算项目的完成时间、可能遇到的障碍以及所需的工具和资源。为了做到这一点，你首先需要知道自己究竟想要创造什么！

一位成功的经理曾告诉我，他的秘诀就是不断提问：*你*想做什么？*为什么*要这么做？为什么要*以这种方式*做？你有多少*时间*？你有多少*资金*？回答这些问题对设计过程非常有帮助，它能让你有清晰的方向。

在他的书《*Think Python, 2nd Edition*》（O'Reilly, 2015）中，艾伦·道尼描述了两种软件开发计划：“原型和补丁”和“设计开发”。在原型和补丁的方式中，你从一个简单的程序开始，然后使用*补丁*（即修改过的代码）来解决测试中遇到的问题。当你面对一个自己不太理解的复杂问题时，这种方法可以是一个不错的选择。但它也可能产生复杂且不可靠的代码。如果你对问题及其解决方案有清晰的理解，那么你应该使用设计开发计划，以避免未来出现问题并需要修补。这种方法能让编码更加轻松高效，通常也能产生更强大且可靠的代码。

对于本书中的所有项目，你将从一个明确定义的问题或目标开始，这将形成你设计决策的基础。然后，我们将讨论策略，以更好地理解问题并制定开发计划。

#### ***策略***

你将从两个列表开始——一个是名字，另一个是姓氏——它们包含一些有趣的名字。由于列表相对较短，不会占用太多内存，不需要动态更新，也不会引发运行时问题。由于你只需要从列表中读取名字，因此可以使用元组作为容器。

使用你那两个名字的元组，你将生成新的名字——将名字与姓氏配对——只需点击按钮即可。这样，用户可以轻松地重复这一过程，直到生成一个足够有趣的名字。

你还应该以某种方式在解释器窗口中突出显示名字，以使其从命令提示符中脱颖而出。IDLE shell 提供的字体选项不多，但你可能已经知道——太清楚了——错误会以红色显示。print() 函数的默认输出是标准输出，但通过加载 sys 模块，你可以使用 file 参数将输出重定向到错误通道，并带有标志性的红色：print(something, file=sys.stderr)。

最后，你将确定针对 Python 编程的风格建议。这些指导方针不仅应涵盖代码，还应涵盖嵌入代码中的文档。

#### ***伪代码***

“你总是可以指望美国人在尝试了其他一切方法后，做出正确的决定。” 这句话，虽然与温斯顿·丘吉尔有所关联，但恰当地总结了许多人写伪代码的方式。

*伪代码* 是一种高层次、非正式的方式，用结构化的英语或任何人类语言来描述计算机程序。它应当类似于简化的编程语言，包含关键字和适当的缩进。开发人员使用它来忽略真正编程语言中的所有复杂语法，专注于底层逻辑。尽管它被广泛使用，但伪代码没有官方标准——只有指导方针。

如果你发现自己在不断尝试中感到沮丧，可能是因为你没有花时间写伪代码。我是伪代码的坚定信徒，因为它在我迷失方向时总能——毫不例外——引导我找到解决方案。因此，你将在本书中的大多数项目中使用某种形式的伪代码。至少，我希望你能看到它的实用性，但我也希望你能培养在自己项目中写伪代码的习惯。

一个非常高层次的伪代码，适用于我们的有趣名字生成器，可能看起来像这样：

加载一个名字列表

加载一个姓氏列表

随机选择一个名字

将名字分配给一个变量

随机选择一个姓氏

将名字分配给一个变量

将名字按顺序以红色字体打印到屏幕上

询问用户是否退出或重新开始

如果用户重新开始：

重复

如果用户退出：

结束并退出

除非你是在通过编程课程或者给别人提供清晰的指导，否则应该专注于伪代码的*目的*；不必过于拘泥于（非标准的）写作规范。并且，不要止步于编程——你可以将伪代码的过程应用到更多的方面。一旦掌握了它，你可能会发现它能帮助你完成其他任务，比如做税务申报、规划投资、建房子，或者准备露营旅行。这是一个帮助你集中思考的好方法，并能将编程上的成功转化到现实生活中。如果国会也能使用它就好了！

### **代码**

[Listing 1-1](ch01.xhtml#ch01list1) 是有趣名字生成器的代码，*pseudonyms.py*，它通过两个名字元组编译并打印出一系列化名。如果你不想输入所有的名字，你可以输入一个子集，或者从 *[https://nostarch.com/impracticalpython/](https://nostarch.com/impracticalpython/)* 下载代码。

*pseudonyms.py*

➊ import sys, random

➋ print("欢迎来到心理学 '助手名字选择器。'\n")

print("一个像肖恩为古斯选的名字：\n\n")

first = ('婴儿油', '坏消息', '大打嗝', "比尔 '比尼-维尼'",

"Bob 'Stinkbug'", '肠胃声音', 'Boxelder', "Bud 'Lite' ",

'Butterbean', 'Buttermilk', 'Buttocks', 'Chad', 'Chesterfield',

'Chewy', 'Chigger', "肉桂包", 'Cleet', '玉米饼', '蟹肉',

'Crapps', 'Dark Skies', 'Dennis Clawhammer', 'Dicman', 'Elphonso',

'Fancypants', 'Figgs', 'Foncy', 'Gootsy', 'Greasy Jim', 'Huckleberry',

'Huggy', 'Ignatious', 'Jimbo', "Joe '种植土'", 'Johnny',

'Lemongrass', 'Lil Debil', 'Longbranch', '"午餐钱"',

'Mergatroid', '"Mr Peabody"', '油罐', 'Oinks', 'Old Scratch',

'Ovaltine', 'Pennywhistle', 'Pitchfork Ben', '土豆虫',

'Pushmeet', 'Rock Candy', 'Schlomo', 'Scratchensniff', 'Scut',

"Sid 'The Squirts'", 'Skidmark', 'Slaps', 'Snakes', 'Snoobs',

'Snorki', 'Soupcan Sam', 'Spitzitout', 'Squids', 'Stinky',

'Storyboard', '甜茶', 'TeeTee', 'Wheezy Joe',

"Winston 'Jazz Hands'", 'Worms')

last = ('Appleyard', 'Bigmeat', 'Bloominshine', 'Boogerbottom',

'Breedslovetrout', 'Butterbaugh', 'Clovenhoof', 'Clutterbuck',

'Cocktoasten', 'Endicott', 'Fewhairs', 'Gooberdapple', 'Goodensmith',

'Goodpasture', 'Guster', 'Henderson', 'Hooperbag', 'Hoosenater',

'Hootkins', 'Jefferson', 'Jenkins', 'Jingley-Schmidt', 'Johnson',

'Kingfish', 'Listenbee', "M'Bembo", 'McFadden', 'Moonshine', 'Nettles',

'Noseworthy', 'Olivetti', 'Outerbridge', 'Overpeck', 'Overturf',

'Oxhandler', 'Pealike', 'Pennywhistle', 'Peterson', 'Pieplow',

'Pinkerton', 'Porkins', 'Putney', 'Quakenbush', 'Rainwater',

'Rosenthal', 'Rubbins', 'Sackrider', 'Snuggleshine', 'Splern',

'Stevens', 'Stroganoff', 'Sugar-Gold', 'Swackhamer', 'Tippins',

'Turnipseed', 'Vinaigrette', 'Walkingstick', 'Wallbanger', 'Weewax',

'Weiners', 'Whipkey', 'Wigglesworth', 'Wimplesnatch', 'Winterkorn',

'Woolysocks')

➌ while True:

➍ firstName = random.choice(first)

➎ lastName = random.choice(last)

print("\n\n")

➏ print("{} {}".format(firstName, lastName), file=sys.stderr)

print("\n\n")

➐ try_again = input("\n\n再试一次？（按 Enter 键继续，按 n 键退出）\n ")

if try_again.lower() == "n":

break

➑ input("\n按 Enter 键退出。")

*列表 1-1：从名字元组生成傻乎乎的化名*

首先，导入 sys 和 random 模块 ➊。你将使用 sys 来访问特定于系统的错误消息功能，这样你就可以在 IDLE 窗口中将输出设置为引人注目的红色。而 random 模块让你可以从名字列表中随机选择元素。

➋ 处的 print 语句向用户介绍程序。换行命令 \n 强制换行，单引号 '' 让你在打印输出时使用引号，而无需使用反斜杠转义字符，这样可以提高代码的可读性。

接下来，定义你的名字元组。然后启动 while 循环 ➌。设置 while = True 基本上意味着“继续运行，直到我告诉你停止”。最终，你会使用 break 语句来结束循环。

循环从随机选择第一个元组中的一个名字开始，然后将该名字赋值给变量 firstName ➍。它使用 random 模块的 choice 方法，从非空序列中返回一个随机元素——在这个例子中，就是从名字元组中随机选择一个名字。

接下来，从最后一个元组中随机选择一个姓氏并将其赋值给变量 lastName ➎。现在你已经有了两个名字，打印它们，并通过在 print 语句中提供可选参数 file=sys.stderr，让 IDLE 使用红色“错误”字体。使用较新的字符串格式*方法*，而不是旧的字符串格式*运算符*（%），将名字变量转换为字符串。要了解更多关于新方法的信息，请参阅 *[https://docs.python.org/3.7/library/string.html](https://docs.python.org/3.7/library/string.html)*。

一旦名字显示出来，询问用户是否选择继续玩或退出，使用 input 提供带引号的提示信息。在这种情况下，还要包含几行空白，以使得 IDLE 窗口中的搞笑名字更加突出。如果用户按下 ENTER 键，则 try_again 变量 ➐ 中不会返回任何值。没有返回值时，if 语句的条件不成立，while 循环继续执行，一个新名字会被打印出来。如果用户按下 N 键，则 if 语句会触发 break 命令，循环结束，因为 while 语句不再计算为 True。使用小写字符串方法 .lower() 可以避免玩家开启大写锁定键。换句话说，无论用户输入小写字母还是大写字母 N，程序都会将其视为小写字母。

最后，提示用户按 Enter 键退出 ➑。按 Enter 键不会将 input() 的返回值赋给变量，程序结束，控制台窗口关闭。在 IDLE 编辑器窗口中按 F5 执行完成的程序。

这段代码可以运行，但仅仅能运行是不够的——Python 程序应该以*风格*来运行。

#### ***使用Python社区的风格指南***

根据*Python的禅意*（* [https://www.python.org/dev/peps/pep-0020/](https://www.python.org/dev/peps/pep-0020/)*），“应该有一种——而且最好只有一种——显而易见的方式来做某件事。”为了提供一种显而易见的“正确方式”并围绕这些做法达成共识，Python社区发布了*Python增强提案*，这些是Python代码的编码规范，涵盖了主Python发行版中的标准库。最重要的提案是*PEP 8*，它是Python编程的风格指南。随着新的规范的出现和旧的规范因语言变化而过时，PEP 8也在不断发展。

PEP 8（* [https://www.python.org/dev/peps/pep-0008/](https://www.python.org/dev/peps/pep-0008/) *）为命名规范、空白行、制表符和空格的使用、最大行长、注释等设置了标准。目标是提高代码的可读性，并使其在广泛的Python程序中保持一致。当你开始编程时，你应该努力学习并遵循这些公认的规范，避免养成坏习惯。本文中的代码将严格遵守PEP 8，但我在某些规范上做了一些调整（例如，使用较少注释的代码、更少的空白行和更简短的文档字符串），以尊重出版行业的要求。

当你在跨职能团队中工作时，标准化的命名和程序尤其重要。在科学家和工程师之间的翻译中，很容易出现误解，就像1999年工程师因不同团队使用不同的测量单位而失去了火星气候轨道器一样。近二十年来，我一直在构建地球的计算机模型，然后将这些模型传递给工程功能。工程师使用脚本将这些模型加载到他们自己的专有软件中。为了提高效率并帮助那些经验不足的人，他们会在项目之间共享这些脚本。由于这些“命令文件”是根据每个项目定制的，因此当模型更新时，属性名称发生变化时，工程师们自然会感到不满。事实上，他们的一个内部指南是：“用一切办法迫使模型设计师使用一致的属性名称！”

#### ***使用Pylint检查你的代码***

你应该熟悉PEP 8，但你仍然会犯错误，而且将你的代码与指南进行比较是一个大麻烦。幸运的是，像Pylint、pycodestyle和Flake8这样的程序可以帮助你轻松遵循PEP 8的风格建议。在这个项目中，你将使用Pylint。

##### **安装Pylint**

Pylint 是 Python 编程语言的源代码、bug 和质量检查工具。要下载免费的副本，请访问 *[https://www.pylint.org/#install](https://www.pylint.org/#install)* 并找到适用于你平台的安装按钮。该按钮将显示安装 Pylint 的命令。例如，在 Windows 中，转到包含 Python 的文件夹（例如 *C:\Python35*），使用 SHIFT + 右键单击以打开上下文菜单，然后点击 **在此处打开命令窗口** 或 **在此处打开 PowerShell 窗口**，具体取决于你使用的 Windows 版本。运行 pip install pylint（如果安装了 Python 2 和 3，则使用 pip3）。

##### **运行 Pylint**

在 Windows 中，Pylint 从命令窗口或较新的系统中的 PowerShell 运行（你可以通过在包含你要检查的 Python 模块的文件夹中使用 SHIFT + 右键单击来打开这两者）。输入 pylint 文件名以运行程序（见 [图 1-1](ch01.xhtml#ch01fig1)）。*.py* 扩展名是可选的，你的目录路径会有所不同。对于 macOS 或其他基于 Unix 的系统，可以使用终端模拟器。

![image](../images/f0007-01.jpg)

*图 1-1：Windows 命令窗口，显示运行 Pylint 的命令*

命令窗口将显示 Pylint 的结果。以下是一个有用的输出示例：

C:\Python35\Python 3 Stuff\Psych>pylint pseudonyms.py

未找到配置文件，使用默认配置

************* 模块 pseudonyms

C: 45, 0: 关键字参数赋值周围不允许有空格

print(firstName, lastName, file = sys.stderr)

^ (bad-whitespace)

C:  1, 0: 缺少模块文档字符串 (missing-docstring)

C:  2, 0: 一行中有多个导入（sys, random）(multiple-imports)

C:  7, 0: 无效的常量名称 "first" (invalid-name)

C: 23, 0: 无效的常量名称 "last" (invalid-name)

C: 40, 4: 无效的常量名称 "firstName" (invalid-name)

C: 42, 4: 无效的常量名称 "lastName" (invalid-name)

C: 48, 4: 无效的常量名称 "try_again" (invalid-name)

每行开头的字母是消息代码。例如，C: 15, 0 表示第 15 行，第 0 列的编码标准违规。你可以参考以下关键字来理解不同的 Pylint 消息代码：

**R** 重构，表示“良好实践”违规

**C** 编码标准违规

**W** 风格问题或小的编程问题的警告

**E** 错误，表示重要的编程问题（即，最可能是 bug）

**F** 致命错误，表示错误会阻止进一步处理

Pylint 会在报告的最后给出你程序遵循 PEP 8 的评分。在这种情况下，你的代码得分为 4/10：

全局评估

-----------------

你的代码评分为 4.00/10（上次运行：4.00/10，+0.00）

##### **处理常量名称错误**

你可能已经注意到，Pylint错误地假设全局空间中的所有变量名都指代常量，因此应该使用全大写字母。你可以通过多种方式解决这个问题。第一种方法是将代码嵌入到一个main()函数中（如[示例 1-2](ch01.xhtml#ch01list2)所示）；这样，它就不再属于全局空间。

def main():

some indented code

some indented code

some indented code

➊ if __name__ == "__main__":

➋ main()

*示例 1-2: 定义并调用一个* main() *函数*

__name__ 变量是一个特殊的内置变量，你可以用它来判断程序是以独立模式运行还是作为导入模块运行；记住，模块只是另一个Python程序中使用的Python程序。如果直接运行该程序，__name__ 会被设置为 __main__。在[示例 1-2](ch01.xhtml#ch01list2)中，__name__ 被用来确保当程序被导入时，main()函数不会自动运行，直到你有意调用它；但当你直接运行程序时，if语句中的条件会被满足➊，main()会自动被调用➋。你并不总是需要遵循这一约定。例如，如果你的代码只是定义一个函数，你可以将它作为模块加载并调用，而不需要使用 __name__。

让我们把除导入语句之外的所有内容都嵌入到*pseudonyms.py*中，放在一个main()函数下，并将main()函数调用嵌入到一个if语句中，如[示例 1-2](ch01.xhtml#ch01list2)所示。你可以自己进行修改，也可以从网站上下载*pseudonyms_main.py*程序。重新运行Pylint。你应该会在命令窗口中看到以下结果。

C:\Python35\Python 3 Stuff\Psych>pylint pseudonyms_main

未找到配置文件，使用默认配置

************* 模块 pseudonyms_main

C: 47, 0: 关键字参数赋值周围不允许有空格

print(firstName, lastName, file = sys.stderr)

^ (bad-whitespace)

C:  1, 0: 缺少模块文档字符串 (missing-docstring)

C:  2, 0: 一行中有多个导入 (sys, random) (multiple-imports)

C:  4, 0: 缺少函数文档字符串 (missing-docstring)

C: 42, 8: 无效的变量名 "firstName" (invalid-name)

C: 44, 8: 无效的变量名 "lastName" (invalid-name)

现在关于无效常量名的烦人注释已经消失了，但你还没有完全解决问题。尽管我喜欢它们，Python的约定不允许使用*驼峰命名法*，比如firstName。

##### **配置Pylint**

在评估小脚本时，我更喜欢使用Pylint的默认设置，并忽略那些错误的“常量名称”错误。我还喜欢运行选项 -rn（即 -reports=n），以抑制Pylint返回的大量无关统计信息：

C:\Python35\Python 3 Stuff\Psych>pylint -rn pseudonyms_main.py

请注意，使用 -rn 将禁用代码评分选项。

Pylint的另一个问题是，它的默认最大行长度是100个字符，但PEP 8推荐79个字符。为了遵循PEP 8，你可以使用以下选项运行Pylint：

C:\Python35\Python 3 Stuff\Psych>pylint --max-line-length=79 pseudonyms_main

现在你会看到，缩进 main() 函数的名称导致一些行超出了规范要求：

C: 12, 0: 行太长 (80/79) (line-too-long)

C: 14, 0: 行太长 (83/79) (line-too-long)

--snip--

你可能不希望每次运行 Pylint 时都配置它，幸运的是，你不必这么做。相反，你可以使用`--generate-rcfile`命令创建你自己的自定义配置文件。例如，要禁止报告并将最大行长度设置为 79，请在命令提示符下输入以下内容：

your pathname>pylint -rn --max-line-length=79 --generate-rcfile > name.pylintrc

将你想要的更改放在`--generate-rcfile > name.pylintrc`语句之前，并在`.pylintrc`扩展名之前提供你自己的名称。你可以单独创建一个配置文件，如前所示，或者在评估 Python 程序时同时创建配置文件。`*.pylintrc*`文件会自动保存在当前工作目录中，但也可以选择添加目录路径（有关更多细节，请参见 *[https://pylint.org](https://pylint.org)* 和 *[https://pylint.readthedocs.io/en/latest/user_guide/run.html](https://pylint.readthedocs.io/en/latest/user_guide/run.html)*）。

若要重新使用你的自定义配置文件，使用`--rcfile`选项，后跟你个人配置文件的名称以及你正在评估的程序的名称。例如，要对`pseudonyms_main.py`程序运行`myconfig.pylintrc`，请输入以下命令：

C:\Python35\Python 3 Stuff\Psych>pylint --rcfile myconfig.pylintrc pseudonyms_main

#### ***使用文档字符串描述你的代码***

Pylint 识别到 *pseudonyms_main.py* 程序缺少文档字符串。根据 PEP 257 风格指南 (*[https://www.python.org/dev/peps/pep-0257/](https://www.python.org/dev/peps/pep-0257/)*)，文档字符串是出现在模块、函数、类或方法定义的第一条语句中的字符串字面量。文档字符串基本上是对代码功能的简短描述，它可能包括代码的特定方面，例如所需的输入。在此，使用三引号，下面是一个函数的单行文档字符串示例：

def circ(r):

"""返回半径为 r 的圆的周长。"""

c = 2 * r * math.pi

return c

前面的文档字符串只是简单地说明了函数的作用，但文档字符串可以更长，包含更多的信息。例如，以下是一个多行文档字符串，描述了相同函数的输入和输出：

def circ(r):

"""返回半径为 r 的圆的周长。

参数：

r – 圆的半径

返回：

float: 圆的周长

"""

c = 2 * r * math.pi

return c

不幸的是，文档字符串是与个人、项目和公司相关的内容，你会找到许多相互矛盾的指导意见。Google 有自己的格式和一个优秀的风格指南。科学界的一些成员使用 NumPy 的文档字符串标准。而 *reStructuredText* 是一种常用格式，主要与 Sphinx 一起使用——Sphinx 是一个生成 Python 项目的文档（如 HTML 和 PDF 格式）工具，使用文档字符串来生成文档。如果你曾经阅读过 Python 模块的文档（*[https://readthedocs.org/](https://readthedocs.org/)）），那么你一定见过 Sphinx 的应用。你可以在 “[进一步阅读](ch01.xhtml#lev22)” 中找到一些不同风格的指南链接，见 [第 14 页](ch01.xhtml#page_14)。

你可以使用一个名为 pydocstyle 的免费工具来检查你的文档字符串是否符合 PEP 257 的规范。在 Windows 或任何其他操作系统上安装它，只需打开命令窗口并运行 pip install pydocstyle（如果同时安装了 Python 2 和 3，请使用 pip3）。

要运行 pydocstyle，请在包含你要检查的代码的文件夹中打开命令窗口。如果你没有指定文件名，pydocstyle 会在文件夹中的 *所有* Python 程序上运行并给出反馈：

C:\Python35\Python 3 Stuff\Psych>pydocstyle

.\OLD_pseudonyms_main.py:1 在模块级别：

D100：公共模块缺少文档字符串

.\OLD_pseudonyms_main.py:4 在公共函数`main`中：

D103：公共函数缺少文档字符串

.\ pseudonyms.py:1 在模块级别：

D100：公共模块缺少文档字符串

.\ pseudonyms_main_broken.py:1 在模块级别：

D200：单行文档字符串应当适合一行并带引号（发现 2 个问题）

.\ pseudonyms_main_broken.py:6 在公共函数`main`中：

D205：摘要行与描述之间需要 1 行空白行

（未发现问题）

如果你指定的文件没有文档字符串问题，pydocstyle 将不会返回任何内容：

C:\Python35\Python 3 Stuff\Psych>pydocstyle pseudonyms_main_fixed.py

C:\Python35\Python 3 Stuff\Psych>

在本书中的所有项目中，我将使用相对简单的文档字符串，以减少注释代码时的视觉噪声。如果你想练习，随时可以扩展这些文档字符串。你可以随时使用 pydocstyle 检查你的结果。

#### ***检查你的代码风格***

当我长大的时候，我的叔叔会从我们的小镇开车去大城市去做“造型”。我一直不明白这与普通理发有何不同，但我知道如何“造型”我们的有趣名字生成器代码，以便使其符合 PEP 8 和 PEP 257。

创建一个名为 *pseudonyms_main_fixed.py* 的副本，并立即使用以下命令通过 Pylint 对其进行评估：

your_path>pylint --max-line-length=79 pseudonyms_main_fixed

不要使用 -rn 抑制报告。你应该能在命令窗口底部看到以下输出：

全局评估

-----------------

你的代码被评为 3.33/10

现在根据 Pylint 输出修正代码。在下面的示例中，我已经提供了以粗体表示的修正内容。我对名字元组进行了更改，以纠正行长问题。你也可以从本书资源中的 *[https://www.nostarch.com/impracticalpython/](https://www.nostarch.com/impracticalpython/)* 下载修正后的代码 *pseudonyms_main_fixed.py*。

*pseudonyms_main_fixed.py*

"""通过随机组合来自两个独立列表的名字来生成有趣的名字。"""

import sys

import random

def main():

"""从两个元组中随机选择名字并打印到屏幕上。"""

print("欢迎来到心理学 '搭档名字挑选器.'\n")

print("就像肖恩为古斯挑选的名字：\n\n")

first = ('婴儿油', '坏消息', '大打嗝', "比尔 '比尼-威尼'",

"鲍勃 '臭虫'", '肠道声音', '箱枯虫', "巴德 '轻型'",

'黄油豆', '黄油奶', '臀部', '查德', '切斯特菲尔德',

'嚼', '疥螨', '肉桂包', '克利特', '玉米面包',

'蟹肉', '克拉普斯', '黑暗天空', '丹尼斯·克劳汉默',

'迪克曼', '埃尔冯索', '花哨裤', '菲格斯', '方西', '古茨',

'油腻吉姆', '哈克贝里', '哈吉', '伊格内修斯', '吉博',

"乔 '土壤壶'", '约翰尼', '柠檬草', '小魔鬼',

'长枝', '"午餐钱"', '梅尔加特罗伊德', '"皮博迪先生"',

'油罐', '猪叫', '老魔鬼', '欧瓦尔廷', '口哨',

'叉子本', '土豆虫', '普什米特', '岩糖',

'施洛莫', '刮痕芬', '斯库特', "西德 '喷射'",

'滑标', '拍击', '蛇', '斯诺布', '斯诺基', '汤罐萨姆',

'吐出来', '鱿鱼', '臭味', '故事板', '甜茶',

'TeeTee', '呼吸困难乔', "温斯顿 '爵士手'", '虫子')

last = ('苹果园', '大肉', '繁花', '粪底',

'布里德斯洛夫特劳特', '巴特巴赫', '克洛文胡夫', '克拉特巴克',

'鸡尾酒烤面包', '恩迪科特', '费尔赫尔', '古伯达普尔',

'古登史密斯', '古德帕斯特', '古斯特', '亨德森', '胡普巴格',

'胡森纳特', '胡特金斯', '杰斐逊', '詹金斯',

'金吉利-施密特', '约翰逊', '国王鱼', '莱森比', "姆本博",

'麦克法登', '月光', '荨麻', '诺斯沃西', '奥利维蒂',

'外桥', '奥弗派克', '奥夫特夫', '奥克斯汉德勒', '皮尔如',

'口哨', '彼得森', '皮普洛', '平克顿', '波金斯',

'普特尼', '夸肯布什', '雨水', '罗森塔尔', '鲁宾斯',

'萨克赖德', '斯纳格尔欣', '斯普伦', '史蒂文斯', '斯特罗根诺夫',

'糖金', '斯瓦卡默', '蒂平斯', '芜菁种子',

'醋汁', '步行棒', '墙撞者', '威瓦克', '香肠',

'威普基', '威戈尔斯沃思', '温普尔斯内奇', '温特科恩',

'毛袜')

while True:

first_name = random.choice(first)

last_name = random.choice(last)

print("\n\n")

# 使用“致命错误”设置来欺骗 IDLE，使名字显示为红色。

print("{} {}".format(first_name, last_name), file=sys.stderr)

print("\n\n")

try_again = input("\n\n再试一次吗？（按 Enter 键继续，否则按 n 退出）\n ")

if try_again.lower() == "n":

break

input("\n按 Enter 键退出。")

if __name__ == "__main__":

main()

Pylint 给修订后的代码打了 10 分（满分）：

Global evaluation

-----------------

你的代码得分为 10.00/10（上次得分：3.33/10，+6.67）

正如你在上一节中看到的，运行 pydocstyle 对 *pseudonyms_main_fixed.py* 进行检查不会产生错误，但不要被这种情况误导，认为它的代码就好或者足够好。例如，以下文档字符串也通过了检查："""ksjkdls lskjds kjs jdi wllk sijkljs dsdw noiu sss。"""

编写简洁、简练且真正有用的文档字符串和注释是很难的。PEP 257 将帮助你编写文档字符串，但注释更自由、没有明确的限制。过多的注释会产生视觉噪声，可能让用户感到厌烦，并且通常不必要，因为写得好的代码基本上就是自文档化的。添加注释的好理由包括澄清意图和防止潜在的用户错误，例如当需要特定的测量单位或输入格式时。要找到合适的注释平衡点，可以参考你遇到的好的示例。此外，想一想如果你在五年后重新查看自己的代码时，想看到的是什么！

Pylint 和 pydocstyle 很容易安装，运行起来也很简单，它们将帮助你学习并遵守 Python 社区接受的编码标准。在将代码发布到网络论坛之前，通过 Pylint 运行代码也是一个好习惯，这样在寻求帮助时，可能会得到“更加友善”的回应！

### **总结**

现在你应该知道如何编写符合 Python 社区期望的代码和文档。更重要的是，你已经为一个配角、黑帮分子、线人等角色起了一些非常有趣的名字。以下是我最喜欢的一些名字：

| Pitchfork Ben Pennywhistle | ‘Bad News’ Bloominshine |
| --- | --- |
| Chewy Stroganoff | ‘Sweet Tea’ Tippins |
| Spitzitout Winterkorn | Wheezy Joe Jenkins |
| ‘Big Burps’ Rosenthal | Soupcan Sam Putney |
| Bill ‘Beenie-Weenie’ Clutterbuck | Greasy Jim Wigglesworth |
| Dark Skies Jingley-Schmidt | Chesterfield Walkingstick |
| Potato Bug Quakenbush | Jimbo Woolysocks |
| Worms Endicott | Fancypants Pinkerton |
| Cleet Weiners | Dicman Overpeck |
| Ignatious Outerbridge | Buttocks Rubbins |

### **进一步阅读**

要查看这些资源的可点击版本，请访问 *[https://www.nostarch.com/impracticalpython/](https://www.nostarch.com/impracticalpython/)*。

#### ***伪代码***

一些相当正式的伪代码标准的描述可以在*[http://users.csc.calpoly.edu/~jdalbey/SWE/pdl_std.html](http://users.csc.calpoly.edu/~jdalbey/SWE/pdl_std.html)* 和 *[http://www.slideshare.net/sabiksabz/pseudo-code-basics/](http://www.slideshare.net/sabiksabz/pseudo-code-basics/)* 找到。

#### ***风格指南***

以下是你可以参考的 Python 程序风格指南列表。

+   PEP 8 风格指南可以在 *[https://www.python.org/dev/peps/pep-0008/](https://www.python.org/dev/peps/pep-0008/)* 找到。

+   PEP 257 指南可以在 *[https://www.python.org/dev/peps/pep-0257/](https://www.python.org/dev/peps/pep-0257/)* 找到。

+   Google 有自己的格式和风格指南，网址是 *[https://google.github.io/styleguide/pyguide.html](https://google.github.io/styleguide/pyguide.html)*。

+   Google 风格的示例可以在 *[https://sphinxcontrib-napoleon.readthedocs.io/en/latest/example_google.html](https://sphinxcontrib-napoleon.readthedocs.io/en/latest/example_google.html)* 找到。

+   NumPy docstring 标准可以在 *[https://numpydoc.readthedocs.io/en/latest/](https://numpydoc.readthedocs.io/en/latest/)* 找到。

+   NumPy docstring 的示例可以在 *[https://sphinxcontrib-napoleon.readthedocs.io/en/latest/example_numpy.html](https://sphinxcontrib-napoleon.readthedocs.io/en/latest/example_numpy.html)* 找到。

+   你可以在 *[https://docs.python.org/devguide/documenting.html](https://docs.python.org/devguide/documenting.html)*、 *[https://docs.python.org/3.1/documenting/rest.html](https://docs.python.org/3.1/documenting/rest.html)* 和 *[https://wiki.python.org/moin/reStructuredText/](https://wiki.python.org/moin/reStructuredText/)* 找到有关 reStructuredText 的信息。

+   *《Python旅途指南》* (*[http://docs.python-guide.org/en/latest/writing/style/](http://docs.python-guide.org/en/latest/writing/style/)*) 包含了一部分关于代码风格和 *autopep8* 的内容。

    将自动为代码重新格式化以符合 PEP 8（到一定程度）。

+   Brett Slatkin的《*有效的Python*》（Addison-Wesley, 2015）中有一部分内容讲述了如何编写程序文档。

#### ***第三方模块***

以下是一些关于使用第三方模块的资源。

+   Pylint 的详细信息可以在 *[https://docs.pylint.org/en/1.8/tutorial.html](https://docs.pylint.org/en/1.8/tutorial.html)* 找到。

+   关于 pydocstyle 的详细信息可以在 *[http://www.pydocstyle.org/en/latest/](http://www.pydocstyle.org/en/latest/)* 找到。

### **实践项目**

试试这些处理字符串的项目。我的解决方案可以在附录中找到。

#### ***猪拉丁语***

要形成猪拉丁语，你需要拿一个以辅音字母开头的英文单词，将这个辅音字母移到单词的末尾，然后在单词末尾加上“ay”。如果单词以元音字母开头，只需在单词末尾加上“way”。最著名的猪拉丁语短语之一是“ixnay on the ottenray”，这是Marty Feldman在梅尔·布鲁克斯的喜剧经典 *《年轻的弗兰肯斯坦》* 中说的。

编写一个程序，接受一个单词作为输入，并使用索引和切片返回其猪拉丁语等价词。运行 Pylint 和 pydocstyle 检查你的代码，并修正任何风格错误。你可以在附录中找到解决方案，或者从 *[https://www.nostarch.com/impracticalpython/](https://www.nostarch.com/impracticalpython/)* 下载 *pig_latin_practice.py*。

#### ***穷人条形图***

英语中最常用的六个字母可以通过助记词“etaoin”来记住（发音为*eh-tay-oh-in*）。编写一个Python脚本，接受一个句子（字符串）作为输入，并返回一个简单的条形图样式的显示，如[图1-2](ch01.xhtml#ch01fig2)所示。提示：我使用了一个字典数据结构和两个我还未讲解过的模块，pprint和collections/defaultdict。

![image](../images/f0016-01.jpg)

*图1-2：附录中* ETAOIN_practice.py *程序的条形图样式输出*

### **挑战项目**

挑战项目不提供解决方案。你需要独立完成这些！

#### ***贫穷外国人的条形图***

使用在线翻译工具将你的文本转换为另一种基于拉丁字母的写作系统（如西班牙语或法语），重新运行来自贫穷人条形图的代码，并比较结果。例如，西班牙语版本的[图1-2](ch01.xhtml#ch01fig2)中的文本生成了[图1-3](ch01.xhtml#ch01fig3)中的结果。

![image](../images/f0016-02.jpg)

*图1-3：运行* EATOIN_challenge.py *在西班牙语翻译文本中的结果，见[图1-2](ch01.xhtml#ch01fig2)*

西班牙语句子中出现的*L*字母是英文的两倍，*U*字母是英文的三倍。为了使不同输入的条形图可以直接比较，修改代码，使每个字母都有一个键，即使没有值也会显示出来。

#### ***中间部分***

重写有趣名字生成器代码，加入中间名。首先，创建一个新的中间名元组，然后拆分现有的名字-中间名组合（如“Joe ‘Pottin Soil’”或“Sid ‘The Squirts’”），并将其添加到元组中。你还应该将一些显而易见的昵称（如“Oil Can”）移动到中间名元组中。最后，添加一些新的中间名（如“The Big News”，“Grunts”或“Tinkie Winkie”）。使用Python的random模块，使得中间名的出现概率为一半或三分之一。

#### ***完全不同的东西***

开始自己有趣名字的列表，并将其添加到有趣名字生成器中。提示：电影片头字幕是一个丰富的猎取来源！
