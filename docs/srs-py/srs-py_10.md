## 第十章：**性能与优化**

![image](img/common01.jpg)

优化通常不是开发过程中最先考虑的事情，但总有那么一刻，优化以提高性能会变得合适。这并不是说你应该以程序会慢为前提来编写程序，而是在没有首先弄清楚应该使用哪些工具并进行适当的性能分析的情况下考虑优化，是浪费时间。正如唐纳德·克努斯所写的，“过早的优化是万恶之源。”^(1)

在这里，我将向你展示如何使用正确的方法编写快速的代码，以及在需要更多优化时该从哪里着手。许多开发者试图猜测 Python 可能在哪些地方更慢或更快。与其猜测，本章将帮助你理解如何分析你的应用程序，以便你知道程序的哪些部分在拖慢速度，瓶颈在哪里。

### **数据结构**

大多数编程问题可以通过正确的数据结构以简洁优雅的方式解决——Python 提供了许多数据结构供你选择。学会利用这些现有的数据结构，能比编写自定义数据结构提供更简洁、更稳定的解决方案。

例如，每个人都会使用`dict`，但是你多少次见到代码试图通过捕获`KeyError`异常来访问字典，如下所示：

```py
def get_fruits(basket, fruit):
    try:
        return basket[fruit]
    except KeyError:
        return None
```

或者通过先检查键是否存在：

```py
def get_fruits(basket, fruit):
    if fruit in basket:
        return basket[fruit]
```

如果你使用`dict`类已经提供的`get()`方法，你可以避免捕获异常或在一开始就检查键是否存在：

```py
def get_fruits(basket, fruit):
    return basket.get(fruit)
```

`dict.get()`方法还可以返回一个默认值，而不是`None`；只需传入第二个参数：

```py
def get_fruits(basket, fruit):
    # Return the fruit, or Banana if the fruit cannot be found.
    return basket.get(fruit, Banana())
```

许多开发者在使用基础的 Python 数据结构时，常常没有意识到它们提供的所有方法。集合也是如此；集合数据结构中的方法可以解决许多本来需要编写嵌套`for`/`if`块来处理的问题。例如，开发者常常使用`for`/`if`循环来判断某个项是否在列表中，如下所示：

```py
def has_invalid_fields(fields):
    for field in fields: if field not in ['foo', 'bar']:
            return True
    return False
```

这个循环遍历列表中的每一项，检查所有项是否是`foo`或`bar`。但是，你可以通过更高效的方式编写代码，避免使用循环：

```py
def has_invalid_fields(fields):
    return bool(set(fields) - set(['foo', 'bar']))
```

这段代码将字段转换为集合，并通过从`set(['foo', 'bar'])`中减去其差集来获取其余部分。然后，它将集合转换为布尔值，表示是否剩下任何非`foo`和`bar`的项。通过使用集合，不再需要遍历任何列表逐一检查项。Python 内部对两个集合进行的单次操作要更快。

Python 还提供了更先进的数据结构，可以大大减轻代码维护的负担。例如，看看列表 10-1。

```py
def add_animal_in_family(species, animal, family):
    if family not in species:
        species[family] = set()
    species[family].add(animal)

species = {}
add_animal_in_family(species, 'cat', 'felidea')
```

*列表 10-1：向集合字典中添加一个条目*

这段代码是完全有效的，但你的程序会有多少次需要清单 10-1 的变种呢？几十次？几百次？

Python 提供了`collections.defaultdict`结构，它以优雅的方式解决了这个问题：

```py
import collections

def add_animal_in_family(species, animal, family):
    species[family].add(animal)

species = collections.defaultdict(set)
add_animal_in_family(species, 'cat', 'felidea')
```

每次你尝试从`dict`中访问一个不存在的项时，`defaultdict`会使用作为参数传递给其构造函数的函数来构建一个新值，而不是引发`KeyError`。在这种情况下，`set()`函数用于每次我们需要时构建一个新的`set`。

`collections`模块提供了几种额外的数据结构，可以用来解决其他类型的问题。例如，假设你想要统计一个可迭代对象中不同项的数量。让我们看一下`collections.Counter()`方法，它提供了解决这个问题的方法：

```py
>>> import collections
>>> c = collections.Counter("Premature optimization is the root of all evil.")
>>> c
>>> c['P']  # Returns the name of occurrence of the letter 'P'
1
>>> c['e']  # Returns the name of occurrence of the letter 'e'
4
>>> c.most_common(2)  # Returns the 2 most common letters
[(' ', 7), ('i', 5)]
```

`collections.Counter`对象适用于任何具有可哈希项的可迭代对象，免去了你编写自己的计数函数的需求。它可以轻松计算字符串中字母的数量，并返回可迭代对象中最常见的前*n*个项。如果你没有意识到 Python 标准库已经提供了这个功能，可能你会尝试自己实现类似的功能。

使用正确的数据结构、正确的方法，以及显然——足够的算法，你的程序应该表现良好。然而，如果它的表现不够好，获得有关程序在哪些地方可能变慢并需要优化的线索，最好的方法是对代码进行性能分析。

### **通过性能分析理解行为**

*性能分析*是一种动态程序分析方法，它让我们了解程序的行为。它使我们能够确定哪些地方可能存在瓶颈并需要优化。程序的性能分析结果以一组统计数据的形式呈现，描述了程序各部分执行的频率和持续时间。

Python 提供了一些工具来分析程序的性能。其中一个是`cProfile`，它是 Python 标准库的一部分，不需要额外安装。我们还将查看`dis`模块，它可以将 Python 代码拆解成更小的部分，使我们更容易理解底层发生了什么。

#### ***cProfile***

自 Python 2.5 起，Python 默认包括了`cProfile`。要使用`cProfile`，可以使用语法`python –m cProfile <program>`来调用它。这将加载并启用`cProfile`模块，然后按正常方式运行程序，并启用性能分析，如清单 10-2 所示。

```py
$ python -m cProfile myscript.py
         343 function calls (342 primitive calls) in 0.000 seconds

   Ordered by: standard name ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    0.000    0.000 :0(_getframe)
        1    0.000    0.000    0.000    0.000 :0(len)
      104    0.000    0.000    0.000    0.000 :0(setattr)
        1    0.000    0.000    0.000    0.000 :0(setprofile)
        1    0.000    0.000    0.000    0.000 :0(startswith)
      2/1    0.000    0.000    0.000    0.000 <string>:1(<module>)
        1    0.000    0.000    0.000    0.000 StringIO.py:30(<module>)
        1    0.000    0.000    0.000    0.000 StringIO.py:42(StringIO)
```

