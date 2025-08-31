## 第八章：测量标准

在一个长而复杂的计算机程序中，混淆测量单位是非常容易发生的。当这种混淆发生时，后果可能是极其昂贵的，甚至是悲剧性的。最著名的例子之一是 1999 年 NASA 的*火星气候轨道探测器*坠毁事件。事故调查揭示，坠毁的原因是单位不匹配；使用了磅力秒而不是牛顿秒。这一错误导致了不正确的轨迹计算，最终导致了探测器的毁灭。

可以争辩说，适当的测试应该能检测到计算错误，从而避免坠毁，但更大的问题是，如果编程语言通过其类型系统强制使用正确的单位，这个错误是否根本不会发生。

多年来，人们一直在尝试在软件系统中强制使用测量单位，通常通过外部库来实现，并且成功程度不一。F#是最早将测量单位作为其静态类型检查系统的原生部分之一的编程语言之一。除了提供比基本类型系统更高的安全性外，F#的测量单位还可以通过消除关于代码中实际期望内容的模糊性来增强代码的可读性，而无需依赖更长的标识符。

## 定义度量

为了启用静态测量检查，你首先需要定义一个度量。*度量*是类似类型的构造，带有`Measure`属性来表示实际世界中的测量。它们可以包含一个可选的*测量公式*，通过其他度量来描述该度量。例如，以下定义创建了一个名为英尺的度量单位：

```
[<Measure>] type foot
```

国际单位制

F# 3.0 包含了国际单位制（SI）单位的预定义度量类型，包括米、千克和安培等。你可以在`Microsoft.FSharp.Data.UnitSystems`命名空间中找到每个 SI 单位。在 F# 3.0 之前，SI 单位包含在 F# PowerPack 中，并可以在`Microsoft.FSharp.Math`命名空间中找到。

## 测量公式

测量公式允许你基于一个或多个先前定义的度量来定义派生度量。最基本的情况是，公式作为一种简单的方式为类型创建同义词。例如，如果你已经定义了一个名为`foot`的度量，并希望将其缩写为`ft`，你可以这样写：

```
[<Measure>] type ft = foot
```

然而，测量公式并不总是那么简单；它们也可以用来描述类型之间更复杂的关系，例如距离与时间的关系。例如，英里每小时可以定义为`m / h`（假设`m`和`h`之前已分别定义为英里和小时）。

在编写测量公式时，以下是一些最重要的指南：

+   你可以通过用空格或星号（`*`）分隔两个度量来乘度量，从而创建一个*积度量*。例如，扭矩有时以磅-英尺为单位，且可以在 F#中表示为：

    ```
    [<Measure>] type lb
    [<Measure>] type ft
    [<Measure>] type lbft = lb ft
    ```

+   你可以通过用斜杠（/）分隔两个度量来除度量，从而创建一个*商度量*。例如，按时间计算的距离，如每小时多少英里，可以这样表示：

    ```
    [<Measure>] type m
    [<Measure>] type h
    [<Measure>] type mph = m / h
    ```

+   正整数和负整数值可以用来表示两个度量之间的指数关系。例如，平方英尺可以这样表示：

    ```
    [<Measure>] type ft
    [<Measure>] type sqft = ft ^ 2
    ```

## 应用度量

一旦你定义了一些度量，你就可以将它们应用于值。F#默认定义了带度量的`sbyte`、`int16`、`int32`、`int64`、`float`、`float32`和`decimal`原始类型。没有度量注释的值称为*无度量*或*无量纲*。

要将度量应用于常量值，你只需将值注释为该度量，就像将度量作为泛型类型参数一样。例如，你可以按如下方式定义一个以英尺为单位的长度和以平方英尺为单位的面积：

```
> **let length = 10.0<ft>**
**let area = 10.0<sqft>;;**

val length : float<ft> = 10.0
val area : float<sqft> = 10.0
```

如你所见，`length`绑定到`float<ft>`，而`area`绑定到`float<sqft>`。

星星去哪儿了？

尽管度量单位在 F#的类型系统中起着重要作用，但它们在编译过程中会被擦除，因此对编译后的代码没有影响。这并不是说度量类型在编译后的程序集内不存在；它只是意味着它们没有附加到任何单独的值上。擦除的最终结果是，度量单位只能在 F#代码中强制执行，而任何其他语言编写的程序集使用的度量感知函数或类型将被视为无度量。

度量注释非常适合常量值，但我们如何将度量应用于外部数据（例如从数据库读取的数据）呢？将无度量值转换为有度量值的最简单方法是将其乘以一个有度量的值，像这样：

```
[<Measure>] type dpi
let resolution = 300.0 * 1.0<dpi>
```

在这里，我们定义了一个表示每英寸点数（`dpi`）的度量，并通过将`300.0`乘以`1.0<dpi>`来创建分辨率。

对于一个更为冗长的替代方案，你可以使用`LanguagePrimitives`模块中的七个`WithMeasure`函数之一。每个`WithMeasure`函数对应于一个测量的原语类型。下面是如何使用`FloatWithMeasure`函数创建一个新的测量值：

