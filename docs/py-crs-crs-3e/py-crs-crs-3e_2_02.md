# 第十一章：11

测试你的代码

![](img/chapterart.png)

当你编写函数或类时，你还可以为这些代码编写测试。测试可以证明你的代码在响应它设计要接收的所有输入时能够按预期工作。当你编写测试时，你可以确信，随着越来越多的人开始使用你的程序，你的代码会正确运行。你还可以在添加新代码时进行测试，确保你的修改不会破坏程序原有的行为。每个程序员都会犯错，因此每个程序员都必须经常测试他们的代码，以便在用户遇到问题之前发现并解决问题。

在本章中，你将学习如何使用`pytest`来测试你的代码。`pytest`库是一套工具集，可以帮助你快速而简单地编写你的第一个测试，并在你的项目复杂性增加时，支持你的测试。Python 默认不包括`pytest`，所以你将学习如何安装外部库。知道如何安装外部库将使你可以使用各种设计良好的代码。这些库将极大扩展你可以从事的项目类型。

你将学习构建一系列测试，并检查每一组输入是否产生你想要的输出。你将看到通过的测试是什么样的，失败的测试是什么样的，你还将学习如何利用失败的测试来改进代码。你将学会测试函数和类，并开始理解为一个项目编写多少测试。

## 使用 pip 安装 pytest

尽管 Python 在标准库中包含了很多功能，但 Python 开发者也非常依赖第三方包。*第三方包*是指在核心 Python 语言之外开发的库。一些流行的第三方库最终会被纳入标准库，并且从那时起将包含在大多数 Python 安装中。这种情况通常发生在那些一旦修复了初始错误就不太可能发生重大变化的库上。这类库可以与整体语言的发展速度保持一致。

然而，许多软件包被排除在标准库之外，以便它们可以在独立于语言本身的时间表上进行开发。这些软件包的更新频率通常比如果它们与 Python 的开发进度挂钩时要高。这对于`pytest`以及我们将在本书后半部分使用的大多数库来说都是如此。你不应该盲目相信每一个第三方包，但也不应该因为很多重要功能是通过这些包来实现的而感到排斥。

### 更新 pip

Python 包含了一个叫做 pip 的工具，用于安装第三方包。由于 pip 帮助从外部资源安装包，它会经常更新，以解决潜在的安全问题。因此，我们首先会更新 pip。

打开一个新的终端窗口，并执行以下命令：

```py
$ **python -m pip install --upgrade pip**
❶ Requirement already satisfied: pip in /.../python3.11/site-packages (22.0.4)
`--snip--`
❷ Successfully installed pip-22.1.2
```

这个命令的第一部分 `python -m pip` 告诉 Python 运行 `pip` 模块。第二部分 `install --upgrade` 告诉 pip 更新已经安装的包。最后一部分 `pip` 指定了要更新的第三方包。输出显示我的当前 pip 版本是 22.0.4 ❶，被替换为写作时的最新版本 22.1.2 ❷。

你可以使用这个命令来更新系统上安装的任何第三方包：

```py
$ **python -m pip install --upgrade** `package_name`
```

### 安装 pytest

既然 pip 已经是最新版本，我们可以安装 `pytest`：

```py
$ **python -m pip install --user pytest**
Collecting pytest
 `--snip--`
Successfully installed attrs-21.4.0 iniconfig-1.1.1 ...pytest-7.`x`.`x`
```

我们仍然使用核心命令 `pip install`，这次没有 `--upgrade` 标志。相反，我们使用了 `--user` 标志，它告诉 Python 只为当前用户安装这个包。输出显示 `pytest` 的最新版本已成功安装，并且安装了 `pytest` 依赖的其他一些包。

你可以使用这个命令来安装许多第三方包：

```py
$ **python -m pip install --user** `package_name`
```

## 测试一个函数

要了解测试，我们需要一些代码来进行测试。这里有一个简单的函数，它接受名字和姓氏，并返回格式化的全名：

**name_function.py**

```py
def get_formatted_name(first, last):
    """Generate a neatly formatted full name."""
    full_name = f"{first} {last}"
    return full_name.title()
```

