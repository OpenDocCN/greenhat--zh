# 第八章：8 WINDOWS 事件追踪

![](img/opener-img.png)

通过 Windows 事件追踪（ETW）日志记录功能，开发人员可以编写应用程序，发出事件、从其他组件接收事件，并控制事件追踪会话。这使得他们能够追踪代码的执行，监控或调试潜在问题。可以将 ETW 看作是 *printf* 调试的替代方案；这些消息通过一个公共通道，使用标准格式发出，而不是打印到控制台。

在安全环境中，ETW 提供了宝贵的遥测数据，否则终端代理无法获取。例如，通用语言运行时（CLR），它被加载到每个 .NET 进程中，利用 ETW 发出独特的事件，能够比任何其他机制提供更多关于托管代码执行情况的洞察。这使得 EDR 代理能够收集新的数据，从中创建新的警报或丰富现有事件。

ETW 很少因其简单性和易用性而受到赞扬，这在很大程度上归功于微软为其提供的极其复杂的技术文档。幸运的是，尽管 ETW 的内部工作原理和实现细节非常有趣，但你不需要完全理解其架构。本章将介绍 ETW 中与遥测相关的部分。我们将演示代理如何从 ETW 收集遥测数据，以及如何避免此类收集。

## 架构

ETW 涉及三个主要组件：提供者、消费者和控制器。这些组件在事件追踪会话中各自发挥独特的作用。以下概述了每个组件在 ETW 架构中的作用。

### 提供者

简单来说，提供者是发出事件的软件组件。这些组件可能包括系统的各个部分，如任务调度程序、第三方应用程序，甚至是内核本身。通常，提供者不是一个单独的应用程序或镜像，而是与该组件关联的主要镜像。

当这个提供者镜像执行一些有趣或令人担忧的代码路径时，开发人员可以选择让其发出与执行相关的事件。例如，如果应用程序处理用户身份验证，当身份验证失败时，它可能会发出事件。这些事件包含开发人员认为调试或监控应用程序所需的任何数据，从简单的字符串到复杂的结构体。

ETW 提供者具有 GUID，其他软件可以使用这些 GUID 来识别它们。此外，提供者还具有更为用户友好的名称，通常在它们的清单中定义，便于人类更轻松地识别它们。在默认的 Windows 10 安装中，约有 1,100 个注册的提供者。表 8-1 包含了终端安全产品可能会觉得有用的提供者。

表 8-1： 与安全监控相关的默认 ETW 提供者

| 提供者名称 | GUID | 描述 |
| --- | --- | --- |
| Microsoft-Antimalware-Scan-Interface | {2A576B87-09A7-520E-C21A-4942F0271D67} | 提供有关通过反恶意软件扫描接口（AMSI）传递的数据的详细信息 |
| Microsoft-Windows-DotNETRuntime | {E13C0D23-CCBC-4E12-931B-D9CC2EEE27E4} | 提供与在本地主机上执行的 .NET 程序集相关的事件 |
| Microsoft-Windows-Audit-CVE | {85A62A0D-7E17-485F-9D4F-749A287193A6} | 提供一种机制，供软件报告尝试利用已知漏洞的行为 |
| Microsoft-Windows-DNS-Client | {1C95126E-7EEA-49A9-A3FE-A378B03DDB4D} | 详细说明主机上的域名解析结果 |
| Microsoft-Windows-Kernel-Process | {22FB2CD6-0E7B-422B-A0C7-2FAD1FD0E716} | 提供与进程的创建和终止相关的信息（类似于驱动程序可以使用的进程创建回调例程） |
| Microsoft-Windows-PowerShell | {A0C1853B-5C40-4B15-8766-3CF1C58F985A} | 提供 PowerShell 脚本块日志记录功能 |
| Microsoft-Windows-RPC | {6AD52B32-D609-4BE9-AE07-CE8DAE937E39} | 包含与本地系统上 RPC 操作相关的信息 |
| Microsoft-Windows-Security-Kerberos | {98E6CFCB-EE0A-41E0-A57B-622D4E1B30B1} | 提供与主机上的 Kerberos 认证相关的信息 |
| Microsoft-Windows-Services | {0063715B-EEDA-4007-9429-AD526F62696E} | 发出与服务的安装、操作和移除相关的事件 |
| Microsoft-Windows-SmartScreen | {3CB2A168-FE34-4A4E-BDAD-DCF422F34473} | 提供与 Microsoft Defender SmartScreen 相关的事件，以及其与从互联网下载的文件的交互 |
| Microsoft-Windows-TaskScheduler | {DE7B24EA-73C8-4A09-985D-5BDADCFA9017} | 提供与计划任务相关的信息 |
| Microsoft-Windows-WebIO | {50B3E73C-9370-461D-BB9F-26F32D68887D} | 提供对系统用户发起的网页请求的可见性 |
| Microsoft-Windows-WMI-Activity | {1418EF04-B0B4-4623-BF7E-D74AB47BBDAA} | 提供与 WMI 操作相关的遥测信息，包括事件订阅 |

ETW 提供者是可安全控制的对象，这意味着可以应用安全描述符。*安全描述符*为 Windows 提供了一种通过自主访问控制列表限制对该对象的访问，或者通过系统访问控制列表记录访问尝试的方式。列表 8-1 显示了应用于 Microsoft-Windows-Services 提供者的安全描述符。

```
PS > **$SDs = Get-ItemProperty -Path HKLM:\System\CurrentControlSet\Control\WMI\Security**
PS > **$sddl = ([wmiclass]"Win32_SecurityDescriptorHelper").**
**>> BinarySDToSDDL($SDs.****'****0063715b-eeda-4007-9429-ad526f62696e****'****).**
**>> SDDL**

PS > **ConvertFrom-SddlString -Sddl $sddl**
Owner            : BUILTIN\Administrators
Group            : BUILTIN\Administrators
DiscretionaryAcl : {NT AUTHORITY\SYSTEM: AccessAllowed,
                   NT AUTHORITY\LOCAL SERVICE: AccessAllowed,
                   BUILTIN\Administrators: AccessAllowed}
SystemAcl        : {}
RawDescriptor    : System.Security.AccessControl.CommonSecurityDescriptor
```

列表 8-1：评估应用于提供者的安全描述符

该命令通过提供者的 GUID 解析提供者注册表配置中的二进制安全描述符。然后，它使用 Win32 _SecurityDescriptorHelper WMI 类将注册表中的字节数组转换为安全描述符定义语言字符串。该字符串随后传递给 PowerShell cmdlet ConvertFrom-SddlString，以返回安全描述符的可读详细信息。默认情况下，该安全描述符仅允许 *NT AUTHORITY\SYSTEM*、*NT AUTHORITY\LOCAL SERVICE* 和本地管理员组成员访问。这意味着控制器代码必须以管理员身份运行，才能直接与提供者交互。

#### 发出事件

目前，有四种主要技术允许开发人员从其提供者应用程序中发出事件：

**托管对象格式 (MOF)**

MOF 是定义事件的语言，使消费者知道如何接收和处理这些事件。为了使用 MOF 注册和写入事件，提供者分别使用 sechost!RegisterTraceGuids() 和 advapi!TraceEvent() 函数。

**Windows 软件跟踪预处理器 (WPP)**