*清单 10-2：使用 cProfile 对 Python 脚本进行的默认输出*

清单 10-2 展示了使用`cProfile`运行一个简单脚本的输出。它告诉你程序中每个函数被调用的次数以及执行花费的时间。你还可以使用`-s`选项按其他字段排序；例如，`-s time`将按内部时间排序结果。

我们可以使用一个非常棒的工具叫做 KCacheGrind 来可视化 `cProfile` 生成的信息。这个工具最初是为处理 C 语言程序而设计的，但幸运的是，我们可以通过将数据转换为调用树，来将其用于 Python 数据。

`cProfile` 模块有一个 `-o` 选项，允许你保存分析数据，而 `pyprof2calltree` 可以将数据从一种格式转换为另一种格式。首先，使用以下命令安装转换器：

```py
$ pip install pyprof2calltree
```

然后运行转换器，如清单 10-3 所示，既转换数据（`-i` 选项），又使用转换后的数据运行 KCacheGrind（`-k` 选项）。

```py
$ python -m cProfile -o myscript.cprof myscript.py
$ pyprof2calltree -k -i myscript.cprof
```

*清单 10-3：运行 cProfile 并启动 KCacheGrind*

一旦 KCacheGrind 打开，它将显示类似于图 10-1 中的信息。通过这些可视化结果，你可以使用调用图来跟踪每个函数所消耗的时间百分比，从而确定程序中可能消耗过多资源的部分。

阅读 KCacheGrind 的最简单方法是从屏幕左侧的表格开始，表格列出了程序执行的所有函数和方法。你可以按执行时间对它们进行排序，然后找出消耗最多 CPU 时间的函数，点击它进行查看。

KCacheGrind 的右侧面板会显示调用该函数的其他函数以及调用的次数，还会展示该函数调用的其他函数。你可以轻松导航程序的调用图，包括每个部分的执行时间。

这样可以帮助你更好地理解哪些部分的代码可能需要优化。如何优化代码取决于你自己的选择，并且取决于你的程序需要实现的目标！

![image](img/f10-01.jpg)

*图 10-1：KCacheGrind 输出示例*

虽然通过获取程序运行信息并进行可视化可以让你从宏观上了解程序，但你可能需要更细致地查看某些代码部分，以便更仔细地检查其元素。在这种情况下，我发现依赖 `dis` 模块来了解背后的运行机制是更好的选择。

#### ***使用 dis 模块进行反汇编***

`dis` 模块是一个 Python 字节码反汇编器。将代码拆解开来可以帮助理解每一行背后的运行机制，从而更好地优化它。例如，清单 10-4 展示了 `dis.dis()` 函数，它反汇编传入的函数，并打印出该函数运行时的字节码指令列表。

```py
>>> def x():
...     return 42
...
>>> import dis
>>> dis.dis(x)
  2           0 LOAD_CONST               1 (42)
              3 RETURN_VALUE
```

*清单 10-4：反汇编一个函数*

在清单 10-4 中，函数 `x` 被反汇编，并打印出由字节码指令构成的组成部分。这里只有两个操作：加载常量（`LOAD_CONST`），它的值是 `42`，然后返回该值（`RETURN_VALUE`）。

为了看到`dis`的实际效果及其如何发挥作用，我们将定义两个执行相同操作的函数——连接三个字母——并对它们进行反汇编，以查看它们如何以不同方式完成任务：

```py
abc = ('a', 'b', 'c')

def concat_a_1():
    for letter in abc:
            abc[0] + letter

def concat_a_2():
    a = abc[0]
    for letter in abc:
            a + letter
```

这两个函数看起来做的事情相同，但如果我们使用`dis.dis`进行反汇编，正如在清单 10-5 中所示，我们会看到生成的字节码稍有不同。

```py
>>> dis.dis(concat_a_1)
  2           0 SETUP_LOOP              26 (to 29)
              3 LOAD_GLOBAL              0 (abc)
              6 GET_ITER
        >>    7 FOR_ITER                18 (to 28)
             10 STORE_FAST               0 (letter)

  3          13 LOAD_GLOBAL              0 (abc)
             16 LOAD_CONST               1 (0)
             19 BINARY_SUBSCR
             20 LOAD_FAST                0 (letter)
             23 BINARY_ADD
             24 POP_TOP
             25 JUMP_ABSOLUTE            7
        >>   28 POP_BLOCK
        >>   29 LOAD_CONST               0 (None)
             32 RETURN_VALUE
>>> dis.dis(concat_a_2)
  2           0 LOAD_GLOBAL              0 (abc)
              3 LOAD_CONST               1 (0)
              6 BINARY_SUBSCR
              7 STORE_FAST               0 (a)

  3          10 SETUP_LOOP              22 (to 35)
             13 LOAD_GLOBAL              0 (abc)
             16 GET_ITER
        >>   17 FOR_ITER                14 (to 34)
             20 STORE_FAST               1 (letter)

  4          23 LOAD_FAST                0 (a)
             26 LOAD_FAST                1 (letter)
             29 BINARY_ADD
             30 POP_TOP
             31 JUMP_ABSOLUTE           17
        >>   34 POP_BLOCK >>   35 LOAD_CONST               0 (None)
             38 RETURN_VALUE
```

*清单 10-5：反汇编连接字符串的函数*

在清单 10-5 中的第二个函数中，我们在运行循环之前将`abc[0]`存储在一个临时变量中。这使得在循环内执行的字节码比第一个函数的字节码稍微小一些，因为我们避免了在每次迭代中都要执行`abc[0]`查找。使用`timeit`测试后，第二个版本比第一个函数快 10%，执行时间少了一整微秒！显然，除非你调用这个函数达到数十亿次，否则这个微秒的优化并不值得，但这正是`dis`模块可以提供的洞察。

是否依赖于像在循环外存储值这样的“技巧”取决于具体情况——最终，应该由编译器来优化这种事情。另一方面，由于 Python 强烈依赖动态特性，编译器很难确定优化不会带来负面副作用。在清单 10-5 中，使用`abc[0]`会调用`abc.__getitem__`，如果它被继承覆盖，可能会产生副作用。根据你使用的函数版本，`abc.__getitem__`方法可能会被调用一次或多次，这可能会有所不同。因此，在编写和优化代码时要小心！