`get_formatted_name()` 函数将名字和姓氏结合，中间用空格隔开，完成全名的拼接，然后将全名首字母大写并返回。为了检查 `get_formatted_name()` 是否正常工作，让我们编写一个使用此函数的程序。程序 *names.py* 让用户输入名字和姓氏，并显示格式化的全名：

**names.py**

```py
from name_function import get_formatted_name

print("Enter 'q' at any time to quit.")
while True:
    first = input("\nPlease give me a first name: ")
    if first == 'q':
        break
    last = input("Please give me a last name: ")
    if last == 'q':
        break

    formatted_name = get_formatted_name(first, last)
    print(f"\tNeatly formatted name: {formatted_name}.")
```

这个程序从 *name_function.py* 导入 `get_formatted_name()`。用户可以输入一系列名字和姓氏，并查看生成的格式化全名：

```py
Enter 'q' at any time to quit.

Please give me a first name: **janis**
Please give me a last name: **joplin**
       Neatly formatted name: Janis Joplin.

Please give me a first name: **bob**
Please give me a last name: **dylan**
        Neatly formatted name: Bob Dylan.

Please give me a first name: **q**
```

我们可以看到这里生成的名字是正确的。但是假设我们想修改 `get_formatted_name()` 以便它也能处理中间名。在我们做这项修改时，我们希望确保不会破坏函数对只有名字和姓氏的处理方式。我们可以通过运行 *names.py* 并每次修改 `get_formatted_name()` 时输入一个名字，如 `Janis Joplin`，来测试我们的代码，但这样会变得很繁琐。幸运的是，`pytest` 提供了一种有效的方法来自动化测试函数的输出。如果我们自动化测试 `get_formatted_name()`，我们就可以始终确保在输入我们已编写测试的名字时，函数能够正常工作。

### 单元测试和测试用例

软件测试有许多种方法。最简单的测试之一就是单元测试。*单元测试* 验证函数行为的某一特定方面是否正确。*测试用例* 是一组单元测试，它们共同证明函数在你期望的所有情境下的行为是正确的。

一个好的测试用例会考虑函数可能接收到的所有输入类型，并包括能够代表这些情况的测试。一个*完整覆盖*的测试用例包括对所有可能使用方式的单元测试。对于大型项目而言，达到完全覆盖可能是一个巨大的挑战。通常，只需编写针对代码关键行为的测试，只有当项目开始广泛使用时，才需要追求完全覆盖。

### 通过测试

使用`pytest`，编写你的第一个单元测试是非常直接的。我们将编写一个简单的测试函数。测试函数将调用我们要测试的函数，然后我们会对返回的值进行断言。如果断言正确，测试将通过；如果断言错误，测试将失败。

这是`get_formatted_name()`函数的第一个测试：

**test_name_function.py**

```py
from name_function import get_formatted_name

❶ def test_first_last_name():
    """Do names like 'Janis Joplin' work?"""
❷     formatted_name = get_formatted_name('janis', 'joplin')
❸     assert formatted_name == 'Janis Joplin'
```

在我们运行测试之前，让我们仔细看看这个函数。测试文件的名称很重要；它必须以*test_*开头。当我们请求`pytest`运行我们编写的测试时，它会查找任何以*test_*开头的文件，并运行该文件中的所有测试。

在测试文件中，我们首先导入我们要测试的函数：`get_formatted_name()`。然后我们定义一个测试函数：在这种情况下是`test_first_last_name()` ❶。这个函数名比我们之前使用的要长，原因是好的。首先，测试函数需要以*test*开头，后跟一个下划线。任何以`test_`开头的函数都将被`pytest`*发现*，并在测试过程中运行。

此外，测试名称应该比常规函数名更长且更具描述性。你不会自己调用这个函数；`pytest`会找到并为你运行它。测试函数名应该足够长，以便当你在测试报告中看到函数名时，能够清楚地了解测试的行为。

