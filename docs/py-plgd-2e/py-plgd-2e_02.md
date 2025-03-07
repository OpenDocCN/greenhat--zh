# 1

# 科赫雪花

![](img/nsp-venkitachalam503045-circle-image.jpg)

我们将通过弄清楚如何绘制一个有趣的形状——*科赫雪花*来开始我们的 Python 冒险。它是瑞典数学家赫尔格·冯·科赫（Helge von Koch）于 1904 年发明的。科赫雪花是一种*分形*——一种随着你不断放大它，它会重复自身的图形。

分形的重复性质来自于*递归*，这是一种将某物定义为其自身的技巧。特别是，你通过*递归算法*绘制分形，这是一种重复的过程，其中一次重复的输出成为下次重复的输入。

在本章学习过程中，你将学到：

+   • 递归算法和函数的基础

+   • 如何使用`turtle`模块创建图形

+   • 绘制科赫雪花的递归算法

+   • 一些线性代数

## 它是如何工作的

图 1-1 显示了科赫雪花的样子。注意中间的大分支在左右两侧以较小的比例重复出现。类似地，中间的大分支本身由一些更小的分支构成，重复了更大的形状。这就是分形的重复自相似特性。

![](img/nsp-venkitachalam503045-f01001.jpg)

图 1-1：科赫雪花

如果你知道如何计算构成雪花基本形状的点，你就可以开发一个算法，通过递归执行相同的计算。通过这种方式，你将绘制出该形状的越来越小的版本，从而构建出分形。在本节中，我们将大致了解递归是如何工作的。接着，我们将考虑如何结合递归、一些线性代数和 Python 的`turtle`模块来绘制科赫雪花。

### 使用递归

为了感受递归是如何工作的，我们先来看一个简单的递归算法：计算一个数字的阶乘。一个数字的阶乘可以通过一个函数来定义，如下所示：

*f*(*N*) = 1 × 2 × 3 × . . . × (*N* − 1) × *N*

换句话说，*N*的阶乘就是 1 到*N*之间所有数字的乘积。你可以将其重写为：

*f*(*N*) = *N* × (*N* − 1) × . . . × 3 × 2 × 1

它可以再次被重写为：

*f*(*N*) = *N* × *f*(*N* − 1)

等等，你刚刚做了什么？你把*f*定义为它自身！这就是递归。调用*f*(*N*)最终会调用*f*(*N* − 1)，而*f*(*N* − 1)又会调用*f*(*N* − 2)，依此类推。那么，怎么知道什么时候停止呢？你需要将*f*(1)定义为 1，这就是递归的最深层次。

这是如何在 Python 中实现递归阶乘函数的：

```py
def factorial(N):
  ❶ if N == 1:
        return 1
    else:
      ❷ return N * factorial(N-1)

```

当*N*等于 1 时，通过简单地返回`1` ❶来处理这种情况，并通过再次调用`factorial()`实现递归调用 ❷，这次传入`N-1`。该函数将一直调用自身，直到*N*等于 1。其效果是，当函数返回时，它将计算出从 1 到*N*的所有数字的乘积。

通常，在尝试使用递归实现算法时，按照以下步骤操作：

1.  1\. 定义递归结束的基本情况。在我们的阶乘示例中，通过将*f*(1)定义为 1 来完成此操作。

1.  2\. 定义递归步骤。为此，你需要考虑如何将算法表达为递归过程。在某些算法中，函数可以从多个递归调用中调用，正如你将很快看到的那样。

递归是解决可以自然地分割成更小版本的问题的有用工具。阶乘算法就是这种分割的完美例子，正如你很快会看到的，绘制科赫雪花也是如此。尽管如此，递归并不总是解决问题的最有效方法。在某些情况下，重新用循环实现递归算法可能更合理。但事实是，递归算法通常比它们的循环对应物更紧凑和优雅。

### 计算雪花

现在让我们看看如何构造科赫雪花。图 1-2 展示了绘制雪花的基本图案。我将这种图案称为*雪花*。图案的基础是长度为*d*的线段![](img/nsp-venkitachalam503045-m01001.jpg)。该段被分为三等分部分，![](img/nsp-venkitachalam503045-m01002.jpg)、![](img/nsp-venkitachalam503045-m01003.jpg)和![](img/nsp-venkitachalam503045-m01004.jpg)，每个部分的长度为*r*。不直接连接点*P*[1]和*P*[3]，而是通过*P*[2]连接这些点，选择*P*[2]使得*P*[1]、*P*[2]和*P*[3]形成边长为*r*、高度为*h*的等边三角形。点*C*，*P*[1]和*P*[3]（以及*A*和*B*）的中点，正好位于*P*[2]的正下方，因此![](img/nsp-venkitachalam503045-m01005.jpg)和![](img/nsp-venkitachalam503045-m01006.jpg)是垂直的。

