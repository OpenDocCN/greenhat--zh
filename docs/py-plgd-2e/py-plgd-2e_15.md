# 11

# 体积渲染

![](img/nsp-venkitachalam503045-circle-image.jpg)

磁共振成像（MRI）和计算机断层扫描（CT）是创建*体积数据*的诊断过程，体积数据由一组显示通过 3D 体积的横截面的 2D 图像组成。*体积渲染*是一种计算机图形技术，用于从这种体积数据中构建 3D 图像。尽管体积渲染通常用于分析医学扫描，但它也可以用于创建学术领域（如地质学、考古学和分子生物学）的 3D 科学可视化。

通过 MRI 和 CT 扫描捕获的数据通常呈现为*N*[x]×*N*[y]×*N*[z]的 3D 网格形式。换句话说，有*N*[z]个 2D“切片”，每个切片是一个大小为*N*[x]×*N*[y]的图像。体积渲染算法用于以某种透明度显示收集的切片数据，且使用各种技术来突出显示渲染体积中感兴趣的部分。

在这个项目中，你将研究一种叫做*体积光线投射*的体积渲染方法，它充分利用图形处理单元（GPU）通过 OpenGL 着色语言（GLSL）着色器进行计算。你的代码将在每个屏幕像素上执行，并利用 GPU，GPU 旨在高效地进行并行计算。你将使用一个包含来自 3D 数据集切片的 2D 图像的文件夹，利用体积光线投射算法构建体积渲染图像。你还将实现一种方法，以显示数据在 x、y 和 z 方向上的 2D 切片，让用户可以通过箭头键滚动浏览切片。键盘命令将让用户在 3D 渲染和 2D 切片之间切换。

这是本项目涵盖的一些主题：

+   • 使用 GLSL 进行 GPU 计算

+   • 创建顶点和片段着色器

+   • 表示 3D 体积数据并使用体积光线投射算法

+   • 使用`numpy`数组进行 3D 变换矩阵

## 工作原理

渲染 3D 数据集有多种方法。在这个项目中，你将使用体积光线投射方法，这是一种基于*图像*的渲染技术，用于从 2D 切片逐像素生成最终图像。相比之下，典型的 3D 渲染方法是*基于对象*的：它们从 3D 对象表示开始，然后应用变换生成投影 2D 图像中的像素。

在本项目中使用的体积光线投射方法中，对于输出图像中的每个像素，一束光线被射入离散的 3D 体积数据集，该数据集通常表示为一个长方体。当光线穿过体积时，数据会在规则的间隔处进行采样，并将这些样本合成，或*合成*，以计算最终图像的颜色值或强度。（你可以将这个过程想象成将一堆透明片叠加在一起，然后对着强光看，看到所有纸片混合后的效果。）

尽管体积光线投射渲染实现通常使用一些技术，如应用梯度以改善最终渲染效果、过滤以隔离 3D 特征、以及使用空间优化技术以提高速度，但在这个项目中，你将只实现基本的光线投射算法，并通过 X 射线投射合成最终图像。（我的实现主要基于 Kruger 和 Westermann 在 2003 年发表的这方面的开创性论文。1）

### 数据格式

在本项目中，你将使用来自斯坦福体积数据档案的 3D 医学扫描数据。2 该档案提供了一些出色的 3D 医学数据集（包括 CT 和 MRI），每个数据集包含表示体积的每个 2D 横截面的 TIFF 图像。你将把这些图像文件夹读入 OpenGL 3D 纹理；这就像是将一组 2D 图像叠加在一起形成一个长方体，正如图 11-1 所示。

![](img/nsp-venkitachalam503045-f11001.jpg)

图 11-1：从 2D 切片构建 3D 体积数据

回想一下第九章中的内容，在 OpenGL 中，2D 纹理使用 2D 坐标（*s*，*t*）进行寻址。类似地，3D 纹理使用类似（*s*，*t*，*p*）的 3D 纹理坐标进行寻址。正如你将看到的，将体积数据存储为 3D 纹理，可以快速访问数据，并为你的光线投射方案提供所需的插值值。

### 光线生成

本项目的目标是生成 3D 体积数据的透视投影，如图 11-2 所示。该图展示了 OpenGL 视锥体，如第九章中所讨论的内容。具体来说，图中展示了一束来自眼睛的光线如何穿过近平面进入视锥体，通过包含体积数据的立方体体积，最后从远平面出口。

![](img/nsp-venkitachalam503045-f11002.jpg)

图 11-2：3D 体积数据的透视投影

要实现光线投射，你需要生成射入体积的光线。对于输出窗口中每个像素，如图 11-2 所示，你需要生成一个向量 *R*，这个向量进入你认为是单位立方体的体积（我将其称为*颜色立方体*），该立方体的坐标范围是（0，0，0）到（1，1，1）。你将这个立方体内的每个点着色，RGB 值等于该点在立方体内的 3D 坐标。原点的颜色为（0，0，0），即黑色；（1，0，0）角为红色；而立方体中与原点对角线相对的点则为（1，1，1），即白色。图 11-3 显示了这个立方体。

![](img/nsp-venkitachalam503045-f11003.jpg)

图 11-3：一个颜色立方体

注意：在 OpenGL 中，颜色可以表示为一组 8 位无符号值（*r*，*g*，*b*），其中 *r*，*g* 和 *b* 的取值范围为 [0, 255]。也可以表示为一组三个 32 位浮点值（*r*，*g*，*b*），其中 *r*，*g* 和 *b* 的取值范围为 [0.0, 1.0]。这两种表示是等效的。例如，前者的红色（255, 0, 0）与后者的（1.0, 0.0, 0.0）是相同的。

要绘制立方体，首先使用 OpenGL 原语 `GL_TRIANGLES` 绘制它的六个面。然后为每个顶点着色，并利用 OpenGL 在光栅化多边形时提供的插值来处理每个顶点之间的颜色。例如，图 11-4(a) 显示了立方体的三个前面。立方体的背面则在图 11-4(b)中绘制，通过设置 OpenGL 剔除前面。

![](img/nsp-venkitachalam503045-f11004.jpg)

图 11-4：用于计算光线的颜色立方体

如果你通过从图 11-4(a)中减去图 11-4(b)中的颜色，即将 (*r*, *g*, *b*)[front] 从 (*r*, *g*, *b*)[back] 中减去，你实际上是在计算一组从立方体前面到背面的向量，因为该立方体上每个颜色的 (*r*, *g*, *b*) 值实际上等于该颜色位置的 3D 坐标。图 11-4(c) 显示了结果。（为便于说明，负值已被转为正值，因为负数不能直接显示为颜色。）读取像素的颜色值 (*r*, *g*, *b*)，如图 11-4(c)所示，将得到光线通过该点进入体积的 (*r*[x], *r*[y], *r*[z]) 坐标。

一旦你获得了投射光线，就可以将它们渲染成图像或 2D 纹理，供以后与 OpenGL 的帧缓冲对象（FBO）功能一起使用。在生成该纹理之后，你可以在实现光线投射算法的着色器中访问它。

#### GPU 中的光线投射

为实现光线投射算法，你首先将色彩立方体的背面绘制到 FBO（帧缓冲对象）中。接着，前面将绘制到屏幕上。大部分的光线投射算法发生在第二次渲染的片段着色器中，这个着色器会针对输出的每个像素运行。光线通过从色彩立方体背面的颜色中减去传入片段的前面颜色来计算，这些颜色从纹理中读取。计算得到的光线随后被用来累积和计算最终的像素值，使用的是在着色器中可用的 3D 体积纹理数据。

#### 显示 2D 切片

除了 3D 渲染，你还会通过从 3D 数据中提取与 x、y 或 z 轴垂直的 2D 截面，并将其作为纹理应用于一个四边形，来显示数据的 2D 切片。因为你将体积存储为 3D 纹理，所以可以通过指定纹理坐标（*s*、*t*、*p*）轻松获得所需的数据。OpenGL 的内置纹理插值可以让你在 3D 纹理的任意位置获取纹理值。

### OpenGL 窗口

如同你在其他 OpenGL 项目中一样，本项目使用 GLFW 库来显示 OpenGL 窗口。你将使用处理程序来绘制、调整窗口大小以及处理键盘事件。你将使用键盘事件在体积渲染和切片渲染之间切换，并通过旋转和切割来操作 3D 数据。

## 要求

你将使用 `PyOpenGL`，这是一个流行的 OpenGL Python 绑定库，用于渲染。你还将使用 Python Imaging Library (PIL) 来加载体积数据集中的 2D 图像，并使用 `numpy` 数组来表示 3D 坐标和变换矩阵。

## 代码

首先，你将从图像文件中读取体积数据，并生成一个 3D 纹理。接下来，你将使用一个色彩立方体技术来生成从眼睛出发、指向体积的光线，这是实现体积光线投射算法的关键概念。你将研究如何定义立方体的几何体，以及如何绘制该立方体的前后面。然后，你将探索体积光线投射算法及其相关的顶点和片段着色器。最后，你将学习如何实现体积数据的 2D 切片。

本项目包含七个 Python 文件：

glutils.py 包含 OpenGL 着色器、变换等的实用方法

makedata.py 包含为测试生成体积数据的实用方法

raycast.py 实现了用于光线投射的 `RayCastRender` 类