接下来，我们调用我们正在测试的函数 ❷。在这里，我们用`'janis'`和`'joplin'`作为参数调用`get_formatted_name()`，就像我们运行*names.py*时使用的那样。我们将此函数的返回值赋给`formatted_name`。

最后，我们进行一个断言 ❸。*断言*是对某个条件的声明。在这里，我们声明`formatted_name`的值应该是`'Janis Joplin'`。

### 运行测试

如果你直接运行文件*test_name_function.py*，不会得到任何输出，因为我们从未调用测试函数。相反，我们将让`pytest`为我们运行测试文件。

为此，打开一个终端窗口并导航到包含测试文件的文件夹。如果你使用的是 VS Code，你可以打开包含测试文件的文件夹，并使用编辑器窗口内嵌的终端。在终端窗口中，输入命令`pytest`。你应该会看到如下内容：

```py
$ **pytest**
========================= test session starts =========================
❶ platform darwin -- Python 3.`x`.`x`, pytest-7.`x`.`x`, pluggy-1.`x`.`x`
❷ rootdir: /.../python_work/chapter_11
❸ collected 1 item

❹ test_name_function.py .                                          [100%]
========================== 1 passed in 0.00s ==========================
```

让我们试着理解这个输出。首先，我们看到一些关于测试运行系统的信息❶。我是在 macOS 系统上进行测试的，所以你可能会看到不同的输出。最重要的是，我们可以看到用于运行测试的 Python、`pytest`和其他包的版本。

接下来，我们看到测试运行的目录❷：在我的情况下是*python_work/chapter_11*。我们可以看到`pytest`找到了一个测试来运行❸，并且可以看到正在运行的测试文件❹。文件名后面的单个点告诉我们有一个测试通过了，而`100%`则清楚地表示所有测试都已运行。一个大项目可能有数百个或数千个测试，点和完成百分比指示器可以帮助监控测试运行的整体进度。

最后一行告诉我们有一个测试通过了，并且运行测试的时间不到 0.01 秒。

这个输出表示，`get_formatted_name()`函数对于包含名字和姓氏的姓名始终有效，除非我们修改该函数。当我们修改`get_formatted_name()`时，可以再次运行这个测试。如果测试通过，我们就知道该函数仍然适用于像 Janis Joplin 这样的名字。

### 一个失败的测试

失败的测试是什么样子的？让我们修改`get_formatted_name()`，使其能够处理中间名，但我们将以一种破坏只包含名字和姓氏的人的功能的方式进行修改，就像 Janis Joplin 那样。

这是一个新的`get_formatted_name()`版本，它需要一个中间名参数：

**name_function.py**

```py
def get_formatted_name(first, middle, last):
    """Generate a neatly formatted full name."""
    full_name = f"{first} {middle} {last}"
    return full_name.title()
```

这个版本应该适用于有中间名的人，但是当我们测试时，发现它已经破坏了只包含名字和姓氏的人的功能。

这次，运行`pytest`会给出以下输出：

```py
$ **pytest**
========================= test session starts =========================
`--snip--`
❶ test_name_function.py F                                          [100%]
❷ ============================== FAILURES ===============================
❸ ________________________ test_first_last_name _________________________
    def test_first_last_name():
        """Do names like 'Janis Joplin' work?"""
❹ >       formatted_name = get_formatted_name('janis', 'joplin')
❺ E       TypeError: get_formatted_name() missing 1 required positional
            argument: 'last'

test_name_function.py:5: TypeError
======================= short test summary info =======================
FAILED test_name_function.py::test_first_last_name - TypeError:
    get_formatted_name() missing 1 required positional argument: 'last'
========================== 1 failed in 0.04s ==========================
```

这里有很多信息，因为当测试失败时，你可能需要知道很多东西。输出中的第一个重点是一个单独的`F`❶，它告诉我们有一个测试失败。接着，我们看到一个专注于`FAILURES`❷的部分，因为失败的测试通常是测试运行中最重要的关注点。接下来，我们看到`test_first_last_name()`是失败的测试函数❸。一个尖括号❹指示了导致测试失败的代码行。下一行的`E`❺显示了导致失败的实际错误：由于缺少必需的位置参数`last`，出现了`TypeError`。最重要的信息在最后的简短总结中得到了重复，因此当你运行多个测试时，可以快速了解哪些测试失败以及原因。

