## **8**

**函数式编程**

![image](img/common01.jpg)

许多 Python 开发者并未意识到在 Python 中可以使用函数式编程的程度，这是很遗憾的：除了少数例外，函数式编程可以让你写出更加简洁和高效的代码。而且，Python 对函数式编程的支持非常广泛。

本章将介绍 Python 中的一些函数式编程方面，包括创建和使用生成器。你将了解一些最有用的函数式包和函数，并学习如何将它们结合使用以获得最高效的代码。

**注意**

*如果你想认真学习函数式编程，以下是我的建议：暂时离开 Python，学习一门功能强大的函数式编程语言，比如 Lisp。我知道在一本 Python 书中谈 Lisp 可能听起来有些奇怪，但我在玩 Lisp 几年后学会了如何“函数式思考”。如果你所有的编程经验都来自命令式编程和面向对象编程，你可能不会发展出充分利用函数式编程所需的思维方式。Lisp 本身并不是纯函数式的，但它对函数式编程的关注比你在 Python 中看到的要多。*

### **创建纯函数**

当你使用函数式风格编写代码时，你的函数设计上不会有副作用：相反，它们接受输入并生成输出，而不会保持状态或修改任何返回值以外的东西。遵循这一理想的函数被称为*纯函数*。

让我们从一个常规的非纯函数示例开始，该函数用于移除列表中的最后一个元素：

```py
def remove_last_item(mylist):
    """Removes the last item from a list."""
    mylist.pop(-1)  # This modifies mylist
```

以下是同一函数的纯粹版本：

```py
def butlast(mylist):

    return mylist[:-1]  # This returns a copy of mylist
```

我们定义了一个 `butlast()` 函数，它的功能类似于 Lisp 中的 `butlast`，返回没有最后一个元素的列表，而*不*修改原始列表。相反，它返回一个已进行修改的列表副本，从而允许我们保留原始列表。

函数式编程的实际优势包括以下几点：

**模块化** 采用函数式编程风格强制在解决各个问题时进行一定程度的分离，使得代码的各个部分更容易在其他上下文中重用。由于函数不依赖于任何外部变量或状态，从不同的代码块中调用它非常直接。

**简洁性** 函数式编程通常比其他编程范式更加简洁。

**并发性** 纯函数是线程安全的，可以并发执行。一些函数式语言会自动实现这一点，这在你需要扩展应用程序时非常有帮助，尽管在 Python 中这还不完全是现状。

**可测试性** 测试一个函数式程序是非常简单的：你只需要一组输入和一组预期的输出。这些输出是*幂等的*，意味着使用相同的参数多次调用同一个函数将始终返回相同的结果。

### **生成器**

*生成器*是一个行为类似于迭代器的对象，它在每次调用`next()`方法时生成并返回一个值，直到引发`StopIteration`为止。生成器首次在 PEP 255 中引入，提供了一种简单的方式来创建实现*迭代器协议*的对象。虽然以函数式风格编写生成器不是严格必要的，但这样做可以使它们更易于编写和调试，这也是一种常见的做法。

要创建生成器，只需编写一个常规的 Python 函数，并包含一个`yield`语句。Python 会检测到`yield`的使用，并将该函数标记为生成器。当执行到`yield`语句时，函数返回一个值，就像`return`语句一样，但有一个显著的区别：解释器会保存一个堆栈引用，这将用于在再次调用`next()`时恢复函数的执行。

当函数执行时，它们的执行链会产生一个*堆栈*——函数调用被认为是堆叠在一起的。当一个函数返回时，它会从堆栈中移除，并将返回值传递给调用函数。对于生成器来说，函数并不真正返回，而是*生成*。因此，Python 会保存函数的状态作为堆栈引用，当需要生成器的下一次迭代时，它会恢复生成器的执行，恢复到保存的点。

#### ***创建生成器***

