## 第二章：基本 PowerShell 概念

![图片](img/common.jpg)

本章介绍了 PowerShell 中的四个基本概念：变量、数据类型、对象和数据结构。这些概念是几乎所有常见编程语言的基础，但 PowerShell 有一个独特之处：PowerShell 中的每一样东西都是一个对象。

这可能现在对你意义不大，但在你继续本章的学习时，请记住这一点。到本章结束时，你应该能明白这一点有多重要。

### 变量

*变量* 是存储 *值* 的地方。你可以把变量想象成一个数字盒子。当你想多次使用一个值时，例如，可以把它放在一个盒子里。然后，代替在代码中反复输入相同的数字，你可以将其放入变量中，并在需要该值时调用该变量。但正如你从名称中可能猜到的那样，变量的真正强大之处在于它可以改变：你可以往盒子里加东西，把盒子里的内容换成别的东西，或者拿出里面的东西展示一下，然后再把它放回去。

正如你将在本书后面看到的那样，这种可变性使你能够构建可以处理一般情况的代码，而不是针对某一个特定场景量身定制的代码。本节介绍了使用变量的基本方法。

#### 显示和更改变量

PowerShell 中的所有变量都以美元符号（`$`）开头，这表示你正在调用一个变量，而不是调用 cmdlet、函数、脚本文件或可执行文件。例如，如果你想显示 `MaximumHistoryCount` 变量的值，你必须在变量名前加上美元符号并调用它，如清单 2-1 所示。

```
PS> $MaximumHistoryCount
4096
```

*清单 2-1：调用 `$MaximumHistoryCount` 变量*

`$MaximumHistoryCount` 变量是一个内建变量，用于确定 PowerShell 在命令历史记录中保存的最大命令数；默认值为 4096 条命令。

你可以通过输入变量名（以美元符号开始），然后使用等号（`=`）和新值来更改变量的值，如清单 2-2 所示。

```
PS> $MaximumHistoryCount = 200
PS> $MaximumHistoryCount
200
```

*清单 2-2：更改 `$MaximumHistoryCount` 变量的值*

在这里，你将 `$MaximumHistoryCount` 变量的值更改为 `200`，这意味着 PowerShell 只会保存最近的 200 条命令。

清单 2-1 和 2-2 使用了已经存在的变量。PowerShell 中的变量大致分为两类：*用户定义的变量*，由用户创建，以及 *自动变量*，这些变量在 PowerShell 中已经存在。我们先来看一下用户定义的变量。

#### 用户定义的变量

在使用变量之前，变量需要先存在。尝试在 PowerShell 控制台中输入 $color，如清单 2-3 所示。

```
PS> $color
The variable '$color' cannot be retrieved because it has not been set.

At line:1 char:1
+ $color
+ ~~~~
 + CategoryInfo          : InvalidOperation: (color:String) [], RuntimeException
    + FullyQualifiedErrorId : VariableIsUndefined
```

*清单 2-3：输入未定义变量会导致错误。*

开启严格模式

如果你在列表 2-3 中没有遇到错误，且控制台没有显示任何输出，尝试运行以下命令以开启严格模式：

```
PS> Set-StrictMode -Version Latest
```

开启严格模式会告知 PowerShell，在你违反良好编码实践时抛出错误。例如，严格模式会强制 PowerShell 在你引用不存在的对象属性或未定义的变量时返回错误。在编写脚本时，开启此模式被认为是最佳实践，因为它强制你编写更简洁、更可预测的代码。而在 PowerShell 控制台中运行交互式代码时，通常不使用此设置。有关严格模式的更多信息，请运行`Get-Help Set-StrictMode -Examples`。

在列表 2-3 中，你尝试在`$color`变量尚不存在之前引用它，结果导致了一个错误。要创建一个变量，你需要*声明*它——即声明它存在——然后*赋值*给它（或*初始化*它）。你可以同时进行这两个操作，像列表 2-4 所示，创建一个值为`blue`的变量`$color`。你可以使用与改变`$MaximumHistoryCount`值相同的技巧来为变量赋值——输入变量名，后跟等号，再加上值。

```
PS> $color = 'blue'
```

*列表 2-4：创建一个值为`blue`的`color`变量*

一旦你创建了变量并为其赋值，你可以通过在控制台中键入变量名来引用它（如列表 2-5 所示）。

```
PS> $color
blue
```

*列表 2-5：检查变量的值*

变量的值不会改变，除非有某个操作或某人明确地改变它。你可以多次调用`$color`变量，它每次都会返回值`blue`，直到该变量被重新定义。

当你使用等号定义变量时（如列表 2-4 所示），你正在做的事情与使用`Set-Variable`命令是一样的。同样，当你在控制台中键入一个变量，它输出该值时，像列表 2-5 所示，你也在做的事情与使用`Get-Variable`命令是一样的。列表 2-6 通过这些命令重新创建了列表 2-4 和 2-5。

```
PS> Set-Variable -Name color -Value blue

PS> Get-Variable -Name color

Name                           Value
----                           -----
color                          blue
```

*列表 2-6：使用`Set-Variable`和`Get-Variable`命令创建变量并显示其值*

你也可以使用`Get-Variable`返回所有可用的变量（如列表 2-7 所示）。

