## 第四章：控制流

![Images](img/common.jpg)

让我们快速回顾一下。在第三章中，你学习了如何通过使用管道和外部脚本来组合命令。在第二章中，你学习了如何使用变量来存储值。使用变量的一个主要好处是，它们让你能够编写处理值意义的代码：例如，不是直接处理数字 3，而是处理更通用的`$serverCount`，这样你就可以编写出无论有一台、两台还是一千台服务器都能运行的代码。将这种编写通用解决方案的能力与将代码存储在脚本中的能力相结合，使你能够在多台计算机上运行，从而开始解决更大规模的问题。

但是在现实世界中，有时候是否使用一台服务器、两台服务器或一千台服务器是很重要的。现在，你没有一个很好的方式来处理这个问题：你的脚本是单向执行的——从上到下——并且它们没有办法根据你正在处理的具体值进行改变。在本章中，你将使用控制流和条件逻辑来编写根据所处理的值执行不同指令序列的脚本。到本章结束时，你将学会如何使用`if/then`语句、`switch`语句以及各种循环语句，为你的代码提供所需的灵活性。

### 理解控制流

你将编写一个脚本，用来读取存储在多台远程计算机中的文件内容。为了跟上步骤，请从本书的资源下载一个名为*App_configuration.txt*的文件，网址是[*https://github.com/adbertram/PowerShellForSysadmins/*](https://github.com/adbertram/PowerShellForSysadmins/)，并将其放置在几台远程计算机的*C:\*驱动器根目录下。（如果你没有远程服务器的访问权限，现在可以仅通过文本跟随。）在这个示例中，我将使用名为`SRV1`、`SRV2`、`SRV3`、`SRV4`和`SRV5`的服务器。

要访问文件内容，你将使用`Get-Content`命令，并将文件的路径作为`Path`参数的参数提供，如下所示：

```
Get-Content -Path "\\servername\c$\App_configuration.txt"
```

作为第一次尝试，让我们把所有的服务器名称存储在一个数组中，并对数组中的每台服务器运行此命令。打开一个新的*.ps1*文件，并在 Listing 4-1 中输入代码。

```
$servers = @('SRV1','SRV2','SRV3','SRV4','SRV5')
Get-Content -Path "\\$($servers[0])\c$\App_configuration.txt"
Get-Content -Path "\\$($servers[1])\c$\App_configuration.txt"
Get-Content -Path "\\$($servers[2])\c$\App_configuration.txt"
Get-Content -Path "\\$($servers[3])\c$\App_configuration.txt"
Get-Content -Path "\\$($servers[4])\c$\App_configuration.txt"
```

*Listing 4-1：获取多台服务器上文件的内容*

从理论上讲，这段代码应该没有问题。但是这个示例假设你的环境中一切都很完美。如果`SRV2`宕机了怎么办？如果有人忘了把*App_configuration.txt*文件移动到`SRV4`上呢？或者用了不同的文件路径？你可以为每台服务器编写不同的脚本，但这种解决方案无法扩展——尤其是在你开始添加越来越多的服务器时。你需要的是可以根据遇到的情况执行不同的代码。

这就是*控制流*的基本概念，即根据预定的逻辑，让你的代码执行不同的指令序列。你可以将脚本看作沿着某一特定路径执行。目前，这条路径从代码的第一行直接走到最后一行，但你可以使用控制流语句在道路上增加分支、回到已经走过的地方，或者跳过某些行。通过为脚本引入不同的执行路径，你可以让脚本更加灵活，从而编写一个能够应对多种情况的脚本。

你将从最基本的控制流类型开始：条件语句。

### 使用条件语句

在第二章中，你了解了布尔值的概念：一个真假值。你可以使用布尔值来构建*条件语句*，该语句根据表达式（称为*条件*）的值是`True`还是`False`来决定是否执行特定的代码块。条件就像一个是/否问题：你有超过五台服务器吗？服务器 3 是否在线？这个文件路径是否存在？要开始使用条件语句，让我们看看如何将这些问题转化为表达式。

#### 使用运算符构建表达式

你可以使用*比较运算符*来编写表达式，这些运算符用来比较值。使用比较运算符时，将它放在两个值之间，如下所示：

```
PS> 1 –eq 1
True
```

你使用`–eq`运算符来判断两个值是否相等。以下是你将使用的最常见比较运算符列表：

**-eq** 比较两个值，如果它们相等，则返回`True`。

**-ne** 比较两个值，如果它们不相等，则返回`True`。

**-gt** 比较两个值，如果第一个值大于第二个，则返回`True`。

**-ge** 比较两个值，如果第一个值大于或等于第二个，则返回`True`。

**-lt** 比较两个值，如果第一个值小于第二个，则返回`True`。

**-le** 比较两个值，如果第一个值小于或等于第二个，则返回`True`。

**-contains** 如果第二个值“在”第一个值中，则返回`True`。你可以使用它来确定一个值是否在数组中。

PowerShell 提供了更高级的比较运算符。我现在不会详细介绍它们，但我建议你查阅微软文档中的[关于比较运算符](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_comparison_operators)*/*，或者在 PowerShell 帮助文档中查看（见第一章）。

你可以使用前面提到的操作符来比较变量和值。但一个表达式不一定非得是比较。有时 PowerShell 命令本身就可以作为条件。在之前的例子中，你想知道服务器是否在线。你可以通过使用 `Test-Connection` cmdlet 来测试服务器是否可以被 ping 通。通常，`Test-Connection` 的输出会返回一个充满信息的对象，但通过使用 `Quiet` 参数，你可以强制命令仅返回简单的 `True` 或 `False`，并通过 `Count` 参数限制测试次数为一次。

```
PS> Test-Connection -ComputerName offlineserver -Quiet -Count 1
False

PS> Test-Connection -ComputerName onlineserver -Quiet -Count 1
True
```

如果你想知道服务器是否离线，可以使用 `–not` 操作符将表达式转换为其相反的值：

```
PS> -not (Test-Connection -ComputerName offlineserver -Quiet -Count 1)
True
```

现在你已经了解了表达式的基础，接下来我们来看最简单的条件语句。

#### if 语句

`if` 语句很简单：如果 *X* 为真，则执行 *Y*。就是这么简单！

编写 `if` 语句时，你首先写 `if` 关键字，后面跟括号包含的条件。表达式之后是一个代码块，用大括号括起来。PowerShell 只有在表达式评估为 `True` 时才会执行该代码块。如果 `if` 表达式评估为 `False` 或根本没有返回任何内容，则跳过该代码块。你可以在清单 4-2 中看到基本的 `if/then` 语句的语法。

```
if (condition) {
    # code to run if the condition evaluates to be True
}
```

*清单 4-2：if 语句的语法*

这个例子使用了一些新的语法：井号 (`#`) 表示 *注释*，这段文本 PowerShell 会忽略。你可以使用注释来给自己或任何阅读代码的人留下有用的备注和说明。

现在让我们再看一遍清单 4-1 中的代码，看看如何使用 `if` 语句确保你不会尝试访问一个无法连接的服务器。在上一节中，你已经看到如何将 `Test-Connection` 用作返回 `True` 或 `False` 的表达式，所以接下来我们将把 `Test-Connection` 命令包装在一个 `if` 语句中，并使用 `Get-Content` 来避免访问无法连接的服务器。暂时只更改第一个服务器的代码，如清单 4-3 所示。

```
$servers = @('SRV1','SRV2','SRV3','SRV4','SRV5')
if (Test-Connection -ComputerName $servers[0] -Quiet -Count 1) {
    Get-Content -Path "\\$($servers[0])\c$\App_configuration.txt"
}
Get-Content -Path "\\$($servers[1])\c$\App_configuration.txt"
--snip--
```

*清单 4-3：使用 if 语句选择性地获取服务器内容*

由于你将 `Get-Content` 放入了 `if` 语句中，因此如果你尝试访问一个死掉的服务器，也不会遇到任何错误；如果测试失败，脚本会知道不再尝试读取文件。只有在你*已经*知道服务器在线时，才会尝试访问它。但请注意，这段代码只处理条件为真的情况。通常情况下，你可能希望在条件为真时有一种行为，在条件为假时有另一种行为。接下来的部分，你将看到如何使用 `else` 语句来指定假条件的行为。

#### else 语句

要为你的 `if` 语句添加备用行为，你需要在 `if` 语句块的右括号后面使用 `else` 关键字，然后再加上一对大括号，里面放置一个代码块。如 列表 4-4 所示，当第一个服务器没有响应时，使用 `else` 语句将错误返回到控制台。

```
if (Test-Connection -ComputerName $servers[0] -Quiet -Count 1) {
    Get-Content -Path "\\$($servers[0])\c$\App_configuration.txt"
} else {
    Write-Error -Message "The server $($servers[0]) is not responding!"
}
```

*列表 4-4：使用 else 语句在条件不成立时运行代码*

当你有两个互斥的情况时，`if/else` 语句非常有效。在这里，服务器要么在线，要么不在线；你只需要两条代码分支。让我们来看一下如何处理更复杂的情况。

#### `elseif` 语句

`else` 语句就像一个兜底语句：如果第一个 `if` 语句失败，不管怎样都会执行。对于一个二元条件，比如服务器是否在线，这个方法很有效。但有时你需要考虑更多的变数。例如，假设你有一台服务器，你知道它没有你想要的文件，而且你已经把这台服务器的名称存储在变量 `$problemServer` 中（这行代码需要你自己加到脚本中！）。这意味着你需要额外的检查，看看你正在处理的服务器是否是问题服务器。你可以通过使用嵌套的 `if` 语句来考虑这个情况，如下面的代码所示：

```
if (Test-Connection -ComputerName $servers[0] -Quiet -Count 1) {
    if ($servers[0] –eq $problemServer) {
        Write-Error -Message "The server $servers[0] does not have the right file!"
    } else {
        Get-Content -Path "\\$servers[0]\c$\App_configuration.txt"
    }
} else {
    Write-Error -Message "The server $servers[0] is not responding!"
}
--snip--
```

但是，更简洁的方式是使用 `elseif` 语句，它允许你在回退到 `else` 代码块之前添加额外的条件检查。`elseif` 语句的语法与 `if` 语句完全相同。因此，要使用 `elseif` 语句检查问题服务器，可以参考 列表 4-5 中的代码。

```
if (-not (Test-Connection -ComputerName $servers[0] -Quiet -Count 1)) { ❶
    Write-Error -Message "The server $servers[0] is not responding!"
} elseif ($servers[0] –eq $problemServer) { ❷
    Write-Error -Message "The server $servers[0] does not have the right file!"
} else {
    Get-Content -Path "\\$servers[0]\c$\App_configuration.txt" ❸
} 
--snip--
```

*列表 4-5：使用 elseif 语句块*

请注意，你不仅仅是添加了一个 `elseif`；你还改变了逻辑。现在你通过先使用 `–not` 运算符 ❶ 来检查服务器是否离线。然后，一旦确定服务器是否在线，你就检查它是否是问题服务器 ❷。如果不是，你就使用 `else` 语句执行默认行为——获取文件 ❸。如你所见，像这样的代码有多种结构方式。重要的是代码能够正常工作，并且对别人（无论是同事第一次阅读，还是你回头查看一段时间前写的脚本）来说具有可读性。

你可以根据需要将多个 `elseif` 语句串联起来，这样就能处理更多的情况。然而，`elseif` 语句是互斥的：当某个 `elseif` 语句的条件为 `True` 时，PowerShell 只会执行该语句块中的代码，而不会测试其他情况。在 列表 4-5 中，这并没有引发问题，因为你只需要在检查服务器是否在线之后，测试是否是问题服务器，但这点在以后编写代码时需要牢记。

`if`、`else` 和 `elseif` 语句非常适合处理简单的是/否问题。在下一部分，你将学习如何处理稍微复杂一点的逻辑。

#### switch 语句

让我们稍微调整一下我们的例子。假设你有五台服务器，*每台*服务器的文件路径不同。根据你现在所知道的，你需要为每个服务器写一个单独的 `elseif` 语句。这样是可以的，但有一种更简洁的方法。

请注意，现在你正在处理一种不同类型的条件。之前你想要的是对是/否问题的答案，而在这里，你想知道的是某个东西的具体值：服务器是 `SRV1` 吗？是 `SRV2` 吗？以此类推。如果你只处理一个或两个特定值，使用 `if` 就可以，但在这种情况下，使用 `switch` 语句会更简洁。

`switch` 语句允许你根据某个值执行不同的代码块。它由 `switch` 关键字和括号内的表达式组成。在 `switch` 块内有一系列的语句，每个语句后跟一个包含代码块的大括号，并最终有一个 `default` 块，像列表 4-6 那样。

```
switch (expression) {
    expressionvalue {
        # Do something with code here.
    }
    expressionvalue {
    }
    default {
        # Stuff to do if no matches were found
    }
}
```

*列表 4-6：switch 语句模板*

`switch` 语句可以包含（几乎）无限多个值。如果表达式计算出一个值，则执行该值块内的代码。关键是，与 `elseif` 不同，PowerShell 在执行完一个代码块后会继续评估其他条件，除非另有指定。如果没有任何值与计算出的值匹配，PowerShell 将执行 `default` 关键字下的嵌入代码。为了强制 PowerShell 停止评估 `switch` 语句中的条件，可以在代码块末尾使用 `break` 关键字，像列表 4-7 那样。

```
switch (expression) {
    expressionvalue {
        # Do something with code here.
        break
    }
--snip--
```

*列表 4-7：在 switch 语句中使用 break 关键字*

`break` 关键字可以用来使你的 `switch` 条件互斥。让我们回到我们五台服务器的例子，这些服务器的文件路径不同。你知道你正在处理的服务器只能有一个值（它不可能同时是 `SRV1` 和 `SRV2`），因此你必须使用 `break` 语句。你的脚本应该像列表 4-8 那样。

```
$currentServer = $servers[0]
switch ($currentServer) {
    $servers[0] {
        # Check if server is online and get content at SRV1 path.
        break
    }
    $servers[1] {
        ## Check if server is online and get content at SRV2 path.
        break
    }

    $servers[2] {
 ## Check if server is online and get content at SRV3 path.
        break
    }
--snip--
```

*列表 4-8：使用 switch 语句检查不同的服务器*

你可以通过仅使用 `if` 和 `elseif` 语句来重写这段代码（我鼓励你尝试一下！）。但无论你如何编写，它都会要求你为列表中的每台服务器重复相同的结构，这意味着你的脚本将变得相当长——而且如果你想要测试 500 台服务器而不是 5 台服务器，想想看会是什么样子。在下一部分，你将学习如何通过使用最基本的控制流结构之一——循环，来避免这种麻烦。

### 使用循环

计算机工作中的一个好法则：不要重复自己（DRY）。如果你发现自己做同样的事情超过一次，很可能有办法自动化它。编写代码也是如此：如果你反复使用相同的代码行，很可能有更好的解决方案。

避免重复代码的一种方法是使用循环。*循环*允许你反复执行代码，直到条件发生变化。*停止条件*可以用于使循环执行设定次数，直到布尔值发生变化，甚至使循环无限执行。我们称每次运行循环为*一次迭代*。

PowerShell 提供了五种类型的循环：`foreach`、`for`、`do/while`、`do/until`和`while`。本节解释了每种类型的循环，指出其独特之处，并强调了使用它的最佳场景。

#### `foreach`循环

我们将从你在 PowerShell 中最常用的循环类型开始，即`foreach`循环。`foreach`循环遍历一个对象列表，并对每个对象执行相同的操作，直到完成最后一个对象为止。这个对象列表通常由数组表示。当你在对象列表上运行循环时，我们称之为*迭代*该列表。

当你需要对许多不同但相关的对象执行相同任务时，`foreach`循环非常有用。让我们回到 Listing 4-1（这里复述一下）：

```
$servers = @('SRV1','SRV2','SRV3','SRV4','SRV5')
Get-Content -Path "\\$($servers[0])\c$\App_configuration.txt"
Get-Content -Path "\\$($servers[1])\c$\App_configuration.txt"
Get-Content -Path "\\$($servers[2])\c$\App_configuration.txt"
Get-Content -Path "\\$($servers[3])\c$\App_configuration.txt"
Get-Content -Path "\\$($servers[4])\c$\App_configuration.txt"
```

现在你将忽略前面一节中添加的所有复杂逻辑，并将其放入`foreach`循环中。但与 PowerShell 中的其他循环不同，`foreach`循环有三种使用方式：作为`foreach`语句、作为`ForEach-Object` cmdlet，或作为`foreach()`方法。虽然每种方式的使用类似，但你应该了解它们的区别。在接下来的三个部分中，你将通过使用每种类型的`foreach`循环来重写 Listing 4-1。

##### `foreach`语句

你将要看的第一个`foreach`类型是`foreach`语句。Listing 4-9 展示了 Listing 4-1 的循环版本。

```
foreach ($server in $servers) {
    Get-Content -Path "\\$server\c$\App_configuration.txt"
}
```

*Listing 4-9: 使用`foreach`语句*

如你所见，`foreach`语句后面跟着包含三个元素的括号，顺序是：一个变量、关键字`in`，以及要迭代的对象或数组。你提供的变量可以有任何名称，但我建议尽可能保持名称具有描述性。

当 PowerShell 遍历列表时，它会将正在查看的对象*复制*到变量中。请注意，由于变量只是副本，你不能直接更改原始列表中的项目。为了验证这一点，试试运行以下代码：

```
$servers = @('SRV1','SRV2','SRV3','SRV4','SRV5')
foreach ($server in $servers) {
    $server = "new $server"
}
$servers
```

你应该得到像这样的结果：

```
SRV1
SRV2
SRV3
SRV4
SRV5
```

什么都没改变！这是因为你只是在修改数组中原始变量的副本。这是使用`foreach`循环（任何类型）的一大缺点。要直接修改你正在遍历的列表的原始内容，你必须使用其他类型的循环。

##### `ForEach-Object` cmdlet

像 `foreach` 语句一样，`ForEach-Object` cmdlet 可以遍历一组对象并执行某个操作。但因为 `ForEach-Object` 是一个 cmdlet，所以你必须将那组对象和完成的操作作为参数传递。

查看清单 4-10，看看如何使用 `ForEach-Object` cmdlet 完成与清单 4-9 相同的操作。

```
$servers = @('SRV1','SRV2','SRV3','SRV4','SRV5')
ForEach-Object -InputObject $servers -Process {
    Get-Content -Path "\\$_\c$\App_configuration.txt"
}
```

*清单 4-10：使用 ForEach-Object cmdlet*

这里有一些不同，让我们一起来看一下。注意，`ForEach-Object` cmdlet 接受一个 `InputObject` 参数。在这个例子中，你使用的是 `$servers` 数组，但你可以使用任何对象，比如字符串或整数。在这些情况下，PowerShell 将只执行一次迭代。该 cmdlet 还接受一个 `Process` 参数，该参数应该是一个包含你希望在每个输入对象内元素上运行的代码的脚本块。（*脚本块* 是你传递给 cmdlet 作为一个整体的语句集合。）

你可能已经注意到清单 4-10 中另一个奇怪的地方。与使用 `foreach` 语句时使用 `$server` 变量不同，这里你使用了语法 `$_`。这种特殊语法表示管道中的当前对象。`foreach` 语句和 `ForEach-Object` cmdlet 之间的主要区别在于，cmdlet 接受管道输入。实际上，`ForEach-Object` 几乎总是通过管道传入 `InputObject` 参数来使用，如下所示：

```
$servers | ForEach-Object -Process {
    Get-Content -Path "\\$_\c$\App_configuration.txt"
}
```

`ForEach-Object` cmdlet 可以节省大量时间。

##### foreach() 方法

你将要查看的最后一种 `foreach` 循环是 PowerShell V4 引入的 `foreach()` 对象方法。`foreach()` 方法在 PowerShell 中所有数组上都存在，可以用来完成与 `foreach` 和 `ForEach-Object` 相同的操作。`foreach()` 方法接受一个脚本块参数，该参数应该包含要执行每次迭代的代码。与 `ForEach-Object` 一样，你可以使用 `$_` 来捕捉当前迭代的对象，正如你在清单 4-11 中看到的那样。

```
$servers.foreach({Get-Content -Path "\\$_\c$\App_configuration.txt"})
```

*清单 4-11：使用 foreach() 方法*

`foreach()` 方法比其他两种方法快得多，尤其在处理大型数据集时，差异尤为明显。我建议在可能的情况下优先使用这种方法。

`foreach` 循环非常适合你希望对每个对象逐个执行任务的情况。但如果你想做一些更简单的事情呢？如果你希望执行某个任务一定次数该怎么办？

#### for 循环

要执行预定次数的代码，你可以使用 `for` 循环。清单 4-12 显示了基本 `for` 循环的语法。

```
for (❶$i = 0; ❷$i -lt 10; ❸$i++) {
    ❹ $i
}
```

*清单 4-12：一个简单的 for 循环*

`for` 循环由四个部分组成：*迭代变量*声明 ❶，继续执行循环的条件 ❷，每次成功循环后对迭代变量进行的操作 ❸，以及你想要执行的代码 ❹。在这个例子中，你从将变量 `$i` 初始化为 0 开始。然后，你检查 `$i` 是否小于 10；如果是，就执行花括号中的代码，这会打印 `$ix`。代码执行后，你将 `$i` 增加 1 ❸，并检查它是否仍然小于 10 ❷。你重复这个过程，直到 `$i` 不再小于 10，从而完成 10 次迭代。

`for` 循环可以这样使用，执行某个任务任意次数——只需替换条件 ❷ 来满足你的需求。但 `for` 循环有更多的用途。最强大的用途之一是操作数组中的元素。之前，你已经看到如何*不能*使用 `foreach` 循环来修改数组中的元素。让我们再试一次，使用 `for` 循环：

```
$servers = @('SERVER1','SERVER2','SERVER3','SERVER4','SERVER5')
for ($i = 0; $i –lt $servers.Length; $i++) {
    $servers[$i] = "new $($servers[$i])"
}
$servers
```

尝试运行这个脚本。服务器名称应该会发生变化。

`for` 循环在执行需要多个数组元素的操作时也特别有用。例如，假设你的 `$servers` 数组按特定顺序排列，你想知道哪个服务器在另一个服务器之后。为此，你可以使用 `for` 循环：

```
for (❶$i = 1; $i –lt $servers.Length; $i++) {
    Write-Host $servers[$i] "comes after" $servers[$i-1]
}
```

请注意，这次你将迭代变量声明为从 1 开始 ❶。这确保你不会尝试访问第一个服务器之前的服务器，否则会导致错误。

正如你将在本书中看到的那样，`for` 循环是一个强大的工具，除了这里提供的简单示例外，它还有许多其他用途。现在，让我们继续讨论下一种类型的循环。

#### `while` 循环

`while` 循环是最简单的循环：只要条件为真，就执行某个操作。为了理解 `while` 循环的语法，让我们将 Listing 4-12 中的 `for` 循环重写为 Listing 4-13 所示。

```
$counter = 0
while ($counter -lt 10) {
    $counter
    $counter++
}
```

*Listing 4-13: 使用 while 循环的简单计数器*

如你所见，要使用 `while` 循环，只需将你想评估的条件放入括号内，将你想运行的代码放入花括号中。

`while` 循环最适合用于循环次数*不*预先确定的情况。例如，假设你有一台频繁宕机的 Windows 服务器（再次称为 `$problemServer`）。但你需要从中获取一个文件，而不想每隔几分钟就去测试服务器是否正常。你可以使用 `while` 循环来自动化这个过程，如 Listing 4-14 所示。

```
while (Test-Connection -ComputerName $problemServer -Quiet -Count 1) {
    Get-Content -Path "\\$problemServer\c$\App_configuration.txt"  
    break
}
```

*Listing 4-14: 使用 while 循环处理有问题的服务器*

通过使用 `while` 循环代替 `if`，你可以反复检查服务是否正常运行。然后，一旦获取到所需的内容，你可以使用 `break` 跳出循环，确保不继续检查服务器。`break` 关键字可以在任何循环中使用，用来停止循环的执行。这在使用最常见的 `while` 循环之一时尤其重要：`while($true)` 循环。通过使用 `$true` 作为条件，除非你通过 `break` 或键盘输入停止它，否则 `while` 循环会永远运行下去。

#### `do/while` 和 `do/until` 循环

与 `while` 循环类似，`do/while` 和 `do/until` 循环也是非常相似的。二者本质上是反向的：`do/while` 循环在条件为真时执行某个操作，而 `do/until` 循环则在条件为真时停止执行某个操作。

一个空的 `do/while` 循环看起来像这样：

```
do {
    } while ($true)
```

如你所见，`do` 代码位于 `while` 条件之前。`while` 循环和 `do/while` 循环之间的主要区别在于，`do/while` 循环会在条件评估之前先执行代码。

在某些情况下，这可能非常有用，特别是当你不断从一个来源接收输入并想要对其进行评估时。例如，假设你想提示用户询问他们最喜欢的编程语言。为了做到这一点，你可以使用 Listing 4-15 中的代码。在这里，你将使用 `do/until` 循环：

```
do {
    $choice = Read-Host -Prompt 'What is the best programming language?'
} until ($choice -eq 'PowerShell')
Write-Host -Object 'Correct!'
```

*Listing 4-15: 使用 do/until 循环*

`do/while` 和 `do/until` 循环非常相似。通常，这意味着你可以通过简单地反转条件，使用每种循环来完成相同的事情，正如你在这里所做的那样。

### 摘要

我们在本章中讲了很多内容。你学习了控制流，了解了如何使用条件逻辑在代码中引入不同的路径。你看到了各种类型的控制流语句，包括 `if` 语句、`switch` 语句，以及 `foreach`、`for` 和 `while` 循环。最后，你通过使用 PowerShell 检查服务器是否正常运行并访问服务器上的文件，获得了一些实践经验。

你可以使用条件逻辑来处理一些错误，但很可能会遗漏一些内容。在 第五章 中，你将更深入地了解错误以及一些处理错误的技巧。