![](img/nsp-venkitachalam503045-f01002.jpg)

图 1-2：绘制科赫雪花的基本图案

一旦你理解了如何计算图 1-2 中显示的点，你就能递归地绘制越来越小的雪花，以重现科赫雪花。基本上，你的目标是这样的：给定点*A*和*B*，你想计算出点*P*[1]、*P*[2]和*P*[3]，并像图中显示的那样将它们连接起来。为了计算这些点，你需要使用一些线性代数，这是一门数学学科，让你可以计算距离，并根据*向量*确定点的坐标，向量是具有大小和方向的量。

这里有一个你将使用的简单线性代数公式。假设你在 3D 空间中有一个点 *A* 和一个单位向量 ![](img/nsp-venkitachalam503045-m01007.jpg)（单位向量是长度为 1 单位的向量）。沿着该单位向量，距离 *d* 的点 *B* 的坐标为：

*B* = *A* + *d* × ![](img/nsp-venkitachalam503045-m01007.jpg)

你可以通过一个例子轻松验证这一点。假设 *A* = (5, 0, 0) 且 ![](img/nsp-venkitachalam503045-m01007.jpg) = (0, 1, 0)。那么，点 *B* 在沿着 ![](img/nsp-venkitachalam503045-m01007.jpg) 方向上与 *A* 相距 10 个单位的坐标是多少呢？使用之前的公式，你得到：

*B* = (5, 0, 0) + 10 × (0, 1, 0) = (5, 10, 0)

换句话说，从 *A* 到 *B*，你沿着正 y 轴移动了 10 个单位。

这是另一个你将使用的结果——我们称之为*垂直向量技巧*。假设你有一个向量 ![](img/nsp-venkitachalam503045-m01008.jpg) = (*a*, *b*)。如果你有另一个向量 ![](img/nsp-venkitachalam503045-m01009.jpg)，它与 ![](img/nsp-venkitachalam503045-m01008.jpg) 垂直，那么它可以表示为 ![](img/nsp-venkitachalam503045-m01009.jpg) = (−*b*, *a*)。你可以通过对 ![](img/nsp-venkitachalam503045-m01008.jpg) 和 ![](img/nsp-venkitachalam503045-m01009.jpg) 进行点积来验证这个技巧的正确性。为了计算一对二维向量的点积，你需要将每个向量的第一个分量相乘，然后将每个向量的第二个分量相乘，最后将结果加在一起。在这种情况下，![](img/nsp-venkitachalam503045-m01008.jpg) 和 ![](img/nsp-venkitachalam503045-m01009.jpg) 的点积为：

![](img/nsp-venkitachalam503045-m01010.jpg) = (*a* × −*b*) + (*b* × *a*) = −*ab* + *ab* = 0

两个垂直向量的点积总是零，因此 ![](img/nsp-venkitachalam503045-m01009.jpg) 确实与 ![](img/nsp-venkitachalam503045-m01008.jpg) 垂直。

有了这个公式，让我们回到图 1-2 中的雪花。给定点 *A* 和 *B* 的坐标，如何计算 *P*[2] 的位置？你知道 *P*[2] 离点 *C* 的距离是 *h*，并且沿着单位向量 ![](img/nsp-venkitachalam503045-m01007.jpg) 方向。你的第一个线性代数公式告诉你：

*P*[2] = *C* + *h* × ![](img/nsp-venkitachalam503045-m01007.jpg)

现在让我们把这些变量转换成你已知的形式。首先，*C* 是线段 ![](img/nsp-venkitachalam503045-m01011.jpg) 的中点，因此 *C* = (*A* + *B*) / 2。接下来，*h* 是一个边长为 *r* 的等边三角形的高度。毕达哥拉斯定理告诉你：

![](img/nsp-venkitachalam503045-m01012.jpg)

