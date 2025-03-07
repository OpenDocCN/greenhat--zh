## 9

使用类构建对象

*老教师不会死，他们只是失去了他们的班级。*

—匿名*

![image](img/fintro-01.jpg)

现在你已经使用函数和其他代码在 Processing 中创建了酷炫的图形，你可以通过使用类来激发你的创造力。*类* 是一种结构，它让你可以创建新的对象类型。这些对象类型（通常简称为 *对象*）可以拥有 *属性*，即变量，以及 *方法*，即函数。有时候你想用 Python 绘制多个对象，但绘制很多对象会非常繁琐。类使得绘制多个具有相同属性的对象变得简单，但它们需要特定的语法，你需要学习。

来自官方 Python 网站的以下示例展示了如何使用类创建一个“狗”对象。为了跟着一起编写代码，请在 IDLE 中打开一个新文件，将其命名为 *dog.py*，并输入以下代码。

*dog.py*

class Dog:

def __init__(self,name):

self.name = name

这创建了一个名为 Dog 的新对象，使用了 Dog 类。在 Python 以及许多其他语言中，类名通常以大写字母开头，但即使不这么做也能正常工作。要实例化或创建类，我们必须使用 Python 的 __init__ 方法，它在 init 前后各有两个下划线，表示它是一个特殊方法，用于创建（或 *构造*）对象。__init__ 这一行使得我们能够创建该类的实例（在本例中是狗）。在 __init__ 方法中，我们可以创建类的任何属性。由于这是狗，它可以有一个名字，而且因为每只狗都有自己的名字，我们使用 self 语法。我们在调用对象时不需要使用它，只在定义它们时需要使用。

然后我们可以通过以下代码行创建一只带名字的狗：

d = Dog('Fido')

现在，d 是一只狗，它的名字是 Fido。你可以通过运行文件并在 shell 中输入以下命令来确认这一点：

>>> d.name

'Fido'

现在，当我们调用 d.name 时，我们得到 Fido，因为这是我们刚刚赋予它的名字属性。我们还可以创建另一只狗，并给它取名 Bettisa，像这样：

>>> b = Dog('Bettisa')

>>> b.name

'Bettisa'

你可以看到，一只狗的名字可以与另一只不同，但程序能完美记住它们！这在我们给对象分配位置和其他属性时将非常重要。

最后，我们可以通过在类中放入一个函数来给这只狗安排一些事情。但不要称它为函数！类中的函数叫做 *方法*。狗会叫，所以我们会将这个方法添加到 示例 9-1 中的代码。

*dog.py*

class Dog:

def __init__(self,name):

self.name = name

def bark(self):

print("汪！")

d = Dog('Fido')

*示例 9-1：创建一只会叫的狗！*

当我们调用 d 狗的 bark() 方法时，它会叫：

>>> d.bark()

汪！

这个简单的例子可能不太能看出为什么需要一个 Dog 类，但了解你可以用类做任何事情，并且可以充分发挥创造力，还是很有帮助的。在本章中，我们使用类来制作许多有用的对象，比如弹跳球和吃草的羊。我们从弹跳球的例子开始，看看使用类是如何让我们做一些非常酷的事情，同时节省大量工作的。

### 弹跳球程序

启动一个 Processing 草图并将其保存为 *BouncingBall.pyde*。我们将在屏幕上绘制一个圆形，并将其变成一个弹跳球。代码清单 9-2 展示了绘制一个圆形的代码。

*BouncingBall.pyde*

def setup():

size(600,600)

def draw():

background(0) #黑色

ellipse(300,300,20,20)

*代码清单 9-2：绘制一个圆形*

首先，我们将窗口的大小设置为 600 像素宽和 600 像素高。然后，我们将背景设置为黑色，并使用 ellipse() 函数绘制一个圆形。函数中的前两个数字描述圆心距窗口左上角的距离，最后两个数字描述椭圆的宽度和高度。在这个例子中，ellipse(300,300, 20,20) 创建了一个 20 像素宽、20 像素高的圆形，位于显示窗口的中央，如图 图 9-1 所示。

![image](img/f177-01.jpg)

*图 9-1：为弹跳球草图绘制一个圆形*

现在我们已经成功地在显示窗口的中心创建了一个圆形，接下来我们尝试让它移动。

#### 让球移动

我们将通过改变球的位置来让它移动。为此，我们首先创建一个用于 x 值的变量和一个用于 y 值的变量，并将它们设置为 300，也就是屏幕的中间。返回到 代码清单 9-2，并在代码的开头插入以下两行，就像在 代码清单 9-3 中那样。

*BouncingBall.pyde*

xcor = 300

ycor = 300

def setup():

size(600,600)

*代码清单 9-3：设置 x 和 y 值的变量*

在这里，我们使用 xcor 变量表示 x 值，使用 ycor 变量表示 y 值。然后，我们将两个变量都设置为 300。

