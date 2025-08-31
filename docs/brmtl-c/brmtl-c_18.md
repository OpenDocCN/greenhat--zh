# 第十六章：16  

浮点数  

![](img/chapterart.png)  

在本书的这一部分，我们将讨论一些通常在嵌入式编程中不常用的 C 特性，但你可能会在大型机编程中遇到。浮点数在嵌入式编程中并不常见，因为许多低端处理器芯片无法处理它们。即使你使用的 CPU 支持浮点数，浮点运算依然很慢、不精确，而且使用起来比较复杂。

然而，由于你在科学或 3D 图形程序中偶尔会遇到这些数字，因此你应该有所准备。本章涵盖了浮点数的基础知识，为什么浮点运算如此昂贵，以及在使用它们时可能出现的一些错误。  

## 什么是浮点数？  

*浮点数* 是小数点可以浮动的数字。它可以出现在数字的不同位置，例如 `1.0`、`0.1`、`0.0001` 或 `1000.0`。严格来说，小数点后面有数字并不是必须的。例如，`1.0` 和 `1.` 是相同的数字。然而，如果浮点数在小数点两侧都有数字，它更容易阅读和理解。  

我们也可以使用指数表示法来书写浮点数，例如 `1.0e33`，表示数字 1.0 × 10³³。 （你可以使用大写的 `E` 或小写的 `e`，但小写版本更易于阅读。）  

### 浮点类型  

在 C 中，浮点类型有 `float`、`double` 和 `long double`。`double` 类型的精度和范围是 `float`（单精度）类型的两倍。`long double` 的精度和范围比其他两种类型更大。  

所有浮点常量默认都是 `double` 类型，除非你明确告诉 C 其他类型。在数字末尾添加 `F` 后缀将其变为单精度 `float`，而在末尾添加 `L` 会将其变为 `long double`。  

浮点数需要小数点。考虑以下代码：  

```
float f1 = 1/3;
float f2 = 1.0 / 3.0;
```

第一行赋值并不会将 `f1` 的值赋为 0.3333\。相反，它将其赋值为 0.0，因为 1 和 3 是整数。C 执行了一个 *整数除法*（结果为整数 0），将其提升为浮点数后再进行赋值。第二行则按我们希望的方式，赋值为 0.3333。  

### 自动转换  

C 会在你不注意的情况下进行一些自动转换。如果表达式的一个操作数是浮点数，C 会自动将另一个操作数转换为浮点数。下面是一个例子：  

```
f = 1.0 / 3;    // Bad form
```

在这种情况下，数字 3 会在除法操作之前被转换为 3.0。这个例子被认为是不好的一种写法，因为如果可以的话，你不应该混合整数和浮点常量。而且，如果你将浮点数赋给一个整数，它会被转换为整数。  

## 浮点数的问题  

浮点数的一个问题是它们不精确。例如，1/3 在十进制浮点数中是 0.333333。无论你使用多少位数，它仍然不精确。我们这里不展示二进制浮点数（计算机使用的），而是使用十进制浮点数（人类熟悉的）。所有在十进制浮点数中可能出错的地方，也会在二进制版本中出错。唯一的区别是，十进制浮点数的例子更容易理解。

十进制浮点数是科学记数法的一种简化版本。下面是一个例子：

```
+1.234e+56
```

这个数字有符号（+）、一个小数部分（四位）和一个指数。对人类来说这不是问题，但在计算机中表示这样的数字是有难度的。

计算机使用类似的格式，只不过指数和小数部分是二进制的。此外，它们还会将顺序混淆，存储的顺序是符号、指数和小数部分。有关更多细节，请参阅几乎所有计算机当前使用的 IEEE-754 浮点数规范。

### 四舍五入误差

你知道 1 + 1 是 2，但 1/3 + 1/3 并不是 2/3。让我们来看看这是如何工作的。首先，我们来加上这两个数字：

```
+3.333e-01    // 1/3 in our notation
+3.333e-01    // 1/3 in our notation
+6.666e-01
```

然而，2/3 是以下这个：

```
+6.667e-01
```

这是一个四舍五入误差的例子。`+3.333e-01`和 1/3 之间有一个小误差。由于我们使用的标准四舍五入规则，我们将结果向下取整。当我们计算 2/3 时，得到`6.67e-1`。在这种情况下，四舍五入规则使我们向上取整，因此虽然 1 + 1 = 2（整数），但 1/3 + 1/3 != 2/3（浮点数）。

我们可以使用一些技巧来最小化四舍五入误差。大多数计算机使用的一个技巧是在计算过程中加入保护位。*保护位*是在进行计算时，给数字加上的一个额外位数。当计算结果出来后，保护位会被丢弃。

### 精度位数

