# 第八章：8 性能与效率

![](img/opener-img.png)

很少有程序员故意编写低效的代码，但我们并不总是有时间对算法进行精细调整，以提取最大可能的性能。然而，了解某些编码实践如何影响性能，以及如何采取替代方法使代码更高效，依然非常重要。在本章中，我们将仔细分析一些常见的技术和实践，检查它们的性能，并将其特点与潜在的替代方案进行比较。

我们将探讨以下内容：

+   在哪些情况下，默认的代码行为可能不是最优的

+   为什么一些常见的性能问题其实是误解

+   如何评估代码性能并找出瓶颈

+   在进行小幅优化时，可能值得付出一定的努力

## 性能测量与优化

*优化* 这个术语通常指的是修改代码，使程序运行得更快，但我们可能还希望优化其他多种结果：更低的内存使用、更高的数值计算精度、更高的数据吞吐量以及更便捷的部署等。有时候，我们为了代码的可读性或便利性而牺牲纯粹的性能。我们可能会决定，让我们的代码更易于测试比让程序以最大速度运行更为重要。然而，优化某一领域往往会对应用程序的其他领域产生负面影响，因此我们必须确保潜在的收益值得成本，避免我们的努力实际上导致了*恶化*：编写或使用阻碍程序高效运行的代码。

优化程序性能的最简单直接方法是启用构建配置中的优化。发布版本构建配置默认启用优化。而在构建调试配置时，编译器生成的代码与源代码的结构和逻辑非常接近，这样可以设置诊断功能，如断点、逐步调试和检查变量。发布版本中启用的优化可能会以微妙的方式改变代码的逻辑结构，虽然会使调试变得更加困难，但有可能提高代码效率或减小程序体积。

C# 编译器本身几乎不执行代码优化，而是将大部分工作留给 JIT 编译器。

### JIT 编译器

C# 编译器将我们的 C# 代码转换为 CIL 格式，CIL 再通过像 CrossGen 工具这样的工具提前转换为本机机器代码（AOT），或者在运行时通过 JIT 编译器转换，后者是默认方式。在正常操作中，JIT 编译器逐块翻译程序；它不会像 AOT 工具那样在运行程序之前一次性生成整个程序的机器代码，而是*及时*地将 CIL 的部分内容翻译为本机格式。一个部分通常是一个方法，但原则上它也可以是方法的一部分，比如一个循环或 if 块。

由于 JIT 编译器的优化发生在程序执行期间，它们会因平台和运行时环境而异。虽然 AOT 编译可能改善程序的启动时间，但 JIT 编译器可以利用特定 CPU、寄存器集、操作系统和程序状态的优化，动态生成高效的代码。

一种常见的优化方法是将代码内联到方法中，避免方法调用的开销。JIT 编译器也可能能够用本机的内在 CPU 指令替换某些方法调用，从而进一步提高性能。一旦一个代码块被 JIT 编译器翻译，它的本机代码就会保留在内存中，因此如果程序多次运行它，通常不需要重新编译。

在调试版本中，JIT 编译器对其应用的优化要保守得多，以支持正常的调试操作。当我们尝试评估代码性能时，通常最合理的做法是基于发布版本进行评估，这样可以考虑到 JIT 编译器所做的所有优化。

### 性能基准

当我们的代码运行速度比预期慢时，仅仅观察正在运行的应用程序可能会给我们一些启示，但精确测量性能将使我们能够更有效地定位优化工作。

记录代码运行所需时间——无论是完整的端到端运行，还是程序的一部分——被称为*基准测试*。更一般来说，*基准*是用来衡量某物的标准。通过计时我们的代码，我们建立了一个基准，可以用来与新版本进行比较，从而判断我们的修改是让代码更快、更慢，还是没有明显效果。

许多单元测试框架会报告测试运行所需的时间，甚至单个测试的耗时。关注这些数字是非常有价值的，因为突如其来的增加可能表明某处引入了效率问题。这种方法在自动化的*持续集成 (CI)* 服务中尤为重要，其中来自多个贡献者的更改会自动集成到程序中；我们可以设置 CI 服务，当单元测试的时间开始发生变化时给我们发出警报。如果一个通常在几百毫秒内完成的测试开始耗时更长，我们可以集中注意力查看正在测试的代码，看看是否需要进一步调查。

更加细粒度且精确的方法是对代码段本身进行计时。基本原理非常简单：在运行要测量的代码之前，我们创建一个计时器来记录经过的时间，当代码执行完毕时，我们记录计时器的测量值。清单 8-1 展示了一个简单但天真的基准测试，使用了来自System.Diagnostics命名空间的<sup class="SANS_TheSansMonoCd_W5Regular_11">Stopwatch</sup>类。

```
// Start the clock
var clock = Stopwatch.StartNew();
// Run the code to be measured
var result = SomeTask();
// Stop the clock, and record elapsed time
clock.Stop();
var millisecs = clock.ElapsedTicks * 1000.0 / Stopwatch.Frequency;
```

清单 8-1：一种简单的基准测试方法

Stopwatch类是一个轻量级的高分辨率计时器，能够以极高的精度记录经过的时间。Stopwatch.Frequency值表示每秒的计时滴答数，因此通过将经过的滴答数乘以1000.0，再除以频率，我们可以以毫秒为单位报告所花费的时间。这种技术仅仅是测量时钟开始后经过的时间，因此它无法确定所测量的代码是否在那段时间内一直在运行。例如，如果代码被中断（比如切换到其他线程），时钟依然会继续计时。

在代码中加入计时器并将其记录到日志或其他审计轨迹中，可以有效地测量在实时系统中运行的代码性能。然而，测量和报告性能本身也需要时间，因此我们必须确保在相对较高的层次进行测量。例如，测量和报告代码响应一个HTTP 请求或调用远程过程的时间，可能不会显著影响应用程序的性能。另一方面，在紧密循环中使用这种技术，可能会带来比循环本身更大的开销。

基准测试也是在测试环境中探索性能的一种有用方式，可能是用来比较解决特定问题的替代方法。清单 8-1 中的技术是幼稚的，因为它只对代码进行了一次测量。更精确的性能测量方法是多次运行代码，并报告平均时间。我们可以基于清单 8-1 编写自己的框架，尽管一些免费的 C# 库可以帮助我们完成繁重的工作，生成包含记录的性能和其他有用统计数据（如误差范围）的报告。

### 分析器

基准测试将告诉我们代码运行的整体速度，但要确定代码的具体操作，我们需要一个*分析器*。将基准测试工具与分析结合使用将提供最准确的测量。在几种可用的分析器中，最常见的两种是性能分析器和内存分析器。

*内存分析器*将显示我们的程序在哪里分配内存，使用了多少内存，以及何时进行垃圾回收。如果我们需要找出哪些部分的代码使用了最多的 CPU 时间，或者哪些方法被调用得最频繁，*性能分析器*将提供精确的测量，帮助我们定位代码中的热点，并在必要时进行优化。虽然优化内存使用很重要，但在本章中，我们将重点通过使用性能分析器来发现代码中的瓶颈。

性能分析器通常在程序的发布版本上运行，因此会考虑编译器和 JIT 编译器所做的任何优化。测量调试版本的性能通常意义不大，尽管有时它可能会有用：例如，比较同一代码的调试版本和发布版本的分析结果，可以为我们提供 JIT 编译器执行的一些优化的见解。

尽管性能测量可以让我们了解瓶颈可能会在哪些地方拖慢代码，但必须牢记，程序的性能受多种因素的影响，除了代码外，还包括我们使用的 CLR 版本或软件开发工具包（SDK）的版本。即使在同一台机器上运行相同的程序两次，也可能产生不同的结果，这取决于缓存内存的分配方式或 CPU 调度器如何流水线处理指令。JIT 编译器还可能为每次运行应用不同的优化，这可能进一步影响结果。因此，我们必须小心不要过于看重性能分析报告中的绝对时间，而是要寻找趋势或明显的异常，例如结果相差一个数量级或更多。

我们将使用性能分析器选择性地测量代码的特定方面，并分析分析器的结果。请记住，本章中显示的具体结果是针对执行测试的机器特定的，但我们会尝试多种方法，衡量每次尝试的结果，以便我们能够识别出结果中的一些常见、可重复的模式。

为了演示这一点，接下来我们将研究如何仅仅改变字段类型就能显著影响依赖于使用 Equals 的代码性能。

## 使用 Equals 测量基本性能

Equals 方法是 C# 中常被忽视的代码优化方面之一。该方法是性能测量的良好候选，因为它始终可用（因为每种类型都从 object 基类继承它），同时也可以自定义（作为 object 的虚拟成员）。在本节中，我们将测量一个简单值类型的 Equals 的默认行为，以便将分析器的结果与通过我们自己的实现重写 Equals 后的结果进行比较。

结构体类型继承自 ValueType 类的基于值的相等性比较，并覆盖了 object 通用基类中定义的默认实现。这确保了当我们复制结构体的实例时，复制品通过比较每个实例的字段与原始实例相等。我们可能会倾向于依赖这种行为，而不是实现我们自己的 Equals 方法的重写，因为它使我们的类型定义更简短、更简单，就像 Listing 8-2 中的 Color 结构体一样。

```
public readonly struct Color
{
    public Color(int r, int g, int b)
        => (Red, Green, Blue) = (r, g, b);
    public int Red {get;}
    public int Green {get;}
    public int Blue {get;}
}
```

Listing 8-2: 定义一个简单的结构体类型

两个属性值相同的 Color 实例会被认为相等。此外，像所有结构体一样，Color 从 ValueType 继承了基于值的 GetHashCode 实现，确保两个相等的 Color 值始终生成相同的哈希码。此外，Color 是不可变类型，适合用作依赖哈希码效率的数据结构的键。在 列表 8-3 中，我们创建了许多随机的 Color 实例，然后将它们添加到 HashSet 中，进行简单的测试，用以衡量 Color 结构体的性能。

```
var rng = new Random(1);
var items = Enumerable.Range(0, 25000)
    .Select(_ => rng.Next())
    .Select(r => new Color(r >> 16 & 0xFF, r >> 8 & 0xFF, r & 0xFF))
    .ToHashSet();
```

列表 8-3：生成哈希集合

这个 Random 类是标准库的 *伪随机数生成器*，它是一个算法的名称，利用确定性过程生成一系列看似随机的数字。值得注意的是，如果使用相同的 *种子* 初始化，Random 类会生成相同的数字序列——即用于计算序列第一个数字的值。

