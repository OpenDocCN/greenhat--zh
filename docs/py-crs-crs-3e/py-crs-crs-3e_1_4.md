
## 处理列表

![](img/chapterart.png)

在第三章中，你学习了如何创建一个简单的列表，并学习了如何处理列表中的单个元素。在本章中，你将学习如何使用仅几行代码遍历整个列表，无论列表有多长。*循环*允许你对列表中的每一项执行相同的操作或一组操作。因此，你将能够高效地处理任何长度的列表，包括那些包含数千或甚至数百万个项目的列表。

## 遍历整个列表

你常常需要遍历列表中的所有条目，对每一项执行相同的任务。例如，在游戏中，你可能想要将屏幕上的每个元素移动相同的距离。在一个数字列表中，你可能想对每个元素执行相同的统计操作。或者，你可能想在一个网站上显示每篇文章的标题。当你想对列表中的每一项执行相同的操作时，可以使用 Python 的 `for` 循环。

假设我们有一个魔术师名字的列表，并且我们想打印出列表中的每个名字。我们可以通过单独取出列表中的每个名字来做到这一点，但这种方法可能会导致几个问题。首先，对于一个很长的名字列表，重复这样的操作会很繁琐。其次，每次列表长度发生变化时，我们必须修改代码。使用 `for` 循环可以避免这两个问题，因为 Python 会内部处理这些问题。

让我们使用 `for` 循环打印出魔术师名单中的每个名字：

**magicians.py**

```py
magicians = ['alice', 'david', 'carolina']
for magician in magicians:
    print(magician)
```

我们从定义一个列表开始，就像我们在第三章中做的那样。然后我们定义一个`for`循环。这一行告诉 Python 从列表`magicians`中取出一个名字，并将其与变量`magician`关联。接下来，我们告诉 Python 打印刚刚分配给`magician`的名字。Python 接着重复这两行代码，每个列表中的名字执行一次。你可以将这段代码理解为“对于`magicians`列表中的每个魔术师，打印该魔术师的名字。”输出将是列表中每个名字的简单打印：

```py
alice
david
carolina
```

### 更深入地了解循环

循环很重要，因为它是计算机自动化重复任务的最常见方式之一。例如，在像我们在*magicians.py*中使用的简单循环中，Python 首先读取循环的第一行：

```py
for magician in magicians:
```

这一行告诉 Python 从列表`magicians`中提取第一个值，并将其与变量`magician`关联。这个第一个值是`'alice'`。然后，Python 继续读取下一行：

```py
 print(magician)
```

Python 再次打印当前`magician`的值，仍然是`'alice'`。由于列表中还有更多值，Python 返回到循环的第一行：

```py
for magician in magicians:
```

Python 检索列表中的下一个名字`'david'`，并将该值与变量`magician`关联。然后，Python 执行这一行：

```py
 print(magician)
```

Python 再次打印当前`magician`的值，这次是`'david'`。Python 再次执行整个循环，使用列表中的最后一个值`'carolina'`。由于列表中没有更多的值，Python 转到程序中的下一行。在这种情况下，`for`循环后面没有更多的代码，所以程序结束。

当你第一次使用循环时，请记住，这些步骤会针对列表中的每个项目重复执行一次，不管列表中有多少项。如果列表中有一百万个项目，Python 将这些步骤重复执行一百万次——通常执行得非常快。

另外，在编写你自己的`for`循环时，记住你可以为与列表中每个值关联的临时变量选择任何你想要的名称。然而，选择一个有意义的名称来表示列表中的单个项是很有帮助的。例如，以下是为猫的列表、狗的列表和一般物品列表编写`for`循环的一个好方法：

```py
for cat in cats:
for dog in dogs:
for item in list_of_items:
```

这些命名约定可以帮助你跟踪在`for`循环中每个项所执行的操作。使用单数和复数名称可以帮助你识别代码的某一部分是在处理列表中的单个元素还是整个列表。

### 在`for`循环中做更多的工作

你可以对`for`循环中的每个项目做几乎任何事情。让我们在之前的例子基础上，向每个魔术师打印一条消息，告诉他们他们表演了一个很棒的魔术：

**magicians.py**

```py
magicians = ['alice', 'david', 'carolina']
for magician in magicians:
    print(f"{magician.title()}, that was a great trick!")
```

