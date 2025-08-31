## 第十二章：12

**异常、计时和可见性**

![image](img/common-01.jpg)

现在你已经看到如何在 R 中编写自己的函数，我们来看看一些常见的函数扩展和行为。在本章中，你将学习如何让你的函数在接收到意外输入时抛出错误或警告。你还将了解一些简单的方式来测量完成时间并检查计算密集型函数的进度。最后，你将看到 R 如何在两个同名但位于不同包中的函数之间进行遮蔽。

### 12.1 异常处理

当函数执行过程中遇到意外问题时，R 会通知你，可能是一个*警告*或*错误*。在本节中，我将演示如何在适当的情况下将这些构造体构建到你自己的函数中。我还将展示如何*尝试*一个计算，检查它是否可以在没有错误的情况下执行（即，看看它是否能正常工作）。

#### *12.1.1 正式通知：错误和警告*

在第十一章，当你的函数无法执行某些操作时，你让它们打印出一个字符串（例如，`"无效矩阵"`）。警告和错误是更正式的机制，用于传达这些类型的信息并处理后续操作。错误会强制函数在发生错误的地方立即终止。警告则较为轻微，表示函数以不典型的方式运行，但会尝试绕过问题并继续执行。在 R 中，你可以使用`warning`命令发出警告，使用`stop`命令抛出错误。以下两个函数展示了各自的例子：

```
warn_test <- function(x){
    if(x<=0){
        warning("'x' is less than or equal to 0 but setting it to 1 and
                continuing")
        x <- 1
    }
    return(5/x)
}

error_test <- function(x){
    if(x<=0){
        stop("'x' is less than or equal to 0... TERMINATE")
    }
    return(5/x)
}
```

`warn_test`和`error_test`都将 5 除以参数`x`。它们也都期望`x`是正数。在`warn_test`中，如果`x`是非正数，函数会发出警告，并将`x`的值覆盖为`1`。而在`error_test`中，如果`x`是非正数，函数会抛出一个错误并立即终止。两个命令`warning`和`stop`都使用字符字符串参数，作为打印到控制台的消息。

你可以通过以下方式导入并调用这些函数来查看通知：

```
R> warn_test(0)
[1] 5
Warning message:
In warn_test(0) :
  'x' is less than or equal to 0 but setting it to 1 and continuing
R> error_test(0)
Error in error_test(0) : 'x' is less than or equal to 0... TERMINATE
```

注意，`warn_test`已继续执行并返回了值`5`—这是将`x`设为`1`后，`5/1`的结果。`error_test`的调用没有返回任何内容，因为 R 在`stop`命令处退出了该函数。

警告在函数即使未得到预期的输入时，仍能以某种自然方式尝试自我修复时非常有用。例如，在第 10.1.3 节中，当你为 `if` 语句提供了一个逻辑向量时，R 会发出警告。请记住，`if` 语句期望一个单一的逻辑值，但如果提供了一个逻辑向量，它不会退出，而是继续执行，使用提供向量中的第一个条目。也就是说，有时实际上抛出错误并完全停止执行会更为合适。

让我们回到第 11.3.3 节中的 `myfibrec` 函数。这个函数期望一个正整数（它应该返回的斐波那契数的位置）。假设你认为如果用户提供了一个负整数，那么用户实际上是想要这个数值的正数版本。你可以添加一个警告来处理这种情况。同时，如果用户输入 0，这个数值在斐波那契数列中没有对应的位置，代码将抛出错误。考虑以下修改：

```
myfibrec2 <- function(n){
    if(n<0){
        warning("Assuming you meant 'n' to be positive -- doing that instead")
        n <- n*-1
    } else if(n==0){
        stop("'n' is uninterpretable at 0")
    }

    if(n==1||n==2){
        return(1)
    } else {
        return(myfibrec2(n-1)+myfibrec2(n-2))
    }
}
```

在 `myfibrec2` 中，你现在检查 `n` 是否为负数或零。如果是负数，函数会发出警告并在交换参数符号后继续执行。如果 `n` 为零，错误会终止执行并显示相应的消息。你可以看到以下不同参数的响应：

