# [A](nsp-venkitachalam503045-0008.xhtml#rappa)

# Python 安装

![](images/nsp-venkitachalam503045-circle-image.jpg)

本附录涵盖了如何安装 Python，以及书中使用的外部模块和代码。使用 Raspberry Pi 进行硬件项目的安装在[附录 B](nsp-venkitachalam503045-0032.xhtml#appb)中进行了说明。本书中的项目已使用 Python 3.9 进行测试。

## [为本书项目安装源代码](nsp-venkitachalam503045-0008.xhtml#rah1801)

你可以从 [https://github.com/mkvenkit/pp2e](https://github.com/mkvenkit/pp2e) 下载本书项目的源代码。使用该网站的 "Download ZIP" 选项来获取代码。

下载并解压代码后，你需要将下载的代码中的 *common* 文件夹路径（通常是 *pp-master/common/*）添加到 `PYTHONPATH` 环境变量中，这样模块才能找到并使用这些 Python 文件。

在 Windows 上，你可以通过创建一个 `PYTHONPATH` 环境变量或在已存在的变量中添加来实现此操作。在 macOS 上，你可以将这一行添加到你的家目录中的 *.profile* 文件（如果需要，可以在该目录下创建该文件）：

export PYTHONPATH=$PYTHONPATH:`path_to_common_folder`

根据需要填写 *common* 文件夹的路径。

Linux 用户可以像 macOS 一样，在适当的 *.bashrc*、*.bash_profile* 或 *.cshrc/.login* 文件中进行类似的操作。使用 `echo $SHELL` 命令查看默认的 shell。

## [安装 Python 和 Python 模块](nsp-venkitachalam503045-0008.xhtml#rah1802)

我推荐使用 Anaconda 发行版的 Python 来运行本书的项目，因为它已经包含了你将需要的大多数 Python 模块。本节介绍了在 Windows、macOS 和 Linux 上的安装过程。

### [Windows](nsp-venkitachalam503045-0008.xhtml#rbh1801)

访问 [https://www.anaconda.com](https://www.anaconda.com) 并下载适用于 Windows 的 Anaconda 发行版。安装完成后，启动 Anaconda 提示符（在搜索栏中输入 **Anaconda prompt**），你将使用它来运行你的程序。只需 `cd` 到本书的代码目录，你就准备好了。

将 Anaconda 及其支持文件的位置添加到你的 `Path` 环境变量中也是非常有用的（在搜索栏中输入 **Edit Environment Variables**）：

`C:\Users\mahes\anaconda3`

`C:\Users\mahes\anaconda3`\Scripts

`C:\Users\mahes\anaconda3`\Library\bin

在我的案例中，我的 Anaconda 安装目录是 `C:\Users\mahes\anaconda3\`。根据需要进行修改。

#### 安装 GLFW

对于本书中基于OpenGL的3D图形项目，你需要GLFW库，可以在[http://www.glfw.org](http://www.glfw.org)下载。在Windows上，安装GLFW后，设置一个`GLFW_LIBRARY`环境变量（在搜索栏中输入**编辑环境变量**）为已安装的*glfw3.dll*的完整路径，以便你的Python绑定能够找到这个库。路径大致如下：*C:\glfw-3.0.4.bin.WIN32\lib-msvc120\glfw3.dll*。

为了在Python中使用GLFW，你需要使用一个名为`pyglfw`的模块，它由一个名为*glfw.py*的Python文件组成。你不需要安装`pyglfw`，因为它随书中的源代码一起提供；它位于*common*目录下。万一你需要安装更新的版本，源代码可以在这里找到：[https://github.com/rougier/pyglfw](https://github.com/rougier/pyglfw)。

你还需要确保计算机上已安装图形卡驱动程序。一般来说，这是一件好事，因为许多程序（尤其是游戏）都利用图形处理单元（GPU）。

#### 安装额外模块

你需要安装一些额外的模块，这些模块不属于标准的Anaconda发行版。请在Anaconda提示符下运行以下命令：

`conda install -c anaconda pyaudio`

`conda install -c anaconda pyopengl`

### [macOS](nsp-venkitachalam503045-0008.xhtml#rbh1802)

访问[https://www.anaconda.com](https://www.anaconda.com)并下载适用于macOS的Anaconda发行版。安装完成后，打开终端应用窗口并输入`which python`。输出应该指向Anaconda Python的版本。如果不是，请手动将路径添加到你的*.profile*文件中。例如：

export PYTHONPATH=`your_anaconda_install_dir_path`:$PYTHONPATH

根据需要填写你Anaconda安装目录的路径。

#### 安装GLFW

对于本书中基于OpenGL的3D图形项目，你需要GLFW库，可以在[http://www.glfw.org](http://www.glfw.org)下载。选择macOS **预编译二进制文件**选项，并将下载的文件夹复制到你的*Home*文件夹中。例如，在我的情况下，我的*Home*文件夹是*/Users/mahesh/*。

现在，你需要将以下内容添加到你的*.profile*中，并根据需要更改路径：

export GLFW_LIBRARY=/`Users/mahesh`/glfw-3.3.8.bin.MACOS/lib-universal/libglfw.3.dylib

当你首次尝试运行一个使用GLFW的程序时，可能会收到安全警告。你需要在系统偏好设置中的安全性选项中允许该程序运行。

#### 安装额外模块

接下来，你需要安装一些额外的模块，这些模块不属于标准的Anaconda发行版。在终端窗口中运行以下命令：

`conda install -c anaconda pyaudio`

`conda install -c anaconda pyopengl`

### [Linux](nsp-venkitachalam503045-0008.xhtml#rbh1803)

Linux 通常自带 Python 以及构建所需包的所有开发工具，因此你无需安装 Anaconda Python。在大多数 Linux 发行版上，你应该能够使用`pip3`来获取书中所需的包。你可以通过以下方式使用 `pip3` 安装包：

sudo pip3 install matplotlib

安装包的另一种方式是下载该模块的源代码分发包，通常是*.gz*或*.zip*文件。解压这些文件到一个文件夹后，你可以按照以下方式安装它们：

sudo python setup.py install

对于书中需要的每个包，使用这些方法之一。
