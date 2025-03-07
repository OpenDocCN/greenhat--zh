## **13**

**少写代码，多做事情**

![image](../images/common01.jpg)

在最后一章，我总结了我用来编写更好代码的一些 Python 高级特性。这些不仅限于 Python 标准库。我们将讨论如何使你的代码兼容 Python 2 和 3，如何创建类似 Lisp 的方法调度器，如何使用上下文管理器，以及如何使用 attr 模块为类创建模板。

### **使用 six 支持 Python 2 和 3**

如你所知，Python 3 打破了与 Python 2 的兼容性，并进行了许多调整。然而，语言的基础在版本间并未改变，这使得实现向前和向后兼容成为可能，创建了 Python 2 和 Python 3 之间的桥梁。

幸运的是，这个模块已经存在了！它叫做 six——因为 2 × 3 = 6。

six 模块提供了有用的 six.PY3 变量，它是一个布尔值，表示你是否在运行 Python 3。这是任何同时支持 Python 2 和 Python 3 版本的代码库中的核心变量。然而，要小心不要滥用它；如果在代码库中到处使用 if six.PY3，会让其他人很难阅读和理解你的代码。

当我们在 “[生成器](ch08.xhtml#lev1sec42)” 中讨论生成器时，曾提到过 Python 3 有一个很好的特性：在许多内置函数（如 map() 或 filter()）中，返回的是可迭代对象，而不是列表。因此，Python 3 摒弃了像 dict.iteritems() 这样的方法（它是 Python 2 中 dict.items() 的可迭代版本），转而让 dict.items() 返回一个迭代器而不是列表。这种方法和返回类型的变化可能会破坏你的 Python 2 代码。

six 模块提供了 six.iteritems() 来解决这种情况，可以用来替代 Python 2 特有的代码，例如：

for k, v in mydict.iteritems():

print(k, v)

使用 six，你可以将 mydict.iteritems() 代码替换为兼容 Python 2 和 3 的代码，像这样：

import six

for k, v in six.iteritems(mydict):

print(k, v)

而是 *瞧*，Python 2 和 Python 3 的兼容性轻松实现！six.iteritems() 函数将根据你使用的 Python 版本，使用 dict.iteritems() 或 dict.items() 返回一个生成器。six 模块提供了许多类似的辅助函数，能够轻松支持多个 Python 版本。

另一个例子是 six 解决方案中关于 raise 关键字的问题，其语法在 Python 2 和 Python 3 之间有所不同。在 Python 2 中，raise 可以接受多个参数，但在 Python 3 中，raise 只接受一个异常作为参数，不能有其他参数。如果在 Python 3 中写一个带有两个或三个参数的 raise 语句，将会导致 SyntaxError。

six 模块在这里提供了一种解决方法，函数 six.reraise() 允许你在使用的 Python 版本中重新抛出异常。

#### ***字符串与 Unicode***

Python 3 增强了处理高级编码的能力，解决了 Python 2 中字符串和 unicode 的问题。在 Python 2 中，基本的字符串类型是 str，只能处理基本的 ASCII 字符串。unicode 类型是在 Python 2.5 后添加的，用于处理真实的文本字符串。

在 Python 3 中，基本的字符串类型仍然是 str，但它继承了 Python 2 中 unicode 类的特性，并能够处理高级编码。bytes 类型取代了 str 类型，用于处理基本字符流。

six 模块再次提供了函数和常量，例如 six.u 和 six.string_types，来处理过渡。同样的兼容性也适用于整数，使用 six.integer_types 来处理 Python 3 中已删除的 long 类型。

#### ***处理 Python 模块的迁移***

在 Python 标准库中，一些模块在 Python 2 和 3 之间发生了移动或重命名。six 模块提供了一个名为 six.moves 的模块，能够透明地处理这些变化。

例如，Python 2 中的 ConfigParser 模块在 Python 3 中被重命名为 configparser。 [列表 13-1](ch13.xhtml#ch13list1) 显示了如何使用 six.moves 将代码移植并使其兼容两个主要的 Python 版本：

from six.moves.configparser import ConfigParser

conf = ConfigParser()

*列表 13-1：使用 six.moves 使 ConfigParser() 兼容 Python 2 和 Python 3*

你也可以通过 six.add_move 添加自己的迁移，来处理 six 不原生支持的代码迁移。

如果 six 库未涵盖所有使用场景，可能值得构建一个兼容模块，封装 six 本身，从而确保你能够增强该模块以适应未来版本的 Python，或者在你不再支持某个版本时停止支持它（或部分功能）。还要注意，six 是开源的，你可以为其做出贡献，而不是自己维护修改过的代码！

#### ***modernize 模块***

最后，有一个名为 modernize 的工具，使用 six 模块通过将代码移植到 Python 3 来“现代化”代码，而不仅仅是将 Python 2 的语法转换为 Python 3 的语法。这为 Python 2 和 Python 3 提供了支持。modernize 工具通过为你完成大部分繁重的工作，帮助你顺利开始移植工作，因此比标准的 2to3 工具更为合适。

### **像 Lisp 一样使用 Python 创建单一调度器**

我常说 Python 是 Lisp 编程语言的一个良好子集，随着时间的推移，我发现这一点越来越真实。PEP 443 证明了这一点：它描述了一种以类似于 Common Lisp 对象系统（CLOS）提供的方式来分发通用函数的方法。

如果你熟悉 Lisp，这对你来说不会是新闻。Lisp 对象系统是 Common Lisp 的基本组成部分之一，提供了一种简单高效的方式来定义和处理方法调度。首先，我会向你展示在 Lisp 中通用方法的工作原理。

#### ***在 Lisp 中创建通用方法***

首先，让我们定义几个非常简单的类，没有任何父类或属性，使用 Lisp：  

(defclass snare-drum ()  

())  

(defclass cymbal ()  

())  

(defclass stick ()

())  

(defclass brushes ()  

())  

这定义了 snare-drum、cymbal、stick 和 brushes 类，它们没有任何父类或属性。这些类组成了一个鼓组，我们可以将它们组合在一起演奏声音。为此，我们定义了一个 play() 方法，它接受两个参数并返回一个声音字符串：  

(defgeneric play (instrument accessory)  

(:documentation "使用乐器和配件播放声音。"))  

这只定义了一个通用方法，该方法没有附加到任何类，因此目前无法调用。在此阶段，我们仅仅告诉对象系统该方法是通用的，可能会使用名为 instrument 和 accessory 的两个参数进行调用。在[示例 13-2](ch13.xhtml#ch13list2)中，我们将实现该方法的不同版本，模拟演奏我们的军鼓。  

(defmethod play ((instrument snare-drum) (accessory stick))  

"POC!"  

(defmethod play ((instrument snare-drum) (accessory brushes))  

"SHHHH!"  

(defmethod play ((instrument cymbal) (accessory brushes))  

"FRCCCHHT!"  

*示例 13-2：在 Lisp 中定义独立于类的通用方法*  

现在我们已经在代码中定义了具体的方法。每个方法接受两个参数：instrument，它是 snare-drum 或 cymbal 的实例，accessory，它是 stick 或 brushes 的实例。  

在此阶段，你应该能看到这个系统与 Python（或类似）对象系统之间的第一个主要区别：该方法并没有绑定到任何特定的类。方法是*通用的*，可以为任何类实现。  

让我们试试。我们可以使用一些对象来调用我们的 play() 方法：  

* (play (make-instance 'snare-drum) (make-instance 'stick))  

"POC!"  

* (play (make-instance 'snare-drum) (make-instance 'brushes))  

"SHHHH!"  

如你所见，调用哪个函数取决于参数的类——对象系统会根据我们传入的参数类型为我们“分发”函数调用到正确的函数。如果我们用一个没有定义方法的对象调用 play()，将会抛出错误。  

在[示例 13-3](ch13.xhtml#ch13list3)中，play() 方法与一个 cymbal 和一个 stick 实例一起调用；然而，play() 方法从未为这些参数定义，因此会引发错误。  

* (play (make-instance 'cymbal) (make-instance 'stick))  

调试器在 SIMPLE-ERROR 中被调用，位于线程  

#<THREAD "main thread" RUNNING {1002ADAF23}>:  

没有适用于该通用函数的方法  

#<STANDARD-GENERIC-FUNCTION PLAY (2)>  

当调用时带有以下参数  

(#<CYMBAL {1002B801D3}> #<STICK {1002B82763}>).  

输入 HELP 获取调试器帮助，或输入 (SB-EXT:EXIT) 退出 SBCL。  

重新启动（可以通过数字或可能的简化名称调用）：  

0: [RETRY] 重试调用通用函数。  

1: [ABORT] 退出调试器，返回到顶层。  

((:METHOD NO-APPLICABLE-METHOD (T)) #<STANDARD-GENERIC-FUNCTION PLAY (2)>  

#<CYMBAL {1002B801D3}> #<STICK {1002B82763}>) [fast-method]

*Listing 13-3: 调用一个没有可用签名的方法*

CLOS提供了更多的功能，比如方法继承或基于对象的调度，而不是使用类。如果你对CLOS提供的众多功能真的感兴趣，建议阅读Jeff Dalton的《CLOS简明指南》（*[http://www.aiai.ed.ac.uk/~jeff/clos-guide.html](http://www.aiai.ed.ac.uk/~jeff/clos-guide.html)*)，作为入门资料。

#### ***使用Python的通用方法***

Python通过singledispatch()函数实现了这个工作流的简化版本，该函数自Python 3.4以来作为functools模块的一部分提供。在2.6到3.3版本中，singledispatch()函数通过Python包索引提供；对于那些急于尝试的人，只需运行pip install singledispatch。

[Listing 13-4](ch13.xhtml#ch13list4)显示了我们在[Listing 13-2](ch13.xhtml#ch13list2)中构建的Lisp程序的粗略等效版本。

import functools

class SnareDrum(object): pass

class Cymbal(object): pass

class Stick(object): pass

class Brushes(object): pass

@functools.singledispatch

def play(instrument, accessory):

raise NotImplementedError("无法播放这些")

➊ @play.register(SnareDrum)

def _(instrument, accessory):

if isinstance(accessory, Stick):

return "POC!"

if isinstance(accessory, Brushes):

return "SHHHH!"

raise NotImplementedError("无法播放这些")

@play.register(Cymbal)

def _(instrument, accessory):

if isinstance(accessory, Brushes):

return "FRCCCHHT!"

raise NotImplementedError("无法播放这些")

*Listing 13-4: 使用singledispatch调度方法调用*

该列表定义了我们的四个类和一个基本的play()函数，抛出NotImplementedError，表明默认情况下我们不知道该怎么做。

然后，我们为特定的乐器编写了一个play()函数的专门版本，这里是SnareDrum ➊。该函数检查传入的配件类型，并返回适当的声音，或者如果配件无法识别，则再次抛出NotImplementedError。

如果我们运行该程序，它的运行方式如下：

>>> play(SnareDrum(), Stick())

'POC!'

>>> play(SnareDrum(), Brushes())

'SHHHH!'

>>> play(Cymbal(), Stick())

Traceback (most recent call last):

NotImplementedError: 无法播放这些

>>> play(SnareDrum(), Cymbal())

NotImplementedError: 无法播放这些

singledispatch模块检查传入的第一个参数的类，并调用相应版本的play()函数。对于对象类，始终运行第一个定义的版本。因此，如果我们的乐器是一个未注册的类的实例，那么将调用这个基本函数。

正如我们在 Lisp 版本的代码中看到的，CLOS 提供了一个多重调度器，可以根据方法原型中*任何一个参数*的类型进行调度，而不仅仅是第一个参数。Python 的调度器被命名为 `singledispatch` 是有原因的：它只知道如何根据第一个参数进行调度。

此外，`singledispatch` 没有办法直接调用父函数。没有类似 Python `super()` 函数的功能；你必须使用各种技巧来绕过这个限制。

虽然 Python 正在改进其对象系统和调度机制，但它仍然缺乏像 CLOS 提供的开箱即用的更高级功能。因此，在实际应用中遇到 `singledispatch` 的情况比较罕见。了解它的存在仍然很有趣，因为你可能会在某个时候自己实现这种机制。

### **上下文管理器**

Python 2.6 引入的 with 语句可能会让老一辈的 Lispers 想起在该语言中经常使用的各种 with-* 宏。Python 提供了一个类似的机制，通过使用实现了*上下文管理协议*的对象。

如果你从未使用过上下文管理协议，下面是它的工作方式。包含在 with 语句中的代码块被两个函数调用包围。with 语句中使用的对象决定了这两个调用。这些对象被认为实现了上下文管理协议。

像 open() 返回的对象那样的对象支持这种协议；这就是为什么你可以像这样编写代码：

with open("myfile", "r") as f:

line = f.readline()

open() 返回的对象有两个方法：一个叫 `__enter__`，另一个叫 `__exit__`。这两个方法分别在 with 块的开始和结束时被调用。

[Listing 13-5](ch13.xhtml#ch13list5) 中展示了一个简单的上下文对象实现。

class MyContext(object):

def __enter__(self):

pass

def __exit__(self, exc_type, exc_value, traceback):

pass

*Listing 13-5: 一个简单的上下文对象实现*

这个实现并没有做任何事情，但它是有效的，展示了为了提供遵循上下文协议的类所需要定义的方法签名。

当你在代码中识别出以下模式时，上下文管理协议可能是适用的：在调用方法 A 后，必须*始终*调用方法 B：

1.  调用方法 A。

1.  执行一些代码。

1.  调用方法 B。

open() 函数很好地说明了这一模式：打开文件并内部分配文件描述符的构造函数是方法 A。释放文件描述符的 close() 方法对应方法 B。显然，close() 函数总是应该在你实例化文件对象之后调用*。

手动实现这个协议可能很繁琐，因此 contextlib 标准库提供了 contextmanager 装饰器，使得实现变得更加简便。contextmanager 装饰器应该用于生成器函数。__enter__ 和 __exit__ 方法会根据包装生成器 yield 语句的代码动态实现。

在 [示例 13-6](ch13.xhtml#ch13list6) 中，MyContext 被定义为一个上下文管理器。

import contextlib

@contextlib.contextmanager

def MyContext():

print("做一些事情")

yield

print("做其他事情")

with MyContext():

print("hello world")

*示例 13-6：使用 contextlib.contextmanager*

yield 语句之前的代码将在 with 语句体运行之前执行；yield 语句之后的代码将在 with 语句体结束后执行。执行时，这个程序输出如下内容：

做一些事情

hello world

做其他事情

这里需要处理几个问题。首先，我们的生成器中可能会 yield 一些内容，这些内容可以在 with 代码块中使用。

[示例 13-7](ch13.xhtml#ch13list7) 展示了如何将一个值返回给调用者。关键字 as 用于将这个值存储在一个变量中。

import contextlib

@contextlib.contextmanager

def MyContext():

print("做一些事情")

yield 42

print("做其他事情")

with MyContext() as value:

print(value)

*示例 13-7：定义一个返回值的上下文管理器*

[示例 13-7](ch13.xhtml#ch13list7) 展示了如何将一个值返回给调用者。关键字 as 用于将这个值存储在一个变量中。执行时，代码输出如下内容：

做一些事情

42

做其他事情

在使用上下文管理器时，你可能需要处理在 with 代码块中可能引发的异常。这可以通过将 yield 语句包裹在 try...except 块中来实现，如 [示例 13-8](ch13.xhtml#ch13list8) 所示。

import contextlib

@contextlib.contextmanager

def MyContext():

print("做一些事情")

try:

yield 42

finally:

print("做其他事情")

with MyContext() as value:

print("即将引发")

➊     引发 ValueError("让我们试试看")

print(value)

*示例 13-8：在上下文管理器中处理异常*

这里，在 with 代码块的开始处引发了一个 ValueError ➊；Python 会将这个错误传播回上下文管理器，yield 语句看起来像是引发了这个异常。我们将 yield 语句包裹在 try 和 finally 中，以确保最终的 print() 被执行。

执行时，[示例 13-8](ch13.xhtml#ch13list8) 输出如下内容：

做一些事情

即将引发

做其他事情

Traceback (most recent call last):

文件 "<stdin>"，第 3 行，在 <module>

ValueError: 让我们试试看

正如你所看到的，错误被传递回上下文管理器，程序恢复并完成执行，因为它通过 try...finally 块忽略了异常。

在某些情况下，同时使用多个上下文管理器是有用的，例如，当同时打开两个文件来复制它们的内容时，如[示例 13-9](ch13.xhtml#ch13list9)所示。

使用`with open("file1", "r") as source:`：

使用`with open("file2", "w") as destination:`：

destination.write(source.read())

*示例 13-9：同时打开两个文件以复制内容*

话虽如此，由于with语句支持多个参数，实际上使用一个单独的with语句会更高效，如[示例 13-10](ch13.xhtml#ch13list10)所示。

使用`with open("file1", "r") as source, open("file2", "w") as destination:`：

destination.write(source.read())

*示例 13-10：仅使用一个 with 语句同时打开两个文件*

上下文管理器是非常强大的设计模式，它有助于确保无论发生什么异常，你的代码流始终正确。它们能在许多场合提供一致且清晰的编程接口，这些场合中代码应当被其他代码所包装，并且使用contextlib.contextmanager。

### **使用 attr 减少模板代码**

编写 Python 类可能会很繁琐。你经常会发现自己只是在重复一些固定的模式，因为没有其他选项。最常见的例子之一，如[示例 13-11](ch13.xhtml#ch13list11)所示，就是在初始化一个对象时，通过构造函数传入几个属性。

class Car(object):

def __init__(self, color, speed=0):

self.color = color

self.speed = speed

*示例 13-11：常见的类初始化模板*

这个过程始终是一样的：你将传递给`__init__`函数的参数值复制到对象中存储的几个属性中。有时你还需要检查传入的值，计算默认值，等等。

显然，如果要打印你的对象时正确显示，你还需要实现一个__repr__方法。如果你的某些类足够简单，可能可以将它们转换为字典进行序列化。当涉及到比较和哈希性（能够在对象上使用哈希并将其存储在集合中）时，事情变得更加复杂。

实际上，大多数 Python 程序员都不会这么做，因为编写所有这些检查和方法的负担太重，特别是在你并不总是确定是否需要它们的情况下。例如，你可能会发现，__repr__ 在你的程序中只在调试或追踪时才有用，并且只会打印对象一次，并没有其他时间需要使用它。

attr 库通过为你的所有类提供通用的模板并为你生成大量代码，旨在提供一个简单的解决方案。你可以使用命令`pip install attr`安装 attr。准备好享受吧！

安装完成后，`attr.s` 装饰器是你进入 attr 神奇世界的入口。将它应用在类声明之上，然后使用 `attr.ib()` 函数在你的类中声明属性。[清单 13-12](ch13.xhtml#ch13list12) 展示了如何使用 attr 重写[清单 13-11](ch13.xhtml#ch13list11)。

import attr

@attr.s

class Car(object):

color = attr.ib()

speed = attr.ib(default=0)

*清单 13-12: 使用 `attr.ib()` 声明属性*

以这种方式声明时，类自动获得一些有用的方法，比如 `__repr__`，该方法在 Python 解释器中打印对象时被调用：

>>> Car("blue")

Car(color='blue', speed=0)

这个输出比 `__repr__` 默认打印的要干净：

<__main__.Car object at 0x104ba4cf8>.

你还可以通过使用验证器和转换器关键字参数来为你的属性添加更多验证。

[清单 13-13](ch13.xhtml#ch13list13) 展示了如何使用 `attr.ib()` 函数声明带有某些约束的属性。

import attr

@attr.s

class Car(object):

color = attr.ib(converter=str)

speed = attr.ib(default=0)

@speed.validator

def speed_validator(self, attribute, value):

if value < 0:

raise ValueError("值不能为负")

*清单 13-13: 使用 attr.ib() 和其转换器参数*

转换器参数负责管理传递给构造函数的值的转换。`validator()` 函数可以作为参数传递给 `attr.ib()`，也可以用作装饰器，如[清单 13-13](ch13.xhtml#ch13list13)所示。

attr 模块提供了一些自己的验证器（例如，`attr.validators.instance_of()` 用于检查属性类型），所以在自己编写验证器之前，最好先检查这些内置的。

attr 模块还提供了一些调整，使你的对象可以哈希，从而可以在集合或字典键中使用：只需将 frozen=True 传递给 `attr.s()`，使类实例不可变。

[清单 13-14](ch13.xhtml#ch13list14) 展示了使用 frozen 参数如何改变类的行为。

>>> import attr

>>> @attr.s(frozen=True)

... class Car(object):

...      color = attr.ib()

...

>>> {Car("blue"), Car("blue"), Car("red")}

{Car(color='red'), Car(color='blue')}

>>> Car("blue").color = "red"

attr.exceptions.FrozenInstanceError

*清单 13-14: 使用 frozen=True*

[清单 13-14](ch13.xhtml#ch13list14) 展示了使用 frozen 参数如何改变 Car 类的行为：它可以被哈希，因此可以存储在集合中，但对象不再可修改。

总结来说，attr 提供了大量有用的方法实现，帮助你省去了自己编写的麻烦。在构建类和建模软件时，我强烈推荐利用 attr 提高效率。

### **总结**

恭喜你！你已经读完了这本书。你提升了你的 Python 技能，对如何编写高效且富有生产力的 Python 代码有了更清晰的理解。我希望你读这本书的体验和我写这本书时一样愉快。

Python 是一门非常棒的语言，可以应用于许多不同的领域，而且在这本书中我们没有涉及到 Python 的许多其他领域。不过每本书都需要一个结尾，对吧？

我强烈推荐通过阅读现有的开源项目源代码并贡献其中的内容来从中获益。让其他开发者审查和讨论你的代码往往是学习的好方法。

祝你编程愉快！
