

## 第六章：5 用户环境与交互检测



![](img/opener.jpg)

随着自动化恶意软件沙盒技术越来越善于隐藏自己免受规避型恶意软件的攻击，恶意软件作者必须做出调整。他们使用的一种策略是列举用户的环境以及用户与其互动的情况。正如第四章所指出的，普通用户的设置通常会有多个打开的浏览器标签页、许多窗口和正在使用的应用程序，且频繁的鼠标和键盘交互，使得它与沙盒环境有很大的不同。自动化的恶意软件分析沙盒设计是启动、引爆恶意软件样本，然后迅速关闭。它可能不会表现出任何正常用户行为或其他表明其是有效最终用户系统的迹象。

现代恶意软件可能会通过寻找典型的用户行为来查找真实用户的证据，例如下载的浏览器 Cookies、桌面壁纸设置或鼠标和键盘交互。在这一章中，我将概述一些恶意软件用来实现这一目标的有趣技巧。

### 浏览器 Cookies、缓存和浏览历史

一些恶意软件可能能够列举主机的互联网 Cookies、缓存和浏览历史。*Cookies* 是网页保存到磁盘的小文件，通常用于存储用户的网页配置和偏好设置。根据浏览器和版本的不同，Cookies 可以存储在单独的文件中，或存储在像 SQLite 这样的数据库中。*缓存* 是存储网站资源（如图片）的文件或一组文件，这样下次用户访问该页面时可以更快地加载。与 Cookies 类似，缓存可以存储在多个文件中，也可以存储在数据库中。最后，*浏览历史* 只是一个先前访问过的网站的列表，通常存储为一个或多个数据库文件。

普通用户通常会有成百上千个存储的 Cookie 和缓存文件，以及大量的上网历史，而典型的沙盒环境或恶意软件分析系统可能根本没有这些内容。恶意软件可以利用这一差异，通过计算 Cookie、缓存条目或之前访问过的网站数量，并将其与阈值进行比较。例如，如果受害者的机器上只有五个浏览历史条目，恶意软件可能会认为它运行在一个干净的沙盒环境中。

每个浏览器都有标准的位置来存储 Cookies、缓存文件和浏览历史，恶意软件可能会尝试列举这些内容。以下是一些最常见的：

**Chrome**

+   *C:\Users\<user>\AppData\Local\Google\Chrome\User Data\Default*

+   *C:\Users\<user>\AppData\Local\Google\Chrome\User Data\Default\Cache*

+   *C:\Users\<user>\AppData\Local\Google\Chrome\User Data\Default\History*

**Firefox**

+   *C:\Users\<user>\AppData\Local\Mozilla\Firefox\Profiles*

+   *C:\Users\<user>\AppData\Roaming\Mozilla\Firefox\Profiles*

**Internet Explorer**

+   *C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Cookies*

+   *C:\Users\<user>\AppData\Local\Microsoft\Windows\Temporary Internet Files*

+   *C:\Users\<user>\AppData\Local\Microsoft\Windows\WebCache*

+   *C:\Users\<user>\AppData\Local\Microsoft\Internet Explorer\Recovery*

**Edge**

+   *C:\Users\<user>\AppData\Local\Packages\<package name>\AC\MicrosoftEdge\User\*

+   *C:\Users\<user>\AppData\Local\Packages\<package name>\AC\MicrosoftEdge\Cache\*

这个列表并不详尽，当然，具体位置可能会根据使用的 Windows 操作系统和浏览器版本的不同而发生变化。

如果你发现恶意软件正在通过这些文件枚举（可能通过调用 Windows 函数，如 FindFirstFile 和 FindNextFile），它可能正在尝试检测分析环境。恶意软件也可能使用 FindFirstUrlCacheEntry 和 FindNextUrlCacheEntry，这些函数会顺序枚举浏览器缓存条目。然而，这些 API 仅限于 Microsoft 浏览器缓存。再次强调，枚举方法很大程度上取决于使用的浏览器和版本。

较旧的浏览器和版本通常使用多个小文件来存储 cookies、缓存和历史记录，而现代浏览器则使用数据库。如果浏览器的 cookies、缓存和历史记录存储在数据库文件中，恶意软件可能会尝试直接与它们交互。例如，在恶意软件的可执行文件或其进程内存地址空间中，你可能会看到引用特定浏览器目录的静态字符串（例如 *C:\Users\<user>\AppData\Local\Google\Chrome\User Data\Default\History*），然后是类似这样的数据库查询：

