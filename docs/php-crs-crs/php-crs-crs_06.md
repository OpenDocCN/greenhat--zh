

## 第五章：5 自定义函数



![](img/opener.jpg)

在本章中，你将学习如何声明和使用你自己的*函数*，这些函数是命名的、独立的代码序列，旨在完成特定任务。你将看到函数如何促进代码的重用，因为将代码放入函数中要比每次执行该任务时都重新编写相同的代码序列更高效。函数还可以让你编写通过少量语句就能完成大量工作的程序，因为每个语句都可以调用隐藏在你函数中的复杂逻辑。

自定义函数通常会在与应用程序主程序语句不同的文件中声明。这源于*PHP 标准推荐（PSRs）*，它是一系列关于 PHP 编程的指南和最佳实践。根据 PSR-1，文件应该要么声明符号（例如函数），要么产生副作用，但不能两者兼具。*副作用*是执行一段代码的具体结果，例如输出文本、更新全局变量、修改文件内容等等。

尽管函数本身可以产生这样的副作用，但*声明*一个函数（定义函数将做什么）与*调用*一个函数（让函数实际执行那个任务）是不同的。因此，函数应该在一个文件中声明，并在另一个文件中调用。为了遵循这一准则，本章首先介绍如何处理跨多个文件的代码基础知识，然后再转到函数部分。在第九章中，我们将更详细地讨论如何处理文件。

### 将代码分成多个文件

即使我们不考虑将函数声明放在单独文件中的最佳实践，仍然有必要将应用程序的代码拆分到多个文件中。考虑到一个复杂的应用程序可能包含成千上万行代码，如果所有代码都在一个庞大的文本文件中，那么浏览该文件并找到需要处理的特定代码段将非常困难。将代码组织成不同的文件使得项目更易于管理。

使用多个文件还促进了代码的重用性。一旦你开始编写自己的函数，你会发现将这些函数声明在不同的文件中，可以轻松地在项目的不同部分或完全不同的项目中重用这些函数。再举个例子，多个页面的网页应用程序通常在许多页面中包含相同的元素，例如 HTML 头部、底部和导航列表。与其为每个需要这些元素的页面重复写这些代码，不如将通用代码写在一个文件中。这样，如果需要对其进行更改（例如，更新网页徽标的图片引用），你只需在一个地方进行更改，而不是追踪并更新每个重复代码的实例。软件工程师称这一做法为*不要重复自己（DRY）*原则。

一旦你开始将应用程序的代码分布到多个文件中，就需要一种方法从一个文件中访问另一个文件的代码。在本节中，我们将探讨一些使这成为可能的 PHP 语言特性。

#### 读取并执行另一个脚本

PHP 的 require_once 命令可以读取另一个文件的代码并执行它。为了了解这个命令是如何工作的，我们将创建两个脚本。其中一个是主脚本，它将使用 require_once 来访问另一个脚本中的代码。首先，创建一个名为*main.php*的文件，包含 Listing 5-1 中的代码。

```
<?php
print "I'm in main.php\n";

require_once 'file2.php';

print "I'm back in main.php\n";
```

Listing 5-1: 读取并执行来自不同脚本的代码的主脚本

在这个脚本中，我们打印出两条消息，指示我们正在主应用程序文件中。在两条消息之间，我们使用 require_once 命令来读取并执行 *file2.php* 脚本的内容。文件名作为字符串立即出现在命令后面。由于我们没有指定与文件名一起的目录路径（例如，*Users/matt/file2.php*），因此默认理解该文件与当前脚本位于同一文件夹中。这被称为*相对路径*：文件的位置是相对于当前脚本的位置确定的。

现在创建一个名为*file2.php*的文件，包含 Listing 5-2 中的代码。确保将此文件保存在与*main.php*相同的位置。

```
<?php
print "\t I'm printing from file2.php\n";
print "\t I'm also printing from file2.php\n";
```

Listing 5-2: 从另一个脚本读取并执行的 file2.php 内容

这个脚本包含两个打印语句，打印出消息，表示它们来自*file2.php*。请注意，每条消息的开头都有一个制表符转义字符（\t）。这样，这些消息就会缩进，而我们主脚本中打印的消息则不会，这是一个视觉提示，表明这些消息来自不同的脚本。

现在在命令行中输入 php main.php 来运行主脚本。以下是输出结果：

```
I'm in main.php
    I'm printing from file2.php
    I'm also printing from file2.php
I'm back in main.php
```

我们看到主脚本的第一条消息，接着是来自 *file2.php* 的两条缩进消息。这确认了由于主脚本中的 require_once 语句，*file2.php* 的内容已被读取并执行。最后，程序控制流返回主脚本，经过 require_once 语句后，我们看到了主脚本打印的最终消息。

> 注意

*除了* require_once*，PHP 还提供了另外三个命令，用于读取和执行声明在独立文件中的代码：* require*，* include* 和 *include_once*。它们的工作方式类似；你可以在 PHP 文档中阅读它们的区别。在我编写的 99.99% 的 Web 应用程序中，我使用 *require_once*。*

#### 创建绝对文件路径

常量 __DIR__ 始终指向当前执行脚本的 *绝对文件路径*，即从根目录开始的完整文件路径。这是 PHP 的 *魔术常量* 之一，内置常量，其值根据上下文而变化。对于 __DIR__ 来说，值取决于 __DIR__ 被评估时所在文件的位置。

编写 require_once 语句时，最好尽可能使用 __DIR__：只需将 __DIR__ 的值与任何剩余的相对路径信息连接起来，即可访问你试图读取和执行的文件。这可以避免混淆路径是与当前脚本（调用 require_once 命令的脚本）相关，还是与可能已经要求当前脚本的脚本相关。假设你有一系列的脚本，其中一个脚本要求另一个脚本，而那个脚本也要求另一个脚本。如果这些脚本位于不同的目录中，使用 __DIR__ 魔术常量可以确保无论你在何处写 require_once 语句，你都能知道路径正确指向你希望读取和执行的文件。

要尝试使用 __DIR__，请按 列表 5-3 所示更新你的 *main.php* 文件。更改部分以黑色文字显示。

```
<?php
print "I'm in main.php\n";

$callingScriptPath = __DIR__;
print "callingScriptPath = $callingScriptPath\n";

❶ require_once __DIR__ . '/file2.php';

print "I'm back in main.php\n";
```

列表 5-3：使用 __DIR__ 读取和执行不同脚本中的代码的主脚本

我们将 $callingScriptPath 变量赋值为 __DIR__ 魔术常量的值，并打印包含该变量的消息。然后在 require_once 命令之后使用 __DIR__，明确表示 *file2.php* 脚本与此主脚本位于同一目录❶。请注意，我们使用字符串连接运算符（.）将 __DIR__ 的值与字符串 '/file2.php' 结合，构建到另一个文件的绝对路径。以下是运行主脚本后的输出：

