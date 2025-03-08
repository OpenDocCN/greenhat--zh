# 9

类

![](img/chapterart.png)

*面向对象编程（OOP）*是编写软件最有效的方法之一。在面向对象编程中，你编写代表现实世界事物和情境的*类*，并基于这些类创建*对象*。当你编写一个类时，你定义了一个整个类别的对象可以拥有的通用行为。

当你从类中创建单独的对象时，每个对象都会自动配备通用的行为；然后你可以为每个对象添加你想要的独特特性。你会惊讶于面向对象编程如何能够很好地模拟现实世界的情况。

从类中创建对象被称为*实例化*，你将与类的*实例*进行工作。在这一章中，你将编写类并创建这些类的实例。你将指定可以存储在实例中的信息类型，并定义可以对这些实例进行的操作。你还将编写扩展现有类功能的类，这样相似的类可以共享通用功能，让你用更少的代码做更多的事情。你将把你的类存储在模块中，并将其他程序员编写的类导入到你自己的程序文件中。

学习面向对象编程将帮助你以程序员的视角来看待世界。它不仅能帮助你理解代码——不仅仅是逐行发生了什么，还能帮助你理解背后的更大概念。了解类的逻辑会训练你进行逻辑思考，从而让你能够编写有效解决几乎所有遇到的问题的程序。

类还使你和其他程序员在面对越来越复杂的挑战时，工作变得更加轻松。当你和其他程序员基于相同的逻辑编写代码时，你们能够相互理解对方的工作。你的程序将对与你合作的人有意义，从而使每个人能够做得更多。

## 创建和使用类

你可以使用类来建模几乎任何事物。让我们从编写一个简单的`Dog`类开始，它代表一只狗——不是特定的某只狗，而是任何一只狗。我们对大多数宠物狗了解什么呢？嗯，它们都有一个名字和年龄。我们还知道大多数狗都会坐下并打滚。这两条信息（名字和年龄）和这两种行为（坐下和打滚）将被放入我们的`Dog`类中，因为它们是大多数狗的共性。这个类将告诉 Python 如何创建一个代表狗的对象。在我们写完这个类后，我们将使用它来创建单独的实例，每个实例代表一只特定的狗。

### 创建狗类

从`Dog`类创建的每个实例将存储一个`name`和一个`age`，我们还将为每只狗提供`sit()`和`roll_over()`的能力：

**dog.py**

```py
❶ class Dog:
    """A simple attempt to model a dog."""

❷     def __init__(self, name, age):
        """Initialize name and age attributes."""
❸         self.name = name
        self.age = age

❹     def sit(self):
        """Simulate a dog sitting in response to a command."""
        print(f"{self.name} is now sitting.")

    def roll_over(self):
        """Simulate rolling over in response to a command."""
        print(f"{self.name} rolled over!")
```

这里有很多需要注意的地方，但不用担心。你将在本章中看到这种结构，并有充足的时间来适应它。我们首先定义了一个名为`Dog`的类❶。按照惯例，首字母大写的名称表示类。在类定义中没有括号，因为我们是从头开始创建这个类。接着，我们编写了一个文档字符串，描述这个类的功能。

### __init__() 方法

属于类的一部分的函数是*方法*。你学到的关于函数的所有知识同样适用于方法；目前唯一的实际区别是我们调用方法的方式。`__init__()`方法❷是一个特殊的方法，Python 会在我们基于`Dog`类创建新实例时自动运行这个方法。这个方法的前后各有两个下划线，这是一个约定，用来防止 Python 默认的方法名称与你的方法名称发生冲突。确保在`__init__()`两边使用两个下划线。如果每边只使用一个，下划线方法就不会在使用类时自动调用，这可能会导致难以发现的错误。

我们定义了`__init__()`方法，包含三个参数：`self`、`name`和`age`。`self`参数在方法定义中是必需的，并且必须排在其他参数之前。它必须包含在定义中，因为当 Python 稍后调用这个方法时（以创建`Dog`的实例），方法调用会自动传入`self`参数。每个与实例相关的调用都会自动传递`self`，它是对实例本身的引用；它让单个实例能够访问类中的属性和方法。当我们创建`Dog`的实例时，Python 将调用`Dog`类的`__init__()`方法。我们将`Dog()`的名称和年龄作为参数传递；`self`会自动传递，所以我们不需要传递它。每当我们想从`Dog`类创建一个实例时，我们只需要为最后两个参数`name`和`age`提供值。

在`__init__()`方法的主体中定义的两个变量都有前缀`self`❸。任何以`self`为前缀的变量在类的每个方法中都可以使用，并且我们还可以通过从该类创建的任何实例来访问这些变量。`self.name = name`这一行将与参数`name`相关联的值赋给变量`name`，然后将其附加到正在创建的实例上。`self.age = age`也做了相同的处理。通过实例可以访问的变量被称为*属性*。

`Dog` 类定义了另外两个方法：`sit()` 和 `roll_over()` ❹。因为这些方法在运行时不需要额外的信息，我们只需定义它们有一个参数，即 `self`。我们稍后创建的实例将能够访问这些方法。换句话说，它们将能够坐下和打滚。现在，`sit()` 和 `roll_over()` 并不会做太多事情。它们仅仅打印一条信息，说明狗狗正在坐下或打滚。但这个概念可以扩展到实际情况：如果这个类是某个电脑游戏的一部分，这些方法将包含代码来让动画中的狗狗坐下或打滚。如果这个类是为了控制机器人编写的，这些方法将指挥机器狗坐下和打滚。

