***主程序创建形状***

列表 9-5 显示了主程序的源代码，该程序创建了一个形状列表

随机选择的形状对象。

**文件：Shapes/Main_ShapesExample.py**

import pygame

import sys

from pygame.locals import *

from Square import *

from Circle import *

from Triangle import *

import pygwidgets

# 设置常量

WHITE = (255, 255, 255)

WINDOW_WIDTH = 640

WINDOW_HEIGHT = 480

FRAMES_PER_SECOND = 30

N_SHAPES = 10

# 设置窗口

pygame.init()

**190** 第9章

window = pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT), 0, 32) clock = pygame.time.Clock()

shapesList = []

shapeClassesTuple = (Square, Circle, Triangle)

for i in range(0, N_SHAPES): 1

randomlyChosenClass = random.choice(shapeClassesTuple)

oShape = randomlyChosenClass (window, WINDOW_WIDTH, WINDOW_HEIGHT)

shapesList.append(oShape)

oStatusLine = pygwidgets.DisplayText(window, (4,4),

'点击形状'，字体大小=28)

# 主循环

while True:

for event in pygame.event.get():

if event.type == QUIT:

pygame.quit()

sys.exit()

if event.type == MOUSEBUTTONDOWN: 2

# 以相反的顺序检查最后绘制的形状

for oShape in reversed(shapesList): 3

if oShape.clickedInside(event.pos): 4

area = oShape.getArea() 5

area = str(area)

theType = oShape.getType()

newText = '点击了一个 ' + theType + '，其面积为' + area)

oStatusLine.setValue(newText)

break # 只处理最上层的形状

# 告诉每个形状绘制自己

window.fill(WHITE)

for oShape in shapesList:

oShape.draw()

oStatusLine.draw()

pygame.display.update()

clock.tick(FRAMES_PER_SECOND)

*列表 9-5：从三个类中创建随机形状的主程序*