在这种情况下，*r* 仅仅是从 *A* 到 *B* 的三分之一的距离。如果 *A* 的坐标是 (*x*[1], *y*[1])，*B* 的坐标是 (*x*[2], *y*[2])，你可以通过以下公式计算它们之间的距离：

![](img/nsp-venkitachalam503045-m01013.jpg)

然后只需将 *d* 除以 3 即可得到 *r*。

最后，你需要一种方法来表示 ![](img/nsp-venkitachalam503045-m01007.jpg)。你知道 ![](img/nsp-venkitachalam503045-m01007.jpg) 垂直于向量 ![](img/nsp-venkitachalam503045-m01014.jpg)，你可以通过将点 *A* 的坐标减去点 *B* 的坐标来表示 ![](img/nsp-venkitachalam503045-m01014.jpg)：

![](img/nsp-venkitachalam503045-m01014.jpg) = (*x*[2] − *x*[1], *y*[2] − *y*[1])

![](img/nsp-venkitachalam503045-m01014.jpg) 的大小由 *d* = ![](img/nsp-venkitachalam503045-m01015.jpg) 给出。现在你可以使用垂直向量技巧，将 ![](img/nsp-venkitachalam503045-m01007.jpg) 用 *A* 和 *B* 来表示：

![](img/nsp-venkitachalam503045-m01016.jpg)

接下来你需要计算 *P*[1] 和 *P*[3]。为此，你将使用线性代数中的另一个结果。假设你有一条线 ![](img/nsp-venkitachalam503045-m01011.jpg) 和这条线上的一点 *C*。设 *a* 为 *C* 到 *A* 的距离，*b* 为 *C* 到 *B* 的距离。点 *C* 可以表示为：

![](img/nsp-venkitachalam503045-m01017.jpg)

为了理解这个公式，想象一下如果 *C* 是 *A* 和 *B* 的中点，也就是说 *a* 和 *b* 会相等。在这种情况下，你可以直观地认为 *C* 应该等于 (*A* + *B*) / 2。将先前方程中的所有 *b* 替换为 *a*，你将得到：

![](img/nsp-venkitachalam503045-m01018.jpg)

现在有了这个新公式，你可以计算 *P*[1] 和 *P*[3]。这些点将线 ![](img/nsp-venkitachalam503045-m01011.jpg) 分成三等分，这意味着从 *P*[1] 到 *B* 的距离是从 *P*[1] 到 *A* 的两倍（*b* = 2*a*），而从 *P*[3] 到 *A* 的距离是从 *P*[3] 到 *B* 的两倍（*a* = 2*b*）。将这个代入公式，你就可以计算出这些点：

![](img/nsp-venkitachalam503045-m01019.jpg) 和 ![](img/nsp-venkitachalam503045-m01020.jpg)

现在你已经具备了绘制雪花分形第一层所需的一切。决定了 *A* 和 *B* 后，你就知道如何计算点 *P*[1]、*P*[2] 和 *P*[3]。但是分形的第二层会发生什么呢？你将第一层雪花中的每一条线段（图 1-2）替换为一小段雪花。结果如 图 1-3 所示。

![](img/nsp-venkitachalam503045-f01003.jpg)

图 1-3：Koch 雪花构造的第二步

请注意，图 1-2 中每个四个线段的变化， ![](img/nsp-venkitachalam503045-m01021.jpg), ![](img/nsp-venkitachalam503045-m01022.jpg), ![](img/nsp-venkitachalam503045-m01023.jpg), 和 ![](img/nsp-venkitachalam503045-m01024.jpg)，都成为了新雪花的基础。在科赫雪花程序中，你将能够使用每个线段的端点，例如，*A*和*P*[1]，作为新的*A*和*B*的值，并递归执行相同的计算，得到图 1-2 中的各个点。

在分形的每一层，你将再次细分雪花，绘制越来越小的自相似图形。这是算法的*递归步骤*，你将重复该步骤，直到达到*基本情况*。当 ![](img/nsp-venkitachalam503045-m01011.jpg) 小于某个阈值时——比如 10 像素——就应停止递归，直接绘制线段。

为了让最终的输出更加花哨，你可以绘制三个相连的雪花，作为分形的第一层。这将呈现出实际雪花的六角对称性。图 1-4 展示了开始绘制时的效果。

![](img/nsp-venkitachalam503045-f01004.jpg)

图 1-4：结合三个雪花

现在你知道了如何计算雪花的坐标，我们来看看如何在 Python 中使用这些坐标实际绘制图像。

