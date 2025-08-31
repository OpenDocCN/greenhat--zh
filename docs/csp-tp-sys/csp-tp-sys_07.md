# 第七章：7 值类型与多态性

![](img/opener-img.png)

作为一种面向对象编程（OOP）语言，C# 对一些能够捕捉复杂思想并直观表达的特性提供了很好的支持，如类、虚方法和继承。然而，语言对继承的支持并未扩展到值类型。结构体和记录结构体隐式地继承自ValueType类，而该类又直接继承自object。

但它们不能继承任何其他类型，也不能被继承；也就是说，结构体和记录结构体隐式地是封闭的。继承是面向对象编程的核心特性之一，它使我们能够将对派生类的引用当作对基类的引用来使用，必要时通过覆盖基类的属性和方法来获得新的行为。这些特性不适用于值类型，但这并不意味着值类型就逊色。

常常将*多态性*与*继承*交替使用，但多态性是一个更为广泛的概念；它涉及编写适用于多种类型的统一代码，以减少重复。正如本章所讨论的，继承只是多态性的一种形式，尽管有技术性和语义上的充分理由表明值类型不能使用继承关系，但它们可以利用其他类型的多态性。

我们将探索以下内容：

+   为什么值类型是封闭的，以及为什么一般来说类似值的类型不应使用继承

+   子类型化与子类化的区别及其重要性

+   *类型可替代性*是什么意思，它与继承有何关系

+   在模型化对象之间的关系时，何时使用其他类型的多态性，而非继承

## 为何值类型是封闭的

禁止结构体及扩展到记录结构体使用继承的主要技术原因是，它们的生命周期和存储特性与引用类型不同。这一限制并非随意规定，而是直接源于值类型变量在内存中的表现，以及与引用类型的区别。

类之间的继承允许我们使用对基类类型的引用来引用派生类的实例，因此，引用变量的静态编译时类型不一定与动态运行时实例的类型相同。这一特性使得*虚拟调度*成为可能——即根据对象在运行时的实际类型调用适当的方法实现——并依赖于引用提供的额外间接层级；因此，继承仅适用于引用类型。

值类型变量直接包含它们的数据，因此我们不能声明一个变量为某种类型来表示另一个类型的实例，除了通过装箱（boxing）方式。继承自结构体因此没有意义，编译器会禁止这种做法。

但请记住，我们可以使用类来模拟值类型行为。正如你在第六章中看到的，string 行为像值类型，但它是作为引用类型实现的。string 类型使用基于值的（而非基于标识的）相等性比较，是不可变的，并具有其他一些标识其为值类型的特征。作为一个类，string 可以支持虚方法调度，但我们不能从 string 类派生，因为它被显式标记为密封类。这意味着我们不能像继承 DateTime、Guid 或任何其他值类型一样，创建我们自己的增强版 string 子类。

像 string 类一样，记录是引用类型，但具有类似值类型的相等性行为。记录可以从其他记录派生，并且可以拥有虚方法，因此它们似乎将值类型和继承的概念统一了。然而，使用记录并不是那么简单。我们需要注意细节，避免在使用任何继承时（无论是在类之间还是记录之间）掉进陷阱。

当我们允许我们的类型参与继承关系时，我们需要注意从这些类型派生可能带来的后果。*实现*继承和*接口*继承是有所区别的。继承实现会带来一些与继承值类型相似的困难。为了探讨为什么这样做不明智，我们来看一个具有值类型特征的类，并使用实现继承来展示可能导致的一些问题。

### 实现继承

每当我们从一个具体类继承——即一个不是完全抽象的类——我们本质上是在继承它的实现。列表 7-1 展示了一个简单的继承关系：一个 TranslucentColor 类从 Color 基类继承，并添加了自己的新特性。

```
public class Color
{
    public Color(int red, int green, int blue)
        => (Red, Green, Blue) = (red, green, blue);
    public int Red   {get;}
    public int Green {get;}
    public int Blue  {get;}
}
public class TranslucentColor : Color
{
    public TranslucentColor(int red, int green, int blue, int alpha)
        : base(red, green, blue) => Alpha = alpha;
    **public int Alpha {get;}**
}
```

列表 7-1：创建一个派生类 TranslucentColor，它继承了 Color 的实现

这个TranslucentColor类是Color类的子类，并继承了Color的所有结构表示，以及它的方法和属性。两个类都有自动属性，每个属性都有一个与属性类型相同类型的后台字段—在这个例子中是int—并且Color的每个字段都被TranslucentColor继承。

即使我们在Color中使用了私有字段，并通过属性返回它们的值，这些字段仍然会被TranslucentColor类继承，尽管它们仍然只能通过继承的*public*属性访问。

TranslucentColor从Color继承的实现依赖于那些私有字段。TranslucentColor的实例需要拥有其基类声明的所有字段的副本，以确保从Color继承的属性能够正确工作。我们可以通过TranslucentColor变量像访问TranslucentColor的成员一样，使用Color的属性，如下所示：

```
var foreground = new TranslucentColor(red: 0xFF, green: 0, blue: 0, alpha: 0x77);
Assert.That(foreground.Red, Is.EqualTo(0xFF));
Assert.That(foreground.Alpha, Is.EqualTo(0x77));
```

在这个简单的测试中，我们使用一个TranslucentColor变量的Red属性，这个属性是从Color继承而来的。我们还可以使用Alpha属性，它是作为TranslucentColor的成员声明的。

通过继承这种方式—在TranslucentColor中重用Color的实现—非常有吸引力，因为这意味着TranslucentColor类型定义不会重复Color的属性。通过从Color派生，TranslucentColor类可以免费获得这些属性。

Color 和 TranslucentColor 看起来是很好的值类型候选，因为相等比较应当比较每个实例的状态。然而，在继承层次结构中实现基于值的相等比较会隐藏一些复杂性，这可能会导致不希望出现的行为。为了证明这一点，让我们按照第五章的建议，为 Color 和 TranslucentColor 提供值语义，重写这两个类的 Equals 方法及其相关方法。

### 基于值的类相等性

我们从基类 Color 开始。在清单 7-2 中的实现下，我们可以比较两个 Color 实例，查看它们的属性是否相等。

```
public class Color : IEquatable<Color>
{
    public int Red   {get;}
    public int Green {get;}
    public int Blue  {get;}
    public bool Equals(Color? other)
        => (object?)this == (object?)other ||
           other is not null &&
           GetType() == other.GetType() &&
           Red == other.Red && Green == other.Green && Blue == other.Blue;
    public override bool Equals(object? obj)
        => Equals(obj as Color);
    public override int GetHashCode()
        => HashCode.Combine(Red, Green, Blue);
    public static bool operator==(Color? left, Color? right)
        => left?.Equals(right) ?? right is null;
 public static bool operator!=(Color? left, Color? right)
        => !left?.Equals(right) ?? right is not null;
}
```

清单 7-2：在基类 Color 中定义值相等性

这种相等性实现遵循了类的值类型相等性实现的常见做法，包括微软文档中的相关指南。Color 类实现了 IEquatable< Color> 接口，这要求为 Color 专门重载 Equals 方法。我们使用这个重载提供完整的实现，可以从任何其他方法中调用，包括从 object 基类重写的 Equals 方法。由于我们已经重写了 Equals(object?)，我们还重写了 GetHashCode，以确保两个相等的 Color 实例生成相同的哈希值。最后，我们为 == 和 != 相等操作符提供了实现。

让我们详细检查每个步骤。

#### Equals 的规范形式

首先，我们必须重写从 object 继承的虚方法 Equals，如 Listing 7-3 中所示。由于 Color 是一个类，默认情况下 Equals 比较的是对象的身份，因此我们需要重写该行为，以便为 Color 提供基于值的实现。

```
public override bool Equals(object? obj)
    => Equals(obj as Color);
```

Listing 7-3: 重写 Equals

Equals 的重写必须与基类的签名匹配。在这个例子中，我们在一个可空的上下文中声明类型，因此我们将 object? 作为 Equals 的参数类型，表示我们知道参数可能是 null，并且能够安全地处理这种情况。这里我们使用 as 操作符将 obj 强制转换为 Color，以便调用类型安全的 Equals 方法。如果 obj 不是一个 Color 或者是 null，传入的参数将是 null，这将通过类型安全的重载在 Listing 7-4 中显式处理。

```
public bool Equals(Color? other)
    => (object?)this == (object?)other ||
       other is not null &&
       GetType() == other.GetType() &&
       Red == other.Red && Green == other.Green && Blue == other.Blue;
```

Listing 7-4: 实现 IEquatable< Color>

类型安全的 Equals 实现是 IEquatable< Color> 接口的一部分，接受一个可空的 Color 参数。当我们比较两个静态类型为 Color 的变量时，包含从 operator== 或 operator!= 方法调用的情况，这个重载方法将优先于接受 object? 参数的方法。

Color 作为引用类型的一个含义是，other 参数可能引用与 this 相同的实例。为了处理这种情况，清单 7-4 将 this 和 other 都强制转换为 object，以明确表示我们打算进行引用比较。尽管在 清单 7-3 中使用的从 object 到更派生类型的强制转换是一个相对昂贵的运行时转换，但从 Color 到其 object 基类的转换非常高效，并且使得比较能够使用 第五章 中引入的原生 ceq 指令进行。一种替代方法是这里使用 ReferenceEquals(this, other)，使引用比较更加显式。

比较这两个变量是否引用同一个对象是一个简单但非强制性的优化。如果其左侧表达式为真，逻辑 || 运算符会短路，因此只有当 this 和 other 引用不同的实例时，才会尝试其余的比较。请注意，代码中比较的顺序依赖于运算符优先级；逻辑 AND 运算符 (&&) 的优先级高于逻辑 OR 运算符 (||)，因此 || 右侧的比较会像被显式括在一对圆括号中一样一起绑定。虽然是冗余的，但额外的圆括号不会影响行为，某些程序员更喜欢添加这些括号，以避免记住运算符优先级规则。

由于 Color 是引用类型，传递的参数可能是 null，因此我们使用 is not 常量模式来将 other 与 null 进行比较，从而避免常见的陷阱——递归调用我们自己的 Equals 方法。

Color类故意没有封闭，因此我们还检查other值是否与this的类型完全相同，方法是使用定义在object基类中的GetType方法。此方法返回实例的运行时类型，如果other是指向更派生类型（如TranslucentColor）的引用，则类型不匹配。不同类型的对象通常不会比较相等，即使它们的类型通过继承相关联。

