## 第五章：5

**列表和数据框**

![image](img/common-01.jpg)

向量、矩阵和数组是 R 中高效且方便的数据存储结构，但它们有一个明显的限制：它们只能存储一种类型的数据。在本章中，你将探讨另外两种数据结构，列表和数据框，它们可以同时存储多种类型的值。

### 5.1 对象的列表

*列表*是一种非常有用的数据结构。它可以用来将任何类型的 R 结构和对象组合在一起。一个单一的列表可以包含一个数值矩阵、一个逻辑数组、一个单一的字符字符串和一个因子对象。你甚至可以将一个列表作为另一个列表的组件。在本节中，你将学习如何创建、修改和访问这些灵活结构的组件。

#### *5.1.1 定义和组件访问*

创建一个列表与创建一个向量非常类似。你将想要包含的元素提供给`list`函数，并用逗号分隔。

```
R> foo <- list(matrix(data=1:4,nrow=2,ncol=2),c(T,F,T,T),"hello")
R> foo
[[1]]
     [,1] [,2]
[1,]    1    3
[2,]    2    4

[[2]]
[1]  TRUE FALSE  TRUE  TRUE

[[3]]
[1] "hello"
```

在列表`foo`中，你存储了一个 2 × 2 的数值矩阵，一个逻辑向量和一个字符字符串。这些元素会按它们提供给`list`函数的顺序打印出来。与向量一样，你可以使用`length`函数检查列表中的组件数量。

```
R> length(x=foo)
[1] 3
```

你可以使用索引来从列表中获取组件，索引是通过双中括号输入的。

```
R> foo[[1]]
     [,1] [,2]
[1,]    1    3
[2,]    2    4
R> foo[[3]]
[1] "hello"
```

这个操作被称为*成员引用*。当你通过这种方式获取组件时，可以像对待工作区中的独立对象一样对待它；不需要做任何特殊处理。

```
R> foo[[1]] + 5.5
     [,1] [,2]
[1,]  6.5  8.5
[2,]  7.5  9.5
R> foo[[1]][1,2]
[1] 3
R> foo[[1]][2,]
[1] 2 4
R> cat(foo[[3]],"you!")
hello you!
```

要覆盖`foo`的某个成员，你可以使用赋值运算符。

```
R> foo[[3]]
[1] "hello"
R> foo[[3]] <- paste(foo[[3]],"you!")
R> foo
[[1]]
     [,1] [,2]
[1,]    1    3
[2,]    2    4

[[2]]
[1]  TRUE FALSE  TRUE  TRUE

[[3]]
[1] "hello you!"
```

假设现在你想访问`foo`的第二和第三个组件，并将它们存储为一个对象。你可能的第一个直觉是尝试如下操作：

```
R> foo[[c(2,3)]]
[1] TRUE
```

但是 R 没有按你想要的方式工作。相反，它返回了第二个组件的第三个元素。这是因为在列表上使用双中括号总是按单个成员来解释的。幸运的是，使用双中括号进行成员引用并不是访问列表组件的唯一方式。你也可以使用单中括号表示法，这被称为*列表切片*，它允许你一次选择多个列表项。

```
R> bar <- foo[c(2,3)]
R> bar
[[1]]
[1]  TRUE FALSE  TRUE  TRUE

[[2]]
[1] "hello you!"
```

请注意，结果`bar`本身就是一个列表，其中包含按请求顺序存储的两个组件。

#### *5.1.2 命名*

你可以*命名*列表组件，以便让元素更易于识别和操作。就像你在第 4.3.1 节中看到的因子水平的信息一样，名称是 R 的*属性*。

让我们从之前的列表`foo`开始，给它添加名称。

```
R> names(foo) <- c("mymatrix","mylogicals","mystring")
R> foo
$mymatrix
     [,1] [,2]
[1,]    1    3
[2,]    2    4

$mylogicals
[1]  TRUE FALSE  TRUE  TRUE

$mystring
[1] "hello you!"
```

