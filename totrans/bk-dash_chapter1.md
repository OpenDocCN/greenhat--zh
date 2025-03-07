## <samp class="SANS_Futura_Std_Bold_Condensed_B_11">1</samp> <samp class="SANS_Dogma_OT_Bold_B_11">PYTHON 刷新器</samp>

![](img/opener-img.png)

如果你打算做 Dash 应用开发，你可能已经至少了解一点 Python。然而，这本书并不假设你是专家，因此我们将回顾一些在 Dash 中更为相关的 Python 概念，包括列表、字典、面向对象编程以及装饰器函数。如果你已经对这些领域非常自信，可以直接跳到 第二章，该章将介绍我们在整本书中使用的 Python IDE：PyCharm。

### <samp class="SANS_Futura_Std_Bold_B_11">列表</samp>

让我们快速回顾一下几乎所有 Dash 应用程序中都使用的最重要的容器数据类型：Python 列表！列表在 Dash 中很重要，因为它们用于定义布局、结合 Dash Bootstrap 主题，并且常见于回调函数和由 Plotly 构建的图形中。

列表容器类型用于存储元素的序列。列表是可变的，这意味着你可以在创建后修改它们。这里我们创建一个名为 <samp class="SANS_TheSansMonoCd_W5Regular_11">lst</samp> 的列表并打印其长度：

<samp class="SANS_TheSansMonoCd_W5Regular_11">lst = [1, 2, 2]</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(len(lst))</samp>

我们的输出仅为：

<samp class="SANS_TheSansMonoCd_W5Regular_11">3</samp>

我们通过使用方括号和逗号分隔的元素来创建一个列表。列表可以包含任意的 Python 对象、重复值，甚至是其他列表，因此它们是 Python 中最灵活的容器类型之一。这里我们将列表 <samp class="SANS_TheSansMonoCd_W5Regular_11">lst</samp> 填充了三个整数元素。<samp class="SANS_TheSansMonoCd_W5Regular_11">len()</samp> 函数返回列表中元素的数量。

<samp class="SANS_Futura_Std_Bold_Condensed_Oblique_I_11">添加元素</samp>

有三种常见的方式可以向一个已存在的列表中添加元素：附加、插入和连接。

<samp class="SANS_TheSansMonoCd_W5Regular_11">append()</samp> 方法将其参数放在列表的末尾。以下是附加的示例：

<samp class="SANS_TheSansMonoCd_W5Regular_11">lst = [1, 2, 2]</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">lst.append(4)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(lst)</samp>

这将打印：

<samp class="SANS_TheSansMonoCd_W5Regular_11">[1, 2, 2, 4]</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">insert()</samp> 方法在给定位置插入一个元素，并将所有后续元素向右移动。下面是一个插入的示例：

<samp class="SANS_TheSansMonoCd_W5Regular_11">lst = [1, 2, 4]</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">lst.insert(2,2)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(lst)</samp>

这将打印相同的结果：

<samp class="SANS_TheSansMonoCd_W5Regular_11">[1, 2, 2, 4]</samp>

最后，拼接操作：

