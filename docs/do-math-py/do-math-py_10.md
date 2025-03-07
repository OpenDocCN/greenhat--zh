## 第十章：**B**

**Python 主题概览**

![image](img/common-01.jpg)

本附录的目标是双重的：一方面是快速回顾一些在章节中没有深入介绍的 Python 主题，另一方面是引入一些可以帮助你编写更好 Python 程序的主题。

### **if __name__ == '__main__'**

在本书中，我们使用了以下代码块，其中`func()`是我们在程序中定义的函数：

```py
if __name__ == '__main__':
    # Do something
    func()
```

这段代码确保只有在程序独立运行时，代码块中的语句才会被执行。

当程序运行时，特殊变量`__name__`会自动被设置为`__main__`，因此`if`条件的结果为`True`，然后调用函数`func()`。然而，当你将程序导入到另一个程序时，`__name__`的值会有所不同（见“重用代码”在第 235 页）。

这是一个简短的演示。考虑以下程序，我们将其命名为*factorial.py*：

```py
   # Find the factorial of a number
   def fact(n):
       p = 1
       for i in range(1, n+1):
           p = p*i
       return p

➊ print(__name__)

   if __name__ == '__main__':
       n = int(input('Enter an integer to find the factorial of: '))
       f = fact(n)
       print('Factorial of {0}: {1}'.format(n, f))
```

程序定义了一个函数`fact()`，它计算传入整数的阶乘。当你运行它时，它会打印`__main__`，这对应于➊处的`print`语句，因为`__name__`会自动被设置为`__main__`。然后，它会要求输入一个整数，计算其阶乘并打印出来：

```py
__main__
Enter an integer to find the factorial of: 5
Factorial of 5: 120
```

现在，假设你需要在另一个程序中计算阶乘。你决定通过导入这个函数来重用它，而不是再次编写该函数：

```py
from factorial import fact
if __name__ == '__main__':
    print('Factorial of 5: {0}'.format(fact(5)))
```

请注意，两个程序必须位于同一个目录下。当你运行此程序时，你将看到以下输出：

```py
factorial
Factorial of 5: 120
```

当你的程序被另一个程序导入时，变量`__main__`的值会被设置为那个程序的文件名，不带扩展名。在这种情况下，`__name__`的值是`factorial`而不是`__main__`。因为条件`__name__ == '__main__'`现在的结果是`False`，程序就不会再要求用户输入了。移除这个条件，看看会发生什么！

总结来说，在你的程序中使用`if __name__ == '__main__'`是个好习惯，这样你希望在程序独立运行时执行的语句在程序被导入到其他程序时就不会被执行。

### **列表推导式**

假设我们有一个整数列表，并且希望创建一个新的列表，其中包含原始列表中元素的平方。以下是我们可以使用的已知方法：

```py
   >>> x = [1, 2, 3, 4]
   >>> x_square = []
➊ >>> for n in x:
➋         x_square.append(n**2)
   >>> x_square
   [1, 4, 9, 16]
```

在这里，我们使用了在本书中多个程序中使用的代码模式。我们创建了一个空列表`x_square`，然后在计算平方时逐步将值附加到该列表。我们可以使用*列表推导式*更高效地完成此操作：

```py
➌ >>> x_square = [n**2 for n in x]
   >>> x_square
   [1, 4, 9, 16]
```

➌处的语句被称为 Python 中的*列表推导式*。它由一个表达式——这里是`n**2`——和一个`for`循环`for n in x`组成。请注意，它基本上允许我们将➊和➋处的两个语句合并成一个语句，来创建一个新的列表。

作为另一个例子，考虑我们在“绘制轨迹”一文中编写的程序，在第 51 页绘制了抛体运动中物体的轨迹。在这些程序中，我们有以下代码块来计算物体在每个时刻的*x*和*y*坐标：

```py
# Find time intervals
intervals = frange(0, t_flight, 0.001)
# List of x and y coordinates
x = []
y = []
for t in intervals:
    x.append(u*math.cos(theta)*t)
    y.append(u*math.sin(theta)*t - 0.5*g*t*t)
```

使用列表推导式，你可以将代码块重写如下：

```py
# Find time intervals
intervals = frange(0, t_flight, 0.001)
# List of x and y coordinates

x = [u*math.cos(theta)*t for t in intervals]
y = [u*math.sin(theta)*t - 0.5*g*t*t for t in intervals]
```

