## **3

处理容器对象和自定义 spaCy**

![Image](img/comm1.jpg)

你可以将构成 spaCy API 的主要对象分为两类：容器（如 Tokens 和 Doc 对象）和处理管道组件（如词性标注器和命名实体识别器）。本章进一步探讨容器对象。通过使用容器对象及其方法，你可以访问 spaCy 为文本中每个标记分配的语言学注释。

你还将学习如何自定义管道组件，以满足你的需求，并使用 Cython 代码加速耗时的自然语言处理任务。

### **spaCy 的容器对象**

一个 *容器对象* 将多个元素组合成一个单一单元。它可以是对象的集合，如标记或句子，或者与单个对象相关的一组注释。例如，spaCy 的 Token 对象是一个容器，包含与文本中单个标记相关的一组注释，例如该标记的词性。spaCy 中的容器对象模仿了自然语言文本的结构：文本由句子组成，每个句子包含标记。

Token、Span 和 Doc 是 spaCy 中最常用的容器对象，分别表示标记、短语或句子和文本。容器可以包含其他容器——例如，一个 Doc 包含 Tokens。在本节中，我们将探讨如何使用这些容器对象。

#### ***获取 Doc 对象中标记的索引***

Doc 对象包含由提交文本的标记化生成的一组 Token 对象。这些标记有索引，允许你根据它们在文本中的位置访问它们，如 图 3-1 所示。

![image](img/fig3-1.jpg)

*图 3-1：Doc 对象中的标记*

标记的索引从 0 开始，这使得文档长度减去 1 就是结束位置的索引。要将 Doc 实例拆分为标记，你可以通过迭代 Doc，从开始标记到结束标记，将标记提取为 Python 列表：

>>> [doc[i] for i in range(len(doc))]

[A, severe, storm, hit, the, beach, .]

值得注意的是，我们可以显式使用构造函数创建一个 Doc 对象，如下例所示：

>>> from spacy.tokens.doc import Doc

>>> from spacy.vocab import Vocab

>>> doc = Doc(➊Vocab(), ➋words=[u'Hi', u'there'])

doc

你好

我们调用 Doc 的构造函数，传入以下两个参数：一个 *词汇对象* ➊——它是一个存储容器，提供词汇数据，如词汇类型（形容词、动词、名词等）——以及一个要添加到正在创建的 Doc 对象中的标记列表 ➋。

#### ***迭代标记的句法子节点***

假设我们需要查找句法依赖分析中一个词的左侧子节点。例如，我们可以将这个操作应用于名词，从而获得其形容词（如果有的话）。如果我们想知道哪些形容词能够修饰给定的名词，就可能需要这样做。举个例子，考虑以下句子：

I want a green apple.

图 3-2 中的图表突出显示了感兴趣的句法依赖关系。

![image](img/fig3-2.jpg)

*图 3-2：一个左侧句法依赖关系的示例*

要在程序中获取这个示例句子中“apple”的左侧句法子节点，我们可以使用以下代码：

>>> doc = nlp(u'I want a green apple.')

>>> [w for w in doc[4].lefts]

[a, green]

在这个脚本中，我们简单地遍历“apple”的子节点，并将它们输出为一个列表。

有趣的是，在这个例子中，单词“apple”的左侧句法子节点表示该词所有句法子节点的整个序列。实际上，这意味着我们可以将 Token.lefts 替换为 Token.children，从而找到一个词的所有句法子节点：

>>> [w for w in doc[4].children]

结果列表将保持不变。

我们还可以使用 Token.rights 来获取一个词的右侧句法子节点：在这个例子中，单词“apple”是单词“want”的右侧子节点，如图 3-1 所示。

#### ***doc.sents 容器***

通常，分配给词汇的语言注释只有在词汇所在的句子上下文中才有意义。例如，关于某个词是名词还是动词的信息可能只适用于该词所在的句子（就像前面章节中讨论的“count”一词）。在这种情况下，能够使用句子级索引访问文档中的词汇将会非常有用。

Doc 对象的 doc.sents 属性允许我们将文本分割成单独的句子，如下例所示：

