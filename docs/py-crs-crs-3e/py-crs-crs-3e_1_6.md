
## 第六章：字典

![](img/chapterart.png)

在本章中，你将学习如何使用 Python 的字典，它们允许你将相关信息连接在一起。你将学习如何访问存储在字典中的信息，并如何修改这些信息。由于字典可以存储几乎无限量的信息，我将向你展示如何遍历字典中的数据。此外，你将学习如何将字典嵌套在列表中，将列表嵌套在字典中，甚至将字典嵌套在其他字典中。

理解字典可以让你更准确地建模各种现实世界中的对象。你可以创建一个字典来表示一个人，并存储关于这个人的所有信息。你可以存储他们的名字、年龄、地点、职业以及任何你能描述的关于这个人的其他方面。你将能够存储任何两种可以配对的信息，比如单词及其意思、人的名字和他们的最爱数字、山脉及其海拔等等。

## 一个简单的字典

考虑一个包含外星人的游戏，外星人可以具有不同的颜色和点数。这个简单的字典存储了一个特定外星人的信息：

**alien.py**

```py
alien_0 = {'color': 'green', 'points': 5}

print(alien_0['color'])
print(alien_0['points'])
```

字典`alien_0`存储了外星人的颜色和点数。最后两行访问并显示了这些信息，如下所示：

```py
green
5
```

与大多数新的编程概念一样，使用字典需要练习。一旦你使用字典一段时间后，你会发现它们在建模现实世界情况方面是多么高效。

## 使用字典

在 Python 中，*字典*是一种由*键-值对*组成的集合。每个*键*与一个值相连，你可以使用键来访问与该键关联的值。一个键的值可以是一个数字、一个字符串、一个列表，甚至是另一个字典。实际上，你可以使用 Python 中可以创建的任何对象作为字典中的值。

在 Python 中，字典是用大括号`{}`包裹的，里面包含一系列键值对，如之前的示例所示：

```py
alien_0 = {'color': 'green', 'points': 5}
```

*键值对*是一组相互关联的值。当你提供一个键时，Python 会返回与该键相关联的值。每个键通过冒号与其值连接，单独的键值对通过逗号分隔。你可以在字典中存储任意数量的键值对。

最简单的字典只有一个键值对，如下所示，这是修改后的`alien_0`字典版本：

```py
alien_0 = {'color': 'green'}
```

这个字典存储了关于`alien_0`的一条信息：外星人的颜色。字符串`'color'`是字典中的一个键，而与之关联的值是`'green'`。

### 访问字典中的值

要获取与键相关联的值，给出字典的名称，然后将键放在一对方括号内，如下所示：

**alien.py**

```py
alien_0 = {'color': 'green'}
print(alien_0['color'])
```

这将返回字典`alien_0`中与键`'color'`关联的值：

```py
green
```

你可以在字典中拥有无限数量的键值对。例如，这里是原始的`alien_0`字典，包含两个键值对：

```py
alien_0 = {'color': 'green', 'points': 5}
```

现在你可以访问`alien_0`的颜色或积分值。如果玩家击败了这个外星人，你可以使用如下代码查找他们应该获得多少积分：

```py
alien_0 = {'color': 'green', 'points': 5}

new_points = alien_0['points']
print(f"You just earned {new_points} points!")
```

一旦字典被定义，我们就从字典中提取与键`'points'`相关联的值。然后将这个值赋给变量`new_points`。最后一行打印出玩家刚刚获得的积分：

```py
You just earned 5 points!
```

如果你每次击败外星人时都运行这段代码，外星人的积分值将会被检索出来。

### 添加新键值对

字典是动态结构，你可以随时向字典添加新的键值对。要添加新的键值对，你只需给出字典的名称，后跟新键的方括号，并指定新值。

让我们向`alien_0`字典添加两个新的信息：外星人的*x*坐标和*y*坐标，这将帮助我们在屏幕上的特定位置显示外星人。我们将外星人放置在屏幕的左边缘，距离顶部 25 像素。由于屏幕坐标通常从屏幕的左上角开始，我们通过将*x*坐标设置为 0 来把外星人放在屏幕的左边缘，并通过将*y*坐标设置为正 25 来把它放置在距离顶部 25 像素的位置，如下所示：

