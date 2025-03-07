## 2

做出决策

![image](img/common.jpg)

我们日常使用的大多数程序会根据执行过程中的不同情况做出不同的反应。例如，当文字处理软件询问我们是否保存工作时，它会根据我们的回答做出决策：如果回答“是”，则保存工作；如果回答“否”，则不保存工作。在本章中，我们将学习 if 语句，它能让我们的程序做出决策。

我们将解决两个问题：确定篮球比赛的结果，以及判断一个电话号码是否属于推销员。

### 问题 #3：获胜队伍

在这个问题中，我们需要输出一条依赖于篮球比赛结果的消息。为此，我们将学习所有关于 if 语句的知识。我们还将学习如何在程序中存储和处理真假值。

这是 DMOJ 问题 ccc19j1。

#### 挑战

在篮球比赛中，有三种方式得分：三分投篮、两分投篮和单分罚球。

你刚观看了一场苹果队与香蕉队的篮球比赛，并记录了每个队伍成功的三分、两分和单分罚球次数。请判断是苹果队获胜、香蕉队获胜，还是比赛平局。

#### 输入

输入有六行。前面三行给出了苹果队的得分，后面三行给出了香蕉队的得分。

+   第一行给出苹果队成功的三分投篮次数。

+   第二行给出苹果队成功的两分投篮次数。

+   第三行给出苹果队成功的单分罚球次数。

+   第四行给出香蕉队成功的三分投篮次数。

+   第五行给出香蕉队成功的两分投篮次数。

+   第六行给出香蕉队成功的单分罚球次数。

每个数字是 0 到 100 之间的整数。

#### 输出

输出是一个单一的字符。

+   如果苹果队得分超过香蕉队，输出 A（*A*表示苹果队）。

+   如果香蕉队得分超过苹果队，输出 B（*B*表示香蕉队）。

+   如果苹果队和香蕉队得分相同，输出 T（*T*表示平局）。

### 条件执行

我们可以通过在第一章中学到的内容来取得很大进展。我们可以使用 input 和 int 来读取输入的六个整数。我们可以使用变量来保存这些值。我们可以将成功的三分投篮次数乘以 3，将成功的两分投篮次数乘以 2。我们可以使用 print 输出 A、B 或 T。

我们还没有学到的是，如何让我们的程序根据比赛结果做出决策。我可以通过两个测试用例来演示为什么我们需要这个。

首先，考虑这个测试用例：

5

1

3

1

1

1

苹果队得了 5 * 3 + 1 * 2 + 3 = 20 分，香蕉队得了 1 * 3 + 1 * 2 + 1 = 6 分。苹果队赢得了比赛，因此这是正确的输出：

A

其次，考虑这个测试案例，其中苹果队和香蕉队的分数被交换了：

1

1

1

5

1

3

这一次，香蕉队赢得了比赛，所以这是正确的输出：

B

我们的程序必须能够比较苹果队和香蕉队的总得分，并根据这个比较结果选择输出 A、B 或 T。

我们可以使用 Python 的 if 语句来做出这些决策。*条件*是一个真假值的表达式，if 语句使用条件来决定该做什么。if 语句导致 *条件执行*，其名称来源于程序的执行受到条件的影响。

我们首先要学习一个新的类型，它让我们能够表示真假值，以及如何构建这种类型的表达式。然后，我们将使用这些表达式来编写 if 语句。

### 布尔类型

将表达式传递给 Python 的 type 函数，它会告诉你该表达式值的类型：

>>> type(14)

<class 'int'>

>>> type(9.5)

<class 'float'>

>>> type('hello')

<class 'str'>

>>> type(12 + 15)

<class 'int'>

我们还没有遇到过的 Python 类型是布尔（bool）类型。与整数、字符串和浮点数不同，布尔类型只有两个可能的值：True 和 False。这正是我们用来表示条件结果的值。

>>> True

True

>>> False

False

>>> type(True)

<class 'bool'>

>>> type(False)

<class 'bool'>

我们能用这些值做什么呢？对于数字，我们有加法和减法等数学运算符，它们允许我们将值组合成更复杂的表达式。我们将需要一组新的操作符来处理布尔值。

### 关系操作符

