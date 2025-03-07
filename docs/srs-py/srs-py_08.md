## **8**

**函数式编程**

![image](../images/common01.jpg)

许多 Python 开发者并不了解在 Python 中使用函数式编程的广泛性，这真是太可惜了：除了少数例外，函数式编程允许你编写更简洁和高效的代码。而且，Python 对函数式编程的支持非常广泛。

本章将介绍 Python 的一些函数式编程方面的内容，包括创建和使用生成器。你将了解最有用的函数式编程包和函数，以及如何将它们结合使用，以获得最高效的代码。

**注意**

*如果你想认真学习函数式编程，以下是我的建议：休息一下，学习一种功能非常强大的函数式编程语言，比如 Lisp。我知道在 Python 书中谈论 Lisp 可能听起来很奇怪，但我玩了几年的 Lisp，它教会了我如何“以函数式思维”思考。如果你所有的经验都来自命令式和面向对象编程，你可能不会培养出充分利用函数式编程的思维过程。Lisp 本身并非纯粹函数式，但它比你在 Python 中看到的更注重函数式编程。*

### **创建纯函数**

当你使用函数式风格编写代码时，你的函数设计上不会有副作用：它们接受输入并产生输出，而不会保留状态或修改任何不反映在返回值中的内容。遵循这一理想的函数被称为*纯函数*。

让我们从一个普通的、非纯函数的例子开始，它移除列表中的最后一个元素：

def remove_last_item(mylist):

"""从列表中移除最后一个元素。"""

mylist.pop(-1)  # 这会修改 mylist

以下是相同函数的纯函数版本：

def butlast(mylist):

return mylist[:-1]  # 这返回 mylist 的副本

我们定义了一个 `butlast()` 函数，它像 Lisp 中的 `butlast` 一样，返回一个去掉最后一个元素的列表，*而不会* 修改原始列表。相反，它返回一个包含修改的列表副本，从而让我们保持原始列表不变。

函数式编程的实际优点包括：

**模块化** 使用函数式编程风格强制在解决单独问题时进行一定程度的分离，使代码的各个部分在其他上下文中更易于重用。由于函数不依赖于任何外部变量或状态，从不同的代码片段调用它是非常直接的。

**简洁性** 函数式编程通常比其他编程范式更简洁。

**并发性** 纯粹的函数式函数是线程安全的，并且可以并发运行。一些函数式语言会自动处理这一点，如果你需要扩展应用程序，这将是一个很大的帮助，尽管在 Python 中情况还不是完全如此。

**可测试性** 测试一个功能性程序非常容易：你只需要一组输入和一组预期的输出。它们是*幂等的*，意味着在相同的参数下反复调用同一个函数将始终返回相同的结果。

### **生成器**

一个*生成器*是一个像迭代器一样工作的对象，它在每次调用 next() 方法时生成并返回一个值，直到引发 StopIteration 为止。生成器最早在 PEP 255 中介绍，它提供了一种创建实现*迭代器协议*的对象的简便方法。虽然以函数式风格编写生成器并不是绝对必要的，但这样做可以使生成器更容易编写和调试，这也是一种常见做法。

要创建一个生成器，只需编写一个包含 yield 语句的常规 Python 函数。Python 会检测到 yield 的使用并将该函数标记为生成器。当执行到 yield 语句时，函数会像使用 return 语句一样返回一个值，但有一个显著的不同：解释器会保存堆栈引用，下一次调用 next() 函数时，将使用该引用恢复函数的执行。

当函数被执行时，它们的执行链会产生一个*堆栈*——函数调用被堆叠在一起。当一个函数返回时，它会从堆栈中移除，返回的值会传递给调用函数。对于生成器来说，函数并不会真正返回，而是*产生*（yield）。因此，Python 会将函数的状态保存为堆栈引用，在需要生成器的下一次迭代时，恢复到保存的执行点。

#### ***创建生成器***

