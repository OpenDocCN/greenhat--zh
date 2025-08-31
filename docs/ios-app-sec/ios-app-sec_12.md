## 第九章：9

**面向 iOS 的 Web 应用**

自从为 iOS 引入第三方开发者 API 以来，Web 已成为 iOS 应用程序的重要组成部分。最初，这些 API 完全是基于 Web 的。虽然这可能让没有 Objective-C 或 Cocoa 经验的人生活更轻松，但它严重限制了非苹果应用的功能，并将它们 relegated 到了二等公民的地位。它们无法访问手机的本地功能，如地理定位，而且只能在浏览器中使用，而无法放置在主屏幕上。

尽管从那时起情况发生了剧烈变化，但从 iOS 集成 Web 应用程序的需求并没有改变。在本章中，你将深入了解本地 iOS 应用程序与 Web 应用程序之间的联系：如何与 Web 应用程序交互，哪些本地 iOS API 可以暴露给 Web 应用程序，以及各种方法的风险。

### 使用 (及滥用) UIWebViews

开发者使用 Web 视图来渲染和与 Web 内容交互，因为它们简单易实现，并提供类似浏览器的功能。大多数 Web 视图是 `UIWebView` 类的实例，使用 WebKit 渲染引擎^(1) 来显示 Web 内容。Web 视图常用于将应用程序的部分逻辑抽象化，以便在不同的移动应用平台之间共享，或者仅仅是将更多的逻辑卸载到 Web 应用程序，通常是因为 Web 应用编程方面的内部技术比 iOS 更为熟练。它们还经常用来作为查看第三方 Web 内容的方式，而无需离开应用并启动 Safari。例如，当你在 Facebook 动态中点击一篇文章时，内容会在 Facebook 应用中渲染。

从 iOS 8 开始，引入了 `WKWebView` 框架。这个框架为开发者提供了额外的灵活性，并可以访问苹果的高性能 Nitro JavaScript 引擎，从而显著提升使用 Web 视图的应用性能。由于你将继续看到 `UIWebView`，本章将首先介绍 `UIWebView`，并探讨两个 API 的使用。

#### *使用 UIWebViews*

Web 视图将应用程序的一部分逻辑转移到远程 Web API 或应用程序。因此，开发者对 Web 视图的控制程度较本地 iOS 应用程序低，但你可以采取一些控制措施，以让 Web 视图符合你的需求。

通过实现协议 `UIWebViewDelegate` 的 `shouldStartLoadWithRequest` 方法^(2)，你可以在允许 URL 被打开之前，决定是否打开所有通过 Web 视图打开的 URL。例如，为了限制攻击面，你可以限制所有请求，只允许它们访问 HTTPS URL 或某些特定域名。如果你希望确保你的应用永远不会加载非 HTTPS 的 URL，你可以像 Listing 9-1 示例中那样进行操作。

```
   - (BOOL)webView:(UIWebView*)webView shouldStartLoadWithRequest:(NSURLRequest*)
        request
    navigationType:(UIWebViewNavigationType)navigationType {

       NSURL *url = [request URL];

➊     if ([[url scheme] isEqualToString:@"https"]) {

           if ([url host] != nil) {
               NSString *goodHost = @"happy.fluffy.bunnies.com";

➋             if ([[url host] isEqualToString:goodHost]) {
                   return YES;
               }
           }
       }
       return NO;
   }
```

*列表 9-1：拒绝非 HTTPS URL 和未知主机名*

这个示例使用了与正在加载的 `NSURLRequest` 关联的 `NSURL` 的两个不同属性。在 ➊，检查 URL 的 `scheme` 属性，看它是否与指定的 `https` 协议匹配。在 ➋，将 `host` 属性与一个白名单域进行比较：*happy.fluffy.bunnies.com*。这两个限制将应用程序的 Web 视图访问仅限于您的域名，而不是任何可能被攻击者控制的域，并确保请求始终通过 HTTPS 传输，从而保护其内容免受网络攻击者的侵害。

