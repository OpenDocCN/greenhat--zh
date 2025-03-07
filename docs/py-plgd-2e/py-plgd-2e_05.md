# [3](nsp-venkitachalam503045-0008.xhtml#rch03)

# 康威生命游戏

![](images/nsp-venkitachalam503045-circle-image.jpg)

你可以使用计算机通过创建一个系统的数学模型，编写程序来表示该模型，然后让模型随时间演化，来研究一个系统。有许多种计算机模拟方法，但我将重点介绍一个著名的模拟——康威的生命游戏，这是英国数学家约翰·康威的作品。生命游戏是一个*细胞自动机*的例子，细胞自动机是一个在网格上由彩色单元格组成的集合，按照一组定义邻居单元格状态的规则，通过多个时间步演化。

在这个项目中，你将创建一个 *N*×*N* 的单元格网格，并通过应用康威生命游戏的规则来模拟系统随时间的演变。你将在每个时间步显示游戏的状态，并提供将输出保存到文件的方法。你将设置系统的初始条件，可以选择随机分布或预设计模式。

该模拟由以下几个组件组成：

+   • 在一维或二维空间中定义的属性

+   • 用于在模拟中每一步改变该属性的数学规则

+   • 一种显示或捕获系统随时间演变状态的方法

康威生命游戏中的单元格可以是开启状态（ON）或关闭状态（OFF）。游戏从一个初始条件开始，其中每个单元格被分配一个这些状态之一。然后，数学规则决定每个单元格的状态如何随时间变化。康威生命游戏的惊人之处在于，仅仅通过四条简单的规则，系统就能演化出以令人难以置信复杂的方式表现的模式，几乎就像它们是活的一样。这些模式包括“滑行者”，它们在网格上滑动，“闪光灯”，它们闪烁开关，甚至是复制的模式。

当然，这个游戏的哲学意义也很重大，因为它暗示着复杂的结构可以从简单的规则中演变出来，而无需遵循任何预设的模式。

以下是本项目涉及的一些主要概念：

+   • 使用 `matplotlib imshow` 来表示二维数据网格

+   • 使用 `matplotlib` 进行动画制作

+   • 使用 `numpy` 数组

+   • 使用 `%` 操作符处理边界条件

+   • 设置随机分布的值

## [它是如何工作的](nsp-venkitachalam503045-0008.xhtml#rah0501)