这改变了对象在控制台上的打印方式。之前它在每个组件前打印`[[1]]`、`[[2]]`和`[[3]]`，现在它打印你指定的名称：`$mymatrix`、`$mylogicals`和`$mystring`。现在，你可以使用这些名称和美元符号运算符来进行成员引用，而不是使用双中括号。

```
R> foo$mymatrix
     [,1] [,2]
[1,]    1    3
[2,]    2    4
```

这与调用`foo[[1]]`是一样的。实际上，即使一个对象已命名，你仍然可以使用数字索引来获取一个成员。

```
R> foo[[1]]
     [,1] [,2]
[1,]    1    3
[2,]    2    4
```

子集化命名成员的工作方式也是相同的。

```
R> all(foo$mymatrix[,2]==foo[[1]][,2])
[1] TRUE
```

这确认了（使用你在第 4.1.2 节中看到的`all`函数）这两种提取`foo`中矩阵第二列的方法，提供了相同的结果。

要在创建列表时命名其组件，可以在`list`命令中为每个组件分配一个标签。使用`foo`的一些组件，创建一个新的命名列表。

```
R> baz <- list(tom=c(foo[[2]],T,T,T,F),dick="g'day mate",harry=foo$mymatrix*2)
R> baz
$tom
[1]  TRUE FALSE  TRUE  TRUE  TRUE  TRUE  TRUE FALSE

$dick
[1] "g'day mate"

$harry
     [,1] [,2]
[1,]    2    6
[2,]    4    8
```

现在，`baz`对象包含了三个命名组件`tom`、`dick`和`harry`。

```
R> names(baz)
[1] "tom"   "dick"  "harry"
```

如果你想重命名这些成员，可以像之前为`foo`所做的那样，简单地将一个长度为 3 的字符向量赋值给`names(baz)`。

**注意**

*当使用* `names` *函数时，组件名称总是以双引号中的字符字符串形式提供和返回。然而，如果在创建列表时指定名称（在* `list` *函数内部），或者使用名称通过美元操作符提取成员时，名称则不带引号（换句话说，它们*不是*以字符串形式给出）。*

#### *5.1.3 嵌套*

如前所述，列表的一个成员本身可以是一个列表。当像这样嵌套列表时，重要的是要跟踪任何成员的深度，以便稍后进行子集提取或提取。

请注意，你可以通过使用美元操作符和一个*新*名称，向任何现有列表添加组件。这里是一个使用之前的`foo`和`baz`的示例：

```
R> baz$bobby <- foo
R> baz
$tom
[1]  TRUE FALSE  TRUE  TRUE  TRUE  TRUE  TRUE FALSE

$dick
[1] "g'day mate"

$harry
     [,1] [,2]
[1,]    2    6
[2,]    4    8

$bobby
$bobby$mymatrix
     [,1] [,2]
[1,]    1    3
[2,]    2    4

$bobby$mylogicals
[1]  TRUE FALSE  TRUE  TRUE

$bobby$mystring
[1] "hello you!"
```

这里，你定义了一个名为`bobby`的第四个组件，属于列表`baz`。成员`bobby`被赋予整个列表`foo`。如你所见，通过打印新的`baz`，`bobby`现在有三个组件。名称和索引现在都是分层的，你可以使用任意一个（或结合使用）来提取内部列表的成员。

```
R> baz$bobby$mylogicals[1:3]
[1]  TRUE FALSE  TRUE
R> baz[[4]][[2]][1:3]
[1]  TRUE FALSE  TRUE
R> baz[[4]]$mylogicals[1:3]
[1]  TRUE FALSE  TRUE
```

这些指令告诉 R 返回存储为列表`bobby`的第二个组件（`[[2]]`，也名为`mylogicals`）中的逻辑向量的前三个元素，而`bobby`又是列表`baz`的第四个组件。只要你了解每一层子集返回的内容，就可以继续根据需要使用名称和数字索引进行子集化。考虑此示例中的第三行。子集的第一层是`baz[[4]]`，它是一个包含三个组件的列表。第二层子集通过调用`baz[[4]]$mylogicals`从该列表中提取组件`mylogicals`。这个组件代表一个长度为 4 的向量，所以第三层子集通过`baz[[4]]$mylogicals[1:3]`提取该向量的前三个元素。

