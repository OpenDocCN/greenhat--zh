## **7

**可视化**

![Image](../Images/comm1.jpg)

也许发现数据洞察的最简单方法是通过图形化的方式表示这些数据。像[图 7-1](../Text/ch07.xhtml#ch07fig01)中展示的可视化图形，可以让你立即识别数据中的模式。

在本章中，你将学习如何使用 spaCy 内置的可视化工具（displaCy 依赖可视化工具和 displaCy 命名实体可视化工具）生成句子的句法结构和文档中的命名实体的可视化图。

我们将通过探索这些可视化工具的互动演示来开始，这些演示可以在 Explosion AI 网站上找到（Explosion AI 是 spaCy 的开发者），以了解 spaCy 可视化工具能够实现的功能。接下来，你将学习如何在你的计算机上启动一个 displaCy 网络服务器，从而能够通过编程方式可视化来自 spaCy 的 Doc 对象。你还将学习如何定制你的可视化图。最后，你将学习如何使用 displaCy 渲染手动准备的数据，而不必传入 Doc 对象。

### **开始使用 spaCy 内置的可视化工具**

让我们首先探索一下 displaCy 依赖可视化工具和 displaCy 命名实体可视化工具的工作原理。开始使用 spaCy 内置的可视化工具的最快方法是利用其互动演示，访问 *[https://explosion.ai/demos/](https://explosion.ai/demos/)*。在这个页面中，你可以找到这两个 displaCy 可视化工具的演示页面链接以及其他一些演示链接。

#### ***displaCy 依赖可视化工具***

displaCy 依赖可视化工具为提交的文本生成句法依赖可视化图。要使用其互动演示，请访问 *[https://explosion.ai/demos/displacy/](https://explosion.ai/demos/displacy/)*。将“Text to parse”文本框中的示例句子替换为你的文本，然后点击框右侧的搜索图标（放大镜），以生成可视化图。结果可能如下所示：[图 7-1](../Text/ch07.xhtml#ch07fig01)。

![image](../Images/fig7-1.jpg)

*图 7-1：Explosion AI 网站上的 displaCy 依赖可视化工具*

依赖可视化工具展示了提交文本中的词性标签和句法依赖关系，立即呈现其句法结构。

该可视化工具允许你通过“合并标点符号”和“合并短语”复选框来定制你的图形。“合并标点符号”将标点符号与前面的标记合并，从而使可视化图形更紧凑，更易读。“合并短语”将每个名词短语合并成一个单独的标记，如[图 7-1](../Text/ch07.xhtml#ch07fig01)中的示例所示。默认情况下，两个选项都已启用。

你可以通过取消选中相应的框来更改这些默认设置中的一个或多个。例如，如果你取消选中“合并短语”框，对于句子“我现在想要一份希腊披萨”，你将看到更详细的依赖解析，展示名词短语“a Greek pizza”中的依赖关系。

保持选中“合并短语”框可以让你获得更简洁的依赖树，这在处理包含多个名词短语的句子时尤其有用。例如，考虑以下句子：“我看到一些年轻人在他们的蔬菜田里工作。”它包含两个名词短语：“一些年轻人”和“他们的蔬菜田”。第一个是动词“看到”的直接宾语，第二个是介词短语中表示“工作”的动词修饰的宾语，分别显示依赖标签 dobj 和 pobj。严格来说，这些依赖标签与相应短语中的名词有关，而不是与整个句子有关。

除了“合并标点符号”和“合并短语”选项外，你还可以从可用模型列表中选择一个统计语言模型。这个选项允许你在不下载和安装模型的情况下尝试进行依赖解析。目前，你可以从以下模型中选择：en_core_web_sm、en_core_web_md 和 en_core_web_lg，以及其他欧洲语言的小型(sm)模型，如德语、西班牙语、葡萄牙语、法语、意大利语和荷兰语。

#### ***displaCy 命名实体可视化工具***

displaCy 命名实体可视化工具为提交的文本生成命名实体可视化。你可以在 *[https://explosion.ai/demos/displacy-ent/](https://explosion.ai/demos/displacy-ent/)* 找到其互动演示。从用户的角度来看，它的工作方式类似于前面一节中讨论的 displaCy 依赖可视化演示。要生成文本的可视化，只需将其输入文本框中，然后点击搜索图标。可视化工具会处理你的查询，并在窗口底部输出原始文本，突出显示已发现的命名实体及其标签，如 [图 7-2](../Text/ch07.xhtml#ch07fig02) 所示。

你可以通过选中或取消选中“实体标签”下的框来选择应用程序应该在提交的文本中识别哪些类型的命名实体。在[图 7-2](../Text/ch07.xhtml#ch07fig02)中所示的示例中，你将 PERCENT 和 CARDINAL 添加到默认选择的实体标签类型列表中。添加 PERCENT 实体类型会告诉可视化工具识别表示百分比或包含“%”符号的短语。添加 CARDINAL 实体类型则保证可视化工具会识别与提交文本中的数字相关的短语。

你应该选择哪些框取决于你的上下文。例如，在处理财务报告时，你可能会选择钱币和日期框。但如果报告包括多个公司的财务活动记录，你可能还想选择 ORG 实体标签框，以指示可视化工具在文本中突出显示公司名称。

![image](../Images/fig7-2.jpg)

*图 7-2：Explosion AI 网站上的 displaCy 命名实体可视化工具*

### **从 spaCy 内部进行可视化**

从 spaCy v2.0 开始，displaCy 可视化工具已集成到核心库中。这意味着你在安装 spaCy 后，可以立即在 Python 代码中使用它们。

为了做到这一点，你必须使用以下过程：启动内置的 Web 服务器，然后将一个 Doc 对象（或 Doc 对象列表）发送给它进行渲染。服务器将为提交的 Doc 生成一个可视化，你可以在浏览器中查看它。在本节中，我们将演示几个例子。

#### ***依存解析可视化***

以下脚本展示了生成句子依存树可视化的最简单方法：

import spacy

nlp = spacy.load('en')

➊ doc = nlp(u"I want a Greek pizza.")

➋ from spacy import displacy

➌ displacy.serve(doc, ➍style='dep')

我们创建一个 Doc 对象以提交给 displaCy ➊。然后我们从核心库导入 displaCy 库 ➋，之后可以启动一个 displaCy Web 服务器，将 Doc 对象传递给它。这两个操作通过调用 displacy.serve() 方法完成 ➌。通过将 style 参数设置为 'dep'，我们指示 displaCy 使用依存可视化工具 ➍，为 Doc 对象中的文本生成一个依存树可视化。如果你有兴趣实现本章前面探讨的复选框选项，请参考 [“试试这个”](../Text/ch07.xhtml#lev97)，并查看 [第 104 页](../Text/ch07.xhtml#page_104)。

无论你是以 Python 会话运行此代码，还是作为独立脚本运行，它都会进入一个无限循环并显示来自 displaCy Web 服务器的消息。你应该首先看到以下初始消息：

正在端口 5000 上提供服务...

使用 'dep' 可视化工具

这意味着服务器已为提交的文本生成了依存树可视化，并在主机的端口 5000（默认端口）上提供 HTTP 请求。实际操作中，这意味着你可以将浏览器指向 *http://localhost:5000* 来查看可视化内容。在这个例子中，它应该看起来像 [图 7-3](../Text/ch07.xhtml#ch07fig03)。

![image](../Images/fig7-3.jpg)

*图 7-3：一个依存树可视化示例，你可以从 Python 代码内部生成并在浏览器中查看*

要关闭 displaCy 服务器，请在你执行启动服务器脚本的终端中按 CTRL-C。这样，你应该会看到服务器发出的以下最终消息：

正在关闭端口 5000 上的服务器。

关闭服务器后，你将无法在浏览器中生成新的可视化副本，但你仍然可以查看已生成的副本。

#### ***尝试这个***

尝试通过修改上一节中的依赖关系可视化器脚本，来使用命名实体可视化器。要指示displaCy应用命名实体可视化器，请将displacy.serve()方法的style参数设置为'ent'。

为了生成更有趣的可视化，你可以使用包含更多句子的较长文本。例如，尝试使用以下文本：

Microsoft Windows是一系列由微软开发的专有操作系统。

由微软销售。比尔·盖茨于11月10日宣布了微软Windows。

1983年，微软于1985年11月20日首次发布Windows。Windows

1.0最初以100美元的价格出售，其销售量在四天内超过了50万份。

1987年4月。为了对比，Windows 95的销量超过了一百万份。

仅在前四天内。

在对Doc对象调用displacy.serve()方法后，将浏览器指向*http://localhost:5000*以查看可视化结果。请注意此示例文本中命名实体及其类型，并观察实体识别器如何识别出人名、产品名、公司名，以及与日期、数字和货币相关的短语。

#### ***逐句可视化***

可视化依赖树在处理单一句子时效果很好。但是，当你可视化长文本时，图形可能会变得冗长且占用空间，这使得它们在一行中显示时难以阅读。尽管displaCy会为每个句子生成单独的可视化，当你传入一个包含多个句子的Doc时，它会将它们排成一行。

你可以选择逐句进行可视化，而不是传入一个Doc对象。例如，当你需要从整篇话语中提取意义并想探索一系列句子时，这可能非常有用。从版本2.0.12开始，displaCy允许你传入Span对象，并将可视化结果按行排列。如果你想为doc.sents中找到的每个句子创建一个可视化，可以传入doc.sents的列表，示例如下：

import spacy

nlp = spacy.load('en')

doc = nlp(u"我有一条宽松的牛仔裤，现在我想要一条紧身的。")

➊ spans = list(doc.sents)

来自spacy的displacy

displacy.serve(➋spans, style='dep')

如[第3章](../Text/ch03.xhtml#ch03)中讨论的，Doc对象的doc.sents属性是一个迭代器，用于迭代Doc对象中的句子。因此，不能通过索引来引用这些句子，但你可以通过循环迭代它们，或创建一个Span对象的列表，其中每个span表示一个句子。在这段代码中，我们将Doc中的句子转换为Span对象列表 ➊。然后我们将这个span列表传递给displacy.serve()进行可视化 ➋。

这应该为每个句子生成一个可视化图，按行排列，允许你通过垂直滚动查看它们。

### **自定义你的可视化与选项参数**

除了你在之前的例子中看到的docs和style参数，displacy.serve()方法还可以接受其他多个参数。options参数可能是最有趣的之一，因为它允许你定义一个设置字典来自定义可视化的布局。在本节中，我们将介绍一些最有用的选项设置。

#### ***使用依赖可视化选项***

长句子在一行显示时可能难以查看。在这种情况下，你可以创建*紧凑模式*的可视化，它占用的空间较少。为此，你需要在选项参数中将“compact”选项设置为True，如本脚本所示。该脚本还更改了可视化器使用的字体。（displaCy API文档中所有可用选项的完整列表可以在*[https://spacy.io/api/top-level/#options-dep/](https://spacy.io/api/top-level/#options-dep/)*找到。）

import spacy

nlp = spacy.load('en')

doc = nlp(u"I want a Greek pizza.")

from spacy import displacy

options = {➊'compact': True, ➋'font': 'Tahoma'}

displacy.serve(doc, style='dep', ➌options=options)

displacy.serve()方法期望options参数是一个字典。在这个例子中，我们只设置了两个选项：'compact'选项为True ➊，以及'font'选项为'Tahoma' ➋，在这两种情况下都改变了默认值。（该可视化器允许你使用大多数标准网页字体，如Arial、Courier等。）然后我们将选项字典作为options参数传递给displacy.serve() ➌。

[图 7-4](../Text/ch07.xhtml#ch07fig04)展示了当你运行脚本后，浏览器指向*http://localhost:5000*时应该显示的内容。

图中看到的方弧可能看起来不太寻常，但它们使整体可视化更紧凑，这通常可以避免你必须滚动才能看到完整的图表。

![image](../Images/fig7-4.jpg)

*图 7-4：自定义依赖关系可视化的示例*

#### ***试试看***

在[第98页](../Text/ch07.xhtml#page_98)的“displaCy依赖可视化工具”互动演示中，您使用了“合并短语”和“合并标点符号”选项。在spaCy中，您可以使用collapse_phrases选项将名词短语合并为一个标记（图7-1显示了在displaCy可视化器网站上的等效选项），该选项默认设置为False。负责将标点符号附加到标记的collapse_punct选项默认设置为True。

修改之前脚本中的代码，使其在传入的选项集合中将collapse_phrases选项设置为True。运行脚本，然后在浏览器中查看生成的可视化结果，确保名词短语以单一标记的形式显示。

#### ***使用命名实体可视化器选项***

命名实体可视化器选项的列表（可在*[https://spacy.io/api/top-level/#displacy_options-ent/](https://spacy.io/api/top-level/#displacy_options-ent/)*查看）比依赖关系可视化器选项的列表要短得多。使用命名实体可视化器时，您可以通过'ents'选项选择要突出显示的实体类型，并通过'colors'选项覆盖默认的颜色设置。

第一个选项是两个选项中更重要的一个，因为它允许您指示可视化器仅突出显示选定类型的实体。以下示例说明了一个您可能希望限制可视化器显示的实体类型的情况。

在此脚本中，我们未为命名实体可视化器提供任何选项，这意味着它将突出显示提交的Doc中的所有类型的实体。

import spacy

nlp = spacy.load('en')

doc = nlp(u"在2011年，Google推出了Google +，这是其在社交领域的第四次尝试")

networking.")

➊ doc.user_data['title'] = "命名实体可视化示例"

from spacy import displacy

displacy.serve(doc, style=➋'ent')

我们使用Doc的user_data属性为Doc设置标题➊。displaCy可视化器会自动将此属性中的文本作为可视化的标题。为可视化添加标题是可选的，但当您需要注释您的可视化时，它是很有用的。

我们将displacy.serve()的style参数设置为'ent' ➋，指示displaCy使用命名实体可视化器。生成的可视化效果应与[图7-5](../Text/ch07.xhtml#ch07fig05)中的内容相似（尽管这些图像是灰度的，但网站使用彩色图像）。

![image](../Images/fig7-5.jpg)

*图7-5：使用默认选项生成的命名实体可视化示例*

总体来看，可视化效果很好。但在这个语境下，突出显示序数词“第四”可能并不必要。很难理解在这个语境下为何我们需要提取这些信息。通过 'ents' 选项，我们选择了希望可视化器突出显示的实体类型。以下代码示范了如何实现这一点。要查看其效果，我们将之前脚本中的最后一行代码替换为以下两行代码，然后运行更新后的脚本：

options = {➊'ents': ["ORG", "PRODUCT", "DATE"], ➋'colors': {"ORG": "aqua",

"PRODUCT": "aqua"}} 

displacy.serve(doc, style='ent', options=options)

这次，可视化器不应该识别除了类型为 ORG、PRODUCT 或 DATE 的实体之外的任何实体 ➊。这个示例还展示了 'colors' 选项的使用，允许我们更改分配给实体类型的默认颜色。在这个示例中，我们将 ORG 和 PRODUCT 类型映射为颜色 "aqua" ➋。

**注意**

*使用 'colors' 选项，你可以将实体类型映射到 Web 颜色名称或十六进制颜色代码。在[第 105 页](../Text/ch07.xhtml#page_105)的示例中，使用十六进制代码 "#00FFFF" 等同于使用颜色名称 "aqua"。*

[图 7-6](../Text/ch07.xhtml#ch07fig06) 展示了生成的可视化效果。

![image](../Images/fig7-6.jpg)

*图 7-6：使用 'ents' 和 'colors' 选项生成的实体可视化示例*

如你所见，这个可视化几乎与[图 7-5](../Text/ch07.xhtml#ch07fig05)中展示的完全相同。不同之处在于，这次可视化没有突出显示类型为 ORDINAL 的实体，因为我们传入的 'ents' 选项列表中不包括这种类型。

### **将可视化导出为文件**

到目前为止的示例中，我们使用了 displaCy Web 服务器来查看正在生成的可视化。正如你在这些示例中所学到的，只要使用相同调用的 displacy.serve() 启动的服务器正在运行，你就可以在浏览器中打开使用 displacy.serve() 方法生成的可视化。

使用 displacy.render() 方法，你可以避免这种限制，并创建一个可供后续使用的可视化。displacy.render() 方法允许你将标记渲染为 HTML 页面，并将其保存到单独的文件中。以后，你可以在任何浏览器中打开该文件，而无需启动 Web 浏览器。

以下脚本展示了如何使用 displacy.render() 方法生成与[图 7-5](../Text/ch07.xhtml#ch07fig05)中所示的命名实体可视化：

import spacy

nlp = spacy.load('en')

doc = nlp(u"2011年，Google 推出了 Google +，这是其第四次进入社交领域")

networking.")

➊ doc.user_data["title"] = "实体可视化示例"

#在下一个代码块中，你指示displaCy渲染标记并将其包装为

完整的 HTML 页面。

from spacy import displacy

➋ html = displacy.render(doc, style='ent', ➌page=True)

#在下一个代码块中，你将保存通过 displacy.render() 生成的 HTML 文件

硬盘上的数据。

➍ from pathlib import Path

➎ output_path = Path("/visualizations/ent_visual.html")

➏ output_path.open("w", encoding="utf-8").write(html)

我们可以将这段代码分为三个部分，每个部分以注释行开始。第一部分应该很熟悉。在这里，我们创建一个文本处理管道，然后将其应用于文本。接着，我们使用 Doc 的 user_data 属性为 Doc 设置一个标题 ➊。

在第二个代码块中，我们使用之前步骤创建的 Doc 渲染命名实体可视化，方法是使用 displacy.render() ➋。与 displacy.serve() 不同，displacy.render() 不会运行 Web 服务器，而是生成可视化的 HTML 标记。通过将页面参数设置为 True，我们指示 displacy.render() 生成完整 HTML 页面的标记 ➌。

在最后一个代码块中，我们导入 pathlib 模块中的 Path 类，该模块是在 Python 3.4 中引入的 ➍。我们可以使用 Path 对路径对象执行系统调用。在这个示例中，我们在以下路径上实例化该类：*/visualizations/ent_visual.html* ➎，假设我们已经在本地文件系统中创建了 */visualizations* 文件夹（否则会抛出异常）。然后我们打开 *ent_visual.html* 文件（如果该文件不存在，将会创建）并将上一步生成的 HTML 页面写入其中 ➏。

总结来说，这段脚本生成一个包含提交文本的命名实体可视化的 HTML 文件，并将该文件保存在你的文件系统中。如果你进入保存文件的文件夹并双击该文件，它将在浏览器中打开。

### **使用 displaCy 手动渲染数据**

displaCy 可视化工具允许你手动创建数据集，然后渲染它，而不是将数据作为 doc 或 span 传递。这在你需要可视化来自其他 NLP 库的输出，或者需要使用一组自定义标签或自定义依赖标签来创建可视化时非常有用。（我将在[第10章](../Text/ch10.xhtml#ch10)讨论如何创建这些自定义标签和标签。）

例如，我们手动渲染句子 “I want a Greek pizza.”。

#### ***格式化数据***

首先，你需要将数据按照 displaCy 的格式放置：一个包含两个列表的字典："words" 和 "arcs"，如下代码所示：

sent = {

"words": [

➊ {"text": "I", "tag": "PRON"}

{"text": "want", "tag": "VERB"},

{"text": "a", "tag": "DET"},

{"text": "Greek", "tag": "ADJ"},

{"text": "pizza", "tag": "NOUN"}

],

"arcs": [

➋ {"start": 0, "end": 1, "label": "nsubj", "dir": "left"}

{"start": 2, "end": 4, "label": "det", "dir": "left"},

{"start": 3, "end": 4, "label": "amod", "dir": "left"},

{"start": 1, "end": 4, "label": "dobj", "dir": "right"}

]

}

sent 字典包含两个列表：“words”和“arcs”，每个列表中又包含一组字典。“words” 列表中的字典为句子中的某个词元分配一个标签 ➊，而 “arcs” 列表中的字典定义了依存树中的一条弧线，连接句子中两个语法相关的词元 ➋。在本例中，句子中有五个词元，并且定义了四个语法关系。因此，字典中的 "words" 列表包含五个项目，而 "arcs" 列表包含四个项目。

现在我们有了包含数据的字典，我们需要为示例句子生成一个依存句法分析的可视化图示；我们可以使用以下代码进行渲染：

➊ from spacy import displacy

displacy.serve(➋sent, style="dep", ➌manual=True)

请注意，我们不需要导入整个 spaCy 库。我们只需要从中导入 displacy 模块 ➊。然后我们调用 displacy.serve() 方法，将 sent 字典作为第一个参数传入，而不是传入 Doc 对象 ➋。第三个参数 manual 告诉 displaCy 我们是手动创建了渲染的数据集 ➌，因此 displaCy 不需要从 Doc 对象中提取数据。

#### ***尝试一下***

当你选择手动创建一个包含数据的字典以渲染为可视化时，你可以使用自定义标签，例如，指定可视化工具应使用细粒度的词性标签，而不是默认使用的粗粒度标签。

你可以通过在传入 Doc 对象进行渲染时将 fine_grained 选项设置为 True 来完成这个任务，但为了练习，尝试手动实现这一过程。

在 “[格式化数据](../Text/ch07.xhtml#lev101)” 示例中，[第 108 页](../Text/ch07.xhtml#page_108)，将 "sent" 字典中 "words" 列表中的标签更改为细粒度标签。接着，启动 displaCy 服务器，并指示它根据 "sent" 字典中指定的数据生成可视化图示。然后在浏览器中访问 *http://localhost:5000* 以查看可视化图示。

### **摘要**

你在之前的章节中已经看过句法结构的可视化图示，但在本章中，你学习了如何使用 displaCy 依存关系可视化工具生成这些可视化图示。你还学会了如何使用 displaCy 命名实体可视化工具生成命名实体信息的图形。