### **高效定义函数**

在审查代码时，我发现的一个常见错误是函数内部定义函数。这是低效的，因为函数会被重复和无谓地重新定义。例如，清单 10-6 展示了`y()`函数被多次定义。

```py
>> import dis
>>> def x():
...     return 42
...
>>> dis.dis(x)
  2           0 LOAD_CONST               1 (42)
              3 RETURN_VALUE
>>> def x():
...     def y():
...             return 42
...     return y()
...
>>> dis.dis(x)
  2           0 LOAD_CONST               1 (<code object y at
x100ce7e30, file "<stdin>", line 2>)
              3 MAKE_FUNCTION            0
              6 STORE_FAST               0 (y)
  4           9 LOAD_FAST                0 (y) 12 CALL_FUNCTION            0
             15 RETURN_VALUE
```

*清单 10-6：函数重定义*

清单 10-6 展示了`MAKE_FUNCTION`、`STORE_FAST`、`LOAD_FAST`和`CALL_FUNCTION`的调用，它们所需的操作码比清单 10-4 中返回`42`所需的要多得多。

你唯一需要在函数内定义函数的情况是构建函数闭包，这是 Python 的操作码中一个明确的使用场景，通过`LOAD_CLOSURE`可以看到，正如在清单 10-7 中所示。

```py
>>> def x():
...     a = 42
...     def y():
...             return a
...     return y()
...
>>> dis.dis(x)
  2           0 LOAD_CONST               1 (42)
              3 STORE_DEREF              0 (a)

  3           6 LOAD_CLOSURE             0 (a)
              9 BUILD_TUPLE              1
             12 LOAD_CONST               2 (<code object y at
x100d139b0, file "<stdin>", line 3>)
             15 MAKE_CLOSURE             0
             18 STORE_FAST               0 (y)

  5          21 LOAD_FAST                0 (y)
             24 CALL_FUNCTION            0
             27 RETURN_VALUE
```

*清单 10-7：定义一个闭包*

虽然你可能不会每天都用到它，但反汇编代码是一个非常有用的工具，当你想要更仔细地了解内部发生了什么时。

### **有序列表与二分法**

接下来，我们来看看如何优化列表。如果一个列表是无序的，查找某一特定项在列表中的位置的最坏情况复杂度为*O(n)*，这意味着在最坏的情况下，你需要遍历列表中的每一项才能找到目标项。

优化这个问题的常用解决方案是改用*排序*列表。排序列表使用二分查找算法来实现查找，时间复杂度为*O(log n)*。其思想是递归地将列表一分为二，看看项应该出现在左侧还是右侧，然后决定下一步应该查找哪一侧。

Python 提供了`bisect`模块，其中包含了一个二分查找算法，如清单 10-8 所示。

```py
>>> farm = sorted(['haystack', 'needle', 'cow', 'pig'])
>>> bisect.bisect(farm, 'needle')
3
>>> bisect.bisect_left(farm, 'needle')
2
>>> bisect.bisect(farm, 'chicken')
0
>>> bisect.bisect_left(farm, 'chicken')
0
>>> bisect.bisect(farm, 'eggs')
1
>>> bisect.bisect_left(farm, 'eggs')
1
```

*清单 10-8：使用 bisect 查找大海捞针*

如清单 10-8 所示，`bisect.bisect()`函数返回一个元素应该插入的位置，以保持列表的有序。显然，这只有在列表已经正确排序的情况下才有效。初始排序允许我们获得一个项的*理论*索引：`bisect()`并不会返回该项是否在列表中，而是返回该项如果在列表中应该位于的位置。通过在此索引处检索项，我们可以回答该项是否存在于列表中的问题。

如果你希望立即将元素插入到正确的排序位置，`bisect`模块提供了`insort_left()`和`insort_right()`函数，如清单 10-9 所示。

```py
>>> farm
['cow', 'haystack', 'needle', 'pig']
>>> bisect.insort(farm, 'eggs')
>>> farm
['cow', 'eggs', 'haystack', 'needle', 'pig']
>>> bisect.insort(farm, 'turkey')
>>> farm
['cow', 'eggs', 'haystack', 'needle', 'pig', 'turkey']
```

*清单 10-9：在排序列表中插入一个项*

使用`bisect`模块，你还可以创建一个特殊的`SortedList`类，继承自`list`，来创建一个始终保持排序的列表，如清单 10-10 所示：

```py
import bisect
import unittest

class SortedList(list):
    def __init__(self, iterable):
        super(SortedList, self).__init__(sorted(iterable))

    def insort(self, item):
        bisect.insort(self, item) def extend(self, other):
        for item in other:
            self.insort(item)

    @staticmethod
    def append(o):
        raise RuntimeError("Cannot append to a sorted list")

    def index(self, value, start=None, stop=None):
        place = bisect.bisect_left(self[start:stop], value)
        if start:
            place += start
        end = stop or len(self)
        if place < end and self[place] == value:
            return place
        raise ValueError("%s is not in list" % value)

class TestSortedList(unittest.TestCase):
    def setUp(self):
        self.mylist = SortedList(
            ['a', 'c', 'd', 'x', 'f', 'g', 'w']
        )

    def test_sorted_init(self):
        self.assertEqual(sorted(['a', 'c', 'd', 'x', 'f', 'g', 'w']),
                         self.mylist)

    def test_sorted_insort(self):
        self.mylist.insort('z')
        self.assertEqual(['a', 'c', 'd', 'f', 'g', 'w', 'x', 'z'],
                         self.mylist)
        self.mylist.insort('b')
        self.assertEqual(['a', 'b', 'c', 'd', 'f', 'g', 'w', 'x', 'z'],
                         self.mylist)

    def test_index(self):
        self.assertEqual(0, self.mylist.index('a'))
        self.assertEqual(1, self.mylist.index('c'))
        self.assertEqual(5, self.mylist.index('w'))
        self.assertEqual(0, self.mylist.index('a', stop=0))
        self.assertEqual(0, self.mylist.index('a', stop=2))
        self.assertEqual(0, self.mylist.index('a', stop=20))
        self.assertRaises(ValueError, self.mylist.index, 'w', stop=3)
        self.assertRaises(ValueError, self.mylist.index, 'a', start=3)
        self.assertRaises(ValueError, self.mylist.index, 'a', start=333)

    def test_extend(self):
        self.mylist.extend(['b', 'h', 'j', 'c'])
        self.assertEqual(
            ['a', 'b', 'c', 'c', 'd', 'f', 'g', 'h', 'j', 'w', 'x']
            self.mylist)
```

