

## 第八章：8 在 Python 中处理数据



Python 基础已掌握，但仍有许多内容需要学习。在本章中，你将扩展编程技能，开始直接调查数据集，包括 BlueLeaks 数据集和 2022 年俄罗斯入侵乌克兰后，一支支持普京的勒索软件团伙泄露的聊天记录。

我们将讨论一些更高级的 Python 主题，比如如何使用模块、如何遍历文件系统，以及如何在 Python 中创建自己的命令行程序。你将编写程序，查找文件夹中的所有文件，包括 BlueLeaks 数据集中的数十万文件，并学习如何为程序添加参数。你还将开始使用 Python 中的一种新类型的变量——字典，它对于处理无法简单存储在列表中的复杂数据非常有用。与上一章一样，未来的章节依赖于你对本章内容的理解。

### 模块

正如你在第七章中学到的，函数是可重用的代码块，你可以运行它们任意次数，而无需重新编写代码。Python *模块* 类似，但它们不仅让单个代码块可重用，而是让整个 Python 文件（或多个文件）都可以重用。你可以将模块看作是一个独立的 Python 文件，可以将其加载到当前正在工作的文件中。

Python 包含丰富的功能，但默认情况下并非所有功能对每个 Python 脚本都可用。相反，它们存储在 *内置* 模块中，这些模块是 Python 自带的。一旦你使用 import 语句将模块导入到脚本中，你就可以通过语法 module_name .item_name 访问该模块中定义的所有函数、变量和其他 Python 对象。

例如，time 模块包含函数 time.sleep()（发音为“time 点 sleep”），它使得程序在继续执行下一行代码之前，暂停给定的秒数。运行以下命令导入 time 模块，然后让它指示 Python 等待五秒钟：

```
>>> **import time**

>>> **time.sleep(5)**
```

你的 Python 解释器应等待五秒钟后才会再次显示提示符。

以下是我最常用的一些内置模块：

os 包含用于浏览文件系统的实用函数，例如 os.listdir() 和 os.walk()。它还包含子模块 os.path，其中有许多函数用于检查文件。例如，它包括 os.path.isfile() 和 os.path.isdir()，可以帮助判断特定路径是文件还是文件夹。

csv 让你处理 CSV 电子表格数据。

json 让你处理 JSON 数据。

datetime 包含用于处理日期和时间的实用 Python 功能。例如，它可以将像 February 24, 2022 5:07:20 UTC+3（俄罗斯入侵乌克兰的确切时间）这样的字符串转换为 Python 可以理解并与其他时间戳进行比较的时间戳，然后再将其转换回你选择的任何格式的字符串。

你将在本章后面广泛使用 os 模块，在第九章中使用 csv 模块，在第十一章中使用 json 模块。在本章稍后，你将简要了解 datetime 的使用，当你查看来自勒索软件团伙的聊天记录时，以及在第十四章的案例研究中，你将分析泄露的新纳粹聊天记录。

随着程序变得越来越复杂，你可能会发现将程序拆分成多个文件是很有用的，每个文件包含你代码的不同部分。当你这样做时，你就在创建自己的模块。模块的名称与其文件名相同。例如，如果你在一个名为 *helpers.py* 的文件中定义了一些函数，另一个 Python 文件可以通过导入 helpers 模块来访问这些函数。*helpers.py* 文件可能包含以下代码：

```
def get_tax(price, tax_rate):

    return price * tax_rate

def get_net_price(price, tax_rate):

    return price + get_tax(price, tax_rate)
```

这个模块包含两个用于计算销售税的函数，get_tax() 和 get_net_price()。以下 Python 脚本，*price.py*，通过如下方式导入：

```
import helpers

total_price = helpers.get_net_price(50, 0.06)

print(f"A book that costs $50, and has 6% sales tax, costs ${total_price}")
```

第一行 import helpers 使得在 helpers 模块中定义的函数对该脚本可用。第二行调用该模块中的 helpers.get_net _price() 函数，并将返回值存储在变量 total_price 中。第三行显示 total_price 的值。

运行这个脚本时，它的样子如下：

```
micah@trapdoor module % **python3 price.py**

A book that costs $50, and has 6% sales tax, costs $53.0
```

运行 *price.py* 脚本执行了在 helpers 模块中定义的代码。在该模块内，get_net_price() 函数调用了 get_tax() 并使用其返回值计算净价，然后将 *该* 值返回到 *price.py* 脚本中。

在你开始编写第一个高级 Python 脚本（练习 8-1）之前，我们先来看一下启动新 Python 脚本的最佳方法。

### Python 脚本模板

我对所有 Python 脚本使用相同的基本模板，将代码放入一个名为 main() 的函数中，然后在文件底部调用该函数。这不是必须的（毕竟你在第七章中编写的脚本都没有这样做），但这是一种很好的组织代码的方法。它的样子如下：

```
def main():

    pass

if __name__ == "__main__":

    main()
```

该模板定义了 main() 函数，并用 pass 语句告诉 Python：“跳过这一行。”稍后我会将 pass 替换为脚本的实际内容。

接下来，if 语句告诉 Python 在什么条件下应该运行 main()。Python 会自动定义 __name__ 变量，定义的值取决于正在运行的 Python 文件。如果你直接运行当前执行的 Python 文件，那么 Python 会将 __name__ 的值设置为 __main__ 字符串。但是，如果你从另一个脚本中导入当前执行的 Python 文件，Python 会将 __name__ 的值设置为导入模块的名称。使用上一节的示例，如果你直接运行 *helpers.py* 脚本，那么该脚本内部的 __name__ 值将是 __main__，但是如果你运行 *price.py* 脚本，那么 *price.py* 内部的 __name__ 值将是 __main__，而 *helpers.py* 内部的 __name__ 值将是 helpers。

简而言之，如果你直接运行脚本，main() 函数会运行。但是，如果你将脚本作为模块导入到另一个脚本或 Python 解释器中，除非你自己调用它，否则 main() 函数不会运行。这样，如果你有多个 Python 脚本在同一个文件夹中，你可以让一个脚本导入另一个脚本并调用其中定义的函数，而不必担心调用后者脚本的 main() 函数。

在我创建这个模板脚本后，我开始在 main() 函数中填写我希望脚本执行的内容。将脚本的主要逻辑放入函数中，可以让你使用 return 语句提前结束 main()，从而提前退出脚本。当你不在函数中时，不能使用 return。

在接下来的练习中，你将通过编写脚本来开始调查 BlueLeaks，实践这一点。

### 练习 8-1：遍历 BlueLeaks 中的文件

为了高效地调查数据集，你需要能够编写代码，扫描大量的文件——有时是成千上万甚至更多的文件。 在本练习中，你将学习如何使用os模块中的函数，在 Python 中遍历文件系统，并处理 BlueLeaks 数据集。你还将依赖在第七章中学到的基础技能，如使用变量、for循环和if语句。

在阅读并运行脚本时，随时可以修改代码，并尝试运行这些版本。你可能会发现一些我没有想到的启示。

#### 列出文件夹中的文件名

