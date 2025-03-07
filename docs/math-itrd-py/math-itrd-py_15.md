## 12

使用遗传算法解决问题

*Steve: 我们迷路了。

Mike: 我们迷路到什么程度了？*

![image](img/fintro-01.jpg)

当许多人想到数学时，他们会想到那些“石刻般”的方程式和运算，以及那些非对即错的答案。他们可能会惊讶地发现，我们在代数探索中已经进行了许多猜测和检查。

在本章中，你将以一种间接的方式学习破解密码和隐藏信息。这有点像 第四章中的“猜测与检查”方法，我们将一堆整数代入方程式，如果有任何一个使方程成立，我们就输出它们。这一次，我们将猜测多个值，而不仅仅是一个。虽然这不是最优雅的解题方式，但有了计算机，有时候蛮力最有效。

为了找出我们的秘密短语，我们生成猜测，然后根据它们与目标的匹配程度进行评分。但这就是我们与猜测与检查方法不同之处：我们保留最好的猜测，并且随机地不断进行变异，直到揭开消息为止。程序不知道哪些字母是对的，哪些是错的，但我们通过不断变异我们到目前为止最好的猜测，越来越接近正确答案。尽管现在看起来这个方法可能并不令人乐观，但你会发现，它实际上能够出奇地快速地破解密码。这个方法叫做 *遗传算法*，计算机科学家使用它基于自然选择和进化生物学理论来寻找问题的解决方案。它的灵感来自于适应和变异的生物体，以及它们如何在微小的优势上逐步发展，正如我们在 第九章的《羊群模型》中看到的那样。

然而，对于更复杂的问题，仅靠随机变异并不足以解决问题。在这些情况下，我们加入 *交叉*，用来将最适应的个体（或最佳猜测）组合在一起，从而提高它们破解密码的可能性，就像最适应的生物更有可能传递他们基因的组合一样。除了评分外，所有这些活动都是相当随机的，因此可能会令人惊讶的是，我们的遗传算法竟然如此有效。

### 使用遗传算法来猜测短语

打开 IDLE，创建一个名为 *geneticQuote.py* 的新文件。与 第四章中通过猜测数字不同，这个程序尝试猜测一个秘密短语。我们只需要告诉程序它猜对了多少个字符——不是在哪个位置，也不是哪些字符，只是猜对了多少个字符。

我们的程序将能够做得比猜测短密码更好。

#### 编写 MAKELIST() 函数

为了了解这如何工作，让我们创建一个目标短语。这里是我儿子从漫画书 *火影忍者* 中想出来的一句长句子：

target = "我从不食言，因为那是我的忍者之道。"

在英语中，我们可以选择许多字符：小写字母、大写字母、空格和一些标点符号。

characters = " abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ.',?!"

我们创建一个名为 makeList() 的函数，它会随机生成一个与目标长度相同的字符列表。稍后，当我们尝试猜测目标短语时，我们会通过逐字符地将猜测与目标进行比较来打分。得分越高，说明猜测越接近目标。接着，我们会随机改变猜测中的一个字符，看是否能够提高其得分。看起来这种随机的方法似乎不可能得到确切的目标短语，但事实上它是会做到的。

首先，导入 random 模块并编写 makeList() 函数，如清单 12-1 所示。

*遗传

Quote.py*

导入 random 模块

target = "I never go back on my word, because that is my Ninja way."

characters = " abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ.',?!"

def makeList():

'''返回一个字符列表，长度与目标相同

与目标的长度相同'''

charList = [] # 用于填充随机字符的空列表

for i in range(len(target)):

charList.append(random.choice(characters))

return charList

*清单 12-1：编写 makeList() 函数来创建一个与目标长度相同的随机字符列表*

在这里，我们创建了一个空列表 charList，并按目标字符数的次数循环。在每次循环中，程序将 characters 中的一个随机字符放入 charList。一旦循环完成，它返回 charList。让我们测试一下，确保它能正常工作。

#### 测试 MAKELIST() 函数

首先，我们找出目标的长度，并检查我们的随机列表是否具有相同的长度：

>>> len(target)

57

>>> newList = makeList()

>>> newList

['p', 'H', 'Z', '!', 'R', 'i', 'e', 'j', 'c', 'F', 'a', 'u', 'F', 'y', '.',

'w', 'u', '.', 'H', 'W', 'w', 'P', 'Z', 'D', 'D', 'E', 'H', 'N', 'f', ' ',

'W', 'S', 'A', 'B', ',', 'w', '?', 'K', 'b', 'N', 'f', 'k', 'g', 'Q', 'T',

'n', 'Q', 'H', 'o', 'r', 'G', 'h', 'w', 'l', 'l', 'W', 'd']

>>> len(newList)

57

我们测量了目标列表的长度，它有 57 个字符。我们的新列表也是相同的长度，57 个字符。为什么要创建一个列表而不是字符串？我们创建列表是因为有时候列表比字符串更容易操作。例如，你不能简单地在字符串中替换一个字符为另一个字符。但在列表中，你是可以做到的，正如你在这里看到的：