*清单 10-10：一个 SortedList 对象的实现*

使用像这样的`list`类，在插入项时稍微慢一些，因为程序需要寻找插入的正确位置。然而，这个类在使用`index()`方法时比其父类更快。显然，不应该在这个类上使用`list.append()`方法：你不能在列表的末尾添加项，否则它可能会变得无序！

许多 Python 库为更多的数据类型实现了多种版本的清单 10-10，例如二叉树或红黑树结构。`blist`和`bintree` Python 包包含可用于这些目的的代码，是实现和调试自己版本的便捷替代方案。

在接下来的章节中，我们将看到如何利用 Python 提供的原生元组数据类型，使你的 Python 代码变得更快。

### **namedtuple 和 Slots**

在编程中，通常需要创建一些简单的对象，它们只有少数几个固定的属性。一个简单的实现可能是这样的：

```py
class Point(object):
    def __init__(self, x, y):
        self.x = x
        self.y = y
```

这肯定能完成任务。然而，这种方法也有一个缺点。在这里，我们创建了一个继承自对象类的类，因此通过使用这个`Point`类，你会实例化完整的对象并分配大量的内存。

在 Python 中，常规对象将其所有属性存储在一个字典中，而这个字典本身存储在`__dict__`属性中，如清单 10-11 所示。

```py
>>> p = Point(1, 2)
>>> p.__dict__
{'y': 2, 'x': 1}
>>> p.z = 42
>>> p.z
42
>>> p.__dict__
{'y': 2, 'x': 1, 'z': 42}
```

*清单 10-11：Python 对象中属性的内部存储方式*

对于 Python 来说，使用`dict`的优势在于它允许你向对象添加任意数量的属性。缺点是，使用字典来存储这些属性在内存方面开销很大——你需要存储对象、键、值引用以及其他所有内容。这使得创建和操作变得缓慢，并且内存成本很高。

作为这种不必要内存使用的示例，请考虑以下简单的类：

```py
class Foobar(object):
    def __init__(self, x):
        self.x = x
```

这创建了一个简单的`Point`对象，其中包含一个名为`x`的属性。让我们使用`memory_profiler`来检查这个类的内存使用情况，这是一个非常实用的 Python 包，允许我们逐行查看程序的内存使用情况，且有一个小脚本可以创建 100,000 个对象，如清单 10-12 所示。

```py
$ python -m memory_profiler object.py
Filename: object.py

Line #    Mem usage    Increment   Line Contents
     5                             @profile
     6     9.879 MB     0.000 MB   def main():
     7    50.289 MB    40.410 MB       f = [ Foobar(42) for i in range(100000) ]
```

*清单 10-12：在使用对象的脚本中使用 memory_profiler*

清单 10-12 演示了创建`Foobar`类的 100,000 个对象会消耗 40MB 的内存。虽然每个对象 400 字节可能不算大，但当你创建成千上万个对象时，内存消耗就会逐渐累积。

有一种方法可以避免使用`dict`的默认行为：Python 中的类可以定义一个`__slots__`属性，列出该类实例允许的属性。这样， instead of 分配整个字典对象来存储对象属性，你可以使用一个*列表*对象来存储它们。

如果你查看 CPython 的源代码并查看 *Objects/typeobject.c* 文件，理解当类上设置了`__slots__`时 Python 所做的事情会很容易。清单 10-13 是处理此功能的函数的简化版：

```py
static PyObject *
type_new(PyTypeObject *metatype, PyObject *args, PyObject *kwds)
{
    --snip--
    /* Check for a __slots__ sequence variable in dict, and count it */
    slots = _PyDict_GetItemId(dict, &PyId___slots__);
    nslots = 0;
    if (slots == NULL) {
        if (may_add_dict)
            add_dict++;
        if (may_add_weak)
            add_weak++;
    } else {
        /* Have slots */
        /* Make it into a tuple */
        if (PyUnicode_Check(slots))
            slots = PyTuple_Pack(1, slots);
        else
            slots = PySequence_Tuple(slots);
        /* Are slots allowed? */
        nslots = PyTuple_GET_SIZE(slots);
        if (nslots > 0 && base->tp_itemsize != 0) {
            PyErr_Format(PyExc_TypeError,
                         "nonempty __slots__ "
                         "not supported for subtype of '%s'",
                         base->tp_name);
            goto error;
        }
        /* Copy slots into a list, mangle names and sort them.
           Sorted names are needed for __class__ assignment.
           Convert them back to tuple at the end.
        */
        newslots = PyList_New(nslots - add_dict - add_weak);
        if (newslots == NULL)
            goto error;
        if (PyList_Sort(newslots) == -1) {
            Py_DECREF(newslots);
            goto error;
        }
        slots = PyList_AsTuple(newslots);
        Py_DECREF(newslots);
        if (slots == NULL)
            goto error;
    }
    /* Allocate the type object */
    type = (PyTypeObject *)metatype->tp_alloc(metatype, nslots);
    --snip--
    /* Keep name and slots alive in the extended type object */
    et = (PyHeapTypeObject *)type;
    Py_INCREF(name);
    et->ht_name = name;
    et->ht_slots = slots;
    slots = NULL;
    --snip--
    return (PyObject *)type;
```

*清单 10-13：来自 Objects/typeobject.c 的摘录*

正如你在清单 10-13 中看到的，Python 会将`__slots__`的内容转换为元组，然后再转换为列表，之后它会构建并排序列表，然后再将列表转换回元组用于使用和存储在类中。通过这种方式，Python 能够快速检索值，而无需分配和使用整个字典。

声明并使用这样的类相当简单。你需要做的就是将`__slots__`属性设置为一个列出将要在类中定义的属性的列表：

```py
class Foobar(object):
    __slots__ = ('x',)

    def __init__(self, x):
        self.x = x
```

我们可以使用 `memory_profiler` Python 包来比较这两种方法的内存使用情况，如清单 10-14 所示。

