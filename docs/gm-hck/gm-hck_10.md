## 第七章：7

**代码注入**

![image](img/common.jpg)

想象一下，你能够走进一家游戏公司的办公室，坐下来，开始向他们的游戏客户端添加代码。想象一下，你可以在任何你想要的时间、为任何你想要的游戏、添加任何你想要的功能。几乎所有你交谈过的玩家都会有改进游戏的想法，但在他们看来，这只是一个空想。然而，你知道梦想是可以实现的，现在你已经学会了一些关于内存如何工作的知识，你准备好开始抛弃规则。通过代码注入，你实际上可以变得和任何游戏的开发者一样强大。

*代码注入*是强制任何进程在其自己的内存空间和执行上下文中执行外部代码的一种方法。我之前在《绕过生产环境中的 ASLR》中提到过这个话题，位于第 128 页，在那里我向你展示了如何通过`CreateRemoteThread()`远程绕过 ASLR，但那个例子只是触及了表面。在本章的第一部分，你将学习如何创建代码洞、注入新线程、并劫持线程执行，强制游戏执行小段的汇编代码。在第二部分，你将学习如何直接将外部二进制文件注入游戏，迫使游戏执行你创建的整个程序。

### 通过线程注入注入代码洞

向另一个进程注入代码的第一步是编写位置无关的汇编代码，通常称为*shellcode*，其形式为字节数组。你可以将 shellcode 写入远程进程中，形成*代码洞*，它们作为你希望游戏执行的新线程的入口点。一旦创建了代码洞，你可以通过*线程注入*或*线程劫持*来执行它。在本节中，我将展示一个线程注入的例子，线程劫持的例子将在《劫持游戏的主线程以执行代码洞》中讲解，详见第 138 页。

你可以在本书的资源文件中找到本章的示例代码，位于目录*GameHackingExamples/Chapter7_CodeInjection*。打开*main-codeInjection.cpp*，跟随我一起讲解如何构建该文件中简化版的`injectCodeUsingThreadInjection()`函数。

#### *创建汇编代码洞*

在《绕过生产环境中的 ASLR》中，位于第 128 页，我使用线程注入通过`CreateRemoteThread()`调用了`GetModuleHandle()`函数并获取了进程句柄。在那种情况下，`GetModuleHandle()`充当了代码洞；它具有合适的代码结构，可以作为新线程的入口点。不过，线程注入并不总是这么简单。

举个例子，假设你希望你的外部机器人远程调用游戏中的一个函数，并且该函数具有以下原型：

```
DWORD __cdecl someFunction(int times, const char* string);
```

有几个因素使得远程调用这个函数变得复杂。首先，它有两个参数，这意味着你需要创建一个代码洞来设置堆栈并正确地进行调用。`CreateRemoteThread()`允许你将一个参数传递给代码洞，你可以相对于`ESP`访问该参数，但另一个参数仍然需要硬编码到代码洞中。硬编码第一个参数`times`是最简单的。此外，你还需要确保代码洞能够正确清理堆栈。

**注意**

*回想一下，在绕过第六章的 ASLR 时，我使用了*`CreateRemoteThread()`*来通过在给定地址执行任意代码并传递一个参数来启动新线程。这就是为什么这些示例可以通过堆栈传递一个参数的原因。*

最终，将调用`someFunction`注入到正在运行的游戏进程中的代码洞将类似于以下伪代码：

```
PUSH DWORD PTR:[ESP+0x4] // get second arg from stack
PUSH times 
CALL someFunction 
ADD ESP, 0x8 
RETN
```

这个代码洞几乎是完美的，但它可以更简单一些。`CALL`操作需要两个操作数中的一个：要么是包含绝对函数地址的寄存器，要么是包含相对返回地址的函数偏移量的立即数。这意味着你需要做一堆偏移量计算，这会非常繁琐。

为了使代码洞的位置无关，修改它以改用寄存器，如列表 7-1 所示。

```
PUSH DWORD PTR:[ESP+0x4] // get second arg from stack
PUSH times 
MOV EAX, someFunction
CALL EAX 
ADD ESP, 0x8 
RETN
```

*列表 7-1：调用`someFunction`的代码洞*

由于调用者知道它调用的函数会用返回值覆盖`EAX`，调用者应该确保`EAX`不包含任何重要数据。了解这一点后，你可以使用`EAX`来存储`someFunction`的绝对地址。

