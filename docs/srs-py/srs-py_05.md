## **5**

**分发你的软件**

![image](img/common01.jpg)

可以肯定地说，在某个时刻，你会希望分发你的软件。虽然你可能会忍不住直接将代码打包并上传到互联网，Python 提供了工具来帮助你的最终用户更轻松地使用你的软件。你应该已经熟悉使用 *setup.py* 来安装 Python 应用程序和库，但你可能从未深入了解过它的背后原理或如何创建你自己的 *setup.py*。

本章将介绍 *setup.py* 的历史、它是如何工作的以及如何创建你自己的定制 *setup.py*。我们还将看一些包安装工具 pip 的不太为人知的功能，以及如何通过 pip 让你的软件可以下载。最后，我们将了解如何使用 Python 的入口点使得函数在程序之间更容易找到。有了这些技能，你就可以让发布的软件更容易为最终用户所使用。

### **setup.py 的一点历史**

distutils 库最初由软件开发者 Greg Ward 创建，自 1998 年以来一直是标准 Python 库的一部分。Ward 希望为开发者提供一种简便的方式，帮助他们自动化为最终用户安装软件包的过程。软件包提供 *setup.py* 文件作为标准的 Python 脚本进行安装，开发者可以使用 distutils 来进行安装，如 Listing 5-1 所示。

#!/usr/bin/python

from distutils.core import setup

setup(name="rebuildd",

description="Debian 软件包重建工具",

author="Julien Danjou",

author_email="acid@debian.org",

url="http://julien.danjou.info/software/rebuildd.html",

packages=['rebuildd'])

*Listing 5-1: 使用 distutils 构建 setup.py*

以 *setup.py* 文件为项目根基，用户只需运行该文件并传入相应的命令作为参数，即可构建或安装你的软件。即使你的分发版除了原生 Python 模块外还包含 C 模块，distutils 也能自动处理它们。

distutils 的开发在 2000 年被放弃；从那时起，其他开发者接手了它的后续工作。一个显著的继承者是名为 setuptools 的打包库，它提供了更频繁的更新和更多高级功能，比如自动处理依赖关系、Egg 分发格式以及 easy_install 命令。由于 distutils 在开发时仍是 Python 标准库中接受的软件打包方式，setuptools 提供了与其一定程度的向后兼容性。 Listing 5-2 展示了如何使用 setuptools 构建与 Listing 5-1 中相同的安装包。

#!/usr/bin/env python

import setuptools

setuptools.setup(

name="rebuildd",

version="0.2",

author="Julien Danjou",

author_email="acid@debian.org",

description="Debian 软件包重建工具",

license="GPL",

url="http://julien.danjou.info/software/rebuildd/",

packages=['rebuildd'],

classifiers=[

"开发状态 :: 2 - 初期版本",

"目标受众 :: 开发人员",

"目标受众 :: 信息技术",

"许可证 :: OSI 认可 :: GNU 通用公共许可证（GPL）", "操作系统 :: 跨平台",

"编程语言 :: Python"

],

)

*列表 5-2：使用 setuptools 构建 setup.py*

最终，setuptools 的开发也放缓了，但很快，另一组开发人员将其分支出来，创建了一个名为 distribute 的新库，该库在 setuptools 的基础上提供了几个优点，包括减少了 bug 和对 Python 3 的支持。

然而，所有最好的故事都有反转：在 2013 年 3 月，setuptools 和 distribute 背后的团队决定将他们的代码库合并到原始 setuptools 项目下。因此，distribute 现在已被弃用，setuptools 再次成为处理高级 Python 安装的规范方式。

在这一切发生的同时，另一个名为 distutils2 的项目也在开发，目的是完全取代 Python 标准库中的 distutils。与 distutils 和 setuptools 不同，distutils2 将包的元数据存储在纯文本文件 *setup.cfg* 中，这既便于开发人员编写，也便于外部工具读取。然而，distutils2 保留了 distutils 的一些缺点，例如它难以理解的基于命令的设计，并且缺乏对入口点和 Windows 本地脚本执行的支持——这两项功能是 setuptools 提供的。由于这些原因，原本计划将 distutils2（更名为 packaging）纳入 Python 3.3 标准库的计划未能实现，该项目在 2012 年被放弃。