### 响应失败的测试

当测试失败时，你该怎么办？假设你正在检查正确的条件，测试通过意味着函数行为正确，而测试失败意味着你写的新代码中有错误。所以，当测试失败时，不要改变测试。如果你这么做，测试可能会通过，但任何像测试一样调用你的函数的代码会突然停止工作。相反，应该修复导致测试失败的代码。检查你刚刚对函数所做的修改，弄清楚这些修改是如何破坏预期行为的。

在这种情况下，`get_formatted_name()` 以前只需要两个参数：一个名和一个姓。现在它需要一个名、一个中间名和一个姓。增加了这个必需的中间名参数后，打破了 `get_formatted_name()` 原来的行为。这里的最佳选择是让中间名变成可选项。一旦我们这么做，像 `Janis Joplin` 这样的姓名测试应该能重新通过，我们也应该能够接受中间名。让我们修改 `get_formatted_name()`，使中间名成为可选项，然后再次运行测试用例。如果通过了，我们就可以继续确保该函数能够正确处理中间名。

为了让中间名成为可选项，我们将参数 `middle` 移到函数定义的参数列表末尾，并赋予它一个空的默认值。我们还添加了一个 `if` 语句，根据是否提供中间名来正确构建全名：

**name_function.py**

```py
def get_formatted_name(first, last, middle=''):
    """Generate a neatly formatted full name."""
    if middle:
        full_name = f"{first} {middle} {last}"
    else:
        full_name = f"{first} {last}"
    return full_name.title()
```

在这个新版本的 `get_formatted_name()` 中，中间名是可选的。如果传入了中间名，返回的全名将包含名、中间名和姓。否则，全名将仅包含名和姓。现在该函数应该能同时适用于两种类型的姓名。为了验证函数是否仍然适用于像 `Janis Joplin` 这样的姓名，让我们再次运行测试：

```py
$ **pytest**
========================= test session starts =========================
`--snip--`
test_name_function.py .                                       [100%]
========================== 1 passed in 0.00s ==========================
```

测试现在通过了。这是理想的情况；意味着该函数又能处理像 `Janis Joplin` 这样的姓名，而不需要我们手动测试函数。修复我们的函数更容易，因为失败的测试帮助我们确定了新代码如何破坏了现有的行为。

### 添加新测试

现在我们知道 `get_formatted_name()` 又能处理简单姓名了，让我们为包含中间名的人编写第二个测试。我们通过在文件 *test_name_function.py* 中添加另一个测试函数来做到这一点：

**test_name_function.py**

```py
from name_function import get_formatted_name

def test_first_last_name():
  *--snip--*

def test_first_last_middle_name():
    """Do names like 'Wolfgang Amadeus Mozart' work?"""
❶     formatted_name = get_formatted_name(
        'wolfgang', 'mozart', 'amadeus')
❷     assert formatted_name == 'Wolfgang Amadeus Mozart'
```

我们将这个新函数命名为 `test_first_last_middle_name()`。函数名必须以 `test_` 开头，这样我们运行 `pytest` 时它就会自动运行。我们为函数命名是为了明确指出我们正在测试 `get_formatted_name()` 的哪个行为。因此，如果测试失败，我们会立刻知道是哪些类型的姓名受到了影响。

为了测试该函数，我们调用 `get_formatted_name()`，并传入一个名、一个姓和一个中间名 ❶，然后我们进行断言 ❷，验证返回的全名是否与我们预期的全名（名、中间名和姓）匹配。当我们再次运行 `pytest` 时，两个测试都通过了：

```py
$ pytest
========================= test session starts =========================
`--snip--`
collected 2 items

❶ test_name_function.py ..                                         [100%]
========================== 2 passed in 0.01s ==========================
```

两个点❶表示两个测试通过，这也从输出的最后一行可以看出。这很棒！我们现在知道这个函数对像 `Janis Joplin` 这样的名字依然有效，并且可以放心它同样适用于像 `Wolfgang Amadeus Mozart` 这样的名字。

