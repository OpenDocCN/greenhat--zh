## **6**

**单元测试**

![image](../images/common01.jpg)

很多人觉得单元测试既繁琐又耗时，也有一些人和项目没有测试政策。本章假设你已经认识到单元测试的智慧！编写没有经过测试的代码本质上是没有意义的，因为无法确凿证明它能正常工作。如果你需要说服自己，我建议你从阅读测试驱动开发的好处开始。

在本章中，你将学习如何使用Python工具构建一个全面的测试套件，使测试更加简便和自动化。我们将讨论如何利用工具使你的软件更加稳定、无回归。我们将涉及创建可重用的测试对象、并行运行测试、揭示未测试的代码，以及使用虚拟环境确保你的测试是干净的，还有一些其他的最佳实践和想法。

### **测试基础**

在Python中编写和运行单元测试并不复杂。这个过程既不具侵入性，也不会造成干扰，单元测试会极大地帮助你和其他开发者维护软件。在这里，我将讨论一些测试的基础知识，帮助你更轻松地进行测试。

#### ***一些简单的测试***

首先，你应该将测试存储在适用于其所在应用或库的tests子模块中。这样做将使你能够将测试作为模块的一部分进行发布，以便任何人都可以运行或重用这些测试——即使软件已经安装——而不必使用源代码包。将测试作为主模块的子模块也能防止它们在顶层的tests模块中被误安装。