>>> doc = nlp(u'A severe storm hit the beach. It started to rain.')

➊ >>> for sent in doc.sents:

➋ ...   [sent[i] for i in range(len(sent))]

...

[A, severe, storm, hit, the, beach, .]

[It, started, to, rain, .]

>>>

我们遍历文档中的句子 ➊，为每个句子创建一个独立的词汇列表 ➋。

与此同时，我们仍然可以使用全局（文档级）索引引用多句文本中的词汇，如下所示：

>>> [doc[i] for i in range(len(doc))]

[A, severe, storm, hit, the, beach, ., It, started, to, rain, .]

通过句子级索引引用文档中的 Token 对象非常有用，举个例子，我们可能需要检查正在处理的文本中第二个句子的第一个单词是否是代词（例如，我们想弄清楚两个句子之间的关系，第一个句子包含一个名词，而第二个句子包含一个指代该名词的代词）：

>>> for i,sent in enumerate(doc.sents):

...   if i==1 and sent[0].pos_== 'PRON':

...     print('第二个句子以代词开头。')

第二个句子以代词开头。

在这个例子中，我们在 for 循环中使用枚举器，通过索引区分句子。这使我们能够筛选出不感兴趣的句子，只检查第二个句子。

确定句子中的第一个词非常简单，因为它的索引总是 0。但是最后一个词呢？例如，如果我们需要找出文本中有多少个句子以动词结尾——（当然不包括句号）怎么办？

>>> counter = 0

>>> for sent in doc.sents:

...     if sent[len(sent)-2].pos_ == 'VERB':

...     counter+=1

>>> print(counter)

1

尽管句子的长度各异，我们可以轻松地使用 len() 函数确定给定句子的长度。我们将 len(sent) 的值减去 2，原因如下：首先，索引总是从 0 开始，到 size-1 结束。其次，示例中两个句子的最后一个词都是句号，我们需要忽略它。

#### ***doc.noun_chunks 容器***

一个 Doc 对象的 doc.noun_chunks 属性允许我们遍历文档中的名词短语。一个 *名词短语* 是以名词为核心的短语。例如，前一句包含以下名词短语：

一个名词短语

一个短语

一个名词

它的核心

使用 doc.noun_chunks，我们可以通过以下方式提取名词短语：

>>> doc = nlp(u'一个名词短语是以名词为核心的短语。')

>>> for chunk in doc.noun_chunks:

...   print(chunk)

或者，我们可以通过遍历句子中的名词，并找到每个名词的句法子项来提取名词短语。早些时候，在 “遍历 Token 的句法子项” 第 33 页 你看到了如何基于句法依赖解析来提取短语的示例。现在，让我们将这个技术应用于本例中的示例句子，手动组成名词短语：

for token in doc:

➊ if token.pos_=='NOUN':

chunk = ''

➋ for w in token.children:

➌ if w.pos_ == 'DET' or w.pos_ == 'ADJ':

chunk = chunk + w.text + ' '

➍ chunk = chunk + token.text

print(chunk)

遍历词元时，我们只拾取名词 ➊。接下来，在内层循环中，我们遍历名词的子项 ➋，只拾取作为名词短语的限定词或形容词（名词短语还可以包含其他词性，如副词） ➌。然后，我们将名词添加到短语中 ➍。因此，脚本的输出应该与前面的示例相同。

#### ***试试这个***

注意，用于修饰名词的词（限定词和形容词）总是名词的左侧句法子项。这使得在前面的代码中可以将 Token.children 替换为 Token.lefts，并在需要时移除检查子项是否为限定词或形容词。

重写之前的代码段，包含这里建议的更改。结果中的名词短语集应保持不变。

#### ***Span 对象***

Span 对象是从 Doc 对象中切片出来的。在前面的部分，你已经看到如何将它作为句子和名词短语的容器，分别来源于 doc.sents 和 doc.noun_chunks。

Span 对象的使用不仅限于作为句子或名词短语的容器。我们可以通过指定索引范围，使用它来容纳文档中任意一组相邻的标记，如下例所示：

>>> doc=nlp('I want a green apple.')