raycube.py 实现了用于 `RayCastRender` 的 `RayCube` 类

slicerender.py 实现了用于体积数据 2D 切片的 `SliceRender` 类

volreader.py 包含读取体积数据并加载到 OpenGL 3D 纹理中的实用方法

volrender.py 包含创建 GLFW 窗口和渲染器的主要方法

我们将在本章中覆盖除了两个文件之外的所有文件。*makedata.py* 文件与本章的其他项目文件一起存放在 [`github.com/mkvenkit/pp2e/tree/main/volrender`](https://github.com/mkvenkit/pp2e/tree/main/volrender) 中。*glutils.py* 文件可以从 [`github.com/mkvenkit/pp2e/tree/main/common`](https://github.com/mkvenkit/pp2e/tree/main/common) 下载。

### 生成 3D 纹理

第一步是从包含图像的文件夹中读取体积数据，如下所示的代码所示。要查看完整的 *volreader.py* 代码，请跳到 “完整的 3D 纹理代码” 在 第 241 页。你也可以在 [`github.com/mkvenkit/pp2e/tree/main/volrender`](https://github.com/mkvenkit/pp2e/tree/main/volrender) 找到 *volreader.py* 文件。请注意，这个文件中的 `loadTexture()` 函数用于打开图像文件，读取内容，并创建一个 OpenGL 纹理对象，随后用于渲染。

def loadVolume(dirName):

"""从目录读取体积数据作为 3D 纹理"""

# 列出目录中的图像

❶ files = sorted(os.listdir(dirName))

print('从以下路径加载图像: %s' % dirName)

imgDataList = []

count = 0

width, height = 0, 0

for file in files:

❷ file_path = os.path.abspath(os.path.join(dirName, file))

try:

# 读取图像

❸ img = Image.open(file_path)

imgData = np.array(img.getdata(), np.uint8)

# 检查所有图像是否大小相同

❹ if count is 0:

width, height = img.size[0], img.size[1]

imgDataList.append(imgData)

else:

❺ if (width, height) == (img.size[0], img.size[1]):

imgDataList.append(imgData)

else:

print('不匹配')

raise RunTimeError("图像尺寸不匹配")

count += 1

# 打印 img.size

❻ except:

# 跳过

print('无效的图像: %s' % file_path)

# 将图像数据加载到单一数组中

depth = count

❼ data = np.concatenate(imgDataList)

print('体积数据维度: %d %d %d' % (width, height, depth))

`loadVolume()` 方法首先使用 `os` 模块中的 `listdir()` 方法 ❶ 列出给定目录中的文件。然后，你会遍历这些图像文件，一次加载一个。为此，你使用 `os.path.abspath()` 和 `os.path.join()` ❷ 将当前文件名追加到目录中，从而避免处理相对文件路径和操作系统特定的路径约定。（你经常在遍历文件和目录的 Python 代码中看到这种有用的习惯用法。）接下来，你使用 PIL 中的 `Image` 类将当前图像加载为 8 位的 `numpy` 数组 ❸。如果指定的文件不是图像，或者图像加载失败，会抛出一个异常，你通过打印错误来捕捉这个异常 ❻。

因为你将这些图像切片加载到 3D 纹理中，所以需要确保它们具有相同的尺寸（宽度 × 高度），这在❹和❺处进行确认。你存储了第一张图像的尺寸，并将其与新加载的图像进行比较。一旦所有图像被加载到各自的数组中，你使用`numpy`的`concatenate()`方法❼将这些数组合并成一个包含 3D 数据的最终数组。

`loadVolume()` 函数接着将 3D 图像数据数组加载到 OpenGL 纹理中：

# 将数据加载到 3D 纹理

❶ texture = glGenTextures(1)

glPixelStorei(GL_UNPACK_ALIGNMENT, 1)

glBindTexture(GL_TEXTURE_3D, texture)

glTexParameterf(GL_TEXTURE_3D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE)

glTexParameterf(GL_TEXTURE_3D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE)

glTexParameterf(GL_TEXTURE_3D, GL_TEXTURE_WRAP_R, GL_CLAMP_TO_EDGE)

glTexParameterf(GL_TEXTURE_3D, GL_TEXTURE_MAG_FILTER, GL_LINEAR)

glTexParameterf(GL_TEXTURE_3D, GL_TEXTURE_MIN_FILTER, GL_LINEAR)

❷ glTexImage3D(GL_TEXTURE_3D, 0, GL_RED,

width, height, depth, 0,

GL_RED, GL_UNSIGNED_BYTE, data)

# 返回纹理

❸ return (texture, width, height, depth)

在这里，你创建了一个 OpenGL 纹理❶并设置了过滤和解包的参数。然后将 3D 数据数组加载到 OpenGL 纹理中❷。此处使用的格式是`GL_RED`，数据格式是`GL_UNSIGNED_BYTE`，因为每个像素只有一个 8 位值。最后，你返回 OpenGL 纹理 ID 和 3D 纹理的尺寸❸。

### 生成光线

生成光线的代码封装在一个名为`RayCube`的类中。这个类负责绘制颜色立方体，并具有将立方体背面绘制到 FBO 或纹理以及将立方体前面绘制到屏幕上的方法。要查看完整的*raycube.py*代码，请跳到 “完整的光线生成代码” 页面 242 处。你也可以在 [`github.com/mkvenkit/pp2e/tree/main/volrender`](https://github.com/mkvenkit/pp2e/tree/main/volrender) 找到*raycube.py*文件。

首先，让我们定义这个类使用的着色器。着色器将作为`RayCube`类构造函数的一部分进行编译：

❶ strVS = """

# version 410 core

layout(location = 1) in vec3 cubePos;

layout(location = 2) in vec3 cubeCol;

uniform mat4 uMVMatrix;

uniform mat4 uPMatrix;

out vec4 vColor;

void main()

{

// 设置背面颜色

vColor = vec4(cubeCol.rgb, 1.0);

// 转换后的位置

vec4 newPos = vec4(cubePos.xyz, 1.0);

// 设置位置

gl_Position = uPMatrix * uMVMatrix * newPos;

}

"""

❷ strFS = """

# version 410 core

in vec4 vColor;

out vec4 fragColor;

void main()

{

fragColor = vColor;

}

"""

你为`RayCube`类定义了顶点着色器❶。这个着色器有两个输入属性，`cubePos`和`cubeCol`，它们分别用于访问顶点的位置和颜色值。模型视图矩阵和投影矩阵通过统一变量`uMVMatrix`和`uPMatrix`传递。`vColor`变量被声明为输出，因为它需要传递到片段着色器，在那里它会被插值。片段着色器❷将片段颜色设置为在顶点着色器中设置的（插值后的）`vColor`值。

#### 定义颜色立方体几何形状

现在让我们看一下在`RayCube`类的构造函数中定义的颜色立方体的几何形状：

class RayCube:

def __init__(self, width, height):

--`snip`--

# 立方体顶点

❶ vertices = numpy.array([

0.0, 0.0, 0.0,

1.0, 0.0, 0.0,

1.0, 1.0, 0.0,

0.0, 1.0, 0.0,

0.0, 0.0, 1.0,

1.0, 0.0, 1.0,

1.0, 1.0, 1.0,

0.0, 1.0, 1.0

], numpy.float32)

# 立方体颜色

❷ colors = numpy.array([

0.0, 0.0, 0.0,

1.0, 0.0, 0.0,

1.0, 1.0, 0.0,

0.0, 1.0, 0.0,

0.0, 0.0, 1.0,

1.0, 0.0, 1.0,

1.0, 1.0, 1.0,

0.0, 1.0, 1.0

], numpy.float32)

# 各个三角形

❸ indices = numpy.array([

4, 5, 7,

7, 5, 6,

5, 1, 6,

6, 1, 2,

1, 0, 2,

2, 0, 3,

0, 4, 3,

3, 4, 7,

6, 2, 7,

7, 2, 3,

4, 0, 5,

5, 0, 1

], numpy.int16)

你将立方体的几何形状❶和颜色❷定义为`numpy`数组。注意，这两个定义中的值是相同的。正如我们之前讨论的，颜色立方体中每个像素的颜色对应于该像素的 3D 坐标。颜色立方体有六个面，每个面可以绘制为两个三角形，总共有 6×6，即 36 个顶点。但与其指定所有 36 个顶点，不如只指定立方体的八个角❶，然后使用`indices`数组❸定义由这些角形成的三角形，如图 11-5 所示。例如，前两组三个索引（4, 5, 7）和（7, 5, 6）定义了立方体顶面上的三角形。

![](img/nsp-venkitachalam503045-f11005.jpg)

图 11-5：通过索引，一个立方体可以表示为一组三角形，每个面由两个三角形组成。

接下来，仍然在`RayCube`类的构造函数中，你需要将顶点信息放入缓冲区：

# 设置顶点数组对象（VAO）

self.vao = glGenVertexArrays(1)

glBindVertexArray(self.vao)

# 顶点缓冲区

self.vertexBuffer = glGenBuffers(1)

glBindBuffer(GL_ARRAY_BUFFER, self.vertexBuffer)

glBufferData(GL_ARRAY_BUFFER, 4*len(vertices), vertices, GL_STATIC_DRAW)

# 顶点缓冲区 – 立方体顶点颜色

self.colorBuffer = glGenBuffers(1)

glBindBuffer(GL_ARRAY_BUFFER, self.colorBuffer)

glBufferData(GL_ARRAY_BUFFER, 4*len(colors), colors, GL_STATIC_DRAW)

# 索引缓冲区

self.indexBuffer = glGenBuffers(1)

glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, self.indexBuffer); ❶

glBufferData(GL_ELEMENT_ARRAY_BUFFER, 2*len(indices), indices,

GL_STATIC_DRAW)

和之前的项目一样，你创建并绑定一个顶点数组对象（VAO），然后定义它管理的缓冲区。这里的一个区别是，`indices` 数组被指定为 `GL_ELEMENT_ARRAY_BUFFER` ❶，这意味着其缓冲区中的元素将用于索引和访问颜色和顶点缓冲中的数据。

#### 创建帧缓冲对象

现在让我们跳到创建帧缓冲对象的 `RayCube` 类方法，在这里你将进行渲染操作：

def initFBO(self):

# 创建帧缓冲对象

self.fboHandle = glGenFramebuffers(1)

# 创建纹理

self.texHandle = glGenTextures(1)

# 创建深度缓冲区

self.depthHandle = glGenRenderbuffers(1)

# 绑定

glBindFramebuffer(GL_FRAMEBUFFER, self.fboHandle)

glActiveTexture(GL_TEXTURE0)

glBindTexture(GL_TEXTURE_2D, self.texHandle)

# 设置参数以在不同大小下绘制图像

❶ glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR)

glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR)

glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE)

glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE)

# 设置纹理

glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, self.width, self.height,

0, GL_RGBA, GL_UNSIGNED_BYTE, None)

# 将纹理绑定到 FBO

❷ glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0,

GL_TEXTURE_2D, self.texHandle, 0)

# 绑定

❸ glBindRenderbuffer(GL_RENDERBUFFER, self.depthHandle)

glRenderbufferStorage(GL_RENDERBUFFER, GL_DEPTH_COMPONENT24,

self.width, self.height)

# 将深度缓冲绑定到 FBO

glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_ATTACHMENT,

GL_RENDERBUFFER, self.depthHandle)

# 检查状态

❹ status = glCheckFramebufferStatus(GL_FRAMEBUFFER)

if status == GL_FRAMEBUFFER_COMPLETE:

pass

# 打印 "fbo %d 完成" % self.fboHandle

elif status == GL_FRAMEBUFFER_UNSUPPORTED:

print "fbo %d 不支持" % self.fboHandle

else:

print "fbo %d 错误" % self.fboHandle

在这里，你创建了一个帧缓冲对象，一个 2D 纹理和一个渲染缓冲对象；然后你设置了纹理参数 ❶。纹理被绑定到帧缓冲 ❷，在 ❸ 及之后的代码行中，渲染缓冲设置了一个 24 位深度缓冲并附加到帧缓冲上。接下来，你检查帧缓冲的状态 ❹，如果出现问题，则打印状态消息。现在，只要帧缓冲和渲染缓冲被正确绑定，所有渲染内容都会输出到纹理中。

#### 渲染立方体的背面

下面是渲染颜色立方体背面的代码：

def renderBackFace(self, pMatrix, mvMatrix):

"""将射线立方体的背面渲染到纹理中并返回"""

# 渲染到 FBO

❶ glBindFramebuffer(GL_FRAMEBUFFER, self.fboHandle)

# 设置活动纹理

glActiveTexture(GL_TEXTURE0)

# 绑定到 FBO 纹理

glBindTexture(GL_TEXTURE_2D, self.texHandle)

# 渲染带有面剔除启用的立方体

❷ self.renderCube(pMatrix, mvMatrix, self.program, True)

# 解绑纹理

❸ glBindTexture(GL_TEXTURE_2D, 0)

glBindFramebuffer(GL_FRAMEBUFFER, 0)

glBindRenderbuffer(GL_RENDERBUFFER, 0)

# 返回纹理 ID

❹ 返回 self.texHandle

首先，你绑定 FBO ❶，设置活动纹理单元，并绑定到纹理句柄，这样就可以渲染到 FBO。然后你调用 `RayCube` 类的 `renderCube()` 方法 ❷，我们稍后会详细讨论。它有一个面剔除标志作为参数，允许你使用相同的代码绘制立方体的前面或背面。你将标志设置为 `True` 以使背面出现在 FBO 纹理中。

接下来，你进行必要的调用以解绑 FBO，以确保其他渲染代码不受影响 ❸。最后，你返回 FBO 纹理 ID ❹，以便在算法的下一阶段使用。

#### 渲染立方体的前面

以下代码在射线投射算法的第二次渲染过程中，用于绘制彩色立方体的前面：

def renderFrontFace(self, pMatrix, mvMatrix, program):

"""渲染射线立方体的前面"""

# 无面剔除

self.renderCube(pMatrix, mvMatrix, program, False)

这个方法简单地调用 `renderCube()`，并将面剔除标志设置为 `False`，这样前面就会显示出来。

#### 渲染整个立方体

现在让我们看一下 `renderCube()` 方法，它绘制了之前讨论过的彩色立方体：

def renderCube(self, pMatrix, mvMatrix, program, cullFace):

"""renderCube 在启用标志时使用面剔除"""

glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)

# 设置着色器程序

glUseProgram(program)

# 设置投影矩阵

glUniformMatrix4fv(glGetUniformLocation(program, b'uPMatrix'),

1, GL_FALSE, pMatrix)

# 设置模型视图矩阵

glUniformMatrix4fv(glGetUniformLocation(program, b'uMVMatrix'),

1, GL_FALSE, mvMatrix)

# 启用面剔除

glDisable(GL_CULL_FACE)

❶ 如果启用面剔除：

glFrontFace(GL_CCW)

glCullFace(GL_FRONT)

glEnable(GL_CULL_FACE)

# 绑定 VAO

glBindVertexArray(self.vao)

# 动画切片

❷ glDrawElements(GL_TRIANGLES, self.nIndices, GL_UNSIGNED_SHORT, None)

# 解绑 VAO

glBindVertexArray(0)

# 重置面剔除

如果启用面剔除：

# 禁用面剔除

glDisable(GL_CULL_FACE)

你清除颜色和深度缓冲区，选择着色器程序，并设置变换矩阵。然后你设置一个标志来控制面剔除 ❶，这决定了是绘制立方体的前面还是背面。注意，因为你使用的是索引数组来渲染立方体，而不是顶点数组，所以你使用 `glDrawElements()` ❷。

#### 调整窗口大小

因为 FBO 是为特定的窗口大小创建的，所以当窗口大小发生变化时，你需要重新创建它。为此，你需要为 `RayCube` 类创建一个调整大小处理器，如下所示：

def reshape(self, width, height):

self.width = width

self.height = height

self.aspect = width/float(height)

# 重新创建 FBO

self.clearFBO()

self.initFBO()

`reshape()` 方法在 OpenGL 窗口大小调整时调用。它检查新的窗口尺寸，然后清除并重新创建 FBO。

### 实现光线投射算法

