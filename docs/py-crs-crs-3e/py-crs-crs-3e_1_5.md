
## 第五章：`if` 语句

![](img/chapterart.png)

编程通常涉及检查一组条件，并根据这些条件决定采取什么行动。Python 的 `if` 语句允许你检查程序的当前状态，并对该状态做出适当的响应。

在本章中，你将学习如何编写条件测试，以检查你感兴趣的任何条件。你将学会编写简单的 `if` 语句，还将学习如何创建更复杂的 `if` 语句系列，以识别出你需要的确切条件。然后，你将把这个概念应用到列表中，这样你就能够编写一个 `for` 循环，以一种方式处理列表中的大多数项，但以另一种方式处理具有特定值的某些项。

## 一个简单的例子

以下示例展示了 `if` 测试如何让你正确地响应特殊情况。假设你有一个汽车名称的列表，你想打印出每辆车的名字。汽车名字是专有名词，因此大多数车的名字应该以标题格式打印。然而，`'bmw'` 的值应该以全大写形式打印。以下代码会遍历汽车名称列表，查找值为 `'bmw'` 的项。每当遇到 `'bmw'` 时，它就会以大写形式打印，而不是标题格式：

**cars.py**

```py
cars = ['audi', 'bmw', 'subaru', 'toyota']

for car in cars:
❶     if car == 'bmw':
        print(car.upper())
    else:
        print(car.title())
```

这个示例中的循环首先检查 `car` 的当前值是否为 `'bmw'` ❶。如果是，它将值以大写形式打印。如果 `car` 的值不是 `'bmw'`，它将以标题格式打印：

```py
Audi
BMW
Subaru
Toyota
```

这个示例结合了你将在本章中学习的多个概念。让我们先看看你可以用来检查程序中条件的各种测试。

## 条件测试

每个`if`语句的核心是一个可以被评估为`True`或`False`的表达式，这被称为*条件测试*。Python 使用`True`和`False`的值来决定是否执行`if`语句中的代码。如果条件测试评估为`True`，Python 就会执行`if`语句后的代码。如果测试评估为`False`，Python 则会忽略`if`语句后的代码。

### 检查相等性

大多数条件测试将一个变量的当前值与一个特定的目标值进行比较。最简单的条件测试检查一个变量的值是否等于目标值：

```py
>>> **car = 'bmw'**
>>> **car == 'bmw'**
True
```

第一行使用单等号将`car`的值设置为`'bmw'`，你已经看到了很多次。接下来的一行使用双等号(`==`)检查`car`的值是否为`'bmw'`。这个*相等运算符*如果左右两边的值相同，则返回`True`，如果不同，则返回`False`。在这个例子中，两个值是匹配的，所以 Python 返回`True`。

当`car`的值不是`'bmw'`时，这个测试返回`False`：

```py
>>> **car = 'audi'**
>>> **car == 'bmw'**
False
```

一个等号实际上是一个赋值语句；你可以把这里的第一行代码读作“将`car`的值设置为`'audi'`。”另一方面，双等号是在问一个问题：“`car`的值是否等于`'bmw'`？”大多数编程语言都以这种方式使用等号。

### 检查相等性时忽略大小写

在 Python 中，检查相等性是区分大小写的。例如，两个大小写不同的值不会被认为是相等的：

```py
>>> **car = 'Audi'**
>>> **car == 'audi'**
False
```

如果大小写很重要，这种行为是有利的。但是，如果大小写不重要，而你只想测试变量的值，可以在进行比较之前将变量的值转换为小写：

```py
>>> **car = 'Audi'**
>>> **car.lower() == 'audi'**
True
```

这个测试会返回`True`，无论`'Audi'`的格式如何，因为现在测试是大小写不敏感的。`lower()`方法不会改变最初存储在`car`中的值，因此你可以在不影响原始变量的情况下进行这种比较：

```py
>>> **car = 'Audi'**
>>> **car.lower() == 'audi'**
True
>>> **car**
'Audi'
```