**alien.py**

```py
alien_0 = {'color': 'green', 'points': 5}
print(alien_0)

alien_0['x_position'] = 0
alien_0['y_position'] = 25
print(alien_0)
```

我们首先定义与之前相同的字典，然后打印这个字典，展示其信息快照。接下来，我们向字典中添加一个新的键值对：键`'x_position'`和值`0`。我们对键`'y_position'`做同样的操作。当我们打印修改后的字典时，我们看到这两个新增的键值对：

```py
{'color': 'green', 'points': 5}
{'color': 'green', 'points': 5, 'x_position': 0, 'y_position': 25}
```

字典的最终版本包含四个键值对。原来的两个指定了颜色和得分值，再加上两个指定了外星人位置的键值对。

字典会保留定义时的顺序。当你打印字典或遍历其元素时，你会看到元素按添加到字典的顺序出现。

### 从一个空字典开始

有时，开始时使用一个空字典，然后再逐个添加新的项是方便的，甚至是必要的。要开始填充一个空字典，定义一个空的大括号字典，然后将每个键值对单独添加在每一行。例如，以下是如何使用这种方法构建`alien_0`字典：

**alien.py**

```py
alien_0 = {}

alien_0['color'] = 'green'
alien_0['points'] = 5

print(alien_0)
```

我们首先定义一个空的`alien_0`字典，然后向其中添加颜色和得分值。结果是我们在之前示例中使用的字典：

```py
{'color': 'green', 'points': 5}
```

通常，在存储用户提供的数据时，或者在编写生成大量键值对的代码时，你会使用空字典。

### 修改字典中的值

要修改字典中的值，给出字典的名称并用方括号括住键，然后指定你希望与该键关联的新值。例如，考虑一个外星人，在游戏进程中从绿色变为黄色：

**alien.py**

```py
alien_0 = {'color': 'green'}
print(f"The alien is {alien_0['color']}.")

alien_0['color'] = 'yellow'
print(f"The alien is now {alien_0['color']}.")
```

我们首先定义一个仅包含外星人颜色的`alien_0`字典；然后我们将与`'color'`键关联的值更改为`'yellow'`。输出显示外星人确实从绿色变成了黄色：

```py
The alien is green.
The alien is now yellow.
```

对于一个更有趣的例子，让我们跟踪一个可以以不同速度移动的外星人。我们将存储一个表示外星人当前速度的值，然后用它来决定外星人应该向右移动多远：

```py
alien_0 = {'x_position': 0, 'y_position': 25, 'speed': 'medium'}
print(f"Original position: {alien_0['x_position']}")

# Move the alien to the right.
# Determine how far to move the alien based on its current speed.
❶ if alien_0['speed'] == 'slow':
    x_increment = 1
elif alien_0['speed'] == 'medium':
    x_increment = 2
else:
    # This must be a fast alien.
    x_increment = 3

# The new position is the old position plus the increment.
❷ alien_0['x_position'] = alien_0['x_position'] + x_increment

print(f"New position: {alien_0['x_position']}")
```

我们首先定义一个外星人，初始的*x*位置和*y*位置为`'medium'`速度。为了简化，我们省略了颜色和得分值，但如果你包括这些键值对，示例仍然适用。我们还打印了`x_position`的原始值，看看外星人向右移动了多少。

一个`if`-`elif`-`else`链决定外星人应该向右移动多远，并将这个值赋给变量`x_increment` ❶。如果外星人的速度是`'slow'`，它向右移动一个单位；如果速度是`'medium'`，它向右移动两个单位；如果是`'fast'`，它向右移动三个单位。计算出增量后，它会加到`x_position` ❷的值上，并将结果存储在字典的`x_position`中。

由于这是一个中等速度的外星人，它的位置向右移动了两个单位：

```py
Original x-position: 0
New x-position: 2
```

