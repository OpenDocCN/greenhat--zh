# 15

生成数据

![](img/chapterart.png)

*数据可视化* 是利用视觉表示来探索和展示数据集中的模式。它与 *数据分析* 密切相关，数据分析使用代码来探索数据集中的模式和联系。一个数据集可以是一个简单的数字列表，可以用一行代码表示，或者它也可以是包含多种信息的数太字节数据。

创建有效的数据可视化不仅仅是让信息看起来好看。当数据集的表示形式简单且具有视觉吸引力时，它的含义会变得对观众更为清晰。人们会在你的数据集中看到以前从未意识到的模式和意义。

幸运的是，你不需要超级计算机就能可视化复杂的数据。Python 非常高效，使用一台笔记本电脑，你就可以快速探索包含数百万个数据点的数据集。这些数据点不一定是数字；利用你在本书第一部分学到的基础知识，你也可以分析非数字数据。

人们使用 Python 进行基因学、气候研究、政治与经济分析等数据密集型工作。数据科学家们为 Python 编写了令人印象深刻的可视化和分析工具，其中许多工具也可以供你使用。最流行的工具之一是 Matplotlib，一个数学绘图库。在本章中，我们将使用 Matplotlib 制作简单的图表，如折线图和散点图。接下来，我们将基于随机游走的概念创建一个更有趣的数据集——这是通过一系列随机决策生成的可视化。

我们还将使用一个名为 Plotly 的包，它可以创建在数字设备上效果良好的可视化，来分析掷骰子的结果。Plotly 生成的可视化会自动调整大小，以适应各种显示设备。这些可视化还可以包括一些交互功能，例如当用户将鼠标悬停在可视化的不同部分时，突出显示数据集的某些方面。学习使用 Matplotlib 和 Plotly 将帮助你开始可视化你最感兴趣的数据类型。

## 安装 Matplotlib

要使用 Matplotlib 进行初步的可视化，你需要像在第十一章中安装 pytest（见“使用 pip 安装 pytest”，第 210 页）一样，使用 pip 安装它。

要安装 Matplotlib，在终端提示符下输入以下命令：

```py
$ **python -m pip install --user matplotlib**
```

如果你使用的命令不是 `python` 来运行程序或启动终端会话，比如 `python3`，那么你的命令应该像这样：

```py
$ **python3 -m pip install --user matplotlib**
```