列表通常用于返回各种 R 函数的输出。但它们在系统资源上可能迅速变成相当大的对象。通常建议，当只有一种类型的数据时，应坚持使用基本的向量、矩阵或数组结构来记录和存储观察值。

**练习 5.1**

1.  创建一个列表，其中包含以下内容：按顺序排列的 20 个均匀分布的数字，介于 −4 和 4 之间；按列填充的 3 × 3 逻辑向量矩阵 `c(F,T,T,T,F,T,T,F,F)`；包含两个字符串 `"don"` 和 `"quixote"` 的字符向量；以及包含观察值 `c("LOW","MED","LOW","MED","MED","HIGH")` 的因子向量。然后执行以下操作：

    1.  提取逻辑矩阵中第 2 行、第 1 行的第 2 列和第 3 列元素，按此顺序。

    1.  使用 `sub` 将 `"quixote"` 替换为 `"Quixote"`，将 `"don"` 替换为 `"Don"`，并在列表中进行修改。然后，使用修改后的列表成员，精确地将以下语句连接到控制台屏幕上：

        ```
        "Windmills! ATTACK!"
            -\Don Quixote/-
        ```

    1.  获取序列中介于 −4 和 4 之间且大于 1 的所有值。

    1.  使用 `which` 确定因子向量中哪些索引被分配为 `"MED"` 级别。

1.  创建一个新列表，其中包含从 (a) 中获取的因子向量作为名为 `"facs"` 的组件；数值向量 `c(3,2.1,3.3,4,1.5,4.9)` 作为名为 `"nums"` 的组件；以及由 (a) 中列表的前三个成员组成的嵌套列表，命名为 `"oldlist"`。然后执行以下操作：

    1.  提取 `"facs"` 中对应于 `"nums"` 中大于或等于 3 的元素的项。

    1.  向列表中添加一个新成员 `"flags"`。该成员应为长度为 6 的逻辑向量，获取方式是将 `"oldlist"` 组件中的逻辑矩阵的第三列重复两次。

    1.  使用 `"flags"` 和逻辑非运算符 `!` 提取与 `FALSE` 对应的 `"num"` 项。

    1.  用单一字符字符串 `"Don Quixote"` 替换 `"oldlist"` 中的字符字符串向量组件。

### 5.2 数据框

*数据框* 是 R 中呈现数据集的最自然方式，它包含一个或多个变量的记录观察集合。与列表一样，数据框对变量的数据类型没有限制；你可以存储数值数据、因子数据等等。R 数据框可以被视为具有一些额外规则的列表。最重要的区别在于，在数据框中（与列表不同），成员必须是相同长度的向量。

数据框是 R 中最重要且最常用的统计数据分析工具之一。在本节中，你将学习如何创建数据框并了解其一般特征。

#### *5.2.1 构建*

要从头创建数据框，使用 `data.frame` 函数。你提供按变量分组的数据，这些数据作为相同长度的向量——就像你构造命名列表一样。考虑以下示例数据集：

```
R> mydata <- data.frame(person=c("Peter","Lois","Meg","Chris","Stewie"),
                        age=c(42,40,17,14,1),
                        sex=factor(c("M","F","F","M","M")))
R> mydata
  person age sex
1  Peter  42   M
2   Lois  40   F
3    Meg  17   F
4  Chris  14   M
5 Stewie   1   M
```

在这里，你已经构建了一个包含五个个体的名字、年龄（以年为单位）和性别的数据框。返回的对象应该清楚地说明为什么传递给 `data.frame` 的向量必须具有相同的长度：长度不同的向量在这个上下文中没有意义。如果你将长度不等的向量传递给 `data.frame`，那么 R 将尝试回收任何较短的向量，以匹配最长的向量，这会破坏你的数据，并可能将观察值分配到错误的变量中。请注意，数据框会以行和列的形式打印到控制台——它们看起来更像是矩阵而非命名列表。这种自然的电子表格样式使得读取和操作数据集变得更加容易。数据框中的每一行叫做 *记录*，每一列叫做 *变量*。