现在让我们通过改变 x 值和 y 值来改变椭圆的位置。确保使用这些变量来绘制椭圆，如 代码清单 9-4 所示。

*BouncingBall.pyde*

xcor = 300

ycor = 300

def setup():

size(600,600)

def draw():

➊ global xcor, ycor

background(0) #黑色

xcor += 1

ycor += 1

ellipse(xcor,ycor,20,20)

*代码清单 9-4：增加 xcor 和 ycor 来改变椭圆的位置*

这个示例中需要注意的重点是全局变量 xcor 和 ycor ➊，它告诉 Python 使用我们已经创建的变量，而不是仅仅为 draw() 函数创建新的变量。如果你不包括这一行，你会得到一个错误消息，比如“局部变量 ‘xcor’ 在赋值前被引用”。一旦 Processing 知道该给 xcor 和 ycor 赋什么值，我们就将它们都加 1，并使用全局变量 (xcor, ycor) 来绘制椭圆，椭圆的中心位置是由这些变量指定的。

当你保存并运行 清单 9-4 时，你应该看到球体移动，就像在 图 9-2 中一样。

现在，球体向下和向右移动，因为它的 x 和 y 值都在增加，但接着它就会离开屏幕，我们再也看不见它了！程序继续听从指令地增加我们的变量。它并不知道它正在绘制一个球体，也不知道我们希望球体能弹回墙壁。接下来让我们看看如何防止球体消失。

![image](img/f179-01.jpg)

*图 9-2：球体移动了！*

#### 让球体弹回墙壁

当我们通过加 1 来改变 x 值和 y 值时，我们实际上是在改变物体的位置。在数学中，这种随时间变化的位置变化被称为 *速度*。在时间内 x 的正向变化（正 x 速度）看起来像是向右移动（因为 x 在变大），而负 x 速度则看起来像是向左移动。我们可以利用这个“正右负左”的概念来让球体弹回墙壁。首先，我们通过向现有代码中添加以下几行来创建 x 速度和 y 速度变量，如 清单 9-5 所示。

*BouncingBall.pyde*

xcor = 300

ycor = 300

xvel = 1

yvel = 2

def setup():

size(600, 600)

def draw():

global xcor, ycor, xvel, yvel

background(0) # 黑色背景

xcor += xvel

ycor += yvel

# 如果球体触及墙壁，改变方向。

如果 xcor > 宽度或 xcor < 0:

xvel = -xvel

如果 ycor > 高度 或 ycor < 0:

yvel = -yvel

ellipse(xcor, ycor, 20, 20)

*清单 9-5：添加代码让球体弹回墙壁*

首先，我们将 xvel = 1 和 yvel = 2 来指定球体的移动方式。你可以尝试其他数值，看看它们如何改变球体的运动。然后，在 draw() 函数中，我们告诉 Python xvel 和 yvel 是全局变量，并通过增加这些变量来改变 x 和 y 坐标。例如，当我们设置 xcor += xvel 时，我们就是通过速度（位置的 *变化*）来更新位置。

这两个 if 语句告诉程序，如果球体的位置超出屏幕边界，它应该将球体的速度改为其负值。当我们将球体的速度改为负值时，我们就是在告诉程序让球体朝相反方向移动，这样就像是球体弹回了墙壁。

我们需要精确地指出球应该在哪个坐标点反向移动。例如，xcor > width 表示 xcor 大于显示窗口的宽度，意味着球触碰到了屏幕的右边缘。而 xcor < 0 表示 xcor 小于 0，或者球触碰到了屏幕的左边缘。类似地，ycor > height 检查 ycor 是否大于窗口的高度，或者球到达了屏幕的底部。最后，ycor < 0 检查球是否到达了屏幕的上边缘。由于向右移动是正的 x 速度（x 的正变化），反方向是负的 x 速度。如果速度已经是负的（它正在向左移动），那么负数的负数是正数，这意味着球将向右移动，正如我们希望的那样。

当你运行列出 9-5 时，你应该能看到类似图 9-3 中所展示的效果。

![image](img/f180-01.jpg)

*图 9-3：一个弹跳的球！*

球看起来像是从墙壁上反弹，因此始终保持在视图内。

#### 在没有类的情况下创建多个球

现在假设我们想要创建另一个弹跳球，或者许多其他的弹跳球。我们该怎么做呢？我们可以为第二个球的 x 值创建一个新变量，为第二个球的 y 值再创建一个变量，为其 x 速度创建第三个变量，再为其 y 速度创建第四个变量。然后，我们必须根据速度增加其位置，检查是否需要从墙壁反弹，最后绘制它。然而，我们最终会得到双倍的代码量！添加第三个球将使我们的代码量增加三倍！二十个球简直无法想象。你*不*想要一直跟踪这些位置和速度的变量。列出 9-6 展示了这种情况。

#ball1:

ball1x = random(width)

ball1y = random(height)