我们首先将大写字符串`'Audi'`赋值给变量`car`。然后，我们将`car`的值转换为小写，并将小写值与字符串`'audi'`进行比较。两个字符串匹配，所以 Python 返回`True`。我们可以看到，存储在`car`中的值没有受到`lower()`方法的影响。

网站会像这样强制执行用户输入数据的某些规则。例如，一个网站可能会使用类似的条件测试来确保每个用户都有一个真正唯一的用户名，而不仅仅是另一个人用户名的大小写变化。当有人提交一个新用户名时，这个新用户名会被转换为小写，并与所有现有用户名的小写版本进行比较。在这个检查过程中，如果`'John'`的任何变体（如`'john'`）已经被使用，新的用户名将被拒绝。

### 检查不相等性

当你想确定两个值是否不相等时，可以使用*不等运算符*（`!=`）。我们再用一个`if`语句来演示如何使用不等运算符。我们将把请求的披萨配料存储在一个变量中，然后如果这个人没有点凤尾鱼，就打印一条消息：

**toppings.py**

```py
requested_topping = 'mushrooms'

if requested_topping != 'anchovies':
    print("Hold the anchovies!")
```

这段代码将`requested_topping`的值与`'anchovies'`进行比较。如果这两个值不匹配，Python 返回`True`并执行`if`语句后的代码。如果两个值匹配，Python 返回`False`，并且不会执行`if`语句后的代码。

由于`requested_topping`的值不是`'anchovies'`，因此执行了`print()`函数：

```py
Hold the anchovies!
```

你编写的大多数条件表达式都会测试相等性，但有时你会发现测试不等式更高效。

### 数值比较

测试数值是相当简单的。例如，下面的代码检查一个人是否已经 18 岁：

```py
>>> **age = 18**
>>> **age == 18**
True
```

你也可以测试两个数字是否不相等。例如，下面的代码如果给定的答案不正确，就会打印一条消息：

**magic_number.py**

```py
answer = 17
if answer != 42:
    print("That is not the correct answer. Please try again!")
```

条件测试通过，因为`answer`（`17`）的值不等于`42`。由于测试通过，缩进的代码块被执行：

```py
That is not the correct answer. Please try again!
```

你还可以在条件语句中包含各种数学比较，例如小于、小于或等于、大于、大于或等于：

```py
>>> **age = 19**
>>> **age < 21**
True
>>> **age <= 21**
True
>>> **age > 21**
False
>>> **age >= 21**
False
```

每个数学比较都可以作为`if`语句的一部分，这有助于你检测感兴趣的具体条件。

### 检查多个条件

你可能希望同时检查多个条件。例如，有时你可能需要两个条件都为`True`才能采取某个动作。其他时候，你可能只需要一个条件为`True`就满足了。`and`和`or`关键字可以在这些情况下帮到你。

#### 使用`and`检查多个条件

要检查两个条件是否同时都为`True`，可以使用`and`关键字将两个条件测试结合起来；如果每个测试都通过，整体表达式就为`True`。如果任何一个测试失败，或者两个测试都失败，表达式就为`False`。

例如，你可以使用以下测试来检查两个人是否都超过 21 岁：

```py
>>> **age_0 = 22**
>>> **age_1 = 18**
❶ >>> **age_0 >= 21 and age_1 >= 21**
False
❷ >>> **age_1 = 22**
>>> **age_0 >= 21 and age_1 >= 21**
True
```

首先，我们定义两个年龄，`age_0`和`age_1`。然后我们检查这两个年龄是否都大于或等于 21 岁❶。左边的测试通过了，但右边的测试失败了，因此整个条件表达式的结果为`False`。然后我们将`age_1`改为 22❷。`age_1`的值现在大于 21，因此两个单独的测试都通过了，导致整个条件表达式的结果为`True`。

为了提高可读性，你可以在单独的测试条件周围使用括号，但这不是必需的。如果使用括号，测试将像这样：

```py
(age_0 >= 21) and (age_1 >= 21)
```

#### 使用`or`检查多个条件

