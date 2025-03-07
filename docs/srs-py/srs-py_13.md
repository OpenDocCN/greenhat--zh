## 第十三章：**写得更少，编码更多**

![image](img/common01.jpg)

在本章的最后，我汇总了我用来写出更好代码的 Python 一些高级特性。这些特性不仅限于 Python 标准库。我们将讨论如何让你的代码兼容 Python 2 和 3，如何创建类似 Lisp 的方法调度器，如何使用上下文管理器，以及如何使用`attr`模块创建类的模板。

### **使用 six 实现 Python 2 和 3 的支持**

正如你可能知道的，Python 3 打破了与 Python 2 的兼容性，并且做了一些调整。然而，语言的基础在不同版本间并没有改变，这使得实现前向和后向兼容成为可能，从而架起了 Python 2 和 Python 3 之间的桥梁。

幸运的是，这个模块已经存在！它叫做`six`——因为 2 × 3 = 6。

`six`模块提供了有用的`six.PY3`变量，这是一个布尔值，指示你是否正在运行 Python 3。这是你代码库中的关键变量，适用于两个版本的代码：一个是 Python 2 版本，另一个是 Python 3 版本。然而，要小心不要滥用它；在代码中到处散布`if six.PY3`会让别人很难阅读和理解你的代码。

当我们在“生成器”一章的第 121 页讨论生成器时，我们看到 Python 3 有一个很棒的特性，它让各种内置函数，如`map()`或`filter()`，返回可迭代对象而不是列表。因此，Python 3 摒弃了像`dict.iteritems()`这样的旧方法，它是 Python 2 中`dict.items()`的可迭代版本，改为让`dict.items()`返回一个迭代器，而不是列表。这个方法及其返回类型的变化可能会导致你的 Python 2 代码出错。

`six`模块为这种情况提供了`six.iteritems()`，可以用来替代像下面这样的 Python 2 特定代码：

```py
for k, v in mydict.iteritems():
    print(k, v)
```

使用`six`，你可以将`mydict.iteritems()`代码替换为兼容 Python 2 和 3 的代码，如下所示：

```py
import six

for k, v in six.iteritems(mydict):
    print(k, v)
```

然后，*瞧*，Python 2 和 Python 3 的兼容性瞬间达成！`six.iteritems()`函数将根据你使用的 Python 版本，使用`dict.iteritems()`或`dict.items()`来返回一个生成器。`six`模块提供了许多类似的辅助函数，可以让你轻松支持多个 Python 版本。

另一个例子是`six`解决方案对于`raise`关键字的不同语法问题，在 Python 2 和 Python 3 之间有所不同。在 Python 2 中，`raise`接受多个参数，但在 Python 3 中，`raise`只接受一个异常作为参数，不能再有其他参数。在 Python 3 中，如果你写一个包含两个或三个参数的`raise`语句，会导致`SyntaxError`。

`six`模块在这里提供了一个解决方法，形式为函数`six.reraise()`，它允许你在任何版本的 Python 中重新抛出异常。

#### ***字符串与 Unicode***

Python 3 增强的编码处理能力解决了 Python 2 中的字符串和 unicode 问题。在 Python 2 中，基本的字符串类型是`str`，只能处理基本的 ASCII 字符串。类型`unicode`是在 Python 2.5 中后来添加的，用于处理实际的文本字符串。

在 Python 3 中，基本的字符串类型依然是`str`，但它与 Python 2 中的`unicode`类共享属性，并能够处理高级编码。`bytes`类型取代了`str`类型，用于处理基本的字符流。

`six`模块再次提供了函数和常量，如`six.u`和`six.string_types`，以处理过渡。同样的兼容性也适用于整数，`six.integer_types`将处理已从 Python 3 中移除的`long`类型。

#### ***处理 Python 模块的迁移***

在 Python 标准库中，一些模块在 Python 2 和 Python 3 之间已经迁移或更名。`six`模块提供了一个名为`six.moves`的模块，透明地处理了很多这些迁移。

例如，Python 2 中的`ConfigParser`模块在 Python 3 中已更名为`configparser`。列表 13-1 展示了如何通过使用`six.moves`将代码移植并兼容两个主要的 Python 版本：

```py
from six.moves.configparser import ConfigParser

conf = ConfigParser()
```

*列表 13-1：使用 six.moves 在 Python 2 和 Python 3 中使用 ConfigParser()*

你还可以通过`six.add_move`添加自己的迁移处理，来处理`six`本身没有原生支持的代码过渡。

