## A

Python 关键字

![图片](Images/common.jpg)

在Python（以及大多数编程语言）中，*关键字*是具有特殊意义的单词。它们是编程语言本身的一部分，因此不能用于其他任何用途。例如，如果你试图将关键字用作变量，或以错误的方式使用它们，Python控制台会显示奇怪的错误信息。本附录描述了每个Python关键字。你可以将它作为一个方便的参考，继续编程时使用。

### and

关键字`and`用于在语句中将两个表达式连接在一起（例如if语句），表示这两个表达式都必须为真。以下是一个示例：

```py
if age > 12 and age < 20:
    print('Beware the teenager!!!!')

```

这段代码意味着变量age的值必须大于12且小于20，消息才会被打印出来。

### as

关键字`as`用于为导入的模块指定一个别名。例如，假设你有一个非常长的模块名称：

i_am_a_python_module_that_is_not_very_useful.

每次使用该模块时都需要输入这个模块名称会非常烦人：

```py
>>> import i_am_a_python_module_that_is_not_very_useful
>>> i_am_a_python_module_that_is_not_very_useful.do_something()

I have done something that is not useful.
>>> i_am_a_python_module_that_is_not_very_useful.do_something_else()

I have done something else that is not useful!!

```

相反，你可以在导入模块时给它一个新的、更短的名字，然后只需使用这个新名字（就像昵称一样），如下所示：

```py
>>> import i_am_a_python_module_that_is_not_very_useful as notuseful
>>> notuseful.do_something()

I have done something that is not useful.
>>> notuseful.do_something_else()

I have done something else that is not useful!!

```

### Assert

`assert`关键字用于表示某个值必须为真。这是捕获代码中的错误和问题的另一种方式，通常用于更高级的程序中（这也是我们在*Python for Kids*中不使用`assert`的原因）。以下是一个简单的assert语句：

```py
>>> mynumber = 10
>>> assert mynumber < 5
Traceback (most recent call last):
  File "<pyshell#1>", line 1, in <module>
    assert mynumber < 5
AssertionError

```

在这个示例中，我们断言变量mynumber的值小于5。它并不是，所以Python会显示一个错误（称为AssertionError）。

### ASYNC

`async`关键字用于定义一种叫做*原生协程*的东西。这是异步编程中的一个高级概念（即并行做多件事，或者做一些延迟的事情）。

### Await

`await`关键字也用于异步编程（与`async`类似）。

### Break

`break`关键字用于停止某段代码的执行。你可以在for循环中使用`break`，像这样：

```py
age = 10
for x in range(1, 100):
    print(f'counting {x}')
    if x == age:
        print('end counting')
        break

```

由于变量age在这里被设置为10，代码将打印出以下内容：

```py
counting 1
counting 2
counting 3
counting 4
counting 5
counting 6
counting 7
counting 8
counting 9
counting 10
end counting

```

一旦变量x的值达到10，代码将打印文本“end counting”，然后跳出循环。

### Class

`class`关键字用于定义一种对象类型，比如车辆、动物或人类。类可以有一个叫做`__init__`的函数，用于执行对象创建时所需的所有任务。例如，创建Car类的对象时可能需要一个颜色变量：

```py
>>> class Car:
        def __init__(self, color):
            self.color = color

>>> car1 = Car('red')
>>> car2 = Car('blue')
>>> print(car1.color)
red
>>> print(car2.color)
blue

```

### Continue

`continue`关键字是“跳到”循环中的下一个项，这样循环块中的其余代码就不会被执行。与`break`不同，我们不是跳出循环——我们只是继续执行下一个项。例如，如果我们有一个物品列表，并且希望跳过以*b*开头的项，可以使用以下代码：

```py
>>> my_items = ['apple', 'aardvark', 'banana', 'badger',
        'clementine', 'camel']
>>> for item in my_items:
        if item.startswith('b'):
            continue
        print(item)

apple
aardvark
clementine
camel

```