<samp class="SANS_TheSansMonoCd_W5Regular_11">print([1, 2, 2] +</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">[4])</samp>

我们得到：

<samp class="SANS_TheSansMonoCd_W5Regular_11">[1, 2, 2, 4]</samp>

对于拼接操作，我们使用加号（<samp class="SANS_TheSansMonoCd_W5Regular_11">+</samp>）运算符。这样可以通过将两个现有列表连接起来，创建一个新列表。

所有操作都会生成相同的列表，<samp class="SANS_TheSansMonoCd_W5Regular_11">[1, 2, 2, 4]</samp>。其中 append 操作是最快的，因为它不需要像插入操作那样遍历列表来将元素插入到正确的位置，也不需要像拼接操作那样通过两个子列表创建一个新列表。

若要向给定列表追加多个元素，可以使用 <samp class="SANS_TheSansMonoCd_W5Regular_11">extend()</samp> 方法：

<samp class="SANS_TheSansMonoCd_W5Regular_11">lst = [1, 2]</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">lst.extend([2, 4])</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(lst)</samp>

代码将按如下方式更改现有的列表对象 <samp class="SANS_TheSansMonoCd_W5Regular_11">lst</samp>：

<samp class="SANS_TheSansMonoCd_W5Regular_11">[1, 2, 2, 4]</samp>

上面的代码是一个示例，展示了一个可以容纳重复值的列表。

<samp class="SANS_Futura_Std_Bold_Condensed_Oblique_I_11">删除元素</samp>

我们可以使用 <samp class="SANS_TheSansMonoCd_W5Regular_11">lst.remove(x)</samp> 从列表中移除一个元素 <samp class="SANS_TheSansMonoCd_W5Regular_11">x</samp>，例如：

<samp class="SANS_TheSansMonoCd_W5Regular_11">lst = [1, 2, 2, 4]</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">lst.remove(1)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(lst)</samp>

这会给我们返回结果：

<samp class="SANS_TheSansMonoCd_W5Regular_11">[2, 2, 4]</samp>

该方法直接操作列表对象本身——不会创建新列表，原始列表会被修改。

<samp class="SANS_Futura_Std_Bold_Condensed_Oblique_I_11">反转列表</samp>

你可以使用方法 <samp class="SANS_TheSansMonoCd_W5Regular_11">lst.reverse()</samp> 来反转列表元素的顺序：

<samp class="SANS_TheSansMonoCd_W5Regular_11">lst = [1, 2, 2, 4]</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">lst.reverse()</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(l)</samp>

这将打印：

<samp class="SANS_TheSansMonoCd_W5Regular_11">[4, 2, 2, 1]</samp>

反转列表也会修改原始列表对象，而不是创建一个新列表对象。

<samp class="SANS_Futura_Std_Bold_Condensed_Oblique_I_11">排序列表</samp>

你可以使用方法 <samp class="SANS_TheSansMonoCd_W5Regular_11">lst.sort()</samp> 来对列表元素进行排序：

<samp class="SANS_TheSansMonoCd_W5Regular_11">lst = [2, 1, 4, 2]</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">lst.sort()</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(lst)</samp>

我们看到了排序后的列表：

<samp class="SANS_TheSansMonoCd_W5Regular_11">[1, 2, 2, 4]</samp>

再次提醒，排序列表会修改原始列表对象。默认情况下，结果列表是按升序排序的。如果要按降序排序，可以传入 <samp class="SANS_TheSansMonoCd_W5Regular_11">reverse=True</samp>，如下所示：

<samp class="SANS_TheSansMonoCd_W5Regular_11">lst = [2, 1, 4, 2]</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">lst.sort(reverse=True)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(lst)</samp>

我们看到结果是按相反顺序排列的：

<samp class="SANS_TheSansMonoCd_W5Regular_11">[4, 2, 2, 1]</samp>

你还可以指定一个 <samp class="SANS_TheSansMonoCd_W5Regular_11">key</samp> 函数，并将其作为参数传递给 <samp class="SANS_TheSansMonoCd_W5Regular_11">sort()</samp> 以自定义排序行为。<samp class="SANS_TheSansMonoCd_W5Regular_11">key</samp> 函数只是将一个列表元素转换成一个可排序的元素。例如，它可以通过使用 Dash 组件的字符串标识符作为关键字，将一个不可排序的对象（如 Dash 组件）转换为可排序类型。通常，这些 <samp class="SANS_TheSansMonoCd_W5Regular_11">key</samp> 函数允许你对自定义对象列表进行排序；例如，按年龄对员工对象列表进行排序。以下示例对列表进行排序，但使用元素的相反值（负值）作为 <samp class="SANS_TheSansMonoCd_W5Regular_11">key</samp>：

<samp class="SANS_TheSansMonoCd_W5Regular_11">lst = [2, 1, 4, 2]</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">lst.sort(key=</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">lambda x:</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">−</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">x)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(lst)</samp>

这给我们带来了：

<samp class="SANS_TheSansMonoCd_W5Regular_11">[4, 2, 2, 1]</samp>

元素 <samp class="SANS_TheSansMonoCd_W5Regular_11">4</samp> 的 <samp class="SANS_TheSansMonoCd_W5Regular_11">key</samp> 是负值 <samp class="SANS_TheSansMonoCd_W5Regular_11">−</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">4</samp>，它是所有列表元素中最小的值。因为列表是按升序排序的，所以这是结果排序列表的第一个值。

<samp class="SANS_Futura_Std_Bold_Condensed_Oblique_I_11">索引列表元素</samp>

你可以使用 <samp class="SANS_TheSansMonoCd_W5Regular_11">list.index(x)</samp> 方法来确定指定列表元素 <samp class="SANS_TheSansMonoCd_W5Regular_11">x</samp> 的索引，像这样：

<samp class="SANS_TheSansMonoCd_W5Regular_11">print([2, 2, 4].index(2))</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print([2, 2, 4].index(2,1))</samp>

`index(x)` 方法查找列表中元素 `<samp class="SANS_TheSansMonoCd_W5Regular_11">x</samp>` 的第一次出现，并返回其索引。

你可以通过传递第二个参数来指定起始索引，从而设置搜索开始的位置。因此，第一行打印出值 2 第一次出现的索引，而第二行打印出值 2 第一次出现的索引，但从索引 1 开始搜索。该方法在两种情况下都会立即找到值 2 并打印：

<samp class="SANS_TheSansMonoCd_W5Regular_11">0</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">1</samp>

<samp class="SANS_Dogma_OT_Bold_B_11">索引基础</samp>

<samp class="SANS_Futura_Std_Book_11">这里是 Python 中索引的快速概览，通过示例说明。假设我们有字符串</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">'universe'</samp><samp class="SANS_Futura_Std_Book_11">，索引就是该字符串中字符的位置，从 0 开始：</samp>

<samp class="SANS_Futura_Std_Book_11">索引             0      1      2      3      4      5      6      7</samp>

<samp class="SANS_Futura_Std_Book_11">字符      u      n      i       v       e      r      s      e</samp>

<samp class="SANS_Futura_Std_Book_11">第一个字符的索引是 0，第二个字符的索引是 1，而第</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">i</samp><samp class="SANS_Futura_Std_Book_11">个字符的索引是</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">i</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">−</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">1</samp><samp class="SANS_Futura_Std_Book_11">。</samp>

### <samp class="SANS_Futura_Std_Bold_B_11">切片</samp>

