## 第十章：10

**条件与循环**

![image](img/common-01.jpg)

要用 R 编写更复杂的程序，你需要控制代码的执行流程和顺序。实现这一点的一种基本方法是使某些代码段的执行依赖于一个*条件*。另一种基本控制机制是*循环*，它会将一段代码重复执行指定的次数。在本章中，我们将使用 `if`-`else` 语句、`for` 和 `while` 循环以及其他控制结构，来探讨这些核心编程技巧。

### 10.1 if 语句

`if` 语句是控制在特定代码块中到底执行哪些操作的关键。`if` 语句只有在某个条件为真时，才会执行代码块中的内容。这些构造使得程序可以根据条件是否为 `TRUE` 或 `FALSE` 做出不同的反应。

#### *10.1.1 独立语句*

让我们从独立的 `if` 语句开始，它看起来大概是这样的：

```
if(condition){
    do any code here
}
```

`condition` 放在 `if` 关键字后面的括号内。这个条件必须是一个表达式，它返回一个单一的逻辑值（`TRUE` 或 `FALSE`）。如果条件为 `TRUE`，大括号 `{}` 中的代码将会被执行。如果条件不成立，大括号中的代码将被跳过，R 将什么都不做（或继续执行在闭合大括号后面的代码）。

这是一个简单的示例。在控制台中，存储以下内容：

```
R> a <- 3
R> mynumber <- 4
```

现在，在 R 编辑器中，写下以下代码块：

```
if(a<=mynumber){
    a <- a²
}
```

当执行这段代码时，`a` 的值会是多少？这取决于定义 `if` 语句的条件，以及大括号内实际指定的内容。在这个例子中，当条件 `a<=mynumber` 被评估时，结果是 `TRUE`，因为 3 确实小于或等于 4。这意味着大括号内的代码会被执行，`a` 被设为 `a²`，即 9。

现在，高亮显示编辑器中的整个代码块，并将其发送到控制台进行评估。记住，你可以通过几种方式做到这一点：

• 直接将选中的文本从编辑器复制并粘贴到控制台中。

• 在菜单中，选择 **编辑** → **运行行或选择**（Windows）或选择 **编辑** → **执行**（OS X）。

• 使用快捷键，例如在 Windows 中按 CTRL-R，或在 Mac 上按 -RETURN。

一旦你在控制台中执行代码，你将看到类似下面的结果：

```
R> if(a<=mynumber){
+   a <- a²
+ }
```

然后，查看对象 `a`，如图所示：

```
R> a
[1] 9
```

接下来，假设你立刻再次执行相同的 `if` 语句。`a` 会再次被平方，变为 81 吗？不会！因为 `a` 现在是 9，而 `mynumber` 依然是 4，条件 `a<=mynumber` 将是 `FALSE`，因此大括号内的代码不会被执行；`a` 将保持为 9。

注意，在你将`if`语句发送到控制台后，每一行的前面都会有一个`+`。这些`+`符号并不表示任何形式的算术加法；相反，它们表示 R 在开始执行之前，期望更多的输入。例如，当左花括号被打开时，R 不会开始执行，直到该部分以右花括号关闭。为了避免重复，今后的示例中，我不会展示从编辑器发送到控制台的这部分代码的重复。

**注意**

*你可以通过将不同的字符字符串分配给 R 的`options`命令中的`continue`组件来改变`+`符号，就像在第 1.2.1 节中重置提示符一样。*

`if`语句提供了极大的灵活性——你可以在花括号区域内放置任何类型的代码，包括更多的`if`语句（参见即将讨论的嵌套部分，见第 10.1.4 节），这样可以使你的程序做出一系列的决策。

为了说明一个更复杂的`if`语句，考虑以下两个新对象：

```
R> myvec <- c(2.73,5.40,2.15,5.29,1.36,2.16,1.41,6.97,7.99,9.52)
R> myvec
 [1] 2.73 5.40 2.15 5.29 1.36 2.16 1.41 6.97 7.99 9.52
R> mymat <- matrix(c(2,0,1,2,3,0,3,0,1,1),5,2)
R> mymat
     [,1] [,2]
[1,]    2    0
[2,]    0    3
[3,]    1    0
[4,]    2    1
[5,]    3    1
```

在这里使用这两个对象的代码块：

```
if(any((myvec-1)>9)||matrix(myvec,2,5)[2,1]<=6){
    cat("Condition satisfied --\n")
    new.myvec <- myvec
    new.myvec[seq(1,9,2)] <- NA
    mylist <- list(aa=new.myvec,bb=mymat+0.5)
    cat("-- a list with",length(mylist),"members now exists.")
}
```

将其发送到控制台，会产生以下输出：

```
Condition satisfied --
-- a list with 2 members now exists.
```

确实，已经创建了一个名为`mylist`的对象，你可以检查它。

```
R> mylist
$aa
 [1]   NA 5.40   NA 5.29   NA 2.16   NA 6.97   NA 9.52

$bb
     [,1] [,2]
[1,]  2.5  0.5
[2,]  0.5  3.5
[3,]  1.5  0.5
[4,]  2.5  1.5
[5,]  3.5  1.5
```

在这个例子中，条件由两部分组成，通过使用`||`的 OR 语句连接，产生一个单一的逻辑结果。我们来逐步分析它。

• 条件的第一部分查看`myvec`，从每个元素中减去`1`，并检查结果是否有任何值大于 9。如果单独运行这一部分，结果是`FALSE`。

R> myvec-1

[1] 1.73 4.40 1.15 4.29 0.36 1.16 0.41 5.97 6.99 8.52

R> (myvec-1)>9

[1] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE

R> any((myvec-1)>9)

[1] FALSE

• 条件的第二部分在调用`matrix`时使用位置匹配，构造了一个由原始`myvec`的条目填充的两行五列的矩阵。然后，检查该结果的第一列第二行的数字，看看它是否小于或等于 6，结果是符合的。

R> matrix(myvec,2,5)

[,1] [,2] [,3] [,4] [,5]

[1,] 2.73 2.15 1.36 1.41 7.99

[2,] 5.40 5.29 2.16 6.97 9.52

R> matrix(myvec,2,5)[2,1]

[1] 5.4

R> matrix(myvec,2,5)[2,1]<=6

[1] TRUE

这意味着`if`语句检查的整体条件将是`FALSE||TRUE`，其结果为`TRUE`。

```
R> any((myvec-1)>9)||matrix(myvec,2,5)[2,1]<=6
[1] TRUE
```

结果是，花括号内的代码被访问并执行。首先，它打印出`"Condition satisfied"`字符串，并将`myvec`复制到`new.myvec`。接着，使用`seq`访问`new.myvec`中的奇数索引，并将其值覆盖为`NA`。然后，它创建了`mylist`，在这个列表中，`new.myvec`被存储在一个名为`aa`的成员中，接着将原始的`mymat`的所有元素增加 0.5，并将结果存储在`bb`中。最后，打印出生成的列表的长度。

请注意，`if`语句不必完全按照我在这里使用的样式。有些程序员，例如，喜欢在条件后面将左大括号放在新的一行，或者有些人可能喜欢不同的缩进方式。

#### *10.1.2 else 语句*

`if`语句只有在定义的条件为`TRUE`时才会执行一段代码。如果你希望在条件为`FALSE`时发生不同的事情，你可以添加一个`else`声明。这里是一个伪代码示例：

```
if(condition){
    do any code in here if condition is TRUE
} else {
    do any code in here if condition is FALSE
}
```

你设置条件，然后在第一组大括号中放置当条件为`TRUE`时要执行的代码。在此之后，你声明`else`，后面跟着一个新的大括号，你可以在其中放置当条件为`FALSE`时要执行的代码。

让我们回到第 10.1.1 节中的第一个例子，再次将这些值存储在控制台提示符下。

```
R> a <- 3
R> mynumber <- 4
```

在编辑器中，创建一个新的版本的早期`if`语句。

```
if(a<=mynumber){
    cat("Condition was",a<=mynumber)
    a <- a²
} else {
    cat("Condition was",a<=mynumber)
    a <- a-3.5
}
a
```

在这里，如果条件`a<=mynumber`为`TRUE`，你再次将`a`平方；但如果为`FALSE`，则将`a`覆盖为自身减去 3.5 的结果。你还会打印文本到控制台，说明条件是否满足。在将`a`和`mynumber`重置为它们的原始值后，`if`循环的第一次运行将`a`计算为 9，就像之前一样，并输出以下内容：