>>> a = "Hello"

>>> a[0] = "J"

回溯（最近的调用最先）：

文件 "<pyshell#16>"，第 1 行，在 <module> 中

a[0] = "J"

TypeError: 'str' 对象不支持项赋值

>>> b = ["H","e","l","l","o"]

>>> b[0] = "J"

>>> b

['J', 'e', 'l', 'l', 'o']

在这个例子中，当我们尝试用 "J" 替换 "Hello" 字符串中的第一个字符时，Python 不允许我们这么做，并且会报错。然而，使用列表做同样的事情则没有问题。

在我们的 *geneticQuote.py* 程序中，我们希望将随机的名言以字符串的形式展示，因为那样更容易阅读。以下是如何使用 Python 的 join() 函数将列表打印为字符串：

>>> print(''.join(newList))

pHZ!RiejcFauFy.wu.HWwPZDDEHNf WSAB,w?KbNfkgQTnQHorGhwllWd

这些都是 newList 中的字符，但以字符串形式呈现。看起来并不是一个非常有前景的开始！

#### 编写 SCORE() 函数

现在我们来编写一个名为 score() 的函数，通过将每个猜测与目标逐字符进行比较来给出得分，像 列表 12-2 中那样。

*genetic

Quote.py*

def score(mylist):

'''返回一个整数：与目标的匹配数'''

matches = 0

for i in range(len(target)):

if mylist[i] == target[i]:

matches += 1

return matches

*列表 12-2：编写 score() 函数来评估猜测结果*

score() 函数接受我们提供的列表（mylist）中的每个项，并检查 mylist 的第一个字符是否与目标列表的第一个字符匹配。然后函数会检查第二个字符是否匹配，依此类推。对于每个匹配的字符，我们将 matches 增加 1。最后，这个函数返回一个数字，而不是哪些字符是正确的，因此我们实际上并不知道*哪些*字符是正确的！

我们的得分是多少？

>>> newList = makeList()

>>> score(newList)

0

我们的第一次尝试完全失败了。没有一个匹配！

#### 编写 MUTATE() 函数

现在我们来编写一个函数，通过随机改变一个字符来变异一个列表。这将允许我们的程序“猜测”直到接近我们想要猜测的目标短语。代码见 列表 12-3。

*genetic

Quote.py*

def mutate(mylist):

'''返回修改了一个字母的 mylist'''

newlist = list(mylist)

new_letter = random.choice(characters)

index = random.randint(0,len(target)-1)

newlist[index] = new_letter

return newlist

*列表 12-3：编写 mutate() 函数来更改列表中的一个字符*

首先，我们将列表的元素复制到一个名为 newlist 的变量中。然后我们从 characters 列表中随机选择一个字符作为新字母，替换掉现有字符中的一个。我们随机选择一个介于 0 和目标长度之间的数字作为替换字母的索引。然后我们将 newlist 中该索引处的字符替换为新字母。这个过程会在一个循环中不断重复。如果新列表的得分更高，它将成为“最佳”列表，而最佳列表将继续进行变异，希望能够进一步提高得分。

#### 生成随机数

在所有函数定义之后启动程序，我们通过调用 random.seed() 来确保随机性。调用 random.seed() 会将随机数生成器重置为当前时间。然后我们创建一个字符列表，并将第一个列表声明为最佳列表，因为它是到目前为止最好的。它的得分将是最高分。

*genetic

Quote.py*

random.seed()

bestList = makeList()

bestScore = score(bestList)

我们跟踪已经做出的猜测次数：

guesses = 0

现在我们开始一个无限循环，变异 bestList 以生成新的猜测。我们计算它的分数并增加 guesses 变量：

while True:

guess = mutate(bestList)

guessScore = score(guess)

guesses += 1

如果新猜测的分数小于或等于当前最佳分数，程序可以“继续”，如下所示。这意味着它将返回循环的开始，因为这不是一个好的猜测，我们不需要对其做其他处理。

if guessScore <= bestScore:

continue

如果我们仍然在循环中，说明这个猜测足够好，可以打印出来。我们也打印它的分数。我们可以打印列表（作为字符串）、分数以及做出的总猜测次数。如果新猜测的分数等于目标的长度，那么我们已经解决了这个问题，可以退出循环：

print(''.join(guess),guessScore,guesses)

if guessScore == len(target):

break

否则，新猜测必须比当前最好的列表更好，但还不完美，所以我们可以宣布它为最佳列表，并将其分数保存为最佳分数：

bestList = list(guess)

bestScore = guessScore

Listing 12-4 显示了 *geneticQuote.py* 程序的完整代码。

*genetic Quote.py*

import random

target = "I never go back on my word, because that is my Ninja way."

characters = " abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ.',?!"

#创建一个与目标长度相同的字符“猜测”列表的函数

def makeList():

'''返回一个字符列表，长度与目标相同

作为目标'''

charList = [] #空列表，用于填充随机字符

for i in range(len(target)):

charList.append(random.choice(characters))

return charList

#函数用于通过与目标比较来“评分”猜测列表

def score(mylist):

