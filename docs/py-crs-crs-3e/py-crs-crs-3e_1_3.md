
## 第三章：介绍列表

![](img/chapterart.png)

在本章及下一章中，你将学习什么是列表，以及如何开始处理列表中的元素。列表允许你将一组信息存储在一个地方，无论你有几个项目还是数百万个项目。列表是 Python 中最强大的功能之一，易于新程序员使用，并且它连接了编程中的许多重要概念。

## 什么是列表？

*列表*是按特定顺序排列的项目集合。你可以创建一个包含字母表的字母、从 0 到 9 的数字，或你家里所有人的名字的列表。你可以将任何想要的东西放入列表中，且列表中的项目不需要以任何特定方式相关。因为列表通常包含多个元素，所以最好将列表的名称设置为复数形式，比如 `letters`、`digits` 或 `names`。

在 Python 中，方括号（`[]`）表示一个列表，列表中的单个元素用逗号分隔。以下是一个简单的包含几种自行车的列表示例：

**bicycles.py**

```py
bicycles = ['trek', 'cannondale', 'redline', 'specialized']
print(bicycles)
```

如果你要求 Python 打印一个列表，Python 会返回它对该列表的表示，包括方括号：

```py
['trek', 'cannondale', 'redline', 'specialized']
```

因为这不是你希望用户看到的输出，让我们学习如何访问列表中的单个项目。

### 访问列表中的元素

列表是有序的集合，因此你可以通过告诉 Python 项的*位置*或*索引*来访问列表中的任何元素。要访问列表中的元素，写出列表名称，然后用方括号括起该项的索引。

例如，让我们从列表`bicycles`中提取第一辆自行车：

```py
bicycles = ['trek', 'cannondale', 'redline', 'specialized']
print(bicycles[0])
```

当我们从列表中请求单个元素时，Python 只返回该元素，而不带方括号：

```py
trek
```

这是你希望用户看到的结果：干净、整齐格式化的输出。

你还可以对列表中的任何元素使用第二章中的字符串方法。例如，你可以使用`title()`方法将元素`'trek'`格式化为更整洁的外观：

```py
bicycles = ['trek', 'cannondale', 'redline', 'specialized']
print(bicycles[0].title())
```

这个例子产生的输出与前一个例子相同，不同之处在于`'Trek'`被大写了。

### 索引位置从 0 开始，而不是从 1 开始

Python 认为列表中的第一个项目位于位置 0，而不是位置 1。这在大多数编程语言中都是如此，原因与列表操作在较低层次上如何实现有关。如果你遇到意外结果，问问自己是否犯了一个简单但常见的错位错误。

列表中的第二项的索引是 1。使用这种计数系统，你可以通过从元素在列表中的位置减去 1 来获取任何你想要的元素。例如，要访问列表中的第四项，你请求索引为 3 的元素。

以下请求的是索引`1`和索引`3`处的自行车：

```py
bicycles = ['trek', 'cannondale', 'redline', 'specialized']
print(bicycles[1])
print(bicycles[3])
```

这段代码返回列表中的第二和第四辆自行车：

```py
cannondale
specialized
```

Python 有一种特殊的语法来访问列表中的最后一个元素。如果你请求索引`-1`的项，Python 总是返回列表中的最后一个元素：

```py
bicycles = ['trek', 'cannondale', 'redline', 'specialized']
print(bicycles[-1])
```

这段代码返回值`'specialized'`。这种语法非常有用，因为你经常希望在不知道列表长度的情况下访问列表中的最后一个元素。这个约定也适用于其他负索引值。索引`-2`返回列表倒数第二项，索引`-3`返回倒数第三项，依此类推。

### 从列表中使用单个值

你可以像使用任何其他变量一样使用列表中的单个值。例如，你可以使用 f-string 根据列表中的一个值创建消息。

让我们试着从列表中提取第一辆自行车，并使用该值组成一条消息：

```py
bicycles = ['trek', 'cannondale', 'redline', 'specialized']
message = f"My first bicycle was a {bicycles[0].title()}."

print(message)
```