现在这段代码更简洁了，因为你不再需要创建空列表、编写`for`循环以及向列表中添加元素。列表推导式让你可以在一行代码中完成这些操作。

你还可以在列表推导式中添加条件，以便有选择地确定哪些列表项会在表达式中进行评估。再考虑一下第一个例子：

```py
>>> x = [1, 2, 3, 4]
>>> x_square = [n**2 for n in x if n%2 == 0]
>>> x_square
[4, 16]
```

在这个列表推导式中，我们使用`if`条件显式地告诉 Python，只在`x`的偶数列表项上计算表达式`n**2`。

### **字典数据结构**

我们在第四章首次使用了 Python 字典，当时在实现 SymPy 中的`subs()`方法时。让我们更详细地探讨一下 Python 字典。考虑一个简单的字典：

```py
>>> d = {'key1': 5, 'key2': 20}
```

这段代码创建了一个包含两个键`'key1'`和`'key2'`的字典，它们的值分别是`5`和`20`。在 Python 字典中，只有字符串、数字和元组可以作为键。这些数据类型被称为*不可变*数据类型——一旦创建，它们不能被更改——因此，列表不能作为键，因为我们可以向列表中添加和移除元素。

我们已经知道，要获取字典中`'key1'`对应的值，我们需要指定`d['key1']`。这是字典最常见的使用场景之一。一个相关的使用场景是检查字典中是否包含某个特定的键，`'x'`。我们可以这样检查：

```py
>>> d = {'key1': 5, 'key2': 20}
>>> 'x' in d
False
```

一旦我们创建了字典，就可以像向列表中添加元素一样向字典中添加新的键值对。这里是一个例子：

```py
>>> d = {'key1': 5, 'key2': 20}
>>> if 'x' in d:
        print(d['x'])
else:
        d['x'] = 1

>>> d
{'key1': 5, 'x': 1, 'key2': 20}
```

这段代码检查字典`d`中是否已经存在键`'x'`。如果存在，它将打印与之对应的值；如果不存在，它将把这个键添加到字典中，并将`1`作为对应的值。与 Python 处理集合的方式类似，Python 不能保证字典中键值对的顺序。键值对可以按任何顺序排列，与插入顺序无关。

除了将键指定为字典的索引外，我们还可以使用`get()`方法来获取与键对应的值：

```py
>>> d.get('x')
1
```

如果你向`get()`方法指定一个不存在的键，返回值将是`None`。另一方面，如果你使用索引方式来获取，指定不存在的键时会报错。

`get()`方法还允许你为不存在的键设置默认值：

```py
>>> d.get('y', 0)
0
```

字典`d`中没有键`'y'`，因此返回`0`。然而，如果存在这个键，返回的是相应的值：

```py
>>> d['y'] = 1
>>> d.get('y', 0)
1
```

`keys()`和`values()`方法分别返回一个类似列表的数据结构，其中包含字典中所有的键和值：

```py
>>> d.keys()
dict_keys(['key1', 'x', 'key2', 'y'])
>>> d.values()
dict_values([5, 1, 20, 1])
```

要遍历字典中的键值对，可以使用 `items()` 方法：

```py
>>> d.items()
dict_items([('key1', 5), ('x', 1), ('key2', 20), ('y', 1)])
```

这种方法返回的是一个元组的 *视图*，每个元组都是一个键值对。我们可以使用以下代码片段来优雅地打印它们：

```py
>>> for k, v in d.items():
        print(k, v)

key1 5
x 1
key2 20
y 1
```

视图比列表更节省内存，而且它们不允许你添加或删除项。

### **多个返回值**

在我们到目前为止编写的程序中，大多数函数返回单一的值，但有时函数也会返回多个值。在 “测量离散度” 中，我们在 第 71 页 的范围计算程序中就看到过一个返回三个值的例子。以下是我们在那个程序中采取的方法：

```py
import math
def components(u, theta):
    x = u*math.cos(theta)
    y = u*math.sin(theta)
    return x, y
```

`components()` 函数接受一个速度 `u` 和一个角度 `theta`（以弧度为单位）作为参数，计算 `x` 和 `y` 组件并返回它们。为了返回计算得到的组件，我们只需在返回语句中列出相应的 Python 标签，并用逗号分隔。这将创建并返回一个包含 `x` 和 `y` 项的元组。在调用代码中，我们接收多个值：

