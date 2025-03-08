
## 第七章：用户输入与 while 循环

![](img/chapterart.png)

大多数程序是为了解决最终用户的问题而编写的。为此，你通常需要从用户那里获取一些信息。例如，假设有人想知道自己是否足够老，可以投票。如果你编写一个程序来回答这个问题，你需要知道用户的年龄，才能给出答案。程序将要求用户输入或*提供*他们的年龄；一旦程序获得了这个输入，它就可以将其与投票年龄进行比较，以判断用户是否足够老，然后报告结果。

在这一章中，你将学习如何接受用户输入，以便你的程序可以使用这些输入。当程序需要一个名字时，你可以提示用户输入名字。当程序需要一系列名字时，你也可以提示用户输入多个名字。为此，你将使用`input()`函数。

你还将学习如何让程序持续运行，直到用户希望它停止，这样他们可以输入所需的所有信息；然后，你的程序可以利用这些信息进行处理。你将使用 Python 的`while`循环来让程序在特定条件成立时持续运行。

通过能够处理用户输入以及控制程序运行时间的能力，你将能够编写完全互动的程序。

## `input()`函数的工作原理

`input()`函数会暂停程序并等待用户输入文本。一旦 Python 接收到用户的输入，它会将输入赋值给一个变量，以便你方便地使用它。

例如，下面的程序会要求用户输入一些文本，然后将该消息显示给用户：

**parrot.py**

```py
message = input("Tell me something, and I will repeat it back to you: ")
print(message)
```

`input()`函数接受一个参数：*提示信息*，用于告诉用户需要输入什么类型的信息。在这个例子中，当 Python 运行第一行时，用户会看到提示信息`Tell me something, and I will repeat it back to you:`。程序在等待用户输入响应，用户按下回车键后，程序继续执行。响应被赋值给变量`message`，然后`print(message)`会将输入内容显示给用户：

```py
Tell me something, and I will repeat it back to you: **Hello everyone!**
Hello everyone!
```

### 编写清晰的提示信息

每次使用`input()`函数时，你都应该包含一个清晰、易于理解的提示，告诉用户你需要什么类型的信息。任何能告诉用户需要输入什么内容的语句都可以。例如：

**greeter.py**

```py
name = input("Please enter your name: ")
print(f"\nHello, {name}!")
```

在提示信息末尾添加一个空格（在前面的示例中冒号后面）来将提示与用户的响应分开，并且清晰地告诉用户应该在何处输入他们的文本。例如：

```py
Please enter your name: **Eric**
Hello, Eric!
```

有时候你可能需要编写一个超过一行的提示信息。例如，你可能想告诉用户你为何要求提供某些输入。你可以将提示信息赋值给一个变量，并将该变量传递给`input()`函数。这使得你能够在多行中构建提示信息，然后编写一个简洁的`input()`语句。

**greeter.py**

```py
prompt = "If you share your name, we can personalize the messages you see."
prompt += "\nWhat is your first name? "

name = input(prompt)
print(f"\nHello, {name}!")
```

这个例子展示了构建多行字符串的一种方式。第一行将消息的第一部分赋值给变量 `prompt`。在第二行，运算符 `+=` 将赋值给 `prompt` 的字符串与新的字符串连接起来。

现在提示符跨越了两行，再次在问号后留有空格以增强清晰度：

```py
If you share your name, we can personalize the messages you see.
What is your first name? **Eric**

Hello, Eric!
```

### 使用 `int()` 接受数值输入

当你使用 `input()` 函数时，Python 将用户输入的所有内容都视为字符串。考虑以下的解释器会话，它询问用户的年龄：

```py
>>> **age = input("How old are you? ")**
How old are you? **21**
>>> **age**
'21'
```

用户输入数字 `21`，但是当我们请求 Python 获取 `age` 的值时，它返回了 `'21'`，即输入的数值的字符串表示。我们知道 Python 将输入解释为字符串，因为该数字现在被引号括起来。如果你只是想打印输入内容，这样没问题。但如果你尝试将输入作为数字使用，你将遇到错误：

