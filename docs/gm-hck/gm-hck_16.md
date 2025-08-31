## 第十二章：12

**保持隐蔽**

![image](img/common.jpg)

游戏作弊是一个不断发展的行为，是黑客和游戏开发者之间的猫鼠游戏，双方都在努力颠覆对方。只要有人制造机器人，游戏公司就会找到阻碍机器人进展并封禁使用机器人玩家的方法。不过，游戏公司并不将游戏本身变得更难破解，而是专注于*检测*。

最大的游戏公司拥有非常复杂的检测套件，称为*反作弊软件*。在本章的开始部分，我将讨论最常见的反作弊套件的功能。在揭示这些套件如何检测机器人后，我将教你一些强大的规避方法。

### 著名的反作弊软件

最著名的反作弊套件使用与大多数杀毒软件相同的方法来扫描机器人并将其标记为威胁。一些反作弊套件也是动态的，这意味着它们的内部工作原理和功能可以根据它们保护的游戏而有所变化。反作弊软件开发人员还会追踪并修补他们的套件，以应对绕过软件，因此在面对任何反作弊软件时，务必进行深入的自我研究。

当这些套件检测到机器人时，它们会将机器人的账户标记为待封禁。每隔几周，游戏公司管理员会在*封禁波*中封禁被标记的玩家。游戏公司使用封禁波而不是即时封禁，因为波次封禁更有利可图。如果机器人在几周的游戏后被封禁，由于他们对游戏的熟悉，可能更容易购买新账户，而不是在机器人开始运行的瞬间就被封禁。

有数十种反作弊套件，但我将重点讨论五个最常见且被彻底了解的套件：*PunkBuster*、*ESEA 反作弊*、*Valve 反作弊（VAC）*、*GameGuard*和*Warden*。

### PunkBuster 工具包

PunkBuster 由 Even Balance 公司制作，是最早的反作弊工具包。许多游戏使用 PunkBuster，但它在第一人称射击游戏中最为常见，如*荣誉勋章*、*孤岛惊魂 3*和*战地*系列的多个版本。

这个工具包使用了多种检测方法，其中最强大的包括基于签名的检测（SBD）、截图和哈希验证。PunkBuster 还因实施硬件封禁而闻名，它通过保存硬件序列号的指纹来永久封禁作弊者的计算机，而不仅仅是他们的游戏账户，并阻止来自匹配该指纹的机器的登录。

#### *基于签名的检测*

PunkBuster 扫描运行该软件的系统上所有进程的内存，搜索已知作弊软件的独特字节模式，称为*签名*。如果 PunkBuster 检测到签名，玩家将被标记为待封禁。PunkBuster 使用`NtQueryVirtualMemory()` Windows API 函数从用户模式执行内存扫描，并有时从多个隐藏进程中运行扫描。

基于签名的检测方法从设计上是盲目的，最终会面临一个致命缺陷：误报。2008 年 3 月 23 日，一组黑客团队试图通过向公共聊天室发送一个 PunkBuster 会识别为机器人签名的文本字符串来证明这个缺陷的存在。由于 SBD 会盲目扫描进程内存中的匹配模式，所有在这些公共聊天室中的合法玩家都被标记为机器人玩家。

这导致成千上万的公平玩家被禁赛，且没有任何正当理由。2013 年 11 月，类似的情况再次发生：PunkBuster 错误地禁止了成千上万的玩家在 *Battlefield 4* 中的账号。那次，没有人试图证明某种观点；公司只是在软件中添加了一个错误的签名。

PunkBuster 通过恢复玩家账户解决了这两个问题，但这些事件显示出它的 SBD 在执行时有多么激进。然而，经过这些攻击之后，PunkBuster 的 SBD 通过仅在预定义的二进制偏移量上检查签名，减少了误报的数量。

#### *截图*

作为另一种机器人检测方法，PunkBuster 还会定期截图玩家的屏幕，并将截图发送到中央游戏服务器。这种检测方式很麻烦，而且与 SDB 相比效果较差。游戏作弊社区推测，PunkBuster 实施这个功能是为了给游戏管理员提供证据，以应对那些对禁令提出异议的机器人玩家。

#### *哈希验证*

除了采用 SBD 和截图，PunkBuster 还通过在玩家系统上创建游戏可执行文件的加密哈希，并将其与存储在中央服务器上的哈希进行比较来检测机器人。如果哈希不匹配，玩家将被标记为禁赛。这个检查只会在文件系统中的二进制文件上进行，而不会在内存中的二进制文件上进行。

### ESEA 反作弊工具包

ESEA 反作弊工具包被 *E-Sports Entertainment Association (ESEA)* 使用，主要用于其 *Counter-Strike: Global Offensive* 联赛。与 PunkBuster 不同，这个工具包以产生非常少的误报并且在抓取作弊者方面非常有效而闻名。

