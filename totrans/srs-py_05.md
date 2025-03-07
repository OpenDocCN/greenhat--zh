## **5**

**发布你的软件**

![image](img/common01.jpg)

可以肯定地说，某个时刻，你会希望发布你的软件。尽管你可能很想直接压缩你的代码并上传到互联网，Python 提供了工具来使你的最终用户更容易使用你的软件。你应该已经熟悉使用*setup.py*来安装 Python 应用程序和库，但你可能从未深入了解过它背后的工作原理，或者如何创建你自己的*setup.py*。

在本章中，你将了解*setup.py*的历史、该文件的工作原理，以及如何创建你自己的自定义*setup.py*。我们还将深入探讨一些打包安装工具`pip`的鲜为人知的功能，以及如何通过`pip`使你的软件可下载。最后，我们将看看如何使用 Python 的入口点，使得不同程序之间的函数可以轻松找到。掌握这些技能后，你就能让你的发布软件对最终用户更加可访问。

### **setup.py 的历史**

`distutils`库最初由软件开发者 Greg Ward 创建，自 1998 年起就成为 Python 标准库的一部分。Ward 旨在为开发者创建一个简便的方式，以便自动化其终端用户的安装过程。软件包提供*setup.py*文件作为标准的 Python 安装脚本，并可以使用`distutils`进行安装，如列表 5-1 所示。

```py
#!/usr/bin/python
from distutils.core import setup

setup(name="rebuildd",
      description="Debian packages rebuild tool",
      author="Julien Danjou",
      author_email="acid@debian.org",
      url="http://julien.danjou.info/software/rebuildd.html",
      packages=['rebuildd'])
```

*列表 5-1：使用 distutils 构建 setup.py*

通过将*setup.py*文件作为项目的根文件，所有用户需要做的就是运行该文件，并将适当的命令作为参数传递。即使你的分发包除了本地 Python 模块外还包含 C 模块，`distutils`也能自动处理它们。

`distutils`的开发在 2000 年停止；从那时起，其他开发者接手了它的工作。一个显著的继任者是名为`setuptools`的打包库，它提供了更多的更新和高级功能，如自动依赖管理、`Egg`分发格式和`easy_install`命令。由于`distutils`在开发时仍然是 Python 标准库中接受的软件打包方式，`setuptools`在一定程度上保持了与其的向后兼容性。列表 5-2 展示了如何使用`setuptools`构建与列表 5-1 相同的安装包。

```py
#!/usr/bin/env python
import setuptools

setuptools.setup(
    name="rebuildd",
    version="0.2",
    author="Julien Danjou",
    author_email="acid@debian.org",
    description="Debian packages rebuild tool",
    license="GPL",
    url="http://julien.danjou.info/software/rebuildd/",
    packages=['rebuildd'],
    classifiers=[
        "Development Status :: 2 - Pre-Alpha",
        "Intended Audience :: Developers",
        "Intended Audience :: Information Technology",
        "License :: OSI Approved :: GNU General Public License (GPL)", "Operating System :: OS Independent",
        "Programming Language :: Python"
    ],
)
```

*列表 5-2：使用 setuptools 构建 setup.py*

最终，`setuptools`的开发也放缓了，但没过多久，另一组开发者将其分叉，创建了一个新的库，名为`distribute`，它相比`setuptools`提供了几个优点，包括更少的 BUG 和对 Python 3 的支持。

所有最好的故事都有一个转折结局：2013 年 3 月，`setuptools` 和 `distribute` 背后的团队决定将它们的代码库合并到原始 `setuptools` 项目的旗下。因此，`distribute` 现在已经被弃用，`setuptools` 再次成为处理高级 Python 安装的标准方式。

在这一切发生的同时，另一个名为 `distutils2` 的项目也在开发中，旨在完全替代 Python 标准库中的 `distutils`。与 `distutils` 和 `setuptools` 不同，它将包的元数据存储在一个纯文本文件 *setup.cfg* 中，这对于开发者来说更容易编写，对于外部工具来说更容易读取。然而，`distutils2` 保留了 `distutils` 的一些缺点，例如命令行设计的晦涩，并且缺乏对 Windows 上入口点和原生脚本执行的支持——这两者是 `setuptools` 提供的特性。由于这些原因以及其他原因，将 `distutils2`（即 `packaging`）包含进 Python 3.3 标准库的计划未能实现，且该项目在 2012 年被放弃。