#### *将汇编代码转换为 Shellcode*

由于代码洞需要写入另一个进程的内存，它们不能直接用汇编语言编写。相反，你需要逐字节编写它们。没有标准的方法来确定哪些字节表示哪些汇编代码，但有一些巧妙的方法。我个人最喜欢的是将包含汇编代码的空 C++应用程序编译出来，然后使用 OllyDbg 检查该函数。或者，你可以在任何任意进程中打开 OllyDbg，扫描反汇编代码，直到找到你需要的所有操作的字节。这种方法实际上非常好，因为你的代码洞应该尽可能简单地编写，这意味着所有操作应该是非常常见的。你也可以在网上找到汇编操作码的图表，但我发现它们都很难阅读；我刚才描述的方法总体上更容易。

当你知道字节应该是什么时，你可以使用 C++轻松生成正确的 shellcode。列表 7-2 展示了列表 7-1 中的汇编代码的最终 shellcode 骨架。

```
 BYTE codeCave[20] = {
    0xFF, 0x74, 0x24, 0x04,       // PUSH DWORD PTR:[ESP+0x4]
    0x68, 0x00, 0x00, 0x00, 0x00, // PUSH 0
    0xB8, 0x00, 0x00, 0x00, 0x00, // MOV EAX, 0x0
    0xFF, 0xD0,                   // CALL EAX
    0x83, 0xC4, 0x08,             // ADD ESP, 0x08
    0xC3                          // RETN
};
```

*列表 7-2：Shellcode 骨架*

这个例子创建了一个`BYTE`数组，包含所需的 shellcode 字节。但是，`times`参数需要动态处理，而且在编译时无法知道`someFunction`的地址，这也是为什么这个 shellcode 是作为骨架编写的原因。两组四个连续的 0x00 字节是`times`和`someFunction`地址的占位符，你可以通过在运行时调用`memcpy()`将实际的值插入到代码洞中，正如清单 7-3 中的代码片段所示。

```
memcpy(&codeCave[5], &times, 4);
memcpy(&codeCave[10], &addressOfSomeFunc, 4);
```

*清单 7-3：将`times`和`someFunction`的位置插入到代码洞中*

`times`和`someFunction`的地址各占 4 个字节（回想一下，`times`是`int`类型，地址是 32 位值），它们分别位于`codeCave[5-8]`和`codeCave[10-13]`。两次调用`memcpy()`将这些信息作为参数传递，以填补`codeCave`数组中的空白。

#### *将代码洞写入内存*

在创建了合适的 shellcode 之后，你可以通过`VirtualAllocEx()`和`WriteProcessMemory()`将它放入目标进程中。清单 7-4 展示了实现这一点的一种方法。

```
   int stringlen = strlen(string) + 1; // +1 to include null terminator
   int cavelen = sizeof(codeCave);
➊ int fulllen = stringlen + cavelen;
   auto remoteString = // allocate the memory with EXECUTE rights
➋      VirtualAllocEx(process, 0, fulllen, MEM_COMMIT, PAGE_EXECUTE);

   auto remoteCave = // keep a note of where the code cave will go
➌      (LPVOID)((DWORD)remoteString + stringlen);

   // write the string first
➍ WriteProcessMemory(process, remoteString, string, stringlen, NULL);

   // write the code cave next
➎ WriteProcessMemory(process, remoteCave, codeCave, cavelen, NULL);
```

*清单 7-4：将最终的 shellcode 写入代码洞内存*

首先，这段代码确定了它需要多少字节的内存来将`string`参数和代码洞写入游戏的内存，并将该值存储在`fulllen` ➊中。接着，它调用 API 函数`VirtualAllocEx()`来分配`fulllen`字节的内存到`process`中，并使用`PAGE_EXECUTE`保护（你总是可以将第二个和第四个参数分别设置为`0`和`MEM_COMMIT`），并将该内存的地址存储在`remoteString` ➋中。它还将`remoteString`地址加上`stringlen`字节，并将结果存储在`remoteCave` ➌中，因为 shellcode 应该直接写入紧随`string`参数后的内存。最后，它使用`WriteProcessMemory()`将`string` ➍和存储在`codeCave`中的汇编字节 ➎填充到分配的缓冲区中。

表 7-1 展示了代码洞的内存转储可能的样子，假设它被分配在 0x030000，`someFunction`位于 0xDEADBEEF，`times`被设置为`5`，而`string`指向`injected!`文本。