ESEA 反作弊的检测能力与 PunkBuster 相似，但有一个显著的不同。ESEA 反作弊的 SBD 算法是通过一个内核模式驱动程序执行的，使用了三个不同的 Windows 内核函数：`MmGetPhysicalMemoryRanges()` 函数、`ZwOpenSection()` 函数和 `ZwMapViewOfSection()` 函数。这个实现使得反作弊系统几乎免疫于内存伪造（这是一种常见的绕过 SBD 的方式），因为这些扫描函数在从驱动程序调用时更难被挂钩。

### VAC 工具包

VAC 是 Valve 公司应用于自家游戏和许多第三方游戏（通过其 Steam 游戏平台提供）的工具包。VAC 使用与 PunkBuster 检测技术相似的 SDB 和哈希验证方法，还使用了域名系统（DNS）缓存扫描和二进制验证。

#### *DNS 缓存扫描*

DNS 是一种平滑转换域名和 IP 地址的协议，DNS 缓存是存储这些信息的地方。当 VAC 的 SBD 算法检测到作弊软件时，VAC 会扫描玩家的 DNS 缓存，查找与作弊网站相关的任何域名。目前尚不确定是否需要进行 DNS 缓存扫描，才能让 VAC 的 SBD 算法标记玩家为封禁对象，或者 DNS 缓存扫描是否仅仅是对已经被 SBD 标记的玩家的进一步确认。

**注意**

*要查看您的 DNS 缓存，请在命令提示符下输入 **`ipconfig /displaydns`**。是的，VAC 会查看所有这些信息。*

#### *二进制验证*

VAC 还使用二进制验证来防止可执行二进制文件在内存中被篡改。它通过将内存中的二进制代码的哈希与文件系统中相同代码的哈希进行比较，扫描如 IAT、跳转和代码钩取等修改。如果发现不匹配，VAC 会标记该玩家为作弊并进行封禁。

这种检测方法非常强大，但 Valve 最初实施该算法时存在缺陷。2010 年 7 月，VAC 的二进制验证错误地封禁了 12,000 名 *使命召唤* 玩家。二进制验证模块未考虑到 Steam 更新，导致玩家的内存中的代码与文件系统中的更新二进制文件不匹配时被封禁。

#### *虚假正例*

VAC 也曾出现过虚假正例。其最初版本经常因为“内存故障”错误地封禁公正玩家。这个早期版本还因玩家使用 *Cedega*（一个在 Linux 上运行 Windows 游戏的平台）而封禁玩家。2004 年 4 月 1 日，Valve 由于服务器端故障错误地封禁了几千名玩家。在 2011 年 6 月和 2014 年 2 月的两次事件中，VAC 也因公司拒绝披露的漏洞错误地封禁了数千名 *Team Fortress 2* 和 *Counter-Strike* 玩家。与 PunkBuster 类似，这些事件表明 VAC 非常具有攻击性。

### GameGuard 工具包

GameGuard 是由 INCA Internet Co. Ltd. 开发的反作弊工具包，被许多 MMORPG 游戏使用，包括 *天堂 II*、*Cabal Online* 和 *仙境传说 Online*。除了使用一些较为激进的 SBD，GameGuard 还利用根工具包积极防止作弊软件运行。

#### *用户模式根工具包*

GameGuard 使用用户模式根工具包来拒绝机器人访问它们操作所需的 Windows API 函数。根工具包在函数的最低级入口点进行钩取，通常位于 *ntdll.dll*、*user32.dll* 和 *kernel32.dll* 中的未文档函数内。这些是 GameGuard 最常钩取的 API 函数，下面是 GameGuard 在每个被钩取函数中的行为：

`NtOpenProcess()` 阻止任何对被保护游戏的 `OpenProcess()` 尝试。

`NtProtectVirtualMemory()` 阻止任何 `VirtualProtect()` 或 `VirtualProtectEx()` 对游戏的尝试。

`NtReadVirtualMemory()` **和** `NtWriteVirtualMemory()` 阻止任何对游戏的 `ReadProcessMemory()` 和 `WriteProcessMemory()` 尝试。

`NtSuspendProcess()` **和** `NtSuspendThread()` 阻止任何试图暂停 GameGuard 的行为。

`NtTerminateProcess()` **和** `NtTerminateThread()` 阻止任何试图终止 GameGuard 的行为。

`PostMessage()`、`SendMessage()`**和** `SendInput()` 阻止任何试图向游戏发送程序化输入的行为。

`SetWindowsHookEx()` 阻止机器人全局拦截鼠标和键盘输入。

`CreateProcessInternal()` 自动检测并钩住新进程。

`GetProcAddress()`、`LoadLibraryEx()`**和** `MapViewOfFileEx()` 阻止任何向游戏或 GameGuard 注入库的尝试。

#### *内核模式 Rootkit*

GameGuard 还使用基于驱动的 rootkit 来防止在内核中工作的机器人。这个 rootkit 具有与其用户模式对等体相同的能力，它通过钩住 `ZwProtectVirtualMemory()`、`ZwReadVirtualMemory()`、`ZwWriteVirtualMemory()`、`SendInput()` 等函数来工作。