```py
>>> **age = input("How old are you? ")**
How old are you? **21**
❶ >>> **age >= 18**
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
❷ TypeError: '>=' not supported between instances of 'str' and 'int'
```

当你尝试使用输入进行数值比较 ❶ 时，Python 会产生错误，因为它不能将字符串与整数进行比较：分配给 `age` 的字符串 `'21'` 不能与数值 `18` ❷ 进行比较。

我们可以通过使用 `int()` 函数来解决这个问题，它将输入的字符串转换为数值。这使得比较能够成功执行：

```py
>>> **age = input("How old are you? ")**
How old are you? **21**
❶ >>> **age = int(age)**
>>> **age >= 18**
True
```

在这个例子中，当我们在提示符下输入 `21` 时，Python 将该数字解释为字符串，但该值随后通过 `int()` ❶ 被转换为数值表示。现在，Python 可以执行条件测试：它将 `age`（现在表示数值 21）与 `18` 进行比较，判断 `age` 是否大于或等于 18。这个测试的结果是 `True`。

如何在实际程序中使用 `int()` 函数？考虑一个程序，判断人们是否足够高，能够坐过山车：

**rollercoaster.py**

```py
height = input("How tall are you, in inches? ")
height = int(height)

if height >= 48:
    print("\nYou're tall enough to ride!")
else:
    print("\nYou'll be able to ride when you're a little older.")
```

该程序可以将 `height` 与 `48` 进行比较，因为 `height = int(height)` 在进行比较之前将输入值转换为数值表示。如果输入的数字大于或等于 48，我们会告诉用户他们的身高足够高：

```py
How tall are you, in inches? **71**

You're tall enough to ride!
```

当你使用数值输入进行计算和比较时，请务必先将输入值转换为数值表示。

### 取余运算符

处理数值信息时，一个有用的工具是 *取余运算符*（`%`），它将一个数字除以另一个数字并返回余数：

```py
>>> **4 % 3**
1
>>> **5 % 3**
2
>>> **6 % 3**
0
>>> **7 % 3**
1
```

取余运算符不会告诉你一个数字能被另一个数字整除多少次；它只告诉你余数是多少。

当一个数字能被另一个数字整除时，余数为 0，因此取余运算符总是返回 0。你可以利用这一点来判断一个数字是偶数还是奇数：

**even_or_odd.py**

```py
number = input("Enter a number, and I'll tell you if it's even or odd: ")
number = int(number)

if number % 2 == 0:
    print(f"\nThe number {number} is even.")
else:
    print(f"\nThe number {number} is odd.")
```

偶数总是可以被二整除，因此如果一个数字与 2 的取余为零（在这里，`if number % 2 == 0`），则该数字是偶数。否则，它是奇数。

```py
Enter a number, and I'll tell you if it's even or odd: **42**

The number 42 is even.
```

## 引入 while 循环

`for`循环会遍历一个集合中的每个项，并为集合中的每个项执行一次代码块。相比之下，`while`循环则是只要某个条件为真，就继续运行。

### `while`循环的实际运行

你可以使用`while`循环来通过一系列数字进行计数。例如，以下`while`循环从 1 计数到 5：

**counting.py**

```py
current_number = 1
while current_number <= 5:
 print(current_number)
    current_number += 1
```

在第一行中，我们通过将`current_number`赋值为 1 来开始计数。然后，`while`循环被设置为在`current_number`的值小于或等于 5 时持续运行。循环中的代码打印`current_number`的值，然后使用`current_number += 1`将该值加 1。（`+=`运算符是`current_number = current_number + 1`的简写。）

只要`current_number <= 5`的条件为真，Python 就会重复执行循环。因为 1 小于 5，所以 Python 打印`1`，然后加 1，使当前数字变为`2`。因为 2 小于 5，所以 Python 打印`2`，再次加 1，使当前数字变为`3`，依此类推。当`current_number`的值大于 5 时，循环停止，程序结束：