```py
if __name__ == '__main__':
    theta = math.radians(45)
    x, y = components(theta)
```

由于 `components()` 函数返回一个元组，我们可以通过元组的索引来获取返回的值：

```py
c = components(theta)
x = c[0]
y = c[1]
```

这样做有优势，因为我们不必知道所有返回的不同值。例如，当函数返回三个值时，你不必写 `x,y,z = myfunc1()`；当函数返回四个值时，也不必写 `a,x,y,z = myfunc1()`，依此类推。

在上述任一情况下，调用 `components()` 函数的代码必须知道返回值与速度的哪个分量对应，因为从值本身无法得知这一点。

一种更友好的做法是改为返回一个字典对象，就像我们在使用 SymPy 的 `solve()` 函数时通过 `dict=True` 关键字参数所看到的那样。下面是我们如何重写之前的 `components` 函数以返回字典：

```py
import math

def components(theta):
    x = math.cos(theta)
    y = math.sin(theta)

    return {'x': x, 'y': y}
```

在这里，我们返回一个字典，键 `'x'` 和 `'y'` 分别对应 `x` 和 `y` 组件及其相应的数值。通过这种新的函数定义，我们不需要担心返回值的顺序。我们只需使用键 `'x'` 来获取 `x` 组件，使用键 `'y'` 来获取 `y` 组件：

```py
if __name__ == '__main__':
    theta = math.radians(45)
    c = components(theta)
    y = c['y']
    x = c['x']
    print(x, y)
```

这种方法消除了使用索引来引用特定返回值的需要。以下代码重写了计算范围的程序（参见 “测量离散度” 中的 第 71 页），使结果作为字典而不是元组返回：

```py
   '''
   Find the range using a dictionary to return values
   '''
   def find_range(numbers):
       lowest = min(numbers)
       highest = max(numbers)
       # Find the range
       r = highest-lowest
       return {'lowest':lowest, 'highest':highest, 'range':r}

   if __name__ == '__main__':
       donations = [100, 60, 70, 900, 100, 200, 500, 500, 503, 600, 1000, 1200]
       result = find_range(donations)
➊     print('Lowest: {0} Highest: {1} Range: {2}'.
              format(result['lowest'], result['highest'], result['range']))
```

`find_range()` 函数现在返回一个字典，包含键 `lowest`、`highest` 和 `range`，它们的值分别对应最小值、最大值和范围。在 ➊ 处，我们简单地使用相应的键来获取相应的值。

如果我们只对一组数字的范围感兴趣，而不关心最小和最大数字，我们只需要使用 `result['range']`，而不必担心返回的其他值。

### **异常处理**

在第一章中，我们学到过尝试将字符串 `'1.1'` 转换为整数时，使用 `int()` 函数会导致 `ValueError` 异常。但通过使用 `try...except` 块，我们可以打印一个用户友好的错误信息：

```py
>>> try:
        int('1.1')
except ValueError:
        print('Failed to convert 1.1 to an integer')

Failed to convert 1.1 to an integer
```

当 `try` 块中的任何语句引发异常时，所引发的异常类型会与 `except` 语句中指定的类型进行匹配。如果匹配成功，程序将继续在 `except` 块中执行。如果异常类型不匹配，程序执行将会停止，并显示异常信息。以下是一个例子：

```py
>>> try:
        print(1/0)
except ValueError:
        print('Division unsuccessful')

Traceback (most recent call last):
  File "<pyshell#66>", line 2, in <module>
    print(1/0)
ZeroDivisionError: division by zero
```

这个代码块尝试进行 0 除法运算，导致 `ZeroDivisionError` 异常。尽管除法操作在 `try...except` 块中执行，但异常类型被错误指定，导致异常没有被正确处理。处理此异常的正确方法是将 `ZeroDivisionError` 指定为异常类型。

#### ***指定多个异常类型***

你还可以指定多个异常类型。考虑下面的 `reciprocal()` 函数，它返回传递给它的数字的倒数：

```py
def reciprocal(n):
    try:
        print(1/n)
    except (ZeroDivisionError, TypeError):
        print('You entered an invalid number')
```

