## **6**

**绘制几何形状和分形**

![image](images/common-01.jpg)

本章，我们将首先学习 matplotlib 中的 patches，它使我们能够绘制几何形状，例如圆形、三角形和多边形。然后我们将学习 matplotlib 的动画支持，并编写一个程序来动画化投射物的轨迹。在最后一部分，我们将学习如何绘制 *分形*——通过反复应用简单的几何变换创造出的复杂几何形状。让我们开始吧！

### **使用 Matplotlib 的 Patches 绘制几何形状**

在 matplotlib 中，*patches* 使我们能够绘制几何形状，每个几何形状我们称之为一个 *patch*。例如，你可以指定一个圆的半径和中心，然后将相应的圆形添加到你的图表中。这与我们之前使用 matplotlib 的方式有所不同，之前我们只是提供了要绘制点的 *x* 和 *y* 坐标。然而，在我们编写程序来利用 patches 特性之前，我们需要更深入地了解 matplotlib 图表是如何创建的。考虑以下程序，它使用 matplotlib 绘制了点 (1, 1)、(2, 2) 和 (3, 3)：

>>> import matplotlib.pyplot as plt

>>> x = [1, 2, 3]

>>> y = [1, 2, 3]

>>> plt.plot(x, y)

[<matplotlib.lines.Line2D object at 0x7fe822d67a20>]

>>> plt.show()