如前所述，您可以通过编写一个常规函数并在函数体内包含`yield`来创建生成器。示例 8-1 创建了一个名为`mygenerator()`的生成器，其中包含三个`yield`，意味着它将在接下来的三次`next()`调用时进行迭代。

```py
>> def mygenerator():
...     yield 1
...     yield 2
...     yield 'a'
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
  File "<stdin>", line 1, in <module>
StopIteration
```

*示例 8-1：创建一个具有三次迭代的生成器*

当它耗尽`yield`语句时，下一次调用`next()`时会引发`StopIteration`。

在 Python 中，当函数产生某个值时，生成器会保留堆栈的引用，当再次调用`next()`时，它们会恢复这个堆栈。

在不使用生成器时，迭代任何数据的朴素方法是首先构建整个列表，这通常会浪费内存。

假设我们想找到 1 和 10,000,000 之间第一个等于 50,000 的数字。听起来很简单，对吧？让我们给自己一个挑战。我们将在 128MB 内存限制下运行 Python，并尝试首先构建整个列表的朴素方法：

```py
$ ulimit -v 131072
$ python3
>>> a = list(range(10000000))
```

这种朴素方法首先尝试构建列表，但如果我们运行程序到这个阶段：

```py
Traceback (most recent call last): File "<stdin>", line 1, in <module>
MemoryError
```

哎呀，结果我们不能仅凭 128MB 的内存构建一个包含 1000 万个项目的列表！

**警告**

*在 Python 3 中，range()在迭代时返回一个生成器。要在 Python 2 中获取生成器，必须使用 xrange()。这个函数在 Python 3 中已不再存在，因为它是多余的。*

让我们改用生成器试试看，同样的 128MB 限制：

```py
$ ulimit -v 131072
$ python3
>>> for value in range(10000000):
...     if value == 50000:
...             print("Found it")
...             break
...
Found it
```

这次，我们的程序没有出现问题。当它被迭代时，`range()`类返回一个生成器，该生成器动态地生成我们的整数列表。更好的是，由于我们只对第 50,000 个数字感兴趣，因此生成器只需要生成 50,000 个数字就会停止，而不是构建完整的列表。

通过动态生成值，生成器允许你以最小的内存和处理周期消耗处理大型数据集。每当你需要处理大量的值时，生成器可以帮助你高效地处理它们。

#### ***通过 yield 返回和传递值***

`yield`语句还有一个不太常用的功能：它可以像函数调用一样返回一个值。这使得我们可以通过调用生成器的`send()`方法向生成器传递一个值。作为使用`send()`的例子，我们将编写一个名为`shorten()`的函数，它接受一个字符串列表，并返回一个由这些相同字符串构成的列表，只是每个字符串都被截断了（清单 8-2）。

```py
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
        # Truncate the next string depending
        # on the number of vowels in the previous one
        s = shortstringlist.send(number_of_vowels)
        result.append(s)
except StopIteration:
    pass
```

*清单 8-2：使用 send()返回和使用值*

在这个例子中，我们写了一个名为`shorten()`的函数，它接受一个字符串列表，并返回一个由这些相同字符串构成的列表，只是每个字符串都被截断了。每个截断后的字符串长度等于前一个字符串中的元音字母数量：*loremipsum*有四个元音，因此生成器返回的第二个值将是*dolorsit*的前四个字母；*dolo*只有两个元音，因此*ametfoobar*将被截断为前两个字母*am*。然后生成器停止并引发`StopIteration`。因此，我们的生成器返回：

```py
['loremipsum', 'dolo', 'am']
```

以这种方式使用`yield`和`send()`允许 Python 生成器像 Lua 和其他语言中的*协程*一样工作。

PEP 289 引入了生成器表达式，使得可以使用类似列表推导的语法构建单行生成器：

```py
>>> (x.upper() for x in ['hello', 'world'])
<generator object <genexpr> at 0x7ffab3832fa0>
>>> gen = (x.upper() for x in ['hello', 'world'])
>>> list(gen)
['HELLO', 'WORLD']
```