### Warden 工具包

Warden，是暴雪专门为其游戏开发的工具包，是我遇到的最先进的反机器人工具包。很难说 Warden 究竟做了什么，因为它在运行时下载动态代码。这些代码以编译后的 shellcode 形式交付，通常有两个主要职责：

• 检测机器人。

• 定期向游戏服务器发送心跳信号。发送的值不是预定义的，而是由某些检测代码的子集生成。

如果 Warden 无法完成第二个任务或发送错误的值，游戏服务器就会知道它已被禁用或篡改。此外，机器人无法禁用检测代码并保持心跳代码运行。

**停机问题**

一个能够禁用 Warden 检测代码并仍能发送心跳信号的机器人，将解决*停机问题*，该问题由艾伦·图灵于 1936 年证明是不可能解决的。停机问题是指通过一个通用算法判断一个程序是否会完成执行或永远运行下去。由于 Warden 使用相同的 shellcode 执行两个任务，编写一个通用算法只禁用其中一个任务是停机问题的一个变种：该算法无法确定哪些代码部分一定会执行，哪些不会，哪些部分负责执行每个任务。

Warden 很强大，因为你不仅无法知道自己在隐藏什么，还无法禁用这个工具包。即使你今天设法避免了检测，明天也可能会使用新的检测方法。

如果你计划公开分发机器人，你最终会遇到前面描述的某种反作弊解决方案——你必须战胜它。根据你机器人的足迹、游戏中的检测类型以及你的实现方式，躲避这些工具包的难度可能从微不足道到极其困难不等。

### 谨慎管理机器人足迹

机器人的*足迹*是指其拥有的独特、可检测的特征。例如，一个挂钩了 100 个函数的机器人通常比一个只挂钩 10 个函数的机器人更容易被检测到，因为前者对游戏代码的改动比后者多了一个数量级。由于目标检测系统只需要检测一个挂钩，前者机器人的开发者需要花费更多的时间确保机器人的所有挂钩都尽可能隐蔽。

另一个足迹特征是机器人的用户界面有多详细。如果一个已知的机器人有很多对话框，并且每个对话框都有特定的标题，游戏公司可以让其反作弊软件通过搜索具有这些标题的窗口来检测该机器人。同样的基本推理也适用于进程名称和文件名。

#### *最小化机器人的足迹*

根据你的机器人如何工作，有很多方法可以最小化它的足迹。例如，如果你的机器人大量依赖挂钩，你可以避免直接挂钩游戏的代码，而是专注于挂钩 Windows API 函数。Windows API 挂钩出乎意料地常见，因此开发者不能假设挂钩 Windows API 的程序就是机器人。

如果你的机器人有一个明确的用户界面，你可以通过去除所有窗口条、按钮等的字符串来掩盖界面。你可以改为显示带有文本的图像。如果你担心特定的进程名称或文件名会被反作弊软件检测到，可以使用通用的文件名，并让你的机器人每次启动时将自己复制到一个新的、随机的目录中。

#### *掩盖你的足迹*

最小化你的足迹是避免被检测的首选方法，但这不是必须的。你也可以对你的机器人进行混淆，使得任何人都更难理解它是如何工作的。混淆可以防止反机器人开发者试图检测你的机器人，也可以防止其他机器人开发者分析你的机器人以窃取专有功能。如果你出售你的机器人，混淆还可以防止别人破解它来绕过你的购买验证。

一种常见的混淆方式叫做*打包*。打包一个可执行文件会将其加密，并隐藏在另一个可执行文件中。当容器可执行文件启动时，打包的可执行文件会在内存中被解密并执行。打包后的机器人，分析其二进制文件以了解机器人做了什么几乎是不可能的，而且调试机器人进程也变得更加困难。一些常见的打包程序有*UPX*、*Armadillo*、*Themida*和*ASPack*。

#### *教会机器人检测调试器*

当反机器人开发者（或其他机器人的创建者）能够调试一个机器人时，他们可以搞清楚它是如何工作的，从而知道如何阻止它。如果有人正在积极试图拆解一个机器人，那么仅仅打包可执行文件可能不足以避开他们。为了防范这种情况，机器人通常采用*反调试*技术，当检测到调试器时，通过改变机器人的行为来混淆控制流。在这一节中，我将简要介绍一些检测调试器附加到机器人上的常用方法，接下来我将展示一些混淆的技巧。

##### 调用 CheckRemoteDebuggerPresent()

`CheckRemoteDebuggerPresent()` 是一个 Windows API 函数，可以告诉你当前进程是否附加了调试器。检测调试器的代码可能如下所示：

```
bool IsRemoteDebuggerPresent() {
    BOOL dbg = false;
    CheckRemoteDebuggerPresent(GetCurrentProcess(), &dbg);
    return dbg;
}
```

这个检查非常简单——它调用 `CheckRemoteDebuggerPresent()`，传入当前进程和指向 `dbg` 布尔值的指针。调用这个函数是检测调试器的最简单方法，但调试器也很容易规避。