我们首先创建我们的项目清单，然后使用`for`循环遍历这些项目，并为每个项目执行一段代码。如果项目以字母*b*开头，我们会跳过该项目。否则，我们打印出该项目。

### 定义

`def`关键字用于定义一个函数。例如，我们可以创建一个函数，将一段年份转化为等效的分钟数：

```py
>>> def minutes(years):
        return years * 365 * 24 * 60
>>> minutes(10)
5256000

```

### 删除

`del`关键字用于删除某个东西。例如，如果你在日记中有一个生日愿望清单，但后来改变了对某个愿望的想法，你可能会把它从清单中划掉，并添加一个新的愿望：

![图片](Images/288fig01.jpg)

在Python中，原始列表看起来像这样：

```py
what_i_want = ['remote controlled car', 'new bike', 'computer game']

```

你可以通过使用`del`和你想删除的项目的索引来移除计算机游戏。然后，你可以通过`append`函数添加新项目：

```py
del what_i_want[2]
what_i_want.append('roboreptile')

```

然后打印出新的列表：

```py
print(what_i_want)
['remote controlled car', 'new bike', 'roboreptile']

```

### 否则

`elif`关键字作为`if`语句的一部分使用。参见`if`关键字的描述，了解示例。

### 否则

`else`关键字作为`if`语句的一部分使用。参见`if`关键字的描述，了解示例。

### 除外

`except`关键字用于捕捉在比较复杂的代码中可能出现的问题。

### 最后

`finally`关键字用于确保发生错误时，某些代码能够运行（通常用于清理代码留下的任何残余）。这个关键字用于更高级的编程。

### 循环

`for`关键字用于创建一个执行特定次数的代码循环。这里有一个例子：

```py
for x in range(0, 5):
    print(f'x is {x}')

```

这个`for`循环执行代码块（打印语句）五次，结果如下输出：

```py
x is 0
x is 1
x is 2
x is 3
x is 4

```

### 从

