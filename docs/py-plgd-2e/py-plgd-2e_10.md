# 7

# 摄影马赛克

![](img/nsp-venkitachalam503045-circle-image.jpg)

当我上六年级时，我看到了像图 7-1 中展示的那种图片，但当时并没有完全弄明白它是什么。经过一阵眯眼看后，我终于搞明白了。（把书倒过来，从房间对面看。我不会告诉任何人。）

这个谜题之所以有效，是因为人眼的工作方式。图中显示的低分辨率、块状图像近距离时很难辨认，但当从远处观看时，你能够识别出它代表的内容，因为你的眼睛感知到的细节较少，从而使得边缘变得平滑。

*摄影马赛克*是基于类似原理的图像。你从一个*目标*图像开始，将其拆分成矩形网格，并用另一个匹配该部分的较小图像替换每个矩形。当你从远处看摄影马赛克时，看到的就是目标图像，但如果你走近了，秘密就会揭开：图像实际上由许多微小的图像组成！

![](img/nsp-venkitachalam503045-f07001.jpg)

图 7-1：一个令人困惑的图像

在这个项目中，你将使用 Python 创建一个摄影马赛克。你将把目标图像划分成网格，并用适当的图像替换网格中的每个块，从而创建一个原始图像的摄影马赛克。你将能够指定网格的尺寸，并选择是否在马赛克中重复使用输入图像。

在你进行项目的过程中，你将学会以下技能：

+   • 使用 Python 图像库（PIL）创建图像。

+   • 计算图像的平均 RGB 值。

+   • 裁剪图像。

+   • 用另一张图像粘贴替换图像的一部分。

+   • 使用三维平均距离的度量来比较 RGB 值。

+   • 使用一种称为*k-d 树*的数据结构，来高效地找到与目标图像某一部分最匹配的图像。

## 工作原理

为了创建一个摄影马赛克，首先从一个块状、低分辨率的目标图像开始（因为在高分辨率图像中瓷砖图像的数量会太多）。用户输入马赛克的尺寸*M*×*N*（其中*M*是行数，*N*是列数）。接下来，按照这种方法构建马赛克：

1.  1\. 读取输入图像，这些图像将被绘制以替换原始图像中的瓷砖。

1.  2\. 读取目标图像，并将其拆分成一个*M*×*N*的瓷砖网格。

1.  3\. 对于每个瓷砖，找到来自输入图像的最佳匹配。

1.  4\. 通过将选定的输入图像按照*M*×*N*的网格排列，创建最终的马赛克。

### 拆分目标图像

我们将从看看如何将目标图像拆分成*M*×*N*的瓷砖网格开始。按照图 7-2 中展示的方案进行。

![](img/nsp-venkitachalam503045-f07002.jpg)

图 7-2：拆分目标图像

我们将原始图像划分为一个网格，沿 x 轴排列*N*列，沿 y 轴排列*M*行。每个瓷砖由索引（*i*，*j*）表示，宽度为*w*像素，高度为*h*像素。按照这个方案，原始图像的宽度为*w* × *N*像素，高度为*h* × *M*像素。

图 7-2 的右侧展示了如何计算从该网格中提取单个瓷砖的像素坐标。索引为（*i*，*j*）的瓷砖，其左上角坐标为（*i* × *w，*i* × *j*），右下角坐标为（（*i* + 1） × *w*，（*j* + 1） × *h*）。这些坐标可以与 PIL 一起使用，从原始图像中裁剪并创建瓷砖。

### 平均颜色值

图像中的每个像素都有一个颜色，可以通过其红、绿、蓝值以数字的形式表示。在这种情况下，你使用的是 8 位图像，因此这三个颜色分量的每一个都具有 0 到 255 范围内的 8 位值。因此，你可以通过对图像中所有像素的红、绿、蓝值求平均，来确定图像的平均颜色。给定一个总共有*N*个像素的图像，平均 RGB 值的计算方式如下：

![](img/nsp-venkitachalam503045-m07001.jpg)

就像单个像素的 RGB 值一样，整个图像的平均 RGB 值是一个三元组，而不是标量或单个数字，因为每个颜色分量的平均值是分别计算的。你计算平均 RGB 值是为了通过输入图像中的替代品来匹配目标图像中的瓷砖。

### 匹配图像

对于目标图像中的每个瓷砖，你需要从用户指定的输入文件夹中的图像中找到一个匹配图像。为了判断两张图像是否匹配，可以使用平均 RGB 值。最佳匹配是那些平均 RGB 值与目标图像瓷砖的平均 RGB 值最接近的输入图像。

寻找最佳匹配的最简单方法是计算平均 RGB 值之间的距离，就像它们是 3D 空间中的点一样。毕竟，每个平均 RGB 值由三个数字组成，你可以将其视为 x、y 和 z 轴坐标。因此，你可以使用以下几何公式来计算两个 3D 点之间的距离：

![](img/nsp-venkitachalam503045-m07002.jpg)

