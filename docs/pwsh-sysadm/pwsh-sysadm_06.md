## 第五章：错误处理

![Images](img/common.jpg)

你已经看到如何使用变量和控制流结构编写灵活的代码，以应对现实世界中的不完美——比如服务器未按预期启动、文件放错了位置等。有些情况是你预料到的，并且可以相应地处理。但你永远无法预测每一个错误，总有些东西会导致代码崩溃。你能做的最好的事情，就是编写能够负责任地崩溃的代码。

这就是*错误处理*的基本前提，开发者使用这些技术确保他们的代码能预见并处理——或者*处理*——错误。在本章中，你将学习一些最基本的错误处理技术。首先，你将深入了解错误本身，看看终止性错误和非终止性错误的区别。然后，你将学习如何使用`try/catch/finally`结构，最后，你将了解 PowerShell 的自动错误变量。

### 处理异常和错误

在第四章中，你已经了解了控制流以及如何将不同的执行路径引入你的代码。当你的代码遇到问题时，它会打乱正常的执行流程；我们将这种打乱流程的事件称为*异常*。像除以零、尝试访问数组范围外的元素或尝试打开缺失的文件等错误，都会导致 PowerShell*抛出*异常。

一旦异常被抛出，如果你什么都不做来阻止它，它将被附加额外信息，并作为*错误*发送给用户。PowerShell 有两种类型的错误。第一种是*终止性错误*：任何会停止代码执行的错误。例如，假设你有一个名为*Get-Files.ps1*的脚本，它查找某个文件夹中的文件列表，然后对这些文件执行相同的操作。如果脚本找不到该文件夹——有人将其移动或重命名——你会希望返回一个终止性错误，因为代码在没有访问所有文件的情况下无法执行。但如果只有其中一个文件损坏了，怎么办呢？

当你尝试访问损坏的文件时，你会遇到另一个异常。但是因为你在每个文件上执行的是相同的独立操作，所以没有理由让一个损坏的文件阻止其余文件的运行。在这种情况下，你会编写代码，将由单个损坏文件引发的异常视为*非终止性错误*，即错误并不严重到足以停止其余代码的执行。

非终止性错误的一般错误处理行为是输出有用的错误信息，并继续执行程序的其余部分。你可以在 PowerShell 的几个内置命令中看到这一点。例如，假设你想检查 Windows 服务`bits`、`foo`和`lanmanserver`的状态。你可以使用一个`Get-Service`命令同时检查它们，如清单 5-1 所示。

```
PS> Get-Service bits,foo,lanmanserver
Get-Service : Cannot find any service with service name 'foo'.
At line:1 char:1
+ Get-Service bits,foo,lanmanserver
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo          : ObjectNotFound: (foo:String) [Get-Service], ServiceCommandException
+ FullyQualifiedErrorId : NoServiceFoundForGivenName,
                          Microsoft.PowerShell.Commands.GetServiceCommand

Status   Name               DisplayName
------   ----               -----------
Running  bits               Background Intelligent Transfer Ser...
Running  lanmanserver       Server
```

*清单 5-1：一个非终止性错误*

当然，根本没有`foo`服务，PowerShell 也会告诉你这一点。但请注意，PowerShell 会获取其他服务的状态；它在遇到这个错误时并不会停止执行。这个非终止性错误可以转换为终止性错误，以防止代码的其余部分执行。

重要的是要理解，决定将异常转为非终止性错误还是终止性错误是由开发者做出的。通常，如清单 5-1 中所示，这个决定是由编写你使用的 cmdlet 的人做出的。在许多情况下，如果 cmdlet 遇到异常，它会返回一个非终止性错误，将错误输出写入控制台，并允许你的脚本继续执行。在下一节中，你将看到几种将非终止性错误转换为终止性错误的方法。

### 处理非终止性错误

假设你想编写一个简单的脚本，进入一个你知道包含多个文本文件的文件夹，并打印出每个文本文件的第一行。如果文件夹不存在，你希望脚本立即结束并报告错误；否则，如果遇到其他任何错误，你希望脚本继续运行并报告错误。

你将开始编写一个脚本，应该返回一个终止性错误。清单 5-2 展示了这个代码的首次尝试。（虽然我本可以将代码简化为更简洁的形式，但为了教学目的，我尽力让每个步骤尽可能清晰。）