> 注意

*不同版本的 .NET（或 .NET Core）可能会为给定的种子生成不同的序列。*

在 列表 8-3 中，我们使用 1 作为种子，并使用通过调用随机数生成器的 Next 方法生成的数字来创建新的 Color 实例。由于我们每次都使用相同的种子，因此每次运行代码时都会生成相同的 Color 实例序列。这一特性通常被认为是伪随机数的缺点，但它完全符合我们的需求，因为我们可以多次运行此代码，并且每次运行都会生成相同的 Color 实例值。因此，在比较不同运行的性能时是公平的，因为每次运行都将比较相同的 Color 值序列。我们使用随机生成的序列来确保最终的 HashSet 包含一个合理真实的 Color 值集合。

在列表 8-3 中，我们通过从每个随机数中屏蔽掉<code>Red</code>、<code>Green</code>和<code>Blue</code>值来创建每个<code>Color</code>实例。表 8-1 中的性能分析器输出显示了哈希表构造函数的性能。对于这个测试，我们只是简单地测量每个方法的经过时间，也称为*CPU 采样*。

表 8-1： 创建 HashSet 的性能报告

| 方法 | 时间 (毫秒) | 签名 |
| --- | --- | --- |
| 87.9% HashSet'1..ctor | 50 | System.Collections.Generic.HashSet`1..ctor (IEnumerable, IEqualityComparer) |
| 87.9% UnionWith | 50 | System.Collections.Generic.HashSet`1.UnionWith (IEnumerable) |
| 87.9% AddIfNotPresent | 50 | System.Collections.Generic.HashSet`1.AddIfNot Present(T, out Int32) |
| 36.5% Equals | 21 | System.ValueType.Equals(Object) |
| 14.0% [垃圾回收] | 7.9 |  |

我们专注于<code>HashSet</code>的创建，并忽略其他所有内容，包括随机数生成和单个<code>Color</code>对象的创建。不同的性能分析器以不同的方式展示报告，但呈现的信息通常是相似的。

报告第一列的缩进显示了正在测量的调用栈。第一行中的HashSet构造函数调用了一个名为UnionWith的方法，该方法又调用了AddIfNotPresent。最后这个方法最终调用了Equals方法。输出中的最左边值显示了该方法所花费时间占测试总时间的百分比。在我们的测试中，创建初始的Color值序列占用了剩余的时间，但与测试Equals的方法并不直接相关。接下来的字段是方法的简单名称，后面跟着该方法所用的绝对时间（以毫秒为单位）。

最后，方法的完全限定名表明了具体报告的是哪个方法。由于我们的简单Color结构体没有提供自己的Equals实现，因此输出显示使用了ValueType.Equals来为哈希表添加唯一键。

如前所述，报告的实际毫秒数可能会基于多个因素的组合而有所变化，因此不应字面理解。然而，它们为我们提供了一个基准，可以用来比较其他测试的结果。

### 简化的隐藏成本

我们的Color类型使用三个值表示 RGB 组件。虽然它们被存储在int属性中，每个属性占 4 个字节，但我们通过将每个参数的最低 8 位掩码掉，使用了每个值仅占 1 个字节的方式来存储。我们可能会推测，通过将属性存储为byte字段，而不是int，可以节省存储空间。示例 8-4 显示了修改后的Color结构体。

```
public readonly struct Color
{
    public Color(int r, int g, int b)
        => (Red, Green, Blue) = **((byte)r, (byte)g, (byte)b)**;
 public **byte** Red {get;}
    public **byte** Green {get;}
    public **byte** Blue {get;}
}
```

示例 8-4：存储颜色组件的字节字段

我们仍然允许<sup class="SANS_TheSansMonoCd_W5Regular_11">int</sup>类型的参数传递给<sup class="SANS_TheSansMonoCd_W5Regular_11">Color</sup>构造函数，以确保用户在创建<sup class="SANS_TheSansMonoCd_W5Regular_11">Color</sup>值时无需显式地将参数转换为<sup class="SANS_TheSansMonoCd_W5Regular_11">byte</sup>类型。将<sup class="SANS_TheSansMonoCd_W5Regular_11">int</sup>值显式转换为<sup class="SANS_TheSansMonoCd_W5Regular_11">byte</sup>与我们在清单 8-3 中使用的掩码操作效果相同：值会被截断，只保留最低的 8 位。如果我们在测试中使用这种版本的<sup class="SANS_TheSansMonoCd_W5Regular_11">Color</sup>来生成一个来自清单 8-3 的<sup class="SANS_TheSansMonoCd_W5Regular_11">HashSet</sup>，结果会大不相同。表 8-2 仅显示了<sup class="SANS_TheSansMonoCd_W5Regular_11">AddIfNotPresent</sup>的调用树。

表 8-2： 使用 byte 字段将对象添加到 HashSet的性能报告

| 方法 | 时间（毫秒） | 签名 |
| --- | --- | --- |
| 99.9% AddIfNotPresent | 7,494 | System.Collections.Generic.HashSet`1 .AddIfNotPresent(T, out Int32) |
| 39.6% Equals | 2,967 | System.ValueType.Equals(Object) |
| 8.66% [垃圾回收] | 650 |  |
| 0.16% [线程挂起] | 12 |  |

我们看到代码的执行情况发生了显著变化，AddIfNotPresent方法完成需要超过七秒钟。将此报告与表 8-1 进行比较，我们可以清楚地看到额外时间的主要原因是<sup class="SANS_TheSansMonoCd_W5Regular_11">Equals</sup>方法，该方法由<sup class="SANS_TheSansMonoCd_W5Regular_11">Color</sup>类从<sup class="SANS_TheSansMonoCd_W5Regular_11">ValueType</sup>基类继承。

在某些情况下，ValueType.Equals可以执行*非常*快速的按位比较，但有几个注意事项：如果任何字段是引用类型、浮点数，或者是重写了<sup class="SANS_TheSansMonoCd_W5Regular_11">Equals</sup>的方法类型，那么这个比较就不能使用。两个不同的引用值可能指向具有自己<sup class="SANS_TheSansMonoCd_W5Regular_11">Equals</sup>方法的对象，按位比较会将它们视为不相等，即使<sup class="SANS_TheSansMonoCd_W5Regular_11">Equals</sup>会返回<sup class="SANS_TheSansMonoCd_W5Regular_11">true</sup>。出于同样的原因，任何具有自己<sup class="SANS_TheSansMonoCd_W5Regular_11">Equals</sup>方法的值类型也可能会用该方法将具有不同位模式的值视为相等。两个具有匹配位模式的浮点数不一定相等；特别是如果两个值都是<sup class="SANS_TheSansMonoCd_W5Regular_11">NaN</sup>，它们不应视为相等。

使用快速比较的另一个条件是，结构体必须是*紧凑打包的*，即其字段不需要额外填充就能在内存中正确对齐。在<sup class="SANS_TheSansMonoCd_W5Regular_11">Color</sup>原始实现中的三个<sup class="SANS_TheSansMonoCd_W5Regular_11">int</sup>字段会自动在内存中对齐。然而，使用<sup class="SANS_TheSansMonoCd_W5Regular_11">byte</sup>代替意味着字段不再是紧凑打包的，因此必须使用另一种速度较慢的比较，这会带来显著的性能损失，见表 8-2。

### ValueType.Equals 方法

当快速的按位比较不可用时，ValueType中<sup class="SANS_TheSansMonoCd_W5Regular_11">Equals</sup>的实现必然是非常通用的，因为它必须适用于任何结构体类型，无论结构体有多少个字段，或字段的类型是什么。除了拥有内建原始类型的字段，结构体还可以包含对类实例和其他用户定义值的引用，而这些值可能有自己的自定义<sup class="SANS_TheSansMonoCd_W5Regular_11">Equals</sup>实现。

ValueType.Equals 方法的实现首先必须确定需要比较哪些字段。它通过使用 *反射*——以编程方式检查（或修改）程序的运行时结构——来发现所有实例字段，这立即带来了相当显著的运行时开销。反射通常与高性能算法无关，这也确实解释了由于我们将结构中的 int 字段改为使用 byte，导致的性能下降。

在确定字段数组之后，ValueType.Equals 会获取每个字段的值。如果字段值不是 null 引用，则会调用其 Equals 方法，并使用来自与之比较的结构中的相应字段的值。因此，两个结构中每个值类型的字段都会进行装箱以执行比较，因为使用反射获取值意味着每个值都是通过 object 引用来访问的，这进一步增加了成本。

我们性能问题的根本原因是，从使用 int 属性改为使用 byte 值意味着 Color 的底层字段不再紧密打包。因此，ValueType.Equals 无法使用快速的按位比较，而是使用反射来发现需要比较的值。为了解决这个问题，在 Listing 8-5 中，我们重写了 Equals 方法，并提供了我们自己的实现来比较属性值。

```
public readonly struct Color
{
    public Color(int r, int g, int b)
        => (Red, Green, Blue) = ((byte)r, (byte)g, (byte)b);
    public byte Red {get;}
    public byte Green {get;}
    public byte Blue {get;}
    **public override bool Equals(object? obj)**
        **=>** **obj is Color other &&**
           **Red** **==** **other.Red && Green** **==** **other.Green && Blue** **==** **other.Blue;**
}
```

Listing 8-5: 重写 Equals 方法

从重新运行测试的报告中，见 Table 8-3，虽然我们已经大幅提高了速度，但仍然有进一步改进的空间。

Table 8-3: 重写的 Equals 方法的性能

| 方法 | 时间（毫秒） | 签名 |
| --- | --- | --- |
| 100% AddIfNotPresent | 2,889 | System.Collections.Generic.HashSet`1 .AddIfNotPresent(T, out Int32) |
| 20.4% Equals | 588 | Color.Equals(Object) |
| 8.15% [垃圾回收] | 236 |  |

请注意，我们重写的Equals方法消耗的时间占总时间的比例远小于<sup>AddIfNotPresent</sup>方法的时间，尽管这种方式仍然比我们使用原始版本的Color（该版本使用了int属性）时要慢得多。

这份报告告诉我们，大部分时间花费在了<sup>AddIfNotPresent</sup>方法上，而不是它所调用的其他方法。为了发现原因，我们将使用另一种类型的性能分析，有时被称为*插装分析*，或*跟踪*，它记录程序中每个方法的调用次数。因为这要求分析工具以侵入性方式测量正在运行的程序，所以时间测量通常要高得多；然而，知道哪些方法被调用得最多是非常有价值的信息。表 8-4 展示了<sup>AddIfNotPresent</sup>方法及其内部调用的跟踪报告，包括每个方法的调用次数。

