# 第十六章：A

# Python 安装

![](img/nsp-venkitachalam503045-circle-image.jpg)

本附录介绍了如何安装 Python 以及书中使用的外部模块和代码。使用 Raspberry Pi 进行硬件项目的安装方法在 附录 B 中有介绍。本书中的项目已在 Python 3.9 环境下进行测试。

## 安装书中的项目源代码

你可以从 [`github.com/mkvenkit/pp2e`](https://github.com/mkvenkit/pp2e) 下载书中项目的源代码。在该网站上使用“Download ZIP”选项来获取代码。

一旦下载并解压代码，你需要将下载代码中的 *common* 文件夹路径（通常是 *pp-master/common/*）添加到 `PYTHONPATH` 环境变量中，这样模块才能找到并使用这些 Python 文件。

在 Windows 上，你可以通过创建一个 `PYTHONPATH` 环境变量，或者如果已经存在该变量，则进行添加。在 macOS 上，你可以将这一行添加到主目录下的 *.profile* 文件中（如果需要的话，可以在该目录下创建该文件）：

```py
export PYTHONPATH=$PYTHONPATH:`path_to_common_folder`

```

根据需要填写公共文件夹的路径。

Linux 用户可以在 *.bashrc*、*.bash_profile* 或 *.cshrc/.login* 文件中进行类似的操作，具体取决于使用的环境。使用 `echo $SHELL` 命令来查看默认的 shell。

## 安装 Python 和 Python 模块

我建议使用 Anaconda 发行版的 Python 来运行本书的项目，因为它已经包含了你所需的大部分 Python 模块。本节将介绍 Windows、macOS 和 Linux 上的安装过程。

### Windows

访问 [`www.anaconda.com`](https://www.anaconda.com) 下载适用于 Windows 的 Anaconda 发行版。安装完成后，打开 Anaconda 提示符（在搜索栏中输入 **Anaconda prompt**），你将使用它来运行你的程序。只需 `cd` 进入书中的代码目录，准备好就可以开始了。

同样，将 Anaconda 及其支持文件的位置添加到 `Path` 环境变量中也是很有用的（在搜索栏中输入 **Edit Environment Variables**）：

```py
`C:\Users\mahes\anaconda3`
`C:\Users\mahes\anaconda3`\Scripts
`C:\Users\mahes\anaconda3`\Library\bin

```

在这个例子中，我的 Anaconda 安装目录是 `C:\Users\mahes\anaconda3\`。根据需要修改此路径。

#### 安装 GLFW

对于本书中的基于 OpenGL 的 3D 图形项目，你需要安装 GLFW 库，可以从 [`www.glfw.org`](http://www.glfw.org) 下载。在 Windows 上，安装 GLFW 后，设置一个 `GLFW_LIBRARY` 环境变量（在搜索栏中输入 **Edit Environment Variables**），其值为已安装的 *glfw3.dll* 文件的完整路径，这样 Python 绑定的 GLFW 就能找到这个库。路径会像 *C:\glfw-3.0.4.bin.WIN32\lib-msvc120\glfw3.dll* 这样的格式。

要在 Python 中使用 GLFW，你需要一个名为`pyglfw`的模块，它由一个名为*glfw.py*的 Python 文件组成。你不需要安装`pyglfw`，因为它随书中的源代码一起提供，位于*common*目录中。万一你需要安装一个更新版本，这里是源代码：[`github.com/rougier/pyglfw`](https://github.com/rougier/pyglfw)。

你还需要确保你的显卡驱动程序已安装在计算机上。这通常是件好事，因为许多程序（特别是游戏）都会使用图形处理单元（GPU）。

#### 安装附加模块

你需要安装一些不包含在标准 Anaconda 发行版中的附加模块。请在 Anaconda 提示符下运行以下命令：

```py
`conda install -c anaconda pyaudio`
`conda install -c anaconda pyopengl`

```

### macOS

访问[`www.anaconda.com`](https://www.anaconda.com)并下载适用于 macOS 的 Anaconda 发行版。安装完成后，打开一个终端应用程序窗口并输入`which python`。输出应该指向 Anaconda Python 的版本。如果没有，手动将路径添加到你的*.profile*文件中。例如：

```py
export PYTHONPATH=`your_anaconda_install_dir_path`:$PYTHONPATH

```

根据需要填写你的 Anaconda 安装目录路径。

#### 安装 GLFW

对于本书中的基于 OpenGL 的 3D 图形项目，你需要 GLFW 库，可以在[`www.glfw.org`](http://www.glfw.org)下载。选择 macOS **预编译二进制**选项，并将下载的文件夹复制到你的*Home*文件夹中。例如，在我的例子中，我的*Home*文件夹是*/Users/mahesh/*。

现在你需要将以下内容添加到你的*.profile*文件中，并根据需要修改路径：

export GLFW_LIBRARY=/`Users/mahesh`/glfw-3.3.8.bin.MACOS/lib-universal/libglfw.3.dylib

当你第一次尝试运行使用 GLFW 的程序时，可能会收到安全警告。你需要在“系统偏好设置”中的“安全性”下允许它运行。

#### 安装附加模块

接下来，你需要安装一些不包含在标准 Anaconda 发行版中的附加模块。在终端窗口中，运行以下命令：

```py
`conda install -c anaconda pyaudio`
`conda install -c anaconda pyopengl`

```

### Linux

Linux 通常自带 Python 以及构建所需包的所有开发工具。因此，你不需要安装 Anaconda Python。在大多数 Linux 发行版中，你应该能够使用`pip3`来获取书中所需的包。你可以像这样使用`pip3`安装包：

```py
sudo pip3 install matplotlib

```

安装包的另一种方法是下载该模块的源代码发行版，通常是*.gz*或*.zip*文件。将这些文件解压到一个文件夹中后，你可以按以下方式安装它们：

```py
sudo python setup.py install

```

对于书中需要的每个包，使用以下方法之一进行安装。