### 从类中创建实例

可以将类看作一套说明，告诉我们如何创建一个实例。`Dog` 类就是一套说明，它告诉 Python 如何创建代表特定狗狗的实例。

让我们创建一个代表特定狗狗的实例：

```py
class Dog:
 *--snip--*

❶ my_dog = Dog('Willie', 6)

❷ print(f"My dog's name is {my_dog.name}.")
❸ print(f"My dog is {my_dog.age} years old.")
```

我们这里使用的 `Dog` 类是我们在前一个示例中编写的类。在这里，我们告诉 Python 创建一只名叫 `'Willie'`、年龄为 `6` ❶ 的狗。当 Python 读取这一行时，它会调用 `Dog` 中的 `__init__()` 方法，传入参数 `'Willie'` 和 `6`。`__init__()` 方法创建一个代表这只特定狗狗的实例，并使用我们提供的值设置 `name` 和 `age` 属性。然后，Python 返回一个代表这只狗的实例，我们将这个实例赋给变量 `my_dog`。命名约定在这里很有帮助；我们通常可以假设像 `Dog` 这样的首字母大写的名称指的是一个类，而像 `my_dog` 这样的全小写名称指的是从类创建的单个实例。

#### 访问属性

要访问实例的属性，使用点符号。我们通过以下方式访问 `my_dog` 的 `name` 属性 ❷ 的值：

```py
my_dog.name
```

点符号在 Python 中使用频繁。这种语法展示了 Python 如何找到一个属性的值。这里，Python 查看实例 `my_dog`，然后找到与 `my_dog` 关联的 `name` 属性。这与在 `Dog` 类中提到的 `self.name` 属性相同。我们用相同的方法来处理 `age` 属性 ❸。

输出是我们关于 `my_dog` 的总结：

```py
My dog's name is Willie.
My dog is 6 years old.
```

#### 调用方法

在我们从 `Dog` 类创建一个实例之后，可以使用点符号调用 `Dog` 中定义的任何方法。让我们让狗狗坐下和打滚：

```py
class Dog:
 *--snip--*

my_dog = Dog('Willie', 6)
my_dog.sit()
my_dog.roll_over()
```

要调用一个方法，给出实例的名称（在本例中是 `my_dog`）和你想调用的方法，中间用点（`.`）隔开。当 Python 读取 `my_dog.sit()` 时，它会在 `Dog` 类中查找 `sit()` 方法并运行该代码。Python 以相同的方式解释 `my_dog.roll_over()` 这一行。

现在，Willie 按照我们的指示行动：

```py
Willie is now sitting.
Willie rolled over!
```

这种语法非常有用。当属性和方法像`name`、`age`、`sit()`和`roll_over()`这样被赋予恰当的描述性名称时，我们可以轻松地推断出即使是我们从未见过的一段代码也应该做什么。

#### 创建多个实例

你可以根据需要从类中创建任意多个实例。让我们创建第二只狗，叫做`your_dog`：

```py
class Dog:
 *--snip--*

my_dog = Dog('Willie', 6)
your_dog = Dog('Lucy', 3)

print(f"My dog's name is {my_dog.name}.")
print(f"My dog is {my_dog.age} years old.")
my_dog.sit()

print(f"\nYour dog's name is {your_dog.name}.")
print(f"Your dog is {your_dog.age} years old.")
your_dog.sit()
```

在这个例子中，我们创建了一只名叫 Willie 的狗和一只名叫 Lucy 的狗。每只狗都是一个独立的实例，拥有自己的一组属性，能够执行相同的一组操作：

```py
My dog's name is Willie.
My dog is 6 years old.
Willie is now sitting.

Your dog's name is Lucy.
Your dog is 3 years old.
Lucy is now sitting.
```

即使我们为第二只狗使用相同的名字和年龄，Python 仍然会从`Dog`类中创建一个单独的实例。你可以根据需要从一个类创建任意多的实例，只要为每个实例赋予一个唯一的变量名，或者它占据列表或字典中的一个唯一位置。

## 使用类和实例

你可以使用类来表示许多现实世界的情况。一旦你编写了一个类，你将大部分时间都花在与从该类创建的实例进行交互上。你首先想要做的任务之一就是修改与特定实例相关的属性。你可以直接修改实例的属性，或者编写方法以特定方式更新属性。

### 汽车类

让我们编写一个表示汽车的新类。我们的类将存储我们正在使用的汽车类型信息，并且它将有一个方法来总结这些信息：

**car.py**

```py
class Car:
    """A simple attempt to represent a car."""

❶     def __init__(self, make, model, year):
        """Initialize attributes to describe a car."""
        self.make = make
        self.model = model
        self.year = year

❷     def get_descriptive_name(self):
        """Return a neatly formatted descriptive name."""
        long_name = f"{self.year} {self.make} {self.model}"
 return long_name.title()

❸ my_new_car = Car('audi', 'a4', 2024)
print(my_new_car.get_descriptive_name())
```