接下来，你将在`RayCastRender`类中实现光线投射算法。该算法的核心发生在该类使用的片段着色器中，片段着色器也使用`RayCube`类帮助生成光线。要查看完整的*raycast.py*代码，请跳到“完整的体积光线投射代码”，该内容位于第 248 页。你还可以在[`github.com/mkvenkit/pp2e/tree/main/volrender`](https://github.com/mkvenkit/pp2e/tree/main/volrender)找到该文件。

在`RayCastRender`构造函数中，首先创建一个`RayCube`对象并加载着色器：

class RayCastRender:

def __init__(self, width, height, volume):

"""RayCastRender 构造函数"""

# 创建 RayCube 对象

❶ self.raycube = raycube.RayCube(width, height)

# 设置尺寸

self.width = width

self.height = height

self.aspect = width/float(height)

# 创建着色器

❷ self.program = glutils.loadShaders(strVS, strFS)

# 纹理

❸ self.texVolume, self.Nx, self.Ny, self.Nz = volume

# 初始化相机

❹ self.camera = Camera()

构造函数创建了一个`RayCube`类型的对象❶，用于生成光线。你加载了用于光线投射的着色器❷，然后设置了传入构造函数的 OpenGL 3D 纹理和尺寸❸，这些是作为元组`volume`传入的。接下来，你创建了一个`Camera`对象❹，你将用它来设置 OpenGL 的透视变换进行 3D 渲染。

注释：`Camera`类也在*raycast.py*中声明，它基本上与第十章中使用的类相同。你将在第 248 页的完整代码列表中看到它。

这里是`RayCastRender`的渲染方法：

def draw(self):

# 构建投影矩阵

❶ pMatrix = glutils.perspective(45.0, self.aspect, 0.1, 100.0)

# 模型视图矩阵

❷ mvMatrix = glutils.lookAt(self.camera.eye, self.camera.center,

self.camera.up)

# 渲染

# 生成光线立方体的背面纹理

❸ texture = self.raycube.renderBackFace(pMatrix, mvMatrix)

# 设置着色器程序

❹ glUseProgram(self.program)

# 设置窗口尺寸

glUniform2f(glGetUniformLocation(self.program, b"uWinDims"),

float(self.width), float(self.height))

# 绑定到纹理单元 0，表示立方体的背面

❺ glActiveTexture(GL_TEXTURE0)

glBindTexture(GL_TEXTURE_2D, texture)

glUniform1i(glGetUniformLocation(self.program, b"texBackFaces"), 0)

# 纹理单元 1：3D 体积纹理

❻ glActiveTexture(GL_TEXTURE1)

glBindTexture(GL_TEXTURE_3D, self.texVolume)

glUniform1i(glGetUniformLocation(self.program, b"texVolume"), 1)

# 绘制立方体的前面

❼ self.raycube.renderFrontFace(pMatrix, mvMatrix, self.program)

首先，你使用`glutils.perspective()`工具方法为渲染设置透视投影矩阵 ❶。然后，你将当前相机参数传入`glutils.lookAt()`方法 ❷。接下来，使用`RayCube`中的`renderBackFace()`方法绘制颜色立方体的背面到纹理中，完成渲染的第一遍 ❸。（这个方法还会返回生成的纹理 ID。）

你通过启用射线投射算法的着色器继续。然后，你为着色器程序设置纹理。在❸处返回的纹理被设置为纹理单元 0 ❺，从你读取的体积数据中创建的 3D 纹理被设置为纹理单元 1 ❻。最后，你使用`RayCube`中的`renderFrontFace()`方法渲染立方体的前面 ❼。当执行这段代码时，`RayCastRender`的着色器将在顶点和片段上进行操作。

#### 顶点着色器

现在你来看`RayCastRender`使用的着色器。首先看一下顶点着色器：

strVS = """

# version 410 core

❶ layout(location = 1) in vec3 cubePos;

layout(location = 2) in vec3 cubeCol;

❷ uniform mat4 uMVMatrix;

uniform mat4 uPMatrix;

❸ out vec4 vColor;

void main()

{

// 设置位置

❹ gl_Position = uPMatrix * uMVMatrix * vec4(cubePos.xyz, 1.0);

// 设置颜色

❺ vColor = vec4(cubeCol.rgb, 1.0);

}

"""

首先，你设置位置和颜色的输入变量 ❶。布局使用与`RayCube`顶点着色器中定义的相同索引，因为`RayCastRender`使用的是该类中定义的 VBO 来绘制几何图形，着色器中的位置必须匹配。然后，你定义输入变换矩阵 ❷，并设置一个颜色值作为着色器的输出 ❸。常规的变换会计算出内建的`gl_Position`输出 ❹，然后将输出设置为当前立方体顶点的颜色 ❺。后者将在顶点之间进行插值，从而在片段着色器中得到正确的颜色。

#### 片段着色器

片段着色器是整个过程的核心。它实现了射线投射算法的核心部分。

strFS = """

# version 410 core

in vec4 vColor;

uniform sampler2D texBackFaces;

uniform sampler3D texVolume;

uniform vec2 uWinDims;

out vec4 fragColor;

void main()

{

// 射线起点

❶ vec3 start = vColor.rgb;

// 计算片段的纹理坐标，

// 这是窗口坐标的一个分数

❷ vec2 texc = gl_FragCoord.xy/uWinDims.xy;

// 通过查找背面颜色获取射线的结束位置

❸ vec3 end = texture(texBackFaces, texc).rgb;

// 计算射线方向

❹ vec3 dir = end – start;

// 标准化射线方向

vec3 norm_dir = normalize(dir);

// 计算从前到后的长度

// 用于终止射线

float len = length(dir.xyz);

// 射线步长

float stepSize = 0.01;

// X 射线投影

vec4 dst = vec4(0.0);

// 沿射线步进

❺ for(float t = 0.0; t < len; t += stepSize) {

// 设置位置为射线的终点

❻ vec3 samplePos = start + t*norm_dir;

// 在当前位置获取纹理值

❼ float val = texture(texVolume, samplePos).r;

vec4 src = vec4(val);

// 设置不透明度

❽ src.a *= 0.1;

src.rgb *= src.a;

// 与先前的值混合

❾ dst = (1.0 - dst.a)*src + dst;

// 当 alpha 超过阈值时退出循环

❿ if(dst.a >= 0.95)

break;

}

// 设置片段颜色

fragColor = dst;

}

"""

片段着色器的输入是立方体的顶点颜色。片段着色器还可以访问由渲染颜色立方体生成的 2D 纹理、包含体积数据的 3D 纹理，以及 OpenGL 窗口的维度。

当片段着色器执行时，你传入了立方体的前面面，因此通过查找传入的颜色值❶，你可以得到射线进入此立方体的起点。（回想一下在“射线生成”中关于立方体中颜色与射线方向之间关系的讨论，参见第 217 页。）

你计算传入片段在屏幕上的纹理坐标❷。这里，通过将片段在窗口坐标中的位置除以窗口的维度，将位置映射到范围[0, 1]。射线的终点通过使用该纹理坐标查找立方体的背面颜色来获得❸。

接下来你计算射线方向❹，然后计算该射线的归一化方向和长度，这将在射线投射计算中很有用。然后，你使用射线的起点和方向循环遍历体积，直到射线达到终点❺。在这个循环中，你计算射线在数据体积中的当前位置❻，并查找该点的数值❼。接着，你在❽和❾处执行混合方程，从而得到 X 射线效果。你将`dst`值与当前的强度值（这个强度值使用 alpha 值进行衰减）相结合，过程沿着射线继续。alpha 值会不断增加，直到达到最大阈值 0.95❿，此时退出循环。最终结果是在每个像素处产生的通过体积的平均不透明度，从而形成“透视”或 X 射线效果。（尝试改变阈值和 alpha 衰减，以产生不同的效果。）

### 显示 2D 切片

除了显示体积数据的 3D 视图外，你还想在屏幕上显示数据在 x、y 和 z 方向上的 2D 切片。实现这个功能的代码封装在一个名为`SliceRender`的类中，该类创建 2D 体积切片。要查看完整的*slicerender.py*代码，请跳转到“完整的 2D 切片代码”，位于第 251 页。你还可以在[`github.com/mkvenkit/pp2e/tree/main/volrender`](https://github.com/mkvenkit/pp2e/tree/main/volrender)找到*slicerender.py*文件。

这是`SliceRender`类构造函数中的初始化代码，用于设置切片的几何形状：

class SliceRender:

def __init__(self, width, height, volume):

--`snip`--

# 设置顶点数组对象 (VAO)

self.vao = glGenVertexArrays(1)

glBindVertexArray(self.vao)

# 定义四边形顶点

❶ vertexData = numpy.array([0.0, 1.0, 0.0,

0.0, 0.0, 0.0,

1.0, 1.0, 0.0,

1.0, 0.0, 0.0], numpy.float32)

# 顶点缓冲区

self.vertexBuffer = glGenBuffers(1)

glBindBuffer(GL_ARRAY_BUFFER, self.vertexBuffer)

glBufferData(GL_ARRAY_BUFFER, 4*len(vertexData), vertexData,

GL_STATIC_DRAW)

# 启用数组

glEnableVertexAttribArray(self.vertIndex)

# 设置缓冲区

glBindBuffer(GL_ARRAY_BUFFER, self.vertexBuffer)

glVertexAttribPointer(self.vertIndex, 3, GL_FLOAT, GL_FALSE, 0, None)

# 解绑 VAO

glBindVertexArray(0)

这段代码设置了一个 VAO 来管理 VBO，类似于早期的示例。你定义了 XY 平面上正方形的几何形状❶。（顶点顺序是`GL_TRIANGLE_STRIP`，在第九章中介绍。）无论你展示的是与 *x*、*y* 还是 *z* 垂直的切片，你都将使用相同的几何形状。唯一不同的是，你选择显示的 3D 纹理中的数据平面。我们将在查看顶点着色器时回到这个思路。

这是一个渲染 2D 切片的方法：

def draw(self):

# 清除缓冲区

glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)

# 构建投影矩阵

❶ pMatrix = glutils.ortho(-0.6, 0.6, -0.6, 0.6, 0.1, 100.0)

# 模型视图矩阵

❷ mvMatrix = numpy.array([1.0, 0.0, 0.0, 0.0,

0.0, 1.0, 0.0, 0.0,

0.0, 0.0, 1.0, 0.0,

-0.5, -0.5, -1.0, 1.0], numpy.float32)

# 使用着色器

glUseProgram(self.program)

# 设置投影矩阵

glUniformMatrix4fv(self.pMatrixUniform, 1, GL_FALSE, pMatrix)

# 设置模型视图矩阵

glUniformMatrix4fv(self.mvMatrixUniform, 1, GL_FALSE, mvMatrix)

# 设置当前切片分数

❸ glUniform1f(glGetUniformLocation(self.program, b"uSliceFrac"),

float(self.currSliceIndex)/float(self.currSliceMax))

# 设置当前切片模式

❹ glUniform1i(glGetUniformLocation(self.program, b"uSliceMode"),

self.mode)

# 启用纹理

glActiveTexture(GL_TEXTURE0)

glBindTexture(GL_TEXTURE_3D, self.texture)

glUniform1i(glGetUniformLocation(self.program, b"tex"), 0)

# 绑定 VAO

glBindVertexArray(self.vao)

# 绘制

glDrawArrays(GL_TRIANGLE_STRIP, 0, 4)

# 解绑 VAO

glBindVertexArray(0)

每个 2D 切片是一个正方形，你通过使用 OpenGL 三角带原语来构建它。此代码展示了三角带的渲染设置。请注意，在❶处，你实现了正交投影，使用了`glutils.ortho()`方法。你设置了一个投影，在表示切片的单位正方形周围增加了一个 0.1 的缓冲区。

当你使用 OpenGL 绘制图形时，默认视图（未应用任何变换）将视点放置在 (0, 0, 0)，并沿 z 轴查看，y 轴朝上。对几何体应用平移（−0.5, −0.5, −1.0）后，几何体将位于 z 轴的中心 ❷。你设置当前的切片分数 ❸（例如，100 个切片中的第 10 个切片的分数是 0.1），设置切片模式 ❹（以查看 x、y 或 z 方向的切片，分别用整数 0、1 和 2 表示），并将这两个值传递给着色器。

#### 顶点着色器

现在我们来看 `SliceRender` 的顶点着色器：

strVS = """

# version 410 core

in vec3 aVert;

uniform mat4 uMVMatrix;

uniform mat4 uPMatrix;

uniform float uSliceFrac;

uniform int uSliceMode;

out vec3 texcoord;

void main() {

// x 切片

if (uSliceMode == 0) {

❶ texcoord = vec3(uSliceFrac, aVert.x, 1.0-aVert.y);

}

// y 切片

else if (uSliceMode == 1) {

❷ texcoord = vec3(aVert.x, uSliceFrac, 1.0-aVert.y);

}

// z 切片

else {

❸ texcoord = vec3(aVert.x, 1.0-aVert.y, uSliceFrac);

}

// 计算变换后的顶点

gl_Position = uPMatrix * uMVMatrix * vec4(aVert, 1.0);

}

"""

顶点着色器接受三角形条带顶点数组作为输入，并将纹理坐标作为输出。当前切片分数和切片模式作为统一变量`uSliceFrac`和`uSliceMode`传入。

该着色器有三个分支，取决于切片模式。例如，如果 `uSliceMode` 为 `0`，你将计算 x 切片的纹理坐标 ❶。因为你正在进行与 x 方向垂直的切片，所以希望切片与 YZ 平面平行。传入顶点着色器的 3D 顶点也充当 3D 纹理坐标，因为它们的范围是 [0, 1]，所以纹理坐标表示为（*f*，*V*[x]，*V*[y]），其中 *f* 是切片号在 x 轴方向上的分数，*V*[x] 和 *V*[y] 是顶点坐标。不幸的是，由于 OpenGL 坐标系的原点在左下角，y 方向朝上，这与期望的相反，因此生成的图像会倒置。为了解决这个问题，你将纹理坐标 *t* 改为 (1 − *t*)，并使用 (*f*, *V*[x], 1 − *V*[y]) ❶。你可以用类似的逻辑来计算 y 方向 ❷ 和 z 方向 ❸ 的切片纹理坐标，如果 `uSliceMode` 值分别为 `1` 或 `2`。

#### 片段着色器

下面是片段着色器：

strFS = """

# version 410 core

❶ in vec3 texcoord;

❷ uniform sampler3D texture;

out vec4 fragColor;

void main() {

// 查找纹理中的颜色

❸ vec4 col = texture(tex, texcoord);

❹ fragColor = col.rrra;

}

"""

片段着色器声明`texcoord`作为输入 ❶，它在顶点着色器中被设置为输出。纹理采样器声明为 `uniform` ❷。你通过 `texcoord` 查找纹理颜色 ❸ 并将 `fragColor` 作为输出 ❹。（因为你只读取纹理的红色通道，所以使用 `col.rrra`。）

#### 2D 切片用户界面

现在你需要为用户提供一种切片数据的方法。你可以通过`SliceRender`类中的键盘处理方法来实现：

def keyPressed(self, key):

"""按键处理器"""

❶ if key == 'x':

self.mode = SliceRender.XSLICE

# 重置切片索引

self.currSliceIndex = int(self.Nx/2)

self.currSliceMax = self.Nx

elif key == 'y':

self.mode = SliceRender.YSLICE

# 重置切片索引

self.currSliceIndex = int(self.Ny/2)

self.currSliceMax = self.Ny

elif key == 'z':

self.mode = SliceRender.ZSLICE

# 重置切片索引

self.currSliceIndex = int(self.Nz/2)

self.currSliceMax = self.Nz

elif key == 'l':

❷ self.currSliceIndex = (self.currSliceIndex + 1) % self.currSliceMax

elif key == 'r':

self.currSliceIndex = (self.currSliceIndex - 1) % self.currSliceMax

当键盘上的 X、Y 或 Z 键被按下时，`SliceRender`会切换到*x*、*y*或*z*切片模式。例如，按下*x*切片时 ❶，你会设置适当的模式，将当前切片索引设置为数据的中间位置，并更新最大切片数。

当键盘上的左箭头或右箭头键被按下时，你可以翻阅切片。例如，当按下左箭头键时，切片索引会递增 ❷。模运算符（`%`）确保当超出最大值时，索引会“回绕”到 0。

### 将代码合并

让我们快速看看项目中的主文件，*volrender.py*。这个文件使用了一个`RenderWin`类，用于创建和管理 GLFW OpenGL 窗口。 (我不会详细讲解这个类，因为它与第九章和第十章中使用的类相似。) 要查看完整的*volrender.py*代码，可以跳转到“完整的主文件代码”，见第 254 页。你也可以在[`github.com/mkvenkit/pp2e/tree/main/volrender`](https://github.com/mkvenkit/pp2e/tree/main/volrender)找到*volrender.py*文件。

在这个类的初始化代码中，你创建了渲染器，代码如下：

class RenderWin:

def __init__(self, imageDir):

--`snip`--

# 加载体积数据

❶ self.volume = volreader.loadVolume(imageDir)

# 创建渲染器

❷ self.renderer = RayCastRender(self.width, self.height, self.volume)

在这里，你通过`loadVolume()`函数将 3D 数据读取到 OpenGL 纹理中，我们之前讨论过这个函数 ❶。然后，你创建一个类型为`RayCastRender`的对象来显示数据 ❷。

#### 键盘按键处理器

`RenderWindow`类需要一个独立的键盘处理方法，用于切换体积渲染和切片渲染，并关闭窗口。这个方法还会将按键事件传递给`RayCastRender`和`SliceRender`类的键盘处理器，用于旋转相机或在 2D 切片中导航。

def onKeyboard(self, win, key, scancode, action, mods):

# 输出 'keyboard: ', win, key, scancode, action, mods

# 按 ESC 键退出

if key is glfw.GLFW_KEY_ESCAPE:

self.renderer.close()

self.exitNow = True

else:

❶ if action is glfw.GLFW_PRESS or action is glfw.GLFW_REPEAT:

if key == glfw.GLFW_KEY_V:

# 切换渲染模式

❷ if isinstance(self.renderer, RayCastRender):

self.renderer = SliceRender(self.width, self.height,

self.volume)

else:

self.renderer = RayCastRender(self.width, self.height,

self.volume)

# 调用渲染器的 reshape 方法

self.renderer.reshape(self.width, self.height)

else:

# 将按键事件传递给渲染器

❸ keyDict = {glfw.GLFW_KEY_X: 'x', glfw.GLFW_KEY_Y: 'y',

glfw.GLFW_KEY_Z: 'z', glfw.GLFW_KEY_LEFT: 'l',

glfw.GLFW_KEY_RIGHT: 'r'}

try:

self.renderer.keyPressed(keyDict[key])

except:

pass

按 ESC 键退出程序。你设置了其他按键，无论是按下键还是长按，都能生效❶。如果按下 V 键，你可以在体积渲染和切片渲染之间切换❷，并使用 Python 的 `isinstance()` 方法来识别当前类类型。为了处理其他按键事件（如 X、Y、Z 键，或左右箭头），你使用一个字典❸并将按键事件传递给当前渲染器的 `keyPressed()` 处理方法。我们在 “2D 切片用户界面” 第 237 页中查看了切片渲染器的 `keyPressed()` 方法。

注意：我选择不直接传递 `glfw.KEY` 值，而是使用字典将其转换为字符值，因为减少源文件中的依赖是一个好习惯。目前，项目中唯一依赖 GLFW 的文件是 *volrender.py*。如果你将 GLFW 特定类型传递到其他代码中，它们也需要导入并依赖 GLFW 库。如果你切换到其他 OpenGL 窗口工具包，代码会变得凌乱。

## 运行程序

这是使用斯坦福体积数据存档中的数据运行程序的一个示例：

$ `python volrender.py --dir mrbrain-8bit/`

你应该看到类似于图 11-6 的内容。

![](img/nsp-venkitachalam503045-f11006.jpg)

图 11-6：*volrender.py* 的一个示例运行。左侧是体积渲染图像，右侧是 2D 切片图像。

在应用程序运行时，使用 V 键在体积渲染和切片渲染之间切换。在切片模式下，使用 X、Y 和 Z 键来更改切片轴，使用箭头键来更改切片位置。

## 总结

在本章中，你实现了使用 Python 和 OpenGL 的体积光线投射算法。你学习了如何使用 GLSL 着色器高效地实现该算法，以及如何从体积数据中创建 2D 切片。

## 实验！

以下是几种你可以继续修改体积光线投射程序的方法：

1.  1\. 当前，在光线投射模式下，很难看到体积数据“立方体”的边界。实现一个类`WireFrame`，在该立方体周围绘制一个框。将 x、y 和 z 轴分别标记为红色、绿色和蓝色，并为每个轴指定自己的着色器。你将从`RayCastRender`类中使用`WireFrame`。

1.  2\. 实现数据缩放。在当前的实现中，你绘制了一个体积的立方体和一个二维切片的正方形，这假设你的数据集是对称的（即每个方向上的切片数相同），但大多数实际数据的切片数量是不同的。特别是医学数据，通常在 z 方向上有较少的切片，例如 256×256×99。为了正确显示这些数据，你需要在计算中引入缩放因子。一个方法是将缩放因子应用于立方体顶点（3D 体积）和正方形顶点（2D 切片）。用户可以通过命令行参数输入缩放参数。

1.  3\. 我们的体积光线投射实现使用 X 射线投射来计算像素的最终颜色或强度。另一种常见的方法是使用*最大强度投影 (MIP)* 在每个像素处设置最大强度。在你的代码中实现这一点。（提示：在`RayCastRender`的片段着色器中，修改通过光线进行步进的代码，检查并设置光线沿途的最大值，而不是混合值。）

1.  4\. 当前，你实现的唯一 UI 是围绕 x、y 和 z 轴的旋转。实现一个缩放功能，使得按下 I/O 键可以放大或缩小体积渲染图像。你可以通过在`glutils.lookAt()`方法中设置适当的相机参数来实现这一点，但有一个注意事项：如果你将视图移入数据立方体内部，光线投射将失败，因为 OpenGL 会裁剪立方体的前面面；光线投射所需的计算需要前后面都被正确渲染。相反，通过调整`glutils.projecton()`方法中的视野范围来进行缩放。

## 完整的 3D 纹理代码

这里是完整的*volreader.py*代码列表。

import os

import numpy as np

from PIL import Image

import OpenGL

from OpenGL.GL import *

from scipy import misc

def loadVolume(dirName):

"""从目录读取体积数据作为 3D 纹理"""

# 列出目录中的图像

files = sorted(os.listdir(dirName))

print('从 %s 加载图像' % dirName)

imgDataList = []

count = 0

width, height = 0, 0

for file in files:

file_path = os.path.abspath(os.path.join(dirName, file))

try:

# 读取图像

img = Image.open(file_path)

imgData = np.array(img.getdata(), np.uint8)

# 检查是否所有图像尺寸相同

if count is 0:

width, height = img.size[0], img.size[1]

imgDataList.append(imgData)

否则：

if (width, height) == (img.size[0], img.size[1]):

imgDataList.append(imgData)

否则：

print('不匹配')

raise RunTimeError("图像大小不匹配")

count += 1

# 打印 img.size

except:

# 跳过

print('无效的图像: %s' % file_path)

# 将图像数据加载到单一数组

depth = count

data = np.concatenate(imgDataList)

print('体积数据尺寸: %d %d %d' % (width, height, depth))

# 加载数据到 3D 纹理

texture = glGenTextures(1)

glPixelStorei(GL_UNPACK_ALIGNMENT, 1)

glBindTexture(GL_TEXTURE_3D, texture)

glTexParameterf(GL_TEXTURE_3D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE)

glTexParameterf(GL_TEXTURE_3D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE)

glTexParameterf(GL_TEXTURE_3D, GL_TEXTURE_WRAP_R, GL_CLAMP_TO_EDGE)

glTexParameterf(GL_TEXTURE_3D, GL_TEXTURE_MAG_FILTER, GL_LINEAR)

glTexParameterf(GL_TEXTURE_3D, GL_TEXTURE_MIN_FILTER, GL_LINEAR)

glTexImage3D(GL_TEXTURE_3D, 0, GL_RED,

width, height, depth, 0,

GL_RED, GL_UNSIGNED_BYTE, data)

# 返回纹理

return (texture, width, height, depth)

# 加载纹理

def loadTexture(filename):

img = Image.open(filename)

img_data = np.array(list(img.getdata()), 'B')

texture = glGenTextures(1)

glPixelStorei(GL_UNPACK_ALIGNMENT,1)

glBindTexture(GL_TEXTURE_2D, texture)

glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE)

glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE)

glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR)

glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR)

glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, img.size[0], img.size[1],

0, GL_RGBA, GL_UNSIGNED_BYTE, img_data)

return texture

## 完整的光线生成代码

这是`RayCube`类的完整代码。

import OpenGL

from OpenGL.GL import *

from OpenGL.GL.shaders import *

import numpy, math, sys

import volreader, glutils

strVS = """

# version 330 core

layout(location = 1) in vec3 cubePos;

layout(location = 2) in vec3 cubeCol;

uniform mat4 uMVMatrix;

uniform mat4 uPMatrix;

out vec4 vColor;

void main()

{

// 设置背面颜色

vColor = vec4(cubeCol.rgb, 1.0);

// 转换后的坐标

vec4 newPos = vec4(cubePos.xyz, 1.0);

// 设置位置

gl_Position = uPMatrix * uMVMatrix * newPos;

}

"""

strFS = """

# version 330 core

in vec4 vColor;

out vec4 fragColor;

void main()

{

fragColor = vColor;

}

"""

class RayCube:

"""用于生成光线的类，应用于射线投射"""

def __init__(self, width, height):

"""RayCube 构造函数"""

# 设置尺寸

self.width, self.height = width, height

# 创建着色器

self.program = glutils.loadShaders(strVS, strFS)