类似于 Windows 事件日志，WPP 是一种系统，允许提供者记录事件 ID 和事件数据，最初以二进制格式存储，稍后格式化为可供人类阅读的形式。WPP 支持比 MOF 更复杂的数据类型，包括时间戳和 GUID，并作为 MOF 基于提供者的补充。与基于 MOF 的提供者类似，WPP 提供者使用 sechost!RegisterTraceGuids() 和 advapi!TraceEvent() 函数来注册和写入事件。WPP 提供者还可以使用 WPP_INIT_TRACING 宏来注册提供者 GUID。

**清单**

清单是包含定义提供者元素的 XML 文件，其中包括有关事件格式和提供者本身的详细信息。这些清单在编译时嵌入到提供者二进制文件中并注册到系统。使用清单的提供者依赖于 advapi!EventRegister() 函数来注册事件，并使用 advapi!EventWrite() 函数来写入事件。如今，这似乎是注册提供者的最常见方式，特别是那些随 Windows 一起发布的提供者。

**跟踪日志记录 (TraceLogging)**

在 Windows 10 中引入的 TraceLogging 是提供事件的最新技术。与其他技术不同，TraceLogging 允许*自描述*事件，这意味着消费者无需为事件注册任何类或清单，便能知道如何处理这些事件。消费者使用 Trace 数据助手（TDH）API 来解码和处理事件。这些提供者使用 advapi!TraceLoggingRegister() 和 advapi!TraceLoggingWrite() 来注册和写入事件。

无论开发者选择哪种方法，结果都是一样的：应用程序发出的事件供其他应用程序使用。

#### 定位事件源

要理解为什么提供者会发出某些事件，通常查看提供者本身会很有帮助。不幸的是，Windows 并没有提供一种简单的方法将提供者的名称或 GUID 转换为磁盘上的映像。有时，你可以从事件的元数据中收集这些信息，但在许多情况下，尤其是当事件源是 DLL 或驱动程序时，发现它需要更多的努力。在这些情况下，可以考虑以下 ETW 提供者的属性：

+   提供者的 PE 文件必须引用其 GUID，通常是在 *.rdata* 区段，该区段保存只读初始化数据。

+   提供者必须是可执行代码文件，通常是 *.exe*、*.dll* 或 *.sys* 文件。

+   提供者必须调用注册 API（具体来说，对于用户模式应用程序是 advapi!EventRegister() 或 ntdll!EtwEventRegister()，对于内核模式组件是 ntoskrnl!EtwRegister()）。

+   如果使用系统注册的清单，提供者的映像将位于注册表项 *HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\WINEVT\Publishers\<PROVIDER_GUID>* 中的 ResourceFileName 值。该文件将包含 *WEVT_TEMPLATE* 资源，这是清单的二进制表示。

你可以对操作系统中的文件进行扫描，并返回符合这些要求的文件。GitHub 上的开源工具 *FindETWProviderImage* 可以简化这个过程。清单 8-2 使用它来定位引用 Microsoft-Windows-TaskScheduler 提供者 GUID 的映像。

```
PS > **.\FindETWProviderImage.exe "Microsoft-Windows-TaskScheduler" "C:\Windows\System32\"**
Translated Microsoft-Windows-TaskScheduler to {de7b24ea-73c8-4a09-985d-5bdadcfa9017}
Found provider in the registry: C:\WINDOWS\system32\schedsvc.dll

Searching 5486 files for {de7b24ea-73c8-4a09-985d-5bdadcfa9017} …

Target File: C:\Windows\System32\aitstatic.exe
Registration Function Imported: True
Found 1 reference:
 1) Offset: 0x2d8330 RVA: 0x2d8330 (.data)

Target File: C:\Windows\System32\schedsvc.dll
Registration Function Imported: True
Found 2 references:
 1) Offset: 0x6cb78 RVA: 0x6d778 (.rdata)
 2) Offset: 0xab910 RVA: 0xaf110 (.pdata) Target File: C:\Windows\System32\taskcomp.dll
Registration Function Imported: False
Found 1 reference:
 1) Offset: 0x39630 RVA: 0x3aa30 (.rdata)

Target File: C:\Windows\System32\ubpm.dll
Registration Function Imported: True
Found 1 reference:
 1) Offset: 0x38288 RVA: 0x39a88 (.rdata)

Total References: 5
Time Elapsed: 1.168 seconds
```

清单 8-2：使用 FindETWProviderImage 定位提供者二进制文件

如果你考虑一下输出，你会发现这种方法存在一些漏洞。例如，工具返回了事件的真实提供者，*schedsvc.dll*，但也返回了另外三个镜像。这些误报可能是因为镜像从目标提供者中消耗了事件，因此包含了提供者的 GUID，或者是因为它们产生了自己的事件，因此调用了其中一个注册 API。这个方法也可能会产生漏报；例如，当事件的来源是*ntoskrnl.exe*时，镜像在注册表中找不到，或者没有导入任何注册函数。

为了确认提供者的身份，你需要进一步调查该镜像。你可以使用一种相对简单的方法。在反汇编器中，导航到*FindETWProviderImage*报告的偏移量或相对虚拟地址，并查找任何来自调用注册 API 的函数的 GUID 引用。你应该能够看到 GUID 的地址被传递到注册函数的 RCX 寄存器中，如清单 8-3 所示。

```
schedsvc!JobsService::Initialize+0xcc:
00007ffe`74096f5c 488935950a0800  mov   qword ptr [schedsvc!g_pEventManager],rsi
00007ffe`74096f63 4c8bce          mov   r9,rsi
00007ffe`74096f66 4533c0          xor   r8d,r8d
00007ffe`74096f69 33d2            xor   edx,edx
00007ffe`74096f6b 488d0d06680400  lea   rcx,[schedsvc!TASKSCHED] ❶
00007ffe`74096f72 48ff150f570400  call  qword ptr [schedsvc!_imp_EtwEventRegister ❷
00007ffe`74096f79 0f1f440000      nop   dword ptr [rax+rax]
00007ffe`74096f7e 8bf8            mov   edi,eax
00007ffe`74096f80 48391e          cmp   qword ptr [rsi],rbx
00007ffe`74096f83 0f84293f0100    je    schedsvc!JobsService::Initialize+0x14022
```

清单 8-3：在 schedsvc.dll 内部的提供者注册函数的反汇编

在这段反汇编代码中，有两个指令对我们很重要。第一个是提供者 GUID 的地址被加载到 RCX 寄存器中 ❶。紧接着是调用导入的ntdll!EtwEventRegister()函数 ❷，将提供者注册到操作系统中。

#### 弄清楚为什么一个事件被触发

到此为止，你已经确定了提供者。从这里开始，许多检测工程师会开始调查是什么条件触发了提供者发出事件。这个过程的细节超出了本书的范围，因为它们根据提供者的不同可能会有很大的差异，尽管我们将在第十二章中更深入地探讨这个话题。然而，通常来说，工作流程如下所示。

在反汇编器中，标记从事件注册 API 返回的REGHANDLE，然后查找该REGHANDLE的引用，来自一个写入 ETW 事件的函数，例如ntoskrnl!EtwWrite()。逐步执行该函数，查找传递给它的UserData参数的来源。跟踪从这个来源到事件写入函数的执行，检查是否存在任何条件分支会阻止事件的发出。对每一个指向全局REGHANDLE的独特引用重复这些步骤。

### 控制器

控制器是定义和控制*跟踪会话*的组件，跟踪会话记录由提供程序写入的事件，并将其刷新到事件消费者。控制器的任务包括启动和停止会话，启用或禁用与会话关联的提供程序，管理事件缓冲池的大小等。单个应用程序可能包含控制器和消费者代码；或者，控制器也可以是一个完全独立的应用程序，例如 Xperf 和 logman，它们是收集和处理 ETW 事件的两个工具。