最后，如果类型匹配，我们依次比较每个属性的值。如果它们都匹配，我们的Equals方法返回true。我们在这里使用==而不是Equals，因为Color的所有属性都是简单的int值。像这样的内置值可以本质上进行比较，比对每个属性调用Equals方法更加紧凑。

为了使比较Color实例变得自然，我们还实现了operator==和operator!=，它们都委托给类型安全的Equals方法，像这样：

```
public static bool operator==(Color? left, Color? right)
    => left?.Equals(right) ?? right is null;
public static bool operator!=(Color? left, Color? right)
    => !left?.Equals(right) ?? right is not null;
```

如果left参数不是null，则==运算符将返回Equals的结果；否则，如果两个参数都为null，则返回true。!=运算符通过反转比较结果返回与==相反的结果。

#### 平等契约

以自洽的方式实现相等性非常关键。如果我们有两个引用指向同一实例的 Color，但它们*不*相等，那将是非常奇怪的，更奇怪的是，如果在比较一个值与它自身时，Equals 返回 false。相等性有一个类似于你在第六章中看到的“小于比较”约定的契约。即，相等性具有以下特征：

**自反性**

x == x 总是 true。

**对称性**

如果 x == y，则 y == x。

**传递性**

如果 x == y *并且* y == z，那么可以推断出 x == z。

**安全性**

非<sup class="SANS_TheSansMonoCd_W5Regular_11">null</sup>值永远不等于 null。

**稳定性**

只要 x 和 y 不变，x == y 的结果不会改变。

在示例 7-5 中，我们编写了一些测试来证明我们已满足相等性契约的要求。

> 注意

*这些测试是为了强调这一点，而不是为了展示一种好的断言写作风格。*

第一个测试还确保我们是通过值来比较变量，而不仅仅是比较引用。

```
var pencil = new Color(0xFF, 0, 0);
var crayon = new Color(0xFF, 0, 0);
var brush =  new Color(0xFF, 0, 0);
// Reflexive, value-based equality
Assert.That(pencil == pencil, Is.True);
Assert.That(pencil == new Color(0xFF, 0, 0), Is.True);
// Symmetric
Assert.That(pencil == crayon, Is.True);
Assert.That(crayon == pencil, Is.True);
// Transitive
Assert.That(pencil == crayon, Is.True); Assert.That(crayon == brush, Is.True);
Assert.That(pencil == brush, Is.True);
// Safe with null
Assert.That(pencil != null, Is.True);
Assert.That(null != pencil, Is.True);
```

示例 7-5：测试 Color 的相等性契约

编写测试以验证比较的稳定性更为复杂，因此在示例 7-6 中，我们测试的是相反的情况：即如果某个值发生变化，实例就不再相等。

```
var pencil = new Color(0xFF, 0, 0);
var crayon = new Color(0xFF, 0, 0);
Assert.That(pencil == crayon, Is.True);
pencil = new Color(0, 0xFF, 0);
Assert.That(pencil != crayon, Is.True);
```

清单 7-6：测试相等性是否稳定

由于Color的属性是不可变的，我们只能通过将pencil赋值为一个新实例来改变它的值。然而，效果和我们修改一个或多个属性是一样的，因为我们已经安排了Color实例按值进行比较。

我们对Equals方法及其操作符对的另一项要求是：它们绝不应抛出异常。我们的实现没有这种风险，因为我们已经测试过它在处理null时是安全的。

### 派生类中的相等行为

下一步是为派生类TranslucentColor实现相等性，正如我们所知，它继承了所有来自Color的函数和属性。由于TranslucentColor是一个类值类型，它应该为自己实现IEquatable< T >接口，将T替换为TranslucentColor。正如清单 7-7 所示，实现IEquatable< TranslucentColor>比Color基类要简单一些，因为基类已经完成了大部分工作。

```
public class TranslucentColor : Color, IEquatable<TranslucentColor>
{
    public int Alpha {get;}
    public bool Equals(TranslucentColor? other) ❶
        => base.Equals(other) && Alpha == other.Alpha;
    public override bool Equals(object? obj)
        => Equals(obj as TranslucentColor);
 public override int GetHashCode()
        => HashCode.Combine(Alpha, base.GetHashCode());
    public static bool operator==(TranslucentColor? left, TranslucentColor? right) ❷
        => left?.Equals(right) ?? right is null;
    public static bool operator!=(TranslucentColor? left, TranslucentColor? right)
        => !left?.Equals(right) ?? right is not null;
}
```

清单 7-7：派生类 TranslucentColor 中的行为继承

和Color的实现一样，TranslucentColor重写了Equals(object?)方法，将object参数转换为TranslucentColor，以调用Equals(TranslucentColor?)方法❶。该方法还检查我们是否正在比较对单个实例的两个引用，并确保other参数不是null。

由于Color已经执行了对相同引用的检查、与null的比较、类型检查，以及对Red、Green和Blue属性的比较，因此我们不需要重复这些比较，只需调用基类的Equals方法，然后最终比较特定于TranslucentColor的Alpha属性。将other传递给base.Equals是可以的，因为TranslucentColor引用会隐式转换为其Color基类类型。

我们还为TranslucentColor提供了它自己的operator==和operator!=实现，它们也遵循与Color相同的模式，唯一不同的是它们接受两个TranslucentColor参数❷。

相等契约不仅适用于Color，还适用于TranslucentColor。我们可以使用类似于清单 7-5 的测试来确保TranslucentColor符合契约的要求。清单 7-8 展示了对清单 7-6 中Color稳定性检查的变体，测试了TranslucentColor类的Alpha属性值的差异会导致TranslucentColor实例比较为不相等。

```
var pencil = new TranslucentColor(0xFF, 0, 0xFF, 0x77);
var crayon = new TranslucentColor(0xFF, 0, 0xFF, 0x77);
Assert.That(pencil == crayon, Is.True);
pencil = new TranslucentColor(0xFF, 0, 0xFF, 0);
**Assert.That(pencil !=** **crayon, Is.True);**
```

清单 7-8：测试 TranslucentColor 的相等契约

在这个例子中，两个TranslucentColor实例仅在其Alpha属性上有所不同，并且正确地比较为*不相等*。那么我们可能会得出结论，一切都没有问题——但我们会错的。

### 相等比较与类型替代

我们使用了一套测试来确保当我们使用动态（运行时）实例类型与静态（编译时）变量类型相同的变量时，`Color` 和 `TranslucentColor` 的相等性契约是完好的。然而，类型可能并不总是匹配。编译器允许我们将一个 `TranslucentColor` 的引用传递到任何需要 `Color` 引用的地方，因为 `Color` 是 `TranslucentColor` 的直接基类。换句话说，`Color` 类型可以被*替代*为 `TranslucentColor`。在运行时，任何 `Color` 引用实际上可能指向一个 `TranslucentColor` 实例。

为了说明使用基类引用到派生类实例对相等性的影响，列表 7-9 明确使用 `Color` 基类引用来声明两个 `TranslucentColor` 值，它们不相等，因为它们的 `Alpha` 属性不同。

```
**Color** pencil = new TranslucentColor(0xFF, 0, 0xFF, **0x77**);
**Color** crayon = new TranslucentColor(0xFF, 0, 0xFF, **0**);
Assert.That(pencil == crayon, Is.False);
```

列表 7-9：从基类测试相等性

这个测试失败了：`pencil` 和 `crayon` 变量在比较时是相等的，即使它们的实例值不同。无论我们使用 `==` 比较这些变量，还是调用 `Equals` 方法，结果都是一样的。

正在比较的静态类型是 Color 变量，因此这里调用的是基类实现的 operator==，它又调用了 Equals 方法。Color 中的 Equals 方法并不了解 TranslucentColor 的 Alpha 属性，因此 Equals 仅通过 Red、Green 和 Blue 属性来判断相等性。这些属性完全相同，因此根据 Color.Equals，这两个对象是相等的。

这些机制解释了为什么我们的 pencil 和 crayon 变量错误地被认为相等，但类型替换并不总是那么容易被发现，它的后果深远。

#### 类型替换的影响

我们很少会显式地使用 Color 引用来表示 TranslucentColor 对象，但我们可以将一个 TranslucentColor 引用作为参数传递给一个接受 Color 参数的方法。当期望传递 Color 时，我们可以用 TranslucentColor 进行替代。

列表 7-10 显示了如果我们将两个仅在其 Alpha 值不同的 TranslucentColor 对象作为引用传递给一个具有 Color 参数的方法，那么这些参数变量在方法内会被认为相等。如果我们将相同的两个引用传递给一个接受 TranslucentColor 参数的方法，即使它们没有发生变化，值也不会被认为相等。

```
**bool EqualViaBase(Color left, Color right)**
    => left.Equals(right);
bool EqualViaDerived(TranslucentColor left, TranslucentColor right)
    => left.Equals(right);
 var pencil = new TranslucentColor(0xFF, 0, 0xFF, 0x77);
var crayon = new TranslucentColor(0xFF, 0, 0xFF, 0);
**Assert.That(EqualViaBase(pencil, crayon), Is.True);**
Assert.That(EqualViaDerived(pencil, crayon), Is.False);
```

列表 7-10：测试稳定性承诺

在第一次断言中，当我们调用EqualViaBase方法时，pencil和crayon引用会被自动转换为Color引用，因为对派生类的引用可以隐式地转换为对其任何基类的引用。在EqualViaBase中的Equals调用会调用Color的实现，而该实现错误地认为参数变量是相等的。EqualViaDerived方法直接调用TranslucentColor.Equals，它正确地报告参数变量不相等。

TranslucentColor的相等实现并不稳定：它可能会根据用于引用这两个实例的变量的静态类型，产生不同的结果，即使它们的基础状态保持不变。

#### 违反契约

清单 7-10 中的测试行为表明，TranslucentColor违反了相等契约所规定的稳定性承诺——即，如果被比较的值没有变化，则< s>Equals</s>的结果不应发生变化。当我们比较两个具有不同值的变量时，它们应该被比较为不相等，并且只要这两个变量的状态没有变化，比较结果就不应改变。

我们的Equals实现的一个问题是，Color类中的类型特定重载Equals并不是虚拟的，因此不能在TranslucentColor中重写。虚拟版本的Equals（接受object参数）在重载解析中并未被考虑，因为带有Color参数的重载更为匹配，即使运行时类型是从Color派生出来的。

