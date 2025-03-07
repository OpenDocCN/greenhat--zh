## 第八章：语言

![](img/circleart.png)

在本章中，我们将进入人类语言的复杂世界。我们将首先讨论语言与数学之间的差异，这些差异使得语言算法变得困难。接下来，我们将构建一个空格插入算法，它能够处理任何语言的文本，并在缺少空格的地方插入空格。之后，我们将构建一个短语补全算法，能够模仿作家的风格，并找到短语中最合适的下一个单词。

本章中的算法 heavily 依赖于我们之前没有使用过的两种工具：列表推导式和语料库。*列表推导式*使我们能够利用循环和迭代的逻辑快速生成列表。它们在 Python 中经过优化，运行非常快，而且容易简洁地编写，但它们可能难以阅读，且其语法需要一些时间来适应。*语料库*是指一组文本，它将“教”我们的算法我们希望它使用的语言和风格。

## 为什么语言算法如此困难

将算法思维应用到语言上，至少可以追溯到笛卡尔，他注意到，尽管数字是无限的，但任何具备基本算术知识的人都知道如何创建或解释一个他们从未遇到过的数字。例如，也许你从未遇到过数字 14,326——从未数过那么高的数，没看过涉及这么多美元的财务报告，也没在键盘上敲过正好那几个键。但我敢肯定，你可以轻松理解这个数字有多大，哪些数字比它大，哪些比它小，以及如何在方程中操作它。

使我们轻松理解前所未见的数字的算法，实际上只是将 10 个数字（0–9）按顺序记住，并结合使用位置系统。我们知道 14,326 比 14,325 大一个单位，因为数字 6 紧跟在数字 5 后面，它们在各自的数字中占据相同的位置，其他所有位置的数字都是一样的。知道这些数字和位置系统让我们能立刻了解 14,326 与 14,325 的相似之处，以及它们都大于 12 且小于 1,000,000。我们还可以一眼看出，14,326 在某些方面与 4,326 相似，但在大小上有很大的区别。

语言则不同。如果你正在学习英语，当你第一次看到单词*stage*时，你不能仅凭它与*stale*、*stake*、*state*、*stave*、*stade* 或 *sage* 的相似性来推断其含义，尽管这些词与*stage*的区别就像 14,326 与 14,325 的区别一样。你也不能仅凭单词的音节数和字符数来推测细菌比麋鹿要大。即使是那些看似可靠的语言规则，比如在英语中通过加*s*来构成复数，当我们推测“princes”比“princess”少一些某物时，也会把我们引入歧途。

为了使用算法处理语言，我们必须让语言变得更简单，这样我们迄今为止探索的简短数学算法就可以可靠地与之配合使用，或者让我们的算法更智能，这样它们就能够处理人类语言自然发展所带来的混乱复杂性。我们将选择后者。

## 空格插入

假设你是一个大型老公司中的首席算法官，该公司拥有一个满是手写纸质记录的仓库。首席记录数字化官员一直在进行一个长期项目，将这些纸质记录扫描成图像文件，然后使用文本识别技术将图像转换为可以轻松存储在公司数据库中的文本。然而，部分手写记录非常难以辨认，而文本识别技术也不完美，因此从纸质记录中提取的最终数字文本有时会出现错误。你只收到了数字化后的文本，要求你找到一种方法来纠正这些错误，而不参考纸质原件。

假设你将第一个数字化的句子读取到 Python 中，并发现它是 G. K. Chesterton 的名言：“唯一完全神圣的事情，唯一在地球上看到的上帝乐园的片刻，是与一场失败的战斗作斗争——而且没有失去它。”你将这段不完全数字化的文本存储在一个叫做`text`的变量中：

```py
text = "The oneperfectly divine thing, the oneglimpse of God's paradisegiven on earth, is to fight a losingbattle - and notlose it."
```

你会注意到这段文本是英文的，虽然每个单词的拼写都是正确的，但其中缺少了空格：`oneperfectly`实际上应该是`one perfectly`，`paradisegiven`应该是`paradise given`，等等。（缺少空格对于人类来说不常见，但文本识别技术经常会犯这种错误。）为了完成你的任务，你需要在文本中的适当位置插入空格。对于一位流利的英语使用者来说，这个任务手动完成可能不难。然而，想象一下你需要为数百万页扫描文件快速完成这一任务——显然，你需要编写一个能够为你自动完成这一任务的算法。

### 定义单词列表并查找单词

我们首先要做的事情是教我们的算法一些英语单词。这并不困难：我们可以定义一个叫做`word_list`的列表，并用单词填充它。我们先从几个单词开始：