```
$folderPath = '.\bogusFolder'
$files = Get-ChildItem -Path $folderPath 
Write-Host "This shouldn't run."
$files.foreach({
    $fileText = Get-Content $files
    $fileText[0]
})
```

*清单 5-2：我们* Get-Files.ps1 *脚本的首次尝试*

在这里，你使用`Get-ChildItem`返回路径中包含的所有文件——在这种情况下，是一个虚假的文件夹。如果你运行这个脚本，你应该得到如下输出：

```
Get-ChildItem : Cannot find path 'C:\bogusFolder' because it does not exist.
At C:\Get-Files.ps1:2 char:10
+ $files = Get-ChildItem -Path $folderPath
+          ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo : ObjectNotFound: (C:\bogusFolder:String) [Get-ChildItem], ItemNotFoundException
    + FullyQualifiedErrorId : PathNotFound,Microsoft.PowerShell.Commands.GetChildItemCommand
This shouldn't run.
```

如你所见，会发生两件事：PowerShell 返回一个错误，指定遇到的异常类型（`ItemNotFoundException`），并且调用`Write-Host`也会运行。这意味着你得到的错误是非终止性的。

要将此错误转化为终止性错误，你将使用`ErrorAction`参数。这是一个*常见参数*，意味着它内置于每个 PowerShell cmdlet 中。`ErrorAction`参数决定了当 cmdlet 遇到非终止性错误时应该采取什么行动。该参数有五个主要选项：

**继续** 输出错误消息并继续执行 cmdlet。这是默认值。

**忽略** 继续执行 cmdlet，且不输出错误或将其记录在`$Error`变量中。

**询问** 输出错误消息并提示用户输入，然后继续执行。

**静默继续** 继续执行 cmdlet 而不输出错误，但将其记录在`$Error`变量中。

**停止** 输出错误消息并停止 cmdlet 的执行。

你将在本章稍后进一步了解`$Error`变量。现在，你想要将`Stop`传递给`Get-ChildItem`。更新你的脚本并再次运行代码。你应该会得到相同的输出，但没有`This shouldn't run.`（这不该运行）。

`ErrorAction`参数对于按情况控制错误行为非常有用。要更改 PowerShell 如何处理所有非终止性错误，可以使用`$ErrorActionPreference`变量，这是一个自动变量，用于控制默认的非终止性错误行为。默认情况下，`$ErrorActionPreference`设置为`Continue`。请注意，`ErrorAction`参数会覆盖`$ErrorActionPreference`的值。

通常来说，我认为最佳实践是始终将`$ErrorActionPreference`设置为`Stop`，以彻底去除非终止性错误的概念。这样可以让你捕获所有类型的异常，并避免事先知道哪些错误是终止性的，哪些是非终止性的。你也可以通过在每个命令上使用`ErrorAction`参数来更细致地定义哪些命令会返回终止性错误，但我更愿意一次性设置规则并忘记它，而不是每次调用命令时都要记得添加`ErrorAction`参数。

现在让我们看看如何使用`try/catch/finally`结构来处理终止性错误。

### 处理终止性错误

为了防止终止性错误停止程序，你需要*捕获*它们。你可以使用`try/catch/finally`结构来实现。列表 5-3 展示了语法。

```
try {
    # initial code
} catch {
    # code that runs if terminating error found
} finally {
    # code that runs at the end
}
```

*列表 5-3：try/catch/finally 结构的语法*

使用`try/catch/finally`基本上设置了一个错误处理的安全网。`try`块包含你想要运行的原始代码；如果发生终止性错误，PowerShell 会将流程重定向到`catch`块中的代码。无论`catch`块中的代码是否运行，`finally`块中的代码都会始终运行——请注意，`finally`块是可选的，不像`try`或`catch`。

为了更好地理解`try/catch/finally`能做什么和不能做什么，我们来重新审视一下我们的*Get-Files.ps1*脚本。你将使用`try/catch`语句来提供更清晰的错误信息，如列表 5-4 所示。

```
$folderPath = '.\bogusFolder'
try {
    $files = Get-ChildItem -Path $folderPath –ErrorAction Stop
    $files.foreach({
        $fileText = Get-Content $files
        $fileText[0]
    })
} catch {
    $_.Exception.Message
}
```