```
Condition was TRUE
R> a
[1] 9
```

现在，立即高亮并再次执行整个语句。这一次，`a<=mynumber`将计算为`FALSE`并执行`else`之后的代码。

```
Condition was FALSE
R> a
[1] 5.5
```

#### *10.1.3 使用 ifelse 进行逐元素检查*

`if`语句只能检查单一的逻辑值。如果你传入一个逻辑向量作为条件，例如，`if`语句将只检查（并基于）第一个元素。它会发出警告，正如下面的虚拟示例所示：

```
R> if(c(FALSE,TRUE,FALSE,TRUE,TRUE)){}
Warning message:
In if (c(FALSE, TRUE, FALSE, TRUE, TRUE)) { :
  the condition has length > 1 and only the first element will be used
```

然而，有一个可用的快捷函数`ifelse`，它可以在相对简单的情况下执行这种向量化检查。为了演示它是如何工作的，考虑以下定义的对象`x`和`y`：

```
R> x <- 5
R> y <- -5:5
R> y
 [1] -5 -4 -3 -2 -1 0 1 2 3 4 5
```

现在，假设你想得到`x/y`的结果，但将任何`Inf`（即`x`除以零的任何实例）替换为`NA`。换句话说，对于`y`中的每个元素，你想检查`y`是否为零。如果是，那么你希望代码输出`NA`，如果不是，它应该输出`x/y`的结果。

正如你刚刚看到的，简单的`if`语句在这里不起作用。由于它只接受单一的逻辑值，它不能遍历`y==0`生成的整个逻辑向量。

```
R> y==0
 [1] FALSE FALSE FALSE FALSE FALSE  TRUE FALSE FALSE FALSE FALSE FALSE
```

相反，你可以在这种情况下使用逐元素的`ifelse`函数。

```
R> result <- ifelse(test=y==0,yes=NA,no=x/y)
R> result
 [1] -1.000000 -1.250000 -1.666667 -2.500000 -5.000000 NA 5.000000 2.500000
 [9]  1.666667  1.250000  1.000000
```

使用精确匹配，这个命令在一行中创建了期望的 `result` 向量。必须指定三个参数：`test` 接受一个逻辑值数据结构，`yes` 提供满足条件时返回的元素，`no` 提供条件为 `FALSE` 时返回的元素。正如函数文档中所指出的（你可以通过 `?ifelse` 访问它），返回的结构将具有与 `test` 相同的长度和属性。

**练习 10.1**

1.  创建以下两个向量：

    ```
    vec1 <- c(2,1,1,3,2,1,0)
    vec2 <- c(3,8,2,2,0,0,0)
    ```

    在不执行它们的情况下，确定以下哪个 `if` 语句会导致字符串被打印到控制台。然后在 R 中确认你的答案。

    1.  ```
        if((vec1[1]+vec2[2])==10){ cat("Print me!") }
        ```

    1.  ```
        if(vec1[1]>=2&&vec2[1]>=2){ cat("Print me!") }
        ```

    1.  ```
        if(all((vec2-vec1)[c(2,6)]<7)){ cat("Print me!") }
        ```

    1.  ```
        if(!is.na(vec2[3])){ cat("Print me!") }
        ```

1.  使用（a）中的 `vec1` 和 `vec2`，编写并执行一行代码，只有当它们的和大于 3 时，才将两个向量的对应元素相乘。否则，代码应简单地将两个元素相加。

1.  在编辑器中编写 R 代码，该代码接受一个方形字符矩阵，并检查对角线上的任何字符字符串（从左上角到右下角）是否以字母 *g*（无论是小写还是大写）开头。如果满足条件，这些特定条目应该被字符串 `"HERE"` 覆盖。否则，整个矩阵应该被同样维度的单位矩阵替换。然后，在以下矩阵上尝试你的代码，并每次检查结果：

    1.  ```
        mymat <- matrix(as.character(1:16),4,4)
        ```

    1.  ```
        mymat <- matrix(c("DANDELION","Hyacinthus","Gerbera",
                          "MARIGOLD","geranium","ligularia",
                          "Pachysandra","SNAPDRAGON","GLADIOLUS"),3,3)
        ```

    1.  ```
        mymat <- matrix(c("GREAT","exercises","right","here"),2,2,
                          byrow=T)
        ```

    提示：这需要一些思考——你会发现 第 3.2.1 节 中的 `diag` 函数和 第 4.2.4 节 中的 `substr` 函数会很有用。

#### *10.1.4 嵌套和堆叠语句*

一个 `if` 语句可以被放置在另一个 `if` 语句的结果中。通过 *嵌套* 或 *堆叠* 多个语句，你可以在执行过程中检查多个条件，从而编织出复杂的决策路径。

在编辑器中再次修改 `mynumber` 示例，如下所示：

```
if(a<=mynumber){
    cat("First condition was TRUE\n")
    a <- a²
    if(mynumber>3){
        cat("Second condition was TRUE")
        b <- seq(1,a,length=mynumber)
    } else {
        cat("Second condition was FALSE")
        b <- a*mynumber
    }
} else {
    cat("First condition was FALSE\n")
    a <- a-3.5
    if(mynumber>=4){
        cat("Second condition was TRUE")
        b <- a^(3-mynumber)
    } else {
        cat("Second condition was FALSE")
        b <- rep(a+mynumber,times=3)
    }
}
a
b
```

这里你会看到与之前相同的初始决策。如果 `a` 小于或等于 `mynumber`，则将其平方；否则，将其减去 3.5。但是现在每个大括号区域内有另一个 `if` 语句。如果第一个条件满足且 `a` 被平方，则继续检查 `mynumber` 是否大于 3。如果是 `TRUE`，则将 `b` 赋值为 `seq(1,a,length=mynumber)`；如果是 `FALSE`，则将 `b` 赋值为 `a*mynumber`。

如果第一个条件失败并且你从 `a` 中减去 3.5，然后检查第二个条件，查看 `mynumber` 是否大于或等于 4。如果是，那么 `b` 变为 `a^(3-mynumber)`。如果不是，`b` 变为 `rep(a+mynumber,times=3)`。请注意，我已经缩进了每个大括号内的代码，以便更容易看到哪些行与每个可能的决策相关。

现在，在控制台中直接或通过编辑器重置 `a <- 3` 和 `mynumber <- 4`。当你运行 `mynumber` 示例代码时，你将得到以下输出：

```
First condition was TRUE
Second condition was TRUE
R> a
[1] 9
R> b
[1] 1.000000 3.666667 6.333333 9.000000
```

结果显示了究竟是哪个代码被调用——第一个条件和第二个条件都为`TRUE`。在再次运行相同代码之前，首先设置

```
R> a <- 6
R> mynumber <- 4
```

你将看到这个输出：

```
First condition was FALSE
Second condition was TRUE
R> a
[1] 2.5
R> b
[1] 0.4
```

这次第一个条件失败了，但在`else`语句内检查的第二个条件是`TRUE`。

另外，你也可以通过依次*堆叠*`if`语句并在每个条件中使用逻辑表达式的组合来实现相同的效果。在下面的示例中，你检查了相同的四种情况，但这次你通过将新的`if`声明直接跟在`else`声明后面来堆叠`if`语句：

```
if(a<=mynumber && mynumber>3){
    cat("Same as 'first condition TRUE and second TRUE'")
    a <- a²
    b <- seq(1,a,length=mynumber)
} else if(a<=mynumber && mynumber<=3){
    cat("Same as 'first condition TRUE and second FALSE'")
    a <- a²
    b <- a*mynumber
} else if(mynumber>=4){
    cat("Same as 'first condition FALSE and second TRUE'")
    a <- a-3.5
    b <- a^(3-mynumber)
} else {
    cat("Same as 'first condition FALSE and second FALSE'")
    a <- a-3.5
    b <- rep(a+mynumber,times=3)
}
a
b
```

就像之前一样，四个括起来的区域中只有一个最终会被执行。与嵌套版本相比，前两个括起来的区域对应于最初的第一个条件（`a<=mynumber`）被满足，但这次你使用`&&`同时检查两个表达式。如果这两个情况都不满足，那么第一个条件就是假，因此在第三个语句中，你只需要检查`mynumber>=4`。对于最终的`else`语句，你无需检查任何条件，因为该语句仅在所有之前的条件未满足时才会执行。

