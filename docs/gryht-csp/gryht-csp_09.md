# 第十章

10

自动化 ClamAV

![](img/00010.jpg)

ClamAV 是一个开源的杀毒解决方案，主要用于在邮件服务器上扫描电子邮件及其附件，识别潜在的病毒，以便它们在到达并感染网络中的计算机之前被发现。但这并不是它唯一的使用场景。在本章中，我们将使用 ClamAV 创建一个自动化病毒扫描器，用于扫描文件中的恶意软件，并借助 ClamAV 的数据库识别病毒。

你将学习如何以几种方式自动化 ClamAV。其中一种方法是与 libclamav 接口，这个本地库驱动着 ClamAV 的命令行工具，如 clamscan，一个你可能熟悉的文件扫描器。第二种方法是通过套接字与 clamd 守护进程接口，以便在没有安装 ClamAV 的计算机上执行扫描。

安装 ClamAV

ClamAV 是用 C 语言编写的，这在与 C#自动化时会带来一些复杂性。它可以通过常见的包管理器（如 yum 和 apt）在 Linux 上使用，同时也适用于 Windows 和 OS X。许多现代 Unix 发行版都包含 ClamAV 包，但该版本可能与 Mono 和.NET 不兼容。

在 Linux 系统上安装 ClamAV 的过程应该是这样的：$ sudo apt-get install clamav 如果你使用的是基于 Red Hat 或 Fedora 的 Linux 版本，并且系统自带 yum，你可以运行如下命令：$ sudo yum install clamav clamav-scanner clamav-update 如果你需要启用额外的仓库才能通过 yum 安装 ClamAV，可以输入以下命令：$ sudo yum install -y epel-release 这些命令会安装与系统架构相匹配的 ClamAV 版本。

注意

> Mono 和.NET 不能与本地非托管库接口，除非它们的架构兼容。例如，32 位的 Mono 和.NET 在编译为 64 位 Linux 或 Windows 机器的 ClamAV 上无法正常运行。你需要安装或编译与 Mono 或.NET 32 位架构匹配的本地 ClamAV 库。

包管理器中的默认 ClamAV 包可能不适合 Mono/.NET 的架构。如果不匹配，你需要专门安装与 Mono/.NET 架构匹配的 ClamAV。你可以编写程序通过检查 IntPtr.Size 的值来验证你的 Mono/.NET 版本。输出为 4 表示 32 位版本，而输出为 8 表示 64 位版本。如果你在 Linux、OS X 或 Windows 上运行 Mono 或 Xamarin，你可以轻松检查，如列表 10-1 所示。

> $ echo "IntPtr.Size" | csharp
> 
> 4

列表 10-1：检查 Mono/.NET 架构的单行命令

Mono 和 Xamarin 提供了一个用于 C# 的交互式解释器（称为 csharp），类似于 Python 解释器或 Ruby 的 irb。通过使用标准输入（stdin）将 IntPtr.Size 字符串传递到解释器中，你可以打印出 Size 属性的值，在本例中是 4，表示 32 位架构。如果你的输出也是 4，那么你需要安装 32 位的 ClamAV。设置一个你预期架构的虚拟机可能是最简单的方式。由于在 Linux、OS X 和 Windows 上编译 ClamAV 的指令不同，如果你需要安装 32 位的 ClamAV，它超出了本书的讨论范围。不过，网上有很多教程可以指导你根据自己的操作系统完成安装步骤。

你还可以使用 Unix 的 file 工具来检查你的 ClamAV 库是 32 位版本还是 64 位版本，如列表 10-2 所示。

> $ file /usr/lib/x86_64-linux-gnu/libclamav.so.7.1.1
> 
> libclamav.so.7.1.1: ELF ➊64 位 LSB 共享对象，x86-64，版本 1（GNU/Linux），
> 
> 动态链接，未剥离 列表 10-2：使用 file 查看 libclamav 架构

使用 file 命令，我们可以查看 libclamav 库是为 32 位还是 64 位架构编译的。我的计算机上，列表 10-2 显示该库是 64 位版本 ➊。但在列表 10-1 中，IntPtr.Size 返回的是 4，而不是 8！这意味着我的 libclamav（64 位）和 Mono（32 位）架构不匹配。我必须重新编译 ClamAV 为 32 位版本，才能与我的 Mono 安装一起使用，或者安装 64 位的 Mono 运行时。

ClamAV 本地库与 clamd 网络守护进程