# 立方体顶点

vertices = numpy.array([

0.0, 0.0, 0.0,

1.0, 0.0, 0.0,

1.0, 1.0, 0.0,

0.0, 1.0, 0.0,

0.0, 0.0, 1.0,

1.0, 0.0, 1.0,

1.0, 1.0, 1.0,

0.0, 1.0, 1.0

], numpy.float32)

# 立方体颜色

colors = numpy.array([

0.0, 0.0, 0.0,

1.0, 0.0, 0.0,

1.0, 1.0, 0.0,

0.0, 1.0, 0.0,

0.0, 0.0, 1.0,

1.0, 0.0, 1.0,

1.0, 1.0, 1.0,

0.0, 1.0, 1.0

], numpy.float32)

# 单独的三角形

indices = numpy.array([

4, 5, 7,

7, 5, 6,

5, 1, 6,

6, 1, 2,

1, 0, 2,

2, 0, 3,

0, 4, 3,

3, 4, 7,

6, 2, 7,

7, 2, 3,

4, 0, 5,

5, 0, 1

], numpy.int16)

self.nIndices = indices.size

# 设置顶点数组对象 (VAO)

self.vao = glGenVertexArrays(1)

glBindVertexArray(self.vao)

# 顶点缓冲区

self.vertexBuffer = glGenBuffers(1)

glBindBuffer(GL_ARRAY_BUFFER, self.vertexBuffer)

glBufferData(GL_ARRAY_BUFFER, 4*len(vertices), vertices, GL_STATIC_DRAW)

# 顶点缓冲区 - 立方体顶点颜色

self.colorBuffer = glGenBuffers(1)

glBindBuffer(GL_ARRAY_BUFFER, self.colorBuffer)

glBufferData(GL_ARRAY_BUFFER, 4*len(colors), colors, GL_STATIC_DRAW);

# 索引缓冲区

self.indexBuffer = glGenBuffers(1)

glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, self.indexBuffer);

glBufferData(GL_ELEMENT_ARRAY_BUFFER, 2*len(indices), indices,

GL_STATIC_DRAW)

# 使用着色器中的布局索引启用属性

aPosLoc = 1

aColorLoc = 2

# 绑定缓冲区

glEnableVertexAttribArray(1)

glEnableVertexAttribArray(2)

# 顶点

glBindBuffer(GL_ARRAY_BUFFER, self.vertexBuffer)

glVertexAttribPointer(aPosLoc, 3, GL_FLOAT, GL_FALSE, 0, None)

# 颜色

glBindBuffer(GL_ARRAY_BUFFER, self.colorBuffer)

glVertexAttribPointer(aColorLoc, 3, GL_FLOAT, GL_FALSE, 0, None)

# 索引

glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, self.indexBuffer)

# 解绑 VAO

glBindVertexArray(0)

# FBO

self.initFBO()

def renderBackFace(self, pMatrix, mvMatrix):

"""渲染射线立方体的背面到纹理并返回"""

# 渲染到 FBO

glBindFramebuffer(GL_FRAMEBUFFER, self.fboHandle)

# 设置活动纹理

glActiveTexture(GL_TEXTURE0)

# 绑定到 FBO 纹理

glBindTexture(GL_TEXTURE_2D, self.texHandle)

# 启用面剔除时渲染立方体

self.renderCube(pMatrix, mvMatrix, self.program, True)

# 解绑纹理

glBindTexture(GL_TEXTURE_2D, 0)

glBindFramebuffer(GL_FRAMEBUFFER, 0)

glBindRenderbuffer(GL_RENDERBUFFER, 0)

# 返回纹理 ID

return self.texHandle

def renderFrontFace(self, pMatrix, mvMatrix, program):

"""渲染射线立方体的正面"""

# 不进行面剔除

self.renderCube(pMatrix, mvMatrix, program, False)