## 测试一个类

本章的第一部分中，你为单个函数编写了测试。现在你将为一个类编写测试。你将在自己的许多程序中使用类，因此能够证明你的类正常工作是非常有帮助的。如果你为正在处理的类编写了通过的测试，你可以放心地做出改进而不会意外地破坏当前的行为。

### 各种断言

到目前为止，你只见过一种类型的断言：声明一个字符串具有特定的值。在编写测试时，你可以做出任何可以表示为条件语句的声明。如果条件按预期为 `True`，则你关于程序某部分行为的假设将得到确认；你可以确信没有错误。如果你假设的条件是 `True`，实际上为 `False`，则测试将失败，你会知道存在需要解决的问题。表 11-1 显示了你可以在初始测试中包含的一些最有用的断言类型。

表 11-1：测试中常用的断言语句

| **断言** | **声明** |
| --- | --- |
| `assert a == b` | 断言两个值相等。 |
| `assert a != b` | 断言两个值不相等。 |
| `assert a` | 断言 `a` 的值为 `True`。 |
| `assert not a` | 断言 `a` 的值为 `False`。 |
| `assert` `element` `in` `list` | 断言一个元素在列表中。 |
| `assert` `element` `not in` `list` | 断言一个元素不在列表中。 |

这些只是一些例子；任何可以表示为条件语句的内容都可以包含在测试中。

### 一个用于测试的类

测试一个类与测试一个函数类似，因为大部分工作都涉及测试类中方法的行为。然而，仍然存在一些差异，因此让我们编写一个类来进行测试。考虑一个帮助进行匿名调查的类：

**survey.py**

```py
class AnonymousSurvey:
    """Collect anonymous answers to a survey question."""

❶     def __init__(self, question):
        """Store a question, and prepare to store responses."""
        self.question = question
        self.responses = []

❷     def show_question(self):
        """Show the survey question."""
        print(self.question)

❸     def store_response(self, new_response):
        """Store a single response to the survey."""
        self.responses.append(new_response)

❹     def show_results(self):
        """Show all the responses that have been given."""
        print("Survey results:")
        for response in self.responses:
            print(f"- {response}")
```

这个类从你提供的调查问题❶开始，并包括一个空列表用于存储回应。这个类有方法来打印调查问题❷、将新的回应添加到回应列表❸，并打印存储在列表中的所有回应❹。要创建该类的实例，你只需要提供一个问题。一旦你有了表示特定调查的实例，你可以使用 `show_question()` 显示调查问题，使用 `store_response()` 存储回应，并使用 `show_results()` 显示结果。

为了展示 `AnonymousSurvey` 类的工作原理，让我们编写一个使用该类的程序：

**language_survey.py**

```py
from survey import AnonymousSurvey

# Define a question, and make a survey.
question = "What language did you first learn to speak?"
language_survey = AnonymousSurvey(question)

# Show the question, and store responses to the question.
language_survey.show_question()
print("Enter 'q' at any time to quit.\n")
while True:
    response = input("Language: ")
    if response == 'q':
        break
    language_survey.store_response(response)

# Show the survey results.
print("\nThank you to everyone who participated in the survey!")
language_survey.show_results()
```

这个程序定义了一个问题（`"你第一次学会说的是什么语言？"`），并创建了一个包含该问题的`AnonymousSurvey`对象。程序调用`show_question()`显示问题，然后提示输入回答。每个回答都会在收到时保存。当所有回答输入完成后（用户输入`q`退出），`show_results()`将打印调查结果：

```py
What language did you first learn to speak?
Enter 'q' at any time to quit.

Language: **English**
Language: **Spanish**
Language: **English**
Language: **Mandarin**
Language: **q**

Thank you to everyone who participated in the survey!
Survey results:
- English
- Spanish
- English
- Mandarin
```

这个类适用于一个简单的匿名调查，但假设我们想要改进`AnonymousSurvey`及其所在的模块`survey`。我们可以允许每个用户输入多个回答，编写一个方法仅列出唯一回答并报告每个回答出现的次数，甚至可以编写另一个类来管理非匿名调查。