```py
1
2
3
4
5
```

你每天使用的程序很可能包含`while`循环。例如，一个游戏需要使用`while`循环来保持运行，直到你想停止为止，这样它就能在你要求退出时停止运行。如果程序在我们要求退出之前就停止运行，或者在我们想退出后仍然运行，那就不好玩了，所以`while`循环非常有用。

### 让用户选择何时退出

我们可以通过将大部分程序放入`while`循环中，让*parrot.py*程序运行直到用户想退出为止。我们将定义一个*退出值*，然后只要用户没有输入退出值，就继续运行程序：

**parrot.py**

```py
prompt = "\nTell me something, and I will repeat it back to you:"
prompt += "\nEnter 'quit' to end the program. "

message = ""
while message != 'quit':
    message = input(prompt)
    print(message)
```

我们首先定义一个提示，告诉用户他们的两个选项：输入一条信息或输入退出值（在此案例中为`'quit'`）。然后，我们设置一个变量`message`来跟踪用户输入的值。我们将`message`定义为空字符串`""`，这样 Python 在第一次进入`while`语句时会有东西可以检查。当程序第一次运行并且 Python 到达`while`语句时，它需要将`message`的值与`'quit'`进行比较，但此时用户尚未输入任何内容。如果 Python 没有东西可以比较，它将无法继续执行程序。为了解决这个问题，我们确保给`message`一个初始值。虽然它只是一个空字符串，但它对 Python 是有意义的，并且可以让 Python 进行比较，从而使`while`循环得以正常工作。这个`while`循环会在`message`的值不是`'quit'`时继续运行。

在第一次循环时，`message`只是一个空字符串，因此 Python 进入循环。在`message = input(prompt)`时，Python 显示提示并等待用户输入。当用户输入的内容被赋值给`message`并打印后，Python 会重新评估`while`语句中的条件。只要用户没有输入`'quit'`，提示就会再次显示，Python 会等待更多输入。当用户最终输入`'quit'`时，Python 会停止执行`while`循环，程序结束：

```py
Tell me something, and I will repeat it back to you:
Enter 'quit' to end the program. **Hello everyone!**
Hello everyone!

Tell me something, and I will repeat it back to you:
Enter 'quit' to end the program. **Hello again.**
Hello again.

Tell me something, and I will repeat it back to you:
Enter 'quit' to end the program. **quit**
quit
```

这个程序运行良好，唯一的问题是它像打印一个实际的消息一样打印了单词`'quit'`。一个简单的`if`测试就能解决这个问题：

```py
prompt = "\nTell me something, and I will repeat it back to you:"
prompt += "\nEnter 'quit' to end the program. "

message = ""
while message != 'quit':
 message = input(prompt)

    if message != 'quit':
        print(message)
```

现在程序在显示消息之前进行快速检查，只有在消息与退出值不匹配时才会打印该消息：

```py
Tell me something, and I will repeat it back to you:
Enter 'quit' to end the program. **Hello everyone!**
Hello everyone!

Tell me something, and I will repeat it back to you:
Enter 'quit' to end the program. **Hello again.**
Hello again.

Tell me something, and I will repeat it back to you:
Enter 'quit' to end the program. **quit**
```

### 使用标志

在前面的例子中，我们让程序在给定条件为真时执行某些任务。但如果是更复杂的程序，其中许多不同的事件可能导致程序停止运行呢？

例如，在游戏中，几个不同的事件可能会结束游戏。当玩家的舰船耗尽、时间用完，或者他们应该保护的城市全部被摧毁时，游戏应该结束。只要发生了其中任何一个事件，游戏就应结束。如果有很多可能的事件可能导致程序停止，尝试在一个`while`语句中测试所有这些条件会变得复杂且困难。

