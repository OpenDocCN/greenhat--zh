## 第八章：逻辑编程

![Image](img/common01.jpg)

*逻辑编程* 起源于形式逻辑学科。它是一种声明式编程风格，专注于 *需要做什么*，而不是 *如何做*。这一领域中最著名的编程语言是 Prolog（参见 [**5**]）。Prolog 和逻辑编程的一个巨大优势是，它提供了一个平台，可以自然流畅地表达和解决某些类型的问题（通常涉及某种搜索）。缺点是，对于其他类型的问题，逻辑编程可能非常低效。

好消息是，Racket 允许你同时享受两全其美的方案。Racket 提供了一个类似 Prolog 风格的逻辑编程库，名为 *Racklog*。Racklog 在语义上与 Prolog 非常相似，但它是 Racket 语法的嵌入式扩展。Racklog 库可以通过 `(require racklog)` 形式访问。

### 前言

逻辑编程完全围绕事实及事实之间的关系展开。在普通的 Racket 中，如果我们想定义什么是咖啡饮品，可能会这样写：

```
> (define coffee '(moka turkish expresso cappuccino latte))
```

然后我们可以通过使用成员函数来询问某个东西是否是咖啡饮品。

```
> (member 'latte coffee)
'(latte)

> (member 'milk coffee)
#f
```

Racklog 定义我们咖啡事实的方式如下。请注意，所有内建的 Racklog 对象名称都以百分号 (`%`) 开头，以避免与标准 Racket 名称冲突。用户名不必遵循这一惯例。

```
> (require racklog)

> (define %coffee
    (%rel ()
          [('moka)]
          [('turkish)]
          [('expresso)]
          [('cappuccino)]
          [('latte)]))
```

这样的事实集合在 Prolog 中通常被称为 *数据库*。我们可以通过 `%which` 形式（查询 *哪些* 事实为真）来 *查询* 我们的咖啡事实（技术上是 *子句*）。请注意，稍后会解释 `%rel` 和 `%which` 形式中的空括号的目的。

```
> (%which () (%coffee 'latte))
'()

> (%which () (%coffee 'milk))
#f
```

由于 `milk` 不在我们的 `%coffee` 事实中，查询 `(%which () (%coffee 'milk))` 如预期返回了 false。表达式 `(%coffee 'milk)` 在 `%which` 子句中被称为 *目标*。以这种方式使用时，`%coffee` 被称为 *谓词*。本质上，我们在问，*牛奶是咖啡吗？* 在这个例子中，目标被认为是*失败*的。当我们询问 `latte` 时，查询返回了空列表 `()`。任何类型的返回列表（即使是空的）都是 Racklog 表示成功的方式。你也可以用明确的目标查询 Racklog，且这些目标总是成功或总是失败，如下所示。

```
> (%which () %true)
'()

> (%which () %fail)
#f
```

假设我们想知道哪些东西被认为是咖啡饮品。我们可以这样提问。

```
> (%which (c) (%coffee c))
'((c . moka))
```

当 `%which` 找到匹配项时，它会返回一对对的列表。`c` 标识符是一个本地逻辑变量，`%which` 形式使用它来指示哪个项被匹配（即 *绑定* 或 *实例化*）到该标识符。请注意，绑定逻辑变量与绑定 Racket 标识符是不同的过程。在这种情况下，标识符 `c` 并没有被赋值，而是作为一种机制，将逻辑变量与从数据库中检索到的值关联起来。虽然“绑定”这个术语可以在两种情况下使用，但我们通常会使用“实例化”这个术语来区分绑定逻辑变量和绑定 Racket 标识符。`%which` 的第二个子形式（即 `(c)`）可以是一个这样的本地逻辑变量列表。这个列表仅仅是用来向 Racklog 声明在接下来的表达式中使用了哪些逻辑变量。

这里正在发生的过程叫做 *统一*。有两个重要的因素在起作用。第一个是模式匹配。第二个是前面提到的实例化。如果查询中没有逻辑变量，那么查询表达式的结构必须与数据库中的对应值完全匹配才能成功。我们在查询尝试 `(%which () (%coffee 'milk))` 时看到了这个过程失败，因为数据库中没有完全匹配的项。如果查询表达式中有逻辑变量，它们可以与数据库中的相应元素匹配。到目前为止，我们只看到过一个简单的例子，查询表达式只包含一个逻辑变量，而数据库只包含一些原子值。我们很快会遇到更有趣的例子。

我们可以使用 `(%more)` 查询我们的咖啡数据库，以获取更多的咖啡饮品。每次调用 `%more` 时，都会生成更多的匹配项。

```
> (%more)
'((c . turkish))

> (%more)
'((c . expresso))

> (%more)
'((c . cappuccino))

> (%more)
'((c . latte))

> (%more)
#f
```

请注意，当我们用完咖啡事实时，`(%more)` 会失败（返回 `#f`）。

如果我们只需要知道是否有咖啡，我们可以这样提问，其中表达式 `(_)` 表示一个匿名变量，它可以匹配任何内容：

```
> (%which () (%coffee (_)))
'()
```

### 基础知识

到目前为止，我所展示的看起来只是在做 Racket 已经能够完成的同样事情，但 Racklog 是为更复杂的事情而设计的。我们将看到可以定义更复杂的关系，比如父子关系。这种关系可以自然地扩展到祖父母–子女关系，依此类推。由于这些关系已在我们的数据库中定义，我们可以提出这样的问题，例如 *Tom 的父母是谁？* 或 *Dick 的孙子是谁？*

#### *了解你的亲戚*

了解咖啡饮品可能不会让你彻夜未眠，但知道你的亲戚是谁可能会。尽管如此，我们将创建一个简单的父子数据库，进一步扩展我们对 Racklog 的了解。

```
> (define %parent
    (%rel ()
          [('Wilma 'Pebbles)]
          [('Fred 'Pebbles)]
          [('Homer 'Bart)]
          [('Dick 'Harry)]
          [('Sam 'Tim)]
          [('William 'Henry)]
          [('Henry 'John)]
          [('Mary 'Sam)]
          [('Dick 'Harriet)]
          [('Tom 'Dick)]
          [('George 'Sam)]
          [('Tim 'Sue)]))
```

每个关系的第一个项是父母，第二个是孩子（实际上，你可以决定哪个是哪个；这只是一种约定）。假设在定义了`%parent`后，发现`Lisa`和`Maggie`需要作为`Homer`的孩子被添加进来。这可以通过使用两种`%assert!`形式之一来解决。

```
> (%assert! %parent () [('Homer 'Lisa)])
> (%assert-after! %parent () [('Homer 'Maggie)])
```

第一个表达式将`Homer`作为`Lisa`的父母添加到所有其他子句之后。但是要注意，`%assert-after!`会在所有其他子句*之前*添加子句（不要问我们为什么）。为了演示这一点，让我们找出`Homer`的所有孩子。

```
> (%which (c) (%parent 'Homer c))
'((c . Maggie))

> (%more)
'((c . Bart))

> (%more)
'((c . Lisa))
```

不需要预先填充关系的值。我们可以创建一个空关系，并像这样向其中添加条目。

```
> (define %parent %empty-rel)
> (%assert! %parent () [('Adam 'Bill)])
> (%assert! %parent () [('Noah 'Andy)])
```

我们不必局限于单一的世代。我们也可以询问祖父母。祖父母是指那些子女是他人父母的人。我们可以这样定义这种关系：

```
> (define %grand
 ➊ (%rel (g p c)
       ➋ [(g c)
            ➌ (%parent g p) (%parent p c)]))
```