如果你再次将`a`和`mynumber`分别重置为 3 和 4，并执行之前展示的堆叠语句，你将得到以下结果：

```
Same as 'first condition TRUE and second TRUE'
R> a
[1] 9
R> b
[1] 1.000000 3.666667 6.333333 9.000000
```

这将产生与之前相同的`a`和`b`的值。如果你使用第二组初始值（`a`为 6，`mynumber`为 4）再次执行代码，你将得到以下结果：

```
Same as 'first condition FALSE and second TRUE'
R> a
[1] 2.5
R> b
[1] 0.4
```

这再次与使用嵌套版本代码的结果相匹配。

#### *10.1.5 switch 函数*

假设你需要根据一个对象的值来选择运行的代码（这是一个常见场景）。一种选择是使用一系列的`if`语句，通过将对象与各种可能的值进行比较，为每个条件生成一个逻辑值。下面是一个示例：

```
if(mystring=="Homer"){
    foo <- 12
} else if(mystring=="Marge"){
    foo <- 34
} else if(mystring=="Bart"){
    foo <- 56
} else if(mystring=="Lisa"){
    foo <- 78
} else if(mystring=="Maggie"){
    foo <- 90
} else {
    foo <- NA
}
```

这段代码的目标是简单地为对象`foo`赋一个数值，其中具体的数字取决于`mystring`的值。`mystring`对象可以有五种可能的值，或者如果`mystring`与这些值都不匹配，则`foo`被赋值为`NA`。

这段代码按原样运行得很好。例如，设置

```
R> mystring <- "Lisa"
```

并执行代码块，你会看到这个结果：

```
R> foo
[1] 78
```

设置以下

```
R> mystring <- "Peter"
```

并再次执行代码块，你会看到这个结果：

```
R> foo
[1] NA
```

然而，使用`if`-`else`语句来设置这种基础操作显得相当繁琐。R 可以通过`switch`函数以更紧凑的形式处理这种多选决策。例如，你可以将堆叠的`if`语句改写为一个更简洁的`switch`语句，如下所示：

```
R> mystring <- "Lisa"
R> foo <- switch(EXPR=mystring,Homer=12,Marge=34,Bart=56,Lisa=78,Maggie=90,NA)
R> foo
[1] 78
```

以及

```
R> mystring <- "Peter"
R> foo <- switch(EXPR=mystring,Homer=12,Marge=34,Bart=56,Lisa=78,Maggie=90,NA)
R> foo
[1] NA
```

第一个参数 `EXPR` 是感兴趣的对象，可以是数值型或字符型字符串。其余的参数提供基于 `EXPR` 值进行的值或操作。如果 `EXPR` 是字符串，这些参数标签必须*完全*匹配 `EXPR` 的可能结果。在这里，如果 `mystring` 是 `"Homer"`，`switch` 语句返回 12；如果 `mystring` 是 `"Marge"`，返回 34，以此类推。最后一个未标记的值 `NA` 表示如果 `mystring` 不匹配任何前面的项时的结果。

整数版的 `switch` 的工作方式稍有不同。它不是使用标签，而是通过位置匹配来确定结果。考虑以下示例：

```
R> mynum <- 3
R> foo <- switch(mynum,12,34,56,78,NA)
R> foo
[1] 56
```

在这里，你提供一个整数 `mynum` 作为第一个参数，并且它与 `EXPR` 按位置匹配。示例代码随后展示了五个未标记的参数：`12` 到 `NA`。`switch` 函数简单地返回由 `mynum` 请求的特定位置的值。由于 `mynum` 为 3，语句将 56 赋值给 `foo`。如果 `mynum` 是 1、2、4 或 5，`foo` 将分别被赋值为 12、34、78 或 `NA`。任何其他值的 `mynum`（小于 1 或大于 5）将返回 `NULL`。

```
R> mynum <- 0
R> foo <- switch(mynum,12,34,56,78,NA)
R> foo
NULL
```

在这些情况下，`switch` 函数的行为与一组堆叠的 `if` 语句相同，因此它可以作为一个方便的快捷方式。然而，如果你需要同时检查多个条件，或者需要根据该决策执行一组更复杂的操作，你将需要使用显式的 `if` 和 `else` 控制结构。

**练习 10.2**

1.  编写一组显式堆叠的 `if` 语句，执行与前面展示的整数版 `switch` 函数相同的操作。使用 `mynum <- 3` 和 `mynum <- 0` 进行测试，正如文中所示。

1.  假设你负责计算某种药物在一系列假设的科学实验中的精确剂量。这些剂量依赖于一些预定的“剂量阈值”（`lowdose`、`meddose` 和 `highdose`），以及一个名为 `doselevel` 的预定剂量水平因子向量。请查看以下项目（i–iv）以了解这些对象的预期形式。然后编写一组嵌套的 `if` 语句，按照以下规则生成一个新的数值向量 `dosage`：

    – 首先，*检查* `doselevel` 中是否有任何 `"High"` 的实例，如果有，执行以下操作：

    *检查* `lowdose` 是否大于或等于 10。如果是，覆盖 `lowdose` 为 10；*否则*，将 `lowdose` 替换为它本身除以 2。

    *检查* `meddose` 是否大于或等于 26。如果是，将 `meddose` 覆盖为 26。

    *检查* `highdose` 是否小于 60。如果是，覆盖 `highdose` 为 60；*否则*，将 `highdose` 替换为它本身乘以 1.5。

    *创建一个名为 `dosage` 的向量，其值为 `lowdose` 重复（`rep`），以匹配 `doselevel` 的 `length`。*

    * 将`dosage`中对应于`doselevel`中`"Med"`实例索引位置的元素覆盖为`meddose`。

    * 将`dosage`中对应于`doselevel`中`"High"`实例索引位置的元素覆盖为`highdose`。

    – *否则*（换句话说，如果`doselevel`中没有`"High"`实例），执行以下操作：

    * 创建`doselevel`的新版本，一个仅具有级别`"Low"`和`"Med"`的因子向量，并将这些级别分别标记为`"Small"`和`"Large"`（有关详细信息，请参见第 4.3 节，或查看`?factor`）。

    * 检查`lowdose`是否小于 15，并且`meddose`是否小于 35。如果是，单独将`lowdose`乘以 2，并将`meddose`覆盖为其本身加上`highdose`。

    * 创建一个名为`dosage`的向量，其值为`lowdose`重复（`rep`）至与`doselevel`的`length`匹配。

    * 将`dosage`中对应于`doselevel`中`"Large"`实例索引位置的元素覆盖为`meddose`。

    现在，确认以下内容：

    1.  给定

        ```
        lowdose <- 12.5
        meddose <- 25.3
        highdose <- 58.1
        doselevel <- factor(c("Low","High","High","High","Low","Med",
                              "Med"),levels=c("Low","Med","High"))
        ```

        运行嵌套`if`语句后，`dosage`的结果如下：

        ```
        R> dosage
        [1] 10.0 60.0 60.0 60.0 10.0 25.3 25.3
        ```

    1.  使用与（i）中相同的`lowdose`、`meddose`和`highdose`阈值，给定

        ```
        doselevel <- factor(c("Low","Low","Low","Med","Low","Med",
                              "Med"),levels=c("Low","Med","High"))
        ```

        运行嵌套`if`语句后，`dosage`的结果如下：

        ```
        R> dosage
        [1] 25.0 25.0 25.0 83.4 25.0 83.4 83.4
        ```

        此外，`doselevel`已被如下覆盖：

        ```
        R> doselevel
        [1] Small Small Small Large Small Large Large
        Levels: Small Large
        ```

    1.  给定

        ```
        lowdose <- 9
        meddose <- 49
        highdose <- 61
        doselevel <- factor(c("Low","Med","Med"),
                            levels=c("Low","Med","High"))
        ```

        运行嵌套`if`语句后，`dosage`的结果如下：

        ```
        R> dosage
        [1] 9 49 49
        ```

        此外，`doselevel`已被如下覆盖：

        ```
        R> doselevel
        [1] Small Large Large
        Levels: Small Large
        ```

    1.  使用与（iii）中相同的`lowdose`、`meddose`和`highdose`阈值，以及与（i）中相同的`doselevel`，运行嵌套`if`语句后，`dosage`的结果如下：

        ```
        R> dosage
        [1] 4.5 91.5 91.5 91.5 4.5 26.0 26.0
        ```