关键字 `or` 允许你检查多个条件，但只要其中一个或两个测试通过，它就会通过。`or` 表达式只有在两个测试都失败时才会失败。

让我们再次考虑两个年龄，但这次我们只关注是否有一个人超过 21 岁：

```py
>>> **age_0 = 22**
>>> **age_1 = 18**
❶ >>> **age_0 >= 21 or age_1 >= 21**
True
❷ >>> **age_0 = 18**
>>> **age_0 >= 21 or age_1 >= 21**
False
```

我们再次开始使用两个年龄变量。由于对 `age_0` ❶ 的测试通过，整个表达式评估为 `True`。然后我们将 `age_0` 降到 18。在最后的测试 ❷ 中，两个测试都失败，整个表达式评估为 `False`。

### 检查值是否在列表中

有时，在执行某个操作之前，检查列表是否包含某个值很重要。例如，在完成某人注册之前，你可能想检查一个新的用户名是否已经存在于当前的用户名列表中。在一个制图项目中，你可能想检查一个提交的地点是否已经存在于已知地点列表中。

要检查某个特定值是否已经在列表中，可以使用关键字 `in`。让我们考虑一些你可能为披萨店编写的代码。我们将创建一个顾客请求的披萨配料列表，然后检查某些配料是否在该列表中。

```py
>>> **requested_toppings = ['mushrooms', 'onions', 'pineapple']**
>>> **'mushrooms' in requested_toppings**
True
>>> **'pepperoni' in requested_toppings**
False
```

关键字 `in` 告诉 Python 检查 `'mushrooms'` 和 `'pepperoni'` 是否存在于列表 `requested_toppings` 中。这种技术非常强大，因为你可以创建一个包含基本值的列表，然后轻松检查你正在测试的值是否与列表中的某个值匹配。

### 检查值是否不在列表中

有时，了解某个值是否不存在于列表中也很重要。在这种情况下，你可以使用关键字 `not`。例如，考虑一个在论坛中被禁止评论的用户列表。你可以在允许用户提交评论之前，检查该用户是否已被禁止：

**banned_users.py**

```py
banned_users = ['andrew', 'carolina', 'david']
user = 'marie'

if user not in banned_users:
    print(f"{user.title()}, you can post a response if you wish.")
```

这里的 `if` 语句很清晰。如果 `user` 的值不在 `banned_users` 列表中，Python 返回 `True` 并执行缩进的语句。

用户 `'marie'` 不在 `banned_users` 列表中，因此她会看到一条邀请她发布回应的消息：

```py
Marie, you can post a response if you wish.
```

### 布尔表达式

当你进一步学习编程时，你会在某个时刻听到 *布尔表达式* 这个术语。布尔表达式就是条件测试的另一种说法。*布尔值* 只有 `True` 或 `False`，就像条件表达式在评估后的值一样。

布尔值常用于跟踪某些条件，例如游戏是否正在运行，或用户是否可以编辑网站上的某些内容：

```py
game_active = True
can_edit = False
```

布尔值提供了一种高效的方式来跟踪程序的状态或程序中某个重要条件的状态。

## if 语句

当你理解了条件测试后，你可以开始编写 `if` 语句。存在几种不同类型的 `if` 语句，你选择使用哪种类型取决于你需要测试的条件数量。在关于条件测试的讨论中，你已经看到了几个 `if` 语句的例子，现在让我们更深入地探讨这个话题。

### 简单的 if 语句

最简单的 `if` 语句类型有一个测试和一个操作：

```py
if `conditional_test`:
    `do something`
```

你可以在第一行中放置任何条件测试，并且在测试后面的缩进块中放置几乎任何操作。如果条件测试的结果为 `True`，Python 将执行 `if` 语句后的代码。如果测试结果为 `False`，Python 会忽略 `if` 语句后的代码。

假设我们有一个表示某人年龄的变量，我们想知道这个人是否足够大，可以投票。以下代码测试该人是否可以投票：

**voting.py**

```py
age = 19
if age >= 18:
    print("You are old enough to vote!")
```

Python 检查 `age` 的值是否大于或等于 18。它是成立的，因此 Python 执行缩进的 `print()` 调用：