```
R> myfibrec2(6)
[1] 8
R> myfibrec2(-3)
[1] 2
Warning message:
In myfibrec2(-3) :
  Assuming you meant 'n' to be positive -- doing that instead
R> myfibrec2(0)
Error in myfibrec2(0) : 'n' is uninterpretable at 0
```

请注意，调用 `myfibrec2(-3)` 返回的是第三个斐波那契数。

广义来说，错误和警告都表明发生了问题。如果你正在使用某个函数或运行代码块，并遇到这些信息，你应该仔细查看已执行的操作以及可能导致这些问题的原因。

**注意**

*识别和修复错误代码被称为* 调试 *，对此有多种策略。最基本的策略之一是使用* `print` *或* `cat` *命令，在实时执行过程中检查计算的各种量。R 确实有一些更复杂的调试工具；如果你有兴趣，可以查看在《*The Art of R Programming*》一书中由 Matloff 提供的第十三章的精彩讨论（2011）。更多的一般性讨论可以在 Matloff 和 Salzman 合著的《*The Art of Debugging*》（2008）中找到。随着你在 R 中积累更多经验，理解错误信息或在问题出现之前定位潜在问题变得越来越容易，这是你部分得益于 R 的解释性风格。*

#### *12.1.2 使用 try 语句捕获错误*

当一个函数因错误终止时，它也会终止任何父函数的执行。例如，如果函数 A 调用函数 B，而函数 B 因错误停止执行，这会导致函数 A 在同一位置停止执行。为了避免这种严重后果，你可以使用 `try` 语句来尝试函数调用并检查是否产生错误。你还可以使用 `if` 语句来指定替代操作，而不是让所有流程停止。

例如，如果你调用之前的 `myfibrec2` 函数并传入 0，函数会抛出错误并终止。但是，看看当你将该函数调用作为第一个参数传递给 `try` 时会发生什么：

```
R> attempt1 <- try(myfibrec2(0),silent=TRUE)
```

似乎什么都没有发生。错误去哪了？实际上，错误仍然发生了，但由于你将 `silent` 设置为 `TRUE`，`try` 抑制了错误信息的打印。错误信息现在被存储在对象 `attempt1` 中，该对象属于 `"try-error"` 类。要查看错误，只需将 `attempt1` 打印到控制台：

```
R> attempt1
[1] "Error in myfibrec2(0) : 'n' is uninterpretable at 0\n"
attr(,"class")
[1] "try-error"
attr(,"condition")
<simpleError in myfibrec2(0): 'n' is uninterpretable at 0>
```

如果你将 `silent` 设置为 `FALSE`，你会看到这个错误信息打印到控制台。以这种方式捕获错误非常有用，尤其是当一个函数在另一个函数的主体代码中产生错误时。使用 `try`，你可以在不终止父函数的情况下处理错误。

同时，如果你将一个函数传递给 `try`，而该函数没有抛出错误，那么 `try` 就不会产生任何影响，你会得到正常的返回值。

```
R> attempt2 <- try(myfibrec2(6),silent=TRUE)
R> attempt2
[1] 8
```

在这里，你用一个有效的参数 `n=6` 执行了 `myfibrec2`。由于此调用没有产生错误，传递给 `attempt2` 的结果是 `myfibrec2` 的正常返回值，在此情况下是 `8`。

##### 在函数主体中使用 try

让我们看一个更完整的例子，展示如何在更大的函数中使用 `try`。以下 `myfibvector` 函数接受一个索引向量作为参数 `nvec`，并提供斐波那契数列中相应的项：

```
myfibvector <- function(nvec){
    nterms <- length(nvec)
    result <- rep(0,nterms)
    for(i in 1:nterms){
        result[i] <- myfibrec2(nvec[i])
    }
    return(result)
}
```

这个函数使用 `for` 循环逐个处理 `nvec` 中的元素，利用之前的函数 `myfibrec2` 计算相应的斐波那契数。只要 `nvec` 中的所有值非零，`myfibvector` 就能正常工作。例如，以下调用会得到第一个、第二个、第十个和第八个斐波那契数：

