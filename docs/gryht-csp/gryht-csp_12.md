# 第十三章

13

反编译与逆向托管程序集

![](img/00010.jpg)

Mono 和 .NET 使用虚拟机，就像 Java 一样，用来运行已编译的可执行文件。 .NET 和 Mono 的可执行文件格式使用一种比原生 x86 或 x86_64 汇编语言更高层次的字节码，称为托管程序集。这与 C 和 C++ 等语言的原生非托管可执行文件不同。由于托管程序集是用更高层次的字节码编写的，所以如果使用一些不属于标准库的库，反编译它们是相对简单的。

在本章中，我们将编写一个简短的反编译器，它接受一个托管程序集并将源代码写回指定的文件夹。这个工具对恶意软件研究人员、逆向工程师，或者任何需要在两个 .NET 库或应用程序之间执行二进制差异比较（比较两个已编译的二进制文件或库在字节级别的差异）的人来说，都是非常有用的。接下来，我们将简要介绍一个随 Mono 提供的程序，叫做 monodis，它在分析程序集时非常有用，除了源代码分析，还可以用于发现潜在的后门和其他恶意代码。

反编译托管程序集

有许多易于使用的 .NET 反编译器。然而，它们的用户界面通常使用像 WPF（Windows Presentation Foundation）这样的工具包，这使得它们不能跨平台使用（通常只能在 Windows 上运行）。许多安全工程师、分析师和渗透测试人员使用 Linux 或 OS X 系统，因此这些工具对于他们来说并不十分实用。ILSpy 就是一个好的 Windows 反编译器示例；它使用跨平台的 ICSharpCode.Decompiler 和 Mono.Cecil 库进行反编译，但它的用户界面是 Windows 专用的，因此在 Linux 或 OS X 上无法使用。幸运的是，我们可以构建一个简单的工具，接受一个程序集作为参数，并使用这两个先前提到的开源库反编译给定的程序集，并将生成的源代码写回磁盘，以供后续分析。

这两个库都可以通过 NuGet 获取。安装方式取决于你的 IDE；如果你使用 Xamarin Studio 或 Visual Studio，你可以在解决方案资源管理器中为每个项目管理 NuGet 包。Listing 13-1 详细列出了整个类，以及反编译给定程序集所需的方法。

> class MainClass
> 
> {
> 
> public static void ➊Main(string[] args)
> 
> {
> 
> if (args.Length != 2)
> 
> {
> 
> Console.Error.WriteLine("Dirty C# decompiler requires two arguments.");
> 
> Console.Error.WriteLine("decompiler.exe <assembly> <path to directory>");
> 
> return;
> 
> }
> 
> IEnumerable<AssemblyClass> klasses = ➋GenerateAssemblyMethodSource(args[0]);
> 
> ➌foreach (AssemblyClass klass in klasses)
> 
> {
> 
> string outdir = Path.Combine(args[1], klass.namespase);
> 
> if (!Directory.Exists(outdir))
> 
> Directory.CreateDirectory(outdir);
> 
> string path = Path.Combine(outdir, klass.name + ".cs");
> 
> File.WriteAllText(path, klass.source);
> 
> }
> 
> }
> 
> private static IEnumerable<AssemblyClass> ➍GenerateAssemblyMethodSource(string assemblyPath)
> 
> {
> 
> AssemblyDefinition assemblyDefinition = AssemblyDefinition.➎ReadAssembly(assemblyPath,
> 
> new ReaderParameters(ReadingMode.Deferred) { ReadSymbols = true });
> 
> AstBuilder astBuilder = null;
> 
> foreach (var defmod in assemblyDefinition.Modules)
> 
> {
> 
> ➏foreach (var typeInAssembly in defmod.Types)
> 
> {
> 
> AssemblyClass klass = new AssemblyClass();
> 
> klass.name = typeInAssembly.Name;
> 
> klass.namespase = typeInAssembly.Namespace;
> 
> astBuilder = new AstBuilder(new DecompilerContext(assemblyDefinition.MainModule)
> 
> { CurrentType = typeInAssembly });
> 
> astBuilder.AddType(typeInAssembly);
> 
> using (StringWriter output = new StringWriter())
> 
> {
> 
> astBuilder.➐GenerateCode(new PlainTextOutput(output));
> 
> klass.➑source = output.ToString();
> 
> }
> 
> ➒yield return klass;
> 
> }
> 
> }
> 
> }
> 
> }
> 
> public class AssemblyClass
> 
> {
> 
> public string namespase;
> 
> public string name;
> 
> public string source;
> 
> }