Web 视图看起来是个不错的选择，因为它可以跨平台重用代码库，同时对本地系统保持一定的控制。然而，Web 视图确实存在一些严重的安全隐患。一个限制是无法升级随 `UIWebView` 一起打包的 WebKit 二进制文件。WebKit 是与 iOS 新版本一起打包的，并且不会从主操作系统中单独更新。这意味着，任何发现的 WebKit 漏洞在发布新版本的 iOS 之前都无法修复。

安全使用 Web 视图的另一个重要方面是妥善处理缓存数据，我将在下一节讨论这个问题。

#### *在 UIWebViews 中执行 JavaScript*

Web 视图的 JavaScript 引擎称为 JavaScriptCore，也由 Apple 称为 Nitro。虽然新的 `WKWebView` 类改进了 JavaScript 支持（请参见 “进入 WKWebView” 页 158），但与现代浏览器中的 JavaScript 引擎相比，`UIWebView` 中使用的 JavaScriptCore 实现存在一些不足之处。主要的限制是缺少即时编译（JIT）。

`UIWebView` 的 JavaScript 执行还将总分配限制为 10MB，并且运行时间限制为 10 秒，超过该时间点，执行将立即且无条件地停止。尽管有这些限制，应用程序仍然可以通过将脚本传递给 `stringByEvaluatingJavaScriptFromString` 来执行有限的 JavaScript，如列表 9-2 所示。

```
[webView stringByEvaluatingJavaScriptFromString:@"var elem =
    document.createElement('script');"
        "elem.type = 'text/javascript';"
        "elem.text = 'aUselessFunc(name) {"
        "       alert('Ohai!'+name);"
        "};"
        "document.getElementById('head').appendChild(elem);"];
[webView stringByEvaluatingJavaScriptFromString:@"aUselessFunc('Mitch');"];
```

*列表 9-2：将 JavaScript 注入到 Web 视图中*

`stringByEvaluatingJavaScriptFromString` 方法接受一个参数，这是一个 JavaScript 代码块，用于插入到视图中。这里，创建了元素 `elem`，定义了一个简单的函数来生成一个警告框，并将该函数插入到 web 视图中。现在，可以通过后续调用 `stringByEvaluatingJavaScriptFromString` 来调用新定义的函数。

然而，请注意，允许在应用中动态执行 JavaScript 会使用户面临 JavaScript 注入攻击的风险。因此，应谨慎使用此功能，开发者切勿将不可信的输入反射到动态生成的脚本中。

你将在下一节中了解更多关于 JavaScriptCore 的内容，届时我将讨论如何绕过我之前提到的`UIWebView`的不足。

### JavaScript-Cocoa 桥接的奖励与风险

为了克服`UIWebView`的限制，开发者使用了各种变通方法，将更多的原生功能暴露给基于网页的应用。例如，Cordova 开发框架通过巧妙（或危险）的网页视图实现，访问 Cocoa API，允许使用相机、加速计、地理定位功能、通讯录等。

在本节中，我将向你介绍一些流行的 JavaScript-Cocoa 桥接，提供它们在实际应用中的使用示例，并讨论它们带来的一些安全风险。

#### *与 JavaScriptCore 进行接口交互*

在 iOS 7 之前，`[UIWebView stringByEvaluatingJavaScriptFromString:]`是应用程序内部调用 JavaScript 的唯一方法。然而，iOS 7 发布了 JavaScriptCore 框架，它完全支持原生 Objective-C 和 JavaScript 运行时之间的桥接通信。该桥接通过新的`JSContext`全局对象创建，提供了访问 JavaScript 虚拟机以评估代码的能力。Objective-C 运行时还可以通过`JSValue`对象获取对 JavaScript 值的强引用。

你可以通过两种基本方式使用 JavaScriptCore 与 JavaScript 运行时进行交互：使用内联块或通过`JSExport`协议直接暴露 Objective-C 对象。我将简要介绍这两种方法的工作原理，然后讨论这种新攻击面带来的安全问题。

##### 直接暴露 Objective-C 块

Objective-C 块的一种用途是提供一个简单的机制，将 Objective-C 代码暴露给 JavaScript。当你将 Objective-C 块暴露给 JavaScript 时，框架会自动将其包装为一个可调用的 JavaScript 函数，这样你就可以直接从 JavaScript 调用 Objective-C 代码。让我们来看一个例子——尽管这是一个假设的例子——在 Listing 9-3 中。

