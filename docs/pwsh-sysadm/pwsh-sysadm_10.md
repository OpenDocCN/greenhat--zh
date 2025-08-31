## 第九章：使用 Pester 进行测试

![图片](img/common.jpg)

无法回避：你需要测试你的代码。很容易认为你的代码没有缺陷；但更容易的事是，证明你错了。当你使用 Pester 进行测试时，你可以停止假设，开始了解。

测试已经成为传统软件开发的一个特点，已经有几十年历史了。但是，尽管像*单元测试*、*功能测试*、*集成测试*和*验收测试*这样的概念对于经验丰富的软件开发人员来说很熟悉，但对于脚本编写者——那些想用 PowerShell 自动化但不拥有软件工程师职位的人来说，这些概念相对较新。由于许多组织越来越依赖 PowerShell 代码来运行关键的生产系统，我们将借鉴编程世界的经验，并将其应用到 PowerShell 中。

在本章中，你将学习如何为你的脚本和模块创建测试，这样你就可以确保代码正常工作，并在你更改代码时保持其稳定性。你将通过被称为 Pester 的测试框架来实现这一点。

### 引入 Pester

*Pester*是一个开源的 PowerShell 测试模块，能够在 PowerShell Gallery 中获取。由于其高效且用 PowerShell 编写，它已经成为 PowerShell 测试的事实标准。它允许你编写多种类型的测试，包括单元测试、集成测试和验收测试。如果这些测试名称让你感到陌生，别担心。在本书中，我们将仅使用 Pester 测试一些环境变化，比如虚拟机是否使用正确的名称创建、IIS 是否安装、操作系统是否正确安装。我们将这些测试称为*基础设施测试*。