**表 7-1：** 代码洞内存转储

| **地址** | **代码表示** | **原始数据** | **数据含义** |
| --- | --- | --- | --- |
| 0x030000 | `remoteString[0-4]` | 0x69 0x6E 0x6A 0x65 0x63 | `injec` |
| 0x030005 | `remoteString[5-9]` | 0x74 0x65 0x64 0x0A 0x00 | `ted!\0` |
| 0x03000A | `remoteCave[0-3]` | 0xFF 0x74 0x24 0x04 | `PUSH DWORD` `PTR[ESP+0x4]` |
| 0x03000E | `remoteCave[4-8]` | 0x68 0x05 0x00 0x00 0x00 | `PUSH 0x05` |
| 0x030013 | `remoteCave[9-13]` | 0xB8 0xEF 0xBE 0xAD 0xDE | `MOV EAX, 0xDEADBEEF` |
| 0x030018 | `remoteCave[14-15]` | 0xFF 0xD0 | `CALL EAX` |
| 0x03001A | `remoteCave[16-18]` | 0x83 0xC4 0x08 | `ADD ESP, 0x08` |
| 0x03001D | `remoteCave[19]` | 0xC3 | `RETN` |

地址列显示每个代码洞部分在内存中的位置；代码表示列告诉你`remoteString`和`remoteCave`的哪些索引对应原始数据列中的字节；数据意义列以人类可读的格式显示字节的含义。你可以看到 0x030000 处的`injected!`字符串，0x03000E 处的`times`值，以及 0x030014 处的`someFunction`地址。

#### *使用线程注入执行代码洞*

在内存中写入完整的代码洞后，剩下的唯一任务就是执行它。在这个例子中，你可以使用以下代码来执行代码洞：

```
HANDLE thread = CreateRemoteThread(process, NULL, NULL,
                    (LPTHREAD_START_ROUTINE)remoteCave,
                    remoteString, NULL, NULL);
 WaitForSingleObject(thread, INFINITE);
CloseHandle(thread);
VirtualFreeEx(process, remoteString, fulllen, MEM_RELEASE)
```

调用`CreateRemoteThread()`、`WaitForSingleObject()`和`CloseHandle()`可以注入并执行代码洞，`VirtalFreeEx()`通过释放代码中分配的内存（如示例 7-4 所示）来掩盖机器人的痕迹。最简单的形式就是执行注入到游戏中的代码洞。实际上，你还应在调用`VirtualAllocEx()`、`WriteProcessMemory()`和`CreateRemoteThread()`后检查返回值，以确保一切顺利。

例如，如果`VirtualAllocEx()`返回 0x00000000，意味着内存分配失败。如果你不处理这个失败，`WriteProcessMemory()`也会失败，且`CreateRemoteThread()`将以 0x00000000 为入口点开始执行，最终导致游戏崩溃。`WriteProcessMemory()`和`CreateRemoteThread()`的返回值也是如此。通常，只有在打开进程句柄时没有使用所需的访问标志时，这些函数才会失败。

### 劫持游戏主线程以执行代码洞

在某些情况下，注入的代码洞需要与游戏进程的主线程同步。解决这个问题可能非常棘手，因为这意味着你必须控制外部进程中的现有线程。

你可以简单地暂停主线程，直到代码洞执行完毕，这可能有效，但速度非常慢。等待代码洞并恢复线程的开销相当大。一个更快的替代方法是强制线程为你执行代码，这个过程称为*线程劫持*。

**注意**

*打开* main-codeInjection.cpp *文件，以便跟随本书中的源代码构建这个线程劫持示例，这是一个简化版的* `injectCodeUsingThreadHijacking()`。

#### *构建汇编代码洞*

与线程注入类似，线程劫持的第一步是知道你希望在代码洞中发生什么。然而，这次你并不知道劫持的线程会执行什么内容，所以你需要确保在代码洞开始时保存线程的状态，并在劫持完成后恢复状态。这意味着你的 shellcode 需要包裹在一些汇编代码中，如示例 7-5 所示。

```
PUSHAD // push general registers to the stack
PUSHFD // push EFLAGS to the stack
 // shellcode should be here

POPFD // pop EFLAGS from the stack
POPAD // pop general registers to the stack

// resume the thread without using registers here
```

*清单 7-5：线程劫持代码洞的框架*