这个技巧非常酷：通过改变外星人字典中的一个值，你可以改变外星人的整体行为。例如，要将这个中等速度的外星人变成一个快速的外星人，你需要添加这一行：

```py
alien_0['speed'] = 'fast'
```

`if`-`elif`-`else` 代码块将在下一次代码运行时将更大的值分配给 `x_increment`。

### 删除键值对

当你不再需要字典中存储的信息时，可以使用 `del` 语句完全删除一个键值对。`del` 只需要字典的名称和你想要删除的键。

例如，假设我们从 `alien_0` 字典中删除键 `'points'`，以及它的值：

**alien.py**

```py
alien_0 = {'color': 'green', 'points': 5}
print(alien_0)

❶ del alien_0['points']
print(alien_0)
```

`del` 语句❶告诉 Python 删除字典 `alien_0` 中的键 `'points'`，并移除与该键关联的值。输出显示，键 `'points'` 和其值 `5` 被从字典中删除，但字典中的其他部分没有受到影响：

```py
{'color': 'green', 'points': 5}
{'color': 'green'}
```

### 相似对象的字典

前一个示例涉及存储关于一个对象（游戏中的外星人）的不同信息。你还可以使用字典存储关于许多对象的一种信息。例如，假设你想对一些人进行调查，询问他们最喜欢的编程语言是什么。字典非常适合存储这种简单调查的结果，如下所示：

**favorite_languages.py**

```py
favorite_languages = {
    'jen': 'python',
    'sarah': 'c',
    'edward': 'rust',
    'phil': 'python',
    }
```

如你所见，我们将一个较大的字典分成了几行。每个键都是参与调查的人的名字，每个值是他们选择的语言。当你知道定义字典时需要多行时，按 ENTER 键后，输入一个新的左大括号。接着将下一行缩进一个级别（四个空格），然后写入第一个键值对，后面跟着一个逗号。从此以后，每次按 ENTER 键时，你的文本编辑器应自动缩进所有后续的键值对，以匹配第一个键值对。

一旦你完成了字典的定义，在最后一个键值对后添加一个闭括号，并在新的一行上进行缩进，使其与字典中的键对齐。将逗号添加到最后一个键值对后也是一个好习惯，这样你就可以在下一行准备添加新的键值对。

为了使用这个字典，给定一个参与调查的人的名字，你可以轻松查找他们最喜欢的编程语言：

**favorite_languages.py**

```py
favorite_languages = {
 'jen': 'python',
 'sarah': 'c',
 'edward': 'rust',
 'phil': 'python',
 }

❶ language = favorite_languages['sarah'].title()
print(f"Sarah's favorite language is {language}.")
```

要查看 Sarah 选择了哪种语言，我们请求该值：

```py
favorite_languages['sarah']
```

我们使用这种语法从字典❶中提取 Sarah 最喜欢的编程语言，并将其赋值给变量 `language`。创建一个新的变量使得 `print()` 调用更加简洁。输出显示了 Sarah 最喜欢的编程语言：

```py
Sarah's favorite language is C.
```

你可以使用相同的语法与字典中的任何单个对象。

### 使用 get() 访问值

使用方括号中的键从字典中检索你感兴趣的值可能会导致一个潜在的问题：如果你请求的键不存在，你将得到一个错误。

让我们看看当你请求一个没有设置点数值的外星人的点数值时会发生什么：

**alien_no_points.py**

```py
alien_0 = {'color': 'green', 'speed': 'slow'}
print(alien_0['points'])
```

这会导致一个追溯错误，显示一个 `KeyError`：

```py
Traceback (most recent call last):
  File "alien_no_points.py", line 2, in <module>
    print(alien_0['points'])
          ~~~~~~~^^^^^^^^^^
KeyError: 'points'
```

你将在第十章中了解如何处理类似的错误。针对字典，你可以使用`get()`方法来设置一个默认值，当请求的键不存在时，返回该默认值。

`get()`方法需要一个键作为第一个参数。作为第二个可选参数，你可以传递一个当键不存在时返回的默认值：

```py
alien_0 = {'color': 'green', 'speed': 'slow'}

point_value = alien_0.get('points', 'No point value assigned.')
print(point_value)
```

