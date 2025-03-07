# [1](nsp-venkitachalam503045-0008.xhtml#rch01)

# 科赫雪花

![](images/nsp-venkitachalam503045-circle-image.jpg)

我们将通过找出如何绘制一个有趣的形状——*科赫雪花*——来开始我们的Python之旅。这个形状由瑞典数学家Helge von Koch在1904年发明。科赫雪花是一种*分形*——一种随着你放大它而重复自身的图形。

分形从*递归*中获得它们的重复特性，递归是一种用自身来定义某事的技术。具体来说，你通过*递归算法*绘制分形，这是一个重复的过程，其中一次重复的输出成为下一次重复的输入。

在你阅读本章内容时，你将学习到：

+   • 递归算法和函数的基础

+   • 如何使用`turtle`模块创建图形

+   • 使用递归算法绘制科赫雪花

+   • 一些线性代数

## [它是如何工作的](nsp-venkitachalam503045-0008.xhtml#rah0301)

[图1-1](nsp-venkitachalam503045-0012.xhtml#fig1-1)展示了科赫雪花的样子。注意，中间的大分支通过左侧和右侧的小分支以更小的比例重复。同样，中间的大分支本身由更小的分支组成，这些分支回响着更大形状的轮廓。这就是分形的重复性、自相似特性。

![](images/nsp-venkitachalam503045-f01001.jpg)

图1-1：科赫雪花

如果你知道如何计算形成雪花基本形状的点，那么你可以开发一个递归算法来执行相同的计算。这样，你将绘制越来越小的那个形状，逐步构建分形。在这一部分，我们将大致了解递归是如何工作的。然后，我们将考虑如何应用递归，以及一些线性代数和Python的`turtle`模块，来绘制科赫雪花。

### [使用递归](nsp-venkitachalam503045-0008.xhtml#rbh0301)

为了理解递归是如何工作的，让我们看一个简单的递归算法：计算一个数字的阶乘。一个数字的阶乘可以通过一个函数来定义，如下所示：

*f*(*N*) = 1 × 2 × 3 × . . . × (*N* − 1) × *N*

换句话说，*N*的阶乘只是从1到*N*的数字的乘积。你可以把它写成：

*f*(*N*) = *N* × (*N* − 1) × . . . × 3 × 2 × 1

这可以重新写成：

*f*(*N*) = *N* × *f*(*N* − 1)

等等，你刚才做了什么？你用*递归*定义了*f*！这就是递归。调用*f*(*N*)会最终调用*f*(*N* − 1)，然后会继续调用*f*(*N* − 2)，以此类推。但你怎么知道什么时候停止呢？嗯，你需要将*f*(1)定义为1，这将是递归的最深一步。

这是在Python中实现递归阶乘函数的方式：

def factorial(N):

❶ if N == 1:

return 1

否则：

❷ return N * factorial(N-1)

你通过简单地返回`1`来处理*N*等于 1 的情况❶，然后通过再次调用`factorial()`实现递归调用❷，这次传入`N-1`。该函数会一直调用自己，直到*N*等于 1。最终的效果是，当函数返回时，它将计算出从 1 到*N*所有数字的积。

一般来说，当你尝试使用递归实现一个算法时，请遵循以下步骤：

1.  1. 定义递归结束的基准情况。在我们的阶乘例子中，你通过将*f*(1)定义为 1 来实现这一点。

1.  2. 定义递归步骤。为此，你需要思考如何将算法表示为递归过程。在一些算法中，函数可能会有多个递归调用——正如你很快将看到的那样。

递归是一个有用的工具，适用于那些可以自然分解为更小版本的问题。阶乘算法就是这种分解的完美例子，正如你将很快看到的那样，绘制科赫雪花也是如此。也就是说，递归并不总是解决问题的最高效方法。在某些情况下，将递归算法重新实现为循环算法是有意义的。但事实仍然是，递归算法通常比其循环对等物更加简洁和优雅。

### 计算雪花

现在让我们来看看如何构造科赫雪花。[图 1-2](nsp-venkitachalam503045-0012.xhtml#fig1-2)展示了绘制雪花的基本模式。我将这个模式称为*雪花片*。图形的基础是长度为*d*的线段！[](images/nsp-venkitachalam503045-m01001.jpg)。该线段被分为三等份，分别是！[](images/nsp-venkitachalam503045-m01002.jpg)，！[](images/nsp-venkitachalam503045-m01003.jpg)，和！[](images/nsp-venkitachalam503045-m01004.jpg)，每一段的长度为*r*。与其直接连接点*P*[1]和*P*[3]，这两个点是通过*P*[2]连接的，选择*P*[2]的方式是使得*P*[1]、*P*[2]和*P*[3]形成一个边长为*r*、高为*h*的等边三角形。点*C*是*P*[1]和*P*[3]的中点（也就是*A*和*B*的中点），它位于*P*[2]正下方，使得！[](images/nsp-venkitachalam503045-m01005.jpg)和！[](images/nsp-venkitachalam503045-m01006.jpg)垂直。

![](images/nsp-venkitachalam503045-f01002.jpg)

图 1-2：绘制科赫雪花的基本模式

一旦你理解了如何计算[图 1-2](nsp-venkitachalam503045-0012.xhtml#fig1-2)中显示的点，你就能递归地绘制越来越小的雪花片，从而重现科赫雪花。基本上，你的目标是这样的：给定点*A*和*B*，你需要计算出点*P*[1]、*P*[2]和*P*[3]，并按照图示将它们连接起来。为了计算这些点，你需要使用一些线性代数知识，这是一门数学学科，可以帮助你计算距离并根据*向量*（具有大小和方向的量）确定点的坐标。

这是一个你将会用到的简单线性代数公式。假设你有一个三维空间中的点 *A* 和一个单位向量 ![](images/nsp-venkitachalam503045-m01007.jpg)（*单位向量* 是长度为 1 单位的向量）。在该单位向量方向上，距离 *A* 为 *d* 的点 *B* 由以下公式给出：

*B* = *A* + *d* × ![](images/nsp-venkitachalam503045-m01007.jpg)

你可以通过一个例子轻松验证这一点。以 *A* = (5, 0, 0) 和 ![](images/nsp-venkitachalam503045-m01007.jpg) = (0, 1, 0) 为例，计算一个点 *B*，它距离 *A* 沿着 ![](images/nsp-venkitachalam503045-m01007.jpg) 方向 10 个单位。使用之前的公式，你可以得到：

*B* = (5, 0, 0) + 10 × (0, 1, 0) = (5, 10, 0)

换句话说，要从 *A* 移动到 *B*，你需要沿着正 y 轴移动 10 个单位。

这里有另一个你会用到的结果——我们称之为*垂直向量技巧*。假设你有一个向量 ![](images/nsp-venkitachalam503045-m01008.jpg) = (*a*, *b*)。如果你有另一个与 ![](images/nsp-venkitachalam503045-m01008.jpg) 垂直的向量 ![](images/nsp-venkitachalam503045-m01009.jpg)，它可以表示为 ![](images/nsp-venkitachalam503045-m01009.jpg) = (−*b*, *a*)。你可以通过对 ![](images/nsp-venkitachalam503045-m01008.jpg) 和 ![](images/nsp-venkitachalam503045-m01009.jpg) 进行点积来验证这个技巧是否有效。要计算一对二维向量的点积，首先将每个向量的第一分量相乘，然后将每个向量的第二分量相乘，最后将结果相加。在这个例子中，![](images/nsp-venkitachalam503045-m01008.jpg) 和 ![](images/nsp-venkitachalam503045-m01009.jpg) 的点积为：

![](images/nsp-venkitachalam503045-m01010.jpg) = (*a* × −*b*) + (*b* × *a*) = −*ab* + *ab* = 0

两个垂直向量的点积总是为零，因此 ![](images/nsp-venkitachalam503045-m01009.jpg) 确实垂直于 ![](images/nsp-venkitachalam503045-m01008.jpg)。

有了这个，我们回到 [图 1-2](nsp-venkitachalam503045-0012.xhtml#fig1-2) 中的雪花。如何计算给定点 *A* 和 *B* 的坐标后，*P*[2] 的位置？你知道 *P*[2] 距离点 *C* 有 *h* 的距离，并沿单位向量 ![](images/nsp-venkitachalam503045-m01007.jpg) 方向移动。你的第一个线性代数公式告诉你：

*P*[2] = *C* + *h* × ![](images/nsp-venkitachalam503045-m01007.jpg)

现在让我们用你熟悉的术语来表示这些变量。首先，*C* 是线段 ![](images/nsp-venkitachalam503045-m01011.jpg) 的中点，因此 *C* = (*A* + *B*) / 2。接下来，*h* 是边长为 *r* 的等边三角形的高度。勾股定理告诉你：

![](images/nsp-venkitachalam503045-m01012.jpg)

在这种情况下，*r* 只是 *A* 到 *B* 之间距离的三分之一。如果 *A* 的坐标是 (*x*[1], *y*[1])，而 *B* 的坐标是 (*x*[2], *y*[2])，你可以通过以下方式计算它们之间的距离：

![](images/nsp-venkitachalam503045-m01013.jpg)

然后，只需将*d*除以3即可得到*r*。

最后，你需要找到表达![](images/nsp-venkitachalam503045-m01007.jpg)的方法。你知道![](images/nsp-venkitachalam503045-m01007.jpg)是垂直于向量![](images/nsp-venkitachalam503045-m01014.jpg)的，你可以通过从点*A*的坐标中减去点*B*的坐标来表示![](images/nsp-venkitachalam503045-m01014.jpg)：

![](images/nsp-venkitachalam503045-m01014.jpg) = (*x*[2] − *x*[1], *y*[2] − *y*[1])

![](images/nsp-venkitachalam503045-m01014.jpg)的大小由*d* = ![](images/nsp-venkitachalam503045-m01015.jpg)给出。现在，你可以使用垂直向量技巧将![](images/nsp-venkitachalam503045-m01007.jpg)用*A*和*B*表示出来：

![](images/nsp-venkitachalam503045-m01016.jpg)

接下来，你需要计算*P*[1]和*P*[3]。为此，你将使用线性代数中的另一个结果。假设你有一条线![](images/nsp-venkitachalam503045-m01011.jpg)和一点*C*在这条线上。令*a*为*C*到*A*的距离，*b*为*C*到*B*的距离。点*C*由以下公式给出：

![](images/nsp-venkitachalam503045-m01017.jpg)

要理解这个公式，考虑如果*C*是*A*和*B*的中点的情况，也就是说*a*和*b*应该是相等的。在这种情况下，你可以直观地推测*C*应该等于(*A* + *B*) / 2。将所有的*b*替换成*a*，你将得到：

![](images/nsp-venkitachalam503045-m01018.jpg)

有了这个新公式，你现在可以计算*P*[1]和*P*[3]。这些点将线![](images/nsp-venkitachalam503045-m01011.jpg)分成三等份，这意味着从*P*[1]到*B*的距离是从*P*[1]到*A*的两倍（*b* = 2*a*），而从*P*[3]到*A*的距离是从*P*[3]到*B*的两倍（*a* = 2*b*）。将这些代入公式，你就可以计算出这些点：

![](images/nsp-venkitachalam503045-m01019.jpg) 和 ![](images/nsp-venkitachalam503045-m01020.jpg)

现在，你已经拥有了绘制雪花分形的第一层所需的一切。一旦确定了*A*和*B*，你就知道如何计算点*P*[1]、*P*[2]和*P*[3]。但是在分形的第二层会发生什么呢？你取第一层雪花中的每个单独的线段（[图1-2](nsp-venkitachalam503045-0012.xhtml#fig1-2)），并用一个更小的雪花替代它。结果如[图1-3](nsp-venkitachalam503045-0012.xhtml#fig1-3)所示。

![](images/nsp-venkitachalam503045-f01003.jpg)

图1-3：Koch雪花构造的第二步

注意，[图 1-2](nsp-venkitachalam503045-0012.xhtml#fig1-2)、![](images/nsp-venkitachalam503045-m01021.jpg)、![](images/nsp-venkitachalam503045-m01022.jpg)、![](images/nsp-venkitachalam503045-m01023.jpg) 和 ![](images/nsp-venkitachalam503045-m01024.jpg) 中的四个线段，每个都成为了一个新雪花的基础。在 Koch 雪花程序中，你将能够使用每个线段的端点，例如 *A* 和 *P* [1]，作为新的 *A* 和 *B* 值，并递归地执行相同的计算，得出 [图 1-2](nsp-venkitachalam503045-0012.xhtml#fig1-2) 中的点。

在分形的每个层级中，你会再次细分雪花，绘制越来越小、相似的自相似图形。这就是算法的*递归步骤*，你会一直重复，直到达到*基本情况*。当 ![](images/nsp-venkitachalam503045-m01011.jpg) 小于某个阈值——比如 10 像素时，停止递归，直接绘制线段。

为了让最终输出更有趣，你可以绘制三个相连的雪花作为分形的第一层。这将为你提供一个实际雪花的六角对称性。[图 1-4](nsp-venkitachalam503045-0012.xhtml#fig1-4) 显示了起始绘图的样子。

![](images/nsp-venkitachalam503045-f01004.jpg)

图 1-4：组合三个雪花

现在你已经知道如何计算绘制雪花的坐标，让我们看看如何在 Python 中使用这些坐标来实际绘制图像。

### 使用 turtle 图形绘制

在本章中，你将使用 Python 的 `turtle` 模块绘制雪花；这是一种简单的绘图程序，灵感来源于乌龟拖动尾巴在沙地上留下图案的想法。`turtle` 模块包括一些方法，你可以用来设置画笔的位置和颜色（即乌龟的尾巴）以及许多其他有用的绘图功能。

如你所见，绘制 Koch 雪花只需要几个简单的图形函数。实际上，从 `turtle` 的角度来看，绘制雪花几乎和绘制三角形一样简单。为了证明这一点，并让你感受 `turtle` 的工作方式，以下程序使用 `turtle` 来绘制该三角形。输入代码，保存为 *test_turtle.py*，并在 Python 中运行：

❶ import turtle

def draw_triangle(x1, y1, x2, y2, x3, y3, t):

# 跳转到三角形的起始位置

❷ t.up()

❸ t.setpos(x1, y1)

❹ t.down()

t.setpos(x2, y2)

t.setpos(x3, y3)

t.setpos(x1, y1)

t.up()

def main():

print('正在测试乌龟图形...')

❺ t = turtle.Turtle()

❻ t.hideturtle()

❼ draw_triangle(-100, 0, 0, -173.2, 100, 0, t)

❽ turtle.mainloop()

# 调用 main

if __name__ == '__main__':

main()

首先导入`turtle`模块❶。接下来，定义`draw_triangle()`方法，其参数为三个坐标对（表示三角形的三个角）以及`t`，一个`turtle`对象。该方法开始时调用`up()`❷，这表示将画笔抬起；换句话说，提起画笔使其在移动乌龟时不会绘制任何东西。你希望在开始绘制之前定位乌龟。`setpos()`调用❸将乌龟的位置设置为第一个坐标对的x和y值。调用`down()`❹将画笔放下，之后每次调用`setpos()`时，当乌龟移动到下一个坐标时，就会绘制一条线。最终的结果是绘制一个三角形。

接下来，你需要声明一个`main()`函数来实际执行绘制操作。在这个函数中，你创建一个`turtle`对象用于绘制❺并隐藏乌龟❻。如果没有这条命令，你会看到一个表示乌龟的小形状出现在绘制的线条前面。然后，你调用`draw_triangle()`来绘制三角形❼，传入所需的坐标作为参数。调用`mainloop()`❽可以在三角形绘制完成后保持`tkinter`窗口开启。（`tkinter`是Python的默认GUI库。）

[图1-5](nsp-venkitachalam503045-0012.xhtml#fig1-5)显示了这个简单程序的输出。

![](images/nsp-venkitachalam503045-f01005.jpg)

图1-5：一个简单的`turtle`程序输出

现在你已经拥有了项目所需的所有内容。让我们开始绘制雪花吧！

## [需求](nsp-venkitachalam503045-0008.xhtml#rah0302)

在这个项目中，你将使用Python的`turtle`模块来绘制雪花。

## [代码](nsp-venkitachalam503045-0008.xhtml#rah0303)

要绘制科赫雪花，定义一个递归函数`drawKochSF()`。该函数根据[图1-2](nsp-venkitachalam503045-0012.xhtml#fig1-2)中的*A*和*B*计算*P*[1]、*P*[2]和*P*[3]，然后递归地调用自身，对越来越小的线段执行相同的计算，直到达到最小的基本情况。然后，它使用`turtle`绘制雪花。完整的项目代码可以跳到[“完整代码”](nsp-venkitachalam503045-0012.xhtml#ah0307)，在[第16页](nsp-venkitachalam503045-0012.xhtml#p16)中查看。代码也可以在书籍的GitHub仓库中找到，网址是[https://github.com/mkvenkit/pp2e/blob/main/koch/koch.py](https://github.com/mkvenkit/pp2e/blob/main/koch/koch.py)。

### 计算各点

在`drawKochSF()`函数中，首先计算绘制基本雪花图案所需的所有点的坐标，参见[图1-2](nsp-venkitachalam503045-0012.xhtml#fig1-2)。

def drawKochSF(x1, y1, x2, y2, t):

d = math.sqrt((x1-x2)*(x1-x2) + (y1-y2)*(y1-y2))

r = d/3.0

h = r*math.sqrt(3)/2.0

p3 = ((x1 + 2*x2)/3.0, (y1 + 2*y2)/3.0)

p1 = ((2*x1 + x2)/3.0, (2*y1 + y2)/3.0)

c = (0.5*(x1+x2), 0.5*(y1+y2))

n = ((y1-y2)/d, (x2-x1)/d)

p2 = (c[0]+h*n[0], c[1]+h*n[1])

你定义了 `drawKochSF()` 函数，传入了一个线段的起点和终点的 x 和 y 坐标 ![](images/nsp-venkitachalam503045-m01011.jpg)，该线段构成了雪花的一边，如 [图 1-4](nsp-venkitachalam503045-0012.xhtml#fig1-4) 所示。你还传入了 `turtle` 对象 `t`，用于实际绘制。然后，你计算了 [图 1-2](nsp-venkitachalam503045-0012.xhtml#fig1-2) 中所示的所有参数，正如在 [“计算雪花”](nsp-venkitachalam503045-0012.xhtml#bh0302) 部分中讨论的那样，从 `d` 开始，表示 *A* 到 *B* 的距离。将 `d` 除以 3 得到 `r`，这是构成雪花的四个线段的长度。你用 `r` 来求得 `h`，即雪花锥形中心的高度。

你将剩余的参数计算为包含 x 和 y 坐标的元组。`p3` 和 `p1` 元组描述了雪花锥形部分基座上的两个点。点 `c` 是 `p1` 和 `p3` 的中点，`n` 是垂直于线 ![](images/nsp-venkitachalam503045-m01011.jpg) 的单位向量。与 `h` 一起，它们帮助你计算 `p2`，即雪花锥形的顶点。

### 递归

`drawKochSF()` 函数的下一部分使用递归将一级雪花分解为越来越小的雪花版本。

❶ if d > 10:

# 雪花 #1

❷ drawKochSF(x1, y1, p1[0], p1[1], t)

# 雪花 #2

drawKochSF(p1[0], p1[1], p2[0], p2[1], t)

# 雪花 #3

drawKochSF(p2[0], p2[1], p3[0], p3[1], t)

# 雪花 #4

drawKochSF(p3[0], p3[1], x2, y2, t)

首先，你检查递归停止条件 ❶。如果 `d`，即线段 ![](images/nsp-venkitachalam503045-m01011.jpg) 的长度，大于 10 像素，则继续递归。你通过再次调用 `drawKochSF()` 函数来实现这一点——四次！每次调用时，你传入不同的参数集，表示构成雪花的四个线段的坐标，这些坐标是在函数开始时计算的。例如，在 ❷ 处，你调用 `drawKochSF()` 来绘制线段 ![](images/nsp-venkitachalam503045-m01021.jpg)。其他的函数调用分别用于线段 ![](images/nsp-venkitachalam503045-m01022.jpg)、![](images/nsp-venkitachalam503045-m01023.jpg) 和 ![](images/nsp-venkitachalam503045-m01024.jpg)。在每个递归调用中，你将根据点 *A* 和 *B* 的新值进行一组新的计算，如果 `d` 仍大于 10 像素，你将再次调用 `drawKochSF()`，四次，依此类推。

### 绘制雪花

现在，让我们看看当线段 ![](images/nsp-venkitachalam503045-m01011.jpg) 小于10像素时会发生什么。这是递归算法的基本情况。因为你已经低于阈值，所以你不会继续递归。相反，你会绘制出组成一个雪花图案的四个线段，并从函数中返回。你将使用`up()`、`down()`和`setpos()`方法，这些是在“使用`turtle`图形”部分中学习到的。

else:

# 绘制圆锥

t.up()

❶ t.setpos(p1[0], p1[1])

t.down()

t.setpos(p2[0], p2[1])

t.setpos(p3[0], p3[1])

# 绘制边

t.up()

❷ t.setpos(x1, y1)

t.down()

t.setpos(p1[0], p1[1])

t.up()

❸ t.setpos(p3[0], p3[1])

t.down()

t.setpos(x2, y2)

首先，你绘制由点`p1`、`p2`和`p3`形成的圆锥 ❶。然后绘制线段 ![](images/nsp-venkitachalam503045-m01021.jpg) ❷ 和 ![](images/nsp-venkitachalam503045-m01024.jpg) ❸。由于你已经在函数开始时执行了所有必要的计算，绘制过程实际上就是将适当的坐标传递给`setpos()`方法。

### [编写main()函数](nsp-venkitachalam503045-0008.xhtml#rbh0307)

`main()`函数设置一个`turtle`对象，并调用`drawKochSF()`。

def main():

print('绘制Koch雪花...')

t = turtle.Turtle()

t.hideturtle()

# 绘制

try:

❶ drawKochSF(-100, 0, 100, 0, t)

❷ drawKochSF(0, -173.2, -100, 0, t)

❸ drawKochSF(100, 0, 0, -173.2, t)

❹ except:

print("异常，程序退出。")

exit(0)

# 等待用户点击屏幕以退出

❺ turtle.Screen().exitonclick()

在[图1-4](nsp-venkitachalam503045-0012.xhtml#fig1-4)中，你看到如何绘制三个雪花，以获得六边对称的图像作为最终输出。你通过三次调用`drawKochSF()`来实现这一点。用于点*A*和*B*的坐标分别是：第一个雪花 ❶ 的坐标为`(-100, 0), (100, 0)`，第二个雪花 ❷ 的坐标为`(0, -173.2), (-100, 0)`，第三个雪花 ❸ 的坐标为`(100, 0), (0, -173.2)`。注意，这些坐标与你之前在*test_turtle.py*程序中绘制三角形时使用的坐标是一样的。尝试自己算出这些坐标。（提示：![](images/nsp-venkitachalam503045-m01025.jpg)）

`drawKochSF()`调用被包含在Python的`try`块中，以捕获绘制过程中发生的任何异常。例如，如果你在绘制过程中关闭窗口，就会抛出异常。你在`except`块 ❹ 中捕获它，在那里你打印一条消息并退出程序。如果你允许绘制完成，你将看到`turtle.Screen().exitonclick()` ❺，它会等待直到你点击窗口中的任意位置来关闭窗口。

## [运行雪花代码](nsp-venkitachalam503045-0008.xhtml#rah0304)

在终端中运行代码如下所示。[图1-6](nsp-venkitachalam503045-0012.xhtml#fig1-6)显示了输出结果。

$ `python koch.py`

![](images/nsp-venkitachalam503045-f01006.jpg)

图1-6：Koch雪花输出

这就是你美丽的雪花！

## [总结](nsp-venkitachalam503045-0008.xhtml#rah0305)

在本章中，你学习了递归函数和算法的基础知识。你还学会了如何使用 Python 的 `turtle` 模块绘制简单的图形。你将这些概念结合起来，创建了一个有趣的分形图形——科赫雪花。

## [实验！](nsp-venkitachalam503045-0008.xhtml#rah0306)

现在你已经完成了一个分形图形的绘制，我们来看另一个有趣的分形，叫做 *Sierpiński 三角形*，它以波兰数学家 Wacław Sierpiński 的名字命名。[图 1-7](nsp-venkitachalam503045-0012.xhtml#fig1-7) 展示了它的样子。

![](images/nsp-venkitachalam503045-f01007.jpg)

图 1-7: Sierpiński 三角形

尝试使用 `turtle` 图形绘制 Sierpiński 三角形。你可以使用类似绘制科赫雪花时的递归算法。如果你看看 [图 1-7](nsp-venkitachalam503045-0012.xhtml#fig1-7)，你会看到大三角形被分成了三个较小的三角形，中间有一个倒置的三角形孔。每个较小的三角形本身又被分成另三个三角形，并且每个中间也有一个孔，如此循环。这个结构可以给你一些关于如何拆分递归的提示。

(这个问题的解答可以在 GitHub 上找到，[https://github.com/mkvenkit/pp2e/blob/main/koch/koch.py](https://github.com/mkvenkit/pp2e/blob/main/koch/koch.py))

## [完整代码](nsp-venkitachalam503045-0008.xhtml#rah0307)

这是该项目的完整代码列表：

"""

koch.py

一个绘制科赫雪花的程序。

作者：Mahesh Venkitachalam

"""

import turtle

import math

# 绘制递归科赫雪花

def drawKochSF(x1, y1, x2, y2, t):

d = math.sqrt((x1-x2)*(x1-x2) + (y1-y2)*(y1-y2))

r = d/3.0

h = r*math.sqrt(3)/2.0

p3 = ((x1 + 2*x2)/3.0, (y1 + 2*y2)/3.0)

p1 = ((2*x1 + x2)/3.0, (2*y1 + y2)/3.0)

c = (0.5*(x1+x2), 0.5*(y1+y2))

n = ((y1-y2)/d, (x2-x1)/d)

p2 = (c[0]+h*n[0], c[1]+h*n[1])

if d > 10:

# 雪花 #1

drawKochSF(x1, y1, p1[0], p1[1], t)

# 雪花 #2

drawKochSF(p1[0], p1[1], p2[0], p2[1], t)

# 雪花 #3

drawKochSF(p2[0], p2[1], p3[0], p3[1], t)

# 雪花 #4

drawKochSF(p3[0], p3[1], x2, y2, t)

else:

# 绘制锥形

t.up()

t.setpos(p1[0], p1[1])

t.down()

t.setpos(p2[0], p2[1])

t.setpos(p3[0], p3[1])

# 绘制边

t.up()

t.setpos(x1, y1)

t.down()

t.setpos(p1[0], p1[1])

t.up()

t.setpos(p3[0], p3[1])

t.down()

t.setpos(x2, y2)

# main() 函数

def main():

print('绘制科赫雪花...')

t = turtle.Turtle()

t.hideturtle()

# 绘制

try:

drawKochSF(-100, 0, 100, 0, t)

drawKochSF(0, -173.2, -100, 0, t)

drawKochSF(100, 0, 0, -173.2, t)

except:

print("异常，退出。")

exit(0)

# 等待用户点击屏幕退出

turtle.Screen().exitonclick()

# 调用 main

if __name__ == '__main__':

main()