单精度浮点数（`float`）应该能提供大约 6.5 位的精度，但这并不总是准确的。你能信任多少位呢？在前面的例子中，我们可能会倾向于认为我们的十进制浮点数的前三位是准确的，但我们不能依赖这一点。

让我们计算 2/3 – 1/3 – 1/3：

```
+6.667e-01    // 2/3
-3.333e-01    // 1/3
-3.333e-01    // 1/3
+0.001e-01    // Result unnormalized
+1.000e-04    // Result normalized
```

有多少位是正确的？我们结果的第一个数字是 1（*规范化*意味着我们将数字改变为使得第一个位置有一个数字。除了少数边缘情况外，所有浮点数都存储为规范化形式，我们稍后会讲解这些边缘情况）。正确的第一个数字应该是 0。

浮点运算的设计中固有许多问题。主要归结为大多数数字都是不精确的，这可能导致计算错误和精确比较时的问题。

如果你只进行了有限量的浮点数运算，它们可能不会咬你，但你应该意识到它们。如果你进行了大量的浮点数运算，你应该查阅计算机科学的一个分支，称为*数值分析*，专门处理浮点数问题以及如何从中获得稳定的结果，但这超出了本书的范围。

## 无穷大、NaN 和子规范数

IEEE 浮点格式有一些位模式是没有意义的数字。例如，考虑数字 0*10⁵。由于 0 乘以任何东西都是 0，我们可以在这种情况下使用指数来表示特殊值。在本节中，我们将查看其中的一些以及浮点格式的边缘情况。

考虑以下表达式：

```
float f = 1.0 / 0.0;
```

如果这是一个整数，将其除以零会中止你的程序。然而，因为它是浮点数，结果是`f`被赋予了值`INFINITY`（这个常量在`#include <math.h>`头文件中定义）。

同样，该语句：

```
float f = -1.0 / 0.0;
```

分配`f`值为`-INFINITY`。

数字`INFINITY`和`-INFINITY`不是浮点数（它们没有数字和小数点），但 IEEE 浮点规范已定义了几种这样的特殊数字。由于你可能会遇到这些类型的数字（特别是如果你的程序包含错误），知道它们是什么很重要。

你也可能遇到`NaN`（不是一个数字），当一个操作无法产生结果时生成。这里是一个例子：

```
#include <math.h>
float f = sqrt(-1.0);
```

C 标准的新版本包括复数，但`sqrt`函数始终返回`double`，因此`sqrt(-1.0)`始终返回`NaN`。

现在，我们能在我们的浮点方案中表示的最小数字是多少？你可能会说它是以下内容：

```
+1.0000e-99
```

分数 1.0000 是我们可以创建的最小分数。（如果我们使用 0.5000，它将被规范化为 5.0000。）而且，-99 是我们可以得到的最小指数，只用了两位数字。

然而，我们可以变得更小：

```
+0.1000e-99   // -99 is the limit on the exponent.
```

而且还要更小：

```
+0.0001e-99
```

到目前为止，我们讨论的数字都是规范化的，这意味着一个数字始终位于第一位。这些数字被认为是*子规范的*。我们还失去了一些有效位数。我们有五个有效位数的数字`+1.2345e-99`，但只有一个有效位数的`+0.0001e-99`。

在 C 中，`isnormal`宏在数字规范化时返回 true，并且`issubnormal`宏在数字是子规范化时返回 true。

如果你遇到了子规范化的数字，你已经进入了 C 浮点的最黑暗的角落。到目前为止，我还没有看到任何真正使用它们的程序，但它们确实存在，你应该意识到它们。

## 实施

浮点数可以用多种方式实现。让我们从我们一直在使用的 STM 芯片开始。实现很简单：你不能有浮点数。硬件不支持，而且机器没有足够的能力在软件中实现它。

低端芯片通常没有浮点单元。因此，浮点运算是通过使用软件库来实现的，这会带来一定的代价。通常，浮点运算的时间大约是整数运算的 1,000 倍。

一旦你使用了更高端的芯片，你会发现有原生的浮点数支持。虽然这些运算依然昂贵；一个浮点运算大约需要比整数运算长 10 倍的时间。

## 替代方案

处理浮点数的最佳方法之一是根本不使用它。如前所述，举个例子，当处理货币时。如果你将货币存储为浮点数，四舍五入误差最终会导致你得出错误的总额。如果你将货币存储为整数形式的分数，你就能避免浮点数及其所有问题。

让我们定义一个简单的定点数，规定小数点后有 2 位数字。以下是一些示例和整数实现：

```
Fixed point    Implementation
12.34          1234
00.01             1
12.00          1200
```

要加减定点数，只需加减底层实现即可：

```
 12.34         1234
+22.22        +2222
------        -----
 34.56         2346

 98.76         9876
-11.11        -1111
------         ------
 87.65         8765 
```