>>> doc[2:5]

一个绿色的苹果

Span 对象包含多个方法，其中最有趣的一个是 span.merge()，它允许我们将该词组合并为一个单一的标记，重新进行标记化。这在文本中包含由多个单词组成的名称时非常有用。

以下示例中的句子包含了由多个单词组成的两个地名（“Golden Gate Bridge”和“San Francisco”），我们可能希望将它们组合在一起。默认的标记化过程不会将这些多词地名识别为单一标记。看看当我们列出文本的标记时会发生什么：

>>> doc = nlp(u'The Golden Gate Bridge is an iconic landmark in San Francisco.')

>>> [doc[i] for i in range(len(doc))]

[The, Golden, Gate, Bridge, is, an, iconic, landmark, in, San, Francisco, .]

每个单词和标点符号都是一个独立的标记。

使用 span.merge() 方法，我们可以改变这个默认行为：

>>> span = doc[1:4]

>>> lem_id = doc.vocab.strings[span.text]

>>> span.merge(lemma = lem_id)

Golden Gate Bridge

在这个例子中，我们为“Golden Gate Bridge”这个词组创建了一个词形，然后将词形作为参数传递给 span.merge() 方法。（准确来说，我们传递的是通过 doc.vocab.string 属性获得的词形 ID。）

注意，span.merge() 方法默认并不会合并相应的词形。它在没有参数的情况下调用时，会将合并标记的词形设置为该词组第一个标记的词形。为了指定我们希望分配给合并标记的词形，我们将其作为词形参数传递给 span.merge()，如这里所示。

让我们检查词形还原器、词性标注器和依赖解析器是否能正确处理新创建的词形：

>>> for token in doc:

print(token.text, token.lemma_, token.pos_, token.dep_)

这应该产生以下输出：

The                the                DET   det

Golden Gate Bridge Golden Gate Bridge PROPN nsubj

is                 be                 VERB  ROOT

an                 an                 DET   det

iconic             iconic             ADJ   amod

landmark           landmark           NOUN  attr

in                 in                 ADP   prep

San                san                PROPN compound

Francisco          francisco          PROPN pobj

.                  .                  PUNCT punct

所有列出的属性已正确分配给“Golden Gate Bridge”标记。

#### ***试试看***

前面示例中的句子还包含 San Francisco，这是另一个你可能希望合并为单个标记的多词地名。为了实现这一点，按照前面代码片段中列出的操作，对“Golden Gate Bridge”跨度执行相同的操作。

在确定文档中“San Francisco”跨度的开始和结束位置时，别忘了新创建的“Golden Gate Bridge”标记右侧的标记索引已分别向下移动。

### **自定义文本处理管道**

在前面的章节中，你了解了 spaCy 的容器对象如何表示语言单元，如文本和单个标记，从而使你能够提取与它们相关的语言特征。现在，让我们来看一下 spaCy API 中创建这些容器并填充相关数据的对象。

这些对象被称为处理管道组件。如你已了解，默认情况下，管道集包括词性标注器、依赖解析器和实体识别器。你可以通过以下方式检查哪些管道组件可用于你的 nlp 对象：

>>> nlp.pipe_names

['tagger', 'parser', 'ner']

如下文所述，spaCy 允许你定制管道中的组件，以最适应你的需求。

#### ***禁用管道组件***

spaCy 允许你加载一组选定的管道组件，禁用那些不必要的组件。你可以在创建 nlp 对象时通过设置 disable 参数来做到这一点：

nlp = spacy.load('en', disable=['parser'])

在这个示例中，我们创建了一个没有依赖解析器的处理管道。如果我们在文本上调用此 nlp 实例，标记将不会得到依赖标签。以下示例清楚地说明了这一点：

>>> doc = nlp(u'I want a green apple.')

>>> for token in doc:

...   print(➊token.text, ➋token.pos_, ➌token.dep_)

I     代词

want 动词

a     限定词

green 形容词

apple 名词

.     标点符号

我们尝试打印出样本文本中每个标记的以下信息：文本内容 ➊、词性标注 ➋、和依赖标签 ➌。但是，依赖标签没有出现。

