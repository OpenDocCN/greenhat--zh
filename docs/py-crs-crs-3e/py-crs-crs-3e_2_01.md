# 第十章：10

文件与异常

![](img/chapterart.png)

现在你已经掌握了编写结构化、易用程序的基本技能，是时候考虑如何使你的程序更具相关性和实用性了。在本章中，你将学习如何处理文件，这样你的程序就能快速分析大量数据。

你将学会如何处理错误，以确保当程序遇到意外情况时不会崩溃。你将了解*异常*，它是 Python 创建的特殊对象，用来管理程序运行时发生的错误。你还将学习 `json` 模块，它可以帮助你保存用户数据，以防程序停止运行时数据丢失。

学会处理文件和保存数据将使你的程序更易于使用。用户将能够选择何时输入哪些数据。人们可以运行你的程序，完成一些工作，然后关闭程序，并从上次离开的地方继续。学习如何处理异常将帮助你处理文件不存在的情况，并应对其他可能导致程序崩溃的问题。这将使你的程序在遇到坏数据时更具稳健性，无论数据是来自无心的错误，还是恶意企图破坏程序。通过本章你将学到的技能，你将使你的程序更具适用性、可用性和稳定性。

## 从文件中读取

大量的数据可以通过文本文件获取。文本文件可以包含天气数据、交通数据、社会经济数据、文学作品等内容。从文件中读取数据在数据分析应用中尤为重要，但它同样适用于任何需要分析或修改存储在文件中的信息的情况。例如，你可以编写一个程序，读取文本文件的内容，并重新编写文件，添加格式以便浏览器能够显示。

当你想处理文本文件中的信息时，第一步是将文件读取到内存中。然后，你可以一次性处理整个文件的内容，或者逐行处理内容。

### 读取文件内容

首先，我们需要一个包含几行文本的文件。让我们从一个包含 *pi* 到 30 位小数的文件开始，每行包含 10 位小数：

**pi_digits.txt**

```py
3.1415926535
  8979323846
  2643383279
```