```
I'm in main.php
❶ callingScriptPath = /Users/matt/magic
    I'm printing from file2.php
    I'm also printing from file2.php
I'm back in main.php
```

如前所述，首先打印出来自 *main.php* 的消息。然后我们看到打印出的主脚本路径（__DIR__ 的值）❶。对我来说，这是 */Users/matt/magic*，即我电脑上该示例项目所在目录的路径。其余输出与之前相同，包含来自 *file2.php* 的消息，最后是主脚本打印的最终消息。

### 声明和调用一个函数

现在，让我们关注如何声明和使用我们的第一个自定义函数。这个函数将确定两个数字中哪个较小。按照最佳实践，我们将函数声明在一个文件中，*my_functions.php*，然后从另一个文件中调用它，*main.php*。开始一个新项目并创建包含列表 5-4 代码的 *my_functions.php*。

```
<?php
function which_is_smaller(int $n1, int $n2): int
{
    if ($n1 < $n2) {
        return $n1;
    } else {
        return $n2;
    }
}
```

列表 5-4：在 my_functions.php 中声明函数

这里我们声明了一个名为 which_is_smaller() 的函数。我们从关键字 function 开始，后跟函数名称。按照约定，函数名使用蛇形命名法，全部小写字母并用下划线连接多个单词。这使得你可以编写有意义、易读的函数名（尽管遗憾的是，由于语言早期设计时的选择，并不是所有 PHP 内置函数都遵循这种命名约定）。

函数名后面跟着一对括号，括号内是函数的*参数*，它们是函数完成工作的输入。在这个例子中，我们有两个参数，$n1 和 $n2，代表我们希望函数比较的两个数字。每个参数名前面都有数据类型，以确保正确的数据类型传递给函数。例如，int $n1 表示参数 $n1 应该是一个整数。

> 注意

*如果一个函数不需要任何参数，你仍然需要在函数名称后面包括一对空括号。*

括号后面跟着一个冒号（:），然后是函数的返回类型。大多数函数会执行一些操作并产生一个结果值，然后该函数将这个值*返回*给调用它的脚本。*返回类型*指定了该值的数据类型。在这个例子中，函数将返回整数 $n1 或 $n2 中较小的一个，因此我们将返回类型设置为 int。

到目前为止我们写的代码已经定义了函数的*签名*，它是函数的名称、参数（及其类型）和返回类型的组合。PHP 引擎使用函数的签名来唯一标识该函数，识别我们何时调用它，验证传递给函数参数的数据是否合适，并确保函数返回一个合适的值。

接下来是函数的*主体*，它是一个被大括号括起来的语句组，包含每次调用函数时会执行的代码。我们 which_is_smaller()函数的主体由一个 if...else 语句组成，测试整数$n1 是否小于整数$n2。如果$n1 较小，则执行 return $n1;语句。否则（如果$n2 较小或等于$n1），则执行 return $n2;语句。在这两种情况下，我们使用 return 关键字将值（$n1 或$n2）传递给调用它的脚本。一旦函数到达 return 语句，函数会停止执行并将控制权交还给调用脚本。即使函数体内在 return 语句之后有其他语句，它们也不会在函数返回值后执行。

现在我们已经声明了一个函数，接下来让我们使用它。创建与*my_functions.php*位于同一位置的*main.php*文件，并输入清单 5-5 中显示的代码。

```
<?php
require_once __DIR__ . '/my_functions.php';

$result1 = which_is_smaller(5, 2);
print "the smaller of 5 and 2 = $result1\n";

$result2 = which_is_smaller(5, 22);
print "the smaller of 5 and 22 = $result2\n";
```

清单 5-5：从 main.php 调用 which_is_smaller()函数

我们使用 require_once 从*my_functions.php*文件中读取函数声明。这并不会调用函数，它只是使函数在*main.php*脚本中可用。接下来，我们通过编写函数名并在括号中跟上我们希望函数比较的值（5 和 2）来调用函数。这些值被称为*参数*；它们填充函数的参数值。注意，我们将函数调用作为$results1 变量赋值语句的一部分进行调用。这样，函数的返回值将存储在$results1 中，以供以后使用（在这种情况下，是在下一行代码中，那里会将其打印出来）。当一个函数有返回值时，通常会遵循这种调用函数并将结果赋值给变量的模式。

我们通过再次调用该函数来结束脚本，这次使用 5 和 22 作为参数。这就是函数的魅力：你可以根据需要调用它们多次，每次使用不同的输入值。我们将第二次函数调用的返回值存储在$result2 变量中，并再次打印出显示结果的消息。下面是运行*main.php*脚本的输出：

```
the smaller of 5 and 2 = 2
the smaller of 5 and 22 = 5
```

我们可以看到我们的函数工作正常。它返回 5 和 2 中的较小值 2，以及 5 和 22 中的较小值 5。

#### 参数与实参

*参数*和*实参*这两个术语密切相关，经常互相混淆。当你*声明*一个函数时，参数是代表函数将要处理的输入的变量。如你在清单 5-4 中所见，参数会列在函数名称后的括号内。在我们的 which_is_smaller() 函数中，参数是 $n1 和 $n2。每个参数都是一个临时变量，仅在函数内部可见，在函数被调用时会被赋予一个值。这些变量只在函数执行期间存在。一旦函数执行完毕，局部参数变量会从计算机的内存中被丢弃。

变量在软件系统中“存在”的时间长度被称为*作用域*。在函数中声明的任何变量，包括参数，其作用域是局部的，仅限于函数本身。因此，你不能期望从函数声明外的任何代码访问函数的变量。在我们的例子中，我们不能在 *main.php* 中使用变量 $n1 和 $n2。相反，获取函数返回值的方法是使用返回语句。

当我们*调用*一个函数时，实参是我们在函数名称后的括号中传递给函数的具体值。这些实参为函数的参数提供值。例如，当我们调用 which_is_smaller(5, 22) 时，实参 5 被赋值给参数 $n1，实参 22 被赋值给参数 $n2。实参的顺序与参数的顺序相匹配。在这个例子中，实参是字面量，但实参也可以是变量，如下所示：

```
which_is_smaller($applesCount, $orangesCount)
```

就是这么简单。实参是在执行函数时传递的值，参数是在函数执行时创建的局部变量，由接收到的实参填充。因此，在函数执行期间，每个传递给函数的实参都会有一个对应的局部（临时）参数变量。（有一个例外是通过引用传递参数的特殊情况，我们将在本章后面讨论。）

#### 错误来源于不正确的函数调用

在两种常见情况下，调用函数时会发生错误：如果你没有传递正确数量的实参，或者传递了错误数据类型的实参。（有关错误及其他类型警告的信息，请参见第 88 页的“错误、警告和通知”。）考虑调用我们自定义的 which_is_smaller() 函数：

```
$result = which_is_smaller(3);
```

该函数需要两个整数类型的实参，但我们只提供了一个。如果你尝试执行这个表达式，应用程序将停止运行，你会看到类似以下的致命错误：