通过 distlib，打破困境的打包工具仍有机会从废墟中崛起，这是一个旨在取代 distutils 的新兴项目。在发布之前，有传言称 distlib 包将成为 Python 3.4 标准库的一部分，但这一计划并未实现。distlib 在集成打包的最佳特性基础上，实施了打包相关 PEPs 中描述的基本工作框架。

所以，简而言之：

+   distutils 是 Python 标准库的一部分，可以处理简单的包安装。

+   setuptools，作为高级包安装的标准，最初被弃用，但现在已经重新进入积极开发阶段，并成为事实上的标准。

+   distribute 已在版本 0.7 中并入 setuptools；distutils2（即 packaging）已被废弃。

+   distlib *可能* 在未来取代 distutils。

还有其他打包库，但这五个是你最常遇到的。在互联网上查找这些库时要小心：由于上文提到的复杂历史，很多文档已经过时。不过，官方文档是最新的。

简而言之，setuptools 目前是使用的打包库，但未来要留意 distlib 的发展。

### **使用 setup.cfg 进行打包**

你可能已经尝试过为某个包编写 *setup.py*，无论是通过复制另一个项目的文件，还是浏览文档后自己动手编写。构建 *setup.py* 不是一项直观的任务。选择合适的工具就是第一大挑战。在本节中，我想向你介绍 setuptools 最近的改进之一：*setup.cfg* 文件支持。

这是一个使用 *setup.cfg* 文件的 *setup.py* 示例：

import setuptools

setuptools.setup()

两行代码——就这么简单。setup 所需的实际元数据存储在 *setup.cfg* 中，如 Listing 5-3 所示。

[metadata]

name = foobar

author = Dave Null

author-email = foobar@example.org

license = MIT

long_description = file: README.rst

url = http://pypi.python.org/pypi/foobar

requires-python = >=2.6

classifiers =

Development Status :: 4 - Beta

Environment :: Console

Intended Audience :: Developers

Intended Audience :: 信息技术

License :: OSI 批准 :: Apache 软件许可证

Operating System :: OS 独立

Programming Language :: Python

*Listing 5-3: setup.cfg 元数据*

如你所见，*setup.cfg* 使用了一种易于编写和阅读的格式，直接受 distutils2 的启发。许多其他工具，例如 Sphinx 或 Wheel，也从这个 *setup.cfg* 文件中读取配置——这本身就是一个很好的理由，值得开始使用它。

在 Listing 5-3 中，项目描述是从 *README.rst* 文件中读取的。保持一个 README 文件是一个好的实践——最好使用 RST 格式——这样用户可以快速了解项目内容。仅凭这些基本的 *setup.py* 和 *setup.cfg* 文件，你的包就已经准备好发布并供其他开发者和应用使用。如果需要，setuptools 文档提供了更多细节，例如，如果你的安装过程有额外步骤，或者你想包括额外的文件。

另一个有用的打包工具是 pbr，*Python Build Reasonableness* 的缩写。该项目最初在 OpenStack 中启动，作为 setuptools 的扩展，旨在简化包的安装和部署。与 setuptools 一起使用的 pbr 打包工具，补充了 setuptools 中缺失的一些功能，包括以下内容：

+   自动生成 Sphinx 文档

+   基于 git 历史自动生成 *AUTHORS* 和 *ChangeLog* 文件

+   自动创建 git 的文件列表

+   基于 git 标签的版本管理，采用语义化版本控制

而且这一切几乎不需要你额外的努力。要使用 pbr，你只需要启用它，如 Listing 5-4 所示。

import setuptools

setuptools.setup(setup_requires=['pbr'], pbr=True)

*Listing 5-4: 使用 pbr 的 setup.py*

setup_requires 参数告诉 setuptools，必须先安装 pbr 才能使用 setuptools。pbr=True 参数确保加载并调用 setuptools 的 pbr 扩展。

启用后，python setup.py 命令会增强 pbr 功能。例如，调用 python setup.py --version 将返回基于现有 git 标签的项目版本号。运行 python setup.py sdist 将创建一个源 tarball，并自动生成 *ChangeLog* 和 *AUTHORS* 文件。

### **Wheel 格式分发标准**

在 Python 大部分历史中，并没有官方的标准分发格式。虽然不同的分发工具通常使用一些通用的归档格式——甚至 setuptools 引入的 Egg 格式只是一个具有不同扩展名的 zip 文件——但它们的元数据和包结构是彼此不兼容的。这个问题在 PEP 376 定义了一个官方安装标准时变得更加复杂，因为这个标准也与现有格式不兼容。