我们使用`bicycles[0]`中的值构建一句话，并将其赋值给变量`message`。输出是关于列表中第一辆自行车的简单句子：

```py
My first bicycle was a Trek.
```

## 修改、添加和删除元素

你创建的大多数列表都是*动态的*，这意味着你将构建一个列表，并在程序运行过程中添加和删除元素。例如，你可能会创建一个游戏，玩家需要把外星人从天上击落。你可以将初始的外星人集合存储在一个列表中，每次击落一个外星人时，就从列表中移除它。每当一个新的外星人出现在屏幕上时，你就将它添加到列表中。你的外星人列表将在游戏过程中不断增加和减少长度。

### 修改列表中的元素

修改元素的语法与访问列表中元素的语法相似。要更改某个元素，使用列表的名称，后跟你想更改的元素的索引，然后提供你希望该项具有的新值。

例如，假设我们有一个摩托车列表，列表中的第一项是`'honda'`。在列表创建后，我们可以更改第一项的值：

**motorcycles.py**

```py
motorcycles = ['honda', 'yamaha', 'suzuki']
print(motorcycles)

motorcycles[0] = 'ducati'
print(motorcycles)
```

这里我们定义了列表`motorcycles`，并将`'honda'`作为第一项。然后我们将第一项的值更改为`'ducati'`。输出显示，第一项已被更改，而其余的列表保持不变：

```py
['honda', 'yamaha', 'suzuki']
['ducati', 'yamaha', 'suzuki']
```

你可以更改列表中任何项的值，而不仅仅是第一项。

### 向列表添加元素

你可能会出于许多原因向列表添加新元素。例如，你可能想要在游戏中让新的外星人出现，向可视化中添加新数据，或者向你构建的网站中添加新的注册用户。Python 提供了几种向现有列表添加新数据的方法。

#### 将元素附加到列表的末尾

向列表添加新元素的最简单方法是将该项*附加*到列表中。当你将一个元素附加到列表时，新的元素会被添加到列表的末尾。使用我们在前面示例中的同一个列表，我们将新元素`'ducati'`添加到列表的末尾：

```py
motorcycles = ['honda', 'yamaha', 'suzuki']
print(motorcycles)

motorcycles.append('ducati')
print(motorcycles)
```

在这里，`append()`方法将`'ducati'`添加到列表的末尾，而不会影响列表中的其他元素：

```py
['honda', 'yamaha', 'suzuki']
['honda', 'yamaha', 'suzuki', 'ducati']
```

`append()`方法使动态构建列表变得非常简单。例如，你可以从一个空列表开始，然后使用一系列`append()`调用将项添加到列表中。使用一个空列表，我们将元素`'honda'`、`'yamaha'`和`'suzuki'`添加到列表中：

```py
motorcycles = []

motorcycles.append('honda')
motorcycles.append('yamaha')
motorcycles.append('suzuki')

print(motorcycles)
```

结果列表与前面的示例中的列表完全相同：

```py
['honda', 'yamaha', 'suzuki']
```

以这种方式构建列表非常常见，因为通常在程序运行之前你无法知道用户想要存储的数据。为了让用户掌控，首先定义一个空列表来保存用户的值。然后将每个新提供的值附加到你刚创建的列表中。

#### 向列表中插入元素

你可以通过使用`insert()`方法将新元素添加到列表中的任何位置。你通过指定新元素的索引和值来实现这一点：

```py
motorcycles = ['honda', 'yamaha', 'suzuki']

motorcycles.insert(0, 'ducati')
print(motorcycles)
```

在这个例子中，我们将值`'ducati'`插入到列表的开头。`insert()`方法在位置`0`处开辟一个空间，并将值`'ducati'`存储在该位置：

```py
['ducati', 'honda', 'yamaha', 'suzuki']
```

这个操作会将列表中的其他每个值向右移动一个位置。

### 从列表中移除元素

