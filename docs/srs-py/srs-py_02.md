## **模块、库和框架**

![image](img/common01.jpg)

模块是 Python 可扩展性的关键部分。没有它们，Python 将只是一个围绕单一解释器构建的语言；它无法在一个庞大的生态系统中蓬勃发展，开发者也无法通过组合扩展快速简便地构建应用程序。在这一章中，我将向你介绍一些使 Python 模块出色的特性，从你需要了解的内置模块到外部管理的框架。

### **导入系统**

要在程序中使用模块和库，你必须使用`import`关键字导入它们。例如，清单 2-1 导入了 Python 之禅的核心指导原则。

```py
>>> import this
The Zen of Python, by Tim Peters

Beautiful is better than ugly.
Explicit is better than implicit.
Simple is better than complex.
Complex is better than complicated.
Flat is better than nested.
Sparse is better than dense.
Readability counts.
Special cases aren't special enough to break the rules.
Although practicality beats purity.
Errors should never pass silently.
Unless explicitly silenced.
In the face of ambiguity, refuse the temptation to guess.
There should be one-- and preferably only one --obvious way to do it.
Although that way may not be obvious at first unless you're Dutch.
Now is better than never.
Although never is often better than *right* now.
If the implementation is hard to explain, it's a bad idea.
If the implementation is easy to explain, it may be a good idea.
Namespaces are one honking great idea -- let's do more of those!
```

*清单 2-1: Python 之禅*

导入系统相当复杂，我假设你已经掌握了基础知识，因此在这里我将向你展示这个系统的一些内部工作原理，包括`sys`模块的工作方式、如何更改或添加导入路径，以及如何使用自定义导入器。

首先，你需要知道`import`关键字实际上是一个名为`__import__`的函数的封装器。以下是一个熟悉的导入模块方式：

```py
>>> import itertools
>>> itertools
<module 'itertools' from '/usr/.../>
```

这与以下方法完全等效：

```py
>>> itertools = __import__("itertools")
>>> itertools
<module 'itertools' from '/usr/.../>
```

你也可以模仿`import`的`as`关键字，就像这两种等效的导入方式所示：

```py
>>> import itertools as it
>>> it
<module 'itertools' from '/usr/.../>
```

这是第二个示例：

```py
>>> it = __import__("itertools")
>>> it
<module 'itertools' from '/usr/.../>
```

虽然`import`是 Python 中的一个关键字，但在内部它实际上是一个可以通过`__import__`名称访问的简单函数。了解`__import__`函数非常有用，因为在某些（边缘）情况下，你可能希望导入一个事先无法知道名称的模块，例如这样：

```py
>>> random = __import__("RANDOM".lower())
>>> random
<module 'random' from '/usr/.../>
```

不要忘记，一旦导入，模块本质上是对象，它们的属性（类、函数、变量等）也是对象。

#### ***sys 模块***

`sys`模块提供了与 Python 本身以及它所运行的操作系统相关的变量和函数。这个模块还包含关于 Python 导入系统的许多信息。

首先，你可以使用`sys.modules`变量检索当前导入的模块列表。`sys.modules`变量是一个字典，其键是你想要查看的模块名称，返回值是模块对象。例如，一旦导入了`os`模块，我们可以通过输入以下内容来检索它：

```py
>>> import sys
>>> import os
>>> sys.modules['os']
<module 'os' from '/usr/lib/python2.7/os.pyc'>
```

`sys.modules`变量是一个标准的 Python 字典，包含所有已加载的模块。这意味着，例如调用`sys.modules.keys()`将返回已加载模块的名称的完整列表。

你还可以使用`sys.builtin_module_names`变量来检索内置模块的列表。根据传递给 Python 构建系统的编译选项，编译到解释器中的内置模块可能会有所不同。

#### ***导入路径***

在导入模块时，Python 依赖一个路径列表来确定在哪里查找模块。这个列表存储在`sys.path`变量中。要查看你的解释器会搜索哪些路径，只需输入`sys.path`。

你可以修改这个列表，按需添加或删除路径，甚至通过修改`PYTHONPATH`环境变量来添加路径，而无需编写 Python 代码。如果你想将 Python 模块安装到非标准位置（例如测试环境），将路径添加到`sys.path`变量可能很有用。然而，在正常操作中，通常不需要更改路径变量。以下几种方法几乎等效——*几乎*因为路径在列表中的位置不同；这个差异在你的使用场景中可能不重要：

