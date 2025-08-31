

## 第六章：6 日志监控



![](img/opener.jpg)

如果你曾花时间研究 macOS，你可能遇到过系统的统一日志机制，这是一种帮助你了解 macOS 内部结构的资源，并且正如你将很快看到的，它也可以帮助揭示恶意软件。在本章中，我将首先强调可以从这些日志中提取的各种信息，用以检测恶意活动。接着，我们将逆向工程 macOS 日志工具和其核心私有框架之一，以便我们能够以编程方式高效地直接从日志子系统中获取实时信息。

### 探索日志信息

我将首先介绍一些有用的活动示例，这些活动可能出现在系统日志中，从摄像头访问开始。尤其是那些狡猾的恶意软件样本，包括 FruitFly、Mokes 和 Crisis，它们通过感染的主机摄像头悄悄地监视受害者。然而，访问摄像头会生成系统日志消息。例如，根据 macOS 的版本，Core Media I/O 子系统可能会生成如下信息：

```
CMIOExtensionProvider.m:2671:-[CMIOExtensionProvider setDevicePropertyValuesForClientID:
deviceID:propertyValues:reply:] <CMIOExtensionProvider>,
3F4ADF48-8358-4A2E-896B-96848FDB6DD5, propertyValues {
    **CMIOExtensionPropertyDeviceControlPID = 90429;**
} 
```

粗体值包含了访问摄像头的进程 ID。虽然这个进程可能是合法的，例如用户发起的 Zoom 或 FaceTime 会议，但为了谨慎起见，最好确认这一点，因为该进程也可能是恶意软件，试图监视用户。因为苹果并未提供一个 API 来标识访问摄像头的进程，所以日志消息通常是获取该信息的唯一可靠方式。

其他经常出现在系统日志中的活动是远程登录，这可能表明系统被攻破，例如攻击者通过 SSH 登录首次访问主机，或者甚至是回到先前被感染的主机。例如，IPStorm 恶意软件通过暴力破解 SSH 登录传播给受害者。^(1) 另一个有趣的案例是 XCSSET，它通过本地发起一个看似是远程连接的行为，绕过 macOS 的透明性、同意和控制（TCC）安全机制。^(2)

当通过 SSH 进行远程登录时，系统会生成如下日志消息：

```
sshd: Accepted keyboard-interactive/pam for Patrick from 192.168.1.176 port 59363 ssh2
sshd: (libpam.2.dylib) in pam_sm_setcred(): Establishing credentials
sshd: (libpam.2.dylib) in pam_sm_setcred(): Got user: Patrick
...
sshd: (libpam.2.dylib) in pam_sm_open_session(): UID: 501
sshd: (libpam.2.dylib) in pam_sm_open_session(): server_URL: (null)
sshd: (libpam.2.dylib) in pam_sm_open_session(): path: (null)
sshd: (libpam.2.dylib) in pam_sm_open_session(): homedir: /Users/Patrick
sshd: (libpam.2.dylib) in pam_sm_open_session(): username: Patrick 
```

这些日志消息提供了连接的源 IP 地址，以及登录用户的身份。这些信息可以帮助防御者判断 SSH 会话是否合法（例如，远程工作者连接到其办公室机器）还是未经授权的。

日志消息还可以提供有关 TCC 机制的洞见，TCC 机制控制着对敏感信息和硬件功能的访问。在一次“海边的目标”会议上，研究人员 Calum Hall 和 Luke Roberts 指出，他们通过统一日志中的消息，能够确定与某个 TCC 事件相关的几项信息（例如，恶意软件试图捕获屏幕或访问用户文档），包括进程请求访问的资源、负责和目标进程，以及系统是否拒绝或批准该请求及其原因。^(3)

此时，可能会产生将日志消息视为恶意软件检测的灵丹妙药的冲动。但不要这样做。苹果并不正式支持日志消息，且经常会改变其内容或将其完全移除，甚至在 macOS 的小版本更新之间。例如，在旧版本的操作系统中，你可以通过查看以下日志消息来检测麦克风访问并识别负责的进程：

```
send: 0/7 synchronous to com.apple.tccd.system: request: msgID=408.11,
function=TCCAccessRequest, service=kTCCServiceMicrophone, target_token={pid:23207, auid:501,
euid:501}, 
```

