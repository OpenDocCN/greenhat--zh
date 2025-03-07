# [15](nsp-venkitachalam503045-0008.xhtml#rch15)

# 树莓派上的音频机器学习（Audio ML）

![](images/nsp-venkitachalam503045-circle-image.jpg)

在过去的十年里，*机器学习（ML）* 已经风靡全球。从面部识别到预测文本，再到自动驾驶汽车，机器学习无处不在，我们几乎每天都能听到关于机器学习的新应用。在这一章节中，你将使用Python和TensorFlow开发一个基于机器学习的语音识别系统，该系统将在一台廉价的树莓派计算机上运行。

语音识别系统已经在大量设备和家电中部署，形式多为语音助手，如Alexa、Google和Siri。这些系统可以执行从设置提醒到在办公室远程打开家中灯光等任务。但是所有这些平台都需要你的设备连接到互联网，并且你需要注册它们的服务。这引发了隐私、安全性和功耗等问题。你的灯泡*真的*需要连接到互联网才能响应语音命令吗？答案是*不*。通过这个项目，你将学到如何设计一个在低功耗设备上运行的语音识别系统，而且该设备无需连接互联网。

通过这个项目，你将学习到一些概念，包括：

+   • 使用机器学习工作流解决问题

+   • 使用TensorFlow和Google Colab创建机器学习模型

+   • 精简机器学习模型以便在树莓派上使用

+   • 使用短时傅里叶变换（STFT）处理音频并生成频谱图

+   • 利用多进程并行执行任务

## [机器学习概述](nsp-venkitachalam503045-0008.xhtml#rah1701)

在一本书的单独一章中，谈论如此广泛的机器学习主题几乎是不可能做到的。因此，我们的做法是将机器学习视为解决问题的另一种工具——在这个案例中，是如何区分不同的口语单词。实际上，像TensorFlow这样的机器学习框架如今已经成熟且易于使用，以至于即使你不是该领域的专家，也能有效地将机器学习应用于实际问题。因此，在这一部分，我们将简要介绍与项目相关的机器学习术语。

ML 是 *人工智能（AI）* 这一计算机科学学科中的一小部分，尽管当大众媒体提到 AI 时，通常指的就是 ML。ML 本身由多个子学科组成，这些子学科涉及不同的方法和算法。在本项目中，你将使用 ML 的一个子集，称为 *深度学习*，它利用 *深度神经网络（**DNNs**）* 来识别大量数据中的特征和模式。DNN 的起源可以追溯到 *人工神经网络（ANNs）*，它们大致模仿了我们大脑中的神经元。ANNs 由一组具有多个输入的 *节点* 组成。每个节点都有一个与之关联的 *权重*。ANN 的输出通常是输入和权重的非线性函数。这个输出可以连接到另一个 ANN 的输入。当网络有多层 ANNs 时，它就变成了深度神经网络。通常，网络的层数越多——也就是说，网络越深——学习模型的准确性也越高。

对于本项目，你将使用 *监督学习* 过程，通常可以分为两个阶段。第一个阶段是 *训练阶段*，你会向模型展示几个输入及其期望的输出。例如，如果你正在构建一个人类存在检测系统来识别视频帧中是否有人，你会在训练阶段展示这两种情况的示例（有人的与没有人的），并为每个示例正确标注标签。接下来是 *推理阶段*，你将展示新的输入，模型基于在训练过程中学到的知识，对这些输入做出预测。继续前面的例子，你会向人类存在检测系统展示新的视频帧，模型将预测每一帧中是否有人。（还有 *无监督学习* 过程，在这种过程中，ML 系统会根据未标注的数据自行寻找模式。）