我们将从使用本地库 libclamav 自动化 ClamAV 开始。这允许我们使用本地副本的 ClamAV 及其病毒库进行病毒扫描；然而，这要求 ClamAV 软件和病毒库必须正确安装并保持更新。引擎可能会占用大量内存和 CPU，使用磁盘空间存储病毒签名。有时这些需求会占用机器比程序员希望的更多资源，因此将扫描任务卸载到另一台机器上是合理的选择。

你可能更希望在一个中心位置执行病毒扫描——例如，当电子邮件服务器发送或接收邮件时——在这种情况下，你可能无法轻松使用 libclamav。相反，你可以使用 clamd 守护进程，将病毒扫描从邮件服务器卸载到专用的病毒扫描服务器。你只需保持一个服务器的病毒签名是最新的，而且你也不会大幅增加让邮件服务器崩溃的风险。

使用 ClamAV 的本地库进行自动化

一旦你正确安装并运行了 ClamAV，你就可以开始自动化它了。首先，我们将直接使用 libclamav 通过 P/Invoke（在 第一章 中介绍）自动化 ClamAV，P/Invoke 允许托管程序集调用本机、非托管库中的函数。尽管你需要实现一些支持类，但总体而言，将 ClamAV 集成到应用程序中是相对直接的。

设置支持的枚举和类

我们将在代码中使用一些辅助类和枚举。所有辅助类都非常简单——大多数只有不到 10 行代码。然而，它们构成了将方法和类连接在一起的“胶水”。

支持的枚举

ClamDatabaseOptions 枚举，如 列表 10-3 所示，用于在 ClamAV 引擎中设置我们将使用的病毒查找数据库的选项。

> [Flags]
> 
> public enum ClamDatabaseOptions
> 
> {
> 
> CL_DB_PHISHING = 0x2，
> 
> CL_DB_PHISHING_URLS = 0x8，
> 
> CL_DB_BYTECODE = 0x2000，
> 
> ➊CL_DB_STDOPT = (CL_DB_PHISHING | CL_DB_PHISHING_URLS | CL_DB_BYTECODE)，
> 
> }

列表 10-3：定义 ClamAV 数据库选项的 ClamDatabaseOptions 枚举

ClamDatabaseOptions 枚举使用直接从 ClamAV C 源代码中获取的值来定义数据库选项。这三个选项启用钓鱼电子邮件的签名、钓鱼网址的签名，以及在启发式扫描中使用的动态字节码签名。综合这三者，构成了 ClamAV 的标准数据库选项，用于扫描病毒或恶意软件。通过使用按位 OR 操作符将这三个选项值组合起来，我们得到了一个我们想要使用的组合选项的位掩码，定义在枚举 ➊ 中。使用位掩码是一种非常高效的存储标志或选项的流行方式。

我们必须实现的另一个枚举是 ClamReturnCode 枚举，它对应于 ClamAV 的已知返回代码，如 列表 10-4 所示。再次说明，这些值是直接从 ClamAV 源代码中获取的。

> public enum ClamReturnCode
> 
> {
> 
> ➊CL_CLEAN = 0x0，
> 
> ➋CL_SUCCESS = 0x0，
> 
> ➌CL_VIRUS = 0x1
> 
> }

列表 10-4：存储我们感兴趣的 ClamAV 返回代码的枚举

这绝不是一个完整的返回代码列表。我只包括了在我们编写的示例中预期会看到的返回代码。这些是清洁代码 ➊ 和成功代码 ➋，分别表示扫描的文件没有病毒或某个操作成功，病毒代码 ➌ 则表示在扫描文件中检测到病毒。如果你遇到 ClamReturnCode 枚举中未定义的错误代码，可以在 ClamAV 源代码的 clamav.h 中查找它们。这些代码在头文件中的 cl_error_t 结构中定义。

我们的 ClamReturnCode 枚举有三个值，其中只有两个是不同的。CL_CLEAN 和 CL_SUCCESS 都共享相同的值 0x0，因为 0x0 既表示一切按预期运行，也表示扫描的文件是干净的。另一个值 0x1 则表示检测到病毒。

我们需要定义的最后一个枚举是 ClamScanOptions 枚举，这是我们需要的最复杂的枚举。它在清单 10-5 中显示。

> [Flags]
> 
> public enum ClamScanOptions
> 
> {
> 
> CL_SCAN_ARCHIVE = 0x1,
> 
> CL_SCAN_MAIL = 0x2,
> 
> CL_SCAN_OLE2 = 0x4,
> 
> CL_SCAN_HTML = 0x10,
> 
> ➊CL_SCAN_PE = 0x20,
> 
> CL_SCAN_ALGORITHMIC = 0x200,
> 
> ➋CL_SCAN_ELF = 0x2000,
> 
> CL_SCAN_PDF = 0x4000,
> 
> ➌CL_SCAN_STDOPT = (CL_SCAN_ARCHIVE | CL_SCAN_MAIL |
> 
> CL_SCAN_OLE2 | CL_SCAN_PDF | CL_SCAN_HTML | CL_SCAN_PE |
> 
> CL_SCAN_ALGORITHMIC | CL_SCAN_ELF)
> 
> }