Listing 13-1: 脏 C#反编译器

Listing 13-1 内容比较密集，所以让我们来梳理一下关键点。在`MainClass`中，我们首先创建了一个`Main()`方法 ➊，它将在我们运行程序时执行。方法开始时会检查指定了多少个参数。如果只指定了一个参数，它会打印使用说明并退出。如果指定了两个参数，我们假设第一个是我们要反编译的程序集的路径，第二个是生成的源代码应写入的文件夹。最后，我们使用`GenerateAssemblyMethodSource()`方法 ➋将第一个参数传递给应用程序，该方法实现就在`Main()`方法下方。

在`GenerateAssemblyMethodSource()`方法➍中，我们使用 Mono.Cecil 的方法`ReadAssembly()` ➎来返回一个 AssemblyDefinition。基本上，这是 Mono.Cecil 中的一个类，它完全表示一个程序集，并允许你以编程方式对其进行探查。一旦我们得到了要反编译的程序集的 AssemblyDefinition，我们就得到了生成与程序集中的原始字节码指令功能上等效的 C#源代码所需的所有信息。我们通过创建抽象语法树（AST）使用 Mono.Cecil 从 AssemblyDefinition 生成我们的 C#代码。我不会深入讲解 AST（这方面有大学课程专门讲解），但你应该知道，AST 可以表达程序中的每一个潜在代码路径，而且 Mono.Cecil 可以用来生成.NET 程序的 AST。

这个过程必须对程序集中的每个类重复。像这样的基础程序集通常只有一两个类，但复杂的应用程序可能会有几十个甚至更多的类。单独为每个类编码会很麻烦，所以我们创建了一个`foreach`循环 ➏来为我们完成这项工作。它对程序集中的每个类执行这些步骤，并根据当前类的信息创建一个新的`AssemblyClass`（它在`GenerateAssemblyMethodSource()`方法下定义）。

这里需要注意的部分是，GenerateCode()方法 ➐ 实际上通过获取我们创建的抽象语法树（AST），为我们提供了程序集类的 C#源代码表示。然后，我们将生成的 C#源代码以及类名和命名空间分配给 AssemblyClass 上的源代码字段 ➑。完成这些后，我们将类和它们的源代码列表返回给调用 GenerateAssemblyMethodSource()方法的地方——在这里是我们的 Main()方法。在我们遍历 GenerateAssemblyMethodSource()方法返回的每个类 ➌ 时，我们为每个类创建一个新文件，并将类的源代码写入该文件。我们在 GenerateAssemblyMethodSource()中使用 yield 关键字 ➒，在 foreach 循环 ➌ 中逐个返回每个类，而不是返回所有类的完整列表然后处理它们。这对于处理包含大量类的二进制文件来说是一个很好的性能提升。

测试反编译器

让我们通过编写一个类似 Hello World 的应用程序来测试这个。创建一个新项目，使用清单 13-2 中的简单类，然后编译它。

> 使用 System;
> 
> 命名空间 hello_world
> 
> {
> 
> class MainClass
> 
> {
> 
> public static void Main(string[] args)
> 
> {
> 
> Console.WriteLine("Hello World!");
> 
> Console.WriteLine(2 + 2);
> 
> }
> 
> }
> 
> }

清单 13-2：反编译前的简单 Hello World 应用

编译项目后，我们将新的反编译器指向它，看看它能生成什么，如清单 13-3 所示。

