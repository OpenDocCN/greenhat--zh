## 第八章：**方法和装饰器**

![image](img/common01.jpg)

Python 的装饰器是修改函数的便捷方式。装饰器首次在 Python 2.2 中引入，最初是 `classmethod()` 和 `staticmethod()` 装饰器，但后来进行了重构，使其更加灵活和易读。除了这两个原始的装饰器，Python 现在提供了一些现成的装饰器，并支持自定义装饰器的简单创建。但似乎大多数开发者并不理解它们背后的工作原理。

本章旨在改变这一点——我们将讨论什么是装饰器以及如何使用它们，并且还会介绍如何创建自己的装饰器。然后，我们将探讨如何使用装饰器创建静态方法、类方法和抽象方法，并深入了解 `super()` 函数，它允许你将可实现的代码放入抽象方法中。

### **装饰器及其使用时机**

*装饰器*是一个接受另一个函数作为参数并将其替换为一个新的、修改后的函数的函数。装饰器的主要用法是在需要在多个函数之前、之后或周围调用的公共代码进行提取。如果你曾经编写过 Emacs Lisp 代码，你可能使用过 `defadvice` 装饰器，它允许你定义在函数周围调用的代码。如果你使用过 Common Lisp 对象系统（CLOS）中的方法组合，Python 的装饰器遵循相同的概念。我们将查看一些简单的装饰器定义，然后我们将研究一些常见的使用装饰器的场景。

#### ***创建装饰器***

你很可能已经使用过装饰器来创建自己的包装函数。最简单的装饰器，也是最无聊的例子，就是 `identity()` 函数，它除了返回原始函数外什么也不做。以下是它的定义：

```py
def identity(f):
    return f
```

然后你可以像这样使用你的装饰器：

```py
@identity
def foo():
    return 'bar'
```

你输入装饰器的名称，前面加上 `@` 符号，然后输入你想要使用装饰器的函数。这与以下代码是等效的：

```py
def foo():
    return 'bar'
foo = identity(foo)
```

这个装饰器没有用处，但它能正常工作。我们来看看另一个更有用的例子，参见清单 7-1。

```py
_functions = {}
def register(f):
    global _functions
    _functions[f.__name__] = f
    return f
@register
def foo():
    return 'bar'
```

*清单 7-1：用于将函数组织到字典中的装饰器*

在清单 7-1 中，`register` 装饰器将装饰的函数名称存储到字典中。然后，可以使用函数名称访问 `_functions` 字典来检索函数：`_functions['foo']` 指向 `foo()` 函数。

在接下来的章节中，我将解释如何编写自己的装饰器。然后我将介绍 Python 提供的内置装饰器是如何工作的，并解释如何（以及何时）使用它们。

#### ***编写装饰器***

如前所述，装饰器通常用于重构围绕函数的重复代码。考虑以下一组函数，它们需要检查作为参数传入的用户名是否是管理员，如果不是管理员，则引发异常：

```py
class Store(object):
    def get_food(self, username, food):
        if username != 'admin':
            raise Exception("This user is not allowed to get food")
        return self.storage.get(food)

    def put_food(self, username, food):
        if username != 'admin':
            raise Exception("This user is not allowed to put food")
        self.storage.put(food)
```

我们可以看到这里有一些重复的代码。使这段代码更高效的明显第一步是提取检查管理员状态的代码：

```py
➊ def check_is_admin(username):
    if username != 'admin':
        raise Exception("This user is not allowed to get or put food")

class Store(object):
    def get_food(self, username, food):
        check_is_admin(username)
        return self.storage.get(food)

    def put_food(self, username, food):
        check_is_admin(username)
        self.storage.put(food)
```

我们已经将检查代码移到它自己的函数中➊。现在我们的代码看起来更简洁了，但如果使用装饰器的话，我们可以做得更好，如列表 7-2 所示。

