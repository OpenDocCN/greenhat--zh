# 第一章：A

安装播放音频文件的模块

![](img/chapterart.png)

在本附录中，我将讨论在 Python 中播放音频文件的各种模块。虽然无法考虑到所有硬件和操作系统组合的差异，但我已经在多种硬件和软件平台上测试了本书中的指示。在此过程中，我遇到了各种问题，我希望帮助你避免这些问题。

播放音频文件有两种类型的模块。第一种类型（我们可以称之为*阻塞型*）将控制脚本的执行，直到音频文件播放完毕才会继续执行下一行代码。我们将讨论这一类别中的两个模块：*playsound* 和 *pydub*。你只需要使其中一个模块在本书中正常工作即可。第二种类型不会控制脚本的执行；它会在音频文件开始播放后立即继续执行下一行代码（我们可以称之为*非阻塞型*）。我们将讨论这一类别中的两个模块：*vlc* 和 *pygame*。同样，你只需要使其中一个模块在本书中正常工作即可。

本书使用了两种类型的模块。阻塞型是最常见的，我们用它来播放大多数脚本中的音频文件。非阻塞型在音频文件较长时很有用，你可以在播放时选择暂停或停止。我们将在第六章的脚本 *news_brief_hs.py* 以及第十七章最终的 VPA 脚本 *vpa_final.py* 中使用这一功能。

接下来，我们将讨论如何在不同的操作系统中安装这四个模块。

## 安装 playsound 模块

*playsound* 模块使用简单，因为所需的代码行数很少。然而，在 Mac 或 Linux 上的安装可能会有些困难，尽管我已经在所有三个操作系统上成功安装了它。

### Windows

在 Windows 上安装 *playsound*，在激活 *chatting* 虚拟环境的 Anaconda 提示符中执行以下命令：

```py
**pip install playsound**
```

跟随指示操作。

### Mac

在 Mac 上安装 *playsound*，在激活 *chatting* 虚拟环境的终端中执行以下两条命令：

```py
**pip install playsound**
**conda install -c conda-forge pygobject**
```

跟随指示操作。

### Linux

在 Linux 上安装 *playsound*，在激活 *chatting* 虚拟环境的终端中执行以下三条命令：

```py
**pip install playsound**
**conda install –c conda-forge pygobject**
**conda install gstreamer**
```

跟随指示操作。

## 安装 pydub 模块

*pydub* 模块在 Mac 或 Linux 上安装比较简单。然而，在 Windows 上的安装可能会有些困难，尽管我已经在所有三个操作系统上成功安装了它。要安装 *pydub*，在激活 *chatting* 虚拟环境的 Anaconda 提示符（Windows）或终端（Mac 和 Linux）中执行以下命令：

```py
**conda install –c conda-forge pydub**
**conda install –c conda-forge ffmpeg**
```

## 安装 pygame 模块

由于软件不断更新，安装说明可能会有所变化。如果你遇到问题，建议参考 Pygame 官方网站 [`www.pygame.org/wiki/GettingStarted/`](https://www.pygame.org/wiki/GettingStarted/) 以获取最新的安装说明。

### Windows

在你的 Anaconda 提示符中执行此命令，确保 *chatting* 虚拟环境已激活：

```py
**pip install pygame**
```

按照说明进行操作。

### Mac

最近版本的 macOS 需要安装 Pygame 2。要安装，请在终端中执行以下命令，并确保 *chatting* 虚拟环境已激活：

```py
**pip install pygame==2.0.0**
```

然后按照说明进行操作。

### Linux

在 Linux 中，在终端执行以下三个命令，并确保 *chatting* 虚拟环境已激活：

```py
**sudo apt-get install python3-pip python3-dev**
**sudo pip3 install pygame**
**pip install pygame**
```

## 安装 vlc 模块

对于 *vlc* 模块，无论你使用哪种操作系统，都需要在计算机上安装 VLC 媒体播放器。请访问 VLC 网站 [`www.videolan.org/index.html`](https://www.videolan.org/index.html) 下载并安装该软件。

在 Linux 中，你可以通过在终端执行此命令来安装该应用：

```py
**sudo apt-get install vlc**
```

在激活了 *chatting* 虚拟环境的 Anaconda 提示符（Windows）或终端（Mac 或 Linux）中，运行以下命令安装 Python *vlc* 模块：

```py
**pip install python-vlc** 
```

## 测试四个模块的示例脚本

在本节中，我们为每个四个模块提供一个示例脚本，以测试模块是否成功运行。再次提醒，本书只需要安装 *playsound* 和 *pydub* 中的一个模块，以及 *vlc* 和 *pygame* 中的一个模块。

前往书籍资源下载用于测试的 *hello.mp3* 文件。确保将文件放置在与接下来创建的测试脚本相同的文件夹中。

### playsound 模块

在你的 Spyder 编辑器中输入以下代码行。将脚本保存为 *test_playsound.py* 并运行。或者，你也可以从书籍资源中下载。

```py
**from playsound import playsound**
**playsound("hello.mp3")**
```

如果成功，你应该能听到一个人类声音说：“Hello, how are you?”

### pydub 模块

在你的 Spyder 编辑器中输入以下代码行。将脚本保存为 *test_pydub.py* 并运行。或者，你也可以从书籍资源中下载。

```py
**from pydub import AudioSegment**
**from pydub.playback import play**
**play(AudioSegment.from_mp3("hello.mp3")**
```

如果成功，你应该能听到一个人类声音说：“Hello, how are you?”

### pygame 模块

在你的 Spyder 编辑器中输入以下代码行。将脚本保存为 *test_pygame.py* 并运行。或者，你也可以从书籍资源中下载。

```py
**from pygame import mixer**
**mixer.init()**
**mixer.music.load("hello.mp3")**
**mixer.music.play()**
```

如果成功，你应该能听到一个人类声音说：“Hello, how are you?”

### vlc 模块

在你的 Spyder 编辑器中输入以下代码行。将脚本保存为 *test_vlc.py* 并运行。或者，你也可以从书籍资源中下载。

```py
**from vlc import MediaPlayer**
**player=MediaPlayer("hello.mp3")**
**player.play()**
```

如果成功，你应该能听到一个人类声音说：“Hello, how are you?”