在这个例子中，`gen`是一个生成器，就像我们使用`yield`语句一样。此时的`yield`是隐式的。

#### ***检查生成器***

要判断一个函数是否被认为是生成器，可以使用`inspect.isgeneratorfunction()`。在清单 8-3 中，我们创建了一个简单的生成器并进行了检查。

```py
>>> import inspect
>>> def mygenerator():
...     yield 1
...
>>> inspect.isgeneratorfunction(mygenerator)
True
>>> inspect.isgeneratorfunction(sum)
False
```

*清单 8-3：检查函数是否为生成器*

导入`inspect`包来使用`isgeneratorfunction()`，然后只需将函数的名称传递给它进行检查。阅读`inspect.isgeneratorfunction()`的源代码可以帮助我们了解 Python 是如何标记函数为生成器的（见清单 8-4）。

```py
def isgeneratorfunction(object):
    """Return true if the object is a user-defined generator function.

    Generator function objects provides same attributes as functions.

    See help(isfunction) for attributes listing."""

  return bool((isfunction(object) or ismethod(object)) and
                object.func_code.co_flags & CO_GENERATOR)
```

*清单 8-4：inspect.isgeneratorfunction()的源代码*

`isgeneratorfunction()`函数检查对象是否为函数或方法，并且其代码是否设置了`CO_GENERATOR`标志。这个例子展示了理解 Python 底层工作原理是多么简单。

`inspect` 包提供了 `inspect.getgeneratorstate()` 函数，可以返回生成器的当前状态。我们将在 `mygenerator()` 中的不同执行点上使用它：

```py
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
   Traceback (most recent call last):
     File "<stdin>", line 1, in <module>
   StopIteration
   >>> inspect.getgeneratorstate(gen)
➌ 'GEN_CLOSED'
```

这使我们能够判断生成器是处于首次运行的等待状态（`GEN_CREATED`） ➊，等待通过调用`next()`恢复执行的状态（`GEN_SUSPENDED`） ➋，还是已经运行完毕的状态（`GEN_CLOSED`） ➌。这个功能在调试生成器时可能会很有用。

### **列表推导式**

列表推导式，简称 *listcomp*，允许你在声明列表时直接定义其内容。要将列表转变为列表推导式，必须像平常一样将其放在方括号中，但还要包括一个表达式来生成列表中的项，并加上一个 `for` 循环来遍历这些项。

以下例子没有使用列表推导式创建列表：

```py
>>> x = []
>>> for i in (1, 2, 3):
...     x.append(i)
...
>>> x
[1, 2, 3]
```

下面这个例子使用列表推导式在一行代码中创建相同的列表：

```py
>>> x = [i for i in (1, 2, 3)]
>>> x
[1, 2, 3]
```

使用列表推导式有两个优点：使用列表推导式编写的代码通常更简短，因此会减少 Python 执行的操作次数。与其反复创建列表并调用`append`，Python 可以直接创建包含所有元素的列表，并在一次操作中将它们移动到新列表中。

你可以将多个 `for` 语句结合使用，并使用 `if` 语句来过滤掉某些元素。这里我们创建了一个单词列表，使用列表推导式将每个单词首字母大写，将包含多个单词的项拆分成单个单词，并删除多余的 *or*：

```py
x = [word.capitalize()
     for line in ("hello world?", "world!", "or not")
     for word in line.split()
     if not word.startswith("or")]
>>> x
['Hello', 'World?', 'World!', 'Not']
```

这段代码有两个 `for` 循环：第一个循环遍历文本行，第二个循环遍历每行中的单词。最后一个 `if` 语句过滤掉以 *or* 开头的单词，将它们从最终的列表中排除。

使用列表推导式而非 `for` 循环是一种快速定义列表的简洁方法。由于我们仍在讨论函数式编程，值得注意的是，使用列表推导式创建的列表不应该依赖于改变程序的状态：在构建列表时，不应修改任何变量。这通常使得列表比没有使用列表推导式的列表更简洁、更易读。