```py
word_list = ['The','one','perfectly','divine']
```

在这一章中，我们将使用列表推导式来创建和操作列表，相信你习惯之后会喜欢这种方式。以下是一个非常简单的列表推导式，它创建了我们`word_list`的副本：

```py
word_list_copy = [word for word in word_list]
```

你可以看到，语法`for word in word_list`与`for`循环的语法非常相似。但我们不需要冒号或额外的行。在这种情况下，列表推导式尽可能简单，只是指定我们希望`word_list`中的每个单词都出现在我们新的列表`word_list_copy`中。这可能没什么用，但我们可以简洁地添加逻辑使它更有用。例如，如果我们想找到`word_list`中每个包含字母*n*的单词，只需简单地添加一个`if`语句：

```py
has_n = [word for word in word_list if 'n' in word]
```

我们可以运行`print(has_n)`来查看结果是否如我们所料：

```py
['one', 'divine']
```

在本章后面，你将看到更复杂的列表推导式，包括一些包含嵌套循环的例子。然而，它们都遵循相同的基本模式：一个`for`循环指定迭代，附加的`if`语句描述我们想要选择的最终列表输出的逻辑。

我们将使用 Python 的`re`模块来访问文本处理工具。`re`模块的一个有用函数是`finditer()`，它可以在我们的文本中搜索，找到`word_list`中任何单词的位置。我们可以像下面这样在列表推导式中使用`finditer()`：

```py
import re
locs = list(set([(m.start(),m.end()) for word in word_list for m in re.finditer(word, text)]))
```

这一行有点密集，花点时间确保你理解它。我们定义了一个名为`locs`的变量，`locs`是“位置”的缩写；这个变量将包含文本中`word_list`中每个单词的位置。我们将使用列表推导式来获得这个位置列表。

列表推导式发生在方括号（`[]`）内。我们使用`for word in word_list`来迭代`word_list`中的每个单词。对于每个单词，我们调用`re.finditer()`，它会在我们的文本中找到该单词，并返回该单词出现的每个位置的列表。我们迭代这些位置，每个位置存储在`m`中。当我们访问`m.start()`和`m.end()`时，我们会分别得到该单词在文本中开始和结束的位置。注意——并习惯于——`for`循环的顺序，因为有些人会觉得它与他们预期的顺序相反。

整个列表推导式被`list(set())`包裹住。这是一种方便的方法，可以得到一个仅包含唯一值且没有重复项的列表。我们的列表推导式可能包含多个相同的元素，但将其转换为集合会自动去除重复项，然后再将其转换回列表，得到我们想要的格式：一个包含唯一单词位置的列表。你可以运行`print(locs)`来查看整个操作的结果：

```py
[(17, 23), (7, 16), (0, 3), (35, 38), (4, 7)]
```

在 Python 中，像这样的有序对被称为*元组*，这些元组显示了`word_list`中每个单词在文本中的位置。例如，当我们运行`text[17:23]`（使用前面列表中第三个元组的数字时），我们发现它是`divine`。这里，`d`是我们文本中的第 17 个字符，`i`是第 18 个字符，依此类推，直到`e`，`divine`的最后一个字母，是我们文本中的第 22 个字符，因此元组以 23 结尾。你可以检查其他元组，它们也指示了`word_list`中单词的位置。

注意，`text[4:7]`是`one`，而`text[7:16]`是`perfectly`。单词`one`的结尾与单词`perfectly`的开头相接，没有任何间隔的空格。如果我们没有通过阅读文本立即注意到这一点，我们可以通过查看`locs`变量中的元组(4, 7)和(7, 16)来捕捉到这一点：由于 7 是(4, 7)的第二个元素，同时也是(7, 16)的第一个元素，我们知道一个单词的结尾恰好与另一个单词的开头相同。为了找到需要插入空格的位置，我们将寻找这样的情况：一个有效单词的结尾恰好与另一个有效单词的开头重合。

### 处理复合词

不幸的是，两个有效单词并排出现而没有空格，并不能作为缺少空格的确凿证据。考虑单词*butterfly*。我们知道*butter*是一个有效单词，*fly*也是一个有效单词，但我们不能简单地得出*butterfly*是错误拼写的结论，因为*butterfly*也是一个有效单词。所以我们不仅需要检查那些没有空格而并排出现的有效单词，还需要检查那些没有空格并排在一起后，组合成另一个无效单词的情况。这意味着在我们的文本中，我们需要检查`oneperfectly`是否是一个有效单词，`paradisegiven`是否是一个有效单词，等等。