##### 检查中断处理程序

*中断* 是处理器发送的信号，用于触发 Windows 内核中的相应处理程序。中断通常由硬件事件生成，但也可以通过使用 INT 汇编指令在软件中生成。内核允许一些中断——即中断 0x2D 和 0x03——触发用户模式的中断处理程序，形式为异常处理程序。你可以利用这些中断来检测调试器。

当调试器在某条指令上设置断点时，它会用断点指令（例如 INT 0x03）替换该指令。当中断被执行时，调试器通过异常处理程序得到通知，在那里它处理断点、替换原始代码，并允许应用程序无缝地恢复执行。当遇到无法识别的中断时，一些调试器甚至会悄无声息地跳过该中断，并允许执行正常继续，而不触发任何其他异常处理程序。

你可以通过故意在代码中的异常处理程序内生成中断来检测这种行为，正如示例 12-1 所示。

```
inline bool Has2DBreakpointHandler() {
    __try { __asm INT 0x2D }
    __except (EXCEPTION_EXECUTE_HANDLER){ return false; }
    return true;
}

inline bool Has03BreakpointHandler() {
    __try { __asm INT 0x03 }
    __except (EXCEPTION_EXECUTE_HANDLER){ return false; }
    return true;
}
```

*示例 12-1：检测中断处理程序*

在正常执行期间，这些中断会触发代码中围绕它们的异常处理程序。在调试会话中，一些调试器可能会拦截这些中断生成的异常并悄无声息地忽略它们，从而阻止周围的异常处理程序执行。因此，如果中断没有触发你的异常处理程序，那么说明存在调试器。

##### 检查硬件断点

调试器还可以使用处理器的调试寄存器设置断点；这些被称为*硬件断点*。调试器可以通过将指令的地址写入其中一个调试寄存器来在某条指令上设置硬件断点。

当一个存在于调试寄存器上的地址被执行时，调试器会收到通知。为了检测硬件断点（从而检测到调试器的存在），你可以像下面这样检查任意四个调试寄存器上的非零值：

```
bool HasHardwareBreakpoints() {
    CONTEXT ctx = {0};
    ctx.ContextFlags = CONTEXT_DEBUG_REGISTERS;
    auto hThread = GetCurrentThread();
    if(GetThreadContext(hThread, &ctx) == 0)
        return false;
    return (ctx.Dr0 != 0 || ctx.Dr1 != 0 || ctx.Dr2 != 0 || ctx.Dr3 != 0);
}
```

##### 打印调试字符串

`OutputDebugString()`是一个 Windows API 函数，可以用于将日志消息打印到调试器控制台。如果没有调试器，函数会返回一个错误代码。然而，如果有调试器，函数则会正常返回而没有错误代码。你可以用这个函数作为一个简单的调试器检查方法：

```
inline bool CanCallOutputDebugString() {
    SetLastError(0);
    OutputDebugStringA("test");
    return (GetLastError() == 0);
}
```

与`CheckRemoteDebuggerPresent()`方法类似，这个方法非常直接，但也很容易被调试器规避。

##### 检查 DBG_RIPEXCEPTION 处理程序

调试器通常有异常处理程序，会盲目捕捉 Windows 的`DBG_RIPEXCEPTION`异常代码，这使得此代码成为一个明显的检测调试器的方式。你可以像 Listing 12-1 检测中断处理程序一样，检测这些异常处理程序：

```
#define DBG_RIPEXCEPTION 0x40010007
inline bool hasRIPExceptionHandler() {
    __try { RaiseException(DBG_RIPEXCEPTION, 0, 0, 0); }
    __except(EXCEPTION_EXECUTE_HANDLER){ return false; }
    return true;
}
```

##### 控制关键代码段的时间

如果一个反机器人开发者正在调试你的机器人，开发者可能会在关键代码上设置断点，并单步执行。这种活动可以通过测量代码执行时间来检测；当某人单步执行代码时，执行时间会比平时长得多。

例如，如果一个函数仅仅是放置一些钩子，你可以确定该代码在进行内存保护时不会花费超过十分之一秒。你可以通过以下方式借助`GetTickCount()` Windows API 函数来检查内存保护的执行时间：

```
--snip--
auto startTime = GetTickCount();
protectMemory<>(...);
if (GetTickCount() - startTime >= 100)
    debuggerDetectedGoConfuseIt();
--snip--
```

##### 检查调试驱动程序

一些调试器会加载内核模式驱动程序来辅助其操作。你可以尝试通过获取它们的内核模式驱动程序句柄来检测这些调试器，像这样：

```
bool DebuggerDriversPresent() {
    // an array of common debugger driver device names
    const char drivers[9][20] = {
        "\\\\.\\EXTREM", "\\\\.\\ICEEXT",
        "\\\\.\\NDBGMSG.VXD", "\\\\.\\RING0",
        "\\\\.\\SIWVID", "\\\\.\\SYSER",
        "\\\\.\\TRW", "\\\\.\\SYSERBOOT",
        "\0"
    };
    for (int i = 0; drivers[i][0] != '\0'; i++) {
        auto h = CreateFileA(drivers[i], 0, 0, 0, OPEN_EXISTING, 0, 0);
        if (h != INVALID_HANDLE_VALUE) {
            CloseHandle(h);
            return true;
        }
    }
    return false;
}
```