```py
% python -m memory_profiler slots.py
Filename: slots.py

Line #    Mem usage    Increment   Line Contents
     7                             @profile
     8     9.879 MB     0.000 MB   def main():
     9    21.609 MB    11.730 MB       f = [ Foobar(42) for i in range(100000) ]
```

*清单 10-14：在使用 __slots__ 的脚本上运行 memory_profiler*

清单 10-14 显示了这次创建 100,000 个对象所需的内存不到 12MB——每个对象少于 120 字节。因此，通过使用 Python 类的 `__slots__` 属性，我们可以减少内存使用量，因此当我们创建大量简单对象时，`__slots__` 属性是一个有效且高效的选择。然而，这种技巧不应当用于通过硬编码每个类的属性列表来执行静态类型检查：这样做不符合 Python 程序的精神。

这里的缺点是属性列表现在是固定的。在运行时，不能向 `Foobar` 类添加任何新属性。由于属性列表的固定特性，我们很容易想象出那些其属性始终有值且字段总是按某种方式排序的类。

这正是 `collection` 模块中的 `namedtuple` 类所实现的功能。这个 `namedtuple` 类允许我们动态创建一个类，该类将继承自元组类，从而共享诸如不可变和具有固定数量条目等特性。

与其通过索引引用它们，`namedtuple` 提供了通过引用命名属性来检索元组元素的能力。这使得元组对人类更易于访问，正如在 清单 10-15 中所示。

```py
>>> import collections
>>> Foobar = collections.namedtuple('Foobar', ['x'])
>>> Foobar = collections.namedtuple('Foobar', ['x', 'y'])
>>> Foobar(42, 43) Foobar(x=42, y=43)
>>> Foobar(42, 43).x
42
>>> Foobar(42, 43).x = 44
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: can't set attribute
>>> Foobar(42, 43).z = 0
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'Foobar' object has no attribute 'z'
>>> list(Foobar(42, 43))
[42, 43]
```

*清单 10-15：使用 namedtuple 来引用元组元素*

清单 10-15 显示了如何用一行代码创建一个简单的类并实例化它。我们不能更改该类对象的任何属性或向其添加属性，既因为该类继承自 `namedtuple`，也因为 `__slots__` 的值被设置为空元组，避免了 `__dict__` 的创建。由于此类继承自元组，我们可以轻松地将其转换为列表。

清单 10-16 演示了 `namedtuple` 类工厂的内存使用情况。

```py
% python -m memory_profiler namedtuple.py
Filename: namedtuple.py

Line #    Mem usage    Increment   Line Contents
     4                             @profile
     5     9.895 MB     0.000 MB   def main():
     6    23.184 MB    13.289 MB       f = [ Foobar(42) for i in range(100000) ]
```

*清单 10-16：使用 namedtuple 来运行 memory_profiler 对脚本进行分析*

对于 100,000 个对象，内存大约是 13MB，使用 `namedtuple` 的效率略低于使用带 `__slots__` 的对象，但其优势在于与元组类兼容。因此，它可以传递给许多期望可迭代对象作为参数的原生 Python 函数和库。`namedtuple` 类工厂还享受元组的各种优化：例如，包含少于 `PyTuple_MAXSAVESIZE`（默认值为 20）项的元组将在 CPython 中使用更快的内存分配器。

`namedtuple`类还提供了一些额外的方法，即使它们以下划线开头，实际上也打算公开使用。`_asdict()`方法可以将`namedtuple`转换为`dict`实例，`_make()`方法允许将现有的可迭代对象转换为此类，`_replace()`则返回一个新实例，并替换掉其中的一些字段。

命名元组是替代只包含少量属性且不需要自定义方法的小对象的绝佳选择—例如，可以考虑使用它们来替代字典。如果你的数据类型需要方法、具有固定的属性列表，并且可能会实例化成千上万次，那么使用`__slots__`创建一个自定义类可能是节省内存的好方法。

### **记忆化**

*记忆化*是一种优化技术，通过缓存函数的结果来加速函数调用。只有当函数是*纯净的*（即没有副作用且不依赖任何全局状态）时，才可以缓存其结果。（有关纯函数的更多内容，请参见第八章。）

一个可以记忆化的简单函数是`sin()`，如列表 10-17 所示。

```py
>>> import math
>>> _SIN_MEMOIZED_VALUES = {}
>>> def memoized_sin(x):
...    if x not in _SIN_MEMOIZED_VALUES:
...        _SIN_MEMOIZED_VALUES[x] = math.sin(x)
...    return _SIN_MEMOIZED_VALUES[x]
>>> memoized_sin(1)
0.8414709848078965
>>> _SIN_MEMOIZED_VALUES
{1: 0.8414709848078965}
>>> memoized_sin(2)
0.9092974268256817
>>> memoized_sin(2)
0.9092974268256817
>>> _SIN_MEMOIZED_VALUES
{1: 0.8414709848078965, 2: 0.9092974268256817}
>>> memoized_sin(1)
0.8414709848078965
>>> _SIN_MEMOIZED_VALUES
{1: 0.8414709848078965, 2: 0.9092974268256817}
```

*列表 10-17：一个记忆化的 sin()函数*

在列表 10-17 中，当第一次调用`memoized_sin()`时，若其参数不在`_SIN_MEMOIZED_VALUES`字典中，该值将被计算并存储在该字典中。如果再次调用该函数并传入相同的值，结果将从字典中获取，而不是重新计算。虽然`sin()`的计算非常快速，但一些涉及复杂计算的高级函数可能会花费更长时间，这正是记忆化真正发挥作用的地方。

如果你已经阅读过装饰器的内容（如果没有，请参见《装饰器及其使用时机》，位于第 100 页），你可能会看到一个完美的机会在这里使用它们，而你是对的。PyPI 列出了几种通过装饰器实现记忆化的实现，从非常简单的案例到最复杂、最完整的实现。

从 Python 3.3 开始，`functools`模块提供了一个*最近最少使用（LRU）*缓存装饰器。它提供了与记忆化相同的功能，但其优势在于它限制了缓存中条目的数量，当缓存达到最大大小时，会删除最久未使用的条目。该模块还提供了缓存命中与未命中的统计数据（即某个数据是否存在于缓存中），以及其他一些数据。在我看来，这些统计数据在实现这种缓存时是必不可少的。使用记忆化或任何缓存技术的优势在于能够度量其使用情况和有效性。