1.  假设对象`mynum`始终是介于 0 和 9 之间的单个整数。使用`ifelse`和`switch`来生成一个命令，该命令接受`mynum`并返回与所有可能值 0, 1, ..., 9 对应的字符字符串。例如，传入`3`时应返回`"three"`；传入`0`时应返回`"zero"`。

### 10.2 编码循环

另一种核心编程机制是*循环*，它会重复指定的代码段，通常是通过递增索引或计数器来实现。有两种循环方式：`for`循环会在向量中逐个元素地执行代码；`while`循环则会在某个特定条件评估为`FALSE`时停止。循环行为还可以通过 R 的`apply`函数系列来实现，相关内容讨论见第 10.2.3 节。

#### *10.2.1 for 循环*

R 的`for`循环始终采用以下通用形式：

```
for(loopindex in loopvector){
    do any code in here
}
```

在这里，`loopindex`是一个占位符，代表`loopvector`中的一个元素——它从向量中的第一个元素开始，并在每次循环重复时移动到下一个元素。当`for`循环开始时，它运行大括号区域中的代码，将`loopindex`的任何出现替换为`loopvector`中的第一个元素。当循环达到闭合大括号时，`loopindex`会增加，取`loopvector`中的第二个元素，并重复大括号中的区域。这个过程一直持续到循环到达`loopvector`的最后一个元素，此时大括号代码被执行最后一次，循环退出。

这是一个在编辑器中编写的简单示例：

```
for(myitem in 5:7){
    cat("--BRACED AREA BEGINS--\n")
    cat("the current item is",myitem,"\n")
    cat("--BRACED AREA ENDS--\n\n")
}
```

这个循环打印了`loopindex`（在这里我将其命名为`myitem`）的当前值，它从 5 递增到 7。以下是将结果输出到控制台后的输出：

```
--BRACED AREA BEGINS--
the current item is 5
--BRACED AREA ENDS--

--BRACED AREA BEGINS--
the current item is 6
--BRACED AREA ENDS--

--BRACED AREA BEGINS--
the current item is 7
--BRACED AREA ENDS--
```

你可以使用循环来操作循环外部存在的对象。考虑以下示例：

```
R> counter <- 0
R> for(myitem in 5:7){
+   counter <- counter+1
+   cat("The item in run",counter,"is",myitem,"\n")
+ }
The item in run 1 is 5
The item in run 2 is 6
The item in run 3 is 7
```

在这里，我首先定义了一个对象`counter`，并在工作空间中将其设置为零。然后，在循环内部，`counter`被其自身加 1 所覆盖。每次循环重复时，`counter`增加，并将当前值打印到控制台。

##### 通过索引或值进行循环

请注意，使用`loopindex`直接表示`loopvector`中的元素与使用它表示向量的*索引*之间的区别。以下两个循环使用这两种不同的方法来`print`每个`myvec`中的数字的双倍：

```
R> myvec <- c(0.4,1.1,0.34,0.55)
R> for(i in myvec){
+   print(2*i)
+ }
[1] 0.8
[1] 2.2
[1] 0.68
[1] 1.1
R> for(i in 1:length(myvec)){
+   print(2*myvec[i])
+ }
[1] 0.8
[1] 2.2
[1] 0.68
[1] 1.1
```

第一个循环使用`loopindex` i 直接表示`myvec`中的元素，打印每个元素乘以 2 的值。另一方面，在第二个循环中，你使用`i`表示`1:length(myvec)`中的整数。这些整数构成了`myvec`所有可能的索引位置，你可以使用这些索引来提取`myvec`的元素（再次将每个元素乘以 2 并打印结果）。虽然这种方式稍显冗长，但使用向量索引位置在你如何使用`loopindex`时提供了更多灵活性。当你需要更复杂的`for`循环时，这一点会更加清晰，正如下一个例子所展示的。

假设你想编写一些代码，检查任何列表对象，并收集列表中作为成员存储的任何矩阵对象的信息。请考虑以下列表：

```
R> foo <- list(aa=c(3.4,1),bb=matrix(1:4,2,2),cc=matrix(c(T,T,F,T,F,F),3,2),
               dd="string here",ee=matrix(c("red","green","blue","yellow")))
R> foo
$aa
[1] 3.4 1.0

$bb
     [,1] [,2]
[1,]    1    3
[2,]    2    4

$cc
      [,1]  [,2]
[1,]  TRUE  TRUE
[2,]  TRUE FALSE
[3,] FALSE FALSE

$dd
[1] "string here"

$ee
     [,1]
[1,] "red"
[2,] "green"
[3,] "blue"
[4,] "yellow"
```

在这里，你创建了`foo`，它包含三个不同维度和数据类型的矩阵。你将编写一个`for`循环，遍历像这样的列表的每个成员，并检查该成员是否为矩阵。如果是，循环将获取矩阵的行数、列数以及数据类型。

在编写`for`循环之前，你应当创建一些向量来存储关于列表成员的信息：`name`用于存储列表成员的名称，`is.mat`用于指示每个成员是否是矩阵（值为`"Yes"`或`"No"`），`nc`和`nr`用于存储每个矩阵的行数和列数，`data.type`用于存储每个矩阵的数据类型。

```
R> name <- names(foo)
R> name
[1] "aa" "bb" "cc" "dd" "ee"
R> is.mat <- rep(NA,length(foo))
R> is.mat
[1] NA NA NA NA NA
R> nr <- is.mat
R> nc <- is.mat
R> data.type <- is.mat
```

在这里，你将`foo`的成员名称存储为`name`。同时，设置`is.mat`、`nr`、`nc`和`data.type`，这些都被分配为长度为`length(foo)`的向量，且初始值为`NA`。这些值将在你的`for`循环中根据需要进行更新，接下来你就可以编写循环了。请在编辑器中输入以下代码：

```
for(i in 1:length(foo)){
    member <- foo[[i]]
    if(is.matrix(member)){
        is.mat[i] <- "Yes"
        nr[i] <- nrow(member)
        nc[i] <- ncol(member)
        data.type[i] <- class(as.vector(member))
    } else {
        is.mat[i] <- "No"
    }
}
bar <- data.frame(name,is.mat,nr,nc,data.type,stringsAsFactors=FALSE)
```

最初，设置`loopindex`变量`i`，使其能够通过`foo`的索引位置递增（即`1:length(foo)`的序列）。在大括号中的代码首先将`foo`中位置为`i`的成员写入一个对象`member`。接下来，你可以使用`is.matrix`检查该成员是否为矩阵（参见第 6.2.3 节）。如果为`TRUE`，执行以下操作：将`is.mat`向量的第`i`位置设置为`"Yes"`；将`nr`和`nc`的第`i`元素分别设置为`member`的行数和列数；将`data.type`的第`i`元素设置为`class(as.vector(member))`的结果。该命令首先通过`as.vector`将矩阵强制转换为向量，然后使用`class`函数（详见第 6.2.2 节）来查找元素的数据类型。

如果`member`不是矩阵且`if`条件失败，则`is.mat`中相应的条目将被设置为`"No"`，其他向量中的条目保持不变（因此它们仍然是`NA`）。

循环执行完毕后，从向量中创建数据框`bar`（注意使用`stringsAsFactors=FALSE`，以防止`bar`中的字符型向量被自动转换为因子；参见第 5.2.1 节）。执行代码后，`bar`的样式如下：

```
R> bar
  name is.mat nr nc data.type
1   aa     No NA NA      <NA>
2   bb    Yes  2  2   integer
3   cc    Yes  3  2   logical
4   dd     No NA NA      <NA>
5   ee    Yes  4  1 character
```

如你所见，这与列表`foo`中矩阵的性质相匹配。

##### 嵌套 for 循环

你还可以像`if`语句一样嵌套`for`循环。当一个`for`循环嵌套在另一个`for`循环中时，内层循环会在外层循环的`loopindex`递增之前执行完整，接着内层循环会再次执行一遍。请在你的 R 控制台中创建以下对象：

```
R> loopvec1 <- 5:7
R> loopvec1
[1] 5 6 7
R> loopvec2 <- 9:6
R> loopvec2
[1] 9 8 7 6
R> foo <- matrix(NA,length(loopvec1),length(loopvec2))
R> foo
     [,1] [,2] [,3] [,4]
[1,]   NA   NA   NA   NA
[2,]   NA   NA   NA   NA
[3,]   NA   NA   NA   NA
```