如前所述，你通过编写一个普通函数并在函数体内包含 yield 来创建一个生成器。[Listing 8-1](ch08.xhtml#ch8list1) 创建了一个名为 mygenerator() 的生成器，包含了三个 yield，这意味着它会在接下来的三次 next() 调用中进行迭代。

>> def mygenerator():

...      yield 1

...      yield 2

...      yield 'a'

...

>>> mygenerator()

<generator object mygenerator at 0x10d77fa50>

>>> g = mygenerator()

>>> next(g)

1

>>> next(g)

2

>>> next(g)

'a' >>> next(g)

Traceback (most recent call last):

文件 "<stdin>"，第 1 行，在 <module>

StopIteration

*Listing 8-1：创建一个包含三次迭代的生成器*

当没有更多的 yield 语句时，下一次调用 next() 时会引发 StopIteration。

在 Python 中，当一个生成器函数 yield 某个值时，生成器会保存对堆栈的引用，并且当再次执行 next() 调用时，生成器会恢复这个堆栈。

在没有使用生成器的情况下迭代任何数据的幼稚方法是先构建整个列表，这通常会浪费内存。

假设我们想找出 1 到 10,000,000 之间第一个等于 50,000 的数字。听起来很简单，不是吗？让我们把它变成一个挑战。我们将 Python 运行在 128MB 的内存限制下，并尝试最初构建整个列表的幼稚方法：

$ ulimit -v 131072

$ python3

>>> a = list(range(10000000))

这个简单的方法首先尝试构建列表，但如果我们执行到目前为止的程序：

Traceback (most recent call last): File "<stdin>", line 1, in <module>

MemoryError

哎呀，结果我们不能用仅有的128MB内存构建1000万个条目的列表！

**警告**

*在Python 3中，range()在迭代时返回一个生成器。要在Python 2中获取生成器，你必须使用xrange()。此函数在Python 3中不再存在，因为它已不再需要。*

让我们尝试使用生成器，仍然保持128MB的限制：

$ ulimit -v 131072

$ python3

>>> for value in range(10000000):

...      if value == 50000:

...             print("找到了")

...             break

...

找到了

这次，我们的程序顺利执行。当它被迭代时，range()类返回一个生成器，该生成器动态生成我们的整数列表。更好的是，由于我们只对第50,000个数字感兴趣，因此生成器仅需要生成50,000个数字就停止了，而不是构建完整的列表。

通过动态生成值，生成器允许你以最小的内存和处理周期消耗来处理大数据集。每当你需要处理大量值时，生成器可以帮助你高效地处理它们。

#### ***使用yield返回和传递值***

yield语句还有一个较少使用的特性：它可以像函数调用一样返回一个值。这使我们能够通过调用生成器的send()方法将一个值传递给生成器。作为使用send()的示例，我们将编写一个名为shorten()的函数，它接受一个字符串列表并返回一个由这些相同字符串组成的列表，只不过是被截断的（[示例 8-2](ch08.xhtml#ch8list2)）。

def shorten(string_list):

length = len(string_list[0])

for s in string_list:

length = yield s[:length]

mystringlist = ['loremipsum', 'dolorsit', 'ametfoobar']

shortstringlist = shorten(mystringlist)

result = []

try:

s = next(shortstringlist)

result.append(s)

while True:

number_of_vowels = len(filter(lambda letter: letter in 'aeiou', s))

# 根据下一个字符串的长度截断

# 基于前一个字符串中的元音字母数

s = shortstringlist.send(number_of_vowels)

result.append(s)

except StopIteration:

pass

*示例 8-2：使用send()返回和使用值*

在这个例子中，我们编写了一个名为shorten()的函数，它接受一个字符串列表，并返回一个由这些相同字符串组成的列表，只不过是被截断的。每个截断字符串的长度等于前一个字符串中的元音字母数：*loremipsum*包含四个元音字母，因此生成器返回的第二个值将是*dolorsit*的前四个字母；*dolo*只有两个元音字母，所以*ametfoobar*将被截断为前两个字母*am*。生成器然后停止并引发StopIteration。我们的生成器因此返回：

['loremipsum', 'dolo', 'am']

使用 yield 和 send() 以这种方式让 Python 生成器像 Lua 和其他语言中的 *协程* 一样工作。

PEP 289 引入了生成器表达式，使得可以使用类似于列表推导式的语法构建单行生成器：

>>> (x.upper() for x in ['hello', 'world'])

<generator object <genexpr> at 0x7ffab3832fa0>

>>> gen = (x.upper() for x in ['hello', 'world'])

>>> list(gen)

['HELLO', 'WORLD']

在这个例子中，gen 是一个生成器，就像我们使用了 yield 语句一样。这里的 yield 是隐式的。

#### ***检查生成器***

要确定一个函数是否被视为生成器，可以使用 inspect.isgeneratorfunction()。在 [列出 8-3](ch08.xhtml#ch8list3) 中，我们创建了一个简单的生成器并检查它。

>>> import inspect

>>> def mygenerator():

...     yield 1

...

>>> inspect.isgeneratorfunction(mygenerator)

True

>>> inspect.isgeneratorfunction(sum)

False

*列出 8-3：检查一个函数是否是生成器*

导入 inspect 包以使用 isgeneratorfunction()，然后传入要检查的函数名称。阅读 inspect.isgeneratorfunction() 的源代码，可以帮助我们了解 Python 如何标记函数为生成器（见 [列出 8-4](ch08.xhtml#ch8list4)）。

def isgeneratorfunction(object):

"""如果对象是用户定义的生成器函数，返回真。

生成器函数对象提供与普通函数相同的属性。

查看帮助(isfunction)，了解属性列表。"""

return bool((isfunction(object) or ismethod(object)) and

object.func_code.co_flags & CO_GENERATOR)

*列出 8-4：inspect.isgeneratorfunction() 的源代码*

isgeneratorfunction() 函数检查对象是否是一个函数或方法，并且其代码中是否设置了 CO_GENERATOR 标志。这个例子展示了理解 Python 内部工作原理是多么简单。

inspect 包提供了 inspect.getgeneratorstate() 函数，它返回生成器的当前状态。我们将在不同的执行点使用它来查看 mygenerator() 的状态：

>>> import inspect

>>> def mygenerator():

...     yield 1

...

>>> gen = mygenerator()

>>> gen

<generator object mygenerator at 0x7f94b44fec30>

>>> inspect.getgeneratorstate(gen)

➊ 'GEN_CREATED'

>>> next(gen)

1

>>> inspect.getgeneratorstate(gen)

➋ 'GEN_SUSPENDED'

>>> next(gen)

回溯（最近的调用最先出现）：

文件 "<stdin>"，第 1 行，在 <module>

StopIteration

>>> inspect.getgeneratorstate(gen)

➌ 'GEN_CLOSED'

这使我们能够判断生成器是否等待第一次运行（GEN_CREATED）➊，等待通过 next() 恢复执行（GEN_SUSPENDED）➋，或者已经完成执行（GEN_CLOSED）➌。这对于调试生成器可能会很有帮助。

### **列表推导式**

列表推导式，简称*listcomp*，允许你在声明列表时直接定义其内容。要将列表转换为列表推导式，你必须像平常一样将其包裹在方括号中，但还需包含一个表达式，该表达式会生成列表中的项，并且需要一个for循环来遍历它们。

以下示例在不使用列表推导式的情况下创建一个列表：

>>> x = []

>>> for i in (1, 2, 3):

...     x.append(i)

...

>>> x

[1, 2, 3]

下一个示例使用列表推导式在一行代码中生成相同的列表：

>>> x = [i for i in (1, 2, 3)]

>>> x

[1, 2, 3]

使用列表推导式有两个优势：使用列表推导式编写的代码通常更简短，因此Python需要执行的操作也更少。与其反复创建列表并调用append，不如让Python一次性创建出项的列表并将它们移动到一个新的列表中。

你可以将多个for语句组合在一起，并使用if语句过滤掉某些项。这里我们创建了一个单词列表，并使用列表推导式将每个项首字母大写，拆分多个单词的项为单个单词，并删除多余的*or*：

x = [word.capitalize()]

for line in ("hello world?", "world!", "or not")

for word in line.split()

if not word.startswith("or")]

>>> x

['Hello', 'World?', 'World!', 'Not']

这段代码有两个for循环：第一个遍历文本行，第二个遍历每行中的单词。最后的if语句会过滤掉以*or*开头的单词，从最终列表中排除它们。

使用列表推导式而不是for循环是一种快速定义列表的简洁方法。由于我们仍在讨论函数式编程，值得注意的是，通过列表推导式构建的列表不应依赖于修改程序的状态：在构建列表时不应修改任何变量。这通常使得列表比没有使用列表推导式的列表更加简洁和易读。

请注意，也有类似的语法可以以相同的方式构建字典或集合，如下所示：

>>> {x:x.upper() for x in ['hello', 'world']}

{'world': 'WORLD', 'hello': 'HELLO'}

>>> {x.upper() for x in ['hello', 'world']}

set(['WORLD', 'HELLO'])

### **函数式编程中的函数**

在使用函数式编程操作数据时，你可能会反复遇到相同的一组问题。为了帮助你高效地处理这种情况，Python包含了许多函数式编程的内置函数。本节将快速概述一些这些内置函数，它们可以帮助你构建完全函数化的程序。一旦你了解了这些函数的作用，我鼓励你进一步研究，并尝试在自己的代码中使用这些函数。

#### ***使用map()函数对项进行操作***

map()函数的形式为map(function, iterable)，并将function应用于iterable中的每一项，返回一个列表（Python 2）或一个可迭代的map对象（Python 3），如[Listing 8-5](ch08.xhtml#ch8list5)所示。

>>> map(lambda x: x + "bzz!", ["我认为", "我很好"])

<map object at 0x7fe7101abdd0>

>>> list(map(lambda x: x + "bzz!", ["我认为", "我很好"]))

['我认为bzz!', "我很好bzz!"]

*Listing 8-5: 在Python 3中使用map()*

你可以通过列表推导式写出一个等效的map()，像这样：

>>> (x + "bzz!" for x in ["我认为", "我很好"])

<generator object <genexpr> at 0x7f9a0d697dc0>

>>> [x + "bzz!" for x in ["我认为", "我很好"]]

['我认为bzz!', "我很好bzz!"]

#### ***使用filter()过滤列表***

filter()函数的形式为filter(function或None, iterable)，并根据function返回的结果过滤iterable中的项目。它将在Python 2中返回一个列表，或在Python 3中返回一个可迭代的filter对象：

>>> filter(lambda x: x.startswith("我 "), ["我认为", "我很好"])

<filter object at 0x7f9a0d636dd0>

>>> list(filter(lambda x: x.startswith("我 "), ["我认为", "我很好"]))

['我认为']

你也可以通过列表推导式写出一个等效的filter()，像这样：

>>> (x for x in ["我认为", "我很好"] if x.startswith("我 "))

<generator object <genexpr> at 0x7f9a0d697dc0>

>>> [x for x in ["我认为", "我很好"] if x.startswith("我 ")]

['我认为']

#### ***使用enumerate()获取索引***

enumerate()函数的形式为enumerate(iterable[, start])，返回一个可迭代的对象，提供一个元组序列，每个元组包含一个整数索引（如果提供了start，则从start开始）和对应的项。这个函数在你需要编写引用数组索引的代码时非常有用。例如，代替写出以下代码：

i = 0

while i < len(mylist): print("项目 %d: %s" % (i, mylist[i]))

i += 1

你可以用enumerate()更高效地完成相同的事情，如下所示：

for i, item in enumerate(mylist):

print("项目 %d: %s" % (i, item))

#### ***使用sorted()排序列表***

sorted()函数的形式为sorted(iterable, key=None, reverse=False)，并返回iterable的排序版本。key参数允许你提供一个返回用于排序的值的函数，如下所示：

>>> sorted([("a", 2), ("c", 1), ("d", 4)])

[('a', 2), ('c', 1), ('d', 4)]

>>> sorted([("a", 2), ("c", 1), ("d", 4)], key=lambda x: x[1])

[('c', 1), ('a', 2), ('d', 4)]

#### ***使用any()和all()找到满足条件的项***

any(iterable)和all(iterable)函数返回一个布尔值，取决于iterable返回的值。这些简单的函数等价于以下完整的Python代码：

def all(iterable):

for x in iterable:

if not x:

return False

return True

def any(iterable):

for x in iterable:

if x:

return True

return False

这些函数用于检查可迭代对象中的任意或所有值是否满足给定的条件。例如，以下代码检查列表是否满足两个条件：

mylist = [0, 1, 3, -1]

if all(map(lambda x: x > 0, mylist)):

print("所有项都大于 0")

if any(map(lambda x: x > 0, mylist)):

print("至少有一项大于 0")

这里的区别在于，any() 在至少一个元素满足条件时返回 True，而 all() 只有在每个元素都满足条件时才返回 True。对于空的可迭代对象，all() 也会返回 True，因为其中没有元素是 False。

#### ***使用 zip() 合并列表***

zip() 函数的形式为 zip(iter1 [,iter2 [...]]). 它接受多个序列并将它们组合成元组。当你需要将一组键与一组值组合成字典时，这非常有用。与这里描述的其他函数一样，zip() 在 Python 2 中返回一个列表，而在 Python 3 中返回一个可迭代对象。这里我们将键列表映射到值列表，从而创建一个字典：

>>> keys = ["foobar", "barzz", "ba!"]

>>> map(len, keys)

<map object at 0x7fc1686100d0>

>>> zip(keys, map(len, keys))

<zip object at 0x7fc16860d440>

>>> list(zip(keys, map(len, keys)))

[('foobar', 6), ('barzz', 5), ('ba!', 3)]

>>> dict(zip(keys, map(len, keys)))

{'foobar': 6, 'barzz': 5, 'ba!': 3}

**Python 2 和 3 中的函数式函数**

你可能已经注意到，Python 2 和 Python 3 中返回值的类型是不同的。Python 的纯函数式内建函数在 Python 2 中返回的是列表，而不是可迭代对象，这使得它们在内存效率上不如 Python 3 的 *x* 等价物。如果你打算使用这些函数编写代码，请记住，它们在 Python 3 中能发挥最大的效益。如果你还在使用 Python 2，不用担心：标准库中的 itertools 模块提供了这些函数的基于迭代器的版本（如 itertools.izip()、itertools.imap()、itertools.ifilter() 等）。

#### ***解决常见问题***

还有一个重要的工具尚未介绍。我们在处理列表时，常常需要找到第一个满足特定条件的项。我们将研究多种实现方式，并最终看到最有效的方法：first 包。

##### **用简单代码查找项**

我们可能能够通过像这样的函数找到第一个满足条件的项：

def first_positive_number(numbers):

for n in numbers:

if n > 0:

return n

我们可以像这样用函数式风格重写 first_positive_number() 函数：

def first(predicate, items):

for item in items:

if predicate(item):

return item

first(lambda x: x > 0, [-1, 0, 1, 2])

通过使用函数式编程方法，将谓词作为参数传递，函数变得更容易重用。我们甚至可以将其写得更简洁，像这样：

# 低效

list(filter(lambda x: x > 0, [-1, 0, 1, 2]))[0]

# 高效

next(filter(lambda x: x > 0, [-1, 0, 1, 2]))

注意，如果没有项目满足条件，这可能会引发IndexError，从而导致list(filter())返回一个空列表。

对于简单的情况，你可以依靠next()来防止IndexError的发生，像这样：

>>> a = range(10)

>>> next(x for x in a if x > 3)

4

[示例 8-6](ch08.xhtml#ch8list6)将引发StopIteration错误，如果条件永远无法满足。这个问题也可以通过在next()中添加第二个参数来解决，像这样：

>>> a = range(10)

>>> next((x for x in a if x > 10), '默认值')

'默认值'

*示例 8-6：在条件不满足时返回默认值*

当条件无法满足时，这将返回默认值，而不是错误。幸运的是，Python提供了一个包来处理这些问题。

##### **使用first()查找项目**

与其在所有程序中都写出[示例 8-6](ch08.xhtml#ch8list6)中的函数，不如首先包含这个小的Python包。[示例 8-7](ch08.xhtml#ch8list7)展示了这个包如何让你找到符合条件的可迭代对象中的第一个元素。

>>> from first import first

>>> first([0, False, None, [], (), 42])

42

>>> first([-1, 0, 1, 2])

-1

>>> first([-1, 0, 1, 2], key=lambda x: x > 0)

1

*示例 8-7：查找列表中第一个满足条件的项目*

你可以看到，first()函数返回列表中第一个有效的非空项目。

##### **使用lambda()与functools**

你会注意到，我们在本章的很多示例中都使用了lambda()。lambda()函数是Python为了方便函数式编程而加入的，用于像map()和filter()这样的函数，否则每次想要检查不同的条件时都需要编写一个全新的函数。[示例 8-8](ch08.xhtml#ch8list8)等同于[示例 8-7](ch08.xhtml#ch8list7)，但没有使用lambda()。

import operator

from first import first

def greater_than_zero(number):

return number > 0

first([-1, 0, 1, 2], key=greater_than_zero)

*示例 8-8：在不使用lambda()的情况下找到第一个满足条件的项目*

这段代码与[示例 8-7](ch08.xhtml#ch8list7)中的代码完全相同，返回满足条件的列表中第一个非空值，但它要繁琐得多：如果我们想要获取一个长度超过42项的序列中的第一个数字，我们就需要通过def定义一个合适的函数，而不是像调用first()时那样在线定义它。

尽管lambda在帮助我们避免类似问题时很有用，但它仍然存在一些问题。first模块包含一个key参数，可以用来提供一个接收每个项目并返回布尔值的函数，指示该项目是否满足条件。然而，我们无法传递key函数，因为它需要的代码超过了一行：lambda语句不能写在多于一行的地方。这是lambda的一个重大限制。

相反，我们将不得不回到繁琐的模式，为每个需要的键编写新的函数定义。但真的是这样吗？

functools 包通过它的 partial() 方法提供了解决方案，这给我们提供了一个比 lambda 更灵活的替代方式。functools.partial() 方法允许我们创建一个包装函数，但有一个不同之处：它不会改变函数的行为，而是改变它接收的 *参数*，就像这样：

从 functools 导入 partial

从 first 导入 first

➊ def greater_than(number, min=0):

return number > min

➋ first([-1, 0, 1, 2], key=partial(greater_than, min=42))

在这里，我们创建了一个新的 greater_than() 函数，它的行为与[示例 8-8](ch08.xhtml#ch8list8)中的 greater_than_zero() 函数相同，但这个版本允许我们指定我们希望与之比较的值，而以前是硬编码的。在这里，我们将 functools.partial() 传递给我们的函数和我们想要的 min 值 ➊，然后我们得到一个新函数，它的 min 值设置为 42，正如我们希望的那样 ➋。换句话说，我们可以编写一个函数并使用 functools.partial() 来定制我们新函数的行为，以适应任何特定情况。

即使是这个版本也可以进一步简化。在这个例子中，我们只是在比较两个数字，结果发现 operator 模块有内置的函数正是用于做这个的：

导入 operator

从 functools 导入 partial

从 first 导入 first

first([-1, 0, 1, 2], key=partial(operator.le, 0))

这是一个很好的例子，展示了 functools.partial() 如何与位置参数配合使用。在这个例子中，函数 operator.le(a, b) 接受两个数字并返回一个布尔值，告诉我们第一个数字是否小于或等于第二个数字，它被传递给 functools.partial()。我们传递给 functools.partial() 的 0 被赋值给 a，传递给 functools.partial() 返回的函数的参数则赋值给 b。所以这个例子与[示例 8-8](ch08.xhtml#ch8list8)的实现方式相同，只不过没有使用 lambda 或定义额外的函数。

**注意**

*functools.partial() 方法通常用于替代 lambda，应该被视为一种更优的替代方案。lambda 函数在 Python 中有点像异类，由于该函数体的大小仅限于一行，Python 3 曾考虑过完全去掉它。*

#### ***有用的 itertools 函数***

最后，我们将了解一些 Python 标准库中 itertools 模块的有用函数，您应该知道它们。许多程序员因为不知道 Python 已经提供了这些功能，所以会自己编写这些函数。它们都是为帮助您操作迭代器而设计的（这就是该模块名为 *iter-tools* 的原因），因此它们都是纯函数式的。这里我将列出其中一些，并简要介绍它们的功能，如果它们对您有用，我鼓励您进一步研究。

+   accumulate(iterable[, func])返回可迭代对象中项目的累积和系列。

+   chain(*iterables)依次迭代多个可迭代对象，而不构建所有项目的中间列表。

+   combinations(iterable, r)从给定的可迭代对象生成长度为r的所有组合。

+   compress(data, selectors)将从selectors中应用布尔掩码到data，并仅返回data中相应selectors元素为True的值。

+   count(start, step)生成一个从start开始并且每次调用增加step的无限序列值。

+   cycle(iterable)在可迭代对象中重复循环值。

+   repeat(elem[, n])重复元素n次。

+   dropwhile(predicate, iterable)会从开始处过滤可迭代对象的元素，直到predicate为False。

+   groupby(iterable, keyfunc)创建一个迭代器，根据keyfunc()函数返回的结果对项目进行分组。

+   permutations(iterable[, r])返回可迭代对象中项目的连续r长度排列。

+   product(*iterables)返回iterables的笛卡尔积的可迭代对象，而不使用嵌套的for循环。

+   takewhile(predicate, iterable)返回从开始处的可迭代对象的元素，直到predicate为False。

这些函数与操作符模块一起使用特别有用。当结合使用时，itertools和operator可以处理程序员通常依赖lambda处理的大多数情况。以下是使用operator.itemgetter()而不是编写lambda x: x['foo']的示例：

>>> import itertools

>>> a = [{'foo': 'bar'}, {'foo': 'bar', 'x': 42}, {'foo': 'baz', 'y': 43}]

>>> import operator

>>> list(itertools.groupby(a, operator.itemgetter('foo')))

[('bar', <itertools._grouper object at 0xb000d0>), ('baz', <itertools._grouper object at 0xb00110>)]

0xb00110>)]

>>> [(key, list(group)) for key, group in itertools.groupby(a, operator.itemgetter('foo'))]

[('bar', [{'foo': 'bar'}, {'x': 42, 'foo': 'bar'}]), ('baz', [{'y': 43, 'foo': 'baz'}])]

在这种情况下，我们也可以写lambda x: x['foo']，但使用操作符可以避免完全使用lambda。

### **总结**

虽然Python经常被宣传为面向对象的，但它也可以以非常函数式的方式使用。它的许多内置概念，如生成器和列表推导式，是函数式的，并且不与面向对象的方法冲突。它们还限制了程序对全局状态的依赖，为了你自己的好处。

在Python中使用函数式编程作为一种范式可以帮助你使程序更可重用、更易于测试和调试，支持“不重复自己”（DRY）的信条。在这种精神下，标准Python模块itertools和operator是改进你的函数式代码可读性的好工具。