5 大于 2 吗？4 小于 1 吗？我们可以使用 Python 的 *关系操作符* 来进行这样的比较。它们返回 True 或 False，因此用于编写 *布尔表达式*。

> 操作符接受两个操作数，如果第一个操作数大于第二个操作数，则返回 True，否则返回 False：

>>> 5 > 2

True

>>> 9 > 10

False

同样，我们有 < 操作符用于小于：

>>> 4 < 1

False

>>> -2 < 0

True

有 >= 用于大于或等于，<= 用于小于或等于：

>>> 4 >= 2

True

>>> 4 >= 4

True

>>> 4 >= 5

False

>>> 8 <= 6

False

为了判断相等性，我们使用 == 操作符。那是两个等号，而不是一个。记住，一个等号（=）用于赋值语句；它与检查相等性无关。

>>> 5 == 5

True

>>> 15 == 10

False

对于不等式，我们使用 != 操作符。如果操作数不相等，则返回 True，如果相等，则返回 False：

>>> 5 != 5

False

>>> 15 != 10

True

真实的程序不会评估那些我们已经知道其值的表达式。我们不需要 Python 告诉我们 15 不等于 10。更常见的是，我们会在这些表达式中使用变量。例如，number != 10 是一个依赖于 number 所指向的值的表达式。

关系操作符也可以作用于字符串。在检查相等性时，大小写是重要的：

>>> 'hello' == 'hello'

真

>>> 'Hello' == 'hello'

False

如果一个字符串按字母顺序排在另一个字符串之前，则前者小于后者：

>>> 'brave' < 'cave'

真

>>> 'cave' < 'cavern'

真

>>> 'orange' < 'apple'

False

但是，当同时涉及小写字母和大写字母时，结果可能会让人惊讶：

>>> 'apple' < 'Banana'

False

奇怪，对吧？这与字符在计算机内部的存储方式有关。通常，大写字母按字母顺序排在小写字母前面。看看这个：

>>> '10' < '4'

真

如果这些是数字，结果会是 False。但字符串是按字符从左到右逐一比较的。Python 比较 '1' 和 '4'，因为 '1' 比 '4' 小，所以 < 操作符返回 True。确保你的值类型符合你的预期！

一个对字符串有效但对数字无效的关系操作符是 in。它会返回 True，如果第一个字符串至少在第二个字符串中出现一次，否则返回 False：

>>> 'ppl' in 'apple'

真

>>> 'ale' in 'apple'

False

**概念检查**

以下代码的输出是什么？

a = 3

b = (a != 3)

print(b)

A. 真

B. 假

C. 3

D. 这段代码会产生语法错误

答案：B. 表达式 a != 3 结果为 False；b 被赋值为这个 False 值。

### if 语句

我们现在将探索 Python 中 if 语句的几种变体。

#### 单独的 if

假设我们的最终得分分别存储在两个变量中，apple_total 和 banana_total，并且我们希望当 apple_total 大于 banana_total 时输出 A。我们可以这样做：

>>> apple_total = 20

>>> banana_total = 6

>>> 如果 apple_total > banana_total:

...     print('A')

...

A

Python 输出 A，正如我们所预期的。

if 语句以关键字 if 开头。*关键字*是 Python 中有特殊意义的单词，不能用作变量名。关键字 if 后面跟着一个布尔表达式，接着是冒号，再后面是一个或多个缩进的语句。这些缩进的语句通常称为 if 语句的 *块*。如果布尔表达式为 True，则执行该块；如果布尔表达式为 False，则跳过该块。

请注意，提示符从 >>> 变成了 .... 这是一个提醒，表示我们在 if 语句的块内，必须对代码进行缩进。我选择使用四个空格进行缩进，因此要缩进代码，请按四次空格键。某些 Python 程序员按 TAB 键进行缩进，但我们在本书中将只使用空格。

一旦你输入 print('A') 并按下 ENTER，应该会看到另一个 ... 提示符。由于我们没有其他内容要放在这个 if 语句中，再按一次 ENTER 以关闭这个提示符并返回 >>> 提示符。这个额外的 ENTER 是 Python shell 的一个特点；当我们在文件中编写 Python 程序时，这种空白行是不用的。