如果你想调用与线程注入时相同的`someFunction`，你可以使用类似于清单 7-2 中的 Shellcode。唯一的不同是，你不能通过栈将第二个参数传递给你的机器人，因为你不会使用`CreateRemoteThread()`。但这没有问题；你可以像推送第一个参数那样推送第二个参数。执行你想要调用的函数的代码洞部分应该类似于清单 7-6 中的内容。

```
PUSH string
PUSH times 
MOV EAX, someFunction
CALL EAX 
ADD ESP, 0x8
```

*清单 7-6：调用`someFunction`的汇编骨架*

与清单 7-1 相比，唯一的变化是这个例子显式地推送了`string`，并且没有`RETN`。在这种情况下，你不调用`RETN`，因为你希望游戏线程回到它在被劫持之前所做的事情。

要正常恢复线程的执行，代码洞需要跳转回线程原始的 EIP，而不使用寄存器。幸运的是，你可以使用`GetThreadContext()`函数来获取`EIP`，然后在 C++中填充 Shellcode 骨架。接着，你可以将其推入栈中并执行返回操作。清单 7-7 展示了代码洞应该如何结束。

```
PUSH originalEIP
RETN
```

*清单 7-7：间接跳转到 EIP*

返回指令跳转到栈顶的值，因此，在压入 EIP 之后立即执行跳转即可实现目标。你应该使用这种方法，而不是跳转指令，因为跳转需要偏移量计算，并且会使生成 Shellcode 变得稍微复杂。如果将清单 7-5 到 7-7 连接起来，你会得到以下代码洞：

```
//save state
PUSHAD           // push general registers to the stack
PUSHFD           // push EFLAGS to the stack
 // do work with shellcode
PUSH string 
PUSH times 
MOV EAX, someFunction
CALL EAX 
ADD ESP, 0x8

// restore state
POPFD            // pop EFLAGS from the stack
POPAD            // pop general registers to the stack

// un-hijack: resume the thread without using registers
PUSH originalEIP 
RETN
```

接下来，按照“将汇编代码转换为 Shellcode”中的指示，在第 135 页上将这些字节插入到表示代码洞的数组中。

#### *生成骨架 Shellcode 并分配内存*

使用在清单 7-2 中展示的相同方法，你可以生成此代码洞的 Shellcode，如清单 7-8 所示。

```
BYTE codeCave[31] = {
    0x60,                         // PUSHAD
    0x9C,                         // PUSHFD
    0x68, 0x00, 0x00, 0x00, 0x00, // PUSH 0
    0x68, 0x00, 0x00, 0x00, 0x00, // PUSH 0
    0xB8, 0x00, 0x00, 0x00, 0x00, // MOV EAX, 0x0
    0xFF, 0xD0,                   // CALL EAX
    0x83, 0xC4, 0x08,             // ADD ESP, 0x08
    0x9D,                         // POPFD
    0x61,                         // POPAD
    0x68, 0x00, 0x00, 0x00, 0x00, // PUSH 0
    0xC3                          // RETN
};

// we'll need to add some code here to place
// the thread's EIP into threadContext.Eip

memcpy(&codeCave[3], &remoteString, 4);
memcpy(&codeCave[8], &times, 4);
memcpy(&codeCave[13], &func, 4);
memcpy(&codeCave[25], &threadContext.Eip, 4);
```

*清单 7-8：创建线程劫持 Shellcode 数组*

如同在清单 7-3 中所示，`memcpy()`被用来将变量放入骨架中。不过，与该清单中的不同之处在于，有两个变量不能立即复制；`times`和`func`是立即已知的，但`remoteString`是分配的结果，`threadContext.Eip`只有在线程被冻结后才会知道。冻结线程之前分配内存也是合理的，因为你不希望线程冻结的时间比必要的更长。下面是可能的实现方式：

```
int stringlen = strlen(string) + 1;
int cavelen = sizeof(codeCave);
int fulllen = stringlen + cavelen;

auto remoteString =
    VirtualAllocEx(process, 0, fulllen, MEM_COMMIT, PAGE_EXECUTE);
auto remoteCave =
    (LPVOID)((DWORD)remoteString + stringlen);
```

分配内存的代码与线程注入时相同，因此你可以重复使用相同的代码片段。

#### *查找并冻结主线程*