```py
>>> import sys
>>> sys.path.append('/foo/bar')
```

这几乎和下面的情况一样：

```py
$ PYTHONPATH=/foo/bar python
>>> import sys
>>> '/foo/bar' in sys.path
True
```

需要注意的是，这个列表会被迭代以查找请求的模块，因此`sys.path`中的路径顺序很重要。将最有可能包含你正在导入模块的路径放在列表前面，有助于加速搜索时间。这样做还可以确保，如果有两个同名的模块可用，首先匹配到的模块会被选中。

最后一项特性尤为重要，因为一个常见的错误是用你自己的模块覆盖 Python 的内置模块。当前目录会在 Python 标准库目录之前被搜索。这意味着，如果你决定将你的脚本命名为*random.py*，然后尝试使用`import random`，那么当前目录中的文件将被导入，而不是 Python 模块。

#### ***自定义导入器***

你还可以使用自定义导入器扩展导入机制。这就是 Lisp-Python 方言`Hy`使用的技术，它教 Python 如何导入除标准的*.py*或*.pyc*文件以外的文件。（`Hy`是基于 Python 的 Lisp 实现，稍后将在第 145 页的“快速了解 Hy”一节中讨论。）

*导入钩子机制*，即此技术，由 PEP 302 定义。它允许你扩展标准导入机制，从而可以修改 Python 导入模块的方式并构建你自己的导入系统。例如，你可以编写一个扩展，通过网络从数据库导入模块，或者在导入任何模块之前进行一些合理性检查。

Python 提供了两种不同但相关的方式来扩展导入系统：用于`sys.meta_path`的元路径查找器和用于`sys.path_hooks`的路径条目查找器。

#### ***元路径查找器***

*元路径查找器*是一个允许你加载自定义对象以及标准的*.py*文件的对象。元路径查找器对象必须暴露一个`find_module(fullname, path=None)`方法，该方法返回一个加载器对象。加载器对象还必须具有一个`load_module(fullname)`方法，负责从源文件加载模块。

为了说明，列表 2-2 展示了 `Hy` 如何使用自定义的元路径查找器，使 Python 能够导入以 *.hy* 结尾的源文件，而不是 *.py*。

```py
class MetaImporter(object):
    def find_on_path(self, fullname):
        fls = ["%s/__init__.hy", "%s.hy"]
        dirpath = "/".join(fullname.split("."))

        for pth in sys.path:
            pth = os.path.abspath(pth)
            for fp in fls:
                composed_path = fp % ("%s/%s" % (pth, dirpath))
                if os.path.exists(composed_path):
                    return composed_path

    def find_module(self, fullname, path=None):
        path = self.find_on_path(fullname)
        if path:
            return MetaLoader(path)

sys.meta_path.append(MetaImporter())
```

*列表 2-2：一个 Hy 模块导入器*

一旦 Python 确定路径有效并且指向一个模块，就会返回一个 `MetaLoader` 对象，如 列表 2-3 所示。

```py
class MetaLoader(object):
    def __init__(self, path):
        self.path = path

    def is_package(self, fullname):
        dirpath = "/".join(fullname.split("."))
        for pth in sys.path:
            pth = os.path.abspath(pth)
            composed_path = "%s/%s/__init__.hy" % (pth, dirpath)
            if os.path.exists(composed_path):
                return True
        return False

    def load_module(self, fullname):
        if fullname in sys.modules: return sys.modules[fullname]

        if not self.path:
            return

        sys.modules[fullname] = None
     ➊ mod = import_file_to_module(fullname, self.path)

        ispkg = self.is_package(fullname)

        mod.__file__ = self.path
        mod.__loader__ = self
        mod.__name__ = fullname

        if ispkg:
            mod.__path__ = []
            mod.__package__ = fullname
        else:
            mod.__package__ = fullname.rpartition('.')[0]

        sys.modules[fullname] = mod
        return mod
```

*列表 2-3：一个 Hy 模块加载器对象*