ball1xvel = random(-2,2)

ball1tvel = random(-2,2)

#ball2:

ball2x = random(width)

ball2y = random(height)

ball2xvel = random(-2,2)

ball2tvel = random(-2,2)

#ball3:

ball3x = random(width)

ball3y = random(height)

ball3xvel = random(-2,2)

ball3tvel = random(-2,2)

#更新球 1：

ball1x += ball1xvel

ball1y += ball1yvel

ellipse(ball1x,ball1y,20,20)

#更新球 2：

ball2x += ball2xvel

ball2y += ball2yvel

ellipse(ball2x,ball2y,20,20)

#更新球 3：

ball3x += ball3xvel

ball3y += ball3yvel

ellipse(ball3x,ball3y,20,20)

*列出 9-6：在没有类的情况下创建多个球。代码太多了！*

这是仅创建三个球的代码。正如你所看到的，它非常长，而且这还不包括反弹部分！让我们看看如何利用类来简化这项任务。

#### 使用类创建对象

在编程中，类就像是一个食谱，详细说明了如何创建一个具有特定属性的对象。通过使用类，我们告诉 Python 如何创建一个球体一次。然后，我们只需使用 for 循环创建多个球体，并将它们放入列表中。列表非常适合保存多个事物——字符串、数字和对象！

使用类创建对象时，请按照以下三个步骤操作：

1.  *编写类*。这就像是制作球体、行星、火箭等的食谱。

1.  *实例化对象*。通过在 setup() 函数中调用对象来执行此操作。

1.  *更新对象*。在 draw() 函数（显示循环）中执行此操作。

让我们使用这些步骤将我们已经编写的代码放入类中。

##### 编写类

使用类创建对象的第一步是编写一个类，告诉程序如何制作一个球体。让我们将 列表 9-7 中的代码添加到我们现有程序的最前面。

*BouncingBall.pyde*

ballList=[]  # 用于存放球体的空列表

class Ball:

def __init__(self, x, y):

'''如何初始化一个 Ball'''

self.xcor = x

self.ycor = y

self.xvel = random(-2, 2)

self.yvel = random(-2, 2)

*列表 9-7：定义一个名为 Ball 的类*

请注意，由于我们将位置和速度变量作为属性放入 Ball 类中，您可以从现有代码中删除以下几行：

xcor = 300

ycor = 300

xvel = 1

yvel = 2

在 列表 9-7 中，我们创建了一个空列表，用于保存球体；然后我们开始定义食谱。类对象的名称（在这里是 Ball）总是首字母大写。__init__ 方法是 Python 中创建类的必要条件，它包含对象在初始化时获得的所有属性。否则，类将无法工作。

self 语法意味着每个对象都有自己的方法和属性，这些函数和变量只能由 Ball 对象使用。这意味着每个 Ball 都有自己的 xcor、ycor 等等。因为我们可能需要在某个特定位置创建一个 Ball，所以我们将 x 和 y 作为 __init__ 方法的参数。添加这些参数使我们能够在创建 Ball 时告诉 Python 其位置，如下所示：

Ball(100, 200)

在这种情况下，球体将位于坐标 (100,200)。

列表 9-7 中的最后几行告诉 Processing 为新球体分配一个介于 -2 和 2 之间的随机数作为其 x 和 y 速度。

##### 实例化对象

现在我们已经创建了一个名为 Ball 的类，我们需要告诉 Processing 如何在每次 draw() 函数循环时更新球体。我们称之为更新方法，并将其嵌套在 Ball 类内部，就像我们在 __init__ 中所做的那样。您只需将所有球体代码剪切并粘贴到 update() 方法中，然后在每个对象的属性前添加 self.，如 列表 9-8 所示。

*BouncingBall.pyde*

ballList=[] #空列表，用于存放小球

class Ball:

def __init__(self,x,y):

'''如何初始化一个小球'''

self.xcor = x

self.ycor = y

self.xvel = random(-2,2)

self.yvel = random(-2,2)

def update(self):

self.xcor += self.xvel

self.ycor += self.yvel

#如果小球碰到墙壁，则改变方向

if self.xcor > width or self.xcor < 0:

self.xvel = -self.xvel

if self.ycor > height or self.ycor < 0:

self.yvel = -self.yvel

ellipse(self.xcor,self.ycor,20,20)

*清单 9-8：创建 update()方法*

在这里，我们将所有用于移动和反弹小球的代码都放入了 Ball 类的 update()方法中。唯一的新代码是速度变量中的 self，使它们成为 Ball 对象的速度属性。虽然看起来有很多 self 的实例，但这就是我们告诉 Python，x 坐标例如属于特定的小球而不是其他小球的方式。很快，Python 将更新上百个小球，因此我们需要 self 来跟踪每个小球的位置和速度。

现在程序已经知道如何创建和更新小球，让我们更新 setup()函数，创建三个小球并将它们放入小球列表（ballList）中，如清单 9-9 所示。