通常，你可能希望从列表中移除一个元素或一组元素。例如，当玩家击落一个外星人时，你通常会希望将其从活动外星人列表中移除。或者，当用户决定取消他们在你创建的 Web 应用程序中的账户时，你会希望将该用户从活动用户列表中移除。你可以根据元素在列表中的位置或其值来移除元素。

#### 使用 del 语句移除元素

如果你知道想从列表中移除的元素的位置，可以使用`del`语句：

```py
motorcycles = ['honda', 'yamaha', 'suzuki']
print(motorcycles)

del motorcycles[0]
print(motorcycles)
```

在这里，我们使用`del`语句从摩托车列表中移除第一个项目`'honda'`：

```py
['honda', 'yamaha', 'suzuki']
['yamaha', 'suzuki']
```

如果你知道元素的索引，可以使用`del`语句从列表中的任何位置移除元素。例如，下面是如何从列表中移除第二个元素`'yamaha'`：

```py
motorcycles = ['honda', 'yamaha', 'suzuki']
print(motorcycles)

del motorcycles[1]
print(motorcycles)
```

第二辆摩托车已从列表中删除：

```py
['honda', 'yamaha', 'suzuki']
['honda', 'suzuki']
```

在这两个例子中，使用`del`语句后，你将无法再访问被移除的列表元素。

#### 使用 pop()方法移除元素

有时你可能希望在从列表中移除元素后使用该元素的值。例如，你可能想获取刚被击落的外星人的*x*和*y*位置，然后在该位置绘制爆炸效果。在一个 Web 应用程序中，你可能希望将一个用户从活动成员列表中移除，然后将该用户添加到非活动成员列表中。

`pop()`方法移除列表中的最后一个元素，但它允许你在移除后继续使用该元素。术语*pop*来源于将列表看作一个堆叠的元素，从堆叠顶部弹出一个元素。在这个类比中，堆叠的顶部对应于列表的末尾。

让我们从摩托车列表中弹出一辆摩托车：

```py
❶ motorcycles = ['honda', 'yamaha', 'suzuki']
print(motorcycles)

❷ popped_motorcycle = motorcycles.pop()
❸ print(motorcycles)
❹ print(popped_motorcycle)
```

我们首先定义并打印列表`motorcycles` ❶。然后我们从列表中弹出一个值，并将该值赋给变量`popped_motorcycle` ❷。我们打印列表 ❸ 来展示某个值已从列表中移除。接着我们打印弹出的值 ❹ 来证明我们仍然可以访问被移除的值。

输出显示值`'suzuki'`已从列表末尾移除，并且现在被赋值给变量`popped_motorcycle`：

```py
['honda', 'yamaha', 'suzuki']
['honda', 'yamaha']
suzuki
```

这个`pop()`方法有什么用呢？假设摩托车列表是按我们拥有它们的时间顺序存储的。如果是这种情况，我们可以使用`pop()`方法打印一条关于我们最后购买的摩托车的语句：

```py
motorcycles = ['honda', 'yamaha', 'suzuki']

last_owned = motorcycles.pop()
print(f"The last motorcycle I owned was a {last_owned.title()}.")
```

输出是关于我们最近拥有的摩托车的一句简单话语：

```py
The last motorcycle I owned was a Suzuki.
```

#### 从列表中的任何位置弹出元素

你可以使用`pop()`方法通过在括号中指定要删除的项的索引，来从列表中的任何位置删除一个项：

```py
motorcycles = ['honda', 'yamaha', 'suzuki']

first_owned = motorcycles.pop(0)
print(f"The first motorcycle I owned was a {first_owned.title()}.")
```

我们首先通过`pop()`删除列表中的第一辆摩托车，然后打印关于这辆摩托车的信息。输出是描述我曾拥有的第一辆摩托车的简单句子：

```py
The first motorcycle I owned was a Honda.
```

记住，每次使用`pop()`时，你处理的项将不再存储在列表中。

如果你不确定是使用`del`语句还是`pop()`方法，以下是一个简单的判断方法：当你想从列表中删除一个项并且不再使用该项时，使用`del`语句；如果你在删除项的同时还希望使用该项，使用`pop()`方法。