表 8-4: Equals 的跟踪报告 Equals

| 方法 | 时间 (毫秒) | 调用次数 | 签名 |
| --- | --- | --- | --- |
| 99.9% AddIfNotPresent | 16,681 | 25,000 | [...] |
| 40.3% Equals | 6,724 | 312,222,485 | Color.Equals(Object) |
| 1.76% [垃圾回收] | 293 | 1,593 |  |

本报告有一个额外的列，显示程序执行过程中每个方法被调用的次数。跟踪报告运行时间显著较长，但更重要的是，它显示了Equals方法被调用了大量次。事实上，Equals的调用次数与 25,000 的三角形数字非常接近——这就是原始序列中的元素数量。某个数字*n*的*三角形数字*是从 1 到*n*的所有整数之和。当*n*为 25,000 时，三角形数字为 312,512,500。

虽然我们为<Color>结构定制了Equals方法，但HashSet类在添加或查找键时也会使用GetHashCode，而我们的<Color>类型依赖于从ValueType继承的默认GetHashCode实现。让我们看看这与在我们的测试中调用Equals方法的次数有何关系。

### ValueType.GetHashCode 方法

如第五章所解释，HashSet中的元素是唯一的；表中的每个键只出现一次。只有当新对象在表中不存在时，才会将其添加到HashSet中；否则，它将被忽略。

当我们在这个例子中向HashSet添加一个项目时，实施方法使用GetHashCode来识别具有相同哈希码的现有键。哈希码相同并不一定意味着任何现有键的值与新项相同。如果没有现有键的哈希码与新项相同，则新对象将被添加到表中。如果一个或多个现有键的哈希码与新项的哈希码匹配，则会使用Equals方法来确定是否应该添加该项。每个与新项哈希码相同的键都会与新项逐一进行比较，如果没有找到匹配项，则将新项作为新键添加到表中。

我们的测试中经常调用 Equals，这表明我们为 Color 类型实现的 GetHashCode 方法生成的哈希码分布不均匀。当第一个元素被添加到哈希表时，根本不会调用 Equals，因为没有可比较的元素。如果第二个元素的哈希码与第一个元素相同，则会调用 Equals 来确定它们是否是相同的键。如果随后的元素的哈希码与已存在的键相同，这个过程会重复进行。

如果初始序列中的 25,000 个 Color 对象生成相同的哈希码但值不同，那么添加最后一个新元素时，需要对所有现有的 24,999 个键调用 Equals。

实际上，ValueType.GetHashCode 的默认实现被 Color 结构体继承时，可能会生成许多相同的哈希码，而不管 Color 实例的值是否不同。原因与 ValueType 提供的 Equals 实现的性能较差有关，这也解释了为什么调用 Equals 的次数接近于序列长度的三角数。

如果一个结构体的实例可以通过快速的位运算比较（使用 Equals）进行比较，那么 ValueType.GetHashCode 方法将根据实例在内存中的位模式生成哈希码。另一方面，如果该结构体不适合进行快速的位运算比较，默认的 GetHashCode 实现则只考虑结构体的第一个非 null 实例字段——在我们的 Color 类型中是 Red 属性——因此最多只能生成 256 个唯一的哈希码。我们通过实现自己的 GetHashCode 方法来解决这个问题，生成更多的唯一哈希码，最好是每个不同的 Color 值生成一个唯一的哈希码。

### HashCode.Combine 方法

在 列表 8-6 中，我们为 Color 结构添加了自定义的 GetHashCode 重写方法，以补充我们重写的 Equals 方法，并通过使用标准库中的 HashCode.Combine 方法来实现新的 GetHashCode 方法。

```
public override bool Equals(object? obj)
    => obj is Color other &&
       Red == other.Red && Green == other.Green && Blue == other.Blue;
public override int GetHashCode()
    => HashCode.Combine(Red, Green, Blue);
```

列表 8-6：重写 GetHashCode 方法

Combine 方法基于输入生成分布均匀的哈希码，尽管我们可能能够编写自己精心优化的替代方法，但这么做远非易事。现在，当我们运行测试时，发现同时重写 Equals 和 GetHashCode 方法的综合效果大大减少了对 Equals 方法的调用次数，如 表 8-5 所示。

表 8-5： 重写 GetHashCode 的跟踪报告

| 方法 | 时间（毫秒） | 调用次数 | 签名 |
| --- | --- | --- | --- |
| 48.8% AddIfNotPresent | 16 | 25,000 | [...] |
| 38.1% Combine | 12 | 25,000 | System.HashCode.Combine(T1, T2, T3) |
| 1.42% Resize | 0.5 | 12 | System.Collections.Generic.HashSet`1.Resize(Int32, Boolean) |
| 0.27% Equals | 0.09 | 18 | Color.Equals(Object) |

即便考虑到方法调用的开销，本报告显示与我们之前的结果相比，速度有了显著提升，并且展示了Equals和GetHashCode之间的紧密关系。如果我们接受由ValueType提供的默认行为，而不是在自定义结构体类型中实现这些方法，我们的效率将会付出较高的代价。

如果我们重新审视原始结构体的分析，原始结构体中有int字段，但没有方法重写，可以看到即使该结构体可以高效地打包，Equals方法仍然比我们最新版本中调用得更频繁（参见表 8-6）。

表 8-6： 无重写的打包结构体跟踪报告

| 方法 | 时间（毫秒） | 调用次数 | 签名 |
| --- | --- | --- | --- |
| 85.6% AddIfNotPresent | 101 | 25,000 | [...] |
| 30.1% Equals | 36 | 1,219,104 | System.ValueType.Equals(Object) |
| 7.54% [垃圾回收] | 8.9 | 17 |  |
| 0.42% 调整大小 | 0.5 | 12 | System.Collections.Generic.HashSet`1 .Resize(Int32, Boolean) |

如果我们增加添加到HashSet中的元素数量，肯定会注意到性能问题。

除了 HashSet，还有几种其他集合类型依赖于哈希码来提高效率，包括 Dictionary 和 Lookup 类型。因此，对于任何可能作为哈希集合的键使用的类型，重写 Equals 和 GetHashCode 方法是至关重要的。

## 优化相等性

重写 Equals 和 GetHashCode 可以带来最显著的性能提升，但我们还可以做更多工作来微调相等性比较。毕竟，Equals 方法不仅在创建依赖于哈希码的数据结构时会使用。

我们的 Color 结构体是一个相对简单的数据类型，其 Equals 方法已经非常高效。为了探讨 Equals 的特性，我们将创建一个更加复杂的 Purchase 值类型，如 列表 8-7 所示。Purchase 结构体重写了 Equals 和 GetHashCode 方法，并提供了自定义实现，但还没有实现 IEquatable< Purchase> 接口。我们稍后会为 Purchase 实现该接口，看看它如何影响 Equals 的性能。

```
public readonly struct Purchase
{
    public Purchase(Product item, DateTime ordered, int quantity)
        => (Item, Ordered, Quantity) = (item, ordered, quantity);
    **public Product   Item {get;}**
    public DateTime  Ordered {get;}
    public int       Quantity {get;}
    public override bool Equals(object? obj)
        => obj is Purchase other &&
           Item.Equals(other.Item) &&
           Ordered == other.Ordered && Quantity == other.Quantity;
    public override int GetHashCode()
        => HashCode.Combine(Item, Ordered, Quantity);
}
```

列表 8-7：定义一个更复杂的数据类型，Purchase

Purchase 类型有三个字段，其中一个是名为 Product 的非平凡类型，如下所示：

```
public readonly struct Product
{
    public Product(int id, decimal price, string name)
        => (Id, Price, Name) = (id, price, name);
    public int     Id {get;}
    public decimal Price {get;}
    public string  Name {get;}
    public override bool Equals(object? obj)
        => obj is Product other &&
           Id == other.Id && Price == other.Price && Name == other.Name;
    public override int GetHashCode()
        => HashCode.Combine(Id, Price, Name);
}
```

Equals 方法在 Purchase 类型中需要做比 Color 类型中的 Equals 方法更多的工作，如 示例 8-5 所示。当我们比较两个 Purchase 实例的相等性时，Equals 方法还必须确保 Item 属性匹配，这涉及到对 Product.Equals 方法的调用。

> 注意

*Purchase 类型相当大——假设是 64 位架构，则为 40 字节加上填充——因此我们应该预期复制实例的效率低于较小的 Color 类型。不过，这不会影响我们的性能分析，因为我们仍然会比较相同类型的报告。我们将在《复制大型实例》一节中返回讨论复制大型结构实例的成本，见 第 272 页。*

我们将使用 SequenceEqual 方法来比较两个非常大的 Purchase 对象列表，而不是使用 HashSet，如 示例 8-8 所示。此过程将测试 Equals 方法，从而允许我们衡量它的效率。为了放大 Equals 方法与周围代码开销的性能对比，我们将元素数量增加到 1000 万。

```
var items = Enumerable.Range(0, 10_000_000)
    .Select(id => new Purchase(new Product(id, id, "Some Description"),
                               DateTime.MinValue, id))
    .ToList();
Assert.That(items.SequenceEqual(items), Is.True);
```

示例 8-8：测试以练习相等性

在 Enumerable.Range 方法中，我们使用自 C# v7.0 起提供的数字分隔符，使得大型字面量数字更易于人类阅读和解析。数字分隔符对编译器没有影响：我们用于初始序列长度的数字仍然是一个普通的 int 值。

SequenceEqual 方法比较两个序列，如果它们的元素按相同顺序排列，则返回 true。该算法从每个序列中获取一个元素，并通过使用 Equals 方法进行比较。SequenceEqual 不会通过检查这两个序列是否实际上是 *相同* 的序列来优化其结果，因此在这里我们只创建一个包含 1000 万个元素的序列，并将其与自身进行比较。表 8-7 显示了调用 SequenceEqual 的性能分析报告。

表 8-7: 执行 Equals 方法

| 方法 | 时间 (毫秒) | 签名 |
| --- | --- | --- |
| 77.5% SequenceEqual | 1,227 | System.Linq.Enumerable .SequenceEqual(IEnumerable, IEnumerable) |
| 49.3% 等于 | 781 | Purchase.Equals(Object) |
| 24.3% [垃圾回收] | 384 |  |
| 10.6% 等于 | 168 | Product.Equals(Object) |
| 0.75% get_Item | 12 | Purchase.get_Item() |
| 0.38% 解包 | 6.0 | System.Runtime.CompilerServices .CastHelpers.Unbox(Void*, Object) |