```
   JSContext *context = [[JSContext alloc] init];
➊ context[@"shasum"] = ^(NSString *data, NSString *salt) {
       const char *cSalt = [salt cStringUsingEncoding:NSUTF8StringEncoding];
       const char *cData = [data cStringUsingEncoding:NSUTF8StringEncoding];
       unsigned char digest[CC_SHA256_DIGEST_LENGTH];
       CCHmac(kCCHmacAlgSHA256, cSalt, strlen(cSalt), cData, strlen(cData),
        digest);
       NSMutableString *hash = [NSMutableString stringWithCapacity:
        CC_SHA256_DIGEST_LENGTH];
       for (int i = 0; i < CC_SHA256_DIGEST_LENGTH; i++) {
           [hash appendFormat:@"%02x", digest[i]];
       }
       return hash;
   };
```

*Listing 9-3: 将 Objective-C 块暴露给 JavaScript*

在这里，暴露了一个块（你可以看到它由`^`操作符在➊处定义），该块接收 JavaScript 中的密码和盐值，并使用 Common-Crypto 框架创建哈希。然后可以直接从 JavaScript 访问此块，生成用户的密码哈希，如 Listing 9-4 所示。

```
var password = document.getElementById('password');
var salt = document.getElementById('salt');
var pwhash = shasum(password, salt);
```

*Listing 9-4: 调用暴露的 Objective-C 块的 JavaScript*

这种技术使你能够利用 Cocoa Touch API，避免重新实现诸如加密或哈希等复杂且容易出错的操作。

Blocks 是将 Objective-C 代码暴露给 JavaScript 的最简单方式，但它们也有一些缺点。例如，所有桥接的对象都是不可变的，因此改变 Objective-C 变量的值不会影响与之映射的 JavaScript 变量。然而，如果你确实需要在两个执行上下文之间共享对象，你也可以使用`JSExport`协议暴露自定义类。

##### 通过 JSExport 连接 Objective-C 和 JavaScript

`JSExport`协议允许应用程序将整个 Objective-C 类和实例暴露给 JavaScript，并像操作 JavaScript 对象一样对它们进行操作。此外，它们与 Objective-C 对应对象的引用是强引用，这意味着在一个环境中对对象的修改会反映到另一个环境中。定义继承了`JSExport`的协议中的变量和方法，向 JavaScriptCore 表明这些元素可以从 JavaScript 访问，如列表 9-5 所示。

```
@protocol UserExports <JSExport>

//exported variables
@property NSString *name;
@property NSString *address;

//exported functions
- (NSString *) updateUser:(NSDictionary *)info;
@end
```

*列表 9-5：使用白名单方法暴露变量和方法*

多亏了`JSExport`协议声明，JavaScript 可以访问变量`name`和`address`以及函数`updateUser`。苹果使得将这些对象暴露给 JavaScriptCore 变得非常简单，这也意味着开发者很容易不小心暴露出各种不必要的功能。幸运的是，这座桥接遵循完全可选择加入的模式：只有你在协议中实际定义的成员才会被暴露。除非在协议定义中明确列入白名单，否则在类接口中做的任何额外声明都会被隐藏，就像在列表 9-6 中`password`属性的情况一样。

```
@interface User : NSObject <UserExports> ➊

// non-exported variable
@property NSString *password;

// non-exported method declaration
- (BOOL) resetPassword;
@end
```

*列表 9-6：在协议定义之外声明的元素在 JavaScript 中不可访问*

`User`接口在➊处继承了`UserExports`，因此它也继承了`JSExport`。但是，`password`属性和`resetPassword`方法并未在`UserExports`中声明，因此它们不会暴露给 JavaScript。

现在，JavaScriptCore 了解了你的`UserExports`协议，它可以在你将其实例添加到`JSContext`时创建一个合适的包装对象，如下一个列表所示：

```
➊ JSContext *context = [[JSContext alloc] init];
➋ User *user= [[User alloc] init];
   [user setName:@"Winston Furchill"];
   [user setValue:24011965];
   [user setHiddenName:@"Enigma"];
➌ context[@"user"] = user;
➍ JSValue val = [context evaluateScript:@"user.value"];
➎ JSValue val = [context evaluateScript:@"user.hiddenName"];
   NSLog(@"value: %d", [val toInt32]); // => 23011965
   NSLog(@"hiddenName: %@", [val toString]); // => undefined
```