#### 按值删除项

有时你并不知道要从列表中删除的值的位置。如果你只知道要删除的项的值，可以使用`remove()`方法。

例如，假设我们想从摩托车列表中删除值`'ducati'`：

```py
motorcycles = ['honda', 'yamaha', 'suzuki', 'ducati']
print(motorcycles)

motorcycles.remove('ducati')
print(motorcycles)
```

这里`remove()`方法告诉 Python 找到`'ducati'`在列表中出现的位置，并删除该元素：

```py
['honda', 'yamaha', 'suzuki', 'ducati']
['honda', 'yamaha', 'suzuki']
```

你也可以使用`remove()`方法来处理从列表中删除的值。让我们删除值`'ducati'`并打印删除该项的原因：

```py
❶ motorcycles = ['honda', 'yamaha', 'suzuki', 'ducati']
print(motorcycles)

❷ too_expensive = 'ducati'
❸ motorcycles.remove(too_expensive)
print(motorcycles)
❹ print(f"\nA {too_expensive.title()} is too expensive for me.")
```

在定义列表❶之后，我们将值`'ducati'`赋给一个名为`too_expensive`的变量❷。然后，我们使用这个变量告诉 Python 要从列表中删除哪个值❸。值`'ducati'`已经从列表中删除❹，但仍可以通过变量`too_expensive`访问，从而允许我们打印一条关于为什么从摩托车列表中删除`'ducati'`的声明：

```py
['honda', 'yamaha', 'suzuki', 'ducati']
['honda', 'yamaha', 'suzuki']

A Ducati is too expensive for me.
```

## 组织列表

通常，列表会以不可预测的顺序创建，因为你无法总是控制用户提供数据的顺序。虽然在大多数情况下这是不可避免的，但你经常会希望以特定的顺序展示信息。有时你想保留列表的原始顺序，其他时候你可能想改变原始顺序。Python 根据不同情况提供了多种方法来组织列表。

### 使用`sort()`方法永久排序列表

Python 的`sort()`方法使得排序列表变得相对简单。假设我们有一份汽车列表，并且想要将列表的顺序改为按字母顺序排列。为了简化任务，假设列表中的所有值都是小写字母：

**cars.py**

```py
cars = ['bmw', 'audi', 'toyota', 'subaru']
cars.sort()
print(cars)
```

`sort()`方法会永久改变列表的顺序。现在这些汽车已经按字母顺序排列，我们将无法恢复到原来的顺序：

```py
['audi', 'bmw', 'subaru', 'toyota']
```

你也可以通过向`sort()`方法传递参数`reverse=True`，按逆字母顺序对列表进行排序。以下示例将汽车列表按逆字母顺序排序：

```py
cars = ['bmw', 'audi', 'toyota', 'subaru']
cars.sort(reverse=True)
print(cars)
```

再次强调，列表的顺序已经永久改变：

```py
['toyota', 'subaru', 'bmw', 'audi']
```

### 使用`sorted()`函数临时排序列表

如果你希望保持列表的原始顺序，但以排序的顺序显示列表，可以使用`sorted()`函数。`sorted()`函数允许你以特定的顺序显示列表，但不会影响列表的实际顺序。

让我们在汽车列表上尝试这个函数。

```py
cars = ['bmw', 'audi', 'toyota', 'subaru']

❶ print("Here is the original list:")
print(cars)

❷ print("\nHere is the sorted list:")
print(sorted(cars))

❸ print("\nHere is the original list again:")
print(cars)
```

我们首先按原始顺序打印列表❶，然后按字母顺序打印❷。在列表按照新顺序显示后，我们展示该列表仍然以原始顺序存储❸：

```py
Here is the original list:
['bmw', 'audi', 'toyota', 'subaru']

Here is the sorted list:
['audi', 'bmw', 'subaru', 'toyota']

❶ Here is the original list again:
['bmw', 'audi', 'toyota', 'subaru']
```