对于一个应在多个条件都为真时才运行的程序，你可以定义一个变量来决定程序是否继续运行。这个变量被称为*标志*，它充当程序的信号。我们可以编写程序，使其在标志被设置为`True`时运行，并在多个事件之一将标志的值设置为`False`时停止运行。结果，我们的`while`语句只需要检查一个条件：标志是否当前为`True`。然后，所有其他的测试（检查是否发生了某个事件应该将标志设置为`False`）可以在程序的其余部分整齐地组织。

让我们在前一节的*parrot.py*程序中添加一个标志。这个标志，我们称之为`active`（当然你也可以用其他名称），将监控程序是否应该继续运行：

```py
prompt = "\nTell me something, and I will repeat it back to you:"
prompt += "\nEnter 'quit' to end the program. "

active = True
❶ while active:
    message = input(prompt)

    if message == 'quit':
        active = False
    else:
        print(message)
```

我们将变量`active`设置为`True`，以便程序在活动状态下启动。这样，`while`语句就变得更简洁，因为在`while`语句本身没有进行任何比较；逻辑被转移到程序的其他部分。只要`active`变量保持`True`，循环就会继续运行❶。

在`while`循环中的`if`语句里，我们在用户输入后检查`message`的值。如果用户输入`'quit'`，我们将`active`设置为`False`，然后`while`循环停止。如果用户输入除`'quit'`以外的任何内容，我们将他们的输入打印为消息。

这个程序与之前的示例有相同的输出，后者是将条件测试直接放在`while`语句中。但现在，由于我们有一个标志来指示整体程序是否处于活动状态，因此可以很容易地添加更多的测试（例如`elif`语句），用于处理应使`active`变为`False`的事件。这在复杂的程序中很有用，比如游戏，在这些程序中可能有许多事件应该导致程序停止运行。当这些事件中的任何一个导致活动标志变为`False`时，主游戏循环将退出，可以显示*游戏结束*消息，并且玩家可以选择重新开始游戏。

### 使用 break 退出循环

要立即退出`while`循环而不运行循环中剩余的任何代码（无论条件测试的结果如何），请使用`break`语句。`break`语句控制程序的流程；你可以使用它来控制哪些代码行被执行，哪些不被执行，从而让程序仅在你希望的时刻执行你想要的代码。

例如，考虑一个询问用户曾经去过哪些地方的程序。我们可以通过在用户输入`'quit'`值时调用`break`来停止此程序中的`while`循环：

**cities.py**

```py
prompt = "\nPlease enter the name of a city you have visited:"
prompt += "\n(Enter 'quit' when you are finished.) "

❶ while True:
    city = input(prompt)

    if city == 'quit':
        break
    else:
        print(f"I'd love to go to {city.title()}!")
```

一个以`while True` ❶开头的循环将永远运行，除非遇到`break`语句。这个程序中的循环会不断询问用户输入他们去过的城市名，直到他们输入`'quit'`。当他们输入`'quit'`时，`break`语句会执行，导致 Python 退出循环：

```py
Please enter the name of a city you have visited:
(Enter 'quit' when you are finished.) **New York**
I'd love to go to New York!

Please enter the name of a city you have visited:
(Enter 'quit' when you are finished.) **San Francisco**
I'd love to go to San Francisco!

Please enter the name of a city you have visited:
(Enter 'quit' when you are finished.) **quit**
```

### 在循环中使用 continue

与其完全跳出循环而不执行其余的代码，不如使用`continue`语句根据条件测试的结果返回到循环的开头。例如，考虑一个从 1 到 10 计数但仅打印该范围内奇数的循环：

**counting.py**

```py
current_number = 0
while current_number < 10:
❶     current_number += 1
    if current_number % 2 == 0:
        continue

    print(current_number)
```

首先，我们将`current_number`设置为 0。因为它小于 10，Python 进入了`while`循环。进入循环后，我们将计数器增加 1 ❶，因此`current_number`变为 1。接着，`if`语句检查`current_number`与 2 的模。如果模为 0（这意味着`current_number`可以被 2 整除），`continue`语句告诉 Python 忽略循环的其余部分并返回到开头。如果当前数字不能被 2 整除，循环的其余部分将继续执行，Python 打印当前的数字：