控制器使用 sechost!StartTrace() API 创建跟踪会话，并使用 sechost!ControlTrace() 和 advapi!EnableTraceEx() 或 sechost!EnableTraceEx2() 进行配置。在 Windows XP 及以后的版本中，控制器最多可以启动并管理 64 个同时的跟踪会话。要查看这些跟踪会话，可以使用 logman，如 Listing 8-4 所示。

```
PS > **logman.exe query -ets**

Data Collector Set                      Type         Status
-------------------------------------------------------------
AppModel                                Trace        Running
BioEnrollment                           Trace        Running
Diagtrack-Listener                      Trace        Running
FaceCredProv                            Trace        Running
FaceTel                                 Trace        Running
LwtNetLog                               Trace        Running
Microsoft-Windows-Rdp-Graphics-RdpIdd-Trace Trace    Running
NetCore                                 Trace        Running
NtfsLog                                 Trace        Running
RadioMgr                                Trace        Running
WiFiDriverIHVSession                    Trace        Running
WiFiSession                             Trace        Running UserNotPresentTraceSession              Trace        Running
NOCAT                                   Trace        Running
Admin_PS_Provider                       Trace        Running
WindowsUpdate_trace_log                 Trace        Running
MpWppTracing-20220120-151932-00000003-ffffffff Trace Running
SHS-01202022-151937-7-7f                Trace        Running
SgrmEtwSession                          Trace        Running
```

Listing 8-4：使用 logman.exe 枚举跟踪会话

Data Collector Set 列下的每个名称表示一个独特的控制器，具有自己的下属跟踪会话。 Listing 8-4 中显示的控制器是内置于 Windows 中的，因为操作系统也大量使用 ETW 进行活动监控。

控制器还可以查询现有的跟踪以获取信息。 Listing 8-5 展示了这一过程。

```
PS > **logman.exe query 'EventLog-System' -ets**

Name:                  EventLog-System
Status:                Running
Root Path:             %systemdrive%\PerfLogs\Admin
Segment:               Off
Schedules:             On
Segment Max Size:      100 MB

Name:                  EventLog-System\EventLog-System
Type:                  Trace
Append:                Off
Circular:              Off
Overwrite:             Off
Buffer Size:           64
Buffers Lost:          0
Buffers Written:       155
Buffer Flush Timer:    1
Clock Type:            System
❶ File Mode:             Real-time

Provider:
❷ Name:                  Microsoft-Windows-FunctionDiscoveryHost
Provider Guid:         {538CBBAD-4877-4EB2-B26E-7CAEE8F0F8CB}
Level:                 255
KeywordsAll:           0x0
❸ KeywordsAny:           0x8000000000000000 (System)
Properties:            65
Filter Type:           0

Provider:
Name:                  Microsoft-Windows-Subsys-SMSS
Provider Guid:         {43E63DA5-41D1-4FBF-ADED-1BBED98FDD1D}
Level:                 255
KeywordsAll:           0x0
KeywordsAny:           0x4000000000000000 (System) Properties:            65
Filter Type:           0
`--snip--`
```

Listing 8-5：使用 logman.exe 查询特定的跟踪

该查询为我们提供了有关会话中启用的提供程序❷以及使用的过滤关键字❸的信息，是否为实时跟踪或基于文件的跟踪❶，以及性能数据。通过这些信息，我们可以开始理解该跟踪是否为 EDR 进行的性能监控或遥测收集。

### 消费者

消费者是接收事件的软件组件，这些事件在被跟踪会话记录后送达。它们可以从磁盘上的日志文件中读取事件，也可以实时消费事件。由于几乎每个 EDR 代理都是实时消费者，我们将专注于这类消费者。

消费者使用 sechost!OpenTrace() 连接到实时会话，并使用 sechost!ProcessTrace() 开始从中消费事件。每次消费者收到新事件时，一个内部定义的回调函数根据提供者提供的信息（如事件清单）解析事件数据。消费者然后可以选择对这些信息执行任意操作。在端点安全软件的情况下，这可能意味着创建警报、采取一些预防措施，或将活动与其他传感器收集的遥测数据关联起来。

## 创建一个消费者来识别恶意 .NET 程序集

让我们逐步了解开发消费者并处理事件的过程。在本节中，我们将识别恶意内存中的 .NET 框架程序集的使用，例如 Cobalt Strike 的 Beacon execute-assembly 功能使用的那些程序集。识别这些程序集的一种策略是寻找属于已知攻击性 C# 项目的类名。尽管攻击者可以通过更改恶意软件的类名和方法轻松绕过此技巧，但它仍然是识别不修改工具的较低技术水平攻击者使用工具的一种有效方式。

我们的消费者将从 Microsoft-Windows-DotNETRuntime 提供者中获取过滤后的事件，特别是关注与 Seatbelt 相关的类，Seatbelt 是一种后期利用的 Windows 侦察工具。

### 创建追踪会话

要开始消费事件，我们必须首先使用 sechost!StartTrace() API 创建一个追踪会话。此函数接受一个指向 EVENT_TRACE_PROPERTIES 结构体的指针，该结构体在示例 8-6 中定义。（在运行 Windows 1703 版本之后的系统上，函数可能会选择接受一个指向 EVENT_TRACE_PROPERTIES_V2 结构体的指针。）

```
typedef struct _EVENT_TRACE_PROPERTIES {
  WNODE_HEADER Wnode;
  ULONG        BufferSize;
  ULONG        MinimumBuffers;
  ULONG        MaximumBuffers;
  ULONG        MaximumFileSize;
  ULONG        LogFileMode;
  ULONG        FlushTimer;
  ULONG        EnableFlags;
  union {
    LONG AgeLimit;
    LONG FlushThreshold;
  } DUMMYUNIONNAME;
  ULONG        NumberOfBuffers;
  ULONG        FreeBuffers;
  ULONG        EventsLost;
  ULONG        BuffersWritten;
  ULONG        LogBuffersLost;
  ULONG        RealTimeBuffersLost;
  HANDLE LoggerThreadId;
  ULONG        LogFileNameOffset;
  ULONG        LoggerNameOffset;
} EVENT_TRACE_PROPERTIES, *PEVENT_TRACE_PROPERTIES;
```

示例 8-6：EVENT_TRACE_PROPERTIES 结构体定义

该结构体描述了追踪会话。消费者将填充该结构体并将其传递给一个启动追踪会话的函数，如示例 8-7 所示。

```
static const GUID g_sessionGuid =
{0xb09ce00c, 0xbcd9, 0x49eb,
{0xae, 0xce, 0x42, 0x45, 0x1, 0x2f, 0x97, 0xa9}
};
static const WCHAR g_sessionName[] = L"DotNETEventConsumer";

int main()
{
    ULONG ulBufferSize =
        sizeof(EVENT_TRACE_PROPERTIES) + sizeof(g_sessionName);
    PEVENT_TRACE_PROPERTIES pTraceProperties =
        (PEVENT_TRACE_PROPERTIES)malloc(ulBufferSize);
    if (!pTraceProperties)
    {
        return ERROR_OUTOFMEMORY;
    }
    ZeroMemory(pTraceProperties, ulBufferSize);

    pTraceProperties->Wnode.BufferSize = ulBufferSize;
    pTraceProperties->Wnode.Flags = WNODE_FLAG_TRACED_GUID;
    pTraceProperties->Wnode.ClientContext = 1;
    pTraceProperties->Wnode.Guid = g_sessionGuid;
    pTraceProperties->LogFileMode = EVENT_TRACE_REAL_TIME_MODE;
    pTraceProperties->LoggerNameOffset = sizeof(EVENT_TRACE_PROPERTIES); wcscpy_s(
        (PWCHAR)(pTraceProperties + 1),
        wcslen(g_sessionName) + 1,
        g_sessionName);

    DWORD dwStatus = 0;
    TRACEHANDLE hTrace = NULL;

    while (TRUE) {
        dwStatus = StartTraceW(
            &hTrace,
            g_sessionName,
            pTraceProperties);

        if (dwStatus == ERROR_ALREADY_EXISTS)
        {
            dwStatus = ControlTraceW(
                hTrace,
                g_sessionName,
                pTraceProperties,
                EVENT_TRACE_CONTROL_STOP);
        }
    if (dwStatus != ERROR_SUCCESS)
    {
            return dwStatus;
    }

    `--snip--`
}
```