这段代码唯一的区别在于我们为每个魔术师构建消息，消息从该魔术师的名字开始。在第一次执行循环时，`magician`的值是`'alice'`，所以 Python 会用名字`'Alice'`开始第一条消息。第二次执行时，消息会以`'David'`开头，第三次执行时，消息会以`'Carolina'`开头。

输出显示了列表中每位魔术师的个性化消息：

```py
Alice, that was a great trick!
David, that was a great trick!
Carolina, that was a great trick!
```

你也可以在`for`循环中写任意多行代码。在`for magician in magicians`这一行之后的每一行缩进代码都被认为是在*循环内*，并且每一行会为列表中的每个值执行一次。因此，你可以在每个列表项上执行任意多的操作。

让我们为消息添加第二行，告诉每位魔术师我们期待他们的下一个魔术：

```py
magicians = ['alice', 'david', 'carolina']
for magician in magicians:
 print(f"{magician.title()}, that was a great trick!")
    print(f"I can't wait to see your next trick, {magician.title()}.\n")
```

由于我们已经缩进了两次`print()`的调用，因此每一行会为列表中的每个魔术师执行一次。第二次`print()`调用中的换行符（`"\n"`）会在每次循环后插入一个空行。这就创建了一组整齐分组的消息，每个人都有一条：

```py
Alice, that was a great trick!
I can't wait to see your next trick, Alice.

David, that was a great trick!
I can't wait to see your next trick, David.

Carolina, that was a great trick!
I can't wait to see your next trick, Carolina.
```

你可以在`for`循环中使用任意多的行。实际上，你会发现，在使用`for`循环时，通常需要对列表中的每个项目执行多种不同的操作。

### 在`for`循环后做一些事情

一旦`for`循环执行完毕会发生什么？通常，你会希望总结输出块，或者继续执行程序中必须完成的其他工作。

在`for`循环后面任何没有缩进的代码行都只会执行一次，而不会重复执行。我们来为魔术师们写一条感谢信息，感谢他们为大家呈现了一场精彩的表演。为了在打印完所有个人消息后显示这条群体消息，我们将感谢信息写在`for`循环之后，且不缩进：

```py
magicians = ['alice', 'david', 'carolina']
for magician in magicians:
 print(f"{magician.title()}, that was a great trick!")
 print(f"I can't wait to see your next trick, {magician.title()}.\n")

print("Thank you, everyone. That was a great magic show!")
```

前两次调用`print()`会为列表中的每个魔术师重复一次，正如你之前所看到的。然而，由于最后一行没有缩进，它只会执行一次：

```py
Alice, that was a great trick!
I can't wait to see your next trick, Alice.

David, that was a great trick!
I can't wait to see your next trick, David.

Carolina, that was a great trick!
I can't wait to see your next trick, Carolina.

Thank you, everyone. That was a great magic show!
```

当你使用`for`循环处理数据时，你会发现这是总结对整个数据集执行的操作的好方法。例如，你可能会用`for`循环来初始化一个游戏，遍历角色列表并在屏幕上显示每个角色。然后你可能会在循环后写一些额外的代码，在所有角色都显示在屏幕后显示一个*现在开始游戏*按钮。

## 避免缩进错误

Python 使用缩进来确定某一行或一组行与程序其余部分的关系。在之前的示例中，打印每个魔术师消息的那些行属于 `for` 循环，因为它们是缩进的。Python 使用缩进使得代码非常容易阅读。基本上，它通过空白符强制你编写格式整洁且具有清晰视觉结构的代码。在更长的 Python 程序中，你会注意到有些代码块在不同的缩进级别上。这些缩进级别帮助你对整个程序的组织结构有一个大致的了解。

当你开始编写依赖于正确缩进的代码时，你需要注意一些常见的*缩进错误*。例如，人们有时会缩进不需要缩进的代码行，或者忘记缩进需要缩进的代码行。现在看到这些错误的示例会帮助你在将来避免它们，并在自己的程序中出现时修正它们。

让我们来看看一些更常见的缩进错误。

### 忘记缩进

在循环中的 `for` 语句后总是需要缩进。如果你忘记了，Python 会提醒你：

**magicians.py**