### 使用 turtle 图形绘制

在本章中，你将使用 Python 的`turtle`模块来绘制雪花；这是一个简单的绘图程序，模仿了乌龟用尾巴在沙子里拖动，创造图案的概念。`turtle`模块包括一些方法，你可以使用这些方法来设置画笔的位置和颜色（即乌龟的尾巴），还有许多其他用于绘图的有用函数。

如你所见，绘制科赫雪花所需的仅仅是少数几个图形函数。实际上，从`turtle`的角度来看，绘制雪花几乎和绘制三角形一样简单。为了证明这一点，并让你感受`turtle`的工作方式，以下程序使用`turtle`绘制了一个三角形。输入代码，保存为*test_turtle.py*，并在 Python 中运行：

```py
❶ import turtle
def draw_triangle(x1, y1, x2, y2, x3, y3, t):
    # go to start of triangle
  ❷ t.up()
  ❸ t.setpos(x1, y1)
  ❹ t.down()
    t.setpos(x2, y2)
    t.setpos(x3, y3)
    t.setpos(x1, y1)
    t.up()
def main():
    print('testing turtle graphics...')
  ❺ t = turtle.Turtle()
  ❻ t.hideturtle()
  ❼ draw_triangle(-100, 0, 0, -173.2, 100, 0, t)
  ❽ turtle.mainloop()
# call main
if __name__ == '__main__':
    main()

```

首先导入 `turtle` 模块 ❶。接下来，定义 `draw_triangle()` 方法，方法的参数是三对 x 坐标和 y 坐标（三个三角形的角），以及 `t`，一个 `turtle` 对象。该方法首先调用 `up()` ❷。这告诉 Python 将画笔抬起；换句话说，把画笔从虚拟纸面上提起来，以便在移动海龟时不会画出任何东西。在开始绘制之前，你需要定位海龟的位置。`setpos()` 调用 ❸ 将海龟的位置设置为第一对 x 和 y 坐标。调用 `down()` ❹ 将画笔放下，在接下来的每次 `setpos()` 调用时，海龟移动到下一个坐标点，画出一条线。最终结果是绘制出一个三角形。

接下来，你声明一个 `main()` 函数来实际进行绘制。在该函数中，你创建了一个用于绘制的 `turtle` 对象 ❺，并隐藏了海龟 ❻。如果没有这个命令，你将看到一个小形状，表示海龟出现在正在绘制的线条前端。然后你调用 `draw_triangle()` 来绘制三角形 ❼，并传入所需的坐标作为参数。调用 `mainloop()` ❽ 可以在三角形绘制完成后保持 `tkinter` 窗口打开。（`tkinter` 是 Python 的默认图形界面库。）

图 1-5 显示了这个简单程序的输出。

![](img/nsp-venkitachalam503045-f01005.jpg)

图 1-5：一个简单的 `turtle` 程序的输出

现在你拥有了完成项目所需的一切。让我们开始绘制雪花吧！

## 需求

在这个项目中，你将使用 Python 的 `turtle` 模块来绘制雪花。

## 代码

