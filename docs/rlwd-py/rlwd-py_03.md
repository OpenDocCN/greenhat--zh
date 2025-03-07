## 3

使用自然语言处理总结演讲

![Image](../images/common.jpg)

“四面水域皆是水，却无一滴可饮。”这句来自《古舟子之歌》的名句总结了数字信息的现状。根据国际数据公司（IDC）的预测，到2025年，我们将每年生成 175 万亿千兆字节的数字数据。但大部分数据——多达 95%——将是*非结构化*的，这意味着它并未被组织成有用的数据库。即使现在，癌症的治愈方法可能就在我们手边，却几乎无法触及。

为了让信息更容易被发现和消费，我们需要通过提取和重新包装关键点来减少数据量，形成易于理解的摘要。由于数据量庞大，人工处理几乎不可能。幸运的是，自然语言处理（NLP）可以帮助计算机理解单词和上下文。例如，NLP 应用程序可以总结新闻源、分析法律合同、研究专利、研究金融市场、捕捉企业知识并生成学习指南。

在本章中，你将使用 Python 的自然语言工具包（NLTK）生成历史上最著名演讲之一——马丁·路德·金的《我有一个梦想》的摘要。在理解基本概念后，你将使用一种简化的替代方法，称为 gensim，来总结威廉·H·麦克雷文海军上将的著名演讲《整理床铺》。最后，你将使用词云来制作一个有趣的可视化摘要，展示亚瑟·柯南·道尔的小说《巴斯克维尔的猎犬》中最常用的单词。

### **项目 #3：我有一个梦想……来总结演讲！**

在机器学习和数据挖掘中，有两种总结文本的方法：*提取*和*抽象*。

基于提取的总结方法使用各种加权函数根据感知的重要性对句子进行排序。使用频率更高的词被认为更重要。因此，包含这些词的句子被认为更重要。总体行为就像是用黄色荧光笔手动选择关键词和句子，而不改变文本。尽管结果可能会显得支离破碎，但这种技术擅长提取重要的单词和短语。

抽象依赖于对文档更深层次的理解，以捕捉意图并生成更具人类化的改写。这包括创造全新的句子。与基于提取的方法相比，抽象方法的结果往往更加连贯且语法正确，但也有其代价。抽象算法需要先进且复杂的深度学习方法和精密的语言建模。

在这个项目中，你将对马丁·路德·金（Dr. Martin Luther King Jr.）1963年8月28日在林肯纪念堂发表的“我有一个梦想”演讲使用提取式技术。与林肯100年前的“葛底斯堡演说”一样，这也是在完美的时刻发表的完美演讲。金博士巧妙地运用了重复手法，这使得这篇演讲特别适合使用提取技术，通过关联单词频率与重要性来提取关键信息。

目标

编写一个Python程序，使用自然语言处理（NLP）文本提取功能总结演讲内容。

#### ***策略***