```py
magicians = ['alice', 'david', 'carolina']
for magician in magicians:
❶ print(magician)
```

调用 `print()` ❶ 应该缩进，但它没有缩进。当 Python 期望一个缩进块却没有找到时，它会告诉你它在哪一行出现了问题：

```py
 File "magicians.py", line 3
    print(magician)
    ^
IndentationError: expected an indented block after 'for' statement on line 2
```

通常，你可以通过在 `for` 语句后立即缩进该行或多行来解决这种缩进错误。

### 忘记缩进额外的行

有时你的循环会在没有错误的情况下运行，但却没有产生预期的结果。这种情况通常发生在你试图在循环中执行多个任务，却忘记缩进其中的一些行时。

例如，当我们忘记缩进循环中的第二行（这行代码告诉每个魔术师我们期待他们的下一个魔术时），就会发生以下情况：

```py
magicians = ['alice', 'david', 'carolina']
for magician in magicians:
    print(f"{magician.title()}, that was a great trick!")
❶ print(f"I can't wait to see your next trick, {magician.title()}.\n")
```

第二次调用 `print()` ❶ 应该缩进，但由于 Python 在 `for` 语句后发现至少有一行缩进的代码，它并不会报错。因此，第一个 `print()` 调用会针对列表中的每个名字执行一次，因为它是缩进的。第二个 `print()` 调用没有缩进，所以它只会在循环结束后执行一次。由于与 `magician` 关联的最终值是 `'carolina'`，因此只有她收到了“期待下一场魔术”的消息：

```py
Alice, that was a great trick!
David, that was a great trick!
Carolina, that was a great trick!
I can't wait to see your next trick, Carolina.
```

这是一个*逻辑错误*。语法上是有效的 Python 代码，但由于逻辑上出现问题，代码并没有产生预期的结果。如果你期望看到某个操作对列表中的每个项都执行一次，而它只执行了一次，请检查是否需要缩进某一行或一组行。

### 不必要的缩进

如果你不小心缩进了一个不需要缩进的行，Python 会提示你关于意外缩进的错误：

**hello_world.py**

```py
message = "Hello Python world!"
    print(message)
```

我们不需要缩进 `print()` 调用，因为它不是循环的一部分；因此，Python 会报告这个错误：

```py
 File "hello_world.py", line 2
    print(message)
    ^
IndentationError: unexpected indent
```

通过只有在有特定理由时才进行缩进，你可以避免意外的缩进错误。在你此时编写的程序中，唯一需要缩进的行是你希望在每次 `for` 循环中的每个项目上重复执行的操作。

### 循环后不必要的缩进

如果你不小心缩进了应当在循环结束后运行的代码，这段代码将在列表中的每一项上都执行一次。有时这会导致 Python 报告错误，但通常会导致逻辑错误。

例如，看看当我们不小心缩进了感谢魔术师们集体表现出色的那一行时会发生什么：

**magicians.py**

```py
magicians = ['alice', 'david', 'carolina']
for magician in magicians:
    print(f"{magician.title()}, that was a great trick!")
    print(f"I can't wait to see your next trick, {magician.title()}.\n")

❶     print("Thank you everyone, that was a great magic show!")
```

由于最后一行❶有缩进，它会为列表中的每一个人都执行一次：

```py
Alice, that was a great trick!
I can't wait to see your next trick, Alice.

Thank you everyone, that was a great magic show!
David, that was a great trick!
I can't wait to see your next trick, David.

Thank you everyone, that was a great magic show!
Carolina, that was a great trick!
I can't wait to see your next trick, Carolina.

Thank you everyone, that was a great magic show!
```

这是另一个逻辑错误，类似于第 54 页中“忘记缩进额外的行”中的错误。由于 Python 不知道你想通过代码实现什么，它会运行所有符合语法规则的代码。如果某个操作应当仅执行一次，却被多次重复执行，你可能需要取消该操作的缩进。

### 忘记冒号

`for`语句末尾的冒号告诉 Python 将下一行解释为循环的开始。

```py
magicians = ['alice', 'david', 'carolina']
❶ for magician in magicians
    print(magician)
```

如果你不小心忘记了冒号❶，你会得到语法错误，因为 Python 不知道你到底想做什么：