```py
1
3
5
7
9
```

### 避免无限循环

每个`while`循环都需要一种停止运行的方法，以免它继续无限运行。例如，这个计数循环应该从 1 计数到 5：

**counting.py**

```py
x = 1
while x <= 5:
    print(x)
    x += 1
```

然而，如果不小心省略了`x += 1`这一行，循环将永远运行：

```py
# This loop runs forever!
x = 1
while x <= 5:
    print(x)
```

现在，`x`的值将从`1`开始，但永远不会改变。因此，条件测试`x <= 5`将始终返回`True`，`while`循环将永远运行，打印一系列的 1，如下所示：

```py
1
1
1
1
`--snip--`
```

每个程序员偶尔都会不小心写出无限`while`循环，尤其是当程序的循环有微妙的退出条件时。如果程序陷入了无限循环，可以按 CTRL-C 或者直接关闭显示程序输出的终端窗口。

为了避免写出无限循环，测试每个`while`循环，确保循环在你预期的时刻停止。如果你希望程序在用户输入某个特定值时结束，运行程序并输入该值。如果程序没有结束，仔细检查程序是如何处理应导致循环退出的值的。确保程序中至少有一部分可以使循环的条件变为`False`，或使其到达`break`语句。

## 使用`while`循环与列表和字典

到目前为止，我们只处理一次一个用户的信息。我们接收用户的输入，然后打印该输入或对此的响应。在下次执行`while`循环时，我们会接收另一个输入值并对此做出回应。但为了跟踪多个用户和信息，我们需要在`while`循环中使用列表和字典。

`for`循环适合遍历列表，但你不应该在`for`循环中修改列表，因为 Python 在跟踪列表中的项时会遇到困难。要在遍历列表时修改它，应该使用`while`循环。使用`while`循环与列表和字典结合，可以收集、存储和组织大量输入，以供后续检查和报告。

### 将项目从一个列表移动到另一个列表

假设有一个新注册但未验证的网站用户列表。在验证这些用户后，我们如何将他们移动到一个单独的已确认用户列表中呢？一种方法是使用`while`循环从未确认用户列表中提取用户进行验证，然后将他们添加到已确认用户的单独列表中。以下是该代码的示例：

**confirmed_users.py**

```py
# Start with users that need to be verified,
#  and an empty list to hold confirmed users.
❶ unconfirmed_users = ['alice', 'brian', 'candace']
confirmed_users = []

# Verify each user until there are no more unconfirmed users.
#  Move each verified user into the list of confirmed users.
❷ while unconfirmed_users:
❸     current_user = unconfirmed_users.pop()

    print(f"Verifying user: {current_user.title()}")
❹     confirmed_users.append(current_user)

# Display all confirmed users.
print("\nThe following users have been confirmed:")
for confirmed_user in confirmed_users:
    print(confirmed_user.title())
```

我们从一个包含未确认用户的列表❶（Alice、Brian 和 Candace）开始，并有一个空列表来存放已确认的用户。只要`unconfirmed_users`列表不为空❷，`while`循环就会继续执行。在这个循环中，`pop()`方法会一次从`unconfirmed_users`列表的末尾移除一个未验证的用户❸。因为 Candace 是`unconfirmed_users`列表中的最后一个，所以她的名字会首先被移除，赋值给`current_user`，然后添加到`confirmed_users`列表中❹。接下来是 Brian，然后是 Alice。

我们通过打印确认消息来模拟确认每个用户，然后将他们添加到已确认用户的列表中。当未确认用户列表缩小时，已确认用户列表会增加。当未确认用户列表为空时，循环停止，并打印已确认用户列表：

```py
Verifying user: Candace
Verifying user: Brian
Verifying user: Alice

The following users have been confirmed:
Candace
Brian
Alice
```

### 从列表中移除特定值的所有实例