如果字典中存在键`'points'`，你将得到对应的值。如果不存在，则会返回默认值。在这种情况下，`points`不存在，我们得到的是一个干净的消息，而不是错误：

```py
No point value assigned.
```

如果你请求的键可能不存在，考虑使用`get()`方法，而不是方括号表示法。

## 遍历字典

一个 Python 字典可以包含少量的键值对，也可以包含数百万个键值对。由于字典可以存储大量数据，Python 允许你遍历字典。字典可以用多种方式存储信息，因此也有多种方式可以遍历它们。你可以遍历字典的所有键值对、遍历它的键，或者遍历它的值。

### 遍历所有键值对

在我们探索不同的遍历方法之前，让我们考虑一个新的字典，用来存储网站上某个用户的信息。以下字典将存储一个人的用户名、名字和姓氏：

**user.py**

```py
user_0 = {
    'username': 'efermi',
    'first': 'enrico',
    'last': 'fermi',
    }
```

基于你在本章中学到的知识，你可以访问`user_0`的任何信息。但是，如果你想查看该用户字典中存储的所有内容怎么办？你可以使用`for`循环遍历字典：

```py
user_0 = {
 'username': 'efermi',
 'first': 'enrico',
 'last': 'fermi',
 }

for key, value in user_0.items():
    print(f"\nKey: {key}")
    print(f"Value: {value}")
```

要为字典编写`for`循环，你需要为每一对键值对创建两个变量名，用于存储键和值。你可以为这两个变量选择任何你喜欢的名字。如果你使用缩写作为变量名，代码依然能够正常工作，例如：

```py
for k, v in user_0.items()
```

`for`语句的第二部分包括字典的名称，后跟`items()`方法，该方法返回键值对的序列。然后，`for`循环将这些对分配给提供的两个变量。在前面的示例中，我们使用这些变量打印每个`key`，后跟相应的`value`。第一个`print()`调用中的`"\n"`确保在输出的每个键值对之前插入一个空行：

```py
Key: username
Value: efermi

Key: first
Value: enrico

Key: last
Value: fermi
```

遍历所有键值对在字典中尤其有效，比如第 96 页的*favorite_languages.py*示例，它存储了许多不同键的相同类型的信息。如果你遍历`favorite_languages`字典，你将得到字典中每个人的名字和他们最喜欢的编程语言。因为键总是指向一个人的名字，值总是指向一种语言，所以我们在循环中会使用变量`name`和`language`，而不是`key`和`value`。这样可以让你更容易理解循环内发生的事情：

**favorite_languages.py**

```py
favorite_languages = {
 'jen': 'python',
 'sarah': 'c',
 'edward': 'rust',
 'phil': 'python',
 }

for name, language in favorite_languages.items():
    print(f"{name.title()}'s favorite language is {language.title()}.")
```

这段代码告诉 Python 遍历字典中的每个键值对。它在处理每个键值对时，将键分配给变量`name`，将值分配给变量`language`。这些描述性的变量名使得你更容易理解`print()`语句在做什么。

现在，只需几行代码，我们就可以显示民意调查的所有信息：

```py
Jen's favorite language is Python.
Sarah's favorite language is C.
Edward's favorite language is Rust.
Phil's favorite language is Python.
```

如果我们的字典存储了对一千人甚至一百万人的调查结果，这种类型的循环同样有效。

### 遍历字典中的所有键

当你不需要处理字典中的所有值时，`keys()`方法非常有用。让我们遍历`favorite_languages`字典，并打印出所有参与调查的人的名字：

```py
favorite_languages = {
 'jen': 'python',
 'sarah': 'c',
 'edward': 'rust',
 'phil': 'python',
 }

for name in favorite_languages.keys():
    print(name.title())
```

这个`for`循环告诉 Python 从字典`favorite_languages`中提取所有的键，并一次性将它们分配给变量`name`。输出结果显示了所有参与调查的人的名字：

```py
Jen
Sarah
Edward
Phil
```

遍历键实际上是默认的行为，因此，如果你写成：