在这种情况下，第二个子形式➊是一个符号列表`(g p c)`（分别表示祖父母、父母和孩子）。如同`%which`所提到的，这个列表只是向 Racklog 声明将在其余表达式中使用的局部逻辑变量。与其他关系不同，每个子句只包含单个表达式，而在这个例子中，子句包含了三个表达式。如果你熟悉 Prolog（如果你不熟悉也没关系），这可以表示为如下形式：

```
grand(G,C) :- parent(G,P), parent(P,C).
```

这种类型的表达式被称为*规则*。在 Racklog 版本中，我们将该表达式与➋进行匹配。在 Prolog 术语中，这被称为规则的*头部*（Racket 代码`(g c)`在 Prolog 版本中相当于`grand(G,C)`）。接下来，我们有两个子目标（称为规则的*主体*），也必须与➌匹配。用通俗的话来说，这意味着如果`g`是`p`的父母，并且`p`是`c`的父母，那么`g`就是`c`的祖父母。

让我们来看看查询`(%which (k) (%grand` ’`Tom k))`的执行过程，这个查询是在问谁是`Tom`的孙子（`k`）。通过这个查询，我们的`%grand`定义中的局部变量`g` ➌ 被实例化为`Tom`。变量`k`和`c`被关联在一起（尽管它们目前还没有具体的值）；如上所述，关联这些变量的过程称为*统一*。Racklog 随后扫描其父数据库（假设我们有原始的父母数据集），直到找到一条记录，其中`Tom`是父母之一。在这种情况下，有一条记录显示`Tom`是`Dick`的父亲。因此，第一个子目标成功，结果是`p`被实例化为`Dick`。现在，第二个子目标被测试（`(%parent p c)`，通过统一变成了`(%parent` ’`Dick c)`）。Racklog 扫描其父数据库，发现`Dick`是`Harry`的父亲，此时变量`c`（通过统一也就是`k`）被实例化为`Harry`。在 DrRacket 中执行该查询时，我们确实得到了预期的结果。

```
> (%which (k) (%grand 'Tom k))
'((k . Harry))
```

如果我们想看看`Tom`是否有其他孙子，可以使用`(%more)`。

```
> (%more)
'((k . Harriet))
```

在最初匹配`Harry`时，父母（`p`）被实例化为`Dick`。在`(%more)`背后发生的事情是，它实际上触发了规则的失败。Racklog 随后*回溯*到目标`(%parent p c)`，并且将变量`c`进行反实例化（它不会反实例化`p`，因为`p`是在之前的目标中实例化的）。然后它在数据库中查找`Dick`的另一个父母匹配，找到了第二条记录，显示`Harriet`是`Dick`的孩子（因此是`Tom`的孙子）。

逻辑编程的一个优点是同一关系可以以不同的方式提问。我们问了谁是`Tom`的孙子，但我们也可以通过以下方式来问谁有孙子：

```
> (%which (g) (%grand g (_)))
'((g . William))

> (%more)
'((g . Tom))

> (%more)
#f
```

或者我们可以问`Homer`是否是祖父。

```
> (%which () (%grand 'Homer (_)))
#f
```

这就是 Racklog 扩展 Racket 功能的一种方式。我们将在下一节中看到更多这种灵活性的例子。

如果我们只是想列出父母，可以使用目标`(%parent p (_))`，然后输入一些`(%more)`命令。每次都输入`(%more)`以查看目标是否能够重新满足，确实有些繁琐。解决这个问题的一种方法是使用`%bag-of`。`%bag-of`谓词接受三个参数：我们想要返回的 Racket 表达式（在这种情况下，就是逻辑变量`p`的值），要测试的目标（在此为`(%parent p (_))`），以及用来实例化计算出的结果列表的变量（也就是`p`）。这里有一个例子。

```
> (%which (p) (%bag-of p (%parent p (_)) p))
'((p Wilma Fred Homer Dick William Henry Mary Sam George Dick Tom Tim))
```

在这个例子中，我们只是使用了`p`作为计算结果，但我们可以通过这种方式构造查询来稍微美化一下输出（从中可以看出，逻辑变量`p`的值与字面量`parent`一起使用，生成最终结果）。

```
> (%which (p) (%bag-of (cons 'parent p) (%parent p (_)) p))
'((p
   (parent . Wilma)
   (parent . Fred)
   (parent . Homer)
   (parent . Dick)
   (parent . William)
   (parent . Henry)
   (parent . Mary)
   (parent . Sam)
   (parent . George)
   (parent . Dick)
   (parent . Tom)
   (parent . Tim)))
```

这里有一种更简单的方式来获得类似的输出。

```
> (%find-all (p) (%parent p (_)))
'(((p . Wilma))
  ((p . Fred))
  ((p . Homer))
  ((p . Dick))
  ((p . William))
  ((p . Henry))
  ((p . Mary))
  ((p . Sam))
  ((p . George))
  ((p . Dick))
  ((p . Tom))
  ((p . Tim)))
```

使用`%bag-of`和`%find-all`将按与使用`(%more)`相同的顺序列出值。因此，一些条目可能会重复（例如本例中的`Dick`）。为了只获得唯一值，我们可以改用`%set-of`。

```
> (%which (p) (%set-of p (%parent p (_)) p))
'((p Wilma Fred Homer Dick William Henry Mary Sam George Tom Tim))
```

在本节中，我们介绍了一些逻辑编程的基本思想。如需更详细地了解回溯、统一等内容，请参阅 Clocksin 和 Mellish 所著的经典且易于理解的作品《*Prolog 编程*》[**5**]。

#### *Racklog 谓词*

到目前为止，我们已经探索了 Racklog 提供的一些基本功能。逻辑编程是一种独特的范式，要求使用一些专门的工具才能充分发挥其作用。在本节中，我们将介绍其中的一些工具。

##### 等式

我们已经看到，统一在逻辑编程的语义中起着关键作用。Racklog 提供了等式谓词`%=`, 它直接使用统一来测试结构相等性并实现实例化过程。以下示例应能为这个谓词的应用提供一些见解。

```
> (%which (a b) (%= '(1 potato sack) (cons a b)))
'((a . 1) (b potato sack))

> (%which (x y) (%= (vector x 5) (vector 4 y)))
'((x . 4) (y . 5))

> (%which (x y) (%= (vector x 5) (list 4 y)))
#f

> (%which () (%= (list 4 5) (list 4 5)))
'()
```

第一个例子中发生的情况比较微妙。请注意，`(1 potato sack)`实际上等同于`(1 . (potato sack))`，而`(cons a b)`等同于`(a . b)`。这意味着通过统一，`a`被实例化为 1，`b`被实例化为`(potato sack)`。结果是`((a . 1) (b potato sack))`。实例化总是以对的形式显示，但我们看到第一个元素`(a . 1)`显示为一对，第二个元素`(b potato sack)`显示为列表。回想一下，列表*实际上*就是一对，只是显示方式略有不同。在`(b potato sack)`的例子中，`b`是这一对的`car`，`(potato sack)`是这一对的`cdr`。

`%=`的相反是`%/=`，表示无法统一。回想一下，统一本质上是一个匹配过程。利用上一个例子，观察以下内容：

```
> (%which (a) (%= (list 4 5) (list 5 a)))
#f

> (%which (a) (%/= (list 4 5) (list 5 a)))
'((a . _))
```

在第一个例子中，虽然可以将逻辑变量`a`实例化为 5，但尝试将第一个列表中的 4 与第二个列表中的 5 匹配导致了统一失败。在第二个例子中，统一仍然失败，但由于我们使用了不相等谓词，因此返回了一个列表，逻辑变量`a`保持未绑定状态。

类似于等式谓词的是`*identical*`谓词`%==`。与`%=`不同，`%==`不进行任何实例化。它检查两个表达式是否*完全*相同。