```py
You are old enough to vote!
```

缩进在 `if` 语句中的作用与在 `for` 循环中相同。`if` 语句后的所有缩进行将会在测试通过时执行，如果测试不通过，则整个缩进块会被忽略。

你可以在 `if` 语句后的块中放置任意多行代码。如果该人足够大，可以投票，我们可以添加另一行输出，询问该人是否已经注册投票：

```py
age = 19
if age >= 18:
 print("You are old enough to vote!")
    print("Have you registered to vote yet?")
```

条件测试通过，且两个 `print()` 调用都已缩进，因此两行都被打印出来：

```py
You are old enough to vote!
Have you registered to vote yet?
```

如果 `age` 的值小于 18，则此程序将不会产生任何输出。

### if-else 语句

通常，你希望在条件测试通过时执行某个操作，在所有其他情况下执行不同的操作。Python 的 `if`-`else` 语法使得这一点成为可能。`if`-`else` 块类似于简单的 `if` 语句，但 `else` 语句允许你定义在条件测试失败时执行的操作或操作集。

如果一个人足够大可以投票，我们将显示与之前相同的消息，但这次我们还会为那些不够大不能投票的人添加一条消息：

```py
age = 17
❶ if age >= 18:
 print("You are old enough to vote!")
 print("Have you registered to vote yet?")
❷ else:
    print("Sorry, you are too young to vote.")
    print("Please register to vote as soon as you turn 18!")
```

如果条件测试 ❶ 通过，则执行第一个缩进的 `print()` 调用块。如果测试结果为 `False`，则执行 `else` 块 ❷。因为这次 `age` 小于 18，条件测试失败，执行 `else` 块中的代码：

```py
Sorry, you are too young to vote.
Please register to vote as soon as you turn 18!
```

这段代码有效，因为它只有两种可能的情况：一个人要么足够大可以投票，要么不够大不能投票。`if`-`else` 结构在你希望 Python 总是执行两种可能操作之一的情况下工作得很好。在像这样的简单 `if-else` 链中，总会执行两种操作中的一种。

### if-elif-else 链

通常，你需要测试两个以上的可能情况，而要评估这些情况，你可以使用 Python 的`if`-`elif`-`else`语法。Python 只会在`if`-`elif`-`else`链中执行一个代码块。它按顺序运行每个条件测试，直到某个测试通过。当某个测试通过时，紧跟着该测试的代码会被执行，然后 Python 会跳过其余的测试。

许多现实世界的情况涉及两个以上的可能条件。例如，考虑一个根据不同年龄段收取不同费用的游乐园：

+   任何未满 4 岁的人可以免费入场。

+   任何 4 岁至 18 岁之间的人入场费用为$25。

+   任何 18 岁或以上的人入场费用为$40。

我们如何使用`if`语句来确定一个人的入场费用？以下代码测试一个人的年龄段，然后打印入场价格消息：

**amusement_park.py**

```py
age = 12
❶ if age < 4:
    print("Your admission cost is $0.")
❷ elif age < 18:
    print("Your admission cost is $25.")
❸ else:
    print("Your admission cost is $40.")
```

`if`测试❶检查一个人是否未满 4 岁。当测试通过时，会打印相应的消息，Python 跳过剩余的测试。`elif`行❷实际上是另一个`if`测试，只有在前一个测试失败时才会执行。在链中的这一点，我们知道这个人至少 4 岁，因为第一个测试失败了。如果这个人未满 18 岁，会打印相应的消息，Python 跳过`else`块。如果`if`和`elif`测试都失败，Python 会执行`else`块中的代码❸。

在这个例子中，`if`测试❶的结果为`False`，因此它的代码块没有被执行。然而，`elif`测试的结果为`True`（12 小于 18），因此它的代码被执行。输出是一个句子，通知用户入场费用：

```py
Your admission cost is $25.
```

任何大于 17 岁的年龄都会导致前两个测试失败。在这些情况下，`else`块会被执行，入场费用为$40。