一个 ML 模型有许多数值 *参数*，这些参数帮助它处理数据。在训练过程中，这些参数会自动调整，以最小化期望值与模型预测值之间的误差。通常，使用一种称为 *梯度下降* 的算法来最小化误差。除了在训练过程中调整的 ML 模型参数外，还有 *超参数*，这些是针对整个模型调整的变量，例如选择使用哪种神经网络架构或训练批次的大小。[图 15-1](nsp-venkitachalam503045-0030.xhtml#fig15-1) 展示了我为本项目选择的神经网络架构。

![](images/nsp-venkitachalam503045-f15001.jpg)

图 15-1：语音识别项目的神经网络架构

网络架构中的每一层都代表了对数据的某种处理方式，旨在帮助提高模型的准确性。网络的设计并非简单，但仅仅定义每一层并不能告诉我们它是如何工作的。一个更广泛的问题是 *为什么* 我选择了这个特定的网络。答案是：需要通过实验来确定适合当前项目的最佳网络架构。尝试不同的神经网络架构并观察哪个架构在训练后能产生最准确的结果是很常见的做法。此外，还有许多由机器学习研究人员发布的架构被证明效果很好，这是实际应用中的一个良好起点。

注：如果你想了解更多关于机器学习的信息，我强烈推荐安德鲁·格拉斯纳（Andrew Glassner）编写的《深度学习：视觉方法》（Deep Learning: A Visual Approach，No Starch Press，2021年）一书。这本书能帮助你对该领域建立直观的理解，而无需深入数学或代码部分。若你希望有一个全面的动手实践方法，我还推荐由吴恩达（Andrew Ng）教授主讲的 Coursera 在线机器学习课程。

## [工作原理](nsp-venkitachalam503045-0008.xhtml#rah1702)

在这个项目中，你将使用谷歌的 TensorFlow 机器学习框架，利用包含语音命令的音频文件集合来训练神经网络。然后，你将把训练好的模型的优化版本加载到配备麦克风的 Raspberry Pi 上，这样当你说出命令时，Pi 就能够识别它们。[图 15-2](nsp-venkitachalam503045-0030.xhtml#fig15-2) 展示了该项目的框图。

![](images/nsp-venkitachalam503045-f15002.jpg)

图 15-2：语音识别项目的框图

对于项目的训练部分，你将使用 Google Colab（即协作平台）进行工作，它是一个免费的基于云的服务，让你可以在网页浏览器中编写和运行 Python 程序。使用 Colab 有两个优势。首先，你无需在本地计算机上安装 TensorFlow，也无需处理与不同版本 TensorFlow 相关的不兼容问题。其次，Colab 运行在比你计算机更强大的机器上，因此训练过程会更快。对于训练数据，你将使用来自 Google 的 Mini Speech Commands 数据集，这是一个较大的 Speech Commands 数据集的子集，后者于 2018 年发布。该数据集包含了成千上万的词汇录音样本，词汇包括 *yes*（是），*no*（不），*up*（上），*down*（下），*left*（左），*right*（右），*stop*（停止）和 *go*（开始），所有录音都标准化为 16 位 WAV 文件，采样率为 16,000 Hz。你将生成每个录音的 *频谱图*，这是一种图像，展示了音频的频率内容随时间的变化，并使用这些频谱图通过 TensorFlow 训练深度神经网络（DNN）。

注意：本项目的训练部分灵感来自谷歌官方的 TensorFlow 示例“简单音频识别”。你将使用与该示例相同的神经网络架构。然而，项目的其余部分与谷歌的示例有很大不同，因为我们的目标是在 Raspberry Pi 上识别实时音频，而后者则是对现有的 WAV 文件进行推理。

一旦训练完成，你将把训练好的模型转换为一种简化格式，称为 TensorFlow Lite，它专为在嵌入式系统等性能较低的硬件上运行而设计，并将该精简版模型加载到 Raspberry Pi 上。在那里，你将运行 Python 代码，持续监控来自 USB 麦克风的音频输入，获取音频的频谱图，并对这些数据进行推理，以识别训练集中的语音命令。你将把模型识别出的命令打印到串口监视器上。

### [频谱图](nsp-venkitachalam503045-0008.xhtml#rbh1701)

本项目的关键步骤之一是生成音频数据的频谱图——包括用于训练模型的预先存在的数据和在推理过程中遇到的实时数据。在[第 4 章](nsp-venkitachalam503045-0016.xhtml#ch04)中，你已经了解了频谱图如何揭示音频样本在特定时刻的频率。然后，在[第 13 章](nsp-venkitachalam503045-0028.xhtml#ch13)中，你学习了如何使用一种数学工具——*离散傅里叶变换（DFT）*——来计算频谱图。频谱图本质上就是一系列频谱图，它们通过一系列傅里叶变换器生成，展示了某些音频数据的频率内容如何随时间变化。

你需要每个音频样本的频谱图，而不是单一的频谱图，因为人类语音的声音非常复杂。即使是一个单词，随着这个单词的发音，音频中的频率变化也会非常显著，并且有其独特的变化方式。在这个项目中，你将处理每个一秒钟长的音频片段，每个片段包含 16,000 个样本。如果你对整个片段一次性计算一个 DFT，你将无法准确看到频率如何在片段中变化，因此无法可靠地识别出所说的单词。相反，你将把片段分成多个重叠的间隔，并计算每个间隔的 DFT，这样就能获得所需的频谱图序列。[图 15-3](nsp-venkitachalam503045-0030.xhtml#fig15-3)展示了这种计算方式，称为*短时傅里叶变换（**STFT）*。

![](images/nsp-venkitachalam503045-f15003.jpg)

图 15-3：计算信号的频谱图

STFT 给你提供了音频的 *M* 个 DFT（离散傅里叶变换），它们是在均匀的时间间隔内采集的。时间在频谱图的 x 轴上显示。每个 DFT 给你 *N* 个频率 bin 和每个 bin 中声音的强度。这些频率 bin 映射到频谱图的 y 轴上。因此，频谱图呈现为一个 *M*×*N* 的图像。图像中的每一列像素表示一个 DFT，其中颜色用于表示在给定频率带中的信号强度。

你可能会想知道，为什么我们在这个项目中需要使用傅里叶变换？为什么不直接使用音频片段的波形，而是从这些波形中提取频率信息呢？为了回答这个问题，看看[图 15-4](nsp-venkitachalam503045-0030.xhtml#fig15-4)。

![](images/nsp-venkitachalam503045-f15004_annotated.jpg)

图 15-4：语音样本的波形和频谱图

图的上半部分显示的是通过说出“左，右，左，右”这一顺序录制的波形。图的下半部分显示的是该录音的频谱图。仅从波形来看，你可以看到两个*左*的波形有些相似，两个*右*的波形也类似，但很难从每个单词的波形中找出明显的识别特征。相比之下，频谱图揭示了每个单词的更多视觉特征，比如在每个*右*的实例中都能看到明亮的 C 形曲线（由箭头标出）。我们可以用肉眼更清楚地看到这些独特的特征，神经网络也能“看到”它们。

最终，由于频谱图本质上是一张图像，将数据转化为频谱图将语音识别问题转化为图像分类问题，从而使我们能够利用现有的丰富机器学习技术来进行图像分类。（当然，波形也可以作为图像处理，但正如你所看到的，频谱图更擅长捕捉音频数据的“特征”，因此更适合机器学习应用。）

### [树莓派上的推断](nsp-venkitachalam503045-0008.xhtml#rbh1702)

树莓派上的代码必须完成几个任务：它需要从连接的麦克风读取音频输入，计算该音频的频谱图，并使用训练好的 TensorFlow Lite 模型进行推断。以下是一个可能的操作顺序：

1.  1\. 读取麦克风数据一秒钟。

1.  2\. 处理数据。

1.  3\. 执行推断。

1.  4\. 重复。

然而，这种方法存在一个大问题。当你忙于执行步骤 2 和 3 时，可能会有更多的语音数据进入，这样你就会错过它们。解决方案是使用 Python 多进程并发执行不同的任务。你的主进程将只收集音频数据并将其放入队列中。在一个单独的、同时进行的进程中，你将从队列中取出这些数据并对其进行推断处理。下面是新的方案：

主进程

1.  1\. 读取麦克风数据一秒钟。

1.  2\. 将数据放入队列中。

推断过程

1.  1\. 检查队列中是否有数据。

1.  2\. 对数据进行推理处理。

现在，主进程不会错过任何音频输入，因为将数据放入队列是一个非常快速的操作。但还有另一个问题。你正在从麦克风连续收集一秒钟的音频样本并进行处理，但你不能假设所有的语音命令都会完美地适应这些一秒钟的间隔。命令可能会出现在边缘，并被拆分到两个连续的样本中，在这种情况下，可能无法在推理时识别该命令。一个更好的方法是创建重叠样本，如下所示：

主进程

1.  1\. 对于第一个帧，收集一个两秒钟的样本。

1.  2\. 将两秒钟的样本放入队列。

1.  3\. 收集另一个一秒钟的样本。

1.  4\. 通过将第2步样本的后半部分移到前面，并用第3步的样本替换后半部分，创建一个两秒钟的样本。

1.  5\. 返回第2步。

推理过程

1.  1\. 检查队列中是否有数据。

1.  2\. 基于峰值幅度，对两秒钟数据的其中一秒部分进行推理。

1.  3\. 返回第1步。

在这个新方案中，放入队列中的每个样本长度为两秒钟，但相邻样本之间有一秒钟的重叠，如[图15-5](nsp-venkitachalam503045-0030.xhtml#fig15-5)所示。这样，即使一个单词在一个样本中部分被截断，你也能在下一个样本中得到完整的单词。你仍然只对一秒钟的片段进行推理，且会聚焦于两秒样本中幅度值最高的部分。这是最可能包含已说单词的部分。你需要这些片段为一秒钟，以确保与训练数据的一致性。

![](images/nsp-venkitachalam503045-f15005.jpg)

图15-5：两帧重叠方案

通过这种多进程和重叠样本的组合，你将设计一个语音识别系统，最大程度减少漏掉的输入，并提高推理结果。

## [要求](nsp-venkitachalam503045-0008.xhtml#rah1703)

对于这个项目，你需要注册Google Colab来训练你的机器学习模型。在树莓派上，你将需要以下Python模块：

+   • `tflite_runtime` 用于运行TensorFlow推理

+   • `scipy` 用于计算音频波形的STFT

+   • `numpy` 数组用于处理音频数据

+   • `pyaudio` 用于从麦克风输入流式传输音频数据

这些模块的安装方法可以参考[附录B](nsp-venkitachalam503045-0032.xhtml#appb)。你还将使用Python内置的`multiprocessing`模块，在与音频处理线程分开的线程中运行机器学习推理。

在硬件部分，你将需要以下设备：

+   • 一台树莓派3B+或更新型号

+   • 一个5V电源用于树莓派

+   • 一张16GB的SD卡

+   • 一个兼容树莓派的单声道USB麦克风

各种类型的 USB 麦克风与 Raspberry Pi 兼容。[图 15-6](nsp-venkitachalam503045-0030.xhtml#fig15-6)展示了一个示例。

![](images/nsp-venkitachalam503045-f15006.jpg)

图 15-6：用于 Raspberry Pi 的 USB 麦克风

要检查您的 Pi 是否能够识别您的 USB 麦克风，请通过 SSH 连接到您的 Pi，并运行以下命令：

$ `dmesg -w`

现在将麦克风插入 Pi 的 USB 端口。您应该能看到类似于以下输出的内容：

[26965.023138] usb 1-1.3: New USB device found, idVendor=cafe, idProduct=4010, bcdDevice= 1.00

[26965.023163] usb 1-1.3: New USB device strings: Mfr=1, Product=2, SerialNumber=3

[26965.023179] usb 1-1.3: Product: Mico

[26965.023194] usb 1-1.3: Manufacturer: Electronut Labs

[26965.023209] usb 1-1.3: SerialNumber: 123456

信息应该与您的麦克风规格匹配，这表明它已经被正确识别。

## [代码](nsp-venkitachalam503045-0008.xhtml#rah1704)

该项目的代码分为两部分：训练部分，您将在 Google Colab 中运行，以及推理部分，您将在 Raspberry Pi 上运行。我们将逐一查看这些部分。

### [在 Google Colab 中训练模型](nsp-venkitachalam503045-0008.xhtml#rbh1703)

在本节中，我们将查看在 Google Colab 中训练语音识别模型所需的代码。我建议您在 Chrome 浏览器中使用 Colab。您将首先进行设置并下载训练数据集。然后，您将运行一些代码以了解数据。接下来，您将清理数据并为训练做准备，探索如何从数据生成频谱图。最后，您将把学到的知识应用到创建和训练模型中。最终结果将是一个 *.tflite* 文件，这是经过训练的模型的简化版 TensorFlow Lite 文件，您可以将其加载到 Raspberry Pi 上。您还可以从本书的 GitHub 仓库下载此文件：[https://github.com/mkvenkit/pp2e/tree/main/audioml](https://github.com/mkvenkit/pp2e/tree/main/audioml)。

Google Colab 笔记本由一系列单元格组成，您可以在其中输入一行或多行 Python 代码。输入所需的代码后，通过点击单元格左上角的播放图标来运行它。与该单元格代码相关的任何输出将显示在单元格下方。在本节中，每个代码清单将代表一个完整的 Google Colab 单元格。该单元格的输出（如果有）将以灰色显示在清单的底部，位于虚线下方。

#### 设置

您从 Colab 笔记本开始时需要进行一些初始设置。首先，您需要导入所需的 Python 模块：

import os

import pathlib

import matplotlib.pyplot as plt

import numpy as np

import scipy

import scipy.signal

from scipy.io import wavfile

import glob

import tensorflow as tf

from tensorflow.keras.layers.experimental import preprocessing

from tensorflow.keras import layers

from tensorflow.keras import models

from tensorflow.keras import applications

在下一个单元格中，你进行了一些初始化：

# 为随机函数设置种子

seed = 42

tf.random.set_seed(seed)

np.random.seed(seed)

在这里，你初始化了将用来打乱输入文件名的随机函数。

接下来，下载训练数据：

data_dir = 'data/mini_speech_commands'

data_path = pathlib.Path(data_dir)

filename = 'mini_speech_commands.zip'

url = "http://storage.googleapis.com/download.tensorflow.org/data/mini_speech_commands.zip"

if not data_path.exists():

tf.keras.utils.get_file(filename, origin=url, extract=True, cache_dir='.',

cache_subdir='data')

这个单元格从Google下载Mini Speech Commands数据集，并将数据解压到一个名为*data*的目录中。由于你在使用Colab，数据将被下载到云端的文件系统，而不是本地计算机，当你的会话结束时，这些文件会被删除。然而，在会话仍然活跃时，你不希望每次运行单元格时都重新下载数据。`tf.keras.utils.get_file()`函数会缓存数据，这样你就不需要每次都重新下载。

#### 了解数据

在开始训练模型之前，查看一下你刚下载的内容，以便更好地了解数据将会很有帮助。你可以使用Python的`glob`模块，它帮助你通过模式匹配查找文件和目录：

glob.glob(data_dir + '/*')

['data/mini_speech_commands/up',

'data/mini_speech_commands/no',

'data/mini_speech_commands/README.md',

'data/mini_speech_commands/stop',

'data/mini_speech_commands/left',

'data/mini_speech_commands/right',

'data/mini_speech_commands/go',

'data/mini_speech_commands/down',

'data/mini_speech_commands/yes']

你将`glob`传入`'/*'`模式，以列出*data*目录下所有一级子目录（`*`是一个通配符字符）。输出结果显示，数据集包含一个*README.md*文本文件，以及八个子目录，分别对应你将训练模型识别的八个语音命令。为了方便，你创建了一个命令列表：

commands = ['up', 'no', 'stop', 'left', 'right', 'go', 'down', 'yes']

在你的机器学习模型中，你将音频样本与一个`label_id`整数进行匹配，`label_id`表示命令之一。这个整数将与`commands`列表中的索引对应。例如，`label_id`为`0`表示`'up'`，而`label_id`为`6`表示`'down'`。

现在来看一下这些子目录中的内容：

❶ wav_file_names = glob.glob(data_dir + '/*/*')

❷ np.random.shuffle(wav_file_names)

print(len(wav_file_names))

for file_name in wav_file_names[:5]:

print(file_name)

8000

data/mini_speech_commands/down/27c30960_nohash_0.wav

data/mini_speech_commands/go/19785c4e_nohash_0.wav

data/mini_speech_commands/yes/d9b50b8b_nohash_0.wav

data/mini_speech_commands/no/f953e1af_nohash_3.wav

data/mini_speech_commands/stop/f632210f_nohash_0.wav

你再次使用`glob`，这次给它传入模式`'/*/*'`，以列出所有子目录中的文件❶。然后，你随机打乱返回的文件名列表，以减少训练数据中的偏差❷。你打印出找到的文件总数，以及前五个文件名。输出结果显示数据集中有8,000个WAV文件，并给出了一些文件命名的提示——例如，*f632210f_nohash_0.wav*。

接下来，查看数据集中的一些单独的WAV文件：

filepath = 'data/mini_speech_commands/stop/f632210f_nohash_1.wav' ❶

rate, data = wavfile.read(filepath) ❷

print("rate = {}, data.shape = {}, data.dtype = {}".format(rate, data.shape, data.dtype))

filepath = 'data/mini_speech_commands/no/f953e1af_nohash_3.wav'

rate, data = wavfile.read(filepath)

print("rate = {}, data.shape = {}, data.dtype = {}".format(rate, data.shape, data.dtype))

rate = 16000, data.shape = (13654,), data.dtype = int16

rate = 16000, data.shape = (16000,), data.dtype = int16

你设置了一个WAV文件的文件名，想要查看它❶，并使用`scipy`中的`wavefile`模块读取该文件的数据❷。然后，你打印出采样率、形状（样本数量）和数据类型。你对第二个WAV文件做了相同的操作。输出结果显示，两个WAV文件的采样率都为16,000，正如预期的那样，并且每个样本都是16位整数——也是预期中的情况。然而，形状显示，第一个文件只有13,654个样本，这是一个问题。为了训练神经网络，每个WAV文件都需要具有相同的长度；在这种情况下，你希望每个录音的长度为一秒，或者16,000个样本。遗憾的是，数据集中的并非所有文件都符合这个标准。我们稍后会看看如何解决这个问题，但首先，尝试绘制其中一个WAV文件的数据：

filepath = 'data/mini_speech_commands/stop/f632210f_nohash_1.wav'

rate, data = wavfile.read(filepath)

❶ plt.plot(data)

![](images/nsp-venkitachalam503045-g15001.jpg)

你使用`matplotlib`来创建音频波形的图表❶。这个数据集中的WAV文件包含16位有符号数据，范围从−32,768到+32,767。图表的y轴显示，该文件中的数据仅在大约−10,000到+7,500之间波动。图表的x轴还表明，数据缺少必要的16,000个样本——该轴仅显示到14,000。

#### 清理数据

你已经看到，数据集需要进行标准化，以确保每段音频长度为1秒。这种类型的准备工作被称为*数据清理*，你可以通过用零填充音频数据，直到其长度达到16,000个样本来完成它。你应该进一步清理数据，通过*归一化*它——将每个样本的值从范围[−32,768, +32,767]映射到范围[−1, 1]。这种归一化对于机器学习至关重要，因为保持输入数据小且统一将有助于训练。（对于数学上感兴趣的人来说，输入数据中的大数字会导致训练过程中梯度下降算法的收敛问题。）

作为数据清理的一个示例，在这里你对之前查看过的WAV文件应用了填充和归一化操作。然后，你绘制结果以确认清理操作已经生效。

❶ padded_data = np.zeros((16000,), dtype=np.int16)

❷ padded_data[:data.shape[0]] = data

❸ norm_data = np.array(padded_data/32768.0, dtype=np.float32)

plt.plot(norm_data)

![](images/nsp-venkitachalam503045-g15002.jpg)

你创建了一个长度为16,000的16位`numpy`数组，填充了零❶。然后，你使用数组切片操作符`[:]`将过短的WAV文件的内容复制到数组的开头❷。这里`data.shape[0]`给出了原始WAV文件的样本数，因为`data.shape`是形如`(13654,)`的元组。你现在得到了1秒钟的WAV数据，包含了原始音频数据，后面是按需填充的零。接下来，你通过将数组中的值除以32,768来创建一个归一化版本的数据，32,768是16位整数的最大值❸。然后，你绘制这些数据。

输出图的x轴显示数据已经被填充，扩展到了16,000个样本，值从大约14,000到16,000的部分全是零。同时，y轴显示这些值已经被归一化，落在了(−1, 1)的范围内。

#### 查看频谱图

如我们所讨论的，你不会直接使用WAV文件的原始数据来训练模型。相反，你将生成这些文件的频谱图，并使用它们来训练模型。下面是如何生成频谱图的一个示例：

filepath = 'data/mini_speech_commands/yes/00f0204f_nohash_0.wav'

rate, data = wavfile.read(filepath)

❶ f, t, spec = scipy.signal.stft(data, fs=16000, nperseg=255,

noverlap = 124, nfft=256)

❷ spec = np.abs(spec)

print("spec: min = {}, max = {}, shape = {}, dtype = {}".format(np.min(spec),

np.max(spec), spec.shape, spec.dtype))

❸ X = t * 129*124

❹ plt.pcolormesh(X, f, spec)

spec: min = 0.0, max = 2089.085693359375, shape = (129, 124), dtype = float32

![](images/nsp-venkitachalam503045-g15003.jpg)

你从*yes*子目录中挑选一个任意的WAV文件，并像之前一样使用`scipy`中的`wavfile`模块提取它的数据。然后，使用`scipy.signal.stft()`函数来计算数据的频谱图❶。在这个函数调用中，`fs`是采样率，`nperseg`是每个段的长度，`noverlap`是连续段之间的重叠样本数。`stft()`函数返回一个包含三个成员的元组：`f`，频率数组；`t`，映射到[0.0, 1.0]范围的时间区间数组；`spec`，STFT本身，一个129×124的复数网格（这些维度在输出中给出了`shape`）。你使用`np.abs()`将`spec`中的复数转换为实数❷。然后，你打印一些关于计算出的频谱图的信息。接下来，你创建一个数组`X`来存储与时间区间对应的样本编号❸。你通过将`t`与网格的维度相乘来获得这些样本编号。最后，你使用`pcolormesh()`方法绘制`spec`中的网格，将`X`中的值作为网格的x轴，`f`中的值作为y轴❹。

输出显示的是频谱图。这个129×124的数值网格（即图像），以及更多类似的图像，将作为神经网络的输入。从约4,000个样本点开始，1,000 Hz和更低频率的亮点区域，正是频率内容最为突出的地方，而较暗的区域则代表频率较为不明显的部分。

注意：请注意，频谱图中的y轴仅显示到大约8,000 Hz。这是数字信号处理中的*采样定理*的结果，该定理指出，在数字采样信号中，能够准确测量的最大频率是采样率的一半。在这种情况下，最大频率为16,000/2 = 8,000 Hz。

#### 训练模型

现在你已经准备好将注意力转向训练机器学习模型，这在很大程度上意味着你需要放弃像`numpy`和`scipy`这样的Python库，转而使用TensorFlow的方法和数据结构，例如`tf.Tensor`和`tf.data.Dataset`。你之所以一直使用`numpy`和`scipy`，是因为它们为你提供了一个方便的方式来探索语音命令数据集，实际上你可以继续使用它们，但那样你将错失TensorFlow所提供的优化机会，而TensorFlow是为大规模机器学习系统设计的。你会发现，TensorFlow对于你至今做的大部分计算都有几乎相同的函数，因此过渡会非常顺利。为了我们的讨论目的，当我提到*张量*时，可以理解为它类似于`numpy`数组。

为了训练机器学习模型，你需要能够从输入音频文件的文件路径中提取频谱图和标签ID（即口令）。为此，首先创建一个计算STFT的函数：

def stft(x):

❶ f, t, spec = scipy.signal.stft(x.numpy(), fs=16000, nperseg=255,

noverlap=124, nfft=256)

❷ 返回 tf.convert_to_tensor(np.abs(spec))

这个函数接收 `x`，即从 WAV 文件中提取的数据，并像之前的 ❶ 一样使用 `scipy` 计算其 STFT。然后，你将返回的 `numpy` 数组转换为 `tf.Tensor` 对象并返回结果 ❷。事实上，TensorFlow 有一个与 `scipy.signal.stft()` 方法类似的函数 `tf.signal.stft()`，那么为什么不使用它呢？答案是，TensorFlow 方法在 Raspberry Pi 上不可用，因为你将使用精简版的 TensorFlow Lite 解释器。你在训练阶段进行的任何预处理操作应当与推理阶段保持一致，因此你需要确保在 Colab 中使用的函数与在 Pi 上使用的相同。

现在你可以在一个辅助函数中使用 `stft()` 函数，该函数从文件路径中提取频谱图和标签 ID。

def get_spec_label_pair(filepath):

# 读取 WAV 文件

file_data = tf.io.read_file(filepath)

data, rate = tf.audio.decode_wav(file_data)

data = tf.squeeze(data, axis=-1)

# 为 N < 16000 添加零填充

❶ zero_padding = tf.zeros([16000] - tf.shape(data), dtype=tf.float32)

# 将数据与零填充合并

❷ padded_data = tf.concat([data, zero_padding], 0)

# 计算频谱图

❸ spec = tf.py_function(func=stft, inp=[padded_data], Tout=tf.float32)

spec.set_shape((129, 124))

spec = tf.expand_dims(spec, -1)

# 获取标签

❹ cmd = tf.strings.split(filepath, os.path.sep)[-2]

❺ label_id = tf.argmax(tf.cast(cmd == commands, "uint32"))

# 返回元组

return (spec, label_id)

你首先使用 `tf.io.read_file()` 读取文件，并使用 `tf.audio.decode_wav()` 函数解码 WAV 格式。（后者与之前使用的 `scipy.io.wavfile.read()` 函数类似。）然后，你使用 `tf.squeeze()` 将 `data` 张量的形状从 (*N*, 1) 转换为 (*N*,)，这是后续函数所要求的形状。接下来，你创建一个张量来为数据进行零填充 ❶。然而，张量是不可变对象，因此你不能像之前那样直接将 WAV 数据复制到充满零的张量中。相反，你需要创建一个张量，包含你需要的零的数量，并将其与数据张量进行拼接 ❷。

接下来，你使用 `tf.py_function()` 调用之前定义的 `stft()` 函数 ❸。在这个调用中，你还需要指定输入和输出的数据类型。这是从 TensorFlow 中调用非 TensorFlow 函数的常见方法。然后，你会对 `stft()` 返回的张量进行一些重塑。首先，使用 `set_shape()` 将其重塑为 (129, 124)，这是必要的，因为你正在从一个 TensorFlow 函数切换到一个 Python 函数，然后再返回。接着，你运行 `tf.expand_dims(spec, -1)` 来添加第三个维度，将形状从 (129, 124) 改为 (129, 124, 1)。这个额外的维度是你将在构建神经网络模型时需要的。最后，你提取与文件路径对应的标签（例如，'no'）❹ 并将标签字符串转换为整数 `label_id` ❺，它是 `commands` 列表中字符串的索引。

接下来，你需要准备输入文件进行训练。回想一下，你在子目录中有 8000 个音频文件，并且已经随机打乱了它们的文件路径字符串，并将它们放入名为 `wav_file_names` 的列表中。现在，你将数据分为三部分：80%，即 6400 个文件用于训练；10%，即 800 个文件用于验证；剩下的 10% 用于测试。这样的数据划分是机器学习中的常见做法。一旦使用训练数据训练了模型，你可以利用验证数据通过调整超参数来优化模型的准确性。测试数据仅用于检查（调整后的）模型的最终准确性。

train_files = wav_file_names[:6400]

val_files = wav_file_names[6400:7200]

test_files = wav_file_names[7200:]

现在你将文件路径字符串加载到 TensorFlow `Dataset` 对象中。这些对象对于使用 TensorFlow 至关重要；它们保存你的输入数据并允许进行数据转换，所有这些都可以在大规模下进行：

train_ds = tf.data.Dataset.from_tensor_slices(train_files)

val_ds = tf.data.Dataset.from_tensor_slices(val_files)

test_ds = tf.data.Dataset.from_tensor_slices(test_files)

看看你刚刚创建的内容：

对于 `train_ds.take(5)` 中的每一个 `val`：

print(val)

tf.Tensor(b'data/mini_speech_commands/stop/b4aa9fef_nohash_2.wav', shape=(), dtype=string)

tf.Tensor(b'data/mini_speech_commands/stop/962f27eb_nohash_0.wav', shape=(), dtype=string)

--snip--

tf.Tensor(b'data/mini_speech_commands/left/cf87b736_nohash_1.wav', shape=(), dtype=string)

每个 `Dataset` 对象包含一组 `string` 类型的张量，每个张量保存一个文件路径。然而，你真正需要的是与这些文件路径对应的 `(spec, label_id)` 配对。你将在这里创建这些配对：

train_ds = train_ds.map(get_spec_label_pair)

val_ds = val_ds.map(get_spec_label_pair)

test_ds = test_ds.map(get_spec_label_pair)

你使用`map()`将`get_spec_label_pair()`函数应用到每个`Dataset`对象。这种将函数映射到事物列表的技巧在计算中很常见。本质上，你在`Dataset`对象中的每个文件路径上调用`get_spec_label_pair()`，并将得到的`(spec, label_id)`对存储到一个新的`Dataset`对象中。

现在，你通过将数据集分割成更小的批次进一步准备数据集进行训练：

batch_size = 64

train_ds = train_ds.batch(batch_size)

val_ds = val_ds.batch(batch_size)

在这里，你将训练集和验证集的批量大小设置为64。这是加速训练过程的常见技巧。如果你试图一次性处理所有6,400个训练样本和800个验证样本，将需要大量内存并且会拖慢训练速度。

现在你终于准备好创建你的神经网络模型了：

❶ input_shape = (129, 124, 1)

❷ num_labels = 8

norm_layer = preprocessing.Normalization()

❸ norm_layer.adapt(train_ds.map(lambda x, _: x))

❹ model = `models`.Sequential([

layers.Input(shape=input_shape),

preprocessing.Resizing(32, 32),

norm_layer,

layers.Conv2D(32, 3, activation='relu'),

layers.Conv2D(64, 3, activation='relu'),

layers.MaxPooling2D(),

layers.Dropout(0.25),

layers.Flatten(),

layers.Dense(128, activation='relu'),

layers.Dropout(0.5),

layers.Dense(num_labels),

])

❺ model.summary()

模型: "sequential_3"

_____________________________________________________________________

层（类型）                      输出形状            参数数量

=====================================================================

resizing_3 (Resizing)            (None, 32, 32, 1)        0

normalization_3 (Normalization)  (None, 32, 32, 1)        3

conv2d_5 (Conv2D)                (None, 30, 30, 32)       320

conv2d_6 (Conv2D)                (None, 28, 28, 64)       18496

max_pooling2d_3 (MaxPooling2D)   (None, 14, 14, 64)       0

dropout_6 (Dropout)              (None, 14, 14, 64)       0

flatten_3 (Flatten)              (None, 12544)            0

dense_6 (Dense)                  (None, 128)              1605760

dropout_7 (Dropout)              (None, 128)              0

dense_7 (Dense)                  (None, 8)                1032

=====================================================================

总参数: 1,625,611

可训练参数: 1,625,608

非可训练参数: 3

你设置了模型第一层的输入形状 ❶，然后设置标签的数量 ❷，这将是模型输出层的单位数。接下来，你为频谱图数据设置了归一化层。这将把数据缩放并偏移到以 1 为中心，标准差为 1 的分布。这是机器学习中的常见做法，可以提高训练效果。不要让`lambda`吓到你 ❸。它所做的只是定义一个匿名函数，从训练数据集中的每个`(spec, label_id)`对中提取出频谱图。`x, _: x`只是表示忽略该对中的第二个元素，仅返回第一个元素。

接下来，你逐层创建神经网络模型 ❹。这些层对应于我们之前在[图 15-1](nsp-venkitachalam503045-0030.xhtml#fig15-1)中看到的架构。最后，你打印出模型的摘要 ❺，该摘要显示在输出中。摘要告诉你模型中的所有层、每个阶段输出张量的形状以及每层可训练参数的数量。

现在你需要编译模型。编译步骤会为模型设置优化器、损失函数和数据收集指标：

model.compile(

optimizer=tf.keras.optimizers.Adam(),

loss=`tf.keras.losses`.SparseCategoricalCrossentropy(from_logits=True),

metrics=['accuracy'],

)

*损失函数*是用来衡量神经网络表现的函数，通过将模型的输出与已知的正确训练数据进行比较。*优化器*是用来调整模型中可训练参数的方法，以减少损失。在这个例子中，你使用了`Adam`类型的优化器和`SparseCategoricalCrossentropy`类型的损失函数。你还设置了准确率指标，用于检查训练效果。

接下来，你开始训练模型：

EPOCHS = 10

history = model.fit(

train_ds,

validation_data=val_ds,

epochs=EPOCHS,

callbacks=tf.keras.callbacks.EarlyStopping(verbose=1, patience=15), ❶

)

Epoch 1/10

100/100 [==============================] - 38s 371ms/step - loss: 1.7219 - accuracy: 0.3700

- val_loss: 1.2672 - val_accuracy: 0.5763

Epoch 2/10

100/100 [==============================] - 37s 368ms/step - loss: 1.1791 - accuracy: 0.5756

- val_loss: 0.9616 - val_accuracy: 0.6650

--snip--

Epoch 10/10

100/100 [==============================] - 39s 388ms/step - loss: 0.3897 - accuracy: 0.8639

- val_loss: 0.4766 - val_accuracy: 0.8450

你通过将训练数据集 `train_ds` 传入 `model.fit()` 来训练模型。你还指定了验证数据集 `val_ds`，用于评估模型的准确性。训练会进行 10 个 *周期*。每个周期，神经网络都会看到完整的训练数据集。每次数据都会随机打乱，因此跨多个周期进行训练可以帮助模型更好地学习。你使用 `callback` 选项 ❶ 设置一个函数，如果发现训练损失在每个周期后不再下降，就退出训练。

运行此 Colab 单元格需要一些时间。训练过程中的进度将显示在屏幕上。从输出结果来看，`val_accuracy` 在第 10 个周期下显示，模型在训练结束时对验证数据的推理准确率大约为 85%。(`val_accuracy` 指标对应于验证数据，而 `accuracy` 指标对应于训练数据。)

现在你可以通过在数据的测试部分运行推理来尝试模型：

test_audio = []

test_labels = []

❶ for audio, label in test_ds:

test_audio.append(audio.numpy())

test_labels.append(label.numpy())

❷ test_audio = np.array(test_audio)

test_labels = np.array(test_labels)

❸ y_pred = np.argmax(model.predict(test_audio), axis=1)

y_true = test_labels

❹ test_acc = sum(y_pred == y_true) / len(y_true)

print(f'测试集准确率: {test_acc:.0%}')

25/25 [==============================] - 1s 35ms/step

测试集准确率: 84%

你首先通过遍历测试数据集 `test_ds` 填充两个列表 `test_audio` 和 `test_labels` ❶。然后将这些列表转换为 `numpy` 数组 ❷，并在数据上运行推理 ❸。你通过计算预测与真实值匹配的次数并将其除以总数来计算测试准确率 ❹。输出结果显示准确率为 84%。虽然不是完美的，但对这个项目来说已经足够好。

#### 导出模型到 Raspberry Pi

恭喜！你已经训练好了一个完整的机器学习模型。现在你需要将它从 Colab 转移到你的 Raspberry Pi 上。第一步是保存模型：

model.save('audioml.sav')

这将把模型保存到云端的一个名为 *audioml.sav* 的文件中。接下来，将该文件转换为 TensorFlow Lite 格式，这样你就可以在 Raspberry Pi 上使用它：

❶ converter = tf.lite.TFLiteConverter.from_saved_model('audioml.sav')

❷ tflite_model = converter.convert()

❸ with open('audioml.tflite', 'wb') as f:

f.write(tflite_model)

你创建了一个 `TFLiteConverter` 对象，并传入保存的模型文件名 ❶。然后进行转换 ❷，并将简化后的 TensorFlow 模型写入名为 *audioml.tflite* 的文件 ❸。现在你需要从 Colab 下载这个 *.tflite* 文件到你的电脑。运行以下代码片段会弹出一个浏览器提示，让你保存 *.tflite* 文件：

from google.colab import files

files.download('audioml.tflite')

一旦你有了文件，就可以像我们在其他章节中讨论过的那样，通过 SSH 将其传输到你的 Raspberry Pi。

### [在 Raspberry Pi 上使用模型](nsp-venkitachalam503045-0008.xhtml#rbh1704)

现在我们将注意力转向 Raspberry Pi 部分的代码。此代码使用并行处理从麦克风获取音频数据，将数据准备好供你训练好的 ML 模型使用，并将数据传递给模型进行推理。像往常一样，你可以在本地计算机上编写代码，然后通过 SSH 将其传输到 Pi 上。要查看完整代码，请参见 [“完整代码”](nsp-venkitachalam503045-0030.xhtml#ah1708) 第 389 页 (nsp-venkitachalam503045-0030.xhtml#p389)。你也可以从 [https://github.com/mkvenkit/pp2e/tree/main/audioml](https://github.com/mkvenkit/pp2e/tree/main/audioml) 下载代码。

#### 设置

首先导入所需的模块：

from scipy.io import wavfile

from scipy import signal

import numpy as np

import argparse

import pyaudio

import wave

import time

from tflite_runtime.interpreter import Interpreter

from multiprocessing import Process, Queue

接下来，你将初始化一些作为全局变量定义的参数：

VERBOSE_DEBUG = False

CHUNK = 4000

FORMAT = pyaudio.paInt16

CHANNELS = 1

SAMPLE_RATE = 16000

RECORD_SECONDS = 1

NCHUNKS = int((SAMPLE_RATE * RECORD_SECONDS) / CHUNK)

ND = 2 * CHUNK * NCHUNKS

NDH = ND // 2

# 麦克风的设备索引

❶ dev_index = -1

`VERBOSE_DEBUG` 是你将在代码中多个地方使用的一个标志。现在，你将其设置为 `False`，但如果设置为 `True`（通过命令行选项），它将输出大量调试信息。

注意：我省略了后续代码清单中的 `print()` 调试语句。你可以在完整代码清单和书籍的 GitHub 仓库中找到它们。

接下来是一些用于处理音频输入的全局变量。`CHUNK` 设置每次读取的数据样本数量，使用的是 `PyAudio`，`FORMAT` 指定音频数据将由 16 位整数构成。你将 `CHANNELS` 设置为 `1`，因为你将使用单通道麦克风，并将 `SAMPLE_RATE` 设置为 `16000`，以确保与 ML 训练数据的一致性。`RECORD_SECONDS` 表示你将音频分为每秒一段（稍后会将其拼接成重叠的两秒片段，如前所述）。你将计算每秒录音中的块数，即 `NCHUNKS`。你将使用 `ND` 和 `NDH` 来实现重叠技术——稍后会详细讲解。

最后，你将麦克风的设备索引初始化为`-1` ❶。一旦知道麦克风的索引，你需要在命令行中更新该值。这里有一个函数可以帮助你找出麦克风的索引。你可以将这个函数作为命令行选项调用。

def list_devices():

"""列出 pyaudio 设备"""

# 初始化 pyaudio

❶ p = pyaudio.PyAudio()

# 获取设备列表

index = None

❷ nDevices = p.get_device_count()

print('\naudioml.py:\n发现以下输入设备：')

❸ for i in range(nDevices):

deviceInfo = p.get_device_info_by_index(i)

if deviceInfo['maxInputChannels'] > 0:

print(deviceInfo['index'], deviceInfo['name'],

deviceInfo['defaultSampleRate'])

# 清理工作

❹ p.terminate()

你初始化了`PyAudio`❶并获取它检测到的音频设备数量❷。然后你遍历设备❸。对于每个设备，你使用`get_device_info_by_index()`获取设备信息，并打印出具有一个或多个输入通道的设备——即麦克风。最后，你清理`PyAudio`❹。

下面是该函数的典型输出：

audioml.py:

找到了以下输入设备：

1 Mico: USB Audio (hw:3,0) 16000.0

这表示有一个名为Mico的输入设备，默认采样率为16,000，索引为`1`。

#### 接收音频数据

Pi的主要任务之一是持续地从麦克风接收音频输入，并将其分割成可以进行推理的片段。为此，你创建了一个`get_live_input()`函数。该函数接收一个`interpreter`对象，用于与TensorFlow Lite模型进行交互。下面是该函数的开始部分：

def get_live_input(interpreter):

# 创建队列对象

❶ dataq = Queue()

# 启动推理进程

❷ proc = Process(target = inference_process, args=(dataq, interpreter))

proc.start()

正如我们在[“工作原理”](nsp-venkitachalam503045-0030.xhtml#ah1702)中讨论的那样，为了避免漏掉任何输入，你需要使用不同的进程来读取音频数据和进行推理。你创建了一个`multiprocessing.Queue`对象，供这些进程之间进行通信❶。然后你创建了推理进程，使用`multiprocessing.Process()`❷。你指定了进程的处理函数为`inference_process`，该函数以`dataq`和`interpreter`对象作为参数（稍后我们将查看此函数）。接下来，你启动进程，使推理与数据捕获并行运行。

你继续`get_live_input()`函数，初始化`PyAudio`：

# 初始化pyaudio

❶ p = pyaudio.PyAudio()

print('正在打开流...')

❷ stream = p.open(format = FORMAT,

channels = CHANNELS,

rate = SAMPLE_RATE,

input = True,

frames_per_buffer = CHUNK,

input_device_index = dev_index)

# 丢弃前1秒

❸ for i in range(0, NCHUNKS):

data = stream.read(CHUNK, exception_on_overflow = False)

你创建了一个`PyAudio`对象`p`❶并打开一个音频输入流❷，使用一些全局变量作为参数。然后，你丢弃了前1秒的数据❸。这样做是为了忽略麦克风第一次启用时可能出现的无效数据。

现在你准备好开始读取数据了：

# 计数，用于每次收集两个帧

❶ count = 0

❷ inference_data = np.zeros((ND,), dtype=np.int16)

print("正在监听...")

try:

❸ while True:

chunks = []

❹ for i in range(0, NCHUNKS):

data = stream.read(CHUNK, exception_on_overflow = False)

chunks.append(data)

# 处理数据

buffer = b''.join(chunks)

❺ audio_data = np.frombuffer(buffer, dtype=np.int16)

你将`count`初始化为`0` ❶。你将使用这个变量来跟踪读取的每一秒音频数据的数量。接下来，你初始化一个全为零的16位数组`inference_data` ❷。它有`ND`个元素，对应两秒钟的音频数据。然后你进入一个`while`循环，持续处理音频数据 ❸。在这个循环中，你使用一个`for`循环 ❹，每次读取一秒钟的音频数据，一次处理一个数据块，并将这些数据块附加到`chunks`列表中。一旦你得到完整的一秒数据，就将其转换成一个`numpy`数组 ❺。

接下来，仍然在之前列出的`while`循环内，你实现了我们在[“它是如何工作的”](nsp-venkitachalam503045-0030.xhtml#ah1702)中讨论的技术，创建了重叠的两秒音频片段。你从全局变量`NDH`中获取帮助。

if count == 0:

# 设置前半部分

❶ inference_data[:NDH] = audio_data

count += 1

elif count == 1:

# 设置后半部分

❷ inference_data[NDH:] = audio_data

# 将数据添加到队列

❸ dataq.put(inference_data)

count += 1

else:

# 将后半部分移动到前半部分

❹ inference_data[:NDH] = inference_data[NDH:]

# 设置后半部分

❺ inference_data[NDH:] = audio_data

# 将数据添加到队列

❻ dataq.put(inference_data)

第一次读取一秒钟的音频帧时，将其存储在`inference_data`的前半部分 ❶。接下来的帧存储在`inference_data`的后半部分 ❷。现在你有了完整的两秒钟音频数据，因此将`inference_data`放入队列，供推理过程提取 ❸。对于随后的每一帧，后半部分的数据将被移动到`inference_data`的前半部分 ❹，新数据会设置到后半部分 ❺，然后`inference_data`会被加入队列 ❻。这样就形成了每个连续的两秒音频片段之间的一秒重叠。

`while`循环发生在`try`块内部。要退出循环，只需按CTRL-C并触发以下`except`块：

except KeyboardInterrupt:

print("正在退出...")

stream.stop_stream()

stream.close()

p.terminate()

这个`except`块执行一些基本的清理操作，停止并关闭流，并终止`PyAudio`。

#### 准备音频数据

接下来，你将创建几个函数来准备音频数据进行推理。第一个是`process_audio_data()`，它接收从队列中提取的原始两秒音频数据，并根据峰值幅度提取出最有趣的一秒音频。我们将在接下来的几段代码中查看这个函数：

def process_audio_data(waveform):

# 通过最大16位值缩放计算峰值-峰值

❶ PTP = np.ptp(waveform / 32768.0)

# 如果过于安静则返回 None

❷ if PTP < 0.3:

return []

如果没有人在说话，你不希望对麦克风音频输入进行推理。然而，环境中总会有一些噪音，因此你不能仅仅查看信号是否为0。相反，如果音频的峰值到峰值幅度（即最高值和最低值之间的差）低于某个阈值，你将跳过推理过程。为此，首先将音频除以`32768`，将其归一化到（−1, 1）范围内，然后将结果传递给`np.ptp()`以获得峰值到峰值幅度❶。归一化使得可以更容易地将阈值表示为一个分数。如果峰值到峰值幅度低于`0.3`❷，你将返回一个空列表（这将跳过推理过程）。你可能需要根据环境噪音水平调整该阈值。

`process_audio_data()` 函数继续使用另一种技术来归一化任何不会被跳过的音频数据：

# 音频归一化

wabs = np.abs(waveform)

wmax = np.max(wabs)

❶ waveform = waveform / wmax

# 基于归一化波形计算峰值到峰值

❷ PTP = np.ptp(waveform)

# 缩放和居中

❸ waveform = 2.0*(waveform - np.min(waveform))/PTP – 1

当你在跳过安静的音频样本之前对数据进行归一化时，你将音频除以了32,768，这是16位有符号整数的最大可能值。然而，在大多数情况下，音频数据的峰值幅度将远低于该值。现在，你希望将音频归一化，使其最大幅度无论是多少，都被缩放到1。为此，你首先确定音频信号的峰值幅度，然后将信号除以该幅度值❶。接着，你计算归一化音频的新的峰值到峰值幅度❷，并使用该值来缩放和居中数据❸。具体来说，表达式`(waveform – np.min(waveform))/PTP`将把波形值缩放到（0, 1）范围。将其乘以2并减去1将把值放置在（−1, 1）范围内，这是你需要的。

函数的下一部分从数据中提取1秒钟的音频：

# 提取16000个采样点（1秒）数据

❶ max_index = np.argmax(waveform)

❷ start_index = max(0, max_index-8000)

❸ end_index = min(max_index+8000, waveform.shape[0])

❹ waveform = waveform[start_index:end_index]

# 对于少于16000个采样点的文件进行填充

waveform_padded = np.zeros((16000,))

waveform_padded[:waveform.shape[0]] = waveform

return waveform_padded

你需要确保你获取的是数据中最有趣的那一秒，所以你找到音频幅度最大的位置索引❶。然后，你尝试在这个索引前❷和后❸各抓取8,000个值，以获取完整的一秒数据，并使用`max()`和`min()`来确保起始和结束索引不会超出原始音频的范围。你使用切片来提取相关的音频数据❹。由于`max()`和`min()`操作，最终你可能会得到少于16,000个样本，但神经网络严格要求每个输入必须是16,000个样本长。为了解决这个问题，你用零填充数据，采用你在训练中看到的相同`numpy`技巧。然后，你返回结果。

[图 15-7](nsp-venkitachalam503045-0030.xhtml#fig15-7)通过展示在不同处理阶段的示例波形，总结了`process_audio_data()`函数。

![](images/nsp-venkitachalam503045-f15007.jpg)

图 15-7：音频准备过程的不同阶段

[图 15-7](nsp-venkitachalam503045-0030.xhtml#fig15-7)中的上方波形展示了未经处理的音频。第二个波形展示了音频经过归一化处理后的值，范围为（-1，1）。第三个波形展示了音频经过平移和缩放后的效果—注意在y轴上，波形现在填满了整个（-1，1）的范围。第四个波形由从第三个波形中提取的16,000个样本组成，中心在最大幅度的位置。

接下来，你需要一个`get_spectrogram()`函数来计算音频数据的频谱图：

定义get_spectrogram(waveform)函数：

❶ waveform_padded = process_audio_data(waveform)

❷ 如果波形数据为空：

return []

# 计算频谱图

❸ f, t, Zxx = signal.stft(waveform_padded, fs=16000, nperseg=255,

noverlap = 124, nfft=256)

# 输出是复数值，所以取绝对值

❹ spectrogram = np.abs(Zxx)

return spectrogram

你调用`process_audio_data()`函数来准备音频❶。如果该函数返回一个空列表（因为音频太安静），`get_spectrogram()`也会返回一个空列表❷。接下来，你使用`scipy`中的`signal.stft()`计算音频的频谱图，正如你在训练模型时做的那样❸。然后你计算STFT的绝对值❹，以从复数转换为实数—这也是你在训练时做的，并返回结果。

#### 运行推理

这个项目的核心是使用你训练好的模型对传入的音频数据进行推理，并识别出任何语音命令。回想一下，这个过程与从麦克风获取音频数据的代码是分开的。下面是协调这一过程的处理函数：

def inference_process(dataq, interpreter):

success = False

while True:

❶ 如果数据队列不为空：

# 从队列获取数据

❷ inference_data = dataq.get()

# 只有在上一个处理不成功时才运行推理

❸ 如果处理不成功：

success = run_inference(inference_data, interpreter)

else:

# 跳过，重置标志以便下次使用

❹ success = False

推理过程在`while`循环内持续运行。在这个循环中，你会检查队列中是否有数据 ❶，如果有，你就将其取出 ❷。然后，你使用`run_inference()`函数对其进行推理，我们接下来会看这个函数，但只有在`success`标志为`False`时 ❸，才会进行推理。这个标志防止你对同一个语音命令作出两次响应。回想一下，由于重叠技术的使用，一个音频片段的后半部分会在下一个片段中作为前半部分重复出现。这允许你捕捉到可能跨越两个帧的音频命令，但这也意味着，一旦推理成功，你应该跳过队列中的下一个元素，因为它包含了前一个元素的一部分音频。当你进行这样的跳过时，你需要将`success`重置为`False` ❹，以便重新开始对下一个数据片段进行推理。

现在让我们来看一下`run_inference()`函数，推理实际上是在这个函数中进行的：

def run_inference(waveform, interpreter):

# 获取谱图数据

❶ spectrogram = get_spectrogram(waveform)

if not len(spectrogram):

return False

# 获取输入和输出张量的详细信息

❷ input_details = interpreter.get_input_details()

❸ output_details = interpreter.get_output_details()

这个函数接受原始音频数据（`waveform`），以与TensorFlow Lite模型（`interpreter`）进行交互。你调用`get_spectrogram()`来处理音频并生成谱图 ❶，如果音频太安静，则返回`False`。然后，你从TensorFlow Lite解释器获取输入 ❷ 和输出 ❸ 详细信息。这些信息告诉你模型期望的输入是什么，以及你可以期待的输出是什么。这就是`input_details`的样子：

[{'name': 'serving_default_input_5:0', 'index': 0, 'shape': array([1, 129, 124,   1]),

'shape_signature': array([ -1, 129, 124,   1]), 'dtype': <class 'numpy.float32'>,

'quantization': (0.0, 0), 'quantization_parameters': {'scales': array([], dtype=float32),

'zero_points': array([], dtype=int32), 'quantized_dimension': 0}, 'sparsity_parameters': {}}]

请注意，`input_details`是一个包含字典的数组。`'shape'`条目尤其重要：`array([1, 129, 124, 1])`。你已经确保了你的谱图（将作为输入传递给解释器）的形状符合这个值。`'index'`条目只是张量在解释器内部张量列表中的索引，`'dtype'`是输入的预期数据类型，在本例中是`float32`，即一个带符号的32位浮动点数。你稍后将在`run_inference()`函数中引用`'index'`和`'dtype'`。

这里是`output_details`：

[{'name': 'StatefulPartitionedCall:0', 'index': 17, 'shape': array([1, 8]), 'shape_signature':

array([-1,  8]), 'dtype': <class 'numpy.float32'>, 'quantization': (0.0, 0),

'quantization_parameters': {'scales': array([], dtype=float32), 'zero_points':

array([], dtype=int32), 'quantized_dimension': 0}, 'sparsity_parameters': {}}]

注意这个字典中的`'shape'`条目。它显示输出将是一个形状为(1, 8)的数组。该形状对应于八个语音命令的标签ID。

你通过实际运行推理来继续`run_inference()`函数：

# 设置输入

❶ input_data = spectrogram.astype(np.float32)

❷ interpreter.set_tensor(input_details[0]['index'], input_data)

# 运行解释器

print("正在运行推理...")

❸ interpreter.invoke()

# 获取输出

❹ output_data = interpreter.get_tensor(output_details[0]['index'])

❺ yvals = output_data[0]

print(yvals)

首先，你将谱图数据转换为32位浮点值❶。回想一下，你的音频数据最初是16位整数。缩放和其他处理操作将数据转换为64位浮动数，但正如你在`input_details`中看到的，TensorFlow Lite模型要求使用32位浮点数，这就是转换的原因。接下来，你将输入值设置为解释器内部适当的张量❷。这里的`[0]`访问`input_details`中的第一个（也是唯一一个）元素，它是一个字典，`['index']`从字典中获取该键下的值，以指定你要设置的张量。你使用`invoke()`方法对输入进行推理❸。然后，你使用类似的索引方式从`output_data`数组中检索输出张量❹，并通过提取`output_data`数组中的第一个元素来获取输出本身❺。（由于你只提供了一个输入，因此预计只有一个输出。）这是`yvals`的一个示例：

[ 6.640185 -26.032831 -26.028618 8.746256 62.545185 -0.5698182 -15.045679 -29.140179 ]

这八个数字对应于你用来训练模型的八个命令。这些值表示输入数据为每个词的可能性。在这个特定的数组中，索引`4`处的值最大，因此神经网络预测这是最可能的答案。以下是如何解释结果：

# 重要！这应该与训练标签/ID完全匹配。

commands = ['up', 'no', 'stop', 'left', 'right', 'go', 'down', 'yes']

print(">>> " + commands[np.argmax(output_data[0])].upper())

你定义了一个`commands`列表，顺序与训练时使用的顺序相同。保持训练和推理中的顺序一致非常重要，否则你可能会误解结果！然后，你使用`np.argmax()`来获取输出数据中最大值的索引，并利用该索引从`commands`中获取相应的字符串。

#### 编写`main()`函数

现在让我们看看`main()`函数，它将所有内容整合在一起：

def main():

# 此函数中设置的全局变量

global VERBOSE_DEBUG

# 创建解析器

descStr = "这个程序对音频数据进行机器学习推理。"

parser = argparse.ArgumentParser(description=descStr)

# 添加一个互斥组

❶ group = parser.add_mutually_exclusive_group(required=True)

# 添加互斥参数

❷ group.add_argument('--list', action='store_true', required=False)

❸ group.add_argument('--input', dest='wavfile_name', required=False)

❹ group.add_argument('--index', dest='index', required=False)

# 添加其他参数

❺ parser.add_argument('--verbose', action='store_true', required=False)

# 解析参数

args = parser.parse_args()

你首先设置 `VERBOSE_DEBUG` 为全局变量，因为你将在此函数中设置它，不希望它被当作局部变量。然后你创建一个常见的 `argparse.ArgumentParser` 对象，并为解析器添加一个互斥组 ❶，因为你的某些命令行选项不能相互兼容。包括 `--list` 选项 ❷，该选项列出所有 `PyAudio` 设备，以便你可以获取麦克风的索引号；`--input` 选项 ❸，该选项允许你指定一个 WAV 文件作为输入，而不是来自麦克风的实时数据（用于测试）；以及 `--index` 选项 ❹，该选项启动音频捕捉并使用指定的麦克风索引进行推理。你还添加了一个非互斥的 `--verbose` 选项 ❺，它会在程序运行时打印详细的调试信息。

接下来，你创建 TensorFlow Lite 解释器，以便可以使用 ML 模型：

# 加载 TF Lite 模型

interpreter = Interpreter('audioml.tflite')

interpreter.allocate_tensors()

在这里，你创建一个 `Interpreter` 对象，传递给它包含你在训练期间创建的模型的 *audioml.tflite* 文件。然后，你调用 `allocate_tensors()` 准备运行推理所需的张量。

`main()`函数根据不同的命令行参数完成分支：

# 检查详细标志

if args.verbose:

VERBOSE_DEBUG = True

# 测试 WAV 文件

if args.wavfile_name:

❶ wavfile_name = args.wavfile_name

# 获取音频数据

❷ rate, waveform = wavfile.read(wavfile_name)

# 运行推理

❸ run_inference(waveform, interpreter)

elif args.list:

# 列出设备

❹ list_devices()

else:

# 存储设备索引

❺ dev_index = int(args.index)

# 获取实时音频

❻ get_live_input(interpreter)

print("完成。")

如果使用了 `--input` 命令行选项，你将获得 WAV 文件的名称 ❶ 并读取其内容 ❷。结果数据将传递进行推理 ❸。如果使用了 `--list` 选项，你将调用 `list_devices()` 函数 ❹。如果使用了 `--index` 选项，你将解析设备索引 ❺ 并通过调用 `get_live_input()` 函数 ❻ 开始处理实时音频。

## [运行语音识别系统](nsp-venkitachalam503045-0008.xhtml#rah1705)

要运行该项目，将你的 Python 代码和 *audioml.tflite* 文件放入 Pi 上的一个文件夹中。为了测试，你也可以从书籍的 GitHub 仓库下载 *right.wav* 文件并将其添加到文件夹中。你可以通过 SSH 操作 Pi，具体操作请参考 [附录 B](nsp-venkitachalam503045-0032.xhtml#appb)。

首先，尝试使用 `--input` 命令行选项对 WAV 文件进行推理：

$ `sudo python audioml.py --input right.wav`

以下是输出：

正在运行推理...

[  6.640185  -26.032831  -26.028618    8.746256   62.545185   -0.5698182

-15.045679  -29.140179 ]

❶ >>> 右

run_inference: 0.029174549999879673秒

完成。

请注意，程序已经正确识别了录制在WAV文件中的*right*命令，位置在❶。

现在将你的麦克风插入Pi，并使用`--list`选项来确定其索引号，如下所示：

$ `sudo python audioml.py --list`

你的输出应与以下类似：

audioml.py:

找到以下输入设备：

1 Mico: USB Audio (hw:3,0) 16000.0

完成。

在这个示例中，麦克风的索引是`1`。使用这个数字运行`--index`命令来进行实时语音检测！以下是一个示例运行：

$ `sudo python audioml.py --index 1`

--`snip`--

正在打开流...

正在监听...

正在运行推理...

[-2.647918    0.17592785 -3.3615346   6.6812882   4.472283   -3.7535028

1.2349942   1.8546474 ]

❶ >>> 左

run_inference: 0.03520956500142347秒

正在运行推理...

[-2.7683923 -5.9614644 -8.532391   6.906795  19.197264  -4.0255833

1.7236844 -4.374415 ]

❷ >>> 右

run_inference: 0.03026762299850816秒

--`snip`--

^C

KeyboardInterrupt

正在退出...

完成。

在启动程序并看到“Listening …”提示后，我说出了“*left*”和“*right*”这两个词。❶和❷处的输出表明程序能够正确识别这些命令。

尝试使用`--verbose`选项运行程序，查看更多关于其工作方式的信息。同时，尝试快速连续地说出不同的命令，以验证多进程和重叠技术是否有效。

## [总结](nsp-venkitachalam503045-0008.xhtml#rah1706)

本章向你介绍了机器学习的世界。你学习了如何使用TensorFlow框架训练一个深度神经网络来识别语音命令，并将结果模型转换为TensorFlow Lite格式以在资源受限的树莓派上使用。你还了解了频谱图及其在机器学习训练中处理输入数据的重要性。你实践了使用Python多进程技术，使用`PyAudio`读取树莓派上的USB麦克风输入，以及运行TensorFlow Lite解释器进行机器学习推理。

## [实验！](nsp-venkitachalam503045-0008.xhtml#rah1707)

1.  1. 现在你知道如何在树莓派上处理语音命令，你可以构建一个响应这些命令的辅助设备，不仅仅是打印出识别到的词。例如，你可以使用*left*、*right*、*up*、*down*、*stop* 和 *go* 命令来控制安装在云台上的相机（或激光器！）。提示：你需要用这六个命令重新训练机器学习模型。你还需要获取一个带有两个伺服电机的双轴云台支架。伺服电机将连接到树莓派，并根据推理结果进行控制。

1.  2. 阅读关于*mel频谱图*的内容，这是一种比你在本项目中使用的频谱图更适合人类语音数据的变体。

1.  3\. 尝试通过添加或删除一些层来修改神经网络。例如，删除第二个Conv2D层。观察这些变化如何影响模型的训练准确性以及在Pi上的推理准确性。

1.  4\. 这个项目使用了一个临时神经网络，但也有可用的预训练神经网络，你可以利用它们。例如，了解MobileNet V2。为了使用这个网络，你的项目需要做哪些改动？

## [完整代码](nsp-venkitachalam503045-0008.xhtml#rah1708)

下面是用于树莓派的完整代码清单，包括用于详细调试的`print()`语句。Google Colab笔记本代码可以在[https://github.com/mkvenkit/pp2e/blob/main/audioml/audioml.ipynb](https://github.com/mkvenkit/pp2e/blob/main/audioml/audioml.ipynb)找到。

"""

simple_audio.py

这个程序从树莓派上的I2S麦克风收集音频数据

并在每个构建模型上运行TensorFlow Lite解释器。

作者：Mahesh Venkitachalam

"""

from scipy.io import wavfile

from scipy import signal

import numpy as np

import argparse

import pyaudio

import wave

import time

from tflite_runtime.interpreter import Interpreter

from multiprocessing import Process, Queue

VERBOSE_DEBUG = False

CHUNK = 4000                # 选择一个能被SAMPLE_RATE整除的值

FORMAT = pyaudio.paInt16

CHANNELS = 1

SAMPLE_RATE = 16000

RECORD_SECONDS = 1

NCHUNKS = int((SAMPLE_RATE * RECORD_SECONDS) / CHUNK)

ND = 2 * SAMPLE_RATE * RECORD_SECONDS

NDH = ND // 2

# 麦克风的设备索引

dev_index = -1

def list_devices():

"""列出pyaudio设备"""

# 初始化pyaudio

p = pyaudio.PyAudio()

# 获取设备列表而得到重复结果

index = None

nDevices = p.get_device_count()

print('\naudioml.py:\n找到以下输入设备：')

for i in range(nDevices):

deviceInfo = p.get_device_info_by_index(i)

if deviceInfo['maxInputChannels'] > 0:

print(deviceInfo['index'], deviceInfo['name'], deviceInfo['defaultSampleRate'])

# 清理

p.terminate()

def inference_process(dataq, interpreter):

"""推理过程处理器"""

success = False

while True:

if not dataq.empty():

# 从队列获取数据

inference_data = dataq.get()

# 仅在上一个推理失败时才进行推理

# 否则，我们会因为

# 输入数据的重叠

if not success:

success = run_inference(inference_data, interpreter)

else:

# 跳过，重置标志以便下次使用

success = False

def process_audio_data(waveform):

"""处理音频输入。

该函数从WAV文件中接收原始音频数据并进行缩放

并将长度填充到16000。

"""

if VERBOSE_DEBUG:

print("waveform:", waveform.shape, waveform.dtype, type(waveform))

print(waveform[:5])

# 基于最大16位值的缩放计算峰值到峰值

PTP = np.ptp(waveform / 32768.0)

if VERBOSE_DEBUG:

print("峰值-峰值（16位缩放）：{}".format(PTP))

# 如果音频过于安静，则返回None

if PTP < 0.3:

return []

# 归一化音频

wabs = np.abs(waveform)

wmax = np.max(wabs)

waveform = waveform / wmax

# 基于归一化波形计算峰值到峰值

PTP = np.ptp(waveform)

if VERBOSE_DEBUG:

print("peak-to-peak (after normalize): {}".format(PTP))

print("After normalization:")

print("waveform:", waveform.shape, waveform.dtype, type(waveform))

print(waveform[:5])

# 缩放和居中

waveform = 2.0*(waveform - np.min(waveform))/PTP - 1

# 提取16000个长度（1秒）的数据

max_index = np.argmax(waveform)

start_index = max(0, max_index-8000)

end_index = min(max_index+8000, waveform.shape[0])

waveform = waveform[start_index:end_index]

# 为少于16000个样本的文件进行填充

if VERBOSE_DEBUG:

print("After padding:")

waveform_padded = np.zeros((16000,))

waveform_padded[:waveform.shape[0]] = waveform

if VERBOSE_DEBUG:

print("waveform_padded:", waveform_padded.shape,

waveform_padded.dtype, type(waveform_padded))

print(waveform_padded[:5])

return waveform_padded

def get_spectrogram(waveform):

"""从音频数据计算频谱图"""

waveform_padded = process_audio_data(waveform)

if not len(waveform_padded):

return []

# 计算频谱图

f, t, Zxx = signal.stft(waveform_padded, fs=16000, nperseg=255,

noverlap = 124, nfft=256)

# 输出是复数，因此取其绝对值

spectrogram = np.abs(Zxx)

if VERBOSE_DEBUG:

print("spectrogram:", spectrogram.shape, type(spectrogram))

print(spectrogram[0, 0])

return spectrogram

def run_inference(waveform, interpreter):

# 开始计时

start = time.perf_counter()

# 获取频谱图数据

spectrogram = get_spectrogram(waveform)

if not len(spectrogram):

if VERBOSE_DEBUG:

print("太安静，跳过...")

return False

if VERBOSE_DEBUG:

print("spectrogram: %s, %s, %s" % (type(spectrogram),

spectrogram.dtype, spectrogram.shape))

# 获取输入和输出张量的详细信息

input_details = interpreter.get_input_details()

output_details = interpreter.get_output_details()

if VERBOSE_DEBUG:

print("input_details: {}".format(input_details))

print("output_details: {}".format(output_details))

# 重塑频谱图以匹配推理器要求

spectrogram = np.reshape(spectrogram, (-1, spectrogram.shape[0],

spectrogram.shape[1], 1))

# 设置输入

input_data = spectrogram.astype(np.float32)

interpreter.set_tensor(input_details[0]['index'], input_data)

# 运行推理器

print("running inference...")

interpreter.invoke()

# 获取输出

output_data = interpreter.get_tensor(output_details[0]['index'])

yvals = output_data[0]

if VERBOSE_DEBUG:

print(output_data)

print(yvals)

# 重要！这应该与训练标签/ID完全匹配。

commands = ['up', 'no', 'stop', 'left', 'right', 'go', 'down', 'yes']

print(">>> " + commands[np.argmax(output_data[0])].upper())

# 停止计时

end = time.perf_counter()

print("run_inference: {}s".format(end - start))

# 返回成功

return True

def get_live_input(interpreter):

"""此函数从麦克风获取实时输入

并在其上运行推理"""

# 创建队列对象

dataq = Queue()

# 开始推理过程

proc = Process(target = inference_process, args=(dataq, interpreter))

proc.start()

# 初始化 pyaudio

p = pyaudio.PyAudio()

print('正在打开流...')

stream = p.open(format = FORMAT,

channels = CHANNELS,

rate = SAMPLE_RATE,

input = True,

frames_per_buffer = CHUNK,

input_device_index = dev_index)

# 丢弃前 1 秒

for i in range(0, NCHUNKS):

data = stream.read(CHUNK, exception_on_overflow = False)

# 用于一次收集两帧数据的计数

count = 0

inference_data = np.zeros((ND,), dtype=np.int16)

print("正在监听...")

try:

while True:

# print("正在监听...")

chunks = []

for i in range(0, NCHUNKS):

data = stream.read(CHUNK, exception_on_overflow = False)

chunks.append(data)

# 处理数据

buffer = b''.join(chunks)

audio_data = np.frombuffer(buffer, dtype=np.int16)

if count == 0:

# 设置前半部分

inference_data[:NDH] = audio_data

count += 1

elif count == 1:

# 设置后半部分

inference_data[NDH:] = audio_data

# 将数据添加到队列

dataq.put(inference_data)

count += 1

else:

# 将后半部分移至前半部分

inference_data[:NDH] = inference_data[NDH:]

# 设置后半部分

inference_data[NDH:] = audio_data

# 将数据添加到队列

dataq.put(inference_data)

# print("队列大小: {}".format(dataq.qsize()))

except KeyboardInterrupt:

print("正在退出...")

stream.stop_stream()

stream.close()

p.terminate()

def main():

"""程序的主函数"""

# 在此函数中设置全局变量

global VERBOSE_DEBUG

# 创建解析器

descStr = "该程序对音频数据进行机器学习推理。"

parser = argparse.ArgumentParser(description=descStr)

# 添加互斥组

group = parser.add_mutually_exclusive_group(required=True)

# 添加互斥参数

group.add_argument('--list', action='store_true', required=False)

group.add_argument('--input', dest='wavfile_name', required=False)

group.add_argument('--index', dest='index', required=False)

# 添加其他参数

parser.add_argument('--verbose', action='store_true', required=False)

# 解析参数

args = parser.parse_args()

# 加载 TF Lite 模型

interpreter = Interpreter('audioml.tflite')

interpreter.allocate_tensors()

# 检查详细标志

if args.verbose:

VERBOSE_DEBUG = True

# 测试 WAV 文件

if args.wavfile_name:

wavfile_name = args.wavfile_name

# 获取音频数据

rate, waveform = wavfile.read(wavfile_name)

# 运行推理

run_inference(waveform, interpreter)

elif args.list:

# 列出设备

list_devices()

else:

# 存储设备索引

dev_index = int(args.index)

# 获取实时音频

get_live_input(interpreter)

print("完成。")

# main 方法

if __name__ == '__main__':

main()