*切片* 是从给定字符串中截取子字符串的过程。我们称这个子字符串为 *切片*。切片的表示方法如下：

<samp class="SANS_TheSansMonoCd_W5Regular_11">string[start:stop:step]</samp>

`<samp class="SANS_TheSansMonoCd_W5Regular_11">start</samp>` 参数是我们希望开始字符串的位置，它会包含在切片中，而 `<samp class="SANS_TheSansMonoCd_W5Regular_11">stop</samp>` 参数是我们希望字符串停止的位置，它不包括在切片内。忘记 `<samp class="SANS_TheSansMonoCd_W5Regular_11">stop</samp>` 索引不包含在内是一个常见的 bug 来源，所以请记住这一点。`<samp class="SANS_TheSansMonoCd_W5Regular_11">step</samp>` 参数告诉 Python 包含哪些元素，因此 `<samp class="SANS_TheSansMonoCd_W5Regular_11">step</samp>` 为 2 时会包含每隔一个的元素，而 `<samp class="SANS_TheSansMonoCd_W5Regular_11">step</samp>` 为 3 时会包含每隔两个的元素。以下是一个步长为 2 的示例：

<samp class="SANS_TheSansMonoCd_W5Regular_11">s = '----p-y-t-h-o-n----'</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(s[4:15:2])</samp>

这将给我们带来：

<samp class="SANS_TheSansMonoCd_W5Regular_11">python</samp>

三个参数都是可选的，因此你可以跳过它们，使用默认值 <samp class="SANS_TheSansMonoCd_W5Regular_11">start=0</samp>、<samp class="SANS_TheSansMonoCd_W5Regular_11">stop=len(string)</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">step=</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">1</samp>。如果在切片的冒号前省略了 <samp class="SANS_TheSansMonoCd_W5Regular_11">start</samp> 参数，表示切片从第一个位置开始；如果省略了 <samp class="SANS_TheSansMonoCd_W5Regular_11">stop</samp> 参数，表示切片直到字符串的最后一个元素；如果省略了 <samp class="SANS_TheSansMonoCd_W5Regular_11">step</samp> 参数，则默认为步长 1。这里我们省略了 <samp class="SANS_TheSansMonoCd_W5Regular_11">step</samp> 参数：

<samp class="SANS_TheSansMonoCd_W5Regular_11">x = 'universe'</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(x[2:4])</samp>

这给我们带来了：

<samp class="SANS_TheSansMonoCd_W5Regular_11">iv</samp>

在这里，我们指定了起始位置，但没有指定结束位置，并且给出了步长为 2，因此我们会得到每隔一个字符的子字符串，从第三个字符开始，一直到字符串的末尾：

<samp class="SANS_TheSansMonoCd_W5Regular_11">x = 'universe'</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(x[2::2])</samp>

这给我们带来了：

<samp class="SANS_TheSansMonoCd_W5Regular_11">ies</samp>

如果我们不小心给出一个超出最大序列索引的 <samp class="SANS_TheSansMonoCd_W5Regular_11">stop</samp> 索引，Python 会假设我们是想在原始字符串的末尾结束切片。这里是一个例子：

<samp class="SANS_TheSansMonoCd_W5Regular_11">word = "galaxy"</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(word[4:50])</samp>

这将打印：

<samp class="SANS_TheSansMonoCd_W5Regular_11">xy</samp>

只需记住，如果切片超出了序列索引，什么意外的事情都不会发生。

你也可以为所有三个参数提供负整数。对于 <samp class="SANS_TheSansMonoCd_W5Regular_11">start</samp> 或 <samp class="SANS_TheSansMonoCd_W5Regular_11">stop</samp> 的负索引，会告诉 Python 从末尾开始计数。例如，<samp class="SANS_TheSansMonoCd_W5Regular_11">string[</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">–</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">3:]</samp> 会从倒数第三个元素开始切片，<samp class="SANS_TheSansMonoCd_W5Regular_11">string[</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">–</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">10:</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">–</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">5]</samp> 会从倒数第十个元素（包括）开始切片，并在倒数第五个元素（不包括）结束。负步长意味着 Python 会从右向左切片。例如，<samp class="SANS_TheSansMonoCd_W5Regular_11">string[::</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">–</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">1]</samp> 会将字符串反转，<samp class="SANS_TheSansMonoCd_W5Regular_11">string[::</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">–</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">2]</samp> 会从最后一个字符开始，逐个取每隔一个字符向左移动。

### <samp class="SANS_Futura_Std_Bold_B_11">字典</samp>

*字典* 是一个非常有用的数据结构，用于存储键值对。我们用花括号定义一个字典，如下所示：

<samp class="SANS_TheSansMonoCd_W5Regular_11">calories = {'apple': 52, 'banana': 89, 'choco': 546}</samp>

键位于前面，后面跟一个冒号，再后面是值。键值对应该用逗号分隔。这里 <samp class="SANS_TheSansMonoCd_W5Regular_11">'apple'</samp> 是第一个键，<samp class="SANS_TheSansMonoCd_W5Regular_11">52</samp> 是它的值。你可以通过指定字典和在方括号中指定键来访问字典中的单个元素。在以下示例中，我们将苹果的卡路里与一块巧克力的卡路里进行比较：

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(calories['apple'] < calories['choco'])</samp>

当然，它会返回：

<samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp>

字典是一个可变的数据结构，因此可以在创建后进行修改。例如，你可以添加、删除或更新现有的键值对。在这里，我们向字典中添加一个新的键值对，存储一杯卡布奇诺有 74 卡路里的信息：

<samp class="SANS_TheSansMonoCd_W5Regular_11">calories['cappu'] = 74</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(calories['banana'] < calories['cappu'])</samp>

当我们断言卡布奇诺比香蕉含有更多的卡路里时，结果是：

<samp class="SANS_TheSansMonoCd_W5Regular_11">False</samp>

我们使用 <samp class="SANS_TheSansMonoCd_W5Regular_11">keys()</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">values()</samp> 函数来访问字典的所有键和值。在这里，我们检查字符串 <samp class="SANS_TheSansMonoCd_W5Regular_11">'apple'</samp> 是否是字典的键之一，以及整数 <samp class="SANS_TheSansMonoCd_W5Regular_11">52</samp> 是否是字典的值之一。实际上这两者的结果都是 <samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp>：

<samp class="SANS_TheSansMonoCd_W5Regular_11">print('apple' in calories.keys())</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(52 in calories.values())</samp>

要访问字典的所有键值对，我们使用 <samp class="SANS_TheSansMonoCd_W5Regular_11">dictionary.items()</samp> 方法。在下面的 <samp class="SANS_TheSansMonoCd_W5Regular_11">for</samp> 循环中，我们遍历字典 <samp class="SANS_TheSansMonoCd_W5Regular_11">calories</samp> 中的每个 <samp class="SANS_TheSansMonoCd_W5Regular_11">(key, value)</samp> 对，并检查每个值是否大于 500 卡路里。如果是，它会打印出相关的键：

<samp class="SANS_TheSansMonoCd_W5Regular_11">for key, value in calories.items():</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   if value > 500:</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      print(key)</samp>

我们唯一的结果是：

<samp class="SANS_TheSansMonoCd_W5Regular_11">'choco'</samp>

这为我们提供了一种简单的方法，可以遍历字典中的所有键和值，而无需单独访问它们。

### <samp class="SANS_Futura_Std_Bold_B_11">列表推导式</samp>

列表推导式是一种创建列表的紧凑方式，采用简单的一行公式 <samp class="SANS_TheSansMonoCd_W5Regular_11">[expression</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">+</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">context]</samp>。其中的 <samp class="SANS_TheSansMonoCd_W5Regular_11">context</samp> 告诉 Python 要将哪些元素添加到新的列表中，而 <samp class="SANS_TheSansMonoCd_W5Regular_11">expression</samp> 则定义了在添加这些元素之前要对它们进行什么操作。例如，列表推导式语句

<samp class="SANS_TheSansMonoCd_W5Regular_11">[x for x in range(3)]</samp>

创建了一个新的列表 <samp class="SANS_TheSansMonoCd_W5Regular_11">[0, 1, 2]</samp>。这个例子中的 <samp class="SANS_TheSansMonoCd_W5Regular_11">context</samp> 是 <samp class="SANS_TheSansMonoCd_W5Regular_11">for x in range(3)</samp>，因此循环变量 <samp class="SANS_TheSansMonoCd_W5Regular_11">x</samp> 会依次取值 0、1 和 2。这里的表达式 <samp class="SANS_TheSansMonoCd_W5Regular_11">x</samp> 非常简单：它只是将当前的循环变量添加到列表中而没有任何修改。然而，列表推导式能够处理更为复杂的表达式。

列表推导式通常用于仪表盘应用中；例如，它用于动态创建下拉菜单的多个选项。这里我们创建了一个字符串列表——工作日，然后使用列表推导式创建一个字典列表。我们将使用这些字典来创建 Dash 下拉菜单的标签和选项，正如图 1-1 所示：

<samp class="SANS_TheSansMonoCd_W5Regular_11">days = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">options = [{'label': day, 'value': day} for day in days]</samp>

![](img/Figure1-1.png)

<samp class="SANS_Futura_Std_Book_Oblique_I_11">图 1-1：一个 Dash 下拉菜单</samp>

上下文是 <samp class="SANS_TheSansMonoCd_W5Regular_11">for day in days</samp>，所以我们遍历每个工作日 <samp class="SANS_TheSansMonoCd_W5Regular_11">'Mon',</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">…,</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">'Sun'</samp>。该表达式创建了一个字典，包含两个键值对： <samp class="SANS_TheSansMonoCd_W5Regular_11">{'label': day, 'value': day}</samp>。这是创建以下字典列表的一种简洁方式：

<samp class="SANS_TheSansMonoCd_W5Regular_11">[{'label': 'Mon', 'value': 'Mon'}, {'label': 'Tue', 'value': 'Tue'},

{'label': 'Wed', 'value': 'Wed'}, {'label': 'Thu', 'value': 'Thu'},

{'label': 'Fri', 'value': 'Fri'}, {'label': 'Sat', 'value': 'Sat'},

{'label': 'Sun', 'value': 'Sun'}]</samp>

另一种方法是使用常规的 Python <samp class="SANS_TheSansMonoCd_W5Regular_11">for</samp> 循环，如下所示的三行代码：

<samp class="SANS_TheSansMonoCd_W5Regular_11">options = []</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">for day in days:</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   options.append({'label': day, 'value': day})</samp>