请注意，在使用`sorted()`函数之后，列表仍然保留原始顺序❶。如果你希望列表按反字母顺序显示，`sorted()`函数还可以接受`reverse=True`参数。

### 以相反的顺序打印列表

要反转列表的原始顺序，你可以使用`reverse()`方法。如果我们最初按照拥有这些车的时间顺序存储了这些车的列表，我们可以很容易地将列表按时间倒序排列：

```py
cars = ['bmw', 'audi', 'toyota', 'subaru']
print(cars)

cars.reverse()
print(cars)
```

请注意，`reverse()`不会按照字母顺序倒排；它只是反转列表的顺序：

```py
['bmw', 'audi', 'toyota', 'subaru']
['subaru', 'toyota', 'audi', 'bmw']
```

`reverse()`方法会永久改变列表的顺序，但你可以通过再次对同一个列表调用`reverse()`来恢复原来的顺序。

### 查找列表的长度

你可以通过使用`len()`函数快速找到列表的长度。这个示例中的列表有四个元素，所以它的长度是`4`：

```py
>>> **cars = ['bmw', 'audi', 'toyota', 'subaru']**
>>> **len(cars)**
4
```

你会发现，`len()`在你需要确定游戏中仍需击落的外星人数量、确定你需要管理的可视化数据量或计算网站上注册用户数量等任务时非常有用。

## 在处理列表时避免索引错误

在你第一次使用列表时，有一种常见的错误类型。假设你有一个包含三个元素的列表，而你请求第四个元素：

**motorcycles.py**

```py
motorcycles = ['honda', 'yamaha', 'suzuki']
print(motorcycles[3])
```

这个例子会导致一个*索引错误*：

```py
Traceback (most recent call last):
  File "motorcycles.py", line 2, in <module>
    print(motorcycles[3])
          ~~~~~~~~~~~^^^
IndexError: list index out of range
```

Python 试图给你索引为 3 的元素。但当它查找列表时，`motorcycles`中没有索引为 3 的元素。由于列表索引的下标从 0 开始，这种错误是常见的。人们通常认为第三个元素是索引 3 的元素，因为他们习惯从 1 开始计数。但在 Python 中，第三个元素的索引是 2，因为索引从 0 开始。

索引错误意味着 Python 无法找到你请求的索引位置的元素。如果你的程序中发生了索引错误，试着调整你请求的索引值，然后再次运行程序看看结果是否正确。

请记住，每当你想访问列表中的最后一个元素时，你应该使用索引`-1`。即使你的列表自上次访问以来发生了变化，这个方法也始终有效：

```py
motorcycles = ['honda', 'yamaha', 'suzuki']
print(motorcycles[-1])
```

索引`-1`总是返回列表中的最后一个元素，在这个例子中是值`suzuki`：

```py
suzuki
```

这种方法唯一会引发错误的情况是，当你请求一个空列表中的最后一个元素时：

```py
motorcycles = []
print(motorcycles[-1])
```

`motorcycles` 中没有项目，因此 Python 返回另一个索引错误：

```py
Traceback (most recent call last):
  File "motorcyles.py", line 3, in <module>
    print(motorcycles[-1])
          ~~~~~~~~~~~^^^^
IndexError: list index out of range
```

如果发生索引错误，并且你无法弄清楚如何解决它，可以尝试打印你的列表或仅打印列表的长度。你的列表可能与你想象的非常不同，尤其是当它由程序动态管理时。查看实际的列表或列表中项目的确切数量，可以帮助你理清这些逻辑错误。

## 总结

在本章中，你学习了什么是列表以及如何处理列表中的各个项目。你学习了如何定义列表，如何添加和移除元素。你还学习了如何对列表进行永久和临时排序，以便展示。你还学习了如何查找列表的长度，以及如何在处理列表时避免索引错误。

在第四章中，你将学习如何更高效地处理列表中的项目。通过仅使用几行代码遍历列表中的每一项，你将能够高效地工作，即使列表包含数千或数百万个项目。