列表 10-18 展示了如何使用`functools.lru_cache()`方法实现函数的记忆化。当函数被装饰后，它将获得一个`cache_info()`方法，可以调用该方法来获取缓存使用情况的统计信息。

```py
>>> import functools
>>> import math
>>> @functools.lru_cache(maxsize=2)
... def memoized_sin(x):
...     return math.sin(x)
...
>>> memoized_sin(2)
0.9092974268256817
>>> memoized_sin.cache_info()
CacheInfo(hits=0, misses=1, maxsize=2, currsize=1)
>>> memoized_sin(2)
0.9092974268256817
>>> memoized_sin.cache_info()
CacheInfo(hits=1, misses=1, maxsize=2, currsize=1)
>>> memoized_sin(3)
0.1411200080598672
>>> memoized_sin.cache_info()
CacheInfo(hits=1, misses=2, maxsize=2, currsize=2)
>>> memoized_sin(4)
-0.7568024953079282
>>> memoized_sin.cache_info()
CacheInfo(hits=1, misses=3, maxsize=2, currsize=2)
>>> memoized_sin(3)
0.1411200080598672
>>> memoized_sin.cache_info()
CacheInfo(hits=2, misses=3, maxsize=2, currsize=2)
>>> memoized_sin.cache_clear()
>>> memoized_sin.cache_info()
CacheInfo(hits=0, misses=0, maxsize=2, currsize=0)
```

*清单 10-18：检查缓存统计信息*

清单 10-18 展示了缓存是如何使用的，以及如何判断是否有优化空间。例如，如果在缓存未满时未命中的次数较高，则缓存可能是无效的，因为传递给函数的参数从未相同。这将有助于确定应该或不应该进行记忆化！

### **使用 PyPy 加速 Python**

*PyPy* 是一种符合标准的高效 Python 实现：你应该能够用它运行任何 Python 程序。事实上，Python 的标准实现 CPython——因为它是用 C 写的——可能非常慢。PyPy 的理念是用 Python 本身编写一个 Python 解释器。随着时间的推移，它发展成用 RPython 编写，RPython 是 Python 语言的一个受限子集。

RPython 对 Python 语言施加了约束，使得变量的类型可以在编译时推断出来。RPython 代码被转换成 C 代码，然后编译生成解释器。当然，RPython 也可以用来实现其他语言，而不仅仅是 Python。

在 PyPy 中，除了技术挑战外，另一个有趣的地方是它现在已进入一个阶段，可以作为 CPython 的更快替代品。PyPy 内置了 *即时编译（JIT）* 编译器；换句话说，它通过将编译代码的速度与解释的灵活性相结合，使代码运行得更快。

有多快？这取决于情况，但对于纯算法代码，它要快得多。对于更一般的代码，PyPy 声称大多数情况下能够达到 CPython 的三倍速度。不幸的是，PyPy 也有 CPython 的一些限制，包括 *全局解释器锁（GIL）*，它只允许一个线程在同一时间执行。

虽然严格来说这不是一种优化技术，但将 PyPy 作为你支持的 Python 实现之一可能是个好主意。为了使 PyPy 成为一个支持实现，你需要确保像在 CPython 下测试一样在 PyPy 下测试你的软件。在第六章中，我们讨论了 `tox`（请参阅“在 第 92 页 中使用 `virtualenv` 与 `tox`”），它支持使用 PyPy 构建虚拟环境，就像它支持任何版本的 CPython 一样，因此将 PyPy 支持加入到项目中应该是相当直接的。

在项目初期测试 PyPy 支持，将确保如果你决定希望能够使用 PyPy 运行你的软件，后期不会需要做太多工作。

**注意**

*在 第九章中讨论的 Hy 项目，我们从一开始就成功采用了这一策略。Hy 一直支持 PyPy 和所有其他 CPython 版本，且几乎没有遇到什么麻烦。另一方面，OpenStack 未能为其项目做到这一点，因此现在受到各种代码路径和依赖的阻碍，这些依赖由于各种原因无法在 PyPy 上运行；它们在早期阶段并不要求进行完全测试。*

PyPy 兼容 Python 2.7 和 Python 3.5，并且其 JIT 编译器支持 32 位和 64 位、x86 和 ARM 架构，以及多种操作系统（Linux、Windows 和 Mac OS X）。PyPy 在功能上通常落后于 CPython，但它会定期赶上。除非你的项目依赖于最新的 CPython 功能，否则这个滞后可能不会成为问题。

### **通过缓冲区协议实现零拷贝**

通常程序需要处理大量的数据，这些数据以大数组字节的形式存在。一旦你开始通过复制、切片和修改数据来操作它，处理如此大量的输入数据在字符串中的效率会非常低。

让我们考虑一个小程序，它读取一个大型二进制数据文件，并将其部分复制到另一个文件中。为了检查该程序的内存使用情况，我们将像之前一样使用 `memory_profiler`。部分复制文件的脚本见 列表 10-19。

```py
@profile
def read_random():
    with open("/dev/urandom", "rb") as source:
        content = source.read(1024 * 10000)
        content_to_write = content[1024:]
    print("Content length: %d, content to write length %d" %
          (len(content), len(content_to_write)))
    with open("/dev/null", "wb") as target:
        target.write(content_to_write)

if __name__ == '__main__':
    read_random()
```

*列表 10-19：部分复制文件*

使用 `memory_profiler` 运行 列表 10-19 中的程序，输出结果如下 列表 10-20 所示。

```py
$ python -m memory_profiler memoryview/copy.py
Content length: 10240000, content to write length 10238976
Filename: memoryview/copy.py

Mem usage    Increment   Line Contents
                         @profile
 9.883 MB     0.000 MB   def read_random():
 9.887 MB     0.004 MB       with open("/dev/urandom", "rb") as source:
19.656 MB     9.770 MB           content = source.read(1024 * 10000)➊
29.422 MB     9.766 MB           content_to_write = content[1024:]➋
29.422 MB     0.000 MB       print("Content length: %d, content to write length %d" %
29.434 MB     0.012 MB             (len(content), len(content_to_write)))
29.434 MB     0.000 MB       with open("/dev/null", "wb") as target:
29.434 MB     0.000 MB           target.write(content_to_write)
```