```
> (%which (a b) (%== (list 1 2) (list a b)))
#f

> (%which () (%== (list 1 2) (list 1 2)))
'()
```

`%==`的相反是`%/==`，表示不相等。

##### Let

有时我们希望在查询中使用局部变量来生成中间结果，而不希望这些变量出现在输出中。`%let`谓词提供了一种建立这些隐藏变量的方式。

```
> (define %friends %empty-rel)
> (%assert! %friends () [('jack 'jill)])
> (%assert! %friends () [('fred 'barny)])
> (%which (pals) (%let (a b) (%bag-of (cons a b) (%friends a b) pals)))
'((pals (jack . jill) (fred . barny)))
```

在这个例子中，`%bag-of` 谓词从 `friends` 的结果创建了一个 cons 对，并将其实例化为 `pals`。这里 `a` 和 `b` 是 `%let` 的词法局部变量，因此只有统一的结果被传递到 `pals`。

##### 是

`%is` 谓词的作用与其他 Racklog 谓词略有不同。它有两个参数：第一个表达式通常（但不总是）是标识符，第二个是普通的 Racket 表达式。`%is` 表达式将第二个表达式求值的结果实例化为第一个表达式。通常，第二个表达式中的所有标识符需要在求值 `%is` 表达式之前先实例化。`%is` 表达式可以用来给第一个参数赋值或测试相等性。

```
> (%which (val) (%is val (+ 1 (* 2 3 4))))
'((val . 25))

> (%which () (%is 25 (+ 1 (* 2 3 4))))
'()

> (%which () (%is 5 (+ 1 (* 2 3 4))))
#f
```

`%is` 和 `%=` 之间的一个区别是，对于 `%is`，它的第二个参数中的任何逻辑变量通常需要先实例化，正如这些例子所示。

```
> (%which (x y) (%= (list x 5) (list 4 y)))
'((x . 4) (y . 5))

> (%which (x y) (%is (list x 5) (list 4 y)))
#f

> (%which (x y) (%is (list x y) (list 4 5)))
'((x . 4) (y . 5))
```

然而，在某些情况下，`%is` 可能更有优势。详细信息请参见 Racket 手册^(1)。

##### 算术比较

Racklog 使用 `%=:=` 来测试数值相等，使用 `%=/=` 来测试数值不等，但其他谓词则是你通常期望的。

```
> (%which () (%=:= 1 2))
#f

> (%which () (%=:= 1 1))
'()

> (%which () (%< 1 2))
'()

> (%which () (%>= 5 (+ 2 3)))
'()
```

请注意，这些比较只执行测试，而不会实例化逻辑变量，因此像 `(%which (a) (%=:= a 2))` 这样的表达式会失败。

##### 逻辑运算符

Racklog 支持常见的逻辑谓词 `%not`、`%and` 和 `%or`，如下所示。内置的 `%fail` 目标总是失败，而 `%true` 目标总是成功。

```
> (%which () (%not %fail))
'()

> (%which () (%not %true))
#f

> (%which () (%and %true %true %true))
'()

> (%which () (%and %true %fail %true))
#f

> (%which () (%or %true %fail %true))
'()
```

还有一个 `%if-then-else` 谓词：当给定三个目标时，如果第一个目标成功，它会求值第二个目标；否则，它会求值第三个目标。这里有一个小的测试框架。

```
#lang racket
(require racklog)

(define %spud
  (%rel ()
        [('Russet 'plain)]
        [('Yam 'sweet)]
        [('Kennebec 'plain)]
        [('Sweet 'sweet)]
        [('LaRette 'nutty)]))

(define %spud-taste
  (%rel (tater t taste)
     [(tater t) 
         (%if-then-else
             (%spud tater taste)
             (%is t taste)
             (%is t 'unknown))]))
```

以下交换示例展示了`%if-then-else`的实际应用。

```
> (%which (taste) (%spud-taste 'LaRette taste))
'((taste . nutty))

> (%which (taste) (%spud-taste 'Yam taste))
'((taste . sweet))

> (%which (taste) (%spud-taste 'broccoli taste))
'((taste . unknown))
```

因为’`broccoli` 不在 `%spud` 数据库中，最后的目标被求值，’`unknown` 被实例化为 `taste`（通过 `t`）。

##### 附加

我们已经看到过标准的 Racket 版本的 `append`，它是一个函数，通常接受两个列表并返回一个由这两个列表连接而成的新列表，如下所示。

```
> (append '(1 2 3) '(4 5 6))
'(1 2 3 4 5 6)
```

这是一条单行街道。我们只能问一个问题：如果我有两个列表，将这两个列表合并后的结果列表是什么样的？在我们即将探索的 Racklog 版本中，我们还可以问这些问题：

1.  如果我有一个结果列表，还有哪些其他列表可以组合成这个列表？

1.  如果我有一个起始列表和一个结果列表，哪个列表可以加入到起始列表中以得到结果列表？

1.  如果我有一个结束列表和一个结果列表，什么列表可以加入到结束列表的开头来得到结果列表？

1.  如果我有三个列表，第三个列表是将前两个列表拼接的结果吗？

在我们解释 Racklog 的 `%append` 是如何工作的之前，让我们先看看几个例子。第一个查询回答了原始问题（两个列表连接的结果）。

```
> (%which (result) (%append '(1 2 3) '(4 5 6) result))
'((result 1 2 3 4 5 6))
```

这个查询回答了第二个问题。

```
> (%which (l1) (%append l1 '(4 5 6) '(1 2 3 4 5 6)))
'((l1 1 2 3))
```

这个查询回答了第三个问题。

```
> (%which (l2) (%append '(1 2 3) l2 '(1 2 3 4 5 6)))
'((l2 4 5 6))
```

而这个查询回答了第一个问题。

```
> (%which (lists)
          (%let (l1 l2)
                (%bag-of (list l1 l2)
                         (%append l1 l2 '(1 2 3 4 5 6)) lists)))
'((lists
   (() (1 2 3 4 5 6))
   ((1) (2 3 4 5 6))
   ((1 2) (3 4 5 6))
   ((1 2 3) (4 5 6))
   ((1 2 3 4) (5 6))
   ((1 2 3 4 5) (6))
   ((1 2 3 4 5 6) ())))
```

生成满足特定条件的所有可能性是逻辑编程的强项之一。

如果 `%append` 在 Racklog 中还没有定义，我们可以很容易地从头开始创建它（改编自 [**5**]）：

```
(define %append
  (%rel (h l l1 l2 l3)
     ➊ [('() l l)]
     ➋ [((cons h l1) l2 (cons h l3))
      	➌ (%append l1 l2 l3)]))
```

那么我们的谓词 `%append` 到底是怎么回事呢？它由两个子句组成。第一个 ➊ 简单地表示，如果第一个列表为空，则将该列表与任何列表 `l` 连接的结果就是 `l`。第二个子句 ➋ 比较复杂：`((cons h l1) l2 (cons h l3))` 是规则的头部。该规则的头部需要三个参数，每个参数要么是一个列表，要么是一个未实例化的变量：

1.  如果该参数是一个列表，则其第一个元素会被实例化为 `h`，其余的部分会实例化为 `l1`。

1.  第二个参数被实例化为 `l2`。

1.  如果第三个参数是逻辑变量，则使用 `(cons h l3)` 从第一个参数中提供的 `h` 和在递归调用 `%append` ➌ 时生成的 `l3` 来构建返回值。如果该参数是一个列表，则它的头部必须与第一个参数中的 `h` 匹配，剩余的列表部分将与最后一行中的 `l3` 匹配 ➌。

