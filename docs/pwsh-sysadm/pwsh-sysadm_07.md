## 第六章：编写函数

![图片](img/common.jpg)

到目前为止，你编写的代码相对单一：你的脚本只有单一的任务。虽然仅仅是一个能访问文件夹中文件的脚本并没有错，但当你编写更强大的 PowerShell 工具时，你会希望代码能做更多事情。没有什么能阻止你把更多内容塞进一个脚本中。你可以编写一千行代码，做数百个任务，所有这些都写在一个不间断的代码块中。但那个脚本将会是一个混乱，既难以阅读也难以操作。你可以将每个任务拆分成独立的脚本，但那样使用起来也会很混乱。你想要的是一个能够做很多事情的工具，而不是一百个只能做单一任务的工具。

为此，你将把每个任务分解为其自己的*函数*，一个标记的代码块，执行单一任务。函数只需要定义一次。你只需要编写代码来解决某个问题一次，将其存储在函数中，遇到这个问题时，你只需使用——或*调用*——解决问题的函数。函数显著提高了代码的可用性和可读性，使得代码更易于操作。在本章中，你将学习如何编写函数，添加和管理函数的参数，并设置函数以接受管道输入。但首先，让我们先了解一些术语。

### 函数与 cmdlet

如果函数的概念听起来很熟悉，那可能是因为它听起来有点像你在本书中一直使用的 cmdlet，例如`Start-Service`和`Write-Host`。这些也是被命名的代码块，用来解决单一问题。函数和 cmdlet 之间的区别在于*如何*创建这些构造。cmdlet 并不是用 PowerShell 编写的。它通常是用另一种语言编写的，通常像 C#，然后编译后在 PowerShell 中提供。另一方面，函数是用 PowerShell 简单的脚本语言编写的。

你可以使用`Get-Command` cmdlet 和它的`CommandType`参数查看哪些命令是 cmdlet，哪些是函数，如清单 6-1 所示。

```
PS> Get-Command –CommandType Function
```

*清单 6-1：显示可用的函数*

该命令将显示当前加载到 PowerShell 会话中的所有函数，或显示 PowerShell 可用的模块中的函数（第七章介绍了模块）。要查看其他函数，你必须将它们复制粘贴到控制台中，或将它们添加到可用模块中，或者*点源*它们（稍后我们也会讨论）。

既然这些问题已经解决，让我们开始编写函数吧。

### 定义一个函数

在使用函数之前，你需要先定义它。定义函数时，使用`function`关键字，后跟一个描述性的用户定义名称，再后跟一对大括号。在大括号内是你希望 PowerShell 执行的脚本块。清单 6-2 定义了一个基本的函数，并在控制台中执行它。

```
PS> function Install-Software { Write-Host 'I installed some software, Yippee!' }
PS> Install-Software
I installed some software, Yippee!
```

*示例 6-2：通过一个简单的函数向控制台输出消息*

你定义的函数`Install-Software`使用`Write-Host`在控制台显示一条消息。一旦定义，你可以使用这个函数的名称来执行其脚本块中的代码。

函数的名称很重要。你可以给你的函数起任何名字，但这个名字应当描述函数的功能。PowerShell 中的函数命名遵循动词-名词的语法，最佳实践是除非必要，否则始终使用这种语法。你可以使用`Get-Verb`命令查看推荐的动词列表。名词通常是你所处理的实体的单数形式——在这个例子中是软件。

如果你想改变函数的行为，你可以重新定义它，正如在示例 6-3 中所示。

```
PS> function Install-Software { Write-Host 'You installed some software, Yay!' }
PS> Install-Software
You installed some software, Yay!
```

*示例 6-3：重新定义 Install-Software 函数以改变其行为*

现在你重新定义了`Install-Software`，它会显示一个稍微不同的消息。

函数可以在脚本中定义，或者直接输入到控制台中。在示例 6-2 中，你定义了一个小函数，所以在控制台中定义它并没有问题。大多数情况下，你会有更大的函数，最好将这些函数定义在脚本或模块中，然后调用这个脚本或模块以将函数加载到内存中。正如你从示例 6-3 中可能想象到的，每次都要重新输入一个百行函数以调整其功能，可能会让人感到有些沮丧。

在本章的其余部分，你将扩展我们的`Install-Software`函数，使其接受参数并接受管道输入。我建议你在使用你最喜欢的编辑器时，将函数存储为*.ps1*文件，一边阅读本章内容一边操作。

### 向函数添加参数