`packaging` 仍然有机会通过 `distlib` 重新崛起，后者是一个新兴的替代 `distutils` 的项目。在发布之前，曾有传闻称 `distlib` 包会成为 Python 3.4 标准库的一部分，但最终并未实现。`distlib` 包含了 `packaging` 的最佳特性，执行了与打包相关的 PEP 中描述的基本工作。

所以，回顾一下：

+   distutils 是 Python 标准库的一部分，能够处理简单的包安装。

+   `setuptools`，作为高级包安装的标准，最初曾被弃用，但现在已经重新进入活跃开发状态，并成为事实上的标准。

+   `distribute` 已经在版本 0.7 中合并回 `setuptools`；`distutils2`（即 `packaging`）已被废弃。

+   `distlib` *可能* 会在未来取代 `distutils`。

还有其他的打包库存在，但这些是你最常遇到的五个。研究这些库时要小心：由于上述复杂的历史，很多文档已经过时。然而，官方文档是最新的。

简而言之，`setuptools` 是目前使用的分发库，但未来要关注 `distlib`。

### **使用 setup.cfg 进行打包**

你可能已经在某个时刻尝试过为某个包编写 *setup.py*，无论是从其他项目中复制，还是通过浏览文档自己构建。编写 *setup.py* 不是一项直观的任务。选择使用哪个工具只是第一道难题。在本节中，我想向你介绍 `setuptools` 的一个近期改进：*setup.cfg* 文件支持。

这就是使用 *setup.cfg* 文件的 *setup.py* 的样子：

```py
import setuptools

setuptools.setup()
```

两行代码——就是这么简单。setup.py 所需的实际元数据存储在 *setup.cfg* 中，如 Listing 5-3 所示。

```py
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
    Intended Audience :: Information Technology
    License :: OSI Approved :: Apache Software License
    Operating System :: OS Independent
    Programming Language :: Python
```

*清单 5-3：setup.cfg 元数据*

如你所见，*setup.cfg* 使用了一种易于编写和阅读的格式，直接受到了`distutils2`的启发。许多其他工具，如`Sphinx`或`Wheel`，也从这个 *setup.cfg* 文件中读取配置—仅这一点就是开始使用它的一个有力论据。

在 清单 5-3 中，项目的描述是从 *README.rst* 文件中读取的。良好的实践是始终包含一个 README 文件—最好是 RST 格式—以便用户快速了解项目的内容。仅凭这些基本的 *setup.py* 和 *setup.cfg* 文件，你的包就已经准备好发布，并供其他开发者和应用程序使用。如果需要，`setuptools` 文档提供了更多细节，例如，如果你在安装过程中有额外的步骤，或者想要包含额外的文件。

另一个有用的打包工具是 `pbr`，即 *Python Build Reasonableness* 的缩写。该项目最初在 OpenStack 中启动，作为 `setuptools` 的扩展，旨在简化包的安装和部署。`pbr` 打包工具与 `setuptools` 一起使用，实现了 `setuptools` 中缺失的功能，包括以下这些：

+   自动生成 Sphinx 文档

+   基于 `git` 历史自动生成 *AUTHORS* 和 *ChangeLog* 文件

+   自动创建 `git` 的文件列表

+   基于 `git` 标签使用语义化版本控制进行版本管理

所有这些都几乎不需要你做任何额外的工作。要使用 `pbr`，你只需启用它，如 清单 5-4 所示。

```py
import setuptools

setuptools.setup(setup_requires=['pbr'], pbr=True)
```

*清单 5-4：使用 pbr 的 setup.py*

`setup_requires` 参数指示 `setuptools` 在使用之前必须安装 `pbr`。`pbr=True` 参数确保 `setuptools` 加载并调用 `pbr` 扩展。

一旦启用，`python setup.py` 命令将增强 `pbr` 功能。例如，调用 `python setup.py --version` 将根据现有的 `git` 标签返回项目的版本号。运行 `python setup.py sdist` 将创建一个源代码 tarball，并自动生成 *ChangeLog* 和 *AUTHORS* 文件。

### **Wheel 格式分发标准**