要使 Listing 7-9 中的测试通过，我们可以使类型特定的Equals方法在Color中变为虚方法，并在TranslucentColor中为其添加重写方法。另一种可能是移除Color中IEquatable< T >接口的实现，使得唯一的Equals方法成为虚方法。我们将失去类型特定比较的功能，虽然会有一些小的性能开销，但这个选项可以解决稳定性问题。然而，任何一种方法都解决了错误的问题。

实际上的根本问题是我们不恰当地使用了继承，而不是我们实现的相等性存在问题。要充分理解这一点，我们需要明确区分子类和子类型的不同。

## 包含多态性与子类型化

我们认为我们编写的类、结构、记录和记录结构是用户定义的类型。因此，按扩展来说，认为类的定义就是它的类型是很自然的。这个看法部分正确，但*类型*和*类*之间存在着更为正式的区别。

使用继承所提供的多态性被称为*包含*多态性。如果我们根据类型将系统中的所有对象进行分组，那么任何特定类型的每一组将*包括*所有继承自它的类型，这些类型被称为*子类型*。在我们的例子中，Color类型的组包括了Color和TranslucentColor类型。

对象的类型是其接口的契约，描述了对该对象可执行的操作。因此，类型定义的操作对其组中的任何子类型都是有效的。从实际角度来看，如果我们从某个给定类型派生，那么所有对基础类型对象有效的操作必须对派生类型的对象也有效，并且行为正确。

在我们的示例中，一个TranslucentColor对象既是一个Color类型的实例，也同时是一个TranslucentColor。这种关系意味着我们可以对一个TranslucentColor调用任何Color操作，这也意味着我们可以将一个TranslucentColor实例传递给一个接受Color参数的方法。就编译器而言，TranslucentColor必须能够支持其基类类型的所有操作，因此它允许这种替换。

对象的类型决定了子类型必须支持哪些操作，但它并未指定任何结构细节或具体实现。我们可以使用不同的类以不同的方式实现相同的接口。然而，虽然类型的契约并不要求具体的实现，它确实定义了任何操作的预期行为。当我们从一个具体类继承时，我们继承了它的实现，这就设定了对该行为的期望。当将类型实现为类或记录时，我们必须注意子类型化和子类化之间的区别，因为我们可以从引用类型继承，除非它们被显式封闭。对于 C#中的值类型，这不是问题，因为它们是隐式封闭的，因此不能有派生类型。

如本章前面所述，实现和接口继承是不同的。换句话说，简单地继承一个类并不等同于真正遵守它的行为特性。用更一般的*类型*编写的代码，定义了接口，可能会依赖于*类*的特定特性，而类代表了特定的实现。如果派生类没有遵守类型契约的行为方面，那么当我们在为基类类型编写的代码中使用派生类的实例时，该代码很可能会出现意外的行为。

当我们从一个类继承时，我们继承了它的行为、特性和预期。一个从另一个具体类继承的类就是子类，只有当使用基类*类型*的代码能够透明地使用派生类且观察不到任何行为变化时，它才是真正的子类型。当我们只继承类型时，就没有实现行为需要考虑。

从机械角度讲，我们可以在需要<code>Color</code>的地方替换为对TranslucentColor的引用，因为我们可以将派生类型的引用作为参数传递给期望基类型的方法。然而，正如你在Equals方法的行为中所见，交替使用<code>Color</code>和TranslucentColor实例并不是完全可行的。

Color和TranslucentColor之间缺乏可替代性，是因为TranslucentColor是一个*子类*，但不是<code>Color</code>的真正*子类型*。

### 处理虚方法的输入输出类型

子类型与子类之间的差异有着超出如何实现继承的Equals方法的影响。我们可以用适合派生类的实现来重写任何虚拟方法。如果调用更派生的方法所产生的可观察效果，包括任何副作用，与基类的效果完全相同，那么派生类型就是其基类型的良好替代—that，即派生类型是一个真正的子类型。副作用可能包括写入文件或屏幕，或更新一个方法外部可见的变量的值。如果派生类在其基类没有的情况下做了这些事情，它就不是一个真正的子类型。

方法的行为包括方法认为有效的输入和输出——即，方法接受哪些参数以及它可能返回什么——这些都会直接影响方法的调用者。举个例子，假设我们像示例 7-11 中一样，向<code>Color</code>类添加一个虚拟方法，用来从另一个<code>Color</code>值中减去一个<code>Color</code>值。

```
public virtual Color Subtract(Color? other)
{
    `--snip--`
}
```

示例 7-11: 为 Color 添加虚拟的 Subtract 方法

由于<code>Subtract</code>方法是虚拟的，我们可以在TranslucentColor中专门化其实现，以适当处理TranslucentColor实例的减法操作。不管实现<code>Subtract</code>的实际算法是什么，只要没有副作用，它的返回值就是该方法的可观察行为。

如果Subtract在Color中的实现从不返回null引用，但在TranslucentColor中被重写的版本可能会返回，那么TranslucentColor方法的行为契约就比基类方法更弱。允许TranslucentColor实现返回null值需要在调用代码中额外检查，以避免null引用异常。调用代码仅知道Color类型，合理地会期望只返回非null值。对返回类型的更弱要求意味着TranslucentColor不能替代Color。

当我们在重写的方法中*加强*对参数的要求时，也会出现类似情况。如果我们在派生类型中坚持使用非null值，但基类接受null引用，就打破了基类方法所建立的契约。同样，以基类编写的代码对派生类中的这些要求没有概念，并且很容易违反这些要求。

列表 7-11 中显示的Subtract方法通过使用自 C# v8.0 以来提供的可空引用类型特性，缓解了这两个潜在问题。基类Subtract方法的返回类型是不可空的，如果我们重写它并使用可空引用类型或者该方法尝试返回null引用，编译器会警告我们。同样，Color.Subtract方法的参数是一个可空引用，表示null是可以接受的参数。如果我们使用不可空引用类型重写该方法，编译器会警告我们方法签名与基类声明不匹配。

请注意，如果基类方法返回一个可空引用，而我们重写它以返回一个不可空引用，编译器也不会给出警告。这是因为在这种情况下，我们在派生方法中加强了行为契约，而对于更派生的方法来说，禁止 null 是完全合理的，即使基类方法允许它。通过基类类型引用调用方法的代码的要求并不受到影响。

同样，如果基类方法有一个不可空的参数，而我们重写该方法以允许传递一个 null 引用给派生方法，编译器将保持沉默，因为在更派生的类型中削弱参数的契约是安全且合理的。这个变化只对直接使用更派生类型的代码可见，而不是通过基类类型引用来使用。

### 遵守类型的契约

我们在清单 7-9 中使用基类引用进行的相等性测试失败了，因为派生类没有正确履行 Color.Equals 的契约。Color 所设定的期望并未被 TranslucentColor 满足，后者对 Equals 增加了新的要求，因为 TranslucentColor 实例之间的相等性还必须比较 Alpha 属性。我们的测试失败直接源于我们使用实现继承并期望类型可替代性，而实际上 TranslucentColor 并不能替代 Color。实现继承的影响适用于所有继承关系，而不仅仅是在建模值语义时。

遵循类型的契约在实际中非常重要。继承方法的行为是该契约的一部分，未能遵守它可能会导致极难诊断的错误。如果我们未能遵守基类的*接口*契约——例如，通过在重写方法中使用不同的签名——编译器会提示我们错误。然而，编译器无法检查我们是否遵守了基类的行为承诺。在这种情况下，我们必须依赖自己的判断，而这不一定像看起来那样简单。

一个经验法则是，每当我们使用继承时，应该避免实现继承。确保这一点的最简单方法是，永远不要从具有任何具体行为的类派生——包括具有任何非抽象方法的抽象类。使用interface关键字定义的类型不能有任何实现，任何实现接口的类都是一个真正的子类型。

另一个规则是，作为类实现的值类型不应该继承任何东西，并且应该是密封的。实际上，第二条规则是由第一条规则引发的：对于值类型来说，完全抽象几乎没有意义，因为值类型和类似值类型的一个定义特征就是根据它们所代表的值进行比较。因此，值类型是具体类型。内置的类似值类string以此为指导，这也是string故意被设计为密封类的原因。

与字符串类似，记录是引用类型，具有用于与Equals比较的值语义。与字符串不同，记录可以继承自其他记录，但就像从具体类派生时一样，派生的记录会继承所有基类记录的行为。因此，我们仍然需要小心，在派生的记录中遵守基类记录的承诺；然而，与类一样，做到这一点并不像看起来那么简单。尽管记录允许继承，但它们特别旨在建模值类型，因此对值类型应用密封的建议同样适用于记录。

### 继承记录类型

编译后，记录类型是一个类，具有一些编译器生成的方法，包括所有与基于值的相等性相关的内容。此外，使用位置语法定义的记录默认是不可变的。因此，使用记录而不是类来创建类似值的类型，可以避免编写大量样板代码。

与结构体不同，记录可以继承自其他记录，尽管它们不能与类形成继承关系。因此，我们可能会将Color和TranslucentColor类型重构为记录类型，如示例 7-12 所示。

```
public record Color(int Red, int Green, int Blue);
public record TranslucentColor(int Red, int Green, int Blue, int Alpha)
            : Color(Red, Green, Blue);
```

示例 7-12：继承记录类型

在这里，我们将<sup class="SANS_TheSansMonoCd_W5Regular_11">Color</sup>和<sup class="SANS_TheSansMonoCd_W5Regular_11">TranslucentColor</sup>定义为带有位置参数的记录，这些位置参数表示具有这些名称的只读属性，并且构造函数接受相同类型的参数。记录的继承语法与类的继承语法略有不同，因为我们需要在基类记录中初始化位置参数。<sup class="SANS_TheSansMonoCd_W5Regular_11">TranslucentColor</sup>记录从<sup class="SANS_TheSansMonoCd_W5Regular_11">Color</sup>派生，并将其<sup class="SANS_TheSansMonoCd_W5Regular_11">Red</sup>、<sup class="SANS_TheSansMonoCd_W5Regular_11">Green</sup>和<sup class="SANS_TheSansMonoCd_W5Regular_11">Blue</sup>参数值传递给<sup class="SANS_TheSansMonoCd_W5Regular_11">Color</sup>的相应位置参数。

正如我们在第五章中探讨的，编译器为我们生成了构造函数和属性的实现，以及<sup class="SANS_TheSansMonoCd_W5Regular_11">Equals</sup>的各种重写实现和一些方法的实现，包括基于值的<sup class="SANS_TheSansMonoCd_W5Regular_11">GetHashCode</sup>、<sup class="SANS_TheSansMonoCd_W5Regular_11">ToString</sup>等方法。记录变量之间的平等比较会比较每个属性的值，因此，如果两个记录变量的所有属性都相等，它们就相等。