与其在`if`-`elif`-`else`块中打印入场价格，不如在`if`-`elif`-`else`链中仅设置价格，然后在链评估后执行一个单独的`print()`调用，这样更简洁：

```py
age = 12

if age < 4:
    price = 0
elif age < 18:
    price = 25
else:
    price = 40

print(f"Your admission cost is ${price}.")
```

缩进的行根据人的年龄设置`price`的值，就像前一个示例一样。在`if`-`elif`-`else`链设置了价格后，一个独立的未缩进的`print()`调用使用这个值来显示一条消息，报告这个人的入场价格。

这段代码产生与前一个示例相同的输出，但`if`-`elif`-`else`链的作用更窄。它不是用来确定价格并显示消息，而是简单地确定入场价格。除了更高效外，修改后的代码也比原始方法更容易修改。如果要更改输出消息的文本，只需要修改一个`print()`调用，而不是三个单独的`print()`调用。

### 使用多个`elif`块

你可以在代码中使用任意多个`elif`块。例如，如果游乐园实施了老年人折扣，你可以在代码中添加一个条件测试来确定某人是否符合老年人折扣的条件。假设 65 岁或以上的人只需支付常规入场费的一半，即 20 美元：

```py
age = 12

if age < 4:
 price = 0
elif age < 18:
 price = 25
elif age < 65:
    price = 40
else:
    price = 20

print(f"Your admission cost is ${price}.")
```

这段代码的大部分内容保持不变。第二个`elif`块现在会检查确保一个人的年龄小于 65 岁，然后才会将其分配为 40 美元的全额入场费。请注意，`else`块中分配的值需要改为 20 美元，因为只有年龄在 65 岁及以上的人才会进入这个块。

### 忽略`else`块

Python 并不要求在`if`-`elif`链的末尾必须有`else`块。有时，`else`块是有用的。其他时候，使用额外的`elif`语句来捕获感兴趣的特定条件会更清晰：

```py
age = 12

if age < 4:
 price = 0
elif age < 18:
 price = 25
elif age < 65:
 price = 40
elif age >= 65:
    price = 20

print(f"Your admission cost is ${price}.")
```

最后的`elif`块在一个人 65 岁或以上时分配 20 美元的价格，这比一般的`else`块更清晰。通过这个变化，每一块代码必须通过一个特定的测试才能执行。

`else`块是一个兜底语句。它匹配任何没有被特定`if`或`elif`测试匹配的条件，这有时可能包括无效或甚至恶意的数据。如果你有一个具体的最终条件要测试，可以考虑使用最后一个`elif`块，并省略`else`块。这样，你会更有信心代码只会在正确的条件下运行。

### 测试多个条件

`if`-`elif`-`else`链非常强大，但仅在你只需要通过一个测试时才适合使用。一旦 Python 找到一个通过的测试，它就会跳过剩下的测试。这种行为是有益的，因为它高效，并且允许你仅测试一个特定的条件。

然而，有时检查所有感兴趣的条件是很重要的。在这种情况下，你应该使用一系列简单的`if`语句，而不使用`elif`或`else`块。当多个条件可能为`True`时，这种技术很有意义，并且你希望对每个为`True`的条件做出响应。

让我们重新考虑披萨店的例子。如果有人请求一个双配料的披萨，你需要确保披萨上有两个配料：

**toppings.py**

```py
requested_toppings = ['mushrooms', 'extra cheese']

if 'mushrooms' in requested_toppings:
    print("Adding mushrooms.")
❶ if 'pepperoni' in requested_toppings:
    print("Adding pepperoni.")
if 'extra cheese' in requested_toppings:
    print("Adding extra cheese.")

print("\nFinished making your pizza!")
```

我们从一个包含请求的配料的列表开始。第一个`if`语句检查这个人是否请求了蘑菇。如果是这样，则打印一条消息确认这个配料。辣香肠的测试❶是另一个简单的`if`语句，而不是`elif`或`else`语句，因此无论前面的测试是否通过，这个测试都会被执行。最后一个`if`语句检查是否请求了额外的奶酪，无论前两个测试的结果如何。这三个独立的测试每次运行这个程序时都会执行。