我们可以看到，垃圾回收在 Equals 所需的时间中占有重要部分。每次调用带有 Purchase 实例的 Equals 方法都会导致参数被装箱，因为 Purchase 是一个结构体，而 Equals 重载的参数类型是 object，即引用类型。此外，Purchase.Equals 方法会调用 Product.Equals，这也需要将其参数装箱。结果是我们在堆上分配了许多装箱对象，这给垃圾回收器带来了相当大的压力，以保持内存使用的控制。

在每个 Equals 方法中，需要将参数拆箱回其原始类型，以便比较其属性；拆箱 object 参数的成本对于每个 Equals 方法来说非常小，但会产生可衡量的影响。通过为 Purchase 和 Product 类型实现 IEquatable< T >，我们可以避免装箱的成本，以及垃圾回收的相关成本。

### IEquatable<T> 的效果

SequenceEqual 方法会自动选择可用的最佳（最有效的）Equals 实现来执行比较。内部，SequenceEqual 使用 第五章 中的 EqualityComparer 辅助类来确定如何比较元素。如果元素类型 T 实现了 IEquatable< T >，则保证实现了类型安全的 Equals 重载，并且该重载会被 SequenceEqual 调用。

如果我们实现 IEquatable< Purchase> 接口并提供我们自己的类型安全的 Equals 重载方法，SequenceEqual 方法将默认使用 IEquatable< Purchase> 接口的方法，从而避免了拆箱和重新拆箱 Equals 参数的需求。这样，内存压力得到减轻，因为参数不会被复制到堆上，从而减少了垃圾回收器需要检查的对象数量。在我们的示例中，这些减少是相当可观的，因此实现 IEquatable< Purchase> 接口应该带来可衡量的好处。清单 8-9 显示了 Purchase 中所需的更改。

```
public readonly struct Purchase : IEquatable<Purchase>
{
    `--snip--`
    public bool Equals(Purchase other)
        => Item.Equals(other.Item) &&
           Ordered == other.Ordered && Quantity == other.Quantity;
    public override bool Equals(object? obj)
        => obj is Purchase other && Equals(other);
}
```

清单 8-9：IEquatable< Purchase> 实现

我们添加了一个 Equals(Purchase other) 重载方法，用于比较每个属性值。原始的 Equals 重写方法仍然需要对其 object 参数进行拆箱，以便调用类型安全的 Equals 重载方法，但 SequenceEqual 方法不会调用 Equals(object?)，因为我们还修改了 Purchase 的声明，使其实现了 IEquatable< Purchase> 接口。在 清单 8-10 中，我们对 Product 进行了类似的更改，以便从 Purchase.Equals 方法调用 Product.Equals 时，不需要对 Product 实例进行拆箱。

```
public readonly struct Product : IEquatable<Product>
{
    `--snip--`
    public bool Equals(Product other)
        => Id == other.Id && Price == other.Price && Name == other.Name;
    public override bool Equals(object? obj)
        => obj is Product other && Equals(other);
}
```

清单 8-10：实现 IEquatable< Product>

结合了 清单 8-9 和 8-10 中的更改后，从 清单 8-8 进行的测试结果，仍然是 1000 万个 Purchase 元素，显示在 表格 8-8 中。

表格 8-8： 衡量类型安全的 Equals 方法

| 方法 | 时间（毫秒） | 签名 |
| --- | --- | --- |
| 62.6% SequenceEqual | 546 | System.Linq.Enumerable.SequenceEqual(IEnumerable, IEnumerable) |
| 13.0% 等于 | 114 | Purchase.Equals(Purchase) |
| 5.48% 等于 | 48 | Product.Equals(Product) |
| 2.05% op_Equality | 18 | System.DateTime.op_Equality(DateTime, DateTime) |
| 1.37% get_Ordered | 12 | Purchase.get_Ordered() |
| 1.37% get_Item | 12 | Purchase.get_Item() |

对比此报告与表 8-7，我们可以看到，SequenceEqual的总时间大幅减少，但也能看出，我们的新Equals方法明显比原本没有类型安全实现的Purchase类型版本运行得更快，后者未实现类型安全的IEquatable<Purchase>。其中很大一部分差异要归功于没有垃圾回收，但我们也从去除打包和解包Purchase和Product值的需求中获益。

### 属性访问

我们的 Equals(Purchase) 方法在比较属性时花费了可测量的时间。Purchase 和 Product 的所有属性都是自动属性，每次访问这些属性都会调用一个方法——例如，访问 get_Item 和 get_Ordered 方法，如表 8-8 所示。尽管 JIT 编译器通常能够通过内联底层方法来优化这些调用，但不能保证它一定会这样做。在清单 8-11 中，我们将 Purchase 修改为引入我们自己的私有字段，并修改 Equals 方法直接比较字段，而不是通过访问属性值来进行比较。

```
public readonly struct Purchase : IEquatable<Purchase>
{
    public Purchase(Product item, DateTime ordered, int quantity)
        => (this.item, this.ordered, this.quantity) = (item, ordered, quantity);
    public Product  Item => item;
    public DateTime Ordered => ordered;
    public int      Quantity => quantity;
    public bool Equals(Purchase other)
        => item.Equals(other.item) &&
           **ordered** **==** **other.ordered && quantity** **==** **other.quantity;**
    public override bool Equals(object? obj)
        => obj is Purchase other && Equals(other);
    public override int GetHashCode()
        => HashCode.Combine(item, ordered, quantity);
    **private readonly Product item;**
    **private readonly DateTime ordered;**
    **private readonly int quantity;**
}
```

清单 8-11：比较字段而不是属性

虽然此处未显示，我们还修改了 Product，将其自动属性替换为私有字段。表 8-9 展示了我们之前同样方式比较 1 千万个元素的结果。

表 8-9： 字段与属性性能比较

| 方法 | 时间 (毫秒) | 签名 |
| --- | --- | --- |
| 51.2% SequenceEqual | 442 | System.Linq.Enumerable .SequenceEqual(IEnumerable, IEnumerable) |
| 9.73% Equals | 84 | Purchase.Equals(Purchase) |
| 3.47% Equals | 30 | Product.Equals(Product) |
| 1.41% op_Equality | 12 | System.DateTime.op_Equality(DateTime, DateTime) |

虽然将自动属性替换为字段显示出了一些小的改进，但这只是一个微优化的例子。与未实现 IEquatable< Purchase> 的版本相比，我们将 SequenceEqual 的执行时间减少了一半以上，但我们仍然只是在谈论几百毫秒的绝对时间。我们不得不显著增加序列的大小，才能放大结果到足够观察到的程度，而大多数应用程序通常不需要比较 1000 万个元素的列表。

实现 IEquatable< T > 接口是一个更重要的步骤。这样不仅能提高速度，我们的类型还能更高效地使用内存，因为不再需要对 Equals 方法的参数进行装箱。对于值类型来说，实施 IEquatable< T > 不仅仅是性能优化，它还证明我们的类型遵循该协议，使得某些库功能能更高效地运作，同时也向人类读者传达了效率的信息。

### 相等操作符

实现类型的完整相等比较的最后一步是编写我们自己的 operator== 及其伴随的 operator!=。清单 8-12 展示了这些操作符是如何为 Purchase 类型实现的。

```
public readonly struct Purchase : IEquatable<Purchase>
{
    `--snip--`
    public bool Equals(Purchase other)
        => item == other.item &&
           ordered == other.ordered && quantity == other.quantity;
    public static bool operator==(Purchase left, Purchase right)
        => left.Equals(right);
    public static bool operator!=(Purchase left, Purchase right)
        => !left.Equals(right);
}
```

清单 8-12：为 Purchase 实现相等操作符

再次，我们还将相等操作符添加到 Product 类型（未展示），使我们可以通过使用 == 来比较 Purchase 类型中的 item 字段，而不必调用其 Equals 方法。每个操作符实现都简单地转发到我们类型安全的 Equals 方法，在该方法中执行比较。

虽然我们可以编写测试来调用 operator== 来衡量其性能特性，但我们也可以通过提供我们自己的相等比较器，安排 SequenceEqual 方法调用该操作符，而不是调用 Equals。

#### 泛型 IEqualityComparer<T> 接口

SequenceEqual 方法不会直接在序列元素上调用 Equals 进行比较。相反，它依赖于 IEqualityComparer< T > 的实现，这部分属于标准库并声明在 System.Collections.Generic 命名空间中。

IEqualityComparer< T > 的实现需要一个 Equals 方法，该方法接受两个类型为 T 的参数，以及一个带有单个 T 参数的 GetHashCode 方法。标准库提供了一些 IEqualityComparer< T > 的默认实现，包括一个用于实现了 IEquatable< T > 接口的 T 实例的实现，这也是我们到目前为止使用 SequenceEqual 时所依赖的。

SequenceEqual 方法有一个重载，接受第二个参数，其类型为 IEqualityComparer< T >，因此我们可以提供自己的实现来替代默认的比较器。在示例 8-13 中，我们创建了自己的 IEqualityComparer< T > 接口的实现，使用 Purchase 作为泛型参数，并将自定义比较器的实例传递给 SequenceEqual。

```
public sealed class EqualsOperatorComparer : IEqualityComparer<Purchase>
{
    **public bool Equals(Purchase x, Purchase y)**
        **=>** **x** **==** **y;**
    public int GetHashCode(Purchase obj)
        => obj.GetHashCode();
}
var items = Enumerable.Range(0, 10_000_000)
    .Select(MakePurchase)
    .ToList();
Assert.That(items.SequenceEqual(items, **new EqualsOperatorComparer()**));
```

示例 8-13：创建自定义 IEqualityComparer< T > 实现

我们对 IEqualityComparer< Purchase> 的实现定义了其 Equals 方法，通过 == 来比较两个参数值，而不是使用参数类型的 Equals 方法。我们不需要为 Product 编写单独的实现，因为 Purchase 中的 Equals 成员方法直接使用 == 来比较 Product 值。现在，当我们使用 SequenceEqual 来比较两个 Purchase 序列时，算法将使用 operator== 进行比较。表格 8-10 显示了比较 1000 万个 Purchase 项目的分析报告。