正如我们所看到的，`%append` 的任何一个或两个参数可能只是一个未实例化的变量。Racklog 使用它的统一过程将具体值与适当的值关联，并使用占位符临时分配空间，以便在适当的实例化后为其他变量分配空间。我们考虑一下第一和第二个参数被实例化为显式列表的情况。一旦统一过程完成 ➋，变量 `l1`（实例化为第一个提供的列表的尾部）和 `l2`（实例化为第二个列表）将用于递归调用 `%append` ➌，期望通过递归调用将现在更短的列表 `l1` 与 `l2` 连接，最终填充 `l3`。由于 `(cons h l3)` 被用来构造最终值，最终结果就是将两个原始提供的列表连接在一起。

这里是一个演示过程，我们将 ’`(1)` 和 ’`(2 3)` 连接起来（为了简洁起见，我们将使用等号（=）来表示逻辑变量绑定）：

1.  第一步是调用 `(%which (a) (%append` ’`(1)` ’`(2 3) a))`。

1.  然后我们来到了第一个判断点 ➊。由于 ’`(1)` 不匹配 ’`()`, 我们继续执行下一个情况。

1.  此时在代码中我们有 `h=1`，`l1=`’`()` 和 `l2=`’`(2 3)` ➋（稍后我们会看到 `l3`，它用于构造返回值）。

1.  接下来是递归调用 ➌。通过实例化的值，结果为 `(%which (l3) (%append` ’`()` ’`(2 3) l3))`。

1.  我们再次来到第一个判断点 ➊，但现在空列表确实匹配。通过将 `l=`’`(2 3)` 与 `l3` 实例化，我们返回 `l3=`’`(2 3)`。

1.  由于我们已经从递归调用中返回，逻辑变量将恢复到第 3 步中给出的值；特别感兴趣的是`h=1`。但是现在我们也得到了从`l3=’(2 3)`的递归调用中返回的值。我们的代码 ➋ 表示从这一阶段返回的值（`a`）是由`(cons h l3)`构造的。那就是`(1 2 3)`，即所需的最终结果。

其他实例化场景可以以类似的方式进行分析。

##### 成员

另一个有 Racklog 等价物的 Racket 函数是`%member`。如果我们需要自己创建这个函数，一种实现方式如下：

```
(define %member
  (%rel (x y)
        [(x (cons x (_)))]
        [(x (cons (_) y)) (%member x y)]))
```

应该很明显，首先检查`x`是否位于列表的开头（也就是说，`(cons x (~_))`将`x`赋值为列表头部的值，因此它必须匹配正在查找的值）；如果不是，它会检查它是否出现在列表的其余部分。

示例：

```
> (define stooges '(larry curly moe))
> (%which () (%member 'larry stooges))
'()

> (%which () (%member 'fred stooges))
#f

> (%find-all (stooge) (%member stooge stooges))
'(((stooge . larry)) ((stooge . curly)) ((stooge . moe)))
```

#### *Racklog 工具*

在本节中，我们将研究在 Racklog 中实现一些额外的谓词。这些都是常见的列表操作，其实现展示了逻辑编程和 Racklog 的能力。稍后我们将使用`%permutation`谓词（我们会详细解释）。其余的可以视为黑盒，即我们通过提供的示例来展示它们的功能和用法，而不对代码进行详细解释。

##### 选择

根据`select`的使用方式，它可以从列表中选择单个项、返回一个删除项的列表，或返回一个插入项的列表。以下是其定义。

```
(define %select
  (%rel (x r h t)
        [(x (cons x t) t)]
        [(x (cons h t) (cons h r))
         	(%select x t r)]))
```

下面是一些示例。

```
> (%which (r) (%select 'x '(u v w x y z) r)) ; remove 'x from list
'((r u v w y z))

> (%which (s) (%select s '(u v w x y z) '(u v x y z))) ; find value in first
     list that is not in the second
'((s . w))

> (%find-all (s) (%select s '(u v w x y z) (_)))
'(((s . u)) ((s . v)) ((s . w)) ((s . x)) ((s . y)) ((s . z)))

> (%find-all (l) (%select 'a l '(u v w x y z)))
'(((l a u v w x y z))
  ((l u a v w x y z))
  ((l u v a w x y z))
  ((l u v w a x y z))
  ((l u v w x a y z))
  ((l u v w x y a z))
  ((l u v w x y z a)))
```

##### 减法

`%subtract`谓词旨在从一个列表中的元素集合中删除另一个列表中的元素集合。它利用`%select`谓词的功能来实现其结果。实现非常直接，应该容易理解。

```
(define %subtract
  (%rel (s r h t u)
        [(s '() s)]
        [(s (cons h t) r)
             (%select h s u)
             (%subtract u t r)]))
```

谓词的第一个参数是源列表，第二个参数是需要删除的项的列表，最后一个参数是返回的列表。

下面是一些说明`%subtract`用法的示例。

```
> (%which (r) (%subtract '(1 2 3 4) '(2 1) r))
'((r 3 4))

> (%which (r) (%subtract '(1 2 3 4) '(3) r))
'((r 1 2 4))

> (%which (t) (%subtract '(1 2 3 4) t '(2)))
'((t 1 3 4))

> (%which (s) (%subtract s '(1 2 4) '(3)))
'((s 1 2 4 3))
```

##### 排列

有时获得给定列表的所有排列是有用的。为了提供以下谓词的工作原理的一些背景，想象一种生成给定列表所有排列的简单方法是很有帮助的。假设我们有一个从 1 到 4 的数字列表。显然，每个数字必须在某一时刻作为列表中的第一个数字出现。因此，一种方法是从四个列表开始，每个列表由 1 到 4 中的一个数字组成。对于这些列表中的每一个，我们创建一个对应的列表，包含所有剩余的数字，如下所示。

```
(1) (2 3 4)
(2) (1 3 4)
(3) (1 2 4)
(4) (1 2 3)
```

我们现在把问题缩小了一些。我们不再需要生成四个数字列表的所有排列，而只需要生成一个三位数字列表的排列。当然，我们足够聪明，知道可以递归地继续这个过程，处理更小的列表。剩下的就是将各部分重新组合起来。这实际上就是 `%permutation` 谓词所做的事情。

在深入代码之前，回顾一下 `%append` 的作用是很有帮助的，它不仅可以将两个列表连接在一起，还可以找到列表分割成两部分的所有方式。例如，如果我们调用 `%which (l1 l2)` `'`(1 2 3 4)`，其中一个可能的输出是 `'`((l1) (l2 1 2 3 4))`（`l1` 的值是空列表）。有了这些背景知识之后，这里是谓词（代码来源于 [5]）。

```
(define %permutation
  (%rel (l h t u v w)
     ➊ [('() '())]
     ➋ [(l (cons h t))
	      ➌ (%append v (cons h u) l)
	      ➍ (%append v u w)
	      ➎ (%permutation w t)]))
```

这个谓词接受两个参数：一个要排列的列表和一个标识符，用于实例化返回的排列列表。让我们看看当我们调用 `(%which (a) (%permutation` `'`(1 2 3 4) a))` 时会发生什么。因为列表不为空，我们跳过了第一次匹配尝试 ➊。接下来，我们有 `l=`'`(1 2 3 4)` ➋。此时其余的代码用于构造返回值，我们稍后再回到这一部分。下一行开始有些有趣的变化 ➌。正如前面提到的，第一次调用 `%append` 时，第三个参数是一个列表，它会生成一个空列表和列表 `'`(1 2 3 4)`。通过这个结果，我们有 `v =` `'()`，`h=1`，`u=`'`(2 3 4)`。接着看下一行，`v=`'`()` 和 `u=`'`(2 3 4)` 被实例化了，但 `w` 并没有，所以 `(%append v u w)` 只是将 `w` 绑定到 `'`(2 3 4)` ➍。最后，我们生成 `'`(2 3 4)` 的排列，并将结果实例化为 `t` ➎。现在我们处于构造返回值的阶段 ➋。这将生成所有以 1 开头的排列。