实现这样的更改可能会影响当前`AnonymousSurvey`类的行为。例如，在尝试允许每个用户输入多个回答时，我们可能会不小心改变单个回答的处理方式。为了确保我们在开发该模块时不会破坏现有行为，我们可以为该类编写测试。

### 测试`AnonymousSurvey`类

让我们编写一个测试，验证`AnonymousSurvey`行为的一个方面。我们将编写一个测试来验证单个回答是否能正确存储在调查中：

**test_survey.py**

```py
from survey import AnonymousSurvey

❶ def test_store_single_response():
    """Test that a single response is stored properly."""
    question = "What language did you first learn to speak?"
❷     language_survey = AnonymousSurvey(question)
    language_survey.store_response('English')
❸     assert 'English' in language_survey.responses
```

我们首先导入要测试的类`AnonymousSurvey`。第一个测试函数将验证当我们存储一个调查问题的回答时，该回答是否会出现在调查的回答列表中。这个函数的一个好描述性名称是`test_store_single_response()` ❶。如果这个测试失败，我们可以从测试总结中的函数名称了解到存储单个回答时出现了问题。

要测试一个类的行为，我们需要创建该类的实例。我们创建一个名为`language_survey`的实例 ❷，并赋予问题`"你第一次学会说的是什么语言？"`。我们使用`store_response()`方法存储一个回答`English`。然后，我们通过断言`English`是否在`language_survey.responses`列表中来验证该回答是否已正确存储 ❸。

默认情况下，运行命令`pytest`且不带任何参数时，会运行`pytest`在当前目录中发现的所有测试。要集中测试某个文件中的测试，可以传入要运行的测试文件名。这里我们将仅运行我们为`AnonymousSurvey`编写的一个测试：

```py
$ **pytest test_survey.py**
========================= test session starts =========================
`--snip--`
test_survey.py .                                                 [100%]
========================== 1 passed in 0.01s ==========================
```

这是一个很好的开始，但一个调查只有生成多个回答时才有意义。让我们验证是否可以正确存储三个回答。为此，我们向`TestAnonymousSurvey`添加另一个方法：

```py
from survey import AnonymousSurvey

def test_store_single_response():
 *--snip--*

def test_store_three_responses():
    """Test that three individual responses are stored properly."""
    question = "What language did you first learn to speak?"
    language_survey = AnonymousSurvey(question)
❶     responses = ['English', 'Spanish', 'Mandarin']
    for response in responses:
        language_survey.store_response(response)

❷     for response in responses:
        assert response in language_survey.responses
```

我们将新函数命名为`test_store_three_responses()`。我们像在`test_store_single_response()`中一样创建一个调查对象。我们定义了一个包含三个不同回答的列表❶，然后对每个回答调用`store_response()`。一旦回答被存储，我们再写一个循环，并断言每个回答现在都在`language_survey.responses`中❷。

当我们再次运行测试文件时，两个测试（一个是单个回答，另一个是三个回答）都通过了：

```py
$ **pytest test_survey.py**
========================= test session starts =========================
`--snip--`
test_survey.py ..                                                [100%]
========================== 2 passed in 0.01s ==========================
```

这完美地工作。然而，这些测试有些重复，所以我们将使用`pytest`的另一个特性来使它们更高效。

### 使用 Fixtures

在*test_survey.py*中，我们在每个测试函数中都创建了一个新的`AnonymousSurvey`实例。这在我们处理的短小示例中没问题，但在一个有数十或数百个测试的实际项目中，这将成为一个问题。

在测试中，*fixture*帮助设置测试环境。通常，这意味着创建一个由多个测试使用的资源。我们通过写一个带有`@pytest.fixture`装饰器的函数来在`pytest`中创建一个 fixture。*装饰器*是一个指令，位于函数定义之前；Python 在运行函数之前应用这个指令，以改变函数代码的行为。如果这听起来很复杂，不用担心；你可以先使用第三方包的装饰器，而不必先学会自己写装饰器。