为了解决这些问题，PEP 427 被编写出来，定义了一种新的 Python 分发包标准，叫做 Wheel。该格式的参考实现作为一个工具提供，也叫做 Wheel。

Wheel 从 pip 版本 1.4 开始得到支持。如果你正在使用 setuptools 并且安装了 Wheel 包，它会自动作为一个名为 bdist_wheel 的 setuptools 命令进行集成。如果你没有安装 Wheel，可以通过命令 pip install wheel 来安装它。清单 5-5 显示了调用 bdist_wheel 时的部分输出，已缩短以便打印。

$ python setup.py bdist_wheel

运行 bdist_wheel

运行构建

运行 build_py

创建 build/lib

创建 build/lib/daiquiri

创建 build/lib/daiquiri/tests

复制 daiquiri/tests/__init__.py -> build/lib/daiquiri/tests

--省略--

运行 egg_info

写入依赖项到 daiquiri.egg-info/requires.txt

写入 daiquiri.egg-info/PKG-INFO

将顶级名称写入 daiquiri.egg-info/top_level.txt

将 dependency_links 写入 daiquiri.egg-info/dependency_links.txt

写入 pbr 到 daiquiri.egg-info/pbr.json

写入清单文件 'daiquiri.egg-info/SOURCES.txt'

安装到 build/bdist.macosx-10.12-x86_64/wheel

运行安装

运行 install_lib

--省略--

运行 install_scripts

创建 build/bdist.macosx-10.12-x86_64/wheel/daiquiri-1.3.0.dist-info/WHEEL

➊ 创建 '/Users/jd/Source/daiquiri/dist/daiquiri-1.3.0-py2.py3-none-any.whl'

并添加 '.' 到它

添加 'daiquiri/__init__.py'

添加 'daiquiri/formatter.py'

添加 'daiquiri/handlers.py'

--省略--

*清单 5-5：调用 setup.py bdist_wheel*

bdist_wheel 命令会在 *dist* 目录 ➊ 中创建一个 *.whl* 文件。与 Egg 格式一样，Wheel 归档也是一个具有不同扩展名的 zip 文件。然而，Wheel 归档不需要安装——你只需添加一个斜杠，后跟模块名，就可以加载并运行代码：

$ python wheel-0.21.0-py2.py3-none-any.whl/wheel -h

用法: wheel [-h]

{keygen,sign,unsign,verify,unpack,install,install-

脚本,转换,帮助}

--省略--

定位参数：

--省略--

你可能会惊讶地发现，这并不是 Wheel 格式本身引入的功能。Python 也可以运行常规的 zip 文件，就像 Java 的 *.jar* 文件一样：

python foobar.zip

这等同于：

PYTHONPATH=foobar.zip python -m __main__

换句话说，你程序的 __main__ 模块会自动从 __main__.py 中导入。你也可以通过附加一个斜杠后跟模块名来从你指定的模块导入 __main__，就像 Wheel 一样：

python foobar.zip/mymod

这等同于：

PYTHONPATH=foobar.zip python -m mymod.__main__

Wheel 的一个优点是它的命名约定允许你指定你的分发是否面向特定架构和/或 Python 实现（如 CPython、PyPy、Jython 等）。如果你需要分发用 C 编写的模块，这特别有用。

默认情况下，Wheel 包与构建它们时使用的 Python 主版本绑定。当用 python2 setup.py bdist_wheel 调用时，Wheel 文件名的模式类似于 *library-version-py2-none-any.whl*。

如果你的代码兼容所有主流 Python 版本（即 Python 2 和 Python 3），你可以构建一个通用的 Wheel：

python setup.py bdist_wheel --universal

生成的文件名会有所不同，并且包含了两个 Python 主版本——类似于*library-version-py2.py3-none-any.whl*。构建一个通用的 Wheel 可以避免只用一个 Wheel 覆盖两个 Python 主版本时，结果却生成两个不同的 Wheel。

如果你不想在每次构建 Wheel 时都传递 --universal 标志，你可以将以下内容添加到你的*setup.cfg*文件中：

[wheel]

universal=1