def renderCube(self, pMatrix, mvMatrix, program, cullFace):

"""使用面剔除渲染立方体（如果标志设置）"""

glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)

# 设置着色器程序

glUseProgram(program)

# 设置投影矩阵

glUniformMatrix4fv(glGetUniformLocation(program, b'uPMatrix'),

1, GL_FALSE, pMatrix)

# 设置模型视图矩阵

glUniformMatrix4fv(glGetUniformLocation(program, b'uMVMatrix'),

1, GL_FALSE, mvMatrix)

# 启用面剔除

glDisable(GL_CULL_FACE)

if cullFace:

glFrontFace(GL_CCW)

glCullFace(GL_FRONT)

glEnable(GL_CULL_FACE)

# 绑定 VAO

glBindVertexArray(self.vao)

# 动画切片

glDrawElements(GL_TRIANGLES, self.nIndices, GL_UNSIGNED_SHORT, None)

# 解绑 VAO

glBindVertexArray(0)

# 重置剔除面

if cullFace:

# 禁用面剔除

glDisable(GL_CULL_FACE)

def reshape(self, width, height):

self.width = width

self.height = height

self.aspect = width/float(height)

# 重新创建 FBO

self.clearFBO()

self.initFBO()

def initFBO(self):

# 创建帧缓冲对象

self.fboHandle = glGenFramebuffers(1)

# 创建纹理

self.texHandle = glGenTextures(1)

# 创建深度缓冲区

self.depthHandle = glGenRenderbuffers(1)

# 绑定

glBindFramebuffer(GL_FRAMEBUFFER, self.fboHandle)

glActiveTexture(GL_TEXTURE0)

glBindTexture(GL_TEXTURE_2D, self.texHandle)

# 设置参数以绘制不同大小的图像

glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR)

glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR)

glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE)

glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE)

# 设置纹理

glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, self.width, self.height,

0, GL_RGBA, GL_UNSIGNED_BYTE, None)

# 绑定纹理到 FBO

glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0,

GL_TEXTURE_2D, self.texHandle, 0)

# 绑定

glBindRenderbuffer(GL_RENDERBUFFER, self.depthHandle)

glRenderbufferStorage(GL_RENDERBUFFER, GL_DEPTH_COMPONENT24,

self.width, self.height)

# 绑定深度缓冲区到 FBO

glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_ATTACHMENT,

GL_RENDERBUFFER, self.depthHandle)

# 检查状态

status = glCheckFramebufferStatus(GL_FRAMEBUFFER)

if status == GL_FRAMEBUFFER_COMPLETE:

pass

# 打印 "fbo %d 完成" % self.fboHandle

elif status == GL_FRAMEBUFFER_UNSUPPORTED:

print("fbo %d 不支持" % self.fboHandle)

else:

print("fbo %d 错误" % self.fboHandle)

glBindTexture(GL_TEXTURE_2D, 0)

glBindFramebuffer(GL_FRAMEBUFFER, 0)

glBindRenderbuffer(GL_RENDERBUFFER, 0)

return

def clearFBO(self):

"""清除旧的 FBO"""

# 删除 FBO

if glIsFramebuffer(self.fboHandle):

glDeleteFramebuffers(1, int(self.fboHandle))

# 删除纹理

if glIsTexture(self.texHandle):

glDeleteTextures(int(self.texHandle))

def close(self):

"""调用此方法释放 OpenGL 资源"""

glBindTexture(GL_TEXTURE_2D, 0)

glBindFramebuffer(GL_FRAMEBUFFER, 0)

glBindRenderbuffer(GL_RENDERBUFFER, 0)

# 删除 FBO

if glIsFramebuffer(self.fboHandle):

glDeleteFramebuffers(1, int(self.fboHandle))

# 删除纹理

if glIsTexture(self.texHandle):

glDeleteTextures(int(self.texHandle))

# 删除渲染缓冲区

"""

if glIsRenderbuffer(self.depthHandle):

glDeleteRenderbuffers(1, int(self.depthHandle))

"""

# 删除缓冲区

"""

glDeleteBuffers(1, self._vertexBuffer)

glDeleteBuffers(1, &_indexBuffer)

glDeleteBuffers(1, &_colorBuffer)

"""

## 完整的体积光线投射代码

这是完整的*raycast.py*代码列表。

import OpenGL

from OpenGL.GL import *

from OpenGL.GL.shaders import *

import numpy as np

import math, sys

import raycube, glutils, volreader

strVS = """

# version 330 core

layout(location = 1) in vec3 cubePos;

layout(location = 2) in vec3 cubeCol;

uniform mat4 uMVMatrix;

uniform mat4 uPMatrix;

out vec4 vColor;

void main()

{

// 设置位置

gl_Position = uPMatrix * uMVMatrix * vec4(cubePos.xyz, 1.0);

// 设置颜色

vColor = vec4(cubeCol.rgb, 1.0);

}

"""

strFS = """

# version 330 core

in vec4 vColor;

uniform sampler2D texBackFaces;

uniform sampler3D texVolume;

uniform vec2 uWinDims;

out vec4 fragColor;

void main()

{

// 光线的起始点

vec3 start = vColor.rgb;

// 在片段着色器中计算纹理坐标，

// 它是窗口坐标的一个分数

vec2 texc = gl_FragCoord.xy/uWinDims.xy;

// 通过查找背面颜色获得光线终点

vec3 end = texture(texBackFaces, texc).rgb;

// 计算光线方向

vec3 dir = end - start;

// 规范化光线方向

vec3 norm_dir = normalize(dir);

// 从前到后的长度已计算

// 用于终止光线

float len = length(dir.xyz);

// 光线步长

float stepSize = 0.01;

// X 射线投影

vec4 dst = vec4(0.0);

// 通过光线进行步进

for(float t = 0.0; t < len; t += stepSize) {

// 设置位置为光线的终点

vec3 samplePos = start + t*norm_dir;

// 获取位置的纹理值

float val = texture(texVolume, samplePos).r;

vec4 src = vec4(val);

// 设置不透明度

src.a *= 0.1;

src.rgb *= src.a;

// 与之前的值混合

dst = (1.0 - dst.a)*src + dst;

// 当 alpha 超过阈值时退出循环

if(dst.a >= 0.95)

break;

}

// 设置片段颜色

fragColor = dst;

}

"""

class Camera:

"""辅助类用于查看"""

def __init__(self):

self.r = 1.5

self.theta = 0

self.center = [0.5, 0.5, 0.5]

self.eye = [0.5 + self.r, 0.5, 0.5]

self.up = [0.0, 0.0, 1.0]

def rotate(self, clockWise):

"""将眼睛旋转一步"""

if clockWise:

self.theta = (self.theta + 5) % 360

else:

self.theta = (self.theta - 5) % 360

# 重新计算眼睛位置

self.eye = [0.5 + self.r*math.cos(math.radians(self.theta)),

0.5 + self.r*math.sin(math.radians(self.theta)),

0.5]

class RayCastRender:

"""进行光线投射的类"""

def __init__(self, width, height, volume):

"""RayCastRender 构造函数"""

# 创建 RayCube 对象

self.raycube = raycube.RayCube(width, height)

# 设置尺寸

self.width = width

self.height = height

self.aspect = width/float(height)

# 创建着色器

self.program = glutils.loadShaders(strVS, strFS)

# 纹理

self.texVolume, self.Nx, self.Ny, self.Nz = volume

# 初始化相机

self.camera = Camera()

def draw(self):

# 构建投影矩阵

pMatrix = glutils.perspective(45.0, self.aspect, 0.1, 100.0)

# 模型视图矩阵

mvMatrix = glutils.lookAt(self.camera.eye, self.camera.center,

self.camera.up)

# 渲染

# 生成 ray-cube 背面纹理

texture = self.raycube.renderBackFace(pMatrix, mvMatrix)

# 设置着色器程序

glUseProgram(self.program)

# 设置窗口尺寸

glUniform2f(glGetUniformLocation(self.program, b"uWinDims"),

float(self.width), float(self.height))

# 纹理单元 0，表示立方体的背面

glActiveTexture(GL_TEXTURE0)

glBindTexture(GL_TEXTURE_2D, texture)

glUniform1i(glGetUniformLocation(self.program, b"texBackFaces"), 0)

# 纹理单元 1: 3D 体积纹理

glActiveTexture(GL_TEXTURE1)

glBindTexture(GL_TEXTURE_3D, self.texVolume)

glUniform1i(glGetUniformLocation(self.program, b"texVolume"), 1)

# 绘制立方体的前面

self.raycube.renderFrontFace(pMatrix, mvMatrix, self.program)

#self.render(pMatrix, mvMatrix)

def keyPressed(self, key):

if key == 'l':

self.camera.rotate(True)

elif key == 'r':

self.camera.rotate(False)

def reshape(self, width, height):

self.width = width

self.height = height

self.aspect = width/float(height)

self.raycube.reshape(width, height)

def close(self):

self.raycube.close()

## 完整的 2D 切片代码

这里是完整的 2D 切片代码列表。

import OpenGL

from OpenGL.GL import *

from OpenGL.GL.shaders import *

import numpy, math, sys

import volreader, glutils