要绘制 Koch 雪花，定义一个递归函数 `drawKochSF()`。该函数根据 图 1-2 中的 *A* 和 *B* 计算 *P*[1]、*P*[2] 和 *P*[3]，然后递归调用自身，进行相同的计算，逐渐处理更小的线段，直到达到最小的基准情况。然后使用 `turtle` 绘制雪花。有关完整的项目代码，请跳转到 “完整代码”，位于 第 16 页。代码也可以在本书的 GitHub 仓库中找到，链接为 [`github.com/mkvenkit/pp2e/blob/main/koch/koch.py`](https://github.com/mkvenkit/pp2e/blob/main/koch/koch.py)。

### 计算各个点

在 `drawKochSF()` 函数中，首先计算出绘制基本雪花图案所需的所有点的坐标，如 图 1-2 所示。

```py
def drawKochSF(x1, y1, x2, y2, t):
    d = math.sqrt((x1-x2)*(x1-x2) + (y1-y2)*(y1-y2))
    r = d/3.0
    h = r*math.sqrt(3)/2.0
    p3 = ((x1 + 2*x2)/3.0, (y1 + 2*y2)/3.0)
    p1 = ((2*x1 + x2)/3.0, (2*y1 + y2)/3.0)
    c = (0.5*(x1+x2), 0.5*(y1+y2))
    n = ((y1-y2)/d, (x2-x1)/d)
    p2 = (c[0]+h*n[0], c[1]+h*n[1])

```

你定义了`drawKochSF()`，并传入了线段![](img/nsp-venkitachalam503045-m01011.jpg)的 x 和 y 坐标，这构成了雪花的一个边，如图 1-4 所示。你还传入了`turtle`对象`t`，它用于实际绘制。然后，你计算出图 1-2 中显示的所有参数，如“计算雪花”部分中所述，从`d`开始，`d`是点*A*到*B*的距离。将`d`除以 3 得到`r`，这是构成雪花的一条边的长度。你使用`r`来计算`h`，雪花锥体的高度。

你将其余的参数计算为包含 x 和 y 坐标的元组。`p3`和`p1`元组描述了雪花锥形部分底部的两个点。点`c`是`p1`和`p3`的中点，`n`是垂直于线段![](img/nsp-venkitachalam503045-m01011.jpg)的单位向量。连同`h`一起，它们帮助你计算雪花锥体的顶点`p2`。

### 递归

`drawKochSF()`函数的下一部分使用递归将一级雪花分解为更小的版本。

```py
  ❶ if d > 10:
        # flake #1
      ❷ drawKochSF(x1, y1, p1[0], p1[1], t)
        # flake #2
        drawKochSF(p1[0], p1[1], p2[0], p2[1], t)
        # flake #3
        drawKochSF(p2[0], p2[1], p3[0], p3[1], t)
        # flake #4
        drawKochSF(p3[0], p3[1], x2, y2, t)

```

首先检查递归停止的条件❶。如果`d`，即线段的长度 ![](img/nsp-venkitachalam503045-m01011.jpg)，大于 10 像素，则继续递归。你通过再次调用`drawKochSF()`函数来实现这一点——四次！每次调用时，你会传入一组不同的参数，对应于构成雪花的四条线段的坐标，这些坐标是在函数开始时计算出来的。例如，在❷处，你会调用`drawKochSF()`来处理线段 ![](img/nsp-venkitachalam503045-m01021.jpg)。其他的函数调用则对应于线段 ![](img/nsp-venkitachalam503045-m01022.jpg)，![](img/nsp-venkitachalam503045-m01023.jpg) 和 ![](img/nsp-venkitachalam503045-m01024.jpg)。在这些递归调用中，你将基于新计算的点*A*和*B*的值，执行一组新的计算，如果`d`仍然大于 10 像素，你将继续进行四次递归调用`drawKochSF()`，以此类推。

### 绘制雪花

现在我们来看一下如果线段 ![](img/nsp-venkitachalam503045-m01011.jpg) 小于 10 像素时会发生什么。这是递归算法的基本情况。由于你已低于阈值，因此不再进行递归。相反，你将绘制构成单个雪花图案的四条线段，并从函数中返回。你使用了`up()`、`down()`和`setpos()`方法，这些方法来自`turtle`模块，你在“使用`turtle`图形绘制”部分已经学习过。

```py
    else:
        # draw cone
        t.up()
      ❶ t.setpos(p1[0], p1[1])
        t.down()
        t.setpos(p2[0], p2[1])
        t.setpos(p3[0], p3[1])
        # draw sides
        t.up()
      ❷ t.setpos(x1, y1)
        t.down()
        t.setpos(p1[0], p1[1])
        t.up()
      ❸ t.setpos(p3[0], p3[1])
        t.down()
        t.setpos(x2, y2)

```

首先，你绘制由点 `p1`、`p2` 和 `p3` 形成的圆锥形 ❶。然后你绘制线条 ![](img/nsp-venkitachalam503045-m01021.jpg) ❷ 和 ![](img/nsp-venkitachalam503045-m01024.jpg) ❸。由于你在函数开始时已经完成了所有必要的计算，绘制仅仅是将适当的坐标传递给 `setpos()` 方法的问题。

### 编写 main() 函数

`main()` 函数设置了一个 `turtle` 对象，并调用 `drawKochSF()`。

```py
def main():
    print('Drawing the Koch Snowflake...')
    t = turtle.Turtle()
    t.hideturtle()
    # draw
    try:
      ❶ drawKochSF(-100, 0, 100, 0, t)
      ❷ drawKochSF(0, -173.2, -100, 0, t)
      ❸ drawKochSF(100, 0, 0, -173.2, t)
  ❹ except:
         print("Exception, exiting.")
         exit(0)
    # wait for user to click on screen to exit
  ❺ turtle.Screen().exitonclick()

```

在 图 1-4 中，你看到你将如何绘制三个雪花，以获得六边形对称的最终输出。你通过三次调用 `drawKochSF()` 来实现这一点。用于点 *A* 和 *B* 的坐标为：第一个雪花 ❶ 的坐标是 `(-100, 0), (100, 0)`，第二个雪花 ❷ 的坐标是 `(0, -173.2), (-100, 0)`，第三个雪花 ❸ 的坐标是 `(100, 0), (0, -173.2)`。注意，这些坐标和你之前在 *test_turtle.py* 程序中绘制三角形时使用的坐标是一样的。试着自己算出这些坐标。（提示： ![](img/nsp-venkitachalam503045-m01025.jpg)）。

`drawKochSF()` 的调用被封装在一个 Python `try` 块中，以捕获绘图过程中发生的任何异常。例如，如果你在绘制过程中关闭窗口，就会抛出异常。你在 `except` 块 ❹ 中捕获它，打印一条消息并退出程序。如果你允许绘制完成，你将进入 `turtle.Screen().exitonclick()` ❺，该方法会等待你通过点击窗口中的任何地方来关闭窗口。

## 运行雪花代码

在终端中运行代码，如下所示。图 1-6 展示了输出结果。

```py
$ `python koch.py`

```

![](img/nsp-venkitachalam503045-f01006.jpg)

图 1-6：科赫雪花输出

这就是你美丽的雪花！

## 总结

在这一章中，你学习了递归函数和算法的基础知识。你还学习了如何使用 Python 的 `turtle` 模块绘制简单的图形。你将这些概念结合在一起，创造了一个有趣的分形图形——科赫雪花。

## 实验！

现在你已经完成了一个分形图形，我们来看看另一个有趣的图形，叫做 *Sierpi*ń*ski 三角形*，它以波兰数学家瓦茨瓦夫·谢尔平斯基（Wacław Sierpiński）命名。图 1-7 展示了它的样子。

![](img/nsp-venkitachalam503045-f01007.jpg)

图 1-7：Sierpiński 三角形

尝试使用`turtle`图形绘制谢尔宾斯基三角形。你可以使用递归算法，就像你绘制科赫雪花时那样。如果你查看图 1-7，你会看到大三角形被分成三个较小的三角形，中间有一个倒三角形的空洞。这三个较小的三角形每个又被分成三个三角形，且中间都有一个空洞，依此类推。这为你如何拆分递归提供了一个提示。

（此问题的解决方案在 GitHub 上的书籍仓库中，链接为[`github.com/mkvenkit/pp2e/blob/main/koch/koch.py`](https://github.com/mkvenkit/pp2e/blob/main/koch/koch.py)）

## 完整代码

这是该项目的完整代码列表：

```py
"""
koch.py
A program that draws the Koch snowflake.
Author: Mahesh Venkitachalam
"""
import turtle
import math
# draw the recursive Koch snowflake
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
        # flake #1
        drawKochSF(x1, y1, p1[0], p1[1], t)
        # flake #2
        drawKochSF(p1[0], p1[1], p2[0], p2[1], t)
        # flake #3
        drawKochSF(p2[0], p2[1], p3[0], p3[1], t)
        # flake #4
        drawKochSF(p3[0], p3[1], x2, y2, t)
    else:
        # draw cone
        t.up()
        t.setpos(p1[0], p1[1])
        t.down()
        t.setpos(p2[0], p2[1])
        t.setpos(p3[0], p3[1])
        # draw sides
        t.up()
        t.setpos(x1, y1)
        t.down()
        t.setpos(p1[0], p1[1])
        t.up()
        t.setpos(p3[0], p3[1])
        t.down()
        t.setpos(x2, y2)
# main() function
def main():
    print('Drawing the Koch Snowflake...')
    t = turtle.Turtle()
    t.hideturtle()
    # draw
    try:
        drawKochSF(-100, 0, 100, 0, t)
        drawKochSF(0, -173.2, -100, 0, t)
        drawKochSF(100, 0, 0, -173.2, t)
    except:
        print("Exception, exiting.")
        exit(0)
    # wait for user to click on screen to exit
    turtle.Screen().exitonclick()
# call main
if __name__ == '__main__':
    main()

```