PowerShell 函数可以有任意数量的参数。当你创建自己的函数时，你将有机会添加参数，并决定这些参数如何工作。参数可以是必需的，也可以是可选的，它们可以接受任何值，或者被限制只能接受一个有限列表中的某些参数。

例如，你通过`Install-Software`函数安装的软件可能有多个版本，但当前，`Install-Software`函数并没有提供让用户指定想要安装的版本的方式。如果只有你一个人在使用这个函数，你可以每次想要特定版本时重新定义这个函数——但那样会浪费时间，而且容易出错，更不用说你希望其他人也能使用你的代码了。

向函数引入参数使其具有变动性。就像变量让你能够编写处理同一情况多种版本的脚本一样，参数允许你编写一个函数，以多种方式完成同一任务。在本例中，你希望它能够安装同一软件的多个版本，并且在多台计算机上执行此操作。

让我们首先为函数添加一个参数，使你或用户能够指定要安装的版本。

#### 创建一个简单的参数

在函数上创建一个参数需要一个`param`块，这个块将包含所有函数的参数。你可以通过`param`关键字后跟圆括号来定义一个`param`块，如在示例 6-4 中所示。

```
function Install-Software {
    [CmdletBinding()]
    param()

    Write-Host 'I installed software version 2\. Yippee!' 
}
```

*示例 6-4：定义一个 param 块*

到此为止，你的函数的实际功能并没有改变。你只是在安装管道，为函数准备了一个参数。你将使用`Write-Host`命令来模拟软件安装，以便专注于编写函数。

**注意**

*在这本书的演示中，你将只构建*高级*函数。也有*基础*函数，但如今它们通常仅在一些小的、特定的场景中使用。两者的区别非常微妙，不便于详细讨论，但如果你在函数名下看到[CmdletBinding()]引用，或者看到一个参数被定义为[Parameter()]，你就知道你在使用的是高级函数。*

一旦你添加了`param`块，就可以通过将参数放入`param`块的圆括号内来创建参数，正如在示例 6-5 中所示。

```
function Install-Software {
     [CmdletBinding()]
     param(	
    ❶ [Parameter()]
    ❷ [string] $Version
    )

 ❸ Write-Host "I installed software version $Version. Yippee!" 
}
```

*示例 6-5：创建一个参数*

在`param`块内，你首先定义了`Parameter`块❶。像这里这样的空`Parameter`块什么都不做，但它是必需的（我将在下一节中解释如何使用它）。

让我们重点关注参数名称前的`[string]`类型❷。通过将参数类型放在方括号中，放置在参数变量名称之前，你可以将该参数转换为指定类型，这样 PowerShell 将始终尝试将传递给此参数的任何值转换为字符串——如果它还不是字符串的话。在这里，传递给`$Version`的任何内容都将始终被当作字符串处理。将参数转换为类型不是强制性的，但我强烈推荐这样做，因为明确地定义类型会显著减少未来的错误。

你还将`$Version`添加到打印语句❸中，这意味着当你运行带有`Version`参数的`Install-Software`命令并传递一个版本号时，你应该看到一条说明信息，如在示例 6-6 中所示。

```
PS> Install-Software -Version 2
I installed software version 2\. Yippee!
```

*示例 6-6：将参数传递给你的函数*

你现在已经为函数定义了一个有效的参数。让我们看看你可以如何使用这个参数。

#### 强制参数属性

你可以使用`Parameter`块来控制各种*参数属性*，这将允许你改变参数的行为。例如，如果你希望确保任何调用该函数的人都必须传入特定的参数，你可以将该参数定义为`Mandatory`。

默认情况下，参数是可选的。让我们通过在`Parameter`块中使用`Mandatory`关键字来强制用户传入版本，如 Listing 6-7 所示。

```
function Install-Software {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory)]
        [string]$Version
    )

    Write-Host "I installed software version $Version. Yippee!"
}
Install-Software
```

*Listing 6-7：使用强制参数*

如果你运行这个，你应该会看到以下提示：

```
cmdlet Install-Software at command pipeline position 1
Supply values for the following parameters:
Version:
```

一旦设置了`Mandatory`属性，在没有传入参数的情况下执行该函数将停止执行，直到用户输入一个值。该函数将等待直到用户为`Version`参数指定一个值，一旦他们输入了值，PowerShell 将执行该函数并继续执行。为了避免这个提示，只需在调用该函数时使用-ParameterName 语法传递该值——例如，`Install-Software -Version 2`。

#### 默认参数值