strVS = """

# version 330 core

in vec3 aVert;

uniform mat4 uMVMatrix;

uniform mat4 uPMatrix;

uniform float uSliceFrac;

uniform int uSliceMode;

out vec3 texcoord;

void main() {

// x 切片

if (uSliceMode == 0) {

texcoord = vec3(uSliceFrac, aVert.x, 1.0-aVert.y);

}

// y 切片

else if (uSliceMode == 1) {

texcoord = vec3(aVert.x, uSliceFrac, 1.0-aVert.y);

}

// z 切片

else {

texcoord = vec3(aVert.x, 1.0-aVert.y, uSliceFrac);

}

// 计算变换后的顶点

gl_Position = uPMatrix * uMVMatrix * vec4(aVert, 1.0);

}

"""

strFS = """

# version 330 core

in vec3 texcoord;

uniform sampler3D tex;

out vec4 fragColor;

void main() {

// 在纹理中查找颜色

vec4 col = texture(tex, texcoord);

fragColor = col.rrra;

}

"""

class SliceRender:

# 切片模式

XSLICE, YSLICE, ZSLICE = 0, 1, 2

def __init__(self, width, height, volume):

"""SliceRender 构造函数"""

self.width = width

self.height = height

self.aspect = width/float(height)

# 切片模式

self.mode = SliceRender.ZSLICE

# 创建着色器

self.program = glutils.loadShaders(strVS, strFS)

glUseProgram(self.program)

self.pMatrixUniform = glGetUniformLocation(self.program, b'uPMatrix')

self.mvMatrixUniform = glGetUniformLocation(self.program, b"uMVMatrix")

# 属性

self.vertIndex = glGetAttribLocation(self.program, b"aVert")

# 设置顶点数组对象（VAO）

self.vao = glGenVertexArrays(1)

glBindVertexArray(self.vao)

# 定义四边形顶点

vertexData = numpy.array([0.0, 1.0, 0.0,

0.0, 0.0, 0.0,

1.0, 1.0, 0.0,

1.0, 0.0, 0.0], numpy.float32)

# 顶点缓冲区

self.vertexBuffer = glGenBuffers(1)

glBindBuffer(GL_ARRAY_BUFFER, self.vertexBuffer)

glBufferData(GL_ARRAY_BUFFER, 4*len(vertexData), vertexData,

GL_STATIC_DRAW)

# 启用数组

glEnableVertexAttribArray(self.vertIndex)

# 设置缓冲区

glBindBuffer(GL_ARRAY_BUFFER, self.vertexBuffer)

glVertexAttribPointer(self.vertIndex, 3, GL_FLOAT, GL_FALSE, 0, None)

# 解绑 VAO

glBindVertexArray(0)

# 加载纹理

self.texture, self.Nx, self.Ny, self.Nz = volume

# 当前切片索引

self.currSliceIndex = int(self.Nz/2);

self.currSliceMax = self.Nz;

def reshape(self, width, height):

self.width = width

self.height = height

self.aspect = width/float(height)

def draw(self):

# 清除缓冲区

glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)

# 构建投影矩阵

pMatrix = glutils.ortho(-0.6, 0.6, -0.6, 0.6, 0.1, 100.0)

# 视图矩阵

mvMatrix = numpy.array([1.0, 0.0, 0.0, 0.0,

0.0, 1.0, 0.0, 0.0,

0.0, 0.0, 1.0, 0.0,

-0.5, -0.5, -1.0, 1.0], numpy.float32)

# 使用着色器

glUseProgram(self.program)

# 设置投影矩阵

glUniformMatrix4fv(self.pMatrixUniform, 1, GL_FALSE, pMatrix)

# 设置视图矩阵

glUniformMatrix4fv(self.mvMatrixUniform, 1, GL_FALSE, mvMatrix)

# 设置当前切片比例

glUniform1f(glGetUniformLocation(self.program, b"uSliceFrac"),

float(self.currSliceIndex)/float(self.currSliceMax))

# 设置当前切片模式

glUniform1i(glGetUniformLocation(self.program, b"uSliceMode"),

self.mode)

# 启用纹理

glActiveTexture(GL_TEXTURE0)

glBindTexture(GL_TEXTURE_3D, self.texture)

glUniform1i(glGetUniformLocation(self.program, b"tex"), 0)

# 绑定 VAO

glBindVertexArray(self.vao)

# 绘制

glDrawArrays(GL_TRIANGLE_STRIP, 0, 4)

# 解绑 VAO

glBindVertexArray(0)

def keyPressed(self, key):

"""按键处理函数"""

if key == 'x':

self.mode = SliceRender.XSLICE

# 重置切片索引

self.currSliceIndex = int(self.Nx/2)

self.currSliceMax = self.Nx

elif key == 'y':

self.mode = SliceRender.YSLICE

# 重置切片索引

self.currSliceIndex = int(self.Ny/2)

self.currSliceMax = self.Ny

elif key == 'z':

self.mode = SliceRender.ZSLICE

# 重置切片索引

self.currSliceIndex = int(self.Nz/2)

self.currSliceMax = self.Nz

elif key == 'l':

self.currSliceIndex = (self.currSliceIndex + 1) % self.currSliceMax

elif key == 'r':

self.currSliceIndex = (self.currSliceIndex - 1) % self.currSliceMax

def close(self):

pass

## 完整的主文件代码

这里是主文件的完整代码列表。

import sys, argparse, os

from slicerender import *

from raycast import *

import glfw

class RenderWin:

"""GLFW 渲染窗口类"""

def __init__(self, imageDir):

# 保存当前工作目录

cwd = os.getcwd()

# 初始化 glfw; 这会改变 cwd

glfw.glfwInit()

# 恢复 cwd

os.chdir(cwd)

# 版本提示

glfw.glfwWindowHint(glfw.GLFW_CONTEXT_VERSION_MAJOR, 3)

glfw.glfwWindowHint(glfw.GLFW_CONTEXT_VERSION_MINOR, 3)

glfw.glfwWindowHint(glfw.GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE)

glfw.glfwWindowHint(glfw.GLFW_OPENGL_PROFILE,

glfw.GLFW_OPENGL_CORE_PROFILE)

# 创建窗口

self.width, self.height = 512, 512

self.aspect = self.width/float(self.height)

self.win = glfw.glfwCreateWindow(self.width, self.height, b"volrender")

# 使上下文当前

glfw.glfwMakeContextCurrent(self.win)

# 初始化 GL

glViewport(0, 0, self.width, self.height)

glEnable(GL_DEPTH_TEST)

glClearColor(0.0, 0.0, 0.0, 0.0)

# 设置窗口回调函数

glfw.glfwSetMouseButtonCallback(self.win, self.onMouseButton)

glfw.glfwSetKeyCallback(self.win, self.onKeyboard)

glfw.glfwSetWindowSizeCallback(self.win, self.onSize)

# 加载体积数据

self.volume = volreader.loadVolume(imageDir)

# 创建渲染器

self.renderer = RayCastRender(self.width, self.height, self.volume)

# 退出标志

self.exitNow = False

def onMouseButton(self, win, button, action, mods):

# 打印 'mouse button: ', win, button, action, mods

pass

def onKeyboard(self, win, key, scancode, action, mods):

# 打印 'keyboard: ', win, key, scancode, action, mods

# ESC 退出

if key is glfw.GLFW_KEY_ESCAPE:

self.renderer.close()

self.exitNow = True

else:

if action is glfw.GLFW_PRESS or action is glfw.GLFW_REPEAT:

if key == glfw.GLFW_KEY_V:

# 切换渲染模式

if isinstance(self.renderer, RayCastRender):

self.renderer = SliceRender(self.width, self.height,

self.volume)

else:

self.renderer = RayCastRender(self.width, self.height,

self.volume)

# 调用渲染器的 reshape

self.renderer.reshape(self.width, self.height)

else:

# 发送按键到渲染器

keyDict = {glfw.GLFW_KEY_X: 'x', glfw.GLFW_KEY_Y: 'y',

glfw.GLFW_KEY_Z: 'z', glfw.GLFW_KEY_LEFT: 'l',

glfw.GLFW_KEY_RIGHT: 'r'}

try:

self.renderer.keyPressed(keyDict[key])

except:

pass

def onSize(self, win, width, height):

# 打印 'onsize: ', win, width, height

self.width = width

self.height = height

self.aspect = width/float(height)

glViewport(0, 0, self.width, self.height)

self.renderer.reshape(width, height)

def run(self):

# 开始循环

while not glfw.glfwWindowShouldClose(self.win) and not self.exitNow:

# 渲染

self.renderer.draw()

# 交换缓冲区

glfw.glfwSwapBuffers(self.win)

# 等待事件

glfw.glfwWaitEvents()

# 结束

glfw.glfwTerminate()

# main() function

def main():

print('开始体积渲染...')

# 创建解析器

parser = argparse.ArgumentParser(description="体积渲染...")

# 添加预期参数

parser.add_argument('--dir', dest='imageDir', required=True)

# 解析参数

args = parser.parse_args()

# 创建渲染窗口

rwin = RenderWin(args.imageDir)

rwin.run()

# 调用 main

if __name__ == '__main__':

main()

1 J. Kruger 和 R. Westermann, “基于 GPU 的体积渲染加速技术，”IEEE Visualization，2003 年。

2 [`graphics.stanford.edu/data/voldata/`](https://graphics.stanford.edu/data/voldata/)
