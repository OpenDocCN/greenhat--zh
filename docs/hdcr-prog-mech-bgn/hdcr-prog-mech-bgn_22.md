## 17

从文件读取输入

![Image](../images/common01.jpg)

我们开发的任何工程应用都需要一些数据输入。例如，要使用我们在上一章中开发的算法解决一个桁架结构问题，我们首先需要构建结构模型。每次想要解决结构问题时手动实例化类来构建模型是非常繁琐的；如果我们能简单地将一个符合给定且明确定义的方案的纯文本文件传递给应用程序，定义我们想要解决的结构，那将更加方便。在本章中，我们将为我们的应用程序配备一个文件解析器功能，它可以读取文本文件，解释文件内容，并构建应用程序内部使用的模型。

### **定义输入格式**

为了使我们的应用程序正常工作，我们提供给它的文件必须具有明确定义的结构。文本文件必须包括节点的定义、施加到它们的载荷以及结构的杆件。让我们为这些部分决定一种格式。

#### ***节点格式***

每个节点将在它自己的行中定义，遵循以下格式，

```py
<node_id>: (<x_coord>, <y_coord>) (<external_constraints>)
```

其中

+   *node_id* 是赋予节点的 ID。

+   *x_coord* 是节点的 x 位置。

+   *y_coord* 是节点的 y 位置。

+   *external_constraints* 是一组约束运动。

这是一个例子：

```py
1: (250, 400) (xy)
```

这定义了一个 ID 为 1 的节点，位置为 (250, 400)，其 x 和 y 位移受到外部约束。

#### ***载荷格式***

载荷将与其应用的节点分开定义，因此我们需要指明载荷应用的节点 ID。将节点和载荷定义在不同的行可以通过使用两个简单的正则表达式（一个用于节点，另一个用于载荷）来简化输入解析过程，而不必使用一个长且复杂的正则表达式。每个载荷将在单独的一行中定义。

我们将使用以下格式来定义载荷，

```py
<node_id> -> (<Fx>, <Fy>)
```

其中

+   *node_id* 是载荷作用的节点。

+   *Fx* 是载荷的 x 分量。

+   *Fy* 是载荷的 y 分量。

这是一个例子：

```py
3 -> (500, -1000)
```

这定义了一个载荷 ⟨500,–1000⟩，作用在 ID 为 3 的节点上。我们使用 -> 字符序列来分隔节点 ID 和载荷分量，而不是使用冒号，以便明确我们不是将 ID 分配给载荷本身，而是将载荷应用于具有该 ID 的节点。

#### ***杆件格式***

杆件定义在两个节点之间，并具有截面和杨氏模量。与节点和载荷一样，每个杆件将定义在它自己的行中。我们可以为杆件设置以下格式，

```py
<bar_id>: (<start_node_id> -> <end_node_id>) <A> <E>
```

其中

+   *bar_id* 是赋予杆件的 ID。

+   *start_node_id* 是起始节点的 ID。

+   *end_node_id* 是结束节点的 ID。

+   *A* 是横截面积。

+   *E* 是杨氏模量。

这是一个例子：

```py
1: (1 -> 2) 30 20000000
```

这定义了一个位于节点 1 和 2 之间的杆件，横截面积为 30，杨氏模量为 20000000。这个杆件的 ID 设置为 1。

#### ***文件格式***

现在我们已经为节点、荷载和杆件设计了格式，让我们看看如何将它们组合到一个文件中。我们正在寻找一种既容易手动编写又容易解析的文件结构。

一个有趣的想法是将文件分成几个部分，每个部分由一个标题开始：

```py
<section_name>
```

每个部分应仅包含定义相同类型实体的行。