示例 8-7：配置追踪属性

我们填充指向跟踪属性中的 WNODE_HEADER 结构。请注意，Guid 成员包含的是跟踪会话的 GUID，而不是所需提供者的 GUID。此外，跟踪属性结构中的 LogFileMode 成员通常设置为 EVENT_TRACE_REAL_TIME_MODE，以启用实时事件跟踪。

### 启用提供者

该跟踪会话尚未开始收集事件，因为没有为其启用任何提供者。为了添加提供者，我们使用 sechost!EnableTraceEx2() API。此函数将先前返回的 TRACEHANDLE 作为参数，并在 Listing 8-8 中定义。

```
ULONG WMIAPI EnableTraceEx2(
  [in]           TRACEHANDLE               TraceHandle,
  [in]           LPCGUID                   ProviderId,
  [in]           ULONG                     ControlCode,
  [in]           UCHAR                     Level,
  [in]           ULONGLONG                 MatchAnyKeyword,
  [in]           ULONGLONG                 MatchAllKeyword, [in]           ULONG                     Timeout,
  [in, optional] PENABLE_TRACE_PARAMETERS EnableParameters
);
```

Listing 8-8: sechost!EnableTraceEx2() 函数定义

ProviderId 参数是目标提供者的 GUID，Level 参数决定了传递给消费者的事件的严重性。它的范围可以从 TRACE_LEVEL_VERBOSE (*5*) 到 TRACE_LEVEL_CRITICAL (*1*)。消费者将接收所有级别小于或等于指定值的事件。

MatchAllKeyword 参数是一个位掩码，只有当事件的关键字位与该值中设置的所有位匹配时，事件才会被写入（或者如果事件没有设置关键字位）。在大多数情况下，该成员设置为零。MatchAnyKeyword 参数是一个位掩码，只有当事件的关键字位与该值中设置的任意位匹配时，事件才会被写入。

EnableParameters 参数允许消费者在每个事件中接收一个或多个扩展数据项，包括但不限于以下内容：

EVENT_ENABLE_PROPERTY_PROCESS_START_KEY   标识进程的序列号，保证在当前启动会话中唯一

EVENT_ENABLE_PROPERTY_SID   事件发出时的主体的安全标识符，例如系统的用户

EVENT_ENABLE_PROPERTY_TS_ID   事件发出时的终端会话标识符

EVENT_ENABLE_PROPERTY_STACK_TRACE   如果事件是使用advapi!EventWrite() API 写入的，则该值会添加调用堆栈。

sechost!EnableTraceEx2() API 可以将任意数量的提供程序添加到跟踪会话中，每个提供程序都有自己的过滤配置。列表 8-9 继续展示了 列表 8-7 中的代码，并演示了此 API 的常见用法。

```
❶ static const GUID g_providerGuid =
{0xe13c0d23, 0xccbc, 0x4e12,
{0x93, 0x1b, 0xd9, 0xcc, 0x2e, 0xee, 0x27, 0xe4}
};
int main()
{
    `--snip--`

    dwStatus = EnableTraceEx2(
        hTrace,
        &g_providerGuid,
        EVENT_CONTROL_CODE_ENABLE_PROVIDER,
        TRACE_LEVEL_INFORMATION,
      ❷ 0x2038,
        0,
        INFINITE,
        NULL); if (dwStatus != ERROR_SUCCESS)
    {
        goto Cleanup;
    }

    `--snip--`
}
```

列表 8-9：为跟踪会话配置提供程序

我们将 Microsoft-Windows-DotNETRuntime 提供程序 ❶ 添加到跟踪会话，并将 MatchAnyKeyword 设置为使用 Interop (0x2000)、NGen (0x20)、Jit (0x10) 和 Loader (0x8) 关键字 ❷。这些关键字使我们能够过滤掉不感兴趣的事件，只收集与我们试图监视的内容相关的事件。

### 启动跟踪会话

在完成所有这些准备工作后，我们可以启动跟踪会话。为此，EDR 代理会调用 sechost!OpenTrace()，并将指向 列表 8-10 中定义的 EVENT_TRACE_LOGFILE 结构的指针作为唯一参数传递。

```
typedef struct _EVENT_TRACE_LOGFILEW {
   LPWSTR                        LogFileName;
   LPWSTR                        LoggerName;
   LONGLONG                      CurrentTime;
   ULONG                         BuffersRead;
   union {
     ULONG LogFileMode;
     ULONG ProcessTraceMode;
   } DUMMYUNIONNAME;
   EVENT_TRACE                   CurrentEvent;
   TRACE_LOGFILE_HEADER          LogfileHeader;
   PEVENT_TRACE_BUFFER_CALLBACKW BufferCallback;
   ULONG                         BufferSize;
   ULONG                         Filled;
   ULONG                         EventsLost;
   union {
     PEVENT_CALLBACK        EventCallback;
     PEVENT_RECORD_CALLBACK EventRecordCallback;
   } DUMMYUNIONNAME2;
   ULONG                          IsKernelTrace;
   PVOID                          Context;
}  EVENT_TRACE_LOGFILEW, *PEVENT_TRACE_LOGFILEW;
```

列表 8-10：EVENT_TRACE_LOGFILE 结构定义

列表 8-11 演示了如何使用此结构。

```
int main()
{
    `--snip--`

    EVENT_TRACE_LOGFILEW etl = {0}; ❶ etl.LoggerName = g_sessionName;
 ❷ etl.ProcessTraceMode = PROCESS_TRACE_MODE_EVENT_RECORD |
                           PROCESS_TRACE_MODE_REAL_TIME;
 ❸ etl.EventRecordCallback = OnEvent;

    TRACEHANDLE hSession = NULL;
    hSession = OpenTrace(&etl);
    if (hSession == INVALID_PROCESSTRACE_HANDLE)
    {
        goto Cleanup;
    }

    `--snip--`
}
```

列表 8-11：将 EVENT_TRACE_LOGFILE 结构传递给 sechost!OpenTrace()

虽然这是一个相对较大的结构体，但只有三个成员与我们直接相关。LoggerName 成员是跟踪会话的名称 ❶，ProcessTraceMode 是一个位掩码，包含 PROCESS_TRACE_MODE_EVENT_RECORD（0x10000000）的值，表示事件应使用 Windows Vista 中引入的 EVENT_RECORD 格式，以及 PROCESS_TRACE_MODE_REAL_TIME（0x100），表示事件应实时接收 ❷。最后，EventRecordCallback 是指向内部回调函数的指针 ❸（稍后介绍），ETW 在每个新事件发生时会调用该函数，并将一个 EVENT_RECORD 结构体传递给它。

