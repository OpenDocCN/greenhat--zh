# 第十三章：创建二维和三维动画

![图片](img/common.jpg)

一个随时间变化的图像可以很好地可视化许多情况。Haskell Prelude 本身并不支持动画，但在 [`hackage.haskell.org`](https://hackage.haskell.org) 上有一些不错的库包可用。对于二维图像和动画，我们将使用 `gloss` 包。对于三维图像和动画，我们将使用一个名为 `not-gloss` 的包。

### 二维动画

`gloss` 包提供了 `Graphics.Gloss` 模块，其中包含四个主要函数：display、animate、simulate 和 play。第一个用于静态图像，第二和第三个用于随时间变化的图像，第四个用于随时间和用户输入变化的图像。我们主要关注前面三个函数。接下来的几节将详细描述这些函数。

#### 显示二维图像

函数 `display` 生成一个静态图像。让我们向 GHCi 查询 `display` 的类型。由于 `display` 不是 Prelude 的一部分，因此我们必须先加载该模块。

```
Prelude> :m Graphics.Gloss
Prelude Graphics.Gloss> :t display
display :: Display -> Color -> Picture -> IO ()
```

类型 `Display`、`Color` 和 `Picture` 是由 `Graphics.Gloss` 模块定义的，或者可能是由 `Graphics.Gloss` 导入的 `gloss` 包中的另一个模块定义的。类型 `Display` 和 `Color` 分别用于显示模式和背景颜色。最有趣的类型是 `Picture`，它代表了可以显示的内容类型。`gloss` 文档中关于 `Picture` 的描述说明了我们可以创建的图像（例如线条、圆形、多边形等）。你可以通过点击 `Graphics.Gloss` 在 [`hackage.haskell.org/package/gloss`](https://hackage.haskell.org/package/gloss) 查阅文档。

GHCi 在显示 `gloss` 创建的图像方面不太理想，因此最好创建一个独立的程序。让我们编写一个程序，帮助我们熟悉 `Graphics.Gloss` 模块使用的默认坐标系。我们将从原点绘制一条红色的线段，终点为（100,0），以及一条绿色的线段，终点为（0,100）。因为 `gloss` 是以像素为单位来测量距离的，我们使用 100 这样线段就足够长，能够在屏幕上看到。

```
{-# OPTIONS -Wall #-}

import Graphics.Gloss

displayMode :: Display
displayMode = InWindow "Axes" (1000, 700) (10, 10)

axes :: Picture
axes = Pictures [Color red   $ Line [(0,0),(100,  0)]
                ,Color green $ Line [(0,0),(  0,100)]
                ]

main :: IO ()
main = display displayMode black axes
```

如常，我们打开警告。然后导入 `Graphics.Gloss` 模块。我们需要这么做，因为接下来的代码使用了类型 `Display` 和 `Picture`；数据构造器 `InWindow`、`Pictures`、`Color` 和 `Line`；常量 `red`、`green` 和 `black`；以及函数 `display`。这些名称都是在 `Graphics.Gloss` 模块中定义的。如果没有 `import` 语句，每次使用这些名称时都会出现“变量不在作用域”错误。

我们定义了一个常量`displayMode`来保存`display`函数所需的`Display`类型的值。我们将窗口命名为“Axes”，该窗口将由`display`函数打开。我们请求窗口宽度为 1000 像素，高度为 700 像素，并要求将窗口位置设置为距原点 10 像素上方和 10 像素右侧。

我们定义了一个名为`axes`的常量，用来保存我们想要制作的`Picture`。我们使用数据构造器`Pictures`来生成这幅图像，它提供了一种将图像列表组合成单一图像的方法。我们可以在 GHCi 中查看一些未明确指定类型的内容的类型。

```
Prelude Graphics.Gloss> :t Line [(0,0),(100,0)]
Line [(0,0),(100,0)] :: Picture
Prelude Graphics.Gloss> :t Color green $ Line [(0,0),(0,100)]
Color green $ Line [(0,0),(0,100)] :: Picture
```

这里的`main`函数使用`display`函数来生成图像。我们将`displayMode`、背景色`black`和图像`axes`传递给`display`。

当我们使用第十二章中描述的三种方法之一编译并运行上述程序时，我们应该能看到一条红色的水平线和一条绿色的垂直线。默认的`gloss`坐标系中，x 轴朝右，y 轴朝上。根据你的操作系统，可能需要按两次 CTRL-C 才能关闭图形窗口。

`gloss`包没有提供原生的圆盘或填充圆形图像。作为`display`函数的第二个示例，让我们制作一幅蓝色圆圈和红色圆盘并排的图像。

```
{-# OPTIONS -Wall #-}

import Graphics.Gloss

displayMode :: Display
displayMode = InWindow "My Window" (1000, 700) (10, 10)

blueCircle :: Picture
blueCircle = Color blue (Circle 100)

disk :: Float -> Picture
disk radius = ThickCircle (radius / 2) radius

redDisk :: Picture
redDisk = Color red (disk 100)

wholePicture :: Picture
wholePicture = Pictures [Translate (-120) 0 blueCircle
                        ,Translate   120  0 redDisk
                        ]

main :: IO ()
main = display displayMode black wholePicture
```

在这里，我们使用了与之前相同的警告、导入和`displayMode`行。常量`blueCircle`是一个半径为 100 像素的蓝色圆圈。

由于`gloss`没有提供生成圆盘的函数，我们将自己编写一个。我们的`disk`函数使用`gloss`内建的`ThickCircle`函数来生成圆盘。`ThickCircle`接受半径和厚度作为输入。在这里，我们选择将厚圆的半径设置为圆盘所需半径的一半，并将厚度设置为圆盘的完整半径。这个圆圈非常厚，中心没有留下空洞，从而形成了一个圆盘。

常量`redDisk`是一个半径为 100 像素的红色圆盘。常量`wholePicture`使用`Picture`类型的`Translate`数据构造器将圆圈向左移动，圆盘向右移动。`main`函数与上一个程序非常相似，只不过现在我们显示的是`wholePicture`。

当我们运行程序时，我们应该能看到一个蓝色圆圈位于一个大小相同的红色圆盘的左侧。

#### 创建二维动画

给定一个随时间变化的图像，`animate`函数可以生成动画。让我们在 GHCi 中查看`animate`的类型。

```
Prelude> :m Graphics.Gloss
Prelude Graphics.Gloss> :t animate
animate :: Display -> Color -> (Float -> Picture) -> IO ()
```

与`display`相比，类型的区别在于`display`中的`Picture`被`animate`中的`Float -> Picture`所替代。`animate`函数使用`Float`来描述时间，因此`Float -> Picture`类型的表达式是从时间到图像的函数，或者说是时间函数的图像。

这是如何使用`animate`的示例：

```
{-# OPTIONS -Wall #-}

import Graphics.Gloss

displayMode :: Display
displayMode = InWindow "My Window" (1000, 700) (10, 10)

disk :: Float -> Picture
disk radius = ThickCircle (radius / 2) radius

redDisk :: Picture
redDisk = Color red (disk 25)

projectileMotion :: Float -> Picture
projectileMotion t = Translate (xDisk t) (yDisk t) redDisk

xDisk :: Float -> Float
xDisk t = 40 * t

yDisk :: Float -> Float
yDisk t = 80 * t - 4.9 * t**2

main :: IO ()
main = animate displayMode black projectileMotion
```

`projectileMotion` 函数接受一个 `Float` 类型的时间作为输入，并通过将红色圆盘水平移动 `xDisk t`，垂直移动 `yDisk t` 来生成一个 `Picture`。`xDisk` 和 `yDisk` 函数明确地给出了时间的函数。

当我们编译并运行这段代码时，我们会看到一个经历抛物线运动的红色圆盘。一个米被表示为一个像素，现实世界中的一秒钟在动画中也等于一秒钟。抛物线的初始 x 分量速度为 40 米/秒，初始 y 分量速度为 80 米/秒。

#### 创建一个二维模拟

`gloss` 包的 `simulate` 函数允许用户在没有明确描述为时间函数的画面函数时，创建动画。让我们询问 GHCi `simulate` 的类型。

```
Prelude> :m Graphics.Gloss
Prelude Graphics.Gloss> :t simulate
simulate
  :: Display
     -> Color
     -> Int
     -> model
     -> (model -> Picture)
     -> (Graphics.Gloss.Data.ViewPort.ViewPort
         -> Float -> model -> model)
     -> IO ()
```

`simulate` 函数需要六个信息项。前两个，显示模式（类型为 `Display`）和背景颜色（类型为 `Color`），与 `display` 和 `animate` 中相同。第三个信息项是速率（类型为 `Int`），表示模拟运行的更新频率（每秒更新次数）。第四个信息项是一个类型变量 `model`，而不是具体类型。我们可以通过 `model` 以小写字母开头来判断它是类型变量；它不能是常量或函数，因为它位于需要类型的类型签名中。我们，`simulate` 函数的使用者，决定为 `model` 选择什么类型。我们需要一个类型，可以保存我们正在模拟的系统的*状态*，也就是在任意时刻（1）生成一个画面所需的信息，以及（2）随着时间的推移，确定接下来会发生什么的必要信息。这种状态的概念将在本书的第二部分和第三部分的物理学描述中发挥重要作用。`simulate` 函数所需的 `model` 类型的值是要显示的情况的初始状态。

第五个信息项（类型为 `model -> Picture`）是一个函数，它描述了给定 `model` 类型的值时，应该生成什么样的画面。`simulate` 所需的第六个信息项是一个函数（类型为 `Viewport -> Float -> model -> model`），它描述了系统的状态如何随时间推进。这里的 `Float` 表示时间步长，而我们不会使用 `Viewport`。

示例 13-1 给出了一个完整的程序，展示了如何使用 `simulate` 函数。

```
{-# OPTIONS -Wall #-}

import Graphics.Gloss

displayMode :: Display
displayMode = InWindow "My Window" (1000, 700) (10, 10)

-- updates per second of real time
rate :: Int
rate = 2

disk :: Float -> Picture
disk radius = ThickCircle (radius / 2) radius

redDisk :: Picture
redDisk = Color red (disk 25)

type State = (Float,Float)

initialState :: State
initialState = (0,0)

displayFunc :: State -> Picture
displayFunc (x,y) = Translate x y redDisk

updateFunc :: Float -> State -> State
updateFunc dt (x,y) = (x + 10 * dt, y - 5 * dt)

main :: IO ()
main = simulate displayMode black rate initialState displayFunc
       (\_ -> updateFunc)
```

*示例 13-1：使用 gloss 包中的 simulate 函数的示例*

显示模式和背景颜色与之前相同。我们定义了一个常量 `rate` 来保存模拟的速率。对于类型变量 `model`，我们选择了 `(Float,Float)` 并给它定义了类型别名 `State`。这个状态用来表示红色圆盘当前位置的 (x, y) 坐标。状态的初始值在 `initialState` 中定义。

模拟的核心包含在两个函数`displayFunc`和`updateFunc`中。第一个函数负责根据状态生成图像。在这个例子中，我们使用状态中的(x, y)坐标来将红色圆盘沿*x*轴平移并沿*y*轴上移。显示函数只关心当前的状态（*x*和*y*的当前值）。它与图像如何随时间变化无关。

更新函数`updateFunc`解释了状态如何随时间变化。我们需要给出一个规则，说明新状态如何通过旧状态和时间步长`dt`计算得出。在这个例子中，我们将 x 值增加 10 像素/秒，并将 y 值减少 5 像素/秒。

当我们运行程序时，应该看到红色圆盘随着模拟的推进向右和向下移动。由于我们选择了每秒 2 次更新的速率，模拟会显得有些生硬，所以你会看到每次更新是一个离散的运动。试着增加更新速率来获得更流畅的动画。高清电视使用每秒 24 到 60 帧，所以你不需要超过这个范围。如果你所在建筑的灯光变暗了，那说明你选择的帧率太高。

让我们再看一个`simulate`函数的例子，看看我们之前用`animate`实现的抛体运动在使用`simulate`时会是什么样子。匀速运动和抛体运动的区别在于，抛体运动中的速度是会变化的。为了允许速度变化，我们需要扩展状态中的信息，同时包含红色圆盘的位置和速度。为此，我们在 Listing 13-2 的代码中定义了类型同义词`Position`、`Velocity`和`State`。

现在我们的`initialState`需要同时包含初始位置`(0,0)`和初始速度`(40,80)`。初始的 x 分量速度是 40 米/秒，初始的 y 分量速度是 80 米/秒。

我们的显示函数在意义上不需要改变，和之前的模拟一样，红色圆盘的显示仍然只依赖于圆盘的位置，而与当前速度无关。然而，由于状态的类型发生了变化，`displayFunc`函数需要做一些语法上的修正。语法上的修正是将参数`(x, y)`替换为`((x, y), _)`，以反映新的状态类型。如果我们完全不修改函数，编译器会认为参数`(x, y)`表示位置的 x 值和速度的 y 值。这样会产生类型错误，提示 x 的预期类型是`Float`，而实际类型是`Position`。`Position`的“实际类型”来自于`displayFunc`的类型签名，而`Float`的“预期类型”则来自于 x 作为`Translate`函数的参数在`displayFunc`中的使用。

让我们实现这些变化，结果请参见 Listing 13-2。

```
{-# OPTIONS -Wall #-}

import Graphics.Gloss

displayMode :: Display
displayMode = InWindow "My Window" (1000, 700) (10, 10)

-- updates per second of real time
rate :: Int rate = 24

disk :: Float -> Picture
disk radius = ThickCircle (radius / 2) radius

redDisk :: Picture
redDisk = Color red (disk 25)

type Position = (Float,Float)
type Velocity = (Float,Float)
type State = (Position,Velocity)

initialState :: State
initialState = ((0,0),(40,80))

displayFunc :: State -> Picture
displayFunc ((x,y),_) = Translate x y redDisk

updateFunc :: Float -> State -> State
updateFunc dt ((x,y),(vx,vy))
   = (( x + vx * dt, y +  vy * dt)
     ,(vx         ,vy - 9.8 * dt))

main :: IO ()
main = simulate displayMode black rate initialState displayFunc
       (\_ -> updateFunc)
```

*Listing 13-2：使用`simulate`函数生成抛体运动的示例*

更新函数是所有操作发生的地方。位置的 x 和 y 分量根据当前的速度更新。速度的 x 和 y 分量根据加速度的分量更新。加速度的 x 分量为 0，所以速度的 x 分量保持不变。加速度的 y 分量为–9.8 米/秒²，因此我们使用它更新速度的 y 分量，假设 1 米等于我们模拟中的 1 像素。

当我们运行这个程序时，结果应该与我们用`animate`编写的抛体程序相同。

注意使用`animate`和`simulate`时，在生成抛体运动动画所需信息方面的差异。使用`animate`时，我们需要有明确的关于位置随时间变化的表达式。使用`simulate`时，我们提供了等效的信息，但看起来我们提供的更少。状态更新过程是数值求解运动方程的强大工具。在本书的第二部分和第三部分中，我们将更深入地利用这个工具。

### 3D 动画

`not-gloss`包提供了四个主要函数，它们的名称与`gloss`中的相同：`display`、`animate`、`simulate`和`play`。与`gloss`中一样，第一个用于静止图像，第二个和第三个用于随时间变化的图像，第四个用于随时间和用户输入变化的图像。我们主要关心前三个函数。这些函数的类型与`gloss`函数的类型不同，部分原因是`not-gloss`包的作者不同于`gloss`包的作者。两个包之间有相似之处，但也有我们将指出的差异。

#### 显示 3D 图像

让我们检查一下`display`的类型。正如`gloss`包中必须导入名为`Graphics.Gloss`的模块才能使用其函数一样，`not-gloss`包也有一个名为`Vis`的模块，我们必须导入它才能使用。

```
Prelude> :m Vis
Prelude Vis> :t display
display :: Real b => Options -> VisObject b -> IO ()
```

如果我们查询`Real`类型类，我们会发现`Real`适用于可以转换为有理数的数值类型：

```
Prelude Vis> :i Real
class (Num a, Ord a) => Real a where
  toRational :: a -> Rational
  {-# MINIMAL toRational #-}
   -- Defined in 'GHC.Real'
instance Real Word -- Defined in 'GHC.Real'
instance Real Integer -- Defined in 'GHC.Real'
instance Real Int -- Defined in 'GHC.Real'
instance Real Float -- Defined in 'GHC.Float'
instance Real Double -- Defined in 'GHC.Float'
```

我们最喜欢的`Real`类型类实例是`R`（或`Double`）。除非有特别的理由选择其他类型，否则我们将默认选择它。如果`display`类型中的类型变量`b`是`R`，则`display`的类型如下：

```
display :: Options -> VisObject R -> IO ()
```

显示函数要求我们提供两件事：一个类型为`Options`的对象，以及要显示的对象（类型`VisObject R`）。返回类型`IO ()`意味着计算机将执行某些操作（在这种情况下是显示对象）。

有哪些类型是`VisObject R`？`not-gloss`包提供了一个长长的列表，包括球体、立方体、线条、文本等等。你可以通过访问[`hackage.haskell.org`](https://hackage.haskell.org)并搜索`not-gloss`来查看文档。

下面是一个生成蓝色立方体的示例：

```
{-# OPTIONS -Wall #-}

import Vis

type R = Double

blueCube :: VisObject R
blueCube = Cube 1 Solid blue

main :: IO ()
main = display defaultOpts blueCube
```

常量`defaultOpts`由`Vis`模块提供，作为一组默认选项。你可以像以前一样将此代码编译成独立程序。当你运行程序时，一个包含蓝色立方体的显示窗口将打开。显示窗口打开后，按 e 键放大，按 q 键缩小。你还可以使用鼠标旋转立方体。这些是`not-gloss`的标准功能，我们无需编写代码来实现。

下一个程序将帮助我们熟悉`Vis`模块使用的默认坐标系统。我们将绘制一条从原点到点(1, 0, 0)的红色线段，一条从原点到点(0, 1, 0)的绿色线段，以及一条从原点到点(0, 0, 1)的蓝色线段。

```
{-# OPTIONS -Wall #-}

import Vis
import Linear

type R = Double

axes :: VisObject R
axes = VisObjects [Line Nothing [V3 0 0 0, V3 1 0 0] red
                  ,Line Nothing [V3 0 0 0, V3 0 1 0] green
                  ,Line Nothing [V3 0 0 0, V3 0 0 1] blue
                  ]

main :: IO ()
main = display defaultOpts axes
```

在这里我们导入`Linear`模块，以便使用`V3`构造函数。`Linear`模块定义了几种类型的向量；`V3`是`Vis`模块使用的类型。`Nothing`表示使用默认的线宽（尝试将`Nothing`替换为(Just 5)，以获得更粗的线宽）。

当我们编译并运行刚才展示的程序时，我们会看到三维坐标系的坐标轴。我们看到`not-gloss`的默认方向是 x 轴指向右方并朝向观察者，y 轴指向左方并朝向观察者，z 轴指向下方。

个人来说，我认为 z 轴正方向指向下方令人不安且无法接受。我喜欢认为自己是一个灵活的人，但这真的有些过分了。（`not-gloss`的作者 Greg Horn 告诉我，z 向下的约定在航空航天行业中是标准的。）幸运的是，`not-gloss`提供了让我们按自己的方式旋转物体的工具。我喜欢将 x 轴主要指向页面外，y 轴指向右侧，z 轴指向上方。以下是一个实现这一点的程序：

```
{-# OPTIONS -Wall #-}

import Vis
import Linear
import SpatialMath

type R = Double

axes :: VisObject R
axes = VisObjects [Line Nothing [V3 0 0 0, V3 1 0 0] red
                  ,Line Nothing [V3 0 0 0, V3 0 1 0] green
                  ,Line Nothing [V3 0 0 0, V3 0 0 1] blue
                  ]

orient :: VisObject R -> VisObject R
orient pict = RotEulerDeg (Euler 270 180 0) $ pict

main :: IO ()
main = display defaultOpts (orient axes)
```

我们像之前一样导入`Vis`和`Linear`，但在这里我们还导入了`SpatialMath`，这样我们就可以使用`Euler`来进行三维旋转，使用欧拉角。`axes`图像没有变化。我们定义了一个`orient`函数，该函数接受一个图片作为输入，并返回一个重新定向后的图片作为输出。为此，我们使用`VisObject`类型的`RotEulerDeg`数据构造函数，执行由欧拉角指定的旋转。在这种情况下，欧拉角意味着我们首先绕 x 轴旋转 0^∘，然后绕 y 轴旋转 180^∘，最后绕 z 轴旋转 270^∘。等效地，我们可以将其视为首先绕 z 轴旋转 270^∘，然后绕*旋转后的* y 轴旋转 180^∘，最后绕旋转后的 x 轴旋转 0^∘。

最后，我们将`orient axes`传递给`display`，作为要显示的图片。如果你喜欢这种方向系统，你可以在显示之前将任何图片传递给`orient`函数，作为使用该坐标系统的一种方式。你甚至可以定义自己的显示函数，来为你进行重新定向。

```
myDisplay :: VisObject R -> IO ()
myDisplay pict = display defaultOpts (orient pict)
```

#### 创建 3D 动画

让我们看一下`animate`的类型。

```
Prelude> :m Vis
Prelude Vis> :t animate
animate :: Real b => Options -> (Float -> VisObject b) -> IO ()
```

`animate` 的类型与 `display` 的类型相同，区别在于 `display` 中的 `VisObject b` 被 `animate` 中的 `Float -> VisObject b` 所替代。`animate` 不要求我们提供一张图片，而是要求我们提供一个从时间到图片的函数。`animate` 函数要求我们使用 `Float` 来表示时间的实数值。

以下是一个旋转蓝色立方体的动画，立方体绕着 x 轴逆时针旋转，我最喜欢的坐标系为（x 轴指向屏幕外，y 轴指向右，z 轴指向屏幕上方）：

```
{-# OPTIONS -Wall #-}

import Vis
import SpatialMath

rotatingCube :: Float -> VisObject Float
rotatingCube t = RotEulerRad (Euler 0 0 t) (Cube 1 Solid blue)

orient :: VisObject Float -> VisObject Float
orient pict = RotEulerDeg (Euler 270 180 0) $ pict

main :: IO ()
main = animate defaultOpts (orient . rotatingCube)
```

注意 `rotatingCube` 和 `orient` 之间的函数组合。`rotatingCube` 函数接受一个数字作为输入，输出一张图片。`orient` 函数接受一张图片作为输入，输出一张（重新定向的）图片。组合后的函数 `orient . rotatingCube` 接受一个数字作为输入，输出一张图片，这正是 `animate` 所需要的函数类型。

#### 制作 3D 仿真

`not-gloss` 函数 `simulate` 允许用户在没有明确描述图片随时间变化的函数时，制作动画。我们来询问 GHCi `simulate` 的类型。

```
Prelude> :m Vis
Prelude Vis> :t simulate
simulate
  :: Real b =>
     Options
     -> Double
     -> world
     -> (world -> VisObject b)
     -> (Float -> world -> world)
     -> IO ()
```

`simulate` 函数需要五个信息。第一个信息（类型 `Options`）与 `display` 和 `animate` 中的一样。第二个信息（一个 `Double`）是时间步长，单位是每次更新之间的秒数，表示动画显示中连续帧之间的时间间隔。请注意与 `gloss` 库的不同：`gloss` 要求以每秒更新次数为速率，而 `not-gloss` 要求以每次更新的秒数为时间步长。

第三个信息是要显示的情况的初始状态。类型变量 `world` 代表一个由用户选择的类型，用来描述情况的状态，类似于 `gloss` 函数 `simulate` 中使用的类型变量 `model`。

第四个信息（类型 `world -> VisObject b`）是一个显示函数，它描述了给定类型 `world` 值时应生成什么图片。这个显示函数与 `gloss` 的显示函数非常相似。

最后，`simulate` 需要的第五个信息是一个函数（类型 `Float -> world -> world`），它描述了系统状态如何随时间推进。这个类型中的 `Float` 代表自仿真开始以来经过的总时间。这与 `gloss` 不同，`gloss` 中类似项中的 `Float` 描述的是自上一帧以来的时间步长。

清单 13-3 演示了 `simulate` 函数如何使用我们提供的第五个参数——更新函数。代码的目的是通过实验确定 `simulate` 函数如何使用我们提供的 `Float -> world -> world` 类型的函数。如果你不习惯使用高阶函数，这可能会显得很奇怪。通常，我们编写函数供自己使用，或者使用别人写的函数。但当别人为我们编写一个高阶函数并且这个高阶函数接受一个用户定义的函数作为输入时，我们可能会想知道高阶函数打算如何使用我们提供的用户定义函数。（我们可以阅读高阶函数的代码或文档，但这里我们将通过实验来弄清楚。）

奇怪的是，我们在 清单 13-3 中编写了一个函数 `updateFunc`，但我们并不直接使用这个函数。我们并不决定传递给 `updateFunc` 的 `Float` 值；是另一个函数 `simulate` 来决定的。

```
{-# OPTIONS -Wall #-}

import Vis

type State = (Int,[Float])

-- seconds / update
dt :: Double dt = 0.5

displayFunc :: State -> VisObject Double
displayFunc (n,ts) = Text2d (show n ++ " " ++ show (take 4 ts))
                     (100,100) Fixed9By15 orange

updateFunc :: Float -> State -> State
updateFunc t (n,ts) = (n+1,t:ts)

main :: IO ()
main = simulate defaultOpts dt (0,[]) displayFunc updateFunc
```

*清单 13-3：使用 `not-gloss` 库中的 `simulate` 函数。本代码的目的是通过实验确定 `simulate` 如何使用 `updateFunc`。*

我们通过导入 `Vis` 模块来开始代码。对于类型变量 `world`，我们选择了一个对 `(Int,[Float])` 的元组，其中 `Int` 用于表示自仿真开始以来所执行的更新次数，而浮点数列表则表示传递给更新函数 `updateFunc` 的时间值。我们并不选择这些时间值；是 `simulate` 来决定的。

我们设置了一个时间步长 `dt`，它是半秒钟。`displayFunc` 定义了如何从 `State` 生成图像。它使用了 `VisObject` 类型中的 `Text2d` 数据构造器，你可以查看 `not-gloss` 中 `simulate` 函数的文档来了解更多信息。

更新函数 `updateFunc` 跟踪两件事：它被调用的次数和它所使用的 `Float` 值。每次调用 `updateFunc` 时，它会将调用次数加一，并将最新的 `Float` 值添加到列表的前面。

当我们运行这个程序时，可以看到更新次数以每秒两次的速度增加，并且看到传递进去的时间值在不断增大，从而确认了更新函数以仿真开始以来的时间作为输入的说法。

### 总结

在本章中，我们探讨了几种生成二维和三维图形及动画的方法。我们提供了展示每种图形功能的代码，这些功能将在本书后续章节中帮助我们在讨论和书写物理学内容时提供可视化支持。

本章结束后，我们已经完成了书的第一部分—这是对函数式编程思想的一般介绍，特别是对 Haskell 编程语言的介绍。在第二部分中，我们将探索牛顿力学，目标是预测受力的一个或多个物体的运动。牛顿力学的核心原理是牛顿第二定律，这是下一章的主题。

### 习题

**习题 13.1.** 查阅`gloss`文档中的`Picture`类型，使用`display`函数创建一个有趣的图形。结合线条、圆形、文字、颜色和你喜欢的任何元素。发挥创意。

**习题 13.2.** 使用`animate`制作一个简单的动画。发挥创意。

**习题 13.3.** 使用`animate`使红色圆盘左右摆动。然后，稍微修改代码，使红色圆盘沿圆形轨道运动。你能让红色圆盘沿椭圆轨道移动吗？

**习题 13.4.** 使用`animate`制作与我们在示例 13-1 中通过`simulate`实现的红色圆盘相同的运动。

**习题 13.5.** 使用`simulate`做一些你认为有趣的事情。发挥创意。

**习题 13.6.** 在示例 13-2 中的二维抛体运动示例中，现实世界中的一米由动画中的一个像素表示。修改代码，使一米由 10 个像素表示。可以自由更改初始速度分量，以便抛体不会立即飞出屏幕。

**习题 13.7.** 挑战性习题：尝试使用`simulate`使红色圆盘左右摆动，而不显式给它提供像 sin 或 cos 这样的振荡函数。我们将在书的第二部分展示如何实现这一点。

**习题 13.8.** 重写三维坐标轴代码，使 x 轴指向右，y 轴指向上，z 轴指向页面外。这是我第二喜欢的坐标系。

**习题 13.9.** 修改旋转立方体的动画，使旋转围绕 x 轴顺时针进行，而不是逆时针。

**习题 13.10.** 编写一个实验程序，类似于示例 13-3，使用`gloss`函数`simulate`来理解`gloss`的`simulate`如何使用更新函数。使用与示例 13-3 中相同的`updateFunc`和`State`表达式。你需要更改`displayFunc`和`main`的值。使用`rate`为 2，而不是`dt`为 0.5。当你运行这个时，你应该会看到`gloss`的`simulate`传入的时间步长都接近 0.5。
