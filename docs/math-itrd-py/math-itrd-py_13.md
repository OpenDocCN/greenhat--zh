## 10

使用递归创建分形

*“thesaurus”还有另一个词是什么？*

—史蒂文·赖特*

![image](../images/fintro-01.jpg)

分形是一种令人愉悦的复杂设计，其中每个设计的较小部分都包含整个设计（见[图10-1](ch10.xhtml#ch10fig1)）。它们是由Benoit Mandelbrot在1980年发明的（或者说是发现的，因为分形在自然界中就存在），当时他正在用一台先进的IBM计算机可视化一些复杂的函数。

![image](../images/f201-01.jpg)

*图10-1：分形的例子*

分形的形状看起来与我们在几何学中认知的规则形状不同，比如正方形、三角形和圆形。它们的形状曲折不规则，使它们成为模拟自然现象的极好模型。实际上，科学家们使用分形来模拟从心脏动脉到地震，再到大脑中的神经元等一切事物。

分形之所以如此有趣，是因为它们展示了如何通过简单的规则不断运行和在更小的尺度上重复图案，从而产生出令人惊讶的复杂设计。

我们的主要兴趣是通过分形可以制作出有趣且复杂的设计。如今，每本数学书中都有分形的图片，但教科书从未告诉你如何制作分形——你需要用计算机来实现。在本章中，你将学习如何使用Python制作你自己的分形。

### 海岸线的长度

在你开始创建分形之前，让我们看一个简单的例子，来理解分形如何有用。一位数学家名叫Lewis Richardson提出了一个简单的问题：“英格兰的海岸线有多长？”正如你在[图10-2](ch10.xhtml#ch10fig2)中看到的，答案取决于你标尺的长度。

![image](../images/f202-01.jpg)

*图10-2：近似海岸线长度*

你的标尺越小，你就能越接近逼近海岸线的曲折边缘，这意味着你最终会得到一个更长的测量值。更酷的是，*当标尺长度接近零时，海岸线的长度趋向于无限大！* 这被称为海岸线悖论。

觉得这只是抽象的数学涂鸦吗？在现实世界中，海岸线长度的估算可以有很大的差异。即使是现代技术，也完全取决于用来测量地图的尺度。我们将绘制一个类似于[图10-3](ch10.xhtml#ch10fig3)的图形——科赫雪花，展示分形如何证明即使是一个粗略的海岸线，也可以延长到你想要的任何长度！

![image](../images/f203-01.jpg)

*图10-3：一个越来越详细的分形，模拟越来越粗糙的海岸线*

首先，你需要学习一些技巧，比如递归。

#### 什么是递归？

分形的强大之处在于，你可以重复缩小的数字或形状的模式，直到你处理的是非常小的数字。重复所有这些代码的关键是一个叫做*递归*的概念，递归是指某个事物通过自身来定义。以下一些笑话展示了递归是如何工作的：

+   如果你在谷歌搜索“递归”，它会问你：“你是想搜索 *recursion* 吗？”

+   在很多计算机编程书籍的索引中，会看到类似这样的条目：“递归，参见 *recursion*。”

正如你所想，递归是一个相当奇怪的概念。递归的优点是它可以简化本来可能过于复杂的代码，但缺点是它可能会耗费太多内存。

#### 编写 factorial() 函数

让我们通过编写一个计算数字阶乘的函数来看递归的实际应用。你可能还记得数学课上，*n* 的阶乘（表示为 *n!*）是从 1 到 *n* 的所有整数的乘积。例如，5! = 1 × 2 × 3 × 4 × 5 = 120。

这个公式看起来是这样的：*n*! = 1 × 2 × 3 . . . × (*n* – 2) × (*n* – 1) × *n*。这是递归序列的一个例子，因为 5! = 5 × 4!，而 4! = 4 × 3!，依此类推。递归是数学中的一个重要概念，因为数学的核心就是模式，而递归让你能够无限复制和扩展这些模式！

我们可以将 *n* 的阶乘定义为 *n* 与 *n* – 1 的阶乘的乘积。我们只需要定义 0 的阶乘（为 1，而不是 0）和 1 的阶乘，然后使用递归语句。打开 IDLE 中的新文件，将其保存为 *factorial.py*，然后输入 [清单 10-1](ch10.xhtml#ch10list1) 中的代码。

*factorial.py*

def factorial(n):

if n == 0:

return 1

else:

return n * factorial(n – 1)

*清单 10-1：使用递归语句编写 factorial() 函数*

首先，我们在这里说的是，“如果用户（或程序）请求计算 0 或 1 的阶乘，返回 1。”这是因为 0! 和 1! 都等于 1。接着我们告诉程序，“对于任何其他的数字 *n*，返回 *n* 乘以 *n* 减去 1 的阶乘。”

请注意，在 [清单 10-1](ch10.xhtml#ch10list1) 的最后一行，我们在调用 factorial() 函数时，是在 factorial() 函数的定义内部进行调用！这就像一份面包制作食谱，其中写着“烤一块面包”。人们根本不会开始按照这样的食谱制作面包。但是计算机可以从头到尾按照步骤执行。

在这个例子中，当我们请求 5 的阶乘时，程序会顺从地执行，直到最后一行，它会请求 *n* – 1 的阶乘，在这个例子中（因为 *n* = 5），就是 4 的阶乘。为了计算 factorial(5 – 1)，程序会再次启动 factorial() 函数，将 *n* 设置为 4，并尝试以相同的方式计算 4 的阶乘，接着是 3 的阶乘、2 的阶乘、1 的阶乘，最后是 0 的阶乘。因为我们已经定义了 0 的阶乘为 1，所以函数可以沿着这个过程返回，计算 1 的阶乘，再是 2 的阶乘、3 的阶乘、4 的阶乘，最后是 5 的阶乘。

递归定义一个函数（在函数内部调用自身）可能看起来有些困惑，但这是本章中所有分形图形的关键。我们从一个经典例子开始：分形树。

#### 构建分形树

创建分形图形从定义一个简单的函数开始，并在函数内部添加对该函数的调用。让我们尝试创建一个看起来像[图 10-4](ch10.xhtml#ch10fig4)的分形树。

![image](../images/f204-01.jpg)

*图 10-4：一棵分形树*

如果你必须告诉程序每一条线如何绘制，这将是一个非常复杂的设计。但如果使用递归，代码量惊人地少。通过使用平移、旋转和 line() 函数，我们首先会在 Processing 中绘制一个 Y 形状，正如[图 10-5](ch10.xhtml#ch10fig5)所示。

![image](../images/f205-01.jpg)

*图 10-5：分形树的初步形态*

要使这个 Y 最终成为一个分形图形，唯一的要求是，在程序绘制完 Y 树及其分支后，程序必须返回到“树干”的底部。这是因为“分支”将会变成 Y 形状。如果程序每次都不返回到 Y 的底部，我们就无法得到树形。

##### 编写 y() 函数

你的 Y 不必是完美或对称的，但这是我绘制 Y 形状的代码。在 Processing 中打开一个新草图，将其命名为 *fractals.pyde*，并输入[清单 10-2](ch10.xhtml#ch10list2)中的代码。

*fractals.pyde*

def setup():

size(600,600)

def draw():

background(255)

translate(300,500)

y(100)

def y(sz):

line(0,0,0,-sz)

translate(0,-sz)

rotate(radians(30))

line(0,0,0,-0.8*sz) #右侧分支

rotate(radians(-60))

line(0,0,0,-0.8*sz) #左侧分支

rotate(radians(30))

translate(0,sz)

*清单 10-2：为分形树编写 y() 函数*

我们像往常一样设置 Processing 草图：在 setup() 函数中，我们告诉程序设置显示窗口的大小，然后在 draw() 函数中，我们设置背景颜色（255 是白色）并平移到我们想要开始绘制的位置。最后，我们调用 y() 函数，并传递数字 100 作为分形树“树干”的大小。

y() 函数接受一个数字 sz 作为参数，表示树干的长度。然后所有的分支都基于这个数字。y() 函数中的第一行代码通过一条垂直线绘制树干。为了创建一个向右分叉的线，我们将垂直线沿树干（沿负 y 方向）平移，并将其旋转 30 度到右边。接下来，我们绘制另一条右分支线，向左旋转（负 60 度），并绘制另一条左分支线。最后，我们必须旋转以使我们再次面朝上，这样才能再次沿着树干向下平移。保存并运行这个草图，你应该会看到[图 10-5](ch10.xhtml#ch10fig5)中的 Y 形状。

我们可以将这个绘制单个 Y 的程序改造为绘制分形，通过将分支变成 *更小* 的 Y。但是，如果我们在 y() 函数中简单地将 “line” 替换为 “y”，程序就会陷入无限循环，并抛出如下错误：

RuntimeError: 超过最大递归深度

回想一下，我们没有在 factorial 函数内部调用 factorial(n)，而是调用了 factorial(n-1)。我们需要在 y() 函数中引入一个 level 参数。然后，每次分支向上，树的级别会减小，因此分支会获得参数 level - 1。这意味着树干始终是最高级别，最后一组分支总是第 0 级。以下是如何修改 [清单 10-3](ch10.xhtml#ch10list3) 中的 y() 函数。

*fractals.pyde*

def setup():

size(600,600)

def draw():

background(255)

translate(300,500)

y(100,2)

def y(sz,level):

if level > 0:

line(0,0,0,-sz)

translate(0,-sz)

rotate(radians(30))

y(0.8*sz,level-1)

rotate(radians(-60))

y(0.8*sz,level-1)

rotate(radians(30))

translate(0,sz)

*清单 10-3：向 y() 函数添加递归*

请注意，我们将代码中的所有 line() 函数替换为 y() 函数来绘制分支。由于我们将 draw() 中对 y() 函数的调用改为 y(100,2)，我们将得到一个树干大小为 100 且有两个级别的树。尝试三层树、四层树，等等！你应该会看到像 [图 10-6](ch10.xhtml#ch10fig6) 这样的效果。

![image](../images/f207-01.jpg)

*图 10-6：1 到 4 级的树木*

##### 映射鼠标

现在，让我们制作一个程序，允许你通过上下移动鼠标来实时控制分形的形状！我们可以通过跟踪鼠标并根据其位置返回一个介于 0 到 10 之间的值来动态地改变旋转的级别。使用 [清单 10-4](ch10.xhtml#ch10list4) 中的代码更新 draw() 函数。

*fractals.pyde*

def draw():

background(255)

translate(300,500)

level = int(map(mouseX,0,width,0,10))

y(100,level)

*清单 10-4：向 draw() 函数添加 level 参数*

我们鼠标的 x 值可以是从 0 到窗口宽度之间的任何值。map() 函数将一个值范围转换为另一个范围。在 [清单 10-4](ch10.xhtml#ch10list4) 中，map() 将取 x 值，并将其输出范围从 0 到 600（显示屏的宽度）转换为 0 到 10，这正是我们想要绘制的级别范围。所以我们将该值赋给一个名为 level 的变量，并将其传递给 y() 函数的下一行。

现在我们已经调整了 draw() 函数，使其根据鼠标的位置返回一个值，我们可以通过将鼠标的 y 坐标与旋转角度关联，来改变树的形状。

旋转角度应该最大为180度，因为树木在180度时会“完全折叠”，但是鼠标的y值可以达到600，因为我们在setup()中声明了屏幕的高度。我们本可以自己做一些数学计算来转换这些值，但使用Processing的内置map()函数会更简单。我们告诉map()函数我们想要映射的变量，指定其当前的最小值和最大值以及所需的最小值和最大值。Y分形树的完整代码见[列表 10-5](ch10.xhtml#ch10list5)。

*fractals.pyde*

def setup():

size(600,600)

def draw():

background(255)

translate(300,500)

level = int(map(mouseX,0,width,0,15))

y(100,level)

def y(sz,level):

if level > 0:

line(0,0,0,-sz)

translate(0,-sz)

angle = map(mouseY,0,height,0,180)

rotate(radians(angle))

y(0.8*sz,level-1)

rotate(radians(-2*angle))

y(0.8*sz,level-1)

rotate(radians(angle))

translate(0,sz)

*列表 10-5：制作动态分形树的完整代码*

我们取鼠标的y值，并将其转换为0到180之间的范围（如果你已经习惯用弧度思考，可以将其映射到0到pi之间）。在rotate()语句中，我们传入这个角度（单位为度），并让Processing将度数转换为弧度。第一条rotate()语句将旋转到右侧。第二条rotate()语句将旋转一个负角度，意味着向左旋转。它会向左旋转两倍的角度。然后，第三条rotate()语句将再次向右旋转。

当你运行代码时，你应该看到类似于[图 10-7](ch10.xhtml#ch10fig7)的效果。

![image](../images/f208-01.jpg)

*图 10-7：动态分形树*

现在，当你上下、左右移动鼠标时，分形的级别和形状应该相应变化。

通过绘制分形树，你学会了如何使用递归在非常少量的代码中绘制复杂的设计。现在我们将回到海岸线问题。如何通过增加锯齿状的曲折，使得海岸线或任何线条的长度翻倍或三倍？

### 科赫雪花

科赫雪花是一个著名的分形图形，得名于瑞典数学家Helge von Koch，他在1904年的一篇论文中描述了这个形状！它是由一个等边三角形构成的。我们从一条线开始，然后在其上添加一个“凸起”。接着，我们为每个结果线段添加一个更小的凸起并重复这个过程，就像[图 10-8](ch10.xhtml#ch10fig8)中展示的那样。

![image](../images/f209-01.jpg)

*图 10-8：给每个线段添加一个“凸起”*

让我们开始一个新的Processing草图，命名为*snowflake.pyde*，并添加[列表 10-6](ch10.xhtml#ch10list6)中的代码，这将绘制一个倒立的等边三角形。

*snowflake.pyde*

def setup():

size(600,600)

def draw():

background(255)

translate(100,100)

snowflake(400,1)

def snowflake(sz,level):

for i in range(3):

line(0,0,sz,0)

translate(sz,0)

rotate(radians(120))

*列表 10-6：编写snowflake()函数*

在draw()函数中，我们调用snowflake()函数，该函数目前只接受两个参数：sz（三角形的初始大小）和level（分形的级别）。snowflake()函数通过开始一个循环绘制一个三角形，该循环重复执行三次代码。在循环内部，我们绘制长度为sz的线段，这将是三角形的边，然后沿着该线段平移到三角形的下一个顶点并旋转120度。然后我们绘制三角形的下一条边。

当你运行[清单10-6](ch10.xhtml#ch10list6)中的代码时，你应该看到[图10-9](ch10.xhtml#ch10fig9)。

![image](../images/f210-01.jpg)

*图10-9：级别1的雪花：一个三角形*

#### 编写SEGMENT()函数

现在我们需要告诉程序如何将一条线变成一个具有不同级别的线段。级别0只是一条直线，但下一个级别会在边上添加“凸起”。我们实际上是将线段分成三段，然后将中间段复制，使其成为一个小等边三角形。我们将修改snowflake()函数，调用另一个函数来绘制这段线段。这将是递归函数，因为随着级别的提高，线段将变成[图10-10](ch10.xhtml#ch10fig10)中线段的更小副本。

![image](../images/f210-02.jpg)

*图10-10：将一段线切成三等分，并在中间三分之一处添加一个“凸起”*

我们将边称为*线段*。如果级别为0，则线段只是直线，即三角形的边。在下一步中，边的中间会添加一个凸起。[图10-10](ch10.xhtml#ch10fig10)中的所有线段长度相同，都是整个边长的三分之一。这个过程需要11步：

1.  绘制三分之一的边长。

1.  翻译到你刚刚绘制的线段的末尾。

1.  旋转-60度（向左）。

1.  绘制另一段线段。

1.  翻译到该段落的末尾。

1.  旋转120度（向右）。

1.  绘制第三段线段。

1.  翻译到该段落的末尾。

1.  再次旋转-60度（向左）。

1.  绘制最后一段。

1.  翻译到该段落的末尾。

现在，snowflake()函数不再绘制一条线，而是调用segment()函数，这个函数将进行绘制和平移。请在[清单10-7](ch10.xhtml#ch10list7)中添加segment()函数。

*snowflake.pyde*

def snowflake(sz,level):

for i in range(3):

segment(sz,level)

rotate(radians(120))

def segment(sz,level):

如果级别 == 0:

line(0,0,sz,0)

translate(sz,0)

else:

line(0,0,sz/3.0,0)

translate(sz/3.0,0)

rotate(radians(-60))

line(0,0,sz/3.0,0)

translate(sz/3.0,0)

rotate(radians(120))

line(0,0,sz/3.0,0)

translate(sz/3.0,0)

rotate(radians(-60))

line(0,0,sz/3.0,0)

translate(sz/3.0,0)

*清单10-7：在三角形的边上绘制一个“凸起”*

在`segment()`函数中，如果级别为0，它只是画一条直线，并且我们将其平移到该线的末端。否则，我们有11行代码对应于制作“凸起”的11个步骤。首先，我们画出三角形边长的三分之一的线段，然后将其平移到该线段的末端。我们向左旋转（–60度）画第二段线。该段的长度也是三角形边长的三分之一。我们将平移到该段的末端，然后向右旋转120度。接着我们画一段线，并最后一次通过旋转–60度向左转。最后，我们画第四条线段，并将其平移到边的末端。

如果级别为0，则画出一个三角形，如果级别不为0，则在每条边上添加一个凸起。正如你在[图 10-8](ch10.xhtml#ch10fig8)中看到的那样，在每一步，前一步中的每个段落都会得到一个凸起。没有递归的话，做这个会让人头疼！但是我们会将绘制直线的代码行改成一个线段，只是级别低一个级别。这就是递归步骤。

接下来，我们需要将每条线替换为一个级别较低的线段，长度为sz除以3。`segment()`函数的代码见[列表 10-8](ch10.xhtml#ch10list8)。

*snowflake.pyde*

def segment(sz,level):

if level == 0:

line(0,0,sz,0)

translate(sz,0)

else:

segment(sz/3.0,level-1)

rotate(radians(-60))

segment(sz/3.0,level-1)

rotate(radians(120))

segment(sz/3.0,level-1)

rotate(radians(-60))

segment(sz/3.0,level-1)

*列表 10-8: 将线替换为线段*

所以我们所做的就是将[列表 10-7](ch10.xhtml#ch10list7)中每个line的实例（其级别大于0）替换为segment()。因为我们不想进入无限循环，段落必须比前一段低一个级别（level – 1）。现在我们可以在`draw()`函数中更改雪花的级别，如下所示的代码所示，并且我们将看到不同的设计，正如[图 10-11](ch10.xhtml#ch10fig11)所示。

def draw():

background(255)

translate(100,height-100)

snowflake(400,3)

![image](../images/f212--01.jpg)

*图 10-11: 三级雪花*

更好的是，我们可以通过将鼠标的x值映射到级别来使其具有交互性。鼠标的x值可以从0到屏幕的宽度。我们想要将这个范围更改为0到7之间。以下是实现这一点的代码：

level = map(mouseX,0,width,0,7)

然而，我们只想要整数级别，所以我们将使用`int`将该值转换为整数，像这样：

level = int(map(mouseX,0,width,0,7))

我们将其添加到我们的`draw()`函数中，并将输出的“level”传递给`snowflake()`函数。科赫雪花的完整代码见[列表 10-9](ch10.xhtml#ch10list9)。

*snowflake.pyde*

def setup():

size(600,600)

def draw():

background(255)

translate(100,200)

level = int(map(mouseX,0,width,0,7))

#y(100,level)

snowflake(400,level)

def snowflake(sz,level):

for i in range(3):

segment(sz,level)

rotate(radians(120))

def segment(sz,level):

if level == 0:

line(0,0,sz,0)

translate(sz,0)

else:

segment(sz/3.0,level-1)

rotate(radians(-60))

segment(sz/3.0,level-1)

rotate(radians(120))

segment(sz/3.0,level-1)

rotate(radians(-60))

segment(sz/3.0,level-1)

*列表 10-9：Koch 雪花的完整代码*

现在，当你运行程序并左右移动鼠标时，你会看到雪花的各个段落上出现更多的“凸起”，就像在[图 10-12](ch10.xhtml#ch10fig12)中所示。

![image](../images/f213-01.jpg)

*图 10-12：7级雪花*

这如何帮助我们理解海岸线悖论呢？回顾[图 10-3](ch10.xhtml#ch10fig3)，我们把线段的长度（即三角形的一边）设为 1 个单位（例如 1 英里）。当我们将其分为三等份，去掉中间部分，并在中间加上一个长度为三分之二的“凸起”时，边长变成了 1 1/3 个单位。它刚刚变长了三分之一，对吧？雪花的周长（即“海岸线”）每一步都变长三分之一。因此，在第 *n* 步时，海岸线的长度是原始三角形周长的 (4/3)^(*n*) 倍。虽然可能不容易察觉，但经过 20 步后，雪花的海岸线已经如此锯齿状，以至于其总长度是原始测量值的 300 多倍！

### SIERPINSKI 三角形

Sierpinski 三角形是一个著名的分形，最早由波兰数学家 Wacław Sierpiński 在 1915 年描述，但在意大利的教堂地板上，从 11 世纪起就有这种设计的例子！它遵循一种几何模式，描述起来很简单，但设计却出奇复杂。它基于一个有趣的递归思想：在第一层画一个三角形，在下一层将每个三角形的角落转变为三个更小的三角形，如[图 10-13](ch10.xhtml#ch10fig13)所示。

![image](../images/f214-01.jpg)

*图 10-13：Sierpinski 三角形，第 0、1 和 2 级*

第一步很简单：只需要画一个三角形。打开一个新的草图并命名为 *sierpinski.pyde*。我们像往常一样设置它，包括 setup() 和 draw() 函数。在 setup() 中，我们将输出窗口的大小设置为 600 像素 × 600 像素。在 draw() 中，我们设置背景为白色，并将坐标系平移到屏幕左下角的 (50,450) 点开始绘制三角形。接下来，我们写一个名为 sierpinski() 的函数，类似于我们在 tree() 中做的，如果级别是 0，就绘制一个三角形。目前为止的代码如[列表 10-10](ch10.xhtml#ch10list10)所示。

*sierpinski.pyde*

def setup():

size(600,600)

def draw():

background(255)

translate(50,450)

sierpinski(400,0)

def sierpinski(sz, level):

if level == 0: #画一个黑色三角形

fill(0)

triangle(0,0,sz,0,sz/2.0,-sz*sqrt(3)/2.0)

*列表 10-10：Sierpinski 分形的设置*

sierpinski() 函数接受两个参数：图形的大小 (sz) 和级别 (level) 变量。填充颜色为 0，表示黑色，但你可以使用 RGB 值来设置为任何颜色。三角形的线包含六个数字：等边三角形三个角的 x 和 y 坐标，边长为 sz。

正如你在 [Figure 10-13](ch10.xhtml#ch10fig13) 中看到的，级别 1 包含了原三角形每个角上的三个小三角形。这些三角形的大小是上一级三角形的一半。我们将做的是创建一个较小的、较低级别的谢尔宾斯基三角形，平移到下一个角，并旋转 120 度。将 [Listing 10-11](ch10.xhtml#ch10list11) 中的代码添加到 sierpinski() 函数中。

def draw():

background(255)

translate(50,450)

sierpinski(400,8)

def sierpinski(sz, level):

if level == 0: # 绘制一个黑色三角形

fill(0)

triangle(0,0,sz,0,sz/2.0,-sz*sqrt(3)/2.0)

else: # 在每个顶点绘制谢尔宾斯基三角形

for i in range(3):

sierpinski(sz/2.0,level-1)

translate(sz/2.0,-sz*sqrt(3)/2.0)

rotate(radians(120))

*Listing 10-11: 为谢尔宾斯基程序添加递归步骤*

这段新代码告诉 Processing 在级别不为 0 时该做什么（i in range(3): 意味着“重复三次”）：绘制一个半尺寸的谢尔宾斯基三角形，级别比当前低一级，然后平移到等边三角形的中点并旋转 120 度。注意在 sierpinski(sz/2.0,level-1) 中执行了 sierpinski() 函数，这就是递归步骤！当你调用

sierpinski(400,8)

![image](../images/f215-01.jpg)

*Figure 10-14: 8 级谢尔宾斯基三角形*

在 draw() 函数中，你将得到一个 8 级谢尔宾斯基三角形，正如 [Figure 10-14](ch10.xhtml#ch10fig14) 所示。

关于谢尔宾斯基三角形，一个有趣的现象是它也出现在其他分形中，像下一个分形，尽管它并不是以三角形为起点。

### 正方形分形

我们也可以用正方形来创建谢尔宾斯基三角形。例如，我们可以创建一个正方形，去掉右下角的四分之一部分，然后用得到的形状替换其余的四分之一部分。当我们重复这个过程时，应该会得到类似于 [Figure 10-15](ch10.xhtml#ch10fig15) 的图形。

![image](../images/f216--01.jpg)

*Figure 10-15: 正方形分形在级别 0、1、2 和 3 的表现*

为了创建这个分形，我们需要将三个较小的正方形每个都变成整个图形的一个副本。启动一个新的 Processing 草图，命名为 *squareFractal.pyde*，然后用 [Listing 10-12](ch10.xhtml#ch10list12) 中的代码设置草图。

*squareFractal.pyde*

def setup():

size(600,600)

fill(150,0,150) # 紫色

noStroke()

def draw():

background(255)

translate(50,50)

squareFractal(500,0)

def squareFractal(sz,level):

if level == 0:

rect(0,0,sz,sz)

*Listing 10-12: 创建 squareFractal() 函数*

![image](../images/f216--02.jpg)

*Figure 10-16: 紫色正方形（级别 0）*

我们可以在setup()函数中使用紫色的RGB值，因为我们在其他地方不会更改填充颜色。我们使用noStroke()，这样就看不见方块的黑色轮廓。在draw()函数中，我们调用squareFractal()函数，告诉它将每个方块的大小设置为500像素，层级为0。在函数定义中，我们告诉程序如果层级为零，就简单地绘制一个方块。这应该会给我们一个漂亮的大紫色方块，如[图10-16](ch10.xhtml#ch10fig16)所示。

对于下一层，我们将创建初始方块边长的一半的方块。其中一个将位于图形的左上角；然后我们将进行平移，将另外两个方块放在左下角和右上角，如[图10-16](ch10.xhtml#ch10fig16)所示。[示例10-13](ch10.xhtml#ch10list13)执行了这个操作，同时将大方块的四分之一去掉。

*squareFractal.pyde*

def squareFractal(sz,level):

if level == 0:

rect(0,0,sz,sz)

else:

rect(0,0,sz/2.0,sz/2.0)

translate(sz/2.0,0)

rect(0,0,sz/2.0,sz/2.0)

translate(-sz/2.0,sz/2.0)

rect(0,0,sz/2.0,sz/2.0)

*示例10-13：向方块分形添加更多方块*

在这里，如果层级为0，我们绘制一个大方块。如果层级不是0，我们在屏幕的左上角添加一个较小的方块，接着向右平移，添加另一个较小的方块在右上角，然后向左（负x方向）和平移向下（正y方向），并在屏幕左下角添加一个较小的方块。

这是下一层，当我们在draw()函数中将squareFractal(500,0)更新为squareFractal(500,1)时，它应该给我们一个没有右下四分之一的方块，如[图10-17](ch10.xhtml#ch10fig17)所示。

![image](../images/f217-01.jpg)

*图10-17：方块分形的下一层*

对于下一层，我们希望每个方块都进一步细分为分形，因此我们将rect行替换为squareFractal()，将sz的值除以2，并告诉它降一级，就像在[示例10-14](ch10.xhtml#ch10list14)中那样。

*squareFractal.pyde*

def squareFractal(sz,level):

if level == 0:

rect(0,0,sz,sz)

else:

squareFractal(sz/2.0,level-1)

translate(sz/2.0,0)

squareFractal(sz/2.0,level-1)

translate(-sz/2.0,sz/2.0)

squareFractal(sz/2.0,level-1)

*示例10-14：向方块分形添加递归步骤*

![image](../images/f218-01.jpg)

*图10-18：不是我们预期的结果！*

在[示例10-14](ch10.xhtml#ch10list14)中，注意到rect行（当层级不是0时）被squareFractal()替换。当我们在draw()函数中调用squareFractal(500,2)时，我们没有得到预期的输出，而是得到了[图10-18](ch10.xhtml#ch10fig18)。

这是因为我们没有像本章前面所做的那样，将平移操作恢复到起始点。

虽然我们可以手动计算平移量，但我们也可以在Processing中使用pushMatrix()和popMatrix()函数，你在[第5章](ch05.xhtml#ch05)中已经学过这些。

我们可以使用pushMatrix()函数保存当前屏幕的方向——也就是原点(0,0)的位置以及网格的旋转角度。之后，我们可以任意进行平移和旋转，最后使用popMatrix()函数恢复到之前保存的方向，而无需进行任何计算！

让我们在squareFractal()函数的开头添加pushMatrix()，并在结尾添加popMatrix()，就像[列表10-15](ch10.xhtml#ch10list15)中一样。

*squareFractal.pyde*

def squareFractal(sz,level):

if level == 0:

rect(0,0,sz,sz)

else:

pushMatrix()

squareFractal(sz/2.0,level-1)

translate(sz/2.0,0)

squareFractal(sz/2.0,level-1)

translate(-sz/2.0,sz/2.0)

squareFractal(sz/2.0,level-1)

popMatrix()

*列表10-15：使用pushMatrix()和popMatrix()完成方形分形*

现在，第1层中的每个小方形都应该转变为一个分形，右下角的方形被去除，正如[图10-19](ch10.xhtml#ch10fig19)所示。

![image](../images/f219-01.jpg)

*图10-19：方形分形的第2层*

现在，让我们尝试通过将squareFractal(500,2)替换为[列表10-16](ch10.xhtml#ch10list16)中的代码，让鼠标生成层数。

*squareFractal.pyde*

def draw():

background(255)

translate(50,50)

level = int(map(mouseX,0,width,0,7))

squareFractal(500,level)

*列表10-16：让方形分形变得交互式*

在更高层次中，方形分形看起来很像谢尔宾斯基三角形，正如你在[图10-20](ch10.xhtml#ch10fig20)中所看到的那样！

![image](../images/f219-02.jpg)

*图10-20：高层次的方形分形看起来像谢尔宾斯基三角形！*

### 龙曲线

我们将要创建的最终分形与之前创建的其他分形有所不同，因为每一层的形状不是变小，而是变大。[图10-21](ch10.xhtml#ch10fig21)展示了龙曲线从第0层到第3层的例子。

![image](../images/f220-01.jpg)

*图10-21：龙曲线的前四个层次*

正如数学娱乐家Vi Hart在她的一段YouTube视频中展示的那样，龙曲线的后半部分是前半部分的完美复制，她通过折叠然后展开纸片来模拟这一过程。[图10-21](ch10.xhtml#ch10fig21)中的第三层（level 2）看起来像是两个左转后接一个右转。每条龙曲线的“铰链”或“折叠”位于曲线的中点。看看你能不能在你的龙曲线中找到它！接下来，你将动态地旋转曲线的一部分，以匹配下一层的曲线。

打开一个新的Processing草图，并命名为*dragonCurve.pyde*。为了创建这个分形，我们首先创建一个“左龙”的函数，就像[列表10-17](ch10.xhtml#ch10list17)中所示。

*dragonCurve.pyde*

def setup():

size(600,600)

strokeWeight(2) #稍微加粗的线条

def draw():

background(255)

translate(width/2,height/2)

leftDragon(5,11)

def leftDragon(sz,level):

if level == 0:

line(0,0,sz,0)

translate(sz,0)

else:

leftDragon(sz,level-1)

rotate(radians(-90))

rightDragon(sz,level-1)

*Listing 10-17: 编写 leftDragon() 函数*

在常规的 setup() 和 draw() 函数之后，我们定义了我们的 leftDragon() 函数。如果 level 为 0，我们只需画一条线，然后沿线平移。这有点像 [第 1 章](ch01.xhtml#ch01) 中的海龟在行进时画线。如果 level 大于 0，先画一个左龙（降一级），然后左转 90 度，画一个右龙（降一级）。

现在我们将创建 “右龙” 函数（见 [Listing 10-18](ch10.xhtml#ch10list18)）。它与 leftDragon() 函数非常相似。如果 level 为 0，只需画一条线并沿着它移动。否则，先画一个左龙，然后这次右转 90 度，画一个右龙。

*dragonCurve.pyde*

def rightDragon(sz,level):

如果 level == 0:

line(0,0,sz,0)

translate(sz,0)

否则:

leftDragon(sz,level-1)

rotate(radians(90))

rightDragon(sz,level-1)

*Listing 10-18: 编写 rightDragon() 函数*

有趣的是，这里的递归语句不仅存在于一个函数内，而且还在左龙函数和右龙函数之间来回跳跃！执行它，第 11 级的图形将像[图 10-22](ch10.xhtml#ch10fig22)一样。

![image](../images/f221-01.jpg)

*Figure 10-22: 第 11 级龙曲线*

这条分形曲线远不只是一个混乱的角度堆砌，经过足够的层级，它开始看起来像一条龙！还记得我说过龙曲线在中间是“折叠”的吗？在[Listing 10-19](ch10.xhtml#ch10list19)中，我添加了一些变量来改变层级和大小，并且让我添加了一个角度变量，该变量根据鼠标的 x 坐标发生变化。这将使龙曲线绕着“铰链”旋转，铰链位于下一层龙曲线的中间。看看如何通过简单地旋转曲线来得到下一层曲线的两个半部分！

*dragonCurve.pyde*

➊ RED = color(255,0,0)

BLACK = color(0)

def setup():

➋ global thelevel,size1

size(600,600)

➌ thelevel = 1

size1 = 40

def draw():

global thelevel

background(255)

translate(width/2,height/2)

➍ angle = map(mouseX,0,width,0,2*PI)

stroke(RED)

strokeWeight(3)

pushMatrix()

leftDragon(size1,thelevel)

popMatrix()

leftDragon(size1,thelevel-1)

➎ rotate(angle)

stroke(BLACK)

rightDragon(size1,thelevel-1)

def leftDragon(sz,level):

如果 level == 0:

line(0,0,sz,0)

translate(sz,0)

否则:

leftDragon(sz,level-1)

rotate(radians(-90))

rightDragon(sz,level-1)

def rightDragon(sz,level):

如果 level == 0:

line(0,0,sz,0)

translate(sz,0)

否则:

leftDragon(sz,level-1)

rotate(radians(90))

rightDragon(sz,level-1)

def keyPressed():

global thelevel,size1

➏ 如果 key == CODED:

如果 keyCode == UP:

thelevel += 1

如果 keyCode == DOWN:

thelevel -= 1

如果 keyCode == LEFT:

size1 -= 5

如果 keyCode == RIGHT:

size1 += 5

*Listing 10-19: 一个动态的龙曲线*

在 [清单 10-19](ch10.xhtml#ch10list19) 中，我们添加了几种颜色 ➊ 用于曲线。在 setup() 函数中，我们声明了两个全局变量，`thelevel` 和 `size1` ➋，并在 ➌ 处声明了它们的初始值，随后在文件末尾的 keyPressed() 函数中通过箭头键来更改它们。

在 draw() 函数中，我们将一个角度变量 ➍ 与鼠标的 x 坐标关联。之后，我们将描边颜色设置为红色，稍微加大描边宽度，并用 `thelevel` 和 `size1` 的初始值绘制一个左龙曲线。你会记得，pushMatrix() 和 popMatrix() 函数仅仅是将绘图点恢复到原来的位置，以便绘制另一条曲线。然后，我们将网格旋转角度变量所表示的弧度 ➎，并绘制另一条黑色的龙曲线。leftDragon() 和 rightDragon() 函数与之前完全相同。

Processing 内置的 keyPressed() 函数对于在草图中更改变量非常有用！你只需要声明你想要通过键盘上的左（在此情况下）、右、上和下箭头键来更改的全局变量。请注意，CODED ➏ 只是表示它不是字母或字符键。最后，程序会检查哪个箭头键被按下，并根据按下的上或下箭头键使 `level` 变量增加或减少，或者根据按下的左或右箭头键使 `size` 变量增加或减少。

当你运行这个版本的 *dragonCurve* 草图时，它会绘制一个五级的红色龙曲线；然后你可以旋转一个四级曲线，看到五级曲线由两个四级曲线组成，只是中间进行了旋转，正如 [图 10-23](ch10.xhtml#ch10fig23) 所示。

![image](../images/f223-01.jpg)

*图 10-23：一个五级龙曲线和一个动态的、互动的四级曲线*

当你移动鼠标时，黑色的龙曲线会旋转，你可以看到它如何适应红色曲线的两半。上下箭头键控制曲线的级别；按下上箭头键，曲线变长。如果曲线超出了显示窗口，可以按左箭头键将每个段缩短，以使其适应屏幕。右箭头键使其变大。

这很有道理，因为 leftDragon() 函数先执行，向左转，绘制一个右龙曲线。而 rightDragon() 函数则与 leftDragon() 相反：它在中间向右转，而不是向左转。难怪它会变成一个完美的复制。

### 摘要

我们只是刚刚触及分形的表面，但希望你已经领略到了分形的美丽，以及它们在模拟自然的复杂性方面的强大能力。分形和递归可以帮助我们重新审视关于逻辑和度量的观念。问题不再是“海岸线有多长？”而是“它有多么曲折？”

对于像海岸线和蜿蜒河流这样的分形线条，标准特征是自相似的尺度，或者我们需要将地图缩放多少，才能使其看起来像是同一事物的不同尺度。这实际上就是你通过输入0.8*sz、sz/2.0或sz/3.0到下一个层级所做的。

在下一章中，我们将创建细胞自动机（CAs），我们将在屏幕上绘制它们，作为小方块，它们会诞生、成长并响应周围的环境。就像在[第9章](ch09.xhtml#ch09)中我们的吃草的羊一样，我们将创建CAs并让它们运行——就像分形一样，我们将观察从非常简单的规则中产生的惊人而美丽的图案。