#### ***逐步加载模型***

你可以通过 spacy.load()一次性执行多个操作，该函数加载一个模型。例如，当你调用以下代码时：

nlp = spacy.load('en')

spaCy 在幕后执行以下步骤：

1.  查看要加载的模型名称时，spaCy 会识别应该初始化哪个 Language 类。在此示例中，spaCy 创建了一个包含共享词汇和其他语言数据的英语类实例。

1.  spaCy 遍历处理管道的名称，创建相应的组件，并将其添加到处理管道中。

1.  spaCy 从磁盘加载模型数据，并将其提供给 Language 类实例。

这些实现细节被 spacy.load()隐藏，在大多数情况下，它为你节省了精力和时间。但有时，你可能需要显式地实现这些步骤，以便更精细地控制过程。例如，你可能需要将自定义组件加载到处理管道中。该组件可以打印关于管道中 Doc 对象的一些信息，如标记数或某些词性是否存在。

如常，更多的精细控制要求你提供更多的信息。首先，不是指定快捷方式，而是需要获取实际的模型名称，这样你才能获取到模型包的路径。

你可以通过以下方式识别模型的完整名称：

>>> print(nlp.meta['lang'] + '_' + nlp.meta['name'])

en_core_web_sm

该代码中使用的 nlp.meta 属性是一个字典，包含加载的模型的元数据。在这个示例中，你需要的是模型的语言和模型的名称。

现在你已经知道了模型的名称，你可以通过使用 get_package_path 工具函数来查找它在系统中的位置：

>>> from spacy import util

>>> util.get_package_path('en_core_web_sm')

PosixPath('/usr/local/lib/python3.5/site-packages/en_core_web_sm')

代码中指定的路径在你的机器上可能不同，具体取决于你的 Python 安装目录。不管怎样，这不是完整路径，你需要在其后追加一个文件夹。该文件夹的名称由模型名称和模型版本组成。（这是模型包所在的位置。）你可以按以下方式确定其名称：

>>> print(nlp.meta['lang'] + '_' + nlp.meta['name'] + '-' + nlp.

meta['version'])

en_core_web_sm-2.0.0

你还可能想查看与模型一起使用的管道组件列表。（了解模型支持哪些组件非常重要，因此可以将它们加载到管道中。）你可以通过 nlp.meta 属性的'pipeline'字段获取这些信息，如下所示（或者通过 nlp.pipe_names 属性，它在“定制文本处理管道”一节的第 37 页中介绍）：

>>> nlp.meta['pipeline']

['tagger', 'parser', 'ner']

有了这些信息，我们可以创建一个脚本，按照本节开始时提供的步骤进行操作：

>>> lang = 'en'

>>> pipeline = ['tagger', 'parser', 'ner']

>>> model_data_path = '/usr/local/lib/python3.5/site-packages/en_core_web_sm/

en_core_web_sm-2.0.0'

➊ >>> lang_cls = spacy.util.get_lang_class(lang)

>>> nlp = lang_cls()

➋ >>> for name in pipeline:

➌ ...   component = nlp.create_pipe(name)

➍ ...   nlp.add_pipe(component)

➎ >>> nlp.from_disk(model_data_path)

在这个脚本中，我们使用 spacy.util.get_lang_class() ➊ 来加载一个 Language 类。我们加载哪个类取决于作为参数指定的两位字母语言代码。在这个例子中，我们加载英语。接下来，在一个循环 ➋ 中，我们创建 ➌ 并添加 ➍ 处理管道组件到处理管道中。然后我们从磁盘加载一个模型，指定用于你机器上的路径 ➎。

查看此脚本中的代码，可能会觉得一旦我们将管道组件添加到处理管道中，它们就会变得可用。实际上，在加载模型数据之前，我们无法使用它们，因此，如果省略脚本中的最后一行代码，我们甚至无法使用这个 nlp 实例创建一个 Doc 对象。

#### ***自定义处理管道组件***

通过自定义管道组件，你可以最好地满足应用程序的需求。例如，假设你希望你的模型的命名实体识别系统将“Festy”识别为一个市区。默认情况下，它会将其识别为一个组织，如下所示：