你可以通过指定行和列的索引位置来提取数据的部分内容（就像操作矩阵一样）。下面是一个示例：

```
R> mydata[2,2]
[1] 40
```

这会给你第二行第二列的元素——Lois 的年龄。现在提取第三列的第三、第四和第五个元素：

```
R> mydata[3:5,3]
[1] F M M
Levels: F M
```

这将返回一个因子向量，包含 Meg、Chris 和 Stewie 的性别。以下代码提取了第三列和第一列的整个数据（顺序为这样）：

```
R> mydata[,c(3,1)]
  sex person
1   M  Peter
2   F   Lois
3   F    Meg
4   M  Chris
5   M Stewie
```

这将生成另一个数据框，显示每个人的性别和姓名。

你还可以使用传递给 `data.frame` 的向量名称来访问变量，即使你不知道它们的列索引位置，这对于大数据集来说非常有用。你使用的是和引用命名列表成员时相同的美元符号操作符。

```
R> mydata$age
[1] 42 40 17 14  1
```

你也可以对这个返回的向量进行子集操作：

```
R> mydata$age[2]
[1] 40
```

这将返回与之前调用 `mydata[2,2]` 相同的结果。

你可以报告数据框的大小——记录数和变量数——就像你在矩阵的维度中看到的那样（首次展示于 第 3.1.3 节）。

```
R> nrow(mydata)
[1] 5
R> ncol(mydata)
[1] 3
R> dim(mydata)
[1] 5 3
```

`nrow` 函数获取行数（记录数），`ncol` 获取列数（变量数），`dim` 则返回两者。

R 在传递给 `data.frame` 的字符向量中的默认行为是将每个变量转换为因子对象。观察以下内容：

```
R> mydata$person
[1] Peter  Lois  Meg     Chris  Stewie
Levels: Chris Lois Meg Peter Stewie
```

注意，这个变量有层级，这表明它被视为一个因子。但是这不是你在之前定义 `mydata` 时的初衷——你明确地将 `sex` 定义为因子，但将 `person` 留作字符向量。为了防止在使用 `data.frame` 时字符字符串自动转换为因子，可以将可选参数 `stringsAsFactors` 设置为 `FALSE`（否则，它默认为 `TRUE`）。使用这种方式重新构建 `mydata` 如下所示：

```
R> mydata <- data.frame(person=c("Peter","Lois","Meg","Chris","Stewie"),
                        age=c(42,40,17,14,1),
                        sex=factor(c("M","F","F","M","M")),
                        stringsAsFactors=FALSE)
R> mydata
  person age sex
1  Peter  42   M
2   Lois  40   F
3    Meg  17   F
4  Chris  14   M
5 Stewie   1   M
R> mydata$person
[1] "Peter"  "Lois"   "Meg"   "Chris"  "Stewie"
```

你现在已经得到了期望的、非因子的 `person`。

#### *5.2.2 添加数据列和合并数据框*

假设你想要向现有的数据框添加数据。这可以是新增变量的观察值（增加列数），或者是更多的记录（增加行数）。同样，你可以使用一些之前已经应用于矩阵的函数。

回顾一下第 3.1.2 节中的`rbind`和`cbind`函数，它们分别让你追加行和列。这些相同的函数可以直观地用于扩展数据框。例如，假设你有另一个记录需要包含在`mydata`中：另一个人的年龄和性别，Brian。第一步是创建一个包含 Brian 信息的新数据框。

```
R> newrecord <- data.frame(person="Brian",age=7,
                           sex=factor("M",levels=levels(mydata$sex)))
R> newrecord
  person age sex
1  Brian   7   M
```