由于生命游戏是基于九个方格的网格构建的，每个单元格有八个邻居单元格，如[图 3-1](nsp-venkitachalam503045-0015.xhtml#fig3-1)所示。在模拟中，给定的单元格 (*i*, *j*) 在网格 [*i*][*j*] 上访问，其中 *i* 和 *j* 分别是行和列的索引。给定单元格在某一时刻的值取决于其邻居在前一时间步的状态。

![](images/nsp-venkitachalam503045-f03001.jpg)

图 3-1：一个中心单元格 (*i*, *j*) 与八个邻居单元格

康威生命游戏有四条规则：

1.  1\. 如果一个单元格是开启的，并且有少于两个邻居是开启的，它将关闭。

1.  2\. 如果一个单元格是开启的，并且有两个或三个邻居也是开启的，它保持开启状态。

1.  3\. 如果一个单元格是开启的，并且有超过三个邻居是开启的，它将关闭。

1.  4\. 如果一个单元格是关闭的，并且恰好有三个邻居是开启的，它将开启。

这些规则旨在模拟一群有机体随着时间推移的基本生存状态：人口不足和人口过多通过将单元格关闭来“杀死”它们，当单元格的邻居少于两个或多于三个时，但当人口平衡时，单元格保持开启并通过将另一个单元格从关闭变为开启来繁殖。

我说每个单元格有八个邻居，但网格边缘的单元格呢？它们的邻居是哪些？为了回答这个问题，你需要考虑*边界条件*，即控制网格边缘或边界上单元格行为的规则。我将通过使用*环面边界条件*来解决这个问题，这意味着方形网格会像环面一样进行包装。如[图 3-2](nsp-venkitachalam503045-0015.xhtml#fig3-2)所示，首先将网格的水平边缘（A 和 B）弯曲连接形成一个圆柱体，然后将圆柱体的垂直边缘（C 和 D）连接起来形成一个环面。一旦形成环面，所有单元格都有邻居，因为整个空间没有边缘。

![](images/nsp-venkitachalam503045-f03002.jpg)

图 3-2：环面边界条件的概念性可视化

环面边界条件在二维模拟和游戏中非常常见。例如，游戏*吃豆人*使用了这些条件。如果你从屏幕顶部离开，你会从底部重新出现。如果你从屏幕左侧离开，你会从右侧重新出现。你将在“生命游戏”的模拟中使用相同的逻辑：例如，对于网格左上角的单元格，它上方的邻居将是左下角的单元格，而它左侧的邻居将是右上角的单元格。

以下是你将用于应用四条规则并运行模拟的算法描述：

1.  1\. 初始化网格中的单元格。

1.  2\. 在模拟的每一个时间步，对于网格中的每个单元格（*i*, *j*），执行以下操作：

    1.  a. 基于邻居的状态更新单元格（*i*, *j*）的值，并考虑边界条件。

    1.  b. 更新网格值的显示。

## [需求](nsp-venkitachalam503045-0008.xhtml#rah0502)

你将使用`numpy`数组和`matplotlib`库来显示模拟输出，并使用`matplotlib`的`animation`模块来更新模拟。

## [代码](nsp-venkitachalam503045-0008.xhtml#rah0503)

我们将逐步分析程序的关键部分，包括如何使用`numpy`和`matplotlib`表示模拟网格，如何设置模拟的初始条件，如何考虑环形边界条件，以及如何实现生命游戏规则。我们还将讨论程序的`main()`函数，它接收命令行参数并启动模拟。若要查看完整的项目代码，请跳至[“完整代码”](nsp-venkitachalam503045-0015.xhtml#ah0507)部分，位于[第56页](nsp-venkitachalam503045-0015.xhtml#p56)。你还可以从[https://github.com/mkvenkit/pp2e/tree/main/conway](https://github.com/mkvenkit/pp2e/tree/main/conway)下载代码。

### [表示网格](nsp-venkitachalam503045-0008.xhtml#rbh0501)

为了表示一个单元格是活的（ON）还是死的（OFF），你将使用值`255`表示ON，`0`表示OFF。你将使用`matplotlib`中的`imshow()`方法显示当前网格的状态，该方法将数字矩阵表示为图像。为了了解它是如何工作的，我们可以在Python解释器中尝试一个简单的例子。输入以下内容：

>>> `import numpy as np`

>>> `import matplotlib.pyplot as plt`

❶ >>> `x = np.array([[0, 0, 255], [255, 255, 0], [0, 255, 0]])`

❷ >>> `plt.imshow(x, interpolation='nearest')`

>>> `plt.show()`

你定义了一个形状为（3，3）的二维`numpy`数组❶，这意味着数组有三行三列。数组的每个元素都是一个整数值。然后你使用`plt.imshow()`方法将这些值的矩阵显示为图像❷，并传入插值选项`'nearest'`，以确保单元格的边缘清晰（否则它们会显得模糊）。[图3-3](nsp-venkitachalam503045-0015.xhtml#fig3-3)展示了这段代码的输出。

![](images/nsp-venkitachalam503045-f03003.jpg)

图3-3：显示一个值的网格

请注意，值为`0`（OFF）的方格会显示较暗的颜色，而值为`255`（ON）的方格则显示较亮的颜色。

### [设置初始条件](nsp-venkitachalam503045-0008.xhtml#rbh0502)

为了开始模拟，需要为二维网格中的每个单元格设置一个初始状态。你可以使用随机分布的ON和OFF单元格，看看会出现什么样的模式，或者可以添加一些特定的模式，看看它们如何演变。我们将同时讨论这两种方法。

对于随机初始状态，可以使用`numpy`中`random`模块的`choice()`方法。请在Python解释器中输入以下内容，看看它是如何工作的：

>>> `np.random.choice([0, 255], 4*4, p=[0.1, 0.9]).reshape(4, 4)`

输出将类似于以下内容：

array([[255, 255, 255, 255],

[255, 255, 255, 255]，

[255, 255, 255, 255]，

[255, 255, 255, 0]])

`np.random.choice()`从给定的列表`[0, 255]`中选择一个值，选择每个值的概率由参数`p=[0.1, 0.9]`给出。在这里，你请求0出现的概率为0.1（或10%），而255出现的概率为90%。(`p`中的两个值必须加起来等于1。）`choice()`方法创建一个一维数组，在这个例子中，数组包含16个值（由`4*4`指定）。你使用`reshape()`将其转换为一个具有四行四列的二维数组。

为了设置初始条件以匹配特定的图案，而不是仅仅填充一个随机值集合，首先使用`np.zeros()`初始化网格，使所有值都为零：

grid = np.zeros(N*N).reshape(N, N)

这会创建一个*N*×*N*的零数组。现在定义一个函数，在网格的特定行和列添加图案：

def addGlider(i, j, grid):

"""在(i, j)位置添加一个滑翔机图案"""

❶ glider = np.array([[0, 0, 255],

[255, 0, 255],

[0, 255, 255]])

❷ grid[i:i+3, j:j+3] = glider

你通过使用一个形状为(3, 3)的`numpy`数组定义滑翔机图案 ❶（一个在网格上稳定移动的观察到的图案）。然后，你使用`numpy`切片操作将`glider`数组复制到模拟的二维网格 ❷，并将图案的左上角放置在你指定的`i`和`j`坐标上。

现在，你可以通过调用你刚才定义的`addGlider()`函数将滑翔机图案添加到全零的网格中：

addGlider(1, 1, grid)

你指定坐标(1, 1)将滑翔机添加到接近网格左上角的地方，即(0, 0)。请注意，对于`grid[i, j]`，`i`从`0`开始，按从上到下的顺序排列，而`j`从`0`开始，按从左到右的顺序排列。

### [强制边界条件](nsp-venkitachalam503045-0008.xhtml#rbh0503)

现在我们可以考虑如何实现环形边界条件。首先，让我们看看在大小为*N*×*N*的网格的右边缘会发生什么。位于第*i*行末尾的单元格可以通过`grid[i, N-1]`访问。它右边的邻居是`grid[i, N]`，但根据环形边界条件，访问`grid[i, N]`时应该被替换为`grid[i, 0]`。以下是一种实现方法：

if j == N-1:

right = grid[i, 0]

else:

right = grid[i, j+1]

当然，你需要对网格的左侧、顶部和底部应用类似的边界条件，但这样做需要添加更多代码，因为网格的四个边缘都需要进行测试。实现边界条件的一个更加简洁的方法是使用Python的取余（`%`）运算符，下面的Python解释器演示了这一点：

>>> `N = 16`

>>> `i1 = 14`

>>> `i2 = 15`

>>> `(i1+1)%N`

15

>>> `(i2+1)%N`

0

正如你所看到的，`%`运算符给出了整数除法的余数。在这个例子中，`15%16`的结果是`15`，而`16%16`的结果是`0`。你可以通过重写网格访问代码，使用`%`运算符使值在右边缘处环绕：

right = grid[i, (j+1)%N]

现在，当一个单元格位于网格的边缘时（换句话说，当 `j = N-1` 时），用这种方法请求右侧的单元格会返回 `(j+1)%N`，这会将 `j` 重置为 `0`，使得网格的右侧与左侧连接。当你对网格底部做同样操作时，它会回绕到顶部：

bottom = grid[(i+1)%N, j]

### [实现规则](nsp-venkitachalam503045-0008.xhtml#rbh0504)

生命游戏的规则基于邻近单元格的 ON 或 OFF 状态数量。为了简化这些规则的应用，你可以直接计算邻居单元格的 ON 状态总数。因为 ON 对应的值为 255，所以只需要加总所有邻居的值，再除以 255 得到 ON 单元格的数量。以下是相关代码：

total = int((grid[i, (j-1)%N] + grid[i, (j+1)%N] +

grid[(i-1)%N, j] + grid[(i+1)%N, j] +

grid[(i-1)%N, (j-1)%N] + grid[(i-1)%N, (j+1)%N] +

grid[(i+1)%N, (j-1)%N] + grid[(i+1)%N, (j+1)%N])/255)

对于任何给定的单元格 (*i*, *j*)，你需要加总它的八个邻居的值，使用 `%` 运算符来处理环形边界条件。将结果除以 255 得到 ON 状态的邻居数量，这个值存储在变量 `total` 中。现在你可以使用 `total` 来应用生命游戏规则：

# 应用康威的规则

如果 grid[i, j] == ON:

❶ 如果 (total < 2) 或 (total > 3)：

newGrid[i, j] = OFF

否则：

❷ 如果 total == 3：

newGrid[i, j] = ON

任意处于 ON 状态的单元格，如果它周围有少于两个 ON 邻居或超过三个 ON 邻居，则会变为 OFF ❶。`else` 分支中的代码仅适用于 OFF 单元格：如果恰好有三个 ON 邻居，单元格会变为 ON ❷。这些更改会应用到 `newGrid` 中相应的单元格，`newGrid` 最初是 `grid` 的副本。一旦每个单元格都被评估和更新，`newGrid` 就包含了下一时刻显示的数据。你不能直接更改 `grid`，否则在尝试评估时单元格的状态会不断变化。

### [发送命令行参数到程序](nsp-venkitachalam503045-0008.xhtml#rbh0505)

现在你可以开始编写模拟的`main()`函数，它通过将命令行参数传递给程序来启动：

def main():

# 命令行参数在 sys.argv[1], sys.argv[2], ...

# sys.argv[0] 是脚本名称，可以忽略

# 解析参数

❶ parser = argparse.ArgumentParser(description="运行康威的生命游戏

模拟。")

# 添加参数

❷ parser.add_argument('--grid-size', dest='N', required=False)

❸ parser.add_argument('--interval', dest='interval', required=False)

❹ parser.add_argument('--glider', action='store_true', required=False)

args = parser.parse_args()

你创建了一个`argparse.ArgumentParser`对象，以将命令行选项添加到代码中❶，然后在接下来的行中添加了各种选项。❷的选项指定了模拟的网格大小*N*，❸的选项设置了动画更新间隔（以毫秒为单位）。你还创建了一个选项来启动一个滑行器模式的模拟❹。如果没有设置此选项，模拟将以随机的开/关值开始。

### [初始化模拟](nsp-venkitachalam503045-0008.xhtml#rbh0506)

在`main()`函数中继续，你会进入以下部分，这部分初始化了模拟：

# 设置网格大小

❶ N = 100

# 设置动画更新间隔

❷ updateInterval = 50

如果 args.interval:

updateInterval = int(args.interval)

# 声明网格

grid = np.array([])

# 检查是否指定了“滑行器”演示标志

❸ 如果 args.glider:

grid = np.zeros(N*N).reshape(N, N)

addGlider(1, 1, grid)

❹ 否则：

# 如果指定且有效，设置 N

如果 args.N 且 int(args.N) > 8:

N = int(args.N)

# 用随机的开/关填充网格——关的比开的多

grid = randomGrid(N)

这部分代码应用命令行中调用的任何参数，命令行选项解析完成后。首先，设置默认的网格大小为100×100个单元格❶和50毫秒的更新间隔❷，以防这些选项在命令行中没有设置。然后设置初始条件，默认是随机模式❹，或者使用滑行器模式❸。

最后，你设置动画：

# 设置动画

❶ fig, ax = plt.subplots()

img = ax.imshow(grid, interpolation='nearest')

❷ ani = animation.FuncAnimation(fig, update, fargs=(img, grid, N,  ),

interval=updateInterval,

save_count=50)

plt.show()

仍然在`main()`函数中，你配置了`matplotlib`绘图和动画参数❶。然后设置`animation.FuncAnimation()`，定期调用之前在程序中定义的`update()`函数，该函数根据康威生命游戏的规则，使用环形边界条件❷来更新网格。

## [运行生命游戏模拟](nsp-venkitachalam503045-0008.xhtml#rah0504)

现在运行代码：

$ `python conway.py`

这使用了模拟的默认参数：100×100个单元格的网格和50毫秒的更新间隔。随着模拟的进行，你将看到它如何在时间推移中创建并维持各种模式，如[图 3-4 (a) 和 (b)](nsp-venkitachalam503045-0015.xhtml#fig3-4)所示。

![](images/nsp-venkitachalam503045-f03004a.jpg)

(a)

![](images/nsp-venkitachalam503045-f03004b.jpg)

(b)

图 3-4：生命游戏进行中

[图 3-5](nsp-venkitachalam503045-0015.xhtml#fig3-5)展示了模拟中可能出现的一些模式。除了滑行器外，还可以看到三单元的闪烁器和静态模式，如块状或面包形态。

![](images/nsp-venkitachalam503045-f03005.jpg)

图 3-5：生命游戏中的一些模式

现在通过以下参数运行模拟，稍微改变一下设置：

$ `python conway.py --grid-size 32 --interval 500 --glider`

这会创建一个32×32的模拟网格，每500毫秒更新一次动画，并使用右下角显示的初始滑翔机模式，[图3-5](nsp-venkitachalam503045-0015.xhtml#fig3-5)。

## [摘要](nsp-venkitachalam503045-0008.xhtml#rah0505)

在这个项目中，你探索了康威的生命游戏。你学习了如何基于一些规则建立一个基本的计算机模拟，并如何使用`matplotlib`来可视化系统随时间的演变状态。

我的生命游戏实现强调简洁性而非性能。你可以通过多种方式加速生命游戏中的计算，并且有大量的研究探讨了如何做到这一点。你可以通过快速的互联网搜索找到很多相关研究。

## [实验！](nsp-venkitachalam503045-0008.xhtml#rah0506)

以下是一些进一步实验生命游戏的方式：

1.  1\. 编写一个`addGosperGun()`方法，将[图3-6](nsp-venkitachalam503045-0015.xhtml#fig3-6)中显示的模式添加到网格中。这个模式叫做*戈斯帕滑翔机枪*。运行模拟并观察枪械的行为。

    ![](images/nsp-venkitachalam503045-f03006.jpg)

    图3-6：戈斯帕滑翔机枪

1.  2\. 编写一个`readPattern()`方法，从文本文件中读取初始模式，并用它来设置模拟的初始条件。你可以使用Python方法，如`open`和`file.read`来实现。这里是输入文件的建议格式：

    8

    0 0 0 255...

    文件的第一行定义了*N*，接下来的内容是*N*×*N*的整数（0或255），并由空格分隔。这个探索将帮助你研究任何给定模式在“生命游戏”规则下的演变。添加一个`--pattern-file`命令行选项，用于在运行程序时使用此文件。

## [完整代码](nsp-venkitachalam503045-0008.xhtml#rah0507)

这里是生命游戏项目的完整代码：

"""

conway.py

一个简单的Python/matplotlib实现的康威生命游戏。

作者：Mahesh Venkitachalam

"""

import sys, argparse

import numpy as np

import matplotlib.pyplot as plt

import matplotlib.animation as animation

def randomGrid(N)

"""返回一个NxN随机值的网格"""

return np.random.choice([255, 0], N*N, p=[0.2, 0.8]).reshape(N, N)

def addGlider(i, j, grid):

"""在(i, j)位置添加一个滑翔机"""

glider = np.array([[0,  0, 255],

[255,  0, 255],

[0,  255, 255]])

grid[i:i+3, j:j+3] = glider

def update(frameNum, img, grid, N):

# 复制网格，因为我们需要8个邻居来进行计算

# 并且我们按行进行处理

newGrid = grid.copy()

for i in range(N):

for j in range(N):

# 计算8个邻居的和

# 使用环形边界条件 - x和y会循环

# 这样模拟就发生在一个环形表面上

total = int((grid[i, (j-1)%N] + grid[i, (j+1)%N] +

grid[(i-1)%N, j] + grid[(i+1)%N, j] +

grid[(i-1)%N, (j-1)%N] + grid[(i-1)%N, (j+1)%N] +

grid[(i+1)%N, (j-1)%N] + grid[(i+1)%N, (j+1)%N])/255)  

# 应用康威的规则  

if grid[i, j] == 255:  

if (total < 2) or (total > 3):  

newGrid[i, j] = 0  

else:  

if total == 3:  

newGrid[i, j] = 255

# 更新数据  

img.set_data(newGrid)  

grid[:] = newGrid[:]  

# 这里需要返回一个元组，因为这个回调  

# 函数需要返回一个可迭代对象  

return img,  

# main() 函数  

def main():  

# 命令行参数在 sys.argv[1], sys.argv[2]...  

# sys.argv[0] 是脚本名本身，可以忽略  

# 解析参数  

parser = argparse.ArgumentParser(description="运行康威的生命游戏")  

simulation.")  

# 添加参数  

parser.add_argument('--grid-size', dest='N', required=False)  

parser.add_argument('--interval', dest='interval', required=False)  

parser.add_argument('--glider', action='store_true', required=False)  

parser.add_argument('--gosper', action='store_true', required=False)  

args = parser.parse_args()  

# 设置网格大小  

N = 100  

# 设置动画更新间隔  

updateInterval = 50  

if args.interval:  

updateInterval = int(args.interval)  

# 声明网格  

grid = np.array([])  

# 检查是否指定了 "glider" 演示标志  

if args.glider:  

grid = np.zeros(N*N).reshape(N, N)  

addGlider(1, 1, grid)  

elif args.gosper:  

grid = np.zeros(N*N).reshape(N, N)  

addGosperGliderGun(10, 10, grid)  

else:  

# 设置 N（如果指定且有效）  

if args.N and int(args.N) > 8:  

N = int(args.N)  

# 使用随机的开/关值填充网格——关闭的比打开的多  

grid = randomGrid(N)  

# 设置动画  

fig, ax = plt.subplots()  

img = ax.imshow(grid, interpolation='nearest')  

ani = animation.FuncAnimation(fig, update, fargs=(img, grid, N, )  

frames = 10,  

interval=updateInterval)  

plt.show()  

# call main  

if __name__ == '__main__':  

main()  