```
R> foo <- myfibvector(nvec=c(1,2,10,8))
R> foo
[1]  1  1 55 21
```

假设有一个错误，`nvec` 中的某个条目是零。

```
R> bar <- myfibvector(nvec=c(3,2,7,0,9,13))
Error in myfibrec2(nvec[i]) : 'n' is uninterpretable at 0
```

当在 `n=0` 时调用 `myfibrec2` 时，内部调用抛出了一个错误，这导致 `myfibvector` 执行终止。没有返回任何结果，整个调用失败。

你可以通过在 `for` 循环中使用 `try` 来防止这种失败，检查每次调用 `myfibrec2` 并捕获任何错误。以下函数 `myfibvectorTRY` 就实现了这一点。

```
myfibvectorTRY <- function(nvec){
    nterms <- length(nvec)
    result <- rep(0,nterms)
    for(i in 1:nterms){
        attempt <- try(myfibrec2(nvec[i]),silent=T)
        if(class(attempt)=="try-error"){
            result[i] <- NA
        } else {
            result[i] <- attempt
        }
    }
    return(result)
}
```

在`for`循环中，你使用`attempt`存储每次调用`myfibrec2`的结果。然后，你检查`attempt`。如果该对象的类是`"try-error"`，则表示`myfibrec2`产生了错误，你在`result`向量中相应的位置填充`NA`。否则，`attempt`将代表`myfibrec2`的有效返回值，因此你将其放入`result`向量的相应位置。现在，如果你导入并在相同的`nvec`上调用`myfibvectorTRY`，你会看到完整的结果集。

```
R> baz <- myfibvectorTRY(nvec=c(3,2,7,0,9,13))
R> baz
[1]   2   1  13  NA  34 233
```

本来会导致终止的错误被悄无声息地捕获，替代的响应是`NA`，它被插入到`result`向量中。

**注意**

*`try`*命令是 R 中更复杂的*`tryCatch`*函数的简化版本，后者超出了本书的讨论范围，但它提供了更精确的控制方式，用于测试和执行代码块。如果你有兴趣了解更多，输入*`?tryCatch`*以获得帮助。

##### 抑制警告消息

在我展示的所有`try`调用中，我都将`silent`参数设置为`TRUE`，这样可以停止打印错误消息。如果将`silent`设置为`FALSE`（默认值），则错误消息会被打印出来，但错误仍然会被捕获而不会终止执行。

请注意，设置`silent=TRUE`仅会抑制错误消息，而不会抑制警告消息。请观察以下内容：

```
R> attempt3 <- try(myfibrec2(-3),silent=TRUE)
Warning message:
In myfibrec2(-3) :
  Assuming you meant 'n' to be positive -- doing that instead
R> attempt3
[1] 2
```

尽管`silent`设置为`TRUE`，但仍然会发出警告（在这个例子中是针对`n`的负值）。警告在这种情况下与错误被分别处理，因为它们应该被分开处理——警告可以在代码执行过程中高亮显示其他未预见到的问题。如果你完全确定不希望看到任何警告，可以使用`suppressWarnings`。

```
R> attempt4 <- suppressWarnings(myfibrec2(-3))
R> attempt4
[1] 2
```

`suppressWarnings`函数应仅在你确定可以安全忽略某个调用中的每个警告，并且希望保持输出整洁时使用。

**习题 12.1**

1.  在习题 11.3 (b)中，第 238 页的任务是编写一个递归的 R 函数来计算整数阶乘，给定某个非负整数`x`。现在，修改你的函数，使其在`x`为负时抛出错误（并给出适当的消息）。通过以下方法测试你的新函数响应：

    1.  `x`为`5`

    1.  `x`为`8`

    1.  `x`为`-8`