以下嵌套循环将`foo`填充为将`loopvec1`中的每个整数与`loopvec2`中的每个整数相乘的结果：

```
R> for(i in 1:length(loopvec1)){
+   for(j in 1:length(loopvec2)){
+       foo[i,j] <- loopvec1[i]*loopvec2[j]
+   }
+ }
R> foo
     [,1] [,2] [,3] [,4]
[1,]   45   40   35   30
[2,]   54   48   42   36
[3,]   63   56   49   42
```

请注意，嵌套循环需要为每个`for`循环使用唯一的`loopindex`。在这种情况下，外部循环的`loopindex`是`i`，内部循环的`loopindex`是`j`。当代码执行时，`i`首先被赋值为`1`，然后开始内部循环，此时`j`也被赋值为`1`。内部循环中唯一的命令是将`loopvec1`的第`i`个元素与`loopvec2`的第`j`个元素相乘，并将结果赋值给`foo`矩阵的第`i`行、第`j`列。内部循环会重复执行，直到`j`达到`length(loopvec2)`，填满`foo`的第一行；然后，`i`递增，重新启动内部循环。整个过程将在`i`达到`length(loopvec1)`并填满矩阵后完成。

内部的`loopvector`甚至可以被定义为与外部循环的当前`loopindex`值相匹配。以下是使用之前的`loopvec1`和`loopvec2`的一个示例：

```
R> foo <- matrix(NA,length(loopvec1),length(loopvec2))
R> foo
     [,1] [,2] [,3] [,4]
[1,]   NA   NA   NA   NA
[2,]   NA   NA   NA   NA
[3,]   NA   NA   NA   NA
R> for(i in 1:length(loopvec1)){
+   for(j in 1:i){
+       foo[i,j] <- loopvec1[i]+loopvec2[j]
+   }
+ }
R> foo
     [,1] [,2] [,3] [,4]
[1,]   14   NA   NA   NA
[2,]   15   14   NA   NA
[3,]   16   15   14   NA
```

在这里，`foo`矩阵的第`i`行、第`j`列元素被填充为`loopvec1[i]`与`loopvec2[j]`的和。然而，内部循环的`j`值现在是根据`i`的值来决定的。例如，当`i`为`1`时，内部的`loopvector`是`1:1`，因此内部循环只执行一次，然后返回外部循环。当`i`为`2`时，内部的`loopvector`是`1:2`，依此类推。这使得`foo`的每一行仅部分被填充。以这种方式编写循环时需要格外小心。例如，在这里，`j`的值依赖于`loopvec1`的长度，因此如果`length(loopvec1)`大于`length(loopvec2)`，就会发生错误。

任意数量的`for`循环可以嵌套使用，但如果嵌套循环使用不当，计算开销可能会成为问题。循环一般会增加一些计算成本，因此在 R 中编写更高效的代码时，你应该始终问自己：“我能否以面向向量的方式来做这件事？”只有当单独的操作不可能或无法轻松批量实现时，才应考虑探索迭代的、循环的方法。你可以在 Ligges 和 Fox 的《R Help Desk》文章中找到一些关于 R 循环和相关最佳实践编程的有价值评论（2008）。

**习题 10.3**

1.  为了提高编码效率，请重新编写本节中的嵌套循环示例，该示例将矩阵`foo`填充为`loopvec1`和`loopvec2`元素的倍数，改为仅使用一个`for`循环。

1.  在第 10.1.5 节中，你使用了命令

    ```
    switch(EXPR=mystring,Homer=12,Marge=34,Bart=56,Lisa=78,Maggie=90,
           NA)
    ```

    该命令根据提供的单字符字符串值返回一个数字。如果`mystring`是字符向量，则此行代码将无法正常工作。编写一些代码，接受一个字符向量并返回一个适当的数字值向量。用以下向量进行测试：

    ```
    c("Peter","Homer","Lois","Stewie","Maggie","Bart")
    ```

1.  假设你有一个名为`mylist`的列表，它可以包含其他列表作为成员，但假设这些“成员列表”本身不能包含列表。编写嵌套循环，能够搜索任何以这种方式定义的`mylist`，并统计其中有多少个矩阵。提示：只需在开始循环之前设置一个计数器，每次找到矩阵时递增，无论它是`mylist`的直接成员，还是`mylist`的成员列表中的成员。

    然后确认以下内容：

    1.  如果你有以下内容，答案是 4：

        ```
        mylist <- list(aa=c(3.4,1),bb=matrix(1:4,2,2),
                       cc=matrix(c(T,T,F,T,F,F),3,2),dd="string here",
                       ee=list(c("hello","you"),matrix(c("hello",
                                                         "there"))),
                       ff=matrix(c("red","green","blue","yellow")))
        ```

    1.  如果你有以下内容，答案是 0：

        ```
        mylist <- list("tricked you",as.vector(matrix(1:6,3,2)))
        ```

    1.  如果你有以下内容，答案是 2：

        ```
        mylist <- list(list(1,2,3),list(c(3,2),2),
                       list(c(1,2),matrix(c(1,2))),
                       rbind(1:10,100:91))
        ```

#### *10.2.2 while 循环*

要使用`for`循环，你必须知道或能够轻松计算循环应重复的次数。如果你不知道需要运行多少次所需的操作，可以使用`while`循环。`while`循环在指定的条件返回`TRUE`时运行并重复，并且具有以下通用形式：

```
while(loopcondition){
    do any code in here
}
```

`while`循环使用单一的逻辑值`loopcondition`来控制循环重复的次数。在执行时，`loopcondition`会被评估。如果条件为`TRUE`，则代码块会按常规逐行执行，直到完成，此时会再次检查`loopcondition`。循环仅在条件评估为`FALSE`时终止，而且是立即终止——代码块*不会*再执行最后一次。

这意味着代码块中执行的操作必须以某种方式导致循环退出，要么通过以某种方式影响`loopcondition`，要么通过声明`break`，稍后你会看到。如果没有，循环将会永远重复下去，形成一个*无限循环*，这将冻结控制台（并且，根据代码块中的操作，R 可能会因为内存限制崩溃）。如果发生这种情况，你可以通过点击顶部菜单的停止按钮或按 ESC 键在 R 用户界面中终止循环。

作为一个简单的`while`循环示例，考虑以下代码：

```
myval <- 5
while(myval<10){
    myval <- myval+1
    cat("\n'myval' is now",myval,"\n")
    cat("'mycondition' is now",myval<10,"\n")
}
```

在这里，你将一个新对象`myval`设置为`5`。然后你开始一个`while`循环，条件为`myval<10`。由于一开始条件为`TRUE`，你进入了代码块。在循环内部，你将`myval`加 1，打印当前值，并打印条件`myval<5`的逻辑值。循环会继续，直到下次评估时条件`myval<10`为`FALSE`。执行代码块，你会看到以下结果：

```
'myval' is now 6
'mycondition' is now TRUE

'myval' is now 7
'mycondition' is now TRUE

'myval' is now 8
'mycondition' is now TRUE

'myval' is now 9
'mycondition' is now TRUE

'myval' is now 10
'mycondition' is now FALSE
```

正如预期的那样，循环会重复直到`myval`被设置为`10`，此时`myval<10`返回`FALSE`，导致循环退出，因为初始条件不再是`TRUE`。

在更复杂的设置中，通常将 `loopcondition` 设置为一个独立的对象是非常有用的，这样你可以在大括号内根据需要修改它。在下一个示例中，你将使用 `while` 循环迭代一个整数向量，并创建一个单位矩阵（参见 第 3.3.2 节），其维度与当前整数匹配。这个循环应该在遇到向量中的一个大于 5 的数字时停止，或者当它到达整数向量的末尾时停止。

在编辑器中，定义一些初始对象，然后是循环本身。

```
mylist <- list()
counter <- 1
mynumbers <- c(4,5,1,2,6,2,4,6,6,2)
mycondition <- mynumbers[counter]<=5
while(mycondition){
    mylist[[counter]] <- diag(mynumbers[counter])
    counter <- counter+1
    if(counter<=length(mynumbers)){
        mycondition <- mynumbers[counter]<=5
    } else {
        mycondition <- FALSE
    }
}
```

第一个对象 `mylist` 将存储循环创建的所有矩阵。你将使用向量 `mynumbers` 提供矩阵的大小，并使用 `counter` 和 `mycondition` 来控制循环。