要亲自尝试以下示例，你可以在编辑器中输入这些行并将文件保存为 *pi_digits.txt*，或者你可以通过 [`ehmatthes.github.io/pcc_3e`](https://ehmatthes.github.io/pcc_3e) 下载该文件。将文件保存在与你存储本章程序相同的目录中。

下面是一个程序，它打开文件，读取文件内容，并将文件内容打印到屏幕上：

**file_reader.py**

```py
from pathlib import Path

❶ path = Path('pi_digits.txt')
❷ contents = path.read_text()
print(contents)
```

为了处理文件内容，我们需要告诉 Python 文件的路径。*路径*是系统中一个文件或文件夹的确切位置。Python 提供了一个叫 `pathlib` 的模块，使得无论你或你的程序用户使用的是哪个操作系统，都能更轻松地处理文件和目录。像这样提供特定功能的模块通常被称为 *库*，因此命名为 `pathlib`。

我们首先从 `pathlib` 导入 `Path` 类。使用 `Path` 对象指向文件后，你可以做很多事情。例如，在操作文件之前，你可以检查文件是否存在，读取文件内容，或者向文件写入新数据。在这里，我们构建了一个表示文件 *pi_digits.txt* 的 `Path` 对象，并将其赋值给变量 `path` ❶。因为这个文件保存在与我们编写的 *.py* 文件相同的目录中，所以 `Path` 只需要文件名就能访问该文件。

一旦我们有了一个表示 *pi_digits.txt* 的 `Path` 对象，我们就可以使用 `read_text()` 方法读取文件的全部内容 ❷。文件的内容会作为一个字符串返回，我们将其赋值给变量 `contents`。当我们打印 `contents` 的值时，看到的就是文件的全部内容：

```py
3.1415926535
  8979323846
  2643383279 
```

这段输出与原始文件的唯一区别是输出末尾多了一行空白行。空白行出现是因为 `read_text()` 在到达文件末尾时返回了一个空字符串；这个空字符串显示为一行空白。

我们可以通过对 `contents` 字符串使用 `rstrip()` 来去除多余的空白行：

```py
from pathlib import Path

path = Path('pi_digits.txt')
contents = path.read_text()
contents = contents.rstrip()
print(contents)
```

回想一下第二章，Python 的 `rstrip()` 方法会移除字符串右侧的所有空白字符。现在输出的内容与原始文件完全一致：

```py
3.1415926535
  8979323846
  2643383279
```

我们可以通过在调用 `read_text()` 后立即应用 `rstrip()` 方法来去掉读取文件内容时的末尾换行符：

```py
contents = path.read_text().rstrip()
```

这一行代码告诉 Python 调用我们正在处理的文件的 `read_text()` 方法。然后它将 `read_text()` 返回的字符串应用 `rstrip()` 方法。清理后的字符串随后被赋值给变量 `contents`。这种方法称为 *方法链*，在编程中你会经常看到它。

### 相对路径和绝对路径

当你将一个简单的文件名，比如 *pi_digits.txt*，传递给 `Path` 时，Python 会在当前执行的文件所在的目录中查找该文件（也就是你的 *.py* 程序文件）。

有时，取决于你如何组织工作，你想要打开的文件可能不在与你的程序文件相同的目录中。例如，你可能将程序文件存储在名为*python_work*的文件夹中；在*python_work*文件夹内，你可能会有另一个名为*text_files*的文件夹，用以区分程序文件和它们所操作的文本文件。即使*text_files*在*python_work*中，直接传递文件名给`Path`也无法正常工作，因为 Python 只会在*python_work*中查找，并且在那停止；它不会继续查找*text_files*中的文件。为了让 Python 从与你的程序文件存储位置不同的目录打开文件，你需要提供正确的路径。

编程中指定路径有两种主要方法。*相对文件路径*告诉 Python 从当前正在运行的程序文件所在目录的位置查找给定位置。由于*text_files*位于*python_work*内，我们需要构建一个从*text_files*文件夹开始，并以文件名结束的路径。以下是如何构建这个路径的示例：

```py
path = Path('text_files/`filename`.txt')
```

你也可以告诉 Python 文件在你电脑上的确切位置，无论正在执行的程序存储在哪里。这称为*绝对文件路径*。如果相对路径不起作用，你可以使用绝对路径。例如，如果你把*text_files*放在了与*python_work*不同的文件夹中，那么直接传递路径`Path` `'text_files/``filename``.txt'` 是行不通的，因为 Python 只会在*python_work*中查找该位置。你需要写出绝对路径来明确指定 Python 应该查找的位置。

绝对路径通常比相对路径长，因为它们是从系统的根文件夹开始的：

```py
path = Path('/home/eric/data_files/text_files/`filename`.txt')
```

使用绝对路径，你可以从系统上的任何位置读取文件。现在，最简单的做法是将文件存储在与你的程序文件相同的目录中，或存储在一个名为*text_files*的文件夹内，该文件夹位于存储程序文件的目录中。

### 访问文件的行

当你处理文件时，你通常需要检查文件的每一行。你可能在文件中查找特定的信息，或者你可能想以某种方式修改文件中的文本。例如，你可能希望阅读一份天气数据文件，并处理包含*sunny*（晴天）字样的所有描述该日天气的行。在新闻报道中，你可能需要查找任何带有标签`<headline>`的行，并以特定格式重新写入该行。

你可以使用`splitlines()`方法将一长串字符串转换为一组行，然后使用`for`循环逐行检查文件中的内容：

**file_reader.py**

```py
from pathlib import Path

path = Path('pi_digits.txt')
❶ contents = path.read_text()

❷ lines = contents.splitlines()
for line in lines:
    print(line)
```

我们首先读取文件的全部内容，就像之前那样 ❶。如果你打算处理文件中的单独行，在读取文件时不需要去除任何空白字符。`splitlines()`方法会返回文件中所有行的列表，我们将这个列表赋值给变量`lines` ❷。然后我们遍历这些行并打印每一行：

```py
3.1415926535
  8979323846
  2643383279
```

由于我们没有修改任何行，输出与原始文本文件完全一致。

### 处理文件内容

在你将文件的内容读取到内存中后，你可以对这些数据做任何你想做的事情，所以让我们简要地探索一下*π*的数字。首先，我们将尝试构建一个不包含任何空白字符的字符串，包含文件中的所有数字：

**pi_string.py**

```py
from pathlib import Path

path = Path('pi_digits.txt')
contents = path.read_text()

lines = contents.splitlines()
pi_string = ''
❶ for line in lines:
    pi_string += line

print(pi_string)
print(len(pi_string))
```

我们首先读取文件并将每行数字存储在一个列表中，就像我们在前面的示例中所做的那样。然后我们创建一个变量`pi_string`，用来存储*π*的数字。我们写一个循环，将每一行数字添加到`pi_string`中 ❶。然后我们打印这个字符串，并显示字符串的长度：

```py
3.1415926535  8979323846  2643383279
36
```

变量`pi_string`包含了每行数字左侧的空白字符，但我们可以通过对每一行使用`lstrip()`来去除这些空白字符：

```py
*--snip--*
for line in lines:
  pi_string += line.lstrip()

print(pi_string)
print(len(pi_string))
```

现在我们有一个包含*π*到 30 位小数的字符串。这个字符串长度为 32 个字符，因为它还包括了前导的`3`和小数点：

```py
3.141592653589793238462643383279
32
```

### 大文件：一百万位

到目前为止，我们专注于分析一个只包含三行的文本文件，但这些示例中的代码同样适用于更大的文件。如果我们从一个包含*π*到 1,000,000 位的小数的文本文件开始，而不是仅仅 30 位，我们可以创建一个包含所有这些数字的字符串。我们不需要更改程序，只需传入一个不同的文件。我们还会只打印前 50 位小数，这样就不需要在终端上看一百万个数字滚动过去：

**pi_string.py**

```py
from pathlib import Path

path = Path('pi_million_digits.txt')
contents = path.read_text()

lines = contents.splitlines()
pi_string = ''
for line in lines:
 pi_string += line.lstrip()

print(f"{pi_string[:52]}...")
print(len(pi_string))
```

输出显示，我们确实得到了一个包含*π*到 1,000,000 位小数的字符串：

```py
3.14159265358979323846264338327950288419716939937510...
1000002
```

Python 没有固有的限制来限制你可以处理的数据量；你可以处理尽可能多的数据，只要你的系统内存能承受。

### 你的生日出现在*π*中吗？

我一直很好奇我的生日是否出现在*π*的数字中。让我们用刚刚写的程序来查找一下，看看某人的生日是否出现在*π*的前一百万位数字中。我们可以通过将每个生日表示为一个数字字符串，然后查看该字符串是否出现在`pi_string`中来实现：

**pi_birthday.py**

```py
*--snip--*
for line in lines:
 pi_string += line.strip()

birthday = input("Enter your birthday, in the form mmddyy: ")
if birthday in pi_string:
    print("Your birthday appears in the first million digits of pi!")
else:
    print("Your birthday does not appear in the first million digits of pi.")
```

我们首先提示用户输入生日，然后检查该字符串是否在`pi_string`中。让我们试试：

```py
Enter your birthdate, in the form mmddyy: **120372**
Your birthday appears in the first million digits of pi!
```

我的生日确实出现在*π*的数字中！一旦你从文件中读取数据，就可以以任何你能想象的方式分析其内容。

## 写入文件

保存数据的最简单方法之一就是将其写入文件。当你将文本写入文件时，输出在你关闭包含程序输出的终端后仍然可用。你可以在程序运行完毕后检查输出，也可以将输出文件分享给他人。你还可以编写读取文本并重新加载到内存中的程序，以便以后继续使用。

### 写入单行

一旦定义了路径，你就可以使用 `write_text()` 方法向文件写入内容。为了演示这一过程，让我们写一个简单的消息并将其存储到文件中，而不是直接输出到屏幕：

**write_message.py**

```py
from pathlib import Path

path = Path('programming.txt')
path.write_text("I love programming.")
```

`write_text()` 方法接受一个参数：你想写入文件的字符串。这个程序没有终端输出，但如果你打开 *programming.txt* 文件，你会看到一行内容：

**programming.txt**

```py
I love programming.
```

这个文件的行为就像你电脑上的任何其他文件。你可以打开它，写入新文本，复制其中的内容，粘贴到其他地方，等等。

### 写入多行

`write_text()` 方法在后台做了几件事。如果 `path` 指向的文件不存在，它会创建该文件。而且，在将字符串写入文件后，它会确保文件被正确关闭。没有正确关闭的文件可能会导致数据丢失或损坏。

要向文件写入多行，你需要构建一个包含文件所有内容的字符串，然后使用该字符串调用 `write_text()`。让我们写几行内容到 *programming.txt* 文件中：

```py
from pathlib import Path

contents = "I love programming.\n"
contents += "I love creating new games.\n"
contents += "I also love working with data.\n"

path = Path('programming.txt')
path.write_text(contents)
```

我们定义了一个名为 `contents` 的变量，用来保存文件的全部内容。在下一行，我们使用 `+=` 运算符将新的内容添加到这个字符串中。你可以根据需要多次执行此操作，构建任意长度的字符串。在这个例子中，我们在每一行的末尾添加了换行符，以确保每条语句都出现在单独的一行上。

如果你运行这个程序并打开 *programming.txt* 文件，你会看到每一行内容都出现在文本文件中：

```py
I love programming.
I love creating new games.
I also love working with data.
```

你还可以使用空格、制表符和空白行来格式化输出，就像你在终端输出中所做的那样。字符串的长度没有限制，这也是许多计算机生成文档的创建方式。

## 异常

Python 使用名为 *exceptions*（异常）的特殊对象来处理程序执行过程中发生的错误。当发生一个错误时，导致 Python 不确定下一步该做什么，它会创建一个异常对象。如果你编写了处理异常的代码，程序将继续运行。如果你没有处理异常，程序会停止并显示 *traceback*（追踪信息），其中包含已引发的异常报告。

异常通过`try`-`except`块进行处理。一个*try*-*except*块要求 Python 执行某些操作，但它也告诉 Python 如果发生异常时该怎么做。当你使用`try`-`except`块时，即使出现问题，程序也会继续运行。用户将看到你编写的友好错误信息，而不是可能让用户困惑的回溯信息。

### 处理`ZeroDivisionError`异常

让我们看一个简单的错误，导致 Python 引发异常。你可能知道，除以零是不可能的，但我们还是要求 Python 这么做：

**division_calculator.py**

```py
print(5/0)
```

Python 无法执行此操作，因此我们会得到一个回溯信息：

```py
Traceback (most recent call last):
  File "division_calculator.py", line 1, in <module>
    print(5/0)
          ~^~
❶ ZeroDivisionError: division by zero
```

回溯中报告的错误，`ZeroDivisionError`，是一个异常对象 ❶。Python 会在无法执行我们要求的操作时创建这种对象。当发生这种情况时，Python 会停止程序并告诉我们引发的异常类型。我们可以使用这些信息来修改我们的程序。我们将告诉 Python 在这种异常发生时该做什么，这样如果它再次发生，我们就能做好准备。

### 使用`try`-`except`块

当你认为可能发生错误时，可以编写一个`try`-`except`块来处理可能引发的异常。你告诉 Python 尝试运行某些代码，并告诉它在代码引发特定类型的异常时应该怎么做。

下面是一个用于处理`ZeroDivisionError`异常的`try`-`except`块的示例：

```py
try:
    print(5/0)
except ZeroDivisionError:
    print("You can't divide by zero!")
```

我们将导致错误的`print(5/0)`语句放在`try`块中。如果`try`块中的代码执行成功，Python 会跳过`except`块。如果`try`块中的代码发生错误，Python 会查找与引发的错误匹配的`except`块，并执行该块中的代码。

在这个例子中，`try`块中的代码引发了一个`ZeroDivisionError`，因此 Python 会查找一个`except`块，告诉它如何响应。然后 Python 会执行该块中的代码，用户将看到一个友好的错误信息，而不是回溯信息：

```py
You can't divide by zero!
```

如果`try`-`except`块后面还有更多的代码，程序将继续运行，因为我们告诉 Python 如何处理错误。让我们看一个示例，捕获错误可以让程序继续运行。

### 使用异常防止崩溃

正确处理错误在程序发生错误后仍有更多工作要做时尤为重要。尤其是在提示用户输入的程序中，这种情况经常发生。如果程序能够适当响应无效输入，它可以提示用户输入更多有效的内容，而不是崩溃。

让我们创建一个简单的仅进行除法的计算器：

**division_calculator.py**

```py
print("Give me two numbers, and I'll divide them.")
print("Enter 'q' to quit.")

while True:
❶     first_number = input("\nFirst number: ")
    if first_number == 'q':
        break
❷     second_number = input("Second number: ")
    if second_number == 'q':
        break
❸     answer = int(first_number) / int(second_number)
    print(answer)
```

这个程序提示用户输入一个`first_number` ❶，如果用户没有输入`q`来退出，则输入一个`second_number` ❷。然后我们将这两个数字相除得到`answer` ❸。这个程序没有做任何错误处理，因此如果要求它进行除以零的操作，它会崩溃：

```py
Give me two numbers, and I'll divide them.
Enter 'q' to quit.

First number: **5**
Second number: **0**
Traceback (most recent call last):
  File "division_calculator.py", line 11, in <module>
    answer = int(first_number) / int(second_number)
             ~~~~~~~~~~~~~~~~~~^~~~~~~~~~~~~~~~~~~~
ZeroDivisionError: division by zero
```

程序崩溃虽然不好，但让用户看到追踪信息也不是一个好主意。非技术用户会被这些信息弄得很困惑，而在恶意环境中，攻击者会通过这些信息学到你不希望他们知道的内容。例如，他们会知道你的程序文件名，还会看到某个无法正常工作的代码部分。一个有经验的攻击者有时可以利用这些信息来确定攻击代码的方式。

### `else` 块

我们可以通过将可能产生错误的行包装在`try`-`except`块中，使程序更具抗错能力。错误发生在执行除法的那一行，所以我们将在那里放置`try`-`except`块。这个例子还包括了一个`else`块。任何依赖于`try`块成功执行的代码都会放入`else`块中：

```py
*--snip--*
while True:
  *--snip--*
 if second_number == 'q':
 break
❶     try:
        answer = int(first_number) / int(second_number)
❷     except ZeroDivisionError:
        print("You can't divide by 0!")
❸     else:
 print(answer)
```

我们要求 Python 在`try`块中尝试完成除法操作❶，该块只包含可能引发错误的代码。任何依赖于`try`块成功的代码都放入`else`块。在这种情况下，如果除法操作成功，我们使用`else`块来打印结果❸。

`except`块告诉 Python 在遇到`ZeroDivisionError`时如何响应❷。如果`try`块因为除零错误而未能成功执行，我们会打印一条友好的消息，告诉用户如何避免这种错误。程序会继续运行，用户也看不到追踪信息：

```py
Give me two numbers, and I'll divide them.
Enter 'q' to quit.

First number: **5**
Second number: **0**
You can't divide by 0!

First number: **5**
Second number: **2**
2.5

First number: **q**
```

只有可能引发异常的代码应该放入`try`块中。有时你还会有一些额外的代码，只有在`try`块成功执行后才应该运行，这些代码会放在`else`块中。`except`块告诉 Python 在`try`块运行代码时遇到特定异常时应该怎么做。

通过预测可能出现的错误来源，你可以编写出更加健壮的程序，即使遇到无效数据或缺失资源时也能继续运行。你的代码将能抵抗无心的用户错误和恶意攻击。

### 处理 FileNotFoundError 异常

在处理文件时，一个常见的问题是如何处理丢失的文件。你要找的文件可能在不同的位置，文件名可能拼写错误，或者文件根本不存在。你可以通过`try`-`except`块来处理所有这些情况。

让我们尝试读取一个不存在的文件。以下程序尝试读取*爱丽丝梦游仙境*的内容，但我没有将文件*alice.txt*保存在与*alice.py*相同的目录下：

**alice.py**

```py
from pathlib import Path

path = Path('alice.txt')
contents = path.read_text(encoding='utf-8')
```

请注意，我们在这里使用`read_text()`的方式与之前所见的稍有不同。当你的系统默认编码与正在读取的文件编码不匹配时，需要使用`encoding`参数。这种情况最常见于读取来自非本地系统创建的文件时。

Python 无法从丢失的文件中读取数据，因此会引发异常：

```py
Traceback (most recent call last):
❶   File "alice.py", line 4, in <module>
❷     contents = path.read_text(encoding='utf-8')
               ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
 File "/.../pathlib.py", line 1056, in read_text
    with self.open(mode='r', encoding=encoding, errors=errors) as f:
         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/.../pathlib.py", line 1042, in open
    return io.open(self, mode, buffering, encoding, errors, newline)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
❸ FileNotFoundError: [Errno 2] No such file or directory: 'alice.txt'
```

这是比我们之前见过的回溯信息更长的回溯，所以让我们来看看如何理解更复杂的回溯。通常最好从回溯的最后一行开始查看。在最后一行，我们可以看到引发了`FileNotFoundError`异常 ❸。这很重要，因为它告诉我们在`except`块中使用哪种类型的异常。

回溯信息的开头附近 ❶，我们可以看到错误发生在文件*alice.py*的第 4 行。下一行显示了导致错误的代码 ❷。其余的回溯信息展示了涉及打开和读取文件的库中的一些代码。通常，你不需要逐行阅读或理解回溯中的所有这些内容。

为了处理引发的错误，`try`块将从回溯中标识为问题行的那一行开始。在我们的示例中，这一行包含了`read_text()`：

```py
from pathlib import Path

path = Path('alice.txt')
try:
    contents = path.read_text(encoding='utf-8')
❶ except FileNotFoundError:
    print(f"Sorry, the file {path} does not exist.")
```

在这个示例中，`try`块中的代码会产生`FileNotFoundError`，因此我们编写一个匹配该错误的`except`块 ❶。当找不到文件时，Python 会运行该块中的代码，结果是显示一个友好的错误信息，而不是回溯信息：

```py
Sorry, the file alice.txt does not exist.
```

如果文件不存在，程序没有其他操作要做，因此我们只会看到这些输出。让我们基于这个例子继续，看看异常处理如何帮助你处理多个文件时的问题。

### 分析文本

你可以分析包含整本书的文本文件。许多经典文学作品作为简单的文本文件可供使用，因为它们属于公有领域。本节使用的文本来自古腾堡计划（[`gutenberg.org`](https://gutenberg.org)）。古腾堡计划维护着一系列公有领域的文学作品，如果你有兴趣在编程项目中使用文学文本，这是一个很好的资源。

让我们导入*爱丽丝梦游仙境*的文本并尝试统计其中的单词数。为此，我们将使用字符串方法`split()`，它默认在任何空白字符处将字符串拆分：

```py
from pathlib import Path

path = Path('alice.txt')
try:
 contents = path.read_text(encoding='utf-8')
except FileNotFoundError:
 print(f"Sorry, the file {path} does not exist.")
else:
    # Count the approximate number of words in the file:
❶     words = contents.split()
❷     num_words = len(words)
    print(f"The file {path} has about {num_words} words.")
```

我将文件*alice.txt*移动到了正确的目录，因此这次`try`块会正常工作。我们获取包含*爱丽丝梦游仙境*全文的字符串`contents`，并使用`split()`方法将其转换为一本书中所有单词的列表 ❶。使用`len()`函数计算该列表的长度 ❷，可以很好地估算出原始文本中的单词数。最后，我们打印一条报告，显示文件中找到的单词数。此代码放在`else`块中，因为只有在`try`块中的代码成功执行时，它才会执行。

输出告诉我们*alice.txt*中有多少个单词：

```py
The file alice.txt has about 29594 words.
```

计数有点偏高，因为出版商在此处使用的文本文件中提供了额外的信息，但它大致上可以反映出*爱丽丝梦游仙境*的字数。

### 处理多个文件

让我们添加更多的书籍来分析，但在此之前，先将程序的大部分内容移动到一个名为`count_words()`的函数中。这将使我们能够更轻松地对多本书进行分析：

**word_count.py**

```py
from pathlib import Path

def count_words(path):
❶     """Count the approximate number of words in a file."""
    try:
 contents = path.read_text(encoding='utf-8')
 except FileNotFoundError:
 print(f"Sorry, the file {path} does not exist.")
 else:
 # Count the approximate number of words in the file:
 words = contents.split()
 num_words = len(words)
 print(f"The file {path} has about {num_words} words.")

path = Path('alice.txt')
count_words(path)
```

这段代码大部分没有变化。它只是被缩进并移到了`count_words()`的主体内。修改程序时保持注释更新是一个好习惯，因此注释也已经被更改为文档字符串，并稍微重新措辞了 ❶。

现在我们可以编写一个简短的循环，统计我们想要分析的任何文本中的单词。我们通过将要分析的文件名存储在一个列表中来实现，然后对列表中的每个文件调用`count_words()`。我们将尝试统计*Alice in Wonderland*（《爱丽丝梦游仙境》）、*Siddhartha*（《悉达多》）、*Moby Dick*（《白鲸》）和*Little Women*（《小妇人》）的字数，这些书籍都可以在公共领域找到。我故意将*siddhartha.txt*文件从包含*word_count.py*的目录中遗漏，以便我们可以看到程序如何处理缺失的文件：

```py
from pathlib import Path

def count_words(filename):
  *--snip--*

filenames = ['alice.txt', 'siddhartha.txt', 'moby_dick.txt', 
        'little_women.txt']
for filename in filenames:
❶     path = Path(filename)
    count_words(path)
```

文件名作为简单的字符串存储。每个字符串在调用`count_words()`之前都会被转换为一个`Path`对象 ❶。缺失的*siddhartha.txt*文件不会影响程序的其余部分执行：

```py
The file alice.txt has about 29594 words.
Sorry, the file siddhartha.txt does not exist.
The file moby_dick.txt has about 215864 words.
The file little_women.txt has about 189142 words.
```

在这个例子中使用`try`-`except`块提供了两个显著的优势。我们防止用户看到回溯信息，并且让程序继续分析它能够找到的文本。如果我们没有捕获到*siddhartha.txt*引发的`FileNotFoundError`，用户将看到完整的回溯信息，程序会在尝试分析*Siddhartha*后停止运行。它将永远不会分析*Moby Dick*或*Little Women*。

### 安静地失败

在之前的示例中，我们告知用户某个文件不可用。但并不是所有捕获到的异常都需要报告。有时你希望程序在发生异常时默默地失败，并继续执行，就好像什么都没发生一样。要让程序安静地失败，你照常编写`try`块，但在`except`块中明确告诉 Python 不做任何事。Python 有一个`pass`语句，它会告诉 Python 在该块中什么也不做：

```py
def count_words(path):
 """Count the approximate number of words in a file."""
 try:
  *--snip--*
 except FileNotFoundError:
        pass
 else:
  *--snip--*
```

这份列表和之前的唯一不同之处在于`except`块中的`pass`语句。现在，当抛出`FileNotFoundError`时，`except`块中的代码会运行，但什么也不会发生。不会生成回溯信息，用户也不会看到针对抛出的错误的任何输出。用户只能看到每个文件的字数，但不会看到文件未找到的任何提示：

```py
The file alice.txt has about 29594 words.
The file moby_dick.txt has about 215864 words.
The file little_women.txt has about 189142 words.
```

`pass`语句也充当占位符。它提醒你在程序执行的特定位置选择不做任何操作，并且可能稍后在该位置做些事情。例如，在这个程序中，我们可能决定将任何缺失的文件名写入名为*missing_files.txt*的文件。我们的用户看不到这个文件，但我们可以读取这个文件并处理任何缺失的文本。

### 决定报告哪些错误

你如何知道何时向用户报告错误，以及何时让程序悄无声息地失败？如果用户知道哪些文本应该被分析，他们可能会希望看到一条消息，告诉他们为什么某些文本未被分析。如果用户期望看到一些结果，但不知道哪些书籍应该被分析，他们可能不需要知道某些文本不可用。向用户提供他们不需要的信息可能会降低程序的可用性。Python 的错误处理结构让你能精细地控制在出现问题时与用户共享多少信息；决定共享多少信息由你来决定。

编写良好、经过充分测试的代码不容易出现内部错误，如语法或逻辑错误。但是每当你的程序依赖于外部因素，比如用户输入、文件的存在或网络连接的可用性时，都有可能引发异常。通过一些经验，你将能知道在程序中应该在哪里加入异常处理块，以及在出现错误时应该向用户报告多少信息。

## 存储数据

你的许多程序都会要求用户输入某些类型的信息。你可能会允许用户在游戏中存储偏好设置，或为可视化提供数据。无论你的程序关注的是什么，你都会将用户提供的信息存储在列表和字典等数据结构中。当用户关闭程序时，你几乎总是希望保存他们输入的信息。一种简单的实现方法是使用`json`模块来存储数据。

`json`模块允许你将简单的 Python 数据结构转换为 JSON 格式的字符串，然后在程序下次运行时从该文件中加载数据。你还可以使用`json`在不同的 Python 程序之间共享数据。更好的是，JSON 数据格式并不限于 Python，因此你可以将存储在 JSON 格式中的数据与使用其他编程语言的人员共享。这是一个有用且便于移植的格式，而且容易学习。

### 使用 json.dumps()和 json.loads()

让我们编写一个简单的程序，存储一组数字，再编写另一个程序将这些数字读取回内存。第一个程序将使用`json.dumps()`来存储这组数字，第二个程序将使用`json.loads()`。

`json.dumps()`函数接受一个参数：一个应该转换为 JSON 格式的数据。该函数返回一个字符串，我们可以将其写入数据文件：

**number_writer.py**

```py
from pathlib import Path
import json

numbers = [2, 3, 5, 7, 11, 13]

❶ path = Path('numbers.json')
❷ contents = json.dumps(numbers)
path.write_text(contents)
```

我们首先导入`json`模块，然后创建一个数字列表来处理。接着我们选择一个文件名来存储数字列表❶。通常使用文件扩展名*.json*来表示文件中的数据采用 JSON 格式。然后，我们使用`json.dumps()`❷函数生成一个包含数据 JSON 表示形式的字符串。得到这个字符串后，我们使用之前使用的`write_text()`方法将其写入文件。

该程序没有输出，但让我们打开文件*numbers.json*并查看它。数据以类似 Python 的格式存储：

```py
[2, 3, 5, 7, 11, 13]
```

现在我们将编写一个单独的程序，使用`json.loads()`将列表读回内存：

**number_reader.py**

```py
from pathlib import Path
import json

❶ path = Path('numbers.json')
❷ contents = path.read_text()
❸ numbers = json.loads(contents)

print(numbers)
```

我们确保从我们写入的同一个文件中读取❶。由于数据文件只是一个具有特定格式的文本文件，我们可以使用`read_text()`方法读取它❷。然后我们将文件内容传递给`json.loads()`❸。该函数接收一个 JSON 格式的字符串并返回一个 Python 对象（在此情况下是一个列表），我们将其赋值给`numbers`。最后，我们打印恢复的数字列表，看到它与*number_writer.py*中创建的列表相同：

```py
[2, 3, 5, 7, 11, 13]
```

这是在两个程序之间共享数据的一种简单方法。

### 保存和读取用户生成的数据

使用`json`保存数据在处理用户生成的数据时非常有用，因为如果你不以某种方式存储用户的信息，当程序停止运行时，你将丢失这些数据。让我们看一个例子，我们首次运行程序时提示用户输入名字，然后下次运行时记住他们的名字。

让我们从存储用户的名字开始：

**remember_me.py**

```py
from pathlib import Path
import json

❶ username = input("What is your name? ")

❷ path = Path('username.json')
contents = json.dumps(username)
path.write_text(contents)

❸ print(f"We'll remember you when you come back, {username}!")
```

我们首先提示用户输入用户名以进行存储❶。接下来，我们将刚刚收集的数据写入一个名为*username.json*的文件❷。然后我们打印一条消息，告知用户我们已经存储了他们的信息❸：

```py
What is your name? **Eric**
We'll remember you when you come back, Eric!
```

现在让我们编写一个新程序，向已存储姓名的用户打招呼：

**greet_user.py**

```py
from pathlib import Path
import json

❶ path = Path('username.json')
contents = path.read_text()
❷ username = json.loads(contents)

print(f"Welcome back, {username}!")
```

我们读取数据文件的内容❶，然后使用`json.loads()`将恢复的数据赋值给变量`username`❷。由于我们已经恢复了用户名，我们可以用个性化的问候语欢迎用户回来：

```py
Welcome back, Eric!
```

我们需要将这两个程序合并为一个文件。当有人运行*remember_me.py*时，我们希望尽可能从内存中检索他们的用户名；如果没有找到，我们将提示输入用户名并将其存储在*username.json*中，以便下次使用。我们可以在这里写一个`try`-`except`块来处理*username.json*不存在的情况，但我们将使用`pathlib`模块中的一个方便方法：

**remember_me.py**

```py
from pathlib import Path
import json

path = Path('username.json')
❶ if path.exists():
    contents = path.read_text()
    username = json.loads(contents)
    print(f"Welcome back, {username}!")
❷ else:
    username = input("What is your name? ")
    contents = json.dumps(username)
    path.write_text(contents)
    print(f"We'll remember you when you come back, {username}!")
```

你可以使用`Path`对象的许多有用方法。`exists()`方法会在文件或文件夹存在时返回`True`，如果不存在则返回`False`。这里我们使用`path.exists()`来检查用户名是否已经存储❶。如果*username.json*存在，我们加载用户名并打印个性化的问候语。

如果文件*username.json*不存在❷，我们会提示用户输入用户名并存储用户输入的值。我们还会打印一个熟悉的消息，告诉用户我们会记住他们，当他们回来时。

无论执行哪个代码块，结果都会是一个用户名和相应的问候语。如果这是程序第一次运行，输出将是：

```py
What is your name? **Eric**
We'll remember you when you come back, Eric!
```

否则：

```py
Welcome back, Eric!
```

这是你在程序至少运行过一次后看到的输出。即使这一部分的数据只是一个字符串，程序也同样适用于任何可以转换为 JSON 格式字符串的数据。

### 重构

通常，你会遇到一种情况，代码可以正常运行，但你会意识到可以通过将代码拆分成一系列具有特定功能的函数来改进代码。这个过程叫做*重构*。重构使你的代码更简洁、更易于理解，也更容易扩展。

我们可以通过将大部分逻辑移动到一个或多个函数中来重构*remember_me.py*。*remember_me.py*的重点是问候用户，所以让我们把现有的所有代码移到一个叫做`greet_user()`的函数中：

**remember_me.py**

```py
from pathlib import Path
import json

def greet_user():
❶     """Greet the user by name."""
    path = Path('username.json')
 if path.exists():
 contents = path.read_text()
 username = json.loads(contents)
 print(f"Welcome back, {username}!")
 else:
 username = input("What is your name? ")
 contents = json.dumps(username)
 path.write_text(contents)
 print(f"We'll remember you when you come back, {username}!")

greet_user()
```

因为我们现在使用了函数，所以我们将注释重写为反映程序当前工作方式的文档字符串❶。这个文件变得稍微更简洁了一些，但函数`greet_user()`做的不仅仅是问候用户——它还会检索已存储的用户名（如果存在），如果没有，则会提示输入新用户名。

让我们重构`greet_user()`，使它不再做这么多不同的任务。我们将首先把检索已存储用户名的代码移到一个单独的函数中：

```py
from pathlib import Path
import json

def get_stored_username(path):
❶     """Get stored username if available."""
 if path.exists():
 contents = path.read_text()
 username = json.loads(contents)
        return username
    else:
❷         return None

def greet_user():
 """Greet the user by name."""
    path = Path('username.json')
    username = get_stored_username(path)
❸     if username:
        print(f"Welcome back, {username}!")
    else:
 username = input("What is your name? ")
 contents = json.dumps(username)
 path.write_text(contents)
 print(f"We'll remember you when you come back, {username}!")

greet_user()
```

新函数`get_stored_username()`❶有一个明确的目的，正如文档字符串所述。这个函数检索存储的用户名，并返回用户名（如果找到）。如果传递给`get_stored_username()`的路径不存在，函数返回`None`❷。这是一个好习惯：函数应该返回你期望的值，或者返回`None`。这使我们可以用函数的返回值进行简单的测试。如果成功检索到用户名，我们会向用户打印欢迎回来的消息❸；如果失败，我们会提示输入新用户名。

我们应该再从`greet_user()`中提取出一个代码块。如果用户名不存在，我们应该把提示输入新用户名的代码移到一个专门的函数中：

```py
from pathlib import Path
import json

def get_stored_username(path):
 """Get stored username if available."""
    *--snip--*

def get_new_username(path):
    """Prompt for a new username."""
    username = input("What is your name? ")
 contents = json.dumps(username)
 path.write_text(contents)
    return username

def greet_user():
 """Greet the user by name."""
 path = Path('username.json')
❶ username = get_stored_username(path)
 if username:
 print(f"Welcome back, {username}!")
 else:
❷         username = get_new_username(path)
        print(f"We'll remember you when you come back, {username}!")

greet_user()
```

在最终版本的*remember_me.py*中，每个函数都有一个单一且明确的目的。我们调用`greet_user()`，该函数会打印出合适的消息：它要么欢迎回来的老用户，要么向新用户问好。它通过调用`get_stored_username()`❶来完成这一任务，`get_stored_username()`只负责获取已存储的用户名（如果存在）。最后，如果有必要，`greet_user()`会调用`get_new_username()`❷，它仅负责获取新的用户名并将其存储。这样的工作分工是编写清晰代码的重要组成部分，能够使代码易于维护和扩展。

## 总结

在这一章中，你学会了如何处理文件。你学会了读取文件的全部内容，并且如果需要的话，可以逐行处理文件内容。你学会了向文件中写入任意文本。你还了解了异常处理以及如何处理程序中可能遇到的异常。最后，你学会了如何存储 Python 数据结构，以便保存用户提供的信息，避免用户每次运行程序时都要重新开始。

在第十一章，你将学习高效的代码测试方法。这将帮助你确保所编写的代码是正确的，并且帮助你识别在继续构建程序时可能引入的错误。
