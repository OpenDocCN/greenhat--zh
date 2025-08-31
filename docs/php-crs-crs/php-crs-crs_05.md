

## 第四章：4 条件语句



![](img/opener.jpg)

在本章中，你将学习 PHP 语言中的*条件*元素，包括 if...else 语句、switch 语句和 match 语句。这些结构，以及三元运算符、空合并运算符和逻辑运算符等语言特性，使得编写动态代码成为可能，代码根据一组条件来决定执行什么操作。这些条件可能依赖于特定的输入（例如来自用户或软件系统如数据库或 API 的输入），也可能依赖于其他变化的数据（例如当前日期或时间，或者文件是否存在）。

### 条件为真或为假

在任何决策逻辑的核心都是*布尔表达式*，即返回真或假的代码。最简单的布尔表达式就是 true 或 false 的字面值。然而，在几乎所有情况下，我们都会编写包含某种测试的表达式。这个测试可能会检查变量的值，或者调用一个函数并检查它返回的值。无论哪种方式，测试最终都会评估为真或假。测试的示例包括以下内容：

+   一个变量是否包含特定的值？

+   变量是否有任何值被赋值？

+   文件或文件夹是否存在？

+   一个变量的值是否大于或小于另一个值？

+   一个函数是否根据提供的参数返回真或假？

+   字符串的长度是否大于某个最小值？

+   一个变量是否包含特定数据类型的值？

+   两个表达式是否都为真，还是只有一个为真，或者都不为真？

这些测试都会评估为真或假。它们构成了你可以在代码中使用的选择语句的条件。

### if 语句

可能任何编程语言中最常见的条件语句是 if 语句。它允许你仅在某个条件为真时执行语句。否则，该语句会被跳过。在 PHP 中，if 语句的写法如下：

```
if (condition) statementToPerform;
```

从 if 关键字开始，后面跟着括号中的条件，条件就是评估为真或假的布尔表达式。通常做法是在 if 关键字后、左括号前添加一个空格。接下来是当条件为真时应该执行的语句。

列表 4-1 展示了一个 if 语句的示例。如果一天的小时数小于 12（假设使用 24 小时制时钟），它会打印“早上好”。

```
<?php
$hourNumber = 10;
if ($hourNumber < 12) print 'Good morning';
```

列表 4-1：一个 if 语句示例

首先，我们将变量$hourNumber 设置为 10。然后我们使用 if 语句来测试条件：$hourNumber 的值是否小于 12。由于 10 小于 12，条件为真，因此条件后的语句会被执行，打印出“早上好”。

在这个例子中，我们只想在条件为真时执行一条语句。但如果我们想执行多条语句呢？我们需要一种方法将这些语句分组，以便明确它们都是 if 语句的一部分。为此，我们可以在条件之后立即将这些语句用大括号括起来。大括号划定了一个*语句组*，这是 PHP 的一个结构，可以包含零个、一个或多个语句，并且 PHP 会将其视为一个单一的语句。列表 4-2 展示了一个带语句组的条件语句示例。

```
<?php
$hourNumber = 10;
if ($hourNumber < 12) {
    print 'Good';
    print ' morning';
}
```

列表 4-2：一个重构过的 if 语句，包含语句组

这个 if 语句产生与列表 4-1 相同的结果，但我们已将其重写为包含多个打印语句，每个语句对应消息中的一个单词。这些语句被大括号括起来，以便将它们分组。通常写法是将开括号写在条件的同一行，然后在后续行中写出语句组中的每个语句，最后在另起一行写出闭括号。根据约定，语句组中的每个语句都会缩进。

> 注意

*即使你只有一个条件语句要执行，通常也会将该语句用大括号括起来，形成语句组。这样，所有的* if *语句都会遵循相同的风格，不管涉及多少语句。*

#### if...else 语句

许多情况下需要程序在条件为真时执行一组操作，在条件为假时执行另一组操作。对于这些情况，使用 if...else 语句。列表 4-3 展示了一个例子，其中我们选择打印“Good morning”或“Good day”。

```
<?php
$hourNumber = 14;
if ($hourNumber < 12) {
    print 'Good morning';
} else {
    print 'Good day';
}
```

列表 4-3：一个 if...else 语句