'''返回一个整数：与目标匹配的数量'''

matches = 0

for i in range(len(target)):

if mylist[i] == target[i]:

matches += 1

return matches

#函数用于通过随机更改一个字母来“变异”列表

def mutate(mylist):

'''返回我的列表，并将一个字母更改'''

newlist = list(mylist)

new_letter = random.choice(characters)

index = random.randint(0,len(target)-1)

newlist[index] = new_letter

return newlist

#创建一个列表，并将该列表设置为 bestList

#将 bestList 的分数设置为 bestScore

random.seed()

bestList = makeList()

bestScore = score(bestList)

guesses = 0

#创建一个无限循环来生成变异

#对 bestList 进行评分

while True:

guess = mutate(bestList)

guessScore = score(guess)

guesses += 1

#如果 newList 的分数低于 bestList，

#"继续"进入下一次循环迭代

if guessScore <= bestScore:

continue

#如果新列表的分数是最佳分数，

#打印列表并退出循环

print(''.join(guess),guessScore,guesses)

if guessScore == len(target):

break

#否则，将 bestList 设置为 newList 的值

#并且 bestScore 赋值为 newList 中分数的值

bestList = list(guess)

bestScore = guessScore

*列表 12-4：* geneticQuote.py *程序的完整代码*

现在当我们运行这个时，我们得到了一个非常快速的解决方案，所有提高分数的猜测都会被打印出来。

i.fpzgPG.'kHT!NW WXxM?rCcdsRCiRGe.LWVZzhJe zSzuWKV.FfaCAV 1 178

i.fpzgPG.'kHT!N  WXxM?rCcdsRCiRGe.LWVZzhJe zSzuWKV.FfaCAV 2 237

i.fpzgPG.'kHT!N  WXxM?rCcdsRCiRGe.LWVZzhJe zSzuWKV.FfwCAV 3 266

i fpzgPG.'kHT!N  WXxM?rCcdsRCiRGe.LWVZzhJe zSzuWKV.FfwCAV 4 324

--略--

我 nevgP 从不食言，因为那是我的忍者之道。55 8936

我 neveP 从不食言，因为那是我的忍者之道。56 10019

我从不食言，因为那是我的忍者之道。57 16028

这个输出显示最终分数是 57，并且总共进行了 16,028 次猜测才完全匹配引用。在输出的第一行中，注意到需要 178 次猜测才能得到 1 分！有更高效的方式来猜测一个引用，但我想通过一个简单的例子来介绍遗传算法的概念。重点是展示如何通过对猜测进行评分并随机变异“迄今为止最好的猜测”，在惊人的短时间内得出准确的结果。

现在，你可以使用这种对猜测进行评分和变异的思路来解决其他问题。

### 解决旅行推销员问题（TSP）

我的一个学生对引用猜测程序不感兴趣，因为“我们已经知道引用是什么了。”所以让我们使用遗传算法来解决一个我们还不知道解决方案的问题。*旅行推销员问题*，简称 *TSP*，是一个古老的智力题，容易理解但可能变得非常难以解决。一个推销员必须到达给定数量的城市，目标是找到最短的路线。听起来很简单吧？而且有了计算机，我们应该只需运行所有可能的路线，并测量它们的距离，对吧？

事实证明，超过某个城市数量后，即便是今天的超级计算机，计算复杂度也会变得过于庞大。让我们来看一下，当你有六个城市时，会有多少条可能的路线，正如在图 12-1 中所示。

![image](img/f254-01.jpg)

*图 12-1：* n *个城市之间路径的数量，* n *在 2 到 6 之间*

当城市数量为两三个时，只有一条可能的路线。加上第四个城市，它可以在之前的任何三个城市之间被访问，所以将前一个路线数量乘以 3。那么四个城市之间有三条可能的路线。再加上第五个城市，它可以在之前的四个城市之间被访问，所以是前一步的四倍，即有 12 条可能的路线。看出规律了吗？在 *n* 个城市之间，有

![image](img/e254-01.jpg)

可能的路线。比如，在 10 个城市之间有 181,440 种可能的路线。在 20 个城市之间，有 60,822,550,204,416,000 条路线。超过一万亿的是什么？即使一台计算机每秒能检查一百万条路线，也需要近 2000 年才能完成计算。对于我们的目的来说，这样的速度太慢了。必须有更好的方法。

#### 使用遗传算法

类似于我们的猜名言程序，我们将创建一个带有“基因”中路线的对象，并通过路线的长度来评分。最佳路线将随机变异，然后我们将对其变异进行评分。我们可以收集一堆“最佳路线”，将它们的列表拼接在一起，然后评分它们的“后代”。这次探索的最棒之处在于我们*并不知道*答案。我们可以给程序一组城市及其位置，或者让程序随机绘制城市并尝试优化路线。

打开一个新的 Processing 草图并命名为*travelingSalesperson.pyde*。我们首先要创建的是 City 对象。每个城市将有自己的 x 和 y 坐标，以及一个用于标识的编号。这样，我们就可以使用城市编号的列表来定义一条路线。例如，[5,3,0,2,4,1]表示你从城市 5 出发，前往城市 3，然后是城市 0，依此类推。规则是销售员最终必须返回到第一个城市。Listing 12-5 展示了 City 类。