`loopcondition`，即 `mycondition`，最初设置为 `TRUE`，因为 `mynumbers` 的第一个元素小于或等于 5。在从 `while` 开始的循环内，第一行使用双重方括号和 `counter` 的值动态创建 `mylist` 中该位置的新条目（你之前在 第 5.1.3 节 中使用命名列表做过类似的操作）。该条目被分配一个单位矩阵，其大小与 `mynumbers` 中相应元素的大小匹配。接着，`counter` 增加，你需要更新 `mycondition`。在这里，你要检查 `mynumbers[counter] <= 5`，但还需要检查是否已经到达整数向量的末尾（否则，试图访问 `mynumbers` 范围外的索引位置会导致错误）。因此，可以使用 `if` 语句首先检查条件 `counter <= length(mynumbers)`。如果条件为 `TRUE`，则将 `mycondition` 设置为 `mynumbers[counter] <= 5` 的结果。如果条件不成立，意味着你已到达 `mynumbers` 的末尾，因此需要通过设置 `mycondition <- FALSE` 来确保循环退出。

使用那些预定义的对象执行循环，它将生成如下所示的 `mylist` 对象：

```
R> mylist
[[1]]
     [,1] [,2] [,3] [,4]
[1,]    1    0    0    0
[2,]    0    1    0    0
[3,]    0    0    1    0
[4,]    0    0    0    1

[[2]]
     [,1] [,2] [,3] [,4] [,5]
[1,]    1    0    0    0    0
[2,]    0    1    0    0    0
[3,]    0    0    1    0    0
[4,]    0    0    0    1    0
[5,]    0    0    0    0    1

[[3]]
     [,1]
[1,]    1
[[4]]
     [,1] [,2]
[1,]    1    0
[2,]    0    1
```

正如预期的那样，你有一个包含四个元素的列表——大小分别为 4 × 4、5 × 5、1 × 1 和 2 × 2 的单位矩阵——与 `mynumbers` 的前四个元素相匹配。当循环执行到 `mynumbers` 的第五个元素（`6`）时停止，因为它大于 5。

**练习 10.4**

1.  基于最近一个将单位矩阵存储在列表中的示例，确定在不执行任何操作的情况下，对于以下每个可能的 `mynumbers` 向量，结果 `mylist` 会是什么样子：

    1.  `mynumbers <- c(2,2,2,2,5,2)`

    1.  `mynumbers <- 2:20`

    1.  `mynumbers <- c(10,1,10,1,2)`

    然后，在 R 中确认你的答案（注意，每次你还需要像文本中所示的那样重置 `mylist`、`counter` 和 `mycondition` 的初始值）。

1.  对于这个问题，我将介绍 *阶乘* 运算符。一个非负整数 *x* 的阶乘，表示为 *x*!，是 *x* 乘以所有小于 *x* 的整数的积，一直到 1。形式上，它可以这样表示：

    “*x*的阶乘” = *x*! = *x* × (*x* − 1) × (*x* − 2) × ... × 1

    请注意，*零的阶乘*是一个特殊情况，总是等于 1。也就是说：

    0! = 1

    例如，要计算 3 的阶乘，你需要做如下计算：

    3 × 2 × 1 = 6

    要计算 7 的阶乘，你需要做如下计算：

    7 × 6 × 5 × 4 × 3 × 2 × 1 = 5040

    写一个`while`循环，通过每次递减`mynum`的值来计算并存储任意非负整数`mynum`的阶乘，直到循环结束。

    使用你的循环，确认以下内容：

    1.  使用`mynum <- 5`时结果为`120`

    1.  使用`mynum <- 12`时结果为`479001600`

    1.  当`mynum <- 0`时，正确返回`1`

1.  考虑以下代码，其中`while`循环内的操作部分已省略：

    ```
    mystring <- "R fever"
    index <- 1
    ecount <- 0
    result <- mystring
    while(ecount<2 && index<=nchar(mystring)){
        # several omitted operations #
    }
    result
    ```

    你的任务是完成大括号中的代码，使其逐个检查`mystring`中的字符，直到达到第二个字母*e*或字符串的末尾，以先到者为准。如果没有第二个*e*，则`result`对象应该是整个字符字符串；如果有第二个*e*，则`result`应该是从头到第二个*e*之前的所有字符。例如，`mystring <- "R fever"`应该返回`result`为`"R fev"`。这必须通过在大括号内执行以下操作来实现：

    1.  使用`substr`（第 4.2.4 节）提取`mystring`中`index`位置的单个字符。

    1.  使用等式检查，判断这个单字符字符串是否为`"e"`或`"E"`。如果是，则将`ecount`加`1`。

    1.  接下来，进行单独检查，查看`ecount`是否等于`2`。如果是，使用`substr`将`result`设置为从`1`到`index-1`（包括`index-1`）之间的字符。

    1.  将`index`增加`1`。

    测试你的代码—确保`mystring <- "R fever"`的前一个`result`。此外，确认以下内容：

    – 使用`mystring <- "beautiful"`会得到`result`为`"beautiful"`

    – 使用`mystring <- "ECCENTRIC"`会得到`result`为`"ECC"`

    – 使用`mystring <- "ElAbOrAte"`会得到`result`为`"ElAbOrAt"`

    – 使用`mystring <- "eeeeek!"`会得到`result`为`"e"`

#### *10.2.3 使用 apply 进行隐式循环*

在某些情况下，特别是对于相对常规的`for`循环（例如对列表中的每个成员执行某个函数），你可以通过使用`apply`函数避免一些与显式循环相关的细节。`apply`函数是最基本的隐式循环形式——它接受一个函数，并将其应用于数组的每个*维度*。

对于一个简单的示例，假设你有以下矩阵：

```
R> foo <- matrix(1:12,4,3)
R> foo
     [,1] [,2] [,3]
[1,]    1    5    9
[2,]    2    6   10
[3,]    3    7   11
[4,]    4    8   12
```

假设你想计算每一行的和。如果你调用以下代码，你只会得到所有元素的总和，而这不是你想要的。

```
R> sum(foo)
[1] 78
```

你也可以像这样使用`for`循环：

```
R> row.totals <- rep(NA,times=nrow(foo))
R> for(i in 1:nrow(foo)){
+   row.totals[i] <- sum(foo[i,])
+ }
R> row.totals
[1] 15 18 21 24
```

这将循环遍历每一行，并将总和存储在`row.totals`中。但你可以使用`apply`以更简洁的形式得到相同的结果。调用`apply`时，你必须指定至少三个参数。第一个参数`X`是你要循环处理的对象。第二个参数`MARGIN`是一个整数，标记了要操作的`X`的哪一维（行、列等）。最后，`FUN`提供你希望在每个维度上执行的函数。通过以下调用，你将得到与之前`for`循环相同的结果。

```
R> row.totals2 <- apply(X=foo,MARGIN=1,FUN=sum)
R> row.totals2
[1] 15 18 21 24
```

`MARGIN`索引遵循矩阵和数组维度的位置顺序，如第三章所讨论的那样——`1`总是指行，`2`指列，`3`指层，`4`指块，依此类推。要指示 R 对`foo`的每一列求和，只需将`MARGIN`参数更改为`2`。

```
R> apply(X=foo,MARGIN=2,FUN=sum)
[1] 10 26 42
```

`FUN`所提供的操作应与所选择的`MARGIN`相适应。因此，如果你选择了`MARGIN=1`或`MARGIN=2`的行或列，请确保`FUN`函数适用于向量。或者，如果你有一个三维数组并使用`apply`函数设置`MARGIN=3`，请确保将`FUN`设置为适用于矩阵的函数。下面是你可以输入的示例：

```
R> bar <- array(1:18,dim=c(3,3,2))
R> bar
, , 1

     [,1] [,2] [,3]
[1,]    1    4    7
[2,]    2    5    8
[3,]    3    6    9

, , 2

     [,1] [,2] [,3]
[1,]   10   13   16
[2,]   11   14   17
[3,]   12   15   18
```

然后，进行以下调用：

```
R> apply(bar,3,FUN=diag)
     [,1] [,2]
[1,]    1   10
[2,]    5   14
[3,]    9   18
```

这将提取`bar`的每个矩阵层的对角元素。每次对矩阵调用`diag`时都会返回一个向量，这些向量会作为新矩阵的列返回。`FUN`参数也可以是任何适当的用户定义函数，你将在第十一章中看到使用自己函数的`apply`示例。

##### 其他 apply 函数