如果`six`库无法覆盖你所有的使用场景，可能值得构建一个封装`six`的兼容性模块，从而确保你能够增强该模块以适应未来的 Python 版本，或者在你希望停止支持某个特定版本的语言时，弃用（部分）模块。还需要注意的是，`six`是开源的，你可以为其贡献代码，而不是维护自己的“黑客”修改！

#### ***现代化模块***

最后，有一个名为`modernize`的工具，它使用`six`模块来“现代化”你的代码，将其移植到 Python 3，而不是仅仅将 Python 2 的语法转换为 Python 3 的语法。它同时支持 Python 2 和 Python 3。`modernize`工具通过为你完成大部分繁重的工作，帮助你的移植工作顺利起步，因此比标准的`2to3`工具更值得选择。

### **像 Lisp 一样使用 Python 来创建单一的调度器**

我喜欢说，Python 是 Lisp 编程语言的一个良好子集，随着时间的推移，我发现这一点越来越真实。PEP 443 证明了这一点：它描述了一种以类似于 Common Lisp 对象系统（CLOS）提供的方式来调度通用函数的方法。

如果你熟悉 Lisp，这对你来说并不是什么新闻。Lisp 对象系统是 Common Lisp 的基本组成部分之一，它提供了一种简单高效的方式来定义和处理方法分发。我将首先向你展示 Lisp 中的通用方法是如何工作的。

#### ***在 Lisp 中创建通用方法***

首先，让我们在 Lisp 中定义几个非常简单的类，不包含任何父类或属性：

```py
(defclass snare-drum ()
  ())

(defclass cymbal ()
  ())

(defclass stick ()
  ())

(defclass brushes ()
  ())
```

这定义了 `snare-drum`、`cymbal`、`stick` 和 `brushes` 类，这些类没有任何父类或属性。这些类组成了一个鼓组，我们可以将它们组合来发出声音。为此，我们定义了一个 `play()` 方法，它接受两个参数并返回一个表示声音的字符串：

```py
(defgeneric play (instrument accessory)
  (:documentation "Play sound with instrument and accessory."))
```

这仅定义了一个不附加到任何类上的通用方法，因此尚不能调用。在这一阶段，我们仅向对象系统说明该方法是通用的，可能会使用两个名为 `instrument` 和 `accessory` 的参数进行调用。在列表 13-2 中，我们将实现这个方法的版本，以模拟演奏我们的小军鼓。

```py
(defmethod play ((instrument snare-drum) (accessory stick))
  "POC!")

(defmethod play ((instrument snare-drum) (accessory brushes))
  "SHHHH!")

(defmethod play ((instrument cymbal) (accessory brushes))
  "FRCCCHHT!")
```

*列表 13-2：在 Lisp 中定义独立于类的通用方法*

现在我们已经在代码中定义了具体的方法。每个方法接受两个参数：`instrument`，它是 `snare-drum` 或 `cymbal` 的实例，以及 `accessory`，它是 `stick` 或 `brushes` 的实例。

在这一阶段，你应该能看到该系统与 Python（或类似）对象系统的第一个主要区别：方法并不绑定到任何特定类上。方法是*通用*的，可以为任何类实现。

让我们尝试一下。我们可以使用一些对象来调用 `play()` 方法：

```py
* (play (make-instance 'snare-drum) (make-instance 'stick))
"POC!"

* (play (make-instance 'snare-drum) (make-instance 'brushes))
"SHHHH!"
```

如你所见，调用哪个函数取决于参数的类——对象系统会根据我们传递的参数类型，将函数调用*调度*到正确的函数。如果我们用一个没有定义方法的类实例调用 `play()`，则会抛出错误。

在 列表 13-3 中，`play()` 方法是使用 `cymbal` 和 `stick` 实例调用的；然而，由于从未为这些参数定义 `play()` 方法，因此它会抛出错误。

```py
* (play (make-instance 'cymbal) (make-instance 'stick))
debugger invoked on a SIMPLE-ERROR in thread
#<THREAD "main thread" RUNNING {1002ADAF23}>:
  There is no applicable method for the generic function
    #<STANDARD-GENERIC-FUNCTION PLAY (2)>
  when called with arguments
    (#<CYMBAL {1002B801D3}> #<STICK {1002B82763}>).

Type HELP for debugger help, or (SB-EXT:EXIT) to exit from SBCL.

restarts (invokable by number or by possibly abbreviated name):
  0: [RETRY] Retry calling the generic function.
  1: [ABORT] Exit debugger, returning to top level.

((:METHOD NO-APPLICABLE-METHOD (T)) #<STANDARD-GENERIC-FUNCTION PLAY (2)>
#<CYMBAL {1002B801D3}> #<STICK {1002B82763}>) [fast-method]
```