>>> doc = nlp(u'I need a taxi to Festy.')

>>> for ent in doc.ents:

...  print(ent.text, ent.label_)

Festy ORG

标签 ORG 代表公司、机构和其他组织。但你希望让实体识别器将其分类为 DISTRICT 类型的实体。

实体识别组件在 spaCy API 中实现为 EntityRecognizer 类。使用该类的方法，你可以初始化一个 ner 实例，然后将其应用于文本。在大多数情况下，你不需要显式地执行这些操作；当你创建一个 nlp 对象并分别创建一个 Doc 对象时，spaCy 会自动为你处理这些操作。

但当你希望用自己的示例更新现有模型的命名实体识别系统时，你需要显式地使用一些 ner 对象的方法。

在以下示例中，你首先需要将一个名为 DISTRICT 的新标签添加到支持的实体类型列表中。然后你需要创建一个训练示例，实体识别器将通过它学习将 DISTRICT 标签应用于哪些内容。准备步骤的最简单实现可能如下所示：

LABEL = 'DISTRICT'

TRAIN_DATA = [

➊ ('We need to deliver it to Festy.', {

➋ 'entities': [(25, 30, 'DISTRICT')]

}),

➌ ('I like red oranges', {

'entities': []

})

]

为了简化起见，这个训练集仅包含两个训练样本（通常，你需要提供更多样本）。每个训练样本包括一个句子，该句子可能包含或不包含需要分配新实体标签的实体（或多个实体）➊。如果样本中有实体，你需要指定它的起始和结束位置➋。训练集中的第二个句子根本不包含“Festy”这个词➌。这是由于训练过程的组织方式。第十章更详细地介绍了这个过程的细节。

下一步是向实体识别器添加一个新的实体标签 DISTRICT：但在此之前，你必须获取 ner 管道组件的实例。你可以通过以下方式实现：

ner = nlp.get_pipe('ner')

一旦你拥有一个 ner 对象，你可以使用 ner.add_label() 方法向其中添加一个新的标签，如下所示：

ner.add_label(LABEL)

在开始训练实体识别器之前，你还需要采取一个动作，即禁用其他管道，以确保在训练过程中只有实体识别器会被更新：

nlp.disable_pipes('tagger')

nlp.disable_pipes('parser')

然后，你可以使用之前在本节中创建的 TRAIN_DATA 列表中的训练样本开始训练实体识别器：

optimizer = nlp.entity.create_optimizer()

import random

for i in range(25):

random.shuffle(TRAIN_DATA)

for text, annotations in TRAIN_DATA:

nlp.update([text], [annotations], sgd=optimizer)

在训练过程中，示例会以循环、随机顺序展示给模型，以有效更新底层模型的数据，并避免根据训练示例的顺序进行任何概括。执行过程可能需要一些时间。

一旦前面的代码成功完成，你就可以测试更新后的优化器如何识别 token Festy：

>>> doc = nlp(u'我需要一辆去 Festy 的出租车。')

>>> for ent in doc.ents:

... print(ent.text, ent.label_)

...

Festy DISTRICT

根据输出，它正常工作。

请记住，当你关闭这个 Python 解释器会话时，刚才所做的更新将会丢失。为了解决这个问题，Pipe 类——EntityRecognizer 类及其他管道组件类的父类——提供了 to_disk() 方法，允许你将管道序列化到磁盘：

>>> ner.to_disk('/usr/to/ner')

现在，你可以使用 from_disk() 方法将更新后的组件加载到新会话中。为了确保它正常工作，关闭当前的解释器会话，启动一个新会话，然后运行以下代码：

>>> import spacy

>>> from spacy.pipeline import EntityRecognizer

➊ >>> nlp = spacy.load('en', disable=['ner'])

➋ >>> ner = EntityRecognizer(nlp.vocab)

➌ >>> ner.from_disk('/usr/to/ner')

➍ >>> nlp.add_pipe(ner)