你还可以在定义参数时为其指定默认值。当你预计某个参数的大多数时候都会有特定值时，这非常有用。例如，如果你希望在 90%的情况下安装该软件的版本 2，并且不想每次运行该函数时都设置该值，你可以为`$Version`参数指定默认值`2`，如 Listing 6-8 所示。

```
function Install-Software {
    [CmdletBinding()]
    param(
        [Parameter()]
        [string]$Version = 2
    )

    Write-Host "I installed software version $Version. Yippee!"
}
Install-Software
```

*Listing 6-8：使用默认参数值*

拥有默认参数并不阻止你传入值。你传入的值将覆盖默认值。

#### 添加参数验证属性

除了让参数变为强制项并赋予默认值外，你还可以通过使用*参数验证属性*来限制它们的可选值。当可能时，限制用户（甚至你自己！）传递给函数或脚本的信息，将消除函数内部不必要的代码。例如，假设你向`Install-Software`函数传递了值 3，知道版本 3 是一个存在的版本。你的函数假设每个用户都知道哪些版本是存在的，因此它没有考虑当你尝试指定版本 4 时会发生什么。在这种情况下，函数将无法找到该版本的文件夹，因为它不存在。

在 Listing 6-9 中，你在文件路径中使用了`$Version`字符串。如果有人传入的值不能完整匹配现有的文件夹名称（例如，SoftwareV3 或 SoftwareV4），代码将会失败。

```
function Install-Software {
    param(
        [Parameter(Mandatory)]
        [string]$Version
    )
    Get-ChildItem -Path \\SRV1\Installers\SoftwareV$Version
}

Install-Software -Version 3
```

*Listing 6-9：假设参数值*

这将导致以下错误：

```
Get-ChildItem : Cannot find path '\\SRV1\Installers\SoftwareV3' because it does not exist.
At line:7 char:5
+     Get-ChildItem -Path \\SRV1\Installers\SoftwareV3
+     ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ObjectNotFound: (\\SRV1\Installers\SoftwareV3:String)
                              [Get-ChildItem], ItemNotFoundException
    + FullyQualifiedErrorId : PathNotFound,Microsoft.PowerShell.Commands.GetChildItemCommand
```

你可以编写错误处理代码来解决这个问题，或者通过要求用户仅传入已存在的软件版本来从根本上解决问题。为了限制用户的输入，你将添加参数验证。

存在多种类型的参数验证，但就你的 `Install-Software` 函数而言，`ValidateSet` 属性是最合适的。`ValidateSet` 属性允许你指定允许用于该参数的值列表。如果你只考虑字符串 1 或 2，你会确保用户只能指定这些值；否则，函数将立即失败并通知用户原因。

让我们在 `param` 块内添加参数验证属性，紧接在原始 `Parameter` 块下面，如列表 6-10 所示。

```
function Install-Software {
    param(
        [Parameter(Mandatory)]
        [ValidateSet('1','2')]
        [string]$Version
    )
    Get-ChildItem -Path \\SRV1\Installers\SoftwareV$Version
}

Install-Software -Version 3
```

*列表 6-10：使用 ValidateSet 参数验证属性*

你将项 1 和 2 的集合添加到 `ValidateSet` 属性的尾部括号内，这告诉 PowerShell，`Version` 的有效值只能是 1 或 2。如果用户尝试传递集合中没有的值，他们将收到错误信息（请参阅列表 6-11），通知他们只有特定数量的选项可用。

```
Install-Software : Cannot validate argument on parameter 'Version'. The argument "3" does not
belong to the set "1,2" specified by the ValidateSet attribute.
Supply an argument that is in the set and then try the command again.
At line:1 char:25
+ Install-Software -Version 3
+                         ~~~~
+ CategoryInfo          : InvalidData: (:) [Install-Software],ParameterBindingValidationException
    + FullyQualifiedErrorId : ParameterArgumentValidationError,Install-Software
```

*列表 6-11：传递一个不在 ValidateSet 块中的参数值*

`ValidateSet` 属性是一个常见的验证属性，但还有其他属性可用。要了解有关参数值如何受限的完整说明，请运行 `Get-Help about_Functions_Advanced_Parameters`，查看 `Functions_Advanced_Parameters` 帮助主题。

### 接受管道输入

到目前为止，你已经创建了一个函数，该函数的参数只能通过典型的 `-ParameterName <Value>` 语法传递。但是在第三章，你学到了 PowerShell 有一个管道，允许你无缝地将对象从一个命令传递到另一个命令。回想一下，一些函数没有管道功能——在使用自己的函数时，这是你可以控制的。让我们给 `Install-Software` 函数添加管道功能。