def setup():

size(600,600)

for i in range(3):

ballList.append(Ball(random(width),

random(height)))

*清单 9-9：在 setup()函数中创建三个小球*

我们已经在清单 9-7 中创建了 ballList，这里我们向列表添加一个在随机位置的小球。当程序创建（实例化）一个新小球时，它将选择一个在 0 和屏幕宽度之间的随机数作为 x 坐标，并选择另一个在 0 和屏幕高度之间的随机数作为 y 坐标。然后，它会将这个新小球放入列表中。由于我们使用了循环 for i in range(3)，程序将向小球列表中添加三个小球。

##### 更新对象

现在让我们告诉程序每次循环时都遍历 ballList，并更新列表中的所有小球（即绘制它们），使用如下的 draw()函数：

*BouncingBall.pyde*

def draw():

background(0) #黑色

for ball in ballList:

ball.update()

注意，我们仍然希望背景是黑色的，然后我们遍历小球列表，并对列表中的每个小球运行它的 update()方法。所有之前在 draw()中的代码都已移入 Ball 类中！

![image](img/f184-01.jpg)

*图 9-4：创建任意数量的反弹小球！*

当你运行这个示例时，你应该会看到三个小球在屏幕上移动并撞击墙壁反弹！使用类的一个好处是，改变小球数量非常简单。你只需要更改 setup()函数中 for i in range(*number*):中的*number*，就能创建更多反弹的小球。例如，当你将其改为 20 时，你将看到类似图 9-4 的效果。

使用类的一个好处是，你可以为一个对象赋予任何你想要的属性或方法。例如，我们不需要让所有球体的颜色都相同。将清单 9-10 中的三行代码添加到你现有的 Ball 类中。

*BouncingBall.pyde*

class Ball:

def __init__(self,x,y):

'''如何初始化一个 Ball 对象'''

self.xcor = x

self.ycor = y

self.xvel = random(-2,2)

self.yvel = random(-2,2)

self.col = color(random(255),

random(255),

random(255))

*清单 9-10：更新 Ball 类*

这段代码在每个球体创建时赋予其独特的颜色。Processing 的 color()函数需要三个数字，分别表示红色、绿色和蓝色。RGB 值的范围是从 0 到 255。使用 random(255)允许程序随机选择这三个数值，从而得到一个随机的颜色。然而，由于 __init__ 方法只会运行一次，一旦球体有了颜色，它就会保持这个颜色。

接下来，在 update()方法中，添加以下代码行，使椭圆填充其自身随机选择的颜色：

fill(self.col)

ellipse(self.xcor,self.ycor,20,20)

在绘制形状或线条之前，可以使用 fill 来声明形状的颜色，使用 stroke 来声明线条的颜色。在这里，我们告诉 Processing 使用球体自身的颜色（使用 self）来填充以下形状。

现在，当你运行程序时，每个球体都应该具有随机的颜色，如图 9-5 所示！

![image](img/f185-01.jpg)

*图 9-5：给球体赋予各自的颜色*

练习 9-1：创建不同大小的球体

给每个球体设置一个自己的大小，范围在 5 到 50 个单位之间。

### 吃草的羊程序

现在你可以创建类了，接下来我们来做一些有用的东西。我们将编写一个 Processing 的生态系统草图，模拟羊在四处走动并吃草。在这个草图中，羊有一定的能量，随着它们走动，能量会逐渐消耗，当它们吃草时，能量会被补充。如果它们的能量足够，它们就会繁殖；如果能量不足，它们就会死亡。通过创建和调整这个模型，我们有可能学到很多关于生物学、生态学和进化的知识。

在这个程序中，Sheep 对象有点像你在本章早些时候创建的 Ball 对象；每个都有自己的 x 和 y 坐标，以及大小，并用圆形表示。

#### 编写羊类

开始一个新的 Processing 草图并保存为*SheepAndGrass.pyde*。首先，让我们创建一个类，使其能够生成一个具有自定义 x 和 y 坐标以及自定义大小的 Sheep 对象。接着，我们将创建一个 update 方法，在羊的位置绘制一个表示羊大小的椭圆。

该类代码几乎与 Ball 类相同，正如你在清单 9-11 中看到的那样。

*SheepAndGrass.pyde*

class Sheep:

def __init__(self,x,y):

self.x = x #x-position

self.y = y #y-position

self.sz = 10 #大小

def update(self):

ellipse(self.x,self.y,self.sz,self.sz)

*清单 9-11：创建一个羊类*

因为我们知道以后会创建很多羊，所以我们开始创建一个 Sheep 类。在必需的 __init__ 方法中，我们将羊的 x 和 y 坐标设置为我们在创建羊实例时声明的参数。我将羊的大小（椭圆的直径）设置为 10 像素，但如果你喜欢，也可以设置更大或更小的羊。update()方法仅仅是根据羊的位置绘制一个与羊大小相同的椭圆。

