## **12

实现网络数据和处理图像**

![Image](../Images/comm1.jpg)

现实中的聊天机器人应该能够应对各种输入，例如用户提出的关于不熟悉主题的问题，甚至是通过消息应用程序发送的图片。例如，聊天机器人应用程序的用户不仅可以发送文本消息，还可以发送照片，而机器人应该能够对这两者做出适当的反应。

本章提供了一些使用 Python 的 AI 生态系统中其他库的示例，用于开发机器人应用程序。首先，你将结合 spaCy 和维基百科来查找用户问题中提取的关键词的信息。接下来，你将借助 Clarifai（一种图像和视频识别工具）来获取提交图像的描述性标签，从而让你的应用程序能够解读视觉内容。

然后你会将所有组件整合在一起，构建一个 Telegram 机器人，通过从维基百科提取信息来生成对文本和图像的相关回应。

### **工作原理**

[图 12-1](../Text/ch12.xhtml#ch12fig01)展示了我们将在本章中构建的机器人的示意图。这个机器人旨在理解文本信息和图片，并用来自维基百科的文本进行回应。

![image](../Images/fig12-1.jpg)

*图 12-1：一个能够处理文本消息和图片的机器人的工作原理*

使用这个机器人，用户可以发布文本消息或图片。如果发布的是图片，机器人会将其发送到图像识别工具进行处理。该工具会返回一份以描述性标签形式呈现的图片文字描述。如果发布的是文本消息，机器人会使用像 spaCy 这样的自然语言处理工具从中提取关键词或关键短语。然后，机器人使用这个标签或关键短语来查找维基百科（或网络其他地方）上的最相关内容，并返回一部分给用户。你可以在你设计的聊天机器人中使用这一场景来进行各种主题的对话，供娱乐、学习或个人使用。

### **让你的机器人从维基百科中找到问题的答案**

让我们从讨论一些可以在你的机器人中实现的技术开始，这些技术能让机器人解读各种各样的文本信息。前几章讨论了用于商业目的的机器人通常会向用户索要某些信息，然后用回答来填充订单或预定请求。相比之下，旨在进行非正式对话的机器人应该能够回答各种各样的用户问题。

帮助聊天机器人回答用户问题的一种方法是从问题中选择一个关键词或关键短语，这可以提供有关答案中应包含哪些信息的线索。一旦你获得了这个关键词或关键短语，就可以使用像维基百科的 Python API 这样的工具来搜索答案。维基百科的 API 让你能够以编程方式访问和解析维基百科内容，进行关键词搜索，从而从最相关的维基百科文章中获取内容。接下来的章节将描述如何做到这一点。

但在继续之前，请确保你使用的是最新的spaCy模型，因为新版本的依赖解析准确度更高。你可以使用以下命令检查当前模型的版本：

nlp.meta['version']

然后访问*[https://explosion.ai/demos/displacy/](https://explosion.ai/demos/displacy/)*演示页面（在[第7章](../Text/ch07.xhtml#ch07)中讨论过），查看最新稳定版的spaCy模型。或者，你也可以访问spaCy的文档页面*[https://spacy.io/usage/](https://spacy.io/usage/)*来查看最新版本的spaCy。spaCy及其模型遵循相同的版本管理规则。根据这些信息，你可能想要更新你当前使用的模型。有关如何下载和安装spaCy模型的详细信息，请参见[第2章](../Text/ch02.xhtml#ch02)。

#### ***确定问题的主题***

在试图确定说话者想要问什么时，问题中的某些词比其他词更为重要。有时，观察问题中的一个词就足够了，比如跟在介词后面的名词。例如，用户可能会使用以下任何问题来要求机器人查找关于犀牛的信息：

你听说过犀牛吗？你了解犀牛吗？你能告诉我关于犀牛的什么信息？

让我们看看这些句子的依赖解析可能是什么样子。[图12-2](../Text/ch12.xhtml#ch12fig02)展示了第一个句子解析的图形表示。

![image](../Images/fig12-2.jpg)

*图12-2：包含介词宾语的句子的依赖解析*

解析表明，在这种类型的问题中，通过提取介词的宾语，可以得到“犀牛”一词。“犀牛”将是问题中最有助于找到答案的词。以下代码片段展示了如何提取问题中第一个出现的介词宾语：

doc = nlp(u"你听说过犀牛吗？")

for t in doc:

if t.dep_ == 'pobj' and (t.pos_ == 'NOUN' or t.pos_ == 'PROPN'):

phrase = (' '.join([child.text for child in t.lefts]) + ' ' + t.text).lstrip()

break

在代码中，我们还会提取介词宾语的左侧子节点，因为宾语可能有重要的修饰词，如以下示例：“你能说些什么关于野生山羊？”对于这个问题，代码应该将“野生山羊”赋值给phrase变量。

注意在末尾使用了break语句，它确保只会提取句子中第一个介词宾语。例如，在句子“告诉我关于美利坚合众国的事”中，短语“美利坚合众国”会被提取，但“美国”则不会。

但这并不总是理想的行为。如果用户问：“告诉我天空的颜色。”怎么办？这时我们需要应用更复杂的逻辑。特别是，我们可能希望获取紧跟第一个介词对象后的任何介词对象，前提是后者依赖于前者。

这是你如何实现这一逻辑的方法：

doc = nlp(u"告诉我天空的颜色。")

for t in doc:

if t.dep_ == 'pobj' and (t.pos_ == 'NOUN' or t.pos_ == 'PROPN'):

phrase = (' '.join([child.text for child in t.lefts]) + ' ' + t.text).lstrip()

if bool([prep for prep in t.rights if prep.dep_ == 'prep']):

prep = list(t.rights)[0]

pobj = list(prep.children)[0]

phrase = phrase + ' ' + prep.text + ' ' + pobj.text

break

请注意，如果第一个介词对象的依赖项存在，代码将处理作为第一个介词对象依赖项的介词对象。否则，代码将与之前展示的代码相同。

现在让我们来看一下以下示例中的另一种问题类型，在这些示例中，动词及其主语为回答问题提供了最佳信息：

你知道大象吃什么吗？告诉我海豚如何睡觉。什么是API？

[图12-3](../Text/ch12.xhtml#ch12fig03) 展示了其中一个句子的依赖解析可能是什么样子。

![image](../Images/fig12-3.jpg)

*图12-3：句子的依赖解析，其中主语/动词对是发现说话者想知道什么的最有信息量的元素*

通过查看图中的解析，可以注意到，出现在句子末尾的主语/动词对在确定说话者询问的内容时是最具信息量的。程序化地，你可以通过以下代码从句子中提取主语和动词对：

doc = nlp(u"你知道大象吃什么吗？")

for t in reversed(doc):

if t.dep_ == 'nsubj' and (t.pos_ == 'NOUN' or t.pos_ == 'PROPN'):

phrase = t.text + ' ' + t.head.text

break

在检查这段代码时，请注意我们使用 Python 的 `reversed()` 函数从句子的末尾向后循环。原因是我们需要获取句子中的最后一个主语/动词对，如这个例子：“你知道大象吃什么吗？”在这个句子中，我们关注的是“elephant eats”短语，而不是“you know”，后者也是一个主语/动词对。

此外，在某些问题中，句子中的最后一个名词是动词的直接宾语，这对于确定问题的内容至关重要，如以下示例所示：

如何喂猫？

在这个句子中，提取直接宾语“cat”并不足够，因为我们还需要动词“feed”来理解问题。理想情况下，我们会生成关键词短语“feeding a cat”。也就是说，我们通过添加“-ing”将动词的不定式形式“to”替换为动名词，从而优化关键词以进行互联网搜索。[图12-4](../Text/ch12.xhtml#ch12fig04)展示了这个句子的依存句法分析。

![image](../Images/fig12-4.jpg)

*图12-4：包含动词/直接宾语对的句子依存句法分析作为最有信息量的短语*

这种句法分析表明，提取所需短语非常简单，因为直接宾语及其及物动词之间有直接的联系。

这里讨论的提取代码实现可能如下所示：

doc = nlp(u"How to feed a cat?")

for t in reversed(doc):

if t.dep_ == 'dobj' and (t.pos_ == 'NOUN' or t.pos_ == 'PROPN'):

phrase = t.head.lemma_ + 'ing' + ' ' + t.text

break

在这种情况下，我们再次从句子的末尾向前循环。为了理解为什么这么做，请考虑以下句子：“Tell me something about how to feed a cat。”它包含两个动词/直接宾语对，但我们只对句子末尾的那一个感兴趣。

#### ***试试这个***

修改上一节中提取短语“elephant eats”的代码，使得从句子中提取的关键词包含可能修饰主语的词，排除可能的限定词。例如，在句子“Tell me how a female cheetah hunts”中，您的脚本应返回“female cheetah hunts”，并从名词短语中去除限定词“a”。查看[图12-2](../Text/ch12.xhtml#ch12fig02)中代码的示例，您会看到如何为提取的介词宾语获取修饰词。

此外，添加检查以查看提取的短语中包含的动词是否有直接宾语，如果有，将直接宾语附加到关键词中。例如，问题“Do you know how many eggs a sea turtle lays?”应该生成以下关键词：“sea turtle lays eggs”。

#### ***使用维基百科回答用户问题***

现在，您有了一个关键词，可以帮助您找到生成相关回复所需的信息，接下来需要获取这些信息。一个机器人可以从多个地方获取用户问题的答案，选择哪个源取决于应用程序，但维基百科是一个不错的起点。维基百科 Python 库（*[https://pypi.org/project/wikipedia/](https://pypi.org/project/wikipedia/)*) 允许您在 Python 代码中访问维基百科文章。

您可以通过以下方式使用 pip 安装该库：

pip install wikipedia

为了测试新安装的库，使用以下脚本，它依赖前一节的代码片段从提交的句子中提取关键字。然后，它将这个关键字用作维基百科的搜索词。

导入 spacy

导入 wikipedia

nlp = spacy.load('en')

doc = nlp(u"你知道关于犀牛的哪些信息？")

对于 t 在 doc 中：

如果 t.dep_ == 'pobj' 且（t.pos_ == 'NOUN' 或 t.pos_ == 'PROPN'）：

➊ phrase = (' '.join([child.text for child in t.lefts]) + ' ' + t.text)。

lstrip()

break

➋ wiki_resp = wikipedia.page(phrase)

print("文章标题: ", wiki_resp.title)

print("文章链接: ", wiki_resp.url)

print("文章摘要: ", wikipedia.summary(phrase, sentences=1))

在这个脚本中，我们从提交的句子 ➊ 中提取关键字或关键短语，并将其发送到 wikipedia.page() 函数，这个函数返回与给定关键字最相关的文章 ➋。然后我们简单地打印出文章的标题、网址和第一句话。

这个脚本生成的输出应该如下所示：

文章标题: Rhinoceros

文章链接: https://en.wikipedia.org/wiki/Rhinoceros

文章摘要: 犀牛（来自希腊语 rhinokero–s，意为“鼻角”，其中 rhis 意为“鼻子”，keras 意为“角”），常简称为犀牛，是一种...

意为“鼻子”，keras，意为“角”），通常简称为犀牛，是一种...

#### ***试试看***

增强前一节的脚本，使其能够“查看”第一个介词对象的子节点以及依赖的介词对象。例如，在问题“你听说过黄番茄炒蛋吗？”中，它应该提取关键短语“黄番茄炒蛋”。

### **对聊天中发送的图片进行反应**

除了文本消息外，消息应用的用户通常还会发布图片。其他人通常会就图片中的内容发表评论。例如，某用户发布了一张葡萄的照片，另一个用户评论道：“我喜欢水果。它含有丰富的纤维和维生素。”你如何教一个机器人做同样的事情？一种方法是为图片生成描述标签，机器人可以利用这些标签进行处理。这时你需要一个图像识别工具，比如 Clarifai，它提供了经过不同领域照片训练的内置模型，如服装、旅行或名人。

Clarifai 允许机器人为提交的照片获取一组类别，从而使机器人能够猜测图片中展示的内容。你可以通过两个步骤来获取照片的有用类别。首先，你使用 Clarifai 的通用图像识别模型来获取描述标签（带有概率的对象），这些标签可以给你一个大致的图像展示内容。例如，“no person”标签的出现表示照片中没有人。

其次，在检查标签之后，你可以对同一张照片应用更具体的模型，例如 Clarifai 的食品或服装模型。两者分别经过训练，能够识别与食物和时尚相关的物品。这一次，你将获得另一个更为详细的标签集，从而更好地了解照片的内容。如需查看 Clarifai 所有的图像识别模型，请访问其模型页面 *[https://www.clarifai.com/models/](https://www.clarifai.com/models/)*。

#### ***使用 Clarifai 生成图像的描述性标签***

Clarifai 提供了一个 Python 客户端，用于与其识别 API 进行交互。你可以使用 pip 安装最新的稳定版本：

pip install clarifai --upgrade

在你开始使用 Clarifai 库之前，必须先通过创建账户并点击 **GET API KEY** 按钮来获得 API 密钥，地址是 *[https://www.clarifai.com/](https://www.clarifai.com/)*。

一旦你获得了密钥，就可以测试 Clarifai 库。以下简单的脚本将一张图像传递给 Clarifai 模型，并打印出表示图像可能所属类别的标签列表：

from clarifai.rest import ClarifaiApp, client, Image

app = ClarifaiApp(api_key='YOUR_API_KEY')

➊ model = app.public_models.general_model

filename = '/your_path/grape.jpg'

➋ image = Image(file_obj=open(filename, 'rb'))

response = model.predict([image])

➌ concepts = response['outputs'][0]['data']['concepts']

for concept in concepts:

print(concept['name'], concept['value'])

在本示例中，我们调用了 Clarifai 的预测 API，并使用了通用模型 ➊。Clarifai 仅接受像素作为输入，因此请确保以 'rb' 模式打开图像文件 ➋，该模式以二进制格式打开文件进行读取。预测 API 为提交的照片生成了一系列描述性标签，如 fruit、grape、health 等 ➌，从而使得代码能够“理解”其显示的内容。

示例中使用的 *grape.jpg* 文件包含 [图 12-5](../Text/ch12.xhtml#ch12fig05) 所示的照片。

![image](../Images/fig12-5.jpg)

*图 12-5：提交给 Clarifai 的照片（如前面的脚本所示）*

脚本为照片生成的概念列表应如下所示：

no person 0.9968359470367432

wine 0.9812138080596924

fruit 0.9805494546890259

juicy 0.9788177013397217

health 0.9755384922027588

grow 0.9669009447097778

grape 0.9660607576370239

...

每个条目代表一个类别以及图像属于该类别的概率。因此，列表中的第一个标签告诉我们，提交的照片中没有人，概率为 0.99。注意，并非所有标签都会直接描述所描绘的内容。例如，标签“wine”可能出现在这里，因为酒是由葡萄酿成的。列表中间接标签的存在，给你的机器人提供了更多解读图像的选项。

#### ***使用标签生成图像的文本响应***

现在你知道如何获取图像的描述性标签，如何利用这些标签来回应图像？或者你如何从生成的标签列表中选择最重要的标签？考虑以下一般性问题：

+   你可能只想考虑那些具有较高可能性的标签。为此，你可以为标签选择一个可能性的阈值。例如，只考虑前五个或十个标签。

+   你可能只选择那些在当前聊天上下文中的标签。[第11章](../Text/ch11.xhtml#ch11)展示了如何在 Telegram 机器人中使用 context.user_data 字典来保持当前聊天的上下文。

+   你可能会遍历生成的标签，寻找特定的标签。例如，你可以搜索标签“fruit”或“health”以确定是否应该继续讨论这个话题。

下一节讨论的机器人将实现第三个选项。

### **将所有部分组合成一个 Telegram 机器人**

在本章的其余部分，我们将构建一个使用 Wikipedia API 和 Clarifai API 的 Telegram 聊天机器人。这个机器人将能够智能地响应食物的文本和图像。有关如何在 Telegram 中创建新机器人的详细信息，请参考 [第11章](../Text/ch11.xhtml#ch11)。

#### ***导入库***

代码的导入部分必须包含我们将在机器人代码中使用的所有库。在这个例子中，我们包括了访问 Telegram Bot API、Wikipedia API、Clarifai API 和 spaCy 所需的库。

导入 spacy

导入 wikipedia

从 telegram.ext 导入 Updater, CommandHandler, MessageHandler, Filters

从 clarifai.rest 导入 ClarifaiApp, Image

如果你已经按照本章和 [第11章](../Text/ch11.xhtml#ch11) 中提供的说明操作，那么所有这些库应该都可以在你的系统上使用。

#### ***编写辅助函数***

接下来，我们需要实现将在机器人回调函数中调用的辅助函数。keyphrase() 函数将一个句子作为 Doc 对象，并试图从中提取最具信息量的单词或短语，正如之前在“[确定问题的内容](../Text/ch12.xhtml#lev159)”一节中讨论的那样。以下实现使用了你在该节中看到的代码片段，并对其进行了调整，以便我们可以在一个函数中使用它们：

定义 keyphrase(doc):

对 doc 中的每个 t 进行迭代：

如果 t.dep_ == 'pobj' 且 (t.pos_ == 'NOUN' 或 t.pos_ == 'PROPN'):

返回 (' '.join([child.text for child in t.lefts]) + ' ' + t.text)

lstrip()

对 doc 中的每个 t 进行逆序迭代：

如果 t.dep_ == 'nsubj' 且 (t.pos_ == 'NOUN' 或 t.pos_ == 'PROPN'):

返回 t.text + ' ' + t.head.text

对 doc 中的每个 t 进行逆序迭代：

如果 t.dep_ == 'dobj' 且 (t.pos_ == 'NOUN' 或 t.pos_ == 'PROPN'):

返回 t.head.text + 'ing' + ' ' + t.text

返回 False

请注意，在此代码中，条件按照优先级顺序排列。因此，如果找到介词的宾语，我们将提取它并退出，而不再检查其他条件。当然，某些复杂的问题可能会符合多个条件，但检查这些条件会使函数实现变得复杂。

像 keyphrase() 函数一样，photo_tags() 函数也应该确定用户输入的最具描述性的词汇。但与 keyphrase() 不同，它分析的是一张照片。它借助 Clarifai 执行分析，Clarifai 会为提交的照片生成一组描述性标签。此实现仅使用两个 Clarifai 模型：通用模型和食品模型。

def photo_tags(filename):

app = ClarifaiApp(api_key=CLARIFAI_API_KEY)

model = app.public_models.general_model

image = Image(file_obj=open(filename, 'rb'))

response = model.predict([image])

concepts = response['outputs'][0]['data']['concepts']

for concept in concepts:

if concept['name'] == 'food':

food_model = app.public_models.food_model

result = food_model.predict([image])

first_concept = result['outputs'][0]['data']['concepts'][0]['name']

return first_concept

return response['outputs'][0]['data']['concepts'][1]['name']

这段代码首先应用通用模型。如果在生成的标签列表中找到“food”标签，它将应用食品模型，以获得图像中食品项的更多描述性标签。此实现将只使用第一个标签作为搜索的关键字。

现在，我们已经通过 keyphrase() 函数或 photo_tags() 函数确定了关键字或关键短语，我们需要获取一条与这个关键字或关键短语密切相关的信息。以下的 wiki() 函数可以完成这个任务：

def wiki(concept):

nlp = spacy.load('en')

wiki_resp = wikipedia.page(concept)

doc = nlp(wiki_resp.content)

if len(concept.split()) == 1:

for sent in doc.sents:

for t in sent:

if t.text == concept and t.dep_ == 'dobj':

return sent.text

return list(doc.sents)[0].text

我们在这里使用的算法会搜索检索到的内容中的一个句子，这个句子包含关键字作为直接宾语。

但这种简单的实现只能智能地处理单个词汇的输入。当提交一个词汇时，我们在这里使用的算法只是提取出 Wikipedia 文章中找到的第一个句子。

#### ***编写回调函数和 main() 函数***

接下来，我们添加机器人的回调函数。start() 函数简单地响应 /start 命令，向用户发送问候。

def start(update, context):

update.message.reply_text('你好！我是一个对话机器人，问我点什么吧。')

text_msg() 函数是机器人用户文本消息处理器的回调函数。

def text_msg(update, context):

msg = update.message.text

nlp = spacy.load('en')

doc = nlp(msg)

concept = keyphrase(doc)

if concept != False:

update.message.reply_text(wiki(concept))

else:

update.message.reply_text('请重新表述您的问题。')

首先，我们将 spaCy 的处理管道应用于消息，将其转换为 Doc 对象。然后，我们将 Doc 传递给之前讨论的 keyphrase() 函数，从消息中提取关键词或关键词短语。返回的关键词或关键词短语将被传递给 wiki() 函数，以获取相关信息，这在此实现中应为一句话。

以下代码中展示的 photo() 函数是机器人处理用户提交照片的回调函数：

def photo(update, context):

photo_file = update.message.photo[-1].get_file()

filename = '{}.jpg'.format(photo_file.file_id)

photo_file.download(filename)

concept = photo_tags(filename)

update.message.reply_text(wiki(concept))

该函数将接收到的图片作为文件提取，并发送给之前讨论的助手函数以进行进一步处理，详情请参见“[编写助手函数](../Text/ch12.xhtml#lev168)”以及 [第 178 页](../Text/ch12.xhtml#page_178)。

最后，我们添加了 main() 函数，在该函数中注册了文本消息和照片的处理程序。

def main():

updater = Updater("YOUR_TOKEN", use_context=True)

disp = updater.dispatcher

disp.add_handler(CommandHandler("start", start))

disp.add_handler(MessageHandler(Filters.text, text_msg))

disp.add_handler(MessageHandler(Filters.photo, photo))

updater.start_polling()

updater.idle()

if __name__ == '__main__':

main()

这个 Telegram 机器人的 main() 函数非常简洁。我们创建了 Updater 并传递了机器人的令牌。然后，我们获取调度器来注册处理程序。在这个例子中，我们只注册了三个处理程序。第一个是 /start 命令的处理程序，第二个处理来自用户的文本消息，第三个处理用户提交的照片。注册完处理程序后，我们通过调用 updater.start_polling() 启动机器人，然后调用 updater.idle() 来阻塞脚本，等待用户消息或退出快捷键（CTRL-C）。

#### ***测试机器人***

现在我们已经创建了机器人，接下来是测试它。您可以在智能手机或计算机上测试它。在智能手机上，打开 Telegram 应用，搜索您的机器人名称并加上 @ 符号，然后输入 /start 命令开始聊天。在计算机上，使用 Telegram Web 访问 *[https://web.telegram.org](https://web.telegram.org)*。

在接收到机器人的问候后，向其发送一个简单的请求，例如“告诉我关于水果的事情”。机器人应该回应一段从相关 Wikipedia 文章中提取出的句子。为简便起见，选择一个使用句子中的直接宾语（例如这里的“水果”）作为关键词的句子。

您还可以提交一张照片，以检查机器人会给出什么评论作为回应。 [图 12-6](../Text/ch12.xhtml#ch12fig06) 显示了这种测试的截图。

![image](../Images/fig12-6.jpg)

*图 12-6：我们创建的机器人的截图*

记住，这个实现只能正确处理食物的照片。

#### ***尝试这个***

请注意，前一节提供的机器人实现无法对多种不同类型的用户输入生成智能响应。我们使用的wiki()函数只能正确处理那些keyphrase()返回单个词的请求。如果该关键字是直接宾语，效果会最好。此外，机器人只能智能地响应食物的图片。

增强wiki()函数，使其能够处理短语，而不仅仅是单个词，例如“海豚睡觉”。找到适合这样的短语的句子需要使用依赖标签，因为你需要找到一个主语/动词对。此外，你还需要将单词还原为词元。例如，“海豚睡觉”和“海豚睡”应该满足搜索标准。

你也许想增强photo_tags()函数的功能，使其不仅能处理食物照片，还能处理其他类型的照片——例如，服装类照片。

### **总结**

在这一章中，你看到了如何将spaCy与Python AI生态系统中的其他库结合使用，构建一个能够处理不同类型数据的AI应用程序。通过使用Wikipedia和Clarifai的Python API，我们设计了一个能够对图片作出反应并从Wikipedia中提取文本的聊天机器人，这些技术使得机器人成为一个更智能的对话者。

阅读完本书后，你可能会想扩展和提升所学的知识。最自然的方式是继续尝试使用聊天机器人。首先，按照[第11章](../Text/ch11.xhtml#ch11)中的说明，使用Python构建一个Telegram脚本；接着，按照本章提供的说明，增强其功能。然后，改进你在本书中学到的算法，使其更适合你的使用场景。