这里，`JSContext`在➊处被设置，`User`类的一个实例在➋处被创建，并且给新用户的三个属性赋值。其中一个属性`hiddenName`只在`@implementation`中定义，而不是在协议中定义——这与列表 9-6 中`password`属性的情况相同。在➌处，创建的用户被桥接到`JSContext`。当代码随后尝试从 JavaScript 访问用户对象的值时，`value`属性在➍处成功访问，而访问`hiddenName`的尝试在➎处失败。

**注意**

*在将对象导出到 JavaScriptCore 时要谨慎。如果攻击者利用了脚本注入漏洞，就可以运行任何导出的函数，本质上将脚本注入转变为在用户设备上的本地远程代码执行。*

另一个有趣的点是，JavaScriptCore 不允许调用导出的类构造函数。（这是 iOS 中的一个 bug，截至 iOS 8，尚未解决。）因此，即使你将`[User class]`添加到你的上下文中，也无法使用`new`来创建新对象。然而，正如我通过一些测试发现的那样，实际上是可以绕过这个限制的。你可以实现一个导出的 Objective-C 块，该块接受一个类名，然后创建并返回一个任意类的实例给 JavaScript，正如我在这里所做的：

```
self.context[@"newInstance"] = ^(NSString *className) {
    Class clazz = NSClassFromString(className);
    id inst = [clazz alloc];
    return inst;
};

[self.context evaluateScript:@"var u = newInstance('User');"];
JSValue *val = self.context[@"u"];
User *user = [val toObject];
NSLog(@"%@", [user class]); // => User
```

这种技术绕过了显式导出任何类的需求，并允许你实例化任何类型的对象并将其暴露给 JavaScript-Core。然而，没有成员被列入白名单以供导出，因此没有对类对象的任何方法或变量的强引用。显然，对于绕过 JavaScriptCore 实现的限制，仍然有很大的安全研究空间，因为 Objective-C 运行时是如此动态且强大。

关于 JavaScriptCore 框架的一个常见抱怨是，没有文档说明如何访问`UIWebView`的`JSContext`。我接下来会讨论一些可能的解决方法。

##### 在 Web 视图中操作 JavaScript

为什么要在没有访问 Web 视图内`JSContext`的方式下暴露这种`JSContext`功能？苹果的意图尚不明确，但开发者只完成了一半的工作，即文档化 JavaScriptCore API。到目前为止，苹果并没有提供官方的方式来操作`UIWebView`的`JSContext`，但已有几个人发现了方法来实现这一点。它们大多数都涉及使用`valueForKeyPath`方法，正如在清单 9-7 中所示。

```
- (void)webViewDidFinishLoad:(UIWebView *)webView {
      JSContext *context = [webView valueForKeyPath:@"documentView.webView.
     mainFrame.javaScriptContext"];

      context[@"document"][@"cookie"] = @"hello, I'm messing with cookies";
    }
```

*清单 9-7：通过 Objective-C 操作 DOM*

由于这不是苹果官方认可的方法，因此无法保证此类代码能够通过 App Store 审核，但了解开发者可能会尝试在 JavaScript 与 Objective-C 之间进行通信的方式及其可能带来的问题是很有必要的。

当然，`JSContext`并不是将 JavaScript 与 Objective-C 连接的唯一方式。我将在下一节中描述另一种流行的桥接方式——Cordova。

#### *使用 Cordova 执行 JavaScript*

Cordova（在 Adobe 收购开发公司 Nitobi 之前称为 PhoneGap）是一个 SDK，它以平台无关的方式将原生移动 API 提供给 Web 视图的 JavaScript 执行环境。这使得可以像标准 Web 应用程序一样使用 HTML、CSS 和 JavaScript 开发移动应用程序。这些应用程序随后可以在 Cordova 支持的所有平台上运行。这可以显著减少开发时间，并且无需开发公司聘请特定平台的工程师，但 Cordova 的实现显著增加了应用程序的攻击面。

##### Cordova 的工作原理

