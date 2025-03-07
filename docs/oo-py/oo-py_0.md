列表的工作方式类似：

>>> myList = [10, 20, 30, 40]

>>> print(type(myList))

<class 'list'>

所有列表都是列表类的实例，该类具有许多方法，包括

例如myList.append()、myList.count()、myList.index()等等。

当你编写一个类时，你实际上是在定义一个新的数据类型。你的代码

通过定义它维护的数据和执行的操作来提供细节。

它可以执行的操作。在创建类的实例并将其分配给

一个变量，你可以使用type()内建函数来确定该变量的类

像使用内建数据类型一样，用它创建一个对象。这里我们实例化了一个

LightSwitch 对象并输出其数据类型：

>>> oLightSwitch = LightSwitch()

>>> print(type(oLightSwitch))

<class 'LightSwitch'>

**32** 第二章

就像Python的内建数据类型一样，我们可以用变量oLightSwitch来调用oLightSwitch类中可用的方法。

***对象的定义***

为了总结这一部分，我将给出我对*对象*的正式定义。

**对象**

数据，以及随着时间推移对这些数据进行操作的代码。

一个类定义了实例化时对象的外观。

一个对象是一组实例变量和方法代码，

类，从中实例化对象。可以实例化任意数量的对象

可以从一个类实例化，每个对象都有自己的一组实例变量。

当你调用对象的方法时，该方法会运行，并使用该对象中的一组

该对象中的实例变量。

**构建一个稍微复杂的类**

让我们在已有概念的基础上继续构建，并通过一个稍微复杂的例子，

稍微复杂的例子，我们将做一个调光开关

类。一个调光开关有一个开/关开关，但它还有一个多位置

影响光亮度的滑块。

这个滑块可以在一系列亮度值中移动。为了让

为了简化事情，我们的调光数字滑块有11个位置，从0

（完全关闭）到10（完全开启）。要调高或调低亮度，

要将灯泡的亮度调至最大限度，你必须将滑块调至

所有可能的设置。

这个DimmerSwitch类比我们的LightSwitch类具有更多功能。

并且需要记住更多的数据：

• 开关状态（开或关）

• 亮度级别（0到10）

以下是DimmerSwitch对象可以执行的行为：

• 打开

• 关闭

• 提高亮度

• 低级别

• 显示（用于调试）

DimmerSwitch 类使用前面展示的标准模板

列表 2-2：它以一个类声明开始，并有一个名为的第一个方法

__init__()，然后定义了一些额外的方法，每个方法对应

所列的行为。这个类的完整代码在列表2-5中给出。

用面向对象编程建模物理对象 **33**

**文件：DimmerSwitch.py**

# DimmerSwitch 类

class DimmerSwitch():

def __init__(self):

self.switchIsOn = False

self.brightness = 0

def turnOn(self):

self.switchIsOn = True

def turnOff(self):

self.switchIsOn = False

def raiseLevel(self):

如果 self.brightness < 10：

self.brightness = self.brightness + 1

def lowerLevel(self):

如果 self.brightness > 0：

self.brightness = self.brightness - 1

# 用于调试的额外方法

def show(self):