要乘以定点数，先将两个数相乘，然后除以 100 以修正小数点的位置：

```
 12.00           1200
x 00.50         x 0050
                  60000 (Uncorrected)
 ------          ------
x 06.00            0600 (After 100 correction)
```

要进行除法，你需要做相反的操作：除以底层数字，并乘以一个修正值。

列表 15-1 包含了一个演示定点数使用的程序。

**fixed.c**

```
/**
 * Demonstrate fixed-point numbers.
 */
#include <stdio.h>

/**
 * Our fixed-point numbers have the form
 * of xxxxx.xx with two digits to the right
 * of the decimal place.
 */
typedef long int fixedPoint;            // Fixed-point data type
static const int FIXED_FACTOR = 100;    // Adjustment factor for fixed point
/**
 * Add two fixed-point numbers.
 *
 * @param f1 First number to add
 * @param f2 Second number to add
 * @returns f1+f2
 */
static inline fixedPoint fixedAdd(const fixedPoint f1, const fixedPoint f2)
{
    return (f1+f2);
}
/**
 * Subtract two fixed-point numbers.
 *
 * @param f1 First number to subtract
 * @param f2 Second number to subtract
 * @returns f1-f2
 */
static inline fixedPoint fixedSubtract(
    const fixedPoint f1, 
    const fixedPoint f2)
{
    return (f1-f2);
}
/**
 * Multiply two fixed-point numbers.
 *
 * @param f1 First number to multiply
 * @param f2 Second number to multiply
 * @returns f1*f2
 */
static inline fixedPoint fixedMultiply(
    const fixedPoint f1,
    const fixedPoint f2)
{
    return ((f1*f2)/FIXED_FACTOR);
}
/**
 * Divide two fixed-point numbers.
 *
 * @param f1 First number to divide
 * @param f2 Second number to divide
 * @returns f1/f2
 */
static inline fixedPoint fixedDivide(
    const fixedPoint f1,
    const fixedPoint f2)
{
    return ((f1*FIXED_FACTOR) / f2);
}
/**
 * Turn a fixed-point number into a floating one (for printing).
 *
 * @param f1 Fixed-point number
 * @returns Floating-point number
 */
static inline double fixedToFloat(const fixedPoint f1)
{
    return (((double)f1) / ((double)FIXED_FACTOR));
}
/**
 * Turn a floating-point number into a fixed one.
 *
 * @param f1 Floating-point number
 * @returns Fixed-point number
 */
static inline fixedPoint floatToFixed(const double f1)
{
    return (f1 * ((double)FIXED_FACTOR));
}

int main()
{
    fixedPoint f1 = floatToFixed(1.2);  // A fixed-point number
    fixedPoint f2 = floatToFixed(3.4);  // Another fixed-point number

    printf("f1 = %.2f\n", fixedToFloat(f1));
    printf("f2 = %.2f\n", fixedToFloat(f2));
    printf("f1+f2 = %.2f\n", fixedToFloat(fixedAdd(f1, f2)));
    printf("f2-f1 = %.2f\n", fixedToFloat(fixedSubtract(f2, f1)));
    printf("f1*f2 = %.2f\n", fixedToFloat(fixedMultiply(f1, f2)));
    printf("f2/f1 = %.2f\n", fixedToFloat(fixedDivide(f1, f2)));
    return (0);
}
```

列表 16-1：使用定点数

这不是一个完美的实现。在某些地方，例如乘法和除法运算，可能会出现四舍五入误差，但如果你真的精通定点数，你应该能够轻松发现它们。

## 总结

理解浮点数的底层实现和限制非常重要。正如之前提到的，你永远不应该将浮点数用于货币。会计人员需要精确的数字，而四舍五入误差可能导致错误的结果。计算机科学中的数值分析分支负责分析如何进行计算，并找出如何最小化误差。本章向你展示了基础知识。如果你要广泛使用浮点数，你应该具备一定的数值分析知识。然而，使用浮点数的最佳方式是完全避免使用它，因此请确保你理解，浮点数有替代方案，例如定点数。

维基百科有一篇关于 IEEE 浮点标准的好文章，并提供了大量在线参考材料：[`en.wikipedia.org/wiki/IEEE_754`](https://en.wikipedia.org/wiki/IEEE_754)。

## 编程问题

1.  编写一个计算角度`sin`值的函数。为了得到准确的结果，你需要计算多少个因子？

1.  使用`float`类型，计算π的尽可能多的数字。如果你将数据类型改为`double`，你能得到多少位数字？`long double`呢？

1.  假设你想找出浮点数小数部分的位数。编写一个程序，从*x* = 1 开始，并不断将*x*除以 2，直到(1.0 + *x* = 1.0)。你除以 2 的次数就是浮点计算中位数的数量。