*travelingSalesperson.pyde*

class City:

def __init__(self,x,y,num):

self.x = x

self.y = y

self.number = num #标识编号

def display(self):

fill(0,255,255) #天空蓝

ellipse(self.x,self.y,10,10)

noFill()

*Listing 12-5：为* travelingSalesperson.pyde *程序编写 City 类*

在初始化 City 时，我们获得 x 和 y 坐标，并为每个 City 对象赋予自己的 x 和 y 分量。我们还得到一个编号，这是城市的标识号。在 display()方法中，我们选择一个颜色（此例为天空蓝），并在该位置创建一个椭圆。绘制完城市后，我们使用 noFill()函数关闭填充，因为其他形状无需用颜色填充。

让我们确保它有效。我们来创建 setup()函数，为显示窗口指定大小，并创建 City 类的一个实例。记住，我们需要为它提供两个坐标的位置以及一个标识编号，如 Listing 12-6 所示。

def setup():

size(600,600)

background(0)

city0 = City(100,200,0)

city0.display()

*Listing 12-6：编写创建一个城市的 setup()函数*

运行这个，你将看到你的第一个城市（见图 12-2）！

![image](img/f256-01.jpg)

*图 12-2：第一个城市*

可能有助于在城市上方显示其编号。为此，在城市的 display()方法中，在 noFill()之前添加以下代码：

textSize(20)

text(self.number,self.x-10,self.y-10)

我们使用 Processing 内置的`textSize()`函数来声明文本的大小。然后我们使用`text()`函数告诉程序要打印什么（城市的编号）以及打印的位置（在城市的左上方，距离城市 10 像素）。在创建城市时，让我们启动一个`cities`列表，并在屏幕上随机位置添加更多城市。为了使用`random`模块中的方法，我们需要在文件顶部导入`random`模块：

import random

现在我们可以像列表 12-7 中一样更新`setup()`函数。

*travelingSalesperson.pyde*

cities = []

def setup():

size(600,600)

background(0)

for i in range(6):

cities.append(City(random.randint(50,width-50),

random.randint(50,height-50),i))

for city in cities:

city.display()

*列表 12-7：编写`setup()`函数以创建六个随机城市*

在`setup()`函数中，我们添加了一个循环，循环六次。每次循环都会在屏幕上随机位置（距离边缘 50 个单位）添加一个 City 对象。下一个循环遍历`cities`列表中的所有元素并显示每个城市。运行这个代码，你将看到六个城市出现在随机位置，并标上它们的 ID 编号，如图 12-3 所示。

![image](img/f257-01.jpg)

*图 12-3：六个城市，标有它们的编号*

现在让我们思考城市之间的路线。我们将 City 对象（包含它们的位置和编号）放入`cities`列表，最终这些编号（我们的“遗传物质”）会按特定的顺序排列。因此，Route 对象也需要一个随机的数字列表：一个包含所有城市编号的随机顺序。当然，这些编号会在 0 到城市数量减 1 之间。我们不想在代码中每次想要改变城市数量时都去更改这些数字，因此我们会为城市数量创建一个变量。把这行代码放在文件的开头，City 类之前：

N_CITIES = 10

为什么`N_CITIES`是全大写字母？在整个代码中，我们不会改变城市的数量。因此它实际上不是一个变量，而是一个常量。在 Python 中，习惯上会将常量的名称大写，以与变量区分。这并不会改变 Python 对它们的处理方式；即使是大写名称的变量也依然可以修改。所以要小心。

我们将在代码中需要使用城市总数的地方使用`N_CITIES`，这样我们只需要更改一次值！把列表 12-8 中显示的代码放在 City 类之后。

class Route:

def __init__(self):

self.distance = 0

#按顺序将城市放入列表：

self.cityNums = random.sample(list(range(N_CITIES)), N_CITIES)

*列表 12-8：Route 类*

首先，我们将路线的距离（或长度，虽然“长度”是 Processing 中的一个关键字）设置为零，然后我们创建一个`cityNums`列表，将城市的编号按随机顺序排列，形成这条路线的城市顺序。

你可以使用 random 模块的 sample() 函数，给 Python 一个列表，然后通过告诉它要随机选择多少项来从列表中选择一些项。这类似于 choice()，但它不会选择同一项两次。在概率论中，这叫做“无替代抽样”。在 IDLE 中输入以下内容，看看抽样是如何工作的：

>>> n = list(range(10))

>>> n