```py
 File "magicians.py", line 2
    for magician in magicians
                             ^
SyntaxError: expected ':'
```

Python 不知道你是仅仅忘记了冒号，还是打算写额外的代码来设置一个更复杂的循环。如果解释器能够识别出一个可能的修复，它会给出建议，比如在行尾加上冒号，就像这里的 `expected ':'` 响应一样。有些错误有简单、显而易见的修复，这要归功于 Python 错误追踪中提供的建议。也有一些错误，即使最终的修复仅涉及一个字符，也更难解决。当一个小修复花费很长时间才找到时，不必感到沮丧；你绝对不是唯一有这种经历的人。

## 创建数字列表

存储一组数字有很多理由。例如，你可能需要跟踪游戏中每个角色的位置，也许还想记录玩家的最高分。在数据可视化中，你几乎总是会处理一组数字，比如温度、距离、人口大小，或者经纬度值等各种类型的数字集合。

列表非常适合存储数字集合，Python 提供了多种工具来帮助你高效地处理数字列表。一旦你理解了如何有效地使用这些工具，即使你的列表包含数百万个项目，代码也能顺利运行。

### 使用 `range()` 函数

Python 的 `range()` 函数使得生成一系列数字变得简单。例如，你可以使用 `range()` 函数像这样打印一系列数字：

**first_numbers.py**

```py
for value in range(1, 5):
    print(value)
```

尽管这段代码看起来应该打印从 1 到 5 的数字，但它没有打印数字 5：

```py
1
2
3
4
```

在这个例子中，`range()`只打印 1 到 4 之间的数字。这是编程语言中常见的“偏差一”行为的另一个例子。`range()`函数会让 Python 从你提供的第一个值开始计数，并在达到你提供的第二个值时停止。因为它会在第二个值时停止，输出中永远不会包含结束值，而在这个例子中，结束值本应是 5。

要打印从 1 到 5 的数字，你可以使用`range(1, 6)`：

```py
for value in range(1, 6):
    print(value)
```

这次输出从 1 开始，到 5 结束：

```py
1
2
3
4
5
```

如果你在使用`range()`时，输出与预期不符，尝试调整结束值加 1。

你也可以只传递一个参数给`range()`，它会从 0 开始生成数字序列。例如，`range(6)`将返回从 0 到 5 的数字。

### 使用`range()`创建一个数字列表

如果你想制作一个数字列表，你可以使用`list()`函数将`range()`的结果直接转换为列表。当你将`list()`包裹在对`range()`函数的调用周围时，输出将是一个数字列表。

在前面章节的例子中，我们仅仅是打印了一系列数字。我们可以使用`list()`将这组数字转换为列表：

```py
numbers = list(range(1, 6))
print(numbers)
```

这是结果：

```py
[1, 2, 3, 4, 5]
```

我们还可以使用`range()`函数告诉 Python 跳过给定范围内的数字。如果你传递第三个参数给`range()`，Python 会使用该值作为生成数字时的步长。

例如，下面是如何列出 1 到 10 之间的偶数：

**even_numbers.py**

```py
even_numbers = list(range(2, 11, 2))
print(even_numbers)
```

在这个例子中，`range()`函数从值 2 开始，然后将 2 加到这个值上。它会重复加 2，直到达到或超过结束值 11，并产生如下结果：

```py
[2, 4, 6, 8, 10]
```

你可以使用`range()`函数创建几乎任何你想要的数字集。例如，考虑如何制作一个包含前 10 个平方数的列表（即从 1 到 10 的每个整数的平方）。在 Python 中，两个星号（`**`）表示指数运算。下面是如何将前 10 个平方数放入列表的示例：

**square_numbers.py**

```py
squares = []
for value in range(1, 11):
❶     square = value ** 2
❷     squares.append(square)

print(squares)
```

我们首先创建一个空列表`squares`。然后，告诉 Python 使用`range()`函数循环遍历 1 到 10 之间的每个值。在循环内部，当前值被平方并赋值给变量`square` ❶。每次新的`square`值都会被追加到`squares`列表中 ❷。最后，当循环结束时，平方数的列表会被打印出来：

```py
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

为了使这段代码更简洁，可以省略临时变量`square`，并将每个新值直接添加到列表中：

```py
squares = []
for value in range(1,11):
    squares.append(value**2)