```
PHP Fatal error:  Uncaught ArgumentCountError: Too few arguments to function
which_is_smaller(), 1 passed in /Users/matt/main.php on line 9 and exactly 2
expected in /Users/matt/my_functions.php:2
```

如果你传递了错误数据类型的实参（即不能转换为函数声明中指定的参数数据类型的值），你也会遇到致命错误。考虑下面这个表达式，我们将非数字的字符串传递给 which_is_smaller() 函数：

```
$result = which_is_smaller('mouse', 'lion');
```

尝试执行该语句会产生类似于以下的错误消息：

```
PHP Fatal error:  Uncaught TypeError: which_is_smaller(): Argument #1 ($n1)
must be of type int, string given, called in /Users/matt/main.php on line 10
and defined in /Users/matt/my_functions.php:2
```

发生了一个致命的类型错误（TypeError），因为我们的函数需要两个整数参数，但我们提供了字符串类型的参数。

#### 类型转换

为了避免我们刚刚看到的那种类型错误（TypeError），当传入错误类型的参数时，PHP 引擎会尝试将这些参数转换成期望的数据类型。（有关类型转换的回顾，请参阅第二章。）清单 5-6 展示了在我们向`which_is_smaller()`函数传入非整数类型参数时的一些示例。更新你的*main.php*文件，使其与清单一致。

```
<?php
require_once __DIR__ . '/my_functions.php';

$result1 = which_is_smaller(3.5, 2);
print "the smaller of 3.5 and 2 = $result1\n";

$result2 = which_is_smaller(3, '55');
print "the smaller of 3 and '55' = $result2\n";

$result3 = which_is_smaller(false, -8);
print "the smaller of false and -8 = $result3\n";
```

清单 5-6：更新 main.php 脚本以演示类型转换

我们调用`which_is_smaller()`函数三次并打印结果。由于所有参数都可以转换为整数，因此这些函数调用都不会触发错误。首先，我们使用浮动类型 3.5 和整数 2 调用函数。浮动类型将转换为整数 3。接下来，我们使用整数 3 和字符串'55'作为参数。这一次，字符串将转换为整数 55。最后，我们传递布尔值 false 和整数-8 作为参数。false 将转换为整数 0。以下是运行脚本后的输出结果：

```
PHP Deprecated:  Implicit conversion from float 3.5 to int loses precision in
/Users/matt/my_functions.php on line 2

the smaller of 3.5 and 2 = 2
the smaller of 3 and '55' = 3
the smaller of false and -8 = -8
```

当你运行脚本时，首先打印出来的应该是一个废弃警告消息，告知你当浮动类型 3.5 转换为整数 3 时，精度会丢失。此消息表示，在未来的某个时刻（可能是 PHP 9），PHP 将停止自动将带有小数部分的浮动类型转换为整数，因此代码有朝一日将停止工作并触发错误。在该消息之后，你应该能看到三次打印语句的结果，表明由于 PHP 的自动类型转换，这三次函数调用都正常执行了。

> 注意