使用一个模拟你模块树层次结构的测试树层次结构会让测试更加易于管理。这意味着，覆盖*mylib/foobar.py*代码的测试应存储在*mylib/tests/test_foobar.py*中。统一的命名法会让你在查找与特定文件相关的测试时更加简便。[列表 6-1](ch06.xhtml#ch6list1)展示了你可以编写的最简单的单元测试。

def test_true():

assert True

*列表 6-1: 在test_true.py中的一个非常简单的测试*

这将简单地断言程序的行为符合你的预期。要运行这个测试，你需要加载*test_true.py*文件并运行其中定义的test_true()函数。

然而，为每个测试文件和函数编写和运行单独的测试会是一件麻烦事。对于简单用法的小型项目，pytest包能够提供帮助——通过pip安装后，pytest提供pytest命令，加载所有文件名以*test_*开头的文件，并执行所有以test_开头的函数。

仅凭我们源代码树中的*test_true.py*文件，运行pytest会给出以下输出：

$ pytest -v test_true.py

========================== 测试会话开始 ===========================

platform darwin -- Python 3.6.4, pytest-3.3.2, py-1.5.2, pluggy-0.6.0 --

/usr/local/opt/python/bin/python3.6

缓存目录：.cache

根目录：examples, 配置文件：

收集到 1 项

test_true.py::test_true 测试通过                                     [100%]

======================== 1 个通过，耗时 0.01 秒 ========================

-v 选项告诉 pytest 显示详细信息，并在每个测试运行时单独打印测试名称。如果测试失败，输出会更改以指示失败，并附带完整的回溯信息。

这次我们添加一个失败的测试，如 [清单 6-2](ch06.xhtml#ch6list2) 所示。

def test_false():

assert False

*清单 6-2：test_true.py 中的失败测试*

如果我们再次运行测试文件，结果会是这样：

$ pytest -v test_true.py

========================== 测试会话开始 ===========================

平台 darwin -- Python 3.6.4, pytest-3.3.2, py-1.5.2, pluggy-0.6.0 -- /usr/

local/opt/python/bin/python3.6

缓存目录：.cache

根目录：examples, 配置文件：

收集到 2 项

test_true.py::test_true 测试通过                                     [ 50%]

test_true.py::test_false 测试失败                                    [100%]

================================ 测试失败 ================================

_______________________________ test_false _______________________________

def test_false():

>       assert False

E       assert False

test_true.py:5: 断言错误

=================== 1 个失败，1 个通过，耗时 0.07 秒 ===================

当断言错误异常被抛出时，测试会立即失败；我们的 assert 测试会在其参数评估为假（False、None、0 等）时抛出断言错误。如果抛出任何其他异常，测试也会失败。

简单吧？尽管这种方法很简化，但很多小项目都使用这种方式，并且效果很好。这些项目除了 pytest 外不需要其他工具或库，因此可以依赖简单的 assert 测试。

当你开始编写更复杂的测试时，pytest 将帮助你理解失败测试的原因。假设以下是一个测试：

def test_key():

a = ['a', 'b']

b = ['b']

assert a == b

当运行 pytest 时，它会给出如下输出：

$ pytest test_true.py

========================== 测试会话开始 =========================== 平台 darwin -- Python 3.6.4, pytest-3.3.2, py-1.5.2, pluggy-0.6.0

根目录：/Users/jd/Source/python-book/examples, 配置文件：

插件：celery-4.1.0

收集到 1 项

test_true.py 测试失败                                                 [100%]

================================ 测试失败 ================================

________________________________ test_key ________________________________

def test_key():

a = ['a', 'b']

b = ['b']

>       assert a == b

E       断言错误：assert ['a', 'b'] == ['b']

E         在索引 0 处有差异：'a' != 'b'

E         左侧包含更多项，第一个额外项：'b'

E         使用 -v 获取完整的差异

test_true.py:10: 断言错误

======================== 1 个失败，耗时 0.07 秒 ========================

这告诉我们 a 和 b 是不同的，并且此测试未通过。它还准确地告诉我们它们之间的不同之处，使得修复测试或代码变得容易。

#### ***跳过测试***

如果一个测试无法运行，你可能想要跳过该测试。例如，你可能希望根据某个特定库是否存在来有条件地运行一个测试。为此，你可以使用 pytest.skip() 函数，它会将测试标记为跳过，并继续执行下一个测试。pytest.mark.skip 装饰器会无条件跳过被装饰的测试函数，所以当一个测试总是需要被跳过时，你会使用它。[Listing 6-3](ch06.xhtml#ch6list3) 显示了如何使用这些方法跳过一个测试。

import pytest

try:

import mylib

except ImportError:

mylib = None

@pytest.mark.skip("不要运行这个")

def test_fail():

assert False

@pytest.mark.skipif(mylib is None, reason="mylib 不可用")

def test_mylib():

assert mylib.foobar() == 42 def test_skip_at_runtime():

if True:

pytest.skip("最终我不想运行它")

*Listing 6-3: 跳过测试*

执行时，这个测试文件将输出以下内容：

$ pytest -v examples/test_skip.py

========================== 测试会话开始 ===========================

平台 darwin -- Python 3.6.4, pytest-3.3.2, py-1.5.2, pluggy-0.6.0 -- /usr/

local/opt/python/bin/python3.6

cachedir: .cache

rootdir: examples, inifile:

已收集 3 项

examples/test_skip.py::test_fail SKIPPED

[ 33%]

examples/test_skip.py::test_mylib SKIPPED

[ 66%]

examples/test_skip.py::test_skip_at_runtime SKIPPED

[100%]

================= 3 个测试在 0.01 秒内被跳过 =================

在 [Listing 6-3](ch06.xhtml#ch6list3) 中的测试运行输出显示，在这种情况下，所有测试都被跳过了。这些信息可以帮助你确保没有意外跳过你本希望运行的测试。

#### ***运行特定的测试***

在使用 pytest 时，你经常只想运行某些特定的测试子集。你可以通过将它们的目录或文件作为参数传递给 pytest 命令行来选择要运行的测试。例如，调用 pytest test_one.py 将只运行 *test_one.py* 测试。Pytest 也接受目录作为参数，在这种情况下，它会递归地扫描目录并运行任何匹配 *test_*.py* 模式的文件。

你还可以在命令行中使用 -k 参数添加过滤器，以便只执行匹配某个名称的测试，如 [Listing 6-4](ch06.xhtml#ch6list4) 所示。

$ pytest -v examples/test_skip.py -k test_fail

========================== 测试会话开始 ===========================

平台 darwin -- Python 3.6.4, pytest-3.3.2, py-1.5.2, pluggy-0.6.0 -- /usr/

local/opt/python/bin/python3.6

cachedir: .cache

rootdir: examples, inifile:

已收集 3 项

examples/test_skip.py::test_fail SKIPPED

[100%] === 2 个测试被取消选择 ===

=== 1 个测试被跳过，2 个被取消选择，在 0.04 秒内完成 ===

*Listing 6-4: 按名称过滤运行的测试*

名称并不总是过滤哪些测试会运行的最佳方式。通常，开发人员会根据功能或类型将测试分组。Pytest 提供了一种动态标记系统，允许您使用关键字标记测试，作为筛选器。要以这种方式标记测试，请使用 -m 选项。如果我们设置了几个这样的测试：

import pytest

@pytest.mark.dicttest

def test_something():

a = ['a', 'b']

assert a == a

def test_something_else():

assert False

我们可以使用 -m 参数与 pytest 一起运行其中一个测试：

$ pytest -v test_mark.py -m dicttest

=== 测试会话开始 ===

platform darwin -- Python 3.6.4, pytest-3.3.2, py-1.5.2, pluggy-0.6.0 -- /usr/

local/opt/python/bin/python3.6

cachedir: .cache

rootdir: examples, inifile:

收集了 2 个项目

test_mark.py::test_something 通过

[100%]

=== 1 个测试被取消选择 ===

=== 1 通过，1 被取消选择，耗时 0.01 秒 ===

-m 标记接受更复杂的查询，因此我们也可以运行所有*未*标记的测试：

$ pytest test_mark.py -m 'not dicttest'

=== 测试会话开始 ===

platform darwin -- Python 3.6.4, pytest-3.3.2, py-1.5.2, pluggy-0.6.0

rootdir: examples, inifile:

收集了 2 个项目

test_mark.py F

[100%]

=== 失败的测试 ===

test_something_else def test_something_else():

>       assert False

E       assert False

test_mark.py:10: AssertionError

=== 1 个测试被取消选择 ===

=== 1 个失败，1 被取消选择，耗时 0.07 秒 ===

这里 pytest 执行了所有未标记为 dicttest 的测试——在此案例中是 test_something_else 测试，它失败了。剩余的标记测试 test_something 没有执行，因此列为取消选择。

Pytest 接受由 or、and 和 not 关键字组成的复杂表达式，使您能够进行更高级的筛选。

#### ***并行运行测试***

测试套件可能需要很长时间才能完成。对于大型软件项目来说，完整的单元测试套件可能需要花费数十分钟来运行。默认情况下，pytest 会按顺序运行所有测试，但顺序是未定义的。由于大多数计算机都配备了多个 CPU，通常可以通过将测试列表分割并在多个 CPU 上运行来加快速度。

为了支持这种方法，pytest 提供了插件 pytest-xdist，您可以通过 pip 安装。该插件通过 --numprocesses 参数（缩写为 -n）扩展了 pytest 命令行，该参数接受的值是要使用的 CPU 数量。运行 pytest -n 4 将使用四个并行进程运行您的测试套件，均衡地分配负载到可用的 CPU 上。

因为 CPU 的数量会因计算机而异，所以插件也接受 auto 关键字作为值。在这种情况下，它将探测机器以获取可用的 CPU 数量，并启动这个数量的进程。

#### ***使用 Fixtures 创建测试中使用的对象***

在单元测试中，你通常需要在运行测试前后执行一组常见的指令，这些指令将使用某些组件。例如，你可能需要一个代表应用程序配置状态的对象，并且希望在每个测试前初始化该对象，测试完成后将其重置为默认值。同样，如果测试依赖于临时创建文件，文件必须在测试开始前创建，并在测试完成后删除。这些组件被称为*fixtures*，它们在测试前设置，并在测试完成后清理。

在pytest中，fixtures被定义为简单的函数。fixture函数应该返回所需的对象，以便使用该fixture的测试可以使用该对象。

这是一个简单的fixture：

import pytest

@pytest.fixture

def database():

return <某个数据库连接>

def test_insert(database):

database.insert(123)

数据库fixture会自动被任何包含数据库作为参数的测试使用。test_insert()函数将接收database()函数的返回值作为其第一个参数，并根据需要使用该结果。当我们以这种方式使用fixture时，无需重复数据库初始化代码。

代码测试中的另一个常见特性是在测试使用fixture后进行清理。例如，你可能需要关闭数据库连接。将fixture实现为生成器可以让我们添加清理功能，如[示例 6-5](ch06.xhtml#ch6list5)所示。

import pytest

@pytest.fixture

def database():

db = <某个数据库连接>

yield db

db.close()

def test_insert(database):

database.insert(123)

*清理功能示例 6-5*

因为我们使用了yield关键字并将database实现为生成器，所以yield语句后的代码将在测试完成时运行。那段代码会在测试结束时关闭数据库连接。

然而，对于每个测试都关闭数据库连接可能会带来不必要的运行时成本，因为测试可能能够重用相同的连接。在这种情况下，你可以将scope参数传递给fixture装饰器，指定fixture的作用范围：

import pytest

@pytest.fixture(scope="module")

def database():

db = <某个数据库连接>

yield db

db.close()

def test_insert(database):

database.insert(123)

通过指定scope="module"参数，你为整个模块初始化一次fixture，并且相同的数据库连接将被传递给所有请求数据库连接的测试函数。

最后，你可以通过使用autouse关键字将fixtures标记为*自动使用*，从而在每个测试函数中不需要将它们作为参数。将autouse=True关键字参数传递给pytest.fixture()函数，将确保在运行模块或类中定义的任何测试之前先调用该fixture，如下所示的示例：

import os

import pytest

@pytest.fixture(autouse=True)

def change_user_env():

curuser = os.environ.get("USER")

os.environ["USER"] = "foobar"

yield

os.environ["USER"] = curuser

def test_user():

assert os.getenv("USER") == "foobar"

这种自动启用的功能非常方便，但要确保不要滥用 fixture：它们会在每个包含在其范围内的测试之前运行，因此可能会显著减慢测试的运行速度。

#### ***运行测试场景***

在单元测试时，你可能希望使用触发同样错误的多个不同对象来运行相同的错误处理测试，或者你可能希望使用不同的驱动程序运行整个测试套件。

在开发*Gnocchi*（一个时间序列数据库）时，我们大量依赖了这种后者的方法。Gnocchi 提供了一个我们称之为*存储 API*的抽象类。任何 Python 类都可以实现这个抽象基类并注册自己成为一个驱动程序。软件会在需要时加载配置的存储驱动程序，并使用实现的存储 API 来存储或检索数据。在这种情况下，我们需要一类单元测试，针对每个驱动程序运行——即针对这个存储 API 的每个实现运行——以确保所有驱动程序都符合调用者的预期。

实现这一目标的简单方法是使用*参数化 fixture*，它会多次运行所有使用该 fixture 的测试，每次使用一个不同的定义参数。[列表 6-6](ch06.xhtml#ch6list6)展示了使用参数化 fixture 来用不同的参数运行同一个测试两次的示例：一次使用 mysql，另一次使用 postgresql。

import pytest

import myapp @pytest.fixture(params=["mysql", "postgresql"])

def database(request):

d = myapp.driver(request.param)

d.start()

yield d

d.stop()

def test_insert(database):

database.insert("somedata")

*列表 6-6：使用参数化 fixture 运行测试*

在[列表 6-6](ch06.xhtml#ch6list6)中，驱动程序 fixture 被用两个不同的值进行了参数化，每个值都是应用程序支持的数据库驱动程序的名称。当 test_insert 运行时，实际上会运行两次：一次使用 MySQL 数据库连接，另一次使用 PostgreSQL 数据库连接。这使我们能够轻松地在不同场景下重用相同的测试，而不需要增加大量的代码行。

#### ***使用 Mock 对象的受控测试***

Mock 对象是模拟的对象，它们模仿真实应用对象的行为，但在特定和受控的方式下。这些对象在创建精确描述你想要测试代码的条件的环境时特别有用。你可以将所有对象替换为 mock 对象，只保留一个，来隔离你关注的对象的行为，并为测试代码创建一个环境。

一个应用场景是在编写 HTTP 客户端时，因为几乎不可能（或者至少极其复杂）启动 HTTP 服务器并通过所有场景进行测试，以返回每个可能的值。HTTP 客户端尤其难以测试所有失败场景。

Python中用于创建mock对象的标准库是mock。从Python 3.3开始，mock被合并进Python标准库，成为unittest.mock。因此，你可以像以下代码片段一样使用它，以保持Python 3.3及更早版本的向后兼容性：

尝试：

从unittest导入mock

除非ImportError：

导入mock

mock库使用起来非常简单。对mock.Mock对象访问的任何属性都会在运行时动态创建。任何值都可以赋给这样的属性。[列表6-7](ch06.xhtml#ch6list7)展示了如何使用mock创建一个带有虚假属性的虚假对象。

>>> 从unittest导入mock

>>> m = mock.Mock()

>>> m.some_attribute = "hello world" >>> m.some_attribute

"hello world"

*列表6-7：访问mock.Mock属性*

你还可以像[列表6-8](ch06.xhtml#ch6list8)那样，在一个可变对象上动态创建一个方法，在这个例子中我们创建了一个始终返回42并接受任意参数的虚假方法。

>>> 从unittest导入mock

>>> m = mock.Mock()

>>> m.some_method.return_value = 42

>>> m.some_method()

42

>>> m.some_method("with", "arguments")

42

*列表6-8：在mock.Mock对象上创建方法*

仅仅几行代码，你的mock.Mock对象现在就有了一个返回42的some_method()方法。它接受任何类型的参数，且目前没有对这些参数的值进行任何检查。

动态创建的方法也可以有（故意的）副作用。它们不仅仅是返回一个值的模板方法，还可以被定义为执行有用的代码。

[列表6-9](ch06.xhtml#ch6list9)创建了一个虚假方法，其副作用是打印"hello world"字符串。

>>> 从unittest导入mock

>>> m = mock.Mock()

>>> def print_hello():

...      print("hello world!")

...      return 43

...

➊ >>> m.some_method.side_effect = print_hello

>>> m.some_method()

hello world!

43

➋ >>> m.some_method.call_count

1

*列表6-9：创建带有副作用的mock.Mock对象方法*

我们将整个函数赋值给some_method属性 ➊。这种技术让我们能够在测试中实现更复杂的场景，因为我们可以将任何需要的代码插入到mock对象中。然后，我们只需要将这个mock对象传递给任何需要它的函数。

call_count属性 ➋是检查方法被调用次数的简单方式。

mock库使用动作/断言模式：这意味着一旦测试运行完成，你需要自行检查你正在模拟的动作是否已正确执行。[列表6-10](ch06.xhtml#ch6list10)对我们的mock对象应用assert()方法来执行这些检查。

>>> 从unittest导入mock

>>> m = mock.Mock()

➊ >>> m.some_method('foo', 'bar')

<Mock name='mock.some_method()' id='26144272'>

➋ >>> m.some_method.assert_called_once_with('foo', 'bar')

>>> m.some_method.assert_called_once_with('foo', ➌mock.ANY)

>>> m.some_method.assert_called_once_with('foo', 'baz')

回溯（最近的调用最后）：

文件 "<stdin>"，第 1 行，位于 <module>

文件 "/usr/lib/python2.7/dist-packages/mock.py"，第 846 行，位于 assert_called_

once_with

返回 self.assert_called_with(*args, **kwargs)

文件 "/usr/lib/python2.7/dist-packages/mock.py"，第 835 行，位于 assert_called_

使用

引发 AssertionError(msg)

AssertionError: 预期的调用：some_method('foo', 'baz')

实际调用：some_method('foo', 'bar')

*清单 6-10：检查方法调用*

我们创建一个方法，参数为 foo 和 bar，通过调用该方法 ➊ 来作为我们的测试。检查对 mock 对象的调用通常使用 assert_called() 方法，比如 assert_called_once_with() ➋。对于这些方法，你需要传入期望调用者在调用你的 mock 方法时使用的值。如果传入的值不是预期的，mock 会抛出 AssertionError。如果你不知道可能传入什么参数，可以使用 mock.ANY 作为值 ➌；它会匹配传递给你的 mock 方法的任何参数。

mock 库也可以用于替换外部模块中的某个函数、方法或对象。在 [清单 6-11](ch06.xhtml#ch6list11) 中，我们将 os.unlink() 函数替换为我们提供的一个伪函数。

>>> from unittest import mock

>>> import os

>>> def fake_os_unlink(path):

...     引发 IOError("测试！")

...

>>> 使用 mock.patch('os.unlink', fake_os_unlink):

...     os.unlink('foobar')

...

Traceback (most recent call last):

文件 "<stdin>"，第 2 行，位于 <module>

文件 "<stdin>"，第 2 行，位于 fake_os_unlink

IOError: 测试！

*清单 6-11：使用 mock.patch*

当 mock.patch() 作为上下文管理器使用时，它将目标函数替换为我们提供的函数，因此在上下文中执行的代码会使用该修补后的方法。通过 mock.patch() 方法，您可以更改外部代码的任何部分，使其按您希望的方式行为，从而让您在应用程序中测试所有条件，如 [清单 6-12](ch06.xhtml#ch6list12) 所示。

from unittest import mock

import pytest

import requests

class WhereIsPythonError(Exception):

pass

➊ def is_python_still_a_programming_language():

尝试：

r = requests.get("http://python.org")

except IOError:

pass

否则：

如果 r.status_code == 200:

返回 'Python 是一种编程语言' 在 r.content 中

引发 WhereIsPythonError("发生了某些错误")

def get_fake_get(status_code, content):

m = mock.Mock()

m.status_code = status_code

m.content = content

def fake_get(url):

返回 m

返回 fake_get

def raise_get(url):

引发 IOError("无法获取 url %s" % url)

➋ @mock.patch('requests.get', get_fake_get(

200, 'Python 确实是一种编程语言'))

def test_python_is():

assert is_python_still_a_programming_language() 是 True

@mock.patch('requests.get', get_fake_get(

200, 'Python 不再是编程语言'))

def test_python_is_not():

assert is_python_still_a_programming_language() 是 False

@mock.patch('requests.get', get_fake_get(404, 'Whatever'))

def test_bad_status_code():

使用 pytest.raises(WhereIsPythonError):

is_python_still_a_programming_language()

@mock.patch('requests.get', raise_get)

def test_ioerror():

with pytest.raises(WhereIsPythonError):

is_python_still_a_programming_language()

*列表 6-12：使用 mock.patch() 测试一组行为*

[列表 6-12](ch06.xhtml#ch6list12) 实现了一个测试套件，该套件在 *[http://python.org/](http://python.org/)* 网页上搜索所有包含“Python is a programming language”字符串的实例 ➊。如果不修改网页本身（显然我们无法做到这一点），就无法测试负面场景（即该句子不在网页上）。在这种情况下，我们使用 mock 来模拟并更改请求的行为，使其返回一个虚假的页面，该页面不包含该字符串。这使我们能够测试负面场景，即 *[http://python.org/](http://python.org/)* 页面不包含该句子，从而确保程序能正确处理这种情况。

本例使用了 mock.patch() 的装饰器版本 ➋。使用装饰器不会改变模拟行为，但当你需要在整个测试函数的上下文中使用模拟时，它会更简洁。

使用模拟，我们可以模拟任何问题，例如 web 服务器返回 404 错误、I/O 错误或网络延迟问题。我们可以确保代码在每种情况下都返回正确的值或抛出正确的异常，确保我们的代码始终按预期行为运行。

#### ***使用 coverage 显示未测试的代码***

作为单元测试的一个重要补充，coverage 工具可以识别测试过程中是否有代码被遗漏。它使用代码分析工具和追踪钩子来确定哪些代码行已被执行；在单元测试运行时使用它，可以显示哪些代码部分已经被覆盖，哪些没有。编写测试是有用的，但拥有一种方法来了解在测试过程中可能遗漏了哪些代码部分，才是锦上添花的关键。

通过 pip 在系统上安装 coverage Python 模块，以便从你的 shell 中访问 coverage 程序命令。

**注意**

*如果你通过操作系统安装软件安装了 coverage，该命令也可能被命名为 python-coverage。例如，在 Debian 上就是这种情况。*

以独立模式使用 coverage 很简单。它可以显示程序中从未执行的部分，以及哪些代码可能是“死代码”，也就是说，可以删除而不修改程序的正常工作流程。到目前为止，在本章讨论的所有测试工具都与 coverage 集成。

使用 pytest 时，只需通过 pip install pytest-pycov 安装 pytest-cov 插件，并添加一些选项开关来生成详细的代码覆盖率输出，如 [列表 6-13](ch06.xhtml#ch6list13) 所示。

$ pytest --cov=gnocchiclient gnocchiclient/tests/unit

---------- coverage: platform darwin, python 3.6.4-final-0 -----------

名称                                          语句数   遗漏  分支  部分分支  覆盖率

---------------------------

gnocchiclient/__init__.py                         0      0      0      0   100%

gnocchiclient/auth.py                            51     23      6      0    49%

gnocchiclient/benchmark.py                      175    175     36      0     0%

--省略--

---------------------------

总计                                          2040   1868    424      6     8%

=== 在 5.00 秒内通过 ===

*清单 6-13：在pytest中使用覆盖率*

--cov选项会在测试运行结束时启用覆盖率报告。你需要传递包名作为参数，以便插件正确地过滤覆盖率报告。输出包括未被执行的代码行，因此没有测试。现在你只需启动你喜欢的文本编辑器，开始为这些代码编写测试。

然而，覆盖率更进一步，允许你生成清晰的HTML报告。只需添加--cov-report=html标志，执行该命令的*htmlcov*目录将会填充HTML页面。每个页面都会显示你的源代码的哪些部分已被执行，哪些未被执行。

如果你想成为*那样的*人，可以使用选项--cover-fail-under=COVER_MIN_PERCENTAGE，当测试套件运行时，如果代码的最低执行百分比未达到，测试套件将会失败。虽然拥有良好的覆盖率是一个不错的目标，且该工具在获得测试覆盖率状态的洞察上非常有用，但定义一个任意的百分比值并不会提供太多有价值的洞察。[图 6-1](ch06.xhtml#ch6fig1)展示了一个带有顶部百分比的覆盖率报告示例。

例如，100%的代码覆盖率是一个值得尊敬的目标，但它并不一定意味着代码已经完全测试过，你可以放心了。它只证明了你的所有代码路径都已经运行过；但并没有说明每种可能的条件都已经被测试。

你应该使用覆盖率信息来整合你的测试套件，并为任何当前未运行的代码添加测试。这有助于以后的项目维护，并提高你代码的整体质量。

![image](../images/f06-01.jpg)

*图 6-1：ceilometer.publisher 的覆盖率*

### **虚拟环境**

我们之前提到过，你的测试可能无法捕捉到缺少依赖项的危险。任何具有相当规模的应用程序都会依赖外部库来提供应用程序所需的功能，但外部库可能会在你的操作系统上引发许多问题。以下是其中一些：

+   你的系统没有打包你所需要的库。

+   你的系统没有正确的*版本*的所需库。

+   你需要为两个不同的应用程序提供同一个库的两个不同版本。

这些问题可能发生在你首次部署应用程序时，或者在应用程序运行过程中发生。在通过系统管理器升级已安装的 Python 库时，可能会突然破坏应用程序，原因可能只是库中的 API 发生了变化。

解决方案是让每个应用程序使用一个包含所有应用程序依赖项的库目录。然后，这个目录用于加载所需的 Python 模块，而不是使用系统安装的模块。

这样的目录被称为 *虚拟环境*。

#### ***设置虚拟环境***

工具 virtualenv 会自动为你处理虚拟环境。在 Python 3.2 之前，你可以通过 pip install virtualenv 安装虚拟环境包来使用它。如果你使用的是 Python 3.3 或更高版本，则可以直接通过 Python 使用 venv 名称。

要使用该模块，将其作为主程序加载，并将目标目录作为其参数，如下所示：

$ python3 -m venv myvenv

$ ls foobar

bin        include     lib        pyvenv.cfg

一旦运行，venv 会创建一个 *lib/pythonX.Y* 目录，并使用它将 pip 安装到虚拟环境中，这对于安装进一步的 Python 包非常有用。

然后，你可以通过“源”激活命令来激活虚拟环境。在 Posix 系统上，使用以下命令：

$ source myvenv/bin/activate

在 Windows 系统上，使用以下代码：

> \myvenv\Scripts\activate

一旦完成，命令行提示符前面应显示虚拟环境的名称。执行 python 将调用已复制到虚拟环境中的 Python 版本。你可以通过查看 sys.path 变量并检查它是否将虚拟环境目录作为第一个组件来检查它是否正常工作。

你可以随时通过调用 deactivate 命令停止并退出虚拟环境：

$ deactivate

就这样。还要注意，如果你只想使用一次虚拟环境中安装的 Python，是不强制要求运行 activate 的。直接调用 python 二进制文件也可以工作：

$ myvenv/bin/python

现在，在激活的虚拟环境中，我们无法访问系统主机上安装和可用的任何模块。这就是使用虚拟环境的目的，但也意味着我们可能需要安装所需的包。为此，使用标准的 pip 命令安装每个包，包将安装到正确的位置，而不会改变系统的任何内容：

$ source myvenv/bin/activate

(myvenv) $ pip install six

正在下载/解压 six

正在下载 six-1.4.1.tar.gz

正在为包 six 执行 setup.py egg_info

正在安装已收集的包：six

正在为 six 执行 setup.py install

成功安装 six

清理中...

Voilà！我们可以安装所有需要的库，然后从这个虚拟环境中运行我们的应用程序，而不会破坏系统。很容易看出，我们可以如何编写脚本来自动化基于依赖项列表的虚拟环境安装，正如 [列表 6-14](ch06.xhtml#ch6list14) 所示。

virtualenv myappvenv

source myappvenv/bin/activate

pip install -r requirements.txt

deactivate

*列表 6-14：自动虚拟环境创建*

如果你需要访问系统中安装的包，虚拟环境仍然可以提供帮助，因此 virtualenv 允许你在创建虚拟环境时通过传递 --system-site-packages 标志来启用这些系统包。

在 myvenv 中，你会找到一个 *pyvenv.cfg* 文件，这是该环境的配置文件。默认情况下，它没有很多配置选项。你应该能认出 include-system-site-package，它的作用与我们之前描述的 virtualenv 的 --system-site-packages 相同。

正如你可能猜到的，虚拟环境对于自动化运行单元测试套件非常有用。它们的使用非常广泛，因此专门为此任务开发了一个工具。

#### ***使用 virtualenv 与 tox***

虚拟环境的一个主要用途是为运行单元测试提供一个干净的环境。如果你误以为测试正常运行，但实际上并没有，譬如没有遵守依赖列表，那将是非常有害的。

确保考虑到所有依赖项的一种方式是编写一个脚本来部署虚拟环境，安装 setuptools，然后安装应用程序/库运行时和单元测试所需的所有依赖项。幸运的是，这样的使用场景非常流行，已经有一个专门用于此任务的应用程序——tox。

tox 管理工具旨在自动化和标准化 Python 中测试的运行方式。为此，它提供了在干净的虚拟环境中运行整个测试套件所需的一切，同时还会安装你的应用程序，以检查安装是否成功。

在使用 tox 之前，你需要提供一个名为 *tox.ini* 的配置文件，应该将其放置在项目的根目录中，与 *setup.py* 文件一起：

$ touch tox.ini

然后你可以成功运行 tox：

% tox

GLOB sdist-make: /home/jd/project/setup.py

python create: /home/jd/project/.tox/python

python inst: /home/jd/project/.tox/dist/project-1.zip

____________________ 总结 _____________________

python: commands succeeded

恭喜你 :)

在这个例子中，tox 使用默认的 Python 版本在 *.tox/python* 中创建一个虚拟环境。它使用 *setup.py* 来创建你的包的分发版本，然后将其安装到该虚拟环境中。没有运行任何命令，因为我们在配置文件中没有指定任何命令。仅此而已，并没有特别的用途。

我们可以通过在测试环境中添加要运行的命令来改变这种默认行为。编辑*tox.ini*并添加以下内容：

[testenv]

commands=pytest

现在，tox运行pytest命令。然而，由于我们在虚拟环境中没有安装pytest，这个命令很可能会失败。我们需要将pytest列为一个依赖项以便安装：

[testenv]

deps=pytest

commands=pytest

现在运行时，tox会重新创建环境，安装新的依赖项，并运行pytest命令，这将执行所有单元测试。要添加更多依赖项，你可以将它们列在deps配置选项中，如此处所示，或者使用-rfile语法从文件中读取。

#### ***重新创建环境***

有时你需要重新创建一个环境，例如，当新开发者克隆源代码仓库并首次运行tox时，确保一切按预期工作。为此，tox接受--recreate选项，它将根据你指定的参数从头开始重建虚拟环境。

你在*tox.ini*的[testenv]部分定义了所有虚拟环境的参数。如前所述，tox可以管理多个Python虚拟环境——实际上，通过向tox传递-e标志，你可以在除默认Python版本之外的其他Python版本下运行我们的测试，如下所示：

% tox -e py26

GLOB sdist-make: /home/jd/project/setup.py

py26 创建: /home/jd/project/.tox/py26

py26 安装依赖项: nose

py26 安装: /home/jd/project/.tox/dist/rebuildd-1.zip

py26 运行测试: commands[0] | pytests

--snip--

== 测试会话开始 ==

=== 5个测试通过，耗时4.87秒 ====

默认情况下，tox会模拟任何匹配现有Python版本的环境：py24, py25, py26, py27, py30, py31, py32, py33, py34, py35, py36, py37, jython和pypy！此外，你可以定义自己的环境。只需添加一个名为[testenv:_envname_]的部分。如果你只想为某个特定环境运行某个命令，可以在*tox.ini*文件中列出以下内容：

[testenv]

deps=pytest

commands=pytest

[testenv:py36-coverage]

deps={[testenv]deps}

pytest-cov

commands=pytest --cov=myproject

通过在py36-coverage部分使用pytest --cov=myproject，如下所示，你可以覆盖py36-coverage环境中的命令，这意味着当你运行tox -e py36-coverage时，pytest会作为依赖项被安装，但实际运行的命令是带有coverage选项的pytest。为了使其工作，必须安装pytest-cov扩展：为此，我们将deps值替换为testenv中的deps，并添加pytest-cov依赖项。tox也支持变量插值，因此你可以引用*tox.ini*文件中的任何其他字段并将其作为变量使用，语法为{[env_name]variable_name}。这样可以避免重复写相同的内容。

#### ***使用不同的Python版本***

我们还可以通过以下配置，在 *tox.ini* 中立即创建一个使用不支持的 Python 版本的新环境：

[testenv]

deps=pytest

commands=pytest

[testenv:py21]

basepython=python2.1

当我们运行这个命令时，它现在将（尝试）使用 Python 2.1 来运行测试套件——尽管由于你很可能没有在系统上安装这个古老的 Python 版本，我怀疑它对你有效！

你可能希望支持多个 Python 版本，在这种情况下，默认情况下让 tox 为你想要支持的所有 Python 版本运行所有测试将非常有用。你可以通过在没有任何参数的情况下运行 tox 来指定环境列表：

[tox]

envlist=py35,py36,pypy

[testenv]

deps=pytest

commands=pytest

当没有进一步的参数时运行 tox 时，所有列出的四个环境都会被创建，安装依赖项和应用程序，然后使用命令 pytest 运行。

#### ***集成其他测试***

我们还可以使用 tox 集成像 flake8 这样的测试，正如 [第 1 章](ch01.xhtml#ch01) 中讨论的那样。以下 *tox.ini* 文件提供了一个 PEP 8 环境，它将安装 flake8 并运行它：

[tox]

envlist=py35,py36,pypy,pep8

[testenv]

deps=pytest

commands=pytest

[testenv:pep8]

deps=flake8

commands=flake8

在这种情况下，pep8 环境使用默认版本的 Python 运行，这通常是可以的，但如果你想更改它，仍然可以指定 basepython 选项。

运行 tox 时，你会注意到所有的环境都按顺序构建并运行。这可能会使过程变得非常长，但由于虚拟环境是隔离的，因此没有什么阻止你并行运行 tox 命令。这正是 detox 包所做的，通过提供一个 detox 命令，来并行运行 *envlist* 中的所有默认环境。你应该安装它！

### **测试政策**

将测试代码嵌入项目是一个极好的主意，但如何运行这些代码同样至关重要。许多项目中有测试代码存在，但由于某些原因无法运行。这个话题不仅限于 Python，但我认为它足够重要，值得在这里强调：你应该对未测试的代码采取零容忍政策。没有经过适当单元测试的代码不应被合并。

你应该至少确保每个你推送的提交都能通过所有测试。自动化这个过程更好。例如，OpenStack 依赖于基于 *Gerrit*（一个基于 web 的代码审查服务）和 *Zuul*（一个持续集成和交付服务）的特定工作流。每个提交都会通过 Gerrit 提供的代码审查系统，Zuul 负责运行一系列测试任务。Zuul 会为每个项目运行单元测试和各种更高层次的功能测试。这个由几位开发者执行的代码审查，确保所有提交的代码都有相应的单元测试。

如果你使用的是流行的GitHub托管服务，*Travis CI*是一个工具，可以在每次推送、合并后或提交的拉取请求中运行测试。虽然在推送后进行测试是遗憾的，但它仍然是跟踪回归的绝佳方式。Travis默认支持所有主要的Python版本，并且可以进行显著的自定义。一旦通过*https://www.travis-ci.org/*的Web界面在你的项目中激活Travis，只需添加一个*.travis.yml*文件来确定如何运行测试。[清单6-15](ch06.xhtml#ch6list15)展示了一个.*travis.yml*文件的示例。

语言：python

python：

- "2.7"

- "3.6"

# 安装依赖项的命令

安装："pip install -r requirements.txt --use-mirrors"

# 运行测试的命令

脚本：pytest

*清单6-15：一个.travis.yml示例文件*

将此文件放入你的代码库并启用Travis后，Travis将启动一组作业，使用相关的单元测试对你的代码进行测试。很容易看到，你可以通过简单地添加依赖项和测试来定制这一过程。Travis是一个付费服务，但好消息是，对于开源项目，它完全免费！

tox-travis包（*[https://pypi.python.org/pypi/tox-travis/](https://pypi.python.org/pypi/tox-travis/)）也值得研究，它通过根据所使用的Travis环境运行正确的tox目标，优化了tox与Travis之间的集成。[清单6-16](ch06.xhtml#ch6list16)展示了一个示例*.travis.yml*文件，它将在运行tox之前安装tox-travis。

sudo：false

语言：python

python：

- "2.7"

- "3.4"

安装：pip install tox-travis

脚本：tox

*清单6-16：带有tox-travis的.travis.yml示例文件*

使用tox-travis，你只需在Travis中调用tox作为脚本，它会使用你在*.travis.yml*文件中指定的环境来调用tox，构建必要的虚拟环境，安装依赖项，并运行你在*tox.ini*中指定的命令。这使得在本地开发机器和Travis持续集成平台上使用相同的工作流程变得容易。

如今，无论你的代码托管在哪里，始终可以对软件进行自动化测试，并确保你的项目在向前推进，而不是因为引入漏洞而停滞不前。

### **罗伯特·柯林斯关于测试**

罗伯特·柯林斯不仅是*Bazaar*分布式版本控制系统的原创作者，还担任HP云服务的杰出技术专家，致力于OpenStack的工作。罗伯特还是本书中描述的许多Python工具的作者，比如fixtures、testscenarios、testrepository，甚至python-subunit——你可能在不知情的情况下使用过他的某个程序！

**你建议使用什么样的测试策略？是否有时候可以不进行代码测试？**

我认为测试是一个工程权衡：你必须考虑故障未被检测到而滑入生产环境的可能性、未被检测到的故障的成本和规模，以及执行工作的团队的凝聚力。以 OpenStack 为例，它有 1,600 个贡献者：如此多人各有不同意见，很难协作制定微妙的政策。一般来说，一个项目需要一些自动化测试，以检查代码是否按预期工作，以及它是否做了所需的事情。通常这需要跨不同代码库的功能测试。单元测试非常适合快速定位和解决边界情况。我认为在测试风格之间调整平衡是可以的，只要有进行测试。

当测试成本非常高，而收益非常低时，我认为做出不进行测试的知情决定是可以接受的，但这种情况相对较少：大多数事情可以以较低的成本进行合理的测试，且早期发现错误的收益通常很高。

**编写 Python 代码时，哪些策略能使测试变得可管理并提高代码质量？**

将不同的关注点分离开来，不要在一个地方做多个事情；这样可以让重用变得自然，并且让测试替代品的使用更加容易。尽可能采用纯粹的函数式方法；例如，在一个方法中要么进行计算，要么改变某些状态，但避免同时做两者。这样你可以仅测试所有的计算行为，而不涉及状态变化，比如写入数据库或与 HTTP 服务器交互。反过来也有好处——你可以替换测试中的计算逻辑，激发边界情况行为，并使用 mocks 和测试替代品来检查预期的状态传播是否如愿发生。最难测试的情况是深层堆栈和复杂的跨层行为依赖。对于这种情况，你需要使代码演进，以确保各层之间的契约简单、可预测，并且——对测试最有用——可替代。

**如何在源代码中组织单元测试最为合适？**

拥有清晰的层次结构，比如 *$ROOT / $PACKAGE / tests*。我倾向于为整个源代码树做一个层次结构，例如 *$ROOT / $PACKAGE / $SUBPACKAGE / tests*。

在测试中，我经常镜像源代码树的结构：* $ROOT / $PACKAGE / foo.py* 会在 *$ROOT / $PACKAGE / tests / test_foo.py* 中进行测试。

其他部分的代码不应导入测试代码树，除了可能在顶层的 __init__ 中使用 test_suite/load_tests 函数。这使得你可以轻松地将测试与小体积的安装隔离开来。

**你如何看待 Python 中单元测试库和框架的未来？**

我看到的主要挑战有以下几点：

+   新机器中并行能力的持续扩展，例如拥有四个CPU的手机。现有的单元测试内部API并未针对并行工作负载进行优化。我在StreamResult Java类上的工作正是直接解决这一问题。

+   更复杂的调度支持——为类和模块作用域设置目标提供一种不那么丑陋的解决方案。

+   寻找某种方法来整合我们今天所拥有的各种框架：对于集成测试来说，能够在多个使用不同测试运行器的项目之间获得一个整合视图将是非常有帮助的。