首先使用os.listdir()列出*BlueLeaks-extracted*文件夹中的文件。在你的文本编辑器中，创建一个名为*list-files1.py*的文件，并输入这段简短的脚本（或从[*https://<wbr>github<wbr>.com<wbr>/micahflee<wbr>/hacks<wbr>-leaks<wbr>-and<wbr>-revelations<wbr>/blob<wbr>/main<wbr>/chapter<wbr>-8<wbr>/list<wbr>-files1<wbr>.py*](https://github.com/micahflee/hacks-leaks-and-revelations/blob/main/chapter-8/list-files1.py)复制粘贴它）：

```
import os

def main():

    blueleaks_path ="`/Volumes/datasets/BlueLeaks-extracted`"

    for filename in os.listdir(blueleaks_path):

        print(filename)

if __name__ == "__main__":

    main()
```

首先，脚本导入了os模块。然后，它定义了变量blueleaks_path，该变量保存了*BlueLeaks-extracted*文件夹的路径（请更新脚本以包含该文件夹在你计算机上的路径）。os.listdir()函数接受文件夹路径作为参数，并返回该文件夹中的文件名列表。代码使用for循环遍历os.listdir(blueleaks_path)的输出，显示每个文件名。

> 注意

*Windows 路径包括反斜杠字符(\)，而 Python 字符串将其视为转义字符。例如，如果你的* BlueLeaks-extracted *文件夹位于* D:\BlueLeaks-extracted*，Python 会误解字符串 "D:\BlueLeaks-extracted"，认为 \B是一个特殊字符。为了正确表示任何存储为字符串的 Windows 路径，请使用 \\ 而不是 \。在这种情况下，将 blueleaks_path 字符串设置为 "D:\\BlueLeaks-extracted"。*

运行这个脚本。以下是我电脑上输出的内容：

```
micah@trapdoor chapter-8 % **python3 list-files1.py**

211sfbay

Securitypartnership

acprlea

acticaz

akorca

`--snip--`
```

接下来，你将尝试稍微复杂一点的操作。你不仅仅列出 BlueLeaks 中的文件名，还会检查每个文件名是否是文件夹，如果是文件夹，你将打开每个文件夹并统计它们包含多少个文件和子文件夹。

#### 计算文件夹中的文件和文件夹数量

创建一个名为*list-files2.py*的文件，并输入以下代码（或从 [*https://<wbr>github<wbr>.com<wbr>/micahflee<wbr>/hacks<wbr>-leaks<wbr>-and<wbr>-revelations<wbr>/blob<wbr>/main<wbr>/chapter<wbr>-8<wbr>/list<wbr>-files2<wbr>.py*](https://github.com/micahflee/hacks-leaks-and-revelations/blob/main/chapter-8/list-files2.py) 复制粘贴）：

```
import os

def main():

    blueleaks_path = "`/Volumes/datasets/BlueLeaks-extracted`"

  ❶ for bl_folder in os.listdir(blueleaks_path):

bl_folder_path = os.path.join(blueleaks_path, bl_folder) ❷ if not os.path.isdir(bl_folder_path):

            continue

      ❸ files_count = 0

        folders_count = 0

      ❹ for filename in os.listdir(bl_folder_path):

            filename_path = os.path.join(bl_folder_path, filename)

          ❺ if os.path.isfile(filename_path):

                files_count += 1

            if os.path.isdir(filename_path):

                folders_count += 1

      ❻ print(f"{bl_folder} has {files_count} files, {folders_count} folders")

if __name__ == "__main__":

    main()
```

这个脚本计算它在每个 BlueLeaks 文件夹中找到的文件和文件夹的数量。它像*list-files1.py*一样开始，导入 os 模块，并定义 blueleaks_path 变量（记得更新变量的值以匹配你电脑上的正确路径）。

第一个 for 循环遍历 *BlueLeaks-extracted* 文件夹中的文件名，这一次将每个文件名保存在 bl_folder 变量中，因此它的值会类似于 miacx 或 ncric ❶。接下来，脚本相应地设置新的 bl_folder_path 变量的值。os.path.join() 函数将文件名连接起来，形成完整的路径。它的第一个参数是起始路径，然后将所有其他参数添加到路径的末尾。例如，如果 bl_folder 的值是 miacx，那么该函数将返回字符串 /Volumes/datasets/BlueLeaks-extracted/miacx（如果你的 blueleaks_path 不同，或者你使用的是 Windows 且文件名使用反斜杠而不是斜杠，输出结果会有所不同）。

由于你想要查看 bl_folder_path 目录内部并统计它包含的文件和文件夹数量，脚本需要检查它是否确实是一个文件夹而不是一个文件，可以使用 os.path.isdir() 函数 ❷。如果 bl_folder_path 不是一个文件夹，脚本会执行 continue 语句。这个语句只能在循环内部执行，告诉 Python 立即继续到下一次循环的迭代。简而言之，如果脚本遇到的是一个文件而不是文件夹，它会忽略它并继续向下执行。

然后，脚本准备计算每个单独的 BlueLeaks 文件夹中的文件和文件夹数量，在循环中定义变量 files_count 和 folders_count，并将它们的初始值设置为 0 ❸。

第二个 `for` 循环遍历第一个 `for` 循环中的 BlueLeaks 文件夹中的文件，将每个文件名保存在 `filename` 变量中 ❹。在这个循环内部，脚本将 `filename_path` 定义为当前文件名的绝对路径。例如，如果 `filename` 的值是像 `Directory.csv` 这样的字符串，那么 `filename_path` 的值将是像 `/Volumes/datasets/BlueLeaks-extracted/211sfbay/Directory.csv` 这样的字符串。

脚本接着会检查这个绝对路径是文件还是文件夹，使用 `os.path.isfile()` 和 `os.path.isdir()` 函数 ❺。如果路径是文件，脚本会将 `files_count` 变量增加 1；如果是文件夹，脚本会将 `folders_count` 增加 1。当第二个 `for` 循环执行完毕时，这两个变量应该包含你当前在第一个 `for` 循环中遍历的 BlueLeaks 文件夹的文件和文件夹总数。最后，脚本会显示一个 f-string，展示这些数字 ❻。

尝试运行脚本。输出应该显示每个 BlueLeaks 文件夹包含的文件和文件夹数量，并可能显示文件夹的列表顺序有所不同：

```
micah@trapdoor chapter-8 % **python3 list-files2.py**

bostonbric has 506 files, 10 folders

terrorismtip has 207 files, 0 folders

ociac has 216 files, 1 folders

usao has 0 files, 84 folders

alertmidsouth has 512 files, 10 folders

chicagoheat has 499 files, 10 folders

`--snip--`
```

到目前为止，你已经结合了 `os` 模块中的各种函数，列出了 BlueLeaks 文件夹中的文件名，并检查每个名字是否实际上是文件还是另一个文件夹。现在是时候学习编写可以遍历 BlueLeaks 文件夹中嵌套文件夹的代码了。

### 使用 os.walk() 遍历文件夹

假设你想编写一个程序，显示文件夹及其子文件夹、子子文件夹等所有文件。当你有嵌套文件夹时，且不知道文件夹结构的深度，仅使用 `os.listdir()`、`os.path.isfile()` 和 `os.path.isdir()` 列出所有文件名并不那么简单。Python 的 `os.walk()` 函数解决了这个问题。

os.walk() 函数将文件夹路径作为参数，并返回一个包含多个值的 *元组* 列表。要定义一个元组，你需要将所有值用逗号分隔并放在括号中。例如，(3, 4) 是一个元组，("cinco", "seis", "siete") 也是一个元组。元组也可以包含混合类型的值，例如 (1, "dos")，并且可以包含任意数量的值。

os.walk() 函数返回一个元组列表，每个元组包含三个值：

```
(dirname, subdirnames, filenames)
```

其中 dirname 是一个字符串，subdirnames 是一个字符串列表，filenames 是一个字符串列表。例如，以下代码会遍历 os.walk(path) 的返回值：

```
for dirname, subdirnames, filenames in os.walk(path):

    print(f"The folder {dirname} has subfolders: {subdirnames} and files: {filenames}")
```

当你使用 `for` 循环遍历列表时，通常会为列表中的每一项分配一个单独的变量。然而，由于每个项目是一个元组，你可以为它分配三个变量：`dirname`、subdirnames 和 filenames。在每次循环中，这组变量的值会有所不同：dirname 的值是文件夹的路径，subdirnames 的值是该文件夹中的子文件夹列表，而 filenames 的值是该文件夹中的文件列表。

例如，假设你有一个名为 *example* 的文件夹，它包含这些子文件夹和文件：

```
example

├── downloads

│   ├── screenshot.png

│   └── paper.pdf

└── documents

    ├── work

    │   └── finances.xlsx

    └── personal
```

这个文件夹有两个子文件夹：*downloads*（包含 *screenshot.png* 和 *paper.pdf*）和 *documents*。*documents* 文件夹有它自己的子文件夹：*work*（包含 *finances.xlsx*）和 *personal*。

以下命令会遍历 os.walk("./example") 的返回值，其中 *./example* 是指向 *example* 文件夹的路径，以查找每次循环中 dirname、subdirnames 和 filenames 的值：

```
>>> **for dirname, subdirnames, filenames in os.walk("./example"):**

...     **print(f"The folder {dirname} has subfolders: {subdirnames} and files: {filenames}")**

...
```

运行此命令会返回以下输出：

```
The folder ./example has subfolders: ['documents', 'downloads'] and files: []

The folder ./example/documents has subfolders: ['personal', 'work'] and files: []

The folder ./example/documents/personal has subfolders: [] and files: []

The folder ./example/documents/work has subfolders: [] and files: ['finances.xlsx']

The folder ./example/downloads has subfolders: [] and files: ['paper.pdf', 'screenshot.png']
```

这段代码每次循环都会针对一个文件夹，包括所有的子文件夹，文件夹的路径存储在 dirname 中。该文件夹中的子文件夹列表存储在 subdirnames 中，文件列表存储在 filenames 中。一旦遍历完文件夹及其所有子文件夹，for 循环就会结束。

每当你需要遍历包含大量嵌套文件夹的数据集中的所有文件时，你将希望使用 os.walk()。通过一个简单的 for 循环，你就能够编写代码，检查整个数据集中的每个文件。os.walk() 函数有许多用途，包括找出哪些文件是最大的或最小的，正如接下来你将看到的那样。

### 练习 8-2: 查找 BlueLeaks 中最大的文件

在这个练习中，你将使用 os.walk() 编写一个脚本，遍历 BlueLeaks 中的所有文件、文件夹和子文件夹；测量每个文件的大小；并显示超过 100MB 的文件名。这个代码允许你循环遍历文件夹中的所有文件，无论文件夹结构有多深。

创建一个名为 *find-big-files.py* 的文件，并输入以下代码（或从 [*https://<wbr>github<wbr>.com<wbr>/micahflee<wbr>/hacks<wbr>-leaks<wbr>-and<wbr>-revelations<wbr>/blob<wbr>/main<wbr>/chapter<wbr>-8<wbr>/find<wbr>-big<wbr>-files<wbr>.py*](https://github.com/micahflee/hacks-leaks-and-revelations/blob/main/chapter-8/find-big-files.py) 复制并粘贴代码）：

```
import os

def main():

    blueleaks_path ="`/Volumes/datasets/BlueLeaks-extracted`"

    for dirname, subdirnames, filenames in os.walk(blueleaks_path):

        for filename in filenames:

            absolute_filename = os.path.join(dirname, filename)

            size_in_bytes = os.path.getsize(absolute_filename)

            size_in_mb = int(size_in_bytes / 1024 / 1024)

            if size_in_mb >= 100:

                print(f"{absolute_filename} is {size_in_mb}MB")

if __name__ == "__main__":

    main()
```

在 main() 函数中，脚本首先将 blueleaks_path 变量定义为 *BlueLeaks-extracted* 文件夹的路径，并使用 os.walk() 函数遍历整个 BlueLeaks 数据集中的所有文件。在第一个 for 循环中的每次循环中，都有 dirname、subdirnames 和 filenames 变量。os.walk() 返回的每个项目表示 BlueLeaks 数据集中的一个不同文件夹或子文件夹，因此当这个循环结束时，代码将已经遍历了整个数据集。

为了找到最大的文件，下一步是使用另一个 for 循环查看每个文件，这次是循环遍历 filenames。在这个第二个 for 循环中，脚本将 absolute_filename 定义为文件名的绝对路径。由于 dirname 告诉脚本它正在查看哪个文件夹，而 filename 告诉脚本它正在查看哪个文件，脚本将这些值传递给 os.path.join() 以将它们合并，创建文件名的绝对路径。

一个新函数，os.path.getsize()，返回所考虑文件的大小（以字节为单位），并将其存储在变量 size_in_bytes 中。脚本接着将该值从字节转换为兆字节（并将其存储在变量 size_in_mb 中），并检查它是否大于或等于 100MB。如果是，输出将显示文件名及其大小（以兆字节为单位），通过 print() 函数显示。

尝试运行这个脚本。它将比本章之前的脚本运行得更慢，因为这一次，你要测量 BlueLeaks 中每一个文件的大小。以下是我运行它时输出的样子（你的输出可能会以不同的顺序显示）：

```
micah@trapdoor chapter-8 % **python3 find-big-files.py**

/Volumes/datasets/BlueLeaks-extracted/usao/usaoflntraining/files/VVSF00000/001.mp4 is 644MB /Volumes/datasets/BlueLeaks-extracted/chicagoheat/html/ZA-CHICAGO HEaT_LR-20160830-034_Final

Files.pdf is 102MB

/Volumes/datasets/BlueLeaks-extracted/nmhidta/files/RFIF300000/722.pdf is 148MB

/Volumes/datasets/BlueLeaks-extracted/nmhidta/files/RFIF200000/543.pdf is 161MB

/Volumes/datasets/BlueLeaks-extracted/nmhidta/files/RFIF100000/723.pdf is 206MB

/Volumes/datasets/BlueLeaks-extracted/fbicahouston/files/VVSF00000/002.mp4 is 145MB

/Volumes/datasets/BlueLeaks-extracted/fbicahouston/files/PSAVF100000/009.mp4 is 146MB

/Volumes/datasets/BlueLeaks-extracted/fbicahouston/files/PSAVF100000/026.mp4 is 105MB

`--snip--`
```

脚本应显示 BlueLeaks 中至少 100MB 的 101 个文件的绝对路径，以及每个文件的大小。

### 第三方模块

除了内置模块，Python 还支持第三方模块，你可以轻松地将其集成到自己的代码中。我编写的大多数 Python 脚本，即使是简单的脚本，也依赖于至少一个第三方模块（当 Python 程序依赖第三方模块时，这些模块被称为*依赖项*）。在这一节中，你将学习如何安装第三方模块并在自己的脚本中使用它们。

Python 包索引（PyPI）包含数十万个第三方 Python *包*，即 Python 模块和子包的集合。Pip，即 Python 包安装器，是一个类似于 Ubuntu 的 apt 或 macOS 的 Homebrew 的包管理器，用于安装托管在 PyPI 上的包。你可以在 PyPI 的网站上搜索包（[*https://<wbr>pypi<wbr>.org*](https://pypi.org)），然后通过运行 python3 -m pip install package_name 命令来安装一个包。

例如，我经常使用一个叫做 Click 的包，它代表的是命令行接口创建工具包（Command Line Interface Creation Kit）。`click` Python 模块使得向脚本添加命令行参数变得简单。要查看在尚未安装该模块的情况下尝试导入它会发生什么，打开 Python 解释器并运行 `import click`。假设你尚未安装该包，你应该会看到一个 `ModuleNotFoundError` 错误信息：

```
Traceback (most recent call last):

  File "<stdin>", line 1, in <module>

ModuleNotFoundError: No module named 'click'

>>>
```

现在退出 Python 解释器，并通过运行以下命令使用 pip 安装 `click`：

```
micah@trapdoor ~ % **python3 -m pip install click**

Collecting click

  Using cached click-8.1.3-py3-none-any.whl (96 kB)

Installing collected packages: click

Successfully installed click-8.1.3
```

再次打开 Python 解释器，尝试重新导入 `click`：

```
>>> **import click**

>>>
```

如果没有弹出错误信息，说明你已经成功导入了 `click` 模块，并且它的附加功能现在可以使用了。

卸载包的命令是 `python3 -m pip uninstall` package_name。尝试卸载 `click`：

```
micah@trapdoor ~ % **python3 -m pip uninstall click**

Found existing installation: click 8.1.3

Uninstalling click-8.1.3:

  Would remove:

    /usr/local/lib/python3.10/site-packages/click-8.1.3.dist-info/*

    /usr/local/lib/python3.10/site-packages/click/*

Proceed (Y/n)? **y**

  Successfully uninstalled click-8.1.3
```

如你所见，当我运行这个命令时，输出列出了 pip 需要删除的文件，以卸载 `click` 模块，然后询问我是否要继续。我输入了 `y` 并按下了回车键，文件被删除，模块被卸载。

你可以像下面这样一次性安装多个 Python 包：

```
**python3 -m pip install** **`package_name1 package_name2 package_name3`**
```

卸载操作也同样适用。

通常，你会在一个名为 *requirements.txt* 的文件中定义脚本所需的 Python 包，然后使用 `python3 -m pip install -r requirements.txt` 命令一次性安装所有包。例如，假设除了使用 `click`，你还想使用 HTTP 客户端 `httpx` 来加载 Python 中的网页，并且使用 `sqlalchemy` 模块来处理 SQL 数据库。为了在 Python 脚本中包含这三者，首先创建一个 *requirements.txt* 文件，将每个包的名称写在单独的一行上：

```
click

httpx

sqlalchemy
```

然后运行以下命令以同时安装它们：

```
micah@trapdoor chapter-8 % **python3 -m pip install -r requirements.txt**

Collecting click

  Using cached click-8.1.3-py3-none-any.whl (96 kB)

Collecting httpx

  Using cached httpx-0.23.0-py3-none-any.whl (84 kB)

`--snip--`

Successfully installed anyio-3.6.1 certifi-2022.9.24 click-8.1.3 h11-0.12.0 httpcore-0.15.0

httpx-0.23.0 idna-3.4 rfc3986-1.5.0 sniffio-1.3.0 sqlalchemy-1.4.41
```

正如你所看到的，这个命令不仅仅安装了那三个 Python 包：rfc3986、certifi、sniffio，等等也被包含在内。这是因为 click、httpx 和 sqlachemy 各自有自己的依赖项。例如，httpcore 是 httpx 包的依赖项，因此也会安装它。总结一下，*requirements.txt* 文件定义了你项目的依赖项，每个依赖项可能会有自己的一组包。

> 注意

*要了解更多关于如何使用* *httpx* *以及其他 Python 模块来自动化与网站的交互，查阅 附录 B。不过我建议你等到完成 第七章、第八章、第九章 和 第十一章 后再学习，因为 附录 B 中的内容依赖于你在这些章节中学到的技能。*

现在你已经知道如何安装第三方模块，接下来你将练习使用 Click。

### 练习 8-3：使用 Click 练习命令行参数

正如你在上一节中所学到的，Click 包使得将命令行参数添加到脚本中变得非常简单。你可以使用它来定义变量，从终端将这些变量传递到你的 main() 函数，而不需要在代码中定义这些变量。在这个练习中，你将通过编写一个示例脚本来学习如何使用 Click，为后续练习中使用这个模块做准备。

首先，再次使用 pip 安装 Click 包，通过运行 python3 -m pip install click。接下来，打开你的文本编辑器，输入以下 Python 脚本，*exercise-8-3.py*（或从 [*https://<wbr>github<wbr>.com<wbr>/micahflee<wbr>/hacks<wbr>-leaks<wbr>-and<wbr>-revelations<wbr>/blob<wbr>/main<wbr>/chapter<wbr>-8<wbr>/exercise<wbr>-8<wbr>-3<wbr>.py*](https://github.com/micahflee/hacks-leaks-and-revelations/blob/main/chapter-8/exercise-8-3.py) 复制并粘贴）：

```
import click

@click.command()

@click.argument("name")

def main(name):

    """Simple program that greets NAME"""

    print(f"Hello {name}!")

if __name__ == "__main__":

    main()
```

首先，脚本导入了 click 模块。接着它运行了一些 *装饰器*，这些是以 @ 开头的函数调用，它们为你即将定义的另一个函数——在此案例中是 main() 函数——添加功能。@click.command() 装饰器告诉 Click main() 是一个命令，而 @click.argument("name") 装饰器告诉 Click 这个命令有一个名为 name 的参数。

接下来，脚本定义了 main() 函数，该函数以 name 作为参数。这个函数有一个文档字符串，Simple program that greets NAME。Click 使用这个文档字符串来生成 --help 命令的输出，正如你很快会看到的那样。main() 函数仅仅显示你作为参数传入的名字。

最后，脚本调用了 main() 函数。请注意，尽管 main() 函数需要一个参数（name），但是脚本在调用该函数时并没有显式传递该参数。这就是 Click 装饰器的魔力所在。当脚本调用 main() 时，Click 会自动判断需要传入哪些参数，查找 CLI 参数中的值，然后为你传入这些值。

按照如下方式运行脚本：

```
micah@trapdoor chapter-8 % **python3 exercise-8-3.py**

Usage: click-example.py [OPTIONS] NAME

Try 'click-example.py --help' for help.

Error: Missing argument 'NAME'.
```

当你运行程序时，如果没有传入正确的 CLI 参数，Click 会告诉你哪里出错了。正如你所看到的，你缺少了必需的 NAME 参数。Click 还会告诉你可以通过再次运行脚本并传入 --help 参数来获取帮助。

尝试运行 --help 命令：

```
micah@trapdoor chapter-8 % **python3 exercise-8-3.py --help**

Usage: click-example.py [OPTIONS] NAME

  Simple program that greets NAME

Options:

  --help  Show this message and exit.
```

这次，输出将显示基于文档字符串的程序描述。任何使用 Click 的 CLI 程序，在你运行它并传入 --help 时，都会显示该命令的文档字符串。

尝试再次运行命令，这次传入一个名字。例如，当我传入 Eve 作为名字时，结果是这样的：

```
micah@trapdoor chapter-8 % **python3 exercise-8-3.py Eve**

Hello Eve!
```

> 注意

*你可以在此阅读更多关于使用 Click 的信息：* [`click.palletsprojects.com`](https://click.palletsprojects.com)*。*

### 通过命令行参数避免硬编码

正如你在前几章中看到的，CLI 参数允许你以多种方式运行同一个程序，针对不同的数据。例如，在第四章中，你使用了 du 命令，通过将文件夹路径作为参数来估算文件夹的磁盘空间。在 du -sh --apparent-size path 中，参数是 -sh、--apparent-size 和 path。

如果 du 命令只能测量单个硬编码文件夹的磁盘空间，它的用途将大大降低。*硬编码*意味着将信息（如路径）直接嵌入到源代码中。通过让用户在运行程序时作为参数提供这些信息，你可以避免在 CLI 程序中硬编码任何内容。

将路径传入脚本，而不是硬编码它们，会带来更好的用户体验。在本章之前的练习中，你将 BlueLeaks 数据集的路径硬编码到 Python 脚本中。然而，如果你将适当的路径作为参数传入，其他人也可以在不修改脚本的情况下使用它——他们只需在运行时传入*他们*的路径即可。

使用参数而不是硬编码也可以使你的脚本更具普适性。例如，在练习 8-2 中，你编写了一个脚本来查找 BlueLeaks 数据集中所有至少 100MB 的文件。通过使用 CLI 参数，你可以让这个脚本适用于任何数据集，而不仅仅是 BlueLeaks，还可以设置任何最小文件大小，使得你能在各种情况中运行它。你只需要将数据集路径和最小文件大小作为 CLI 参数传入即可。你将在下一个练习中尝试这个。

### 练习 8-4：在任何数据集中查找最大的文件

在本练习中，你将修改在练习 8-2 中编写的脚本，使其能够处理任何数据集和任何最小文件大小，使用 CLI 参数。在接下来的章节中，你将编写简单的 Python 脚本，利用 Click 处理 CLI 参数，这样你就可以提供你将要使用的数据集路径。

创建一个名为*exercise-8-4.py*的新文件，并将*exercise-8-2.py*代码复制粘贴到其中。接下来，按照加粗部分修改代码（或者在[*https://<wbr>github<wbr>.com<wbr>/micahflee<wbr>/hacks<wbr>-leaks<wbr>-and<wbr>-revelations<wbr>/blob<wbr>/main<wbr>/chapter<wbr>-8<wbr>/exercise<wbr>-8<wbr>-4<wbr>.py*](https://github.com/micahflee/hacks-leaks-and-revelations/blob/main/chapter-8/exercise-8-4.py)查看完整的修改脚本）：

```
import os

**import click**

**@click.command()**

**@click.argument("path")**

**@click.argument("min_file_size", type=click.INT)**

def main(**path, min_file_size**):

    **"""Find files in PATH that are at least MIN_FILE_SIZE MB big"""**

    for dirname, subdirnames, filenames in os.walk(**path**):

        for filename in filenames:

            absolute_filename = os.path.join(dirname, filename)

            size_in_bytes = os.path.getsize(absolute_filename)

            size_in_mb = int(size_in_bytes / 1024 / 1024)

            if size_in_mb >= **min_file_size**:

                  print(f"{absolute_filename} is {size_in_mb}MB")

if __name__ == "__main__":

    main()
```

这段代码在文件的顶部导入了click模块。接着，它在main()函数之前添加了 Click 装饰器：@click.command()使得main()函数成为一个 Click 命令，而@click.argument()将path和min_file_size添加为参数。脚本通过type=click.INT指定min_file_size参数应为*整数*，即一个完整的数字，而不是字符串。然后，它将path和min_file_size作为参数添加到main()函数，并添加了一个描述该命令功能的文档字符串。

新的脚本使用参数而不是硬编码的值。它删除了定义blueleaks_path变量的那一行，并在os.walk()函数调用中，将blueleaks_path改为参数path。最后，它将size_in_mb中的100改为min_file_size。

现在，你可以使用这个程序查找 BlueLeaks 数据集或其他地方任何文件夹中的大文件。例如，当我在 Mac 的*/Applications*目录下查找所有至少 500MB 的文件时，它看起来是这样的：

```
micah@trapdoor chapter-8 % **python3 exercise-8-4.py /Applications 500**

/Applications/Dangerzone.app/Contents/Resources/share/container.tar.gz is 668MB

/Applications/Docker.app/Contents/Resources/linuxkit/services.iso is 602MB
```

如你所见，我安装的应用中只有两个包含这么大文件的：Dangerzone 和 Docker Desktop。

现在你已经了解了如何使用 Click 向 Python 脚本添加 CLI 参数，未来编写程序时，你应该能够避免像数据集路径这样的信息被硬编码。

接下来，我们将转换方向，探索一种新的强大类型的 Python 变量——字典。

### 字典

在你的探索过程中，有时你需要记录比简单列表更具结构的数据。为此，你可以使用 Python 字典。与一组项目不同，*字典*（简写为*dict*）是一个键的集合，键映射到对应的值。*键*是你用来在字典中保存或获取信息的标签，*值*是实际保存或获取的信息。我写的几乎每个涉及数据的 Python 脚本都会使用字典。在这一节中，你将学习如何定义字典、从字典中获取值、向字典中添加值以及更新字典中的现有值。

#### 定义字典

字典使用大括号（{和<|samp class="SANS_TheSansMonoCd_W5Regular_11">}）定义，有时也被称为花括号。在大括号内是按格式排列的键值对列表，格式为key: value，每一对键值之间用逗号分隔，例如{"country": "Italy", "drinking_age": 18}。对于较长的字典，你可以通过将每个键值对放在独立的行上来提高代码的可读性。

Listing 8-1 显示了存储在变量capitals中的字典示例。

```
capitals = {

    "United States": "Washington, DC",

    "India": "New Delhi",

    "South Africa": "Cape Town",

    "Brazil": "Brasília",

    "Germany": "Berlin",

    "Russia": "Moscow",

    "China": "Beijing"	

}
```

Listing 8-1：存储在capitals变量中的字典

在这种情况下，键是国家名称，而值是这些国家的首都。

字典中的每个键只能有一个值。如果你尝试多次设置相同的键，Python 将保存你最后设置的版本。例如，如果你定义一个字典并多次使用name键，字典将用最新的值覆盖之前的值：

```
>>> **test_dict = {"name": "Alice", "name": "Bob", "hobby": "cryptography"}**

>>> **print(test_dict)**

{'name': 'Bob', 'hobby': 'cryptography'}
```

然而，你也可以使用列表或其他字典作为值：

```
>>> **test_dict = {"names": ["Alice", "Bob"], "hobby": "cryptography"}**

>>> **print(test_dict)**

{'names': ['Alice', 'Bob'], 'hobby': 'cryptography'}
```

在这种情况下，键names的值是['Alice', 'Bob']，这本身就是一个列表。你可以使用列表和字典的组合来组织几乎任何类型的数据，无论数据多么复杂，从而让你在 Python 中更容易处理它。

#### 获取与设置值

要从字典中检索你存储的项，将包含项键的方括号添加到字典名称的末尾。如果你尝试使用一个未定义的键，脚本会因KeyError而崩溃。例如，下面是如何在capitals字典中查找某些国家的首都：

```
>>> **capitals["United States"]**

'Washington, DC'

>>> **capitals["China"]**

'Beijing'

>>> **capitals["Kenya"]**

Traceback (most recent call last):

  File "<stdin>", line 1, in <module>

KeyError: 'Kenya'
```

当你运行capitals["Kenya"]时，Python 会抛出错误消息KeyError: 'Kenya'。这意味着Kenya不是capitals字典中的有效键。你可以看到，在清单 8-1 中定义的唯一键是United States、India、South Africa、Brazil、Germany、Russia和China。由于Kenya不是此字典中的键，因此无法检索其值。

你可以像这样向字典中添加新的键值对，或者更新现有的键值对：

```
>>> **capitals["Kenya"] = "Nairobi"**

>>> **capitals["United States"] = "Mar-a-Lago"**

>>> **print(capitals)**

{'United States': 'Mar-a-Lago', 'India': 'New Delhi', 'South Africa': 'Cape Town', 'Brazil': 'Brasília', 'Germany': 'Berlin', 'Russia': 'Moscow', 'China': 'Beijing', 'Kenya': 'Nairobi'}
```

这段代码定义了一个新的键Kenya，并将其值设置为Nairobi。它还更新了一个现有的键United States，将其值更新为Mar-a-Lago，并覆盖了其旧值Washington, DC。

### 在 Conti 聊天日志中导航字典和列表

你可以将字典和列表结合在一个灵活的数据结构中，这样可以表示各种各样的信息。如果你正在编写用于处理数据集的 Python 代码，那么你很可能需要这两者。你可能会直接以这种格式加载数据，或者你可能会创建自己的字典和列表来存储数据的各个方面。

为了描述如何使用包含字典和列表组合的数据结构，我将使用一个来自真实数据集的例子。2022 年 2 月 24 日俄罗斯入侵乌克兰的第二天，臭名昭著的俄罗斯勒索软件团伙 Conti，这个以黑客攻击全球公司并勒索数百万美元而闻名的团伙，在其网站上发布了一份声明，表示全力支持俄罗斯政府。它威胁任何对俄罗斯发起网络攻击的“敌人”，将会对其“关键基础设施”进行报复。三天后，一位乌克兰安全研究员匿名泄露了 30GB 的 Conti 内部数据：黑客工具、培训文档、源代码和聊天记录。这些 Conti 聊天记录最初是以 JSON 文件的形式存在的，这是结构化数据。当你将 JSON 文件加载到 Python 中时，它们会自动作为字典和列表的组合加载。

在这一部分中，你将浏览一些这些聊天记录，以练习处理存储在字典和列表中的真实泄露数据。通过 Python 代码，你将学习如何遍历这些结构以访问特定的数据部分，并且如何快速循环遍历聊天记录，选择你感兴趣的部分。

#### 在 Python 中探索充满数据的字典和列表

你可以从[*https://<wbr>ddosecrets<wbr>.com<wbr>/wiki<wbr>/Conti<wbr>_ransomware<wbr>_chats*](https://ddosecrets.com/wiki/Conti_ransomware_chats)下载完整的 Conti 数据集。然而，在这一部分中，你只会使用数据集中的一个文件，*2022-02-24-general.json*，该文件是由乌克兰安全研究员从一个名为 RocketChat 的聊天系统中提取的。

从[*https://<wbr>github<wbr>.com<wbr>/micahflee<wbr>/hacks<wbr>-leaks<wbr>-and<wbr>-revelations<wbr>/blob<wbr>/main<wbr>/chapter<wbr>-8<wbr>/2022<wbr>-02<wbr>-24<wbr>-general<wbr>.json*](https://github.com/micahflee/hacks-leaks-and-revelations/blob/main/chapter-8/2022-02-24-general.json)下载*2022-02-24-general.json*文件。打开终端，切换到存储该文件的文件夹，并打开 Python 解释器。使用以下命令将该文件加载到字典中：

```
>>> **import json**

>>> **with open("2022-02-24-general.json") as f:**

...     **data** **= json.load(f)**

...
```

这段代码使用了json模块，并将*2022-02-24-general.json*文件中的数据加载到<data>变量中。由于该文件中的聊天记录过长，无法完全显示，但清单 8-2 展示了<data>字典的一部分值，说明了其结构。

```
{

    "messages": [❶

        {

`--snip--`

        },

        {

            "_id": "FmFZbde9ACs3gtw27",

            "rid": "GENERAL",

            "msg": "Некоторые американские сенаторы предлагают помимо соцсетей блокировать в

Россииещё и PornHub!",

            "ts": "2022-02-24T22:02:38.276Z",

            "u": {"_id": "NKrXj9edAPWNrYv5r", "username": "thomas", "name": "thomas"},

            "urls": [],

            "mentions": [],

            "channels": [],

            "md": [

                {

                    "type": "PARAGRAPH",

                    "value": [

                        {

                            "type": "PLAIN_TEXT",

                            "value": "Некоторые американские сенаторы предлагают помимо

соцсетейблокировать в России ещё и PornHub!",

                        }

                  ],

                }

            ],

            "_updatedAt": "2022-02-24T22:02:38.293Z",

        },

        {

`--snip--`

        },

    ],

    "success": True ❷

}
```

清单 8-2：来自 RocketChat 的 Conti 聊天记录

data 变量是一个字典，包含两个键：messages 和 success。你可以通过表达式 data["messages"] ❶ 访问 messages 键的值，它是一个字典的列表。你可以通过它被方括号（[和]）包围来判断 data["messages"] 的值是一个列表，并且你可以通过它被大括号（{ 和 }）包围来判断其中的项目是字典。几乎所有的文件数据都存储在这个列表中。

data["messages"] 列表中的每个字典描述了一条聊天消息。这个代码片段只包括其中一个字典，即列表中的第九条聊天消息（我删除了前八条消息，所以你不能通过这段代码看出它是第九条消息，除非查看原始文件）。你可以通过表达式 data["messages"][8] 访问包含该特定聊天消息的字典。（记住，在编程中我们是从 0 开始计数的，而不是从 1 开始，所以第一个项目在索引 0，第二个项目在索引 1，依此类推。）如果你运行命令 print(data["messages"][8]) 来显示第九条消息的字典，输出应该与列表中的消息相匹配。请注意，正如你在方括号中放置索引数字来从列表中选择项目一样，你也可以在方括号中放置键来从字典中选择项目，例如 ["messages"] 或 ["success"]。

你还可以通过 data["success"] 访问 success 键的值。它的值是布尔值 True ❷。我不完全确定这是什么意思，但我猜测 success 键是从乌克兰研究人员用来导出这些来自 RocketChat 的聊天信息的系统中遗留下来的，表明数据导出成功且没有错误。

我加载此代码的文件包含了 604 条不同的聊天消息，每条消息都有自己的字典，它们是在 2022 年 2 月 24 日通过 Conti 的 #general RocketChat 频道发送的。我通过使用 len() 函数来测量列表的长度，从而发现这个列表有 604 个项目，代码如下：

```
>>> **len(data["messages"])**

604
```

每个聊天消息的字典包含多个键：_id、rid、msg、u、urls 等等。

你可以通过使用 for key_variable in dictionary 语法来找出这些键包含的数据类型，并且可以使用 type() 函数来确定变量的数据类型。可以尝试使用以下命令：

```
>>> **for key in data["messages"][8]:**

...     **print(f"{key}: {type(data['messages'][8][key])}")**

...
```

该命令会遍历 data["messages"][8] 字典，并将每个键存储在 key 变量中。然后，使用 print() 函数和 f-string，它会显示该键（key）及存储在该键中的数据类型，输出如下所示：

```
_id: <class 'str'>

rid: <class 'str'>

msg: <class 'str'>

ts: <class 'str'>

u: <class 'dict'>

urls: <class 'list'>

mentions: <class 'list'>

channels: <class 'list'>

md: <class 'list'>

_updatedAt: <class 'str'>
```

在输出中，_id、rid、msg、ts 和 _updatedAt 键的值都是字符串。u 键的值是一个字典。urls、mentions、channels 和 md 键的值是列表。

你可以通过 data['messages'][8][key] 获取该键的值。记住，在字典中获取键的值时，需要将键放在方括号内。在这种情况下，键本身存储在 key 变量中，因此可以通过将 key 放入方括号中来获取其值。然后，为了找出该数据类型，只需将值传递给 type() 函数。

#### 选择字典和列表中的值

在处理数据集时，你经常会遇到这样的结构：一堆字典和列表，你需要理解其中的内容。能够选择你需要的精确值是一项重要的技能。为了练习浏览字典和列表，仔细查看这些键中的任意一个键的值，使用以下命令来查看 md 键的值：

```
>>> **print(data["messages"][8]["md"])**
```

在输出中，你可以看出这个值是一个列表，因为它被方括号括起来：

```
[{'type': 'PARAGRAPH', 'value': [{'type': 'PLAIN_TEXT', 'value': 'Некоторые американские

сенаторы предлагают помимо соцсетей блокировать в России ещё и PornHub!'}]}]
```

这个列表的唯一项目是一个字典，字典由大括号包围。字典有一个 type 键，其值为 PARAGRAPH，还有一个 value 键。value 的值是另一个包含一个项目的列表；该项目本身包含 type 和 value 键，其中 type 的值为 PLAIN_TEXT。

这些数据结构可以包含任意数量的子列表和子字典。为了选择特定的值，在 data 变量后继续添加包含索引（如果是列表）或键（如果是字典）的方括号，直到找到你需要的值。例如，使用以下命令可以访问外部字典中外部列表内另一个 value 键内的内部字典中的内部列表中的 value 键的值：

```
>>> **print(data["messages"][8]["md"][0]["value"][0]["value"])**
```

你已经知道 data["messages"][8] 是一个表示聊天信息的字典。为了查找该字典中 md 键的值，你需要在命令中包含 ["md"]。从 列表 8-2 中检查结构可以看出，这是一个包含一个项目的列表，因此添加 [0] 选择该项目。这个项目是一个字典，你可以通过添加 ["value"] 来选择它的 value 键的值。这个项目又是一个包含一个项目的列表，所以你再次添加 [0] 来选择那个项目。这个项目是另一个字典，因此你可以通过再添加一个 ["value"] 来选择最终内部 value 键的值。

你应该得到以下输出：

```
Некоторые американские сенаторы предлагают помимо соцсетей блокировать в России ещё и PornHub!
```

这条你刚刚显示的消息的内容是：“一些美国参议员建议在俄罗斯封锁 PornHub 以及社交网络！” 这条消息发布在俄罗斯开始入侵乌克兰后不久，美国和欧洲的领导人立即开始对俄罗斯实施经济制裁。入侵乌克兰后，俄罗斯政府屏蔽了从俄罗斯互联网访问 Twitter 和 Facebook。随后有传言称，美国著名色情网站 PornHub 将封锁俄罗斯用户的访问（尽管这并没有发生）。同一个用户在发布完第一条消息后，跟进发布了“就是这样，我们完了”，然后又发布了“他们会夺走我们最后的快乐！”

#### 分析存储在字典和列表中的数据

每当我处理任何类型的结构化数据时，我发现自己总是在遍历一个字典列表并选择特定的数据片段。只要你理解数据结构，你就可以编写自己的类似代码，快速提取相关信息，无论你正在处理什么数据集。例如，你可能希望以 timestamp username: message 的格式查看聊天记录，以隐藏不重要的数据部分，这样你就可以直接将相关部分复制粘贴到像 DeepL 或 Google 翻译这样的机器翻译系统中。运行以下命令，以该格式显示 data["messages"] 中的所有消息：

```
>>> **for message in data["messages"]:**

...     **print(f"{message['ts']} {message['u']['username']}: {message['msg']}")**

...
```

你应该得到以下输出：

```
`--snip--`

2022-02-24T22:02:49.448Z thomas: последние радости у нас заберут

2022-02-24T22:02:44.463Z thomas: ну все, приплыли)

2022-02-24T22:02:38.276Z thomas: Некоторые американские сенаторы предлагают помимо соцсетей

блокировать в России ещё и PornHub!

2022-02-24T22:00:00.347Z thomas:

2022-02-24T21:58:56.152Z rags: угу :(

`--snip--`
```

由于 data["messages"] 是一个列表，每次这个命令中的 for 循环执行时，它会将 message 变量更新为该列表中的不同项。在这种情况下，每一项都是一个不同的字典。在 for 循环内，print() 函数会显示三个值：时间戳 (message['ts'])，用户名 (message['u']['username']) 和消息本身 (message['msg'])。

你可以修改这个命令，显示你希望从每条消息中提取的任何信息。也许你更感兴趣的是用户的 ID，而不是他们的用户名。在这种情况下，你可以显示 message['u']['_id']。

之前的输出显示了刚才讨论的关于 PornHub 的相同消息，还显示了一条来自另一个用户 rags 在此之前发布的消息。如果你只对查看 rags 发布的消息感兴趣，可以通过运行以下命令来查看：

```
>>> **for message in data["messages"]:**

...     **if message["u"]["username"] ==** **"rags":**

...         **print(f"{message['ts']} {message['u']['username']}: {message['msg']}")**

...
```

这段代码与之前的示例类似。一个 for 循环遍历 data["messages"] 中的每条消息，然后一个 print() 语句显示该消息中的特定信息。不过，这次每个循环还包含一个 if 语句。每次代码找到一条新消息时，它会检查用户名是否为 rags，如果是，则显示该消息。否则，它会继续处理下一条消息。你应该会看到以下输出：

```
2022-02-24T22:08:49.684Z rags: давай бро спокойной ночи

2022-02-24T22:03:50.131Z rags: сча посмотрю спасиб =)

2022-02-24T21:58:56.152Z rags: угу :(

`--snip--`
```

最后，假设你想找出每个人发布了多少条消息，也许是为了找出当天在 #general 聊天室中最活跃的用户。最简单的办法是创建一个新的空字典，然后编写代码填充它。运行以下命令来创建一个名为 user_posts 的空字典：

```
>>> **user_posts = {}**
```

这个字典中的键将是用户名，值将是该用户的帖子数量。使用以下代码填充 user_posts 字典：

```
>>> **for message in data["messages"]:**

...     **username = message["u"]["username"]**

...     **if username not in user_posts:**

...         **user_posts[username] = 1**

...     **else:**

...         **user_posts[username] += 1**

...

>>>
```

同样，这段代码使用 for 循环遍历这些消息。接下来，它将 username 变量定义为 message["u"]["username"]，即代码当前循环的这条消息的发布者的用户名。接着，使用一个 if 语句，代码检查该用户名是否已经是 user_posts 字典中的一个键。（它并不是在检查字符串 username 是否是一个键，而是在检查 username 变量的*值*，例如 thomas 或 rags，是否是一个键。）

如果这个用户不存在于user_posts字典中，程序会向该字典添加一个键，并将该键的值设置为1，对应的代码行是user_posts[username] = 1。否则，程序会将该值加 1，对应的代码行是user_posts[username] += 1。当for循环完成时，user_posts字典应已完整。键应该是所有在消息中找到的用户名，而值应是该用户的消息总数。

使用以下代码显示user_posts字典中的信息，查看你刚刚收集的数据：

```
>>> **for username in user_posts:**

...     **print(f"{username} posted {user_posts[username]} times")**

...
```

你应该得到以下输出：

```
weldon posted 64 times

patrick posted 62 times

rags posted 38 times

thomas posted 58 times

ryan posted 2 times

kermit posted 151 times

biggie posted 39 times

stanton posted 12 times

angelo posted 102 times

Garfield posted 61 times

jaime posted 2 times

grem posted 5 times

jefferson posted 1 times

elijah posted 6 times

chad posted 1 times
```

这些是 2022 年俄罗斯入侵乌克兰当天在 Conti 的#general 聊天室中发布的用户，在他们的 RocketChat 服务器上。用户*kermit*发布了 151 条消息，超过了其他任何用户。

在这些示例中，你遍历了数百条聊天信息，但相同的概念也可以应用于数百万或数十亿条信息，或者任何类型的数据。

在本节中，你学习了如何处理结合了字典和列表的灵活数据结构，包括如何挑选出你感兴趣的特定元素，以及如何通过循环快速遍历它们。这些技巧在你编写 Python 脚本帮助分析数据时会非常有用。

现在你已经熟悉了结合字典和列表的数据结构，是时候创建你自己的结构来绘制 BlueLeaks 中的 CSV 文件了。

### 练习 8-5：绘制 BlueLeaks 中的 CSV 文件

BlueLeaks 中的每个文件夹都包含一个被黑客攻击的执法网站的数据，形式是数百个 CSV 文件。这些文件包含 BlueLeaks 中一些最有趣的信息，比如融合中心发送给当地警察的大宗电子邮件内容，或“可疑活动报告”。在本练习中，你将构建一个数据集内容的地图。

通过手动查看不同的 BlueLeaks 文件夹，我注意到每个文件夹似乎都有一个名为*Company.csv*的文件（每个文件包含不同的内容），但只有一个文件夹*ncric*中有一个名为*911Centers.csv*的文件。显然，并不是所有 BlueLeaks 站点都有相同的数据。那么，哪些 CSV 文件在 BlueLeaks 的每个文件夹中都有，哪些文件只在一些文件夹中有，哪些则是唯一属于某个文件夹的？让我们写一个 Python 脚本来找出答案。

和大多数编程问题一样，你可以用多种方式编写脚本来回答这个问题。如果你已经对 Python 感到足够自信，愿意挑战自己，可以尝试自己编写一个脚本。否则，可以跟着这个练习一步步做。无论如何，程序必须满足以下要求：

+   使脚本接受一个名为 blueleaks_path 的参数，使用 Click 库。

+   创建一个空字典，命名为 csv_to_folders。你的脚本应该将数据填充到这个字典中。字典的键应为 CSV 文件名，值应为包含该 CSV 的 BlueLeaks 文件夹的列表。

+   遍历 blueleaks_path 中的所有文件和文件夹。对于每个文件夹，遍历它包含的所有文件。对于每个 CSV 文件，将数据添加到 csv_to_folders 字典中。

+   显示 csv_to_folders 字典的内容。

在接下来的每一步中，我将引用一段代码，解释它的工作原理，并让你按照原样运行它。然后你会在这段代码的基础上添加更多功能，并再次运行它。将代码分成小块编写是一个好习惯，频繁暂停来测试它是否按预期工作。这将帮助你及早发现并解决 bug，使调试过程变得更加简单。

#### 接受命令行参数

创建一个 *exercise-8-5.py* 文件，并输入以下 Python 模板：

```
def main():

    pass

if __name__ == "__main__":

    main()
```

接下来，不再像在练习 8-2 中那样硬编码 BlueLeaks 数据的路径，而是使用 Click 库将路径作为命令行参数 blueleaks_path 传递。为此，对你的代码进行以下修改（新增的语法部分已加粗）：

```
**import click**

**@click.command()**

**@click.argument("blueleaks_path")**

def main(**blueleaks_path**):

    **"""Map out the CSVs in BlueLeaks"""**

    **print(f"blueleaks_path is: {blueleaks_path}")**

if __name__ == "__main__":

    main()
```

这段代码修改了模板，导入了 click 模块，在 main() 函数前添加了正确的装饰器，向 main() 函数添加了 blueleaks_path 参数，并为 main() 函数添加了一个简单的文档字符串，这样在运行脚本时使用 --help 会更加有用。最后，它还包含一行代码来显示 blueleaks_path 的值，方便你确认代码是否正常工作。

尝试运行你的脚本并加上--help，查看帮助文本是否有效，并传入blueleaks_path的值，看看参数是否成功传递到main()函数：

```
micah@trapdoor chapter-8 % **python3 exercise-8-5.py --help**

Usage: exercise-8-4.py [OPTIONS] BLUELEAKS_PATH

  Map out the CSVs in BlueLeaks

Options:

  --help  Show this message and exit.

micah@trapdoor chapter-8 % **python3 exercise-8-5.py test-path**

blueleaks_path is: test-path
```

如果你的输出是这样的，说明到目前为止一切正常。

#### 遍历 BlueLeaks 文件夹

现在你可以使用blueleaks_path命令行参数了，接下来请修改代码，使其能够遍历在该路径下找到的所有文件夹：

```
import click

**import os**

@click.command()

@click.argument("blueleaks_path")

def main(blueleaks_path):

    """Map out the CSVs in BlueLeaks"""

    **for folder in os.listdir(blueleaks_path):**

        **blueleaks_folder_path = os.path.join(blueleaks_path, folder)**

        **if os.path.isdir(blueleaks_folder_path):**

            **print(f"folder: {folder}, path: {blueleaks_folder_path}")**

if __name__ == "__main__":

    main()
```

首先，你需要导入os模块，以便能够使用os.listdir()函数列出*BlueLeaks-extracted*文件夹中的所有文件。在main()函数中，使用for循环遍历os.listdir(blueleaks_path)的返回值，即位于<sup class="SANS_TheSansMonoCd_W5Regular_11">blueleaks_path文件夹中的文件名列表。

在循环中，代码将blueleaks_folder_path定义为当前循环中对应的 BlueLeaks 文件夹路径。例如，如果blueleaks_path的值是*/Volumes/datasets/BlueLeaks-extracted*，并且在此时for循环中的folder的值是*icefishx*，那么blueleaks_folder_path的值将是*/Volumes/datasets/BlueLeaks-extracted/icefishx*。

你需要查看*BlueLeaks-extracted*文件夹中的子文件夹，而不是文件夹中的文件。如果该文件夹中有文件，你需要跳过它们。为了满足这些要求，代码中包含了一个if语句，用于检查blueleaks_folder_path是否实际上是一个文件夹。最后，代码会显示当前的folder和blueleaks_folder_path的值。

再次运行你的脚本。这次，传入*BlueLeaks-extracted*文件夹的实际路径：

```
micah@trapdoor chapter-8 % **python3 exercise-8-5.py** **`/Volumes/datasets/BlueLeaks-extracted`**

folder: bostonbric, path: /Volumes/datasets/BlueLeaks-extracted/bostonbric

folder: terrorismtip, path: /Volumes/datasets/BlueLeaks-extracted/terrorismtip

folder: ociac, path: /Volumes/datasets/BlueLeaks-extracted/ociac

`--snip--`
```

输出应显示 folder 变量仅包含文件夹名称，如 *bostonbric*，而 blueleaks_folder_path 变量则包括该文件夹的完整路径，如 */Volumes/datasets/BlueLeaks-extracted/bostonbric*。当你在自己的计算机上运行时，可能会看到这些值的顺序与这里展示的不同。

#### 填充字典

现在你有了一个接受 blueleaks_path 作为参数的脚本，并且它会遍历该路径下的每个文件夹。添加加粗部分的代码会创建 csv_to_folders 字典，并开始用数据填充它：

```
import click

import os

@click.command()

@click.argument("blueleaks_path")

def main(blueleaks_path):

    """Map out the CSVs in BlueLeaks"""

    **csv_to_folders = {}**

    for folder in os.listdir(blueleaks_path):

        blueleaks_folder_path = os.path.join(blueleaks_path, folder)

        if os.path.isdir(blueleaks_folder_path):

            **for filename in os.listdir(blueleaks_folder_path):**

                **if filename.lower().endswith(".csv"):**

                    **if filename not in csv_to_folders:**

                        **csv_to_folders[filename] = []**

                    **csv_to_folders[filename].append(folder)**

if __name__ == "__main__":

    main()
```

这个脚本的目标是映射出哪些 CSV 文件位于哪些 BlueLeaks 文件夹中。为了存储这些数据，代码在 main() 函数的顶部创建了一个空字典 csv_to_folders。接下来的步骤是填充这个字典。

代码会遍历 blueleaks_path 中的所有文件名，检查每个文件是否是文件夹。去掉前一版本代码中的 print() 语句后，这段代码添加了第二个 for 循环，用于遍历特定 BlueLeaks 文件夹中的所有文件。

在第二个 for 循环中，一个 if 语句检查文件名是否以 *.csv* 结尾。这个 if 语句会对 filename 字符串调用 lower() 方法，这将返回该字符串的小写版本。然后代码会对这个小写字符串调用 endswith() 方法，该方法返回一个布尔值，表示字符串是否以传入的子字符串结尾。如果字符串 filename 以 *.csv*、*.CSV* 或 *.cSv* 结尾，lower() 方法会将文件扩展名转换为 *.csv*，而 endswith() 会返回 True。如果 filename 以其他任何东西结尾，比如 *.docx*，那么 endswith() 会返回 False。

每次代码执行到此处的 if 语句时，表示程序已经在当前的 BlueLeaks 文件夹（称为 folder）中找到了一个 CSV 文件（称为 filename）。你希望 csv_to_folders 是一个字典，其中键是 CSV 文件名，值是文件夹列表。此代码会检查 csv_to_folders 中是否已经创建了 filename 这个键，如果没有创建，则会创建它并将其值设置为空列表 ([])。最后，在代码确认 filename 键已经创建并且是一个列表后，它将当前 folder 的值追加到该列表中。

这些最后几行有点复杂，我们来深入探讨一下。第一次脚本遇到一个 CSV 文件名（例如 *CatalogRelated.csv*）时，脚本将该键在 csv_to_folders 中的值设置为空列表。如果同样的文件名稍后在另一个 BlueLeaks 文件夹中出现，那么表达式 filename not in csv_to_folders 将评估为 False（意味着 csv_to_folders["CatalogRelated.csv"] 已经存在），因此 if 语句后面的代码将不会运行。最后，代码会将 folder（当前正在查看的 BlueLeaks 文件夹的名称）追加到包含该文件名的文件夹列表中。

暂停一下，尝试运行到目前为止的脚本：

```
micah@trapdoor chapter-8 % **python3 exercise-8-5.py** **`/Volumes/datasets/BlueLeaks-extracted`**
```

这应该需要一点时间，但不会显示任何输出，因为你还没有在任何地方使用 print() 函数。代码只是创建了 csv_to_folders 字典，并填充了数据。

#### 显示输出

当前版本的脚本运行完毕时，csv_to_folders 字典应该包含完整的 CSV 文件名，并将其映射到找到它们的 BlueLeaks 站点。以下代码应该能显示程序找到的内容：

```
import click

import os

@click.command()

@click.argument("blueleaks_path")

def main(blueleaks_path):

    """Map out the CSVs in BlueLeaks"""

    csv_to_folders = {}

    for folder in os.listdir(blueleaks_path):

        blueleaks_folder_path = os.path.join(blueleaks_path, folder) if os.path.isdir(blueleaks_folder_path):

            for filename in os.listdir(blueleaks_folder_path):

                if filename.lower().endswith(".csv"):

                    if filename not in csv_to_folders:

                        csv_to_folders[filename] = []

                    csv_to_folders[filename].append(folder)

    **for filename in csv_to_folders:**

        **print(f"{len(csv_to_folders[filename])} folders | {filename}")**

if __name__ == "__main__":

    main()
```

在加粗的代码中，循环遍历了 csv_to_folders 中的所有键（每个键都是一个 CSV 文件名），然后显示包含该文件的 BlueLeaks 文件夹的数量 (len(csv_to_folders[filename])) 和文件名本身。

你可以在[*https://<wbr>github<wbr>.com<wbr>/micahflee<wbr>/hacks<wbr>-leaks<wbr>-and<wbr>-revelations<wbr>/blob<wbr>/main<wbr>/chapter<wbr>-8<wbr>/exercise<wbr>-8<wbr>-5<wbr>.py*](https://github.com/micahflee/hacks-leaks-and-revelations/blob/main/chapter-8/exercise-8-5.py)找到这个最终的脚本。当你运行它时，输出应该如下所示：

```
micah@trapdoor chapter-8 % **python3 exercise-8-5.py** **`/Volumes/datasets/BlueLeaks-extracted`**

161 folders | CatalogRelated.csv

161 folders | Blog.csv

161 folders | EmailBuilderOptions.csv

`--snip--`

1 folders | HIDTAAgentCategory.csv

1 folders | Lost.csv

1 folders | AgencyContacts.csv
```

由于这个脚本在每行输出的开头显示了文件夹的数量，你可以将输出通过管道传递到sort -n，以按升序对其进行数字排序，像这样：

```
micah@trapdoor chapter-8 % **python3 exercise-8-5.py** **`/Volumes/datasets/BlueLeaks-extracted`** **| sort**

**-n**

1 folders | 1Cadets.csv

1 folders | 1Mentors.csv

1 folders | 1Unit.csv

`--snip--`

161 folders | VideoDownload.csv

161 folders | VideoHistory.csv

161 folders | VideoOptions.csv
```

大多数 CSV 文件位于一个文件夹或所有 161 个文件夹中。然而，也有一些例外：*Donations.csv* 应该位于 10 个文件夹中，*SARs.csv* 应该位于 25 个文件夹中，等等。手动查找这些信息可能需要花费你几个小时的繁琐工作。

到目前为止，你已经学习了如何在 Python 中浏览文件系统的基础知识。你已经看到了如何使用os.listdir()循环遍历文件夹，如何使用os.walk()遍历整个文件夹结构，以及如何查找关于你找到的文件和文件夹的信息。在下一节中，你将学习如何实际读取你找到的文件内容，并自己创建新文件。

### 读取和写入文件

为了继续学习本书的内容，你还需要掌握一个 Python 中的重要概念：如何读取和写入文件。在进行数据调查时，你几乎肯定需要读取文件的内容，特别是 CSV 和 JSON 文件。你也可能希望能够创建新文件，例如通过计算一些你自己的数据并将其保存到电子表格中。在本节中，你将学习如何打开文件并写入或读取内容。

在编程中，要操作文件，你首先需要打开它并指定*模式*——也就是你是打算*读取*文件，还是*写入*文件。要打开一个现有的文件并访问其内容，使用模式 r 来进行读取。要创建一个新文件并将数据写入其中，使用模式 w 来进行写入。

#### 打开文件

为了准备处理文件，无论是写入还是读取，你使用 Python 内置函数open()。要打开文件进行读取，你可以使用以下代码：

```
with open("`some_file.txt`", "r") as f:

    text = f.read()
```

这段代码使用了 with 语句，告诉 Python 在 open() 函数执行完毕后，应该将变量 f 设置为该函数的返回值。f 变量是一个 *文件对象*，一种允许你读取或写入文件数据的变量类型。open() 函数的第一个参数是路径，第二个参数是模式，在这个例子中是 "r"，表示读取模式。

在 with 语句之后的代码块中，你可以调用方法操作 f 来与文件进行交互。例如，f.read() 将读取文件中的所有数据并返回——在此情况下，将其存储在 text 变量中。

要打开一个文件进行写入，你需要将模式设置为 "w"，如下所示：

```
with open("output.txt", **"w"**) as f:

    f.write("hello world")
```

open() 函数返回文件对象 f。要向文件中写入数据，可以使用 f.write() 方法。在这里，这段代码打开了一个名为 *output.txt* 的文件，并将字符串 hello world 写入其中。

在接下来的两个部分中，你将学习更多关于使用 f.write() 向文件写入，以及使用 f.read() 和 f.readlines() 从文件读取的内容。

#### 写入文件的行

文本文件由一系列独立的字符组成。考虑一个包含以下内容的文本文件：

```
Hello World

Hola Mundo
```

你也可以将整个文件的内容表示为一个 Python 字符串：

```
"Hello World\nHola Mundo\n"
```

字符串的第一个字符是 H，接着是 e，然后是 l，依此类推。第 12 个字符（包括空格），\n，是一个特殊字符，称为 *换行符*，表示行与行之间的断开。与 shell 脚本一样，反斜杠是 Python 中的转义字符，因此反斜杠后跟其他字符表示一个特殊字符。

换行符用于将行写入文件。试着在你的 Python 解释器中运行这些命令：

```
>>> **with open("output.txt", "w") as f:**

...     **f.write("Hello World\n")**

...     **f.write("Hola Mundo\n")**

...

12

11
```

输出中的 12 和 11 代表写入的字节数。第一次 f.write() 调用写入了 12 个字节，因为字符串 Hello World 占用 11 个字节的内存：它有 11 个字符，再加上 1 个换行符。第二次调用写入了 11 个字节，因为 Hola Mundo 占用 10 个字节的内存，再加上 1 个换行符。

在你的终端中，使用以下命令查看你刚刚写入的文件：

```
micah@trapdoor ~ % **cat output.txt**

Hello World

Hola Mundo
```

如果你写的是相同的代码，但没有换行符，输出将会是 Hello WorldHola Mundo，没有换行符。

#### 从文件中读取行

运行以下命令来读取你刚刚创建的文件：

```
>>> **with open("output.txt", "r") as f:**

...     **text = f.read()**

...
```

这段代码从文件中读取所有数据，并将其保存在字符串 text 中。实际上，这看起来可能很熟悉：在本章的“探索 Python 中充满数据的字典和列表”一节中，我们使用类似的代码将泄漏的 Conti 聊天记录加载到 Python 字典中。

由于将文本文件拆分成多行非常常见，文件对象还提供了一个方便的方法，叫做 readlines()。与其将所有数据一次性读取到文件中，它只会一次读取一行，并且你可以在 for 循环中遍历每一行。通过运行以下命令试试看：

```
>>> **with open("/tmp/output.txt", "r") as f:**

...     **for line in f.readlines():**

...         **print(line)**

...

Hello World

Hola Mundo
```

这段代码打开文件进行读取，然后循环遍历文件中的每一行。每一行被存储在变量 line 中，然后通过 print() 函数显示出来。因为每次循环中的 line 变量以 \n 结尾（例如，第一行是 Hello World\n，而不是 Hello World），而且 print() 函数会自动添加一个额外的 \n，所以输出每行之后都会显示额外的硬回车符。

如果你不想显示这些额外的换行符，可以使用 strip() 方法去掉字符串开头和结尾的任何空白（空格、制表符或换行符）。运行相同的代码，但这次去掉每一行的换行符：

```
>>> **with open("/tmp/output.txt", "r") as f:**

...     **for line in f.readlines():**

...         **line = line.strip()**

...         **print(line)**

...

Hello World

Hola Mundo
```

在接下来的练习中，你将练习如何在 Python 中读取和写入文件的基础知识。

### 练习 8-6：实践读取和写入文件

在练习 7-5 中，你写了一个函数，将字符串转换为交替大小写版本，像 This book is amazing 转换为 ThIs bOoK Is aMaZiNg。为了练习你新学的读取和写入文件技能，在本练习中，你将编写一个脚本，创建整个文本文件中所有文本的交替大小写版本。

如果你想挑战自己，可以尝试编写一个脚本来满足以下要求：

+   接受两个参数，input_filename 和 output_filename，使用 Click

+   打开文件 input_filename 进行读取，并将其内容加载到字符串 text 中

+   打开文件 output_filename 进行写入，并将 text 的交替大小写版本保存到该新文件中

否则，跟随我对以下代码的解释，这段代码实现了这个**极其有用**的命令行程序。

从复制你在练习 7-5 中写的 alternating_caps() 函数开始，粘贴到一个新的 Python 脚本中，命名为 *exercise-8-6.py*。接下来，做出这里加粗部分的修改（或者直接复制最终的脚本到 [*https://<wbr>github<wbr>.com<wbr>/micahflee<wbr>/hacks<wbr>-leaks<wbr>-and<wbr>-revelations<wbr>/blob<wbr>/main<wbr>/chapter<wbr>-8<wbr>/exercise<wbr>-8<wbr>-6<wbr>.py*](https://github.com/micahflee/hacks-leaks-and-revelations/blob/main/chapter-8/exercise-8-6.py)）：

```
**import click**

def alternating_caps(text):

    """Returns an aLtErNaTiNg cApS version of text"""

    alternating_caps_text = ""

    should_be_capital = True

    for character in text:

        if should_be_capital:

            alternating_caps_text += character.upper()

            should_be_capital = False

        else:

            alternating_caps_text += character.lower()

            should_be_capital = True

    return alternating_caps_text

**@click.command()**

**@click.argument("input_filename")**

**@click.argument("output_filename")**

**def main(input_filename, output_filename):**

    **"""Converts a text file to an aLtErNaTiNg cApS version"""**

    **with open(input_filename, "r") as f:**

        **text = f.read()**

    **with open(output_filename, "w") as f:**

        **f.write(alternating_caps(text))**

if __name__ == "__main__":

      main()
```

这段代码首先导入了 click 模块，用于处理参数，然后定义了 alternating_caps() 函数。同样，main() 函数是一个 Click 命令，但这次它接受两个参数，input_filename 和 output_filename。

一旦 main() 函数运行，读取和写入文件的部分就会执行。代码打开 input_filename 以供读取，并将该文件的所有内容加载到字符串 text 中。然后，它打开 output_filename 以供写入，并将该字符串的交替大小写版本保存到新文件中。它通过运行 alternating_caps(text) 来实现这一点，alternating_caps 接受 text 作为参数并返回其交替大小写版本，然后将返回值直接传递给 f.write()，将其写入文件。

为了演示这个脚本的工作原理，试着在著名的《哈姆雷特》“生存还是毁灭”独白上运行它。首先，将独白的副本保存到名为 *shakespeare.txt* 的文件中，文件内容来自 [*https://<wbr>github<wbr>.com<wbr>/micahflee<wbr>/hacks<wbr>-leaks<wbr>-and<wbr>-revelations<wbr>/blob<wbr>/main<wbr>/chapter<wbr>-8<wbr>/shakespeare<wbr>.txt*](https://github.com/micahflee/hacks-leaks-and-revelations/blob/main/chapter-8/shakespeare.txt)。以下是使用 cat 命令显示的 *shakespeare.txt* 的原始内容：

```
micah@trapdoor chapter-8 % **cat shakespeare.txt**

To be, or not to be, that is the question:

Whether 'tis nobler in the mind to suffer

The slings and arrows of outrageous fortune,

Or to take Arms against a Sea of troubles,

And by opposing end them: to die, to sleep

No more; and by a sleep, to say we end

`--snip--`
```

接下来，将该文件名传入脚本，以创建该文件的交替大小写版本。以下是我执行该操作时发生的情况：

```
micah@trapdoor chapter-8 % **python3 exercise-8-5.py shakespeare.txt shakespeare-mocking.txt**

micah@trapdoor chapter-8 % **cat shakespeare-mocking.txt**

To bE, oR NoT To bE, tHaT Is tHe qUeStIoN:

wHeThEr 'TiS NoBlEr iN ThE MiNd tO SuFfEr

tHe sLiNgS AnD ArRoWs oF OuTrAgEoUs fOrTuNe,

Or tO TaKe aRmS AgAiNsT A SeA Of tRoUbLeS,

aNd bY OpPoSiNg eNd tHeM: tO DiE, tO SlEeP

No mOrE; aNd bY A SlEeP, tO SaY We eNd

`--snip--`
```

首先，我运行了脚本，将 *shakespeare.txt* 作为 input_filename 传入，将 *shakespeare-mocking.txt* 作为 output_filename 传入。脚本本身没有输出（它不包括任何 print() 语句），但它确实创建了一个新文件。然后，我使用 cat 命令显示了新文件的内容，的确是《哈姆雷特》独白的交替大小写版本。

### 总结

恭喜你成功完成了 Python 编程基础的速成课程！你已经学会了如何通过内置和第三方 Python 模块为脚本增加额外功能。你还学会了如何使用 Click 创建自己的 CLI 程序，如何编写遍历文件系统的代码，如何使用字典和列表处理结构化数据，以及如何读取和写入文件。

在接下来的章节中，你将使用这些技能，深入挖掘各种数据集，揭示一些你否则无法发现的发现。在下一章节中，你将编写 Python 程序，遍历 BlueLeaks CSV 表格中的行，将数据转化为更易于操作的格式。你将练习将执法机关的大宗邮件内容写入文件，并且使用 Python 创建你自己的 CSV 表格。
