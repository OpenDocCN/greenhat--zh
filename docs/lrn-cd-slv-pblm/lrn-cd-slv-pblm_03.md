## 3

重复代码：确定性循环

![image](Images/common.jpg)

计算机在我们需要它们反复执行某个过程时表现得非常出色。它们不知疲倦地做着我们要求的事情，无论是做10次、100次，还是10亿次。在这一章中，我们将学习关于循环的知识，循环是指令计算机重复执行程序的一部分的语句。

我们将使用循环来解决三个问题：追踪球在杯子下的位置，统计停车位的占用数量，以及确定手机套餐中可用的数据量。

### 问题#5：三只杯子

在这个问题中，我们将追踪杯子下球的位置，并随着杯子的位置变化进行跟踪。但由于杯子可能移动多次，所以我们不能为每次移动单独编写代码。相反，我们将学习并使用`for`循环，它允许我们更轻松地为每次移动执行代码。

这是DMOJ问题coci06c5p1。

#### 挑战

Borko有三只不透明的杯子：一只在左侧（位置1），一只在中间（位置2），一只在右侧（位置3）。球在左侧的杯子下。我们的任务是随着Borko交换杯子的位置，跟踪球的位置。

Borko可以进行三种类型的交换：

**A** 交换左侧和中间的杯子

**B** 交换中间和右侧的杯子

**C** 交换左侧和右侧的杯子

举例来说，如果Borko的第一次交换是A类型，那么他交换了左侧和中间的杯子；由于球起始在左侧，这次交换将它移到中间。如果第一次交换是B类型，那么他交换了中间和右侧的杯子；左侧的杯子保持不动，因此球的位置没有变化。

#### 输入

输入是一行最多50个字符。每个字符指定Borko进行的交换类型：A、B或C。

#### 输出

输出球的最终位置：

+   如果球在左侧，输出1

+   如果球在中间，输入2

+   如果球在右侧，输出3

### 为什么要使用循环？

考虑这个测试案例：

ACBA

这里有四次交换。为了确定球的最终位置，我们需要进行每一次交换。

第一次交换是A类型，交换左侧和中间的杯子。由于球起始在左侧，这使得球移动到中间。第二次交换是C类型，交换左侧和右侧的杯子。由于球当前在中间，这对球的位置没有影响。第三次交换是B类型，交换中间和右侧的杯子。这使球从中间移到右侧。第四次交换是A类型，交换左侧和中间的杯子。这对球没有影响。因此，正确的输出是3，因为球最终位于右侧。

