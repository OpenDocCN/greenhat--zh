## **6

**用隐形墨水书写**

![image](img/common01.jpg)

2012 年秋季，犯罪剧情剧 *Elementary* 在 CBS 电视网首播。这部剧重新构思了 21 世纪纽约的 Sherlock Holmes 神话，主演 Jonny Lee Miller 饰演 Holmes，Lucy Liu 饰演他的搭档 Joan Watson 博士。在 2016 年的一集（“你抓住了我，那谁抓住了你？”）中，Sherlock 的疏远父亲 Morland Holmes 雇佣 Joan 查找他组织中的间谍。她迅速通过在一封电子邮件中发现 Vigenère 密码来破案。但一些剧迷对此不满：Vigenère 密码并不隐蔽，像 Morland Holmes 这样聪明的人怎么可能自己没发现？

在这个项目中，你将使用隐写术解决这个难题，但不会像在 第五章 中那样使用空密码。为了隐藏这个消息，你将使用一个名为 `python-docx` 的第三方模块，它允许你通过 Python 直接操作 Microsoft Word 文档来隐藏文本。

### **项目 #12：隐藏 Vigenère 密码**

在 *Elementary* 的这一集里，中国投资者雇佣 Morland Holmes 的咨询公司与哥伦比亚政府谈判石油许可证和钻探权。一年过去了，最后一刻，一位竞争对手抢先达成了交易，留下中国投资者空手而归。Morland 怀疑自己团队中的某个成员背叛了他，便让 Joan Watson 独自进行调查。Joan 通过在一封电子邮件中发现 Vigenère 密码找到了间谍。

**剧透警告**

*解密后的内容从未提及，间谍在随后的剧集中被谋杀。*

*Vigenère 加密*，也被称为不可破译的密码，毫无疑问是有史以来最著名的加密方法。它由法国学者 Blaise de Vigenère 于 16 世纪发明，是一种多表替换加密方法，在最常用的版本中，采用了一个单一的关键字。这个关键字，比如 *BAGGINS*，会反复打印在明文上，就像在 图 6-1 中所示的消息一样。

![image](img/f0106-01.jpg)

*图 6-1：使用 Vigenère 加密关键字* BAGGINS *打印在上方的明文消息*

然后使用字母表的表格，或称 *表格*，来加密消息。图 6-2 是 Vigenère 表格前五行的示例。请注意，每一行中的字母都会向左移动一个字母。

![image](img/f0106-02.jpg)

*图 6-2：Vigenère 表格的一部分*

明文字母上方的关键字字母决定了用于加密的行。例如，要加密 *s* 在 *speak* 中，注意它上方的关键字字母是 *B*。往下找到 B 行，并横向阅读到明文 *s* 位于列顶部的位置。使用交点处的 *T* 作为密文字母。

图 6-3 显示了使用维吉尼亚密码加密的完整信息示例。如果这样的文本出现在文档中，肯定会引起注意，并成为审查的对象！

![image](img/f0107-01.jpg)

*图 6-3：使用维吉尼亚密码加密的信息*

维吉尼亚密码一直未被破解，直到 19 世纪中期，计算机前身的发明者查尔斯·巴贝奇意识到，使用一个短的关键字与长消息配合时，会导致重复的模式，这些模式可以揭示关键字的长度，并最终破解出密钥。维吉尼亚密码的破译对专业密码学是一次重创，维多利亚时代的原版《福尔摩斯与华生》时期，密码学领域没有取得显著进展。

这种密码的存在也导致了 *Elementary* 这一集中的“难以置信”问题。为什么需要外部顾问来查找这么明显可疑的电子邮件？让我们看看是否能用 Python 提供一个合理的解释。

**目标**

假设你是这一集中的公司间谍，并使用 Python 将一个总结竞标细节的秘密信息隐藏在一个正式文本文件中。从未加密的信息开始，到加密版本为止。

### **平台**

你的程序应当能够与广泛使用的文字处理软件兼容，因为输出文件需要在不同的公司之间共享。这意味着需要使用适用于 Windows 的 Microsoft Office 套件，或兼容的 macOS 或 Linux 版本。将输出限制为标准的 Word 文档也意味着硬件问题由 Microsoft 承担！