1.  *矩阵求逆*的概念在第 3.3.6 节中简要讨论，仅对某些方阵（列数与行数相等的矩阵）有效。这些逆矩阵可以通过`solve`函数来计算，例如：

    ```
    R> solve(matrix(1:4,2,2))
         [,1] [,2]
    [1,]   -2  1.5
    [2,]    1 -0.5
    ```

    请注意，如果提供的矩阵无法求逆，`solve`会抛出错误。考虑到这一点，编写一个 R 函数，尝试根据以下指南对列表中的每个矩阵进行求逆：

    – 该函数应接受四个参数。

    * 要测试是否能进行矩阵求逆的列表`x`

    * 一个值`noninv`，如果`x`的给定矩阵成员无法求逆，则填充结果，默认值为`NA`。

    * 一个字符字符串`nonmat`，如果`x`的给定成员不是矩阵，则返回该结果，默认值为`"not a matrix"`。

    * 一个逻辑值`silent`，默认为`TRUE`，传递给`try`函数的主体代码。

    – 函数应该首先检查`x`是否为列表。如果不是，应该抛出一个带有适当信息的错误。

    – 然后，函数应确保`x`至少包含一个成员。如果没有，应该抛出一个带有适当信息的错误。

    – 接下来，函数应检查`nonmat`是否为字符字符串。如果不是，应该尝试使用适当的“as-dot”函数（见第 6.2.4 节）将其强制转换为字符字符串，并且应发出适当的警告。

    – 在这些检查之后，循环应该检查列表`x`的每个成员`i`。

    * 如果成员`i`是矩阵，尝试使用`try`对其进行求逆。如果可以无误地求逆，则用结果覆盖`x`中的成员`i`。如果捕获到错误，则应用`noninv`的值覆盖`x`中的成员`i`。

    * 如果成员`i`不是矩阵，则应用`nonmat`的值覆盖`x`中的成员`i`。

    – 最后，修改后的列表`x`应被返回。

    现在，使用以下参数值测试你的函数，以确保其按预期响应：

    1.  `x`为

        ```
        list(1:4,matrix(1:4,1,4),matrix(1:4,4,1),matrix(1:4,2,2))
        ```

    以及所有其他参数均使用默认值。

    1.  `x`如(i)所示，`noninv`为`Inf`，`nonmat`为`666`，`silent`使用默认值。

    1.  重复(ii)，但这次`silent=FALSE`。

    1.  `x`为

    ```
    list(diag(9),matrix(c(0.2,0.4,0.2,0.1,0.1,0.2),3,3),
         rbind(c(5,5,1,2),c(2,2,1,8),c(6,1,5,5),c(1,0,2,0)),
         matrix(1:6,2,3),cbind(c(3,5),c(6,5)),as.vector(diag(2)))
    ```

    以及`noninv`为`"unsuitable matrix"`；所有其他值使用默认值。

    最后，通过以下调用测试错误信息，确保你的函数能按预期响应：

    1.  `x`为`"hello"`

    1.  `x`为`list()`

### 12.2 进度和计时

R 常用于长时间的数值计算，如模拟或随机变量生成。对于这些复杂、耗时的操作，通常很有用的是跟踪进度或查看某个任务完成所花的时间。例如，你可能想要比较两种不同编程方法在解决同一问题时的速度。在本节中，你将学习如何计时代码执行并显示其进度。

#### *12.2.1 文本进度条：我们快到了吗？*

一个*进度条*显示 R 在执行一组操作时的进展情况。为了演示这一点，你需要运行一些需要一定时间才能执行的代码，你可以通过让 R*休眠*来实现。`Sys.sleep`命令使 R 在继续执行之前暂停指定的秒数。

```
R> Sys.sleep(3)
```

如果你运行这段代码，R 将在继续使用控制台之前暂停三秒钟。休眠将被用作在这部分中替代由于计算量大而造成的延迟，这正是进度条最有用的地方。

要更常规地使用`Sys.sleep`，可以考虑以下方式：

```
sleep_test <- function(n){
    result <- 0
    for(i in 1:n){
        result <- result + 1
        Sys.sleep(0.5)
    }
    return(result)
}
```