让我们看一个例子，演示如何将两个语句放在 if 语句的块中：

>>> apple_total = 20

>>> banana_total = 6

>>> 如果 apple_total > banana_total:

...     print('A')

...     print('Apples win!')

...

A

苹果胜利！

两个 print 调用都会执行，输出两行。

让我们试试另一个 if 语句，这个语句包含一个值为 False 的布尔表达式：

>>> apple_total = 6

>>> banana_total = 20

>>> if apple_total > banana_total:

...     print('A')

...

这次没有调用 print 函数：因为 apple_total > banana_total 为 False，所以 if 语句的代码块被跳过。

#### 使用 elif

让我们使用三个连续的 if 语句，如果苹果胜利就打印 A，如果香蕉胜利就打印 B，如果平局就打印 T：

>>> apple_total = 6

>>> banana_total = 6

>>> if apple_total > banana_total:

...     print('A')

...

>>> if banana_total > apple_total:

...     print('B')

...

>>> if apple_total == banana_total:

...     print('T')

...

T

前两个 if 语句的代码块被跳过，因为它们的布尔表达式为 False。但第三个 if 语句的代码块会执行，输出 T。

当你把一个 if 语句放在另一个后面时，它们是独立的。每个布尔表达式都会被评估，无论前面的布尔表达式是否为 True 或 False。

对于给定的 apple_total 和 banana_total 值，只有一个 if 语句会被执行。例如，如果 apple_total > banana_total 为 True，那么第一个 if 语句会被执行，其他两个将不会。我们可以通过编写代码来突出显示只有一个代码块被允许执行。下面是实现方法：

❶ >>> if apple_total > banana_total:

...     print('A')

❷ ... elif banana_total > apple_total:

...     print('B')

... elif apple_total == banana_total:

...     print('T')

...

T

现在这是一个单独的 if 语句，而不是三个独立的 if 语句。因此，不要在 ... 提示符下按 ENTER；而是直接输入 elif 行。

为了执行这个 if 语句，Python 会首先评估第一个布尔表达式 ❶。如果它为 True，则输出 A，其他的 elif 会被跳过。如果它为 False，Python 会继续评估第二个布尔表达式 ❷。如果它为 True，则输出 B，剩下的 elif 会被跳过。如果它为 False，Python 会继续评估第三个布尔表达式 ❸。如果它为 True，则输出 T。

关键字 elif 代表“else-if”（否则如果）。它提醒我们，只有在前面的 if 语句中的“else”部分没有被执行时，才会检查 elif 表达式。

这版本的代码等同于我们之前使用三个独立 if 语句的代码。如果我们希望允许执行多个代码块，我们应该使用三个独立的 if 语句，而不是一个包含 elif 块的单一 if 语句。

#### 使用 if 和 else

我们可以使用 else 关键字在所有 if 语句的布尔表达式为 False 时执行代码。下面是一个示例：

>>> if apple_total > banana_total:

...     print('A')

... elif banana_total > apple_total:

...     print('B')

... else:

...     print('T')

...

T

Python 从上到下评估布尔表达式。如果其中任何一个为 True，Python 将执行相关的代码块并跳过剩下的 if 语句。如果所有布尔表达式都为 False，Python 将执行 else 代码块。

请注意，不再测试 apple_total == banana_total。只有在 apple_total > banana_total 为 False 且 banana_total > apple_total 为 False 时，也就是当值相等时，才会执行 else 部分。

你应该使用独立的 if 语句吗？使用带有 elif 的 if 语句？还是使用带有 else 的 if 语句？这通常取决于个人偏好。如果你希望最多只有一个代码块被执行，可以使用一系列的 elif 语句。else 语句可以帮助代码更清晰，并且避免需要写一个捕捉所有情况的布尔表达式。比起 if 语句的精确格式，更重要的是编写正确的逻辑！

**概念检查**

以下代码执行后，x 的值是多少？

x = 5

if x > 2:

x = -3

if x > 1:

x = 1

else:

x = 3

A. -3

B. 1

C. 2

D. 3

E. 5