print(squares)
```

这一行完成了与前面列表中`for`循环内部相同的工作。循环中的每个值都会被平方，然后立即追加到平方数列表中。

在制作更复杂的列表时，你可以使用这两种方法中的任何一种。有时使用临时变量可以让你的代码更易读；有时则可能使代码变得冗长。首先关注编写自己清楚理解的代码，并确保它能按预期执行。然后，在复查代码时，寻找更高效的方法。

### 使用数字列表进行简单统计

在处理数字列表时，几个 Python 函数非常有用。例如，你可以轻松地找到数字列表中的最小值、最大值和总和：

```py
>>> **digits = [1, 2, 3, 4, 5, 6, 7, 8, 9, 0]**
>>> **min(digits)**
0
>>> **max(digits)**
9
>>> **sum(digits)**
45
```

### 列表推导式

前面描述的生成 `squares` 列表的方法需要使用三到四行代码。一个 *列表推导式* 可以让你仅用一行代码生成相同的列表。列表推导式将 `for` 循环和新元素的创建合并为一行，并自动附加每个新元素。列表推导式通常不会首先介绍给初学者，但我在这里包含它们，因为一旦你开始查看其他人的代码，你很可能会看到它们。

以下示例构建了与之前相同的平方数列表，但使用了列表推导式：

**squares.py**

```py
squares = [value**2 for value in range(1, 11)]
print(squares)
```

要使用这种语法，首先为列表取一个描述性名称，例如 `squares`。接着，打开一对方括号，定义你想存储到新列表中的表达式。在这个例子中，表达式是 `value**2`，它将值提升到二次方。然后，写一个 `for` 循环来生成你想要传入表达式的数字，最后关闭方括号。这个例子中的 `for` 循环是 `for value in range(1, 11)`，它将值 1 到 10 传入表达式 `value**2`。注意，在 `for` 语句末尾不使用冒号。

结果是你之前看到的相同的平方数列表：

```py
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

编写你自己的列表推导式需要练习，但一旦你熟悉了创建普通列表，你会发现它们非常值得。当你编写三到四行代码来生成列表，并开始觉得重复时，考虑编写你自己的列表推导式。

## 操作列表的一部分

在第三章，你学习了如何访问列表中的单个元素，而在本章，你正在学习如何遍历列表中的所有元素。你还可以操作列表中的特定项集合，这在 Python 中称为 *切片*。

### 切片列表

要制作切片，你需要指定你想操作的第一个和最后一个元素的索引。与 `range()` 函数一样，Python 会在你指定的第二个索引之前停止。所以，要输出列表中的前三个元素，你需要请求索引 `0` 到 `3`，这将返回元素 `0`、`1` 和 `2`。

以下示例涉及一个团队的玩家列表：

**players.py**

```py
players = ['charles', 'martina', 'michael', 'florence', 'eli']
print(players[0:3])
```

这段代码打印了列表的一个切片。输出保留了列表的结构，并包括了列表中的前三个玩家：

```py
['charles', 'martina', 'michael']
```

你可以生成列表的任何子集。例如，如果你想要列表中的第二、第三和第四个项目，你可以从索引`1`开始切片，结束于索引`4`：

```py
players = ['charles', 'martina', 'michael', 'florence', 'eli']
print(players[1:4])
```

这次切片从`'martina'`开始，直到`'florence'`结束：

```py
['martina', 'michael', 'florence']
```

如果你在切片中省略第一个索引，Python 会自动从列表的开头开始切片：

```py
players = ['charles', 'martina', 'michael', 'florence', 'eli']
print(players[:4])
```

没有起始索引时，Python 会从列表的开头开始：

```py
['charles', 'martina', 'michael', 'florence']
```

如果你想要包括列表末尾的切片，类似的语法也适用。例如，如果你想要从第三个项目到最后一个项目的所有项，可以从索引`2`开始，省略第二个索引：

```py
players = ['charles', 'martina', 'michael', 'florence', 'eli']
print(players[2:])
```

Python 会返回从第三个项目到列表末尾的所有项目：

```py
['michael', 'florence', 'eli']
```