```py
def check_is_admin(f):
 ➊ def wrapper(*args, **kwargs):
        if kwargs.get('username') != 'admin':
            raise Exception("This user is not allowed to get or put food")
        return f(*args, **kwargs) return wrapper

class Store(object):
    @check_is_admin
    def get_food(self, username, food):
        return self.storage.get(food)

    @check_is_admin
    def put_food(self, username, food):
        self.storage.put(food)
```

*列表 7-2：将装饰器添加到提取的代码中*

我们定义了`check_is_admin`装饰器➊，然后每当需要检查访问权限时就调用它。装饰器使用`kwargs`变量检查传递给函数的参数，并获取`username`参数，在调用实际函数之前执行用户名检查。像这样使用装饰器使得管理通用功能变得更加容易。对于有较多 Python 经验的人来说，这可能是司空见惯的事，但你可能没有意识到，这种实现装饰器的简单方法存在一些重大缺陷。

#### ***堆叠装饰器***

你还可以在单个函数或方法上使用多个装饰器，如列表 7-3 所示。

```py
def check_user_is_not(username):
    def user_check_decorator(f):
        def wrapper(*args, **kwargs):
            if kwargs.get('username') == username:
                raise Exception("This user is not allowed to get food")
            return f(*args, **kwargs)
        return wrapper
    return user_check_decorator

class Store(object):
    @check_user_is_not("admin")
    @check_user_is_not("user123")
    def get_food(self, username, food):
        return self.storage.get(food)
```

*列表 7-3：在单个函数上使用多个装饰器*

这里，`check_user_is_not()`是我们的装饰器`user_check_decorator()`的工厂函数。它创建一个依赖于`username`变量的函数装饰器，然后返回该变量。函数`user_check_decorator()`将作为`get_food()`的函数装饰器。

函数`get_food()`被`check_user_is_not()`装饰了两次。这里的问题是应该先检查哪个用户名——`admin`还是`user123`？答案在下面的代码中，我将列表 7-3 翻译成了没有使用装饰器的等效代码。

```py
class Store(object):
    def get_food(self, username, food):
        return self.storage.get(food)

Store.get_food = check_user_is_not("user123")(Store.get_food)
Store.get_food = check_user_is_not("admin")(Store.get_food)
```

装饰器列表是从上到下应用的，因此最靠近`def`关键字的装饰器会首先被应用并最后执行。在上面的示例中，程序会首先检查`admin`，然后检查`user123`。

#### ***编写类装饰器***

也可以实现类装饰器，尽管这些在实际应用中使用得较少。*类装饰器*的工作方式与函数装饰器相同，但它们作用于类而不是函数。以下是一个类装饰器的示例，它为两个类设置属性：

```py
import uuid

def set_class_name_and_id(klass):
    klass.name = str(klass)
    klass.random_id = uuid.uuid4()
    return klass

@set_class_name_and_id
class SomeClass(object):
    pass
```

当类被加载和定义时，它将设置`name`和`random_id`属性，如下所示：

```py
>>> SomeClass.name
"<class '__main__.SomeClass'>"
>>> SomeClass.random_id
UUID('d244dc42-f0ca-451c-9670-732dc32417cd')
```

与函数装饰器一样，这对于提取处理类的通用代码非常有用。

类装饰器的另一个可能用途是用类来包装函数或类。例如，类装饰器通常用于包装存储状态的函数。以下示例包装了`print()`函数，用于检查它在一个会话中被调用了多少次：

```py
class CountCalls(object):
    def __init__(self, f):
        self.f = f
        self.called = 0 def __call__(self, *args, **kwargs):
        self.called += 1
        return self.f(*args, **kwargs)

@CountCalls
def print_hello():
    print("hello")
```

然后我们可以使用它来检查`print_hello()`函数被调用了多少次：

```py
>>> print_hello.called
0
>>> print_hello()
hello
>>> print_hello.called
1
```