答案：D。因为 x > 2 为 True，第一个 if 语句的代码块被执行。赋值语句 x = -3 使 x 的值变为 -3。接下来是第二个 if 语句，这里 x > 1 为 False，因此执行 else 代码块，x = 3 使得 x 的值变为 3。我建议将 if x > 1 改为 elif x > 1，看看程序的行为如何变化！

**概念检查**

以下两段代码完全做相同的事情吗？假设 temperature 已经是一个数字。

片段 1:

if temperature > 0:

print('warm')

elif temperature == 0:

print('zero')

else:

print('cold')

片段 2:

if temperature > 0:

print('warm')

elif temperature == 0:

print('zero')

print('cold')

A. 是的

B. 否

答案：B。片段 2 *总是*将 cold 打印为最后一行输出，因为 print('cold') 没有缩进！它不属于任何 if 语句。

### 解题过程

现在是时候解决“胜利团队”问题了。在本书中，我通常会先展示完整的代码，然后进行讨论。但由于我们的解决方案比第一章中的那些长，我决定在此先将代码分为三部分展示，再将其作为整体展示。

首先，我们需要读取输入。这需要六次调用 input，因为我们有两个团队，每个团队有三项信息。我们还需要将每一项输入转换为整数。以下是代码：

apple_three = int(input())

apple_two = int(input())

apple_one = int(input())

banana_three = int(input())

banana_two = int(input())

banana_one = int(input())

第二，我们需要确定 Apples 和 Bananas 得分的总数。对于每个团队，我们将三分球、两分球和一分球的得分加起来。我们可以这样做：

apple_total = apple_three * 3 + apple_two * 2 + apple_one

banana_total = banana_three * 3 + banana_two * 2 + banana_one

第三步，我们生成输出。如果苹果队获胜，我们输出 A；如果香蕉队获胜，我们输出 B；否则，我们知道比赛是平局，因此输出 T。我们使用 if 语句来实现这一点，如下所示：

if apple_total > banana_total:

print('A')

elif banana_total > apple_total:

print('B')

else:

print('T')

这就是我们需要的所有代码。查看列表 2-1 以获取完整的解决方案。

apple_three = int(input())

apple_two = int(input())

apple_one = int(input())

banana_three = int(input())

banana_two = int(input())

banana_one = int(input())

apple_total = apple_three * 3 + apple_two * 2 + apple_one

banana_total = banana_three * 3 + banana_two * 2 + banana_one

if apple_total > banana_total:

print('A')

elif banana_total > apple_total:

print('B')

else:

print('T')

*列表 2-1：解决获胜团队*

如果你将我们的代码提交给评测系统，你应该看到所有测试用例通过。

**概念检查**

以下版本的代码是否正确解决了问题？

apple_three = int(input())

apple_two = int(input())

apple_one = int(input())

banana_three = int(input())

banana_two = int(input())

banana_one = int(input())

apple_total = apple_three * 3 + apple_two * 2 + apple_one

banana_total = banana_three * 3 + banana_two * 2 + banana_one

if apple_total < banana_total:

print('B')

elif apple_total > banana_total:

print('A')

else:

print('T')

A. 是

B. 否

答案：A。运算符和代码顺序不同，但代码仍然正确。如果苹果队失败，我们输出 B（因为香蕉队获胜）；如果苹果队获胜，我们输出 A；否则，我们知道比赛是平局，因此输出 T。

在继续之前，您可能想尝试解决《章节练习》中的第 1 题，详见第 45 页。

### 问题 #4：电话推销员

有时我们需要编码比目前所见更复杂的布尔表达式。在这个问题中，我们将学习帮助我们完成这项工作的布尔运算符。

这是 DMOJ 问题 ccc18j1。

#### 挑战

在这个问题中，我们假设电话号码是四位数字。如果电话号码的四个数字满足以下三个条件中的所有条件，则属于电话推销员：

+   第一位数字是 8 或 9。

+   第四位数字是 8 或 9。

+   第二位和第三位数字相同。

举个例子，电话号码的四个数字是 8119，属于电话推销员。

确定电话号码是否属于电话推销员，并指示我们是否应该接电话或忽略它。

#### 输入

输入有四行。这些行分别给出电话号码的第一、第二、第三和第四位数字。每位数字都是 0 到 9 之间的整数。

#### 输出

如果电话号码属于电话推销员，输出 ignore；否则，输出 answer。