基本的`apply`函数有不同的变体。例如，`tapply`函数对目标对象的子集执行操作，这些子集是通过一个或多个因子向量定义的。作为示例，让我们回到第 8.2.3 节的代码，该代码读取一个关于钻石定价的基于 Web 的数据文件，设置数据框的适当变量名称，并显示前五条记录。

```
R> dia.url <- "http://www.amstat.org/publications/jse/v9n2/4cdata.txt"
R> diamonds <- read.table(dia.url)
R> names(diamonds) <- c("Carat","Color","Clarity","Cert","Price")
R> diamonds[1:5,]
 Carat Color Clarity Cert Price
1 0.30     D     VS2  GIA  1302
2 0.30     E     VS1  GIA  1510
3 0.30     G    VVS1  GIA  1510
4 0.30     G     VS1  GIA  1260
5 0.31     D     VS1  GIA  1641
```

若要计算按`Color`分组的钻石总值，你可以像这样使用`tapply`：

```
R> tapply(diamonds$Price,INDEX=diamonds$Color,FUN=sum)
     D      E      F      G      H      I
113598 242349 392485 287702 302866 207001
```

这将对目标向量`diamonds$Price`的相关元素求和。相应的因子向量`diamonds$Color`传递给`INDEX`，感兴趣的函数通过`FUN=sum`指定，正如之前所做的那样。

另一个特别有用的替代方案是`lapply`，它可以对列表中的每个成员逐个操作。在第 10.2.1 节中，回忆一下你写了一个`for`循环来检查以下列表中的矩阵：

```
R> baz <- list(aa=c(3.4,1),bb=matrix(1:4,2,2),cc=matrix(c(T,T,F,T,F,F),3,2),
               dd="string here",ee=matrix(c("red","green","blue","yellow")))
```

使用`lapply`，你可以用一行简短的代码检查列表中的矩阵。

```
R> lapply(baz,FUN=is.matrix)
$aa
[1] FALSE

$bb
[1] TRUE

$cc
[1] TRUE

$dd
[1] FALSE

$ee
[1] TRUE
```

请注意，`lapply` 不需要任何边界或索引信息；R 知道将 `FUN` 应用到指定列表的每个成员。返回值本身是一个列表。另一种变体 `sapply` 返回与 `lapply` 相同的结果，但以数组的形式呈现。

```
R> sapply(baz,FUN=is.matrix)
   aa    bb    cc    dd    ee
FALSE  TRUE  TRUE FALSE  TRUE
```

这里，结果作为一个向量提供。在这个例子中，`baz` 有一个 `names` 属性，它会被复制到返回对象的相应条目中。

`apply` 的其他变体包括 `vapply`，它类似于 `sapply`，但有一些相对细微的区别；还有 `mapply`，它可以同时操作多个向量或列表。要了解更多关于 `mapply` 的内容，请参阅 `?mapply` 帮助文件；`vapply` 和 `sapply` 都在 `?lapply` 帮助文件中有介绍。

R 的所有 `apply` 函数都允许将额外的参数传递给 `FUN`；其中大多数通过省略号来实现这一点。例如，再次查看矩阵 `foo`：

```
R> apply(foo,1,sort,decreasing=TRUE)
     [,1] [,2] [,3] [,4]
[1,]    9   10   11   12
[2,]    5    6    7    8
[3,]    1    2    3    4
```

这里你已对矩阵的每一行应用了 `sort` 函数，并提供了额外的参数 `decreasing=TRUE`，将行按从大到小排序。

一些程序员更倾向于在可能的情况下使用一系列 `apply` 函数，以提高代码的简洁性和整洁性。然而，请注意，这些函数在计算速度或效率上通常不会比显式循环有任何实质性的提升（尤其是在 R 的较新版本中）。此外，当你刚开始学习 R 语言时，显式循环通常更容易阅读和理解，因为操作是逐行清晰地呈现的。

**练习 10.5**

1.  在文本中最近的例子基础上，编写一个隐式循环，计算通过调用 `apply(foo,1,sort,decreasing=TRUE)` 返回的矩阵中所有列元素的乘积。

1.  将以下 `for` 循环转换为一个隐式循环，实现完全相同的功能：

    ```
    matlist <- list(matrix(c(T,F,T,T),2,2),
                    matrix(c("a","c","b","z","p","q"),3,2),
                    matrix(1:8,2,4))
    matlist
    for(i in 1:length(matlist)){
        matlist[[i]] <- t(matlist[[i]])
    }
    matlist
    ```

1.  在 R 中，将以下 4 × 4 × 2 × 3 数组存储为对象 `qux`：

    ```
    R> qux <- array(96:1,dim=c(4,4,2,3))
    ```

    即，它是一个四维数组，由三个块组成，每个块是一个由两层 4 × 4 矩阵组成的数组。然后，执行以下操作：

    1.  编写一个隐式循环，获取所有第二层矩阵的对角元素，生成以下矩阵：

        ```
             [,1] [,2] [,3]
        [1,]   80   48   16
        [2,]   75   43   11
        [3,]   70   38    6
        [4,]   65   33    1
        ```

    1.  编写一个隐式循环，返回通过访问 `qux` 中每个矩阵的第四列形成的三个矩阵的 `dim`，无论层或块如何，再通过另一个隐式循环计算该返回结构的行和，最终得到以下向量：

        ```
        [1] 12 6
        ```

### 10.3 其他控制流机制

为了结束本章，你将学习另外三种控制流机制：`break`、`next` 和 `repeat`。这些机制通常与之前学过的循环和 `if` 语句一起使用。

#### *10.3.1 声明 break 或 next*

通常，`for` 循环只有在 `loopindex` 耗尽 `loopvector` 时才会退出，而 `while` 循环则只有在 `loopcondition` 评估为 `FALSE` 时才会退出。但你也可以通过声明 `break` 来预先终止循环。

例如，假设你有一个数字 `foo`，你想用它除以数值向量 `bar` 中的每个元素。

```
R> foo <- 5
R> bar <- c(2,3,1.1,4,0,4.1,3)
```

此外，假设你想逐个元素地将 `foo` 除以 `bar`，但如果某个结果评估为 `Inf`（例如除以零时），则希望停止执行。为此，你可以在每次迭代中使用 `is.finite` 函数（第 6.1.1 节），并在返回 `FALSE` 时发出 `break` 命令以终止循环。

```
R> loop1.result <- rep(NA,length(bar))
R> loop1.result
[1] NA NA NA NA NA NA NA
R> for(i in 1:length(bar)){
+   temp <- foo/bar[i]
+   if(is.finite(temp)){
+       loop1.result[i] <- temp
+   } else {
+       break
+   }
+ }
R> loop1.result
[1] 2.500000 1.666667 4.545455 1.250000     NA     NA     NA
```

在这里，循环通常进行除法运算，直到遇到 `bar` 的第五个元素并进行除以零的运算，结果为 `Inf`。经过条件检查后，循环立即结束，剩余的 `loop1.result` 条目保持原样——为 `NA`。

调用 `break` 是一种相对激烈的操作。通常，程序员只会在作为安全防护措施时使用它，用来突出或避免意外的计算。对于更常规的操作，最好使用其他方法。例如，示例循环完全可以通过 `while` 循环或面向向量的 `ifelse` 函数来复制，而不依赖 `break`。

你也可以使用 `next` 来代替 `break`，这样就可以简单地跳到下一次迭代并继续执行。考虑以下例子，其中使用 `next` 可以避免除以零的情况：

```
R> loop2.result <- rep(NA,length(bar))
R> loop2.result
[1] NA NA NA NA NA NA NA
R> for(i in 1:length(bar)){
+   if(bar[i]==0){
+       next
+   }
+   loop2.result[i] <- foo/bar[i]
+ }
R> loop2.result
[1] 2.500000 1.666667 4.545455 1.250000       NA 1.219512 1.666667
```

首先，循环检查 `bar` 的第 `i` 个元素是否为零。如果是，则声明 `next`，因此 R 会忽略循环中大括号部分的后续代码行，并自动跳到 `loopindex` 的下一个值。在当前的示例中，循环跳过了 `bar` 的第五个元素（保持该位置的原始 `NA` 值），并继续执行剩余的 `bar`。

请注意，如果你在嵌套循环中使用 `break` 或 `next`，该命令只会应用于最内层的循环。只有内层循环会退出或跳到下一次迭代，任何外层循环将正常继续。例如，让我们回到 第 10.2.1 节 中的嵌套 `for` 循环，这些循环用于填充一个矩阵，矩阵中是两个向量的倍数。这次你将在内层循环中使用 `next` 跳过某些值。