为了检查这一点，我们需要找到文本中的所有空格。我们可以查看两个连续空格之间的所有子字符串，并将其称为潜在单词。如果一个潜在单词不在我们的单词列表中，那么我们将认为它是无效的。我们可以检查每个无效单词，看看它是否由两个较小的单词组合而成；如果是，我们将认为缺少了一个空格，并将其添加回去，正好插入在两个有效单词之间，它们合并成了无效单词。

### 检查现有空格之间的潜在单词

我们可以再次使用`re.finditer()`来查找文本中的所有空格，并将它们存储在名为`spacestarts`的变量中。我们还将向`spacestarts`变量添加两个元素：一个表示文本开始位置，另一个表示文本结束位置。这样可以确保我们找到每个潜在的单词，因为文本最开始和最末尾的单词是唯一不在空格之间的单词。我们还添加了一行来对`spacestarts`列表进行排序：

```py
spacestarts = [m.start() for m in re.finditer(' ', text)]
spacestarts.append(-1)
spacestarts.append(len(text))
spacestarts.sort()
```

列表 `spacestarts` 记录了文本中空格的位置。我们通过使用列表推导式和 `re.finditer()` 工具获得这些位置。在这种情况下，`re.finditer()` 查找文本中每个空格的位置并将其存储在一个列表中，每个元素被称为 `m`。对于每个 `m` 元素，也就是空格，我们通过使用 `start()` 函数获取空格的起始位置。我们正在寻找那些空格之间的潜在单词。我们还需要另一个列表，记录空格后紧跟的字符位置，这些将是每个潜在单词第一个字符的位置。我们将这个列表称为 `spacestarts_affine`，因为从技术角度来说，这个新列表是 `spacestarts` 列表的仿射变换。*仿射* 通常用于指代线性变换，比如在每个位置上加 1，我们将在这里这么做。我们还将对这个列表进行排序：

```py
spacestarts_affine = [ss+1 for ss in spacestarts]
spacestarts_affine.sort()
```

接下来，我们可以获取所有位于两个空格之间的子字符串：

```py
between_spaces = [(spacestarts[k] + 1,spacestarts[k + 1]) for k in range(0,len(spacestarts) - 1 )]
```

我们在这里创建的变量叫做 `between_spaces`，它是一个包含元组的列表，元组的形式是 (*<子字符串起始位置>*, *<子字符串结束位置>*)，像 (17, 23)。我们通过列表推导式来获取这些元组。这个列表推导式会遍历 `k`。在这种情况下，`k` 取值为从 0 到 `spacestarts` 列表长度减一之间的整数。对于每个 `k`，我们将生成一个元组。元组的第一个元素是 `spacestarts[k]+1`，它是每个空格位置之后的位置。第二个元素是 `spacestarts[k+1]`，它是文本中下一个空格的位置。通过这种方式，我们最终输出的将是指示空格之间每个子字符串起始和结束位置的元组。

现在，考虑所有位于空格之间的潜在单词，并找出那些无效的单词（不在我们的单词列表中）：

```py
between_spaces_notvalid = [loc for loc in between_spaces if \text[loc[0]:loc[1]] not in word_list]
```

查看 `between_spaces_notvalid`，我们可以看到它是一个包含所有无效潜在单词位置的列表：

```py
[(4, 16), (24, 30), (31, 34), (35, 45), (46, 48), (49, 54), (55, 68), (69, 71), (72, 78), (79, 81), (82, 84), (85, 90), (91, 92), (93, 105), (106, 107), (108, 111), (112, 119), (120, 123)]
```

我们的代码认为所有这些位置都指向无效的单词。然而，如果你查看这里提到的一些单词，它们看起来是相当有效的。例如，`text[103:106]` 输出了有效单词 `and`。我们的代码认为 `and` 是无效单词的原因是它不在我们的单词列表中。当然，我们可以手动将其添加到我们的单词列表中，并继续使用这种方法，因为我们需要让代码识别单词。但是请记住，我们希望这个空格插入算法能够处理数百万页扫描的文本，而这些文本可能包含成千上万个独特的单词。如果我们能够导入一个已经包含大量有效英语单词的单词列表，那将会很有帮助。这样的单词集合被称为 *语料库*。

### 使用导入的语料库来检查有效单词

幸运的是，有一些现成的 Python 模块可以让我们只用几行代码就能导入完整的语料库。首先，我们需要下载语料库：

```py
import nltk
nltk.download('brown')
```

我们已经从名为`nltk`的模块中下载了一个名为`brown`的语料库。接下来，我们将导入这个语料库：

```py
from nltk.corpus import brown
wordlist = set(brown.words())
word_list = list(wordlist)
```