### 布尔运算符

关于属于推销员的电话号码，必须满足什么条件？它的首位数字必须是 8 *或* 9。*并且*，它的第四位数字必须是 8 *或* 9。*并且*，第二位和第三位数字必须相同。我们可以使用 Python 的 *布尔操作符* 来编码这种 “or” 和 “and” 逻辑。

#### or 操作符

or 操作符将两个布尔表达式作为操作数。如果至少有一个操作数为 True，则返回 True，否则返回 False：

>>> True or True

True

>>> True or False

True

>>> False or True

True

>>> False or False

False

要使 or 操作符返回 False，唯一的方法是两个操作数都为 False。

我们可以使用 or 来判断一个数字是否为 8 或 9：

>>> digit = 8

>>> digit == 8 or digit == 9

True

>>> digit = 3

>>> digit == 8 or digit == 9

False

请记得在 第一章 的“整数和浮点数”中提到，Python 使用操作符优先级来决定操作符的应用顺序。or 的优先级低于关系操作符的优先级，这意味着我们通常不需要在操作数周围加括号。例如，在 digit == 8 or digit == 9 中，or 的两个操作数是 digit == 8 和 digit == 9。这与我们写成 (digit == 8) or (digit == 9) 是一样的。

在英语中，如果有人说“如果数字是 8 或 9”，这很合理。但在 Python 中直接这么写是不行的：

>>> digit = 3

>>> if digit == 8 or 9:

...     print('yes!')

...

yes!

请注意，我错误地将第二个操作数写成了 9，而不是 digit == 9。Python 输出了 yes!，这显然不是我们想要的结果，因为 digit 的值是 3。原因是 Python 将非零数字视为 True。由于 9 被视为 True，这使得整个 or 表达式为 True。在将自然语言转为 Python 时，务必仔细检查布尔表达式，避免这类错误。

#### and 操作符

and 操作符在两个操作数都为 True 时返回 True，否则返回 False：

>>> True and True

True

>>> True and False

False

>>> False and True

False

>>> False and False

False

要使 And 操作符返回 True，唯一的方法是两个操作数都为 True。

and 的优先级高于 or。下面是一个说明这点的例子：

>>> True or True and False

True

Python 这样解析该表达式，首先执行 and 操作：

>>> True or (True and False)

True

结果为 True，因为 or 的第一个操作数为 True。

我们可以通过加括号强制先执行 or 操作：

>>> (True or True) and False

False

结果为 False，因为 and 的第二个操作数为 False。

#### not 操作符

另一个重要的布尔操作符是 not。与 or 和 and 不同，not 只有一个操作数。如果该操作数为 True，则 not 返回 False，反之亦然：

>>> not True

False

>>> not False

True

not 的优先级高于 or 和 and。

**概念检查**

下面是一个表达式，以及该表达式的带括号版本。哪一个会返回 True？

A. not True and False

B. (not True) and False

C. not (True and False)

D. 以上都不是

答案：C. 表达式 (True and False) 结果为 False；因此 not 会使整个表达式为 True。

**概念检查**

考虑表达式 not a or b。

以下哪个使得该表达式为 False？

A. a 为 False, b 为 False

B. a 为 False, b 为 True

C. a 为 True, b 为 False

D. a 为 True, b 为 True

E. 上述多个条件

答案：C. 如果 a 为 True，则 not a 为 False。由于 b 也是 False，两个 or 运算符的操作数都为 False，因此整个表达式的结果为 False。

### 解决问题

在准备好布尔运算符后，我们可以解决推销员问题。我们的解决方案见清单 2-2。

num1 = int(input())

num2 = int(input())

num3 = int(input())

num4 = int(input())

❶ if ((num1 == 8 or num1 == 9) and

(num4 == 8 or num4 == 9) and

(num2 == num3)):

print('ignore')

else:

print('answer')

*清单 2-2：解决推销员问题*

与《胜利团队》一样，我们首先读取输入并将其转换为整数。

我们 if 语句的高级结构 ❶ 是三个通过 and 运算符连接的表达式；每个表达式必须为 True，整个表达式才为 True。我们要求第一个数字为 8 或 9，第四个数字为 8 或 9，第二和第三个数字相等。如果这三种条件都成立，那么我们知道该电话号码属于推销员，我们输出 ignore。否则，该电话号码不属于推销员，我们输出 answer。