```
SELECT title FROM urls
```

这个命令可以用来枚举历史数据库中的所有网络历史记录。由于数据库交互超出了本书的范围，我们不会在这里深入讨论，但需要注意这种技术。

### 最近的 Office 文件

使用最近的 Office 文件是恶意软件判断是否在分析实验室中运行的另一种有效方式。真实的最终用户很可能已经打开了许多 Microsoft Office 应用程序的文件，而 Windows 会跟踪这些文件。例如，当你在 Word 中打开文档时，该文件将会被添加到你的*Office 最近文件*列表中。

有关您最近的 Office 文件的信息包含在注册表项HKEY_CURRENT_USER\SOFTWARE\Microsoft\Office\<Office_Version_Number>下的特定子项中，如Excel、Word、Powerpoint等。进一步的信息可能存储在文件系统目录*C:\Users\<user>\AppData\Roaming\Microsoft\Office\Recent*中。如果您发现恶意软件正在枚举此注册表项或文件夹路径（使用之前提到的任何 Windows 文件和注册表枚举函数），它可能正在尝试识别最近的 Office 文档，以确定受害主机是否由“真实”最终用户使用。

### 用户文件和目录

一个典型的用户可能在系统的不同用户目录中拥有许多文件，例如*文档*、*图片*、*桌面*等。通过使用第四章中描述的文件枚举方法，恶意软件可以枚举这些目录，以判断主机是否为真实用户。如果恶意软件发现这些目录中没有用户活动，它可能会得出结论，认为自己正在沙箱或分析环境中运行，并采取规避措施。

### 桌面壁纸

恶意软件用来检测分析机器的一种特别有创意的方法是检查当前配置的壁纸，因为真实用户往往会更改桌面壁纸，而不是使用 Windows 默认壁纸。为此，恶意软件可以简单地检查壁纸注册表项HKEY_CURRENT_USER\Software\Microsoft\Internet Explorer\Desktop\General\WallpaperSource。如果用户仍然使用默认的 Windows 壁纸，WallpaperSource值将包含该壁纸的路径，通常位于*C:\Windows\*目录中。另一方面，如果用户配置了自定义桌面壁纸，WallpaperSource值可能包含自定义目录和图像名称，例如*C:\Users\<user>\Pictures\my_wallpaper.jpg*。

### 桌面窗口

一些恶意软件变种会统计活动桌面窗口的数量或搜索特定的窗口。它们可以使用GetForegroundWindow函数来测试前景窗口（即当前活动窗口）是否发生变化。因为我正在 LibreOffice 中输入这段文本，所以这个程序是我的活动前景窗口。作为合法用户，我的活动窗口可能会有很大变化；例如，我可能会最小化 LibreOffice，休息一下，去 Chrome 里看 YouTube 猫咪视频。在自动化的恶意软件分析沙箱环境中，活动窗口可能不会有太大变化。一些恶意软件变种意识到这一点，并可能利用这一特性对抗分析系统进行检测。在这个例子中，恶意软件正在检查前景窗口在五秒后是否发生了变化：

```
loc_34E642:
call GetForegroundWindow
mov dword ptr ds:[ebx+WindowHandle], eax
push 1388h ; "5s"
call Sleep
call GetForegroundWindow
cmp dword ptr ds:[ebx+WindowHandle], eax
je loc_34E642
```

首先，恶意软件调用GetForegroundWindow，该函数返回当前前景窗口的句柄，并将其存储在地址[ebx+WindowHandle]的缓冲区中。接下来，恶意软件调用Sleep函数，这将暂停样本五秒钟。恶意软件第二次调用GetForegroundWindow，然后通过cmp dword ptr ds:[ebx+WindowHandle], eax来比较两次GetForegroundWindow调用的句柄值。如果句柄匹配（即前景窗口没有变化），该例程将再次循环。这个恶意软件样本可能会无限循环，完全避免在自动化沙箱中被分析，或者它也可能循环几次然后终止自己。无论哪种情况，都为恶意软件分析沙箱带来了有趣的挑战。幸运的是，许多现代沙箱模拟了用户活动来抵御这种技术。有些沙箱甚至可以在交互模式下运行，这使得你可以直接与沙箱中的恶意软件进行互动，从而帮助规避这种策略。