请注意，也有类似的语法可以构建字典或集合，像这样：

```py
>>> {x:x.upper() for x in ['hello', 'world']}
{'world': 'WORLD', 'hello': 'HELLO'}
>>> {x.upper() for x in ['hello', 'world']}
set(['WORLD', 'HELLO'])
```

### **功能性函数的运作**

当使用函数式编程处理数据时，你可能会反复遇到相同的一组问题。为了帮助你高效应对这种情况，Python 包含了一些函数来支持函数式编程。本节将快速概述一些内置函数，它们使你能够构建完全函数式的程序。一旦你了解了可用的函数，我鼓励你进一步研究，并尝试将这些函数应用到你自己的代码中。

#### ***使用 map() 将函数应用于元素***

`map()` 函数的形式为 `map(function, iterable)`，并将 `function` 应用到 `iterable` 中的每一项，返回 Python 2 中的一个列表，或 Python 3 中的一个可迭代的 `map` 对象，如 列表 8-5 所示。

```py
>>> map(lambda x: x + "bzz!", ["I think", "I'm good"])
<map object at 0x7fe7101abdd0>
>>> list(map(lambda x: x + "bzz!", ["I think", "I'm good"]))
['I thinkbzz!', "I'm goodbzz!"]
```

*列表 8-5：在 Python 3 中使用 map()*

你可以使用列表推导式编写一个等效的 `map()`，如下所示：

```py
>>> (x + "bzz!" for x in ["I think", "I'm good"])
<generator object <genexpr> at 0x7f9a0d697dc0>
>>> [x + "bzz!" for x in ["I think", "I'm good"]]
['I thinkbzz!', "I'm goodbzz!"]
```

#### ***使用 filter() 过滤列表***

`filter()` 函数的形式为 `filter(function or None, iterable)`，它根据 `function` 返回的结果过滤 `iterable` 中的项。在 Python 2 中，这将返回一个列表，在 Python 3 中则返回一个可迭代的 `filter` 对象：

```py
>>> filter(lambda x: x.startswith("I "), ["I think", "I'm good"])
<filter object at 0x7f9a0d636dd0>
>>> list(filter(lambda x: x.startswith("I "), ["I think", "I'm good"]))
['I think']
```

你也可以使用列表推导式编写一个等效的 `filter()`，如下所示：

```py
>>> (x for x in ["I think", "I'm good"] if x.startswith("I "))
<generator object <genexpr> at 0x7f9a0d697dc0>
>>> [x for x in ["I think", "I'm good"] if x.startswith("I ")]
['I think']
```

#### ***使用 enumerate() 获取索引***

`enumerate()` 函数的形式为 `enumerate(iterable [,start])`，并返回一个可迭代的对象，该对象提供一系列元组，每个元组包含一个整数索引（如果提供了 `start`，则从 `start` 开始）和 `iterable` 中对应的项。当你需要编写引用数组索引的代码时，这个函数非常有用。例如，代替编写如下代码：

```py
i = 0
while i < len(mylist): print("Item %d: %s" % (i, mylist[i]))
    i += 1
```

你可以更高效地使用 `enumerate()` 完成同样的事情，如下所示：

```py
for i, item in enumerate(mylist):
    print("Item %d: %s" % (i, item))
```

#### ***使用 sorted() 排序列表***

`sorted()` 函数的形式为 `sorted(iterable, key=None, reverse=False)`，并返回 `iterable` 的排序版本。`key` 参数允许你提供一个返回排序值的函数，如下所示：

```py
>>> sorted([("a", 2), ("c", 1), ("d", 4)])
[('a', 2), ('c', 1), ('d', 4)]
>>> sorted([("a", 2), ("c", 1), ("d", 4)], key=lambda x: x[1])
[('c', 1), ('a', 2), ('d', 4)]
```