我们已经导入了语料库，并将其中的单词集合转换成了一个 Python 列表。然而，在使用这个新的`word_list`之前，我们应该进行一些清理，去除它认为是单词但实际上是标点符号的部分：

```py
word_list = [word.replace('*','') for word in word_list]
word_list = [word.replace('[','') for word in word_list]
word_list = [word.replace(']','') for word in word_list]
word_list = [word.replace('?','') for word in word_list]
word_list = [word.replace('.','') for word in word_list]
word_list = [word.replace('+','') for word in word_list]
word_list = [word.replace('/','') for word in word_list]
word_list = [word.replace(';','') for word in word_list]
word_list = [word.replace(':','') for word in word_list]
word_list = [word.replace(',','') for word in word_list]
word_list = [word.replace(')','') for word in word_list]
word_list = [word.replace('(','') for word in word_list]
word_list.remove('')
```

这些行使用了`remove()`和`replace()`函数，将标点符号替换为空字符串，并去除空字符串。现在我们有了一个合适的单词列表，我们将能够更准确地识别无效单词。我们可以使用新的`word_list`重新运行无效单词检查，并获得更好的结果：

```py
between_spaces_notvalid = [loc for loc in between_spaces if \text[loc[0]:loc[1]] not in word_list]
```

当我们打印列表`between_spaces_notvalid`时，得到的是一个更短且更准确的列表：

```py
[(4, 16), (24, 30), (35, 45), (55, 68), (72, 78), (93, 105), (112, 119), (120, 123)]
```

现在我们已经找到了文本中无效的潜在单词，我们将在单词列表中查找可能组合成这些无效单词的单词。我们可以从查找那些在空格后开始的单词入手。这些单词可能是无效单词的前半部分：

```py
partial_words = [loc for loc in locs if loc[0] in spacestarts_affine and \loc[1] not in spacestarts]
```

我们的列表推导式会遍历`locs`变量中的每个元素，该变量包含文本中每个单词的位置。它会检查`locs[0]`（单词的起始位置）是否在`spacestarts_affine`中，`spacestarts_affine`是一个包含紧接着空格后的字符的列表。接着，它会检查`loc[1]`是否不在`spacestarts`中，`spacestarts`检查单词是否在空格开始的位置结束。如果一个单词在空格后开始，并且不在空格的位置结束，我们将它放入`partial_words`变量中，因为这可能是一个需要在其后插入空格的单词。

接下来，我们来查找以空格结尾的单词。这些单词可能是无效单词的后半部分。为了找到它们，我们对之前的逻辑做了一些小的调整：

```py
partial_words_end = [loc for loc in locs if loc[0] not in spacestarts_affine \and loc[1] in spacestarts]
```

现在我们可以开始插入空格。

### 查找潜在单词的前半部分和后半部分

我们从`oneperfectly`开始插入空格。我们将定义一个名为`loc`的变量，存储`oneperfectly`在文本中的位置：

```py
loc = between_spaces_notvalid[0]
```

我们现在需要检查`partial_words`中的任何单词是否可能是`oneperfectly`的前半部分。要成为`oneperfectly`的前半部分，一个有效的单词必须在文本中与`oneperfectly`有相同的起始位置，但结束位置不同。我们将编写一个列表推导式，找到每个有效单词的结束位置，这些单词与`oneperfectly`的起始位置相同：

```py
endsofbeginnings = [loc2[1] for loc2 in partial_words if loc2[0] == loc[0] \and (loc2[1] - loc[0]) > 1]
```

我们指定了`loc2[0] == loc[0]`，这意味着我们的有效单词必须与`oneperfectly`在同一位置开始。我们还指定了`(loc2[1]-loc[0])>1`，这确保我们找到的有效单词长度大于一个字符。这不是严格必要的，但它可以帮助我们避免误报。想想像*avoid*、*aside*、*along*、*irate*和*iconic*这样的单词，其中第一个字母本身可能被认为是一个单词，但可能不应该这么处理。

我们的列表`endsofbeginnings`应该包含每个有效单词的结束位置，这些单词的开始位置与`oneperfectly`相同。我们可以使用列表推导式创建一个类似的变量，叫做`beginningsofends`，它会找到每个有效单词的开始位置，这些单词的结束位置与`oneperfectly`相同：

```py
beginningsofends = [loc2[0] for loc2 in partial_words_end if loc2[1] == loc[1] and \(loc2[1] - loc[0]) > 1]
```

我们指定了`loc2[1] == loc[1]`，这意味着我们的有效单词必须和`oneperfectly`在同一位置结束。我们还指定了`(loc2[1]-loc[0])>1`，这确保我们找到的有效单词长度大于一个字符，就像我们之前所做的那样。