另外，恶意软件样本可以使用EnumWindows函数，该函数返回用户桌面上打开窗口的数量。Windows 会因多种原因创建许多窗口对象，因此在正常的用户环境中，这个数字通常会很高。例如，我在我的个人系统上运行了EnumWindows，它返回了 97 个窗口！在沙箱分析环境中，这个数字可能会显著较低。以下代码片段演示了EnumWindows函数的使用：

```
push ebx
call EnumWindows
pop eax
cmp eax, 20
jle terminate_process
```

EnumWindows 接受一个参数，基本上作为一个指向缓冲区的指针，用于存储函数调用的结果（push ebx）。在调用 EnumWindows 后，pop eax 指令将会从 ebx 缓冲区中弹出指针，放入 eax 中。恶意软件将 EnumWindows 的值（现在存储在 eax 中）与 20 进行比较，如果打开的窗口数量小于或等于该值，样本程序将会自我终止。这个样本假设恶意软件分析沙箱在任何时候都会有最多 20 个窗口处于激活状态。

除了枚举受害者系统上活动窗口的数量，或者感知前景窗口是否发生变化外，恶意软件还可以主动搜索特定的应用程序窗口。这有两个原因。首先，恶意软件样本可以搜索典型用户常用的已打开应用程序窗口：Microsoft Office 产品、电子邮件程序、浏览器等。如果打开了足够多的这些应用程序，恶意软件可以合理地推测它*不是*在恶意软件分析员的实验室中运行。其次，恶意软件样本还可以查找某些恶意软件分析工具，类似于我在第四章中“进程”部分所描述的。例如，恶意软件可能会搜索包含 *Procmon*、*Fiddler* 或 *Wireshark* 等术语的已打开窗口，通常是通过调用 EnumWindows 或 FindWindow 函数来实现。与查找特定进程类似，它会遍历打开的窗口，并将每个窗口的标题与一个字符串进行比较，结果可以让它察觉到自己正在被分析。

### 鼠标和键盘交互

日常使用者几乎总是使用鼠标在屏幕上移动光标，无论是在浏览互联网、编辑文档还是玩视频游戏。一些恶意软件可以使用 GetCursorPos 来检测这一活动，该函数返回用户鼠标光标的坐标。以下伪代码展示了这种情况可能的实现方式：

```
GetCursorPos(&CursorPos1)
Sleep(30)
GetCursorPos(&CursorPos2)

if (CursorPos1 == CursorPos2)
  TerminateProcess()
```

首先，恶意软件调用GetCursorPos函数，并将得到的鼠标光标坐标存储到CursorPos1缓冲区中。接着，它调用Sleep函数，使恶意软件的执行暂停 30 秒，然后再次调用GetCursorPos。最后，它比较两个得到的光标坐标值，如果相同（即光标没有移动），样本将自行终止。你大概可以看出，这是一种有效的绕过自动沙箱的方式，因为光标不太可能自行移动（除非沙箱被设计为模仿真实用户的行为）。

另一种类似的技术是恶意软件等待特定的鼠标按钮被按下，或者等待一定次数的鼠标点击事件发生后再执行其恶意代码。FireEye 于 2012 年撰写了一篇研究文章《Hot Knives Through Butter: Evading File-based Sandboxes》，讨论了这种技术被一个名为 Upclicker 的恶意软件家族所使用。为了监控这些鼠标操作，Upclicker 在鼠标上建立了一个*钩子*，使恶意软件能够拦截并监视所有鼠标活动，等待特定事件的发生。以下是恶意软件代码可能的实现方式：

```
push offset jump_location
push 0Eh
call SetWindowsHookExA
`--snip--`
loc jump_location:
call do_evil_things
```

恶意软件样本首先将jump_location参数推送到栈中；当特定的鼠标事件发生时，恶意软件将跳转到这个位置。另一个参数0E（十六进制，或十进制 14）告诉SetWindowsHookExA来钩取鼠标操作。调用SetWindowsHookExA会告诉程序在受害者用户点击鼠标按钮时跳转到jump_location指定的代码。