##### **使用 update_wrapper 装饰器获取原始属性**

如前所述，装饰器会用一个新的函数替换原始函数，这个新函数是动态生成的。然而，这个新函数缺少了原始函数的许多属性，如文档字符串和名称。示例 7-4 展示了函数`foobar()`在被`is_admin`装饰器装饰后，失去了文档字符串和名称属性。

```py
>>> def is_admin(f):
...     def wrapper(*args, **kwargs):
...         if kwargs.get('username') != 'admin':
...             raise Exception("This user is not allowed to get food")
...         return f(*args, **kwargs)
...     return wrapper
...
>>> def foobar(username="someone"):
...     """Do crazy stuff."""
...     pass
...
>>> foobar.func_doc
'Do crazy stuff.'
>>> foobar.__name__
'foobar'
>>> @is_admin
... def foobar(username="someone"):
...     """Do crazy stuff."""
...     pass
...
>>> foobar.__doc__
>>> foobar.__name__
'wrapper'
```

*示例 7-4：一个被装饰的函数失去了其文档字符串和名称属性。*

在各种情况下，函数没有正确的文档字符串和名称属性可能会带来问题，例如在生成源代码文档时。

幸运的是，Python 标准库中的`functools`模块通过`update_wrapper()`函数解决了这个问题，该函数将原始函数丢失的属性复制到包装器本身。示例 7-5 展示了`update_wrapper()`的源代码。

```py
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
    # Issue #17482: set __wrapped__ last so we don't inadvertently copy it
    # from the wrapped function when updating __dict__
    wrapper.__wrapped__ = wrapped
    # Return the wrapper so this can be used as a decorator via partial()
    return wrapper
```

*示例 7-5：update_wrapper()源代码*

在示例 7-5 中，`update_wrapper()`源代码突出了在使用装饰器包装函数时哪些属性值得保存。默认情况下，`__name__`属性、`__doc__`属性以及其他一些属性会被复制。你也可以自定义哪些函数的属性被复制到装饰后的函数上。当我们使用`update_wrapper()`重写示例 7-4 中的例子时，效果更好：

```py
>>> def foobar(username="someone"):
...     """Do crazy stuff."""
...     pass
...
>>> foobar = functools.update_wrapper(is_admin, foobar)
>>> foobar.__name__
'foobar'
>>> foobar.__doc__
'Do crazy stuff.'
```

现在，即使`foobar()`函数被`is_admin`装饰，它仍然拥有正确的名称和文档字符串。

##### **wraps：为装饰器设计的装饰器**

在创建装饰器时，手动使用`update_wrapper()`可能会变得很繁琐，因此`functools`提供了一个装饰器，用于装饰器本身，称为`wraps`。示例 7-6 展示了`wraps`装饰器的使用。

```py
import functools

def check_is_admin(f):
    @functools.wraps(f)
    def wrapper(*args, **kwargs):
        if kwargs.get('username') != 'admin':
            raise Exception("This user is not allowed to get food")
        return f(*args, **kwargs)
    return wrapper

class Store(object):
    @check_is_admin
    def get_food(self, username, food):
        """Get food from storage."""
        return self.storage.get(food)
```

*示例 7-6：使用 functools 中的 wraps 更新我们的装饰器*

使用`functools.wrap`时，返回`wrapper()`函数的装饰器函数`check_is_admin()`会负责从传入参数`f`的函数中复制文档字符串、名称函数以及其他信息。因此，被装饰的函数（在本例中是`get_food()`）仍然保持不变的签名。

##### **使用 inspect 提取相关信息**

在我们迄今为止的例子中，我们假设被装饰的函数总是会接收到一个作为关键字参数传递的`username`，但实际情况可能并非如此。它可能会接收到一堆信息，我们需要从中提取出`username`来进行检查。考虑到这一点，我们将构建一个更智能的装饰器版本，能够查看被装饰函数的参数并提取所需的信息。