有一些常见的内核模式驱动程序设备名称可以检查，比如`\\\\.\\EXTREM`和`drivers`数组中列出的其他名称。如果此句柄获取代码成功，那么系统上就运行着调试器。不过，与前面的方法不同，获取其中一个驱动程序的句柄并不总是意味着调试器已经附加到你的机器人上。

#### *反调试技巧*

一旦你检测到调试器，便有多种方式来混淆你的控制流。例如，你可能尝试让调试器崩溃。以下代码会使 OllyDbg v1.10 崩溃：

```
OutputDebugString("%s%s%s%s");
```

字符串`"%s%s%s%s"`包含格式说明符，而 OllyDbg 将其传递给`printf()`而不附加任何额外的参数，这就是为什么调试器会崩溃的原因。你可以将这段代码放入一个函数中，当检测到调试器时调用，但此方法只对 OllyDbg 有效。

##### 引发无法避免的无限循环

另一种尝试的混淆方法是使系统过载，直到调试你的机器人时，调试者不得不关闭机器人和调试器。这个函数可以达到这个效果：

```
void SelfDestruct() {
    std::vector<char*> explosion;
    while (true)
        explosion.push_back(new char[10000]);
}
```

无限的`while`循环不断向`explosion`中添加元素，直到进程耗尽内存或有人断开电源。

##### 栈溢出

如果你想真正让分析人员困惑，你可以创建一个函数链，最终导致栈溢出，但以间接的方式：

```
#include <random>
typedef void (* _recurse)();
void recurse1(); void recurse2();
void recurse3(); void recurse4();
void recurse5();
_recurse recfuncs[5] = {
    &recurse1, &recurse2, &recurse3,
    &recurse4, &recurse5
};
void recurse1() { recfuncs[rand() % 5](); }
void recurse2() { recfuncs[(rand() % 3) + 2](); }
void recurse3() {
    if (rand() % 100 < 50) recurse1();
    else recfuncs[(rand() % 3) + 1]();
}
void recurse4() { recfuncs[rand() % 2](); }
void recurse5() {
    for (int i = 0; i < 100; i++)
        if (rand() % 50 == 1)
            recfuncs[i % 5]();
    recurse5();
}
// call any of the above functions to trigger a stack overflow
```

简而言之，这些函数会随机且无限递归，直到调用栈没有空间为止。间接导致溢出使得分析人员很难在他们意识到发生了什么之前暂停并检查之前的调用。

##### 导致蓝屏死机（BSOD）

如果你认真对待混淆，你甚至可以在检测到调试器时触发蓝屏死机（BSOD）。一种方法是使用`SetProcessIsCritical()` Windows API 函数将你的机器人的进程设置为关键进程，然后调用`exit()`，因为当关键进程被终止时，Windows 会触发蓝屏死机。你可以这样做：

```
void BSODBaby() {
    typedef long (WINAPI *RtlSetProcessIsCritical)
        (BOOLEAN New, BOOLEAN *Old, BOOLEAN NeedScb);
    auto ntdll = LoadLibraryA("ntdll.dll");
    if (ntdll) {
        auto SetProcessIsCritical = (RtlSetProcessIsCritical)
            GetProcAddress(ntdll, "RtlSetProcessIsCritical");
        if (SetProcessIsCritical)
            SetProcessIsCritical(1, 0, 0);
    }
}

BSODBaby();
exit(1);
```

或者，也许你是个坏人，在这种情况下你可以这样做：

```
BSODBaby();
OutputDebugString("%s%s%s%s");
recurse1();
exit(1);
```

假设你已经实现了本节中描述的所有技巧，这段代码将导致蓝屏死机（BSOD），崩溃调试器（如果是 OllyDbg v1.10），溢出栈并退出正在运行的程序。如果任何一种方法失败或被修补，分析人员仍然需要处理剩下的方法，才能继续调试。

### 击败基于签名的检测

即使有令人惊叹的混淆技术，你也不容易打败签名检测。分析机器人并编写签名的工程师非常熟练，而混淆技术充其量只是让他们的工作变得稍微困难一些。

要完全躲避基于签名的检测，你需要颠覆检测代码。这要求你准确了解 SBD 的工作原理。例如，PunkBuster 使用`NtQueryVirtualMemory()`扫描所有正在运行的进程内存中的任何签名。如果你想绕过它，你可以通过在`NtQueryVirtualMemory()`函数上设置钩子，将代码注入所有 PunkBuster 进程。

当函数尝试从你的机器人进程查询内存时，你可以给它任何你想要的数据，像这样：