在`Car`类中，我们首先定义`__init__()`方法，并将`self`参数放在最前面❶，就像我们在`Dog`类中所做的那样。我们还为它提供了三个其他参数：`make`、`model`和`year`。`__init__()`方法接收这些参数并将它们分配给将与从此类创建的实例相关联的属性。当我们创建一个新的`Car`实例时，我们需要为我们的实例指定品牌、型号和年份。

我们定义了一个名为`get_descriptive_name()`的方法❷，它将一辆车的`year`、`make`和`model`组合成一个简洁的字符串来描述汽车。这将节省我们逐个打印每个属性值的麻烦。为了在这个方法中操作属性值，我们使用`self.make`、`self.model`和`self.year`。在类外部，我们从`Car`类创建一个实例，并将其赋值给变量`my_new_car`❸。然后我们调用`get_descriptive_name()`方法来显示我们拥有的是什么样的汽车：

```py
2024 Audi A4
```

为了让类更有趣，让我们添加一个随时间变化的属性。我们将添加一个属性，用来存储汽车的总里程。

### 为属性设置默认值

当实例被创建时，属性可以在不作为参数传递的情况下定义。这些属性可以在`__init__()`方法中定义，并为其分配默认值。

让我们添加一个名为`odometer_reading`的属性，它的初始值始终为 0。我们还将添加一个方法`read_odometer()`，帮助我们读取每辆车的里程表：

```py
class Car:

 def __init__(self, make, model, year):
 """Initialize attributes to describe a car."""
 self.make = make
 self.model = model
 self.year = year
❶         self.odometer_reading = 0

    def get_descriptive_name(self):
 *--snip--*

❷     def read_odometer(self):
        """Print a statement showing the car's mileage."""
        print(f"This car has {self.odometer_reading} miles on it.")

my_new_car = Car('audi', 'a4', 2024)
print(my_new_car.get_descriptive_name())
my_new_car.read_odometer()
```

这次，当 Python 调用`__init__()`方法创建新实例时，它将像前一个示例中那样存储制造商、型号和年份作为属性。然后，Python 创建一个名为`odometer_reading`的新属性，并将其初始值设置为 0❶。我们还创建了一个新方法`read_odometer()`❷，它使读取汽车的里程数变得非常容易。

我们的车的初始里程为 0：

```py
2024 Audi A4
This car has 0 miles on it.
```

很少有汽车的里程表上显示完全为 0 的里程，因此我们需要一种方法来更改这个属性的值。

### 修改属性值

你可以通过三种方式修改属性值：可以通过实例直接修改值，或者通过方法设置值，或者通过方法增量修改值（即增加一定的数值）。让我们来看看这三种方法。

#### 直接修改属性值

修改属性值的最简单方法是通过实例直接访问该属性。在这里，我们直接将里程表读数设置为 23：

```py
class Car:
 *--snip--*

my_new_car = Car('audi', 'a4', 2024)
print(my_new_car.get_descriptive_name())

my_new_car.odometer_reading = 23
my_new_car.read_odometer()
```

我们使用点符号访问汽车的`odometer_reading`属性，并直接设置它的值。这行代码告诉 Python，获取实例`my_new_car`，找到与其相关的`odometer_reading`属性，并将该属性的值设置为 23：

```py
2024 Audi A4
This car has 23 miles on it.
```

有时，你可能希望直接访问属性值，但也有时你会希望编写一个方法来更新该值。

#### 通过方法修改属性值

有时，方法可以帮助你更新某些属性。你不需要直接访问属性，而是将新值传递给方法，由方法内部处理更新。

这是一个示例，展示了一个名为`update_odometer()`的方法：

```py
class Car:
 *--snip--*

    def update_odometer(self, mileage):
        """Set the odometer reading to the given value."""
        self.odometer_reading = mileage

my_new_car = Car('audi', 'a4', 2024)
print(my_new_car.get_descriptive_name())

❶ my_new_car.update_odometer(23)
my_new_car.read_odometer()
```

对`Car`类的唯一修改是添加了`update_odometer()`方法。这个方法接受一个里程值并将其赋值给`self.odometer_reading`。通过`my_new_car`实例，我们调用`update_odometer()`并传入`23`作为参数❶。这将把里程表读数设置为 23，接着`read_odometer()`打印出该读数：

```py
2024 Audi A4
This car has 23 miles on it.
```

我们可以扩展`update_odometer()`方法，使其每次修改里程表读数时执行额外的操作。让我们添加一些逻辑，确保没有人试图回滚里程表读数：

```py
class Car:
 *--snip--*

 def update_odometer(self, mileage):
        """
        Set the odometer reading to the given value.
        Reject the change if it attempts to roll the odometer back.
        """
❶         if mileage >= self.odometer_reading:
 self.odometer_reading = mileage
        else:
❷             print("You can't roll back an odometer!")
```

现在`update_odometer()`会在修改属性之前检查新读数是否合理。如果提供的`mileage`值大于或等于现有的里程数`self.odometer_reading`，则可以将里程表读数更新为新的里程数❶。如果新里程数小于现有里程数，系统会发出警告，提示不能回滚里程表❷。

#### 通过方法增加属性值