注意，对于每次交换，我们都需要做出决定，判断球是否移动，并且如果移动了，要将球适当地移动。做决策是我们在[第二章](ch02.xhtml#ch02)中学到的内容。例如，如果交换类型是A，且球在左侧，那么球将移动到中间。这个过程如下所示：

如果 `swap_type == 'A'` 且 `ball_location == 1`：

`ball_location = 2`

我们可以为球移动的其他情况添加一个`elif`：当球在中间时交换类型A，当球在中间时交换类型B，当球在右侧时交换类型B，依此类推。这个大的`if`语句足以处理一次交换。但这不足以解决“三杯问题”，因为我们可能有一个最多包含50次交换的测试用例。我们需要为每一次交换重复`if`语句逻辑。而且我们肯定不想复制粘贴相同的代码50次。试想一下，如果你打错了一个字母，需要修复50次，或者如果你突然对最多包含百万次交换的测试用例感兴趣。现在，我们到目前为止学到的东西是远远不够的。我们需要一种方法来逐一处理这些交换，对每一个交换执行相同的逻辑。我们需要一个循环。

### `for`循环

Python的`for`语句产生*for循环*。`for`循环允许我们处理序列中的每个元素。到目前为止，我们看到的唯一序列类型是字符串。随着学习的深入，我们将学习其他类型；`for`循环适用于所有这些类型。

这是我们第一个`for`循环的例子：

>>> `secret_word = 'olive'`

>>> `for char in secret_word:` 

...     `print('Letter: ' + char)`

...

字母：o

字母：l

字母：i

字母：v

字母：e

在关键字`for`之后，我们写下一个*循环变量*的名称。循环变量是指在循环执行过程中表示不同值的变量。在一个字符串的`for`循环中，循环变量表示字符串的每个字符。

我选择了变量名`char`（表示“字符”）来提醒我们该变量表示的是字符串中的一个字符。有时，使用上下文相关的变量名会更加清晰。例如，在“三杯问题”中，我们可以使用`swap_type`这个名称来提醒我们它表示的是交换类型。

在变量名之后，我们写下关键字`in`，然后是我们想要循环遍历的字符串。在我们的例子中，我们正在遍历由`secret_word`表示的字符串，即'olive'。

和`if`、`elif`、`else`语句类似，`for`语句的行也以冒号（`:`）结尾。而且，和`if`语句一样，`for`语句也有一个缩进的代码块，其中包含一个或多个语句。

执行缩进语句的过程称为循环的*迭代*。以下是每次迭代时我们循环的执行过程：

+   在第一次迭代时，Python将`char`设置为表示'olive'中的第一个字符'o'。然后它运行循环块，这个块只包含对`print`的调用。由于`char`表示'o'，所以输出为`Letter: o`。

+   在第二次迭代时，Python将`char`设置为表示'olive'中的第二个字符'l'。然后它调用`print`，输出`Letter: l`。

+   这个过程还会再重复三次，每次处理'olive'中剩下的一个字符。

+   然后，循环终止。我们在循环之后没有其他代码，所以程序运行结束。如果循环之后有额外的代码，程序会继续执行这些代码。

You can put multiple statements in the block of a for loop. Here’s an example:

>>> secret_word = 'olive'

>>> for char in secret_word:

...     print('Letter: ' + char)

...     print('*')

...

Letter: o

*

Letter: l

*

Letter: i

*

Letter: v

*

Letter: e

*

Now we have two statements executing on each iteration of the loop: one that outputs the current letter of the string, and one that outputs a * character.

A for loop loops through the elements of a sequence, so the sequence’s length tells us how many iterations there will be. The len function takes a string and returns its length:

>>> len('olive')

5

Our for loop on 'olive' will therefore consist of five iterations:

>>> secret_word = 'olive'

❶ >>> print(len(secret_word), 'iterations, coming right up!')

>>> for char in secret_word:

...     print('Letter: ' + char)

...

5 iterations, coming right up!

Letter: o

Letter: l

Letter: i

Letter: v

Letter: e

I called print with multiple arguments ❶, rather than using concatenation, to avoid having to convert the length to a string.

A for loop is what’s called a *definite loop*, referring to the idea that the number of iterations is predetermined. There are also *indefinite loops*, whose iterations depend on the vagaries of what happens when your program runs. We’ll study those in the next chapter.

**CONCEPT CHECK**

What is the output of the following code?

s = 'garage'

total = 0

for char in s:

total = total + s.count(char)

print(total)

A. 6

B. 10

C. 12

D. 36

Answer: B. For each character in 'garage', we add its count to total. There are two g’s, two a’s, one r, two a’s (again!), two g’s (again!), and 1 e.

### Nesting

The for loop block is one or more statements. Those statements can include one-line statements such as function calls and assignment statements. But they can also include multiline statements such as if statements and loops.

Let’s start with an example of an if statement inside a for loop. Suppose we wanted to output only the uppercase characters from a string. Strings have an isupper method that we can use to determine whether a character is uppercase:

>>> 'q'.isupper()

False

>>> 'Q'.isupper()

True

We can use isupper in an if statement to control what happens on each iteration of a for loop:

>>> title = 'The Escape'

>>> for char in title:

...     if char.isupper():

...         print(char)

...

T

E

Be careful with the indentation here. We need one level of indentation for the for loop, and an extra level of indentation for the nested if statement.

On the first iteration, char refers to 'T'. Since 'T' is uppercase, the isupper test returns True, and the if statement block runs. That results in the output of T. On the second iteration, char refers to 'h'. This time, the isupper test returns False, so the if statement block doesn’t run. Overall, the for loop loops through each character of the string, but the nested if statement fires only twice: on the 'T' at the beginning of the string and on the 'E' at the beginning of 'Escape'.

那么，嵌套的 for 循环呢？我们可以这样做！这里有一个示例：

>>> letters = 'ABC'

>>> digits = '123'

>>> for letter in letters:

...     for digit in digits:

...         print(letter + digit)

...

A1

A2

A3

B1

B2

B3

C1

C2

C3

这段代码生成所有由两个字符组成的字符串，第一个字符来自字母表，第二个字符来自数字。

在外层（字母）循环的第一次迭代中，letter 指向 'A'。这次迭代完全运行内层（数字）循环。在内层循环运行的整个过程中，letter 都指向 'A'。在内层循环的第一次迭代中，digit 为 1，这就解释了 A1 的输出。在内层循环的第二次迭代中，digit 为 2，输出 A2。在内层循环的第三次也是最后一次迭代中，digit 为 3，输出 A3。

我们还没完成！我们只进行了外层循环的一次迭代。在外层循环的第二次迭代中，letter 变量指向 'B'。现在，内层循环的三次迭代再次运行，这次 letter 是 'B'。这就解释了 B1、B2 和 B3 的输出。最后，在外层循环的第三次迭代中，letter 是 'C'，内层循环生成了 C1、C2 和 C3。

**概念检查**

以下代码的输出是什么？

title = 'The Escape'

total = 0

for char1 in title:

for char2 in title:

total = total + 1

print(total)

A. 10

B. 20

C. 100

D. 这段代码会产生语法错误，因为两个嵌套循环不能都使用 title

答案：C。total 初始值为 0，并在内层循环的每次迭代中增加 1。'The Escape' 的长度是 10。外层循环因此进行 10 次迭代。每次迭代，内层循环都会进行 10 次迭代。内层循环总共会进行 10 * 10 = 100 次迭代。

### 解决问题

回到三杯问题。我们需要的结构是一个 for 循环来遍历每次交换，一个嵌套的 if 语句来跟踪球的位置：

for swap_type in swaps:

# 大的 if 语句来跟踪球的位置

有三种交换类型（A、B 和 C）和三种可能的位置，因此很容易得出结论，必须写一个包含 3 * 3 = 9 个布尔表达式的 if 语句（每个 elif 后面跟一个）。事实上，我们只需要六个布尔表达式。九个中有三个不会移动球的位置：当球在右边时是交换类型 A，球在左边时是交换类型 B，球在中间时是交换类型 C。

[清单 3-1](ch03.xhtml#ch03ex01) 解决了三杯问题。

swaps = input()

ball_location = 1

❶ for swap_type in swaps:

❷ if swap_type == 'A' and ball_location == 1:

❸ ball_location = 2

elif swap_type == 'A' and ball_location == 2:

ball_location = 1

elif swap_type == 'B' and ball_location == 2:

ball_location = 3

elif swap_type == 'B' and ball_location == 3:

ball_location = 2

elif swap_type == 'C' and ball_location == 1:

ball_location = 3

elif swap_type == 'C' and ball_location == 3:

ball_location = 1

print(ball_location)

*Listing 3-1: 解决三杯问题*

我使用input将交换字符串赋给了swaps变量。for循环❶遍历这些交换。每个交换都由嵌套的if语句❷处理。if和elif分支分别编码了在给定交换类型和球位置时发生的事情，并相应地移动球。例如，如果交换类型是A，球在位置1❷，那么球会最终到达位置2❸。

这是一个代码示例，说明我们是否使用多个`elif`（一个大的if语句）或多个`if`（多个if语句）很重要。如果我们将`elif`改为`if`，那么代码就不再正确了。[Listing 3-2](ch03.xhtml#ch03ex02)展示了不正确的代码。

# 这段代码是错误的

swaps = input()

ball_location = 1

for swap_type in swaps:

❶ if swap_type == 'A' and ball_location == 1:

ball_location = 2

❷ if swap_type == 'A' and ball_location == 2:

ball_location = 1

if swap_type == 'B' and ball_location == 2:

ball_location = 3

if swap_type == 'B' and ball_location == 3:

ball_location = 2

if swap_type == 'C' and ball_location == 1:

ball_location = 3

if swap_type == 'C' and ball_location == 3:

ball_location = 1

print(ball_location)

*Listing 3-2: 错误地解决三杯问题*

如果我们说代码是错误的，我们就是在声明它至少在某个测试用例中失败了。你能找到一个测试用例，这段代码产生了错误的结果吗？

这是一个这样的测试用例：

A

对我们来说，可能很合理认为每次交换时，球最多只能移动一次。但Python会机械地执行你编写的代码，无论它是否符合我们的预期。在这种情况下，我们只有一次交换，所以球应该最多移动一次。在for循环的第一次也是唯一一次迭代中，Python会检查表达式❶。它为True，因此Python将ball_location设置为2。然后，Python检查表达式❷。因为我们刚刚将ball_location更改为2，所以这个表达式为True！因此，Python将ball_location设置为1。程序的输出是1，但它应该是2。

这是一个*逻辑错误*的例子：一种导致程序执行错误逻辑并产生错误答案的错误。逻辑错误的常用术语是*bug*。当程序员修复代码中的bug时，这个过程被称为*调试*。

通常只需要一个简单的测试用例就能展示程序何时是错误的。当你在尝试找出代码中出错的地方时，不要从长的测试用例开始。这样的测试结果很难人工验证，而且往往会触发复杂的执行路径，我们可能从中学不到多少东西。相反，一个小的测试用例不会让程序做太多事情；如果它做错了什么，我们就不用找太远就能找到问题所在。设计小而有针对性的测试用例并不总是容易的。这是一项你可以通过实践来锻炼的技能。

提交我们正确的代码给评审，然后继续进行。

在继续之前，你可以尝试解决《章节练习》中第 1 题和第 2 题，见 [第 67 页](ch03.xhtml#ch03lev1sec16)。

### 问题 #6：占用的停车位

我们知道如何遍历字符串的字符。但有时我们需要知道自己在字符串中的位置，而不仅仅是存储在那里字符的内容。这道题就是一个这样的例子。

这是 DMOJ 问题 ccc18j2。

#### 挑战

你管理一个有 *n* 个停车位的停车场。昨天，你记录了每个停车位是否被车占用或者是否为空。今天，你再次记录了每个停车位是否被车占用或者是否为空。请计算出这两天都被占用的停车位数量。

#### 输入

输入包含三行。

+   第一行包含整数 *n*，即停车位的数量。*n* 在 1 到 100 之间。

+   第二行包含一个长度为 *n* 的字符串，表示昨天的信息，每个字符代表一个停车位。C 表示一个被占用的停车位（C 代表车），而 . 表示一个空的停车位。例如，CC. 表示前两个停车位被占用，第三个停车位为空。

+   第三行包含一个长度为 *n* 的字符串，表示今天的信息，格式与第二行相同。

#### 输出

输出两天都被占用的停车位数量。

### 一种新的循环方式

我们可能有最多 100 个停车位，所以你可能不惊讶于循环会在某处出现。我们在解决“三杯问题”时学到的那种 for 循环，当然可以遍历停车位信息的字符串：

>>> yesterday = 'CC.'

>>> for parking_space in yesterday:

...     print('该停车位是 ' + parking_space)

...

该停车位是 C

该停车位是 C

该停车位是 .

这告诉我们每个停车位昨天是否被占用。但我们还需要知道每个停车位今天是否也被占用。

考虑这个测试案例：

3

CC.

.C.

第一个停车位昨天被占用了。这个停车位今天也被占用了么？为了回答这个问题，我们需要查看今天字符串中对应的字符。它是 .（空的），所以这个停车位两天都没有被占用。

那么第二个停车位怎么样呢？它昨天也被占用了。而且，看今天字符串的第二个字符，它今天也被占用了。所以这是一个两天都被占用的停车位。（这是唯一一个这样的停车位；这个测试案例的正确输出是 1。）

遍历一个字符串的字符无法帮助我们找到另一个字符串中的对应字符。但如果我们能跟踪自己在字符串中的位置——我们在第一个停车位，第二个停车位，以此类推——我们就可以查找每个字符串中的对应字符。到目前为止，我们学到的 for 循环并不是实现这一点的方式。实现这一点的方法是使用索引和一种新的 for 循环类型。

### 索引

字符串中的每个字符都有一个 *索引*，它表示该字符的位置。第一个字符的索引是 0，第二个字符的索引是 1，以此类推。在自然语言中，我们通常从 1 开始计数。英语中没有人会说“*hello* 中第 0 位的字符是 *h*。”但是大多数编程语言，包括 Python，都是从 0 开始的。

要使用索引，我们在字符串后面加上一个方括号中的索引。以下是一些索引示例：

>>> word = 'splore'

>>> word[0]

's'

>>> word[3]

'o'

>>> word[5]

'e'

如果我们愿意，可以在索引中使用变量：

>>> where = 2

>>> word[where]

'l'

>>> word[where + 2]

'r'

我们可以在非空字符串上使用的最高索引是它的长度减 1。（空字符串没有有效的索引。）例如，'splore' 的长度是 6，所以索引 5 是它的最高索引。再大就会出错：

>>> word[len(word)]

Traceback (most recent call last):

文件 "<stdin>"，第 1 行，在 <module> 中

IndexError: 字符串索引超出范围

>>> word[len(word) - 1]

'e'

我们如何访问字符串从右数的第二个字符呢？可以这样做：

>>> word[len(word) - 2]

'r'

但是有一个更简单的方法。Python 支持负数索引作为访问字符的另一种选择。索引 -1 是最右侧的字符，索引 -2 是从右数的第二个字符，以此类推：

>>> word[-2]

'r'

>>> word[-1]

'e'

>>> word[-5]

'p'

>>> word[-6]

's'

>>> word[-7]

Traceback (most recent call last):

文件 "<stdin>"，第 1 行，在 <module> 中

IndexError: 字符串索引超出范围

计划是使用索引来访问昨天和今天停车信息中相应的位置。我们可以使用每个字符串的索引 0 来访问第一个停车位的信息，使用索引 1 来访问第二个停车位的信息，以此类推。但在执行这一计划之前，我们需要学习一种新的 for 循环类型。

**概念检查**

以下代码的输出是什么？

s = 'abcde'

t = s[0] + s[-5] + s[len(s) - 5]

print(t)

A. aaa

B. aae

C. aee

D. 这段代码会产生一个错误

答案：A. 这三个索引都指向 'abcde' 中的第一个字符。首先，s[0] 指向 'a'，因为 'a' 位于字符串的索引 0 处。其次，s[-5] 指向 'a'，因为 'a' 是从右数的第五个字符。第三，s[len(s) - 5] 指向 'a'，因为索引的计算结果是 0：5（字符串的长度）减去 5。

### 范围 for 循环

Python 的 range 函数生成整数范围，我们可以使用这些范围来控制 for 循环。与其循环字符串中的字符，不如使用一个整数范围的 for 循环。如果我们给 range 提供一个参数，将得到从 0 到该参数减 1 的范围：

>>> for num in range(5):

...     print(num)

...

0

1

2

3

4

请注意，5 没有输出。

如果我们给 range 提供两个参数，我们将得到一个从第一个参数开始，到第二个参数为止但不包括第二个参数的序列：

>>> for num in range(3, 7):

...     print(num)

...

3

4

5

6

我们可以通过包含第三个参数来以不同的 *步长* 递增。默认的步长是 1，即每次递增 1。让我们尝试其他几种步长：

>>> for num in range(0, 10, 2):

...     print(num)

...

0

2

4

6

8

>>> for num in range(0, 10, 3):

...     print(num)

...

0

3

6

9

我们也可以倒数，但*不是*这样：

>>> for num in range(6, 2):

...     print(num)

...

这不起作用，因为默认情况下 range 是递增的。步长为 -1 可以让我们一步一步地倒退：

>>> for num in range(6, 2, -1):

...     print(num)

...

6

5

4

3

要从 6 倒计时到 0（包括 0），我们需要第二个参数为 -1：

>>> for num in range(6, -1, -1):

...     print(num)

...

6

5

4

3

2

1

0

有时候，快速查看一个范围内的数字而不编写循环是很有帮助的。不幸的是，range 函数并不会直接显示这些数字：

>>> range(3, 7)

range(3, 7)

我们可以将结果传递给 list 函数，来获取我们想要的结果：

>>> list(range(3, 7))

[3, 4, 5, 6]

当使用 range 时，list 函数会生成一个范围内整数的列表。稍后我们会学习关于列表的更多内容；现在，记住列表作为诊断范围错误的辅助工具。

**概念检查**

以下循环会执行多少次迭代？

for i in range(10, 20):

# 这里有一些代码

A. 9

B. 10

C. 11

D. 20

答案：B。范围遍历了数字 10、11、12、13、14、15、16、17、18 和 19，共有 10 个数字，因此有 10 次迭代。

### 通过索引的范围循环

假设我们有表示昨天和今天停车位信息的字符串：

>>> yesterday = 'CC.'

>>> today = '.C.'

给定一个索引，我们可以查看昨天和今天该索引处的信息：

>>> yesterday[0]

'C'

>>> today[0]

'.'

我们可以使用 range 来通过索引循环，处理每一对对应的字符。我们知道昨天和今天的字符串长度相同。但这个长度可以是从 1 到 100 之间的任何值，所以我们不能像 range(3) 这样写。我们想要以 0、1、2 等索引迭代，一直到字符串的长度减去 1。我们可以通过将其中一个字符串的长度作为 range 的参数来实现：

>>> for index in range(len(yesterday)):

...     print(yesterday[index], today[index])

...

C .

C C

. .

我将循环变量命名为 index。其他常见的命名方式包括 i（index 的第一个字母）和 ind。从这里开始我将使用 i。

不要将这个循环变量命名为 status 或 information。这些名称暗示它取的是 'C' 和 '.' 值，而实际上它取的是整数。

### 解决问题

使用我们的 range 循环，我们准备好解决“占用空间”问题了。我们的策略是从字符串的开头到结尾，逐个检查每个索引处的信息。我们可以检查昨天和今天每个索引处的信息，并使用嵌套的 if 语句来判断两个日期的停车位是否被占用。

[列表 3-3](ch03.xhtml#ch03ex03)包含了我们的解决方案。

n = int(input())

yesterday = input()

today = input()

❶ occupied = 0

❷ for i in range(len(yesterday)):

❸ 如果昨天[i] == 'C' 且今天[i] == 'C'：

❹ occupied = occupied + 1

print(occupied)

*列表3-3：解决占用停车位问题*

程序首先读取三行输入：n表示停车位的数量；yesterday和today分别表示昨天和今天的停车位信息。

请注意，我们不再提及停车位数量（n）。我们本可以利用它来告诉我们字符串的长度，但我选择忽略它，因为在现实场景中，它通常没有提供。

我们使用occupied变量来计算昨天和今天被占用的停车位数量。我们将这个变量初始化为0 ❶。

现在我们进入range循环，它遍历昨天和今天的有效索引 ❷。对于每个索引，我们检查该停车位昨天和今天是否都被占用 ❸。如果是，那么我们就将这个停车位包含在我们的总数中，将occupied加1 ❹。

当range循环结束时，我们将遍历所有停车位。昨天和今天被占用的停车位总数可以通过occupied变量访问。剩下的就是输出这个总数。

这就是这个问题的全部内容。现在可以将代码提交给评审员了。

### 问题 #7：数据计划

我们已经学到，for循环在处理输入数据后非常有用。它们在读取数据时也常常非常有用。在这个问题中，我们将处理分布在多行中的数据，并使用for循环帮助我们读取这些数据。

这是DMOJ问题coci16c1p1。

#### 挑战任务

Pero与他的手机运营商签订了一个数据计划，每月提供给他*x*兆字节的数据。此外，任何在一个月内未使用的数据会转移到下个月。例如，如果*x*是10，而Pero在一个月内只使用了4MB，那么剩余的6MB将转移到下个月（在下个月，他将拥有10 + 6 = 16MB的可用数据）。

我们得到Pero在前*n*个月使用的兆字节数。我们的任务是确定下个月可用的兆字节数。

#### 输入

输入包括以下几行：

+   一行包含整数*x*，表示Pero每月分配的兆字节数。*x*的值在1到100之间。

+   一行包含整数*n*，表示Pero拥有数据计划的月份数。*n*的值在1到100之间。

+   *n*行，每行给出Pero在该月使用的兆字节数。每个数字至少为0，且不会超过可用的兆字节数。（例如，如果*x*是10，而Pero目前有30MB可用，下一个数字最多为30。）

#### 输出

输出下个月可用的兆字节数。

### 循环读取输入

到目前为止，我们的所有问题都已明确知道需要从输入中读取多少行。例如，在《三杯》中，我们读取一行；在《占用空间》中，我们读取三行。而在《数据计划》中，我们不知道需要读取多少行，因为这取决于我们从第二行读取的数字。

我们可以读取输入的第一行：

monthly_mb = int(input())

（我使用变量名 monthly_mb 而不是 x，是为了让它有更明确的含义。）

然后我们可以读取输入的第二行：

n = int(input())

但是如果没有循环，我们就无法继续读取更多的数据。这里使用范围循环是完美的，因为我们可以用它准确地循环 n 次：

for i in range(n):

# 处理月份

### 解决问题

我的解决策略是跟踪从前几个月结转的数据量。我将其称为 *excess*。

请考虑这个测试用例：

10

3

4

12

1

每个月，Pero 会得到 10MB 的数据，我们需要处理他在提供的三个月中使用的数据。在第一个月，Pero 获得了 10MB，使用了 4MB，因此结转的剩余数据是 6MB。第二个月，Pero 再次获得 10MB，总共拥有 16MB。他在本月使用了 12MB，因此结转的剩余数据是 16 - 12 = 4MB。在第三个月，Pero 又获得了 10MB，总共有 14MB。他在本月使用了 1MB，因此结转的剩余数据是 14 - 1 = 13MB。

我们需要知道 Pero 在下个月（即第四个月）可用的兆字节数。他从前三个月结转的 13MB，再加上本月的 10MB，因此他总共有 13 + 10 = 23MB 可用。

当我根据这个解释编写代码时，我忽略了添加最后的 10MB，因此我的输出是 13，而不是 23。我只专注于 excess，忘记了我们需要的不是结转到下个月的 excess，而是下个月可用的兆字节总数。这个总数是 excess 加上 Pero 每个月得到的数据量。

请参阅[列出 3-4](ch03.xhtml#ch03ex04)查看（已修正的！）代码。

monthly_mb = int(input())

n = int(input())

excess = 0

❶ for i in range(n):

used = int(input())

❷ excess = excess + monthly_mb - used

❸ print(excess + monthly_mb)

*列出 3-4：解决数据计划问题*

excess 变量从 0 开始。在每次范围循环的迭代中，我们给 excess 赋一个值，考虑到每个月给定的兆字节数和本月使用的兆字节数。

范围循环会循环 n 次，每次处理 Pero 所拥有的数据计划月份 ❶。i 的取值—0、1，依此类推—对我们来说并不重要，因为我们没有理由关心我们正在处理的是哪个月份。因此，我们在程序中并没有使用 i 的值。你可以将 i 替换为 _（下划线），明确表示该变量是“无关紧要”的状态，但为了与其他示例一致，我会保留它为 i。

在范围 for 循环中，我们读取本月使用的兆字节数。然后，我们更新超出部分的兆字节数 ❷：它是之前的值，加上Pero每月获得的兆字节数，减去Pero本月使用的兆字节数。

计算出经过 n 个月后的超出兆字节数后，我们报告下个月可用的兆字节数 ❸。

解决问题的方式总是有很多种。编程是富有创意的，我很喜欢观察人们提出的各种解决策略。即使你已经成功解决了问题，也可以通过Google搜索问题，看看别人是怎么解决的。此外，一些在线判题系统，如DMOJ，允许你在解决问题后查看其他人的提交。在那些通过所有测试用例的提交中：这些程序员和你的做法有何不同？在那些未通过部分测试用例的提交中：代码出了什么问题？阅读他人的代码是提升自己编程技能的好方法！

你能想到解决数据计划的另一种方法吗？

这是一个提示：你可以从计算Pero获得的总兆字节数开始，然后减去他使用的兆字节数。我鼓励你在继续之前花点时间思考如何做到这一点！

给Pero的总兆字节数，包括下个月分配的部分，是 *x* * (*n* + 1)，其中 *x* 是每月分配的兆字节数。为了确定下个月可用的兆字节数，我们可以从这个总数开始，然后减去Pero每个月使用的兆字节数。这种策略在[清单 3-5](ch03.xhtml#ch03ex05)中有代码实现。

monthly_mb = int(input())

n = int(input())

total_mb = monthly_mb * (n + 1)

for i in range(n):

used = int(input())

total_mb = total_mb - used

print(total_mb)

*清单 3-5：解决数据计划，替代方法*

选择你最喜欢的解决方案，提交给判题系统。

对一个人来说直观的东西，可能对另一个人并不直观。你可能读到某些解释或代码时无法理解。这并不意味着你不够聪明，而是说明你需要另一种更贴合你当前思维方式的呈现方式。你也可以将困难的解释和例子标记下来，稍后再复习。一旦你有了更多的练习，它们可能会变得非常有用。

### 总结

本章我们学习了 for 循环。标准的 for 循环遍历序列中的字符；range for 循环遍历一个范围内的整数。我们解决的每一个问题都需要处理许多输入数据，没有循环我们无法管理这些数据。

`for` 循环是每当你需要重复代码指定次数时的首选循环。Python 还有另一种类型的循环，我们将在下一章学习如何使用它。除了 `for` 循环，我们为什么还需要其他循环？`for` 循环不能做什么？这些问题问得很好！我现在告诉你：练习 `for` 循环是为将来要学的内容做好准备的绝佳方式。

### 章节练习

这里有一些练习供你尝试。

1.  DMOJ 问题 wc17c3j3, 无法破解

1.  DMOJ 问题 coci18c3p1, Magnus

1.  DMOJ 问题 ccc11s1, 英语或法语

1.  DMOJ 问题 ccc11s2, 多项选择

1.  DMOJ 问题 coci12c5p1, Ljestvica

1.  DMOJ 问题 coci13c3p1, Rijeci

1.  DMOJ 问题 coci18c4p1, Elder

### 注释

三杯问题最初来自2006/2007年克罗地亚信息学公开赛，第5场。占据空间问题最初来自2018年加拿大计算机竞赛，初级组。数据计划问题最初来自2016/2017年克罗地亚信息学公开赛，第1场。