如果我们愿意，可以编写自己实现类型安全的<sup class="SANS_TheSansMonoCd_W5Regular_11">Equals</sup>方法，这是由编译器创建的。然而，编译器提供的<sup class="SANS_TheSansMonoCd_W5Regular_11">Equals</sup>实现特别考虑了继承的情况。

#### <sup class="SANS_Futura_Std_Bold_Condensed_B_11">记录与平等合同</sup>

平等合同适用于记录类型，就像适用于任何其他类型一样，编译器提供的代码确保合同的各个方面都得到遵守，包括通过基类引用进行比较时的稳定性。列表 7-13 中的测试与列表 7-10 中的测试不同，因为<sup class="SANS_TheSansMonoCd_W5Regular_11">Color</sup>和<sup class="SANS_TheSansMonoCd_W5Regular_11">TranslucentColor</sup>类型是记录类型，而不是类类型。在这里，我们比较了两个具有不同<sup class="SANS_TheSansMonoCd_W5Regular_11">Alpha</sup>属性的<sup class="SANS_TheSansMonoCd_W5Regular_11">TranslucentColor</sup>记录值，并断言它们在直接使用具体类型进行比较或通过基类引用间接比较时都不相等。

```
bool EqualViaBase(Color left, Color right)
    => left.Equals(right);
bool EqualViaDerived(TranslucentColor left, TranslucentColor right)
    => left.Equals(right);
var pencil = new TranslucentColor(0xFF, 0, 0xFF, 0x77);
var crayon = new TranslucentColor(0xFF, 0, 0xFF, 0);
**Assert.That(EqualViaBase(pencil, crayon), Is.False);**
**Assert.That(EqualViaDerived(pencil, crayon), Is.False);**
```

<sup class="SANS_Futura_Std_Book_Oblique_I_11">列表 7-13：记录类型之间的平等</sup>

该测试通过，无论我们调用EqualViaDerived还是EqualViaBase方法，pencil和crayon变量比较结果始终为不相等。

由于编译器生成的平等实现特别关注平等契约，因此无论我们使用基类的Color记录引用还是派生类的TranslucentColor引用，变量都会被认为不相等。特别地，Color.Equals(Color)的类型安全实现是虚拟的，并且在派生的TranslucentColor记录中被重写。如前所述，若我们对类实现这样做，测试将正确执行。对于记录类型，编译器会为我们注入这些实现。

我们可以自行重写Equals方法，在这种情况下，编译器不会生成与我们自定义实现签名匹配的方法。然而，如果我们这么做，我们必须像编译器生成版本那样，特别注意平等契约。

在非密封记录中，编译器创建了一个虚拟属性，命名为EqualityContract，它使用typeof报告其包含记录的静态（编译时）类型。如在清单 7-14 中展示的Color记录的Equals实现与编译器生成的实现相同，尽管正如你在第五章中看到的，某些实现细节有所不同。

```
public class Color : IEquatable<Color>
{
    `--snip--`
    **protected virtual Type EqualityContract**
        **=>** **typeof(Color);**
 public virtual bool Equals(Color? other)
        => (object?)this == (object?)other ||
           other is not null &&
           **EqualityContract** **==** **other.EqualityContract &&**
           Red == other.Red && Green == other.Green && Blue == other.Blue;
}
```

清单 7-14：在非密封记录中使用平等契约

当一个记录继承自另一个记录时，正如TranslucentColor从Color继承一样，编译器会在派生记录中添加一个重写的EqualityContract，以报告其静态类型。基类中编译器生成的Equals实现会检查两个对象的EqualityContract属性是否匹配。如果不匹配，Equals将返回false。

尽管有EqualityContract属性，Equals的实现仍遵循清单 7-4 中展示的规范形式。由于TranslucentColor继承自Color，other参数可能引用TranslucentColor的实例。如果我们尝试将Color记录与TranslucentColor进行比较，EqualityContract属性将不匹配，两个对象将（正确地）被判定为不相等。检查EqualityContract属性类似于我们在清单 7-4 中检查原始Color类时检查GetType是否返回相同类型的对象。将静态类型用作EqualityContract相比于GetType有一个轻微的优势，因为typeof在编译时进行评估，而GetType是在运行时评估的。

EqualityContract属性是protected，以便由派生类型重写，但不能公开调用。如清单 7-15 所示，虚拟的EqualityContract属性在TranslucentColor记录中被重写，返回TranslucentColor的类型。

```
public class TranslucentColor : Color, IEquatable<TranslucentColor>
{
    `--snip--`
    protected override Type EqualityContract
        => typeof(TranslucentColor);
    **public override bool Equals(Color? obj)**
        **=>** **Equals(obj as TranslucentColor);**
    public virtual bool Equals(TranslucentColor? other)
        => base.Equals(other) && Alpha == other.Alpha;
}
```

清单 7-15：在 TranslucentColor 记录中重写相等契约

TranslucentColor 中的 Equals 实现会在比较每个对象的本地属性之前调用基类的实现，以确保始终比较契约属性。关键是，虚拟的 Equals(Color?) 方法在 TranslucentColor 中被重写，并将其参数强制转换为 TranslucentColor。如果转换失败，传入的参数将是 null。当我们使用 Color 引用变量比较两个 TranslucentColor 实例时，就像在 示例 7-13 中调用 EqualViaBase 方法一样，是通过虚拟分发执行的重写版本的 Equals 方法。

如果我们编写自己的 Equals 方法，它也必须比较 EqualityContract 属性，因为不同类型的实例比较相等通常没有意义。

仅仅有 EqualityContract 属性并不能解决使用 Color 引用比较两个 TranslucentColor 实例的问题。我们在 示例 7-13 中的测试通过了，因为编译器为 Color 生成了一个虚拟的类型安全的 Equals 方法，并在派生记录中重写了它。当我们在 Color 变量上调用 Equals 时，如果运行时实例是 TranslucentColor，我们就会调用更派生的实现。请注意，Equals(TranslucentColor?) 方法也是虚拟的，因为 TranslucentColor 本身可以被继承。从 TranslucentColor 派生的记录将会生成编译器的重写版本，分别是 Equals(TranslucentColor?) 和 Equals(Color?)，并且还会有它自己的类型安全的 Equals 方法。

然而，平等性并不是我们可以从类或记录中继承的唯一实现。我们可以像为类定义虚拟和非虚拟方法一样，为记录定义我们自己的虚拟和非虚拟方法。

#### 除平等性之外的契约

基类或记录所建立的行为契约适用于其所有方法，而不仅仅是Equals。编译器会生成比较两个记录实例是否相等的正确实现，但我们必须为其他任何实现提供自己的代码。值类型实现的一个常见接口是IComparable< T >，正如你在第六章中看到的，它允许我们对值类型的集合进行排序。Listing 7-16 中的Area和Volume记录通过继承相关联，并且每个都通过定义CompareTo方法来实现IComparable< T >接口。

```
public record Area(double Width, double Height)
    : IComparable<Area>
{
    public int CompareTo(Area? other)
    {
        if(other is null) return 1;
        return (int)(Width * Height - other.Width * other.Height);
    }
    public static bool operator<(Area left, Area right)
        => left.CompareTo(right) < 0;
    public static bool operator>(Area left, Area right)
        => left.CompareTo(right) > 0;
}
public record Volume(double Width, double Height, double Depth)
    : Area(Width, Height), IComparable<Volume>
{
    public int CompareTo(Volume? other)
    { if(other is null) return 1;
        return (int)(Width * Height * Depth -
                     other.Width * other.Height * other.Depth);
    }
    public static bool operator<(Volume left, Volume right)
       => left.CompareTo(right) < 0;
    public static bool operator>(Volume left, Volume right)
       => left.CompareTo(right) > 0;
}
```

Listing 7-16: 使用 IComparable 排序 Area 和 Volume 记录< T >

编译器会为Area和Volume生成实现IEquatable< T >的代码，尽管我们应该记住，Equals在每种情况下比较的是double值，这可能会导致问题，正如我们在第五章中发现的那样。然而，编译器并没有提供IComparable< T >的实现，因此我们必须自己编写。在这里，我们为Area定义了排序规则，当一个对象的总面积较小时，它小于另一个对象。类似地，对于Volume，当一个对象的总体积较小时，它小于另一个对象。我们还为Area和Volume添加了operator<和operator>，这些运算符是通过CompareTo方法实现的。

正如我们在第六章中探讨的那样，排序比较有其自身的契约，而在示例 7-16 中的`IComparable<T>`实现，实际上遭遇了与我们原始实现的`IEquatable<T>`接口在`Color`和`TranslucentColor`类中的相同问题。

尽管在声明中我们使用了`record`而不是`class`，但我们仍然在使用实现继承。子类化对于记录类型的问题与类类型一样。我们可以通过一个新的测试来演示这一点，就像检查两个`TranslucentColor`实例在它们的`Alpha`属性不同的情况下是否不相等一样。在示例 7-17 中，两个`Volume`实例仅在它们的`Depth`属性上有所不同，但我们通过使用基记录类型的引用，使用`<`进行比较。

```
Area door =   new Volume(Width: 100, Height: 200, Depth: 25);
Area window = new Volume(Width: 100, Height: 200, Depth: 5);
Assert.That(window < door, Is.True);
```

示例 7-17：测试两个 Volume 实例中 CompareTo 契约

这个测试*失败*了，因为`door`和`window`变量的静态编译时类型与它们的动态运行时类型不同。当我们使用基类的静态类型进行比较时，派生记录的`Depth`属性被忽略，导致了错误的结果。

正如编译器实现的IEquatable< T >一样，我们可以将CompareTo的实现定义为虚拟方法，放在面积记录中，并在体积类型中重写它。尽管这样做可以解决当前测试失败的问题，但它并不能解决我们实现中的所有问题。例如，当我们比较一个面积和一个体积时，CompareTo应该返回什么？不同类型实例之间的相等比较会简单地返回false，但对CompareTo来说就不那么简单了。我们可能会选择将任何面积视为小于任何体积，但这也可能导致混淆。

面积是否小于体积的问题并没有实际意义，但禁止对面积和体积类型分别进行排序比较将极为繁琐；比较两个面积是否大小关系，或比较两个体积也是非常合理的。我们可以让CompareTo在比较的对象类型不同时抛出异常，但这会增加调用代码的复杂性，可能会让某些用户感到困惑。

