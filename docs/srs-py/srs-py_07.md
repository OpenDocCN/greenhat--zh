## **7**

**方法和装饰器**

![image](../images/common01.jpg)

Python 的装饰器是修改函数的便捷方式。装饰器最初在 Python 2.2 中引入，当时有 classmethod() 和 staticmethod() 装饰器，但它们经过大幅改进，变得更加灵活和易于阅读。除了这两个原始装饰器，Python 现在还提供了几个现成的装饰器，并支持轻松创建自定义装饰器。但是，似乎大多数开发者并不理解它们在幕后是如何工作的。

本章的目的是改变这一点——我们将讨论什么是装饰器，如何使用它们，以及如何创建自己的装饰器。接着我们将看看如何使用装饰器来创建静态方法、类方法和抽象方法，并深入了解 super() 函数，它允许你在抽象方法中实现可执行代码。

### **装饰器及其使用时机**

一个 *装饰器* 是一个接受另一个函数作为参数并将其替换为新的修改过的函数的函数。装饰器的主要用途是提取需要在多个函数之前、之后或周围调用的公共代码。如果你曾经写过 Emacs Lisp 代码，你可能使用过 defadvice 装饰器，它允许你定义围绕某个函数调用的代码。如果你使用过 Common Lisp 对象系统（CLOS）中的方法组合，Python 装饰器遵循相同的概念。我们将先看一些简单的装饰器定义，然后再看看在一些常见场景下如何使用装饰器。

#### ***创建装饰器***

你很可能已经使用过装饰器来创建你自己的包装函数。最简单的装饰器，也是最枯燥的例子，就是 identity() 函数，它除了返回原始函数外什么也不做。下面是它的定义：

def identity(f):

return f

然后你可以像这样使用你的装饰器：

@identity

def foo():

return 'bar'

你只需在装饰器的名称前加上 @ 符号，然后输入你想要使用它的函数。这与以下代码是等效的：

def foo():

return 'bar'

foo = identity(foo)