冻结主线程的代码稍微复杂一些。首先，你需要获取线程的唯一标识符。这与获取 PID 类似，你可以使用 `CreateToolhelp32Snapshot()`、`Thread32First()` 和 `Thread32Next()` 函数，来自 *TlHelp32.h* 文件。如同在 “获取游戏的进程标识符”（第 120 页）中所讨论的，这些函数基本上是用来遍历一个列表的。一个进程可以有多个线程，但以下示例假设游戏进程创建的第一个线程是需要被劫持的线程：

```
DWORD GetProcessThreadID(HANDLE Process) {
    THREADENTRY32 entry;
    entry.dwSize = sizeof(THREADENTRY32);
    HANDLE snapshot = CreateToolhelp32Snapshot(TH32CS_SNAPTHREAD, 0);

    if (Thread32First(snapshot, &entry) == TRUE) {
        DWORD PID = GetProcessId(Process);
        while (Thread32Next(snapshot, &entry) == TRUE) {
            if (entry.th32OwnerProcessID == PID) {
                CloseHandle(snapshot);
                return entry.th32ThreadID;
            }
        }
    }
    CloseHandle(snapshot);
    return NULL;
}
```

这段代码简单地遍历系统中所有线程的列表，找到与游戏的 PID 匹配的第一个线程。然后它从快照条目中获取线程标识符。一旦你知道了线程标识符，就可以像这样获取线程当前的寄存器状态：

```
HANDLE thread = OpenThread(
    (THREAD_GET_CONTEXT | THREAD_SUSPEND_RESUME | THREAD_SET_CONTEXT),
    false, threadID);
SuspendThread(thread);
 CONTEXT threadContext;
threadContext.ContextFlags = CONTEXT_CONTROL;
GetThreadContext(thread, &threadContext);
```

这段代码使用 `OpenThread()` 获取线程句柄。然后，它通过 `SuspendThread()` 暂停线程，并使用 `GetThreadContext()` 获取其寄存器的值。之后，列表 7-8 中的 `memcpy()` 代码应该拥有生成 shellcode 所需的所有变量。

在生成了 shellcode 后，可以像在 列表 7-4 中一样将代码洞写入已分配的内存：

```
WriteProcessMemory(process, remoteString, string, stringlen, NULL);
WriteProcessMemory(process, remoteCave, codeCave, cavelen, NULL);
```

一旦代码洞准备好并在内存中等待，你所需要做的就是将线程的 `EIP` 设置为代码洞的地址，让线程恢复执行，如下所示：

```
threadContext.Eip = (DWORD)remoteCave;
threadContext.ContextFlags = CONTEXT_CONTROL;
SetThreadContext(thread, &threadContext);
ResumeThread(thread);
```

这段代码使线程在代码洞的地址处恢复执行。由于代码洞的写法，线程根本不知道任何事情已经改变。代码洞保存了线程的原始状态，执行有效负载，恢复线程的原始状态，然后带着一切完整无损地返回到原始代码。

当你使用任何形式的代码注入时，了解你的代码洞会接触到哪些数据也很重要。例如，如果你创建一个代码洞来调用游戏的内部函数以创建并发送网络数据包，你需要确保当你完成后，任何函数接触到的全局变量（如数据包缓冲区、数据包位置标记等）都能安全地恢复。你永远无法知道在代码洞执行时游戏在做什么——它可能也在调用与你相同的函数！

### 注入 DLL 实现完全控制

代码洞非常强大（你可以使用汇编语言的 shellcode 让游戏做任何事情），但手工编写 shellcode 并不实际。注入 C++ 代码会方便得多，不是吗？这是可能的，但过程要复杂得多：代码必须先编译成汇编语言，打包成与位置无关的格式，意识到任何外部依赖项，完全映射到内存中，然后在某个入口点执行。

幸运的是，Windows 已经处理了所有这些问题。通过将一个 C++项目改为编译为动态库，你可以创建一个自包含、位置无关的二进制文件，称为*动态链接库（DLL）*。然后，你可以使用线程注入或劫持和`LoadLibrary()`API 函数的混合方法，将你的 DLL 文件映射到游戏的内存中。

打开*main-codeInjection.cpp*，该文件位于*GameHackingExamples/Chapter7_ CodeInjection*目录下，以及*GameHackingExamples/Chapter7_CodeInjection_DLL*中的*dllmain.cpp*，按照这部分内容中的一些示例代码进行操作。在*main-codeInjection.cpp*中，特别查看`LoadDLL()`函数。

#### *欺骗进程加载你的 DLL*