*当你遇到废弃警告消息时，阅读有关即将更改的讨论可能会很有帮助。例如，解释清单 5-6 中废弃消息输出的请求评论（RFC）文档可以在线查看，链接为* [`wiki.php.net/rfc/implicit-float-int-deprecate`](https://wiki.php.net/rfc/implicit-float-int-deprecate)*。*

尽管这些函数调用在参数类型不正确的情况下仍然成功，但编写良好的程序应尽量避免依赖类型转换。注意类似我们刚刚遇到的废弃警告，并寻找方法修改代码，以便在没有警告或错误的情况下处理不同类型的值。在这种情况下，我们可以重构该函数，使用联合类型（在第 98 页的《联合类型》中讨论），这样就可以同时接受整数和浮动类型作为参数。

### 没有显式返回值的函数

不是每个函数都必须显式返回一个值。例如，你可以编写一个函数，仅仅打印一条消息而不返回任何内容给调用脚本。当一个函数没有显式的返回值时，应将其返回类型声明为 void。

为了演示，我们将声明一个函数，该函数打印出给定数量的星号，并在两侧用另一个填充字符进行填充，以实现固定的行长度。我们可以使用该函数创建 ASCII 艺术图像，即通过排列字符文本来形成的图像。启动一个新项目，并创建包含 列表 5-7 中代码的 *my_functions.php* 文件。

```
<?php
function print_stars(int $numStars, string $spacer): void
{
    $lineLength = 20;
    $starsString = str_repeat('*', $numStars);
    $centeredStars = str_pad($starsString, $lineLength, $spacer, STR_PAD_BOTH);
    print $centeredStars . "\n";
}
```

列表 5-7：在 my_functions.php 中声明 print_stars() 函数

在这里，我们声明了一个名为 print_stars() 的函数。该函数需要两个参数：$numStars 和 $spacer。整数 $numStars 是要打印的星号（*字符）数量。字符串 $spacer 是在星号两边作为填充的字符。在括号后面，我们使用 : void 来指示该函数不会显式返回任何值。

在函数体内，我们将要打印的行长度设置为 20 个字符。（由于这个值是*硬编码*在函数中的，所以每次调用该函数时，它的值都将相同；一个更灵活的替代方法是将 $lineLength 设置为一个参数。）然后，我们生成一个包含由 $numStars 参数指定数量的星号的字符串（$starsString）。接着，我们使用内置的 str_pad() 函数（在第三章中讨论）来创建一个 20 个字符长的字符串，$starsString 在其中居中，并且两侧对称地用 $spacer 参数中的字符进行填充。例如，如果 $numStars 为 10，$spacer 为 '.'，则会生成字符串 '.....**********.....'，即 10 个星号，两侧各有 5 个句点，总长度为 20。最后，我们打印出结果，并输出一个换行符。

请注意，我们没有在函数体内包含 return 语句。因为没有必要，函数的作用仅仅是构造并打印一个字符串。如果我们尝试从该函数返回一个值，会触发一个致命错误，因为我们将该函数声明为 void。

现在让我们使用我们的函数生成一个树的 ASCII 艺术图像。创建 *main.php* 文件，包含 列表 5-8 中的代码。

```
<?php
require_once __DIR__ . '/my_functions.php';

❶ $spacer = '/';
print_stars(1, $spacer);
print_stars(5, $spacer);
print_stars(9, $spacer);
print_stars(13, $spacer);
print_stars(1, $spacer);
print_stars(1, $spacer);
```

列表 5-8：在 main.php 中使用 print_stars() 函数生成树形图案的脚本

在通过 require_once 引入函数声明后，我们将填充字符设置为正斜杠 (/) ❶。然后我们调用 print_stars() 函数六次，打印出由 1、5、9 和 13 个星号组成的树形图案，并且再加上两行只有 1 个星号的树干。以下是在终端运行 *main.php* 脚本的输出：

```
/////////*//////////
///////*****////////
/////*********//////
///*************////
/////////*//////////
/////////*//////////
```

我们在暴风雨中创建了一棵树！

#### 返回 NULL

即使函数声明为 `void`，它在技术上仍然有一个返回值：NULL。如果一个函数执行完毕没有返回值，函数会默认返回 NULL。为了证明这一点，我们可以再次调用 `print_stars()` 函数，并像处理有返回值的函数一样，将结果赋给一个变量。更新你的 *main.php* 文件以匹配 Listing 5-9。更改部分用黑色文本显示。

```
<?php
require_once __DIR__ . '/my_functions.php';

$spacer = '/';
print_stars(1, $spacer);
print_stars(5, $spacer);
print_stars(9, $spacer);
print_stars(13, $spacer);
print_stars(1, $spacer);
$result = print_stars(1, $spacer);

var_dump($result);
```

Listing 5-9: 更新 *main.php* 来存储并打印 `print_tree()` 函数的 NULL 返回值

我们像之前一样调用 `print_stars()` 函数，但这次我们将最后一次函数调用的返回值存储在 `$result` 变量中。然后使用 `var_dump()` 查看 `$result` 的内容。由于 `print_stars()` 没有显式的返回值，因此 `$result` 应该包含 NULL。以下是运行 *main.php* 脚本的输出：

```
/////////*//////////
///////*****////////
/////*********//////
///*************////
/////////*//////////
/////////*//////////
NULL
```

我们可以再次看到 ASCII 树，随后是从调用 `var_dump()` 得到的 NULL。这证明了尽管函数声明为 `void`，它仍然默认返回 NULL。

#### 提前退出函数

声明为 `void` 的函数仍然可以使用 `return` 语句，只要该语句不包含值。如前所述，函数在遇到 `return` 语句时会立即停止执行，因此不带值的 `return` 提供了一种提前退出函数的机制。这在例如函数的某个参数出现问题时非常有用。你可以在函数开始时添加验证逻辑来检查参数，并使用 `return` 来提前停止函数的执行，如果一个或多个参数值不符合预期，则会恢复主调用脚本的执行。

我们一直在使用的 `str_pad()` 函数，如果填充字符串为空，会触发致命错误。为了避免程序崩溃，我们将更新 `print_stars()` 函数，首先检查 `$spacer` 字符串参数是否为空。如果为空，我们将使用 `return` 提前退出函数。修改 *my_functions.php* 以匹配 Listing 5-10。

```
<?php
function print_stars(int $numStars, string $spacer): void
{
if (empty($spacer)) {
    return;
    }
 $lineLength = 20;
 $starsString = str_repeat('*', $numStars);

 $centeredStars = str_pad($starsString, $lineLength, $spacer, STR_PAD_BOTH);
 print $centeredStars . "\n";
}
```

Listing 5-10: 向 `print_stars()` 函数中添加 `return` 语句以提前退出

我们在函数体开始时添加了一个 `if` 语句，使用内建的 `empty()` 函数来检查 `$spacer` 是否为空字符串。如果为空，我们使用不带值的 `return` 来提前结束函数执行，并将程序控制返回给调用脚本。如果函数执行通过了这个 `if` 语句，则表示 `$spacer` 不为空，这样我们的 `str_pad()` 调用应该能正常工作。

为了查看 `return` 语句是否有效，更新 *main.php* 脚本，如 Listing 5-11 所示。

```
<?php
require_once __DIR__ . '/my_functions.php';

$spacer = '';
print_stars(1, $spacer);
print_stars(5, $spacer);
print_stars(9, $spacer);
print_stars(13, $spacer);
print_stars(1, $spacer);
$result = print_stars(1, $spacer);

var_dump($result);
```

Listing 5-11: 更新 *main.php* 来调用 `print_tree()` 并传入一个空的填充字符串

我们将$spacer 设置为空字符串，而不是斜杠，之后再调用 print_stars()。运行主脚本的输出现在应该只是 NULL。每次调用 print_stars()函数时，它都会提前返回，因为$spacer 是空字符串，因此我们不再看到 ASCII 树形图。另一方面，我们也没有看到致命错误，因为返回语句阻止我们使用无效的参数调用 str_pad()。我们依然在输出中看到 NULL，这是 var_dump()调用的结果。这表明，当函数遇到没有返回值的返回语句时，它会返回 NULL，就像没有返回语句一样。

### 在函数内部调用函数

在一个函数的主体内调用另一个函数是完全合理的。事实上，我们已经多次这样做了，在 print_stars()函数内部调用了内置的 PHP 函数，如 str_repeat()和 str_pad()。同样，在其他自定义函数中调用你自己的自定义函数也是可能的，实际上这是非常常见的做法。

编程的强大之处在于将问题分解为更小的任务。你编写基本的函数来处理这些小任务，然后再编写更高层次的函数，将这些任务组合在一起解决更大的问题。最终，你的主应用脚本看起来非常简单：你只需要调用一到两个函数。诀窍在于，这些函数本身会调用多个其他函数，以此类推。

我们调用 print_stars()函数六次来生成一个 ASCII 树。让我们将这六次调用移到另一个函数 print_tree()中。这样，每次我们想打印一棵树时，主脚本中只需要一次函数调用。将新的 print_tree()函数添加到*my_functions.php*中，如 Listing 5-12 所示。

```
<?php
function print_stars(int $numStars, string $spacer): void
{
--snip--
}

function print_tree(string $spacer): void
{
    print_stars(1, $spacer);
    print_stars(5, $spacer);
    print_stars(9, $spacer);
    print_stars(13, $spacer);
    print_stars(1, $spacer);
    print_stars(1, $spacer);
}
```

Listing 5-12：将 print_tree()函数添加到 my_functions.php

我们在之前声明的 print_stars()函数后声明 print_tree()函数。它需要一个名为$spacer 的字符串参数。在函数体内，我们编写了六个原始的 print_stars()调用。请注意，print_tree()函数的参数$spacer 在调用 print_stars()时也充当了一个参数。这样，我们只需在调用 print_tree()时传入不同的字符串，就可以轻松地打印带有不同填充字符的星号树形图。

有了这个新函数，我们现在可以大大简化我们的主脚本。按照 Listing 5-13 所示更新*main.php*。

```
<?php
require_once __DIR__ . '/my_functions.php';
print_tree('/');
print_tree(' ');
```

Listing 5-13：通过 print_tree()函数简化 main.php 脚本

在读取函数声明文件后，我们调用 print_tree()两次生成两棵树。第一次我们像之前一样使用正斜杠作为间隔符，第二次我们使用空格字符。以下是结果：

```
/////////*//////////
///////*****////////
/////*********//////
///*************////
/////////*//////////
/////////*//////////
         *
       *****
     *********
   *************
         *
         *
```

我们的主脚本通过调用两次`print_tree()`，完成了原本需要调用 12 次`print_stars()`的任务。当然，那些`print_stars()`的调用仍然存在，但我们将它们隐藏在`print_tree()`的定义中，使我们的主脚本变得更加简洁。你可以开始看到函数在组织代码和促进可重用性方面的强大作用。

### 返回多个值和参数类型的函数

对于简单的情况，你通常可以编写一个执行某项任务并返回单一类型值或不返回任何值的函数。然而，其他时候，你可能希望通过允许函数根据情况返回不同数据类型的值来提高其可重用性。同样，你可能希望函数的参数能够接受不同数据类型的值，以确保代码能够应对输入验证问题。*可空类型*和*联合类型*提供了优雅的方法来允许多种类型，既适用于函数的返回值，也适用于函数的参数。

#### 可空类型

编写通常返回一种类型值（如字符串或数字），但有时返回`NULL`的函数是非常常见的。例如，一个通常执行计算的函数，如果接收到无效输入，可能会返回`NULL`；或者一个从数据库中检索信息的函数，如果无法建立数据库连接，也可能返回`NULL`（我们将在第六部分讨论数据库时看到这一点）。为了实现这一点，可以通过在返回类型前立即添加问号（?）来声明函数的返回类型为*可空*。例如，在函数声明的第一行末尾添加`: ?int`意味着该函数将返回`NULL`或整数。

让我们通过一个尝试返回拼写出来的数字的整数值的函数来看看它是如何工作的（例如返回 1 而不是'one'）。如果函数无法识别输入的字符串，它将返回`NULL`。开始一个新项目，创建包含列表 5-14 内容的`my_functions.php`文件。

```
<?php
function string_to_int(string $numberString): ❶ ?int
{
    return match ($numberString) {
        'one' => 1,
        'two' => 2,
        'three' => 3,
        'four' => 4,
        'five' => 5,
      ❷ default => NULL
    };
}
```

列表 5-14：一个返回整数或`NULL`的函数

我们声明了`string_to_int()`函数，使用可空类型`?int`来表示该函数将返回`NULL`或整数 ❶。该函数接收字符串参数`$numberString`。它的主体是一个单一的返回语句，通过使用`match`表达式选择要返回的值。这是可能的，因为`match`表达式的值总是一个单一的结果。该表达式有五个子句，将字符串'one'到'five'分别匹配到相应的整数。第六个子句设置了默认情况 ❷，如果提供任何其他字符串，则返回`NULL`。通过这种方式，`match`表达式返回一个整数或`NULL`，就如同函数的可空返回类型所指示的那样。

现在我们将编写一个 *main.php* 文件，其中包含一个调用我们函数的脚本。当你调用具有可空返回类型的函数时，测试返回值是否为 NULL 是很重要的。清单 5-15 显示了如何操作。

```
<?php
require_once __DIR__ . '/my_functions.php';

❶ $text1 = 'three';
$number1 = string_to_int($text1);
❷ if (is_null($number1)) {
    print "sorry, could not convert '$text1' to an integer\n";
} else {
    print "'$text1' as an integer = $number1\n";
}

$text2 = 'onee';
$number2 = string_to_int($text2);
if (is_null($number2)) {
    print "sorry, could not convert '$text2' to an integer\n";
} else {
    print "'$text2' as an integer = $number2\n";
}
```

清单 5-15：一个调用可空类型 string_to_int() 函数的 main.php 脚本

我们将字符串 'three' 赋值给变量 $text1，然后将该变量传递给 string_to_int() 函数，将返回值存储在 $number1 ❶ 中。接下来，我们使用 if...else 语句测试 $number1 中的值是否为空（NULL） ❷。如果是，我们打印一条消息，说明该字符串无法转换为整数。否则，我们打印一条显示字符串及其对应整数的消息。然后，我们重复这个过程，使用字符串 'onee'。以下是输出：

```
'three' as an integer = 3
sorry, could not convert 'onee' to an integer
```

我们可以看到，当参数是字符串 'three' 时，函数返回整数 3，但当参数是拼写错误的字符串 'onee' 时，它返回 NULL。通过将 string_to_int() 函数声明为可空返回类型，我们可以灵活地以有意义的方式应对这种问题输入。

就像函数可以有可空返回类型一样，你也可以使用相同的问号语法来声明函数参数为可空类型，这意味着参数可以是 NULL 或其他类型。例如，参数列表 (?string $name) 意味着该函数接受一个 $name 参数，该参数可以是 NULL 或字符串。

我们不需要像在 清单 5-15 中那样，每次调用 string_to_int() 函数时都重复编写 if...else 语句。我们可以将函数的 NULL 或整数返回值作为参数传递给另一个函数，以生成适当的消息。因此，该函数需要能够接受一个可能为 NULL 或整数的参数。清单 5-16 显示了这样一个名为 int_to_message() 的函数。将该函数添加到你的 *my_functions.php* 文件的末尾。

```
function int_to_message(?int $number): string
{
    if (is_null($number)) {
        return "sorry, could not convert string to an integer\n";
    } else {
        return "an integer = $number\n";
    }
}
```

清单 5-16：一个具有可空类型的 $number 参数的函数

该函数的签名包含一个名为 $number 的单一参数，其类型为可空的 ?int。这意味着传递给该函数的参数可以是 NULL 或整数。函数体使用了我们在 *main.php* 脚本中写的 if...else 语句，根据传递的数据类型返回相应的消息。

现在，通过移除重复的 if...else 语句并改为调用我们的新函数，我们可以大大简化主脚本。清单 5-17 显示了更新后的脚本。

```
<?php
require_once __DIR__ . '/my_functions.php';

❶ $text1 = 'three';
❷ $number1 = string_to_int($text1);
❸ print int_to_message($number1);

$text2 = 'onee';
$number2 = string_to_int($text2);
print int_to_message($number2);

❹ print int_to_message(string_to_int('four'));
```

清单 5-17：使用 int_to_message() 函数简化 main.php

请注意，由于生成消息的逻辑已经移到函数中，我们的主脚本现在变得更简洁了。对于每个输入，我们遵循三个基本语句的模式：声明一个字符串❶，存储调用`string_to_int()`函数时返回的整数（或 NULL）❷，并打印通过将该整数或 NULL 值传递给`int_to_message()`函数而返回的字符串❸。

如果我们真的想让代码更加简洁，可以将这三条语句合并成一行❹，在调用`int_to_message()`函数时，将`string_to_int()`函数放在括号内。这样，前者的返回值就会直接作为参数传递给后者，而无需使用中介变量。这种做法属于编程风格的选择。就我个人而言，我更倾向于使用中介变量，以防止一行代码过于复杂。

#### 联合类型

如果你希望一个函数能够返回多种数据类型，可以使用*联合类型*来声明其返回值。这是一个值的可能数据类型的列表，类型之间用竖线分隔。例如，`int|float`表示一个值可以是整数或浮动值。联合类型既可以应用于函数参数，也可以应用于返回值。

可空类型本质上是联合类型的一种特殊类别，其问号语法提供了当某种可能的数据类型为 NULL 时的简便写法。例如，联合类型`string|NULL`与更简洁的可空类型`?string`相同。联合类型在代码中有多个非 NULL 类型时特别有用，比如`int|float`，或者有多个非 NULL 类型加上 NULL 时，比如`string|int|NULL`，表示数据类型可能是字符串、整数或 NULL。使用可空类型语法无法表达这种情况，因为你不能像写`?string|int`那样在联合类型中混合可空类型。你也不能在联合类型中包含`void`类型。

为了演示联合类型，我们将`string_to_int()`函数修改为`string_to_number()`函数，该函数根据传入的字符串返回整数、浮动值或 NULL。我们还将`int_to_message()`函数更新为`number_to_message()`，该函数可以接受整数、浮动值或 NULL 作为参数。更新*my_functions.php*以匹配示例 5-18。

```
<?php
function string_to_number(string $numberString):❶int|float|NULL
{
 return match ($numberString) {
    ❷'half' => 0.5,
 'one' => 1,
 'two' => 2,
 'three' => 3,
 'four' => 4,
 'five' => 5,
 default => NULL
 };
}

function number_to_message(string $text, ❸ int|float|NULL $number): string
{
❹if (is_int($number)) {
        return "'$text' as an integer = $number\n";
    }

❺if (is_float($number)) {
        return "'$text' as a float = $number\n";
    }

  ❻ return "sorry, could not convert '$text' to a number\n";
}
```

示例 5-18：使用联合类型作为函数的返回值和参数

首先，我们声明`string_to_number()`，它是我们`string_to_int()`函数的修订版。我们使用联合类型`int|float|null`来表示该函数将返回一个整数、浮动值或 NULL❶。就像以前的`string_to_int()`一样，这个函数接受一个字符串参数。我们在函数体的`match`语句中增加了一个新的条件，将字符串'half'匹配为浮动值 0.5❷，因此需要使用联合类型。

接下来，我们声明 number_to_message()，它是 int_to_message() 的修订版本，返回一个字符串。这个函数接受两个参数。第一个参数，字符串 $text，将与传递给我们的 string_to_number() 函数的字符串相同。第二个参数，$number，将是该函数的返回值，因此它可能是一个整数、一个浮点数或 NULL。因此，我们对参数 ❸ 使用相同的 int|float|NULL 联合类型。

在函数体中，我们首先测试 $number 是否包含一个整数值 ❹，如果是，我们返回一条消息，说明 $text 是一个整数。接下来，我们测试 $number 是否包含一个浮点数值 ❺，如果是，则返回一条适当的消息。最后，我们返回一条消息，说明 $text 无法转换为数字 ❻。如果之前的任何一个 return 语句被执行，执行就不会到达这一步，所以我们知道此时 $number 既不是整数也不是浮点数。因此，我们不需要将这个最终的 return 语句放在一个 else 子句或另一个 if 语句中，尽管我们可以这样做。

这种选择是个人的编程风格问题。我喜欢像这样用一个无条件的 return 语句来结束函数，这样我可以清楚地看到要返回的默认值。然而，一些程序员更喜欢用一个 else 子句来结束最后一个 if 语句，以此来传达默认值。无论哪种方式，执行结果都是一样的。

现在让我们测试一下我们的函数。更新你的 *main.php* 脚本以匹配 清单 5-19。

```
<?php
require_once __DIR__ . '/my_functions.php';

$text1 = 'three';
$number1 = string_to_number($text1);
print number_to_message($text1, $number1);
```

清单 5-19：在 main.php 中使用联合类型参数和返回值调用函数

我们调用 string_to_number() 函数，传入字符串 'three'，并将结果存储在 $number1 变量中。然后我们将 $number1 传递给我们的 number_to_message() 函数，并打印它返回的消息。这段代码应该输出消息 'three' as an integer = 3。

### 可选参数

如果一个参数的值在每次调用函数时通常都是相同的，你可以在声明函数时为该参数设置一个默认值。实际上，这使得该参数成为可选的。只有当你知道你希望该值与默认值不同时，你才需要包含一个与该参数对应的参数。

PHP 的许多内置函数都有带有默认值的可选参数。例如，PHP 的 number_format() 函数，它接受一个浮点数并将其转换为字符串，有几个可选参数控制字符串的格式。在命令行输入 php -a 以在交互模式下尝试以下代码：

```
❶ php > **print number_format(1.2345);**
1
❷ php > **print number_format(1.2345, 2);**
1.23
❸ php > **print number_format(1.2345, 1, ',');**
1,2
```

number_format()函数的第一个参数是必需的，它是我们想要格式化的浮动数值。默认情况下，仅传递一个参数❶时，函数会返回去掉小数部分的数字字符串。当我们添加一个整数作为可选的第二个参数❷时，函数使用该整数来设置保留的小数位数。我们使用了 2 这个值来保留两位小数。默认情况下，小数分隔符使用句点，但如果我们添加一个字符串作为可选的第三个参数❸，则函数会使用该字符串作为小数分隔符。在这种情况下，我们使用逗号，这是欧洲大陆常见的小数分隔符。

Listing 5-20 显示了 number_format()函数的签名，取自 PHP 在线文档，用于说明如何声明参数的默认值。

```
number_format(
    float $num,
    int $decimals = 0,
    ?string $decimal_separator = ".",
    ?string $thousands_separator = ","
): string
```

Listing 5-20: 内置的 number_format()函数，包括具有默认值的可选参数

首先，请注意，当你有一个长参数列表时，可以将它们分散在多行中，以使代码更具可读性。该函数最多接受四个参数，但第二、第三和第四个参数都使用赋值运算符（=）在参数名后面赋予了默认值。例如，第二个参数$decimals 的默认值为 0，因此当我们调用 number_format(1.2345)而没有提供第二个参数时，函数将使用$decimals 的默认值，并将数字格式化为没有小数位。同样，$decimal_separator 参数的默认值为句点，而$thousands_separator 参数的默认值为逗号。

参数声明的顺序很重要。所有必填参数（没有默认值的参数）必须先列出，后跟可选参数。这是因为调用函数时，参数的顺序必须与声明的顺序匹配。如果可选参数在必填参数之前，而你又省略了可选参数，那么就无法知道你的第一个参数是对应于第二个参数的。此规则的唯一例外是当你使用命名参数时，正如我们稍后在本章讨论的那样。

现在我们已经了解了可选参数的工作方式，让我们为一个自定义函数添加一个可选参数。我们将重新回顾本章前面提到的 which_is_smaller()函数，并添加一个可选参数，用于控制当传入的比较值相等时函数的行为。返回到该项目的*my_functions.php*文件，并更新脚本以匹配 Listing 5-21。

```
<?php
function which_is_smaller(int $n1, int $n2, ❶ bool $nullIfSame = false): ?int
{
 if ($n1 < $n2) {
 return $n1;
 }

    if ($n2 < $n1) {
        return $n2;
    }

 ❷ if ($nullIfSame) {
        return NULL;
    }

  ❸ return $n1;
}
```

Listing 5-21: 更新 which_is_smaller()函数以包含一个可选参数

我们向函数中添加了第三个参数，布尔值$nullIfSame，并给它设置了默认值 false ❶。由于这个默认值，当$n1 和$n2 被发现相等时，函数通常会返回$n1 ❸。然而，如果用户在调用函数时传递 true 作为第三个参数来覆盖这个默认值，则会返回 NULL ❷。为了考虑这种情况，我们使用可空类型?int 来设置函数的返回类型。

这里 if 和 return 语句的顺序非常重要。只有当$n1 和$n2 相等时，代码才会进入 if ($nullIfSame) ❷。由于$nullIfSame 默认是 false，因此这个条件通常会失败，所以最终会执行 return $n1; ❸。只有当用户将$nullIfSame 设置为 true 时，函数才会返回 NULL。

更新项目的*main.php*文件，如清单 5-22 所示，以测试该功能。

```
<?php
require_once __DIR__ . '/my_functions.php';

$result1 = which_is_smaller(1, 1);
var_dump($result1);
$result2 = which_is_smaller(1, 1, true);
var_dump($result2);
```

清单 5-22：在 main.php 中调用 which_is_smaller()，带有和不带有可选参数

我们调用 which_is_smaller()两次，使用 var_dump()显示结果。第一次我们传入 1 和 1，并省略可选参数，因此$nullIfSame 将默认是 false。第二次，我们添加了 true 作为第三个参数，覆盖了默认值。以下是运行主脚本时的输出：

```
int(1)
NULL
```

第一行表示函数遵循了默认行为，当我们省略可选参数时，返回 1（第一个参数的值）。然而，当我们使用第三个参数将$nullIfSame 设置为 true 时，函数返回 NULL。

#### 位置参数与命名参数

当你调用一个函数时，PHP 引擎默认会*按位置*解释参数，依据它们的顺序将其与函数的参数匹配。然而，你也可以通过使用*命名参数*来调用函数：你显式地将参数的值与相应参数的名称配对。在这种情况下，参数的顺序就不再重要。命名参数在函数有可选参数时特别有用。

要使用命名参数而非位置参数，你不需要以任何方式修改函数声明，尽管此时函数参数的名称变得更加重要。你需要做的只是，在调用函数时，在括号内包含参数名称（去掉美元符号），然后加上冒号(:)和所需的参数值。例如，要在调用 which_is_smaller()函数时使用命名参数将 true 作为$nullIfSame 参数的值传入，你应在参数列表中加入 nullIfSame: true。约定是在冒号后加一个空格。

清单 5-23 显示了更新后的*main.php*文件，增加了一个使用命名参数调用 which_is_smaller()的额外实例。

```
<?php
require_once __DIR__ . '/my_functions.php';

$result1 = which_is_smaller(1, 1);
var_dump($result1);
$result2 = which_is_smaller(1, 1, true);
var_dump($result2);
❶ $result3 = which_is_smaller(nullIfSame: true, n1: 1, n2: 1);
var_dump($result3);
```

清单 5-23：使用位置参数和命名参数调用 which_is_smaller()

新的 which_is_smaller() ❶调用在功能上与之前的调用等效，但我们使用了命名参数。因此，我们能够按不同于参数声明顺序的顺序列出参数：首先是$nullIfSame，然后是$n1，最后是$n2。以下是结果：

```
int(1)
NULL
NULL
```

输出的最后两行都是 NULL，表示最后两个函数调用通过位置参数和命名参数达成了相同的结果。

在这个例子中，每个函数调用要么完全使用位置参数，要么完全使用命名参数，但你也可以在同一次函数调用中混合使用两种参数方式。在这种情况下，位置参数必须排在前面，顺序与函数声明中的顺序一致，然后是你选择的顺序排列的命名参数。考虑以下例子：

```
$result = which_is_smaller(5, nullIfSame: true, n2: 5);
```

这里，第一个参数 5 没有名称。因此，PHP 会按照位置来处理它，并将其匹配到声明的第一个参数$n1。剩下的参数是有名称的，因此可以按任意顺序出现。相比之下，这是另一个调用该函数的例子：

```
$result = which_is_smaller(nullIfSame: true, 5, n2: 5);
```

这次我们首先用了命名参数$nullIfSame。然后我们使用了一个没有名称的参数 5，可能是用来传递$n1 参数的。然而，由于我们一开始就用了命名参数，PHP 引擎无法识别这一点，因此这个函数调用会触发一个错误。

#### 跳过的参数

当一个函数有多个可选参数时，你可以使用命名参数仅设置你想要的可选参数，同时跳过其余的参数。这之所以有效，是因为命名参数让你不必遵循参数的顺序。任何跳过的参数将使用默认值。为了说明这一点，我们来创建一个打印自定义问候语的函数。创建一个新项目，并创建*my_functions.php*，使其符合列表 5-24。

```
<?php
function greet(
    string $name,
    string $greeting = 'Good morning',
    bool $hasPhD = false
): void
{
    if ($hasPhD) {
      ❶ print "$greeting, Dr. $name\n";
    } else {
        print "$greeting, $name\n";
    }
}
```

列表 5-24：一个带有两个可选参数的 greet()函数

我们将 greet()函数声明为 void，因为它输出一条消息，但不返回任何值。该函数有一个必需的字符串参数$name，以及两个带默认值的可选参数$greeting 和$hasPhD。函数体是一个 if 语句，它输出$greeting 和$name 的值，如果$hasPhD 参数为真，则在两者之间插入标题 Dr. ❶。

现在我们来看看几种调用 greet()函数的方法。创建一个包含列表 5-25 中代码的*main.php*。

```
<?php
require_once __DIR__ . '/my_functions.php';

greet('Matt');
greet('Matt', hasPhD: true);
```

列表 5-25：一个主脚本调用 greet()，并跳过参数

第一次调用 greet()时，我们只传递字符串'Matt'作为参数。我们没有使用命名参数，因此这个参数会按位置匹配到$name 参数。其他参数会使用默认值，结果输出的消息是“Good morning, Matt”。

第二次调用 greet()时，我们使用位置参数'Matt'和命名参数 hasPhD: true。请注意，`$hasPhD`是函数声明中的第三个参数；我们跳过了第二个参数！这是完全没问题的。我们跳过的参数`$message`有一个默认值，感谢我们使用命名参数，PHP 引擎会清楚地知道哪些提供的参数与哪些函数参数匹配。最终我们应该得到消息：Good morning, Dr. Matt。

这是运行*main.php*脚本的输出：

```
Good morning, Matt
Good morning, Dr. Matt
```

输出正如我们所预期的那样。由于默认参数值和命名参数的结合，我们能够顺利跳过`$message`参数。

### 值传递与引用传递

默认情况下，PHP 函数使用*值传递*的方法将参数与参数匹配：参数的值会被复制并赋值（传递）给适当的参数，这些参数在函数的作用域内作为临时变量存在。通过这种方式，如果在函数执行过程中修改了任何参数的值，这些更改将不会对函数外部的任何值产生影响。毕竟，函数是处理原始值的副本。

另一种方法是*引用传递*：函数参数传递的是指向原始变量本身的引用，而不是副本。通过这种方式，如果一个变量作为参数传递给函数，函数可以永久改变该变量的值。为了指示引用传递参数，在声明函数时，请在参数名之前立即放置一个“&”符号。

我通常不推荐使用引用传递参数；事实上，在过去 20 年里，我无法想到任何我写过的引用传递参数。允许函数修改传递给它们的变量会使程序变得更加复杂，从而更难理解、测试和调试。尽管如此，熟悉这一概念仍然很重要，因为你可能会在别人编写的代码中遇到引用传递参数，包括在你可能希望在自己项目中使用的第三方库中。不了解如何使用引用传递参数就调用函数，可能会导致意外的结果。

> 注意

*在某些编程语言中，程序员使用多个引用传递参数，作为函数“返回”多个值的一种方式，而无需使用*return*语句。然而，在现代 PHP 中有更好的方法，例如返回一个数组（参见第七章）或一个对象（参见第 V 部分）。*

为了说明值传递和引用传递参数之间的区别，并展示为什么后者通常最好避免，我们将创建一个计算某人未来年龄的函数的两个版本。开始一个新项目并创建包含清单 5-26 内容的*my_functions.php*。

```
<?php
function future_age (int $age): void
{
    $age = $age + 1;
    print "You will be $age years old on your next birthday.\n";
}
```

清单 5-26：按值传递的 future_age()版本

在这里，我们声明了一个名为 future_age()的函数。它具有一个整数参数$age，按通常方式声明，因此这是一个正常的值传递参数。由于不需要返回任何值，函数被声明为 void。在函数体内，我们将$age 加 1，并打印出包含结果的消息。

现在在*main.php*中创建一个主脚本，包含清单 5-27 中显示的代码。

```
<?php
require_once __DIR__ . '/my_functions.php';

$currentAge = 20;
print "You are $currentAge years old.\n";
future_age($currentAge);
print "You are $currentAge years old.\n";
```

清单 5-27：测试按值传递的 future_age()版本

我们为$currentAge 变量赋值为整数 20。然后我们打印出显示该变量值的消息。接着，我们调用我们的 future_age()函数，并将$currentAge 作为参数传递。然后我们再打印出另一条消息，显示变量的值。这让我们可以查看函数调用前后$currentAge 的值。以下是结果：

```
You are 20 years old.
You will be 21 years old on your next birthday.
You are 20 years old.
```

输出的第一行和最后一行是相同的，表明调用 future_age()对$currentAge 变量的值没有影响。实际上，当函数被调用时，会在函数的作用域内创建一个局部变量$age，并将$currentAge 的值复制到其中。这样，当函数将$age 加 1 时，它是在不改变$currentAge 值的情况下进行的。这就是值传递参数的工作方式：它们不会对函数外部的作用域产生任何影响。

现在让我们修改我们的 future_age()函数，改为使用按引用传递的参数，看看这会有什么不同。更新你的*my_functions.php*文件，如清单 5-28 所示。

```
<?php
function future_age (int &$age): void
{
 $age = $age + 1;
 print "You will be $age years old on your next birthday.\n";
}
```

清单 5-28：按引用传递的 future_age()版本

这里唯一的变化是，在参数名称前加上了一个&符号，表示$age 是按引用传递的参数。因此，$age 将不再是一个局部变量，包含调用函数时传入参数的值的副本。相反，$age 将成为该变量的引用，因此对$age 所做的任何更改也将反映到该变量上。为了验证这一点，再次运行你的*main.php*脚本。这次你应该会看到以下输出：

```
You are 20 years old.
You will be 21 years old on your next birthday.
You are 21 years old.
```

请注意，在函数内部对$age 参数加 1 也会使得函数外部的$currentAge 变量加 1。除非用户的生日发生在函数调用和最终打印语句之间的那一瞬间，否则这可能不是我们想要的。这说明了使用按引用传递参数的危险：它们可能会改变通常在函数作用域之外的变量值。

### 总结

在本章中，我们探讨了如何通过声明和调用函数来促进代码的可重用性，函数是完成特定任务的一系列命名代码。你练习了在独立的.p*hp*文件中声明函数，然后通过 require_once 将其加载到主应用程序文件中，从而编写简洁、结构良好的脚本。你看到返回语句如何允许函数将值返回给调用脚本，同时也提供了一种提前终止函数的机制，并且你探索了如何通过可空和联合类型使函数能够灵活地接受或输出各种数据类型的值。

你了解了参数（函数内使用的变量）和实参（调用函数时传递给这些变量的值）之间的区别。你看到如何通过为参数设置默认值使其变为可选参数，以及如何使用命名参数按照任意顺序传入值，甚至可以跳过某些参数。最后，你了解了值传递和引用传递参数之间的区别，在你希望函数能够更新其作用域外的变量时，这是一个罕见的情况。

### 练习

1.   创建一个项目，其中包含独立的*main.php*和*file2.php*脚本。*file2.php*脚本应该打印出字符串 '456'。在你的*main.php*脚本中，首先打印出 '123'，然后读取并执行 *file2.php*，接着打印出 '789'。最终输出应该是 123456789，但中间的 456 是从 *file2.php* 打印出来的。

2.   编写一个项目，声明一个 which_is_larger() 函数，该函数返回两个整数中的较大者。你的*main.php*脚本应当读取并执行声明该函数的文件，然后打印出以下参数传入函数的结果：

4 和 5

21 和 19

3 和 3

最后一种情况发生了什么，当参数相同时？

3.   修改你的 which_is_larger() 函数，使其能够接受整数或浮点数，并在两个数字相同的情况下返回整数、浮点数或 NULL。

4.   创建一个 *my_functions.php* 文件，声明一个无返回值的函数，用于打印出你名字的首字母的 ASCII 艺术风格。此函数应有两个参数，一个（$character）为字符串，设置用于制作艺术作品的字符，另一个（$spacer）为字符串，设置用于填充空白的字符。为每个参数指定合适的默认值。例如，由于我名字的第一个字母是 M，我的函数可能是 capital_m(string $character = 'M', string $spacer = ' ')，如果没有传入任何参数，它可能会提供如下输出：

```
MM          MM
MMMM      MMMM
MM MMM  MMM MM
MM   MMMM   MM
MM          MM
MM          MM
MM          MM
```

接下来，编写一个 *main.php* 脚本，调用你的函数且不传入任何参数（使用默认值）。然后使用命名参数再次调用该函数两次，一次只传入主字符，另一次只传入填充字符。