#### 添加另一个参数

首先，你需要向代码中添加另一个参数，用于指定你想要安装软件的计算机。你还需要将该参数添加到 `Write-Host` 命令中，以模拟安装。列表 6-12 添加了新参数：

```
function Install-Software {
    param(
        [Parameter(Mandatory)]
        [ValidateSet('1','2')],
        [string]$Version

        [Parameter(Mandatory)]
        [string]$ComputerName
    )
    Write-Host "I installed software version $Version on $ComputerName. Yippee!"

}

Install-Software -Version 2 -ComputerName "SRV1"
```

*列表 6-12：添加 ComputerName 参数*

就像 `$Version` 一样，你已经将 `ComputerName` 参数添加到 `param` 块中。

一旦你将 `ComputerName` 参数添加到函数中，你就可以遍历计算机名称列表，并将计算机名称和版本的值传递给 `Install-Software` 函数，如下所示：

```
$computers = @("SRV1", "SRV2", "SRV3")
foreach ($pc in $computers) {
    Install-Software -Version 2 -ComputerName $pc
}
```

但正如你已经看到的几次，你应该避免使用像这样的 `foreach` 循环，而是应该使用管道。

#### 使函数支持管道

不幸的是，如果你直接尝试使用管道，将会出现错误。在向函数添加管道支持之前，你应该决定希望函数接受哪种类型的管道输入。正如你在第三章中学到的，PowerShell 函数使用两种类型的管道输入：`ByValue`（整个对象）和`ByPropertyName`（单个对象属性）。在这里，由于我们的`$computers`列表只包含字符串，因此你将通过`ByValue`传递这些字符串。

要添加管道支持，你需要为你想要支持管道输入的参数添加一个参数属性，使用两个关键字之一：`ValueFromPipeline` 或 `ValueFromPipelineByPropertyName`，如示例 6-13 所示。

```
function Install-Software {
    param(
        [Parameter(Mandatory)]
        [string]$Version
        [ValidateSet('1','2')],
        [Parameter(Mandatory, ValueFromPipeline)]
        [string]$ComputerName
    )
    Write-Host "I installed software version $Version on $ComputerName. Yippee!"
}

$computers = @("SRV1", "SRV2", "SRV3")
$computers | Install-Software -Version 2
```

*示例 6-13：添加管道支持*

再次运行脚本，你应该得到如下结果：

```
I installed software version 2 on SRV3\. Yippee!
```

注意，`Install-Software`仅对数组中的最后一个字符串执行。你将在下一节中看到如何解决这个问题。

#### 添加一个 process 块

要告诉 PowerShell 对每个传入的对象执行此函数，必须包含一个`process`块。在`process`块内，放入你希望每次函数接收管道输入时执行的代码。按照示例 6-14 中的方式，向你的脚本添加一个`process`块。

```
function Install-Software {
    param(
        [Parameter(Mandatory)]
        [string]$Version
        [ValidateSet('1','2')],
        [Parameter(Mandatory, ValueFromPipeline)]
        [string]$ComputerName
    )
 process {
        Write-Host "I installed software version $Version on $ComputerName. Yippee!"
    }
}

$computers = @("SRV1", "SRV2", "SRV3")
$computers | Install-Software -Version 2
```

*示例 6-14：添加一个 process 块*

注意，`process`关键字后跟一对花括号，花括号内包含你的函数要执行的代码。

使用`process`块后，你应该能看到`$computers`中所有三台服务器的输出：

```
I installed software version 2 on SRV1\. Yippee!
I installed software version 2 on SRV2\. Yippee!
I installed software version 2 on SRV3\. Yippee!
```

`process`块应该包含你希望执行的主要代码。你还可以使用`begin`和`end`块来执行在函数调用开始和结束时运行的代码。有关构建高级函数的信息，包括`begin`、`process`和`end`块，请通过运行`Get-Help about_Functions_Advanced`查看`about_Functions_Advanced`帮助主题。

### 总结

函数允许你将代码模块化成独立的构建块。它们不仅帮助你将工作拆分成更小、更易于管理的部分，还迫使你编写可读和可测试的代码。当你为函数使用描述性名称时，代码变得自文档化，任何阅读它的人都能直观地理解它在做什么。

在本章中，你学习了函数的基础知识：如何定义它们，如何指定参数及其属性，以及如何接收管道输入。在下一章中，你将看到如何通过使用模块将多个函数打包在一起。
