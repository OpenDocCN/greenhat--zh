## 使用*函数*设计程序

![image](img/common.jpg)

在编写大型程序时，组织代码为更小的逻辑单元非常重要，每个单元都为实现整体目标做出贡献。这样，我们就能单独考虑每个单元，而不用担心其他单元在做什么。然后我们将这些单元组合在一起。这些单元被称为*函数*。

在本章中，我们将使用函数来分解并解决两个问题：计算双人卡牌游戏的得分以及判断动作人偶盒子是否能够合理组织。

### 问题 #14：卡牌游戏

在这个问题中，我们将实现一个双人卡牌游戏。在思考问题的过程中，我们会发现相同的逻辑会多次出现。我们将学习如何将这部分代码封装成一个 Python 函数，以避免代码重复并提高代码的清晰度。

这是 DMOJ 问题 `ccc99s1`。

#### 挑战

两名玩家，A 和 B，正在进行一场卡牌游戏。（你不需要了解扑克牌或卡牌游戏才能理解这个问题。）

游戏开始时有一副 52 张牌。玩家 A 从牌堆中抽取一张牌，然后玩家 B 抽取一张牌，然后是玩家 A，再然后是玩家 B，直到牌堆中没有牌为止。

牌堆中有 13 种类型的牌。这些类型如下：二、三、四、五、六、七、八、九、十、杰克、皇后、国王、王牌。每种类型的牌在牌堆中都有四张。例如，有四张二、四张三，以此类推，一直到四张王牌。（这就是为什么牌堆中有 52 张牌：13 种类型乘以每种类型 4 张牌。）

*高牌*是指杰克、皇后、国王或王牌。当玩家抽到一张高牌时，他们可能会得分。以下是得分规则：

+   如果玩家抽到一张杰克，且此时牌堆中至少还有一张牌，并且下一张牌不是高牌，那么该玩家得 1 分。

+   如果玩家抽到一张皇后，且此时牌堆中至少还有两张牌，并且接下来的两张牌中没有高牌，那么该玩家得 2 分。

+   如果玩家抽到一张国王，且此时牌堆中至少还有三张牌，并且接下来的三张牌中没有高牌，那么该玩家得 3 分。

+   如果玩家抽到一张王牌，且此时牌堆中至少还有四张牌，并且接下来的四张牌中没有高牌，那么该玩家得 4 分。

我们需要在每次玩家得分时输出信息，并在游戏结束时输出每个玩家的总分。

#### 输入

输入由 52 行组成。每一行包含一张牌的类型。这些行是从牌堆中抽取的顺序；也就是说，第一行是从牌堆中抽取的第一张牌，第二行是第二张牌，以此类推。

#### 输出

每当玩家得分时，输出以下内容：

```py
Player p scores q point(s).
```

其中，p 代表玩家 A 为`A`，玩家 B 为`B`，q 代表他们刚得的分数。

游戏结束时，输出以下两行：

```py
Player A: m point(s).

Player B: n point(s).
```

其中，m 是玩家 A 的总分，n 是玩家 B 的总分。

### 探索一个测试用例

如果你仔细考虑如何解决这个问题，你可能会想，是否可以在不学习任何新知识的情况下，直接解决它。事实上，我们可以！我们处于非常有利的状态。我们可以用列表表示牌堆。我们知道如何使用列表的`append`方法将一张牌添加到牌堆中。我们可以访问列表中的值来查找高牌。我们甚至可以使用 f-strings 来帮助我们输出玩家和分数的信息。

不过，在深入探讨之前，我们先通过一个小例子来演示。这样做将突显出我们缺少一个 Python 中至关重要的特性，这个特性将使得我们更容易组织解决方案并解决这个问题。

如果我们用一副 52 张牌的例子来演示，那我们得忙到明年了。所以我们使用一个只有 10 张牌的小例子。这个例子不是一个完整的测试用例，所以我们写的程序无法在其上运行，但它足以帮助我们理解游戏的机制以及我们的解决方案需要做些什么。以下是测试用例：

```py
queen

three

seven

king

nine

jack

eight

king

jack

four
```

玩家 A 拿到了第一张牌，那是一个皇后牌。皇后牌是高牌，玩家 A 可能会得到 2 分。首先，我们确认这张皇后之后，牌堆中至少剩下两张牌。接着，我们需要检查这两张牌，希望其中没有高牌。这两张牌不是高牌——分别是三号牌和七号牌——所以玩家 A 得到了 2 分。

玩家 B 现在拿到第二张牌，那是三号牌。三号牌不是高牌，所以玩家 B 没有得分。

玩家 A 现在拿到七号牌，没有得分。

玩家 B 现在拿到国王牌，因此有机会得到 3 分。国王牌之后，牌堆中至少剩下三张牌。我们需要检查这三张牌，希望其中没有高牌。不幸的是，其中有一张高牌——杰克——所以玩家 B 没有得分。

玩家 A 现在拿到九号牌，没有得分。

玩家 B 现在拿到第一张杰克牌。在这张杰克牌之后，牌堆中至少剩下一张牌。我们需要检查这张牌，希望它不是高牌。好消息：它不是高牌——是八号牌——所以玩家 B 得到了 1 分。

只剩下最后一分了，这一分由玩家 A 获得，当他们从牌堆中拿到倒数第二张牌（杰克）时。

因此，这是这个测试用例的输出：

```py
Player A scores 2 point(s).

Player B scores 1 point(s).

Player A scores 1 point(s).

Player A: 3 point(s).

Player B: 1 point(s).
```

注意，每次玩家拿到一张高牌时，我们需要检查两件事：第一，剩余的牌堆中是否至少还有一定数量的牌；第二，剩下的牌中是否没有高牌。第一个问题我们可以通过一个变量来管理，它会告诉我们已经拿了多少张牌。第二个问题则更复杂。我们需要一些代码来检查给定数量的牌中是否有高牌。更糟糕的是，如果我们不小心，我们会重复写四次非常相似的代码：一次检查拿到一张杰克后的牌，一次检查拿到一张皇后的两张牌，一次检查拿到一张国王后的三张牌，最后一次检查拿到一张王牌后的四张牌。如果我们后来发现逻辑有问题，我们就必须在多达四个不同的地方进行修正。

是否有 Python 特性可以让我们将这段“没有高牌”的逻辑仅封装一次，然后调用四次呢？有的。这就是所谓的*函数*，它是一个命名的代码块，用来执行一个小任务。函数对于我们代码的组织和清晰度至关重要。所有程序员都会使用它们。如果没有函数，编写像游戏和文字处理器这样的庞大软件系统将变得不可行。让我们学习如何使用函数。

### 定义和调用函数

我们已经学习了如何调用 Python 自带的函数。例如，我们使用了`input`函数来读取输入。这里是没有参数的`input`函数调用：

```py
>>> s = input()

hello

>>> s

'hello'
```