```py
for name in favorite_languages:
```

而不是：

```py
for name in favorite_languages.keys():
```

如果你觉得这样能让代码更易读，可以显式使用`keys()`方法，或者如果你愿意，也可以省略它。

你可以在循环内通过当前的键来访问任何你关心的键值对的值。让我们打印一些朋友选择的编程语言的消息。我们将像之前那样遍历字典中的名字，但当名字与我们的朋友之一匹配时，我们将显示关于他们最喜欢的语言的消息：

```py
favorite_languages = {
    *--snip--*
 }

friends = ['phil', 'sarah']
for name in favorite_languages.keys():
    print(f"Hi {name.title()}.")

❶     if name in friends:
❷         language = favorite_languages[name].title()
        print(f"\t{name.title()}, I see you love {language}!")
```

首先，我们列出一些朋友的名字，准备给他们打印一条消息。在循环中，我们打印每个人的名字。然后，我们检查当前正在处理的`name`是否在`friends`列表中❶。如果在，我们使用字典的名字和当前`name`的值作为键来确定这个人最喜欢的语言❷。接着，我们打印一条特别的问候语，包含他们选择的编程语言。

每个人的名字都会被打印出来，但我们的朋友们会收到一条特别的消息：

```py
Hi Jen.
Hi Sarah.
    Sarah, I see you love C!
Hi Edward.
Hi Phil.
    Phil, I see you love Python!
```

你还可以使用`keys()`方法来检查某个人是否参与了调查。这次，让我们来看看 Erin 是否参与了调查：

```py
favorite_languages = {
 *--snip--*
 }

if 'erin' not in favorite_languages.keys():
    print("Erin, please take our poll!")
```

`keys()` 方法不仅用于循环：它实际上返回所有键的序列，而 `if` 语句仅检查 `'erin'` 是否在这个序列中。因为她不在，所以打印出一条信息邀请她参与调查：

```py
Erin, please take our poll!
```

### 按特定顺序遍历字典的键

遍历字典时，返回的项与它们插入字典的顺序相同。不过，有时你可能希望以不同的顺序遍历字典。

一种方法是在 `for` 循环中按返回的顺序对键进行排序。你可以使用 `sorted()` 函数来按顺序获取键的副本：

```py
favorite_languages = {
 'jen': 'python',
 'sarah': 'c',
 'edward': 'rust',
 'phil': 'python',
 }

for name in sorted(favorite_languages.keys()):
    print(f"{name.title()}, thank you for taking the poll.")
```

这个 `for` 语句与其他 `for` 语句类似，唯一的区别是我们在 `dictionary.keys()` 方法周围包裹了 `sorted()` 函数。这告诉 Python 获取字典中的所有键，并在开始循环之前将其排序。输出结果显示所有参与调查的人，名字按顺序排列：

```py
Edward, thank you for taking the poll.
Jen, thank you for taking the poll.
Phil, thank you for taking the poll.
Sarah, thank you for taking the poll.
```

### 遍历字典中的所有值

如果你主要对字典中包含的值感兴趣，可以使用 `values()` 方法返回一个值的序列，而不包含任何键。例如，假设我们只想要一个所有语言的列表，而不包含选择每种语言的人的名字：

```py
favorite_languages = {
 'jen': 'python',
 'sarah': 'c',
 'edward': 'rust',
 'phil': 'python',
 }

print("The following languages have been mentioned:")
for language in favorite_languages.values():
    print(language.title())
```

这里的 `for` 语句从字典中提取每个值，并将其分配给变量 `language`。当这些值被打印时，我们得到一个所有选择语言的列表：

```py
The following languages have been mentioned:
Python
C
Rust
Python
```

这种方法从字典中提取所有值，而不检查是否有重复项。对于较少的值数量，这可能没问题，但如果是调查中有大量回应者时，结果会是一个非常重复的列表。为了查看每种语言而不重复，我们可以使用集合。*集合* 是一个其中每个项必须唯一的集合：

```py
favorite_languages = {
 *--snip--*
 }

print("The following languages have been mentioned:")
for language in set(favorite_languages.values()):
 print(language.title())
```

