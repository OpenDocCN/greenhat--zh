## 7

仿射变换

![Image](../images/common01.jpg)

如果我必须从本书中选择我最喜欢的主题，那就是仿射变换。仿射变换有某种奇异的美感，正如你在[第12章](ch12.xhtml#ch12)中看到我们为其制作动画时所见。

仿射变换对于二维图形应用至关重要；它们决定了如何在屏幕上平移、缩放和旋转所看到的内容。如果你使用过AutoCAD，你应该已经习惯了对图纸的某一部分进行缩放，这就是通过仿射变换实现的。每当你在Instagram上缩放和旋转图片时，也是通过仿射变换来完成的。掌握这一主题对于编写任何涉及图形的软件都是必不可少的，尤其是那些允许用户与图形互动的软件。

仿射变换背后的数学原理非常简单，但其概念却异常强大。在本章结束时，你将拥有一个表示这些变换的类，并具备应用这些变换到几何原始图形的能力。我们还将学习如何组合变换以计算复合变换，并了解一些有用的变换，例如围绕某个具体点缩放图形的变换。

### **仿射变换**

由于仿射变换应用于仿射空间，首先我们需要理解什么是仿射空间。你可以把*仿射空间*看作是一个向量空间，其中原点可以移动。在向量空间中使用的线性变换保持空间原点的位置，而在仿射空间中，由于我们不再关心固定的原点，平移是允许的。

那么，*仿射变换*是指两个仿射空间之间的一种映射，它保持点、直线和平面的不变。仿射变换后的点依然是点，直线仍然是直线，平面依然是平面。这些变换的一个有趣特性是，直线之间的平行性被保持。我们将在[第12章](ch12.xhtml#ch12)中通过动画展示仿射变换时看到这一点。在那个练习中，我们将看到原本平行的多边形的边在整个模拟过程中保持平行。

仿射变换类似于*线性变换*。唯一的区别是，后者保持原点不变；也就是说，点（0，0）不会移动。仿射变换则可以改变原点的位置。[图7-1](ch07.xhtml#ch7fig1)展示了线性变换和仿射变换。

![Image](../images/07fig01.jpg)

*图7-1：线性变换与仿射变换*

每一对坐标轴*x, y*在[图7-1](ch07.xhtml#ch7fig1)中显示了变换前空间的状态；每一对*x*^′, *y*^′则展示了变换后空间的状态。在线性变换的情况下，坐标原点*O*得以保留；而在仿射变换中，除了对坐标轴进行缩放和旋转外，还将原点*O*平移到了*O*^′。

给定一个点*P*，我们可以使用以下表达式定义仿射变换

![Image](../images/f00172-p1.jpg)

其中，*M*是一个线性变换，![Image](../images/tvictorit.jpg)是一个平移向量，而*P*^′是应用变换后的结果点。因此，仿射变换是线性变换*M*加上平移![Image](../images/tvictorit.jpg)。这个表达式可以按照[方程7.1](ch07.xhtml#ch07eqa01)所示写出所有项。

![Image](../images/07eqa01.jpg)

线性变换矩阵*M*包含以下项目

*s*[*x*]    在x方向上的缩放

*s*[*y*]    在y方向上的缩放

*sh*[*x*]    在x方向上的剪切变换

*sh*[*y*]    在y方向上的剪切变换

平移![Image](../images/tvictorit.jpg)的项为

*t*[*x*]    在x方向上的平移

*t*[*y*]    在y方向上的平移

[方程7.2](ch07.xhtml#ch07eqa02)展示了使用所谓的*增广矩阵*的等效形式。

![Image](../images/07eqa02.jpg)

这种版本通过扩展输入和输出向量的大小，并附加一个1来简化变换为一次矩阵乘法，这个1作为辅助值，在变换完成后可以丢弃。与前一种方法相比，这通常更受偏爱，因为它只需要一步，而不需要额外的加法。你可以观察到，在[方程7.1](ch07.xhtml#ch07eqa01)和[7.2](ch07.xhtml#ch07eqa02)中，结果坐标如[方程7.3](ch07.xhtml#ch07eqa03)所示。

![Image](../images/07eqa03.jpg)

来自[方程7.2](ch07.xhtml#ch07eqa02)矩阵中的每个值，在变换过程中都有不同的贡献。[图7-2](ch07.xhtml#ch7fig2)展示了每个组件产生的变换效果。因此，通用的仿射变换是这些单位变换的组合。

![Image](../images/07fig02.jpg)

*图7-2：仿射变换的组件*

有一种特殊的仿射变换，它将每个点映射到自身，即*恒等变换*。

![Image](../images/f0174-01.jpg)

如你所见，这是一个恒等矩阵：无论将这个矩阵乘以哪个点，结果都将保持不变。

#### ***仿射变换的例子***

让我们来看几个仿射变换的实际例子。在本节中，请放下你的电脑，拿出笔和纸。如果你能手工完成使用仿射变换转换空间的运算，那么编码实现这些变换就会变得很简单。

##### **例子1：缩放**

给定一个点 (2, 3)，在应用水平缩放 2 和垂直缩放 5 后，结果会是什么点？

在这种情况下，仿射变换矩阵中的项除了*s*[*x*] = 2 和 *s*[*y*] = 5 之外，其他都是零。将这些值代入[方程 7.2](ch07.xhtml#ch07eqa02)，我们得到如下结果：

![Image](../images/f0174-02.jpg)

因此，结果点为 (4, 15)。[图 7-3](ch07.xhtml#ch7fig3)展示了此变换对点的影响。

![Image](../images/07fig03.jpg)

*图 7-3：缩放变换的示例*

##### **示例 2：缩放和平移**

给定一个点 (2, 3)，在应用水平缩放 2、垂直缩放 5 和平移 ⟨10, 15⟩ 后，结果会是什么点？

这个案例与前一个案例具有相同的缩放值，外加一个位移向量。我们将这些值代入仿射变换方程：

![Image](../images/f0175-01.jpg)

这次，结果点为 (14, 30)。稍后我们将进一步探讨这一点，但值得注意的是，我们可以通过两个连续的仿射变换来实现相同的效果，第一个变换是缩放点，第二个变换是平移它：

![Image](../images/f0175-02.jpg)

请注意，变换是从右到左应用的。在前面的案例中，首先是缩放，然后是平移。如果交换变换的顺序，结果会不同，我们可以通过分别在两个方向上相乘变换矩阵并比较结果来验证这一点。这将得到我们的原始矩阵：

![Image](../images/f0175-03.jpg)

但是，交换顺序后得到的是：

![Image](../images/f0175-04.jpg)

[图 7-4](ch07.xhtml#ch7fig4)展示了先应用缩放然后应用平移的效果。

![Image](../images/07fig04.jpg)

*图 7-4：缩放加平移*

##### **示例 3：垂直反射**

反射可以通过使用具有负缩放值的仿射变换来实现。要将点 (2, 3) 在垂直方向上进行反射，可以使用 *s*[*y*] = –1：

![Image](../images/f0176-01.jpg)

这会得到原始点的垂直反射：(2,–3)。[图 7-5](ch07.xhtml#ch7fig5)表示了这个垂直反射。

![Image](../images/07fig05.jpg)

*图 7-5：垂直反射的示例*

##### **示例 4：水平剪切**

将水平剪切 *sh*[*x*] = 2 应用于一个矩形，矩形的左下角位于原点，宽度为 10 单位，高度为 5 单位，结果会怎样？

这次我们需要将相同的变换应用到矩形的四个顶点：(0, 0)、(10, 0)、(10, 5) 和 (0, 5)。仿射变换矩阵如下：

![Image](../images/f0176-02.jpg)

使用[方程 7.2](ch07.xhtml#ch07eqa02)并用这个矩阵来变换顶点，得到以下结果：(0, 0)、(10, 0)、(20, 5) 和 (10, 5)。绘制出结果矩形，应该像[图 7-6](ch07.xhtml#ch7fig6)那样。

![Image](../images/07fig06.jpg)

*图 7-6：剪切变换示例*

### **仿射变换类**

不再赘述，让我们创建一个新的类来表示仿射变换。我们希望使用类来管理变换的缩放、平移和剪切值，这些值将作为类的内部状态，而无需在每次调用变换方法时都传递这些参数。如果我们使用函数来变换几何图形，那我们就需要将这些值作为参数传递给每个函数，但这将需要大量的参数。

在 *geom2d* 包中，创建一个名为 *affine_transf.py* 的新文件，并输入 [列表 7-1](ch07.xhtml#ch7lis1) 中的代码。

```py
class AffineTransform:
    def __init__(self, sx=1, sy=1, tx=0, ty=0, shx=0, shy=0):
        self.sx = sx
        self.sy = sy
        self.tx = tx
        self.ty = ty
        self.shx = shx
        self.shy = shy
```

*列表 7-1：AffineTransform 类*

仿射变换存储了缩放值 *s*[*x*] 和 *s*[*y*]、平移值 *t*[*x*] 和 *t*[*y*]，以及剪切值 *sh*[*x*] 和 *sh*[*y*]。所有值的默认值为零，除非缩放值，它们初始化为一，以防在初始化器中省略。这是为了方便，因为我们会创建许多变换，其中剪切或平移值为零。

有了这些值，我们已经可以实现一个方法，借助 [公式 7.3](ch07.xhtml#ch07eqa03) 将变换应用于一个点。在 *geom2d* 包中输入 [列表 7-2](ch07.xhtml#ch7lis2) 中的代码。

```py
from geom2d.point import Point

class AffineTransform:
   --snip--

   def apply_to_point(self, point: Point):
       return Point(
           (self.sx * point.x) + (self.shx * point.y) + self.tx,
           (self.shy * point.x) + (self.sy * point.y) + self.ty
       )
```

*列表 7-2：将仿射变换应用于一个点*

要将仿射变换应用于一个点，我们创建一个新的 Point，其中投影值是根据 [公式 7.3](ch07.xhtml#ch07eqa03) 计算的。让我们使用几种不同的变换来测试此方法。

#### ***测试点的变换***

在 *geom2d* 包中创建一个名为 *affine_transf_test.py* 的新文件，并输入 [列表 7-3](ch07.xhtml#ch7lis3) 中的代码。

```py
import unittest

from geom2d.point import Point
from geom2d.affine_transf import AffineTransform

class TestAffineTransform(unittest.TestCase):
    point = Point(2, 3)
    scale = AffineTransform(2, 5)
    trans = AffineTransform(1, 1, 10, 15)
    shear = AffineTransform(1, 1, 0, 0, 3, 4)

 ➊ def test_scale_point(self):
        expected = Point(4, 15)
        actual = self.scale.apply_to_point(self.point)
        self.assertEqual(expected, actual)
 ➋ def test_translate_point(self):
        expected = Point(12, 18)
        actual = self.trans.apply_to_point(self.point)
        self.assertEqual(expected, actual)
 ➌ def test_shear_point(self):
        expected = Point(11, 11)
        actual = self.shear.apply_to_point(self.point)
        self.assertEqual(expected, actual)
```

*列表 7-3：测试仿射变换应用*

测试文件包含 TestAffineTransform 类，继承自 unittest.TestCase，和往常一样。在类内部，我们定义一个在所有测试中使用的点，以及三个仿射变换，即：

scale    缩放变换

trans    平移变换

shear    剪切变换

然后我们有第一个测试，确保缩放正确应用于点 ➊。第二个测试应用平移于点，并断言结果符合预期 ➋。第三个测试对剪切变换做同样的操作 ➌。运行测试。你可以从 shell 执行：

```py
$ python3 -m unittest geom2d/affine_transf_test.py
```

这将产生如下结果：

```py
Ran 3 tests in 0.001s

OK
```

太好了！现在我们确信仿射变换正确应用于点，我们来将逻辑扩展到其他更复杂的图形。

#### ***变换线段和多边形***

我们可以利用实现的点变换方法来变换任何形状，只要它是通过点或向量定义的。下一步是实现线段的变换，因此在 apply_to _point 方法之后，输入 [列表 7-4](ch07.xhtml#ch7lis4) 中的方法。

```py
from geom2d.segment import Segment
from geom2d.point import Point

class AffineTransform:
   --snip--

   def apply_to_segment(self, segment: Segment):
       return Segment(
           self.apply_to_point(segment.start),
           self.apply_to_point(segment.end)
       )
```

*列表 7-4：将仿射变换应用于线段*

这很简单，不是吗？要变换一个线段，我们只需使用前面的方法将两个端点都进行变换，创建一个新的线段。我们可以将类似的逻辑应用于多边形（见[列表 7-5](ch07.xhtml#ch7lis5)）。

```py
from geom2d.polygon import Polygon
from geom2d.segment import Segment
from geom2d.point import Point

class AffineTransform:
   --snip--

   def apply_to_polygon(self, polygon: Polygon):
       return Polygon(
           [self.apply_to_point(v) for v in polygon.vertices]
       )
```

*列表 7-5：将仿射变换应用于多边形*

在这种情况下，我们返回一个新的多边形，其中所有的顶点都已经被变换。那矩形和圆形呢？思路类似，但有一个警告：在缩放、错切和旋转这些原始图形后，结果可能不再是矩形或圆形。这就是为什么在前一章中，我们为矩形和圆形提供了一个 `to_polygon` 方法，用于为这些原始图形创建一个通用的多边形表示。代码因此非常简单。请从[列表 7-6](ch07.xhtml#ch7lis6)输入代码：

```py
from geom2d.rect import Rect
from geom2d.circle import Circle
from geom2d.polygon import Polygon
from geom2d.segment import Segment
from geom2d.point import Point

class AffineTransform:
   --snip--

   def apply_to_rect(self, rect: Rect):
       return self.apply_to_polygon(
           rect.to_polygon()
       )

   def apply_to_circle(self, circle: Circle, divisions=30):
       return self.apply_to_polygon(
           circle.to_polygon(divisions)
       )
```

*列表 7-6：将仿射变换应用于矩形和圆形*

该过程包括获取矩形或圆形的多边形表示，并将其余过程委托给 `apply_to_polygon`。对于圆形，需要选择分割数，默认值为 30。即使应用的仿射变换是恒等变换（不会改变几何形状），这两个方法也会返回一个 `Polygon` 实例。一旦矩形或圆形经过仿射变换，它就会变成一个通用的多边形，无论是什么变换。

由于空间原因，我们在这里不会这么做，但你可以随时为这三个新方法添加单元测试。

#### ***连接变换***

仿射变换的一个有趣性质是，任何复杂的变换都可以表示为一系列简单变换的组合。事实上，当你使用像 Sketch 或 Photoshop 这样的 2D 图形应用程序时，每一次画布的缩放或平移，都是当前变换与新的仿射变换的组合或连接，这决定了你在特定时刻在屏幕上看到的投影。

给定两个仿射变换 [*T*[1]] 和 [*T*[2]] 以及输入点 *P*，将 [*T*[1]] 应用到该点的结果如下：

*P′* = [*T*[1]]*P*

然后，将第二个变换 [*T*[2]] 应用到先前的结果，我们得到：

*P*^″ = [*T*[2]]*P*^′

如果我们将第一个表达式中的 *P*^′ 代入第二个表达式，就得到应用这两次变换到输入点 *P* 的结果（[方程 7.4](ch07.xhtml#ch07eqa04)），

![Image](../images/07eqa04.jpg)

其中，[*T*[*r*]] 是等同于先应用[*T*[1]]再应用[*T*[2]]的仿射变换。注意，如果从左到右阅读，原始变换的顺序是如何反过来的？

[*T*[r]] = [*T*[2]][*T*[1]]

在前面的公式中，从左到右[*T*[2]]首先出现，但应用[*T*[*r*]]的效果相当于先应用[*T*[1]]，再应用[*T*[2]]。我们需要小心顺序，因为矩阵乘法是不交换的。如果我们交换操作数的顺序，我们将得到一个不同的变换，这在之前的一个练习中已经证明过了。结果变换随后用矩阵的乘积表示（参见[方程 7.5](ch07.xhtml#ch07eqa05)）。

![Image](../images/07eqa05.jpg)

让我们为AffineTransform类提供一个方法，用[方程 7.5](ch07.xhtml#ch07eqa05)来串联仿射变换。我们将这个方法命名为then()，接受参数self和other。第一个参数self是变换[*T*[1]]，而other是[*T*[2]]。在*affine_transf.py*中，类的末尾输入[列表 7-7](ch07.xhtml#ch7lis7)中的代码。

```py
class AffineTransform:
   --snip--

   def then(self, other):
       return AffineTransform(
           sx=other.sx * self.sx + other.shx * self.shy,
           sy=other.shy * self.shx + other.sy * self.sy,
           tx=other.sx * self.tx + other.shx * self.ty + other.tx,
           ty=other.shy * self.tx + other.sy * self.ty + other.ty,
           shx=other.sx * self.shx + other.shx * self.sy,
           shy=other.shy * self.sx + other.sy * self.shy
       )
```

*列表 7-7：串联变换的方法*

然后选择这个名称是为了确保完全清楚地表明self在other之前应用（即方法的参数）。

由于这是一个非常重要的方法，我们希望它能通过单元测试覆盖；这意味着我们需要一种方法来判断两个给定的仿射变换是否相等。让我们在AffineTransform中实现特殊的__eq__方法（参见[列表 7-8](ch07.xhtml#ch7lis8)）。

```py
from geom2d.nums import are_close_enough
from geom2d.rect import Rect
from geom2d.circle import Circle
from geom2d.polygon import Polygon
from geom2d.segment import Segment
from geom2d.point import Point

class AffineTransform:
    --snip--

    def __eq__(self, other):
        if self is other:
            return True

        if not isinstance(other, AffineTransform):
            return False

        return are_close_enough(self.sx, other.sx) \
               and are_close_enough(self.sy, other.sy) \
               and are_close_enough(self.tx, other.tx) \
               and are_close_enough(self.ty, other.ty) \
               and are_close_enough(self.shx, other.shx) \
               and are_close_enough(self.shy, other.shy)
```

*列表 7-8：检查仿射变换相等性*

##### **测试变换的串联**

现在让我们在*affine_transf_test.py*中输入两个新测试；它们都列在[列表 7-9](ch07.xhtml#ch7lis9)中。

```py
class TestAffineTransform(unittest.TestCase):
   --snip--

   def test_concatenate_scale_then_translate(self):
       expected = AffineTransform(2, 5, 10, 15)
       actual = self.scale.then(self.trans)
       self.assertEqual(expected, actual)

   def test_concatenate_translate_then_scale(self):
       expected = AffineTransform(2, 5, 20, 75)
       actual = self.trans.then(self.scale)
       self.assertEqual(expected, actual)
```

*列表 7-9：测试仿射变换串联*

正如你可能已经意识到的，这两个测试重复了我们在本章开头的某个练习中手动进行的操作。运行它们，以确保你正确实现了这些操作。

```py
$ python3 -m unittest geom2d/affine_transf_test.py
```

由于self和other之间有很多加法和乘法运算，很容易把代码写错。如果测试未通过，那就意味着它们通过指出代码中的问题来完成它们的工作。返回你的实现并逐行确保你一切都做对了。

#### ***反转仿射变换***

为了撤销一个变换或应用已知变换[*T*]的逆，我们希望能够计算一个变换[*T*[*I*]]，使得

[*T*][*T*[I]] = [*T*[I]][*T*] = [*I*]

其中[*I*]是3x3的单位矩阵：

![Image](../images/f00184-p1.jpg)

这些变换对[*T*]和[*T*[*I*]]的一个有趣属性是它们相互抵消。例如，以下是将这些变换按顺序（无论顺序如何）作用于一个点*P*的结果：

[*T*[I]]([*T*]*P*) = ([*T*[I]][*T*])*P* = [*I*]*P* = *P*

逆仿射变换之所以有趣的另一个原因是，它将屏幕上的点映射回我们的“模型空间”，即定义我们模型的仿射空间。直接变换用于计算几何图形如何投影到屏幕上，也就是说，模型中的每个点应该绘制到哪里——但反过来呢？要知道屏幕上某个给定点在模型中的位置，需要逆变换，即将“屏幕空间”转换为模型空间的变换。这在某些情况下很有用，例如当试图弄清楚屏幕上用户的鼠标指针是否映射到模型中的某个可选项时。

看一下[图 7-7](ch07.xhtml#ch7fig7)。这是我们的模型空间，其中只定义了一个三角形。为了将模型绘制到用户的屏幕上，我们必须应用一个仿射变换，将模型空间中的每个点投影到屏幕空间。现在，假设用户的鼠标位于屏幕上的*P*^′点，我们想知道该点是否位于我们的三角形内部。由于三角形是定义在模型空间中的几何图形，我们希望对屏幕上的该点应用逆变换：即将屏幕空间转换回模型空间的变换。回想一下，为了将我们的模型几何图形投影到屏幕上，我们应用了直接的仿射变换，因此要将几何图形映射回原始的模型空间，需要应用该变换的逆变换。通过将该点映射到我们的模型空间(*P*)，我们可以进行计算，以确定*P*是否在三角形内部。

![图片](../images/07fig07.jpg)

*图 7-7：模型和屏幕空间*

你可以尝试自己计算逆仿射变换矩阵，这是一个很好的练习，但手动求逆矩阵是一个繁琐的任务，因此[方程 7.6](ch07.xhtml#ch07eqa06)展示了结果。

![图片](../images/07eqa06.jpg)

使用[方程 7.6](ch07.xhtml#ch07eqa06)中的变换，计算逆变换只需要几行代码。在AffineTransform类中，之后输入[清单 7-10](ch07.xhtml#ch7lis10)中的代码。

```py
class AffineTransform:
    --snip--

   def inverse(self):
       denom = self.sx * self.sy - self.shx * self.shy
       return AffineTransform(
           sx=self.sy / denom,
           sy=self.sx / denom,
           tx=(self.ty * self.shx - self.sy * self.tx) / denom,
           ty=(self.tx * self.shy - self.sx * self.ty) / denom,
           shx=-self.shx / denom,
           shy=-self.shy / denom
       )
```

*清单 7-10：逆仿射变换*

让我们再添加一个测试，确保逆变换能够正确计算。在*affine_transf_test.py*中，向TestAffineTransform类中添加一个新方法，测试内容参见[清单 7-11](ch07.xhtml#ch7lis11)。

```py
class TestAffineTransform(unittest.TestCase):
   --snip--

   def test_inverse(self):
       transf = AffineTransform(1, 2, 3, 4, 5, 6)
       expected = AffineTransform()
       actual = transf.then(transf.inverse())
       self.assertEqual(expected, actual)
```

*清单 7-11：测试逆仿射变换*

在这个测试中，我们创建了一个新的仿射变换transf，并将所有值设置为非零值。然后，我们将连接transf及其逆变换的结果存储在实际结果中，如果你记得的话，如果逆变换正确构造，结果应该是单位矩阵。最后，我们将得到的结果与实际的单位矩阵进行比较。运行文件中的所有测试，确保它们都能成功通过。

```py
$ python3 -m unittest geom2d/affine_transf_test.py
```

让我们试一个例子。我们将对一个点应用平移，然后对得到的点应用反向平移，最终应该得到原始点。在Python shell中，写下以下代码：

```py
>>> from geom2d.affine_transf import AffineTransform
>>> from geom2d.point import Point
>>> trans = AffineTransform(tx=10, ty=20)
>>> original = Point(5, 7)
```

我们知道，如果我们将⟨10, 20⟩的平移应用到点（5, 7），则得到的点应该是（15, 27）。让我们来验证一下。

```py
>>> translated = trans.apply_to_point(original)
>>> str(translated)
'(15, 27)'
```

使用str函数，我们可以获得平移后的点的字符串表示。现在，让我们对这个点应用反向平移变换。

```py
>>> inverse = trans.inverse().apply_to_point(translated)
>>> str(inverse)
'(5.0, 7.0)'
```

对平移后的点应用反向变换，得到原始点，正如预期的那样。

#### ***缩放***

每当你使用AutoCAD或Illustrator等图形应用程序进行缩放时，都会对几何模型应用一个缩放仿射变换，使得模型在屏幕上显示的大小与真实大小不同。建筑师绘制的建筑蓝图可能有几百米高，而这些蓝图需要适应几英寸宽的笔记本电脑屏幕。在计算机的内存中存储着具有真实尺寸的几何模型，但为了在屏幕上绘制它，需要应用一个缩放：一个缩放仿射变换。

为了获得这种仿射变换的视觉直觉，让我们看一下[图 7-8](ch07.xhtml#ch7fig8)。给定一个点*P*，我们可以想象有一个从原点出发的向量![Image](../images/vvictorit.jpg)，其尖端在*P*上。对点*P*应用缩放*S*[*x*]和*S*[*y*]，将其转换为点*P*^′，其向量![Image](../images/vivictorit.jpg)的水平投影为*S*[*x*] ⋅ *v*[*x*]，垂直投影为*S*[*y*] ⋅ *v*[*y*]。正如你所看到的，缩放是衡量点相对于其原始距离与原点之间的远近。事实上，纯粹的缩放变换并不会移动原点。绝对值小于1的缩放会将点拉近原点，而大于1的缩放则会把点推远原点。

![Image](../images/07fig08.jpg)

*图 7-8：一个尺度仿射变换*

这是有用的，但通常我们希望相对于原点以外的某个点应用缩放。例如，假设你正在使用AutoCAD并想放大图纸。如果缩放是相对于原点（假设它位于应用窗口的左下角）而不是围绕屏幕的中心或鼠标位置进行的，那么你会觉得图纸被移远了，如[图 7-9](ch07.xhtml#ch7fig9)左图所示。

![Image](../images/07fig09.jpg)

*图 7-9：围绕原点缩放（左）与围绕屏幕中心缩放（右）*

你可能更习惯使用一个在屏幕中间某个点或甚至鼠标位置周围缩放绘图的功能，这种功能实际上在大多数情况下都会发生。许多图形设计程序都像这样工作，这样对用户来说更方便，但按照我们定义的纯缩放变换，它只能相对于原点发生。那么，如何在任意点周围进行缩放呢？好吧，现在我们知道了如何构造复合变换，实际上，获得这个变换就是轻而易举的事了。

**注意**

*我花了相当长的时间才完全理解如何有效地使用仿射变换，以及如何从简单的变换中创建复合变换。我在尝试在我的软件 InkStructure 中实现一个正确的“放大”选项时遇到了很大的困难，这也是为什么原始版本在尝试放大绘图时感觉有些 bug，当时图形会随机移动到屏幕的其他位置。所以，当我说“轻而易举”时，我应该加上一些说明：只有在理解了仿射变换之后，它才变得简单。*

让我们快速说明我们想解决的问题：我们想找到一个仿射变换，它相对于中心点 *C* 应用缩放 *S*[*x*] 和 *S*[*y*]。定义 *O* 为坐标系的原点，我们可以通过组合以下更简单的变换来构建这样的变换：

1.  [*T*[1]]: 移动 *C* 使其与原点 *O* 重合 (![Image](../images/tvictorit.jpg) = ![Image](../images/coline.jpg) = ⟨–*C*[x], –*C*[y]⟩)。

1.  [*T*[2]]: 使用缩放因子 *S*[*x*] 和 *S*[*y*] 进行缩放。

1.  [*T*[3]]: 将 *C* 移动回原来的位置 (![Image](../images/tivictor.jpg) = ![Image](../images/ocline.jpg) = ⟨*C*[x], *C*[y]⟩)。

由于缩放只能相对于原点应用，我们将整个空间移动，使得我们的点 *C* 恰好位于原点，然后应用缩放并将其移动回原来的位置。漂亮吧？因此，[*T*[*r*]] 可以通过[方程 7.7](ch07.xhtml#ch07eqa07)计算得出。

![Image](../images/07eqa07.jpg)

让我们创建一个工厂函数来生成这些类型的变换。首先，创建一个名为 *affine_transforms.py* 的新文件；在其中输入[列表 7-12](ch07.xhtml#ch7lis12)中的函数。

```py
from geom2d.affine_transf import AffineTransform
from geom2d.point import Point

def make_scale(sx: float, sy: float , center=Point(0, 0)):
    return AffineTransform(
        sx=sx,
        sy=sy,
        tx=center.x * (1.0 - sx),
        ty=center.y * (1.0 - sy)
    )
```

*列表 7-12：创建一个缩放变换*

最好添加一些测试用例，检查该函数的行为。为了简洁起见，我会把这个留给你作为练习。

#### ***旋转***

类似于缩放，旋转总是围绕原点进行。就像之前一样，通过使用巧妙的变换序列，我们可以围绕任何我们想要的点进行旋转。你可能在 Sketch、Illustrator 或类似应用程序中旋转过图形，在这种情况下，你已经习惯选择*旋转中心*，一个围绕它旋转的点，你可以使用方框控制柄，类似于[图 7-10](ch07.xhtml#ch7fig10)。

![Image](../images/07fig10.jpg)

*图 7-10：围绕中心的旋转*

旋转中心点可以移动，从而使旋转围绕不同的点进行。例如，将旋转中心移到边界框的左下角，旋转可能会像[图 7-11](ch07.xhtml#ch7fig11)那样。

![图片](../images/07fig11.jpg)

*图 7-11：围绕角落的旋转*

让我们从学习如何围绕原点构建旋转仿射变换开始；这将作为构建围绕任何点旋转的更复杂变换的基础。[方程 7.8](ch07.xhtml#ch07eqa08)展示了如何围绕原点旋转点 *θ* 弧度。

![图片](../images/07eqa08.jpg)

记住这一点后，让我们找到一个仿射变换，使得点围绕中心点 *C* 旋转 *θ* 弧度。以 *O* 作为坐标系的原点，变换是以下内容的组合：

1.  [*T*[1]]：将 *C* 平移到原点 *O*，使得旋转中心为 *C* (![图片](../images/tvictorit.jpg) = ![图片](../images/coline.jpg) = ⟨–*C*[x], –*C*[y]⟩)。

1.  [*T*[2]]：旋转 *θ* 弧度。

1.  [*T*[3]]：将 *C* 平移回它原来的位置 (![图片](../images/tivictor.jpg) = ![图片](../images/ocline.jpg) = ⟨*C*[x], *C*[y]⟩)。

这与之前的算法相同，但这次我们使用旋转而不是缩放。[*T*[*r*]]现在的计算方式如下：

![图片](../images/f0190-01.jpg)

这得到了[方程 7.9](ch07.xhtml#ch07eqa09)中的仿射变换。

![图片](../images/07eqa09.jpg)

让我们创建一个新的工厂函数，用于围绕中心点生成旋转。在*affine_transforms.py*中，在[方程 7.9](ch07.xhtml#ch07eqa09)的帮助下，实现新的函数，[清单 7-13](ch07.xhtml#ch7lis13)中可以找到。

```py
import math

from geom2d.affine_transf import AffineTransform
from geom2d.point import Point

--snip--

def make_rotation(radians: float, center=Point(0, 0)):
    cos = math.cos(radians)
    sin = math.sin(radians)
    one_minus_cos = 1.0 - cos

    return AffineTransform(
        sx=cos,
        sy=cos,
        tx=center.x * one_minus_cos + center.y * sin,
        ty=center.y * one_minus_cos - center.x * sin,
        shx=-sin,
        shy=sin
    )
```

*清单 7-13：创建旋转变换*

再次提醒，您需要至少编写一个单元测试，确保我们的实现没有漏洞。

让我们在 shell 中尝试一下：创建两个旋转，分别是*π*/4 弧度，一个围绕原点，另一个围绕点 (10, 10)。然后，我们将这两个旋转应用到点 (15, 15)，看看它们在两种情况下的结果。重新加载 Python shell 并写入以下内容：

```py
>>> from geom2d.affine_transforms import make_rotation
>>> from geom2d.point import Point
>>> import math
>>> point = Point(15, 15)
```

现在让我们尝试围绕原点进行旋转：

```py
>>> rot_origin = make_rotation(math.pi / 4)
>>> str(rot_origin.apply_to_point(point))
'(1.7763568394002505e-15, 21.213203435596427)'
```

结果点的 x 坐标基本为零（注意指数 e-15），y 坐标为 21.2132...，这就是从原点到原始点的向量长度 ![图片](../images/f00192.jpg)。

让我们尝试第二次旋转：

```py
>>> rot_other = make_rotation(math.pi / 4, Point(10, 10))
>>> str(rot_other.apply_to_point(point))
'(10.000000000000002, 17.071067811865476)'
```

这次结果点是 (10, 17.071...)。为了帮助我们理解刚刚做的练习，[图 7-12](ch07.xhtml#ch7fig12)展示了这两次旋转变换。

![图片](../images/07fig12.jpg)

*图 7-12：围绕原点旋转（左）与围绕点 (10, 10) 旋转（右）的示例*

### **插值变换**

当你进行放大或缩小时，大多数图形程序不会一次性应用缩放，而是通常通过一个快速平滑的动画来展示缩放过程。这可以帮助用户更好地理解图形是如何被转换的。为了实现这一点，图形程序通常使用变换插值。在本书的后续部分，我们将为仿射变换制作动画，也就是说，我们将创建一种电影，通过每一帧逐步展示给定几何图形如何变化。每一帧动画将展示在应用部分仿射变换后的几何图形，这也是我们第一次使用*插值*的地方。

#### ***激励插值***

在深入讨论插值变换的概念之前，先看一下[图7-13](ch07.xhtml#ch7fig13)。

![Image](../images/07fig13.jpg)

*图7-13：动画中的仿射变换*

在图中，最初位于窗口左下角的三角形经过一些位置（以较浅的灰色绘制）后，最终到达窗口的顶部中间。每个三角形表示我们在某一时刻看到的结果，是动画中的一个具体帧。

如果我们希望动画有*n*帧，其中*n* > 1，那么需要有*n*个仿射变换[*T*[0]]，[*T*[1]]，...，[*T*[n–1]]，使得每一帧都是将相应变换应用到输入几何体后的结果。显然，最后一个变换[*T*[n–1]]必须是目标仿射变换，因为最后一帧应展示应用该变换后的几何图形。那么[*T*[0]]应该是什么呢？让我们思考一下。什么变换应用到输入几何体后能得到原本的几何体呢？嗯，只有一个这样的变换，它不会移动任何东西，那就是恒等变换。所以，我们的起始变换和结束变换如下所示：

![Image](../images/f0193-01.jpg)

我们如何计算[*T*[1]]，...，[*T*[n–2]]呢？其实很简单：我们可以通过插值法将每个起始值和结束值进行插值，从而得到所需的中间值。例如，从0到5进行线性插值，使用五个步骤会得到[0, 1, 2, 3, 4, 5]。注意，五个步骤产生六个值，所以要得到*n*帧，我们需要使用*n–1*步骤。

要从起始值*v*[*s*]插值到结束值*v*[*e*]，我们可以使用任何一个通过这两个值的函数。直线（线性函数）是最简单的，得到的值是均匀间隔的。这就是线性插值。如果我们使用这种插值来生成动画的帧，结果将以恒定的速度从开始到结束移动（插值函数的斜率是恒定的），这在人眼中看起来不自然。为什么会这样呢？因为我们不习惯看到现实生活中的事物突然加速、以相同速度运动并突然停止。虽然这对于投射物或子弹可能没问题，但对于大多数现实生活中的运动物体来说就显得很奇怪。我们可以尝试一种看起来更自然的插值函数，比如在[图 7-14](ch07.xhtml#ch7fig14)右侧图形中绘制的*ease-in-out*插值。

![Image](../images/07fig14.jpg)

*图 7-14：两种插值函数*

在一个ease-in-out函数中，开始和结束时的值变化较慢，这给人一种事物开始运动时加速，达到运动终点时缓慢减速的感觉。这个函数以更自然的方式定义了运动，跟随这种位置随时间变化的动画在人眼中看起来很美观。

要获得介于*v*[*s*]和*v*[*e*]之间的值，我们使用一个参数*t*，使得0 ≤ *t* ≤ 1（见[方程 7.10](ch07.xhtml#ch07eqa10)）。

![Image](../images/07eqa10.jpg)

你可以很容易地观察到，[方程 7.10](ch07.xhtml#ch07eqa10)在*t* = 0时给出*v*[*s*]的结果，在*t* = 1时给出*v*[*e*]的结果。对于任何介于这两个值之间的*t*值，结果都会在这两个值之间变化。如果我们想得到一个从*v*[*s*]到*v*[*e*]的值序列，并且该序列呈线性分布，我们只需为*t*使用均匀间隔的值，例如[0, 0.25, 0.5, 0.75, 1]。

要生成ease-in-out分布的插值值，我们需要一个从0到1的不均匀间隔的*t*值序列，极值附近步长较小，中央附近步长较大。如果我们将*t*值表示为在水平线上从*t* = 0到*t* = 1的圆点，我们可以通过[图 7-15](ch07.xhtml#ch7fig15)来直观了解均匀和ease-in-out值是如何分布的。

![Image](../images/07fig15.jpg)

*图 7-15：插值t值*

要构建一个按[图 7-14](ch07.xhtml#ch7fig14)右侧图形分布的*t*值序列，我们可以将一系列均匀间隔的*t*值代入[方程 7.11](ch07.xhtml#ch07eqa11)。

![Image](../images/07eqa11.jpg)

这改变了它们的间隔，使得更多的值位于极值0和1附近，较少的值位于中间。

我们已经准备好所需的所有要素，开始动手吧！

#### ***实现插值***

在 *geom2d* 中创建一个名为 *interpolation.py* 的新文件，并输入 [列表 7-14](ch07.xhtml#ch7lis14) 中的代码。

```py
def uniform_t_sequence(steps: int):
    return [t / steps for t in range(steps + 1)]

def ease_in_out_t_sequence(steps: int):
    return [ease_in_out_t(t) for t in uniform_t_sequence(steps)]

def ease_in_out_t(t: float):
    return t ** 2 / (t ** 2 + (1 - t) ** 2)
```

*列表 7-14：插值的 t 值*

从底部开始，我们有函数 ease_in_out_t，它仅仅是 [方程 7.11](ch07.xhtml#ch07eqa11) 的实现。第一个函数使用给定的步数构建一个均匀分布的 *t* 值序列，从而生成与步数加一相同数量的值。我们可以在 shell 中测试这一点。重新加载并尝试以下操作：

```py
>>> from geom2d.interpolation import uniform_t_sequence
>>> uniform_t_sequence(10)
[0.0, 0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1.0]
```

另一方面，函数 ease_in_out_t_sequence 创建遵循 ease-in-out 分布的序列。为此，它将 [方程 7.11](ch07.xhtml#ch07eqa11) 应用到均匀序列的值上。我们也可以在 shell 中尝试一下：

```py
>>> ease_in_out_t_sequence(10)
[0.0, 0.012195121951219514, 0.058823529411764705,
0.15517241379310345, 0.30769230769230776, 0.5,
0.6923076923076923, 0.8448275862068965,
0.9411764705882353, 0.9878048780487805, 1.0]
```

看看接近 0 和 1 的值是否更接近，而中间的值（接近 0.5）则相隔更远？太好了，现在我们只缺一个函数，用于给定 *t* 值时在两个值之间进行插值，就像 [方程 7.10](ch07.xhtml#ch07eqa10) 定义的那样。在 *interpolation.py* 中添加 [列表 7-15](ch07.xhtml#ch7lis15)。

```py
import geom2d.tparam as tparam

--snip--

def interpolate(vs: float, ve: float, t: float):
    tparam.ensure_valid(t)
    return vs + t * (ve - vs)
```

*列表 7-15：给定 t 值时的插值*

如果你还记得 [第 5 章](ch05.xhtml#ch05)，当我们使用传入的 *t* 参数值进行操作时，我们需要检查它是否在预期的范围内，确保有效函数（ensure_valid）用于此。现在我们已准备好最后一步，希望你跟上了，因为这就是我们一直追求的仿射变换插值。打开你的文件 *affine_transforms.py*，在其中我们定义了创建几种特殊类型仿射变换的工厂函数，然后输入 [列表 7-16](ch07.xhtml#ch7lis16) 中的代码。

```py
import math

from geom2d.affine_transf import AffineTransform
from geom2d.interpolation import ease_in_out_t_sequence, interpolate
from geom2d.point import Point

--snip--

def ease_in_out_interpolation(start, end, steps):
 ➊ t_seq = ease_in_out_t_sequence(steps)
 ➋ return [__interpolated(start, end, t) for t in t_seq]

def __interpolated(s: AffineTransform, e: AffineTransform, t):
 ➌ return AffineTransform(
        sx=interpolate(s.sx, e.sx, t),
        sy=interpolate(s.sy, e.sy, t),
        tx=interpolate(s.tx, e.tx, t),
        ty=interpolate(s.ty, e.ty, t),
        shx=interpolate(s.shx, e.shx, t),
        shy=interpolate(s.shy, e.shy, t)
    )
```

*列表 7-16：插值的仿射变换序列*

为了帮助生成插值的仿射变换序列，我们定义了一个私有函数 __interpolated，它接受两个变换和一个 *t* 值作为输入，返回该 *t* 值的插值 ➌。新变换的每个值都是对起始变换和结束变换的值进行插值的结果。然后，我们构建一个遵循 ease-in-out 分布的 *t* 值序列 ➊，每个 *t* 值都通过列表推导映射到相应的插值变换 ➋。

我们先将这个部分留到 [第 12 章](ch12.xhtml#ch12)，到时我们将使用插值的仿射变换序列来制作动画。如果这一章最后部分探讨的概念看起来有些抽象，不要担心。我们将在本书的下一部分构建动画运动的基础，到那时，这个插值的概念可能会变得更容易理解。

### **Geom2D 最后润色**

我们的 *geom2d* 包已经经过测试，准备在本书剩余部分使用。我们使它变得稳健，但在本书这一部分结束之前，我们可以添加一些小的改进。

#### ***测试文件***

我们首先要做的是将实现文件和测试文件分开，当前它们都在同一个文件夹中。这样做是为了让 *geom2d* 包文件夹看起来不那么杂乱，方便你更容易找到实现文件。在包内创建一个名为 *tests* 的新文件夹，然后选择所有测试文件（我们便捷地将它们命名为以 *_test.py* 结尾），并将它们拖动到该文件夹中。你的文件夹结构和文件应如下所示：

Mechanics

|- geom2d

|    |- tests

|    |    |- affine_transf_test.py

|    |    |- affine_transforms_test.py

|    |    |- circle_test.py

|    |    |- ...

|    |    |- vector_test.py

|    |- __init__.py

|    |- affine_transf.py

|    |- affine_transforms.py

|    |- circle.py

|    |- ...

|    |- vectors.py

#### ***运行所有测试***

现在，所有测试文件都在同一个文件夹中，那如何一次性运行所有测试用例呢？可能是你更改了部分代码，想确保没有破坏任何功能，于是你决定运行包中的每个测试。按照我们之前的方法，这需要花费一些时间，因为你需要逐个打开测试文件，并点击每个类名称旁边的绿色播放按钮。其实有更好的方法！

打开 PyCharm 内的终端视图。如果看不到，可以从菜单中选择 **View** ▸ **Tool Windows** ▸ **Terminal**。默认情况下，终端会在项目的根目录下打开，这正是我们所需要的。在终端中，运行以下命令：

```py
$ python3 -m unittest discover -s geom2d/tests/ -p '*_test.py'
```

该命令告诉 Python 查找并运行所有位于 *geom2d/tests/* 中的单元测试，文件名符合 **_test.py** 模式，即所有以 *_test.py* 结尾的文件。运行该命令后，结果应类似于以下内容：

```py
Ran 58 tests in 0.004s

OK
```

你可以将此命令保存在项目根目录下的 bash 文件中，这样你就可以随时执行它，而无需记住它。

#### ***包导入***

最后，我们要做的是将所有模块包含在包的导出中，这样它们就可以像这样被加载：

```py
from geom2d import Point, Polygon
```

将其与以下内容进行比较：

```py
from geom2d.point import Point
from geom2d.polygon import Polygon
```

后者要求用户输入每个模块在 *geom2d* 中的路径，而前者则不需要：包内的所有内容都可以直接从包本身导入。采用这种导出模块的方式有两个优点：（1）它允许我们在不破坏用户导入的情况下改变模块的目录结构，（2）用户不需要知道每个模块在包内的具体位置，所有内容都可以从包本身直接导入。如你所见，这大大减少了使用包时的认知负担。

当 PyCharm 创建 *geom2d* 包时，它会在其中包含一个名为 *__init__.py* 的空文件。你能找到它吗？包中的文件名为此的文件会在包被导入时自动加载。我们可以利用它们来导入包内定义的内容。

**注意**

*如果由于某种原因，文件* __init__.py *在你的* geom2d *包中不存在，直接创建它即可。也许你在 PyCharm 中将包创建为普通目录，因此 IDE 没有为你自动添加它。*

所以，打开文件，它应该是空的，并导入我们定义的所有基本元素（见[列表 7-17](ch07.xhtml#ch7lis17)）。

```py
from .point import Point
from .vector import Vector
from .vectors import *
from .circle import Circle
from .circles import *
from .interpolation import *
from .line import Line
from .nums import *
from .open_interval import OpenInterval
from .polygon import Polygon
from .rect import Rect
from .rects import *
from .segment import Segment
from .size import Size
from .tparam import *
from .affine_transf import *
from .affine_transforms import *
```

*列表 7-17：geom2d 包的初始化文件*

就是这样！要理解我们通过这个更改实现了什么，你可以在 Python 的 shell 中（不是我们刚才用来运行命令的 shell）尝试以下操作：

```py
>>> from geom2d import Point, Size, Rect
>>> origin = Point(2, 3)
>>> size = Size(10, 15)
>>> rect = Rect(origin, size)
```

这将在后续章节中非常方便，因为我们可以直接从包中导入任何*geom2d*模块。

### **总结**

在本章中，我们探讨了计算机图形学中的一个核心概念：仿射变换。它们允许我们通过缩放、旋转、平移和剪切来变换几何图形。

我们首先回顾了它们的数学定义，以及它们与线性变换的区别。结论是，仿射变换可以移动原点，而线性变换不能。仿射变换可以表示为线性变换与平移的组合，但我们看到了一个更方便的表示方法：增广矩阵。接下来，我们在 AffineTransform 类中实现了方法，用于变换我们的几何原始元素：点、线段和多边形。

然后，我们学习了如何将变换组合起来，以通过简单的变换实现复杂的变换。凭借这个强大的思想，我们能够构造出在几乎所有图形应用程序中都会用到的两个基本仿射变换：缩放和绕原点以外的点旋转。

最后，我们实现了一个函数，用于在两个仿射变换之间进行插值，从而生成一些中间变换，这些变换将很快用于制作动画。
