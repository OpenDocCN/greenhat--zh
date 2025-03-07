# 9

# 理解 OpenGL

![](img/nsp-venkitachalam503045-circle-image.jpg)

在这个项目中，你将创建一个简单的程序，通过 OpenGL 和 GLFW 显示一个纹理映射的方形。OpenGL 是一种图形处理单元（GPU）的软件接口，GLFW 是一个窗口工具包。你还将学习如何使用类似 C 的 OpenGL 着色语言（GLSL）编写*着色器*——在 GPU 中执行的代码。着色器为 OpenGL 中的计算带来了巨大的灵活性。我将向你展示如何使用 GLSL 着色器来转换和着色几何图形，创建一个旋转的纹理多边形（如图 9-1 所示）。

GPU 被优化为在巨大数据量上重复执行相同操作，并行处理，这使得它们在渲染计算机图形时比中央处理单元（CPU）快得多。此外，GPU 也被用于通用计算，现在有专门的语言可以让你利用 GPU 硬件进行各种应用。在这个项目中，你将利用 GPU、OpenGL 和着色器。

![](img/nsp-venkitachalam503045-f09001.jpg)

图 9-1：本章项目的最终图像——一个旋转的多边形，包含星星图像。这个方形多边形的边界通过着色器被裁剪成一个黑色圆形。

Python 是一种出色的“粘合”语言。对于用其他语言（如 C）编写的库，有大量的 Python *绑定* 可供使用，这些绑定使你能够在 Python 中使用这些库。在本章以及第十章和第十一章中，你将使用 `PyOpenGL`，它是 Python 对 OpenGL 的绑定，用于创建计算机图形。

下面是本项目中介绍的一些概念：

+   • 使用 GLFW 窗口库进行 OpenGL

+   • 使用 GLSL 编写顶点和片段着色器

+   • 执行纹理映射

+   • 使用 3D 变换

首先，让我们来看看 OpenGL 是如何工作的。

注意：几年前，OpenGL 经历了一个重大转变。从使用固定功能图形管线转变为使用可编程管线，并引入了专用的着色语言。我们称后者为*现代 OpenGL*，并且这就是我们在本书中将要使用的版本。具体来说，我们将使用 OpenGL 版本 4.1。

## OpenGL 工作原理

现代 OpenGL 通过一系列操作使图形出现在你的屏幕上，这些操作通常被称为*3D 图形管线*。图 9-2 展示了 OpenGL 3D 图形管线的简化表示。

![](img/nsp-venkitachalam503045-f09002.jpg)

图 9-2：简化版 OpenGL 图形管线

计算机图形学的核心归结为为屏幕上的像素计算颜色值。假设你想显示一个三角形。在管线的第一步中，你通过在 3D 空间中定义三角形的顶点并指定与每个顶点相关的颜色来定义 3D 几何图形。这些顶点和颜色被存储在名为*顶点缓冲对象 (VBOs)*的数据结构中。接下来，你需要变换这些顶点：第一次变换将顶点放置在 3D 空间中，第二次变换将 3D 坐标投影到 2D 空间，以便在 2D 屏幕上显示。此步骤中，还会根据光照等因素计算相应顶点的颜色值，通常在被称为*顶点着色器*的代码中进行计算。

接下来，几何图形会被*光栅化*（从 3D 表示转换为 2D 像素），并且对于每个像素（或更准确地说是*片段*），会执行另一段代码，称为*片段着色器*。就像顶点着色器处理 3D 顶点一样，片段着色器会在光栅化后处理 2D 片段。我之所以说*片段*而不是*像素*，是因为像素是屏幕上显示的内容，而片段是片段着色器计算后的输出，取决于管线中的下一步操作，片段可能在成为屏幕上的像素之前被丢弃。

最后，每个片段会经过一系列帧缓冲操作，在这些操作中，它会进行*深度缓冲测试*（检查一个片段是否遮挡另一个片段）、*混合*（将两个带有透明度的片段混合）以及其他操作，这些操作将当前颜色与该位置帧缓冲中已有的颜色进行合并。这些变化最终会反映在最终的帧缓冲中，通常会显示在屏幕上。

### 几何原始图形

因为 OpenGL 是一个低级图形库，你不能直接要求它绘制一个立方体或球体，尽管在其基础上构建的库可以为你完成这些任务。OpenGL 只理解低级几何原始图形，如点、线和三角形。

现代 OpenGL 只支持原始类型 `GL_POINTS`、`GL_LINES`、`GL_LINE_STRIP`、`GL_LINE_LOOP`、`GL_TRIANGLES`、`GL_TRIANGLE_STRIP` 和 `GL_TRIANGLE_FAN`。图 9-3 显示了原始图形顶点的组织方式。每个顶点都有一个 3D 坐标，例如 (*x*, *y*, *z*)。

![](img/nsp-venkitachalam503045-f09003.jpg)

图 9-3：OpenGL 原始图形

要在 OpenGL 中绘制一个球体，首先需要数学上定义球体的几何形状，并计算其 3D 顶点。然后，将顶点组合成基本的几何原始图形；例如，你可以将每组三个顶点组合成一个三角形。接着，使用 OpenGL 渲染这些顶点。

### 3D 变换

你不能学会计算机图形学而不学习 3D 变换。从概念上来说，这些变换是相当简单的。你有一个物体——你能对它做什么？你可以移动它，拉伸（或压缩）它，或者旋转它。你也可以对它做其他事情，但这三项任务——平移、缩放和旋转——是最常见的对物体执行的操作或变换。除了这些常见的变换外，你还会使用透视投影将 3D 物体映射到屏幕的 2D 平面上。这些变换都应用于你试图变换的物体的坐标。