这段代码为了简洁起见已做了简化。实际上，恶意软件可能会实现额外的逻辑，只在特定的鼠标事件发生时采取行动，比如左键点击（如 Upclicker 的情况）。想了解更多关于 Upclicker 的信息，并且获得一个关于沙箱规避的良好介绍，可以查看 FireEye 的报告，链接在这里：[*https://<wbr>media<wbr>.blackhat<wbr>.com<wbr>/us<wbr>-13<wbr>/US<wbr>-13<wbr>-Singh<wbr>-Hot<wbr>-Knives<wbr>-Through<wbr>-Butter<wbr>-Evading<wbr>-File<wbr>-based<wbr>-Sandboxes<wbr>-WP<wbr>.pdf<wbr>*](https://media.blackhat.com/us-13/US-13-Singh-Hot-Knives-Through-Butter-Evading-File-based-Sandboxes-WP.pdf)。

这种挂钩技巧不仅适用于鼠标。恶意软件也可以通过将 0Dh（十进制为 13）传递给 SetWindowsHookEx 函数，来挂钩键盘，然后等待特定的按键被按下后再完全执行。（挂钩技术将在第八章和第十二章中详细讨论。）另外，恶意软件也可以调用 GetAsyncKeyState 函数来监视按键。

监控鼠标和键盘交互可以是检测并绕过自动化恶意软件分析沙盒的一个非常有效的方法。除非沙盒或恶意软件分析员按下特定的键或鼠标按钮，否则在沙盒环境下，恶意软件样本可能看起来完全无害。

> 注意

*为了模拟真实的终端用户环境，让你的分析虚拟机和沙盒尽可能像真实用户一样。更改桌面背景并访问一些网站（以填充你的 cookies 和缓存目录）可以起到很大作用。甚至打开额外的窗口并在屏幕上移动鼠标，也可能帮助避免一些检测技术，即使你觉得这样做有点傻。*

### 系统运行时间

*系统运行时间* 是指系统已经开机的时间长度，它可以是恶意软件判断自己是否处于分析环境中的一个重要指标。一个典型的终端用户设备通常会持续开机数小时，甚至数天。服务器可能会在没有重启的情况下开机数月或数年。由于恶意软件分析员通常会根据需要启动虚拟机和沙盒来分析恶意软件样本，因此短时间的系统运行时间可能是一个明显的提示，表明系统是一个分析机器。

检查系统运行时间有多种方式，可以通过 Windows API 或其他辅助命令实现。最常见的方法可能是 GetTickCount Windows API 函数，它以毫秒为单位返回系统的运行时间。一个 *tick* 是由处理器时钟产生的，时钟负责保持时间并协调指令。当系统关闭或重启时，GetTickCount 基本上会重置为 0。以下代码使用 GetTickCount 来查看系统是否已开机 20 分钟：

```
mov ebx, 124F80h
call GetTickCount
cmp eax, ebx
jb  terminate_process
```

该样本首先将十六进制的 124F80（十进制的 1200000）存入 ebx 寄存器，代表 1,200,000 毫秒，即 20 分钟。然后，它调用 GetTickCount 并将返回的计时值与 ebx 中的值进行比较。如果 GetTickCount 返回的值小于 ebx 的值，意味着系统开机时间少于 20 分钟，则恶意软件样本会终止自身。

恶意软件也可能使用 Windows 命令行获取系统的正常运行时间。可选命令包括 sysinfo 命令，它返回关于系统的各种信息，包括正常运行时间；uptime.exe，这是大多数版本的 Windows 自带的一个二进制文件；以及 net statistics workstation 命令。最后，恶意软件还可以通过调用 WMIC 来获取系统的正常运行时间，命令是 wmic os get lastbootuptime。

这里的最后一个重要提示是，GetTickCount及其他方法常用于良性和恶意应用程序中，而不仅仅是用于暴露分析环境和沙箱。仅仅因为恶意软件样本正在检查系统的正常运行时间，并不意味着它在采取逃避行为，但你应该将这种行为视为一个警示信号。

### 总结

在本章中，我们介绍了一些恶意软件枚举环境并寻找实际用户活动证据的创新且狡猾的方法。通过设计一个看起来对恶意软件合法的分析环境，你可以有效阻止这些用户检测技术中的许多。诸如更改默认的 Windows 桌面背景并确保浏览历史中有一些项目等变化，都是容易实施的。我们将在附录 A 中讨论其他阻止检测技术的方法。还要注意，一些高级沙箱已内置防护措施，能抵御许多这些技术。

在下一章中，我们将探讨如何通过逃避型恶意软件枚举系统硬件和网络设备信息，以检测虚拟机分析环境。