表格 8-10： 如何 operator== 执行

| 方法 | 时间 (毫秒) | 签名 |
| --- | --- | --- |
| 48.8% SequenceEqual | 475 | System.Linq.Enumerable.SequenceEqual [...] |
| 22.2% Equals | 216 | EqualsOperatorComparer .Equals(Purchase, Purchase) |
| 9.28% op_Equality | 90 | Purchase.op_Equality(Purchase, Purchase) |
| 9.28% Equals | 90 | Purchase.Equals(Purchase) |
| 5.53% op_Equality | 54 | Product.op_Equality(Product, Product) |
| 3.69% Equals | 36 | Product.Equals(Product) |

当我们为任何类型定义operator==时，编译器将其转换为一个名为op_Equality的静态方法，如此性能分析报告中所示。该方法按值传递其两个参数，因此我们实际上会复制多个Purchase和Product实例。我们可以通过将operator==方法改为按引用传递参数来减少需要复制的次数。

#### 只读参数

为了利用将operator==方法改为按引用传递其参数而不是按值传递的好处，我们可以使用只读in参数。它们专门用于避免复制大型值类型实例，并且适用于我们不需要修改参数变量的情况。

然而，我们不应指望会有巨大的改进，因为在比较我们序列中的Purchase元素时，我们无法避免所有复制的发生。特别是，EqualsOperatorComparer.Equals方法必须按值传递其参数，以匹配IEqualityComparer< T >接口中定义的签名。

类似地，如列表 8-14 所示，在<sup class="SANS_TheSansMonoCd_W5Regular_11">Purchase中定义的类型安全的Equals方法按照IEquatable< T >接口的要求按值传递其参数，但我们可以添加一个新的重载的Equals方法，使用in参数，并使用相同的机制来修改相等操作符，使它们按引用传递所有参数。

```
public readonly struct Purchase : IEquatable<Purchase>
{
    `--snip--`
    **public bool Equals(in Purchase other)**
        **=>** **item** **==** **other.item &&**
           **ordered** **==** **other.ordered && quantity** **==** **other.quantity;**
    public bool Equals(Purchase other)
        => Equals(in other);
    public static bool operator==(**in Purchase left, in Purchase right**)
        => **left.Equals(in right);**
    public static bool operator!=(in Purchase left, in Purchase right)
        => !left.Equals(in right);
}
```

列表 8-14: 使用 in 参数的重载

我们将带有 in 参数的 Equals 方法作为主要实现，并从等式运算符和 IEquatable< Purchase> 的实现中转发到它。尽管 in 参数对调用代码是透明的，但重载规则将优先选择没有参数修饰符的方法，除非我们在调用方法时将 in 修饰符添加到参数。因此，我们通过在调用 Equals 时向传递的参数添加 in 关键字，显式选择具有 in 参数的重载。

> 注意

*将值参数替换为 in 参数是一个破坏版本的更改，如果二进制兼容性是一个考虑因素，则需要格外小心。*

我们无需更改 EqualsOperatorComparer 的实现以通过引用传递参数，因为我们的 operator== 方法没有接受值传递的重载。我们可以重用来自列表 8-13 的 EqualsOperatorComparer 来运行测试，结果显示在表格 8-11 中。

表 8-11： 通过引用传递给 operator== 的结果

| 方法 | 时间 (毫秒) | 签名 |
| --- | --- | --- |
| 45.1% SequenceEqual | 437 | System.Linq.Enumerable.SequenceEqual [...] |
| 20.9% 等于 | 203 | EqualsOperatorComparer.Equals(Purchase, Purchase) |
| 10.5% op_Equality | 102 | Purchase.op_Equality(in Purchase, in Purchase) |
| 9.23% 等于 | 90 | Purchase.Equals(in Purchase) |
| 7.38% op_Equality | 72 | Product.op_Equality(in Product, in Product) |
| 7.38% Equals | 72 | Product.Equals(in Product) |

将这些结果与表 8-10 进行比较，我们可以看到改进是相当有限的。虽然我们确实从避免复制Purchase对象中受益，但这种好处仅限于实际调用operator==。 表 8-12 展示了一个追踪报告，列出了方法调用的次数，显示 JIT 编译器将除了少数几个调用外，所有对operator==的调用都内联了。

表 8-12： 比较的追踪报告 在 参数值

| 方法 | 时间（毫秒） | 调用次数 | 签名 |
| --- | --- | --- | --- |
| 1.88% SequenceEqual | 2,013 | 1 次调用 | System.Linq.Enumerable.SequenceEqual [...] |
| 0.69% Equals | 735 | 10,000,000 次调用 | EqualsOperatorComparer.Equals(Purchase, Purchase) |
| [...] |  |  |  |
| 0.08% op_Equality | 82 | 126,402 次调用 | Purchase.op_Equality(in Purchase, in Purchase) |

尽管在定义<code class="SANS_TheSansMonoCd_W5Regular_11">operator==</code>时使用<code class="SANS_TheSansMonoCd_W5Regular_11">in</code>参数是免费的，因为它不需要修改调用代码，但我们不应期望它带来太多的好处。我们也不应当在使用<code class="SANS_TheSansMonoCd_W5Regular_11">in</code>参数时简单地例行公事，即使使用它不会影响方法的可读性。通过引用传递小的值类型可能会产生额外的开销，因为访问值本身需要通过引用变量进行额外的间接访问。与任何代码优化特性一样，我们只有在测量表明有必要时，才应该引入<code class="SANS_TheSansMonoCd_W5Regular_11">in</code>参数。

## <code class="SANS_Futura_Std_Bold_B_11">类型如何影响性能</code>

在应用程序中选择类型可能会以各种方式影响其整体性能。我们用来表示应用程序中值的类型是这种选择中最重要的部分，因为其他类型通常无论如何都会是类。另一方面，值可以通过结构体、类、记录或记录结构体来表示。在本节中，我们将探讨一些有助于我们决定使用结构类型还是类类型来实现这些值类型的因素，以及这些因素对性能的影响程度。

我们常常听说结构体，因此记录结构体，应该尽量小，因为在内存中复制大型实例是昂贵的。考虑到这一点，我们将首先尝试将复制实例的开销与其他影响性能的因素分开。

### <code class="SANS_Futura_Std_Bold_Condensed_Oblique_BI_11">测量复制的开销</code>

与我们之前的性能测量一样，我们需要建立一个简单的基准线，以便可以将进一步的性能报告与之进行比较。由于我们希望测量复制大值类型的开销，因此首先需要测量复制一个小而简单的类型的开销，比如我们在清单 8-15 中创建的<code class="SANS_TheSansMonoCd_W5Regular_11">IntField</code>结构体。

```
public readonly struct IntField : IEquatable<IntField>
{
    public IntField(int value)
        => this.value = value;
    public bool Equals(IntField other)
        => value == other.value;
    private readonly int value;
}
```

<code class="SANS_Futura_Std_Book_Oblique_I_11">清单 8-15：创建一个具有单个 int 字段的简单结构体</code>

为了测试复制，我们将再次使用<code class="SANS_TheSansMonoCd_W5Regular_11">SequenceEqual</code>方法，它将从序列中复制元素进行比较，并再次复制它们以调用<code class="SANS_TheSansMonoCd_W5Regular_11">IEqualityComparer<T></code><code class="SANS_TheSansMonoCd_W5Regular_11">.Equals</code>方法。在这里，我们返回使用默认的相等比较器，它将调用我们的类型安全的<code class="SANS_TheSansMonoCd_W5Regular_11">Equals</code>方法，并按值传递其参数。清单 8-16 展示了我们将用来生成基准性能分析的代码。

```
var items = Enumerable.Range(0, 10_000_000)
    .Select(i => new IntField(i))
    .ToList();
Assert.That(items.SequenceEqual(items));
```

示例 8-16：测试简单的复制

对于这个测试，我们将分析此代码的调试版本，以尽量减少 JIT 编译器进行方法内联的影响。只有在方法正常调用时，方法参数才会被复制，而内联会使得测量这些复制的成本变得不可靠；两次不同的代码运行可能会产生不同的复制次数。表 8-13 展示了在调试版本中比较两个包含 1000 万个IntField项的序列的 CPU 采样报告，这会抑制 JIT 编译器进行方法调用的内联。

表 8-13： 测量复制简单结构体的成本

| 方法 | 时间 (ms) | 签名 |
| --- | --- | --- |
| 57.0% SequenceEqual | 90 | System.Linq.Enumerable .SequenceEqual(IEnumerable, IEnumerable) |
| 7.60% Equals | 12 | IntField.Equals(IntField) |

SequenceEqual算法所做的事情很少，它仅仅是从每个序列中获取一个元素，并使用Equals将它们进行比较。这里Equals所花的时间与SequenceEqual的总时间之间的差异，都是开销，表示获取每对元素以及为Equals复制参数所花费的时间。

### 复制大型实例

复制一个简单的结构体类型，如列表 8-15 中的 IntField 结构体，和复制一个普通的 int 值一样，并不会增加额外的开销。一个简单的测试（这里未展示）可以验证这一点，通过比较两个 int 值的序列。IntPlus3x16 结构体，位于列表 8-17，它添加了三个完全冗余的 Guid 字段，比 IntField 结构体要大得多。每个 Guid 字段占用 16 字节，因此这个结构体的大小甚至比推荐的值类型大小限制还要大。

```
public readonly struct IntPlus3x16 : IEquatable<IntPlus3x16>
{
    public IntPlus3x16(int value)
        => this.value = value;
    public bool Equals(IntPlus3x16 other)
        => value == other.value;
    private readonly int value;
    **private readonly Guid _padding1** **=** **Guid.Empty;**
    **private readonly Guid _padding2** **=** **Guid.Empty;**
    **private readonly Guid _padding3** **=** **Guid.Empty;**
}
```

列表 8-17：创建一个极大的结构体

注意 IntPlus3x16 结构体的一个细节：Equals 方法不会考虑类型中的任何 Guid 字段，因为它们在任何情况下都是相同的。原因在于我们只是想衡量复制的成本，因此这个 Equals 方法与列表 8-15 中的 IntField 类型执行的是完全相同的操作。虽然填充字段在 Equals 方法或其他任何操作中不起作用，但由于 IntPlus3x16 类型是一个结构体，因此按值复制它时，*每个* 字段都会被复制。我们从列表 8-16 中运行相同的测试，结果显示在表 8-14 中。

表 8-14： 衡量复制超大结构体的成本

