# 第一部分

基础知识

本书的第一部分将教授你编写Python程序所需的基本概念。这些概念在所有编程语言中都是通用的，因此它们将在你作为程序员的一生中始终有用。

在**第1章**中，你将安装Python，并运行你的第一个程序，它会将*Hello world!*消息打印到屏幕上。

在**第2章**中，你将学习如何将信息赋值给变量，并处理文本和数值数据。

**第3章**和**第4章**介绍了列表。列表可以将你想要的所有信息存储在一个地方，使你能够高效地处理这些数据。你将能够在几行代码中处理数百、数千甚至数百万个值。

在**第5章**中，你将使用`if`语句编写代码，当某些条件为真时以一种方式响应，当这些条件不为真时则以另一种方式响应。

**第6章**展示了如何使用Python的字典，它允许你在不同的信息之间建立联系。像列表一样，字典可以存储你需要的所有信息。

在**第7章**中，你将学习如何接受用户输入，使你的程序变得互动。你还将学习`while`循环，它会在某些条件为真时反复执行代码块。

在**第8章**中，你将编写函数，函数是执行特定任务的命名代码块，可以在需要时运行。

**第9章**介绍了类，它允许你模拟现实世界中的对象。你将编写代码来表示狗、猫、人、车、火箭等等。

**第10章**展示了如何处理文件和错误，以防止程序意外崩溃。你将学会在程序关闭前保存数据，并在程序再次运行时读取数据。你将了解Python的异常，它们让你能够预见错误并使程序优雅地处理这些错误。

在**第11章**中，你将学习如何为代码编写测试，以确保程序按预期工作。因此，你可以在不担心引入新错误的情况下扩展程序。编写测试是帮助你从初学者过渡到中级程序员的第一项技能。

# 1

入门

![](Images/chapterart.png)

在这一章中，你将运行你的第一个Python程序，*hello_world.py*。首先，你需要检查计算机上是否安装了最新版本的Python；如果没有，你将安装它。你还将安装一个文本编辑器来处理你的Python程序。文本编辑器会识别Python代码，并在你编写时突出显示代码部分，使你更容易理解代码的结构。

## 设置你的编程环境

Python在不同的操作系统上略有不同，因此你需要记住一些注意事项。在接下来的章节中，我们将确保Python在你的系统上正确设置。

### Python版本

每种编程语言都会随着新想法和新技术的出现而不断发展，Python的开发者们也不断让这门语言变得更加多功能和强大。截至目前，本书的内容应该可以在Python 3.9或更高版本上运行，但目前最新版本是Python 3.11。在本节中，我们将检查你的系统是否已经安装了Python，以及是否需要安装更新的版本。附录A还包含了在各大操作系统上安装最新版本Python的详细信息。

### 运行Python代码片段

你可以在终端窗口中运行Python的解释器，允许你尝试Python代码的片段，而不必保存并运行整个程序。

在本书中，你将看到如下所示的代码片段：

```py
>>> **print("Hello Python interpreter!")**
Hello Python interpreter!
```

三个尖括号（`>>>`）提示符，我们称之为*Python提示符*，表示你应该使用终端窗口。粗体文本是你应该输入并通过按下ENTER键执行的代码。本书中的大部分示例是小型、独立的程序，你将在文本编辑器中运行它们，而不是通过终端，因为你将大部分代码编写在文本编辑器中。但有时，基本概念会通过一系列在Python终端会话中运行的代码片段来展示，以便更高效地演示特定概念。当你在代码列表中看到三个尖括号时，你正在查看来自终端会话的代码和输出。稍后我们将尝试在你的系统中运行解释器进行编码。

我们还将使用文本编辑器创建一个简单的程序，名为*Hello World!*，它已经成为学习编程的必备程序。在编程界有一个长期的传统，认为将`Hello world!`消息输出到屏幕作为新语言的第一个程序，会带来好运。如此简单的程序有着非常实际的用途。如果它在你的系统上正确运行，那么你编写的任何Python程序也应该能够正常运行。

### 关于VS Code编辑器

*VS Code* 是一个功能强大、专业级的文本编辑器，免费且适合初学者。VS Code非常适合简单和复杂的项目，因此如果你在学习Python的过程中熟悉使用它，你可以继续使用它来处理更大更复杂的项目。VS Code可以安装在所有现代操作系统上，并且支持包括Python在内的大多数编程语言。

附录B提供了关于其他文本编辑器的信息。如果你对其他选项感兴趣，可以在此时浏览该附录。如果你想快速开始编程，可以使用VS Code来启动。然后，在你积累了一些编程经验后，你可以考虑使用其他编辑器。在这一章中，我将带你完成在你的操作系统上安装VS Code的过程。

## 不同操作系统上的Python

Python 是一种跨平台编程语言，这意味着它可以在所有主流操作系统上运行。你编写的任何 Python 程序应该可以在任何安装了 Python 的现代计算机上运行。然而，在不同操作系统上设置 Python 的方法略有不同。

在这一部分中，你将学习如何在你的系统上设置 Python。你将首先检查是否安装了最新版本的 Python，如果没有安装，则进行安装。然后，你将安装 VS Code。这是每个操作系统中唯一不同的两步。

在接下来的章节中，你将运行 *hello_world.py* 并解决任何无法正常工作的部分。我将为每个操作系统带你完成这个过程，这样你就能拥有一个可靠的 Python 编程环境。

### Windows 上的 Python

Windows 通常不会预装 Python，因此你可能需要先安装它，然后再安装 VS Code。

#### 安装 Python

首先，检查你的系统是否已安装 Python。通过在开始菜单中输入 `command` 并点击 **命令提示符** 应用来打开命令窗口。在终端窗口中，输入小写的 `python`。如果你看到 Python 提示符（`>>>`），则表示 Python 已安装在你的系统中。如果你看到错误信息，告诉你 `python` 不是一个已识别的命令，或者如果 Microsoft Store 被打开，那么 Python 就没有安装。如果 Microsoft Store 打开，关闭它；最好从官方渠道下载安装程序，而不是使用 Microsoft 版本。