当你将 `set()` 包裹在包含重复项的值集合周围时，Python 会识别集合中唯一的项，并根据这些项构建一个集合。这里我们使用 `set()` 从 `favorite_languages.values()` 中提取唯一语言。

结果是一个没有重复项的语言列表，这些语言是参与调查的人提到的：

```py
The following languages have been mentioned:
Python
C
Rust
```

随着你继续学习 Python，你会经常发现语言内置的某个特性可以帮助你精确地处理数据。

## 嵌套

有时候你可能需要将多个字典存储在一个列表中，或者将一个列表作为字典的值。这被称为 *嵌套*。你可以在列表中嵌套字典，在字典中嵌套项的列表，甚至将字典嵌套在另一个字典中。嵌套是一个强大的特性，接下来的示例将演示这一点。

### 字典的列表

`alien_0`字典包含有关一个外星人的各种信息，但它没有空间存储第二个外星人的信息，更不用说一整屏外星人了。你如何管理一支外星人舰队呢？一种方法是创建一个外星人列表，其中每个外星人都是一个包含该外星人信息的字典。例如，以下代码构建了一个包含三个外星人的列表：

**aliens.py**

```py
alien_0 = {'color': 'green', 'points': 5}
alien_1 = {'color': 'yellow', 'points': 10}
alien_2 = {'color': 'red', 'points': 15}

❶ aliens = [alien_0, alien_1, alien_2]

for alien in aliens:
    print(alien)
```

我们首先创建三个字典，每个字典代表一个不同的外星人。我们将这些字典存储在一个名为`aliens`的列表中❶。最后，我们遍历列表并打印出每个外星人：

```py
{'color': 'green', 'points': 5}
{'color': 'yellow', 'points': 10}
{'color': 'red', 'points': 15}
```

一个更现实的例子可能包含超过三个外星人，并且代码会自动生成每个外星人。在下面的示例中，我们使用`range()`来创建一个 30 个外星人的舰队：

```py
# Make an empty list for storing aliens.
aliens = []

# Make 30 green aliens.
❶ for alien_number in range(30):
❷     new_alien = {'color': 'green', 'points': 5, 'speed': 'slow'}
❸     aliens.append(new_alien)

# Show the first 5 aliens.
❹ for alien in aliens[:5]:
    print(alien)
print("...")

# Show how many aliens have been created.
print(f"Total number of aliens: {len(aliens)}")
```

这个示例从一个空的列表开始，用来保存将要创建的所有外星人。`range()`函数❶返回一系列数字，它只是告诉 Python 我们希望循环重复多少次。每次循环运行时，我们创建一个新的外星人❷，然后将每个新的外星人追加到列表`aliens`中❸。我们使用切片打印前五个外星人❹，最后，我们打印列表的长度，以证明我们已经生成了完整的 30 个外星人的舰队：

```py
{'color': 'green', 'points': 5, 'speed': 'slow'}
{'color': 'green', 'points': 5, 'speed': 'slow'}
{'color': 'green', 'points': 5, 'speed': 'slow'}
{'color': 'green', 'points': 5, 'speed': 'slow'}
{'color': 'green', 'points': 5, 'speed': 'slow'}
...

Total number of aliens: 30
```

这些外星人都有相同的特征，但 Python 将每个外星人视为一个独立的对象，这使得我们可以单独修改每个外星人。

你如何处理这样一组外星人呢？假设游戏的某个环节中，外星人随着游戏的进展改变颜色并且移动得更快。当是时候改变颜色时，我们可以使用`for`循环和`if`语句来改变外星人的颜色。例如，为了将前三个外星人改为黄色的、中等速度的外星人，每个外星人值 10 分，我们可以这样做：

```py
# Make an empty list for storing aliens.
aliens = []

# Make 30 green aliens.
for alien_number in range (30):
 new_alien = {'color': 'green', 'points': 5, 'speed': 'slow'}
 aliens.append(new_alien)

for alien in aliens[:3]:
    if alien['color'] == 'green':
        alien['color'] = 'yellow'
        alien['speed'] = 'medium'
        alien['points'] = 10

# Show the first 5 aliens.
for alien in aliens[:5]:
 print(alien)
print("...")
```