在 Python 的大部分历史中，并没有官方的标准分发格式。尽管不同的分发工具通常使用一些通用的归档格式—甚至由 `setuptools` 引入的 `Egg` 格式仅仅是一个具有不同扩展名的 zip 文件—它们的元数据和包结构彼此不兼容。当官方安装标准终于在 PEP 376 中定义时，这个问题更为严重，因为它与现有格式也不兼容。

为了解决这些问题，PEP 427 被编写出来，定义了 Python 分发包的新标准，称为 `Wheel`。该格式的参考实现可作为工具使用，亦称为 `Wheel`。

`Wheel` 从版本 1.4 开始被 `pip` 支持。如果你使用 `setuptools` 并且已经安装了 `Wheel` 包，它会自动集成为一个名为 `bdist_wheel` 的 `setuptools` 命令。如果你没有安装 `Wheel`，可以通过命令 `pip install wheel` 来安装它。列表 5-5 显示了调用 `bdist_wheel` 时的部分输出，已缩短以便打印。

```py
   $ python setup.py bdist_wheel
   running bdist_wheel
   running build
   running build_py
   creating build/lib
   creating build/lib/daiquiri
   creating build/lib/daiquiri/tests
   copying daiquiri/tests/__init__.py -> build/lib/daiquiri/tests
   --snip--
   running egg_info
   writing requirements to daiquiri.egg-info/requires.txt
   writing daiquiri.egg-info/PKG-INFO
   writing top-level names to daiquiri.egg-info/top_level.txt
   writing dependency_links to daiquiri.egg-info/dependency_links.txt
   writing pbr to daiquiri.egg-info/pbr.json
   writing manifest file 'daiquiri.egg-info/SOURCES.txt'
   installing to build/bdist.macosx-10.12-x86_64/wheel
   running install
   running install_lib
   --snip--

   running install_scripts
   creating build/bdist.macosx-10.12-x86_64/wheel/daiquiri-1.3.0.dist-info/WHEEL
➊ creating '/Users/jd/Source/daiquiri/dist/daiquiri-1.3.0-py2.py3-none-any.whl'
   and adding '.' to it
   adding 'daiquiri/__init__.py'
   adding 'daiquiri/formatter.py'
   adding 'daiquiri/handlers.py'

   --snip--
```

*列表 5-5：调用 setup.py bdist_wheel*

`bdist_wheel` 命令会在 *dist* 目录中创建一个 *.whl* 文件 ➊。与 `Egg` 格式一样，`Wheel` 存档只是一个具有不同扩展名的压缩文件。然而，`Wheel` 存档不需要安装——你只需添加一个斜杠后跟模块名，就可以加载并运行你的代码：

```py
$ python wheel-0.21.0-py2.py3-none-any.whl/wheel -h
usage: wheel [-h]

             {keygen,sign,unsign,verify,unpack,install,install-
scripts,convert,help}
             --snip--

positional arguments:
--snip--
```

你可能会惊讶地发现，这并不是 `Wheel` 格式本身引入的功能。Python 也可以运行常规的 zip 文件，就像 Java 的 *.jar* 文件一样：

```py
python foobar.zip
```

这等价于：

```py
PYTHONPATH=foobar.zip python -m __main__
```

换句话说，你程序的 `__main__` 模块会从 `__main__.py` 自动导入。你也可以通过追加斜杠后跟模块名，从指定的模块导入 `__main__`，就像使用 `Wheel` 一样：

```py
python foobar.zip/mymod
```

这等价于：

```py
PYTHONPATH=foobar.zip python -m mymod.__main__
```

`Wheel` 的一个优势是，它的命名约定允许你指定你的发行版是针对特定架构和/或 Python 实现（如 CPython、PyPy、Jython 等）。如果你需要分发用 C 编写的模块，这一点特别有用。

默认情况下，`Wheel` 包与构建它时所用的 Python 主要版本绑定。当使用 `python2 setup.py bdist_wheel` 调用时，`Wheel` 文件名的格式会类似于 *library-version-py2-none-any.whl*。

如果你的代码兼容所有主要的 Python 版本（即 Python 2 和 Python 3），你可以构建一个通用的 `Wheel`：

```py
python setup.py bdist_wheel --universal
```

生成的文件名会有所不同，并且包含两个 Python 主要版本——类似于 *library-version-py2.py3-none-any.whl*。构建一个通用的 `Wheel` 可以避免当只需要一个 `Wheel` 覆盖两个 Python 主要版本时，产生两个不同的 `Wheel` 文件。