那剩下的排列呢？一旦我们通过回溯 ➌ 耗尽了以 1 开头的所有排列，我们最终得到 `%append` 生成的列表 `'(1)` 和 `'(2 3 4)`。此时我们有 `v=`'`(1)`，`h=2`，`u=`'`(3 4)`，所以现在我们有 `w=`'`(1 3 4)` ➍。过程继续进行，就像之前一样，现在开始构建以 2 开头的列表的排列。

让我们看看如何安排四种扑克牌的花色。

```
> (%find-all (s) (%permutation '(♠ ♣ ♡ ♢) s))
'(((s ♠ ♣ ♡ ♢))
  ((s ♠ ♣ ♢ ♡))
  ((s ♠ ♡ ♣ ♢))
  ((s ♠ ♡ ♢ ♣))
  ((s ♠ ♢ ♣ ♡))
  ((s ♠ ♢ ♡ ♣))
  ((s ♣ ♠ ♡ ♢))
  ((s ♣ ♠ ♢ ♡))
  ((s ♣ ♡ ♠ ♢))
  ((s ♣ ♡ ♢ ♠))
  ((s ♣ ♢ ♠ ♡))
  ((s ♣ ♢ ♡ ♠))
  ((s ♡ ♠ ♣ ♢))
  ((s ♡ ♠ ♢ ♣))
  ((s ♡ ♣ ♠ ♢))
  ((s ♡ ♣ ♢ ♠))
  ((s ♡ ♢ ♠ ♣))
  ((s ♡ ♢ ♣ ♠))
  ((s ♢ ♠ ♣ ♡))
  ((s ♢ ♠ ♡ ♣))
  ((s ♢ ♣ ♠ ♡))
  ((s ♢ ♣ ♡ ♠))
  ((s ♢ ♡ ♠ ♣))
  ((s ♢ ♡ ♣ ♠)))
```

通过做一个小调整，我们可以创建一个版本的 `%permutation`，它通过额外的参数——所需的长度——来生成某一长度的所有排列：

```
(define %permute-n
  (%rel (l h t u v w n m)
        [((_) '() 0) !]
        [(l (cons h t) n)
            (%append v (cons h u) l)
            (%append v u w)
            (%is m (sub1 n))
            (%permute-n w t m)]))
```

第三行中的感叹号（`!`）称为 *cut*。cut 是一个总是成功的目标，但它用于防止在 cut 之前回溯。这意味着，如果紧跟在 cut 后面的目标失败（无论是通过回溯还是其他原因），cut 会阻止回溯到任何之前的目标。在这个例子中，一旦我们达到了零的计数，就不需要再寻找额外的、更长的排列。这将使过程更加高效（也就是说，谓词在没有它的情况下仍然能正常工作，但不会测试那些不必要的额外排列）。

由于 Racklog 的模式匹配功能，我们不需要使用两个独立的谓词。我们可以将它们合并为一个谓词，具体如下：

```
(define %permute
  (%rel (l h t u v w n m)

        ;permute all
        [('() '())]
        [(l (cons h t))
            (%append v (cons h u) l)
            (%append v u w)
            (%permute w t)]

        ;permute n
        [((_) '() 0) !]
        [(l (cons h t) n)
            (%append v (cons h u) l)
            (%append v u w)
            (%is m (sub1 n))
            (%permute w t m)]))
```

这里有几个例子：

```
> (%find-all (p) (%permute '(1 2 3) p))
'(((p 1 2 3)) ((p 1 3 2)) ((p 2 1 3)) ((p 2 3 1)) ((p 3 1 2)) ((p 3 2 1)))

> (%find-all (p) (%permute '(1 2 3) p 2))
'(((p 1 2)) ((p 1 3)) ((p 2 1)) ((p 2 3)) ((p 3 1)) ((p 3 2)))
```

现在我们已经打下了基础，让我们来看几个应用实例。

### 应用实例

到目前为止，我们已经介绍了逻辑编程的基本机制。尽管这些话题很有趣，但接下来我们将看看如何解决一些现实世界中的（但属于娱乐性质的）问题。在这里，我们将看到逻辑编程如何提供一个框架，通过声明式的方式来解决问题，更直接地映射问题的约束条件。

#### *SEND + MORE = MONEY*

下面这个著名的娱乐数学问题由 Henry Dudeney 在 1924 年 7 月的《The Strand Magazine》上发表。

![Image](img/p0245-01.jpg)

每个字母代表解答中的一个不同数字。这类问题通常被称为字母算式、加密算术、加密算式或文字加法。尽管这个问题可以通过纸和笔来解决，我们将利用 Racket（通过 Racklog）来解决它。我们将使用一种通常不被推荐的方法：穷举法。这意味着我们将生成（几乎）所有可能的方式，将数字分配给字母（显然 M 是 1，所以我们不会再去寻找那个值）。

在以下代码中，我们使用了上一节定义的 `%permute-n` 谓词。

```
   #lang at-exp racket

   (require infix racklog)

   (define %permute-n
       ; see previous section
       ...)

➊ (define %check
  (%rel (S E N D O R Y s1 s2)
        [((list S E N D O R Y))
        ➋ (%is s1 @${S*1000 + E*100 + N*10 + D +
              1000 + O*100 + R*10 + E})
        ➌ (%is s2 @${10000 + O*1000 + N*100 + E*10 + Y})
           (%=:= s1 s2)]))

➍ (define %solve
  (%rel (S E N D M O R Y p)
        [(S E N D M O R Y)
            (%is M 1)
         ➎ (%permute-n '(0 2 3 4 5 6 7 8 9) p 7) 
            (%check p) 
         ➏ (%= p (list S E N D O R Y))]))
```

解这个谜题的谓词是 `%solve` ➍。首先，它将 1 分配给 M，如前所述。这个谜题中使用的唯一字母（除了 M）是 S、E、N、D、O、R 和 Y。下一步是生成所有可能的排列 `'(0, 2, 3, 4, 5, 6, 7, 8, 9)` ➎（每次取 7 个数字）。调用 `%check` 谓词来测试特定排列是否能解出这个谜题（稍后会介绍 `%check`）。如果当前排列生成了一个解，结果的赋值将被返回 ➏。请注意，如果 `%check` 失败，我们将回溯 ➎ 以生成另一个排列。

`%check`的代码也相当简单。在第一个`%is`语句 ➋ 中，我们只需为当前排列计算算术和 `s1` = SEND + MORE（记住 M 隐含为 1——这里扩展为 1000）。在第二个`%is`语句 ➌ 中，我们计算和 `s2` = MONEY。最后，我们测试 `s1` 是否等于 `s2`。由于算术表达式 ➋ ➌ 相当冗长，我们利用了 *infix* 库，使得计算过程更为清晰。

我们如下生成了解决方案。

```
> (%which (S E N D M O R Y) (%solve S E N D M O R Y))
'((S . 9) (E . 5) (N . 6) (D . 7) (M . 1) (O . 0) (R . 8) (Y . 2))
```

即使我们使用的是一种效率非常低的暴力破解方法，在一台相对健康的计算机上，解答应该在一分钟之内出现。

#### *狐狸、鹅和豆子*

狐狸、鹅和豆子谜题是河流过河类谜题的一个例子。这种谜题相当古老，至少可以追溯到 9 世纪。这类谜题非常适合逻辑编程系统。谜题的叙述大致是这样的：