我将布尔表达式分成了三行。这需要将整个表达式包裹在一个额外的括号中，正如我所做的那样。（如果没有这些括号，您将会得到语法错误，因为 Python 无法判断该表达式将在下一行继续。）

Python 风格指南建议一行代码不超过 79 个字符。一行完整的布尔表达式会达到 76 个字符，但我认为三行版本更清晰，突出了每个条件必须为 True 的要求。

我们这里有一个很好的解决方案。为了进一步探索，我们来讨论一些替代方法。

我们的代码使用布尔表达式来检测电话号码是否属于推销员。我们也可以选择编写代码来检测电话号码是否*不*属于推销员。如果电话号码不属于推销员，我们应该输出 answer；否则，我们应该输出 ignore。

如果第一个数字既不是 8 也不是 9，那么该电话号码不属于推销员。或者，如果第四个数字既不是 8 也不是 9，那么该电话号码不属于推销员。或者，如果第二和第三个数字不相等，那么该电话号码不属于推销员。如果这些表达式中有任何一个为 True，那么该电话号码不属于推销员。

请参阅清单 2-3，其中包含了捕捉此逻辑的代码版本。

num1 = int(input())

num2 = int(input())

num3 = int(input())

num4 = int(input())

如果 ((num1 != 8 并且 num1 != 9) 或

(num4 != 8 和 num4 != 9) 或

(num2 != num3)):

print('answer')

否则：

print('ignore')

*列表 2-3：解决电话推销员问题，替代方法*

正确处理所有这些 !=、or 和 and 运算符可不容易！例如，注意我们不得不将所有 == 运算符改为 !=，所有 or 运算符改为 and，所有 and 运算符改为 or。

另一种方法是使用 not 运算符来一次性否定“是电话推销员”的表达式。请参阅 列表 2-4 查看该代码。

num1 = int(input())

num2 = int(input())

num3 = int(input())

num4 = int(input())

如果不是 ((num1 == 8 或 num1 == 9) 并且

(num4 == 8 或 num4 == 9) 并且

(num2 == num3)):

print('answer')

否则：

print('ignore')

*列表 2-4：解决电话推销员问题，not 运算符*

你觉得这些解决方案哪种最直观？if 语句的逻辑结构通常有多种方式，我们应该选择最容易正确的方式。对我而言，列表 2-2 是最自然的，但你可能有不同的看法！

选择你最喜欢的版本并提交给评审员。你应该会看到所有测试用例都通过了。

### 注释

我们应该始终努力使我们的程序尽可能清晰。这有助于避免编程时引入错误，并且在错误出现时更容易修复代码。具有意义的变量名、运算符周围的空格、用空行将程序分段、简洁的 if 语句逻辑：这些做法都能提高我们编写代码的质量。另一个好习惯是给代码添加*注释*。

注释由 # 字符引入，并持续到行末。Python 会忽略注释，因此它们不会对程序的执行产生任何影响。我们添加注释是为了提醒自己或他人我们所做的设计决策。假设读代码的人已经了解 Python，因此避免添加仅仅重复代码功能的注释。以下是一个带有不必要注释的代码：

>>> x = 5

>>> x = x + 1  # 将 x 增加 1

该注释没有添加任何超出我们已知的关于赋值语句的信息。

请参阅 列表 2-5 查看带有注释的 列表 2-2 的版本。

❶ # ccc18j1，电话推销员

num1 = int(input())

num2 = int(input())

num3 = int(input())

num4 = int(input())

❷ # 电话推销员号码：首位数字为 8 或 9，第四位数字为 8 或 9，

# 第二位和第三位相同

如果 ((num1 == 8 或 num1 == 9) 并且

(num4 == 8 或 num4 == 9) 并且

(num2 == num3)):

print('ignore')

否则：

print('answer')

*列表 2-5：解决电话推销员问题，已添加注释*

我已添加了三行注释：顶部的 ❶ 提醒我们问题代码和名称，if 语句之前的两行 ❷ 提醒我们检测电话推销员电话号码的规则。