因为在这个例子中每个条件都会被评估，所以蘑菇和额外的奶酪都会被添加到披萨上：

```py
Adding mushrooms.
Adding extra cheese.

Finished making your pizza!
```

如果我们使用`if`-`elif`-`else`块，这段代码将无法正常工作，因为代码在只有一个测试通过后就会停止运行。下面是它的样子：

```py
requested_toppings = ['mushrooms', 'extra cheese']

if 'mushrooms' in requested_toppings:
 print("Adding mushrooms.")
elif 'pepperoni' in requested_toppings:
    print("Adding pepperoni.")
elif 'extra cheese' in requested_toppings:
    print("Adding extra cheese.")

print("\nFinished making your pizza!")
```

对`'mushrooms'`的测试是第一个通过的测试，所以蘑菇被添加到比萨上。然而，值`'extra cheese'`和`'pepperoni'`永远不会被检查，因为在`if-elif-else`链中，Python 不会运行通过的第一个测试后的任何其他测试。顾客的第一个配料会被添加，但所有其他配料都会被遗漏：

```py
Adding mushrooms.

Finished making your pizza!
```

总结来说，如果你希望只有一个代码块运行，使用`if`-`elif`-`else`链。如果需要多个代码块运行，使用一系列独立的`if`语句。

## 在列表中使用`if`语句

当你将列表和`if`语句结合起来时，可以做一些有趣的工作。你可以观察需要与列表中其他值不同对待的特殊值。你可以有效地管理变化的条件，比如餐厅在一个班次中的某些物品的可用性。你还可以开始证明你的代码在所有可能的情况下都能按预期工作。

### 检查特殊项

本章开始时展示了一个简单的例子，演示了如何处理像`'bmw'`这样的特殊值，特别是它需要以不同于其他列表值的格式打印。现在你已经对条件测试和`if`语句有了基本的理解，我们来仔细看看如何在列表中检查特殊值并适当地处理这些值。

让我们继续以比萨店的例子。每当在制作比萨时添加配料，比萨店会显示一条消息。这个操作的代码可以通过列出顾客请求的配料，并使用循环在每次配料添加到比萨上时宣布它，来高效地编写：

**toppings.py**

```py
requested_toppings = ['mushrooms', 'green peppers', 'extra cheese']

for requested_topping in requested_toppings:
    print(f"Adding {requested_topping}.")

print("\nFinished making your pizza!")
```

输出是直观的，因为这段代码只是一个简单的`for`循环：

```py
Adding mushrooms.
Adding green peppers.
Adding extra cheese.

Finished making your pizza!
```

那如果比萨店没有绿椒了怎么办？`for`循环中的`if`语句可以适当地处理这种情况：

```py
requested_toppings = ['mushrooms', 'green peppers', 'extra cheese']

for requested_topping in requested_toppings:
    if requested_topping == 'green peppers':
        print("Sorry, we are out of green peppers right now.")
    else:
        print(f"Adding {requested_topping}.")

print("\nFinished making your pizza!")
```

这次，在将配料添加到比萨之前，我们检查每个请求的项。`if`语句检查顾客是否请求了绿椒。如果是，我们会显示一条消息告知他们为什么不能要绿椒。`else`块确保所有其他配料都将添加到比萨上。

输出显示每个请求的配料都得到了适当的处理。

```py
Adding mushrooms.
Sorry, we are out of green peppers right now.
Adding extra cheese.

Finished making your pizza!
```

### 检查列表是否为空

到目前为止，我们对每个使用过的列表做了一个简单的假设：我们假设每个列表至少有一个项目。很快，我们将允许用户提供存储在列表中的信息，因此每次运行循环时，我们不能假设列表中有任何项目。在这种情况下，在运行`for`循环之前检查列表是否为空是很有用的。

作为示例，让我们在制作披萨之前检查请求的配料列表是否为空。如果列表为空，我们将提示用户并确认他们是否想要一份普通披萨。如果列表不为空，我们将像之前的示例一样制作披萨：