这段代码再次检查 $hourNumber 的值是否小于 12。如果是，条件为真，执行 if 分支的语句，打印出“Good morning”，如同之前一样。然而，如果条件为假，并且 $hourNumber 不小于 12，我们将执行 else 分支的语句，打印出“Good day”。请注意，else 关键字出现在 if 分支的语句组的右大括号之后。然后，else 分支会有一个独立的语句组，用大括号括起来。

在这种情况下，$hourNumber 是 14（下午 2 点），因此条件判断为假，执行了 else 分支的语句。

#### 嵌套 if...else 语句

if...else 语句在两个行动之间做出选择。如果你有多个行动可以选择，你有几个选项。一个方法是将更多的 if...else 语句嵌套在原有的 else 分支中。列表 4-4 展示了一个例子。这个脚本编码了以下逻辑：如果小时数小于 12，打印“Good morning”；如果小时数在 12 到 17（下午 5 点）之间，打印“Good afternoon”；否则，打印“Good day”。

```
<?php
$hourNumber = 14;
❶ if ($hourNumber < 12) {
    print 'Good morning';
} else {
  ❷ if ($hourNumber < 17) {
        print 'Good afternoon';
    } else {
        print 'Good day';
    }
}
```

列表 4-4：嵌套的 if...else 语句

首先，我们有一个 if 语句测试小时数是否小于 12 ❶。如果这个条件不成立，else 语句将会被执行。else 语句的语句组是一个第二个（嵌套的）if...else 语句。这个第二个 if...else 语句的条件是小时数是否小于 17 ❷。（如果我们已经执行到这个步骤，就表示小时数不小于 12，因此实际上我们是在测试小时数是否介于 12 和 17 之间。）如果这个新的测试通过，系统会打印“下午好”。否则，我们进入嵌套 if...else 语句的 else 部分，打印“日安”。尝试使用不同的$hourNumber 值来观察它如何影响脚本的输出。

#### if...elseif...else 语句

在编程中，选择三种或更多行为是一个非常常见的模式，因此 PHP 提供了一种更简洁的语法，避免了嵌套的需要：在 if 语句和 else 语句之间，放置一个或多个 elseif 语句。PHP 引擎会首先测试 if 语句的条件。如果该语句为假，PHP 引擎会继续测试第一个 elseif 语句的条件，然后是下一个 elseif 语句，以此类推。当 PHP 找到一个成立的条件时，会执行该分支的语句，并跳过其余的条件检查。如果没有任何 if 或 elseif 条件成立，那么最后的 else 语句（如果有）将会被执行。

清单 4-5 展示了与清单 4-4 相同的逻辑，但它是使用 if...elseif...else 重写的。

```
<?php
$hourNumber = 14;
if ($hourNumber < 12) {
    print 'Good morning';
❶ } elseif ($hourNumber < 17) {
    print 'Good afternoon';
} else {
    print 'Good day';
}
```

清单 4-5：用 if...elseif...else 简化嵌套的 if...else 语句

我们的第二个条件现在以 elseif 语句的形式出现在 if 语句之后 ❶，而不再需要嵌套在 else 语句中。你可以在 if 和 else 之间添加任意数量的 elseif 语句。

#### 替代语法

PHP 为 if、if...else 和 if...elseif...else 语句提供了一种替代语法，使用冒号而不是大括号来区分代码的各个部分。这种语法在清单 4-6 中得到了展示，复现了清单 4-3 中的 if...else 语句。

```
<?php
$hourNumber = 14;
if ($hourNumber < 12):
    print 'Good morning';
else:
    print 'Good day';
endif;
```

清单 4-6：条件语句的替代语法

在这种替代语法中，if 语句的条件后面跟一个冒号(:)。这一行就像是一个大括号的开始，因此它与 else（或 elseif）关键字之间的所有语句都被认为是条件成立时需要执行的语句组。else 关键字同样后面跟一个冒号，而不是一个大括号。else 分支的语句组以 endif 关键字结束，标志着整个 if...else 结构的结束。

这种替代语法在 Web 应用程序中特别有用，因为 HTML 模板文本可能出现在 if 语句和 else 语句之间，而使用缩进的花括号在代码中可能会让人难以跟踪。同样，endif 关键字清楚地表示整体条件语句的结束。