通过使用代码洞，你可以欺骗远程进程调用`LoadLibrary()`来加载 DLL，从而有效地将外部代码加载到其内存空间中。由于`LoadLibrary()`只接受一个参数，因此你可以创建一个代码洞来调用它，如下所示：

```
// write the dll name to memory
wchar_t* dllName = "c:\\something.dll";
int namelen = wcslen(dllName) + 1;
LPVOID remoteString =
    VirtualAllocEx(process, NULL, namelen * 2, MEM_COMMIT, PAGE_EXECUTE);
WriteProcessMemory(process, remoteString, dllName, namelen * 2, NULL);

// get the address of LoadLibraryW()
HMODULE k32 = GetModuleHandleA("kernel32.dll");
LPVOID funcAdr = GetProcAddress(k32, "LoadLibraryW");

// create a thread to call LoadLibraryW(dllName)
HANDLE thread =
    CreateRemoteThread(process, NULL, NULL,
        (LPTHREAD_START_ROUTINE)funcAdr,
        remoteString, NULL, NULL);

// let the thread finish and clean up
WaitForSingleObject(thread, INFINITE);
CloseHandle(thread);
```

这段代码实际上是混合了来自“绕过 ASLR 生产环境”第 128 页的线程注入代码和在 Listings 7-2 和 7-3 中创建的调用`someFunction`的代码洞。与前者类似，这个示例使用单参数 API 函数的函数体，具体是`LoadLibrary`，作为代码洞的主体。不过像后者一样，它需要将一个字符串注入到内存中，因为`LoadLibrary`将字符串指针作为第一个参数。一旦线程被注入，它会强制`LoadLibrary`加载被注入内存中的 DLL，从而有效地将外部代码注入到游戏中。

**注意**

*为你计划注入的任何 DLL 起一个独特的名字，比如*MySuperBotV2Hook.dll。*更简单的名字，例如*Hook.dll*或*Injected.dll*，则过于通用，具有潜在危险。如果名字与已经加载的 DLL 冲突，*`LoadLibrary()`*将认为它是同一个 DLL，从而不加载它！*

一旦`LoadLibrary()`代码洞将你的 DLL 加载到游戏中，DLL 的入口点——即`DllMain()`——将以`DLL_PROCESS_ATTACH`作为原因被执行。当进程被终止或调用`FreeLibrary()`时，DLL 的入口点将以`DLL_PROCESS_DETACH`作为原因被调用。从入口点处理这些事件可能看起来是这样的：

```
BOOL APIENTRY DllMain(HMODULE hModule,
                      DWORD ul_reason_for_call,
                      LPVOID lpReserved) {
    switch (ul_reason_for_call) {
        case DLL_PROCESS_ATTACH:
            printf("DLL attached!\n");
            break;
        case DLL_PROCESS_DETACH:
            printf("DLL detached!\n");
            break;
    }
    return TRUE;
}
```

这个示例函数首先检查为什么调用了`DllMain()`。然后输出文本，指示它是因为 DLL 被附加还是分离而被调用，无论哪种情况，都会返回`TRUE`。

请记住，DLL 的入口点是在*加载器锁*内执行的，加载器锁是一个全局同步锁，用于所有读取或修改进程中加载的模块列表的函数。像`GetModuleHandle()`、`GetModuleFileName()`、`Module32First()`和`Module32Next()`等函数会使用这个加载器锁，这意味着从 DLL 入口点运行复杂代码可能会导致死锁，应当避免。

如果你需要从 DLL 入口点运行代码，请通过新线程来执行，如下所示：

```
DWORD WINAPI runBot(LPVOID lpParam) {
    // run your bot
    return 1;
}

// do this from DllMain() for case DLL_PROCESS_ATTACH
auto thread = CreateThread(NULL, 0, &runBot, NULL, 0, NULL);
CloseHandle(thread);
```

从`DllMain()`开始，这段代码创建了一个新线程，线程从`runBot()`函数开始。然后它立即关闭了对该线程的句柄，因为从`DllMain()`执行进一步操作可能会导致严重的问题。在`runBot()`内部，你可以开始执行你的机器人代码。代码在游戏内部运行，这意味着你可以直接使用类型转换方法来操作内存。你还能做更多的事情，正如你将在第八章中看到的。