> $ ./decompiler.exe ~/projects/hello_world/bin/Debug/hello_world.exe hello_world
> 
> $ cat hello_world/hello_world/MainClass.cs
> 
> 使用 System;
> 
> 命名空间 hello_world
> 
> {
> 
> internal class MainClass
> 
> {
> 
> public static void Main(string[] args)
> 
> {
> 
> Console.WriteLine("Hello World!");
> 
> Console.WriteLine(➊4);
> 
> }
> 
> }
> 
> }

清单 13-3：反编译后的 Hello World 源代码

非常接近！唯一的实际区别是第二次调用 WriteLine()方法。在原始代码中，我们有 2 + 2，但反编译后的版本输出的是 4 ➊。这不是问题。在编译时，任何计算为常量的值都会被替换为该常量，因此 2 + 2 在汇编中会写成 4——在处理执行大量数学运算以达成特定结果的程序集时，需要注意这一点。

使用 monodis 分析程序集

假设我们想在反编译恶意二进制文件之前进行一些初步调查。Mono 附带的 monodis 工具为此提供了很多功能。它有特定的字符串类型选项（strings 是一个常见的 Unix 工具，能打印文件中找到的任何可读字符串），并且可以列出和导出编译到程序集中的资源，如配置文件或私钥。monodis 的使用输出可能会显得晦涩难懂，如清单 13-4 所示（不过 man 页面稍微好些）。

> $ monodis
> 
> monodis -- Mono 公共中间语言反汇编器
> 
> 使用方法是：monodis [--output=filename] [--filter=filename] [--help] [--mscorlib]
> 
> [--assembly] [--assemblyref] [--classlayout]
> 
> [--constant] [--customattr] [--declsec] [--event] [--exported]
> 
> [--fields] [--file] [--genericpar] [--interface] [--manifest]
> 
> [--marshal] [--memberref] [--method] [--methodimpl] [--methodsem]
> 
> [--methodspec] [--moduleref] [--module] [--mresources] [--presources]
> 
> [--nested] [--param] [--parconst] [--property] [--propertymap]
> 
> [--typedef] [--typeref] [--typespec] [--implmap] [--fieldrva]
> 
> [--standalonesig] [--methodptr] [--fieldptr] [--paramptr] [--eventptr]
> 
> [--propertyptr] [--blob] [--strings] [--userstrings] [--forward-decls] file ..

清单 13-4：monodis 使用输出

运行 monodis 不带任何参数将打印程序集的完整反汇编，这些反汇编属于通用中间语言（CIL）字节码，或者你也可以将反汇编输出到文件中。清单 13-5 显示了 ICSharpCode.Decompiler.dll 程序集的一些反汇编输出，这与您可能在本地编译应用程序中看到的 x86 汇编语言类似。