### 逻辑运算符

PHP 的 *逻辑运算符* 用于操作或组合布尔表达式，产生一个单一的真或假值。通过这种方式，你可以编写比单纯比较两个值更复杂的条件判断语句，就像我们迄今为止所做的那样（例如，测试两个条件是否为真）。这些逻辑运算符执行如 AND、OR 和 NOT 等操作。运算符的总结见 表 4-1。

表 4-1：PHP 逻辑运算符

| 名称 | 运算符 | 示例 | 描述 |
| --- | --- | --- | --- |
| NOT | ! | !$a | 如果 $a 为假，则为真 |
| AND | and&& | $a and $b$a && $b | 如果 $a 和 $b 都为真，则为真 |
| OR | 或&#124;&#124; | $a 或 $b$a &#124;&#124; $b | 如果 $a 或 $b 为真，或两者都为真，则为真 |
| XOR | xor | $a xor $b | 如果 $a 或 $b 为真，但不能两者都为真，则为真 |

请注意，AND 和 OR 操作可以有两种写法：使用单词（and 或 or）或使用符号（&& 或 ||）。这两种写法执行相同的功能，但符号版本在表达式求值时优先级高于单词版本。（我们在 第一章 中讨论了运算符优先级的顺序，主要是在算术运算符的上下文中。）

#### NOT

感叹号（!）表示“NOT”操作符。该操作符用于否定一个布尔表达式或测试该表达式是否不为真。例如，清单 4-7 使用 NOT 操作符测试驾驶员的年龄。在爱尔兰，开车必须年满 17 岁。

```
<?php
$age = 15;
if (!($age >= 17)) {
    print 'Sorry, you are too young to drive a car in Ireland.';
}
```

清单 4-7：使用 NOT (!) 操作符的 if 语句

if 语句检查 $age 是否 *不* 为真，即是否小于 17。由于 15 小于 17，因此运行脚本时应该看到以下消息被打印出来：

```
Sorry, you are too young to drive a car in Ireland.
```

请注意，我们将 $age >= 17 放在括号中，以将其与 NOT 操作符分开。这是因为 NOT 操作符通常优先级高于 >= 操作符，但我们希望在使用 ! 否定结果之前，先检查 $age 是否大于或等于 17。如果我们写成 if (!$age >= 17) 而没有内括号，PHP 会首先尝试计算 !$age。NOT 操作符要求布尔操作数，因此 $age 中的值 15 会被转换为 true（任何非零值都一样）。然后，由于 !true 为 false，表达式变成 false >= 17。

接下来，PHP 会尝试评估>=比较操作，因为其中一个操作数是布尔值，它也会尝试将第二个操作数转换为布尔值。因此，整数 17 会被转换为 true（因为它是非零的），从而得到表达式 false >= true，这个结果为 false。最终，在没有额外括号的情况下，!$age >= 17 会对任何非零整数值的$age 评估为 false。

为了避免所有这些类型转换和因缺少括号而可能导致的错误，我通常会在引入 NOT 运算符之前，为 if 语句创建一个临时布尔变量。例如，清单 4-8 展示了清单 4-7 的另一版本，增加了一个额外的变量来避免混合整数和布尔值的任何可能性。

```
<?php
$age = 15;
$seventeenAndOlder = ($age >= 17);
if (!$seventeenAndOlder) {
    print 'Sorry, you are too young to drive a car in Ireland.';
}
```

清单 4-8：清单 4-7 的简洁版本，增加了一个布尔变量

我们使用$seventeenAndOlder 变量来存储$age >= 17 测试的布尔值。然后，if 语句使用 NOT 运算符测试$seventeenAndOlder 是否为假。虽然与清单 4-7 相比，这增加了一行代码，但由于我们将年龄测试的布尔表达式与 if 语句的条件分开，它更容易理解。

> 注意

*将像* $age >= 17 *这样的表达式放在括号内，在将其值赋给变量时并非必要。清单 4-8 使用括号来帮助使代码更易于阅读。*

#### 和