| 方法 | 时间 (ms) | 签名 |
| --- | --- | --- |
| 52.5% SequenceEqual | 228 | System.Linq.Enumerable .SequenceEqual(IEnumerable, IEnumerable) |
| 2.71% Equals | 12 | IntPlus3x16.Equals(IntPlus3x16) |

对比 表 8-14 和 表 8-13：两份报告中 Equals 方法所花费的时间相同，尽管 SequenceEqual 方法的执行时间是其两倍以上，因为复制较大的 IntPlus3x16 类型实例的额外开销。在两次测试中，Equals 方法执行的是相同的操作，因此时间的增加完全是由于复制实例的成本。

### 衡量对象构造成本

复制大型结构体的成本并不是使用具有多个字段的类型时需要考虑的唯一因素。首先，相等比较通常会考虑每个字段或属性，这使得这些比较比只有一个或两个字段的类型更为昂贵。初始化一个具有多个字段的类型实例同样会产生额外的开销。

列表 8-18 中的 Purchase 和 Product 类型是我们在 列表 8-7 中定义的 Purchase 和 Product 结构体的定位记录结构等效体。由于它们是记录结构类型，编译器会自动生成所有的相等比较，使得它们比结构体版本更易于定义。

```
public readonly record struct Product
    (int Id, decimal Price, string Name);
public readonly record struct Purchase
    (Product Item, DateTime Ordered, int Quantity);
```

列表 8-18：将 Product 和 Purchase 定义为记录结构体

我们将使用 列表 8-19 中展示的 CompareSequences 方法来创建一系列 Purchase 实例并记录性能。我们回到发布版构建的性能分析，这样结果就能够考虑到 JIT（或 AOT）编译器所带来的任何优化。

```
private static Purchase MakePurchase(int id)
    => new Purchase(new Product(id, id, "Some Description"),
        DateTime.MinValue, id);
public static void CompareSequences(int count)
{
    var items = Enumerable.Range(0, count)
        **.Select(MakePurchase)**
        .ToList();
    Assert.That(items.SequenceEqual(items));
}
```

列表 8-19：创建一个随机生成对象的序列

CompareSequences 方法遵循与我们之前创建序列并调用 SequenceEqual 比较元素的模式相似。为了使性能报告更加清晰，我们使用 MakePurchase 作为 Select 表达式的一个方法组参数。这样，我们就可以直接测量它的性能，而不会通过使用 lambda 表达式引入任何开销——我们将在《常见习惯用法和实践如何影响性能》中详细讨论这个问题，见 第 279 页。表 8-15 显示了使用 MakePurchase 方法创建 1000 万个 Purchase 对象的分析器报告。

表 8-15： 创建 Purchase 序列的性能报告

| 方法 | 时间（毫秒） | 签名 |
| --- | --- | --- |
| 29.4% MakePurchase | 294 | MakePurchase(Int32) |
| 2.45% op_Implicit | 25 | System.Decimal.op_Implicit(Int32) |
| 2.03% Purchase..ctor | 20 | Purchase..ctor(Product, DateTime, Int32) |
| 1.41% Product..ctor | 14 | Product..ctor(Int32, Decimal, String) |

虽然嵌套的构造函数会增加创建Purchase对象所需的时间，但大部分时间都花费在MakePurchase的实现上，这表明初始化实例并进行复制是更昂贵的因素。特别是，创建一个新的Product并将实例复制到Purchase构造函数时，复制是我们可以通过将Product变为引用类型来避免的。

#### 引用类型性能

当我们复制一个引用变量时，对象实例本身并不会被复制，因此复制成本很低。在这里，我们将Product变成一个封闭的记录，而不是只读记录结构体：

```
public sealed record Product
    (int Id, decimal Price, string Name);
```

使用这种位置语法的记录默认是不可变的引用类型。对于Product类型，编译器为Id、Price和Name属性插入了仅限init的属性，这意味着一个实例可以被多个包含对象安全高效地引用。由于这些属性没有set访问器，因此不存在通过别名引用进行意外修改的风险。更重要的是，对于我们的测试来说，一旦Product实例被创建，仅需将其引用传递给Purchase构造函数。

对于这个测试，我们将Purchase类型保持为记录结构体，因为我们希望避免复制其嵌套的Product。然而，使用引用类型来表示Product引入了其他开销，正如我们在表 8-16 中看到的，对于创建 1000 万个Purchase对象的分析报告。

表 8-16： 创建引用类型值的性能报告

| 方法 | 时间（毫秒） | 签名 |
| --- | --- | --- |
| 77.8% MakePurchase | 1,409 | MakePurchase(Int32) |
| 34.9% [垃圾回收] | 632 |  |
| 0.33% Product..ctor | 6.0 | Product..ctor(Int32, Decimal, String) |
| 0.33% Purchase..ctor | 6.0 | Purchase..ctor(Product, DateTime, Int32) |

MakePurchase 方法比 表 8-15 中的速度慢得多，主要原因是垃圾回收。将 Product 改为记录类型而不是记录结构体，给垃圾回收器带来了很大的压力，即使它无法回收任何对象，也需要花费时间。

这里的教训是，关于短生命周期对象应使用值类型的常见建议，至少部分原因与内存压力和垃圾回收的成本有关。值类型实例因为不在堆上分配内存，因此不会产生这些成本。即使是复制巨大对象实例，也不总是最显著的开销，因此像这个例子一样，将大型值类型更改为引用类型以避免复制，可能会对程序的整体性能产生不利影响。

我们还有其他因素需要考虑。例如，如果我们预计应用程序中的许多 Purchase 对象具有相同的 Product 值，那么让所有这些 Purchase 实例共享相同的 Product 实例，可能会带来显著的好处，使得引用类型的实现更加吸引人。

#### 引用相等性的好处

我们在 列表 8-19 中使用的 MakePurchase 方法，用于创建 购买 实例时，每次都会为每个 购买 对象创建一个新的 产品 对象。在 列表 8-20 中，我们修改了 MakePurchase 方法，使得它不再每次都创建一个新的 产品，而是将少数几个共享的 产品 实例中的一个分配给每个新的 购买。由于 产品 是一个记录类型，因此它是引用类型，每个 产品 实例会被多个 购买 对象共享。

```
private static readonly List<Product> SharedProducts = new()
{
    new Product(0, 0, "Some Description"),
    new Product(1, 1, "Some Description"),
    new Product(2, 2, "Some Description"),
    new Product(3, 3, "Some Description"),
    new Product(4, 4, "Some Description"),
};
private static Purchase MakePurchase(int id)
{
    **var component** **=** **SharedProducts[id % SharedProducts.Count];**
    return new Purchase(component, DateTime.MinValue, id);
}
```

列表 8-20：在对象间共享引用

在创建任何 购买 对象之前，我们先初始化一个包含少量 产品 实例的短列表。根据用于创建 购买 对象的 id 值，从该列表中选择一个 产品 引用。由于 MakePurchase 方法不再创建新的 产品 实例，我们预期它的执行会更快，报告中的 表 8-17 已经确认了这一点。

表 8-17： 分配预分配的 产品 对象

| 方法 | 时间 (毫秒) | 签名 |
| --- | --- | --- |
| 17.1% MakePurchase | 86 | MakePurchase(Int32) |
| 2.38% Purchase..ctor | 12 | Purchase..ctor(Product, DateTime, Int32) |
| 1.18% get_Item | 5.9 | System.Collections.Generic.List`1 .get_Item(Int32) |

更重要的是，由于许多`Purchase`实例共享一个`Product`实例，比较`Purchase`实例是否相等的速度现在会更快。记录类型的`Equals`方法的实现包含了一个简单的优化，即首先进行两个引用的身份比较。当两个被比较的`Product`变量都引用内存中的同一个实例时，就不需要继续检查单独的字段，因为它们必须是相同的。表 8-18 展示了对 1000 万个`Purchase`对象进行序列元素比较的报告。

表 8-18： 共享引用的序列比较

| 方法 | 时间（毫秒） | 签名 |
| --- | --- | --- |
| 68.3% SequenceEqual | 350 | System.Linq.Enumerable .SequenceEqual(IEnumerable, IEnumerable) |
| 27.1% Equals | 139 | Purchase.Equals(Purchase) |
| 11.7% get_Default | 60 | System.Collections.Generic .EqualityComparer`1.get_Default() |
| 9.36% Equals | 48 | System.Collections.Generic .GenericEqualityComparer`1.Equals(T, T) |
| 3.52% Equals | 18 | Product.Equals(Product) |

如果我们使用记录结构（record struct）来运行相同的测试，针对 Product——也就是说，将几个预创建的 Product 实例之一分配给每个 Purchase——我们可以比较共享引用与复制每个 Product 对象的性能。表 8-19 显示了在 Product 是记录结构时，10 百万个 Purchase 对象的 SequenceEqual 的报告。

表 8-19： 比较复制实例的序列

| 方法 | 时间 (毫秒) | 签名 |
| --- | --- | --- |
| 59.5% SequenceEqual | 591 | System.Linq.Enumerable .SequenceEqual(IEnumerable, IEnumerable) |
| 13.3% Equals | 132 | Purchase.Equals(Purchase) |
| 12.7% Equals | 126 | System.Collections.Generic .GenericEqualityComparer`1.Equals(T, T) |
| 9.01% Equals | 89 | Product.Equals(Product) |
| 1.22% Equals | 12 | System.DateTime.Equals(DateTime) |
| 0.60% get_Default | 6.0 | System.Collections.Generic .EqualityComparer`1.get_Default() |

尽管Purchase.Equals方法的头时间在每种情况下几乎相同，但在表 8-19 中使用记录结构的SequenceEqual方法的执行速度明显比在表 8-18 中使用记录的要慢。记录结构无法利用记录类型的简单引用标识优化，尽管许多对Product.Equals的调用可能已经被 JIT 编译器内联。结果是，我们看到了在SequenceEqual中必须复制记录结构值并比较其字段的额外开销，而不是在Purchase.Equals中。

### 衡量编译器生成的 Equals 方法

在列表 8-18 中使用的位置记录结构语法使得<sup class="SANS_TheSansMonoCd_W5Regular_11">Purchase</sup>和<sup class="SANS_TheSansMonoCd_W5Regular_11">Product</sup>类型的定义紧凑，但这也带来了一些轻微但可测量的效率折衷。实现了IEquatable<T>接口的类型安全的Equals方法是由编译器生成的，无论它们是否使用位置语法。虽然方便，但这不一定是最有效的实现。当我们处理大量对象时，编写我们自己的Equals方法可能会更有价值，在这种情况下，编译器将不会为我们生成一个。

在第五章中，你看到编译器插入了代码以获取每个字段的默认EqualityComparer对象。例如，列表 8-21 大致展示了编译器为列表 8-18 中的<sup class="SANS_TheSansMonoCd_W5Regular_11">Purchase</sup>记录结构创建的Equals方法。

```
public bool Equals(Purchase other)
    => EqualityComparer<Product>.Default.Equals(_Item_field, other._Item_field) &&
       EqualityComparer<DateTime>.Default.Equals(_Ordered_field, other._Ordered_field) &&
       EqualityComparer<int>.Default.Equals(_Quantity_field, other._Quantity_field);