从前，一位农民去市场买了狐狸、一只鹅和一袋豆子。回家的路上，农民来到河边，他把船停在了那里。但他的船很小，农民只能带着自己和他购买的其中一个物品——狐狸、鹅或豆子。如果留下狐狸，它会吃掉鹅；如果留下鹅，它会吃掉豆子。

农民的任务是将自己和他的购买物品（保持完好）带到河的另一岸。他是如何做到的呢？

尽管这个谜题手动解答并不困难，但它为我们提供了一个机会，展示 Racklog 执行 *深度优先搜索（DFS）* 的固有能力。为了帮助你理解这种搜索是如何工作的，可以想象你在一个小岛上，需要到达灯塔，但你不知道该怎么走，也没有地图。到达目的地的一个方法是开始行驶，每次遇到分岔路口时，仔细记录下你走的路。你继续前进，直到到达目的地，或者走到死胡同或已经走过的地方。如果你到达死胡同或已经走过的地方，你需要 *回溯* 到上一个分岔口，选择一条没走过的路。如果你已经尝试过所有分岔口的路径，你会回到更早的分岔口。最终，如果你按照这种方式继续工作，你将尝试所有可能的路径，最终到达目的地，或者你会发现自己其实在错误的岛屿上（哎呀）。

假设农夫在河流两岸之间往返。使用深度优先搜索（DFS）策略，我们在搜索过程中跟踪每一岸上放置了哪些物品。然后我们从所有物品都在东岸的记录开始。任何时候，我们可以选择不带物品返回对岸，或者选择携带一件物品回到对岸（前提是这些动作不违反谜题的约束）。我们还必须确保，所做的移动不会造成之前已经存在的物品排列。例如，假设我们首先将鹅带过河。现在我们有两个已存状态：一个是所有物品（包括农夫）都在东岸，另一个是狐狸和豆子在东岸，农夫和鹅在西岸。此时，农夫可以选择独自返回东岸，因为这会生成一个新的状态，但如果农夫（愚蠢地）将鹅带回东岸，这将导致已经出现过的状态（起始状态），因此不应考虑。游戏以这种方式继续进行，直到找到解决方案。

西岸用数字 0 表示，东岸用数字 1 表示。使用一个四元素向量来跟踪程序状态。向量的每个元素将表示每个角色的位置（即岸），如表 8-1 所示。

**表 8-1**：狐狸、鹅、豆子状态向量

| **索引** | **角色** |
| --- | --- |
| 0 | 农夫 |
| 1 | 狐狸 |
| 2 | 鹅 |
| 3 | 豆子 |

我们首先定义哪些状态在谓词`%rejects`中是不允许的。

```
#lang racket
(require racklog)

(define %reject
  (%rel ()
        [(#(0 1 1 1))]
        [(#(0 1 1 0))]
        [(#(0 0 1 1))]
        [(#(1 0 0 0))]
        [(#(1 0 0 1))]
        [(#(1 1 0 0))]))
```

第一个被拒绝的状态表明，如果农夫在 0 号岸，则不允许狐狸、鹅和豆子都在 1 号岸。其余状态可以类似地进行分析。通过观察数字模式，`%rejects`可以更简洁地编写：

```
(define %reject
  (%rel (x y)
        [((vector x y y (_))) (%=/= x y)]
        [((vector x x y y)) (%=/= x y)]))
```

如果农夫将物品从一岸移到另一岸，就必须切换农夫的岸和物品的岸。这是由`toggle-item`函数处理的，该函数接受一个状态向量和一个元素索引，并返回一个新的状态向量。请注意，这是一个普通的 Racket 函数，而不是 Racklog 谓词。接下来将展示这一点如何适应。

```
(define (toggle-item s a)
  (for/vector ([i (in-range 4)])
    (let ([loc (vector-ref s i)])
   ➊ (if (or (zero? i) (= i a))
          (- 1 loc)
          loc))))
```

代码`(zero? i)`测试农夫的索引（0），而`(= i a)`检查物品的索引 ➊。回忆一下，`for/vector`根据`let`体中计算的每个项的结果形成一个新的向量。

以下的`%gen-move`谓词生成的移动包括四种可能的船上乘客类型（分别用数字 0 到 3 表示）：农夫单独，或农夫带狐狸、鹅或一袋豆子。

```
(define %gen-move
  (%rel (n t s0 s1)
     ➊ [('() s0 s1)
             (%is s1 (cons 0 (toggle-item s0 0))) !]
     ➋ [((cons n (_)) s0 s1)
             (%is s1 (cons n (toggle-item s0 n)))]
     ➌ [((cons (_) t) s0 s1)
             (%gen-move t s0 s1)]))
```

谓词最初使用列表’`(0 1 2 3)`（表示所有可以移动的项目）和当前状态进行调用。它返回一个对，其中`car`表示正在移动的项目，`cdr`给出结果状态。我们遇到了没有剩余项目可以移动的情况 ➊，因此下一行仅切换农民的状态。注意切割（!）：不需要生成额外的移动，因为没有剩余的项目可以移动。接下来，我们有一个非空列表，因此我们取列表的头部并切换该项目的状态 ➋。最后，我们使用递归调用`%gen-move`处理其余的列表 ➌。

随着搜索的推进，需要确保程序不会通过重新检查已经测试过的状态进入无限循环。为此，我们维护一个包含已访问状态的列表，并将此列表和待检查的状态传递给`%check-history`谓词。如果该状态在历史列表中，检查将失败。

```
(define %check-history
  (%rel (state h t)
        [(state '())]
        [(state (cons h t))
          ➊ (%is #t (equal? state h)) ! %fail]
        [(state (cons (_) t))
	         (%check-history state t)]))
```

在这里，我们遇到了一个先前的状态，因此通过紧随其后的`%fail` ➊失败而不回溯。

接下来是`%gen-valid-move`谓词。该谓词接收当前状态和移动历史。它首先生成一个潜在的移动，并检查移动后银行上剩余的项目是否形成一个合法的组合（即状态不在拒绝列表中）。如果是，它接着检查当前状态是否曾经出现过。如果没有，它将返回这个移动作为有效的移动。

```
(define %gen-valid-move
  (%rel (state hist move s a left-behind)
        [(state hist move)
            (%gen-move '(0 1 2 3)  state (cons a s))
            (%is left-behind (toggle-item state a))         
            (%not (%reject left-behind))
            (%check-history s hist)
            (%is move (cons a s))]))
```

通过前面的开胃菜，我们现在进入正餐：

```
(define %solve
  (%rel (a s state hist move moves m1 m2)
        [(state (_) moves moves)
          ➊ (%is #t (equal? state #(1 1 1 1))) !]
        [(state hist m1 m2)
          ➋ (%gen-valid-move state hist (cons a s))
          ➌ (%is move (cons a s))
          ➍ (%solve s (cons s hist) (cons move m1) m2)]))
```

整体策略非常简单：生成一个有效的移动并检查是否到达解决状态。如果我们遇到死胡同，Racklog 的自动回溯机制将回退并尝试另一个不会导致重复状态的移动。`%solve`谓词以初始状态、一个空列表（表示状态历史）和一个包含至今生成的移动的列表（也是空的）进行调用。最后一个参数是一个标识符，将被实例化为解决难题的移动列表。首先，我们检查谜题是否处于解决状态 ➊；如果是，我们返回移动列表。如果不是，我们获取下一个移动候选和结果状态 ➋（这些被赋值给`move` ➌），然后递归调用`%solve` ➍。如果`%solve`谓词 ➍ 生成了失败，回溯发生。由于`%is`不能重新满足，回溯会继续回到 ➋ 处，生成另一个可能的解决方案。`%solve`谓词返回一个对：第一个元素是船上的乘客指示器（有关数字的含义，请参见`%gen-move`的讨论），第二个元素是移动后的东岸状态。