使用 AND 运算符的表达式在两个操作数都为真时为真。你可以使用关键字 and 或双重与符号（&&）来创建 AND 操作。例如，清单 4-9 中的 if...else 语句使用 AND 运算符来判断一个驾驶员是否满足两个条件才能申请驾照考试。在爱尔兰，必须通过理论考试并持有至少六个月的学习驾驶执照，才能申请驾照考试。

```
<?php
$passedTheoryTest = true;
$monthsHeldLearnersLicense = 10;
$heldLearnersLicenseEnough = ($monthsHeldLearnersLicense >= 6);

if ($passedTheoryTest and $heldLearnersLicenseEnough) {
    print 'You may apply for a driving test.';
} else {
    print "Sorry, you don't meet all conditions to take a driver's test.";
}
```

清单 4-9：使用 AND 运算符的 if...else 语句

我们将$passedTheoryTest 变量声明为 true，并将$monthsHeldLearnersLicense 的值设置为 10。然后，我们测试$monthsHeldLearnersLicense 是否大于或等于 6，并将结果布尔值（此例为 true）存储在$heldLearnersLicenseEnough 变量中。接下来，我们声明一个 if...else 语句，条件为$passedTheoryTest 和$heldLearnersLicenseEnough。由于两个值都为真，AND 操作也为真，因此消息“你可以申请驾照考试”将被输出。

尝试将$passedTheoryTest 更改为 false 或将$monthsHeldLearnersLicense 设置为小于 6 的值。此时，AND 操作应评估为 false，并且语句的 else 分支中的消息应该输出。

#### 或

OR 运算在任一操作数或两个操作数都为真时返回 true。你可以使用关键字 or 或双竖线（||）来编写 OR 运算。清单 4-10 展示了一个使用 OR 运算符的 if 语句，判断密码是否未通过基本安全规则（通过包含字符串 'password' 或长度小于六个字符）。

```
<?php
$password = '1234';
$passwordContainsPassword = str_contains($password, 'password');
$passwordTooShort = (strlen($password) < 6);

❶ if ($passwordContainsPassword || $passwordTooShort) {
    print 'Your password does not meet minimal security requirements.';
}
```

清单 4-10：一个使用 OR 运算符的 if 语句

我们声明了一个 $password 变量，存储字符串 '1234'。然后我们声明了两个布尔变量来帮助测试。首先，$passwordContainsPassword 被赋值为将变量 $password 和字符串 'password' 传递给内置的 str_contains() 函数的结果。如果第二个字符串参数（“needle”）在第一个字符串参数（“haystack”）中找到，函数返回 true，否则返回 false。由于此例中 $password 变量不包含字符串 'password'，$passwordContainsPassword 的值为 false。另一个布尔变量 $passwordTooShort，如果 $password 的长度小于 6，则为 true，通过内置的 strlen() 函数进行测试。由于 $password 中的字符串 '1234' 长度小于六个字符，因此该变量将被赋值为 true。

最后，我们声明一个 if 语句，使用 OR 运算符（||）根据两个布尔变量 ❶ 创建条件。由于至少有一个变量为真，if 语句条件通过，并打印出一条消息，指示密码不安全：

```
Your password does not meet minimal security requirements.
```

尝试将 $password 的值更改为六个字符或更长的字符串（不是 'password'），例如 "red$99poppy"。此时，$passwordContainsPassword 和 $passwordTooShort 都不会为 true，因此 if 语句中的逻辑 OR 测试将为假，且不会打印任何消息。

#### 异或

异或运算（XOR，*exclusive OR*）仅在两个操作数中只有一个为真时返回 true，而两个都不为真时返回 false。我们使用关键字 xor 来创建一个 XOR 表达式。清单 4-11 展示了一个使用 XOR 运算符的 if...else 语句。该代码判断一个甜点是否奶油丰富，但不至于过于奶油。 （卡仕达 *和* 冰淇淋就太多了！）

```
<?php
$containsIceCream = true;
$containsCustard = false;
if ($containsIceCream xor $containsCustard) {
    print 'a nice creamy dessert';
} else {
    print 'either too creamy or not creamy enough!';
}
```

清单 4-11：一个带有异或运算符的 if...else 语句