Cordova 通过实现 `NSURLProtocol` 来桥接 JavaScript 和 Objective-C，以处理任何 JavaScript 发起的 `XmlHttpRequest` 到 *file://!gap_exec*。如果原生的 Cordova 库检测到对此 URI 的调用，它将尝试从请求头中提取类、方法、参数和回调信息，正如示例 9-8 所证明的那样。

```
   + (BOOL)canInitWithRequest:(NSURLRequest*)theRequest {
       NSURL* theUrl = [theRequest URL];
       CDVViewController* viewController = viewControllerForRequest(theRequest);

       if ([[theUrl absoluteString] hasPrefix:kCDVAssetsLibraryPrefixs]) {
           return YES;
       } else if (viewController != nil) {
➊          if ([[theUrl path] isEqualToString:@"/!gap_exec"]) {
➋              NSString* queuedCommandsJSON = [theRequest valueForHTTPHeaderField:@"
        cmds"];
               NSString* requestId = [theRequest valueForHTTPHeaderField:@"rc"];
               if (requestId == nil) {
                   NSLog(@"!cordova request missing rc header");
                   return NO;
               }
               BOOL hasCmds = [queuedCommandsJSON length] > 0;
               if (hasCmds) {
                   SEL sel = @selector(enqueCommandBatch:);
➌                 [viewController.commandQueue performSelectorOnMainThread:sel
       withObject:queuedCommandsJSON waitUntilDone:NO];
```

*示例 9-8：在* CDVURLProtocol.m^(3) *中检测原生库调用*

在➊处，请求的 URL 被检查是否包含路径组件*/!gap_exec*，在➋处，提取 `cmds` HTTP 头部的值。然后，Cordova 将这些命令传递到命令队列➌，在那里它们将尽可能地被执行。当这些命令被排队时，Cordova 会在可用的 Cordova 插件映射中查找相关信息，这些插件本质上只是暴露了原生功能的各个部分，并且可以任意扩展。如果某个特定插件被启用，并且请求中的类可以实例化，那么将使用强大的 `objc_msgSend` 方法调用该方法，并传入提供的参数。

当调用完成时，原生代码通过 `[UIWebView stringByEvaluatingJavaScriptFromString]` 回调到 JavaScript 运行时，调用在 *cordova.js* 中定义的 `cordova.require('cordova/exec').nativeCallback` 方法，并提供原始回调 ID 以及原生代码执行的返回值。

这将前所未有地将大量原生对象控制权导出到 JavaScript 运行时，允许应用程序读取和写入文件，读取和写入钥匙串存储，通过 FTP 将本地文件上传到远程服务器等等。但随着功能的增加，也带来了潜在的风险。

##### 使用 Cordova 的风险

如果你的应用程序包含任何脚本注入漏洞，并且用户能够影响应用程序的导航，攻击者就可能获得远程代码执行的机会。他们只需要注入回调函数，并结合调用来启动与原生代码的通信。例如，攻击者可能会注入一个调用来访问钥匙串中的项，获取所有用户联系人信息的副本，或读取文件并将其传递给他们选择的 JavaScript 函数，如示例 9-9 所示。

```
<script type="text/javascript">
    var exec = cordova.require('cordova/exec');
    function callback(msg) {
        console.log(msg);
    }
    exec(callback, callback, "File", "readAsText", ["/private/var/mobile/Library/
     Preferences/com.apple.MobileSMS.plist", "UTF-8",
        0, 2048]);
</script>
```

*示例 9-9：使用 Cordova 调用 Objective-C 来读取文件内容*

这个攻击者提供的 JavaScript 读取了设备的*com.apple.MobileSMS.plist*，在 iOS 8 中，该文件对设备上的所有应用程序都是可访问的。^(4) 这使得攻击者能够检查用户的联系人信息，并确定设备的所有者。

一项合理的内建安全措施是*域名白名单*，它可以显著降低脚本注入的风险。^(5) Cordova 的默认安全策略阻止所有网络访问，仅允许与应用配置中 `<access>` 元素下的白名单域进行交互。白名单确实允许通过通配符（[*]）访问所有域，但不要懒惰—确保白名单中只有应用程序正常工作所需的域。您可以通过 Xcode 配置此项，通过向 `Cordova.plist` 中的 `ExternalHosts` 键添加值，如图 9-1 所示。