```
[<Measure>] type dpi
let resolution = LanguagePrimitives.FloatWithMeasure<dpi> 300.0
```

`WithMeasure`函数在其意图上稍微显得更为明确，并且显然更为冗长。通常，它们的使用保留在类型推断失败时。

## 去除度量

绝大多数函数不接受带有单位的值，因此你可能需要从值中去除度量。幸运的是，像应用度量一样，去除度量也很简单。

去除度量的典型方法是简单地将值除以一个度量为`1`的数值，像这样：

```
[<Measure>] type dpi
300.0<dpi> / 1.0<dpi>
```

另外，你可以使用相应的类型转换运算符来达到相同的效果。例如，我们可以通过调用`float`函数来去除`300.0<dpi>`的单位，如下所示：

```
[<Measure>] type dpi
float 300.0<dpi>
```

## 强制措施

由于度量单位是 F#类型系统的一部分，你可以通过参数上的类型注解来强制传递给函数的值使用正确的单位。在这里，我们定义了一个`getArea`函数，要求传入的宽度和高度必须以英尺为单位：

```
> **let getArea (w : float<ft>) (h : float<ft>) = w * h;;**

val getArea : w:float<ft> -> h:float<ft> -> float<ft ^ 2>
```

如果你使用无单位的参数调用`getArea`，如图所示，你将收到以下错误：

```
> **getArea 10.0 10.0;;**

  getArea 10.0 10.0;;
  --------^^^^

C:\Users\Dave\AppData\Local\Temp\stdin(9,9): error FS0001: This expression was expected to have type
    float<ft>
but here has type
    float
```

同样，如果你使用带有错误度量（或没有度量单位）注解的参数调用`getArea`，将导致编译器错误。要正确调用`getArea`函数，你必须提供正确单位的值，如下所示：

```
> **getArea 10.0<ft> 10.0<ft>;;**
val it : float<ft ^ 2> = 100.0
```

请注意，尽管我们已将`sqft`定义为`ft ^ 2`，但函数的返回值是`float<ft ^ 2>`。编译器不会自动转换度量单位，除非通过返回类型注解明确指示进行转换，如下所示：

```
> **let getArea (w : float<ft>) (h : float<ft>) : float<sqft> = w * h;;**

val getArea : w:float<ft> -> h:float<ft> -> float<sqft>

> **getArea 10.0<ft> 10.0<ft>;;**
val it : float<sqft> = 100.0
```

## 范围

在范围表达式中是允许使用带单位的度量单位的，但有一个限制：你必须提供步长值。要创建带单位的范围，你可以像这样写：

```
> **let measuredRange = [1.0<ft>..1.0<ft>..10.0<ft>];;**

val measuredRange : float<ft> list =
  [1.0; 2.0; 3.0; 4.0; 5.0; 6.0; 7.0; 8.0; 9.0; 10.0]
```

如果没有明确的步长值，编译器将尝试使用底层类型的默认无单位值来创建范围，并会抛出错误。

## 度量单位之间的转换

尽管度量公式允许你创建导出单位，但它们实际上没有足够的灵活性来支持度量单位之间的任意转换。为了绕过这个限制，你可以为度量类型定义静态成员，用于转换因子和函数。

### 静态转换因子

在度量类型上定义转换因子与定义静态属性的语法相同。例如，由于每英尺有 12 英寸，你可以像这样写：

```
[<Measure>] type ft
[<Measure>] type inch = static member perFoot = 12.0<inch/ft>
```

`perFoot`转换可以通过`inch`类型访问，像访问任何静态属性一样。要将英尺转换为英寸，你需要将以英尺为单位的值乘以`inch.perFoot`，如下所示：

```
> **2.0<ft> * inch.perFoot;;**
val it : float<inch> = 24.0
```

注意，编译器如何通过乘法操作推断结果应该以英寸为单位。类似地，我们可以通过将以英寸为单位的值除以`inch.perFoot`来将英寸转换为英尺：

```
> **36.0<inch> / inch.perFoot;;**
val it : float<ft> = 3.0
```

### 静态转换函数

当你需要的不仅仅是转换因子时，你可以直接在度量类型上定义静态转换函数（及其逆转换）。在两个度量类型上始终如一地定义转换函数有助于避免混淆它们的定义位置。

为了最大化代码重用，你可以通过使用`and`关键字将度量类型定义为相互递归的类型。在这里，我们将华氏度和摄氏度的度量定义为相互递归的类型：

```
[<Measure>]
type f =
  static member toCelsius (t : float<f>) = ((float t - 32.0) * (5.0/9.0)) * 1.0<c>
  static member fromCelsius (t : float<c>) = ((float t * (9.0/5.0)) + 32.0) * 1.0<f>
and
  [<Measure>]
  c =
    static member toFahrenheit = f.fromCelsius
    static member fromFahrenheit = f.toCelsius
```