在 ➊ 处，`import_file_to_module` 读取一个 *.hy* 源文件，将其编译为 Python 代码，并返回一个 Python 模块对象。

这个加载器非常直接：一旦找到 *.hy* 文件，就将其传递给这个加载器，若有必要，它会编译文件、注册文件、设置一些属性，然后返回给 Python 解释器。

`uprefix` 模块是此功能应用的另一个很好的示例。Python 3.0 到 3.2 不支持 Python 2 中用于表示 Unicode 字符串的 `u` 前缀；`uprefix` 模块通过在编译前移除字符串中的 `u` 前缀，确保 Python 2 和 3 版本之间的兼容性。

### **有用的标准库**

Python 附带了一个庞大的标准库，里面包含了几乎任何你能想到的工具和功能。习惯于为基本任务编写自己函数的 Python 新手，通常会被语言本身已经内置并准备好使用的如此丰富的功能所震惊。

每当你有写自己函数来处理简单任务的冲动时，先停下来查阅一下标准库。事实上，在你开始使用 Python 之前，至少浏览一遍标准库，这样下次需要某个函数时，你就能知道它是否已经存在于标准库中。

我们将在后续章节讨论这些模块中的一些，比如 `functools` 和 `itertools`，但这里有一些你肯定会发现有用的标准模块：

+   `atexit` 允许你注册程序在退出时调用的函数。

+   `argparse` 提供用于解析命令行参数的函数。

+   `bisect` 提供用于排序列表的二分法算法（参见 第十章）。

+   `calendar` 提供多个与日期相关的函数。

+   `codecs` 提供用于编码和解码数据的函数。

+   `collections` 提供多种有用的数据结构。

+   `copy` 提供用于复制数据的函数。

+   `csv` 提供用于读取和写入 CSV 文件的函数。

+   `datetime` 提供处理日期和时间的类。

+   `fnmatch` 提供用于匹配 Unix 风格文件名模式的函数。

+   `concurrent` 提供异步计算（Python 3 中为原生支持，Python 2 通过 PyPI 提供）。

+   `glob` 提供用于匹配 Unix 风格路径模式的函数。

+   `io` 提供用于处理 I/O 流的函数。在 Python 3 中，它还包含 StringIO（在 Python 2 中是同名模块），允许你将字符串当作文件来处理。

+   `json` 提供用于读取和写入 JSON 格式数据的函数。

+   `logging` 提供对 Python 内置日志功能的访问。

+   `multiprocessing` 允许你从应用程序中运行多个子进程，同时提供一个 API，使它们看起来像线程。

+   `operator` 提供实现基本 Python 运算符的函数，你可以使用这些函数，而无需编写自己的 lambda 表达式（参见 第十章）。

+   `os` 提供对基本操作系统功能的访问。

+   `random` 提供生成伪随机数的函数。

+   `re` 提供正则表达式功能。

+   `sched` 提供一个事件调度器，而无需使用多线程。

+   `select` 提供对 `select()` 和 `poll()` 函数的访问，用于创建事件循环。

+   `shutil` 提供对高级文件操作的访问。

+   `signal` 提供处理 POSIX 信号的函数。

+   `tempfile` 提供用于创建临时文件和目录的函数。

+   `threading` 提供对高级线程功能的访问。

+   `urllib`（以及 Python 2.*x* 中的 `urllib2` 和 `urlparse`）提供处理和解析 URL 的函数。

+   `uuid` 允许你生成全局唯一标识符（UUID）。

使用这个列表作为快速参考，了解这些有用库模块的功能。如果你能记住其中的一部分，那就更好了。你花在查找库模块上的时间越少，就可以花更多的时间编写你真正需要的代码。

大多数标准库是用 Python 编写的，因此没有任何阻止你查看模块和函数源代码的理由。如果有疑问，打开代码看看它自己是如何工作的。即使文档中包含了你需要知道的所有内容，依然有可能学到一些有用的东西。

### **外部库**

Python 的“自带电池”理念是，一旦你安装了 Python，你应该拥有构建任何你想要的东西所需的一切。这是为了避免编程中的“拆开一个很棒的礼物，却发现赠送者忘记给它买电池”的情况。