你加载了模型，禁用了它的默认 ner 组件 ➊。接着，你创建一个新的 ner 实例 ➋，然后将其从磁盘加载数据 ➌。然后你将 ner 组件添加到处理管道中 ➍。

现在你可以像这样测试它：

>>> doc = nlp(u'我们需要把它送到 Festy。')

>>> for ent in doc.ents:

... print(ent.text, ent.label_)

Festy DISTRICT

如你所见，实体识别器正确地标记了名字 Festy。

尽管我只向你展示了如何定制命名实体识别器，但你也可以以类似的方式定制其他管道组件。

### **使用 spaCy 的 C 层数据结构**

即使使用 spaCy，涉及处理大量文本的 NLP 操作也可能非常耗时。例如，你可能需要编写一个最适合某个名词的形容词列表，为此你需要检查大量的文本。如果处理速度对你的应用至关重要，spaCy 允许你利用 Cython 的 C 级数据结构和接口。Cython 是 spaCy 编写的语言之一（另一个是 Python）。由于它是 Python 的超集，Cython 几乎认为所有 Python 代码都是有效的 Cython 代码。除了 Python 的功能，Cython 还允许你原生调用 C 函数并声明快速的 C 类型，从而使编译器能够生成非常高效的代码。你可能希望使用 Cython 来加速耗时的文本处理操作。