为此，Python 提供了`inspect`模块，它允许我们获取一个函数的签名并对其进行操作，如示例 7-7 所示。

```py
import functools
import inspect

def check_is_admin(f):
    @functools.wraps(f)
    def wrapper(*args, **kwargs):
        func_args = inspect.getcallargs(f, *args, **kwargs)
        if func_args.get('username') != 'admin':
            raise Exception("This user is not allowed to get food")
        return f(*args, **kwargs)
    return wrapper @check_is_admin
def get_food(username, type='chocolate'):
    return type + " nom nom nom!"
```

*示例 7-7：使用 inspect 模块中的工具来提取信息*

执行繁重工作的函数是`inspect.getcallargs()`，它返回一个字典，字典包含了参数的名称和值，以键值对的形式。在我们的示例中，这个函数返回`{'username': 'admin','type': 'chocolate'}`。这意味着我们的装饰器不需要检查`username`参数是位置参数还是关键字参数；装饰器只需要在字典中查找`username`即可。

使用`functools.wraps`和`inspect`模块，你应该能够编写任何自定义装饰器。不过，不要滥用`inspect`模块：虽然能够猜测函数会接受什么样的参数听起来很方便，但这个功能可能是脆弱的，在函数签名发生变化时容易破坏。装饰器是实现开发者所珍视的*不要重复自己（DRY）*原则的绝佳方式。

### **Python 中的方法是如何工作的**

方法的使用和理解相当简单，你很可能已经正确使用了它们，而无需深入探讨。但是，要理解某些装饰器的作用，你需要知道方法背后是如何运作的。

*方法*是作为类属性存储的函数。让我们看看当我们尝试直接访问这样的属性时会发生什么：

```py
>>> class Pizza(object):
...     def __init__(self, size):
...         self.size = size
...     def get_size(self):
...         return self.size
...
>>> Pizza.get_size
<function Pizza.get_size at 0x7fdbfd1a8b90>
```

我们被告知`get_size()`是一个函数——但为什么会这样呢？原因是，在这个阶段，`get_size()`并没有绑定到任何特定的对象。因此，它被当作普通的函数来处理。如果我们直接调用它，Python 会抛出错误，像这样：

```py
>>> Pizza.get_size()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: get_size() missing 1 required positional argument: 'self'
```

Python 会抱怨我们没有提供必要的`self`参数。确实，由于它没有绑定到任何对象，`self`参数无法自动设置。然而，我们不仅可以通过传递类的任意实例来调用`get_size()`函数，如果我们愿意，还可以传递*任何*对象，只要它具备方法所期待的属性。以下是一个示例：

```py
>>> Pizza.get_size(Pizza(42))
42
```

这个调用是有效的，正如所承诺的那样。然而，它并不是很方便：每次我们想调用类的某个方法时，都需要提到类本身。

所以，Python 为我们做了额外的工作，通过将类的方法绑定到其实例上。换句话说，我们可以从任何`Pizza`实例访问`get_size()`，更重要的是，Python 会自动将对象本身传递给方法的`self`参数，像这样：

```py
>>> Pizza(42).get_size
<bound method Pizza.get_size of <__main__.Pizza object at 0x7f3138827910>>
>>> Pizza(42).get_size()
42
```

正如预期的那样，我们不需要向`get_size()`提供任何参数，因为它是一个绑定方法：它的`self`参数会自动设置为我们的`Pizza`实例。这里有一个更清晰的示例：

```py
>>> m = Pizza(42).get_size
>>> m()
42
```

只要你有绑定方法的引用，你甚至不需要保留对`Pizza`对象的引用。此外，如果你有方法的引用，但想知道它绑定到哪个对象，你可以直接检查方法的`__self__`属性，像这样：

```py
>>> m = Pizza(42).get_size
>>> m.__self__
<__main__.Pizza object at 0x7f3138827910>
>>> m == m.__self__.get_size
True
```