因为我们想要修改前三个外星人，所以我们遍历一个只包含前三个外星人的切片。现在所有外星人都是绿色的，但并不总是这样，因此我们写一个`if`语句来确保只修改绿色外星人。如果外星人是绿色的，我们将其颜色改为`'yellow'`，速度改为`'medium'`，分值改为`10`，如下所示的输出：

```py
{'color': 'yellow', 'points': 10, 'speed': 'medium'}
{'color': 'yellow', 'points': 10, 'speed': 'medium'}
{'color': 'yellow', 'points': 10, 'speed': 'medium'}
{'color': 'green', 'points': 5, 'speed': 'slow'}
{'color': 'green', 'points': 5, 'speed': 'slow'}
...
```

你可以通过添加一个`elif`块来扩展这个循环，将黄色的外星人变成红色的、快速移动的外星人，每个外星人的分数为 15 分。省略完整程序的情况下，这个循环看起来像这样：

```py
for alien in aliens[0:3]:
 if alien['color'] == 'green':
 alien['color'] = 'yellow'
 alien['speed'] = 'medium'
 alien['points'] = 10
    elif alien['color'] == 'yellow':
        alien['color'] = 'red'
        alien['speed'] = 'fast'
        alien['points'] = 15
```

当每个字典包含关于一个对象的多种信息时，将多个字典存储在列表中是很常见的做法。例如，你可以为网站上的每个用户创建一个字典，就像我们在第 99 页的*user.py*中所做的那样，并将这些字典存储在一个名为`users`的列表中。列表中的所有字典应该具有相同的结构，这样你就可以遍历列表，并以相同的方式处理每个字典对象。

### 字典中的列表

有时，将列表放入字典中比将字典放入列表中更为有用。例如，考虑一下如何描述一个人正在点的披萨。如果你只使用列表，实际上你只能存储披萨的配料列表。使用字典时，配料列表可以仅仅是你描述的披萨的一个方面。

在以下示例中，每个披萨都存储了两种信息：一种是饼皮类型，另一种是配料列表。配料列表是与键 `'toppings'` 关联的值。为了使用列表中的项目，我们只需给出字典的名称和键 `'toppings'`，就像访问字典中的任何值一样。返回的不是单个值，而是一个配料列表：

**pizza.py**

```py
# Store information about a pizza being ordered.
pizza = {
    'crust': 'thick',
    'toppings': ['mushrooms', 'extra cheese'],
    }

# Summarize the order.
❶ print(f"You ordered a {pizza['crust']}-crust pizza "
    "with the following toppings:")

❷ for topping in pizza['toppings']:
    print(f"\t{topping}")
```

我们从一个字典开始，字典中包含已点披萨的信息。字典中的一个键是 `'crust'`，其对应的值是字符串 `'thick'`。下一个键 `'toppings'` 的值是一个存储所有请求配料的列表。在制作披萨之前，我们先总结一下订单❶。当你需要在 `print()` 调用中拆分一行时，选择一个适当的位置拆分打印的行，并以引号结束该行。接下来缩进新的一行，添加开引号，并继续字符串。Python 会自动将括号内的所有字符串组合在一起。为了打印配料，我们写一个 `for` 循环❷。要访问配料列表，我们使用键 `'toppings'`，然后 Python 会从字典中获取配料列表。

以下输出总结了我们计划制作的披萨：

```py
You ordered a thick-crust pizza with the following toppings:
    mushrooms
    extra cheese
```

你可以随时将列表嵌套在字典中，只要你希望将多个值与字典中的单个键相关联。在之前的最喜欢编程语言的示例中，如果我们将每个人的回答存储在一个列表中，那么人们可以选择多个最喜欢的语言。当我们遍历字典时，与每个人相关联的值将是一个语言列表，而不是单一的语言。在字典的 `for` 循环内部，我们使用另一个 `for` 循环来遍历与每个人相关联的语言列表：

**favorite_languages.py**