在这里，你计算点（*r*[1]，*g*[1]，*b*[1]）与（*r*[2]，*g*[2]，*b*[2]）之间的距离。给定目标平均 RGB 值（*r*[1]，*g*[1]，*b*[1]），你可以将来自输入图像的平均 RGB 值列表代入之前的公式中的（*r*[2]，*g*[2]，*b*[2]），以找到最接近的匹配图像。然而，可能需要检查成百上千张输入图像。因此，我们应当考虑如何高效地搜索输入图像集合，以找到最佳匹配。

#### 使用线性搜索

搜索匹配项的最简单方法是*线性搜索*。在这种方法中，你只需逐一遍历所有 RGB 值，找到与查询值距离最小的那个。代码大致如下：

min_dist = MAX_VAL

for val in vals:

dist = distance(query, val)

if dist < MAX_VAL:

min_dist = dist

你逐一遍历列表中的每个值 `vals`，并计算该值与 `query` 之间的距离。如果结果小于 `min_dist`（`min_dist` 初始值为两个点之间的最大可能距离），你就用刚刚计算出的距离更新 `min_dist`。检查完 `vals` 中的每个项后，`min_dist` 将包含整个数据集中最小的距离。

虽然线性搜索方法易于理解和实现，但它并不是很高效。如果 `vals` 列表中有 *N* 个值，那么搜索的时间将与 *N* 成正比。你可以通过使用不同的数据结构和搜索算法来实现更好的性能。

#### 使用 k-d 树

*k-d 树*，或者称为*k 维树*，是一种将*k*维空间划分为多个非重叠子空间的数据结构。该数据结构提供了一种方法来排序和搜索数据集，其中的每个成员是*k*维空间中的一个点。数据集被表示为一个*二叉树*：数据集中的每个点都成为树中的一个节点，每个节点可以有两个子节点。换句话说，树中的每个节点将空间分为两部分，称为*子树*。一部分指向节点的左侧（节点的左子节点及其后代），另一部分指向节点的右侧（节点的右子节点及其后代）。

树的每个节点都与空间的一个维度相关联，这个维度用于判断点是否属于该节点的左子树或右子树。例如，如果一个节点与 x 轴相关联，那么 x 值小于该节点 x 值的点将放入该节点的左子树，x 值大于该节点 x 值的点将放入右子树。选择与每个节点相关联的维度的常见方法是，在向下遍历树的层级时按顺序循环选择这些维度。例如，在三维 k-d 树的情况下，可以将维度设置为 x、y、z、x、y、z 等，依次向下遍历树。处于同一树高的节点将具有相同的分割维度。

让我们来看一个简单的 k-d 树示例。假设你有以下点集，*P*：

*P* = {(5, 3), (2, 4), (1, 2), (6, 6), (7, 2), (4, 6), (2, 8)}

在这种情况下，你将构建一个二维 k-d 树，因为*P*中的每个成员描述的是二维空间中的一个点。你首先将第一个节点，或称*根*节点（5, 3），与 x 维度关联。然后，你将下一个点（2, 4）添加为根节点的左子节点，因为该点的 x 坐标 2 小于根节点的 x 坐标 5。节点（2, 4）位于 k-d 树的第二层，将使用 y 维度进行划分。列表中的下一个点是（1, 2）。从根节点开始，1 < 5，所以你去根节点的左子节点。然后，你使用 y 维度比较（1, 2）和（2, 4）。由于 2 < 4，你将（1, 2）添加为（2, 4）的左子节点。

如果你按这种方式继续处理*P*中的所有点，你将创建图 7-3 中所示的树和空间划分。

![](img/nsp-venkitachalam503045-f07003.jpg)

图 7-3：k-d 树的示例

图 7-3 顶部的图像展示了我们刚才讨论的树的空间划分方案。从点（5, 3）开始，你沿 x 维度将空间划分为两部分，通过该点绘制一条垂直线。接下来，你使用点（2, 4）沿 y 维度将第一个分区的左半部分划分，方法是在该点绘制一条水平线，直到该线与垂直线相交。以这种方式继续处理剩余的点，你将得到图中所示的划分方案。

为什么你应该关心 k-d 树？答案是，一旦你以这种方式排列数据集，你就可以更快地搜索它。具体来说，*最近邻搜索*——找到距离查询点最近的点——在 k-d 树中比在线性搜索中要快得多。对于包含*N*个值的数据集，k-d 树的平均最近邻搜索时间与 log(*N*)成正比，远小于线性搜索所需的与*N*成正比的时间。

为了演示，假设我们要找到*P*中离点*q*（2, 3）最近的点，如图 7-3 所示。从图中可以看到，点（2, 4）是匹配点。最近邻算法将通过从（5, 3）遍历到（2, 4）来找到匹配点。算法知道，例如，可以跳过根节点右子树，因为*q*的 x 坐标小于根节点的 x 坐标。因此，空间划分方案使得你可以跳过比线性搜索更多的比较操作。这就是 k-d 树对我们问题有用的原因。

如何在拼图代码中使用 k-d 树？你可以尝试从头实现一个，但有一个更简单的选项：`scipy`库已经有一个内置的 k-d 树类。我们将在本章稍后讨论如何利用这个类。