不幸的是，Python 的开发者无法预测你可能想要制作的*所有*内容。即使他们能做到，大多数人也不愿意处理一个几 GB 的下载文件，尤其是当他们只是想写一个快速的脚本来重命名文件时。因此，即使 Python 标准库功能强大，它也不能覆盖所有内容。幸运的是，Python 社区的成员们创建了外部库。

Python 标准库是安全的、已充分探测的领域：其模块有丰富的文档，且足够多的人定期使用它，你可以放心地尝试它，而不会发生混乱的错误——即使它*确实*出现问题，你也可以确信有人会在短时间内修复它。另一方面，外部库就像地图上标注的“这里有龙”的地方：文档可能稀缺，功能可能有缺陷，更新可能零散甚至不存在。任何严肃的项目都可能需要外部库才能提供的功能，但你需要意识到使用它们的风险。

这是一个来自战壕的外部库危险故事。OpenStack 使用 SQLAlchemy，这是一个 Python 的数据库工具包。如果你熟悉 SQL，你知道数据库模式可能会随时间变化，因此 OpenStack 还使用了 `sqlalchemy-migrate` 来处理模式迁移需求。而且它曾经有效……直到它失效。漏洞开始堆积，但没有人去修复它们。那时，OpenStack 也开始考虑支持 Python 3，但 `sqlalchemy-migrate` 并没有向 Python 3 兼容性发展。到那个时候，显然 `sqlalchemy-migrate` 对我们来说已经没有用了，我们需要换成其他工具——我们的需求超出了这个外部库的能力范围。目前，OpenStack 项目正在向使用 Alembic 迁移，这是一个支持 Python 3 的新 SQL 数据库迁移工具。虽然这不是没有努力，但幸运的是，过程并没有带来太多痛苦。

#### ***外部库安全检查清单***

所有这些都引出一个重要问题：你如何确保自己不会陷入外部库的陷阱？不幸的是，你无法确定：程序员也是人，也没有办法确切知道今天仍然积极维护的库，几个月后是否还能保持良好状态。然而，使用这些库可能值得承担一定的风险；重要的是要仔细评估你的情况。在 OpenStack 中，我们使用以下清单来决定是否使用外部库，我鼓励你也这样做。

**Python 3 兼容性** 即使你当前不针对 Python 3，未来你可能会需要它，所以最好检查你选择的库是否已经支持 Python 3，并承诺保持兼容性。

**活跃开发** GitHub 和 Ohloh 通常提供足够的信息，帮助判断某个库是否正在被其维护者积极开发。

**积极维护** 即使一个库被认为已完成（即功能完整），其维护者也应该确保它保持无漏洞。检查项目的跟踪系统，看看维护者对漏洞的响应速度如何。

**与操作系统分发版捆绑** 如果一个库与主要的 Linux 发行版一起打包，这意味着其他项目也依赖于它——所以如果出现问题，你不会是唯一抱怨的人。如果你计划将软件发布给公众，检查这一点也是个好主意：如果它的依赖项已经安装在终端用户的机器上，你的代码将更容易分发。

**API 兼容性承诺** 没有什么比软件因其依赖的库更改了整个 API 而突然崩溃更糟糕的了。你可能想检查一下你选择的库是否曾经发生过类似的情况。

**许可证** 你需要确保许可证与你计划编写的软件兼容，并且它允许你按计划对代码进行分发、修改和执行。

将此检查清单应用于依赖项也是个不错的主意，尽管这可能是一个庞大的工作量。作为折衷，如果你知道你的应用程序将严重依赖某个特定的库，你应该将此检查清单应用于该库的每一个依赖项。

#### ***通过 API 封装保护你的代码***

无论你最终使用哪些库，都需要把它们当作可能造成严重损害的有用工具来对待。为了安全起见，库应该像任何物理工具一样存放：放在工具棚里，远离易碎的贵重物品，但在需要时可以随时使用。

无论外部库多么有用，都要小心不要让它深入到你的源代码中。否则，如果出现问题需要切换库，你可能需要重写大量的程序代码。一个更好的主意是编写你自己的 API——一个封装外部库并将其隔离于源代码的包装器。你的程序永远不需要知道它使用了哪些外部库，只需知道你的 API 提供了哪些功能。然后，如果你需要使用不同的库，只需更改你的包装器。只要新库提供相同的功能，你就不需要触动代码库的其余部分。可能会有例外，但大多数库都是为了处理一个紧密集中的问题范围而设计的，因此可以轻松地进行隔离。