`sleep_test`函数是基本的——它接受一个正整数`n`，并在`n`次迭代中，每次都将`result`值加`1`。在每次迭代中，你还会告诉循环休眠半秒。由于有这个休眠命令，执行以下代码大约需要四秒钟才能返回结果：

```
R> sleep_test(8)
[1] 8
```

现在，假设你想要跟踪这种类型的函数执行进度。你可以通过三步来实现文本进度条：使用`txtProgressBar`初始化进度条对象，使用`setTxtProgressBar`更新进度条，使用`close`终止进度条。下一个函数`prog_test`修改了`sleep_test`，加入了这三个命令。

```
prog_test <- function(n){
    result <- 0
    progbar <- txtProgressBar(min=0,max=n,style=1,char="=")
    for(i in 1:n){
        result <- result + 1
        Sys.sleep(0.5)
        setTxtProgressBar(progbar,value=i)
    }
    close(progbar)
    return(result)
}
```

在`for`循环之前，你通过调用`txtProgressBar`并传入四个参数来创建一个名为`progbar`的对象。`min`和`max`参数是定义进度条范围的数值。在这种情况下，你设置`max=n`，它与即将执行的`for`循环的迭代次数相匹配。`style`参数（整数，可以是`1`、`2`或`3`）和`char`参数（字符字符串，通常是单个字符）决定了进度条的外观。设置`style=1`表示进度条将仅显示一行`char`；如果`char="="`，则会显示一系列等号。

一旦创建了这个对象，你需要通过调用`setTxtProgressBar`来指示进度条在执行过程中实际前进。你将进度条对象（`progbar`）和需要更新的`value`（在这种情况下是`i`）传递给它。完成后（退出循环之后），进度条必须通过调用`close`来终止，传入相关的进度条对象。导入并执行`prog_test`，你将看到等号`"="`在循环完成时逐步绘制出来。

```
R> prog_test(8)
================================================================
[1] 8
```

进度条的宽度默认由执行`txtProgressBar`命令时，R 控制台窗格的宽度决定。你可以通过改变`style`和`char`参数来稍微定制进度条。例如，选择`style=3`会显示进度条，并且还会有一个“完成百分比”计数器。一些包还提供了更复杂的选项，比如弹出小部件，但文本版本是最简单且在不同系统中兼容性最好的版本。

#### *12.2.2 测量完成时间：需要多长时间？*

如果你想知道一个计算任务需要多长时间才能完成，可以使用`Sys.time`命令。该命令输出一个对象，详细列出基于你系统的当前日期和时间信息。

```
R> Sys.time()
[1] "2016-03-06 16:39:27 NZDT"
```

你可以在某些代码执行前后存储这些对象，然后比较它们，以查看经过了多少时间。在编辑器中输入以下内容：

```
t1 <- Sys.time()
Sys.sleep(3)
t2 <- Sys.time()
t2-t1
```

现在高亮显示这四行并在控制台中执行它们。

```
R> t1 <- Sys.time()
R> Sys.sleep(3)
R> t2 <- Sys.time()
R> t2-t1
Time difference of 3.012889 secs
```

通过一起执行整个代码块，你可以轻松地衡量总完成时间，并将格式良好的字符串打印到控制台。请注意，解释和调用任何命令都需要一点时间，除了你告诉 R 休眠的三秒钟外，这个时间在不同计算机之间会有所不同。

如果你需要更详细的计时报告，有更复杂的工具可供使用。例如，你可以使用`proc.time()`来获得不仅是总的“墙钟”时间，还包括计算机相关的 CPU 时间（请参见帮助文件`?proc.time`中的定义）。要计时一个单独的表达式，你还可以使用`system.time`函数（它的输出细节与`proc.time`相同）。还有*基准测试*工具（对不同方法的正式或系统性比较）用于计时你的代码；例如，参见`rbenchmark`包（Kusnierczyk, 2012）。然而，对于日常使用，本文使用的时间对象差分方法易于理解，并提供了关于计算开销的良好指示。

**习题 12.2**