你创建了一个字典列表，其中标签和对应的值与每个特定的日期相关联。在这里，下拉菜单将显示标签 <samp class="SANS_TheSansMonoCd_W5Regular_11">'Mon'</samp>，如果用户选择该标签，标签将与值 <samp class="SANS_TheSansMonoCd_W5Regular_11">'Mon'</samp> 关联。

上下文包含任意数量的 <samp class="SANS_TheSansMonoCd_W5Regular_11">for</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">if</samp> 语句。我们可以在列表推导式中使用 <samp class="SANS_TheSansMonoCd_W5Regular_11">if</samp> 语句来过滤结果；例如，我们可以只创建包含工作日的下拉菜单选项：

<samp class="SANS_TheSansMonoCd_W5Regular_11">options = [{'label': day, 'value': day} for day in days if day not in ['Sat', 'Sun']]</samp>

这里我们使用了<samp class="SANS_TheSansMonoCd_W5Regular_11">if</samp>语句来排除结果列表中的<samp class="SANS_TheSansMonoCd_W5Regular_11">Sat</samp>和<samp class="SANS_TheSansMonoCd_W5Regular_11">Sun</samp>。这是在一个<samp class="SANS_TheSansMonoCd_W5Regular_11">for</samp>循环中写这个常规<samp class="SANS_TheSansMonoCd_W5Regular_11">if</samp>语句的一种更快速、更简洁的方式。

### <samp class="SANS_Futura_Std_Bold_B_11">面向对象编程</samp>

在 Python 中，一切都是对象。即使是整数值也是对象。这与 C 语言等编程语言不同，在 C 语言中，整数、浮动值和布尔值是原始数据类型。这样，Python 是基于严格一致的面向对象范式构建的。

<samp class="SANS_Futura_Std_Bold_Condensed_Oblique_I_11">类与对象</samp>

面向对象的 Python 编程的核心是类。类是创建对象的蓝图。类的描述告诉你一个对象的外观和它的功能，分别称为对象的*数据*和*功能*。数据通过*属性*来定义，属性是与特定对象相关联的变量。功能则通过*方法*来定义，方法是与特定对象相关联的函数。

让我们通过哈利·波特的例子来看这些概念如何应用。首先，我们将创建一个具有属性但没有方法的类。在这里，我们创建了一个<samp class="SANS_TheSansMonoCd_W5Regular_11">Muggle</samp>类，并从中创建了两个<samp class="SANS_TheSansMonoCd_W5Regular_11">Muggle</samp>对象：

<samp class="SANS_TheSansMonoCd_W5Regular_11">class Muggle:</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   def __init__(self, age, name, liking_person):</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      self.age =</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">age</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      self.name = name</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      self.likes = liking_person</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">Vernon = Muggle(52, "Vernon", None)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">Petunia = Muggle(49, "Petunia", Vernon)</samp>

我们使用关键字<samp class="SANS_TheSansMonoCd_W5Regular_11">class</samp>为<samp class="SANS_TheSansMonoCd_W5Regular_11">Muggle</samp>对象创建一个新的蓝图。这决定了每个<samp class="SANS_TheSansMonoCd_W5Regular_11">Muggle</samp>对象将具备的数据和功能。在这里，我们声明每个<samp class="SANS_TheSansMonoCd_W5Regular_11">Muggle</samp>对象应该有一个年龄、一个名字和一个他们喜欢的人。

对于每个类，您必须使用方法<code>__init__()</code>来初始化包含数据的类。 每个<code>Muggle</code>对象都将具有属性<code>age</code>、<code>name</code>和<code>likes</code>。 通过将它们作为参数传递给<code>def</code>语句，我们使它们在创建对象时成为必需的参数。 任何类方法的第一个值都是对对象本身的引用，表示为<code>self</code>。 一旦您在代码中调用初始化方法，Python 就会创建一个空对象，您可以使用名称<code>self</code>访问它。

<samp class="SANS_Dogma_OT_Bold_B_15">注意</samp>

*尽管在定义方法时第一个参数是 self，但在调用方法时不需要指定此参数。 Python 会在内部为您执行这些操作。*

创建一个来自该类的对象时，初始化方法<code>__init__</code>会自动首先调用，通过使用类名作为函数调用来实例化一个新对象。 调用<code>Muggle(52, "Vernon", None)</code>和<code>Muggle(49, "Petunia", Vernon)</code>创建两个新的类对象，均定义了三个属性，如下所示：

<code>Muggle</code>

<code>   age = 52</code>

<code>   name = "Vernon"</code>

<code>   likes = None</code>

<code>Muggle</code>

<code>   age = 49</code>

<code>   name = "Petunia"</code>

<code>   likes = "Vernon"</code>

您可以看到这些对象遵循相同的蓝图，但是是<code>Muggle</code>的不同实例；它们具有相同的属性但不同的 DNA。

从现在起，这些对象将存储在您计算机的内存中，直到 Python 在程序终止时将它们清除。

目前故事中有令人伤感的元素吗？ 佩图尼亚喜欢弗农，但弗农谁也不喜欢。 让我们稍微振作一点，好吗？ 我们将为<code>Vernon</code>的<code>likes</code>属性更改为<code>Petunia</code>。 我们可以使用对象的名称、点符号和属性的名称来访问对象的不同属性，如下所示：