不幸的是，苹果更新了相关的 macOS 框架，导致不再生成此类消息。如果你的安全工具仅依赖此指示器来检测未经授权的麦克风访问，它将不再起作用。因此，最好将日志消息视为可疑行为的初步迹象，然后进行进一步调查。

### 统一日志子系统

我们通常认为日志消息是用来弄清楚过去发生了什么。但 macOS 还允许你订阅消息流，以便几乎实时地将它们送入日志子系统。更妙的是，日志子系统支持通过自定义谓词过滤这些消息，从而提供对系统活动的高效且无与伦比的洞察。

从 macOS 10.12 开始，这种日志机制被称为*统一日志系统*。^(4) 它取代了传统的 syslog 接口，记录来自核心系统守护进程、操作系统组件和任何通过 OSLog API 生成日志消息的第三方软件的消息。

值得注意的是，如果你检查统一系统日志中的日志消息，可能会遇到部分内容被编辑；日志子系统会将任何被认为敏感的信息替换为字符串<private>。要禁用此功能，你可以安装一个配置文件。^(5) 尽管这种功能有助于理解操作系统中未记录的特性，但你不应在终端用户或生产系统中禁用日志编辑，因为这会使敏感数据对任何能够访问日志的人开放。

#### 手动查询日志工具

若要手动与日志子系统交互，可以使用位于*/usr/bin*中的 macOS 日志工具：

```
% **/usr/bin/log**
usage:
    log <command>

global options:
    -?, --help
    -q, --quiet
    -v, --verbose

commands:
    collect         gather system logs into a log archive
    config          view/change logging system settings
    erase           delete system logging data
    show            view/search system logs
    stream          watch live system logs
    stats           show system logging statistics

further help:
    log help <command>
    log help predicates 
```

你可以使用 `show` 标志搜索以前记录的数据，或使用 `stream` 标志查看实时生成的日志数据。除非另行指定，否则输出将仅包括默认日志级别的消息。要覆盖过去数据的此设置，可以使用 --info 或 --debug 标志，并配合 `show` 查看更多信息或调试消息。对于流式数据，指定 `stream` 和 --level，然后选择 info 或 debug。这些标志是层级关系的；指定调试级别时，也会返回信息性和默认消息。

使用 --predicate 标志和谓词来过滤输出。一个相当广泛的有效谓词字段列表使你能够根据进程、子系统、类型等多种条件查找日志消息。例如，要从内核流式传输日志消息，请执行以下命令：

```
% **log stream --predicate 'process == "kernel"'**
```

通常有不止一种方式来构造谓词。例如，我们也可以通过使用 'processIdentifier == 0' 来接收内核消息，因为内核的进程 ID 总是 0。

要从安全子系统流式传输消息，请输入以下命令：

```
% **log stream --predicate 'subsystem == "com.apple.securityd"'**
```

这里展示的示例都使用了相等运算符（==）。然而，谓词可以使用许多其他运算符，包括比较运算符（如 ==、!= 和 <）、逻辑运算符（如 AND 和 OR），甚至是成员运算符（如 BEGINSWITH 和 CONTAINS）。成员运算符非常强大，因为它们允许你构造类似正则表达式的筛选谓词。

`log` 手册页和命令 `log help predicates` 提供了谓词的简洁概述。^(6)

#### 逆向工程日志 API

要以编程方式读取日志数据，我们可以使用 OSLog API。^(7) 然而，这些 API 仅返回历史数据，在恶意软件检测的背景下，我们更关心的是实时事件。没有公共 API 允许我们实现这一点，但通过逆向工程日志工具（特别是支持 stream 命令的代码），我们可以确切地了解如何在日志消息进入统一日志子系统时获取它们。此外，通过提供筛选谓词，我们只会接收对我们有兴趣的消息。

虽然我不会详细讲解如何逆向日志工具，但在这一部分我将概述整个过程。当然，你也可以对其他苹果工具和框架应用类似的过程，以提取对恶意软件检测有用的私有 API（正如我们在第三章中实现软件包代码签名检查时所展示的）。

首先，我们需要找到实现日志子系统 API 的二进制文件，以便我们能够从自己的代码中调用它们。通常，我们会在一个动态链接到工具二进制文件的框架中找到这些 API。通过执行 `otool -L` 命令行选项，我们可以查看日志工具动态链接的框架：