#### ***使用 any() 和 all() 查找满足条件的项***

`any(iterable)` 和 `all(iterable)` 函数根据 `iterable` 返回的值返回布尔值。这些简单的函数等价于以下完整的 Python 代码：

```py
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
```

这些函数对于检查可迭代对象中的任何值或所有值是否满足给定条件非常有用。例如，以下代码检查一个列表是否满足两个条件：

```py
mylist = [0, 1, 3, -1]
if all(map(lambda x: x > 0, mylist)):
    print("All items are greater than 0")
if any(map(lambda x: x > 0, mylist)):
    print("At least one item is greater than 0")
```

这里的区别是，`any()` 在至少一个元素满足条件时返回 `True`，而 `all()` 只有在每个元素都满足条件时才返回 `True`。`all()` 函数对于空的可迭代对象也会返回 `True`，因为没有元素为 `False`。

#### ***使用 zip() 合并列表***

`zip()` 函数的形式为 `zip(iter1 [,iter2 [...]])`。它接受多个序列并将它们合并成元组。当你需要将一组键和一组值合并成字典时，这非常有用。与此处描述的其他函数一样，`zip()` 在 Python 2 中返回一个列表，在 Python 3 中返回一个可迭代的对象。这里我们将一组键映射到一组值以创建字典：

```py
>>> keys = ["foobar", "barzz", "ba!"]
>>> map(len, keys)
<map object at 0x7fc1686100d0>
>>> zip(keys, map(len, keys))
<zip object at 0x7fc16860d440>
>>> list(zip(keys, map(len, keys)))
[('foobar', 6), ('barzz', 5), ('ba!', 3)]
>>> dict(zip(keys, map(len, keys)))
{'foobar': 6, 'barzz': 5, 'ba!': 3}
```

**Python 2 和 3 中的函数式编程函数**

你可能已经注意到 Python 2 和 Python 3 之间的返回类型差异。Python 的大多数纯函数式内置函数在 Python 2 中返回一个列表，而不是可迭代对象，这使得它们在内存使用上不如 Python 3 的*x*版本高效。如果你打算使用这些函数编写代码，请记住在 Python 3 中你将获得最大收益。如果你仍然使用 Python 2，不用担心：标准库中的`itertools`模块提供了这些函数的基于迭代器的版本（如`itertools.izip()`、`itertools.imap()`、`itertools.ifilter()`等）。

#### ***一个常见问题的解决方案***

还有一个重要的工具需要介绍。在处理列表时，我们经常想找到第一个满足特定条件的项。我们将看看多种实现方法，并最终找到最有效的方法：`first`包。

##### **使用简单代码找到项**

我们或许能够通过这样的函数找到第一个满足条件的项：

```py
def first_positive_number(numbers):
    for n in numbers:
        if n > 0:
            return n
```

我们可以像这样以函数式风格重写`first_positive_number()`函数：

```py
def first(predicate, items):
    for item in items:
        if predicate(item):
            return item

first(lambda x: x > 0, [-1, 0, 1, 2])
```

通过使用函数式方法并将谓词作为参数传递，函数变得非常可复用。我们甚至可以像这样更简洁地编写它：

```py
# Less efficient
list(filter(lambda x: x > 0, [-1, 0, 1, 2]))[0]
# Efficient
next(filter(lambda x: x > 0, [-1, 0, 1, 2]))
```

请注意，如果没有项满足条件，这可能会引发`IndexError`，导致`list(filter())`返回一个空列表。

对于简单的情况，你可以依赖`next()`来防止发生`IndexError`，像这样：

```py
>>> a = range(10)
>>> next(x for x in a if x > 3)
4
```

示例 8-6 会在条件永远无法满足时抛出`StopIteration`。这也可以通过为`next()`添加第二个参数来解决，像这样：

```py
>>> a = range(10)
>>> next((x for x in a if x > 10), 'default')
'default'
```