这是包含一个名为 shawn 的 Sheep 的 Processing 草图中的 setup()和 draw()代码。将清单 9-12 中显示的代码添加到你刚才在清单 9-11 中写的 update()方法下面。

def setup():

global shawn

size(600,600)

#在(300,200)创建一个名为 shawn 的 Sheep 对象

shawn = Sheep(300,200)

def draw():

background(255)

shawn.update()

*清单 9-12：创建一个名为 shawn 的 Sheep 对象*

我们首先在 setup()函数中创建了 shawn，一个 Sheep 对象的实例。然后我们在 draw()函数中更新它——但是 Python 不知道我们指的是同一个 shawn，除非我们告诉它 shawn 是一个全局变量。

当你运行这段代码时，你应该会看到类似图 9-6 所示的效果。

![image](img/f187-01.jpg)

*图 9-6：一只羊*

你会看到一个白色的屏幕，上面有一只小圆形的羊，位于坐标（300,200），即距离起始点 300 像素，向下 200 像素。

#### 编程让羊四处移动

现在让我们教一只羊如何四处移动。我们将首先编程让羊随机移动。（如果你愿意，未来你可以让它按照不同的方式移动。）清单 9-13 会把一只羊的 x 和 y 坐标更改为-10 到 10 之间的随机数。回到你现有的代码中，并在 update()方法中的 ellipse()函数上方添加以下代码：

*SheepAndGrass.pyde*

def update(self):

#让羊随机行走

move = 10  #它可以在任何方向上移动的最大距离

self.x += random(-move, move)

self.y += random(-move, move)

fill(255)  #白色

ellipse(self.x,self.y,self.sz,self.sz)

*清单 9-13：让羊随机移动*

这段代码创建了一个名为 move 的变量，用来指定羊在屏幕上能够移动的最大值或距离。然后我们将 move 设置为 10，并使用它通过- move（-10）到 move（10）之间的随机数来更新羊的 x 和 y 坐标。最后，我们使用 fill(255)将羊的颜色暂时设置为白色。

当你运行这段代码时，你应该会看到羊在屏幕上随机徘徊——它可能会走出屏幕。

让我们给羊找个伴。如果我们想创建并更新多个对象，最好将它们放入一个列表中。然后在 draw()函数中，我们将遍历这个列表，并更新每一只羊。更新你的现有代码，使其看起来像清单 9-14。

*SheepAndGrass.pyde*

class Sheep:

def __init__(self,x,y):

self.x = x  #x-坐标

self.y = y #y 坐标

self.sz = 10 #大小

def update(self):

#让羊随机走动

move = 10 #它可以在任何方向上移动的最大值

self.x += random(-move, move)

self.y += random(-move, move)

fill(255) #白色

ellipse(self.x,self.y,self.sz,self.sz)

sheepList = [] #存储羊的列表

def setup():

size(600,600)

for i in range(3):

sheepList.append(Sheep(random(width),

random(height)))

def draw():

background(255)

for sheep in sheepList:

sheep.update()

*清单 9-14：使用 for 循环创建更多的羊*

这段代码类似于我们之前写的将弹跳球放入列表的代码。首先，我们创建一个列表来存储羊。然后我们创建一个 for 循环，将一只羊放入羊列表。接着，在 draw() 函数中，我们再写一个 for 循环，遍历羊列表并根据我们已定义的 update() 方法更新每只羊。当你运行这段代码时，你应该会看到三只羊在随机走动。将 for i in range(3): 中的数字 3 改为更大的数字，可以增加更多的羊。

#### 创建能量属性

行走需要消耗能量！让我们在羊被创建时给它们一定的能量，在它们走动时消耗掉这些能量。使用清单 9-15 中的代码来更新你现有的 __init__ 和 update()方法，位于*SheepAndGrass.pyde*文件中。

class Sheep:

def __init__(self,x,y):

self.x = x #x 坐标

self.y = y #y 坐标

self.sz = 10 #大小

self.energy = 20 #能量水平

def update(self):

#让羊随机走动

move = 1

self.energy -= 1 #走动消耗能量

if sheep.energy <= 0:

sheepList.remove(self)

self.x += random(-move, move)

self.y += random(-move, move)

fill(255) #白色

ellipse(self.x,self.y,self.sz,self.sz)

*清单 9-15：使用能量属性更新 __init__ 和 update() 方法*

我们通过在 __init__ 方法中创建一个能量属性并将其设置为 20（每只羊的初始能量值）来实现这一点。然后，在 update() 方法中通过 self.energy -= 1 每次走动时降低羊的能量值。

然后我们检查羊是否耗尽了能量，如果是，就从 sheepList 中移除它。这里，我们使用一个条件语句来检查 sheep.energy <= 0 是否返回 True。如果是，我们就用 remove() 函数将该羊从 sheepList 中移除。一旦该羊实例从列表中消失，它就不再存在。