我们还使用了 Python 的`print`函数来输出文本。这里是带有一个参数的`print`函数调用：

```py
>>> print('well, well')

well, well
```

内建的 Python 函数是通用的，旨在在各种不同的场景中使用。当我们需要一个函数来解决特定任务时，我们就得自己定义一个。

#### 没有参数的函数

要*定义*或创建一个函数，我们使用 Python 的`def`关键字。这里是一个输出三行的函数定义：

```py
>>> def intro():

...     print('*********')

...     print('*WELCOME*')

...     print('*********')

...
```

函数定义的结构类似于`if`语句或循环。`def`后面的名称是我们正在定义的函数名；在这里，我们定义了一个名为`intro`的函数。在函数名后面是一个空的圆括号`()`。稍后我们会看到，我们可以在这些括号中包含信息来传递参数给函数。这个`intro`函数没有接受任何参数，所以括号是空的。括号后面是一个冒号；就像`if`语句或循环一样，省略冒号会导致语法错误。在接下来的行中，我们提供了一段缩进的语句块，每次调用该函数时，这些语句将被执行。

当你定义`intro`函数时，你可能希望看到如下的输出：

```py
*********

*WELCOME*

*********
```

但是不行：到目前为止我们只是定义了函数，并没有调用它。定义函数不会产生任何可见的效果；它只是将函数存储在计算机的内存中，以便我们稍后调用。我们调用自定义函数的方式就像调用 Python 的内置函数一样。由于这个`intro`函数不需要任何参数，我们在调用时使用一个空的括号：

```py
>>> intro()

*********

*WELCOME*

*********
```

你可以根据需要多次调用这个函数。它会在我们需要时随时存在。

#### 带参数的函数

我们的`intro`函数不够灵活，因为每次调用时它做的事情都是一样的。我们可以修改这个函数，使得我们可以传入参数，并且传入的参数可以影响函数的行为。这里是一个新的`intro`函数版本，它允许我们传入一个参数：

```py
>>> def intro2(message):

...     line_length = len(message) + 2

...     print('*' * line_length)

...     print(f'*{message}*')

...     print('*' * line_length)

...
```

要调用这个函数，我们提供一个字符串参数：

```py
>>> intro2('HELLO')

*******

*HELLO*

*******

>>> intro2('WIN')

*****

*WIN*

*****
```

我们不能在没有参数的情况下调用`intro2`函数——如果尝试调用，将会报错：

```py
>>> intro2()

Traceback (most recent call last):

  File "<stdin>", line 1, in <module>

TypeError: intro2() missing 1 required positional argument: 'message'
```

错误提醒我们没有为`message`提供参数。`message`是一个函数*参数*。当我们调用`intro2`时，Python 首先让`message`指向我们传入的参数；也就是说，`message`成为了我们参数的别名。

我们可以创建具有多个参数的函数。这里是一个需要两个参数的函数，一个是要打印的消息，另一个是打印的次数：

```py
>>> def intro3(message, num_times):

...     for i in range(num_times):

...         print(message)

...
```

要调用这个函数，我们提供两个参数。Python 从左到右工作，将第一个参数赋给第一个参数，第二个参数赋给第二个参数。在以下调用中，`'high'`被赋值给`message`参数，`5`被赋值给`num_times`参数：

```py
>>> intro3('high', 5)

high

high

 high

high

high
```

一定要提供正确数量的参数。对于`intro3`，我们需要两个参数。任何其他情况都会导致错误：

```py
>>> intro3()

Traceback (most recent call last):

  File "<stdin>", line 1, in <module>

TypeError: intro3() missing 2 required positional arguments: 'message'

and 'num_times'

>>> intro3('high')

Traceback (most recent call last):

  File "<stdin>", line 1, in <module>

TypeError: intro3() missing 1 required positional argument: 'num_times'
```

我们还必须确保提供正确类型的值。错误的类型不会阻止我们调用函数，但会导致函数内部出错：

```py
>>> intro3('high', 'low')

Traceback (most recent call last):

  File "<stdin>", line 1, in <module>

  File "<stdin>", line 2, in intro3

TypeError: 'str' object cannot be interpreted as an integer
```

这个`TypeError`错误是由于`intro3`使用了`for`循环遍历`num_times`变量。如果我们为`num_times`提供的参数不是整数，`for`循环就会失败。

#### 关键字参数

在调用函数时，实际上可以覆盖参数和参数值从左到右的对应关系。为了做到这一点，我们可以按照任何顺序使用参数的名称。使用参数名称的参数被称为*关键字参数*。下面是它的工作方式：

```py
>>> def intro3(message, num_times):

...     for i in range(num_times):

...         print(message)

...

>>> intro3(message='high', num_times=3)

high

high

high

>>> intro3(num_times=3, message='high')

 high

high

high
```

这里的每个函数调用都使用了两个关键字参数。关键字参数的写法是参数名、等号和对应的参数值。

你甚至可以先使用普通参数，然后再使用关键字参数：

```py
>>> intro3('high', num_times=3)

high

high

high
```

但是一旦你使用了关键字参数，就不能再回到普通参数了：

```py
>>> intro3(message='high', 3)

  File "<stdin>", line 1

SyntaxError: positional argument follows keyword argument
```

在第五章的“排序列表”中，我们在调用`sort`方法时使用了`reverse`关键字参数。Python 的设计者决定将`reverse`设置为仅限关键字的参数，这意味着在没有使用关键字参数的情况下，无法为它填充值。Python 也允许我们在函数中使用这种方式，但在本书中我们不需要这种级别的控制。

#### 局部变量

参数的名称就像普通变量一样工作，但它们是*局部*的，仅限于定义它们的函数。也就是说，函数参数在其函数外部是不存在的：

```py
>>> def intro2(message):

...     line_length = len(message) + 2

...     print('*' * line_length)

...     print(f'*{message}*')

...     print('*' * line_length)

...

>>> intro2('hello')

*******

*hello*

*******

>>> message

Traceback (most recent call last):

  File "<stdin>", line 1, in <module>

NameError: name 'message' is not defined
```

那个`line_length`变量怎么样，是局部变量吗？是的：

```py
>>> line_length

Traceback (most recent call last):

  File "<stdin>", line 1, in <module>

NameError: name 'line_length' is not defined
```

如果你有一个变量，并且调用一个使用相同名称的参数或局部变量的函数，会发生什么？你的值会丢失吗？让我们来看一下：

```py
>>> line_length = 999

>>> intro2('hello')

*******

*hello*

*******

>>> line_length

999
```

呼——它仍然是`999`，就像我们离开时一样。局部变量在函数被调用时创建，在函数结束时销毁，而不会影响其他具有相同名称的变量。

一个函数可以访问在函数外部创建的变量。然而，依赖于这种做法并不明智，因为这样会导致该函数不具备自我封装性，而是依赖于期望存在的变量是否真的存在。在本书中，我们将编写只使用局部变量的函数。函数所需的所有信息都会通过参数提供给它。