清单 10-5: 用于保存 ClamAV 扫描选项的类

如你所见，ClamScanOptions 看起来是 ClamDatabaseOptions 的复杂版本。它定义了可以扫描的各种文件类型（Windows PE 可执行文件 ➊、Unix ELF 可执行文件 ➋、PDF 等），以及一组标准选项 ➌。与之前的枚举一样，这些枚举值直接取自 ClamAV 源代码。

ClamResult 支持类

现在我们只需要实现 ClamResult 类（见清单 10-6），以完成 libclamav 的支持功能。

> public class ClamResult
> 
> {
> 
> public ➊ClamReturnCode ReturnCode { get; set; }
> 
> public string VirusName { get; set; }
> 
> public string FullPath { get; set; }
> 
> }

清单 10-6: 用于保存 ClamAV 扫描结果的类

这个非常简单！第一个属性是 ClamReturnCode ➊，用于存储扫描的返回代码（通常应该是 CL_VIRUS）。我们还拥有两个字符串属性：一个用于存储 ClamAV 返回的病毒名称，另一个用于存储文件路径，以便后续使用。我们将使用这个类来保存每个文件扫描的结果作为一个对象。

访问 ClamAV 的本地库函数

为了保持我们从 libclamav 调用的本地函数与其余 C# 代码和类之间的分离，我们定义了一个类来封装所有我们将使用的 ClamAV 函数（见清单 10-7）。

> static class ClamBindings
> 
> {
> 
> const string ➊_clamLibPath = "/Users/bperry/clamav/libclamav/.libs/libclamav.7.dylib";
> 
> [➋DllImport(_clamLibPath)]
> 
> public extern static ➌ClamReturnCode cl_init(uint options);
> 
> [DllImport(_clamLibPath)]
> 
> public extern static IntPtr cl_engine_new();
> 
> [DllImport(_clamLibPath)]
> 
> public extern static ClamReturnCode cl_engine_free(IntPtr engine);
> 
> [DllImport(_clamLibPath)]
> 
> public extern static IntPtr cl_retdbdir();
> 
> [DllImport(_clamLibPath)]
> 
> public extern static ClamReturnCode cl_load(string path, IntPtr engine,
> 
> ref uint signo, uint options);
> 
> [DllImport(_clamLibPath)]
> 
> public extern static ClamReturnCode cl_scanfile(string path, ref IntPtr virusName,
> 
> ref ulong scanned, IntPtr engine, uint options);
> 
> [DllImport(_clamLibPath)]
> 
> public extern static ClamReturnCode cl_engine_compile(IntPtr engine);
> 
> }

列表 10-7: ClamBindings 类，包含所有 ClamAV 函数

ClamBindings 类首先定义了一个字符串，表示我们将要接口的 ClamAV 库的完整路径 ➊。在这个例子中，我指向的是我从源代码编译的一个 OS X .dylib 文件，以匹配我的 Mono 安装的架构。根据你编译或安装 ClamAV 的方式，原生 ClamAV 库的路径在你的系统上可能有所不同。如果你使用 ClamAV 安装程序，在 Windows 上，这个文件将是一个位于 /Program Files 目录中的 .dll 文件。在 OS X 上，它将是一个 .dylib 文件，在 Linux 上则是 .so 文件。在后两种系统上，你可以使用 find 命令定位正确的库。

在 Linux 上，类似以下命令会打印出任何 libclamav 库的路径：$ find / -name libclamav*so$

在 OS X 上，使用此命令：$ find / -name libclamav*dylib$

DllImport 属性 ➋ 告诉 Mono/.NET 运行时在我们在参数中指定的库中查找给定的函数。这样，我们就能直接在程序中调用 ClamAV 函数。接下来，我们将介绍在实现 ClamEngine 类时，列表 10-7 中显示的函数的功能。你还可以看到我们已经在使用 ClamReturnCode 类 ➌，它是在调用某些 ClamAV 本地函数时返回的。

编译 ClamAV 引擎

列表 10-8 中的 ClamEngine 类将执行大部分实际的扫描和潜在恶意文件报告工作。

