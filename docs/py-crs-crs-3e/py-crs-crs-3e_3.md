# 第二章：A

安装与故障排除

![](img/chapterart.png)

有许多版本的 Python 可供选择，并且每个操作系统上都有多种安装方式。如果第一章中的方法不起作用，或者你想安装与当前安装版本不同的 Python 版本，本附录中的说明可以帮助你。

## Windows 上的 Python

第一章中的说明向你展示了如何使用[`python.org`](https://python.org)的官方安装程序来安装 Python。如果你在使用安装程序后无法运行 Python，本节中的故障排除说明将帮助你让 Python 正常运行。

### 使用 py 代替 python

如果你运行了最近的 Python 安装程序，然后在终端中输入 `python` 命令，你应该会看到终端会话的 Python 提示符（`>>>`）。如果 Windows 无法识别 `python` 命令，它会打开微软商店，因为它认为 Python 未安装，或者你会看到类似“未找到 Python”的消息。如果微软商店打开了，请关闭它；与其使用微软维护的版本，还是更好地使用来自[`python.org`](https://python.org)的官方 Python 安装程序。

最简单的解决方法是尝试使用`py`命令，而无需对系统进行任何更改。这是一个 Windows 工具，可以找到系统上安装的最新 Python 版本并运行该解释器。如果此命令有效且你希望使用它，只需在本书中看到`python`或`python3`命令时，使用`py`即可。

### 重新运行安装程序

`python`命令无法正常工作最常见的原因是人们在运行安装程序时忘记选择“Add Python to PATH”选项；这是一个容易犯的错误。`PATH`变量是一个系统设置，告诉 Python 去哪里查找常用的程序。在这种情况下，Windows 不知道如何找到 Python 解释器。

这种情况下最简单的解决方法是重新运行安装程序。如果从[`python.org`](https://python.org)可以获取更新的安装程序，请下载新版本并运行，确保勾选**Add Python to PATH**选项。

如果你已经有了最新的安装程序，请再次运行并选择**Modify**选项。你将看到一个可选功能的列表；在这个界面上保持默认选项。然后点击**Next**并勾选**Add Python to Environment Variables**框。最后点击**Install**。安装程序会识别到 Python 已经安装，并将 Python 解释器的位置添加到 `PATH` 变量中。确保关闭所有打开的终端，因为它们仍然会使用旧的 `PATH` 变量。打开一个新的终端窗口并再次输入 `python` 命令；你应该会看到 Python 提示符（`>>>`）。

## macOS 上的 Python

第一章中的安装说明使用了[`python.org`](https://python.org)的官方 Python 安装程序。该官方安装程序多年来一直运行良好，但有一些情况可能会让你偏离正轨。如果有什么不按预期工作，本节将帮助你解决问题。

### 意外安装了 Apple 版本的 Python

如果你运行`python3`命令时，系统中尚未安装 Python，你很可能会看到需要安装*命令行开发者工具*的消息。此时，最好的做法是关闭显示该消息的弹窗，下载 Python 安装程序，来自[`python.org`](https://python.org)，并运行安装程序。

如果你选择在此时安装命令行开发者工具，macOS 会与开发者工具一起安装 Apple 的 Python 版本。唯一的问题是，Apple 的 Python 版本通常会比最新的官方版本稍微滞后。然而，你仍然可以从[`python.org`](https://python.org)下载并运行官方安装程序，`python3`就会指向更新的版本。不要担心开发者工具已安装；其中有一些有用的工具，包括附录 D 中讨论的 Git 版本控制系统。

### 旧版 macOS 上的 Python 2

在旧版 macOS（Monterey 之前的版本，macOS 12）上，默认安装的是过时的 Python 2。在这些系统中，`python`命令会指向过时的系统解释器。如果你使用的是安装了 Python 2 的 macOS 版本，请确保使用`python3`命令，这样你就能始终使用你安装的 Python 版本。

## Linux 上的 Python

Python 默认已经包含在几乎所有 Linux 系统中。然而，如果你系统上的默认版本早于 Python 3.9，建议安装最新版本。你也可以安装最新版本，以便获得最新的功能，比如 Python 改进后的错误信息。以下的说明适用于大多数基于 apt 的系统。

### 使用默认的 Python 安装

如果你想使用`python3`所指向的 Python 版本，请确保安装以下三个附加包：

```py
$ **sudo apt install python3-dev python3-pip python3-venv**
```

这些包包括一些对开发人员有用的工具，以及让你安装第三方包的工具，比如本书项目部分使用的包。

### 安装最新版本的 Python

我们将使用一个叫做`deadsnakes`的包，它可以方便地安装多个版本的 Python。输入以下命令：

```py
$ **sudo add-apt-repository ppa:deadsnakes/ppa**
$ **sudo apt update**
$ **sudo apt install python3.11**
```

这些命令将会把 Python 3.11 安装到你的系统中。

输入以下命令，启动一个运行 Python 3.11 的终端会话：

```py
$ **python3.11**
>>>
```

在本书中，凡是出现`python`命令的地方，请使用`python3.11`代替。你也需要在从终端运行程序时使用这个命令。

你需要再安装两个包，才能充分利用你的 Python 安装：

```py
$ **sudo apt install python3.11-dev python3.11-venv**
```

这些包包括在安装和运行第三方包时所需的模块，例如书籍下半部分项目中使用的那些包。

## 检查你正在使用的 Python 版本

如果你在运行 Python 或安装额外包时遇到任何问题，知道自己正在使用哪个版本的 Python 会很有帮助。你可能安装了多个版本的 Python，且不清楚当前使用的是哪个版本。

在终端中执行以下命令：

```py
$ **python --version**
Python 3.11.0
```

这告诉你当前命令`python`所指向的确切版本。较短的命令`python -V`也会给出相同的输出。

## Python 关键字和内置函数

Python 自带了一套关键字和内置函数。了解这些非常重要，因为在 Python 中命名时，你的名称不能与这些关键字相同，也不应与函数名相同，否则你将覆盖这些函数。

在本节中，我们将列出 Python 的关键字和内置函数名称，以便你知道哪些名称需要避免使用。

### Python 关键字

以下每个关键字都有特定的含义，如果你尝试将它们用作变量名，会出现错误。

```py
False      await      else       import     pass
None       break      except     in         raise
True       class      finally    is         return
and        continue   for        lambda     try
as         def        from       nonlocal   while
assert     del        global     not        with
async      elif       if         or         yield
```

### Python 内置函数

如果你使用以下这些现成的内置函数作为变量名，你不会遇到错误，但会覆盖该函数的行为：

```py
abs()           hash()            slice()
aiter()         help()            sorted()
all()           hex()             staticmethod()
any()           id()              str()
anext()         input()           sum()
ascii()         int()             super()
bin()           isinstance()      tuple()
bool()          issubclass()      type()
breakpoint()    iter()            vars()
bytearray()     len()             zip()
bytes()         list()            __import__()
callable()      locals()
chr()           map()
classmethod()   max()
compile()       memoryview()
complex()       min()
delattr()       next()
dict()          object()
dir()           oct()
divmod()        open()
enumerate()     ord()
eval()          pow()
exec()          print()
filter()        property()
float()         range()
format()        repr()
frozenset()     reversed()
getattr()       round()
globals()       set()
hasattr()       setattr()
```
