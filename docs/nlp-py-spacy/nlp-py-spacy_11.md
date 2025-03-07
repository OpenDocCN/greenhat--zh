## **11

部署你自己的聊天机器人**

![Image](img/comm1.jpg)

在之前的章节中，你通过手动将文本分配给文档对象来硬编码所有输入到你的 NLP 脚本中。但当你为类似接受订单这样的任务构建聊天机器人时，事情会变得更复杂。你将需要将应用部署到*机器人频道*，例如 Telegram，它促进了机器人与用户之间的通信。

本章首先概述了如何组织一个聊天机器人应用。你将被引导通过在 Telegram 上为你的聊天机器人准备平台并将机器人部署到该平台的过程。你将学习如何使用 Telegram API 处理多种类型的用户输入，并保持对话状态，以便跟踪已问过的问题。

### **实现和部署聊天机器人的工作原理**

本节将深入探讨典型聊天机器人与用户之间如何传输信息，以及这种传输所需的结构。

一个典型的聊天机器人应用由多个层次组成。在你完成了处理用户输入的逻辑后，你需要一个允许创建程序账户的消息应用。用户不会直接与机器上的机器人实现交互；相反，他们会通过消息应用与机器人聊天。除了消息应用之外，你的聊天机器人可能还需要一些额外的服务，例如数据库或其他存储。

图 11-1 中的示意图展示了典型聊天机器人应用如何结合这些层次。

![image](img/fig11-1.jpg)

*图 11-1：用户与集成到消息应用中的机器人之间的典型交互*

机器人应用首先向消息应用发送请求，进入一个无限循环，检查用户是否已经开始对话。这些请求包括开发者在消息应用中创建机器人时生成的身份验证令牌。*身份验证令牌*（也叫访问令牌或 API 密钥）是唯一的，用于让消息应用识别来自特定机器人的请求。

当用户向机器人发送消息时，消息应用处理该消息，并将其转发给收件人。机器人选择一个合适的*处理程序*——一个针对特定类型用户消息生成响应的例程——并将生成的回复发送给用户。

聊天机器人用来与用户交互的中介程序通常是由消息应用提供的机器人平台，如 Skype、Facebook Messenger 或 Telegram。从消息应用的角度看，机器人是运行在消息应用中的第三方应用。

下一节将引导你完成将用 Python 实现的聊天机器人部署到 Telegram 机器人平台的过程。你将看到一些特定于 Telegram 机器人平台的实现细节，并学会利用其功能简化机器人开发。

我选择 Telegram 机器人平台作为这个示例，因为它为 Python 开发者提供了全面的资源，包括 Python Telegram Bot 文档、指南和教程，以及 GitHub 上的示例。也就是说，Telegram 提供了您构建 Python 聊天机器人的一切所需资源。在其他消息应用中，例如 Facebook Messenger，您需要使用第三方工具，如 Flask 或 Ngrok，这样会让机器人的实现变得更复杂，并且无法专注于 NLP 任务。

### **将 Telegram 作为您机器人平台的使用**

Telegram 是一个基于云的消息应用，是全球最顶尖的消息应用之一。除了其他功能外，它还提供了一个平台来创建机器人，并且提供了一个易于使用的 Python 库接口。您可以在 Android、iOS、Windows、Linux 和 macOS 平台上使用 Telegram。但它主要是为智能手机设计的。

#### ***创建 Telegram 账户并授权您的机器人***

在您可以在 Telegram 中创建机器人之前，您必须先注册一个 Telegram 账户。为此，您需要一部运行 iOS 或 Android 系统的智能手机或平板电脑。PC 版本的 Telegram 不支持此操作。不过，一旦您创建了 Telegram 账户，就可以在 PC 上使用它。