> public class ClamEngine : IDisposable
> 
> {
> 
> private ➊IntPtr engine;
> 
> public ➋ClamEngine()
> 
> {
> 
> ClamReturnCode ret = ClamBindings.➌cl_init((uint)ClamDatabaseOptions.CL_DB_STDOPT);
> 
> if (ret != ClamReturnCode.CL_SUCCESS)
> 
> throw new Exception("预期返回 CL_SUCCESS，但得到 " + ret);
> 
> engine = ClamBindings.➍cl_engine_new();
> 
> try
> 
> {
> 
> string ➎dbDir = Marshal.PtrToStringAnsi(ClamBindings.cl_retdbdir());
> 
> uint ➏signatureCount = 0;
> 
> ret = ClamBindings.➐cl_load(dbDir, engine, ref signatureCount,
> 
> (uint)ClamScanOptions.CL_SCAN_STDOPT);
> 
> if (ret != ClamReturnCode.CL_SUCCESS)
> 
> throw new Exception("预期返回 CL_SUCCESS，但得到 " + ret);
> 
> ret = (ClamReturnCode)ClamBindings.➑cl_engine_compile(engine);
> 
> if (ret != ClamReturnCode.CL_SUCCESS)
> 
> throw new Exception("预期返回 CL_SUCCESS，但得到 " + ret);
> 
> }
> 
> catch
> 
> {
> 
> ret = ClamBindings.cl_engine_free(engine);
> 
> if (ret != ClamReturnCode.CL_SUCCESS)
> 
> Console.Error.WriteLine("释放分配的引擎失败");
> 
> throw;
> 
> }
> 
> }

列表 10-8: ClamEngine 类，用于扫描和报告文件

首先，我们声明一个类级别的 IntPtr 变量 ➊，名为 engine，它将指向我们的 ClamAV 引擎，供类中的其他方法使用。虽然 C# 不需要指针来引用对象在内存中的确切地址，但 C 需要。C 有指针，类型为 intptr_t，而 IntPtr 是 C# 版的 C 指针。由于 ClamAV 引擎将在 .NET 和 C 之间来回传递，我们需要一个指针来引用它在内存中存储的地址，以便将其传递给 C。这就是创建 engine 变量时发生的事情，我们将在构造函数中为其赋值。

接下来，我们定义构造函数。ClamEngine 类的构造函数 ➋ 不需要任何参数。为了初始化 ClamAV 开始分配用于扫描的引擎，我们通过传递加载签名时要使用的签名数据库选项来调用 ClamBindings 类中的 cl_init() ➌。为了防止 ClamAV 初始化失败，我们检查 cl_init() 的返回代码，如果初始化失败，则抛出异常。如果 ClamAV 初始化成功，我们使用 cl_engine_new() ➍ 分配一个新引擎，该方法不接受任何参数，并返回指向新 ClamAV 引擎的指针，我们将其存储在 engine 变量中以供后续使用。

一旦分配了引擎，我们需要加载病毒签名以供扫描。cl_retdbdir() 函数返回 ClamAV 配置使用的定义数据库路径，并将其存储在 dbDir 变量中 ➎。由于 cl_retdbdir() 返回的是 C 指针字符串，我们通过使用 Marshal 类中的 PtrToStringAnsi() 函数将其转换为常规字符串，Marshal 类用于在托管类型和非托管类型之间转换数据类型。一旦存储了数据库路径，我们定义一个整数变量 signatureCount ➏，该变量被传递给 cl_load() 并赋值为从数据库中加载的签名数量。

我们使用 ClamBindings 类中的 cl_load() ➐ 方法将签名数据库加载到引擎中。我们将 ClamAV 数据库目录 dbDir 和新引擎作为参数传递，还传递一些其他值。传递给 cl_load() 的最后一个参数是一个枚举值，用于指定我们希望支持扫描的文件类型（例如 HTML、PDF 或其他特定类型的文件）。我们使用之前创建的类 ClamScanOptions 来定义扫描选项，设置为 CL_SCAN_STDOPT，这样我们就使用标准的扫描选项。在加载完病毒数据库后（根据选项，可能需要几秒钟），我们再次检查返回代码是否等于 CL_SUCCESS；如果是，我们最终通过将其传递给 cl_engine_compile() 函数 ➑ 来编译引擎，准备引擎开始扫描文件。然后，我们最后一次检查是否收到了 CL_SUCCESS 返回代码。

扫描文件

为了简化文件扫描，我们将 cl_scanfile()（ClamAV 库中的扫描文件并返回结果的函数）封装为我们自己的方法，命名为 ScanFile()。这样我们就可以准备传递给 cl_scanfile()的参数，并能够处理并返回来自 ClamAV 的结果，作为一个 ClamResult 对象返回。此过程在 Listing 10-9 中展示。

> public ClamResult ScanFile(string filepath, uint options = (uint)ClamScanOptions.➊CL_SCAN_STDOPT)
> 
> {
> 
> ➋ulong scanned = 0;
> 
> ➌IntPtr vname = (IntPtr)null;
> 
> ClamReturnCode ret = ClamBindings.➍cl_scanfile(filepath, ref vname, ref scanned,
> 
> engine, options);
> 
> if (ret == ClamReturnCode.CL_VIRUS)
> 
> {
> 
> string virus = Marshal.➎PtrToStringAnsi(vname);
> 
> ➏ClamResult result = new ClamResult();
> 
> result.ReturnCode = ret;
> 
> result.VirusName = virus;
> 
> result.FullPath = filepath;
> 
> return result;
> 
> }
> 
> else if (ret == ClamReturnCode.CL_CLEAN)
> 
> return new ClamResult() { ReturnCode = ret, FullPath = filepath };
> 
> else
> 
> throw new Exception("Expected either CL_CLEAN or CL_VIRUS, got: " + ret);
> 
> }

Listing 10-9: ScanFile()方法，它扫描并返回一个 ClamResult 对象

我们实现的 ScanFile()方法接受两个参数，但我们只需要第一个参数，即要扫描的文件路径。用户可以通过第二个参数定义扫描选项，但如果未指定第二个参数，则会使用我们在 ClamScanOptions 中定义的标准扫描选项➊来扫描文件。

我们通过定义一些变量来开始 ScanFile()方法的实现。扫描用的 ulong 类型变量最初设置为 0➋。在扫描完文件后，我们实际上不会再使用这个变量，但 cl_scanfile()函数需要这个变量才能正确调用。我们定义的下一个变量是另一个 IntPtr，我们称之为 vname（病毒名称）➌。最初将其设置为 null，但稍后我们会为它分配一个 C 字符串指针，当检测到病毒时，该指针指向 ClamAV 数据库中的病毒名称。

我们使用 ClamBindings 中定义的 cl_scanfile()函数➍来扫描文件，并传递给它一些参数。第一个参数是我们要扫描的文件路径，接着是一个变量，如果检测到病毒，这个变量将被赋值为病毒名称。最后两个参数分别是我们将用于扫描的引擎和扫描选项。中间的参数 scanned 是调用 cl_scanfile()所必需的，但对我们来说并没有实际用途。我们在将它作为参数传递给该函数后就不会再使用它。

该方法的其余部分将扫描信息包装成便于程序员使用的形式。如果 cl_scanfile() 的返回代码表明发现了病毒，我们使用 PtrToStringAnsi() ➎ 返回 vname 变量在内存中指向的字符串。一旦得到病毒名称，我们创建一个新的 ClamResult 类 ➏ 并使用 cl_scanfile() 返回代码、病毒名称和扫描文件的路径为其赋值三个属性。然后，我们将 ClamResult 类返回给调用者。如果返回代码是 CL_CLEAN，我们将返回一个带有 CL_CLEAN 返回代码的新 ClamResult 类。然而，如果它既不是 CL_CLEAN 也不是 CL_VIRUS，我们会抛出异常，因为我们得到了一个我们没有预料的返回代码。

清理工作

ClamEngine 类中最后需要实现的方法是 Dispose()，如列表 10-10 所示，它在使用语句的上下文中自动清理扫描后的工作，并且是 IDisposable 接口所要求的。

> public void Dispose()
> 
> {
> 
> ClamReturnCode ret = ClamBindings.➊cl_engine_free(engine);
> 
> if (ret != ClamReturnCode.CL_SUCCESS)
> 
> Console.Error.WriteLine("释放分配的引擎失败");
> 
> }
> 
> }