## 要求

在这个项目中，你将使用`Pillow`来读取图像，访问其底层数据，并创建和修改图像。你还会使用`numpy`来处理图像数据，使用`scipy`通过 k-d 树搜索图像数据。

## 代码

首先，你将读取输入的图像，然后根据这些图像创建照片拼贴。接下来，你会计算图像的平均 RGB 值，将目标图像拆分成网格，并找到最适合每个网格单元的图像。最后，你会将这些图像拼接起来，形成实际的照片拼贴。要查看完整的项目代码，请跳转到“完整代码”部分，见第 129 页。你也可以在[`github.com/mkvenkit/pp2e/tree/main/photomosaic`](https://github.com/mkvenkit/pp2e/tree/main/photomosaic)找到代码。

### 读取输入图像

首先，从给定文件夹中读取输入图像。以下是实现方法：

def getImages(imageDir):

"""

给定一个图像目录，返回图像列表

"""

❶ files = os.listdir(imageDir)

images = []

对于文件夹中的每个文件：

❷ filePath = os.path.abspath(os.path.join(imageDir, file))

try:

# 显式加载，以免遇到资源紧张问题

❸ fp = open(filePath, "rb")

im = Image.open(fp)

images.append(im)

# 强制加载图像数据

❹ im.load()

# 关闭文件

❺ fp.close()

except:

# 跳过

print("无效图像：%s" % (filePath,))

return images

首先，你使用`os.listdir()`从*imageDir*目录中获取文件名，并将它们存储在名为`files`的列表中❶。接下来，遍历列表中的每个文件并将其加载到 PIL 的`Image`对象中。

你使用`os.path.abspath()`和`os.path.join()`来获取图像的完整文件名❷。这种写法在 Python 中非常常见，可以确保你的代码在使用相对路径（例如，*\foo\bar\*）和绝对路径（*c:\foo\bar\*）时都能正常工作，并且可以跨操作系统使用不同的目录命名约定（Windows 中的\和 Linux 中的/）。

为了将文件加载到 PIL 的`Image`对象中，你可以将每个文件名传递给`Image.open()`方法，但如果你的照片拼贴文件夹中有成百上千张图片，这样做会非常消耗资源。相反，你可以使用 Python 打开每个图像，并将文件句柄`fp`传递给 PIL 的`Image.open()`。加载图像后，关闭文件句柄并释放系统资源。

你使用`open()`❸打开图像文件，然后将文件句柄传递给`Image.open()`，并将得到的图像`im`存储在名为`images`的列表中。调用`Image.load()`❹强制加载图像数据，因为`open()`是一个懒加载操作。它能识别图像，但在你尝试使用图像之前不会真正读取所有图像数据。最后，关闭文件句柄以释放系统资源❺。

### 计算图像的平均颜色值

一旦你读取了输入图像，你需要计算每个图像的平均颜色值。你还需要计算目标图像每个部分的平均颜色值。创建一个函数`getAverageRGB()`来处理这两个任务。

def getAverageRGB(image):

"""

返回每个输入图像的平均颜色值（r, g, b）

"""

# 将每个小块图像作为一个 numpy 数组

❶ im = np.array(image)

# 获取每个输入图像的形状

❷ w,h,d = im.shape

# 获取平均 RGB 值

❸ return tuple(np.average(im.reshape(w*h, d), axis=0))

该函数接受一个`Image`对象——它可以是输入图像之一或目标图像的一个部分——并使用`numpy`将其转换为数据数组❶。得到的`numpy`数组具有形状（`w`，`h`，`d`），其中`w`是图像的宽度，`h`是高度，`d`是深度，对于 RGB 图像，`d`是三个单位（分别对应 R、G 和 B）。你保存了`shape`元组❷，然后通过将数组重塑为形状（`w*h`，`d`）来计算平均 RGB 值，这样你就可以使用`numpy.average()`❸来计算平均值。（你在第六章中执行了类似的操作，以获取灰度图像的平均亮度。）你将结果作为元组返回。

### 将目标图像分割成网格

现在，你需要将目标图像分割成一个 *M*×*N* 的小图像网格。我们来创建一个函数来完成这个任务：

def splitImage(image, size):

"""

给定图像和尺寸（行，列），返回一个 m*n 的图像列表

"""

❶ W, H = image.size[0], image.size[1]

❷ m, n = size

❸ w, h = int(W/n), int(H/m)

# 图像列表

imgs = []

# 生成图像列表

for j in range(m):

for i in range(n):

# 添加裁剪后的图像

❹ imgs.append(image.crop((i*w, j*h, (i+1)*w, (j+1)*h)))

return imgs

首先，你收集目标图像的尺寸❶和网格大小❷。然后，你使用基本的除法计算每个小块的尺寸❸。接下来，你需要遍历网格尺寸，并将每个小块切割出来并存储为单独的图像。调用`image.crop()`❹使用左上角和右下角的图像坐标作为参数来裁剪图像（如“将目标图像分割成网格”一节中所讨论的，见第 115 页）。你最终得到一个图像列表——首先是网格第一行的所有图像，从左到右；然后是网格第二行的所有图像；依此类推。

### 寻找最佳匹配的图像块

现在，我们将尝试从输入图像文件夹中找到最佳匹配的瓦片。我们将使用两种方法：线性搜索和 k-d 树搜索。对于线性搜索方法，你可以创建一个实用函数`getBestMatchIndex()`，如下所示：

def getBestMatchIndex(input_avg, avgs):

"""

返回基于平均 RGB 值距离的最佳图像匹配索引

"""

# 输入图像平均值

avg = input_avg

# 根据 RGB 距离获取与输入图像最接近的 RGB 值

index = 0

❶ min_index = 0

❷ min_dist = float("inf")

❸ 对于 avgs 中的 val:

❹ dist = ((val[0] - avg[0])*(val[0] - avg[0]) +

(val[1] - avg[1])*(val[1] - avg[1]) +

(val[2] - avg[2])*(val[2] - avg[2]))

❺ 如果 dist 小于 min_dist:

min_dist = dist

min_index = index

index += 1

返回 min_index

你正在尝试搜索`avgs`，它是输入图像的平均 RGB 值列表，目的是找到与`input_avg`最接近的值，`input_avg`是目标图像中某个瓦片的平均 RGB 值。首先，你将最接近的匹配索引初始化为 0 ❶，并将最小距离初始化为无穷大 ❷。然后，你遍历平均值列表中的值 ❸，并开始计算距离 ❹，使用“匹配图像”中第 116 页中显示的标准公式。（你跳过了计算平方根，以减少计算时间。）如果计算的距离小于存储的最小距离`min_dist`，则用新的最小距离替换它 ❺。由于任何距离都会小于无穷大，因此第一次迭代时这个测试总是会通过。迭代结束时，`min_index`是`avgs`列表中最接近`input_avg`的平均 RGB 值的索引。现在，你可以使用此索引从输入图像列表中选择匹配的图像。

现在，我们将使用 k-d 树而不是线性搜索来找到最佳匹配。以下是该函数：

def getBestMatchIndicesKDT(qavgs, kdtree):

"""

返回基于 RGB 值距离的最佳图像匹配索引

使用 k-d 树

"""

# 例如，[array([2.]), array([9], dtype=int64)]

❶ res = list(kdtree.query(qavgs, k=1))

❷ min_indices = res[1]

return min_indices

`getBestMatchIndicesKDT()` 函数接受两个参数：`qavgs` 是目标图像中每个瓷砖的平均 RGB 值列表，`kdtree` 是通过输入图像的平均 RGB 值列表创建的 `scipy KDTree` 对象。（我们将在 “创建图像马赛克” 中的 第 124 页创建 `KDTree` 对象。）您使用 `KDTree` 对象的 `query()` 方法来获取树中最接近 `qavgs` 中点的那些点❶。在这里，`k` 参数是您希望返回的与查询点最接近的邻居数。因为您只需要最接近的匹配项，所以将 `k=1` 传入。`query()` 方法的返回值是一个元组，由两个 `numpy` 数组组成，分别表示匹配的距离和索引。您需要的是索引，因此选择结果中的第二个值❷。

请注意，`query()` 方法❶允许您传入一个查询点列表，而不仅仅是一个。这实际上比逐个查询结果更快，并且意味着您只需调用一次 `getBestMatchIndicesKDT()` 函数，而与此相反，您将不得不多次调用线性搜索的 `getBestMatch()` 函数，每次针对每个瓷砖图像进行调用。

完整的程序将包括一个选项，允许您选择使用前两种函数中的哪一种，线性搜索版本或 k-d 树版本。它还将包含一个计时器，用于测试哪种搜索方法更快。

### 创建图像网格

在继续创建图像马赛克之前，您需要一个额外的工具函数。`createImageGrid()` 函数将创建一个尺寸为 *M*×*N* 的图像网格。这个图像网格就是最终的图像马赛克，由选择的输入图像列表生成。

def createImageGrid(images, dims):

"""

给定一个图像列表和一个网格尺寸 (m, n)，创建一个图像网格

"""

❶ m, n = dims

# 校验

assert m*n == len(images)

# 获取图像的最大高度和宽度

# 不要假设它们都相同

❷ width = max([img.size[0] for img in images])

height = max([img.size[1] for img in images])

# 创建目标图像

❸ grid_img = Image.new('RGB', (n*width, m*height))

# 将瓷砖图像粘贴到图像网格中

for index in range(len(images)):

❹ row = int(index/n)

❺ col = index - n*row

❻ grid_img.paste(images[index], (col*width, row*height))

return grid_img

该函数接受两个参数：一个图像列表（你根据与目标图像各个拼图块的 RGB 匹配程度选择的输入图像）和一个包含照片拼贴画维度的元组（你希望拼贴画的行数和列数）。你首先获取网格的维度 ❶，然后使用 `assert` 方法检查传给 `createImageGrid()` 的图像数量是否与网格大小匹配。（`assert` 方法用于检查代码中的假设，尤其是在开发和测试过程中。）接着，你计算所选图像的最大宽度和高度 ❷，因为这些图像的尺寸可能不完全相同。你将使用这些最大尺寸来设置照片拼贴画的标准拼图块大小。如果某个输入图像无法完全填充一个拼图块，默认情况下，拼图块之间的空白部分将显示为纯黑色。

接下来，你创建一个空的 `Image`，其大小足以容纳网格中的所有图像 ❸；你将把拼图块图像粘贴到这里。然后，通过遍历所选图像并使用 `Image.paste()` 方法将它们粘贴到网格的适当位置，你填充图像网格 ❻。`Image.paste()` 的第一个参数是要粘贴的 `Image` 对象，第二个参数是左上角的坐标。现在，你需要确定将输入图像粘贴到图像网格中的哪一行和哪一列。为此，你可以将图像索引转换为行列坐标。图像网格中一个拼图块的索引由 *N* × *row* + *col* 给出，其中 *N* 是宽度方向上的单元格数，(*row*, *col*) 是网格中的坐标；在 ❹ 处，你通过前面的公式计算行，在 ❺ 处，计算列。

### 创建照片拼贴画

现在你已经拥有了所有必需的工具，让我们编写创建照片拼贴画的主函数。以下是函数的开始部分：

def createPhotomosaic(target_image, input_images, grid_size,

reuse_images, use_kdt):

"""

创建给定目标图像和输入图像的照片拼贴画

"""

print('正在分割输入图像...')

# 分割目标图像

❶ target_images = splitImage(target_image, grid_size)

print('正在寻找图像匹配...')

# 对于每个目标图像，从输入图像中选择一张

output_images = []

# 用户反馈

count = 0

❷ batch_size = int(len(target_images)/10)

# 计算输入图像的平均值

avgs = []

❸ 对于 input_images 中的每张图像：

avgs.append(getAverageRGB(img))

# 计算目标图像的平均值

avgs_target = []

❹ 对于 target_images 中的每张图像：

# 目标子图像平均值

avgs_target.append(getAverageRGB(img))

`createPhotomosaic()`函数以目标图像、输入图像列表、生成的光栅图像的大小（行和列的数量）以及指示是否可以重用图像和是否使用 k-d 树来查找图像匹配的标志作为输入。该函数首先调用`splitImage()`❶将目标图像拆分成一个小的图像网格。一旦图像被拆分，你就可以开始为每个瓦片从输入文件夹中的图像中查找匹配项。然而，由于这个过程可能会比较耗时，因此最好向用户提供反馈，让他们知道程序仍在运行。为了提供这种反馈，你将`batch_size`设置为瓦片图像总数的十分之一❷。选择十分之一是任意的，只是程序的一种方式来表示“我还活着”。每当程序处理完十分之一的图像时，它会打印一条消息，指示程序仍在运行。

要找到图像匹配项，你需要平均 RGB 值。你遍历输入图像❸并使用`getAverageRGB()`函数计算每个图像的平均 RGB 值，将结果存储在`avgs`列表中。然后，你对目标图像中的每个瓦片做同样的操作❹，将平均 RGB 值存储到`avgs_target`列表中。

函数继续使用`if...else`语句来使用 k-d 树或线性搜索来查找 RGB 匹配项。让我们先看一下`if`分支，当`use_kdt`标志被设置为`True`时它会执行：

# 使用 k-d 树进行平均值匹配？

如果使用 kdt:

# 创建 k-d 树

❶ kdtree = KDTree(avgs)

# 查询 k-d 树

❷ match_indices = getBestMatchIndicesKDT(avgs_target, kdtree)

# 处理匹配项

❸ 对于 match_index 在 match_indices 中:

❹ output_images.append(input_images[match_index])

你使用来自输入图像的平均 RGB 值列表❶来创建一个`KDTree`对象，并通过将`avgs_target`和`KDTree`对象传递给`getBestMatchIndicesKDT()`辅助函数❷来获取最佳匹配的索引。然后，你遍历所有匹配的索引❸，找到相应的输入图像，并将它们附加到`output_images`列表❹。

接下来，让我们看看`else`分支，它执行线性搜索来查找匹配项：

否则:

# 使用线性搜索

❶ 对于 avgs_target 中的 avg:

# 查找匹配索引

❷ match_index = getBestMatchIndex(avg, avgs)

❸ output_images.append(input_images[match_index])

# 用户反馈

❹ 如果 count > 0 且 batch_size > 10 且 count % batch_size == 0:

print('已处理 %d / %d ...' % (count, len(target_images)))

count += 1

# 如果设置了标志，从输入中移除所选图像

❺ 如果不是 reuse_images:

input_images.remove(match)

对于线性搜索，你首先开始遍历目标图像的平均 RGB 值 ❶。对于每个瓦片，你使用`getBestMatchIndex()`函数在输入图像的平均值列表中查找最接近的匹配 ❷。结果返回为一个索引，你可以使用它来获取`Image`对象并将其存储在`output_images`列表中 ❸。每处理`batch_size`数量的图像 ❹，你会向用户输出一条信息。如果`reuse_images`标志设置为`False` ❺，你会将选定的输入图像从列表中移除，以便它不会在另一个瓦片中重复使用。（当你有大量输入图像可供选择时，这种方法最为有效。）

在`createPhotomosaic()`函数中剩下的任务是将输入图像安排到最终的图像拼接中：

print('正在创建图像拼接...')

# 将图像拼接绘制到输出图像

❶ mosaic_image = createImageGrid(output_images, grid_size)

# 返回拼接图像

return mosaic_image

你使用`createImageGrid()`函数构建图像拼接 ❶。然后你将生成的图像返回为`mosaic_image`。

### 编写 main() 函数

程序的`main()`函数接受并解析命令行参数，加载所有图像，并进行一些额外的设置。然后它调用`createPhotomosaic()`函数并保存生成的图像拼接。在构建图像拼接的过程中，Python 会记录该过程所需的时间，从而让你比较 k-d 树和线性搜索的性能。

#### 添加命令行选项

`main()`函数支持以下命令行选项：

# 解析参数

parser = argparse.ArgumentParser(description='根据目标图像创建图像拼接')

输入图像')

# 添加参数

parser.add_argument('--target-image', dest='target_image', required=True)

parser.add_argument('--input-folder', dest='input_folder', required=True)

parser.add_argument('--grid-size', nargs=2, dest='grid_size',

required=True)

parser.add_argument('--output-file', dest='outfile', required=False)

parser.add_argument('--kdt', action='store_true', required=False)

该代码包含三个必需的命令行参数：目标图像的名称、输入图像文件夹的名称和网格大小。第四个参数是可选的输出文件名。如果省略文件名，图像拼接将被写入名为*mosaic.png*的文件中。第五个参数是一个布尔值标志，它启用 k-d 树搜索，而不是线性搜索来匹配平均 RGB 值。

#### 控制图像拼接的大小

一旦所有图像加载完成，在 `main()` 函数中需要解决的一个问题是最终拼图的大小（以像素为单位）。如果你只是盲目地将输入图像拼接在一起，根据目标图像中匹配的拼图块，你可能会得到一个比目标图像大得多的拼图。为避免这种情况，应调整输入图像的大小，使其与网格中每个拼图块的大小匹配。（这还有一个附加好处：因为你将使用更小的图像，所以平均 RGB 计算速度会更快。）下面是处理此任务的 `main()` 函数部分：

print('正在调整图像大小...')

# 对于给定的网格大小，计算最大宽度和高度的拼图块

❶ dims = (int(target_image.size[0]/grid_size[1]),

int(target_image.size[1]/grid_size[0]))

print("最大拼图尺寸: %s" % (dims,))

# 调整大小

对于 img in input_images:

❷ img.thumbnail(dims)

你根据网格中指定的行数和列数计算目标尺寸 ❶；然后，你使用 PIL `Image.``thumbnail()` 方法调整输入图像的大小，以适应这些尺寸 ❷。

#### 性能计时

当程序运行时，你希望知道它执行了多长时间。可以使用 Python `timeit` 模块来实现这一目的。寻找执行时间的方法在这里概述：

导入 timeit

# 开始计时

❶ start = timeit.default_timer()

# 在此运行一些代码...

--`snip`--

# 停止计时

❷ stop = timeit.default_timer()

print('执行时间: %f 秒' % (stop - start, ))

你通过 `timeit` 模块的默认计时器 ❶ 记录开始时间。然后，在运行一些代码后，你记录结束时间 ❷。计算差值即可得到以秒为单位的执行时间。

## 运行 Photomosaic 生成器

首先，使用默认的线性搜索方法运行程序。这个拼图将由 128×128 的图像网格组成：

$ `python photomosaic.py --target-image test-data/cherai.jpg --input-folder`

`  test-data/set6/ --grid-size 128 128`

正在读取输入文件夹...

开始创建拼图...

正在调整图像大小...

最大拼图尺寸: (23, 15)

分割输入图像...

查找图像匹配...

已处理 1638 张，共 16384 张...

已处理 3276 张，共 16384 张...

已处理 4914 张，共 16384 张...

已处理 6552 张，共 16384 张...

已处理 8190 张，共 16384 张...

已处理 9828 张，共 16384 张...

已处理 11466 张，共 16384 张...

已处理 13104 张，共 16384 张...

已处理 14742 张，共 16384 张...

已处理 16380 张，共 16384 张...

创建拼图...

输出已保存至 mosaic.png

完成。

执行时间:    设置: 0.402047 秒

❶ 执行时间: 创建: 2.123931 秒

执行时间:    总计: 2.525978 秒

图 7-4(a) 显示了目标图像， 图 7-4(b) 显示了生成的照片拼贴。你可以在 图 7-4(c) 中看到照片拼贴的特写。从输出中可以看出，使用线性搜索为拼贴中的每个 16,384 个瓦片找到最佳匹配大约需要 2.1 秒 ❶。这个速度不错，但我们可以做得更好。

![](img/nsp-venkitachalam503045-f07004.jpg)

图 7-4：照片拼贴生成器的示例运行

现在，使用 `--kdt` 选项运行相同的程序，这将启用使用 k-d 树来搜索图像匹配。以下是结果：

$ `python photomosaic.py --target-image test-data/cherai.jpg --input-folder`

`  test-data/set6/ --grid-size 128 128 --kdt`

正在读取输入文件夹...

正在开始照片拼贴创建...

正在调整图像大小...

最大瓦片尺寸： (23, 15)

正在拆分输入图像...

正在寻找图像匹配...

创建拼贴...

已将输出保存为 mosaic.png

完成。

执行时间：设置：0.410334 秒

❶ 执行时间：创建：1.089237 秒

执行时间：总计：1.499571 秒

使用 k-d 树后，照片拼贴的创建时间从大约 2.1 秒减少到不到 1.1 秒 ❶。这几乎是速度的两倍！

## 摘要

在这个项目中，你学习了如何根据目标图像和一组输入图像创建照片拼贴。当从远处查看时，照片拼贴看起来像原始图像，但近距离观察时，你可以看到构成拼贴的各个单独图像。你还学习了一种有趣的数据结构——k-d 树，它显著加速了为拼贴中的每个瓦片寻找最匹配图像的过程。

## 实验！

这里有一些进一步探索照片拼贴的方法：

1.  1\. 编写一个程序，创建任何图像的块状版本，类似于 图 7-1。

1.  2\. 在本章的代码中，你通过将匹配的图像无缝拼接来创建照片拼贴。更具艺术性的表现可能会在每个瓦片图像周围加入均匀的间隙。你如何创建这个间隙？（提示：在计算最终图像尺寸时考虑间隙，并在 `createImageGrid()` 中粘贴图像时考虑间隙。）

## 完整代码

这是项目的完整代码：

"""

photomosaic.py

给定目标图像和输入图像文件夹，创建一个照片拼贴。

作者：Mahesh Venkitachalam

"""

导入 os, random, argparse

从 PIL 导入图像

导入 numpy 作为 np

来自 scipy.spatial 导入 KDTree

导入 timeit

def getAverageRGBOld(image):

"""

给定 PIL 图像，返回颜色的平均值 (r, g, b)

"""

# 图像中的像素数量

npixels = image.size[0]*image.size[1]

# 获取颜色列表 [(cnt1, (r1, g1, b1)), ...]

cols = image.getcolors(npixels)

# 获取 [(c1*r1, c1*g1, c1*g2), ...]

sumRGB = [(x[0]*x[1][0], x[0]*x[1][1], x[0]*x[1][2]) for x in cols]

# 计算 (sum(ci*ri)/np, sum(ci*gi)/np, sum(ci*bi)/np)

# zip 给我们 [(c1*r1, c2*r2, ...), (c1*g1, c1*g2, ...), ...]

avg = tuple([int(sum(x)/npixels) for x in zip(*sumRGB)])

return avg

def getAverageRGB(image):

"""

给定 PIL 图像，返回颜色的平均值（r, g, b）

"""

# 获取图像作为 numpy 数组

im = np.array(image)

# 获取形状

w,h,d = im.shape

# 获取平均值

return tuple(np.average(im.reshape(w*h, d), axis=0))

def splitImage(image, size):

"""

给定图像和维度（行，列），返回一个 m*n 的图像列表

"""

W, H = image.size[0], image.size[1]

m, n = size

w, h = int(W/n), int(H/m)

# 图像列表

imgs = []

# 生成图像列表

for j in range(m):

for i in range(n):

# 添加裁剪后的图像

imgs.append(image.crop((i*w, j*h, (i+1)*w, (j+1)*h)))

return imgs

def getImages(imageDir):

"""

给定一个图像目录，返回一个图像列表

"""

files = os.listdir(imageDir)

images = []

for file in files:

filePath = os.path.abspath(os.path.join(imageDir, file))

try:

# 显式加载以避免资源紧张

fp = open(filePath, "rb")

im = Image.open(fp)

images.append(im)

# 强制从文件加载图像数据

im.load()

# 关闭文件

fp.close()

except:

# 跳过

print("无效图像: %s" % (filePath,))

return images

def getBestMatchIndex(input_avg, avgs):

"""

返回基于 RGB 值距离的最佳图像匹配索引

"""

# 输入图像的平均值

avg = input_avg

# 获取与输入最接近的 RGB 值，基于 x/y/z 距离

index = 0

min_index = 0

min_dist = float("inf")

for val in avgs:

dist = ((val[0] - avg[0])*(val[0] - avg[0]) +

(val[1] - avg[1])*(val[1] - avg[1]) +

(val[2] - avg[2])*(val[2] - avg[2]))

if dist < min_dist:

min_dist = dist

min_index = index

index += 1

return min_index

def getBestMatchIndicesKDT(qavgs, kdtree):

"""

返回基于 RGB 值距离的最佳图像匹配的索引

使用 k-d 树

"""

# 例如， [array([2.]), array([9], dtype=int64)]

res = list(kdtree.query(qavgs, k=1))

min_indices = res[1]

return min_indices

def createImageGrid(images, dims):

"""

给定一组图像和网格大小（m, n），创建

一个图像网格

"""

m, n = dims

# 合理性检查

assert m*n == len(images)

# 获取图像的最大高度和宽度

# 即，不假设它们都是相等的

width = max([img.size[0] for img in images])

height = max([img.size[1] for img in images])

# 创建输出图像

grid_img = Image.new('RGB', (n*width, m*height))

# 粘贴图像

for index in range(len(images)):

row = int(index/n)

col = index - n*row

grid_img.paste(images[index], (col*width, row*height))

return grid_img

def createPhotomosaic(target_image, input_images, grid_size,

reuse_images, use_kdt):

"""

创建给定目标和输入图像的照片马赛克

"""

print('正在拆分输入图像...')

# 拆分目标图像

target_images = splitImage(target_image, grid_size)

print('正在寻找图像匹配...')

# 对于每个目标图像，从输入中选择一个

output_images = []

# 用户反馈

count = 0

batch_size = int(len(target_images)/10)

# 计算输入图像的平均值

avgs = []

for img in input_images:

avgs.append(getAverageRGB(img))

# 计算目标图像的平均值

avgs_target = []

for img in target_images:

# 目标子图像的平均值

avgs_target.append(getAverageRGB(img))

# 使用 k-d 树进行平均匹配？

if use_kdt:

# 创建 k-d 树

kdtree = KDTree(avgs)

# 查询 k-d 树

match_indices = getBestMatchIndicesKDT(avgs_target, kdtree)

# 处理匹配项

for match_index in match_indices:

output_images.append(input_images[match_index])

else:

# 使用线性搜索

for avg in avgs_target:

# 查找匹配索引

match_index = getBestMatchIndex(avg, avgs)

output_images.append(input_images[match_index])

# 用户反馈

if count > 0 and batch_size > 10 and count % batch_size == 0:

print('已处理 {} 张，共 {} 张...'.format(count,

len(target_images)))

count += 1

# 如果设置了标志，移除输入中的已选图像

if not reuse_images:

input_images.remove(match)

print('正在创建拼贴图...')

# 将拼贴图绘制到图像中

mosaic_image = createImageGrid(output_images, grid_size)

# 返回拼贴图

return mosaic_image

# 将我们的代码收集到 main() 函数中

def main():

# 命令行参数在 sys.argv[1], sys.argv[2] 等中...

# sys.argv[0] 是脚本本身，可以忽略

# 解析参数

parser = argparse.ArgumentParser(description='创建拼贴图')

from input images')

# 添加参数

parser.add_argument('--target-image', dest='target_image', required=True)

parser.add_argument('--input-folder', dest='input_folder', required=True)

parser.add_argument('--grid-size', nargs=2, dest='grid_size',

required=True)

parser.add_argument('--output-file', dest='outfile', required=False)

parser.add_argument('--kdt', action='store_true', required=False)

args = parser.parse_args()

# 开始计时

start = timeit.default_timer()

###### 输入部分 ######

# 目标图像

target_image = Image.open(args.target_image)

# 输入图像

print('正在读取输入文件夹...')

input_images = getImages(args.input_folder)

# 检查是否找到任何有效的输入图像

if input_images == []:

print('在 %s 中未找到输入图像。退出程序。' % (args.input_folder,))

exit()

# 打乱列表——以获得更丰富的输出？

random.shuffle(input_images)

# 网格大小

grid_size = (int(args.grid_size[0]), int(args.grid_size[1]))

# 输出

output_filename = 'mosaic.png'

if args.outfile:

output_filename = args.outfile

# 重新使用输入中的任何图像

reuse_images = True

# 是否将输入图像大小调整为适应原始图像大小？

resize_input = True

# 使用 k-d 树进行匹配

use_kdt = False

if args.kdt:

use_kdt = True

##### 输入部分结束 #####

print('开始创建拼贴图...')

# 如果图像不能重用，确保 m*n <= 图像数量

if not reuse_images:

if grid_size[0]*grid_size[1] > len(input_images):

print('网格大小小于图像数量')

exit()

# 调整输入图像大小

if resize_input:

print('正在调整图像大小...')

# 对于给定的网格大小，计算瓦片的最大尺寸 w,h

dims = (int(target_image.size[0]/grid_size[1]),

int(target_image.size[1]/grid_size[0]))

print("最大瓷砖尺寸：%s" % (dims,))

# 调整大小

for img in input_images:

img.thumbnail(dims)

# 设置时间

t1 = timeit.default_timer()

# 创建照片拼贴

mosaic_image = createPhotomosaic(target_image, input_images, grid_size,

reuse_images, use_kdt)

# 写出拼贴

mosaic_image.save(output_filename, 'PNG')

print("已保存输出至 %s" % (output_filename,))

print('完成。')

# 创建时间

t2 = timeit.default_timer()

print('执行时间：设置：%f 秒' % (t1 - start, ))

print('执行时间：创建：%f 秒' % (t2 - t1, ))

print('执行时间：总计：%f 秒' % (t2 - start, ))

# 标准样板代码，调用 main() 函数开始

# 程序。

if __name__ == '__main__':

main()
