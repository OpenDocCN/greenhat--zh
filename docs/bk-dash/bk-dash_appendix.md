# <samp class="SANS_Dogma_OT_Bold_B_11">附录</samp>

PYTHON 基础知识</samp>

![](img/opener-img.png)

本附录的重点是快速回顾 Python 的基础知识。Dash 书籍的范围不包括完整的 Python 速成课程，我们只会介绍关键词、数据结构、控制流和函数等基础知识。您可以通过一些优秀的资源全面学习 Python，包括我们的免费电子邮件学院：[*https://<wbr>blog<wbr>.finxter<wbr>.com<wbr>/email<wbr>-academy*](https://blog.finxter.com/email-academy)。

<samp class="SANS_Dogma_OT_Bold_B_15">注意</samp>

*介绍部分使用了* Python One-Liners *(No Starch Press, 2020)* 中的代码示例和文本片段，该书是本书的一位作者所著。我们建议您阅读此书，以全面了解 Python 单行代码的含义。

## <samp class="SANS_Futura_Std_Bold_B_11">安装和入门</samp>

如果您尚未安装 Python，则需要在计算机上设置 Python。由于 Python 不断发展，我们将保持这些信息的概括。

1.  首先，请访问官方 Python 网站[*https://<wbr>www<wbr>.python<wbr>.org<wbr>/downloads*](https://www.python.org/downloads)，下载适合你操作系统的最新版本的 Python。

2.  在你的计算机上运行安装程序。根据版本和操作系统的不同，你应该会看到一个类似于图 A-1 的对话框。确保点击框以将 Python 添加到 PATH 中，以便通过 Python 访问计算机上的任何目录。

![](img/FigureA-1.png)

<samp class="SANS_Futura_Std_Book_Oblique_I_11">图 A-1：安装 Python 弹出窗口</samp>

3.  通过在命令行（Windows）、终端（macOS）或 shell（Linux）中运行以下命令，检查您的 Python 安装是否正常运行：

<samp class="SANS_TheSansMonoCd_W5Regular_11">$</samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">python—version</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">Python 3.</samp><samp class="SANS_TheSansMonoCd_W5Regular_Italic_I_11">x.x</samp>

<samp class="SANS_Dogma_OT_Bold_B_15">注意</samp>

*美元符号（$）只是一个提示，告诉你在终端或代码 shell 中运行接下来的代码。粗体文本是你应输入的命令。*

恭喜！您已在计算机上安装了 Python。您可以开始使用系统内置的 IDLE 编辑器编写自己的程序。只需在操作系统中搜索 *IDLE* 并打开该程序即可。

作为第一个程序，将以下命令输入到您的 shell 中：

<samp class="SANS_TheSansMonoCd_W7Bold_B_11">print('hello world!')</samp>

Python 将解释您的命令并将所需的文字打印到您的 shell（参见图 A-2）：

<samp class="SANS_TheSansMonoCd_W5Regular_11">hello world!</samp>

![](img/FigureA-2.png)

<samp class="SANS_Futura_Std_Book_Oblique_I_11">图 A-2：Python 中的 hello world 程序</samp>

与 Python 解释器进行交互式双向通信的这种模式称为*交互模式*。它的优点是能够立即反馈。然而，编程计算机最令人兴奋的结果是自动化：编写一次程序并多次运行它。

让我们从一个简单的程序开始，每次运行时都用你的名字打招呼。你可以保存该程序，并在任何时候重新运行。此类程序称为*脚本*，你可以将它们保存为 *.py* 后缀文件，如 *my_first_program.py*，以保存为 Python 文件。

你可以通过 IDLE shell 的菜单创建脚本，如 图 A-3 所示。

![](img/FigureA-3.png)

<samp class="SANS_Futura_Std_Book_Oblique_I_11">图 A-3：创建你自己的模块</samp>

点击 **文件** ► **新建文件**，并将以下代码复制粘贴到你的新文件中：

<samp class="SANS_TheSansMonoCd_W5Regular_11">name = input("What's your name?")</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print('hello' + name)</samp>

将文件保存为 *hello.py*，存放在桌面或其他任何位置。你当前的脚本应该类似于 图 A-4。

![](img/FigureA-4.png)

<samp class="SANS_Futura_Std_Book_Oblique_I_11">图 A-4：一个接受用户输入并将响应打印到标准输出的程序</samp>

现在，让我们开始一些操作：点击 **运行** ► **运行模块**。Python 程序将在交互式 shell 中开始执行，无需你逐行输入。它会逐行执行代码文件。第一行会提示你输入名字，并等待你输入。第二行则会接收你的名字并将其打印到 shell 中。图 A-5 展示了程序的执行效果。

![](img/FigureA-5.png)

<samp class="SANS_Futura_Std_Book_Oblique_I_11">图 A-5：程序在图 A-4 中的示例执行</samp>

## <samp class="SANS_Futura_Std_Bold_B_11">数据类型</samp>

现在你已经看到一个 Python 程序的运行，我们将回顾一些基本数据类型。

<samp class="SANS_Futura_Std_Bold_Condensed_Oblique_I_11">布尔值</samp>

布尔数据类型表示的是关键字 <samp class="SANS_TheSansMonoCd_W5Regular_11">False</samp> 或 <samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp>。在 Python 中，布尔和整数数据类型紧密相关，因为布尔类型在内部使用整数值：<samp class="SANS_TheSansMonoCd_W5Regular_11">False</samp> 表示整数 0，<samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp> 表示整数 1。布尔值通常用于断言或作为比较的结果。以下代码片段展示了这两个布尔关键字的使用示例：

<samp class="SANS_TheSansMonoCd_W5Regular_11">X = 1 > 2</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(x)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11"># False</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">y = 2 > 1</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(y)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11"># True</samp>

在评估给定的表达式后，变量 <samp class="SANS_TheSansMonoCd_W5Regular_11">x</samp> 代表值 <samp class="SANS_TheSansMonoCd_W5Regular_11">False</samp>，变量 <samp class="SANS_TheSansMonoCd_W5Regular_11">y</samp> 代表值 <samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp>。布尔值允许我们创建条件执行的代码，因此它们在处理数据时非常重要，因为它们可以让我们在使用某个值之前检查它是否超过阈值（有关基于阈值的数据分类，请参见 第七章 中的 SVM Explorer 应用）。

布尔值有几个主要运算符，表示基本的逻辑运算符：<samp class="SANS_TheSansMonoCd_W5Regular_11">and</samp>、<samp class="SANS_TheSansMonoCd_W5Regular_11">or</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">not</samp>。关键字 <samp class="SANS_TheSansMonoCd_W5Regular_11">and</samp> 在表达式 <samp class="SANS_TheSansMonoCd_W5Regular_11">x and y</samp> 中，当 <samp class="SANS_TheSansMonoCd_W5Regular_11">x</samp> 为 <samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp> 且 <samp class="SANS_TheSansMonoCd_W5Regular_11">y</samp> 为 <samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp> 时，评估为 <samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp>。如果其中只有一个为 <samp class="SANS_TheSansMonoCd_W5Regular_11">False</samp>，则整个表达式为 <samp class="SANS_TheSansMonoCd_W5Regular_11">False</samp>。

关键字 <samp class="SANS_TheSansMonoCd_W5Regular_11">or</samp> 在表达式 <samp class="SANS_TheSansMonoCd_W5Regular_11">x or y</samp> 中，如果 <samp class="SANS_TheSansMonoCd_W5Regular_11">x</samp> 为 <samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp> *或* <samp class="SANS_TheSansMonoCd_W5Regular_11">y</samp> 为 <samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp> *或* 两者都是 <samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp>，则评估为 <samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp>。如果其中只有一个为 <samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp>，则整个表达式为 <samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp>。

关键字 <samp class="SANS_TheSansMonoCd_W5Regular_11">not</samp> 在表达式 <samp class="SANS_TheSansMonoCd_W5Regular_11">not x</samp> 中，当 <samp class="SANS_TheSansMonoCd_W5Regular_11">x</samp> 为 <samp class="SANS_TheSansMonoCd_W5Regular_11">False</samp> 时，评估为 <samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp>。考虑以下使用每个布尔运算符的 Python 代码示例：

<samp class="SANS_TheSansMonoCd_W5Regular_11">x, y = True, False</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(x or y)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11"># True</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(x and y)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11"># True</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(not y)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11"># True</samp>

通过使用这三种操作符——<samp class="SANS_TheSansMonoCd_W5Regular_11">and</samp>、<samp class="SANS_TheSansMonoCd_W5Regular_11">or</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">not</samp>——你可以表达你所需要的所有逻辑表达式。

布尔运算符按优先级排序。运算符 <samp class="SANS_TheSansMonoCd_W5Regular_11">not</samp> 具有最高优先级，其次是运算符 <samp class="SANS_TheSansMonoCd_W5Regular_11">and</samp>，最后是运算符 <samp class="SANS_TheSansMonoCd_W5Regular_11">or</samp>。请考虑以下示例：

<samp class="SANS_TheSansMonoCd_W5Regular_11">x, y = True, False</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(x and not y)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11"># True</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(not x and y or x)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11"># True</samp>

我们将变量 <samp class="SANS_TheSansMonoCd_W5Regular_11">x</samp> 设置为 <samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp>，将 <samp class="SANS_TheSansMonoCd_W5Regular_11">y</samp> 设置为 <samp class="SANS_TheSansMonoCd_W5Regular_11">False</samp>。当调用 <samp class="SANS_TheSansMonoCd_W5Regular_11">not x and y or x</samp> 时，Python 会将其解释为 <samp class="SANS_TheSansMonoCd_W5Regular_11">((not x) and y) or x</samp>，这与例如 <samp class="SANS_TheSansMonoCd_W5Regular_11">(not x) and (y or x)</samp> 不同。作为练习，弄明白*为什么*。

<samp class="SANS_Futura_Std_Bold_Condensed_Oblique_I_11">数值类型</samp>

两种最重要的数值数据类型是整数和浮点数。*整数*是没有浮动精度的正数或负数（例如，<samp class="SANS_TheSansMonoCd_W5Regular_11">3</samp>）。*浮点数*是具有浮动精度的正数或负数（例如，<samp class="SANS_TheSansMonoCd_W5Regular_11">3.14159265359</samp>）。Python 提供了多种内置的数值操作，还可以将不同的数值数据类型之间进行转换。以下是几个算术操作的示例。首先，我们将创建一个值为 3 的 <samp class="SANS_TheSansMonoCd_W5Regular_11">x</samp> 变量和一个值为 2 的 <samp class="SANS_TheSansMonoCd_W5Regular_11">y</samp> 变量：

<samp class="SANS_TheSansMonoCd_W5Regular_11">>>></samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">x, y</samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">=</samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">3, 2</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">>>></samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">x</samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">+</samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">y</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">5</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">>>></samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">x</samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">—</samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">y</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">1</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">>>></samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">x * y</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">6</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">>>></samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">x / y</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">1.5</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">>>></samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">x // y</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">1</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">>>></samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">x % y</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">1</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">>>></samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">-x</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">-3</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">>>></samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">abs(-x)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">3</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">>>></samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">int(3.9)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">3</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">>>></samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">float(3)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">3.0</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">>>></samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">x ** y</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">9</samp>

前四种运算分别是加法、减法、乘法和除法。<samp class="SANS_TheSansMonoCd_W5Regular_11">//</samp> 运算符执行整数除法。结果是一个整数值，会向下舍入到较小的整数（例如，<samp class="SANS_TheSansMonoCd_W5Regular_11">3 // 2</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">==</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">1</samp>）。<samp class="SANS_TheSansMonoCd_W5Regular_11">%</samp> 运算是 *取模运算*，它仅返回除法的余数。减号运算符 <samp class="SANS_TheSansMonoCd_W5Regular_11">–</samp> 将值转换为负数。<samp class="SANS_TheSansMonoCd_W5Regular_11">abs()</samp> 给出绝对值（即值作为非负数）。<samp class="SANS_TheSansMonoCd_W5Regular_11">int()</samp> 将值转换为整数，舍弃小数点后的任何数字。<samp class="SANS_TheSansMonoCd_W5Regular_11">float()</samp> 将给定值转换为浮动点数。双星号 <samp class="SANS_TheSansMonoCd_W5Regular_11">**</samp> 表示乘方运算。运算符优先级与您在学校学到的一样：括号优先于指数运算，指数优先于乘法，乘法优先于加法，依此类推。

<samp class="SANS_Futura_Std_Bold_Condensed_Oblique_I_11">字符串</samp>

Python 字符串是字符序列。字符串是不可变的，因此一旦创建就不能更改；如果需要更改字符串，必须创建一个新的字符串。字符串通常是引号内的文本（包括数字）：<samp class="SANS_TheSansMonoCd_W5Regular_11">"this is a string"</samp>。以下是创建字符串的五种最常见方法：

单引号：<samp class="SANS_TheSansMonoCd_W5Regular_11">'Yes'</samp>

双引号：<samp class="SANS_TheSansMonoCd_W5Regular_11">"Yes"</samp>

三引号（用于多行字符串）：<samp class="SANS_TheSansMonoCd_W5Regular_11">'''</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">Yes'''</samp> 或 <samp class="SANS_TheSansMonoCd_W5Regular_11">" ""Yes" ""</samp>

字符串方法：<samp class="SANS_TheSansMonoCd_W5Regular_11">str(yes)</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">==</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">'yes'</samp> 是 <samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp>

字符串连接：<samp class="SANS_TheSansMonoCd_W5Regular_11">'Py'</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">+</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">'thon'</samp> 变为 <samp class="SANS_TheSansMonoCd_W5Regular_11">'Python'</samp>

要在字符串中使用空白字符，您需要明确地指定它们。要在字符串内开始新的一行，请使用换行符 <samp class="SANS_TheSansMonoCd_W5Regular_11">'\n'</samp>。要添加一个制表符的空格，请使用制表符字符 <samp class="SANS_TheSansMonoCd_W5Regular_11">'\t'</samp>。

字符串也有自己的一组方法。<samp class="SANS_TheSansMonoCd_W5Regular_11">strip()</samp> 方法移除字符串的前导和尾随空白字符，包括空格、制表符和换行符：

<samp class="SANS_TheSansMonoCd_W5Regular_11">y = " This is lazy\t\n "</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(y.strip())</samp>

结果更加整洁：

<samp class="SANS_TheSansMonoCd_W5Regular_11">'This is lazy'</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">lower()</samp> 方法将整个字符串转为小写：

<samp class="SANS_TheSansMonoCd_W5Regular_11">print("DrDre".lower())</samp>

我们得到了：

<samp class="SANS_TheSansMonoCd_W5Regular_11">'drdre'</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">upper()</samp> 方法将整个字符串转换为大写：

<samp class="SANS_TheSansMonoCd_W5Regular_11">print("attention".upper())</samp>

这给我们带来了：

<samp class="SANS_TheSansMonoCd_W5Regular_11">'ATTENTION'</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">startswith()</samp> 方法检查您提供的参数是否出现在字符串的开头：

<samp class="SANS_TheSansMonoCd_W5Regular_11">print("smartphone".startswith("smart"))</samp>

它返回一个布尔值：

<samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">endswith()</samp> 方法检查您提供的参数是否出现在字符串的末尾：

<samp class="SANS_TheSansMonoCd_W5Regular_11">print("smartphone".endswith("phone"))</samp>

这也返回一个布尔值：

<samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">find()</samp> 方法返回子字符串在原始字符串中第一次出现的索引：

<samp class="SANS_TheSansMonoCd_W5Regular_11">print("another".find("other"))</samp>

如下所示：

<samp class="SANS_TheSansMonoCd_W5Regular_11">Match index: 2</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">replace()</samp> 方法将第一个参数中的字符替换为第二个参数中的字符：

<samp class="SANS_TheSansMonoCd_W5Regular_11">print("cheat".replace("ch", "m"))</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">cheat</samp> 变成：

<samp class="SANS_TheSansMonoCd_W5Regular_11">meat</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">join()</samp> 方法将可迭代参数中的所有值组合起来，使用调用它的字符串作为分隔符：

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(','.join(["F", "B", "I"]))</samp>

我们得到了：

<samp class="SANS_TheSansMonoCd_W5Regular_11">F,B,I</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">len()</samp> 方法返回字符串中的字符数量，包括空白字符：

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(len("Rumpelstiltskin"))</samp>

这给我们带来了：

<samp class="SANS_TheSansMonoCd_W5Regular_11">String length: 15</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">in</samp> 关键字在与字符串操作数一起使用时，会检查一个字符串是否出现在另一个字符串中：

<samp class="SANS_TheSansMonoCd_W5Regular_11">print("ear" in "earth")</samp>

这也返回一个布尔值：

<samp class="SANS_TheSansMonoCd_W5Regular_11">Contains: True</samp>

这个非排他性的字符串方法列表表明，Python 的字符串数据类型是灵活且强大的，你可以利用内置的 Python 功能解决许多常见的字符串问题。

## <samp class="SANS_Futura_Std_Bold_B_11">控制流</samp>

让我们深入了解一些让我们的代码做出决策的编程逻辑。算法就像一个烹饪食谱：如果这个食谱只是一个顺序的命令列表——将水倒入锅中，加入盐，加入米，倒掉水，最后上桌——你可能几秒钟内就完成了步骤，却得到了一碗未煮熟的米饭。我们需要根据不同的情况作出不同的反应：只有*当*米饭变软时，才从锅中去除水；*当*水开始沸腾时，才把米饭放入锅中。根据不同条件做出不同响应的代码被称为*条件执行*代码。在 Python 中，条件执行的关键字有 <samp class="SANS_TheSansMonoCd_W5Regular_11">if</samp>、<samp class="SANS_TheSansMonoCd_W5Regular_11">else</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">elif</samp>。

这是一个比较两个数字的基本示例：

<samp class="SANS_TheSansMonoCd_W5Regular_11">half_truth = 21</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">if 2 * half_truth == 42:</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print('Truth!')</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">else:</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print('Lie!')</samp>

这将打印：

<samp class="SANS_TheSansMonoCd_W5Regular_11">Truth!</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">if</samp> 条件 <samp class="SANS_TheSansMonoCd_W5Regular_11">2 * half_truth</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">==</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">42</samp> 会生成一个结果，结果可以是 <samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp> 或 <samp class="SANS_TheSansMonoCd_W5Regular_11">False</samp>。如果表达式的结果是 <samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp>，则进入第一个分支并打印 <samp class="SANS_TheSansMonoCd_W5Regular_11">Truth!</samp>。如果表达式的结果是 <samp class="SANS_TheSansMonoCd_W5Regular_11">False</samp>，则进入第二个分支并打印 <samp class="SANS_TheSansMonoCd_W5Regular_11">Lie!</samp>。由于表达式的结果是 <samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp>，所以进入第一个分支，shell 输出为 <samp class="SANS_TheSansMonoCd_W5Regular_11">Truth!</samp>。

每个 Python 对象，如变量或列表，都有一个隐式关联的布尔值，这意味着我们可以将 Python 对象用作条件。例如，一个空列表的布尔值为 <samp class="SANS_TheSansMonoCd_W5Regular_11">False</samp>，而非空列表的布尔值为 <samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp>：

<samp class="SANS_TheSansMonoCd_W5Regular_11">lst = []</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">if lst:</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print('Full!')</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">else:</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print('Empty!')</samp>

这会打印：

<samp class="SANS_TheSansMonoCd_W5Regular_11">Empty!</samp>

如果你不需要 <samp class="SANS_TheSansMonoCd_W5Regular_11">else</samp> 分支，可以简单跳过它，当条件评估为 <samp class="SANS_TheSansMonoCd_W5Regular_11">False</samp> 时，Python 会跳过整个代码块：

<samp class="SANS_TheSansMonoCd_W5Regular_11">if 2 + 2 == 4:</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print('FOUR')</samp>

这会打印：

<samp class="SANS_TheSansMonoCd_W5Regular_11">FOUR</samp>

只有当 <samp class="SANS_TheSansMonoCd_W5Regular_11">if</samp> 条件评估为 <samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp> 时，才会打印输出。否则，不会发生任何事情。代码没有副作用，因为它会被执行流跳过。

你也可以有超过两个条件的代码。在这种情况下，你可以使用 <samp class="SANS_TheSansMonoCd_W5Regular_11">elif</samp> 语句：

<samp class="SANS_TheSansMonoCd_W5Regular_11">x = input('Your Number: ')</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">if x ==</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">'1':</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print('ONE')</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">elif x == '2':</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print('TWO')</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">elif x == '3':</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print('THREE')</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">else:</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print('MANY')</samp>

这段代码会接收你的输入并将其与字符串 <samp class="SANS_TheSansMonoCd_W5Regular_11">'1'</samp>、<samp class="SANS_TheSansMonoCd_W5Regular_11">'2'</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">'3'</samp> 进行比较。在每种情况下，都会输出不同的结果。如果输入与任何字符串都不匹配，则进入最后一个分支，输出 <samp class="SANS_TheSansMonoCd_W5Regular_11">'MANY'</samp>。

以下代码片段获取用户输入，将其转换为整数，并将其存储在变量<samp class="SANS_TheSansMonoCd_W5Regular_11">x</samp>中。然后，它测试该变量是否大于、等于或小于 3，并根据评估结果打印不同的消息。换句话说，这段代码对现实世界中不可预测的输入做出有差异的响应：

<samp class="SANS_TheSansMonoCd_W5Regular_11">x = int(input('请输入您的值: '))</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">if x > 3:</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print('Big')</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">elif x ==</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">3:</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print('Medium')</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">else:</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print('Small')</samp>

我们给出关键字<samp class="SANS_TheSansMonoCd_W5Regular_11">if</samp>，后跟一个条件，该条件决定执行路径。若条件为<samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp>，执行路径将沿着紧接着的缩进代码块中的第一分支。如果条件为<samp class="SANS_TheSansMonoCd_W5Regular_11">False</samp>，执行流会继续向下，执行以下三种情况之一：

1.  根据任意数量的<samp class="SANS_TheSansMonoCd_W5Regular_11">elif</samp>分支进一步评估其他条件

2.  如果没有满足条件的<samp class="SANS_TheSansMonoCd_W5Regular_11">if</samp>或<samp class="SANS_TheSansMonoCd_W5Regular_11">elif</samp>条件，则进入<samp class="SANS_TheSansMonoCd_W5Regular_11">else</samp>分支

3.  当没有给出<samp class="SANS_TheSansMonoCd_W5Regular_11">else</samp>分支，并且没有满足条件的<samp class="SANS_TheSansMonoCd_W5Regular_11">elif</samp>分支时，跳过整个结构

规则是，执行路径从顶部开始，向下移动，直到某个条件匹配——在这种情况下，执行相应的代码分支——或者所有条件都被探查过，但没有任何条件匹配。

这里你可以看到，将对象传入<samp class="SANS_TheSansMonoCd_W5Regular_11">if</samp>条件并像布尔值一样使用它们是可能的：

<samp class="SANS_TheSansMonoCd_W5Regular_11">if None or 0 or 0.0 or ' ' or [] or {} or set():</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print('Dead code') # 不会到达</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">if</samp> 条件计算结果为 <samp class="SANS_TheSansMonoCd_W5Regular_11">False</samp>，因此 <samp class="SANS_TheSansMonoCd_W5Regular_11">print</samp> 语句永远不会被执行。这是因为以下值会被计算为布尔值 <samp class="SANS_TheSansMonoCd_W5Regular_11">False</samp>：关键字 <samp class="SANS_TheSansMonoCd_W5Regular_11">None</samp>、整数值 <samp class="SANS_TheSansMonoCd_W5Regular_11">0</samp>、浮点值 <samp class="SANS_TheSansMonoCd_W5Regular_11">0.0</samp>、空字符串和空容器类型。表达式 <samp class="SANS_TheSansMonoCd_W5Regular_11">None or 0 or 0.0 or ' ' or [] or {} or set()</samp> 会被计算为 <samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp>，如果 Python 能隐式地将任何操作数转换为 <samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp>，但在这里并不会，因为所有这些值都会被转换为 <samp class="SANS_TheSansMonoCd_W5Regular_11">False</samp>。

## <samp class="SANS_Futura_Std_Bold_B_11">重复执行</samp>

为了允许重复执行类似的代码片段，Python 提供了两种循环类型：<samp class="SANS_TheSansMonoCd_W5Regular_11">for</samp> 循环和 <samp class="SANS_TheSansMonoCd_W5Regular_11">while</samp> 循环。我们将创建一个 <samp class="SANS_TheSansMonoCd_W5Regular_11">for</samp> 循环和一个 <samp class="SANS_TheSansMonoCd_W5Regular_11">while</samp> 循环，以不同的方式实现相同的功能：将整数 0、1 和 2 打印到 Python shell 中。

下面是 <samp class="SANS_TheSansMonoCd_W5Regular_11">for</samp> 循环：

<samp class="SANS_TheSansMonoCd_W5Regular_11">for i in [0, 1, 2]:</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print(i)</samp>

这将输出：

<samp class="SANS_TheSansMonoCd_W5Regular_11">0</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">1</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">2</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">for</samp> 循环通过声明一个循环变量 <samp class="SANS_TheSansMonoCd_W5Regular_11">i</samp>，该变量会依次取出列表 <samp class="SANS_TheSansMonoCd_W5Regular_11">[0, 1, 2]</samp> 中的所有值。然后它会打印变量 <samp class="SANS_TheSansMonoCd_W5Regular_11">i</samp>，直到列表中的值用尽。

下面是具有相似语义的 <samp class="SANS_TheSansMonoCd_W5Regular_11">while</samp> 循环版本：

<samp class="SANS_TheSansMonoCd_W5Regular_11">i = 0</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">while i < 3:</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print(i)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   i = i + 1</samp>

这也将输出：

<samp class="SANS_TheSansMonoCd_W5Regular_11">0</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">1</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">2</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">while</samp>循环只要条件满足，就会执行循环体——在我们的例子中，只要<samp class="SANS_TheSansMonoCd_W5Regular_11">i</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11"><</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">3</samp>。使用哪种方法取决于你的具体情况。通常，当你遍历一个固定数量的元素时（例如，遍历列表中的所有元素），你会使用<samp class="SANS_TheSansMonoCd_W5Regular_11">for</samp>循环；当你希望重复某个操作，直到达成某个结果时（例如，猜密码直到成功），你会使用<samp class="SANS_TheSansMonoCd_W5Regular_11">while</samp>循环。

终止循环有两种基本方式：定义一个循环条件，使其评估为<samp class="SANS_TheSansMonoCd_W5Regular_11">False</samp>，如前面的例子所示，或者在循环体内的确切位置使用关键字<samp class="SANS_TheSansMonoCd_W5Regular_11">break</samp>来停止循环。这里我们使用<samp class="SANS_TheSansMonoCd_W5Regular_11">break</samp>来退出原本会变成无限循环的情况：

<samp class="SANS_TheSansMonoCd_W5Regular_11">while True:</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   break # No infinite loop</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print('hello world')</samp>

从中我们得到：

<samp class="SANS_TheSansMonoCd_W5Regular_11">hello world</samp>

我们创建了一个<samp class="SANS_TheSansMonoCd_W5Regular_11">while</samp>循环，循环条件始终评估为<samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp>，因为循环条件<samp class="SANS_TheSansMonoCd_W5Regular_11">while True</samp>本身就固有地为<samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp>。循环在<samp class="SANS_TheSansMonoCd_W5Regular_11">break</samp>处提前结束，因此代码会继续执行并输出<samp class="SANS_TheSansMonoCd_W5Regular_11">print('hello world')</samp>。

你可能会想，如果我们不希望它一直运行，为什么还要创建一个无限循环呢？这是一个常见的做法，例如，在开发需要使用无限循环等待新网页请求并提供服务的 Web 服务器时。然而，你仍然希望能够在循环未完成时提前终止它。在 Web 服务器的例子中，如果服务器检测到遭受攻击，可能会希望停止提供文件服务。在这些情况下，你可以使用关键字<samp class="SANS_TheSansMonoCd_W5Regular_11">break</samp>，当满足某个条件时停止循环。

也可以强制 Python 解释器跳过循环中的某些部分，而不是提前结束循环。在我们的 Web 服务器示例中，你可能希望跳过恶意的 Web 请求，而不是完全停止服务器。你可以使用 <samp class="SANS_TheSansMonoCd_W5Regular_11">continue</samp> 关键字实现这一点，它会结束当前的循环迭代，并将执行流带回到循环条件的开始：

<samp class="SANS_TheSansMonoCd_W5Regular_11">while True:</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   continue</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print('43') # 死代码</samp>

这段代码将永远执行下去，而不会执行一次 <samp class="SANS_TheSansMonoCd_W5Regular_11">print</samp> 语句，因为 <samp class="SANS_TheSansMonoCd_W5Regular_11">continue</samp> 语句结束了当前的循环迭代，并将执行流带回到循环开始处，未到达 <samp class="SANS_TheSansMonoCd_W5Regular_11">print()</samp> 语句。因此，<samp class="SANS_TheSansMonoCd_W5Regular_11">print()</samp> 语句现在被视为 *死代码*：即永远不会被执行的代码。<samp class="SANS_TheSansMonoCd_W5Regular_11">continue</samp> 语句和 <samp class="SANS_TheSansMonoCd_W5Regular_11">break</samp> 语句通常只有在与 <samp class="SANS_TheSansMonoCd_W5Regular_11">if-else</samp> 语句配合使用时才有意义，像这样：

<samp class="SANS_TheSansMonoCd_W5Regular_11">while True:</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   user_input = input('请输入密码: ')</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   if user_input == '42':</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      break</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print('错误！') # 死代码</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print('恭喜，你找到了秘密密码！')</samp>

这段代码会请求一个密码，并会一直执行下去，直到用户猜到正确的密码。如果他们输入正确的密码 <samp class="SANS_TheSansMonoCd_W5Regular_11">42</samp>，程序会执行到 <samp class="SANS_TheSansMonoCd_W5Regular_11">break</samp> 语句，结束循环，并执行成功的 <samp class="SANS_TheSansMonoCd_W5Regular_11">print</samp> 语句。如果密码错误，循环会中断，执行返回到开始位置，用户需要重新尝试。以下是一个示例用法：

<samp class="SANS_TheSansMonoCd_W5Regular_11">your password: 41</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">错误！</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">your password: 21</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">错误！</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">your password: 42</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">恭喜，你找到了秘密密码！</samp>

这些是控制程序执行流程的最重要关键字。

## <samp class="SANS_Futura_Std_Bold_B_11">其他有用的关键字</samp>

让我们来看一些额外有用的关键字。<samp class="SANS_TheSansMonoCd_W5Regular_11">in</samp> 关键字检查某个元素是否存在于给定的序列或容器类型中。这里我们检查 42 是否在后面的列表中，然后检查 21 是否可以作为字符串在集合中找到：

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(42 in [2, 39, 42])</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11"># True</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print('21' in {'2', '39', '42'})</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11"># False</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">in</samp> 关键字返回一个布尔值，因此第一个语句会返回 <samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp>，第二个语句会返回 <samp class="SANS_TheSansMonoCd_W5Regular_11">False</samp>。

<samp class="SANS_TheSansMonoCd_W5Regular_11">is</samp> 关键字检查两个变量是否指向内存中的同一个对象。Python 初学者通常会对 <samp class="SANS_TheSansMonoCd_W5Regular_11">is</samp> 关键字的确切含义感到困惑，但值得花时间理解它的真正含义。在这里，我们可以看到两个变量指向内存中的同一个对象与两个看似相似但指向不同对象的列表之间的区别：

<samp class="SANS_TheSansMonoCd_W5Regular_11">x =</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">3</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">y = x</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(x is y)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11"># True</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print([3] is [3])</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11"># False</samp>

如你在后面的例子中所见，如果你创建了两个列表——即使它们包含相同的元素——它们仍然在内存中指向两个不同的列表对象。如果你后来决定修改其中一个列表对象，这不会影响另一个列表对象。如果你检查某个列表是否指向内存中的同一个对象，结果是 <samp class="SANS_TheSansMonoCd_W5Regular_11">False</samp>。在这里，当我们检查 <samp class="SANS_TheSansMonoCd_W5Regular_11">x</samp> 是否与 <samp class="SANS_TheSansMonoCd_W5Regular_11">y</samp> 相同时，得到的是 <samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp>，因为我们显式地将 <samp class="SANS_TheSansMonoCd_W5Regular_11">y</samp> 设置为指向 <samp class="SANS_TheSansMonoCd_W5Regular_11">x</samp>。然而，当我们检查列表 <samp class="SANS_TheSansMonoCd_W5Regular_11">[3]</samp> 是否与 <samp class="SANS_TheSansMonoCd_W5Regular_11">[3]</samp> 相同时，我们得到的是 <samp class="SANS_TheSansMonoCd_W5Regular_11">False</samp>，因为这两个列表指向内存中的不同对象。如果你修改其中一个，另一个不会改变！

## <samp class="SANS_Futura_Std_Bold_B_11">函数</samp>

*函数* 是可重用的代码片段，用来完成特定任务。程序员可以并且经常将函数与其他程序员共享，以应对特定的任务，节省自己编写代码的时间和精力。

你可以使用 <samp class="SANS_TheSansMonoCd_W5Regular_11">def</samp> 关键字定义一个函数。这里我们定义了两个简单的函数，每个函数都打印一个字符串：

<samp class="SANS_TheSansMonoCd_W5Regular_11">def say_hi():</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print('hi!')</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">def say_hello():</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print('hello!')</samp>

函数由函数名和括号组成，前面加上关键字 <samp class="SANS_TheSansMonoCd_W5Regular_11">def</samp>，后面跟随函数体，它是一个缩进的代码块。这个代码块可以包含其他缩进的块，比如 <samp class="SANS_TheSansMonoCd_W5Regular_11">if</samp> 语句，甚至是更多的函数定义。和 Python 中任何的块定义一样，函数体必须缩进。

这是一个打印两个字符串并分别换行的函数：

<samp class="SANS_TheSansMonoCd_W5Regular_11">def say_bye():</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print('Time to go…')</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print('Bye! ')</samp>

像这样在你的 Python shell 中运行这三个函数：

<samp class="SANS_TheSansMonoCd_W5Regular_11">Say_hi()</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">Say_hello()</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">Say_bye()</samp>

这是输出结果：

<samp class="SANS_TheSansMonoCd_W5Regular_11">hi!</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">hello!</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">Time to go…</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">Bye!</samp>

函数按顺序执行。

<samp class="SANS_Futura_Std_Bold_Condensed_Oblique_I_11">参数</samp>

函数还可以在括号内接受参数。*参数* 允许你定制输出。考虑这个函数，它以名字作为唯一参数，并向 shell 打印一个定制的字符串：

<samp class="SANS_TheSansMonoCd_W5Regular_11">def say_hi(name):</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print('hi ' +</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">name)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">say_hi('Alice')</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">say_hi('Bob')</samp>

我们定义了函数，然后用不同的参数分别运行它两次：首先是 <samp class="SANS_TheSansMonoCd_W5Regular_11">'Alice'</samp>，然后是 <samp class="SANS_TheSansMonoCd_W5Regular_11">'Bob'</samp>。因此，函数执行的输出是不同的：

<samp class="SANS_TheSansMonoCd_W5Regular_11">hi Alice</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">hi Bob</samp>

函数还可以接受多个参数：

<samp class="SANS_TheSansMonoCd_W5Regular_11">def say_hi(a, b):</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print(a + ' 向 ' + b + ' 打招呼')</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">say_hi('Alice', 'Bob')</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">say_hi('Bob', 'Alice')</samp>

输出结果如下：

<samp class="SANS_TheSansMonoCd_W5Regular_11">Alice 向 Bob 打招呼</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">Bob 向 Alice 打招呼</samp>

在第一次函数执行时，参数变量 <samp class="SANS_TheSansMonoCd_W5Regular_11">a</samp> 的值为 <samp class="SANS_TheSansMonoCd_W5Regular_11">'Alice'</samp>，参数变量 <samp class="SANS_TheSansMonoCd_W5Regular_11">b</samp> 的值为 <samp class="SANS_TheSansMonoCd_W5Regular_11">'Bob'</samp>。第二次执行函数时，这两个值会交换，输出结果也不同。

函数也可以有返回值，因此你可以将一个值传入函数，并获得一个返回值，之后可以在代码中使用它，正如在清单 A-1 中所示。

<samp class="SANS_TheSansMonoCd_W5Regular_11">def f(a, b):</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   return a + b</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">x =</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">f(2, 2)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">y = f(40, 2)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(x)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(y)</samp>

<samp class="SANS_Futura_Std_Book_Oblique_I_11">清单 A-1：使用 return 关键字</samp>

返回值是紧跟在 <samp class="SANS_TheSansMonoCd_W5Regular_11">return</samp> 关键字后面的表达式，这也会终止函数的执行。Python 在遇到 <samp class="SANS_TheSansMonoCd_W5Regular_11">return</samp> 关键字时检查返回值并结束函数，立即将该值返回给函数的调用者。

如果你没有显式提供返回表达式，Python 会隐式地在函数末尾加上表达式 <samp class="SANS_TheSansMonoCd_W5Regular_11">return None</samp>。关键字 <samp class="SANS_TheSansMonoCd_W5Regular_11">None</samp> 意味着 *没有值*。其他编程语言（例如 Java）使用关键字 <samp class="SANS_TheSansMonoCd_W5Regular_11">null</samp>，这常常会导致初学者误认为它等于整数值 0。实际上，Python 使用关键字 <samp class="SANS_TheSansMonoCd_W5Regular_11">None</samp> 来表示它是一个空对象，比如一个空列表或空字符串，而不是数字 0。

当一个函数执行完成时，执行权总是返回给调用该函数的地方；<samp class="SANS_TheSansMonoCd_W5Regular_11">return</samp> 关键字只是让你更精确地控制 *何时* 结束函数和 *返回什么* 值。

我们将 <samp class="SANS_TheSansMonoCd_W5Regular_11">a=2</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">b=2</samp> 传入函数 列表 A-1，得到结果 <samp class="SANS_TheSansMonoCd_W5Regular_11">4</samp>。然后我们传入 <samp class="SANS_TheSansMonoCd_W5Regular_11">a=40</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">b=2</samp>，并得到（唯一的）答案 <samp class="SANS_TheSansMonoCd_W5Regular_11">42</samp>。这是输出结果：

<samp class="SANS_TheSansMonoCd_W5Regular_11">4</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">42</samp>

几乎每个仪表板应用程序都包含至少一个增加交互性的函数。通常，你可能有一个根据用户输入更新图表的函数，类似这样的：

<samp class="SANS_TheSansMonoCd_W5Regular_11">def update_graph(value):</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   if value == 2:</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      return 'something'</samp>

接下来，我们将讨论 Python 的一个更高级且高度相关的特性：*默认函数参数*。

<samp class="SANS_Futura_Std_Bold_Condensed_Oblique_I_11">默认函数参数</samp>

默认参数允许你在 Python 中定义带有可选参数的函数。如果用户在调用函数时选择不提供参数，则会使用默认参数。你可以通过在参数名后使用等号 (<samp class="SANS_TheSansMonoCd_W5Regular_11">=</samp>) 并附加默认值来设置默认参数。

附录 A-2 展示了一个更有趣的例子。在这里，我们定义了一个函数 <samp class="SANS_TheSansMonoCd_W5Regular_11">add()</samp>，它返回函数参数 <samp class="SANS_TheSansMonoCd_W5Regular_11">a</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">b</samp> 的和。因此，<samp class="SANS_TheSansMonoCd_W5Regular_11">add(1,2)</samp> 将返回 <samp class="SANS_TheSansMonoCd_W5Regular_11">3</samp>，<samp class="SANS_TheSansMonoCd_W5Regular_11">add(41,1)</samp> 将返回 <samp class="SANS_TheSansMonoCd_W5Regular_11">42</samp>。我们为函数参数指定了默认值：<samp class="SANS_TheSansMonoCd_W5Regular_11">a</samp> 默认为 <samp class="SANS_TheSansMonoCd_W5Regular_11">0</samp>，<samp class="SANS_TheSansMonoCd_W5Regular_11">b</samp> 默认为 <samp class="SANS_TheSansMonoCd_W5Regular_11">1</samp>。如果在函数调用中未传递任何值给这些参数中的一个或两个，它将使用其默认值。因此，<samp class="SANS_TheSansMonoCd_W5Regular_11">add(1)</samp> 将返回 <samp class="SANS_TheSansMonoCd_W5Regular_11">2</samp>，<samp class="SANS_TheSansMonoCd_W5Regular_11">add(-1)</samp> 将返回 <samp class="SANS_TheSansMonoCd_W5Regular_11">0</samp>，<samp class="SANS_TheSansMonoCd_W5Regular_11">add()</samp> 将返回 <samp class="SANS_TheSansMonoCd_W5Regular_11">1</samp>，因为分别使用了 <samp class="SANS_TheSansMonoCd_W5Regular_11">a</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">b</samp> 的默认值 <samp class="SANS_TheSansMonoCd_W5Regular_11">0</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">1</samp>。

<samp class="SANS_TheSansMonoCd_W5Regular_11">def add(a=0, b=1):</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   return a + b</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(add(add(</samp><samp class="SANS_TheSansMonoCd_W7Bold_B_11">add()</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">)))</samp>

<samp class="SANS_Futura_Std_Book_Oblique_I_11">附录 A-2：定义具有默认参数的函数</samp>

这将打印：

<samp class="SANS_TheSansMonoCd_W5Regular_11">3</samp>

在我们在 <samp class="SANS_TheSansMonoCd_W5Regular_11">print(add(add(</samp><samp class="SANS_TheSansMonoCd_W7Bold_B_11">add()</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">)))</samp> 中突出显示的最内层函数调用中，我们调用了函数 <samp class="SANS_TheSansMonoCd_W5Regular_11">add()</samp>，没有传递任何参数，因此它使用了 <samp class="SANS_TheSansMonoCd_W5Regular_11">a</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">b</samp> 的默认值（分别为 0 和 1）。

对于剩余的两个调用，你只需传递一个参数给 <samp class="SANS_TheSansMonoCd_W5Regular_11">add()</samp>，该参数是前一个函数调用的返回值。这个参数将接收 <samp class="SANS_TheSansMonoCd_W5Regular_11">a</samp>，根据参数的位置来决定将哪个参数传递给哪个变量，而 <samp class="SANS_TheSansMonoCd_W5Regular_11">b</samp> 将具有默认值 1。第一个，最内层的 <samp class="SANS_TheSansMonoCd_W5Regular_11">add()</samp> 调用返回 1。这个值传递给第二次调用的 <samp class="SANS_TheSansMonoCd_W5Regular_11">add()</samp>，然后增加 1，接着这个值在第三次调用时再增加 1。

以下是我们执行 Listing A-2 时发生的幕后步骤：

<samp class="SANS_TheSansMonoCd_W5Regular_11">add(add(add()))</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   = add(add(1))</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   = add(2)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   = 3</samp>

你可以看到，默认参数可以帮助你让函数在处理输入时更加灵活。

## <samp class="SANS_Futura_Std_Bold_B_11">Python 资源与进一步阅读</samp>

+   随时查看 *Python 一行代码* 的入门 Python 视频，视频免费提供，地址是 [*https://<wbr>pythononeliners<wbr>.com*](https://pythononeliners.com)。

+   可以在 [*https://<wbr>www<wbr>.python<wbr>.org*](https://www.python.org) 找到最新版 Python 的官方下载网站。

+   你可以在 Finxter 博客上找到关于 Python 列表的完整教程，包含详细的视频内容，地址是 [*https://<wbr>blog<wbr>.finxter<wbr>.com<wbr>/python<wbr>-lists*](https://blog.finxter.com/python-lists)。

+   要查看关于 Python 切片的完整教程和视频，请访问 [*https://<wbr>blog<wbr>.finxter<wbr>.com<wbr>/introduction<wbr>-to<wbr>-slicing<wbr>-in<wbr>-python*](https://blog.finxter.com/introduction-to-slicing-in-python)。

+   你可以在 [*https://<wbr>blog<wbr>.finxter<wbr>.com<wbr>/python<wbr>-dictionary*](https://blog.finxter.com/python-dictionary) 找到有关 Python 字典的完整指南。

+   关于列表推导的教程和视频，请访问 [*https://<wbr>blog<wbr>.finxter<wbr>.com<wbr>/list<wbr>-comprehension*](https://blog.finxter.com/list-comprehension)。

+   一份面向面向对象编程（OOP）的备忘单可以作为 PDF 文件下载，地址是 [*https://<wbr>blog<wbr>.finxter<wbr>.com<wbr>/object<wbr>-oriented<wbr>-programming<wbr>-terminology<wbr>-cheat<wbr>-sheet<wbr>*](https://blog.finxter.com/object-oriented-programming-terminology-cheat-sheet)。

+   在 [*https://<wbr>blog<wbr>.finxter<wbr>.com<wbr>/python<wbr>-crash<wbr>-course*](https://blog.finxter.com/python-crash-course) 找到更多备忘单和免费的 Python 快速入门课程。