#### 使用类创建草

当你运行程序时，你应该会看到羊走动了一会儿然后消失——走动消耗了羊的能量，一旦能量耗尽，羊就会死掉。我们需要做的是给羊提供草来吃。我们将每一块草称为 Grass，并为其创建一个新类。Grass 会有自己的 x 和 y 值、大小和能量含量。我们还会在草被吃掉时改变它的颜色。

实际上，我们在这个草图中会使用许多不同的颜色来表示我们的羊和草，所以我们将在程序的最开始添加 列表 9-16 中的代码，这样我们就可以通过颜色名称来引用这些颜色。也可以随意添加其他颜色。

WHITE = color(255)

BROWN = color(102,51,0)

RED = color(255,0,0)

GREEN = color(0,102,0)

YELLOW = color(255,255,0)

PURPLE = color(102,0,204)

*列表 9-16：将颜色设置为常量*

使用全大写字母表示颜色名称表示它们是常量，并且值不会改变，但这只是对程序员的约定。常量本身没有神奇之处，如果你愿意，可以更改这些值。设置常量让你只需要输入颜色名称，而不用每次都写 RGB 值。我们将在将草变绿时使用这个方法。通过在 *SheepAndGrass.pyde* 中的 Sheep 类之后添加 列表 9-17 中的代码，来更新现有代码：

class Grass:

def __init__(self,x,y,sz):

self.x = x

self.y = y

self.energy = 5 #通过吃这个草块获得的能量

self.eaten = False #尚未被吃掉

self.sz = sz

def update(self):

fill(GREEN)

rect(self.x,self.y,self.sz,self.sz)

*列表 9-17：编写 Grass 类*

你可能已经开始习惯类表示法的结构了。它通常以 __init__ 方法开始，在那里你创建类的属性。在这种情况下，你告诉程序，Grass 将具有 x 和 y 坐标、能量级别、一个布尔值（True/False）变量来追踪草是否被吃掉过，以及草的大小。要更新一个草块，我们只需要在 Grass 对象的位置创建一个绿色矩形。

现在，我们需要像处理羊一样初始化和更新我们的草。因为草会很多，所以让我们为它创建一个列表。在 setup() 函数之前，添加以下代码。

sheepList = [] #存储羊的列表

grassList = [] #存储草的列表

patchSize = 10 #每个草块的大小

我们可能想在未来改变草块的大小，所以让我们创建一个名为 patchSize 的变量，这样我们就只需要在一个地方修改它。在 setup() 函数中，在创建羊之后，通过添加 列表 9-18 中的新代码来创建草。

def setup():

global patchSize

size(600,600)

#创建羊

for i in range(3):

sheepList.append(Sheep(random(width),

random(height)))

#创建草：

for x in range(0,width,patchSize):

for y in range(0,height,patchSize):

grassList.append(Grass(x,y,patchSize))

*列表 9-18：使用 patchSize 变量更新 Grass 对象*

在这个例子中，global patchSize 告诉 Python 我们在任何地方都使用相同的 patchSize 变量。然后，我们写了两个 for 循环（一个用于 x，另一个用于 y）来将 Grass 添加到草列表中，这样我们就可以创建一个草的方形网格。

然后，我们像更新羊一样，在 draw()函数中更新所有内容。先绘制的内容会被后绘制的内容覆盖，因此我们首先通过将 draw()函数改成列表 9-19 中的代码来更新草地。

*SheepAndGrass.pyde*

def draw():

background(255)

#先更新草地

for grass in grassList:

grass.update()

#然后是羊

for sheep in sheepList:

sheep.update()

*列表 9-19：在羊之前更新草地*

当你运行这段代码时，你应该能看到一组绿色方块，就像图 9-7 中所示。

![image](img/f192-01.jpg)

*图 9-7：带网格线的草地*

让我们关闭黑色边框，这样草地看起来就像是平滑的草地。将 noStroke()添加到 setup()函数中，以去除绿色方块的边框：

def setup():

global patchSize

size(600,600)

noStroke()

现在我们有了草地！

#### 让草地在被吃掉时变成棕色

我们如何使得当一只羊站在一片草地上时，羊能获得草的能量，并且草地变成棕色，显示羊已经吃过它呢？通过添加以下代码行来修改草地的 update()方法：

def update(self):

if self.eaten:

fill(BROWN)

else:

fill(GREEN)

rect(self.x,self.y,self.sz,self.sz)

这段代码告诉 Processing，如果草地“被吃了”，则矩形应该填充为棕色。否则，草地应该是绿色的。羊“吃”草的方式有多种。一种方式是让每一片草地检查整个 sheepList，看是否有羊在它的位置，这意味着成千上万的草地要检查数千只羊。那些数字可能会变得非常大。然而，因为每片草地都在 grassList 中，另一种方法是当羊改变位置时，它可以简单地将该位置的草地标记为“已吃”（如果尚未标记），并从中获取能量。这样就会减少很多检查。

