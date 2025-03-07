# 6

# ASCII 艺术

![](img/nsp-venkitachalam503045-circle-image.jpg)

在 1990 年代，当电子邮件占主导地位且图形功能有限时，在电子邮件中包含由文本构成的图形是很常见的，这种图形通常被称为 *ASCII 艺术*。（ASCII 只是一个字符编码方案。）图 6-1 展示了几个示例。尽管互联网使得分享图像变得极为方便，但这种简朴的文本图形仍然没有完全消失。

ASCII 艺术起源于 19 世纪末期的打字机艺术。在 1960 年代，当计算机的图形处理硬件有限时，ASCII 被用来表示图像。如今，ASCII 艺术作为一种表达形式仍然活跃在互联网上，你可以在网上找到各种创意的例子。

![](img/nsp-venkitachalam503045-f06001.jpg)

图 6-1：ASCII 艺术示例

在这个项目中，你将使用 Python 创建一个从图像生成 ASCII 艺术的程序。该程序将允许你指定输出的宽度（即文本列数）并设置垂直缩放因子。它还支持两种灰度值到 ASCII 字符的映射：一种是稀疏的 10 级映射，另一种是更精细调校的 70 级映射。

要从图像生成 ASCII 艺术，你将学习如何执行以下操作：

+   • 使用 `Pillow`，即 Python 图像库（PIL）的一个分支，将图像转换为灰度。

+   • 使用 `numpy` 计算灰度图像的平均亮度。

+   • 使用字符串作为灰度值的快速查找表。

## 工作原理

这个项目利用了一个事实：从远处看，我们将灰度图像视为其亮度的平均值。例如，在图 6-2 中，你可以看到一幅建筑物的灰度图像，旁边是填充了建筑物图像平均亮度值的图像。如果你从房间另一端看这些图像，它们看起来会很相似。

ASCII 艺术是通过将图像拆分成多个小块，并根据每个小块的平均亮度值用一个 ASCII 字符替换每个小块来生成的。较亮的小块用较稀疏的 ASCII 字符替换（即包含大量空白的字符），例如句点（`.`）或冒号（`:`），而较暗的小块则用更密集的 ASCII 字符替换，例如和号（`@`）或美元符号（`$`）。由于我们的眼睛分辨率有限，从远处看时，我们会看到 ASCII 艺术的“平均”值，而失去那些本应让艺术看起来更真实的细节。

![](img/nsp-venkitachalam503045-f06002.jpg)

图 6-2：灰度图像的平均值

该程序会接受一个给定的图像，并首先将其转换为 8 位灰度图，使得每个像素的灰度值位于[0, 255]的范围内（这是 8 位整数的范围）。可以将这个 8 位值看作像素的*亮度*，其中 0 表示黑色，255 表示白色，介于其间的值则是灰色的不同阴影。

接下来，程序会将图像拆分成*M*×*N*的图块（其中*M*是 ASCII 图像中的行数，*N*是列数）。然后，程序会计算每个图块的平均亮度值，并通过预定义的*渐变*（一组递增的值）来为每个图块匹配一个合适的 ASCII 字符，表示灰度值范围[0, 255]。它将使用这些渐变值作为亮度值的查找表。

完成的 ASCII 艺术仅仅是一串文本行。为了显示这些文本，你将使用一种等宽（也叫做*等距*）字体，比如 Courier，因为除非每个字符的宽度相同，否则图像中的字符无法在网格中正确对齐，最终会导致输出不均匀且混乱。

字体的*长宽比*（宽度与高度的比率）也会影响最终的图像。如果字符占据的空间的长宽比与它所替换的图像块的长宽比不同，最终的 ASCII 图像将显得变形。实际上，你是在尝试用一个 ASCII 字符替换图像块，因此它们的形状需要匹配。例如，如果你将图像拆分成方形的图块，并用一种字符比宽度高的字体替换每个图块，那么最终的输出会显得在垂直方向上被拉伸。为了解决这个问题，你需要调整网格中的行数以匹配 Courier 字体的长宽比。（你可以通过发送程序命令行参数来修改缩放比例，以匹配其他字体。）