在第三章中，我们使用了`remove()`函数从列表中移除特定的值。`remove()`函数之所以有效，是因为我们关心的值在列表中只出现一次。但是，如果你想从列表中移除某个值的所有实例呢？

假设你有一个包含多个`'cat'`值的宠物列表。要移除该值的所有实例，你可以运行一个`while`循环，直到`'cat'`不再出现在列表中，如下所示：

**pets.py**

```py
pets = ['dog', 'cat', 'dog', 'goldfish', 'cat', 'rabbit', 'cat']
print(pets)

while 'cat' in pets:
    pets.remove('cat')

print(pets)
```

我们从一个包含多个`'cat'`实例的列表开始。打印列表后，Python 进入`while`循环，因为它至少发现一次`'cat'`出现在列表中。一旦进入循环，Python 会移除第一个`'cat'`实例，返回到`while`行，然后当它发现`'cat'`仍然在列表中时再次进入循环。它会移除每一个`'cat'`实例，直到列表中不再有该值，此时 Python 退出循环并再次打印列表：

```py
['dog', 'cat', 'dog', 'goldfish', 'cat', 'rabbit', 'cat']
['dog', 'dog', 'goldfish', 'rabbit']
```

### 使用用户输入填充字典

你可以在每次通过`while`循环时提示用户输入所需的内容。让我们制作一个投票程序，其中每次通过循环时都提示参与者输入名字和回答。我们将把收集的数据存储在字典中，因为我们希望将每个回答与特定的用户关联：

**mountain_poll.py**

```py
responses = {}
# Set a flag to indicate that polling is active.
polling_active = True

while polling_active:
    # Prompt for the person's name and response.
❶     name = input("\nWhat is your name? ")
    response = input("Which mountain would you like to climb someday? ")

    # Store the response in the dictionary.
❷     responses[name] = response

    # Find out if anyone else is going to take the poll.
❸     repeat = input("Would you like to let another person respond? (yes/ no) ")
    if repeat == 'no':
        polling_active = False

# Polling is complete. Show the results.
print("\n--- Poll Results ---")
❹ for name, response in responses.items():
    print(f"{name} would like to climb {response}.")
```

程序首先定义一个空字典（`responses`），并设置一个标志（`polling_active`）来指示投票处于激活状态。只要`polling_active`为`True`，Python 就会执行`while`循环中的代码。

在循环中，用户被提示输入他们的名字和想要攀登的山峰❶。这些信息会存储在`responses`字典中❷，然后用户会被询问是否继续进行投票❸。如果他们输入`yes`，程序会再次进入`while`循环。如果他们输入`no`，`polling_active`标志被设置为`False`，`while`循环停止运行，最后的代码块❹会显示投票结果。

如果你运行这个程序并输入一些示例响应，你应该看到类似这样的输出：

```py
What is your name? **Eric**
Which mountain would you like to climb someday? **Denali**
Would you like to let another person respond? (yes/ no) **yes**

What is your name? **Lynn**
Which mountain would you like to climb someday? **Devil's Thumb**
Would you like to let another person respond? (yes/ no) **no**

--- Poll Results ---
Eric would like to climb Denali.
Lynn would like to climb Devil's Thumb.
```

## 摘要

在这一章中，你学习了如何使用`input()`让用户在程序中提供自己的信息。你学会了处理文本和数字输入，并且掌握了如何使用`while`循环让程序根据用户的需求运行。你还看到几种控制`while`循环流程的方法，诸如设置`active`标志、使用`break`语句以及使用`continue`语句。你学习了如何使用`while`循环将项目从一个列表移动到另一个列表，并且如何从列表中移除某个值的所有实例。你还学到了如何在字典中使用`while`循环。

在第八章中，你将学习函数。*函数*允许你将程序分解成小块，每一块都执行一个特定的任务。你可以根据需要调用一个函数任意次数，也可以将函数存储在单独的文件中。通过使用函数，你将能够编写更高效的代码，这些代码更容易排除故障和维护，并且可以在多个不同的程序中重复使用。