```

列表 8-21：记录结构的 Equals 方法

编译器分配的后台字段的真实名称在常规 C# 中是无效的，因此不可能与我们的任何标识符发生冲突；这里使用的名称仅用于说明这个概念。尽管直接使用后台字段而不是访问属性来执行比较，但每次调用时为每个字段获取默认的 EqualityComparer 实现可能会影响效率。表 8-20 显示了在使用 SequenceEqual 方法比较两个包含 1000 万个 Purchase 记录结构对象的列表时的分析器输出。

表 8-20： 使用记录结构实例比较序列

| 方法 | 时间 (毫秒) | 签名 |
| --- | --- | --- |
| 55.7% SequenceEqual | 558 | System.Linq.Enumerable .SequenceEqual(IEnumerable, IEnumerable) |
| 13.7% Equals | 138 | Purchase.Equals(Purchase) |
| 10.2% Equals | 102 | System.Collections.Generic .GenericEqualityComparer`1.Equals(T, T) |
| 3.58% Equals | 36 | Product.Equals(Product) |
| 1.80% Equals | 18 | System.Decimal.Equals(Decimal) |
| 0.60% Equals | 6.0 | System.Int32.Equals(Int32) |
| 0.60% get_Default | 6.0 | System.Collections.Generic .EqualityComparer`1.get_Default() |

虽然 JIT 编译器可能会内联一些或所有对 `EqualityComparer< T >.Default 属性的使用，以及对其 `Equals` 方法的调用，但并不能保证它能够做到这一点。正如我们之前在用字段替代属性访问时所做的那样，我们可以定义自己的 `Equals` 方法，直接比较值，而无需使用 `EqualityComparer< T >`。然而，我们无法访问为位置记录结构生成的属性的编译器生成的后备字段。相反，在 Listing 8-22 中，我们使用了一个简单的记录结构体 `Purchase`，在其中定义了我们自己的私有字段和构造函数来初始化它们。

```
public readonly record struct Purchase
{
    public Purchase(Product item, DateTime ordered, int quantity)
        => (this.item, this.ordered, this.quantity) =
              (item, ordered, quantity);
    `--snip--`
    public bool Equals(Purchase other)
        => item.Equals(other.item) &&
           ordered.Equals(other.ordered) && quantity == other.quantity;
    private readonly Product item;
    private readonly DateTime ordered;
    private readonly int quantity;
}
```

Listing 8-22: 为 Purchase 结构体构造私有字段

我们还添加了自己的 `Equals` 实现，直接比较我们定义的字段。这个自定义的 `Equals` 替代了编译器在我们没有定义自己的情况下会引入的实现。我们还需要添加属性来暴露字段的值，尽管这些内容，以及以类似方式变化的 `Product` 类型在此处没有显示。重新运行代码以比较两个包含 1000 万个 `Purchase` 项目的序列时，会生成在 表 8-21 中显示的报告。

表 8-21： 使用自定义的 Equals

| 方法 | 时间（毫秒） | 签名 |
| --- | --- | --- |
| 100% SequenceEqual | 440 | System.Linq.Enumerable .SequenceEqual(IEnumerable, IEnumerable) |
| 12.3% Equals | 54 | Purchase.Equals(Purchase) |
| 8.18% Equals | 36 | Product.Equals(Product) |
| 1.36% 等于 | 6.0 | System.DateTime.Equals(DateTime) |

通过提供我们自己的Equals方法，我们将SequenceEqual的性能提高了大约 20%，与表 8-20 中的结果相比，部分原因是我们的实现可能为 JIT 编译器提供了更有效的内联代码机会。比较更大的序列会产生类似的结果，因此，如果我们特别关注性能并且频繁比较许多项目，这种优化可能会带来好处。

我们在这里看到的性能提升，主要是因为Purchase是一个相对复杂的类型。一个更简单的位置记录结构——例如，只有一个int字段——可能不会像在Purchase和Product中进行的优化那样带来好处。位置记录语法的主要优点是它的简洁性，使任何读者都能清楚地了解类型所代表的内容。我们为了一点点原始性能提升而牺牲了这种简洁性，而这一提升只有在使用性能分析工具时才可见。这个例子突显了在尝试通过猜测编译器来手动优化代码之前，先进行性能测量的重要性。

## 常见的习惯用法和实践如何影响性能

在 C#中，一些常见的做法由于性能问题受到过度批评。我们通常会认为源代码中更高层次的抽象会导致性能上的开销，这在某种程度上是正确的：C#是一种*高级*编程语言，我们的程序最终会经过多个步骤转换为本地机器代码。我们可以手动编写机器代码来执行相同的任务，但 C#代码更具可移植性、更容易维护、出错的可能性较低，并且比机器代码更容易阅读和编写。这些优势通常远大于性能上的任何损失。

然而，高级代码并不总是会导致性能上的惩罚。在本节中，我们将探讨循环和模式匹配这两个常见的 C#特性，它们使我们能够简洁地表达复杂的 C#思想，同时提供与底层代码相当甚至更优的性能。

### 循环和迭代

在本章中，我们已多次使用 LINQ 的*流畅语法*形式来创建对象序列。LINQ 已经成为 C# 的一部分多年，对于大多数对该语言及其习惯用法有一定了解的程序员来说，应该是熟悉的。清单 8-23 展示了使用流畅语法创建一个<sup class="SANS_TheSansMonoCd_W5Regular_11">Purchase</sup>对象列表的例子。

```
private static Purchase MakePurchase(int id)
    => new Purchase(new Product(id, id, "Some Description"),
        DateTime.MinValue, id);
var items = Enumerable.Range(0, count)
    .Select(i => MakePurchase(i))
    .ToList();
```

清单 8-23：LINQ 流畅语法

LINQ 还有一种替代的*查询语法*，一些 C# 程序员觉得它更加容易使用。清单 8-24 展示了使用查询语法创建清单 8-23 中的<sup class="SANS_TheSansMonoCd_W5Regular_11">items</sup>序列的等效方式。

```
var query = from i in Enumerable.Range(0, count)
            select MakePurchase(i);
var items = query.ToList();
```

清单 8-24：LINQ 查询语法

编译器为清单 8-23 和 8-24 生成相同的 CIL，因此选择哪种方式主要取决于我们认为哪种方式更易读。有一种优化是可能的，但仅适用于流畅语法版本：避免将 lambda 作为参数传递给<sup class="SANS_TheSansMonoCd_W5Regular_11">Select</sup>方法。该 lambda 需要捕获<sup class="SANS_TheSansMonoCd_W5Regular_11">i</sup>变量，因此编译器会生成一个闭包对象，这会导致调用<sup class="SANS_TheSansMonoCd_W5Regular_11">MakePurchase</sup>方法时需要额外的间接层。为了避免闭包，我们可以改为将<sup class="SANS_TheSansMonoCd_W5Regular_11">MakePurchase</sup>作为方法组参数传递，如清单 8-25 所示。

```
var items = Enumerable.Range(0, count)
    .Select(MakePurchase)
    .ToList();
```

清单 8-25：通过使用方法组优化 LINQ

为了比较每种方法的效率，首先我们分析了来自清单 8-23 的版本，该版本使用了一个 lambda。表格 8-22 展示了创建 1000 万项列表的性能报告。

表 8-22： 使用 LINQ 和 Lambda 创建序列的性能

| 方法 | 时间 (毫秒) | 签名 |
| --- | --- | --- |
| 98.1% ToList | 415 | System.Linq.Enumerable .ToList(IEnumerable) |
| 36.0% MakePurchase | 152 | MakePurchase(Int32) |
| 31.3% < Closure>b__3_0 | 132 | <>c.< Closure>b__3_0(Int32) |
| 31.3% MakePurchase | 132 | MakePurchase(Int32) |

标识符名称 <> c 是编译器生成的闭包对象，用于捕获 i 变量，这是编译器引入的一个示例，展示了在我们自己的代码中非法的名称。闭包有一个实例方法 < Closure>b__3_0，该方法又调用了我们的 MakePurchase 方法。MakePurchase 方法在本报告中出现了两次——一次是在闭包方法内部，一次是在闭包方法外部——这是由于 JIT 编译器内联了一些对 < Closure>b__3_0 方法的调用，并直接调用了 MakePurchase。

表 8-23 中的报告显示了使用方法组方式创建 1000 万个项目时的性能。

表 8-23： 使用方法组通过 LINQ 创建序列的性能

| 方法 | 时间 (毫秒) | 签名 |
| --- | --- | --- |
| 100% ToList | 430 | System.Linq.Enumerable.ToList(IEnumerable) |
| 71.9% MakePurchase | 309 | MakePurchase(Int32) |

有些出乎意料的是，带闭包对象的版本略微比方法组版本更快。我们不应过度解读这一点，因为这个差异在比较多次运行时的误差范围内。然而，这确实告诉我们，无论绝对差异如何，使用 lambda 表达式不会带来显著的性能损失。

表示 lambda 的闭包对象只会为整个表达式创建一次，而不是为每个通过Select方法生成的元素创建一次。尽管闭包对象为每次调用MakePurchase提供了一个额外的间接层，但 JIT 编译器会内联许多对闭包< Closure>b__3_0方法的调用，直接调用MakePurchase，或者也内联其内容。

我们可以通过其他几种方式创建类似的序列。让我们探讨两种常见的方法，看看它们的性能与使用 LINQ 相比如何。

#### 迭代器方法