我们快到了；我们只需要找出是否有任何位置同时出现在`endsofbeginnings`和`beginningsofends`中。如果有，说明我们的无效单词确实是由两个有效单词组成的，且中间没有空格。我们可以使用`intersection()`函数来找到两个列表中共享的所有元素：

```py
pivot = list(set(endsofbeginnings).intersection(beginningsofends))
```

我们再次使用`list(set())`语法；和之前一样，这是为了确保我们的列表只包含唯一值，没有重复。我们将结果称为`pivot`。有可能`pivot`会包含多个元素，这意味着有多个可能的有效单词组合可以构成我们的无效单词。如果发生这种情况，我们必须决定哪个组合是原作者的意图。这无法确定。例如，考虑无效单词*choosespain*。有可能这个无效单词来自伊比利亚航空的旅行宣传单（“Choose Spain!”），但也可能来自描述一个受虐狂（“chooses pain”）。由于我们语言中单词的数量庞大，且它们可以以多种方式组合，有时我们无法确定哪个是正确的。一个更复杂的方法会考虑上下文——*choosespain*周围的其他单词是否倾向于讲橄榄和斗牛，或者是鞭子和多余的牙医预约。这样的做法很难做到好，而且不可能做到完美，这再次说明了语言算法的难度。在我们的情况下，我们将选择`pivot`中最小的元素，不是因为它一定是正确的，而仅仅因为我们必须选一个：

```py
import numpy as np
pivot = np.min(pivot)
```

最后，我们可以写一行代码，用两个有效的组成单词加一个空格来替换我们无效的单词：

```py
textnew = text
textnew = textnew.replace(text[loc[0]:loc[1]],text[loc[0]:pivot]+' '+text[pivot:loc[1]])
```

如果我们打印出这个新的文本，我们可以看到它已经正确地在拼写错误的`oneperfectly`中插入了空格，尽管它还没有在其他拼写错误中插入空格。

```py
The one perfectly divine thing, the oneglimpse of God's paradisegiven on earth, is to fight a losingbattle - and notlose it.
```

我们可以将所有这些功能结合成一个漂亮的函数，如 Listing 8-1 所示。这个函数将使用一个`for`循环，在每一对有效单词组合成无效单词的地方插入空格。

```py
def insertspaces(text,word_list):

    locs = list(set([(m.start(),m.end()) for word in word_list for m in re.finditer(word, \text)]))
 spacestarts = [m.start() for m in re.finditer(' ', text)]
    spacestarts.append(-1)
    spacestarts.append(len(text))
    spacestarts.sort()
    spacestarts_affine = [ss + 1 for ss in spacestarts]
    spacestarts_affine.sort()
    partial_words = [loc for loc in locs if loc[0] in spacestarts_affine and loc[1] not in \    spacestarts]
    partial_words_end = [loc for loc in locs if loc[0] not in spacestarts_affine and loc[1] \    in spacestarts]
    between_spaces = [(spacestarts[k] + 1,spacestarts[k+1]) for k in \    range(0,len(spacestarts) - 1)]
    between_spaces_notvalid = [loc for loc in between_spaces if text[loc[0]:loc[1]] not in \    word_list]
    textnew = text
    for loc in between_spaces_notvalid:
        endsofbeginnings = [loc2[1] for loc2 in partial_words if loc2[0] == loc[0] and \    (loc2[1] - loc[0]) > 1]
        beginningsofends = [loc2[0] for loc2 in partial_words_end if loc2[1] == loc[1] and \    (loc2[1] - loc[0]) > 1]
        pivot = list(set(endsofbeginnings).intersection(beginningsofends))
        if(len(pivot) > 0):
            pivot = np.min(pivot)
            textnew = textnew.replace(text[loc[0]:loc[1]],text[loc[0]:pivot]+' \            '+text[pivot:loc[1]])
    textnew = textnew.replace('  ',' ')
    return(textnew)
```

Listing 8-1: 一个插入空格的函数，结合了本章中大部分代码

然后我们可以定义任何文本并调用我们的函数，如下所示：

```py
text = "The oneperfectly divine thing, the oneglimpse of God's paradisegiven on earth, is to \fight a losingbattle - and notlose it."
print(insertspaces(text,word_list))
```

我们看到的输出正如我们预期的那样，空格已完美插入：

```py
The one perfectly divine thing, the one glimpse of God's paradise given on earth, is to fight a losing battle - and not lose it.
```