这种语法允许你从列表的任何位置输出所有元素到列表的末尾，无论列表的长度如何。记住，负索引返回距离列表末尾一定距离的元素；因此，你可以从列表的末尾输出任何切片。例如，如果我们想要输出名册中的最后三位玩家，我们可以使用切片`players[-3:]`：

```py
players = ['charles', 'martina', 'michael', 'florence', 'eli']
print(players[-3:])
```

这会打印出最后三位玩家的名字，并且会随着玩家列表的变化继续有效。

### 遍历切片

如果你想要遍历列表中某个子集的元素，可以在`for`循环中使用切片。在下一个例子中，我们遍历前三个玩家，并将他们的名字作为简单名册的一部分打印出来：

```py
players = ['charles', 'martina', 'michael', 'florence', 'eli']

print("Here are the first three players on my team:")
❶ for player in players[:3]:
    print(player.title())
```

Python 并不是遍历整个玩家列表，而是只遍历前三个名字 ❶：

```py
Here are the first three players on my team:
Charles
Martina
Michael
```

切片在许多情况下非常有用。例如，当你创建游戏时，每次玩家完成游戏后，你可以将该玩家的最终得分添加到列表中。然后，你可以通过将列表按降序排序，并提取一个包含前三个得分的切片，来获取玩家的前三个得分。当你处理数据时，你可以使用切片将数据分成特定大小的块进行处理。或者，当你构建一个 Web 应用程序时，你可以使用切片在一系列页面中显示信息，每个页面显示适量的信息。

### 复制列表

通常，你会想从一个现有的列表开始，并根据第一个列表创建一个全新的列表。让我们来探索如何复制一个列表，并看看在什么情况下复制列表是有用的。

要复制一个列表，你可以通过省略第一个索引和第二个索引（`[:]`）来创建一个包含原始列表所有元素的切片。这告诉 Python 从第一个项目开始切片，到最后一个项目结束，从而生成整个列表的副本。

例如，假设我们有一个关于我们最喜欢的食物的列表，并想创建一个包含朋友喜欢的食物的单独列表。这个朋友喜欢我们列表中的所有食物，因此我们可以通过复制我们的列表来创建他们的列表：

**foods.py**

```py
my_foods = ['pizza', 'falafel', 'carrot cake']
❶ friend_foods = my_foods[:]

print("My favorite foods are:")
print(my_foods)

print("\nMy friend's favorite foods are:")
print(friend_foods)
```

首先，我们创建一个包含我们喜欢的食物的列表，叫做 `my_foods`。然后，我们创建一个新列表，叫做 `friend_foods`。我们通过请求 `my_foods` 的切片而不指定任何索引❶，并将复制品赋值给 `friend_foods`。当我们打印每个列表时，我们会看到它们都包含相同的食物：

```py
My favorite foods are:
['pizza', 'falafel', 'carrot cake']

My friend's favorite foods are:
['pizza', 'falafel', 'carrot cake']
```

为了证明我们实际上有两个独立的列表，我们将向每个列表中添加新的食物，并展示每个列表是如何追踪相应人物的最爱食物的：

```py
my_foods = ['pizza', 'falafel', 'carrot cake']
❶ friend_foods = my_foods[:]

❷ my_foods.append('cannoli')
❸ friend_foods.append('ice cream')

print("My favorite foods are:")
print(my_foods)

print("\nMy friend's favorite foods are:")
print(friend_foods)
```

我们将原始的 `my_foods` 中的项目复制到新列表 `friend_foods` 中，就像在之前的示例中一样❶。接下来，我们向每个列表中添加新食物：我们将 `'cannoli'` 添加到 `my_foods` ❷，将 `'ice cream'` 添加到 `friend_foods` ❸。然后我们打印这两个列表，看看这些食物是否出现在相应的列表中：

```py
My favorite foods are:
['pizza', 'falafel', 'carrot cake', 'cannoli']

My friend's favorite foods are:
['pizza', 'falafel', 'carrot cake', 'ice cream']
```

输出结果显示，`'cannoli'` 现在出现在我们的最爱食物列表中，但 `'ice cream'` 没有。我们可以看到，`'ice cream'` 现在出现在朋友的列表中，但 `'cannoli'` 没有。如果我们只是将 `friend_foods` 设置为 `my_foods`，我们就不会得到两个独立的列表。例如，下面是当你尝试在没有使用切片的情况下复制列表时发生的情况：