当 sechost!OpenTrace() 完成时，它返回一个新的 TRACEHANDLE（在我们的示例中是 hSession）。然后我们可以将这个句柄传递给 sechost!ProcessTrace()，如列表 8-12 所示，开始处理事件。

```
void ProcessEvents(PTRACEHANDLE phSession)
{
    FILETIME now;
 ❶ GetSystemTimeAsFileTime(&now);
    ProcessTrace(phSession, 1, &now, NULL);

}
int main()
{
    `--snip--`

    HANDLE hThread = NULL;
 ❷ hThread = CreateThread(
                  NULL, 0,
                  ProcessEvents,
                  &hSession,
                  0, NULL);

    if (!hThread)
    {
        goto Cleanup;
    } `--snip--`
}
```

列表 8-12：创建处理事件的线程

我们将当前系统时间 ❶ 传递给 sechost!ProcessTrace()，告诉系统我们只想捕获此时间之后发生的事件。当调用此函数时，它将接管当前线程，因此，为了避免完全阻塞应用程序的其他部分，我们为跟踪会话创建一个新的线程 ❷。

假设没有返回错误，事件应该开始从提供者流向消费者，并在 EVENT_TRACE_LOGFILE 结构体的 EventRecordCallback 成员指定的内部回调函数中进行处理。我们将在“处理事件”一节中讲解这个函数，见第 158 页。

### 停止跟踪会话

最后，我们需要一种方式来在需要时停止跟踪。一个方法是使用全局布尔值，当需要停止跟踪时，我们可以改变这个值，但任何可以通知线程退出的技术都可以使用。不过，如果外部用户能够调用此方法（例如在未检查的 RPC 函数的情况下），恶意用户可能会通过跟踪会话完全停止代理收集事件。列表 8-13 展示了停止跟踪的可能方式。

```
HANDLE g_hStop = NULL;
BOOL ConsoleCtrlHandler(DWORD dwCtrlType)

{
 ❶ if (dwCtrlType == CTRL_C_EVENT) {
      ❷ SetEvent(g_hStop);
        return TRUE;
    }
    return FALSE;
}

int main()
{
    `--snip--`

    g_hStop = CreateEvent(NULL, TRUE, FALSE, NULL);
    SetConsoleCtrlHandler(ConsoleCtrlHandler, TRUE);

    WaitForSingleObject(g_hStop, INFINITE);

 ❸ CloseTrace(hSession);
    WaitForSingleObject(hThread, INFINITE);
    CloseHandle(g_hStop);
    CloseHandle(hThread); return dwStatus
}
```

列表 8-13：使用控制台控制处理程序来信号线程退出

在此示例中，我们使用一个内部控制台控制处理程序例程 ConsoleCtrlHandler()，并且使用一个事件对象来监视 CTRL-C 键盘组合 ❶。当处理程序检测到此键盘组合时，内部函数会通知 *事件对象* ❷，这是一种常用于通知线程某些事件已经发生的同步对象，然后返回。由于事件对象已经被信号通知，应用程序恢复执行并关闭跟踪会话 ❸。

### 处理事件

当消费者线程接收到一个新事件时，它的回调函数（在我们的示例代码中为 OnEvent()）会被调用，并传递一个指向 EVENT_RECORD 结构的指针。这个结构在列表 8-14 中定义，表示整个事件。

```
typedef struct _EVENT_RECORD {
  EVENT_HEADER                      EventHeader;
  ETW_BUFFER_CONTEXT                BufferContext;
  USHORT                            ExtendedDataCount;
  USHORT                            UserDataLength;
  PEVENT_HEADER_EXTENDED_DATA_ITEM  ExtendedData;
  PVOID                             UserData;
  PVOID                             UserContext;
} EVENT_RECORD, *PEVENT_RECORD;
```

列表 8-14: EVENT_RECORD 结构定义

这个结构乍看之下可能很简单，但它可能包含大量信息。第一个字段 EventHeader 包含基本的事件元数据，例如提供者二进制文件的进程 ID、时间戳，以及一个 EVENT_DESCRIPTOR，它详细描述了事件本身。ExtendedData 成员与传递给 sechost!EnableTraceEx2() 中的 EnableProperty 参数的数据匹配。该字段是指向一个 EVENT_HEADER_EXTENDED_DATA_ITEM 的指针，在列表 8-15 中定义。

```
typedef struct _EVENT_HEADER_EXTENDED_DATA_ITEM {
  USHORT   Reserved1;
  USHORT   ExtType;
  struct {
    USHORT Linkage : 1;
    USHORT Reserved2 : 15;
  };
  USHORT   DataSize;
  ULONGLONG DataPtr;
} EVENT_HEADER_EXTENDED_DATA_ITEM, *PEVENT_HEADER_EXTENDED_DATA_ITEM;
```

列表 8-15: EVENT_HEADER_EXTENDED_DATA_ITEM 结构定义

ExtType 成员包含一个标识符（在 *eventcons.h* 中定义，并在列表 8-16 中显示），它告诉消费者 DataPtr 成员指向的数据类型。请注意，许多在头文件中定义的值在微软文档中并没有正式支持作为 API 调用者使用。

```
#define EVENT_HEADER_EXT_TYPE_RELATED_ACTIVITYID   0x0001
#define EVENT_HEADER_EXT_TYPE_SID                  0x0002
#define EVENT_HEADER_EXT_TYPE_TS_ID                0x0003
#define EVENT_HEADER_EXT_TYPE_INSTANCE_INFO        0x0004
#define EVENT_HEADER_EXT_TYPE_STACK_TRACE32        0x0005
#define EVENT_HEADER_EXT_TYPE_STACK_TRACE64        0x0006
#define EVENT_HEADER_EXT_TYPE_PEBS_INDEX           0x0007
#define EVENT_HEADER_EXT_TYPE_PMC_COUNTERS         0x0008
#define EVENT_HEADER_EXT_TYPE_PSM_KEY              0x0009
#define EVENT_HEADER_EXT_TYPE_EVENT_KEY            0x000A
#define EVENT_HEADER_EXT_TYPE_EVENT_SCHEMA_TL      0x000B
#define EVENT_HEADER_EXT_TYPE_PROV_TRAITS          0x000C
#define EVENT_HEADER_EXT_TYPE_PROCESS_START_KEY    0x000D
#define EVENT_HEADER_EXT_TYPE_CONTROL_GUID         0x000E
#define EVENT_HEADER_EXT_TYPE_QPC_DELTA            0x000F
#define EVENT_HEADER_EXT_TYPE_CONTAINER_ID         0x0010
#define EVENT_HEADER_EXT_TYPE_MAX                  0x0011
```

列表 8-16: EVENT_HEADER_EXT_TYPE 常量

EVENT_RECORD 的 ExtendedData 成员包含有价值的数据，但代理通常会用它来补充其他来源，特别是 EVENT_RECORD 的 UserData 成员。这部分比较复杂，因为微软表示，几乎在所有情况下，我们必须通过 TDH API 来检索这些数据。

我们将在回调函数中逐步完成这个过程，但请记住，这个例子只是提取相关信息的一种方法，可能并不代表生产代码。为了开始处理事件数据，代理调用 tdh!TdhGetEventInformation()，如列表 8-17 所示。