在第五章中，我们还会探讨如何利用入口点构建驱动系统，这样你就可以把项目的部分内容当作模块来随意切换。

### **包安装：从 pip 获取更多功能**

`pip` 项目提供了一种非常简单的方法来处理包和外部库的安装。它正在积极开发，得到了良好的维护，并从 Python 3.4 版本开始与 Python 一起包含。它可以从 *Python 包索引 (PyPI)*、tar 包或 `Wheel` 归档文件安装或卸载包（我们将在第五章中讨论这些）。

它的使用非常简单：

```py
$ pip install --user voluptuous
Downloading/unpacking voluptuous
  Downloading voluptuous-0.8.3.tar.gz
  Storing download in cache at ./.cache/pip/https%3A%2F%2Fpypi.python.org%2Fpa
ckages%2Fsource%2Fv%2Fvoluptuous%2Fvoluptuous-0.8.3.tar.gz
  Running setup.py egg_info for package voluptuous

Requirement already satisfied (use --upgrade to upgrade): distribute in /usr/
lib/python2.7/dist-packages (from voluptuous)
Installing collected packages: voluptuous
  Running setup.py install for voluptuous

Successfully installed voluptuous
Cleaning up...
```

通过在 PyPI 分发索引中查找，任何人都可以上传包以供其他人分发和安装，`pip install` 可以安装任何包。

你还可以提供 `--user` 选项，使 `pip` 将包安装到你的主目录中。这可以避免将包安装到系统目录中，从而污染操作系统的目录。

你可以使用 `pip freeze` 命令列出你已经安装的包，如下所示：

```py
$ pip freeze
Babel==1.3
Jinja2==2.7.1
commando=0.3.4
--snip--
```

`pip` 也支持卸载包，可以使用 `uninstall` 命令：

```py
$ pip uninstall pika-pool
Uninstalling pika-pool-0.1.3:
  /usr/local/lib/python2.7/site-packages/pika_pool-0.1.3.dist-info/
DESCRIPTION.rst
  /usr/local/lib/python2.7/site-packages/pika_pool-0.1.3.dist-info/INSTALLER
  /usr/local/lib/python2.7/site-packages/pika_pool-0.1.3.dist-info/METADATA

--snip--
Proceed (y/n)? y
  Successfully uninstalled pika-pool-0.1.3
```

`pip` 的一个非常有价值的功能是能够安装一个包而不复制包的文件。这个功能的典型使用场景是当你在积极开发一个包并且想避免每次需要测试更改时都重新安装的漫长乏味过程。你可以通过使用 `-e <directory>` 标志来实现：

```py
$ pip install -e .
Obtaining file:///Users/jd/Source/daiquiri
Installing collected packages: daiquiri
  Running setup.py develop for daiquiri
Successfully installed daiquiri
```

在这里，`pip` 不会从本地源目录复制文件，而是将一个名为 `egg-link` 的特殊文件放入你的分发路径中。例如：

```py
$ cat /usr/local/lib/python2.7/site-packages/daiquiri.egg-link
/Users/jd/Source/daiquiri
```

`egg-link` 文件包含要添加到 `sys.path` 中以查找包的路径。结果可以通过运行以下命令轻松检查：

```py
$ python -c "import sys; print('/Users/jd/Source/daiquiri' in sys.path)"
True
```

另一个有用的 `pip` 工具是 `pip install` 的 `-e` 选项，它有助于从各种版本控制系统的仓库中部署代码：支持 git、Mercurial、Subversion，甚至是 Bazaar。例如，你可以通过将仓库地址作为 URL 传递给 `-e` 选项来直接从 git 仓库安装任何库：

```py
$ pip install -e git+https://github.com/jd/daiquiri.git\#egg=daiquiri
Obtaining daiquiri from git+https://github.com/jd/daiquiri.git#egg=daiquiri
  Cloning https://github.com/jd/daiquiri.git to ./src/daiquiri
Installing collected packages: daiquiri
  Running setup.py develop for daiquiri
Successfully installed daiquiri
```