我们声明了两个布尔变量 $containsIceCream 和 $containsCustard，将其中一个设置为 true，另一个设置为 false。然后我们声明了一个 if...else 语句，条件为 $containsIceCream xor $containsCustard。由于 XOR 运算符的缘故，若这两个变量中只有一个为真，条件将评估为真，并打印出一条美味的奶油甜点消息。如果两个变量都不为真，或者两个都为真，则 XOR 表达式为假，相应地会打印出“奶油过多”或“奶油不足”的消息。

在此示例中，由于只有一个变量为 true，我们应该会得到一条漂亮的奶油甜点消息。尝试更改两个布尔变量的值，看看 XOR 表达式的结果如何受到影响。

### switch 语句

switch 语句是一种条件结构，用于将一个变量与多个可能的值进行比较，或称为 *case*。每个 case 有一个或多个语句，如果其值与变量匹配（通过类型转换，因此它执行类似 == 的相等性测试），则这些语句会被执行。你还可以提供一个默认的 case，如果没有任何值匹配。如果你需要从三个或更多可能的路径中选择，switch 语句是 if...elseif...else 语句的一个方便替代方案，只要决策依据是单一变量的值。

清单 4-12 显示了一个 switch 语句，该语句根据 $country 变量的值打印相应的货币信息。

```
<?php
$country = 'Ireland';

❶ switch ($country) {
  ❷ case 'UK':
        print "The pound is the currency of $country\n";
        break;
  ❸ case 'Ireland':
    case 'France':
    case 'Spain':
      ❹ print "The euro is the currency of $country\n";
        break;
    case 'USA':
        print "The dollar is the currency of $country\n";
        break;
  ❺ default:
        print "(country '$country' not recognized)\n";
}
```

清单 4-12：使用 switch 语句根据 $country 的值打印货币

首先，我们将 $country 赋值为 'Ireland'。然后，我们开始一个 switch 语句，使用 switch 关键字并将要测试的变量放在括号中（$country）❶。switch 语句的其余部分被一对花括号括起来。在 switch 语句内部，我们声明需要检查的 $country 的值，每个值都在各自的缩进的 case 子句中定义。每个 case 子句使用 case 关键字定义，后跟要测试的值，再后跟一个冒号（:）。然后，如果该 case 匹配，就在新的、更深的缩进行中写出要执行的语句。例如，如果 $country 的值是 'UK' ❷，则会打印出信息：The pound is the currency of UK（英镑是英国的货币）。

如果你希望相同的操作适用于多个 case，可以将这些 case 按顺序列出，只需在最后列出一次要执行的语句。例如，爱尔兰、法国和西班牙都使用欧元，因此我们按顺序列出了这些 case ❸。这些 case 后面的 print 语句 ❹ 将适用于它们中的任何一个；你不需要为每个 case 重复该语句。

我们的脚本为当 $country 的值为 'USA' 时增加了一个额外的 case。然后，switch 语句的最后部分使用 default 关键字声明一个默认的 case，而不是使用 case ❺。如果没有任何其他 case 与正在测试的变量匹配，默认的 case 会被执行。考虑到我们将 $country 设置为 'Ireland'，脚本应该输出信息：The euro is the currency of Ireland（欧元是爱尔兰的货币）。

请注意，我们在每个 case 的语句组中都包含了 break 关键字，在每个 print 语句后面。这会中断，或*跳出* switch 语句，防止执行该语句中的任何后续代码。理解 break 语句的作用非常重要。一旦找到匹配的 case，switch 语句主体中的所有剩余语句都会被执行，即使是其他不匹配的 case 的语句，除非遇到 break 语句中断执行。例如，如果我们从 清单 4-12 中删除所有的 break 语句，最终输出将是：

```
The euro is the currency of Ireland
The dollar is the currency of Ireland
(country 'Ireland' not recognized)
```

`$country` 的值是 'Ireland'，而不是 'UK'，因此第一个 case 不匹配，第一个打印语句被跳过。然而，一旦我们遇到 'Ireland' 的 case，接下来的三个打印语句就会执行，因为没有 `break` 语句来中断 `switch` 语句。这通常不是你希望从 `switch` 语句中得到的行为，因此几乎在每种情况下，你都需要在每个 case（或一组 cases）的末尾添加 `break` 语句，就像我们在 示例 4-12 中所做的那样。

### `match` 语句