1.  修改第 12.2.1 节中的`prog_test`，使其参数列表中包含省略号，旨在接收`txtProgressBar`中的附加参数；将新函数命名为`prog_test_fancy`。计时`prog_test_fancy`执行所需的时间。设置`50`为`n`，通过省略号指示进度条使用`style=3`，并将进度条字符设置为`"r"`。

1.  在第 12.1.2 节中，你定义了一个名为`myfibvectorTRY`的函数（它本身调用了第 12.1.1 节中的`myfibrec2`），用于根据提供的“项向量”`nvec`返回斐波那契数列的多个项。编写一个新版本的`myfibvectorTRY`，其中包含一个`style=3`的进度条，以及你选择的字符，在每次通过内部`for`循环时递增。然后，执行以下操作：

    1.  使用你的新函数重新生成文本中`nvec=c(3,2,7,0,9,13)`的结果。

    1.  计时使用你的新函数返回斐波那契数列前 35 项所需的时间。你注意到了什么？这说明了你的递归斐波那契函数的什么问题？

1.  继续使用斐波那契数列。编写一个独立的`for`循环，用来计算并存储前 35 项（与(b)(ii)中的相同）。并进行计时。你更喜欢哪种方法？

### 12.3 屏蔽

由于 R 中有大量内建和贡献的数据和功能，几乎不可避免地，你会在某些时候遇到那些在不同加载的包中共享相同名称的对象，通常是函数。

那么，在那些情况下会发生什么呢？例如，假设你定义了一个与已加载的 R 包中的函数同名的函数。R 会通过*屏蔽*其中一个对象来响应——也就是说，一个对象或函数将优先于另一个，并假定该对象或函数的名称，而被屏蔽的函数必须通过额外的命令进行调用。这可以防止对象互相覆盖或阻塞。在本节中，你将了解 R 中最常见的两种屏蔽情况。

#### *12.3.1 函数与对象的区别*

当不同环境中的两个函数或对象具有相同的名称时，搜索路径中较早的对象会覆盖较晚的对象。也就是说，当搜索该对象时，R 会使用它首先找到的对象或函数，且你需要额外的代码才能访问另一个被覆盖的版本。记住，你可以通过执行 `search()` 来查看当前的搜索路径。

```
R> search()
 [1] ".GlobalEnv"        "tools:RGUI"        "package:stats"
 [4] "package:graphics"  "package:grDevices" "package:utils"
 [7] "package:datasets"  "package:methods"   "Autoloads"
[10] "package:base"
```

当 R 搜索时，搜索路径中最接近起始位置（全局环境）的函数或对象会首先被找到，并覆盖路径中稍后的同名函数或对象。为了展示遮蔽的简单例子，你将定义一个与基础包中的 `sum` 函数同名的函数：`sum`。以下是 `sum` 正常工作的方式，它会将向量 `foo` 中的所有元素加起来：

```
R> foo <- c(4,1.5,3)
R> sum(foo)
[1] 8.5
```

现在，假设你输入以下函数：

```
sum <- function(x){
    result <- 0
    for(i in 1:length(x)){
        result <- result + x[i]²
    }
    return(result)
}
```

这个版本的 `sum` 接收一个向量 `x`，并使用 `for` 循环将每个元素平方后再求和并返回结果。这可以毫无问题地导入到 R 控制台，但显然，它提供的功能与内建的（原始）版本的 `sum` 不同。现在，在导入该函数后，如果你调用 `sum`，将使用你自己定义的版本。

```
R> sum(foo)
[1] 27.25
```

之所以发生这种情况，是因为用户自定义的函数存储在全局环境（`.GlobalEnv`）中，而全局环境总是位于搜索路径的最前面。R 的内建函数属于 `base` 包，它位于搜索路径的最后。此时，用户自定义的函数遮蔽了原函数。

现在，如果你希望 R 运行 `base` 版本的 `sum`，你必须在调用时包括它所属包的名称，并使用双冒号。

```
R> base::sum(foo)
[1] 8.5
```

这会告诉 R 使用 `base` 中的版本，即使全局环境中有另一个版本的函数。

为了避免任何混淆，让我们从全局环境中移除 `sum` 函数。

