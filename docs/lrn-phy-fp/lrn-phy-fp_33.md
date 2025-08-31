## 附录

安装 HASKELL

![Image](img/common.jpg)

本附录解释了如何安装格拉斯哥哈斯克尔编译器以及其他人编写的库。

### 安装 GHC

格拉斯哥哈斯克尔编译器（GHC）是本书中使用的哈斯克尔编译器。它是免费的开源软件，任何人都可以下载和安装。

安装过程取决于你使用的操作系统。对于 GNU/Linux 和 macOS 用户，我推荐访问[*https://www.haskell.org*](https://www.haskell.org)，然后选择**下载**。根据你的操作系统，按照相应的说明进行操作。你将知道安装成功，当你可以启动 GHCi 交互式编译器，通常通过在命令提示符下输入 `ghci` 来实现。此时，你已经准备好开始学习第一章。除了 GHC 本身，你使用的安装方法还将安装 Cabal 或 Stack。Cabal 和 Stack 是最常用的两种安装额外库包的工具。我将在本附录后面描述它们的使用。

对于微软 Windows 用户，我推荐按照[`www.fpcomplete.com/haskell/get-started/windows`](https://www.fpcomplete.com/haskell/get-started/windows)上的说明进行操作。FPComplete 是一家为工业哈斯克尔用户提供服务的公司。他们提供的安装程序将同时安装格拉斯哥哈斯克尔编译器和 Stack 库包管理器。当你能够启动 GHCi 交互式编译器，并在 PowerShell 提示符下输入 `stack ghci` 后看到 GHCi 提示符时，说明你已经成功安装。在此时，你已经准备好开始学习第一章。

### 安装文本编辑器

要编写源代码文件，你需要一个文本编辑器。你可以使用像 macOS 上的 Notes 或 Linux 上的 gedit 这样的基本文本编辑器，或者选择多个更复杂的文本编辑器。这些更复杂的编辑器通常具有对程序员有帮助的功能，例如文本高亮显示，且通常可以配置为根据你编写的编程语言进行敏感处理。

你可以在 Haskell 维基页面[*https://wiki.haskell.org/Haskell*](https://wiki.haskell.org/Haskell)找到关于如何使你的 Haskell 环境与编辑器顺利配合的建议。适合 Haskell 的编辑器有 Emacs、Vim、Visual Studio Code 和 Atom。像 Notes 这样的简单文本编辑器通常会随操作系统一起提供。Emacs 可以在[`www.gnu.org/software/emacs`](https://www.gnu.org/software/emacs)下载，Vim 可以在[`www.vim.org`](https://www.vim.org)下载，Atom 可以在[`atom.io`](https://atom.io)下载，Visual Studio Code 可以在[`code.visualstudio.com`](https://code.visualstudio.com)下载。按照你操作系统的说明进行操作。（想要在 macOS 上运行 Emacs 的用户应从[`emacsforosx.com`](https://emacsforosx.com)下载。这一链接提供了为 macOS 环境定制的标准 Emacs。由于它是标准的 Emacs，因此可以根据你在网上找到的建议，可靠地对其进行定制。定制的第一个步骤是[`www.emacswiki.org`](https://www.emacswiki.org)。）

### 安装 Gnuplot

从第七章开始，我们使用`gnuplot`来制作图表。`Gnuplot`是一个独立的图形程序，与 Haskell 无关，官网为[*http://gnuplut.info*](http://gnuplut.info)。安装`gnuplot`使其可以与 Haskell 一起使用是一个两步过程。首先，你需要安装`gnuplot`程序，使其能独立于 Haskell 运行。其次，你需要安装 Haskell 的`gnuplot`包，使得 Haskell 代码能够访问`gnuplot`的功能。本节内容涉及安装`gnuplot`程序，接下来的章节将解释如何安装 Haskell 的`gnuplot`包。

安装`gnuplot`程序的过程取决于你的操作系统。对于 GNU/Linux，你通常可以使用包管理器。例如，在 Ubuntu Linux 上，使用以下命令：

```
$ sudo apt install gnuplot
```

将会安装`gnuplot`程序。

在 macOS 上，我推荐使用 Homebrew 包管理器，网址为[`brew.sh`](https://brew.sh)。按照安装 Homebrew 的说明操作后，你可以执行以下命令来安装`gnuplot`程序：

```
$ brew install gnuplot
```

在 Microsoft Windows 上，按照[*http://www.gnuplot.info*](http://www.gnuplot.info)的说明下载 Windows 版`gnuplot`安装程序。运行安装程序，它会询问一系列问题，比如安装位置和其他安装细节。记下`gnuplot`安装的目录（可能是*C:*\*Program Files*\*gnuplot*\*bin*）。除了一个问题外，你可以接受安装程序的所有默认设置：当安装程序询问是否“将应用程序目录添加到你的 PATH 环境变量”时，勾选该选项。安装程序完成工作后，还有一件事需要做。使用文件浏览器，导航到`gnuplot`安装的目录，找到名为*wgnuplot*_*pipes*的文件。将此文件复制为同一目录下名为*pgnuplot*的新文件。如果文件名是*wgnuplot*_*pipes.exe*，将其复制为同一目录下名为*pgnuplot.exe*的新文件。这将允许 Haskell 使用`gnuplot`。

在这一点上，不管你的操作系统是什么，你应该能够独立于 Haskell 运行`gnuplot`程序。在命令行中，你需要输入以下内容：

```
$ gnuplot
```

启动`gnuplot`后，你应该能够在`gnuplot`提示符下发出命令，比如

```
gnuplot> plot cos(x)
```

应该会弹出一个包含图形的窗口。一旦你成功安装了`gnuplot`程序，你就可以准备安装 Haskell 的`gnuplot`包，它允许你从 Haskell 控制`gnuplot`。

### 安装 Haskell 库包

还有一些其他人编写的函数，我们希望使用，但这些函数没有包含在 Prelude（默认可用的标准函数集合）中。这些函数存在于可以在源代码文件中导入或直接加载到 GHCi 中的库模块中。GHC 附带了一组标准库模块，还有一些你可以通过 Cabal 或 Stack 安装的模块。标准库之外的库模块被组织成*包*，每个包包含一个或多个模块。

假设我们希望访问 Haskell `gnuplot`包提供的`Graphics.Gnuplot` `.Simple`模块中的`plotFunc`函数。我们必须安装`gnuplot`包。

安装 Haskell 库包的两个主要工具是 Cabal 和 Stack。你只需要使用其中一个。至少按照 GHC 安装说明，你会有其中一个工具可用。

Cabal（构建应用程序和库的通用架构）最早出现。在它编写时（大约 2005 年），为了最小化所需的下载量，Cabal 被设计为安装一组全局包，所有应用程序都应该针对这组通用包进行构建。同样，为了提高效率，Cabal 只允许安装每个包的一个版本。

这导致了一个问题：许多库迅速发展，添加了新特性并改变了它们的接口。一个常见的问题是，应用程序可能会依赖那些又依赖于不同版本的共同祖先的库。这有时需要卸载并重新安装所有包，偶尔还需要重新加载所有包的不同版本来构建一个新的应用程序。这个问题被称为“依赖地狱”或“Cabal 地狱”，这个名字足以让你了解它有多痛苦。

解决方法是允许安装多个版本的包，Cabal 现在允许这样做。

Stack 系统提供了与 Cabal 类似的许多功能，事实上，它可以与 Cabal 平稳共存，但它的目标略有不同。Stack 的目标是满足商业用户的需求，这些用户需要确保他们的应用程序即使在 Haskell 库基础设施不断发展的情况下也能正常构建。Stack 将这一目标称为“可重复构建”。为了实现可重复构建，Stack 的默认操作模式是让你指定一个编译器版本和一组已知与该编译器正常工作的精选包。精选包集包含超过 2000 个包，因此你很可能会在其中找到大部分需要的内容（如果没有，也不难指定你希望下载和构建的其他包）。这种看似复杂的方式的好处是，你的 Haskell 程序不仅每次都以相同的方式构建，而且以相同的方式运行。

Stack 和 Cabal 通常能够避免不一致的依赖项破坏大型复杂项目构建的问题。然而，这也有代价。它们可能会下载比你预期更多的包。特别是 Stack，可能会下载多个编译器，以确保包和编译器已知能产生一致的结果。这看起来似乎不必要，但这是 GHC 编译器工作方式所要求的。出于一些重要但繁琐的技术原因，GHC 编译器没有标准化的“应用二进制接口”（ABI）。这意味着你不能将用一个版本的 GHC 编译的库与用另一个版本编译的应用程序一起使用。这不是一个 bug——事实证明，为了得到一个纯粹的函数式语言、惰性求值和良好的性能，你需要放弃某些东西。而其中之一就是稳定的 ABI。

#### 使用 Cabal

要将模块加载到 GHCi 中，工作目录必须能够访问该模块。对于 GHC 安装本身提供的标准模块以外的模块，必须安装包含该模块的包。有两种使用 Cabal 安装包的方式：全局安装，这样包可以从任何目录访问；本地安装，这样它只能从当前工作目录访问。

##### 使用 Cabal 全局安装一个包

要全局安装 `gnuplot` 包，请执行以下命令：

```
$ cabal install --lib gnuplot
```

在我的计算机上，此命令会创建或更改文件*/home/walck/ .ghc/x86_64-linux-8.10.5/environments/default*，该文件包含全局安装的 Haskell 包列表。在您全局安装了一个或多个包后，类似我们刚刚发出的 Cabal 命令，可能会无法安装新包，因为 Cabal 找不到与已安装全局包兼容的请求包版本。解决此问题的一种方法是重命名包含全局包列表的文件，然后尝试同时安装所有需要的包。例如，要同时安装`gnuplot`、`gloss`和`cyclotomic`包，您可以发出以下命令：

```
$ cabal install --lib gnuplot gloss cyclotomic
```

因为我们重命名了全局包列表，Cabal 将找不到全局包列表，因此会创建一个新的包列表。

##### 使用 Cabal 本地安装包

要在本地（当前工作目录）安装`gnuplot`包，请发出以下命令：

```
$ cabal install --lib gnuplot --package-env .
```

命令末尾的点表示当前工作目录。此命令会在当前工作目录中创建或更改一个名为*.ghc.environment .x86_64-linux-8.10.5*的文件。该文件包含本地安装的包列表（位于当前工作目录）。当您在某个目录中本地安装了一个或多个包后，类似我们刚刚发出的 Cabal 命令，可能会无法安装新包，因为 Cabal 找不到与已安装本地包兼容的请求包版本。解决此问题的一种方法是重命名包含本地包列表的文件，然后尝试同时安装所有需要的包。例如，要同时安装`gnuplot`、`gloss`和`cyclotomic`包，您可以发出以下命令：

```
$ cabal install --lib gnuplot gloss cyclotomic --package-env .
```

因为我们重命名了本地包列表，Cabal 将找不到本地包列表，因此会创建一个新的包列表。

#### 使用 Stack

要使用 Stack 安装`gnuplot`包，请发出以下命令：

```
$ stack install gnuplot
```

在命令提示符下。Stack 比 Cabal 跟踪更多的幕后事项，全局安装通过 Stack 通常就是您所需要的。

安装完`gnuplot`包后，您可以将`Graphics.Gnuplot` `.Simple`模块加载到 GHCi 中。如果您使用的是 Stack，应该通过`stack ghci`启动 GHCi，而不是`ghci`。这样，Stack 就能找到您已安装的包的模块。

```
Prelude> :m Graphics.Gnuplot.Simple
Prelude Graphics.Gnuplot.Simple> :t plotFunc
plotFunc
  :: (Graphics.Gnuplot.Value.Atom.C a,
      Graphics.Gnuplot.Value.Tuple.C a) =>
     [Attribute] -> [a] -> (a -> a) -> IO ()
```

在这里，我们请求`plotFunc`函数的类型，仅仅是为了展示它在我们加载了定义它的模块之后已经可以使用。

要在源代码文件中使用`plotFunc`函数，请包含以下行：

```
import Graphics.Gnuplot.Simple
```

在您的源代码文件顶部。

### 安装 Gloss

从第十三章开始，我们使用`gloss`来制作动画。与`gnuplot`不同，`gloss`不是一个独立的程序；它只是一个 Haskell 包。然而，`gloss`使用 freeglut 图形库来完成工作，freeglut 的功能由非 Haskell 库提供，这些库必须与`gloss`包本身分开安装。因此，像安装`gnuplot`一样，安装`gloss`是一个两步过程。首先，你需要安装非 Haskell 的 freeglut 库。其次，你需要安装 Haskell 的`gloss`包。

安装 freeglut 库的过程取决于你的操作系统。对于 GNU/Linux 系统，可以使用类似下面的命令：

```
$ sudo apt install freeglut3
```

应该可以解决问题。在 macOS 上，你可以使用类似的命令，借助`brew`包管理器。

```
$ brew install freeglut3
```

是你所需要的。你需要安装`brew`包管理器才能使用此命令。在 macOS 上，你可能还需要安装`xquartz`包来使用 freeglut，你可以通过下面的命令来安装：

```
$ brew install xquartz
```

对于 Microsoft Windows 系统，请在网上搜索“freeglut windows”并按照找到的说明操作。

安装了 freeglut 库后，你可以通过类似下面的命令来安装`gloss`包：

```
$ cabal install --lib gloss
```

或者

```
$ stack install gloss
```

这取决于你是使用 Cabal 还是 Stack。

### 安装 Diagrams

从第二十二章开始，我们使用`diagrams`包来可视化向量场。实际上，`diagrams`包只是对三个包的封装，分别是`diagrams-core`、`diagrams-lib`和`diagrams-contrib`。封装的目的是简化安装过程，因为你只需要发出一个命令而不是三个。我们将使用这三个包中的两个，再加上另一个。我们将使用`diagrams-core`、`diagrams-lib`和`diagrams-cairo`。

与`gloss`类似，`diagrams-cairo`包使用一些图形库来完成工作，必须将这些非 Haskell 库与`diagrams-cairo`包本身分开安装。因此，像安装`gnuplot`和`gloss`一样，安装`diagrams`也是一个两步过程。首先，你需要安装非 Haskell 的图形库。其次，你需要安装 Haskell 的`diagrams`包。

所需的图形库是`cairo`和`pango`。安装这些库的过程取决于你的操作系统。对于 GNU/Linux 系统，可以使用类似下面的命令：

```
$ sudo apt install libcairo2-dev libpango1.0-dev
```

应该可以解决问题。在 macOS 上，你可以使用类似的命令，借助`brew`包管理器。

在安装了`cairo`和`pango`库后，你可以通过类似下面的命令来安装`diagrams`包：

```
$ cabal install --lib diagrams-core diagrams-lib diagrams-cairo
```

或者

```
$ stack install diagrams-core diagrams-lib diagrams-cairo
```

这取决于你是使用 Cabal 还是 Stack。

### 设置你的编码环境

随着本书的进展，我们的代码变得越来越复杂，因为我们开始使用其他人编写的模块以及我们自己编写的模块。我们希望将一些代码加载到 GHCi 中，同时我们还希望编写独立的程序。因此，我们需要一种方法来保持代码的组织性，以便能够访问我们所需的模块，从而使我们能够做我们想做的事情。保持组织性的主要方法有两种：

(1) 将所有源代码文件保存在一个目录中。这包括用于加载到 GHCi 中的文件以及独立的程序。安装软件包，以便该目录能够访问它们。确保该目录能够访问书中的模块。

(2) 为你正在进行的每个项目创建一个新的目录。确保该目录能够访问项目所需的模块和软件包。每个目录可能会有一个 *.cabal* 文件，如果你使用 `stack`，还可能有一个 *stack.yaml* 文件。这些文件描述了你项目的需求。

我建议采用方法 (1)，至少在你没有看到为新项目创建新目录的任何优势之前。就本书的目的而言，你需要做的练习并不大，每个练习并不需要自己的目录。

#### 我们对编码环境的需求

在给出关于如何组织你的编码环境的两条具体建议之前，让我们先明确我们想要实现的目标。以下是我们希望编码环境具备的四个期望特性：

(a) 我们希望能够通过 GHCi 的 `:l` 命令将我们编写的源代码文件加载到 GHCi 中。这样的源代码文件可能有模块名，也可能没有模块名。这样的源代码文件也可能会使用 Haskell 的 `import` 关键字导入模块，也可能不会导入模块。

(b) 我们希望能够通过 GHCi 的 `:m` 命令将他人编写的模块，如 `Graphics.Gnuplot.Simple`，加载到 GHCi 中。

(c) 我们希望能够从我们编写的源代码文件生成可执行程序。这样的源代码文件可能会使用 Haskell 的 `import` 关键字导入模块，也可能不会导入模块。

(d) 我们希望能够通过将模块加载到 GHCi 中以及编写源代码 `import` 这些模块来使用本书中定义的模块。

如需将源代码文件加载到 GHCi 中，如 (a) 所述，我们需要在源代码文件所在的目录中启动 GHCi。如果我们的源代码文件导入了模块，它需要能够访问这些模块。如果源代码文件导入的模块由某个包提供，则当前工作目录必须能够访问该包。这可以是本地访问，也可以是全局访问，如本附录前文所定义。如果该模块是在源代码文件中定义的，例如本书中编写的模块之一，那么该文件必须位于工作目录中，或者位于 GHC 知道要查找的位置。

如需将他人编写的模块加载到 GHCi 中，如 (b) 所述，工作目录需要能够访问提供我们希望加载的模块的包。这可以是本地访问，也可以是全局访问，如前文所述。

生产一个独立的程序，如(c)所需的内容，是第十二章的主题。在那里，我们讨论了三种生成独立程序的方法：一种使用 GHC，一种使用 Cabal，另一种使用 Stack。如该章所述，使用 Cabal 或 Stack 是一种方法(2)，因为我们每个目录中只能拥有一个*.cabal*文件。然而，该*.cabal*文件允许指定多个独立程序，因此可以使用 Cabal 或 Stack 与方法(1)结合使用。

为了实现(d)，最简单的方法是将所有定义模块的*.hs*文件（例如*Mechanics3D.hs*，它定义了`Mechanics3D`模块）放入你的工作目录。由于你编写的源代码文件也在此目录中，GHC 在你加载该文件到 GHCi 时，或在你使用 GHC 编译它时，会在工作目录中查找你的源代码文件所导入的模块。

以下两个部分将提供关于将本书中定义的模块放置在哪些位置的具体建议，你可以在[`lpfp.io`](https://lpfp.io)下载相关文件。这两个建议是替代方案，你只需遵循其中一个即可。

#### 所有代码放在一个目录中

如前所述，保持组织最简单的方法是将所有内容放在一个目录中。这包括：

+   你打算加载到 GHCi 中的源代码文件

+   你打算编译成可执行程序的源代码文件

+   本书中定义的模块的源代码文件，例如*Mechanics3D.hs*

这个目录将是你所有 Haskell 工作的工作目录。如果你继续编程 Haskell，你会逐渐超越这种方法。你将希望处理不同目的和需求的不同项目，而不希望将所有代码放在一个目录中。当你到达这个阶段时，有很多前进的方式。Cabal 和 Stack 工具提供了许多组织工作的方式。

目前，我们需要确保我们的工作目录可以访问本书项目所需的所有包。以下命令需要在命令提示符下以一行输入，它将本地安装我们本书所需的所有包。

```
$ cabal install --lib gnuplot gloss not-gloss spatial-math diagrams-lib
  diagrams-cairo --package-env .
```

这种方法的一个缺点是我们可以通过 GHCi 的`:l`命令加载书中的模块，但无法通过 GHCi 的`:m`命令加载，这意味着我们一次只能加载一个书中的模块。如果我们希望在 GHCi 中访问不同模块中定义的函数，这可能会很不方便。一种解决方法是创建一个新的源代码文件，将我们需要的所有模块导入其中，然后使用`:l`命令将该源代码文件加载到 GHCi 中。

另一种解决此缺点的方法是使用 Stack 工具来管理本书中的模块，如下一节所述。

#### 使用 Stack 的一种方式

Cabal 和 Stack 工具提供了许多（可能太多）方法来组织你的 Haskell 工作。在这里，我们将详细探讨一种方法。在这种方法中，我们仍然有一个目录来存放所有的 Haskell 工作，但这个目录有两个子目录：一个用于书籍模块，另一个用于独立程序。因此，源代码文件可以存在三个地方。它们可以存放在主工作目录中，也可以存放在模块子目录中，或者可以存放在独立程序子目录中。你打算加载到 GHCi 中的源代码文件可能会存放在主工作目录中。

Stack 需要两个配置文件来管理事务。一个名为 *LPFP.cabal*，另一个名为 *stack.yaml*。这两个文件将位于主工作目录中。文件 *LPFP.cabal* 描述了我们希望访问的模块，以及我们希望 Stack 为我们构建的可执行程序。列表 A-1 给出了这个文件。

```
cabal-version:  1.12

name:           LPFP
version:        1.0
description:    Code for the book Learn Physics with Functional Programming
homepage:       http://lpfp.io
author:         Scott N. Walck
maintainer:     walck@lvc.edu
copyright:      2022 Scott N. Walck
license:        BSD3
license-file:   LICENSE
build-type:     Simple

library
  exposed-modules:
      Charge, CoordinateSystems, Current, ElectricField, Electricity, Geometry
    , Integrals, Lorentz, MagneticField, Maxwell, Mechanics1D, Mechanics3D
    , MOExamples, MultipleObjects, Newton2, SimpleVec
  hs-source-dirs: src
  build-depends:
      base >=4.7 && <5, gnuplot, spatial-math, gloss, not-gloss, diagrams-lib
    , diagrams-cairo, containers
  default-language: Haskell2010

executable LPFP-VisTwoSprings
  main-is: VisTwoSprings.hs
  hs-source-dirs: app
  build-depends: LPFP, base >=4.7 && <5, not-gloss
  default-language: Haskell2010

executable LPFP-GlossWave
  main-is: GlossWave.hs
  hs-source-dirs: app
  build-depends: LPFP, base >=4.7 && <5, gloss
  default-language: Haskell2010
```

*列表 A-1：描述我们希望访问的模块和我们希望生成的可执行程序的文件* `LPFP.cabal`

在一些介绍性内容之后，出现了一个库段和两个可执行段。库段列出了我们希望访问的本书中的所有模块。它说明了这些模块的源代码位于 *src* 子目录中，并且这些模块依赖于几个包，例如 `gnuplot` 和 `gloss`。`base` 模块包含了大多数简单数据类型所需的基础库。版本规范表示“版本 4.7 或更新，但主版本必须小于 5。” “default-language” 规范告诉我们我们使用的是 2010 版本的 Haskell 语言规范，这是当前版本。之前的版本是 Haskell98，这让你对语言的主要版本修订间隔有个概念。

每个我们希望 Stack 为我们构建的独立程序都有一个可执行段。这里列出了两个，但你可以根据需要列出任意数量的程序。第一个可执行段描述了位于名为 *app* 的子目录中的源代码文件 *VisTwoSprings.hs* 的独立程序。该可执行程序将被命名为 *LPFP-VisTwoSprings*，并可以在任何目录下全局运行。此独立程序所需的包也在此列出。

在撰写本文时，`diagrams` 包不包括在 Stack 默认使用的精选包列表中，因此我们必须在名为 *stack.yaml* 的文件中列出一些额外的包。列表 A-2 显示了这个文件。

```
resolver: lts-18.21

packages:
- .

extra-deps:
- diagrams-cairo-1.4.1.1
- diagrams-lib-1.4.4
- active-0.2.0.15
- cairo-0.13.8.1
- diagrams-core-1.5.0
- dual-tree-0.2.3.0
- monoid-extras-0.6.1
- pango-0.13.8.1
- statestack-0.3
- glib-0.13.8.1
- gtk2hs-buildtools-0.13.8.2
```

*列表 A-2：描述本书中模块所需额外依赖的文件 stack.yaml*

对于每个编译器版本，Stack 支持一组已知能与该编译器一起构建并且通常互相兼容的精选软件包。通过版本号指定编译器和软件包集。在示例 A-2 中，`resolver`字段中的`lts-18.21`表示“GHC 8.10.7 及其兼容的软件包”。这个特定的编译器/软件包集合有长期支持（`lts-`前缀）。这意味着你可以依赖它会持续一段时间，通常是几年。

如果你需要在前沿技术中生活以获取所需的功能，可以使用快照集合，若需要最新的功能，则可以使用夜间构建版本。

下一个字段`packages`指的是*你*自己编写的包，通常是对你自己项目有用的库。在示例 A-2 中，包只是当前目录中的文件，或者用 Unix 术语表示为“`.`”。

`extra-deps`是你的应用程序所依赖的额外包，这些包不属于`resolver`字段指定的精选包集合。（事实上，`package`和`extra-dep`之间没有太大区别，除了我们可以为自己的包编写测试和基准目标——这是大型应用程序中非常重要的部分——而这些对于`extra-deps`来说是不可用的。）

如果你有关于*stack.yaml*文件的问题，首先可以访问[`docs.haskellstack.org/en/stable/README`](https://docs.haskellstack.org/en/stable/README)。

你可以看到我们感兴趣的包`diagrams-core`、`diagrams-lib`和`diagrams-cairo`。其余的包是`diagrams`所依赖的包。具体版本的这些包也列出了。在本书出版时，这些包的更新版本可能已经发布。

要构建可执行程序，在主工作目录中输入以下命令（该目录包含*stack.yaml*和*LPFP.cabal*文件）：

```
$ stack install
```

要启动一个 GHCi 会话，其中所有的书籍模块都会自动加载，你可以输入以下命令：

```
$ stack ghci
```

使用这种方法，我们可以将任何或所有的书籍模块加载到 GHCi 中。要移除一个模块，你可以使用 GHCi 的`:m`命令，模块名前加上减号。要移除`Newton2`模块，输入以下命令：

```
ghci>  :m -Newton2
```

同样，要添加一个额外的模块，使用加号前缀。要添加`Graphics.Gnuplot.Simple`模块，输入以下命令：

```
ghci>  :m +Graphics.Gnuplot.Simple
```

输入`stack ghci`命令还可以为你提供将可执行程序之一加载到 GHCi 中的选项，如果你愿意的话。

### 总结

本附录介绍了如何安装 Haskell 编译器和文本编辑器，并讲解了使用 Cabal 和 Stack 安装额外库包的方法。它还展示了组织库和源代码文件以便在 Haskell 中构建项目的不同方式。