要查看你可以使用 Matplotlib 制作的可视化类型，访问 Matplotlib 官网 [`matplotlib.org`](https://matplotlib.org)，然后点击 **Plot types**。当你点击画廊中的某个可视化时，你将看到生成该图表所使用的代码。

## 绘制简单的折线图

让我们使用 Matplotlib 绘制一个简单的线性图，然后对其进行自定义，创建一个更具信息量的数据可视化图表。我们将使用平方数序列 1、4、9、16 和 25 作为图表的数据。

要制作一个简单的线性图，只需指定你要使用的数字，让 Matplotlib 来完成剩下的工作：

**mpl_squares.py**

```py
import matplotlib.pyplot as plt

squares = [1, 4, 9, 16, 25]

❶ fig, ax = plt.subplots()
ax.plot(squares)

plt.show()
```

我们首先导入 `pyplot` 模块，并使用别名 `plt`，这样就不需要重复输入 `pyplot` 了。（你会在在线示例中经常看到这种约定，所以我们也在这里使用它。）`pyplot` 模块包含许多函数，帮助生成图表和图形。

我们创建一个名为 `squares` 的列表来保存我们要绘制的数据。接着，我们遵循 Matplotlib 中的另一个常见约定，调用 `subplots()` 函数 ❶。这个函数可以在同一图形中生成一个或多个子图。变量 `fig` 代表整个*图形*，即生成的所有子图集合。变量 `ax` 代表图形中的一个单独子图；这是我们大多数时候用来定义和自定义单个图表的变量。

然后，我们使用`plot()`方法，它会尝试以有意义的方式绘制给定的数据。函数`plt.show()`打开 Matplotlib 的查看器并显示图表，如图 15-1 所示。查看器允许你缩放和浏览图表，并且你可以通过点击磁盘图标保存你喜欢的任何图表图像。

![](img/f15001.png)

图 15-1：在 Matplotlib 中，你可以制作的最简单的图表之一

### 更改标签类型和线条粗细

尽管图 15-1 中的图表显示数字在增加，但标签类型太小，线条也有些细，难以轻松阅读。幸运的是，Matplotlib 允许你调整可视化的每一个特性。

我们将使用一些可用的自定义选项来提高图表的可读性。让我们从添加标题和标注坐标轴开始：

**mpl_squares.py**

```py
import matplotlib.pyplot as plt

squares = [1, 4, 9, 16, 25]

fig, ax = plt.subplots()
❶ ax.plot(squares, linewidth=3)

# Set chart title and label axes.
❷ ax.set_title("Square Numbers", fontsize=24)
❸ ax.set_xlabel("Value", fontsize=14)
ax.set_ylabel("Square of Value", fontsize=14)

# Set size of tick labels.
❹ ax.tick_params(labelsize=14)

plt.show()
```

`linewidth` 参数控制 `plot()` 生成的线条的粗细 ❶。一旦图表生成，仍然有许多方法可以修改图表，直到它被展示出来。`set_title()` 方法为整个图表设置标题 ❷。`fontsize` 参数（在代码中反复出现）控制图表中各个元素的文字大小。

`set_xlabel()` 和 `set_ylabel()` 方法允许你为每个坐标轴设置标题 ❸，而 `tick_params()` 方法用于设置刻度线样式 ❹。这里，`tick_params()` 将两个坐标轴上的刻度标签字体大小设置为 14。

如图 15-2 所示，生成的图表更容易阅读了。标签字体更大，线条图更粗了。通常，值得尝试调整这些值，看看在最终图表中哪些效果最好。

![](img/f15002.png)

图 15-2：现在图表更容易阅读了。

### 修正图表

现在我们能更清晰地阅读图表，发现数据并没有正确绘制。注意在图表的末尾，4.0 的平方被错误地显示为 25！让我们来修正这个问题。

当你给`plot()`传递一个数字序列时，它会假设第一个数据点对应的*x*值为 0，但我们的第一个点对应的*x*值是 1。我们可以通过给`plot()`提供用于计算平方的输入和输出值来覆盖默认行为：

**mpl_squares.py**

```py
import matplotlib.pyplot as plt

input_values = [1, 2, 3, 4, 5]
squares = [1, 4, 9, 16, 25]

fig, ax = plt.subplots()
ax.plot(input_values, squares, linewidth=3)

# Set chart title and label axes.
*--snip--*
```

现在，`plot()`不需要假设输出的数字是如何生成的。生成的图表，如图 15-3 所示，是正确的。

![](img/f15003.png)

图 15-3：数据现在已经正确绘制。

你可以在调用`plot()`时指定多个参数，并在生成图表后使用多种方法来定制图表。我们将在本章中继续探索这些定制方法，并与更有趣的数据集一起工作。

### 使用内置样式

Matplotlib 有许多预定义的样式可供使用。这些样式包含各种默认设置，涵盖背景颜色、网格线、线宽、字体、字号等。它们可以让你的可视化效果更具吸引力，而无需太多的自定义。要查看所有可用样式的完整列表，请在终端会话中运行以下命令：

```py
>>> **import matplotlib.pyplot as plt**
>>> **plt.style.available**
['Solarize_Light2', '_classic_test_patch', '_mpl-gallery',
`--snip--`
```

要使用这些样式中的任何一个，只需在调用`subplots()`之前添加一行代码：

**mpl_squares.py**

```py
import matplotlib.pyplot as plt

input_values = [1, 2, 3, 4, 5]
squares = [1, 4, 9, 16, 25]

plt.style.use('seaborn')
fig, ax = plt.subplots()
*--snip--*
```

这段代码生成了如图 15-4 所示的图表。提供了多种样式；可以尝试这些样式，找到你喜欢的样式。

![](img/f15004.png)

图 15-4：内置的 seaborn 样式

### 使用`scatter()`绘制和样式化单个点

有时候，根据某些特征绘制和样式化单个点是很有用的。例如，你可以用一种颜色绘制较小的值，另一种颜色绘制较大的值。你还可以用一组样式选项绘制大型数据集，然后通过重新绘制个别点并使用不同的选项来强调它们。

要绘制一个单独的点，可以将该点的单个*x*和*y*值传递给`scatter()`：

**scatter_squares.py**

```py
import matplotlib.pyplot as plt

plt.style.use('seaborn')
fig, ax = plt.subplots()
ax.scatter(2, 4)

plt.show()
```

让我们为输出添加样式，使其更有趣。我们将添加标题，标记坐标轴，并确保所有文本都足够大以便阅读：

```py
import matplotlib.pyplot as plt

plt.style.use('seaborn')
fig, ax = plt.subplots()
❶ ax.scatter(2, 4, s=200)

# Set chart title and label axes.
ax.set_title("Square Numbers", fontsize=24)
ax.set_xlabel("Value", fontsize=14)
ax.set_ylabel("Square of Value", fontsize=14)

# Set size of tick labels.
ax.tick_params(labelsize=14)

plt.show()
```

我们调用`scatter()`并使用`s`参数设置绘制图表时使用的点的大小❶。现在运行*scatter_squares.py*，你应该能看到图表中心有一个单独的点，如图 15-5 所示。

![](img/f15005.png)

图 15-5：绘制一个单独的点

### 使用`scatter()`绘制一系列点

要绘制一系列点，我们可以将独立的*x*和*y*值列表传递给`scatter()`，像这样：

**scatter_squares.py**

```py
import matplotlib.pyplot as plt

x_values = [1, 2, 3, 4, 5]
y_values = [1, 4, 9, 16, 25]

plt.style.use('seaborn')
fig, ax = plt.subplots()
ax.scatter(x_values, y_values, s=100)

# Set chart title and label axes.
*--snip--*
```

`x_values`列表包含待平方的数字，而`y_values`包含每个数字的平方。当这些列表传递给`scatter()`时，Matplotlib 会在绘制每个点时从每个列表中读取一个值。待绘制的点为（1，1），（2，4），（3，9），（4，16）和（5，25）；图 15-6 显示了结果。

![](img/f15006.png)

图 15-6：具有多个点的散点图

### 自动计算数据

手动编写列表可能效率低下，尤其是当我们有很多点时。与其逐个写出每个值，不如使用循环来进行计算。

以下是使用 1,000 个点时的效果：

**scatter_squares.py**

```py
import matplotlib.pyplot as plt

❶ x_values = range(1, 1001)
y_values = [x**2 for x in x_values]

plt.style.use('seaborn')
fig, ax = plt.subplots()
❷ ax.scatter(x_values, y_values, s=10)

# Set chart title and label axes.
*--snip--*

# Set the range for each axis.
❸ ax.axis([0, 1100, 0, 1_100_000])

plt.show()
```

我们从一个包含 1 到 1,000 的数字范围的*x*值开始❶。接下来，列表推导通过循环遍历*x*值（`for x in x_values`），将每个数字平方（`x**2`），并将结果赋值给`y_values`。然后，我们将输入和输出列表传递给`scatter()`❷。由于这是一个大数据集，我们使用较小的点大小。

在显示图表之前，我们使用`axis()`方法指定每个轴的范围❸。`axis()`方法需要四个值：*x*轴和*y*轴的最小值和最大值。在这里，我们将*x*轴范围设置为 0 到 1,100，*y*轴范围设置为 0 到 1,100,000。图 15-7 显示了结果。

![](img/f15007.png)

图 15-7：Python 绘制 1,000 个点和绘制 5 个点一样容易。

### 自定义刻度标签

当轴上的数字足够大时，Matplotlib 默认使用科学计数法表示刻度标签。这通常是一个好事，因为用常规表示法表示大数字会占用可视化中大量不必要的空间。

几乎每个图表元素都可以自定义，因此如果你更喜欢常规表示法，可以告诉 Matplotlib 继续使用普通表示法：

```py
*--snip--*
# Set the range for each axis.
ax.axis([0, 1100, 0, 1_100_000])
ax.ticklabel_format(style='plain')

plt.show()
```

`ticklabel_format()`方法允许你覆盖任何图表的默认刻度标签样式。

### 定义自定义颜色

要更改点的颜色，将`color`参数传递给`scatter()`，并使用引号括起颜色名称，如下所示：

```py
ax.scatter(x_values, y_values, color='red', s=10)
```

你还可以使用 RGB 颜色模型定义自定义颜色。要定义颜色，将`color`参数传递一个包含三个浮动值的元组（分别代表红色、绿色和蓝色），这些值的范围是 0 到 1。例如，以下代码行创建一个包含浅绿色点的图：

```py
ax.scatter(x_values, y_values, color=(0, 0.8, 0), s=10)
```

靠近 0 的值会产生较暗的颜色，而靠近 1 的值会产生较浅的颜色。

### 使用色彩图

*色彩图*是一个渐变的颜色序列，从起始颜色到结束颜色。在可视化中，色彩图用于强调数据中的模式。例如，你可以将低值设为浅色，将高值设为深色。使用色彩图可以确保可视化中的所有点在一个精心设计的颜色尺度上平滑、准确地变化。

`pyplot` 模块包含一组内置的颜色映射。要使用这些颜色映射之一，你需要指定 `pyplot` 如何根据数据集中的每个点的 *y* 值来分配颜色。以下是如何根据每个点的 *y* 值来为其分配颜色：

**scatter_squares.py**

```py
*--snip--*
plt.style.use('seaborn')
fig, ax = plt.subplots()
ax.scatter(x_values, y_values, c=y_values, cmap=plt.cm.Blues, s=10)

# Set chart title and label axes.
*--snip--*
```

`c` 参数类似于 `color`，但它用于将一系列值与颜色映射关联。我们将 *y* 值列表传递给 `c`，然后通过 `cmap` 参数告诉 `pyplot` 使用哪种颜色映射。此代码将较低 *y* 值的点颜色设置为浅蓝色，将较高 *y* 值的点颜色设置为深蓝色。图 15-8 显示了结果图。

![](img/f15008.png)

图 15-8：使用 `Blues` 颜色映射的图像

### 自动保存你的图像

如果你想将图像保存到文件而不是在 Matplotlib 查看器中显示它，可以使用 `plt.savefig()` 替代 `plt.show()`：

```py
plt.savefig('squares_plot.png', bbox_inches='tight')
```

第一个参数是图像文件的文件名，图像将保存在与 *scatter_squares.py* 相同的目录中。第二个参数用于修剪图像周围的额外空白。如果你希望保留图像周围的空白，可以省略此参数。你也可以使用 `Path` 对象调用 `savefig()`，并将输出文件保存到系统中的任何位置。

## 随机漫步

在本节中，我们将使用 Python 生成随机漫步的数据，并使用 Matplotlib 创建该数据的视觉表示。*随机漫步* 是由一系列简单决策决定的路径，每个决策完全依赖于随机性。你可以把随机漫步想象成一只迷失的蚂蚁，如果它每走一步都是随机选择方向的话，它走的路径就是随机漫步。

随机漫步在自然界、物理学、生物学、化学和经济学中有许多实际应用。例如，漂浮在水滴上的花粉粒子因不断受到水分子推动而在水面上移动。水滴中的分子运动是随机的，因此花粉粒子在水面上留下的轨迹就是一个随机漫步。我们接下来编写的代码将模拟许多现实世界的情况。

### 创建 RandomWalk 类

为了创建一个随机漫步，我们将创建一个 `RandomWalk` 类，该类将做出关于漫步方向的随机决定。这个类需要三个属性：一个变量来跟踪漫步中的点数，两个列表用来存储漫步中每个点的 *x* 和 *y* 坐标。

我们只需要为 `RandomWalk` 类编写两个方法：`__init__()` 方法和 `fill_walk()` 方法，后者将计算漫步中的各个点。我们先从 `__init__()` 方法开始：

**random_walk.py**

```py
❶ from random import choice

class RandomWalk:
    """A class to generate random walks."""

❷     def __init__(self, num_points=5000):
        """Initialize attributes of a walk."""
        self.num_points = num_points

        # All walks start at (0, 0).
❸         self.x_values = [0]
        self.y_values = [0]
```

为了做出随机决策，我们将在一个列表中存储可能的移动，并使用`choice()`函数（来自`random`模块）来决定每次走一步时的移动方向❶。我们将默认的步数设置为`5000`，这个值足够大，可以生成一些有趣的图案，同时又足够小，可以快速生成游走❷。然后我们创建两个列表来保存* x *和* y *的值，并将每次游走的起点设为(0, 0)❸。

### 选择方向

我们将使用`fill_walk()`方法来确定游走中的完整点序列。将此方法添加到*random_walk.py*中：

**random_walk.py**

```py
def fill_walk(self):
"""Calculate all the points in the walk."""

 # Keep taking steps until the walk reaches the desired length.
❶     while len(self.x_values) < self.num_points:

        # Decide which direction to go, and how far to go.
❷         x_direction = choice([1, -1])
        x_distance = choice([0, 1, 2, 3, 4])
❸         x_step = x_direction * x_distance

        y_direction = choice([1, -1])
        y_distance = choice([0, 1, 2, 3, 4])
❹         y_step = y_direction * y_distance

        # Reject moves that go nowhere.
❺         if x_step == 0 and y_step == 0:
            continue

        # Calculate the new position.
❻         x = self.x_values[-1] + x_step
        y = self.y_values[-1] + y_step

        self.x_values.append(x)
        self.y_values.append(y)
```

我们首先设置一个循环，直到游走填充了正确数量的点❶。`fill_walk()`的主要部分告诉 Python 如何模拟四个随机决策：游走是向右还是向左？在该方向上将移动多远？游走是向上还是向下？在该方向上将移动多远？

我们使用`choice([1, -1])`来为`x_direction`选择一个值，返回值为 1 表示向右移动，返回值为−1 表示向左移动❷。接着，`choice([0, 1, 2, 3, 4])`会随机选择一个距离来决定在该方向上移动的步数。我们将该值赋给`x_distance`。包括 0 的选择允许步伐仅在一个轴上移动。

我们通过将运动方向与选定的距离相乘来确定每一步在* x *和* y *方向上的步长❸❹。`x_step`为正值表示向右移动，负值表示向左移动，0 表示垂直移动。`y_step`为正值表示向上移动，负值表示向下移动，0 表示水平移动。如果`x_step`和`y_step`的值都为 0，则表示游走没有前进；在这种情况下，我们会继续循环❺。

为了获得下一个游走的* x *值，我们将`x_step`中的值加到`x_values`中最后存储的值❻，* y *值同理。获得新的点的坐标后，我们将它们附加到`x_values`和`y_values`中。

### 绘制随机游走

以下是绘制游走中所有点的代码：

**rw_visual.py**

```py
import matplotlib.pyplot as plt

from random_walk import RandomWalk

# Make a random walk.
❶ rw = RandomWalk()
rw.fill_walk()

# Plot the points in the walk.
plt.style.use('classic')
fig, ax = plt.subplots()
❷ ax.scatter(rw.x_values, rw.y_values, s=15)
❸ ax.set_aspect('equal')
plt.show()
```

我们首先导入`pyplot`和`RandomWalk`。然后我们创建一个随机游走并将其赋值给`rw`❶，确保调用`fill_walk()`。为了可视化游走，我们将游走的* x *和* y *值传递给`scatter()`并选择合适的点大小❷。默认情况下，Matplotlib 会独立缩放每个坐标轴。但这种方式会使大多数游走在水平方向或垂直方向上拉伸。这里我们使用`set_aspect()`方法来指定两个坐标轴之间的刻度间距应保持相等❸。

图 15-9 展示了生成的包含 5,000 个点的图表。本节中的图像省略了 Matplotlib 的查看器，但当你运行*rw_visual.py*时，你仍然会看到它。

![](img/f15009.png)

图 15-9：一个包含 5,000 个点的随机游走

### 生成多个随机游走

每次随机漫步都是不同的，探索生成的各种模式非常有趣。使用前面的代码生成多个漫步而不需要多次运行程序的一种方式是将其包装在`while`循环中，如下所示：

**rw_visual.py**

```py
import matplotlib.pyplot as plt

from random_walk import RandomWalk

# Keep making new walks, as long as the program is active.
while True:
 # Make a random walk.
    *--snip--*
 plt.show()

    keep_running = input("Make another walk? (y/n): ")
    if keep_running == 'n':
        break
```

这段代码生成一个随机漫步，将其显示在 Matplotlib 的查看器中，并暂停等待查看器关闭。当你关闭查看器时，系统会询问你是否要生成另一个漫步。如果你生成几个漫步，你应该会看到一些漫步保持在起点附近，一些漫步主要朝一个方向走，某些漫步有细长的部分连接着多个点，还有很多其他种类的漫步。当你想结束程序时，按 N 键。

### 漫步的样式

在这一节中，我们将自定义我们的图表，突出显示每次随机漫步的关键特征，并淡化干扰元素。为此，我们确定需要强调的特征，如漫步的起点、终点以及所走的路径。接下来，我们确定需要淡化的特征，如刻度标记和标签。最终结果应该是一个简单的视觉表示，清晰地传达每次随机漫步的路径。

#### 给点着色

我们将使用颜色图来展示漫步中各点的顺序，并去除每个点的黑色轮廓，使点的颜色更加清晰。为了根据点在漫步中的位置为其着色，我们将包含每个点位置的列表传递给`c`参数。由于这些点是按顺序绘制的，这个列表只包含从 0 到 4,999 的数字：

**rw_visual.py**

```py
*--snip--*
while True:
 # Make a random walk.
 rw = RandomWalk()
 rw.fill_walk()

    # Plot the points in the walk.
 plt.style.use('classic')
 fig, ax = plt.subplots()
❶     point_numbers = range(rw.num_points)
    ax.scatter(rw.x_values, rw.y_values, c=point_numbers, cmap=plt.cm.Blues,
        edgecolors='none', s=15)
    ax.set_aspect('equal')
 plt.show()
  *--snip--*
```

我们使用`range()`生成一个与漫步点数相等的数字列表❶。我们将这个列表赋值给`point_numbers`，并将其用于设置每个点的颜色。我们将`point_numbers`传递给`c`参数，使用`Blues`颜色图，然后传递`edgecolors='none'`来去除每个点周围的黑色轮廓。结果是一个从浅蓝到深蓝变化的图表，准确地显示了漫步从起点到终点的过程。如图 15-10 所示。

![](img/f15010.png)

图 15-10: 使用`Blues`颜色图着色的随机漫步

#### 绘制起点和终点

除了通过颜色来标识每个点在漫步中的位置外，显示每次漫步的起点和终点也是很有用的。为此，我们可以在主要序列绘制完成后，单独绘制第一个和最后一个点。我们将使终点的点更大，并使用不同的颜色使其更加突出：

**rw_visual.py**

```py
*--snip--*
while True:
  *--snip--*
 ax.scatter(rw.x_values, rw.y_values, c=point_numbers, cmap=plt.cm.Blues,
 edgecolors='none', s=15)
 ax.set_aspect('equal')

    # Emphasize the first and last points.
    ax.scatter(0, 0, c='green', edgecolors='none', s=100)
    ax.scatter(rw.x_values[-1], rw.y_values[-1], c='red', edgecolors='none',
        s=100)

 plt.show()
  *--snip--*
```

为了显示起点，我们将点(0, 0)以绿色和比其他点更大的尺寸（`s=100`）绘制。为了标记终点，我们也将最后的 *x* 和 *y* 值以红色绘制，尺寸为 100。确保将这段代码插入到`plt.show()`调用之前，以便起点和终点能够绘制在所有其他点的上面。

当你运行这段代码时，你应该能够准确找到每次漫步的开始和结束位置。如果这些端点不够显眼，可以调整它们的颜色和大小，直到它们清晰可见。

#### 清理坐标轴

让我们移除此图中的坐标轴，以避免它们分散每次漫步的注意力。以下是隐藏坐标轴的方法：

**rw_visual.py**

```py
*--snip--*
while True:
  *--snip--*
 ax.scatter(rw.x_values[-1], rw.y_values[-1], c='red', edgecolors='none',
 s=100)

    # Remove the axes.
    ax.get_xaxis().set_visible(False)
    ax.get_yaxis().set_visible(False)

 plt.show()
  *--snip--*
```

为了修改坐标轴，我们使用 `ax.get_xaxis()` 和 `ax.get_yaxis()` 方法来获取每个坐标轴，然后通过链式调用 `set_visible()` 方法使每个坐标轴变得不可见。随着你继续进行可视化工作，你将经常看到这种方法链的使用，用来定制可视化的不同方面。

现在运行 *rw_visual.py*；你应该能看到一系列没有坐标轴的图表。

#### 添加绘图点

让我们增加点的数量，以便为我们提供更多的数据来处理。为此，我们在创建 `RandomWalk` 实例时增加 `num_points` 的值，并在绘制图表时调整每个点的大小：

**rw_visual.py**

```py
*--snip--*
while True:
 # Make a random walk.
    rw = RandomWalk(50_000)
 rw.fill_walk()

 # Plot the points in the walk.
 plt.style.use('classic')
 fig, ax = plt.subplots()
 point_numbers = range(rw.num_points)
  ax.scatter(rw.x_values, rw.y_values, c=point_numbers, cmap=plt.cm.Blues,
        edgecolors='none', s=1)
  *--snip--*
```

这个例子创建了一个包含 50,000 个点的随机漫步，并且将每个点的大小设置为 `s=1`。最终的随机漫步呈现出云雾状，如 图 15-11 所示。我们已经从一个简单的散点图中创造出了一幅艺术作品！

尝试调整此代码，看看你可以在系统开始显著变慢或图表失去视觉吸引力之前，增加多少步数。

![](img/f15011.png)

图 15-11：50,000 个点的随机漫步

#### 调整大小以填充屏幕

如果可视化能够很好地适应屏幕，它在传达数据模式时会更加有效。为了使绘图窗口更好地适应屏幕，你可以调整 Matplotlib 输出的大小。这是在 `subplots()` 调用中完成的：

```py
fig, ax = plt.subplots(figsize=(15, 9))
```

在创建图表时，你可以将 `subplots()` 的 `figsize` 参数传递进去，这样可以设置图形的大小。`figsize` 参数接受一个元组，告诉 Matplotlib 绘图窗口的尺寸（单位为英寸）。

Matplotlib 假设你的屏幕分辨率为每英寸 100 像素；如果这段代码没有给出准确的图表大小，请根据需要调整数字。或者，如果你知道系统的分辨率，可以使用 `dpi` 参数将分辨率传递给 `subplots()`：

```py
fig, ax = plt.subplots(figsize=(10, 6), dpi=128)
```

这应该有助于更有效地利用屏幕上可用的空间。

## 使用 Plotly 掷骰子

在本节中，我们将使用 Plotly 来制作交互式可视化。Plotly 特别适用于在浏览器中显示的可视化，因为这些可视化会自动缩放以适应查看者的屏幕。它们也是交互式的；当用户将鼠标悬停在屏幕上的某些元素上时，这些元素的信息会被高亮显示。我们将使用 *Plotly Express* 来构建最初的可视化，Plotly Express 是 Plotly 的一个子集，专注于以尽可能少的代码生成图表。一旦确认我们的图表正确无误，我们将像使用 Matplotlib 一样自定义输出。

在这个项目中，我们将分析投掷骰子的结果。当你投掷一个普通的六面骰时，每个数字从 1 到 6 出现的概率是相等的。然而，当你使用两个骰子时，某些数字比其他数字更可能出现。我们将尝试通过生成一个代表投掷骰子的 数据集来确定哪些数字最有可能出现。然后我们将绘制大量投掷的结果，以确定哪些结果比其他结果更有可能。

这项工作有助于建模涉及骰子的游戏，但其核心思想也适用于任何形式的机会游戏，如扑克牌游戏。它还与许多现实世界中的情况相关，这些情况中的随机性扮演着重要角色。

### 安装 Plotly

使用 pip 安装 Plotly，就像安装 Matplotlib 一样：

```py
$ **python -m pip install --user plotly**
$ **python -m pip install --user pandas**
```

Plotly Express 依赖于 *pandas*，这是一个高效处理数据的库，因此我们也需要安装它。如果在安装 Matplotlib 时使用了 `python3` 或其他版本，确保在这里使用相同的命令。

要查看 Plotly 能实现哪些可视化，请访问 [`plotly.com/python`](https://plotly.com/python) 上的图表类型画廊。每个示例都包括源代码，您可以查看 Plotly 如何生成这些可视化。

### 创建 Die 类

我们将创建以下 `Die` 类来模拟投掷一个骰子：

**die.py**

```py
from random import randint

class Die:
    """A class representing a single die."""

❶     def __init__(self, num_sides=6):
        """Assume a six-sided die."""
        self.num_sides = num_sides

    def roll(self):
        """"Return a random value between 1 and number of sides."""
❷         return randint(1, self.num_sides)
```

`__init__()` 方法接受一个可选参数 ❶。在 `Die` 类中，当创建骰子的实例时，如果没有传入参数，骰子的面数默认为六个。如果传入了参数，则该值将设置骰子的面数。（骰子的命名取决于面数：六面骰为 D6，八面骰为 D8，依此类推。）

`roll()` 方法使用 `randint()` 函数返回一个介于 1 和面数之间的随机数 ❷。此函数可以返回起始值（1）、结束值（`num_sides`）或介于两者之间的任何整数。

### 投掷骰子

在创建基于 `Die` 类的可视化之前，让我们先投掷一个 D6，打印结果，并检查这些结果是否合理：

**die_visual.py**

```py
from die import Die

# Create a D6.
❶ die = Die()

# Make some rolls, and store results in a list.
results = []
❷ for roll_num in range(100):
    result = die.roll()
    results.append(result)

print(results)
```

我们创建一个 `Die` 类的实例，默认有六个面 ❶。然后我们投掷骰子 `100` 次 ❷，并将每次投掷的结果存储在 `results` 列表中。以下是一个结果示例：

```py
[4, 6, 5, 6, 1, 5, 6, 3, 5, 3, 5, 3, 2, 2, 1, 3, 1, 5, 3, 6, 3, 6, 5, 4,  1, 1, 4, 2, 3, 6, 4, 2, 6, 4, 1, 3, 2, 5, 6, 3, 6, 2, 1, 1, 3, 4, 1, 4, 3, 5, 1, 4, 5, 5, 2, 3, 3, 1, 2, 3, 5, 6, 2, 5, 6, 1, 3, 2, 1, 1, 1, 6, 5, 5, 2, 2, 6, 4, 1, 4, 5, 1, 1, 1, 4, 5, 3, 3, 1, 3, 5, 4, 5, 6, 5, 4, 1, 5, 1, 2]
```

快速扫描这些结果表明 `Die` 类似乎在正常工作。我们看到了 1 和 6，因此我们知道返回的最小值和最大值是正确的，而且因为没有看到 0 或 7，我们知道所有结果都在正确的范围内。我们还看到了从 1 到 6 的每个数字，这表示所有可能的结果都有出现。接下来，我们将确定每个数字出现的具体次数。

### 分析结果

我们将通过统计每个数字出现的次数来分析投掷一个 D6 骰子的结果：

**die_visual.py**

```py
*--snip--*
# Make some rolls, and store results in a list.
results = []
❶ for roll_num in range(1000):
 result = die.roll()
 results.append(result)

# Analyze the results.
frequencies = []
❷ poss_results = range(1, die.num_sides+1)
for value in poss_results:
❸     frequency = results.count(value)
❹     frequencies.append(frequency)

print(frequencies)
```

因为我们不再打印结果，所以我们可以将模拟掷骰的次数增加到`1000` ❶。为了分析这些掷骰结果，我们创建了一个空列表`frequencies`来存储每个结果出现的次数。接着，我们生成所有可能的结果；在这个例子中，这些结果是从`1`到骰子面数的所有数字 ❷。我们遍历这些可能的值，统计每个数字在`results`中出现的次数 ❸，然后将此值添加到`frequencies`中 ❹。我们在生成可视化之前先打印出这个列表：

```py
[155, 167, 168, 170, 159, 181]
```

这些结果看起来是合理的：我们看到六个频率，每个频率对应掷 D6 骰子时的一个可能数字。我们还看到没有任何一个频率显著高于其他频率。现在让我们将这些结果可视化。

### 创建直方图

现在我们已经拥有所需的数据，我们可以仅通过几行代码使用 Plotly Express 生成可视化图表：

**die_visual.py**

```py
import plotly.express as px

from die import Die
*--snip--*

for value in poss_results:
 frequency = results.count(value)
 frequencies.append(frequency)

# Visualize the results.
fig = px.bar(x=poss_results, y=frequencies)
fig.show()
```

我们首先导入`plotly.express`模块，使用传统别名`px`。然后，我们使用`px.bar()`函数创建条形图。在该函数的最简单使用中，我们只需要传入一组*x-*值和一组*y-*值。这里的*x-*值是掷单个骰子可能的结果，而*y-*值是每个可能结果的频率。

最后一行调用了`fig.show()`，这告诉 Plotly 渲染结果图表为 HTML 文件，并在新浏览器标签页中打开该文件。结果如图 15-12 所示。

这只是一个非常简单的图表，当然也不完整。但这正是 Plotly Express 的设计初衷；你写几行代码，查看图表，确保它准确地表示了你的数据。如果你喜欢看到的效果，可以继续自定义图表的标签和样式。如果你想探索其他可能的图表类型，你可以在没有花费额外时间进行自定义的情况下尝试。现在就可以通过将`px.bar()`更改为`px.scatter()`或`px.line()`等来试试。你可以在[`plotly.com/python/plotly-express`](https://plotly.com/python/plotly-express)找到所有可用图表类型的完整列表。

该图表是动态且交互式的。如果你调整浏览器窗口的大小，图表会自动调整大小以适应可用空间。如果你将鼠标悬停在任何一个柱状条上，你将看到一个弹出窗口，突出显示与该柱状条相关的特定数据。

![](img/f15012.png)

图 15-12：Plotly Express 生成的初始图表

### 自定义图表

现在我们知道图表类型正确，数据也被准确地呈现出来，我们可以专注于为图表添加适当的标签和样式。

使用 Plotly 自定义图表的第一种方式是在初始生成图表时使用一些可选参数，这里是`px.bar()`。以下是如何添加一个总体标题和每个轴的标签：

**die_visual.py**

```py
*--snip--*
# Visualize the results.
❶ title = "Results of Rolling One D6 1,000 Times"
❷ labels = {'x': 'Result', 'y': 'Frequency of Result'}
fig = px.bar(x=poss_results, y=frequencies, title=title, labels=labels)
fig.show()
```

我们首先定义了我们想要的标题，这里赋值给 `title` ❶。为了定义轴标签，我们写了一个字典 ❷。字典中的键是我们想要自定义的标签，值是我们想使用的自定义标签。这里，我们为 *x* 轴赋予了 `Result` 标签，为 *y* 轴赋予了 `Frequency of Result` 标签。调用 `px.bar()` 时，现在包括了可选参数 `title` 和 `labels`。

现在，当绘图生成时，它包括了适当的标题和每个轴的标签，如图 15-13 所示。

![](img/f15013.png)

图 15-13：使用 Plotly 创建的简单柱状图

### 掷两个骰子

掷两个骰子会得到更大的数字，并且结果的分布也不同。让我们修改代码，创建两个 D6 骰子，模拟我们掷一对骰子的方式。每次掷骰子时，我们将两个数字（每个骰子上的一个数字）相加，并将和存储在 `results` 中。将 *die_visual.py* 保存为 *dice_visual.py*，并进行如下更改：

**dice_visual.py**

```py
import plotly.express as px

from die import Die

# Create two D6 dice.
die_1 = Die()
die_2 = Die()

# Make some rolls, and store results in a list.
results = []
for roll_num in range(1000):
❶     result = die_1.roll() + die_2.roll()
 results.append(result)

# Analyze the results.
frequencies = []
❷ max_result = die_1.num_sides + die_2.num_sides
❸ poss_results = range(2, max_result+1)
for value in poss_results:
 frequency = results.count(value)
 frequencies.append(frequency)

# Visualize the results.
title = "Results of Rolling Two D6 Dice 1,000 Times"
labels = {'x': 'Result', 'y': 'Frequency of Result'}
fig = px.bar(x=poss_results, y=frequencies, title=title, labels=labels)
fig.show()
```

在创建了两个 `Die` 实例后，我们掷骰子并计算每次掷骰子的和 ❶。最小的可能结果（2）是每个骰子上的最小数字之和。最大的可能结果（12）是每个骰子上的最大数字之和，我们将其赋值给 `max_result` ❷。变量 `max_result` 使得生成 `poss_results` 的代码更容易阅读 ❸。我们本可以写 `range(2, 13)`，但这只适用于两个 D6 骰子。在模拟现实世界情况时，最好编写能轻松模拟多种情况的代码。这段代码允许我们模拟掷一对具有任意面数的骰子。

运行此代码后，您应该会看到一个类似于图 15-14 的图表。

![](img/f15014.png)

图 15-14：模拟掷两个六面骰子 1,000 次的结果

该图展示了您掷一对 D6 骰子时，可能得到的结果的大致分布。正如您所见，掷出 2 或 12 的可能性最小，而掷出 7 的可能性最大。之所以这样，是因为掷出 7 有六种方式：1 和 6，2 和 5，3 和 4，4 和 3，5 和 2，6 和 1。

### 进一步自定义

我们刚刚生成的图表有一个问题需要解决。由于现在有 11 根柱子，*x* 轴的默认布局设置导致一些柱子没有标签。虽然默认设置适用于大多数可视化，但这个图表如果将所有柱子标注上，会看起来更好。

Plotly 有一个 `update_layout()` 方法，可以在图形创建后对其进行各种更新。以下是告诉 Plotly 给每个柱子添加标签的方法：

**dice_visual.py**

```py
*--snip--*
fig = px.bar(x=poss_results, y=frequencies, title=title, labels=labels)

# Further customize chart.
fig.update_layout(xaxis_dtick=1)

fig.show()
```

`update_layout()`方法作用于`fig`对象，该对象表示整个图表。在这里，我们使用了`xaxis_dtick`参数，该参数指定了*x*轴上刻度线之间的间距。我们将间距设置为`1`，这样每个条形图上都会有标签。当你再次运行*dice_visual.py*时，你应该能看到每个条形图上都有标签。

### 掷不同大小的骰子

让我们创建一个六面骰和一个十面骰，看看当我们掷它们 50,000 次时会发生什么：

**dice_visual_d6d10.py**

```py
import plotly.express as px

from die import Die

# Create a D6 and a D10.
die_1 = Die()
❶ die_2 = Die(10)

# Make some rolls, and store results in a list.
results = []
for roll_num in range(50_000):
 result = die_1.roll() + die_2.roll()
 results.append(result)

# Analyze the results.
*--snip--*

# Visualize the results.
❷ title = "Results of Rolling a D6 and a D10 50,000 Times"
labels = {'x': 'Result', 'y': 'Frequency of Result'}
*--snip--*
```

为了制作 D10，我们在创建第二个`Die`实例时传递参数`10` ❶，并将第一个循环更改为模拟 50,000 次掷骰，而不是 1,000 次。我们还更改了图表的标题 ❷。

图 15-15 显示了结果图表。与只有一个最可能结果的情况不同，这里有五个这样的结果。这是因为掷出最小值（1 和 1）和最大值（6 和 10）的方法只有一种，但较小的骰子限制了生成中间数字的方式。掷出 7、8、9、10 或 11 有六种方式，这些是最常见的结果，你有相同的概率掷出其中的任何一个。

![](img/f15015.png)

图 15-15：掷一个六面骰和一个十面骰 50,000 次的结果

我们使用 Plotly 模拟骰子掷出的能力，给了我们相当大的自由度来探索这一现象。在短短几分钟内，你可以使用各种骰子模拟大量的掷骰结果。

### 保存图形

当你得到一个喜欢的图形时，你可以通过浏览器将图表保存为 HTML 文件。但你也可以通过编程方式实现这一点。要将图表保存为 HTML 文件，只需将对`fig.show()`的调用替换为对`fig.write_html()`的调用：

```py
fig.write_html('dice_visual_d6d10.xhtml')
```

`write_html()`方法需要一个参数：要写入的文件名。如果你只提供文件名，文件将保存在与*.py*文件相同的目录中。你也可以通过`Path`对象调用`write_html()`，并将输出文件保存到系统中的任何位置。

## 总结

在本章中，你学会了生成数据集并创建数据的可视化。你使用 Matplotlib 创建了简单的图表，并使用散点图探索了随机游走。你还使用 Plotly 创建了一个直方图，并用它来探索掷不同大小骰子的结果。

使用代码生成自己的数据集是一种有趣且强大的方式，可以模拟和探索各种现实世界的情况。在接下来的数据可视化项目中，保持警觉，看看你是否能找到可以通过代码模拟的情景。观察新闻媒体中的可视化内容，看看你能否识别出那些是使用类似于你在这些项目中学习的方式生成的。

在第十六章中，你将从在线源下载数据，并继续使用 Matplotlib 和 Plotly 探索这些数据。