为了真正解决这个难题，我们像这样调用`%solve`：

```
> (%which (moves) (%solve #(0 0 0 0) '() '() moves))
'((moves
   (2 . #(1 1 1 1))
   (0 . #(0 1 0 1))
   (1 . #(1 1 0 1))
   (2 . #(0 0 0 1))
   (3 . #(1 0 1 1))
   (0 . #(0 0 1 0))
   (2 . #(1 0 1 0))))
```

除了顺序被反转外，输出列表在可读性上还有些欠缺。为了获得更直观的输出，我们定义了几个新的辅助过程。首先，我们创建了 Racklog 谓词版的 Racket `printf`形式，称为`%print`。它的第一个参数是格式化字符串，第二个参数是要打印的值。让它工作需要一些技巧。由于`printf`函数不是谓词，所以不能作为 Racklog 目标调用，也不会返回值，因此正常的实例化方法无法工作。技巧在于将`printf`形式封装在`begin`形式中（该形式按顺序求值表达式，并返回最后一个表达式的值），我们将`#t`作为最终表达式返回。然后，我们可以使用`%is`实例化它，得到一个始终成功的谓词。

```
(define %print
  (%rel (fmt val)
        [(fmt val) (%is #t (begin (printf fmt val) #t))]))
```

第二个辅助过程是一个常规的 Racket 函数，它接受一个状态向量和一个银行编号。它返回一个列表，指示当前银行上有哪些物品。

```
(define (get-items s b)
  (for/list ([i (in-range 4)] #:when (= b (vector-ref s i)))
    (vector-ref #(Farmer Fox Goose Beans) i)))
```

给定一系列解法步骤，`%print-moves`（见下文）将为每个步骤提供两行输出：第一行将表示移动方向和船上的乘客；第二行输出将是一个列表，其中第一个项目是银行 0 的占用者，第二个项目是银行 1 的占用者。我们将此作为一个小练习留给读者去理解其工作原理。

```
(define %print-moves
  (%rel (s t i pass dir b0 b1 d)
        [('()) %true]
        [((cons (cons i s) t))
         (%is pass (vector-ref
             #(Farmer Farmer-Fox Farmer-Goose Farmer-Beans) i))
         (%is d (vector-ref s 0))
         (%is dir (vector-ref #( <- -> ) d))
         (%print "~a\n" (list dir pass))
         (%is b0 (get-items s (- 1 d)))
         (%is b1 (get-items s (vector-ref s 0)))
         (%if-then-else
            (%=:= 0 d)
            (%print "~a\n\n" (list b1 b0))
            (%print "~a\n\n" (list b0 b1)))
         (%print-moves t)]))
```

最后，我们得到了这个：

```
(define %print-solution
  (%rel (moves rev-moves)
        [()
             (%print "~a\n\n" (list (get-items #(0 0 0 0) 0) '()))
             (%solve #(0 0 0 0) '() '() moves)
             (%is rev-moves (reverse moves))
             (%print-moves rev-moves)]))
```

过程`%print-solution`不接受任何参数，但它会生成谜题的解法，反转动作列表，并调用`%print-moves`打印出解法。这里是一个更易读的最终结果：

```
> (%which () (%print-solution))
((Farmer Fox Goose Beans) ())

(-> Farmer-Goose)
((Fox Beans) (Farmer Goose))

(<- Farmer)
((Farmer Fox Beans) (Goose))

(-> Farmer-Fox)
((Beans) (Farmer Fox Goose))

(<- Farmer-Goose)
((Farmer Goose Beans) (Fox))

(-> Farmer-Beans)
((Goose) (Farmer Fox Beans))

(<- Farmer)
((Farmer Goose) (Fox Beans))

(-> Farmer-Goose)
(() (Farmer Fox Goose Beans))

'()
```

请记住，最终的空列表是 Racklog 表示成功的方式。

#### *多少块甜甜圈？*

以下问题出现在 2007 年 10 月 27 日的*Parade*杂志“AskMarilyn”专栏中：

Jack、Janet 和 Chrissy 在他们常去的咖啡馆碰面并买了六个甜甜圈。每个朋友要么总是说真话，要么总是撒谎。Jack 说他拿了一块甜甜圈，但 Janet 说 Jack 拿了两块，Chrissy 则说 Jack 拿了三块以上。另一方面，三个人都一致认为 Janet 拿了两块。假设每个人至少拿了一块，而且没有甜甜圈被切分，问每个人拿了多少块甜甜圈？

逻辑编程系统对于这种类型的问题简直是轻松应对（甜甜圈，早餐——有趣吧？），而 Racklog 也不例外。这个问题的特别之处在于，在 Racklog 中的解决方案主要只是对事实的声明（并附带了一些辅助项）。

这里有一些基本定义；注释应该足以解释它们的功能。

```
#lang racket
(require racklog)

; Each person can have from one to six donuts
(define %can-have
  (%rel (d)
        [(d) (%member d '(1 2 3 4 5 6))]))

; an alias for equality
(define %has (%rel (n) [(n n)]))

; if a person doesn't have d donuts, they have n donuts
(define %not-have
  (%rel (n d)
        [(n d)
           (%can-have n)
           (%=/= n d)]))
```

这里的目的是确定一个人可以拥有多少个甜甜圈，前提是我们说他们不能拥有某个数量（作为第二个参数提供）。由于`%can-have`给出了一个人可以拥有的所有甜甜圈，语句`(%=/= n d)])`将给出他们可以拥有的所有甜甜圈，排除他们不能拥有的数量。

现在我们以两种版本列出每个人的陈述（一种是他们说真话的情况，另一种是他们撒谎的情况）。这里我们将“Chrissy”缩写为“Chris”。

```
(define %statement
  (%rel (Jack Janet Chris)

        ; Jack's statements
        [('jack Jack Janet)
            (%has Janet 2) (%has Jack 1)]
        [('jack Jack Janet)
            (%not-have Janet 2) (%not-have Jack 1)]

        ; Janet's statements
        [('janet Jack Janet)
            (%has Janet 2) (%has Jack 2)]
        [('janet Jack Janet)
            (%not-have Janet 2) (%not-have Jack 2)]

        ; Chris's statements
        [('chris Jack Janet)
            (%has Janet 2) (%can-have Jack) (%> Jack 3)]
        [('chris Jack Janet)
            (%not-have Janet 2) (%can-have Jack) (%<= Jack 3)]))
```

我们的求解器只需要检查每个人的陈述，并查看总甜甜圈是否加起来为六个。

```
(define %solve
  (%rel (Jack Janet Chris)
        [(Jack Janet Chris)
            (%statement 'jack Jack Janet)
            (%statement 'janet Jack Janet)
            (%statement 'chris Jack Janet)
            (%can-have Chris)
            (%is 6 (+ Jack Janet Chris))]))
```

然后就像魔法一样：

```
> (%which (Jack Janet Chris) (%solve Jack Janet Chris))
'((Jack . 3) (Janet . 1) (Chris . 2))
```

#### *Boles 和 Creots*

Boles 和 Creots 是一种传统的纸笔破译游戏，也叫做 Bulls 和 Cows，或 Pigs 和 Bulls。一种商业变体叫做 *Mastermind*，使用的是由彩色小圆钉组成的代码。游戏玩法是由一个玩家选择一个秘密代码（通常是四个或五个独特的数字或字母组成的序列）。另一个玩家然后提出一个猜测，系统提供提示，告诉他们有多少个 boles（正确的数字在正确的位置）和多少个 creots（正确的数字在错误的位置）。玩家继续交换猜测和提示，直到猜测玩家将所有数字按正确顺序猜中。