有时，你可能希望按一定的增量来修改属性值，而不是设置一个全新的值。假设我们购买了一辆二手车，并在购车和注册之间行驶了 100 英里。这里有一个方法，可以让我们传入这个增量并将其添加到里程表读数中：

```py
class Car:
    *--snip--*

 def update_odometer(self, mileage):
 *--snip--*

    def increment_odometer(self, miles):
        """Add the given amount to the odometer reading."""
        self.odometer_reading += miles

❶ my_used_car = Car('subaru', 'outback', 2019)
print(my_used_car.get_descriptive_name())

❷ my_used_car.update_odometer(23_500)
my_used_car.read_odometer()

my_used_car.increment_odometer(100)
my_used_car.read_odometer()
```

新的方法 `increment_odometer()` 接受一个里程数，并将其添加到 `self.odometer_reading`。首先，我们创建一个二手车 `my_used_car` ❶。我们通过调用 `update_odometer()` 并传入 `23_500` ❷，将其里程表设置为 23,500。最后，我们调用 `increment_odometer()` 并传入 `100`，以便将我们从购车到注册这段时间行驶的 100 英里加到里程表上：

```py
2019 Subaru Outback
This car has 23500 miles on it.
This car has 23600 miles on it.
```

你可以修改此方法，以拒绝负数增量，防止有人使用此功能回调里程表。

## 继承

编写类时，你不一定要从头开始。如果你编写的类是你之前写的另一个类的特化版本，你可以使用 *继承*。当一个类 *继承* 另一个类时，它会继承第一个类的属性和方法。原始类称为 *父类*，新类称为 *子类*。子类可以继承父类的部分或全部属性和方法，但它也可以自由地定义自己新的属性和方法。

### 子类的 `__init__()` 方法

当你基于现有类编写新类时，通常会希望调用父类的 `__init__()` 方法。这将初始化父类 `__init__()` 方法中定义的任何属性，并使它们在子类中可用。

举个例子，我们来建模一辆电动汽车。电动汽车只是一种特殊类型的汽车，因此我们可以将新的 `ElectricCar` 类基于我们之前编写的 `Car` 类。这样，我们只需要为电动汽车特有的属性和行为编写代码。

让我们先做一个简单版的 `ElectricCar` 类，它完成 `Car` 类所做的所有工作：

**electric_car.py**

```py
❶ class Car:
 """A simple attempt to represent a car."""

 def __init__(self, make, model, year):
 """Initialize attributes to describe a car.""
 self.make = make
 self.model = model
 self.year = year
 self.odometer_reading = 0

 def get_descriptive_name(self):
 """Return a neatly formatted descriptive name."""
 long_name = f"{self.year} {self.make} {self.model}"
 return long_name.title()

 def read_odometer(self):
 """Print a statement showing the car's mileage."""
 print(f"This car has {self.odometer_reading} miles on it.")

 def update_odometer(self, mileage):
 """Set the odometer reading to the given value."""
 if mileage >= self.odometer_reading:
 self.odometer_reading = mileage
 else:
 print("You can't roll back an odometer!")

 def increment_odometer(self, miles):
 """Add the given amount to the odometer reading."""
 self.odometer_reading += miles

❷ class ElectricCar(Car):
    """Represent aspects of a car, specific to electric vehicles."""

❸     def __init__(self, make, model, year):
        """Initialize attributes of the parent class."""
❹         super().__init__(make, model, year)

❺ my_leaf = ElectricCar('nissan', 'leaf', 2024)
print(my_leaf.get_descriptive_name())
```

我们从 `Car` ❶ 开始。当你创建一个子类时，父类必须位于当前文件中，并且必须出现在子类之前。然后，我们定义了子类 `ElectricCar` ❷。在定义子类时，父类的名称必须包含在括号内。`__init__()` 方法接收创建 `Car` 实例所需的信息 ❸。

`super()` 函数❹是一个特殊的函数，允许你调用父类的方法。这一行代码告诉 Python 调用 `Car` 类中的 `__init__()` 方法，从而为 `ElectricCar` 实例赋予该方法中定义的所有属性。*super* 这个名字来源于一种约定，称父类为 *超类*，子类为 *子类*。

我们通过尝试使用与制作普通汽车时相同的信息创建一辆电动汽车来测试继承是否正常工作。我们创建了一个 `ElectricCar` 类的实例并将其赋值给 `my_leaf` ❺。这一行代码调用了 `ElectricCar` 中定义的 `__init__()` 方法，进而告诉 Python 调用父类 `Car` 中定义的 `__init__()` 方法。我们提供了参数 `'nissan'`、`'leaf'` 和 `2024`。

除了`__init__()`之外，尚未有任何特定于电动汽车的属性或方法。此时我们只是确保电动汽车具有适当的`Car`类行为：

```py
2024 Nissan Leaf
```

`ElectricCar`实例的行为和`Car`实例一样，因此现在我们可以开始定义特定于电动汽车的属性和方法。

### 为子类定义属性和方法

一旦你有了一个从父类继承的子类，你可以添加任何必要的新属性和方法，以便将子类与父类区分开来。

让我们添加一个特定于电动汽车的属性（例如电池）以及一个报告此属性的方法。我们将存储电池大小并编写一个打印电池描述的方法：