这首先证明了记录并不是“灵丹妙药”，更重要的是，它表明我们可能仍在试图解决错误的问题。

### 避免实现继承

我们继承Equals和CompareTo时遇到的问题表明，值类型并不适合作为基类型，无论我们是使用class还是record来定义它们。

更一般地说，继承任何已实现的行为使得确保为基类型编写的代码在替换继承类型时能正确运行变得具有挑战性。即使我们没有重写基类型的方法，我们也无法轻易保证这些方法对于任何派生类型都能正确工作。虽然继承是重用基类型实现的常见机制，但在派生类型中履行基类型行为合同通常比看起来要困难得多。

确保一种类型可以替代另一种类型的一种方法是完全避免实现继承。记住，当我们实现一个接口时，实现类实际上是接口类型的子类型；因为接口没有实现，所以没有行为合同需要考虑。接口类型定义了实现类型必须能够做的事情，但并不规定任何具体的实现。接口实际上仅仅定义了*一个*类型，而不是一个类。

接口类型可以被任何实现类型替代，因此我们可以在不同的情况下使用不同的实现。仅依赖于接口类型的代码——无论是在方法中的参数，还是在类型中的字段——都完全与接口的实现方式解耦。这意味着接口是*接口*——我们代码中的定制点，允许我们在不同实现之间进行切换。

用接口类型而非具体实现编写的代码更具灵活性，因为它不依赖于特定的实现。它也更容易测试，因为我们可以用自己的*测试替身*（有时称为*存根*、*假对象*或*模拟对象*）来替换接口的具体实现。

通常我们会看到接口类型代表第六章中描述的控制器和服务，有时会有多种实现。特定的具体实现可能会在运行时根据配置参数或运行环境选择。然而，*使用*控制器或服务的代码——通常是在实体类型中——不需要修改，因为它的行为仅依赖于接口，而不是具体的实现类型。控制器和服务通常是我们在测试中最希望使用假的实现的地方，这样测试就不需要访问外部或昂贵的资源，比如真实的数据库。

当值类型实现一个或多个接口时，它是为了定义特定的协议，例如IEquatable< T > 和 IComparable< T >，而不是为了允许客户端代码使用不同的实现。值类型，无论我们使用什么机制来实现它们，都应该独立存在，并且在应用程序中应该大多数、如果不是完全独立于其他类型。

避免实现继承的建议导致了“记录应该*始终*是密封的”这一推荐，因为它们专门用于建模值类型。类也应该默认是密封的，无论我们是否使用它们来建模值，只有在我们有特定设计理由时才启用继承。

虽然这个建议可能看起来限制了我们设计的灵活性，但我们可以通过其他方式在代码中定义关系，而无需从现有类型派生新类型。继承并不是唯一可以复用现有类型实现并扩展其功能的选项。

### 包含类型而非继承类型

我们可以通过将一个具体类型的实例作为字段或属性来简单地*包含*（或*组合*）另一个类型的行为，从而实现对另一个类型的实现。这在我们需要值类型时尤其适用，比如 TranslucentColor，它可以通过更简单的类型 Color 来实现，但并不意味着它们之间有任何类型替代关系。虽然值类型通常应该是独立的，但作为字段包含另一个值类型是一个经常带来好处的例外。

我们先将 Color 实现为类，再实现为记录，以便利用继承。用 class 来建模值并不是不合理的——而且，正如我们所知，记录专门用于此目的。但如果我们在 TranslucentColor 中*包含*一个 Color 实例，而不是从 Color 派生，那么使用 struct 来实现这两种类型会更简单。使用 record struct 甚至更简单，就像我们在清单 7-18 中所做的那样，其中 TranslucentColor 包含一个 Color 实例。

```
public readonly record struct Color(int Red, int Green, int Blue);
public readonly record struct TranslucentColor**(Color Color, int Alpha)**
{
    public TranslucentColor(int red, int green, int blue, int alpha)
        : this(new Color(red, green, blue), alpha)
    {
    }
 public int Red => Color.Red;
    public int Green => Color.Green;
    public int Blue => Color.Blue;
}
```

清单 7-18：包含颜色而非继承颜色

在这里，编译器为每种类型提供了 IEquatable< T > 的实现，我们只需要定义这些类型的属性和行为。TranslucentColor 类型包含一个只读的 Color 实例，并且我们增加了一个新的构造函数，方便我们的用户创建新的 Color 值并传递给 TranslucentColor 生成的构造函数，或者通过分别传递每个组件调用我们的新构造函数。我们还在 TranslucentColor 中镜像了 Color 的属性，并将它们转发到包含的 Color 值。我们没有免费获得这些属性，但它们为 TranslucentColor 的用户提供了一个更自然的接口，像这样：

```
var bg = new TranslucentColor(0xFF, 0xA0, 0, 0x77);
Assert.That(bg.Red,   Is.EqualTo(0xFF));
Assert.That(bg.Green, Is.EqualTo(0xA0));
Assert.That(bg.Blue,  Is.EqualTo(0));
Assert.That(bg.Alpha, Is.EqualTo(0x77));
`--snip--`
```

另一种选择将强迫用户显式获取 Color 属性，以便访问其属性，像这样：

```
Assert.That(bg.Color.Red,   Is.EqualTo(0xFF));
Assert.That(bg.Color.Green, Is.EqualTo(0xA0));
Assert.That(bg.Color.Blue,  Is.EqualTo(0));
Assert.That(bg.Alpha,       Is.EqualTo(0x77));
`--snip--`
```

无论我们是使用结构体，还是通过封闭记录或类，或记录结构体来定义值类型，测试我们新的类型现在变得更容易理解，因为我们不需要考虑 TranslucentColor 实例是否通过 Color 引用进行引用。这本身是一个重要的考虑，因为这些测试不仅更容易编写，而且下一个访问代码的程序员也能更容易阅读。

类型组合与采用继承的版本并不完全匹配，因为我们不能将 TranslucentColor 实例作为方法参数传递给期望接收 Color 的方法。正如你所见，有时这种替代性并不合适。

继承并不是唯一的多态形式，也不是唯一展示类型替代性的机制，但其他方法允许编译器在我们错误地替代某个类型时通知我们。让我们来看看其中的一些。

## 使用泛型的参数多态性

C# 泛型提供了*参数多态性*，这是一种多态性形式，通过使用泛型类型参数而非实际类型，可以编写一次代码以适应多种类型。这种方法为所有能够替代这些参数的类型提供了一个共同的形式和目的。

这一点在标准库中的泛型集合类中得到了最清晰的展示，比如 List< T >，其中 T 是一个泛型参数类型，可以由 *任何* 运行时类型替代，包括我们自己定义的类型。例如，在 清单 7-19 中，我们声明了两个泛型类型为 List< T > 的变量，分别使用不同且不相关的类型。

```
var colors = new List<Color>();
var names = new List<string>();
```

清单 7-19: 使用泛型类型

尽管 List 实现的行为没有改变，但 List< Color> 是与 List< string> 完全不同的类型，这两个类型之间没有任何关系。泛型 List< T > 代码是根据 T 泛型参数编写的，而由于 List< T > 不需要了解 T 的结构或行为特征，因此它可以与任何类型一起使用。

换句话说，在 List< T > 的上下文中，任何类型都可以替代 T 参数，而不意味着任何子类型关系。我们不需要考虑行为契约，因为 List< T > 对 T 没有任何假设。

如果我们需要对适用于泛型参数类型的类型进行更精确的选择，或者如果我们要求泛型代码使用比 object 提供的更多方法和属性，我们可以对参数进行约束，只允许具有特定行为的类型。

### 泛型约束与协议接口

因为 object 是所有类型的基类，所以泛型可以通过类型为 T 的变量使用其方法，但要访问其他内容时，编译器需要更多关于 T 可以是什么的信息。我们通过泛型类型约束提供这些信息。一个例子是接口约束，它将 T 限制为实现指定接口的类型，确保该泛型类型的变量对所有接口操作都是合法的。例如，考虑 清单 7-20 中展示的接口。

```
public interface IParser<T>
{
    public T Parse(string input);
}
```

列表 7-20：一个契约接口

泛型接口 `IParser< T >` 定义了一个将字符串值转换为 `T` 类型对象实例的 `Parse` 方法。`IParser< T >` 中的 `T` 参数没有约束，因此此接口可以被任何类型实现。在 列表 7-21 中，我们使用 `IParser< T >` 接口来限制 `DataAdapter` 泛型类的 `TParser` 参数。

```
public sealed class DataAdapter<**TParser**, TResult>
    **where TParser : IParser<TResult>**
{
    public DataAdapter(TParser parser, IEnumerable<string> source)
        => (this.parser, items) = (parser, source);
    public IEnumerable<TResult> Read()
    {
        foreach (var item in items)
        {
            yield return **parser.Parse(item);**
        }
    }
    private readonly TParser parser;
    private readonly IEnumerable<string> items;
}
```

列表 7-21：为其 API 限制一个类型

`DataAdapter` 类有两个泛型参数。`TParser` 参数在类型定义后的 `where` 子句中被限制为 `IParser< T >` 接口。`TParser` 通过第二个泛型参数 `TResult` 进行约束，`TResult` 也对应于 `Read` 方法的返回类型，这意味着 `TParser` 可以由 `IParser< TResult >` 的实现来替代。`DataAdapter` 的构造函数接受一个 `TParser` 参数，因此传递的参数必须是 `IParser< T >` 的实现，且 `T` 必须与 `DataAdapter` 的 `TResult` 参数的类型一致。

为简化起见，`DataAdapter` 的构造函数接受一系列字符串值作为输入值，但在实际应用中，`DataAdapter` 可能是从数据库或更复杂的数据源获取数据。

对 TParser 泛型类型参数的接口类型约束使得我们可以在 Read 方法中调用 parser.Parse，该方法返回一个 TResult 元素的序列。如果没有 where 约束来限制 TParser，则 Read 方法将无法编译，因为 object 类型没有 Parse 方法。

示例 7-20 中的 IParser< T > 接口并非用于作为变量类型；相反，它是一个契约接口，其目的是描述解析字符串为对象的*协议*。我们甚至在 DataAdapter 类中使用 TParser 作为字段类型，而不是声明字段为 IParser< T >。

对 TParser 泛型参数的约束存在于 DataAdapter 中，意味着我们只能通过提供 IParser< T > 协议的实现来创建一个 DataAdapter。该约束保证了在运行时为 TParser 替代的任何类型都会有一个 Parse 方法，其签名与 IParser< T > 接口中定义的操作相匹配。