<samp class="SANS_TheSansMonoCd_W5Regular_11">Vernon.likes = "Petunia"</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(Vernon.likes)</samp>

这将输出：

<samp class="SANS_TheSansMonoCd_W5Regular_11">Petunia</samp>

让我们定义 <samp class="SANS_TheSansMonoCd_W5Regular_11">Wizard</samp> 类，这样我们就可以在我们的小世界里创建一些巫师。这次我们将增加一些功能：

<samp class="SANS_TheSansMonoCd_W5Regular_11">class Wizard:</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   def __init__(self, age, name):</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      self.age = age</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      self.name = name</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      self.mana = 100</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   def love_me(self, victim):</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      if self.mana >= 100:</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">         victim.likes = self.name</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">         self.mana =</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">self.mana</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">–</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">100</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">Wiz = Wizard(42, "Tom")</samp>

每个 <samp class="SANS_TheSansMonoCd_W5Regular_11">Wizard</samp> 对象有三个属性：<samp class="SANS_TheSansMonoCd_W5Regular_11">age</samp>、<samp class="SANS_TheSansMonoCd_W5Regular_11">name</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">mana</samp> 等级（表示巫师剩余的魔法力量）。<samp class="SANS_TheSansMonoCd_W5Regular_11">age</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">name</samp> 属性是在创建 <samp class="SANS_TheSansMonoCd_W5Regular_11">Wizard</samp> 对象时，根据传入的参数值进行设置的。<samp class="SANS_TheSansMonoCd_W5Regular_11">mana</samp> 属性则在 <samp class="SANS_TheSansMonoCd_W5Regular_11">__init__</samp> 方法中被硬编码为 <samp class="SANS_TheSansMonoCd_W5Regular_11">100</samp>。例如，调用 <samp class="SANS_TheSansMonoCd_W5Regular_11">Wizard(42, "Tom")</samp> 将会把 <samp class="SANS_TheSansMonoCd_W5Regular_11">self.age</samp> 设置为 <samp class="SANS_TheSansMonoCd_W5Regular_11">42</samp>，<samp class="SANS_TheSansMonoCd_W5Regular_11">self.name</samp> 设置为 <samp class="SANS_TheSansMonoCd_W5Regular_11">"Tom"</samp>，并将 <samp class="SANS_TheSansMonoCd_W5Regular_11">self.mana</samp> 设置为 <samp class="SANS_TheSansMonoCd_W5Regular_11">100</samp>。

我们还添加了一个方法 <samp class="SANS_TheSansMonoCd_W5Regular_11">love_me()</samp>，它会对目标施下爱的魔法。如果法师的法力足够，他们可以通过将目标的 <samp class="SANS_TheSansMonoCd_W5Regular_11">likes</samp> 属性设置为施法者的名字来强迫目标爱上他们。然而，这只有在法师的法力值大于或等于 100（<samp class="SANS_TheSansMonoCd_W5Regular_11">self.mana</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">>=</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">100</samp>）时才有效。成功时，目标的 <samp class="SANS_TheSansMonoCd_W5Regular_11">likes</samp> 属性指向施法者的名字，施法者的法力值减少 100。

我们创建了一个 42 岁的法师名叫 Tom。Tom 孤独且希望被喜欢。让我们让 Petunia 和 Vernon 爱上他。我们使用点符号访问对象的方法，并传入 <samp class="SANS_TheSansMonoCd_W5Regular_11">Petunia</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">Vernon</samp> 对象：

<samp class="SANS_TheSansMonoCd_W5Regular_11">Wiz.love_me(Petunia)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">Wiz.love_me(Vernon)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(Petunia.likes=="Tom" and Vernon.likes=="Tom")</samp>

你能判断 Tom 是否成功让 Petunia 和 Vernon 都爱上他吗？

面向对象编程中最常见的困惑之一是忘记在定义方法时包含 <samp class="SANS_TheSansMonoCd_W5Regular_11">self</samp> 参数。另一个常见的困惑是初始化方法的定义使用了语法 <samp class="SANS_TheSansMonoCd_W5Regular_11">__init__()</samp>，而你应该使用语法 <samp class="SANS_TheSansMonoCd_W5Regular_11">ClassName()</samp> 来调用类创建方法，而不是像你预期的那样调用 <samp class="SANS_TheSansMonoCd_W5Regular_11">ClassName.__init__()</samp>。这在代码中有所展示，我们并没有调用 <samp class="SANS_TheSansMonoCd_W5Regular_11">Wizard.__init__(20, 'Ron')</samp>，而是直接调用 <samp class="SANS_TheSansMonoCd_W5Regular_11">Wizard(20, 'Ron')</samp> 来创建一个新的 <samp class="SANS_TheSansMonoCd_W5Regular_11">Wizard</samp> 对象。

这只是 Python 中面向对象编程的简要概述，但确保你完全理解如何在 Python 中构建类和对象是非常值得的。