这个装饰器是无用的，但它仍然有效。让我们来看一个更有用的例子，在 [Listing 7-1](ch07.xhtml#ch7list1) 中。

_functions = {}

def register(f):

global _functions

_functions[f.__name__] = f

return f

@register

def foo():

return 'bar'

*Listing 7-1：一个将函数组织在字典中的装饰器*

在 [Listing 7-1](ch07.xhtml#ch7list1) 中，register 装饰器将被装饰的函数名称存储到字典中。然后可以使用函数名称来访问 _functions 字典，进而检索函数：_functions['foo'] 指向 foo() 函数。

在接下来的章节中，我将解释如何编写自己的装饰器。然后，我会讲解 Python 提供的内置装饰器如何工作，并解释如何（以及何时）使用它们。

#### ***编写装饰器***

如前所述，装饰器通常用于重构围绕函数的重复代码。考虑以下一组函数，它们需要检查接收到的用户名参数是否为管理员，如果用户不是管理员，则抛出异常：

class Store(object):

def get_food(self, username, food):

if username != 'admin':

raise Exception("此用户无法获取食物")

return self.storage.get(food)

def put_food(self, username, food):

if username != 'admin':

raise Exception("此用户无法放置食物")

self.storage.put(food)

我们可以看到这里有一些重复的代码。使这段代码更高效的显而易见的第一步是将检查管理员状态的代码提取出来：

➊ def check_is_admin(username):

if username != 'admin':

raise Exception("此用户无法获取或放置食物")

class Store(object):

def get_food(self, username, food):

check_is_admin(username)

return self.storage.get(food)

def put_food(self, username, food):

check_is_admin(username)

self.storage.put(food)

我们将检查代码移到了它自己的函数中 ➊。现在我们的代码看起来更简洁了，但如果我们使用装饰器，效果会更好，如[清单 7-2](ch07.xhtml#ch7list2)所示。

def check_is_admin(f):

➊ def wrapper(*args, **kwargs):

if kwargs.get('username') != 'admin':

raise Exception("此用户无法获取或放置食物")

return f(*args, **kwargs) return wrapper

class Store(object):

@check_is_admin

def get_food(self, username, food):

return self.storage.get(food)

@check_is_admin

def put_food(self, username, food):

self.storage.put(food)

*清单 7-2：为简化代码添加装饰器*

我们定义了我们的check_is_admin装饰器 ➊，然后每当需要检查访问权限时调用它。装饰器使用kwargs变量检查传递给函数的参数，并获取用户名参数，在调用实际函数之前执行用户名检查。像这样使用装饰器可以更轻松地管理通用功能。对于有经验的Python程序员来说，这可能是老生常谈，但你可能没有意识到，采用这种简单的装饰器实现方法有一些重大的缺陷。

#### ***叠加装饰器***

你还可以在单个函数或方法上使用多个装饰器，如[清单 7-3](ch07.xhtml#ch7list3)所示。

def check_user_is_not(username):

def user_check_decorator(f):

def wrapper(*args, **kwargs):

if kwargs.get('username') == username:

raise Exception("此用户无法获取食物")

return f(*args, **kwargs)

return wrapper

return user_check_decorator

class Store(object):

@check_user_is_not("admin")

@check_user_is_not("user123")

def get_food(self, username, food):

return self.storage.get(food)

*清单 7-3：使用多个装饰器修饰单个函数*

在这里，check_user_is_not() 是我们装饰器 user_check_decorator() 的工厂函数。它创建了一个依赖于用户名变量的函数装饰器，并返回该变量。函数 user_check_decorator() 将作为 get_food() 的函数装饰器。

函数 get_food() 被 check_user_is_not() 装饰了两次。问题在于，应该首先检查哪个用户名——admin 还是 user123？答案在下面的代码中，我将[列表 7-3](ch07.xhtml#ch7list3)转换成了不使用装饰器的等效代码。

class Store(object):

def get_food(self, username, food):

return self.storage.get(food)

Store.get_food = check_user_is_not("user123")(Store.get_food)

Store.get_food = check_user_is_not("admin")(Store.get_food)

装饰器列表是从上到下应用的，因此离 `def` 关键字最近的装饰器将首先应用并最后执行。在上面的例子中，程序会先检查 admin，然后检查 user123。

#### ***编写类装饰器***

也可以实现类装饰器，尽管这些在实际使用中较少见。*类装饰器*的工作原理与函数装饰器相同，但它们作用于类而不是函数。以下是一个类装饰器的示例，它为两个类设置属性：

import uuid

def set_class_name_and_id(klass):

klass.name = str(klass)

klass.random_id = uuid.uuid4()

return klass

@set_class_name_and_id

class SomeClass(object):

pass

当类被加载和定义时，它将像下面这样设置名称和 random_id 属性：

>>> SomeClass.name

"<class '__main__.SomeClass'>"

>>> SomeClass.random_id

UUID('d244dc42-f0ca-451c-9670-732dc32417cd')

与函数装饰器类似，类装饰器对于提取操作类的公共代码非常有用。

类装饰器的另一种可能用途是将函数或类与类进行封装。例如，类装饰器通常用于封装一个存储状态的函数。以下示例封装了 print() 函数，以检查它在一个会话中被调用了多少次：

class CountCalls(object):

def __init__(self, f):

self.f = f

self.called = 0 def __call__(self, *args, **kwargs):

self.called += 1

return self.f(*args, **kwargs)

@CountCalls

def print_hello():

print("hello")

然后我们可以用它来检查函数 print_hello() 被调用了多少次：

>>> print_hello.called

0

>>> print_hello()

hello

>>> print_hello.called

1

##### **使用 update_wrapper 装饰器获取原始属性**

如前所述，装饰器将原始函数替换为一个新的即时生成的函数。然而，这个新函数缺少原始函数的许多属性，如其文档字符串和名称。[列表 7-4](ch07.xhtml#ch7list4) 显示了函数 foobar() 被 is_admin 装饰器装饰后，如何丧失其文档字符串和名称属性。

>>> def is_admin(f):

...     def wrapper(*args, **kwargs):

...         if kwargs.get('username') != 'admin':

...             raise Exception("该用户没有权限获取食物")

...         return f(*args, **kwargs)

...     return wrapper

...

>>> def foobar(username="someone"):

...     """做疯狂的事情。"""

...     pass

...

>>> foobar.func_doc

'做疯狂的事情。'

>>> foobar.__name__

'foobar'

>>> @is_admin

... def foobar(username="someone"):

...     """做疯狂的事情。"""

...     pass

...

>>> foobar.__doc__

>>> foobar.__name__

'wrapper'

*Listing 7-4: 一个被装饰的函数失去了它的文档字符串和名称属性。*

如果函数没有正确的文档字符串和名称属性，在生成源代码文档等各种情况下可能会造成问题。

幸运的是，Python 标准库中的 functools 模块通过 update_wrapper() 函数解决了这个问题，该函数将从原始函数中丢失的属性复制到包装器本身。update_wrapper() 的源代码显示在 [Listing 7-5](ch07.xhtml#ch7list5) 中。

WRAPPER_ASSIGNMENTS = ('__module__', '__name__', '__qualname__', '__doc__',

'__annotations__')

WRAPPER_UPDATES = ('__dict__',)

def update_wrapper(wrapper,

wrapped,

assigned = WRAPPER_ASSIGNMENTS,

updated = WRAPPER_UPDATES):

for attr in assigned:

try:

value = getattr(wrapped, attr)

except AttributeError:

pass

else:

setattr(wrapper, attr, value)

for attr in updated:

getattr(wrapper, attr).update(getattr(wrapped, attr, {}))

# 问题 #17482：最后设置 __wrapped__，以避免不小心复制它

# 在更新 __dict__ 时来自包装函数

wrapper.__wrapped__ = wrapped

# 返回包装器，以便可以通过 partial() 用作装饰器

return wrapper

*Listing 7-5: update_wrapper() 源代码*

在 [Listing 7-5](ch07.xhtml#ch7list5) 中，update_wrapper() 源代码突出了包装函数时值得保存的属性。默认情况下，__name__ 属性、__doc__ 属性以及一些其他属性会被复制。你还可以个性化哪些函数的属性会被复制到装饰函数中。当我们使用 update_wrapper() 重写我们在 [Listing 7-4](ch07.xhtml#ch7list4) 中的示例时，效果要好得多：

>>> def foobar(username="someone"):

...     """做疯狂的事情。"""

...     pass

...

>>> foobar = functools.update_wrapper(is_admin, foobar)

>>> foobar.__name__

'foobar'

>>> foobar.__doc__

'做疯狂的事情。'

现在，foobar() 函数即使被 is_admin 装饰，也具有正确的名称和文档字符串。

##### **wraps：装饰器的装饰器**

使用 update_wrapper() 手动创建装饰器时可能会变得繁琐，因此 functools 提供了一个装饰器装饰器，叫做 wraps。[Listing 7-6](ch07.xhtml#ch7list6) 展示了 wraps 装饰器的使用。

import functools

def check_is_admin(f):

@functools.wraps(f)

def wrapper(*args, **kwargs):

if kwargs.get('username') != 'admin':

raise Exception("该用户没有权限获取食物")

return f(*args, **kwargs)

return wrapper

class Store(object):

@check_is_admin

def get_food(self, username, food):

"""从存储中获取食物。"""

返回 `self.storage.get(food)`

*列表 7-6：使用 functools 中的 wraps 更新我们的装饰器*

使用 functools.wrap，返回 `wrapper()` 函数的装饰器函数 `check_is_admin()` 会负责复制传入的函数 `f` 的文档字符串、函数名和其他信息。因此，被装饰的函数（在这个例子中是 `get_food()`）仍然保留其未改变的签名。

##### **使用 inspect 提取相关信息**

到目前为止，在我们的示例中，我们假设被装饰的函数会总是作为关键字参数传入一个 `username`，但实际情况可能并非如此。它可能会传入一堆信息，我们需要从中提取出 `username` 进行检查。考虑到这一点，我们将构建一个更智能的装饰器版本，它可以查看被装饰函数的参数并提取所需的信息。

为此，Python 有一个 inspect 模块，它允许我们检索一个函数的签名并对其进行操作，如 [列表 7-7](ch07.xhtml#ch7list7) 所示。

导入 functools

导入 inspect

def check_is_admin(f):

@functools.wraps(f)

def wrapper(*args, **kwargs):

func_args = inspect.getcallargs(f, *args, **kwargs)

如果 `func_args.get('username')` 不等于 'admin'：

抛出异常("该用户没有权限获取食物")

返回 f(*args, **kwargs)

返回 wrapper @check_is_admin

def get_food(username, type='chocolate'):

返回类型 + " nom nom nom!"

*列表 7-7：使用 inspect 模块中的工具提取信息*

这里负责主要工作的函数是 `inspect.getcallargs()`，它返回一个字典，其中包含参数的名称和值作为键值对。在我们的示例中，这个函数返回 `{'username': 'admin', 'type': 'chocolate'}`。这意味着我们的装饰器不需要检查 `username` 参数是位置参数还是关键字参数；装饰器只需要在字典中查找 `username`。

使用 functools.wraps 和 inspect 模块，你应该能够编写任何你所需要的自定义装饰器。然而，千万不要滥用 inspect 模块：尽管能够猜测函数将接受什么参数听起来很方便，但这种能力可能会变得脆弱，当函数签名发生变化时，可能会轻易破坏。装饰器是实现 *不要重复自己（Don’t Repeat Yourself）* 这一开发者理念的绝佳方式。

### **Python 中的方法是如何工作的**

方法使用起来相当简单，你可能已经在不深入了解的情况下正确使用过它们。但要理解某些装饰器的作用，你需要了解方法在幕后是如何工作的。

*方法* 是一个作为类属性存储的函数。让我们看看当我们直接访问这样的属性时会发生什么：

>>> class Pizza(object):

...     def __init__(self, size):

...         self.size = size

...     def get_size(self):

...         返回 self.size

...

>>> Pizza.get_size

<function Pizza.get_size at 0x7fdbfd1a8b90>

我们被告知 get_size() 是一个函数——但为什么会是这样呢？原因在于此时，get_size() 并没有绑定到任何特定的对象上。因此，它被当作普通函数对待。如果我们试图直接调用它，Python 会抛出错误，像这样：

>>> Pizza.get_size()

Traceback (most recent call last):

File "<stdin>", line 1, in <module>

TypeError: get_size() 缺少 1 个必需的位置参数：'self'

Python 报告我们没有提供必要的 self 参数。确实，由于它没有绑定到任何对象，self 参数无法自动设置。然而，我们能够使用 get_size() 函数，不仅可以传递类的任意实例给方法，如果需要的话，还可以传递 *任何* 对象，只要它具备该方法期望找到的属性。下面是一个例子：

>>> Pizza.get_size(Pizza(42))

42

这个调用成功了，正如承诺的那样。然而，它并不是特别方便：每次我们想调用某个方法时，都必须引用类。

所以 Python 为我们额外做了一步，将类的方法绑定到它的实例上。换句话说，我们可以从任何 Pizza 实例访问 get_size()，而且更好的是，Python 会自动将对象本身传递给方法的 self 参数，像这样：

>>> Pizza(42).get_size

<bound method Pizza.get_size of <__main__.Pizza object at 0x7f3138827910>>

>>> Pizza(42).get_size()

42

正如预期的那样，我们不需要为 get_size() 提供任何参数，因为它是一个绑定方法：它的 self 参数会自动设置为我们的 Pizza 实例。这里有一个更清晰的例子：

>>> m = Pizza(42).get_size

>>> m()

42

只要你有绑定方法的引用，你甚至不需要保存对 Pizza 对象的引用。此外，如果你有一个方法的引用，但你想知道它绑定的是哪个对象，你可以直接检查该方法的 __self__ 属性，像这样：

>>> m = Pizza(42).get_size

>>> m.__self__

<__main__.Pizza object at 0x7f3138827910>

>>> m == m.__self__.get_size

True

显然，我们仍然可以访问到我们的对象引用，如果需要的话，我们可以找到它。

### **静态方法**

*静态方法* 属于类，而不是类的实例，因此它们实际上并不操作或影响类的实例。相反，静态方法作用于它所接受的参数。静态方法通常用于创建工具函数，因为它们不依赖于类或对象的状态。

例如，在 [Listing 7-8](ch07.xhtml#ch7list8) 中，静态方法 mix_ingredients() 属于 Pizza 类，但实际上它可以用于混合任何其他食物的食材。

class Pizza(object):

@staticmethod

def mix_ingredients(x, y):

return x + y

def cook(self):

return self.mix_ingredients(self.cheese, self.vegetables)

*Listing 7-8: 作为类一部分创建静态方法*

如果你愿意，可以将 mix_ingredients() 写成一个非静态方法，但它会需要一个 self 参数，而这个参数实际上永远不会被使用。使用 @staticmethod 装饰器为我们带来了几个好处。

第一个好处是速度：Python 不需要为我们创建的每一个 Pizza 对象实例化一个绑定方法。绑定方法本身也是对象，创建它们会消耗 CPU 和内存资源——尽管这个开销很小。使用静态方法可以避免这种开销，像这样：

>>> Pizza().cook 是 Pizza().cook

False

>>> Pizza().mix_ingredients 是 Pizza.mix_ingredients

True

>>> Pizza().mix_ingredients 是 Pizza().mix_ingredients

True

第二，静态方法提高了代码的可读性。当我们看到 @staticmethod 时，我们知道该方法不依赖于对象的状态。

第三，静态方法可以在子类中被重写。如果我们不是使用静态方法，而是使用在模块顶层定义的 mix_ingredients() 函数，那么继承自 Pizza 的类就无法在不重写 cook() 方法的情况下改变我们混合比萨配料的方式。使用静态方法后，子类可以根据自己的需求重写该方法。

不幸的是，Python 并不总是能够自动检测一个方法是否是静态方法——我认为这是语言设计上的一个缺陷。一种可能的做法是添加一个检查，检测到这种模式时通过 flake8 发出警告。我们将在《扩展 flake8 与 AST 检查》一节中探讨如何做到这一点，详见 [第 140 页](ch09.xhtml#page_140)。

### **类方法**

*类方法* 是绑定到类而不是其实例的。这意味着这些方法无法访问对象的状态，只能访问类的状态和方法。[清单 7-9](ch07.xhtml#ch7list9)展示了如何编写类方法。

>>> class Pizza(object):

...     radius = 42

...     @classmethod

...     def get_radius(cls):

...         return cls.radius

...

>>> Pizza.get_radius

<bound method type.get_radius of <class '__main__.Pizza'>>

>>> Pizza().get_radius

<bound method type.get_radius of <class '__main__.Pizza'>>

>>> Pizza.get_radius 是 Pizza().get_radius

True

>>> Pizza.get_radius()

42

*清单 7-9：将类方法绑定到类*

如你所见，有多种方式可以访问 get_radius() 类方法，但无论你选择哪种方式，方法总是与它所附加的类绑定在一起。而且，它的第一个参数必须是类本身。记住：类也是对象！

类方法主要用于创建 *工厂方法*，这些方法使用不同于 __init__ 的签名来实例化对象：

class Pizza(object):

def __init__(self, ingredients):

self.ingredients = ingredients

@classmethod

def from_fridge(cls, fridge):

return cls(fridge.get_cheese() + fridge.get_vegetables())

如果我们在这里使用@staticmethod而不是@classmethod，我们将不得不在方法中硬编码Pizza类的名称，这样任何继承自Pizza的类都无法使用我们的工厂方法。在这种情况下，我们提供了一个from_fridge()工厂方法，允许我们传入一个Fridge对象。如果我们像这样调用该方法Pizza.from_fridge(myfridge)，它会返回一个全新的Pizza，使用myfridge中可用的食材。

每当你编写一个仅关心对象类而不关心对象状态的方法时，它应该被声明为类方法。

### **抽象方法**

*抽象方法*定义在抽象基类中，该类本身可能不提供任何实现。当一个类有一个抽象方法时，它不能被实例化。因此，*抽象类*（定义为至少有一个抽象方法的类）必须作为另一个类的父类。这个子类将负责实现抽象方法，从而使得父类能够被实例化。

我们可以使用抽象基类明确表示从基类派生的其他相关类之间的关系，但使得抽象基类本身无法被实例化。通过使用抽象基类，你可以确保从基类派生的类实现基类中特定的方法，否则将引发异常。以下示例展示了在Python中编写抽象方法的最简单方法：

class Pizza(object):

@staticmethod

def get_radius():

raise NotImplementedError

使用这个定义，任何继承Pizza的类必须实现并重写get_radius()方法；否则，调用该方法时将引发如这里所示的异常。这对于确保Pizza的每个子类都实现其计算并返回半径的方式非常有用。

这种实现抽象方法的方式有一个缺点：如果你编写了一个继承自Pizza的类，但忘记实现get_radius()方法，只有在你尝试在运行时使用该方法时，才会引发错误。以下是一个示例：

>>> Pizza()

<__main__.Pizza对象，位于0x7fb747353d90>

>>> Pizza().get_radius()

回溯（最近的调用最后）：

文件“<stdin>”，第1行，位于<module>

文件“<stdin>”，第3行，位于get_radius

NotImplementedError

由于Pizza是可以直接实例化的，因此无法阻止这种情况的发生。为了确保你能提前警告忘记实现和重写方法，或者尝试实例化一个带有抽象方法的对象，可以改用Python的内置abc（抽象基类）模块，像这样：

import abc

class BasePizza(object, metaclass=abc.ABCMeta):

@abc.abstractmethod

def get_radius(self):

"""此方法应执行某个操作。"""

abc 模块提供了一组装饰器，可以用于定义为抽象方法的方法，并提供一个元类来启用这一功能。当你使用 abc 及其特殊元类，如上所示，如果实例化一个 BasePizza 或其子类而没有重写 `get_radius()`，就会导致 TypeError：

>>> BasePizza()

Traceback (most recent call last):

文件 "<stdin>"，第 1 行，在 <module> 中

TypeError: 无法实例化具有抽象方法的抽象类 BasePizza

get_radius

我们尝试实例化抽象的 BasePizza 类，立即被告知无法完成此操作！

虽然使用抽象方法不能保证用户实现该方法，但这个装饰器帮助你更早地捕获错误。特别是在你提供必须由其他开发者实现的接口时，这是一种很好的文档提示。

### **混合静态方法、类方法和抽象方法**

这些装饰器各自独立有用，但有时你可能需要将它们结合使用。

例如，你可以将工厂方法定义为类方法，同时强制要求在子类中实现。在这种情况下，你需要将类方法同时定义为抽象方法和类方法。本节提供了一些帮助你实现这一点的提示。

首先，抽象方法的原型并非固定不变。当你实现该方法时，没有什么能阻止你根据需要扩展参数列表。[清单 7-10](ch07.xhtml#ch7list10) 就是一个例子，展示了子类如何扩展其父类抽象方法的签名。

import abc

class BasePizza(object, metaclass=abc.ABCMeta):

@abc.abstractmethod

def get_ingredients(self):

"""返回配料列表。"""

class Calzone(BasePizza):

def get_ingredients(self, with_egg=False):

egg = Egg() if with_egg else None

return self.ingredients + [egg]

*清单 7-10：使用子类扩展其父类的抽象方法签名*

我们定义了 Calzone 子类继承自 BasePizza 类。我们可以按任何方式定义 Calzone 子类的方法，只要它们支持我们在 BasePizza 中定义的接口。这包括将方法实现为类方法或静态方法。以下代码定义了一个抽象的 `get_ingredients()` 方法在基类中，并在 DietPizza 子类中定义了一个静态 `get_ingredients()` 方法：

import abc

class BasePizza(object, metaclass=abc.ABCMeta):

@abc.abstractmethod

def get_ingredients(self):

"""返回配料列表。"""

class DietPizza(BasePizza):

@staticmethod

def get_ingredients():

return None

尽管我们的静态 `get_ingredients()` 方法并没有基于对象的状态返回结果，但它支持我们抽象的 BasePizza 类接口，因此它仍然有效。

也可以在@abstractmethod之上使用@staticmethod和@classmethod装饰器，表示一个方法是静态方法且是抽象的，正如在[示例7-11](ch07.xhtml#ch7list11)中展示的那样。

import abc

class BasePizza(object, metaclass=abc.ABCMeta):

ingredients = ['cheese']

@classmethod

@abc.abstractmethod

def get_ingredients(cls):

"""返回食材列表。"""

return cls.ingredients

*示例7-11：将类方法装饰器与抽象方法一起使用*

抽象方法get_ingredients()需要在子类中实现，但它也是一个类方法，这意味着它接收的第一个参数将是一个类（而不是一个对象）。

请注意，通过像这样在BasePizza中将get_ingredients()定义为类方法，你并没有强制任何子类必须将get_ingredients()定义为类方法——它也可以是一个普通方法。如果我们将其定义为静态方法，也是一样的：没有办法强制子类将抽象方法实现为特定类型的方法。正如我们所看到的，你可以在子类中以任何你喜欢的方式改变抽象方法的签名。

#### ***在抽象方法中放置实现***

等一下：在[示例7-12](ch07.xhtml#ch7list12)中，我们在一个抽象方法中实现了代码。我们能*这样做*吗？答案是可以的。Python对此没有问题！你可以在抽象方法中放置代码，并通过super()调用它，正如在[示例7-12](ch07.xhtml#ch7list12)中展示的那样。

import abc

class BasePizza(object, metaclass=abc.ABCMeta):

default_ingredients = ['cheese']

@classmethod

@abc.abstractmethod

def get_ingredients(cls):

"""返回默认食材列表。"""

return cls.default_ingredients

class DietPizza(BasePizza):

def get_ingredients(self):

return [Egg()] + super(DietPizza, self).get_ingredients()

*示例7-12：在抽象方法中使用实现*

在这个例子中，继承自BasePizza的每个Pizza都必须重写get_ingredients()方法，但每个Pizza也可以访问基类的获取食材列表的默认机制。这个机制在提供接口的同时，也提供了可能对所有继承类都有用的基础代码。

#### ***关于super的真相***

Python一直允许开发者使用单继承和多继承来扩展他们的类，但即使到今天，许多开发者似乎并不理解这些机制，以及与之相关的super()方法是如何工作的。要完全理解你的代码，你需要理解这些权衡。

多重继承在许多地方都有使用，特别是在涉及mixin模式的代码中。*mixin*是一种从两个或更多类继承的类，它将这些类的特性结合在一起。

**注意**

*单继承与多继承、组合或甚至鸭子类型的优缺点超出了本书的讨论范围，因此我们不会在这里覆盖所有内容。如果你不熟悉这些概念，建议你阅读相关资料，形成你自己的观点。*

如你现在应该已经知道的，类在 Python 中是对象。用于创建类的构造是一个特殊的语句，你应该对它非常熟悉：class 类名（继承表达式）。

括号中的代码是一个 Python 表达式，它返回要用作类父类的类对象列表。通常，你会直接指定它们，但你也可以像这样写来指定父类对象的列表：

>>> def parent():

...     return object

...

>>> class A(parent()):

...     pass

...

>>> A.mro()

[<class '__main__.A'>, <type 'object'>]

这段代码按预期工作：我们声明类 A，并以 object 作为其父类。类方法 mro() 返回 *方法解析顺序*，用于解析属性——它定义了如何通过类之间的继承树找到下一个调用的方法。当前的 MRO 系统首次在 Python 2.3 中实现，其内部工作原理在 Python 2.3 发行说明中有描述。它定义了系统如何浏览类之间的继承树来找到要调用的方法。

我们已经看到，在父类中调用方法的标准方式是使用 super() 函数，但你可能不知道的是，super() 实际上是一个构造函数，每次调用它时你都会实例化一个超类对象。它接受一个或两个参数：第一个参数是一个类，第二个（可选）参数可以是第一个参数的子类或其实例。

由构造函数返回的对象充当第一个参数父类的代理。它有自己的 __getattribute__ 方法，该方法遍历 MRO 列表中的类并返回它找到的第一个匹配的属性。__getattribute__ 方法会在检索 super() 对象的属性时被调用，如 [示例 7-13](ch07.xhtml#ch7list13) 所示。

>>> class A(object):

...     bar = 42

...     def foo(self):

...             pass

...

>>> class B(object):

...     bar = 0

...

>>> class C(A, B):

...     xyz = 'abc'

...

>>> C.mro()

[<class '__main__.C'>, <class '__main__.A'>, <class '__main__.B'>, <type 'object'>]

>>> super(C, C()).bar

42

>>> super(C, C()).foo

<bound method C.foo of <__main__.C object at 0x7f0299255a90>>

>>> super(B).__self__ >>> super(B, B()).__self__

<__main__.B object at 0x1096717f0>

*示例 7-13：super() 函数是一个构造函数，每次调用它时都会实例化一个超类对象。*

当请求 C 实例的超类属性时，super() 对象的 __getattribute__ 方法会遍历 MRO 列表，并返回它找到的第一个具有该超类属性的类中的属性。

在 [列表 7-13](ch07.xhtml#ch7list13) 中，我们使用了两个参数调用了 super()，意味着我们使用了 *绑定* 的 super 对象。如果我们只用一个参数调用 super()，它会返回一个 *未绑定* 的 super 对象：

>>> super(C)

<super: <class 'C'>, NULL>

由于没有提供实例作为第二个参数，super 对象无法绑定到任何实例。因此，你不能使用这个未绑定的对象来访问类属性。如果你尝试这样做，你会得到以下错误：

>>> super(C).foo

最近的调用栈（Traceback）：

文件 "<stdin>"，第 1 行，位于 <module> 中

AttributeError: 'super' 对象没有属性 'foo'

>>> super(C).bar

最近的调用栈（Traceback）：

文件 "<stdin>"，第 1 行，位于 <module> 中

AttributeError: 'super' 对象没有属性 'bar'

>>> super(C).xyz

最近的调用栈（Traceback）：

文件 "<stdin>"，第 1 行，位于 <module> 中

AttributeError: 'super' 对象没有属性 'xyz'

初看之下，似乎这种未绑定的 super 对象没有用，但实际上，超类实现描述符协议 __get__ 的方式使得未绑定的 super 对象作为类属性非常有用：

>>> class D(C):

...     sup = super(C)

...

>>> D().sup

<super: <class 'C'>, <D object>>

>>> D().sup.foo

<bound method D.foo of <__main__.D object at 0x7f0299255bd0>>

>>> D().sup.bar

42

未绑定的 super 对象的 __get__ 方法被调用，使用实例 super(C).__get__(D()) 和属性名 'foo' 作为参数，从而能够找到并解析 foo。

**注意**

*即使你从未听说过描述符协议，你也很可能通过 @property 装饰器使用过它，却没有意识到。描述符协议是 Python 中的一种机制，它允许存储为属性的对象返回除自身以外的内容。这个协议在本书中没有涉及，但你可以在 Python 数据模型文档中找到更多相关信息。*

在许多情况下，使用 super() 可能会很棘手，比如在处理沿继承链的不同方法签名时。不幸的是，并没有适用于所有情况的灵丹妙药。最好的预防措施是使用一些技巧，比如让所有方法都通过 *args, **kwargs 接受参数。

从 Python 3 开始，super() 增加了一些魔法：现在可以在方法内部调用 super()，而不需要任何参数。当没有传递参数给 super() 时，它会自动搜索栈帧中的参数：

class B(A):

def foo(self):

super().foo()

访问子类中父类属性的标准方式是 super()，你应该始终使用它。它允许父类方法的协作调用，避免出现意外情况，比如父类方法未被调用或在多重继承时被调用两次。

### **总结**

掌握了本章内容后，你应该在任何与 Python 方法定义相关的事情上无往不利。装饰器在代码抽象化方面至关重要，合理使用 Python 提供的内置装饰器可以极大地提升 Python 代码的整洁度。抽象类在向其他开发者和服务提供 API 时尤为有用。

类继承往往难以完全理解，了解语言内部机制的概览是全面掌握其工作原理的好方法。现在，这个话题对你来说应该没有任何秘密了！