```
R> rm(sum)
```

##### 当包对象发生冲突时

当你加载一个包时，R 会通知你包中的任何对象是否与当前会话中可以访问的其他对象发生冲突。为了说明这一点，我将使用两个贡献包：`car` 包（你在 练习 8.1 (b) 中见过，位于 第 162 页）和 `spatstat` 包（你将在 第 V 部分中使用）。确保这两个包已经安装后，当我按照以下顺序加载它们时，我会看到这个信息：

```
R> library("spatstat")
spatstat 1.40-0       (nickname: 'Do The Maths')
For an introduction to spatstat, type 'beginner'
R> library("car")

Attaching package: 'car'

The following object is masked from 'package:spatstat':

    ellipse
```

这表明两个包中各自有一个同名对象——`ellipse`。R 自动通知你该对象正在被遮蔽。注意，`car` 和 `spatstat` 的功能仍然完全可用；只是如果需要使用 `ellipse` 对象，它们需要加以区分。使用提示符中的 `ellipse` 将访问 `car` 的对象，因为该包加载得更晚。若要使用 `spatstat` 的版本，必须输入 `spatstat::ellipse`。这些规则同样适用于访问各自的帮助文件。

当你加载一个包含被全局环境对象（全局环境对象总是优先于包对象）的对象时，会出现类似的通知。要查看一个例子，你可以加载`MASS`包（Venables 和 Ripley，2002），这个包是 R 自带的，但不会自动加载。继续在当前的 R 会话中，创建以下对象：

```
R> cats <- "meow"
```

现在，假设你需要加载`MASS`。

```
R> library("MASS")

Attaching package: 'MASS'

The following object is masked _by_ '.GlobalEnv':

    cats

The following object is masked from 'package:spatstat':

    area
```

加载包后，你会被通知到，你刚创建的`cats`对象正在遮蔽`MASS`中同名的对象。（如你在`?MASS::cats`中所见，这个对象是一个包含家猫体重测量的 数据框。）此外，`MASS`似乎也与`spatstat`共享一个对象名称——`area`。对于该特定项，显示了与之前相同的“包遮蔽”消息。

##### 卸载包

你可以从搜索路径中卸载已加载的包。根据本讨论中加载的包，我当前的搜索路径如下：

```
R> search()
 [1] ".GlobalEnv"        "package:MASS"      "package:car"
 [4] "package:spatstat"  "tools:RGUI"        "package:stats"
 [7] "package:graphics"  "package:grDevices" "package:utils"
[10] "package:datasets"  "package:methods"   "Autoloads"
[13] "package:base"
```

现在，假设你不再需要`car`。你可以通过`detach`函数将其移除，方法如下。

```
R> detach("package:car",unload=TRUE)
R> search()
 [1] ".GlobalEnv"        "package:MASS"     "package:spatstat"
 [4] "tools:RGUI"        "package:stats"    "package:graphics"
 [7] "package:grDevices" "package:utils"    "package:datasets"
[10] "package:methods"   "Autoloads"        "package:base"
```

这将从路径中移除选定的包，卸载其命名空间。现在，`car`的功能不再立即可用，`spatstat`的`ellipsis`函数也不再被遮蔽。

**注意**

*随着贡献包被维护者更新，它们可能会包含新的对象，导致新的遮蔽，或者移除或重命名以前引起遮蔽的对象（与其他贡献包相比）。这里所示的`car`、`spatstat`和`MASS`之间的具体遮蔽发生在写作时的版本，并可能在未来发生变化。*

#### *12.3.2 数据框变量区分*

还有一种常见情况，你会明确收到遮蔽通知：当你将一个数据框添加到搜索路径时。让我们看看这如何工作。继续在当前工作区中，定义以下数据框：

```
R> foo <- data.frame(surname=c("a","b","c","d"),
                     sex=c(0,1,1,0),height=c(170,168,181,180),
                     stringsAsFactors=F)
R> foo
  surname sex height
1       a   0    170
2       b   1    168
3       c   1    181
4       d   0    180
```