#### 实现 IParser<T> 协议

IParser< T > 接口本身是泛型的，允许实现的类型指定 Parse 方法的返回类型。示例 7-22 中的 ColorParser 类实现了 IParser< Color> 接口，将一个字符串转换为 Color 对象。在此示例中，输入字符串表示每个颜色组件的两位十六进制值，因此整个值的格式为 "RRGGBB"。

```
public interface IParser<T>
{
    public T Parse(string input);
}
public sealed class ColorParser : **IParser<Color>**
{
    public static int FromHex(string part)
        => int.Parse(part, NumberStyles.HexNumber);
    **public Color Parse(string input)**
        => new(Red:   FromHex(input[0..2]),
               Green: FromHex(input[2..4]),
               Blue:  FromHex(input[4..6]));
}
```

示例 7-22：实现一个契约接口

ColorParser 类的 Parse 方法使用了 C# v8.0 中引入的范围操作符语法，在 input 参数上将字符串拆分为三个部分，每部分两个字符。像 [begin..end] 这样的范围，也叫做 *切片*，表示从 begin 索引开始，到但不包括 end 索引。范围还可以与数组一起使用，指定数组的子范围。

> 注意

*一个* 范围 *是一个半开区间，应该更准确地写为 begin..end)，但 C# 语法不允许不匹配的括号或圆括号。请小心不要将此语法与 Enumerable.Range 方法混淆，该方法接受起始索引和要包括的项数作为参数。*

#### 为 DataAdapter 类参数化