问题是羊的 x 和 y 坐标与 grassList 中草地的位置不完全匹配。例如，我们的 patchSize 是 10，这意味着如果羊在(92,35)的位置，它将位于右边第 10 个草地和下方第 4 个草地（因为“第一个”草地的范围是 x = 0 到 x = 9）。我们通过 patchSize 来划分，得到“缩放后的”x 和 y 值，分别是 9 和 3。

然而，grassList 没有行和列。我们知道 x 值 9 意味着它是第 10 行（不要忘了第 0 行），所以我们只需添加九行 60（即高度除以 patchSize），然后加上 y 值来得到羊所在的草地索引。因此，我们需要一个变量来告诉我们每行有多少片草地，称之为 rows_of_grass。将 global rows_of_grass 添加到 setup()函数的开头，然后在声明大小之后，添加这一行：

rows_of_grass = height/patchSize

这段代码将显示窗口的宽度除以草地块的大小，告诉我们有多少列草。添加到 Sheep 类中的代码在清单 9-20 中。

*SheepAndGrass.pyde*

self.x += random(-move, move)

self.y += random(-move, move)

“包裹”世界，就像小行星游戏一样

➊ if self.x > width:

self.x %= width

if self.y > height:

self.y %= height

if self.x < 0:

self.x += width

if self.y < 0:

self.y += height

#找到你所在的草地区域在 grassList 中的位置：

➋ xscl = int(self.x / patchSize)

yscl = int(self.y / patchSize)

➌ grass = grassList[xscl * rows_of_grass + yscl]

if not grass.eaten:

self.energy += grass.energy

grass.eaten = True

*清单 9-20：更新羊的能量并让草变成棕色*

在更新羊的位置后，我们“包裹”坐标 ➊，所以如果羊走出屏幕的一侧，它会出现在屏幕的另一侧，就像视频游戏 *小行星* 中一样。我们根据 patchSize ➋ 来计算羊所在的草地块。然后我们用代码将 x 和 y 的值转化为该草地块在 grassList 中的索引 ➌。现在我们知道羊所在草地块的确切索引。如果这块草地还没有被吃掉，羊就会吃掉它！它从草地中获取能量，并将草地的 eaten 属性设置为 True。

运行这段代码，你会看到三只羊在四处奔跑，吃掉草地，草地一旦被吃掉就变成棕色。通过将 move 变量设置为较小的值（例如 5）可以让羊走得慢一点。你还可以通过修改一个数字——patchSize 变量，将草地的大小缩小到 5。如果你愿意，可以尝试其他值。

现在我们可以创建更多的羊。让我们将 for i in range 行中的数字改为 20，如下所示：

#创建羊群

for i in range(20):

sheepList.append(Sheep(random(width),

random(height)))

当你运行这段代码时，你应该能看到类似图 9-8 的效果。

![image](img/f194-01.jpg)

*图 9-8：一群羊！*

现在有 20 只羊在四处走动，留下了一片片棕色的草地。

#### 为每只羊赋予一个随机颜色

让羊在“出生”时选择一个颜色。在定义颜色常量的代码之后，让我们将一些颜色放入一个颜色列表中，像这样：

YELLOW = color(255,255,0)

PURPLE = color(102,0,204)

colorList = [WHITE,RED,YELLOW,PURPLE]

对 Sheep 类进行以下更改，以使用不同的颜色。首先，你需要为 Sheep 添加一个颜色属性。由于 color 已经是 Processing 中的关键字，因此在清单 9-21 中使用了 col。

class Sheep:

def __init__(self,x,y,col):

self.x = x #x 位置

self.y = y #y 位置

self.sz = 10 #大小

self.energy = 20

self.col = col

*清单 9-21：向 Sheep 类添加颜色属性*

然后在 update() 方法中，将 fill 行替换为以下代码：

fill(self.col) #它自己的颜色

ellipse(self.x,self.y,self.sz,self.sz)

在画椭圆之前，fill(self.col) 会告诉 Processing 使用羊自己的随机颜色填充椭圆。

当所有羊在 setup() 函数中被实例化时，你需要给它们一个随机颜色。这意味着你必须在程序顶部导入 random 模块中的 choice() 函数，如下所示：

from random import choice

Python 的 choice() 函数允许你从列表中随机选择一个项目并返回。我们应该能让程序这样做：

choice(colorList)

现在程序将从颜色列表中返回一个单一的值。最后，在创建羊时，将从颜色列表中随机选择的颜色作为传递给 Sheep 构造函数的参数之一，如下所示：

def setup():

size(600,600)

noStroke()

#创建羊

for i in range(20):

sheepList.append(Sheep(random(width),

random(height),

choice(colorList)))

现在当你运行这段代码时，你应该会看到一群随机颜色的羊在屏幕上走动，正如图 9-9 所示。