显然，我们仍然可以访问我们的对象，如果需要，我们可以找到它。

### **静态方法**

*静态方法*属于类，而不是类的实例，因此它们不会实际操作或影响类的实例。相反，静态方法操作的是它所接受的参数。静态方法通常用于创建实用函数，因为它们不依赖于类或其对象的状态。

例如，在示例 7-8 中，静态`mix_ingredients()`方法属于`Pizza`类，但实际上可以用于混合任何其他食物的配料。

```py
class Pizza(object):
    @staticmethod
    def mix_ingredients(x, y):
        return x + y

    def cook(self):
        return self.mix_ingredients(self.cheese, self.vegetables)
```

*示例 7-8：将静态方法作为类的一部分创建*

如果你愿意，你可以将`mix_ingredients()`编写为一个非静态方法，但它将需要一个`self`参数，而这个参数实际上从未被使用过。使用`@staticmethod`装饰器能为我们带来多个好处。

第一个优点是速度：Python 不必为我们创建的每个`Pizza`对象实例化一个绑定方法。绑定方法本身也是对象，创建它们会有 CPU 和内存的开销——即使开销很小。使用静态方法可以避免这种情况，像这样：

```py
>>> Pizza().cook is Pizza().cook
False
>>> Pizza().mix_ingredients is Pizza.mix_ingredients
True
>>> Pizza().mix_ingredients is Pizza().mix_ingredients
True
```

其次，静态方法提高了代码的可读性。当我们看到`@staticmethod`时，我们就知道该方法不依赖于对象的状态。

第三，静态方法可以在子类中被重写。如果我们使用的是一个在模块顶部定义的`mix_ingredients()`函数，而不是静态方法，那么继承自`Pizza`类的子类就无法改变我们混合比萨配料的方式，除非重写`cook()`方法本身。使用静态方法时，子类可以根据自己的需求重写该方法。

不幸的是，Python 并不总是能够自己检测一个方法是否是静态的——我称之为语言设计的缺陷。一种可能的方法是添加一个检测此模式的检查，并使用`flake8`发出警告。我们将在《通过 AST 检查扩展`flake8`》一章中，查看如何实现这一点，详见第 140 页。

### **类方法**

*类方法*是绑定到类本身，而不是其实例。这意味着这些方法无法访问对象的状态，而只能访问类的状态和方法。示例 7-9 展示了如何编写类方法。

```py
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
>>> Pizza.get_radius is Pizza().get_radius
True
>>> Pizza.get_radius()
42
```

*示例 7-9：将类方法绑定到类上*

如你所见，访问`get_radius()`类方法有多种方式，但无论你选择哪种方式，方法总是绑定到它所属的类上。此外，它的第一个参数必须是类本身。记住：类也是对象！

类方法主要用于创建*工厂方法*，这些方法使用不同于`__init__`的签名来实例化对象：

```py
class Pizza(object):
    def __init__(self, ingredients):
        self.ingredients = ingredients

    @classmethod
    def from_fridge(cls, fridge):
        return cls(fridge.get_cheese() + fridge.get_vegetables())
```

如果我们在这里使用`@staticmethod`而不是`@classmethod`，我们将不得不在方法中硬编码`Pizza`类名，这样任何继承自`Pizza`的类都无法使用我们的工厂方法。可是，在这种情况下，我们提供了一个`from_fridge()`工厂方法，允许我们传入一个`Fridge`对象。如果我们用类似`Pizza.from_fridge(myfridge)`的方式调用该方法，它将返回一个从`myfridge`中获取的全新`Pizza`。

每当你编写一个只关心对象的类而不关心对象状态的方法时，应该将其声明为类方法。

### **抽象方法**

*抽象方法*定义在一个抽象基类中，该基类本身可能不提供任何实现。当一个类有抽象方法时，它无法被实例化。因此，*抽象类*（定义为至少有一个抽象方法的类）必须作为父类被另一个类继承。这个子类将负责实现抽象方法，从而使得可以实例化父类。