导入模块时，你可以使用`from`关键字仅导入你需要的部分。例如，第[4章](ch04.xhtml#ch04)介绍的`turtle`模块有一个叫做`Turtle`的类，我们使用它来创建一个Turtle对象（该对象包括海龟移动的画布）。下面是如何导入整个`turtle`模块并使用`Turtle`类：

```py
import turtle
t = turtle.Turtle()

```

你也可以单独导入Turtle类，然后直接使用它（完全不需要引用turtle模块）：

```py
from turtle import Turtle
t = Turtle()

```

你可能这么做是为了，下次查看该程序顶部时，可以看到你正在使用的所有函数和类（这在导入许多模块的较大程序中特别有用）。然而，如果你选择这么做，你将无法使用你没有导入的模块部分。例如，`time`模块有`localtime`和`gmtime`函数，但如果你只导入了`localtime`，然后尝试使用`gmtime`，你会得到一个错误：

```py
>>> from time import localtime
>>> print(localtime())
(2019, 1, 30, 20, 53, 42, 1, 30, 0)
>>> print(gmtime())
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'gmtime' is not defined

```

错误信息“`gmtime`未定义”意味着Python对`gmtime`函数一无所知，这是因为你没有导入它。

如果一个特定模块有多个你想使用的函数，而你不想通过模块名来引用它们（例如，time.localtime，或time.gmtime），你可以使用星号（*）导入模块中的所有内容，像这样：

```py
>>> from time import *
>>> print(localtime())
(2021, 1, 30, 20, 57, 7, 1, 30, 0)
>>> print(gmtime())
(2021, 1, 30, 13, 57, 9, 1, 30, 0)

```

这种形式从time模块中导入了所有内容，你现在可以按名称引用各个函数。

### 全局

程序中*作用域*的概念在[第7章](ch07.xhtml#ch07)中介绍。作用域是指变量的可见性。如果一个变量在函数外部定义，通常可以在函数内部看到（它是可见的）。另一方面，如果变量在函数内部定义，通常无法在函数外部看到。`global`关键字是此规则的一个例外。定义为全局变量的变量可以在任何地方看到。下面是一个示例：

```py
>>> def test():
        global a
        a = 1
        b = 2

```

你认为当你调用`print(a)`然后运行`print(b)`时会发生什么？第一个会正常工作，但第二个会显示一个错误消息：

```py
>>> test()
>>> print(a)
1
>>> print(b)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'b' is not defined

```

变量a已经被更改为在函数内部具有全局作用域，因此它在函数执行完毕后仍然可见；但变量b仍然仅在函数内部可见。（在设置变量值之前，必须使用`global`关键字。）

### 如果

`if`关键字用于对某件事做出决策。它也可以与`else`和`elif`（else if）一起使用。一个if语句就是在说：“如果某个条件为真，那么执行某个操作。”下面是一个示例：

```py
if toy_price > 1000:
    print('That toy is overpriced')
elif toy_price > 100:
    print('That toy is expensive')
else:
    print('I can afford that toy')

```

这个if语句表示，如果玩具价格超过$1,000，显示一条消息说它被高价出售；否则，如果玩具价格超过$100，显示一条消息说它很贵。如果这两个条件都不成立，则应该显示消息：“我能买得起这个玩具。”

### 导入

`import`关键字告诉Python加载一个模块，以便可以使用。例如，以下代码告诉Python使用`sys`模块：

```py
import sys

```

### 在

`in`关键字用于表达式中，查看某个项目是否在一组项目中。例如，数字1是否可以在一个数字列表（一个集合）中找到？

```py
>>> if 1 in [1,2,3,4]:
        print('number is in the list')
number is in the list

```

以下是如何判断字符串’pants’是否在服装列表中的示例：

```py
>>> clothing_list = ['shorts', 'undies', 'boxers', 'long johns',
                 'knickers']
>>> if 'pants' in clothing_list:
        print('pants is in the list')
    else:
        print('where are my pants?')
where are my pants?

```

### 是

`is`关键字有点像*等于*操作符（==），用于判断两个东西是否相等（例如，10 == 10为真，10 == 11为假）。然而，`is`和`==`之间有一个根本的区别。如果你在比较两个对象（如列表），`==`可能返回真，而`is`则可能不返回真（即使你认为这些对象是相同的）。这是一个高级编程概念。

### Lambda

`lambda`关键字用于创建匿名或内联函数。这个关键字用于更高级的程序中。

### `nonlocal`

`nonlocal`关键字用于将一个变量包含到一个函数的作用域中，当它在函数外部声明时。这个关键字用于更高级的程序中。

### `not`

如果某个条件为真，`not`关键字使其变为假。例如，如果我们创建一个变量a并将其设置为True的值，然后使用`not`打印该变量的值，结果如下：

```py
>>> a = True
>>> print(not a)
False

```

同样地，对于一个False值，我们得到True：

```py
>>> b = False
>>> print(not b)
True

```

这看起来似乎不是很有用，直到你开始在if语句中使用这个关键字。例如，要检查一个项目是否不在列表中，我们可以写如下代码：

```py
>>> clothing_list = ['shorts', 'undies', 'boxers', 'long johns',
                 'knickers']
>>> if 'pants' not in clothing_list:
        print('You really need to buy some pants')
You really need to buy some pants

```

### 或者

or关键字将两个条件连接在一起，表示至少一个条件应为真。下面是一个示例：

```py
if dino == 'Tyrannosaurus' or dino == 'Allosaurus':
    print('Carnivores')
elif dino == 'Ankylosaurus' or dino == 'Apatosaurus':
    print('Herbivores')

```

在这种情况下，如果变量dino的值是暴龙（Tyrannosaurus）或异特龙（Allosaurus），程序会打印出“食肉动物”。如果是甲龙（Ankylosaurus）或梁龙（Apatosaurus），程序会打印出“食草动物”。

### pass

有时候在开发程序时，你可能只想写一些小片段来尝试一些功能。这样做的问题是，如果你没有代码块，不能使用if语句来检查条件是否为真；同样，如果没有执行体，你也不能使用for循环。例如，以下代码是可以正常工作的：

```py
>>> age = 15
>>> if age > 10:
        print('older than 10')
older than 10

```

但是如果你没有为if语句提供代码块（主体），你会得到一个错误信息：

```py
>>> age = 15
>>> if age > 10:
File "<stdin>", line 2
    ^
IndentationError: expected an indented block after 'if' statement on line 1
```

这是Python显示的错误信息，当你在某些语句后面缺少代码块时（如果你使用的是IDLE，它甚至不允许你输入这样的代码）。在这种情况下，你可以使用pass关键字写一个语句，但不提供与之对应的代码块。

举个例子，假设你想在一个for循环中使用if语句。也许你还没决定在if语句中写什么——或许你会使用print函数，或者加入break，或者其他什么操作。此时，你可以使用pass，代码仍然能正常工作（即使它还没完全做你想做的事）。以下是我们再次使用pass关键字的if语句：

```py
>>> age = 15
>>> if age > 10:
        pass

```

以下代码展示了pass关键字的另一种用法：

```py
>>> for x in range(0, 7):
>>>     print(f'x is {x}')
>>>     if x == 4:
            pass

x is 0
x is 1
x is 2
x is 3
x is 4
x is 5
x is 6

```

每次Python执行循环体中的代码时，它仍然会检查x变量是否等于4，但因为没有提供其他操作，它什么也不做，所以它会打印出从0到7的每一个数字。稍后，你可以在if语句的代码块中添加其他代码，将pass关键字替换为其他内容，比如break：

```py
>>> for x in range(0, 7):
        print(f'x is {x}')
        if x == 4:
            break

x is 1
x is 2
x is 3
x is 4

```

pass关键字最常见的用法是，在创建函数时，如果你还不想写出函数的代码，可以先使用它。

### raise

raise关键字可以用来触发一个错误。听起来可能有点奇怪，但在高级编程中，它实际上非常有用。

### return

return关键字用于从函数中返回一个值。例如，你可以创建一个函数来计算你到上一个生日为止活了多少秒：

```py
def age_in_seconds(age_in_years):
    return age_in_years * 365 * 24 * 60 * 60

```

当你调用这个函数时，返回的值可以赋值给另一个变量，或者可以直接打印出来：

```py
>>> seconds = age_in_seconds(9)
>>> print(seconds)
283824000
>>> print(age_in_seconds(12))
378432000

```

### try

try 关键字开始一个代码块，这个块以 except 和 finally 关键字结束。try/except/finally 代码块一起用于处理程序中的错误，例如确保程序向用户显示有用的消息，而不是一个难懂的 Python 错误。它们在高级程序中非常有用。

### while

while 关键字有点像 for，不过 for 循环遍历一个范围（数字），而 while 循环在某个表达式为真时会一直运行。使用 while 循环时要小心——如果表达式始终为真，循环将永远不会结束（这叫做 *无限循环*）。以下是一个例子：

```py
>>> x = 1
>>> while x == 1:
        print('hello')

```

如果你运行这段代码，它会永远循环下去，或者至少直到你关闭 Python Shell 或按下 CTRL-C 来中断它。然而，以下代码会打印出“hello”九次（每次将变量 x 加 1，直到 x 不再小于 10）：

```py
>>> x = 1
>>> while x < 10:
        print('hello')
        x = x + 1

```

### with

with 关键字与一种特殊的对象一起使用，用来创建一个代码块，类似于 try 和 finally 关键字，然后管理该对象的资源。这个关键字通常用于高级程序中。

### yield

yield 关键字有点像 return，但它与一种叫做 *生成器* 的特定对象一起使用。生成器根据请求生成值，因此在这方面，range 函数的行为像一个生成器。
