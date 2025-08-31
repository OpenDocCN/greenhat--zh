# 第十二章：创建独立程序

![图片](img/common.jpg)

到目前为止，我们已经使用 GHCi 进行所有计算并显示结果。我们已经编写了相当复杂的源代码文件，但我们一直将它们加载到 GHCi 中以使用其函数。然而，Haskell 是一种功能齐全、适合生产环境的计算机语言，完全能够编译无需任何 GHCi 参与的独立程序。第十三章及后续章节中的动画演示最好使用独立程序来执行，而不是 GHCi。

本章解释了三种不同的构建独立（可执行）程序的方法。最基本的方法是使用`ghc`来生成可执行程序。使用这种方法时，你需要自己负责安装程序所需的任何库包。第二种方法是使用`cabal`，它会自动安装程序所需的库包，但这些包必须在配置文件的适当位置列出。第三种方法是使用`stack`，它会自动执行更多操作，比如安装与你请求的包版本兼容的 GHC 编译器版本。要创建一个独立程序，你只需使用这三种方法中的一种。如果你是 Haskell 新手，你可能会发现`stack`方法是最容易使用的。

对于这三种方法中的每一种，我们将逐步讲解如何生成一个可执行程序：（a）用于一个非常简单的程序，（b）用于一个同时使用我们编写的模块和其他人编写的模块的程序。

### 使用 GHC 构建独立程序

在本节中，我们直接使用 GHC 来创建独立程序。首先，我们为一个非常简单的程序“你好，世界！”做演示，然后再为一个更复杂的程序做演示，该程序导入了模块。

#### 你好，世界！

在学习一门新语言时，人们通常编写的最简单的独立程序被称为“你好，世界！”。这个程序的功能就是打印“你好，世界！”并退出。对于许多计算机语言来说，学习如何编写“你好，世界！”程序是语言学习过程中的早期步骤。然而，在 Haskell 中，提早学习“你好，世界！”没有太大意义，因为“你好，世界！”程序的重点是产生效果，即在屏幕上打印某些内容，而 Haskell 编程的核心，甚至是函数式编程的核心，是关于纯函数的，纯函数没有副作用。

Haskell 中的“你好，世界！”程序由两行代码组成：

```
main :: IO ()
main = putStrLn "Hello, world!"
```

每个独立程序都需要一个名为`main`的函数，它通常具有类型`IO ()`。我们在第七章中首次介绍了`IO ()`，它是一个不返回有意义值但会产生副作用的非纯函数的类型。一般来说，`IO a`类型表示一个类型为`a`的值以及一个副作用。

`main`函数需要产生某些效果，否则我们无法确认程序是否真正运行。`main`这个有副作用的函数的目的是向编译器描述我们希望计算机*做什么*，而`IO ()`类型正好适合这个目的，因为它表示一个没有实际值的副作用。

函数`putStrLn`是一个 Prelude 函数，它接受一个字符串作为输入，在屏幕上打印该字符串，并换行，以便任何后续打印将在新的一行显示。还有一个名为`putStr`的函数，类型与`putStrLn`相同，它打印一个字符串，但不会换行，因此后续的打印会直接跟在打印的字符串后面。名称中的`Ln`提醒我们，该函数在打印后会换行。`putStrLn`的类型显示它接受一个字符串作为输入，并产生一个效果。

```
Prelude> :t putStrLn
putStrLn :: String -> IO ()
```

假设我们将这两行代码放在一个名为*hello.hs*的源代码文件中。如果你的操作系统提供了命令行，命令如下：

```
$ ghc hello.hs
```

将编译源代码文件*hello.hs*，生成一个名为*hello*的可执行文件，你可以运行它。在 Linux 系统上，你可以通过命令行使用命令运行程序*hello*：

```
$ ./hello
```

程序名称前面的点斜杠告诉操作系统执行当前工作目录中的名为*hello*的程序。如果省略点斜杠，操作系统会在标准搜索路径中查找名为*hello*的程序，如果当前工作目录不在搜索路径中，可能找不到该程序。

#### 一个导入模块的程序