数据框`foo`有三个列变量：`person`、`sex`和`height`。要访问这些列中的一个，通常需要使用`$`运算符，输入类似`foo$surname`的内容。然而，你可以*附加*一个数据框到你的搜索路径，这样更容易访问一个变量。

```
R> attach(foo)
R> search()
 [1] ".GlobalEnv"        "foo"               "package:MASS"
 [4] "package:spatstat"  "tools:RGUI"        "package:stats"
 [7] "package:graphics"  "package:grDevices" "package:utils"
[10] "package:datasets"  "package:methods"   "Autoloads"
[13] "package:base"
```

现在`surname`变量可以直接访问了。

```
R> surname
[1] "a" "b" "c" "d"
```

这可以避免每次访问一个变量时都输入`foo$`，如果你的分析完全处理一个静态且不变的数据框，这可以是一个便捷的快捷方式。然而，如果你忘记了附加的对象，它们可能会在之后造成问题，特别是如果你在同一会话中继续将更多对象挂载到搜索路径中。例如，假设你输入了另一个数据框。

```
R> bar <- data.frame(surname=c("e","f","g","h"),
                     sex=c(1,0,1,0),weight=c(55,70,87,79),
                     stringsAsFactors=F)
R> bar
  surname sex weight
1       e   1     55
2       f   0     70
3       g   1     87
4       h   0     79
```

然后也将其添加到搜索路径中。

```
R> attach(bar)
The following objects are masked from foo:

    sex, surname
```

通知告诉你，`bar`对象现在在搜索路径中排在`foo`之前。

```
R> search()
 [1] ".GlobalEnv"       "bar"               "foo"
 [4] "package:MASS"     "package:spatstat"  "tools:RGUI"
 [7] "package:stats"    "package:graphics"  "package:grDevices"
[10] "package:utils"    "package:datasets"  "package:methods"
[13] "Autoloads"        "package:base"
```

结果是，任何直接使用`sex`或`surname`的操作现在将访问`bar`的内容，而不是`foo`的内容。同时，未遮蔽的变量`height`来自`foo`，仍然可以直接访问。

```
R> height
[1] 170 168 181 180
```

这是一个相当简单的例子，但它突出了在将数据框、列表或其他对象添加到搜索路径时可能出现的混淆。以这种方式挂载对象可能会迅速变得难以追踪，尤其是对于包含许多不同变量的大型数据集。因此，作为一般准则，最好避免以这种方式附加对象——除非如前所述，你仅仅在处理一个数据框。

请注意，`detach`可以用于从搜索路径中移除对象，方法与之前看到的移除包的方法类似。在这种情况下，你只需输入对象的名称即可。

```
R> detach(foo)
R> search()
 [1] ".GlobalEnv"        "bar"               "package:MASS"
 [4] "package:spatstat"  "tools:RGUI"        "package:stats"
 [7] "package:graphics"  "package:grDevices" "package:utils"
[10] "package:datasets"  "package:methods"   "Autoloads"
[13] "package:base"
```

##### 本章重要代码

| **函数/操作符** | **简要描述** | **首次出现** |
| --- | --- | --- |
| `warning` | 发出警告 | 第 12.1.1 节, 第 242 页 |
| `stop` | 抛出错误 | 第 12.1.1 节, 第 242 页 |
| `try` | 尝试捕获错误 | 第 12.1.2 节, 第 244 页 |
| `Sys.sleep` | 睡眠（暂停）执行 | 第 12.2.1 节, 第 249 页 |
| `txtProgressBar` | 初始化进度条 | 第 12.2.1 节, 第 249 页 |
| `setTxtProgressBar` | 增加进度条 | 第 12.2.1 节, 第 249 页 |
| `close` | 关闭进度条 | 第 12.2.1 节, 第 249 页 |
| `Sys.time` | 获取本地系统时间 | 第 12.2.2 节, 第 250 页 |
| `detach` | 从路径中移除库/对象 | 第 12.3.1 节, 第 255 页 |
| `attach` | 将对象附加到搜索路径 | 第 12.3.2 节, 第 256 页 |