由于 ColorParser 实现了 IParser< T > 接口，我们可以将 ColorParser 与 DataAdapter 类一起使用，如 [列表 7-23 所示。

```
string messages = "FFA000 A0FF00 00F0F0"; …
var provider = new DataAdapter<ColorParser, **Color**>
                        (new ColorParser(), messages.Split(' '));
foreach(Color color in provider.Read()) {
    `--snip--`
    // Do something with a color
}
```

列表 7-23：使用泛型类型

虽然 DataAdapter 类本身是以多态的方式编写的（因为它可以处理任何实现了所需的 IParser< T > 协议的类型），但使用它时我们需要显式指定我们为 TParser 参数和 Read 方法返回的 TResult 参数类型所替代的具体类型。

这可以防止我们不小心将 TranslucentColor 用作 DataAdapter 的 TResult 参数，并与 ColorParser 类一起使用，像这样：

```
var other = new DataAdapter<ColorParser, **TranslucentColor**>
                    (new ColorParser(), messages.Split(' '));
```

ColorParser 类是专门针对 Color 类型的，因为它实现了 IParser< Color> 接口。编译器会捕捉到这种违规行为并报告错误：

```
[CS0311] The type 'ColorParser' cannot be used as type parameter 'TParser' in the generic
type or method 'DataAdapter<TParser, T>'. There is no implicit reference conversion from
'ColorParser' to 'IParser<TranslucentColor>'.
```

然而，DataAdapter 的 TResult 泛型参数已经由我们提供的具体 IParser< T > 实现的类型隐式地指定，因为它必须与 IParser< T >.Parse 返回的类型相同。我们在 Listing 7-21 中为 DataAdapter 的 TParser 泛型参数使用的类型约束使这种关系变得明确：

```
public sealed class DataAdapter<TParser, TResult>
    where TParser : IParser<TResult>
`--snip--`
```

由于我们已经费尽心思确保 DataAdapter 类可以与任何 IParser< T > 实现一起工作，因此指定我们指的哪种实现似乎是多余的。相反，我们可以让编译器根据实际使用的类型推导出 TParser 参数的正确类型。

### 泛型方法参数和类型推导

虽然编译器不会为任何泛型类的参数推导实际类型，但如果泛型参数类型在方法的形式参数中使用，它可以为泛型方法推导类型。由于 TParser 类型参数仅被 DataAdapter.Read 方法使用，我们可以将其从 DataAdapter 类中移除，并将其添加到 Read 方法中，使 Read 成为一个泛型方法，如 Listing 7-24 所示。

```
public sealed class DataAdapter<TResult>
{
    public DataAdapter(IEnumerable<string> source)
        => items = source;
    **public IEnumerable<TResult>** **Read<TParser>(TParser parser)**
        **where TParser : IParser<TResult>**
    {
        foreach (var item in items)
        {
            yield return parser.Parse(item);
        }
    }
    private readonly IEnumerable<string> items;
}
```

Listing 7-24: 将 DataAdapter.Read 定义为泛型方法

DataAdapter 不再需要一个字段来存储 TParser 对象，因为它已经被传递到 Read 方法中。泛型方法仍然要求接口约束，以便我们能够通过 parser 变量调用 Parse 方法，但在传递时我们不需要指定解析器参数的类型；编译器根据传递给 Read 的参数推断 TParser 的类型，如 Listing 7-25 所示。

```
var provider = new DataAdapter<Color>(messages);
foreach (Color color in provider.Read(new ColorParser()))
{
    `--snip--`
}
```

<sup class="SANS_Futura_Std_Book_Oblique_I_11">列表 7-25：参数类型推断</sup>

我们仅在创建<sup class="SANS_TheSansMonoCd_W5Regular_11">ColorParser</sup>实例并将其传递给<sup class="SANS_TheSansMonoCd_W5Regular_11">Read</sup>方法时提到过<sup class="SANS_TheSansMonoCd_W5Regular_11">ColorParser</sup>类型。将此与列表 7-23 进行对比，在那里我们不仅需要<sup class="SANS_TheSansMonoCd_W5Regular_11">ColorParser</sup>的实例，还需要为<sup class="SANS_TheSansMonoCd_W5Regular_11">DataAdapter</sup>的<sup class="SANS_TheSansMonoCd_W5Regular_11">TParser</sup>参数指定其类型。通过利用泛型方法提供的类型推断功能，我们避免了冗余代码。

### <sup class="SANS_Futura_Std_Bold_Condensed_Oblique_BI_11">参数化类型</sup>

我们仍然需要为列表 7-25 中的<sup class="SANS_TheSansMonoCd_W5Regular_11">TResult</sup>参数指定<sup class="SANS_TheSansMonoCd_W5Regular_11">Color</sup>，即使<sup class="SANS_TheSansMonoCd_W5Regular_11">TResult</sup>仅由<sup class="SANS_TheSansMonoCd_W5Regular_11">Read</sup>方法使用。编译器只能根据我们传递给方法的参数推断泛型参数的实际类型，而<sup class="SANS_TheSansMonoCd_W5Regular_11">TResult</sup>在<sup class="SANS_TheSansMonoCd_W5Regular_11">Read</sup>方法中并未作为任何参数的类型使用。如果一个方法有泛型参数，它们必须全部明确指定或从传递的参数中推断；编译器不会仅根据可用的参数部分推断类型。

然而，这是<sup class="SANS_TheSansMonoCd_W5Regular_11">DataAdapter</sup>类的一个好处，因为它确保了<IParser< T >`的<sup class="SANS_TheSansMonoCd_W5Regular_11">T</sup>参数与<sup class="SANS_TheSansMonoCd_W5Regular_11">DataAdapter</sup>的<sup class="SANS_TheSansMonoCd_W5Regular_11">TResult</sup>参数匹配。如果我们想用不同的类型替代<sup class="SANS_TheSansMonoCd_W5Regular_11">TResult</sup>，我们需要一个不同的解析器实现。在列表 7-26 中，我们在一个<sup class="SANS_TheSansMonoCd_W5Regular_11">TranslucentColor</sup>类型中实现了<sup class="SANS_TheSansMonoCd_W5Regular_11">IParser< T ></sup>接口，并为新的类型创建了一个<sup class="SANS_TheSansMonoCd_W5Regular_11">DataAdapter</sup>。

```
public sealed class **TranslucentColorParser : IParser<TranslucentColor>**
{
    **public TranslucentColor Parse(string input)**
        => new(Color: color.Parse(input[0..6]),
              Alpha: ColorParser.FromHex(input[6..8]));
    private readonly ColorParser color = new();
}
`--snip--`
var provider = new DataAdapter<TranslucentColor>(messages);
var colors = provider.Read(new TranslucentColorParser()).ToList();
```

<sup class="SANS_Futura_Std_Book_Oblique_I_11">列表 7-26：使用不同类型参数化 DataAdapter</sup>

我们在 TranslucentColorParser 类中实现 IParser< T > 时，指定了 TranslucentColor 而不是 Color，并且我们将 TranslucentColor 作为 DataAdapter 的 TResult 参数的类型。TranslucentColorParser 的实现使用了一个 ColorParser 对象来解析 TranslucentColor 的 Color 部分，作为一种便利，但除此之外，它是一个完全新的类型。类似地，DataAdapter< TranslucentColor> 类型与 DataAdapter< Color> 无关。

DataAdapter 类是多态的，具体取决于我们作为参数传递给其 TResult 参数的类型，因为该类型影响 Read 方法的返回值。Read 方法本身也是多态的，因为它有自己的泛型参数。我们只需要编写一次 Read 方法，它适用于任何实现了 IParser< T > 的类型，其中 T 与 DataAdapter 的 TResult 类型匹配。

我们可以将泛型方法看作是表示多个方法重载，每个重载具有不同的参数类型，但都具有相同的实现。即使没有泛型，重载方法也代表了一种特殊的多态性，称为*临时多态性*。

## 临时多态性与方法重载

临时多态性，或称为*方法重载*，是我们定义一组具有相同名称但在参数的类型或数量上有所不同的操作的方式。编译器根据方法名称和我们调用方法时使用的参数来选择正确的方法重载。每个方法可以有不同的实现，因此方法的*名称*在参数上是多态的。

在本章及其他章节中，你已经看过了一些重载实例方法的例子，我们重写了虚拟的Equals方法，然后用类型安全的实现重载了它。如果参数的静态类型与实现类型匹配，而不是一个object或其他类型，编译器将选择类型安全的Equals重载。在记录结构体中，编译器为这两个方法提供了实现，尽管如果需要，我们可以提供自己的类型安全的Equals方法。清单 7-27 展示了使用不同参数如何改变在比较变量是值类型时调用的方法。

```
public readonly record struct Color(int Red, int Green, int Blue);
var plum = new Color(0xDD, 0xA0, 0xDD);
var other = new Color(0xDD, 0xA0, 0xDD);
Assert.That(plum.Equals(**null**), Is.False);
Assert.That(plum.Equals(**other**), Is.True);
```

清单 7-27：选择方法重载

第一个断言比较plum变量与null，将调用带有object?参数的Equals方法重载，因为object是引用类型，且null会自动转换为引用参数。在第二个断言中，接受Color作为参数的方法更适合另一个参数，因为类型完全匹配，因此调用了特定类型的重载。如果Color是记录而不是记录结构体，那么这两个断言将直接调用Equals(Color)重载，因为在这种情况下Color将是引用类型，但比object更为具体，使其成为重载解析时更好的转换目标，尤其当参数为null时。

当我们调用一个重载的实例方法时，编译器通过使用调用方法的变量的静态类型来识别候选方法。如果在方法调用的作用域内存在同名的扩展方法，候选方法中也可能包括这些扩展方法。始终是调用变量决定了如何选择可能的重载方法列表，而传递的参数则决定了从候选列表中选择哪个具体的重载方法。在列表 7-28 中，我们在调用<sup class="SANS_TheSansMonoCd_W5Regular_11">Equals方法之前，将<sup class="SANS_TheSansMonoCd_W5Regular_11">plum变量的静态类型从<sup class="SANS_TheSansMonoCd_W5Regular_11">Color改为<sup class="SANS_TheSansMonoCd_W5Regular_11">object。

```
**object plum** = new Color(0xDD, 0xA0, 0xDD);
Color other = new Color(0xDD, 0xA0, 0xDD);
Assert.That(plum.Equals(other), Is.True);
```

列表 7-28：参数类型与调用类型

Equals方法的候选方法是从<sup class="SANS_TheSansMonoCd_W5Regular_11">object上定义的方法中选择的，因为那是<sup class="SANS_TheSansMonoCd_W5Regular_11">plum变量的编译时类型。我们只有一个这样的重载方法，它接受一个<sup class="SANS_TheSansMonoCd_W5Regular_11">object?类型的参数，因此这自动成为匹配项，即使<sup class="SANS_TheSansMonoCd_W5Regular_11">其他</sup>参数是<sup class="SANS_TheSansMonoCd_W5Regular_11">Color类型，并且<sup class="SANS_TheSansMonoCd_W5Regular_11">plum仍然是一个指向<sup class="SANS_TheSansMonoCd_W5Regular_11">Color的引用，该类型有一个重载的<sup class="SANS_TheSansMonoCd_W5Regular_11">Equals方法，接受<sup class="SANS_TheSansMonoCd_W5Regular_11">Color参数。如果<sup class="SANS_TheSansMonoCd_W5Regular_11">Color是引用类型，情况也相同：类型特定的重载在重载解析过程中不会被考虑，因为它不是用于调用该方法的变量类型的成员。

静态方法也可以重载，尽管候选重载方法是通过调用者使用的类型名称来识别的。在这两种情况下，从这些候选方法中，称为*方法组*，编译器根据传递的参数选择最佳匹配项。

如果没有找到匹配项——也就是说，参数不能隐式转换为任何参数类型，或者有多个同样好的候选方法但没有明确的最佳匹配——我们的程序将无法编译。

### 使用重载运算符的符号多态性

重载在结合自定义运算符时尤其强大。值类型通常会重载 operator==，以与 Equals 方法对应。这样不仅更加简洁，而且用 == 比较值比通过调用方法来比较它们看起来更自然。

我们必须为结构体编写自己的运算符实现，但编译器为记录和记录结构体提供了 operator== 和 operator!=，这使得比较同一类型的两个变量变得非常方便，如下所示：

```
var plum = new Color(0xDD, 0xA0, 0xDD);
var pink = new Color(0xFF, 0xCC, 0xCC);
Assert.That(plum != pink, Is.True);
```

我们不能修改编译器为记录（record）和记录结构体（record structs）合成的 operator== 或 operator!= 的实现，但我们可以为它们添加重载，以接受不同的类型，就像我们可以为其他方法添加重载一样。例如，在 Listing 7-29 中，我们为 Color 重载了 operator==，以便允许在 Color 和 int 之间进行比较。

```
public static bool operator==(Color left, int right)
    => left.Equals(new (right));
public static bool operator==(int left, Color right)
    => right.Equals(new (left));
```

Listing 7-29: 重载运算符

我们需要为每个重载添加一个相应的 operator!=（此处未显示）。这些重载为用户提供了便利，用户无需显式地构造 Color 实例即可将其与原始的 RGB 值进行比较，并且可以像这样进行比较：

```
var plum = new Color(Red: 0xDD, Green: 0xA0, Blue: 0xDD);
Assert.That(plum == 0xDDA0DD, Is.True);
Assert.That(0xDDA0DD == plum, Is.True);
```

重载运算符本质上与重载其他方法没有太大区别，区别在于我们不是使用命名的方法，而是重载*符号*，使得这些符号能够以多态的方式与我们的类型一起工作。符号多态的一个好例子体现在 string 类中，它将 + 符号定义为连接符而非加法符号。这是一个被大多数程序员广泛接受的约定。

我们应谨慎引入那些不遵循常规规则的操作。方法重载，特别是运算符重载，需要仔细思考，并且需要一些我们可能称之为“良好品味”的东西。为不同类型重载的函数族，通过给操作一个通用名称，使得该操作对于每个类型可能有不同的实现，给人一种类型可替代性的印象。

例如，string类不允许我们将数字与字符串相加，原因很简单，因为结果的类型可能会被误解：应该认为"5" + 0.5和0.5 + "5"是相同的吗？string类的设计者决定不允许这两种用法，以避免任何潜在的混淆。

### 多态的泛型委托

*委托*是一种表示具有特定签名（即参数类型和数量）的方法的类型，并且只要签名与委托类型匹配，委托对象就可以从不同的方法构造。委托是 LINQ 库的核心特性；例如，Select方法接受一个委托参数，表示将序列中的一个元素转换为不同类型的方法。我们最常见的是看到 lambda 作为带有委托类型参数的方法的参数，如清单 7-30 所示。

```
var colors = new List<Color>
    {
        `--snip--`
    };
var formatted = colors.Select(
    color => $"{color.Red:X2}{color.Green:X2}{color.Blue:X2}");
```

清单 7-30：为委托参数传递 lambda

Select是一个用于<IEnumerable< T >>的扩展方法，在这里我们通过< s   amp class="SANS_TheSansMonoCd_W5Regular_11">colors变量调用它，并传递一个 lambda 表示一个接受<Color>参数的方法，因为<Color>是< s   amp class="SANS_TheSansMonoCd_W5Regular_11">colors序列的元素类型。该 lambda 会为序列中的每个元素调用，并返回该值的十六进制表示，作为一个格式化为"RRGGBB"的字符串——这是清单 7-22 中定义的< samp class="SANS_TheSansMonoCd_W5Regular_11">Parse方法的反向操作。

像列表 7-30 中那样的内联 lambda 表达式很方便，但通常缺乏方法重载所提供的灵活性。例如，如果我们将colors的元素类型从Color更改为TranslucentColor，我们的代码仍然能够编译，并且 lambda 实现将继续与列表 7-18 中定义的TranslucentColor类型一起工作，但结果不会为Alpha属性额外占用 2 个字节。我们必须为TranslucentColor编写一个新的 lambda 表达式，如果我们需要同时支持Color和TranslucentColor元素，我们就必须分别处理它们。

重载方法是捕获我们需要的共同目的的完美方式，同时使我们能够封装所需的不同实现。请参考列表 7-31 中的两个静态方法。

```
public static class Formatter
{
    public static string Format(Color color)
        => $"{color.Red:X2}{color.Green:X2}{color.Blue:X2}";
    public static string Format(TranslucentColor color)
        => $"{**Format(color.Color)**}{color.Alpha:X2}";
}
```

列表 7-31：用于不同类型的重载方法

请注意，Format(TranslucentColor)方法的实现调用了Format(Color)重载——这是我们无法通过单独的匿名 lambda 表达式来做到的。

我们可以将<sup>Format方法组作为参数传递给Select，而不是传递一个 lambda 表达式，如列表 7-32 所示。

```
var colors = new List<TranslucentColor>
{
    `--snip--`
};
var formatted = colors.Select(**Formatter.Format**);
```

列表 7-32：作为参数的方法组

在这里，Formatter.Format是两个方法重载的通用名称，代表一个方法组。编译器根据用于调用Select的序列元素类型，选择方法组中的正确重载。Select的委托参数是一个泛型委托，也就是说，它有自己的泛型类型参数。像泛型方法一样，编译器会根据传递给委托的参数推断出实际的类型。

Formatter.Format 方法组根据传递给内部 Select 方法的参数具有多态性。在这里，由于 colors 序列的元素类型是 TranslucentColor，所以调用了 清单 7-31 中的 Format(TranslucentColor) 方法。如果我们将 colors 变量改为 List< Color>，则 Select 方法将调用 Format(Color)，但我们无需以任何方式更改 Select 表达式。

## 使用转换的强制多态性

如你所见，继承允许我们在需要不同类型时，使用某种类型的实例作为引用，只要第一个类型继承自第二个类型。派生类在语法上可以替代其基类，因为从特定类型到任何父类类型都有一个自然的隐式转换。

我们可以实现自己的类型转换，以模拟两个本无关类型之间的可替换性。将变量转换（或 *强制*）为不同的类型是方便的，无论是通过隐式转换还是显式转换，但这样做可能会掩盖表面下的问题。然而，谨慎应用时，不相关类型之间的转换可以成为表达设计的一种有效且简洁的方式。

为了演示一些我们尚未探索的隐式转换问题，清单 7-33 在 TranslucentColor 中实现了一个隐式转换运算符，将实例转换为 Color 类型。

```
public readonly record struct TranslucentColor(Color Color, int Alpha)
{
    `--snip--`
    public static implicit operator Color(TranslucentColor color)
        => color.Color;
}
```

清单 7-33：隐式转换运算符

TranslucentColor 中的转换运算符是一个 *外向* 转换：我们将一个实现类型的实例转换为其他类型。这将允许我们在拥有一个 TranslucentColor 实例时，调用一个期望 Color 类型值的方法，就像我们在调用 清单 7-34 中的 EqualViaColor 方法时一样。

```
public bool EqualViaColor(Color left, Color right)
    => left.Equals(right);
var red = new TranslucentColor(0xFF, 0, 0, 0);
var blue = new TranslucentColor(0, 0, 0xFF, 0);
**Assert.That(EqualViaColor(red, blue), Is.False);**
```

清单 7-34：隐式转换的实际应用

由于隐式转换运算符，当我们将 red 和 blue 变量作为参数传递给 EqualViaColor 方法时，它们会被转换为 Color 实例。转换是隐式发生的，因为转换运算符被定义为 implicit。

我们可以通过在 Color 类型上定义一个内部转换运算符，接收一个 TranslucentColor 参数，从而实现相同的效果。不同之处仅在于我们选择在哪里定义运算符。由于 TranslucentColor 已经依赖于 Color 类型，而 Color 对 TranslucentColor 没有任何了解，因此在 TranslucentColor 中定义的外部转换更为合理。

然而，我们必须小心所有的转换，特别是隐式转换。正如你在 第一章 中看到的，隐式转换可能会隐藏复杂性，甚至导致不希望发生的行为。用户定义的强制转换与从派生类型到基类型的隐式引用转换并不完全相同。

### 扩展转换与缩小转换

当 TranslucentColor 继承自 Color 时，我们可以将 TranslucentColor 引用传递给一个期望 Color 的方法，但它仍然是对同一 TranslucentColor 实例的引用，只会复制引用。在 列表 7-33 中，TranslucentColor 和 Color 是记录结构体，因此是值类型。当我们调用 TranslucentColor 的转换运算符时，我们只是创建了一个新的 Color 实例，因此复制操作丢失了某些特定于 TranslucentColor 的信息——特别是 Alpha 属性。

从派生类引用到基类引用的转换是 *拓宽* 转换。我们可以使用更通用的（基类）类型引用特定的实例，但不会丢失信息。我们仍然可以显式地将基类引用转换回原始的派生实例，尽管这是一个相对昂贵的运行时操作。从 TranslucentColor 结构体到 Color 的隐式转换通过我们自己的运算符方法是 *缩窄* 转换：两者类型并没有真正的上下级关系，它们是独立的值，但转换的过程会丢失信息。

虽然我们已复制了从派生类到基类的转换行为，但这并没有给我们带来相同的灵活性。转换后的值确实 *就是* 一个 Color，如果需要恢复其额外的属性，我们需要其他方式来捕获 TranslucentColor 的额外属性。

转换不适合用来复制继承的特性，但它们在其他场景下可能是有用的。

### 用于表示

当类型具有相同的含义但不同的表示形式时，进行无关类型之间的转换更有意义。例如，我们可能需要使用一个外部 API，该 API 使用常见的 int 表示颜色的十六进制 RGB 值。改变值的表示通常更适合通过显式转换而非隐式转换来实现，如 Listing 7-35 所示。然而，任何转换——无论是显式的还是隐式的——都需要仔细考虑其他的替代方法。

```
public readonly struct Color
{
    `--snip--`
    public static explicit operator int(Color color)
        => color.Red << 16 | color.Green << 8 | color.Blue;
}
```

Listing 7-35: 转换为不同的类型表示

在 Listing 7-36 中，我们通过将 plum 值转换为 int，来测试显式转换运算符的实现，以便将其作为参数传递给一个接受 int 类型参数的方法。

```
int Converted(int color)
{
    return color;
}
var plum = new Color(0xDD, 0xA0, 0xDD);
Assert.That(Converted(**(int)plum**), Is.EqualTo(0xDDA0DD));
```

Listing 7-36: 测试显式转换

这个本地的 Converted 函数接收一个 int 参数，并且为了测试的目的，它仅仅返回这个参数的值。由于转换运算符是显式的，我们在调用 Converted 方法时必须强制转换 Color 值；如果我们尝试直接将 Color 值作为参数传递给 Converted，编译器会报错。编译器还会捕捉到任何这种无意中的不合适表达：

```
var blue = new Color(0, 0, 0xFF);
var green = new Color(0, 0xFF, 0);
Assert.That(blue < green, Is.True);
```

如果我们将 Color 中的转换运算符改为隐式，这段代码会编译通过，但会比较两个 int 值，结果可能会是意外的。

在 清单 7-36 中对 int 的强制转换，虽然在代码中是显式且明显的，但并没有表达出转换背后的意图，这一点在某种程度上通过使用的方式得到了暗示。我们或许可以考虑用一个方法或属性来替代这种显式的外部转换，更加明确地描述转换的意图，可能将其命名为 ToWebColor。

命名转换使我们能够更好地表达我们想要的含义和原因，从而使代码更加自我文档化，同时又不会像显式强制转换那样过于侵入或冗长。使用命名属性而不是强制转换经常被忽视的一个后果是，属性名称更容易被搜索到，这样我们就可以方便地找到它在何处被使用。

### 用途

转换运算符，即使是隐式转换，也并不一定是一个坏选择。转换通常用于允许一个值由支持不同操作的无关类型表示，尽管这个值本身有一个共同的表示形式。例如，Color 是一个不可变的值类型，但我们可能希望逐步构建它的值。一个 Color 有多个属性，有时候单独设置它们比在构造函数中一次性设置所有属性更为方便。

为了不通过为 Color 的属性添加 set 访问器而破坏其不可变性，我们引入了一种新的伴随类型，它与 Color 非常相似，唯一不同的是它允许更改其属性。当值处于最终状态时，我们可以将可变类型的实例转化为不可变的 Color。关键在于我们可以轻松地从伴随类型转换到目标值类型。列表 7-37 展示了这种*可变伴随对象*类型，它允许隐式转换到不可变的目标值。

```
public class ColorBuilder
{
    public int Red {get; set;}
    public int Green {get; set;}
    public int Blue {get; set;}
    public static implicit operator Color(ColorBuilder color)
        => new Color(color.Red, color.Green, color.Blue);
}
```

列表 7-37：Color 的可变伴随对象

ColorBuilder 类型本身不是一个值类型；它唯一的目的是为 Color 值提供一种工厂机制。

*可变伴随对象* 模式的应用相当常见，我们在标准库中看到过 string 和 StringBuilder。string 类型是不可变的，当我们需要从多个部分构建一个 string 变量时，使用其可变伴随对象 StringBuilder 是高效的。当我们“构建”完字符串后，将其转化为*不可变*状态。

与 ColorBuilder 不同，我们必须调用 StringBuilder 的 ToString 方法将其转换为字符串，但这里可以很好地使用隐式转换。由于 ColorBuilder 可以隐式转换为 Color，我们可以用 ColorBuilder 的值调用一个接收 Color 参数的方法，如 列表 7-38 所示，我们用 ColorBuilder 和 Color 值都调用了 RelativeLuminance 方法。

```
public static double **RelativeLuminance(Color color)**
    => 0.2126 * color.Red + 0.7152 * color.Green + 0.0722 * color.Blue;
var background = new Color(0, 0, 0);
var builder = new ColorBuilder();
builder.Red = 0xFF;
builder.Green = 0xFF;
builder.Blue = 0;
if(**RelativeLuminance(builder)** **<** **RelativeLuminance(background)**)
    background = builder;
```

列表 7-38：转换伴随类型

我们在清单 7-37 中为ColorBuilder定义的隐式转换操作符允许我们将可变的builder变量作为参数传递给任何期望接受Color的函数。任何为Color实例编写的代码都不会期望使用伴生类的可变属性，因此这种转换是安全且方便的。

ColorBuilder可以通过隐式转换替代为Color。由于这两种类型共享相同的表示方式，因此没有信息丢失；然而，接口发生了缩小，因为Color目标类型没有属性的set访问器。

转换代表了一种多态性，因为我们显式地允许一个类型的变量被强制转换为另一种不相关类型的变量。像使用泛型的参数化多态性和通过重载实现的特定多态性一样，强制转换多态性是一种编译时活动，区别于通过继承实现的包含多态性，它是在运行时动态决定的。包含多态性是一个强大的工具，但由于类型关系是在运行时解决的，编译器无法识别可能发生的许多错误。当我们不正确地使用泛型、重载或强制转换时，可以依赖编译器来告诉我们代码中的大部分错误。

## 总结

> *试图聪明地对付编译器会削弱使用编译器的主要目的。*
> 
> —布赖恩·柯宁汉和 P.J. 普劳杰，*《编程风格的元素》*

问如何使值类型在多态使用时表现正确是错误的问题：多态本身有许多形式！将值类型与继承结合使用可能会导致难以诊断的错误，但继承只是多态性的一种形式。通过虚拟调度实现的包含多态性的动态特性带来了类型替代的期望，而这种特性与基于值的相等性不兼容。

从一种类型继承另一种类型会对派生类型施加责任，要求其遵守基类建立的契约。如果未能遵守该契约，可能会导致不良行为。只有当两种类型共享相同的行为契约时，一种类型才真正可以替代另一种类型，而这正是编译器无法强制执行的。是否使用继承是由我们程序员来判断的。对于结构体，甚至不允许继承，解放了我们对这种责任的承担。

对于记录类型，我们需要像对待类一样，关注基类的契约。尽管编译器精心设计了相等性的实现，确保Equals在记录类型中正确运行，但对于我们自己在这些类型中虚拟和重写的方法，它并没有做同样的处理。

记录类型并不一定适用于所有情况，正如前面提到的，使用继承让Equals对值类型“自动工作”是一个不完整的解决方案，解决了错误的问题。特别是，记录类型是引用类型，因此会受到垃圾回收的影响。Equals的实现都是虚拟的，EqualityContract属性也是虚拟的，它们都带有相关的开销。记录类型是一种非常简洁的声明不可变值类型的方式，但编程不仅仅是我们输入的字符数量。

值类型与其他多态行为表示方式结合得更好：类型转换、重载和泛型。这三种多态形式是静态的；也就是说，它们由编译器解决。虽然泛型类和方法中的类型参数是在运行时解析的，但我们仍然必须提供关于这些参数支持哪些操作的编译时保证。

使用继承来重用基类的代码可能很有诱惑力。但这是一个坏主意，因为继承一个类意味着基类可以被继承类替代，但很难确保基类的行为特征得到正确满足。我们仍然可以通过包含一个类型的实例并私下使用这个实例来实现我们的新类型，从而重用另一个类型的实现。

从具体类型——即非抽象类——继承通常会给我们带来遵循基类契约的挑战。当我们重写一个抽象方法或实现一个接口时，我们不会遇到这些问题，因为没有基类实现需要遵守。在这些情况下，我们仅继承接口契约，这要更容易遵守。

这个故事的道理是，如果我们总是实现真正的接口或继承完全抽象的类，那么本章中遇到的问题就永远不会给我们带来困难。相应地，我们应该封闭任何表示值类型的类或记录，确保它没有用户定义的基类型。我们仍然可以编写与我们使用和创建的值类型多态的代码，但应该通过使用泛型、重载方法和允许类型转换来以不同的方式表达。