```py
my_foods = ['pizza', 'falafel', 'carrot cake']

# This doesn't work:
friend_foods = my_foods

my_foods.append('cannoli')
friend_foods.append('ice cream')

print("My favorite foods are:")
print(my_foods)

print("\nMy friend's favorite foods are:")
print(friend_foods)
```

我们不是将 `my_foods` 的副本赋给 `friend_foods`，而是将 `friend_foods` 设置为 `my_foods`。这种语法实际上告诉 Python，将新的变量 `friend_foods` 与已经与 `my_foods` 关联的列表关联起来，因此现在两个变量都指向同一个列表。因此，当我们将 `'cannoli'` 添加到 `my_foods` 时，它也会出现在 `friend_foods` 中。同样，`'ice cream'` 会出现在两个列表中，即使它似乎只被添加到了 `friend_foods`。

输出结果显示，两个列表现在是相同的，这不是我们想要的：

```py
My favorite foods are:
['pizza', 'falafel', 'carrot cake', 'cannoli', 'ice cream']

My friend's favorite foods are:
['pizza', 'falafel', 'carrot cake', 'cannoli', 'ice cream']
```

## 元组

列表适用于存储在程序运行期间可能会发生变化的项目集合。当你在网站上处理用户列表或游戏中的角色列表时，能够修改列表尤其重要。然而，有时你希望创建一个不能改变的项目列表。元组可以让你做到这一点。Python 将不能更改的值称为*不可变*，而不可变的列表称为*元组*。

### 定义元组

元组看起来就像列表，只不过使用圆括号而不是方括号。一旦定义了元组，你可以通过使用每个项的索引来访问个别元素，就像访问列表一样。

例如，如果我们有一个矩形，它的大小应该始终保持不变，我们可以通过将其尺寸放入元组中来确保其大小不会改变：

**dimensions.py**

```py
dimensions = (200, 50)
print(dimensions[0])
print(dimensions[1])
```

我们定义了元组 `dimensions`，使用圆括号而不是方括号。然后，我们使用与访问列表元素相同的语法，逐个打印元组中的每个元素：

```py
200
50
```

让我们看看如果尝试改变元组 `dimensions` 中的某个项目会发生什么：

```py
dimensions = (200, 50)
dimensions[0] = 250
```

这段代码试图更改第一个维度的值，但 Python 返回了类型错误。因为我们试图修改元组的值，而元组是不可更改的对象类型，Python 告诉我们不能为元组中的项赋予新值：

```py
Traceback (most recent call last):
  File "dimensions.py", line 2, in <module>
    dimensions[0] = 250
TypeError: 'tuple' object does not support item assignment
```

这是有益的，因为我们希望在代码行尝试更改矩形的维度时，Python 能引发错误。

### 遍历元组中的所有值

你可以像遍历列表一样，使用`for`循环遍历元组中的所有值：

```py
dimensions = (200, 50)
for dimension in dimensions:
    print(dimension)
```

Python 返回元组中的所有元素，就像它对待列表一样：

```py
200
50
```

### 重写元组

虽然你不能修改元组，但你可以为表示元组的变量赋予新值。例如，如果我们想改变这个矩形的维度，我们可以重新定义整个元组：

```py
dimensions = (200, 50)
print("Original dimensions:")
for dimension in dimensions:
    print(dimension)

dimensions = (400, 100)
print("\nModified dimensions:")
for dimension in dimensions:
    print(dimension)
```

前四行定义了原始元组并打印了初始维度。然后，我们将一个新元组与变量`dimensions`关联，并打印新值。这次 Python 不会引发错误，因为重新赋值变量是有效的：

```py
Original dimensions:
200
50

Modified dimensions:
400
100
```

与列表相比，元组是简单的数据结构。当你想存储一组在程序生命周期中不应更改的值时，使用元组。

## 代码风格

现在你正在编写更长的程序，学习如何一致地编写代码风格是一个好主意。花时间让你的代码尽可能易读。编写易读的代码有助于你跟踪程序的运行，并且也能帮助别人理解你的代码。