print(开关是否开启？', self.switchIsOn)

print('亮度为：', self.brightness)

*列表 2-5：稍微复杂一些的 DimmerSwitch 类*

在这个__init__()方法中，我们有两个实例变量：家庭-

变量实例维护它们在方法调用之间的值

亮度级别。我们为两个实例变量分配了初始值。所有

其他方法可以访问这些的当前值。此外

为了实现 turnOn() 和 turnOff()，我们为这个类添加了两个新方法：

raiseLevel() 和 lowerLevel()，它们的作用正如名称所示。

show()方法在开发和调试过程中使用，直接

打印实例变量的当前值。

列表 2-6 中的主代码通过创建一个 DimmerSwitch 来测试我们的类

对象（oDimmer），然后调用各种方法。

**文件：OO_DimmerSwitch_with_Test_Code.py**

# 带有测试代码的 DimmerSwitch 类

类 DimmerSwitch():

--- 删除的 DimmerSwitch 类代码，如列表 2-5 所示 ---

# 主代码

oDimmer = DimmerSwitch()

# Turn 开关开启，并提升级别5次

**第2章 第34页**

oDimmer.turnOn()

oDimmer.raiseLevel()

oDimmer.raiseLevel()

oDimmer.raiseLevel()

oDimmer.raiseLevel()

oDimmer.raiseLevel()

oDimmer.show()

# 降低级别2次，并关闭开关

oDimmer.lowerLevel()

oDimmer.lowerLevel()

oDimmer.turnOff()

oDimmer.show()

# 打开开关，并提升级别3次

oDimmer.turnOn()

oDimmer.raiseLevel()

oDimmer.raiseLevel()

oDimmer.raiseLevel()

oDimmer.show()

*列表 2-6：带有测试代码的 DimmerSwitch 类*

当我们运行这段代码时，输出结果是：

开关是否开启？ True

亮度为：5

开关是否开启？ False

亮度为：3

开关是否开启？ True

亮度为：6

主代码创建了 oDimmer 对象，然后调用了多个

其他方法。每次调用 show() 方法时，开/关状态和

亮度级别被打印。这里需要记住的关键点是 oDimmer

代表一个对象。它允许访问类中的所有方法

它被实例化（DimmerSwitch 类），*并且*它拥有在类中定义的所有实例变量（self.switchIsOn 和 self.brightness）。再次，

实例变量在方法调用之间保持它们的值

对象，所以 self.brightness 实例变量每次增加1

oDimmer.raiseLevel()

**将更复杂的物理对象表示为类**

让我们考虑一个更复杂的物理对象：电视机。对于这个

在这个更复杂的示例中，我们将更深入地了解参数的工作原理

在类中。

电视机需要比灯光开关更多的数据来表示

它的状态，并且有更多的行为。为了创建一个电视类，我们必须考虑

用户通常如何使用电视以及电视需要记住什么

让我们来看一下典型电视遥控器上的一些重要按钮

（图 2-4）。

使用面向对象编程建模物理对象 **35**

电源

音量

频道

静音

获取信息

1

2

3

4

5

6

7

8

9

0

*图 2-4：简化版电视遥控器*

从中我们可以得出结论，要跟踪其状态，TV 类

必须维护以下数据：

• 电源状态（开或关）

• 静音状态（是否静音？）

• 可用频道列表

• 当前频道设置

• 当前音量设置

• 可用音量级别范围

电视必须提供的操作包括：

• 开关电源

• 调节音量

• 切换频道

• 静音和取消静音

• 获取当前设置的信息

• 转到指定频道

**36** 第二章

我们的 TV 类代码显示在清单 2-7 中。我们包括了用于初始化的 __init__() 方法，接下来是每个行为的方法。

**文件：TV.py**

# TV 类

class TV():

def __init__(self): 1

self.isOn = False

self.isMuted = False

# 一些默认的频道列表

self.channelList = [2, 4, 5, 7, 9, 11, 20, 36, 44, 54, 65]

self.nChannels = len(self.channelList)

self.channelIndex = 0

self.VOLUME_MINIMUM = 0 # 常量

self.VOLUME_MAXIMUM = 10 # 常量

self.volume = self.VOLUME_MAXIMUM // # 整数除法

def power(self): 2

self.isOn = not self.isOn # 切换状态

def volumeUp(self):

如果 not self.isOn:

返回

如果 self.isMuted:

self.isMuted = False # 在静音状态下改变音量会取消静音

如果 self.volume < self.VOLUME_MAXIMUM:

self.volume = self.volume + 1

def volumeDown(self):

如果 not self.isOn:

返回

如果 self.isMuted:

self.isMuted = False # 在静音状态下改变音量会取消静音

如果 self.volume > self.VOLUME_MINIMUM:

self.volume = self.volume - 1

def channelUp(self): 3

如果 not self.isOn:

返回

self.channelIndex = self.channelIndex + 1

如果 self.channelIndex > self.nChannels:

self.channelIndex = 0 # 回绕到第一个频道

def channelDown(self): 4

如果 not self.isOn:

返回

self.channelIndex = self.channelIndex - 1

如果 self.channelIndex < 0:

self.channelIndex = self.nChannels - 1 # 回绕到第一个频道

def mute(self): 5

如果 not self.isOn:

使用面向对象编程建模物理对象 **37**

返回

self.isMuted = not self.isMuted

def setChannel(self, newChannel):

如果 newChannel 在 self.channelList 中:

self.channelIndex = self.channelList.index(newChannel)

# 如果 newChannel 不在我们的频道列表中，则不执行任何操作

def showInfo(self): 6

print()

print('电视状态：')

如果 self.isOn:

print(' 电视状态：开')

print(' 当前频道为:', self.channelList[self.channelIndex])

如果 self.isMuted:

print(' 音量为:', self.volume, '(声音已静音)')

否则：

print(' 音量为:', self.volume)

否则：

print(' 电视状态：关闭')

*清单 2-7：具有多个实例变量和方法的 TV 类*

__init__() 方法 1 创建了所有在各个方法中使用的实例变量。

方法并为每个方法分配合理的初始值。从技术上讲，你可以

在任何方法内创建一个实例变量；然而，这是一个良好的编程实践——

在 __init__() 方法中定义所有实例变量是一种良好的编程实践。

避免在尝试使用实例变量时出现错误的风险

方法在定义之前使用。

power() 方法 2 表示按下遥控器上的电源按钮时发生的情况。

如果电视关闭，按下电源按钮将其打开；

打开电视；如果电视已经开启，按下电源按钮将其关闭。要编码此功能，

我使用了一个*切换*，它是一个布尔值，用于表示两个状态之一，可以轻松地在它们之间切换。有了这个切换，

非运算符将 self.isOn 变量从 True 切换到 False，或从 False

将值设为 True。mute() 方法代码 5 执行类似的操作，使用 self.muted

变量在静音和非静音之间切换，但首先必须检查

电视是开启的。如果电视关闭，调用 mute() 方法没有任何效果。

一个有趣的事情是，我们并没有真正跟踪

当前频道。相反，我们跟踪当前频道的*索引*，

频道，它允许我们随时通过使用

self.channelList[self.channelIndex]。

channelUp() 3 和 channelDown() 4 方法基本上增加和

递减频道索引，但它们也有一些巧妙的代码来

允许环绕。如果你当前在频道列表的最后一个索引

用户要求切换到下一个频道时，电视会跳转到频道列表中的第一个频道。

如果你当前在频道列表的第一个索引并且用户要求切换到下一个频道，那么电视将跳转到频道列表中的最后一个频道。

showInfo() 方法 6 输出基于实例变量的电视当前状态（开/关，当前频道，当前

基于实例变量的值（开/关，当前频道，当前频道，当前

音量设置和静音设置）。

**38** 第2章

在示例 2-8 中，我们将创建一个电视对象并调用该对象的方法。

**文件：OO_TV_with_Test_Code.py**

# 带测试代码的电视类

--- 剪辑自电视类代码，如示例 2-7 ---

# 主代码

oTV = TV()  # 创建电视对象

# 打开电视并显示状态

oTV.power()

oTV.showInfo()

# 将频道提升两次，将音量提升两次，显示状态

oTV.channelUp()

oTV.channelUp()

oTV.volumeUp()

oTV.volumeUp()

oTV.showInfo()

# 关闭电视，显示状态，打开电视，显示状态

oTV.power()

oTV.showInfo()

oTV.power()

oTV.showInfo()

# 降低音量，静音，显示状态

oTV.volumeDown()

oTV.mute()

oTV.showInfo()

# 将频道更改为 11，静音，显示状态

oTV.setChannel(11)

oTV.mute()

oTV.showInfo()

*示例 2-8：带测试代码的电视类*

当我们运行此代码时，得到的输出如下：

电视状态：

电视是：开启

频道是：2

音量是：5

电视状态：

电视是：开启

频道是：5

音量是：7

电视状态：

电视是：关闭

使用面向对象编程建模物理对象 **39**

电视状态：

电视是：开启

频道是：5

音量是：7

电视状态：

电视是：开启

频道是：5

音量是：6（声音已静音）

电视状态：

电视是：开启

当前频道是：11

音量是：6

所有方法都正常工作，我们得到了预期的

输出。

***将参数传递给方法***

调用任何函数时，参数的数量必须与定义中的参数数量匹配。

匹配的 def 语句中列出的参数数量：

def myFunction(param1, param2, param3):

# 函数体

# 调用一个函数：

myFunction(argument1, argument2, argument3)

相同的规则适用于方法和方法调用。然而，你

你可能会注意到，每当我们调用一个方法时，似乎我们正在

指定比参数数量少一个参数。例如，

我们的 TV 类中 power() 方法的定义如下：

def power(self):

这意味着 power() 方法期望传入一个值

传递进来的值会被分配给变量 self。然而，

当我们在列表 2-8 中开启电视时，我们进行了这个调用：

oTV.power()

当我们调用时，我们没有明确地在括号中传递任何东西

括号。

在 setChannel() 方法的情况下，这可能看起来更奇怪。

该方法被写成接受两个参数：

def setchannel(self, newchannel):

如果 newChannel 在 self.channelList 中：

self.channelIndex = self.channelList.index(newChannel)

**40** 第 2 章

但我们是这样调用 setChannel() 的：

oTV.setChannel(11)

看起来似乎只传递了一个值。

你可能期望 Python 在这里生成错误，因为参数数量不匹配

在参数的数量（一个）和定义中的参数（两个）之间的不匹配。在

实际上，Python 会做一些幕后工作，使语法更加简洁。

更容易理解。

我们来看看这个。之前我说过，要调用一个对象的方法，

对象时，使用以下通用语法：

*<对象>* . *<方法>*（*<任何参数>*）

Python 会接收你在调用中指定的<object>，并将其重新排列为

变成第一个参数。方法括号中的任何值

调用中的参数被视为后续的参数。因此，Python 会使其

似乎你写的是这个：

*<方法名>*（*<对象>*，*<任何参数>*）图 2-5 展示了这一点在我们示例代码中的工作方式，仍然使用

setChannel() 方法属于 TV 类。

# TV 类中的方法

**def setChannel(self, newChannel):**

**…**

#调用

**oTV.setChannel(**

**V**

**11）**

*图 2-5：调用方法*

尽管看起来我们这里只提供了一个参数（对于

newChannel），实际上传递了两个参数——oTV 和 11——并且

方法提供两个参数来接收这些值（self 和 newChannel，

Python 会为我们重新排列参数，分别）。Python 会为我们处理调用中的参数。

起初可能会觉得奇怪，但很快你会发现这是第二天性。

很快。将对象放在前面写调用使得程序员更容易看出操作的是哪个对象。

程序员可以很容易地看出操作的是哪个对象。

这是一个微妙但重要的特性。记住，对象（在

在这种情况下，oTV 保持其所有实例变量的当前设置。

将对象作为第一个参数传递，使方法能够使用

该对象的实例变量的值。

***多个实例***

每个方法的第一个参数都是 self，所以 self 变量

每次调用时都会接收到使用的对象。这有一个重要的含义：它允许

使用面向对象编程建模物理对象 **41**

类中的任何方法都可以使用 *不同* 的对象来工作。我将通过示例来解释这一点。

在清单 2-9 中，我们将创建两个 TV 对象并将它们保存到两个变量中，

oTV1 和 oTV2。每个 TV 对象都有音量设置、频道列表、频道

设置等。我们将调用多个不同方法来操作

不同的对象。最后，我们会在每个 TV 上调用 showInfo() 方法

查看对象的设置。

**文件：OO_TV_TwoInstances.py**

# 两个 TV 对象以及对它们方法的调用

class TV():

--- 截略 TV 类代码，如清单 2-7 ---

# 主代码

oTV1 = TV() # 创建一个 TV 对象

oTV2 = TV() # 创建另一个 TV 对象

# 打开两个电视

oTV1.power()

oTV2.power()

# 抬高 TV1 的音量

oTV1.volumeUp()

oTV1.volumeUp()

# 抬高 TV2 的音量

oTV2.volumeUp()

oTV2.volumeUp()

oTV2.volumeUp()

oTV2.volumeUp()

oTV2.volumeUp()

# 更改 TV2 的频道，然后将其静音

oTV2.setChannel(44)

oTV2.mute()

# 现在显示两个电视

oTV1.showInfo()

oTV2.showInfo()

*清单 2-9：创建 TV 类的两个实例并调用每个实例的方法* 如果我们运行这段代码，它将生成以下输出：

TV 状态：

TV 状态：开

频道是：2

音量是：7

TV 状态：

TV 状态：开

频道是：44

音量是：10（音量已静音）

**42** 第二章

每个 TV 对象都维护着自己在类中定义的实例变量。这样，每个 TV 对象的实例变量可以被单独操作

与其他任何 TV 对象的实例变量独立。

***初始化参数***

向方法调用传递参数的能力在实例化时同样有效—

使用对象时传递的参数。到目前为止，当我们创建对象时，总是设置它们的

实例变量的值设置为常量值。然而，你通常会希望创建

不同实例有不同的初始值。例如，假设我们

想要实例化不同的电视并通过它们的品牌名进行区分

和位置。这样，我们可以区分一台三星电视

在家庭房间使用一台电视，在卧室使用一台索尼电视。常量值

在这种情况下，我们的方法将不起作用。

为了用不同的值初始化对象，我们为

__init__() 方法的定义，如下所示：

# TV 类

class TV():

def __init__(self, brand, location): # 为电视传递品牌和位置

self.brand = brand

self.location = location

--- 截略 TV 初始化的其余部分 ---

...

在所有方法中，参数是局部变量，所以它们会在方法结束时消失

当方法结束时。例如，在 TV 类的 __init__() 方法中

这里展示的类，品牌和位置是局部变量，最终会消失

当方法结束时，通常我们希望保存那些通过参数传递进来的值，

通过参数传入的值，用于在其他方法中使用它们。

为了让对象记住初始值，标准做法是

这种方法是将任何传入的值存储到实例变量中。因为

实例变量具有对象作用域，它们可以在类中的其他方法中使用

这个类。Python的约定是实例变量的名称

应该与参数名相同，但前面加上self和一个

句点：

def __init__(self, someVariableName):

self.someVariableName = someVariableName

在电视类中，def语句之后的一行告诉Python去获取

将品牌参数的值分配给名为

self.brand。下一行做的事情与位置参数相同

以及实例变量self.location。完成这些赋值后，我们可以使用

self.brand和self.location可以在其他方法中使用。

用面向对象编程建模物理对象 **43**

使用这种方法，我们可以从同一个类创建多个对象，但每个对象都以不同的数据开始。所以，我们可以创建我们的两个电视对象

类似这样：

oTV1 = TV('Samsung', '家庭房')

oTV2 = TV('Sony', 'Bedroom')

执行第一行时，Python首先为一个TV对象分配空间

对象。然后，它按照前一节中讨论的方式重新排列参数

然后调用电视类的__init__()方法，并传入三个参数：

新分配的oTV1对象，品牌和位置。

当初始化oTV1对象时，self.brand被设置为字符串'Samsung'

并且self.location被设置为字符串'家庭房'。初始化oTV2时，

它的self.brand被设置为字符串'Sony'，并且self.location被设置为

字符串'Bedroom'。

我们可以修改showInfo()方法，报告电视的名称和位置

电视的状态。

**文件：OO_TV_TwoInstances_with_Init_Params.py**

def showInfo(self):

print()

print('电视状态:', self.brand)

print(' 位置:', self.location)

如果self.isOn:

...

我们将看到如下输出：

电视状态：Sony

位置：家庭房

电视是：开

频道是：2

音量是：7

电视状态：Samsung

位置：卧室

电视是：开

频道是：44

音量是：10（声音静音）

我们在之前的例子中做了相同的方法调用

列表2-9。不同之处在于，现在每个TV对象都用一个

品牌和位置，现在你可以看到这些信息已经打印出来

对每次调用修改后的showInfo()方法的响应。

**44** 第2章

**使用中的类**

使用我们在本章学到的所有内容，我们现在可以创建类

并且可以从这些类中构建多个独立的实例。这里有一些

这里有一些我们可能会使用的例子：

• 假设我们想要建模一个学生的课程。我们可以有一个Student

类中有实例变量，用于存储名称、电子邮件地址、当前成绩等

依此类推。我们从这个类创建的每个Student对象都会有自己的

这些实例变量的集合，并且给实例变量赋值的值-

变量对于每个学生都会有所不同。

• 考虑一个有多个玩家的游戏。一个玩家可以是

由一个具有名称、积分、生命值等实例变量的玩家类建模。

位置等。每个玩家将具有相同的能力，但

方法可能会根据不同的值表现出不同的行为。

实例变量。

• 想象一下一个通讯录。我们可以创建一个具有实例变量

包含名称、地址、电话号码和生日的实例变量。我们可以创建

我们可以根据需要从人物类创建任意数量的对象，每个对象代表一个人

我们已知的信息。每个人物对象中的实例变量将包含

不同的值。然后我们可以编写代码来搜索所有的

人物对象，并获取我们正在寻找的一个或多个信息。

正在寻找。

在未来的章节中，我将探讨实例化多个

从一个类创建对象，并为你提供帮助管理对象集合的工具。

对象。

**面向对象编程作为解决方案**

在第1章的结尾，我提到了程序性编码固有的三个问题。希望在处理完这些示例后，

在本章中，你可以看到面向对象编程如何解决所有这些问题。

这些问题：

1\. 编写良好的类可以在许多不同的程序中轻松重用。

类不需要访问全局数据。相反，对象提供代码

和数据处于同一层次。

2\. 面向对象编程可以大大减少全局

所需的变量，因为类提供了一个框架，其中数据

并且作用于数据的代码存在于一个分组中。这通常也有助于

使代码更易于调试。

3\. 从一个类创建的对象只能访问自己的数据——它们的

类中的实例变量集。即使你有多个

从同一个类创建的对象无法访问彼此的变量。

彼此的数据。

使用面向对象编程建模物理对象 **45**

**总结**

在本章中，我介绍了面向对象编程的基础

编程，通过展示类与对象之间的关系。这个

类定义了对象的形状和功能。对象是一个单独的

每个类的实例都有自己的一组所有数据，这些数据在实例中定义

类的变量。每一项你希望对象包含的数据都需要

存储在一个实例变量中，该变量具有对象作用域，这意味着它仅在对象内部可用。

在类中定义的所有方法中都可以访问这些变量。所有从

类的多个对象可以拥有自己的实例变量集，并且因为

这些可能包含不同的值，调用不同对象上的方法

这可能导致不同的行为。

我展示了如何通过实例化一个类来创建一个对象，通常通过

赋值语句。实例化一个对象后，你可以使用它来进行

调用类中任何方法的次数。我还展示了如何

从同一个类实例化多个对象。

在本章中，演示类实现了物理对象

（灯开关、电视等）。这是开始理解类和对象概念的好方法。

类和对象的概念。然而，在接下来的章节中，我将介绍

并不代表物理对象的对象。

**46** 第2章

**3**

**物体的心理模型**

**和“self”含义的理解**

希望这些新概念和术语

到目前为止，我所介绍的术语和概念开始

使其合理。有些刚接触面向对象编程的人可能会感到困惑

难以想象对象是什么以及如何运作

对象的方法与其实例变量一起工作

变量。具体情况比较复杂，所以

对于开发如何理解对象和

类如何运作。

在本章中，我将展示两个 OOP 的心理模型。一开始，

我想明确的是，这些模型都不是完全准确的表示

如何在 Python 中工作对象的一个理解模型。而这些模型旨在提供

给你一种思考对象外观及其操作方式的方法

你调用一个方法时。本章还将详细介绍 self 和

展示它是如何用于使方法与多个实例化对象一起工作的

从同一个类实例化的对象。在本书的其余部分，你将获得

对对象和类有更深的洞察力。

**重新审视 DimmerSwitch 类**

在接下来的例子中，我们将继续使用 DimmerSwitch 类

[第2章（](index_split_000.html#p50)清单 2-5）。DimmerSwitch 类已经有了两个实例变量：self.isOn 和 self.brightness。我们唯一要修改的是

添加一个 self.label 实例变量，以便我们创建的每个对象都能被标识

当我们运行程序时，输出中很容易看到这些变量。它们是被创建的

并在 __init__() 方法中分配初始值。然后可以访问它们

或在类的其他五个方法中被修改。

清单 3-1 提供了一些测试代码，用于创建三个 DimmerSwitch 对象

来自 DimmerSwitch 类，我们将在心理模型中使用它。我将称其为

每个 DimmerSwitch 对象的各种方法。

**文件：OO_DimmerSwitch_Model1.py**

# 创建第一个 DimmerSwitch，打开它，并将亮度提高两次

oDimmer1 = DimmerSwitch('Dimmer1')

oDimmer1.turnOn()

oDimmer1.raiseLevel()

oDimmer1.raiseLevel()

# 创建第二个 DimmerSwitch，打开它，并将亮度提高 3 次

oDimmer2 = DimmerSwitch('Dimmer2')

oDimmer2.turnOn()

oDimmer2.raiseLevel()

oDimmer2.raiseLevel()

oDimmer2.raiseLevel()

# 创建第三个 DimmerSwitch，使用默认设置

oDimmer3 = DimmerSwitch('Dimmer3')

# 请求每个开关显示自己

oDimmer1.show()

oDimmer2.show()

oDimmer3.show()

*清单 3-1：创建三个 DimmerSwitch 对象并调用它们的各种方法* 当与 DimmerSwitch 类一起运行时，这段代码会生成如下输出

输出：

标签：Dimmer1

灯是开的吗？是

亮度是：2

标签：Dimmer2

灯是开的吗？是

亮度是：3

标签：Dimmer3

灯是开的吗？否

亮度是：0

**48** 第3章

这正是你所期望的。每个 DimmerSwitch 对象都独立于其他 DimmerSwitch 对象，每个对象都包含和

修改它自己的实例变量。

**高级心理模型 #1**

在这个第一个模型中，你可以把每个对象看作一个自包含的单元，它

包含一个数据类型、一组在类中定义的实例变量，以及

类中定义的所有方法的副本（图 3-1）。

oDimmer1

oDimmer2

oDimmer3

类型：

类型：

类型：

DimmerSwitch

DimmerSwitch

DimmerSwitch

数据：

数据：

数据：

label: Dimmer1

label: Dimmer2

label: Dimmer3

isOn: True

isOn: True

isOn: False

brightness: 2

brightness: 3

brightness: 0

方法：

方法：

方法：

_init_()

_init_()

_init_()

turnOn()

turnOn()

turnOn()

turnOff()

turnOff()

turnOff()

raiseLevel()

raiseLevel()

raiseLevel()

lowerLevel()

lowerLevel()

lowerLevel()

show()

show()

show()

*图 3-1：在心理模型 #1 中，每个对象都是一个具有类型、数据的单元，*

*和方法。*

每个对象的数据和方法被打包在一起。该

实例变量的作用范围被定义为类中的所有方法，

所以所有方法都可以访问与该

对象。

如果这个心理模型让概念变得清晰，那么你就已经掌握得很好了

形态。虽然这*不是*对象实际的实现方式，但它是一个完全合理的*思考方式*，可以帮助你理解对象的实例变量和方法如何协同工作。

**更深层的心理模型 #2**

这个第二个模型探索了较低层次的对象，并将进一步解释

关于对象是什么。

每次实例化一个对象时，Python 会返回一个值。

我们通常将返回值存储在一个引用该对象的变量中。

在 Listing 3-2 中，我们创建了三个 DimmerSwitch 对象。每创建一个对象后，

我们将添加代码来检查结果，通过打印出类型和值

每个变量。

对象的心理模型和“self” **49**

**文件：OO_DimmerSwitch_Model2_Instantiation.py**

# 创建三个 DimmerSwitch 对象

oDimmer1 = DimmerSwitch('Dimmer1')

print(type(oDimmer1))

print(oDimmer1)

print()

oDimmer2 = DimmerSwitch('Dimmer2')

print(type(oDimmer2))

print(oDimmer2)

print()

oDimmer3 = DimmerSwitch('Dimmer3')

print(type(oDimmer3))

print(oDimmer3)

print()

*Listing 3-2：创建三个 DimmerSwitch 对象并打印每个对象的类型和值* 下面是输出：

<class '__main__.DimmerSwitch'>

<__main__.DimmerSwitch 对象位于 0x7ffe503b32e0>

<class '__main__.DimmerSwitch'>

<__main__.DimmerSwitch 对象位于 0x7ffe503b3970>

<class '__main__.DimmerSwitch'>

<__main__.DimmerSwitch 对象位于 0x7ffe503b39d0>

每组中的第一行告诉我们数据类型。与内置的

类型，例如整数或浮点数，我们可以看到所有三个对象都是程序员定义的 DimmerSwitch 类型。

定义的 DimmerSwitch 类型。（__main__ 表示 DimmerSwitch 代码

被发现位于我们的单一 Python 文件中，而不是从其他文件导入。)

每组的第二行包含一串字符。每个

字符串表示计算机内存中的一个位置。内存

位置是与每个对象相关的所有数据所在的地方。

注意，每个对象位于内存中的不同位置。如果你运行此代码

在你的计算机上运行时，你可能会得到不同的值，但实际的

变量值对于理解这个概念并不重要。

所有 DimmerSwitch 对象报告相同的类型：DimmerSwitch 类。

一个极其重要的结论是，所有对象都引用同一份代码，

相同的类，它实际上只在一个地方存在。当你的程序开始时

运行时，Python 会读取所有类定义并记住

所有类及其方法的位置。

Python Tutor 网站（[http://PythonTutor.com](http://PythonTutor.com)）提供了一些有用的工具，可以帮助你通过逐行调试代码来可视化小程序的执行。图 3-2 是来自该网站的截图

通过可视化工具运行 DimmerSwitch 类和测试代码，

在实例化第一个 DimmerSwitch 对象之前停止执行。

在这张截图中，你可以看到 Python 记住了

DimmerSwitch 类及其所有方法。虽然类可以包含

尽管类中可能包含数百或数千行代码，但每个对象实际上并不会得到代码副本

类代码的副本。只有一份代码是非常重要的，因为

它保持了面向对象编程程序的紧凑性。当你实例化一个对象时，

**50** 第 3 章

![图片 14](index-80_1.png)

![图片 15](index-80_2.png)

Python 为每个对象分配足够的内存来表示它自己的变量集。

类中定义的实例变量。在一般情况下，实例化一个

从一个类创建对象时是节省内存的。

*图 3-2：Python 记住了所有类及每个类中的所有方法。*

图 3-3 中的截图展示了运行所有测试的结果

清单 3-2 中的代码\。

*图 3-3：运行清单 3-2 演示了对象不包括代码，这符合我们的第二个心理模型。*

对象的心理模型和“self”**51**

这与我们的第二个心理模型相匹配。在这一边，

截图中，DimmerSwitch 类的代码只出现一次。每个

对象知道它是从哪个类实例化的，并包含它自己的变量集。

类中定义的实例变量。

**注意**

*虽然以下内容属于实现细节，但它可能有助于你进一步理解对象。内部，所有对象的实例变量都作为名称/*

*Python 字典中的值对。你可以通过在任何对象上调用内置的 vars() 函数来检查所有实例变量。例如，在清单 3-2 的测试代码中，如果你想查看实例变量的内部表示，可以在最后添加这一行：*

print('oDimmer1 变量:', vars(oDimmer1))

*当你运行它时，你将看到以下输出：*

oDimmer1 变量：{'label': 'Dimmer1', 'isOn': True, 'brightness': 2}

**“self”是什么意思？**

哲学家们在这个问题上困惑了几个世纪，所以这将是

试图在几页中解释这个问题真是有些自负。在 Python 中，

然而，名为 self 的变量确实有一个高度专业化且明确的

含义。在本节中，我将展示如何给 self 赋值，以及如何

类中方法的代码与任何从类实例化的对象的实例变量一起工作

将从类中实例化。

**注意**

*self 变量名不是 Python 的关键字，但按照约定使用——可以使用任何其他名称，代码也能正常工作。然而，使用 self 是* *Python 中普遍接受的做法，我将在本书中贯穿使用。如果你* *希望你的代码被其他 Python 程序员理解，请在所有类方法的第一个参数中使用 self 作为名称。（其他面向对象语言有相同的概念，但使用其他名称，例如 this 或 me。）*

假设你编写了一个名为 SomeClass 的类，然后从

那个类，像这样：

oSomeObject = SomeClass( *<可选参数>* )

对象 oSomeObject 包含所有实例变量的集合

在类中定义。SomeClass 类的每个方法都有一个定义

它看起来像这样：

def someMethod(self, *<其他参数>* ):

这里是调用此类方法的一般形式：

oSomeObject.someMethod( *<其他参数>* )

**52** 第 3 章

如我们所知，Python 会重新排列方法调用中的参数，使得对象作为第一个参数传递。该值将在

方法的第一个参数并将其放入变量 self 中（图 3-4）。

def someMethod(self, *<其他参数>* ):

oSomeObject.someMethod( *<其他参数>* )

*图 3-4：Python 如何重新排列方法调用中的参数*

因此，每当调用一个方法时，self 将被设置为该对象

在调用中。这意味着方法的代码可以操作

从类实例化的 *任何* 对象的实例变量。它通过这种方式

使用这种形式：

self. *<实例变量名>*

这实际上表示使用 self 引用的对象并访问

由 < *实例变量名*> 指定的实例变量。由于每个方法都将 self 作为第一个参数，因此类中的每个方法都使用这个

同样的方法。

为了说明这个概念，我们使用 DimmerSwitch 类。在接下来的

作为示例，我们将实例化两个 DimmerSwitch 对象，然后逐步演示

当我们通过调用这些对象的亮度级别来增加它们时会发生什么

每个方法都使用 raiseLevel() 方法。

我们调用的方法的代码是：

def raiseLevel(self):

如果 self.brightness < 10:

self.brightness = self.brightness + 1

列表 3-3 显示了两个 DimmerSwitch 对象的一些示例测试代码。

**文件：OO_DimmerSwitch_Model2_Method_Cal s.py**

# 创建两个 DimmerSwitch 对象

oDimmer1 = DimmerSwitch('Dimmer1')

oDimmer2 = DimmerSwitch('Dimmer2')

# 告诉 oDimmer1 提高它的亮度级别

oDimmer1.raiseLevel()

# 告诉oDimmer2提高其级别

oDimmer2.raiseLevel()

*列表3-3：对不同的DimmerSwitch对象调用相同的方法*

对象的思维模型与“self”的意义 **53**

![Image 16](index-83_1.png)

在这个列表中，我们首先实例化了两个DimmerSwitch对象。然后我们有

两次调用raiseLevel()方法：首先我们用oDimmer1调用它，然后我们

使用oDimmer2调用相同的方法。

图3-5展示了运行列表3-3中测试代码的结果

在Python Tutor中，执行在首次调用时停止

raiseLevel()。

*图3-5：列表3-3中的程序在调用oDimmer1.raiseLevel()时停止* 请注意，self和oDimmer1指向同一个对象。当

当方法执行并使用任何self.< *instanceVariable*>时，它将使用oDimmer1的实例变量。因此，当这个方法运行时，self.brightness指

指向oDimmer1中的亮度实例变量。 

如果我们继续执行列表3-3中的测试代码，我们将进入第二个

第二次调用raiseLevel()时，使用oDimmer2。在图3-6中，我已经停止了执行

在这个方法调用中。

请注意，这次self指向的是与oDimmer2相同的对象。现在，

self.brightness指的是oDimmer2的亮度实例变量。

无论我们使用什么对象，或调用哪个方法，self的值

该对象被分配给被调用方法中的变量self。你应该

将self视为当前对象——方法正在操作的对象

被调用时。每当方法执行时，它会使用实例的变量集

变量是为在调用中指定的对象设置的。

**54** 第三章

![Image 17](index-84_1.png)

*图3-6：列表3-3中的程序在调用oDimmer2.raiseLevel()时停止* **总结**

在这一章中，我展示了两种不同的思维方式来理解对象。

这些思维模型应有助于你建立对

当你从同一个类实例化多个对象时，会发生什么。

类。

第一个模型展示了如何将对象视为拥有一组

所有实例变量和类的所有方法被打包在一个

很好的封装。

第二个模型详细讨论了实现的更多细节，

解释类的代码只存在于一个地方。一个重要的

主要收获是，从类中创建新对象是空间高效的。

当你创建一个新对象的实例时，Python会为其分配内存

代表类中定义的实例变量。没有实例变量的重复。

类的代码是在某个地方创建或需要的。

方法如何与多个对象一起工作的关键在于第一个

所有方法的参数self，总是设置为在调用时使用的对象

该方法。采用这种方法，每个方法使用实例变量

当前对象的

对象的思维模型与“self”的意义 **55**

**4**

**管理多个对象**

本章将向你展示管理多个对象的技巧

管理任何数量的实例化对象

来自相同类的对象实例化。我将首先讲解

一个银行账户的面向对象实现

来自[第1章 T](index_split_000.html#p32)的示例，面向对象方法允许

数据和代码将与账户保持在同一

层级结构，消除了依赖全局数据的需求。

然后，我会将程序拆分为主要代码，这些

提供一个顶级菜单和一个单独的银行对象

该账户管理类，除了处理任意数量的

账户对象。我们还将讨论更好的错误处理方法。

使用异常处理来处理错误。

**银行账户类**

我们的银行账户类至少需要有名称、密码、

以及余额作为数据。对于行为，用户必须能够创建一个

账户、存款、取款以及查询余额。

我们将为名称、密码和

余额，并构建方法来实现每个操作。我们

然后，你应该能够实例化任意数量的账户对象。像

来自第1章的初始类，这个简化版的账户类只使用整数来表示余额，并且将密码以明文形式保存。虽然你

在真实的银行应用中，你不会使用这样的简化方法，但它们

这将帮助我们集中处理面向对象的方面。

账户类的新代码展示在Listing 4-1中。

**文件：Account.py**

# 账户类

class Account():

1 def __init__(self, name, balance, password):

self.name = name

self.balance = int(balance)

self.password = password

2 def deposit(self, amountToDeposit, password):

如果密码 != self.password:

print('抱歉，密码错误')

return None

如果存款金额小于0：

print('您不能存入负数金额')

return None

self.balance = self.balance + amountToDeposit

return self.balance

3 def withdraw(self, amountToWithdraw, password):

如果密码 != self.password:

print('此账户的密码不正确')

return None

如果要提取的金额小于0：

print('您不能提取负数金额')

return None

如果提取的金额大于账户余额：

print('您不能提取超过账户余额的金额')

return None

self.balance = self.balance - amountToWithdraw

return self.balance

**58** 第4章

4 def getBalance(self, password):

如果密码 != self.password:

print('抱歉，密码错误')

return None

return self.balance

# 添加用于调试

5 def show(self):

print(' 姓名:', self.name)

print(' 余额:', self.balance)

print(' 密码:', self.password)

print()

*Listing 4-1：一个简化的账户类*

**注意**

*Listing 4-1中的错误处理非常简单。如果我们发现错误条件，我们* *打印一条错误信息并返回一个特殊的None值。稍后的章节中，我将* *展示一种更好的错误处理方式。*

请注意，这些方法如何操作和记住数据。数据是

传递到每个方法中的参数，作为局部变量

只在方法运行期间存在。数据会在实例中被记住

变量具有对象作用域，因此会记住它们的值

跨不同方法调用传递的参数，这些是局部变量

首先是__init__()方法1，带有三个参数。当

从这个类创建对象时，需要三个数据：名称，

余额和密码。实例化可能看起来像这样：

oAccount = Account('Joe Schmoe', 1000, 'magic')

当我们实例化对象时，三个参数的值是

传入__init__()方法，这会将这些值分配给

同样命名的实例变量：self.name、self.balance和self.password。

我们将在其他方法中访问这些实例变量。

deposit()方法2允许用户向账户存款。

在实例化一个Account对象并将其保存在oAccount中后，我们可以调用

以这种方式调用deposit()方法：

newBalance = oAccount.deposit(500, 'magic')

这个调用表示存入$500并提供密码“magic”。该

方法对存款请求执行两次有效性检查。第一次

通过测试提供的密码来确保密码正确

通过对创建Account对象时设置的密码进行调用，这是一个

是如何检查实例变量中保存的原始密码的一个好例子

使用了self.password。第二次有效性检查确保我们没有存入

输入负数金额（这实际上是取款）时。

如果这两个测试中的任何一个失败，现在我们返回特殊值None

显示一些错误发生的情况。如果两个测试都通过，我们增加

实例变量self.balance，通过存款金额来更新余额。因为

管理多个对象 **59**

余额存储在self.balance中，它被记住并可供未来的调用使用。最后，我们返回新的余额。

withdraw()方法3的工作方式非常相似，也会是

像这样调用：

oAccount.withdraw(250, 'magic')

withdraw()方法检查我们是否提供了正确的密码

通过验证它与实例变量self.password进行比较。它还会检查

我们没有要求提取负数金额或超出账户余额的金额。

账户，使用实例变量self.balance。一旦这些测试通过，

方法通过提取的金额来减少self.balance。它返回

结果余额。

要检查余额4时，我们只需要提供正确的密码

对账户进行操作：

currentBalance = oAccount.getBalance('magic')

如果提供的密码与实例变量中保存的密码匹配

self.password时，方法返回self.balance中的值。

最后，为了调试，我们添加了一个show()方法5来显示

当前保存的self.name、self.balance和self.password的值

账户。

Account类是我们第一个表示某个事物的示例，

它不是一个物理对象。银行账户不是你可以看到的东西，

感觉或触感。然而，它完美地融入了计算机对象的世界

因为它有数据（名称、余额、密码）和对这些数据进行操作的

这些数据（创建、存款、取款、获取余额、显示）。

**导入类代码**

使用自己构建的类有两种方法。正如我们  

如前几章所见，最简单的方法是将所有类的代码放在  

直接在主 Python 源文件中定义类。但这样做会使得  

需要重用一个类。  

第二种方法是将类的代码单独放在一个文件中。  

并将其导入使用该类的程序中。我们将所有的代码放在了  

*Account.py* 文件中的 Account 类，但是如果我们尝试单独运行 *Account.py*，什么也不会发生，因为它仅仅是类的定义。为了使用我们的类代码，我们  

必须实例化一个或多个对象，并调用对象的方法。  

随着我们的类变得越来越大且复杂，将每个类  

将它们作为单独的文件导入是处理它们的首选方式。  

为了使用我们的 Account 类，我们必须构建另一个 *.py* 文件并导入该文件。  

从 *Account.py* 导入代码，就像我们对待其他内建包（如 random 和 time）一样。通常，Python 程序员会将导入了  

其他类文件如 *main.py* 或 *Main_<SomeName>.py*。然后我们必须确保 *Account.py* 和主程序文件在同一个文件夹中。  

**60** 第 4 章  

在主程序的开头，我们通过一个 import 语句导入 Account 代码（注意我们省略了 **.py* 文件扩展名）：from Account import *  

使用带有星号（*）的 import 语句会将整个内容导入。  

导入文件的内容。一个导入的文件可以包含多个类。在  

在这种情况下，尽可能指定你想导入的具体类或类集合，  

你想导入的类，而不是导入整个文件。以下是语法  

导入特定类时，  

from *<ExternalFile>* import *<ClassName1>, <ClassName2>, ...*  

导入类代码有两个好处：  

1\. 模块可重用，因此如果我们想在其他地方使用 *Account.py*，  

项目中，我们只需要复制该文件并将其放入相应的目录中。  

项目文件夹中。以这种方式重用代码是面向对象编程的基础。  

编程。  

2\. 如果你的类代码包含在主程序中，每次运行  

程序时，Python 会编译你类中的所有代码（将其翻译成  

转换为更易在计算机上运行的低级语言。  

计算机），即使你没有对类做任何修改。  

然而，当你运行主程序并导入类代码时，  

Python 会优化编译步骤，而你无需进行任何操作。  

什么也不做。它会在项目文件夹中创建一个名为 *__pycache__* 的文件夹，  

然后 Python 会编译类文件中的代码，并保存编译后的代码。  

存储在 *__pycache__* 文件夹中，并以原始 Python 文件名的变体保存。  

例如，对于 *Account.py* 文件，Python 将使用  

文件名 *Account.cpython-39.pyc*（或类似名称，取决于你使用的 Python 版本）。  

你正在使用的 Python 版本）。*.pyc* 扩展名表示 *Python 编译文件*。只有当类文件的源代码发生变化时，Python 才会重新编译该类文件。如果

由于*Account.py*的源文件没有变化，Python知道它不需要

无需重新编译它，可以更高效地使用*.pyc*版本的

文件替代。

**创建一些测试代码**

我们将通过四个主程序来测试我们的新类。第一个将创建Account

使用单独命名的变量来存储对象。第二个将对象存储在一个

列表，而第三个将账户号码和对象存储在字典中。

最终，第四个版本将拆分功能，使我们有一个主程序-

程序响应用户并管理不同账户的Bank对象

账户。

在每个示例中，主程序导入*Account.py*。您的项目

文件夹应包含主程序和*Account.py*文件。在

管理多个对象 **61**

随后的讨论中，主程序的不同版本将命名为*Main_Bank_VersionX.py*，其中*X*代表版本号*。*

***创建多个账户***

在这个版本中，我们将创建两个示例账户并填充它们

使用有效的数据进行测试。我们将每个账户保存在一个明确命名的

代表对象的变量。

**文件：BankOOP1_IndividualVariables/Main_Bank_Version1.py**

# 使用账户测试程序

# 版本1，使用每个Account对象的显式变量

# 导入所有来自Account类文件的代码

从 Account 导入 *

# 创建两个账户

1 oJoesAccount = Account('Joe', 100, 'JoesPassword')

print("为Joe创建了一个账户")

2 oMarysAccount = Account('Mary', 12345, 'MarysPassword')

print("为Mary创建了一个账户")

3 oJoesAccount.show()

oMarysAccount.show()

print()

# 调用不同账户的一些方法

print('调用两个账户的方法...')

4 oJoesAccount.deposit(50, 'JoesPassword')

oMarysAccount.withdraw(345, 'MarysPassword')

oMarysAccount.deposit(100, 'MarysPassword')

# 显示账户

oJoesAccount.show()

oMarysAccount.show()

*列表 4-2：用于测试Account类的主程序*

我们为Joe创建了一个账户1，为Mary创建了一个账户2，我们

存储结果到两个Account对象中。然后我们调用show()方法

显示账户以证明它们已经正确创建3。Joe存款-

它的$50。Mary进行了一次$345的取款操作，然后存入$1004。如果我们

现在运行程序，这将是我们的输出：

为Joe创建了一个账户

为Mary创建了一个账户

姓名：Joe

余额：100

密码：JoesPassword

姓名：Mary

**62** 第4章

余额：12345

密码：MarysPassword

调用两个账户的方法...

姓名：Joe

余额：150

密码：JoesPassword

姓名：Mary

余额：12100

密码：MarysPassword

现在我们将扩展测试程序，创建第三个账户，进行交互-

通过请求用户输入来交互地创建账户。列表 4-3 显示了代码

为此。

# 根据用户信息创建另一个账户

print()

userName = input('新用户账户的名字是什么？') 1

userBalance = input('此账户的初始余额是多少？')

userBalance = int(userBalance)

userPassword = input('您想为此账户使用什么密码？')

oNewAccount = Account(userName, userBalance, userPassword) 2

# 显示新创建的用户账户

oNewAccount.show() 3

# 让我们将100存入新账户

oNewAccount.deposit(100, userPassword) 4

usersBalance = oNewAccount.getBalance(userPassword)

print()

print('存入100后，用户余额为：', usersBalance)

# 显示新账户

oNewAccount.show()

*清单 4-3：测试程序的扩展，动态创建账户*

这个测试代码要求用户输入一个名字、一个初始余额和一个密码

密码 1。它使用这些值创建一个新账户，并将新创建的账户存储

在变量oNewAccount中创建的对象 2。然后我们调用show()方法

对新对象进行操作 3。我们将$100存入账户并获取新的

通过调用getBalance()方法获取余额 4。当我们运行完整的程序时，

程序运行时，我们将从清单4-2获得输出，以及以下输出：

新用户账户的名称是什么？Irv

该账户的起始余额是多少？777

您想为此账户使用什么密码？IrvsPassword

名字：Irv

余额：777

密码：IrvsPassword

存入100后，用户余额为：877

管理多个对象 **63**

名字：Irv

余额：877

密码：IrvsPassword

这里需要注意的关键点是，每个Account对象都会维护它自己的

每个对象（oJoesAccount、oMarysAccount等）都有自己的实例变量集。

oNewAccount) 是一个全局变量，包含三个实例

变量。如果我们扩展Account类的定义以包含

信息，比如地址、电话号码和出生日期，每个

对象将获得一组这些附加的实例变量。

***多个账户对象在列表中***

将每个账户表示为一个单独的全局变量是可行的，但这并不是最佳方案

这是处理大量对象时的一个不错的方法。一个银行

需要一种方法来处理任意数量的账户。每当我们

需要任意数量的数据时，列表是典型的解决方案。

在这个版本的测试代码中，我们将从一个空的Account列表开始

每当用户开设账户时，我们都会实例化一个Account

对象，并将生成的对象追加到我们的列表中。账户号码

对于任何给定的账户，账户在列表中的索引将从0开始

从0开始。我们首先为Joe和Mary创建一个测试账户，

如清单4-4所示。

**文件：BankOOP2_ListOfAccountObjects/Main_Bank_Version2.py**

# 使用账户的测试程序

# 版本2，使用账户列表

# 从Account类文件中导入所有代码

从Account导入*

# 从一个空的账户列表开始

accountsList = [ ] 1

# 创建两个账户

oAccount = Account('Joe', 100, 'JoesPassword') 2

accountsList.append(oAccount)

print("Joe的账户号码是0")

oAccount = Account('Mary', 12345, 'MarysPassword') 3

accountsList.append(oAccount)

print("玛丽的账户号码是1")

accountsList[0].show() 4

accountsList[1].show()

print()

# 在不同的账户上调用一些方法

print('正在调用两个账户的方法...')

accountsList[0].deposit(50, 'JoesPassword') 5

accountsList[1].withdraw(345, 'MarysPassword') 6

**64** 第4章

accountsList[1].deposit(100, 'MarysPassword') 7

# 显示账户

accountsList[0].show() 8

accountsList[1].show()

# 根据用户的信息创建另一个账户

print()

userName = input('新用户账户的名称是什么？')

userBalance = input('此账户的初始余额是多少？')

userBalance = int(userBalance)

userPassword = input('你想为这个账户设置的密码是什么？')

oAccount = Account(userName, userBalance, userPassword)

accountsList.append(oAccount) # 将其附加到账户列表中

# 显示新创建的用户账户

print('创建了新账户，账户号码是2')

accountsList[2].show()

# 让我们向新账户存入100

accountsList[2].deposit(100, userPassword)

usersBalance = accountsList[2].getBalance(userPassword)

print()

print('存入100后，用户的余额为：', usersBalance)

# 显示新账户

accountsList[2].show()

*清单4-4：修改后的测试代码，将对象存储在列表中*

我们从创建一个空的账户列表开始1。我们创建一个账户

对乔来说，将返回的值存储到变量oAccount中，并立即

将该对象附加到我们的账户列表中2。由于这是第一个账户

的账户号码是0。就像在真实银行中一样，每次乔

任何时候他想对账户进行交易，他都会提供他的账户号码-

号码。我们使用他的账户号码来显示他账户的余额4，并进行

存款5，然后再次显示余额8。我们还为

玛丽的账户号码是1 3，并对她的账户进行一些测试操作

账户在6和7处。

结果与清单4-3中的测试代码相同。然而，

这里有一个非常重要的区别：这两个测试程序之间的差异：

现在只有一个全局变量accountsList。每个账户都有一个

唯一账户号码，我们用来访问特定的账户。我们在列表中有

采取了减少全局变量数量的重要步骤。

另一个重要的注意事项是，我们做了一些相当重大的

对主程序进行了更改，但我们没有触动Account类中的任何内容

class文件。面向对象编程通常允许你在不同级别上隐藏细节。如果我们假设

负责处理与个人账户相关的细节

账户，我们可以集中精力改进主代码。

还要注意，我们正在使用变量oAccount作为*临时*变量。

也就是说，每当我们创建一个新的Account对象时，我们都会分配结果

将其赋值给变量oAccount。就在那之后，我们将oAccount附加到

管理多个对象 **65**

账户列表。我们在调用任何特定Account对象的方法时从不使用变量oAccount。这样，我们可以重用变量oAccount来

获取下一个创建的账户的值。

***具有唯一标识符的多个对象***

Account对象必须是唯一可识别的，以便每个用户可以进行

存款和取款，并获取他们特定账户的余额。

使用列表来存储我们的银行账户是可行的，但有一个严重的缺陷。想象一下

我们有五个账户，编号为0、1、2、3和4。如果拥有

如果账户2决定关闭他们的账户，我们很可能会使用标准的pop()

对列表进行操作以删除账户2。这将引发连锁反应：

曾在位置3的账户现在处于位置2，而

曾在位置4的账户现在处于位置3。然而，这些账户的用户

他们仍然保留原始账户号码3和4。因此，客户

拥有账户3的人现在将获得上一个账户的信息

4，并且账户号码4现在是一个无效的索引。

为了处理大量具有唯一标识符的对象，我们通常

使用字典。与列表不同，字典允许我们删除账户

而不改变与之相关的账户号码。我们构建每个

以账户号码作为键，Account对象作为值的键/值对

值。这样，如果我们需要删除某个账户，其他账户不会

受影响。账户的字典将如下所示：

{0 : *<account 0的对象>* , 1 : *<account 1的对象>* , ... }

然后我们可以轻松获取相关的Account对象并调用方法

如下所示：

oAccount = accountsDict[accountNumber]

oAccount.someMethodCall()

或者，我们可以直接使用accountNumber来调用

单个Account的调用方法：

accountsDict[accountNumber].someMethodCall()

列表4-5展示了使用账户对象字典的测试代码。

同样，尽管我们对测试代码进行了许多更改，但我们并没有改变

在Account类中增加一行代码。在我们的测试代码中，而不是使用

硬编码的账户号码，我们增加了一个计数器nextAccountNumber，我们将

在创建新Account后递增。

**文件：BankOOP3_DictionaryOfAccountObjects/Main_Bank_Version3.py**

# 使用账户的测试程序

# 版本3，使用账户字典

# 导入所有来自Account类文件的代码

从Account导入*

**66** 第四章

accountsDict = {} 1

nextAccountNumber = 0 2

# 创建两个账户：

oAccount = Account('Joe', 100, 'JoesPassword')

joesAccountNumber = nextAccountNumber

accountsDict[joesAccountNumber] = oAccount 3

print('乔的账户号码是:', joesAccountNumber)

nextAccountNumber = nextAccountNumber + 1 4

oAccount = Account('Mary', 12345, 'MarysPassword')

marysAccountNumber = nextAccountNumber

accountsDict[marysAccountNumber] = oAccount 5

print('玛丽的账户号码是:', marysAccountNumber)

nextAccountNumber = nextAccountNumber + 1

accountsDict[joesAccountNumber].show()

accountsDict[marysAccountNumber].show()

print()

# 调用不同账户上的一些方法

print('正在调用两个账户的方法...')

accountsDict[joesAccountNumber].deposit(50, 'JoesPassword')

accountsDict[marysAccountNumber].withdraw(345, 'MarysPassword')

accountsDict[marysAccountNumber].deposit(100, 'MarysPassword')

# 显示所有账户

accountsDict[joesAccountNumber].show()

accountsDict[marysAccountNumber].show()

# 使用用户提供的信息创建另一个账户

print()

userName = input('新用户账户的名称是什么？')

userBalance = input('这个账户的起始余额是多少？')

userBalance = int(userBalance)

userPassword = input('你想为这个账户设置什么密码？')

oAccount = Account(userName, userBalance, userPassword)

newAccountNumber = nextAccountNumber

accountsDict[newAccountNumber] = oAccount

print('新账户的账户号码为:', newAccountNumber)

nextAccountNumber = nextAccountNumber + 1

# 显示新创建的用户账户

accountsDict[newAccountNumber].show()

# 让我们将100存入新账户

accountsDict[newAccountNumber].deposit(100, userPassword)

usersBalance = accountsDict[newAccountNumber].getBalance(userPassword)

print()

print('存入100后，用户的余额为:', usersBalance)

# 显示新账户

accountsDict[newAccountNumber].show()

*列表4-5：修改后的测试代码，将账户号码和对象存储在字典中* 管理多个对象 **67**

运行此代码将产生与之前示例几乎相同的结果。我们从一个空的账户字典开始，1，并初始化

初始化我们的nextAccountNumber变量为0 2\。每次我们实例化一个新

账户时，我们使用当前的账户号码作为键，向账户字典中添加一个新条目

使用nextAccountNumber的租用值作为键，Account对象作为值3\。

我们对每个客户都这么做，如你所见，针对Mary 5\。每次我们

创建一个新账户时，我们增加nextAccountNumber，为下一个账户做准备

账户4\。使用账户号码作为字典中的键，如果一个客户

如果一个客户关闭了他们的账户，我们可以将该键和值从字典中删除

ary，且不会影响其他账户。

***构建互动菜单***

在我们的Account类正确工作的基础上，我们将使主代码互动

通过询问用户他们想进行的操作，使程序更加互动：获取

余额，进行存款，进行取款，或者开设新账户。在

响应时，我们的主要代码将从用户那里收集所需的信息，

从他们的账户号码开始，并调用相应的账户方法

用户的Account对象。

作为快捷方式，我们将再次预填充两个账户，一个是Joe的，另一个是

另一个是Mary的。列表4-6展示了我们扩展的主要代码，它使用字典

我省略了创建字典来跟踪所有账户的代码。 

针对Joe和Mary的账户，并将它们添加到账户字典中

为简洁起见，如同 Listing 4-5 中一样\。

**文件：BankOOP4_InteractiveMenu/Main_Bank_Version4.py**

# 创建账户字典的互动测试程序

# 版本 4，带有交互菜单

from Account import *

accountsDict = {}

nextAccountNumber = 0

--- snip 创建账户，添加到字典 ---

while True:

print()

print('按 b 获取余额')

print('按 d 存款')

print('按 o 开设新账户')

print('按 w 取款')

print('按 s 显示所有账户')

print('按 q 退出')

print()

action = input('你想做什么？') 1

action = action.lower()

action = action[0]  # 只取第一个字母

**68** 第4章

print()

如果 action == 'b'：

print('*** 获取余额 ***')

userAccountNumber = input('请输入您的账户号码：')

userAccountNumber = int(userAccountNumber)

userAccountPassword = input('请输入密码：')

oAccount = accountsDict[userAccountNumber]

theBalance = oAccount.getBalance(userAccountPassword)

如果 theBalance 不为 None：

print('您的余额为：', theBalance)

elif action == 'd': 2

print('*** 存款 ***')

userAccountNumber = input('请输入账户号码：') 3

userAccountNumber = int(userAccountNumber)

userDepositAmount = input('请输入存款金额：')

userDepositAmount = int(userDepositAmount)

userPassword = input('请输入密码：')

oAccount = accountsDict[userAccountNumber] 4

theBalance = oAccount.deposit(userDepositAmount, userPassword) 5

如果 theBalance 不为 None：

print('您的新余额为：', theBalance)

elif action == 'o':

print('*** 开设账户 ***')

userName = input('新用户账户的用户名是什么？')

userStartingAmount = input('此账户的初始余额是多少？')

userStartingAmount = int(userStartingAmount)

userPassword = input('您想为此账户设置什么密码？')

oAccount = Account(userName, userStartingAmount, userPassword)

accountsDict[nextAccountNumber] = oAccount

print('您的新账户号码为：', nextAccountNumber)

nextAccountNumber = nextAccountNumber + 1

print()

elif action == 's':

print('显示：')

for userAccountNumber in accountsDict:

oAccount = accountsDict[userAccountNumber]

print(' 账户号：', userAccountNumber)

oAccount.show()

elif action == 'q':

break

elif action == 'w':

print('*** 取款 ***')

userAccountNumber = input('请输入您的账户号码：')

userAccountNumber = int(userAccountNumber)

userWithdrawalAmount = input('请输入取款金额：')

userWithdrawalAmount = int(userWithdrawalAmount)

userPassword = input('请输入密码：')

oAccount = accountsDict[userAccountNumber]

theBalance = oAccount.withdraw(userWithdrawalAmount, userPassword)

管理多个对象 **69**

如果 theBalance 不为 None：

print('已取出：', userWithdrawalAmount)

print('您的新余额为：', theBalance)

else:

print('抱歉，那不是有效的操作，请重试。')

print('完成')

*Listing 4-6：添加交互菜单*

在这个版本中，我们向用户展示一个选项菜单。当用户

选择一个操作 1，代码询问关于预期交易的问题

收集所有我们需要的信息以调用用户账户的方法。

例如，如果用户想要存款 2，程序会询问

账户号码、存款金额和账户密码的主代码 3。

我们使用账户号作为字典中 Account 对象的键

获取适当的 Account 对象 4. 使用该对象，我们随后调用

deposit() 方法，传入存款金额和用户的密码 5。

我们再次修改了主要代码部分，并且保持了我们

Account 类保持不变。

**创建一个对象管理器对象**

列表 4-6 中的代码实际上做了两件不同的事情。该程序

首先提供一个简单的菜单界面。然后，当选择一个操作时，它

收集数据并调用 Account 对象的方法。与其

通过将一个大型主程序分割成两项不同任务，我们可以将

这段代码将被拆分成两个更小的逻辑单元，每个单元有着明确的角色。

菜单系统变成了决定执行哪种操作的主要代码

处理该任务，其余的代码则处理银行实际操作的内容。

银行可以被建模为一个管理其他（账户）对象的对象

对象，称为 *对象管理器对象*。

**对象管理器** 一个维护受管理对象的列表或字典的对象（通常是单一的 **对象** 类），并调用这些对象的方法。

这种分割可以轻松且合乎逻辑地完成：我们将所有与

将这部分代码移至银行，并将其放入一个新的 Bank 类。然后，在主代码的开头

主程序中，我们从新建的 Bank 类实例化一个单一的 Bank 对象。

Bank 类将管理一个 Account 对象的列表或字典。在这个

这样，Bank 对象将是唯一直接与之通信的代码

Account 对象（图 4-1）。

为了创建这个层次结构，我们需要一些处理

最高级别的菜单系统。响应用户选择的操作时，主代码

将调用 Bank 对象的方法（例如 deposit() 或 withdraw()）。该

Bank 对象将收集它需要的信息（账户号、密码、

存款或取款金额），从其账户字典中查找

**70** 第 4 章

匹配用户账户，并调用该用户账户的相应方法。

主代码

Bank 对象

Account

Account

Account

对象

对象

对象

…

Account

对象

*图 4-1：主代码管理一个 Bank 对象，后者管理*

*许多 Account 对象。*

这种劳动分工有三个层次：

1. 创建并与单一 Bank 对象交互的主要代码

2. 该

管理一个字典的 Account 对象并调用

这些对象的方法

3. 该

Account 对象本身

采用这种方法，我们只有一个全局变量，即 Bank

对象。实际上，主代码根本不知道 Account 对象的存在。

相反，每个Account对象不清楚（也不关心）其

程序的顶层用户界面是。Bank对象接收来自

来自主代码并与相应的Account对象进行通信。

这种方法的主要优点是我们将一个

更大的程序分解成了更小的子程序：在这种情况下，主代码

和两个类组成。这使得编程每个部分变得更加容易，因为

每个的工作范围更小，职责更清晰。

此外，仅有一个全局变量确保了低级代码

不会意外影响全局数据。

在计算机文献中，图4-1所示的构造通常

被称为*组合*或*对象组合*。

**组合**

一种逻辑结构，其中一个对象管理一个或多个其他对象

你可以将一个对象看作是由其他对象组成的。一个

示例是，一个汽车对象由一个发动机对象、一个方向盘

对象、一些门对象、四个车轮和轮胎对象，以及

讨论通常集中在对象之间的关系。

在这个例子中，人们会说一辆汽车“拥有”方向盘、发动机，

一些门的数量，等等。因此，汽车对象是一个复合

其他对象的关系。

管理多个对象 **71**

我们将有三个独立的文件。主代码位于它自己的文件中。

它导入了我们新*Bank.py*文件的代码，该文件包含Bank类

(Listing 4-7)。Bank类导入了*Account.py*文件的代码，并

使用它来根据需要实例化Account对象。

***构建对象管理器对象***

Listing 4-7 显示了新Bank类的代码，它是一个对象管理器

对象。

**文件：BankOOP5_SeparateBankClass/Bank.py**

# 管理一个Account对象字典的银行

from Account import *

class Bank():

def __init__(self):

self.accountsDict = {} 1

self.nextAccountNumber = 0

def createAccount(self, theName, theStartingAmount, thePassword): 2

oAccount = Account(theName, theStartingAmount, thePassword)

newAccountNumber = self.nextAccountNumber

self.accountsDict[newAccountNumber] = oAccount

# 增加以准备创建下一个账户

self.nextAccountNumber = self.nextAccountNumber + 1

return newAccountNumber

def openAccount(self): 3

print('*** 开户 ***')

userName = input('新用户账户的名称是什么？')

userStartingAmount = input('此账户的初始余额是多少？')

userStartingAmount = int(userStartingAmount)

userPassword = input('您想为此账户使用的密码是什么？')

userAccountNumber = self.createAccount(userName, userStartingAmount, userPassword) 4

print('您的新账户号码是：', userAccountNumber)

print()

def closeAccount(self): 5

print('*** 关闭账户 ***')

userAccountNumber = input('您的账户号码是什么？')

userAccountNumber = int(userAccountNumber)

userPassword = input('您的密码是什么？')

oAccount = self.accountsDict[userAccountNumber]

theBalance = oAccount.getBalance(userPassword)

如果theBalance不为None：

print('您账户中有', theBalance, '正在退还给您。')

# 从账户字典中删除用户的账户

del self.accountsDict[userAccountNumber]

print('您的账户现已关闭。')

**72** 第4章

def balance(self):

print('*** 获取余额 ***')

userAccountNumber = input('请输入您的账户号码：')

userAccountNumber = int(userAccountNumber)

userAccountPassword = input('请输入密码：')

oAccount = self.accountsDict[userAccountNumber]

theBalance = oAccount.getBalance(userAccountPassword)

如果theBalance不为None：

print('您的余额是：', theBalance)

def deposit(self):

print('*** 存款 ***')

accountNum = input('请输入账户号码：')

accountNum = int(accountNum)

depositAmount = input('请输入存款金额：')

depositAmount = int(depositAmount)

userAccountPassword = input('请输入密码：')

oAccount = self.accountsDict[accountNum]

theBalance = oAccount.deposit(depositAmount, userAccountPassword)

如果theBalance不为None：

print('您的新余额是：', theBalance)

def show(self):

print('*** 显示 ***')

for userAccountNumber in self.accountsDict:

oAccount = self.accountsDict[userAccountNumber]

print(' 账户：', userAccountNumber)

oAccount.show()

def withdraw(self):

print('*** 提款 ***')

userAccountNumber = input('请输入您的账户号码：')

userAccountNumber = int(userAccountNumber)

userAmount = input('请输入提取金额：')

userAmount = int(userAmount)

userAccountPassword = input('请输入密码：')

oAccount = self.accountsDict[userAccountNumber]

theBalance = oAccount.withdraw(userAmount, userAccountPassword)

如果theBalance不为None：

print('已提取：', userAmount)

print('您的新余额是：', theBalance)

*列表 4-7：具有不同银行操作方法的Bank类* 我将重点介绍Bank类中需要注意的最重要的内容。

首先，在__init__()方法中，Bank初始化了两个变量：self.accountsDict

和self.nextAccountNumber 1\. 前缀self.指定这些为实例变量，

实例变量，这意味着Bank类可以在其任何方法中引用这些变量。

其次，有两个方法用于创建账户：createAccount()

和openAccount()。createAccount()方法实例化一个新账户 2

使用用户的姓名、初始金额和传入的密码

新账户。openAccount()方法会向用户询问问题以获取

这三条信息3并调用createAccount()方法

在同一个类内部。

管理多个对象 **73**

在同一个类中，一个方法调用另一个方法是常见的。

但被调用的方法并不知道它是从内部还是

在类外部；它只知道第一个参数是该对象，

它应该运行的地方。因此，方法的调用必须以self.开头，

因为self总是指当前对象。通常，要在

在同一类中的一个方法调用另一个方法时，我们需要写：

def myMethod(self, *<其他可选参数>* ):

...

self.methodInSameClass( *<任何必要的参数>* )

在收集用户信息以调用openAccount()后，我们已经

这一行4：

userAccountNumber = self.createAccount(userName, userStartingAmount, userPassword)

在这里，openAccount()调用来自同一类的createAccount()来创建—

创建账户。createAccount()方法运行，实例化一个账户对象

对象间调用方法，我们需要写：

将账户号码返回给用户。

最后，新的方法closeAccount()允许用户关闭一个账户

已存在账户 5\. 这是我们将提供的额外功能

我们的主代码。

我们的银行类表示银行的抽象视图，而非实际

实体化的有形物理对象。这是类的另一个好例子

这并不代表一个物理结构。

***创建对象管理器对象的主代码***

创建并对银行对象进行调用的主代码如下所示：

清单4-8\。

**文件：BankOOP5_SeparateBankClass/Main_Bank_Version5.py**

# Main程序控制一个由账户组成的银行

# 引入银行类的所有代码

from Bank import *

# 创建一个银行实例

oBank = Bank()

# 主代码

# 创建两个测试账户

joesAccountNumber = oBank.createAccount('Joe', 100, 'JoesPassword')

print("Joe的账户号码是:", joesAccountNumber)

marysAccountNumber = oBank.createAccount('Mary', 12345, 'MarysPassword')

print("Mary的账户号码是:", marysAccountNumber)

while True:

print()

**第74章**

print('要查询账户余额，请按 b')

print('要关闭账户，请按 c')

print('要进行存款，请按 d')

print('要开设新账户，请按 o')

print('要退出，请按 q')

print('要显示所有账户，请按 s')

print('要进行取款，请按 w ')

print()

1 action = input('你想做什么？')

action = action.lower()

action = action[0]  # 获取第一个字母

print()

2 如果 action == 'b':

oBank.balance()

3 elif action == 'c':

oBank.closeAccount()

elif action == 'd':

oBank.deposit()

elif action == 'o':

oBank.openAccount()

elif action == 's':

oBank.show()

elif action == 'q':

break

elif action == 'w':

oBank.withdraw()

else:

print('抱歉，那不是一个有效的操作。请再试一次。')

print('完成')

*清单4-8：创建一个银行对象并对其进行调用的主代码*

注意如何在清单4-8中，代码呈现了顶级菜单系统。

系统询问用户一个操作 1，然后调用相应的方法

使用银行对象来完成工作 2\. 你可以轻松扩展银行对象

处理一些附加查询，比如询问银行的营业时间，或

地址或电话号码。这些数据可以作为额外信息存储

实例变量存在于银行对象内部。银行会回答这些问题—

无需与任何 Account 对象进行通信即可处理这些异常。

当提出关闭请求时，主代码调用 closeAccount()

Bank 对象的方法来关闭账户。Bank 对象会从

使用类似这样的代码从账户字典中删除特定的账户：

del self.accountsDict[userAccountNumber]

管理多个对象 **75**

回想一下，我们对对象的定义是：数据加上作用于这些数据的代码，随着时间的推移。能够删除对象展示了第三个

我们对对象的定义的一部分。我们可以创建一个对象（在这个例子中是

Account 对象）我们可以随时使用，而不仅仅是在程序启动时。这里

程序运行时，我们会在用户决定开设新账户时创建一个新的 Account 对象

account。我们的代码可以通过调用它的方法来使用该对象。我们还可以

随时删除一个对象，在这种情况下，当用户选择关闭其账户时

account。这是一个如何处理对象（如 Account 对象）具有

生命周期，从对象创建之时直到被删除之时。

**更好的错误处理与异常**

到目前为止，在我们的 Account 类中，如果方法检测到错误（例如，如果

用户存入负数金额，输入错误的密码，提取过多金额等

负数金额等）我们的占位符解决方案是返回 None

信号某些操作出错。在本节中，我们将讨论一种更好的错误处理方式，

通过使用 try/except 块和抛出异常来处理错误的示例。

***try 和 except***

当函数中发生运行时错误或异常情况时

来自 Python 标准库的方法，该函数或方法签名

通过引发异常来处理错误（有时称为 *抛出* 或 *生成* 异常）。我们可以使用

try/except 结构。以下是一般形式：

try:

# 可能导致错误的代码（引发异常）

except *<某个异常名称>* : # 如果发生异常

# 处理异常的一些代码

如果 try 块中的代码正常工作且没有生成

异常发生时，except 子句被跳过，执行在

except 块。可是，如果 try 块中的代码导致了异常，

异常时，控制权将传递给 except 语句。如果异常与

异常（或多种异常之一）列在 except 语句中，

控制权会转移到 except 子句的代码中。这通常被称为

被称为 *捕获* 异常。那个缩进块通常包含用于报告和/或恢复错误的代码。

这是一个简单的示例，我们请求用户输入一个数字并

尝试将其转换为整数：

age = input('请输入您的年龄: ')

try: # 尝试转换为整数

age = int(age)

except ValueError: # 如果在尝试转换时抛出异常

print('抱歉，那不是一个有效的数字')

**76** 第4章

对Python标准库的调用可能会生成标准异常，例如TypeError、ValueError、NameError、ZeroDivisionError等。在这种

例如，如果用户输入字母或浮动点数，内建的

int()函数会引发ValueError异常，控制权会转移到

在except块中执行代码。

***raise语句和自定义异常***

如果您的代码检测到运行时错误条件，您可以使用raise语句

用来表示异常的raise语句有多种形式，

但是标准方法是使用以下语法：

raise *<ExceptionName>* (' *<任何描述错误的字符串>* ') 对于*<ExceptionName>*，您有三个选择。首先，如果有一个标准异常匹配您检测到的错误（例如TypeError，

ValueError、NameError、ZeroDivisionError等，如果存在，使用它也可以。您

也可以添加您自己的描述字符串：

raise ValueError('您需要指定一个整数')

其次，您可以使用通用的Exception异常：

raise Exception('金额不能是浮动点数')

然而，通常不推荐这样做，因为标准做法

是编写except语句来按名称查找异常，这样做

不提供特定名称。

第三个选择，也许是最好的选择，是创建您自己的自定义

异常。这很容易做到，但涉及到一种叫做继承的技术

（我们将在第10章中详细讨论）(index_split_004.html#p240)。以下是创建自定义异常所需的一切：

# 定义一个自定义异常

类 *<CustomExceptionName>* (Exception):

pass

您为您的异常提供一个唯一的名称。然后，您可以引发您的

在您的代码中创建自定义异常。这意味着您

您可以显式地按名称检查这些异常，位于更高级别的

代码。在接下来的章节中，我们将重写银行示例的代码，以便

我们会在银行和账户类中引发自定义异常并进行检查

并在主代码中报告错误。主代码会报告

错误，但允许程序继续运行。

在典型情况下，raise语句会导致当前函数或

方法退出并将控制权转移回调用者。如果调用者包含

如果存在一个except子句捕获异常，则执行将在该

except子句。否则，该函数或方法会退出。这个过程是

管理多个对象 **77**

直到某个except子句捕获该异常为止。控制权会回传到调用序列中，如果没有except子句捕获该异常，

异常时，程序退出并显示错误。

**在我们的银行程序中使用异常**

我们现在可以重写程序的所有三个层次（主程序、银行和账户）

是通过raise语句来表示错误，并使用try/except

块。

***使用异常的账户类***

示例4-9是重写的Account类的新版本，采用了异常处理

并进行了优化，确保没有重复的代码。我们首先定义一个自定义的

AbortTransaction 异常，如果发现某些错误，将会抛出该异常

当用户尝试在我们银行进行交易时。

**文件: BankOOP6_UsingExceptions/Account.py（已修改以适应即将到来的 Bank.py）**

# Account 类

# 由“raise”语句指示的错误

# 定义一个自定义异常

class AbortTransaction(Exception): 1

'''抛出此异常以中止银行交易'''

pass

class Account():

def __init__(self, name, balance, password):

self.name = name

self.balance = self.validateAmount(balance) 2

self.password = password

def validateAmount(self, amount):

try:

amount = int(amount)

except ValueError:

raise AbortTransaction('金额必须是整数') 3

如果金额 <= 0:

raise AbortTransaction('金额必须是正数') 4

return amount

def checkPasswordMatch(self, password): 5

如果密码 != self.password:

raise AbortTransaction('此账户的密码错误')

def deposit(self, amountToDeposit): 6

amountToDeposit = self.validateAmount(amountToDeposit)

self.balance = self.balance + amountToDeposit

return self.balance

def getBalance(self):

**78** 第四章

return self.balance

def withdraw(self, amountToWithdraw): 7

amountToWithdraw = self.validateAmount(amountToWithdraw)

如果 amountToWithdraw > self.balance:

raise AbortTransaction('你不能提取超过账户余额的金额')

self.balance = self.balance - amountToWithdraw

return self.balance

# 为调试添加

def show(self):

print(' 姓名:', self.name)

print(' 余额:', self.balance)

print(' 密码:', self.password)

*列表 4-9：修改后的 Account 类，抛出异常*

我们首先定义了我们的自定义 AbortTransaction 异常 1，以便我们可以

在此类及其他导入此类的代码中使用它。

在 Account 类的 __init__() 方法中，我们确保金额

提供的起始余额通过调用 validateAmount() 2 进行验证。这

方法使用 try/except 块来确保起始金额可以成功转换为

成功地将其转换为整数。如果 int() 调用失败，它会抛出一个

ValueError 异常，在 except 子句中捕获。与其仅仅

允许将通用的 ValueError 返回给调用者，代码中的

此 except 块 3 执行一个 raise 语句，抛出我们的 AbortTransaction 异常

异常，并包括一个更有意义的错误信息字符串。如果存在

如果版本转换为整数成功后，我们进行另一项测试。如果用户提供了一个

负金额时，我们也会抛出 AbortTransaction 异常 4，但会带有一个

不同的错误信息字符串。

checkPasswordMatch() 方法 5 由 Bank 类中的方法调用

对象检查用户提供的密码是否与该账户的密码匹配

保存在 Account 中。如果不符合要求，我们将执行另一个 raise 语句，并

相同的异常，但我们提供了一个更具描述性的错误信息字符串。

这样，deposit() 6 和 withdraw() 7 的代码得以简化，

因为这些方法假设金额已经经过验证且

在调用这些方法之前，密码必须先进行验证。还有一个附加检查

在withdraw()中检查，以确保用户不会试图提取超过账户余额的金额

比账户中的金额还要多；如果是，我们抛出AbortTransaction异常并附带

适当的描述。

因为该类中没有代码来处理AbortTransaction异常，

例外，如果抛出任何异常，控制会返回给调用者。如果调用者

如果没有代码处理异常，控制权将返回到预先设定的地方。

然后返回给前一个调用者，如此类推，直到回到调用栈的顶部。正如我们所见，主代码将

处理此异常。

***优化的银行类***

完整的银行类代码可供下载。在清单4-10中，我展示了

一些示范方法，演示了使用try/except技巧并调用

之前更新过的Account类中的方法。

管理多个对象 **79**

**文件：BankOOP6_UsingExceptions/Bank.py（已修改以与前面的Account.py一起使用）**

# 管理一个账户对象字典的银行

from Account import *

class 银行():

def __init__(self, hours, address, phone): 1

self.accountsDict = {}

self.nextAccountNumber = 0

self.hours = hours

self.address = address

self.phone = phone

def 请求有效账户号码(self): 2

账户号码 = input('您的账户号码是什么？')

try: 3

账户号码 = int(账户号码)

except ValueError:

raise AbortTransaction('账户号码必须是整数')

如果账户号码不在self.accountsDict中：

raise AbortTransaction('账户 ' + str(账户号码) + ' 不存在')

return 账户号码

def getUsersAccount(self): 4

账户号码 = self.askForValidAccountNumber()

oAccount = self.accountsDict[账户号码]

self.askForValidPassword(oAccount)

return oAccount

--- 省略其他方法 ---

def 存款(self): 5

print('*** 存款 ***')

oAccount = self.getUsersAccount()

存款金额 = input('请输入存款金额：')

当前余额 = oAccount.deposit(存款金额)

print('已存入：', 存款金额)

print('您的新余额为：', 当前余额)

def 提款(self): 6

print('*** 提款 ***')

oAccount = self.getUsersAccount()

用户金额 = input('请输入要提取的金额：')

当前余额 = oAccount.withdraw(用户金额)

print('已提款：', 用户金额)

print('您的新余额为：', 当前余额)

def 获取信息(self): 7

print('营业时间：', self.hours)

print('地址：', self.address)

print('电话：', self.phone)

print('我们当前有', len(self.accountsDict), '个账户开启。')

**80** 第4章

# 银行管理员专用方法

def 显示(self):

print('*** 显示 ***')

print('(这通常需要管理员密码)')

for 用户账户号码 in self.accountsDict:

oAccount = self.accountsDict[用户账户号码]

print('账户：', 用户账户号码)

oAccount.show()

print()

*清单4-10：修改后的银行类*

银行类从__init__()方法1开始，该方法保存所有相关的

信息存储在实例变量中。

新的askForValidAccountNumber()2方法从其他方法中调用，

其他方法要求用户输入账户号码，并尝试

验证给定的号码。首先，它有一个try/except块3，以确保

该号码是整数。如果不是，except块将错误检测为

ValueError异常，但通过抛出自定义对象来更清楚地报告错误

AbortTransaction异常带有描述性消息。接下来，它检查

确保给定的账户号码是银行已知的号码。如果

否则，它还会抛出一个AbortTransaction异常，但会给出不同的错误

消息字符串。

新的getUsersAccount()方法4首先调用先前的askForValid

AccountNumber()，然后使用账户号码查找相应的账户

请注意，此方法中没有try/except。如果发生异常

如果在askForValidAccountNumber()中（或在更低级别）抛出异常，该方法将

立即返回其调用者。

deposit() 5 和 withdraw() 6 方法在

同样，如果它们调用getUsersAccount()时抛出异常，

该方法将退出并将异常传递到调用者。如果所有

测试通过后，deposit()和withdraw()方法调用类似名称的方法

在指定的Account对象中执行实际交易的ods。

getInfo() 7方法报告有关银行的信息（营业时间，

地址、电话）并且不访问任何单独的账户。

***处理异常的主代码***

列表 4-11 显示了更新后的主代码，重写为处理自定义

异常。这是报告给用户的错误发生位置。

**文件：BankOOP6_UsingException/Main_Bank_Version6.py**

# 控制一个由账户组成的银行的主程序

from Bank import *

# 创建一个Bank类的实例

1 oBank = Bank('9到5', '123 主街，任意城，美国', '(650) 555-1212')

# 主代码

管理多个对象 **81**

2 while True:

print()

print('要查看账户余额，请按 b')

print('要关闭账户，请按 c')

print('要进行存款，请按 d')

print('要获取银行信息，请按 i')

print('要开设新账户，请按 o')

print('要退出，请按 q')

print('要显示所有账户，请按 s')

print('要进行提款，请按 w')

print()

action = input('你想做什么？')

action = action.lower()

action = action[0]  # 获取首字母

print()

3 try:

如果 action == 'b':

oBank.balance()

elif action == 'c':

oBank.closeAccount()

elif action == 'd':

oBank.deposit()

elif action == 'i':

oBank.getInfo()

elif action == 'o':

oBank.openAccount()

elif action == 'q':

break

elif action == 's':

oBank.show()

elif action == 'w':

oBank.withdraw()

4 except AbortTransaction as error:

# 输出错误消息文本

print(error)

print('完成')

*列表 4-11：处理错误的主代码（使用 try/except）*

主代码通过创建一个单一的Bank对象1开始。然后，在一个循环中，

它向用户展示一个顶层菜单，并询问他们希望进行的操作

它会为每个命令调用适当的方法。

这个列表中的重要内容是我们已经为所有调用oBank对象3的方法添加了一个try块。

这样，如果有任何问题，