我们不会涵盖如何测试诸如函数是否被调用、变量是否被正确设置或脚本是否返回特定对象类型之类的内容——这些都属于*单元测试*的范畴。如果你对 Pester 中的单元测试感兴趣，并想学习如何在不同情况下使用 Pester，可以查阅*Pester 书籍*（LeanPub，2019 年，*[`leanpub.com/pesterbook/`](https://leanpub.com/pesterbook/)*），该书几乎解释了你需要了解的所有关于 PowerShell 测试的内容。

### Pester 基础知识

要使用 Pester，你必须先安装它。如果你使用的是 Windows 10，Pester 默认已经安装，但如果你使用的是其他 Windows 操作系统，它也可以通过 PowerShell Gallery 获取。如果你使用的是 Windows 10，Pester 很可能已经过时了，因此你最好从 PowerShell Gallery 获取最新版本。由于 Pester 通过 PowerShell Gallery 提供，你可以运行`Install-Module -Name Pester`来下载并安装它。安装后，它将包含你需要的所有命令。

值得一提的是，你将使用 Pester 来编写和运行基础设施测试，目的是验证脚本对环境所做的任何预期更改。例如，在通过 `Test-Path` 创建一个新文件路径后，你可能会运行一个基础设施测试来确保文件路径已被创建。基础设施测试是一种保障措施，用来确认你的代码按预期执行了任务。

#### 一个 Pester 文件

在最基本的形式中，Pester 测试脚本由一个以 *.Tests.ps1* 结尾的 PowerShell 脚本组成。你可以随意命名主脚本；命名约定和测试结构完全由你决定。在这里，你将脚本命名为 *Sample.Tests.ps1*。

Pester 测试脚本的基本结构是一个或多个 `describe` 块，每个 `describe` 块包含（可选的）`context` 块，每个 `context` 块又包含 `it` 块，而每个 `it` 块包含断言。如果这听起来有点复杂， 列表 9-1 提供了一个视觉指南。

```
C:\Sample.Tests.ps1
    describe
        context
          it
            assertions
```

*列表 9-1：基础 Pester 测试结构*

让我们逐一了解这些部分。

#### `describe` 块

`describe` 块是一种将相似测试分组在一起的方法。在 列表 9-2 中，你创建了一个名为 `IIS` 的 `describe` 块，它可以用于包括所有测试 Windows 功能、应用池和网站的代码。

`describe` 块的基本语法是单词 `describe` 后跟一个名称（用单引号括起来），然后是一个开括号和闭括号。

```
describe 'IIS' {
}
```

*列表 9-2：Pester `describe` 块*

尽管这个结构看起来像是一个 `if/then` 条件，但不要被误导！这是一个传递给 `describe` 函数的脚本块。请注意，如果你是那种喜欢将大括号放在新行上的人，你可能会失望：开大括号必须与 `describe` 关键字在同一行。

#### `context` 块

一旦创建了 `describe` 块，你可以添加一个可选的 `context` 块。`context` 块将类似的 `it` 块组合在一起，这有助于在进行基础设施测试时组织测试。在 列表 9-3 中，你将添加一个 `context` 块，它将包含所有与 Windows 功能相关的测试。将测试按这种方式分类在 `context` 块中是个好主意，可以更轻松地管理它们。

```
describe 'IIS' {
    context 'Windows features' {
    }
}
```

*列表 9-3：Pester `context` 块*

虽然 `context` 块是可选的，但当你创建了数十个或数百个组件的测试时，它将变得极为重要！

#### `it` 块

现在让我们在 `context` 块中添加一个 `it` 块。`it` 块是一个更小的组件，用于标记实际的测试。其语法如 列表 9-4 所示，包含一个名称后跟一个块，就像你在 `describe` 块中看到的那样。

```
describe 'IIS' {
    context 'Windows features' {
        it 'installs the Web-Server Windows feature' {
        }
    }
}
```

*列表 9-4：带有 `context` 和 `it` 块的 Pester `describe` 块*

请注意，到目前为止，你更多的是为测试添加了不同的标签，并且这些标签的作用范围有所不同。在接下来的部分，你将添加实际的测试内容。

#### 断言

在 `it` 块内，你包含了一个或多个断言。*断言* 可以看作是实际的测试，或者是比较预期状态和实际状态的代码。Pester 中最常见的断言是 `should` 断言。`should` 断言有不同的运算符可以与之配合使用，如 `be`、`bein`、`belessthan` 等。如果你想查看完整的运算符列表，Pester 的 Wiki (*[`github.com/pester/Pester/wiki/`](https://github.com/pester/Pester/wiki/)*) 提供了完整的列表。

在我们的 IIS 示例中，我们来检查名为 `test` 的应用池是否已在我们的服务器上创建。为此，你首先需要编写代码来查找服务器上 `Web-Server` Windows 特性的当前状态（我们将其称为 `WEBSRV1`）。经过一些调查，浏览可用的 PowerShell 命令并筛选 `Get-WindowsFeature` 命令的帮助文档后，你发现完成此操作的代码如下：

```
PS> (Get-WindowsFeature -ComputerName WEBSRV1 -Name Web-Server).Installed
True
```

你知道，如果安装了 `Web-Server` 特性，`Installed` 属性将返回 `True`；否则，它将返回 `False`。知道这一点后，你可以断言，当你运行这个 `Get-WindowsFeature` 命令时，你期望 `Installed` 属性为 `True`。你想测试这个命令的输出是否*等于* `True`。你可以在 `it` 块中表示这种情况，如示例 9-5 所示。

```
describe 'IIS' {
    context 'Windows features' {
        it 'installs the Web-Server Windows feature' {
            $parameters = @{
 ComputerName = 'WEBSRV1'
                  Name         = 'Web-Server'
            }
            (Get-WindowsFeature @parameters).Installed | should -Be $true
        }
    }
}
```

*示例 9-5：使用 Pester 断言测试条件*

在这里，你创建了一个基础的 Pester 测试，用来测试 Windows 特性是否已安装。你首先输入你想要运行的测试，然后通过管道将测试结果传递给你的测试条件，在本例中是 `should be $true`。

编写 Pester 测试还有更多内容，我鼓励你通过 *《Pester 手册》* (*[`leanpub.com/pesterbook/`](https://leanpub.com/pesterbook/)*) 或者通过 4sysops 上的一系列文章 (*[`4sysops.com/archives/powershell-pester-testing-getting-started/`](https://4sysops.com/archives/powershell-pester-testing-getting-started/)*）来学习更多细节。这些内容应该足够让你理解我在本书中提供的测试。一旦你完成这本书，编写自己的 Pester 测试将是测试你 PowerShell 技能的一个好方法。

现在你有了一个 Pester 脚本。当然，一旦你有了脚本，你需要运行它！

### 执行 Pester 测试

使用 Pester 执行测试的最常见方法是使用 `Invoke-Pester` 命令。这个命令是 Pester 模块的一部分，允许测试者传递测试脚本的路径，Pester 然后会解释并执行该脚本，如示例 9-6 所示。

```
PS> Invoke-Pester -Path C:\Sample.Tests.ps1
Executing all tests in 'C:\Sample.Tests.ps1'

Executing script C:\Sample.Tests.ps1

  Describing IIS
    [+] installs the Web-Server Windows feature 2.85s
Tests completed in 2.85s
Tests Passed: 1, Failed: 0, Skipped: 0, Pending: 0, Inconclusive: 0
```

*示例 9-6：运行 Pester 测试*

你可以看到，`Invoke-Pester`命令已执行了*Sample.Tests.ps1*脚本，并提供了基本信息，如显示`describe`块的名称、测试结果，以及在该测试运行期间执行的所有测试的摘要。请注意，`Invoke-Pester`命令将始终显示每个执行的测试状态摘要。在本例中，`installs the Web-Server Windows feature`测试成功，通过`+`符号和绿色输出进行指示。

### 总结

本章介绍了 Pester 测试框架的基础知识。你已经下载、安装并构建了一个简单的 Pester 测试。这应该帮助你理解 Pester 测试的结构以及如何执行它。在接下来的章节中，你将反复使用这个框架。你将添加大量的`describe`块、`it`块和各种断言，但基本结构将保持相对不变。

这标志着我们第一部分的最后一章结束。你已经了解了在使用 PowerShell 编写脚本时需要用到的基本语法和概念。现在，让我们进入第二部分，开始动手实践，解决实际问题！