`match` 语句根据另一个变量的值为一个变量选择一个值。你可以使用 `switch` 语句来完成相同的任务，但 `match` 语句更加简洁。此外，`match` 语句依赖于严格比较（相当于使用 `===` 测试身份），而 `switch` 语句在进行任何类型转换后才进行比较（相当于使用 `==` 测试相等）。因此，当一个变量需要与多个相同类型的值进行比较，并且根据该测试执行的操作是为变量赋值时，使用 `match` 语句会更合适。

示例 4-13 展示了与 示例 4-12 中的 `switch` 语句相同的逻辑，但采用了 `match` 语句来实现。

```
<?php
$country = 'Ireland';

❶ $currency = match ($country) {
    'UK' => 'pound',
    'Ireland' => 'euro',
    'France' => 'euro',
    'Spain' => 'euro',
    'USA' => 'dollar',
  ❷ default => '(country not recognized)'
};

print "The currency of $country is the $currency";
```

示例 4-13：使用 `match` 语句根据 `$country` 的值设置 `$currency`

我们将 `match` 语句写成 `$currency` 变量赋值的一部分 ❶。它由 `match` 关键字组成，后面跟着要检查的变量（括号内），接着是由逗号分隔的、用大括号包围的 *arms* 序列。每个 arm 的形式是 `x => y`，其中 y 是当 `$country` 的值与 x 匹配时赋值给 `$currency` 的值。与 `switch` 语句一样，我们还提供了一个默认的 arm，以防没有值匹配 ❷。在 `match` 语句之后，我们打印一条消息，包含 `$country` 和 `$currency` 的值。

与 示例 4-12 中的 `switch` 语句相比，这个 `match` 语句更简洁。赋值给 `$currency` 后，我们只需要写一个打印语句，而不需要为 `switch` 语句中的每个 case 写一个单独的打印语句。我们也不再需要所有的 `break` 语句；使用 `match` 语句后，一旦找到匹配，余下的语句会被忽略。

`match` 语句是 PHP 语言中新出现的一种语法。许多经验丰富的程序员仍然使用 `switch`，而在某些情况下，`match` 更加高效。（我有时也会犯这个错误。）一般来说，如果你需要测试一个变量的多个值，我建议你首先尝试使用 `match` 语句。只有在该方案不适用时，才应切换到 `switch` 语句。

### 三元运算符

PHP 的*三元运算符*（或*三部分运算符*）根据测试条件是否为真来选择两个值之一。这个运算符由两个独立的符号组成，一个问号（?）和一个冒号（:），并且按以下形式书写：

```
booleanExpression ? valueIfTrue : valueIfFalse
```

问号左侧写一个布尔表达式，它的值为真或假（例如，比较两个值）。问号右侧写两个由冒号分隔的值。如果布尔表达式为真，则选择冒号左侧的值（valueIfTrue）。如果布尔表达式为假，则选择冒号右侧的值（valueIfFalse）。通常，结果会赋值给一个变量。

本质上，三元运算符提供了一种更简洁的方式来编写 if...else 语句，只要 if...else 语句的目的是给变量赋值（而不是执行其他操作）。为了说明，列表 4-14 展示了根据$region 的值选择$currency 的两种方法：首先使用 if...else 语句，然后使用三元运算符。

```
<?php
$region = 'Europe';

❶ if ($region == 'Europe') {
    $currency = 'euro';
} else {
 $currency = 'dollar';
}

print "The currency of $region is the $currency (from if...else statement)\n";

$region = 'USA';
❷ $currency = ($region == 'Europe') ? 'euro' : 'dollar';

print "The currency of $region is the $currency (from ternary operator statement)\n";
```

列表 4-14：比较 if...else 和三元运算符语句

我们将$region 赋值为'欧洲'。然后声明一个 if...else 语句，如果地区是'欧洲'，则将$currency 的值设置为'euro'，否则设置为'dollar' ❶。我们打印一条消息以验证结果。接下来，我们将$region 更改为'美国'，并使用三元运算符重新赋值$currency ❷。三元运算符表达式遵循与 if...else 语句相同的逻辑：如果$region 等于'欧洲'，代码将$currency 设置为'euro'，否则将$currency 设置为'dollar'。再次，我们打印消息以检查结果。以下是运行脚本后的输出：