我们可以使用抽象基类来明确基类与其他连接类之间的关系，同时使得抽象基类本身无法实例化。通过使用抽象基类，你可以确保从基类派生的类实现基类的特定方法，否则会引发异常。以下示例展示了在 Python 中编写抽象方法的最简单方式：

```py
class Pizza(object):
    @staticmethod
    def get_radius():
        raise NotImplementedError
```

通过这个定义，任何继承自`Pizza`的类都必须实现并覆盖`get_radius()`方法；否则，调用该方法时会抛出如示例中的异常。这对于确保`Pizza`的每个子类都能实现并返回自己的半径计算方法非常有用。

这种实现抽象方法的方式有一个缺点：如果你编写一个继承自`Pizza`的类，但忘记实现`get_radius()`，只有在你尝试在运行时使用该方法时才会抛出错误。下面是一个例子：

```py
>>> Pizza()
<__main__.Pizza object at 0x7fb747353d90>
>>> Pizza().get_radius()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 3, in get_radius
NotImplementedError
```

由于`Pizza`是直接可实例化的，因此无法阻止这种情况的发生。确保在忘记实现和覆盖方法，或者尝试实例化包含抽象方法的对象时收到早期警告的一种方法是使用 Python 的内置`abc`（抽象基类）模块，示例如下：

```py
import abc

class BasePizza(object, metaclass=abc.ABCMeta):

    @abc.abstractmethod
    def get_radius(self):
         """Method that should do something."""
```

`abc`模块提供了一组装饰器，可以用来装饰定义为抽象的那些方法，并且提供一个元类来支持这一点。当你使用`abc`及其特殊的`metaclass`时，如上所示，实例化一个没有覆盖`get_radius()`方法的`BasePizza`类或继承它的类会导致`TypeError`：

```py
>>> BasePizza()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: Can't instantiate abstract class BasePizza with abstract methods
get_radius
```

我们尝试实例化抽象类`BasePizza`，并立即被告知无法执行此操作！

虽然使用抽象方法不能保证方法一定会被用户实现，但这个装饰器帮助你更早地发现错误。当你提供必须由其他开发者实现的接口时，这非常有用；它是一个很好的文档提示。

### **混合静态方法、类方法和抽象方法**

每个装饰器都有其独立的用处，但有时你可能需要将它们一起使用。

例如，你可以将工厂方法定义为类方法，同时强制要求在子类中实现。在这种情况下，你需要将类方法同时定义为抽象方法和类方法。本节提供了一些可以帮助你解决这个问题的技巧。

首先，抽象方法的原型并不是一成不变的。当你实现方法时，并没有什么阻止你根据需要扩展参数列表。示例 7-10 是一个代码示例，其中子类扩展了其父类抽象方法的签名。

```py
import abc

class BasePizza(object, metaclass=abc.ABCMeta):

    @abc.abstractmethod
    def get_ingredients(self):
         """Returns the ingredient list."""

class Calzone(BasePizza):
    def get_ingredients(self, with_egg=False):
        egg = Egg() if with_egg else None
        return self.ingredients + [egg]
```

*示例 7-10：使用子类扩展其父类抽象方法的签名*

我们定义了`Calzone`子类继承自`BasePizza`类。我们可以根据需要定义`Calzone`子类的方法，只要它们支持我们在`BasePizza`中定义的接口。这包括将方法实现为类方法或静态方法。以下代码在基类中定义了一个抽象的`get_ingredients()`方法，并在`DietPizza`子类中定义了一个静态的`get_ingredients()`方法：

```py
import abc

class BasePizza(object, metaclass=abc.ABCMeta):

    @abc.abstractmethod
    def get_ingredients(self):
         """Returns the ingredient list."""

class DietPizza(BasePizza):
    @staticmethod
    def get_ingredients():
        return None
```