```py
class Car:
 *--snip--*

class ElectricCar(Car):
 """Represent aspects of a car, specific to electric vehicles."""

 def __init__(self, make, model, year):
  """
        Initialize attributes of the parent class.
        Then initialize attributes specific to an electric car.
        """
 super().__init__(make, model, year)
❶         self.battery_size = 40

❷     def describe_battery(self):
        """Print a statement describing the battery size."""
        print(f"This car has a {self.battery_size}-kWh battery.")

my_leaf = ElectricCar('nissan', 'leaf', 2024)
print(my_leaf.get_descriptive_name())
my_leaf.describe_battery()
```

我们添加了一个新的属性`self.battery_size`，并将其初始值设置为`40` ❶。这个属性将与从`ElectricCar`类创建的所有实例相关联，但不会与任何`Car`实例相关联。我们还添加了一个名为`describe_battery()`的方法，用于打印电池的信息 ❷。当我们调用这个方法时，会得到一个明确针对电动汽车的描述：

```py
2024 Nissan Leaf
This car has a 40-kWh battery.
```

`ElectricCar`类的专业化没有限制。你可以根据需要添加任意数量的属性和方法，以达到你需要的电动汽车建模精度。任何可以属于所有汽车的属性或方法，而不是特定于电动汽车的，应该添加到`Car`类中，而不是`ElectricCar`类中。这样，使用`Car`类的任何人都可以获得这些功能，而`ElectricCar`类将仅包含与电动汽车特定信息和行为相关的代码。

### 重写父类的方法

你可以重写父类中不适合你在子类中建模的方法。为了做到这一点，你需要在子类中定义一个与父类中要重写的方法同名的方法。Python 会忽略父类的方法，只关注你在子类中定义的方法。

假设`Car`类有一个名为`fill_gas_tank()`的方法。这个方法对于全电动汽车来说没有意义，所以你可能想要重写这个方法。下面是实现这一点的一种方式：

```py
class ElectricCar(Car):
 *--snip--*

    def fill_gas_tank(self):
        """Electric cars don't have gas tanks."""
        print("This car doesn't have a gas tank!")
```

现在，如果有人尝试对电动汽车调用`fill_gas_tank()`，Python 会忽略`Car`类中的`fill_gas_tank()`方法，而运行这段代码。当你使用继承时，你可以让子类保留你需要的内容，并重写你不需要的父类方法。

### 实例作为属性

在用代码建模现实世界中的事物时，你可能会发现自己在不断向一个类中添加更多的细节。你会发现属性和方法的列表越来越长，文件也变得越来越冗长。在这种情况下，你可能会意识到某个类的部分内容可以作为单独的类来编写。你可以将一个大的类拆分为多个相互协作的小类，这种方法叫做*组合*。

例如，如果我们继续向`ElectricCar`类添加细节，可能会注意到我们正在添加许多与汽车电池相关的属性和方法。当我们看到这种情况时，可以停下来，将这些属性和方法移动到一个名为`Battery`的单独类中。然后我们可以在`ElectricCar`类中使用`Battery`实例作为属性：

```py
class Car:
 *--snip--*

class Battery:
    """A simple attempt to model a battery for an electric car."""

❶     def __init__(self, battery_size=40):
 """Initialize the battery's attributes."""
        self.battery_size = battery_size

❷     def describe_battery(self):
        """Print a statement describing the battery size."""
        print(f"This car has a {self.battery_size}-kWh battery.")

class ElectricCar(Car):
 """Represent aspects of a car, specific to electric vehicles."""

 def __init__(self, make, model, year):
 """
 Initialize attributes of the parent class.
 Then initialize attributes specific to an electric car.
 """
 super().__init__(make, model, year)
❸         self.battery = Battery()

my_leaf = ElectricCar('nissan', 'leaf', 2024)
print(my_leaf.get_descriptive_name())
my_leaf.battery.describe_battery()
```

我们定义了一个名为`Battery`的新类，它没有继承任何其他类。`__init__()`方法❶除了`self`之外，还有一个参数`battery_size`。这是一个可选参数，如果没有提供值，则将电池大小设为 40。`describe_battery()`方法也已移到这个类中❷。

在`ElectricCar`类中，我们现在添加了一个名为`self.battery`的属性❸。这一行代码告诉 Python 创建一个新的`Battery`实例（默认大小为 40，因为我们没有指定值），并将该实例分配给属性`self.battery`。每次调用`__init__()`方法时，这个操作都会发生；任何`ElectricCar`实例现在都会自动创建一个`Battery`实例。

我们创建一辆电动汽车，并将其分配给变量`my_leaf`。当我们想描述电池时，我们需要通过汽车的`battery`属性来操作：

```py
my_leaf.battery.describe_battery()
```

这一行代码告诉 Python 查看实例`my_leaf`，找到其`battery`属性，并调用与分配给该属性的`Battery`实例相关联的`describe_battery()`方法。

输出与我们之前看到的完全相同：

```py
2024 Nissan Leaf
This car has a 40-kWh battery.
```

这看起来是多余的工作，但现在我们可以详细描述电池，而不必让`ElectricCar`类变得臃肿。让我们向`Battery`中添加另一个方法，用于报告基于电池大小的汽车续航：