在注入 DLL 时，确保没有依赖问题。如果你的 DLL 依赖某些非标准的 DLL，例如，你必须先将这些 DLL 注入游戏中，或者将它们放在`LoadLibrary()`会搜索的文件夹中，比如`PATH`环境变量中的任何文件夹。前者只有在这些 DLL 没有自己的依赖关系时才有效，而后者实现起来有些棘手，并且容易发生名称冲突。最佳的选择是将所有外部库静态链接，这样它们就会直接编译到你的 DLL 中。

#### *访问注入 DLL 中的内存*

当你尝试从注入的 DLL 访问游戏内存时，进程句柄和 API 函数会成为障碍。因为游戏与所有注入其中的代码共享相同的内存空间，所以你可以直接从注入的代码访问游戏的内存。例如，要从注入的代码访问一个`DWORD`值，你可以写如下代码：

```
DWORD value = *((DWORD*)adr); // read a DWORD from adr
*((DWORD*)adr) = 1234;        // write 1234 to DWORD adr
```

这只是将内存地址`adr`强制转换为`DWORD*`类型，然后解引用该指针为一个`DWORD`。这样进行类型转换是可以的，但如果将函数抽象化并通用化，像 Windows API 包装器一样，你的内存访问代码会更加简洁。

用于从注入的代码内部访问内存的通用函数看起来像这样：

```
template<typename T>
T readMemory(LPVOID adr) {
    return *((T*)adr);
}

template<typename T>
void writeMemory(LPVOID adr, T val) {
    *((T*)adr) = val;
}
```

使用这些模板就像在第 123 页的“编写模板化内存访问函数”部分使用函数一样。以下是一个例子：

```
DWORD value = readMemory<DWORD>(adr); // read
writeMemory<DWORD>(adr, value++);     // increment and write
```

这些调用与清单 6-6 中第 124 页的调用几乎完全相同；它们只是无需将进程句柄作为参数传入，因为它们是从进程内部调用的。你可以通过创建一个名为`pointMemory()`的第三个模板函数来使这个方法更灵活，如下所示：

```
template<typename T>
T* pointMemory(LPVOID adr) {
    return ((T*)adr);
}
```

这个函数跳过了内存读取的解引用步骤，直接给你数据的指针。从这里开始，你可以自由地通过解引用这个指针来读取和写入内存，就像这样：

```
DWORD* pValue = pointMemory<DWORD>(adr); // point
DWORD value = *pValue;                   // 'read'
(*pValue)++;                             // increment and 'write'
```

使用像`pointMemory()`这样的函数，你可以省略对`readMemory()`和`writeMemory()`的调用。你仍然需要事先找到`adr`，但从那时起，读取值、改变值并将其写回的代码会变得更加简洁。

#### *绕过注入 DLL 中的 ASLR*

类似地，由于代码已经被注入，因此无需再为游戏注入一个线程来获取基地址。相反，你可以直接调用`GetModuleHandle()`，像这样：

```
DWORD newBase = (DWORD)GetModuleHandle(NULL);
```

获取基地址的一个更快方法是利用游戏的 FS 内存段，这是你从注入的代码中获得的另一个超能力。这个内存段指向一个叫做*线程环境块（TEB）*的结构体，而 TEB 中偏移 0x30 的地方是指向*进程环境块（PEB）*结构体的指针。操作系统使用这些结构体，它们包含大量关于当前线程和当前进程的数据，但我们只对存储在 PEB 中的主模块基地址感兴趣，基地址位于 PEB 的偏移 0x8 处。通过内联汇编，你可以遍历这些结构来获取`newBase`，像这样：

```
DWORD newBase;
__asm {
    MOV EAX, DWORD PTR FS:[0x30]
    MOV EAX, DWORD PTR DS:[EAX+0x8]
    MOV newBase, EAX
}
```

第一个命令将`PEB`地址存储在`EAX`中，第二个命令读取主模块的基地址并将其存储在`EAX`中。最后一个命令将`EAX`复制到`newBase`。

### 总结思考

在第六章中，我向你展示了如何远程读取内存，以及注入的 DLL 如何通过指针直接访问游戏的内存。本章展示了如何注入各种类型的代码，从纯汇编字节码到完整的 C++二进制文件。在下一章，你将了解到进入游戏内存空间究竟能赋予你多少权力。如果你觉得汇编代码注入很酷，那么你会喜欢将注入的 C++与控制流操作结合后的效果。

本章的示例代码包含了我们讨论的所有概念验证。如果你对其中的任何主题仍然不清楚，可以通过查看代码来了解具体发生了什么，并看到所有技巧的实际应用。