*迭代器*是 C#的一个基础部分，并且支撑着其他更高层次的功能，包括 LINQ。实际上，LINQ 已经在现代 C#中变得如此普遍，以至于很容易忘记它是基于两个系统接口的：IEnumerable< T >接口，它是类型为T的元素序列的抽象视图，以及IEnumerator< T >接口，它表示一个可以逐个获取IEnumerable< T >元素的迭代器。基本原理是，IEnumerable< T >接口有一个名为GetEnumerator的方法，它返回IEnumerator< T >的实现。

尽管这两个接口在现代代码中大多被隐藏，但IEnumerable< T >仍然是代表序列的类型协议的重要组成部分，并且是扩展方法的“家”，如Select和Where，这些方法构成了 LINQ 系统的大部分。

IEnumerator< T > 接口也是 foreach 循环的基础，这是枚举实现了 IEnumerable< T > 序列元素的一种方式。在 Listing 8-26 中，我们编写了一个简单的 ToList 方法，允许我们记录其性能，并与 LINQ 版本进行对比。我们的 ToList 使用 foreach 填充一个 Purchase 对象的列表，因此它依赖于 Enumerable.Range 方法提供的迭代器。

```
public static List<Purchase> ToList(int count)
{
    var items = new List<Purchase>();
    foreach(var i in Enumerable.Range(0, count))
    {
        items.Add(MakePurchase(i));
    }
    return items;
}
```

Listing 8-26: 使用 foreach 填充列表

比较我们的 ToList 方法和 LINQ 版本的 Listing 8-23，首先需要注意的是，在循环之前我们需要声明目标的 Purchase 对象列表。foreach 循环从 Enumerable.Range 获取一个 IEnumerator< int>，并且 foreach 块的主体会针对迭代器中的每个元素运行。当我们查看 Table 8-24 中 ToList 方法的分析报告时，我们可以看到基本的机制。

Table 8-24: 分析迭代器方法

| 方法 | 时间 (ms) | 签名 |
| --- | --- | --- |
| 100% ToList | 638 | ToList(Int32) |
| 41.2% AddWithResize | 263 | System.Collections.Generic.List`1 .AddWithResize(T) |
| 38.1% MakePurchase | 243 | MakePurchase(Int32) |
| 0.95% MoveNext | 6.1 | System.Linq.Enumerable+RangeIterator .MoveNext() |
| 0.95% get_Current | 6.0 | System.Linq.Enumerable+Iterator`1 .get_Current() |

这个性能报告展示了foreach结构的工作原理；get_Current和MoveNext方法属于IEnumerator< T >接口，正如它们的名字所示，它们允许我们获取当前元素并将迭代器移动到序列中的下一个项。

这个报告还显示，我们手动实现的ToList比在表 8-22 中报告的 LINQ 版本要慢得多，但我们并没有充分利用List< Purchase>的功能。因为我们提前知道需要的项数，我们可以避免大部分AddWithResize方法的开销，并在构造函数调用中像这样指定列表的容量：

```
var items = new List<Purchase>(count);
```

通过显式请求容量，我们在添加新元素之前为count个项分配足够的内存，这样列表在空间不足时就无需重新调整大小。如果我们重新运行性能测试，如表 8-25 所示，它与之前的测试结果更加一致。

表 8-25: 预分配列表的容量

| 方法 | 时间 (毫秒) | 签名 |
| --- | --- | --- |
| 100% ToList | 426 | ToList(Int32) |
| 63.0% MakePurchase | 268 | MakePurchase(Int32) |
| 4.28% MoveNext | 18 | System.Linq.Enumerable+RangeIterator .MoveNext() |
| 1.41% get_Current | 6.0 | System.Linq.Enumerable+Iterator`1 .get_Current() |

我们的测试表明，使用 LINQ，至少在生成元素序列这一相对简单的任务中，其效率至少与使用 foreach 循环一样。然而，我们可以尝试另一种方法：for 循环。

#### 循环方法

我们创建 Purchase 对象的方式是基于创建一系列 int 值，并使用 Select 方法将其转换为一个新的 Purchase 对象序列。清单 8-27 展示了如何使用基本的 for 循环实现相同的结果，该循环不依赖于迭代器，而仅仅是根据循环条件指定的次数执行循环体。

```
public static List<Purchase> ToList(int count)
{
    var items = new List<Purchase>(count);
    for(int i = 0; i != count; ++i)
    {
        items.Add(MakePurchase(i));
    }
    return items;
}
```

清单 8-27：使用简单的 for 循环

就像我们之前使用 foreach 循环时一样，我们必须在进入循环之前创建目标 List< Purchase>，并使用构造函数设置其容量。在循环体内，我们使用 MakePurchase 方法来添加新的 Purchase，就像我们之前那样。表 8-26 展示了使用 for 循环创建 1000 万个 Purchase 对象的性能报告。

表 8-26： 直接 for 循环性能

| 方法 | 时间 (毫秒) | 签名 |
| --- | --- | --- |
| 100% ToList | 417 | ToList(Int32) |
| 67.3% MakePurchase | 281 | MakePurchase(Int32) |
| 5.70% op_Implicit | 24 | System.Decimal.op_Implicit(Int32) |

再次强调，for循环方法和我们尝试过的其他方法在性能上没有显著差异。使用 LINQ 与使用foreach或for循环的主要区别在于风格：LINQ 代码更直接，使我们能够声明性地表达意图，而for和foreach循环则更加过程化。LINQ 表达式让我们关注所需的结果，而循环方式则更多关注执行的步骤或指令。

### 模式匹配与选择

声明式编程风格相比过程式编程风格的一个常见好处是，我们能够通过更少的代码实现相同的结果。虽然这节省了我们输入的量，但这只是一个副作用。真正的好处在于对于人类读者来说，语法更简洁易懂。用 LINQ 风格的函数表达式替代显式的循环就是一个例子。虽然许多 LINQ 表达式在内部基于循环，但循环结构本身对于用户代码是隐藏的。手动用循环和显式条件遍历序列容易出错，而且复杂的循环结构通常比像Select或ToList这样的函数调用更难被人类读者理解。

声明式技术的另一个常见应用是选择代码：用模式匹配表达式替代if和switch语句。

考虑一下示例 8-28 中的构造函数，它通过将参数值与某些模式指定的规则进行匹配来验证参数值。

```
private const double ZeroKelvin = -273.15;
private Temperature(double celsius)
    => amount = celsius switch
    {
        double.NaN
            => throw new ArgumentException(`--snip--`),
        < ZeroKelvin or double.PositiveInfinity
            => throw new ArgumentOutOfRangeException(`--snip--`),
        _ => celsius
    };
```

示例 8-28：用于验证的模式匹配

如果传入的参数是 double.NaN，则 Temperature 构造函数会抛出异常，并且禁止小于 ZeroKelvin 或等于 PositiveInfinity 的值。对于不符合这些规则的 celsius 参数值，会通过丢弃模式将其分配给 amount 字段，这个模式是 switch 表达式中的最后一个模式。

比较 Listing 8-28 和 Listing 8-29，后者实现了完全相同的结果，但使用了 if…else 语句来测试传入的参数值。

```
private Temperature(double celsius)
{
    if(celsius is double.NaN)
    {
        throw new ArgumentException(`--snip--`);
    }
    else if(celsius < ZeroKelvin || celsius is double.PositiveInfinity)
    {
        throw new ArgumentOutOfRangeException(`--snip--`);
    }
    else
    {
        this.amount = celsius;
    }
}
```

Listing 8-29: 链接 if 和 else 进行验证

我们可以通过删除冗余的 else 语句，允许 if 块在值不符合 if 条件时继续执行，从而使这段代码不那么繁琐。虽然这样会缩短代码，但如果添加新条件时，它会变得更容易出错。

另一种选择是使用 switch 语句，如 Listing 8-30 所示。

```
switch (celsius)
{
    case double.NaN:
         throw new ArgumentException(`--snip--`);
    case < ZeroKelvin:
    case double.PositiveInfinity:
         throw new ArgumentOutOfRangeException(`--snip--`);
    default:
         this.amount = celsius;
         break;
}
```

Listing 8-30: 使用 switch 语句进行验证

这个版本更接近于 Listing 8-28 中的 switch 表达式，且这两种形式的 switch 容易混淆。主要的区别在于，在这里我们将 amount 字段作为 default 分支的一部分进行赋值，而在 switch 表达式中，amount 字段的值是整个表达式的结果。

与本章其他部分不同的是，我们无需运行性能分析来比较 Listings 8-28 和 8-30，因为编译器为每种情况生成的代码几乎完全相同——大致上与 Listing 8-29 中显示的代码相同。编译器可能会改变 CIL 中条件的顺序，但这不会改变逻辑。

## 总结

> *我们认为，单纯因为资源便宜而过度消耗资源并不是一种好的工程实践。*
> 
> —尼克劳斯·维尔特，《*奥伯龙计划：操作系统、编译器和计算机的设计*》

手动优化的代码通常更难为人类读者理解，通常是因为它常常涉及将简单的习惯用法，如循环和模式匹配，替换为更低级的构造。当一个程序运行速度比我们预期的要慢时，我们很容易冲动地直接修改我们怀疑是瓶颈的代码部分。然而，程序员的优化直觉通常并不可靠。我们可能会让代码变得更加难以阅读，却未能以任何有意义的方式提高性能。

手动优化代码几乎总是将清晰性和简洁性换取性能的过程。我们只能通过在变更前后测量性能，来判断这种交换是否合理。即便我们在某段代码中提高了性能，我们仍然需要判断是否使代码变得不那么清晰，若是如此，这个改动是否合理。我们还必须确信我们的优化没有以任何方式改变程序的行为。慢而正确的代码总是优于错误的代码，无论其性能如何。这并不是说“足够好”就不能是正确的——通常在性能与准确性或精度之间需要做出妥协——但我们必须知道不准确究竟意味着什么程度的错误。

使用众所周知的习惯用法和模式有助于人类读者轻松理解代码。因此，当我们偏离这些常见设计时，我们的代码也会变得更难理解。因此，我们必须在优化代码时有选择地针对那些能够带来最大效益的区域。

重写 Equals 的行为对大多数类型来说并不困难，但它增加了一个实现细节，这对任何需要*理解*我们代码的人来说，都是额外的认知负担。

使用记录类型来表示值类型可以去除许多附加的复杂性，因为编译器会为我们生成正确的实现。然而，即便接受这种默认行为，也不一定会产生最有效的代码。

精心的代码优化，结合分析器提供的证据，可以在速度*和*内存使用方面带来更好的性能。现代计算机速度很快，通常内存也足够，但这并不意味着我们可以浪费任何一种资源。