```py
class Car:
 *--snip--*

class Battery:
 *--snip--*

    def get_range(self):
        """Print a statement about the range this battery provides."""
        if self.battery_size == 40:
            range = 150
        elif self.battery_size == 65:
            range = 225

        print(f"This car can go about {range} miles on a full charge.")

class ElectricCar(Car):
 *--snip--*

my_leaf = ElectricCar('nissan', 'leaf', 2024)
print(my_leaf.get_descriptive_name())
my_leaf.battery.describe_battery()
❶ my_leaf.battery.get_range()
```

新方法`get_range()`执行一些简单的分析。如果电池的容量是 40 kWh，`get_range()`将续航设定为 150 英里；如果容量是 65 kWh，则设定为 225 英里。然后，它会报告这个值。当我们想使用这个方法时，我们需要通过汽车的`battery`属性再次调用它❶。

输出告诉我们基于电池大小的汽车续航：

```py
2024 Nissan Leaf
This car has a 40-kWh battery.
This car can go about 150 miles on a full charge.
```

### 真实世界对象建模

当你开始建模更复杂的事物，比如电动汽车时，你会遇到有趣的问题。电动汽车的续航是电池的属性，还是汽车的属性？如果我们只描述一辆车，可能保持`get_range()`方法与`Battery`类的关联是可以的。但如果我们在描述一家制造商的整个汽车系列，我们可能希望将`get_range()`方法移到`ElectricCar`类中。`get_range()`方法仍然会在确定续航之前检查电池大小，但它会报告一个特定于所关联汽车类型的续航。或者，我们可以保持`get_range()`方法与电池的关联，但传递一个像`car_model`这样的参数给它。`get_range()`方法然后会根据电池大小和汽车模型报告续航。

这带你进入作为程序员成长的一个有趣点。当你与这些问题作斗争时，你是在以更高的逻辑层次思考，而不是专注于语法层面。你思考的不是 Python，而是如何用代码表示现实世界。当你达到这个层次时，你会意识到，建模现实世界情境时，通常没有对错之分。一些方法比其他方法更高效，但要找到最有效的表示方法需要实践。如果你的代码按照你希望的方式运行，那么你做得很好！如果你发现自己不断拆解类并用不同的方法重写它们，不要灰心。每个人在写出准确、高效的代码的过程中都会经历这一过程。

## 导入类

当你为类添加更多功能时，即使在正确使用继承和组合的情况下，文件也可能变得很长。为了符合 Python 的整体哲学，你会希望保持文件尽可能简洁。为了帮助实现这一点，Python 允许你将类存储在模块中，然后将需要的类导入到主程序中。

### 导入单个类

让我们创建一个只包含`Car`类的模块。这引出了一个微妙的命名问题：在这一章中，我们已经有一个名为*car.py*的文件，但这个模块应该命名为*car.py*，因为它包含了代表汽车的代码。我们将通过将`Car`类存储在名为*car.py*的模块中来解决这个命名问题，取代我们之前使用的*car.py*文件。从现在起，任何使用这个模块的程序都需要一个更具体的文件名，比如*my_car.py*。以下是仅包含`Car`类代码的*car.py*：

**car.py**

```py
❶ """A class that can be used to represent a car."""

class Car:
 """A simple attempt to represent a car."""

 def __init__(self, make, model, year):
 """Initialize attributes to describe a car."""
 self.make = make
 self.model = model
 self.year = year
 self.odometer_reading = 0

 def get_descriptive_name(self):
 """Return a neatly formatted descriptive name."""
 long_name = f"{self.year} {self.make} {self.model}"
 return long_name.title()

 def read_odometer(self):
 """Print a statement showing the car's mileage."""
 print(f"This car has {self.odometer_reading} miles on it.")

 def update_odometer(self, mileage):
 """
 Set the odometer reading to the given value.
 Reject the change if it attempts to roll the odometer back.
 """
 if mileage >= self.odometer_reading:
 self.odometer_reading = mileage
 else:
 print("You can't roll back an odometer!")

 def increment_odometer(self, miles):
 """Add the given amount to the odometer reading."""
 self.odometer_reading += miles
```

我们在模块级别包含一个文档字符串，简要描述该模块的内容 ❶。你应该为你创建的每个模块编写文档字符串。

现在我们创建一个单独的文件，名为*my_car.py*。这个文件将导入`Car`类，然后从这个类创建一个实例：

**my_car.py**

```py
❶ from car import Car

my_new_car = Car('audi', 'a4', 2024)
print(my_new_car.get_descriptive_name())

my_new_car.odometer_reading = 23
my_new_car.read_odometer()
```

`import`语句❶告诉 Python 打开`car`模块并导入`Car`类。现在我们可以像在此文件中定义该类一样使用`Car`类。输出与我们之前看到的相同：

```py
2024 Audi A4
This car has 23 miles on it.
```

导入类是一种有效的编程方式。想象一下，如果将整个`Car`类包含在程序文件中，这个文件会有多长。相反，如果将类移动到模块中并导入该模块，你仍然可以获得相同的功能，但能保持主程序文件清晰易读。你还将大部分逻辑存储在单独的文件中；一旦你的类按照预期工作，你可以将这些文件保留不动，专注于主程序的更高层逻辑。