```
   NTSTATUS onNtQueryVirtualMemory(
       HANDLE process, PVOID baseAddress,
       MEMORY_INFORMATION_CLASS memoryInformationClass,
       PVOID buffer, ULONG numberOfBytes, PULONG numberOfBytesRead) {

       // if the scan is on this process, make sure it can't see the hook DLL
       if ((process == INVALID_HANDLE_VALUE ||
           process == GetCurrentProcess()) &&
           baseAddress >= MY_HOOK_DLL_BASE &&
           baseAddress <= MY_HOOK_DLL_BASE_PLUS_SIZE)
➊             return STATUS_ACCESS_DENIED;

       // if the scan is on the bot, zero the returned memory
       auto ret = origNtQueryVirtualMemory(
           process, baseAddress,
           memoryInformationClass,
           buffer, numberOfBytes, numberOfBytesRead);
       if(GetProcessId(process) == MY_BOT_PROCESS)
➋         ZeroMemory(buffer, numberOfBytesRead);
       return ret;
   }
```

这个`onNtQueryVirtualMemory()`钩子在`NtQueryVirtualMemory()`尝试查询钩子 DLL 的内存时返回`STATUS_ACCESS_DENIED` ➊，但是当`NtQueryVirtualMemory()`尝试查询机器人内存时，它返回零填充的内存 ➋。这种差异并没有特别的原因；我只是展示了两种可以躲避`NtQueryVirtualMemory()`函数调用的方式。如果你真的很疑心，你甚至可以用随机字节序列替换整个缓冲区。

当然，这种方法只对来自用户模式的基于签名的检测（SBD）有效，例如 PunkBuster 或 VAC 中的 SBD。来自驱动程序的 SBD，如 ESEA，或不可预测的 SBD，如 Warden 的，不容易绕过。

在这些情况下，你可以采取预防措施来消除你机器人中的独特标识符。然而，如果你将机器人分发给十几个人以上，去除所有的区分特征是非常棘手的。为了迷惑分析师，每次你给某人一个副本时，你可以尝试以下几种组合：

• 使用不同的编译器编译机器人

• 更改编译器优化设置

• 在使用`__fastcall`和`__cdecl`之间切换

• 使用不同的打包工具打包二进制文件

• 在静态链接和动态链接运行时库之间切换

改变这些元素会为每个用户创建不同的汇编代码，但通过这种方式，你可以生产的独特版本数量是有限的。超过某个点后，这种方法就无法满足需求了，最终，游戏公司将会为你的机器人每一个版本都创建特征。

除了混淆和代码变异外，几乎没有其他方法能击败高级的 SBD 机制。你可以将机器人实现为驱动程序，或者创建一个内核模式的 rootkit 来隐藏你的机器人，但即使是这些方法也并非万无一失。

**注意**

*本书没有涉及如何在驱动程序中实现机器人或创建 rootkit 来隐藏机器人，因为这两个主题都相当复杂。单单是 rootkit 开发就是一个已经被许多书籍详细讨论的主题。我推荐 Bill Blunden 的*《Rootkit Arsenal: Escape and Evasion in The Dark Corners of The System》*（Jones & Bartlett Learning，2009 年）。*

一些游戏黑客试图覆盖每一个细节，挂钩每个内存读取函数和整个文件系统 API，但仍然会被像 Warden 这样的系统抓到。事实上，我建议你避免与 Warden 和暴雪有任何接触。

### 击败截图

如果你遇到一种检测机制，利用截图作为额外证据来抓捕机器人使用者，那么你很幸运。绕过截图机制非常简单：不要让你的机器人被看到。

你可以通过保持最小的用户界面并且不对游戏客户端做出明显可区分的改动来规避这种类型的检测。如果你的机器人需要一个 HUD 或者其他独特的 UI 显示，别担心——你完全可以两者兼得。只要你能够拦截截图代码，你就可以在截图时隐藏你的“指纹”。

在某些版本的 PunkBuster 中，例如，Windows API 函数`GetSystemTimeAsFileTime()`会在截图拍摄前被调用。你可以在这个函数上设置钩子，以便快速隐藏你的 UI 几秒钟，确保它不会被看到：

```
void onGetSystemTimeAsFileTime(LPFILETIME systemTimeAsFileTime) {
    myBot->hideUI(2000); // hide UI for 2 seconds
    origGetSystemTimeAsFileTime(systemTimeAsFileTime);
}
```

只需使用在“重定向游戏执行的挂钩”中描述的技术钩住`GetSystemTimeAsFileTime()`函数，第 153 页上有详细说明，编写一个`hideUI()`函数，并在执行继续前调用该`hideUI()`函数。

### 击败二进制验证

击败二进制验证的方法很简单——不要在游戏特定的二进制文件中放置挂钩。Windows API 函数中的跳转挂钩和 IAT 挂钩非常常见，所以只要可能，尽量使用这些方法，而不是在游戏的二进制文件中使用跳转或近调用挂钩。在必须直接挂钩游戏代码的情况下，你可以通过拦截二进制扫描并伪造数据，使其匹配反作弊软件预期的数据，从而欺骗反作弊软件的二进制验证过程。