*示例 8-6：在条件不满足时返回默认值*

当条件无法满足时，这将返回一个默认值，而不是抛出错误。幸运的是，Python 提供了一个包来处理这一切。

##### **使用 first()找到项**

与其在所有程序中都编写示例 8-6 中的函数，不如引入一个小型 Python 包`first`。示例 8-7 展示了该包如何帮助你找到匹配条件的可迭代对象的第一个元素。

```py
>>> from first import first
>>> first([0, False, None, [], (), 42])
42
>>> first([-1, 0, 1, 2])
-1
>>> first([-1, 0, 1, 2], key=lambda x: x > 0)
1
```

*示例 8-7：找到列表中第一个满足条件的项*

你可以看到，`first()`函数返回列表中第一个有效的、非空的项。

##### **使用 lambda()与 functools**

你会注意到，在本章的许多例子中，我们已经使用了`lambda()`。`lambda()`函数是为方便函数式编程而添加到 Python 中的，用于`map()`和`filter()`等函数，否则每次想要检查不同的条件时，你都需要编写一个全新的函数。示例 8-8 与示例 8-7 等效，但没有使用`lambda()`。

```py
import operator
from first import first

def greater_than_zero(number):
    return number > 0

first([-1, 0, 1, 2], key=greater_than_zero)
```

*示例 8-8：找到满足条件的第一个项，不使用 lambda()*

这段代码的工作原理与示例 8-7 中的代码相同，返回列表中满足条件的第一个非空值，但它要更加繁琐：如果我们想要得到序列中第一个超过 42 个项目的数字，我们就需要通过`def`来定义一个适当的函数，而不是直接在调用`first()`时内联定义它。

尽管`lambda`有助于我们避免类似的情况，但它仍然存在问题。`first`模块包含一个`key`参数，可以用于提供一个函数，该函数接收每个项作为参数并返回一个布尔值，表示是否满足条件。然而，我们不能传递`key`函数，因为它需要超过一行的代码：`lambda`语句不能跨越多行。这是`lambda`的一个重大限制。

相反，我们必须回到繁琐的方式，为每个我们需要的`key`编写新的函数定义。或者我们就不需要这样做吗？

`functools`包通过其`partial()`方法为我们提供了帮助，它为我们提供了一个比`lambda`更灵活的替代方案。`functools.partial()`方法允许我们创建一个带有“变形”的包装函数：它不是改变函数的行为，而是改变它接收的*参数*，就像这样：

```py
   from functools import partial
   from first import first

➊ def greater_than(number, min=0):
       return number > min

➋ first([-1, 0, 1, 2], key=partial(greater_than, min=42))
```

在这里，我们创建了一个新的`greater_than()`函数，它的默认行为与示例 8-8 中的旧版`greater_than_zero()`函数一样，但这个版本允许我们指定要与其比较的值，而之前的版本是硬编码的。在这里，我们将`functools.partial()`传递给我们的函数，并指定我们想要的`min`值➊，然后我们得到一个新的函数，这个函数将`min`设置为 42，正如我们想要的那样➋。换句话说，我们可以编写一个函数，并使用`functools.partial()`来定制我们新函数的行为，以适应任何特定情况下的需求。

即使是这个版本也可以简化。我们在这个例子中所做的只是比较两个数字，事实证明，`operator`模块正好为此提供了内置函数：

```py
import operator
from functools import partial
from first import first

first([-1, 0, 1, 2], key=partial(operator.le, 0))
```

这是一个很好的例子，展示了`functools.partial()`如何与位置参数一起使用。在这个例子中，函数`operator.le(a, b)`接受两个数字并返回一个布尔值，告诉我们第一个数字是否小于或等于第二个数字。我们将其传递给`functools.partial()`，传递给`functools.partial()`的 0 被分配给`a`，而传递给`functools.partial()`返回的函数的参数则被分配给`b`。因此，这个例子与示例 8-8 中的代码完全相同，但不使用`lambda`或定义任何额外的函数。