为了避免任何混淆，确保变量名和数据类型与你打算添加到的那个数据框匹配是非常重要的。请注意，对于因子，你可以使用`levels`提取现有因子变量的水平。

现在，你可以简单地调用以下内容：

```
R> mydata <- rbind(mydata,newrecord)
R> mydata
  person age sex
1  Peter  42   M
2   Lois  40   F
3    Meg  17   F
4  Chris  14   M
5 Stewie   1   M
6  Brian   7   M
```

使用`rbind`，你将`mydata`与新记录合并，并用结果覆盖了`mydata`。

向数据框添加变量也非常简单。假设现在你获得了关于这六个人的幽默程度分类数据，定义为“幽默度”。幽默度可以有三个可能的值：`Low`（低），`Med`（中），和`High`（高）。假设 Peter、Lois 和 Stewie 的幽默度很高，Chris 和 Brian 的幽默度为中等，而 Meg 的幽默度较低。在 R 中，你会有一个这样的因子向量：

```
R> funny <- c("High","High","Low","Med","High","Med")
R> funny <- factor(x=funny,levels=c("Low","Med","High"))
R> funny
[1] High High Low  Med  High Med
Levels: Low Med High
```

第一行创建了基本的字符向量`funny`，第二行通过将其转换为因子来覆盖`funny`。这些元素的顺序必须与数据框中的记录相对应。现在，你可以简单地使用`cbind`将这个因子向量作为一列附加到现有的`mydata`中。

```
R> mydata <- cbind(mydata,funny)
R> mydata
  person age sex funny
1  Peter  42   M  High
2   Lois  40   F  High
3    Meg  17   F   Low
4  Chris  14   M   Med
5 Stewie   1   M  High
6  Brian   7   M   Med
```

`rbind`和`cbind`函数并不是扩展数据框的唯一方式。添加变量的一个有用替代方法是使用美元符号运算符，类似于第 5.1.3 节中添加命名列表成员的方式。假设现在你想通过包含个体年龄（以月为单位，而不是年）来为`mydata`添加另一个变量，将此新变量命名为`age.mon`。

```
R> mydata$age.mon <- mydata$age*12
R> mydata
  person age sex funny age.mon
1  Peter  42   M  High     504
2   Lois  40   F  High     480
3    Meg  17   F   Low     204
4  Chris  14   M   Med     168
5 Stewie   1   M  High      12
6  Brian   7   M   Med      84
```

这使用美元符号运算符创建了一个新的`age.mon`列，并同时将其赋值为年龄（已经以年为单位存储在`age`中）乘以 12 的向量。

#### *5.2.3 逻辑记录子集*

在第 4.1.5 节中，你学习了如何使用逻辑标志向量来对子集数据结构进行筛选。这在数据框中尤其有用，因为你通常会想查看满足特定条件的记录子集。例如，在处理临床药物试验数据时，研究人员可能想查看仅男性参与者的结果，并将其与女性的结果进行比较。或者，研究人员可能想查看对药物反应最积极的个体的特征。

让我们继续处理`mydata`。假设你想查看所有与男性相关的记录。从第 4.3.1 节中，你知道以下这一行可以识别`sex`因子向量中相关的位置：

```
R> mydata$sex=="M"
[1]  TRUE FALSE FALSE  TRUE  TRUE  TRUE
```

这标记了男性记录。你可以结合第 5.2.1 节中看到的类似矩阵的语法来获取仅限男性的子集。

```
R> mydata[mydata$sex=="M",]
  person age sex funny age.mon
1  Peter  42   M  High     504
4  Chris  14   M   Med     168
5 Stewie   1   M  High      12
6  Brian   7   M   Med      84
```

这将返回所有变量的数据，但仅限于男性参与者。你可以使用相同的行为来选择哪些变量返回在子集中。例如，由于你知道你只选择男性，你可以使用负数的列索引来从结果中省略`sex`。

```
R> mydata[mydata$sex=="M",-3]
  person age funny age.mon
1  Peter  42  High     504
4  Chris  14   Med     168
5 Stewie   1  High      12
6  Brian   7   Med      84
```