#### 可变参数

由于参数是对应参数的别名，它可以用来改变一个可变值。这里有一个函数，用来从列表`lst`中移除所有出现的`value`：

```py
>>> def remove_all(lst, value):

...     while value in lst:

...         lst.remove(value)

...

>>> lst = [5, 10, 20, 5, 45, 5, 9]

>>> remove_all(lst, 5)

>>> lst

[10, 20, 45, 9]
```

注意，我们通过使用变量传递了一个列表给`remove_all`。如果你直接传递一个列表值（而不是一个引用该列表的变量），这个函数将不会完成任何有用的操作：

```py
>>> remove_all([5, 10, 20, 5, 45, 5, 9], 5)
```

该函数移除了列表中的所有 5，但由于我们没有使用一个变量，我们再也无法引用这个列表。

**概念检查**

以下代码的输出是什么？

```py
def mystery(s, lst):

    s = s.upper()

    lst = lst + [2]

s = 'a'

lst = [1]

mystery(s, lst)

print(s, lst)
```

A. `a [1]`

B. `a [1, 2]`

C. `A [1]`

D. `A [1, 2]`

答案：A. 当调用`mystery`时，它的`s`参数被赋值为`s`参数所指向的内容，也就是`'a'`字符串。类似地，它的`lst`参数被赋值为`lst`参数所指向的内容，也就是`[1]`列表。在`mystery`函数内部，`s`和`lst`是局部变量。

现在，让我们研究一下该函数本身的两个语句。

首先，`s = s.upper()`。这将使局部变量`s`指向`'A'`（大写）。但它并没有改变函数外部`s`所指向的内容。外部的`s`仍然指向`'a'`（小写）。

其次，`lst = lst + [2]`。使用`+`运算符和列表会创建一个新列表（它不会改变现有的列表！），所以这会让局部变量`lst`指向新的列表：[1, 2]。但同样，它并没有改变函数外部`lst`所指向的内容；它仍然是`[1]`。

怎么回事——我之前不是告诉过你函数可以改变可变参数吗？我说过；但是为了实现这一点，你真的需要改变值本身，而不是改变局部变量指向的内容。比较一下前一个程序和下一个程序，后者的输出是不同的：

```py

def mystery(s, lst):

    s.upper()  # upper creates a new string

    lst.append(2)  # append changes the list

 s = 'a'

lst = [1]

mystery(s, lst)

print(s, lst)
```

#### 返回值

让我们回到卡牌游戏问题。我们的目标是定义一个函数，告诉我们列表中是否没有高牌。我们将这个函数命名为`no_high`。我们还没有编写`no_high`，但我们仍然可以指定我们希望实现的目标。我们想要的是：

```py
>>> no_high(['two', 'six'])

True

>>> no_high(['eight'])

True

>>> no_high(['two', 'jack', 'four'])

False

>>> no_high(['queen', 'king', 'three', 'queen'])

False
```

我们希望前两个调用返回`True`，因为在这些牌列表中没有高牌。而我们希望第三个和第四个调用返回`False`，因为这些牌列表中至少有一张高牌。

我们如何定义一个返回这些`True`和`False`值的函数？那是函数谜题的最后一块拼图。

为了从函数返回一个值，我们使用 Python 的 `return` 关键字。一旦遇到`return`，函数的执行就会终止，并将指定的值返回给调用者。

下面是我们如何编写`no_high`函数：

```py
>>> def no_high(lst):

...     if 'jack' in lst:

...         return False

...     if 'queen' in lst:

...         return False

...     if 'king' in lst:

...         return False

...     if 'ace' in lst:

...         return False

...     return True

...
```

我们首先检查列表中是否有任何`'jack'`卡牌。如果有，那么我们知道列表中包含一张或多张高牌，因此我们立即返回`False`。

如果我们还在这里，那就说明没有任何杰克牌。但可能会有其他高牌，所以我们需要检查它们。剩下的`if`语句分别检查皇后、国王和王牌，如果列表中包含任何一张，则返回`False`。

如果我们没有命中任何四个`return`语句，那么列表中就没有高牌。在这种情况下，我们返回`True`。

一个单独的`return`，没有给定值时，返回`None`。如果你写的函数不返回任何有用的东西，并且需要在到达代码底部之前终止函数，这是非常有用的。

如果在循环中遇到`return`，函数仍然会立即终止，不管它有多深嵌套。下面是一个示例，展示了`return`如何让我们跳出嵌套的循环：

```py
>>> def func():

...     for i in range(10):

...         for j in range(10):

...             print(i, j)

...             if j == 4:

...                 return

...

>>> func()

0 0

0 1

0 2

0 3

0 4
```

`return`就像是一个超级`break`！有些人不喜欢在循环中使用`return`，原因与不喜欢使用`break`相同：它可能会模糊循环的目的和逻辑。当方便时，我会在循环中使用`return`。与`break`可以出现在任何地方不同，`return`仅限于出现在函数内部，且与其他代码分开。如果我们保持函数小巧，那么在循环中使用`return`可以帮助我们编写清晰的代码，而不干扰周围的代码。

**概念检查**

以下版本的`no_high`是否正确？也就是说，如果列表中至少有一张高牌，它是否返回`True`，否则返回`False`？

```py
def no_high(lst):

    for card in lst:

 if card in ['jack', 'queen', 'king', 'ace']:

            return False

        else:

            return True
```

A. 对

B. 不对；例如，对于`['two', 'three']`，它返回了错误的值。

C. 不对；例如，对于`['jack']`，它返回了错误的值。

D. 不；例如，它返回了错误的值 `['jack', 'two']`

E. 不；例如，它返回了错误的值 `['two', 'jack']`

答案：E. `if-else`语句导致循环在第一次迭代时总是终止。如果第一张卡片是高牌，函数终止并返回`False`；如果第一张卡片不是高牌，函数终止并返回`True`。它不会查看其他卡片！这就是为什么它在`['two', 'jack']`上失败的原因：第一张卡片不是高牌，所以函数返回`True`。返回`True`意味着列表中没有高牌。但这错了：里面有一张杰克！函数做错了，它应该返回`False`。

### 函数文档

现在我们可以清楚地知道`no_high`函数的作用以及如何调用它。但几个月后，当我们不再立即记得旧代码的目的时会怎么样呢？如果我们积累了大量自己的函数，难以记住每个函数的作用，又该怎么办呢？

对于我们编写的每个函数，我们会添加文档说明，指定每个参数的含义以及函数返回的内容。这种文档称为*文档字符串*，即“documentation string”的缩写。文档字符串应从函数块的第一行开始编写。以下是`no_high`函数，这次带有文档说明：

```py
>>> def no_high(lst):

...     """

...     lst is a list of strings representing cards.

...

...     Return True if there are no high cards in lst, False otherwise.

...     """

...     if 'jack' in lst:

...         return False

...     if 'queen' in lst:

...         return False

...     if 'king' in lst:

 ...         return False

...     if 'ace' in lst:

...         return False

...     return True

...
```