```py
favorite_languages = {
 'jen': ['python', 'rust'],
 'sarah': ['c'],
    'edward': ['rust', 'go'],
    'phil': ['python', 'haskell'],
    }

❶ for name, languages in favorite_languages.items():
    print(f"\n{name.title()}'s favorite languages are:")
❷     for language in languages:
        print(f"\t{language.title()}")
```

现在，`favorite_languages` 中每个名字所对应的值都是一个列表。注意，有些人只有一个最喜欢的语言，而有些人有多个最喜欢的语言。当我们遍历字典❶时，我们使用变量名 `languages` 来存储字典中的每个值，因为我们知道每个值将是一个列表。在主字典循环内部，我们再用一个 `for` 循环❷来遍历每个人的最喜欢语言列表。现在，每个人都可以列出他们喜欢的多个编程语言：

```py
Jen's favorite languages are:
    Python
    Rust

Sarah's favorite languages are:
    C

Edward's favorite languages are:
    Rust
    Go

Phil's favorite languages are:
    Python
    Haskell
```

为了进一步优化这个程序，你可以在字典的`for`循环开始时加入一个`if`语句，检查每个人是否有多个喜欢的编程语言，方法是检查`len(languages)`的值。如果某人有多个喜欢的语言，输出保持不变。如果某人只有一个喜欢的语言，你可以修改措辞以反映这一点。例如，你可以说：“Sarah 最喜欢的编程语言是 C。”

### 字典中的字典

你可以将一个字典嵌套在另一个字典中，但这样做时你的代码可能会迅速变得复杂。例如，如果你为一个网站拥有多个用户，每个用户都有一个唯一的用户名，你可以使用用户名作为字典中的键。然后，你可以使用字典作为与用户名相关联的值来存储每个用户的信息。在下面的代码中，我们存储了每个用户的三项信息：名字、姓氏和位置。我们将通过遍历用户名和与每个用户名相关联的字典来访问这些信息：

**many_users.py**

```py
users = {
    'aeinstein': {
        'first': 'albert',
        'last': 'einstein',
        'location': 'princeton',
        },

    'mcurie': {
        'first': 'marie',
        'last': 'curie',
        'location': 'paris',
        },

    }

❶ for username, user_info in users.items():
❷     print(f"\nUsername: {username}")
❸     full_name = f"{user_info['first']} {user_info['last']}"
    location = user_info['location']

❹     print(f"\tFull name: {full_name.title()}")
    print(f"\tLocation: {location.title()}")
```

我们首先定义一个名为`users`的字典，它包含两个键：分别对应用户名`'aeinstein'`和`'mcurie'`。与每个键关联的值是一个字典，包含每个用户的名字、姓氏和位置。接着，我们遍历`users`字典 ❶。Python 将每个键赋值给变量`username`，与每个用户名相关联的字典则赋值给变量`user_info`。进入主字典循环后，我们打印出用户名 ❷。

接下来，我们开始访问内层字典 ❸。包含用户信息的字典`user_info`有三个键：`'first'`、`'last'`和`'location'`。我们使用每个键来生成每个人整齐格式化的全名和位置，然后打印出我们对每个用户的总结 ❹：

```py
Username: aeinstein
    Full name: Albert Einstein
    Location: Princeton

Username: mcurie
    Full name: Marie Curie
    Location: Paris
```

请注意，每个用户字典的结构是相同的。虽然 Python 没有强制要求这一点，但这种结构使得嵌套字典更容易操作。如果每个用户字典的键不同，`for`循环中的代码将更加复杂。

## 总结

在本章中，你学习了如何定义字典以及如何操作字典中存储的信息。你学习了如何访问和修改字典中的单个元素，以及如何遍历字典中的所有信息。你学会了如何遍历字典的键值对、键和值。你还学会了如何将多个字典嵌套在列表中，将列表嵌套在字典中，和将字典嵌套在字典中。

在下一章，你将学习`while`循环以及如何接受使用你程序的人的输入。这将是一个令人兴奋的章节，因为你将学习如何让你的程序变得互动：它们将能够响应用户输入。