即使我们的静态`get_ingredients()`方法并不基于对象的状态返回结果，它仍然支持我们抽象`BasePizza`类的接口，因此它是有效的。

你也可以在`@abstractmethod`上方使用`@staticmethod`和`@classmethod`装饰器，以表示一个方法例如既是静态的又是抽象的，如在示例 7-11 中所示。

```py
import abc

class BasePizza(object, metaclass=abc.ABCMeta):

    ingredients = ['cheese']

    @classmethod
    @abc.abstractmethod
    def get_ingredients(cls):
         """Returns the ingredient list."""
         return cls.ingredients
```

*示例 7-11：在抽象方法中使用类方法装饰器*

抽象方法`get_ingredients()`需要由子类实现，但它也是一个类方法，这意味着它将接收的第一个参数是类（而不是对象）。

请注意，通过像这样在`BasePizza`中将`get_ingredients()`定义为类方法，并不会强制任何子类将`get_ingredients()`定义为类方法——它可以是一个普通方法。如果我们将其定义为静态方法，同样不会强制子类将抽象方法实现为特定类型的方法。正如我们所见，子类在实现抽象方法时可以根据需要改变方法的签名。

#### ***将实现放入抽象方法中***

等等：在 Listing 7-12 中，我们有一个实现 *在* 抽象方法中的例子。我们能*这么做*吗？答案是肯定的。Python 对此没有问题！你可以在抽象方法中写代码，并使用`super()`调用它，正如 Listing 7-12 所示。

```py
import abc

class BasePizza(object, metaclass=abc.ABCMeta):

    default_ingredients = ['cheese']

    @classmethod
    @abc.abstractmethod
    def get_ingredients(cls):
         """Returns the default ingredient list."""
         return cls.default_ingredients

class DietPizza(BasePizza):
    def get_ingredients(self):
        return [Egg()] + super(DietPizza, self).get_ingredients()
```

*Listing 7-12：在抽象方法中使用实现*

在这个例子中，每个继承自`BasePizza`的`Pizza`都必须重写`get_ingredients()`方法，但每个`Pizza`也可以访问基类提供的默认机制来获取配料列表。这个机制在提供接口实现的同时，也提供了可能对所有继承类有用的基础代码，非常有用。

#### ***关于 super 的真相***

Python 一直允许开发者使用单继承和多重继承来扩展他们的类，但即使在今天，许多开发者似乎并不完全理解这些机制，以及与之相关的`super()`方法是如何工作的。要完全理解你的代码，你需要理解这些机制的权衡。

多重继承在许多地方都有使用，尤其是在涉及混入模式的代码中。*混入*是一个继承自两个或更多其他类的类，它结合了这些类的特性。

**注意**

*单继承和多重继承、组合甚至鸭子类型的优缺点超出了本书的范围，因此我们不会在这里讨论所有内容。如果你不熟悉这些概念，我建议你阅读相关资料，形成自己的看法。*

正如你现在应该已经知道的，类在 Python 中是对象。用于创建类的构造是一个你应该非常熟悉的特殊语句：`class classname`（继承的表达式）。

括号中的代码是一个 Python 表达式，它返回用于指定类的父类列表。通常，你会直接指定这些父类，但你也可以像这样写来指定父类对象的列表：

```py
>>> def parent():
...     return object
...
>>> class A(parent()):
...     pass
...
>>> A.mro()
[<class '__main__.A'>, <type 'object'>]
```

这段代码按预期工作：我们声明类`A`，其父类是`object`。类方法`mro()`返回*方法解析顺序*，用于解析属性——它定义了如何通过类之间的继承树找到下一个要调用的方法。当前的 MRO 系统最早在 Python 2.3 中实现，其内部工作原理在 Python 2.3 的发布说明中有所描述。它定义了系统如何浏览类之间的继承树来找到要调用的方法。