华氏度度量包含用于转换为摄氏度和从摄氏度转换回来的函数。同样，摄氏度度量也包含用于转换为华氏度和从华氏度转换回来的函数，但通过相互递归定义，它可以重用华氏度类型上定义的函数。

根据你的度量定义或转换函数的复杂性，你可能会发现将类型独立定义，然后通过内建类型扩展添加静态方法会更清晰。以下代码片段展示了一种可能的方法：

```
[<Measure>] type f
[<Measure>] type c

let fahrenheitToCelsius (t : float<f>) =
  ((float t - 32.0) * (5.0/9.0)) * 1.0<c>

let celsiusToFahrenheit (t : float<c>) =
  ((float t * (9.0/5.0)) + 32.0) * 1.0<f>

type f with static member toCelsius = fahrenheitToCelsius
            static member fromCelsius = celsiusToFahrenheit

type c with static member toFahrenheit = celsiusToFahrenheit
            static member fromFahrenheit = fahrenheitToCelsius
```

在这里，度量类型是独立定义的（没有相互递归），并紧跟着转换函数。由于转换函数没有附加到度量类型上，我们通过扩展度量类型并添加静态属性来公开这些转换函数。

## 通用度量

你已经看到了许多如何为特定度量类型编写度量感知函数的例子，但也可以使用*通用度量*编写针对任意度量的函数。编写这样的函数与为特定度量类型编写函数相同，只不过你不使用具体的单位值，而是使用下划线字符（`_`）。或者，当你的函数接受多个必须使用相同通用度量类型的参数时，你可以使用通用标识符（例如`'U`）代替下划线。

当你需要针对多种度量执行相同操作时，可能会使用通用度量。例如，你可以编写一个计算任意测量值`float`平方的函数，代码如下：

```
let square (v : float<_>) = v * v
```

因为`square`被定义为使用通用度量，所以它的参数可以接受任何度量类型。事实上，它的参数甚至可以是没有度量的。在这里，我们使用平方函数来计算平方英寸、平方英尺和无度量的平方：

```
> **square 10.0<inch>;;**
val it : float<inch ^ 2> = 100.0
> **square 10.0<ft>;;**
val it : float<ft ^ 2> = 100.0
> **square 10.0;;**
val it : float = 100.0
```

## 自定义度量感知类型

你可以通过定义一个带有`Measure`属性的类型参数来创建你自己的度量感知类型。考虑以下记录类型：

```
type Point< ① [<Measure>] 'u > = { X : ② float<'u>; Y : ③ float<'u> } with
  member ④ this.FindDistance other =
    let deltaX = other.X - this.X
    let deltaY = other.Y - this.Y
    sqrt ((deltaX * deltaX) + (deltaY * deltaY))
```

`Point`类型的行为与其他记录类型相同，只是它的成员被定义为通用度量。`Point`不只是处理没有度量的浮动值，而是包含一个度量`'u`①，`X`②和`Y`③使用此度量。`Point`还定义了一个`FindDistance`函数④，该函数执行度量安全计算，以查找两个点之间的距离。这里我们创建了一个`Point`实例，并对另一个`Point`调用`FindDistance`函数：

```
> **let p = { X = 10.0<inch>; Y = 10.0<inch> }**
**p.FindDistance { X = 20.0<inch>; Y = 15.0<inch> };;**

val p : Point<inch> = {X = 10.0;
                       Y = 10.0;}
val it : float<inch> = 11.18033989
```

如果你尝试用使用不同度量单位的`Point`调用`FindDistance`，编译器会抛出类似这样的类型不匹配错误：

```
> **p.FindDistance { X = 20.0<ft>; Y = 15.0<ft> };;**

  p.FindDistance { X = 20.0<ft>; Y = 15.0<ft> };;
  ---------------------^^^^^^^^

C:\Users\Dave\AppData\Local\Temp\stdin(5,22): error FS0001: Type mismatch. Expecting a
    float<inch>
but given a
    float<ft>
The unit of measure 'inch' does not match the unit of measure 'ft'
```

自定义度量感知类型也不限于记录类型。例如，你可以像这样定义一个等效的度量感知类：

```
type Point< [<Measure>] 'u > (x : float<'u>, y : float<'u>) =
  member this.X = x
  member this.Y = y
  member this.FindDistance (other : Point<'u>) =
    let deltaX = other.X - this.X
    let deltaY = other.Y - this.Y
    sqrt ((deltaX * deltaX) + (deltaY * deltaY))
```

## 总结

大多数编程语言依赖程序员的自律来确保度量单位的正确和一致使用。F# 帮助开发者生成更准确代码的独特方式之一，就是通过在其类型系统中直接包含丰富的度量单位语法。

F# 不仅包括国际单位制（SI）的预定义度量类型，而且还允许你定义自己的度量单位。你可以通过在常量值上添加适当的度量单位注解，或在函数定义中将其包含在类型注解中，从而强制使用正确的度量单位进行计算。最后，你还可以使用类似泛型的语法定义自己的度量单位感知类型。