[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

>>> import random

>>> x = random.sample(n,5)

>>> x

[2, 0, 5, 3, 8]

在这里，我们通过调用 range(10) 并将其转换（它是一个“生成器”）成列表，创建一个名为 n 的列表，包含 0 到 9 之间的数字。然后我们导入 random 模块，并要求 Python 使用 sample() 函数从列表 n 中随机挑选五个项目，并将它们保存到列表 x 中。在我们的 Route 代码 列表 12-8 中，由于变量 N_CITIES 表示城市数量为 10，我们正在使用 range(10) 随机选择 10 个数字（0 到 9），并将它们赋值给 Route 的 cityNums 属性。

那么这个如何显示呢？让我们在城市之间绘制紫色的线条。你可以使用任何你喜欢的颜色。

像这样在城市之间绘制线条，应该会让你想起代数或三角学课程中在图表点之间绘制线条。唯一的区别是现在在图形的末端，我们必须回到起点。记得第六章中使用过 beginShape、vertex 和 endShape 吗？就像我们用线条绘制形状一样，我们会把 Route 对象画成一个形状的轮廓，只不过这次我们不会填充它。使用 endShape(CLOSE) 将自动闭合循环！将 列表 12-9 中的代码添加到 Route 类中。

def display(self):

strokeWeight(3)

stroke(255,0,255) #紫色

beginShape()

for i in self.cityNums:

vertex(cities[i].x,cities[i].y)

#然后显示城市及其编号

cities[i].display()

endShape(CLOSE)

*列表 12-9：编写 Route 类的 display 方法*

这个循环使 Route 的 cityNums 列表中的每个城市成为多边形的一个顶点。路线是多边形的轮廓。注意在 Route 的 display() 方法中，我们调用了城市的 display() 方法。这样，我们就不需要手动命令每个城市单独显示了。

在 setup() 函数中，我们将使用城市列表和数字列表作为参数创建一个 Route 对象。然后我们会显示它。下面 列表 12-10 的最后两行代码就是这样做的。

def setup():

size(600,600)

background(0)

for i in range(N_CITIES):

cities.append(City(random.randint(50,width-50),

random.randint(50,height-50),i))

route1 = Route()

route1.display()

*列表 12-10：显示路线*

运行此代码，你将看到城市之间的路径，按随机顺序排列，如 图 12-4 所示。

![image](img/f259-01.jpg)

*图 12-4：随机的路线顺序*

要更改城市的数量，只需更改第一行中声明 N_CITIES 的数值，然后运行程序即可。图 12-5 显示了 N_CITIES = 7 时的输出结果。

![image](img/f260-01.jpg)

*图 12-5: 七个城市的路线*

现在你可以创建并显示路线了，让我们编写一个函数来衡量每条路线的距离。

#### 编写 CALCLENGTH() 方法

Route 对象有一个距离属性，在创建时被设置为零。每个 Route 对象还有一个城市列表，按顺序排列，称为 cityNums。我们只需要遍历 cityNums 列表，保持每对城市之间的距离总和。对于城市 0 到 4 没问题，但我们还需要计算从最后一个城市回到第一个城市的距离。

列表 12-11 显示了 calcLength() 方法的代码，该方法位于 Route 对象内部。

def calcLength(self):

self.distance = 0

for i,num in enumerate(self.cityNums):

# 计算当前城市到前一个城市的距离

self.distance += dist(cities[num].x,

cities[num].y,

cities[self.cityNums[i-1]].x,

cities[self.cityNums[i-1]].y)

return self.distance

*列表 12-11: 计算路线的长度*

首先，我们将 Route 的距离属性清零，这样每次调用该方法时，它都会从零开始。我们使用 enumerate() 函数，这样不仅可以获取 cityNums 列表中的城市编号，还能得到其索引。然后，我们将距离属性按从当前城市（num）到前一个城市（self.cityNums[i-1]）的距离递增。接下来，让我们将这行代码添加到 setup() 函数的末尾：

println(route1.calcLength())

我们现在可以在控制台中看到销售员行驶的总距离，如 图 12-6 所示。

![image](img/f261-01.jpg)

*图 12-6: 我们已经计算出距离...我想说*。

这真的是距离吗？让我们确认一下。

#### 测试 CALCLENGTH() 方法

让我们给程序一个简单的路线，这条路线是一个边长为 200 的正方形，接着检查距离。首先，我们将城市数量的常量更改为 4：

N_CITIES = 4

接下来，我们将 setup() 函数更改为 列表 12-12 中所示的内容。

cities = [City(100,100,0), City(300,100,1),

City(300,300,2), City(100,300,3)]

def setup():

size(600,600)

background(0)

'''for i in range(N_CITIES):

cities.append(City(random.randint(0,width),

random.randint(0,height),i))'''

route1 = Route()

route1.cityNums = [0,1,2,3]

route1.display()

println(route1.calcLength())

*列表 12-12: 手动创建一个路线以测试 calcLength() 方法*

我们注释掉了随机创建城市的循环，因为我们会在检查完 calcLength() 方法后再回来处理这个部分。我们创建了一个新的城市列表，包含了边长为 200 的正方形的顶点。我们还为 route1 声明了 cityNums 列表；否则，它会随机排列城市。我们预计这个 Route 的长度应该是 800。

当我们运行代码时，看到的内容如图 12-7 所示。

![image](img/f262-01.jpg)

*图 12-7：calcLength() 方法有效！*

这正是 800 个单位，正如预测的那样！你可以尝试一些矩形或其他容易验证的路线。

#### 随机路线

为了找到到达目的地的最短路径，我们需要找到所有可能的路线。为此，我们需要一个无限循环和 Processing 的内置 draw()函数。我们将把路线代码从 setup()移动到 draw()函数。我们还将创建一些随机路线，并显示它们及其长度。完整代码如下所示清单 12-13。

*travelingSales person.pyde*

import random

N_CITIES = 10

class City:

def __init__(self,x,y,num):

self.x = x

self.y = y

self.number = num #识别号码

def display(self):

fill(0,255,255) #天蓝色

ellipse(self.x,self.y,10,10)

textSize(20)

text(self.number,self.x-10,self.y-10)

noFill()

class Route:

def __init__(self):

self.distance = 0

#按照 numList 顺序将城市放入列表：

self.cityNums = random.sample(list(range(N_CITIES)),N_CITIES)

def display(self):

strokeWeight(3)

stroke(255,0,255) #紫色

beginShape()

for i in self.cityNums:

vertex(cities[i].x,cities[i].y)

#然后显示城市及其编号

cities[i].display()

endShape(CLOSE)

def calcLength(self):

self.distance = 0

for i,num in enumerate(self.cityNums):

#找到到达前一个城市的距离

self.distance += dist(cities[num].x,

cities[num].y,

cities[self.cityNums[i-1]].x,

cities[self.cityNums[i-1]].y)

return self.distance

cities = []

def setup():

size(600,600)

for i in range(N_CITIES):

cities.append(City(random.randint(50,width-50),

random.randint(50,height-50),i))

def draw():

background(0)

route1 = Route()

route1.display()

println(route1.calcLength())

*清单 12-13：创建并显示随机路线*

当你运行这个时，你应该会看到一堆路线被显示出来，并且一些数字被打印到控制台。

但我们实际上只关心保持最好的（最短的）路线，因此我们将添加一些代码来保存“bestRoute”并检查新的随机路线。将 setup()和 draw()修改为清单 12-14 所示。

cities = []

random_improvements = 0

mutated_improvements = 0

def setup():

global best, record_distance

size(600,600)

for i in range(N_CITIES):

cities.append(City(random.randint(50,width-50),

random.randint(50,height-50),i))

best = Route()

record_distance = best.calcLength()

def draw():

global best, record_distance, random_improvements

background(0)

best.display()

println(record_distance)

println("random: "+str(random_improvements))

route1 = Route()

length1 = route1.calcLength()

if length1 < record_distance:

record_distance = length1

best = route1

random_improvements += 1

*清单 12-14：跟踪随机改进*

在 `setup()` 函数之前，我们创建了一个变量，用来计算程序进行的随机改进次数。同时，我们还创建了一个变量，稍后将用于计算变异后的改进次数。

在 `setup()` 函数中，我们创建了 route1，作为第一个路线，并称其为“最佳路线”，并将其距离命名为 `record_distance`。由于我们希望在其他函数中共享这些变量，所以在函数开始时我们将它们声明为全局变量。

在 `draw()` 函数中，我们不断生成新的随机路线，并检查它们是否比我们目前认为的最佳路线更好。由于我们只使用了 10 个城市，如果我们让程序运行一段时间，可能会找到最优解。你会看到，它只需要大约十几个随机改进。但请记住，10 个城市的唯一路线只有 181,440 条。一个 10 城市的路线在图 12-8 中展示。

然而，如果你将城市数量更改为 20，你的程序将会不断运行，如果让它继续运行几天，可能也无法接近最优解。我们需要开始使用本章开头提到的短语猜测程序中的思想——对我们的猜测进行评分，并变异出最好的猜测。与之前不同的是，我们将创建一个“配对池”，将最佳路线的数字列表结合在一起，就像它们是基因一样。

![image](img/f265-01.jpg)

*图 12-8：随机找到一个最优路线——如果你能等几分钟*

#### 应用短语猜测变异思想

数字列表（销售员将按顺序访问的城市）将成为路线的遗传物质。首先，我们看看一些随机变异的路线如何解决旅行商问题（就像我们的短语猜测程序一样），然后我们对更好的路线进行变异和“配对”，以便（希望）创建一个更优的路线。

#### 变异列表中的两个数字

我们来编写一个方法，随机变异 `Route` 对象的 `cityNums` 列表中的两个数字。其实这只是一个交换。你大概可以猜到我们将如何随机选择两个数字，并让列表中具有这些索引的城市号码互换位置。

Python 有一种独特的表示法来交换两个数字的值。你可以在不创建临时变量的情况下交换两个数字。例如，如果你在 IDLE 中输入 Listing 12-15 中的代码，它将无法运行。

>>> x = 2

>>> y = 3

>>> x = y

>>> y = x

>>> x

3

>>> y

3

*Listing 12-15：交换变量值的错误方式*

当你通过输入 `x = y` 将 x 的值更改为与 y 相同的时候，它们都会变成 3。现在，当你尝试将 y 设置为与 x 相同的值时，它不会设置为原始的 x 值（2），而是设置为当前的 x 值，即 3。所以，两个变量最终都变成了 3。

但你*可以*在同一行中交换值，像这样：

>>> x = 2

>>> y = 3

>>> x,y = y,x

>>> x

3

>>> y

2

交换两个变量的值这种方法对于我们即将进行的变异非常有用。我们不仅限于交换两个数字，而是可以变异更多的城市。我们可以将交换放入一个循环中，这样程序就能选择任意数量的城市并交换前两个数字，然后交换下一个数字对，依此类推。mutateN()方法的代码展示了这种做法。

def mutateN(self,num):

indices = random.sample(list(range(N_CITIES)),num)

child = Route()

child.cityNums = self.cityNums[::]

for i in range(num-1):

child.cityNums[indices[i]],child.cityNums[indices[(i+1)%num]] = \

child.cityNums[indices[(i+1)%num]],child.cityNums[indices[i]]

return child

*清单 12-16：编写 mutateN()方法，用于变异任意数量的城市*

我们给 mutateN()方法传递 num，表示需要交换的城市数量。然后，方法通过从城市编号范围中随机采样来创建一个要交换的索引列表。它创建一个“子”路线，并将自己的城市编号列表复制到子路线中。然后它交换 num-1 次。如果交换了所有 num 次，第一个被交换的城市会与其他所有城市交换，最终回到原来的位置。

那一长行代码实际上就是我们之前看到的 a,b = b,a 语法，只不过这里交换的是两个 cityNums。mod（%）操作符确保索引不会超过 num，也就是样本中的城市数量。所以，如果你交换的是四个城市，例如，当 i 为 4 时，它会将 i + 1 从 5 变为 5 % 4，结果是 1。

接下来，我们在 draw()函数的末尾添加一部分代码，用来变异最佳路线的数字列表并测试变异后路线的长度，如清单 12-17 所示。

def draw():

global best,record_distance,random_improvements

global mutated_improvements

background(0)

best.display()

println(record_distance)

println("random: "+str(random_improvements))

println("mutated: "+str(mutated_improvements))

route1 = Route()

length1 = route1.calcLength()

if length1 < record_distance:

record_distance = length1

best = route1

random_improvements += 1

for i in range(2,6):

#创建一条新路线

mutated = Route()

#将其数字列表设置为最佳

mutated.cityNums = best.cityNums[::]

mutated = mutated.mutateN(i) #进行变异

length2 = mutated.calcLength()

if length2 < record_distance:

record_distance = length2

best = mutated

mutated_improvements += 1

*清单 12-17：变异最佳“生物体”*

在 for i in range(2,6):循环中，我们告诉程序变异数字列表中的 2、3、4 和 5 个数字并检查结果。现在，程序通常在几秒钟内就能在 20 个城市的路线中表现得很好，如图 12-9 所示。

![image](img/f267-01.jpg)

*图 12-9：20 个城市的路线*

变异后的“生物体”比随机生成的改进效果好得多！图 12-10 展示了输出结果。

![image](img/f268-01.jpg)

*图 12-10：变异比随机改进表现得好得多！*

图 12-10 分类了所有的改进，其中 29 个是由于突变，只有一个是由于随机生成的路线。这表明，突变列表比创建新的随机路线更能找到最优路径。我通过更改这一行加大了突变的力度，使得从 2 到 10 个城市的交换成为可能：

for i in range(2,11):

尽管它改善了 20 城市问题的表现，甚至对一些 30 城市问题也有帮助，但程序经常陷入非最优路径，就像图 12-11 所示。

![image](img/f268-02.jpg)

*图 12-11：一个困在非最优路径的 30 城市问题*

我们将进行最后一步，全面使用基因算法。现在我们不再局限于我们认为目前最好的路线，而是从庞大的人口中选择。

我们将为任意数量的路线创建一个人口列表，选取最“适应”的路线，交叉它们的号码列表，并希望能创造出更好的路线！就在 setup()函数之前，在城市列表之后，添加人口列表和路线数量常量，如清单 12-18 所示。

cities = []

random_improvements = 0

mutated_improvements = 0

population = []

POP_N = 1000 #路线数量

*清单 12-18：开始一个人口列表并为人口大小创建一个变量*

我们刚刚创建了一个空列表来存放我们的路线种群，并为路线总数创建了一个变量。在 setup()函数中，我们使用 POP_N 条路线填充人口列表，如清单 12-19 所示。

def setup():

global best,record_distance,first,population

size(600,600)

for i in range(N_CITIES):

cities.append(City(random.randint(50,width-50),

random.randint(50,height-50),i))

#将个体放入人口列表

for i in range(POP_N):

population.append(Route())

best = random.choice(population)

record_distance = best.calcLength()

first = record_distance

*清单 12-19：创建一个路线种群*

请注意，我们必须声明人口列表为全局变量。我们通过使用 for i in range(POP_N)将 POP_N 条路线放入人口列表，然后我们随机选择了一条路线作为目前最好的路线。

#### 通过交叉改进路线

在 draw()函数中，我们将对人口列表进行排序，将路线对象中长度最短的放在最前面。我们将创建一个名为 crossover()的方法，随机地将 cityNums 列表拼接在一起。它将执行以下操作：

a: [6, 0, 7, 8, 2, 1, 3, 9, 4, 5]

b: [1, 0, 4, 9, 6, 2, 5, 8, 7, 3]

index: 3

c: [6, 0, 7, 1, 4, 9, 2, 5, 8, 3]

“父母”是列表 a 和 b。索引随机选择：索引为 3。然后在索引 2（7）和索引 3（8）之间切割列表，因此子列表从 [6,0,7] 开始。剩下的不在切片中的数字按照它们在列表 b 中出现的顺序添加到子列表中：[1,4,9,2,5,8,3]。我们将这两个列表连接起来，这就是子列表。crossover() 方法的代码见 Listing 12-20。

def crossover(self, partner):

'''与合作伙伴的基因拼接在一起'''

child = Route()

#随机选择切片点

index = random.randint(1, N_CITIES - 2)

#添加切片点之前的数字

child.cityNums = self.cityNums[:index]

#有一半的时间反转它们

如果 random.random() < 0.5:

child.cityNums = child.cityNums[::-1]

#列表中的数字不在切片中

notinslice = [x for x in partner.cityNums if x not in child.cityNums]

#添加不在切片中的数字

child.cityNums += notinslice

返回 child

*Listing 12-20：编写 Route 类的 crossover() 方法*

crossover() 方法要求我们指定合作伙伴，即另一个父母。创建子路线，并随机选择一个切割点。子列表首先获取切片中的数字，然后有一半的时间我们会反转这些数字，以增加基因多样性。我们创建一个包含不在切片中的数字的列表，并按照它们在另一个父母（或合作伙伴）列表中出现的顺序添加它们。最后，将这些切片连接起来并返回子路线。

在 draw() 函数中，我们需要检查种群列表中的路线，找到最短的那一条。我们还需要像之前那样检查每一条吗？幸运的是，Python 提供了一个方便的 sort() 函数，可以用来按 calcLength() 对种群列表进行排序。所以列表中的第一个 Route 将是最短的。draw() 函数的最终代码见 Listing 12-21。

def draw():

global best, record_distance, population

background(0)

best.display()

println(record_distance)

#println(best.cityNums)  #如果你需要经过所有城市的准确路线！

➊ population.sort(key=Route.calcLength)

population = population[:POP_N]  #限制种群大小

length1 = population[0].calcLength()

如果 length1 < record_distance:

record_distance = length1

best = population[0]

#对种群进行交叉操作

➋ 对于 i 在 POP_N 范围内：

parentA, parentB = random.sample(population, 2)

#繁殖：

child = parentA.crossover(parentB)

population.append(child)

#对种群中的最佳个体进行 mutateN

➌ 对于 i 在 3 到 25 之间：

如果 i < N_CITIES:

new = best.mutateN(i)

population.append(new)

#在种群中随机变异一些路线

➍ 对于 i 在 3 到 25 之间：

如果 i < N_CITIES:

new = random.choice(population)

new = new.mutateN(i)

population.append(new)

*Listing 12-21：编写最终的 draw() 函数*

我们在➊使用 sort()函数，然后修剪种群列表的末尾（最长的路线），确保列表保持为 POP_N 条路线的长度。接着我们检查种群列表中的第一个项目，看看它是否比最佳路线短。如果是，我们像之前一样将其设为最佳。接下来，我们从种群中随机选取两条路线，对它们的 cityNums 列表进行交叉操作，并将得到的子路线添加到种群中 ➋。在➌，我们突变最佳路线，交换 3、4 和 5 这几个数字，一直到 24 个数字（如果这个数字少于草图中的城市数）。最后，我们从种群中随机选择路线并进行突变，尝试改善距离 ➍。

现在，使用一个包含 10,000 条路线的种群，我们的程序能够对 100 个城市之间的最佳路线做出相当不错的近似。图 12-12 展示了程序将一条初始长度为 26,000 单位的路线改善到不足 4,000 单位的情况。

![image](img/f271-01.jpg)

*图 12-12：通过 100 个城市的路线改进*

这个过程“仅仅”花了半个小时就完成了！

### 总结

在本章中，我们不仅仅使用 Python 来回答数学课上那些答案已经知道的问题。相反，我们使用间接方法（评估一串字符或通过一堆城市的路线）来解决那些没有答案的课题！

为此，我们模仿了基因突变的生物行为，利用一些突变比其他突变更有利于解决当前问题的事实。我们在章节开始时就知道了目标路线，但为了确定最终路线是否最优，我们必须保存城市位置并多次运行程序。这是因为遗传算法，像真实的生物一样，只能利用它们起初拥有的东西，并且它们常常陷入非最优的死胡同，正如你所看到的那样。

但这些间接方法出奇地有效，并广泛应用于机器学习和工业过程。方程式适用于表达非常简单的关系，但许多情况并不那么简单。现在，你有了很多有用的工具，如我们的“羊和草”模型、分形、细胞自动机，最后还有遗传算法，可以用来研究和建模非常复杂的系统。