列表 10-10：Dispose() 方法，自动清理引擎

我们实现 Dispose() 方法是因为，如果在使用完 ClamAV 引擎后不释放它，可能会导致内存泄漏。使用像 C# 这样的语言与 C 库进行工作有一个缺点，因为 C# 有垃圾回收机制，很多程序员不会主动考虑清理资源。然而，C 语言没有垃圾回收机制。如果我们在 C 中分配了某些内容，使用完后就需要手动释放。这就是 cl_engine_free() 函数 ➊ 的作用。为了确保我们做到谨慎，我们还会检查引擎是否已成功释放，通过将返回代码与 CL_SUCCESS 进行比较。如果它们相同，一切正常。否则，我们会抛出异常，因为我们应该能够释放已分配的引擎，如果做不到，这可能表明代码中存在问题。

通过扫描 EICAR 文件来测试程序

现在我们可以将所有内容整合起来，扫描一些文件来测试我们的绑定。EICAR 文件是一个行业公认的文本文件，用于测试防病毒产品。它无害，但任何正常工作的防病毒产品应该将其识别为病毒，因此我们将用它来测试我们的程序。在列表 10-11 中，我们使用 Unix 的 cat 命令打印用于专门测试防病毒的测试文件的内容——EICAR 文件。

> $ cat ~/eicar.com.txt
> 
> X5O!P%@AP4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*