不要过度使用注释。尽可能编写不需要注释的代码。但对于复杂的代码，或者解释你为什么要以某种特定方式做事情，一个合适的注释可以在以后节省时间并避免挫折。

### 输入和输出重定向

当你将 Python 代码提交给裁判时，它会运行多个测试用例来判断代码是否正确。是不是有人在那里，尽职尽责地等待新的代码，并疯狂地从键盘输入测试用例？

不可能！这一切都是自动化的。没有人会在键盘上输入测试用例。那么，如果我们通过键盘输入某些内容来满足输入要求，裁判是如何测试我们的代码的呢？

事实上，输入不一定是从键盘读取的。它是从一个叫做*标准输入*的输入源读取的，默认情况下就是从键盘读取。

你可以改变标准输入，使其指向一个文件而不是键盘。这种技术叫做*输入重定向*，也是裁判用来提供输入的方式。

我们也可以尝试自己进行输入重定向。对于那些输入较小的程序——比如一行文本或者几个整数——输入重定向可能不会给我们节省太多时间。但对于那些测试用例有数十行或数百行长的程序，输入重定向可以让我们更轻松地测试我们的工作。我们不必一次次输入相同的测试用例，而是可以把它们存储在文件中，然后按需运行程序。

让我们在 Telemarketers 上尝试输入重定向。进入你的*programming*文件夹，创建一个名为*telemarketers_input.txt*的新文件。在该文件中输入以下内容：

8

1

1

9

题目要求我们每行提供一个整数，所以我们在这里也把它们写成每行一个。

保存文件。现在输入 `python telemarketers.py < telemarketers_input.txt` 来使用输入重定向运行你的程序。程序应该输出忽略，就像你直接从键盘输入测试用例一样。

`<` 符号指示你的操作系统使用文件而不是键盘提供输入。在 `<` 符号后面是包含输入的文件名。

要在不同的测试用例上测试你的程序，只需修改*telemarketers_input.txt*文件，然后再次运行你的程序。

我们也可以改变输出的去向，尽管在本书中不需要这么做。`print`函数默认输出到*标准输出*，通常是屏幕。我们可以改变标准输出，让它指向一个文件。我们通过*输出重定向*来实现这一点，写法是一个 > 符号后面跟着文件名。

输入 `python telemarketers.py > telemarketers_output.txt` 来使用输出重定向运行你的程序。提供四个整数的输入后，你应该会返回到操作系统提示符。但你不应该看到来自电话推销员程序的任何输出！这是因为我们已将输出重定向到文件 telemarketers_output.txt。如果你在文本编辑器中打开 telemarketers_output.txt，你应该能看到里面的输出。

小心使用输出重定向。如果你使用了一个已存在的文件名，旧文件会被覆盖！始终再次检查你是否使用了正确的文件名。

### 总结

在这一章，我们学习了如何使用 if 语句来控制程序的执行。if 语句的关键成分是布尔表达式，它是一个值为 True 或 False 的表达式。为了构建布尔表达式，我们使用关系运算符，比如==和>=，还使用布尔运算符，如 and 和 or。

根据 True 和 False 决定做什么，使得我们的程序更灵活，能够适应当前的情况。但是我们的程序仍然局限于处理小量的输入和输出——无论是通过单独的 input 和 print 调用能做到什么。在下一章，我们将开始学习循环结构，它允许我们重复代码，从而处理任意数量的输入和输出。

想处理 100 个值吗？那 1,000 个呢？而且只需要少量的 Python 代码？我知道现在挑衅你有点早，因为你还有以下的练习要做。但当你准备好了，继续往下读吧！

### 本章练习

这里有一些练习供你尝试。

1.  DMOJ 题目 ccc06j1, 加拿大卡路里计算

1.  DMOJ 题目 ccc15j1, 特别的日子

1.  DMOJ 题目 ccc15j2, 开心还是难过

1.  DMOJ 题目 dmopc16c1p0, C.C. 和奶酪君

1.  DMOJ 题目 ccc07j1, 谁在中间

### 备注

《获胜队伍》最初来源于 2019 年加拿大计算机竞赛，初级组。《电话推销员》最初来源于 2018 年加拿大计算机竞赛，初级组。