```
The currency of Europe is the euro (from if...else statement)
The currency of USA is the dollar (from ternary operator statement)
```

第二行显示三元运算符已按预期工作，因为$region 的值不是'欧洲'，因此$currency 的值被赋为'dollar'。正如你所见，在这种需要在两个可能的值之间做出简单选择的情况下，三元运算符非常简洁，只有一行代码，而 if...else 语句则有四行代码。

### 空合并运算符

另一种在两个值之间进行选择的运算符是*空合并运算符*，使用两个问号（??）表示。这个运算符根据变量是否为 NULL 来做出选择。使用空合并运算符的表达式的一般形式如下：

```
$variable = value ?? valueIfNull
```

首先，空合并运算符检查左侧的值，即 ?? 运算符左侧的表达式。这可以是一个变量，或者是一个返回值的函数。如果该表达式不是 NULL，则将值赋给 $variable。否则，空合并运算符右侧的值（valueIfNull）将被赋给该变量。这提供了一种后备机制，防止在变量未定义（或为 NULL）时抛出警告或错误。当你期望从用户那里获得一个值但没有提供，或者当你在数据库中查找记录而该记录不存在时，这种机制特别有用。

列表 4-15 显示了空合并运算符的使用示例。我们使用它来测试 $lastname_from_user 变量两次，第一次是在它还未被赋值时（因此是 NULL），第二次是在它被赋值之后。

```
<?php
❶ $lastname = $lastname_from_user ?? 'Anonymous';
print "Hello Mr. $lastname\n";

$lastname_from_user = 'Smith';
❷ $lastname = $lastname_from_user ?? 'Anonymous';
print "Hello Mr. $lastname\n";
```

列表 4-15：使用空合并运算符测试 NULL

首先，我们使用空合并运算符来设置 $lastname ❶ 的值。该运算符测试 $lastname_from_user 变量，由于该变量尚未被赋值，因此为 NULL。因此，$lastname 应该被赋予 ?? 运算符右侧的值（字符串 'Anonymous'）。我们打印出一条消息以检查结果。接着，在给 $lastname_from_user 赋值后，我们使用相同的空合并运算符表达式再次设置 $lastname 的值 ❷。这次，由于 $lastname_from_user 包含非 NULL 值，该值应该被传递给 $lastname。以下是结果：

```
Hello Mr. Anonymous
Hello Mr. Smith
```

第一行显示，由于变量 $lastname_from_user 为 NULL，$lastname 被赋值为字符串 'Anonymous'。然而，第二次，$lastname_from_user 中的字符串 'Smith' 成功地存储在 $lastname 变量中并被打印出来。

### 小结

在本章中，你学习了用于编写做出决策的代码的关键字和运算符。计算机和编程语言的许多强大功能都建立在我们讨论的各种运算符和选择语句上。你看到 if 和 if...else 语句如何基于单一测试做出选择，尽管该测试本身可能会结合布尔表达式与逻辑运算符，如 AND 或 OR。你还看到了如何通过在 if 和 else 之间添加 elseif 分支来结合多个测试。然后你学习了其他条件结构，包括 switch 和 match 语句，它们会测试变量是否具有不同的可能值。这些结构允许你定义一个或多个在找到特定值时执行的语句。与这些结构紧密相关的是三元运算符和空合并运算符，它们都在两个可能的值之间做选择。

### 练习

1.   编写一个脚本，将一个名字赋值给 $name 变量，然后如果字符串的长度小于四个字符，则打印消息 "That is a short name"。

2.   编写一个脚本，用于确定洗衣机的大小。该脚本应检查变量$laundryWeightKg 的值，如果值小于 9，则打印“适合标准洗衣机”，否则打印“需要中型到大型洗衣机”。

3.   使用 switch 语句或 match 语句测试$vehicle 变量的值，并根据该值打印相应的消息。使用以下值/消息组合：

bus   "滴滴声"

train   "在轨道上行驶"

car   "至少有三个轮子"

helicopter   "可以飞行"

bicycle   "学会了就永远不会忘记"

**(上述内容无)**   "你选择了人迹罕至的道路"

4.   编写一个脚本，如果$用户名字正确且$密码正确，打印消息“您现在已登录”。否则，打印“凭证无效，请重试”。