我们定义了函数 `reciprocal()`，它会打印用户输入的倒数。我们知道，如果该函数以 0 为参数调用，它会引发 `ZeroDivisionError` 异常。然而，如果传入一个字符串，它会引发 `TypeError` 异常。该函数将这两种情况视为无效输入，并在 `except` 语句中将 `ZeroDivisionError` 和 `TypeError` 作为元组指定。

让我们尝试用一个有效的输入调用函数——即一个非零数字：

```py
>>> reciprocal(5)
0.2
```

接下来，我们用 0 作为参数调用函数：

```py
>>> reciprocal(0)
Enter an integer: 0
You entered an invalid number
```

参数 `0` 引发了 `ZeroDivisionError` 异常，而这个异常类型在指定给 `except` 语句的异常类型元组中，因此代码会打印错误信息。

现在，让我们输入一个字符串：

```py
>>> reciprocal('1')
```

在这种情况下，我们输入了一个无效的数字，导致引发了 `TypeError` 异常。该异常也在指定异常的元组中，因此代码会打印错误信息。如果你想提供更具体的错误信息，我们可以像下面这样指定多个 `except` 语句：

```py
def reciprocal(n):
    try:
        print(1/n)
    except TypeError:
        print('You must specify a number')
    except ZeroDivisionError:
        print('Division by 0 is invalid')

>>> reciprocal(0)
Division by 0 is invalid
>>> reciprocal('1')
You must specify a number
```