您可以在 *[`telegramguide.com/create-a-telegram-account/`](https://telegramguide.com/create-a-telegram-account/)* 找到指导您创建 Telegram 账户的步骤。一旦您拥有了 Telegram 账户，就可以创建一个机器人。您可以通过智能手机或 PC 来完成此操作，具体步骤如下所示：

1.  在 Telegram 应用中，搜索 @BotFather 或打开 URL *[`telegram.me/botfather/`](https://telegram.me/botfather/)*。BotFather 是一个 Telegram 机器人，用于管理您账户中的所有其他机器人。

1.  在 BotFather 页面，点击**开始**按钮，查看您可以使用的命令列表，以设置您的 Telegram 机器人。

1.  要创建一个新机器人，在**写消息**框中输入 /newbot 命令。系统将提示您为机器人设置一个名称和用户名。然后，您将获得新机器人的授权令牌。图 11-2 展示了在智能手机上进行此过程的截图。

![image](img/fig11-2.jpg)

*图 11-2：在 Telegram 中使用智能手机创建新机器人*

现在，您可以将您在机器上实现的机器人功能集成到 Telegram 中刚创建的机器人中，正如下一节所讨论的那样。

**注意**

*需要注意的是，您在 Telegram 中刚创建的机器人并不包含处理用户输入的逻辑。事实上，它只是您需要自己实现的实际机器人的外壳。*

#### ***使用 python-telegram-bot 库入门***

要连接用 Python 实现的聊天机器人功能，你需要使用 python-telegram-bot 库，该库基于 Telegram Bot API 构建。该库为开发 Telegram 应用的机器人程序员提供了一个易于使用的接口，使你可以专注于编写机器人的代码，而不必关注与消息传递应用程序和机器人实现之间的交互细节。

python-telegram-bot 库是一个免费的软件，按照 LGPLv3 许可证分发。你可以通过以下命令使用 pip 安装或升级它：

$ pip install python-telegram-bot --upgrade

**注意**

*本章后续提供的示例假设你正在使用 python-telegram-bot 版本 12.0 或更高版本。*

安装该库后，使用以下代码进行快速测试，验证你是否能够从 Python 访问你的 Telegram 机器人。进行此测试时，你必须有一个互联网连接。

import telegram

bot = telegram.Bot(token='XXXXXX...')

在 'XXXXX' 位置，填写在创建机器人时获得的 token。然后使用以下代码检查你的凭证：

print(bot.get_me())

如果 bot.get_me() 函数返回了你的凭证，那么你之前指定的机器人认证 token 是有效的。

#### ***使用 telegram.ext 对象***

要构建一个真正的机器人，你需要使用 telegram.ext 对象，包括 telegram.ext.Updater 和 telegram.ext.Dispatcher。这是库中最重要的两个对象，因为每个实现都需要使用它们。简而言之，Updater 对象接收来自 Telegram 的消息，并将其传递给 Dispatcher。然后，Dispatcher 将数据传递给适当的处理器进行处理。以下代码演示了如何在一个简单的回声机器人中使用这些对象，该机器人会用相同的文本回复每一条消息：

from telegram.ext import Updater, MessageHandler, Filters

#实现消息处理器的函数

➊ def echo(update, context):

update.message.reply_text(update.message.text)

#创建一个 Updater 实例

➋ updater = Updater('TOKEN', use_context=True)

#注册一个处理器来处理输入的文本消息

updater.dispatcher.add_handler(MessageHandler(Filters.text, echo))

#开始从消息传递应用程序轮询更新

updater.start_polling()

updater.idle()

我们首先从 telegram.ext 包中导入 Updater 和 MessageHandler 模块。然后定义 echo() 函数，它接受两个对象作为参数：update 和 context ➊。update 对象代表传入的消息，可以是文本、照片、贴纸等。context 对象包含可以保存来自同一聊天和用户的数据的属性。update 和 context 都是在后台为你生成的，并传递给 *回调函数*——一个分配给特定处理程序的消息处理函数。在这个例子中，文本消息处理程序的回调函数是 echo()；它包含一行代码，指示 Telegram 返回用户的文本消息，而不做任何更改。

接下来，我们创建一个 Updater 对象 ➋，我们将使用它来在整个脚本中协调机器人执行过程。当我们创建 Updater 对象时，会自动为我们创建一个 Dispatcher 对象，允许我们注册不同类型输入数据的处理程序，例如文本和照片。在这个例子中，我们注册了一个只处理文本消息的处理程序，并将之前实现的回调函数传递给它。现在，每次聊天机器人收到包含文本的 Telegram 消息时，它都会调用回调函数。

然后我们通过调用 Updater 的 start_polling() 方法来启动机器人，这个方法会启动从消息传递者拉取新消息的过程。因为 start_polling() 是一个非阻塞方法，我们还需要调用 Updater 的 idle() 方法，它会阻塞脚本，直到收到消息或用户输入退出快捷键（CTRL-C）。有关 python-telegram-bot 库中可用的类和方法的更多详细信息，请阅读 Python Telegram Bot 的文档。

要测试脚本，将其运行在一台连接到互联网的机器上。一旦它运行，任何 Telegram 用户都可以与你的聊天机器人开始对话。在 Telegram 应用中，搜索 @*<username>*，输入你在创建机器人时给它的用户名，然后选择它。要开始对话，点击 **/start** 按钮或输入 /start 命令。然后你可以开始向你的机器人发送消息。因为你实现了一个回音机器人，所以机器人回复的任何消息应该包含你发送的相同文本。

#### ***创建一个使用 spaCy 的 Telegram 聊天机器人***

在前一部分中，我们使用了 python-telegram-bot 库，并构建了一个简单的脚本，集成到 Telegram 中。现在让我们增强我们的实现，并加入 spaCy，以确保在 Telegram 中创建的机器人能够完全正常工作。

以下代码创建了一个简单的机器人，它处理用户的言论，并判断是否包含直接宾语。根据这些信息，它生成一个回复消息给用户。这个代码本身并没有太大用处，但它应该能展示如何将使用 spaCy 实现的文本处理代码与使用 python-telegram-bot 库实现的代码连接起来。

import spacy

from telegram.ext import Updater, MessageHandler, Filters

#使用 spaCy 的回调函数

➊ def utterance(update, context):

msg = update.message.text

nlp = spacy.load('en')

doc = nlp(msg)

for token in doc:

if token.dep_ == 'dobj':

update.message.reply_text('我们正在处理您的请求...')

return

update.message.reply_text('请重新表述您的请求。请尽量具体一些。')

possible!')

#负责与 Telegram 交互的代码

updater = Updater('TOKEN', use_context=True)

updater.dispatcher.add_handler(MessageHandler(Filters.text, utterance))

updater.start_polling()

updater.idle()

请注意，负责与 Telegram 交互的代码与前面的脚本相同。唯一的不同之处在于回调函数 ➊ 的实现。在这种情况下，utterance() 函数使用 spaCy 来处理用户的输入。

在该函数中，我们首先从传递给函数的 update 对象中提取消息文本。接下来，我们将其转换为 spaCy Doc 对象，然后检查该语句中是否包含直接宾语。如果语句中没有直接宾语，我们会请求用户提供更具体的信息。例如，用户可能会说“我饿了”，这意味着他们想订些食物。但要下单，我们需要他们提供更具体的信息；例如，“我想要一个比萨”。

也许这个例子最有趣的部分在于，它展示了 spaCy 处理的语句可以来自聊天应用程序中的哪里。在前几章的例子中，我们使用的是硬编码在脚本中的语句。这是第一次，您可以看到真实的聊天机器人是如何获取它们的输入的。

#### ***扩展聊天机器人功能***

现在，您大致了解了如何将一个使用 spaCy 的聊天机器人集成到 Telegram 中，让我们创建一个更有趣的机器人。例如，您可以增强上一节中机器人的功能，使其从用户的消息中提取意图，而不仅仅是打印一个“请求正在处理中”的消息。为此，您可以重复使用前几章中的一个脚本。

返回到脚本 “使用预定义列表识别同义词” 中，位于第 117 页的第八章，该脚本使用同义词列表从提交的语句中提取意图。将此脚本中的代码放入一个单独的函数中，比如 `extract_intent()`，该函数应该接受一个参数——用户消息的文本作为一个 Doc 对象（确保排除脚本开头的硬编码语句以及打印意图的代码行）。此外，您创建的函数必须返回一个被识别的意图作为字符串。在您创建的脚本中，将新函数放置在回调函数之上，并修改回调函数，使其如下所示：

...

def extract_intent(doc):

#将来自第八章的代码放在这里

def utterance(update, context):

msg = update.message.text

nlp = spacy.load('en')

doc = nlp(msg)

对于文档中的每个 token：

如果 token.dep_ == 'dobj':

➊ intent = extract_intent(doc)

➋ 如果意图是'orderPizza'：

update.message.reply_text('我们需要更多的信息才能完成您的订单。')

订单。')

elif intent == 'showPizza':

update.message.reply_text('你想看看我们的菜单吗？')

else:

update.message.reply_text('无法识别您的意图。')

return

update.message.reply_text('请重新表述您的请求。尽量具体一些。')

可能的！'

...

我们在发言回调函数中调用新创建的 extract_intent() 函数，以获取用户的意图 ➊。然后根据获得的意图采取相应的行动。在这个例子中，我们简单地发送一个相关的消息给用户 ➋。

尽管我们可以将第八章中的代码直接放入回调函数中，但这样做会降低代码的整体可读性，因此被认为是一个不好的实践。

#### ***保持当前聊天的状态***

现在你拥有的机器人不仅仅是评估用户消息；它还能识别用户的意图。不过，这还不足以接收用户的订单。主要的缺陷是即便意图已经被识别，聊天机器人仍会使用相同的发言回调函数处理每个用户输入，而这是时候该询问用户更多问题了。

为了解决这个问题，你需要*保持当前聊天的状态*，这样你的机器人才能知道哪些问题已经回答，哪些问题仍然需要问。然后你需要修改回调函数，以便它能够根据聊天的当前状态处理用户消息。

这个机器人可以这样工作：如果聊天机器人尚未识别意图，它应该要求用户表达意图。在找到意图后，机器人应该切换到与当前对话状态相关的下一个问题。

为了帮助你简化这种实现，python-telegram-bot 库包括了 ConversationHandler 对象；它允许你通过将入口点和对话的状态与处理程序关联来定义这些入口点和状态。

入口点——例如，像/start 这样的 Telegram 命令——会附加到一个处理程序上，触发聊天的开始。处理程序的回调必须返回对话的初始状态；这个操作决定了后续用户消息使用哪个处理程序。为了改变对话的状态，处理程序的回调会在回复用户后返回一个新的状态。

以下代码展示了如何使用 ConversationHandler 在聊天机器人和用户之间改变对话状态：

def start(update, context):

...

➊ return 'ORDERING'

def intent_ext(update, context):

...

➋ 如果 context.user_data.has_key('intent'):

return 'ADD_INFO'

else:

update.message.reply_text('请重新表述您的请求。')

return 'ORDERING'

def add_info(update, context):

...

return ConversationHandler.END

def cancel(update, context):

...

return ConversationHandler.END

...

def main():

...

disp = updater.dispatcher

conv_handler = ConversationHandler(

entry_points=[CommandHandler('start', start)],

states={

➌ 'ORDERING': [MessageHandler(Filters.text,

intent_ext)

],

'ADD_INFO': [MessageHandler(Filters.text,

add_info)

],

},

fallbacks=[CommandHandler('cancel', cancel)]

)

disp.add_handler(conv_handler)

...

使用 ConversationHandler 允许我们定义多个回调函数，并且指定它们应该被调用的顺序。回调函数处理用户的消息，并根据处理结果，可能会改变对话流的状态。

在这个例子中，/start 命令的回调将对话切换到 ORDERING 状态 ➊，这意味着来自用户的下一条消息将由 intent_ext() 函数处理。原因是，intent_ext() 函数是属于 ORDERING 状态处理程序 ➌ 的回调函数，正如在 ConversationHandler 对象的 states 字典中所定义的那样。

请注意，聊天机器人可以根据条件逻辑在状态之间切换，如 intent_ext() 函数中所示：在那里，只有当意图被识别时，才会将对话状态切换到 ADD_INFO（收集附加信息的状态） ➋。

#### ***将所有部分组合在一起***

现在你对如何构建一个遵循预定义对话流程的 Telegram 机器人有了初步了解，让我们看看一个完整实现的脚本应该是什么样的。这个机器人需要询问用户一系列问题，逐个进行，以完成一个订单表单。因为这是一个简化的示例，聊天机器人只能处理一个意图——orderPizza，并要求用户在填写订单表单时只指定披萨的类型。

以下脚本被分为多个块，每个块代表一个函数定义：

import logging

import sys

import spacy

from telegram.ext import Updater, CommandHandler, MessageHandler, Filters, ConversationHandler

# 允许你获取通用的调试信息

logger = logging.getLogger(__name__)

logging.basicConfig(stream=sys.stdout, level=logging.DEBUG)

def extract_intent(doc):

# 这里应该是早期在“扩展聊天机器人”部分建议的代码

...

return intent

extract_intent() 函数从用户提交的语句中提取意图。我们将在下一个定义的 intent_ext() 回调函数中调用这个函数。extract_intent() 函数的代码在此未提供，但你可以参考早些时候在 “扩展聊天机器人” 一节中的代码，见 第 161 页。

def details_to_str(user_data):

details = list()

对于用户数据中的每个键值对，使用 `for key, value in user_data.items():`

details.append('{} - {}'.format(key, value))

return "\n".join(details).join(['\n', '\n'])

details_to_str()函数仅仅是将 user_data 字典的内容转换为字符串。user_data 字典包含了聊天机器人从对话中提取的信息，例如用户想要的披萨种类和数量。机器人将在最终发送给用户的消息中包含这些信息。

到目前为止，我们已经定义了将被调用的辅助函数——这些函数将直接或间接地从机器人的回调函数中被调用。现在让我们定义回调函数。

def start(update, context):

update.message.reply_text('嗨！这是一个披萨订购应用。你想点点什么吗？')

return 'ORDERING'

start()函数是/start Telegram 命令的回调函数。换句话说，聊天机器人在开始聊天时会调用此函数。该函数返回 ORDERING 状态，这意味着接收到的下一条消息将由附加到 ORDERING 状态处理器的回调函数（在本示例中是 intent_ext()函数）处理。

def intent_ext(update, context):

msg = update.message.text

nlp = spacy.load('en')

doc = nlp(msg)

for token in doc:

if token.dep_ == 'dobj':

intent = extract_intent(doc)

if intent == 'orderPizza':

context.user_data['product'] = 'pizza'

update.message.reply_text('我们需要更多的信息来完成您的订单。请问您想要什么类型的披萨？')

你想要多少个披萨？')

return 'ADD_INFO'

else:

update.message.reply_text('无法识别您的意图。请重新表述您的请求。')

return 'ORDERING'

return

update.message.reply_text('请重新表述您的请求。尽量具体一些！')

为了简化，intent_ext()函数在这里仅能识别一个意图：orderPizza。如果它检测到该意图，则返回 ADD_INFO 状态。否则，它返回 ORDERING 状态，这将导致 intent_ext()函数再次被调用以处理下一个用户消息。ADD_INFO 状态的处理器可以如下实现：

def add_info(update, context):

msg = update.message.text

nlp = spacy.load('en')

doc = nlp(msg)

for token in doc:

if token.dep_ == 'dobj':

dobj = token

for child in dobj.lefts:

if child.dep_ == 'amod' or child.dep_ == 'compound':

context.user_data['type'] = child.text

user_data = context.user_data

update.message.reply_text("您的订单已提交。")

"{}"

"祝你有个愉快的一天！".format(details_to_str(user_data)))

return ConversationHandler.END

update.message.reply_text("无法提取必要的信息。请再试一次。")

return 'ADD_INFO'

add_info()函数是 ADD_INFO 状态处理器的回调函数。在这个实现中，它期望用户在点披萨时会指定他们想要的披萨类型，然后将状态切换到 ConversationHandler.END，即最后一个状态，具体如下：

def cancel(update, context):

update.message.reply_text("祝你有个愉快的一天！")

return ConversationHandler.END

这里使用的 cancel()函数仅仅是向用户发送一条告别消息，并将状态切换到 ConversationHandler.END。

最后，main() 函数应如下所示：

def main():

#将 TOKEN 替换为实际的令牌

updater = Updater("TOKEN", use_context=True)

disp = updater.dispatcher

conv_handler = ConversationHandler(

entry_points=[CommandHandler('start', start)],

states={

'ORDERING': [MessageHandler(Filters.text,

intent_ext)

],

'ADD_INFO': [MessageHandler(Filters.text,

add_info)

],

},

fallbacks=[CommandHandler('cancel', cancel)]

)

disp.add_handler(conv_handler)

updater.start_polling()

updater.idle()

if __name__ == '__main__':

main()

像往常一样，机器人脚本的 main() 函数负责协调机器人的执行过程。

你可以通过计算机上的 Telegram 网页应用或智能手机上的 Telegram 应用来测试该脚本。图 11-3 显示了运行脚本时，Telegram 网页应用的屏幕截图。

![image](img/fig11-3.jpg)

*图 11-3：使用 Telegram 网页应用测试你的机器人*

#### ***试试这个***

修改前面的脚本，使其能够识别和处理比 orderPizza 更多的意图。另一个相关的意图可以是 showPizza，表示用户想查看菜单。为了实现这一点，你需要修改 intent_ext() 函数，在文档处理循环中添加 `if intent == 'showPizza'` 条件。同时，你还需要在 ConversationHandler 对象的 states 字典中添加一个新状态——比如 SHOW_MENU——并为其实现回调函数。

### **总结**

在本章中，你学习了如何使用 Telegram 机器人平台（一个流行的消息应用）实现和部署一个简单的聊天机器人应用。你学会了如何定义和保持对话中的状态。特别地，你还看到了一个例子，其中用户消息的处理方式，可能来自你使用 spaCy 处理的地方。