我们已经创建了一个可以正确在英语文本中插入空格的算法。需要考虑的一点是，你是否也能在其他语言中做到这一点。可以的——只要你读取适合目标语言的良好语料库，定义我们在此示例中定义并调用的`word_list`，该函数就可以正确地在任何语言的文本中插入空格。它甚至可以纠正你从未学过或听说过的语言中的文本。尝试不同的语料库、不同的语言和不同的文本，看看你能得到什么样的结果，你就能一窥语言算法的强大功能。

## 短语补全

假设你正在为一个初创公司提供算法咨询服务，该公司正在构建搜索引擎并计划添加一些新功能。他们想要添加短语补全功能，以便能够为用户提供搜索建议。例如，当用户输入`peanut``butter and`时，搜索建议功能可能会建议添加单词`jelly`。当用户输入`squash`时，搜索引擎可能会同时建议`court`和`soup`。

构建这个功能很简单。我们将从一个语料库开始，就像我们在空间检查器中做的那样。在这种情况下，我们不仅关注语料库中的单个单词，还关注单词是如何组合在一起的，因此我们将从语料库中编译 n-gram 列表。*n-gram*仅仅是指一组*连续出现的 n*个单词。例如，句子“Reality is not always probable, or likely”由七个单词组成，这是伟大的 Jorge Luis Borges 曾经说过的话。1-gram 是单个单词，所以这个句子的 1-grams 是*reality*、*is*、*not*、*always*、*probable*、*or*和*likely*。2-gram 是每一对连续的两个单词，包括*reality**is*、*is not*、*not always*、*always probable*，等等。3-gram 是*reality is not*、*is not always*，依此类推。

### 分词和获取 n-gram

我们将使用一个名为`nltk`的 Python 模块来简化 n-gram 的收集。我们首先对文本进行分词。*分词*仅仅是指将一个字符串拆分成其组成的单词，忽略标点符号。例如：

```py
from nltk.tokenize import sent_tokenize, word_tokenize
text = "Time forks perpetually toward innumerable futures"
print(word_tokenize(text))
```

我们看到的结果是：

```py
['Time', 'forks', 'perpetually', 'toward', 'innumerable', 'futures']
```

我们可以对文本进行分词并获得 n-gram，如下所示：

```py
import nltk
from nltk.util import ngrams
token = nltk.word_tokenize(text)
bigrams = ngrams(token,2)
trigrams = ngrams(token,3)
fourgrams = ngrams(token,4)
fivegrams = ngrams(token,5)
```

或者，我们可以将所有的 n-gram 放入一个名为`grams`的列表中：

```py
grams = [ngrams(token,2),ngrams(token,3),ngrams(token,4),ngrams(token,5)]
```

在这种情况下，我们已经获得了一个短句文本的分词和 n-grams 列表。然而，为了拥有一个通用的短语补全工具，我们需要一个相当大的语料库。我们用于空间插入的`brown`语料库不适用，因为它只包含单个单词，因此我们无法获取它的 n-grams。