为了正确安装，你需要通过在 URL 末尾添加 `#egg=` 来提供包的 egg 名称。然后，`pip` 仅使用 `git clone` 将仓库克隆到 `src/<eggname>` 目录下，并创建一个指向该克隆目录的 `egg-link` 文件。

当依赖于未发布版本的库或在持续测试系统中工作时，这个机制非常有用。然而，由于没有版本控制，`-e` 选项也可能非常麻烦。你无法提前知道远程仓库中的下一个提交是否会破坏一切。

最后，所有其他安装工具都被弃用，推荐使用 `pip`，所以你可以放心地将它作为你所有包管理需求的一站式解决方案。

### **使用和选择框架**

Python 为各种 Python 应用程序提供了多种框架：如果你正在编写一个 Web 应用程序，可以使用 Django、Pylons、TurboGears、Tornado、Zope 或 Plone；如果你在寻找一个事件驱动框架，可以使用 Twisted 或 Circuits；等等。

框架和外部库之间的主要区别在于应用程序通过在框架上进行构建来使用框架：你的代码会扩展框架，而不是反过来。与库不同，库本质上是你可以引入的附加功能，来为你的代码增添一些额外的动力；框架则构成了你代码的 *底盘*：你所做的一切都以某种方式建立在这个底盘之上。这既有利也有弊。使用框架有许多优点，例如快速原型开发和快速开发，但也有一些值得注意的缺点，比如锁定效应。在决定是否使用框架时，你需要考虑这些因素。

在选择适合你的 Python 应用程序的框架时，要检查的推荐内容大体上与在《"外部库安全检查清单"》第 23 页 中描述的内容相同——这也说得通，因为框架通常作为 Python 库的捆绑包分发。有时框架还包括用于创建、运行和部署应用程序的工具，但这并不改变你应该应用的标准。我们已经确认，替换你已经编写了代码并依赖于的外部库是件麻烦事，但替换框架则更糟，通常需要从头开始完全重写程序。

举个例子，前面提到的 Twisted 框架仍然没有完全支持 Python 3：如果你几年前写了一个使用 Twisted 的程序并希望将其更新以在 Python 3 上运行，那么你可能会很失望。你要么需要重写整个程序以使用另一个框架，要么只能等到有人最终升级 Twisted，完全支持 Python 3。

一些框架比其他框架更轻量。例如，Django 有自己内置的 ORM 功能；而 Flask 则完全没有这种功能。框架为你做的事情越少，未来你遇到的问题就会越少。然而，每个框架缺失的功能都意味着你需要解决另一个问题，可能是通过编写自己的代码，或者麻烦地挑选另一个库来处理它。选择处理哪个场景完全取决于你，但请谨慎选择：当事情变得糟糕时，从框架迁移出去可能是一个艰巨的任务，且即使有其他所有功能，Python 也没有什么能够帮助你应对这种情况的工具。

### **Doug Hellmann，Python 核心开发者，谈 Python 库**