```
void CALLBACK OnEvent(PEVENT_RECORD pRecord)
{
    ULONG ulSize = 0;
    DWORD dwStatus = 0;
    PBYTE pUserData = (PBYTE)pRecord->UserData;

    dwStatus = TdhGetEventInformation(pRecord, 0, NULL, NULL, &ulSize);

    PTRACE_EVENT_INFO pEventInfo = (PTRACE_EVENT_INFO)malloc(ulSize);
    if (!pEventInfo)
    {
        // Exit immediately if we're out of memory
        ExitProcess(ERROR_OUTOFMEMORY);
    }

    dwStatus = TdhGetEventInformation(
        pRecord, 0,
        NULL,
        pEventInfo,
        &ulSize);
    if (dwStatus != ERROR_SUCCESS)
    {
        return;
    }

    `--snip--`
}
```

列表 8-17：开始处理事件数据

在分配了所需大小的内存后，我们将指针传递给 TRACE_EVENT_INFO 结构体，作为函数的第一个参数。列表 8-18 定义了这个结构体。

```
typedef struct _TRACE_EVENT_INFO {
  GUID                ProviderGuid;
  GUID                EventGuid;
  EVENT_DESCRIPTOR    EventDescriptor;
❶ DECODING_SOURCE     DecodingSource;
  ULONG               ProviderNameOffset;
  ULONG               LevelNameOffset;
  ULONG               ChannelNameOffset;
  ULONG               KeywordsNameOffset;
  ULONG               TaskNameOffset;
  ULONG               OpcodeNameOffset;
  ULONG               EventMessageOffset;
  ULONG               ProviderMessageOffset;
  ULONG               BinaryXMLOffset;
  ULONG               BinaryXMLSize;
  union {
    ULONG EventNameOffset;
    ULONG ActivityIDNameOffset;
  };
  union {
    ULONG EventAttributesOffset;
    ULONG RelatedActivityIDNameOffset;
};
  ULONG               PropertyCount;
  ULONG               TopLevelPropertyCount;
  union {
    TEMPLATE_FLAGS Flags;
    struct {
      ULONG Reserved : 4;
      ULONG Tags : 28;
    };
  };
❷ EVENT_PROPERTY_INFO EventPropertyInfoArray[ANYSIZE_ARRAY];
} TRACE_EVENT_INFO;
```

列表 8-18：TRACE_EVENT_INFO 结构体定义

当函数返回时，它将用有用的元数据填充此结构体，例如用于标识事件定义方式（在仪表清单、MOF 类或 WPP 模板中）的 DecodingSource ❶。但最重要的值是 EventPropertyInfoArray ❷，这是一个 EVENT_PROPERTY_INFO 结构体数组，在列表 8-19 中定义，提供有关 EVENT_RECORD 的 UserData 成员每个属性的信息。

```
typedef struct _EVENT_PROPERTY_INFO {
❶ PROPERTY_FLAGS Flags;
  ULONG   NameOffset;
  union {
    struct {
      USHORT InType;
      USHORT OutType;
      ULONG MapNameOffset;
    } nonStructType;
    struct {
      USHORT StructStartIndex;
      USHORT NumOfStructMembers;
      ULONG padding;
    } structType;
    struct {
      USHORT InType;
      USHORT OutType;
      ULONG CustomSchemaOffset;
    } customSchemaType;
  };
  union {
  ❷ USHORT count;
    USHORT countPropertyIndex;
  };
  union {
  ❸ USHORT length;
    USHORT lengthPropertyIndex;
  };
 union {
    ULONG Reserved;
    struct {
      ULONG Tags : 28;
    };
  };
} EVENT_PROPERTY_INFO;
```

列表 8-19：EVENT_PROPERTY_INFO 结构体

我们必须逐一解析数组中的每个结构体。首先，它获取所操作属性的长度。这个长度依赖于事件的定义方式（例如，MOF 或清单）。通常，我们通过以下方式来推导属性的大小：从length 成员 ❸，从已知数据类型的大小（例如无符号长整型或ulong），或通过调用 tdh!TdhGetPropertySize()。如果属性本身是一个数组，我们需要通过评估 count 成员 ❷ 或再次调用 tdh!TdhGetPropertySize() 来获取它的大小。

接下来，我们需要确定正在评估的数据是否本身是一个结构。由于调用者通常知道他们正在处理的数据格式，在大多数情况下这并不困难，通常只有在解析来自不熟悉提供者的事件时才变得重要。然而，如果代理确实需要处理事件中的结构，则 Flags 成员❶将包括 PropertyStruct (0x1) 标志。

当数据不是结构时，例如在 Microsoft-Windows-DotNETRuntime 提供者的情况下，它将是一个简单的值映射，我们可以使用 tdh!TdhGetEventMapInformation() 获取这个映射信息。此函数接受指向 TRACE_EVENT_INFO 的指针，以及指向映射名称偏移量的指针，它可以通过 MapNameOffset 成员进行访问。完成后，它返回指向 EVENT_MAP_INFO 结构的指针，该结构在 列表 8-20 中定义，描述了事件映射的元数据。

```
typedef struct _EVENT_MAP_INFO {
  ULONG           NameOffset;
  MAP_FLAGS       Flag;
  ULONG           EntryCount;
  union {
    MAP_VALUETYPE MapEntryValueType;
    ULONG         FormatStringOffset;
  };
  EVENT_MAP_ENTRY MapEntryArray[ANYSIZE_ARRAY];
} EVENT_MAP_INFO;
```

列表 8-20: EVENT_MAP_INFO 结构定义

列表 8-21 显示了我们的回调函数如何使用这个结构。

```
void CALLBACK OnEvent(PEVENT_RECORD pRecord)
{
  `--snip--`

    WCHAR pszValue[512];
    USHORT wPropertyLen = 0;
    ULONG ulPointerSize =
      (pRecord->EventHeader.Flags & EVENT_HEADER_FLAG_32_BIT_HEADER) ? 4 : 8;

    USHORT wUserDataLen = pRecord->UserDataLength;

 ❶ for (USHORT i = 0; i < pEventInfo->TopLevelPropertyCount; i++)
    {
        EVENT_PROPERTY_INFO propertyInfo =
          pEventInfo->EventPropertyInfoArray[i];
        PCWSTR pszPropertyName =
          PCWSTR)((BYTE*)pEventInfo + propertyInfo.NameOffset);

      wPropertyLen = propertyInfo.length;

    ❷ if ((propertyInfo.Flags & PropertyStruct | PropertyParamCount)) != 0)
      {
          return;
      }
      PEVENT_MAP_INFO pMapInfo = NULL; PWSTR mapName = NULL;

    ❸ if (propertyInfo.nonStructType.MapNameOffset)
      {
          ULONG ulMapSize = 0;
          mapName = (PWSTR)((BYTE*)pEventInfo +
            propertyInfo.nonStructType.MapNameOffset);

          dwStatus = TdhGetEventMapInformation(
                       pRecord,
                       mapName,
                       pMapInfo,
                       &ulMapSize);

          if (dwStatus == ERROR_INSUFFICIENT_BUFFER)
          {
            pMapInfo = (PEVENT_MAP_INFO)malloc(ulMapSize);

          ❹ dwStatus = TdhGetEventMapInformation(
                         pRecord,
                         mapName,
                         pMapInfo,
                         &ulMapSize);
        if (dwStatus != ERROR_SUCCESS)
        {
            pMapInfo = NULL;
        }
      }
    }
    `--snip--`
}
```

列表 8-21: 解析事件映射信息

为了解析提供者发出的事件，我们通过使用在跟踪事件信息结构中找到的属性总数 TopLevelPropertyCount，遍历事件中的每个顶级属性❶。然后，如果我们不是在处理结构❷，并且成员名称的偏移量存在❸，我们将偏移量传递给 tdh!TdhGetEventMapInformation()❹，以获取事件映射信息。