spaCy 的核心数据结构是作为 Cython 对象实现的，而 spaCy 的公共 API 允许你访问这些结构。有关详细信息，请参考文档中的 Cython 架构页面，链接为 *[`spacy.io/api/cython/`](https://spacy.io/api/cython/)*。

#### ***它是如何工作的***

要将 Cython 代码与 spaCy 一起使用，你必须将其转换为 Python 扩展模块，然后可以将其导入到程序中，如 图 3-3 所示。

![image](img/fig3-3.jpg)

*图 3-3：从 Cython 脚本构建 Python 扩展模块*

你可以通过将 Cython 代码保存为 *.pyx* 文件，然后运行 *setup.py* Python 脚本来完成这项工作，该脚本首先将 Cython 代码转换为相应的 C 或 C++ 代码，然后调用 C 或 C++ 编译器。该脚本生成 Python 扩展模块。

#### ***准备你的工作环境并获取文本文件***

在开始构建 Cython 代码之前，你需要在你的机器上安装 Cython 并获得一个大型文本文件来进行处理。

使用 pip 在你的机器上安装 Cython：

pip install Cython

接下来，为了模拟一个耗时的任务并测量性能，你需要一个大型文本文件。你可以使用 *Wikipedia dump file*，它包含一组以 XML 格式封装的页面。Wikipedia dump 文件可以从 *[`dumps.wikimedia.org/enwiki/latest/`](https://dumps.wikimedia.org/enwiki/latest/)* 下载。向下滚动找到 *enwiki-latest-pages-articles*.xml-*.bz2* 文件，并选择一个足够大的文件进行测试。但不要选择太大的文件，除非你想花几个小时等待机器完成测试代码。一个 10–100MB 的 dump 文件应该合适。

下载文件后，你可以使用 *gensim.corpora.wikicorpus*（*[`radimrehurek.com/gensim/corpora/wikicorpus.html`](https://radimrehurek.com/gensim/corpora/wikicorpus.html)*）等工具从中提取原始文本，它专门用于从 Wikipedia 数据库 dump 中构建文本语料库。

#### ***你的 Cython 脚本***

现在让我们编写一个分析文本文件的 Cython 脚本。为了简化，假设您只想计算提交文本中个人代词的数量。这意味着您需要计算那些被分配了 PRP 词性标签的 token 数量。

**警告**

*如文档中所述，面向 Cython 的 C 层方法优先考虑速度而非安全性。代码中的错误可能导致执行突然崩溃。*

在您本地文件系统中的一个目录中，创建一个名为 *spacytext.pyx* 的文件，并插入以下代码：

从 cymem.cymem cimport Pool

从 spacy.tokens.doc cimport Doc

从 spacy.structs cimport TokenC

从 spacy.typedefs cimport hash_t

➊ cdef struct DocStruct：

TokenC* c

int length

➋ cdef int counter(DocStruct* doc, hash_t tag)：

cdef int cnt = 0

对于 c 在 doc.c[:doc.length] 中：

如果 c.tag == tag：

cnt += 1

返回 cnt

➌ cpdef main(Doc mydoc)：

cdef int cnt

cdef Pool mem = Pool()

cdef DocStruct* doc_ptr = <DocStruct*>mem.alloc(1, sizeof(DocStruct))

doc_ptr.c = mydoc.c

doc_ptr.length = mydoc.length

tag = mydoc.vocab.strings.add('PRP')

cnt = counter(doc_ptr, tag)

打印(doc_ptr.length)

打印(cnt)

我们从一组 cimport 语句开始，以导入所需的 Cython 模块，主要来自 spaCy 库。

然后我们定义 Cython 结构体 DocStruct，作为处理文本的容器，以及 TokenC* 变量 ➊，它是指向 TokenC 结构体的指针，spaCy 使用它作为 Token 对象的数据容器。

接下来，我们定义一个 Cython 函数 counter ➋，用于计算文本中个人代词的数量。

**注意**

*Cdef* 函数 *在导入该模块的 Python 代码中不可用。如果您想创建一个同时对 Python 可见并能利用 C 层数据结构和接口的函数，您需要将该函数声明为* cpdef。

最后，我们定义一个 cpdef Cython/Python 主函数 ➌，可以在 Python 中使用。

#### ***构建 Cython 模块***

与 Python 不同，您必须编译 Cython 代码。您可以通过几种方式进行编译，最好的方法是编写一个 distutils/setuptools *setup.py* Python 脚本。创建一个与 Cython 脚本位于同一目录下的 *setup.py* 文件。您的 *setup.py* 文件应包含以下代码：

从 distutils.core 导入 setup

从 Cython.Build 导入 cythonize

➊ import numpy

setup(name='spacy text app'，

➋ ext_modules=cythonize("spacytext.pyx", language="c++")，

➌ include_dirs=[numpy.get_include()]

)

这是一个常规的 distutils/setuptools *setup.py* 脚本，除了与我们正在处理的示例相关的两个补充项。首先，我们导入 numpy ➊，然后显式指定在哪里找到库的 *.h* 文件 ➌。这样做是为了避免在某些系统中出现 *numpy/arrayobject.h* 编译错误。我们使用另一个 setup 选项，language = "c++" ➋，指示 setup 过程使用 C++ 编译器，而不是执行默认的 C 编译。

现在我们已经有了设置脚本，你可以开始构建你的 Cython 代码。你可以通过系统终端来完成此操作，方法如下：

python setup.py build_ext --inplace

在编译过程中，会显示一堆信息。其中一些可能是警告，但它们通常不是关键性的。例如，你可能会看到这样的信息，但这对过程并不关键：

#warning "使用已弃用的 NumPy API ..."

#### ***测试模块***

编译成功后，spacytext 模块将被添加到你的 Python 环境中。要测试新创建的模块，打开一个 Python 会话并运行以下命令：

>>> from spacytext import main

如果没有显示错误，你可以输入以下命令（假设你的文本数据位于 *test.txt* 文件中）：

>>> import spacy

>>> nlp = spacy.load('en')

➊ >>> f= open("test.txt","rb")

>>> contents =f.read()

➋ >>> doc = nlp(contents[:100000].decode('utf8'))

➌ >>> main(doc)

21498

216

你以二进制模式打开包含文本数据的文件，以获取一个字节对象 ➊。如果文件过大，你可以在创建 Doc 对象 ➋ 时只提取部分内容。一旦创建了 Doc 对象，你可以测试刚刚用 Cython 创建的 spacytext 模块，调用其 main() 函数 ➌。

spacytext.main() 函数生成的输出中的第一个数字表示提交文本中找到的总标记数。第二个数字表示在同一文本中找到的个人代词的数量。

### **总结**

在本章中，你了解了 spaCy 中最重要的容器对象。你还学习了如何自定义文本处理管道，并使用 spaCy 的 C 级数据结构和接口进行 Cython 开发。