在这里，我们让计算机尝试猜测一个由人类玩家提供的数字。

策略相当简单：猜测玩家（在此案例中是 Racklog 程序）记录每次猜测和相应的 boles 和 creots 数量。候选猜测由所有可能的数字 0 到 9 的排列组合的暴力生成生成，每个候选猜测都与之前的猜测进行比对，看看是否能得出一致的 boles 和 creots 数量。如果一个候选猜测与之前的猜测没有不一致的地方，它就会成为下一个展示给用户的猜测。为了说明我们的意思，假设游戏已经按下面的表 8-2 进行。

**表 8-2**：Boles 和 Creots 进展情况

| **猜测** | **Boles** | **Creots** |
| --- | --- | --- |
| 2359 | 0 | 2 |
| 1297 | 2 | 1 |

在下一轮中，第一个候选猜测是 1973。与表中的第一个猜测相比，这个猜测有两个正确的数字（但位置错误），因此得到了 0 个 boles 和 2 个 creots。到此为止一切正常；但与第二个猜测相比，得到了 1 个 bole 和 2 个 creots，因此被拒绝。假设下一个候选猜测是 9247。与第一个猜测相比，得到了 0 个 boles 和 2 个 creots；与第二个猜测相比，得到了 2 个 boles 和 1 个 creot，因此它是一个不错的候选猜测。程序猜测 9247，得到用户的提示，并更新表格，记录猜测、boles 和 creots。这个过程会重复，直到有人获胜。

为了模拟一个猜测计算机与提示人类之间的游戏，我们的 Racklog 程序使用一个读-评估-打印循环（REPL），该循环打印出一个猜测，等待用户输入（提示），读取输入，并评估该输入以形成下一个猜测。

在我们开始深入代码之前，先来看看一个示例会话。我已经决定了要猜的数字是 12345。对于每次猜测，我的回应是一个两位数，分别表示 boles 和 creots 的数量。

```
> (%which () (%repl))

Guess: (3 8 2 1 7)
03

Guess: (8 3 1 0 5)
12

Guess: (8 2 3 5 6)
21

Guess: (8 2 0 3 4)
12

Guess: (8 1 4 5 2)
04

Guess: (1 2 3 4 5)
50
'()
```

整个过程的代码如下所示。它依赖于一些支持过程，稍后将更详细地解释。

```
(require racklog)

(define DIGITS 5)

(define %repl
  (%rel (digits guess val boles creots)
        [()
         ➊ (%is digits (randomize-digits))
         ➋ (%is #t (begin (set! history '()) #t))
            (%repl digits)]
        [(digits)
         ➌ (%permute-n digits guess DIGITS)
         ➍ (%consistent? guess)
         ➎ (%print "\nGuess: ~a\n" guess)
         ➏ (%= (cons boles creots) (get-input))
         ➐ (%update-history guess boles creots)
         ➑ (%if-then-else (%=:= boles DIGITS) ! %fail)]))
```

常量`DIGITS`指定用于猜测的数字个数。`%repl`谓词实现了读-评估-打印循环。`%repl`代码生成一个随机化的数字列表，用于生成猜测➊，同时清空`history`列表➋。实际的循环从➌开始，此时生成排列。每个排列都会进行测试➍，直到生成一个可接受的候选猜测为止，回溯过程会继续进行。生成候选猜测后，用户将看到该猜测➎。接着，系统提示用户提供 boles 和 creots 的数量，结果输入将被解析➏。然后，`history`列表会被更新➐。最后，输入将被测试，看是否所有数字都正确➑，如果是，使用一个切割符号（`!`）来终止过程。否则，生成失败，触发回溯并进行额外的猜测。

为了跟踪之前的猜测，定义了一个`history`列表。列表的每个元素是一个包含以下三个元素的三元组：猜测、boles 的数量和 creots 的数量。`history`列表由`%update-history`谓词填充。

```
(define history '())

(define %update-history
  (%rel (guess boles creots)
        [(guess boles creots)
         (%is #t
              (begin
                (set! history (cons (list guess boles creots) history))
                #t))]))
```

如上所示，一个猜测由一组数字表示。我们定义了一个`score`函数，给定两个数字列表，比较它们并返回对应的 boles 和 creots 的数量。

```
(define (score c h)
  (let loop ([l1 c] [l2 h] [boles 0] [creots 0])
    (if (equal? l1 null)
        (cons boles creots)
        (let ([d1 (car l1)]
              [d2 (car l2)]
              [t1 (cdr l1)]
              [t2 (cdr l2)])
          (if (= d1 d2) 
              (loop t1 t2 (add1 boles) creots)
              (loop t1 t2 boles (+ creots (if (member d1 h) 1 0))))))))
```

为了防止程序每次都从相同的初始猜测开始，我们定义了一个数字生成器函数，用于创建一个混乱的数字集合以供选择：

```
(define (randomize-digits)
  (let loop([count 10] [l '()])
    (if (= count 0) l
    (let ([d (random 10)])
      (if (member d l)
          (loop count l)
          (loop (sub1 count) (cons d l)))))))
```

为了创建猜测候选项，我们需要生成随机数字的排列列表。为此，我们重用了在早期章节中介绍的`%permute-n`谓词。

```
(define %permute-n
  (%rel (l h t u v w n m)
        [((_) '() 0) !]
        [(l (cons h t) n)
            (%append v (cons h u) l)
            (%append v u w)
            (%is m (sub1 n))
            (%permute-n w t m)]))
```

一个名为`%consistent?`的谓词接受一个猜测并测试它是否与`history`中的元素一致（如上所定义）。它通过候选猜测调用。

```
(define %consistent?
  (%rel (g h hb hc gb gc t)
        [((_) '()) %true]
        [(g (cons (list h hb hc) t))
            (%is (cons gb gc) (score g h))
            (%and (%=:= hb gb) (%=:= hc gc))
            (%consistent? g t)]
        [(g) (%consistent? g history)]))
```

控制输入和输出的工作由`get-input`和`%print`负责，具体如下所示。

```
(define %print
  (%rel (fmt val)
        [(fmt val)
         (%is #t (begin (printf fmt val) #t))]))
(define (get-input)
  (let ([val (read (current-input-port))])
    (let-values ([(boles creots) (quotient/remainder val 10)])
      (cons boles creots))))
```

在本章的开头部分，我们介绍了逻辑编程范式以及扩展其能力的各种工具和实用程序。在本节中，我们讨论了一些可以通过逻辑编程以自然和声明的方式解决的数学谜题和问题。这些问题展示了逻辑编程内在的强大搜索机制。

### 总结

在这一章，我们概述了逻辑编程范式，并考察了一些有趣的应用。我们已经看到，除了 Racket 的函数式和命令式编程能力之外，它在逻辑编程方面也相当擅长，得益于其 Racklog 库。逻辑编程（特别是 Prolog）被认为是图灵完备的。简单来说，这意味着任何可以用典型的命令式编程语言计算的内容，也可以用逻辑程序计算。技术上来说，它可以用来模拟图灵机（稍后会详细讲解）。话虽如此，逻辑编程在某些问题领域并不总是最优的。例如，涉及大量数值计算的案例，或者已经有了公认的命令式算法的情况，都不适合使用逻辑编程。逻辑编程尤其在搜索问题上表现出色，就像我们在应用部分看到的那样，以及在定理证明等符号计算中。好消息是，使用 Racket 时，你可以选择最适合当前问题的编程方法。

在下一章，我们将探讨一些抽象的计算机模型，例如前文提到的图灵机。