到此为止，我们已经收集了完全解析事件数据所需的所有信息。接下来，我们调用 tdh!TdhFormatProperty()，并传入我们之前收集的信息。列表 8-22 显示了该函数的实际应用。

```
void CALLBACK OnEvent(PEVENT_RECORD pRecord)
{
    `--snip--`

    ULONG ulBufferSize = sizeof(pszValue);
    USHORT wSizeConsumed = 0;

    dwStatus = TdhFormatProperty(
                pEventInfo,
                pMapInfo, ulPointerSize,
                propertyInfo.nonStructType.InType,
                propertyInfo.nonStructType.OutType,
                wPropertyLen,
                wUserDataLen,
                pUserData,
                &ulBufferSize,
              ❶ pszValue,
                &wSizeConsumed);

    if (dwStatus == ERROR_SUCCESS)
    {
      `--snip--`

      wprintf(L"%s: %s\n", ❷ pszPropertyName, pszValue);

      `--snip--`
    }

    `--snip--`
}
```

列表 8-22: 使用 tdh!TdhFormatProperty() 检索事件数据

函数完成后，属性的名称（如键值对中的 *key* 部分）将存储在事件映射信息结构的 NameOffset 成员中（我们已将其存储在 pszPropertyName 变量中 ❷，为了简洁起见）。其值将存储在传递给 tdh!TdhFormatProperty() 的缓冲区中，作为 Buffer 参数 ❶（在我们的示例中是 pszValue）。

### 测试消费者

清单 8-23 中展示的代码来自我们的 .NET 事件消费者。它显示了 Seatbelt 侦察工具通过命令与控制代理加载到内存中的程序集加载事件。

```
AssemblyID: 0x266B1031DC0
AppDomainID: 0x26696BBA650
BindingID: 0x0
AssemblyFlags: 0
FullyQualifiedAssemblyName: Seatbelt, Version=1.0.0.0, `--snip--`
ClrInstanceID: 10
```

清单 8-23：Microsoft-Windows-DotNETRuntime 提供者的消费者检测到 Seatbelt 被加载

从这里开始，代理可以根据需要使用这些值。例如，如果代理想要终止加载 Seatbelt 程序集的任何进程，它可以利用这个事件来触发预防性操作。或者，如果想采取更为被动的措施，它可以将从这个事件收集到的信息，结合关于源进程的其他信息，创建自己的事件并将其输入到检测逻辑中。

## 规避基于 ETW 的检测

如我们所示，ETW 是从系统组件收集信息的一种非常有用的方法，否则这些信息是无法获取的。然而，这项技术也有其局限性。由于 ETW 是为监控或调试而设计的，而不是作为关键的安全组件，因此其保护机制不如其他传感器组件那样强大。

在 2021 年，Claudiu Teodorescu、Igor Korkin 和 Andrey Golchikov（来自 Binarly）在 Black Hat Europe 上进行了精彩的演讲，他们对现有的 ETW 规避技术进行了分类，并介绍了新的技术。他们的演讲确定了 36 种绕过 ETW 提供者和跟踪会话的独特策略。演讲者将这些技术分为五大类：来自攻击者控制的进程的攻击；对 ETW 环境变量、注册表和文件的攻击；对用户模式 ETW 提供者的攻击；对内核模式 ETW 提供者的攻击；以及对 ETW 会话的攻击。

这些技术在其他方面也有所重叠。此外，尽管一些技术适用于大多数提供者，另一些则针对特定提供者或跟踪会话。几种技术也在 Palantir 的博客文章《篡改 Windows 事件跟踪：背景、攻击与防御》中进行了讨论。为了总结这两组的发现，本节将这些规避技术分为更广泛的类别，并讨论每种方法的优缺点。

### 修补

可以说，在攻击领域中，绕过 ETW 的最常见技术是修补关键功能、结构和其他内存中与事件发射相关的地方。这些修补程序的目的是完全阻止提供者发射事件，或有选择地过滤它发送的事件。

你最常见到的这种修补方式是函数钩取，但攻击者也可以篡改许多其他组件来改变事件流。例如，攻击者可以将提供者使用的TRACEHANDLE置为无效，或者修改其TraceLevel，以防止某些类型的事件被发射。在内核中，攻击者还可以修改如ETW_REG_ENTRY这样的结构，这是内核中表示事件注册对象的方式。我们将在“绕过.NET 消费者”一节中更详细地讨论这一技术，见第 166 页。

### 配置修改

另一种常见的技术涉及修改系统的持久属性，包括注册表键、文件和环境变量。许多程序都属于这一类，但它们的共同目标通常是通过滥用类似注册表中的“关闭”开关来防止跟踪会话或提供者按预期功能工作。

“关闭”开关的两个例子是COMPlus_ETWEnabled环境变量和*HKCU:\Software\Microsoft\.NETFramework*注册表键下的ETWEnabled值。通过将这两个值中的任何一个设置为0，攻击者可以指示*clr.dll*，即 Microsoft-Windows-DotNETRuntime 提供者的镜像，不注册任何TRACEHANDLE，从而防止该提供者发射 ETW 事件。

### 跟踪会话篡改

下一个技术涉及干扰系统上已经运行的跟踪会话。虽然这通常需要系统级的权限，但已经提升权限的攻击者可以与他们不是显式拥有者的跟踪会话进行交互。例如，攻击者可以使用sechost!EnableTraceEx2()，或者更简单地使用 logman 和以下语法，来从跟踪会话中移除提供者：

```
logman.exe update trace `TRACE_NAME` --p `PROVIDER_NAME` --ets
```

更直接地说，攻击者可能选择完全停止跟踪：

```
logman.exe stop "`TRACE_NAME`" -ets
```

### 跟踪会话干扰

最后一个技巧是对前一个技巧的补充：它侧重于在跟踪会话开始之前，防止自动记录器等跟踪会话按预期工作，从而对系统进行持久性更改。

这种技术的一个例子是通过修改注册表手动从自动记录器会话中移除提供者。通过删除与提供者相关的子项 *HKLM:\SYSTEM\CurrentControlSet\Control\WMI\Autologger\<AUTOLOGGER_NAME>\<PROVIDER_GUID>*，或者将其 Enabled 值设置为 0，攻击者可以在下一次重启后从跟踪会话中移除提供者。

攻击者还可以利用 ETW 的机制来阻止会话按预期工作。例如，每个主机每次只能启用一个遗留提供者（如 MOF 或 TMF 基于 WPP）。如果新会话启用了该提供者，原始会话将不再接收到所需的事件。同样，攻击者可以在安全产品有机会启动会话之前，创建一个与目标同名的跟踪会话。当代理尝试启动其会话时，它将遇到 ERROR_ALREADY_EXISTS 错误代码。

## 绕过 .NET 消费者

让我们通过瞄准一个类似本章早些时候编写的 .NET 运行时消费者来练习规避基于 ETW 的遥测源。在他的博客文章《隐藏你的 .NET—ETW》中，Adam Chester 介绍了如何阻止公共语言运行时发出 ETW 事件，从而使传感器无法识别 SharpHound 的加载。SharpHound 是一个 C# 工具，用于收集将输入到路径映射攻击工具 BloodHound 中的数据。

绕过技术通过修补负责发出 ETW 事件的函数 ntdll!EtwEventWrite() 来实现，并指示该函数在进入时立即返回。Chester 发现，通过在 WinDbg 中设置断点并观察来自 *clr.dll* 的调用，最终发现该函数负责发出该事件。设置此条件断点的语法如下：