> $ monodis ICSharpCode.Decompiler.dll | tail -n30 | head -n10
> 
> IL_000c: mul
> 
> IL_000d: call class [mscorlib]System.Collections.Generic.EqualityComparer`1<!0> class
> 
> [mscorlib]System.Collections.Generic.EqualityComparer`1<!'<expr>j__TPar'>::get_Default()
> 
> IL_0012: ldarg.0
> 
> IL_0013: ldfld !0 class '<>f__AnonymousType5`2'<!0,!1>::'<expr>i__Field'
> 
> IL_0018: callvirt instance int32 class [mscorlib]System.Collections.Generic.Equality
> 
> Comparer`1<!'<expr>j__TPar'>::GetHashCode(!0)
> 
> IL_001d: add
> 
> IL_001e: stloc.0
> 
> IL_001f: ldc.i4 -1521134295
> 
> IL_0024: ldloc.0
> 
> IL_0025: mul $

清单 13-5：来自 ICSharpCode.Decompiler.dll 的一些 CIL 反汇编

这很不错，但如果你不知道自己在看什么，那就不太有用了。请注意，输出的代码看起来类似于 x86 汇编。这实际上是原始的中间语言（IL），有点像 JAR 文件中的 Java 字节码，看起来可能有点晦涩。你可能会发现，当比较两个版本的库，查看哪些内容发生了变化时，这个功能最有用。

它还有其他有助于逆向工程的强大功能。例如，你可以运行 GNU strings 工具在一个程序集上查看里面存储了哪些字符串，但你总是会得到一些你不想要的杂乱内容，比如随机的字节序列，恰好是 ASCII 可打印字符。另一方面，如果你将 --userstrings 参数传递给 monodis，它将打印任何为代码使用存储的字符串，如变量赋值或常量，正如清单 13-6 所示。由于 monodis 实际上会解析程序集以确定哪些字符串是通过编程定义的，它能生成更加干净的结果，信号与噪音的比率更高。

> $ monodis --userstrings ~/projects/hello_world/bin/Debug/hello_world.exe
> 
> 用户字符串堆内容
> 
> 00: ""
> 
> 01: "Hello World!"
> 
> 1b: ""
> 
> $

清单 13-6：使用 --userstrings 参数的 monodis

你还可以将 --userstrings 和 --strings 结合使用（用于元数据和其他内容），这将输出存储在程序集中的所有字符串，而不是 GNU strings 检测到的随机垃圾。这在你寻找加密密钥或硬编码在程序集中的凭证时非常有用。

然而，我最喜欢的 monodis 标志是 --manifest 和 --mresources。第一个，--manifest，会列出程序集中的所有嵌入式资源。这些通常是图片或配置文件，但有时你会找到私钥和其他敏感资料。第二个参数，--mresources，会将每个嵌入式资源保存到当前工作目录。列表 13-7 演示了这一点。

> $ monodis --manifest ~/projects/hello_world/bin/Debug/hello_world.exe
> 
> Manifestresource 表 (1..1)
> 
> 1: public 'hello_world.til_neo.png' 位于当前模块的偏移量 0 处
> 
> $ monodis --mresources ~/projects/hello_world/bin/Debug/hello_world.exe
> 
> $ file hello_world.til_neo.png
> 
> hello_world.til_neo.png：PNG 图像数据，1440 x 948，8 位/色 RGBA，非交错
> 
> $

列表 13-7：使用 monodis 将嵌入式资源保存到文件系统

显然，有人把一张 Neo 的图片隐藏在了我的 Hello World 应用程序中！可以肯定的是，monodis 是我在处理未知程序集时最喜欢的工具，当我想要获取更多关于它的信息时，比如方法或二进制文件中的特定字符串。

最后，我们有一个非常有用的 monodis 参数，--method，它列出了库或二进制文件中所有可用的方法和参数（参见 列表 13-8）。

> $ monodis --method ch1_hello_world.exe
> 
> 方法表 (1..2)
> 
> ########## ch1_hello_world.MainClass
> 
> 1: ➊实例默认 void '.ctor' () (参数: 1 实现标志: cil 管理)
> 
> 2: ➋默认的 void Main (string[] args) (参数: 1 实现标志: cil 管理) 列表 13-8：演示 monodis 的 --method 参数

当你在 第一章的 Hello World 程序上运行 monodis --method 时，你会注意到 monodis 打印了两行方法。第一行 ➊ 是包含 Main() 方法的 MainClass 类的构造函数，它位于第 2 行 ➋。所以，这个参数不仅列出了所有的方法（以及这些方法所在的类），还打印了类的构造函数！这能提供关于程序如何工作的深刻见解：方法名称通常是对内部工作原理的良好描述。

结论

在本章的第一部分，我们讨论了如何利用开源的 ICSharpCode.Decompiler 和 Mono.Cecil 库将任意程序集反编译回 C# 代码。通过编译一个简单的 Hello World 应用程序，我们看到了反编译后的程序集与原始源代码之间的一个区别。其他差异也可能出现，例如关键字 var 被替换为实际的对象类型。然而，生成的代码应该仍然在功能上等价，即使它不完全是与之前相同的源代码。

然后，我们使用 monodis 工具来查看如何解剖和分析程序集，从中提取更多的信息，以便比我们轻松获取到的方式更深入地了解一个恶意应用程序。希望这些工具能够缩短从“发生了什么？”到“我们该如何修复？”的时间，尤其是在出现问题或发现新的恶意软件时。