像 SBD 一样，二进制验证通常使用`NtQueryVirtualMemory()`来扫描内存。为了欺骗验证代码，从挂钩这个函数开始。然后，写一个像这样的函数，当`NtQueryVirtualMemory()`被调用时伪造数据：

```
NTSTATUS onNtQueryVirtualMemory(
    HANDLE process, PVOID baseAddress,
    MEMORY_INFORMATION_CLASS memoryInformationClass,
    PVOID buffer, ULONG numberOfBytes, PULONG numberOfBytesRead) {

    auto ret = origNtQueryVirtualMemory(
        process, baseAddress,
        memoryInformationClass,
        buffer, numberOfBytes, numberOfBytesRead);
    // place tricky code somewhere in here
    return ret;
}
```

在这个挂钩内，你需要监控任何对已被你的挂钩修改的内存进行的扫描。

**注意**

*这个示例假设机器人只有一个挂钩，并且以`HOOK_`为前缀的变量已经存在，并描述了挂钩替换的代码。*

列表 12-2 展示了一些扫描监控代码。

```
   // is the scan on the current process?
   bool currentProcess =
       process == INVALID_HANDLE_VALUE ||
       process == GetCurrentProcess();

   // is the hook in the memory range being scanned?
   auto endAddress = baseAddress + numberOfBytesRead - 1;
   bool containsHook =
       (HOOK_START_ADDRESS >= baseAddress &&
        HOOK_START_ADDRESS <= endAddress) ||
       (HOOK_END_ADDRESS >= baseAddress &&
        HOOK_END_ADDRESS <= endAddress);
➊ if (currentProcess && containsHook) {
       // hide the hook
   }
```

*列表 12-2：检查是否挂钩的内存正在被扫描*

当对已挂钩代码进行内存扫描时（这会使得`currentProcess`和`containsHook`同时变为`true`），`if()`语句内的代码➊会更新输出缓冲区，以反映原始代码。这意味着你必须知道挂钩代码在扫描块中的位置，考虑到该块可能只跨越挂钩代码的一个子集。

所以如果`baseAddress`标记了扫描开始的地址，`HOOK_START_ADDRESS`标记了修改后的代码开始的位置，`endAddress`标记了扫描结束的地址，`HOOK_END_ADDRESS`标记了修改后的代码结束的位置，你可以使用一些简单的数学计算来确定修改后的代码在缓冲区的哪些部分。你可以按照以下步骤操作，使用`writeStart`来存储修改代码在扫描缓冲区中的偏移量，使用`readStart`来存储扫描缓冲区相对于修改代码的偏移量，以防扫描缓冲区开始的位置在修改代码的中间：

```
int readStart, writeStart;
if (HOOK_START_ADDRESS >= baseAddress) {
    readStart = 0;
    writeStart = HOOK_START_ADDRESS - baseAddress;
} else {
    readStart = baseAddress - HOOK_START_ADDRESS;
    writeStart = baseAddress;
}

int readEnd;
if (HOOK_END_ADDRESS <= endAddress)
    readEnd = HOOK_LENGTH - readStart - 1;
else
    readEnd = endAddress – HOOK_START_ADDRESS;
```

一旦你知道需要替换多少字节、放置它们的位置以及从哪里获取它们，你可以通过三行代码来完成欺骗：

```
char* replaceBuffer = (char*)buffer;
for ( ; readStart <= readEnd; readStart++, writeStart++)
    replaceBuffer[writeStart] = HOOK_ORIG_DATA[readStart];
```

完全组装后的代码如下所示：

```
NTSTATUS onNtQueryVirtualMemory(
    HANDLE process, PVOID baseAddress,
    MEMORY_INFORMATION_CLASS memoryInformationClass,
    PVOID buffer, ULONG numberOfBytes, PULONG numberOfBytesRead) {
    auto ret = origNtQueryVirtualMemory(
        process, baseAddress,
        memoryInformationClass,
        buffer, numberOfBytes, numberOfBytesRead);
    bool currentProcess =
        process == INVALID_HANDLE_VALUE ||
        process == GetCurrentProcess();
    auto endAddress = baseAddress + numberOfBytesRead - 1;
    bool containsHook =
        (HOOK_START_ADDRESS >= baseAddress &&
         HOOK_START_ADDRESS <= endAddress) ||
        (HOOK_END_ADDRESS >= baseAddress &&
         HOOK_END_ADDRESS <= endAddress);
    if (currentProcess && containsHook) {
        int readStart, writeStart;
        if (HOOK_START_ADDRESS >= baseAddress) {
            readStart = 0;
            writeStart = HOOK_START_ADDRESS - baseAddress;
        } else {
            readStart = baseAddress - HOOK_START_ADDRESS;
            writeStart = baseAddress;
        }

        int readEnd;
        if (HOOK_END_ADDRESS <= endAddress)
            readEnd = HOOK_LENGTH - readStart - 1;
        else
            readEnd = endAddress – HOOK_START_ADDRESS;

        char* replaceBuffer = (char*)buffer;
        for ( ; readStart <= readEnd; readStart++, writeStart++)
            replaceBuffer[writeStart] = HOOK_ORIG_DATA[readStart];
    }
    return ret;
}
```