虽然你可能熟悉形式为（*x*，*y*，*z*）的 3D 坐标，但在 3D 计算机图形学中，你使用的是形式为（*x*，*y*，*z*，*w*）的坐标，称为*齐次坐标*。（这些坐标来源于数学的一个分支——*投影几何*，这超出了本书的范围。）齐次坐标允许你将常见的 3D 变换，如平移、缩放和旋转，表示为 4×4 矩阵。但就这些 OpenGL 项目而言，你只需要知道的是，齐次坐标（*x*，*y*，*z*，*w*）等价于 3D 坐标（*x*/*w*，*y*/*w*，*z*/*w*，1.0）。一个 3D 点（1.0，2.0，3.0）可以用齐次坐标表示为（1.0，2.0，3.0，1.0）。

这是一个使用 4×4 矩阵的 3D 变换示例。看看矩阵乘法如何将一个点（*x*，*y*，*z*，1.0）转换为（*x* + *t* [x]，*y* + *t* [y]，*z* + *t* [z]，1.0）：

![](img/nsp-venkitachalam503045-m09001.jpg)

由于这个操作是将一个点在空间中进行平移，因此涉及的 4×4 矩阵被称为*平移矩阵*。

现在让我们来看另一个有用的 3D 变换矩阵——旋转矩阵。以下矩阵将一个点（*x*，*y*，*z*，1.0）绕 x 轴逆时针旋转θ弧度：

![](img/nsp-venkitachalam503045-m09002.jpg)

但有一点需要记住：如果你打算在着色器代码中应用这个旋转，矩阵将以*列主序格式*存储，这意味着你应该按如下方式声明它：

// 旋转变换

mat4 rot =  mat4(

vec4(1.0,  0.0,          0.0,         0.0),

vec4(0.0,  cos(uTheta),  sin(uTheta), 0.0),

vec4(0.0, -sin(uTheta),  cos(uTheta), 0.0),

vec4(0.0,  0.0,          0.0,         1.0)

);

请注意，在代码中，与*R* [θ] [,][x] 的定义相比，矩阵沿对角线翻转。

在 OpenGL 中你会经常遇到两个术语：*模型视图*和*投影*变换。随着现代 OpenGL 引入可自定义着色器，模型视图和投影变换变成了通用的变换。在老版本的 OpenGL 中，模型视图变换用于将 3D 模型定位到空间中，而投影变换则用于将 3D 坐标映射到 2D 表面进行显示，正如你将看到的那样。模型视图变换是用户自定义的变换，让你定位 3D 物体，而投影变换则是投影变换，将 3D 映射到 2D。

两种最常用的 3D 图形投影变换是*正交投影*和*透视投影*，但在这里你将只使用透视投影，它由*视野*（眼睛能看到的范围）、*近平面*（离眼睛最近的平面）、*远平面*（离眼睛最远的平面）和*纵横比*（近平面宽高的比例）定义。这些参数共同构成了一个相机模型，用于定义如何将 3D 物体映射到 2D 屏幕上，如图 9-4 所示。图中的截头金字塔就是*视锥体*。*眼睛*是你放置相机的 3D 位置。（对于正交投影，眼睛的位置会在无限远处，金字塔将变成一个矩形立方体。）

![](img/nsp-venkitachalam503045-f09004.jpg)

图 9-4：一个透视投影相机模型

一旦透视投影完成，并且在光栅化之前，图形原语会在图 9-4 所示的近平面和远平面之间进行裁剪（或剔除）。近平面和远平面的选择确保了你希望在屏幕上显示的 3D 物体位于视锥体内；否则，它们将被裁剪掉。

### 着色器

你已经了解了着色器如何融入现代 OpenGL 可编程图形管线。现在，让我们看一个简单的顶点着色器和片段着色器，了解 GLSL 如何工作。

#### 一个顶点着色器

这里是一个简单的顶点着色器，用于计算顶点的位置和颜色：

❶ # version 410 core

❷ in vec3 aVert;

❸ uniform mat4 uMVMatrix;

❹ uniform mat4 uPMatrix;

❺ out vec4 vCol;

void main() {

// 应用变换

❻ gl_Position = uPMatrix * uMVMatrix * vec4(aVert, 1.0);

// 设置颜色

❼ vCol = vec4(1.0, 0.0, 0.0, 1.0);

}

你首先将着色器使用的 GLSL 版本设置为 4.1 版本❶。然后你定义了一个名为`aVert`的`vec3`类型（3D 向量）输入，用于顶点着色器，使用了`in`关键字❷。接下来，你定义了两个`mat4`类型（4×4 矩阵）的变量，它们分别对应模型视图❸和投影❹矩阵。这些变量的`uniform`前缀表示它们在给定渲染调用的一组顶点上，顶点着色器执行期间不会发生变化。你使用`out`前缀来定义顶点着色器的输出，它是一个`vec4`类型的颜色变量（一个 4D 向量，用于存储红、绿、蓝和透明度通道）❺。

现在你来到了`main()`函数，这里是顶点着色器程序的开始。`gl_Position`的值是通过使用传入的统一矩阵❻变换输入的`aVert`来计算的。GLSL 变量`gl_Position`用于存储变换后的顶点。你通过使用值(1, 0, 0, 1)❼将顶点着色器的输出颜色设置为红色且无透明度。你将在管线中的下一个着色器中使用这个值作为输入。

#### 片段着色器

现在让我们来看一个简单的片段着色器，它根据传入的顶点颜色来计算片段颜色：

❶ # version 410 core

❷ in vec4 vCol;

❸ out vec4 fragColor;

void main() {

// 使用顶点颜色

❹ fragColor = vCol;

}

在设置了着色器中使用的 GLSL 版本❶之后，你将`vCol`设置为片段着色器的输入❷。这个变量`vCol`是从顶点着色器输出的。(记住，顶点着色器在 3D 场景中的每个顶点上执行，而片段着色器在屏幕上的每个片段上执行。)你还设置了片段着色器的输出颜色变量`fragColor`❸。

在光栅化阶段（发生在顶点着色器和片段着色器之间），OpenGL 将变换后的顶点转换为片段，并通过插值计算位于顶点之间的片段的颜色；在之前的代码中，`vCol`就是这个插值后的颜色。你将片段着色器的输出设置为传入片段着色器的插值颜色❹。默认情况下，且在大多数情况下，片段着色器的预期输出是屏幕，你设置的颜色最终会显示在那里（除非它受到在图形管线最后阶段发生的操作（如深度测试）的影响）。

为了让 GPU 执行着色器代码，着色器代码需要被编译并链接为硬件能够理解的指令。OpenGL 提供了方法来完成这一过程，并报告详细的编译器和链接器错误，帮助你开发着色器代码。编译过程还会生成一个包含着色器中声明的变量位置或索引的表格，以便你将它们连接到 Python 代码中的变量。

### 顶点缓冲区

*顶点缓冲区*是 OpenGL 着色器使用的一个重要机制。现代图形硬件和 OpenGL 被设计用来处理大量的 3D 几何体。因此，OpenGL 内建了几种机制来帮助将数据从程序传输到 GPU。绘制 3D 几何体的典型设置如下：

1.  1\. 为每个顶点定义坐标、颜色和其他属性的数组，用于表示 3D 几何体。

1.  2\. 创建一个顶点数组对象（VAO）并进行绑定。

1.  3\. 为每个顶点的每个属性创建顶点缓冲区对象（VBO）。

1.  4\. 绑定到 VBO 并使用预定义的数组设置缓冲区数据。

1.  5\. 指定顶点属性的数据和位置，以便在着色器中使用。

1.  6\. 启用顶点属性。

1.  7\. 渲染数据。

在你定义了顶点的 3D 几何体后，你创建并绑定到一个顶点数组对象。VAO 是一种方便的方式，将几何体分组为多个坐标、颜色等的数组。然后，对于每个顶点的每个属性，你创建一个顶点缓冲区对象，并将你的 3D 数据设置到其中。VBO 将顶点数据存储在 GPU 内存中。现在，剩下的就是将缓冲区数据连接起来，以便你可以在着色器中访问它。你可以通过调用使用着色器中变量位置的方法来完成这一步。

### 纹理映射

现在让我们来看一下纹理映射，这是一种重要的计算机图形技术，你将在本章中使用它。*纹理映射*是一种通过 2D 图像为 3D 物体（比如舞台剧中的背景）赋予场景真实感的方法。纹理通常从图像文件读取，并通过将 2D 坐标（范围为[0, 1]）映射到多边形的 3D 坐标，拉伸并覆盖到几何区域。例如，图 9-5 展示了一个图像被映射到立方体的一个面上。（我使用了`GL_TRIANGLE_STRIP`图元来绘制立方体的面，顶点的顺序通过面上的线条指示。）

![](img/nsp-venkitachalam503045-f09005.jpg)

图 9-5：纹理映射

在图 9-5 中，纹理的(0, 0)角映射到立方体面的左下角。同样，你可以看到纹理的其他角是如何被映射的，最终效果是纹理“粘贴”到这个立方体面上。立方体面的几何体本身被定义为一个三角带，顶点从底部到左上角，再从底部到右上角呈锯齿形排列。纹理是非常强大且多用途的计算机图形工具，正如你将在第十一章中看到的那样。

### OpenGL 上下文

现在让我们来谈谈如何让 OpenGL 在屏幕上绘制内容。存储所有 OpenGL 状态信息的实体被称为 *OpenGL 上下文*。上下文拥有一个可视的、类似窗口的区域用于显示 OpenGL 绘图，你可以为每个进程或应用程序运行拥有多个上下文，但每个线程一次只能有一个上下文是当前的。（幸运的是，窗口工具包会处理大部分的上下文管理。）

为了让你的 OpenGL 输出显示在屏幕上的窗口中，你需要操作系统的帮助。在这些项目中，你将使用 GLFW，一个轻量级的跨平台 C 库，它让你创建和管理 OpenGL 上下文，展示 3D 图形在窗口中，并处理诸如鼠标点击和键盘按键等用户输入。（附录 A 介绍了该库的安装细节。）

由于你使用 Python 编写代码而非 C，因此你还将使用一个 Python 绑定库来访问 GLFW（*glfw.py*，可以在本书代码仓库中的 *common* 目录下找到），该库让你能够使用 Python 来访问所有 GLFW 的功能。

## 需求

你将使用 `PyOpenGL`，一个流行的 Python 绑定库，用于 OpenGL 渲染，同时使用 `numpy` 数组表示 3D 坐标和变换矩阵。

## 代码

在这个项目中，你将构建一个简单的 Python 应用程序，用于使用 OpenGL 显示一个旋转的、带有纹理的多边形。要查看完整的项目代码，请跳转到 “完整代码”，见 第 172 页。我们简单的 OpenGL 应用程序的完整代码分布在两个文件中。本章讨论的主要项目代码在 *simpleglfw.py* 中，可以在 [`github.com/mkvenkit/pp2e/tree/main/simplegl`](https://github.com/mkvenkit/pp2e/tree/main/simplegl) 找到。辅助函数位于 *glutils.py* 文件中，可以在 GitHub 仓库的 *common* 目录中找到。

### RenderWindow 类

`RenderWindow` 类负责管理显示 OpenGL 图形的窗口的创建。它初始化 GLFW，设置 OpenGL，管理渲染，并设置回调以接收键盘输入。

#### 创建 OpenGL 窗口

`RenderWindow` 类的首要任务是设置 GLFW，以便你能拥有一个用于渲染的 OpenGL 窗口。该类的初始化代码完成了这一任务：

class RenderWindow:

"""GLFW 渲染窗口类"""

def __init__(self):

# 保存当前工作目录

cwd = os.getcwd()

# 初始化 glfw

❶ glfw.glfwInit()

# 恢复 cwd

os.chdir(cwd)

# 版本提示

❷ glfw.glfwWindowHint(glfw.GLFW_CONTEXT_VERSION_MAJOR, 4)

glfw.glfwWindowHint(glfw.GLFW_CONTEXT_VERSION_MINOR, 1)

glfw.glfwWindowHint(glfw.GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE)

glfw.glfwWindowHint(glfw.GLFW_OPENGL_PROFILE,

glfw.GLFW_OPENGL_CORE_PROFILE)

# 创建一个窗口

self.width, self.height = 800, 600

self.aspect = self.width/float(self.height)

❸ self.win = glfw.glfwCreateWindow(self.width, self.height,

b'simpleglfw')

# 使上下文成为当前上下文

❹ glfw.glfwMakeContextCurrent(self.win)

你在❶初始化了 GLFW 库，然后，从❷开始，将 OpenGL 版本设置为 OpenGL 4.1 核心配置文件。接着，创建一个尺寸为 800×600 的 OpenGL 能力窗口❸。最后，你使上下文成为当前上下文❹，然后就可以开始进行 OpenGL 调用了。

接下来，在 `__init__()` 定义内，你进行了一些初始化调用：

# 初始化 GL

❶ glViewport(0, 0, self.width, self.height)

❷ glEnable(GL_DEPTH_TEST)

❸ glClearColor(0.5, 0.5, 0.5, 1.0)

在这里，你设置了 OpenGL 渲染 3D 场景的视口或屏幕尺寸（宽度和高度）❶。然后，你开启了深度测试 `GL_DEPTH_TEST` ❷，并设置了背景色，这个背景色将会在调用 `glClear()` 渲染时显示 ❸。你选择了 50% 灰色，`alpha` 设置为 1.0（Alpha 是片段透明度的度量值，1.0 表示完全不透明）。

#### 设置回调函数

你通过为 GLFW 窗口内的用户界面事件注册回调函数，完成了 `__init__()` 的定义，这样你就可以响应键盘按键事件了：

# 设置窗口回调函数

glfw.glfwSetKeyCallback(self.win, self.onKeyboard)

这段代码为按键事件设置了回调函数。每次这些事件发生时，注册为回调的函数 `onKeyboard()` 就会被执行。现在，让我们来看一下这个键盘回调函数的定义：

def onKeyboard(self, win, key, scancode, action, mods):

# 打印 'keyboard: ', win, key, scancode, action, mods

❶ if action == glfw.GLFW_PRESS:

# 按 ESC 键退出

if key == glfw.GLFW_KEY_ESCAPE:

❷ self.exitNow = True

else:

# 切换剪切

❸ self.scene.showCircle = not self.scene.showCircle

`onKeyboard()` 回调函数会在每次发生键盘事件时被调用。该函数的参数包含了有用的信息，比如发生了什么类型的事件（例如按键按下与按键释放），以及哪个键被按下。代码 `glfw.GLFW_PRESS` 表示只查找按键按下（`PRESS`）事件 ❶。如果按下的是 ESC 键，你设置一个退出标志 ❷。如果按下的是其他键，你会切换 `showCircle` 布尔值 ❸。这个变量将被用在片段着色器中，用来决定是否保留或丢弃圆形区域外的片段。

#### 定义主循环

`RenderWindow` 类还通过其 `run()` 方法定义了程序的主循环。（GLFW 并没有提供默认的程序循环。）`run()` 方法会在预定的时间间隔内更新 OpenGL 窗口。调用渲染方法绘制场景后，它还会轮询系统，检查是否有待处理的窗口或键盘事件。让我们看一下该方法的定义：

def run(self):

# 初始化计时器

❶ glfw.glfwSetTime(0)

t = 0.0

❷ while not glfw.glfwWindowShouldClose(self.win) and not self.exitNow:

# 每 x 秒更新一次

❸ currT = glfw.glfwGetTime()

if currT - t > 0.1:

# 更新时间

t = currT

# 清除

❹ glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)

# 设置视口

❺ self.width, self.height = glfw.glfwGetFramebufferSize(self.win)

❻ self.aspect = self.width/float(self.height)

❼ glViewport(0, 0, self.width, self.height)

在主循环中，`glfw.glfwSetTime()`将 GLFW 计时器重置为 0 ❶。你将使用这个计时器定期重新绘制图形。你启动了一个`while`循环 ❷，只有当窗口关闭或`exitNow`设置为`True`时，循环才会退出。循环退出后，调用`glfw.glfwTerminate()`来干净地关闭 GLFW。

在循环内部，`glfw.glfwGetTime()`获取当前计时器值 ❸，你用它来计算自上次绘制以来的经过时间。通过在这里设置一个期望的间隔（在这种情况下为 0.1 秒或 100 毫秒），你可以调整渲染的帧率。接着，`glClear()`清除深度和颜色缓冲区，并用设置的背景色替换它们，以准备下一帧 ❹。

你使用`glfwGetFramebufferSize()`函数查询并设置窗口的宽度和高度 ❺。这样做是为了应对用户可能改变了窗口的大小。请注意，在某些系统中（如配备视网膜显示屏的 MacBook），窗口大小和帧缓冲区大小可能不同，因此为了安全起见，始终查询后者。接下来，你计算窗口的宽高比 ❻，稍后将用它来设置投影矩阵。然后你使用获取到的新帧缓冲区尺寸 ❼ 清除视口。

现在让我们看一下`run()`方法的剩余部分：

# 构建投影矩阵

❶ pMatrix = glutils.perspective(45.0, self.aspect, 0.1, 100.0)

❷ mvMatrix = glutils.lookAt([0.0, 0.0, -2.0], [0.0, 0.0, 0.0],

[0.0, 1.0, 0.0])

# 渲染

❸ self.scene.render(pMatrix, mvMatrix)

# 步骤

❹ self.scene.step()

❺ glfw.glfwSwapBuffers(self.win)

# 查询和处理事件

❻ glfw.glfwPollEvents()

# 结束

glfw.glfwTerminate()

仍然在`while`循环中，你使用*glutils.py*中定义的`perspective()`方法 ❶来计算投影矩阵。投影矩阵是将 3D 场景映射到 2D 屏幕的变换。这里你请求了 45 度的视野角度和近远平面距离为 0.1/100.0。接着，你使用`lookAt()`方法 ❷设置模型视图矩阵，该方法同样定义在*glutils.py*中。默认的 OpenGL 视图将你的眼睛位置设置在原点，沿负 z 轴方向观看。`lookAt()`方法创建的模型视图矩阵会转换顶点，以使视图与调用中指定的眼睛位置和方向对齐。你将眼睛位置设置为(0, 0, −2)，朝向原点(0, 0, 0)，且“上”向量为(0, 1, 0)。接下来，你在`scene`对象上调用`render()`方法 ❸，传入这些矩阵，并调用`scene.step()`以便它更新时间步长所需的变量 ❹。（`Scene`类，我们将在下文讨论，它封装了多边形的设置和渲染。）`glfwSwapBuffers()`调用 ❺ 会交换前后缓冲区，从而显示更新后的 3D 图形，`glfwPollEvents()`调用 ❻ 会检查是否有任何 UI 事件并将控制权返回给`while`循环。

### Scene 类

现在让我们来看看`Scene`类，它负责初始化和绘制 3D 几何体。下面是类声明的开始部分：

class Scene:

""" OpenGL 3D 场景类"""

# 初始化

def __init__(self):

# 创建着色器

❶ self.program = glutils.loadShaders(strVS, strFS)

❷ glUseProgram(self.program)

在`Scene`类的构造函数中，首先编译并加载着色器。为此，你使用了*glutils.py*中定义的工具方法`loadShaders()` ❶，该方法为加载着色器代码、编译并将其链接到 OpenGL 程序对象的整个 OpenGL 调用过程提供了一个方便的封装。由于 OpenGL 是一个状态机，你需要使用`glUseProgram()`调用 ❷来设置代码使用特定的“程序对象”（因为一个项目可能包含多个程序）。

`__init__()`方法继续通过将 Python 代码中的变量与着色器中的变量连接起来：

self.pMatrixUniform = glGetUniformLocation(self.program, b'uPMatrix')

self.mvMatrixUniform = glGetUniformLocation(self.program, b'uMVMatrix')

# 纹理

self.tex2D = glGetUniformLocation(self.program, b'tex2D')

这段代码使用`glGetUniformLocation()`方法获取在顶点着色器和片段着色器中定义的变量`uPMatrix`、`uMVMatrix`和`tex2D`的位置。然后可以使用这些位置来设置着色器变量的值。

#### 定义 3D 几何体

`Scene`类的`__init__()`方法的下一部分定义了场景的 3D 几何体。你首先定义了多边形的几何形状，它将呈现为一个正方形：

# 定义三角形带顶点

❶ vertexData = numpy.array(

[-0.5, -0.5, 0.0,

0.5, -0.5, 0.0,

-0.5, 0.5, 0.0,

0.5, 0.5, 0.0], numpy.float32)

# 设置顶点数组对象 (VAO)

❷ self.vao = glGenVertexArrays(1)

glBindVertexArray(self.vao)

# 顶点

❸ self.vertexBuffer = glGenBuffers(1)

glBindBuffer(GL_ARRAY_BUFFER, self.vertexBuffer)

# 设置缓冲区数据

❹ glBufferData(GL_ARRAY_BUFFER, 4*len(vertexData), vertexData,

GL_STATIC_DRAW)

# 启用顶点数组

❺ glEnableVertexAttribArray(0)

# 设置缓冲区数据指针

❻ glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, None)

# 解除绑定 VAO

❼ glBindVertexArray(0)

首先，你定义了用于绘制正方形的三角形带顶点数组 ❶。可以想象一个边长为 1.0 的正方形，中心位于原点。这个正方形的左下顶点坐标为(−0.5, −0.5, 0.0)；接下来的顶点（右下角的顶点）坐标为(0.5, −0.5, 0.0)；依此类推。这四个坐标的顺序符合`GL_TRIANGLE_STRIP`。本质上，你通过定义两个共享斜边的直角三角形来创建正方形。

接下来，你创建一个 VAO ❷。一旦你绑定到这个 VAO，所有接下来的调用都会绑定到它。然后你创建一个 VBO 来管理顶点数据的渲染 ❸。一旦缓冲区被绑定，你就会将你定义的顶点数据设置到缓冲区中 ❹。

现在你需要启用着色器访问这些数据。为此，你需要调用`glEnableVertexAttribArray()` ❺。你使用索引 0，因为这是你在顶点着色器中为顶点数据变量设置的位置。调用`glVertexAttribPointer()`设置顶点属性数组的位置和数据格式 ❻。属性的索引是 0，组件数是 3（你使用的是 3D 顶点），顶点的数据类型是`GL_FLOAT`。然后，你解除绑定 VAO ❼，以免其他相关调用干扰它。在 OpenGL 中，完成操作后最好重置状态。OpenGL 是一个状态机，如果你把状态留在混乱中，它就会保持不变。

以下代码将加载一张星星的图片作为 OpenGL 纹理：

# 纹理

self.texId = glutils.loadTexture('star.png')

返回的纹理 ID 将在渲染时使用。

#### 旋转正方形

接下来，你需要更新`Scene`对象中的变量，以便让正方形在屏幕上旋转。使用该类的`step()`方法：

# step

def step(self):

# 递增角度

❶ self.t = (self.t + 1) % 360

在❶处，你递增角度变量`t`并使用取余运算符（`%`）将此值保持在[0, 360]范围内。这个变量将用于更新顶点着色器中的旋转角度。

#### 渲染场景

现在，让我们看看`Scene`对象的主要渲染代码：

def render(self, pMatrix, mvMatrix)

# 使用着色器

❶ glUseProgram(self.program)

# 设置投影矩阵

❷ glUniformMatrix4fv(self.pMatrixUniform, 1, GL_FALSE, pMatrix)

# 设置模型视图矩阵

glUniformMatrix4fv(self.mvMatrixUniform, 1, GL_FALSE, mvMatrix)

# 设置着色器角度（弧度）

❸ glUniform1f(glGetUniformLocation(self.program, 'uTheta'),

math.radians(self.t))

# 显示圆形？

❹ glUniform1i(glGetUniformLocation(self.program, b'showCircle'),

self.showCircle)

# 启用纹理

❺ glActiveTexture(GL_TEXTURE0)

❻ glBindTexture(GL_TEXTURE_2D, self.texId)

❼ glUniform1i(self.tex2D, 0)

# 绑定 VAO

❽ glBindVertexArray(self.vao)

# 绘制

❾ glDrawArrays(GL_TRIANGLE_STRIP, 0, 4)

# 解绑 VAO

❿ glBindVertexArray(0)

首先，你设置渲染以使用着色器程序 ❶。从 ❷ 开始，你使用 `glUniformMatrix4fv()` 方法将计算得到的投影矩阵和模型视图矩阵传递到着色器中。然后，你使用 `glUniform1f()` 方法设置着色器程序中的 `uTheta` 变量 ❸。像之前一样，你使用 `glGetUniformLocation()` 获取着色器中 `uTheta` 角度变量的位置，并使用 Python 的 `math.radians()` 方法将角度从度数转换为弧度。接下来，你使用 `glUniform1i()` 设置片段着色器中 `showCircle` 变量的当前值 ❹。OpenGL 有多个纹理单元的概念，`glActiveTexture()` ❺ 激活纹理单元 0（默认值）。你将之前从 *star.png* 图像生成的纹理 ID 绑定到纹理单元 0 上以激活它进行渲染 ❻。片段着色器中的 `sampler2D` 变量被设置为纹理单元 0 ❼。

你继续绑定之前创建的 VAO ❽。现在你可以看到使用 VAO 的好处：你不需要在实际绘制之前重复一堆与顶点缓冲区相关的调用。然后，你调用 `glDrawArrays()` 来渲染绑定的顶点缓冲区 ❾。基元类型是三角带，并且有四个顶点需要渲染。最后，你在 ❿ 处解除绑定 VAO，这始终是一个良好的编码实践。

#### 定义 GLSL 着色器

现在让我们来看一下项目中最令人兴奋的部分——GLSL 着色器。首先是顶点着色器，它计算顶点的位置和纹理坐标：

# version 410 core

❶ layout(location = 0) in vec3 aVert;

❷ uniform mat4 uMVMatrix;

uniform mat4 uPMatrix;

uniform float uTheta;

❸ out vec2 vTexCoord;

void main() {

// 旋转变换

❹ mat4 rot = mat4(

vec4(1.0, 0.0, 0.0, 0.0),

vec4(0.0, cos(uTheta), -sin(uTheta), 0.0),

vec4(0.0, sin(uTheta),  cos(uTheta), 0.0),

vec4(0.0, 0.0,          0.0,          1.0)

);

// 变换顶点

❺ gl_Position = uPMatrix * uMVMatrix * rot * vec4(aVert, 1.0);

// 设置纹理坐标

❻ vTexCoord = aVert.xy + vec2(0.5, 0.5);

}

你使用 `layout` 关键字 ❶ 显式设置顶点属性 `aVert` 的位置——在这个例子中是 0。这个属性让顶点着色器可以访问你为多边形定义的顶点。从 ❷ 开始，你声明了三个 `uniform` 变量，用于存储投影矩阵、模型视图矩阵和旋转角度。它们将从 Python 代码中设置。你还将一个 2D 向量 `vTexCoord` 作为此着色器的输出 ❸。这个向量将作为片段着色器的输入。

在着色器的 `main()` 方法中，你设置了一个旋转矩阵❹，该矩阵按照给定的角度 `uTheta` 绕 x 轴旋转。你使用投影矩阵、模型视图矩阵和旋转矩阵的组合来计算 `gl_Position` ❺。这给出了着色器输出顶点的位置。接下来，你设置了一个 2D 向量作为纹理坐标❻。你可能记得，你定义了一个中心在原点、边长为 1.0 的方形三角带。由于纹理坐标的范围是 [0, 1]，你可以通过将 (0.5, 0.5) 加到 x 和 y 的值来生成这些坐标。这也展示了着色器在计算中的强大和灵活性。纹理坐标和其他变量并非不可修改；你可以将它们设置为几乎任何值。

现在让我们来看一下片段着色器，它计算了我们 OpenGL 程序的输出像素：

# version 410 core

❶ 输入变量 vec2 vTexCoord;

❷ uniform sampler2D tex2D;

❸ uniform bool showCircle;

❹ 输出变量 vec4 fragColor;

void main() {

如果 (showCircle) {

// 丢弃圆形外部的片段

❺ 如果 (distance(vTexCoord, vec2(0.5, 0.5)) > 0.5) {

discard;

}

else {

❻ fragColor = texture(tex2D, vTexCoord);

}

}

else {

❼ fragColor = texture(tex2D, vTexCoord);

}

}

你从定义片段着色器的输入开始——在这种情况下，就是你在顶点着色器中设置为输出的纹理坐标❶。回想一下，片段着色器是逐像素操作的，因此这些变量的值是当前像素的值，在多边形内进行插值。你声明了一个 `sampler2D` 变量❷，它与一个特定的纹理单元关联，用于查找纹理值，并且声明了一个布尔型的统一标志 `showCircle` ❸，该标志由 Python 代码设置。你还声明了 `fragColor` 作为片段着色器的输出❹。默认情况下，这个值会传送到屏幕（经过最终的帧缓冲操作，如深度测试和混合）。

在 `main()` 方法内，如果 `showCircle` 标志未设置❼，你使用 GLSL 的 `texture()` 方法，通过纹理坐标和采样器查找纹理的颜色值。实际上，你就是在用星形图像对三角带进行纹理映射。然而，如果 `showCircle` 标志为 `true` ❺，你使用 GLSL 内建的 `distance()` 方法来检查当前像素到多边形中心的距离。它使用（插值后的）纹理坐标进行此操作，这些坐标是由顶点着色器传入的。如果距离大于某个阈值（在本例中为 0.5），你调用 GLSL 的 `discard()` 方法，丢弃当前像素。如果距离小于阈值，你从纹理中设置适当的颜色❻。基本上，这个操作会忽略那些位于半径为 0.5 的圆形外部的像素，并在设置了 `showCircle` 时，将多边形剪切成一个圆形。

### 工具函数

我参考了为你定义的几个实用函数，存放在 *glutils.py* 文件中，帮助你更容易地使用 OpenGL。现在我们来看其中一个函数的示例。`loadTexture()` 函数将图像加载到 OpenGL 纹理中：

def loadTexture(filename):

"""从给定的图像文件加载 OpenGL 2D 纹理"""

❶ img = Image.open(filename)

❷ imgData = numpy.array(list(img.getdata()), np.int8)

❸ texture = glGenTextures(1)

❹ glBindTexture(GL_TEXTURE_2D, texture)

❺ glPixelStorei(GL_UNPACK_ALIGNMENT, 1)

❻ glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE)

glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE)

❼ glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR)

glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR)

❽ glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, img.size[0], img.size[1],

0, GL_RGBA, GL_UNSIGNED_BYTE, imgData)

return texture

`loadTexture()` 函数使用 Python 图像库（PIL）的 `Image` 模块来读取图像文件 ❶。然后，它将 `Image` 对象中的数据提取到一个 8 位的 `numpy` 数组 ❷ 中，并创建一个 OpenGL 的 `texture` 对象 ❸，这是在 OpenGL 中处理纹理的前提条件。接下来，你执行了我们已经熟悉的纹理绑定操作 ❹，使得所有后续的纹理相关设置都应用于此对象。你将数据的解包对齐方式设置为 1 ❺，这意味着硬件会将图像数据视为 1 字节或 8 位数据。从 ❻ 开始，你告诉 OpenGL 如何处理纹理的边缘。在这个例子中，你指示它将纹理颜色限制在几何体的边缘。（在指定纹理坐标时，约定使用字母 `S` 和 `T` 来表示坐标轴，而不是 `x` 和 `y`。）在 ❼ 和随后的行中，你指定了纹理在被拉伸或压缩映射到多边形时使用的插值类型。在这个例子中，指定了 *线性过滤*。最后，你将图像数据设置到绑定的纹理中 ❽。此时，图像数据已传输到图形内存，纹理已准备好使用。

## 运行 OpenGL 应用程序

这是项目的一个示例运行：

$ `python simpleglfw.py`

你可以在 图 9-1 中看到输出。记得尝试按键切换圆形的显示与隐藏。

## 总结

恭喜你完成了使用 Python 和 OpenGL 的第一个程序！通过这个项目，你学习了如何创建 3D 变换，使用 OpenGL 3D 图形管线，以及使用 GLSL 顶点和片段着色器来创建有趣的 3D 图形。你已经开始踏入 3D 图形编程的迷人世界。

## 实验！

这是一些修改此项目的想法：

1.  1\. 该项目中的顶点着色器将正方形绕 x 轴（1, 0, 0）旋转。你能让它绕 y 轴（0, 0, 1）旋转吗？你可以通过两种方式来实现：首先，修改着色器中的旋转矩阵；或者，第二，计算这个矩阵并将其作为一个*uniform*传入着色器。试试看！

1.  2\. 在该项目中，纹理坐标是在顶点着色器内生成并传递给片段着色器的。这是一个技巧，只有在选择了适当的三角带顶点值时才有效。像传递顶点一样，将纹理坐标作为一个独立的属性传入顶点着色器。现在，你能让星星纹理在三角带上*平铺*吗？你不希望只显示一个星星，而是要在正方形上显示一个 4×4 的星星网格。（提示：使用大于 1.0 的纹理坐标，并将 `glTexParameterf()` 中的 `GL_TEXTURE_WRAP_S/T` 参数设置为 `GL_REPEAT`。）

1.  3\. 仅通过改变片段着色器，你能让你的正方形看起来像图 9-6 吗？（提示：使用 GLSL 的 `sin()` 函数。）

![](img/nsp-venkitachalam503045-f09006.jpg)

图 9-6：使用片段着色器绘制同心圆

## 完整代码

这是完整的*simpleglfw.py*代码：

"""

simpleglfw.py

一个简单的 Python OpenGL 程序，使用 PyOpenGL + GLFW 获取一个

OpenGL 4.1 上下文。

作者：Mahesh Venkitachalam

"""

import OpenGL

from OpenGL.GL import *

import numpy, math, sys, os

import glutils

import glfw

strVS = """

# version 410 core

layout(location = 0) in vec3 aVert;

uniform mat4 uMVMatrix;

uniform mat4 uPMatrix;

uniform float uTheta;

out vec2 vTexCoord;

void main() {

// 旋转变换

mat4 rot =  mat4(

vec4(1.0,  0.0,          0.0,          0.0),

vec4(0.0,  cos(uTheta),  sin(uTheta), 0.0),

vec4(0.0, -sin(uTheta),  cos(uTheta), 0.0),

vec4(0.0,  0.0,          0.0,          1.0)

);

// 变换顶点

gl_Position = uPMatrix * uMVMatrix * rot * vec4(aVert, 1.0);

// 设置纹理坐标

vTexCoord = aVert.xy + vec2(0.5, 0.5);

}

"""

strFS = """

# version 410 core

in vec2 vTexCoord;

uniform sampler2D tex2D;

uniform bool showCircle;

out vec4 fragColor;

void main() {

if (showCircle) {

// 丢弃圆外的片段

if (distance(vTexCoord, vec2(0.5, 0.5)) > 0.5) {

discard;

}

else {

fragColor = texture(tex2D, vTexCoord);

}

}

else {

fragColor = texture(tex2D, vTexCoord);

}

}

"""

class Scene:

""" OpenGL 3D 场景类"""

# 初始化

def __init__(self):

# 创建着色器

self.program = glutils.loadShaders(strVS, strFS)

glUseProgram(self.program)

self.pMatrixUniform = glGetUniformLocation(self.program,

b'uPMatrix')

self.mvMatrixUniform = glGetUniformLocation(self.program,

b'uMVMatrix')

# 纹理

self.tex2D = glGetUniformLocation(self.program, b'tex2D')

# 定义三角带顶点

vertexData = numpy.array(

[-0.5, -0.5, 0.0,

0.5, -0.5, 0.0,

-0.5, 0.5, 0.0,

0.5, 0.5, 0.0], numpy.float32)

# 设置顶点数组对象 (VAO)

self.vao = glGenVertexArrays(1)

glBindVertexArray(self.vao)

# 顶点

self.vertexBuffer = glGenBuffers(1)

glBindBuffer(GL_ARRAY_BUFFER, self.vertexBuffer)

# 设置缓冲区数据

glBufferData(GL_ARRAY_BUFFER, 4*len(vertexData), vertexData,

GL_STATIC_DRAW)

# 启用顶点数组

glEnableVertexAttribArray(0)

# 设置缓冲区数据指针

glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 0, None)

# 解除绑定 VAO

glBindVertexArray(0)

# 时间

self.t = 0

# 纹理

self.texId = glutils.loadTexture('star.png')

# 显示圆圈?

self.showCircle = False

# 步骤

def step(self):

# 增加角度

self.t = (self.t + 1) % 360

# 渲染

def render(self, pMatrix, mvMatrix):

# 使用着色器

glUseProgram(self.program)

# 设置投影矩阵

glUniformMatrix4fv(self.pMatrixUniform, 1, GL_FALSE, pMatrix)

# 设置模型视图矩阵

glUniformMatrix4fv(self.mvMatrixUniform, 1, GL_FALSE, mvMatrix)

# 设置着色器角度（弧度）

glUniform1f(glGetUniformLocation(self.program, 'uTheta'),

math.radians(self.t))

# 显示圆圈?

glUniform1i(glGetUniformLocation(self.program, b'showCircle'),

self.showCircle)

# 启用纹理

glActiveTexture(GL_TEXTURE0)

glBindTexture(GL_TEXTURE_2D, self.texId)

glUniform1i(self.tex2D, 0)

# 绑定 VAO

glBindVertexArray(self.vao)

# 绘制

glDrawArrays(GL_TRIANGLE_STRIP, 0, 4)

# 解除绑定 VAO

glBindVertexArray(0)

class RenderWindow:

"""GLFW 渲染窗口类"""

def __init__(self):

# 保存当前工作目录

cwd = os.getcwd()

# 初始化 glfw - 这会改变当前工作目录

glfw.glfwInit()

# 恢复工作目录

os.chdir(cwd)

# 版本提示

glfw.glfwWindowHint(glfw.GLFW_CONTEXT_VERSION_MAJOR, 4)

glfw.glfwWindowHint(glfw.GLFW_CONTEXT_VERSION_MINOR, 1)

glfw.glfwWindowHint(glfw.GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE)

glfw.glfwWindowHint(glfw.GLFW_OPENGL_PROFILE,

glfw.GLFW_OPENGL_CORE_PROFILE)

# 创建窗口

self.width, self.height = 800, 600

self.aspect = self.width/float(self.height)

self.win = glfw.glfwCreateWindow(self.width, self.height,

b'simpleglfw')

# 使上下文当前

glfw.glfwMakeContextCurrent(self.win)

# 初始化 GL

glViewport(0, 0, self.width, self.height)

glEnable(GL_DEPTH_TEST)

glClearColor(0.5, 0.5, 0.5, 1.0)

# 设置窗口回调

glfw.glfwSetKeyCallback(self.win, self.onKeyboard)

# 创建 3D

self.scene = Scene()

# 退出标志

self.exitNow = False

def onKeyboard(self, win, key, scancode, action, mods):

# 打印 'keyboard: ', win, key, scancode, action, mods

if action == glfw.GLFW_PRESS:

# ESC 退出

if key == glfw.GLFW_KEY_ESCAPE:

self.exitNow = True

else:

# 切换裁剪

self.scene.showCircle = not self.scene.showCircle

def run(self):

# 初始化计时器

glfw.glfwSetTime(0)

t = 0.0

while not glfw.glfwWindowShouldClose(self.win) and not self.exitNow:

# 每隔 x 秒更新

currT = glfw.glfwGetTime()

if currT - t > 0.1:

# 更新时间

t = currT

# 清除

glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)

# 设置视口

self.width, self.height =

glfw.glfwGetFramebufferSize(self.win)

self.aspect = self.width/float(self.height)

glViewport(0, 0, self.width, self.height)

# 构建投影矩阵

pMatrix = glutils.perspective(45.0, self.aspect, 0.1, 100.0)

mvMatrix = glutils.lookAt([0.0, 0.0, -2.0], [0.0, 0.0, 0.0],

[0.0, 1.0, 0.0])

# 渲染

self.scene.render(pMatrix, mvMatrix)

# 步骤

self.scene.step()

glfw.glfwSwapBuffers(self.win)

# 获取并处理事件

glfw.glfwPollEvents()

# 结束

glfw.glfwTerminate()

def step(self):

# 步骤

self.scene.step()

# main() 函数

def main():

print("正在启动 simpleglfw. "

"按任意键切换裁剪。按 ESC 退出。"

rw = RenderWindow()

rw.run()

# 调用 main

if __name__ == '__main__':

main()