自然语言工具包（NLTK）包含了你需要用来总结马丁·路德·金（Dr. King）演讲的功能。如果你跳过了[第2章](ch02.xhtml)，请查看[第29页](ch02.xhtml#page_29)获取安装说明。

为了总结演讲内容，你需要一个数字化副本。在之前的章节中，你手动从互联网上下载了需要的文件。这一次，你将使用一种更高效的技术——*网络爬取*，它可以让你编程方式从网站中提取并保存大量数据。

一旦你将演讲加载为字符串，你可以使用NLTK来分割并统计每个单词。接着，你将通过将每个句子中的单词计数求和，来“评分”每个句子。你可以根据句子的得分来打印排名最高的句子，基于你希望在总结中包含多少句子。

#### ***网络爬取***

网络爬取是指使用程序下载和处理内容。这是一个非常常见的任务，现成的爬虫程序可以自由获取。你将使用requests库来下载文件和网页，并使用Beautiful Soup（bs4）包来解析HTML。HTML是*超文本标记语言*（Hypertext Markup Language）的缩写，是用于创建网页的标准格式。

要安装这两个模块，请在终端窗口或Windows PowerShell中使用pip（有关使用和安装pip的说明，请参见[第8页](ch01.xhtml#page_8)的[第1章](ch01.xhtml)）。

```py
pip install requests 
pip install beautifulsoup4
```

要检查安装情况，打开终端并按如下方式导入每个模块。如果没有报错，说明安装成功，可以开始使用了！

```py
>>> import requests
>>> 
>>> import bs4
>>>
```

要了解更多关于requests的信息，请访问*[https://pypi.org/project/requests/](https://pypi.org/project/requests/)*。有关Beautiful Soup的更多信息，请查看*[https://www.crummy.com/software/BeautifulSoup/](https://www.crummy.com/software/BeautifulSoup/)*。

#### ***“我有一个梦想”代码***

*dream_summary.py*程序执行以下步骤：

1.  打开包含“我有一个梦想”演讲的网页

1.  将文本加载为字符串

1.  将文本分解为单词和句子

1.  移除没有上下文内容的停用词

1.  计算剩余单词的数量

1.  使用计数来对句子进行排名

1.  显示排名最高的句子

如果你已经下载了书中的文件，请在*Chapter_3*文件夹中找到该程序。否则，访问*[https://nostarch.com/real-world-python/](https://nostarch.com/real-world-python/)*并从书籍的GitHub页面下载。

##### **导入模块并定义main()函数**

[清单 3-1](ch03.xhtml#ch03list1) 导入模块并定义了 main() 函数的第一部分，该部分用于抓取网页并将演讲内容作为字符串赋值给变量。

```py
dream_summary.py, part 1
from collections import Counter
import re
import requests
import bs4
import nltk
from nltk.corpus import stopwords

def main():
 ➊ url = 'http://www.analytictech.com/mb021/mlk.htm'
    page = requests.get(url)
    page.raise_for_status()
 ➋ soup = bs4.BeautifulSoup(page.text, 'html.parser')
    p_elems = [element.text for element in soup.find_all('p')]

    speech = ''.join(p_elems)
```

清单 3-1：导入模块并定义 main() 函数

首先，从 collections 模块导入 Counter，用于帮助你跟踪句子的得分。collections 模块是 Python 标准库的一部分，包含了几种容器数据类型。Counter 是字典的子类，用于计数可哈希对象。元素作为字典的键存储，而它们的计数作为字典的值存储。

接下来，为了在总结内容之前清理演讲内容，导入 re 模块。*re* 代表 *正则表达式*，也称为 *regex*，它们是定义搜索模式的字符序列。这个模块将帮助你通过允许你有选择地删除不需要的部分来清理演讲内容。

用抓取网页和进行自然语言处理的模块完成导入。最后一个模块引入了功能性停用词的列表（例如 *if*、*and*、*but*、*for*），这些词不包含有用信息。在总结之前，你将从演讲中移除这些词。

接下来，定义一个 main() 函数来运行程序。为了从网上抓取演讲内容，提供 url 地址作为字符串 ➊。你可以从想要提取文本的网站复制并粘贴这个地址。

requests 库抽象了在 Python 中发起 HTTP 请求的复杂性。HTTP，即超文本传输协议，是通过超链接在万维网上进行数据通信的基础。使用 requests.get() 方法来获取 url，并将输出赋值给 page 变量，该变量引用了网页为请求返回的 Response 对象。该对象的 text 属性包含了网页内容，包括演讲内容，以字符串形式呈现。

为了检查下载是否成功，调用 Response 对象的 raise_for_status() 方法。如果一切正常，这个方法什么也不做；否则，它会引发异常并终止程序。

此时，数据是 HTML 格式，如下所示：

```py
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML//EN">
<html>

<head>
<meta http-equiv="Content-Type"
content="text/html; charset=iso-8859-1">
<meta name="GENERATOR" content="Microsoft FrontPage 4.0">
<title>Martin Luther King Jr.'s 1962 Speech</title>
</head>
--snip--
<p>I am happy to join with you today in what will go down in
history as the greatest demonstration for freedom in the history
of our nation. </p>
--snip--
```

如你所见，HTML 中有许多 *标签*，例如 <head> 和 <p>，它们告诉浏览器如何格式化网页。开始标签和结束标签之间的文本称为 *元素*。例如，“马丁·路德·金 Jr. 的 1962 演讲”是一个标题元素，它被夹在开始标签 <title> 和结束标签 </title> 之间。段落使用 <p> 和 </p> 标签进行格式化。

因为这些标签不是原始文本的一部分，所以在进行任何自然语言处理之前应该删除它们。为了移除这些标签，调用 bs4.BeautifulSoup() 方法，并将包含 HTML 的字符串传递给它 ➋。请注意，我已经明确指定了 html.parser。即使没有这个指定，程序仍然可以运行，但会在终端中发出警告。

soup变量现在引用一个BeautifulSoup对象，这意味着你可以使用该对象的find_all()方法来定位HTML文档中的演讲内容。在此案例中，通过列表推导和find_all()来获取仅包含段落元素的列表，以查找段落标签（<p>）之间的文本。

最后，将演讲内容转化为连续的字符串。使用join()方法将p_elems列表转化为字符串。设置“连接符”字符为空格，用''表示。

请注意，在Python中，通常有不止一种方式可以完成任务。列表中的最后两行也可以按以下方式编写：

```py
    p_elems = soup.select('p')
    speech = ''.join(p_elems)
```

select()方法总体上比find_all()方法功能更有限，但在这种情况下，它的作用与find_all()相同，而且更简洁。在前面的代码中，select()方法查找<p>标签，结果在与演讲内容字符串连接时转换为文本。

##### **完成main()函数**

接下来，你将准备演讲内容，修正拼写错误并去除标点符号、特殊字符和空格。然后，你将调用三个函数，分别去除停用词、计算词频并根据词频对句子进行评分。最后，你将对句子进行排序，并在命令行显示得分最高的句子。

[列表 3-2](ch03.xhtml#ch03list2)完成了执行这些任务的main()函数定义。

```py
dream_summary.py, part 2
   speech = speech.replace(')mowing', 'knowing')
   speech = re.sub('\s+', ' ', speech) 
   speech_edit = re.sub('[^a-zA-Z]', ' ', speech)
   speech_edit = re.sub('\s+', ' ', speech_edit)

➊ while True:
       max_words = input("Enter max words per sentence for summary: ")
       num_sents = input("Enter number of sentences for summary: ")
       if max_words.isdigit() and num_sents.isdigit():
           break
       else:
           print("\nInput must be in whole numbers.\n")

   speech_edit_no_stop = remove_stop_words(speech_edit)
   word_freq = get_word_freq(speech_edit_no_stop)
   sent_scores = score_sentences(speech, word_freq, max_words)

➋ counts = Counter(sent_scores)
   summary = counts.most_common(int(num_sents))
   print("\nSUMMARY:")
   for i in summary:
       print(i[0])
```

列表 3-2：完成main()函数

原始文档包含一个拼写错误（*mowing* 应为 *knowing*），因此首先使用string.replace()方法修正这个错误。接着，继续使用正则表达式清理演讲内容。许多非专业程序员对这个模块的复杂语法感到厌烦，但它是一个强大且有用的工具，每个人都应该了解基本的正则表达式语法。

使用re.sub()函数去除多余的空格，该函数将子字符串替换为新的字符。使用简写字符类代码\s+来识别连续的空白符并将其替换为一个空格，表示为' '。最后，将字符串（speech）传递给re.sub()方法。

接下来，使用[^a-zA-Z]模式删除任何*不是*字母的字符。开头的脱字符指示正则表达式“匹配任何不在括号中的字符”。因此，数字、标点符号等将被空格替换。

删除标点符号等字符会留下额外的空格。为了去除这些空格，再次调用re.sub()方法。

接下来，要求用户输入要包括在摘要中的句子数量以及每个句子的最大字数。使用while循环和Python内置的isdigit()函数确保用户输入一个整数➊。

**注意**

*根据美国新闻研究所的研究，理解力最佳的句子长度为不超过15个单词。同样，《牛津简明英语指南》建议，整篇文档中的句子平均长度应为15到20个单词。*

继续清理文本，调用remove_stop_words()函数。然后调用get_word_freq()和score_sentences()函数，分别计算剩余单词的频率并为句子打分。你将在完成main()函数后定义这些函数。

要对句子进行排名，调用集合模块的Counter()方法 ➋。将sent_scores变量传入该方法。

要生成摘要，使用Counter对象的most_common()方法。将用户输入的num_sents变量传递给该方法。结果摘要变量将包含一个元组列表。每个元组中的第一个元素是句子，第二个元素是其排名。

```py
[('From every mountainside, let freedom ring.', 4.625), --snip-- ]
```

为了提高可读性，将摘要中的每个句子打印在单独的一行上。

##### **去除停用词**

记住在[第2章](ch02.xhtml)中提到的，停用词是像*if*、*but*、*for*和*so*这样的短小功能词。由于它们不包含重要的上下文信息，因此不应使用它们来排名句子。

[示例 3-3](ch03.xhtml#ch03list3)定义了一个名为remove_stop_words()的函数，用来从语音中移除停用词。

```py
dream_summary.py, part 3
def remove_stop_words(speech_edit):
    """Remove stop words from string and return string."""
    stop_words = set(stopwords.words('english'))
    speech_edit_no_stop = ''
    for word in nltk.word_tokenize(speech_edit):
        if word.lower() not in stop_words:
            speech_edit_no_stop += word + ' '
    return speech_edit_no_stop
```

示例 3-3：定义一个函数来移除语音中的停用词

定义函数时，接收speech_edit作为参数，该参数是编辑后的语音字符串。然后，创建一个包含NLTK中的英语停用词的集合。使用集合而不是列表，因为集合的查找速度更快。

为了保存去除停用词后的编辑语音，分配一个空字符串给该变量。speech_edit变量目前是一个字符串，其中每个元素都是一个字母。

为了处理单词，调用NLTK的word_tokenize()方法。注意，你可以在循环遍历单词时执行此操作。将每个单词转换为小写，并检查它是否在stop_words集合中。如果它不是停用词，就将其与一个空格一起拼接到新字符串中。函数返回该字符串并结束。

在这个程序中，字母的大小写处理非常重要。你希望摘要能同时打印出大写和小写字母，但在进行自然语言处理时，必须将所有字母转换为小写，以避免错误计数。为了理解原因，请看下面这段代码示例，它统计了一个包含大小写混合的字符串（s）中的单词：

```py
>>> import nltk
>>> s = 'one One one'
>>> fd = nltk.FreqDist(nltk.word_tokenize(s))
>>> fd
FreqDist({'one': 2, 'One': 1})
>>> fd_lower = nltk.FreqDist(nltk.word_tokenize(s.lower()))
>>> fd_lower
FreqDist({'one': 3})
```

如果不将单词转换为小写，*one* 和 *One* 会被认为是不同的元素。为了计数，每个*one*的实例，不论其大小写，应该被视为相同的单词。否则，*one*对文档的贡献将被稀释。

##### **计算单词出现频率**

为了统计每个单词在语音中的出现次数，你需要创建一个名为get_word_freq()的函数，该函数返回一个字典，字典的键是单词，值是对应的出现次数。[示例 3-4](ch03.xhtml#ch03list4)定义了这个函数。

```py
dream_summary.py, part 4 
def get_word_freq(speech_edit_no_stop):
    """Return a dictionary of word frequency in a string."""        
    word_freq = nltk.FreqDist(nltk.word_tokenize(speech_edit_no_stop.lower()))
    return word_freq
```

示例 3-4：定义一个函数来计算语音中的单词频率

get_word_freq() 函数以编辑过的没有停用词的语音字符串作为参数。NLTK 的 FreqDist 类类似于一个字典，单词作为键，计数作为值。作为过程的一部分，将输入字符串转换为小写，并将其分割成单词。结束函数时，返回 word_freq 字典。

##### **评分句子**

[列表 3-5](ch03.xhtml#ch03list5) 定义了一个函数，该函数根据句子中单词的频率分布对句子进行评分。它返回一个字典，每个句子作为键，其得分作为值。

```py
dream_summary.py, part 5
def score_sentences(speech, word_freq, max_words):
    """Return dictionary of sentence scores based on word frequency."""
    sent_scores = dict()
    sentences = nltk.sent_tokenize(speech)
 ➊ for sent in sentences:
        sent_scores[sent] = 0
        words = nltk.word_tokenize(sent.lower())
        sent_word_count = len(words)
     ➋ if sent_word_count <= int(max_words):
            for word in words:
                if word in word_freq.keys():
                    sent_scores[sent] += word_freq[word]
       ➌ sent_scores[sent] = sent_scores[sent] / sent_word_count
  return sent_scores

if __name__ == '__main__':
    main()
```

列表 3-5：定义一个根据单词频率为句子评分的函数

定义一个名为 score_sentences() 的函数，参数包括原始的语音字符串、word_freq 对象和用户输入的 max_words 变量。你希望摘要包含停用词和大写单词——因此使用语音字符串。

创建一个空字典，命名为 sent_scores，用于保存每个句子的得分。接下来，将语音字符串分割成句子。

现在，开始遍历句子 ➊。首先更新 sent_scores 字典，将句子作为键，并将其初始值（计数）设为 0。

要计算单词频率，首先需要将句子分割成单词。确保使用小写字母，以便与 word_freq 字典兼容。

在对每个句子的单词数进行求和以创建得分时，你需要小心，以免将结果偏向较长的句子。毕竟，较长的句子更可能包含更多重要单词。为了避免排除短但重要的句子，你需要通过将其除以句子的 *长度* 来对每个计数进行 *标准化*。将长度存储在一个名为 sent_word_count 的变量中。

接下来，使用一个条件语句，限制句子的长度为用户输入的最大值 ➋。如果句子通过测试，开始遍历其中的单词。如果一个单词出现在 word_freq 字典中，就将它添加到存储在 sent_scores 中的计数值中。

在每次循环结束时，将当前句子的得分除以句子中的单词数 ➌。这会将得分标准化，避免长句子占有不公平的优势。

通过返回 sent_scores 字典来结束函数。然后，在全局空间中，添加运行程序的代码，无论是作为模块还是独立模式。

##### **运行程序**

使用 *dream_summary.py* 程序，设置最大句子长度为 14 个单词。如前所述，良好、易读的句子通常包含 14 个单词或更少。然后将摘要截断为 15 个句子，约为演讲的三分之一。你应该得到以下结果。请注意，句子的顺序不一定与原文相同。

```py
Enter max words per sentence for summary: 14
Enter number of sentences for summary: 15

SUMMARY:
From every mountainside, let freedom ring.
Let freedom ring from Lookout Mountain in Tennessee!
Let freedom ring from every hill and molehill in Mississippi.
Let freedom ring from the curvaceous slopes of California!
Let freedom ring from the snow capped Rockies of Colorado!
But one hundred years later the Negro is still not free.
From the mighty mountains of New York, let freedom ring.
From the prodigious hilltops of New Hampshire, let freedom ring.
And I say to you today my friends, let freedom ring.
I have a dream today.
It is a dream deeply rooted in the American dream.
Free at last!
Thank God almighty, we're free at last!"
We must not allow our creative protest to degenerate into physical violence.
This is the faith that I go back to the mount with.
```

摘要不仅捕捉了演讲的标题，还捕捉了主要内容。

但如果你再次运行程序，将每个句子的字数限制为 10，很多句子明显太长。由于整个演讲中只有 7 个句子包含 10 个或更少的字，因此程序无法遵循输入要求。它会默认从演讲的开头开始打印，直到句子数至少达到 num_sents 变量中指定的数目。

现在，重新运行程序并尝试将字数限制设置为 1,000。

```py
Enter max words per sentence for summary: 1000
Enter number of sentences for summary: 15

SUMMARY:
From every mountainside, let freedom ring.
Let freedom ring from Lookout Mountain in Tennessee!
Let freedom ring from every hill and molehill in Mississippi.
Let freedom ring from the curvaceous slopes of California!
Let freedom ring from the snow capped Rockies of Colorado!
But one hundred years later the Negro is still not free.
From the mighty mountains of New York, let freedom ring.
From the prodigious hilltops of New Hampshire, let freedom ring.
And I say to you today my friends, let freedom ring.
I have a dream today.
But not only there; let freedom ring from the Stone Mountain of Georgia!
It is a dream deeply rooted in the American dream.
With this faith we will be able to work together, pray together; to struggle
together, to go to jail together, to stand up for freedom forever, knowing
that we will be free one day.
Free at last!
One hundred years later the life of the Negro is still sadly crippled by the
manacles of segregation and the chains of discrimination.
```

尽管较长的句子并未主导摘要，但仍有一些被遗漏，使得这个摘要比之前的版本少了些诗意。较低的字数限制迫使之前的版本更多依赖于短语，这些短语起到了类似副歌的作用。

### **项目 #4：使用 gensim 摘要演讲**

在一集获得艾美奖的 *辛普森一家*（The Simpsons）中，霍默竞选卫生委员会委员，使用的竞选口号是：“难道不能让别人做吗？”这也正是许多 Python 应用程序的情况：经常在你需要编写脚本时，发现别人已经做过了！一个例子就是 gensim，这是一个用于自然语言处理的开源库，使用统计机器学习。

*gensim* 这个词代表“生成相似的（generate similar）”。它使用了一种基于图形的排名算法，称为 TextRank。这种算法受 PageRank 的启发，PageRank 是由拉里·佩奇（Larry Page）发明的，用于在 Google 搜索中对网页进行排名。使用 PageRank 时，网站的重要性是由指向该网站的其他页面的数量来决定的。要将这种方法应用于文本处理，算法会衡量每个句子与其他句子的相似度。与其他句子最相似的句子被认为是最重要的。

在这个项目中，你将使用 gensim 来总结威廉·H·麦克雷文（William H. McRaven）海军上将于 2014 年在德克萨斯大学奥斯汀分校所做的毕业演讲《整理床铺》。这场鼓舞人心的 20 分钟演讲在 YouTube 上已经观看超过 1,000 万次，并于 2017 年激发了一本 *纽约时报* 的畅销书。

目标

编写一个使用 gensim 模块来总结演讲的 Python 程序。

#### ***安装 gensim***

gensim 模块可以在所有主要操作系统上运行，但依赖于 NumPy 和 SciPy。如果你没有安装它们，请返回到 [第 1 章](ch01.xhtml)，并按照“安装 Python 库”中的说明操作，[第 6 页](ch01.xhtml#page_6)。

在 Windows 上安装 gensim，使用 pip install -U gensim。在终端中安装，使用 pip install --upgrade gensim。对于 conda 环境，使用 conda install -c conda-forge gensim。有关 gensim 的更多信息，请访问 *[https://radimrehurek.com/gensim/](https://radimrehurek.com/gensim/)*。

#### ***整理床铺代码***

在项目 3 中，你通过 *dream_summary.py* 程序学习了文本提取的基本概念。既然你已经了解了一些细节，现在可以使用 gensim 作为 *dream_summary.py* 的简化替代方案。命名这个新程序为 *bed_summary.py*，或者从本书网站下载它。

##### **导入模块、抓取网页并准备演讲字符串**

[清单 3-6](ch03.xhtml#ch03list6)重复了在*dream_summary.py*中用于准备演讲的代码。要重新查看详细的代码解释，请参阅[第 54 页](ch03.xhtml#page_54)。

```py
bed_summary.py, part 1
   import requests
   import bs4
   from nltk.tokenize import sent_tokenize
➊ from gensim.summarization import summarize

➋ url = 'https://jamesclear.com/great-speeches/make-your-bed-by-admiral
          -william-h-mcraven'
   page = requests.get(url)
   page.raise_for_status()
   soup = bs4.BeautifulSoup(page.text, 'html.parser')
   p_elems = [element.text for element in soup.find_all('p')]

   speech = ''.join(p_elems)
```

清单 3-6：导入模块并将演讲加载为字符串

您将在从网页抓取的原始演讲文本上测试 gensim，因此不需要用于清理文本的模块。gensim 模块也会在内部进行任何计数，因此您不需要 Counter，但您需要 gensim 的 summarize() 函数来总结文本 ➊。唯一的其他变化是 url 地址 ➋。

##### **总结演讲内容**

[清单 3-7](ch03.xhtml#ch03list7)通过总结演讲并打印结果完成了程序。

```py
bed_summary.py, part 2
print("\nSummary of Make Your Bed speech:")
summary = summarize(speech, word_count=225)
sentences = sent_tokenize(summary)
sents = set(sentences)
print(' '.join(sents))
```

清单 3-7：运行 gensim，删除重复行并打印摘要

首先打印摘要的标题。然后，调用 gensim 的 summarize() 函数来总结 225 个字的演讲。根据平均每句15个字的假设，这个字数将产生约15个句子的摘要。除了字数，您还可以传递一个比例给 summarize()，例如 ratio=0.01。这将生成一个长度为完整文档1%的摘要。

理想情况下，您可以一步完成演讲的总结和打印摘要。

```py
print(summarize(speech, word_count=225))
```

不幸的是，gensim 有时会在摘要中重复句子，这在这里发生了。

```py
Summary of Make Your Bed speech:
Basic SEAL training is six months of long torturous runs in the soft sand,
midnight swims in the cold water off San Diego, obstacle courses, unending
calisthenics, days without sleep and always being cold, wet and miserable.
Basic SEAL training is six months of long torturous runs in the soft sand,
midnight swims in the cold water off San Diego, obstacle courses, unending
calisthenics, days without sleep and always being cold, wet and miserable.
--snip--
```

为了避免重复文本，您首先需要使用 NLTK 的 sent_tokenize() 函数将摘要变量中的句子分解。然后从这些句子中创建一个集合，这将删除重复项。最后打印结果。

因为集合是无序的，所以如果您多次运行程序，句子的排列可能会发生变化。

```py
Summary of Make Your Bed speech:
If you can't do the little things right, you will never do the big things
right.And, if by chance you have a miserable day, you will come home to a
bed that is made — that you made — and a made bed gives you encouragement
that tomorrow will be better.If you want to change the world, start off
by making your bed.During SEAL training the students are broken down into
boat crews. It's just the way life is sometimes.If you want to change the
world get over being a sugar cookie and keep moving forward.Every day during
training you were challenged with multiple physical events — long runs, long
swims, obstacle courses, hours of calisthenics — something designed to test
your mettle. Basic SEAL training is six months of long torturous runs in the
soft sand, midnight swims in the cold water off San Diego, obstacle courses,
unending calisthenics, days without sleep and always being cold, wet and
miserable.
>>>
======= RESTART: C:\Python372\sequel\wordcloud\bed_summary.py =======

Summary of Make Your Bed speech:
It's just the way life is sometimes.If you want to change the world get over
being a sugar cookie and keep moving forward.Every day during training you
were challenged with multiple physical events — long runs, long swims,
obstacle courses, hours of calisthenics — something designed to test your
mettle. If you can't do the little things right, you will never do the big
things right.And, if by chance you have a miserable day, you will come home to
a bed that is made — that you made — and a made bed gives you encouragement
that tomorrow will be better.If you want to change the world, start off by
making your bed.During SEAL training the students are broken down into boat
crews. Basic SEAL training is six months of long torturous runs in the soft
sand, midnight swims in the cold water off San Diego, obstacle courses,
unending calisthenics, days without sleep and always being cold, wet and
miserable.
```

如果你花时间阅读全文，你可能会得出结论，gensim生成了一个公平的摘要。尽管这两个结果不同，但两者都提取了演讲的要点，包括提到如何整理床铺。考虑到文档的大小，我觉得这很令人印象深刻。

接下来，我们将看看使用关键词和词云总结文本的另一种方法。

### **项目 #5：使用词云总结文本**

*词云*是用于显示网站上关键词元数据的文本数据的视觉表示。在词云中，字体大小或颜色显示每个标签或单词的重要性。

词云对于突出文档中的关键词非常有用。例如，为每位美国总统的国情咨文生成词云可以快速概述该年国家面临的问题。在比尔·克林顿的首年，重点放在了像医疗保健、就业和税收等和平时期的关切事项上（[图 3-1](ch03.xhtml#ch03fig1)）。

![图像](../images/fig03_01.jpg)

图 3-1：由比尔·克林顿1993年国情咨文制作的词云

不到10年后，乔治·W·布什的词云显示出对安全问题的关注（[图 3-2](ch03.xhtml#ch03fig2)）。

![图像](../images/fig03_02.jpg)

图 3-2：由乔治·W·布什在 2002 年的国情咨文中演讲的词云。

词云的另一个用途是从客户反馈中提取关键词。如果像*差*、*慢*和*贵*这样的词占主导地位，那么你就有问题了！作家还可以使用词云来比较书中的章节或剧本中的场景。如果作者在动作场景和浪漫插曲中使用了非常相似的语言，就需要进行一些编辑。如果你是文案写手，词云可以帮助你检查关键词密度，以优化搜索引擎优化（SEO）。

生成词云的方法有很多，包括一些免费的在线网站，如 *[https://www.wordclouds.com/](https://www.wordclouds.com/)* 和 *[https://www.jasondavies.com/wordcloud/](https://www.jasondavies.com/wordcloud/)*。但是，如果你想完全自定义你的词云，或者将生成器嵌入到另一个程序中，就需要自己动手。在本项目中，你将使用词云为学校戏剧制作一张宣传单，基于《福尔摩斯探案集》中的故事 *《巴斯克维尔的猎犬》*。

你将不再使用[图 3-1](ch03.xhtml#ch03fig1)和[3-2](ch03.xhtml#ch03fig2)中显示的基本矩形，而是将单词嵌入到福尔摩斯头部的轮廓中（见[图 3-3](ch03.xhtml#ch03fig3)）。

![Image](../images/fig03_03.jpg)

图 3-3：福尔摩斯的剪影

这将使得显示效果更加易于识别和引人注目。

目标

使用 wordcloud 模块为小说生成一个定形的词云。

#### ***词云和 PIL 模块***

你将使用名为 wordcloud 的模块来生成词云。你可以通过 pip 安装它。

```py
pip install wordcloud
```

或者，如果你使用的是 Anaconda，可以使用以下命令：

```py
conda install -c conda-forge wordcloud
```

你可以在这里找到 wordcloud 的网页：*[http://amueller.github.io/word_cloud/](http://amueller.github.io/word_cloud/)*。

你还需要使用 Python Imaging Library (PIL) 来处理图像。可以再次使用 pip 安装它。

```py
pip install pillow
```

或者，对于 Anaconda，请使用此命令：

```py
conda install -c anaconda pillow
```

如果你在想，pillow 是 PIL 的继任项目，PIL 在 2011 年已停止更新。要了解更多信息，请访问 *[https://pillow.readthedocs.io/en/stable/](https://pillow.readthedocs.io/en/stable/)*。

#### ***词云代码***

要生成定形的词云，你需要一个图像文件和一个文本文件。图 3-3 中显示的图像来自 Getty Images 的 iStock（*[https://www.istockphoto.com/vector/detective-hat-gm698950970-129478957/](https://www.istockphoto.com/vector/detective-hat-gm698950970-129478957/)*）。这是大约 500×600 像素的“低”分辨率图像。

本书的下载文件中提供了一个相似但不受版权保护的图像（*holmes.png*）。你可以在 *Chapter_3* 文件夹中找到文本文件（*hound.txt*）、图像文件（*holmes.png*）和代码（*wc_hound.py*）。

##### **导入模块、文本文件、图像文件和停用词**

[示例 3-8](ch03.xhtml#ch03list8) 导入模块，加载小说，加载福尔摩斯的轮廓图像，并创建一组你想从词云中排除的停用词。

```py
wc_hound.py, part 1
   import numpy as np
   from PIL import Image
   import matplotlib.pyplot as plt
   from wordcloud import WordCloud, STOPWORDS

   # Load a text file as a string.
➊ with open('hound.txt') as infile:
      text = infile.read()

   # Load an image as a NumPy array.
   mask = np.array(Image.open('holmes.png'))

   # Get stop words as a set and add extra words.
   stopwords = STOPWORDS
➋ stopwords.update(['us', 'one', 'will', 'said', 'now', 'well', 'man', 'may',
                     'little', 'say', 'must', 'way', 'long', 'yet', 'mean',
                     'put', 'seem', 'asked', 'made', 'half', 'much',
                     'certainly', 'might', 'came'])
```

示例 3-8：导入模块并加载文本、图像和停用词

首先导入 NumPy 和 PIL。PIL 将打开图像，NumPy 将其转换为蒙版。你在[第 1 章](ch01.xhtml)中开始使用 NumPy；如果你跳过了该部分，请参阅[第 1 章](ch01.xhtml#page_6)的“安装 Python 库”部分。请注意，Pillow 模块继续使用 PIL 的缩写，以保证向后兼容。

你将需要 matplotlib，它在[第 1 章](ch01.xhtml)的“安装 Python 库”部分中已经下载，用于显示词云。wordcloud 模块自带一套停用词列表，因此需要同时导入 STOPWORDS 和词云功能。

接下来，加载小说的文本文件并将其存储在名为 text ➊ 的变量中。如[第 2 章](ch02.xhtml)中[示例 2-2](ch02.xhtml#ch02list2)的讨论所述，在加载文本时，你可能会遇到 UnicodeDecodeError 错误。

```py
UnicodeDecodeError: 'ascii' codec can't decode byte 0x93 in position 365:
ordinal not in range(128)
```

在这种情况下，尝试通过添加编码和错误参数来修改 open() 函数。

```py
    with open('hound.txt', encoding='utf-8', errors='ignore') as infile:
```

加载文本后，使用 PIL 的 Image.open() 方法打开福尔摩斯的图像，并使用 NumPy 将其转换为数组。如果你使用的是 iStock 上的福尔摩斯图像，请适当更改图像文件名。

将从 wordcloud 导入的 STOPWORDS 集合分配给 stopwords 变量。然后，使用你想排除的额外单词更新该集合 ➋。这些将是像 *said* 和 *now* 这样的词，它们在词云中占主导地位，但并未增加有用内容。确定哪些是这样的词是一个迭代过程。你生成词云，移除你认为不贡献的词，并重复此过程。你可以注释掉这行代码以查看效果。

**注意**

*要更新像 STOPWORDS 这样的容器，你需要知道它是列表、字典、集合等。Python 的内建函数 type() 会返回传入对象的类类型。在这种情况下，print(type(STOPWORDS)) 的输出为 <class 'set'>。*

##### **生成词云**

[示例 3-9](ch03.xhtml#ch03list9) 生成词云并使用轮廓图像作为 *蒙版*，即用来隐藏另一图像部分的图像。wordcloud 使用的处理过程足够复杂，可以将词语适应于蒙版中，而不仅仅是将它们截断在边缘。此外，有多个参数可以更改蒙版中单词的外观。

```py
wc_hound.py, part 2
wc = WordCloud(max_words=500,
               relative_scaling=0.5,
               mask=mask,
               background_color='white',
               stopwords=stopwords,
               margin=2,
               random_state=7,
               contour_width=2,
               contour_color='brown',
               colormap='copper').generate(text)

colors = wc.to_array()
```

示例 3-9：生成词云

给变量命名为 wc 并调用 WordCloud()。有很多参数，因此我将每个参数单独放在一行以便清晰。有关所有可用参数的列表和描述，请访问 *[https://amueller.github.io/word_cloud/generated/wordcloud.WordCloud.html](https://amueller.github.io/word_cloud/generated/wordcloud.WordCloud.html)*。

首先传入你想要使用的最大单词数。你设置的数字将显示文本中最常见的 *n* 个单词。选择显示更多单词将更容易定义掩码的边缘并使其可识别。不幸的是，设置的最大数字过高也会导致许多微小的、难以辨认的单词。对于这个项目，从 500 开始。

接下来，为了控制字体大小和每个单词的相对重要性，将 `relative_scaling` 参数设置为 0.5。例如，值为 0 时，优先根据单词的排名来确定字体大小，而值为 1 时，出现频率是两倍的单词将显示为两倍大。0 到 0.5 之间的值通常能在排名和频率之间取得最佳平衡。

引用 mask 变量并将其背景颜色设置为白色。未指定颜色时，默认为黑色。然后引用你在前一个列表中编辑的 stopwords 集。

`margin` 参数将控制显示单词的间距。使用 0 会导致单词紧密排列。使用 2 会允许一些空白填充。

要将单词放置在词云周围，使用随机数生成器并将 `random_state` 设置为 7。这个值并没有特别的含义；我只是觉得它产生了一个吸引人的单词排列。

`random_state` 参数固定种子值，以便结果可重复，前提是没有更改其他参数。这意味着单词的排列将始终相同。只接受整数。

现在，将 contour_width 设置为 2。任何大于零的值都会在掩码周围创建一个轮廓。在这种情况下，由于图像的分辨率，轮廓是波浪形的（[图 3-4](ch03.xhtml#ch03fig4)）。

使用 contour_color 参数将轮廓颜色设置为棕色。通过将 colormap 设置为 copper，继续使用棕色调的色盘。在 matplotlib 中，*colormap* 是一个字典，它将数字映射到颜色。copper 色盘生成的文本颜色范围从浅肤色到黑色。你可以在 *[https://matplotlib.org/gallery/color/colormap_reference.html](https://matplotlib.org/gallery/color/colormap_reference.html)* 查看它的色谱以及许多其他颜色选项。如果你没有指定 colormap，程序将使用默认颜色。

使用点符号调用 `generate()` 方法来构建词云。将文本字符串作为参数传递给它。通过命名一个 colors 变量并在 wc 对象上调用 `to_array()` 方法来结束这个列表。该方法将词云图像转换为 NumPy 数组，以便与 matplotlib 一起使用。

![Image](../images/fig03_04.jpg)

图 3-4：带有轮廓的掩码词云示例（左）与不带轮廓的示例（右）

##### **绘制词云**

[列表 3-10](ch03.xhtml#ch03list10) 为词云添加标题，并使用 matplotlib 显示它。它还将词云图像保存为文件。

```py
wc_hound.py, part 3
plt.figure()
plt.title("Chamberlain Hunt Academy Senior Class Presents:\n",
          fontsize=15, color='brown')
plt.text(-10, 0, "The Hound of the Baskervilles",
         fontsize=20, fontweight='bold', color='brown')
plt.suptitle("7:00 pm May 10-12 McComb Auditorium",
             x=0.52, y=0.095, fontsize=15, color='brown')
plt.imshow(colors, interpolation="bilinear")
plt.axis('off')
plt.show()
##plt.savefig('hound_wordcloud.png')
```

列表 3-10：绘制和保存词云

从初始化matplotlib图形开始。然后调用title()方法，传递学校的名称、字体大小和颜色。

您希望剧本的名称比其他标题更大更粗。由于无法在matplotlib中改变字符串的文本样式，请使用text()方法定义一个新标题。传递给它(*x*, *y*)坐标（基于图形轴）、文本字符串和文本样式细节。通过反复试验坐标来优化文本位置。如果您使用的是Holmes的iStock图片，您可能需要将*x*坐标从-10更改为其他值，以达到最佳的平衡效果，尤其是在不对称的轮廓下。

完成标题时，将剧本的时间和地点放在图形的底部。您可以再次使用text()方法，但我们来看看另一种方法——pyplot的suptitle()方法。这个名字代表“超级标题”。传递给它文本、(*x*, *y*)图形坐标和样式细节。

要显示词云，请调用imshow()——用于图像显示——并传递之前创建的颜色数组。指定bilinear进行颜色插值。

通过调用show()关闭图形坐标轴并显示词云。如果您想保存图形，可以取消注释savefig()方法。请注意，matplotlib可以读取文件名中的扩展名，并以正确的格式保存图形。按原样，保存命令将在您手动关闭图形后才会执行。

#### ***微调词云***

[列表 3-10](ch03.xhtml#ch03list10)将生成[图 3-5](ch03.xhtml#ch03fig5)中的词云。由于算法是随机的，您可能会得到不同的单词排列。

![Image](../images/fig03_05.jpg)

图 3-5：由wc_hound.py代码生成的传单

您可以通过在初始化图形时添加一个参数来更改显示的大小。示例如下：plt.figure(figsize=(50, 60))。

还有许多其他方法可以更改结果。例如，将margin参数设置为10会生成一个更稀疏的词云（[图 3-6](ch03.xhtml#ch03fig6)）。

![Image](../images/fig03_06.jpg)

图 3-6：使用margin=10生成的词云

更改`random_state`参数也会重新排列遮罩中的单词（[图 3-7](ch03.xhtml#ch03fig7)）。

![Image](../images/fig03_07.jpg)

图 3-7：使用margin=10和random_state=6生成的词云

调整max_words和relative_scaling参数也会改变词云的外观。根据您的细致程度，这一切可能是福是祸！

### **总结**

在本章中，您使用了基于提取的摘要技术，生成了马丁·路德·金“我有一个梦想”演讲的概要。然后，您使用了一个免费的现成模块gensim，以更少的代码总结了麦克雷文海军上将的“整理床铺”演讲。最后，您使用wordcloud模块创建了一个有趣的单词设计。

### **进一步阅读**

*用Python自动化枯燥的事情：完全初学者的实用编程*（No Starch Press，2015年），由阿尔·斯威加特编写，涵盖了第[7章](ch07.xhtml)中的正则表达式和第[11章](ch11.xhtml)中的网页抓取，包括使用requests和Beautiful Soup模块。

*整理床铺：那些能改变你生活的小事*……*也许还能改变世界*，第二版（大中央出版社，2017年），威廉·H·麦克雷文著，是一本基于这位海军上将于德克萨斯大学的毕业典礼演讲的自助书籍。你可以在 *[https://www.youtube.com/](https://www.youtube.com/)* 上找到这篇演讲的实际视频。

### **挑战项目：游戏之夜**

使用wordcloud为游戏之夜发明一个新游戏。总结维基百科或IMDb的电影简介，看看你的朋友们能否猜出电影名称。[图 3-8](ch03.xhtml#ch03fig8)展示了一些示例。

![图片](../images/fig03_08.jpg)

图 3-8：2010年发布的两部电影的词云：如何训练你的龙和波斯王子

如果你不喜欢电影，可以选择其他内容。替代品包括著名小说、*星际迷航*剧集以及歌词（见[图 3-9](ch03.xhtml#ch03fig9)）。

![图片](../images/fig03_09.jpg)

图 3-9：从歌曲歌词中制作的词云（唐纳德·费根的“I.G.Y.”）

桌面游戏近年来有了复兴，所以你可以跟随这一趋势，将词云打印在卡纸上。或者，你可以保持数字化，给玩家提供每个词云的多项选择答案。游戏应跟踪正确答案的数量。

### **挑战项目：总结摘要**

使用项目3中的程序对之前总结过的文本进行测试，比如维基百科页面。仅五个句子就能提供一个关于gensim的良好概述。

```py
Enter max words per sentence for summary: 30
Enter number of sentences for summary: 5

SUMMARY:
Gensim is implemented in Python and Cython.
Gensim is an open-source library for unsupervised topic modeling and natural
language processing, using modern statistical machine learning.
[12] Gensim is commercially supported by the company rare-technologies.com,
who also provide student mentorships and academic thesis projects for Gensim
via their Student Incubator programme.
The software has been covered in several new articles, podcasts and
interviews.
Gensim is designed to handle large text collections using data streaming and
incremental online algorithms, which differentiates it from most other machine
learning software packages that target only in-memory processing.
```

接下来，尝试在那些没人愿意阅读的无聊服务协议上使用项目4中的gensim版本。一个微软协议的示例可在 *[https://www.microsoft.com/en-us/servicesagreement/default.aspx](https://www.microsoft.com/en-us/servicesagreement/default.aspx)* 获取。当然，要评估结果，你必须阅读完整的协议，而几乎没有人愿意这么做！享受这个进退两难的困境吧！

### **挑战项目：总结一部小说**

编写一个程序，通过章节总结《巴斯克维尔的猎犬》。每个章节的总结保持简短，大约75个单词。

若要获取带章节标题的小说副本，可以通过以下代码从古腾堡计划网站抓取文本：url = 'http://www.gutenberg.org/files/2852/2852-h/2852-h.htm'。

若要分解章节元素，而非段落元素，使用此代码：

```py
chapter_elems = soup.select('div[class="chapter"]')
chapters = chapter_elems[2:]
```

你还需要从每个章节中选择段落元素（p_elems），使用与 *dream_summary.py* 中相同的方法。

以下代码片段展示了使用每章75个单词的结果：

```py
--snip--

Chapter 3:
"Besides, besides—" "Why do you hesitate?” "There is a realm in which the most
acute and most experienced of detectives is helpless." "You mean that the
thing is supernatural?" "I did not positively say so." "No, but you evidently
think it." "Since the tragedy, Mr. Holmes, there have come to my ears several
incidents which are hard to reconcile with the settled order of Nature." "For
example?" "I find that before the terrible event occurred several people had
seen a creature upon the moor which corresponds with this Baskerville demon,
and which could not possibly be any animal known to science.

--snip--

Chapter 6:
"Bear in mind, Sir Henry, one of the phrases in that queer old legend which
Dr. Mortimer has read to us, and avoid the moor in those hours of darkness
when the powers of evil are exalted." I looked back at the platform when we
had left it far behind and saw the tall, austere figure of Holmes standing
motionless and gazing after us.

Chapter 7:
I feared that some disaster might occur, for I was very fond of the old man,
and I knew that his heart was weak." "How did you know that?" "My friend
Mortimer told me." "You think, then, that some dog pursued Sir Charles, and
that he died of fright in consequence?" "Have you any better explanation?" "I
have not come to any conclusion." "Has Mr. Sherlock Holmes?" The words took
away my breath for an instant but a glance at the placid face and steadfast
eyes of my companion showed that no surprise was intended.

--snip--

Chapter 14:
"What’s the game now?" "A waiting game." "My word, it does not seem a very
cheerful place," said the detective with a shiver, glancing round him at the
gloomy slopes of the hill and at the huge lake of fog which lay over the
Grimpen Mire.

Far away on the path we saw Sir Henry looking back, his face white in the
moonlight, his hands raised in horror, glaring helplessly at the frightful
thing which was hunting him down.

--snip--
```

### **挑战项目：不仅仅是你说了什么，更重要的是你怎么说！**

你迄今为止编写的文本摘要程序是严格按照句子的*重要性顺序*打印句子的。这意味着演讲（或任何文本）中的最后一句话可能会成为摘要中的第一句。摘要的目标是找到重要的句子，但你完全可以改变它们的展示方式。

编写一个文本摘要程序，显示最重要的句子，并保持它们原始的*出现顺序*。将结果与第3个项目中的程序生成的结果进行比较。这样做会对摘要产生明显的改进吗？