**注意**

*`functools.partial()`方法通常用于替代 lambda 表达式，并且应被视为一种更优秀的替代方案。lambda 函数在 Python 语言中算是一种例外，并且由于该函数的体积限制（只能写单行代码），Python 3 中曾考虑过完全去除它。*

#### ***有用的 itertools 函数***

最后，我们将看看 Python 标准库中 `itertools` 模块中的一些有用函数，你应该了解它们。很多程序员最终会编写这些函数的自定义版本，仅仅因为他们不知道 Python 提供了这些现成的函数。它们都旨在帮助你操作 `iterator`（这就是该模块名为 *iter-tools* 的原因），因此它们都是纯粹的函数式函数。我将在这里列出其中的一些，并简要介绍它们的功能，鼓励你在有需要时进一步研究它们。

+   `accumulate(iterable`[, func]) 返回来自可迭代对象的项目累积和序列。

+   `chain(*iterables`) 依次遍历多个可迭代对象，而无需构建一个包含所有项目的中间列表。

+   `combinations(iterable`, r) 生成从给定 `iterable` 中长度为 r 的所有组合。

+   `compress(data`, selectors) 将 selectors 中的布尔掩码应用于 data，只返回 data 中对应 selectors 元素为 `True` 的值。

+   `count(start`, step) 生成一个无限的数值序列，从 start 开始，每次调用时递增 step。

+   `cycle(iterable`) 循环遍历 iterable 中的值。

+   `repeat(elem`[, n]) 重复元素 elem，重复 n 次。

+   `dropwhile(predicate`, iterable) 从开始筛选可迭代对象的元素，直到谓词为 `False`。

+   `groupby(iterable`, keyfunc) 创建一个迭代器，根据 `keyfunc()` 函数返回的结果对项目进行分组。

+   `permutations(iterable`[, r]) 返回 iterable 中项目的连续 r 长度排列。

+   `product(*iterables`) 返回一个可迭代对象，包含多个可迭代对象的笛卡尔积，而不需要使用嵌套的 `for` 循环。

+   `takewhile(predicate`, iterable) 从开始返回可迭代对象的元素，直到谓词为 `False`。

这些函数在与 `operator` 模块结合使用时特别有用。将 `itertools` 和 `operator` 一起使用，可以处理大多数程序员通常依赖 `lambda` 的情况。以下是一个示例，展示了如何使用 `operator.itemgetter()` 来代替编写 `lambda x: x['foo']`：

```py
>>> import itertools
>>> a = [{'foo': 'bar'}, {'foo': 'bar', 'x': 42}, {'foo': 'baz', 'y': 43}]
>>> import operator
>>> list(itertools.groupby(a, operator.itemgetter('foo')))
[('bar', <itertools._grouper object at 0xb000d0>), ('baz', <itertools._grouper object at
0xb00110>)]
>>> [(key, list(group)) for key, group in itertools.groupby(a, operator.itemgetter('foo'))]
[('bar', [{'foo': 'bar'}, {'x': 42, 'foo': 'bar'}]), ('baz', [{'y': 43, 'foo': 'baz'}])]
```

在这种情况下，我们也可以写作 `lambda x: x['foo']`，但是使用 `operator` 可以避免完全使用 `lambda`。

### **总结**

虽然 Python 通常被宣传为面向对象的语言，但它也可以以非常函数化的方式使用。它的许多内建概念，如生成器和列表推导式，都是函数式的，并且与面向对象的方法不冲突。它们还减少了对程序全局状态的依赖，这对你来说是有益的。

使用函数式编程范式在 Python 中可以帮助你让程序更具可重用性，更容易测试和调试，支持“不要重复自己”（DRY）原则。在这个精神下，标准 Python 模块 `itertools` 和 `operator` 是改善函数式代码可读性的好工具。