```
bp ntdll!EtwEventWrite "r $t0 = 0;
  .foreach (p {k}) {.if ($spat(\"p\", \"clr!*\")) {r $t0 = 1; .break}};
  .if($t0 = 0) {gc}"
```

该命令中的条件逻辑指示 WinDbg 解析调用堆栈（k）并检查每一行输出。如果某些行以 clr! 开头，表示对 ntdll!EtwEventWrite() 的调用来源于公共语言运行时，则触发断点。如果调用堆栈中没有这个子字符串的实例，应用程序将继续执行。

如果我们查看检测到子字符串时的调用堆栈，如 Listing 8-24 所示，我们可以观察到公共语言运行时发出了事件。

```
 0:000> **k**
  # RetAddr                Call Site
❶ 00 ntdll!EtwEventWrite
  01 clr!CoTemplate_xxxqzh+0xd5
  02 clr!ETW::LoaderLog::SendAssemblyEvent+0x1cd
❷ 03 clr!ETW::LoaderLog::ModuleLoad+0x155
  04 clr!DomainAssembly::DeliverSyncEvents+0x29
  05 clr!DomainFile::DoIncrementalLoad+0xd9
  06 clr!AppDomain::TryIncrementalLoad+0x135
  07 clr!AppDomain::LoadDomainFile+0x149
  08 clr!AppDomain::LoadDomainAssemblyInternal+0x23e
  09 clr!AppDomain::LoadDomainAssembly+0xd9
  0a clr!AssemblyNative::GetPostPolicyAssembly+0x4dd
  0b clr!AssemblyNative::LoadFromBuffer+0x702
  0c clr!AssemblyNative::LoadImage+0x1ef
❸ 0d mscorlib_ni!System.AppDomain.Load(Byte[])$ 60007DB+0x3b
  0e mscorlib_ni!DomainNeutralILStubClass.IL_STUB_CLRtoCOM(Byte[])
  0f clr!COMToCLRDispatchHelper+0x39
  10 clr!COMToCLRWorker+0x1b4
  11 clr!GenericComCallStub+0x57
  12 0x00000209`24af19a6
  13 0x00000209`243a0020
  14 0x00000209`24a7f390
  15 0x000000c2`29fcf950
```

Listing 8-24：一个简略的调用堆栈，显示在公共语言运行时中生成 ETW 事件

从下往上阅读，我们可以看到事件源自 System.AppDomain.Load()，这是负责将程序集加载到当前应用程序域中的函数 ❸。一连串内部调用最终进入 ETW::Loaderlog 类 ❷，该类最终调用 ntdll!EtwEventWrite() ❶。

尽管微软并不希望开发人员直接调用此函数，但该实践是有文档记录的。此函数预计会返回一个 Win32 错误代码。因此，如果我们可以手动将 EAX 寄存器中的值（它作为 Windows 上的返回值）设置为 0（表示 ERROR_SUCCESS），函数应该会立即返回，表现得总是成功完成，而不会生成事件。

修补此函数是一个相对简单的四步过程。我们在 Listing 8-25 中深入了解这一过程。

```
#define WIN32_LEAN_AND_MEAN
#include <Windows.h>

void PatchedAssemblyLoader()
{
    PVOID pfnEtwEventWrite = NULL;
    DWORD dwOldProtection = 0;

 ❶ pfnEtwEventWrite = GetProcAddress(
      LoadLibraryW(L"ntdll"),
      "EtwEventWrite"
    );

    if (!pfnEtwEventWrite)
    {
        return;
    }

 ❷ VirtualProtect(
      pfnEtwEventWrite,
      3,
      PAGE_READWRITE,
      &dwOldProtection
      );

 ❸ memcpy(
      pfnEtwEventWrite,
      "\x33\xc0\xc3", // xor eax, eax; ret
      3
      );

 ❹ VirtualProtect(
      pfnEtwEventWrite,
      3,
      dwOldProtection,
      NULL
      );

      `--snip--`
}
```

Listing 8-25：修补 ntdll!EtwEventWrite() 函数

我们通过 kernel32!GetProcAddress() ❶ 在当前加载的 *ntdll.dll* 中定位 ntdll!EtwEventWrite() 的入口点。定位到函数后，我们将前 3 个字节（即我们的补丁大小）的内存保护从只读执行（rx）更改为读写（rw） ❷，以便我们能够覆盖入口点。现在，我们只需要使用像 memcpy() ❸ 之类的函数复制补丁，然后将内存保护恢复到原始状态 ❹。此时，我们可以执行我们的汇编加载器功能，而不必担心生成公共语言运行时加载器事件。

我们可以使用 WinDbg 来验证 ntdll!EtwEventWrite() 不再生成事件，如 Listing 8-26 所示。

```
0:000> **u ntdll!EtwEventWrite**
ntdll!EtwEventWrite:
00007ff8`7e8bf1a0 33c0         xor     eax,eax
00007ff8`7e8bf1a2 c3           ret
00007ff8`7e8bf1a3 4883ec58     sub     rsp,58h
00007ff8`7e8bf1a7 4d894be8     mov     qword ptr [r11-18h],r9
00007ff8`7e8bf1ab 33c0         xor     eax,eax
00007ff8`7e8bf1ad 458943e0     mov     dword ptr [r11-20h],r8d
00007ff8`7e8bf1b1 4533c9       xor     r9d,r9d
00007ff8`7e8bf1b4 498943d8     mov      qword ptr [r11-28h],rax
```

Listing 8-26：修补后的 ntdll!EtwEventWrite() 函数

当调用此函数时，它会立即通过将 EAX 寄存器设置为 0 来清除该寄存器，然后返回。这样可以防止生成 ETW 事件的逻辑被执行，从而有效地阻止提供程序的遥测数据流向 EDR 代理。

即便如此，这种绕过方式也有其局限性。因为*clr.dll*和*ntdll.dll*被映射到各自的进程中，它们能够以非常直接的方式篡改提供者。然而，在大多数情况下，提供者作为一个独立的进程运行，超出了攻击者的直接控制范围。修补映射的*ntdll.dll*中的事件发射功能并不能阻止其他进程中的事件发射。

在他的博客文章《普遍绕过 Sysmon 和 ETW》中，Dylan Halls 描述了一种不同的技术，防止 ETW 事件被发射，该技术涉及修补ntdll!NtTraceEvent()，这个系统调用最终会导致 ETW 事件在内核模式下被触发。这意味着，在修补程序生效期间，系统上通过该系统调用路由的任何 ETW 事件都不会被发射。这种技术依赖于使用内核驱动工具（KDU）来规避驱动程序签名强制执行，以及使用 InfinityHook 来降低 PatchGuard 检测到补丁后崩溃系统的风险。虽然这种技术扩展了绕过 ETW 检测的能力，但它需要加载驱动程序并修改受保护的内核模式代码，因此会受到 KDU 或 InfinityHook 所依赖的任何缓解技术的影响。

## 结论

ETW（事件跟踪 Windows）是收集 Windows 主机基础遥测数据的最重要技术之一。它为 EDR（端点检测与响应）提供对组件和进程的可视性，比如任务调度器和本地 DNS 客户端，这些是其他传感器无法监控的。一个代理可以消费它找到的几乎所有提供者的事件，并使用这些信息来获得关于系统活动的大量上下文。绕过 ETW 的技术已经有很多研究，绝大多数策略集中在禁用、注销或以其他方式使提供者或消费者无法处理事件。