除了 `TypeError`、`ValueError` 和 `ZeroDivisionError`，还有许多其他内置异常类型。Python 3.4 的文档中列出了内置的异常类型，地址为 *[`docs.python.org/3.4/library/exceptions.html#bltin-exceptions`](https://docs.python.org/3.4/library/exceptions.html#bltin-exceptions)*。

#### ***else 块***

`else` 块用于指定在没有异常发生时要执行的语句。考虑一下我们编写的一个程序示例，用来绘制抛物体的轨迹（见 “绘制轨迹” 在 第 51 页）：

```py
   if __name__ == '__main__':
       try:
           u = float(input('Enter the initial velocity (m/s): '))
           theta = float(input('Enter the angle of projection (degrees): '))
       except ValueError:
           print('You entered an invalid input')
➊     else:
           draw_trajectory(u, theta)
           plt.show()
```

如果无法将 `u` 或 `theta` 的输入转换为浮动点数，那么程序调用 `draw_trajectory()` 和 `plt.show()` 函数就没有意义了。相反，我们将这两个语句指定在 ➊ 处的 `else` 块中。使用 `try...except...else` 结构可以让你在运行时管理不同类型的错误，并在发生错误时或没有错误时采取适当的措施：

1\. 如果发生异常且有与之对应的 `except` 语句，执行会转到对应的 `except` 块。

2\. 如果没有发生异常，执行将转到 `else` 块。

### **在 Python 中读取文件**

打开文件是从中读取数据的第一步。我们先看一个简单的例子。假设有一个文件，其中包含了一系列数字，每行一个数字：

```py
100
60
70
900
100
200
500
500
503
600
1000
1200
```

我们想编写一个函数，它读取文件并返回一个包含这些数字的列表：

```py
   def read_data(path):
       numbers = []
➊     f = open(path)
➋     for line in f:
           numbers.append(float(line))
       f.close()
       return numbers
```

首先，我们定义函数 `read_data()` 并创建一个空列表来存储所有数字。在 ➊ 处，我们使用 `open()` 函数打开通过参数 `path` 指定位置的文件。路径的示例可能是 Linux 下的 */home/username/mydata.txt*，Microsoft Windows 下的 *C:\mydata.txt*，或者 OS X 下的 */Users/Username/mydata.txt*。`open()` 函数返回一个文件对象，我们使用标签 `f` 来引用它。我们可以通过在 ➋ 处使用 `for` 循环遍历文件中的每一行。由于每一行返回的是一个字符串，我们将其转换为数字并追加到列表 `numbers` 中。循环在所有行读取完毕后停止，我们使用 `close()` 方法关闭文件。最后，我们返回 `numbers` 列表。

这与我们在第三章中从文件读取数字的方式类似，尽管因为我们在那里的方法不同，所以不需要显式关闭文件。使用我们在第三章中采用的方法，我们可以将前面的函数重写如下：

```py
   def read_data(path):
       numbers = []
➊     with open(path) as f:
           for line in f:
               numbers.append(float(line))
➋     return numbers
```

这里的关键语句在 ➊。它类似于编写 `f = open(path)`，但仅是部分相似。除了打开文件并将 `open()` 返回的文件对象赋值给 `f`，它还会创建一个新的*上下文*，包含该块中的所有语句——在此情况下，所有 `return` 语句之前的语句。当该块中的所有语句执行完毕后，文件会自动关闭。也就是说，当执行到 ➋ 语句时，文件会在不需要显式调用 `close()` 方法的情况下自动关闭。这种方法还意味着，如果在处理文件时发生任何异常，文件将在程序退出之前被关闭。这是处理文件的推荐方法。

#### ***一次性读取所有行***

我们不需要一行一行地读取文件来构建列表，而可以使用 `readlines()` 方法一次性将所有行读取到一个列表中。这会导致一个更加简洁的函数：

```py
   def read_data(path):
       with open(path) as f:
➊         lines = f.readlines()
       numbers = [float(n) for n in lines]
       return numbers
```

我们通过`readlines()`方法将文件的所有行读入一个列表，在 ➊ 处。然后，我们使用`float()`函数和列表推导式将列表中的每一项转换为浮点数。最后，我们返回列表`numbers`。

#### ***指定文件名作为输入***

`read_data()`函数将文件路径作为参数。如果你的程序允许你指定文件名作为输入，那么这个函数应该适用于任何文件，只要文件包含我们预期读取的数据。以下是一个示例：

```py
if __name__=='__main__':
    data_file = input('Enter the path of the file: ')
    data = read_data(data_file)
    print(data)
```

一旦你将这段代码添加到`read_data()`函数的末尾并运行，它将要求你输入文件的路径。然后，它会打印从文件中读取的数字：

输入文件路径 /home/amit/work/mydata.txt

[100.0,60.0,70.0,900.0,100.0,200.0,500.0,500.0,503.0,600.0,1000.0,1200.0]

#### ***处理读取文件时的错误***

在读取文件时可能会出错的情况有几个：(1) 文件无法读取，或者 (2) 文件中的数据格式不符合预期。以下是文件无法读取时发生的情况示例：

```py
Enter the path of the file: /home/amit/work/mydata2.txt
Traceback (most recent call last):
  File "read_file.py", line 11, in <module>
    data = read_data(data_file)
  File "read_file.py", line 4, in read_data
    with open(path) as f:
FileNotFoundError: [Errno 2] No such file or directory: '/home/amit/work/
mydata2.txt'
```

因为我输入了一个不存在的文件路径，所以当我们尝试打开文件时，会引发`FileNotFoundError`异常。我们可以通过如下方式修改`read_data()`函数，使程序显示用户友好的错误信息：

```py
def read_data(path):
    numbers = []
    try:
        with open(path) as f:
            for line in f:
                numbers.append(float(line))
    except FileNotFoundError:
        print('File not found')
    return numbers
```

现在，当你指定一个不存在的文件路径时，将会收到一条错误信息：

```py
Enter the path of the file: /home/amit/work/mydata2.txt
File not found
```

错误的第二个来源可能是文件中的数据不是你的程序预期的内容。例如，考虑一个包含以下内容的文件：

```py
10
20
3o
1/5
5.6
```

文件中的第三行无法转换为浮点数，因为它包含字母`o`而不是数字`0`，而第四行包含了`1/5`，这是一个字符串形式的分数，`float()`无法处理。

如果你将这个数据文件提供给之前的程序，它将产生如下错误：

```py
Enter the path of the file: bad_data.txt
Traceback (most recent call last):
  File "read_file.py", line 13, in <module>
    data = read_data(data_file)
  File "read_file.py", line 6, in read_data
    numbers.append(float(line))
ValueError: could not convert string to float: '3o\n'
```

文件中的第三行是`3o`，而不是数字`30`，所以当我们尝试将其转换为浮点数时，会得到`ValueError`。当文件中出现这种数据时，你可以采取两种方法。第一种是报告错误并退出程序。修改后的`read_data()`函数如下所示：

```py
   def read_data(path):
       numbers = []
       try:
           with open(path) as f:
               for line in f:
➊               try:
➋                   n = float(line)
                 except ValueError:
                     print('Bad data: {0}'.format(line))
➌                   break
➍               numbers.append(n)
       except FileNotFoundError:
           print('File not found')
       return numbers
```

我们在函数中插入了另一个`try...except`块，从 ➊ 开始，然后在 ➋ 处将该行转换为浮点数。如果程序引发`ValueError`异常，我们将打印带有错误行的错误信息，并在 ➌ 处使用`break`退出`for`循环。程序随后停止读取文件。返回的列表`numbers`包含在遇到错误数据之前成功读取的所有数据。如果没有错误，我们将在 ➍ 处将浮点数追加到`numbers`列表中。

现在，当你将文件*bad_data.txt*提供给程序时，它只会读取前两行，显示错误信息，并退出：

```py
Enter the path of the file: bad_data.txt
Bad data: 3o

[10.0, 20.0]
```

返回部分数据可能不太理想，因此我们可以将➌处的`break`语句替换为`return`，这样就不会返回任何数据。

第二种方法是忽略错误，继续处理文件的其余部分。下面是一个修改过的`read_data()`函数，它实现了这一点：

```py
   def read_data(path):
       numbers = []
       try:
           with open(path) as f:
               for line in f:
                   try:
                       n = float(line)
                   except ValueError:
                       print('Bad data: {0}'.format(line))
➊                     continue
                   numbers.append(n)
       except FileNotFoundError:
           print('File not found')
       return numbers
```

唯一的变化是，我们不再退出`for`循环，而是使用`continue`语句在➊处继续执行下一个迭代。程序的输出现在如下所示：

```py
Bad data: 3o

Bad data: 1/5

[10.0, 20.0, 5.6]
```

你读取文件的具体应用场景将决定你选择哪种方法来处理不良数据。

### **重用代码**

在本书中，我们使用的类和函数要么是 Python 标准库的一部分，要么是通过安装第三方包（如 matplotlib 和 SymPy）后提供的。现在，我们将快速展示如何将我们自己的程序导入到其他程序中。

考虑我们在“计算两个数据集之间的相关性”一节中编写的`find_corr_x_y()`函数，它出现在第 75 页。我们将创建一个单独的文件，*correlation.py*，该文件仅包含函数定义：

```py
'''
Function to calculate the linear correlation coefficient
'''

def find_corr_x_y(x,y):
    # Size of each set
    n = len(x)

    # Find the sum of the products
    prod=[]
    for xi,yi in zip(x,y):
        prod.append(xi*yi)

    sum_prod_x_y = sum(prod)
    sum_x = sum(x)
    sum_y = sum(y)
    squared_sum_x = sum_x**2
    squared_sum_y = sum_y**2

    x_square=[]
    for xi in x:
        x_square.append(xi**2)
    x_square_sum = sum(x_square)

    y_square=[]
    for yi in y:
        y_square.append(yi**2)
    y_square_sum = sum(y_square)

    numerator = n*sum_prod_x_y - sum_x*sum_y
    denominator_term1 = n*x_square_sum - squared_sum_x
    denominator_term2 = n*y_square_sum - squared_sum_y
    denominator = (denominator_term1*denominator_term2)**0.5

    correlation = numerator/denominator

    return correlation
```

没有*.py*扩展名时，Python 文件被称为模块。这通常用于定义将用于其他程序的类和函数的文件。以下程序从我们刚才定义的相关性模块中导入`find_corr_x_y()`函数：

```py
from correlation import find_corr_x_y
if __name__ == '__main__':
    high_school_math = [83, 85, 84, 96, 94, 86, 87, 97, 97, 85]
    college_admission = [85, 87, 86, 97, 96, 88, 89, 98, 98, 87]
    corr = find_corr_x_y(high_school_math, college_admission)
    print('Correlation coefficient: {0}'.format(corr))
```

这个程序找到我们在第 3-3 表格和第 80 页中考虑的高中数学成绩和大学录取分数之间的相关性。我们从相关性模块中导入`find_corr_x_y()`函数，创建代表两组成绩的列表，并使用这两个列表作为参数调用`find_corr_x_y()`函数。当你运行程序时，它将打印出相关系数。请注意，这两个文件必须位于同一目录下——这是为了简化操作。