![image](img/f09-01.jpg)

*图 9-1：使用 `ExternalHosts` 键在 Cordova 中进行域名白名单配置*

除了将本地代码对象暴露给网页视图之外，使用像 Cordova 这样的网页平台封装来实现移动应用程序还有许多其他缺点。主要是每个平台都有其特定的安全模型，基于特定的假设、API 和功能来保护用户并保障本地存储安全。一个平台的安全模型在其他平台上根本行不通。提供一个一刀切的实现，必然会为了易用性而忽视一些平台特有的安全优势。

例如，iOS 通过数据保护 API 提供安全存储（如我在第十三章中描述的），这些 API 需要特定的参数，这些参数不适合跨平台实现。因此，Cordova 不支持这些 API，从而无法精细控制何时对文件数据进行加密存储。为了解决这个问题，您可以启用权限级别的数据保护（参考“DataProtectionClass Entitlement”和第 223 页），这将为应用程序写入磁盘的所有数据应用一个默认的保护级别。

另一个常见问题是缺乏跨平台的类似安全存储元素。这意味着在 iOS 上无法直接访问 Keychain，尽管 Adobe 最终开发了一个开源插件^(6) 来解决这个问题。

这就是 `UIWebView` 和 JavaScript 桥接的全部内容，但新的应用程序（针对 iOS 8 及更新版本）将越来越多地使用 `WKWebView` API。我将在接下来的章节中介绍如何处理 `WKWebView`。

### 引入 WKWebView

如前所述，iOS 8 引入了一个新的 WebKit 接口来替代 `UIWebView`。`WKWebView` 解决了 `UIWebView` 的几个缺点，包括访问 Nitro JavaScript 引擎，这大大提高了 JavaScript 密集型任务的性能。让我们来看一下应用程序如何创建 `WKWebView`，以及 `WKWebView` 如何提升应用程序的安全性。

#### *与 WKWebView 的互动*

`WKWebView` 的实例化方式与 `UIWebView` 基本相同，如下所示：

```
CGRect webFrame = CGRectMake(0, 0, width, height);
WKWebViewConfiguration *conf = [[WKWebViewConfiguration alloc] init];
WKWebView *webView =[[WKWebView alloc] initWithFrame:webFrame
                                       configuration:conf];
NSURL *url = [NSURL URLWithString:@"http://www.nostarch.com"];
NSURLRequest *request = [NSURLRequest requestWithURL:url];
[webView loadRequest:request];
```

这仅仅分配了一个新的 `WKWebView` 实例，然后通过 `initWithFrame` 方法对其进行初始化。

要自定义行为，`WKWebView` 还可以通过用户提供的 JavaScript 来实例化，如 示例 9-10 中所示。这允许你加载一个第三方网站，同时执行你自己自定义的 JavaScript 脚本。

```
   CGRect webFrame = CGRectMake(0, 0, width, height);
➊ NSString *src = @"alert('Welcome to my WKWebView!')";
➋ WKWebViewConfiguration *conf = [[WKWebViewConfiguration alloc] init];
➌ WKUserScript *script = [[WKUserScript alloc] initWithSource:src
           injectionTime:WKUserScriptInjectionTimeAtDocumentStart
        forMainFrameOnly:YES];
➍ WKUserContentController *controller = [[WKUserContentController alloc] init];
➎ [conf setUserContentController:controller];
➏ [controller addUserScript:script];
➐ WKWebView *webView =[[WKWebView alloc] initWithFrame:webFrame
                                              configuration:conf];
```

*示例 9-10：带自定义 JavaScript 的* `WKWebView` *实例化*

在 ➊ 处，创建了一个由单一 JavaScript 命令组成的简单 `NSString`。在 ➋ 处，创建了一个配置对象，用于保存稍后创建的 Web 视图的配置参数。在 ➌ 处，创建并初始化了一个 `WKUserScript` 对象，该对象的 `src` 包含你希望执行的 JavaScript。然后，在 ➍ 处创建了一个 `WKUserContentController`，并在 ➎ 处将其设置到配置对象中。最后，在 ➏ 处通过 `addUserScript` 方法将脚本添加到控制器中，并在 ➐ 处实例化 Web 视图。