我们可以使用的一个语料库是由 Google 的 Peter Norvig 提供的在线文学文本合集，网址为[`norvig.com/big.txt`](http://norvig.com/big.txt)。在本章的示例中，我下载了莎士比亚全集的文件，它可以在* [`www.gutenberg.org/files/100/100-0.txt`](http://www.gutenberg.org/files/100/100-0.txt)*免费获取，然后删除了 Project Gutenberg 的页眉文本。你也可以使用马克·吐温的全集，网址为* [`www.gutenberg.org/cache/epub/3200/pg3200.txt`](http://www.gutenberg.org/cache/epub/3200/pg3200.txt)*。你可以通过以下方式将语料库读取到 Python 中：

```py
import requests
file = requests.get('http://www.bradfordtuckfield.com/shakespeare.txt')
file = file.text
text = file.replace('\n', '')
```

在这里，我们使用`requests`模块直接从托管网站读取包含莎士比亚全集的文本文件，然后将其读取到名为`text`的 Python 变量中。

在读取所选语料库后，重新运行创建`grams`变量的代码。以下是新定义的`text`变量的代码：

```py
token = nltk.word_tokenize(text)
bigrams = ngrams(token,2)
trigrams = ngrams(token,3)
fourgrams = ngrams(token,4)
fivegrams = ngrams(token,5)
grams = [ngrams(token,2),ngrams(token,3),ngrams(token,4),ngrams(token,5)]
```

### 我们的策略

我们生成搜索建议的策略很简单。当用户输入搜索内容时，我们检查搜索中包含多少个单词。换句话说，用户输入一个 n-gram，我们确定*n*的值。当用户搜索 n-gram 时，我们帮助他们扩展搜索，因此我们将建议一个*n* + 1-gram。我们将在语料库中搜索并找到所有首个*n*元素与我们的 n-gram 匹配的*n* + 1-grams。例如，用户可能搜索`crane`（起重机），一个 1-gram，而我们的语料库可能包含 2-gram `crane feather`（起重机羽毛）、`crane operator`（起重机操作员）和`crane neck`（起重机颈部）。每一个都是我们可以提供的潜在搜索建议。

我们可以在此停止，提供每个首个*n*元素与用户输入的*n* + 1-gram 匹配的所有*n* + 1-gram。然而，并不是所有的建议都是同等优秀的。例如，如果我们为一个搜索工业建筑设备手册的定制引擎工作，那么`crane operator`（起重机操作员）可能比`crane feather`（起重机羽毛）更相关、更有用。确定哪个*n* + 1-gram 是最佳建议的最简单方法是提供在我们的语料库中出现频率最高的那个。

因此，我们的完整算法是：用户搜索一个 n-gram，我们找到所有首个*n*元素与用户的 n-gram 匹配的*n* + 1-grams，并推荐在语料库中最常出现的匹配*n* + 1-gram。

### 查找候选*n* + 1-grams

为了找到将构成我们搜索建议的*n* + 1-grams，我们需要知道用户搜索词的长度。假设搜索词是`life is a`，意思是我们正在寻找如何完成短语“life is a . . .”的建议。我们可以使用以下简单的代码来获取搜索词的长度：

```py
from nltk.tokenize import sent_tokenize, word_tokenize
search_term = 'life is a'
split_term = tuple(search_term.split(' '))
search_term_length = len(search_term.split(' '))
```

现在我们知道搜索词的长度，知道了*n*的值——它是 3。记住，我们将返回最频繁的*n* + 1-gram（即 4-gram）给用户。所以我们需要考虑不同*n* + 1-gram 的频率差异。我们将使用一个名为`Counter()`的函数，它会计算每个*n* + 1-gram 在我们语料库中出现的次数。

```py
from collections import Counter
counted_grams = Counter(grams[search_term_length - 1])
```

这一行仅从我们的`grams`变量中选择了*n* + 1-gram。应用`Counter()`函数会创建一个元组列表。每个元组的第一个元素是*n* + 1-gram，第二个元素是该*n* + 1-gram 在我们语料库中的出现频率。例如，我们可以打印`counted_grams`的第一个元素：

```py
print(list(counted_grams.items())[0])
```

输出显示我们语料库中的第一个*n* + 1-gram，并告诉我们它在整个语料库中仅出现一次：

```py
(('From', 'fairest', 'creatures', 'we'), 1)
```

这个 n-gram 是莎士比亚《第一首十四行诗》的开头。看看我们在莎士比亚作品中随机找到的一些有趣的 4-grams 非常有趣。例如，如果你运行`print(list(counted_grams)[10])`，你可以看到莎士比亚作品中的第十个 4-gram 是“rose might never die”。如果你运行`print(list(counted_grams)[240000])`，你可以看到第 240,000 个 n-gram 是“I shall command all”。第 323,002 个是“far more glorious star”，第 328,004 个是“crack my arms asunder”。但我们要做的是短语补全，而不仅仅是*n* + 1-gram 浏览。我们需要找到一个*n* + 1-grams 的子集，其前*n*个元素与我们的搜索词匹配。我们可以通过以下方式来做到这一点：

```py
matching_terms = [element for element in list(counted_grams.items()) if \element[0][:-1] == tuple(split_term)]
```

这个列表推导式会遍历每个*n* + 1-gram，并在此过程中调用每个元素。对于每个元素，它检查`element[0][:-1]==tuple(split_term)`。这个等式的左边，`element[0][:-1]`，简单地获取每个*n* + 1-gram 的前*n*个元素：`[:-1]`是一个方便的方式来忽略列表的最后一个元素。等式的右边，`tuple(split_term)`，是我们要搜索的 n-gram（“life is a”）。所以我们在检查那些前*n*个元素与我们感兴趣的 n-gram 相同的*n* + 1-grams。所有匹配的词汇都存储在我们的最终输出中，名为`matching_terms`。

### 基于频率选择短语

我们的`matching_terms`列表包含完成任务所需的所有内容；它由那些前*n*个元素与搜索词匹配的*n* + 1-grams 组成，并包括它们在我们语料库中的频率。只要匹配的词汇列表中至少有一个元素，我们就可以找到在语料库中出现最频繁的元素，并将其建议给用户作为完整的短语。以下代码片段可以完成这个任务：

```py
if(len(matching_terms)>0):
    frequencies = [item[1] for item in matching_terms]
    maximum_frequency = np.max(frequencies)
    highest_frequency_term = [item[0] for item in matching_terms if item[1] == \maximum_frequency][0]
    combined_term = ' '.join(highest_frequency_term)
```

在这个代码片段中，我们首先定义了`frequencies`，它是一个包含我们语料库中每个与搜索词匹配的*n* + 1-gram 频率的列表。接着，我们使用了`numpy`模块的`max()`函数来找到这些频率中的最大值。然后，我们使用了另一个列表推导式来获取语料库中出现频率最高的第一个*n* + 1-gram，最后我们创建了一个`combined_term`，它是一个将搜索词中所有单词连接在一起的字符串，单词之间用空格分隔。

最后，我们可以将所有代码整合成一个函数，如列表 8-2 所示。

```py
def search_suggestion(search_term, text):
    token = nltk.word_tokenize(text)
    bigrams = ngrams(token,2)
    trigrams = ngrams(token,3)
    fourgrams = ngrams(token,4)
    fivegrams = ngrams(token,5)
    grams = [ngrams(token,2),ngrams(token,3),ngrams(token,4),ngrams(token,5)]
    split_term = tuple(search_term.split(' '))
    search_term_length = len(search_term.split(' '))
    counted_grams = Counter(grams[search_term_length-1])
    combined_term = 'No suggested searches'    
    matching_terms = [element for element in list(counted_grams.items()) if \element[0][:-1] == tuple(split_term)]
    if(len(matching_terms) > 0):
        frequencies = [item[1] for item in matching_terms]
        maximum_frequency = np.max(frequencies)
 highest_frequency_term = [item[0] for item in matching_terms if item[1] == \maximum_frequency][0]
        combined_term = ' '.join(highest_frequency_term)
    return(combined_term)
```

列表 8-2: 一个通过输入一个 n-gram 并返回最可能的以该 n-gram 开头的 n + 1-gram 的搜索建议函数

当我们调用我们的函数时，我们传入一个 n-gram 作为参数，函数将返回一个*n* + 1-gram。我们按如下方式调用它：

```py
file = requests.get('http://www.bradfordtuckfield.com/shakespeare.txt')
file = file=file.text
text = file.replace('\n', '')
print(search_suggestion('life is a', text))
```

你会看到，建议是`life is a tedious`，这是莎士比亚最常用的以`life is a`开头的 4-gram（与其他两个 4-gram 并列）。莎士比亚只在一次使用了这个 4-gram，出现在《辛白林》中，当伊莫金说：“我看到一个人的生命是乏味的。”在《李尔王》中，埃德加对格洛斯特说：“你的生命是一个奇迹”（或者根据不同版本是“你的生命是奇迹”），所以这个 4-gram 也可以作为我们短语的有效补全。

我们可以尝试使用不同的语料库，看看结果有何不同。让我们使用马克·吐温的全集语料库：

```py
file = requests.get('http://www.bradfordtuckfield.com/marktwain.txt')
file = file=file.text
text = file.replace('\n', '')
```

使用这个新的语料库，我们可以再次检查搜索建议：

```py
print(search_suggestion('life is a',text))
```

在这个例子中，补全后的短语是`life is a failure`，这表明了两个文本语料库之间的差异，也许还暗示了莎士比亚和马克·吐温在风格和态度上的不同。你也可以尝试其他搜索词。例如，如果使用马克·吐温的语料库，`I love`会被补全为`you`，而如果使用莎士比亚的语料库，它会被补全为`thee`，这展示了跨越几个世纪和大洋的风格差异，尽管观点可能没有太大不同。尝试使用另一个语料库和其他短语，看看你的短语如何被补全。如果你使用的是另一种语言写成的语料库，使用我们刚才编写的函数，你甚至可以对你不懂的语言进行短语补全。

## 总结

在这一章中，我们讨论了可以用于处理人类语言的算法。我们从一个空格插入算法开始，该算法可以修正扫描错误的文本，接着我们讨论了一个短语补全算法，它可以向输入的短语中添加单词，以匹配文本语料库的内容和风格。我们采取的这些算法方法类似于适用于其他类型语言算法的方法，包括拼写检查器和意图解析器。

在下一章，我们将探讨机器学习，这是一门强大且不断发展的领域，每一个优秀的算法工程师都应该熟悉它。我们将重点介绍一种名为*决策树*的机器学习算法，它是一种简单、灵活、准确且可解释的模型，可以在你踏上算法和人生的旅程中帮助你走得更远。