Python 程序员已达成共识，遵循一系列的代码风格规范，以确保每个人的代码结构大致相同。一旦你学会了编写干净的 Python 代码，你应该能够理解任何其他人写的 Python 代码，只要他们遵循相同的规范。如果你希望将来成为一名专业程序员，你应该尽早开始遵循这些规范，养成良好的编程习惯。

### 风格指南

当有人想对 Python 语言做出更改时，他们会写一份*Python 增强提案（PEP）*。最早的 PEP 之一是*PEP 8*，它指导 Python 程序员如何规范代码风格。PEP 8 内容相当长，但其中很多内容涉及到比你当前看到的更复杂的编码结构。

Python 的风格指南写作时考虑到代码比编写的更常被阅读。你编写代码一次后，就会在开始调试时反复阅读它。当你向程序添加新功能时，你会花更多时间阅读代码。当你与其他程序员分享代码时，他们也会阅读你的代码。

在写代码时，如果面临选择更容易写的代码和更容易读的代码之间的抉择，Python 程序员几乎总是会鼓励你写易读的代码。以下规范将帮助你从一开始就编写清晰的代码。

### 缩进

PEP 8 推荐每个缩进级别使用四个空格。使用四个空格可以提高可读性，同时在每一行留出空间以支持多个缩进级别。

在文字处理文档中，人们通常使用制表符而非空格来进行缩进。这对文字处理文档非常有效，但当制表符与空格混合时，Python 解释器会感到困惑。每个文本编辑器都提供一个设置，允许你使用 TAB 键，但会将每个制表符转换为一定数量的空格。你应该使用 TAB 键，但也要确保编辑器设置为将制表符转换为空格，而不是直接插入制表符。

在文件中混合使用制表符和空格可能会导致很难诊断的问题。如果你怀疑文件中混用了制表符和空格，大多数编辑器允许你将文件中的所有制表符转换为空格。

### 行长度

许多 Python 程序员建议每行字符数不应超过 80 个。这个指导方针最初源于大多数计算机只能在终端窗口的单行中显示 79 个字符。现在，人们可以在屏幕上显示更长的行，但仍然有其他原因支持遵循 79 字符的标准行长度。

专业程序员通常会在同一屏幕上打开多个文件，使用标准的行长度使他们能够同时查看屏幕上并排显示的两个或三个文件中的完整行。PEP 8 还建议将所有注释的每行字符数限制为 72 个，因为一些为大型项目生成自动文档的工具会在每行注释的开头添加格式字符。

PEP 8 关于行长度的指导方针并非一成不变，某些团队偏好 99 字符的限制。作为初学者，你不必过于担心代码行的长度，但要注意，协作开发的人员几乎总是遵循 PEP 8 的规范。大多数编辑器允许你设置一个视觉提示，通常是在屏幕上显示一条垂直线，帮助你了解行长度的限制。

### 空白行

为了在视觉上分组程序的各个部分，使用空白行。你应该使用空白行来组织文件，但不要过度使用。通过遵循本书中提供的示例，你应该能够找到合适的平衡。例如，如果你有五行代码用于构建一个列表，接着再有三行代码操作该列表，那么在这两部分之间插入一个空白行是合适的。然而，你不应在这两部分之间插入三到四个空白行。

空白行不会影响代码的运行，但会影响代码的可读性。Python 解释器使用水平缩进来解释代码的含义，但它忽略垂直空白。

### 其他样式指南

PEP 8 提供了许多额外的样式建议，但大多数指导原则适用于比你目前编写的更复杂的程序。当你学习更复杂的 Python 结构时，我会分享 PEP 8 指南中相关的部分。

## 总结

在本章中，你学会了如何高效地处理列表中的元素。你学会了如何使用 `for` 循环遍历列表，Python 如何使用缩进来结构化程序，以及如何避免一些常见的缩进错误。你学会了创建简单的数字列表，以及你可以对数字列表执行的一些操作。你学会了如何切片列表以处理子集项，以及如何通过切片正确地复制列表。你还学习了元组，它提供了一定的保护，防止一组值被改变，以及如何使你越来越复杂的代码具有可读性。

在第五章中，你将学习通过使用 `if` 语句来对不同的条件做出适当的反应。你将学会将相对复杂的条件测试组合起来，以准确应对你所寻找的特定情况或信息。你还将学习在循环遍历列表时使用 `if` 语句，以对列表中的特定元素执行特定操作。