让我们使用一个 fixture 来创建一个可以在*test_survey.py*中的两个测试函数中使用的单一调查实例：

```py
import pytest
from survey import AnonymousSurvey

❶ @pytest.fixture
❷ def language_survey():
    """A survey that will be available to all test functions."""
 question = "What language did you first learn to speak?"
    language_survey = AnonymousSurvey(question)
    return language_survey

❸ def test_store_single_response(language_survey):
    """Test that a single response is stored properly."""
❹     language_survey.store_response('English')
 assert 'English' in language_survey.responses

❺ def test_store_three_responses(language_survey):
    """Test that three individual responses are stored properly."""
    responses = ['English', 'Spanish', 'Mandarin']
 for response in responses:
❻ language_survey.store_response(response)

 for response in responses:
 assert response in language_survey.responses
```

现在我们需要导入`pytest`，因为我们使用了一个在`pytest`中定义的装饰器。我们将`@pytest.fixture`装饰器❶应用到新的函数`language_survey()`❷上。这个函数构建了一个`AnonymousSurvey`对象，并返回新的调查。

请注意，两个测试函数的定义发生了变化❸❺；每个测试函数现在都有一个名为`language_survey`的参数。当测试函数中的参数与带有`@pytest.fixture`装饰器的函数名称匹配时，fixture 会自动运行，并将返回值传递给测试函数。在这个示例中，`language_survey()`函数为`test_store_single_response()`和`test_store_three_responses()`提供了一个`language_survey`实例。

两个测试函数中没有新增代码，但请注意，每个函数中有两行代码被移除❹❻：定义问题的那一行和创建`AnonymousSurvey`对象的那一行。

当我们再次运行测试文件时，两个测试仍然通过。这些测试在尝试扩展`AnonymousSurvey`以处理每个人的多个回答时尤其有用。在修改代码以接受多个回答后，您可以运行这些测试，确保没有影响存储单个回答或一系列个体回答的功能。

上述结构看起来几乎肯定会很复杂；它包含了一些你到目前为止见过的最抽象的代码。你不需要立刻使用 fixtures；与其完全不写测试，不如写一些有很多重复代码的测试。只要记住，当你写了足够多的测试，重复的代码开始成为障碍时，有一种成熟的方法可以处理这些重复。而且像这种简单的例子中的 fixture 并不会使代码更简洁或更容易理解。但在有许多测试的项目中，或者在需要多行代码来构建多个测试所用资源的情况下，fixtures 能显著提升你的测试代码质量。

当你想编写一个 fixture 时，写一个生成多个测试函数所用资源的函数。在新函数上添加`@pytest.fixture`装饰器，并将该函数的名称作为参数添加到每个使用该资源的测试函数中。从那时起，你的测试会更简洁，更容易编写和维护。

## 总结

在本章中，你学会了使用 `pytest` 模块中的工具为函数和类编写测试。你学会了编写测试函数来验证你的函数和类应该表现出的特定行为。你还了解了如何使用 fixtures 高效地创建可以在测试文件中多个测试函数中使用的资源。

测试是一个重要的主题，但许多新手程序员并未接触过。你不需要为所有简单的项目编写测试，尤其是作为新手程序员时。但一旦你开始从事涉及重大开发工作的项目，你应该测试函数和类的关键行为。这样你会更有信心，知道对项目进行的新工作不会破坏现有功能，并且这将让你有自由改进代码。如果你不小心破坏了现有功能，你会立刻发现，这样你仍然能轻松修复问题。响应一个失败的测试远比响应一个来自不满用户的 bug 报告要容易得多。

如果你在项目中加入一些初始测试，其他程序员会更尊重你的项目。他们会更愿意实验你的代码，并且更愿意与你合作。如果你想为其他程序员正在进行的项目做贡献，你需要展示你的代码通过了现有的测试，并且通常需要为你引入的任何新行为编写测试。

玩转测试，熟悉测试代码的编写过程。为你的函数和类的关键行为编写测试，但在早期项目中，除非有特定的原因，不要追求完全覆盖。