由于我们的结构定义文件将包含三种不同类型的实体——节点、荷载和杆件——因此它们需要三个不同的部分。例如，我们在上一章的单元测试中使用的结构，这里作为[图 17-1](ch17.xhtml#ch17fig1)展示，将按如下方式定义：

![图片](../images/17fig01.jpg)

*图 17-1：来自上一章单元测试的结构*

```py
nodes
1: (0, 0)     (xy)
2: (0, 200)   (xy)
3: (400, 200) ()

loads
3 -> (500, -1000)

bars
1: (1 -> 2) 5 10
2: (2 -> 3) 5 10
3: (1 -> 3) 5 10
```

现在我们已经为结构定义文件定义了格式，接下来需要开发一个解析器。一个*解析器*是一个组件（函数或类），它读取文本，解释它，并将其转换为数据结构或模型。在这种情况下，模型是我们的桁架结构类：Structure。我们将像在[第9章](ch09.xhtml#ch09)中一样使用正则表达式。

### **查找正则表达式**

如果我们事先知道结构，正则表达式是一种可靠的提取纯文本中所有需要信息的方法。我们将需要三种不同的正则表达式：一个用于节点，一个用于荷载，一个用于杆件。如果你需要复习正则表达式，请花点时间回顾一下[第9页](ch01.xhtml#page_9)中的“正则表达式”部分。让我们设计这些正则表达式。

#### ***节点正则表达式***

为了匹配我们格式中定义的节点，我们可以使用以下正则表达式：

/(?P<id>\d+)\s*:\s*

\((?P<pos>[\d\s\.,\-]+)\)\s*

\((?P<ec>[xy]{0,2})\)/

这是一个非常复杂的正则表达式。它被分成了几行，因为它太长，无法放入一行，但你可以想象它只是一个单行的正则表达式。让我们分解这个正则表达式的各个部分。

(?P<id>\d+) 这匹配节点的 ID，一个由一位或多位数字组成的数字（\d+），并将其捕获为名为 id 的组。

\s*:\s* 这匹配 ID 后的冒号，并允许冒号前后有任意数量的空格（\s*）。

\((?P<pos>[\d\s\.,\-]+)\) 这匹配括号内的节点位置坐标，并将其捕获为名为 pos 的组。请注意，我们匹配的是括号内的整个表达式；它包括两个坐标和分隔它们的逗号。我们将在代码中分割这两个数字。我们这样做是为了避免让我们已经非常庞大的正则表达式变得更加可怕。将正则表达式与 Python 的字符串操作方法结合起来是一种强大的技术。

\s* 这匹配零个或多个空格，用于分隔坐标组和外部约束组。

\((?P<ec>[xy]{0, 2})\) 这一部分匹配括号内定义的外部约束，并将其捕获到名为ec的组中。括号内的内容限于字符集[xy]，即字符“x”和“y”。字符数也有限制，允许的字符数为0到2之间（{0, 2}）。

我们很快会看到这个正则表达式的实际应用。[图17-2](ch17.xhtml#ch17fig2)可能会帮助你理解正则表达式中每个子部分的含义。

![图片](../images/17fig02.jpg)

*图17-2：节点正则表达式的可视化*

让我们来看一下如何解析负载。

#### ***负载正则表达式***

为了匹配使用我们定义的格式编写的负载，我们将使用以下正则表达式：

/(?P<node_id>\d+)\s*->\s*\((?P<vec>[\d\s\.,\-]+)\)/

这个正则表达式并不像前一个那样可怕；让我们将其分解为多个子部分。

(?P<node_id>\d+) 这个表达式匹配节点ID，并将其捕获到名为node_id的组中。

\s*->\s* 这个表达式匹配`->`字符序列及其周围的可选空格。

\((?P<vec>[\d\s\.,\-]+)\) 这个表达式匹配括号内的整个内容，其中定义了力向量分量。括号内允许的字符集是[\d\s\.,\-]，包括数字、空格、点、逗号和减号。捕获的内容会存储在名为vec的捕获组中。

[图17-3](ch17.xhtml#ch17fig3)是正则表达式不同部分的分解。确保你理解它们的每一部分。

![图片](../images/17fig03.jpg)

*图17-3：负载正则表达式的可视化*

最后，让我们来看看条形的正则表达式。

#### ***条形正则表达式***

为了匹配使用我们之前定义的格式编写的条形，我们将使用以下正则表达式：

/(?P<id>\d+)\s*:\s*

\((?P<start_id>\d+)\s*->\s*(?P<end_id>\d+)\)\s*

(?P<sec>[\d\.]+)\s+

(?P<young>[\d\.]+)/

这个正则表达式因为其长度也被拆分成了几行，但你可以想象它是写在一行中的。让我们逐步分析它：

(?P<id>\d+) 这个表达式匹配分配给条形的ID，并将其捕获到名为id的组中。

\s*:\s* 这个表达式匹配冒号字符及其周围的可选空白字符。

\((?P<start_id>\d+)\s*->\s*(?P<end_id>\d+)\) 这个表达式匹配由`->`字符序列分隔并且周围有可选空格的两个节点ID。ID会被捕获到名为start_id和end_id的组中。整个表达式需要出现在括号内。

\s* 这个表达式匹配最后一个括号与下一个值（即节）之间的可选空白字符。

(?P<sec>[\d\.]+) 这个表达式捕获一个十进制数，并将其赋值给名为sec的组。

\s+ 这个正则表达式匹配括号和下一个值（杨氏模量）之间的必需空格。回想一下，在这种情况下，我们至少需要一个空格，否则无法知道截面值和杨氏模量值的起始和结束位置。

(?P<young>[\d\.]+) 这个捕获的是一个小数，并将其分配给名为 young 的组。

这是我们在本书中看到的最大和最复杂的正则表达式。[图 17-4](ch17.xhtml#ch17fig4) 应该帮助你识别它的各个部分。

![图片](../images/17fig04.jpg)

*图 17-4：条形正则表达式可视化*

现在我们已经有了正则表达式，让我们开始编写解析结构文件的代码。

### **设置**

现在，我们的*structures*包包含以下子目录：

structures

|- 模型

|- 解决方案

|- 测试

让我们通过右键点击 *structures* 并选择 **新建** ▸ **Python 包** 来创建一个名为 *parse* 的新包文件夹。如果你是在 IDE 外部进行操作，别忘了在文件夹中创建一个空的 *__init__.py* 文件。我们的 *structures* 包目录应该如下所示：

structures

|- 模型

|- 解析

|- 解决方案

|- 测试

我们已经准备好开始实现代码了。我们将首先实现解析节点、载荷和条形的逻辑。每个逻辑将定义在自己的函数中，并包含单元测试。然后，我们将把这些内容整合到一个函数中，该函数读取整个文件的内容，将其拆分成行，并将每一行解析成正确的模型类。

### **解析节点**

我们从节点开始。在 *structures/parse* 中，创建一个新文件，命名为 *node_parse.py*。在此文件中，输入[清单 17-1](ch17.xhtml#ch17lis1)中的代码。

```py
import re

from geom2d import Point
from structures.model.node import StrNode

__NODE_REGEX = r'(?P<id>\d+)\s*:\s*' \
               r'\((?P<pos>[\d\s\.,\-]+)\)\s*' \
               r'\((?P<ec>[xy]{0,2})\)'

def parse_node(node_str: str):
 ➊ match = re.match(__NODE_REGEX, node_str)
    if not match:
        raise ValueError(
            f'Cannot parse node from string: {node_str}'
        )

 ➋ _id = int(match.group('id'))
 ➌ [x, y] = [
        float(num)
        for num in match.group('pos').split(',')
    ]
 ➍ ext_const = match.group('ec')

 ➎ return StrNode(
        _id,
        Point(x, y),
        None,
        'x' in ext_const,
        'y' in ext_const
    )
```

*清单 17-1：从字符串解析节点*

我们从定义之前看到的正则表达式开始。它需要被拆分成多行，因为单行长度太长，但由于我们使用了续行反斜杠字符（\），Python 会将所有内容读取为单行。

接下来是 parse_node 函数，它接受一个字符串参数作为输入。这个字符串应该按照我们之前定义的节点格式进行格式化。我们在 node_str 字符串中查找与节点正则表达式 ➊ 的匹配项。如果没有匹配项，我们将引发 ValueError 错误，并附带错误字符串，这样更容易调试错误。

然后我们从名为 id 的捕获组中提取 ID，并将其存储在 _id 变量中 ➋。

接下来，我们解析 x 和 y 位置坐标：我们读取 pos 捕获组的内容，并使用逗号字符分割字符串。

```py
match.group('pos').split(',')
```

这将返回表示节点位置的两个字符串。

使用列表推导式，我们将每个字符串映射为浮点数：

```py
[x, y] = [
    float(num)
    for num in match.group('pos').split(',')
]
```

然后，我们将结果解构为变量 x 和 y ➌。

最后一个命名捕获组是 ec。它包含了外部约束的定义。我们读取其内容并将其存储在变量 ext_const ➍ 中。最后，我们创建节点实例，传递它所需的所有参数 ➎。我们传递 ID、位置点、None 作为负载（稍后会添加），以及外部约束。外部约束是通过检查约束字符串中是否包含字符 “x” 或 “y” 来添加的。为此，我们使用 Python 的 in 运算符，它检查给定的值是否存在于一个序列中。这里有一个示例：

```py
>>> 'hardcore' in 'hardcore programming for mechanical engineers'
True

>>> 3 in [1, 2]
False
```

让我们使用一些单元测试来确保我们的代码能够正确解析节点。

#### ***测试节点解析器***

让我们在 *structures/tests* 目录下创建一个新的测试文件，命名为 *node_parse_test.py*。在该文件中，输入 [清单 17-2](ch17.xhtml#ch17lis2) 中的代码。

```py
import unittest

from geom2d import Point
from structures.parse.node_parse import parse_node

class NodeParseTest(unittest.TestCase):
 ➊ node_str = '1 : (25.0, 45.0)   (xy)'
 ➋ node = parse_node(node_str)

    def test_parse_id(self):
        self.assertEqual(1, self.node.id)

    def test_parse_position(self):
        expected = Point(25.0, 45.0)
        self.assertEqual(expected, self.node.position)

    def test_parse_dx_external_constraint(self):
        self.assertTrue(self.node.dx_constrained)

   def test_parse_dy_external_constraint(self):
        self.assertTrue(self.node.dy_constrained)
```

*清单 17-2：测试节点解析*

这个文件定义了一个新的测试类：NodeParseTest。我们已经定义了一个具有正确格式的字符串，以便我们可以测试是否能够解析其所有部分。这个字符串是 node_str ➊。我们编写了所有测试，以便与解析该字符串 ➋ 后得到的节点一起使用；我们这样做是为了避免在每个测试中重复相同的解析操作。

然后，我们有一个测试来确保结果节点中的 ID 被正确设置，另一个测试检查节点的位置，还有两个测试用来验证外部约束是否已被添加。

让我们运行测试，确保它们都通过了。你可以通过 IDE 或在 shell 中使用以下命令来完成：

```py
$ python3 -m unittest structures/tests/node_parse_test.py
```

现在，让我们开始解析条形图。

### **解析条形图**

在 *structures/parse* 目录下，创建一个名为 *bar_parse.py* 的新文件。在该文件中，输入 [清单 17-3](ch17.xhtml#ch17lis3) 中的代码。

```py
import re
from structures.model.bar import StrBar

__BAR_REGEX = r'(?P<id>\d+)\s*:\s*' \
              r'\((?P<start_id>\d+)\s*->\s*(?P<end_id>\d+)\)\s*' \
              r'(?P<sec>[\d\.]+)\s+' \
              r'(?P<young>[\d\.]+)'

def parse_bar(bar_str: str, nodes_dict):
 ➊ match = re.match(__BAR_REGEX, bar_str)
    if not match:
        raise ValueError(
            f'Cannot parse bar from string: {bar_str}'
        )

 ➋ _id = int(match.group('id'))
 ➌ start_id = int(match.group('start_id'))
 ➍ end_id = int(match.group('end_id'))
 ➎ section = float(match.group('sec'))
 ➏ young_mod = float(match.group('young'))

 ➐ start_node = nodes_dict[start_id]
    if start_node is None:
        raise ValueError(f'Node with id: ${start_id} undefined')

    end_node = nodes_dict[end_id]
    if end_node is None:
        raise ValueError(f'Node with id: ${start_id} undefined')

 ➑ return StrBar(_id, start_node, end_node, section, young_mod)
```

*清单 17-3：从字符串解析一个条形图*

用于匹配条形图定义的正则表达式 (__BAR_REGEX) 有点长且复杂。确保小心输入它。我们稍后会编写一些单元测试，所以这里的任何错误将在测试中暴露出来。

我们编写了 parse_bar 函数，它接受两个参数：定义条形图的字符串和一个节点字典。在这个字典中，键是节点的 ID，值是节点本身。条形图需要引用其结束节点，因此必须先解析这些节点，然后传递给 parse_bar 函数。这对我们解析结构文件的方式增加了一个约束：节点应该首先出现。

与节点一样，我们首先将传入的字符串与我们的正则表达式 ➊ 进行匹配。如果没有匹配项，我们会引发一个 ValueError，并附上一个有帮助的消息，包含无法解析的字符串。

接下来，我们检索并解析捕获组：id 解析为整数 ➋，start_id ➌ 和 end_id ➍ 解析为整数，sec ➎ 和 young ➏ 解析为浮动数值。

然后，我们在节点字典中查找起始节点 ➐，如果未找到，则抛出错误：我们不能构建没有节点的条形。同样处理终止节点，然后在最后一行 ➑ 创建并返回条形实例，传入所有解析的值。

让我们测试一下这个代码。

#### ***测试条形解析器***

为了测试条形的解析过程，在 *structures/tests* 中创建一个名为 *bar_parse_test.py* 的新文件。将新的测试代码输入到 [列表 17-4](ch17.xhtml#ch17lis4)。

```py
import unittest

from structures.parse.bar_parse import parse_bar

class BarParseTest(unittest.TestCase):
 ➊ bar_str = '1: (3 -> 5) 25.0 20000000.0'
 ➋ nodes_dict = {
        3: 'Node 3',
        5: 'Node 5'
    }
 ➌ bar = parse_bar(bar_str, nodes_dict)

    def test_parse_id(self):
        self.assertEqual(1, self.bar.id)

    def test_parse_start_node(self):
        self.assertEqual('Node 3', self.bar.start_node)

    def test_parse_end_node_id(self):
        self.assertEqual('Node 5', self.bar.end_node)

    def test_parse_section(self):
        self.assertEqual(25.0, self.bar.cross_section)

    def test_parse_young_modulus(self):
        self.assertEqual(20000000.0, self.bar.young_mod)
```

*列表 17-4：测试条形的解析*

在这个测试中，我们使用条形的字符串表示法 ➊ 来定义一个条形。`parse_bar` 函数需要一个包含按 ID 分类的节点字典作为第二个参数；我们创建了一个虚拟字典（回想一下第 16 页 [447](ch16.xhtml#ch00lev1sec97) 中的类型），称为 `nodes_dict` ➋。这个字典包含两个节点 ID，并将它们映射到一个字符串。我们的解析代码实际上并不会对节点做任何处理，甚至不会检查它们的类型；它只是简单地将它们添加到条形实例中。因此，在测试中，模拟节点的字符串就足够了。

同样，我们首先解析 ➌ 并将结果存储在 `bar` 变量中。然后我们创建五个测试，检查是否正确解析了 ID、起始节点、终止节点、截面和杨氏模量。

运行测试，确保它们都通过了。你可以从命令行进行测试：

```py
$ python3 -m unittest structures/tests/bar_parse_test.py
```

最后，我们需要解析负载。

### **解析负载**

我们现在编写一个函数来解析负载字符串，但我们不会在这里将负载应用到节点上。这将在稍后将所有部分结合时进行。

在 *structures/parse* 文件夹中创建一个名为 *load_parse.py* 的新文件。将代码输入到 [列表 17-5](ch17.xhtml#ch17lis5)。

```py
import re

from geom2d import Vector

__LOAD_REGEX = r'(?P<node_id>\d+)\s*->\s*' \
               r'\((?P<vec>[\d\s\.,\-]+)\)'

def parse_load(load_str: str):
 ➊ match = re.match(__LOAD_REGEX, load_str)
    if not match:
        raise ValueError(
            f'Cannot parse load from string: "{load_str}"'
        )

 ➋ node_id = int(match.group('node_id'))
 ➌ [fx, fy] = [
        float(num)
        for num in match.group('vec').split(',')
    ]

 ➍ return node_id, Vector(fx, fy)
```

*列表 17-5：从字符串中解析负载*

在这个列表中，我们定义了匹配负载的正则表达式 `__LOAD_REGEX`。接着是 `parse_load` 函数，它首先在传入的字符串（`load_str`）中查找匹配项 ➊。如果字符串不匹配 `__LOAD_REGEX`，我们会抛出一个错误。

正则表达式定义了两个捕获组：`node_id` 和 `vec`。第一个组是需要施加负载的节点的 ID。我们将第一个组的值转换为整数，并将其存储在 `node_id` 变量中 ➋。

为了提取力的分量，我们拆分由 `vec` 捕获组匹配到的值，然后解析每个部分，将其转换为浮动值，并使用解构赋值将分量提取到 `fx` 和 `fy` 变量中 ➌。

最后，我们返回一个元组，包含节点 ID 和一个带有力的分量的向量 ➍。

让我们测试一下这个逻辑，确保它能正确解析负载。

#### ***测试负载解析器***

在 *structures/tests* 文件夹中创建一个名为 *load_parse_test.py* 的新文件。将测试代码输入到 [列表 17-6](ch17.xhtml#ch17lis6)。

```py
import unittest

from geom2d import Vector
from structures.parse.load_parse import parse_load

class LoadParseTest(unittest.TestCase):

    load_str = '1 -> (250.0, -3500.0)'
    (node_id, load) = parse_load(load_str)

    def test_parse_node_id(self):
        self.assertEqual(1, self.node_id)

    def test_parse_load_vector(self):
        expected = Vector(250.0, -3500.0)
        self.assertEqual(expected, self.load)
```

*列表 17-6：测试负载的解析*

这个测试定义了一个表示施加到 ID 为 1 的节点上的负载的字符串，负载的分量是 ⟨250.0,–3500.0⟩。该字符串存储在 `load_str` 变量中并传递给 `parse_load` 函数。

在第一次测试中，我们检查是否正确解析了节点 ID，节点 ID 作为元组的第一个值由函数返回。然后，我们检查是否正确解析了元组的第二个值，即向量。这两个简单的测试足以确保我们的函数能够正常工作。

从 IDE 或命令行运行测试：

```py
$ python3 -m unittest structures/tests/load_parse_test.py
```

现在我们已经有了可以解析结构各个部分的函数，是时候将它们组合起来了。在接下来的章节中，我们将编写一个函数，读取结构定义文件的所有行，并生成相应的模型。

### **解析结构**

我们的结构文件将每个实体定义在单独的行上，实体按部分分组出现。如果你还记得，我们为需要解析的三个不同实体定义了三个部分：节点、杆件和荷载。以下是之前的结构文件示例：

```py
nodes
1: (0, 0)     (xy)
2: (0, 200)   (xy)
3: (400, 200) ()

loads
3 -> (500, -1000)

bars
1: (1 -> 2) 5 10
2: (2 -> 3) 5 10
3: (1 -> 3) 5 10
```

因为这些文件大多数是手动编写的，因此如果我们允许包含注释那就太好了：注释行会被解析机制忽略，但能向文件的阅读者解释一些内容，就像代码中的注释一样。

这里是一个示例：

```py
# only node with a load applied
3: (400, 200) ()
```

我们将借用 Python 的语法，使用 # 符号标记注释的开始。注释必须单独占一行。

#### ***概览***

由于我们需要编写一些函数，可能有一个带注解的结构解析过程图会更有帮助，可以标注出各个步骤对应的函数名。请看一下[图17-5](ch17.xhtml#ch17fig5)。

![Image](../images/17fig05.jpg)

*图17-5：结构解析过程*

在这个图示中，我们展示了解析过程的每个步骤。我们从一个结构文件开始，定义了以纯文本形式表示的结构，遵循我们的标准格式。

第一步是将文件内容读取到一个字符串中。我们将在[第19章](ch19.xhtml#ch19)的应用中实现这一部分。

第二步是将大的字符串拆分成多行。

第三步是将这些行解析成结构原始数据的字典。这一步由私有的__parse_lines函数处理。

第四步也是最后一步是将那些解析后的结构项聚合到一个结构实例中。

parse_structure_from_lines 函数是第 3 步和第 4 步的结合：它将一个定义行的列表转化为一个完整的结构。parse_structure 函数更进一步，将单一的字符串拆分成多行。

#### ***设置***

在 *structures/parse* 目录中，创建一个名为 *str_parse.py* 的新文件。此时 *structures* 包应该是这样的：

structures

|- model

|    | ...

|- parse

|    |- __init__.py

|    |- bar_parse.py

|    |- load_parse.py

|    |- node_parse.py

|    |- str_parse.py

|- solution

|    | ...

|- tests

|    | ...

让我们从一个函数开始，该函数判断文件中的一行是否为空白或是注释。这个函数将告诉我们给定的行是否可以忽略，或者是否需要解析。

#### ***忽略空行和注释***

在*str_parse.py*中，输入[Listing 17-7](ch17.xhtml#ch17lis7)中的代码。

```py
__COMMENT_INDICATOR = '#'

def __should_ignore_line(line: str):
    stripped = line.strip()
    return len(stripped) == 0 or \
           stripped.startswith(__COMMENT_INDICATOR)
```

*Listing 17-7：确定需要忽略的行的函数*

我们定义了一个常量`__COMMENT_INDICATOR`，它的值是`#`字符。如果我们想要更改注释的识别方式，只需编辑这一行。

接下来是`__should_ignore_line`函数。该函数接收一个字符串并去除两端的空白字符（换句话说，它会去除字符串的空白）。然后，如果该行的长度为零或以注释标识符开头，函数将返回True值，否则返回False。

#### ***解析行***

现在我们已经有了一种方法来过滤掉不需要解析的行，接下来看看需要解析的行。我们将定义一个接收字符串列表的函数，这些字符串代表行，并识别该行是部分头（“nodes”、“bars”或“loads”）还是实体。如果是部分头，函数将设置一个标志来跟踪当前正在读取的部分。函数的其余部分将负责使用相应的解析器解析每一行。

在文件*str_parse.py*中，输入[Listing 17-8](ch17.xhtml#ch17lis8)中的代码。

```py
import re

from .bar_parse import parse_bar
from .load_parse import parse_load
from .node_parse import parse_node

__COMMENT_INDICATOR = '#'
__NODES_HEADER = 'nodes'
__LOADS_HEADER = 'loads'
__BARS_HEADER = 'bars'

def __parse_lines(lines: [str]):
 ➊ reading = ''
 ➋ result = {'nodes': {}, 'loads': [], 'bars': []}

    for i, line in enumerate(lines):
     ➌ if __should_ignore_line(line):
            continue

        # <--- header ---> #
     ➍ if re.match(__NODES_HEADER, line):
            reading = 'nodes'
        elif re.match(__BARS_HEADER, line):
            reading = 'bars'
        elif re.match(__LOADS_HEADER, line):
            reading = 'loads'

        # <--- definition ---> #
     ➎ elif reading == 'nodes':
            node = parse_node(line)
            result['nodes'][node.id] = node
        elif reading == 'bars':
            bar = parse_bar(line, result['nodes'])
            result['bars'].append(bar)
        elif reading == 'loads':
            load = parse_load(line)
            result['loads'].append(load)
        else:
            raise RuntimeError(
                f'Unknown error in line ${i}: ${line}'
            )

    return result

def __should_ignore_line(line: str):

    --snip--
```

*Listing 17-8：解析行*

我们首先添加了三个变量，分别是文件头的名称：`__NODES_HEADER`、`__LOADS_HEADER`和`__BARS_HEADER`。这些常量定义了各个部分的名称。

接下来是`__parse_lines`函数的定义，它接收一个参数：结构文件中的行列表。该函数声明了一个名为`reading ➊`的变量。此变量指示后续循环当前处于哪个结构部分。例如，当它的值为“bars”时，随后的行应该使用`parse_bar`函数进行解析，直到文件结束或遇到新的部分。

接下来是结果字典`➋`的定义。它通过三个键来初始化：'nodes'、'loads'和'bars'。我们将解析后的元素添加到此字典中，按照它们对应的键的集合进行存储。`loads`和`bars`存储在列表中，`nodes`则存储在字典中，键是它们的ID。我们将节点映射到它们的键存储在字典中，因为在结构文件中，`loads`和`bars`都通过ID来引用节点；因此，当我们将它们关联时，通过ID查找会更方便。

接下来是一个循环，遍历行的枚举。回想一下，Python的`enumerate`函数返回一个可迭代序列，其中包含原始对象以及它们的索引。我们只会在遇到错误时使用该索引，借助行号在错误信息中可以更容易地找到输入文件中的错误。我们对每一行的处理第一步是检查它是否为空行或是注释行 ➌，如果是这种情况，我们会使用`continue`语句跳过该行。

接下来，我们有一些 if-else 语句。它们的第一个块用于匹配头部行 ➍。当找到一个与三种可能的头部之一匹配的行时，我们将 `reading` 变量设置为该头部的值。后面的 if-else 语句评估 `reading`，以确定要解析的结构元素 ➎。如果 `reading` 的值为 'nodes'，我们使用 `parse_node` 函数解析该行，并将结果存储在结果字典的 'nodes' 键下：

```py
result['nodes'][node.id] = node
```

对于 bars 和 loads 也一样，但请记住，在它们的情况下，它们被存储在一个列表中：

```py
result['bars'].append(bar)
```

函数随后返回结果字典。

我们实现了一个函数，读取一系列文本行，并将每一行转换为结构类实例（我们称之为解析）。这些实例代表结构中的节点、bars 和 loads。该函数返回一个将这些实例按类型分类的字典。接下来的步骤是使用这些解析后的对象来构造一个 `Structure` 实例。

#### ***拆分行并实例化结构***

给定结构文件的内容作为字符串，我们希望将这个字符串拆分成行。我们将这些行传递给之前编写的 `__parse_lines` 函数，通过解析后的对象，我们可以构造一个 `Structure` 类的实例。

在 *str_parse.py* 文件中，在 __parse_lines 函数之前，输入 [示例 17-9](ch17.xhtml#ch17lis9) 中的代码。

```py
import re

from structures.model.structure import Structure
from .bar_parse import parse_bar
from .load_parse import parse_load
from .node_parse import parse_node

__COMMENT_INDICATOR = '#'
__NODES_HEADER = 'nodes'
__LOADS_HEADER = 'loads'
__BARS_HEADER = 'bars'

def parse_structure(structure_string: str):
 ➊ lines = structure_string.split('\n')
    return parse_structure_from_lines(lines)

def parse_structure_from_lines(lines: [str]):
 ➋ parsed = __parse_lines(lines)
    nodes_dict = parsed['nodes']
    loads = parsed['loads']
    bars = parsed['bars']

 ➌__apply_loads_to_nodes(loads, nodes_dict)

   return Structure(
     ➍ list(nodes_dict.values()),
        bars
    )

def __apply_loads_to_nodes(loads, nodes):
 ➎ for node_id, load in loads:
        nodes[node_id].add_load(load)

--snip--
```

*示例 17-9：拆分行*

我们编写了三个新函数。第一个函数 `parse_structure` 将传入的字符串拆分成行 ➊，并将这些行传递给随后定义的 `parse_structure_from_lines` 函数。

第二个函数 `parse_structure_from_lines` 将这些行传递给 __parse_lines，并将结果保存在一个名为 parsed ➋ 的变量中。然后，它将结果字典中的内容提取到以下变量中：nodes_dict、loads 和 bars。

负载与它们应用的节点是分开定义的；因此，我们需要将每个负载添加到其相应的节点 ➌。为此，我们编写了另一个小函数：__apply_loads_to_nodes。回想一下，负载是使用以下格式定义的：

```py
1 -> (500, -1000)
```

并且由我们的 `parse_load` 函数解析，作为一个由节点 ID 和负载组件（作为向量）组成的元组：

```py
(1, Vector(500, -1000))
```

记住这一点对于理解 __apply_loads_to_nodes ➎ 中的循环非常重要。该循环遍历负载元组，在每次迭代中，它分别将节点 ID 和负载向量存储到 `node_id` 和 `load` 变量中。因为我们的节点存储在一个字典中，其键是节点 ID，所以应用负载非常简单。

一旦加载操作应用到节点（在parse_structure_from_lines中），最后一步是返回一个Structure类的实例。该类的构造函数需要一个节点列表和一个条形码列表。条形码已经作为列表解析，但节点是以字典的形式存在。为了将字典的值转换为列表，我们只需要使用Python的list函数处理字典的值，这些值通过values()方法提取 ➍。

到此为止，我们的解析逻辑已经准备好了！

#### ***结果***

供您参考，[列表 17-10](ch17.xhtml#ch17lis10)展示了*str_parse.py*的完整代码。

```py
import re

from structures.model.structure import Structure
from .bar_parse import parse_bar
from .load_parse import parse_load
from .node_parse import parse_node

__COMMENT_INDICATOR = '#'
__NODES_HEADER = 'nodes'
__LOADS_HEADER = 'loads'
__BARS_HEADER = 'bars'

def parse_structure(structure_string: str):
    lines = structure_string.split('\n')
    return parse_structure_from_lines(lines)

def parse_structure_from_lines(lines: [str]):
    parsed = __parse_lines(lines)
    nodes_dict = parsed['nodes']
    loads = parsed['loads']
    bars = parsed['bars']

    __apply_loads_to_nodes(loads, nodes_dict)

    return Structure(
        list(nodes_dict.values()),
        bars
    )

def __apply_loads_to_nodes(loads, nodes):
    for node_id, load in loads:
        nodes[node_id].add_load(load)

def __parse_lines(lines: [str]):
    reading = ''
    result = {'nodes': {}, 'loads': [], 'bars': []}

    for i, line in enumerate(lines):
        if __should_ignore_line(line):
            continue

        # <--- header ---> #
        if re.match(__NODES_HEADER, line):
            reading = 'nodes'
        elif re.match(__BARS_HEADER, line):
            reading = 'bars'
        elif re.match(__LOADS_HEADER, line):
            reading = 'loads'

        # <--- definition ---> #
        elif reading == 'nodes':
            node = parse_node(line)
            result['nodes'][node.id] = node
        elif reading == 'bars':
            bar = parse_bar(line, result['nodes'])
            result['bars'].append(bar)
        elif reading == 'loads':
            load = parse_load(line)
            result['loads'].append(load)
        else:
            raise RuntimeError(
                f'Unknown error in line ${i}: ${line}'
            )

    return result

def __should_ignore_line(line: str):
    stripped = line.strip()
    return len(stripped) == 0 or \
           stripped.startswith(__COMMENT_INDICATOR)
```

*列表 17-10：解析结构*

在进入下一部分之前，打开*__init__.py*文件，位于*parse*目录中，并输入以下导入语句：

```py
from .str_parse import parse_structure
```

这允许我们像这样导入parse_structure函数，

```py
from structures.parse import parse_structure
```

而不是这个稍微长一点的版本：

```py
from structures.parse.str_parse import parse_structure
```

让我们通过实现一些自动化测试来确保我们的解析函数正常工作。

#### ***测试结构解析器***

为确保parse_structure函数按预期工作，我们现在将添加一些单元测试。首先，我们需要创建一个结构定义文件用于测试。在*structures/tests*目录中，创建一个新文件，命名为*test_str.txt*，其内容如下：

```py
# Nodes
nodes
1: (0.0, 0.0)      (xy)
2: (200.0, 150.0)  ()
3: (400.0, 0.0)    (y)

# Loads
loads
2 -> (2500.0, -3500.0)

# Bars
bars
1: (1 -> 2) 25 20000000
2: (2 -> 3) 25 20000000
3: (1 -> 3) 25 20000000
```

我们已经添加了注释行和一些额外的空行；我们的函数应该忽略这些。创建一个新的测试文件：*str_parse_test.py*（[列表 17-11](ch17.xhtml#ch17lis11)）。

```py
import unittest

import pkg_resources as res

from structures.parse import parse_structure

class StructureParseTest(unittest.TestCase):

    def setUp(self):
        str_bytes = res.resource_string(__name__, 'test_str.txt')
        str_string = str_bytes.decode("utf-8")
        self.structure = parse_structure(str_string)
```

*列表 17-11：设置结构解析测试*

该文件定义了一个新的测试类：StructureParseTest。在setUp方法中，我们使用resource_string函数将*test_str.txt*文件加载为字节。然后，我们将这些字节解码为UTF-8编码的Python字符串。最后，使用parse_structure解析结构字符串，并将结果存储在类的一个属性中：self.structure。

##### **测试节点解析器**

让我们添加一些测试用例，确保从*test_str.txt*文件解析的结构包含预期的节点。在setUp方法之后，输入第一个测试（[列表 17-12](ch17.xhtml#ch17lis12)）。

```py
import unittest

import pkg_resources as res

from geom2d import Point
from structures.parse import parse_structure

class StructureParseTest(unittest.TestCase):
    --snip--

    def test_parse_nodes_count(self):
        self.assertEqual(3, self.structure.nodes_count)

    def test_parse_nodes(self):
     ➊ nodes = self.structure._Structure__nodes
        self.assertEqual(
            Point(0, 0),
            nodes[0].position
        )
        self.assertEqual(
            Point(200, 150),
            nodes[1].position
        )
        self.assertEqual(
            Point(400, 0),
            nodes[2].position
        )

    def test_parse_node_constraints(self):
        nodes = self.structure._Structure__nodes

        self.assertTrue(nodes[0].dx_constrained)
        self.assertTrue(nodes[0].dy_constrained)

        self.assertFalse(nodes[1].dx_constrained)
        self.assertFalse(nodes[1].dy_constrained)

        self.assertFalse(nodes[2].dx_constrained)
        self.assertTrue(nodes[2].dy_constrained)
```

*列表 17-12：测试结构解析：节点*

我们写了三个测试。第一个测试检查结构中是否有三个节点。下一个测试确保这三个节点的位置正确。

有一点值得注意。由于__nodes列表是Structure类的私有属性，Python通过一个技巧来试图隐藏它。Python会在私有属性的名称前加上一个下划线和类名。因此，__nodes属性将被称为_Structure__nodes，而不是我们预期的__nodes。这就是为什么在我们的测试中，要通过这个名称来访问它 ➊。

第三个也是最后一个测试检查节点中的外部约束是否具有结构定义文件中定义的正确值。让我们运行测试。您可以在IDE中点击绿色播放按钮，或者使用终端：

```py
$ python3 -m unittest structures/tests/str_parse_test.py
```

应该在终端中显示一条成功消息。

##### **测试条形码解析器**

现在，让我们测试杆件是否也被正确解析。在我们刚刚编写的测试用例之后，输入[列表 17-13](ch17.xhtml#ch17lis13)中的测试用例。

```py
class StructureParseTest(unittest.TestCase):
    --snip--

    def test_parse_bars_count(self):
        self.assertEqual(3, self.structure.bars_count)

    def test_parse_bars(self):
        bars = self.structure._Structure__bars

        self.assertEqual(1, bars[0].start_node.id)
        self.assertEqual(2, bars[0].end_node.id)

        self.assertEqual(2, bars[1].start_node.id)
        self.assertEqual(3, bars[1].end_node.id)

        self.assertEqual(1, bars[2].start_node.id)
        self.assertEqual(3, bars[2].end_node.id)
```

*列表 17-13：测试结构解析：杆件*

第一个测试断言结构中有三根杆件。第二个测试检查结构中的每根杆件是否与正确的节点 ID 关联。和以前一样，要访问私有的杆件列表，我们需要在属性名之前加上 _Structure：_Structure__bars。

我邀请你添加两个额外的测试，检查横截面和杨氏模量的值是否正确地解析到杆件中。由于篇幅原因，我们在此不包括这些测试。

再次运行测试类，以确保我们的新测试也通过。在终端中运行以下命令：

```py
$ python3 -m unittest structures/tests/str_parse_test.py
```

##### **测试荷载解析器**

让我们添加最后两个测试，确保荷载被正确解析。输入[列表 17-14](ch17.xhtml#ch17lis14)中的代码。

```py
import unittest

import pkg_resources as res

from geom2d import Point, Vector
from structures.parse import parse_structure

class StructureParseTest(unittest.TestCase):
    --snip--

    def test_parse_loads_count(self):
        self.assertEqual(1, self.structure.loads_count)

    def test_apply_load_to_node(self):
        node = self.structure._Structure__nodes[1]
        self.assertEqual(
            Vector(2500, -3500),
            node.net_load
        )
```

*列表 17-14：测试结构解析：荷载*

在这两个最后的测试中，我们检查结构中的荷载数量是否为 1，并且它是否正确地应用到第二个节点。

让我们运行所有测试，确保所有测试都通过：

```py
$ python3 -m unittest structures/tests/str_parse_test.py
```

如果你的代码实现良好，所有测试应该都通过，并且你应该在终端看到以下内容：

```py
Ran 7 tests in 0.033s

OK
```

##### **测试类结果**

我们已经做了一些测试，因此[列表 17-15](ch17.xhtml#ch17lis15)展示了结果测试类，供您参考。

```py
import unittest

import pkg_resources as res

from geom2d import Point, Vector
from structures.parse import parse_structure

class StructureParseTest(unittest.TestCase):

    def setUp(self):
        str_bytes = res.resource_string(__name__, 'test_str.txt')
        str_string = str_bytes.decode("utf-8")
        self.structure = parse_structure(str_string)

    def test_parse_nodes_count(self):
        self.assertEqual(3, self.structure.nodes_count)

    def test_parse_nodes(self):
        nodes = self.structure._Structure__nodes
        self.assertEqual(
            Point(0, 0),
            nodes[0].position
        )
        self.assertEqual(
            Point(200, 150),
            nodes[1].position
        )
        self.assertEqual(
            Point(400, 0),
            nodes[2].position
        )

    def test_parse_node_constraints(self):
        nodes = self.structure._Structure__nodes

        self.assertTrue(nodes[0].dx_constrained)
        self.assertTrue(nodes[0].dy_constrained)

        self.assertFalse(nodes[1].dx_constrained)
        self.assertFalse(nodes[1].dy_constrained)

        self.assertFalse(nodes[2].dx_constrained)
        self.assertTrue(nodes[2].dy_constrained)

    def test_parse_bars_count(self):
        self.assertEqual(3, self.structure.bars_count)

    def test_parse_bars(self):
        bars = self.structure._Structure__bars

        self.assertEqual(1, bars[0].start_node.id)
        self.assertEqual(2, bars[0].end_node.id)

        self.assertEqual(2, bars[1].start_node.id)
        self.assertEqual(3, bars[1].end_node.id)

        self.assertEqual(1, bars[2].start_node.id)
        self.assertEqual(3, bars[2].end_node.id)

    def test_parse_loads_count(self):
        self.assertEqual(1, self.structure.loads_count)

    def test_apply_load_to_node(self):
        node = self.structure._Structure__nodes[1]
        self.assertEqual(
            Vector(2500, -3500),
            node.net_load
        )
```

*列表 17-15：测试结构解析*

我们的结构解析逻辑已经准备好并通过测试！

### **总结**

在本章中，我们首先定义了结构文件的格式。这是一个简单的纯文本格式，可以手动编写。

然后，我们实现了函数，将结构文件中的每一行解析为相应的结构元素：节点、荷载和杆件。正则表达式是本次工作的明星；有了它们，解析结构化文本变得轻而易举。

最后，我们将所有内容合并到一个函数中，该函数将大字符串分割成各行，并决定每一行使用哪个解析器。我们将使用这个函数来读取结构文件，并创建我们的桁架解法应用程序将使用的结构模型。

现在是时候开始生成结构解法的输出图表了。这正是我们将在下一章中做的事情。