**注意**

*与其他注入 JavaScript 的方法一样，务必小心不要在没有严格清理的情况下插入第三方提供的内容。*

#### *WKWebView 的安全优势*

使用 `WKWebView` 有几个安全优势。首先，如果你计划加载的页面不需要 JavaScript，可以通过 `setJavaScriptEnabled` 方法禁用加载 JavaScript；如果远程站点包含恶意脚本，这将防止该脚本执行。你还可以启用 JavaScript，但通过 `setJavaScriptCanOpenWindowsAutomatically` 方法禁用从 JavaScript 打开新窗口——这样可以防止大多数弹出窗口，这在 Web 视图中非常烦人。

最后，可能最重要的一点是，你实际上可以检测 Web 视图的内容是否通过 HTTPS 加载，从而确保页面的任何部分都没有通过不安全的渠道加载。对于 `UIWebView`，当 Web 视图加载混合内容时，用户或开发者并不会收到任何提示——而 `WKWebView` 的 `hasOnlySecureContent` 方法解决了这个问题。示例 9-11 展示了一种实现相对安全的 `WKWebView` 的方法。

```
   @interface ViewController ()
   @property (strong, nonatomic) WKWebView *webView;

   @end

   @implementation ViewController

   - (void)viewDidLoad {
       [super viewDidLoad];

➊     WKPreferences *pref = [[WKPreferences alloc] init];
       [pref setJavaScriptEnabled:NO];
       [pref setJavaScriptCanOpenWindowsAutomatically:NO];

➋     WKWebViewConfiguration *conf = [[WKWebViewConfiguration alloc] init];
       [conf setPreferences:pref];

➌     NSURL *myURL = [NSURL URLWithString:@"https://people.mozilla.org/~mkelly/
       mixed_test.html"];

➍     _webView = [[WKWebView alloc] initWithFrame:[[self view] frame]
                                     configuration:conf];

       [_webView setNavigationDelegate:self];
➎     [_webView loadRequest:[NSURLRequest requestWithURL:myURL]];
       [[self view] addSubview:_webView];
   }

➏ - (void)webView:(WKWebView *)webView didFinishNavigation:(WKNavigation *)navigation
   {
       if (![webView hasOnlySecureContent]) {

           NSString *title = @"Ack! Mixed content!";
           NSString *message = @"Not all content on this page was loaded securely.";
           UIAlertView *alert = [[UIAlertView alloc] initWithTitle:title
                                                            message:message
                                                           delegate:self
                                                  cancelButtonTitle:@"OK"
                                                  otherButtonTitles:nil];
           [alert show];
       }
   }
```

*示例 9-11：一个安全的* `WKWebView`

这段代码使用了`WKWebView`提供的几个额外安全机制。在 ➊ 处，实例化了一个 `WKPreferences` 实例，并设置了 `setJavaScriptEnabled` 和 `setJavaScriptCanOpenWindowsAutomatically` 属性。（当然，这些是多余的，你可以选择最适合你需求的属性。）然后，在 ➋ 处实例化了一个 `WKWebViewConfiguration` 对象，并传入之前创建的 `WKPreferences`。在 ➌ 处，定义了要加载的 URL；在这个例子中，它只是一个包含混合内容的示例页面。在 ➍ 处，使用之前创建的配置生成了 `WKWebView` 实例。然后，代码请求在 ➎ 处加载给定的 URL。最后，在 ➏ 处实现了 `didFinishNavigation` 委托方法，该方法随后调用了网页视图的 `hasOnlySecureContent`。如果内容是混合的，用户会收到警告。

### 总结思考

尽管现代版本的 iOS 在允许开发者控制原生代码和网页内容之间的交互方面取得了很大进展，但仍存在一些遗留的黑客方式来桥接这二者，并且这些方式各有其独特性。此时，你应该了解主要的桥接机制，以及在哪里寻找潜在的恶意外部提供数据。

我还简要介绍了在处理网页内容时发生的一些缓存。在 第十章中，你将深入探讨数据如何泄露到本地文件系统，并被攻击者恢复的多种方式。