如果你的系统上没有安装 Python，或者你看到的版本早于 Python 3.9，那么你需要下载 Windows 版本的 Python 安装程序。访问 [https://python.org](https://python.org)，将鼠标悬停在 **Downloads** 链接上。你应该看到一个按钮，点击该按钮即可自动下载适合你系统的最新版本的 Python 安装程序。下载完成后，运行安装程序。确保你选择了 **Add Python to PATH** 选项，这将使配置系统更加方便。[图 1-1](#figure1-1) 显示了该选项已被选中。

![](Images/f01001.png)

图 1-1：确保你勾选了标有 *Add Python to PATH* 的复选框。

#### 在终端会话中运行 Python

打开一个新的命令窗口并输入小写的 `python`。你应该看到 Python 提示符（`>>>`），这意味着 Windows 找到了你刚安装的 Python 版本。

```py
C:\> **python**
Python 3.`x`.`x` (main, Jun . . . , 13:29:14) [MSC v.1932 64 bit (AMD64)] on win32
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

在你的 Python 会话中输入以下命令：

```py
>>> **print("Hello Python interpreter!")**
Hello Python interpreter!
>>>
```

你应该看到输出 `Hello Python interpreter!`。每当你想运行一段 Python 代码时，只需打开命令窗口并启动一个 Python 终端会话。要关闭终端会话，按 CTRL-Z 然后按 ENTER，或者输入命令 `exit()`。

#### 安装 VS Code

你可以在 [https://code.visualstudio.com](https://code.visualstudio.com) 下载 VS Code 安装程序。点击 **Download for Windows** 按钮并运行安装程序。跳过接下来的 macOS 和 Linux 部分，按照第 9 页的“运行 Hello World 程序”步骤操作。

### macOS 上的 Python

在最新版本的 macOS 上，Python 默认没有安装，因此如果你还没有安装，你需要进行安装。在本节中，你将安装最新版本的 Python，然后安装 VS Code 并确保其正确配置。

#### 检查是否安装了 Python 3

通过访问 **应用程序**▶**实用工具**▶**终端** 来打开终端窗口。你也可以按 ⌘-空格键，输入 `terminal`，然后按回车键。要检查是否安装了足够新的 Python 版本，输入 `python3`。你很可能会看到一条关于安装 *命令行开发工具* 的消息。最好在安装 Python 后再安装这些工具，所以如果出现此消息，请取消弹出的窗口。

如果输出显示你安装了 Python 3.9 或更高版本，可以跳过下一节，直接进入“在终端会话中运行 Python”。如果你看到任何早于 Python 3.9 的版本，请按照下一节的说明安装最新版本。

请注意，在 macOS 上，每当你在本书中看到 `python` 命令时，你需要使用 `python3` 命令来确保你正在使用 Python 3。大多数 macOS 系统中，`python` 命令要么指向一个过时的 Python 版本，只应由内部系统工具使用，要么指向空白并产生错误信息。

#### 安装最新版本的 Python

你可以在 [https://python.org](https://python.org) 找到适用于你的系统的 Python 安装程序。将鼠标悬停在 **Download** 链接上，你应该会看到一个下载最新版本 Python 的按钮。点击该按钮，应该会自动开始下载适用于你系统的正确安装程序。文件下载后，运行该安装程序。

安装程序运行后，应该会出现一个 Finder 窗口。双击 *Install Certificates.command* 文件。运行此文件将帮助你更轻松地安装你在现实项目中需要的附加库，包括本书后半部分的项目。

#### 在终端会话中运行 Python

你现在可以通过打开一个新的终端窗口并输入 `python3` 来尝试运行 Python 代码片段：

```py
$ **python3**
Python 3.`x`.`x` (v3.11.0:eb0004c271, Jun . . . , 10:03:01)
[Clang 13.0.0 (clang-1300.0.29.30)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

该命令会启动 Python 终端会话。你应该会看到 Python 提示符（`>>>`），这意味着 macOS 已经找到了你刚刚安装的 Python 版本。

在终端会话中输入以下命令：

```py
>>> **print("Hello Python interpreter!")**
Hello Python interpreter!
>>>
```

你应该会看到消息 `Hello Python interpreter!`，它应该会直接在当前终端窗口中打印出来。你可以通过按 CTRL-D 或输入命令 `exit()` 来关闭 Python 解释器。

#### 安装 VS Code

要安装 VS Code 编辑器，你需要在 [https://code.visualstudio.com](https://code.visualstudio.com) 下载安装程序。点击**下载**按钮，然后打开**Finder**窗口，进入**下载**文件夹。将**Visual Studio Code**安装程序拖到应用程序文件夹，然后双击安装程序运行。

跳过接下来的关于 Linux 上 Python 的部分，按照第 9 页“运行 Hello World 程序”中的步骤进行操作。

### Linux 上的 Python

Linux 系统专为编程设计，因此大多数 Linux 计算机上已经安装了 Python。编写和维护 Linux 的人期望你在某个时候自己进行编程，并鼓励你这样做。因此，开始编程时几乎没有需要安装的内容，只有少量的设置需要更改。

#### 检查你的 Python 版本

通过运行系统上的终端应用程序打开终端窗口（在 Ubuntu 中，你可以按 CTRL-ALT-T）。要查找安装的 Python 版本，输入小写的`python3`。当 Python 安装完成时，这个命令会启动 Python 解释器。你应该能看到显示已安装 Python 版本的输出。你还应该看到一个 Python 提示符（`>>>`），在这里你可以开始输入 Python 命令：

```py
$ **python3**
Python 3.10.4 (main, Apr  . . . , 09:04:19) [GCC 11.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

这个输出表示 Python 3.10.4 是当前安装在这台计算机上的默认 Python 版本。当你看到这个输出时，按 CTRL-D 或输入`exit()`以退出 Python 提示符并返回终端提示符。每当你在本书中看到`python`命令时，应该输入`python3`代替。

你需要 Python 3.9 或更高版本才能运行本书中的代码。如果你系统上安装的 Python 版本低于 3.9，或者如果你想更新到当前可用的最新版本，请参考附录 A 中的说明。

#### 在终端会话中运行 Python

你可以通过打开终端并输入`python3`来尝试运行 Python 代码片段，正如你在检查版本时所做的那样。再次执行此操作，当 Python 正在运行时，在终端会话中输入以下行：

```py
>>> **print("Hello Python interpreter!")**
Hello Python interpreter!
>>>
```

消息应该直接显示在当前的终端窗口中。记住，你可以通过按 CTRL-D 或输入命令`exit()`来关闭 Python 解释器。

#### 安装 VS Code

在 Ubuntu Linux 上，你可以通过 Ubuntu 软件中心安装 VS Code。点击菜单中的 Ubuntu 软件图标，然后搜索*vscode*。点击名为**Visual Studio Code**的应用（有时也叫*code*），然后点击**安装**。安装完成后，搜索你的系统，找到*VS Code*，然后启动该应用。

## 运行 Hello World 程序

在安装了最新版本的 Python 和 VS Code 后，你几乎准备好在文本编辑器中运行你的第一个 Python 程序了。但在此之前，你需要为 VS Code 安装 Python 扩展。

### 为 VS Code 安装 Python 扩展

VS Code 支持多种编程语言；作为 Python 开发者，想要最大限度地发挥它的作用，你需要安装 Python 扩展。这个扩展提供了编写、编辑和运行 Python 程序的支持。

要安装 Python 扩展，点击 VS Code 应用程序左下角的齿轮图标。在出现的菜单中，点击 **扩展**。在搜索框中输入 `python`，然后点击 **Python** 扩展。（如果你看到多个名为 *Python* 的扩展，请选择由 Microsoft 提供的那个。）点击 **安装**，并安装任何你的系统需要的附加工具来完成安装。如果看到提示需要安装 Python，而你已经安装了，可以忽略此消息。

### 运行 hello_world.py

在编写第一个程序之前，在桌面上创建一个名为 *python_work* 的文件夹来存放你的项目。最好在文件和文件夹名称中使用小写字母和下划线代替空格，因为 Python 使用这种命名惯例。你可以在桌面以外的地方创建此文件夹，但如果直接将 *python_work* 文件夹保存在桌面上，之后的一些步骤会更容易跟随。

打开 VS Code，如果 **Get Started** 标签页还未关闭，请关闭它。通过点击 **文件**▶**新建文件**，或者按 CTRL-N（在 macOS 上为 ⌘-N）来创建一个新文件。将文件保存为 *hello_world.py*，保存在你的 *python_work* 文件夹中。扩展名 *.py* 告诉 VS Code 你的文件是用 Python 编写的，并且告诉它如何运行程序并以有帮助的方式高亮显示文本。

保存文件后，在编辑器中输入以下内容：

**hello_world.py**

```py
print("Hello Python world!")
```

要运行程序，选择 **运行**▶**无调试运行**，或者按 CTRL-F5。VS Code 窗口底部应该出现一个终端屏幕，显示你程序的输出：

```py
Hello Python world!
```

你可能会看到一些额外的输出，显示用于运行程序的 Python 解释器。如果你想简化显示的信息，只看到程序的输出，请参阅附录 B。附录 B 中也提供了关于如何更高效使用 VS Code 的一些有用建议。

如果你没有看到这个输出，可能程序出了问题。检查你输入的每个字符。你是否不小心把 `print` 大小写写错了？你是否忘记了一个或两个引号或括号？编程语言要求非常具体的语法，如果你没有遵循，程序就会出错。如果你无法运行程序，请参阅下一节中的建议。

## 故障排除

如果无法运行 *hello_world.py*，这里有一些可以尝试的解决办法，它们也是解决任何编程问题的常见方法：

+   当程序包含重大错误时，Python会显示一个*traceback*（回溯），这是一个错误报告。Python会浏览文件并尝试识别问题。检查回溯，它可能会给你一个线索，指示是什么问题导致程序无法运行。

+   离开电脑，休息一下，然后再试一次。记住，编程中语法非常重要，因此像引号不匹配或括号不匹配这样的简单问题就可能导致程序无法正常运行。重新阅读本章相关部分，检查你的代码，试着找到问题所在。

+   从头开始。你可能不需要卸载任何软件，但删除你的*hello_world.py*文件并从头创建它可能更有意义。

+   请别人按照本章中的步骤在你的电脑或其他电脑上操作，仔细观察他们的操作。你可能会错过一个小步骤，而别人恰好会发现。

+   请参阅附录A中的额外安装说明；附录中包含的一些细节可能会帮助你解决问题。

+   找一个懂Python的人，向他们求助帮助你完成设置。如果你问问周围的人，你可能会意外地发现有认识的人会使用Python。

+   本章中的设置说明也可以通过本书的伴随网站访问：[https://ehmatthes.github.io/pcc_3e](https://ehmatthes.github.io/pcc_3e)。在线版本的这些说明可能会更有效，因为你可以直接剪切粘贴代码并点击链接获取所需资源。

+   在线寻求帮助。附录C提供了许多资源，例如论坛和在线聊天网站，你可以在这些地方向已经解决过你当前问题的人寻求解决方案。

永远不要担心打扰经验丰富的程序员。每个程序员都曾经卡在某个点上，而且大多数程序员都很乐意帮助你正确设置系统。只要你能清楚地说明你在做什么，已经尝试了哪些方法，得到了什么结果，那么很有可能有人能够帮助你。正如在导言中提到的，Python社区对初学者非常友好和欢迎。

Python应该能在任何现代计算机上良好运行。最初的设置问题可能令人沮丧，但完全值得解决。一旦你成功运行*hello_world.py*，你就可以开始学习Python，并且你的编程工作将变得更加有趣和令人满足。

## 从终端运行Python程序

你将大部分程序直接在文本编辑器中运行。但是，有时从终端运行程序更有用。例如，你可能想运行一个现有的程序，而无需打开它进行编辑。

只要你知道如何访问存储程序文件的目录，你可以在任何安装了Python的系统上进行此操作。为了尝试这个，确保你已将*hello_world.py*文件保存在桌面上的*python_work*文件夹中。

### 在Windows上

你可以使用终端命令`cd`，即*切换目录*，在命令窗口中浏览你的文件系统。命令`dir`，即*目录*，会显示当前目录下所有存在的文件。

打开一个新的终端窗口，输入以下命令来运行*hello_world.py*：

```py
C:\> **cd Desktop\python_work**
C:\Desktop\python_work> **dir**
hello_world.py
C:\Desktop\python_work> **python hello_world.py**
Hello Python world!
```

首先，使用`cd`命令导航到*python_work*文件夹，该文件夹位于*Desktop*文件夹中。接下来，使用`dir`命令确认*hello_world.py*在该文件夹中。然后，使用命令`python hello_world.py`来运行该文件。

大多数程序可以直接从编辑器运行。然而，随着工作变得更加复杂，你可能会希望从终端运行一些程序。

### 在macOS和Linux上

在终端会话中运行Python程序在Linux和macOS上是相同的。你可以使用终端命令`cd`，即*切换目录*，在终端会话中浏览你的文件系统。命令`ls`，即*列出*，会显示当前目录下所有非隐藏的文件。

打开一个新的终端窗口，输入以下命令来运行*hello_world.py*：

```py
~$ **cd Desktop/python_work/**
~/Desktop/python_work$ **ls**
hello_world.py
~/Desktop/python_work$ **python3 hello_world.py**
Hello Python world!
```

首先，使用`cd`命令导航到*python_work*文件夹，该文件夹位于*Desktop*文件夹中。接下来，使用`ls`命令确认*hello_world.py*在该文件夹中。然后，使用命令`python3 hello_world.py`来运行该文件。

大多数程序可以直接从编辑器运行。但随着工作变得更加复杂，你可能会希望从终端运行一些程序。

## 小结

在本章中，你了解了Python的基本知识，如果你的系统尚未安装Python，你也安装了它。你还安装了一个文本编辑器，以便更轻松地编写Python代码。你在终端会话中运行了Python代码片段，并运行了第一个程序*hello_world.py*。你可能还学到了一些故障排除的知识。

在下一章中，你将学习可以在Python程序中使用的不同数据类型，并且你也将开始使用变量。

# 2

变量和简单数据类型

![](Images/chapterart.png)

在本章中，你将学习到可以在Python程序中使用的各种数据类型。你还将学习如何使用变量来表示程序中的数据。

## 当你运行`hello_world.py`时，会发生什么

让我们仔细看看，当你运行*hello_world.py*时，Python到底做了什么。事实证明，即使是运行一个简单的程序，Python也要做相当多的工作：

**hello_world.py**

```py
print("Hello Python world!")
```

当你运行这段代码时，你应该会看到以下输出：

```py
Hello Python world!
```

当你运行文件*hello_world.py*时，文件名以*.py*结尾，表示该文件是一个Python程序。然后，编辑器通过*Python解释器*运行该文件，解释器逐行读取程序并确定每个单词的含义。例如，当解释器看到`print`后跟括号时，它会将括号内的内容打印到屏幕上。

在编写程序时，编辑器会以不同的方式突出显示程序的不同部分。例如，它会识别 `print()` 是一个函数的名称，并将该词以一种颜色显示。它会识别 `"Hello Python world!"` 不是 Python 代码，并将该短语以不同的颜色显示。此功能称为 *语法高亮*，当你开始编写自己的程序时非常有用。

## 变量

让我们尝试在 *hello_world.py* 中使用一个变量。为文件开头添加一行，并修改第二行：

**hello_world.py**

```py
message = "Hello Python world!"
print(message)
```

运行这个程序看看会发生什么。你应该看到与之前相同的输出：

```py
Hello Python world!
```

我们已经添加了一个名为 *message* 的变量。每个变量都与一个 *值* 相关联，这个值就是与该变量相关的信息。在此情况下，值为 `"Hello Python world!"` 文本。

添加变量会稍微增加 Python 解释器的工作量。当它处理第一行时，会将变量 `message` 与 `"Hello Python world!"` 文本关联。当它处理到第二行时，会将与 `message` 相关联的值打印到屏幕上。

让我们通过修改 *hello_world.py* 来扩展这个程序，使其打印第二条消息。为 *hello_world.py* 添加一个空行，然后再添加两行新代码：

```py
message = "Hello Python world!"
print(message)

message = "Hello Python Crash Course world!"
print(message)
```

现在，当你运行 *hello_world.py* 时，你应该会看到两行输出：

```py
Hello Python world!
Hello Python Crash Course world!
```

你可以随时更改程序中变量的值，Python 会始终跟踪其当前值。

### 命名和使用变量

在使用 Python 变量时，你需要遵守一些规则和准则。违反其中某些规则会导致错误；其他准则则有助于你编写更易于阅读和理解的代码。编写变量时请务必记住以下规则：

+   变量名只能包含字母、数字和下划线。它们可以以字母或下划线开头，但不能以数字开头。例如，你可以将变量命名为 `message_1`，但不能命名为 `1_message`。

+   变量名中不允许有空格，但可以使用下划线分隔变量名中的单词。例如，`greeting_message` 可行，但 `greeting message` 会导致错误。

+   避免使用 Python 的关键字和函数名作为变量名。例如，不能使用 `print` 作为变量名；Python 已经将其保留为特定的程序用途。（请参阅第466页的“Python关键字和内置函数”）

+   变量名应该简洁且具有描述性。例如，`name` 比 `n` 更好，`student_name` 比 `s_n` 更好，`name_length` 比 `length_of_persons_name` 更好。

+   在使用小写字母 *l* 和大写字母 *O* 时要小心，因为它们可能会与数字 *1* 和 *0* 混淆。

学习如何创建好的变量名可能需要一些练习，尤其是当你的程序变得越来越有趣和复杂时。随着你写更多的程序，并开始阅读其他人的代码，你会越来越擅长为变量命名并赋予它们有意义的名字。

### 避免在使用变量时出现名称错误

每个程序员都会犯错误，大多数程序员每天都会犯错误。虽然优秀的程序员也可能会产生错误，但他们知道如何高效地应对这些错误。让我们看看你可能会在早期犯的一个错误，并学习如何修复它。

我们将编写一些故意生成错误的代码。输入以下代码，其中包含拼写错误的单词`mesage`*，*该单词用粗体显示：

```py
message = "Hello Python Crash Course reader!"
print(**mesage**)
```

当程序出现错误时，Python 解释器会尽力帮助你找出问题所在。如果程序无法成功运行，解释器会提供回溯信息。*回溯*是记录解释器在执行代码时遇到问题的位置。以下是 Python 提供的回溯示例，当你不小心拼写错了变量名时：

```py
Traceback (most recent call last):
❶   File "hello_world.py", line 2, in <module>
❷     print(mesage)
          ^^^^^^
❸ NameError: name 'mesage' is not defined. Did you mean: 'message'?
```

输出报告了文件 *hello_world.py* 第 2 行发生了错误 ❶。解释器显示了这一行 ❷，帮助我们快速找到错误，并告诉我们它发现了什么类型的错误 ❸。在这个例子中，它发现了一个 *名称错误*，并报告说正在打印的变量 `mesage` 尚未定义。Python 无法识别提供的变量名。名称错误通常意味着我们要么忘记在使用变量之前设置它的值，要么在输入变量名时拼写错误。如果 Python 找到一个与它无法识别的变量名相似的名字，它会询问这是否是你想使用的名称。

在这个例子中，我们在第二行的变量名 `message` 中漏掉了字母 *s*。Python 解释器不会检查你的代码拼写，但它会确保变量名拼写一致。例如，当我们在定义变量的那一行错误拼写 *message* 时，看看会发生什么：

```py
mesage = "Hello Python Crash Course reader!"
print(mesage)
```

在这种情况下，程序成功运行！

```py
Hello Python Crash Course reader!
```

变量名称匹配，因此 Python 没有问题。编程语言是严格的，但它们忽略了拼写的好坏。因此，当你在创建变量名并编写代码时，你不需要考虑英语的拼写和语法规则。

许多编程错误是程序中单行的简单字符错误。如果你发现自己花了很长时间寻找这些错误，知道你并不孤单。许多经验丰富且才华横溢的程序员也会花几个小时寻找这些微小的错误。尽量笑对它，继续前进，知道在你的编程生涯中，这种事情会经常发生。

### 变量是标签

变量通常被描述为可以存储值的盒子。这个概念在你第一次使用变量时可能会有所帮助，但它并不准确地描述了变量在 Python 中是如何表示的。更好的方式是将变量视为你可以分配给值的标签。你也可以说，变量引用了某个特定的值。

这种区分在你最初的程序中可能并不重要，但早点学习总比晚学要好。到某个时候，你可能会遇到变量的意外行为，而准确理解变量的工作原理将帮助你找出代码中的问题。

## 字符串

由于大多数程序会定义和收集某种数据，然后对其执行有用的操作，因此分类不同类型的数据很有帮助。我们将要查看的第一个数据类型是字符串。字符串乍看之下很简单，但你可以以多种方式使用它们。

*字符串* 是一系列字符。Python 中任何引号内的内容都被视为字符串，你可以像这样使用单引号或双引号包围字符串：

```py
"This is a string."
'This is also a string.'
```

这种灵活性允许你在字符串中使用引号和撇号：

```py
'I told my friend, "Python is my favorite language!"'
"The language 'Python' is named after Monty Python, not the snake."
"One of Python's strengths is its diverse and supportive community."
```

让我们探索一下使用字符串的一些方法。

### 使用方法更改字符串的大小写

你可以对字符串执行的最简单操作之一是更改字符串中单词的大小写。看看下面的代码，尝试理解发生了什么：

**name.py**

```py
name = "ada lovelace"
print(name.title())
```

将此文件保存为 *name.py* 然后运行，你应该会看到如下输出：

```py
Ada Lovelace
```

在这个例子中，变量 `name` 引用了小写字符串 `"ada lovelace"`。方法 `title()` 出现在 `print()` 调用中的变量后面。*方法* 是 Python 可以对一块数据执行的操作。`name` 后的点号（`.`）告诉 Python 执行 `title()` 方法对变量 `name` 进行操作。每个方法后面都有一对括号，因为方法通常需要额外的信息来完成工作。这些信息在括号内提供。`title()` 方法不需要额外的信息，因此其括号是空的。

`title()` 方法将每个单词的首字母改为大写，这对于将名字视作信息非常有用。比如，你可能希望你的程序能够识别输入值 `Ada`、`ADA` 和 `ada` 为相同的名字，并将它们都显示为 `Ada`。

还有一些其他有用的方法可以处理大小写。例如，你可以将字符串转换为全大写或全小写字母，像这样：

```py
name = "Ada Lovelace"
print(name.upper())
print(name.lower())
```

这将显示以下内容：

```py
ADA LOVELACE
ada lovelace
```

`lower()` 方法在存储数据时特别有用。你通常不希望完全信任用户提供的大小写，因此你会将字符串转换为小写再存储。然后当你想显示信息时，你会使用最适合每个字符串的大小写形式。

### 在字符串中使用变量

在某些情况下，你可能想在字符串中使用变量的值。例如，你可能希望分别使用两个变量来表示名字和姓氏，然后将这些值组合在一起显示某人的全名：

**full_name.py**

```py
first_name = "ada"
last_name = "lovelace"
❶ full_name = f"{first_name} {last_name}"
print(full_name)
```

要将变量的值插入到字符串中，需在开头的引号前面立即加上字母`f` ❶。将大括号包围你想在字符串中使用的变量名称。Python会在显示字符串时，将每个变量替换为它的值。

这些字符串被称为*f-strings*。*f*代表*format*，因为Python通过将大括号内的变量名替换为其值来格式化字符串。前面代码的输出是：

```py
ada lovelace
```

你可以用f-strings做很多事情。例如，你可以使用f-strings通过与变量相关联的信息来组成完整的消息，如下所示：

```py
first_name = "ada"
last_name = "lovelace"
full_name = f"{first_name} {last_name}"
❶ print(f"Hello, {full_name.title()}!")
```

全名被用在一个句子中来问候用户 ❶，而`title()`方法将名称转换为标题格式。此代码返回一个简单而格式化良好的问候语：

```py
Hello, Ada Lovelace!
```

你也可以使用f-strings来组合一条消息，然后将整个消息赋值给一个变量：

```py
first_name = "ada"
last_name = "lovelace"
full_name = f"{first_name} {last_name}"
❶ message = f"Hello, {full_name.title()}!"
❷ print(message)
```

这段代码同样显示消息`Hello, Ada Lovelace!`，但通过将消息赋值给变量 ❶，我们使得最后的`print()`调用变得更加简洁 ❷。

### 使用制表符或换行符向字符串添加空白

在编程中，*空白字符*指的是任何非打印字符，如空格、制表符和行尾符号。你可以使用空白字符来组织输出，使其更易于用户阅读。

要在文本中添加制表符，使用字符组合`\t`：

```py
>>> **print("Python")**
Python
>>> **print("\tPython")**
    Python
```

要在字符串中添加换行符，使用字符组合`\n`：

```py
>>> **print("Languages:\nPython\nC\nJavaScript")**
Languages:
Python
C
JavaScript
```

你还可以在一个字符串中结合制表符和换行符。字符串`"\n\t"`告诉Python跳到新的一行，并在新的一行开始时插入一个制表符。下面的示例展示了如何用单行字符串生成四行输出：

```py
>>> **print("Languages:\n\tPython\n\tC\n\tJavaScript")**
Languages:
    Python
    C
    JavaScript
```

换行符和制表符将在接下来的两章中非常有用，当你开始用少量代码生成许多行输出时。

### 去除空白字符

额外的空白字符在你的程序中可能会让人困惑。对于程序员来说，`'python'`和`'python '`看起来几乎是一样的。但对于程序来说，它们是两个不同的字符串。Python会检测到`'python '`中的额外空格，并将其视为重要，除非你告诉它不这么做。

思考空白字符非常重要，因为你通常需要比较两个字符串来判断它们是否相同。例如，一个重要的情况可能是检查人们登录网站时的用户名。额外的空白字符在很多简单情况下也会让人困惑。幸运的是，Python让你可以轻松地去除用户输入数据中的额外空白。

Python可以查找字符串左右两侧的额外空白字符。为了确保字符串右侧没有空白字符，可以使用`rstrip()`方法：

```py
❶ >>> **favorite_language = 'python '**
❷ >>> **favorite_language**
'python '
❸ >>> **favorite_language.rstrip()**
'python'
❹ >>> **favorite_language**
'python '
```

与 `favorite_language` 变量 ❶ 关联的值包含了字符串末尾的额外空白字符。当你在终端会话中请求这个值时，你可以看到末尾有一个空格 ❷。当 `rstrip()` 方法作用于变量 `favorite_language` ❸ 时，这个额外的空格会被移除。然而，这只是暂时移除。如果你再次请求 `favorite_language` 的值，字符串会和输入时一样，包括多余的空白字符 ❹。

要永久移除字符串中的空白字符，你需要将去除空白后的值与变量名关联：

```py
>>> **favorite_language = 'python '**
❶ >>> **favorite_language = favorite_language.rstrip()**
>>> **favorite_language**
'python'
```

要去除字符串中的空白字符，你可以先去除右侧的空白字符，然后将这个新值与原始变量关联 ❶。在编程中，经常会修改变量的值。这就是如何在程序执行过程中或者响应用户输入时更新变量的值。

你也可以使用 `lstrip()` 方法去除字符串左侧的空白字符，或者使用 `strip()` 一次性去除两侧的空白字符：

```py
❶ >>> **favorite_language = ' python '**
❷ >>> **favorite_language.rstrip()**
' python'
❸ >>> **favorite_language.lstrip()**
'python '
❹ >>> **favorite_language.strip()**
'python'
```

在这个示例中，我们从一个字符串开始，该字符串的开头和结尾有空白字符 ❶。接着，我们移除右侧的额外空白字符 ❷、左侧的空白字符 ❸，以及两侧的空白字符 ❹。通过尝试这些去除空白字符的函数，你可以熟悉如何处理字符串。在现实中，这些去除空白的函数通常用于清理用户输入的数据，确保它们在程序中存储之前是干净的。

### 去除前缀

在处理字符串时，另一个常见的任务是去除前缀。以带有常见前缀 *https://* 的 URL 为例。我们希望去除这个前缀，这样就可以专注于用户需要在地址栏中输入的 URL 部分。以下是如何做到这一点：

```py
>>> nostarch_url = 'https://nostarch.com'
>>> nostarch_url.removeprefix('https://')
'nostarch.com'
```

输入变量名，后跟一个点，然后是方法 `removeprefix()`。在括号内，输入你想从原始字符串中移除的前缀。

与去除空白字符的方法类似，`removeprefix()` 不会改变原始字符串。如果你希望保留去掉前缀后的新值，可以将其重新赋值给原变量，或者赋值给一个新变量：

```py
>>> simple_url = nostarch_url.removeprefix('https://')
```

当你在地址栏中看到一个 URL，且 *https://* 部分没有显示时，浏览器可能在后台使用了类似 `removeprefix()` 的方法。

### 避免字符串中的语法错误

你可能会经常看到的一种错误是语法错误。*语法错误* 发生在 Python 无法将程序的某一部分识别为有效的 Python 代码时。例如，如果你在单引号中使用了撇号，程序就会报错。这是因为 Python 会把第一个单引号和撇号之间的内容当作字符串处理，然后试图将剩余的文本解释为 Python 代码，导致错误。

下面是如何正确使用单引号和双引号。将这个程序保存为 *apostrophe.py* 然后运行它：

**apostrophe.py**

```py
message = "One of Python's strengths is its diverse community."
print(message)
```

单引号出现在一对双引号内部，所以 Python 解释器能够正确地读取字符串：

```py
One of Python's strengths is its diverse community.
```

然而，如果你使用单引号，Python 就无法识别字符串应该在哪里结束：

```py
message = 'One of Python's strengths is its diverse community.'
print(message)
```

你将看到以下输出：

```py
 File "apostrophe.py", line 1
    message = 'One of Python's strengths is its diverse community.'
                                                                ❶ ^
SyntaxError: unterminated string literal (detected at line 1)
```

在输出中，你可以看到错误发生在最后一个单引号之后 ❶。这个语法错误表示解释器无法将代码中的某些内容识别为有效的 Python 代码，并且它认为问题可能出在未正确引用的字符串。错误的来源有很多种，我会在出现时指出一些常见的错误。当你学习编写正确的 Python 代码时，可能会经常遇到语法错误。语法错误也是最不具体的一种错误，因此它们可能很难识别和修正。如果你在解决特别棘手的错误时卡住了，可以参考附录 C 中的建议。

## 数字

在编程中，数字经常用于游戏中的得分、数据可视化、存储 Web 应用中的信息等。Python 根据数字的使用方式以几种不同的方式处理数字。我们首先来看 Python 如何处理整数，因为它们是最简单的。

### 整数

你可以在 Python 中对整数进行加法（`+`）、减法（`-`）、乘法（`*`）和除法（`/`）。

```py
>>> **2 + 3**
5
>>> **3 - 2**
1
>>> **2 * 3**
6
>>> **3 / 2**
1.5
```

在终端会话中，Python 会直接返回运算结果。Python 使用两个乘号来表示指数：

```py
>>> **3 ** 2**
9
>>> **3 ** 3**
27
>>> **10 ** 6**
1000000
```

Python 也支持运算顺序，因此你可以在一个表达式中使用多个运算符。你还可以使用括号来修改运算顺序，这样 Python 就能按照你指定的顺序计算表达式。例如：

```py
>>> **2 + 3*4**
14
>>> **(2 + 3) * 4**
20
```

这些示例中的空格对 Python 计算表达式没有影响；它们只是帮助你在阅读代码时更快速地识别优先级较高的运算。

### 浮点数

Python 将任何带有小数点的数字称为 *浮点数*。这个术语在大多数编程语言中都使用，它指的是小数点可以出现在数字的任何位置。每种编程语言都必须精心设计，以正确地处理小数数字，使得数字无论小数点出现在什么位置都能正确表现。

大部分情况下，你可以使用浮点数而不需要担心它们的表现。只需输入你想要使用的数字，Python 很可能会按你预期的方式进行计算：

```py
>>> **0.1 + 0.1**
0.2
>>> **0.2 + 0.2**
0.4
>>> **2 * 0.1**
0.2
>>> **2 * 0.2**
0.4
```

然而，请注意，有时你会得到一个任意的小数位数作为答案：

```py
>>> **0.2 + 0.1**
0.30000000000000004
>>> **3 * 0.1**
0.30000000000000004
```

这种情况在所有语言中都会发生，通常不需要担心。Python 尝试以尽可能精确的方式表示结果，这有时很困难，因为计算机需要内部表示数字。现在先忽略额外的小数位；当你在第二部分的项目中需要处理这些额外的位数时，你将学习如何应对。

### 整数和浮点数

当你将两个数字相除时，即使它们是整数并且结果是一个整数，你仍然会得到一个浮点数：

```py
>>> **4/2**
2.0
```

如果你在其他操作中混合使用整数和浮点数，你也会得到一个浮点数：

```py
>>> **1 + 2.0**
3.0
>>> **2 * 3.0**
6.0
>>> **3.0 ** 2**
9.0
```

Python 在任何涉及浮点数的操作中默认会使用浮点数，即使输出结果是一个整数。

### 数字中的下划线

当你编写长数字时，可以使用下划线将数字分组，使大数字更易读：

```py
>>> **universe_age = 14_000_000_000**
```

当你打印一个使用下划线定义的数字时，Python 只会打印出数字本身：

```py
>>> **print(universe_age)**
14000000000
```

Python 在存储这些类型的值时会忽略下划线。即使你没有将数字分为三位一组，值也不会受到影响。对 Python 来说，`1000` 和 `1_000`，以及 `10_00` 都是一样的。这一特性适用于整数和浮点数。

### 多重赋值

你可以使用一行代码为多个变量赋值。这有助于缩短你的程序并使其更易读；你通常在初始化一组数字时使用这种技巧。

例如，下面是如何将变量 `x`、`y` 和 `z` 初始化为零：

```py
>>> **x, y, z = 0, 0, 0**
```

你需要使用逗号分隔变量名，并且对值也要做同样的处理，Python 会将每个值分配给相应的变量。只要值的数量与变量的数量相匹配，Python 会正确地进行匹配。

### 常量

*常量* 是一个在程序生命周期内其值保持不变的变量。Python 没有内建的常量类型，但 Python 程序员通过使用全大写字母来表示一个变量应视为常量且永远不应改变：

```py
MAX_CONNECTIONS = 5000
```

当你想在代码中将一个变量视为常量时，可以将该变量的名称写成全大写字母。

## 注释

注释是大多数编程语言中非常有用的功能。到目前为止，你在程序中编写的所有内容都是 Python 代码。随着程序变得越来越长且复杂，你应该在程序中添加注释，描述你解决问题的整体思路。*注释* 允许你在程序中以口语化的语言编写说明。

### 如何编写注释？

在 Python 中，井号（`#`）表示注释。井号后面的所有内容都将被 Python 解释器忽略。例如：

**comment.py**

```py
# Say hello to everyone.
print("Hello Python people!")
```

Python 会忽略第一行并执行第二行。

```py
Hello Python people!
```

### 你应该写什么样的注释？

编写注释的主要原因是解释你的代码应该做什么，以及你如何使其工作。当你正在进行项目时，你会明白所有部分是如何配合工作的。但当你在一段时间后回到项目时，你很可能会忘记一些细节。你可以总是花时间去研究代码，弄明白各个部分应该如何工作，但编写好的注释可以通过清晰地总结你的整体思路，节省你时间。

如果你想成为一名专业程序员或与其他程序员合作，你应该编写有意义的评论。如今，大多数软件都是以协作的方式编写的，无论是由一家公司的员工团队，还是由一群人在一个开源项目中合作。技术熟练的程序员期望在代码中看到评论，因此最好从现在开始为你的程序添加描述性的评论。编写清晰、简洁的评论是你作为新程序员可以培养的最有益的习惯之一。

当你决定是否写评论时，问问自己是否在考虑了几种方法之后，才想出了一个合理的解决方案；如果是这样，那就写一个关于你解决方案的评论。比起回头为一段评论稀少的程序写评论，稍后删除多余的评论要容易得多。从现在开始，我将在本书的示例中使用评论，帮助解释代码部分。

## Python 之禅

经验丰富的 Python 程序员会鼓励你避免复杂性，尽可能追求简单。Python 社区的哲学体现在 Tim Peters 的《Python 之禅》中。你可以通过在解释器中输入 `import this` 来查看这组简短的编写优秀 Python 代码的原则。我不会在这里完整地重现《Python 之禅》，但我会分享其中几行，帮助你理解它们为什么对你作为一个初学者非常重要。

```py
>>> **import this**
The Zen of Python, by Tim Peters
Beautiful is better than ugly.
```

Python 程序员认同代码可以是美丽且优雅的这个概念。在编程中，人们是用来解决问题的。程序员一直尊重那些设计良好、高效甚至美丽的解决方案。当你对 Python 了解得更多，并用它写更多代码时，有一天可能会有人站在你肩膀上，说：“哇，那是一些美丽的代码！”

```py
Simple is better than complex.
```

如果你在简单和复杂的解决方案之间有选择，而且两者都有效，那就选择简单的解决方案。你的代码将更容易维护，并且以后你和其他人也能更轻松地在这段代码上进行扩展。

```py
Complex is better than complicated.
```

现实生活是混乱的，有时候简单的解决方案是无法实现的。在这种情况下，使用最简单的有效解决方案。

```py
Readability counts.
```

即使你的代码很复杂，也要尽量使它易于阅读。当你在一个涉及复杂编码的项目中工作时，专注于为这些代码编写具有信息量的评论。

```py
There should be one-- and preferably only one --obvious way to do it.
```

如果两个 Python 程序员被要求解决同一个问题，他们应该会提出相当兼容的解决方案。这并不是说编程中没有创造性的空间。相反，编程中有很多创造性空间！然而，编程的许多部分是利用小而常见的方法来解决较大、更具创意项目中的简单问题。你程序中的基础部分应该能让其他 Python 程序员理解。

```py
Now is better than never.
```

你可以花费余生学习 Python 和编程的所有细节，但那样你将永远无法完成任何项目。不要试图编写完美的代码；编写能工作的代码，然后决定是改进该项目的代码，还是继续做新的项目。

当你继续进入下一章并开始深入探讨更复杂的话题时，试着保持这个简单明了的哲学理念。经验丰富的程序员会更尊重你的代码，并乐意为你提供反馈，并与你一起合作进行有趣的项目。

## 总结

在本章中，你学习了如何使用变量。你学会了使用描述性变量名，并在出现名称错误和语法错误时进行解决。你了解了什么是字符串，并学习了如何使用小写字母、大写字母和标题大小写显示字符串。你开始使用空格来整理输出，使其整洁，并学习了如何从字符串中删除不需要的元素。你开始使用整数和浮动小数，学习了处理数字数据的一些方法。你还学会了编写解释性注释，使你的代码更容易被你和他人阅读。最后，你阅读了保持代码尽可能简单的哲学理念，无论何时可能。

在第3章中，你将学习如何在数据结构中存储信息集合，这些数据结构被称为*列表*。你还将学习如何遍历列表，操作列表中的任何信息。

# 3

介绍列表

![](Images/chapterart.png)

在本章及下一章中，你将学习什么是列表，以及如何开始处理列表中的元素。列表允许你将一组信息存储在一个地方，无论你有几个项目还是数百万个项目。列表是 Python 中最强大的功能之一，易于新程序员使用，并且它连接了编程中的许多重要概念。

## 什么是列表？

*列表*是按特定顺序排列的项目集合。你可以创建一个包含字母表的字母、从 0 到 9 的数字，或你家里所有人的名字的列表。你可以将任何想要的东西放入列表中，且列表中的项目不需要以任何特定方式相关。因为列表通常包含多个元素，所以最好将列表的名称设置为复数形式，比如 `letters`、`digits` 或 `names`。

在 Python 中，方括号（`[]`）表示一个列表，列表中的单个元素用逗号分隔。以下是一个简单的包含几种自行车的列表示例：

**bicycles.py**

```py
bicycles = ['trek', 'cannondale', 'redline', 'specialized']
print(bicycles)
```

如果你要求 Python 打印一个列表，Python 会返回它对该列表的表示，包括方括号：

```py
['trek', 'cannondale', 'redline', 'specialized']
```

因为这不是你希望用户看到的输出，让我们学习如何访问列表中的单个项目。

### 访问列表中的元素

列表是有序的集合，因此你可以通过告诉Python项的*位置*或*索引*来访问列表中的任何元素。要访问列表中的元素，写出列表名称，然后用方括号括起该项的索引。

例如，让我们从列表`bicycles`中提取第一辆自行车：

```py
bicycles = ['trek', 'cannondale', 'redline', 'specialized']
print(bicycles[0])
```

当我们从列表中请求单个元素时，Python只返回该元素，而不带方括号：

```py
trek
```

这是你希望用户看到的结果：干净、整齐格式化的输出。

你还可以对列表中的任何元素使用第二章中的字符串方法。例如，你可以使用`title()`方法将元素`'trek'`格式化为更整洁的外观：

```py
bicycles = ['trek', 'cannondale', 'redline', 'specialized']
print(bicycles[0].title())
```

这个例子产生的输出与前一个例子相同，不同之处在于`'Trek'`被大写了。

### 索引位置从0开始，而不是从1开始

Python认为列表中的第一个项目位于位置0，而不是位置1。这在大多数编程语言中都是如此，原因与列表操作在较低层次上如何实现有关。如果你遇到意外结果，问问自己是否犯了一个简单但常见的错位错误。

列表中的第二项的索引是1。使用这种计数系统，你可以通过从元素在列表中的位置减去1来获取任何你想要的元素。例如，要访问列表中的第四项，你请求索引为3的元素。

以下请求的是索引`1`和索引`3`处的自行车：

```py
bicycles = ['trek', 'cannondale', 'redline', 'specialized']
print(bicycles[1])
print(bicycles[3])
```

这段代码返回列表中的第二和第四辆自行车：

```py
cannondale
specialized
```

Python有一种特殊的语法来访问列表中的最后一个元素。如果你请求索引`-1`的项，Python总是返回列表中的最后一个元素：

```py
bicycles = ['trek', 'cannondale', 'redline', 'specialized']
print(bicycles[-1])
```

这段代码返回值`'specialized'`。这种语法非常有用，因为你经常希望在不知道列表长度的情况下访问列表中的最后一个元素。这个约定也适用于其他负索引值。索引`-2`返回列表倒数第二项，索引`-3`返回倒数第三项，依此类推。

### 从列表中使用单个值

你可以像使用任何其他变量一样使用列表中的单个值。例如，你可以使用f-string根据列表中的一个值创建消息。

让我们试着从列表中提取第一辆自行车，并使用该值组成一条消息：

```py
bicycles = ['trek', 'cannondale', 'redline', 'specialized']
message = f"My first bicycle was a {bicycles[0].title()}."

print(message)
```

我们使用`bicycles[0]`中的值构建一句话，并将其赋值给变量`message`。输出是关于列表中第一辆自行车的简单句子：

```py
My first bicycle was a Trek.
```

## 修改、添加和删除元素

你创建的大多数列表都是*动态的*，这意味着你将构建一个列表，并在程序运行过程中添加和删除元素。例如，你可能会创建一个游戏，玩家需要把外星人从天上击落。你可以将初始的外星人集合存储在一个列表中，每次击落一个外星人时，就从列表中移除它。每当一个新的外星人出现在屏幕上时，你就将它添加到列表中。你的外星人列表将在游戏过程中不断增加和减少长度。

### 修改列表中的元素

修改元素的语法与访问列表中元素的语法相似。要更改某个元素，使用列表的名称，后跟你想更改的元素的索引，然后提供你希望该项具有的新值。

例如，假设我们有一个摩托车列表，列表中的第一项是`'honda'`。在列表创建后，我们可以更改第一项的值：

**motorcycles.py**

```py
motorcycles = ['honda', 'yamaha', 'suzuki']
print(motorcycles)

motorcycles[0] = 'ducati'
print(motorcycles)
```

这里我们定义了列表`motorcycles`，并将`'honda'`作为第一项。然后我们将第一项的值更改为`'ducati'`。输出显示，第一项已被更改，而其余的列表保持不变：

```py
['honda', 'yamaha', 'suzuki']
['ducati', 'yamaha', 'suzuki']
```

你可以更改列表中任何项的值，而不仅仅是第一项。

### 向列表添加元素

你可能会出于许多原因向列表添加新元素。例如，你可能想要在游戏中让新的外星人出现，向可视化中添加新数据，或者向你构建的网站中添加新的注册用户。Python 提供了几种向现有列表添加新数据的方法。

#### 将元素附加到列表的末尾

向列表添加新元素的最简单方法是将该项*附加*到列表中。当你将一个元素附加到列表时，新的元素会被添加到列表的末尾。使用我们在前面示例中的同一个列表，我们将新元素`'ducati'`添加到列表的末尾：

```py
motorcycles = ['honda', 'yamaha', 'suzuki']
print(motorcycles)

motorcycles.append('ducati')
print(motorcycles)
```

在这里，`append()`方法将`'ducati'`添加到列表的末尾，而不会影响列表中的其他元素：

```py
['honda', 'yamaha', 'suzuki']
['honda', 'yamaha', 'suzuki', 'ducati']
```

`append()`方法使动态构建列表变得非常简单。例如，你可以从一个空列表开始，然后使用一系列`append()`调用将项添加到列表中。使用一个空列表，我们将元素`'honda'`、`'yamaha'`和`'suzuki'`添加到列表中：

```py
motorcycles = []

motorcycles.append('honda')
motorcycles.append('yamaha')
motorcycles.append('suzuki')

print(motorcycles)
```

结果列表与前面的示例中的列表完全相同：

```py
['honda', 'yamaha', 'suzuki']
```

以这种方式构建列表非常常见，因为通常在程序运行之前你无法知道用户想要存储的数据。为了让用户掌控，首先定义一个空列表来保存用户的值。然后将每个新提供的值附加到你刚创建的列表中。

#### 向列表中插入元素

你可以通过使用`insert()`方法将新元素添加到列表中的任何位置。你通过指定新元素的索引和值来实现这一点：

```py
motorcycles = ['honda', 'yamaha', 'suzuki']

motorcycles.insert(0, 'ducati')
print(motorcycles)
```

在这个例子中，我们将值`'ducati'`插入到列表的开头。`insert()`方法在位置`0`处开辟一个空间，并将值`'ducati'`存储在该位置：

```py
['ducati', 'honda', 'yamaha', 'suzuki']
```

这个操作会将列表中的其他每个值向右移动一个位置。

### 从列表中移除元素

通常，你可能希望从列表中移除一个元素或一组元素。例如，当玩家击落一个外星人时，你通常会希望将其从活动外星人列表中移除。或者，当用户决定取消他们在你创建的Web应用程序中的账户时，你会希望将该用户从活动用户列表中移除。你可以根据元素在列表中的位置或其值来移除元素。

#### 使用del语句移除元素

如果你知道想从列表中移除的元素的位置，可以使用`del`语句：

```py
motorcycles = ['honda', 'yamaha', 'suzuki']
print(motorcycles)

del motorcycles[0]
print(motorcycles)
```

在这里，我们使用`del`语句从摩托车列表中移除第一个项目`'honda'`：

```py
['honda', 'yamaha', 'suzuki']
['yamaha', 'suzuki']
```

如果你知道元素的索引，可以使用`del`语句从列表中的任何位置移除元素。例如，下面是如何从列表中移除第二个元素`'yamaha'`：

```py
motorcycles = ['honda', 'yamaha', 'suzuki']
print(motorcycles)

del motorcycles[1]
print(motorcycles)
```

第二辆摩托车已从列表中删除：

```py
['honda', 'yamaha', 'suzuki']
['honda', 'suzuki']
```

在这两个例子中，使用`del`语句后，你将无法再访问被移除的列表元素。

#### 使用pop()方法移除元素

有时你可能希望在从列表中移除元素后使用该元素的值。例如，你可能想获取刚被击落的外星人的*x*和*y*位置，然后在该位置绘制爆炸效果。在一个Web应用程序中，你可能希望将一个用户从活动成员列表中移除，然后将该用户添加到非活动成员列表中。

`pop()`方法移除列表中的最后一个元素，但它允许你在移除后继续使用该元素。术语*pop*来源于将列表看作一个堆叠的元素，从堆叠顶部弹出一个元素。在这个类比中，堆叠的顶部对应于列表的末尾。

让我们从摩托车列表中弹出一辆摩托车：

```py
❶ motorcycles = ['honda', 'yamaha', 'suzuki']
print(motorcycles)

❷ popped_motorcycle = motorcycles.pop()
❸ print(motorcycles)
❹ print(popped_motorcycle)
```

我们首先定义并打印列表`motorcycles` ❶。然后我们从列表中弹出一个值，并将该值赋给变量`popped_motorcycle` ❷。我们打印列表 ❸ 来展示某个值已从列表中移除。接着我们打印弹出的值 ❹ 来证明我们仍然可以访问被移除的值。

输出显示值`'suzuki'`已从列表末尾移除，并且现在被赋值给变量`popped_motorcycle`：

```py
['honda', 'yamaha', 'suzuki']
['honda', 'yamaha']
suzuki
```

这个`pop()`方法有什么用呢？假设摩托车列表是按我们拥有它们的时间顺序存储的。如果是这种情况，我们可以使用`pop()`方法打印一条关于我们最后购买的摩托车的语句：

```py
motorcycles = ['honda', 'yamaha', 'suzuki']

last_owned = motorcycles.pop()
print(f"The last motorcycle I owned was a {last_owned.title()}.")
```

输出是关于我们最近拥有的摩托车的一句简单话语：

```py
The last motorcycle I owned was a Suzuki.
```

#### 从列表中的任何位置弹出元素

你可以使用`pop()`方法通过在括号中指定要删除的项的索引，来从列表中的任何位置删除一个项：

```py
motorcycles = ['honda', 'yamaha', 'suzuki']

first_owned = motorcycles.pop(0)
print(f"The first motorcycle I owned was a {first_owned.title()}.")
```

我们首先通过`pop()`删除列表中的第一辆摩托车，然后打印关于这辆摩托车的信息。输出是描述我曾拥有的第一辆摩托车的简单句子：

```py
The first motorcycle I owned was a Honda.
```

记住，每次使用`pop()`时，你处理的项将不再存储在列表中。

如果你不确定是使用`del`语句还是`pop()`方法，以下是一个简单的判断方法：当你想从列表中删除一个项并且不再使用该项时，使用`del`语句；如果你在删除项的同时还希望使用该项，使用`pop()`方法。

#### 按值删除项

有时你并不知道要从列表中删除的值的位置。如果你只知道要删除的项的值，可以使用`remove()`方法。

例如，假设我们想从摩托车列表中删除值`'ducati'`：

```py
motorcycles = ['honda', 'yamaha', 'suzuki', 'ducati']
print(motorcycles)

motorcycles.remove('ducati')
print(motorcycles)
```

这里`remove()`方法告诉Python找到`'ducati'`在列表中出现的位置，并删除该元素：

```py
['honda', 'yamaha', 'suzuki', 'ducati']
['honda', 'yamaha', 'suzuki']
```

你也可以使用`remove()`方法来处理从列表中删除的值。让我们删除值`'ducati'`并打印删除该项的原因：

```py
❶ motorcycles = ['honda', 'yamaha', 'suzuki', 'ducati']
print(motorcycles)

❷ too_expensive = 'ducati'
❸ motorcycles.remove(too_expensive)
print(motorcycles)
❹ print(f"\nA {too_expensive.title()} is too expensive for me.")
```

在定义列表❶之后，我们将值`'ducati'`赋给一个名为`too_expensive`的变量❷。然后，我们使用这个变量告诉Python要从列表中删除哪个值❸。值`'ducati'`已经从列表中删除❹，但仍可以通过变量`too_expensive`访问，从而允许我们打印一条关于为什么从摩托车列表中删除`'ducati'`的声明：

```py
['honda', 'yamaha', 'suzuki', 'ducati']
['honda', 'yamaha', 'suzuki']

A Ducati is too expensive for me.
```

## 组织列表

通常，列表会以不可预测的顺序创建，因为你无法总是控制用户提供数据的顺序。虽然在大多数情况下这是不可避免的，但你经常会希望以特定的顺序展示信息。有时你想保留列表的原始顺序，其他时候你可能想改变原始顺序。Python根据不同情况提供了多种方法来组织列表。

### 使用`sort()`方法永久排序列表

Python的`sort()`方法使得排序列表变得相对简单。假设我们有一份汽车列表，并且想要将列表的顺序改为按字母顺序排列。为了简化任务，假设列表中的所有值都是小写字母：

**cars.py**

```py
cars = ['bmw', 'audi', 'toyota', 'subaru']
cars.sort()
print(cars)
```

`sort()`方法会永久改变列表的顺序。现在这些汽车已经按字母顺序排列，我们将无法恢复到原来的顺序：

```py
['audi', 'bmw', 'subaru', 'toyota']
```

你也可以通过向`sort()`方法传递参数`reverse=True`，按逆字母顺序对列表进行排序。以下示例将汽车列表按逆字母顺序排序：

```py
cars = ['bmw', 'audi', 'toyota', 'subaru']
cars.sort(reverse=True)
print(cars)
```

再次强调，列表的顺序已经永久改变：

```py
['toyota', 'subaru', 'bmw', 'audi']
```

### 使用`sorted()`函数临时排序列表

如果你希望保持列表的原始顺序，但以排序的顺序显示列表，可以使用`sorted()`函数。`sorted()`函数允许你以特定的顺序显示列表，但不会影响列表的实际顺序。

让我们在汽车列表上尝试这个函数。

```py
cars = ['bmw', 'audi', 'toyota', 'subaru']

❶ print("Here is the original list:")
print(cars)

❷ print("\nHere is the sorted list:")
print(sorted(cars))

❸ print("\nHere is the original list again:")
print(cars)
```

我们首先按原始顺序打印列表❶，然后按字母顺序打印❷。在列表按照新顺序显示后，我们展示该列表仍然以原始顺序存储❸：

```py
Here is the original list:
['bmw', 'audi', 'toyota', 'subaru']

Here is the sorted list:
['audi', 'bmw', 'subaru', 'toyota']

❶ Here is the original list again:
['bmw', 'audi', 'toyota', 'subaru']
```

请注意，在使用`sorted()`函数之后，列表仍然保留原始顺序❶。如果你希望列表按反字母顺序显示，`sorted()`函数还可以接受`reverse=True`参数。

### 以相反的顺序打印列表

要反转列表的原始顺序，你可以使用`reverse()`方法。如果我们最初按照拥有这些车的时间顺序存储了这些车的列表，我们可以很容易地将列表按时间倒序排列：

```py
cars = ['bmw', 'audi', 'toyota', 'subaru']
print(cars)

cars.reverse()
print(cars)
```

请注意，`reverse()`不会按照字母顺序倒排；它只是反转列表的顺序：

```py
['bmw', 'audi', 'toyota', 'subaru']
['subaru', 'toyota', 'audi', 'bmw']
```

`reverse()`方法会永久改变列表的顺序，但你可以通过再次对同一个列表调用`reverse()`来恢复原来的顺序。

### 查找列表的长度

你可以通过使用`len()`函数快速找到列表的长度。这个示例中的列表有四个元素，所以它的长度是`4`：

```py
>>> **cars = ['bmw', 'audi', 'toyota', 'subaru']**
>>> **len(cars)**
4
```

你会发现，`len()`在你需要确定游戏中仍需击落的外星人数量、确定你需要管理的可视化数据量或计算网站上注册用户数量等任务时非常有用。

## 在处理列表时避免索引错误

在你第一次使用列表时，有一种常见的错误类型。假设你有一个包含三个元素的列表，而你请求第四个元素：

**motorcycles.py**

```py
motorcycles = ['honda', 'yamaha', 'suzuki']
print(motorcycles[3])
```

这个例子会导致一个*索引错误*：

```py
Traceback (most recent call last):
  File "motorcycles.py", line 2, in <module>
    print(motorcycles[3])
          ~~~~~~~~~~~^^^
IndexError: list index out of range
```

Python试图给你索引为3的元素。但当它查找列表时，`motorcycles`中没有索引为3的元素。由于列表索引的下标从0开始，这种错误是常见的。人们通常认为第三个元素是索引3的元素，因为他们习惯从1开始计数。但在Python中，第三个元素的索引是2，因为索引从0开始。

索引错误意味着Python无法找到你请求的索引位置的元素。如果你的程序中发生了索引错误，试着调整你请求的索引值，然后再次运行程序看看结果是否正确。

请记住，每当你想访问列表中的最后一个元素时，你应该使用索引`-1`。即使你的列表自上次访问以来发生了变化，这个方法也始终有效：

```py
motorcycles = ['honda', 'yamaha', 'suzuki']
print(motorcycles[-1])
```

索引`-1`总是返回列表中的最后一个元素，在这个例子中是值`suzuki`：

```py
suzuki
```

这种方法唯一会引发错误的情况是，当你请求一个空列表中的最后一个元素时：

```py
motorcycles = []
print(motorcycles[-1])
```

`motorcycles` 中没有项目，因此 Python 返回另一个索引错误：

```py
Traceback (most recent call last):
  File "motorcyles.py", line 3, in <module>
    print(motorcycles[-1])
          ~~~~~~~~~~~^^^^
IndexError: list index out of range
```

如果发生索引错误，并且你无法弄清楚如何解决它，可以尝试打印你的列表或仅打印列表的长度。你的列表可能与你想象的非常不同，尤其是当它由程序动态管理时。查看实际的列表或列表中项目的确切数量，可以帮助你理清这些逻辑错误。

## 总结

在本章中，你学习了什么是列表以及如何处理列表中的各个项目。你学习了如何定义列表，如何添加和移除元素。你还学习了如何对列表进行永久和临时排序，以便展示。你还学习了如何查找列表的长度，以及如何在处理列表时避免索引错误。

在第 4 章中，你将学习如何更高效地处理列表中的项目。通过仅使用几行代码遍历列表中的每一项，你将能够高效地工作，即使列表包含数千或数百万个项目。

# 4

处理列表

![](Images/chapterart.png)

在第 3 章中，你学习了如何创建一个简单的列表，并学习了如何处理列表中的单个元素。在本章中，你将学习如何使用仅几行代码遍历整个列表，无论列表有多长。*循环*允许你对列表中的每一项执行相同的操作或一组操作。因此，你将能够高效地处理任何长度的列表，包括那些包含数千或甚至数百万个项目的列表。

## 遍历整个列表

你常常需要遍历列表中的所有条目，对每一项执行相同的任务。例如，在游戏中，你可能想要将屏幕上的每个元素移动相同的距离。在一个数字列表中，你可能想对每个元素执行相同的统计操作。或者，你可能想在一个网站上显示每篇文章的标题。当你想对列表中的每一项执行相同的操作时，可以使用 Python 的 `for` 循环。

假设我们有一个魔术师名字的列表，并且我们想打印出列表中的每个名字。我们可以通过单独取出列表中的每个名字来做到这一点，但这种方法可能会导致几个问题。首先，对于一个很长的名字列表，重复这样的操作会很繁琐。其次，每次列表长度发生变化时，我们必须修改代码。使用 `for` 循环可以避免这两个问题，因为 Python 会内部处理这些问题。

让我们使用 `for` 循环打印出魔术师名单中的每个名字：

**magicians.py**

```py
magicians = ['alice', 'david', 'carolina']
for magician in magicians:
    print(magician)
```

我们从定义一个列表开始，就像我们在第3章中做的那样。然后我们定义一个`for`循环。这一行告诉Python从列表`magicians`中取出一个名字，并将其与变量`magician`关联。接下来，我们告诉Python打印刚刚分配给`magician`的名字。Python接着重复这两行代码，每个列表中的名字执行一次。你可以将这段代码理解为“对于`magicians`列表中的每个魔术师，打印该魔术师的名字。”输出将是列表中每个名字的简单打印：

```py
alice
david
carolina
```

### 更深入地了解循环

循环很重要，因为它是计算机自动化重复任务的最常见方式之一。例如，在像我们在*magicians.py*中使用的简单循环中，Python首先读取循环的第一行：

```py
for magician in magicians:
```

这一行告诉Python从列表`magicians`中提取第一个值，并将其与变量`magician`关联。这个第一个值是`'alice'`。然后，Python继续读取下一行：

```py
 print(magician)
```

Python再次打印当前`magician`的值，仍然是`'alice'`。由于列表中还有更多值，Python返回到循环的第一行：

```py
for magician in magicians:
```

Python检索列表中的下一个名字`'david'`，并将该值与变量`magician`关联。然后，Python执行这一行：

```py
 print(magician)
```

Python再次打印当前`magician`的值，这次是`'david'`。Python再次执行整个循环，使用列表中的最后一个值`'carolina'`。由于列表中没有更多的值，Python转到程序中的下一行。在这种情况下，`for`循环后面没有更多的代码，所以程序结束。

当你第一次使用循环时，请记住，这些步骤会针对列表中的每个项目重复执行一次，不管列表中有多少项。如果列表中有一百万个项目，Python将这些步骤重复执行一百万次——通常执行得非常快。

另外，在编写你自己的`for`循环时，记住你可以为与列表中每个值关联的临时变量选择任何你想要的名称。然而，选择一个有意义的名称来表示列表中的单个项是很有帮助的。例如，以下是为猫的列表、狗的列表和一般物品列表编写`for`循环的一个好方法：

```py
for cat in cats:
for dog in dogs:
for item in list_of_items:
```

这些命名约定可以帮助你跟踪在`for`循环中每个项所执行的操作。使用单数和复数名称可以帮助你识别代码的某一部分是在处理列表中的单个元素还是整个列表。

### 在`for`循环中做更多的工作

你可以对`for`循环中的每个项目做几乎任何事情。让我们在之前的例子基础上，向每个魔术师打印一条消息，告诉他们他们表演了一个很棒的魔术：

**magicians.py**

```py
magicians = ['alice', 'david', 'carolina']
for magician in magicians:
    print(f"{magician.title()}, that was a great trick!")
```

这段代码唯一的区别在于我们为每个魔术师构建消息，消息从该魔术师的名字开始。在第一次执行循环时，`magician`的值是`'alice'`，所以Python会用名字`'Alice'`开始第一条消息。第二次执行时，消息会以`'David'`开头，第三次执行时，消息会以`'Carolina'`开头。

输出显示了列表中每位魔术师的个性化消息：

```py
Alice, that was a great trick!
David, that was a great trick!
Carolina, that was a great trick!
```

你也可以在`for`循环中写任意多行代码。在`for magician in magicians`这一行之后的每一行缩进代码都被认为是在*循环内*，并且每一行会为列表中的每个值执行一次。因此，你可以在每个列表项上执行任意多的操作。

让我们为消息添加第二行，告诉每位魔术师我们期待他们的下一个魔术：

```py
magicians = ['alice', 'david', 'carolina']
for magician in magicians:
 print(f"{magician.title()}, that was a great trick!")
    print(f"I can't wait to see your next trick, {magician.title()}.\n")
```

由于我们已经缩进了两次`print()`的调用，因此每一行会为列表中的每个魔术师执行一次。第二次`print()`调用中的换行符（`"\n"`）会在每次循环后插入一个空行。这就创建了一组整齐分组的消息，每个人都有一条：

```py
Alice, that was a great trick!
I can't wait to see your next trick, Alice.

David, that was a great trick!
I can't wait to see your next trick, David.

Carolina, that was a great trick!
I can't wait to see your next trick, Carolina.
```

你可以在`for`循环中使用任意多的行。实际上，你会发现，在使用`for`循环时，通常需要对列表中的每个项目执行多种不同的操作。

### 在`for`循环后做一些事情

一旦`for`循环执行完毕会发生什么？通常，你会希望总结输出块，或者继续执行程序中必须完成的其他工作。

在`for`循环后面任何没有缩进的代码行都只会执行一次，而不会重复执行。我们来为魔术师们写一条感谢信息，感谢他们为大家呈现了一场精彩的表演。为了在打印完所有个人消息后显示这条群体消息，我们将感谢信息写在`for`循环之后，且不缩进：

```py
magicians = ['alice', 'david', 'carolina']
for magician in magicians:
 print(f"{magician.title()}, that was a great trick!")
 print(f"I can't wait to see your next trick, {magician.title()}.\n")

print("Thank you, everyone. That was a great magic show!")
```

前两次调用`print()`会为列表中的每个魔术师重复一次，正如你之前所看到的。然而，由于最后一行没有缩进，它只会执行一次：

```py
Alice, that was a great trick!
I can't wait to see your next trick, Alice.

David, that was a great trick!
I can't wait to see your next trick, David.

Carolina, that was a great trick!
I can't wait to see your next trick, Carolina.

Thank you, everyone. That was a great magic show!
```

当你使用`for`循环处理数据时，你会发现这是总结对整个数据集执行的操作的好方法。例如，你可能会用`for`循环来初始化一个游戏，遍历角色列表并在屏幕上显示每个角色。然后你可能会在循环后写一些额外的代码，在所有角色都显示在屏幕后显示一个*现在开始游戏*按钮。

## 避免缩进错误

Python 使用缩进来确定某一行或一组行与程序其余部分的关系。在之前的示例中，打印每个魔术师消息的那些行属于 `for` 循环，因为它们是缩进的。Python 使用缩进使得代码非常容易阅读。基本上，它通过空白符强制你编写格式整洁且具有清晰视觉结构的代码。在更长的 Python 程序中，你会注意到有些代码块在不同的缩进级别上。这些缩进级别帮助你对整个程序的组织结构有一个大致的了解。

当你开始编写依赖于正确缩进的代码时，你需要注意一些常见的*缩进错误*。例如，人们有时会缩进不需要缩进的代码行，或者忘记缩进需要缩进的代码行。现在看到这些错误的示例会帮助你在将来避免它们，并在自己的程序中出现时修正它们。

让我们来看看一些更常见的缩进错误。

### 忘记缩进

在循环中的 `for` 语句后总是需要缩进。如果你忘记了，Python 会提醒你：

**magicians.py**

```py
magicians = ['alice', 'david', 'carolina']
for magician in magicians:
❶ print(magician)
```

调用 `print()` ❶ 应该缩进，但它没有缩进。当 Python 期望一个缩进块却没有找到时，它会告诉你它在哪一行出现了问题：

```py
 File "magicians.py", line 3
    print(magician)
    ^
IndentationError: expected an indented block after 'for' statement on line 2
```

通常，你可以通过在 `for` 语句后立即缩进该行或多行来解决这种缩进错误。

### 忘记缩进额外的行

有时你的循环会在没有错误的情况下运行，但却没有产生预期的结果。这种情况通常发生在你试图在循环中执行多个任务，却忘记缩进其中的一些行时。

例如，当我们忘记缩进循环中的第二行（这行代码告诉每个魔术师我们期待他们的下一个魔术时），就会发生以下情况：

```py
magicians = ['alice', 'david', 'carolina']
for magician in magicians:
    print(f"{magician.title()}, that was a great trick!")
❶ print(f"I can't wait to see your next trick, {magician.title()}.\n")
```

第二次调用 `print()` ❶ 应该缩进，但由于 Python 在 `for` 语句后发现至少有一行缩进的代码，它并不会报错。因此，第一个 `print()` 调用会针对列表中的每个名字执行一次，因为它是缩进的。第二个 `print()` 调用没有缩进，所以它只会在循环结束后执行一次。由于与 `magician` 关联的最终值是 `'carolina'`，因此只有她收到了“期待下一场魔术”的消息：

```py
Alice, that was a great trick!
David, that was a great trick!
Carolina, that was a great trick!
I can't wait to see your next trick, Carolina.
```

这是一个*逻辑错误*。语法上是有效的 Python 代码，但由于逻辑上出现问题，代码并没有产生预期的结果。如果你期望看到某个操作对列表中的每个项都执行一次，而它只执行了一次，请检查是否需要缩进某一行或一组行。

### 不必要的缩进

如果你不小心缩进了一个不需要缩进的行，Python 会提示你关于意外缩进的错误：

**hello_world.py**

```py
message = "Hello Python world!"
    print(message)
```

我们不需要缩进 `print()` 调用，因为它不是循环的一部分；因此，Python 会报告这个错误：

```py
 File "hello_world.py", line 2
    print(message)
    ^
IndentationError: unexpected indent
```

通过只有在有特定理由时才进行缩进，你可以避免意外的缩进错误。在你此时编写的程序中，唯一需要缩进的行是你希望在每次 `for` 循环中的每个项目上重复执行的操作。

### 循环后不必要的缩进

如果你不小心缩进了应当在循环结束后运行的代码，这段代码将在列表中的每一项上都执行一次。有时这会导致 Python 报告错误，但通常会导致逻辑错误。

例如，看看当我们不小心缩进了感谢魔术师们集体表现出色的那一行时会发生什么：

**magicians.py**

```py
magicians = ['alice', 'david', 'carolina']
for magician in magicians:
    print(f"{magician.title()}, that was a great trick!")
    print(f"I can't wait to see your next trick, {magician.title()}.\n")

❶     print("Thank you everyone, that was a great magic show!")
```

由于最后一行❶有缩进，它会为列表中的每一个人都执行一次：

```py
Alice, that was a great trick!
I can't wait to see your next trick, Alice.

Thank you everyone, that was a great magic show!
David, that was a great trick!
I can't wait to see your next trick, David.

Thank you everyone, that was a great magic show!
Carolina, that was a great trick!
I can't wait to see your next trick, Carolina.

Thank you everyone, that was a great magic show!
```

这是另一个逻辑错误，类似于第54页中“忘记缩进额外的行”中的错误。由于 Python 不知道你想通过代码实现什么，它会运行所有符合语法规则的代码。如果某个操作应当仅执行一次，却被多次重复执行，你可能需要取消该操作的缩进。

### 忘记冒号

`for`语句末尾的冒号告诉 Python 将下一行解释为循环的开始。

```py
magicians = ['alice', 'david', 'carolina']
❶ for magician in magicians
    print(magician)
```

如果你不小心忘记了冒号❶，你会得到语法错误，因为 Python 不知道你到底想做什么：

```py
 File "magicians.py", line 2
    for magician in magicians
                             ^
SyntaxError: expected ':'
```

Python 不知道你是仅仅忘记了冒号，还是打算写额外的代码来设置一个更复杂的循环。如果解释器能够识别出一个可能的修复，它会给出建议，比如在行尾加上冒号，就像这里的 `expected ':'` 响应一样。有些错误有简单、显而易见的修复，这要归功于 Python 错误追踪中提供的建议。也有一些错误，即使最终的修复仅涉及一个字符，也更难解决。当一个小修复花费很长时间才找到时，不必感到沮丧；你绝对不是唯一有这种经历的人。

## 创建数字列表

存储一组数字有很多理由。例如，你可能需要跟踪游戏中每个角色的位置，也许还想记录玩家的最高分。在数据可视化中，你几乎总是会处理一组数字，比如温度、距离、人口大小，或者经纬度值等各种类型的数字集合。

列表非常适合存储数字集合，Python 提供了多种工具来帮助你高效地处理数字列表。一旦你理解了如何有效地使用这些工具，即使你的列表包含数百万个项目，代码也能顺利运行。

### 使用 `range()` 函数

Python 的 `range()` 函数使得生成一系列数字变得简单。例如，你可以使用 `range()` 函数像这样打印一系列数字：

**first_numbers.py**

```py
for value in range(1, 5):
    print(value)
```

尽管这段代码看起来应该打印从 1 到 5 的数字，但它没有打印数字 5：

```py
1
2
3
4
```

在这个例子中，`range()`只打印1到4之间的数字。这是编程语言中常见的“偏差一”行为的另一个例子。`range()`函数会让Python从你提供的第一个值开始计数，并在达到你提供的第二个值时停止。因为它会在第二个值时停止，输出中永远不会包含结束值，而在这个例子中，结束值本应是5。

要打印从1到5的数字，你可以使用`range(1, 6)`：

```py
for value in range(1, 6):
    print(value)
```

这次输出从1开始，到5结束：

```py
1
2
3
4
5
```

如果你在使用`range()`时，输出与预期不符，尝试调整结束值加1。

你也可以只传递一个参数给`range()`，它会从0开始生成数字序列。例如，`range(6)`将返回从0到5的数字。

### 使用`range()`创建一个数字列表

如果你想制作一个数字列表，你可以使用`list()`函数将`range()`的结果直接转换为列表。当你将`list()`包裹在对`range()`函数的调用周围时，输出将是一个数字列表。

在前面章节的例子中，我们仅仅是打印了一系列数字。我们可以使用`list()`将这组数字转换为列表：

```py
numbers = list(range(1, 6))
print(numbers)
```

这是结果：

```py
[1, 2, 3, 4, 5]
```

我们还可以使用`range()`函数告诉Python跳过给定范围内的数字。如果你传递第三个参数给`range()`，Python会使用该值作为生成数字时的步长。

例如，下面是如何列出1到10之间的偶数：

**even_numbers.py**

```py
even_numbers = list(range(2, 11, 2))
print(even_numbers)
```

在这个例子中，`range()`函数从值2开始，然后将2加到这个值上。它会重复加2，直到达到或超过结束值11，并产生如下结果：

```py
[2, 4, 6, 8, 10]
```

你可以使用`range()`函数创建几乎任何你想要的数字集。例如，考虑如何制作一个包含前10个平方数的列表（即从1到10的每个整数的平方）。在Python中，两个星号（`**`）表示指数运算。下面是如何将前10个平方数放入列表的示例：

**square_numbers.py**

```py
squares = []
for value in range(1, 11):
❶     square = value ** 2
❷     squares.append(square)

print(squares)
```

我们首先创建一个空列表`squares`。然后，告诉Python使用`range()`函数循环遍历1到10之间的每个值。在循环内部，当前值被平方并赋值给变量`square` ❶。每次新的`square`值都会被追加到`squares`列表中 ❷。最后，当循环结束时，平方数的列表会被打印出来：

```py
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

为了使这段代码更简洁，可以省略临时变量`square`，并将每个新值直接添加到列表中：

```py
squares = []
for value in range(1,11):
    squares.append(value**2)

print(squares)
```

这一行完成了与前面列表中`for`循环内部相同的工作。循环中的每个值都会被平方，然后立即追加到平方数列表中。

在制作更复杂的列表时，你可以使用这两种方法中的任何一种。有时使用临时变量可以让你的代码更易读；有时则可能使代码变得冗长。首先关注编写自己清楚理解的代码，并确保它能按预期执行。然后，在复查代码时，寻找更高效的方法。

### 使用数字列表进行简单统计

在处理数字列表时，几个 Python 函数非常有用。例如，你可以轻松地找到数字列表中的最小值、最大值和总和：

```py
>>> **digits = [1, 2, 3, 4, 5, 6, 7, 8, 9, 0]**
>>> **min(digits)**
0
>>> **max(digits)**
9
>>> **sum(digits)**
45
```

### 列表推导式

前面描述的生成 `squares` 列表的方法需要使用三到四行代码。一个 *列表推导式* 可以让你仅用一行代码生成相同的列表。列表推导式将 `for` 循环和新元素的创建合并为一行，并自动附加每个新元素。列表推导式通常不会首先介绍给初学者，但我在这里包含它们，因为一旦你开始查看其他人的代码，你很可能会看到它们。

以下示例构建了与之前相同的平方数列表，但使用了列表推导式：

**squares.py**

```py
squares = [value**2 for value in range(1, 11)]
print(squares)
```

要使用这种语法，首先为列表取一个描述性名称，例如 `squares`。接着，打开一对方括号，定义你想存储到新列表中的表达式。在这个例子中，表达式是 `value**2`，它将值提升到二次方。然后，写一个 `for` 循环来生成你想要传入表达式的数字，最后关闭方括号。这个例子中的 `for` 循环是 `for value in range(1, 11)`，它将值 1 到 10 传入表达式 `value**2`。注意，在 `for` 语句末尾不使用冒号。

结果是你之前看到的相同的平方数列表：

```py
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

编写你自己的列表推导式需要练习，但一旦你熟悉了创建普通列表，你会发现它们非常值得。当你编写三到四行代码来生成列表，并开始觉得重复时，考虑编写你自己的列表推导式。

## 操作列表的一部分

在第三章，你学习了如何访问列表中的单个元素，而在本章，你正在学习如何遍历列表中的所有元素。你还可以操作列表中的特定项集合，这在 Python 中称为 *切片*。

### 切片列表

要制作切片，你需要指定你想操作的第一个和最后一个元素的索引。与 `range()` 函数一样，Python 会在你指定的第二个索引之前停止。所以，要输出列表中的前三个元素，你需要请求索引 `0` 到 `3`，这将返回元素 `0`、`1` 和 `2`。

以下示例涉及一个团队的玩家列表：

**players.py**

```py
players = ['charles', 'martina', 'michael', 'florence', 'eli']
print(players[0:3])
```

这段代码打印了列表的一个切片。输出保留了列表的结构，并包括了列表中的前三个玩家：

```py
['charles', 'martina', 'michael']
```

你可以生成列表的任何子集。例如，如果你想要列表中的第二、第三和第四个项目，你可以从索引`1`开始切片，结束于索引`4`：

```py
players = ['charles', 'martina', 'michael', 'florence', 'eli']
print(players[1:4])
```

这次切片从`'martina'`开始，直到`'florence'`结束：

```py
['martina', 'michael', 'florence']
```

如果你在切片中省略第一个索引，Python 会自动从列表的开头开始切片：

```py
players = ['charles', 'martina', 'michael', 'florence', 'eli']
print(players[:4])
```

没有起始索引时，Python 会从列表的开头开始：

```py
['charles', 'martina', 'michael', 'florence']
```

如果你想要包括列表末尾的切片，类似的语法也适用。例如，如果你想要从第三个项目到最后一个项目的所有项，可以从索引`2`开始，省略第二个索引：

```py
players = ['charles', 'martina', 'michael', 'florence', 'eli']
print(players[2:])
```

Python 会返回从第三个项目到列表末尾的所有项目：

```py
['michael', 'florence', 'eli']
```

这种语法允许你从列表的任何位置输出所有元素到列表的末尾，无论列表的长度如何。记住，负索引返回距离列表末尾一定距离的元素；因此，你可以从列表的末尾输出任何切片。例如，如果我们想要输出名册中的最后三位玩家，我们可以使用切片`players[-3:]`：

```py
players = ['charles', 'martina', 'michael', 'florence', 'eli']
print(players[-3:])
```

这会打印出最后三位玩家的名字，并且会随着玩家列表的变化继续有效。

### 遍历切片

如果你想要遍历列表中某个子集的元素，可以在`for`循环中使用切片。在下一个例子中，我们遍历前三个玩家，并将他们的名字作为简单名册的一部分打印出来：

```py
players = ['charles', 'martina', 'michael', 'florence', 'eli']

print("Here are the first three players on my team:")
❶ for player in players[:3]:
    print(player.title())
```

Python 并不是遍历整个玩家列表，而是只遍历前三个名字 ❶：

```py
Here are the first three players on my team:
Charles
Martina
Michael
```

切片在许多情况下非常有用。例如，当你创建游戏时，每次玩家完成游戏后，你可以将该玩家的最终得分添加到列表中。然后，你可以通过将列表按降序排序，并提取一个包含前三个得分的切片，来获取玩家的前三个得分。当你处理数据时，你可以使用切片将数据分成特定大小的块进行处理。或者，当你构建一个Web应用程序时，你可以使用切片在一系列页面中显示信息，每个页面显示适量的信息。

### 复制列表

通常，你会想从一个现有的列表开始，并根据第一个列表创建一个全新的列表。让我们来探索如何复制一个列表，并看看在什么情况下复制列表是有用的。

要复制一个列表，你可以通过省略第一个索引和第二个索引（`[:]`）来创建一个包含原始列表所有元素的切片。这告诉 Python 从第一个项目开始切片，到最后一个项目结束，从而生成整个列表的副本。

例如，假设我们有一个关于我们最喜欢的食物的列表，并想创建一个包含朋友喜欢的食物的单独列表。这个朋友喜欢我们列表中的所有食物，因此我们可以通过复制我们的列表来创建他们的列表：

**foods.py**

```py
my_foods = ['pizza', 'falafel', 'carrot cake']
❶ friend_foods = my_foods[:]

print("My favorite foods are:")
print(my_foods)

print("\nMy friend's favorite foods are:")
print(friend_foods)
```

首先，我们创建一个包含我们喜欢的食物的列表，叫做 `my_foods`。然后，我们创建一个新列表，叫做 `friend_foods`。我们通过请求 `my_foods` 的切片而不指定任何索引❶，并将复制品赋值给 `friend_foods`。当我们打印每个列表时，我们会看到它们都包含相同的食物：

```py
My favorite foods are:
['pizza', 'falafel', 'carrot cake']

My friend's favorite foods are:
['pizza', 'falafel', 'carrot cake']
```

为了证明我们实际上有两个独立的列表，我们将向每个列表中添加新的食物，并展示每个列表是如何追踪相应人物的最爱食物的：

```py
my_foods = ['pizza', 'falafel', 'carrot cake']
❶ friend_foods = my_foods[:]

❷ my_foods.append('cannoli')
❸ friend_foods.append('ice cream')

print("My favorite foods are:")
print(my_foods)

print("\nMy friend's favorite foods are:")
print(friend_foods)
```

我们将原始的 `my_foods` 中的项目复制到新列表 `friend_foods` 中，就像在之前的示例中一样❶。接下来，我们向每个列表中添加新食物：我们将 `'cannoli'` 添加到 `my_foods` ❷，将 `'ice cream'` 添加到 `friend_foods` ❸。然后我们打印这两个列表，看看这些食物是否出现在相应的列表中：

```py
My favorite foods are:
['pizza', 'falafel', 'carrot cake', 'cannoli']

My friend's favorite foods are:
['pizza', 'falafel', 'carrot cake', 'ice cream']
```

输出结果显示，`'cannoli'` 现在出现在我们的最爱食物列表中，但 `'ice cream'` 没有。我们可以看到，`'ice cream'` 现在出现在朋友的列表中，但 `'cannoli'` 没有。如果我们只是将 `friend_foods` 设置为 `my_foods`，我们就不会得到两个独立的列表。例如，下面是当你尝试在没有使用切片的情况下复制列表时发生的情况：

```py
my_foods = ['pizza', 'falafel', 'carrot cake']

# This doesn't work:
friend_foods = my_foods

my_foods.append('cannoli')
friend_foods.append('ice cream')

print("My favorite foods are:")
print(my_foods)

print("\nMy friend's favorite foods are:")
print(friend_foods)
```

我们不是将 `my_foods` 的副本赋给 `friend_foods`，而是将 `friend_foods` 设置为 `my_foods`。这种语法实际上告诉 Python，将新的变量 `friend_foods` 与已经与 `my_foods` 关联的列表关联起来，因此现在两个变量都指向同一个列表。因此，当我们将 `'cannoli'` 添加到 `my_foods` 时，它也会出现在 `friend_foods` 中。同样，`'ice cream'` 会出现在两个列表中，即使它似乎只被添加到了 `friend_foods`。

输出结果显示，两个列表现在是相同的，这不是我们想要的：

```py
My favorite foods are:
['pizza', 'falafel', 'carrot cake', 'cannoli', 'ice cream']

My friend's favorite foods are:
['pizza', 'falafel', 'carrot cake', 'cannoli', 'ice cream']
```

## 元组

列表适用于存储在程序运行期间可能会发生变化的项目集合。当你在网站上处理用户列表或游戏中的角色列表时，能够修改列表尤其重要。然而，有时你希望创建一个不能改变的项目列表。元组可以让你做到这一点。Python 将不能更改的值称为*不可变*，而不可变的列表称为*元组*。

### 定义元组

元组看起来就像列表，只不过使用圆括号而不是方括号。一旦定义了元组，你可以通过使用每个项的索引来访问个别元素，就像访问列表一样。

例如，如果我们有一个矩形，它的大小应该始终保持不变，我们可以通过将其尺寸放入元组中来确保其大小不会改变：

**dimensions.py**

```py
dimensions = (200, 50)
print(dimensions[0])
print(dimensions[1])
```

我们定义了元组 `dimensions`，使用圆括号而不是方括号。然后，我们使用与访问列表元素相同的语法，逐个打印元组中的每个元素：

```py
200
50
```

让我们看看如果尝试改变元组 `dimensions` 中的某个项目会发生什么：

```py
dimensions = (200, 50)
dimensions[0] = 250
```

这段代码试图更改第一个维度的值，但 Python 返回了类型错误。因为我们试图修改元组的值，而元组是不可更改的对象类型，Python 告诉我们不能为元组中的项赋予新值：

```py
Traceback (most recent call last):
  File "dimensions.py", line 2, in <module>
    dimensions[0] = 250
TypeError: 'tuple' object does not support item assignment
```

这是有益的，因为我们希望在代码行尝试更改矩形的维度时，Python 能引发错误。

### 遍历元组中的所有值

你可以像遍历列表一样，使用`for`循环遍历元组中的所有值：

```py
dimensions = (200, 50)
for dimension in dimensions:
    print(dimension)
```

Python 返回元组中的所有元素，就像它对待列表一样：

```py
200
50
```

### 重写元组

虽然你不能修改元组，但你可以为表示元组的变量赋予新值。例如，如果我们想改变这个矩形的维度，我们可以重新定义整个元组：

```py
dimensions = (200, 50)
print("Original dimensions:")
for dimension in dimensions:
    print(dimension)

dimensions = (400, 100)
print("\nModified dimensions:")
for dimension in dimensions:
    print(dimension)
```

前四行定义了原始元组并打印了初始维度。然后，我们将一个新元组与变量`dimensions`关联，并打印新值。这次 Python 不会引发错误，因为重新赋值变量是有效的：

```py
Original dimensions:
200
50

Modified dimensions:
400
100
```

与列表相比，元组是简单的数据结构。当你想存储一组在程序生命周期中不应更改的值时，使用元组。

## 代码风格

现在你正在编写更长的程序，学习如何一致地编写代码风格是一个好主意。花时间让你的代码尽可能易读。编写易读的代码有助于你跟踪程序的运行，并且也能帮助别人理解你的代码。

Python 程序员已达成共识，遵循一系列的代码风格规范，以确保每个人的代码结构大致相同。一旦你学会了编写干净的 Python 代码，你应该能够理解任何其他人写的 Python 代码，只要他们遵循相同的规范。如果你希望将来成为一名专业程序员，你应该尽早开始遵循这些规范，养成良好的编程习惯。

### 风格指南

当有人想对 Python 语言做出更改时，他们会写一份*Python 增强提案（PEP）*。最早的 PEP 之一是*PEP 8*，它指导 Python 程序员如何规范代码风格。PEP 8 内容相当长，但其中很多内容涉及到比你当前看到的更复杂的编码结构。

Python 的风格指南写作时考虑到代码比编写的更常被阅读。你编写代码一次后，就会在开始调试时反复阅读它。当你向程序添加新功能时，你会花更多时间阅读代码。当你与其他程序员分享代码时，他们也会阅读你的代码。

在写代码时，如果面临选择更容易写的代码和更容易读的代码之间的抉择，Python 程序员几乎总是会鼓励你写易读的代码。以下规范将帮助你从一开始就编写清晰的代码。

### 缩进

PEP 8 推荐每个缩进级别使用四个空格。使用四个空格可以提高可读性，同时在每一行留出空间以支持多个缩进级别。

在文字处理文档中，人们通常使用制表符而非空格来进行缩进。这对文字处理文档非常有效，但当制表符与空格混合时，Python 解释器会感到困惑。每个文本编辑器都提供一个设置，允许你使用 TAB 键，但会将每个制表符转换为一定数量的空格。你应该使用 TAB 键，但也要确保编辑器设置为将制表符转换为空格，而不是直接插入制表符。

在文件中混合使用制表符和空格可能会导致很难诊断的问题。如果你怀疑文件中混用了制表符和空格，大多数编辑器允许你将文件中的所有制表符转换为空格。

### 行长度

许多 Python 程序员建议每行字符数不应超过 80 个。这个指导方针最初源于大多数计算机只能在终端窗口的单行中显示 79 个字符。现在，人们可以在屏幕上显示更长的行，但仍然有其他原因支持遵循 79 字符的标准行长度。

专业程序员通常会在同一屏幕上打开多个文件，使用标准的行长度使他们能够同时查看屏幕上并排显示的两个或三个文件中的完整行。PEP 8 还建议将所有注释的每行字符数限制为 72 个，因为一些为大型项目生成自动文档的工具会在每行注释的开头添加格式字符。

PEP 8 关于行长度的指导方针并非一成不变，某些团队偏好99字符的限制。作为初学者，你不必过于担心代码行的长度，但要注意，协作开发的人员几乎总是遵循 PEP 8 的规范。大多数编辑器允许你设置一个视觉提示，通常是在屏幕上显示一条垂直线，帮助你了解行长度的限制。

### 空白行

为了在视觉上分组程序的各个部分，使用空白行。你应该使用空白行来组织文件，但不要过度使用。通过遵循本书中提供的示例，你应该能够找到合适的平衡。例如，如果你有五行代码用于构建一个列表，接着再有三行代码操作该列表，那么在这两部分之间插入一个空白行是合适的。然而，你不应在这两部分之间插入三到四个空白行。

空白行不会影响代码的运行，但会影响代码的可读性。Python 解释器使用水平缩进来解释代码的含义，但它忽略垂直空白。

### 其他样式指南

PEP 8 提供了许多额外的样式建议，但大多数指导原则适用于比你目前编写的更复杂的程序。当你学习更复杂的 Python 结构时，我会分享 PEP 8 指南中相关的部分。

## 总结

在本章中，你学会了如何高效地处理列表中的元素。你学会了如何使用 `for` 循环遍历列表，Python 如何使用缩进来结构化程序，以及如何避免一些常见的缩进错误。你学会了创建简单的数字列表，以及你可以对数字列表执行的一些操作。你学会了如何切片列表以处理子集项，以及如何通过切片正确地复制列表。你还学习了元组，它提供了一定的保护，防止一组值被改变，以及如何使你越来越复杂的代码具有可读性。

在第 5 章中，你将学习通过使用 `if` 语句来对不同的条件做出适当的反应。你将学会将相对复杂的条件测试组合起来，以准确应对你所寻找的特定情况或信息。你还将学习在循环遍历列表时使用 `if` 语句，以对列表中的特定元素执行特定操作。

# 5

`if` 语句

![](Images/chapterart.png)

编程通常涉及检查一组条件，并根据这些条件决定采取什么行动。Python 的 `if` 语句允许你检查程序的当前状态，并对该状态做出适当的响应。

在本章中，你将学习如何编写条件测试，以检查你感兴趣的任何条件。你将学会编写简单的 `if` 语句，还将学习如何创建更复杂的 `if` 语句系列，以识别出你需要的确切条件。然后，你将把这个概念应用到列表中，这样你就能够编写一个 `for` 循环，以一种方式处理列表中的大多数项，但以另一种方式处理具有特定值的某些项。

## 一个简单的例子

以下示例展示了 `if` 测试如何让你正确地响应特殊情况。假设你有一个汽车名称的列表，你想打印出每辆车的名字。汽车名字是专有名词，因此大多数车的名字应该以标题格式打印。然而，`'bmw'` 的值应该以全大写形式打印。以下代码会遍历汽车名称列表，查找值为 `'bmw'` 的项。每当遇到 `'bmw'` 时，它就会以大写形式打印，而不是标题格式：

**cars.py**

```py
cars = ['audi', 'bmw', 'subaru', 'toyota']

for car in cars:
❶     if car == 'bmw':
        print(car.upper())
    else:
        print(car.title())
```

这个示例中的循环首先检查 `car` 的当前值是否为 `'bmw'` ❶。如果是，它将值以大写形式打印。如果 `car` 的值不是 `'bmw'`，它将以标题格式打印：

```py
Audi
BMW
Subaru
Toyota
```

这个示例结合了你将在本章中学习的多个概念。让我们先看看你可以用来检查程序中条件的各种测试。

## 条件测试

每个`if`语句的核心是一个可以被评估为`True`或`False`的表达式，这被称为*条件测试*。Python使用`True`和`False`的值来决定是否执行`if`语句中的代码。如果条件测试评估为`True`，Python就会执行`if`语句后的代码。如果测试评估为`False`，Python则会忽略`if`语句后的代码。

### 检查相等性

大多数条件测试将一个变量的当前值与一个特定的目标值进行比较。最简单的条件测试检查一个变量的值是否等于目标值：

```py
>>> **car = 'bmw'**
>>> **car == 'bmw'**
True
```

第一行使用单等号将`car`的值设置为`'bmw'`，你已经看到了很多次。接下来的一行使用双等号(`==`)检查`car`的值是否为`'bmw'`。这个*相等运算符*如果左右两边的值相同，则返回`True`，如果不同，则返回`False`。在这个例子中，两个值是匹配的，所以Python返回`True`。

当`car`的值不是`'bmw'`时，这个测试返回`False`：

```py
>>> **car = 'audi'**
>>> **car == 'bmw'**
False
```

一个等号实际上是一个赋值语句；你可以把这里的第一行代码读作“将`car`的值设置为`'audi'`。”另一方面，双等号是在问一个问题：“`car`的值是否等于`'bmw'`？”大多数编程语言都以这种方式使用等号。

### 检查相等性时忽略大小写

在Python中，检查相等性是区分大小写的。例如，两个大小写不同的值不会被认为是相等的：

```py
>>> **car = 'Audi'**
>>> **car == 'audi'**
False
```

如果大小写很重要，这种行为是有利的。但是，如果大小写不重要，而你只想测试变量的值，可以在进行比较之前将变量的值转换为小写：

```py
>>> **car = 'Audi'**
>>> **car.lower() == 'audi'**
True
```

这个测试会返回`True`，无论`'Audi'`的格式如何，因为现在测试是大小写不敏感的。`lower()`方法不会改变最初存储在`car`中的值，因此你可以在不影响原始变量的情况下进行这种比较：

```py
>>> **car = 'Audi'**
>>> **car.lower() == 'audi'**
True
>>> **car**
'Audi'
```

我们首先将大写字符串`'Audi'`赋值给变量`car`。然后，我们将`car`的值转换为小写，并将小写值与字符串`'audi'`进行比较。两个字符串匹配，所以Python返回`True`。我们可以看到，存储在`car`中的值没有受到`lower()`方法的影响。

网站会像这样强制执行用户输入数据的某些规则。例如，一个网站可能会使用类似的条件测试来确保每个用户都有一个真正唯一的用户名，而不仅仅是另一个人用户名的大小写变化。当有人提交一个新用户名时，这个新用户名会被转换为小写，并与所有现有用户名的小写版本进行比较。在这个检查过程中，如果`'John'`的任何变体（如`'john'`）已经被使用，新的用户名将被拒绝。

### 检查不相等性

当你想确定两个值是否不相等时，可以使用*不等运算符*（`!=`）。我们再用一个`if`语句来演示如何使用不等运算符。我们将把请求的披萨配料存储在一个变量中，然后如果这个人没有点凤尾鱼，就打印一条消息：

**toppings.py**

```py
requested_topping = 'mushrooms'

if requested_topping != 'anchovies':
    print("Hold the anchovies!")
```

这段代码将`requested_topping`的值与`'anchovies'`进行比较。如果这两个值不匹配，Python返回`True`并执行`if`语句后的代码。如果两个值匹配，Python返回`False`，并且不会执行`if`语句后的代码。

由于`requested_topping`的值不是`'anchovies'`，因此执行了`print()`函数：

```py
Hold the anchovies!
```

你编写的大多数条件表达式都会测试相等性，但有时你会发现测试不等式更高效。

### 数值比较

测试数值是相当简单的。例如，下面的代码检查一个人是否已经18岁：

```py
>>> **age = 18**
>>> **age == 18**
True
```

你也可以测试两个数字是否不相等。例如，下面的代码如果给定的答案不正确，就会打印一条消息：

**magic_number.py**

```py
answer = 17
if answer != 42:
    print("That is not the correct answer. Please try again!")
```

条件测试通过，因为`answer`（`17`）的值不等于`42`。由于测试通过，缩进的代码块被执行：

```py
That is not the correct answer. Please try again!
```

你还可以在条件语句中包含各种数学比较，例如小于、小于或等于、大于、大于或等于：

```py
>>> **age = 19**
>>> **age < 21**
True
>>> **age <= 21**
True
>>> **age > 21**
False
>>> **age >= 21**
False
```

每个数学比较都可以作为`if`语句的一部分，这有助于你检测感兴趣的具体条件。

### 检查多个条件

你可能希望同时检查多个条件。例如，有时你可能需要两个条件都为`True`才能采取某个动作。其他时候，你可能只需要一个条件为`True`就满足了。`and`和`or`关键字可以在这些情况下帮到你。

#### 使用`and`检查多个条件

要检查两个条件是否同时都为`True`，可以使用`and`关键字将两个条件测试结合起来；如果每个测试都通过，整体表达式就为`True`。如果任何一个测试失败，或者两个测试都失败，表达式就为`False`。

例如，你可以使用以下测试来检查两个人是否都超过21岁：

```py
>>> **age_0 = 22**
>>> **age_1 = 18**
❶ >>> **age_0 >= 21 and age_1 >= 21**
False
❷ >>> **age_1 = 22**
>>> **age_0 >= 21 and age_1 >= 21**
True
```

首先，我们定义两个年龄，`age_0`和`age_1`。然后我们检查这两个年龄是否都大于或等于21岁❶。左边的测试通过了，但右边的测试失败了，因此整个条件表达式的结果为`False`。然后我们将`age_1`改为22❷。`age_1`的值现在大于21，因此两个单独的测试都通过了，导致整个条件表达式的结果为`True`。

为了提高可读性，你可以在单独的测试条件周围使用括号，但这不是必需的。如果使用括号，测试将像这样：

```py
(age_0 >= 21) and (age_1 >= 21)
```

#### 使用`or`检查多个条件

关键字 `or` 允许你检查多个条件，但只要其中一个或两个测试通过，它就会通过。`or` 表达式只有在两个测试都失败时才会失败。

让我们再次考虑两个年龄，但这次我们只关注是否有一个人超过 21 岁：

```py
>>> **age_0 = 22**
>>> **age_1 = 18**
❶ >>> **age_0 >= 21 or age_1 >= 21**
True
❷ >>> **age_0 = 18**
>>> **age_0 >= 21 or age_1 >= 21**
False
```

我们再次开始使用两个年龄变量。由于对 `age_0` ❶ 的测试通过，整个表达式评估为 `True`。然后我们将 `age_0` 降到 18。在最后的测试 ❷ 中，两个测试都失败，整个表达式评估为 `False`。

### 检查值是否在列表中

有时，在执行某个操作之前，检查列表是否包含某个值很重要。例如，在完成某人注册之前，你可能想检查一个新的用户名是否已经存在于当前的用户名列表中。在一个制图项目中，你可能想检查一个提交的地点是否已经存在于已知地点列表中。

要检查某个特定值是否已经在列表中，可以使用关键字 `in`。让我们考虑一些你可能为披萨店编写的代码。我们将创建一个顾客请求的披萨配料列表，然后检查某些配料是否在该列表中。

```py
>>> **requested_toppings = ['mushrooms', 'onions', 'pineapple']**
>>> **'mushrooms' in requested_toppings**
True
>>> **'pepperoni' in requested_toppings**
False
```

关键字 `in` 告诉 Python 检查 `'mushrooms'` 和 `'pepperoni'` 是否存在于列表 `requested_toppings` 中。这种技术非常强大，因为你可以创建一个包含基本值的列表，然后轻松检查你正在测试的值是否与列表中的某个值匹配。

### 检查值是否不在列表中

有时，了解某个值是否不存在于列表中也很重要。在这种情况下，你可以使用关键字 `not`。例如，考虑一个在论坛中被禁止评论的用户列表。你可以在允许用户提交评论之前，检查该用户是否已被禁止：

**banned_users.py**

```py
banned_users = ['andrew', 'carolina', 'david']
user = 'marie'

if user not in banned_users:
    print(f"{user.title()}, you can post a response if you wish.")
```

这里的 `if` 语句很清晰。如果 `user` 的值不在 `banned_users` 列表中，Python 返回 `True` 并执行缩进的语句。

用户 `'marie'` 不在 `banned_users` 列表中，因此她会看到一条邀请她发布回应的消息：

```py
Marie, you can post a response if you wish.
```

### 布尔表达式

当你进一步学习编程时，你会在某个时刻听到 *布尔表达式* 这个术语。布尔表达式就是条件测试的另一种说法。*布尔值* 只有 `True` 或 `False`，就像条件表达式在评估后的值一样。

布尔值常用于跟踪某些条件，例如游戏是否正在运行，或用户是否可以编辑网站上的某些内容：

```py
game_active = True
can_edit = False
```

布尔值提供了一种高效的方式来跟踪程序的状态或程序中某个重要条件的状态。

## if 语句

当你理解了条件测试后，你可以开始编写 `if` 语句。存在几种不同类型的 `if` 语句，你选择使用哪种类型取决于你需要测试的条件数量。在关于条件测试的讨论中，你已经看到了几个 `if` 语句的例子，现在让我们更深入地探讨这个话题。

### 简单的 if 语句

最简单的 `if` 语句类型有一个测试和一个操作：

```py
if `conditional_test`:
    `do something`
```

你可以在第一行中放置任何条件测试，并且在测试后面的缩进块中放置几乎任何操作。如果条件测试的结果为 `True`，Python 将执行 `if` 语句后的代码。如果测试结果为 `False`，Python 会忽略 `if` 语句后的代码。

假设我们有一个表示某人年龄的变量，我们想知道这个人是否足够大，可以投票。以下代码测试该人是否可以投票：

**voting.py**

```py
age = 19
if age >= 18:
    print("You are old enough to vote!")
```

Python 检查 `age` 的值是否大于或等于 18。它是成立的，因此 Python 执行缩进的 `print()` 调用：

```py
You are old enough to vote!
```

缩进在 `if` 语句中的作用与在 `for` 循环中相同。`if` 语句后的所有缩进行将会在测试通过时执行，如果测试不通过，则整个缩进块会被忽略。

你可以在 `if` 语句后的块中放置任意多行代码。如果该人足够大，可以投票，我们可以添加另一行输出，询问该人是否已经注册投票：

```py
age = 19
if age >= 18:
 print("You are old enough to vote!")
    print("Have you registered to vote yet?")
```

条件测试通过，且两个 `print()` 调用都已缩进，因此两行都被打印出来：

```py
You are old enough to vote!
Have you registered to vote yet?
```

如果 `age` 的值小于 18，则此程序将不会产生任何输出。

### if-else 语句

通常，你希望在条件测试通过时执行某个操作，在所有其他情况下执行不同的操作。Python 的 `if`-`else` 语法使得这一点成为可能。`if`-`else` 块类似于简单的 `if` 语句，但 `else` 语句允许你定义在条件测试失败时执行的操作或操作集。

如果一个人足够大可以投票，我们将显示与之前相同的消息，但这次我们还会为那些不够大不能投票的人添加一条消息：

```py
age = 17
❶ if age >= 18:
 print("You are old enough to vote!")
 print("Have you registered to vote yet?")
❷ else:
    print("Sorry, you are too young to vote.")
    print("Please register to vote as soon as you turn 18!")
```

如果条件测试 ❶ 通过，则执行第一个缩进的 `print()` 调用块。如果测试结果为 `False`，则执行 `else` 块 ❷。因为这次 `age` 小于 18，条件测试失败，执行 `else` 块中的代码：

```py
Sorry, you are too young to vote.
Please register to vote as soon as you turn 18!
```

这段代码有效，因为它只有两种可能的情况：一个人要么足够大可以投票，要么不够大不能投票。`if`-`else` 结构在你希望 Python 总是执行两种可能操作之一的情况下工作得很好。在像这样的简单 `if-else` 链中，总会执行两种操作中的一种。

### if-elif-else 链

通常，你需要测试两个以上的可能情况，而要评估这些情况，你可以使用Python的`if`-`elif`-`else`语法。Python只会在`if`-`elif`-`else`链中执行一个代码块。它按顺序运行每个条件测试，直到某个测试通过。当某个测试通过时，紧跟着该测试的代码会被执行，然后Python会跳过其余的测试。

许多现实世界的情况涉及两个以上的可能条件。例如，考虑一个根据不同年龄段收取不同费用的游乐园：

+   任何未满4岁的人可以免费入场。

+   任何4岁至18岁之间的人入场费用为$25。

+   任何18岁或以上的人入场费用为$40。

我们如何使用`if`语句来确定一个人的入场费用？以下代码测试一个人的年龄段，然后打印入场价格消息：

**amusement_park.py**

```py
age = 12
❶ if age < 4:
    print("Your admission cost is $0.")
❷ elif age < 18:
    print("Your admission cost is $25.")
❸ else:
    print("Your admission cost is $40.")
```

`if`测试❶检查一个人是否未满4岁。当测试通过时，会打印相应的消息，Python跳过剩余的测试。`elif`行❷实际上是另一个`if`测试，只有在前一个测试失败时才会执行。在链中的这一点，我们知道这个人至少4岁，因为第一个测试失败了。如果这个人未满18岁，会打印相应的消息，Python跳过`else`块。如果`if`和`elif`测试都失败，Python会执行`else`块中的代码❸。

在这个例子中，`if`测试❶的结果为`False`，因此它的代码块没有被执行。然而，`elif`测试的结果为`True`（12小于18），因此它的代码被执行。输出是一个句子，通知用户入场费用：

```py
Your admission cost is $25.
```

任何大于17岁的年龄都会导致前两个测试失败。在这些情况下，`else`块会被执行，入场费用为$40。

与其在`if`-`elif`-`else`块中打印入场价格，不如在`if`-`elif`-`else`链中仅设置价格，然后在链评估后执行一个单独的`print()`调用，这样更简洁：

```py
age = 12

if age < 4:
    price = 0
elif age < 18:
    price = 25
else:
    price = 40

print(f"Your admission cost is ${price}.")
```

缩进的行根据人的年龄设置`price`的值，就像前一个示例一样。在`if`-`elif`-`else`链设置了价格后，一个独立的未缩进的`print()`调用使用这个值来显示一条消息，报告这个人的入场价格。

这段代码产生与前一个示例相同的输出，但`if`-`elif`-`else`链的作用更窄。它不是用来确定价格并显示消息，而是简单地确定入场价格。除了更高效外，修改后的代码也比原始方法更容易修改。如果要更改输出消息的文本，只需要修改一个`print()`调用，而不是三个单独的`print()`调用。

### 使用多个`elif`块

你可以在代码中使用任意多个`elif`块。例如，如果游乐园实施了老年人折扣，你可以在代码中添加一个条件测试来确定某人是否符合老年人折扣的条件。假设65岁或以上的人只需支付常规入场费的一半，即20美元：

```py
age = 12

if age < 4:
 price = 0
elif age < 18:
 price = 25
elif age < 65:
    price = 40
else:
    price = 20

print(f"Your admission cost is ${price}.")
```

这段代码的大部分内容保持不变。第二个`elif`块现在会检查确保一个人的年龄小于65岁，然后才会将其分配为40美元的全额入场费。请注意，`else`块中分配的值需要改为20美元，因为只有年龄在65岁及以上的人才会进入这个块。

### 忽略`else`块

Python并不要求在`if`-`elif`链的末尾必须有`else`块。有时，`else`块是有用的。其他时候，使用额外的`elif`语句来捕获感兴趣的特定条件会更清晰：

```py
age = 12

if age < 4:
 price = 0
elif age < 18:
 price = 25
elif age < 65:
 price = 40
elif age >= 65:
    price = 20

print(f"Your admission cost is ${price}.")
```

最后的`elif`块在一个人65岁或以上时分配20美元的价格，这比一般的`else`块更清晰。通过这个变化，每一块代码必须通过一个特定的测试才能执行。

`else`块是一个兜底语句。它匹配任何没有被特定`if`或`elif`测试匹配的条件，这有时可能包括无效或甚至恶意的数据。如果你有一个具体的最终条件要测试，可以考虑使用最后一个`elif`块，并省略`else`块。这样，你会更有信心代码只会在正确的条件下运行。

### 测试多个条件

`if`-`elif`-`else`链非常强大，但仅在你只需要通过一个测试时才适合使用。一旦Python找到一个通过的测试，它就会跳过剩下的测试。这种行为是有益的，因为它高效，并且允许你仅测试一个特定的条件。

然而，有时检查所有感兴趣的条件是很重要的。在这种情况下，你应该使用一系列简单的`if`语句，而不使用`elif`或`else`块。当多个条件可能为`True`时，这种技术很有意义，并且你希望对每个为`True`的条件做出响应。

让我们重新考虑披萨店的例子。如果有人请求一个双配料的披萨，你需要确保披萨上有两个配料：

**toppings.py**

```py
requested_toppings = ['mushrooms', 'extra cheese']

if 'mushrooms' in requested_toppings:
    print("Adding mushrooms.")
❶ if 'pepperoni' in requested_toppings:
    print("Adding pepperoni.")
if 'extra cheese' in requested_toppings:
    print("Adding extra cheese.")

print("\nFinished making your pizza!")
```

我们从一个包含请求的配料的列表开始。第一个`if`语句检查这个人是否请求了蘑菇。如果是这样，则打印一条消息确认这个配料。辣香肠的测试❶是另一个简单的`if`语句，而不是`elif`或`else`语句，因此无论前面的测试是否通过，这个测试都会被执行。最后一个`if`语句检查是否请求了额外的奶酪，无论前两个测试的结果如何。这三个独立的测试每次运行这个程序时都会执行。

因为在这个例子中每个条件都会被评估，所以蘑菇和额外的奶酪都会被添加到披萨上：

```py
Adding mushrooms.
Adding extra cheese.

Finished making your pizza!
```

如果我们使用`if`-`elif`-`else`块，这段代码将无法正常工作，因为代码在只有一个测试通过后就会停止运行。下面是它的样子：

```py
requested_toppings = ['mushrooms', 'extra cheese']

if 'mushrooms' in requested_toppings:
 print("Adding mushrooms.")
elif 'pepperoni' in requested_toppings:
    print("Adding pepperoni.")
elif 'extra cheese' in requested_toppings:
    print("Adding extra cheese.")

print("\nFinished making your pizza!")
```

对`'mushrooms'`的测试是第一个通过的测试，所以蘑菇被添加到比萨上。然而，值`'extra cheese'`和`'pepperoni'`永远不会被检查，因为在`if-elif-else`链中，Python不会运行通过的第一个测试后的任何其他测试。顾客的第一个配料会被添加，但所有其他配料都会被遗漏：

```py
Adding mushrooms.

Finished making your pizza!
```

总结来说，如果你希望只有一个代码块运行，使用`if`-`elif`-`else`链。如果需要多个代码块运行，使用一系列独立的`if`语句。

## 在列表中使用`if`语句

当你将列表和`if`语句结合起来时，可以做一些有趣的工作。你可以观察需要与列表中其他值不同对待的特殊值。你可以有效地管理变化的条件，比如餐厅在一个班次中的某些物品的可用性。你还可以开始证明你的代码在所有可能的情况下都能按预期工作。

### 检查特殊项

本章开始时展示了一个简单的例子，演示了如何处理像`'bmw'`这样的特殊值，特别是它需要以不同于其他列表值的格式打印。现在你已经对条件测试和`if`语句有了基本的理解，我们来仔细看看如何在列表中检查特殊值并适当地处理这些值。

让我们继续以比萨店的例子。每当在制作比萨时添加配料，比萨店会显示一条消息。这个操作的代码可以通过列出顾客请求的配料，并使用循环在每次配料添加到比萨上时宣布它，来高效地编写：

**toppings.py**

```py
requested_toppings = ['mushrooms', 'green peppers', 'extra cheese']

for requested_topping in requested_toppings:
    print(f"Adding {requested_topping}.")

print("\nFinished making your pizza!")
```

输出是直观的，因为这段代码只是一个简单的`for`循环：

```py
Adding mushrooms.
Adding green peppers.
Adding extra cheese.

Finished making your pizza!
```

那如果比萨店没有绿椒了怎么办？`for`循环中的`if`语句可以适当地处理这种情况：

```py
requested_toppings = ['mushrooms', 'green peppers', 'extra cheese']

for requested_topping in requested_toppings:
    if requested_topping == 'green peppers':
        print("Sorry, we are out of green peppers right now.")
    else:
        print(f"Adding {requested_topping}.")

print("\nFinished making your pizza!")
```

这次，在将配料添加到比萨之前，我们检查每个请求的项。`if`语句检查顾客是否请求了绿椒。如果是，我们会显示一条消息告知他们为什么不能要绿椒。`else`块确保所有其他配料都将添加到比萨上。

输出显示每个请求的配料都得到了适当的处理。

```py
Adding mushrooms.
Sorry, we are out of green peppers right now.
Adding extra cheese.

Finished making your pizza!
```

### 检查列表是否为空

到目前为止，我们对每个使用过的列表做了一个简单的假设：我们假设每个列表至少有一个项目。很快，我们将允许用户提供存储在列表中的信息，因此每次运行循环时，我们不能假设列表中有任何项目。在这种情况下，在运行`for`循环之前检查列表是否为空是很有用的。

作为示例，让我们在制作披萨之前检查请求的配料列表是否为空。如果列表为空，我们将提示用户并确认他们是否想要一份普通披萨。如果列表不为空，我们将像之前的示例一样制作披萨：

```py
requested_toppings = []

if requested_toppings:
    for requested_topping in requested_toppings:
 print(f"Adding {requested_topping}.")
 print("\nFinished making your pizza!")
else:
    print("Are you sure you want a plain pizza?")
```

这次我们从一个空的请求配料列表开始。我们没有直接进入 `for` 循环，而是先做一个快速检查。当列表的名称在 `if` 语句中使用时，如果列表包含至少一个项，Python 会返回 `True`；如果是空列表，则返回 `False`。如果 `requested_toppings` 通过条件测试，我们就运行与前一个示例中相同的 `for` 循环。如果条件测试失败，我们会打印一条消息，询问顾客是否真的想要没有配料的普通披萨。

在这种情况下，列表为空，因此输出会询问用户是否真的想要一份普通披萨：

```py
Are you sure you want a plain pizza?
```

如果列表不为空，输出将显示每个请求的配料被添加到披萨上。

### 使用多个列表

人们对配料的请求几乎无所不包，特别是涉及披萨配料时。如果顾客真的想要在披萨上放薯条呢？你可以使用列表和 `if` 语句，确保在执行操作之前输入是合理的。

在制作披萨之前，让我们留意一些不寻常的配料请求。以下示例定义了两个列表。第一个是披萨店提供的配料列表，第二个是用户请求的配料列表。这一次，`requested_toppings` 中的每个项在添加到披萨之前都会与可用配料列表进行检查：

```py
available_toppings = ['mushrooms', 'olives', 'green peppers',
                      'pepperoni', 'pineapple', 'extra cheese']

❶ requested_toppings = ['mushrooms', 'french fries', 'extra cheese']

for requested_topping in requested_toppings:
❷     if requested_topping in available_toppings:
        print(f"Adding {requested_topping}.")
❸     else:
        print(f"Sorry, we don't have {requested_topping}.")

print("\nFinished making your pizza!")
```

首先，我们定义了这家披萨店提供的配料列表。请注意，如果披萨店提供稳定的配料选择，这个可以是一个元组。然后，我们创建了一个客户请求的配料列表。在这个示例中，有一个不寻常的配料请求：`'french fries'` ❶。接下来，我们遍历请求的配料列表。在循环内，我们检查每个请求的配料是否实际上在可用配料列表中 ❷。如果有，我们就将该配料添加到披萨上。如果请求的配料不在可用配料列表中，`else` 块将执行 ❸。`else` 块会打印一条消息，告诉用户哪些配料不可用。

这段代码语法产生了干净且富有信息的输出：

```py
Adding mushrooms.
Sorry, we don't have french fries.
Adding extra cheese.

Finished making your pizza!
```

仅凭几行代码，我们就有效地处理了一个现实世界中的情况！

## 给你的 `if` 语句添加样式

在本章的每个示例中，你都看到了良好的样式习惯。PEP 8 对条件测试样式的唯一建议是，在比较运算符（如 `==`、`>=` 和 `<=`）周围使用单个空格。例如：

```py
if age < 4:
```

比下面这种写法更好：

```py
if age<4:
```

这样的空格不会影响 Python 解释代码的方式；它只是让你的代码更容易阅读，无论是你自己还是其他人。

## 总结

在本章中，你学习了如何编写条件测试，这些测试总是返回`True`或`False`。你学习了如何编写简单的`if`语句、`if`-`else`链式结构以及`if`-`elif`-`else`链式结构。你开始使用这些结构来识别你需要测试的特定条件，并知道在你的程序中何时满足这些条件。你学会了如何在列表中处理某些项目与其他项目不同，同时继续利用`for`循环的高效性。你还重新回顾了Python的编码风格建议，以确保你日益复杂的程序仍然相对容易阅读和理解。

在第6章中，你将了解Python的字典。字典类似于列表，但它允许你将信息片段连接在一起。你将学习如何构建字典、遍历字典，并将它们与列表和`if`语句结合使用。学习字典将使你能够建模更广泛的现实世界情况。

# 6

字典

![](Images/chapterart.png)

在本章中，你将学习如何使用Python的字典，它们允许你将相关信息连接在一起。你将学习如何访问存储在字典中的信息，并如何修改这些信息。由于字典可以存储几乎无限量的信息，我将向你展示如何遍历字典中的数据。此外，你将学习如何将字典嵌套在列表中，将列表嵌套在字典中，甚至将字典嵌套在其他字典中。

理解字典可以让你更准确地建模各种现实世界中的对象。你可以创建一个字典来表示一个人，并存储关于这个人的所有信息。你可以存储他们的名字、年龄、地点、职业以及任何你能描述的关于这个人的其他方面。你将能够存储任何两种可以配对的信息，比如单词及其意思、人的名字和他们的最爱数字、山脉及其海拔等等。

## 一个简单的字典

考虑一个包含外星人的游戏，外星人可以具有不同的颜色和点数。这个简单的字典存储了一个特定外星人的信息：

**alien.py**

```py
alien_0 = {'color': 'green', 'points': 5}

print(alien_0['color'])
print(alien_0['points'])
```

字典`alien_0`存储了外星人的颜色和点数。最后两行访问并显示了这些信息，如下所示：

```py
green
5
```

与大多数新的编程概念一样，使用字典需要练习。一旦你使用字典一段时间后，你会发现它们在建模现实世界情况方面是多么高效。

## 使用字典

在Python中，*字典*是一种由*键-值对*组成的集合。每个*键*与一个值相连，你可以使用键来访问与该键关联的值。一个键的值可以是一个数字、一个字符串、一个列表，甚至是另一个字典。实际上，你可以使用Python中可以创建的任何对象作为字典中的值。

在Python中，字典是用大括号`{}`包裹的，里面包含一系列键值对，如之前的示例所示：

```py
alien_0 = {'color': 'green', 'points': 5}
```

*键值对*是一组相互关联的值。当你提供一个键时，Python会返回与该键相关联的值。每个键通过冒号与其值连接，单独的键值对通过逗号分隔。你可以在字典中存储任意数量的键值对。

最简单的字典只有一个键值对，如下所示，这是修改后的`alien_0`字典版本：

```py
alien_0 = {'color': 'green'}
```

这个字典存储了关于`alien_0`的一条信息：外星人的颜色。字符串`'color'`是字典中的一个键，而与之关联的值是`'green'`。

### 访问字典中的值

要获取与键相关联的值，给出字典的名称，然后将键放在一对方括号内，如下所示：

**alien.py**

```py
alien_0 = {'color': 'green'}
print(alien_0['color'])
```

这将返回字典`alien_0`中与键`'color'`关联的值：

```py
green
```

你可以在字典中拥有无限数量的键值对。例如，这里是原始的`alien_0`字典，包含两个键值对：

```py
alien_0 = {'color': 'green', 'points': 5}
```

现在你可以访问`alien_0`的颜色或积分值。如果玩家击败了这个外星人，你可以使用如下代码查找他们应该获得多少积分：

```py
alien_0 = {'color': 'green', 'points': 5}

new_points = alien_0['points']
print(f"You just earned {new_points} points!")
```

一旦字典被定义，我们就从字典中提取与键`'points'`相关联的值。然后将这个值赋给变量`new_points`。最后一行打印出玩家刚刚获得的积分：

```py
You just earned 5 points!
```

如果你每次击败外星人时都运行这段代码，外星人的积分值将会被检索出来。

### 添加新键值对

字典是动态结构，你可以随时向字典添加新的键值对。要添加新的键值对，你只需给出字典的名称，后跟新键的方括号，并指定新值。

让我们向`alien_0`字典添加两个新的信息：外星人的*x*坐标和*y*坐标，这将帮助我们在屏幕上的特定位置显示外星人。我们将外星人放置在屏幕的左边缘，距离顶部25像素。由于屏幕坐标通常从屏幕的左上角开始，我们通过将*x*坐标设置为0来把外星人放在屏幕的左边缘，并通过将*y*坐标设置为正25来把它放置在距离顶部25像素的位置，如下所示：

**alien.py**

```py
alien_0 = {'color': 'green', 'points': 5}
print(alien_0)

alien_0['x_position'] = 0
alien_0['y_position'] = 25
print(alien_0)
```

我们首先定义与之前相同的字典，然后打印这个字典，展示其信息快照。接下来，我们向字典中添加一个新的键值对：键`'x_position'`和值`0`。我们对键`'y_position'`做同样的操作。当我们打印修改后的字典时，我们看到这两个新增的键值对：

```py
{'color': 'green', 'points': 5}
{'color': 'green', 'points': 5, 'x_position': 0, 'y_position': 25}
```

字典的最终版本包含四个键值对。原来的两个指定了颜色和得分值，再加上两个指定了外星人位置的键值对。

字典会保留定义时的顺序。当你打印字典或遍历其元素时，你会看到元素按添加到字典的顺序出现。

### 从一个空字典开始

有时，开始时使用一个空字典，然后再逐个添加新的项是方便的，甚至是必要的。要开始填充一个空字典，定义一个空的大括号字典，然后将每个键值对单独添加在每一行。例如，以下是如何使用这种方法构建`alien_0`字典：

**alien.py**

```py
alien_0 = {}

alien_0['color'] = 'green'
alien_0['points'] = 5

print(alien_0)
```

我们首先定义一个空的`alien_0`字典，然后向其中添加颜色和得分值。结果是我们在之前示例中使用的字典：

```py
{'color': 'green', 'points': 5}
```

通常，在存储用户提供的数据时，或者在编写生成大量键值对的代码时，你会使用空字典。

### 修改字典中的值

要修改字典中的值，给出字典的名称并用方括号括住键，然后指定你希望与该键关联的新值。例如，考虑一个外星人，在游戏进程中从绿色变为黄色：

**alien.py**

```py
alien_0 = {'color': 'green'}
print(f"The alien is {alien_0['color']}.")

alien_0['color'] = 'yellow'
print(f"The alien is now {alien_0['color']}.")
```

我们首先定义一个仅包含外星人颜色的`alien_0`字典；然后我们将与`'color'`键关联的值更改为`'yellow'`。输出显示外星人确实从绿色变成了黄色：

```py
The alien is green.
The alien is now yellow.
```

对于一个更有趣的例子，让我们跟踪一个可以以不同速度移动的外星人。我们将存储一个表示外星人当前速度的值，然后用它来决定外星人应该向右移动多远：

```py
alien_0 = {'x_position': 0, 'y_position': 25, 'speed': 'medium'}
print(f"Original position: {alien_0['x_position']}")

# Move the alien to the right.
# Determine how far to move the alien based on its current speed.
❶ if alien_0['speed'] == 'slow':
    x_increment = 1
elif alien_0['speed'] == 'medium':
    x_increment = 2
else:
    # This must be a fast alien.
    x_increment = 3

# The new position is the old position plus the increment.
❷ alien_0['x_position'] = alien_0['x_position'] + x_increment

print(f"New position: {alien_0['x_position']}")
```

我们首先定义一个外星人，初始的*x*位置和*y*位置为`'medium'`速度。为了简化，我们省略了颜色和得分值，但如果你包括这些键值对，示例仍然适用。我们还打印了`x_position`的原始值，看看外星人向右移动了多少。

一个`if`-`elif`-`else`链决定外星人应该向右移动多远，并将这个值赋给变量`x_increment` ❶。如果外星人的速度是`'slow'`，它向右移动一个单位；如果速度是`'medium'`，它向右移动两个单位；如果是`'fast'`，它向右移动三个单位。计算出增量后，它会加到`x_position` ❷的值上，并将结果存储在字典的`x_position`中。

由于这是一个中等速度的外星人，它的位置向右移动了两个单位：

```py
Original x-position: 0
New x-position: 2
```

这个技巧非常酷：通过改变外星人字典中的一个值，你可以改变外星人的整体行为。例如，要将这个中等速度的外星人变成一个快速的外星人，你需要添加这一行：

```py
alien_0['speed'] = 'fast'
```

`if`-`elif`-`else` 代码块将在下一次代码运行时将更大的值分配给 `x_increment`。

### 删除键值对

当你不再需要字典中存储的信息时，可以使用 `del` 语句完全删除一个键值对。`del` 只需要字典的名称和你想要删除的键。

例如，假设我们从 `alien_0` 字典中删除键 `'points'`，以及它的值：

**alien.py**

```py
alien_0 = {'color': 'green', 'points': 5}
print(alien_0)

❶ del alien_0['points']
print(alien_0)
```

`del` 语句❶告诉 Python 删除字典 `alien_0` 中的键 `'points'`，并移除与该键关联的值。输出显示，键 `'points'` 和其值 `5` 被从字典中删除，但字典中的其他部分没有受到影响：

```py
{'color': 'green', 'points': 5}
{'color': 'green'}
```

### 相似对象的字典

前一个示例涉及存储关于一个对象（游戏中的外星人）的不同信息。你还可以使用字典存储关于许多对象的一种信息。例如，假设你想对一些人进行调查，询问他们最喜欢的编程语言是什么。字典非常适合存储这种简单调查的结果，如下所示：

**favorite_languages.py**

```py
favorite_languages = {
    'jen': 'python',
    'sarah': 'c',
    'edward': 'rust',
    'phil': 'python',
    }
```

如你所见，我们将一个较大的字典分成了几行。每个键都是参与调查的人的名字，每个值是他们选择的语言。当你知道定义字典时需要多行时，按 ENTER 键后，输入一个新的左大括号。接着将下一行缩进一个级别（四个空格），然后写入第一个键值对，后面跟着一个逗号。从此以后，每次按 ENTER 键时，你的文本编辑器应自动缩进所有后续的键值对，以匹配第一个键值对。

一旦你完成了字典的定义，在最后一个键值对后添加一个闭括号，并在新的一行上进行缩进，使其与字典中的键对齐。将逗号添加到最后一个键值对后也是一个好习惯，这样你就可以在下一行准备添加新的键值对。

为了使用这个字典，给定一个参与调查的人的名字，你可以轻松查找他们最喜欢的编程语言：

**favorite_languages.py**

```py
favorite_languages = {
 'jen': 'python',
 'sarah': 'c',
 'edward': 'rust',
 'phil': 'python',
 }

❶ language = favorite_languages['sarah'].title()
print(f"Sarah's favorite language is {language}.")
```

要查看 Sarah 选择了哪种语言，我们请求该值：

```py
favorite_languages['sarah']
```

我们使用这种语法从字典❶中提取 Sarah 最喜欢的编程语言，并将其赋值给变量 `language`。创建一个新的变量使得 `print()` 调用更加简洁。输出显示了 Sarah 最喜欢的编程语言：

```py
Sarah's favorite language is C.
```

你可以使用相同的语法与字典中的任何单个对象。

### 使用 get() 访问值

使用方括号中的键从字典中检索你感兴趣的值可能会导致一个潜在的问题：如果你请求的键不存在，你将得到一个错误。

让我们看看当你请求一个没有设置点数值的外星人的点数值时会发生什么：

**alien_no_points.py**

```py
alien_0 = {'color': 'green', 'speed': 'slow'}
print(alien_0['points'])
```

这会导致一个追溯错误，显示一个 `KeyError`：

```py
Traceback (most recent call last):
  File "alien_no_points.py", line 2, in <module>
    print(alien_0['points'])
          ~~~~~~~^^^^^^^^^^
KeyError: 'points'
```

你将在第10章中了解如何处理类似的错误。针对字典，你可以使用`get()`方法来设置一个默认值，当请求的键不存在时，返回该默认值。

`get()`方法需要一个键作为第一个参数。作为第二个可选参数，你可以传递一个当键不存在时返回的默认值：

```py
alien_0 = {'color': 'green', 'speed': 'slow'}

point_value = alien_0.get('points', 'No point value assigned.')
print(point_value)
```

如果字典中存在键`'points'`，你将得到对应的值。如果不存在，则会返回默认值。在这种情况下，`points`不存在，我们得到的是一个干净的消息，而不是错误：

```py
No point value assigned.
```

如果你请求的键可能不存在，考虑使用`get()`方法，而不是方括号表示法。

## 遍历字典

一个Python字典可以包含少量的键值对，也可以包含数百万个键值对。由于字典可以存储大量数据，Python允许你遍历字典。字典可以用多种方式存储信息，因此也有多种方式可以遍历它们。你可以遍历字典的所有键值对、遍历它的键，或者遍历它的值。

### 遍历所有键值对

在我们探索不同的遍历方法之前，让我们考虑一个新的字典，用来存储网站上某个用户的信息。以下字典将存储一个人的用户名、名字和姓氏：

**user.py**

```py
user_0 = {
    'username': 'efermi',
    'first': 'enrico',
    'last': 'fermi',
    }
```

基于你在本章中学到的知识，你可以访问`user_0`的任何信息。但是，如果你想查看该用户字典中存储的所有内容怎么办？你可以使用`for`循环遍历字典：

```py
user_0 = {
 'username': 'efermi',
 'first': 'enrico',
 'last': 'fermi',
 }

for key, value in user_0.items():
    print(f"\nKey: {key}")
    print(f"Value: {value}")
```

要为字典编写`for`循环，你需要为每一对键值对创建两个变量名，用于存储键和值。你可以为这两个变量选择任何你喜欢的名字。如果你使用缩写作为变量名，代码依然能够正常工作，例如：

```py
for k, v in user_0.items()
```

`for`语句的第二部分包括字典的名称，后跟`items()`方法，该方法返回键值对的序列。然后，`for`循环将这些对分配给提供的两个变量。在前面的示例中，我们使用这些变量打印每个`key`，后跟相应的`value`。第一个`print()`调用中的`"\n"`确保在输出的每个键值对之前插入一个空行：

```py
Key: username
Value: efermi

Key: first
Value: enrico

Key: last
Value: fermi
```

遍历所有键值对在字典中尤其有效，比如第96页的*favorite_languages.py*示例，它存储了许多不同键的相同类型的信息。如果你遍历`favorite_languages`字典，你将得到字典中每个人的名字和他们最喜欢的编程语言。因为键总是指向一个人的名字，值总是指向一种语言，所以我们在循环中会使用变量`name`和`language`，而不是`key`和`value`。这样可以让你更容易理解循环内发生的事情：

**favorite_languages.py**

```py
favorite_languages = {
 'jen': 'python',
 'sarah': 'c',
 'edward': 'rust',
 'phil': 'python',
 }

for name, language in favorite_languages.items():
    print(f"{name.title()}'s favorite language is {language.title()}.")
```

这段代码告诉Python遍历字典中的每个键值对。它在处理每个键值对时，将键分配给变量`name`，将值分配给变量`language`。这些描述性的变量名使得你更容易理解`print()`语句在做什么。

现在，只需几行代码，我们就可以显示民意调查的所有信息：

```py
Jen's favorite language is Python.
Sarah's favorite language is C.
Edward's favorite language is Rust.
Phil's favorite language is Python.
```

如果我们的字典存储了对一千人甚至一百万人的调查结果，这种类型的循环同样有效。

### 遍历字典中的所有键

当你不需要处理字典中的所有值时，`keys()`方法非常有用。让我们遍历`favorite_languages`字典，并打印出所有参与调查的人的名字：

```py
favorite_languages = {
 'jen': 'python',
 'sarah': 'c',
 'edward': 'rust',
 'phil': 'python',
 }

for name in favorite_languages.keys():
    print(name.title())
```

这个`for`循环告诉Python从字典`favorite_languages`中提取所有的键，并一次性将它们分配给变量`name`。输出结果显示了所有参与调查的人的名字：

```py
Jen
Sarah
Edward
Phil
```

遍历键实际上是默认的行为，因此，如果你写成：

```py
for name in favorite_languages:
```

而不是：

```py
for name in favorite_languages.keys():
```

如果你觉得这样能让代码更易读，可以显式使用`keys()`方法，或者如果你愿意，也可以省略它。

你可以在循环内通过当前的键来访问任何你关心的键值对的值。让我们打印一些朋友选择的编程语言的消息。我们将像之前那样遍历字典中的名字，但当名字与我们的朋友之一匹配时，我们将显示关于他们最喜欢的语言的消息：

```py
favorite_languages = {
    *--snip--*
 }

friends = ['phil', 'sarah']
for name in favorite_languages.keys():
    print(f"Hi {name.title()}.")

❶     if name in friends:
❷         language = favorite_languages[name].title()
        print(f"\t{name.title()}, I see you love {language}!")
```

首先，我们列出一些朋友的名字，准备给他们打印一条消息。在循环中，我们打印每个人的名字。然后，我们检查当前正在处理的`name`是否在`friends`列表中❶。如果在，我们使用字典的名字和当前`name`的值作为键来确定这个人最喜欢的语言❷。接着，我们打印一条特别的问候语，包含他们选择的编程语言。

每个人的名字都会被打印出来，但我们的朋友们会收到一条特别的消息：

```py
Hi Jen.
Hi Sarah.
    Sarah, I see you love C!
Hi Edward.
Hi Phil.
    Phil, I see you love Python!
```

你还可以使用`keys()`方法来检查某个人是否参与了调查。这次，让我们来看看Erin是否参与了调查：

```py
favorite_languages = {
 *--snip--*
 }

if 'erin' not in favorite_languages.keys():
    print("Erin, please take our poll!")
```

`keys()` 方法不仅用于循环：它实际上返回所有键的序列，而 `if` 语句仅检查 `'erin'` 是否在这个序列中。因为她不在，所以打印出一条信息邀请她参与调查：

```py
Erin, please take our poll!
```

### 按特定顺序遍历字典的键

遍历字典时，返回的项与它们插入字典的顺序相同。不过，有时你可能希望以不同的顺序遍历字典。

一种方法是在 `for` 循环中按返回的顺序对键进行排序。你可以使用 `sorted()` 函数来按顺序获取键的副本：

```py
favorite_languages = {
 'jen': 'python',
 'sarah': 'c',
 'edward': 'rust',
 'phil': 'python',
 }

for name in sorted(favorite_languages.keys()):
    print(f"{name.title()}, thank you for taking the poll.")
```

这个 `for` 语句与其他 `for` 语句类似，唯一的区别是我们在 `dictionary.keys()` 方法周围包裹了 `sorted()` 函数。这告诉 Python 获取字典中的所有键，并在开始循环之前将其排序。输出结果显示所有参与调查的人，名字按顺序排列：

```py
Edward, thank you for taking the poll.
Jen, thank you for taking the poll.
Phil, thank you for taking the poll.
Sarah, thank you for taking the poll.
```

### 遍历字典中的所有值

如果你主要对字典中包含的值感兴趣，可以使用 `values()` 方法返回一个值的序列，而不包含任何键。例如，假设我们只想要一个所有语言的列表，而不包含选择每种语言的人的名字：

```py
favorite_languages = {
 'jen': 'python',
 'sarah': 'c',
 'edward': 'rust',
 'phil': 'python',
 }

print("The following languages have been mentioned:")
for language in favorite_languages.values():
    print(language.title())
```

这里的 `for` 语句从字典中提取每个值，并将其分配给变量 `language`。当这些值被打印时，我们得到一个所有选择语言的列表：

```py
The following languages have been mentioned:
Python
C
Rust
Python
```

这种方法从字典中提取所有值，而不检查是否有重复项。对于较少的值数量，这可能没问题，但如果是调查中有大量回应者时，结果会是一个非常重复的列表。为了查看每种语言而不重复，我们可以使用集合。*集合* 是一个其中每个项必须唯一的集合：

```py
favorite_languages = {
 *--snip--*
 }

print("The following languages have been mentioned:")
for language in set(favorite_languages.values()):
 print(language.title())
```

当你将 `set()` 包裹在包含重复项的值集合周围时，Python 会识别集合中唯一的项，并根据这些项构建一个集合。这里我们使用 `set()` 从 `favorite_languages.values()` 中提取唯一语言。

结果是一个没有重复项的语言列表，这些语言是参与调查的人提到的：

```py
The following languages have been mentioned:
Python
C
Rust
```

随着你继续学习 Python，你会经常发现语言内置的某个特性可以帮助你精确地处理数据。

## 嵌套

有时候你可能需要将多个字典存储在一个列表中，或者将一个列表作为字典的值。这被称为 *嵌套*。你可以在列表中嵌套字典，在字典中嵌套项的列表，甚至将字典嵌套在另一个字典中。嵌套是一个强大的特性，接下来的示例将演示这一点。

### 字典的列表

`alien_0`字典包含有关一个外星人的各种信息，但它没有空间存储第二个外星人的信息，更不用说一整屏外星人了。你如何管理一支外星人舰队呢？一种方法是创建一个外星人列表，其中每个外星人都是一个包含该外星人信息的字典。例如，以下代码构建了一个包含三个外星人的列表：

**aliens.py**

```py
alien_0 = {'color': 'green', 'points': 5}
alien_1 = {'color': 'yellow', 'points': 10}
alien_2 = {'color': 'red', 'points': 15}

❶ aliens = [alien_0, alien_1, alien_2]

for alien in aliens:
    print(alien)
```

我们首先创建三个字典，每个字典代表一个不同的外星人。我们将这些字典存储在一个名为`aliens`的列表中❶。最后，我们遍历列表并打印出每个外星人：

```py
{'color': 'green', 'points': 5}
{'color': 'yellow', 'points': 10}
{'color': 'red', 'points': 15}
```

一个更现实的例子可能包含超过三个外星人，并且代码会自动生成每个外星人。在下面的示例中，我们使用`range()`来创建一个30个外星人的舰队：

```py
# Make an empty list for storing aliens.
aliens = []

# Make 30 green aliens.
❶ for alien_number in range(30):
❷     new_alien = {'color': 'green', 'points': 5, 'speed': 'slow'}
❸     aliens.append(new_alien)

# Show the first 5 aliens.
❹ for alien in aliens[:5]:
    print(alien)
print("...")

# Show how many aliens have been created.
print(f"Total number of aliens: {len(aliens)}")
```

这个示例从一个空的列表开始，用来保存将要创建的所有外星人。`range()`函数❶返回一系列数字，它只是告诉Python我们希望循环重复多少次。每次循环运行时，我们创建一个新的外星人❷，然后将每个新的外星人追加到列表`aliens`中❸。我们使用切片打印前五个外星人❹，最后，我们打印列表的长度，以证明我们已经生成了完整的30个外星人的舰队：

```py
{'color': 'green', 'points': 5, 'speed': 'slow'}
{'color': 'green', 'points': 5, 'speed': 'slow'}
{'color': 'green', 'points': 5, 'speed': 'slow'}
{'color': 'green', 'points': 5, 'speed': 'slow'}
{'color': 'green', 'points': 5, 'speed': 'slow'}
...

Total number of aliens: 30
```

这些外星人都有相同的特征，但Python将每个外星人视为一个独立的对象，这使得我们可以单独修改每个外星人。

你如何处理这样一组外星人呢？假设游戏的某个环节中，外星人随着游戏的进展改变颜色并且移动得更快。当是时候改变颜色时，我们可以使用`for`循环和`if`语句来改变外星人的颜色。例如，为了将前三个外星人改为黄色的、中等速度的外星人，每个外星人值10分，我们可以这样做：

```py
# Make an empty list for storing aliens.
aliens = []

# Make 30 green aliens.
for alien_number in range (30):
 new_alien = {'color': 'green', 'points': 5, 'speed': 'slow'}
 aliens.append(new_alien)

for alien in aliens[:3]:
    if alien['color'] == 'green':
        alien['color'] = 'yellow'
        alien['speed'] = 'medium'
        alien['points'] = 10

# Show the first 5 aliens.
for alien in aliens[:5]:
 print(alien)
print("...")
```

因为我们想要修改前三个外星人，所以我们遍历一个只包含前三个外星人的切片。现在所有外星人都是绿色的，但并不总是这样，因此我们写一个`if`语句来确保只修改绿色外星人。如果外星人是绿色的，我们将其颜色改为`'yellow'`，速度改为`'medium'`，分值改为`10`，如下所示的输出：

```py
{'color': 'yellow', 'points': 10, 'speed': 'medium'}
{'color': 'yellow', 'points': 10, 'speed': 'medium'}
{'color': 'yellow', 'points': 10, 'speed': 'medium'}
{'color': 'green', 'points': 5, 'speed': 'slow'}
{'color': 'green', 'points': 5, 'speed': 'slow'}
...
```

你可以通过添加一个`elif`块来扩展这个循环，将黄色的外星人变成红色的、快速移动的外星人，每个外星人的分数为15分。省略完整程序的情况下，这个循环看起来像这样：

```py
for alien in aliens[0:3]:
 if alien['color'] == 'green':
 alien['color'] = 'yellow'
 alien['speed'] = 'medium'
 alien['points'] = 10
    elif alien['color'] == 'yellow':
        alien['color'] = 'red'
        alien['speed'] = 'fast'
        alien['points'] = 15
```

当每个字典包含关于一个对象的多种信息时，将多个字典存储在列表中是很常见的做法。例如，你可以为网站上的每个用户创建一个字典，就像我们在第99页的*user.py*中所做的那样，并将这些字典存储在一个名为`users`的列表中。列表中的所有字典应该具有相同的结构，这样你就可以遍历列表，并以相同的方式处理每个字典对象。

### 字典中的列表

有时，将列表放入字典中比将字典放入列表中更为有用。例如，考虑一下如何描述一个人正在点的披萨。如果你只使用列表，实际上你只能存储披萨的配料列表。使用字典时，配料列表可以仅仅是你描述的披萨的一个方面。

在以下示例中，每个披萨都存储了两种信息：一种是饼皮类型，另一种是配料列表。配料列表是与键 `'toppings'` 关联的值。为了使用列表中的项目，我们只需给出字典的名称和键 `'toppings'`，就像访问字典中的任何值一样。返回的不是单个值，而是一个配料列表：

**pizza.py**

```py
# Store information about a pizza being ordered.
pizza = {
    'crust': 'thick',
    'toppings': ['mushrooms', 'extra cheese'],
    }

# Summarize the order.
❶ print(f"You ordered a {pizza['crust']}-crust pizza "
    "with the following toppings:")

❷ for topping in pizza['toppings']:
    print(f"\t{topping}")
```

我们从一个字典开始，字典中包含已点披萨的信息。字典中的一个键是 `'crust'`，其对应的值是字符串 `'thick'`。下一个键 `'toppings'` 的值是一个存储所有请求配料的列表。在制作披萨之前，我们先总结一下订单❶。当你需要在 `print()` 调用中拆分一行时，选择一个适当的位置拆分打印的行，并以引号结束该行。接下来缩进新的一行，添加开引号，并继续字符串。Python 会自动将括号内的所有字符串组合在一起。为了打印配料，我们写一个 `for` 循环❷。要访问配料列表，我们使用键 `'toppings'`，然后 Python 会从字典中获取配料列表。

以下输出总结了我们计划制作的披萨：

```py
You ordered a thick-crust pizza with the following toppings:
    mushrooms
    extra cheese
```

你可以随时将列表嵌套在字典中，只要你希望将多个值与字典中的单个键相关联。在之前的最喜欢编程语言的示例中，如果我们将每个人的回答存储在一个列表中，那么人们可以选择多个最喜欢的语言。当我们遍历字典时，与每个人相关联的值将是一个语言列表，而不是单一的语言。在字典的 `for` 循环内部，我们使用另一个 `for` 循环来遍历与每个人相关联的语言列表：

**favorite_languages.py**

```py
favorite_languages = {
 'jen': ['python', 'rust'],
 'sarah': ['c'],
    'edward': ['rust', 'go'],
    'phil': ['python', 'haskell'],
    }

❶ for name, languages in favorite_languages.items():
    print(f"\n{name.title()}'s favorite languages are:")
❷     for language in languages:
        print(f"\t{language.title()}")
```

现在，`favorite_languages` 中每个名字所对应的值都是一个列表。注意，有些人只有一个最喜欢的语言，而有些人有多个最喜欢的语言。当我们遍历字典❶时，我们使用变量名 `languages` 来存储字典中的每个值，因为我们知道每个值将是一个列表。在主字典循环内部，我们再用一个 `for` 循环❷来遍历每个人的最喜欢语言列表。现在，每个人都可以列出他们喜欢的多个编程语言：

```py
Jen's favorite languages are:
    Python
    Rust

Sarah's favorite languages are:
    C

Edward's favorite languages are:
    Rust
    Go

Phil's favorite languages are:
    Python
    Haskell
```

为了进一步优化这个程序，你可以在字典的`for`循环开始时加入一个`if`语句，检查每个人是否有多个喜欢的编程语言，方法是检查`len(languages)`的值。如果某人有多个喜欢的语言，输出保持不变。如果某人只有一个喜欢的语言，你可以修改措辞以反映这一点。例如，你可以说：“Sarah最喜欢的编程语言是C。”

### 字典中的字典

你可以将一个字典嵌套在另一个字典中，但这样做时你的代码可能会迅速变得复杂。例如，如果你为一个网站拥有多个用户，每个用户都有一个唯一的用户名，你可以使用用户名作为字典中的键。然后，你可以使用字典作为与用户名相关联的值来存储每个用户的信息。在下面的代码中，我们存储了每个用户的三项信息：名字、姓氏和位置。我们将通过遍历用户名和与每个用户名相关联的字典来访问这些信息：

**many_users.py**

```py
users = {
    'aeinstein': {
        'first': 'albert',
        'last': 'einstein',
        'location': 'princeton',
        },

    'mcurie': {
        'first': 'marie',
        'last': 'curie',
        'location': 'paris',
        },

    }

❶ for username, user_info in users.items():
❷     print(f"\nUsername: {username}")
❸     full_name = f"{user_info['first']} {user_info['last']}"
    location = user_info['location']

❹     print(f"\tFull name: {full_name.title()}")
    print(f"\tLocation: {location.title()}")
```

我们首先定义一个名为`users`的字典，它包含两个键：分别对应用户名`'aeinstein'`和`'mcurie'`。与每个键关联的值是一个字典，包含每个用户的名字、姓氏和位置。接着，我们遍历`users`字典 ❶。Python将每个键赋值给变量`username`，与每个用户名相关联的字典则赋值给变量`user_info`。进入主字典循环后，我们打印出用户名 ❷。

接下来，我们开始访问内层字典 ❸。包含用户信息的字典`user_info`有三个键：`'first'`、`'last'`和`'location'`。我们使用每个键来生成每个人整齐格式化的全名和位置，然后打印出我们对每个用户的总结 ❹：

```py
Username: aeinstein
    Full name: Albert Einstein
    Location: Princeton

Username: mcurie
    Full name: Marie Curie
    Location: Paris
```

请注意，每个用户字典的结构是相同的。虽然Python没有强制要求这一点，但这种结构使得嵌套字典更容易操作。如果每个用户字典的键不同，`for`循环中的代码将更加复杂。

## 总结

在本章中，你学习了如何定义字典以及如何操作字典中存储的信息。你学习了如何访问和修改字典中的单个元素，以及如何遍历字典中的所有信息。你学会了如何遍历字典的键值对、键和值。你还学会了如何将多个字典嵌套在列表中，将列表嵌套在字典中，和将字典嵌套在字典中。

在下一章，你将学习`while`循环以及如何接受使用你程序的人的输入。这将是一个令人兴奋的章节，因为你将学习如何让你的程序变得互动：它们将能够响应用户输入。

# 7

用户输入与while循环

![](Images/chapterart.png)

大多数程序是为了解决最终用户的问题而编写的。为此，你通常需要从用户那里获取一些信息。例如，假设有人想知道自己是否足够老，可以投票。如果你编写一个程序来回答这个问题，你需要知道用户的年龄，才能给出答案。程序将要求用户输入或*提供*他们的年龄；一旦程序获得了这个输入，它就可以将其与投票年龄进行比较，以判断用户是否足够老，然后报告结果。

在这一章中，你将学习如何接受用户输入，以便你的程序可以使用这些输入。当程序需要一个名字时，你可以提示用户输入名字。当程序需要一系列名字时，你也可以提示用户输入多个名字。为此，你将使用`input()`函数。

你还将学习如何让程序持续运行，直到用户希望它停止，这样他们可以输入所需的所有信息；然后，你的程序可以利用这些信息进行处理。你将使用Python的`while`循环来让程序在特定条件成立时持续运行。

通过能够处理用户输入以及控制程序运行时间的能力，你将能够编写完全互动的程序。

## `input()`函数的工作原理

`input()`函数会暂停程序并等待用户输入文本。一旦Python接收到用户的输入，它会将输入赋值给一个变量，以便你方便地使用它。

例如，下面的程序会要求用户输入一些文本，然后将该消息显示给用户：

**parrot.py**

```py
message = input("Tell me something, and I will repeat it back to you: ")
print(message)
```

`input()`函数接受一个参数：*提示信息*，用于告诉用户需要输入什么类型的信息。在这个例子中，当Python运行第一行时，用户会看到提示信息`Tell me something, and I will repeat it back to you:`。程序在等待用户输入响应，用户按下回车键后，程序继续执行。响应被赋值给变量`message`，然后`print(message)`会将输入内容显示给用户：

```py
Tell me something, and I will repeat it back to you: **Hello everyone!**
Hello everyone!
```

### 编写清晰的提示信息

每次使用`input()`函数时，你都应该包含一个清晰、易于理解的提示，告诉用户你需要什么类型的信息。任何能告诉用户需要输入什么内容的语句都可以。例如：

**greeter.py**

```py
name = input("Please enter your name: ")
print(f"\nHello, {name}!")
```

在提示信息末尾添加一个空格（在前面的示例中冒号后面）来将提示与用户的响应分开，并且清晰地告诉用户应该在何处输入他们的文本。例如：

```py
Please enter your name: **Eric**
Hello, Eric!
```

有时候你可能需要编写一个超过一行的提示信息。例如，你可能想告诉用户你为何要求提供某些输入。你可以将提示信息赋值给一个变量，并将该变量传递给`input()`函数。这使得你能够在多行中构建提示信息，然后编写一个简洁的`input()`语句。

**greeter.py**

```py
prompt = "If you share your name, we can personalize the messages you see."
prompt += "\nWhat is your first name? "

name = input(prompt)
print(f"\nHello, {name}!")
```

这个例子展示了构建多行字符串的一种方式。第一行将消息的第一部分赋值给变量 `prompt`。在第二行，运算符 `+=` 将赋值给 `prompt` 的字符串与新的字符串连接起来。

现在提示符跨越了两行，再次在问号后留有空格以增强清晰度：

```py
If you share your name, we can personalize the messages you see.
What is your first name? **Eric**

Hello, Eric!
```

### 使用 `int()` 接受数值输入

当你使用 `input()` 函数时，Python 将用户输入的所有内容都视为字符串。考虑以下的解释器会话，它询问用户的年龄：

```py
>>> **age = input("How old are you? ")**
How old are you? **21**
>>> **age**
'21'
```

用户输入数字 `21`，但是当我们请求 Python 获取 `age` 的值时，它返回了 `'21'`，即输入的数值的字符串表示。我们知道 Python 将输入解释为字符串，因为该数字现在被引号括起来。如果你只是想打印输入内容，这样没问题。但如果你尝试将输入作为数字使用，你将遇到错误：

```py
>>> **age = input("How old are you? ")**
How old are you? **21**
❶ >>> **age >= 18**
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
❷ TypeError: '>=' not supported between instances of 'str' and 'int'
```

当你尝试使用输入进行数值比较 ❶ 时，Python 会产生错误，因为它不能将字符串与整数进行比较：分配给 `age` 的字符串 `'21'` 不能与数值 `18` ❷ 进行比较。

我们可以通过使用 `int()` 函数来解决这个问题，它将输入的字符串转换为数值。这使得比较能够成功执行：

```py
>>> **age = input("How old are you? ")**
How old are you? **21**
❶ >>> **age = int(age)**
>>> **age >= 18**
True
```

在这个例子中，当我们在提示符下输入 `21` 时，Python 将该数字解释为字符串，但该值随后通过 `int()` ❶ 被转换为数值表示。现在，Python 可以执行条件测试：它将 `age`（现在表示数值 21）与 `18` 进行比较，判断 `age` 是否大于或等于 18。这个测试的结果是 `True`。

如何在实际程序中使用 `int()` 函数？考虑一个程序，判断人们是否足够高，能够坐过山车：

**rollercoaster.py**

```py
height = input("How tall are you, in inches? ")
height = int(height)

if height >= 48:
    print("\nYou're tall enough to ride!")
else:
    print("\nYou'll be able to ride when you're a little older.")
```

该程序可以将 `height` 与 `48` 进行比较，因为 `height = int(height)` 在进行比较之前将输入值转换为数值表示。如果输入的数字大于或等于 48，我们会告诉用户他们的身高足够高：

```py
How tall are you, in inches? **71**

You're tall enough to ride!
```

当你使用数值输入进行计算和比较时，请务必先将输入值转换为数值表示。

### 取余运算符

处理数值信息时，一个有用的工具是 *取余运算符*（`%`），它将一个数字除以另一个数字并返回余数：

```py
>>> **4 % 3**
1
>>> **5 % 3**
2
>>> **6 % 3**
0
>>> **7 % 3**
1
```

取余运算符不会告诉你一个数字能被另一个数字整除多少次；它只告诉你余数是多少。

当一个数字能被另一个数字整除时，余数为 0，因此取余运算符总是返回 0。你可以利用这一点来判断一个数字是偶数还是奇数：

**even_or_odd.py**

```py
number = input("Enter a number, and I'll tell you if it's even or odd: ")
number = int(number)

if number % 2 == 0:
    print(f"\nThe number {number} is even.")
else:
    print(f"\nThe number {number} is odd.")
```

偶数总是可以被二整除，因此如果一个数字与 2 的取余为零（在这里，`if number % 2 == 0`），则该数字是偶数。否则，它是奇数。

```py
Enter a number, and I'll tell you if it's even or odd: **42**

The number 42 is even.
```

## 引入 while 循环

`for`循环会遍历一个集合中的每个项，并为集合中的每个项执行一次代码块。相比之下，`while`循环则是只要某个条件为真，就继续运行。

### `while`循环的实际运行

你可以使用`while`循环来通过一系列数字进行计数。例如，以下`while`循环从1计数到5：

**counting.py**

```py
current_number = 1
while current_number <= 5:
 print(current_number)
    current_number += 1
```

在第一行中，我们通过将`current_number`赋值为1来开始计数。然后，`while`循环被设置为在`current_number`的值小于或等于5时持续运行。循环中的代码打印`current_number`的值，然后使用`current_number += 1`将该值加1。（`+=`运算符是`current_number = current_number + 1`的简写。）

只要`current_number <= 5`的条件为真，Python就会重复执行循环。因为1小于5，所以Python打印`1`，然后加1，使当前数字变为`2`。因为2小于5，所以Python打印`2`，再次加1，使当前数字变为`3`，依此类推。当`current_number`的值大于5时，循环停止，程序结束：

```py
1
2
3
4
5
```

你每天使用的程序很可能包含`while`循环。例如，一个游戏需要使用`while`循环来保持运行，直到你想停止为止，这样它就能在你要求退出时停止运行。如果程序在我们要求退出之前就停止运行，或者在我们想退出后仍然运行，那就不好玩了，所以`while`循环非常有用。

### 让用户选择何时退出

我们可以通过将大部分程序放入`while`循环中，让*parrot.py*程序运行直到用户想退出为止。我们将定义一个*退出值*，然后只要用户没有输入退出值，就继续运行程序：

**parrot.py**

```py
prompt = "\nTell me something, and I will repeat it back to you:"
prompt += "\nEnter 'quit' to end the program. "

message = ""
while message != 'quit':
    message = input(prompt)
    print(message)
```

我们首先定义一个提示，告诉用户他们的两个选项：输入一条信息或输入退出值（在此案例中为`'quit'`）。然后，我们设置一个变量`message`来跟踪用户输入的值。我们将`message`定义为空字符串`""`，这样Python在第一次进入`while`语句时会有东西可以检查。当程序第一次运行并且Python到达`while`语句时，它需要将`message`的值与`'quit'`进行比较，但此时用户尚未输入任何内容。如果Python没有东西可以比较，它将无法继续执行程序。为了解决这个问题，我们确保给`message`一个初始值。虽然它只是一个空字符串，但它对Python是有意义的，并且可以让Python进行比较，从而使`while`循环得以正常工作。这个`while`循环会在`message`的值不是`'quit'`时继续运行。

在第一次循环时，`message`只是一个空字符串，因此Python进入循环。在`message = input(prompt)`时，Python显示提示并等待用户输入。当用户输入的内容被赋值给`message`并打印后，Python会重新评估`while`语句中的条件。只要用户没有输入`'quit'`，提示就会再次显示，Python会等待更多输入。当用户最终输入`'quit'`时，Python会停止执行`while`循环，程序结束：

```py
Tell me something, and I will repeat it back to you:
Enter 'quit' to end the program. **Hello everyone!**
Hello everyone!

Tell me something, and I will repeat it back to you:
Enter 'quit' to end the program. **Hello again.**
Hello again.

Tell me something, and I will repeat it back to you:
Enter 'quit' to end the program. **quit**
quit
```

这个程序运行良好，唯一的问题是它像打印一个实际的消息一样打印了单词`'quit'`。一个简单的`if`测试就能解决这个问题：

```py
prompt = "\nTell me something, and I will repeat it back to you:"
prompt += "\nEnter 'quit' to end the program. "

message = ""
while message != 'quit':
 message = input(prompt)

    if message != 'quit':
        print(message)
```

现在程序在显示消息之前进行快速检查，只有在消息与退出值不匹配时才会打印该消息：

```py
Tell me something, and I will repeat it back to you:
Enter 'quit' to end the program. **Hello everyone!**
Hello everyone!

Tell me something, and I will repeat it back to you:
Enter 'quit' to end the program. **Hello again.**
Hello again.

Tell me something, and I will repeat it back to you:
Enter 'quit' to end the program. **quit**
```

### 使用标志

在前面的例子中，我们让程序在给定条件为真时执行某些任务。但如果是更复杂的程序，其中许多不同的事件可能导致程序停止运行呢？

例如，在游戏中，几个不同的事件可能会结束游戏。当玩家的舰船耗尽、时间用完，或者他们应该保护的城市全部被摧毁时，游戏应该结束。只要发生了其中任何一个事件，游戏就应结束。如果有很多可能的事件可能导致程序停止，尝试在一个`while`语句中测试所有这些条件会变得复杂且困难。

对于一个应在多个条件都为真时才运行的程序，你可以定义一个变量来决定程序是否继续运行。这个变量被称为*标志*，它充当程序的信号。我们可以编写程序，使其在标志被设置为`True`时运行，并在多个事件之一将标志的值设置为`False`时停止运行。结果，我们的`while`语句只需要检查一个条件：标志是否当前为`True`。然后，所有其他的测试（检查是否发生了某个事件应该将标志设置为`False`）可以在程序的其余部分整齐地组织。

让我们在前一节的*parrot.py*程序中添加一个标志。这个标志，我们称之为`active`（当然你也可以用其他名称），将监控程序是否应该继续运行：

```py
prompt = "\nTell me something, and I will repeat it back to you:"
prompt += "\nEnter 'quit' to end the program. "

active = True
❶ while active:
    message = input(prompt)

    if message == 'quit':
        active = False
    else:
        print(message)
```

我们将变量`active`设置为`True`，以便程序在活动状态下启动。这样，`while`语句就变得更简洁，因为在`while`语句本身没有进行任何比较；逻辑被转移到程序的其他部分。只要`active`变量保持`True`，循环就会继续运行❶。

在`while`循环中的`if`语句里，我们在用户输入后检查`message`的值。如果用户输入`'quit'`，我们将`active`设置为`False`，然后`while`循环停止。如果用户输入除`'quit'`以外的任何内容，我们将他们的输入打印为消息。

这个程序与之前的示例有相同的输出，后者是将条件测试直接放在`while`语句中。但现在，由于我们有一个标志来指示整体程序是否处于活动状态，因此可以很容易地添加更多的测试（例如`elif`语句），用于处理应使`active`变为`False`的事件。这在复杂的程序中很有用，比如游戏，在这些程序中可能有许多事件应该导致程序停止运行。当这些事件中的任何一个导致活动标志变为`False`时，主游戏循环将退出，可以显示*游戏结束*消息，并且玩家可以选择重新开始游戏。

### 使用break退出循环

要立即退出`while`循环而不运行循环中剩余的任何代码（无论条件测试的结果如何），请使用`break`语句。`break`语句控制程序的流程；你可以使用它来控制哪些代码行被执行，哪些不被执行，从而让程序仅在你希望的时刻执行你想要的代码。

例如，考虑一个询问用户曾经去过哪些地方的程序。我们可以通过在用户输入`'quit'`值时调用`break`来停止此程序中的`while`循环：

**cities.py**

```py
prompt = "\nPlease enter the name of a city you have visited:"
prompt += "\n(Enter 'quit' when you are finished.) "

❶ while True:
    city = input(prompt)

    if city == 'quit':
        break
    else:
        print(f"I'd love to go to {city.title()}!")
```

一个以`while True` ❶开头的循环将永远运行，除非遇到`break`语句。这个程序中的循环会不断询问用户输入他们去过的城市名，直到他们输入`'quit'`。当他们输入`'quit'`时，`break`语句会执行，导致Python退出循环：

```py
Please enter the name of a city you have visited:
(Enter 'quit' when you are finished.) **New York**
I'd love to go to New York!

Please enter the name of a city you have visited:
(Enter 'quit' when you are finished.) **San Francisco**
I'd love to go to San Francisco!

Please enter the name of a city you have visited:
(Enter 'quit' when you are finished.) **quit**
```

### 在循环中使用continue

与其完全跳出循环而不执行其余的代码，不如使用`continue`语句根据条件测试的结果返回到循环的开头。例如，考虑一个从1到10计数但仅打印该范围内奇数的循环：

**counting.py**

```py
current_number = 0
while current_number < 10:
❶     current_number += 1
    if current_number % 2 == 0:
        continue

    print(current_number)
```

首先，我们将`current_number`设置为0。因为它小于10，Python进入了`while`循环。进入循环后，我们将计数器增加1 ❶，因此`current_number`变为1。接着，`if`语句检查`current_number`与2的模。如果模为0（这意味着`current_number`可以被2整除），`continue`语句告诉Python忽略循环的其余部分并返回到开头。如果当前数字不能被2整除，循环的其余部分将继续执行，Python打印当前的数字：

```py
1
3
5
7
9
```

### 避免无限循环

每个`while`循环都需要一种停止运行的方法，以免它继续无限运行。例如，这个计数循环应该从1计数到5：

**counting.py**

```py
x = 1
while x <= 5:
    print(x)
    x += 1
```

然而，如果不小心省略了`x += 1`这一行，循环将永远运行：

```py
# This loop runs forever!
x = 1
while x <= 5:
    print(x)
```

现在，`x`的值将从`1`开始，但永远不会改变。因此，条件测试`x <= 5`将始终返回`True`，`while`循环将永远运行，打印一系列的1，如下所示：

```py
1
1
1
1
`--snip--`
```

每个程序员偶尔都会不小心写出无限`while`循环，尤其是当程序的循环有微妙的退出条件时。如果程序陷入了无限循环，可以按CTRL-C或者直接关闭显示程序输出的终端窗口。

为了避免写出无限循环，测试每个`while`循环，确保循环在你预期的时刻停止。如果你希望程序在用户输入某个特定值时结束，运行程序并输入该值。如果程序没有结束，仔细检查程序是如何处理应导致循环退出的值的。确保程序中至少有一部分可以使循环的条件变为`False`，或使其到达`break`语句。

## 使用`while`循环与列表和字典

到目前为止，我们只处理一次一个用户的信息。我们接收用户的输入，然后打印该输入或对此的响应。在下次执行`while`循环时，我们会接收另一个输入值并对此做出回应。但为了跟踪多个用户和信息，我们需要在`while`循环中使用列表和字典。

`for`循环适合遍历列表，但你不应该在`for`循环中修改列表，因为 Python 在跟踪列表中的项时会遇到困难。要在遍历列表时修改它，应该使用`while`循环。使用`while`循环与列表和字典结合，可以收集、存储和组织大量输入，以供后续检查和报告。

### 将项目从一个列表移动到另一个列表

假设有一个新注册但未验证的网站用户列表。在验证这些用户后，我们如何将他们移动到一个单独的已确认用户列表中呢？一种方法是使用`while`循环从未确认用户列表中提取用户进行验证，然后将他们添加到已确认用户的单独列表中。以下是该代码的示例：

**confirmed_users.py**

```py
# Start with users that need to be verified,
#  and an empty list to hold confirmed users.
❶ unconfirmed_users = ['alice', 'brian', 'candace']
confirmed_users = []

# Verify each user until there are no more unconfirmed users.
#  Move each verified user into the list of confirmed users.
❷ while unconfirmed_users:
❸     current_user = unconfirmed_users.pop()

    print(f"Verifying user: {current_user.title()}")
❹     confirmed_users.append(current_user)

# Display all confirmed users.
print("\nThe following users have been confirmed:")
for confirmed_user in confirmed_users:
    print(confirmed_user.title())
```

我们从一个包含未确认用户的列表❶（Alice、Brian 和 Candace）开始，并有一个空列表来存放已确认的用户。只要`unconfirmed_users`列表不为空❷，`while`循环就会继续执行。在这个循环中，`pop()`方法会一次从`unconfirmed_users`列表的末尾移除一个未验证的用户❸。因为Candace是`unconfirmed_users`列表中的最后一个，所以她的名字会首先被移除，赋值给`current_user`，然后添加到`confirmed_users`列表中❹。接下来是Brian，然后是Alice。

我们通过打印确认消息来模拟确认每个用户，然后将他们添加到已确认用户的列表中。当未确认用户列表缩小时，已确认用户列表会增加。当未确认用户列表为空时，循环停止，并打印已确认用户列表：

```py
Verifying user: Candace
Verifying user: Brian
Verifying user: Alice

The following users have been confirmed:
Candace
Brian
Alice
```

### 从列表中移除特定值的所有实例

在第3章中，我们使用了`remove()`函数从列表中移除特定的值。`remove()`函数之所以有效，是因为我们关心的值在列表中只出现一次。但是，如果你想从列表中移除某个值的所有实例呢？

假设你有一个包含多个`'cat'`值的宠物列表。要移除该值的所有实例，你可以运行一个`while`循环，直到`'cat'`不再出现在列表中，如下所示：

**pets.py**

```py
pets = ['dog', 'cat', 'dog', 'goldfish', 'cat', 'rabbit', 'cat']
print(pets)

while 'cat' in pets:
    pets.remove('cat')

print(pets)
```

我们从一个包含多个`'cat'`实例的列表开始。打印列表后，Python进入`while`循环，因为它至少发现一次`'cat'`出现在列表中。一旦进入循环，Python会移除第一个`'cat'`实例，返回到`while`行，然后当它发现`'cat'`仍然在列表中时再次进入循环。它会移除每一个`'cat'`实例，直到列表中不再有该值，此时Python退出循环并再次打印列表：

```py
['dog', 'cat', 'dog', 'goldfish', 'cat', 'rabbit', 'cat']
['dog', 'dog', 'goldfish', 'rabbit']
```

### 使用用户输入填充字典

你可以在每次通过`while`循环时提示用户输入所需的内容。让我们制作一个投票程序，其中每次通过循环时都提示参与者输入名字和回答。我们将把收集的数据存储在字典中，因为我们希望将每个回答与特定的用户关联：

**mountain_poll.py**

```py
responses = {}
# Set a flag to indicate that polling is active.
polling_active = True

while polling_active:
    # Prompt for the person's name and response.
❶     name = input("\nWhat is your name? ")
    response = input("Which mountain would you like to climb someday? ")

    # Store the response in the dictionary.
❷     responses[name] = response

    # Find out if anyone else is going to take the poll.
❸     repeat = input("Would you like to let another person respond? (yes/ no) ")
    if repeat == 'no':
        polling_active = False

# Polling is complete. Show the results.
print("\n--- Poll Results ---")
❹ for name, response in responses.items():
    print(f"{name} would like to climb {response}.")
```

程序首先定义一个空字典（`responses`），并设置一个标志（`polling_active`）来指示投票处于激活状态。只要`polling_active`为`True`，Python就会执行`while`循环中的代码。

在循环中，用户被提示输入他们的名字和想要攀登的山峰❶。这些信息会存储在`responses`字典中❷，然后用户会被询问是否继续进行投票❸。如果他们输入`yes`，程序会再次进入`while`循环。如果他们输入`no`，`polling_active`标志被设置为`False`，`while`循环停止运行，最后的代码块❹会显示投票结果。

如果你运行这个程序并输入一些示例响应，你应该看到类似这样的输出：

```py
What is your name? **Eric**
Which mountain would you like to climb someday? **Denali**
Would you like to let another person respond? (yes/ no) **yes**

What is your name? **Lynn**
Which mountain would you like to climb someday? **Devil's Thumb**
Would you like to let another person respond? (yes/ no) **no**

--- Poll Results ---
Eric would like to climb Denali.
Lynn would like to climb Devil's Thumb.
```

## 摘要

在这一章中，你学习了如何使用`input()`让用户在程序中提供自己的信息。你学会了处理文本和数字输入，并且掌握了如何使用`while`循环让程序根据用户的需求运行。你还看到几种控制`while`循环流程的方法，诸如设置`active`标志、使用`break`语句以及使用`continue`语句。你学习了如何使用`while`循环将项目从一个列表移动到另一个列表，并且如何从列表中移除某个值的所有实例。你还学到了如何在字典中使用`while`循环。

在第8章中，你将学习函数。*函数*允许你将程序分解成小块，每一块都执行一个特定的任务。你可以根据需要调用一个函数任意次数，也可以将函数存储在单独的文件中。通过使用函数，你将能够编写更高效的代码，这些代码更容易排除故障和维护，并且可以在多个不同的程序中重复使用。

# 8

函数

![](Images/chapterart.png)

在本章中，你将学习编写*函数*，它们是用于执行特定任务的命名代码块。当你想执行某个已经在函数中定义的特定任务时，你需要*调用*该函数。如果你需要在程序中多次执行该任务，你不需要一次又一次地编写相同的代码；只需调用专门处理该任务的函数，调用会告诉 Python 执行函数内的代码。你会发现，使用函数会使得你的程序更容易编写、阅读、测试和修复。

在本章中，你还将学习多种将信息传递给函数的方式。你将学习如何编写一些主要用于显示信息的函数，以及一些用于处理数据并返回一个或多个值的函数。最后，你将学习如何将函数存储在单独的文件中，这些文件被称为*模块*，以帮助组织你的主程序文件。

## 定义一个函数

这是一个简单的名为`greet_user()`的函数，用于打印问候语：

**greeter.py**

```py
def greet_user():
    """Display a simple greeting."""
    print("Hello!")

greet_user()
```

这个示例展示了函数最简单的结构。第一行使用关键字`def`来通知 Python 你正在定义一个函数。这是*函数定义*，它告诉 Python 函数的名称以及（如果适用）该函数执行任务所需的信息。括号中包含这些信息。在这个例子中，函数的名称是`greet_user()`，它不需要任何信息来完成工作，因此它的括号是空的。（尽管如此，括号仍然是必须的。）最后，定义以冒号结尾。

紧跟在`def greet_user():`之后的所有缩进行构成了函数的*主体*。第二行的文本是一个注释，称为*文档字符串*，它描述了该函数的功能。当 Python 为程序中的函数生成文档时，它会查找函数定义后的字符串。这些字符串通常用三重引号括起来，这样你就可以写多行文本。

`print("Hello!")`这一行是该函数主体中唯一的实际代码，因此`greet_user()`只有一个任务：`print("Hello!")`。

当你想使用这个函数时，必须调用它。*函数调用*告诉Python执行函数中的代码。要*调用*一个函数，你需要写出函数的名字，后面跟上括号内的必要信息。因为这里不需要额外的信息，所以调用我们的函数只需输入`greet_user()`。如预期，它会打印`Hello!`：

```py
Hello!
```

### 向函数传递信息

如果你稍微修改`greet_user()`函数，它可以通过名字问候用户。为了让函数实现这一功能，你需要在函数定义的`def greet_user()`的括号内输入`username`。通过在这里添加`username`，你允许函数接受你指定的任何`username`值。现在，每次调用该函数时，它都期望你提供一个`username`值。当你调用`greet_user()`时，你可以在括号内传递一个名字，比如`'jesse'`：

```py
def greet_user(username):
    """Display a simple greeting."""
    print(f"Hello, {username.title()}!")

greet_user('jesse')
```

输入`greet_user('jesse')`调用`greet_user()`并为函数提供执行`print()`调用所需的信息。函数接受你传递的名字，并显示该名字的问候语：

```py
Hello, Jesse!
```

同样地，输入`greet_user('sarah')`会调用`greet_user()`，并将`sarah`传递给它，然后打印`Hello, Sarah!`。你可以多次调用`greet_user()`并传递任何你想要的名字，每次都会产生可预测的输出。

### 参数和参数值

在前面的`greet_user()`函数中，我们定义了`greet_user()`需要一个`username`变量的值。当我们调用该函数并提供信息（一个人的名字）时，它打印出正确的问候语。

在`greet_user()`的定义中，变量`username`是*参数*的一个例子，它是函数完成任务所需的信息。在`greet_user('jesse')`中，值`'jesse'`是一个参数值，它是从函数调用传递给函数的信息。我们调用函数时，将我们希望函数处理的值放入括号内。在这个例子中，参数值`'jesse'`被传递给函数`greet_user()`，并赋值给参数`username`。

## 传递参数

由于一个函数定义可以有多个参数，函数调用可能需要多个参数。你可以通过多种方式向函数传递参数。你可以使用*位置参数*，它们需要按参数定义时的顺序传递；*关键字参数*，每个参数由变量名和值组成；以及值的列表和字典。我们将逐一看看这些方法。

### 位置参数

当你调用一个函数时，Python必须将函数调用中的每个参数与函数定义中的一个参数进行匹配。实现这一点最简单的方法是根据提供参数的顺序进行匹配。通过这种方式匹配的值被称为*位置参数*。

为了理解它是如何工作的，考虑一个展示宠物信息的函数。该函数告诉我们每只宠物是什么种类的动物以及宠物的名字，如下所示：

**pets.py**

```py
❶ def describe_pet(animal_type, pet_name):
    """Display information about a pet."""
    print(f"\nI have a {animal_type}.")
    print(f"My {animal_type}'s name is {pet_name.title()}.")

❷ describe_pet('hamster', 'harry')
```

定义表明这个函数需要一个动物的种类和动物的名字❶。当我们调用`describe_pet()`时，我们需要按顺序提供动物种类和名字。例如，在函数调用中，参数`'hamster'`被分配给`animal_type`，参数`'harry'`被分配给`pet_name`❷。在函数体内，这两个参数被用来显示正在描述的宠物信息。

输出描述了一只名叫Harry的仓鼠：

```py
I have a hamster.
My hamster's name is Harry.
```

#### 多次函数调用

你可以根据需要多次调用一个函数。描述第二个不同的宠物只需要再调用一次`describe_pet()`：

```py
def describe_pet(animal_type, pet_name):
 """Display information about a pet."""
 print(f"\nI have a {animal_type}.")
 print(f"My {animal_type}'s name is {pet_name.title()}.")

describe_pet('hamster', 'harry')
describe_pet('dog', 'willie')
```

在第二次函数调用中，我们将`describe_pet()`的参数设置为`'dog'`和`'willie'`。和之前使用的参数组一样，Python将`'dog'`与参数`animal_type`匹配，将`'willie'`与参数`pet_name`匹配。和之前一样，函数执行其功能，但这次它打印出一只名叫Willie的狗的值。现在我们有一只名叫Harry的仓鼠和一只名叫Willie的狗：

```py
I have a hamster.
My hamster's name is Harry.

I have a dog.
My dog's name is Willie.
```

多次调用一个函数是一个非常高效的工作方式。描述宠物的代码在函数中只需要写一次。然后，每当你想描述一只新宠物时，只需要用新宠物的信息再次调用该函数。即使描述宠物的代码扩展到10行，你依然可以通过调用函数一次，用一行代码描述一只新宠物。

#### 位置参数中的顺序很重要

如果你在函数调用中混淆了位置参数的顺序，可能会得到意想不到的结果：

```py
def describe_pet(animal_type, pet_name):
 """Display information about a pet."""
 print(f"\nI have a {animal_type}.")
 print(f"My {animal_type}'s name is {pet_name.title()}.")

describe_pet('harry', 'hamster')
```

在这个函数调用中，我们将名字列在前面，动物种类列在后面。因为这次`'harry'`被列在前面，所以这个值被分配给`animal_type`。同样，`'hamster'`被分配给`pet_name`。现在我们得到了一只名叫“Harry”的仓鼠：

```py
I have a harry.
My harry's name is Hamster.
```

如果你得到像这样的奇怪结果，请检查确保函数调用中的参数顺序与函数定义中的参数顺序一致。

### 关键字参数

一个*关键字参数*是你传递给函数的一个名称-值对。你在参数中直接关联名称和值，因此当你将参数传递给函数时，不会产生混淆（你不会把一只名为“哈利”的仓鼠变成狗）。关键字参数让你不必担心在函数调用中正确地排列参数的顺序，同时它们清晰地指明了每个值在函数调用中的角色。

让我们使用关键字参数重写*pets.py*来调用`describe_pet()`：

```py
def describe_pet(animal_type, pet_name):
 """Display information about a pet."""
 print(f"\nI have a {animal_type}.")
 print(f"My {animal_type}'s name is {pet_name.title()}.")

describe_pet(animal_type='hamster', pet_name='harry')
```

`describe_pet()`函数本身没有改变。但是当我们调用该函数时，我们显式告诉Python每个参数应与哪个实参匹配。当Python读取函数调用时，它知道将实参`'hamster'`分配给参数`animal_type`，并将实参`'harry'`分配给`pet_name`。输出结果正确地显示我们有一只名叫Harry的仓鼠。

关键字参数的顺序不重要，因为Python知道每个值应该放在哪个位置。以下两个函数调用是等效的：

```py
describe_pet(animal_type='hamster', pet_name='harry')
describe_pet(pet_name='harry', animal_type='hamster')
```

### 默认值

编写函数时，你可以为每个参数定义*默认值*。如果在函数调用中为某个参数提供了实参，Python 会使用该实参的值。如果没有提供，Python 会使用该参数的默认值。因此，当你为参数定义了默认值时，可以省略通常需要在函数调用中编写的相应实参。使用默认值可以简化你的函数调用，并明确函数通常的使用方式。

例如，如果你注意到大多数调用`describe_pet()`的地方都是用来描述狗的，你可以将`animal_type`的默认值设置为`'dog'`。现在，任何调用`describe_pet()`来描述狗的人都可以省略这一信息：

```py
def describe_pet(pet_name, animal_type='dog'):
    """Display information about a pet."""
 print(f"\nI have a {animal_type}.")
 print(f"My {animal_type}'s name is {pet_name.title()}.")

describe_pet(pet_name='willie')
```

我们更改了`describe_pet()`的定义，为`animal_type`添加了默认值`'dog'`。现在，当函数调用时没有指定`animal_type`，Python 会使用`'dog'`作为该参数的值：

```py
I have a dog.
My dog's name is Willie.
```

请注意，函数定义中参数的顺序必须更改。由于默认值使得不需要将动物类型作为实参，因此函数调用中唯一需要提供的实参就是宠物的名字。Python 仍然将其解释为位置参数，因此如果函数仅使用宠物的名字进行调用，该实参将与函数定义中列出的第一个参数匹配。这就是第一个参数需要是`pet_name`的原因。

现在使用此函数的最简单方法是只在函数调用中提供一只狗的名字：

```py
describe_pet('willie')
```

这个函数调用的输出与之前的示例相同。唯一提供的实参是`'willie'`，因此它与函数定义中的第一个参数`pet_name`匹配。因为没有为`animal_type`提供实参，Python 会使用默认值`'dog'`。

要描述一只不是狗的动物，你可以使用如下的函数调用：

```py
describe_pet(pet_name='harry', animal_type='hamster')
```

因为提供了`animal_type`的显式实参，Python 会忽略该参数的默认值。

### 等效的函数调用

因为位置参数、关键字参数和默认值可以一起使用，所以你通常会有几种等效的方式来调用一个函数。考虑以下为`describe_pet()`提供一个默认值的定义：

```py
def describe_pet(pet_name, animal_type='dog'):
```

根据这个定义，`pet_name`始终需要提供参数，这个值可以通过位置参数或关键字参数的形式提供。如果描述的动物不是狗，那么调用时必须包括`animal_type`的参数，这个参数也可以通过位置参数或关键字参数的形式指定。

以下所有调用都会适用于这个函数：

```py
# A dog named Willie.
describe_pet('willie')
describe_pet(pet_name='willie')

# A hamster named Harry.
describe_pet('harry', 'hamster')
describe_pet(pet_name='harry', animal_type='hamster')
describe_pet(animal_type='hamster', pet_name='harry')
```

每个函数调用的输出都将与之前的示例相同。

你使用哪种调用风格并不重要。只要你的函数调用能产生你想要的输出，就使用你最容易理解的风格。

### 避免参数错误

当你开始使用函数时，如果遇到关于不匹配参数的错误，不要感到惊讶。不匹配参数的错误发生在你提供的参数数量比函数所需的少或多。例如，如果我们尝试调用`describe_pet()`而不提供任何参数，会发生以下情况：

```py
def describe_pet(animal_type, pet_name):
 """Display information about a pet."""
 print(f"\nI have a {animal_type}.")
 print(f"My {animal_type}'s name is {pet_name.title()}.")

describe_pet()
```

Python 识别出函数调用中缺少了一些信息，错误追踪告诉我们：

```py
Traceback (most recent call last):
❶   File "pets.py", line 6, in <module>
❷     describe_pet()
    ^^^^^^^^^^^^^^
❸ TypeError: describe_pet() missing 2 required positional arguments: 
    'animal_type' and 'pet_name'
```

错误追踪首先告诉我们问题所在的位置 ❶，让我们回过头来看，发现问题出现在函数调用中。接下来，错误的函数调用被列出 ❷。最后，错误追踪告诉我们调用缺少两个参数，并报告缺少的参数名称 ❸。如果这个函数在一个独立的文件中，我们可能不需要打开文件并阅读函数代码，就能正确地重写调用。

Python 很有帮助，它会读取函数的代码并告诉我们需要提供的参数名称。这也是给变量和函数命名时使用描述性名称的另一个原因。如果这样做，Python 的错误信息将更有用，帮助你或其他使用你代码的人解决问题。

如果你提供了太多参数，你应该会得到类似的错误追踪信息，帮助你正确匹配函数调用与函数定义。

## 返回值

一个函数并不总是需要直接显示它的输出。相反，它可以处理一些数据，然后返回一个值或一组值。函数返回的值称为*返回值*。`return`语句将函数内部的值取出，并发送回调用该函数的行。返回值使得你可以将程序中许多繁琐的工作移到函数中，从而简化程序的主体部分。

### 返回一个简单的值

让我们看一个接受名字和姓氏并返回格式化全名的函数：

**formatted_name.py**

```py
def get_formatted_name(first_name, last_name):
    """Return a full name, neatly formatted."""
❶     full_name = f"{first_name} {last_name}"
❷     return full_name.title()

❸ musician = get_formatted_name('jimi', 'hendrix')
print(musician)
```

`get_formatted_name()`的定义接受两个参数：名字和姓氏。该函数将这两个名字组合起来，在它们之间添加一个空格，并将结果赋值给`full_name` ❶。`full_name`的值被转换为标题大小写，然后返回给调用行 ❷。

当你调用一个返回值的函数时，需要提供一个变量来接收返回的值。在这种情况下，返回的值被赋值给变量`musician` ❸。输出显示了一个整齐格式化的名字，由一个人的名字部分组成：

```py
Jimi Hendrix
```

这可能看起来像是为了得到一个格式化好的名字而做了很多工作，但我们也可以直接写成：

```py
print("Jimi Hendrix")
```

然而，当你考虑在一个需要分别存储许多名和姓的大型程序时，像`get_formatted_name()`这样的函数就变得非常有用。你可以分别存储名和姓，然后每当你想要显示全名时调用这个函数。

### 使参数变为可选

有时将一个参数设为可选是有意义的，这样使用该函数的人可以根据需要提供额外的信息。你可以使用默认值来使参数成为可选项。

例如，假设我们想要扩展`get_formatted_name()`以处理中间名。首次尝试包括中间名可能如下所示：

```py
def get_formatted_name(first_name, middle_name, last_name):
    """Return a full name, neatly formatted."""
    full_name = f"{first_name} {middle_name} {last_name}"
    return full_name.title()

musician = get_formatted_name('john', 'lee', 'hooker')
print(musician)
```

这个函数在提供名、中间名和姓时能正常工作。该函数接受名字的三个部分，然后将它们组合成一个字符串。函数会在适当的位置添加空格，并将全名转换为标题大小写：

```py
John Lee Hooker
```

但是中间名并非总是需要的，这个函数的写法如果你只传入名和姓时将无法正常工作。为了使中间名成为可选项，我们可以给`middle_name`参数一个空的默认值，并且只有在用户提供值时才处理该参数。为了让`get_formatted_name()`在没有中间名的情况下工作，我们将`middle_name`的默认值设为空字符串，并将其移动到参数列表的最后：

```py
def get_formatted_name(first_name, last_name, middle_name=''):
    """Return a full name, neatly formatted."""
❶     if middle_name:
        full_name = f"{first_name} {middle_name} {last_name}"
❷     else:
        full_name = f"{first_name} {last_name}"
    return full_name.title()

musician = get_formatted_name('jimi', 'hendrix')
print(musician)

❸ musician = get_formatted_name('john', 'hooker', 'lee')
print(musician)
```

在这个示例中，名字由三个可能的部分组成。因为总是有姓和名，所以这些参数在函数定义中列出。中间名是可选的，因此它在定义中列出最后，并且其默认值为空字符串。

在函数体内，我们检查是否提供了中间名。Python 将非空字符串解释为`True`，因此条件测试`if middle_name`如果函数调用中有中间名参数，结果会评估为`True` ❶。如果提供了中间名，首先、其次和最后的名字将被组合成一个全名。这个名字然后被转换为标题大小写，并返回到函数调用行，在那里它被赋值给变量`musician`并打印。如果没有提供中间名，空字符串将使`if`测试失败，`else`块将执行 ❷。全名将只由名和姓组成，格式化后的名字返回到调用行，在那里它被赋值给`musician`并打印。

用名字和姓氏调用这个函数非常简单。然而，如果我们使用中间名，就必须确保中间名是最后一个传入的参数，这样Python才能正确匹配位置参数 ❸。

这个修改后的版本适用于只有名字和姓氏的人，也适用于有中间名的人：

```py
Jimi Hendrix
John Lee Hooker
```

可选值使得函数能够处理广泛的使用场景，同时保持函数调用尽可能简洁。

### 返回字典

函数可以返回任何你需要的值，包括更复杂的数据结构，如列表和字典。例如，以下函数接受姓名的各个部分，并返回一个表示人的字典：

**person.py**

```py
def build_person(first_name, last_name):
    """Return a dictionary of information about a person."""
❶     person = {'first': first_name, 'last': last_name}
❷     return person

musician = build_person('jimi', 'hendrix')
❸ print(musician)
```

函数`build_person()`接受名字和姓氏，并将这些值放入一个字典中 ❶。`first_name`的值被存储在键`'first'`下，`last_name`的值被存储在键`'last'`下。然后，表示这个人的整个字典被返回 ❷。返回值被打印 ❸，原始的两条文本信息现在被存储在字典中：

```py
{'first': 'jimi', 'last': 'hendrix'}
```

这个函数接受简单的文本信息，并将其放入更有意义的数据结构中，这样你就可以在打印信息之外，对其进行更多操作。字符串`'jimi'`和`'hendrix'`现在被标记为名字和姓氏。你可以轻松地扩展这个函数，使其接受可选的值，比如中间名、年龄、职业或你想要存储的任何其他个人信息。例如，以下的更改允许你同时存储一个人的年龄：

```py
def build_person(first_name, last_name, age=None):
 """Return a dictionary of information about a person."""
 person = {'first': first_name, 'last': last_name}
    if age:
        person['age'] = age
 return person

musician = build_person('jimi', 'hendrix', age=27)
print(musician)
```

我们在函数定义中添加了一个新的可选参数`age`，并为该参数赋予特殊值`None`，用于表示一个变量没有具体的值。你可以把`None`看作占位符值。在条件测试中，`None`会被评估为`False`。如果函数调用中包含`age`的值，那么该值将被存储在字典中。这个函数始终存储一个人的名字，但也可以修改以存储关于此人的任何其他信息。

### 使用带`while`循环的函数

你可以使用所有迄今为止学过的Python结构与函数。例如，让我们使用`get_formatted_name()`函数与`while`循环结合，来更正式地问候用户。这里是第一次尝试使用用户的名字和姓氏来问候他们：

**greeter.py**

```py
def get_formatted_name(first_name, last_name):
 """Return a full name, neatly formatted."""
 full_name = f"{first_name} {last_name}"
 return full_name.title()

# This is an infinite loop!
while True:
❶     print("\nPlease tell me your name:")
    f_name = input("First name: ")
    l_name = input("Last name: ")

    formatted_name = get_formatted_name(f_name, l_name)
    print(f"\nHello, {formatted_name}!")
```

对于这个例子，我们使用了一个简单版本的`get_formatted_name()`，它不涉及中间名。`while`循环会要求用户输入他们的名字，我们分别提示用户输入他们的名字和姓氏 ❶。

但是这个`while`循环有一个问题：我们没有定义退出条件。当你请求一系列输入时，应该在哪里设置退出条件呢？我们希望用户尽可能容易地退出，因此每个提示都应提供一种退出方式。`break`语句为在任意提示中退出循环提供了一种简便的方法：

```py
def get_formatted_name(first_name, last_name):
 """Return a full name, neatly formatted."""
 full_name = f"{first_name} {last_name}"
 return full_name.title()

while True:
 print("\nPlease tell me your name:")
  print("(enter 'q' at any time to quit)")

 f_name = input("First name: ")
  if f_name == 'q':
        break

 l_name = input("Last name: ")
  if l_name == 'q':
        break

 formatted_name = get_formatted_name(f_name, l_name)
 print(f"\nHello, {formatted_name}!")
```

我们添加了一条消息，告知用户如何退出，然后如果用户在任意提示中输入退出值，我们就跳出循环。现在，程序将继续向用户问候，直到有人输入`q`作为名字：

```py
Please tell me your name:
(enter 'q' at any time to quit)
First name: **eric**
Last name: **matthes**

Hello, Eric Matthes!

Please tell me your name:
(enter 'q' at any time to quit)
First name: **q**
```

## 传递一个列表

无论是名字、数字还是更复杂的对象（例如字典），将一个列表传递给函数通常是非常有用的。当你将列表传递给函数时，函数可以直接访问列表的内容。让我们通过使用函数来提高处理列表的效率。

假设我们有一个用户列表，并希望向每个用户打印问候语。以下示例将一个名字列表传递给一个名为`greet_users()`的函数，该函数会逐一向列表中的每个人问好：

**greet_users.py**

```py
def greet_users(names):
    """Print a simple greeting to each user in the list."""
    for name in names:
        msg = f"Hello, {name.title()}!"
        print(msg)

usernames = ['hannah', 'ty', 'margot']
greet_users(usernames)
```

我们定义了`greet_users()`，使其期望接收一个名字列表，并将其分配给参数`names`。该函数会遍历它收到的列表，并向每个用户打印问候语。在函数外部，我们定义了一个用户列表，然后在函数调用中将列表`usernames`传递给`greet_users()`：

```py
Hello, Hannah!
Hello, Ty!
Hello, Margot!
```

这是我们想要的输出。每个用户都会看到个性化的问候语，且你可以在任何时候调用该函数来问候特定的用户群体。

### 在函数中修改一个列表

当你将一个列表传递给函数时，函数可以修改这个列表。在函数体内对列表所做的任何更改都是永久性的，这样即使你处理大量数据，也能高效工作。

考虑一个公司，该公司创建用户提交的设计的 3D 打印模型。需要打印的设计存储在一个列表中，打印完成后，它们会被移动到另一个单独的列表。以下代码在不使用函数的情况下实现这一功能：

**printing_models.py**

```py
# Start with some designs that need to be printed.
unprinted_designs = ['phone case', 'robot pendant', 'dodecahedron']
completed_models = []

# Simulate printing each design, until none are left.
#  Move each design to completed_models after printing.
while unprinted_designs:
    current_design = unprinted_designs.pop()
    print(f"Printing model: {current_design}")
    completed_models.append(current_design)

# Display all completed models.
print("\nThe following models have been printed:")
for completed_model in completed_models:
    print(completed_model)
```

这个程序以一个需要打印的设计列表开始，并有一个空列表`completed_models`，每个设计在打印完成后会被移到该列表中。只要`unprinted_designs`列表中还有设计，`while`循环就会通过从列表末尾移除一个设计、将其存储到`current_design`中，并显示当前设计正在打印的消息来模拟打印每个设计。然后，它会将该设计添加到已完成模型列表中。当循环完成后，将显示打印过的设计列表：

```py
Printing model: dodecahedron
Printing model: robot pendant
Printing model: phone case

The following models have been printed:
dodecahedron
robot pendant
phone case
```

我们可以通过编写两个函数来重新组织这段代码，每个函数执行一个特定的任务。大部分代码不会改变；我们只是更仔细地进行结构化。第一个函数将处理打印设计，第二个函数将总结已打印的设计：

```py
❶ def print_models(unprinted_designs, completed_models):
    """
    Simulate printing each design, until none are left.
 Move each design to completed_models after printing.
    """
    while unprinted_designs:
        current_design = unprinted_designs.pop()
        print(f"Printing model: {current_design}")
        completed_models.append(current_design)

❷ def show_completed_models(completed_models):
    """Show all the models that were printed."""
    print("\nThe following models have been printed:")
    for completed_model in completed_models:
        print(completed_model)

unprinted_designs = ['phone case', 'robot pendant', 'dodecahedron']
completed_models = []

print_models(unprinted_designs, completed_models)
show_completed_models(completed_models)
```

我们定义了`print_models()`函数，它有两个参数：一个是需要打印的设计列表，另一个是已完成模型列表❶。给定这两个列表，该函数通过清空未打印设计的列表并填充已完成模型的列表来模拟打印每个设计。然后我们定义了`show_completed_models()`函数，它有一个参数：已完成模型的列表❷。根据这个列表，`show_completed_models()`会显示每个已打印模型的名称。

这个程序的输出与没有函数的版本相同，但代码更有条理。做大部分工作的代码已被移到两个单独的函数中，这使得程序的主体部分更容易理解。看看程序主体，注意你能多么轻松地跟随发生的事情：

```py
unprinted_designs = ['phone case', 'robot pendant', 'dodecahedron']
completed_models = []

print_models(unprinted_designs, completed_models)
show_completed_models(completed_models)
```

我们建立了一个未打印设计的列表和一个空的已完成模型列表。然后，因为我们已经定义了两个函数，我们所需要做的就是调用它们并传递正确的参数。我们调用`print_models()`并传递它所需要的两个列表；正如预期的那样，`print_models()`模拟打印这些设计。然后我们调用`show_completed_models()`并传递已完成模型的列表，以便它能报告已打印的模型。描述性的函数名称让其他人即使没有注释也能阅读这段代码并理解它。

这个程序比没有函数的版本更容易扩展和维护。如果我们以后需要打印更多设计，只需再次调用`print_models()`。如果我们意识到打印代码需要修改，我们只需修改一次代码，这些修改就会在函数被调用的所有地方生效。这种方法比在程序中的多个地方分别更新代码更高效。

```py`This example also demonstrates the idea that every function should have one specific job. The first function prints each design, and the second displays the completed models. This is more beneficial than using one function to do both jobs. If you’re writing a function and notice the function is doing too many different tasks, try to split the code into two functions. Remember that you can always call a function from another function, which can be helpful when splitting a complex task into a series of steps.    ### Preventing a Function from Modifying a List    Sometimes you’ll want to prevent a function from modifying a list. For example, say that you start with a list of unprinted designs and write a function to move them to a list of completed models, as in the previous example. You may decide that even though you’ve printed all the designs, you want to keep the original list of unprinted designs for your records. But because you moved all the design names out of `unprinted_designs`, the list is now empty, and the empty list is the only version you have; the original is gone. In this case, you can address this issue by passing the function a copy of the list, not the original. Any changes the function makes to the list will affect only the copy, leaving the original list intact.    You can send a copy of a list to a function like this:    ``` `function_name`(`list_name`[:]) ```py    The slice notation `[:]` makes a copy of the list to send to the function. If we didn’t want to empty the list of unprinted designs in *printing_models.py*, we could call `print_models()` like this:    ``` print_models(unprinted_designs[:], completed_models) ```py    The function `print_models()` can do its work because it still receives the names of all unprinted designs. But this time it uses a copy of the original unprinted designs list, not the actual `unprinted_designs` list. The list `completed_models` will fill up with the names of printed models like it did before, but the original list of unprinted designs will be unaffected by the function.    Even though you can preserve the contents of a list by passing a copy of it to your functions, you should pass the original list to functions unless you have a specific reason to pass a copy. It’s more efficient for a function to work with an existing list, because this avoids using the time and memory needed to make a separate copy. This is especially true when working with large lists.    ## Passing an Arbitrary Number of Arguments    Sometimes you won’t know ahead of time how many arguments a function needs to accept. Fortunately, Python allows a function to collect an arbitrary number of arguments from the calling statement.    For example, consider a function that builds a pizza. It needs to accept a number of toppings, but you can’t know ahead of time how many toppings a person will want. The function in the following example has one parameter, `*toppings`, but this parameter collects as many arguments as the calling line provides:    **pizza.py**    ``` def make_pizza(*toppings): """打印请求的配料列表。""" print(toppings) make_pizza('pepperoni') make_pizza('mushrooms', 'green peppers', 'extra cheese') ```py    The asterisk in the parameter name `*toppings` tells Python to make a tuple called `toppings`, containing all the values this function receives. The `print()` call in the function body produces output showing that Python can handle a function call with one value and a call with three values. It treats the different calls similarly. Note that Python packs the arguments into a tuple, even if the function receives only one value:    ``` ('pepperoni',) ('mushrooms', 'green peppers', 'extra cheese') ```py    Now we can replace the `print()` call with a loop that runs through the list of toppings and describes the pizza being ordered:    ``` def make_pizza(*toppings): """总结一下我们要做披萨。""" print("\nMaking a pizza with the following toppings:") for topping in toppings: print(f"- {topping}") make_pizza('pepperoni') make_pizza('mushrooms', 'green peppers', 'extra cheese') ```py    The function responds appropriately, whether it receives one value or three values:    ``` Making a pizza with the following toppings: - pepperoni Making a pizza with the following toppings: - mushrooms - green peppers - extra cheese ```py    This syntax works no matter how many arguments the function receives.    ### Mixing Positional and Arbitrary Arguments    If you want a function to accept several different kinds of arguments, the parameter that accepts an arbitrary number of arguments must be placed last in the function definition. Python matches positional and keyword arguments first and then collects any remaining arguments in the final parameter.    For example, if the function needs to take in a size for the pizza, that parameter must come before the parameter `*toppings`:    ``` def make_pizza(size, *toppings): """总结一下我们要做披萨。""" print(f"\nMaking a {size}-inch pizza with the following toppings:") for topping in toppings: print(f"- {topping}") make_pizza(16, 'pepperoni') make_pizza(12, 'mushrooms', 'green peppers', 'extra cheese') ```py    In the function definition, Python assigns the first value it receives to the parameter `size`. All other values that come after are stored in the tuple `toppings`. The function calls include an argument for the size first, followed by as many toppings as needed.    Now each pizza has a size and a number of toppings, and each piece of information is printed in the proper place, showing size first and toppings after:    ``` Making a 16-inch pizza with the following toppings: - pepperoni Making a 12-inch pizza with the following toppings: - mushrooms - green peppers - extra cheese ```py    ### Using Arbitrary Keyword Arguments    Sometimes you’ll want to accept an arbitrary number of arguments, but you won’t know ahead of time what kind of information will be passed to the function. In this case, you can write functions that accept as many key-value pairs as the calling statement provides. One example involves building user profiles: you know you’ll get information about a user, but you’re not sure what kind of information you’ll receive. The function `build_profile()` in the following example always takes in a first and last name, but it accepts an arbitrary number of keyword arguments as well:    **user_profile.py**    ``` def build_profile(first, last, **user_info): """创建一个包含用户所有信息的字典。""" ❶ user_info['first_name'] = first user_info['last_name'] = last return user_info user_profile = build_profile('albert', 'einstein', location='princeton', field='physics') print(user_profile) ```py    The definition of `build_profile()` expects a first and last name, and then it allows the user to pass in as many name-value pairs as they want. The double asterisks before the parameter `**user_info` cause Python to create a dictionary called `user_info` containing all the extra name-value pairs the function receives. Within the function, you can access the key-value pairs in `user_info` just as you would for any dictionary.    In the body of `build_profile()`, we add the first and last names to the `user_info` dictionary because we’ll always receive these two pieces of information from the user ❶, and they haven’t been placed into the dictionary yet. Then we return the `user_info` dictionary to the function call line.    We call `build_profile()`, passing it the first name `'albert'`, the last name `'einstein'`, and the two key-value pairs `location='princeton'` and `field='physics'`. We assign the returned `profile` to `user_profile` and print `user_profile`:    ``` {'location': 'princeton', 'field': 'physics', 'first_name': 'albert', 'last_name': 'einstein'} ```py    The returned dictionary contains the user’s first and last names and, in this case, the location and field of study as well. The function will work no matter how many additional key-value pairs are provided in the function call.    You can mix positional, keyword, and arbitrary values in many different ways when writing your own functions. It’s useful to know that all these argument types exist because you’ll see them often when you start reading other people’s code. It takes practice to use the different types correctly and to know when to use each type. For now, remember to use the simplest approach that gets the job done. As you progress, you’ll learn to use the most efficient approach each time.    ## Storing Your Functions in Modules    One advantage of functions is the way they separate blocks of code from your main program. When you use descriptive names for your functions, your programs become much easier to follow. You can go a step further by storing your functions in a separate file called a *module* and then *importing* that module into your main program. An `import` statement tells Python to make the code in a module available in the currently running program file.    Storing your functions in a separate file allows you to hide the details of your program’s code and focus on its higher-level logic. It also allows you to reuse functions in many different programs. When you store your functions in separate files, you can share those files with other programmers without having to share your entire program. Knowing how to import functions also allows you to use libraries of functions that other programmers have written.    There are several ways to import a module, and I’ll show you each of these briefly.    ### Importing an Entire Module    To start importing functions, we first need to create a module. A *module* is a file ending in *.py* that contains the code you want to import into your program. Let’s make a module that contains the function `make_pizza()`. To make this module, we’ll remove everything from the file *pizza.py* except the function `make_pizza()`:    **pizza.py**    ``` def make_pizza(size, *toppings): """总结一下我们要做披萨。""" print(f"\nMaking a {size}-inch pizza with the following toppings:") for topping in toppings: print(f"- {topping}") ```py    Now we’ll make a separate file called *making_pizzas.py* in the same directory as *pizza.py*. This file imports the module we just created and then makes two calls to `make_pizza()`:    **making_pizzas.py**    ``` import pizza ❶ pizza.make_pizza(16, 'pepperoni') pizza.make_pizza(12, 'mushrooms', 'green peppers', 'extra cheese') ```py    When Python reads this file, the line `import pizza` tells Python to open the file *pizza.py* and copy all the functions from it into this program. You don’t actually see code being copied between files because Python copies the code behind the scenes, just before the program runs. All you need to know is that any function defined in *pizza.py* will now be available in *making_pizzas.py*.    To call a function from an imported module, enter the name of the module you imported, `pizza`, followed by the name of the function, `make_pizza()`, separated by a dot ❶. This code produces the same output as the original program that didn’t import a module:    ``` Making a 16-inch pizza with the following toppings: - pepperoni Making a 12-inch pizza with the following toppings: - mushrooms - green peppers - extra cheese ```py    This first approach to importing, in which you simply write `import` followed by the name of the module, makes every function from the module available in your program. If you use this kind of `import` statement to import an entire module named *module_name.py*, each function in the module is available through the following syntax:    ``` `module_name`.`function_name`() ```py    ### Importing Specific Functions    You can also import a specific function from a module. Here’s the general syntax for this approach:    ``` from `module_name` import `function_name` ```py    You can import as many functions as you want from a module by separating each function’s name with a comma:    ``` from `module_name` import `function_0`, `function_1`, `function_2` ```py    The *making_pizzas.py* example would look like this if we want to import just the function we’re going to use:    ``` from pizza import make_pizza make_pizza(16, 'pepperoni') make_pizza(12, 'mushrooms', 'green peppers', 'extra cheese') ```py    With this syntax, you don’t need to use the dot notation when you call a function. Because we’ve explicitly imported the function `make_pizza()` in the `import` statement, we can call it by name when we use the function.    ### Using as to Give a Function an Alias    If the name of a function you’re importing might conflict with an existing name in your program, or if the function name is long, you can use a short, unique *alias*—an alternate name similar to a nickname for the function. You’ll give the function this special nickname when you import the function.    Here we give the function `make_pizza()` an alias, `mp()`, by importing `make_pizza as mp`. The `as` keyword renames a function using the alias you provide:    ``` from pizza import make_pizza as mp mp(16, 'pepperoni') mp(12, 'mushrooms', 'green peppers', 'extra cheese') ```py    The `import` statement shown here renames the function `make_pizza()` to `mp()` in this program. Anytime we want to call `make_pizza()` we can simply write `mp()` instead, and Python will run the code in `make_pizza()` while avoiding any confusion with another `make_pizza()` function you might have written in this program file.    The general syntax for providing an alias is:    ``` from `module_name` import `function_name` as `fn` ```py    ### Using as to Give a Module an Alias    You can also provide an alias for a module name. Giving a module a short alias, like `p` for `pizza`, allows you to call the module’s functions more quickly. Calling `p.make_pizza()` is more concise than calling `pizza.make_pizza()`:    ``` import pizza as p p.make_pizza(16, 'pepperoni') p.make_pizza(12, 'mushrooms', 'green peppers', 'extra cheese') ```py    The module `pizza` is given the alias `p` in the `import` statement, but all of the module’s functions retain their original names. Calling the functions by writing `p.make_pizza()` is not only more concise than `pizza.make_pizza()`, but it also redirects your attention from the module name and allows you to focus on the descriptive names of its functions. These function names, which clearly tell you what each function does, are more important to the readability of your code than using the full module name.    The general syntax for this approach is:    ``` import `module_name` as `mn` ```py    ### Importing All Functions in a Module    You can tell Python to import every function in a module by using the asterisk (`*`) operator:    ``` from pizza import * make_pizza(16, 'pepperoni') make_pizza(12, 'mushrooms', 'green peppers', 'extra cheese') ```py    The asterisk in the `import` statement tells Python to copy every function from the module `pizza` into this program file. Because every function is imported, you can call each function by name without using the dot notation. However, it’s best not to use this approach when you’re working with larger modules that you didn’t write: if the module has a function name that matches an existing name in your project, you can get unexpected results. Python may see several functions or variables with the same name, and instead of importing all the functions separately, it will overwrite the functions.    The best approach is to import the function or functions you want, or import the entire module and use the dot notation. This leads to clear code that’s easy to read and understand. I include this section so you’ll recognize `import` statements like the following when you see them in other people’s code:    ``` from `module_name` import * ```py    ## Styling Functions    You need to keep a few details in mind when you’re styling functions. Functions should have descriptive names, and these names should use lowercase letters and underscores. Descriptive names help you and others understand what your code is trying to do. Module names should use these conventions as well.    Every function should have a comment that explains concisely what the function does. This comment should appear immediately after the function definition and use the docstring format. In a well-documented function, other programmers can use the function by reading only the description in the docstring. They should be able to trust that the code works as described, and as long as they know the name of the function, the arguments it needs, and the kind of value it returns, they should be able to use it in their programs.    If you specify a default value for a parameter, no spaces should be used on either side of the equal sign:    ``` def `function_name`(`parameter_0`, `parameter_1`='`default value`') ```py    The same convention should be used for keyword arguments in function calls:    ``` `function_name`(`value_0`, `parameter_1`='`value`') ```py    PEP 8 ([https://www.python.org/dev/peps/pep-0008](https://www.python.org/dev/peps/pep-0008)) recommends that you limit lines of code to 79 characters so every line is visible in a reasonably sized editor window. If a set of parameters causes a function’s definition to be longer than 79 characters, press ENTER after the opening parenthesis on the definition line. On the next line, press the TAB key twice to separate the list of arguments from the body of the function, which will only be indented one level.    Most editors automatically line up any additional lines of arguments to match the indentation you have established on the first line:    ``` def `function_name`( `parameter_0`, `parameter_1`, `parameter_2`, `parameter_3`, `parameter_4`, `parameter_5`): `function body...` ```py    If your program or module has more than one function, you can separate each by two blank lines to make it easier to see where one function ends and the next one begins.    All `import` statements should be written at the beginning of a file. The only exception is if you use comments at the beginning of your file to describe the overall program.    ## Summary    In this chapter, you learned how to write functions and to pass arguments so that your functions have access to the information they need to do their work. You learned how to use positional and keyword arguments, and also how to accept an arbitrary number of arguments. You saw functions that display output and functions that return values. You learned how to use functions with lists, dictionaries, `if` statements, and `while` loops. You also saw how to store your functions in separate files called *modules*, so your program files will be simpler and easier to understand. Finally, you learned to style your functions so your programs will continue to be well-structured and as easy as possible for you and others to read.    One of your goals as a programmer should be to write simple code that does what you want it to, and functions help you do this. They allow you to write blocks of code and leave them alone once you know they work. When you know a function does its job correctly, you can trust that it will continue to work and move on to your next coding task.    Functions allow you to write code once and then reuse that code as many times as you want. When you need to run the code in a function, all you need to do is write a one-line call and the function does its job. When you need to modify a function’s behavior, you only have to modify one block of code, and your change takes effect everywhere you’ve made a call to that function.    Using functions makes your programs easier to read, and good function names summarize what each part of a program does. Reading a series of function calls gives you a much quicker sense of what a program does than reading a long series of code blocks.    Functions also make your code easier to test and debug. When the bulk of your program’s work is done by a set of functions, each of which has a specific job, it’s much easier to test and maintain the code you’ve written. You can write a separate program that calls each function and tests whether each function works in all the situations it may encounter. When you do this, you can be confident that your functions will work properly each time you call them.    In Chapter 9, you’ll learn to write classes. *Classes* combine functions and data into one neat package that can be used in flexible and efficient ways.``` `  ```py`# 9 Classes  ![](Images/chapterart.png)  *Object-oriented programming (OOP**)* is one of the most effective approaches to writing software. In object-oriented programming, you write *classes* that represent real-world things and situations, and you create *objects* based on these classes. When you write a class, you define the general behavior that a whole category of objects can have.    When you create individual objects from the class, each object is automatically equipped with the general behavior; you can then give each object whatever unique traits you desire. You’ll be amazed how well real-world situations can be modeled with object-oriented programming.    Making an object from a class is called *instantiation*, and you work with *instances* of a class. In this chapter you’ll write classes and create instances of those classes. You’ll specify the kind of information that can be stored in instances, and you’ll define actions that can be taken with these instances. You’ll also write classes that extend the functionality of existing classes, so similar classes can share common functionality, and you can do more with less code. You’ll store your classes in modules and import classes written by other programmers into your own program files.    Learning about object-oriented programming will help you see the world as a programmer does. It’ll help you understand your code—not just what’s happening line by line, but also the bigger concepts behind it. Knowing the logic behind classes will train you to think logically, so you can write programs that effectively address almost any problem you encounter.    Classes also make life easier for you and the other programmers you’ll work with as you take on increasingly complex challenges. When you and other programmers write code based on the same kind of logic, you’ll be able to understand each other’s work. Your programs will make sense to the people you work with, allowing everyone to accomplish more.    ## Creating and Using a Class    You can model almost anything using classes. Let’s start by writing a simple class, `Dog`, that represents a dog—not one dog in particular, but any dog. What do we know about most pet dogs? Well, they all have a name and an age. We also know that most dogs sit and roll over. Those two pieces of information (name and age) and those two behaviors (sit and roll over) will go in our `Dog` class because they’re common to most dogs. This class will tell Python how to make an object representing a dog. After our class is written, we’ll use it to make individual instances, each of which represents one specific dog.    ### Creating the Dog Class    Each instance created from the `Dog` class will store a `name` and an `age`, and we’ll give each dog the ability to `sit()` and `roll_over()`:    **dog.py**    ``` ❶ class Dog: """一个简单的模拟狗的尝试。""" ❷ def __init__(self, name, age): """初始化姓名和年龄属性。""" ❸ self.name = name self.age = age ❹ def sit(self): """模拟狗根据命令坐下。""" print(f"{self.name} is now sitting.") def roll_over(self): """模拟根据命令翻滚。""" print(f"{self.name} rolled over!") ```py    There’s a lot to notice here, but don’t worry. You’ll see this structure throughout this chapter and have lots of time to get used to it. We first define a class called `Dog` ❶. By convention, capitalized names refer to classes in Python. There are no parentheses in the class definition because we’re creating this class from scratch. We then write a docstring describing what this class does.    ### The __init__() Method    A function that’s part of a class is a *method*. Everything you learned about functions applies to methods as well; the only practical difference for now is the way we’ll call methods. The `__init__()` method ❷ is a special method that Python runs automatically whenever we create a new instance based on the `Dog` class. This method has two leading underscores and two trailing underscores, a convention that helps prevent Python’s default method names from conflicting with your method names. Make sure to use two underscores on each side of `__init__()`. If you use just one on each side, the method won’t be called automatically when you use your class, which can result in errors that are difficult to identify.    We define the `__init__()` method to have three parameters: `self`, `name`, and `age`. The `self` parameter is required in the method definition, and it must come first, before the other parameters. It must be included in the definition because when Python calls this method later (to create an instance of `Dog`), the method call will automatically pass the `self` argument. Every method call associated with an instance automatically passes `self`, which is a reference to the instance itself; it gives the individual instance access to the attributes and methods in the class. When we make an instance of `Dog`, Python will call the `__init__()` method from the `Dog` class. We’ll pass `Dog()` a name and an age as arguments; `self` is passed automatically, so we don’t need to pass it. Whenever we want to make an instance from the `Dog` class, we’ll provide values for only the last two parameters, `name` and `age`.    The two variables defined in the body of the `__init__()` method each have the prefix `self` ❸. Any variable prefixed with `self` is available to every method in the class, and we’ll also be able to access these variables through any instance created from the class. The line `self.name = name` takes the value associated with the parameter `name` and assigns it to the variable `name`, which is then attached to the instance being created. The same process happens with `self.age = age`. Variables that are accessible through instances like this are called *attributes*.    The `Dog` class has two other methods defined: `sit()` and `roll_over()` ❹. Because these methods don’t need additional information to run, we just define them to have one parameter, `self`. The instances we create later will have access to these methods. In other words, they’ll be able to sit and roll over. For now, `sit()` and `roll_over()` don’t do much. They simply print a message saying the dog is sitting or rolling over. But the concept can be extended to realistic situations: if this class were part of a computer game, these methods would contain code to make an animated dog sit and roll over. If this class was written to control a robot, these methods would direct movements that cause a robotic dog to sit and roll over.    ### Making an Instance from a Class    Think of a class as a set of instructions for how to make an instance. The `Dog` class is a set of instructions that tells Python how to make individual instances representing specific dogs.    Let’s make an instance representing a specific dog:    ``` class Dog: *--snip--* ❶ my_dog = Dog('Willie', 6) ❷ print(f"My dog's name is {my_dog.name}.") ❸ print(f"My dog is {my_dog.age} years old.") ```py    The `Dog` class we’re using here is the one we just wrote in the previous example. Here, we tell Python to create a dog whose name is `'Willie'` and whose age is `6` ❶. When Python reads this line, it calls the `__init__()` method in `Dog` with the arguments `'Willie'` and `6`. The `__init__()` method creates an instance representing this particular dog and sets the `name` and `age` attributes using the values we provided. Python then returns an instance representing this dog. We assign that instance to the variable `my_dog`. The naming convention is helpful here; we can usually assume that a capitalized name like `Dog` refers to a class, and a lowercase name like `my_dog` refers to a single instance created from a class.    #### Accessing Attributes    To access the attributes of an instance, you use dot notation. We access the value of `my_dog`’s attribute `name` ❷ by writing:    ``` my_dog.name ```py    Dot notation is used often in Python. This syntax demonstrates how Python finds an attribute’s value. Here, Python looks at the instance `my_dog` and then finds the attribute `name` associated with `my_dog`. This is the same attribute referred to as `self.name` in the class `Dog`. We use the same approach to work with the attribute `age` ❸.    The output is a summary of what we know about `my_dog`:    ``` My dog's name is Willie. My dog is 6 years old. ```py    #### Calling Methods    After we create an instance from the class `Dog`, we can use dot notation to call any method defined in `Dog`. Let’s make our dog sit and roll over:    ``` class Dog: *--snip--* my_dog = Dog('Willie', 6) my_dog.sit() my_dog.roll_over() ```py    To call a method, give the name of the instance (in this case, `my_dog`) and the method you want to call, separated by a dot. When Python reads `my_dog.sit()`, it looks for the method `sit()` in the class `Dog` and runs that code. Python interprets the line `my_dog.roll_over()` in the same way.    Now Willie does what we tell him to:    ``` Willie is now sitting. Willie rolled over! ```py    This syntax is quite useful. When attributes and methods have been given appropriately descriptive names like `name`, `age`, `sit()`, and `roll_over()`, we can easily infer what a block of code, even one we’ve never seen before, is supposed to do.    #### Creating Multiple Instances    You can create as many instances from a class as you need. Let’s create a second dog called `your_dog`:    ``` class Dog: *--snip--* my_dog = Dog('Willie', 6) your_dog = Dog('Lucy', 3) print(f"My dog's name is {my_dog.name}.") print(f"My dog is {my_dog.age} years old.") my_dog.sit() print(f"\nYour dog's name is {your_dog.name}.") print(f"Your dog is {your_dog.age} years old.") your_dog.sit() ```py    In this example we create a dog named Willie and a dog named Lucy. Each dog is a separate instance with its own set of attributes, capable of the same set of actions:    ``` My dog's name is Willie. My dog is 6 years old. Willie is now sitting. Your dog's name is Lucy. Your dog is 3 years old. Lucy is now sitting. ```py    Even if we used the same name and age for the second dog, Python would still create a separate instance from the `Dog` class. You can make as many instances from one class as you need, as long as you give each instance a unique variable name or it occupies a unique spot in a list or dictionary.    ## Working with Classes and Instances    You can use classes to represent many real-world situations. Once you write a class, you’ll spend most of your time working with instances created from that class. One of the first tasks you’ll want to do is modify the attributes associated with a particular instance. You can modify the attributes of an instance directly or write methods that update attributes in specific ways.    ### The Car Class    Let’s write a new class representing a car. Our class will store information about the kind of car we’re working with, and it will have a method that summarizes this information:    **car.py**    ``` class Car: """一个简单的表示汽车的尝试。""" ❶ def __init__(self, make, model, year): """初始化属性以描述汽车。""" self.make = make self.model = model self.year = year ❷ def get_descriptive_name(self): """返回一个格式整洁的描述性名称。""" long_name = f"{self.year} {self.make} {self.model}" return long_name.title() ❸ my_new_car = Car('audi', 'a4', 2024) print(my_new_car.get_descriptive_name()) ```py    In the `Car` class, we define the `__init__()` method with the `self` parameter first ❶, just like we did with the `Dog` class. We also give it three other parameters: `make`, `model`, and `year`. The `__init__()` method takes in these parameters and assigns them to the attributes that will be associated with instances made from this class. When we make a new `Car` instance, we’ll need to specify a make, model, and year for our instance.    We define a method called `get_descriptive_name()` ❷ that puts a car’s `year`, `make`, and `model` into one string neatly describing the car. This will spare us from having to print each attribute’s value individually. To work with the attribute values in this method, we use `self.make`, `self.model`, and `self.year`. Outside of the class, we make an instance from the `Car` class and assign it to the variable `my_new_car` ❸. Then we call `get_descriptive_name()` to show what kind of car we have:    ``` 2024 Audi A4 ```py    To make the class more interesting, let’s add an attribute that changes over time. We’ll add an attribute that stores the car’s overall mileage.    ### Setting a Default Value for an Attribute    When an instance is created, attributes can be defined without being passed in as parameters. These attributes can be defined in the `__init__()` method, where they are assigned a default value.    Let’s add an attribute called `odometer_reading` that always starts with a value of 0\. We’ll also add a method `read_odometer()` that helps us read each car’s odometer:    ``` class Car: def __init__(self, make, model, year): """初始化属性以描述汽车。""" self.make = make self.model = model self.year = year ❶ self.odometer_reading = 0 def get_descriptive_name(self): *--snip--* ❷ def read_odometer(self): """打印一条显示汽车里程的语句。""" print(f"This car has {self.odometer_reading} miles on it.") my_new_car = Car('audi', 'a4', 2024) print(my_new_car.get_descriptive_name()) my_new_car.read_odometer() ```py    This time, when Python calls the `__init__()` method to create a new instance, it stores the make, model, and year values as attributes, like it did in the previous example. Then Python creates a new attribute called `odometer_reading` and sets its initial value to 0 ❶. We also have a new method called `read_odometer()` ❷ that makes it easy to read a car’s mileage.    Our car starts with a mileage of 0:    ``` 2024 Audi A4 This car has 0 miles on it. ```py    Not many cars are sold with exactly 0 miles on the odometer, so we need a way to change the value of this attribute.    ### Modifying Attribute Values    You can change an attribute’s value in three ways: you can change the value directly through an instance, set the value through a method, or increment the value (add a certain amount to it) through a method. Let’s look at each of these approaches.    #### Modifying an Attribute’s Value Directly    The simplest way to modify the value of an attribute is to access the attribute directly through an instance. Here we set the odometer reading to 23 directly:    ``` class Car: *--snip--* my_new_car = Car('audi', 'a4', 2024) print(my_new_car.get_descriptive_name()) my_new_car.odometer_reading = 23 my_new_car.read_odometer() ```py    We use dot notation to access the car’s `odometer_reading` attribute, and set its value directly. This line tells Python to take the instance `my_new_car`, find the attribute `odometer_reading` associated with it, and set the value of that attribute to 23:    ``` 2024 Audi A4 This car has 23 miles on it. ```py    Sometimes you’ll want to access attributes directly like this, but other times you’ll want to write a method that updates the value for you.    #### Modifying an Attribute’s Value Through a Method    It can be helpful to have methods that update certain attributes for you. Instead of accessing the attribute directly, you pass the new value to a method that handles the updating internally.    Here’s an example showing a method called `update_odometer()`:    ``` class Car: *--snip--* def update_odometer(self, mileage): """将里程表读数设置为给定值。""" self.odometer_reading = mileage my_new_car = Car('audi', 'a4', 2024) print(my_new_car.get_descriptive_name()) ❶ my_new_car.update_odometer(23) my_new_car.read_odometer() ```py    The only modification to `Car` is the addition of `update_odometer()`. This method takes in a mileage value and assigns it to `self.odometer_reading`. Using the `my_new_car` instance, we call `update_odometer()` with `23` as an argument ❶. This sets the odometer reading to 23, and `read_odometer()` prints the reading:    ``` 2024 Audi A4 This car has 23 miles on it. ```py    We can extend the method `update_odometer()` to do additional work every time the odometer reading is modified. Let’s add a little logic to make sure no one tries to roll back the odometer reading:    ``` class Car: *--snip--* def update_odometer(self, mileage): """ 将里程表读数设置为给定值。 如果尝试倒转里程表，则拒绝更改。 """ ❶ if mileage >= self.odometer_reading: self.odometer_reading = mileage else: ❷ print("You can't roll back an odometer!") ```py    Now `update_odometer()` checks that the new reading makes sense before modifying the attribute. If the value provided for `mileage` is greater than or equal to the existing mileage, `self.odometer_reading`, you can update the odometer reading to the new mileage ❶. If the new mileage is less than the existing mileage, you’ll get a warning that you can’t roll back an odometer ❷.    #### Incrementing an Attribute’s Value Through a Method    Sometimes you’ll want to increment an attribute’s value by a certain amount, rather than set an entirely new value. Say we buy a used car and put 100 miles on it between the time we buy it and the time we register it. Here’s a method that allows us to pass this incremental amount and add that value to the odometer reading:    ``` class Car: *--snip--* def update_odometer(self, mileage): *--snip--* def increment_odometer(self, miles): """将给定的量添加到里程表读数。""" self.odometer_reading += miles ❶ my_used_car = Car('subaru', 'outback', 2019) print(my_used_car.get_descriptive_name()) ❷ my_used_car.update_odometer(23_500) my_used_car.read_odometer() my_used_car.increment_odometer(100) my_used_car.read_odometer() ```py    The new method `increment_odometer()` takes in a number of miles, and adds this value to `self.odometer_reading`. First, we create a used car, `my_used_car` ❶. We set its odometer to 23,500 by calling `update_odometer()` and passing it `23_500` ❷. Finally, we call `increment_odometer()` and pass it `100` to add the 100 miles that we drove between buying the car and registering it:    ``` 2019 Subaru Outback This car has 23500 miles on it. This car has 23600 miles on it. ```py    You can modify this method to reject negative increments so no one uses this function to roll back an odometer as well.    ## Inheritance    You don’t always have to start from scratch when writing a class. If the class you’re writing is a specialized version of another class you wrote, you can use *inheritance*. When one class *inherits* from another, it takes on the attributes and methods of the first class. The original class is called the *parent class*, and the new class is the *child class*. The child class can inherit any or all of the attributes and methods of its parent class, but it’s also free to define new attributes and methods of its own.    ### The __init__() Method for a Child Class    When you’re writing a new class based on an existing class, you’ll often want to call the `__init__()` method from the parent class. This will initialize any attributes that were defined in the parent `__init__()` method and make them available in the child class.    As an example, let’s model an electric car. An electric car is just a specific kind of car, so we can base our new `ElectricCar` class on the `Car` class we wrote earlier. Then we’ll only have to write code for the attributes and behaviors specific to electric cars.    Let’s start by making a simple version of the `ElectricCar` class, which does everything the `Car` class does:    **electric_car.py**    ``` ❶ class Car: """一个简单的表示汽车的尝试。""" def __init__(self, make, model, year): """初始化属性以描述汽车。""" self.make = make self.model = model self.year = year self.odometer_reading = 0 def get_descriptive_name(self): """返回一个格式整洁的描述性名称。""" long_name = f"{self.year} {self.make} {self.model}" return long_name.title() def read_odometer(self): """打印一条显示汽车里程的语句。""" print(f"This car has {self.odometer_reading} miles on it.") def update_odometer(self, mileage): """将里程表读数设置为给定值。""" if mileage >= self.odometer_reading: self.odometer_reading = mileage else: print("You can't roll back an odometer!") def increment_odometer(self, miles): """将给定的量添加到里程表读数。""" self.odometer_reading += miles ❷ class ElectricCar(Car): """表示汽车的各个方面，特别是电动汽车。""" ❸ def __init__(self, make, model, year): """初始化父类的属性。""" ❹ super().__init__(make, model, year) ❺ my_leaf = ElectricCar('nissan', 'leaf', 2024) print(my_leaf.get_descriptive_name()) ```py    We start with `Car` ❶. When you create a child class, the parent class must be part of the current file and must appear before the child class in the file. We then define the child class, `ElectricCar` ❷. The name of the parent class must be included in parentheses in the definition of a child class. The `__init__()` method takes in the information required to make a `Car` instance ❸.    The `super()` function ❹ is a special function that allows you to call a method from the parent class. This line tells Python to call the `__init__()` method from `Car`, which gives an `ElectricCar` instance all the attributes defined in that method. The name *super* comes from a convention of calling the parent class a *superclass* and the child class a *subclass*.    We test whether inheritance is working properly by trying to create an electric car with the same kind of information we’d provide when making a regular car. We make an instance of the `ElectricCar` class and assign it to `my_leaf` ❺. This line calls the `__init__()` method defined in `ElectricCar`, which in turn tells Python to call the `__init__()` method defined in the parent class `Car`. We provide the arguments `'nissan'`, `'leaf'`, and `2024`.    Aside from `__init__()`, there are no attributes or methods yet that are particular to an electric car. At this point we’re just making sure the electric car has the appropriate `Car` behaviors:    ``` 2024 Nissan Leaf ```py    The `ElectricCar` instance works just like an instance of `Car`, so now we can begin defining attributes and methods specific to electric cars.    ### Defining Attributes and Methods for the Child Class    Once you have a child class that inherits from a parent class, you can add any new attributes and methods necessary to differentiate the child class from the parent class.    Let’s add an attribute that’s specific to electric cars (a battery, for example) and a method to report on this attribute. We’ll store the battery size and write a method that prints a description of the battery:    ``` class Car: *--snip--* class ElectricCar(Car): """表示汽车的各个方面，特别是电动汽车。""" def __init__(self, make, model, year): """ 初始化父类的属性。 然后初始化电动汽车特有的属性。 """ super().__init__(make, model, year) ❶ self.battery_size = 40 ❷ def describe_battery(self): """打印一条描述电池大小的语句。""" print(f"This car has a {self.battery_size}-kWh battery.") my_leaf = ElectricCar('nissan', 'leaf', 2024) print(my_leaf.get_descriptive_name()) my_leaf.describe_battery() ```py    We add a new attribute `self.battery_size` and set its initial value to `40` ❶. This attribute will be associated with all instances created from the `ElectricCar` class but won’t be associated with any instances of `Car`. We also add a method called `describe_battery()` that prints information about the battery ❷. When we call this method, we get a description that is clearly specific to an electric car:    ``` 2024 Nissan Leaf This car has a 40-kWh battery. ```py    There’s no limit to how much you can specialize the `ElectricCar` class. You can add as many attributes and methods as you need to model an electric car to whatever degree of accuracy you need. An attribute or method that could belong to any car, rather than one that’s specific to an electric car, should be added to the `Car` class instead of the `ElectricCar` class. Then anyone who uses the `Car` class will have that functionality available as well, and the `ElectricCar` class will only contain code for the information and behavior specific to electric vehicles.    ### Overriding Methods from the Parent Class    You can override any method from the parent class that doesn’t fit what you’re trying to model with the child class. To do this, you define a method in the child class with the same name as the method you want to override in the parent class. Python will disregard the parent class method and only pay attention to the method you define in the child class.    Say the class `Car` had a method called `fill_gas_tank()`. This method is meaningless for an all-electric vehicle, so you might want to override this method. Here’s one way to do that:    ``` class ElectricCar(Car): *--snip--* def fill_gas_tank(self): """电动汽车没有油箱。""" print("This car doesn't have a gas tank!") ```py    Now if someone tries to call `fill_gas_tank()` with an electric car, Python will ignore the method `fill_gas_tank()` in `Car` and run this code instead. When you use inheritance, you can make your child classes retain what you need and override anything you don’t need from the parent class.    ### Instances as Attributes    When modeling something from the real world in code, you may find that you’re adding more and more detail to a class. You’ll find that you have a growing list of attributes and methods and that your files are becoming lengthy. In these situations, you might recognize that part of one class can be written as a separate class. You can break your large class into smaller classes that work together; this approach is called *composition*.    For example, if we continue adding detail to the `ElectricCar` class, we might notice that we’re adding many attributes and methods specific to the car’s battery. When we see this happening, we can stop and move those attributes and methods to a separate class called `Battery`. Then we can use a `Battery` instance as an attribute in the `ElectricCar` class:    ``` class Car: *--snip--* class Battery: """一个简单的电动汽车电池模型。""" ❶ def __init__(self, battery_size=40): """初始化电池的属性。""" self.battery_size = battery_size ❷ def describe_battery(self): """打印一条描述电池大小的语句。""" print(f"This car has a {self.battery_size}-kWh battery.") class ElectricCar(Car): """表示汽车的各个方面，特别是电动汽车。""" def __init__(self, make, model, year): """ 初始化父类的属性。 然后初始化电动汽车特有的属性。 """ super().__init__(make, model, year) ❸ self.battery = Battery() my_leaf = ElectricCar('nissan', 'leaf', 2024) print(my_leaf.get_descriptive_name()) my_leaf.battery.describe_battery() ```py    We define a new class called `Battery` that doesn’t inherit from any other class. The `__init__()` method ❶ has one parameter, `battery_size`, in addition to `self`. This is an optional parameter that sets the battery’s size to 40 if no value is provided. The method `describe_battery()` has been moved to this class as well ❷.    In the `ElectricCar` class, we now add an attribute called `self.battery` ❸. This line tells Python to create a new instance of `Battery` (with a default size of 40, because we’re not specifying a value) and assign that instance to the attribute `self.battery`. This will happen every time the `__init__()` method is called; any `ElectricCar` instance will now have a `Battery` instance created automatically.    We create an electric car and assign it to the variable `my_leaf`. When we want to describe the battery, we need to work through the car’s `battery` attribute:    ``` my_leaf.battery.describe_battery() ```py    This line tells Python to look at the instance `my_leaf`, find its `battery` attribute, and call the method `describe_battery()` that’s associated with the `Battery` instance assigned to the attribute.    The output is identical to what we saw previously:    ``` 2024 Nissan Leaf This car has a 40-kWh battery. ```py    This looks like a lot of extra work, but now we can describe the battery in as much detail as we want without cluttering the `ElectricCar` class. Let’s add another method to `Battery` that reports the range of the car based on the battery size:    ``` class Car: *--snip--* class Battery: *--snip--* def get_range(self): """打印一条关于此电池提供的续航里程的语句。""" if self.battery_size == 40: range = 150 elif self.battery_size == 65: range = 225 print(f"This car can go about {range} miles on a full charge.") class ElectricCar(Car): *--snip--* my_leaf = ElectricCar('nissan', 'leaf', 2024) print(my_leaf.get_descriptive_name()) my_leaf.battery.describe_battery() ❶ my_leaf.battery.get_range() ```py    The new method `get_range()` performs some simple analysis. If the battery’s capacity is 40 kWh, `get_range()` sets the range to 150 miles, and if the capacity is 65 kWh, it sets the range to 225 miles. It then reports this value. When we want to use this method, we again have to call it through the car’s `battery` attribute ❶.    The output tells us the range of the car based on its battery size:    ``` 2024 Nissan Leaf This car has a 40-kWh battery. This car can go about 150 miles on a full charge. ```py    ### Modeling Real-World Objects    As you begin to model more complicated things like electric cars, you’ll wrestle with interesting questions. Is the range of an electric car a property of the battery or of the car? If we’re only describing one car, it’s probably fine to maintain the association of the method `get_range()` with the `Battery` class. But if we’re describing a manufacturer’s entire line of cars, we probably want to move `get_range()` to the `ElectricCar` class. The `get_range()` method would still check the battery size before determining the range, but it would report a range specific to the kind of car it’s associated with. Alternatively, we could maintain the association of the `get_range()` method with the battery but pass it a parameter such as `car_model`. The `get_range()` method would then report a range based on the battery size and car model.    This brings you to an interesting point in your growth as a programmer. When you wrestle with questions like these, you’re thinking at a higher logical level rather than a syntax-focused level. You’re thinking not about Python, but about how to represent the real world in code. When you reach this point, you’ll realize there are often no right or wrong approaches to modeling real-world situations. Some approaches are more efficient than others, but it takes practice to find the most efficient representations. If your code is working as you want it to, you’re doing well! Don’t be discouraged if you find you’re ripping apart your classes and rewriting them several times using different approaches. In the quest to write accurate, efficient code, everyone goes through this process.    ## Importing Classes    As you add more functionality to your classes, your files can get long, even when you use inheritance and composition properly. In keeping with the overall philosophy of Python, you’ll want to keep your files as uncluttered as possible. To help, Python lets you store classes in modules and then import the classes you need into your main program.    ### Importing a Single Class    Let’s create a module containing just the `Car` class. This brings up a subtle naming issue: we already have a file named *car.py* in this chapter, but this module should be named *car.py* because it contains code representing a car. We’ll resolve this naming issue by storing the `Car` class in a module named *car.py*, replacing the *car.py* file we were previously using. From now on, any program that uses this module will need a more specific filename, such as *my_car.py*. Here’s *car.py* with just the code from the class `Car`:    **car.py**    ``` ❶ """一个可用于表示汽车的类。""" class Car: """一个简单的表示汽车的尝试。""" def __init__(self, make, model, year): """初始化属性以描述汽车。""" self.make = make self.model = model self.year = year self.odometer_reading = 0 def get_descriptive_name(self): """返回一个格式整洁的描述性名称。""" long_name = f"{self.year} {self.make} {self.model}" return long_name.title() def read_odometer(self): """打印一条显示汽车里程的语句。""" print(f"This car has {self.odometer_reading} miles on it.") def update_odometer(self, mileage): """ 将里程表读数设置为给定值。 如果尝试倒转里程表，则拒绝更改。 """ if mileage >= self.odometer_reading: self.odometer_reading = mileage else: print("You can't roll back an odometer!") def increment_odometer(self, miles): """将给定的量添加到里程表读数。""" self.odometer_reading += miles ```py    We include a module-level docstring that briefly describes the contents of this module ❶. You should write a docstring for each module you create.    Now we make a separate file called *my_car.py*. This file will import the `Car` class and then create an instance from that class:    **my_car.py**    ``` ❶ from car import Car my_new_car = Car('audi', 'a4', 2024) print(my_new_car.get_descriptive_name()) my_new_car.odometer_reading = 23 my_new_car.read_odometer() ```py    The `import` statement ❶ tells Python to open the `car` module and import the class `Car`. Now we can use the `Car` class as if it were defined in this file. The output is the same as we saw earlier:    ``` 2024 Audi A4 This car has 23 miles on it. ```py    Importing classes is an effective way to program. Picture how long this program file would be if the entire `Car` class were included. When you instead move the class to a module and import the module, you still get all the same functionality, but you keep your main program file clean and easy to read. You also store most of the logic in separate files; once your classes work as you want them to, you can leave those files alone and focus on the higher-level logic of your main program.    ### Storing Multiple Classes in a Module    You can store as many classes as you need in a single module, although each class in a module should be related somehow. The classes `Battery` and `ElectricCar` both help represent cars, so let’s add them to the module *car.py*.    **car.py**    ``` """一组用于表示汽油和电动汽车的类。""" class Car: *--snip--* class Battery: """一个简单的电动汽车电池模型。""" def __init__(self, battery_size=40): """初始化电池的属性。""" self.battery_size = battery_size def describe_battery(self): """打印一条描述电池大小的语句。""" print(f"This car has a {self.battery_size}-kWh battery.") def get_range(self): """打印一条关于此电池提供的续航里程的语句。""" if self.battery_size == 40: range = 150 elif self.battery_size == 65: range = 225 print(f"This car can go about {range} miles on a full charge.") class ElectricCar(Car): """模拟汽车的各个方面，特别是电动汽车。""" def __init__(self, make, model, year): """ 初始化父类的属性。 然后初始化电动汽车特有的属性。 """ super().__init__(make, model, year) self.battery = Battery() ```py    Now we can make a new file called *my_electric_car.py*, import the `ElectricCar` class, and make an electric car:    **my_electric_car.py**    ``` from car import ElectricCar my_leaf = ElectricCar('nissan', 'leaf', 2024) print(my_leaf.get_descriptive_name()) my_leaf.battery.describe_battery() my_leaf.battery.get_range() ```py    This has the same output we saw earlier, even though most of the logic is hidden away in a module:    ``` 2024 Nissan Leaf This car has a 40-kWh battery. This car can go about 150 miles on a full charge. ```py    ### Importing Multiple Classes from a Module    You can import as many classes as you need into a program file. If we want to make a regular car and an electric car in the same file, we need to import both classes, `Car` and `ElectricCar`:    **my_cars.py**    ``` ❶ from car import Car, ElectricCar ❷ my_mustang = Car('ford', 'mustang', 2024) print(my_mustang.get_descriptive_name()) ❸ my_leaf = ElectricCar('nissan', 'leaf', 2024) print(my_leaf.get_descriptive_name()) ```py    You import multiple classes from a module by separating each class with a comma ❶. Once you’ve imported the necessary classes, you’re free to make as many instances of each class as you need.    In this example we make a gas-powered Ford Mustang ❷ and then an electric Nissan Leaf ❸:    ``` 2024 Ford Mustang 2024 Nissan Leaf ```py    ### Importing an Entire Module    You can also import an entire module and then access the classes you need using dot notation. This approach is simple and results in code that is easy to read. Because every call that creates an instance of a class includes the module name, you won’t have naming conflicts with any names used in the current file.    Here’s what it looks like to import the entire `car` module and then create a regular car and an electric car:    **my_cars.py**    ``` ❶ import car ❷ my_mustang = car.Car('ford', 'mustang', 2024) print(my_mustang.get_descriptive_name()) ❸ my_leaf = car.ElectricCar('nissan', 'leaf', 2024) print(my_leaf.get_descriptive_name()) ```py    First we import the entire `car` module ❶. We then access the classes we need through the `module_name`.`ClassName` syntax. We again create a Ford Mustang ❷, and a Nissan Leaf ❸.    ### Importing All Classes from a Module    You can import every class from a module using the following syntax:    ``` from `module_name` import * ```py    This method is not recommended for two reasons. First, it’s helpful to be able to read the `import` statements at the top of a file and get a clear sense of which classes a program uses. With this approach it’s unclear which classes you’re using from the module. This approach can also lead to confusion with names in the file. If you accidentally import a class with the same name as something else in your program file, you can create errors that are hard to diagnose. I show this here because even though it’s not a recommended approach, you’re likely to see it in other people’s code at some point.    If you need to import many classes from a module, you’re better off importing the entire module and using the `module_name.ClassName` syntax. You won’t see all the classes used at the top of the file, but you’ll see clearly where the module is used in the program. You’ll also avoid the potential naming conflicts that can arise when you import every class in a module.    ### Importing a Module into a Module    Sometimes you’ll want to spread out your classes over several modules to keep any one file from growing too large and avoid storing unrelated classes in the same module. When you store your classes in several modules, you may find that a class in one module depends on a class in another module. When this happens, you can import the required class into the first module.    For example, let’s store the `Car` class in one module and the `ElectricCar` and `Battery` classes in a separate module. We’ll make a new module called *electric_car.py*—replacing the *electric_car.py* file we created earlier—and copy just the `Battery` and `ElectricCar` classes into this file:    **electric_car.py**    ``` """一组可用于表示电动汽车的类。""" from car import Car class Battery: *--snip--* class ElectricCar(Car): *--snip--* ```py    The class `ElectricCar` needs access to its parent class `Car`, so we import `Car` directly into the module. If we forget this line, Python will raise an error when we try to import the `electric_car` module. We also need to update the `Car` module so it contains only the `Car` class:    **car.py**    ``` """一个可用于表示汽车的类。""" class Car: *--snip--* ```py    Now we can import from each module separately and create whatever kind of car we need:    **my_cars.py**    ``` from car import Car from electric_car import ElectricCar my_mustang = Car('ford', 'mustang', 2024) print(my_mustang.get_descriptive_name()) my_leaf = ElectricCar('nissan', 'leaf', 2024) print(my_leaf.get_descriptive_name()) ```py    We import `Car` from its module, and `ElectricCar` from its module. We then create one regular car and one electric car. Both cars are created correctly:    ``` 2024 Ford Mustang 2024 Nissan Leaf ```py    ### Using Aliases    As you saw in Chapter 8, aliases can be quite helpful when using modules to organize your projects’ code. You can use aliases when importing classes as well.    As an example, consider a program where you want to make a bunch of electric cars. It might get tedious to type (and read) `ElectricCar` over and over again. You can give `ElectricCar` an alias in the import statement:    ``` from electric_car import ElectricCar as EC ```py    Now you can use this alias whenever you want to make an electric car:    ``` my_leaf = EC('nissan', 'leaf', 2024) ```py    You can also give a module an alias. Here’s how to import the entire `electric_car` module using an alias:    ``` import electric_car as ec ```py    Now you can use this module alias with the full class name:    ``` my_leaf = ec.ElectricCar('nissan', 'leaf', 2024) ```py    ### Finding Your Own Workflow    As you can see, Python gives you many options for how to structure code in a large project. It’s important to know all these possibilities so you can determine the best ways to organize your projects as well as understand other people’s projects.    When you’re starting out, keep your code structure simple. Try doing everything in one file and moving your classes to separate modules once everything is working. If you like how modules and files interact, try storing your classes in modules when you start a project. Find an approach that lets you write code that works, and go from there.    ## The Python Standard Library    The *Python standard library* is a set of modules included with every Python installation. Now that you have a basic understanding of how functions and classes work, you can start to use modules like these that other programmers have written. You can use any function or class in the standard library by including a simple `import` statement at the top of your file. Let’s look at one module, `random`, which can be useful in modeling many real-world situations.    One interesting function from the random module is `randint()`. This function takes two integer arguments and returns a randomly selected integer between (and including) those numbers.    Here’s how to generate a random number between 1 and 6:    ``` >>> **from random import randint** >>> **randint(1, 6)** 3 ```py    Another useful function is `choice()`. This function takes in a list or tuple and returns a randomly chosen element:    ``` >>> **from random import choice** >>> **players = ['charles', 'martina', 'michael', 'florence', 'eli']** >>> **first_up = choice(players)** >>> **first_up** 'florence' ```py    The `random` module shouldn’t be used when building security-related applications, but it works well for many fun and interesting projects.    ## Styling Classes    A few styling issues related to classes are worth clarifying, especially as your programs become more complicated.    Class names should be written in *CamelCase*. To do this, capitalize the first letter of each word in the name, and don’t use underscores. Instance and module names should be written in lowercase, with underscores between words.    Every class should have a docstring immediately following the class definition. The docstring should be a brief description of what the class does, and you should follow the same formatting conventions you used for writing docstrings in functions. Each module should also have a docstring describing what the classes in a module can be used for.    You can use blank lines to organize code, but don’t use them excessively. Within a class you can use one blank line between methods, and within a module you can use two blank lines to separate classes.    If you need to import a module from the standard library and a module that you wrote, place the import statement for the standard library module first. Then add a blank line and the import statement for the module you wrote. In programs with multiple import statements, this convention makes it easier to see where the different modules used in the program come from.    ## Summary    In this chapter, you learned how to write your own classes. You learned how to store information in a class using attributes and how to write methods that give your classes the behavior they need. You learned to write `__init__()` methods that create instances from your classes with exactly the attributes you want. You saw how to modify the attributes of an instance directly and through methods. You learned that inheritance can simplify the creation of classes that are related to each other, and you learned to use instances of one class as attributes in another class to keep each class simple.    You saw how storing classes in modules and importing classes you need into the files where they’ll be used can keep your projects organized. You started learning about the Python standard library, and you saw an example based on the `random` module. Finally, you learned to style your classes using Python conventions.    In Chapter 10, you’ll learn to work with files so you can save the work you’ve done in a program and the work you’ve allowed users to do. You’ll also learn about *exceptions*, a special Python class designed to help you respond to errors when they arise.    # 10 Files and Exceptions  ![](Images/chapterart.png)  Now that you’ve mastered the basic skills you need to write organized programs that are easy to use, it’s time to think about making your programs even more relevant and usable. In this chapter, you’ll learn to work with files so your programs can quickly analyze lots of data.    You’ll learn to handle errors so your programs don’t crash when they encounter unexpected situations. You’ll learn about *exceptions*, which are special objects Python creates to manage errors that arise while a program is running. You’ll also learn about the `json` module, which allows you to save user data so it isn’t lost when your program stops running.    Learning to work with files and save data will make your programs easier for people to use. Users will be able to choose what data to enter and when to enter it. People will be able to run your program, do some work, and then close the program and pick up where they left off. Learning to handle exceptions will help you deal with situations in which files don’t exist and deal with other problems that can cause your programs to crash. This will make your programs more robust when they encounter bad data, whether it comes from innocent mistakes or from malicious attempts to break your programs. With the skills you’ll learn in this chapter, you’ll make your programs more applicable, usable, and stable.    ## Reading from a File    An incredible amount of data is available in text files. Text files can contain weather data, traffic data, socioeconomic data, literary works, and more. Reading from a file is particularly useful in data analysis applications, but it’s also applicable to any situation in which you want to analyze or modify information stored in a file. For example, you can write a program that reads in the contents of a text file and rewrites the file with formatting that allows a browser to display it.    When you want to work with the information in a text file, the first step is to read the file into memory. You can then work through all of the file’s contents at once or work through the contents line by line.    ### Reading the Contents of a File    To begin, we need a file with a few lines of text in it. Let’s start with a file that contains *pi* to 30 decimal places, with 10 decimal places per line:    **pi_digits.txt**    ``` 3.1415926535 8979323846 2643383279 ```py    To try the following examples yourself, you can enter these lines in an editor and save the file as *pi_digits.txt*, or you can download the file from the book’s resources through [https://ehmatthes.github.io/pcc_3e](https://ehmatthes.github.io/pcc_3e). Save the file in the same directory where you’ll store this chapter’s programs.    Here’s a program that opens this file, reads it, and prints the contents of the file to the screen:    **file_reader.py**    ``` from pathlib import Path ❶ path = Path('pi_digits.txt') ❷ contents = path.read_text() print(contents) ```py    To work with the contents of a file, we need to tell Python the path to the file. A *path* is the exact location of a file or folder on a system. Python provides a module called `pathlib` that makes it easier to work with files and directories, no matter which operating system you or your program’s users are working with. A module that provides specific functionality like this is often called a *library*, hence the name `pathlib`.    We start by importing the `Path` class from `pathlib`. There’s a lot you can do with a `Path` object that points to a file. For example, you can check that the file exists before working with it, read the file’s contents, or write new data to the file. Here, we build a `Path` object representing the file *pi_digits.txt*, which we assign to the variable `path` ❶. Since this file is saved in the same directory as the *.py* file we’re writing, the filename is all that `Path` needs to access the file.    Once we have a `Path` object representing *pi_digits.txt*, we use the `read_text()` method to read the entire contents of the file ❷. The contents of the file are returned as a single string, which we assign to the variable `contents`. When we print the value of `contents`, we see the entire contents of the text file:    ``` 3.1415926535 8979323846 2643383279 ```py    The only difference between this output and the original file is the extra blank line at the end of the output. The blank line appears because `read_text()` returns an empty string when it reaches the end of the file; this empty string shows up as a blank line.    We can remove the extra blank line by using `rstrip()` on the `contents` string:    ``` from pathlib import Path path = Path('pi_digits.txt') contents = path.read_text() contents = contents.rstrip() print(contents) ```py    Recall from Chapter 2 that Python’s `rstrip()` method removes, or strips, any whitespace characters from the right side of a string. Now the output matches the contents of the original file exactly:    ``` 3.1415926535 8979323846 2643383279 ```py    We can strip the trailing newline character when we read the contents of the file, by applying the `rstrip()` method immediately after calling `read_text()`:    ``` contents = path.read_text().rstrip() ```py    This line tells Python to call the `read_text()` method on the file we’re working with. Then it applies the `rstrip()` method to the string that `read_text()` returns. The cleaned-up string is then assigned to the variable `contents`. This approach is called *method chaining*, and you’ll see it used often in programming.    ### Relative and Absolute File Paths    When you pass a simple filename like *pi_digits.txt* to `Path`, Python looks in the directory where the file that’s currently being executed (that is, your *.py* program file) is stored.    Sometimes, depending on how you organize your work, the file you want to open won’t be in the same directory as your program file. For example, you might store your program files in a folder called *python_work*; inside *python_work*, you might have another folder called *text_files* to distinguish your program files from the text files they’re manipulating. Even though *text_files* is in *python_work*, just passing `Path` the name of a file in *text_files* won’t work, because Python will only look in *python_work* and stop there; it won’t go on and look in *text_files.* To get Python to open files from a directory other than the one where your program file is stored, you need to provide the correct path.    There are two main ways to specify paths in programming. A *relative file path* tells Python to look for a given location relative to the directory where the currently running program file is stored. Since *text_files* is inside *python_work*, we need to build a path that starts with the directory *text_files*, and ends with the filename. Here’s how to build this path:    ``` path = Path('text_files/`filename`.txt') ```py    You can also tell Python exactly where the file is on your computer, regardless of where the program that’s being executed is stored. This is called an *absolute file path*. You can use an absolute path if a relative path doesn’t work. For instance, if you’ve put *text_files* in some folder other than *python_work*, then just passing `Path` the path `'text_files/``filename``.txt'` won’t work because Python will only look for that location inside *python_work*. You’ll need to write out an absolute path to clarify where you want Python to look.    Absolute paths are usually longer than relative paths, because they start at your system’s root folder:    ``` path = Path('/home/eric/data_files/text_files/`filename`.txt') ```py    Using absolute paths, you can read files from any location on your system. For now it’s easiest to store files in the same directory as your program files, or in a folder such as *text_files* within the directory that stores your program files.    ### Accessing a File’s Lines    When you’re working with a file, you’ll often want to examine each line of the file. You might be looking for certain information in the file, or you might want to modify the text in the file in some way. For example, you might want to read through a file of weather data and work with any line that includes the word *sunny* in the description of that day’s weather. In a news report, you might look for any line with the tag `<headline>` and rewrite that line with a specific kind of formatting.    You can use the `splitlines()` method to turn a long string into a set of lines, and then use a `for` loop to examine each line from a file, one at a time:    **file_reader.py**    ``` from pathlib import Path path = Path('pi_digits.txt') ❶ contents = path.read_text() ❷ lines = contents.splitlines() for line in lines: print(line) ```py    We start out by reading the entire contents of the file, as we did earlier ❶. If you’re planning to work with the individual lines in a file, you don’t need to strip any whitespace when reading the file. The `splitlines()` method returns a list of all lines in the file, and we assign this list to the variable `lines` ❷. We then loop over these lines and print each one:    ``` 3.1415926535 8979323846 2643383279 ```py    Since we haven’t modified any of the lines, the output matches the original text file exactly.    ### Working with a File’s Contents    After you’ve read the contents of a file into memory, you can do whatever you want with that data, so let’s briefly explore the digits of *pi*. First, we’ll attempt to build a single string containing all the digits in the file with no whitespace in it:    **pi_string.py**    ``` from pathlib import Path path = Path('pi_digits.txt') contents = path.read_text() lines = contents.splitlines() pi_string = '' ❶ for line in lines: pi_string += line print(pi_string) print(len(pi_string)) ```py    We start by reading the file and storing each line of digits in a list, just as we did in the previous example. We then create a variable, `pi_string`, to hold the digits of *pi*. We write a loop that adds each line of digits to `pi_string` ❶. We print this string, and also show how long the string is:    ``` 3.1415926535 8979323846 2643383279 36 ```py    The variable `pi_string` contains the whitespace that was on the left side of the digits in each line, but we can get rid of that by using `lstrip()` on each line:    ``` *--snip--* for line in lines: pi_string += line.lstrip() print(pi_string) print(len(pi_string)) ```py    Now we have a string containing *pi* to 30 decimal places. The string is 32 characters long because it also includes the leading `3` and a decimal point:    ``` 3.141592653589793238462643383279 32 ```py    ### Large Files: One Million Digits    So far, we’ve focused on analyzing a text file that contains only three lines, but the code in these examples would work just as well on much larger files. If we start with a text file that contains *pi* to 1,000,000 decimal places, instead of just 30, we can create a single string containing all these digits. We don’t need to change our program at all, except to pass it a different file. We’ll also print just the first 50 decimal places, so we don’t have to watch a million digits scroll by in the terminal:    **pi_string.py**    ``` from pathlib import Path path = Path('pi_million_digits.txt') contents = path.read_text() lines = contents.splitlines() pi_string = '' for line in lines: pi_string += line.lstrip() print(f"{pi_string[:52]}...") print(len(pi_string)) ```py    The output shows that we do indeed have a string containing *pi* to 1,000,000 decimal places:    ``` 3.14159265358979323846264338327950288419716939937510... 1000002 ```py    Python has no inherent limit to how much data you can work with; you can work with as much data as your system’s memory can handle.    ### Is Your Birthday Contained in Pi?    I’ve always been curious to know if my birthday appears anywhere in the digits of *pi*. Let’s use the program we just wrote to find out if someone’s birthday appears anywhere in the first million digits of *pi*. We can do this by expressing each birthday as a string of digits and seeing if that string appears anywhere in `pi_string`:    **pi_birthday.py**    ``` *--snip--* for line in lines: pi_string += line.strip() birthday = input("Enter your birthday, in the form mmddyy: ") if birthday in pi_string: print("Your birthday appears in the first million digits of pi!") else: print("Your birthday does not appear in the first million digits of pi.") ```py    We first prompt for the user’s birthday, and then check if that string is in `pi_string`. Let’s try it:    ``` Enter your birthdate, in the form mmddyy: **120372** Your birthday appears in the first million digits of pi! ```py    My birthday does appear in the digits of *pi*! Once you’ve read from a file, you can analyze its contents in just about any way you can imagine.    ## Writing to a File    One of the simplest ways to save data is to write it to a file. When you write text to a file, the output will still be available after you close the terminal containing your program’s output. You can examine output after a program finishes running, and you can share the output files with others as well. You can also write programs that read the text back into memory and work with it again later.    ### Writing a Single Line    Once you have a path defined, you can write to a file using the `write_text()` method. To see how this works, let’s write a simple message and store it in a file instead of printing it to the screen:    **write_message.py**    ``` from pathlib import Path path = Path('programming.txt') path.write_text("I love programming.") ```py    The `write_text()` method takes a single argument: the string that you want to write to the file. This program has no terminal output, but if you open the file *programming.txt*, you’ll see one line:    **programming.txt**    ``` I love programming. ```py    This file behaves like any other file on your computer. You can open it, write new text in it, copy from it, paste to it, and so forth.    ### Writing Multiple Lines    The `write_text()` method does a few things behind the scenes. If the file that `path` points to doesn’t exist, it creates that file. Also, after writing the string to the file, it makes sure the file is closed properly. Files that aren’t closed properly can lead to missing or corrupted data.    To write more than one line to a file, you need to build a string containing the entire contents of the file, and then call `write_text()` with that string. Let’s write several lines to the *programming.txt* file:    ``` from pathlib import Path contents = "I love programming.\n" contents += "I love creating new games.\n" contents += "I also love working with data.\n" path = Path('programming.txt') path.write_text(contents) ```py    We define a variable called `contents` that will hold the entire contents of the file. On the next line, we use the `+=` operator to add to this string. You can do this as many times as you need, to build strings of any length. In this case we include newline characters at the end of each line, to make sure each statement appears on its own line.    If you run this and then open *programming.txt*, you’ll see each of these lines in the text file:    ``` I love programming. I love creating new games. I also love working with data. ```py    You can also use spaces, tab characters, and blank lines to format your output, just as you’ve been doing with terminal-based output. There’s no limit to the length of your strings, and this is how many computer-generated documents are created.    ## Exceptions    Python uses special objects called *exceptions* to manage errors that arise during a program’s execution. Whenever an error occurs that makes Python unsure of what to do next, it creates an exception object. If you write code that handles the exception, the program will continue running. If you don’t handle the exception, the program will halt and show a *traceback*, which includes a report of the exception that was raised.    Exceptions are handled with `try`-`except` blocks. A *try*-*except* block asks Python to do something, but it also tells Python what to do if an exception is raised. When you use `try`-`except` blocks, your programs will continue running even if things start to go wrong. Instead of tracebacks, which can be confusing for users to read, users will see friendly error messages that you’ve written.    ### Handling the ZeroDivisionError Exception    Let’s look at a simple error that causes Python to raise an exception. You probably know that it’s impossible to divide a number by zero, but let’s ask Python to do it anyway:    **division_calculator.py**    ``` print(5/0) ```py    Python can’t do this, so we get a traceback:    ``` Traceback (most recent call last): File "division_calculator.py", line 1, in <module> print(5/0) ~^~ ❶ ZeroDivisionError: division by zero ```py    The error reported in the traceback, `ZeroDivisionError`, is an exception object ❶. Python creates this kind of object in response to a situation where it can’t do what we ask it to. When this happens, Python stops the program and tells us the kind of exception that was raised. We can use this information to modify our program. We’ll tell Python what to do when this kind of exception occurs; that way, if it happens again, we’ll be prepared.    ### Using try-except Blocks    When you think an error may occur, you can write a `try`-`except` block to handle the exception that might be raised. You tell Python to try running some code, and you tell it what to do if the code results in a particular kind of exception.    Here’s what a `try`-`except` block for handling the `ZeroDivisionError` exception looks like:    ``` try: print(5/0) except ZeroDivisionError: print("You can't divide by zero!") ```py    We put `print(5/0)`, the line that caused the error, inside a `try` block. If the code in a `try` block works, Python skips over the `except` block. If the code in the `try` block causes an error, Python looks for an `except` block whose error matches the one that was raised, and runs the code in that block.    In this example, the code in the `try` block produces a `ZeroDivisionError`, so Python looks for an `except` block telling it how to respond. Python then runs the code in that block, and the user sees a friendly error message instead of a traceback:    ``` You can't divide by zero! ```py    If more code followed the `try`-`except` block, the program would continue running because we told Python how to handle the error. Let’s look at an example where catching an error can allow a program to continue running.    ### Using Exceptions to Prevent Crashes    Handling errors correctly is especially important when the program has more work to do after the error occurs. This happens often in programs that prompt users for input. If the program responds to invalid input appropriately, it can prompt for more valid input instead of crashing.    Let’s create a simple calculator that does only division:    **division_calculator.py**    ``` print("Give me two numbers, and I'll divide them.") print("Enter 'q' to quit.") while True: ❶ first_number = input("\nFirst number: ") if first_number == 'q': break ❷ second_number = input("Second number: ") if second_number == 'q': break ❸ answer = int(first_number) / int(second_number) print(answer) ```py    This program prompts the user to input a `first_number` ❶ and, if the user does not enter `q` to quit, a `second_number` ❷. We then divide these two numbers to get an `answer` ❸. This program does nothing to handle errors, so asking it to divide by zero causes it to crash:    ``` Give me two numbers, and I'll divide them. Enter 'q' to quit. First number: **5** Second number: **0** Traceback (most recent call last): File "division_calculator.py", line 11, in <module> answer = int(first_number) / int(second_number) ~~~~~~~~~~~~~~~~~~^~~~~~~~~~~~~~~~~~~~ ZeroDivisionError: division by zero ```py    It’s bad that the program crashed, but it’s also not a good idea to let users see tracebacks. Nontechnical users will be confused by them, and in a malicious setting, attackers will learn more than you want them to. For example, they’ll know the name of your program file, and they’ll see a part of your code that isn’t working properly. A skilled attacker can sometimes use this information to determine which kind of attacks to use against your code.    ### The else Block    We can make this program more error resistant by wrapping the line that might produce errors in a `try`-`except` block. The error occurs on the line that performs the division, so that’s where we’ll put the `try`-`except` block. This example also includes an `else` block. Any code that depends on the `try` block executing successfully goes in the `else` block:    ``` *--snip--* while True: *--snip--* if second_number == 'q': break ❶ try: answer = int(first_number) / int(second_number) ❷ except ZeroDivisionError: print("You can't divide by 0!") ❸ else: print(answer) ```py    We ask Python to try to complete the division operation in a `try` block ❶, which includes only the code that might cause an error. Any code that depends on the `try` block succeeding is added to the `else` block. In this case, if the division operation is successful, we use the `else` block to print the result ❸.    The `except` block tells Python how to respond when a `ZeroDivisionError` arises ❷. If the `try` block doesn’t succeed because of a division-by-zero error, we print a friendly message telling the user how to avoid this kind of error. The program continues to run, and the user never sees a traceback:    ``` Give me two numbers, and I'll divide them. Enter 'q' to quit. First number: **5** Second number: **0** You can't divide by 0! First number: **5** Second number: **2** 2.5 First number: **q** ```py    The only code that should go in a `try` block is code that might cause an exception to be raised. Sometimes you’ll have additional code that should run only if the `try` block was successful; this code goes in the `else` block. The `except` block tells Python what to do in case a certain exception arises when it tries to run the code in the `try` block.    By anticipating likely sources of errors, you can write robust programs that continue to run even when they encounter invalid data and missing resources. Your code will be resistant to innocent user mistakes and malicious attacks.    ### Handling the FileNotFoundError Exception    One common issue when working with files is handling missing files. The file you’re looking for might be in a different location, the filename might be misspelled, or the file might not exist at all. You can handle all of these situations with a `try`-`except` block.    Let’s try to read a file that doesn’t exist. The following program tries to read in the contents of *Alice in Wonderland*, but I haven’t saved the file *alice.txt* in the same directory as *alice.py*:    **alice.py**    ``` from pathlib import Path path = Path('alice.txt') contents = path.read_text(encoding='utf-8') ```py    Note that we’re using `read_text()` in a slightly different way here than what you saw earlier. The `encoding` argument is needed when your system’s default encoding doesn’t match the encoding of the file that’s being read. This is most likely to happen when reading from a file that wasn’t created on your system.    Python can’t read from a missing file, so it raises an exception:    ``` Traceback (most recent call last): ❶ File "alice.py", line 4, in <module> ❷ contents = path.read_text(encoding='utf-8') ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ File "/.../pathlib.py", line 1056, in read_text with self.open(mode='r', encoding=encoding, errors=errors) as f: ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ File "/.../pathlib.py", line 1042, in open return io.open(self, mode, buffering, encoding, errors, newline) ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ ❸ FileNotFoundError: [Errno 2] No such file or directory: 'alice.txt' ```py    This is a longer traceback than the ones we’ve seen previously, so let’s look at how you can make sense of more complex tracebacks. It’s often best to start at the very end of the traceback. On the last line, we can see that a `FileNotFoundError` exception was raised ❸. This is important because it tells us what kind of exception to use in the `except` block that we’ll write.    Looking back near the beginning of the traceback ❶, we can see that the error occurred at line 4 in the file *alice.py*. The next line shows the line of code that caused the error ❷. The rest of the traceback shows some code from the libraries that are involved in opening and reading from files. You don’t usually need to read through or understand all of these lines in a traceback.    To handle the error that’s being raised, the `try` block will begin with the line that was identified as problematic in the traceback. In our example, this is the line that contains `read_text()`:    ``` from pathlib import Path path = Path('alice.txt') try: contents = path.read_text(encoding='utf-8') ❶ except FileNotFoundError: print(f"Sorry, the file {path} does not exist.") ```py    In this example, the code in the `try` block produces a `FileNotFoundError`, so we write an `except` block that matches that error ❶. Python then runs the code in that block when the file can’t be found, and the result is a friendly error message instead of a traceback:    ``` Sorry, the file alice.txt does not exist. ```py    The program has nothing more to do if the file doesn’t exist, so this is all the output we see. Let’s build on this example and see how exception handling can help when you’re working with more than one file.    ### Analyzing Text    You can analyze text files containing entire books. Many classic works of literature are available as simple text files because they are in the public domain. The texts used in this section come from Project Gutenberg ([https://gutenberg.org](https://gutenberg.org)). Project Gutenberg maintains a collection of literary works that are available in the public domain, and it’s a great resource if you’re interested in working with literary texts in your programming projects.    Let’s pull in the text of *Alice in Wonderland* and try to count the number of words in the text. To do this, we’ll use the string method `split()`, which by default splits a string wherever it finds any whitespace:    ``` from pathlib import Path path = Path('alice.txt') try: contents = path.read_text(encoding='utf-8') except FileNotFoundError: print(f"Sorry, the file {path} does not exist.") else: # Count the approximate number of words in the file: ❶ words = contents.split() ❷ num_words = len(words) print(f"The file {path} has about {num_words} words.") ```py    I moved the file *alice.txt* to the correct directory, so the `try` block will work this time. We take the string `contents`, which now contains the entire text of *Alice in Wonderland* as one long string, and use `split()` to produce a list of all the words in the book ❶. Using `len()` on this list ❷ gives us a good approximation of the number of words in the original text. Lastly, we print a statement that reports how many words were found in the file. This code is placed in the `else` block because it only works if the code in the `try` block was executed successfully.    The output tells us how many words are in *alice.txt*:    ``` The file alice.txt has about 29594 words. ```py    The count is a little high because extra information is provided by the publisher in the text file used here, but it’s a good approximation of the length of *Alice in Wonderland*.    ### Working with Multiple Files    Let’s add more books to analyze, but before we do, let’s move the bulk of this program to a function called `count_words()`. This will make it easier to run the analysis for multiple books:    **word_count.py**    ``` from pathlib import Path def count_words(path): ❶ """计算文件中单词的大概数量。""" try: contents = path.read_text(encoding='utf-8') except FileNotFoundError: print(f"Sorry, the file {path} does not exist.") else: # Count the approximate number of words in the file: words = contents.split() num_words = len(words) print(f"The file {path} has about {num_words} words.") path = Path('alice.txt') count_words(path) ```py    Most of this code is unchanged. It’s only been indented, and moved into the body of `count_words()`. It’s a good habit to keep comments up to date when you’re modifying a program, so the comment has also been changed to a docstring and reworded slightly ❶.    Now we can write a short loop to count the words in any text we want to analyze. We do this by storing the names of the files we want to analyze in a list, and then we call `count_words()` for each file in the list. We’ll try to count the words for *Alice in Wonderland*, *Siddhartha*, *Moby Dick*, and *Little Women*, which are all available in the public domain. I’ve intentionally left *siddhartha.txt* out of the directory containing *word_count.py*, so we can see how well our program handles a missing file:    ``` from pathlib import Path def count_words(filename): *--snip--* filenames = ['alice.txt', 'siddhartha.txt', 'moby_dick.txt', 'little_women.txt'] for filename in filenames: ❶ path = Path(filename) count_words(path) ```py    The names of the files are stored as simple strings. Each string is then converted to a `Path` object ❶, before the call to `count_words()`. The missing *siddhartha.txt* file has no effect on the rest of the program’s execution:    ``` The file alice.txt has about 29594 words. Sorry, the file siddhartha.txt does not exist. The file moby_dick.txt has about 215864 words. The file little_women.txt has about 189142 words. ```py    Using the `try`-`except` block in this example provides two significant advantages. We prevent our users from seeing a traceback, and we let the program continue analyzing the texts it’s able to find. If we don’t catch the `FileNotFoundError` that *siddhartha.txt* raises, the user would see a full traceback, and the program would stop running after trying to analyze *Siddhartha*. It would never analyze *Moby Dick* or *Little Women*.    ### Failing Silently    In the previous example, we informed our users that one of the files was unavailable. But you don’t need to report every exception you catch. Sometimes, you’ll want the program to fail silently when an exception occurs and continue on as if nothing happened. To make a program fail silently, you write a `try` block as usual, but you explicitly tell Python to do nothing in the `except` block. Python has a `pass` statement that tells it to do nothing in a block:    ``` def count_words(path): """计算文件中单词的大概数量。""" try: *--snip--* except FileNotFoundError: pass else: *--snip--* ```py    The only difference between this listing and the previous one is the `pass` statement in the `except` block. Now when a `FileNotFoundError` is raised, the code in the `except` block runs, but nothing happens. No traceback is produced, and there’s no output in response to the error that was raised. Users see the word counts for each file that exists, but they don’t see any indication that a file wasn’t found:    ``` The file alice.txt has about 29594 words. The file moby_dick.txt has about 215864 words. The file little_women.txt has about 189142 words. ```py    The `pass` statement also acts as a placeholder. It’s a reminder that you’re choosing to do nothing at a specific point in your program’s execution and that you might want to do something there later. For example, in this program we might decide to write any missing filenames to a file called *missing_files.txt*. Our users wouldn’t see this file, but we’d be able to read the file and deal with any missing texts.    ### Deciding Which Errors to Report    How do you know when to report an error to your users and when to let your program fail silently? If users know which texts are supposed to be analyzed, they might appreciate a message informing them why some texts were not analyzed. If users expect to see some results but don’t know which books are supposed to be analyzed, they might not need to know that some texts were unavailable. Giving users information they aren’t looking for can decrease the usability of your program. Python’s error-handling structures give you fine-grained control over how much to share with users when things go wrong; it’s up to you to decide how much information to share.    Well-written, properly tested code is not very prone to internal errors, such as syntax or logical errors. But every time your program depends on something external such as user input, the existence of a file, or the availability of a network connection, there is a possibility of an exception being raised. A little experience will help you know where to include exception-handling blocks in your program and how much to report to users about errors that arise.    ## Storing Data    Many of your programs will ask users to input certain kinds of information. You might allow users to store preferences in a game or provide data for a visualization. Whatever the focus of your program is, you’ll store the information users provide in data structures such as lists and dictionaries. When users close a program, you’ll almost always want to save the information they entered. A simple way to do this involves storing your data using the `json` module.    The `json` module allows you to convert simple Python data structures into JSON-formatted strings, and then load the data from that file the next time the program runs. You can also use `json` to share data between different Python programs. Even better, the JSON data format is not specific to Python, so you can share data you store in the JSON format with people who work in many other programming languages. It’s a useful and portable format, and it’s easy to learn.    ### Using json.dumps() and json.loads()    Let’s write a short program that stores a set of numbers and another program that reads these numbers back into memory. The first program will use `json.dumps()` to store the set of numbers, and the second program will use `json.loads()`.    The `json.dumps()` function takes one argument: a piece of data that should be converted to the JSON format. The function returns a string, which we can then write to a data file:    **number_writer.py**    ``` from pathlib import Path import json numbers = [2, 3, 5, 7, 11, 13] ❶ path = Path('numbers.json') ❷ contents = json.dumps(numbers) path.write_text(contents) ```py    We first import the `json` module, and then create a list of numbers to work with. Then we choose a filename in which to store the list of numbers ❶. It’s customary to use the file extension *.json* to indicate that the data in the file is stored in the JSON format. Next, we use the `json.dumps()` ❷ function to generate a string containing the JSON representation of the data we’re working with. Once we have this string, we write it to the file using the same `write_text()` method we used earlier.    This program has no output, but let’s open the file *numbers.json* and look at it. The data is stored in a format that looks just like Python:    ``` [2, 3, 5, 7, 11, 13] ```py    Now we’ll write a separate program that uses `json.loads()` to read the list back into memory:    **number_reader.py**    ``` from pathlib import Path import json ❶ path = Path('numbers.json') ❷ contents = path.read_text() ❸ numbers = json.loads(contents) print(numbers) ```py    We make sure to read from the same file we wrote to ❶. Since the data file is just a text file with specific formatting, we can read it with the `read_text()` method ❷. We then pass the contents of the file to `json.loads()` ❸. This function takes in a JSON-formatted string and returns a Python object (in this case, a list), which we assign to `numbers`. Finally, we print the recovered list of numbers and see that it’s the same list created in *number_writer.py*:    ``` [2, 3, 5, 7, 11, 13] ```py    This is a simple way to share data between two programs.    ### Saving and Reading User-Generated Data    Saving data with `json` is useful when you’re working with user-generated data, because if you don’t store your user’s information somehow, you’ll lose it when the program stops running. Let’s look at an example where we prompt the user for their name the first time they run a program and then remember their name when they run the program again.    Let’s start by storing the user’s name:    **remember_me.py**    ``` from pathlib import Path import json ❶ username = input("What is your name? ") ❷ path = Path('username.json') contents = json.dumps(username) path.write_text(contents) ❸ print(f"We'll remember you when you come back, {username}!") ```py    We first prompt for a username to store ❶. Next, we write the data we just collected to a file called *username.json* ❷. Then we print a message informing the user that we’ve stored their information ❸:    ``` What is your name? **Eric** We'll remember you when you come back, Eric! ```py    Now let’s write a new program that greets a user whose name has already been stored:    **greet_user.py**    ``` from pathlib import Path import json ❶ path = Path('username.json') contents = path.read_text() ❷ username = json.loads(contents) print(f"Welcome back, {username}!") ```py    We read the contents of the data file ❶ and then use `json.loads()` to assign the recovered data to the variable `username` ❷. Since we’ve recovered the username, we can welcome the user back with a personalized greeting:    ``` Welcome back, Eric! ```py    We need to combine these two programs into one file. When someone runs *remember_me.py*, we want to retrieve their username from memory if possible; if not, we’ll prompt for a username and store it in *username.json* for next time. We could write a `try`-`except` block here to respond appropriately if *username.json* doesn’t exist, but instead we’ll use a handy method from the `pathlib` module:    **remember_me.py**    ``` from pathlib import Path import json path = Path('username.json') ❶ if path.exists(): contents = path.read_text() username = json.loads(contents) print(f"Welcome back, {username}!") ❷ else: username = input("What is your name? ") contents = json.dumps(username) path.write_text(contents) print(f"We'll remember you when you come back, {username}!") ```py    There are many helpful methods you can use with `Path` objects. The `exists()` method returns `True` if a file or folder exists and `False` if it doesn’t. Here we use `path.exists()` to find out if a username has already been stored ❶. If *username.json* exists, we load the username and print a personalized greeting to the user.    If the file *username.json* doesn’t exist ❷, we prompt for a username and store the value that the user enters. We also print the familiar message that we’ll remember them when they come back.    Whichever block executes, the result is a username and an appropriate greeting. If this is the first time the program runs, this is the output:    ``` What is your name? **Eric** We'll remember you when you come back, Eric! ```py    Otherwise:    ``` Welcome back, Eric! ```py    This is the output you see if the program was already run at least once. Even though the data in this section is just a single string, the program would work just as well with any data that can be converted to a JSON-formatted string.    ### Refactoring    Often, you’ll come to a point where your code will work, but you’ll recognize that you could improve the code by breaking it up into a series of functions that have specific jobs. This process is called *refactoring*. Refactoring makes your code cleaner, easier to understand, and easier to extend.    We can refactor *remember_me.py* by moving the bulk of its logic into one or more functions. The focus of *remember_me.py* is on greeting the user, so let’s move all of our existing code into a function called `greet_user()`:    **remember_me.py**    ``` from pathlib import Path import json def greet_user(): ❶ """通过姓名问候用户。""" path = Path('username.json') if path.exists(): contents = path.read_text() username = json.loads(contents) print(f"Welcome back, {username}!") else: username = input("What is your name? ") contents = json.dumps(username) path.write_text(contents) print(f"We'll remember you when you come back, {username}!") greet_user() ```py    Because we’re using a function now, we rewrite the comments as a docstring that reflects how the program currently works ❶. This file is a little cleaner, but the function `greet_user()` is doing more than just greeting the user—it’s also retrieving a stored username if one exists and prompting for a new username if one doesn’t.    Let’s refactor `greet_user()` so it’s not doing so many different tasks. We’ll start by moving the code for retrieving a stored username to a separate function:    ``` from pathlib import Path import json def get_stored_username(path): ❶ """如果可用，获取存储的用户名。""" if path.exists(): contents = path.read_text() username = json.loads(contents) return username else: ❷ return None def greet_user(): """通过姓名问候用户。""" path = Path('username.json') username = get_stored_username(path) ❸ if username: print(f"Welcome back, {username}!") else: username = input("What is your name? ") contents = json.dumps(username) path.write_text(contents) print(f"We'll remember you when you come back, {username}!") greet_user() ```py    The new function `get_stored_username()` ❶ has a clear purpose, as stated in the docstring. This function retrieves a stored username and returns the username if it finds one. If the path that’s passed to `get_stored_username()` doesn’t exist, the function returns `None` ❷. This is good practice: a function should either return the value you’re expecting, or it should return `None`. This allows us to perform a simple test with the return value of the function. We print a welcome back message to the user if the attempt to retrieve a username is successful ❸, and if it isn’t, we prompt for a new username.    We should factor one more block of code out of `greet_user()`. If the username doesn’t exist, we should move the code that prompts for a new username to a function dedicated to that purpose:    ``` from pathlib import Path import json def get_stored_username(path): """如果可用，获取存储的用户名。""" *--snip--* def get_new_username(path): """提示输入新用户名。""" username = input("What is your name? ") contents = json.dumps(username) path.write_text(contents) return username def greet_user(): """通过姓名问候用户。""" path = Path('username.json') ❶ username = get_stored_username(path) if username: print(f"Welcome back, {username}!") else: ❷ username = get_new_username(path) print(f"We'll remember you when you come back, {username}!") greet_user() ```py    Each function in this final version of *remember_me.py* has a single, clear purpose. We call `greet_user()`, and that function prints an appropriate message: it either welcomes back an existing user or greets a new user. It does this by calling `get_stored_username()` ❶, which is responsible only for retrieving a stored username if one exists. Finally, if necessary, `greet_user()` calls `get_new_username()`❷, which is responsible only for getting a new username and storing it. This compartmentalization of work is an essential part of writing clear code that will be easy to maintain and extend.    ## Summary    In this chapter, you learned how to work with files. You learned to read the entire contents of a file, and then work through the contents one line at a time if you need to. You learned to write as much text as you want to a file. You also read about exceptions and how to handle the exceptions you’re likely to see in your programs. Finally, you learned how to store Python data structures so you can save information your users provide, preventing them from having to start over each time they run a program.    In Chapter 11, you’ll learn efficient ways to test your code. This will help you trust that the code you develop is correct, and it will help you identify bugs that are introduced as you continue to build on the programs you’ve written.    # 11 Testing Your Code  ![](Images/chapterart.png)  When you write a function or a class, you can also write tests for that code. Testing proves that your code works as it’s supposed to in response to all the kinds of input it’s designed to receive. When you write tests, you can be confident that your code will work correctly as more people begin to use your programs. You’ll also be able to test new code as you add it, to make sure your changes don’t break your program’s existing behavior. Every programmer makes mistakes, so every programmer must test their code often, to catch problems before users encounter them.    In this chapter, you’ll learn to test your code using `pytest`. The `pytest` library is a collection of tools that will help you write your first tests quickly and simply, while supporting your tests as they grow in complexity along with your projects. Python doesn’t include `pytest` by default, so you’ll learn to install external libraries. Knowing how to install external libraries will make a wide variety of well-designed code available to you. These libraries will expand the kinds of projects you can work on immensely.    You’ll learn to build a series of tests and check that each set of inputs results in the output you want. You’ll see what a passing test looks like and what a failing test looks like, and you’ll learn how a failing test can help you improve your code. You’ll learn to test functions and classes, and you’ll start to understand how many tests to write for a project.    ## Installing pytest with pip    While Python includes a lot of functionality in the standard library, Python developers also depend heavily on third-party packages. A *third-party package* is a library that’s developed outside the core Python language. Some popular third-party libraries are eventually adopted into the standard library, and end up being included in most Python installations from that point forward. This happens most often with libraries that are unlikely to change much once they’ve had their initial bugs worked out. These kinds of libraries can evolve at the same pace as the overall language.    Many packages, however, are kept out of the standard library so they can be developed on a timeline independent of the language itself. These packages tend to be updated more frequently than they would be if they were tied to Python’s development schedule. This is true of `pytest` and most of the libraries we’ll use in the second half of this book. You shouldn’t blindly trust every third-party package, but you also shouldn’t be put off by the fact that a lot of important functionality is implemented through such packages.    ### Updating pip    Python includes a tool called pip that’s used to install third-party packages. Because pip helps install packages from external resources, it’s updated often to address potential security issues. So, we’ll start by updating pip.    Open a new terminal window and issue the following command:    ``` $ **python -m pip install --upgrade pip** ❶ Requirement already satisfied: pip in /.../python3.11/site-packages (22.0.4) `--snip--` ❷ Successfully installed pip-22.1.2 ```py    The first part of this command, `python -m pip`, tells Python to run the module `pip`. The second part, `install --upgrade`, tells pip to update a package that’s already been installed. The last part, `pip`, specifies which third-party package should be updated. The output shows that my current version of pip, version 22.0.4 ❶, was replaced by the latest version at the time of this writing, 22.1.2 ❷.    You can use this command to update any third-party package installed on your system:    ``` $ **python -m pip install --upgrade** `package_name` ```py    ### Installing pytest    Now that pip is up to date, we can install `pytest`:    ``` $ **python -m pip install --user pytest** Collecting pytest `--snip--` Successfully installed attrs-21.4.0 iniconfig-1.1.1 ...pytest-7.`x`.`x` ```py    We’re still using the core command `pip install`, without the `--upgrade` flag this time. Instead, we’re using the `--user` flag, which tells Python to install this package for the current user only. The output shows that the latest version of `pytest` was successfully installed, along with a number of other packages that `pytest` depends on.    You can use this command to install many third-party packages:    ``` $ **python -m pip install --user** `package_name` ```py    ## Testing a Function    To learn about testing, we need code to test. Here’s a simple function that takes in a first and last name, and returns a neatly formatted full name:    **name_function.py**    ``` def get_formatted_name(first, last): """生成一个格式整洁的全名。""" full_name = f"{first} {last}" return full_name.title() ```py    The function `get_formatted_name()` combines the first and last name with a space in between to complete a full name, and then capitalizes and returns the full name. To check that `get_formatted_name()` works, let’s make a program that uses this function. The program *names.py* lets users enter a first and last name, and see a neatly formatted full name:    **names.py**    ``` from name_function import get_formatted_name print("Enter 'q' at any time to quit.") while True: first = input("\nPlease give me a first name: ") if first == 'q': break last = input("Please give me a last name: ") if last == 'q': break formatted_name = get_formatted_name(first, last) print(f"\tNeatly formatted name: {formatted_name}.") ```py    This program imports `get_formatted_name()` from *name_function.py*. The user can enter a series of first and last names and see the formatted full names that are generated:    ``` Enter 'q' at any time to quit. Please give me a first name: **janis** Please give me a last name: **joplin** Neatly formatted name: Janis Joplin. Please give me a first name: **bob** Please give me a last name: **dylan** Neatly formatted name: Bob Dylan. Please give me a first name: **q** ```py    We can see that the names generated here are correct. But say we want to modify `get_formatted_name()` so it can also handle middle names. As we do so, we want to make sure we don’t break the way the function handles names that have only a first and last name. We could test our code by running *names.py* and entering a name like `Janis Joplin` every time we modify `get_formatted_name()`, but that would become tedious. Fortunately, `pytest` provides an efficient way to automate the testing of a function’s output. If we automate the testing of `get_formatted_name()`, we can always be confident that the function will work when given the kinds of names we’ve written tests for.    ### Unit Tests and Test Cases    There is a wide variety of approaches to testing software. One of the simplest kinds of test is a unit test. A *unit test* verifies that one specific aspect of a function’s behavior is correct. A *test case* is a collection of unit tests that together prove that a function behaves as it’s supposed to, within the full range of situations you expect it to handle.    A good test case considers all the possible kinds of input a function could receive and includes tests to represent each of these situations. A test case with *full coverage* includes a full range of unit tests covering all the possible ways you can use a function. Achieving full coverage on a large project can be daunting. It’s often good enough to write tests for your code’s critical behaviors and then aim for full coverage only if the project starts to see widespread use.    ### A Passing Test    With `pytest`, writing your first unit test is pretty straightforward. We’ll write a single test function. The test function will call the function we’re testing, and we’ll make an assertion about the value that’s returned. If our assertion is correct, the test will pass; if the assertion is incorrect, the test will fail.    Here’s the first test of the function `get_formatted_name()`:    **test_name_function.py**    ``` from name_function import get_formatted_name ❶ def test_first_last_name(): """像“Janis Joplin”这样的名字有效吗？""" ❷ formatted_name = get_formatted_name('janis', 'joplin') ❸ assert formatted_name == 'Janis Joplin' ```py    Before we run the test, let’s take a closer look at this function. The name of a test file is important; it must start with *test_*. When we ask `pytest` to run the tests we’ve written, it will look for any file that begins with *test_*, and run all of the tests it finds in that file.    In the test file, we first import the function that we want to test: `get_formatted_name()`. Then we define a test function: in this case, `test_first_last_name()` ❶. This is a longer function name than we’ve been using, for a good reason. First, test functions need to start with the word *test*, followed by an underscore. Any function that starts with `test_` will be *discovered* by `pytest`, and will be run as part of the testing process.    Also, test names should be longer and more descriptive than a typical function name. You’ll never call the function yourself; `pytest` will find the function and run it for you. Test function names should be long enough that if you see the function name in a test report, you’ll have a good sense of what behavior was being tested.    Next, we call the function we’re testing ❷. Here we call `get_formatted_name()` with the arguments `'janis'` and `'joplin'`, just like we used when we ran *names.py*. We assign the return value of this function to `formatted_name`.    Finally, we make an assertion ❸. An *assertion* is a claim about a condition. Here we’re claiming that the value of `formatted_name` should be `'Janis Joplin'`.    ### Running a Test    If you run the file *test_name_function.py* directly, you won’t get any output because we never called the test function. Instead, we’ll have `pytest` run the test file for us.    To do this, open a terminal window and navigate to the folder that contains the test file. If you’re using VS Code, you can open the folder containing the test file and use the terminal that’s embedded in the editor window. In the terminal window, enter the command `pytest`. Here’s what you should see:    ``` $ **pytest** ========================= test session starts ========================= ❶ platform darwin -- Python 3.`x`.`x`, pytest-7.`x`.`x`, pluggy-1.`x`.`x` ❷ rootdir: /.../python_work/chapter_11 ❸ collected 1 item ❹ test_name_function.py . [100%] ========================== 1 passed in 0.00s ========================== ```py    Let’s try to make sense of this output. First of all, we see some information about the system the test is running on ❶. I’m testing this on a macOS system, so you may see some different output here. Most importantly, we can see which versions of Python, `pytest`, and other packages are being used to run the test.    Next, we see the directory where the test is being run from ❷: in my case, *python_work/chapter_11*. We can see that `pytest` found one test to run ❸, and we can see the test file that’s being run ❹. The single dot after the name of the file tells us that a single test passed, and the `100%` makes it clear that all of the tests have been run. A large project can have hundreds or thousands of tests, and the dots and percentage-complete indicator can be helpful in monitoring the overall progress of the test run.    The last line tells us that one test passed, and it took less than 0.01 seconds to run the test.    This output indicates that the function `get_formatted_name()` will always work for names that have a first and last name, unless we modify the function. When we modify `get_formatted_name()`, we can run this test again. If the test passes, we know the function will still work for names like Janis Joplin.    ### A Failing Test    What does a failing test look like? Let’s modify `get_formatted_name()` so it can handle middle names, but let’s do so in a way that breaks the function for names with just a first and last name, like Janis Joplin.    Here’s a new version of `get_formatted_name()` that requires a middle name argument:    **name_function.py**    ``` def get_formatted_name(first, middle, last): """生成一个格式整洁的全名。""" full_name = f"{first} {middle} {last}" return full_name.title() ```py    This version should work for people with middle names, but when we test it, we see that we’ve broken the function for people with just a first and last name.    This time, running `pytest` gives the following output:    ``` $ **pytest** ========================= test session starts ========================= `--snip--` ❶ test_name_function.py F [100%] ❷ ============================== FAILURES =============================== ❸ ________________________ test_first_last_name
