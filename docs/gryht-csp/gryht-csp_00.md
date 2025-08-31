# 第一章

1

C#快速入门

![](img/00010.jpg)

与其他语言（如 Ruby、Python 和 Perl）不同，C#程序默认可以在所有现代 Windows 机器上运行。此外，在 Linux 系统（如 Ubuntu、Fedora 或其他发行版）上运行用 C#编写的程序也非常简单，特别是因为 Mono 可以通过大多数 Linux 包管理器（如 apt 或 yum）迅速安装。这使得 C#在满足跨平台需求方面比大多数语言更具优势，而且标准库简单而强大，随时可用。总的来说，C#和 Mono/.NET 库为任何想快速轻松编写跨平台工具的人提供了一个有力的框架。

选择 IDE

大多数想学习 C#的人会使用像 Visual Studio 这样的集成开发环境（IDE）来编写和编译他们的代码。由微软开发的 Visual Studio 是全球 C#开发的事实标准。微软提供了如 Visual Studio Community Edition 这样的免费版本，供个人使用，可以从微软官网 [`www.visualstudio.com/downloads/`](https://www.visualstudio.com/downloads/) 下载。

在本书的开发过程中，我根据自己是在 Ubuntu 还是 OS X 上，分别使用了 MonoDevelop 和 Xamarin Studio。在 Ubuntu 上，你可以通过 apt 包管理器轻松安装 MonoDevelop。MonoDevelop 由 Xamarin 公司维护，Xamarin 也是 Mono 的维护方。要安装它，可以使用以下命令：$ sudo apt-get install monodevelop。Xamarin Studio 是 OS X 版的 MonoDevelop IDE。Xamarin Studio 和 MonoDevelop 具有相同的功能，只是用户界面略有不同。你可以从 Xamarin 官网 [`www.xamarin.com/download-it/`](https://www.xamarin.com/download-it/) 下载 Xamarin Studio IDE 的安装程序。

这三种 IDE 中的任何一个都能满足本书的需求。事实上，如果你只想使用 vim，你甚至不需要 IDE！我们还会很快介绍如何使用 Mono 自带的命令行 C#编译器，而不是 IDE，来编译一个简单的示例。

一个简单的示例

对于任何使用过 C 或 Java 的人来说，C#的语法会显得非常熟悉。C#是一种强类型语言，像 C 和 Java 一样，这意味着你在代码中声明的变量只能是一个类型（例如整数、字符串或 Dog 类），并且无论如何，它始终是那个类型。让我们先快速看一下列表 1-1 中的 Hello World 示例，它展示了一些基本的 C#类型和语法。

> using ➊System;
> 
> namespace ➋ch1_hello_world
> 
> {
> 
> class ➌MainClass
> 
> {
> 
> public static void ➍Main(string[] ➎args)
> 
> {
> 
> ➏ string hello = "Hello World!";
> 
> ➐ DateTime now = DateTime.Now;
> 
> ➑ Console.Write(hello);
> 
> ➒ Console.WriteLine(" The date is " + now.ToLongDateString());
> 
> }
> 
> }
> 
> }

列表 1-1：一个基本的 Hello World 应用程序

一开始，我们需要导入将要使用的命名空间，我们通过使用 using 语句来实现这一点，导入 System 命名空间 ➊。这使得我们可以访问程序中的库，类似于 C 中的#include，Java 和 Python 中的 import，Ruby 和 Perl 中的 require。在声明了要使用的库之后，我们声明我们的类所在的命名空间 ➋。

与 C（和旧版本的 Perl）不同，C#是一种面向对象的语言，类似于 Ruby、Python 和 Java。这意味着我们可以构建复杂的类来表示数据结构，并为这些数据结构编写相应的方法，同时编写代码。命名空间让我们可以组织类和代码，避免潜在的命名冲突，比如当两个程序员创建了同名的两个类时。如果两个同名类位于不同的命名空间中，就不会有问题。每个类都必须有一个命名空间。

在处理完命名空间之后，我们可以声明一个类 ➌，该类将包含我们的 Main()方法 ➍。正如我们之前所说，类允许我们创建复杂的数据类型以及更适合现实世界对象的数据结构。在这个例子中，类的名称实际上并不重要；它只是我们 Main()方法的容器，Main()方法才是关键，因为它是当我们运行示例应用程序时会执行的部分。每个 C#应用程序都需要一个 Main()方法，就像 C 和 Java 一样。如果你的 C#应用程序接受命令行参数，你可以使用 args 变量 ➎ 来访问传递给应用程序的参数。

C#中存在简单的数据结构，如字符串 ➏，也可以创建更复杂的数据结构，如表示日期和时间的类 ➐。DateTime 类是处理日期的核心 C#类。在我们的示例中，我们使用它来存储当前日期和时间（DateTime.Now）到变量 now 中。最后，在声明了我们的变量之后，我们可以使用 Console 类的 Write() ➑和 WriteLine() ➒方法打印友好的信息（后者在末尾包含换行符）。

如果你使用的是 IDE，你可以通过点击运行按钮来编译并运行代码，该按钮位于 IDE 的左上角，看起来像一个播放按钮，或者按下 F5 键。不过，如果你希望通过命令行使用 Mono 编译器来编译源代码，你也可以轻松实现。在包含 C#类代码的目录中，使用 Mono 附带的 mcs 工具将你的类编译成可执行文件，如下所示：$ mcs Main.cs -out:ch1_hello_world.exe 从清单 1-1 中运行代码应该会打印出"Hello World!"字符串和当前日期在同一行，如清单 1-2 所示。在某些 Unix 系统中，你可能需要运行 mono ch1_hello_world.exe。

> $ ./ch1_hello_world.exe
> 
> 你好，世界！今天是 2017 年 6 月 28 日，星期三。

清单 1-2：运行 Hello World 应用程序

恭喜你完成了第一个 C#应用程序！

介绍类和接口

类和接口用于创建复杂的数据结构，这些结构仅凭内建结构难以表示。类和接口可以具有属性，属性是用于获取或设置类或接口值的变量；也可以具有方法，方法类似于函数，在类（或子类）或接口上执行，并且是唯一的。属性和方法用于表示对象的数据。例如，Firefighter 类可能需要一个 int 类型的属性来表示消防员的养老金，或者一个方法来指示消防员开车到发生火灾的地方。

类可以作为蓝图来创建其他类，这种技术叫做子类化。当一个类继承另一个类时，它会继承该类的属性和方法（称为父类）。接口也用作新类的蓝图，但与类不同，它们没有继承。因此，如果一个基类实现了一个接口，当它被子类化时，不会传递接口的属性和方法。

创建一个类

我们将创建一个简单的类，如列表 1-3 所示，作为一个例子，表示一个每天为让我们的生活变得更轻松、更美好而工作的公务员数据结构。

> public ➊abstract class PublicServant
> 
> {
> 
> public int ➋PensionAmount { get; set; }
> 
> public abstract void ➌DriveToPlaceOfInterest();
> 
> }

列表 1-3: 公务员抽象类

PublicServant 类是一种特殊类型的类。它是一个抽象类 ➊。通常，您可以像创建任何其他类型的变量一样创建一个类，它被称为实例或对象。然而，抽象类不能像其他类一样被实例化；它们只能通过子类化来继承。公共服务人员有很多类型——消防员和警察是我立刻想到的两个类型。因此，拥有一个基类供这两种公共服务人员继承是合理的。在这种情况下，如果这两个类是 PublicServant 的子类，它们将继承一个 PensionAmount 属性 ➋和一个 DriveToPlaceOfInterest 委托 ➌，这些必须由 PublicServant 的子类实现。没有一个通用的“公务员”职位可以申请，因此没有理由仅创建一个 PublicServant 实例。

创建接口

在 C#中，接口是类的补充。接口允许程序员强制一个类实现某些不被继承的属性或方法。让我们从一个简单的接口开始，如列表 1-4 所示。这个接口叫做 IPerson，将声明一些人们通常拥有的属性。

> public interface ➊IPerson
> 
> {
> 
> string ➋Name { get; set; }
> 
> int ➌Age { get; set; }
> 
> }

列表 1-4: IPerson 接口

注意

> C# 中的接口通常以 I 为前缀，以区分可能实现它们的类。这个 I 并不是强制要求的，但它是主流 C# 开发中非常常见的模式。

如果一个类要实现 IPerson 接口 ➊，该类需要自己实现 Name ➋ 和 Age ➌ 属性。否则，代码将无法编译。我将在接下来实现 Firefighter 类时准确展示这意味着什么，Firefighter 类实现了 IPerson 接口。目前，您只需知道接口是 C# 中一个重要且有用的功能。熟悉 Java 的程序员会觉得它们非常自然。C 程序员可以将其视为包含函数声明的头文件，期望 .c 文件来实现函数。熟悉 Perl、Ruby 或 Python 的人可能会觉得接口最初有些奇怪，因为这些语言没有类似的功能。

从抽象类继承并实现接口

让我们将 PublicServant 类和 IPerson 接口应用于实际场景，巩固我们所讨论的一些内容。我们可以创建一个类来表示我们的消防员，该类继承自 PublicServant 类并实现 IPerson 接口，如 示例 1-5 所示。

> public class ➊Firefighter : ➋PublicServant, ➌IPerson
> 
> {
> 
> public ➍Firefighter(string name, int age)
> 
> {
> 
> this.Name = name;
> 
> this.Age = age;
> 
> }
> 
> // 实现 IPerson 接口
> 
> public string ➎Name { get; set; }
> 
> public int ➏Age { get; set; }
> 
> public override void ➐DriveToPlaceOfInterest()
> 
> {
> 
> GetInFiretruck();
> 
> TurnOnSiren();
> 
> FollowDirections();
> 
> }
> 
> private void GetInFiretruck() {}
> 
> private void TurnOnSiren() {}
> 
> private void FollowDirections() {}
> 
> }

示例 1-5：Firefighter 类

Firefighter 类 ➊ 比我们之前实现的任何东西都要复杂一些。首先，注意 Firefighter 类继承自 PublicServant 类 ➋，并实现了 IPerson 接口 ➌。通过在 Firefighter 类名和冒号后列出类和接口，并用逗号分隔，我们实现了这一点。然后，我们创建了一个新的构造函数 ➍，它用于在创建新类实例时设置类的属性。这个新的构造函数将接受消防员的姓名和年龄作为参数，这些值将设置 IPerson 接口所需的 Name ➎ 和 Age ➏ 属性。接着，我们重写了从 PublicServant 类继承的 DriveToPlaceOfInterest() 方法 ➐，并定义了我们自己的一些空方法。我们需要实现 DriveToPlaceOfInterest() 方法，因为它在 PublicServant 类中被标记为抽象方法，抽象方法必须由子类进行重写。

注意

> 类具有默认构造函数，该构造函数没有参数用于创建实例。创建新的构造函数实际上是覆盖了默认构造函数。

PublicServant 类和 IPerson 接口非常灵活，可以用来创建功能完全不同的类。我们将再实现一个类，警察类，如 示例 1-6 所示，使用 PublicServant 和 IPerson。

> public class ➊警察 : PublicServant, IPerson
> 
> {
> 
> private bool _hasEmergency;
> 
> public PoliceOfficer(string name, int age)
> 
> {
> 
> this.Name = name;
> 
> this.Age = age;
> 
> _hasEmergency = ➋false;
> 
> }
> 
> //实现 IPerson 接口
> 
> public string Name { get; set; }
> 
> public int Age { get; set; }
> 
> public bool ➌HasEmergency
> 
> {
> 
> get { return _hasEmergency; }
> 
> set { _hasEmergency = value; }
> 
> }
> 
> public override void ➍DriveToPlaceOfInterest()
> 
> {
> 
> GetInPoliceCar();
> 
> if (this.➎HasEmergency)
> 
> TurnOnSiren();
> 
> FollowDirections();
> 
> }
> 
> private void GetInPoliceCar() {}
> 
> private void TurnOnSiren() {}
> 
> private void FollowDirections() {}
> 
> }

示例 1-6：警察类

警察类 ➊ 与消防员类相似，但有一些不同之处。最显著的区别是，在构造函数 ➋ 中设置了一个名为 HasEmergency ➌ 的新属性。我们还重写了 DriveToPlaceOfInterest() 方法 ➍，与之前的消防员类类似，但这次我们使用 HasEmergency 属性 ➎ 来判断警察是否应该开启警车警笛。我们可以使用相同的父类和接口组合来创建具有完全不同功能的类。

使用 Main() 方法将一切连接起来

我们可以使用新类来测试 C# 的一些新特性。让我们编写一个新的 Main() 方法来展示这些新类，参考 示例 1-7。

> using System;
> 
> namespace ch1_the_basics
> 
> {
> 
> public class MainClass
> 
> {
> 
> public static void Main(string[] args)
> 
> {
> 
> Firefighter firefighter = new ➊Firefighter("Joe Carrington", 35);
> 
> firefighter.➋PensionAmount = 5000;
> 
> PrintNameAndAge(firefighter);
> 
> PrintPensionAmount(firefighter);
> 
> firefighter.DriveToPlaceOfInterest();
> 
> PoliceOfficer officer = new PoliceOfficer("Jane Hope", 32);
> 
> officer.PensionAmount = 5500;
> 
> officer.➌HasEmergency = true;
> 
> ➍PrintNameAndAge(officer);
> 
> PrintPensionAmount(officer);
> 
> officer.➎DriveToPlaceOfInterest();
> 
> }
> 
> static void PrintNameAndAge(➏IPerson person)
> 
> {
> 
> Console.WriteLine("姓名: " + person.Name);
> 
> Console.WriteLine("年龄: " + person.Age);
> 
> }
> 
> static void PrintPensionAmount(➐PublicServant servant)
> 
> {
> 
> if (servant is ➑消防员)
> 
> Console.WriteLine("消防员养老金: " + servant.PensionAmount);
> 
> else if (servant is ➒警察)
> 
> Console.WriteLine("警察养老金: " + servant.PensionAmount);
> 
> }
> 
> }
> 
> }

示例 1-7：通过 Main() 方法将警察类和消防员类连接起来

要使用警察类和消防员类，我们必须使用我们在各自类中定义的构造函数来实例化它们。我们首先用消防员类 ➊，将姓名乔·卡灵顿和年龄 35 传递给类的构造函数，并将新类分配给消防员变量。我们还将消防员的 PensionAmount 属性 ➋ 设置为 5000。设置好消防员之后，我们将对象传递给 PrintNameAndAge() 和 PrintPension() 方法。

请注意，PrintNameAndAge() 方法接受 IPerson 接口 ➏ 作为参数，而不是 Firefighter、PoliceOfficer 或 PublicServant 类。当一个类实现了某个接口时，你可以创建接受该接口（在我们的例子中是 IPerson）作为参数的方法。如果将 IPerson 传递给方法，那么该方法只能访问接口要求的属性或方法，而不是整个类。在我们的例子中，只有 Name 和 Age 属性是可用的，这正是我们在该方法中所需要的。

类似地，PrintPensionAmount() 方法接受 PublicServant ➐ 作为参数，因此它只能访问 PublicServant 的属性和方法。我们可以使用 C# 的 is 关键字来检查一个对象是否属于某种类型的类，因此我们用它来检查我们的公务员是消防员 ➑ 还是警察 ➒，并根据实际情况打印相应的消息。

我们对警察类做了与对消防员类相同的处理，创建了一个名为简·霍普、年龄为 32 岁的新类；然后将她的养老金设置为 5500，HasEmergency 属性 ➌ 设置为 true。在打印姓名、年龄和养老金 ➍ 之后，我们调用该官员的 DriveToPlaceOfInterest() 方法 ➎。

运行 Main() 方法

运行该应用程序应展示类和方法如何相互作用，如列表 1-8 所示。

> $ ./ch1_the_basics.exe
> 
> 姓名：乔·卡灵顿
> 
> 年龄：35
> 
> 消防员养老金：5000
> 
> 姓名：简·霍普
> 
> 年龄：32
> 
> 官员养老金：5500

列表 1-8：运行基础程序的 Main() 方法

如你所见，公务员的姓名、年龄和养老金已经打印到屏幕上，完全符合预期！

匿名方法

到目前为止，我们使用的方法都是类方法，但我们也可以使用匿名方法。C#的这个强大功能允许我们通过委托动态地传递和分配方法。通过委托，会创建一个委托对象，它持有将被调用的方法的引用。我们在父类中创建这个委托，然后将委托的引用分配给父类子类中的匿名方法。这样，我们可以动态地将子类中的一段代码分配给委托，而不是覆盖父类的方法。为了演示如何使用委托和匿名方法，我们可以基于已经创建的类进行构建。

将委托分配给方法

让我们更新 PublicServant 类，以便使用 delegate 来替代方法 DriveToPlaceOfInterest()，如 Listing 1-9 所示。

> public abstract class PublicServant
> 
> {
> 
> public int PensionAmount { get; set; }
> 
> public delegate void ➊DriveToPlaceOfInterestDelegate();
> 
> public DriveToPlaceOfInterestDelegate ➋DriveToPlaceOfInterest { get; set; }
> 
> }

Listing 1-9: 带有 delegate 的 PublicServant 类

在之前的 PublicServant 类中，如果我们想要修改 DriveToPlaceOfInterest() 方法，需要覆盖它。而在新的 PublicServant 类中，DriveToPlaceOfInterest() 被一个 delegate ➊和一个属性 ➋所替代，允许我们调用和分配 DriveToPlaceOfInterest()。现在，任何继承自 PublicServant 类的类，都将拥有一个 delegate，可以用来设置自己的匿名方法来替代每个类中需要覆盖的该方法。因为它们继承自 PublicServant，所以我们需要相应地更新 Firefighter 和 PoliceOfficer 类的构造函数。

更新 Firefighter 类

我们首先更新 Firefighter 类，增加新的 delegate 属性。构造函数，如 Listing 1-10 所示，是我们所做的唯一修改。

> public ➊Firefighter(string name, int age)
> 
> {
> 
> this.➋Name = name;
> 
> this.➌Age = age;
> 
> this.DriveToPlaceOfInterest ➍+= delegate
> 
> {
> 
> Console.WriteLine("驾驶消防车");
> 
> GetInFiretruck();
> 
> TurnOnSiren();
> 
> FollowDirections();
> 
> };
> 
> }

Listing 1-10: 使用 delegate 来实现 DriveToPlaceOfInterest() 方法的 Firefighter 类

在新的 Firefighter 类构造函数 ➊ 中，我们像之前一样分配 Name ➋ 和 Age ➌。接下来，我们创建匿名方法并将其分配给 DriveToPlaceOfInterest delegate 属性，使用 += 操作符 ➍，这样调用 DriveToPlaceOfInterest() 时会调用该匿名方法。这个匿名方法打印 "驾驶消防车" 然后运行原类中的空方法。这样，我们可以在类中的每个方法中添加自定义代码，而不必覆盖它。

创建可选参数

PoliceOfficer 类需要类似的修改；我们更新构造函数，如 Listing 1-11 所示。因为我们已经在更新这个类，我们还可以将其修改为使用可选参数，即构造函数中的一个参数，在创建新实例时可以不包含它。我们将创建两个匿名方法，并使用可选参数来决定将哪个方法分配给 delegate。

> public ➊PoliceOfficer(string name, int age, bool ➋hasEmergency = false)
> 
> {
> 
> this.➌Name = name;
> 
> this.➍Age = age;
> 
> this.➎HasEmergency = hasEmergency;
> 
> if (this.➏HasEmergency)
> 
> {
> 
> this.DriveToPlaceOfInterest += delegate
> 
> {
> 
> Console.WriteLine("驾驶警车，开启警报");
> 
> GetInPoliceCar();
> 
> TurnOnSiren();
> 
> FollowDirections();
> 
> };
> 
> } else
> 
> {
> 
> this.DriveToPlaceOfInterest += delegate
> 
> {
> 
> Console.WriteLine("驾驶警车");
> 
> GetInPoliceCar();
> 
> FollowDirections();
> 
> };
> 
> }
> 
> }

列表 1-11：新的 PoliceOfficer 构造函数

在新的 PoliceOfficer 构造函数 ➊ 中，我们像之前一样设置了姓名 ➌ 和年龄 ➍ 属性。然而，这次我们还使用了一个可选的第三个参数 ➋ 来分配 HasEmergency 属性 ➎。第三个参数是可选的，因为它不需要被指定；当构造函数仅提供前两个参数时，它的默认值为 false。然后，我们根据 HasEmergency 是否为 true ➏，用一个新的匿名方法设置 DriveToPlaceOfInterest 委托属性。

更新 Main() 方法

使用新的构造函数，我们可以运行几乎与第一次相同的更新版 Main() 方法。详细内容见 列表 1-12。

> public static void Main(string[] args)
> 
> {
> 
> Firefighter firefighter = new Firefighter("Joe Carrington", 35);
> 
> firefighter.PensionAmount = 5000;
> 
> PrintNameAndAge(firefighter);
> 
> PrintPensionAmount(firefighter);
> 
> firefighter.DriveToPlaceOfInterest();
> 
> PoliceOfficer officer = new ➊PoliceOfficer("Jane Hope", 32);
> 
> officer.PensionAmount = 5500;
> 
> PrintNameAndAge(officer);
> 
> PrintPensionAmount(officer);
> 
> officer.DriveToPlaceOfInterest();
> 
> officer = new ➋PoliceOfficer("John Valor", 32, true);
> 
> PrintNameAndAge(officer);
> 
> officer.➌DriveToPlaceOfInterest();
> 
> }

列表 1-12：使用我们带有委托的类来驾驶到兴趣地点的更新 Main() 方法

唯一的不同在于最后三行，展示了如何创建一个新的有紧急情况的 PoliceOfficer ➋（构造函数的第三个参数为 true），与没有紧急情况的 Jane Hope ➊ 进行对比。然后我们调用 John Valor 警员的 DriveToPlaceOfInterest() 方法 ➌。

运行更新后的 Main() 方法

运行新方法显示如何创建两个 PoliceOfficer 类——一个有紧急情况，另一个没有——会打印出不同的内容，如 列表 1-13 所示。

> $ ./ch1_the_basics_advanced.exe
> 
> 姓名：Joe Carrington
> 
> 年龄：35
> 
> 消防员养老金：5000
> 
> 驾驶消防车
> 
> 姓名：Jane Hope
> 
> 年龄：32
> 
> 警员养老金：5500
> 
> ➊ 驾驶警车
> 
> 姓名：John Valor
> 
> 年龄：32
> 
> ➋ 带警报器驾驶警车 列表 1-13：使用委托的类运行新的 Main() 方法

如你所见，创建一个带有紧急情况的 PoliceOfficer 类会导致警员打开警报器驾驶 ➋。而 Jane Hope 则可以不开警报器驾驶 ➊，因为她没有紧急情况。

与本地库集成

最后，有时你需要使用仅在标准操作系统库中可用的库，例如 Linux 上的 libc 和 Windows 上的 user32.dll。如果你计划使用一个用 C、C++或其他编译为本地汇编的语言编写的库中的代码，C#使得与这些本地库的工作变得非常容易，我们将在第四章中介绍如何在制作跨平台的 Metasploit 有效载荷时使用此技术。这个特性被称为平台调用，简称 P/Invoke。程序员经常需要使用本地库，因为它们比.NET 或 Java 使用的虚拟机要快。像金融或科学专业人士这样的程序员，通常需要编写快速的代码来进行重数学运算，他们可能会用 C 语言编写需要快速执行的代码（例如，直接与硬件接口的代码），但使用 C#来处理那些对速度要求不高的代码。

Listing 1-14 显示了一个简单的应用程序，使用 P/Invoke 在 Linux 中调用标准 C 函数 printf()，或者在 Windows 上使用 user32.dll 弹出消息框。

> class MainClass
> 
> {
> 
> [➊DllImport("user32", CharSet=CharSet.Auto)]
> 
> static extern int MessageBox(IntPtr hWnd, String text, String caption, int options);
> 
> [DllImport("libc")]
> 
> static extern void printf(string message);
> 
> static void ➋Main(string[] args)
> 
> {
> 
> OperatingSystem os = Environment.OSVersion;
> 
> if (➌os.Platform == ➍PlatformID.Win32Windows||os.Platform == PlatformID.Win32NT)
> 
> {
> 
> ➎MessageBox(IntPtr.Zero, "Hello world!", "Hello world!", 0);
> 
> } else
> 
> {
> 
> ➏printf("Hello world!");
> 
> }
> 
> }
> 
> }

清单 1-14：通过一个简单的示例演示 P/Invoke

这个示例看起来比实际复杂。我们首先声明了两个函数，它们将在不同的库中被外部查找。我们使用 DllImport 特性➊来完成这一操作。特性允许你向方法（或类、类属性等）添加额外的信息，这些信息将在运行时被.NET 或 Mono 虚拟机使用。在我们的例子中，DllImport 特性告诉运行时查找我们声明的方法，这些方法位于另一个 DLL 中，而不是期望我们自己编写。

我们还声明了精确的函数名称以及函数所期望的参数。对于 Windows 系统，我们可以使用 MessageBox()函数，该函数需要一些参数，例如弹出框的标题和要显示的文本。对于 Linux 系统，printf()函数需要一个字符串来打印。这两个函数都会在运行时被查找，这意味着我们可以在任何系统上编译这个程序，因为外部库中的函数直到程序运行并被调用时才会被查找。这使我们能够在任何操作系统上编译应用程序，而不管该系统是否包含这两个库。

在声明了我们的本地函数后，我们可以编写一个简单的 Main() 方法 ➋，利用 if 语句和 os.Platform ➌ 来检查当前的操作系统。我们使用的 Platform 属性对应于 PlatformID 枚举 ➍，该枚举存储了程序可能运行的操作系统。通过使用 PlatformID 枚举，我们可以测试是否在 Windows 上运行，然后调用相应的方法：在 Windows 上调用 MessageBox() ➎，在 Unix 上调用 printf() ➏。编译后的这个应用程序可以在 Windows 机器或 Linux 机器上运行，无论是什么操作系统编译的。

结论

C# 语言拥有许多现代特性，使其成为处理复杂数据和应用程序的优秀语言。我们只触及了其中一些更强大的特性，如匿名方法和 P/Invoke。在接下来的章节中，你将深入了解类和接口的概念，以及许多其他高级特性。此外，你还将学习更多核心类的使用，如 HTTP 和 TCP 客户端等。

在本书中，我们将开发自己的自定义安全工具，同时你还将学习到通用的编程模式，这些是创建类的有用规范，能够使基于它们的开发变得更快捷和简单。编程模式的良好示例可以在 第五章 和 第十一章 中看到，在这些章节中，我们与第三方工具如 Nessus 和 Metasploit 的 API 和 RPC 进行了接口交互。

到本书结束时，我们将涵盖如何使用 C# 来完成每位安全从业人员的工作——从安全分析师到工程师，甚至是家庭中的业余研究人员。C# 是一门美丽且强大的语言，借助 Mono 的跨平台支持，将 C# 带到手机和嵌入式设备，它与 Java 及其他替代语言同样强大且易于使用。
