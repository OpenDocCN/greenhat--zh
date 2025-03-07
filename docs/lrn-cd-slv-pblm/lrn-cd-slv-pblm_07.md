## 读取和写入文件

![image](img/common.jpg)

到目前为止，我们已经使用`input`函数读取所有输入，并使用`print`函数输出所有结果。这些函数分别从标准输入（默认为键盘）读取，并将结果写入标准输出（默认为屏幕）。尽管我们可以通过输入和输出重定向改变这些默认值，但有时程序需要更好地控制文件。例如，你的文字处理器允许你打开任何文档文件并保存任何你喜欢的文件，而无需处理标准输入和标准输出。

本章我们将学习如何编写处理文本文件的程序。我们将通过文件解决两个问题：正确格式化文章和播种农场以喂养奶牛。

### 问题 #16：文章格式化

本问题与我们之前解决的所有问题有一个重要区别：这个问题要求我们从特定的文件中读取并写入数据！在阅读问题描述时要特别注意这一点。

这是 USACO 2020 年 1 月青铜组比赛问题《文字处理器》。这是 USACO（美国计算机奥林匹克）评委书中的第一个问题。要查看该问题，请访问*[`usaco.org/`](http://usaco.org/)*，点击**Contests**，点击**2020 January Contest Results**，然后点击**View problem**下的《文字处理器》。

#### 挑战

贝西（Bessie）奶牛正在写一篇文章。文章中的每个单词仅包含小写字母或大写字母。她的老师规定了每行的最大字符数（不包括空格）。为了满足这一要求，贝西使用以下规则写下文章中的单词：

+   如果下一个单词适合当前行，则将其添加到当前行。在每对单词之间包括一个空格。

+   否则，将该单词放到新的一行；这一行成为新的当前行。

输出文章，并确保每行包含正确的单词。

#### 输入

从名为*word.in*的文件中读取输入。

输入由两行组成。

+   第一行包含两个由空格分隔的整数。第一个整数是*n*，文章中的单词数；它介于 1 和 100 之间。第二个整数是*k*，每行允许的最大字符数（不包括空格）；它介于 1 和 80 之间。

+   第二行包含*n*个单词，每对单词之间有一个空格。每个单词最多有*k*个字符。

#### 输出

将输出写入名为*word.out*的文件中。

输出格式化正确的文章。

### 与文件打交道

《文章格式化》问题要求我们从文件*word.in*中读取，并写入文件*word.out*。但是，在进行这些操作之前，我们需要学习如何在程序中打开文件。

#### 打开文件

使用你的文本编辑器创建一个名为*word.in*的新文件。将该文件放在与*.py* Python 程序相同的目录中。

这是我们第一次创建一个不以*.py*结尾的文件。相反，它以*.in*结尾。确保将文件命名为*word.in*，而不是*word.py*。*in*是“输入”的缩写，你将经常看到它用于包含程序输入的文件。

在这个文件中，让我们为“论文格式化”问题放入有效的输入。将以下内容输入文件中：

```py
12 13

perhaps better poetry will be written in the language of digital computers
```

保存文件。

要在 Python 中打开文件，我们使用`open`函数。我们传递两个参数：第一个是文件名，第二个是打开文件的模式。模式决定了我们如何与文件进行交互。

下面是我们如何打开*word.in*：

```py
   >>> open('word.in', 'r')

❶ <_io.TextIOWrapper name='word.in' mode='r' encoding='cp1252'>
```

在这个函数调用中，我们提供了一个模式`'r'`。`r`代表“读取”，并以此模式打开文件，供我们从中读取。模式是一个可选的参数，其默认值为`'r'`，所以我们可以选择省略它。但为了保持一致性，我将在全书中明确地包括`'r'`。

当我们使用`open`时，Python 会给我们一些关于文件如何打开的信息❶。例如，它会确认文件名和模式。关于`encoding`的部分表示文件是如何从磁盘上的状态解码成我们可以读取的形式。文件可以使用多种编码方式进行编码，但在本书中我们不需要担心编码问题。

如果我们尝试打开一个不存在的文件进行读取，就会出现错误：

```py
>>> open('blah.in', 'r')

Traceback (most recent call last):

  File "<stdin>", line 1, in <module>

FileNotFoundError: [Errno 2] No such file or directory: 'blah.in'
```

如果在打开*word.in*时遇到此错误，请仔细检查文件是否正确命名，并且位于你启动 Python 时的目录中。

除了用于读取的模式`'r'`，还有用于写入的模式`'w'`。如果我们使用`'w'`，那么我们就是在打开一个文件以便往其中写入文本。

小心使用模式`'w'`。如果你用`'w'`打开一个已经存在的文件，该文件的内容将被删除。我刚刚不小心在我的*word.in*文件上做了这件事。没关系，因为重新创建它很容易。但如果我们不小心覆盖了一个重要文件，没人会开心。

如果你使用`'w'`打开一个不存在的文件，它会创建一个空文件。

让我们使用模式`'w'`来创建一个名为*blah.in*的空文件：

```py
>>> open('blah.in', 'w')

<_io.TextIOWrapper name='blah.in' mode='w' encoding='cp1252'>
```

现在*blah.in*已存在，我们可以打开它进行读取而不会出错：

```py
>>> open('blah.in', 'r')

<_io.TextIOWrapper name='blah.in' mode='r' encoding='cp1252'>
```

那个我们一直看到的`_io.TextIOWrapper`是什么？那是`open`返回的值的类型：

```py
>>> type(open('word.in', 'r'))

<class '_io.TextIOWrapper'>
```

把这个类型想象成文件类型。它的值表示打开的文件，而且你很快就会看到它有我们可以调用的方法。

和任何函数一样，如果我们不将`open`返回的内容赋值给一个变量，那么它的返回值就会丢失。到目前为止，我们调用`open`的方式并没有提供任何方式来引用我们打开的文件！

下面是我们如何让一个变量指向一个已打开的文件：

```py
>>> input_file = open('word.in', 'r')

>>> input_file

<_io.TextIOWrapper name='word.in' mode='r' encoding='cp1252'>
```

我们将能够使用`input_file`从`'word.in'`中读取数据。

在解决论文格式化问题时，我们还需要一种方法来写入文件`'word.out'`。下面是一个有助于我们做到这一点的变量：

```py
>>> output_file = open('word.out', 'w')

>>> output_file

<_io.TextIOWrapper name='word.out' mode='w' encoding='cp1252'>
```

#### 从文件中读取

要从打开的文件中读取一行，我们使用文件的`readline`方法。该方法返回一个包含文件下一行内容的字符串。这样，它类似于`input`函数。然而，与`input`不同的是，`readline`是从文件中读取，而不是从标准输入中读取。

让我们打开*word.in*并读取其中的两行：

```py
>>> input_file = open('word.in', 'r')

>>> input_file.readline()

'12 13\n'

>>> input_file.readline()

'perhaps better poetry will be written in the language of digital computers\n'
```

这里的意外之处在于每个字符串末尾的`\n`。当我们使用`input`读取一行时，显然没有看到这个符号。字符串中的`\`符号是一个*转义字符*。它从字符的标准解释中跳脱出来，改变它们的含义。我们并不将`\n`视为两个独立的字符`\`和`n`。相反，`\n`是一个字符：换行符。文件中的所有行（可能除了最后一行）都以换行符结尾。如果没有换行符，那么所有内容都会显示在同一行！`readline`方法实际上返回了整个行，包括它的结束换行符。

这是我们如何在自己的字符串中嵌入换行符：

```py
>>> 'one\ntwo\nthree'

'one\ntwo\nthree'

>>> print('one\ntwo\nthree')

one

two

three
```

Python shell 不处理转义字符的效果，但`print`会处理。

`\n`序列在字符串中很有用，因为它帮助我们添加多行。但我们很少希望在从文件读取的行中出现这些换行符。为了去除它们，我们可以使用字符串的`rstrip`方法。这个方法类似于`strip`，不过它只删除字符串右侧的空白字符（而不是左侧）。在它看来，换行符就像空格一样，都是空白字符：

```py
>>> 'hello\nthere\n\n'

'hello\nthere\n\n'

>>> 'hello\nthere\n\n'.rstrip()

'hello\nthere'
```

让我们再试着从文件中读取，这次去掉换行符：

```py
>>> input_file = open('word.in', 'r')

>>> input_file.readline().rstrip()

'12 13'

>>> input_file.readline().rstrip()

'perhaps better poetry will be written in the language of digital computers'
```

到此为止，我们已经读取了两行，所以文件中没有剩余的内容可读。`readline`方法通过返回一个空字符串来表示这一点。

```py
>>> input_file.readline().rstrip()

''
```

空字符串意味着我们已经到达了文件末尾。如果我们想再次读取这些行，我们必须重新打开文件，从头开始。

让我们这么做，这次使用变量保存每一行：

```py
>>> input_file = open('word.in', 'r')

>>> first = input_file.readline().rstrip()

>>> second = input_file.readline().rstrip()

>>> first

'12 13'

>>> second

'perhaps better poetry will be written in the language of digital computers'
```

如果我们需要读取文件中的所有行，不论有多少行，我们可以使用`for`循环。Python 中的文件就像行的序列，因此我们可以像遍历字符串和列表一样遍历文件：

```py
>>> input_file = open('word.in', 'r')

>>> for line in input_file:

...     print(line.rstrip())

...

12 13

perhaps better poetry will be written in the language of digital computers
```

然而，与字符串或循环不同的是，我们不能第二次循环读取文件，因为第一次循环已经把文件读取到了末尾。如果我们尝试这样做，就什么也得不到：

```py
>>> for line in input_file:

...     print(line.rstrip())

...
```

**概念检查**

我们想要使用`while`循环输出打开的文件`input_file`中的每一行。（该文件可以是任何文件；我并不假设它与论文格式化有关。）下面哪段代码正确地完成了这项任务？

A.

while input_file.readline() != '':

print(input_file.readline().rstrip())

B.

line = 'x'

while line != '':

line = input_file.readline()

print(line.rstrip())

C.

line = input_file.readline()

while line != '':

line = input_file.readline()

print(line.rstrip())

D. 以上所有

E. 以上都不是

在查看答案之前，我建议你创建一个包含四五行文本的文件，并在文件上尝试每一段代码。你还可以考虑在每一行输出的开头添加一个字符，比如`*`，这样你就能看到任何原本为空的行。

答案：E。每一段代码都有一个微妙的错误。

代码 A 只输出文件中的每隔一行。例如，`while`循环的布尔表达式导致第一行被读取……并丢失，因为它没有被赋值给变量。因此，循环的第一次迭代输出的是文件的第二行。

代码 B 非常接近正确的做法。它输出了文件的所有行，但在末尾多输出了一个空白行。

代码 C 未能打印文件的第一行。这是因为第一行在循环前就已被读取，但之后循环读取了第二行，且没有打印第一行。它同样在末尾产生了多余的空白行，就像代码 B 一样。

这是正确的代码，用来读取并打印每一行：

```py
line = input_file.readline()

while line != '':

    print(line.rstrip())

    line = input_file.readline()
```

#### 写入文件

要向一个打开的文件写入一行，我们使用文件的`write`方法。我们传递给它一个字符串，这个字符串会被添加到文件的末尾。

为了解决作文格式化问题，我们将写入*word.out*。但我们还没有准备好解决这个问题，所以我们暂时写入*blah.out*。以下是我们如何向该文件写入一行：

```py
>>> output_file = open('blah.out', 'w')

>>> output_file.write('hello')

5
```

那个`5`在那里干什么？`write`方法返回的是写入的字符数。这是一个很好的确认，表示我们已经写入了预期的文本数量。

如果你在文本编辑器中打开*blah.out*，你应该看到文件中有`hello`这个文本。

让我们尝试向文件写入三行。开始吧：

```py
>>> output_file = open('blah.out', 'w')

>>> output_file.write('sq')

2

>>> output_file.write('ui')

2

>>> output_file.write('sh')

2
```

根据我到目前为止告诉你的内容，你可能会期望*blah.out*看起来像这样：

```py
sq

ui

sh
```

但是，如果你在文本编辑器中打开*blah.out*，你应该看到如下内容：

```py
squish
```

字符之所以显示在同一行，是因为`write`方法不会为我们自动添加换行符！如果我们想要单独的行，我们需要明确指定，就像这样：

```py
>>> output_file = open('blah.out', 'w')

>>> output_file.write('sq\n')

3

>>> output_file.write('ui\n')

3

>>> output_file.write('sh\n')

3
```

注意，在每种情况下，`write`方法写入的是三个字符，而不是两个。换行符也算作一个字符。现在，如果你在文本编辑器中打开*blah.out*，你应该看到文本分布在三行中：

```py
sq

ui

sh
```

与`print`不同，`write`只有在你用字符串调用它时才有效。要向文件写入数字，首先需要将其转换为字符串：

```py
>>> num = 7788

>>> output_file = open('blah.out', 'w')

>>> output_file.write(str(num) + '\n')

5
```

##### 关闭文件

在完成文件操作后关闭文件是一个好习惯。这向阅读你代码的人表明，文件不再被使用。

关闭文件还可以帮助操作系统管理计算机的资源。当你使用`write`方法时，你写入的内容可能不会立即写入文件。相反，Python 或者操作系统可能会等到有多个`write`请求时，再将它们一次性写入。关闭你写入的文件可以确保你写入的内容已安全地存储在文件中。

要关闭文件，调用其`close`方法。以下是打开文件、读取一行并关闭文件的示例：

```py
>>> input_file = open('word.in', 'r')

>>> input_file.readline()

'12 13\n'

>>> input_file.close()
```

一旦你关闭了文件，就不能再从该文件读取或写入：

```py
>>> input_file.readline()

Traceback (most recent call last):

  File "<stdin>", line 1, in <module>

ValueError: I/O operation on closed file.
```

### 解决问题

回到文章格式化问题。现在我们知道如何从*word.in*读取并写入到*word.out*。这样就完成了输入输出要求。接下来是解决问题本身。

让我们首先通过探索一个测试用例来确保我们知道如何解决这个问题。然后我们会看到代码。

#### 探索一个测试用例

这是我一直在使用的*word.in*文件：

```py
12 13

perhaps better poetry will be written in the language of digital computers
```

共有 12 个单词，每行最大字符数（不包括空格）是 13 个。只要单词能放下，我们就将它添加到当前行；一旦单词放不下，我们就用这个单词开始新的一行。

单词`perhaps`包含七个字符，因此可以放在第一行。单词`better`包含六个字符。我们也可以把它放在第一行；由于`perhaps`已经在那，两个单词加起来总共有 13 个字符（不包括单词间的空格）。

单词`poetry`不能放在第一行，所以我们以`poetry`作为第一单词开始新的一行。接着，单词`will`适合放在第二行与`poetry`并排。类似地，`be`可以放在`will`后面。到目前为止，我们已经有了 12 个非空格字符。现在我们有了单词`written`，由于第二行只剩下一个字符的空间，我们只能在新的一行开始时把`written`作为第一个单词。

按照这个过程继续直到结束，我们需要写入*word.out*的完整文章是：

```py
perhaps better

poetry will be

written in the

language of

digital

computers
```

#### 代码

我们的解决方案在清单 7-1 中。

```py
❶ input_file = open('word.in', 'r')

❷ output_file = open('word.out', 'w')

❸ lst = input_file.readline().split()

   n = int(lst[0])  # n not needed

   k = int(lst[1])

   words = input_file.readline().split()

❹ line = ''

   chars_on_line = 0

   for word in words:

    ❺ if chars_on_line + len(word) <= k:

           line = line + word + ' '

           chars_on_line = chars_on_line + len(word)

       else:

        ❻ output_file.write(line[:-1] + '\n')

           line = word + ' '

           chars_on_line = len(word)

❼ output_file.write(line[:-1] + '\n')

   input_file.close()

   output_file.close()
```

*清单 7-1: 解决文章格式化问题*

首先，我们打开输入文件❶和输出文件❷。注意文件模式：我们用模式`'r'`（读取模式）打开输入文件，用模式`'w'`（写入模式）打开输出文件。我们本可以稍后再打开输出文件，在需要使用它之前，但为了简化程序结构，我选择在这里同时打开两个文件。同样，我们本可以在不再需要文件时立刻关闭它，但在本书中，我选择在程序的最后一起关闭所有文件。对于操作多个文件的长时间运行程序，你可能希望只在需要时才保持文件打开。

接下来，我们读取输入文件的第一行❸。这一行包含两个用空格分隔的整数：`n`，单词的数量，以及`k`，每行允许的最大字符数（不包括空格）。像处理空格分隔的值一样，我们使用`split`来分割它们。然后我们读取第二行，它包含文章的单词。我们同样使用`split`，这次将单词字符串分割成一个单词列表。这样就处理好了输入。

两个变量驱动程序的主要部分：`line`和`chars_on_line`。`line`变量表示当前行；我们一开始将它设置为空字符串❹。`chars_on_line`变量表示当前行上字符的数量，不包括空格。

你可能会想，为什么我要维持`chars_on_line`变量呢？我们不能直接使用`len(line)`吗？如果我们这么做，会把空格也算入总数，而空格不算在每行允许的字符数内。如果你觉得用`len(line)`会更直观，我鼓励你自己尝试一下，通过减去空格的数量来解决这个问题。

现在是时候遍历所有单词了。对于每个单词，我们需要确定它是放在当前行还是下一行。

如果当前行上的非空字符数加上当前单词的字符数不超过`k`，则当前单词适合当前行❺。在这种情况下，我们将单词和一个空格添加到当前行，并更新当前行上的非空字符数。

否则，当前单词不适合当前行。当前行已结束！因此，我们将当前行写入输出文件❻，并更新`line`和`chars_on_line`变量，以反映当前行只有这个单词。

关于`write`调用❻有两点需要注意。首先，`[:-1]`切片的存在是为了防止我们输出行末单词后的空格。其次，你可能会期待我在这里使用 f-string，像这样：

```py
        output_file.write(f'{line[:-1]}\n')
```

然而，在写作时，USACO 评测系统正在运行一个较旧版本的 Python，该版本不支持 f-strings。

为什么我们在循环结束后❼要输出`line`？原因是每次迭代的`for`循环都会确保`line`中包含一个或多个我们尚未输出的单词。考虑一下我们处理每个单词时发生的情况。如果当前单词适合当前行，我们就不输出任何内容。如果当前单词不适合当前行，我们就输出当前行，但不输出下一行的单词。因此，我们需要在循环结束后❼将`line`写入输出文件；否则，文章的最后一行将会丢失。

最后，我们做的事情是关闭两个文件。

写入文件而不是屏幕的一个令人烦恼的方面是，当我们运行程序时，看不到输出。为了查看输出，我们必须在文本编辑器中打开输出文件。

这里有一个小技巧：在开发程序时，使用`print`调用而不是`write`调用，这样所有的输出都将显示在屏幕上。这样可以更容易地找到程序中的错误，并避免在代码和输出文件之间来回切换。一旦你对代码满意了，可以将`print`调用改回`write`调用。然后一定要再做一些测试，以确保所有内容都按照预期写入了文件。

我们准备好提交给 USACO 评测了。把我们的代码发过去吧！所有测试用例都应该通过。

### 问题 #17：农场播种

我们可以使用循环从文件中读取指定数量的行。我们将在这个问题中做到这一点，并且会发现它与使用`input`从标准输入读取数据非常相似。

在第六章中，当我们解决《动作人物》问题时，我们学习了使用函数的自上而下设计。这是一个重要的技能，能够通过组合多个函数来解决一个问题。由于关于文件没有更多要说的内容，我选择了一个既能作为解决问题的场景，又能作为自上而下设计的练习问题。

这是一个具有挑战性的问题。我们首先需要准确理解我们被要求做的事情。之后，我们需要制定解决问题的方法，并仔细思考为什么我们的解决方案是正确的。

这是 USACO 2019 年 2 月青铜级竞赛问题——《大规模重新植草》。

#### 挑战

农夫约翰有*n*个牧场，他想要为这些牧场播种草。牧场编号为 1、2、...、*n*。

农夫约翰有四种不同的草种子，编号为 1、2、3 和 4。他将为每个牧场选择其中一种草种类型。

农夫约翰还有*m*只牛。每只牛有两个最喜欢的牧场，它在这些牧场中吃草。每只牛只关心它的两个最喜欢的牧场，其他牧场无关紧要。为了保持健康饮食，每只牛要求它的两个牧场有不同的草种类型。例如，对于某只给定的牛，如果一个牧场是草种类型 1，另一个是草种类型 4，那么是可以接受的。但如果两个牧场都是草种类型 1，那就不行。

一个牧场可能是多个牛的最喜欢牧场，但保证每个牧场最多只会是三只牛的最喜欢牧场。

确定每个牧场使用的草种类型。每个牧场必须使用 1 到 4 之间的草种类型，并且每只牛的两个最喜欢的牧场必须有不同的草种类型。

#### 输入

从名为*revegetate.in*的文件中读取输入。

输入包括以下几行：

+   一行包含两个由空格分隔的整数。第一个整数是*n*，即牧场的数量；它介于 2 和 100 之间。第二个整数是*m*，即牛的数量；它介于 1 和 150 之间。

+   *m*行，每行给出一只牛的两个最喜欢的牧场编号。这些牧场编号是 1 到*n*之间的整数，且用空格分隔。

#### 输出

将输出写入名为*revegetate.out*的文件。

输出一种有效的牧场播种方式。输出是一行*n*个字符，每个字符为`'1'`、`'2'`、`'3'`或`'4'`。第一个字符是牧场 1 的草种类型，第二个字符是牧场 2 的草种类型，依此类推。

我们可以将这*n*个字符解释为一个整数，包含*n*个数字。例如，如果我们有五个草种类型`'11123'`，那么我们可以将其解释为整数`11123`。

当我们有多种输出选择时，这种整数解释法就派上用场了。如果有多种有效的播种方法，我们必须输出解释为整数时最小的那一种。例如，如果 `'11123'` 和 `'22123'` 都是有效的，我们输出字符串 `'11123'`，因为 `11123` 小于 `22123`。

### 探索测试用例

我们将使用自上而下的设计方法来找到这个问题的解决方案。通过处理一个测试用例，我们可以帮助自己梳理任务。

下面是测试用例：

```py
8 6

5 4

2 4

3 5

4 1

2 1

5 2
```

测试用例的第一行告诉我们有八个牧场，它们从 1 到 8 编号。第一行还告诉我们有六头牛。问题并没有指定牛的编号，所以我从 0 开始给它们编号。每头牛最喜欢的两个牧场在表 7-1 中可以方便查看。

**表 7-1：** 农场播种示例，牛

| **牛** | **牧场 1** | **牧场 2** |
| --- | --- | --- |
| 0 | 5 | 4 |
| 1 | 2 | 4 |
| 2 | 3 | 5 |
| 3 | 4 | 1 |
| 4 | 2 | 1 |
| 5 | 5 | 2 |

在这个问题中，我们需要做出 *n* 个决策。我们应该为牧场 1 选择什么草种？牧场 2 呢？牧场 3 呢？牧场 4 呢？依此类推，一直到牧场 *n*。解决这类问题的一种策略是一次做一个决策，不犯任何错误。如果我们能够完成第 *n* 个决策，并且途中没有犯错，那么我们的解决方案必定是正确的。

让我们从牧场 1 到牧场 8 依次查看，看看能否为每个牧场分配一个草种。我们需要优先选择编号较小的草种，以便最终得到解释为数字时最小的草种。

我们应该为牧场 1 选择什么草种呢？唯一关心牧场 1 的牛是牛 3 和牛 4，所以我们只需要关注这两头牛。如果我们已经为某些牛的牧场选择了草种，那么我们就需要小心选择牧场 1 的草种。我们不希望给同一头牛安排两个草种相同的牧场，因为那样会违反规则！由于我们还没有选择任何草种，所以不管选择什么草种，都不会出错。不过，由于我们希望选择最小的草种，因此我们会选择草种 1。

我将把我们的草种决策记录在表格中。以下是我们刚刚做出的决策，为牧场 1 选择草种 1：

| **牧场** | **草种** |
| --- | --- |
| 1 | 1 |

让我们继续前进。我们应该为牧场 2 选择什么草种呢？关心牧场 2 的牛是牛 1、4 和 5，所以我们集中在这些牛上。牛 4 的一个牧场是牧场 1，我们为那个牧场选择了草种 1，所以草种 1 被排除为牧场 2 的草种。如果我们为牧场 2 选择草种 1，那么我们就会给牛 4 两个牧场分配相同的草种，这违反了规则。然而，牛 1 和牛 5 并不会排除其他草种，因为我们还没有为它们的牧场选择草种。因此，我们选择草种 2，最小的可用草种。现在的情况如下：

| **牧场** | **草种** |
| --- | --- |
| 1 | 1 |
| 2 | 2 |

我们应该为牧场 3 选择什么草种呢？关心牧场 3 的唯一牛是牛 2。牛 2 的牧场是牧场 3 和 5。然而，这头牛不会排除任何草种，因为我们还没有为牧场 5 分配草种！为了选择最小的数字，我们将为牧场 3 选择草种 1。现在我们的进展如下：

| **牧场** | **草种** |
| --- | --- |
| 1 | 1 |
| 2 | 2 |
| 3 | 1 |

我能看到我们从自顶向下设计中逐步清晰化的三项任务。首先，我们需要获取关心当前牧场的牛。其次，我们需要确定这些牛排除的草种。第三，我们需要选择未被排除的最小编号草种。每一项任务都是函数的理想候选。

继续前进。我们有三头牛关心牧场 4：牛 0、1 和 3。牛 0 不会排除任何草种，因为我们还没有为它的牧场分配草种。牛 1 排除了草种 2，因为我们将草种 2 分配给了牧场 2（它的另一个牧场）。牛 3 排除了草种 1，因为我们将草种 1 分配给了牧场 1（它的另一个牧场）。因此，最小的可用草种是 3，所以我们为牧场 4 选择了这个草种：

| **牧场** | **草种** |
| --- | --- |
| 1 | 1 |
| 2 | 2 |
| 3 | 1 |
| 4 | 3 |

接下来是牧场 5。关心牧场 5 的牛是牛 0、2 和 5。牛 0 排除了草种 3；牛 2 排除了草种 1；牛 5 排除了草种 2。所以草种 1、2 和 3 都被排除了。我们唯一的选择是草种 4。

那真是险些出事！我们差点就用完草种了。幸好，没有其他牛关心牧场 5 并排除了草种 4。

等等。其实这并不是运气，因为问题描述中有这么一句：“保证每个牧场最多只会是三头牛的最爱。”这意味着每个牧场最多只能排除三种草种。我们永远不会陷入困境！我们甚至不需要担心过去的选择对下一次决策的影响。无论我们之前做了什么，至少总会有一种可用的草种。

让我们把牧场 5 添加到表格中：

| **牧场** | **草种** |
| --- | --- |
| 1 | 1 |
| 2 | 2 |
| 3 | 1 |
| 4 | 3 |
| 5 | 4 |

有三个草场要处理。但没有牛关心它们，所以我们可以在每种情况下使用草种类型 1。这样我们得到的是：

| **草场** | **草种类型** |
| --- | --- |
| 1 | 1 |
| 2 | 2 |
| 3 | 1 |
| 4 | 3 |
| 5 | 4 |
| 6 | 1 |
| 7 | 1 |
| 8 | 1 |

我们可以从上到下读取草种类型，得到这个示例的正确输出。输出如下：

```py
12134111
```

### 自上而下设计

通过充分理解我们需要完成的任务，我们将转向问题的自上而下设计。

#### 顶层设计

我们在上一节通过测试用例发现了三个任务。在我们的程序能够解决任何这些任务之前，我们需要先读取输入，因此这是第四个任务。我们还需要写输出。这将需要一些思考和几行代码，所以我们将其定为第五个任务。

这是我们的五个主要任务：

1.  读取输入。

1.  确定关心当前草场的牛。

1.  去除当前草场的草种类型。

1.  选择当前草场的最小编号草种类型。

1.  写输出。

就像我们在解决《行动人物》时在 第六章 所做的那样，我们将从一个包含 `TODO` 注释的框架开始，并在解决每个问题时去掉对应的 `TODO`。

我们开始时主要是注释。由于我们在开始时需要打开文件，在结束时需要关闭文件，我还添加了相关代码。

这里是我们开始的地方：

```py
# Main Program

input_file = open('revegetate.in', 'r')

output_file = open('revegetate.out', 'w')

# TODO: Read input

# TODO: Identify cows that care about pasture

# TODO: Eliminate grass types for pasture

# TODO: Choose smallest-numbered grass type for pasture

# TODO: Write output

input_file.close()

output_file.close()
```

#### 任务 1：读取输入

读取输入的第一行，其中包含整数 *n* 和 *m*，是我们已经知道如何做的事情。它足够直接，我认为我们不需要为此编写一个函数，所以我们直接处理它。接下来，我们需要读取 *m* 头牛的草场信息，在这里一个函数似乎是必要的。让我们去掉 `TODO` 注释中的内容，处理第一行输入，并调用 `read_cows` 函数，我们稍后会编写这个函数：

```py
   # Main Program

   input_file = open('revegetate.in', 'r')

   output_file = open('revegetate.out', 'w')

   # Read input

   lst = input_file.readline().split()

   num_pastures = int(lst[0])

   num_cows = int(lst[1])

❶ favorites = read_cows(input_file, num_cows)

   # TODO: Identify cows that care about pasture

   # TODO: Eliminate grass types for pasture

   # TODO: Choose smallest-numbered grass type for pasture

   # TODO: Write output

   input_file.close()

   output_file.close()
```

我们正在调用的 `read_cows` 函数 ❶ 将接受一个已经打开的文件并读取每头牛的两个最爱草场。它将返回一个列表的列表，每个内部列表包含一头牛的两个草场编号。以下是代码：

```py
def read_cows(input_file, num_cows):

    """

    input_file is a file open for reading; cow information is next to read.

    num_cows is the number of cows in the file.

    Read the cows' favorite pastures from input_file.

    Return a list of each cow's two favorite pastures;

    each value in the list is a list of two values giving the

    favorite pastures for one cow.

    """

    favorites = []

    for i in range(num_cows):

     ❶ lst = input_file.readline().split()

        lst[0] = int(lst[0])

        lst[1] = int(lst[1])

     ❷ favorites.append(lst)

    return favorites
```

这个函数将牛的最爱草场收集到 `favorites` 列表中。它使用一个范围 `for` 循环来循环 `num_cows` 次，每次处理一头牛。我们需要这个循环，因为要读取的行数取决于文件中牛的数量。

在每次循环中，我们读取下一行并将其拆分成两个部分 ❶。然后，我们使用 `int` 将这些部分从字符串转换为整数。当我们将这个列表添加到 `favorites` ❷ 时，我们实际上是将一个包含两个整数的列表添加进去了。

我们做的最后一件事是返回最爱的草场列表。

在继续之前，让我们确保我们知道如何调用这个函数。我们将单独练习调用它，而不依赖于我们正在构建的更大程序。测试这样的函数非常有用，因为我们可以在过程中修复任何发现的错误。

使用你的文本编辑器创建一个名为*revegetate.in*的文件，内容如下（与我们之前学习的测试用例相同）：

```py
8 6

5 4

2 4

3 5

4 1

2 1

5 2
```

现在，在 Python shell 中输入我们`read_cows`函数的代码。

这是我们调用`read_cows`的方法：

```py
   >>> input_file = open('revegetate.in', 'r')

❶ >>> input_file.readline()

   '8 6\n'

❷ >>> read_cows(input_file, 6)

   [[5, 4], [2, 4], [3, 5], [4, 1], [2, 1], [5, 2]]
```

`read_cows`函数只读取奶牛的信息。由于我们在程序外单独测试这个函数，因此我们需要在调用它之前自己读取文件的第一行❶。当我们调用`read_cows`时，它会返回一个列表，给出每只奶牛的最爱牧场。还要注意，我们在调用`read_cows`时传入的是打开的文件，而不是文件名❷。

请确保在`# Main Program`注释之前包含我们的`read_cows`函数，以及我们为其他任务编写的函数。然后我们可以继续进行任务 2。

#### 任务 2：识别奶牛

我们解决这个问题的总体策略是逐个考虑每个牧场，决定使用哪种草类型。我们将在一个循环中组织这项工作，每次循环的迭代负责播种一个牧场。对于每个牧场，我们需要识别关心该牧场的奶牛，排除已使用的草类型，并选择编号最小的可用草类型。这三项任务必须对每个牧场执行，因此我们将它们缩进到循环内部。

我们将编写一个名为`cows_with_favorite`的函数，用于告诉我们哪些奶牛关心当前的牧场。

这是我们当前的主程序：

```py
# Main Program

input_file = open('revegetate.in', 'r')

output_file = open('revegetate.out', 'w')

# Read input

lst = input_file.readline().split()

num_pastures = int(lst[0])

num_cows = int(lst[1])

favorites = read_cows(input_file, num_cows)

for i in range(1, num_pastures + 1):

    # Identify cows that care about pasture

 ❶ cows = cows_with_favorite(favorites, i)

    # TODO: Eliminate grass types for pasture

    # TODO: Choose smallest-numbered grass type for pasture

# TODO: Write output

input_file.close()

output_file.close()
```

我们正在调用的`cows_with_favorite`函数❶接收一个奶牛最喜欢的牧场列表和一个牧场编号，并返回关心该牧场的奶牛。以下是代码：

```py
def cows_with_favorite(favorites, pasture):

    """

    favorites is a list of favorite pastures, as returned by read_cows.

    pasture is a pasture number.

    Return list of cows that care about pasture.

    """

    cows = []

    for i in range(len(favorites)):

        if favorites[i][0] == pasture or favorites[i][1] == pasture:

            cows.append(i)

    return cows
```

该函数遍历`favorites`，查找关心牧场编号`pasture`的奶牛。每一只关心该牧场的奶牛都会被添加到`cows`列表中，最后该列表会被返回。

让我们做一个小测试。在 Python shell 中输入我们的`cows_with_favorite`函数。这里是我们将尝试的调用：

```py
>>> cows_with_favorite([[5, 4], [2, 4], [3, 5]], 5)
```

这里有三只奶牛，我们要找出哪些奶牛关心牧场`5`。位于索引`0`和`2`的奶牛关心牧场`5`，这正是该函数告诉我们的内容：

```py
[0, 2]
```

#### 任务 3：排除草类型

现在我们知道了哪些奶牛关心当前的牧场。我们的下一步是弄清楚这些奶牛会排除哪些草类型，无法用于当前牧场。我们将排除与这些奶牛相关的牧场中已经使用的草类型。我们将编写一个名为`types_used`的函数，告诉我们哪些草类型已经被使用（因此不能用于当前牧场）。

这是我们更新后的主程序，包含了对这个函数的调用：

```py
   # Main Program

   input_file = open('revegetate.in', 'r')

   output_file = open('revegetate.out', 'w')

   # Read input

   lst = input_file.readline().split()

   num_pastures = int(lst[0])

   num_cows = int(lst[1])

   favorites = read_cows(input_file, num_cows)

❶ pasture_types = [0]

   for i in range(1, num_pastures + 1):

       # Identify cows that care about pasture

       cows = cows_with_favorite(favorites, i)

       # Eliminate grass types for pasture

    ❷ eliminated = types_used(favorites, cows, pasture_types)

       # TODO: Choose smallest-numbered grass type for pasture

   # TODO: Write output

   input_file.close()

   output_file.close()
```

除了调用`types_used`函数❷，我还添加了一个名为`pasture_types`的变量❶。该变量所引用的列表将跟踪每个牧场的草类型。

请记住，牧场的编号是从 1 开始的，而 Python 的列表是从 0 开始索引的。我不喜欢这种差异；如果我们只是简单地开始将草种添加到`pasture_types`中，那么牧场 1 的草种会在索引 0 的位置，牧场 2 的草种会在索引 1 的位置，以此类推，始终相差一个位置。这就是为什么我在列表开头添加了一个虚假的`0` ❶；当我们后续为牧场 1 添加草种时，它会被放置在索引 1 的位置，以便与之匹配。

假设我们已经弄清楚了前四个牧场的草种。这时`pasture_types`可能看起来是这样的：

```py
[0, 1, 2, 1, 3]
```

如果我们想要牧场 1 的草种，我们查看索引 1；如果我们想要牧场 2 的草种，我们查看索引 2；以此类推。如果我们想要牧场 5 的草种？嗯，不行，因为我们还没有弄清楚。如果`pasture_types`的长度是`5`，这意味着我们只弄清楚了前四个牧场的草种。一般来说，我们弄清楚的草种数量比列表的长度少一个。

现在我们准备使用`types_used`函数了。它接受三个参数：每头牛最喜欢的牧场列表、关心当前牧场的牛、以及目前已选定的牧场草种。它返回当前牧场已经使用并因此被排除的草种列表。接下来是：

```py
def types_used(favorites, cows, pasture_types):

    """

    favorites is a list of favorite pastures, as returned by read_cows.

    cows is a list of cows.

    pasture_types is a list of grass types.

    Return a list of the grass types already used by cows.

    """

    used = []

    for cow in cows:

        pasture_a = favorites[cow][0]

        pasture_b = favorites[cow][1]

 ❶ if pasture_a < len(pasture_types):

            used.append(pasture_types[pasture_a])

     ❷ if pasture_b < len(pasture_types):

            used.append(pasture_types[pasture_b])

    return used
```

每头牛有两个最喜欢的牧场，我称之为`pasture_a`和`pasture_b`。对于这些牧场，我们检查在❶和❷的位置，是否已经为其选择了草种。如果该牧场已经是`pasture_types`中的一个索引，那么草种就已经被选择。这些草种都被添加到`used`列表中，函数将在遍历所有相关牛后返回该列表。

如果多头牛使用同一片牧场——我们的代码会怎么处理呢？让我们提出一个简单的测试用例来回答这个问题。

在 Python shell 中输入我们的`types_used`函数。以下是对该函数的调用；我们来预测它的返回值：

```py
>>> types_used([[5, 4], [2, 4], [3, 5]], [0, 1], [0, 1, 2, 1, 3])
```

我们要小心，以免迷失。第一个参数给出了三头牛的最喜欢的牧场。第二个参数给出了关心某个特定牧场的牛，这些是牛`0`和牛`1`。第三个参数给出了我们目前为止决定的草种。

那么，牛`0`和牛`1`已经使用并因此被排除的草种有哪些呢？牛`0`关心牧场`4`，而牧场`4`使用草种`3`，所以草种`3`被排除。牛`1`关心牧场`2`，而牧场`2`使用草种`2`，因此草种`2`被排除。牛`1`还关心牧场`4`——但我们已经从牛`0`那里知道，牧场`4`的草种`3`已经被排除。

我们函数的返回值是这样的：

```py
[3, 2, 3]
```

有两个`3`，一个来自牛`0`，另一个来自牛`1`。

或许只有一个`3`看起来更整洁，但现在这种带重复值的方式其实也没问题。如果某个草类型出现在列表中，那它就被排除，无论它出现了一次、两次，还是三次。

#### 任务 4：选择最小编号的草类型

在得到已经被排除的草类型后，我们可以进行下一个任务：选择当前牧场的最小编号可用草类型。为了解决这个问题，我们将调用一个新的函数`smallest_available`。它将返回我们应该为当前牧场使用的草类型。

以下是主程序，更新后调用了`smallest_available`函数：

```py
# Main Program

input_file = open('revegetate.in', 'r')

output_file = open('revegetate.out', 'w')

# Read input

lst = input_file.readline().split()

num_pastures = int(lst[0])

num_cows = int(lst[1])

favorites = read_cows(input_file, num_cows)

pasture_types = [0]

for i in range(1, num_pastures + 1):

    # Identify cows that care about pasture

    cows = cows_with_favorite(favorites, i)

    # Eliminate grass types for pasture

    eliminated = types_used(favorites, cows, pasture_types)

    # Choose smallest-numbered grass type for pasture

 ❶ pasture_type = smallest_available(eliminated)

 ❷ pasture_types.append(pasture_type)

# TODO: Write output

input_file.close()

output_file.close()
```

一旦我们获得了当前牧场的最小编号草类型❶，我们将其添加到已选择的草类型列表中❷。

这是`smallest_available`函数本身：

```py
def smallest_available(used):

    """

    used is a list of used grass types.

    Return the smallest-numbered grass type that is not in used.

    """

    grass_type = 1

    while grass_type in used:

        grass_type = grass_type + 1

    return grass_type
```

该函数从草类型`1`开始。然后它循环，直到找到一个未被使用的草类型，每次循环时将草类型加 1。一旦找到一个空闲的草类型，函数就返回它。请记住，最多只有四种草类型，其中已经使用了最多三种，因此该函数一定能成功。

#### 任务 5：写入输出

我们已经得到了答案，就在`pasture_types`中！现在我们只需输出它。以下是最终的主程序：

```py
   # Main Program

   input_file = open('revegetate.in', 'r')

   output_file = open('revegetate.out', 'w')

   # Read input

   lst = input_file.readline().split()

   num_pastures = int(lst[0])

   num_cows = int(lst[1])

   favorites = read_cows(input_file, num_cows)

   pasture_types = [0]

   for i in range(1, num_pastures + 1):

       # Identify cows that care about pasture

       cows = cows_with_favorite(favorites, i)

       # Eliminate grass types for pasture

       eliminated = types_used(favorites, cows, pasture_types)

       # Choose smallest-numbered grass type for pasture

       pasture_type = smallest_available(eliminated)

       pasture_types.append(pasture_type)

   # Write output

❶ pasture_types.pop(0)

❷ write_pastures(output_file, pasture_types)

   input_file.close()

   output_file.close()
```

在写入输出之前，我们首先去除`pasture_types`开头的无效`0`❶。我们不想输出那个`0`，因为它并不是真正的草类型。然后，我们调用`write_pastures`函数，实际上写入输出❷。

现在我们所需要的就是`write_pastures`函数。它接受一个打开用于写入的文件和一个草类型列表，并将草类型输出到文件中。以下是代码：

```py
   def write_pastures(output_file, pasture_types):

       """

       output_file is a file open for writing.

       pasture_types is a list of integer grass types.

       Output pasture_types to output_file.

       """

       pasture_types_str = []

    ❶ for pasture_type in pasture_types:

           pasture_types_str.append(str(pasture_type))

    ❷ output = ''.join(pasture_types_str)

    ❸ output_file.write(output + '\n')
```

现在，`pasture_types`是一个整数列表。正如我们稍后会看到的，在这里使用字符串列表会更加方便，因此我创建了一个新的列表，将每个整数转换为字符串❶。我没有修改`pasture_types`列表本身，因为那样会影响到调用此函数的代码。调用者调用此函数时，只期望输出结果写入`output_file`，并不希望`pasture_types`列表被修改。函数没有理由修改它的列表参数。

为了输出结果，我们需要使用`write`函数传入一个字符串，而不是一个列表。而且我们需要将列表中的字符串输出，且它们之间不能有空格。`join`方法在这里非常有效。正如我们在第五章的“将列表连接成字符串”部分所学，调用`join`的字符串作为分隔符，插入到列表中的值之间。由于我们不想在值之间加任何分隔符，因此我们使用空字符串作为分隔符❷。`join`方法只能作用于字符串列表，而不能作用于整数列表，这就是为什么我在这个函数开始时将整数列表转换为字符串列表❶。

现在，输出已经是一个单一的字符串，我们可以将其写入文件❸。

##### 将所有内容整合起来

完整的程序在清单 7-2 中。

```py
def read_cows(input_file, num_cows):

    """

    input_file is a file open for reading; cow information is next to read.

    num_cows is the number of cows in the file.

    Read the cows' favorite pastures from input_file.

    Return a list of each cow's two favorite pastures;

    each value in the list is a list of two values giving the

    favorite pastures for one cow.

    """

    favorites = []

    for i in range(num_cows):

        lst = input_file.readline().split()

        lst[0] = int(lst[0])

        lst[1] = int(lst[1])

        favorites.append(lst)

    return favorites

 def cows_with_favorite(favorites, pasture):

    """

    favorites is a list of favorite pastures, as returned by read_cows.

    pasture is a pasture number.

    Return list of cows that care about pasture.

    """

    cows = []

    for i in range(len(favorites)):

        if favorites[i][0] == pasture or favorites[i][1] == pasture:

            cows.append(i)

    return cows

def types_used(favorites, cows, pasture_types):

    """

    favorites is a list of favorite pastures, as returned by read_cows.

    cows is a list of cows.

    pasture_types is a list of grass types.

    Return a list of the grass types already used by cows.

    """

    used = []

    for cow in cows:

        pasture_a = favorites[cow][0]

        pasture_b = favorites[cow][1]

        if pasture_a < len(pasture_types):

            used.append(pasture_types[pasture_a])

        if pasture_b < len(pasture_types):

            used.append(pasture_types[pasture_b])

    return used

def smallest_available(used):

    """

    used is a list of used grass types.

    Return the smallest-numbered grass type that is not in used.

    """

    grass_type = 1

    while grass_type in used:

        grass_type = grass_type + 1

    return grass_type

 def write_pastures(output_file, pasture_types):

    """

    output_file is a file open for writing.

    pasture_types is a list of integer grass types.

    Output pasture_types to output_file.

    """

    pasture_types_str = []

    for pasture_type in pasture_types:

        pasture_types_str.append(str(pasture_type))

    output = ''.join(pasture_types_str)

    output_file.write(output + '\n')

# Main Program

input_file = open('revegetate.in', 'r')

output_file = open('revegetate.out', 'w')

# Read input

lst = input_file.readline().split()

num_pastures = int(lst[0])

num_cows = int(lst[1])

favorites = read_cows(input_file, num_cows)

pasture_types = [0]

for i in range(1, num_pastures + 1):

    # Identify cows that care about pasture

    cows = cows_with_favorite(favorites, i)

    # Eliminate grass types for pasture

    eliminated = types_used(favorites, cows, pasture_types)

    # Choose smallest-numbered grass type for pasture

    pasture_type = smallest_available(eliminated)

    pasture_types.append(pasture_type)

# Write output

pasture_types.pop(0)

write_pastures(output_file, pasture_types)

input_file.close()

output_file.close()
```

*清单 7-2：解决农场播种问题*

我们做到了！一个让人望而生畏的问题，通过自顶向下的设计变得更容易处理。可以放心地将我们的工作提交给 USACO 评审。

初次阅读一个问题时，可能会感到不知所措。但记住，你不需要一次性完成所有的步骤。把问题分解开，解决你能解决的每个小任务，你就能为解决整体问题迈出重要的一步。你在 Python 知识和程序设计与问题解决能力方面已经取得了巨大的进展。解决这些问题触手可及！

**概念检查**

让我们考虑一个新的农场播种版本，在这个版本中，牧场不再有限制，允许多只奶牛关心一个牧场。一个牧场可能是四只奶牛、五只奶牛甚至更多的最爱。我们仍然不能给同一只奶牛分配两块草类型相同的牧场。

假设我们正在解决这个新版本的问题，并且有一个测试用例，其中一个牧场是超过三只奶牛最喜欢的。在这个测试用例中，以下哪项是正确的？

A。*可以保证*仅使用四种草类型是*无法*解决这个问题的。

B。可能有一种解决方法。如果有的话，*可能*我们的原始解决方案（清单 7-2）*会*做到这一点。

C。可能有一种解决方法。如果有的话，*可以保证*我们的原始解决方案（清单 7-2）*会*做到这一点。

D。可能有一种解决方法。如果有的话，*可以保证*我们的原始解决方案（清单 7-2）*无法*做到这一点。

答案：B。我们可以找到一个能被我们的程序正确解决的测试用例，也可以找到一个能被解决但无法通过我们程序解决的测试用例。前者排除了 A 和 D 为正确答案；后者排除了 C 为正确答案。

这是一个能被我们程序正确解决的测试用例：

```py
2 4

1 2

1 2

1 2

1 2
```

每个牧场都有四只奶牛最喜欢。尽管如此，我们可以仅使用两种草的类型来解决这个测试用例。试试我们的程序，你应该会看到它能够正确解决这个测试用例。

现在，这里有一个能解决的问题，但不是通过我们的程序：

```py
6 10

2 3

2 4

3 4

2 5

3 5

4 5

1 6

3 6

4 6

5 6
```

我们程序的错误在于将草类型 1 分配给了牧场 1。这样，它被迫将草类型 5——这是不允许的！——分配给牧场 6。我们的程序失败了，但不要因此得出没有办法解决这个测试用例的结论。特别是，可以将草类型 2 分配给牧场 1，你应该能够找到只使用四种草类型来解决这个测试用例的方法。通过更复杂的程序来解决这类问题是可行的，如果你感兴趣，鼓励你自己思考一下。

### 总结

在本章中，我们学习了如何打开、读取、写入和关闭文件。文件在你需要存储信息并稍后作为输入使用时非常有用。它们也在向用户传递信息时非常有用。我们还学到了如何像处理标准输入和标准输出一样处理文件。

在下一章中，我们将学习如何在 Python 集合或字典中存储一组值。存储一组值——听起来像是列表的功能。不过我们会发现，集合和字典可以帮助我们更轻松地解决某些问题。

### 章节练习

下面是一些练习，供你尝试。它们都是来自 USACO 的评审，要求你读取和写入文件。它们还会要求你复习前面章节中的一些知识点。

1.  USACO 2018 年 12 月铜奖竞赛问题 混合牛奶

1.  USACO 2017 年 2 月铜奖竞赛问题 为什么牛要过马路

1.  USACO 2017 年美国公开赛铜奖竞赛问题 失踪的牛

1.  USACO 2019 年 12 月铜奖竞赛问题 牛体操

1.  USACO 2017 年美国公开赛铜奖竞赛问题 牛基因组学

1.  USACO 2018 年美国公开赛铜奖竞赛问题 团队井字棋

1.  USACO 2019 年 2 月铜奖竞赛问题 昏昏欲睡的牛群

### 注释

文章格式化最初来自 USACO 2020 年 1 月的铜奖竞赛。农场播种最初来自 USACO 2019 年 2 月的铜奖竞赛。

除了文本文件，还有许多其他类型的文件。你可能想处理 HTML 文件、Excel 电子表格、PDF 文件、Word 文档或图像文件。Python 可以帮忙！欲了解更多信息，请参阅*《用 Python 自动化无聊的事》*，第二版，作者：Al Sweigart（No Starch Press 出版社，2019 年）。

这句“或许更好的诗歌”出自 J. C. R. Licklider 的名言，引用自*《计算机与未来的世界》*，由马丁·格林伯格（Martin Greenberger）编辑（MIT 出版社，1962 年）：

但有些人用我们说的语言写诗。或许将来用数字计算机的语言写出的诗，比用英语写的更好。