### 在一个模块中存储多个类

你可以在一个模块中存储尽可能多的类，尽管每个类应该以某种方式相关。`Battery`类和`ElectricCar`类都帮助表示汽车，因此我们将它们添加到模块*car.py*中。

**car.py**

```py
"""A set of classes used to represent gas and electric cars."""

class Car:
 *--snip--*

class Battery:
 """A simple attempt to model a battery for an electric car."""

 def __init__(self, battery_size=40):
 """Initialize the battery's attributes."""
 self.battery_size = battery_size

 def describe_battery(self):
 """Print a statement describing the battery size."""
 print(f"This car has a {self.battery_size}-kWh battery.")

 def get_range(self):
 """Print a statement about the range this battery provides."""
 if self.battery_size == 40:
 range = 150
 elif self.battery_size == 65:
 range = 225

 print(f"This car can go about {range} miles on a full charge.")

class ElectricCar(Car):
 """Models aspects of a car, specific to electric vehicles."""

 def __init__(self, make, model, year):
 """
 Initialize attributes of the parent class.
 Then initialize attributes specific to an electric car.
 """
 super().__init__(make, model, year)
 self.battery = Battery()
```

现在我们可以创建一个新文件，命名为*my_electric_car.py*，导入`ElectricCar`类并制造一辆电动汽车：

**my_electric_car.py**

```py
from car import ElectricCar

my_leaf = ElectricCar('nissan', 'leaf', 2024)
print(my_leaf.get_descriptive_name())
my_leaf.battery.describe_battery()
my_leaf.battery.get_range()
```

即使大部分逻辑被隐藏在模块中，这仍然输出我们之前看到的结果：

```py
2024 Nissan Leaf
This car has a 40-kWh battery.
This car can go about 150 miles on a full charge.
```

### 从模块中导入多个类

你可以将所需的多个类导入到程序文件中。如果我们想在同一个文件中制作一辆常规汽车和一辆电动汽车，我们需要导入`Car`类和`ElectricCar`类：

**my_cars.py**

```py
❶ from car import Car, ElectricCar

❷ my_mustang = Car('ford', 'mustang', 2024)
print(my_mustang.get_descriptive_name())
❸ my_leaf = ElectricCar('nissan', 'leaf', 2024)
print(my_leaf.get_descriptive_name())
```

你可以通过用逗号❶分隔每个类来从模块中导入多个类。一旦导入了必要的类，你可以根据需要创建每个类的任意数量的实例。

在这个示例中，我们创建了一辆汽油驱动的福特野马❷，然后是电动的日产聆风❸：

```py
2024 Ford Mustang
2024 Nissan Leaf
```

### 导入整个模块

你也可以导入整个模块，然后使用点符号访问所需的类。这种方法简单，代码也容易阅读。因为每次创建类实例的调用都包括模块名称，所以你不会与当前文件中使用的任何名称发生命名冲突。

下面是导入整个`car`模块并创建一辆常规汽车和一辆电动汽车的代码：

**my_cars.py**

```py
❶ import car

❷ my_mustang = car.Car('ford', 'mustang', 2024)
print(my_mustang.get_descriptive_name())

❸ my_leaf = car.ElectricCar('nissan', 'leaf', 2024)
print(my_leaf.get_descriptive_name())
```

首先，我们导入整个`car`模块❶。然后，我们通过`module_name`.`ClassName`语法访问所需的类。我们再次创建一辆福特野马❷和一辆日产聆风❸。

### 从模块中导入所有类

你可以使用以下语法导入模块中的每个类：

```py
from `module_name` import *
```

这种方法不推荐使用，原因有两个。首先，能够阅读文件顶部的`import`语句并清楚了解程序使用了哪些类是很有帮助的。采用这种方法时，模块中你使用了哪些类并不清晰。这个方法也可能导致文件中的命名混乱。如果你不小心导入了一个与程序文件中其他内容同名的类，可能会产生难以诊断的错误。我之所以在这里展示这种方法，是因为尽管它并不推荐，但你可能会在其他人的代码中看到它。

如果你需要从一个模块导入多个类，最好导入整个模块，并使用`module_name.ClassName`语法。虽然你在文件顶部看不到所有类，但你会清楚地看到模块在程序中的使用位置。而且，你也能避免导入模块中每个类时可能出现的命名冲突。

### 将模块导入到模块中

有时候，你可能希望将类分散到多个模块中，以防某个文件过大，同时避免将不相关的类存储在同一个模块中。当你将类存储在多个模块中时，可能会发现一个模块中的类依赖于另一个模块中的类。在这种情况下，你可以将所需的类导入到第一个模块中。

例如，我们将`Car`类存储在一个模块中，将`ElectricCar`和`Battery`类存储在另一个模块中。我们将创建一个新模块，叫做*electric_car.py*——替换我们之前创建的*electric_car.py*文件——并将`Battery`和`ElectricCar`类仅复制到这个文件中：

**electric_car.py**

```py
"""A set of classes that can be used to represent electric cars."""

from car import Car

class Battery:
 *--snip--*

class ElectricCar(Car):
 *--snip--*
```