该程序创建了一个 matplotlib 窗口，显示了一条通过给定点的直线。在背后，当调用 plt.plot() 函数时，创建了一个 Figure 对象，在其中创建了坐标轴，最后数据在坐标轴中绘制（参见 [图 6-1](ch06.html#ch6fig1)）。[¹](footnote.html#fn03)

![image](images/f06-01.jpg)

*图 6-1：Matplotlib 图表架构*

以下程序重新创建了这个图表，但我们将显式创建 Figure 对象并为其添加坐标轴，而不是仅仅调用 plot() 函数并依赖它来创建这些：

>>> import matplotlib.pyplot as plt

>>> x = [1, 2, 3]

>>> y = [1, 2, 3]

➊ >>> fig = plt.figure()

➋ >>> ax = plt.axes()

>>> plt.plot(x, y)

[<matplotlib.lines.Line2D object at 0x7f9bad1dcc18>]

>>> plt.show()

>>>

在这里，我们使用 figure() 函数在 ➊ 创建了 Figure 对象，然后在 ➋ 使用 axes() 函数创建了坐标轴。axes() 函数也将坐标轴添加到 Figure 对象中。最后两行与之前的程序相同。这次，当我们调用 plot() 函数时，它会看到已经存在一个带有坐标轴的 Figure 对象，并直接开始绘制提供的数据。

除了手动创建Figure和Axes对象外，您还可以使用pyplot模块中的两个不同函数来获取当前Figure和Axes对象的引用。当您调用gcf()函数时，它会返回当前Figure的引用；当您调用gca()函数时，它会返回当前Axes的引用。一个有趣的特点是，如果这些对象还不存在，每个函数都会创建它们。随着我们在本章后续内容中的使用，您将会更清楚地理解这些函数的工作原理。

#### ***绘制圆形***

要绘制一个圆形，您可以将Circle补丁添加到当前的Axes对象中，以下例子演示了这一方法：

'''

使用matplotlib的Circle补丁示例

'''

import matplotlib.pyplot as plt

def create_circle():

➊     circle = plt.Circle((0, 0), radius = 0.5)

return circle

def show_shape(patch):

➋     ax = plt.gca()

ax.add_patch(patch)

plt.axis('scaled')

plt.show()

if __name__ == '__main__':

➌     c = create_circle()

show_shape(c)

在这个程序中，我们将Circle补丁对象的创建与将补丁添加到图形中的操作分成了两个函数：create_circle()和show_shape()。在create_circle()函数中，我们通过创建一个Circle对象并传入中心坐标(0, 0)作为元组，并使用相同名称的关键字参数radius传入半径0.5，来绘制一个圆形。该函数返回创建的Circle对象。

show_shape()函数的编写方式是，它能够与任何matplotlib补丁一起工作。它首先通过在➋处使用gca()函数获取当前Axes对象的引用。然后，它使用add_patch()函数将传入的补丁添加到该对象，最后调用show()函数来显示图形。我们在这里调用了axis()函数，并使用了scaled参数，这基本上告诉matplotlib自动调整轴的限制。在所有使用补丁的程序中，我们都需要有这一语句来自动缩放坐标轴。当然，您也可以像在[第2章](ch02.html#ch02)中所见那样，指定固定的限制值。

在➌处，我们调用create_circle()函数，并使用标签c来引用返回的Circle对象。然后，我们调用show_shape()函数，并将c作为参数传入。当您运行程序时，您将看到一个matplotlib窗口显示出圆形（见[图6-2](ch06.html#ch6fig2)）。

![image](images/f06-02.jpg)

*图6-2：一个中心在(0, 0)，半径为0.5的圆*

如您所见，圆形看起来并不像圆。这是由于自动的纵横比，它决定了*x*轴和*y*轴的长度比例。如果在➋之后插入语句ax.set_aspect('equal')，您会发现圆形确实变成了一个圆。set_aspect()函数用于设置图形的纵横比；使用equal参数，我们要求matplotlib将*x*轴和*y*轴的长度比例设置为1:1。

贴图的边缘颜色和面颜色（填充颜色）可以使用 ec 和 fc 关键字参数来更改。例如，传递 fc='g' 和 ec='r' 将创建一个面颜色为绿色、边缘颜色为红色的圆形。

Matplotlib 支持许多其他的图形对象，比如椭圆（Ellipse）、多边形（Polygon）和矩形（Rectangle）。

#### ***创建动画图形***

有时我们可能希望创建带有动态形状的图形。Matplotlib 的动画支持将帮助我们实现这一目标。在本节结束时，我们将创建一个动态版本的抛物线轨迹绘制程序。

首先，我们来看一个更简单的例子。我们将绘制一个 matplotlib 图形，其中的圆形从小到大增长到一定半径（直到关闭 matplotlib 窗口为止）：

'''

一个正在增长的圆形

'''

from matplotlib import pyplot as plt

从 matplotlib 导入 animation

def create_circle():

circle = plt.Circle((0, 0), 0.05)

return circle

def update_radius(i, circle):

circle.radius = i*0.5

return circle,

def create_animation():

➊     fig = plt.gcf()

ax = plt.axes(xlim=(-10, 10), ylim=(-10, 10))

ax.set_aspect('equal')

circle = create_circle()

➋     ax.add_patch(circle)

➌     anim = animation.FuncAnimation(

fig, update_radius, fargs = (circle,), frames=30, interval=50)

plt.title('简单圆形动画')

plt.show()

if __name__ == '__main__':

create_animation()

我们首先从 matplotlib 包中导入动画模块。create_animation() 函数执行核心功能。在 ➊ 处，它通过 gcf() 函数获取当前的 Figure 对象引用，然后创建一个坐标轴，设置 x 轴和 y 轴的限制为 -10 到 10。接着，它创建一个 Circle 对象，表示一个半径为 0.05、中心点在 (0, 0) 的圆形，并在 ➋ 处将其添加到当前的坐标轴中。然后，我们创建一个 FuncAnimation 对象 ➌，传递我们想要创建的动画的以下数据：

fig 这是当前的 Figure 对象。

update_radius 这个函数将负责绘制*每一帧*。它接收两个参数——一个帧编号，系统在调用时会自动传递，另一个是我们希望每一帧更新的图形对象。这个函数还必须返回该对象。

fargs 这个元组包含传递给 update_radius() 函数的所有参数，除了帧编号。如果没有需要传递的参数，可以不指定此关键字参数。

frames 这是动画中的帧数。我们的 update_radius() 函数会被调用这么多次。在这里，我们随便选择了 30 帧。

interval 这是两个帧之间的时间间隔，单位是毫秒。如果你的动画感觉太慢，可以减少这个值；如果感觉太快，可以增加这个值。

然后，我们使用 title() 函数设置标题，最后使用 show() 函数显示图形。

如前所述，update_radius()函数负责更新每一帧会变化的圆圈属性。在这里，我们将半径设置为i*0.5，其中i是帧数。因此，你会看到一个每帧增长的圆圈，持续30帧——因此，最大圆圈的半径为15。由于坐标轴的限制设置为-10到10，这会给圆圈超出图像尺寸的效果。当你运行程序时，你将看到你的第一个动画图形，如[图6-3](ch06.html#ch6fig3)所示。

你会注意到动画会持续运行，直到你关闭matplotlib窗口。这是默认行为，你可以通过在创建FuncAnimation对象时将关键词参数设置为repeat=False来改变这一行为。

![image](images/f06-03.jpg)

*图6-3：简单的圆圈动画*

**FUNCANIMATION对象与持久性**

你可能已经注意到，在动画圆圈程序中，尽管我们没有在其他地方再次使用它，我们仍然将创建的FuncAnimation对象分配给标签anim。这是由于matplotlib当前的行为问题——它并不会存储对FuncAnimation对象的任何引用，这使得该对象会被Python的垃圾回收机制回收。这意味着动画不会被创建。通过创建一个指向该对象的标签可以防止这种情况发生。

有关此问题的更多信息，您可以关注讨论：[https://github.com/matplotlib/matplotlib/issues/1656/](https://github.com/matplotlib/matplotlib/issues/1656/)。

#### ***动画化抛射物体的轨迹***

在[第2章](ch02.html#ch02)中，我们画出了一个抛射物体的轨迹图。在这里，我们将在此基础上进行扩展，利用matplotlib的动画功能来为轨迹加上动画效果，从而更接近地展示现实中你看到一个球体运动的样子：

'''

动画演示一个物体的抛射运动轨迹

'''

from matplotlib import pyplot as plt

from matplotlib import animation

import math

g = 9.8

def get_intervals(u, theta):

t_flight = 2*u*math.sin(theta)/g

intervals = []

start = 0

interval = 0.005

while start < t_flight:

intervals.append(start)

start = start + interval

return intervals

def update_position(i, circle, intervals, u, theta):

t = intervals[i]

x = u*math.cos(theta)*t

y = u*math.sin(theta)*t - 0.5*g*t*t

circle.center = x, y

return circle,

def create_animation(u, theta):

intervals = get_intervals(u, theta)

xmin = 0

xmax = u*math.cos(theta)*intervals[-1]

ymin = 0

t_max = u*math.sin(theta)/g

➊     ymax = u*math.sin(theta)*t_max - 0.5*g*t_max**2

fig = plt.gcf()

➋     ax = plt.axes(xlim=(xmin, xmax), ylim=(ymin, ymax))

circle = plt.Circle((xmin, ymin), 1.0)

ax.add_patch(circle)

➌     anim = animation.FuncAnimation(fig, update_position,

fargs=(circle, intervals, u, theta),

frames=len(intervals), interval=1,

repeat=False)

plt.title('抛射运动')

plt.xlabel('X')

plt.ylabel('Y')

plt.show()

if __name__ == '__main__':

try:

u = float(input('请输入初始速度（m/s）：'))

theta = float(input('请输入投射角度（度）：'))

except ValueError:

print('你输入了无效的内容')

else:

theta = math.radians(theta)

create_animation(u, theta)

create_animation()函数接受两个参数：u和theta。这些参数对应于程序输入的初速度和投射角度（*θ*）。get_intervals()函数用于找出计算*x*和*y*坐标的时间间隔。这个函数的实现借用了我们在[第2章](ch02.html#ch02)中所使用的相同逻辑，当时我们实现了一个独立的函数frange()来帮助我们。

为了设置动画的坐标轴范围，我们需要找到*x*和*y*的最小值和最大值。每个坐标的最小值为0，这是它们的初始值。*x*坐标的最大值是小球飞行结束时的坐标值，也就是列表intervals中最后一个时间间隔时的值。*y*坐标的最大值是小球处于最高点时的坐标，也就是在➊处，我们使用公式计算该点。

![image](images/e0157-01.jpg)

一旦获得了这些值，我们在➋创建坐标轴，并传入适当的坐标轴范围。在接下来的两条语句中，我们创建了一个表示小球的图形，并通过在(xmin, ymin)—*x*轴和*y*轴的最小坐标位置创建半径为1.0的圆形，将其添加到图形的Axes对象中。

然后我们创建了FuncAnimation对象 ➌，并将当前图形对象以及以下参数传递给它：

update_position 这个函数将在每一帧中改变圆的中心。这里的想法是，每个时间间隔都会创建一帧，因此我们将帧的数量设置为时间间隔的大小（请参阅此列表中的帧描述）。我们计算小球在第i个时间间隔时刻的*x*和*y*坐标，并将圆心设置为这些值。

fargs update_position()函数需要访问时间间隔列表intervals、初速度和theta，这些通过此关键字参数指定。

frames 由于我们每个时间间隔绘制一帧，因此我们将帧的数量设置为intervals列表的大小。

repeat 正如我们在第一个动画示例中所讨论的，动画默认会无限重复。在本例中我们不希望出现这种情况，因此将此关键字设置为False。

当你运行程序时，它会要求输入初始数据，然后创建动画，如[图6-4](ch06.html#ch6fig4)所示。

![image](images/f06-04.jpg)

*图6-4：投射物轨迹的动画*

### **绘制分形图**

分形是由出人意料的简单数学公式生成的复杂几何图案或形状。与圆形和矩形等几何形状相比，分形看起来不规则，似乎没有任何明显的模式或描述，但如果仔细观察，你会发现图案在显现，整个形状是由无数个自身的复制组成的。由于分形涉及对平面中点进行相同*几何变换*的重复应用，计算机程序非常适合用来生成它们。在本章中，我们将学习如何绘制巴恩斯利蕨、谢尔宾斯基三角形和曼德尔布罗集合（后两者在挑战部分）——这些都是分形研究领域中的流行示例。自然界中也充满了分形——一些常见示例包括海岸线、树木和雪花。

#### ***平面中的点变换***

创建分形的基本思想之一是点的变换。给定一个点，*P*(*x*, *y*)，在 *x*-*y* 平面中，一个变换的例子是 *P* (*x*, *y*) → *Q* (*x* + 1, *y* + 1)，这意味着应用变换后，会生成一个新的点 *Q*，该点位于 *P* 的右上方一单位。如果你将 *Q* 作为起始点，就会得到另一个点 *R*，该点位于 *Q* 的右上方一单位。假设起始点 *P* 为 (1, 1)。[图 6-5](ch06.html#ch6fig5) 展示了这些点的样子。

![image](images/f06-05.jpg)

*图 6-5：点* Q *和* R *是通过对点* P *进行两次迭代变换得到的。*

因此，这种变换是一个规则，描述了一个点如何在 *x*-*y* 平面中移动，从初始位置开始，在每次迭代时移动到不同的点。我们可以将变换看作是点在平面中的*轨迹*。现在，考虑到不仅仅有一个变换规则，而是有两个变换规则，并且在每一步中会随机选择其中一个变换。让我们考虑这些规则：

规则 1：*P* 1 (*x*, *y*) → *P* 2 (*x* + 1, *y* – 1)

规则 2：*P* 1 (*x*, *y*) → *P* 2 (*x* + 1, *y* + 1)

假设 *P*1(1, 1) 为起始点。如果我们进行四次迭代，得到的点序列如下：

*P* 1 (1, 1) → *P* 2 (2, 0) （规则 1）

*P* 2 (2, 0) → *P* 3 (3, 1) （规则 2）

*P* 3 (3, 1) → *P* 4 (4, 2) （规则 2）

*P* 4 (4, 2) → *P* 5 (5, 1) （规则 1）

…等等。

变换规则是随机选择的，每个规则被选择的概率相等。无论选择哪一个，点都会向右移动，因为在两种情况下我们都增加了 *x* 坐标。随着点向右移动，它们要么向上移动，要么向下移动，从而形成一个之字形路径。以下程序绘制了一个点在经过指定次数的变换迭代后所经过的路径：

'''

从两个等概率的变换中选择一个的示例

变换

'''

import matplotlib.pyplot as plt

import random

def transformation_1(p):

x = p[0]

y = p[1]

return x + 1, y - 1

def transformation_2(p):

x = p[0]

y = p[1]

return x + 1, y + 1

def transform(p):

➊     # 变换函数列表

transformations = [transformation_1, transformation_2]

# 随机选择一个变换函数并调用它

➋     t = random.choice(transformations)

➌     x, y = t(p)

return x, y

def build_trajectory(p, n):

x = [p[0]]

y = [p[1]]

for i in range(n):

p = transform(p)

x.append(p[0])

y.append(p[1])

return x, y

if __name__ == '__main__':

# 初始点

p = (1, 1)

n = int(input('请输入迭代次数: '))

➍     x, y = build_trajectory(p, n)

# 绘图

➎     plt.plot(x, y)

plt.xlabel('X')

plt.ylabel('Y')

plt.show()

我们定义了两个函数，transformation_1()和transformation_2()，分别对应前面提到的两个变换。在transform()函数中，我们创建一个包含这两个函数名的列表 ➊，然后使用random.choice()函数从列表中随机选择一个变换 ➋。现在我们已经选定了要应用的变换，调用它，并将变换后点的坐标存入x、y标签 ➌，并返回这些坐标。

**从列表中随机选择一个元素**

我们在第一个分形程序中看到的random.choice()函数可以用来从列表中随机选择一个元素。每个元素都有 *相等* 的机会被返回。以下是一个例子：

>>> import random

>>> l = [1, 2, 3]

>>> random.choice(l)

3

>>> random.choice(l)

1

>>> random.choice(l)

1

>>> random.choice(l)

3

>>> random.choice(l)

3

>>> random.choice(l)

2

该函数也适用于元组和字符串。在后者情况下，它会返回字符串中的一个随机字符。

当你运行程序时，它会要求你输入迭代次数 n——也就是变换应用的次数。接着，它会调用build_trajectory()函数，传入n和初始点 *P*，默认设置为(1, 1) ➍。build_trajectory()函数会重复调用transform()函数n次，使用两个列表x和y来存储所有变换点的 *x* 坐标和 *y* 坐标。最后，它会返回这两个列表，之后会进行绘制 ➎。

[图6-6](ch06.html#ch6fig6)和[图6-7](ch06.html#ch6fig7)分别展示了点的轨迹在100次和10,000次迭代后的变化。这种锯齿形的运动在两张图中都非常明显。这个锯齿路径通常被称为 *随机漫步*。

![image](images/f06-06.jpg)

*图6-6：点（1, 1）在经历两个变换之一随机作用100次后的锯齿路径*

![image](images/f06-07.jpg)

*图6-7：点（1, 1）在经历两个变换之一随机作用10,000次后的锯齿路径*

这个例子展示了创建分形的基本思想——从一个初始点开始，反复应用一个变换到该点。接下来，我们将看到应用相同思想来绘制*巴恩斯利蕨*的例子。

#### ***绘制巴恩斯利蕨***

英国数学家迈克尔·巴恩斯利描述了如何通过对一个点反复应用简单的变换来创建蕨类植物的结构（见[图 6-8](ch06.html#ch6fig8)）。

![image](images/f06-08.jpg)

*图 6-8：女士蕨[²](footnote.html#fn04)*

他提出了以下步骤来创建类似蕨类植物的结构：从点 (0, 0) 开始，并*随机*选择以下变换之一，选择的*概率*如下：

**变换 1**（0.85 概率）：

*x*[*n*+1] = 0.85*x[n]* + 0.04*y[n]*

*y*[*n*+1] = –0.04*y[n]* + 0.85*y[n]* + 1.6

**变换 2**（0.07 概率）：

*x*[*n*+1] = 0.2*x[n]* – 0.26*y[n]*

*y*[*n*+1] = 0.23*y[n]* + 0.22*y[n]* + 1.6

**变换 3**（0.07 概率）：

*x*[*n*+1] = –0.15*x[n]* – 0.28*x[n]*

*y*[*n*+1] = 0.26*y[n]* + 0.24*y[n]* + 0.44

**变换 4**（0.01 概率）：

*x*[*n*+1] = 0

*y*[*n*+1] = 0.16*y[n]*

每个变换负责创建蕨类植物的一部分。第一个变换由于具有最高的概率——因此最多的次数——创建了蕨类植物的茎部和底部叶片。第二个和第三个变换分别创建了左侧和右侧的底部叶片，而第四个变换则创建了蕨类植物的茎部。

这是非均匀概率选择的一个例子，我们在[第5章](ch05.html#ch05)首次了解了这个概念。以下程序绘制了指定数量点的巴恩斯利蕨：

'''

绘制巴恩斯利蕨

'''

import random

import matplotlib.pyplot as plt

def transformation_1(p):

x = p[0]

y = p[1]

x1 = 0.85*x + 0.04*y

y1 = -0.04*x + 0.85*y + 1.6

return x1, y1

def transformation_2(p):

x = p[0]

y = p[1]

x1 = 0.2*x - 0.26*y

y1 = 0.23*x + 0.22*y + 1.6

return x1, y1

def transformation_3(p):

x = p[0]

y = p[1]

x1 = -0.15*x + 0.28*y

y1 = 0.26*x + 0.24*y + 0.44

return x1, y1

def transformation_4(p):

x = p[0]

y = p[1]

x1 = 0

y1 = 0.16*y

return x1, y1

def get_index(probability):

r = random.random()

c_probability = 0

sum_probability = []

for p in probability:

c_probability += p

sum_probability.append(c_probability)

for item, sp in enumerate(sum_probability):

if r <= sp

return item

return len(probability)-1

def transform(p):

# 变换函数列表

transformations = [transformation_1, transformation_2,

transformation_3, transformation_4]

➊      probability = [0.85, 0.07, 0.07, 0.01]

# 随机选择一个变换函数并调用它

tindex = get_index(probability)

➋      t = transformations[tindex]

x, y = t(p)

return x, y

def draw_fern(n):

# 我们从 (0, 0) 开始

x = [0]

y = [0]

x1, y1 = 0, 0

for i in range(n):

x1, y1 = transform((x1, y1))

x.append(x1)

y.append(y1)

return x, y

if __name__ == '__main__':

n = int(input('请输入蕨类植物的点数：'))

x, y = draw_fern(n)

# 绘制点

plt.plot(x, y, 'o')

plt.title('蕨类植物，点数为{0}'.format(n))

plt.show()

当你运行这个程序时，它会要求你指定蕨类植物的点数，并随后创建出蕨类植物。 [图6-9](ch06.html#ch6fig9) 和 [6-10](ch06.html#ch6fig10) 分别展示了包含1000个和10000个点的蕨类植物。

![image](images/f06-09.jpg)

*图6-9：含有1000个点的蕨类植物*

![image](images/f06-10.jpg)

*图6-10：含有10000个点的蕨类植物*

四个变换规则分别在 transformation_1()、transformation_2()、transformation_3() 和 transformation_4() 函数中定义。每个规则被选中的概率在➊处的列表中声明，然后每次调用 draw_fern() 函数时，transform() 函数都会应用其中的一个变换。

初始点 (0, 0) 被变换的次数与作为程序输入的蕨类植物点数相同。

### **你学到了什么**

在这一章中，你首先学习了如何绘制基本的几何形状以及如何为它们添加动画。这个过程让你了解了许多新的 matplotlib 功能。接着，你学习了几何变换，并看到了如何通过重复简单的变换帮助你绘制复杂的几何形状——被称为*分形*。

### **编程挑战**

这里有一些编程挑战，应该能帮助你进一步应用所学内容。你可以在 *[http://www.nostarch.com/doingmathwithpython/](http://www.nostarch.com/doingmathwithpython/)* 找到示例解答。

#### ***#1: 将圆形装入正方形***

我之前提到过，matplotlib 支持创建其他几何形状。多边形补丁尤其有趣，因为它允许你绘制具有不同边数的多边形。下面是我们如何绘制一个正方形（每边长度为4）：

'''

绘制一个正方形

'''

from matplotlib import pyplot as plt

def draw_square():

ax = plt.axes(xlim = (0, 6), ylim = (0, 6))

square = plt.Polygon([(1, 1), (5, 1), (5, 5), (1, 5)], closed = True)

ax.add_patch(square)

plt.show()

if __name__ == '__main__':

draw_square()

多边形对象是通过将顶点坐标列表作为第一个参数传入创建的。因为我们要绘制正方形，所以传入四个顶点的坐标：(1, 1)、(5, 1)、(5, 5)、(1, 5)。将 closed=True 传入 matplotlib，表示我们希望绘制一个闭合的多边形，即起始点和终止点是相同的。

在这个挑战中，你将尝试解决“将圆形装入正方形”的简化版本问题。半径为0.5的圆形可以在由此代码生成的正方形中放置多少个？画出来并找出答案！[图6-11](ch06.html#ch6fig11)展示了最终图像的样子。

![image](images/f06-11.jpg)

*图6-11：圆形装入正方形*

这里的技巧是从正方形的左下角开始——也就是(1, 1)——然后不断添加圆圈，直到整个正方形被填满。下面的代码片段展示了如何创建圆圈并将它们添加到图形中：

y = 1.5

while y < 5:

x = 1.5

while x < 5:

c = draw_circle(x, y)

ax.add_patch(c)

x += 1.0

y += 1.0

这里需要注意的是，这*不是*将圆形填充到正方形中的最优方法，或者说，它不是唯一的方式，寻找不同的解决方法在数学爱好者中很受欢迎。

#### ***#2：绘制谢尔宾斯基三角形***

谢尔宾斯基三角形以波兰数学家瓦茨瓦夫·谢尔宾斯基命名，是一个由更小的等边三角形嵌套在其中构成的分形。[图6-12](ch06.html#ch6fig12)展示了由10,000个点构成的谢尔宾斯基三角形。

![image](images/f06-12.jpg)

*图6-12：包含10,000个点的谢尔宾斯基三角形*

有趣的是，我们用来绘制蕨类植物的相同过程也可以绘制谢尔宾斯基三角形——只是转换规则和它们的概率会发生变化。你可以这样绘制谢尔宾斯基三角形：从点(0, 0)开始，并应用以下其中一个转换：

**转换1：**

*x*[*n*+1] = 0.5*x[n]*

*y*[*n*+1] = 0.5*y[n]*

**转换2**：

*x*[*n*+1] = 0.5*x[n]* + 0.5

*y*[*n*+1] = 0.5*y[n]* + 0.5

**转换3**：

*x*[*n*+1] = 0.5*x[n]* + 1

*y*[*n*+1] = 0.5*y[n]*

每个转换的选择概率相等——1/3。你的挑战是编写一个程序，绘制由输入指定的点数构成的谢尔宾斯基三角形。

#### ***#3：探索亨农函数***

1976年，Michel Hénon提出了亨农函数，它描述了一个点*P*(*x*, *y*)的转换规则，如下所示：

*P* (*x*, *y*) → *Q* (*y* + 1 – 1.4*x*², 0.3*x*)

无论初始点如何（只要它离原点不太远），你会看到随着更多点的创建，它们开始沿着弯曲的线排列，如[图6-13](ch06.html#ch6fig13)所示。

![image](images/f06-13.jpg)

*图6-13：包含10,000个点的亨农函数*

你的挑战是编写一个程序，生成一个图形，显示从点(1, 1)开始的20,000次迭代的结果。

额外加分：编写另一个程序，创建一个动画图形，展示点开始沿着曲线排列！请参考* [https://www.youtube.com/watch?v=76ll818RlpQ](https://www.youtube.com/watch?v=76ll818RlpQ)*查看示例。

这是一个动态系统的例子，所有点似乎被吸引的弯曲线被称为*吸引子*。要了解更多关于这个函数、动态系统以及分形的知识，你可以参考Kenneth Falconer的《Fractals: A Very Short Introduction》（牛津大学出版社，2013年）。

#### ***#4：绘制曼德尔布罗特集合***

你的挑战是编写一个程序来绘制*曼德博集合*——这是一个简单规则应用的另一个例子，最终呈现出一个看似复杂的形状（参见[图6-14](ch06.html#ch6fig14)）。不过，在我列出步骤之前，我们将首先了解matplotlib的imshow()函数。

![image](images/f06-14.jpg)

*图6-14：位于（–2.5, –1.0）和（1.0, 1.0）之间的曼德博集合平面*

##### **imshow()函数**

imshow()函数通常用于显示外部图像，如JPEG或PNG图像。你可以在* [http://matplotlib.org/users/image_tutorial.html](http://matplotlib.org/users/image_tutorial.html) *看到一个例子。然而，在这里，我们将使用该函数绘制我们自己通过matplotlib创建的新图像。

考虑笛卡尔平面中* x *和* y *都从0到5的部分。现在，考虑沿每个轴的六个等距点：* x *轴上的(0, 1, 2, 3, 4, 5)以及* y *轴上的相同点集。如果我们取这些点的笛卡尔积，我们将得到在* x *-* y *平面中均匀分布的36个点，坐标为(0, 0)，(0, 1)，... (0, 5)，(1, 0)，(1, 1)，... (1, 5)，... (5, 5)。现在，假设我们要用灰色的不同色调为这些点着色——即，这些点中有些是黑色的，有些是白色的，其他的将随机选取一个介于两者之间的色调。[图6-15](ch06.html#ch6fig15)展示了这个场景。

![image](images/f06-15.jpg)

*图6-15：* x *-* y *平面的一个部分，* x *和* y *都从0到5。我们考虑了区域内36个等距的点，并且每个点都用灰色的色调着色。*

为了创建这个图形，我们需要创建一个包含六个列表的列表。每个列表将包含六个从0到10的整数。每个数字将对应每个点的颜色，0代表黑色，10代表白色。然后，我们将这个列表传递给imshow()函数以及其他必要的参数。

##### **创建一个列表的列表**

一个列表也可以包含列表作为其成员：

>>> l1 = [1, 2, 3]

>>> l2 = [4, 5, 6]

>>> l = [l1, l2]

在这里，我们创建了一个列表l，包含两个列表l1和l2。列表的第一个元素l[0]与l1列表相同，第二个元素l[1]与l2列表相同：

>>> l[0]

[1, 2, 3]

>>> l[1]

[4, 5, 6]

为了引用成员列表中的单个元素，我们必须指定两个索引——l[0][1]表示第一个列表的第二个元素，l[1][2]表示第二个列表的第三个元素，依此类推。

现在我们知道如何处理列表的列表了，我们可以编写程序来创建一个类似于[图6-15](ch06.html#ch6fig15)的图形：

导入 matplotlib.pyplot 作为 plt

导入 matplotlib.cm 作为 cm

导入 random

➊ def initialize_image(x_p, y_p):

图像 = []

for i in range(y_p):

x_colors = []

for j in range(x_p):

x_colors.append(0)

image.append(x_colors)

返回图像

def color_points():

x_p = 6

y_p = 6

image = initialize_image(x_p, y_p)

for i in range(y_p):

for j in range(x_p):

➋             image[i][j] = random.randint(0, 10)

➌     plt.imshow(image, origin='lower', extent=(0, 5, 0, 5),

cmap=cm.Greys_r, interpolation='nearest')

plt.colorbar()

plt.show()

if __name__ == '__main__':

color_points()

在 ➊ 的 initialize_image() 函数创建了一个列表的列表，列表中的每个元素都被初始化为 0。它接受两个参数，x_p 和 y_p，分别对应 *x* 轴和 *y* 轴上的点数。这意味着初始化后的列表 image 将由 x_p 个列表组成，每个列表包含 y_p 个零。

在 color_points() 函数中，一旦从 initialize_image() 获取了图像列表，就将一个介于 0 到 10 之间的随机整数赋值给元素 image[i][j] 在 ➋ 处。当我们将这个随机整数赋值给元素时，我们实际上是在为位于 *y* 轴上 i 步，*x* 轴上 j 步的直角坐标系中的点指定颜色。需要注意的是，imshow() 函数会自动根据点在图像列表中的位置推断出颜色，而不关心它的具体 *x* 和 *y* 坐标。

然后，调用 ➌ 处的 imshow() 函数，将图像作为第一个参数传入。关键字参数 origin='lower' 指定 image[0][0] 中的数字对应于点 (0, 0) 的颜色。关键字参数 extent=(0, 5, 0, 5) 设置图像的左下角和右上角分别为 (0, 0) 和 (5, 5)。关键字参数 cmap=cm.Greys_r 指定我们将创建一个灰度图像。

最后的关键字参数 interpolation='nearest' 指定了当 matplotlib 无法为某个点指定颜色时，会使用与其最近的点的颜色。那这是什么意思呢？请注意，我们只在区域 (0, 5) 和 (5, 5) 中为 36 个点指定颜色。由于该区域中有无限多个点，我们告诉 matplotlib 将未指定颜色的点设置为其最近的点的颜色。这就是你在图形中看到每个点周围出现“颜色框”的原因。

调用 colorbar() 函数在图形中显示一个颜色条，显示哪个整数对应哪个颜色。最后，调用 show() 显示图像。请注意，由于使用了 random.randint() 函数，你的图像颜色将与 [图 6-15](ch06.html#ch6fig15) 中的图像不同。

如果你通过将 color_points() 中的 x_p 和 y_p 设置为，比如 20，来增加每个轴上的点数，你将看到一个类似于 [图 6-16](ch06.html#ch6fig16) 的图形。请注意，颜色框的大小变得更小。如果你进一步增加点数，你将看到框的大小进一步缩小，给人一种每个点都有不同颜色的错觉。

![image](images/f06-16.jpg)

*图 6-16：x-y 平面的一部分，其中* x *和* y *都从 0 到 5。我们在该区域内考虑了 400 个相互等距的点，并用灰色的不同色调为每个点上色。*

##### **绘制曼德布罗特集合**

我们将考虑 x-y 平面中区域（–2.5，–1.0）和（1.0，1.0）之间的面积，并将每个轴划分为 400 个等距的点。这些点的笛卡尔积将给我们 1,600 个等距的点。我们将这些点称为（*x*[1]，*y*[1]），（*x*[1]，*y*[2]）……（*x*[400]，*y*[400]）。

创建一个列表，image，通过调用之前看到的 initialize_image() 函数，并将 x_p 和 y_p 都设置为 400。然后，按照以下步骤处理生成的每个点（*x[i]*，*y[k]*）：

1\. 首先，创建两个复数，*z*[1] = 0 + 0*j* 和 *c* = *x[i]* + *y[k] j*。（请记住，我们用 *j* 来表示 ![image](images/e0176-01.jpg)。）

2\. 创建一个标签迭代并将其设置为 0——即，iteration=0。

3\. 创建一个复数，![image](images/e0176-02.jpg)。

4\. 将迭代值增加 1——即，iteration = iteration + 1。

5\. 如果 abs(z1) < 2 且 iteration < max_iteration，则返回第 3 步；否则，转到第 6 步。max_iteration 的值越大，图像越详细，但创建图像的时间也会更长。在这里将 max_iteration 设置为 1,000。

6\. 将点（*x[i]*，*y[k]*）的颜色设置为迭代值——即，image[k][i] = iteration。

一旦你得到完整的图像列表，调用 imshow() 函数，并更改 extent 关键字参数，以指示由（–2.5，–1.0）和（1.0，1.0）所界定的区域。

该算法通常称为 *逃逸时间算法*。当一个点的幅度在最大迭代次数达到之前未超过 2 时，该点属于曼德布罗特集合并显示为白色。那些在较少迭代次数内就超出幅度的点被称为“逃逸”；它们不属于曼德布罗特集合，且显示为黑色。你可以通过减少或增加每个轴上的点数来进行实验。减少点数会导致图像颗粒感增强，而增加点数则会得到更加详细的图像。