现在我们来编译一个独立的程序，使用我们在第十章中编写的`SimpleVec`模块的函数，以及`gnuplot`包中的`Graphics` `.Gnuplot.Simple`模块的函数。包含`SimpleVec`模块源代码的文件*SimpleVec.hs*可以在[`lpfp.io`](https://lpfp.io)找到。列表 12-1 展示了我们想要编译的独立程序。

```
{-# OPTIONS -Wall #-}

import SimpleVec ( iHat, kHat, xComp, zComp, projectilePos, (^+^), (*^) )
import Graphics.Gnuplot.Simple ( Attribute(..), plotPath )

main :: IO ()
main = let posInitial = 10 *^ kHat
           velInitial = 20 *^ cos (pi/6) *^ iHat ^+^ 20 *^ sin (pi/6) *^ kHat
           posFunc = projectilePos posInitial velInitial
           pairs = [(xComp r, zComp r) | t <- [0, 0.01 ..], let r = posFunc t]
           plottingPairs = takeWhile (\(_,z) -> z >= 0) pairs
       in plotPath [Title "Projectile Motion"
                   ,XLabel "Horizontal position (m)"
                   ,YLabel "Height of projectile (m)"
                   ,PNG "projectile.png"
                   ,Key Nothing
                   ] plottingPairs
```

*列表 12-1：独立程序 MakeTrajectoryGraph.hs，使用了 SimpleVec 模块和 Graphics.Gnuplot.Simple 模块的函数*

清单 12-1 中的程序生成了一个图表，展示了从离地面 10 米高的建筑物顶部抛出的一个球的轨迹，初速为 20 米/秒，抛掷角度为水平面上方 3^(0∘)。程序生成了一个名为*projectile.png*的文件，包含了该图表。为了完成这项工作，程序从第十章的`SimpleVec`模块导入了如`projectilePos`、`xComp`、`zComp`、`iHat`和`kHat`等函数。程序还使用了来自`Graphics.Gnuplot.Simple`模块的`plotPath`函数。由于使用了数据构造器`Title`、`XLabel`等来自`Attribute`数据类型，因此我们通过在类型名`Attribute`后附加两个冒号和括号，导入了`Attribute`数据类型及其构造器。

我们假设清单 12-1 中的代码包含在名为*MakeTrajectoryGraph.hs*的源代码文件中。要使用`ghc`编译程序，必须满足两个条件：

+   包含`SimpleVec`模块的文件*SimpleVec.hs*必须与包含主程序的文件*MakeTrajectoryGraph.hs*位于同一目录。我们将这个目录称为*工作目录*。

+   工作目录必须能够访问`Graphics.Gnuplot.Simple`模块。这要求`gnuplot`包已被安装，（a）全局安装，以便从任何目录访问，或（b）局部安装，以便从工作目录访问。要全局安装`gnuplot`包，请发出以下命令：

    ```
    $ cabal install --lib gnuplot
    ```

    在我的计算机上，此命令会创建或更改文件*/home/walck/.ghc/x86_64-linux-8.10.5/environments/default*，其中包含了全局安装的 Haskell 包列表。要在本地（即工作目录中）安装`gnuplot`包，请发出以下命令：

    ```
    $ cabal install --lib gnuplot --package-env .
    ```

    此命令会在当前工作目录中创建或更改一个名为*.ghc .environment.x86_64-linux-8.10.5*的文件。该文件包含了在本地安装的包列表（即当前工作目录中的包）。有关安装 Haskell 包的更多信息，请参见附录。

一旦满足这两个条件，我们通过发出以下命令，将源代码文件*MakeTrajectoryGraph.hs*编译成可执行程序：

```
$ ghc MakeTrajectoryGraph.hs
```

此命令必须从包含文件*MakeTrajectoryGraph.hs*、文件*SimpleVec.hs*并且能够访问`Graphics.Gnuplot.Simple`模块的同一工作目录中发出。

如果编译器找不到`Graphics.Gnuplot.Simple`模块，你将看到如下错误：

```
MakeTrajectoryGraph.hs:4:1: error:
    Could not load module 'Graphics.Gnuplot.Simple'
```

在这种情况下，必须安装`gnuplot`包，无论是全局安装还是局部安装，这样它才可以从工作目录中访问。

如果一切顺利，编译器将在当前工作目录下生成一个名为*Make TrajectoryGraph*的可执行文件。该可执行文件不会安装到任何全局位置，因此要运行程序，你需要提供可执行文件的完整路径名，或者通过在可执行文件名前加上`./`来从所在目录运行，如下所示：

```
$ ./MakeTrajectoryGraph
```

使用`ghc`来生成可执行程序的好处是你不需要担心配置文件。缺点是程序所需的任何模块，不论是你自己写的还是其他人写的，都必须能够从程序所在的目录中访问。随着你的程序依赖的库包数量增加，这种安装负担也在增加，特别是因为程序可以接受的包版本可能与其他程序或你想使用的其他库包的可接受版本发生冲突。我们接下来要介绍的`cabal`和`stack`工具是为了解决这种复杂性设计的，这样你就不必亲自处理这些问题。

### 使用 Cabal 制作独立程序

在上一节中，我们使用了`cabal`安装了一个包。但是`cabal`工具在你的 Haskell 生态系统中可以发挥更大的作用，它可以管理你的独立程序所需的模块和包，并使用兼容的版本，即使它们与其他项目使用的包发生冲突。要获取有关`cabal`工具可以做什么的基本信息，请输入以下命令

```
$ cabal help
```

在命令提示符下输入。

使用`cabal`来管理项目依赖关系的第一步是创建一个新的子目录，里面包含你的独立程序的源代码以及`cabal`执行其工作的所需文件。我们使用以下命令在当前目录下创建一个名为*Trajectory*的新目录。为这个目录使用一个唯一的名称，因为该名称将成为可执行程序的默认名称，也是项目的一般名称。

```
$ mkdir Trajectory
```

我们进入这个新目录，并通过输入命令使其成为工作目录

```
$ cd Trajectory
```

其中`cd`代表“更改目录”。在这个新目录内，我们输入以下命令：

```
$ cabal init
```

这将创建一个名为*Trajectory.cabal*的文件和一个名为*app*的子目录，其中包含一个名为*Main.hs*的文件。旧版本的`cabal`会将*Main.hs*创建在当前目录，而不是*app*子目录中。

假设你可能希望将代码与其他人共享，`cabal`要求你有一个名为*LICENSE*的文件，内容包括其他人可以使用你代码的条款。`cabal`工具可能要求你在编译代码之前有这样的文件，因此请准备好提供一个。`cabal`程序并不关心*LICENSE*文件的内容，只关心它是否存在。

文件*Main.hs*是一个默认的源代码文件，包含一个非常简单的程序。要编译它，请输入

```
$ cabal install
```

在命令提示符下，当当前工作目录为*Trajectory*时，如果一切顺利，`cabal`将编译*Main.hs*中的代码，生成名为*Trajectory*的可执行文件，并使该可执行文件在全局可用，这意味着可以通过输入其名称*Trajectory*来运行，而不需要输入包含文件路径结构的完整路径名。

我们可以使用以下命令测试可执行文件：

```
$ Trajectory
```

然后我们应该看到一条简短的欢迎信息。

继续使用`cabal`为清单 12-1 中的代码生成独立程序，我们查看*Trajectory.cabal*文件。这是`cabal`的配置文件，告诉它如何将源代码编译成当前目录项目的可执行代码。之前展示的`cabal` `init`命令在创建*Trajectory.cabal*时选择了多个选项的默认值。我们现在关注的行大致如下所示：

```
executable Trajectory
    main-is:          Main.hs

    -- Modules included in this executable, other than Main.
    -- other-modules:

    -- LANGUAGE extensions used by modules in this package.
    -- other-extensions: build-depends:    base ^>=4.14.2.0
    hs-source-dirs:   app
```

第一行表示可执行程序的名称将是*Trajectory*。这个默认名称与项目目录的名称相匹配；但是，如果我们愿意，也可以将其更改为其他名称。第二行给出了包含`main`函数的源代码文件的名称。默认情况下，这个文件被称为*Main.hs*，并位于*Trajectory*目录的*app*子目录中。由双短横线开头的行是注释。以`build-depends:`开头的行列出了主程序所依赖的包。默认情况下，*.cabal*文件只包含对`base`包的依赖。`base`包使所有 Prelude 函数和类型可用。以`hs-source-dirs:`开头的行是包含源文件的子目录列表。

要编译清单 12-1 中的代码，该代码包含在*Make TrajectoryGraph.hs*文件中，我们需要做三件事：

1.  将文件*MakeTrajectoryGraph.hs*复制或移动到*Trajectory*目录的*app*子目录中。然后编辑*Trajectory.cabal*，将主源代码文件的名称从*Main.hs*更改为*MakeTrajectoryGraph.hs*。修改后的*Trajectory.cabal*中的行如下所示：

    ```
       main-is:         MakeTrajectoryGraph.hs
    ```

1.  将包含`SimpleVec`模块的文件*SimpleVec.hs*复制或移动到*Trajectory*目录的*app*子目录中。（该文件以及本书中的所有其他模块可以在[`lpfp.io`](https://lpfp.io)获取。）然后编辑*Trajectory.cabal*，取消注释（去掉双短横线）`other-modules:`行，并添加`SimpleVec`模块（不带*.hs*扩展名）。

    ```
       other-modules:    SimpleVec
    ```

1.  编辑*Trajectory.cabal*，在`build-depends:`行中包含`gnuplot`。这样，我们就可以在主程序中导入模块`Graphics.Gnuplot.Simple`。做完这三项更改后，修改后的*Trajectory.cabal*中的行如下所示：

    ```
    executable Trajectory
        main-is:          MakeTrajectoryGraph.hs

        -- Modules included in this executable, other than Main.
        other-modules:    SimpleVec

        -- LANGUAGE extensions used by modules in this package.
        -- other-extensions:
        build-depends:    base ^>=4.14.2.0, gnuplot
        hs-source-dirs:   app
    ```

虽然 `base` 包对允许的 `base` 版本有界限，但我们并未为 `gnuplot` 包指定版本限制。版本限制的目的是允许仍在开发中的代码以与以前版本不兼容的方式演变。库包的作者遵循约定，表示小版本号变化和修复是由小的版本号变化表示的，而主要变更则通过版本号的较大变化来表示。使用版本限制，就像刚刚展示的 `base`，是一种确保你获得期望功能的技术。

将 `gnuplot` 添加到构建依赖列表中，会导致 `cabal` 安装 `gnuplot` 包，但以使其对这个项目私有的方式，即在 *Trajectory* 目录中的项目。作为结果，`gnuplot` 包在 GHCi 中将不可用。要使 `gnuplot` 在 GHCi 中可用，请按照附录中的说明进行操作。

现在重新发布

```
$ cabal install
```

重新编译名为 *Trajectory* 的程序。我们可以用以下命令测试可执行文件：

```
$ Trajectory
```

可执行文件应生成一个名为 *projectile.png* 的文件。

`cabal` 安装的包，如 `gnuplot`，存放在 [*https://hackage.haskell.org*](https://hackage.haskell.org) 上。你可以访问该网站搜索、浏览并阅读关于任何 `cabal` 可以安装的包的文档。

### 使用 Stack 来创建独立程序

`stack` 工具可以管理你的独立程序所需的模块和包，使用能够协同工作的版本，即使它们与你可能拥有的其他项目使用的包发生冲突。要获取有关 `stack` 工具功能的基本信息，请执行以下命令

```
$ stack --help
```

在你的命令提示符下。

使用 `stack` 管理名为 *Trajectory* 的新项目的依赖项的第一步是执行以下命令

```
$ stack new Trajectory
```

这将创建一个名为 *Trajectory* 的子目录。我们通过执行以下命令进入这个新目录，并将其设置为当前目录：

```
$ cd Trajectory
```

在这个目录中，我们可以找到 `stack` 为我们创建的几个文件和子目录。最重要的文件是 *Trajectory.cabal*，它包含关于如何编译你的程序的重要信息。`stack` 工具是在 `cabal` 工具的基础上构建的，并使用其配置文件。*Trajectory.cabal* 中最重要的几行内容如下：

```
library
  exposed-modules:
     Lib
  other-modules:
     Paths_Trajectory
  hs-source-dirs:
     src
  build-depends:
     base >=4.7 && <5
  default-language: Haskell2010

executable Trajectory-exe
  main-is: Main.hs
  other-modules:
     Paths_Trajectory
  hs-source-dirs:
     app
  ghc-options: -threaded -rtsopts -with-rtsopts=-N
  build-depends:
     Trajectory
   , base >=4.7 && <5
  default-language: Haskell2010
```

在这里，我们看到两个部分：一个以`library`开头，另一个以`executable`开头。`library`部分负责我们编写的模块的名称、位置和依赖项，例如`SimpleVec`。我们希望`stack`管理的模块名称放在`exposed-modules:`下，如果有多个模块，它们之间用逗号隔开。新建的`stack`项目的默认程序只使用一个模块，名为`Lib`。在这里，我们不需要使用`other-modules:`这一部分，可以保持为空。我们模块所在的目录放在`hs-source-dirs:`下。默认情况下，*Trajectory*目录下的子目录*src*是模块的存放位置，我们不需要修改这个设置。我们只需将模块复制或移动到`stack`为我们创建的*src*目录中。我们没有编写，但模块依赖的包（例如`gnuplot`）列在`build-depends:`下。

`executable`部分的第一行表明可执行程序的名称将是*Trajectory-exe*。这个默认名称与我们为项目命名时一致，但如果需要，我们可以将其改为其他名称。`main-is:`后跟包含`main`函数的源代码文件的名称。默认值为*Main.hs*。在`executable`部分，和`library`部分一样，我们不需要使用`other-modules:`这一部分，可以保持为空。可执行文件（独立程序）源代码所在的目录放在`hs-source-dirs:`下。默认情况下，*Trajectory*目录下的子目录*app*是主程序源代码的存放位置，我们不需要更改这个设置。我们只需将代码复制或移动到`stack`为我们创建的*app*目录中。目前，*app*子目录包含*Main.hs*源代码文件。

文件*Main.hs*是一个默认的源代码文件，包含一个非常简单的程序。要编译它，输入以下命令：

```
$ stack install
```

在命令提示符下，当前工作目录是*Trajectory*（包含*.cabal*文件的目录）。如果一切顺利，`stack`将编译*Main.hs*中的代码，生成一个名为*Trajectory-exe*的可执行文件，并将该可执行文件设置为全局可用，因此即使从其他目录，也可以通过输入其名称*Trajectory-exe*来运行它。

我们可以通过以下方式测试可执行文件：

```
$ Trajectory-exe
```

然后，我们应该看到屏幕上出现一串简短的文本。

接下来，我们使用`stack`将清单 12-1 中的代码，存储在文件*MakeTrajectoryGraph.hs*中，生成一个独立程序，步骤如下：

1.  将文件 *MakeTrajectoryGraph.hs* 复制或移动到 *Trajectory* 目录下的 *app* 子目录中。然后编辑 *Trajectory.cabal*，将主源代码文件的名称从 *Main.hs* 改为 *MakeTrajectoryGraph.hs*。修改后的 *Trajectory.cabal* 中的行如下：

    ```
      main-is: MakeTrajectoryGraph.hs
    ```

1.  将包含 `SimpleVec` 模块的文件 *SimpleVec.hs* 复制或移动到 *Trajectory* 目录下的 *src* 子目录中。该文件以及本书中的所有其他模块可以在 [`lpfp.io`](https://lpfp.io) 获取。然后编辑 *Trajectory.cabal*，将 `SimpleVec` 模块添加到 `library` 部分的 `exposed-modules:` 字段中。

    ```
    library
      exposed-modules:
          SimpleVec
    ```

1.  编辑 *Trajectory.cabal*，在 `executable` 部分的 `build` `-depends:` 头下包含 `gnuplot` 包。这允许我们在主程序中导入 `Graphics.Gnuplot.Simple` 模块。做完这三项更改后，*Trajectory.cabal* 中的修改行如下：```
    library
      exposed-modules:
          SimpleVec
      other-modules:
          Paths_Trajectory
      hs-source-dirs:
          src
      build-depends:
          base >=4.7 && <5
      default-language: Haskell2010

    executable Trajectory-exe
      main-is: MakeTrajectoryGraph.hs
      other-modules:
          Paths_Trajectory
      hs-source-dirs:
          app
      ghc-options: -threaded -rtsopts -with-rtsopts=-N
      build-depends:
          Trajectory
        , base >=4.7 && <5
        , gnuplot
      default-language: Haskell2010
    ```

请记住，构建依赖中需要包含的是包名，而不是模块名。当使用 `stack` 时，错误地将模块名 `Graphics.Gnuplot.Simple` 替换为包名 `gnuplot` 会导致解析错误，并且没有任何提示告诉你真正的问题所在。

现在重新发布

```
$ stack install
```

重新编译名为 *Trajectory-exe* 的程序。我们可以用以下命令测试该可执行文件：

```
$ Trajectory-exe
```

可执行文件应当创建一个名为 *projectile.png* 的文件。

`stack` 安装的包，例如 `gnuplot`，存放在 [*https://hackage.haskell.org*](https://hackage.haskell.org) 上。你可以访问该网站，搜索、浏览并阅读关于 `stack` 可以安装的任何包的文档。

### 总结

本章展示了三种生成独立 Haskell 程序的方法。第一种使用 `ghc`，你必须自己安装任何需要的库包。第二种使用 `cabal`，它可以帮助管理库包的依赖。第三种使用 `stack`，它同样可以帮助管理库包的依赖。在下一章，我们将把这些技巧应用于制作动画。

### 练习

**练习 12.1.** print 函数在独立程序中非常有用。询问 GHCi print 的类型，GHCi 会告诉你 print 是一个函数，它的输入可以是任何 Show 实例的类型，输出是 IO ()，意味着它*执行*某些操作。print 的作用是将输入的值输出到屏幕上。你可以打印数字、列表、字符串以及任何可以显示的内容。你可以在 GHCi 中使用 print，但在 GHCi 中并不需要它，因为 GHCi 会自动打印你提供的值。

编写一个独立程序，打印前 21 个 2 的幂，从 2⁰ 到 2²⁰。当你运行程序时，输出应该如下所示：

```
[1,2,4,8,16,32,64,128,256,512,1024,2048,4096,8192,16384,32768,65536,131072,
262144,524288,1048576]
```
