## 7

读取和写入文件

![image](Images/common.jpg)

到目前为止，我们通过 input 函数读取所有输入，并通过 print 函数写入所有输出。这些函数分别从标准输入（默认为键盘）读取并写入到标准输出（默认为屏幕）。虽然我们可以使用输入和输出重定向来改变这些默认设置，但有时程序需要更多的文件控制。例如，你的文字处理器允许你打开任何文档文件并以你喜欢的名称保存文件，而无需操作标准输入和标准输出。

在本章中，我们将学习如何编写处理文本文件的程序。我们将通过文件解决两个问题：正确格式化文章和为农场播种以喂养奶牛。

### 问题 #16：文章格式化

这个问题与我们之前解决的所有问题之间有一个重要的区别：这个问题要求我们从特定的文件中读取并写入文件！在阅读问题描述时要特别注意这一点。

这是 USACO 2020 年 1 月铜奖竞赛问题《文字处理器》。这是本书中来自 USACO（美国计算机奥林匹克竞赛）评审的第一个问题。要找到该问题，请访问 *[http://usaco.org/](http://usaco.org/)*，点击 **Contests**，点击 **2020 January Contest Results**，然后点击 **View problem** 查看《文字处理器》问题。

#### 挑战

奶牛贝西正在写一篇文章。文章中的每个单词仅包含小写或大写字母。她的老师指定了每行最多可以出现的字符数（不包括空格）。为了满足这一要求，贝西使用以下规则写下文章中的单词：

+   如果下一个单词可以放在当前行中，则将其添加到当前行。在每两个单词之间加上一个空格。

+   否则，将该单词放在新的一行；这一行变为新的当前行。

输出格式正确的文章，每行显示正确的单词。

#### 输入

从名为 *word.in* 的文件中读取输入。

输入由两行组成。

+   第一行包含两个由空格分隔的整数。第一个整数是 *n*，即文章中的单词数；它的范围是 1 到 100。第二个整数是 *k*，即每行最多可包含的字符数（不包括空格）；它的范围是 1 到 80。

+   第二行包含 *n* 个单词，每两个单词之间用空格隔开。每个单词最多包含 *k* 个字符。

#### 输出

将输出写入名为 *word.out* 的文件。

输出格式正确的文章。

### 文件操作

《文章格式化》问题要求我们从 *word.in* 文件中读取，并将输出写入 *word.out* 文件。不过，在我们做这些事情之前，我们需要学习如何在程序中打开文件。

#### 打开文件

使用文本编辑器创建一个名为 *word.in* 的新文件。将该文件放在你用来存放 *.py* Python 程序的相同目录下。

这是我们第一次创建一个不以 *.py* 结尾的文件。相反，它以 *.in* 结尾。请确保将文件命名为 *word.in*，而不是 *word.py*。*in* 是输入的缩写；你会经常看到它用于包含程序输入的文件。

在这个文件中，我们将为论文格式化问题输入有效内容。请将以下内容输入文件：

12 13

也许更好的诗歌将在数字计算机的语言中写成

保存文件。

为了在 Python 中打开文件，我们使用 open 函数。我们传递两个参数：第一个是文件名，第二个是打开文件的模式。模式决定了我们如何与文件进行交互。

这是我们如何打开 *word.in*：

>>> open('word.in', 'r')

❶ <_io.TextIOWrapper name='word.in' mode='r' encoding='cp1252'>

在这个函数调用中，我们提供了 'r' 模式。r 代表“读取”，并打开文件以便我们从中读取。模式是一个可选参数，其默认值是 'r'，所以我们可以选择不写。但为了保持一致性，我会在本书中明确包含 'r'。

当我们使用 open 时，Python 会提供一些关于文件打开方式的信息 ❶。例如，它会确认文件名和模式。关于编码的部分表示文件如何从磁盘上的状态解码为我们可以读取的形式。文件可以使用多种编码方式进行编码，但在本书中我们不需要担心编码问题。

如果我们尝试打开一个不存在的文件进行读取，就会得到一个错误：

>>> open('blah.in', 'r')

Traceback (most recent call last):

文件 "<stdin>"，第 1 行，位于 <module>

FileNotFoundError: [Errno 2] 没有那个文件或目录: 'blah.in'

如果你在打开 *word.in* 时遇到此错误，请仔细检查文件名是否正确，且文件是否位于你启动 Python 时的目录中。

除了 'r' 模式用于读取，还有 'w' 模式用于写入。如果我们使用 'w'，则表示我们正在打开一个文件，以便向其中写入文本。

小心使用 'w' 模式。如果你用 'w' 打开一个已存在的文件，那个文件的内容会被删除。我刚刚就不小心用 'w' 打开了我的 *word.in* 文件。没关系，因为很容易重新创建它。但是如果我们不小心覆盖了一个重要文件，那可就糟了。

如果你使用 'w' 打开一个不存在的文件，它会创建一个空文件。

让我们使用 'w' 模式创建一个空文件，名为 *blah.in*：

>>> open('blah.in', 'w')

<_io.TextIOWrapper name='blah.in' mode='w' encoding='cp1252'>

现在 *blah.in* 文件已经存在，我们可以打开它进行读取而不会出错：

>>> open('blah.in', 'r')

<_io.TextIOWrapper name='blah.in' mode='r' encoding='cp1252'>

那个我们一直看到的 _io.TextIOWrapper 是什么？那是 open 返回值的类型：

>>> type(open('word.in', 'r'))

<class '_io.TextIOWrapper'>

将这个类型视为文件类型。它的值表示已打开的文件，你会很快看到它有一些我们可以调用的方法。

和任何函数一样，如果我们没有将 open 返回的结果赋值给变量，那么它的返回值会丢失。到目前为止，我们调用 open 的方式并没有给我们提供任何方式来引用我们打开的文件！

这是我们可以让变量引用打开文件的方式：

>>> input_file = open('word.in', 'r')

>>> input_file

<_io.TextIOWrapper name='word.in' mode='r' encoding='cp1252'>

我们可以使用 input_file 从 'word.in' 中读取数据。

在解决 Essay Formatting 时，我们还需要一种方式将内容写入文件 'word.out'。以下是一个有助于我们完成这项任务的变量：

>>> output_file = open('word.out', 'w')

>>> output_file

<_io.TextIOWrapper name='word.out' mode='w' encoding='cp1252'>

#### 从文件中读取

要从已打开的文件中读取一行，我们使用文件的 readline 方法。该方法返回一个字符串，包含文件中下一行的内容。从这个角度来看，它类似于 input 函数。然而，与 input 不同，readline 是从文件中读取数据，而不是从标准输入读取。

让我们打开 *word.in* 并读取它的两行：

>>> input_file = open('word.in', 'r')

>>> input_file.readline()

'12 13\n'

>>> input_file.readline()

'或许更好的诗歌将会用数字计算机的语言写成\n'

这里意外的是每个字符串末尾的 \n。我们在使用 input 读取一行时显然没有看到这个。字符串中的 \ 符号是 *转义字符*。它跳脱了字符的标准解释并改变了其含义。我们不把 \n 当作两个独立的字符 \ 和 n 来处理。相反，\n 就是一个字符：换行符。文件中的所有行，除了最后一行，都会以换行符结束。如果没有换行符，所有内容就会被挤到一行！readline 方法字面上返回的是整行内容，包括它的结束换行符。

这是我们可以如何在字符串中嵌入换行符：

>>> 'one\ntwo\nthree'

'one\ntwo\nthree'

>>> print('one\ntwo\nthree')

one

two

three

Python shell 不处理转义字符的效果，但 print 会。

\n 序列在字符串中非常有用，因为它帮助我们添加多行内容。但我们很少希望从文件中读取的行包含这些换行符。为了去除它们，我们可以使用字符串的 rstrip 方法。这个方法类似于 strip，不过它只会从字符串的右边（而不是左边）去除空白字符。对于它而言，换行符就像空格一样是空白字符：

>>> 'hello\nthere\n\n'

'hello\nthere\n\n'

>>> 'hello\nthere\n\n'.rstrip()

'hello\nthere'

让我们再尝试从文件中读取一次，这次去掉换行符：

>>> input_file = open('word.in', 'r')

>>> input_file.readline().rstrip()

'12 13'

>>> input_file.readline().rstrip()

'或许更好的诗歌将会用数字计算机的语言写成'

此时，我们已经读取了两行，因此文件中没有剩余内容可读。readline 方法通过返回一个空字符串来标识这一点。

>>> input_file.readline().rstrip()

''

空字符串意味着我们已经到达了文件的末尾。如果我们想再次读取这些行，必须重新打开文件，从头开始。

让我们这么做，这次使用变量保存每一行：

>>> input_file = open('word.in', 'r')

>>> first = input_file.readline().rstrip()

>>> second = input_file.readline().rstrip()

>>> first

'12 13'

>>> second

'也许更好的诗歌将用数字计算机的语言写成'

如果我们需要读取文件中的所有行，不管有多少行，我们可以使用 for 循环。Python 中的文件就像是行的序列，我们可以像遍历字符串和列表一样遍历它们：

>>> input_file = open('word.in', 'r')

>>> for line in input_file:

...     print(line.rstrip())

...

12 13

也许更好的诗歌将用数字计算机的语言写成。

与字符串或循环不同，我们不能再次遍历文件，因为第一次遍历已经将我们带到了文件的末尾。如果我们再试一次，什么也得不到：

>>> for line in input_file:

...     print(line.rstrip())

...

**概念检查**

我们希望使用 while 循环输出打开的文件 input_file 中的每一行。（该文件可以是任何文件；我并不假设它与《论文格式化》有关。）以下哪段代码正确地实现了这一点？

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

D. 上述所有选项

E. 以上皆非

在查看答案之前，我建议你创建一个包含四五行内容的文件，并在该文件上尝试每段代码。你还可以考虑在每行输出的开头加一个像 * 这样的字符，以便你能看到任何原本是空白的行。

答案：E. 每段代码都有一个微妙的错误。

代码 A 只输出文件的每隔一行。例如，while 循环的布尔表达式使得第一行被读取……但丢失了，因为它没有被赋值给一个变量。循环的第一次迭代因此输出文件的第二行。

代码 B 非常接近正确的做法。它输出了文件的所有行，但也在末尾输出了一个多余的空白行。

代码 C 未能打印文件的第一行。原因是第一行在循环之前被读取，但循环接着读取第二行，而没有打印第一行。它还会像代码 B 一样在末尾输出一个多余的空白行。

下面是正确的代码来读取并打印每一行：

line = input_file.readline()

while line != '':

print(line.rstrip())

line = input_file.readline()

#### 写入文件

要写一行到一个打开的文件中，我们使用文件的 write 方法。我们传递一个字符串，这个字符串会被添加到文件的末尾。

为了解决作文格式化问题，我们将写入*word.out*。但我们还没有准备好解决这个问题，所以我们暂时写入*blah.out*。下面是我们如何将一行写入该文件：

>>> output_file = open('blah.out', 'w')

>>> output_file.write('hello')

5

那个5是怎么回事？`write`方法返回写入的字符数。这是确认我们确实写入了预期的文本量的好方法。

如果你在文本编辑器中打开*blah.out*，你应该会看到里面有文本hello。

让我们尝试向文件写入三行。开始吧：

>>> output_file = open('blah.out', 'w')

>>> output_file.write('sq')

2

>>> output_file.write('ui')

2

>>> output_file.write('sh')

2

根据我到目前为止告诉你的内容，你可能会期望*blah.out*看起来像这样：

sq

ui

sh

但如果你在文本编辑器中打开*blah.out*，你应该会看到以下内容：

squish

字符会在同一行上，因为`write`不会为我们添加换行符！如果我们希望分开行，我们需要显式地这样做：

>>> output_file = open('blah.out', 'w')

>>> output_file.write('sq\n')

3

>>> output_file.write('ui\n')

3

>>> output_file.write('sh\n')

3

注意在每个例子中，`write`写入了三个字符，而不是两个。换行符也算作一个字符。现在如果你在文本编辑器中打开*blah.out*，你应该会看到文本分布在三行上：

sq

ui

sh

与`print`不同，`write`只有在你传入一个字符串时才有效。要将数字写入文件，首先将其转换为字符串：

>>> num = 7788

>>> output_file = open('blah.out', 'w')

>>> output_file.write(str(num) + '\n')

5

##### 关闭文件

养成在完成操作后关闭文件的好习惯。这告诉你的代码的读者，文件已经不再使用。

关闭文件还有助于操作系统管理计算机的资源。当你使用`write`方法时，你写入的内容可能不会立即写入文件。相反，Python或你的操作系统可能会等到有一堆写入请求，然后一次性将它们写入。关闭一个你曾写入的文件能确保你写入的内容安全地存储在文件中。

要关闭文件，调用其`close`方法。下面是打开文件、读取一行并关闭它的示例：

>>> input_file = open('word.in', 'r')

>>> input_file.readline()

'12 13\n'

>>> input_file.close()

一旦关闭文件，你就不能再从文件中读取或写入：

>>> input_file.readline()

Traceback (most recent call last):

File "<stdin>", line 1, in <module>

ValueError: 在已关闭的文件上进行I/O操作。

### 解决问题

回到作文格式化问题。现在我们知道如何从*word.in*读取并写入*word.out*了。这就处理了输入和输出的要求。接下来是解决问题本身。

让我们通过探索一个测试用例来确保我们知道如何解决这个问题。然后我们再看代码。

#### 探索一个测试用例

这是我一直在使用的*word.in*文件：

12 13

也许更好的诗歌将用数字计算机的语言来写作

总共有12个单词，且每行的最大字符数（不包括空格）是13。我们应该尽可能将单词添加到当前行，只要它们适合；一旦一个单词无法放入当前行，我们就从下一个新行开始，并将该单词放入新行。

这个单词perhaps包含七个字符，所以它可以放在第一行。单词better包含六个字符。它也能放在第一行；已经有perhaps在这里，我们一共是13个字符（不包括两个单词之间的空格）。

单词poetry不能放在第一行，所以我们从新的一行开始，poetry作为第一单词。这个单词will可以紧接着poetry放在第二行。同样，be可以紧接will放置。到目前为止我们已经有了12个非空格字符。现在我们有了单词written，由于第二行只剩下一个字符的位置，我们被迫从新的一行开始，written作为第一单词。

按照这个过程一直走下去，我们需要写到*word.out*的完整文章是这样的：

也许更好

诗歌将是

写在

语言

数字

计算机

#### 代码

我们的解决方案在[列表7-1](ch07.xhtml#ch07ex01)中。

❶ input_file = open('word.in', 'r')

❷ output_file = open('word.out', 'w')

❸ lst = input_file.readline().split()

n = int(lst[0])  # n 不需要

k = int(lst[1])

words = input_file.readline().split()

❹ line = ''

chars_on_line = 0

对于每个单词 in words:

❺ 如果 chars_on_line + len(word) <= k:

line = line + word + ' '

chars_on_line = chars_on_line + len(word)

否则：

❻ output_file.write(line[:-1] + '\n')

line = word + ' '

chars_on_line = len(word)

❼ output_file.write(line[:-1] + '\n')

input_file.close()

output_file.close()

*列表7-1：解决文章格式问题*

首先，我们打开输入文件 ❶ 和输出文件 ❷。注意模式：我们以'r'模式（读取）打开输入文件，以'w'模式（写入）打开输出文件。我们也可以稍后再打开输出文件，正好在我们需要它的时候，但我选择在这里同时打开两个文件，以简化程序的组织。同样的，我们可以在不再需要文件时立即关闭它们，但在本书中，我选择在程序结束时一起关闭所有文件。对于长时间运行且操作多个文件的程序，你可能只希望在需要时才保持文件打开。

接下来，我们读取输入文件的第一行 ❸。这一行包含两个由空格分隔的整数：n，单词的数量，以及k，每行允许的最大字符数（不包括空格）。像往常一样，使用split来分割这些值。然后我们读取第二行，这一行包含文章的单词。我们再次使用split，这次是将单词的字符串分割成一个单词列表。这就完成了输入的处理。

有两个变量驱动程序的主要部分：line和chars_on_line。line变量表示当前行；我们从让它指向空字符串❹开始。chars_on_line变量表示当前行上的字符数，不包括空格。

你可能会想知道为什么我要维护chars_on_line。我们不可以直接使用len(line)吗？嗯，如果这样做，我们会将空格也计入在内，而空格不计入每行允许的字符数。我们可以通过减去空格的数量来解决这个问题，如果你觉得这样更直观，而不是保持chars_on_line变量在那里的话，我鼓励你自己试试看。

现在是时候遍历所有单词了。对于每个单词，我们需要确定它是放在当前行还是下一行。

如果当前行上的非空字符数量加上当前单词的字符数量最多为k，那么当前单词可以放入当前行❺。在这种情况下，我们将单词和一个空格添加到当前行，并更新当前行上的非空字符数量。

否则，当前单词不能放入当前行。当前行已结束！因此，我们将当前行写入输出文件❻，并更新line和chars_on_line变量，以反映这是现在的唯一单词所在的当前行。

关于write调用❻，有两点需要注意。首先，[ :-1 ]切片是用来防止我们输出行末尾跟随的空格。其次，你可能期待我在这里使用f-string，像这样：

output_file.write(f'{line[:-1]}\n')

然而，写作时，USACO评测器正在运行一个较旧版本的Python，这个版本不支持f-string。

为什么在循环结束后我们要输出line❼？原因是每次for循环迭代结束时，line中都会留下一个或多个尚未输出的单词。考虑我们处理每个单词时发生的情况。如果当前单词可以放入当前行，我们什么都不输出。如果当前单词不能放入当前行，那么我们会输出当前行，但不会输出下行的单词。因此，我们需要在循环后将line写入输出文件❼；否则，文章的最后一行会丢失。

最后，我们关闭两个文件。

将数据写入文件而不是屏幕的一个烦人之处是，当我们运行程序时，看不到输出。为了查看输出，我们必须在文本编辑器中打开输出文件。

这是一个小提示：在开发程序时，使用print调用而不是write调用，这样所有输出都会显示在屏幕上。这应该能让你更容易找到程序中的错误，并避免在代码和输出文件之间来回切换。一旦你对代码满意了，就可以把print调用改回write调用。然后，一定要进行更多的测试，确保一切都正确地写入文件中。

我们准备好提交给USACO评测员了。把我们的代码提交给他们吧！所有测试用例都应该通过。

### 问题#17：农场播种

我们可以使用循环从文件中读取指定数量的行。在这个问题中我们会这么做，而且我们会发现它和使用循环从标准输入中读取输入非常相似。

在[第6章](ch06.xhtml#ch06)中，当我们解决“动作人偶”问题时，我们学习了如何使用函数进行自顶向下的设计。这是一项重要技能，能够通过组合多个函数来解决问题。由于关于文件的内容不多，我选择了一个问题，它同时也是自顶向下设计的一个应用场景。

这是一个具有挑战性的问题。我们首先需要明确我们被要求做什么。之后，我们需要开发一种解决问题的方法，并仔细思考我们的解决方案为何是正确的。

这是USACO 2019年2月青铜竞赛问题“伟大的再生草”。

#### 挑战

Farmer John有*n*个牧场，所有这些牧场他都希望种植草种。这些牧场的编号为1、2、……、*n*。

Farmer John有四种不同类型的草种，编号为1、2、3和4。他将从中选择一种草种类型用于每个牧场。

Farmer John还拥有*m*头牛。每头牛有两个最喜欢的牧场，在这些牧场中它吃草。每头牛只关心它的两个最喜欢的牧场，其他牧场不关心。为了保持健康的饮食，每头牛要求它的两个牧场使用不同类型的草种。例如，对于某头牛来说，如果其中一个牧场的草种类型为1，另一个牧场的草种类型为4，那是可以的。但如果两个牧场的草种类型都是1，那就是不允许的。

一个牧场可能是多头牛的最爱。但保证一个牧场最多只有三头牛最喜欢。

确定每个牧场使用的草种类型。每个牧场需要使用1到4之间的某种草种类型，而且每头牛的两个最喜欢的牧场必须使用不同的草种类型。

#### 输入

从名为*revegetate.in*的文件中读取输入。

输入由以下几行组成：

+   一行包含两个整数，用空格分隔。第一个整数是*n*，表示牧场的数量；它的值介于2和100之间。第二个整数是*m*，表示牛的数量；它的值介于1和150之间。

+   *m*行，每行给出一头牛最喜欢的两个牧场编号。这些牧场编号是介于1和*n*之间的整数，并且用空格分隔。

#### 输出

输出写入名为*revegetate.out*的文件中。

输出一种有效的牧场播种方式。输出是一行*n*个字符，每个字符是'1'、'2'、'3'或'4'。第一个字符表示牧场1的草种类型，第二个表示牧场2的草种类型，以此类推。

我们可以将这*n*个字符解释为一个具有*n*位数字的整数。例如，如果我们有五个草种类型'11123'，我们可以将其解释为整数11123。

当我们有选择输出内容时，这种整数解释就起作用了。如果有多种有效的方式来给牧场播种，我们必须输出以整数方式解释后最小的那个。例如，如果'11123'和'22123'都是有效的，我们会输出字符串'11123'，因为11123小于22123。

### 探索一个测试用例

我们将使用自顶向下的设计方法来解决这个问题。通过分析测试用例，我们可以帮助我们筛选任务。

这是测试用例：

8 6

5 4

2 4

3 5

4 1

2 1

5 2

测试用例的第一行告诉我们有八个牧场，它们的编号从 1 到 8。同时第一行还告诉我们有六头奶牛。问题没有为奶牛指定编号，所以我将从编号 0 开始编号。每头奶牛最喜欢的两个牧场列在[表 7-1](ch07.xhtml#ch07tab01)中，方便参考。

**表 7-1：** 农场播种示例，奶牛

| **奶牛** | **牧场 1** | **牧场 2** |
| --- | --- | --- |
| 0 | 5 | 4 |
| 1 | 2 | 4 |
| 2 | 3 | 5 |
| 3 | 4 | 1 |
| 4 | 2 | 1 |
| 5 | 5 | 2 |

在这个问题中，我们需要做出 *n* 个决策。我们应该为牧场 1 选择哪种草种类型？为牧场 2 选择哪种草种类型？牧场 3 呢？牧场 4 呢？依此类推，直到牧场 *n*。解决这类问题的一种策略是一次做一个决策，确保每次决策都没有错误。如果我们在做出第 *n* 个决策时没有犯错，那么我们的解答一定是正确的。

我们将从牧场 1 到 8 一一检查，看看是否能为每个牧场分配草种类型。我们需要优先选择编号较小的草种类型，这样当作为数字来解释时，最终的草种类型会尽可能小。

我们应该为牧场 1 选择哪种草种类型？关心牧场 1 的奶牛只有奶牛 3 和奶牛 4，所以我们只需要关注它们。如果我们已经为这些奶牛的某些牧场选择了草种类型，我们就需要小心选择牧场 1 的草种类型。我们不想给某头奶牛两个相同草种类型的牧场，因为那样会违反规则！不过我们还没有选择任何草种类型，所以无论选择什么，牧场 1 都不会出错。不过，既然我们希望选择最小的草种类型，那么我们选择草种类型 1。

我会将我们的草种类型决策收集到表格中。下面是我们刚才做出的决策，牧场 1 使用草种类型 1：

| **牧场** | **草种类型** |
| --- | --- |
| 1 | 1 |

继续前进。我们应该为牧场 2 选择什么草种？关心牧场 2 的牛是牛 1、牛 4 和牛 5，因此我们聚焦于这几头牛。牛 4 的牧场之一是牧场 1，我们为那个牧场选择了草种 1，所以草种 1 被排除作为牧场 2 的草种。如果我们为牧场 2 使用草种 1，那么我们将给牛 4 分配两个相同草种的牧场，这违反了规则。然而，牛 1 和牛 5 不排除任何其他草种，因为我们还没有为它们的牧场选择草种。因此，我们选择草种 2，最小的可用草种。现在的情况是：

| **牧场** | **草种** |
| --- | --- |
| 1 | 1 |
| 2 | 2 |

我们应该为牧场 3 选择什么草种？唯一关心牧场 3 的牛是牛 2。牛 2 的牧场是牧场 3 和牧场 5。然而，这头牛并没有排除任何草种，因为我们还没有为牧场 5 分配草种！为了选出最小的编号，我们为牧场 3 使用草种 1。以下是我们的下一个快照：

| **牧场** | **草种** |
| --- | --- |
| 1 | 1 |
| 2 | 2 |
| 3 | 1 |

我能看到我们从自上而下的设计中提炼出了三个任务。首先，我们需要获得关心当前牧场的牛。第二，我们需要确定这些牛排除的草种。第三，我们需要选择一个最小编号的未被排除的草种。每一个任务都非常适合成为一个函数。

继续前进。我们有三头牛关心牧场 4：牛 0、牛 1 和牛 3。牛 0 不排除任何草种，因为我们还没有为它的牧场分配草种。牛 1 排除了草种 2，因为我们为牧场 2（它的另一个牧场）分配了草种 2。牛 3 排除了草种 1，因为我们为牧场 1（它的另一个牧场）分配了草种 1。那么，最小的可用草种是 3，因此我们为牧场 4 使用草种 3：

| **牧场** | **草种** |
| --- | --- |
| 1 | 1 |
| 2 | 2 |
| 3 | 1 |
| 4 | 3 |

进入牧场 5。关心牧场 5 的牛是牛 0、牛 2 和牛 5。牛 0 排除了草种 3；牛 2 排除了草种 1；牛 5 排除了草种 2。所以草种 1、2 和 3 被排除。我们唯一的选择是草种 4。

真是差点！我们差点用尽了草种。幸运的是，没有其他牛关心牧场 5 并排除了草种 4。

或者，等等，这根本不是运气，因为问题描述中有这么一段：“保证每个牧场最多只有三头牛喜欢。”这意味着每个牧场最多会排除三个草种。我们永远不会卡住！我们甚至不需要担心之前的选择对下一个决策的影响。不管我们过去做了什么，我们总会有至少一个可用的草种。

让我们在表格中添加牧场 5：

| **牧场** | **草种** |
| --- | --- |
| 1 | 1 |
| 2 | 2 |
| 3 | 1 |
| 4 | 3 |
| 5 | 4 |

还有三个牧场要处理。但没有牛关心其中任何一个，所以我们可以在每种情况下都使用草地类型1。结果就是：

| **牧场** | **草地类型** |
| --- | --- |
| 1 | 1 |
| 2 | 2 |
| 3 | 1 |
| 4 | 3 |
| 5 | 4 |
| 6 | 1 |
| 7 | 1 |
| 8 | 1 |

我们可以从上到下读取草地类型，以获得这个例子的正确输出。输出如下：

12134111

### 自上而下设计

通过对我们需要完成的任务有了充分的理解，我们将转向这个问题的自上而下设计。

#### 顶层设计

在前一部分中，当我们处理一个测试用例时，发现了三个任务。在我们的程序能够解决这些任务之前，我们需要先读取输入，这就成了第四个任务。我们还需要编写输出。这将需要一些思考和几行代码，所以我们把它叫做第五个任务。

以下是我们五个主要任务：

1.  读取输入。

1.  确定关心当前牧场的牛。

1.  消除当前牧场的草地类型。

1.  为当前牧场选择编号最小的草地类型。

1.  编写输出。

正如我们在解决[第六章](ch06.xhtml#ch06)的动作模型时所做的那样，我们将从TODO注释的框架开始，并在解决每个TODO时将其移除。

我们开始时大部分是注释。由于我们需要在开始时打开文件并在结束时关闭文件，我还加了这段代码。

这里是我们开始的地方：

# 主程序

input_file = open('revegetate.in', 'r')

output_file = open('revegetate.out', 'w')

# TODO: 读取输入

# TODO: 确定关心牧场的牛

# TODO: 消除草地类型对于牧场的影响

# TODO: 为牧场选择编号最小的草地类型

# TODO: 编写输出

input_file.close()

output_file.close()

#### 任务 1：读取输入

读取输入的第一行，其中包括整数*n*和*m*，这是我们知道如何完成的任务。它足够简单，我认为我们不需要为此写一个函数，所以我们直接进行。接下来，我们需要读取*m*头牛的牧场信息，这时就需要一个函数。让我们去掉“读取输入”注释中的TODO，处理第一行输入，并调用read_cows函数，稍后我们将编写这个函数：

# 主程序

input_file = open('revegetate.in', 'r')

output_file = open('revegetate.out', 'w')

# 读取输入

lst = input_file.readline().split()

num_pastures = int(lst[0])

num_cows = int(lst[1])

❶ favorites = read_cows(input_file, num_cows)

# TODO: 确定关心牧场的牛

# TODO: 消除草地类型对于牧场的影响

# TODO: 为牧场选择编号最小的草地类型

# TODO: 编写输出

input_file.close()

output_file.close()

我们调用的read_cows函数 ❶ 将会接收一个已经打开的文件进行读取，读取每头牛的两个最喜欢的牧场。它将返回一个列表，其中每个内部列表包含一头牛的两个牧场编号。代码如下：

def read_cows(input_file, num_cows):

"""

input_file是一个打开用于读取的文件；牛的信息接下来将被读取。

num_cows是文件中的牛的数量。

从input_file读取牛最喜欢的牧场。

返回每头牛两个最喜欢的牧场的列表；

列表中的每个值都是一个包含两个值的列表，表示

每头牛最喜欢的牧场。

"""

favorites = []

for i in range(num_cows):

❶ lst = input_file.readline().split()

lst[0] = int(lst[0])

lst[1] = int(lst[1])

❷ favorites.append(lst)

return favorites

这个函数将牛的最喜欢的牧场累积到favorites列表中。它使用一个range循环，循环次数为num_cows，即每头牛一次。我们需要这个循环，因为读取的行数取决于文件中牛的数量。

在每次循环迭代中，我们读取下一行并将其拆分成两个部分❶。然后我们使用int将这些部分从字符串转换为整数。当我们将这个列表追加到favorites❷时，我们实际上是在追加一个包含两个整数的列表。

最后我们做的是返回一个包含每头牛最喜欢的牧场的列表。

在继续之前，让我们确保知道如何调用这个函数。我们将独立于我们正在构建的更大程序来练习调用它。像这样测试函数是有用的，这样我们可以在发现任何错误时进行修复。

使用你的文本编辑器创建一个名为*revegetate.in*的文件，并填入以下内容（与我们之前研究的测试用例相同）：

8 6

5 4

2 4

3 5

4 1

2 1

5 2

现在，在Python shell中输入我们的read_cows函数代码。

这是我们调用read_cows的方式：

>>> input_file = open('revegetate.in', 'r')

❶ >>> input_file.readline()

'8 6\n'

❷ >>> read_cows(input_file, 6)

[[5, 4], [2, 4], [3, 5], [4, 1], [2, 1], [5, 2]]

read_cows函数仅读取有关牛的信息。由于我们在程序外单独测试这个函数，在调用它❶之前，我们需要自己读取文件的第一行。然后，当我们调用read_cows时，返回一个列表，列出每头牛最喜欢的牧场。同时注意，我们是用打开的文件调用read_cows，而*不是*文件名❷。

确保在# 主程序注释之前包含我们的read_cows函数，以及我们将为其他任务编写的函数。然后我们可以继续进行任务2。

#### 任务2：识别牛

我们解决这个问题的整体策略是依次考虑每个牧场，决定使用哪种草类型。我们将在循环内组织这项工作，每次循环迭代负责播种一个牧场。对于每个牧场，我们需要识别出关心该牧场的牛，排除已用的草类型，并选择编号最小的可用草类型。这三项任务必须针对每个牧场执行，因此我们会将它们缩进到循环内。

我们将编写一个名为cows_with_favorite的函数，告诉我们关心当前牧场的牛。

这是目前我们的主程序：

# 主程序

input_file = open('revegetate.in', 'r')

output_file = open('revegetate.out', 'w')

# 读取输入

lst = input_file.readline().split()

num_pastures = int(lst[0])

num_cows = int(lst[1])

favorites = read_cows(input_file, num_cows)

for i in range(1, num_pastures + 1):

# 识别关心牧场的奶牛

❶ cows = cows_with_favorite(favorites, i)

# TODO: 为牧场消除草种

# TODO: 选择编号最小的草种作为牧场草种

# TODO: 写入输出

input_file.close()

output_file.close()

我们调用的 cows_with_favorite 函数❶接受一份奶牛的最爱牧场列表和一个牧场编号，并返回关心该牧场的奶牛。以下是代码：

def cows_with_favorite(favorites, pasture):

"""

favorites 是由 read_cows 返回的最爱牧场列表。

pasture 是牧场编号。

返回关心牧场的奶牛列表。

"""

cows = []

for i in range(len(favorites)):

if favorites[i][0] == pasture or favorites[i][1] == pasture:

cows.append(i)

return cows

该函数循环遍历 favorites，查找关心牧场编号 pasture 的奶牛。每头关心该牧场的奶牛都会被添加到最终返回的 cows 列表中。

做一个小测试。在Python shell中输入我们的 cows_with_favorite 函数。以下是我们将尝试的调用：

>>> cows_with_favorite([[5, 4], [2, 4], [3, 5]], 5)

这里有三头奶牛，我们在询问哪些奶牛关心牧场5。索引为0和2的奶牛关心牧场5，这正是该函数告诉我们的：

[0, 2]

#### 任务3：消除草种

现在我们知道了关心当前牧场的奶牛。我们的下一步是弄清楚这些奶牛会从当前牧场的草种中消除哪些选择。我们会消除已经在这些奶牛相关的牧场中使用过的草种。我们将编写一个名为 types_used 的函数，告诉我们哪些草种已经被使用（因此对于当前牧场被消除）。

这是我们的主程序，已经更新，加入了对该函数的调用：

# 主程序

input_file = open('revegetate.in', 'r')

output_file = open('revegetate.out', 'w')

# 读取输入

lst = input_file.readline().split()

num_pastures = int(lst[0])

num_cows = int(lst[1])

favorites = read_cows(input_file, num_cows)

❶ pasture_types = [0]

for i in range(1, num_pastures + 1):

# 识别关心牧场的奶牛

cows = cows_with_favorite(favorites, i)

# 为牧场消除草种

❷ eliminated = types_used(favorites, cows, pasture_types)

# TODO: 选择编号最小的草种作为牧场草种

# TODO: 写入输出

input_file.close()

output_file.close()

除了调用 types_used 函数❷外，我还添加了一个名为 pasture_types ❶ 的变量。这个变量所引用的列表将跟踪每个牧场的草种。

记住，牧场是从 1 开始编号的。然而，Python 的列表是从 0 开始索引的。我不喜欢这种不一致；如果我们直接开始向 pasture_types 添加草的类型，那么牧场 1 的草的类型就会位于索引 0，牧场 2 的草的类型位于索引 1，依此类推，总是会有一个偏差。因此，我在列表的开头加了一个假的 0 ❶；当我们之后为牧场 1 添加草的类型时，它会被放置在索引 1 处，以便与编号匹配。

假设我们已经找出了前四个牧场的草的类型。这时，pasture_types 可能是这样的：

[0, 1, 2, 1, 3]

如果我们想要牧场 1 的草的类型，我们查看索引 1；如果我们想要牧场 2 的草的类型，我们查看索引 2；以此类推。如果我们想要牧场 5 的草的类型？不行，我们还没找出来。如果 pasture_types 的长度是 5，这意味着我们只找出了前四个牧场的草的类型。一般来说，我们找出的草的类型数量比列表的长度少一个。

现在我们准备好使用 types_used 函数了。它有三个参数：每头牛最喜欢的牧场列表、关心当前牧场的牛列表，以及迄今为止已选定的草的类型。它返回一个已经被使用的草的类型列表，这些类型因此被排除在当前牧场之外。接下来我们就开始：

def types_used(favorites, cows, pasture_types):

"""

favorites 是由 read_cows 返回的最喜欢的牧场列表。

cows 是一组牛的列表。

pasture_types 是一个草的类型列表。

返回一个已经被牛使用过的草的类型列表。

"""

used = []

for cow in cows:

pasture_a = favorites[cow][0]

pasture_b = favorites[cow][1]

❶ 如果 pasture_a < len(pasture_types):

used.append(pasture_types[pasture_a])

❷ 如果 pasture_b < len(pasture_types):

used.append(pasture_types[pasture_b])

return used

每头牛有两个最喜欢的牧场，我将其分别称为 pasture_a 和 pasture_b。对于每一个牧场，我们会在 ❶ 和 ❷ 处检查是否已经为其选择了草的类型。如果该牧场已经是 pasture_types 中的一个索引，那么就说明草的类型已经选好了。所有这些草的类型都会被添加到 used 列表中，并在循环遍历所有相关牛后返回该列表。

如果多头牛使用相同的牧场——我们的代码会怎么做呢？让我们设计一个简单的测试用例来回答这个问题。

进入我们的 types_used 函数并在 Python shell 中运行它。以下是该函数的调用，让我们预测它返回什么：

>>> types_used([[5, 4], [2, 4], [3, 5]], [0, 1], [0, 1, 2, 1, 3])

让我们小心点，避免迷路。第一个参数给出了三头牛最喜欢的牧场。第二个参数给出了关心某个特定牧场的牛；这些是牛0和牛1。第三个参数给出了我们迄今为止决定的草的类型。

那么，奶牛 0 和奶牛 1 已经使用并因此淘汰了哪些草地类型呢？奶牛 0 关心牧场 4，而牧场 4 使用草地类型 3，因此草地类型 3 被淘汰。奶牛 1 关心牧场 2，而牧场 2 使用草地类型 2，因此草地类型 2 被淘汰。奶牛 1 还关心牧场 4——但我们已经知道，从奶牛 0 那里，牧场 4 的草地类型 3 被淘汰了。

我们函数的返回值是这个：

[3, 2, 3]

那里面有两个 3，一个来自奶牛 0，另一个来自奶牛 1。

可能看起来只保留一个 3 会更整洁，但实际上我们现有的重复值是完全可以接受的。如果某个草地类型在列表中，那么它就被淘汰了，不管它出现一次、两次还是三次。

#### 任务 4：选择最小编号的草地类型

在得到已淘汰的草地类型后，我们可以开始下一步任务：为当前牧场选择最小编号的可用草地类型。为了解决这个问题，我们将调用一个新的函数 `smallest_available`。它将返回我们应该为当前牧场使用的草地类型。

这是更新后的主程序，其中包含对 `smallest_available` 函数的调用：

# 主程序

input_file = open('revegetate.in', 'r')

output_file = open('revegetate.out', 'w')

# 读取输入

lst = input_file.readline().split()

num_pastures = int(lst[0])

num_cows = int(lst[1])

favorites = read_cows(input_file, num_cows)

pasture_types = [0]

for i in range(1, num_pastures + 1):

# 确定关心牧场的奶牛

cows = cows_with_favorite(favorites, i)

# 淘汰牧场的草地类型

eliminated = types_used(favorites, cows, pasture_types)

# 为牧场选择最小编号的草地类型

❶ pasture_type = smallest_available(eliminated)

❷ pasture_types.append(pasture_type)

# TODO: 写入输出

input_file.close()

output_file.close()

一旦我们获得当前牧场的最小编号草地类型 ❶，我们将它添加到已选择的草地类型列表 ❷ 中。

下面是 `smallest_available` 函数本身：

def smallest_available(used):

"""

used 是一个已使用的草地类型列表。

返回未使用的最小编号的草地类型。

"""

grass_type = 1

while grass_type in used:

grass_type = grass_type + 1

return grass_type

这个函数从草地类型 1 开始。然后它循环，直到找到一个未被使用的草地类型，每次迭代时将草地类型加 1。一旦找到一个空闲的草地类型，函数就返回它。请记住，最多只有四种草地类型中的三种被使用过，因此这个函数一定会成功。

#### 任务 5：编写输出

我们的答案就在 `pasture_types` 中！现在我们要做的就是输出它。这里是最终的主程序：

# 主程序

input_file = open('revegetate.in', 'r')

output_file = open('revegetate.out', 'w')

# 读取输入

lst = input_file.readline().split()

num_pastures = int(lst[0])

num_cows = int(lst[1])

favorites = read_cows(input_file, num_cows)

pasture_types = [0]

for i in range(1, num_pastures + 1):

# 确定关心牧场的牛

cows = cows_with_favorite(favorites, i)

# 排除牧场的草类型

eliminated = types_used(favorites, cows, pasture_types)

# 选择编号最小的草类型作为牧场

pasture_type = smallest_available(eliminated)

pasture_types.append(pasture_type)

# 写入输出

❶ pasture_types.pop(0)

❷ write_pastures(output_file, pasture_types)

input_file.close()

output_file.close()

在写入输出之前，我们移除pasture_types开头的虚假0❶。我们不想输出这个0，因为它不是一个真实的草类型。然后，我们调用write_pastures来实际写入输出❷。

现在我们只需要write_pastures函数。它接受一个打开供写入的文件和一个草类型列表，并将这些草类型输出到文件中。下面是代码：

def write_pastures(output_file, pasture_types):

"""

output_file是一个打开供写入的文件。

pasture_types是一个整数草类型的列表。

将牧场类型输出到output_file。

"""

pasture_types_str = []

❶ 对于pasture_type中的每一个牧场类型：

pasture_types_str.append(str(pasture_type))

❷ output = ''.join(pasture_types_str)

❸ output_file.write(output + '\n')

现在，pasture_types是一个整数列表。正如我们稍后会看到的，在这里使用字符串列表会更方便，所以我创建了一个新的列表，其中每个整数作为字符串❶。我没有修改pasture_types列表本身，因为那样可能会影响到调用此函数的代码。调用者调用此函数时，期望的只是将输出写入output_file，而不是其pasture_types列表被修改。这个函数不应该修改其列表参数。

为了产生输出，我们需要使用字符串而不是列表来调用write。并且我们需要无空格地输出列表中的字符串。这里使用字符串的join方法非常有效。正如我们在[第5章](ch05.xhtml#ch05)中的“将列表连接成字符串”部分所学，调用join方法的字符串作为分隔符，插入列表值之间。由于我们不希望在值之间有任何分隔符，我们使用空字符串作为分隔符❷。join方法仅对字符串列表有效，而不是整数列表，这也是为什么我在这个函数开始时将整数列表转换为字符串列表❶。

通过将输出作为单个字符串，我们可以将其写入文件❸。

##### 将所有内容结合起来

完整的程序见[示例7-2](ch07.xhtml#ch07ex02)。

def read_cows(input_file, num_cows):

"""

input_file是一个打开供读取的文件；接下来是读取牛群信息。

num_cows是文件中的牛的数量。

从input_file读取牛群喜欢的牧场。

返回每头牛最喜欢的两个牧场的列表；

列表中的每个值是一个包含两个值的列表，表示

每头牛最喜欢的牧场。

"""

favorites = []

对于i在范围(num_cows)内：

lst = input_file.readline().split()

lst[0] = int(lst[0])

lst[1] = int(lst[1])

favorites.append(lst)

返回favorites

def cows_with_favorite(favorites, pasture):

"""

favorites 是由 read_cows 返回的最喜欢的牧场列表。

pasture 是一个牧场编号。

返回关注牧场的奶牛列表。

"""

cows = []

for i in range(len(favorites)):

if favorites[i][0] == pasture or favorites[i][1] == pasture:

cows.append(i)

return cows

def types_used(favorites, cows, pasture_types):

"""

favorites 是由 read_cows 返回的最喜欢的牧场列表。

cows 是一组奶牛的列表。

pasture_types 是草地类型的列表。

返回一个包含奶牛已使用草地类型的列表。

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

used 是已使用的草地类型列表。

返回未在 used 中出现的最小编号的草地类型。

"""

grass_type = 1

while grass_type in used:

grass_type = grass_type + 1

return grass_type

def write_pastures(output_file, pasture_types):

"""

output_file 是一个用于写入的文件。

pasture_types 是一组整数类型的草地类型列表。

将 pasture_types 输出到 output_file。

"""

pasture_types_str = []

for pasture_type in pasture_types:

pasture_types_str.append(str(pasture_type))

output = ''.join(pasture_types_str)

output_file.write(output + '\n')

# 主程序

input_file = open('revegetate.in', 'r')

output_file = open('revegetate.out', 'w')

# 读取输入

lst = input_file.readline().split()

num_pastures = int(lst[0])

num_cows = int(lst[1])

favorites = read_cows(input_file, num_cows)

pasture_types = [0]

for i in range(1, num_pastures + 1):

# 确定关注牧场的奶牛

cows = cows_with_favorite(favorites, i)

# 消除牧场的草地类型

eliminated = types_used(favorites, cows, pasture_types)

# 选择牧场的最小编号草地类型

pasture_type = smallest_available(eliminated)

pasture_types.append(pasture_type)

# 写入输出

pasture_types.pop(0)

write_pastures(output_file, pasture_types)

input_file.close()

output_file.close()

*清单 7-2：解决农场播种问题*

我们做到了！一个令人畏惧的问题，通过自上而下的设计变得更加可控。随时将我们的作品提交给 USACO 评审。

初看一个问题时，很容易感到不知所措。但请记住，你不需要一次性解决所有问题。将问题拆解，解决每个你能解决的任务，你就离整体问题的解决方案更近了一步。你在 Python 知识和程序设计能力上已经取得了巨大的进步。解决这些问题已经触手可及！

**概念检查**

让我们考虑一个新的农场播种版本，其中没有限制哪些奶牛喜欢某个牧场的数量。一个牧场可能是四头、五头甚至更多奶牛的最爱。我们仍然不允许给一头奶牛分配两个草地类型相同的牧场。

假设我们正在解决这个新版本的问题，并且遇到一个测试用例，其中一个牧场是三头以上的奶牛最喜欢的。以下哪个选项适用于这个测试用例？  

A. *保证*使用四种草类型无法解决这个问题。  

B. 可能有办法解决这个问题。如果有，*可能*我们的原始解决方案（[Listing 7-2](ch07.xhtml#ch07ex02)）*能够*解决它。  

C. 可能有办法解决这个问题。如果有，*保证*我们的原始解决方案（[Listing 7-2](ch07.xhtml#ch07ex02)）*能够*解决它。  

D. 可能有办法解决这个问题。如果有，*保证*我们的原始解决方案（[Listing 7-2](ch07.xhtml#ch07ex02)）*无法*解决它。  

答案：B。我们可以找到一个由程序正确解决的测试用例，也可以找到一个可以解决但我们程序无法解决的测试用例。前者排除了A和D是正确答案；后者排除了C是正确答案。  

这是一个程序能正确解决的测试用例：  

2 4  

1 2  

1 2  

1 2  

1 2  

每个牧场都是四头奶牛的最爱。然而，我们只需使用两种草类型即可解决这个测试用例。试试我们的程序，你应该能看到它正确地解决了这个测试用例。  

现在有一个可以解决的测试用例，但我们的程序无法解决：  

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

我们程序中的错误在于为牧场1使用了草类型1。这样，它就不得不为牧场6使用草类型5——这是不允许的！——我们的程序失败了，但不要因此就认为没有办法解决这个测试用例。特别地，为牧场1使用草类型2，你应该能够找到仅使用四种草类型解决此测试用例的方法。用更复杂的程序也能解决这类问题，如果你感兴趣，我鼓励你自己思考一下。  

### 总结  

在这一章，我们学习了如何打开、读取、写入和关闭文件。文件在需要存储信息并稍后作为输入使用时非常有用。它们对于与用户传递信息也很有用。我们还学习到，处理文件的方式与处理标准输入和标准输出类似。  

在下一章，我们将学习如何在 Python 集合或字典中存储一组值。存储一组值——这听起来像是列表的功能。然而，我们会看到，集合和字典能让我们更容易地解决某些类型的问题。  

### 章节练习  

这里有一些练习题供你尝试。它们都是来自 USACO 的评测题，要求你读取和写入文件。这些练习还会要求你复习前面章节的内容。  

1.  USACO 2018年12月铜奖竞赛问题《混合牛奶》  

1.  USACO 2017年2月铜奖竞赛问题《为什么奶牛过马路》  

1.  USACO 2017年美国公开赛铜奖竞赛问题《迷失的牛》  

1.  USACO 2019年12月铜奖竞赛问题《牛体操》  

1.  USACO 2017年美国公开赛铜奖竞赛问题《牛的基因组》  

1.  USACO 2018年美国公开赛青铜奖竞赛问题：井字棋队伍

1.  USACO 2019年2月青铜奖竞赛问题：睡意朦胧的牛群

### 备注

《作文格式》最初来源于USACO 2020年1月的青铜奖竞赛。 《农场播种》最初来源于USACO 2019年2月的青铜奖竞赛。

除了文本文件之外，还有许多种类的文件。你可能会喜欢处理HTML文件、Excel电子表格、PDF文件、Word文档或图像文件。Python可以帮助你！有关更多信息，请参阅*《用Python自动化无聊的事情》*（第二版），作者：Al Sweigart（No Starch Press，2019年）。

“也许更好的诗歌”这一句话来自J. C. R. Licklider，引用自*《计算机与未来世界》*，由马丁·格林伯格编辑（麻省理工学院出版社，1962年）：

但是有些人用我们说的语言写诗。也许在未来数字计算机的语言中，写出的诗会比用英语写的更好。