文档字符串以三个双引号（`"""`）开头和结束。像单引号（`'`）或双引号（`"`）一样，三个双引号也可以用来开始和结束任何字符串。用三个引号创建的字符串称为*三引号字符串*。（三个单引号也可以使用，但 Python 的惯例是使用三个双引号。）它们的优点是，允许我们通过在每一行后按下 ENTER 键来添加多行文本；用`'`或`"`创建的字符串则不能像这样跨越多行。我们使用三引号字符串作为文档字符串，这样我们就可以包含任意多的行。

这里的文档字符串告诉我们`lst`是什么：它是一个表示卡片的字符串列表。它还告诉我们，函数返回一个`True`或`False`值，并说明每个返回值的含义。这些信息足以让任何人调用此函数，而不必查看其代码。只要有人知道一个函数的作用，他们就可以直接使用它。我们一直在使用 Python 函数，而从未查看过它们的代码。`print`是如何工作的？`input`是如何工作的？我们不知道！但这并不重要：我们知道这些函数的作用，因此可以专注于调用它们。

对于具有多个参数的函数，文档字符串应列出每个参数并给出预期的类型。以下是本章《可变参数》中的`remove_all`函数，并附上了合适的文档字符串：

```py
>>> def remove_all(lst, value):

...     """

...     lst is a list.

...     value is a value.

...

...     Remove all occurrences of value from lst.

...     """

...     while value in lst:

...         lst.remove(value)

...
```

请注意，这个文档字符串没有提到返回任何内容。这是因为这个函数不返回任何有用的东西！它从`lst`中移除内容，文档字符串就是这么说的。

### 解决问题

我们刚刚学习了定义和调用函数的基础知识。在本书接下来的部分，每当我们面临一个大问题时，我们将能够将问题的解决方案拆分为更小的任务，每个任务将由一个函数来解决。

让我们在卡牌游戏的解决方案中使用我们的`no_high`函数。代码见清单 6-1。

```py
❶ NUM_CARDS = 52

❷ def no_high(lst):

       """

       lst is a list of strings representing cards.

       Return True if there are no high cards in lst, False otherwise.

       """

       if 'jack' in lst:

           return False

       if 'queen' in lst:

           return False

       if 'king' in lst:

           return False

       if 'ace' in lst:

           return False

       return True

❸ deck = []

❹ for i in range(NUM_CARDS):

       deck.append(input())

   score_a = 0

   score_b = 0

   player = 'A'

❺ for i in range(NUM_CARDS):

       card = deck[i]

       points = 0

    ❻ remaining = NUM_CARDS - i - 1

    ❼ if card == 'jack' and remaining >= 1 and no_high(deck[i+1:i+2]):

           points = 1

       elif card == 'queen' and remaining >= 2 and no_high(deck[i+1:i+3]):

           points = 2

       elif card == 'king' and remaining >= 3 and no_high(deck[i+1:i+4]):

           points = 3

       elif card == 'ace' and remaining >= 4 and no_high(deck[i+1:i+5]):

           points = 4

    ❽ if points > 0:

           print(f'Player {player} scores {points} point(s).')

 ❾ if player == 'A':

           score_a = score_a + points

           player = 'B'

       else:

           score_b = score_b + points

           player = 'A'

   print(f'Player A: {score_a} point(s).')

   print(f'Player B: {score_b} point(s).')
```

*清单 6-1：解决卡牌游戏问题*

我引入了常量`NUM_CARDS`来表示`52`❶。我们将在代码中多次使用它，记住`NUM_CARDS`的含义比记住`52`的含义更容易。

接下来我们定义`no_high`函数，包括我们已深入讨论过的文档字符串❷。我们总是把函数放在程序的顶部。这样，函数可以被后续的代码调用。

程序的主要部分从创建一个列表开始，这个列表将包含牌堆中的卡片❸。然后我们从输入中读取卡片❹，并将每张卡片添加到牌堆中。你会注意到，卡片从未被字面上移除或取出（整个程序执行过程中，牌堆保持不变）。我们本可以那样做，但我选择了跟踪我们在牌堆中的位置，这样我们就知道下一张卡片会被移除。

还有三个其他关键变量我们需要维护：`score_a`，玩家 A 当前的总分；`score_b`，玩家 B 当前的总分；以及`player`，当前玩家的名字。

接下来的任务是查看牌堆中的每一张卡片，以便为玩家打分。一个普通的`for`循环可以让我们查看当前的卡片。但这还不够：如果当前卡片是高牌，那么我们还必须能够查看后面的卡片。为此，我们使用了一个范围`for`循环❺。

在每次循环迭代中，我们根据当前玩家从牌堆中拿到的卡片来确定该玩家获得的积分。每个得分规则都依赖于牌堆中剩余卡片的数量。`remaining`变量❻告诉我们剩余卡片的数量。当`i`为`0`时，剩余卡片的数量是`51`，因为我们刚刚拿了第一张卡片。当`i`为`1`时，剩余卡片的数量是`50`，因为我们刚刚拿了第二张卡片。一般来说，剩余卡片的数量可以通过总卡片数减去`i`再减去`1`来表示。

现在我们有了四个测试，每个测试对应一种得分方式❼。每个测试检查当前卡牌和剩余的卡牌数量。如果这两个条件都为`True`，则调用我们的`no_high`函数，并传入包含适当数量卡牌的卡组切片。例如，如果当前卡牌是`'jack'`，并且至少剩余`1`张卡牌，则我们将长度为`1`的列表传递给`no_high`❼。如果`no_high`返回`True`，则说明切片中没有高卡，当前玩家得分。`points`变量决定将要获得的得分；它在每次循环迭代时从`0`开始，并根据需要设置为`1`、`2`、`3`或`4`。

如果玩家得分❽，那么我们将输出一条消息，指明得分的玩家以及他们所获得的分数。

当前迭代剩下的任务就是将得分加到当前玩家的分数上，并轮到另一个玩家。我们通过`if-else`语句❾来完成这两个任务。（如果当前迭代中的`points`为`0`，则会向玩家的分数中添加一个无害的`0`，不需要专门测试和避免这种情况。）

最后的两个`print`语句输出每个玩家的总得分。

就这样：我们使用一个函数解决了这个问题，组织了我们的代码，使其更易于阅读。可以自由将我们的代码提交给裁判，你会看到所有测试用例都通过了。

### 问题 #15：动作人物模型

为了解决卡牌游戏问题，我们首先通过一个例子来理解，这个例子突出了函数可能有用的地方。现在，我们将使用函数解决另一个问题，但我们会通过更系统化的方法来发现所需的函数。