```
PS> Get-Variable 

Name                           Value
----                           -----
$                              Get-PSDrive
?                              True
^                              Get-PSDrive
args                           {}
color                          blue
--snip--
```

*列表 2-7：使用`Get-Variable`返回所有变量。*

这个命令将列出当前内存中的所有变量，但请注意，有些变量是你尚未定义的。你将在下一节中查看这种类型的变量。

#### 自动变量

之前我介绍了自动变量，这些是 PowerShell 自己使用的预定义变量。虽然 PowerShell 允许你更改其中一些变量，就像你在示例 2-2 中所做的那样，但我通常不建议这样做，因为可能会引发意外后果。通常情况下，你应该将自动变量视为*只读*。 （现在可能是时候将`$MaximumHistoryCount`恢复为 4096 了！）

本节涵盖了一些你可能会使用的自动变量：`$null`变量、`$LASTEXITCODE`和偏好设置变量。

##### `$null`变量

`$null`变量是一个奇怪的变量：它代表“无”。将`$null`赋值给一个变量，可以让你创建该变量，但不为其分配实际的值，就像在示例 2-8 中所示。

```
PS> $foo = $null
PS> $foo
PS> $bar
The variable '$bar' cannot be retrieved because it has not been set.
At line:1 char:1
+ $bar
+ ~~~~
    + CategoryInfo          : InvalidOperation: (bar:String) [], RuntimeException
    + FullyQualifiedErrorId : VariableIsUndefined
```

*示例 2-8：将变量赋值为`$null`*

在这里，你将`$null`赋值给`$foo`变量。然后，当你调用`$foo`时，不会显示任何内容，但不会出现错误，因为 PowerShell 能够识别该变量。

你可以通过向`Get-Variable`命令传递参数来查看 PowerShell 识别的变量。在示例 2-9 中，你可以看到 PowerShell 知道`$foo`变量存在，但没有识别出`$bar`变量。

```
PS> Get-Variable -Name foo

Name                           Value
----                           -----
foo

PS> Get-Variable -Name bar
Get-Variable : Cannot find a variable with the name 'bar'.
At line:1 char:1
+ Get-Variable -Name bar
+ ~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (bar:String) [Get-Variable], ItemNotFoundException
    + FullyQualifiedErrorId : VariableNotFound,Microsoft.PowerShell.Commands.GetVariableCommand
```

*示例 2-9：使用`Get-Variable`查找变量*

你可能会好奇为什么我们需要将某些东西定义为`$null`。其实，`$null`是非常有用的。例如，正如你将在本章后面看到的，通常你会根据其他事情的结果为一个变量赋值，比如某个函数的输出。如果你检查该变量，发现其值仍然是`$null`，那就意味着函数出了问题，你可以据此采取相应的措施。

##### LASTEXITCODE 变量

另一个常用的自动变量是`$LASTEXITCODE`。PowerShell 允许你调用外部可执行应用程序，比如老式的*ping.exe*，它 ping 一个网站并获取响应。当外部应用程序运行结束时，它会以一个*退出代码*或*返回代码*结束，该代码表示一个消息。通常，0 表示成功，其他任何值都表示失败或其他异常。对于*ping.exe*，0 表示能够成功 ping 到一个节点，1 表示无法 ping 通。