如果你构建的 Wheel 包含二进制程序或库（例如用 C 编写的 Python 扩展），那么这个二进制 Wheel 可能并不像你想象的那样具有可移植性。它默认在一些平台上可用，比如 Darwin（macOS）或 Microsoft Windows，但可能在所有 Linux 发行版上都无法工作。PEP 513 (*[`www.python.org/dev/peps/pep-0513`](https://www.python.org/dev/peps/pep-0513)*) 针对这个 Linux 问题，通过定义一个名为 manylinux1 的新平台标签以及保证在该平台上可用的最小库集合来解决这个问题。

Wheel 是一个很好的格式，用于分发可直接安装的库和应用程序，因此建议你也将它们构建并上传到 PyPI。

### **与世界分享你的工作**

一旦你有了一个合适的 *setup.py* 文件，构建一个可以分发的源代码 tarball 变得很容易。sdist setuptools 命令正是做了这件事，正如 Listing 5-6 中所示。

$ python setup.py sdist

正在运行 sdist

[pbr] 正在生成 AUTHORS

正在运行 egg_info

正在将依赖项写入 ceilometer.egg-info/requires.txt

正在将 ceilometer.egg-info/PKG-INFO 写入

正在将顶级名称写入 ceilometer.egg-info/top_level.txt

正在将依赖链接写入 ceilometer.egg-info/dependency_links.txt

正在将入口点写入 ceilometer.egg-info/entry_points.txt

[pbr] 处理 SOURCES.txt

[pbr] 在 git 上生成文件列表

警告：没有找到任何匹配 '*.pyc' 的文件

发布

写入清单文件 'ceilometer.egg-info/SOURCES.txt'

运行检查

复制 setup.cfg -> ceilometer-2014.1.a6-g772e1a7

写入 ceilometer-2014.1.a6-g772e1a7/setup.cfg

--snip--

创建 tar 压缩包

移除 'ceilometer-2014.1.a6.g772e1a7'（以及其下的所有内容）

*清单 5-6：使用 setup.py sdist 构建源代码 tar 包*

sdist 命令会在源代码树的 *dist* 目录下创建一个 tar 包。这个 tar 包包含了所有源代码树中的 Python 模块。如前一节所述，你也可以使用 bdist_wheel 命令构建 Wheel 格式的归档。Wheel 格式的归档安装速度更快，因为它们已经是安装所需的正确格式。

让代码可以访问的最后一步是将你的包导出到用户可以通过 pip 安装的地方。这意味着将项目发布到 PyPI。

如果这是你第一次导出到 PyPI，最好在一个安全的沙盒中测试发布过程，而不是在生产服务器上。你可以使用 PyPI 的测试服务器进行此目的；它复制了主索引的所有功能，但仅用于测试。

第一步是将项目注册到测试服务器。首先打开你的 *~/.pypirc* 文件，并添加以下几行：

[distutils]

index-servers =

testpypi [testpypi]

用户名 = <你的用户名>

password = <你的密码>

仓库 = https://testpypi.python.org/pypi

保存文件，现在你可以在索引中注册你的项目了：

$ python setup.py register -r testpypi

运行注册

运行 egg_info

写入依赖要求到 ceilometer.egg-info/requires.txt

写入 ceilometer.egg-info/PKG-INFO

写入顶级名称到 ceilometer.egg-info/top_level.txt

写入依赖链接到 ceilometer.egg-info/dependency_links.txt

写入入口点到 ceilometer.egg-info/entry_points.txt

[pbr] 重用现有的 SOURCES.txt

运行检查

正在注册 ceilometer 到 https://testpypi.python.org/pypi

服务器响应（200）：OK

这会连接到测试 PyPI 服务器实例并创建一个新条目。别忘了使用 -r 选项；否则会使用真实的生产 PyPI 实例！

显然，如果一个同名的项目已经在那注册过了，过程将会失败。请重新尝试使用一个新的名称，一旦你成功注册了程序并收到 OK 响应，你就可以上传源代码的 tar 包，参考清单 5-7。

$ python setup.py sdist upload -r testpypi

运行 sdist

[pbr] 写入 ChangeLog

[pbr] 生成 AUTHORS

运行 egg_info

写入依赖要求到 ceilometer.egg-info/requires.txt

写入 ceilometer.egg-info/PKG-INFO

写入顶级名称到 ceilometer.egg-info/top_level.txt

写入依赖链接到 ceilometer.egg-info/dependency_links.txt

写入入口点到 ceilometer.egg-info/entry_points.txt

[pbr] 处理中 SOURCES.txt

[pbr] 在 git 环境中，从 git 生成文件列表

警告：没有找到匹配 '*.pyc' 的先前包含文件

分发

正在写入清单文件 'ceilometer.egg-info/SOURCES.txt'

正在运行检查

正在创建 ceilometer-2014.1.a6.g772e1a7

--省略--

正在复制 setup.cfg -> ceilometer-2014.1.a6.g772e1a7

正在写入 ceilometer-2014.1.a6.g772e1a7/setup.cfg

正在创建 tar 存档

正在移除 'ceilometer-2014.1.a6.g772e1a7'（以及其下所有内容）

正在上传 提交 dist/ceilometer-2014.1.a6.g772e1a7.tar.gz 到 https://testpypi

.python.org/pypi

服务器响应 (200)：OK

*清单 5-7：将您的 tarball 上传到 PyPI*

另外，您也可以上传一个 Wheel 存档，如 清单 5-8 所示。

$ python setup.py bdist_wheel upload -r testpypi

正在运行 bdist_wheel

正在运行构建

正在运行 build_py

正在运行 egg_info

正在将要求写入 ceilometer.egg-info/requires.txt

正在写入 ceilometer.egg-info/PKG-INFO

正在写入顶级名称到 ceilometer.egg-info/top_level.txt

正在将依赖关系链接写入 ceilometer.egg-info/dependency_links.txt

正在将入口点写入 ceilometer.egg-info/entry_points.txt

[pbr] 正在重用现有 SOURCES.txt

正在安装到 build/bdist.linux-x86_64/wheel

正在运行安装

正在运行 install_lib

正在创建 build/bdist.linux-x86_64/wheel

--省略--

正在创建 build/bdist.linux-x86_64/wheel/ceilometer-2014.1.a6.g772e1a7

.dist-info/WHEEL

正在运行上传

正在提交 /home/jd/Source/ceilometer/dist/ceilometer-2014.1.a6

.g772e1a7-py27-none-any.whl 到 https://testpypi.python.org/pypi

服务器响应 (200)：OK

*清单 5-8：将 Wheel 存档上传到 PyPI*

完成这些操作后，您和其他用户可以在 PyPI 阶段服务器上搜索上传的包，甚至可以通过指定测试服务器并使用 -i 选项，使用 pip 安装这些包：

$ pip install -i https://testpypi.python.org/pypi ceilometer

如果一切正常，您可以将项目上传到主 PyPI 服务器。只需确保首先将您的凭据和服务器详细信息添加到 ~/.pypirc 文件，如下所示：

[distutils]

索引服务器 =

pypi

testpypi

[pypi]

用户名 = <您的用户名>

密码 = <您的密码> [testpypi]

仓库 = https://testpypi.python.org/pypi

用户名 = <您的用户名>

密码 = <您的密码>

现在，如果您使用 -r pypi 切换运行注册和上传命令，您的包应上传到 PyPI。

**注意**

*PyPI 可以在其索引中保留多个版本的您的软件，允许您在需要时安装特定或较旧的版本。只需在 pip install 命令中传递版本号；例如，pip install foobar==1.0.2。*

这个过程简单易用，允许进行任意次数的上传。您可以根据需要频繁发布您的软件，用户也可以随时安装和更新。

### **入口点**

你可能已经在不知情的情况下使用过 setuptools 入口点。使用 setuptools 分发的软件包括描述其所需依赖项等特性的重要元数据——更与本主题相关的是，包含了一个 *入口点* 的列表。入口点是其他 Python 程序可以通过它们发现一个包所提供的动态特性的方式。

以下示例展示了如何在 console_scripts 入口点组中提供一个名为 rebuildd 的入口点：

#!/usr/bin/python

from distutils.core import setup

setup(name="rebuildd",

description="Debian 包重建工具",

author="Julien Danjou",

author_email="acid@debian.org",

url="http://julien.danjou.info/software/rebuildd.html",

entry_points={

'console_scripts': [

'rebuildd = rebuildd:main',

],

},

packages=['rebuildd'])

任何 Python 包都可以注册入口点。入口点按组进行组织：每组由一系列键值对组成。这些键值对采用 path.to.module:variable_name 的格式。在前面的示例中，键是 rebuildd，值是 rebuildd:main。

入口点的列表可以使用各种工具进行操作，从 setuptools 到 epi，正如我在这里将展示的那样。在接下来的部分中，我们将讨论如何利用入口点为我们的软件添加扩展性。

#### ***可视化入口点***

查看包中可用入口点的最简单方法是使用名为 entry point inspector 的包。你可以通过运行 `pip install entry-point-inspector` 来安装它。安装后，它提供了一个名为 epi 的命令，你可以从终端运行该命令，交互式地发现已安装软件包提供的入口点。列表 5-9 展示了我在系统上运行 epi group list 的示例。

$ epi group list

---------------------------

| 名称                    |

--------------------------

| console_scripts |
| --- |
| distutils.commands |
| distutils.setup_keywords |
| egg_info.writers |
| epi.commands |
| flake8.extension |
| setuptools.file_finders |
| setuptools.installation |

--------------------------

*列表 5-9：获取入口点组的列表*

列表 5-9 中来自 epi group list 的输出显示了系统中提供入口点的不同软件包。表格中的每一项是一个入口点组的名称。请注意，此列表包括 console_scripts，我们稍后会讨论。我们可以使用 epi 命令与 show 命令来显示特定入口点组的详细信息，如列表 5-10 所示。

$ epi group show console_scripts

-------------------------------------------------

| 名称     | 模块   | 成员 | 分发版本 | 错误 |
| --- | --- | --- | --- | --- |

-------------------------------------------------

| coverage | coverage | main   | coverage 3.4 |       |

*列表 5-10：显示入口点组的详细信息*

我们可以看到，在 group console_scripts 中，有一个名为 coverage 的入口点指向模块 coverage 的 main 成员。这个特定的入口点由 coverage 3.4 包提供，指示在执行命令行脚本 coverage 时调用哪个 Python 函数。在这里，调用的函数是 coverage.main。

epi 工具只是完整 Python 库 pkg_resources 上的一层薄薄的封装。这个模块允许我们发现任何 Python 库或程序的入口点。入口点对于多种用途都很有价值，包括控制台脚本和动态代码发现，正如你将在接下来的几节中看到的。

#### ***使用控制台脚本***

在编写 Python 应用程序时，几乎总是需要提供一个可启动的程序——一个最终用户可以运行的 Python 脚本——该脚本需要安装在系统路径的某个目录中。

大多数项目都有一个类似这样的可启动程序：

#!/usr/bin/python

import sys

import mysoftware

mysoftware.SomeClass(sys.argv).run()

这种脚本是最理想的情况：许多项目在系统路径中安装的是更长的脚本。然而，这种脚本也带来了一些重大问题：

+   用户无法知道 Python 解释器的位置或它使用的是哪个版本。

+   该脚本泄露了无法被软件或单元测试导入的二进制代码。

+   没有简单的方法来定义脚本的安装位置。

+   这种方式并不明显如何以可移植的方式安装（例如，既可以在 Unix 上，也可以在 Windows 上）。

为了帮助我们绕过这些问题，setuptools 提供了 console_scripts 功能。这个入口点可以用来让 setuptools 在系统路径中安装一个小程序，该程序调用你模块中的特定函数。使用 setuptools，你可以通过在 console_scripts 入口点组中设置键/值对来指定启动程序的函数调用：键是将要安装的脚本名称，值是指向你函数的 Python 路径（类似 my_module.main）。

假设有一个 foobar 程序，它由客户端和服务器两部分组成。每一部分都写在自己的模块中——分别是 *foobar.client* 和 *foobar.server*，位于 *foobar/client.py* 中：

def main():

print("Client started")

以及在 *foobar/server.py* 中：

def main():

print("Server started")

当然，这个程序并没有做太多的事情——我们的客户端和服务器甚至没有互相通信。对于我们的例子来说，它们只需要打印一条消息，告诉我们它们已经成功启动。

我们现在可以在根目录中编写以下 *setup.py* 文件，其中在 *setup.py* 中定义了入口点。

from setuptools import setup

setup(

name="foobar",

version="1",

description="Foo!",

author="Julien Danjou",

author_email="julien@danjou.info",

packages=["foobar"],

entry_points={

"console_scripts": [

➊ "foobard = foobar.server:main",

"foobar = foobar.client:main",

],

},

)

我们使用格式 module.submodule:function 来定义入口点。你可以看到，这里我们为客户端和服务器分别定义了一个入口点 ➊。

当运行 python setup.py install 时，setuptools 将创建一个类似 Listing 5-11 中的脚本。

#!/usr/bin/python

# EASY-INSTALL-ENTRY-SCRIPT: 'foobar==1','console_scripts','foobar'

__requires__ = 'foobar==1'

import sys

from pkg_resources import load_entry_point

if __name__ == '__main__':

sys.exit(

load_entry_point('foobar==1', 'console_scripts', 'foobar')()

)

*Listing 5-11: setuptools 生成的控制台脚本*

这段代码扫描 foobar 包的入口点，并从 console_scripts 组中检索 foobar 键，用于定位并运行相应的函数。load_entry_point 的返回值将是对 foobar.client.main 函数的引用，该函数将不带任何参数地被调用，并且它的返回值将作为退出码。

请注意，这段代码使用 pkg_resources 来发现并加载 Python 程序中的入口点文件。

**注意**

*如果你在 setuptools 上使用 pbr，生成的脚本比 setuptools 默认构建的脚本更简单（因此更快），因为它会直接调用你在入口点中编写的函数，而无需在运行时动态搜索入口点列表。*

使用控制台脚本是一种技术，它消除了编写可移植脚本的负担，同时确保你的代码保留在 Python 包中，并且可以被其他程序导入（和测试）。

#### ***使用插件和驱动程序***

入口点使得发现和动态加载由其他包部署的代码变得容易，但这并不是它们唯一的用途。任何应用程序都可以提议并注册入口点和组，然后按需使用它们。

在本节中，我们将创建一个类似 cron 风格的守护进程 pycrond，它将允许任何 Python 程序通过在 pytimed 组中注册入口点来注册一个命令，使得该命令每隔几秒钟执行一次。该入口点所指示的属性应为一个返回 number_of_seconds 和 callable 的对象。

这是我们使用 pkg_resources 来发现入口点并在一个名为*pytimed.py*的程序中实现 pycrond 的代码：

import pkg_resources

import time

def main():

seconds_passed = 0

while True:

for entry_point in pkg_resources.iter_entry_points('pytimed'):

try:

seconds, callable = entry_point.load()()

except:

# 忽略失败

pass

else:

if seconds_passed % seconds == 0:

callable()

time.sleep(1)

seconds_passed += 1

该程序由一个无限循环组成，循环遍历 pytimed 组的每个入口点。每个入口点通过 load()方法加载。程序随后调用返回的方法，该方法需要返回等待的秒数，并且还需要返回前面提到的可调用对象。

*pytimed.py* 中的程序是一个非常简化且天真的实现，但足以满足我们的示例需求。现在我们可以编写另一个 Python 程序，命名为 *hello.py*，它需要定期调用其中的某个函数：

定义 print_hello():

print("你好，世界！")

定义 say_hello():

返回 2, print_hello

一旦我们定义了该函数，就使用适当的扩展点在 *setup.py* 中注册它。

从 setuptools 导入 setup

setup(

name="hello",

version="1",

packages=["hello"],

entry_points={

"pytimed": [

"hello = hello:say_hello",

],

},)

*setup.py* 脚本在 pytimed 组中注册了一个名为 hello 的扩展点，其值指向函数 hello.say_hello。一旦使用该 *setup.py* 安装了该包—例如，使用 pip install—pytimed 脚本就能检测到新添加的扩展点。

在启动时，pytimed 会扫描 pytimed 组并找到 key 为 hello 的扩展。然后，它会调用 hello.say_hello 函数，获取两个值：每次调用之间等待的秒数和要调用的函数，在此示例中为 2 秒和 print_hello。通过运行该程序，就像我们在 列表 5-12 中所做的那样，你可以看到每隔 2 秒就会在屏幕上打印出“你好，世界！”。

>>> 导入 pytimed

>>> pytimed.main()

你好，世界！

你好，世界！

你好，世界！

*列表 5-12: 运行 pytimed*

这个机制提供的可能性是巨大的：你可以轻松且通用地构建驱动系统、钩子系统和扩展。每个程序都手动实现这个机制会很繁琐，但幸运的是，有一个 Python 库可以为我们处理这些枯燥的部分。

stevedore 库提供了对基于相同机制的动态插件的支持，正如我们在前面的示例中所展示的那样。本示例中的用例已经非常简化，但我们仍然可以在这个脚本中进一步简化它，*pytimed_stevedore.py*：

从 stevedore.extension 导入 ExtensionManager

导入 time

定义 main():

seconds_passed = 0

extensions = ExtensionManager('pytimed', invoke_on_load=True)

while True:

对于 extensions 中的每个扩展：

尝试：

seconds, callable = extension.obj 除外:

# 忽略失败

pass

否则：

如果 seconds_passed % seconds == 0:

callable()

time.sleep(1)

seconds_passed += 1

stevedore 的 ExtensionManager 类提供了一种简单的方法来加载所有扩展点组的扩展。名称作为第一个参数传入。参数 invoke_on_load=True 确保在每个扩展被发现时，其组内的每个函数都会被调用一次。这使得结果可以直接通过扩展的 obj 属性访问。

如果你查看 stevedore 文档，你会发现 ExtensionManager 有多种子类，可以处理不同的情况，比如根据扩展名称或函数的结果加载特定的扩展。这些都是常用的模式，你可以将它们应用到你的程序中，以直接实现这些模式。

例如，我们可能只想从我们的入口点组加载并运行一个扩展。利用 stevedore.driver.DriverManager 类可以实现这一点，正如列表 5-13 所示。

from stevedore.driver import DriverManager

import time

def main(name):

seconds_passed = 0

seconds, callable = DriverManager('pytimed', name, invoke_on_load=True).

driver

while True:

if seconds_passed % seconds == 0:

callable()

time.sleep(1)

seconds_passed += 1

main("hello")

*列表 5-13：使用 stevedore 从入口点运行单个扩展*

在这种情况下，只有一个扩展根据名称被加载并选择。这使得我们可以快速构建一个*驱动系统*，在其中只有一个扩展被程序加载并使用。

### **总结**

Python 的打包生态系统有一段坎坷的历史；然而，目前情况正在稳定。setuptools 库提供了完整的打包解决方案，不仅能将你的代码以不同的格式传输并上传到 PyPI，还能通过入口点与其他软件和库进行连接。

### **Nick Coghlan 谈打包**

Nick 是 Red Hat 的 Python 核心开发人员。他撰写了几份 PEP 提案，包括 PEP 426（Python 软件包元数据 2.0），并且他作为我们的终身仁慈独裁者 Guido van Rossum 的代表，Guido 是 Python 的作者。

**Python 的打包解决方案（distutils、setuptools、distutils2、distlib、bento、pbr 等）种类繁多。你认为这种碎片化和分歧的原因是什么？**

简单来说，软件发布、分发和集成是一个复杂的问题，仍有很多空间可以为不同的使用场景提供多种解决方案。在我最近的讲座中，我提到这个问题主要与时间有关，因为不同的打包工具诞生于软件分发的不同时代。

**PEP 426 定义了 Python 包的新元数据格式，仍然相对较新，尚未批准。你认为它将如何解决当前的打包问题？**

PEP 426 最初作为 Wheel 格式定义的一部分开始，但 Daniel Holth 意识到 Wheel 可以与 setuptools 定义的现有元数据格式兼容。因此，PEP 426 是现有 setuptools 元数据与 distutils2 和其他打包系统（如 RPM 和 npm）的一些理念的整合。它解决了一些现有工具所遇到的挫折（例如，清晰地区分不同类型的依赖）。

主要的收益将是 PyPI 上的 REST API，提供完整的元数据访问，以及（希望）能够根据上游元数据自动生成符合分发策略的包。

**Wheel 格式相对较新，尚未广泛使用，但它似乎很有前景。为什么它不是标准库的一部分？**

事实证明，标准库并不是一个适合用于打包标准的地方：它演化得太慢，而且标准库的新增功能不能在 Python 的早期版本中使用。因此，在今年早些时候的 Python 语言峰会上，我们调整了 PEP 流程，允许 distutils-sig 管理与打包相关的 PEP 的完整审批周期，python-dev 只会参与涉及直接修改 CPython 的提案（如 pip 启动）。

**Wheel** 包的未来是什么？

在 Wheel 适用于 Linux 之前，我们还需要做一些调整。不过，pip 已经将 Wheel 作为替代 Egg 格式的方法，允许本地缓存构建文件，以便快速创建虚拟环境，而 PyPI 也允许上传适用于 Windows 和 macOS 的 Wheel 包。