*列表 5-4：使用 try/catch 语句来处理终止性错误*

当捕获到终止错误时，错误对象会存储在`$_`变量中。在此示例中，你使用`$_.Exception.Message`来仅返回异常消息。在这种情况下，代码应该返回类似`无法找到路径 'C:\ bogusFolder'，因为该路径不存在`的内容。错误对象还包含其他信息，包括抛出异常的类型、显示异常抛出前代码执行历史的堆栈跟踪等。然而，现在最有用的信息是`Message`属性，因为它通常包含你需要查看代码发生了什么的基本信息。

到目前为止，你的代码应该按预期工作。通过将`Stop`传递给`ErrorAction`，你确保缺少文件夹时将返回终止错误并捕获该错误。但是，如果在尝试使用`Get-Content`访问文件时遇到错误会发生什么呢？

作为实验，尝试运行以下代码：

```
$filePath = '.\bogusFile.txt'
try {
    Get-Content $filePath
} catch {
    Write-Host "We found an error"
}
```

你应该从 PowerShell 收到错误信息，而不是你在`catch`块中编写的自定义错误。这是因为`Get-Content`在找不到项时返回的是非终止错误，而`try/catch`只能捕获终止错误。这意味着清单 5-4 中的代码将按预期工作——任何访问文件本身时出现的错误不会停止程序的执行，而是会返回控制台。

请注意，你在这段代码中没有使用`finally`块。`finally`块是执行必要清理任务（例如断开数据库连接、清理 PowerShell 远程会话等）的一好地方。在这里，没有必要进行这样的操作。

### 探索`$Error`自动变量

在本章中，你已强制 PowerShell 返回了许多错误。无论是终止错误还是非终止错误，每个错误都被存储在名为`$Error`的 PowerShell 自动变量中，按出现的时间顺序排列。

为了演示`$Error`变量，让我们打开控制台并运行一个你知道会返回非终止错误的命令（清单 5-5）。

```
PS> Get-Item -Path C:\NotFound.txt
Get-Item : Cannot find path 'C:\NotFound.txt' because it does not exist.
At line:1 char:1
+ Get-Item -Path C:\NotFound.txt
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo : ObjectNotFound: (C:\NotFound.txt:String) [Get-Item], ItemNotFoundException
    + FullyQualifiedErrorId : PathNotFound,Microsoft.PowerShell.Commands.GetItemCommand
```

*清单 5-5：示例错误*

现在，在相同的 PowerShell 会话中，检查`$Error`变量（清单 5-6）。

```
PS> $Error
Get-Item : Cannot find path 'C:\NotFound.txt' because it does not exist.
At line:1 char:1
+ Get-Item -Path C:\NotFound.txt
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo : ObjectNotFound: (C:\NotFound.txt:String) [Get-Item], ItemNotFoundException
    + FullyQualifiedErrorId : PathNotFound,Microsoft.PowerShell.Commands.GetItemCommand
--snip--
```

*清单 5-6：`$Error`变量*

除非你正在处理一个全新的会话，否则很可能你会看到一长串错误。要访问特定的错误，你可以像访问任何其他数组一样使用索引符号。`$Error`中的错误会被添加到数组的前面，因此`$Error[0]`是最新的，`$Error[1]`是第二新，依此类推。

### 摘要

PowerShell 中的错误处理是一个庞大的主题，本章仅涵盖了基础知识。如果你想深入了解，可以通过运行`Get-Help about_try_catch_finally`来查看`about_try_catch_finally`帮助文档。另一个很好的资源是由 Dave Wyatt 在 DevOps Collective 编写的*PowerShell 错误处理大全*（[`leanpub.com/thebigbookofpowershellerrorhandling`](https://leanpub.com/thebigbookofpowershellerrorhandling)*/*）。

这里的主要要点是理解终止错误和非终止错误之间的区别，`try/catch`语句的使用，以及各种`ErrorAction`选项，这些都将帮助你构建应对代码可能抛出的任何错误所需的技能。

到目前为止，你一直是在一个代码块中完成所有操作。在下一章中，你将看到如何将代码组织成独立的、可执行的单元，称为*函数*。