```py
requested_toppings = []

if requested_toppings:
    for requested_topping in requested_toppings:
 print(f"Adding {requested_topping}.")
 print("\nFinished making your pizza!")
else:
    print("Are you sure you want a plain pizza?")
```

这次我们从一个空的请求配料列表开始。我们没有直接进入 `for` 循环，而是先做一个快速检查。当列表的名称在 `if` 语句中使用时，如果列表包含至少一个项，Python 会返回 `True`；如果是空列表，则返回 `False`。如果 `requested_toppings` 通过条件测试，我们就运行与前一个示例中相同的 `for` 循环。如果条件测试失败，我们会打印一条消息，询问顾客是否真的想要没有配料的普通披萨。

在这种情况下，列表为空，因此输出会询问用户是否真的想要一份普通披萨：

```py
Are you sure you want a plain pizza?
```

如果列表不为空，输出将显示每个请求的配料被添加到披萨上。

### 使用多个列表

人们对配料的请求几乎无所不包，特别是涉及披萨配料时。如果顾客真的想要在披萨上放薯条呢？你可以使用列表和 `if` 语句，确保在执行操作之前输入是合理的。

在制作披萨之前，让我们留意一些不寻常的配料请求。以下示例定义了两个列表。第一个是披萨店提供的配料列表，第二个是用户请求的配料列表。这一次，`requested_toppings` 中的每个项在添加到披萨之前都会与可用配料列表进行检查：

```py
available_toppings = ['mushrooms', 'olives', 'green peppers',
                      'pepperoni', 'pineapple', 'extra cheese']

❶ requested_toppings = ['mushrooms', 'french fries', 'extra cheese']

for requested_topping in requested_toppings:
❷     if requested_topping in available_toppings:
        print(f"Adding {requested_topping}.")
❸     else:
        print(f"Sorry, we don't have {requested_topping}.")

print("\nFinished making your pizza!")
```

首先，我们定义了这家披萨店提供的配料列表。请注意，如果披萨店提供稳定的配料选择，这个可以是一个元组。然后，我们创建了一个客户请求的配料列表。在这个示例中，有一个不寻常的配料请求：`'french fries'` ❶。接下来，我们遍历请求的配料列表。在循环内，我们检查每个请求的配料是否实际上在可用配料列表中 ❷。如果有，我们就将该配料添加到披萨上。如果请求的配料不在可用配料列表中，`else` 块将执行 ❸。`else` 块会打印一条消息，告诉用户哪些配料不可用。

这段代码语法产生了干净且富有信息的输出：

```py
Adding mushrooms.
Sorry, we don't have french fries.
Adding extra cheese.

Finished making your pizza!
```

仅凭几行代码，我们就有效地处理了一个现实世界中的情况！

## 给你的 `if` 语句添加样式

在本章的每个示例中，你都看到了良好的样式习惯。PEP 8 对条件测试样式的唯一建议是，在比较运算符（如 `==`、`>=` 和 `<=`）周围使用单个空格。例如：

```py
if age < 4:
```

比下面这种写法更好：

```py
if age<4:
```

这样的空格不会影响 Python 解释代码的方式；它只是让你的代码更容易阅读，无论是你自己还是其他人。

## 总结

在本章中，你学习了如何编写条件测试，这些测试总是返回`True`或`False`。你学习了如何编写简单的`if`语句、`if`-`else`链式结构以及`if`-`elif`-`else`链式结构。你开始使用这些结构来识别你需要测试的特定条件，并知道在你的程序中何时满足这些条件。你学会了如何在列表中处理某些项目与其他项目不同，同时继续利用`for`循环的高效性。你还重新回顾了 Python 的编码风格建议，以确保你日益复杂的程序仍然相对容易阅读和理解。

在第六章中，你将了解 Python 的字典。字典类似于列表，但它允许你将信息片段连接在一起。你将学习如何构建字典、遍历字典，并将它们与列表和`if`语句结合使用。学习字典将使你能够建模更广泛的现实世界情况。