这是 Timus 的第`2144`号问题。这是书中唯一来自 Timus 裁判的题目。要找到这个问题，请访问*[`acm.timus.ru/`](https://acm.timus.ru/)*，点击**Problem set**，点击**Volume 12**，然后找到 2144 号问题（在裁判系统中叫做 Cleaning the Room）。

#### 挑战

Lena 有*n*个未开封的动作人物模型盒子。盒子不能打开（否则动作人物模型的价值会降低），因此盒子中的人物模型顺序不能更改。此外，盒子不能旋转（否则人物模型会朝向错误的方向）。

每个动作人物模型都由其高度来表示。例如，一个盒子可能有三个人物模型，分别是高度 4、5 和 7。从左到右排列。当我提到动作人物模型盒子时，我总是会从左到右列出人物模型的高度。

Lena 想要*整理这些盒子*，即将盒子排列成从左到右人物模型的高度逐渐增加或保持不变。

是否可以整理这些盒子取决于盒子中人物模型的高度。例如，如果第一个盒子的高度是 4、5 和 7，第二个盒子的高度是 1 和 2，那么她可以通过先放第二个盒子来整理这些盒子。但如果我们保持第一个盒子不变，第二个盒子的高度改为 6 和 8，那么就无法整理这两个盒子。

判断 Lena 是否可以整理这些盒子。

#### 输入

输入包括以下几行：

+   一行包含整数 *n*，即盒子的数量。*n* 的范围是 1 到 100。

+   *n* 行，每行对应一个盒子。每行的开头是整数 *k*，表示这个盒子中的人物模型数量。*k* 的范围在 1 到 100 之间。（因为 *k* 至少为 1，所以我们不需要担心空盒子。）紧接着 *k* 后面，是 *k* 个整数，表示这个盒子中人物模型的高度，从左到右排列。每个高度是一个介于 1 到 10,000 之间的整数。每对整数之间有一个空格。

#### 输出

如果 Lena 可以整理这些盒子，输出 `YES`；否则，输出 `NO`。

### 代表盒子

这个问题由几个较小的问题组成，我们可以通过编写函数来解决每个问题。首先，让我们看看如何在 Python 中表示这些盒子，然后我们将设计我们需要的函数。

在第五章中，当我们解决贝克奖金问题时，我们学到了列表可以将其他列表作为其值。这使得我们可以将列表嵌套在列表中。我们可以使用这种结构来表示人物模型的盒子。例如，这里有一个列表，表示两个盒子：

```py
>>> boxes = [[4, 5, 7], [1, 2]]
```

第一个盒子里有三个人物模型，第二个盒子里有两个。我们可以单独访问每个盒子：

```py
>>> boxes[0]

[4, 5, 7]

>>> boxes[1]

[1, 2]
```

我们将从输入中读取盒子的内容，并将这些信息放入一个嵌套列表，就像我所展示的那样。然后我们将使用这个嵌套列表来判断这些盒子是否能够整理。

### 自顶向下设计

我们将使用一种程序设计方法来解决这个问题，称为 *自顶向下设计*。自顶向下设计将一个大问题分解为多个较小的问题。这很有用，因为每个较小的问题都更容易解决。然后，我们可以将这些子问题的解决方案组合起来，解决原始问题。

#### 做自顶向下设计

这就是自顶向下设计的工作方式。我们从编写一个不完整的 Python 程序开始，它捕捉了问题解决方案中的主要任务。其中一些任务不需要太多代码，所以我们可以直接解决它们。其他任务则需要更多代码，我们会将每个任务转换成一个函数来调用。我们也可能通过编写一些代码 *并且* 调用函数来解决某些任务。不过，这些函数现在还不存在，我们必须编写它们！

要写一个所需的函数，我们对该函数的任务重复这个相同的过程。也就是说，我们首先写下该函数的任务。如果我们能直接为某个任务编写代码，那就直接编写；否则，我们调用另一个函数（我们稍后会编写）来处理该任务。

我们不断重复这个过程，直到没有更多的函数需要编写。到那时，我们就能得出问题的解决方案。

这就是所谓的自顶向下设计，因为我们从问题的最顶层或最高层开始，逐步向下，深入问题的核心，直到每个任务都被完全编写成代码。我们现在将使用这种方法来解决动作人物问题。

#### 顶层

在开始设计之前，我们专注于我们需要解决的主要任务。

我们肯定需要读取输入，因此这将是我们的第一个任务。

现在，假设我们已经读取了输入。我们应该做什么来确定箱子是否可以组织起来？一个重要的步骤是检查每个箱子，确保它的动作人物的身高是按顺序排列的。例如，假设我们有一个箱子`[18, 20, 4]`。这个箱子的身高乱序，意味着我们没有机会整理所有箱子。我们甚至不能整理这个箱子！

所以，这就是我们的第二个任务：确定每个箱子本身是否有序。如果任何一个箱子的动作人物顺序错乱，那么我们知道这些箱子无法组织。如果所有的箱子都没问题，那么我们还有更多的检查。

如果每个箱子本身都没有问题，接下来的问题是我们是否能够组织所有的箱子。我们在这里可以做出一个重要的观察：从现在开始，我们关心的唯一动作人物是每个箱子左右两侧的动作人物。箱子中间的动作人物不再重要。

这里有一个例子，我们有三个箱子：

```py
[[9, 13, 14, 17, 25],

 [32, 33, 34, 36],

 [1, 6]]
```

第一个箱子以身高 9 的动作人物开始，以身高 25 的动作人物结束。放在这个箱子左边的动作人物的身高必须都不超过 9；例如，我们可以将第三个箱子放在这个箱子左边。放在这个箱子右边的动作人物的身高必须都不低于 25；例如，我们可以将第二个箱子放在这个箱子右边。身高为 13、14 和 17 的动作人物没有任何影响；它们可以不存在。

那么，第三个任务就是：忽略所有动作人物，除了那些在箱子两端的动作人物。

在第三个任务之后，我们会得到如下的任务列表：

```py
[[9, 25],

 [32, 36],

 [1, 6]]
```

如果我们先对这些箱子进行排序，那么判断它们是否可以组织起来就容易多了，就像这样：

```py
[[1, 6],

 [9, 25],

 [32, 36]]
```

现在很容易看出一个箱子的邻近箱子是什么样子的。（我们在解决第五章的村庄邻里问题时也使用了类似的方法。）所以，我们的第四个任务是对箱子进行排序。

我们的第五个也是最后一个任务是判断这些排序后的盒子是否已经被整理好。只有当动作人物的高度从左到右是有序时，盒子才算整理好。高度为 1、6、9、25、32 和 36 的动作人物已经按正确的顺序排列，因此前面的盒子可以被整理好。但考虑下面这个例子：

```py
[[1, 6],

 [9, 50],

 [32, 36]]
```

这些盒子无法排序，因为第二个盒子里有个巨大的动作人物。第二个盒子占据了 9 到 50 的高度；第三个盒子不能放在第二个盒子的右边，因为它的高度太小。

我们现在已经完成了问题的设计，并决定了五个主要任务：

1.  读取输入。

1.  检查所有盒子是否正常。

1.  从每个盒子中获取一个新的盒子列表，只包含左右动作人物的高度。

1.  对这些新盒子进行排序。

1.  判断这些排序后的盒子是否已被整理好。

你可能会好奇，为什么我们有一个“读取输入”的任务，但没有“写入输出”的任务。对于这个问题，写输出仅仅是根据需要输出`YES`或`NO`；不会有太多复杂的操作。此外，我们会在知道答案的第一时间就输出`YES`或`NO`，所以输出会与其他任务交替进行。基于这些原因，我决定不把它作为一个主要任务。在进行自上而下的设计时，不用担心如果后来发现漏掉了任务，可以随时添加并继续设计。

这是我们如何在代码中捕捉所需任务的方式：

```py
❶ # Main Program

   # TODO: Read input

   # TODO: Check whether all boxes are OK

   # TODO: Obtain a new list of boxes with only left and right heights

   # TODO: Sort boxes

   # TODO: Determine whether boxes are organized
```

我将这个称为主程序❶。我们写的任何函数都应该包含在这个注释之前。

每个任务目前仅作为注释书写。`TODO`标记用来突出这些任务是我们需要从英语转换为 Python 的任务。每当我们完成一个任务时，我们会移除它的`TODO`。这样，我们就能追踪哪些任务已经完成，哪些还没完成。让我们开始吧！

#### 任务 1：读取输入

我们需要读取包含*n*（盒子数量）的那一行，然后读取盒子。读取整数是一行代码就能完成的任务，所以我们直接读取*n*。而读取盒子则是一个定义良好的任务，需要几行代码来解决，因此我们将通过一个函数来完成；我们称之为`read_boxes`。在主程序中，这就是我们当前的进度：

```py
   # Main Program

❶ # Read input

   n = int(input())

   boxes = read_boxes(n)

   # TODO: Check whether all boxes are OK

   # TODO: Obtain a new list of boxes with only left and right heights

   # TODO: Sort boxes

   # TODO: Determine whether boxes are organized
```

我已从注释❶中移除了`TODO`，因为从主程序的角度来看，我们已经解决了这个任务。当然，我们还需要编写`read_boxes`函数，接下来我们就来做这个。

`read_boxes`函数接受一个整数`n`作为参数，读取并返回`n`个盒子。以下是代码：

```py
def read_boxes(n):

    """

    n is the number of boxes to read.

    Read the boxes from the input, and return them as a

    list of boxes; each box is a list of action figure heights.

    """

    boxes = []

 ❶ for i in range(n):

        box = input().split()

     ❷ box.pop(0)

        for i in range(len(box)):

            box[i] = int(box[i])

        boxes.append(box)

    return boxes
```

我们需要读取`n`个盒子，因此我们循环`n`次❶。在每次循环中，我们读取当前行并将其拆分成单独的动作人物高度。行的开头是一个整数，表示这一行的高度数量，所以我们在继续之前会将该值从列表中移除（它在索引`0`）。然后，我们将每个高度转换为整数，并将当前盒子添加到盒子列表中。最后，我们返回盒子列表。

我们没有将`read_boxes`的任何部分推迟到一个尚未编写的函数中，所以我们完成了这个任务！我们会将这个函数与其他编写的函数一起放在`# 主程序`注释之前。

#### 任务 2：检查所有盒子是否合格

每个盒子是否从最短的动作人物到最高的动作人物依次排列？好问题，这不是我们能在一两行代码中回答的。我们依赖一个新的函数`all_boxes_ok`来告诉我们。如果该函数返回`False`，说明至少有一个盒子的高度有问题，我们无法整理这些盒子。在这种情况下，我们应该输出`NO`。如果`all_boxes_ok`返回`True`，那么我们应该继续执行剩余任务，判断这些盒子是否能够整理。让我们也将这个`if-else`逻辑添加到我们的程序中。以下是我们得到的：

```py
   # Main Program

   # Read input

   n = int(input())

   boxes = read_boxes(n)

   # Check whether all boxes are OK

❶ if not all_boxes_ok(boxes):

 print('NO')

   else:

       # TODO: Obtain a new list of boxes with only left and right heights

       # TODO: Sort boxes

       # TODO: Determine whether boxes are organized
```

现在我们需要编写我们调用的`all_boxes_ok`函数 ❶。我们可以检查每个盒子是否按顺序排列。如果没有，我们立即返回`False`。如果按顺序排列，我们检查下一个盒子。如果检查每个盒子且它们都按顺序排列，我们就返回`True`。

啊哈，所以我们需要能够检查一个单独的盒子！听起来像是另一个函数。我们叫它`box_ok`。

这是我们为`all_boxes_ok`写的：

```py
def all_boxes_ok(boxes):

    """

    boxes is a list of boxes; each box is a list of action figure heights.

    Return True if each box in boxes has its action figures in

    nondecreasing order of height, False otherwise.

    """

    for box in boxes:

        if not box_ok(box):

            return False

    return True
```

我在注释中使用了`nondecreasing`这个词，而不是`increasing`，因为动作人物的高度可以相等。例如，盒子`[4, 4, 4]`是完全可以的；如果说这个盒子是“递增”的，那就是不正确的。

我们将`all_boxes_ok`任务的一部分推到`box_ok`函数中，所以接下来我们来编写这个函数。开始吧：

```py
def box_ok(box):

    """

    box is the list of action figure heights in a given box.

    Return True if the heights in box are in nondecreasing order,

    False otherwise.

    """

    for i in range(len(box)):

        if box[i] > box[i + 1]:

            return False

    return True
```

如果某个高度大于它右边的高度，我们返回`False`，因为高度顺序不对。如果通过了`for`循环，那么就没有高度违规，返回`True`。

使用自顶向下设计的一个好处是，我们得到了一小块块的代码，以函数的形式封装起来，可以单独测试。例如，将`box_ok`的代码输入到 Python shell 中。然后我们可以测试它：

```py
>>> box_ok([4, 5, 6])
```

我们希望这里返回`True`，因为盒子的高度是从小到大排列的。我们当然不希望得到我们实际得到的结果：

```py
Traceback (most recent call last):

  File "<stdin>", line 1, in <module>

  File "<stdin>", line 9, in box_ok

IndexError: list index out of range
```

错误从来都不好玩，当我们需要翻遍页面的代码才能找到它们时，更加令人沮丧。但在这里，我们知道错误局限于这个小函数，因此我们找出它的工作量大大减少。问题在于，我们最终会将最右边的高度与右边的高度进行比较——当然，右边的高度是不存在的！所以我们需要提前一轮停止，比较倒数第二个高度和最后一个高度。以下是更新后的代码：

```py
def box_ok(box):

    """

    box is the list of action figure heights in a given box.

    Return True if the heights in box are in nondecreasing order,

    False otherwise.

    """

 ❶ for i in range(len(box) - 1):

        if box[i] > box[i + 1]:

            return False

    return True
```

唯一的变化是在对`range` ❶的调用。如果你测试这个版本的函数，你会看到它按要求工作。任务 2 完成！

#### 任务 3：获得只有左右高度的新盒子列表

现在我们已经掌握了自上而下的设计方法。在这个任务中，我们需要一种方法，将包含所有动作人物的盒子，转变为只包含最左侧和最右侧动作人物的盒子。我将把最左侧和最右侧的动作人物称为盒子的*端点*。

一种方法是创建一个新的仅包含端点的盒子列表，这也是我在这里要做的。你也可以考虑从原始盒子中实际删除高度，虽然这稍微复杂一些。

我已经为这个任务将函数命名为`boxes_endpoints`。这是程序的主要部分，已经更新为调用该函数：

```py
# Main Program

# Read input

n = int(input())

boxes = read_boxes(n)

# Check whether all boxes are OK

if not all_boxes_ok(boxes):

    print('NO')

else:

    # Obtain a new list of boxes with only left and right heights

 ❶ endpoints = boxes_endpoints(boxes)

    # TODO: Sort boxes

    # TODO: Determine whether boxes are organized
```

当我们用包含盒子的列表❶调用`boxes_endpoints`时，我们期望返回一个仅包含盒子端点的新列表。这是满足此描述的`boxes_endpoints`代码：

```py
def boxes_endpoints(boxes):

    """

    boxes is a list of boxes; each box is a list of action figure heights.

    Return a list, where each value is a list of two values:

    the heights of the leftmost and rightmost action figures in a box.

    """

 ❶ endpoints = []

    for box in boxes:

     ❷ endpoints.append([box[0], box[-1]])

    return endpoints
```

我们创建一个新的列表❶，用来存放每个盒子的端点。然后我们遍历盒子。对于每个盒子，我们使用索引来找到盒子中的最左侧和最右侧的高度，并将它们添加到我们的端点列表❷中。最后，我们返回端点列表。

等一下：如果一个盒子里只有一个动作人物会怎么样？我们的`boxes_endpoints`函数会怎么处理它？根据它的文档字符串，它会为任何有效的盒子返回一个包含两个值的列表。所以这里最好也发生这种情况，否则函数就没有做到它承诺的功能。我们来测试一下。将`boxes_endpoints`函数输入到 Python 终端，并尝试使用一个只包含一个动作人物的盒子的列表：

```py
>>> boxes_endpoints([[2]])

[[2, 2]]
```

成功了！最左侧的高度是`2`，最右侧的高度是`2`，所以我们得到了一个包含两个`2`的列表。我们的函数在这个案例中工作正常，因为当`box`只有一个值时，`box[0]`和`box[-1]`都指向相同的值。（不必担心空盒子的情况，问题描述禁止了空盒子。）

#### 任务 4：排序盒子

到了这个时候，我们有了一个端点列表——大概是这样的：

```py
>>> endpoints = [[9, 25], [32, 36], [1, 6]]

>>> endpoints

[[9, 25], [32, 36], [1, 6]]
```

我们想对它们进行排序。我们需要另一个函数吗？比如`sort_endpoints`函数？

这次不是！`sort`方法正是我们需要的：

```py
>>> endpoints.sort()

>>> endpoints

[[1, 6], [9, 25], [32, 36]]
```

当对包含两个值的列表调用`sort`时，它会使用第一个值进行排序。（如果有并列情况，它会进一步使用第二个值进行排序。）

我们可以立即通过调用`sort`来更新程序的主要部分，再解决一个`TODO`。这是更新后的代码：

```py
# Main Program

# Read input

n = int(input())

boxes = read_boxes(n)

# Check whether all boxes are OK

if not all_boxes_ok(boxes):

    print('NO')

else:

    # Obtain a new list of boxes with only left and right heights

    endpoints = boxes_endpoints(boxes)

    # Sort boxes

    endpoints.sort()

    # TODO: Determine whether boxes are organized
```

我们快完成了。只剩下一个`TODO`。

#### 任务 5：确定盒子是否有序

我们的最终任务是检查端点。它们可能是有序的，像这样：

```py
[[1, 6],

 [9, 25],

 [32, 36]]
```

或者它们也可能不是有序的，像这样：

```py
[[1, 6],

 [9, 50],

 [32, 36]]
```

在前一种情况下，我们应该打印`YES`；在后一种情况下，我们应该打印`NO`。我们需要一个函数来告诉我们端点是否有序。最后一次更新程序的主要部分，结果是这样的：

```py
# Main Program

# Read input

n = int(input())

boxes = read_boxes(n)

# Check whether all boxes are OK

if not all_boxes_ok(boxes):

    print('NO')

else:

    # Obtain a new list of boxes with only left and right heights

    endpoints = boxes_endpoints(boxes)

    # Sort boxes

    endpoints.sort()

    # Determine whether boxes are organized

 ❶ if all_endpoints_ok(endpoints):

        print('YES')

    else:

        print('NO')
```

我们和问题的完整解决方案之间只差一个 `all_endpoints_ok` 函数❶。它接收一个列表，每个值都是一个端点列表，如果端点顺序正确，返回 `True`，否则返回 `False`。

让我们通过一个示例来理解如何实现这个函数。以下是我们将使用的端点列表：

```py
[[1, 6],

 [9, 25],

 [32, 36]]
```

第一个盒子的右端点高度是 6。因此，第二个盒子的左端点高度必须至少是 6。如果不是，我们就返回 `False`，表示端点没有按照顺序排列。但在这里没问题，因为第二个盒子的左端点高度是 9。

现在我们使用第二个盒子的右端点 25 重复检查。第三个盒子的左端点是 32，所以没问题，因为 32 至少是 25。

一般来说，如果一个盒子的左端点小于前一个盒子的右端点，我们就返回 `False`。否则，如果所有检查都通过，我们返回 `True`。

以下是代码：

```py
def all_endpoints_ok(endpoints):

    """

    endpoints is a list, where each value is a list of two values:

    the heights of the leftmost and rightmost action figures in a box.

 ❶ Requires: endpoints is sorted by action figure heights.

    Return True if the endpoints came from boxes that can be

    put in order, False otherwise.

    """

 ❷ maximum = endpoints[0][1]

    for i in range(1, len(endpoints)):

        if endpoints[i][0] < maximum:

            return False

     ❸ maximum = endpoints[i][1]

    return True
```

我在文档字符串中添加了一些信息，提醒我们在调用此函数时需要注意的事项❶。特别是，我们必须记住在调用此函数之前，端点应该是排序好的。否则，函数可能会返回错误的值。

`endpoints` 的每个值是一个包含两个值的列表：索引 `0` 是最左边（最小）的高度，索引 `1` 是最右边（最大）的高度。代码使用 `maximum` 变量来追踪盒子的最大高度。在 `for` 循环之前，它引用第一个盒子的最大高度❷。`for` 循环比较下一个盒子的最小值与最大值。如果下一个盒子的最小值太小，我们返回 `False`，因为这两个盒子无法正确组织。在每次迭代的最后，我们更新 `maximum`，使其引用下一个盒子的最大值❸。

#### 将所有部分整合在一起

我们已经编写了所有任务的代码，包括作为设计一部分出现的函数，现在我们准备将它们整合成一个完整的解决方案。是否保留程序主部分的注释由你决定。我保留了它们，但实际上这可能会是过度文档化的做法，因为函数名称本身已经足够说明代码在做什么。完整代码请参见 清单 6-2。

```py
def read_boxes(n):

    """

    n is the number of boxes to read.

    Read the boxes from the input, and return them as a

    list of boxes; each box is a list of action figure heights.

    """

    boxes = []

    for i in range(n):

        box = input().split()

        box.pop(0)

        for i in range(len(box)):

            box[i] = int(box[i])

        boxes.append(box)

    return boxes

def box_ok(box):

    """

    box is the list of action figure heights in a given box.

    Return True if the heights in box are in nondecreasing order,

    False otherwise.

    """

    for i in range(len(box) - 1):

        if box[i] > box[i + 1]:

            return False

    return True

def all_boxes_ok(boxes):

    """

    boxes is a list of boxes; each box is a list of action figure heights.

    Return True if each box in boxes has its action figures in

    nondecreasing order of height, False otherwise.

    """

    for box in boxes:

        if not box_ok(box):

            return False

    return True

def boxes_endpoints(boxes):

    """

    boxes is a list of boxes; each box is a list of action figure heights.

    Return a list, where each value is a list of two values:

    the heights of the leftmost and rightmost action figures in a box.

    """

    endpoints = []

    for box in boxes:

        endpoints.append([box[0], box[-1]])

    return endpoints

def all_endpoints_ok(endpoints):

    """

    endpoints is a list, where each value is a list of two values:

    the heights of the leftmost and rightmost action figures in a box.

    Requires: endpoints is sorted by action figure heights.

    Return True if the endpoints came from boxes that can be

    put in order, False otherwise.

    """

    maximum = endpoints[0][1]

    for i in range(1, len(endpoints)):

        if endpoints[i][0] < maximum:

            return False

        maximum = endpoints[i][1]

    return True

# Main Program

# Read input

n = int(input())

boxes = read_boxes(n)

# Check whether all boxes are OK

if not all_boxes_ok(boxes):

    print('NO')

else:

    # Obtain a new list of boxes with only left and right heights

    endpoints = boxes_endpoints(boxes)

    # Sort boxes

    endpoints.sort()

    # Determine whether boxes are organized

    if all_endpoints_ok(endpoints):

        print('YES')

    else:

        print('NO')
```

*清单 6-2：解决动作人物问题*

这是我们目前为止在本书中编写的最大程序。但看看程序的主部分是多么简洁和最小化：它主要是函数调用，只有少量的 `if-else` 逻辑将它们连接起来。

我们这里只调用了每个函数一次。与我们四次调用的 `no_high` 卡牌游戏函数相比，即使一个函数只调用一次，它仍然能帮助代码更有组织且易于阅读。

是时候提交给 Timus 判定器了。你应该看到所有测试用例都通过了。

**概念检查**

在任务 2 中，我们编写了函数`box_ok`来判断单个盒子中的高度是否按顺序排列。它使用了一个`for`循环。以下是`box_ok`的`while`循环版本，它是否正确？

```py
def box_ok(box):

    """

    box is the list of action figure heights in a given box.

    Return True if the heights in box are in nondecreasing order,

    False otherwise.

    """

    ok = True

    i = 0

    while i < len(box) - 1 and ok:

        if box[i] > box[i + 1]:

            ok = False

        i = i + 1

    return ok
```

A. 是的

B. 否；它可能导致`IndexError`错误

C. 否；它不会引发任何错误，但可能返回错误的值

答案：A。这与我们之前使用`for`循环的范围版本等价。`ok`变量初始为`True`，意味着我们已经检查过的所有高度都是合格的（因为我们还没有检查过任何高度！）。`while`循环会继续执行，只要

因为有更多的盒子需要检查，并且没有高度违规的情况。如果一个动作人物的顺序不正确，`ok`将被设置为`False`，这会终止循环。如果所有动作人物都按顺序排列，那么`ok`的值将从`True`保持到`False`。因此，当我们在函数的底部执行`return ok`时，如果所有动作人物都按顺序排列，我们将返回`True`，否则返回`False`。

### 总结

在这一章中，我们学习了函数。函数是一个自包含的代码块，解决了一个较大问题的一个小部分。我们学会了如何将信息传递给函数（通过参数）并获取返回值（通过返回值）。

为了确定首先要编写哪些函数，我们可以使用自顶向下设计方法。自顶向下设计帮助我们将一个大问题的解决方案拆解为多个较小的任务；对于每个任务，如果可以直接解决，就直接解决；如果无法直接解决，就为其编写函数。如果某个任务过于繁琐，我们可以进一步对其进行自顶向下设计。

在下一章中，我们将学习如何使用我们选择的文件，而不是使用标准输入和标准输出。随着我们不断拓展所知的边界，我们将在下一章及本书的其余部分找到很多函数的应用。通过一些以下的练习来增加你使用函数的信心。

### 章节练习

这里有一些练习供你尝试。对于每个练习，使用自顶向下设计方法来识别一个或多个函数，以帮助你组织代码。每个函数中都要包含文档字符串！

1.  DMOJ 问题`ccc13s1`，从 1987 到 2013

1.  DMOJ 问题`ccc18j3`，我们到了吗？

1.  DMOJ 问题`ecoo12r1p2`，解码 DNA

1.  DMOJ 问题`crci07p1`，Platforme

1.  DMOJ 问题`coci13c2p2`，Misa

1.  回顾一下第五章中的一些练习，并通过使用函数来改进你的解法。我特别建议你回顾一下 DMOJ 问题`coci18c2p1`（Preokret）和 DMOJ 问题`ccc00s2`（Babbling Brooks）。

### 备注

纸牌游戏最初来源于 1999 年加拿大计算机竞赛。动作人物最初来源于 2019 年乌拉尔学校编程竞赛。

许多现代编程语言，包括 Python，支持两种不同的编程范式。一种是基于函数的；这就是我们在本章中学习的内容。另一种是基于*对象*的，导致了一种被称为*面向对象编程（OOP）*的范式。OOP 涉及定义新类型并为这些类型编写方法。我们在全书中使用 Python 类型（如整数和字符串），但不会进一步讨论 OOP。关于 OOP 的入门以及 OOP 实践案例，我推荐由 Eric Matthes 编写的《*Python Crash Course*》第二版（No Starch Press，2019）。