![image](img/f196-01.jpg)

*图 9-9：多彩的羊*

每只新羊都会从我们在 colorList 中定义的四种颜色中随机选一个：白色、红色、黄色或紫色。

#### 编程让羊繁殖

不幸的是，在我们当前的程序中，羊会吃掉草，直到它们走得太远，远离草地，耗尽能量并死亡。为防止这种情况发生，我们可以让羊用一部分能量进行繁殖。

让我们使用清单 9-22 中的代码，当羊的能量达到 50 时，让它们繁殖。

if self.energy <= 0:

sheepList.remove(self)

if self.energy >= 50:

self.energy -= 30 #生育需要能量

#向列表中添加另一只羊

sheepList.append(Sheep(self.x,self.y,self.col))

*清单 9-22：为羊的繁殖添加条件*

条件 if self.energy >= 50: 会检查羊的能量是否大于或等于 50。如果是，它将能量减少 30 用于生育，并向羊群列表中添加另一只羊。请注意，新的羊与父母的颜色相同，并且位于相同的位置。运行这段代码，你应该能看到羊繁殖，就像图 9-10 中所示的那样。

![image](img/f197-01.jpg)

*图 9-10：羊吃草和繁殖*

很快你会看到看起来像是部落的相似颜色的羊群。

#### 让草重新生长

不幸的是，羊很快就吃光了它们区域里的所有草并死去（这大概是某种教训）。我们需要让草重新生长。为此，将草的 update() 方法改成如下：

def update(self):

if self.eaten:

if random(100) < 5:

self.eaten = False

else:

fill(BROWN)

else:

fill(GREEN)

rect(self.x,self.y,self.sz,self.sz)

Processing 代码 random(100) 会生成一个介于 0 和 100 之间的随机数。如果这个数字小于 5，我们会通过将草的 eaten 属性设置为 False 来重新种植一片草地。我们选择数字 5，因为这样可以使每一帧中被吃掉的草有 5/100 的几率重新生长。否则，它会保持棕色。

运行代码，你应该会看到类似图 9-11 的内容。

![image](img/f198-01.jpg)

*图 9-11：草地重新生长，羊群占满了整个屏幕！*

现在，你可能会发现羊群越来越多，程序开始变慢了！这可能是因为羊有太多的能量。如果是这样，尝试将每片草地的能量从 5 降到 2：

class Grass:

def __init__(self,x,y,sz):

self.x = x

self.y = y

self.energy = 2 #吃这片草得到的能量

self.eaten = False #还没被吃掉

self.sz = sz

这似乎是一个良好的平衡，使得羊群能够以合理的速度增长。你可以随意调整数字——这是你的世界！

#### 提供进化优势

让我们给某一组羊一个优势。你可以选择任何你能想到的优势（例如从草地获得更多的能量或一次性产生更多的后代）。在这个例子中，我们将让紫色羊走得比其他羊更远。这样会有变化吗？为了找出答案，将 Sheep 的 update() 方法修改为以下代码：

def update(self):

#让羊随机走动

move = 5 #它在任何方向上最多能移动的距离

if self.col == PURPLE:

move = 7

self.energy -= 1

这个条件判断检查羊的颜色是否为紫色。如果是，它将羊的移动值设置为 7。否则，它将值保持为 5。这样，紫色羊就能比其他羊走得更远，因此更有可能找到绿色草地。让我们运行代码并检查结果，应该类似于图 9-12。

![image](img/f199-01.jpg)

*图 9-12：给紫色羊一个优势*

一段时间后，紫色羊的小小优势似乎真的带来了回报。它们在环境中占据主导地位，并且通过与其他羊争夺草地，把其他羊挤了出去。这个模拟可能会激发关于生态学、外来物种、生物多样性和进化的有趣讨论。

练习 9-2：设置羊的寿命

创建一个“age”属性，并在每次羊更新时减少它，让它们的生命只持续有限的时间。

练习 9-3：改变羊的大小

根据羊的能量水平改变羊的大小。

### 总结

在这一章，你学会了如何使用类来创建对象，这包括通过属性定义类，然后实例化（“创建”）并更新对象。这让你能够更高效地创建多个相似但独立的具有相同属性的对象。你使用类越多，你就能越有创意，通过让自主对象走动、飞行或弹跳，而无需为每个步骤都写代码！

学会使用类能极大提升你的编程能力。现在你可以轻松创建复杂情况的模型，一旦你告诉程序如何处理一个粒子、行星或羊，它就能非常轻松地生成十个、百个，甚至上百万个！

你还体验了如何搭建模型，探索物理、生物、化学或环境等情况，几乎不需要用到任何方程！一位物理学家曾告诉我，这通常是解决涉及多因素或“代理”问题的最有效方法。你搭建一个计算机模型，让它运行，然后观察结果。

在下一章，你将学习如何使用一种几乎是魔法般的现象——递归，来创建分形。
