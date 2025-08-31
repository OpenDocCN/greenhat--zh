# 第七章：绘制函数

![Image](img/common.jpg)

类型为`R -> R`的函数是可以在图表上绘制的函数。本章将展示如何绘制这类函数。绘图工具不属于 Prelude 的一部分，因此我们将首先讨论如何安装和使用库模块。

### 使用库模块

有一些别人写的函数是我们希望使用的，但它们并不包含在 Prelude 中。然而，这些函数存在于可以导入到源代码文件或直接加载到 GHCi 中的库模块中。GHC（我们使用的格拉斯哥 Haskell 编译器）自带一套标准的库模块，但其他模块则需要安装。

#### 标准库模块

`Data.List`是标准库模块之一。它包含用于处理列表的函数。要将其加载到 GHCi 中，可以使用`:module`命令（简写为`:m`）。

```
Prelude> :m Data.List
```

现在，我们可以使用此模块中的函数，比如`sort`。

```
Prelude Data.List> :t sort
sort :: Ord a => [a] -> [a]
Prelude Data.List> sort [7,5,6]
[5,6,7]
```

请注意，通常显示`Prelude>`的 GHCi 提示符已经扩展，包含了我们刚刚加载的模块的名称。

若要在源代码文件中使用`sort`函数，可以在文件中包含以下行：

```
import Data.List
```

在源代码文件的顶部。

