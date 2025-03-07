## **9**

**抽象语法树、Hy 和类 Lisp 的特性**

![image](../images/common01.jpg)

*抽象语法树（AST）* 是对任何编程语言源代码结构的表示。每种语言，包括 Python，都有其特定的 AST；Python 的 AST 通过解析 Python 源文件构建。像任何树一样，它由相互连接的节点组成。一个节点可以表示一个操作、语句、表达式，甚至是一个模块。每个节点可以包含对其他构成树的节点的引用。

Python 的 AST 文档不多，因此一开始处理起来可能比较困难，但理解 Python 构建的深层次方面可以帮助你掌握它的使用。

本章将检查一些简单 Python 命令的 AST，帮助你熟悉其结构和使用方式。一旦你熟悉了 AST，我们将构建一个程序，利用 flake8 和 AST 检查错误声明的方法。最后，我们将介绍 Hy，这是一种基于 Python AST 构建的 Python-Lisp 混合语言。

### **查看 AST**

查看 Python AST 最简单的方法是解析一些 Python 代码并输出生成的 AST。为此，Python 的 ast 模块提供了所需的一切，正如 [清单 9-1](ch09.xhtml#ch9list1) 中所示。

>>> import ast

>>> ast.parse

<function parse at 0x7f062731d950>

>>> ast.parse("x = 42")

<_ast.Module object at 0x7f0628a5ad10>

>>> ast.dump(ast.parse("x = 42"))

"Module(body=[Assign(targets=[Name(id='x', ctx=Store())], value=Num(n=42))])"

*清单 9-1：使用 ast 模块输出解析代码生成的 AST*

ast.parse() 函数解析任何包含 Python 代码的字符串，并返回一个 _ast.Module 对象。该对象实际上是树的根节点：你可以浏览它，发现构成树的每个节点。为了可视化树的结构，你可以使用 ast.dump() 函数，它将返回整个树的字符串表示。

在 [清单 9-1](ch09.xhtml#ch9list1) 中，代码 x = 42 被 ast.parse() 解析，结果通过 ast.dump() 输出。这个抽象语法树可以按 [图 9-1](ch09.xhtml#ch9fig1) 所示渲染，展示了 Python 赋值命令的结构。

![image](../images/f09-01.jpg)

*图 9-1：Python 中赋值命令的 AST*

AST 总是从根元素开始，通常是一个 _ast.Module 对象。这个模块对象包含一个语句或表达式的列表，在它的 *body* 属性中进行求值，通常表示一个文件的内容。

正如你可能猜到的，图[9-1](ch09.xhtml#ch9fig1)中展示的ast.Assign对象表示一个*赋值*，它映射到Python语法中的=符号。ast.Assign对象有一个*targets*列表和一个*value*，用来给targets赋值。在此案例中，targets列表由一个ast.Name对象组成，它代表一个ID为*x*的变量。value是一个值为42的数字*n*。ctx属性存储一个*上下文*，它可以是ast.Store或ast.Load，取决于变量是用于读取还是写入。在此例中，变量正在被赋值，因此使用了ast.Store上下文。

我们可以将这个AST传递给Python，利用内置的compile()函数进行编译和求值。此函数接受一个AST作为参数、源文件名和模式（可以是'exec'、'eval'或'single'）。源文件名可以是任何你希望AST来自的名称；如果数据不是来自存储文件，通常会使用字符串<input>作为源文件名，正如在[清单 9-2](ch09.xhtml#ch9list2)中所示。

>>> compile(ast.parse("x = 42"), '<input>', 'exec')

<code object <module> at 0x111b3b0, file "<input>", line 1>

>>> eval(compile(ast.parse("x = 42"), '<input>', 'exec'))

>>> x

42

*清单 9-2：使用compile()函数编译非存储文件中的数据*

这些模式分别代表执行（exec）、求值（eval）和单一语句（single）。模式应与提供给ast.parse()的内容匹配，默认模式为exec。

+   exec模式是Python的常规模式，当_ast.Module是树的根节点时使用。

+   eval模式是一种特殊模式，期望树结构为单个ast.Expression。

+   最后，single是一种特殊模式，期望一个单一的语句或表达式。如果它得到的是表达式，则会调用sys.displayhook()来显示结果，就像代码在交互式Shell中运行一样。

AST的根节点是ast.Interactive，它的body属性是一个节点列表。

我们可以使用ast模块提供的类手动构建一个AST。显然，这是一种非常冗长的写Python代码的方式，并不是我推荐的做法！然而，这样做很有趣，并且有助于了解AST。让我们来看看使用AST编程是什么样子的。

#### ***使用AST编写程序***

让我们通过手动构建抽象语法树，写一个传统的“Hello World！”程序。

➊ >>> hello_world = ast.Str(s='hello world!', lineno=1, col_offset=1)

➋ >>> print_name = ast.Name(id='print', ctx=ast.Load(), lineno=1, col_offset=1)

➌ >>> print_call = ast.Call(func=print_name, ctx=ast.Load(),

... args=[hello_world], keywords=[], lineno=1, col_offset=1)

➍ >>> module = ast.Module(body=[ast.Expr(print_call, ... lineno=1, col_offset=1)], lineno=1, col_offset=1)

➎ >>> code = compile(module, '', 'exec')

>>> eval(code)

hello world!

*清单 9-3：使用AST编写“Hello World！”*

在 [Listing 9-3](ch09.xhtml#ch9list3) 中，我们一步步构建树，每个叶子节点都是程序的一个元素（无论是值还是指令）。

第一个叶子节点是一个简单的字符串 ➊：`ast.Str` 表示一个字面量字符串，这里包含了 "hello world!" 文本。`print_name` 变量 ➋ 包含一个 `ast.Name` 对象，它引用了一个变量——在这个例子中是指向 `print()` 函数的 `print` 变量。

`print_call` 变量 ➌ 包含一个函数调用。它指向要调用的函数名、传递给函数调用的常规参数以及关键字参数。使用哪些参数取决于调用的函数。在本例中，由于调用的是 `print()` 函数，我们将传递我们制作并存储在 `hello_world` 中的字符串。

最后，我们创建一个 _ast.Module 对象 ➍ 来包含所有这些代码，作为一个包含一个表达式的列表。我们可以使用 `compile()` 函数 ➎ 编译 _ast.Module 对象，这会解析树并生成本地代码对象。这些代码对象是已编译的 Python 代码，最终可以通过 Python 虚拟机使用 `eval` 执行！

这个过程正是你在运行 Python *.py* 文件时发生的事情：一旦文本标记被解析，它们就会被转换为一个 AST 对象的树，进行编译并求值。

**注意**

*参数 `lineno` 和 `col_offset` 分别表示生成 AST 的源代码的行号和列偏移量。在这个上下文中设置这些值没有太大意义，因为我们并没有解析源文件，但能够找到生成 AST 的代码位置是有用的。例如，Python 在生成回溯信息时会使用这些信息。事实上，Python 拒绝编译没有提供这些信息的 AST 对象，因此我们将虚假的值传递给这些参数。你也可以使用 `ast.fix_missing_locations()` 函数来将缺失的值设置为父节点上设置的值。*

#### ***AST 对象***

你可以通过阅读 _ast 模块文档（注意下划线）来查看 AST 中所有可用对象的完整列表。

这些对象分为两个主要类别：语句和表达式。*语句* 包括如 assert、赋值（=）、增强赋值（+=、/= 等）、global、def、if、return、for、class、pass、import、raise 等类型。语句继承自 `ast.stmt`；它们影响程序的控制流，并且通常由表达式组成。

*表达式* 包括 lambda、数字、yield、名称（变量）、比较和调用等类型。表达式继承自 `ast.expr`；它们与语句的不同之处在于，它们通常会生成一个值并且不会影响程序流程。

还有一些较小的类别，例如 `ast.operator` 类，它定义了标准运算符，如 *add*（+）、*div*（/）和 *right shift*（>>），以及 `ast.cmpop` 模块，它定义了比较运算符。

这里的简单示例应该能让你了解如何从零开始构建AST。接下来你可以想象，如何利用这个AST构建一个解析字符串并生成代码的编译器，从而实现你自己的Python语法！这正是促使Hy项目开发的原因，我们将在本章后续部分讨论。

#### ***遍历AST***

为了跟踪树的构建过程或访问特定节点，有时你需要遍历树，浏览并迭代节点。你可以使用ast.walk()函数来做到这一点。另外，ast模块还提供了NodeTransformer类，你可以继承这个类来遍历AST并修改特定节点。使用NodeTransformer使得动态更改代码变得容易，如[列表9-4](ch09.xhtml#ch9list4)所示。

import ast

class ReplaceBinOp(ast.NodeTransformer):

"""在二元操作中将操作替换为加法"""

def visit_BinOp(self, node):

return ast.BinOp(left=node.left,

op=ast.Add(),

right=node.right)

➊ tree = ast.parse("x = 1/3")

ast.fix_missing_locations(tree)

eval(compile(tree, '', 'exec'))

print(ast.dump(tree))

➋ print(x)

➌ tree = ReplaceBinOp().visit(tree)

ast.fix_missing_locations(tree)

print(ast.dump(tree))

eval(compile(tree, '', 'exec'))

➍ print(x)

*列表9-4：通过NodeTransformer遍历树以修改节点*

第一个树对象 ➊ 构建的是一个表示表达式x = 1/3的AST。一旦这个AST被编译并执行，打印x的结果 ➋ 将是0.33333，这是1/3的预期结果。

第二个树对象 ➌ 是ReplaceBinOp的实例，它继承自ast.NodeTransformer。它实现了自己版本的ast.NodeTransformer.visit()方法，并将任何ast.BinOp操作更改为执行ast.Add的ast.BinOp。具体而言，这将任何二元运算符（如 +、-、/ 等）更改为 + 运算符。当第二个树被编译并执行 ➍ 后，结果变为4，这是1 + 3的结果，因为第一个对象中的 / 被替换成了 +。

你可以在这里看到程序的执行：

Module(body=[Assign(targets=[Name(id='x', ctx=Store())],

value=BinOp(left=Num(n=1), op=Div(), right=Num(n=3)))])

0.3333333333333333

Module(body=[Assign(targets=[Name(id='x', ctx=Store())],

value=BinOp(left=Num(n=1), op=Add(), right=Num(n=3)))])

4

**注意**

*如果你需要评估一个返回简单数据类型的字符串，可以使用ast.literal_eval。作为eval的更安全替代方法，它防止输入字符串执行任何代码。*

### **扩展flake8与AST检查**

在[第7章](ch07.xhtml#ch07)中，你学习了那些不依赖于对象状态的方法应该使用@staticmethod装饰器声明为静态方法。问题是，许多开发者忘记这么做。我个人花了太多时间检查代码并要求别人修复这个问题。

我们已经看到了如何使用 flake8 对代码进行一些自动检查。事实上，flake8 是可扩展的，能够提供更多的检查功能。我们将编写一个 flake8 扩展，通过分析 AST 来检查静态方法声明是否缺失。

[示例 9-5](ch09.xhtml#ch9list5)展示了一个省略静态声明的类和一个正确包含静态声明的类的例子。编写这个程序并将其保存为 *ast_ext.py*；我们稍后将用它来编写扩展。

class Bad(object):

# 未使用 self，该方法不需要

# 要绑定，应该声明为静态

def foo(self, a, b, c):

return a + b - c

class OK(object):

# 这是正确的

@staticmethod

def foo(a, b, c):

return a + b - c

*示例 9-5：省略和包括 @staticmethod*

尽管 Bad.foo 方法可以正常工作，但严格来说，更正确的写法是 OK.foo（关于为什么可以参考[第7章](ch07.xhtml#ch07)的更多细节）。要检查一个 Python 文件中所有方法是否正确声明，我们需要执行以下操作：

+   遍历 AST 的所有语句节点。

+   检查语句是否为类定义（ast.ClassDef）。

+   遍历该类语句中的所有函数定义（ast.FunctionDef），检查是否已声明 @staticmethod。

+   如果方法没有声明为静态，检查方法中是否使用了第一个参数（self）。如果没有使用self，则可以标记该方法为可能写错。

我们项目的名称将是 ast_ext。为了在 flake8 中注册一个新的插件，我们需要创建一个带有常规 *setup.py* 和 *setup.cfg* 文件的打包项目。然后，我们只需要在 ast_ext 项目的 *setup.cfg* 中添加一个入口点。

[entry_points]

flake8.extension =

--snip--

H904 = ast_ext:StaticmethodChecker

H905 = ast_ext:StaticmethodChecker

*示例 9-6：为我们章节允许 flake8 插件*

在 [示例 9-6](ch09.xhtml#ch9list6) 中，我们还注册了两个 flake8 错误代码。如你所见，稍后我们将实际在代码中增加额外的检查！

下一步是编写插件。

#### ***编写类***

由于我们正在编写一个 flake8 的 AST 检查，插件需要是一个遵循特定签名的类，如 [示例 9-7](ch09.xhtml#ch9list7) 所示。

class StaticmethodChecker(object):

def __init__(self, tree, filename):

self.tree = tree

def run(self):

pass

*示例 9-7：检查 AST 的类*

默认模板很容易理解：它将树本地存储以供 run() 方法使用，该方法将 *yield* 发现的问题。将要返回的值必须遵循预期的 PEP 8 签名：一个形式为 (lineno, col_offset, error_string, code) 的元组。

#### ***忽略无关代码***

如前所述，ast 模块提供了 walk() 函数，允许你轻松遍历树。我们将利用它来遍历 AST，找出需要检查的内容和不需要检查的内容。

首先，让我们编写一个循环，忽略非类定义的语句。将其添加到您的 ast_ext 项目中，如[清单 9-8](ch09.xhtml#ch9list8)所示；不需要更改的代码已灰显。

class StaticmethodChecker(object):

def __init__(self, tree, filename):

self.tree = tree

def run(self):

for stmt in ast.walk(self.tree):

# 忽略非类的语句

if not isinstance(stmt, ast.ClassDef):

continue

*清单 9-8：忽略非类定义的语句*

[清单 9-8](ch09.xhtml#ch9list8) 中的代码仍然没有进行任何检查，但现在它知道如何忽略非类定义的语句。下一步是让我们的检查器忽略所有非函数定义的语句。

for stmt in ast.walk(self.tree):

# 忽略非类的语句

if not isinstance(stmt, ast.ClassDef):

continue

# 如果是类，则遍历其成员体以查找方法

for body_item in stmt.body:

# 不是方法，跳过

if not isinstance(body_item, ast.FunctionDef):

continue

*清单 9-9：忽略非函数定义的语句*

在[清单 9-9](ch09.xhtml#ch9list9)中，我们通过遍历类定义的属性来忽略不相关的语句。

#### ***检查正确的装饰器***

我们已准备好编写检查方法，它存储在 body_item 属性中。首先，我们需要检查被检查的方法是否已声明为静态。如果是，我们不需要做任何进一步的检查，可以直接退出。

for stmt in ast.walk(self.tree):

# 忽略非类的语句

if not isinstance(stmt, ast.ClassDef):

continue

# 如果是类，则遍历其成员体以查找方法

for body_item in stmt.body:

# 不是方法，跳过

if not isinstance(body_item, ast.FunctionDef):

continue

# 检查是否有装饰器

for decorator in body_item.decorator_list:

if (isinstance(decorator, ast.Name)

and decorator.id == 'staticmethod'):

# 它是一个静态函数，没问题

break else:

# 函数不是静态的，我们暂时不做任何处理

Pass

*清单 9-10：检查静态装饰器*

请注意，在[清单 9-10](ch09.xhtml#ch9list10)中，我们使用了 Python 的特殊 for/else 结构，其中除非使用 break 退出 for 循环，否则 else 部分会被执行。在这一点上，我们已经能够检测方法是否声明为静态。

#### ***查找 self***

下一步是检查未声明为静态的方法是否使用了 self 参数。首先，检查方法是否包含任何参数，如[清单 9-11](ch09.xhtml#ch9list11)所示。

--snip--

# 检查是否有装饰器

for decorator in body_item.decorator_list:

if (isinstance(decorator, ast.Name)

and decorator.id == 'staticmethod'):

# 它是一个静态函数，没问题

break

else:

try:

first_arg = body_item.args.args[0]

except IndexError:

yield (

body_item.lineno,

body_item.col_offset,

"H905: 方法缺少第一个参数",

"H905",

)

# 检查下一个方法

Continue

*清单 9-11：检查方法的参数*

我们终于添加了一个检查！[Listing 9-11](ch09.xhtml#ch9list11)中的这个 try 语句抓取方法签名中的第一个参数。如果代码未能从签名中获取第一个参数，因为第一个参数不存在，我们已经知道有问题：没有 self 参数就不能有绑定方法。如果插件检测到这种情况，它会引发我们之前设置的 H905 错误码，表示方法缺少第一个参数。

**注意**

*PEP 8 错误码遵循特定的格式（字母后跟数字），但没有规定应该选择哪个错误码。你可以为这个错误选择任何其他的错误码，只要它没有被 PEP 8 或其他扩展使用。*

现在你知道我们为什么在*setup.cfg*中注册了两个错误码：我们有一个很好的机会，一石二鸟。

下一步是检查方法的代码中是否使用了 self 参数。

--snip--

try:

first_arg = body_item.args.args[0]

except IndexError:

yield (

body_item.lineno,

body_item.col_offset,

"H905: 方法缺少第一个参数",

"H905",

)

# 检查下一个方法

continue

对于每个 body_item 中的 func_stmt，使用 ast.walk 遍历：

# 检查方法在 Python 2 和 Python 3 中的差异

如果是 six.PY3:

如果 (isinstance(func_stmt, ast.Name)

并且 first_arg.arg == func_stmt.id):

# 第一个参数已使用，可以

break

否则：

如果 (func_stmt != first_arg

并且 isinstance(func_stmt, ast.Name)

和 func_stmt.id == first_arg.id):

# 第一个参数已使用，可以

break

否则：

yield (

body_item.lineno,

body_item.col_offset,

"H904: 方法应该声明为静态方法",

"H904",

)

*Listing 9-12：检查方法是否使用 self 参数*

为了检查方法体中是否使用了 self 参数，[Listing 9-12](ch09.xhtml#ch9list12)中的插件递归迭代，使用 ast.walk 遍历方法体，查找名为 self 的变量。如果没有找到该变量，程序最终会返回 H904 错误码。否则，什么也不做，代码被认为是正常的。

**注意**

*正如你可能已经注意到的，代码多次遍历模块的 AST 定义。可能有一定的优化空间，通过一次遍历来浏览 AST，但考虑到工具的实际使用情况，我不确定是否值得这样做。我把这个练习留给你，亲爱的读者。*

了解 Python AST 对使用 Python 并非严格必要，但它确实提供了对语言构建和工作原理的强大洞察力。它让你更好地理解你写的代码在背后是如何被使用的。

### **Hy 快速介绍**

现在你已经很好地理解了 Python AST 的工作原理，你可以开始梦想为 Python 创建一种新的语法。你可以解析这种新语法，构建出 AST，并将其编译为 Python 代码。

这正是 Hy 所做的。*Hy* 是一种 Lisp 方言，它解析类似 Lisp 的语言并将其转换为常规的 Python AST，使其与 Python 生态系统完全兼容。你可以将其与 Clojure 对 Java 的作用做类比。Hy 本身就能填满一本书，因此我们这里只做简单的概览。Hy 使用了 Lisp 家族语言的语法和一些特性：它是面向函数的，提供宏功能，并且易于扩展。

如果你还不熟悉 Lisp——你应该熟悉——Hy 的语法看起来会很熟悉。一旦你安装了 Hy（运行 pip install hy），启动 hy 解释器会给你一个标准的 REPL 提示符，你可以从中开始与解释器进行交互，如 [列表 9-13](ch09.xhtml#ch9list13) 所示。

% hy

hy 0.9.10

=> (+ 1 2)

3

*列表 9-13：与 Hy 解释器的交互*

对于不熟悉 Lisp 语法的人来说，括号用于构建列表。如果列表没有加引号，它会被求值：第一个元素必须是一个函数，后续元素作为参数传入。这里的代码 (+ 1 2) 等同于 Python 中的 1 + 2。

在 Hy 中，大多数构造，如函数定义，都直接映射自 Python。

=> (defn hello [name]

...  (print "你好，世界！")

...  (print (% "很高兴认识你 %s" name)))

=> (hello "jd")

你好，世界！

很高兴认识你 jd

*列表 9-14：从 Python 映射一个函数定义*

如 [列表 9-14](ch09.xhtml#ch9list14) 所示，Hy 在内部解析提供的代码，将其转换为 Python AST，编译并求值。幸运的是，Lisp 是一种容易解析的树结构：每一对括号表示树的一个节点，这意味着转换实际上比原生 Python 语法更容易！

类定义通过 defclass 构造来支持，这受到 Common Lisp 对象系统（CLOS）的启发。

(defclass A [object]

[[x 42] [y (fn [self value]

(+ self.x value))]])

*列表 9-15：使用 defclass 定义一个类*

[列表 9-15](ch09.xhtml#ch9list15) 定义了一个名为 A 的类，它继承自 object，具有一个名为 x 的类属性，值为 42；然后一个方法 y 返回 x 属性加上传入的值。

真正神奇的是，你可以直接将*任何 Python 库*导入到 Hy 中，并且无需任何额外负担。使用 import() 函数导入模块，如 [列表 9-16](ch09.xhtml#ch9list16) 所示，就像你在常规 Python 中那样使用。

=> (import uuid)

=> (uuid.uuid4)

UUID('f823a749-a65a-4a62-b853-2687c69d0e1e')

=> (str (uuid.uuid4))

'4efa60f2-23a4-4fc1-8134-00f5c271f809'

*列表 9-16：导入常规 Python 模块*

Hy 还具有更高级的构造和宏功能。在 [列表 9-17](ch09.xhtml#ch9list17) 中，看看 cond() 函数能为你做什么，而不是经典但冗长的 if/elif/else。

(cond

[(> somevar 50)

(print "那个变量太大了！")]

[(< somevar 10)

(print "那个变量太小了！")]

[true

(print "那个变量正好！")]

*示例 9-17：使用cond代替if/elif/else*

cond宏具有以下签名：(cond [condition_expression return_expression] ...)。每个条件表达式都会被依次评估：一旦某个条件表达式返回真值，就会评估并返回相应的返回表达式。如果没有提供返回表达式，则返回条件表达式的值。因此，cond等价于if/elif构造，只是它能够返回条件表达式的值，而无需对其进行两次评估或将其存储在临时变量中！

Hy让你可以在不远离舒适区的情况下进入Lisp世界，因为你依然在编写Python代码。hy2py工具甚至可以显示你的Hy代码转换成Python后会是什么样子。虽然Hy并不广泛使用，但它是展示Python语言潜力的一个很好的工具。如果你有兴趣了解更多，我建议你查看在线文档并加入社区。

### **总结**

就像任何其他编程语言一样，Python源代码可以通过抽象语法树表示。你很少会直接使用AST，但当你理解它的工作原理时，它可以提供一种有用的视角。

### **Paul Tagliamonte谈AST和Hy**

Paul在2013年创建了Hy，作为一个Lisp爱好者，我和他一起加入了这个精彩的冒险。Paul目前是Sunlight Foundation的开发人员。

**你是如何学会正确使用AST的？对于想要学习AST的人，你有什么建议吗？**

AST（抽象语法树）文档非常少，所以大部分知识都来自于经过逆向工程生成的AST。通过编写简单的Python脚本，可以使用类似`import ast; ast.dump(ast.parse("print foo"))`的方式生成一个等效的AST来帮助完成任务。只要有一点猜测和耐心，通过这种方式建立基础理解并非不可行。

在某个时刻，我会承担起记录我对AST模块理解的任务，但我发现编写代码是学习AST的最佳方式。

**Python的AST在不同版本和用法之间有何不同？**

Python的AST不是私有的，但也不是公共接口。从版本到版本，无法保证稳定性——事实上，Python 2和3之间，甚至不同的Python 3版本之间，都存在一些相当烦人的差异。此外，不同的实现可能会以不同的方式解释AST，甚至有可能有独特的AST。没有任何规定Jython、PyPy或CPython必须以相同的方式处理Python AST。

例如，CPython 可以处理稍微乱序的 AST 条目（通过 lineno 和 col_offset），而 PyPy 会抛出一个断言错误。虽然有时会让人恼火，但 AST 一般是合理的。构建一个能在大量 Python 实例上工作的 AST 并非不可能。只要加上一些条件，创建一个能在 CPython 2.6 到 3.3 以及 PyPy 上工作的 AST 只会稍微令人烦恼，这使得这个工具非常方便。

**你在创建 Hy 时的过程是什么？**

我开始开发 Hy 是因为一次关于拥有一个编译成 Python 而不是 Java JVM（Clojure）语言的讨论。几天后，我就有了第一个版本的 Hy。这个版本像 Lisp，甚至在某些方面像一个正常的 Lisp，然而它非常慢。我的意思是，真的非常慢。它比本地 Python 慢了一个数量级，因为 Lisp 的运行时本身是用 Python 实现的。

一度感到沮丧，我差点放弃，但后来有位同事建议使用抽象语法树（AST）来实现运行时，而不是在 Python 中实现运行时。这个建议成为了整个项目的催化剂。我在2012年的整个假期都在疯狂开发 Hy。大约一周后，我有了一个类似当前 Hy 代码库的东西。

在使 Hy 足够工作以实现一个基本的 Flask 应用之后，我在波士顿的 Python 会议上做了一个关于该项目的演讲，反响非常热烈——实际上，热烈到让我开始把 Hy 视为一种很好的方式来教人们了解 Python 的内部机制，比如 REPL 是如何工作的，PEP 302 导入钩子，以及 Python 的 AST。这是代码生成代码概念的一个很好的介绍。

我重写了编译器的一部分，以解决一些过程中的哲学问题，从而引导我们到了当前的代码库版本——这个版本的表现相当稳定！

学习 Hy 也是开始理解如何阅读 Lisp 的一个好方法。用户可以在一个熟悉的环境中习惯 s 表达式，甚至使用他们已经在用的库，从而使得过渡到其他 Lisp（如 Common Lisp、Scheme 或 Clojure）变得更加顺利。

**Hy 与 Python 的互操作性有多强？**

Hy 的互操作性令人惊讶。如此之强，以至于 pdb 可以正确调试 Hy，而你无需做任何修改。我已经用 Hy 编写了 Flask 应用、Django 应用以及各种模块。Python 可以导入 Python，Hy 可以导入 Hy，Hy 可以导入 Python，Python 也可以导入 Hy。这正是 Hy 独特之处；其他类似的 Lisp 变体，如 Clojure，都是单向的。Clojure 可以导入 Java，但 Java 却很难导入 Clojure。

Hy 的工作方式是将 Hy 代码（以 s 表达式形式）几乎直接转换为 Python 的 AST。这一编译步骤意味着生成的字节码相当“理智”，也就是说 Python 很难分辨出该模块根本不是用 Python 编写的。

常见的 Lisp 特性，如 *earmuffs* 或使用连字符（using-dashes），通过将它们转换为 Python 等效形式得到了完全支持（在这种情况下，*earmuffs* 变为 EARMUFFS，使用连字符变为 using_dashes），这意味着 Python 使用它们一点也不困难。

确保我们有良好的互操作性是我们的首要任务之一，因此如果你发现任何漏洞，请及时报告！

**选择 Hy 的优缺点是什么？**

Hy 的一个优势是它拥有完整的宏系统，而这一点是 Python 所无法做到的。宏是特殊的函数，它们在编译阶段改变代码。这使得创建新的领域特定语言变得容易，这些语言由基础语言（在这里是 Hy/Python）和许多宏组成，允许编写出独特且简洁的代码。

至于缺点，Hy 作为一种用 s 表达式编写的 Lisp，承受着难学、难读、难维护的污名。人们可能会因为其复杂性而不愿使用 Hy 来进行项目开发。

Hy 是每个人都喜欢讨厌的 Lisp。Python 用户可能不喜欢它的语法，而 Lisp 爱好者可能会避开它，因为 Hy 直接使用 Python 对象，这意味着一些基本对象的行为可能会让经验丰富的 Lisp 开发者感到惊讶。

希望人们能够超越它的语法，考虑探索之前未触及的 Python 部分。