*列表 13-3：调用带有不可用签名的方法*

CLOS 提供了更多功能，比如方法继承或基于对象的调度，而不是使用类。如果你对 CLOS 提供的众多功能非常好奇，我建议阅读 Jeff Dalton 的《CLOS 简明指南》作为入门 (*[`www.aiai.ed.ac.uk/~jeff/clos-guide.html`](http://www.aiai.ed.ac.uk/~jeff/clos-guide.html)*)。

#### ***Python 中的通用方法***

Python 实现了该工作流程的简化版本，使用 `singledispatch()` 函数，该函数自 Python 3.4 起作为 `functools` 模块的一部分发布。在 2.6 到 3.3 版本中，`singledispatch()` 函数通过 Python 包索引提供；对于那些急于尝试的人，只需运行 `pip install singledispatch`。

列表 13-4 显示了我们在 列表 13-2 中构建的 Lisp 程序的大致等效代码。

```py
   import functools

   class SnareDrum(object): pass
   class Cymbal(object): pass
   class Stick(object): pass
   class Brushes(object): pass

   @functools.singledispatch
   def play(instrument, accessory):
       raise NotImplementedError("Cannot play these")

➊ @play.register(SnareDrum)
   def _(instrument, accessory):
       if isinstance(accessory, Stick):
           return "POC!"
       if isinstance(accessory, Brushes):
           return "SHHHH!"
       raise NotImplementedError("Cannot play these")

   @play.register(Cymbal)
   def _(instrument, accessory):
       if isinstance(accessory, Brushes):
           return "FRCCCHHT!"
       raise NotImplementedError("Cannot play these")
```

*列表 13-4：使用 singledispatch 调度方法调用*

这个列表定义了我们的四个类和一个基础的 `play()` 函数，它抛出 `NotImplementedError`，表示默认情况下我们不知道该怎么做。

然后我们为一个特定的乐器 `SnareDrum` ➊ 编写一个专门版本的 `play()` 函数。该函数检查传入的配件类型，并返回相应的声音，如果配件无法识别，则再次引发 `NotImplementedError`。

如果我们运行程序，它的工作方式如下：

```py
>>> play(SnareDrum(), Stick())
'POC!'
>>> play(SnareDrum(), Brushes())
'SHHHH!'
>>> play(Cymbal(), Stick())
Traceback (most recent call last):
NotImplementedError: Cannot play these
>>> play(SnareDrum(), Cymbal())
NotImplementedError: Cannot play these
```

`singledispatch` 模块会检查传入的第一个参数的类，并调用适当版本的 `play()` 函数。对于 `object` 类，第一个定义的版本总是会被执行。因此，如果我们的乐器是一个我们没有注册的类的实例，那么将调用这个基础函数。

正如我们在 Lisp 版本的代码中看到的那样，CLOS 提供了一个多重调度器，可以根据方法原型中定义的 *任何参数的类型* 来调度，而不仅仅是第一个参数。Python 的调度器被称为 `singledispatch` 是有原因的：它只知道如何根据第一个参数进行调度。

此外，`singledispatch` 不提供直接调用父函数的方法。没有类似 Python `super()` 的功能；你需要使用各种技巧来绕过这一限制。

虽然 Python 正在改进其对象系统和调度机制，但它仍然缺乏一些像 CLOS 提供的那种更高级的功能。这使得在实际应用中遇到 `singledispatch` 相对罕见。尽管如此，知道它的存在仍然很有意思，因为你可能会在某个时刻自己实现类似的机制。

### **上下文管理器**

Python 2.6 引入的 `with` 语句很可能会让老派的 Lisp 程序员想起该语言中常用的各种 `with-*` 宏。Python 提供了一个类似的机制，通过使用实现了 *上下文管理协议* 的对象来实现。

如果你从未使用过上下文管理协议，这里是它的工作原理。`with` 语句中包含的代码块被两个函数调用包围。`with` 语句中使用的对象决定了这两个函数调用。那些对象被称为实现了上下文管理协议。

像 `open()` 返回的这些对象支持该协议；这就是为什么你可以写出如下代码：

```py
with open("myfile", "r") as f:
   line = f.readline()
```

`open()` 返回的对象有两个方法：一个是 `__enter__`，另一个是 `__exit__`。这两个方法分别在 `with` 块的开始和结束时被调用。

一个简单的上下文对象实现如 示例 13-5 所示。

```py
class MyContext(object):
    def __enter__(self):
        pass

    def __exit__(self, exc_type, exc_value, traceback):
        pass
```

*示例 13-5：一个简单的上下文对象实现*

该实现什么都不做，但它是有效的，并展示了需要定义的方法签名，以便提供一个遵循上下文协议的类。

当你在代码中识别到以下模式时，上下文管理协议可能是适合使用的。这种模式要求在调用方法 `A` 后，*必须* 始终调用方法 `B`：

1.  调用方法 `A`。

1.  执行一些代码。

1.  调用方法`B`。

`open()`函数很好地说明了这个模式：打开文件并在内部分配文件描述符的构造函数是方法`A`。释放文件描述符的`close()`方法对应方法`B`。显然，`close()`函数总是应该在你实例化文件对象之后调用*。

手动实现这个协议可能会很繁琐，因此`contextlib`标准库提供了`contextmanager`装饰器，使得实现变得更加容易。`contextmanager`装饰器应该应用于一个生成器函数。`__enter__`和`__exit__`方法会根据包裹`yield`语句的代码动态地为你实现。

在清单 13-6 中，`MyContext`被定义为一个上下文管理器。

```py
import contextlib

@contextlib.contextmanager
def MyContext():
    print("do something first")
    yield
    print("do something else")

with MyContext():
    print("hello world")
```

*清单 13-6：使用 contextlib.contextmanager*

`yield`语句之前的代码将在`with`语句体执行之前运行；`yield`语句之后的代码将在`with`语句体结束后执行。执行时，这个程序会输出以下内容：

```py
do something first
hello world
do something else
```

这里有几件事需要处理。首先，可能会在生成器中`yield`一些内容，作为`with`块的一部分使用。

清单 13-7 展示了如何将一个值传递给调用者。关键字`as`用于将该值存储在一个变量中。

```py
import contextlib

@contextlib.contextmanager
def MyContext():
    print("do something first")
    yield 42
    print("do something else")

with MyContext() as value:
    print(value)
```

*清单 13-7：定义一个返回值的上下文管理器*

清单 13-7 展示了如何将一个值传递给调用者。关键字`as`用于将该值存储在一个变量中。执行时，代码会输出以下内容：

```py
do something first
42
do something else
```

使用上下文管理器时，你可能需要处理在`with`代码块中引发的异常。可以通过将`yield`语句包裹在`try...except`块中来处理异常，正如在清单 13-8 中展示的那样。

```py
   import contextlib

   @contextlib.contextmanager
   def MyContext():
       print("do something first")
       try:
           yield 42
       finally:
           print("do something else")

   with MyContext() as value:
       print("about to raise")
➊     raise ValueError("let's try it")
       print(value)
```

*清单 13-8：在上下文管理器中处理异常*

在这里，`ValueError`在`with`代码块的开始处被引发 ➊；Python 会将这个错误传播回上下文管理器，并且`yield`语句似乎会引发这个异常。我们将`yield`语句封装在`try`和`finally`中，以确保最终的`print()`被执行。

执行时，清单 13-8 输出以下内容：

```py
do something first
about to raise
do something else
Traceback (most recent call last):
  File "<stdin>", line 3, in <module>
ValueError: let's try it
```

如你所见，错误被传播回上下文管理器，程序继续执行并完成，因为它通过`try...finally`块忽略了异常。

在某些情况下，可能需要同时使用多个上下文管理器，例如，在清单 13-9 中展示的那样，同时打开两个文件以复制它们的内容。

```py
with open("file1", "r") as source:
    with open("file2", "w") as destination:
        destination.write(source.read())
```

*清单 13-9：同时打开两个文件以复制内容*

话虽如此，由于 `with` 语句支持多个参数，实际上使用单个 `with` 编写的版本效率更高，如 列表 13-10 所示。

```py
with open("file1", "r") as source, open("file2", "w") as destination:
    destination.write(source.read())
```

*列表 13-10：使用一个 with 语句同时打开两个文件*

上下文管理器是非常强大的设计模式，帮助确保你的代码流始终正确，无论可能发生什么异常。在许多需要由其他代码包装的代码场景中，它们能提供一致且干净的编程接口，`contextlib.contextmanager` 可以帮助你实现这一点。

### **减少模板代码，使用 attr**

编写 Python 类可能会让人觉得繁琐。你经常会发现自己重复一些模式，因为没有其他选择。最常见的例子之一，如 列表 13-11 所示，是当使用几个传递给构造函数的属性来初始化一个对象时。

```py
class Car(object):
    def __init__(self, color, speed=0):
        self.color = color
        self.speed = speed
```

*列表 13-11：常见的类初始化模板代码*

这个过程始终是一样的：你将传递给 `__init__` 函数的参数值复制到存储在对象中的一些属性中。有时你还需要检查传递的值，计算默认值，等等。

显然，你还希望当对象被打印时，它能够正确地表示自己，因此你需要实现一个 `__repr__` 方法。如果你的某些类足够简单，能够转换为字典进行序列化，那么事情会更加复杂。谈到比较和可哈希性（能够对一个对象使用 `hash` 并将其存储在 `set` 中）时，情况就更复杂了。

实际上，大多数 Python 程序员并不会做这些工作，因为编写这些检查和方法的负担太重，尤其是在你并不总是确定是否需要它们的情况下。例如，你可能发现 `__repr__` 只在你调试或追踪程序时才有用，决定在标准输出中打印对象——而不是其他时候。

`attr` 库通过为你的所有类提供通用模板代码，并为你生成大部分代码，旨在提供一个简单的解决方案。你可以使用命令 `pip install attr` 来安装 `attr`。准备好享受它吧！

安装完成后，`attr.s` 装饰器是进入 `attr` 神奇世界的入口。将其放在类声明之上，然后使用函数 `attr.ib()` 来声明类中的属性。列表 13-12 展示了如何使用 `attr` 重写 列表 13-11。

```py
import attr

@attr.s
class Car(object):
    color = attr.ib()
    speed = attr.ib(default=0)
```

*列表 13-12：使用 attr.ib() 声明属性*

以这种方式声明时，类会自动获得一些有用的方法，例如 `__repr__`，该方法在 Python 解释器中打印对象时会被调用：

```py
>>> Car("blue")
Car(color='blue', speed=0)
```

这个输出比 `__repr__` 默认打印的内容更简洁：

```py
<__main__.Car object at 0x104ba4cf8>.
```

你还可以通过使用 `validator` 和 `converter` 关键字参数来为属性添加更多验证。

示例 13-13 展示了如何使用`attr.ib()`函数声明带有一些约束的属性。

```py
import attr

@attr.s
class Car(object):
    color = attr.ib(converter=str)
    speed = attr.ib(default=0)

    @speed.validator
    def speed_validator(self, attribute, value):
        if value < 0:
            raise ValueError("Value cannot be negative")
```

*示例 13-13：使用带有转换器参数的 attr.ib()*

`converter` 参数管理传递给构造函数的任何内容的转换。`validator()`函数可以作为参数传递给 `attr.ib()` 或作为装饰器使用，如 示例 13-13 所示。

`attr`模块提供了一些内置的验证器（例如，`attr.validators.instance_of()`用于检查属性的类型），因此在浪费时间自己实现之前，务必先查看它们。

`attr` 模块还提供了使对象可哈希的调整，使其可以用作集合或字典的键：只需将 `frozen=True` 传递给 `attr.s()`，使类实例变为不可变。

示例 13-14 展示了使用 `frozen` 参数如何改变类的行为。

```py
>>> import attr
>>> @attr.s(frozen=True)
... class Car(object):
...     color = attr.ib()
...
>>> {Car("blue"), Car("blue"), Car("red")}
{Car(color='red'), Car(color='blue')}
>>> Car("blue").color = "red"
attr.exceptions.FrozenInstanceError
```

*示例 13-14：使用 frozen=True*

示例 13-14 展示了使用 `frozen` 参数如何改变 `Car` 类的行为：它可以被哈希化，因此可以存储在集合中，但对象再也无法被修改了。

总之，`attr`提供了许多有用方法的实现，从而节省了你自己编写这些方法的时间。我强烈推荐在构建类和建模软件时充分利用`attr`的高效性。

### **总结**

恭喜！你已经读完了这本书。你刚刚提升了你的 Python 技能，并且对如何编写高效且富有生产力的 Python 代码有了更清晰的认识。我希望你像我写这本书一样享受阅读它的过程。

Python 是一门非常棒的语言，可以应用于许多不同的领域，而且还有很多 Python 的内容我们在本书中没有涉及。但每本书总得有个结尾，对吧？

我强烈建议通过阅读现有的开源项目源代码并参与其中来从中获益。让其他开发者审查和讨论你的代码通常是学习的好方法。

快乐编程！