我们已经看到，在父类中调用方法的标准方式是使用`super()`函数，但你可能不知道的是，`super()`实际上是一个构造函数，每次调用它时，你都会实例化一个`super`对象。它接受一个或两个参数：第一个参数是一个类，第二个参数是可选的，要么是第一个参数的子类，要么是第一个参数的实例。

构造函数返回的对象充当了第一个参数的父类代理。它有自己的 `__getattribute__` 方法，遍历 MRO（方法解析顺序）列表并返回它找到的第一个匹配属性。当获取 `super()` 对象的属性时，会调用 `__getattribute__` 方法，如 Listing 7-13 所示。

```py
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
```

*Listing 7-13：super() 函数是一个构造函数，用于实例化一个 super 对象。*

当请求 `C` 类实例的 `super` 对象的属性时，`super()` 对象的 `__getattribute__` 方法会遍历 MRO 列表，并返回它找到的第一个包含 `super` 属性的类的属性。

在 Listing 7-13 中，我们传递了两个参数调用了 `super()`，这意味着我们使用了一个 *绑定* 的 `super` 对象。如果我们仅用一个参数调用 `super()`，它将返回一个 *未绑定* 的 `super` 对象：

```py
>>> super(C)
<super: <class 'C'>, NULL>
```

由于没有提供实例作为第二个参数，`super` 对象无法绑定到任何实例。因此，你不能使用这个未绑定的对象来访问类属性。如果你尝试这样做，你将得到以下错误：

```py
>>> super(C).foo
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'super' object has no attribute 'foo'
>>> super(C).bar
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'super' object has no attribute 'bar'
>>> super(C).xyz
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: 'super' object has no attribute 'xyz'
```

乍一看，似乎这个未绑定的 `super` 对象毫无用处，但实际上，`super` 类通过实现描述符协议 `__get__` 的方式，使得未绑定的 `super` 对象可以作为类属性使用：

```py
>>> class D(C):
...     sup = super(C)
...
>>> D().sup
<super: <class 'C'>, <D object>>
>>> D().sup.foo
<bound method D.foo of <__main__.D object at 0x7f0299255bd0>>
>>> D().sup.bar
42
```

未绑定的 `super` 对象的 `__get__` 方法使用实例 `super(C).__get__(D())` 和属性名 `'foo'` 作为参数调用，从而能够找到并解析 `foo`。

**注意**

*即使你从未听说过描述符协议，你很可能已经通过 @property 装饰器使用过它，而你自己并未意识到。描述符协议是 Python 中的一种机制，允许存储为属性的对象返回除了自身之外的其他内容。本书中没有覆盖描述符协议的内容，但你可以在 Python 数据模型文档中找到更多相关信息。*

使用 `super()` 可能会有很多棘手的情况，例如在继承链中处理不同的方法签名。不幸的是，并没有一种万能的解决方案来应对所有场合。最好的预防措施是使用技巧，例如让你的所有方法都接受 `*args, **kwargs` 作为参数。

从 Python 3 开始，`super()` 增添了一些魔法：它现在可以在方法内部不带任何参数地调用。当不向 `super()` 传递任何参数时，它会自动在栈帧中查找参数：

```py
class B(A):
      def foo(self):
          super().foo()
```

访问子类父类属性的标准方式是 `super()`，你应该始终使用它。它可以在不引发意外的情况下协作调用父类方法，避免出现父类方法没有被调用或在多重继承时被调用两次的情况。

### **总结**

凭借你在本章中学到的知识，你应该在涉及 Python 方法定义的所有问题上都能无敌了。装饰器在代码抽象化时至关重要，正确使用 Python 提供的内置装饰器可以极大提升 Python 代码的整洁性。抽象类在为其他开发者和服务提供 API 时尤其有用。

类继承往往不容易完全理解，了解语言内部机制的概览是全面理解其工作原理的好方法。现在这个话题应该没有什么你无法掌握的秘密了！