标准库的文档可以在线访问，网址是[*https://www.haskell.org*](https://www.haskell.org)，点击 Documentation 然后选择 Library Documentation，或者你也可以直接访问[`downloads.haskell.org/~ghc/latest/docs/html/libraries/index.html`](https://downloads.haskell.org/~ghc/latest/docs/html/libraries/index.html)。

#### 其他库模块

标准库之外的库模块被组织成*包*。附录中描述了如何安装 Haskell 库包。每个包包含一个或多个模块。对于本章中的绘图，我们需要`Graphics.Gnuplot.Simple`模块，该模块由`gnuplot`包提供。

按照附录中的说明安装 gnuplot。安装过程需要几个步骤。安装结束时会执行如下命令：

```
$ cabal install gnuplot
```

或者

```
$ stack install gnuplot
```

安装完`gnuplot`包后，你可以重启 GHCi 并将`Graphics.Gnuplot.Simple`模块加载到 GHCi 中，方法如下：

```
Prelude Data.List> :m Graphics.Gnuplot.Simple
```

在开始下一节之前，让我们卸载`Graphics.Gnuplot.Simple`模块，这样我们就可以从干净的状态开始：

```
Prelude Graphics.Gnuplot.Simple> :m
```

执行`:m`命令而不带任何模块名称将清除所有已加载的模块。

### 绘图

有时你可能需要快速绘制一个图形，以查看一个函数的形态。下面是使用 GHCi 进行绘制的示例：

```
Prelude> :m Graphics.Gnuplot.Simple
Prelude Graphics.Gnuplot.Simple> plotFunc [] [0,0.1..10] cos
```

第一个命令加载一个可以绘制图形的图形模块。第二个命令绘制从 0 到 10 的`cos`函数，增量为 0.1。这个操作通过`plotFunc`函数实现，`plotFunc`是`Graphics.Gnuplot.Simple`模块提供的函数之一。`plotFunc`函数接受一组属性（在这里是空列表`[]`），一组计算函数的值（在这里是`[0,0.1..10]`，这是从 0 到 10 的 101 个数字，增量为 0.1），以及一个待绘制的函数（在这里是`cos`）。

100 个点通常足以得到一个平滑的图形。如果对平滑度要求更高，你可以使用 500 个点或更多。如果只使用 4 个点，你将无法得到平滑的图形（试试看，看看会发生什么）。在第十一章中，我们将学习如何为演示或作业制作一个带有标题和轴标签的漂亮图形。

如果你希望绘制一个在程序文件中定义的函数，你有几种选择：

+   只在程序文件中放入你想绘制的函数。

+   使用程序文件导入绘图模块，并定义你想要绘制的函数。

+   使用程序文件导入绘图模块，定义你想要绘制的函数，并定义图形。

我们将依次探索这些选项。

#### 仅函数

假设我们想要绘制在第二章中定义的`square`函数，从*x* = –3 到*x* = 3。让我们卸载`Graphics.Gnuplot.Simple`模块，以便从一个干净的状态开始：

```
Prelude Graphics.Gnuplot.Simple> :m
```

现在，我们执行以下命令序列：

```
Prelude> :m Graphics.Gnuplot.Simple
Prelude Graphics.Gnuplot.Simple> :l first.hs
[1 of 1] Compiling Main            ( first.hs, interpreted )
Ok, one module loaded.
*Main Graphics.Gnuplot.Simple> plotFunc [] [-3,-2.99..3] square
```

第一个命令加载绘图模块，第二个命令加载包含函数定义的文件，第三个命令绘制图形。使用`:module`命令会清除之前使用`:load`命令加载的任何源代码文件，因此必须在加载源代码文件之前先加载模块。

#### 函数和模块

如果我们知道程序文件包含我们希望绘制的函数，我们可以在程序文件中导入`Graphics.Gnuplot.Simple`模块，这样我们就不需要在 GHCi 命令行中执行了。我们可以在程序文件顶部添加以下代码，而不必在 GHCi 中输入`:m Graphics.Gnuplot.Simple`：

```
import Graphics.Gnuplot.Simple
```

假设这个扩展的程序文件叫做*firstWithImport.hs*。让我们从卸载文件和模块开始，清理一下：

```
*Main Graphics.Gnuplot.Simple> :l
Ok, no modules loaded.
Prelude Graphics.Gnuplot.Simple> :m
```

在没有文件名的情况下执行`:l`命令将清除已加载的程序文件，但会保留任何已加载的模块。

现在在 GHCi 中我们执行以下操作：

```
Prelude> :l firstWithImport.hs
[1 of 1] Compiling Main            ( firstWithImport.hs, interpreted )
Ok, one module loaded.
*Main> plotFunc [] [-3,-2.99..3] square
```

你应该会看到你在上一节中看到的相同图形。

#### 函数、模块和绘图定义

如果我们提前知道想要的图形，我们可以将绘图命令包含在程序文件中。在我们的源代码文件中，我们将包含`import`命令，

```
import Graphics.Gnuplot.Simple
```

定义类型`R`的类型同义词，

```
type R = Double
```

我们将绘制的函数，

```
square :: R -> R
square x = x**2
```

我们想要的图形，

```
plot1 :: IO ()
plot1 = plotFunc [] [-3,-2.99..3] square
```

注意`plot1`的类型`IO ()`（读作“eye oh unit”）。`IO`代表输入/输出，它表示一个具有副作用的非纯函数的类型。在这种情况下，副作用是图形在屏幕上弹出。任何类型为`IO ()`的内容，都是仅为了其副作用而执行的，而不是因为我们期待返回一个值。

让我们在 GHCi 中清理一下工作区。

```
*Main> :l
Ok, no modules loaded.
Prelude> :m
```

如果源代码文件名为*QuickPlotting.hs*，我们只需加载文件并给出我们的图形名称。

```
Prelude> :l QuickPlotting.hs
[1 of 1] Compiling Main            ( QuickPlotting.hs, interpreted )
Ok, one module loaded.
*Main> plot1
```

你应该再次看到图形。

### 总结

本章介绍了库模块，包括标准库模块以及需要安装的模块。我们安装了`gnuplot`包，该包提供了`Graphics.Gnuplot.Simple`模块，并展示了如何使用函数`plotFunc`绘制基本图形。本章还展示了使用模块提供的函数的不同方式，既可以通过`:module`命令将模块加载到 GHCi 中，也可以通过`import`关键字将模块导入源代码文件。

下一章将介绍*类型类*，这是一个利用类型间共性的机制。

### 练习

**练习 7.1.** 绘制从*x* = –10 到*x* = 10 的 sin(*x*)图形。

**练习 7.2.** 绘制从*t* = 0 到*t* = 6 秒的`yRock30`函数图形。

**练习 7.3.** 绘制从*t* = 0 到*t* = 4 秒的`yRock 20`函数图形。使用`plotFunc`作为参数时，你需要将`yRock 20`括在圆括号中。