如果你没有列号，或者你想对返回的列有更多控制，可以改为使用一个包含变量名的字符向量。

```
R> mydata[mydata$sex=="M",c("person","age","funny","age.mon")]
  person age funny age.mon
1  Peter  42  High     504
4  Chris  14   Med     168
5 Stewie   1  High      12
6  Brian   7   Med      84
```

你用于子集数据框的逻辑条件可以简单或复杂，取决于需要。你放入方括号中的逻辑标志向量必须与数据框中的记录数相匹配。让我们从`mydata`中提取所有年龄大于 10 岁或有很高幽默感的个体的完整记录。

```
R> mydata[mydata$age>10|mydata$funny=="High",]
  person age sex funny age.mon
1  Peter  42   M  High     504
2   Lois  40   F  High     480
3    Meg  17   F   Low     204
4  Chris  14   M   Med     168
5 Stewie   1   M  High      12
```

有时候，要求一个子集时可能不会返回任何记录。在这种情况下，R 会返回一个行数为零的数据框，如下所示：

```
R> mydata[mydata$age>45,]
[1] person  age     sex     funny   age.mon
<0 rows> (or 0-length row.names)
```

在这个例子中，由于没有个体年龄超过 45 岁，`mydata`没有返回任何记录。要检查子集是否包含任何记录，你可以对结果使用`nrow`，如果其结果为零，则表示没有记录满足指定的条件。

**练习 5.2**

1.  在你的 R 工作空间中创建并存储这个数据框作为`dframe`：

    | `person` | `sex` | `funny` |
    | --- | --- | --- |
    | `Stan` | `M` | `High` |
    | `Francine` | `F` | `Med` |
    | `Steve` | `M` | `Low` |
    | `Roger` | `M` | `High` |
    | `Hayley` | `F` | `Med` |
    | `Klaus` | `M` | `Med` |

    变量`person`、`sex`和`funny`应与第 5.2 节中研究的`mydata`对象的变量本质上相同。也就是说，`person`应是字符向量，`sex`应是一个具有`F`和`M`级别的因子，`funny`应是一个具有`Low`、`Med`和`High`级别的因子。

1.  Stan 和 Francine 分别 41 岁，Steve15 岁，Hayley21 岁，Klaus60 岁，Roger 非常老—1600 岁。将这些数据作为新的数值列变量`age`添加到`dframe`中。

1.  利用你关于按列索引位置重新排序列变量的知识来覆盖`dframe`，使其与`mydata`保持一致。也就是说，第一列应为`person`，第二列为`age`，第三列为`sex`，第四列为`funny`。

1.  将注意力集中到在第 5.2.2 节中包含`age.mon`变量后留下的`mydata`上。通过删除`age.mon`列，创建一个名为`mydata2`的新版本。

1.  现在，将`mydata2`与`dframe`合并，并将结果对象命名为`mydataframe`。

1.  编写一行代码，从`mydataframe`中提取仅限女性且幽默感水平为`Med`或`High`的记录的姓名和年龄。

1.  使用你在 R 中处理字符字符串的知识，从`mydataframe`中提取所有名字以*S*开头的人的记录。提示：回忆一下第 4.2.4 节中的`substr`（注意，`substr`可以应用于多个字符字符串的向量）。

##### 本章重要代码

| **函数/操作符** | **简要描述** | **首次出现** |
| --- | --- | --- |
| `list` | 创建一个列表 | 第 5.1.1 节，第 89 页 |
| `[[ ]]` | 无名成员引用 | 第 5.1.1 节，第 90 页 |
| `[ ]` | 列表切片（多个成员） | 第 5.1.1 节，第 91 页 |
| `$` | 获取命名成员/变量 | 第 5.1.2 节，第 92 页 |
| `data.frame` | 创建一个数据框 | 第 5.2.1 节，第 96 页 |
| `[ , ]` | 提取数据框的行/列 | 第 5.2.1 节，第 96 页 |