*列表 10-20：部分文件复制的内存分析*

根据输出，程序从 *_/dev/urandom* 读取了 10MB 的数据 ➊。Python 需要分配大约 10MB 的内存来存储这些数据作为字符串。然后，它复制了整个数据块，去除了前 1KB ➋。

在 列表 10-20 中有趣的是，当构建变量 `content_to_write` 时，程序的内存使用量增加了大约 10MB。实际上，`slice` 操作符正在将整个内容（去除前 1KB）复制到一个新的字符串对象中，从而分配了 10MB 中的大块内存。

在大型字节数组上执行这种操作将是一场灾难，因为将分配和复制大量内存。如果你有写 C 代码的经验，你知道使用 `memcpy()` 函数在内存使用和性能方面都有显著的开销。

但作为一个 C 程序员，你也会知道字符串是字符数组，而且没有任何东西能阻止你只查看数组的*一部分*而不进行复制。你可以通过使用基本的指针算术来做到这一点，前提是整个字符串位于一个连续的内存区域中。

在 Python 中，通过实现*缓冲区协议*的对象也可以做到这一点。缓冲区协议在 PEP 3118 中定义，作为一个 C API，需要在不同类型上实现，才能提供该协议。例如，`string` 类实现了该协议。

当你在一个对象上实现这个协议时，你可以使用 `memoryview` 类构造函数来构建一个新的 `memoryview` 对象，引用原始对象的内存。例如，清单 10-21 展示了如何使用 `memoryview` 来访问字符串的切片，而不进行任何复制：

```py
   >>> s = b"abcdefgh"
   >>> view = memoryview(s)
   >>> view[1]
➊ 98 <1>
   >>> limited = view[1:3]
   >>> limited
   <memory at 0x7fca18b8d460>
   >>> bytes(view[1:3])
   b'bc'
```

*清单 10-21：使用 memoryview 避免复制数据*

在 ➊ 处，你可以找到字母 *b* 的 ASCII 码。在清单 10-21 中，我们利用了 `memoryview` 对象的 `slice` 操作符本身返回一个 `memoryview` 对象的特性。这意味着它*不*复制任何数据，而只是引用它的某一部分，从而节省了复制所需的内存。图 10-2 展示了清单 10-21 中的操作。

![image](img/f10-02.jpg)

*图 10-2：在 memoryview 对象上使用切片*

我们可以重写清单 10-19 中的程序，这次通过引用我们想要写入的数据，而不是分配一个新的字符串，使用一个 `memoryview` 对象。

```py
@profile
def read_random():
    with open("/dev/urandom", "rb") as source:
        content = source.read(1024 * 10000)
        content_to_write = memoryview(content)[1024:]
    print("Content length: %d, content to write length %d" %
          (len(content), len(content_to_write)))
    with open("/dev/null", "wb") as target:
        target.write(content_to_write)

if __name__ == '__main__':
    read_random()
```

*清单 10-22：使用 memoryview 部分复制文件*

清单 10-22 中的程序使用的内存是清单 10-19 中的一半。我们可以通过再次使用 `memory_profiler` 来验证这一点，方法如下：

```py
$ python -m memory_profiler memoryview/copy-memoryview.py
Content length: 10240000, content to write length 10238976
Filename: memoryview/copy-memoryview.py

Mem usage    Increment   Line Contents
                         @profile
 9.887 MB     0.000 MB   def read_random():
 9.891 MB     0.004 MB ➊     with open("/dev/urandom", "rb") as source:
19.660 MB     9.770 MB ➋        content = source.read(1024 * 10000)
19.660 MB     0.000 MB           content_to_write = memoryview(content)[1024:]
19.660 MB     0.000 MB       print("Content length: %d, content to write length %d" %
19.672 MB     0.012 MB             (len(content), len(content_to_write)))
19.672 MB     0.000 MB       with open("/dev/null", "wb") as target:
19.672 MB     0.000 MB           target.write(content_to_write)
```

这些结果表明，我们从*/dev/urandom*读取了 10,000KB 的数据，但并没有做太多处理 ➊。Python 需要分配 9.77MB 的内存来存储这段数据作为一个字符串 ➋。

我们引用了整个数据块，去掉了前面的第一个 KB，因为我们不会将该第一个 KB 写入目标文件。由于我们没有进行复制，因此没有占用更多内存！

这种技巧在处理套接字时特别有用。通过套接字发送数据时，数据可能会在多次调用中被分割发送，而不是在一次调用中全部发送：`socket.send` 方法返回网络实际能够发送的数据长度，这可能比原本想要发送的数据长度小。清单 10-23 展示了这种情况通常是如何处理的。

```py
   import socket
   s = socket.socket(...)
   s.connect(...)
➊ data = b"a" * (1024 * 100000) <1> while data:
       sent = s.send(data)
    ➋ data = data[sent:] <2>
```

*清单 10-23：通过套接字发送数据*

首先，我们构建一个 `bytes` 对象，该对象包含超过 1 亿次的字母 *a* ➊。然后，我们去掉前面的 `sent` 字节 ➋。

使用清单 10-23 中实现的机制，程序会不断复制数据，直到套接字将所有数据发送完毕。

我们可以修改清单 10-23 中的程序，使用`memoryview`来实现相同的功能，且无需复制，从而提高性能，如清单 10-24 所示。

```py
   import socket
   s = socket.socket(...)
   s.connect(...)
➊ data = b"a" * (1024 * 100000) <1>
   mv = memoryview(data)
   while mv:
       sent = s.send(mv)
    ➋ mv = mv[sent:] <2>
```

*清单 10-24：通过 memoryview 发送数据到套接字*

首先，我们构建一个包含超过 1 亿个字母*a*的`bytes`对象➊。然后，我们构建一个新的`memoryview`对象，指向剩余需要发送的数据，而不是复制这些数据➋。这个程序不会进行任何复制，因此它不会使用比最初为`data`变量所需的 100MB 更多的内存。

我们已经看到`memoryview`对象如何用于高效写入数据，同样的方法也可以用于*读取*数据。Python 中的大多数 I/O 操作都知道如何处理实现了缓冲区协议的对象：它们可以从中读取数据，也可以向其中写入数据。在这种情况下，我们不需要`memoryview`对象；我们可以直接要求 I/O 函数将数据写入我们预分配的对象，如清单 10-25 所示。

