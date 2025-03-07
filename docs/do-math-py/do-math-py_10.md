## **B**

**Python主题概览**

![image](images/common-01.jpg)

本附录的目的是双重的：一是简要回顾一些在章节中未详细介绍的Python主题，二是介绍一些可以帮助你编写更好Python程序的主题。

### **if __name__ == '__main__'**

在整本书中，我们使用了以下代码块，其中func()是我们在程序中定义的一个函数：

if __name__ == '__main__':

# 做些事情

func()

这段代码确保只有当程序独立运行时，代码块中的语句才会被执行。

当程序运行时，特殊变量__name__会自动被设置为__main__，所以if条件评估为True，并调用函数func()。然而，当你将程序导入到另一个程序中时，__name__的值会有所不同（参见“[重用代码](app02.html#app02lev1sec07)”在[第235页](app02.html#page_235)）。

这里有一个快速的演示。考虑以下程序，我们称其为*factorial.py*：

# 计算一个数字的阶乘

def fact(n):

p = 1

for i in range(1, n+1):

p = p * i

return p

➊ print(__name__)

if __name__ == '__main__':

n = int(input('输入一个整数来计算其阶乘：'))

f = fact(n)

print('阶乘 {0} 的值为: {1}'.format(n, f))

该程序定义了一个函数fact()，用于计算传入整数的阶乘。当你运行它时，它会打印出__main__，这是因为__name__被自动设置为__main__，对应于➊处的print语句。然后，程序会提示输入一个整数，计算其阶乘并打印出来：

__main__

输入一个整数来计算其阶乘：5

5的阶乘：120

现在，假设你需要在另一个程序中计算阶乘。你可以通过导入这个函数来重用它，而不必再次编写函数：

from factorial import fact

if __name__ == '__main__':

print('阶乘 5 的值为: {0}'.format(fact(5)))

请注意，两个程序必须在同一目录下。当你运行这个程序时，你将得到以下输出：

factorial

5的阶乘：120

当你的程序被另一个程序导入时，变量__main__的值会被设置为该程序的文件名，不包括扩展名。在这种情况下，__name__的值为factorial，而不是__main__。由于__name__ == '__main__'条件现在评估为False，程序不再要求用户输入。删除该条件，看看会发生什么！

总结来说，使用if __name__ == '__main__'是一个好习惯，这样当你的程序作为独立程序运行时你希望执行的语句不会在程序被另一个程序导入时执行。

### **列表推导式**

假设我们有一个整数列表，并且想要创建一个新的列表，包含原列表元素的平方。这里有一种方法，应该对你来说已经很熟悉：

>>> x = [1, 2, 3, 4]

>>> x_square = []

➊ >>> for n in x:

➋         x_square.append(n**2)

>>> x_square

[1, 4, 9, 16]

这里，我们使用了在本书各个程序中都用过的代码模式。我们创建了一个空列表 x_square，然后在计算平方时将其逐一添加到列表中。我们可以使用 *列表推导式* 更高效地实现这一点：

➌ >>> x_square = [n**2 for n in x]

>>> x_square

[1, 4, 9, 16]

➌ 处的语句在 Python 中被称为 *列表推导式*。它由一个表达式——这里是 n**2——以及一个 for 循环组成，for n in x。请注意，它基本上让我们将 ➊ 和 ➋ 两个语句合并为一个，在一条语句中创建一个新的列表。

作为另一个例子，考虑我们在 “[绘制轨迹](ch02.html#ch02lev3sec05)” 中写的一个程序，位于 [第 51 页](ch02.html#page_51)，用于绘制抛体运动中物体的轨迹。在这些程序中，我们有以下代码块来计算物体在每个时间瞬间的 *x* 和 *y* 坐标：

# 查找时间间隔

intervals = frange(0, t_flight, 0.001)

# x 和 y 坐标的列表

x = []

y = []

for t in intervals:

x.append(u*math.cos(theta)*t)

y.append(u*math.sin(theta)*t - 0.5*g*t*t)

使用列表推导式，你可以将代码块重写如下：

# 查找时间间隔

intervals = frange(0, t_flight, 0.001)

# x 和 y 坐标的列表

x = [u*math.cos(theta)*t for t in intervals]

y = [u*math.sin(theta)*t - 0.5*g*t*t for t in intervals]

现在，代码更加简洁，因为你不需要创建空列表、编写 for 循环并向列表中添加元素。列表推导式让你可以通过单条语句完成这一切。

你还可以在列表推导式中添加条件，以选择性地评估哪些列表项在表达式中被计算。再一次考虑第一个例子：

>>> x = [1, 2, 3, 4]

>>> x_square = [n**2 for n in x if n%2 == 0]

>>> x_square

[4, 16]

在这个列表推导式中，我们使用 if 条件显式告诉 Python 仅在 x 中的偶数项上计算表达式 n**2。

### **字典数据结构**

我们在 [第 4 章](ch04.html#ch04)中首次使用了 Python 字典，使用它实现了 SymPy 中的 subs() 方法。让我们更详细地探讨 Python 字典。考虑一个简单的字典：

>>> d = {'key1': 5, 'key2': 20}

这段代码创建了一个字典，包含两个键——'key1' 和 'key2'——对应的值分别是 5 和 20。在 Python 字典中，只有字符串、数字和元组可以作为键。这些数据类型被称为 *不可变* 数据类型——一旦创建，它们就不能改变——因此列表不能作为键，因为我们可以向列表中添加和删除元素。

我们已经知道，要检索字典中对应于 'key1' 的值，我们需要指定为 d['key1']。这是字典最常见的用例之一。一个相关的用例是检查字典是否包含某个特定的键 'x'。我们可以按如下方式检查：

>>> d = {'key1': 5, 'key2': 20}

>>> 'x' in d

False

一旦我们创建了字典，就可以像向列表添加元素一样，向字典中添加一个新的键值对。下面是一个例子：

>>> d = {'key1': 5, 'key2': 20}

>>> if 'x' in d:

print(d['x'])

else:

d['x'] = 1

>>> d

{'key1': 5, 'x': 1, 'key2': 20}

这段代码检查字典 `d` 中是否已经存在键 'x'。如果存在，它会打印出对应的值；如果不存在，则会将该键添加到字典中，并将其值设为 1。与 Python 中集合的行为类似，Python 不能保证字典中键值对的顺序。键值对的顺序可能是任何顺序，而不管插入顺序如何。

除了使用键作为字典的索引外，我们还可以使用 `get()` 方法来获取对应键的值：

>>> d.get('x')

1

如果你向 `get()` 方法指定一个不存在的键，它会返回 `None`。而如果你使用索引方式获取并指定不存在的键，则会报错。

`get()` 方法还允许你为不存在的键设置默认值：

>>> d.get('y', 0)

0

字典 `d` 中没有键 'y'，因此返回的是 0。如果存在该键，则会返回对应的值：

>>> d['y'] = 1

>>> d.get('y', 0)

1

`keys()` 和 `values()` 方法分别返回字典中所有键和值的类列表数据结构：

>>> d.keys()

dict_keys(['key1', 'x', 'key2', 'y'])

>>> d.values()

dict_values([5, 1, 20, 1])

要遍历字典中的键值对，可以使用 `items()` 方法：

>>> d.items()

dict_items([('key1', 5), ('x', 1), ('key2', 20), ('y', 1)])

此方法返回一个*视图*，该视图是一个元组集合，每个元组包含一个键值对。我们可以使用以下代码将它们打印得更整齐：

>>> for k, v in d.items():

print(k, v)

key1 5

x 1

key2 20

y 1

视图比列表更节省内存，并且不允许你添加或删除项。

### **多个返回值**

在我们写的程序中，大多数函数返回单个值，但有时函数会返回多个值。我们在“[测量离散度](ch03.html#ch03lev1sec04)”一节中看到过这样的例子，在该程序中，查找范围时，我们从 `find_range()` 函数中返回了三个数字。下面是我们在其中采用的另一种方法：

import math

def components(u, theta):

x = u*math.cos(theta)

y = u*math.sin(theta)

return x, y

`components()` 函数接受一个速度 `u` 和一个角度 `theta`（单位为弧度），然后计算 x 和 y 分量并返回它们。为了返回计算出的分量，我们只需在 `return` 语句中列出对应的 Python 标签，并用逗号分隔。这将创建并返回一个包含 x 和 y 项的元组。在调用代码中，我们接收这些多个值：

if __name__ == '__main__':

theta = math.radians(45)

x, y = components(theta)

因为 components() 函数返回一个元组，我们可以通过元组索引来获取返回值：

c = components(theta)

x = c[0]

y = c[1]

这样做的好处是我们不需要知道所有返回的不同值。例如，当函数返回三个值时，你不必写 x,y,z = myfunc1()；如果返回四个值，就不必写 a,x,y,z = myfunc1()，依此类推。

在上述任一情况中，调用 components() 函数的代码必须知道哪些返回值对应于速度的哪个分量，因为从返回值本身无法得知。

一种用户友好的方法是返回一个字典对象，正如我们在使用 dict=True 关键字参数时，在 SymPy 的 solve() 函数中看到的那样。下面是如何重写前面的 components 函数以返回字典：

import math

def components(theta):

x = math.cos(theta)

y = math.sin(theta)

return {'x': x, 'y': y}

在这里，我们返回一个字典，键 'x' 和 'y' 分别表示 x 和 y 分量及其对应的数值。通过这种新的函数定义，我们不必担心返回值的顺序。我们只需要使用键 'x' 来获取 x 分量，使用键 'y' 来获取 y 分量：

if __name__ == '__main__':

theta = math.radians(45)

c = components(theta)

y = c['y']

x = c['x']

print(x, y)

这种方法消除了使用索引来引用特定返回值的需求。以下代码重写了程序以查找范围（请参阅 “[测量离散度](ch03.html#ch03lev1sec04)” 在 [第71页](ch03.html#page_71)），使得结果以字典的形式返回，而不是元组：

'''

使用字典查找范围并返回值

'''

def find_range(numbers):

lowest = min(numbers)

highest = max(numbers)

# 查找范围

r = highest - lowest

return {'lowest':lowest, 'highest':highest, 'range':r}

if __name__ == '__main__':

donations = [100, 60, 70, 900, 100, 200, 500, 500, 503, 600, 1000, 1200]

result = find_range(donations)

➊     print('最低值: {0} 最高值: {1} 范围: {2}'.

format(result['lowest'], result['highest'], result['range']))

find_range() 函数现在返回一个包含最低值、最高值和范围的字典，并将最低值、最高值和范围作为对应的值。在 ➊ 处，我们只需使用对应的键来获取对应的值。

如果我们只关心一组数字的范围，而不在乎最低值和最高值，我们只需要使用 result['range']，而不必担心其他返回的值。

### **异常处理**

在 [第1章](ch01.html#ch01) 中，我们学习了尝试使用 int() 函数将字符串 '1.1' 转换为整数时会导致 ValueError 异常。但是使用 try...except 块时，我们可以打印出友好的错误信息：

>>> try:

int('1.1')

except ValueError:

print('无法将 1.1 转换为整数')

未能将 1.1 转换为整数

当 try 块中的任何语句引发异常时，引发的异常类型将与 except 语句指定的类型匹配。如果匹配成功，则程序在 except 块中恢复执行。如果异常类型不匹配，则程序执行停止并显示异常。以下是一个例子：

>>> try:

print(1/0)

except ValueError:

print('Division unsuccessful')

Traceback (most recent call last):

File "<pyshell#66>", line 2, in <module>

print(1/0)

ZeroDivisionError: division by zero

这段代码尝试除以 0，导致 ZeroDivisionError 异常。虽然在 try...except 块中进行了除法操作，但是异常类型被错误地指定，并且异常没有被正确处理。正确处理此异常的方法是指定 ZeroDivisionError 作为异常类型。

#### ***指定多个异常类型***

您还可以指定多个异常类型。考虑函数 reciprocal()，它返回传递给它的数字的倒数：

def reciprocal(n):

try:

print(1/n)

except (ZeroDivisionError, TypeError):

print('您输入了一个无效的数字')

我们定义了函数 reciprocal()，它打印用户输入的倒数。我们知道，如果函数以 0 被调用，将会引发 ZeroDivisionError 异常。然而，如果传入一个字符串，将会引发 TypeError 异常。函数将这两种情况视为无效输入，并在 except 语句中作为一个元组指定了 ZeroDivisionError 和 TypeError。

让我们尝试使用一个有效的输入来调用该函数——即非零数：

>>> reciprocal(5)

0.2

接下来，我们以 0 作为参数调用该函数：

>>> reciprocal(0)

输入一个整数：0

您输入了一个无效的数字

参数 0 引发了 ZeroDivisionError 异常，该异常在 except 语句指定的异常类型元组中，因此代码打印出错误消息。

现在，让我们输入一个字符串：

>>> reciprocal('1')

在这种情况下，我们输入了一个无效的数字，导致 TypeError 异常。此异常也在指定的异常元组中，因此代码打印出错误消息。如果想要提供更具体的错误消息，我们可以像下面这样指定多个 except 语句：

def reciprocal(n):

try:

print(1/n)

except TypeError:

print('您必须指定一个数字')

except ZeroDivisionError:

print('除以 0 是无效的')

>>> reciprocal(0)

除以 0 是无效的

>>> reciprocal('1')

您必须指定一个数字

除了 TypeError、ValueError 和 ZeroDivisionError 外，Python 3.4 还有许多其他内置异常类型。Python 文档位于 *[https://docs.python.org/3.4/library/exceptions.html#bltin-exceptions](https://docs.python.org/3.4/library/exceptions.html#bltin-exceptions)* 列出了 Python 3.4 的内置异常。

#### ***else 块***

else 块用于指定当没有异常发生时要执行的语句。考虑我们编写的程序中的一个例子，该程序用于绘制抛物线轨迹（参见 “[绘制轨迹](ch02.html#ch02lev3sec05)” [第51页](ch02.html#page_51)）：

if __name__ == '__main__':

try:

u = float(input('请输入初始速度 (m/s): '))

theta = float(input('请输入抛射角度（度）：'))

except ValueError:

print('你输入了无效的内容')

➊     else:

draw_trajectory(u, theta)

plt.show()

如果 u 或 theta 的输入无法转换为浮动点数字，那么程序调用 draw_trajectory() 和 plt.show() 函数就没有意义。相反，我们在 ➊ 处将这两条语句放入 else 块中。使用 try...except...else 可以让你在运行时管理不同类型的错误，并在发生错误或没有错误时采取适当的行动：

1\. 如果发生异常，并且有与引发的异常类型相对应的 except 语句，则执行会转移到相应的 except 块。

2\. 如果没有异常，执行会转移到 else 块。

### **在 Python 中读取文件**

打开文件是从文件中读取数据的第一步。让我们从一个简单的例子开始。考虑一个由一组数字组成的文件，每行包含一个数字：

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

我们想编写一个函数，读取文件并返回这些数字的列表：

def read_data(path):

numbers = []

➊     f = open(path)

➋     for line in f:

numbers.append(float(line))

f.close()

return numbers

首先，我们定义函数 read_data() 并创建一个空列表来存储所有的数字。在 ➊ 处，我们使用 open() 函数打开由路径参数指定的文件。路径的示例可能是 Linux 上的 */home/username/mydata.txt*、Microsoft Windows 上的 *C:\mydata.txt*，或 OS X 上的 */Users/Username/mydata.txt*。open() 函数返回一个文件对象，我们用标签 f 来引用它。我们可以在 ➋ 处使用 for 循环逐行读取文件。由于每一行都是作为字符串返回的，我们将其转换为数字并将其附加到列表 numbers 中。循环在所有行被读取后停止执行，并通过 close() 方法关闭文件。最后，我们返回 numbers 列表。

这类似于我们在 [第3章](ch03.html#ch03) 中读取文件中的数字，尽管在那里我们没有显式地关闭文件，因为我们采用了不同的方法。使用我们在 [第3章](ch03.html#ch03) 中的方法，我们可以将前面的函数重写如下：

def read_data(path):

numbers = []

➊     with open(path) as f:

for line in f:

numbers.append(float(line))

➋     return numbers

这里的关键语句在 ➊。它类似于写 `f = open(path)`，但只有部分相同。除了打开文件并将 `open()` 返回的文件对象赋值给 f，它还设置了一个新的 *上下文*，包括该代码块中的所有语句——在这种情况下，就是所有在 `return` 语句之前的语句。当代码块中的所有语句执行完毕时，文件会自动关闭。也就是说，当执行到 ➋ 语句时，文件会被关闭，无需显式调用 `close()` 方法。这种方法也意味着，如果在操作文件时发生了异常，它仍会在程序退出前关闭文件。这是操作文件时推荐的方式。

#### ***一次性读取所有行***

我们可以使用 `readlines()` 方法一次性将所有行读取到一个列表中，而不是逐行读取，从而实现更紧凑的功能：

def read_data(path):

使用 `open(path)` 打开文件并将其赋值给 f：

➊         lines = f.readlines()

numbers = [float(n) for n in lines]

return numbers

我们使用 ➊ 的 `readlines()` 方法将文件中的所有行读取到一个列表中。然后，我们使用 `float()` 函数和列表推导将列表中的每个项目转换为浮动点数。最后，我们返回该列表 `numbers`。

#### ***将文件名指定为输入***

`read_data()` 函数将文件路径作为参数。如果你的程序允许你指定文件名作为输入，只要文件包含我们预期要读取的数据，该函数就可以适用于任何文件。以下是一个例子：

if __name__ == '__main__':

data_file = input('输入文件路径：')

data = read_data(data_file)

print(data)

一旦你将此代码添加到 `read_data()` 函数的末尾并运行，它会提示你输入文件的路径。然后，它会打印出它从文件中读取的数字：

输入文件路径：/home/amit/work/mydata.txt

[100.0,60.0,70.0,900.0,100.0,200.0,500.0,500.0,503.0,600.0,1000.0,1200.0]

#### ***处理读取文件时的错误***

读取文件时可能出现几种错误：（1）文件无法读取，或（2）文件中的数据格式不符合预期。以下是当文件无法读取时的示例：

输入文件路径：/home/amit/work/mydata2.txt

Traceback（最近一次调用最后）：

文件 "read_file.py"，第 11 行，在 <module> 中

data = read_data(data_file)

文件 "read_file.py"，第 4 行，在 `read_data` 中

使用 `open(path)` 打开文件并将其赋值给 f：

FileNotFoundError: [Errno 2] No such file or directory: '/home/amit/work/

mydata2.txt'

由于我输入了一个不存在的文件路径，当我们尝试打开文件时，抛出了 `FileNotFoundError` 异常。我们可以通过修改 `read_data()` 函数使程序显示用户友好的错误信息，如下所示：

def read_data(path):

numbers = []

try:

使用 `open(path)` 打开文件并将其赋值给 f：

for line in f:

numbers.append(float(line))

except FileNotFoundError:

print('文件未找到')

return numbers

现在，当你指定一个不存在的文件路径时，程序将显示错误信息：

输入文件路径: /home/amit/work/mydata2.txt

文件未找到

错误的第二个来源可能是文件中的数据并不是你的程序预期读取的内容。例如，考虑一个文件，其中包含如下内容：

10

20

3o

1/5

5.6

该文件中的第三行无法转换为浮点数，因为它包含字母o而不是数字0，而第四行则是1/5，以字符串形式表示的分数，float()无法处理。

如果你将这个数据文件提供给早期的程序，它将产生以下错误：

输入文件路径: bad_data.txt

Traceback (最近的调用最后):

文件 "read_file.py"，第13行，在<模块>中

data = read_data(data_file)

文件 "read_file.py"，第6行，在read_data中

numbers.append(float(line))

ValueError: 无法将字符串转换为浮点数: '3o\n'

文件中的第三行是3o，而不是数字30，因此当我们尝试将其转换为浮点数时，结果会是ValueError。当文件中存在这样的数据时，你可以采取两种方法。第一种方法是报告错误并退出程序。修改后的read_data()函数如下所示：

def read_data(path):

numbers = []

try:

with open(path) as f:

for line in f:

➊               try:

➋                   n = float(line)

except ValueError:

print('错误数据: {0}'.format(line))

➌                   break

➍               numbers.append(n)

except FileNotFoundError:

print('文件未找到')

return numbers

我们在从➊开始的函数中插入了另一个try...except块，并在➋处将行转换为浮点数。如果程序引发了ValueError异常，我们会打印包含错误行的错误信息，并通过break语句在➌处退出for循环。程序然后停止读取文件。返回的列表numbers包含了所有在遇到错误数据之前成功读取的数据。如果没有错误，我们会在➋处将浮点数添加到numbers列表中。

现在，当你将文件*bad_data.txt*提供给程序时，它将只读取前两行，显示错误信息，然后退出：

输入文件路径: bad_data.txt

错误数据: 3o

[10.0, 20.0]

返回部分数据可能并不理想，因此我们可以将➌处的break语句替换为return，这样就不会返回任何数据。

第二种方法是忽略错误并继续处理文件的其余部分。这里是一个修改后的read_data()函数：

def read_data(path):

numbers = []

try:

with open(path) as f:

for line in f:

try:

n = float(line)

except ValueError:

print('错误数据: {0}'.format(line))

➊                     continue

numbers.append(n)

except FileNotFoundError:

print('文件未找到')

return numbers

唯一的变化是，不再从for循环中跳出，而是使用continue语句在➊处继续进行下一次迭代。程序现在的输出如下：

错误数据：3o

错误数据：1/5

[10.0, 20.0, 5.6]

你读取文件的具体应用场景将决定你采用哪种方法来处理错误数据。

### **重用代码**

在本书中，我们使用了Python标准库的一些类和函数，或者通过安装第三方包（如matplotlib和SymPy）获得的类和函数。现在，我们来看一个简单的例子，演示如何将我们自己的程序导入到其他程序中。

考虑我们在“[计算两个数据集之间的相关性](ch03.html#ch03lev1sec05)”一节中写的函数find_corr_x_y()，该函数出现在[第75页](ch03.html#page_75)。我们将创建一个单独的文件，*correlation.py*，仅包含该函数的定义：

'''

用于计算线性相关系数的函数

'''

def find_corr_x_y(x,y):

# 每个集合的大小

n = len(x)

# 计算乘积和

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

如果没有*.py*扩展名，Python文件被称为模块。这通常用于定义将在其他程序中使用的类和函数的文件。以下程序从我们刚定义的correlation模块中导入find_corr_x_y()函数：

from correlation import find_corr_x_y

if __name__ == '__main__':

高中数学 = [83, 85, 84, 96, 94, 86, 87, 97, 97, 85]

大学录取 = [85, 87, 86, 97, 96, 88, 89, 98, 98, 87]

corr = find_corr_x_y(高中数学, 大学录取)

print('相关系数：{0}'.format(corr))

该程序计算我们在[表3-3](ch03.html#ch3tab3)（[第80页](ch03.html#page_80)）中考虑的学生的高中数学成绩和大学录取成绩之间的相关性。我们从correlation模块导入find_corr_x_y()函数，创建表示两组成绩的列表，并将这两个列表作为参数调用find_corr_x_y()函数。当你运行程序时，它将打印相关系数。请注意，这两个文件必须位于同一目录下——这是为了保持简单。