当`*ping.exe*`运行时，正如在示例 2-10 中所示，你会看到预期的输出，但不会看到退出代码。这是因为退出代码被隐藏在`$LASTEXITCODE`中。`$LASTEXITCODE`的值始终是最后执行的应用程序的退出代码。示例 2-10 会 ping *[google.com](http://google.com)*，返回其退出代码，然后 ping 一个不存在的域名并返回其退出代码。

```
PS> ping.exe -n 1 dfdfdfdfd.com

Pinging dfdfdfdfd.com [14.63.216.242] with 32 bytes of data:
Request timed out.

Ping statistics for 14.63.216.242:
    Packets: Sent = 1, Received = 0, Lost = 1 (100% loss),
PS> $LASTEXITCODE
1
PS> ping.exe -n 1 google.com

Pinging google.com [2607:f8b0:4004:80c::200e] with 32 bytes of data:
Reply from 2607:f8b0:4004:80c::200e: time=47ms

Ping statistics for 2607:f8b0:4004:80c::200e:
    Packets: Sent = 1, Received = 1, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 47ms, Maximum = 47ms, Average = 47ms
PS> $LASTEXITCODE
0
```

*示例 2-10：使用*ping.exe*演示`$LASTEXITCODE`变量*

当你 ping *[google.com](http://google.com)* 时，`$LASTEXITCODE`为 0，而当你 ping 虚假的域名*dfdfdfdfd.com*时，`$LASTEXITCODE`的值为 1。

##### 偏好设置变量

PowerShell 有一种称为*首选项变量*的自动变量类型。这些变量控制着各种输出流的默认行为：`Error`、`Warning`、`Verbose`、`Debug` 和 `Information`。

你可以通过运行`Get-Variable`并过滤出所有以*Preference*结尾的变量来查找所有首选项变量，示例如下：

```
PS> Get-Variable -Name *Preference

Name                           Value
----                           -----
ConfirmPreference              High
DebugPreference                SilentlyContinue
ErrorActionPreference          Continue
InformationPreference          SilentlyContinue
ProgressPreference             Continue
VerbosePreference              SilentlyContinue
WarningPreference              Continue
WhatIfPreference               False
```

这些变量可用于配置 PowerShell 可以返回的各种类型的输出。例如，如果你曾经犯过错误并看到过那些难看的红色文本，那么你就见识过`Error`输出流。运行以下命令以生成错误信息：

```
PS> Get-Variable -Name 'doesnotexist'
Get-Variable : Cannot find a variable with the name 'doesnotexist'.
At line:1 char:1
+ Get-Variable -Name 'doesnotexist'
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (doesnotexist:String) [Get-Variable],
                              ItemNotFoundException
    + FullyQualifiedErrorId : VariableNotFound,Microsoft.PowerShell.Commands.GetVariableCommand
```

你应该看到类似的错误信息，因为这是`Error`流的默认行为。如果出于某种原因你不想被这些错误文本打扰，并希望什么都不发生，你可以将`$ErrorActionPreference`变量重新定义为`SilentlyContinue`或`Ignore`，这两种方式都会告诉 PowerShell 不输出任何错误文本：

```
PS> $ErrorActionPreference = 'SilentlyContinue'
PS> Get-Variable -Name 'doesnotexist'
PS>
```

如你所见，没有输出错误文本。忽略错误输出通常被认为是不好的做法，因此请在继续之前将`$ErrorActionPreference`的值改回`Continue`。有关首选项变量的更多信息，请通过运行 `Get-Help about_Preference_Variables` 查看 `about_help` 内容。

### 数据类型

PowerShell 变量有多种形式或*类型*。PowerShell 的数据类型的所有细节超出了本章的讨论范围。你需要知道的是，PowerShell 有几种数据类型——包括布尔值、字符串和整数——而且你可以在不产生错误的情况下更改变量的数据类型。以下代码应该能够顺利运行：

```
PS> $foo = 1
PS> $foo = 'one'
PS> $foo = $true
```

这是因为 PowerShell 可以根据你提供的值来推断数据类型。背后发生的事情有点复杂，不适合本书的讨论，但理解基本类型以及它们如何交互是很重要的。

#### 布尔值

几乎所有编程语言都使用*布尔值*，它们有真或假的值（1 或 0）。布尔值用于表示二元条件，比如开关的开或关。在 PowerShell 中，布尔值被称为*bools*，这两个布尔值分别由自动变量`$true`和`$false`表示。这些自动变量是硬编码到 PowerShell 中的，无法更改。列表 2-11 展示了如何将变量设置为`$true`或`$false`。

```
PS> $isOn = $true
PS> $isOn 
True
```

*列表 2-11：创建一个布尔变量*

你将在第四章中看到更多的布尔值。

#### 整数与浮点数

你可以通过整数或浮点数据类型在 PowerShell 中表示数字。

##### 整数类型

*整数*数据类型仅保存整数，并会将任何小数输入四舍五入为最接近的整数。整数数据类型分为*有符号*和*无符号*类型。有符号数据类型可以存储正数和负数；无符号数据类型只存储没有符号的数值。

默认情况下，PowerShell 使用 32 位有符号 `Int32` 类型来存储整数。位数决定了变量能够存储的数字的大小（或小），在这种情况下，范围是从 -2,147,483,648 到 2,147,483,647。对于超出此范围的数字，你可以使用 64 位有符号 `Int64` 类型，范围为 -9,223,372,036,854,775,808 到 9,223,372,036,854,775,807。

列表 2-12 展示了 PowerShell 如何处理 `Int32` 类型的示例。

```
❶ PS> $num = 1
   PS> $num
   1
❷ PS> $num.GetType().name
   Int32
❸ PS> $num = 1.5
   PS> $num.GetType().name
   Double
❹ PS> [Int32]$num
   2
```

*列表 2-12：使用 `Int` 类型存储不同的值*

让我们逐步了解每一个步骤。别担心所有的语法，现在关注输出。首先，你创建一个变量 `$num` 并赋值为 1 ❶。接下来，你检查 `$num` 的类型 ❷，发现 PowerShell 将 1 解释为 `Int32`。然后，你将 `$num` 改为持有一个小数值 ❸，再次检查类型，发现 PowerShell 已经将类型改为 `Double`。这是因为 PowerShell 会根据变量的值来更改其类型。但你可以通过*强制转换*该变量来让 PowerShell 将其视为某种特定类型，如你在最后通过在 `$num` 前加上 `[Int32]` 语法所做的那样 ❹。如你所见，当强制将 1.5 视为整数时，PowerShell 会将其四舍五入为 2。

现在让我们来看一下 `Double` 类型。

##### 浮动点类型

`Double` 类型属于称为*浮动点*变量的更广泛类别。尽管它们可以用于表示整数，但浮动点变量最常用于表示小数。浮动点变量的另一个主要类型是 `Float`。我不会深入讨论 `Float` 和 `Double` 类型的内部表示。你需要知道的是，虽然 `Float` 和 `Double` 能够表示小数，但这些类型可能会不精确，如 列表 2-13 所示。

```
PS> $num = 0.1234567910
PS> $num.GetType().name
Double
PS> $num + $num
0.2469135782 
PS> [Float]$num + [Float]$num
0.246913582086563
```

*列表 2-13：浮动点类型的精度错误*

如你所见，PowerShell 默认使用 `Double` 类型。但请注意，当你将 `$num` 与自身相加并强制将两者转换为 `Float` 类型时，结果会非常奇怪。原因超出了本书的范围，但要注意，使用 `Float` 和 `Double` 时可能会发生类似的错误。

#### 字符串

你已经见过这种类型的变量。当你在 列表 2-4 中定义 `$color` 变量时，你并没有直接输入 `$color = blue`。相反，你将值括在单引号中，这表示 PowerShell 该值是一串字母，或称为*字符串*。如果你尝试在没有引号的情况下将 `blue` 值赋给 `$color`，PowerShell 将返回一个错误：

```
PS> $color = blue
blue : The term 'blue' is not recognized as the name of a cmdlet, function, script file, or
operable program. Check the spelling of the name, or if a path was included, verify that the
path is correct and try again.
At line:1 char:10
+ $color = blue
+          ~~~~
    + CategoryInfo          : ObjectNotFound: (blue:String) [], CommandNotFoundException
    + FullyQualifiedErrorId : CommandNotFoundException
```

如果没有引号，PowerShell 会将 `blue` 解释为一个命令并尝试执行它。因为命令 `blue` 不存在，所以 PowerShell 会返回一个错误信息。要正确定义字符串，你需要在值周围使用引号。

##### 组合字符串和变量

字符串不限于单词，它们也可以是短语和句子。例如，你可以给`$sentence`赋值如下字符串：

```
PS> $sentence = "Today, you learned that PowerShell loves the color blue"
PS> $sentence
Today, you learned that PowerShell loves the color blue
```

但也许你想使用相同的句子，不过将*PowerShell*和*blue*作为变量的值。例如，假设你有一个叫做`$name`的变量，一个叫做`$language`的变量，和一个叫做`$color`的变量？列表 2-14 通过使用其他变量来定义这些变量。

```
PS> $language = 'PowerShell'
PS> $color = 'blue'

PS> $sentence = "Today, you learned that $language loves the color $color"
PS> $sentence
Today, you learned that PowerShell loves the color blue
```

*列表 2-14：在字符串中插入变量*

注意双引号的使用。将句子用单引号括起来并不能达到预期的效果：

```
PS> 'Today, $name learned that $language loves the color $color'
Today, $name learned that $language loves the color $color
```

这不仅仅是一个奇怪的错误。在 PowerShell 中，单引号和双引号之间有一个重要的区别。

##### 使用双引号与单引号

当你给一个变量赋值一个简单的字符串时，可以使用单引号或双引号，如列表 2-15 所示。

```
PS> $color = "yellow"
PS> $color
yellow
PS> $color = 'red'
PS> $color
red
PS> $color = ''
PS> $color
PS> $color = "blue"
PS> $color
blue
```

*列表 2-15：通过使用单引号和双引号来改变变量值*

如你所见，定义一个简单字符串时使用单引号或双引号都没有关系。那么，为什么当字符串中有变量时就变得重要了呢？答案与*变量插值*，或*变量扩展*有关。通常，当你单独输入`$color`并按下 ENTER 键时，PowerShell 会*插值*，或*扩展*那个变量。这些都是 fancy 的术语，意味着 PowerShell 正在读取变量中的值，或者说是打开箱子让你看里面的内容。当你使用双引号调用变量时，同样的事情会发生：变量会被扩展，正如你在列表 2-16 中看到的那样。

```
PS> "$color"
blue
PS> '$color'
$color
```

*列表 2-16：字符串中的变量行为*

但是注意当你使用单引号时会发生什么：控制台输出的是变量本身，而不是它的值。单引号告诉 PowerShell 你输入的就是*确切的*内容，无论是像*blue*这样的单词，还是看起来像一个叫做`$color`的变量。对 PowerShell 来说，这并不重要。它不会超越单引号中的值。所以，当你在单引号中使用变量时，PowerShell 不知道要扩展该变量的值。这就是为什么在将变量插入字符串时需要使用双引号的原因。

关于布尔值、整数和字符串还有很多内容要讲。但现在，让我们退后一步，看看一些更一般的内容：对象。

### 对象

在 PowerShell 中，*一切*都是对象。从技术上讲，*对象*是特定模板（称为类）的单个实例。*类*指定了一个对象将包含的内容。对象的类决定了它的*方法*，即可以对该对象执行的操作。换句话说，方法就是对象能做的所有事情。例如，一个列表对象可能有一个`sort()`方法，当调用时，它将排序该列表。同样，对象的类决定了它的*属性*，即对象的变量。你可以把属性看作是关于对象的所有数据。以列表对象为例，你可能会有一个`length`属性，它存储列表中元素的数量。有时，一个类会为对象的属性提供默认值，但更多情况下，这些值是你在工作中为对象提供的。

但这一切都是非常抽象的。让我们考虑一个例子：一辆车。车从设计阶段的一个计划开始。这个计划或模板定义了车应该是什么样子，应该有什么样的引擎，应该有什么样的底盘，等等。计划还规定了车完成后能够做什么——前进、倒退、开关天窗。你可以把这个计划看作是车的类。

每辆车都是从这个类构建的，并且该车所有特定的属性和方法都被添加到它身上。一辆车可能是蓝色的，而同一型号的车可能是红色的，另外一辆车可能有不同的变速箱。这些属性是特定车对象的属性。同样，每辆车都会前进、倒退，并且有相同的方法来开关天窗。这些操作就是车的方式。

现在，通过对对象如何工作的总体了解，让我们动手实践，使用 PowerShell。

#### 检查属性

首先，让我们创建一个简单的对象，这样你可以拆解它，揭示 PowerShell 对象的各个方面。列表 2-17 创建了一个名为`$color`的简单字符串对象。

```
PS> $color = 'red'
PS> $color
red
```

*列表 2-17：创建字符串对象*

请注意，当你调用`$color`时，你只会得到变量的值。但通常情况下，因为它们是对象，变量包含的信息不仅仅是它们的值。它们还有属性。

要查看对象的属性，你将使用`Select-Object`命令和`Property`参数。你将向`Property`传递一个星号参数，如列表 2-18 所示，以告诉 PowerShell 返回它找到的所有内容。

```
PS>  Select-Object -InputObject $color -Property *

Length
------
     3
```

*列表 2-18：调查对象属性*

如你所见，`$color`字符串只有一个属性，叫做`Length`。

你可以通过使用*点表示法*直接引用`Length`属性：你使用对象的名称，后跟一个点，再加上你想要访问的属性名称（见列表 2-19）。

```
PS> $color.Length
3
```

*列表 2-19：使用点表示法检查对象的属性*

像这样引用对象将随着时间的推移变得得心应手。

#### 使用 Get-Member cmdlet

使用 `Select-Object`，你发现 `$color` 字符串只有一个属性。但请记住，对象有时也包含方法。要查看此字符串对象上所有的 *方法* 和 *属性*，你可以使用 `Get-Member` cmdlet（清单 2-20）；这个 cmdlet 会是你很长一段时间里的好帮手。它是一个快速列出特定对象所有属性和方法的简便方法，统称为对象的 *成员*。

```
PS> Get-Member -InputObject $color

   TypeName: System.String

Name             MemberType            Definition
----             ----------            ----------
Clone            Method                System.Object Clone(), System.Object ICloneable.Clone()
CompareTo        Method                int CompareTo(System.Object value),
                                       int CompareTo(string strB), int IComparab...
Contains         Method                bool Contains(string value)
CopyTo           Method                void CopyTo(int sourceIndex, char[] destination,
                                       int destinationIndex, int co...
EndsWith         Method                bool EndsWith(string value),
                                       bool EndsWith(string value, System.StringCompari...
Equals           Method                bool Equals(System.Object obj),
                                       bool Equals(string value), bool Equals(string...
--snip--
Length           Property              int Length {get;}
```

*清单 2-20：使用 `Get-Member` 来调查对象的属性和方法*

现在我们开始深入探讨！事实证明，你的简单字符串对象有很多相关的方法。还有许多其他方法值得探索，但并不是所有都在这里展示。一个对象的方法和属性数量取决于它的父类。

#### 调用方法

你可以使用点符号来引用方法。然而，与属性不同，方法总是以一对开闭圆括号结尾，并且可以接受一个或多个参数。

例如，假设你想删除 `$color` 变量中的一个字符。你可以使用 `Remove()` 方法从字符串中删除字符。让我们通过 清单 2-21 中的代码，来聚焦 `$color` 的 `Remove()` 方法。

```
PS> Get-Member -InputObject $color –Name Remove
Name   MemberType Definition
----   ---------- ----------
Remove Method     string Remove(int startIndex, int count), string Remove(int startIndex)
```

*清单 2-21：查看字符串的 `Remove()` 方法*

如你所见，有两个定义。这意味着你可以通过两种方式使用该方法：一种是带有 `startIndex` 和 `count` 参数，另一种是仅使用 `startIndex`。

所以，要删除 `$color` 中的第二个字符，你需要指定你希望开始删除字符的位置，这个位置我们称之为 *索引*。索引从 0 开始，所以第一个字母的起始位置是 0，第二个字母的索引是 1，依此类推。通过索引，你还可以指定希望删除的字符数，方法是使用逗号分隔参数，如 清单 2-22 所示。

```
PS> $color.Remove(1,1)
Rd
PS> $color
red
```

*清单 2-22：调用方法*

使用索引 1，你告诉 PowerShell 从字符串的第二个字符开始删除；第二个参数告诉 PowerShell 只删除一个字符。所以你得到 `Rd`。但是请注意，`Remove()` 方法并不会永久改变字符串变量的值。如果你想保留这个更改，需要将 `Remove()` 方法的输出赋值给一个变量，如 清单 2-23 所示。

```
PS> $newColor = $color.Remove(1,1)
PS> $newColor
Rd
```

*清单 2-23：捕获字符串上 `Remove()` 方法的输出*

**注意**

*如果你需要知道某个方法是返回一个对象（如 Remove() 所做的）还是修改一个现有对象，你可以查看它的描述。如你所见，在示例 2-21 中，Remove() 的定义前面有一个字符串；这意味着该函数返回一个新的字符串。前面有 void 的函数通常会修改现有对象。第六章将更深入地讨论这个话题。*

在这些示例中，你使用了最简单的对象类型之一——字符串。在下一节中，你将了解一些更复杂的对象。

### 数据结构

*数据结构*是组织多个数据项的一种方式。与它们所组织的数据类似，PowerShell 中的数据结构由存储在变量中的对象表示。它们主要有三种类型：数组、ArrayList 和哈希表。

#### 数组

到目前为止，我把变量描述为一个盒子。但如果一个简单的变量（例如`Float`类型）是一个单独的盒子，那么*数组*就是一大堆用胶带粘在一起的盒子——由单个变量表示的一系列项目。

通常你需要几个相关的变量——比如一组标准颜色。与其将每个颜色存储为单独的字符串，然后引用这些单独的变量，不如将所有这些颜色存储在一个单一的数据结构中，这样效率更高。本节将向你展示如何创建、访问、修改和向数组中添加元素。

##### 定义数组

首先，让我们定义一个名为 `$colorPicker` 的变量，并将其赋值为一个包含四种颜色的字符串数组。为此，你使用 `@` 符号，后跟四个字符串（用逗号分隔）放在括号内，如示例 2-24 所示。

```
PS> $colorPicker = @('blue','white','yellow','black')
PS> $colorPicker
blue
white
yellow
black
```

*示例 2-24：创建数组*

`@`符号后面跟着一个左括号和零个或多个由逗号分隔的元素，表示你想要在 PowerShell 中创建一个数组。

注意，在调用 `$colorPicker` 后，PowerShell 会将数组的每个元素显示在新的一行中。在下一节中，你将学习如何单独访问每个元素。

##### 读取数组元素

要访问数组中的元素，你需要使用数组的名称，后跟一对方括号（`[]`），括号内包含你想访问的元素的索引。与字符串字符一样，数组的编号从 0 开始，所以第一个元素的索引是 0，第二个是 1，以此类推。在 PowerShell 中，使用 -1 作为索引将返回最后一个元素。

示例 2-25 访问我们 `$colorPicker` 数组中的多个元素。

```
PS> $colorPicker[0]
blue
PS> $colorPicker[2]
yellow
PS> $colorPicker[3]
black
PS> $colorPicker[4]
Index was outside the bounds of the array.
At line:1 char:1
+ $colorPicker[4]
+ ~~~~~~~~~~~~~~~
    + CategoryInfo          : OperationStopped: (:) [], IndexOutOfRangeException
    + FullyQualifiedErrorId : System.IndexOutOfRangeException
```

*示例 2-25：读取数组元素*

如你所见，如果你试图指定一个在数组中不存在的索引号，PowerShell 会返回一个错误信息。

要同时访问数组中的多个元素，你可以在两个数字之间使用*范围操作符*（`..`）。范围操作符会使 PowerShell 返回这两个数字以及它们之间的每一个数字，像这样：

```
PS> 1..3
1
2
3
```

要使用范围操作符访问数组中的多个项目，你需要使用一个索引范围，如下所示：

```
PS> $colorPicker[1..3]
white
yellow
black
```

现在你已经了解了如何访问数组中的元素，让我们来看看如何更改它们。

##### 修改数组中的元素

如果你想更改数组中的元素，你不需要重新定义整个数组。你可以通过其索引引用某个元素，并使用等号为其赋予一个新值，如 清单 2-26 所示。

```
PS> $colorPicker[3]
black
PS> $colorPicker[3] = 'white'
PS> $colorPicker[3]
white
```

*清单 2-26：修改数组中的元素*

在修改元素之前，确保通过将元素显示到控制台来仔细检查索引号是否正确。

##### 向数组添加元素

你可以使用加法操作符 (`+`) 向数组添加元素，如 清单 2-27 所示。

```
PS> $colorPicker = $colorPicker + 'orange'
PS> $colorPicker
blue
white
yellow
white
orange
```

*清单 2-27：向数组添加单个元素*

请注意，在等号两边你都输入了 `$colorPicker`。这是因为你要求 PowerShell 插值 `$colorPicker` 变量，然后添加一个新元素。

`+` 方法有效，但有一种更快捷、更易读的方法。你可以使用加号和等号一起形成 `+=`（见 清单 2-28）。

```
PS> $colorPicker += 'brown'
PS> $colorPicker
blue
white
yellow
white
orange
brown
```

*清单 2-28：使用 `+=` 快捷方式向数组添加元素*

`+=` 操作符告诉 PowerShell *将该项添加到现有数组中*。这个快捷方式可以避免你两次输入数组名，使用它比输入完整的语法要常见得多。

你还可以将数组添加到其他数组中。假设你想把粉色和青色添加到你的 `$colorPicker` 示例中。清单 2-29 定义了另一个仅包含这两种颜色的数组，并像在 清单 2-28 中一样将它们添加进来。

```
PS> $colorPicker += @('pink','cyan')
PS> $colorPicker
blue
white
yellow
white
orange
brown
pink
cyan
```

*清单 2-29：一次向数组添加多个元素*

一次添加多个元素可以节省你很多时间，尤其是在你创建一个包含大量元素的数组时。请注意，PowerShell 会将任何以逗号分隔的值集视为数组，你无需显式使用 `@` 或括号。

不幸的是，没有类似于 `+=` 的操作符来从数组中删除元素。从数组中删除元素比你想象的更复杂，我们在这里不会详细介绍。要了解原因，请继续阅读！

#### 数组列表（ArrayLists）

当你向数组添加元素时，会发生一些奇怪的事情。每次你向数组添加元素时，实际上是在通过旧的（插值过的）数组和新元素创建一个新的数组。当你从数组中删除元素时也会发生同样的事情：PowerShell 会销毁旧数组并创建一个新数组。这是因为 PowerShell 中的数组大小是固定的。当你改变它们时，无法修改大小，因此必须创建一个新数组。对于像我们之前使用的小数组，你可能不会注意到这种情况。但当你开始处理*庞大的*数组，包含数万或数十万个元素时，你会看到明显的性能下降。

如果你知道你必须从数组中添加或移除许多元素，我建议你使用另一种数据结构，叫做*ArrayList*。ArrayList 的行为几乎与典型的 PowerShell 数组相同，但有一个关键的区别：它们没有固定大小。它们可以动态调整以适应添加或移除的元素，在处理大量数据时提供更高的性能。

定义 ArrayList 与定义数组完全相同，只不过你需要将其转换为 ArrayList。示例 2-30 重新创建了颜色选择器数组，但将其转换为`System.Collections.ArrayList`类型。

```
PS> $colorPicker = [System.Collections.ArrayList]@('blue','white','yellow','black')
PS> $colorPicker
blue
white
yellow
black
```

*示例 2-30：创建 ArrayList*

与数组一样，当你调用一个 ArrayList 时，每个项都会显示在单独的一行上。

##### 向 ArrayList 添加元素

要向 ArrayList 添加或移除元素而不销毁它，你可以使用其方法。你可以使用`Add()`和`Remove()`方法来向 ArrayList 添加或移除项。示例 2-31 使用了`Add()`方法并将新元素放在方法的括号内。

```
PS> $colorPicker.Add('gray')
4
```

*示例 2-31：向 ArrayList 添加单个项*

注意输出：数字 4 是你添加的新元素的索引。通常，你不会使用这个数字，所以可以将`Add()`方法的输出发送到`$null`变量，避免其输出任何内容，如示例 2-32 所示。

```
PS> $null = $colorPicker.Add('gray')
```

*示例 2-32：将输出发送到`$null`*

有几种方法可以取消 PowerShell 命令的输出，但将输出分配给`$null`能提供最佳性能，因为`$null`变量无法重新赋值。

##### 从 ArrayList 中移除元素

你也可以以类似的方式移除元素，使用`Remove()`方法。例如，如果你想从 ArrayList 中移除值`gray`，可以将值放在方法的括号内，如示例 2-33 所示。

```
PS> $colorPicker.Remove('gray')
```

*示例 2-33：从 ArrayList 中移除项*

请注意，要移除一个项目，你不必知道索引号。你可以通过实际值引用元素——在这种情况下是`gray`。如果数组中有多个相同值的元素，PowerShell 将移除离 ArrayList 开头最近的元素。

在像这样的简单示例中很难看到性能差异。但是，ArrayList 在处理大数据集时比数组表现得更好。与大多数编程选择一样，你需要分析你的具体情况，来确定是使用数组还是 ArrayList 更合适。通常来说，你处理的项目集合越大，使用 ArrayList 越好。如果你处理的是少于 100 个元素的小数组，你几乎不会发现数组和 ArrayList 之间有什么区别。

#### 哈希表

当你只需要根据列表中的位置关联数据时，数组和 ArrayList 很有用。但有时你需要更加直接的方式：一种关联两段数据的方式。例如，你可能有一个用户名列表，想要将其匹配到真实姓名。在这种情况下，你可以使用 *哈希表*（或 *字典*），这是 PowerShell 中包含 *键值对* 列表的数据结构。你不使用数字索引，而是给 PowerShell 提供一个输入，称为 *键*，它返回与该键关联的 *值*。因此，在我们的例子中，你可以使用用户名作为哈希表的索引，它将返回该用户的真实姓名。

示例 2-34 定义了一个名为 `$users` 的哈希表，其中包含三名用户的信息。

```
PS> $users = @{
 abertram = 'Adam Bertram'
 raquelcer = 'Raquel Cerillo'
 zheng21 = 'Justin Zheng'
}
PS> $users
Name                           Value
----                           -----
abertram                       Adam Bertram
raquelcer                      Raquel Cerillo
zheng21                        Justin Zheng
```

*示例 2-34：创建哈希表*

PowerShell 不允许你定义具有重复键的哈希表。每个键必须唯一地指向一个值，而这个值可以是一个数组，甚至是另一个哈希表！

##### 从哈希表中读取元素

要访问哈希表中的特定值，你可以使用它的键。你可以通过两种方式来实现这一点。假设你想查找用户 `abertram` 的真实姓名，你可以使用 示例 2-35 中展示的任意两种方法。

```
PS> $users['abertram']
Adam Bertram
PS> $users.abertram
Adam Bertram
```

*示例 2-35：访问哈希表的值*

这两种选项有细微的差别，但现在你可以选择任何你偏好的方法。

示例 2-35 中的第二条命令使用了一个属性：`$users.abertram`。PowerShell 会将每个键添加到对象的属性中。如果你想查看哈希表中的所有键和值，可以访问 `Keys` 和 `Values` 属性，如 示例 2-36 所示。

```
PS> $users.Keys
abertram                       
raquelcer                      
zheng21                        
PS> $users.Values
Adam Bertram
Raquel Cerillo
Justin Zheng
```

*示例 2-36：读取哈希表的键和值*

如果你想查看哈希表（或任何对象）的 *所有* 属性，你可以运行以下命令：

```
PS> Select-Object -InputObject $yourobject -Property *
```

##### 添加和修改哈希表项

要向哈希表中添加元素，你可以使用 `Add()` 方法，或者通过使用方括号和等号创建一个新索引。两种方法都在 示例 2-37 中展示。

```
PS> $users.Add('natice', 'Natalie Ice')
PS> $users['phrigo'] = 'Phil Rigo'
```

*示例 2-37：向哈希表添加项*

现在你的哈希表存储了五个用户。如果你需要修改哈希表中的某个值，应该怎么办？

当你修改哈希表时，最好检查你想要修改的键值对是否存在。要检查一个键是否已经存在于哈希表中，你可以使用 `ContainsKey()` 方法，这是 PowerShell 中每个哈希表的组成部分。当哈希表中包含该键时，它将返回 `True`；否则，返回 `False`，如 示例 2-38 所示。

```
PS> $users.ContainsKey('johnnyq')
False
```

*示例 2-38：检查哈希表中的项*

一旦你确认键存在于哈希表中，你可以通过使用一个简单的等号来修改其值，如 示例 2-39 所示。

```
PS> $users['phrigo'] = 'Phoebe Rigo'
PS> $users['phrigo']
Phoebe Rigo
```

*示例 2-39：修改哈希表值*

如你所见，你可以通过几种方式向哈希表中添加项目。正如你将在下一部分看到的，删除哈希表中的项只有一种方法。

##### 从哈希表中删除项

与 ArrayList 一样，哈希表有一个 `Remove()` 方法。只需调用它并传入你想删除项的键值，如清单 2-40 所示。

```
PS> $users.Remove('natice')
```

*清单 2-40：从哈希表中删除一项*

你的一个用户应该已经不在了，但你可以调用哈希表来进行双重检查。记住，你可以使用 `Keys` 属性来提醒自己任何键的名称。

### 创建自定义对象

到目前为止，在本章中，你一直在创建和使用 PowerShell 内置的对象类型。大多数时候，你可以使用这些类型，避免自己创建对象。但有时候，你需要创建一个自定义对象，定义你自己的属性和方法。

清单 2-41 使用 `New-Object` cmdlet 定义了一个 `PSCustomObject` 类型的新对象。

```
PS> $myFirstCustomObject = New-Object -TypeName PSCustomObject
```

*清单 2-41：使用 `New-Object` 创建自定义对象*

这个例子使用了 `New-Object` 命令，但你也可以通过使用等号和强制转换来做同样的事情，如清单 2-42 所示。你定义一个哈希表，其中键是属性名称，值是属性值，然后将其强制转换为 `PSCustomObject`。

```
PS> $myFirstCustomObject = [PSCustomObject]@{OSBuild = 'x'; OSVersion = 'y'}
```

*清单 2-42：使用 `PSCustomObject` 类型加速器创建自定义对象*

请注意，清单 2-42 使用分号（`；`）来分隔键和值的定义。

一旦你有了自定义对象，你就可以像使用任何其他对象一样使用它。清单 2-43 将我们的自定义对象传递给 `Get_Member` cmdlet 来检查它是否为 `PSCustomObject` 类型。

```
PS> Get-Member  -InputObject $myFirstCustomObject

   TypeName: System.Management.Automation.PSCustomObject

Name        MemberType   Definition
----        ----------   ----------
Equals      Method       bool Equals(System.Object obj)
GetHashCode Method       int GetHashCode()
GetType     Method       type GetType()
ToString    Method       string ToString()
OSBuild     NoteProperty string OSBuild=OSBuild
OSVersion   NoteProperty string OSVersion=Version
```

*清单 2-43：调查自定义对象的属性和方法*

如你所见，你的对象已经有了一些预先存在的方法（例如，其中一个返回对象的类型！），以及你在清单 2-42 中创建对象时定义的属性。

让我们通过使用点表示法来访问这些属性：

```
PS> $myFirstCustomObject.OSBuild
x
PS> $myFirstCustomObject.OSVersion
y
```

看起来不错！在本书的其余部分，你会经常使用 `PSCustomObject` 对象。它们是强大的工具，让你能够创建更加灵活的代码。

### 总结

到目前为止，你应该对对象、变量和数据类型有一个大致的了解。如果你仍然不理解这些概念，请重新阅读这一章。这是我们将要讨论的最基础的内容之一。对这些概念有一个高层次的理解将使得本书的其他部分更容易理解。

下一章将介绍两种在 PowerShell 中组合命令的方法：管道和脚本。