```py
>>> ba = bytearray(8)
>>> ba
bytearray(b'\x00\x00\x00\x00\x00\x00\x00\x00')
>>> with open("/dev/urandom", "rb") as source:
...     source.readinto(ba)
...
8
>>> ba
bytearray(b'`m.z\x8d\x0fp\xa1')
```

*清单 10-25：写入预分配的 bytearray*

在清单 10-25 中，通过使用已打开文件的`readinto()`方法，Python 可以直接从文件中读取数据并将其写入预分配的`bytearray`中。通过这种技术，预分配缓冲区变得容易（就像在 C 语言中为了减少对`malloc()`的调用而做的那样），并且可以根据需要填充它。使用`memoryview`，你可以将数据放置在内存区域的任何位置，如清单 10-26 所示。

```py
   >>> ba = bytearray(8)
➊ >>> ba_at_4 = memoryview(ba)[4:]
   >>> with open("/dev/urandom", "rb") as source:
➋ ...     source.readinto(ba_at_4)
   ...
   4
   >>> ba
   bytearray(b'\x00\x00\x00\x00\x0b\x19\xae\xb2')
```

*清单 10-26：写入 bytearray 的任意位置*

我们从偏移量 4 开始引用`bytearray`直到末尾➊。然后，我们将来自`/dev/urandom`的内容从偏移量 4 开始写入`bytearray`的末尾，有效地只读取了 4 个字节➋。

缓冲区协议对实现低内存开销和优异性能至关重要。由于 Python 隐藏了所有的内存分配，开发人员往往会忽略底层发生的事情，这会大大影响程序的速度！

`array`模块中的对象和`struct`模块中的函数可以正确处理缓冲区协议，因此，在目标为零拷贝时，它们可以高效执行。

### **总结**

正如我们在本章中所看到的，有很多方法可以加快 Python 代码的执行速度。选择正确的数据结构，并使用正确的方法来操作数据，在 CPU 和内存使用方面能产生巨大影响。这就是为什么理解 Python 内部发生的事情非常重要。

然而，优化永远不应该过早进行，必须先进行适当的性能分析。很容易浪费时间用更快的变体重写一些几乎不使用的代码，而忽略了关键的痛点。不要忽视全局的情况。

### **Victor Stinner 关于优化**

Victor 是一个资深的 Python 黑客、核心贡献者以及许多 Python 模块的作者。他于 2013 年撰写了 PEP 454，提出了一个新的`tracemalloc`模块，用于跟踪 Python 内部的内存块分配，他还编写了一个名为 FAT 的简单 AST 优化器。他也定期为提升 CPython 的性能做出贡献。

**优化 Python 代码的一个好的起步策略是什么？**

策略在 Python 中与其他语言相同。首先，你需要一个明确的使用案例，以便获得稳定且可复现的基准测试。没有可靠的基准测试，尝试不同的优化可能会导致浪费时间和过早的优化。无用的优化可能使代码变得更差、更难以阅读，甚至更慢。一个有效的优化，必须至少能使程序加速 5% 才值得追求。

如果代码中的某个特定部分被识别为“慢”，那么应该在该部分代码上进行基准测试。对一个短小函数进行的基准测试通常称为 *微基准测试*。微基准测试的加速应该至少达到 20%，甚至 25%，才能证明优化是值得的。

在不同的计算机、操作系统或编译器上运行基准测试可能会很有趣。例如，`realloc()`的性能可能在 Linux 和 Windows 上有所不同。

**你推荐的性能分析或优化 Python 代码的工具有哪些？**

Python 3.3 提供了 `time.perf_counter()` 函数，用于测量基准测试的经过时间。它具有最佳的分辨率。

一个测试应该运行多次；三次是最少的，五次可能就足够了。重复测试可以填充磁盘缓存和 CPU 缓存。我更倾向于保留最小的运行时间；其他开发者可能更倾向于使用几何平均值。

对于微基准测试，`timeit`模块使用简单且能快速给出结果，但使用默认参数时，结果并不可靠。测试应当手动重复多次，以获得稳定的结果。

优化可能需要花费大量时间，因此最好专注于那些消耗最多 CPU 的函数。要找到这些函数，Python 提供了 `cProfile` 和 `profile` 模块来记录每个函数的耗时。

**你有什么 Python 小技巧能提高性能吗？**

你应该尽量重用标准库——它经过了充分测试，且通常效率很高。内建的 Python 类型是用 C 实现的，性能良好。使用正确的容器可以获得最佳的性能；Python 提供了多种不同类型的容器：`dict`、`list`、`deque`、`set` 等等。

有一些优化 Python 的技巧，但应该避免使用这些技巧，因为它们在带来轻微性能提升的同时，会让代码变得不那么可读。

《Python 之禅》（PEP 20）中写道：“应该有一种——而且最好只有一种——明显的方法来做这件事。” 实际上，写 Python 代码有多种方式，且性能表现也不相同。只应相信适用于你的使用场景的基准测试。

**Python 哪些方面的性能最差，需要特别注意？**

通常来说，在开发新应用时，我倾向于不去担心性能问题。过早优化是万恶之源。当你识别出慢的函数时，可以更换算法。如果算法和容器类型选择得当，你甚至可以将一些短小的函数用 C 重写，以获得最佳性能。

在 CPython 中，一个瓶颈是全局解释器锁（Global Interpreter Lock，简称 GIL）。两个线程不能同时执行 Python 字节码。然而，这个限制仅在两个线程执行纯 Python 代码时才会成为问题。如果大部分处理时间都花费在函数调用上，并且这些函数释放了 GIL，那么 GIL 就不是瓶颈。例如，大多数 I/O 函数都会释放 GIL。

`multiprocessing` 模块可以轻松地绕过 GIL。另一个选择是编写异步代码，尽管这种方法实现起来更复杂。以网络为导向的库，如 Twisted、Tornado 和 Tulip 项目，便采用了这一技术。

**一些常见的性能问题是什么？**

当对 Python 了解不足时，可能会写出低效的代码。例如，我曾见过不需要复制时误用 `copy.deepcopy()`。

另一个性能杀手是低效的数据结构。如果容器中的项数少于 100 个，则容器类型对性能没有影响。项数多了，就必须了解每个操作（`add`、`get`、`delete`）的复杂性及其影响。