如果你不想在每次构建 `Wheel` 时都传递 `--universal` 标志，可以将其添加到你的 *setup.cfg* 文件中：

```py
[wheel]
universal=1
```

如果你构建的 `Wheel` 包含二进制程序或库（例如用 C 编写的 Python 扩展），那么二进制 `Wheel` 可能没有你想象的那样便捷跨平台。它默认可以在一些平台上工作，比如 Darwin（macOS）或微软 Windows，但可能无法在所有 Linux 发行版上运行。PEP 513 (*[`www.python.org/dev/peps/pep-0513`](https://www.python.org/dev/peps/pep-0513)*) 通过定义一个新的平台标签 `manylinux1` 和一组保证在该平台上可用的最小库，来解决这个 Linux 问题。

`Wheel` 是一个用于分发准备安装的库和应用程序的优秀格式，因此建议你构建并将它们上传到 PyPI。

### **与世界分享你的作品**

一旦你拥有一个合适的 *setup.py* 文件，构建一个可以分发的源代码 tarball 就变得很简单。`sdist setuptools` 命令正是完成这项工作的命令，正如列表 5-6 所示。

```py
$ python setup.py sdist
running sdist

[pbr] Generating AUTHORS
running egg_info
writing requirements to ceilometer.egg-info/requires.txt
writing ceilometer.egg-info/PKG-INFO
writing top-level names to ceilometer.egg-info/top_level.txt
writing dependency_links to ceilometer.egg-info/dependency_links.txt
writing entry points to ceilometer.egg-info/entry_points.txt
[pbr] Processing SOURCES.txt
[pbr] In git context, generating filelist from git
warning: no previously-included files matching '*.pyc' found anywhere in
distribution
writing manifest file 'ceilometer.egg-info/SOURCES.txt'
running check
copying setup.cfg -> ceilometer-2014.1.a6-g772e1a7
Writing ceilometer-2014.1.a6-g772e1a7/setup.cfg

--snip--

Creating tar archive
removing 'ceilometer-2014.1.a6.g772e1a7' (and everything under it)
```

*列表 5-6：使用 setup.py sdist 构建源 tarball*

`sdist` 命令会在源代码树的 *dist* 目录下创建一个 tarball。该 tarball 包含源代码树中的所有 Python 模块。如前一节所示，你也可以使用 `bdist_wheel` 命令构建 `Wheel` 归档文件。`Wheel` 归档的安装速度更快，因为它们已经是正确的安装格式。

使代码可用的最后一步是将你的软件包导出到用户可以通过 `pip` 安装的地方。这意味着将你的项目发布到 PyPI。

如果这是你第一次将软件发布到 PyPI，建议先在安全的沙盒中测试发布过程，而不是直接在生产服务器上进行。你可以使用 PyPI 测试服务器，它复制了主索引的所有功能，但仅用于测试目的。

第一步是将你的项目注册到测试服务器。首先，打开你的 *~/.pypirc* 文件并添加以下内容：

```py
[distutils]
index-servers =
    testpypi [testpypi]
username = <your username>
password = <your password>
repository = https://testpypi.python.org/pypi
```

保存文件，现在你可以在索引中注册你的项目：

```py
$ python setup.py register -r testpypi
running register
running egg_info
writing requirements to ceilometer.egg-info/requires.txt
writing ceilometer.egg-info/PKG-INFO
writing top-level names to ceilometer.egg-info/top_level.txt
writing dependency_links to ceilometer.egg-info/dependency_links.txt
writing entry points to ceilometer.egg-info/entry_points.txt
[pbr] Reusing existing SOURCES.txt
running check
Registering ceilometer to https://testpypi.python.org/pypi
Server response (200): OK
```

这将连接到测试 PyPI 服务器实例并创建一个新的条目。不要忘记使用 `-r` 选项；否则，会使用真实的生产 PyPI 实例！

很明显，如果有同名的项目已经在 PyPI 注册，过程将会失败。请尝试使用一个新的名称，一旦你的程序成功注册并收到 `OK` 响应，你就可以上传源代码分发的 tarball，如列表 5-7 所示。

```py
$ python setup.py sdist upload -r testpypi
running sdist
[pbr] Writing ChangeLog
[pbr] Generating AUTHORS
running egg_info
writing requirements to ceilometer.egg-info/requires.txt
writing ceilometer.egg-info/PKG-INFO
writing top-level names to ceilometer.egg-info/top_level.txt
writing dependency_links to ceilometer.egg-info/dependency_links.txt
writing entry points to ceilometer.egg-info/entry_points.txt
[pbr] Processing SOURCES.txt
[pbr] In git context, generating filelist from git
warning: no previously-included files matching '*.pyc' found anywhere in
distribution
writing manifest file 'ceilometer.egg-info/SOURCES.txt'
running check
creating ceilometer-2014.1.a6.g772e1a7

--snip--

copying setup.cfg -> ceilometer-2014.1.a6.g772e1a7
Writing ceilometer-2014.1.a6.g772e1a7/setup.cfg
Creating tar archive
removing 'ceilometer-2014.1.a6.g772e1a7' (and everything under it)
running upload Submitting dist/ceilometer-2014.1.a6.g772e1a7.tar.gz to https://testpypi
.python.org/pypi
Server response (200): OK
```

*列表 5-7：将 tarball 上传到 PyPI*

或者，你可以上传一个 `Wheel` 格式的归档文件，如列表 5-8 所示。

```py
$ python setup.py bdist_wheel upload -r testpypi
running bdist_wheel
running build
running build_py
running egg_info
writing requirements to ceilometer.egg-info/requires.txt
writing ceilometer.egg-info/PKG-INFO
writing top-level names to ceilometer.egg-info/top_level.txt
writing dependency_links to ceilometer.egg-info/dependency_links.txt
writing entry points to ceilometer.egg-info/entry_points.txt
[pbr] Reusing existing SOURCES.txt
installing to build/bdist.linux-x86_64/wheel
running install
running install_lib
creating build/bdist.linux-x86_64/wheel

--snip--

creating build/bdist.linux-x86_64/wheel/ceilometer-2014.1.a6.g772e1a7
.dist-info/WHEEL
running upload
Submitting /home/jd/Source/ceilometer/dist/ceilometer-2014.1.a6
.g772e1a7-py27-none-any.whl to https://testpypi.python.org/pypi
Server response (200): OK
```

*列表 5-8：将 Wheel 归档上传到 PyPI*

一旦这些操作完成，你和其他用户就可以在 PyPI 测试服务器上搜索已上传的软件包，甚至可以使用 `pip` 安装这些软件包，只需要使用 `-i` 选项指定测试服务器：

```py
$ pip install -i https://testpypi.python.org/pypi ceilometer
```

如果一切正常，你可以将你的项目上传到主 PyPI 服务器。在此之前，确保将你的凭证和服务器的详细信息添加到 `~/.pypirc` 文件中，如下所示：

```py
[distutils]
index-servers =
    pypi
    testpypi

[pypi]
username = <your username>
password = <your password> [testpypi]
repository = https://testpypi.python.org/pypi
username = <your username>
password = <your password>
```

现在，如果你使用 `-r pypi` 选项运行 `register` 和 `upload`，你的软件包应该会被上传到 PyPI。

**注意**

*PyPI 可以在其索引中保留多个版本的软件，这样你就可以安装特定的旧版本，如果你需要的话。只需要在 pip install 命令中指定版本号；例如，pip install foobar==1.0.2。*

这个过程非常简便，允许进行任意数量的上传。你可以根据需要频繁发布你的软件，用户也可以根据需要频繁安装和更新。

### **入口点**

你可能已经在不知情的情况下使用了`setuptools`入口点。通过`setuptools`分发的软件包括描述特性的重要元数据，例如所需的依赖关系，以及——对本主题更为相关的——*入口点*的列表。入口点是其他 Python 程序可以用来发现一个包提供的动态功能的方法。

以下示例展示了如何在`console_scripts`入口点组中提供一个名为`rebuildd`的入口点：

```py
#!/usr/bin/python
from distutils.core import setup

setup(name="rebuildd",
    description="Debian packages rebuild tool",
    author="Julien Danjou",
    author_email="acid@debian.org",
    url="http://julien.danjou.info/software/rebuildd.html",
    entry_points={
        'console_scripts': [
            'rebuildd = rebuildd:main',
        ],
    },
    packages=['rebuildd'])
```

任何 Python 包都可以注册入口点。入口点按组组织：每个组由一组键值对组成。这些键值对使用格式`path.to.module:variable_name`。在前面的示例中，键是`rebuildd`，值是`rebuildd:main`。

入口点的列表可以通过各种工具进行操作，从`setuptools`到`epi`，正如我将在这里展示的那样。在接下来的几节中，我们将讨论如何使用入口点为我们的软件添加扩展性。

#### ***可视化入口点***

可视化包中可用入口点的最简单方法是使用名为`entry point inspector`的包。你可以通过运行`pip install entry-point-inspector`来安装它。安装后，它提供了命令`epi`，你可以从终端运行它，交互式地发现已安装包提供的入口点。清单 5-9 展示了在我的系统上运行`epi group list`的示例。

```py
$ epi group list
---------------------------
| Name                    |
--------------------------
| console_scripts |
| distutils.commands |
| distutils.setup_keywords |
| egg_info.writers |
| epi.commands |
| flake8.extension |
| setuptools.file_finders |
| setuptools.installation |
--------------------------
```

*清单 5-9：获取入口点组列表*

来自`epi group list`的输出在清单 5-9 中展示了系统上提供入口点的不同包。表格中的每一项都是一个入口点组的名称。请注意，这个列表包括了`console_scripts`，我们稍后会讨论它。我们可以使用`epi`命令与`show`命令来显示特定入口点组的详细信息，如清单 5-10 所示。

```py
$ epi group show console_scripts
-------------------------------------------------
| Name     | Module   | Member | Distribution | Error |
-------------------------------------------------
| coverage | coverage | main   | coverage 3.4 |       |
```

*清单 5-10：显示入口点组的详细信息*

我们可以看到，在`console_scripts`组中，一个名为`coverage`的入口点指向模块`coverage`的成员`main`。这个入口点，特别是由`coverage 3.4`包提供，指示在执行命令行脚本`coverage`时应调用哪个 Python 函数。在这里，应该调用的函数是`coverage.main`。

`epi`工具只是位于完整 Python 库`pkg_resources`之上的一层薄薄的封装。这个模块允许我们发现任何 Python 库或程序的入口点。入口点对于多种用途很有价值，包括控制台脚本和动态代码发现，正如你将在接下来的几节中看到的那样。

#### ***使用控制台脚本***

在编写 Python 应用程序时，你几乎总是需要提供一个可启动的程序——一个用户可以运行的 Python 脚本——并且该程序需要安装在系统路径中的某个目录内。

大多数项目都有一个类似这样的可启动程序：

```py
#!/usr/bin/python
import sys
import mysoftware

mysoftware.SomeClass(sys.argv).run()
```

这种脚本是最佳情况：许多项目在系统路径中安装了更长的脚本。然而，这些脚本存在一些主要问题：

+   用户无法知道 Python 解释器的位置或它使用的是哪个版本。

+   该脚本泄漏了无法被软件或单元测试导入的二进制代码。

+   没有简单的方法来定义安装该脚本的位置。

+   如何以便携方式安装这个程序（例如，在 Unix 和 Windows 上）并不显而易见。

为了帮助我们绕过这些问题，`setuptools`提供了`console_scripts`功能。此入口点可以用于使`setuptools`在系统路径中安装一个小程序，该程序调用你模块中的特定函数。使用`setuptools`，你可以通过在`console_scripts`入口点组中设置键/值对来指定一个函数调用来启动你的程序：键是将要安装的脚本名称，值是指向你的函数的 Python 路径（类似`my_module.main`）。

假设有一个`foobar`程序，由客户端和服务器组成。每个部分分别写在自己的模块中——`foobar.client`和`foobar.server`，它们分别位于*foobar/client.py*中：

```py
def main():
    print("Client started")
```

然后在*foobar/server.py*中：

```py
def main():
    print("Server started")
```

当然，这个程序并没有做太多事情——我们的客户端和服务器甚至没有互相通信。不过，对于我们的示例来说，它们只需要打印一条消息，告诉我们它们已成功启动。

现在我们可以在根目录中编写以下*setup.py*文件，其中在*setup.py*中定义了入口点。

```py
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
```

我们使用`module.submodule:function`的格式定义入口点。你可以看到，我们为`client`和`server`分别定义了一个入口点 ➊。

当运行`python setup.py install`时，`setuptools`会创建一个脚本，该脚本看起来像列表 5-11 中的内容。

```py
#!/usr/bin/python
# EASY-INSTALL-ENTRY-SCRIPT: 'foobar==1','console_scripts','foobar'
__requires__ = 'foobar==1'
import sys
from pkg_resources import load_entry_point

if __name__ == '__main__':
    sys.exit(
        load_entry_point('foobar==1', 'console_scripts', 'foobar')()
    )
```

*列表 5-11：由 setuptools 生成的控制台脚本*

这段代码扫描`foobar`包的入口点并从`console_scripts`组中获取`foobar`键，该键用于定位并运行相应的函数。`load_entry_point`的返回值将是对函数`foobar.client.main`的引用，该函数将在没有任何参数的情况下被调用，并且它的返回值将作为退出代码使用。

请注意，这段代码使用`pkg_resources`来发现并加载你 Python 程序中的入口点文件。

**注意**

*如果你在`setuptools`上使用了 pbr，生成的脚本会比`setuptools`默认构建的脚本更简洁（因此更快速），因为它将直接调用你在入口点中编写的函数，而无需在运行时动态搜索入口点列表。*

使用控制台脚本是一种技术，它消除了编写便携脚本的负担，同时确保你的代码保持在 Python 包中，并可以被其他程序导入（和测试）。

#### ***使用插件和驱动程序***

入口点使得发现和动态加载其他包部署的代码变得容易，但这并不是它们唯一的用途。任何应用程序都可以提出并注册入口点和组，然后按需使用它们。

在本节中，我们将创建一个 `cron` 风格的守护进程 `pycrond`，它将允许任何 Python 程序注册一个命令，使其每隔几秒钟运行一次，通过在 `pytimed` 组中注册一个入口点。该入口点所指示的属性应为返回 `number_of_seconds, callable` 的对象。

这是我们使用 `pkg_resources` 来发现入口点的 `pycrond` 实现，在我命名为 *pytimed.py* 的程序中：

```py
import pkg_resources
import time

def main():
    seconds_passed = 0
    while True:
        for entry_point in pkg_resources.iter_entry_points('pytimed'):
            try:
                seconds, callable = entry_point.load()()
            except:
                # Ignore failure
                pass
            else:
                if seconds_passed % seconds == 0:
                    callable()
        time.sleep(1)
        seconds_passed += 1
```

这个程序包含一个无限循环，它遍历 `pytimed` 组的每个入口点。每个入口点都使用 `load()` 方法加载。然后程序调用返回的方法，该方法需要返回在调用可调用对象之前等待的秒数以及上述可调用对象。

*pytimed.py* 中的程序是一个非常简单和天真的实现，但它足以用于我们的示例。现在我们可以编写另一个 Python 程序，命名为 *hello.py*，它需要定期调用其中的一个函数：

```py
def print_hello():
    print("Hello, world!")

def say_hello():
    return 2, print_hello
```

一旦我们定义了这个函数，就可以使用适当的入口点在 *setup.py* 中注册它。

```py
from setuptools import setup

setup(
    name="hello",
    version="1",
    packages=["hello"],
    entry_points={
        "pytimed": [
            "hello = hello:say_hello",
        ],
     },)
```

*setup.py* 脚本在 `pytimed` 组中注册一个入口点，键为 `hello`，值指向 `hello.say_hello` 函数。一旦通过该 *setup.py* 安装了该包——例如使用 `pip install`——`pytimed` 脚本就能检测到新添加的入口点。

启动时，`pytimed` 将扫描 `pytimed` 组并找到键 `hello`。然后，它将调用 `hello.say_hello` 函数，得到两个值：每次调用之间等待的秒数和要调用的函数，这里是 2 秒和 `print_hello`。通过运行程序，就像我们在 Listing 5-12 中所做的那样，你可以看到“Hello, world!” 每 2 秒打印一次。

```py
>>> import pytimed
>>> pytimed.main()
Hello, world!
Hello, world!
Hello, world!
```

*Listing 5-12: 运行 pytimed*

这种机制提供的可能性是巨大的：你可以轻松而通用地构建驱动程序系统、挂钩系统和扩展。如果每个程序都手动实现这个机制，肯定会很繁琐，但幸运的是，Python 有一个库可以帮我们处理这些枯燥的部分。

`stevedore` 库提供了基于我们之前示例中演示的相同机制的动态插件支持。本示例中的使用案例已经很简单，但我们仍然可以在这个脚本中进一步简化它，*pytimed_stevedore.py*：

```py
from stevedore.extension import ExtensionManager
import time

def main():
    seconds_passed = 0
    extensions = ExtensionManager('pytimed', invoke_on_load=True)
    while True:
        for extension in extensions:
            try:
                seconds, callable = extension.obj except:
                # Ignore failure
                pass
            else:
                if seconds_passed % seconds == 0:
                    callable()
        time.sleep(1)
        seconds_passed += 1
```

`stevedore` 的 `ExtensionManager` 类提供了一种简单的方法来加载入口点组的所有扩展。名称作为第一个参数传递。参数 `invoke_on_load=True` 确保每个组的函数在被发现后都会被调用。这样，结果可以直接通过扩展的 `obj` 属性访问。

如果你查看`stevedore`的文档，你会看到`ExtensionManager`有多种子类，可以处理不同的情况，例如根据扩展名或函数结果加载特定扩展。所有这些常用模型可以应用到你的程序中，直接实现这些模式。

例如，我们可能只想从入口点组中加载并运行一个扩展。利用`stevedore.driver.DriverManager`类，我们可以实现这一点，如列出 5-13 所示。

```py
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
```

*列出 5-13：使用 stevedore 从入口点运行单个扩展*

在这种情况下，只有一个扩展按名称加载和选择。这使我们能够快速构建一个*驱动程序系统*，其中只有一个扩展被程序加载和使用。

### **总结**

Python 的打包生态系统经历了曲折的历史；然而，现在局势正在逐步稳定。`setuptools`库提供了完整的打包解决方案，不仅可以以不同格式传输代码并上传到 PyPI，还可以通过入口点处理与其他软件和库的连接。

### **Nick Coghlan 谈打包**

Nick 是 Red Hat 的 Python 核心开发者。他撰写了几份 PEP 提案，包括 PEP 426（Python 软件包元数据 2.0），并且担任我们的终身独裁者 Guido van Rossum（Python 的作者）的代表。

**Python 的打包解决方案（distutils、setuptools、distutils2、distlib、bento、pbr 等）种类繁多。你认为这种碎片化和分歧的原因是什么？**

简而言之，软件发布、分发和集成是一个复杂的问题，有足够的空间为不同的使用场景量身定制多种解决方案。在我最近的相关讲座中，我提到问题主要是由于时代差异，不同的打包工具诞生于不同的软件分发时代。

**PEP 426，定义了 Python 包的新元数据格式，仍然相对较新，尚未获得批准。你认为它将如何解决当前的打包问题？**

PEP 426 最初是作为`Wheel`格式定义的一部分开始的，但 Daniel Holth 意识到`Wheel`可以与`setuptools`定义的现有元数据格式兼容。因此，PEP 426 是对现有`setuptools`元数据的整合，并融合了`distutils2`及其他打包系统（如`RPM`和`npm`）的一些理念。它解决了现有工具所遇到的一些问题（例如，清晰地区分不同类型的依赖关系）。

主要的进展将是在 PyPI 上提供 REST API，提供完整的元数据访问，以及（希望）从上游元数据自动生成符合分发政策的打包文件。

**`Wheel`格式相对较新，尚未广泛使用，但似乎很有前景。为什么它不是标准库的一部分？**

事实证明，标准库并不是打包标准的合适场所：它的发展速度太慢，而且标准库的后续版本的新增内容无法与早期版本的 Python 一起使用。因此，在今年早些时候的 Python 语言峰会上，我们对 PEP 流程进行了调整，允许 `distutils-sig` 管理与打包相关的 PEP 的完整审批周期，而 `python-dev` 仅在涉及直接修改 CPython 的提案（例如 `pip` 引导程序）时参与。

**`Wheel` 包的未来是什么？**

在 `Wheel` 适用于 Linux 之前，我们仍需进行一些调整。不过，`pip` 已将 `Wheel` 作为 `Egg` 格式的替代方案，允许对构建进行本地缓存，以便快速创建虚拟环境，并且 PyPI 允许上传适用于 Windows 和 macOS 的 `Wheel` 档案。