当然，如果你有多个挂钩需要隐藏免受二进制验证扫描的影响，你需要以更健壮的方式实现此功能，以便能够相应地跟踪多个修改过的代码区域。

### 击败反作弊 Rootkit

GameGuard 和一些其他反作弊套件带有用户模式的 Rootkit，这些 Rootkit 不仅能检测机器人程序，还能主动防止它们运行。为了击败这种保护方式，你不必跳出框框思考，你可以完全复制这个框，并在这个副本内进行工作。

例如，如果你想向游戏写入内存，必须调用由*kernel32.dll*导出的`WriteProcessMemory()`函数。当你调用这个函数时，它会直接调用*ntdll.dll*中的`NtWriteVirtualMemory()`函数。GameGuard 会钩住`ntdll.NtWriteVirtualMemory()`函数，防止你写入内存。但如果`NtWriteVirtualMemory()`从另一个文件，如*ntdll_copy.dll*中导出，GameGuard 就无法钩住这个函数。

这意味着你可以复制*ntdll.dll*并动态导入所有需要的函数，如下所示：

```
// copy and load ntdll
copyFile("ntdll.dll", "ntdll_copy.dll");
auto module = LoadLibrary("ntdll_copy.dll");

// dynamically import NtWriteVirtualMemory
typedef NTSTATUS (WINAPI* _NtWriteVirtualMemory)
    (HANDLE, PVOID, PVOID, ULONG, PULONG);
auto myWriteVirtualMemory = (_NtWriteVirtualMemory)
    GetProcAddress(module, "NtWriteVirtualMemory");

// call NtWriteVirtualMemory
myWriteVirtualMemory(process, address, data, length, &writtenlength);
```

复制*ntdll.dll*后，这段代码从复制的文件中导入`NtWriteVirtualMemory()`，并将其命名为`myWriteVirtualMemory()`。从此，机器人可以使用这个函数来替代`NtWriteVirtualMemory()`函数。它们实际上是相同的代码，位于相同的库中，只是以不同的名称加载。

复制一个被反作弊软件钩住的函数，只能在你以最低级别的入口点调用该函数时有效。如果这段代码复制了*kernel32.dll*并动态导入了`WriteProcessMemory()`函数，反作弊根套件依然会阻止机器人，因为*kernel32_copy.dll*在调用`WriteProcessMemory()`时仍然依赖于`ntdll.NtWriteVirtualMemory()`。

### 击败启发式算法

除了我们刚才讨论的所有先进的客户端检测机制，游戏公司还会采用服务器端的启发式算法，通过监控玩家的行为来检测机器人。这些系统通过机器学习算法学会区分人类玩家和自动化玩家的行为。它们的决策过程通常是内部的，人类难以理解，因此很难确切指出哪些游戏特征会导致被检测出来。

你不需要了解这些算法如何工作来欺骗它们；你的机器人只需要表现得像人类。以下是一些常见的行为模式，它们在人类和机器人之间有明显的区别：

**操作之间的间隔**

许多机器人执行操作的速度异常快，或者按照固定的间隔进行。机器人如果在操作之间有合理的冷却时间，它们看起来会更加像人类。机器人还应具备某种随机化机制，以防止它们以固定的频率重复执行某个操作。

**路径重复**

自动刷怪的机器人会访问一个预先编程的地点列表，去击杀怪物。这些路径列表通常非常精确，将每个位置标记为一个精确的像素。相比之下，人类玩家的移动方式较不规则，会沿着熟悉的区域访问一些更加独特的地方。为了模拟这种行为，机器人可能会走到目标地点的某个范围内的随机位置，而不是直接到达目标位置。而且，如果机器人随机化访问目标地点的顺序，它所走的路径种类将会进一步增加。

**不真实的游戏方式**

一些机器人使用者会让他们的机器人在同一个位置运行数百小时，但人类不可能连续玩这么长时间。建议你的用户避免一次使用机器人超过八小时，并警告他们，如果连续七天做同样的事情，肯定会在启发式系统中触发警报。

**完美的准确度**

机器人可以连续打出一千个爆头，不打多余的一发子弹，且能稳定地命中每一个技能射击。但对于人类来说，几乎不可能做到这一点，所以一个聪明的机器人有时应该故意不那么精准。

这些只是一些例子，但一般来说，只要你运用常识，你就能绕过启发式检测。不要让机器人做出人类无法做到的事情，也不要让机器人做某一件事做得太久。

### 结束语

游戏黑客和游戏开发者之间一直在进行着智力的较量。黑客会不断寻找规避检测的方法，而开发者则会不断寻找更好的检测方式。然而，如果你决心要胜利，本章的知识应该能帮助你击败你遇到的任何反作弊软件。