总结一下，程序生成 ASCII 图像的步骤如下：

1.  1\. 将输入图像转换为灰度图。

1.  2\. 将图像拆分成*M*×*N*的图块。

1.  3\. 修正*M*（行数），以匹配图像和字体的长宽比。

1.  4\. 计算每个图像块的平均亮度，然后为每个块查找一个合适的 ASCII 字符。

1.  5\. 组装 ASCII 字符字符串行并将其打印到文件中，形成最终图像。

## 要求

在这个项目中，你将使用`Pillow`，一个友好的 Python 图像库分支，来读取图像、访问其底层数据，并进行创建和修改。你还将使用`numpy`库来计算平均值。

## 代码

你将首先定义用于生成 ASCII 艺术的灰度级别。然后，你会看到图像是如何被分割成瓦片的，以及如何计算这些瓦片的平均亮度。接着，你将工作于用 ASCII 字符替换瓦片来生成最终输出。最后，你将设置命令行解析，以便用户指定输出大小、输出文件名等。

要查看完整的项目代码，请跳到“完整代码”部分，见第 109 页。你还可以从[`github.com/mkvenkit/pp2e/blob/main/ascii/ascii.py`](https://github.com/mkvenkit/pp2e/blob/main/ascii/ascii.py)下载此项目的代码。

### 定义灰度级别和网格

作为创建程序的第一步，定义你将用来将图像亮度值转换为 ASCII 字符的灰度级别作为全局变量。

# 70 级灰度

gscale1 = "$@B%8&WM#*oahkbdpqwmZO0QLCJUYXzcvunxrjft/\|()1{}[]?-_+~<>i!lI;:,\"^`'. "

# 10 级灰度

gscale2 = "@%#*+=-:. "

值`gscale1`是一个 70 级灰度渐变，而`gscale2`是一个更简单的 10 级灰度渐变。这两个值都以字符串的形式存储，字符的范围从最暗到最亮。程序默认使用`gscale2`渐变，但你可以通过命令行选项来选择使用更细致的`gscale1`渐变。

注意：要了解更多关于如何将字符表示为灰度值的信息，请参阅 Paul Bourke 的“灰度图像的字符表示”：[`paulbourke.net/dataformats/asciiart/`](http://paulbourke.net/dataformats/asciiart/)。

现在你已经有了灰度渐变，可以设置图像。以下代码使用`Pillow`打开图像并将其分割成网格：

# 打开图像并转换为灰度

image = Image.open(fileName).convert("L")

# 存储图像尺寸

❶ W, H = image.size[0], image.size[1]

# 计算瓦片宽度

❷ w = W/cols

# 根据纵横比和字体的缩放比例计算瓦片的高度

❸ h = w/scale

# 计算最终网格中应使用的行数

❹ rows = int(H/h)

首先，`Image.``open()` 打开输入图像文件，`Image.``convert()` 将图像转换为灰度图。`"L"` 代表*亮度*，是衡量图像亮度的标准。你存储输入图像的宽度和高度（以像素为单位）❶。然后，你计算瓦片的宽度，这是根据用户指定的列数（`cols`）来计算的❷。（如果用户在命令行中没有设置其他值，程序默认使用 80 列。）你使用浮点数除法，而非整数除法，来避免计算瓦片尺寸时出现截断误差。

一旦你知道了瓷砖的宽度，你就可以使用传入的垂直缩放因子`scale` ❸来计算其高度。这样，每个瓷砖都会与您用来显示文本的字体的纵横比匹配，从而确保最终图像不会被拉伸。`scale`的值可以作为参数传入，或者默认为`0.43`，这个值适合在 Courier 字体下显示结果。在计算了每一行的高度后，你需要计算网格中的行数 ❹。

### 计算平均亮度

接下来，你需要一种方法来计算灰度图像中每个瓷砖的平均亮度。函数`getAverageL()`完成这项工作。

def getAverageL(image):

# 获取图像作为 numpy 数组

❶ im = np.array(image)

# 获取维度

w,h = im.shape

# 获取平均亮度

❷ return np.average(im.reshape(w*h))

图像瓷砖作为 PIL `Image`对象传递给函数。你将图像转换为`numpy`数组 ❶，此时`im`成为一个包含图像像素亮度值的二维数组。你存储数组的维度（宽度和高度），然后使用`numpy.reshape()`将二维数组转换为一个一维数组，其长度是原数组宽度和高度的乘积（`w*h`）。你将重塑后的数组传递给`numpy.average()`，它对数组值进行求和并计算整个图像瓷砖的平均亮度 ❷。

### 从图像生成 ASCII 内容

程序的主要部分是从图像生成 ASCII 内容：

# ASCII 图像是一个字符字符串的列表

❶ aimg = []

# 生成瓷砖尺寸列表

❷ for j in range(rows):

y1 = int(j*h)

y2 = int((j+1)*h)

# 修正最后一个瓷砖

if j == rows-1:

❸ y2 = H

# 添加空字符串

❹ aimg.append("")

❺ for i in range(cols):

# 裁剪图像以适应瓷砖

x1 = int(i*w)

x2 = int((i+1)*w)

# 修正最后一个瓷砖

if i == cols-1:

x2 = W

# 裁剪图像以将瓷砖提取到另一个 Image 对象中

❻ img = image.crop((x1, y1, x2, y2))

# 获取平均亮度

❼ avg = int(getAverageL(img))

# 查找灰度值（avg）对应的 ASCII 字符

if moreLevels:

❽ gsval = gscale1[int((avg*69)/255)]

else:

❾ gsval = gscale2[int((avg*9)/255)]

# 将 ASCII 字符附加到字符串

❿ aimg[j] += gsval

在程序的这一部分，ASCII 图像首先作为字符串列表存储，你需要先初始化它 ❶。接下来，你遍历图像瓷砖的行 ❷，计算每个图像瓷砖在给定行中的上下 y 坐标，分别为*y1*和*y2*。这些是浮点计算，但在传递给图像裁剪方法之前，你会将它们截断为整数。

接下来，由于将图像分割成瓦片时，只有当图像的宽度是列数的整数倍时，才能使边缘瓦片大小相同，因此，你通过将 y 坐标设置为图像的实际高度（`H`）❸来修正最后一行瓦片的底部 y 坐标。这样，你就能确保图像的底部边缘不会被截断。

你在 ASCII 图像列表中添加了一个空字符串，以紧凑的方式表示当前图像行❹。接下来，你将填充这个字符串。实际上，你将这个字符串当作一个可以附加字符的字符列表来处理。然后，你逐列遍历图像中的所有瓦片❺。你计算每个瓦片的左侧和右侧 x 坐标，即*x1*和*x2*。当你到达行中的最后一个瓦片时，你将右侧 x 坐标设置为图像的宽度（`W`），这是因为你已经修正了最后一个 y 坐标为图像的高度。

现在，你已经计算出了(*x1*, *y1*)和(*x2*, *y2*)，即当前图像瓦片的左上角和右下角坐标。你将这些坐标传递给`image.crop()`，从完整的图像中提取出瓦片❻。然后，你将该瓦片（以 PIL `Image`对象的形式）传递给`getAverageL()`函数❼，该函数在“计算平均亮度”一节中定义，并位于第 105 页，以获取瓦片的平均亮度。你将平均亮度值从[0, 255]的范围缩放到[0, 9]，这是默认 10 级灰度范围的数值范围❾。然后，你使用`gscale2`（存储的灰度范围字符串）作为查找表，查找相关的 ASCII 字符。❽行类似，只不过它将亮度值缩放到[0, 69]范围内，这是 70 级灰度范围。此行代码仅在设置了`moreLevels`命令行标志时使用。最后，你将查找的 ASCII 字符`gsval`附加到文本行❿，代码循环直到所有行都处理完成。

### 创建命令行选项

接下来，为程序定义一些命令行选项。此代码使用内置的`argparse.ArgumentParser`类：

parser = argparse.ArgumentParser(description="descStr")

# 添加预期的参数

parser.add_argument('--file', dest='imgFile', required=True)

parser.add_argument('--scale', dest='scale', required=False)

parser.add_argument('--out', dest='outFile', required=False)

parser.add_argument('--cols', dest='cols', required=False)

parser.add_argument('--morelevels', dest='moreLevels', action='store_true')

你包括了以下选项：

--file 指定输入的图像文件。这是唯一必需的参数。

--scale 设置字体的垂直缩放因子，适用于除 Courier 以外的字体。

--out 设置生成的 ASCII 艺术的输出文件名。默认为*out.txt*。

--cols 设置 ASCII 输出中的文本列数。

--morelevels 选择 70 级灰度渐变，而不是默认的 10 级渐变。

### 将 ASCII 艺术字符串写入文本文件

最后，将生成的 ASCII 字符串列表写入文本文件：

# 打开一个新的文本文件

❶ f = open(outFile, 'w')

# 将列表中的每个字符串写入新文件

❷ 对于 aimg 中的每一行：

f.write(row + '\n')

# 清理

❸ f.close()

你使用内建的 `open()` 函数打开一个新的文本文件进行写入 ❶。然后，你遍历 `aimg` 列表中的每一行，将它写入文件 ❷。完成后，你关闭文件对象以释放系统资源 ❸。

## 运行 ASCII 艺术生成器

要运行完成的程序，请输入如下命令，替换 `data/robot.jpg` 为你要使用的图像文件的相对路径：

$ `python ascii.py --file` `data/robot.jpg` `--cols 100`

图 6-3 显示了将图像 *robot.jpg*（左侧）发送后的 ASCII 艺术效果。尝试添加 `--morelevels` 选项，查看 70 级灰度渐变与 10 级渐变的对比。

![](img/nsp-venkitachalam503045-f06003.jpg)

图 6-3：*ascii.py* 的示例运行

现在，你已经准备好创建自己的 ASCII 艺术了！

## 总结

在这个项目中，你学习了如何从任何输入图像生成 ASCII 艺术。在这个过程中，你学习了如何将图像分割成瓦片网格，如何计算每个瓦片的平均亮度值，以及如何根据亮度值用字符替代每个瓦片。祝你玩得开心，创造属于自己的 ASCII 艺术！

## 实验！

以下是进一步探索 ASCII 艺术的一些想法：

1.  1\. 使用命令行选项 `--scale 1.0` 运行程序。生成的图像如何？尝试使用不同的 `scale` 值。将输出复制到文本编辑器，并尝试设置不同的等宽字体，看看这样做如何影响最终图像的外观。

1.  2\. 向程序添加一个命令行选项 `--invert`，使生成的 ASCII 图像反转，黑色变为白色，反之亦然。（提示：尝试在查找时将瓦片亮度值从 255 中减去。）

1.  3\. 在此项目中，您为基于两个硬编码字符渐变的灰度值创建了查找表。实现一个命令行选项，传入不同的字符渐变来创建 ASCII 艺术，例如：

    $ ``python ascii.py --map "@$%^`."``

    这将使用给定的六字符渐变生成 ASCII 输出，其中 `@` 映射为亮度值 0，`.` 映射为亮度值 255。

## 完整代码

这是完整的 ASCII 艺术程序。

"""

ascii.py

一个将图像转换为 ASCII 艺术的 Python 程序。

作者：Mahesh Venkitachalam

"""

导入 sys、random、argparse

导入 numpy 为 np

导入 math

from PIL import Image

# 灰度级值来源于：

# http://paulbourke.net/dataformats/asciiart/

# 70 级灰度

gscale1 = "$@B%8&WM#*oahkbdpqwmZO0QLCJUYXzcvunxrjft/\|()1{}[]?-_+~<>i!lI;:,\"^`'. "

# 10 级灰度

gscale2 = '@%#*+=-:. '

def getAverageL(image):

"""

给定 PIL 图像，返回灰度值的平均值

"""

# 将图像转换为 numpy 数组

im = np.array(image)

# 获取形状

w, h = im.shape

# 获取平均值

return np.average(im.reshape(w * h))

def convertImageToAscii(fileName, cols, scale, moreLevels):

"""

给定图像和尺寸（行数，列数）返回一个 m * n 的图像列表

"""

# 声明全局变量

global gscale1, gscale2

# 打开图像并转换为灰度图

image = Image.open(fileName).convert('L')

# 存储尺寸

W, H = image.size[0], image.size[1]

print("输入图像尺寸: {} x {}".format(W, H))

# 计算图块宽度

w = W / cols

# 根据纵横比和缩放比例计算图块高度

h = w / scale

# 计算行数

rows = int(H / h)

print("列数: {}, 行数: {}".format(cols, rows))

print("图块尺寸: {} x {}".format(w, h))

# 检查图像尺寸是否过小

if cols > W or rows > H:

print("图像太小，无法匹配指定的列数！")

exit(0)

# 一个 ASCII 图像是由字符字符串组成的列表

aimg = []

# 生成尺寸列表

for j in range(rows):

y1 = int(j * h)

y2 = int((j + 1) * h)

# 修正最后一个图块

if j == rows - 1:

y2 = H

# 添加一个空字符串

aimg.append("")

for i in range(cols):

# 裁剪图像以获取图块

x1 = int(i * w)

x2 = int((i + 1) * w)

# 修正最后一个图块

if i == cols - 1:

x2 = W

# 裁剪图像以提取图块

img = image.crop((x1, y1, x2, y2))

# 获取平均亮度

avg = int(getAverageL(img))

# 查找 ASCII 字符

if moreLevels:

gsval = gscale1[int((avg * 69) / 255)]

else:

gsval = gscale2[int((avg * 9) / 255)]

# 将 ASCII 字符添加到字符串

aimg[j] += gsval

# 返回图像

return aimg

# main() 函数

def main():

# 创建解析器

descStr = "该程序将图像转换为 ASCII 艺术。"

parser = argparse.ArgumentParser(description=descStr)

# 添加期望的参数

parser.add_argument('--file', dest='imgFile', required=True)

parser.add_argument('--scale', dest='scale', required=False)

parser.add_argument('--out', dest='outFile', required=False)

parser.add_argument('--cols', dest='cols', required=False)

parser.add_argument('--morelevels', dest='moreLevels', action='store_true')

# 解析参数

args = parser.parse_args()

imgFile = args.imgFile

# 设置输出文件

outFile = 'out.txt'

if args.outFile:

outFile = args.outFile

# 设置默认缩放比例为 0.43，适用于 Courier 字体

scale = 0.43

if args.scale:

scale = float(args.scale)

# 设置列数

cols = 80

if args.cols:

cols = int(args.cols)

print('生成 ASCII 艺术...')

# 将图像转换为 ASCII 文本

aimg = convertImageToAscii(imgFile, cols, scale, args.moreLevels)

# 打开文件

f = open(outFile, 'w')

# 写入文件

for row in aimg:

f.write(row + '\n')

# 清理

f.close()

print("ASCII 艺术已写入 {}.".format(outFile))

# 调用 main

if __name__ == '__main__':

main()