Doug Hellmann 是 DreamHost 的高级开发人员，同时也是 OpenStack 项目的贡献者。他创办了名为 Python Module of the Week 的网站 (*[`www.pymotw.com/`](http://www.pymotw.com/)*)，并撰写了一本优秀的书籍《*Python 标准库实例解析*》。他还是 Python 核心开发者。我向 Doug 提出了一些关于标准库以及围绕它设计库和应用程序的问题。

**从零开始编写 Python 应用程序时，你的第一步是什么？**

从零开始编写应用程序的步骤与修改现有应用程序在抽象层面上相似，但细节有所不同。

当我更改现有代码时，我首先弄清楚它是如何工作的，以及我的更改需要放在哪里。我可能会使用一些调试技术：添加日志或打印语句，或使用 `pdb`，并使用测试数据运行应用程序，以确保我理解它的运行方式。我通常会先手动做出更改并测试，然后再添加任何自动化测试，最后再提交补丁。

当我创建一个新应用程序时，我采取相同的探索性方法——先编写一些代码并手动运行，等基本功能实现后，我编写测试以确保覆盖所有边界情况。创建测试的过程也可能导致一些重构，使代码更易于使用。

这在 smiley [一个用于监视 Python 程序并记录其活动的工具] 中的确如此。我从实验 Python 的 trace API 开始，使用一些临时脚本，然后才构建真正的应用程序。最初，我打算有一个部分用于插装并收集另一个正在运行的应用程序的数据，另一个部分则用于收集通过网络发送的数据并保存它。在添加了一些报告功能后，我意识到重播收集数据的处理几乎与收集数据时的处理完全相同。我重构了几个类，并能够创建一个基类来处理数据收集、数据库访问和报告生成。使这些类符合相同的 API，使我能够轻松创建一个直接将数据写入数据库而不是通过网络发送信息的数据收集应用程序版本。

在设计应用程序时，我会考虑用户界面是如何工作的，但对于库，我更关注开发者将如何使用 API。有时，先为将要使用新库的程序编写测试，再编写库代码会更容易。我通常会创建一系列测试形式的示例程序，然后构建库来支持这些程序。

我还发现，在编写代码之前先为库编写文档有助于我思考功能和工作流，而不必立即决定实现细节，同时它也让我记录下我在设计中做出的选择，让读者理解不仅是如何使用库，还能理解我在创建库时的期望。

**将一个模块添加到 Python 标准库的过程是什么？**

提交模块到标准库的完整过程和指南可以在 Python 开发者指南中找到，网址是 *[`docs.python.org/devguide/stdlibchanges.html`](https://docs.python.org/devguide/stdlibchanges.html)*。

在一个模块被添加之前，提交者需要证明它是稳定且广泛有用的。该模块应提供一些功能，这些功能要么是自己实现起来很难正确完成，要么是非常有用，很多开发者已经创建了自己的变体。API 应该清晰，任何模块的依赖项应仅限于标准库内部。

第一步是通过*python-ideas*邮件列表在社区中提出引入该模块到标准库的想法，非正式地评估大家的兴趣。如果反响积极，下一步是创建一个 Python 增强提案（PEP），其中应包括添加该模块的动机和实现细节，说明如何进行过渡。

由于包管理和发现工具已经变得非常可靠，尤其是`pip`和 PyPI，因此将新的库维护在 Python 标准库之外可能更为实际。单独发布允许更频繁地更新新特性和修复 bug，这对于处理新技术或 API 的库尤其重要。

**你希望人们更多了解的标准库中的前三个模块是什么？**

来自标准库的一个非常有用的工具是`abc`模块。我使用`abc`模块将动态加载的扩展的 API 定义为抽象基类，以帮助扩展作者理解哪些 API 方法是必需的，哪些是可选的。抽象基类内置于一些其他面向对象编程（OOP）语言中，但我发现很多 Python 程序员不知道我们也有它们。

`bisect`模块中的二分查找算法是一个很好的例子，它是一个有用的特性，但经常被错误实现，因此非常适合放入标准库。我特别喜欢它可以搜索稀疏列表，其中搜索值可能不在数据中。

在`collections`模块中有一些有用的数据结构，它们的使用频率并不像应有的那样高。我喜欢使用`namedtuple`来创建小型、类似类的数据结构，用于存储数据而不包含任何相关的逻辑。如果以后需要添加逻辑，将`namedtuple`转换为常规类非常简单，因为`namedtuple`支持通过名称访问属性。这个模块中的另一个有趣的数据结构是`ChainMap`，它非常适合做堆叠式命名空间。`ChainMap`可以用于创建渲染模板的上下文，或用于管理来自不同来源的配置设置，并明确规定优先级。

**很多项目，包括 OpenStack 和外部库，都在标准库之上构建了自己的抽象层，比如日期/时间处理等方面。你认为程序员应该坚持使用标准库，自己编写函数，切换到外部库，还是开始向 Python 提交补丁？**

上述所有建议！我倾向于避免重复造轮子，因此我强烈主张将修复和增强贡献给上游项目，这些项目可以作为依赖项使用。另一方面，有时创建另一个抽象并将代码单独维护，无论是在应用程序中还是作为新库，都有其合理性。

在你示例中使用的`timeutils`模块是对 Python `datetime`模块的一个相对薄的封装。大多数函数简短且简单，但创建一个包含最常见操作的模块可以确保它们在所有项目中一致地处理。由于很多函数是应用特定的，意味着它们强制决定了像时间戳格式字符串或“现在”意味着什么这样的事项，因此它们并不适合作为补丁贡献到 Python 库，或者作为一个通用库发布并被其他项目采用。

相比之下，我一直在努力将 OpenStack 中的 API 服务从项目初期创建的 WSGI [Web Server Gateway Interface]框架迁移到第三方 Web 开发框架。在 Python 中创建 WSGI 应用程序有很多选择，虽然我们可能需要增强其中一个框架，使其完全适用于 OpenStack 的 API 服务器，但将这些可重用的更改贡献到上游比维护一个“私有”框架更为可取。

**对于在主要 Python 版本之间犹豫不决的开发者，你有什么建议？**

支持 Python 3 的第三方库数量已经达到了临界点。现在比以往任何时候都更容易为 Python 3 构建新的库和应用程序，而且得益于 3.3 中新增的兼容性功能，维护对 Python 2.7 的支持也变得更容易。主要的 Linux 发行版正在致力于发布默认安装 Python 3 的版本。任何开始新项目的 Python 开发者都应该认真考虑 Python 3，除非他们有尚未迁移的依赖项。不过，到目前为止，不能运行在 Python 3 上的库几乎可以被归类为“未维护”的库。

**从设计、提前规划、迁移等方面来看，将代码从应用程序分支到库的最佳方法是什么？**

应用程序是将库按特定目的组合在一起的“胶水代码”集合。先将应用程序设计为具有实现该目的的功能的库，然后再构建应用程序，确保代码被正确地组织成逻辑单元，从而简化测试。这也意味着应用程序的功能可以通过库访问，并且可以重新组合以创建其他应用程序。如果不采取这种方法，就有可能使应用程序的功能与用户界面紧密绑定，从而使它们更难以修改和重用。

**你会给计划设计自己 Python 库的人什么建议？**

我总是建议从上到下设计库和 API，在每一层应用设计标准，例如单一职责原则（SRP）。考虑调用者希望如何使用库，并创建支持这些功能的 API。考虑实例中可以存储哪些值并由方法使用，以及每次方法调用时需要传递哪些参数。最后，考虑实现方式以及底层代码是否应该与公共 API 的代码组织方式有所不同。

SQLAlchemy 是一个应用这些准则的优秀示例。声明式 ORM [对象关系映射]、数据映射和表达式生成层都是分开的。开发者可以根据自己的需求而非库设计的限制，决定进入 API 和使用库的合适抽象层次。

**你在阅读 Python 开发者代码时，遇到的最常见编程错误是什么？**

Python 的惯用法在循环和迭代方面与其他语言有显著不同。例如，我看到的最常见的反模式之一是使用 `for` 循环过滤列表，方法是首先将项附加到新列表中，然后在第二个循环中处理结果（可能是在将列表作为参数传递给函数之后）。我几乎总是建议将这种过滤循环转换为生成器表达式，这样既更高效又更容易理解。也常见将多个列表合并在一起，以便它们的内容可以以某种方式一起处理，而不是使用 `itertools.chain()`。

在代码审查中，我经常建议其他一些更微妙的改进，比如使用 `dict()` 作为查找表，而不是使用长的 `if:then:else` 语句块，确保函数总是返回相同类型的对象（例如，返回一个空列表而不是 `None`），通过将相关值组合成元组或新类的对象来减少函数所需的参数数量，以及定义类以用于公共 API，而不是依赖字典。

**你对框架的看法是什么？**

框架就像其他任何工具一样。它们能提供帮助，但在选择框架时，你需要小心，确保它适合当前的工作需求。

将应用程序的公共部分提取到框架中有助于你将开发精力集中在应用程序的独特方面。框架还提供了大量的引导代码，用于执行像在开发模式下运行和编写测试套件等任务，帮助你更快地将应用程序带入有用的状态。它们还鼓励应用程序实现的一致性，这意味着你最终得到的代码更容易理解和复用。

当然，也存在一些潜在的陷阱。选择使用特定框架通常意味着对应用程序设计的某些假设。选择错误的框架可能会使应用程序的实现变得更加困难，特别是当这些设计约束与应用程序的需求不自然契合时。如果你尝试使用与框架推荐的模式或惯用法不同的方式，可能会发现自己不得不与框架“斗争”。