如需更多信息，可以查看面向对象编程的备忘单，链接为 [*https://<wbr>blog<wbr>.finxter<wbr>.com<wbr>/object<wbr>-oriented<wbr>-programming<wbr>-terminology<wbr>-cheat<wbr>-sheet*](https://blog.finxter.com/object-oriented-programming-terminology-cheat-sheet)。

<samp class="SANS_Futura_Std_Bold_Condensed_Oblique_I_11">术语</samp>

在这里，我们将快速浏览面向对象 Python 中的一些关键定义。

**类**      是创建对象的蓝图。类定义了对象的属性（数据）和功能（方法）。您可以通过点符号访问属性和方法。

**对象**      是根据类定义构建的一个封装数据及其相关功能的单元。对象也被称为类的*实例*。通常，一个对象用于模拟现实世界中的某个事物。例如，我们可以根据类定义<Person>创建对象<Obama>。一个对象由任意数量的属性和方法组成，这些属性和方法被封装在一个单一的单元中。

**实例化**      是创建类的对象的过程。

**方法**      是与特定对象相关联的函数。我们在类定义中使用关键字 def 来定义方法。一个对象可以有任意数量的方法。

**属性**      是用于存储与类或实例相关数据的变量。

**类属性**是静态创建的变量，在类定义中创建，并且被从该类创建的所有对象共享。这些也被称为*类变量*、*静态变量*和*静态属性*。

**动态属性**      是在程序执行过程中动态定义的对象属性，并未在任何方法中定义。例如，您可以通过调用<samp class="SANS_TheSansMonoCd_W5Regular_11">o.my_attribute</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">=</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">42</samp>来将新的属性<my_attribute>添加到任何对象<samp class="SANS_TheSansMonoCd_W5Regular_11">o</samp>中。

**实例属性**是仅属于一个对象的变量。其他对象不能共享该变量，正如它们共享类属性一样。在大多数情况下，您在使用<self>变量名创建实例时，会创建一个实例属性<samp class="SANS_TheSansMonoCd_W5Regular_11">x</samp>，例如 self.x = 42。这些也被称为*实例变量*。

**继承**       是一种编程概念，允许你通过在定义新类时重用部分或全部数据和功能，来创建作为现有类修改的新类。也就是说，类 A 可以从类<samp class="SANS_TheSansMonoCd_W5Regular_11">B</samp>继承属性或方法，这样它就具有与类<samp class="SANS_TheSansMonoCd_W5Regular_11">B</samp>相同的数据和功能，但类<samp class="SANS_TheSansMonoCd_W5Regular_11">A</samp>可以改变行为或添加数据和方法。例如，类<samp class="SANS_TheSansMonoCd_W5Regular_11">Dog</samp>可以从类 Animal 继承属性<samp class="SANS_TheSansMonoCd_W5Regular_11">number_of_legs</samp>。在这种情况下，你将定义继承类<samp class="SANS_TheSansMonoCd_W5Regular_11">Dog</samp>如下：<samp class="SANS_TheSansMonoCd_W5Regular_11">class Dog(Animal):</samp>，然后是类的主体。

如果你已经理解了这些术语，你可以跟上大多数关于面向对象编程的讨论。掌握面向对象是精通 Python 的重要一步。

### <samp class="SANS_Futura_Std_Bold_B_11">装饰器函数和注解</samp>

Dash 在很大程度上依赖于 Python 的*装饰器*或*装饰器函数*的概念，它们在不修改代码本身的情况下，为现有代码添加功能。这在你想要修改或自定义现有函数的输出时非常有用，而无需更改函数的实际代码。例如，你可能无法访问某个函数的定义，但仍然想改变它的行为。装饰器函数正是来拯救你的！

把装饰器函数看作一个包装器。它接受一个原始函数，调用它，并根据程序员的需求修改它的行为。这样，你可以在函数最初定义后动态地改变它的行为。

让我们从一个简单的例子开始。我们定义一个函数，将一些文本打印到标准输出：

<samp class="SANS_TheSansMonoCd_W5Regular_11">def print_text():</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print("Hello world!")</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print_text()</samp>

输出是：

<samp class="SANS_TheSansMonoCd_W5Regular_11">Hello world!</samp>

这个函数始终会打印相同的消息。假设你想装饰这个输出，让它变得更有趣。一种方法是定义一个新的<samp class="SANS_TheSansMonoCd_W5Regular_11">pretty_print()</samp>函数；但这还不是一个装饰器函数，因为它并没有改变另一个函数的行为。然而，它确实展示了如何包装另一个函数并修改它的行为：

<samp class="SANS_TheSansMonoCd_W5Regular_11">def print_text():</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print("Hello world!")</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">def pretty_print():</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   annotate = '+'</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print(annotate * 30)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print_text()</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print(annotate * 30)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">pretty_print()</samp>

现在输出如下所示：

<samp class="SANS_TheSansMonoCd_W5Regular_11">++++++++++++++++++++++++++++++</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">你好，世界！</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">++++++++++++++++++++++++++++++</samp>

外部函数 <samp class="SANS_TheSansMonoCd_W5Regular_11">pretty_print()</samp> 调用内部函数 <samp class="SANS_TheSansMonoCd_W5Regular_11">print_text()</samp>，并通过在内部函数 <samp class="SANS_TheSansMonoCd_W5Regular_11">print_text()</samp> 输出前后添加 30 个加号（<samp class="SANS_TheSansMonoCd_W5Regular_11">+</samp>）符号来修饰结果。本质上，你是 *包装* 内部函数的结果，并通过附加功能进行增强。

装饰器函数允许你像这样通用化代码。例如，你可能希望将一个任意的内部函数传递给 <samp class="SANS_TheSansMonoCd_W5Regular_11">pretty_print()</samp> 函数，这样你就可以将它应用于任何 Python 函数。在这里，我们创建了一个装饰器函数，但请注意，为了展示它是如何工作的，我们采用了比较长的方法。稍后我们将查看 Python 提供的更简短的方法来实现相同的功能。这是长版本：

<samp class="SANS_TheSansMonoCd_W5Regular_11">def pretty_print_decorator(f):</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   annotate =</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">'+'</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   def pretty_print():</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      print(annotate * 50)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      f()</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      print(annotate * 50)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   return pretty_print</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">def print_text():</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print("你好，世界！")</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">def print_text_2():</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print("你好，宇宙！")</samp>

当我们像这样使用时：

<samp class="SANS_TheSansMonoCd_W5Regular_11">pretty_print_decorator(print_text)()</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">pretty_print_decorator(print_text_2)()</samp>

我们将得到如下输出：

<samp class="SANS_TheSansMonoCd_W5Regular_11">++++++++++++++++++++++++++++++++++++++++++++++++++</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">你好，世界！</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">++++++++++++++++++++++++++++++++++++++++++++++++++</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">++++++++++++++++++++++++++++++++++++++++++++++++++</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">Hello universe!</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">++++++++++++++++++++++++++++++++++++++++++++++++++</samp>

这里，装饰器函数接受一个函数作为输入，并返回另一个通过将输出包裹在 <samp class="SANS_TheSansMonoCd_W5Regular_11">+</samp> 符号中来修改行为的函数。你可以传递任何打印输出的函数，并创建一个类似的函数，额外地将输出包裹在一系列的 <samp class="SANS_TheSansMonoCd_W5Regular_11">+</samp> 符号中。

这个简单的装饰器函数接受一个函数对象，并应用一些输出修改，但装饰器函数可以做各种复杂的事情，比如分析输出、应用额外的逻辑，或者过滤掉一些不需要的信息。

这是一种不切实际的复杂方式来构建装饰器函数。由于这种模式非常常见，Python 提供了一种便捷的方法，通过更少的代码实现相同的功能：你只需在要装饰的函数前添加一行代码。这一行包括一个 "@" 符号，后面跟着你之前定义的装饰器函数的名称。这里我们定义了 <samp class="SANS_TheSansMonoCd_W5Regular_11">pretty_print_decorator(f)</samp> 函数，然后在定义两个打印函数时应用它：

<samp class="SANS_TheSansMonoCd_W5Regular_11">def pretty_print_decorator(f):</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   annotate = '+'</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   def pretty_print():</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      print(annotate * 50)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      f()</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      print(annotate * 50)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   return pretty_print</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">@pretty_print_decorator</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">def print_text():</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print("Hello world!")</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">@pretty_print_decorator</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">def print_text_2():</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print("Hello universe!")</samp>

我们像这样调用我们定义的两个函数：

<samp class="SANS_TheSansMonoCd_W5Regular_11">print_text()</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">print_text_2()</samp>

然后我们应该得到这样的输出：

<samp class="SANS_TheSansMonoCd_W5Regular_11">++++++++++++++++++++++++++++++++++++++++++++++++++</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">Hello world!</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">++++++++++++++++++++++++++++++++++++++++++++++++++</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">++++++++++++++++++++++++++++++++++++++++++++++++++</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">Hello universe!</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">++++++++++++++++++++++++++++++++++++++++++++++++++</samp>

你可以看到输出与之前完全相同。但这次，我们不是显式地调用装饰器函数<samp class="SANS_TheSansMonoCd_W5Regular_11">pretty_print_factory</samp>，比如在<samp class="SANS_TheSansMonoCd_W5Regular_11">pretty_print_decorator(print_text)</samp>中装饰现有函数<samp class="SANS_TheSansMonoCd_W5Regular_11">print_text</samp>，而是直接使用带有<samp class="SANS_TheSansMonoCd_W5Regular_11">@</samp>前缀的装饰器函数修改<samp class="SANS_TheSansMonoCd_W5Regular_11">print_text()</samp>的行为。每次调用被装饰的函数时，它会自动通过装饰器函数。这种方式使我们能够堆叠任意复杂的函数层级，每个层级通过装饰另一个函数的输出，增加新的复杂性。

装饰器函数是 Dash 框架的核心。Dash 提供了高级功能，你可以通过将 Dash 已经定义的装饰器函数应用到你的任何函数上，利用<samp class="SANS_TheSansMonoCd_W5Regular_11">@</samp>注解来访问这些功能。Dash 将这些装饰器函数称为*回调装饰器*。你将在本书讨论的仪表盘应用中看到许多这样的例子。

### <samp class="SANS_Futura_Std_Bold_B_11">总结</samp>

这是对一些与使用 Dash 创建应用最相关的 Python 概念的简要概述。如果你觉得这些内容难以理解，建议在开始构建应用之前查看“Python 基础”附录。

但在我们开始创建仪表盘应用之前，让我们先深入了解我们推荐你用于本书的 PyCharm 框架。如果你已经是 PyCharm 专家，或者你有其他偏好的编程环境，可以直接跳到第三章。