就像我们在 [第4章，w](index_split_001.html#p86) 中看到的，当我们需要管理大量对象时，通常的做法是构建一个对象列表。所以，在

主循环开始时，程序首先通过随机

在圆形、正方形和三角形之间选择；创建该类的对象

类型；并将其添加到列表中。通过这种方法，我们可以接着进行迭代

遍历列表并调用每个对象中同名的方法。

在主循环中，程序检查鼠标按下事件 2

当用户点击时发生的事情。每当检测到该事件时，代码

遍历 shapesList 3 并调用 clickedInside() 4 方法

对每个形状进行处理。由于多态性，形状的类无关紧要

对象是从哪个类实例化的。再次强调，关键是每个类的实现

clickedInside() 方法对于不同的类可以是不同的。

多态性 **191**

当任何 clickedInside() 方法返回 True 5 时，我们调用该对象的 getArea() 和 getType() 方法，而不需要关心对象属于哪种类型

被点击的对象。

下面是典型运行的输出，点击了几个不同的形状后

入口形状：

点击了一个圆形，面积为5026.544

点击了一个正方形，面积为1600

点击了一个三角形，面积为1982.5

点击了一个正方形，面积为1600

点击了一个面积为 100 的正方形

点击了一个面积为 576.0 的三角形

点击了一个面积为 3019.06799 的圆形

***扩展模式***

构建具有常用方法名的类会创建一个一致的模式，

使我们能够轻松扩展程序的模式。例如，要添加

为了使程序能够包含椭圆形，我们会构建一个 Ellipse 类

它实现了 getArea()、clickedInside()、draw() 和 getType() 方法，

ods。（clickedInside() 方法的代码可能在数学上复杂）

对椭圆形来说，复杂的代码更难理解！

一旦我们编写了 Ellipse 类的代码，唯一需要修改的地方是

设置代码唯一需要做的修改是将 Ellipse 添加到形状类的元组中

供选择的内容。主循环中的代码负责检查点击事件，

获取形状的面积等等，完全不需要改变。

这个示例展示了多态性的重要两个特性：

• 多态性扩展了抽象概念的讨论

[第 8 章 t](index_split_003.html#p192) 到一组类。如果多个类的接口方法相同，客户端程序员可以忽略

所有类中这些方法的实现。

• 多态性可以使客户端编程更容易。如果客户端程序员

程序员已经熟悉了一个或多个类提供的接口

更多的类，然后调用另一个多态类的方法

应该像遵循模式一样简单。

**pygwidgets 展示了多态性**

pygwidgets 中的所有类都设计为使用多态性，它们

都实现了两个共同的方法。第一个是 handleEvent() 方法

我们首先在第 6 章中使用了 i[n Chapter 6, w](index_split_002.html#p150)，它接受一个事件对象作为参数。

每个类都必须在该方法中包含自己的代码来处理任何事件

pygame 可能生成的事件。每次通过主循环时，客户端程序

程序需要对每个类的每个实例调用 handleEvent() 方法

是通过 pygwidgets 实例化的。

**192** 第 9 章

第二个是 draw() 方法，它将图像绘制到窗口。使用 pygwidgets 的程序中的典型绘制部分可能如下所示：

inputTextA.draw()

inputTextB.draw()

displayTextA.draw()

displayTextB.draw()

restartButton.draw()

checkBoxA.draw()

checkBoxB.draw()

radioCustom1.draw()

radioCustom2.draw()

radioCustom3.draw()

checkBoxC.draw()

radioDefault1.draw()

radioDefault2.draw()

radioDefault3.draw()

statusButton.draw()

从客户端的角度来看，每行代码只是调用 draw() 方法并且

不传入任何参数。从内部的角度来看，实现代码

这些方法各自非常不同。TextButton 对象的 draw() 方法

类与 InputText 类的完全不同，例如。

此外，所有管理值的小部件都包含一个 setValue() 方法

可选地使用一个 getValue() 方法。例如，获取用户输入的文本

要在 InputText 小部件中获取值，可以调用 getValue() 获取方法。例如，单选按钮

单选按钮和复选框控件也有一个getValue()方法，用于获取它们的当前值

ues。要将新文本放入DisplayText控件中，你需要调用setValue()方法

方法，传入新文本。单选按钮和复选框控件可以

可以通过调用它们的setValue()方法来设置值。

多态使得客户端程序员能舒适地使用

类的集合。当客户端看到一种模式，比如使用名为

handleEvent() 和 draw()，这使得它们很容易预测如何使用一个

新的类属于同一集合。

截至目前，pygwidgets包没有提供水平或垂直Slider类控件

水平或垂直的Slider类控件允许用户从一系列数字中轻松选择

数字范围。如果我将这些控件加起来，它们肯定会包含以下内容：一个handleEvent()方法，用于处理所有用户交互

包含以下内容：一个handleEvent()方法，用于处理所有用户交互

发生；有一个getValue()和一个setValue()方法用于获取和设置当前值

Slider的值；以及一个draw()方法。

**运算符的多态性**

Python还通过运算符展示了多态。考虑以下示例

示例使用 + 运算符：

value1 = 4

value2 = 5

result = value1 + value2

print(result)

多态 **193**

输出结果为：

9

+ 运算符在这里显然表示“加法”，在数学意义上

因为这两个变量都包含整数值。但是现在考虑第二个

示例：

value1 = 'Joe'

value2 = 'Schmoe'

result = value1 + value2

print(result)

输出结果为：

JoeSchmoe

这一行 result = value1 + value2 和第一个示例完全相同，

示例，但它执行的是完全不同的操作。对于字符串值，+运算符会执行字符串连接操作。

+ 运算符执行的是字符串连接操作。相同的运算符被使用了，

但执行了不同的操作。

这种运算符具有多重含义的技术是通过

通常被称为*运算符重载*。对于某些类，能够重载运算符增加了非常有用的特性，并大大提高了可读性

客户端代码的集合。

***魔法方法***

Python保留了带有两个下划线的特殊方法名称，

一个名称和两个下划线是为了某个特定的目的：

__ *<someName>* __()

这些方法官方称为*特殊方法*，但通常被称为

被Python程序员称为*魔法方法*。其中许多方法

已经定义好了，比如__init__()，它在你实例化时会被调用

从类中创建对象，但所有其他类似的名称都是可以使用的

这些方法被称为“魔法”方法，因为Python

Python在后台调用这些方法，每当它检测到一个运算符或特殊情况时

函数调用，或其他某些特殊情况。它们并不打算

可以直接由客户端代码调用。

**注意**

*由于这些魔法方法的名称很难发音——例如，*__init__()读作“下划线下划线 init 下划线下划线”——Python* *程序员通常将它们称为* dunder *方法（即双下划线的缩写）。这个方法将被称为“dunder init”。*

**第194章 第9节**

继续之前的示例，我们来看这个如何与 + 操作符一起工作。内建数据类型（整数、浮点数、字符串、布尔值等

等等）实际上在 Python 中是作为类实现的。我们可以看到这

通过使用内置的isinstance()函数来测试，该函数接受一个对象并

一个类，并且如果对象是从该类实例化的，返回 True，否则返回 False

如果没有。这些行将都报告 True：

print(isinstance(123, int))

print(isinstance('some string', str))

内建数据类型的类包含一组魔法方法，

包括基本数学操作符的 ones。当 Python 检测到 + 

与整数的操作符时，它会调用名为__add__()的魔法方法

内建整数类，它执行整数加法。当 Python 看到

与字符串使用相同的操作符时，它会调用__add__()方法。

字符串类，执行字符串连接操作。

该机制是通用的，因此当 Python 遇到 + 

当处理从你类实例化的对象时，它将调用操作符

如果你的类中有__add__()方法，则会调用它。因此，当类

作为开发者，你可以编写代码，为这个操作符发明新的意义。

每个操作符映射到一个特定的魔法方法名称。虽然有

许多类型的魔法方法，让我们从那些与比较相关的方法开始—

子操作符。

***比较操作符魔法方法***

考虑我们来自列表 9-2 的 Square 类。你希望客户端软件能够

能够比较两个 Square 对象，看它们是否相等。决定权在你

来决定在比较对象时“相等”的意义。例如，你

可能会定义为两个对象颜色相同且位于相同位置

并且具有相同的大小。作为一个简单的示例，我们将定义两个 Square 对象

作为相等，如果它们只有相同的边长。这个可以通过返回布尔值来轻松实现。

比较两个对象的 self.heightAndWidth 实例变量和

你可以编写自己的equals()方法，然后客户端软件可以像这样调用它：

客户端软件可以像这样调用它：

如果 oSquare1.equals(oSquare2):

这样会正常工作。然而，客户端软件使用时会更自然—

你可以使用标准的==比较操作符：

如果 oSquare1 == oSquare2：

以这种方式编写时，Python 将 == 操作符转换为调用魔法

第一个对象的方法。在这种情况下，Python 会尝试调用一个魔法

在 Square 类中有一个名为__eq__()的方法。表 9-1 展示了魔法方法—

所有比较操作符的ods。

多态 **195**

**表 9-1：** 比较操作符符号、意义和魔法方法名称 **符号**

**意义**

**魔法方法名称**

==

等于

__eq__()

!=

不等于

__ne__()

<

小于

__lt__()

大于

__gt__()

<=

小于或等于

__le__()

>=

大于或等于

__ge__()

允许 == 比较操作符检查两个

Square 对象时，你需要在 Square 类中编写一个像这样的函数：

def __eq__(self, oOtherSquare):

如果不是 isinstance(oOtherSquare, Square)：

引发 TypeError('第二个对象不是正方形')

如果 self.heightAndWidth == oOtherSquare.heightAndWidth：

返回 True # 匹配

否则：

返回 False # 不匹配

当 Python 检测到一个 == 比较，并且第一个对象是一个正方形时，

它会调用正方形类中的此方法。由于 Python 是一种松散类型的语言，

测量（它不要求你定义变量类型），第二个参数

第二个参数可以是任何数据类型。然而，为了使比较正常工作，

正确地，第二个参数必须也是一个正方形对象。我们执行

使用 isinstance() 函数进行检查，它与程序员定义的类一样

在定义的类中，工作方式与内置类相同。如果第二个

如果对象不是一个正方形，则会引发异常。

然后我们将当前对象（self）的高度和宽度与

第二个对象（oOtherSquare）的高度和宽度。这是一个

直接访问两个对象的实例变量是完全可以接受的，因为这两个对象属于同一类型，因此它们

是可以接受的，因为这两个对象属于同一类型，因此它们

必须包含相同的实例变量。

***具有魔术方法的矩形类***

为了扩展，我们将构建一个程序来绘制多个矩形

使用矩形类绘制图形。用户可以点击任意两个

矩形，程序将报告这两个矩形是否具有相同的

面积或者第一个矩形的面积是否大于或小于第二个矩形的面积

第二个矩形。我们将使用 ==、< 和 > 运算符并期望结果

每次比较时返回布尔值 True 或 False。列表 9-6 包含

矩形类的代码，它实现了这些运算符的魔术方法

运算符。

**第196页 第9章**

**文件：MagicMethods/Rectangle/Rectangle.py**

# 矩形类

导入 pygame

导入随机模块

# 设置颜色

红色 = (255, 0, 0)

绿色 = (0, 255, 0)

蓝色 = (0, 0, 255)

类矩形（）： 

定义 __init__(self, window):

self.window = window

self.width = random.choice((20, 30, 40))

self.height = random.choice((20, 30, 40))

self.color = random.choice((RED, GREEN, BLUE))

self.x = random.randrange(0, 400)

self.y = random.randrange(0, 400)

self.rect = pygame.Rect(self.x, self.y, self.width, self.height)

self.area = self.width * self.height

定义 clickedInside(self, mousePoint)

clicked = self.rect.collidepoint(mousePoint)

返回 clicked

# 当你比较时调用的魔术方法

# 使用 == 运算符比较两个矩形对象

定义 __eq__(self, oOtherRectangle): 1

如果不是 isinstance(oOtherRectangle, Rectangle)：

引发 TypeError('第二个对象不是矩形')

如果 self.area == oOtherRectangle.area：

返回 True

否则：

返回 False

# 当你比较时调用的魔术方法

# 使用 < 运算符比较两个矩形对象

定义 __lt__(self, oOtherRectangle): 2

如果不是 isinstance(oOtherRectangle, Rectangle)：

引发 TypeError('第二个对象不是矩形')

如果 self.area < oOtherRectangle.area：

返回 True

否则：

返回 False

# 当你比较时调用的魔术方法

# 使用 > 运算符比较两个矩形对象

定义__gt__(self, oOtherRectangle): 3

如果oOtherRectangle不是一个矩形（Rectangle）实例：

引发TypeError('第二个对象不是矩形')

多态 **197**

如果self.area > oOtherRectangle.area：

返回True

否则：

返回False

定义getArea(self)方法：

返回self.area

定义draw(self)方法：

pygame.draw.rect(self.window, self.color, (self.x, self.y, self.width, self.height)) *列表 9-6：Rectangle类*

方法__eq__() 1，__lt__() 2，和__gt__() 3允许客户端代码

使用标准比较运算符比较矩形对象。要比较—

如果要比较两个矩形是否相等，您应该写：

如果oRectangle1 == oRectangle2:

当这行代码执行时，第一个对象的__eq__()方法会被调用，

并且第二个对象作为第二个参数传递。该函数

返回True或False。同样，要进行小于比较，您应该

写一行像这样的代码：

如果oRectangle1 < oRectangle2:

__lt__()方法接着检查第一个矩形的面积

比第二个矩形的面积要小。如果客户端代码使用了>

使用>操作符比较两个矩形时，将调用__gt__()方法。

***使用魔法方法的主程序***

列表 9-7展示了测试魔法方法的主程序代码。

**文件：MagicMethods/Rectangle/Main_RectangleExample.py**

导入pygame

导入sys

从pygame.locals导入所有内容

从Rectangle导入所有内容

# 设置常量

WHITE = (255, 255, 255)

WINDOW_WIDTH = 640

WINDOW_HEIGHT = 480

FRAMES_PER_SECOND = 30

N_RECTANGLES = 10

FIRST_RECTANGLE = 'first'

SECOND_RECTANGLE = 'second'

# 设置窗口

pygame.init()

**198** 第9章

window = pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT), 0, 32) clock = pygame.time.Clock()

rectanglesList = []

对于i在范围(0, N_RECTANGLES)内：

oRectangle = Rectangle(window)

rectanglesList.append(oRectangle)

whichRectangle = FIRST_RECTANGLE

# 主循环

一直循环：

对于pygame.event.get()中的每个事件：

如果事件类型是QUIT：

pygame.quit()

sys.exit()

如果事件类型是MOUSEBUTTONDOWN：

遍历矩形列表中的每个矩形：

如果oRectangle.clickedInside(event.pos):

print('点击了', whichRectangle, '矩形。')

如果whichRectangle == FIRST_RECTANGLE:

oFirstRectangle = oRectangle 1

whichRectangle = SECOND_RECTANGLE

如果whichRectangle == SECOND_RECTANGLE:

oSecondRectangle2 = oRectangle 2

# 用户选择了两个矩形，我们来比较一下

如果oFirstRectangle == oSecondRectangle: 3

print('矩形大小相同。')

如果oFirstRectangle < oSecondRectangle: 4

print('第一个矩形小于第二个矩形。')

否则：# 必须更大 5

print('第一个矩形大于第二个矩形。')

whichRectangle = FIRST_RECTANGLE

# 清除窗口并绘制所有矩形

window.fill(WHITE)

遍历矩形列表中的每个矩形：6

oRectangle.draw()

pygame.display.update()

clock.tick(FRAMES_PER_SECOND)

*列表 9-7：绘制并比较矩形对象的主程序*

程序的用户点击矩形对进行比较

大小。我们将选中的矩形存储在两个变量中 1 2\。

我们使用 == 运算符检查相等性，结果调用

这是矩形类的 __eq__() 方法。如果矩形相同

如果大小相同，我们打印出适当的信息。如果不相同，我们检查第一个矩形

如果角度小于第二个，则使用 < 运算符 4，这将导致

多态性 **199**

调用 __lt__() 方法。如果这个比较结果不为真，我们打印出第一个比第二个大的信息 5\. 我们不需要

你可以在程序中使用 > 运算符；然而，由于其他客户端代码可能

为了实现不同的大小比较，我们包括了 __gt__() 方法

完整性

最后，我们绘制出列表中的所有矩形 6\。

因为我们在矩形类中包含了魔术方法 __eq__()、__lt__() 和 __gt__()

在矩形类中，由于我们包括了这些方法，我们能够使用标准比较运算符

在极其直观和易读的方式中实现向量。

以下是点击多个不同矩形后的输出：

点击了第一个矩形。

点击了第二个矩形。

矩形大小相同。

点击了第一个矩形。

点击了第二个矩形。

第一个矩形小于第二个矩形。

点击了第一个矩形。

点击了第二个矩形。

第一个矩形大于第二个矩形。

***数学运算符魔术方法***

你可以编写额外的魔术方法来定义点击时会发生的事情

现有代码使用其他算术运算符在对象间进行操作

来自你的类。

表 9-2 显示了基本算术操作时调用的方法

运算符。

**表 9-2：** 数学运算符符号、含义和魔术方法名称

**符号**

**含义**

**魔术方法名称**

+

加法

__add__()

-

减法

__sub__()

*

乘法

__mul__()

/

除法（浮动点数）

__truediv__()

结果）

//

整数除法

__floordiv__()

%

求余

__mod__()

abs

绝对值

__abs__()

例如，要处理 + 运算符，你可以实现一个方法

在你的类中像这样：

def __add__(self, oOther):

# 你的代码在这里定义点击时会发生什么

# 尝试添加两个这些对象。

**200** 第9章

所有魔术或双下划线方法的完整列表可以在官方文档中找到，链接如下：[*https://docs.python.org/3/reference/datamodel.html*](https://docs.python.org/3/reference/datamodel.html)。

***向量示例***

在数学中，*向量*是一个有序的 x 和 y 值对，通常表示为

在图中表示为有向线段。在这一部分，我们将构建

一个使用数学运算符魔术方法来操作向量的类。

有许多数学操作可以对向量进行。

图 9-2 显示了两个向量相加的例子。

y 轴

6

5

4

=向量 4, 5

3

向量 1, 3

2

加号

1

向量 3, 2

x 轴

–6 –5

–4

–3

–2

–1

1

2

3

4

5

6

–1

–2

–3

–4

–5

–6

*图 9-2：笛卡尔坐标系中的向量加法*

将两个向量相加会得到一个新的向量，其 x 值是两者之和

两个相加向量的 x 值和其 y 值是 y 值的总和

两个相加向量的值。在图 9-2 中，我们加法了向量 (3, 2) 和

向量 (1, 3) 创建了一个向量 (4, 5)。

如果两个向量的 x 值相同，它们被认为是相等的，且

它们的 y 值相同。一个向量的大小通过计算直角三角形的斜边来得到

直角三角形的一个边长为 x，第二个边长为

y。我们可以使用毕达哥拉斯定理来计算长度并使用

使用边长比较两个向量的大小。

多态 **201**

清单 9-8 是一个 Vector 类，演示了适用于两个向量对象之间进行数学计算和比较的魔术方法。（每个

这些方法还通过调用 isinstance() 来确保

第二个对象是一个向量。检查项包含在下载的文件中，

可下载的文件中，但为了节省空间，我在这里省略了它们。)

**文件：MagicMethods/Vectors/Vector.py**

# 向量类

导入 math

类 Vector():

'''Vector 类表示两个值作为一个向量，

允许进行许多数学计算'''

def __init__(self, x, y):

self.x = x

self.y = y

1 def __add__(self, oOther): # 调用 + 运算符

返回 Vector(self.x + oOther.x, self.y + oOther.y)

def __sub__(self, oOther): # 调用 - 运算符

返回 Vector(self.x - oOther.x, self.y - oOther.y)

2 def __mul__(self, oOther): # 调用 * 运算符

# 允许乘法的特殊代码，用于向量或标量的乘法

如果是实例化(oOther, Vector): # 两个向量相乘

返回 Vector((self.x * oOther.x), (self.y * oOther.y))

elif isinstance(oOther, (int, float)): # 乘以标量

返回 Vector((self.x * oOther), (self.y * oOther))

否则：

引发 TypeError('第二个值必须是向量或标量')

def __abs__(self):

返回 math.sqrt((self.x ** 2) + (self.y ** 2))

def __eq__(self, oOther): # 调用 == 运算符

返回 (self.x == oOther.x) 和 (self.y == oOther.y)

def __ne__(self, oOther): # 调用 != 运算符

返回 not (self == oOther) # 调用 __eq__ 方法

def __lt__(self, oOther): # 调用 < 运算符

如果 abs(self) < abs(oOther): # 调用 __abs__ 方法

返回 True

否则：

返回 False

def __gt__(self, oOther): # 调用 > 运算符

如果 abs(self) > abs(oOther): # 调用 __abs__ 方法

**202** 第 9 章

返回 True

否则：

返回 False

*清单 9-8: 实现多个魔术方法的 Vector 类*

这个类实现了算术和比较运算符，作为魔术方法

方法。客户端代码将使用标准符号进行数学和比较-

比较两个向量对象。例如，向量的加法

图 9-2 可以这样处理：

oVector1 = Vector(3, 2)

oVector2 = Vector(1, 3)

oNewVector = oVector1 + oVector2 # 使用 + 运算符加法两个向量

当第三行运行时，__add__() 方法被调用来加法

两个向量对象，最终创建一个新的向量对象。这里有一个

特殊检查在 __mul__() 方法中，允许 * 运算符可以

将两个向量相乘或将一个向量与标量相乘，具体取决于

第二个值的类型。

**创建对象值的字符串表示**

调试的一种标准方法是添加print()调用，输出

变量在程序某些位置的值：

print('我的变量是', myVariable)

然而，如果你尝试使用print()帮助你调试一个

对象的结果并不特别有用。例如，在这里我们创建

创建一个Vector对象并打印它：

oVector = Vector(3, 4)

print('我的向量是', oVector)

这就是输出的内容：

<Vector对象位于0x10361b518>

这告诉我们我们已经实例化了一个来自Vector类的对象

并显示该对象的内存地址。然而，在大多数情况下，我们

我们真正想知道的是对象中实例变量的值

在那个时刻。幸运的是，我们可以使用魔法方法来实现。

有两个魔法方法在获取信息时非常有用

从一个对象中获取（作为字符串）：

• __str__() 方法用于创建对象的字符串表示

一个对象的字符串表示，便于人类阅读。如果客户端代码调用

str()内置函数并传入一个对象时，Python会调用该

如果该类中存在魔法方法 __str__()，则使用该方法。

多态 **203**

• __repr__() 方法用于创建一个明确的、可能是机器可读的对象字符串表示。如果客户端代码

调用repr()内置函数并传入一个对象时，

Python会尝试调用该类中的魔法方法__repr__()，

如果存在的话。

我将展示__str__()方法，因为它更常用于简单的

调试。当你调用print()函数时，Python会调用内置的

str()函数将每个参数转换为字符串。对于任何参数

对于没有__str__()方法的对象，这个函数会格式化一个字符串

包含对象类型、"对象位于"字样和内存地址的字符串。

地址，然后返回结果字符串。这就是我们之前看到的输出—

包含内存地址的字符串。

你可以写自己的__str__()版本，让它生成

任何你想要的字符串来帮助调试你类的代码。一般的

方法是构建一个包含任何实例变量值的字符串，

你希望查看并将该字符串返回以供打印。例如，

我们可以向来自示例9-8中的Vector类添加以下方法来获取

获取任何Vector对象的信息：

class Vector():

--- 省略所有之前的方法 ---

def __str__(self):

return '这个向量的值是 (' + str(self.x) + ', ' + str(self.y) + ')'

如果你实例化一个Vector对象，之后你可以调用print()函数并

传入一个Vector对象：

oVector = Vector(10, 7)

print(oVector)

与其仅仅打印Vector对象的内存地址，你

将得到一个格式良好的报告，显示两个实例变量的值

对象中包含的内容：

这个向量的值是 (10, 7)

列表 9-9 中的主代码创建了几个 Vector 对象，做了一些

向量数学，并打印一些向量计算的结果。

**文件：Vectors/Main_Vectors.py**

# 向量测试代码

from Vector import *

v1 = Vector(3, 4)

v2 = Vector(2, 2)

v3 = Vector(3, 4)

**204** 第9章

# 这些行打印布尔值或数值

print(v1 == v2)

print(v1 == v3)

print(v1 < v2)

print(v1 > v2)

print(abs(v1))

print(abs(v2))

print()

# 这些行打印向量（调用 __str__() 方法）

print('向量 1:', v1)

print('向量 2:', v2)

print('向量 1 + 向量 2:', v1 + v2)

print('向量 1 - 向量 2:', v1 - v2)

print('向量 1 乘 向量 2:', v1 * v2)

print('向量 2 乘以 5:', v1 * 5)

*列出 9-9：示例主代码，创建并比较 Vectors，进行数学计算并打印* *Vectors*

这将生成以下输出：

False

True

False

True

5.0

2.8284271247461903

向量 1：这个向量的值是 (3, 4)

向量 2：这个向量的值是 (2, 2)

向量 1 + 向量 2：这个向量的值是 (5, 6)

向量 1 - 向量 2：这个向量的值是 (1, 2)

向量 1 乘 向量 2：这个向量的值是 (6, 8)

向量 2 乘以 5：这个向量的值是 (15, 20)

第一次调用 print() 输出布尔值和数值，

这些是调用数学和比较运算符魔法方法后的结果。

在第二组中，我们打印两个 Vector 对象，然后计算并打印

一些新的 Vectors。内部，print() 函数首先调用 Python 的 str()

为每个要打印的项提供一个函数；这会调用 Vector 的

__str__() 魔法方法，用于创建一个包含相关信息的格式化字符串

信息。

**一个带有魔法方法的分数类**

让我们将一些魔法方法结合起来，做一个更复杂的示例

例如。列出 9-10 显示了一个 Fraction 类的代码。每个 Fraction 对象由

由分子（上部分）和分母（下部分）组成。该

类通过在实例变量中存储分数的不同部分来跟踪分数。

ables，连同分数的近似小数值。方法

多态 **205**

允许调用者获取分数的简化值，打印分数及其浮点值，比较两个分数是否相等，并

添加两个 Fraction 对象。

**文件：MagicMethods/Fraction.py**

# Fraction 类

import math

class Fraction():

def __init__(self, numerator, denominator): 1

如果 numerator 不是整数：

raise TypeError('分子', numerator, '必须是整数')

如果 denominator 不是整数：

raise TypeError('分母', denominator, '必须是整数')

self.numerator = numerator

self.denominator = denominator

# 使用 math 包来计算最大公约数

greatestCommonDivisor = math.gcd(self.numerator, self.denominator)

如果 greatestCommonDivisor > 1：

self.numerator = self.numerator // greatestCommonDivisor

self.denominator = self.denominator // greatestCommonDivisor

self.value = self.numerator / self.denominator

# 标准化分子和分母的符号

self.numerator = int(math.copysign(1.0, self.value)) * abs(self.numerator)

self.denominator = abs(self.denominator)

def getValue(self): 2

return self.value

def __str__(self): 3

'''创建分数的字符串表示'''

output = ' Fraction: ' + str(self.numerator) + '/' + \

str(self.denominator) + '\n' + \

' Value: ' + str(self.value) + '\n'

return output

def __add__(self, oOtherFraction): 4

'''将两个 Fraction 对象相加'''

if not isinstance(oOtherFraction, Fraction):

raise TypeError('第二个值尝试加法的对象不是 Fraction')

# 使用 math 包来找到最小公倍数

newDenominator = math.lcm(self.denominator, oOtherFraction.denominator)

multiplicationFactor = newDenominator // self.denominator

equivalentNumerator = self.numerator * multiplicationFactor

otherMultiplicationFactor = newDenominator // oOtherFraction.denominator

oOtherFractionEquivalentNumerator =

oOtherFraction.numerator * otherMultiplicationFactor

**206** 第 9 章

newNumerator = equivalentNumerator + oOtherFractionEquivalentNumerator oAddedFraction = Fraction(newNumerator, newDenominator)

return oAddedFraction

def __eq__(self, oOtherFraction): 5

'''测试相等性'''

if not isinstance(oOtherFraction, Fraction):

return False # 不是与分数进行比较

if (self.numerator == oOtherFraction.numerator) and \

(self.denominator == oOtherFraction.denominator):

return True

else:

return False

*Listing 9-10: 实现多个魔法方法的 Fraction 类*

当你创建一个 Fraction 对象时，你需要传入一个分子和一个

分母为 1，并且 __init__() 方法会立即计算

约简后的分数及其浮点值。任何时候，客户端代码都可以

调用 getValue() 方法来获取该值 2\。客户端代码还可以调用

print() 来打印出该对象，Python 会调用 __str__() 方法来

格式化要打印的字符串 3\.

客户端可以将两个不同的 Fraction 对象相加，使用

+ 运算符。此时，会调用 __add__() 方法 4\。那

方法使用 math.lcd()（最小公分母）来确保

结果的 Fraction 对象具有最小公分母。

最后，客户端代码可以使用 == 运算符来检查两个 Fraction 对象是否相等。

对象是否相等。当你使用此运算符时，会调用 __eq__() 方法

被调用 5 次，检查两个 Fraction 的值并返回 True 或

False.

下面是一些代码，用于实例化 Fraction 对象并测试各种

魔法方法：

# 测试代码

oFraction1 = Fraction(1, 3) # 创建一个 Fraction 对象

oFraction2 = Fraction(2, 5)

print('Fraction1\n', oFraction1) # 打印对象 ... 调用 __str__

print('Fraction2\n', oFraction2)

oSumFraction = oFraction1 + oFraction2 # 调用 __add__

print('和为\n', oSumFraction)

print('分数 1 和 2 是否相等?', (oFraction1 == oFraction2)) # 预期为 False

print()

oFraction3 = Fraction(-20, 80)

oFraction4 = Fraction(4, -16)

print('Fraction3\n', oFraction3)

print('Fraction4\n', oFraction4)

print('分数 3 和 4 是否相等?', (oFraction3 == oFraction4)) # 预期结果为 True

print()

多态 **207**

oFraction5 = Fraction(5, 2)

oFraction6 = Fraction(500, 200)

print('5/2 和 500/2 的和\n', oFraction5 + oFraction6)

运行时，这段代码会输出：

分数1

分数：1/3

值：0.3333333333333333

Fraction2

分数：2/5

值：0.4

总和是

分数：11/15

值：0.7333333333333333

分数 1 和 2 是否相等？ 错误

Fraction3

分数：-1/4

值：-0.25

Fraction4

分数：-1/4

值：-0.25

分数 3 和 4 是否相等？ 正确

5/2 和 500/2 的和

分数：5/1

值：5.0

**总结**

本章讲解了OOP的关键概念——多态。正如

简单来说，多态是指多个类可以实现方法

方法名称相同的多个版本。每个类包含特定的代码，用于执行所需的操作

对象实例化时需要做的事情。作为一个示范，

在该课程中，我展示了如何创建多个不同形状的方法

类，每个类都有一个 __init__()、getArea()、clickedInside() 和

draw() 方法。每个版本的代码针对这些方法的实现是特定的

形状的类型。

正如你所看到的，使用多态有两个关键优点。首先，

它扩展了抽象的概念，适用于一组类，使得

客户端程序员可以忽略实现细节。其次，它允许

一个以相似方式工作的类系统，使得该系统对

客户端程序员。

**208** 第9章

我还讨论了运算符中的多态性，解释了同一运算符如何针对不同类型执行不同的操作

数据的使用。我展示了如何使用Python的魔术方法来实现这一点

以及如何在自己的类中实现这些运算符的方法

自己的类。为了演示算术和比较运算符的使用，

魔术方法中，我展示了一个Vector类和一个Fraction类。我还展示了

你可以使用 __str__() 方法来帮助调试类的内容

一个对象。

多态 **209**

**10**

**继 承**

OOP的第三个原则是*继承*，它

是推导新类的一种机制

从现有类继承。与其从头开始

从头开始并可能重复代码，

继承允许程序员为

一个扩展或与现有类区分开来的新类

现有类。

让我们从一个实际示例开始，展示继承是什么

学厨艺的基本理念。你正在参加烹饪学校。你的一节课

一些涉及制作汉堡的课程演示。你将学习

了解所有不同肉类切割、研磨方式的细节

肉的烹饪方式，最好的面包，最好的生菜、西红柿和配料

其他课程——几乎涵盖了你能想到的所有方面。你还将了解

最好的煮汉堡的方式，煮多久，何时以及多频繁地

翻转汉堡，等等。

课程中的下一课是关于芝士汉堡的。讲师

*可以*从头开始，学习所有关于汉堡的内容

再次，但它们假设您已经保留了上一课的知识，因此已经了解了所有创建美味汉堡的内容。本课因此专注于不同类型的奶酪

如何使用，何时添加，使用多少，等等。

故事的重点是没有必要“重新发明轮子”；

相反，您只需要在已有知识的基础上进行扩展。

**面向对象编程中的继承**

OOP 中的继承是创建一个建立在（*扩展*）现有类基础上的类的能力。当创建大型程序时，您经常会使用类，

提供非常有用的通用功能。你有时会想构建一个

类似于已经存在的类，但做了一些稍微不同的事情

不同地。继承让你正是可以做到这一点，创建一个新的类，

包含现有类的所有方法和实例变量，但增加了

新的和不同的功能。

继承是一个极其强大的概念。当类被设置

正确使用继承可能看起来*简单*。然而，能够设计出清晰使用继承的类是一项很难掌握的技能。作为一名

实现者，继承需要大量的练习才能正确使用并

高效地。

使用继承时，我们讨论的是两个类之间的关系，

通常称为*基类*和*子类*。

**基类**

从中继承的类；它作为子类的起始点。

**子类**

正在继承的类；它增强了基类。

虽然这些是描述两个类之间关系时最常用的术语

在 Python 中定义类时，您可能也会听到它们被以其他方式提到，例如：

• *超类* 和 *子类*

• *基类* 和 *派生* *类*

• *父类* 和 *子类*

图 10-1 是显示这种关系的标准图。

基类

继承自

子类

*图 10-1：子类继承*

*来自基类。*

**212** 第10章

子类继承了基类中定义的所有方法和实例变量。

图 10-2 提供了一种不同的，可能更准确的思考方式

两个类之间的关系。

基类

子类

*图 10-2：基类是*

*并入子类中。*

作为实现者，您可以将基类视为被并入

并入子类中。也就是说，基类实际上成为了子类的一部分

更大的子类。作为子类的客户端，您将子类视为一个

单一单元，不需要知道基类的存在。

在讨论继承时，我们常说存在*“是一个”*的关系

子类和基类之间。例如，学生是一个人，某

橙子是水果，汽车是车辆，等等。子类是一个特殊化

基类的一个版本，它继承了所有的属性和行为

基类，但也提供了额外的细节和功能。

最重要的是，子类可以在一个或两个方面扩展基类，

以以下方式（稍后将解释）：

• 子类可以 *重定义* 基类中定义的方法。那

即，子类可以提供一个与基类中同名的方法

类，但功能不同。这叫做 *重写* 方法。

当客户端代码调用一个被重写的方法时，该方法会在

调用的是子类。 (然而，子类中方法的代码

仍然可以调用基类中同名的方法。）

• 子类可以添加新的方法和实例变量，这些方法和实例变量

在基类中不会出现。

一种思考子类的方式是使用短语 *通过差异编码*。

由于子类继承了基类的所有实例变量和方法，

基类中，它不需要重复所有代码；子类只需要

需要包含与基类不同的代码。代码

因此，子类只包含新的实例变量（以及它们的

初始化），重写方法，和/或基类中没有的新方法

基类。

**实现继承**

Python 中的继承语法简单而优雅。基类

不需要 *知道* 它作为基类被使用。只有子类 继承 **213**

需要表明它想从基类继承。以下是一般语法：

类 *<BaseClassName>* ():

# 基类方法

class *<SubClassName>* ( *<BaseClassName>* ):

# 子类方法

在子类的类声明中，括号内需要指定

明确基类的名称以进行继承。在这种情况下，我们希望

子类 *<SubClassName>* 继承基类 *<BaseClassName>* 。

（程序员经常使用 *子类* 作为动词，例如“我们来子类化 ClassA 来构建 ClassB。”）以下是一个真实类名的示例：

class Widget():

# Widget 的方法

class WidgetWithFrills(Widget):

# WidgetWithFrills 的方法

Widget 类将提供通用功能。WidgetWithFrills

类将包括 Widget 类中的所有内容，并定义任何附加的

方法和实例变量，它想要具备更具体功能的。

**员工和经理示例**

我将从一个非常简单的例子开始，以便使关键概念清晰，

然后进入一些更实用的示例。

***基类：员工***

Listing 10-1 定义了一个名为 Employee 的基类。

**文件：EmployeeManagerInheritance/EmployeeManagerInheritance.py**

# 员工经理继承

#

# 定义 Employee 类，我们将其用作基类

class Employee():

def __init__(self, name, title, ratePerHour=None):

self.name = name

self.title = title

如果 ratePerHour 不是 None：

ratePerHour = float(ratePerHour)

self.ratePerHour = ratePerHour

def getName(self):

return self.name

def getTitle(self):

**214** 第 10 章

return self.title

def payPerYear(self):

# 52 周 * 每周 5 天 * 每天 8 小时

pay = 52 * 5 * 8 * self.ratePerHour

返回 pay

*清单 10-1：Employee 类，它将作为基类使用*

Employee 类包含方法 __init__()、getName()、getTitle()，以及

payPerYear()。它还拥有三个实例变量：self.name、self.title 和

self.ratePerHour，在 __init__() 方法中设置。我们检索

使用 getter 方法获取姓名和头衔。这些员工按小时计薪，因此

self.payPerYear() 进行计算，以确定基于

每小时费率。这个类中的所有内容应该对你来说很熟悉；没有什么

你可以单独实例化一个 Employee 对象，它将正常工作

很好。

***子类：Manager***

对于 Manager 类，我们考虑经理与员工之间的差异

员工：经理是一个有薪员工，管理一组直接

报告。如果这个经理做得好，他们将获得 10% 的奖金

年薪。Manager 类可以扩展 Employee 类，因为经理是一个

员工，但拥有额外的能力和责任。

清单 10-2 显示了 Manager 类的代码。它只需要包含

代码与 Employee 类有所不同，因此你会看到它不

拥有 getName() 或 getTitle() 方法。任何对这些方法的调用，如果

Manager 对象将由 Employee 类中的方法处理。

**文件：EmployeeManagerInheritance/EmployeeManagerInheritance.py**

# 定义一个继承自 Employee 的 Manager 子类

1 class Manager(Employee):

def __init__(self, name, title, salary, reportsList=None):

2 self.salary = float(salary)

如果 reportsList 为 None：

reportsList = []

self.reportsList = reportsList

3 super().__init__(name, title)

4 def getReports(self):

返回 self.reportsList

5 def payPerYear(self, giveBonus=False):

pay = self.salary

如果 giveBonus:

pay = pay + (.10 * self.salary)  # 添加 10% 奖金

6 print(self.name, '因为工作出色获得奖金')

返回 pay

*清单 10-2：作为 Employee 类子类实现的 Manager 类* 继承 **215**

在类声明 1 中，你可以看到这个类继承自 Employee 类，因为 Employee 位于名称后面的括号内

Manager。

Employee 类的 __init__() 方法期望一个姓名，一个头衔和

一个可选的每小时费率。经理是一个有薪员工，并管理一个

员工数量，因此 Manager 类的 __init__() 方法期望

一个姓名，一个头衔，一个薪水，和一个员工列表。遵循

基于差异化编码，__init__() 方法从初始化开始

这正是 Employee 类的 __init__() 方法所未做的事情。因此，我们

将薪水和 reportsList 存储在同名实例变量中 2\。

接下来，我们想调用 Employee 基类的 __init__() 方法 3\。

在这里，我调用了内建函数 super()，它请求 Python 计算

找出哪个类是基类（通常称为*父类*），并调用该类的 __init__() 方法。它还调整了参数，包含了 self

作为此调用中的第一个参数。因此，你可以将这一行理解为

翻译为：

Employee.__init__(self, name, title)

事实上，以这种方式编写代码是完全可行的；使用

调用 super() 只是编写调用的更简洁方式，不需要

指定基类的名称。

其效果是新的 Manager 类的 __init__() 方法初始化

这两个实例变量（self.salary 和 self.reportsList）是不同的，

与 Employee 类中的那些不同，并且 Employee 类的 __init__()

方法初始化了 self.name 和 self.title 实例变量，这些变量

对任何创建的 Employee 或 Manager 对象都通用。对于经理来说，

拥有工资时，self.ratePerHour 设置为 None。

**注意**

*旧版本的 Python 需要你以第三种方式编写此代码，因此你可能* *在旧程序和文档中看到这个：*

super(Employee, self).__init__(name, salary)

*这也做了完全相同的事情。然而，使用更简洁的语法的新方式*

*调用 super() 更容易记住。如果你决定更改基类的名称，使用 super() 还使得出错的概率更低。*

Manager 类添加了一个 getter 方法 getReports()，它

允许客户端代码检索向经理汇报的员工列表。

payPerYear() 方法计算并返回经理的薪水。

注意到 Employee 类和 Manager 类都有一个名为

名为 payPerYear()。如果你使用 Manager 实例调用 payPerYear() 方法

如果是 Employee 的实例，Employee 类的方法会运行并根据

按小时费率计算。如果你使用

Manager 类的方法会运行并执行不同的计算。

Manager 类中的 payPerYear() 方法 *重写*了基类中的方法

与基类中同名的方法相同。在子类中重写方法可以使

**216** 第 10 章

子类用以区分基类。重写的方法必须与它重写的方法完全相同（尽管它

可能有不同的参数列表）。在重写的方法中，你可以：

• 完全替代基类中被重写的方法。我们看到

在 Manager 类的 payPerYear() 方法中看到这个。

• 做一些自己的工作并调用继承或重写的方法

基类中同名的方法。我们在 __init__()

Manager 类的方法。

重写方法的实际内容取决于具体情况。

如果客户端调用子类中不存在的方法，则

方法调用将传递给基类。例如，注意到这里

Manager 类中没有名为 getName() 的方法，但它确实存在于

Employee 基类中。如果客户端在 Manager 实例上调用 getName()，那么

调用由基类 Employee 处理。

经理类中的 payPerYear() 方法包含这段代码：

如果给奖金：

pay = pay + (.10 * self.salary) # 添加10%的奖金

6 print(self.name, '因为表现出色获得奖金')

实例变量 self.name 是在员工类中定义的，但

经理类之前没有提到过它。这表明实例

在基类中定义的变量可以在子类方法中使用

类。这里我们正在计算经理的薪资，并且其工作正常

因为 payPerYear() 访问的是定义在其自身

类（self.salary）*和*在基类中定义的实例变量（使用 self.name 6 打印）。

***测试代码***

让我们测试我们的员工和经理对象，并调用每个对象的方法。

**文件：EmployeeManagerInheritance/EmployeeManagerInheritance.py**

# 创建对象

oEmployee1 = 员工('Joe Schmoe', '披萨师傅', 16)

oEmployee2 = 员工('Chris Smith', '收银员', 14)

oManager = 经理('Sue Jones', '披萨餐厅经理',

55000, [oEmployee1, oEmployee2])

# 调用员工对象的方法

print('员工姓名：', oEmployee1.getName())

print('员工工资：', '{:,.2f}'.format(oEmployee1.payPerYear()))

print('员工姓名：', oEmployee2.getName())

print('员工工资：', '{:,.2f}'.format(oEmployee2.payPerYear()))

print()

# 调用经理对象的方法

managerName = oManager.getName()

继承 **217**

print('经理姓名：', managerName)

# 给经理发放奖金

print('经理工资：', '{:,.2f}'.format(oManager.payPerYear(True)))

print(managerName, '(' + oManager.getTitle() + ')', '直接报告：')

reportsList = oManager.getReports()

对于 oEmployee in reportsList:

print(' ', oEmployee.getName(),

'(' + oEmployee.getTitle() + ')')

当我们运行这段代码时，看到以下输出，就像我们预期的那样

预期：

员工姓名：Joe Schmoe

员工工资：33,280.00

员工姓名：Chris Smith

员工工资：29,120.00

经理姓名：Sue Jones

Sue Jones 因为表现出色获得了奖金

经理工资：60,500.00

Sue Jones（披萨餐厅经理）直接报告：

Joe Schmoe（披萨师傅）

Chris Smith（收银员）

**客户端视角下的子类**

到目前为止的讨论重点是实现细节。

但类的表现可能会有所不同，取决于你是否是开发者

类的使用者或者在编写代码使用类时。让我们改变焦点并进行

从客户端的角度看继承。就客户端代码而言，

从某种意义上说，子类拥有基类的所有功能，外加任何

子类中定义的内容。考虑结果时，可能会有所帮助——

ing 将方法集合看作墙上的一层层涂料。当客户端查看时，

在员工类中，客户端看到的是该类中定义的所有方法

（图10-3）。

员工

_init_()

客户端

getName()

getTitle()

payPerYear()

*图10-3：客户端在查看时会看到的内容*

*员工类的接口*

**218** 第10章

当我们引入继承自 Employee 类的 Manager 类时，就像是在我们想要添加或修正的地方加上一层油漆

更改方法。对于我们不想更改的方法，我们只需保持

旧的一层油漆（图 10-4）。

Manager

Employee

_init_()

_init_()

客户端

getName()

getTitle()

payPerYear()

payPerYear()

getReports()

*图 10-4：客户端在界面中看到的内容*

*Manager 类*

作为开发者，我们知道 Manager 类继承自

Employee 类并重写了一些方法。作为客户端，我们只看到五个

方法。客户端无需了解一些方法是如何实现的

在 Manager 类中实现的，其他则来自继承的 Employee

类。

**继承的现实世界示例**

让我们来看两个继承的现实世界示例。首先，我将展示

教你如何构建一个只允许输入数字的输入框。我将

然后构建一个输出框，用于格式化货币值。

***InputNumber***

在这个第一个示例中，我们将创建一个输入框，允许用户输入

只允许输入数字数据。作为一般的用户界面设计原则，限制输入非常重要

更好地限制输入，只允许正确格式的数据，同时

在用户输入数据时进行限制，而不是允许任何输入然后再检查

它的正确性稍后再验证。输入字母或其他符号，或者

尝试输入多个小数点或多个负号时，应该

不允许其他字符输入。

pygwidgets 包含一个 InputText 类，允许用户

输入任何字符。我们将编写一个 InputNumber 类，只允许有效的

仅允许输入数字作为数据。新的 InputNumber 类将继承大部分代码

来自 InputText。我们只需要重写 InputText 的三个方法：

__init__()，handleEvent() 和 getValue()。清单 10-3 显示了 InputNumber

重写这些方法的类。

继承 **219**

**文件：MoneyExamples/InputNumber.py**

# InputNumber 类 - 只允许用户输入数字

#

# 继承的示例

import pygame

from pygame.locals import *

import pygwidgets

BLACK = (0, 0, 0)

WHITE = (255, 255, 255)

# 合法编辑键的元组

LEGAL_KEYS_TUPLE = (pygame.K_RIGHT, pygame.K_LEFT, pygame.K_HOME,

pygame.K_END, pygame.K_DELETE, pygame.K_BACKSPACE,

pygame.K_RETURN, pygame.K_KP_ENTER)

# 允许输入的合法键

LEGAL_UNICODE_CHARS = ('0123456789.-')

#

# InputNumber 继承自 InputText

#

class InputNumber(pygwidgets.InputText):

def __init__(self, window, loc, value='', fontName=None, 1

fontSize=24, width=200, textColor=BLACK,

backgroundColor=WHITE, focusColor=BLACK,

initialFocus=False, nickName=None, callback=None,

mask=None, keepFocusOnSubmit=False,

allowFloatingNumber=True, allowNegativeNumber=True):

self.allowFloatingNumber = allowFloatingNumber

self.allowNegativeNumber = allowNegativeNumber

# 调用我们基类的 __init__ 方法

super().__init__(window, loc, value, fontName, fontSize, 2

宽度，文本颜色，背景颜色，

focusColor, initialFocus, nickName, callback，

mask, keepFocusOnSubmit)

# 重写 handleEvent 方法，以便我们可以过滤合适的键

def handleEvent(self, event)：3

如果 (event.type == pygame.KEYDOWN):

# 如果不是编辑键或数字键，则忽略它

# Unicode 值仅在按键时存在

允许的键是 (event.key 在 LEGAL_KEYS_TUPLE 中) 或

(event.unicode 在 LEGAL_UNICODE_CHARS 中))

如果不允许的键：

返回 False

如果 event.unicode == '-'：# 用户输入了负号

如果不允许负数：

# 如果没有负号，则不传递它

返回 False

如果 self.cursorPosition > 0：

**220** 第10章

返回 False # 不能在第一个字符后放负号

如果 '-' 在 self.text 中：

返回 False # 不能输入第二个负号

如果 event.unicode == '.'：

如果不允许浮动数字：

# 如果没有浮动数字，不传递小数点

返回 False

如果 '.' 在 self.text 中：

返回 False # 不能输入第二个小数点

# 允许键通过到基类

结果 = super().handleEvent(event)

返回结果

def getValue(self): 4

userString = super().getValue()

尝试：

如果允许浮动数字：

returnValue = float(userString)

否则：

returnValue = int(userString)

except ValueError:

抛出 ValueError('输入的不是数字，必须至少包含一个数字。')

返回 returnValue

*列表 10-3：InputNumber 仅允许用户输入数字数据。*

__init__() 方法允许与 InputText 相同的参数

基类，再加上几个 1\. 它添加了两个布尔值：allowFloatingNumber 用于

确定是否允许用户输入浮点数

并且 allowNegativeNumber 用于确定用户是否可以输入以

带有负号。默认都为 True，因此默认情况下允许用户

输入浮动数字，并允许正负数。您

可以使用这些来限制用户仅输入例如正数

通过将两者都设为 False 来限制输入整数。__init__() 方法保存了值-

使用这两个额外的参数在实例变量中，然后调用

使用 super() 调用基类的 __init__() 方法 2\。

关键代码在 handleEvent() 方法 3 中，它限制了

允许的键限制为一个小子集：数字0到9，

负号、小数点（小数点）、回车键和一些编辑键。当

用户按下键时，会调用此方法并触发 KEYDOWN 或 KEYUP 事件

传入的代码首先确保按下的键在受限集合中。

如果用户输入的键不在该集合中（例如，任何字母），我们返回

返回 False 表示该小部件中没有发生任何重要事件，并且

那个键会被忽略。

handleEvent() 方法然后做更多的检查，以确保

输入的数字是否合法（例如，不能有两个小数点，

只有一个负号，等等）。每当检测到有效的按键时，

代码调用 InputText 基类的 handleEvent() 方法来处理

它对该键做所需的任何操作（显示或编辑字段）。

继承 **221**

当用户按下 RETURN 或 ENTER 时，客户端代码调用 get-

Value() 方法来获取用户的输入。在这个类中的 getValue() 方法

调用 InputText 类中的 getValue() 方法以获取字段中的字符串，然后

尝试将该字符串转换为数字。如果转换失败，将引发异常。

通过重写方法，我们构建了一个非常强大的新可重用类

扩展 InputText 类的功能，而不改变其

基类中的每一行都没有改变。InputText 将继续作为一个类正常工作

本身，没有任何功能上的变化。

***DisplayMoney***

作为第二个现实世界的例子，我们将创建一个字段来显示一个金额

货币。为了使这个功能更加通用，我们将使用选择的货币符号显示金额。

货币符号，将该货币符号放置在文本的左侧或右侧

（视情况而定），并通过在每三个数字之间加上逗号来格式化数字，

然后用逗号分隔每三位数字，再加上一个小数点和两位小数数字。例如，

我们希望能够将 1234.56 美元显示为 $1,234.56\。

pygwidgets 包已经有一个 DisplayText 类。我们可以实例化

使用以下接口从该类创建一个对象：

def __init__(self, window, loc=(0, 0), value='',

fontName=None, fontSize=18, width=None, height=None,

textColor=PYGWIDGETS_BLACK, backgroundColor=None,

justified='left', nickname=None):

假设我们已经有一些代码创建了一个 DisplayText 对象

命名为 oSomeDisplayText，使用适当的参数。每次我们

如果我们想更新 DisplayText 对象中的文本，必须调用其 setValue()

方法，像这样：

oSomeDisplayText.setValue('1234.56')

显示一个数字（作为字符串）与 DisplayText 的功能

对象已经存在。我们希望创建一个新的类，命名为 DisplayMoney，它是

类似于 DisplayText，但增加了功能，因此我们将继承自 DisplayText。

我们的 DisplayMoney 类将有一个增强版的 setValue()

方法，这个方法覆盖了基类的 setValue() 方法。DisplayMoney

版本将添加所需的格式化功能，包括添加货币符号，添加逗号，

添加逗号，可选地截断到两位小数等。最后，方法将调用基类 DisplayText

最终，方法将调用继承的 DisplayText 的 setValue() 方法

基类并传入一个格式化文本的字符串版本以在

窗口。

我们还将在 __init__() 中添加一些额外的设置参数

方法允许客户端代码：

• 选择货币符号（默认为 $）

• 将货币符号放在左侧或右侧（默认为左侧）

• 显示或隐藏两位小数（默认为显示）

**222** 第10章

Listing 10-4 显示了我们新创建的 DisplayMoney 类的代码。

**文件：MoneyExamples/DisplayMoney.py**

# DisplayMoney 类 - 将数字显示为货币金额

#

# 继承示例

import pygwidgets

BLACK = (0, 0, 0)

#

# DisplayMoney 类继承自 DisplayText 类

#

1 class DisplayMoney(pygwidgets.DisplayText):

2 def __init__(self, window, loc, value=None,

fontName=None, fontSize=24, width=150, height=None,

textColor=BLACK, backgroundColor=None,

justified='left', value=None, currencySymbol='$',

currencySymbolOnLeft=True, showCents=True):

3 self.currencySymbol = currencySymbol

self.currencySymbolOnLeft = currencySymbolOnLeft

self.showCents = showCents

如果value为None：

value = 0.00

# 调用我们基类的__init__方法

4 super().__init__(window, loc, value,

fontName, fontSize, width, height,

textColor, backgroundColor, justified)

5 def setValue(self, money):

如果money == '':

money = 0.00

money = float(money)

如果self.showCents：

money = '{:,.2f}'.format(money)

否则：

money = '{:,.0f}'.format(money)

如果self.currencySymbolOnLeft:

theText = self.currencySymbol + money

否则：

theText = money + self.currencySymbol

# 调用我们基类的setValue方法

6 super().setValue(theText)

*清单10-4：DisplayMoney显示一个格式化为货币值的数字。*

继承 **223**

在类定义中，我们显式继承了pygwidgets.DisplayText 1。

DisplayMoney类仅包含两个方法：__init__()和setValue()。

这两个方法重写了基类中相同名称的方法

类。

客户端像这样实例化一个DisplayMoney对象：

oDisplayMoney = DisplayMoney(widow, (100, 100), 1234.56)

通过这一行，DisplayMoney 2中的__init__()方法将运行并

覆盖基类中的__init__()方法。此方法做了一些

初始化，包括保存任何与货币相关的客户端偏好设置

symbol，显示符号的侧边，以及我们是否

应该显示分币，所有的都在实例变量中 3。该方法以

调用基类DisplayText 4的__init__()方法（它

通过调用super()来查找，并传递所需的数据

方法。

后来，客户端像这样调用以显示一个值：

oDisplayMoney.setValue(12233.44)

setValue()方法 5 在DisplayMoney类中运行以创建一个版本

格式化金额为货币值的版本。该方法

最后通过调用DisplayText类中的继承setValue()方法 6 来结束

设置新文本以进行显示。

当对DisplayMoney的实例调用任何其他方法时，

DisplayText中的版本将运行。最重要的是，每次

客户端代码应调用oDisplayMoney.draw()，这将

绘制窗口中的字段。由于DisplayMoney没有*draw()*

方法，调用将转到DisplayText基类，后者确实有一个

draw()方法。

***示例用法***

图10-5显示了一个示例程序的输出，该程序利用了

输入数字和DisplayMoney类的实例。用户通过循环输入数字

输入到InputNumber字段中。当用户按下OK或ENTER时，该值将

将在两个DisplayMoney字段中显示。第一个字段显示数字

带有小数位的，第二个方法使用不同的初始设置四舍五入到最接近的美元

不同的初始设置。

列表 10-5 包含主程序的完整代码。请注意

代码创建了一个InputNumber对象和两个DisplayMoney对象

objects.

**224** 第10章

![图片 28](index-254_1.png)

*图 10-5：一个客户端程序，用户在输入框中输入金额*，*然后金额显示在两个DisplayMoney框中*

**文件：MoneyExamples/Main_MoneyExample.py**

# 金钱示例

#

# 演示重写继承的DisplayText和InputText方法

# 1 - 导入包

import pygame

from pygame.locals import *

import sys

import pygwidgets

from DisplayMoney import *

from InputNumber import *

# 2 - 定义常量

BLACK = (0, 0, 0)

BLACKISH = (10, 10, 10)

GRAY = (128, 128, 128)

WHITE = (255, 255, 255)

BACKGROUND_COLOR = (0, 180, 180)

WINDOW_WIDTH = 640

WINDOW_HEIGHT = 480

FRAMES_PER_SECOND = 30

# 3 - 初始化世界

pygame.init()

window = pygame.display.set_mode([WINDOW_WIDTH, WINDOW_HEIGHT])

clock = pygame.time.Clock()

# 4 - 加载资源：图像、声音等

继承 **225**

# 5 - 初始化变量

title = pygwidgets.DisplayText(window, (0, 40),

'输入数字和显示金额字段的演示',

fontSize=36, width=WINDOW_WIDTH, justified='center')

inputCaption = pygwidgets.DisplayText(window, (20, 150),

'输入金额：', fontSize=24,

width=190, justified='right')

inputField = InputNumber(window, (230, 150), '', width=150)

okButton = pygwidgets.TextButton(window, (430, 150), '确定')

outputCaption1 = pygwidgets.DisplayText(window, (20, 300),

'输出美元和分：', fontSize=24,

width=190, justified='right')

moneyField1 = DisplayMoney(window, (230, 300), '', textColor=BLACK,

backgroundColor=WHITE, width=150)

outputCaption2 = pygwidgets.DisplayText(window, (20, 400),

'仅显示美元：', fontSize=24,

width=190, justified='right')

moneyField2 = DisplayMoney(window, (230, 400), '', textColor=BLACK,

backgroundColor=WHITE, width=150,

showCents=False)

# 6 - 永久循环

while True:

# 7 - 检查并处理事件

for event in pygame.event.get():

# 如果事件是点击关闭框，退出pygame和程序

if event.type == pygame.QUIT:

pygame.quit()

sys.exit()

# 按下Return/Enter或点击OK触发动作

if inputField.handleEvent(event) or okButton.handleEvent(event): 1

try:

theValue = inputField.getValue()

except ValueError: # 其他任何错误

inputField.setValue('(不是数字)')

else: # 输入有效

theText = str(theValue)

moneyField1.setValue(theText)

moneyField2.setValue(theText)

# 8 - 执行任何“每帧”动作

# 9 - 清除窗口

window.fill(BACKGROUND_COLOR)

# 10 - 绘制所有窗口元素

title.draw()

inputCaption.draw()

inputField.draw()

okButton.draw()

**226** 第10章

outputCaption1.draw()

moneyField1.draw()

outputCaption2.draw()

moneyField2.draw()

# 11 - 更新窗口

pygame.display.update()

# 12 - 稍微减慢速度

clock.tick(FRAMES_PER_SECOND) # 使pygame等待

*示例10-5：演示InputNumber和DisplayMoney类的主程序* 用户将数字输入到InputNumber字段中。用户输入时，

handleEvent()会过滤掉任何不适当的字符并忽略它们

方法。当用户点击OK时，代码读取输入并将其传递给

显示金额的两个字段。第一个显示美元和美分金额（保留两位小数），而第二个只显示美元数额。两个字段

保留两位小数），而第二个只显示美元金额。两个字段

添加一个$符号作为货币符号，并每三位数字添加一个逗号。

**多个类继承自同一个基类**

多个不同的类可以继承自同一个基类。你可以

构建一个非常通用的基类，然后构建任意数量的子类

继承自它的类。图10-6 展示了这种关系。

基类

继承自

子类

子类

子类

…

*图10-6：三个或更多不同的子类继承自一个公共基类* 每个不同的子类可以是其变种（更具体的版本） 

基本类的变种，每个子类可以重写任何方法

基类的字段，子类可以根据需要独立重写，和其他任何

子类。

让我们通过使用Shapes程序中的示例来演示

[第9章 t](index_split_003.html#p212) 该程序创建并绘制了圆形、正方形和三角形。代码还允许用户点击窗口中的任何形状，以查看其面积。

那个形状。

该程序实现了三个不同的形状类：

圆形、正方形和三角形。如果我们回顾这三个类，我们会发现

每个类都有这个完全相同的方法：

def getType(self):

return self.shapeType

继承 **227**

此外，查看三个类的__init__()方法时，我们发现有一些共同的代码，用于记住窗口、选择一个

随机颜色，并选择一个随机位置：

self.window = window

self.color = random.choice((RED, GREEN, BLUE))

self.x = random.randrange(1, maxWidth - 100)

self.y = random.randrange(1, maxHeight - 100)

最后，每个类将实例变量self.shapeType设置为一个合适的值

合适的字符串。

每当我们发现一组类实现了完全相同的方法

和/或在一个常用名称的方法中共享一些代码时，我们应该意识到

识别到这是继承的一个良好候选者。

让我们提取三个类中的共同代码，并构建一个通用的

一个名为Shape的通用基类，如示例 10-6 所示。

**文件：InheritedShapes/ShapeBasic.py**

# Shape类 - 基本

import random

# 设置颜色

RED = (255, 0, 0)

GREEN = (0, 255, 0)

BLUE = (0, 0, 255)

class Shape():

1 def __init__(self, window, shapeType, maxWidth, maxHeight):

self.window = window

self.shapeType = shapeType

self.color = random.choice((RED, GREEN, BLUE))

self.x = random.randrange(1, maxWidth - 100)

self.y = random.randrange(25, maxHeight - 100)

2 def getType(self):

return self.shapeType

*示例 10-6：Shape 类，用作基类*

该类仅由两个方法组成：__init__() 和 getType()。该

__init__() 方法 1 会记住传入实例变量的数据

ables，然后随机选择颜色和起始位置（self.x 和

self.y）。getType() 方法 2 只返回给定形状的类型

在初始化时。

我们现在可以编写任意数量的继承自 Shape 的子类。

我们将创建三个子类，它们将调用 Shape

class，传入一个字符串来标识其类型和窗口的大小。

getType() 方法仅会出现在 Shape 类中，因此任何客户端调用

**228** 第10章

对 getType() 的调用将由继承自 Shape 类的方法处理。

我们将从 Square 类的代码开始，如列表 10-7 所示。

**文件：InheritedShapes/Square.py**

# Square 类

import pygame

from Shape import *

class Square(Shape): 1

def __init__(self, window, maxWidth, maxHeight):

super().__init__(window, 'Square', maxWidth, maxHeight) 2

self.widthAndHeight = random.randrange(10, 100)

self.rect = pygame.Rect(self.x, self.y,

self.widthAndHeight, self.widthAndHeight)

def clickedInside(self, mousePoint): 3

clicked = self.rect.collidepoint(mousePoint)

return clicked

def getArea(self): 4

theArea = self.widthAndHeight * self.widthAndHeight

return theArea

def draw(self): 5

pygame.draw.rect(self.window, self.color,

(self.x, self.y, self.widthAndHeight, self.widthAndHeight))

*列表 10-7：继承自 Shape 类的 Square 类*

Square 类从 Shape 类继承开始 1。该

__init__() 方法调用其基类（或 super-）的 __init__() 方法

class) 2，标识这个形状为正方形并随机选择其大小。

接下来我们有三个方法，它们的实现是特定的

转换为正方形。clickedInside() 方法只需要调用

rect.collidepoint() 来确定点击是否发生在其矩形内部。

角度 3。getArea() 方法简单地将 widthAndHeight 乘以

widthAndHeight 4。最后，draw() 方法绘制一个矩形，使用

widthAndHeight 的值 5\。

列表 10-8 显示了 Circle 类，它也已经被修改为

继承自 Shape 类。

**文件：InheritedShapes/Circle.py**

# Circle 类

import pygame

from Shape import *

import math

继承 **229**

class Circle(Shape):

def __init__(self, window, maxWidth, maxHeight):

super().__init__(window, 'Circle', maxWidth, maxHeight)

self.radius = random.randrange(10, 50)

self.centerX = self.x + self.radius

self.centerY = self.y + self.radius

self.rect = pygame.Rect(self.x, self.y, self.radius * 2, self.radius * 2)

def clickedInside(self, mousePoint):

theDistance = math.sqrt(((mousePoint[0] - self.centerX) ** 2) +

((mousePoint[1] - self.centerY) ** 2))

如果 theDistance <= self.radius:

return True

else:

return False

def getArea(self):

theArea = math.pi * (self.radius ** 2)

return theArea

def draw(self):

pygame.draw.circle(self.window, self.color, (self.centerX, self.centerY),

self.radius, 0)

*列表 10-8：继承自 Shape 类的 Circle 类*

Circle 类也包含 clickedInside()、getArea() 和 draw()

方法，其实现特定于一个圆形。

最后，列表 10-9 显示了 Triangle 类的代码。

**文件：InheritedShapes/Triangle.py**

# Triangle 类

导入 pygame

从 Shape 导入 *

class Triangle(Shape):

定义 __init__(self, window, maxWidth, maxHeight)：

super().__init__(window, 'Triangle', maxWidth, maxHeight)

self.width = random.randrange(10, 100)

self.height = random.randrange(10, 100)

self.triangleSlope = -1 * (self.height / self.width)

self.rect = pygame.Rect(self.x, self.y, self.width, self.height)

定义 clickedInside(self, mousePoint)：

inRect = self.rect.collidepoint(mousePoint)

如果不在矩形内：

返回 False

# 做一些数学运算，看看点是否在三角形内

xOffset = mousePoint[0] - self.x

**230** 第 10 章

yOffset = mousePoint[1] – self.y

如果 xOffset == 0：

返回 True

pointSlopeFromYIntercept = (yOffset – self.height) / xOffset # 上升除以运行

如果 pointSlopeFromYIntercept < 1：

返回 True

否则：

返回 False

定义 getArea(self):

theArea = .5 * self.width * self.height

返回 theArea

定义 draw(self):

pygame.draw.polygon(self.window, self.color, (

（self.x, self.y + self.height），

（self.x, self.y），

（self.x + self.width, self.y)））

*列表 10-9：继承自 Shape 类的 Triangle 类*

我们用于测试的主要代码（[第9章](index_split_003.html#p212)）完全不需要更改。作为这些新类的客户端，它实例化了 Square、Circle 和 Triangle 对象，而不必担心这些类的实现。它

不需要知道每个类都是从一个公共的 Shape 类继承的。

**抽象类和方法**

不幸的是，我们的 Shape 基类有一个潜在的 bug。目前，一个

客户端可以实例化一个通用的 Shape 对象，但它太通用，无法拥有

它自己的 getArea() 方法。进一步地，所有继承自 Shape 的类

类（如 Square、Circle 和 Triangle）*必须*实现 clickedInside()，

getArea() 和 draw()。为了解决这两个问题，我将引入

*抽象类* 和 *抽象方法* 的概念。

**抽象类**

一个*不*打算直接实例化的类，而只是作为一个或多个子类的基类使用。（在其他一些语言中，抽象类是

被称为*虚拟类*。)

**抽象方法** 一个*必须*在每个子类中重写的方法。

通常，一个基类不能正确实现抽象方法

因为它无法知道它应该操作的详细数据，或者它可能不会

可能无法实现一个通用的算法。相反，所有子类都需要

去实现它们自己的抽象方法版本。

在我们的图形示例中，我们希望 Shape 类是一个抽象类，

所以没有客户端代码可以实例化一个 Shape 对象。进一步地，我们的 Shape 类

继承 **231**

应该指示所有其子类需要实现 clickedInside()、getArea() 和 draw() 方法。

Python 没有关键字来标识类或方法为

抽象的。然而，Python 标准库包含 abc 模块

模块，简称 *abstract base class*，旨在帮助开发者构建抽象基类和方法。

让我们看看为了构建一个抽象类我们需要做什么

抽象方法。首先，我们需要从 abc 模块中导入两件事

模块：

from abc import ABC, abstractmethod

接下来，我们需要指明我们希望作为抽象类使用的类

基类应该继承自 ABC 类，我们通过在类中添加 ABC 来实现

在类名后面的括号中：

类 *<classWeWantToDesignateAsAbstract>* (ABC)：

然后我们必须在任何方法之前使用特殊的装饰器 @abstractmethod

方法必须由所有子类重写的标识符：

@abstractmethod

def *<someMethodThatMustBeOverwritten>* (self, ...):

Listing 10-10 显示了我们如何将 Shape 类标记为抽象基类

类并指明其抽象方法。

**文件：InheritedShapes/Shape.py**

# Shape 类

#

# 用作其他类的基类

import random

from abc import ABC, abstractmethod

# 设置颜色

RED = (255, 0, 0)

GREEN = (0, 255, 0)

BLUE = (0, 0, 255)

1 class Shape(ABC): # 将此类标识为抽象基类

2 def __init__(self, window, shapeType, maxWidth, maxHeight):

self.window = window

self.shapeType = shapeType

self.color = random.choice((RED, GREEN, BLUE))

self.x = random.randrange(1, maxWidth - 100)

self.y = random.randrange(25, maxHeight - 100)

3 def getType(self):

return self.shapeType

**232** 第 10 章

4 @abstractmethod

def clickedInside(self, mousePoint):

引发 NotImplementedError

5 @abstractmethod

def getArea(self):

引发 NotImplementedError

6 @abstractmethod

def draw(self):

引发 NotImplementedError

*Listing 10-10: 继承自 ABC 并具有抽象方法的 Shape 基类* Shape 类继承自 ABC 类 1，告诉 Python 防止

防止客户端代码直接实例化 Shape 对象。任何尝试这样做的操作

会导致以下错误信息：

TypeError: 无法实例化包含抽象方法的抽象类 Shape

clickedInside, draw, getArea

__init__() 2 和 getType() 3 方法包含的代码将在

所有 Shape 子类共享的部分。

clickedInside() 4、getArea() 5 和 draw() 6 方法都是预设

由 @abstractmethod 装饰器标记。该装饰器表示这些

方法 *必须* 被所有 Shape 子类重写。由于这些方法在这个抽象类中永远不会执行，因此此处的实现仅由

使用引发 NotImplementedError 来进一步强调方法不

做任何事情。

让我们扩展形状演示程序，添加一个新的矩形

类，如 Listing 10-11 所示。矩形类继承自该

抽象 Shape 类，因此必须实现 clickedInside()、

getArea() 和 draw() 方法。我将在这个子类中故意犯一个错误

展示发生的情况。

**文件：InheritedShapes/Rectangle.py**

# 矩形类

import pygame

from Shape import *

class Rectangle(Shape):

def __init__(self, window, maxWidth, maxHeight):

super().__init__(window, 'Rectangle', maxWidth, maxHeight)

self.width = random.randrange(10, 100)

self.height = random.randrange(10, 100)

self.rect = pygame.Rect(self.x, self.y, self.width, self.height)

def clickedInside(self, mousePoint):

继承 **233**

clicked = self.rect.collidepoint(mousePoint)

return clicked

def getArea(self):

theArea = self.width * self.height

return theArea

*代码清单 10-11：实现 clickedInside() 和 getArea()，但没有 draw() 的矩形类* 作为演示，这个类错误地没有包含 draw()

方法。代码清单 10-12 显示了主代码的修改版本，

包含了矩形对象的创建。

**文件：InheritedShapes/Main_ShapesWithRectangle.py**

shapesList = []

shapeClassesTuple = ('Square', 'Circle', 'Triangle', 'Rectangle')

for i in range(0, N_SHAPES):

randomlyChosenClass = random.choice(shapeClassesTuple)

oShape = randomlyChosenClass(window, WINDOW_WIDTH, WINDOW_HEIGHT)

shapesList.append(oShape)

*代码清单 10-12：主代码，它随机创建正方形、圆形、三角形和* *矩形*

当这段代码尝试创建一个矩形对象时，Python 会生成

这个错误信息：

TypeError: 无法实例化抽象类 Rectangle，缺少抽象方法

draw

这告诉我们，我们不能实例化一个矩形对象，因为我们没有

我们在矩形类中没有写 draw() 方法。向其中添加 draw() 方法

矩形类（包含适当的代码来绘制矩形）修复了

错误。

**pygwidgets 如何使用继承**

pygwidgets 模块使用继承来共享公共代码。例如，

例如，考虑我们在[第 7 章：](index_split_003.html#p172)中讨论的两个按钮类：TextButton 和 CustomButton。TextButton 类需要一个字符串作为标签

在按钮上，而 CustomButton 类则要求你提供自己的

艺术。你创建这些类实例的方式是不同的——

不同——你需要指定一组不同的参数。然而，一旦

一旦创建，两个对象的其余方法完全相同。

那是因为这两个类都继承自一个共同的基类，名为

PygWidgetsButton（图 10-7）。

PygWidgetsButton 是一个抽象类。客户端代码不应

创建它的实例，并尝试这样做会生成一个错误

信息。

**234** 第 10 章

![图片 29](index-264_1.png)

（抽象基类）

PygWidgetsButton

继承自

TextButton

CustomButton

*图 10-7：pygwidgets TextButton 和 CustomButton*

*这两个类都继承自 PygWidgetsButton。*

相反，PygWidgetsButton 被 TextButton 和 CustomButton 类继承。

这些类每个都提供了一个方法，__init__()，它

将执行初始化按钮类型所需的任何操作。每个类随后将

将相同的参数传递给基类的 __init__() 方法，

PygWidgetsButton。

TextButton 类用于构建一个基于文本的按钮，功能最简

艺术资源。这在快速启动程序时非常有帮助。

这是创建 TextButton 对象的接口：

def __init__(self, window, loc, text, width=None, height=40,

textColor=PYGWIDGETS_BLACK,

upColor= PYGWIDGETS_NORMAL_GRAY,

overColor= PYGWIDGETS_OVER_GRAY,

downColor=PYGWIDGETS_DOWN_GRAY,

fontName=None, fontSize=20, soundOnClick=None,

enterToActivate=False, callBack=None, nickname=None)

尽管许多参数默认为合理值，调用者

必须提供 text 的值，该值将出现在按钮上。__init__()

方法本身创建按钮的“表面”（图像），这些图像用于显示

显示标准按钮的方式。创建一个典型 TextButton 对象的代码如下

如下所示：

oButton = pygwidgets.TextButton(window, (50, 50), 'Text Button')

绘制时，用户看到的按钮外观如图 10-8 所示。

*图 10-8：一个示例的*

*典型的 TextButton*

CustomButton 类用于使用客户提供的图像构建按钮

提供。以下是创建 CustomButton 的接口：

def __init__(self, window, loc, up, down=None, over=None,

disabled=None, soundOnClick=None,

nickname=None, enterToActivate=False):

继承 **235**

关键的区别是，这个版本的 __init__() 方法要求调用者提供 up 参数的值（记住，一个按钮有

四张图片：up、down、disabled 和 over）。你也可以选择提供

下、上、over 和 disabled 图像。如果没有提供某个图像，

CustomButton 会复制按钮的 up 图像并使用它。

__init__() 方法的最后一行对于 *TextButton 和

CustomButton 类的初始化是对公共基类 __init__() 方法的调用

类，PygWidgetsButton。两个调用都传递了四张按钮图像，以及

与其他参数一起：

super().__init__(window, loc, surfaceUp, surfaceOver,

surfaceDown, surfaceDisabled, buttonRect,

soundOnClick, nickname, enterToActivate, callBack)

从客户的角度来看，你会看到两个完全不同的类

具有许多方法（其中大多数是相同的）。但从实现的角度来看，

从开发者的角度来看，你可以看到继承如何允许我们重写

基类中的单一 __init__() 方法用于提供客户端程序

客户端有两种相似但非常有用的按钮创建方式。这两种

类别除了 __init__() 方法外，其他一切都相同。因此，

按钮的功能方式，以及可用的方法调用

（handleEvent()、draw()、disable()、enable() 等）必须完全相同。

这种继承方式有许多好处。首先，它提供了

提供一致性，既适用于客户端代码，也适用于最终用户：TextButton 和

CustomButton 对象工作原理相同。它还使得修复错误更加容易—

在基类中修复错误意味着你已经修复了所有子类中的错误—

继承它的类。最后，如果你在基类中添加功能，

它会立即在所有继承自基类的类中可用。

**类层次结构**

任何类都可以作为基类，即使是已经继承自其他类的子类。

来自另一个基类。这种关系被称为 *类层次结构*，如图 10-9 所示。

类 A

继承自

类 B

继承自

类 C

*图 10-9：类层次结构*

**236** 第 10 章

在此图中，类 C 继承自类 B，类 B 继承自类 A。

因此，类 C 是一个子类，类 B 是基类，但类 B 也是类 A 的子类。所以，类 B 扮演着两种角色。在这种情况下，类 C

继承了类 B 中所有方法和实例变量的类，不仅如此，

类 A 中的所有方法和实例变量。这种类型的层次结构可以

在构建越来越具体的类时非常有用。类 A 可以是

类 B 很一般，类 C 更详细，类 C 更具体。

图 10-10 提供了一个不同的视角来思考类之间的关系

一个类层次结构。

类 A

类 B

类 C

*图 10-10：一种不同的*

*描绘类层次结构*

在这里，客户端只看到类 C，但这个类由所有

在类 C、B 和 A 中定义的所有方法和实例变量。

pygwidgets 包使用类层次结构来组织所有小部件。首先

pygwidgets 中的类是抽象类 PygWidget，它提供了基本的功能，

它的功能会对所有小部件提供支持。它的代码包含可以

允许显示和隐藏，启用和禁用，获取和设置

获取位置，获取任何小部件的昵称（内部名称）。

pygwidgets 中还有其他作为抽象类使用的类，

包括上述的 PygWidgetsButton，它是

同时适用于 TextButton 和 CustomButton。图 10-11 应该有助于让这段关系变得

关系明确。

PygWidget

继承自

PygWidgetsButton

继承自

TextButton

CustomButton

*图 10-11：pygwidgets 中的类层次结构*

继承 **237**

如你所见，PygWidgetsButton 类既是 PygWidget 的子类，又是 TextButton 和 CustomButton 的基类。

**使用继承编程的难度**

使用继承时，理解如何与继承类交互可能会很困难，

要放在哪里？你会不断问自己：这个应该放在什么地方？应该在哪里？

实例变量应该放在哪个基类中？子类中有足够的公共代码吗？

在基类中创建方法时，应该使用哪些适当的参数？

子类方法中应使用哪些适当的参数？

默认值应在基类中使用，期望被重写或调用

来自子类？

尝试理解所有变量和方法之间的交互会非常复杂，

在类层次结构中调用方法和使用实例变量可能是一个非常困难、棘手且

挫败的任务。这在阅读类的代码时尤其如此

由另一个程序员开发的层次结构。要完全理解它的

发生什么，你通常需要熟悉基类中的代码

类一直向上层次结构。

例如，设想一个层次结构，其中类D是类C的子类，

它是B的子类，B是A的子类。在类D中，你

你可能会遇到根据实例变量值分支的代码，

变得可能，但该变量可能从未在类D的代码中设置过。像这样的情况

为此，你必须查看类C代码中的实例变量。如果没有

如果在类B的代码中没有找到该实例变量，你必须继续查看类B中的代码，依此类推。

在设计类层次结构时，也许避免这种

问题是只调用继承自

层次结构中的上一级代码。在我们的例子中，类D中的代码应该只

调用类C中的方法，而类C只应调用

在类B中的方法等等。这是*法则*的简化版本。

*德墨忒尔*。简单来说，你（指对象）应该只和你的

直接的朋友（附近的对象）永远不要和陌生人（远离的对象）交谈。

对象）。详细讨论超出了本书的范围，但这里

网上有很多相关的参考资料。

另一种方法，我们在[第4章](index_split_001.html#p86)首次讨论过， 是利用*组合*，其中一个对象实例化一个或多个其他

对象。关键区别在于继承用来建模“是一个”的关系，

关系，而组合使用“拥有一个”关系。例如，

如果我们想要一个旋转框小部件（一个可编辑的文本数字字段，

向上和向下箭头），我们可以构建一个SpinBox类，实例化

一个DisplayNumber对象和两个CustomButton对象作为箭头。每个

这些对象已经知道如何处理其用户交互。

**238** 第10章

**多重继承**

你已经看到一个类如何继承另一个类。事实上，Python（像

其他一些编程语言）允许一个类继承多个

只有一个类。这被称为*多重继承*。Python的语法对于继承多个类是相当直接的：

类 *SomeClass*( *<BaseClass1>* , *<BaseClass2>* , ...)：然而，需要注意的是，当你继承的基类包含相同

几乎同名的方法和/或实例变量。Python确实有规则

（称为*方法解析顺序*，或*MRO*）用来解决这些潜在问题。我认为这是一个高级话题，这里不做详细讨论，但如果

如果你想深入了解，详细讨论可以在[ *ht ps://www*](https://www.python.org/download/releases/2.3/mro)找到

[*.python.org/download/releases/2.3/mro.*](https://www.python.org/download/releases/2.3/mro)

**总结**

这是关于继承主题的一章，非常有雄心：继承的艺术

“通过差异编程。”继承的基本思想是构建一个

类（一个子类）包含所有方法和实例变量

另一个类（基类）的类，从而允许你重用现有代码。

你的新子类可以选择使用或重写基类的方法

类，并定义自己的方法。子类中的方法可以找到

通过调用 super() 来使用基类。

我们构建了两个类，InputNumber 和 DisplayMoney，它们提供了高度

可重用的功能。这些类作为子类实现，使用

在 pygwidgets 包中，TextButton 和 CustomButton 类作为基类。

任何使用你的子类的客户端代码将看到一个结合了

结合了子类和基类中定义的方法。任何数量的

可以使用相同的基类构建多个子类。一个抽象类

是一种不打算由客户端代码实例化的类，而是

仅供子类继承使用。抽象方法在一个

基类是必须在每个子类中重写的。

我们通过许多示例演示了继承的概念

在 pygwidgets 包中，包括如何使用 TextButton 和 CustomButton

类都继承自一个共同的基类，PygWidgetsButton。

我展示了如何构建一个类层次结构，其中一个类继承

从另一个类继承，而这个类又继承自第三个类，依此类推。

继承可能很复杂——阅读别人写的代码可能会

可能会让人感到困惑——但正如我们所见，继承实际上可以非常强大。

继承 **239**

**11**

**内存管理**

**被对象使用**

本章将解释一些重要的