```
% **otool -L /usr/bin/log**
/System/Library/PrivateFrameworks/ktrace.framework/Versions/A/ktrace
/System/Library/PrivateFrameworks/LoggingSupport.framework/Versions/A/LoggingSupport
/System/Library/PrivateFrameworks/CoreSymbolication.framework/Versions/A/CoreSymbolication
... 
```

根据其名称，*LoggingSupport* 框架似乎很可能包含相关的日志 API。在过去的 macOS 版本中，你可以在 */System/Library/PrivateFrameworks/* 目录中找到该框架，而在较新的版本中，你会在共享的 *dyld* 缓存中找到它。

将框架加载到 Hopper 中后（Hopper 可以直接从 *dyld* 缓存加载框架），我们发现该框架实现了一个名为 OSLogEventLiveStream 的未记录类，其基类是 OSLogEventStreamBase。 这些类实现了诸如 activate、setEventHandler: 和 setFilterPredicate: 等方法。我们还遇到一个未记录的 OSLogEventProxy 类，似乎代表了日志事件。以下是它的一些属性：

```
NSString* process;
int processIdentifier;
NSString* processImagePath;
NSString* sender;
NSString* senderImagePath;
NSString* category;
NSString* subsystem;
NSDate* date;
NSString* composedMessage; 
```

通过检查日志工具，我们可以看到它如何使用这些类及其方法来捕获流式日志数据。例如，这是从日志二进制文件反编译得到的一个片段：

```
r21 = [OSLogEventLiveStream initWithLiveSource:...];
[r21 setEventHandler:&var_110];
...
[r21 setFilterPredicate:r22];

printf("Filtering the log data using \"%s\"\n", @selector(UTF8String));
...
[r21 activate]; 
```

在反编译中，我们首先看到调用 initWithLiveSource: 初始化一个 OSLogEventLiveStream 对象。接着调用 setEventHandler: 和 setFilterPredicate: 等方法来配置该对象，存储在 r21 寄存器中。设置完谓词后，一条有用的调试信息表明，提供的谓词可以过滤日志数据。最后，该对象激活，触发了符合指定谓词的流式日志消息的接收。

### 流式日志数据

通过逆向工程日志二进制文件和 *LoggingSupport* 框架，我们获得的信息，可以帮助我们编写代码直接从通用日志子系统中流式传输数据到我们的检测工具。在这里，我们将介绍代码的关键部分，尽管建议你查阅本章的完整代码，位于 *logStream* 项目中。

清单 6-1 显示了一个方法，该方法接受一个日志过滤谓词、一个日志级别（如默认、信息或调试），以及一个回调函数，用于对每个符合指定谓词的日志事件进行调用。

```
#define LOGGING_SUPPORT @"/System/Library/PrivateFrameworks/LoggingSupport.framework"

-(void)start:(NSPredicate*)predicate
level:(NSUInteger)level eventHandler:(void(^)(OSLogEventProxy*))eventHandler {
    [[NSBundle bundleWithPath:LOGGING_SUPPORT] load]; ❶
    Class LiveStream = NSClassFromString(@"OSLogEventLiveStream"); ❷

    self.liveStream = [[LiveStream alloc] init]; ❸

    @try {
        [self.liveStream setFilterPredicate:predicate]; ❹
    } @catch (NSException* exception) {
 // Code to handle invalid predicate removed for brevity
    }
    [self.liveStream setInvalidationHandler:^void (int reason, id streamPosition) {
        ;
    }];

    [self.liveStream setDroppedEventHandler:^void (id droppedMessage) {
        ;
    }];

    [self.liveStream setEventHandler:eventHandler]; ❺
    [self.liveStream setFlags:level]; ❻

    [self.liveStream activate]; ❼
} 
```

清单 6-1：使用指定谓词启动日志流

请注意，我已经省略了这部分代码，如自定义日志类的类定义和属性。

加载日志支持框架 ❶ 后，代码通过名称检索私有的 OSLogEventLiveStream 类 ❷。现在我们可以实例化该类的一个实例 ❸。然后，我们通过设置过滤谓词 ❹ 来配置这个实例，确保将其包装在 try...catch 块中，因为如果提供无效的谓词，setFilterPredicate: 方法可能会抛出异常。接下来，我们设置事件处理程序，框架将在每次通用日志子系统接收符合指定谓词的日志消息时调用该处理程序 ❺。我们将这些值传递给 start:level:eventHandler: 方法，其中谓词告诉日志流如何过滤它传递给事件处理程序的消息。我们通过 setFlags: 方法设置日志级别 ❻。最后，我们通过调用 activate 方法启动流 ❼。

清单 6-2 展示了如何创建自定义日志监视类的实例，并使用它开始接收日志消息。

```
NSPredicate* predicate = [NSPredicate predicateWithFormat:<some string predicate>]; ❶

LogMonitor* logMonitor = [[LogMonitor alloc] init]; ❷

[logMonitor start:predicate level:Log_Level_Debug eventHandler:^(OSLogEventProxy* event) {
    printf("New Log Message: %s\n\n", event.description.UTF8String);
}];

[NSRunLoop.mainRunLoop run]; 
```

清单 6-2：与自定义日志流类进行交互

首先，代码从一个字符串 ❶ 创建一个谓词对象。请注意，在生产代码中，你还应该将此操作包装在 try...catch 块中，因为如果提供的谓词无效，predicateWithFormat: 方法会抛出一个可捕获的异常。接下来，我们创建一个 LogMonitor 对象并调用它的 start:level:eventHandler: 方法 ❷。请注意，对于级别，我们传入 Log_Level_Debug。由于级别是分层的，这将确保我们捕获所有类型的消息，包括那些类型为 info 和 default 的消息。现在，每当一个与指定谓词匹配的日志消息流向通用日志子系统时，代码将调用我们的事件处理程序。当前，这个处理程序仅打印出 OSLogEventProxy 对象。

要编译这段代码，我们需要从 *LoggingSupport* 框架中提取的未文档化类和方法定义。这些定义位于 *logStream* 项目的 *LogStream.h* 文件中；清单 6-3 提供了它们的一部分。

```
@interface OSLogEventLiveStream : NSObject
    -(void)activate;
    -(void)setFilterPredicate:(NSPredicate*)predicate;
    -(void)setEventHandler:(void(^)(id))callback;
    ...
    @property(nonatomic) unsigned long long flags;
@end

@interface OSLogEventProxy : NSObject
    @property(readonly, nonatomic) NSString* process;
    @property(readonly, nonatomic) int processIdentifier;
    @property(readonly, nonatomic) NSString* processImagePath;
    ...
@end 
```

清单 6-3：私有的 OSLogEventLiveStream 和 OSLogEventProxy 类的接口

一旦我们编译了这段代码，就可以使用用户指定的谓词来执行它。例如，让我们监视安全子系统的日志消息，*com.apple.securityd*：

```
% **./logStream 'subsystem == "com.apple.securityd"'**
New Log Message:
<OSLogEventProxy: 0x155804080, 0x0, 400, 1300, open(%s,0x%x,0x%x) = %d>
New Log Message:
<OSLogEventProxy: 0x155804080, 0x0, 400, 1300, %p is a thin file (%s)>
New Log Message:
<OSLogEventProxy: 0x155804080, 0x0, 400, 1300, %zd signing bytes in %d blob(s) from %s(%s)>
New Log Message:
<OSLogEventProxy: 0x155804080, 0x0, 400, 1009, network access disabled by policy> 
```

尽管我们确实正在捕获与指定谓词匹配的流式日志消息，但初看这些消息似乎并不那么有用。这是因为我们的事件处理程序仅通过调用其 `description` 方法打印出 OSLogEventProxy 对象，而该方法并不包含消息的所有组成部分。

#### 提取日志对象的属性

为了检测可能表明恶意软件存在的活动，你需要提取 OSLogEventProxy 日志方法对象的属性。在反汇编过程中，我们遇到了几个有用的属性，如进程 ID、路径和消息，但还有其他有趣的属性。由于 Objective-C 是反射性的，你可以动态查询任何对象，包括未文档化的对象，来揭示其属性和值。这需要深入 Objective-C 运行时的内部；尽管如此，你会发现理解你遇到的任何未文档化类是非常有用的，特别是在利用 Apple 的私有框架时。

清单 6-4 是一个简单的函数，它接受任何 Objective-C 对象，然后打印出其属性及其值。它基于 Pat Zearfoss 的代码。^(8)

```
#import <objc/message.h> ❶
#import <objc/runtime.h>

void inspectObject(id object) {
    unsigned int propertyCount = 0 ;
    objc_property_t* properties = class_copyPropertyList([object class], &propertyCount); ❷

    for(unsigned int i = 0; i < propertyCount; i++) {
        NSString* name = [NSString stringWithUTF8String:property_getName(properties[i])]; ❸

        printf("\n%s: ", [name UTF8String]);

        SEL sel = sel_registerName(name.UTF8String); ❹
        const char* attr = property_getAttributes(properties[i]); ❺

        switch(attr[1]) {
            case '@':
                printf("%s\n",
                [[((id (*)(id, SEL))objc_msgSend)(object, sel) description] UTF8String]);
                break;
            case 'i':
                printf("%i\n", ((int (*)(id, SEL))objc_msgSend)(object, sel));
                break;
            case 'f':
                printf("%f\n", ((float (*)(id, SEL))objc_msgSend)(object, sel));
                break;
            default:
                break;
        }
    }

    free(properties);
    return;
} 
```

清单 6-4：检查一个 Objective-C 对象的属性

首先，代码导入所需的 Objective-C 运行时头文件 ❶。然后，它调用 class_copyPropertyList API 来获取对象属性的数组和数量 ❷。我们遍历该数组检查每个属性，调用 property_getName 方法获取属性的名称 ❸。然后，sel_registerName 函数为属性检索选择器 ❹。稍后我们将使用属性选择器来检索对象的值。

接下来，为了确定属性的类型，我们调用属性的 _getAttributes 方法 ❺。这将返回一个属性数组，其中属性类型是第二项（索引为 1）。代码处理常见类型，如 Objective-C 对象（@）、整数（i）和浮点数（f）。对于每种类型，我们在对象上调用 objc_msgSend 函数，并使用属性的选择器来获取属性的值。

如果仔细观察，你会看到调用 objc_msgSend 时，对于每种属性类型，都会适当地进行类型转换。有关类型编码的列表，请参见 Apple 的“类型编码”开发者文档。^(9) 若要检查 Swift 对象，请使用 Swift 的 Mirror API。^(10)

在日志监控代码中，我们现在可以使用 inspectObject 函数，处理从日志子系统接收到的每个 OSLogEventProxy 对象（示例 6-5）。

```
NSPredicate* predicate = [NSPredicate predicateWithFormat:<some string predicate>];

[logMonitor start:predicate level:Log_Level_Debug eventHandler:
^(OSLogEventProxy* event) {
    inspectObject(event);
}]; 
```

示例 6-5：检查每条日志消息，封装在 OSLogEventProxy 对象中

如果我们编译并执行该程序，应该会收到每条日志消息的更全面视图。例如，通过监控与 XProtect 相关的消息（这是某些版本 macOS 上内置的反恶意软件扫描器），我们可以观察它对一个不受信任应用程序的扫描：

```
% **./logStream 'subsystem == "com.apple.xprotect"'**

New Log Message:

composedMessage: Starting malware scan for: /Volumes/Install/Install.app

logType: 1
timeZone: GMT-0700 (GMT-7) offset -25200
...
processIdentifier: 1374
process: XprotectService
processImagePath: /System/Library/PrivateFrameworks/XprotectFramework
.framework/Versions/A/XprotectService.xpc/Contents/MacOS/XprotectService
...
senderImagePath: /System/Library/PrivateFrameworks/XprotectFramework
.framework/Versions/A/XprotectService.xpc/Contents/MacOS/XprotectService
sender: XprotectService
...
subsystem: com.apple.xprotect
category: xprotect
... 
```

精简输出包含了与安全工具最相关的 OSLogEventProxy 对象的属性。表 6-1 按字母顺序总结了这些属性。与许多 OSLogEventProxy 对象属性一样，你可以在自定义谓词中使用它们。

表 6-1：与安全相关的 OSLogEventProxy 属性

| 属性名 | 描述 |
| --- | --- |
| category | 用于记录事件的类别 |
| composedMessage | 日志消息的内容 |
| logType | 对于 logEvent 和 traceEvent，消息的类型（默认、信息、调试、错误或故障） |
| processIdentifier | 导致事件的进程的进程 ID |
| processImagePath | 导致事件的进程的完整路径 |
| senderImagePath | 导致事件的库、框架、内核扩展或 Mach-O 镜像的完整路径 |
| subsystem | 用于记录事件的子系统 |
| type | 事件的类型（如 activityCreateEvent、activityTransitionEvent 或 logEvent） |

#### 确定资源消耗

考虑流式日志消息的潜在资源影响非常重要。如果你采取过度消耗的方式，可能会导致显著的 CPU 开销，并影响系统的响应能力。

首先，注意日志级别。指定调试级别将导致处理的日志消息数量显著增加。尽管谓词评估逻辑非常高效，但更多的消息意味着更多的 CPU 循环。因此，利用日志子系统流式传输功能的安全工具最好还是只消耗默认或信息级别的消息。

同样重要的是你使用的谓词效率。有趣的是，我的实验表明，日志守护进程会完全评估某些谓词，而在客户端程序中加载的日志子系统框架（如日志监视器）会处理其他谓词。前者更好；否则，程序将收到每一条日志消息的副本以进行谓词评估，这可能会占用大量的 CPU 循环。如果是日志守护进程进行谓词评估，你将只收到匹配谓词的消息，这对系统的影响几乎不可察觉。

如何创建一个日志守护进程会评估的谓词？经过反复试验表明，如果你在谓词中指定进程或子系统，守护进程就会评估它，这意味着你只会收到匹配的日志消息。我们来看一个来自 OverSight 的具体例子，OverSight 是在第十二章中讨论的一款工具，用于监控麦克风和摄像头。^(11)

OverSight 需要访问核心媒体 I/O 子系统的日志消息，以识别访问摄像头的进程。在本章开始时，我提到某些版本的 macOS 会将此进程 ID 存储在包含字符串 CMIOExtensionPropertyDeviceControlPID 的核心媒体 I/O 子系统的日志消息中。可以理解的是，你可能会想创建一个与此字符串匹配的谓词：

```
'composedMessage CONTAINS "CMIOExtensionPropertyDeviceControlPID"'
```

然而，这个谓词会导致处理效率低下，因为日志守护进程将发送所有由我们日志监视器加载的日志框架进行谓词过滤的消息。相反，OverSight 利用一个更广泛的谓词，利用了子系统属性：

```
subsystem=='com.apple.cmio'
```

这种方法使日志守护进程执行谓词匹配，然后只传送来自核心媒体 I/O 子系统的消息。OverSight 本身手动执行对 CMIOExtensionPropertyDeviceControlPID 字符串的检查：

```
if(YES == [logEvent.composedMessage
containsString:@"CMIOExtensionPropertyDeviceControlPID ="]) {
    // Extract the PID of the processes accessing the webcam.
} 
```

该工具利用类似的过程返回与麦克风访问相关的日志消息。因此，它能够有效地检测任何尝试使用麦克风或摄像头的进程（包括恶意软件）。

### 结论

在本章中，你了解了如何使用代码与操作系统的通用日志子系统进行交互。通过逆向工程私有的 *LoggingSupport* 框架，我们以编程方式流式传输与自定义谓词匹配的消息，并访问日志子系统中丰富的数据。安全工具可以利用这些信息来检测新的感染，甚至揭示持久性恶意软件的恶意行为。

在下一章中，你将使用 Apple 强大且文档完善的网络扩展来编写网络监控逻辑。

### 备注

1.    1.  Nicole Fishbein 和 Avigayil Mechtinger, “风暴即将来临：IPStorm 现已拥有 Linux 恶意软件，”Intezer，2023 年 11 月 14 日，[*https://<wbr>www<wbr>.intezer<wbr>.com<wbr>/blog<wbr>/research<wbr>/a<wbr>-storm<wbr>-is<wbr>-brewing<wbr>-ipstorm<wbr>-now<wbr>-has<wbr>-linux<wbr>-malware<wbr>/*](https://www.intezer.com/blog/research/a-storm-is-brewing-ipstorm-now-has-linux-malware/).

1.    2.  “XCSSET 恶意软件，”TrendMicro，2020 年 8 月 13 日，[*https://<wbr>documents<wbr>.trendmicro<wbr>.com<wbr>/assets<wbr>/pdf<wbr>/XCSSET<wbr>_Technical<wbr>_Brief<wbr>.pdf*](https://documents.trendmicro.com/assets/pdf/XCSSET_Technical_Brief.pdf)。欲了解更多关于 macOS 中远程登录滥用的信息，请参见 Jaron Bradley， “macOS 上的 APT 活动是什么样的？”，“*The Mitten Mac*”，2021 年 11 月 14 日，[*https://<wbr>themittenmac<wbr>.com<wbr>/what<wbr>-does<wbr>-apt<wbr>-activity<wbr>-look<wbr>-like<wbr>-on<wbr>-macos<wbr>/*](https://themittenmac.com/what-does-apt-activity-look-like-on-macos/).

1.    3.  Calum Hall 和 Luke Roberts, “时钟正在 TCC 中，”论文发表于 Objective by the Sea v6，西班牙，2023 年 10 月 12 日，[*https://<wbr>objectivebythesea<wbr>.org<wbr>/v6<wbr>/talks<wbr>/OBTS<wbr>_v6<wbr>_lRoberts<wbr>_cHall<wbr>.pdf*](https://objectivebythesea.org/v6/talks/OBTS_v6_lRoberts_cHall.pdf).

1.    4.  “日志记录，”Apple 开发者文档，[*https://<wbr>developer<wbr>.apple<wbr>.com<wbr>/documentation<wbr>/os<wbr>/logging*](https://developer.apple.com/documentation/os/logging).

1.    5.  Howard Oakley, “如何在日志中显示‘私密’信息，”Eclectic Light，2020 年 5 月 25 日，[*https://<wbr>eclecticlight<wbr>.co<wbr>/2020<wbr>/05<wbr>/25<wbr>/how<wbr>-to<wbr>-reveal<wbr>-private<wbr>-messages<wbr>-in<wbr>-the<wbr>-log<wbr>/*](https://eclecticlight.co/2020/05/25/how-to-reveal-private-messages-in-the-log/).

1.    6.  请参阅 Howard Oakley，“日志：谓词简介，”Eclectic Light，2016 年 10 月 17 日，[*https://<wbr>eclecticlight<wbr>.co<wbr>/2016<wbr>/10<wbr>/17<wbr>/log<wbr>-a<wbr>-primer<wbr>-on<wbr>-predicates<wbr>/*](https://eclecticlight.co/2016/10/17/log-a-primer-on-predicates/)，以及“谓词编程指南，”Apple 开发者文档，[*https://<wbr>developer<wbr>.apple<wbr>.com<wbr>/library<wbr>/archive<wbr>/documentation<wbr>/Cocoa<wbr>/Conceptual<wbr>/Predicates<wbr>/AdditionalChapters<wbr>/Introduction<wbr>.html*](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Predicates/AdditionalChapters/Introduction.html).

1.    7.  “OSLog，”Apple 开发者文档，[*https://<wbr>developer<wbr>.apple<wbr>.com<wbr>/documentation<wbr>/oslog*](https://developer.apple.com/documentation/oslog).

1.    8.  Pat Zearfoss，“Objective-C Quickie: 打印对象的所有声明属性，”2011 年 4 月 14 日，[*https://<wbr>zearfoss<wbr>.wordpress<wbr>.com<wbr>/2011<wbr>/04<wbr>/14<wbr>/objective<wbr>-c<wbr>-quickie<wbr>-printing<wbr>-all<wbr>-declared<wbr>-properties<wbr>-of<wbr>-an<wbr>-object<wbr>/*](https://zearfoss.wordpress.com/2011/04/14/objective-c-quickie-printing-all-declared-properties-of-an-object/)。

1.    9.  该列表可在 [*https://<wbr>developer<wbr>.apple<wbr>.com<wbr>/library<wbr>/archive<wbr>/documentation<wbr>/Cocoa<wbr>/Conceptual<wbr>/ObjCRuntimeGuide<wbr>/Articles<wbr>/ocrtTypeEncodings<wbr>.html#<wbr>/<wbr>/apple<wbr>_ref<wbr>/doc<wbr>/uid<wbr>/TP40008048<wbr>-CH100<wbr>-SW1*](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html#//apple_ref/doc/uid/TP40008048-CH100-SW1) 查阅。

1.  10.  在 Antoine van der Lee 的文章“Swift 中的反射：Mirror 如何工作”中，了解更多关于 Swift 的 Mirror API，*SwiftLee*，2021 年 12 月 21 日，[*https://<wbr>www<wbr>.avanderlee<wbr>.com<wbr>/swift<wbr>/reflection<wbr>-how<wbr>-mirror<wbr>-works<wbr>/*](https://www.avanderlee.com/swift/reflection-how-mirror-works/)。

1.  11.  请参阅 [*https://<wbr>objective<wbr>-see<wbr>.org<wbr>/products<wbr>/oversight<wbr>.html*](https://objective-see.org/products/oversight.html)。