```
R> loopvec1 <- 5:7
R> loopvec1
[1] 5 6 7
R> loopvec2 <- 9:6
R> loopvec2
[1] 9 8 7 6
R> baz <- matrix(NA,length(loopvec1),length(loopvec2))
R> baz
     [,1] [,2] [,3] [,4]
[1,]   NA   NA   NA   NA
[2,]   NA   NA   NA   NA
[3,]   NA   NA   NA   NA
R> for(i in 1:length(loopvec1)){
+   for(j in 1:length(loopvec2)){
+       temp <- loopvec1[i]*loopvec2[j]
+       if(temp>=54){
+           next
+       }
+       baz[i,j] <- temp
+   }
+ }
R> baz
     [,1] [,2] [,3] [,4]
[1,]   45   40   35   30
[2,]   NA   48   42   36
[3,]   NA   NA   49   42
```

如果当前元素的乘积大于或等于 54，则内层循环跳到 `next` 迭代。请注意，效果仅适用于 *最内层的循环*——即，只有 `j loopindex` 被预先递增，而 `i` 保持不变，外层循环正常继续。

我一直在使用 `for` 循环来说明 `next` 和 `break`，但它们在 `while` 循环中也表现得一样。

#### *10.3.2 repeat 语句*

另一种重复一组操作的选项是`repeat`语句。它的定义非常简单。

```
repeat{
    do any code in here
}
```

请注意，`repeat`语句不包括任何类型的`loopindex`或`loopcondition`。为了停止大括号内代码的重复，你必须在大括号内使用`break`声明（通常在`if`语句中）；如果没有它，大括号内的代码将无限重复，形成一个无限循环。为了避免这种情况，你必须确保操作在某个时刻会导致循环达到`break`。

为了展示`repeat`的实际应用，你将用它来计算著名的数学序列*斐波那契数列*。斐波那契数列是一个无限的整数序列，起始为 1,1,2,3,5,8,13,...其中每个项是前两个项的和。形式化地说，如果*F[n]*表示第*n*个斐波那契数，那么你会得到：

| *F*[n][+][1] = *F[n]* + *F*[n][−][1]; | *n* = 2,3,4,5,... |
| --- | --- |

其中

*F*[1] = *F[2]* = 1。

以下`repeat`语句计算并打印斐波那契数列，直到它达到大于 150 的项为止：

```
R> fib.a <- 1
R> fib.b <- 1
R> repeat{
+   temp <- fib.a+fib.b
+   fib.a <- fib.b
+   fib.b <- temp
+   cat(fib.b,", ",sep="")
+   if(fib.b>150){
+       cat("BREAK NOW...\n")
+       break
+   }
+ }
2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, BREAK NOW...
```

首先，序列通过存储前两个项（都为 1）初始化为`fib.a`和`fib.b`。然后进入`repeat`语句，它使用`fib.a`和`fib.b`计算序列中的下一个项，并将其存储为`temp`。接下来，`fib.a`被覆盖为`fib.b`，`fib.b`被覆盖为`temp`，使得这两个变量在序列中向前移动。也就是说，`fib.b`变成新计算的斐波那契数，`fib.a`变成到目前为止序列中的倒数第二个数字。然后使用`cat`将`fib.b`的新值打印到控制台。最后，会检查最新的项是否大于 150，如果是，则声明`break`。

当你运行代码时，大括号内的区域会反复执行，直到`fib.b`达到第一个大于 150 的数字，即 89 + 144 = 233。一旦发生这种情况，`if`语句的条件被评估为`TRUE`，然后 R 执行`break`，终止循环。

`repeat`语句不像标准的`while`或`for`循环那样常用，但如果你不想受限于正式指定`for`循环的`loopindex`和`loopvector`，或`while`循环的`loopcondition`，它是非常有用的。然而，使用`repeat`时，你必须更加小心，以防止出现无限循环。

**习题 10.6**

1.  使用第 10.3.1 节中的相同对象，

    ```
    foo <- 5
    bar <- c(2,3,1.1,4,0,4.1,3)
    ```

    执行以下操作：

    1.  编写一个`while`循环——*不使用*`break`或`next`——它将达到与第 10.3.1 节中的`break`示例完全相同的结果。也就是说，生成与文本中`loop2.result`相同的向量。

    1.  使用`ifelse`函数而不是循环，获得与`loop3.result`相同的结果，`loop3.result`是关于`next`的示例。

1.  为了展示第 10.2.2 节中的`while`循环，你使用了向量

    ```
    mynumbers <- c(4,5,1,2,6,2,4,6,6,2)
    ```

    逐步填充 `mylist`，使其包含与 `mynumbers` 中的值匹配的单位矩阵。循环的指令是当它到达数值向量的末尾或遇到大于 5 的数字时停止。

    1.  使用 `break` 声明编写一个 `for` 循环，完成相同的操作。

    1.  编写一个 `repeat` 语句，完成相同的操作。

1.  假设你有两个列表，`matlist1` 和 `matlist2`，它们的成员都是数值矩阵。假设所有成员都有有限的、非缺失的值，但*不要*假设矩阵的维度在整个过程中相同。编写一对嵌套的 `for` 循环，目的是根据以下指导方针创建一个结果列表 `reslist`，该列表包含两个列表成员的所有可能的 *矩阵乘积*（参见 第 3.3.5 节）：

    – `matlist1` 对象应该在外层循环中被索引/搜索，而 `matlist2` 对象应该在内层循环中被索引/搜索。

    – 你只对 `matlist1` 中成员与 `matlist2` 中成员按顺序进行的可能矩阵乘积感兴趣。

    – 如果某个乘积不可行（即，如果 `matlist1` 中某个成员的 `ncol` 不匹配 `matlist2` 中某个成员的 `nrow`），则应跳过该乘法，在 `reslist` 中相关位置存储字符串 `"not possible"`，并直接进行下一个矩阵乘法。

    – 你可以定义一个 `counter`，在每次比较时递增（在内层循环中），以跟踪 `reslist` 的当前位置。

    因此，请注意，`reslist` 的 `length` 将等于 `length(matlist1)*length(matlist2)`。现在，确认以下结果：

    1.  如果你有

        ```
        matlist1 <- list(matrix(1:4,2,2),matrix(1:4),matrix(1:8,4,2))
        matlist2 <- matlist1
        ```

        那么除了成员 `[[1]]` 和 `[[7]]` 外，`reslist` 中的所有成员应为 `"not possible"`。

    1.  如果你有

        ```
        matlist1 <- list(matrix(1:4,2,2),matrix(2:5,2,2),
                         matrix(1:16,4,2))
        matlist2 <- list(matrix(1:8,2,4),matrix(10:7,2,2),
                         matrix(9:2,4,2))
        ```

        那么 `reslist` 中只有 `"not possible"` 的成员应为 `[[3]]`、`[[6]]` 和 `[[9]]`。

##### 本章的重要代码

| **函数/操作符** | **简要描述** | **首次出现** |
| --- | --- | --- |
| `if( ){ }` | 条件检查 | 第 10.1.1 节，第 180 页 |
| `if( ){ } else { }` | 检查和替代 | 第 10.1.2 节，第 183 页 |
| `ifelse` | 元素级 `if-else` 检查 | 第 10.1.3 节，第 185 页 |
| `switch` | 多重 `if` 选择 | 第 10.1.5 节，第 190 页 |
| `for( ){ }` | 迭代循环 | 第 10.2.1 节，第 194 页 |
| `while( ){ }` | 条件循环 | 第 10.2.2 节，第 200 页 |
| `apply` | 按边缘隐式循环 | 第 10.2.3 节，第 205 页 |
| `tapply` | 按因子隐式循环 | 第 10.2.3 节，第 207 页 |
| `lapply` | 按成员隐式循环 | 第 10.2.3 节, 第 207 页 |
| `sapply` | 与`lapply`类似，返回数组 | 第 10.2.3 节, 第 207 页 |
| `break` | 退出显式循环 | 第 10.3.1 节, 第 210 页 |
| `next` | 跳过到下一个循环迭代 | 第 10.3.1 节, 第 210 页 |
| `repeat{ }` | 重复执行代码直到遇到`break` | 第 10.3.2 节, 第 212 页 |
