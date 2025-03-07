## 5

线与线段

![Image](../images/common01.jpg)

一个点和一个方向描述了一条无限的直线，没有起点或终点。两个不同的点限定了一条线段，它具有有限的长度，但包含无限个点。在本章中，我们将重点关注这两种基本元素——线段和直线。我们将借助前一章中实现的点和向量来实现这两者。

我们还将花一些时间理解和实现两个算法：一个用于计算离线段最近的点，另一个用于计算线段交点。这些算法使用了一些几何学中的重要概念，这些概念将作为更复杂问题的基础。我们将花时间实现这些操作，以确保我们理解它们，所以准备好你的Python IDE，拿出纸笔——以老式方式画图会很有帮助。

### **线段类**

在平面上任意两点之间都存在一条唯一的*线段*，它是一条有限长度、包含无限个点的直线。[图5-1](ch05.xhtml#ch5fig1)描绘了两点* S *和* E *之间的线段。

![Image](../images/05fig01.jpg)

*图5-1：S点和E点之间的线段*

让我们从创建一个名为Segment的类开始，类中有两个属性：起点*S*和终点*E*。到目前为止，我们的项目结构如下所示：

机械学

|- geom2d

|    |- __init__.py

|    |- nums.py

|    |- point.py

|    |- point_test.py

|    |- vector.py

|    |- vector_test.py

|    |- vectors.py

右键点击*geom2d*包，选择**新建** ▸ **Python文件**，命名为*segment*，然后点击**确定**。PyCharm会为你自动添加*.py*扩展名，但如果你使用的是其他IDE，可能需要手动添加。在文件中，输入如[列表5-1](ch05.xhtml#ch5lis1)所写的类。

```py
from geom2d.point import Point

class Segment:
    def __init__(self, start: Point, end: Point):
        self.start = start
        self.end = end
```

*列表5-1：线段初始化*

我们首先从*geom2d.point*模块导入Point类。然后，我们定义了Segment类，并为其创建了一个初始化函数，该函数接受两个点：起点和终点。这些点将存储在相应的属性中。

请注意，我们对参数进行了类型标注；更具体地说，我们表示它们必须是Point类型。这些是我们在[第2章](ch02.xhtml#ch02)中看到的*类型提示*，主要是为了让IDE为我们提供一些上下文帮助。如果IDE知道起点和终点都是Point类的对象，它会检测我们是否在尝试使用该类没有实现的属性。但需要注意的是，这不会阻止我们在运行时传递错误的参数类型。事实上，如果你在控制台尝试以下代码：

```py
>>> from geom2d.segment import Segment
>>> s = Segment("foo", "bar")

>>> s.start
'foo'
```

你应该会看到，Python允许我们传递字符串而不是Points而不报错，因为类型提示在Python解释器运行时会被忽略。

#### ***线段的方向***

线段的一个重要属性是它的*方向*，定义为从起点*S*到终点*E*的向量。如果我们将其称为![Image](../images/dvictorit.jpg)，我们可以通过[方程式5.1](ch05.xhtml#ch05eqa01)来计算它。

![Image](../images/05eqa01.jpg)

方向向量的归一化得到方向单位向量，这在许多与线段的操作中也常常使用。*方向向量*是一个与线段长度相同并与其平行的向量，方向从起点指向终点。*方向单位向量*是方向向量的归一化版本，即一个具有相同方向但单位长度的向量。

方向单位向量 ![Image](../images/dcapit.jpg)，给定一个长度为*l*的线段，计算公式如[公式 5.2](ch05.xhtml#ch05eqa02)所示。

![Image](../images/05eqa02.jpg)

**注意**

*当我们说*线段的方向*时，大多数时候我们指的是*方向单位向量 ![Image](../images/dcapit.jpg)*，但有时我们也会用这个短语来指代*方向向量 ![Image](../images/dvictorit.jpg)*。如果是这种情况，我们会明确说明。所以，如果没有特别说明，假设我们指的是方向单位向量。*

让我们将它们作为类的属性进行实现。将[清单 5-2](ch05.xhtml#ch5lis2)中的代码输入到你的*segment.py*文件中。

```py
from geom2d.point import Point
from geom2d.vectors import make_vector_between, make_versor_between

class Segment:
    --snip--

    @property
    def direction_vector(self):
        return make_vector_between(self.start, self.end)

    @property
    def direction_versor(self):
        return make_versor_between(self.start, self.end)
```

*清单 5-2：计算线段的方向向量和方向单位向量*

由于我们使用了在*vectors.py*中定义的 make_vector_between 和 make_versor_between 工厂函数，这两个属性的实现非常简单。我们只是创建一个起点和终点之间的向量或单位向量。

现在，线段的方向和它的垂直方向一样重要。我们可能会使用这个垂直方向，例如计算与直线碰撞的粒子速度方向，这条直线可能代表一堵墙或地面，就像[图 5-2](ch05.xhtml#ch5fig2)中的情况。

![Image](../images/05fig02.jpg)

*图 5-2：使用法线方向计算碰撞角度*

将方向单位向量 ![Image](../images/dcapit.jpg) 旋转 *π*/4 弧度（90^°）得到线段的*法线单位向量*。使用 Vector 的 perpendicular 属性计算这个单位向量非常简单。将新的属性输入到[清单 5-3](ch05.xhtml#ch5lis3)中，放在 Segment 类中。

```py
class Segment:
    --snip--

    @property
    def normal_versor(self):
        return self.direction_versor.perpendicular()
```

*清单 5-3：计算垂直于线段方向的向量*

我们添加的这个新属性通过链式调用两个属性：direction_versor 和 perpendicular。我们首先调用 self 的 direction_versor 来获取线段的方向单位向量。结果是一个 Vector 实例，接着我们调用 perpendicular 方法，它返回一个垂直于线段方向的单位向量。

我们本可以将方向单位向量存储在一个新变量中，然后对该变量调用垂直方法：

```py
def normal_versor(self):
    d = self.direction_versor
    return d.perpendicular()
```

在这种情况下，d 变量并没有增加代码的可读性，并且由于我们只使用它一次，我们可以将两个方法链式调用并返回结果。你会在我们的代码中经常看到这种模式。

你可以在[图 5-3](ch05.xhtml#ch5fig3)中看到我们刚刚实现的概念的可视化表示。左侧的线段显示了方向向量![Image](../images/dvictorit.jpg)，其起点为*S*（起点），终点为*E*（终点）。右侧的线段显示了方向向量的标准化版本![Image](../images/dcapit.jpg)及其垂直对应物![Image](../images/ncapit.jpg)，分别是方向向量和法向量。

![Image](../images/05fig03.jpg)

*图 5-3：线段方向向量（左）以及方向和法向量（右）*

在这一节中我们将跳过编写单元测试，但这并不意味着你不应该写单元测试。从现在开始，我不会为我们编写的每个方法都写测试，只会挑选一些方法，以便保持专注并推进内容。但你可以写单元测试来测试这些未测试的方法，这对你是一个很好的练习。你可以参考书中附带的*Mechanics*项目中的测试。

#### ***线段的长度***

线段的另一个重要属性是它的*长度*，即其端点之间的距离。

##### **计算长度**

我们可以通过至少两种方式计算该段的长度：我们可以计算点*S*和*E*之间的距离，或者计算方向向量的长度![Image](../images/dvictorit.jpg)。

我们将使用第一种方法，如[Listing 5-4](ch05.xhtml#ch5lis4)所示，但如果你愿意，也可以实现第二种方法。结果应该是相同的。

```py
class Segment:
    --snip--

    @property
    def length(self):
        return self.start.distance_to(self.end)
```

*Listing 5-4：计算线段的长度*

再次注意，使用我们之前实现的方法使得这个计算变得轻松。此时，你的*segment.py*文件应该看起来像[Listing 5-5](ch05.xhtml#ch5lis5)。

```py
from geom2d.point import Point
from geom2d.vectors import make_vector_between, make_versor_between

class Segment:
    def __init__(self, start: Point, end: Point):
        self.start = start
        self.end = end

    @property
    def direction_vector(self):
        return make_vector_between(self.start, self.end)

    @property
    def direction_versor(self):
        return make_versor_between(self.start, self.end)

    @property
    def normal_versor(self):
        return self.direction_versor.perpendicular()

    @property
    def length(self):
        return self.start.distance_to(self.end)
```

*Listing 5-5：Segment类*

让我们测试一下刚写的方法。

##### **单元测试长度**

为了确保我们在实现长度属性时没有犯错，让我们编写一个单元测试。首先创建一个新的测试文件。右键点击*geom2d*包，选择**新建** ▸ **Python文件**，命名为*segment_test.py*，然后点击**确定**。然后输入[Listing 5-6](ch05.xhtml#ch5lis6)中的代码。

```py
import math
import unittest

from geom2d.point import Point
from geom2d.segment import Segment

class TestSegment(unittest.TestCase):

    start = Point(400, 0)
    end = Point(0, 400)
    segment = Segment(start, end)

    def test_length(self):
        expected = 400 * math.sqrt(2)
        actual = self.segment.length
        self.assertAlmostEqual(expected, actual)
```

*Listing 5-6：测试线段的长度属性*

我们导入了*unittest*和*math*模块以及Segment和Point类。接着，我们定义了两个点：起点为（400, 0），终点为（0, 400）。利用这两个点，我们创建了segment，它是我们的测试对象。按照好单元测试的规则1，一个测试应该仅因一个原因失败，我们的预期结果直接表示为![Image](../images/f00107-p1.jpg)，该结果来自![Image](../images/f00107-p2.jpg)。这里的诱惑是写下如下内容：

```py
expected = self.start.distance_to(self.end)
```

然而，这会违反规则1，因为测试可能因为多个原因失败。此外，在这种情况下，预期值和实际值将使用相同的方法进行计算：distance_to。这破坏了测试与它应该测试的代码之间的独立性。

通过点击TestSegment类定义左侧的绿色播放按钮，并选择**运行‘线段单元测试’**来运行测试。你也可以通过控制台运行：

```py
$ python3 -m unittest geom2d/segment_test.py
```

测试距离属性可能看起来有些愚蠢，因为它唯一的作用就是调用已经测试过的distance_to方法。即使是这样简单的实现，我们也可能会犯错误，比如尝试用同一个点计算两次距离：

```py
self.start.distance_to(self.start)
```

正如你可能从自己的经验中知道的那样，我们开发者经常会犯这样的错误。

#### ***t 参数和中点***

我们之前说过，线段的起点 *E* 和终点 *S* 之间有无数个点。我们该如何获取这些点呢？通常使用一个从0到1（包括1）的参数来获取线段上的每个点。我们将这个参数称为 *t*，并按照[方程 5.3](ch05.xhtml#ch05eqa03)中的定义进行定义。

![Image](../images/05eqa03.jpg)

所有线段上的点都可以通过变化 *t* 的值来获得。对于 *t* = 0，我们得到线段的起点 *S*。同样，对于 *t* = 1，我们得到终点 *E*。为了计算给定 *t* 值的任何中点 *P*，我们可以使用[方程 5.4](ch05.xhtml#ch05eqa04)。

![Image](../images/05eqa04.jpg)

通过意识到前面表达式中的向量正是[方程 5.1](ch05.xhtml#ch05eqa01)中定义的方向向量，我们可以将表达式简化为[方程 5.5](ch05.xhtml#ch05eqa05)中的形式。

![Image](../images/05eqa05.jpg)

我们可以轻松地通过 Point 类的 displaced 方法实现[方程 5.5](ch05.xhtml#ch05eqa05)。将[清单 5-7](ch05.xhtml#ch5lis7)中的point_at方法代码添加到你的Segment类文件中（*segment.py*）。

```py
class Segment:
    --snip--

   def point_at(self, t: float):
      return self.start.displaced(self.direction_vector, t)
```

*清单 5-7: 使用参数 t 从线段获取点*

通过将起始点沿着方向向量* t *方向平移*t*倍（其中0.0 ≤ *t* ≤ 1.0），我们可以得到线段上的任何一点。我们来实现一个属性，直接得到线段的中点，也就是当 *t* = 0.5 时的点（见[图 5-4](ch05.xhtml#ch5fig4)）。

![Image](../images/05fig04.jpg)

*图 5-4: 线段的中点*

这是我们经常计算的特殊点，因此我们希望能够方便地获取它。将[清单 5-8](ch05.xhtml#ch5lis8)中的代码添加进去。

```py
class Segment:
    --snip--

    @property
   def middle(self):
       return self.point_at(0.5)
```

*清单 5-8: 线段的中点*

##### **验证 t 值**

你可能已经意识到，在point_at方法中，我们并没有检查传入的 *t* 值是否在[方程 5.3](ch05.xhtml#ch05eqa03)给出的预期范围内。我们可以传入一个错误的 *t* 值，它依然可以正常工作，返回超出线段的点。例如，如果我们传入 *t* = 1.5 的值，我们将得到[图 5-5](ch05.xhtml#ch5fig5)中所示的点。

![Image](../images/05fig05.jpg)

*图 5-5: t = 1.5 时线段外的点*

如果不验证 *t* 值，这个方法会悄悄地失败，返回一个点，用户可能会误以为它位于线段的端点之间。我们所说的 *悄悄失败*，是指结果在概念上是错误的，但方法仍然愉快地为我们计算出这个结果，并没有任何警告或提示，表示可能存在错误。

稳健的软件 *快速失败*，意味着一旦检测到错误条件，程序就会出现故障并退出，如果可能的话，附带一条提供全面错误信息的消息。

这听起来可能有些可怕，但它帮助很大。假设我们允许用户向我们的 `point_at(t)` 方法传递一个错误的 *t* 值。现在，假设用户没有注意到，传入了一个像 739928393839... 这样的 *t* 值。你可以想象，从这个值得到的点将远离应该包含它的线段。这样的值不会导致我们的程序崩溃，程序会继续执行。我们可能直到几分钟后的某次计算才会发现得到了这样一个值，这时一切都失败了。在我们发现错误之前调试这些发生的事情可能需要几个小时（或者根据代码的复杂性和错误传播的范围，甚至几天）。如果我们能够立刻检测到错误的值，那就简单多了。也许我们可以像这样告诉用户：

```py
Oops! We were expecting the value of 't' to be in the  range [0, 1],
but you gave us a value of '739928393839'.
```

这个消息非常清晰。它告诉用户程序因错误必须退出。如果程序继续执行，错误可能会变得更严重。好的一点是，用户有机会分析错误值的来源，并采取措施防止其再次发生。

**注意**

*在这里我们使用“用户”这个词，指的是任何使用我们代码的人，而不是我们编写的应用程序的最终用户。这包括你自己，因为你将经常是自己代码的用户。*

由于需要为 *t* 参数定义一堆功能，我们最好为它创建一个模块。此时，你的项目结构应该是这样的：

Mechanics

|- geom2d

|    |- __init__.py

|    |- nums.py

|    |- point.py

|    |- point_test.py

|    |- segment.py

|    |- segment_test.py

|    |- vector.py

|    |- vector_test.py

|    |- vectors.py

在 *geom2d* 包内创建一个新文件，命名为 *tparam.py*。在其中输入 [Listing 5-9](ch05.xhtml#ch5lis9) 中的代码。

```py
MIN = 0.0
MIDDLE = 0.5
MAX = 1.0

def make(value: float):
    if value < MIN:
        return MIN

    if value > MAX:
        return MAX

    return value

def ensure_valid(t):
    if not is_valid(t):
        raise TParamError(t)

def is_valid(t):
    return False if t < MIN or t > MAX else True

class TParamError(Exception):
    def __init__(self, t):
        self.t = t

    def __str__(self):
        return f'Expected t to be in [0, 1] but was {self.t}'
```

*Listing 5-9: 验证 t 参数值*

我们首先定义三个有用的常量。MIN 是 *t* 可以取的最小值。MIDDLE 是 (MIN + MAX) / 2 的值。最后，MAX 是 *t* 可以取的最大值。

这些值将会被多次使用，因此我们没有在每个地方写 *魔法数字*（硬编码的数字，没有解释它们的含义），而是给它们命名以便理解它们代表的意义。

一旦我们定义了这些值，我们定义了一个函数 `make` 来创建一个具有有效值的参数。接着是 `ensure_valid` 函数，它通过另一个方法 `is_valid` 来检查 *t* 是否小于或大于范围限制。如果 *t* 的值超出了有效范围，将会引发异常。TParam Error 是 Python 异常的实现。这是一个用户自定义的异常，我们为其提供了格式良好的信息。在 TParam Error 的初始化器中，我们传递了出错的 *t* 值，在特殊方法 `__str__` 中，我们返回实际的错误信息。回想一下，一个类可以定义 `__str__` 方法来提供实例的文本（字符串）表示形式，当该方法被调用时。

要查看它如何打印信息，请在控制台尝试以下操作：

```py
>>> from geom2d import tparam
>>> tparam.ensure_valid(10.5)
Traceback (most recent call last):
  --snip--
geom2d.tparam.TParamError: Expected t to be in [0, 1] but was 10.5
```

错误信息清晰明了：

```py
Expected t to be in [0, 1] but was 10.5
```

让我们在 `Segment` 类的 `point_at` 方法中使用这个验证。首先，在你的 *segment.py* 文件中导入模块：

```py
from geom2d import tparam
```

回到 *segment.py*，重构 `point_at(t)`，包括验证，如 [列表 5-10](ch05.xhtml#ch5lis10) 中所示。

```py
def point_at(self, t: float):
    tparam.ensure_valid(t)
    return self.start.displaced(self.direction_vector, t)
```

*列表 5-10：在 segment 的 point_at 方法中验证 t 的值*

然后按照 [列表 5-11](ch05.xhtml#ch5lis11) 中的示例，重构中点属性，去除 0.5 的魔法数字。

```py
@property
def middle(self):
    return self.point_at(tparam.MIDDLE)
```

*列表 5-11：从中点计算中去除魔法数字*

如果你跟着做，你的 *segment.py* 文件应该如下所示：[列表 5-12](ch05.xhtml#ch5lis12)。

```py
from geom2d import tparam
from geom2d.point import Point
from geom2d.vectors import make_vector_between, make_versor_between

class Segment:
    def __init__(self, start: Point, end: Point):
        self.start = start
        self.end = end

    @property
    def direction_vector(self):
        return make_vector_between(self.start, self.end)

    @property
    def direction_versor(self):
        return make_versor_between(self.start, self.end)

    @property
    def normal_versor(self):
        return self.direction_versor.perpendicular()

    @property
    def length(self):
        return self.start.distance_to(self.end)

    def point_at(self, t: float):
        tparam.ensure_valid(t)
        return self.start.displaced(self.direction_vector, t)

    @property
    def middle(self):
        return self.point_at(tparam.MIDDLE)
```

*列表 5-12：Segment 类*

在我们的 `Segment` 类完成后，让我们编写一些测试。

##### **单元测试 Segment 点**

由于我们将把 `point_at` 作为更复杂计算的一部分，我们真的很想确保它能正常工作，因此让我们从一个测试开始，断言如果它传入一个错误的 *t* 值，实际上会引发异常。这为我们提供了一个学习新断言方法 `assertRaises` 的机会。

在 *segment_test.py* 文件中，首先导入 *tparam* 模块：

```py
from geom2d import tparam
```

然后编写 [列表 5-13](ch05.xhtml#ch5lis13) 中的测试。

```py
class TestSegment(unittest.TestCase):

    start = Point(400, 0)
    end = Point(0, 400)
    segment = Segment(start, end)

    --snip--

    def test_point_at_wrong_t(self):
        self.assertRaises(
         ➊ tparam.TParamError,
         ➋ self.segment.point_at,
         ➌ 56.7
        )
```

*列表 5-13：测试 t 的错误值*

这个断言比我们之前看到的那些稍微复杂一点。我们传递了三个参数。第一个是预期引发的异常（TParamError） ➊。第二个，我们传递了预期引发异常的方法 ➋。最后，我们传递了要传递给前面方法的参数（此例中为 `point_at`），作为以逗号分隔的参数 ➌。

这个断言可以理解为：

断言方法 'point_at' 来自实例 'self.segment'

引发类型为 'tparam.TParamError' 的异常

当调用参数为 '56.7' 时

如果 `point_at` 方法接受多个参数，你可以将它们作为 `assertRaises` 的参数。现在，让我们包含来自 [列表 5-14](ch05.xhtml#ch5lis14) 的两个测试用例。

```py
class TestSegment(unittest.TestCase):

    start = Point(400, 0)
    end = Point(0, 400)
    segment = Segment(start, end)

    --snip--

    def test_point_at(self):
        t = tparam.make(0.25)
        expected = Point(300, 100)
        actual = self.segment.point_at(t)
        self.assertEqual(expected, actual)

    def test_middle_point(self):
        expected = Point(200, 200)
        actual = self.segment.middle
        self.assertEqual(expected, actual)
```

*列表 5-14：测试 point_at 方法*

在第一个测试案例中，我们确保对于有效的 *t* 值（在此情况下为 0.25），可以得到预期的中点。使用 [方程 5.4](ch05.xhtml#ch05eqa04)，可以按以下方式计算该点：

![Image](../images/f0114-01.jpg)

第二个测试是中间属性，它计算 *t* = 0.5 时的点。拿一支笔和一些纸，确保点 (200, 200) 在我们的测试中。然后运行 *segment_test.py* 文件中的所有测试，以确保它们都通过。你可以通过以下方式在控制台运行：

```py
$  python3 -m unittest geom2d/segment_test.py
```

#### ***最近点***

现在假设我们想知道线段上离外部点最近的点是什么。如果外部点与线段不对齐，即通过该点的垂线不与线段相交，那么最近的点必须是两个端点之一：*S* 或 *E*。另一方面，如果外部点与线段对齐，则垂直线与线段的交点就是最近的点。[图 5-6](ch05.xhtml#ch5fig6)展示了这一点。

![Image](../images/05fig06.jpg)

*图 5-6：线段的最近点*

在图中，点 *S* ≡ *A′* 是距离 *A* 最近的点，点 *E* ≡ *B′* 是距离 *B* 最近的点，*C′* 是距离 *C* 最近的点。接下来让我们看看如何实现这个过程。

##### **算法**

在[第 4 章](ch04.xhtml#ch04)中使用 `projection_over` 方法的帮助下，我们可以轻松找到最近点。我们将 *P* 作为外部点，*l* 作为线段的长度，并且使用[图 5-7](ch05.xhtml#ch5fig7)中的各个点、线段和向量。

![Image](../images/05fig07.jpg)

*图 5-7：计算线段最近点的算法辅助向量*

算法如下：

1.  计算一个向量![Image](../images/vvictorit.jpg)，它从线段的 *S* 指向外部点 *P*。

1.  计算![Image](../images/vvictorit.jpg)在线段方向单位向量![Image](../images/dcapit.jpg)上的投影。

1.  根据投影的值，称之为 *v*[*s*]。最近的点 *P′* 可以通过[方程 5.6](ch05.xhtml#ch05eqa06)计算得出。

![Image](../images/05eqa06.jpg)

如果投影 *v*[*s*] 的值为负，则投影位于线段的 *S* 侧之外，因此最近的点是 *S*。对于大于 *l* 的值，线段方向上的投影长度大于线段本身。因此，结果是端点 *E*。对于闭区间 [0,*l*] 中的任何 *v*[*s*] 值，我们通过在![Image](../images/dcapit.jpg)方向上移动 *S* *v*[*s*] 倍来获得该点。[图 5-7](ch05.xhtml#ch5fig7)展示了外部点 *P* 与线段对齐的最后一种情况。

此操作的代码见[清单 5-15](ch05.xhtml#ch5lis15)。

```py
class Segment:
    --snip--

   def closest_point_to(self, p: Point):
       v = make_vector_between(self.start, p)
       d = self.direction_versor
       vs = v.projection_over(d)

       if vs < 0:
           return self.start

       if vs > self.length:
           return self.end

       return self.start.displaced(d, vs)
```

*清单 5-15：线段的最近点*

我们首先计算向量 ![图片](../images/vvictorit.jpg)。然后得到 *v*[*s*]：这是向量 ![图片](../images/vvictorit.jpg) 在段的方向单位向量 ![图片](../images/dcapit.jpg) 上的投影。如果 *v*[*s*] 小于零，我们返回起点。如果大于段的长度，我们返回终点；否则，我们计算起点的位移，得到段上的结果点。

##### **单元测试最近点**

让我们测试之前定义的三种不同情况，即 *v*[*s*] < 0，*v*[*s*] > *l* 和 0 < *v*[*s*] < *l*。[清单 5-16](ch05.xhtml#ch5lis16)显示了测试的代码。

```py
class TestSegment(unittest.TestCase):

    start = Point(400, 0)
    end = Point(0, 400)
    segment = Segment(start, end)

    --snip--

    def test_closest_point_is_start(self):
        p = Point(500, 20)
        expected = self.start
        actual = self.segment.closest_point_to(p)
        self.assertEqual(expected, actual)

    def test_closest_point_is_end(self):
        p = Point(20, 500)
        expected = self.end
        actual = self.segment.closest_point_to(p)
        self.assertEqual(expected, actual)

    def test_closest_point_is_middle(self):
        p = Point(250, 250)
        expected = Point(200, 200)
        actual = self.segment.closest_point_to(p)
        self.assertEqual(expected, actual)
```

*清单 5-16：测试段的最近点*

为了更好地理解这些测试，手动绘制段和每个外部点可能是一个不错的练习，这样你可以看出预期结果为什么会有这些值。你的绘图应该类似于[图 5-8](ch05.xhtml#ch5fig8)。此外，尝试手动解决这三种情况可能会帮助你更好地理解算法。

![图片](../images/05fig08.jpg)

*图 5-8：该段的最近点及其测试案例*

别忘了运行所有的测试并确保它们都通过。你可以通过控制台如下操作：

```py
$ python3 -m unittest geom2d/segment_test.py
```

#### ***点到点的距离***

现在我们已经知道了段与外部点之间最近的点，我们可以轻松地计算出它与该段的距离。输入方法见[清单 5-17](ch05.xhtml#ch5lis17)。

```py
class Segment:
    --snip--

   def distance_to(self, p: Point):
       return p.distance_to(
           self.closest_point_to(p)
       )
```

*清单 5-17：计算点到段的距离*

正如你在代码中看到的，段与任何给定外部点之间的距离就是该点与段上最接近它的点之间的距离。很简单，不是吗？

#### ***段交点***

现在我们进入有趣的部分。我们如何测试两段是否相交？如果它们相交，我们如何计算交点？请参考[图 5-9](ch05.xhtml#ch5fig9)中的情况。

![图片](../images/05fig09.jpg)

*图 5-9：可能的段交点情况*

左列的两个案例没有交点，但它们之间有区别。在第一个案例中，段的方向向量是平行的 (![图片](../images/d1victorit.jpg) × ![图片](../images/d2victorit.jpg) = 0)。因此，很容易知道它们不会相交。在另一个案例中，如果我们用无限长的直线代替段，它们会有交点。交点可能远离段的位置，但仍然会有交点。如我们在接下来的方程中所看到的，我们需要像处理直线那样计算交点，然后确保该点位于两个段内部。

在右上角的情况中，两个线段重叠，因此有不止一个交点——准确地说，是无限多个交点。对于我们的分析，我们将定义两种可能的情况：线段要么有交点，要么根本不相交（我们不会考虑右上角的情况）。我们会忽略重叠的情况，因为我们的应用中不需要它，而且我们希望代码更加简化。

##### **重叠的线段**

如果我们包括线段重叠的情况，交点函数的返回对象可能是点或线段。返回不同对象类型的函数很难操作。一旦得到结果，我们还需要检查返回的对象类型，并采取相应的操作。可以如下实现：

```py
result = seg_a.intersection_with(seg_b)

if type(result) is Point:
    # intersection is a point
elif type(result) is Segment:
    # intersection is a segment
else:
    # no intersection
```

但是这段代码比较混乱。有更好的方式来处理这个逻辑，不过我们不讨论这些，因为对于我们来说，要么有交点，要么根本没有交点。这样会让我们的代码更加简洁，易于操作。

让我们看一下这个算法。

##### **算法**

让我们找出像[图 5-9](ch05.xhtml#ch5fig9)右下角那种情况的交点。假设我们有两条线段：

+   线段 1，起点 *S*[1] 和终点 *E*[1]

+   线段 2，起点 *S*[2] 和终点 *E*[2]

我们可以计算线段 1 上的每个点，记为 *P*[1]，使用如下表达式：

![Image](../images/f0118-01.jpg)

其中 *t*[1] 是从 0 到 1 的参数，![Image](../images/d1victorit.jpg) 是该线段的方向向量（不是单位向量）。同样，线段 2 如下：

![Image](../images/f0118-02.jpg)

为了找出交点，我们必须寻找一对 *t*[1] 和 *t*[2] 的值，使得 *P*[1](*t*[1]) = *P*[2](*t*[2])：

![Image](../images/f0119-01.jpg)

如果这两条线段相交，将这两个*t*参数值代入各自的线段表达式应能得到相同的点，即交点 *P*。让我们将表达式重写为向量形式：

![Image](../images/f0119-02.jpg)

我们可以使用这种形式来得到一个标量系统，包含两个方程和两个未知数，*t*[1] 和 *t*[2]：

![Image](../images/f0119-03.jpg)

我不打算详细介绍这些细节，直接给你结果，不过自己解这个系统来求解 *t*[1] 和 *t*[2] 可能是一个不错的练习。最终的 *t* 参数表达式如[方程 5.7](ch05.xhtml#ch05eqa07)和[5.8](ch05.xhtml#ch05eqa08)所示。

![Image](../images/05eqa07.jpg)![Image](../images/05eqa08.jpg)

这里，Δ*S*[*x*] = *S*[2*x*] *– S*[1*x*]，Δ*S*[*y*] = *S*[2*y*] *– S*[1*y*]，并且 ![Image](../images/f00119-p1.jpg)。请注意，如果线段平行，这些公式将产生 *∞* 的结果 ![Image](../images/f00119-p2.jpg)。我们不能尝试除以零，因为那会在我们的 Python 代码中引发异常，所以我们需要在尝试计算 *t*[1] 和 *t*[2] 的值之前先检测这种情况。

对于线段不平行的情况，当计算出这两个值后，我们有两种可能的结果：

+   值 *t*[1] 和 *t*[2] 都在范围 [0, 1] 内。交点属于两个线段。

+   *t*[1] 和 *t*[2] 中的一个或两个超出了范围 [0, 1]。交点位于至少一个线段之外。

现在我们准备将逻辑实现为算法。在你的 *segment.py* 文件中，按照 [列表 5-18](ch05.xhtml#ch5lis18) 中所示实现 intersection_with 方法。

```py
class Segment:
    --snip--

    def intersection_with(self, other):
        d1, d2 = self.direction_vector, other.direction_vector

        if d1.is_parallel_to(d2):
            return None

        cross_prod = d1.cross(d2)
        delta = other.start - self.start
        t1 = (delta.u * d2.v - delta.v * d2.u) / cross_prod
        t2 = (delta.u * d1.v - delta.v * d1.u) / cross_prod

        if tparam.is_valid(t1) and tparam.is_valid(t2):
            return self.point_at(t1)
        else:
            return None
```

*列表 5-18：两个线段的交点*

我们首先通过使用 Python 的多重赋值将两个线段的方向向量存储到变量 d1 和 d2 中。通过多重赋值，可以一次为多个变量赋值。接着，我们检查方向是否平行，如果平行，则返回 None。如果发现线段不平行，我们计算![Image](../images/f00120-p1.jpg) 和 Δ*S*并将其存储在变量 cross_prod 和 delta 中。借助这些值，我们再计算 *t*[1] 和 *t*[2]。如果这些值在其有效范围内，我们通过在当前线段对象（self）上调用 point_at 返回交点。请确保理解我们也可以使用 *t*[2] 来计算 *P* 并在另一个上调用 point_at，结果会是一样的。

**注意**

*与其他语言（如 Java 或 C# 中的* null*）类似，*None* 应谨慎使用。它应该用于那些拥有类似空值且是完全有效结果的情况。例如，在我们的*intersection_with*方法中，*None* 代表没有交点的情况。*

##### **线段交点单元测试**

随着我们书籍内容的推进以及代码变得更加复杂，测试这些代码片段将变得更加复杂。我们刚刚写的用于计算线段交点的方法有几个分支或路径可以执行。为了尽可能全面地进行单元测试，让我们列出我们希望覆盖的每一种情况（参见 [表 5-1](ch05.xhtml#ch5tab1)）。

**表 5-1：** 线段交点算法结果

| **线段方向** | t[1] | t[2] | **交点结果** |
| --- | --- | --- | --- |
| ![Image](../images/f00120-p2.jpg) | — | — | None |
| ![Image](../images/f00120-p3.jpg) | 超出范围 | 超出范围 | None |
| ![Image](../images/f00120-p3.jpg) | 在范围内 | 超出范围 | None |
| ![Image](../images/f00120-p3.jpg) | 超出范围 | 在范围内 | None |
| ![Image](../images/f00120-p3.jpg) | 在范围内 | 在范围内 | ![Image](../images/f00120-p4.jpg) |

我们将为 [表 5-1](ch05.xhtml#ch5tab1) 中的第一个和最后一个情况编写单元测试；其他三个留给你作为练习。在 *segment_test.py* 文件中，将 [列表 5-19](ch05.xhtml#ch5lis19) 中的测试包含到 TestSegment 类中。

```py
class TestSegment(unittest.TestCase):

    start = Point(400, 0)
    end = Point(0, 400)
    segment = Segment(start, end)

    --snip--

    def test_parallel_segments_no_intersection(self):
        other = Segment(Point(200, 0), Point(0, 200))
        actual = self.segment.intersection_with(other)
        self.assertIsNone(actual)

    def test_segments_intersection(self):
        other = Segment(Point(0, 0), Point(400, 400))
        expected = Point(200, 200)
        actual = self.segment.intersection_with(other)
        self.assertEqual(expected, actual)
```

*列表 5-19：测试线段交点*

在第一个测试中，我们构造了一个平行线段，并通过断言 assertIsNone 来验证这两个线段的交点为 None，assertIsNone 会检查传入的值是否为 None。在第二个测试中，我们构造了一个垂直于第一个线段的线段，并在交点 (200, 200) 与第一个线段相交，断言我们得到了该点作为结果。你可以通过点击 IDE 中的绿色播放按钮，或者在控制台中如下运行文件中的所有测试：

```py
$ python3 -m unittest geom2d/segment_test.py
```

你能想出其他三种情况所需的线段吗？

#### ***相等性和字符串表示***

就像我们在 Point 和 Vector 类中所做的那样，我们想要重载 == 运算符，以便 Python 能够理解具有相同起点和终点的两个线段在逻辑上是相等的，并且我们希望实现一个 `__str__` 方法，以便我们可以获得该线段的漂亮字符串表示。在 *segment.py* 文件中输入代码，[列表 5-20](ch05.xhtml#ch5lis20)。

```py
class Segment:
    --snip--

   def __eq__(self, other):
       if self is other:
           return True

       if not isinstance(other, Segment):
           return False

       return self.start == other.start \
              and self.end == other.end

   def __str__(self):
       return f'segment from {self.start} to {self.end}'
```

*列表 5-20：线段的相等性和字符串表示*

一旦我们开发了 Line 类，我们将添加最后一个属性。如果你跟随代码，您的 Segment 类应该类似于 [列表 5-21](ch05.xhtml#ch5lis21)。

```py
from geom2d import tparam
from geom2d.point import Point
from geom2d.vectors import make_vector_between, make_versor_between

class Segment:
    def __init__(self, start: Point, end: Point):
        self.start = start
        self.end = end

    @property
    def direction_vector(self):
        return make_vector_between(self.start, self.end)

    @property
    def direction_versor(self):
        return make_versor_between(self.start, self.end)

    @property
    def normal_versor(self):
        return self.direction_versor.perpendicular()

    @property
    def length(self):
        return self.start.distance_to(self.end)

    def point_at(self, t: float):
        tparam.ensure_valid(t)
        return self.start.displaced(self.direction_vector, t)

    @property
    def middle(self):
        return self.point_at(tparam.MIDDLE)

    def closest_point_to(self, p: Point):
        v = make_vector_between(self.start, p)
        d = self.direction_versor
        vs = v.projection_over(d)

        if vs < 0:
            return self.start

        if vs > self.length:
            return self.end

        return self.start.displaced(d, vs)

    def distance_to(self, p: Point):
        return p.distance_to(
            self.closest_point_to(p)
        )

    def intersection_with(self, other):
        d1, d2 = self.direction_vector, other.direction_vector

        if d1.is_parallel_to(d2):
            return None

        cross_prod = d1.cross(d2)
        delta = other.start - self.start
        t1 = (delta.u * d2.v - delta.v * d2.u) / cross_prod
        t2 = (delta.u * d1.v - delta.v * d1.u) / cross_prod

        if tparam.is_valid(t1) and tparam.is_valid(t2):
            return self.point_at(t1)
        else:
            return None

    def __eq__(self, other):
        if self is other:
            return True

        if not isinstance(other, Segment):
            return False

        return self.start == other.start \
               and self.end == other.end

    def __str__(self):
        return f'segment from {self.start} to {self.end}'
```

*列表 5-21：线段类*

### **Line 类**

一条无限长的线可以通过一个基准点 *B* 和一个方向向量 ![图像](../images/dvictorit.jpg) 来描述，正如 [图 5-10](ch05.xhtml#ch5fig10) 所示。

![图像](../images/05fig10.jpg)

*图 5-10：带有基准点 B 和方向向量的线 ![图像](../images/dvictor_normal.jpg)*

线是有用的辅助原始数据；通过它们我们可以构建更复杂的几何体和操作。例如，线的一个常见用途是找到两条不平行方向的交点。你将在下一章看到，利用线的交点，像通过三点构建一个圆这样的操作变得轻而易举。

让我们创建一个新的 Line 类，包含这两个属性：基准点和方向。在 *geom2d* 包中，添加一个名为 *line.py* 的新文件，并在其中输入 [列表 5-22](ch05.xhtml#ch5lis22) 中的代码。

```py
from geom2d.point import Point
from geom2d.vector import Vector

class Line:
    def __init__(self, base: Point, direction: Vector):
        self.base = base
        self.direction = direction
```

*列表 5-22：线初始化*

初始化器根据传入的相应参数值设置我们的基准点和方向属性。像之前一样，我们已经为基准点和方向参数添加了类型注解，以便我们的 IDE 能警告我们潜在的错误。

现在让我们提供两个方法，检查一条线是否平行或垂直于另一条线（[列表 5-23](ch05.xhtml#ch5lis23)）。

```py
class Line:
   --snip--

   def is_parallel_to(self, other):
       return self.direction.is_parallel_to(other.direction)

   def is_perpendicular_to(self, other):
       return self.direction.is_perpendicular_to(other.direction)
```

*列表 5-23：检查线条是否平行或垂直*

我们没有为 Segment 实现这些方法，因为我们关心的是线段的无限点及其在平面上的位置；而在这里，我们处理的是方向。处理方向需要了解它们的相对位置：它们是平行的吗？它们是垂直的吗？

对于线，问题通常是它们相对于其他线的位置；而对于线段，问题通常是它们自身的位置。

要检查两条直线是否平行，我们可以简单地访问它们的方向属性，并像这样使用它们的方法：

```py
d1 = line_one.direction
d2 = line_two.direction
d1.is_parallel_to(d2)
```

这确实是可能的，但通常不被认为是最佳做法。常有一种指导原则，称为*最小知识原则*或*德梅特法则*，它指出：“你应该只与你的直接朋友交谈。”在这种情况下，由于我们正在处理直线，直线是我们的直接朋友。直线的基准点和方向向量不是我们的直接朋友；因此，我们不应该向它们索取信息。如果我们需要它们的某些内容，我们必须要求我们的直接朋友——包含这些属性的直线——为我们做。

所以，我们应该如何检查两条直线是否平行或垂直呢：

```py
line_one.is_parallel_to(line_two)
```

让我们再增加两个方法，用于创建与现有直线垂直或平行并通过某个点的新直线。在你的文件中，输入[示例 5-24](ch05.xhtml#ch5lis24)中的代码。

```py
from geom2d.point import Point
from geom2d.vector import Vector

class Line:
    --snip--

    def perpendicular_through(self, point: Point):
        return Line(point, self.direction.perpendicular())

    def parallel_through(self, point: Point):
        return Line(point, self.direction)
```

*示例 5-24：创建垂直和平行的直线*

方法perpendicular_through接收一个点作为参数，并返回一条新直线，该直线使用该基准点和与原始直线垂直的方向向量。类似地，parallel_through构造一条新的直线，该直线具有给定的基准点，但使用与原始直线相同的方向向量。

#### ***直线交点***

本章早些时候已详细解释了计算两条线段交点的一般算法。该算法基于线段的起点和方向向量，但可以通过使用直线的基准点而不是线段的起点，扩展为适用于直线。值得注意的是，在直线的情况下，参数*t*[1]和*t*[2]不再局限于[0, 1]范围；它们可以从*– ∞*到*∞*。

如果我们将[方程 5.7](ch05.xhtml#ch05eqa07)和[5.8](ch05.xhtml#ch05eqa08)重写为直线方程，我们得到[方程 5.9](ch05.xhtml#ch05eqa09)和[5.10](ch05.xhtml#ch05eqa10)。

![Image](../images/05eqa09.jpg)![Image](../images/05eqa10.jpg)

在这种情况下，Δ*B*[*x*] = *B*[2*x*] *– B*[1*x*]，Δ*B*[*y*] = *B*[2*y*] *– B*[1*y*]。为了使这些公式得出正确的值，请回忆一下![Image](../images/f00119-p1.jpg)。由于*t*值不再受限制，因此无需计算*t*[1]和*t*[2]并检查它们是否落在[0, 1]范围内。计算其中一个值就足以得到交点。让我们选择[方程 5.9](ch05.xhtml#ch05eqa09)来计算*t*[1]。有了*t*[1]，我们可以按如下方式确定实际的交点：

![Image](../images/f00126-p1.jpg)

在你的Line类中实现intersection_with方法，如[示例 5-25](ch05.xhtml#ch5lis25)所示。

```py
from geom2d.point import Point
from geom2d.vector import Vector
from geom2d.vectors import make_vector_between

class Line:
   --snip--

   def intersection_with(self, other):
       if self.is_parallel_to(other):
           return None

       d1, d2 = self.direction, other.direction
       cross_prod = d1.cross(d2)
       delta = make_vector_between(self.base, other.base)
       t1 = (delta.u * d2.v - delta.v * d2.u) / cross_prod

       return self.base.displaced(d1, t1)
```

*示例 5-25：计算两条直线的交点*

代码与 Segment 中的算法类似，但更简单一些。为了检查平行性，我们使用 self 方法，而不是使用方向。由于我们在 Line 类中实现了 is_parallel_to 方法，使用它更合适（并且使代码更加易读！）。

##### **单元测试线段交点**

让我们确保我们修改后的算法有效。创建一个新的文件 *line_test.py* 并输入 [列表 5-26](ch05.xhtml#ch5lis26) 中的 Line 类测试代码。

```py
import unittest

from geom2d.line import Line
from geom2d.point import Point
from geom2d.vector import Vector

class TestLine(unittest.TestCase):

    def test_parallel_lines_no_intersection(self):
        l1 = Line(Point(0, 0), Vector(1, 1))
        l2 = Line(Point(10, 10), Vector(1, 1))
        self.assertIsNone(l1.intersection_with(l2))

    def test_lines_intersection(self):
        l1 = Line(Point(50, 0), Vector(0, 1))
        l2 = Line(Point(0, 30), Vector(1, 0))
        actual = l1.intersection_with(l2)
        expected = Point(50, 30)
        self.assertEqual(expected, actual)
```

*列表 5-26：测试线段交点*

在第一个测试中，test_parallel_lines_no_intersection，我们创建了两条平行线，它们有不同的基点，但方向向量相同。然后我们断言 intersection_with 返回 None。第二个测试 test_lines_intersection，创建了两条线，第一条在 *x* = 50 处垂直，第二条在 *y* = 30 处水平；因此，它们的交点是 (50, 30)。

通过点击类定义旁边的绿色播放按钮来运行测试。你应该在控制台中看到如下内容：

```py
Ran 2 tests in 0.001s

OK

Process finished with exit code 0
```

你也可以通过控制台运行测试：

```py
$  python3 -m unittest geom2d/line_test.py
```

[列表 5-27](ch05.xhtml#ch5lis27) 包含了我们为 Line 类编写的所有代码。

```py
from geom2d.point import Point
from geom2d.vector import Vector
from geom2d.vectors import make_vector_between

class Line:
    def __init__(self, base: Point, direction: Vector):
        self.base = base
        self.direction = direction

    def is_parallel_to(self, other):
        return self.direction.is_parallel_to(other.direction)

    def is_perpendicular_to(self, other):
        return self.direction.is_perpendicular_to(other.direction)

    def perpendicular_through(self, point: Point):
        return Line(point, self.direction.perpendicular())

    def parallel_through(self, point: Point):
        return Line(point, self.direction)

    def intersection_with(self, other):
        if self.is_parallel_to(other):
            return None

        d1, d2 = self.direction, other.direction
        cross_prod = d1.cross(d2)
        delta = make_vector_between(self.base, other.base)
        t1 = (delta.u * d2.v - delta.v * d2.u) / cross_prod

        return self.base.displaced(d1, t1)
```

*列表 5-27：Line 类*

### **Segment 的平分线**

现在我们有了 Segment 和 Line，我们可以在 Segment 中实现一个新的属性：它的 *平分线*。这个属性是穿过 Segment 中点 *M* 且与之垂直的线。[图 5-11](ch05.xhtml#ch5fig11) 展示了这一概念。

![图片](../images/05fig11.jpg)

*图 5-11：Segment 的平分线*

计算 Segment 的平分线很简单，因为我们已经可以访问到 Segment 的中点和法向量（别忘了导入 Line 类），如 [列表 5-28](ch05.xhtml#ch5lis28) 所示。

```py
from geom2d import tparam
from geom2d.line import Line
from geom2d.point import Point
from geom2d.vectors import make_vector_between, make_versor_between

class Segment:
    --snip--

    @property
   def bisector(self):
       return Line(self.middle, self.normal_versor)
```

*列表 5-28：Segment 的平分线*

在下一章，我们将使用 Segment 的平分线来创建一个通过三点的圆——这是 CAD 软件中常用的一种求圆方法。在本书的[第三部分](part03.xhtml#part03)，我们将创建一个程序，计算通过三点的圆并绘制一个美丽的图像，图中会标注圆心和半径。

### **总结**

在本章中，我们使用了 Point 和 Vector 类来创建两个新的原始类型：Segment 和 Line。它们都有一个定义好的方向，且都表示一组无限对齐的点，但 Segment 是在两个点之间有限制的，而 Line 没有尽头。

我们还实现了一种方法，通过一个参数 *t* 来获取 Segment 中的无限点，*t* 的取值范围为 [0, 1]。对 Line 来说没有必要做相同的操作，因为我们通常不关心构成它的点。

然后，我们创建了两个算法：我们在 Segment 类中添加了一个方法，用于查找其与外部点的最近点。尽管我们没有在 Line 类中实现该方法，但我们本来可以实现它。我们利用这个方法来计算点到线段的距离。我们还实现了一个算法，用于计算两个线段和两条直线的交点。这些交点的结果是一个点，或者返回值为 None。最后，我们使用 Line 类表示线段的平分线。

这些线性原语对于构建更复杂的几何图形——多边形，将证明是无价的，这也是我们下一章的主题。