因此，本项目是在 Windows 版 Word 2016 上开发的，结果通过 Word for Mac v16.16.2 进行了检查。如果你没有 Word 的许可证，可以使用免费的 Microsoft Office Online 应用，访问 *[`products.office.com/en-us/office-online`](https://products.office.com/en-us/office-online)*。

如果你目前使用 Word 的替代软件，如 LibreOffice Writer 或 OpenOffice Writer，你仍然可以打开并查看本项目中使用和生成的 Word (*.docx*) 文件；然而，正如在 “检测隐藏信息” 中所讨论的，隐藏的信息很可能会被破坏，详见 第 119 页。

### **策略**

你是一名具有初级 Python 知识的会计师，工作在一个非常聪明且多疑的人手下。你从事的项目具有高度保密性，采用了诸如电子邮件过滤器等控制措施来保持机密性。如果你成功地将信息泄露出去，必定会进行彻底的调查。因此，你需要在电子邮件中隐藏一条明显可疑的信息，无论是直接包含在邮件中还是作为附件，且要避免初步检测和后续的内部审计。

以下是一些约束条件：

+   你不能直接将信息发送到竞争公司，只能通过中介发送。

+   你需要将消息打乱，以避开搜索关键词的电子邮件过滤器。

+   你需要将加密信息隐藏起来，以免引起怀疑。

中介设置起来很容易，而且互联网上有很多免费的加密网站——但最后一项就更有问题了。

隐写术是答案，但正如你在上一章中看到的，将一条简短的信息隐藏在空白密码中并不是一项简单的任务。替代技术包括将文本行垂直或单词水平地微调，改变字母的长度，或使用文件的元数据——但你是一个会计师，Python 知识有限，时间更少。如果能像过去的隐形墨水那样有一种简单的方法该多好。

#### ***创建隐形墨水***

在这个电子墨水的时代，隐形墨水或许正好够疯狂，足以奏效！一种隐形字体可以轻松地阻止在线文档的视觉浏览，并且在纸质打印件中也根本不存在。由于内容被加密，数字过滤器在寻找类似*竞标*或生产油田的西班牙名字时将什么也找不到。最棒的是，隐形墨水使用起来非常简单——只需将前景文本设置为背景颜色。

格式化文本和改变颜色需要使用像 Microsoft Word 这样的文字处理器。要在 Word 中制作隐形电子墨水，你只需要选择一个字符、单词或行，并将字体颜色设置为白色。接收者然后需要选择整个文档，并使用高亮工具（参见图 6-4）将选定文本涂成黑色，从而隐藏标准的黑色字母并将隐藏的白色字母显现出来。

![image](img/f0108-01.jpg)

*图 6-4：Word 2016 中的文本高亮颜色工具*

仅在 Word 中选择文档并不会显示白色文本（图 6-5），因此，除非某人非常怀疑，否则很难发现这些隐藏的消息。

![image](img/f0109-01.jpg)

*图 6-5：上图：Word 文档中可见的假消息的一部分；中图：使用 CTRL-A 选中的文档；下图：使用高亮工具将高亮颜色设置为黑色，揭示真实消息*

当然，你可以仅通过文字处理器来完成所有这些工作，但有两种情况采用 Python 方法更为合适：1）当你需要加密一条长消息，不想手动插入并隐藏所有行时，2）当你需要发送多条消息时。正如你将看到的，简短的 Python 程序将大大简化这个过程！

##### **考虑字体类型、字距和字形调整**

放置隐形文本是一个关键的设计决策。一个选择是使用假消息中可见单词之间的空格，但这可能会触发与间距相关的问题，使得最终产品看起来可疑。

*比例字体*使用可变的字符宽度来提高可读性。示例字体有 Arial 和 Times New Roman。*等宽字体*则使用固定的字符宽度，以便支持文本对齐和识别单独的字符，特别是像（或{这样的细小字符。因此，等宽字体在编程界面中非常受欢迎。示例字体有 Consolas 和 Courier New。

*字距调整*是一个排版过程，用来调整单个字符字形之间的间距和重叠，以提高它们的视觉效果。*字间距调整*（tracking）用于调整整行或整块文本的字符间距，目的相同。这些调整有助于可读性，确保字母之间既不会过于紧密以至于无法区分，也不会过于分散以至于单词无法识别。请注意，我们是读单词，而不是字母。如果你有疑问，试试这样读：peopl raed wrds nt lttrs. 当然，语境也有帮助。

字母对之间的字距调整首先进行，然后是字间距调整（tracking），在此过程中，字母对的相对字距得以保持。如前所述，这些可变宽度和自动修正可能会导致问题，尤其是在你试图隐藏使用比例字体的单词之间的字符时：

| 伟大的思想没有什么是微不足道的。 | *比例字体，没有隐藏字母* |
| --- | --- |
| 伟大的思想没有什么是微不足道的。 | *比例字体，单词之间隐藏了字母* |
| To$a3great.mind2nothingKis little. | *隐藏字母揭示（$3.2K）* |
|  |  |
| `伟大的思想没有什么是微不足道的。` | *等宽字体，没有隐藏字母* |
| `伟大的思想没有什么是微不足道的。` | *等宽字体，单词之间隐藏了字母。* |
| `To``$``a``3``great``.``mind``2``nothing``K``is little.` | *隐藏字母揭示（$3.2K）* |

如果使用等宽字体，一致的间距提供了一个方便的隐藏位置。但是，由于专业信件更可能使用比例字体，因此隐形墨水技术应该集中在更易控制的行间空隙上。

在段落之间使用空行是最简单的编程和阅读方式，而且它不需要很长的虚假消息，因为你可以简明扼要地总结一个提案的要点。这一点非常重要，因为你不希望在可见的虚假消息后附加空白页。因此，你的隐藏消息的占用空间应小于虚假消息的占用空间。

##### **避免问题**

在开发软件时，反复问自己一个好问题是：“用户怎么会把这个搞砸？”这里可能出现的问题之一是加密过程会改变隐藏信息中的字母，导致字距和字形调整可能会使某个单词越过换行符，从而导致自动换行。这会导致伪造信息中的段落之间出现不均匀和可疑的空隙。避免这种情况的一种方法是在输入真实信息的每一行时稍微提前按下 ENTER 键，这样可以在行尾留出一些空白空间，以便适应加密过程中可能发生的变化。当然，你仍然需要验证结果。假设代码能正常工作就像假设詹姆斯·邦德已经死了一样危险！

#### ***使用 python-docx 操控 Word 文档***

一个名为 `python-docx` 的免费第三方模块允许 Python 操作 Microsoft Word（.*docx*）文件。要下载和安装本书中提到的第三方模块，你将使用首选安装程序（pip），这是一种包管理系统，可以轻松地安装基于 Python 的软件。对于 Windows 和 macOS 上的 Python 3，版本 3.4 及更高版本自带 pip；对于 Python 2，pip 的预安装从版本 2.7.9 开始。Linux 用户可能需要单独安装 pip。如果你发现需要安装或升级 pip，可以参照 *[`pip.pypa.io/en/stable/installing/`](https://pip.pypa.io/en/stable/installing/)* 上的说明，或者在网上搜索如何在特定操作系统上安装 pip。

使用 pip 工具，你可以通过在命令行、PowerShell 或终端窗口中运行 `pip install python-docx` 来安装 `python-docx`，具体取决于你的操作系统。关于 `python-docx` 的在线说明可以在 *[`python-docx.readthedocs.io/en/latest/`](https://python-docx.readthedocs.io/en/latest/)* 上找到。

对于这个项目，你需要理解 `paragraph` 和 `run` 对象。`python-docx` 模块通过以下层次结构组织数据类型：

+   `document`：包含多个 `paragraph` 对象的整个文档

+   `paragraph`：通过在 Word 中按 ENTER 键分隔的文本块；包含一个或多个 `run` 对象

+   `run`：具有相同 *样式* 的一串连接的文本

`paragraph` 被认为是一个 *块级* 对象，`python-docx` 对其的定义如下：“一个块级项目将其包含的文本流动到其左右边缘之间，每当文本超出右边界时，都会增加一行。对于 `paragraph` 对象，边界通常是页面边距，但如果页面布局为列格式，边界也可以是列边界；如果 `paragraph` 位于表格单元格内，边界则是单元格边界。表格也是一个块级对象。”

`paragraph` 对象具有多种属性，用于指定其在容器中的位置——通常是页面——以及它如何将内容分割成单独的行。你可以通过 `paragraph` 的 `ParagraphFormat` 属性访问 `paragraph` 的格式属性，并且可以使用 *段落样式分组* 来设置所有 `paragraph` 属性，或者直接应用它们到 `paragraph`。

`run` 是一种*内联级别*的对象，出现在段落或其他块级对象中。`run` 对象具有只读的 `font` 属性，可以访问一个 `font` 对象。`font` 对象提供了用于获取和设置该 `run` 字符格式的属性。你需要这个功能来将隐藏消息的文本颜色设置为白色。

*样式*是指 Word 中段落和字符（`run` 对象）或两者组合的属性集合。样式包括常见的属性，如字体、颜色、缩进、行距等。你可能注意到这些属性在 Word 的“开始”功能区的样式窗格中有分组显示（见 图 6-6）。任何样式的变化——甚至是一个字母——都需要创建一个新的 `run` 对象。目前，只有在打开的 *.docx* 文件中存在的样式才可用。这一情况可能会在未来的 `python-docx` 版本中有所变化。

![image](img/f0111-01.jpg)

*图 6-6：Microsoft Word 2016 中的样式窗格*

你可以在 `python-docx` 中找到完整的样式使用文档，地址是 *[`python-docx.readthedocs.io/en/latest/user/styles-using.html`](http://python-docx.readthedocs.io/en/latest/user/styles-using.html)*。

下面是一个 `python-docx` 看到的段落和 `run` 的示例：

*我是一个包含单个 `run` 的段落，因为我的所有文本都是相同的样式。*

*我是一个包含两个 `run` 的段落。**我是第二个 `run`，因为我的样式变成了粗体。***

*我是一个包含三个 `run` 的段落。**我是第二个 `run`，因为我的样式变成了粗体。第三个 `run` 是我最后一个**单词。*

如果这些内容有任何不清楚的地方，别担心。你不需要详细了解 `python-docx`。就像任何一段代码，你主要需要知道*你想做什么*。在线搜索应该能找到大量有用的建议和完整的代码示例。

**注意**

*为了使这一切顺利进行，不要更改真实（隐藏）消息中的样式，并确保每行的结尾都通过手动按下 ENTER 键来强制换行。不幸的是，Word 没有自动换行所产生的软回车的特殊字符。因此，你无法进入一个已有自动换行的 Word 文档，并使用查找和替换将它们全部转换为强制回车。生活就是这样。*

#### ***下载资源***

你需要的外部文件可以从 *[`www.nostarch.com/impracticalpython/`](https://www.nostarch.com/impracticalpython/)* 下载，并应保存在与代码相同的文件夹中：

***template.docx*** 一份格式化为正式 Holmes Corporation 样式、字体和边距的空白 Word 文档

***fakeMessage.docx*** 没有信头和日期的假消息，格式为 Word 文档

***realMessage.docx*** 真实消息的明文，未加信头和日期，格式为 Word 文档

***realMessage_Vig.docx*** 使用维吉尼亚密码加密的真实消息

***example_template_prep.docx*** 用于创建模板文档的假消息示例（程序运行时不需要此文件）

**注意**

*如果你使用的是 Word 2016，制作空白模板文件的简单方法是编写假消息（包括信头）并保存文件。然后删除所有文本，再次保存文件并更改文件名。当你使用* python-docx *将这个空白文件分配给一个变量时，所有现有样式将被保留。当然，你也可以使用已经包含信头的模板文件，但为了更好地学习* python-docx*，我们将在这里使用 Python 构建信头。*

花点时间查看这前四个文档在 Word 中的样子。这些文件构成了*elementary_ink.py*程序的输入。假消息和真实消息——第二和第三项——也展示在图 6-7 和 6-8 中。

![image](img/f0113-01.jpg)

*图 6-7：* fakeMessage.docx *文件中的“假”文本*

![image](img/f0113-02.jpg)

*图 6-8：* realMessage.docx *文件中的真实消息*

请注意，真实消息包含一些数字和特殊字符。这些字符不会被我们将使用的维吉尼亚表格加密，我特意加入它们以强调这一点。理想情况下，当我们稍后添加维吉尼亚密码时，这些数字和字符应该被拼写出来（例如，将“3”拼写为“三”，将“%”拼写为“百分比”），以确保最大程度的保密。

### **伪代码**

以下伪代码描述了如何加载两个消息和模板文档，将它们交织在一起并使用白色字体将真实消息隐藏在空行中，然后保存合成的消息。

```py
Build assets:
In Word, create an empty doc with desired formatting/styles (template)
In Word, create an innocuous fake message that will be visible & have enough     blank lines to hold the real message
In Word, create the real message that will be hidden
Import docx to allow manipulation of Word docs with Python
Use docx module to load the fake & real messages as lists
Use docx to assign the empty doc to a variable
Use docx to add letterhead banner to empty doc
Make counter variable for lines in real message
Define function to format paragraph spacing with docx
For line in fake message:
    If line is blank and there are still lines in the real message:
        Use docx & counter to fill blank with line from real message
        Use docx to set real message font color to white
        Advance counter for real message
    Otherwise:
        Use docx to write fake line
    Run paragraph spacing function
Use docx to save final Word document
```

### **代码**

Listing 6-1 中的*elementary_ink.py*程序加载了真实消息、假消息和空白模板文档。它使用白色字体将真实消息隐藏在假消息的空行中，然后将合成的消息保存为一份无害且专业的信函，可以附加到电子邮件中。你可以从* [`www.nostarch.com/impracticalpython/`](https://www.nostarch.com/impracticalpython/)*下载代码。

#### ***导入 python-docx，创建列表并添加信头***

Listing 6-1 导入`python-docx`，将假消息和真实消息中的文本行转换为列表项，加载设置样式的模板文档，并添加信头。

*elementary_ink.py，* 第一部分

```py
   import docx
➊ from docx.shared import RGBColor, Pt

➋ # get text from fake message & make each line a list item
   fake_text = docx.Document('fakeMessage.docx')
   fake_list = []
   for paragraph in fake_text.paragraphs:
       fake_list.append(paragraph.text)

➌ # get text from real message & make each line a list item
   real_text = docx.Document('realMessage.docx')
   real_list = []
   for paragraph in real_text.paragraphs:
    ➍ if len(paragraph.text) != 0:  # remove blank lines
           real_list.append(paragraph.text)

➎ # load template that sets style, font, margins, etc.
   doc = docx.Document('template.docx')

➏ # add letterhead
   doc.add_heading('Morland Holmes', 0)
   subtitle = doc.add_heading('Global Consulting & Negotiations', 1)
   subtitle.alignment = 1
   doc.add_heading('', 1)
➐ doc.add_paragraph('December 17, 2015')
   doc.add_paragraph('')
```

*Listing 6-1：导入* python-docx*，加载重要的* .docx *文件，并添加信头*

导入 `docx` 模块后——而不是“python-docx”——使用 `docx.shared` 来访问 `docx` 模块中的颜色（`RGBColor`）和长度（`Pt`）对象 ➊。这些将允许你更改字体颜色并设置行间距。接下来的两个代码块将假消息 ➋ 和真实消息 ➌ 的 Word 文档加载为列表。在每个 Word 文档中按下 ENTER 键的位置决定了这些列表中的项。为了隐藏真实消息，删除任何空白行，使得消息尽可能短 ➍。现在，你可以使用列表索引来合并这两条消息，并跟踪它们各自的来源。

接下来，加载包含预先设置样式、字体和页边距的模板文档 ➎。`docx` 模块将写入这个变量，并最终将其保存为最终文档。

输入准备好之后，格式化最终文档的信头，使其与 Holmes Corporation 的信头一致 ➏。`add_heading()` 函数添加一个带有文本和整数参数的标题样式段落。整数 `0` 表示最高级别的标题，或从模板文档继承的标题样式。副标题使用 `1` 格式化，这是下一个可用的标题样式，并且居中对齐，再次使用整数 `1`（`0` = 左对齐，`2` = 右对齐）。请注意，当你添加日期时，不需要提供整数 ➐。当没有提供参数时，默认情况下继承现有样式层级，在模板中是左对齐的。此代码块中的其他语句只是添加空行。

#### ***格式化与交错消息***

列表 6-2 进行实际的工作，格式化行间距并交错消息。

*elementary_ink.py,* 第二部分

```py
➊ def set_spacing(paragraph):
       """Use docx to set line spacing between paragraphs."""
       paragraph_format = paragraph.paragraph_format
       paragraph_format.space_before = Pt(0)
       paragraph_format.space_after = Pt(0)

➋ length_real = len(real_list)
   count_real = 0  # index of current line in real (hidden) message

   # interleave real and fake message lines
   for line in fake_list:
    ➌ if count_real < length_real and line == "":
        ➍ paragraph = doc.add_paragraph(real_list[count_real])
        ➎ paragraph_index = len(doc.paragraphs) - 1
           # set real message color to white
           run = doc.paragraphs[paragraph_index].runs[0]
           font = run.font
        ➏ font.color.rgb = RGBColor(255, 255, 255) # make it red to test
        ➐ count_real += 1
       else:
        ➑ paragraph = doc.add_paragraph(line)

    ➒ set_spacing(paragraph)

➓ doc.save('ciphertext_message_letterhead.docx')

   print("Done")
```

*列表 6-2：格式化段落并交错假消息与真实消息的行*

定义一个函数，使用 `python-docx` 的 `paragraph_format` 属性格式化段落之间的间距 ➊。隐藏行之前和之后的行间距设置为 `0` 点，以确保输出中段落之间没有可疑的大间隙，如 图 6-9 左侧的间隙。

![image](img/f0116-01.jpg)

*图 6-9：没有* python-docx *段落格式化的假消息行间距（左）与有格式化的行间距（右）*

接下来，通过获取存储真实消息的列表的长度来定义工作空间 ➋。记住，隐藏的真实消息需要比可见的假消息短，以便有足够的空白行容纳它。然后启动一个计数器，程序将用它来跟踪当前正在处理的真实消息中的行（列表项）。

由于从伪造信息创建的列表最长，并为真实信息设置了维度空间，因此通过两个条件循环遍历伪造信息：1）是否已达到真实信息的结尾，2）伪造列表中的一行是否为空 ➌。如果仍有真实信息行，而伪造信息行为空，则使用`count_real`作为`real_list`的索引，并使用`python-docx`将其添加到文档中 ➍。

通过获取`doc.paragraphs`的长度并减去 1 ➎，获取你刚刚添加的行的索引。然后使用此索引将真实信息行设置为一个`run`对象（它将是列表中的第一个`run`项`[0]`，因为真实信息使用单一的样式），并将其字体颜色设置为白色 ➏。由于程序现在已经在此块中从真实列表中添加了一行，`count_real`计数器将增加 1 ➐。

随后的`else`块处理从伪造列表中选择的行不是空的情况。在这种情况下，伪造信息行将直接添加到段落中 ➑。通过调用行间距函数`set_spacing()` ➒，完成`for`循环。

一旦超过了真实信息的长度，`for`循环将继续添加伪造信息的剩余部分——在此案例中是 Kurtz 先生的签名信息——并将文档作为 Word *.docx* 文件保存到最后一行 ➓。当然，在实际生活中，你应该使用一个不那么可疑的文件名，比如*ciphertext_message_letterhead.docx*！

请注意，因为你正在使用基于伪造信息的`for`循环，循环结束后，即在你达到伪造列表中项的末尾后，无法再添加任何更多的隐藏行。如果你想要更多空间，你必须在伪造信息的底部输入硬回车，但要小心不要添加太多，以免强制分页并产生一个神秘的空白页！

运行程序，打开保存的 Word 文档，使用 CTRL-A 选择所有文本，然后将高亮颜色（见图 6-4）设置为深灰色，以查看两条信息。秘密信息应已被揭示（见图 6-10）。

![image](img/f0117-01.jpg)

*图 6-10：使用深灰色高亮显示的 Word 文档，展示了伪造的信息和未加密的真实信息*

#### ***添加维吉尼亚密码***

目前的代码使用了真实信息的纯文本版本，因此任何更改文档高亮颜色的人都能读取并理解其中的敏感信息。既然你知道 Kurtz 先生是使用维吉尼亚密码加密的，那么返回并修改代码，将纯文本替换为加密文本。为此，找到以下行：

```py
real_text = docx.Document('realMessage.docx')
```

这一行加载了真实信息的纯文本，所以将文件名更改为此处加粗显示的文件名：

```py
real_text = docx.Document('realMessage_Vig.docx')
```

重新运行程序，通过选择整个文档并将高亮颜色设置为深灰色（见图 6-11）再次显示隐藏的文本。

![image](img/f0118-01.jpg)

*图 6-11：用深灰色突出显示的 Word 文档，展示了假信息和加密后的真实信息*

秘密信息应该是可见的，但对任何无法解读密码的人来说是无法阅读的。比较图 6-11 中的加密信息与图 6-10 中的未加密版本。请注意，数字和百分号在两个版本中都有出现。保留这些内容是为了展示与加密选择相关的潜在陷阱。你可能需要增强维吉尼亚密码，以包括这些字符——或者直接拼写出来。这样，即使你的信息被发现，你也会尽量避免留下关于主题的线索。

如果你想使用维吉尼亚密码来编码自己的信息，可以通过互联网搜索“在线维吉尼亚编码器”。你会找到多个网站，例如*[`www.cs.du.edu/~snarayan/crypt/vigenere.html`](http://www.cs.du.edu/~snarayan/crypt/vigenere.html)*，可以在其中输入或粘贴明文。如果你想编写自己的 Python 程序来加密维吉尼亚密码，请参阅 Al Sweigart 的《Python 破解密码》（No Starch Press，2018）。

如果你自己尝试处理真实信息，无论是加密还是未加密的，确保使用与假信息中相同的字体。字体不仅指的是字体类型，比如 Helvetica 斜体，还包括字体大小，如 12 号。记住在“考虑字体类型、字距和行距”中提到的，在第 109 页，如果你尝试混合字体，尤其是比例字体和等宽字体，隐藏信息的行可能会换行，导致真实信息的段落之间出现不均匀的间距。

### **检测隐藏信息**

琼·沃森或其他任何侦探，能快速找到你隐藏的信息吗？事实上，可能不会。事实上，在我写这些话的时候，我正在看《Elementary》的一集，琼正在通过阅读一箱电子邮件打印件调查一家公司！维吉尼亚密码的使用可能只是整个精心构思的系列剧中的一点懒散写作。尽管如此，我们还是可以推测出哪些因素可能会暴露你的秘密信息。

由于最终的投标很可能直到接近投标日期才被发送，因此搜索可以限制为投标确定后*发送的*通信，从而消除很多噪音。当然，侦探不会确切知道他们在寻找什么——甚至不知道是否*有*内奸——这使得搜索范围很大。而且，信息也有可能通过电话交谈或秘密会议传递。

假设邮件的数量是可控的，并且正在追寻隐藏信息的假设，调查员可能会通过多种方式检测到你的隐形墨水。例如，只要这些白色的无意义加密词没有显示出来，Word 拼写检查器是不会标记它们的。如果你进行检查，选中并重设了某些隐藏词的字体颜色，即便它们的颜色恢复为白色，它们也会被永久暴露，拼写检查器会用一条指责性的红色波浪线标出这些词（参见图 6-12）。

![image](img/f0119-01.jpg)

*图 6-12：之前暴露的隐形加密词被 Word 拼写和语法工具下划线标记*

如果调查员使用与 Word 不同的软件打开文档，该软件的拼写检查器很可能会揭示隐藏的词汇（参见图 6-13）。这种风险在 Microsoft Word 占据市场主导地位的情况下有所减轻。

![image](img/f0120-01.jpg)

*图 6-13：LibreOffice Writer 中的拼写检查器将突出显示隐形词汇。*

其次，使用 CTRL-A 来高亮显示 Word 文档中的所有文本并不会显示隐藏的文本，但它会表明一些空白行比其他行要长（参见图 6-14），这会让非常细心的人察觉到有问题。

![image](img/f0120-02.jpg)

*图 6-14：选择整个 Word 文档会揭示空白行长度的差异。*

第三，使用某些电子邮件软件中的预览功能打开 Word 文档时，选中内容时（通过滑动或使用 CTRL-A）可能会揭示隐藏的文本（参见图 6-15）。

![image](img/f0121-01.jpg)

*图 6-15：在 Yahoo! Mail 预览面板中选择整个文档会显示隐藏的文本。*

但在 Yahoo! Mail 预览面板中选择隐藏文本会显示文本，而在 Microsoft Outlook 预览面板中则不会显示（参见图 6-16）。

![image](img/f0121-02.jpg)

*图 6-16：在 Microsoft Outlook 预览面板中选择整个文档并不会显示隐藏的文本。*

最后，将 Word 文档保存为纯文本（**.txt**）文件将移除所有格式，并将隐藏的文本暴露出来（图 6-17）。

![image](img/f0122-01.jpg)

*图 6-17：将 Word 文档保存为纯文本（**.txt**）文件会显示隐藏的文本。*

要使用隐写术隐藏秘密信息，你不仅需要隐藏信息的*内容*，还要隐藏信息的*存在*。我们的电子隐形墨水并不能总是保证这一点，但从间谍的角度来看，上述提到的弱点要么是他们犯了错误（理论上可以控制），要么是调查员采取了非常规且不太可能的行动，比如拷贝文本、以不同格式保存文件，或使用不常见的文字处理软件。假设《Elementary》中的间谍认为这些是可以接受的风险，那么电子隐形墨水提供了一个合理的解释，为什么公司内部调查会失败。

### **总结**

在这一章中，你使用隐写术将加密信息隐藏在 Microsoft Word 文档中。你使用了一个名为`python-docx`的第三方模块，通过 Python 直接访问和操作文档。其他流行文档类型（如 Excel 表格）也有类似的第三方模块可供使用。

### **进一步阅读**

你可以在 *[`python-docx.readthedocs.io/en/latest/`](https://python-docx.readthedocs.io/en/latest/)* 和 *[`pypi.python.org/pypi/python-docx`](https://pypi.python.org/pypi/python-docx)* 上找到 `python-docx` 的在线文档。

*用 Python 自动化无聊的事情*（No Starch Press，2015）由阿尔·斯威加特（Al Sweigart）编写，涵盖了允许 Python 操作 PDF、Word 文件、Excel 表格等模块。第十三章包含了关于`python-docx`的有用教程，附录部分则介绍了如何使用 pip 安装第三方模块。

你可以在阿尔·斯威加特（Al Sweigart）的《破解 Python 密码学》（No Starch Press，2018）一书中找到适合初学者的密码学 Python 程序。

*神秘信息*（企鹅出版集团，2009）由加里·布莱克伍德（Gary Blackwood）编写，是一本有趣且图文并茂的关于隐写术和密码学的历史书籍。

### **练习项目：检查空白行数**

通过编写一个函数来改进隐藏信息的程序，该函数将比较虚假信息中的空白行数与真实信息中的行数。如果没有足够的空间来隐藏真实信息，函数应该警告用户并告诉他们需要在虚假信息中添加多少空白行。在`elementary_ink.py`代码的副本中插入并调用该函数，调用位置在加载模板文档之前。你可以在附录中找到解决方案，也可以在 *[`www.nostarch.com/impracticalpython/`](https://www.nostarch.com/impracticalpython/)* 上找到 *elementary_ink_practice.py* 的代码。进行测试时，从同一网站下载 *realMessageChallenge.docx* 文件，并使用它作为真实信息。

### **挑战项目：使用等宽字体**

重写 *elementary_ink.py* 代码以使用等宽字体，并在单词之间的空隙中隐藏你自己的短信息。有关等宽字体的描述，请参见 “考虑字体类型、字距和字间距” 以及 第 109 页。像往常一样，挑战项目没有提供解决方案。