`ElectricCar`类需要访问它的父类`Car`，因此我们直接将`Car`导入模块。如果我们忘记了这行代码，当尝试导入`electric_car`模块时，Python 会抛出错误。我们还需要更新`Car`模块，使其只包含`Car`类：

**car.py**

```py
"""A class that can be used to represent a car."""

class Car:
 *--snip--*
```

现在我们可以分别从每个模块导入，并创建任何我们需要的汽车：

**my_cars.py**

```py
from car import Car
from electric_car import ElectricCar

my_mustang = Car('ford', 'mustang', 2024)
print(my_mustang.get_descriptive_name())

my_leaf = ElectricCar('nissan', 'leaf', 2024)
print(my_leaf.get_descriptive_name())
```

我们从它的模块导入`Car`，并从它的模块导入`ElectricCar`。然后我们创建一辆普通汽车和一辆电动汽车。两辆车都创建得正确无误：

```py
2024 Ford Mustang
2024 Nissan Leaf
```

### 使用别名

正如你在第八章看到的，使用模块时，别名非常有帮助，能帮助你组织项目代码。你也可以在导入类时使用别名。

举个例子，假设你有一个程序，你需要制造一堆电动汽车。反复输入（和读取）`ElectricCar`可能会变得很繁琐。你可以在导入语句中为`ElectricCar`起个别名：

```py
from electric_car import ElectricCar as EC
```

现在每当你想制造电动汽车时，你可以使用这个别名：

```py
my_leaf = EC('nissan', 'leaf', 2024)
```

你还可以为模块指定别名。以下是如何使用别名导入整个`electric_car`模块的方法：

```py
import electric_car as ec
```

现在你可以通过模块别名来使用完整的类名：

```py
my_leaf = ec.ElectricCar('nissan', 'leaf', 2024)
```

### 寻找自己的工作流程

正如你所看到的，Python 为大型项目提供了多种代码结构选项。了解这些选项非常重要，这样你就能确定最佳的项目组织方式，同时也能理解他人的项目。

当你刚开始编写代码时，保持代码结构简单。尝试将所有内容写在一个文件中，等一切正常后，再将类移到单独的模块中。如果你喜欢模块和文件之间的交互，试着在开始一个项目时将类存储在模块中。找到一种能让你编写有效代码的方法，并从那里出发。

## Python 标准库

*Python 标准库* 是随每个 Python 安装一起提供的一组模块。现在你已经基本理解了函数和类是如何工作的，你可以开始使用其他程序员编写的类似模块。你可以通过在文件顶部添加简单的`import`语句来使用标准库中的任何函数或类。我们来看一个模块，`random`，它在模拟许多现实世界的情况时非常有用。

来自`random`模块的一个有趣函数是`randint()`。该函数接受两个整数参数，并返回这两个数字之间（包括这两个数字）的随机整数。

下面是如何生成 1 到 6 之间的随机数：

```py
>>> **from random import randint**
>>> **randint(1, 6)**
3
```

另一个有用的函数是`choice()`。这个函数接受一个列表或元组，并返回一个随机选择的元素：

```py
>>> **from random import choice**
>>> **players = ['charles', 'martina', 'michael', 'florence', 'eli']**
>>> **first_up = choice(players)**
>>> **first_up**
'florence'
```

`random`模块不应在构建安全相关的应用程序时使用，但它在许多有趣的项目中表现良好。

## 样式类

关于类的几个样式问题值得澄清，特别是当你的程序变得更加复杂时。

类名应使用*驼峰命名法（CamelCase）*。为此，将每个单词的首字母大写，且不要使用下划线。实例和模块名应使用小写字母，并在单词之间使用下划线。

每个类在类定义之后应有一个文档字符串。文档字符串应简短描述类的功能，你应该遵循编写函数文档字符串时使用的相同格式约定。每个模块也应有一个文档字符串，描述模块中的类的用途。

你可以使用空行来组织代码，但不要过度使用。在一个类内部，你可以在方法之间使用一行空行，而在模块中，你可以使用两行空行来分隔类。

如果你需要导入标准库中的模块和你自己编写的模块，应先导入标准库模块的语句。然后添加一个空行，再导入你自己编写模块的语句。在有多个导入语句的程序中，这种约定能更容易地看到程序中使用的不同模块的来源。

## 概述

在本章中，你学习了如何编写自己的类。你学习了如何使用属性在类中存储信息，以及如何编写方法来赋予类所需的行为。你学习了如何编写`__init__()`方法，通过该方法可以根据你需要的属性创建类的实例。你了解了如何直接通过实例修改属性，也学会了通过方法来修改属性。你学会了继承可以简化创建相关类的过程，并且学会了如何使用一个类的实例作为另一个类的属性，从而保持每个类的简洁性。

你了解了如何将类存储在模块中，并将所需的类导入到使用它们的文件中，这样可以保持项目的组织性。你开始学习 Python 标准库，并且通过`random`模块的示例看到了它的应用。最后，你学会了如何使用 Python 的约定来给类添加风格。

在第十章中，你将学习如何处理文件，这样你就可以保存你在程序中完成的工作以及你允许用户执行的操作。你还将了解*异常*，这是一种特殊的 Python 类，旨在帮助你在错误发生时做出响应。