列表 10-11：打印 EICAR 防病毒测试文件的内容

[列表 10-12 中的简短程序将扫描作为参数指定的任何文件并打印结果。

> public static void Main(string[] args)
> 
> {
> 
> using (➊ClamEngine e = new ClamEngine())
> 
> {
> 
> foreach (string file in args)
> 
> {
> 
> ClamResult result = e.➋ScanFile(file); //非常简单！
> 
> 如果 (result != null && result.ReturnCode == ClamReturnCode.➌CL_VIRUS)
> 
> Console.WriteLine("Found: " + result.VirusName);
> 
> else
> 
> Console.WriteLine("文件干净！");
> 
> }
> 
> } // 引擎在这里被释放，分配的引擎会自动被清理
> 
> }

列表 10-12：自动化 ClamAV 的程序的 Main() 方法

我们首先创建我们的 ClamEngine 类 ➊，并在 using 语句中使用它，这样在完成时可以自动清理引擎。接着，我们遍历传递给 Main() 的每个参数，并假设它是一个文件路径，我们可以用 ClamAV 扫描它。我们将每个文件路径传递给 ScanFile() 方法 ➋，然后检查 ScanFile() 返回的结果，看看 ClamAV 是否返回了 CL_VIRUS 返回代码 ➌。如果是，我们将病毒名称打印到屏幕上，如 列表 10-13 所示。否则，我们打印文本“文件干净！”

> $ mono ./ch10_automating_clamav_fs.exe ~/eicar.com.txt
> 
> ➊ 找到：Eicar 测试签名 列表 10-13：运行我们的 ClamAV 程序在 EICAR 文件上时，会识别到病毒。

如果程序打印出 Found: Eicar-Test-Signature ➊，那就表示它工作正常！这意味着 ClamAV 扫描了 EICAR 文件，并将其与数据库中的 EICAR 定义进行了匹配，然后返回了病毒名称。一个扩展该程序的好方法是使用 FileWatcher 类，允许你定义要监视的目录，并在文件发生更改或创建时自动扫描这些文件。

现在我们有了一个可以使用 ClamAV 扫描文件的工作程序。然而，可能有一些情况，由于许可问题（ClamAV 采用 GNU 公共许可证）或技术原因，你不能有效地将 ClamAV 与应用程序一起打包，但你仍然需要一种方法来扫描网络中的文件是否有病毒。我们将介绍另一种自动化 ClamAV 的方法，这将以更集中的方式解决这个问题。

使用 clamd 自动化

clamd 守护进程为需要接受用户上传文件或类似功能的应用程序提供了一个很好的病毒扫描方法。它通过 TCP 操作，但默认情况下不使用 SSL！它还非常轻量，但必须在网络中的服务器上运行，这会带来一些限制。clamd 服务允许你使用长时间运行的进程来扫描文件，而不是像之前的自动化方式那样管理和分配 ClamAV 引擎。由于它是 ClamAV 的服务器版本，你可以使用 clamd 在不安装应用程序的情况下扫描计算机上的文件。当你只想集中管理病毒定义，或者当你有资源限制并希望将病毒扫描任务卸载到其他机器时，这非常方便，就像前面讨论的那样。在 C# 中，设置 clamd 的自动化非常简单。只需要两个小类：一个会话类和一个管理类。

安装 clamd 守护进程

在大多数平台上，从包管理器安装 ClamAV 可能不会安装 clamd 守护进程。例如，在 Ubuntu 上，你需要使用 apt 单独安装 clamav-daemon 包，如下所示：$ sudo apt-get install clamav-daemon；在 Red Hat 或 Fedora 上，你需要安装一个稍微不同的包：$ sudo yum install clamav-server

启动 clamd 守护进程

安装守护进程后，要使用 clamd，你需要启动守护进程，默认情况下，它会监听端口 3310 和地址 127.0.0.1。你可以使用 clamd 命令启动它，如 Listing 10-14 所示。

> $ clamd  Listing 10-14: 启动 clamd 守护进程

NOTE

> 如果你通过包管理器安装 clamd，它可能默认配置为监听本地 UNIX 套接字，而不是网络接口。如果你在使用 TCP 套接字连接到 clamd 守护进程时遇到问题，请确保 clamd 配置为监听网络接口！

当你运行命令时，可能不会得到任何反馈。没有消息就是好消息！如果 clamd 启动时没有任何消息，那么你已经成功启动它了。我们可以通过 netcat 测试 clamd 是否正常运行，通过连接到监听端口并查看当我们手动运行命令时会发生什么，例如获取当前的 clamd 版本并扫描文件，如 Listing 10-15 所示。

> $ echo VERSION | nc -v 127.0.0.1 3310
> 
> ClamAV 0.99/20563/Thu Jun 11 15:05:30 2015
> 
> $ echo "SCAN /tmp/eicar.com.txt" | nc -v 127.0.0.1 3310
> 
> /tmp/eicar.com.txt: Eicar-Test-Signature FOUND

Listing 10-15: 使用 netcat TCP 工具运行简单的 clamd 命令

连接到 clamd 并发送 VERSION 命令应该会打印 ClamAV 版本。你也可以发送 SCAN 命令，并将文件路径作为参数，它应该返回扫描结果。编写自动化代码非常简单。

为 clamd 创建会话类

ClamdSession 类几乎不需要深入了解类中代码的工作原理，因为它非常简单。我们创建了一些属性来保存 clamd 运行的主机和端口，执行 clamd() 命令并执行的 Execute() 方法，以及一个 TcpClient 类来创建一个新的 TCP 流并将命令写入流中，如 Listing 10-16 所示。TcpClient 类最早是在 第四章 中介绍的，当时我们构建了自定义有效负载。我们也在 第七章 中使用了它，自动化了 OpenVAS 漏洞扫描器。

> public class ClamdSession
> 
> {
> 
> private string _host = null;
> 
> private int _port;
> 
> public ➊ClamdSession(string host, int port)
> 
> {
> 
> _host = host;
> 
> _port = port;
> 
> }
> 
> public string ➋Execute(string command)
> 
> {
> 
> string resp = string.Empty;
> 
> using (➌TcpClient client = new TcpClient(_host, _port))
> 
> {
> 
> using (NetworkStream stream = client.➍GetStream())
> 
> {
> 
> byte[] data = System.Text.Encoding.ASCII.GetBytes(command);
> 
> stream.➎Write(data, 0, data.Length);
> 
> ➏using (StreamReader rdr = new StreamReader(stream))
> 
> resp = rdr.ReadToEnd();
> 
> }
> 
> }
> 
> ➐返回 resp;
> 
> }
> 
> }

Listing 10-16: 创建一个新的 clamd 会话的类

ClamdSession 构造函数 ➊ 接受两个参数——要连接的主机和端口，然后将这些值赋给本地类变量，供 Execute() 方法使用。过去，我们的所有会话类都实现了 IDisposable 接口，但实际上 ClamdSession 类不需要这样做。我们完成工作后不需要清理任何内容，因为 clamd 是一个在端口上运行的守护进程，可以继续运行，因此这简化了我们的工作。

Execute() 方法 ➋ 接受一个参数：要在 clamd 实例上运行的命令。我们的 ClamdManager 类将只实现一些可能的 clamd 命令，因此你应该研究 clamd 协议命令，以了解可以自动化的其他强大命令。为了启动命令并开始读取 clamd 响应，我们首先创建一个新的 TcpClient 类 ➌，它使用主机并将端口作为 TcpClient 参数传递给构造函数。然后我们调用 GetStream() ➍ 来连接到 clamd 实例，以便将命令写入该连接。通过使用 Write() 方法 ➎，我们将命令写入流中，然后创建一个新的 StreamReader 类来读取响应 ➏。最后，我们将响应返回给调用者 ➐。

创建一个 clamd 管理器类

ClamdSession 类的简单性（如 Listing 10-17 所定义）使得 ClamdManager 类也非常简单。它只需要创建一个构造函数和两个方法来执行我们之前手动执行的 Listing 10-15 中的命令。

> public class ClamdManager
> 
> {
> 
> private ClamdSession _session = null;
> 
> public ➊ClamdManager(ClamdSession session)
> 
> {
> 
> _session = session;
> 
> }
> 
> public string ➋获取版本()
> 
> {
> 
> return _session.Execute("VERSION");
> 
> }
> 
> public string ➌扫描(string path)
> 
> {
> 
> return _session.Execute("SCAN " + path);
> 
> }
> 
> }

Listing 10-17: clamd 的管理器类

ClamdManager 构造函数 ➊ 接受一个参数——将执行命令的会话，并将其赋值给一个名为 _session 的本地类变量，其他方法可以使用该变量。

我们创建的第一个方法是 GetVersion() 方法 ➋，它通过将字符串 VERSION 传递给 Execute() 方法来执行 clamd VERSION 命令，该方法在 clamd 会话类中定义。该命令将版本信息返回给调用者。第二个方法 Scan() ➌，接受一个文件路径作为参数，它将该路径与 clamd SCAN 命令一起传递给 Execute() 方法。现在我们有了会话类和管理器类，我们可以将一切组合在一起。

使用 clamd 测试

将所有内容组合在一起只需要为 Main() 方法写几行代码，如 Listing 10-18 所示。

> public static void Main(string[] args)
> 
> {
> 
> ClamdSession session = new ➊ClamdSession("127.0.0.1", 3310);
> 
> ClamdManager manager = new ClamdManager(session);
> 
> Console.WriteLine(manager.➋GetVersion());
> 
> ➌foreach (string path in args)
> 
> Console.WriteLine(manager.Scan(path));
> 
> }

清单 10-18：自动化 clamd 的 Main()方法

我们通过将 127.0.0.1 作为连接主机和 3310 作为主机端口来创建 ClamdSession() ➊。然后我们将新的 ClamdSession 传递给 ClamdManager 构造函数。使用新的 ClamdManager()，我们可以打印 clamd 实例的版本➋；然后我们遍历➌传递给程序的每个参数，尝试扫描文件并将结果打印到屏幕上供用户查看。在我们的例子中，我们只会测试一个文件——EICAR 测试文件。然而，你可以根据命令行的允许，将任意多个文件添加到扫描队列中。

我们将扫描的文件需要位于运行 clamd 守护进程的服务器上，因此，为了在网络中工作，你需要一种方法将文件发送到服务器上的 clamd 可以读取的地方。这可以是一个远程网络共享或其他将文件传送到服务器的方式。在这个例子中，我们让 clamd 监听 127.0.0.1（本地地址），并且它可以扫描我在 Mac 上的主目录，这一点在清单 10-19 中得到了展示。

> $ ./ch10_automating_clamav_clamd.exe ~/eicar.com.txt
> 
> ClamAV 0.99/20563/Thu Jun 11 15:05:30 2015
> 
> /Users/bperry/eicar.com.txt: 找到 Eicar-Test-Signature

清单 10-19：自动化 clamd 程序扫描硬编码的 EICAR 文件

你会注意到，使用 clamd 要比使用 libclamav 自动化快得多。这是因为 libclamav 程序花费的时间大部分用于分配和编译引擎，而不是实际扫描我们的文件。而 clamd 守护进程只需在启动时分配引擎一次；因此，当我们提交文件进行扫描时，结果会更快。我们可以通过运行带有 time 命令的应用程序来测试这一点，该命令会打印程序运行所需的时间，如清单 10-20 所示。

> $ time ./ch10_automating_clamav_fs.exe ~/eicar.com.txt
> 
> 找到：Eicar-Test-Signature
> 
> 实际时间 ➊0m11.872s
> 
> 用户 0m11.508s
> 
> 系统 0m0.254s
> 
> $ time ./ch10_automating_clamav_clamd.exe ~/eicar.com.txt
> 
> ClamAV 0.99/20563/Thu Jun 11 15:05:30 2015
> 
> /Users/bperry/eicar.com.txt: 找到 Eicar-Test-Signature
> 
> 实际时间 ➋0m0.111s
> 
> 用户 0m0.087s
> 
> 系统 0m0.011s 清单 10-20：ClamAV 和 clamd 应用程序扫描同一文件所需时间的比较

请注意，我们的第一个程序扫描 EICAR 测试文件花费了 11 秒钟➊，而第二个使用 clamd 的程序则只用了不到一秒钟➋。

结论

ClamAV 是一个功能强大且灵活的杀毒解决方案，适用于家庭和办公使用。在本章中，我们成功地以两种不同的方式使用了 ClamAV。

首先，我们为原生 libclamav 库实现了一些小型绑定。这让我们可以根据需要分配、扫描并释放 ClamAV 引擎，但代价是每次运行程序时都需要附带一份 libclamav 副本并分配一个昂贵的引擎。接着，我们实现了两个类，允许我们驱动远程 clamd 实例来获取 ClamAV 版本信息，并扫描 clamd 服务器上的指定文件路径。这有效地为我们的程序提供了显著的速度提升，但代价是要求待扫描的文件必须位于运行 clamd 的服务器上。

ClamAV 项目是一个很好的例子，展示了大公司（思科）如何真正支持开源软件，造福每个人。你会发现，扩展这些绑定来更好地保护和防御你的应用程序、用户和网络，是一个非常好的练习。
