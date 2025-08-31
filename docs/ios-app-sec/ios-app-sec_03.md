## 第二章：2

**懒人版 Objective-C**

Objective-C 在其辉煌的历程中既遭遇过嘲笑也获得过赞誉。它通过 NeXTStep 获得了流行，并且受 Smalltalk 设计的启发，Objective-C 是 C 的超集。它最显著的特点是使用中缀表示法和极其冗长的类名。人们往往要么爱它，要么恨它。那些恨它的人是错的。

在本章中，我将介绍 Objective-C 的基础知识，假设你已经熟悉某种语言的编程。然而，需要注意的是，Cocoa 和 Objective-C 在不断变化。我无法在一章中充分覆盖它们的所有细节，但我会提供一些提示，帮助非开发者在查看 Objective-C 代码时能够定位方向。如果你从很少的编程知识开始，可能希望先阅读一本像 Knaster、Malik 和 Dalrymple 合著的 *Learn Objective-C on the Mac: For OS X and iOS*（Apress, 2012）一书，然后再深入学习。

尽管我很想坚持使用最现代的 Objective-C 编码模式，但如果你在审核现有代码时，可能会遇到大量来自 iOS 初期的陈旧、重复使用的代码。所以为了以防万一，我会讲解一些历史上使用的 Objective-C 构造以及现在被认可的版本。

### 关键的 iOS 编程术语

有一些术语你需要熟悉，以便理解 Apple 各种 API 的来源。*Cocoa* 是指在 Objective-C GUI 编程中使用的框架和 API 的总称。*Cocoa Touch* 是 Cocoa 的超集，包含一些与移动相关的 API，如处理手势和移动 GUI 元素。*Foundation* 类是构成我们所说的 Cocoa API 的大量 Objective-C 类。*Core Foundation* 是一个更底层的基于 C 的库，许多 Foundation 类都是基于它的，通常以 `CF` 而不是 `NS` 为前缀。

### 传递消息

理解 Objective-C 的第一个关键是明白该语言围绕 *消息传递* 的概念设计，而不是 *调用*。对我来说，思考 Objective-C 为一个对象在拥挤的房间里彼此大声喊叫的语言，而不是一个层级导演对下属发号施令的语言，这样的比喻很有用，尤其是在代理（delegates）的上下文中，这个比喻更为贴切，稍后我会详细讲解。

基本上，发送 Objective-C 消息的样子是这样的：

```
[Object doThisThingWithValue:myValue];
```

这就像是说：“嘿，`*Object*`！请用 `*myValue*` 的值做这件事。”当传递多个参数时，第一个参数的性质通常由消息名来表示。任何后续的参数都必须是类的一部分，并且在调用时必须明确命名，就像这个例子：

```
if (pantsColor == @"Black") {

    [NSHouseCat sleepOnPerson:person
                   withRegion:[person lap]
                  andShedding:YES
                      retries:INT_MAX];
}
```

在这个简化的模拟程序中，`sleepOnPerson` 指定了一个睡觉的地方（`person`），而 `withRegion` 通过向 `person` 发送消息来指定这个人的“膝盖”区域。`andShedding` 参数接受一个布尔值，`retries` 则指定此操作将尝试的次数——在本例中，最多可以达到平台上整数的最大值，这个值取决于你是否有一只 64 位猫。

如果你已经编写 Objective-C 一段时间，可能会注意到这个代码的格式看起来与你平时使用的有所不同。这是因为这是一种古老的 Objective-C 代码格式化方法，称为“正确方式”，它通过在参数名称和值之间使用垂直对齐的冒号，使得参数名称和值的配对在视觉上更为明显。

### 剖析一个 Objective-C 程序

一个 Objective-C 程序的两个主要部分是 *接口* 和 *实现*，分别存储在 *.h* 和 *.m* 文件中。（这些大致上与 C++ 中的 *.h* 和 *.cpp* 文件相类似。）前者定义所有的类和方法，而后者定义程序的实际内容和逻辑。

#### *声明一个接口*

接口包含三个主要组件：实例变量（或 *ivars*）、类方法和实例方法。示例 2-1 是经典的（即被弃用的）Objective-C 1.0 声明接口的方式。

```
   @interface Classname : NSParentClass {
➊     NSSomeType aThing;
       int anotherThing;
   }
➋ + (type)classMethod:(vartype)myVariable;
➌ - (type)instanceMethod:(vartype)myVariable;
   @end
```

*示例 2-1：声明一个接口，古老版本*

在主 `@interface` 块内的 ➊，实例变量是用类（如 `NSSomeType`）或类型（如 `int`）声明的，后面跟着它们的名称。在 Objective-C 中，`+` 表示声明一个类方法 ➋，而 `-` 表示实例方法 ➌。与 C 语言类似，方法的返回类型在定义的开始部分用括号指定。

当然，在 Objective-C 中声明接口的现代方式稍有不同。示例 2-2 显示了一个示例。

```
➊ @interface Kitty : NSObject {
       @private NSString *name;
       @private NSURL *homepage;
       @public NSString *color;
   }

   @property NSString *name;
   @property NSURL *homepage;
➋ @property(readonly) NSString *color;

   + (type)classMethod:(vartype)myVariable;
   - (type)instanceMethod:(vartype)myVariable;
```

*示例 2-2：声明一个接口，现代版本*

这个新类名为 `Kitty`，继承自 `NSObject` ➊。`Kitty` 有三个不同访问级别的实例变量，并声明了三个属性来匹配这些实例变量。注意，`color` 被声明为 `readonly` ➋；这是因为一个 `Kitty` 对象的颜色不应该发生变化。这意味着当属性被合成时，只会创建一个 getter 方法，而不是同时创建 getter 和 setter 方法。`Kitty` 还有一对方法：一个类方法和一个实例方法。

你可能已经注意到，示例接口声明在声明实例变量时使用了`@private`和`@public`关键字。与其他语言类似，这些关键字定义了实例变量是否只能在声明它的类内部访问（`@private`），是否可以在声明类及其任何子类中访问（`@protected`），或者是否可以被任何类访问（`@public`）。实例变量的默认行为是`@protected`。

**注意**

*语言的新手通常想知道是否有类似于私有方法的概念。严格来说，Objective-C 中并没有私有方法的概念。然而，你可以通过仅在`*@implementation*`块中声明方法来实现其功能等效，而不是在`*@interface*`和`*@implementation*`中都声明它们。*

#### *在实现文件中*

就像*.c*或*.cpp*文件一样，Objective-C 实现文件包含了 Objective-C 应用程序的核心内容。根据约定，Objective-C 文件使用*.m*文件，而 Objective-C++文件（混合了 C++和 Objective-C 代码）存储在*.mm*文件中。列表 2-3 解析了列表 2-2 中`Kitty`接口的实现文件。

```
   @implementation Kitty
➊ @synthesize name;
   @synthesize color;
   @synthesize homepage;

   + (type)classMethod:(vartype)myVariable {
       // method logic
   }

   - (type)instanceMethod:(vartype)myVariable {
       // method logic
   }
   @end

   Kitty *myKitty = [[Kitty alloc] init];

➋ [myKitty setName:@"Ken"];
➌ myKitty.homepage = [[NSURL alloc] initWithString:@"http://me.ow"];
```

*列表 2-3：一个示例实现*

➊处的`@synthesize`语句创建了属性的 setter 和 getter 方法。稍后，这些 getter 和 setter 方法可以使用 Objective-C 的传统中缀符号表示法➋，其中`*propertyName*`和`*setPropertyName*`格式的方法（例如`name`和`setName`，分别用于获取和设置值），也可以使用点符号表示法➌，在这种方式下，像`homepage`这样的属性使用`*.property*`格式来设置或读取，正如在其他语言中可能出现的那样。

**注意**

*小心使用点符号，或者干脆不要使用它。点符号使得你很难知道你是在处理一个对象还是 C 结构体，实际上你可以用它调用*任何*方法——不仅仅是 getter 和 setter 方法。点符号在视觉上也不一致。长话短说，在本书中，我将避免使用点符号，以保持一致性和思想上的纯洁性。但尽管我尽力避免，你在现实世界中可能仍然会遇到它。*

从技术上讲，对于在接口文件中声明的使用`@property`的属性（如列表 2-3 中的`name`、`color`和`homepage`），你不需要合成这些属性；Xcode 的较新版本会自动合成这些属性。但是，为了清晰起见或当你想改变实例变量的名称以便与属性名称区分时，你仍然可能希望手动声明它们。手动合成属性的工作原理如下：

```
@synthesize name = thisCatName;
```

在这里，属性`name`是由实例变量`thisCatName`支持的，因为它是手动合成的。然而，自动属性合成的默认行为类似于这样：

```
@synthesize name = _name;
```

这种默认行为可以防止开发人员直接操作实例变量，而不是使用设置器和获取器，这样可能会引起混淆。例如，如果你直接设置一个 ivar，你将绕过设置器/获取器方法中的任何逻辑。自动合成可能是最好的方式，但你在代码中仍然会看到手动合成很长一段时间，因此最好对此有所了解。

### 使用代码块指定回调

在 Objective-C 代码中，越来越流行的做法是使用*代码块*，它通常用于 Cocoa 中作为指定回调的一种方式。例如，下面是如何使用`NSURLSessionDataTask`类的`dataTaskWithRequest`方法：

```
   NSURLSession *session = [NSURLSession sessionWithConfiguration:configuration
                                                         delegate:self
                                                    delegateQueue:nil];

   NSURLSessionDataTask *task = [session dataTaskWithRequest:request
                                           completionHandler:
➊     ^(NSData *data, NSURLResponse *response, NSError *error) {
               NSLog(@"Error: %@ %@", error, [error userInfo]);
        }];
```

➊处的`^`声明了一个代码块，该代码块将在请求完成后执行。注意，未指定此函数的名称，因为它不会从代码中的任何其他地方被调用。一个代码块的声明只需要指定闭包将接受的参数。从那里开始，代码块的其他部分就像普通函数一样。你可以将代码块用于许多其他用途，但首先，了解它们的基本概念应该足够了：以`^`开头并执行某些操作的东西。

### Objective-C 如何管理内存

与其他一些语言不同，Objective-C 没有垃圾回收机制。历史上，Objective-C 使用了*引用计数模型*，通过`retain`和`release`指令来指示何时需要释放对象，从而避免内存泄漏。当你`retain`一个对象时，你增加了*引用计数*——也就是希望该对象对其可用的事物的数量。当一段代码不再需要该对象时，它会发送一个`release`方法。当引用计数达到零时，对象将被释放，如下所示：

```
➊ NSFish *fish = [[NSFish alloc] init];
   NSString *fishName = [fish name];
➋ [fish release];
```

假设在这段代码运行之前，引用计数为 0。➊之后，引用计数为 1。在➋处，调用了`release`方法，表示`fish`对象不再需要（应用程序只需要`fish`对象的`name`属性），当`fish`被释放时，引用计数应该再次为 0。

`[[Classname alloc] init]`也可以缩写为`[Classname new]`，但`new`方法在 Objective-C 社区中不太受欢迎，因为它不够明确，并且与除了`init`之外的其他对象创建方法不一致。例如，你可以用`[[NSString alloc] initWithString:@"My string"]`来初始化`NSString`对象，但没有类似的`new`语法，因此你的代码中会混用这两种方法。并非每个人都反感`new`，这确实是一个个人喜好的问题，因此你可能会看到这两种写法。但在本书中，我更倾向于使用传统方法。

无论你偏好哪种分配语法，手动 retain/release 的问题在于它可能引发错误：程序员可能会不小心释放已被销毁的对象（导致崩溃）或忘记释放对象（导致内存泄漏）。苹果尝试通过自动引用计数来简化这种情况。

### 自动引用计数

*自动引用计数（ARC）* 是现代的 Objective-C 内存管理方法。它通过在适当的时候自动递增和递减引用计数，消除了手动跟踪引用计数的需求。^(1) 本质上，它为你插入了 `retain` 和 `release` 方法。ARC 引入了一些新的概念，列举如下：

• *弱引用* 和 *强引用* 有助于防止循环引用（即 *强引用循环*），在这种情况下，父对象和子对象相互拥有对方，导致它们永远不会被销毁。

• Core Foundation 对象和 Cocoa 对象之间的所有权可以进行桥接。桥接告诉编译器，将 Core Foundation 对象转换为 Cocoa 对象后，应该由 ARC 管理，方法是使用 `__bridge` 系列关键字。

• `@autoreleasepool` 替代了之前使用的 `NSAutoReleasePool` 机制。

在现代使用 ARC 的 Cocoa 应用程序中，内存管理的细节通常不会在安全上下文中发挥作用。以前可被利用的条件，如双重释放，已不再是问题，内存管理相关的崩溃也变得非常少见。但仍然值得注意的是，仍然有其他方式可能引发内存管理问题，因为 Core Foundation 对象仍然存在 `CFRetain` 和 `CFRelease`，并且 C 语言的 `malloc` 和 `free` 仍然可以使用。我将在第十一章中讨论使用这些低级 API 时可能出现的内存管理问题。

### 委托与协议

还记得对象如何在“拥挤的房间里互相喊叫”以传递消息吗？*委托* 是一个能够特别好地展示 Objective-C 消息传递架构的特性。委托对象可以接收在程序执行过程中发送的消息，并通过响应指令来影响程序的行为。

成为代理对象，必须实现 *代理协议* 中定义的部分或全部方法，这是一种委托者和代理对象之间约定的通信方式。你可以声明自己的协议，但最常用的还是使用核心 API 中的已定义协议。

你编写的委托通常会响应三种基本消息类型之一：*should*、*will* 和 *did*。每当事件即将发生时，调用这些消息，然后让你的委托对象指导程序采取正确的行动。

#### *Should 消息*

对象发送 *should* 消息来请求任何可用委托提供关于是否允许事件发生的意见。这可以看作是最终的反对意见征集。例如，当 `shouldSaveApplicationState` 消息被触发时，如果你已经实现了一个委托来处理此消息，委托可以执行一些逻辑并说类似这样的话：“不，实际上我们不应该保存应用状态，因为用户选中了一个复选框表示不保存。”这些消息通常期望一个布尔值作为响应。

#### *Will 消息*

*will* 消息给你一个在事件发生之前执行某些操作的机会——有时，甚至可以在事件发生之前踩刹车。这种消息类型更像是说：“嘿，伙计们！只是提醒一下，我将要做这件事情，除非你们需要先做些其他的事情。我对这个想法已经比较坚定，但如果这是个完全不可接受的条件，告诉我，我可以停下。”一个例子是 `applicationWillTerminate` 消息。

#### *Did 消息*

*did* 消息表示某件事情已经确定决定并且一个事件无论你是否喜欢都将发生。它还表明，如果有任何委托想要执行某些操作，他们应该直接进行。例如 `applicationDidEnterBackground`。在这种情况下，did 并不是真正表示应用程序*已经*进入后台，而是反映了决定已经被最终做出。

#### *声明并遵循协议*

要声明你的类遵循某个协议，在 `@interface` 声明中指定该协议，并将其放在尖括号中。要查看实际应用，查看列表 2-4，它展示了一个使用 NSCoding 协议的 `@interface` 声明示例。这个协议简单地指定了一个类实现两个用于编码或解码数据的方法：`encodeWithCoder` 用于编码数据，`initWithCoder` 用于解码数据。

```
➊ @interface Kitty : NSObject <NSCoding> {
       @private NSString *name;
       @private NSURL *homepage;
       @public NSString *color;
   }

   @implementation Kitty

➋ - (id)initWithCoder:(NSCoder *)decoder {
       self = [super init];
       if (!self) {
           return nil;
       }

       [self setName:[decoder decodeObjectForKey:@"name"]];
       [self setHomepage:[decoder decodeObjectForKey:@"homepage"]];
       [self setColor:[decoder decodeObjectForKey:@"color"]];

       return self;
   }

➌ - (void)encodeWithCoder:(NSCoder *)encoder {
       [encoder encodeObject:[self name] forKey:@"name"];
       [encoder encodeObject:[self author] forKey:@"homepage"];
       [encoder encodeObject:[self pageCount] forKey:@"color"];
   }
```

*列表 2-4：声明并实现对 NSCoding 协议的遵循*

➊ 处的声明指定了 `Kitty` 类将符合 NSCoding 协议。^(2) 但是，当一个类声明了一个协议时，它也必须遵循该协议，这就是为什么 `Kitty` 实现了所需的 `initWithCoder` ➋ 和 `encodeWithCoder` ➌ 方法。这些特定的方法用于序列化和反序列化对象。

如果内建的消息协议没有满足你的需求，那么你也可以定义自己的协议。查看 Apple 框架头文件中 NSCoding 协议的声明（列表 2-5），看看协议定义是什么样的。

```
@protocol NSCoding

- (void)encodeWithCoder:(NSCoder *)aCoder;
- (id)initWithCoder:(NSCoder *)aDecoder;

@end
```

*列表 2-5：NSCoding 协议的声明，来自* Frameworks/NSCoding.h

注意，NSCoding 的定义包含了两个方法，任何符合该协议的类必须实现这两个方法：`encodeWithCoder` 和 `initWithCoder`。当你定义一个协议时，必须自己指定这些方法。

### 类别的危险

Objective-C 的*分类*机制允许你在运行时为现有类实现新的方法，而无需重新编译这些类。分类可以向受影响的类添加或替换方法，并且可以出现在代码库的任何位置。这是一种无需重新实现类就能快速更改类行为的简便方法。

不幸的是，使用分类也是导致严重安全错误的一个简单途径。因为它们可以在代码库的任何地方影响你的类——即使它们仅出现在第三方代码中——关键功能，如 TLS 端点验证，可能会被一个随机的第三方库或一个粗心的开发者完全覆盖。我曾在重要的 iOS 产品中看到过这种情况：开发者在仔细验证 TLS/SSL 在他们的应用中正确工作后，添加了一个覆盖该行为的第三方库，搞砸了他们自己精心设计的代码。

你通常可以通过注意到`@implementation`指令来识别分类，这些指令声称实现了 Cocoa Touch 中已经存在的类。如果开发者确实在这里创建了一个分类，那么分类的名称会在`@implementation`指令后面用括号标出（参见列表 2-6）。

```
@implementation NSURL (CategoryName)

- (BOOL) isPurple; {
    if ([self isColor:@"purple"])
        return YES;
    else
        return NO;
}
@end
```

*列表 2-6：实现分类方法*

你还可以使用分类来覆盖*现有*的类方法，这是一种潜在有用但特别危险的方法。这可能导致安全机制被禁用（比如前述的 TLS 验证），也可能导致不可预测的行为。苹果曾说：

如果分类中声明的方法名称与原始类中的方法名称相同，或者与同一类中的另一个分类中的方法名称相同（甚至是父类中的方法），则在运行时无法确定使用哪个方法的实现。

换句话说，多个分类可以定义或覆盖相同的方法，但只有一个会“胜出”并被调用。请注意，一些框架方法可能本身就是通过分类实现的——如果你试图覆盖它们，你的分类*可能*会被调用，但也有可能不会。

分类还可能意外地覆盖子类的功能，即使你只打算添加一个新方法。例如，如果你在`NSObject`上定义了一个`isPurple`方法，那么`NSObject`的所有子类（也就是说，所有 Cocoa 对象）都会继承这个方法。任何其他定义了相同方法名的类，可能会或可能不会被覆盖。因此，没错，分类非常方便，但要谨慎使用；它们可能会导致严重的混乱以及安全副作用。

### 方法交换

*方法交换* 是一种机制，你可以使用它来替换你不拥有的类或实例方法的实现（也就是 Cocoa API 自身提供的方法）。方法交换在功能上类似于类别或子类化，但它通过实际交换方法的实现与一个全新的实现，而不是扩展它，提供了一些额外的能力和灵活性。开发者通常使用这种技术来增强许多不同子类共享使用的方法的功能，这样他们就不必重复代码。

清单 2-7 中的代码使用方法交换（method swizzling）将日志语句添加到任何对 `setHidden` 方法的调用。这将影响任何 `UIView` 的子类，包括 `UITextView`、`UITextField` 等。

```
   #import <objc/runtime.h>

   @implementation UIView(Loghiding)

➊ - (BOOL)swizzled_setHidden {
       NSLog(@"We're calling setHidden now!");

➋     BOOL result = [self swizzled_setHidden];

       return result;
   }

➌ + (void)load {
       Method original_setHidden;
       Method swizzled_setHidden;

       original_setHidden = class_getInstanceMethod(self, @selector(setHidden));
       swizzled_setHidden = class_getInstanceMethod(self, @selector(swizzled_
        setHidden));
➍     method_exchangeImplementations(original_setHidden, swizzled_setHidden);
   }

   @end
```

*清单 2-7：交换现有方法的实现和替代方法的实现*

在➊处，定义了一个包装方法，该方法只是输出一个 `SLog`，表明 `setHidden` 方法正在被调用。但在➋处，`swizzle_SetHidden` 方法似乎在调用自身。这是因为，在执行任何附加功能后，最好调用原始方法，以防止出现不可预测的行为，比如未能返回调用者期望的值类型。当你在 `swizzled_setHidden` 内部调用自己时，实际上会调用*原始*方法，因为原始方法和替换方法已经被交换。

实际的交换发生在 `load` 类方法 ➌ 中，当 Objective-C 运行时第一次加载该类时会调用此方法。在获取原始方法和交换方法的引用后，在 ➍ 处调用 `method_exchangeImplementations` 方法，顾名思义，它交换原始实现和交换实现。

实现方法交换有几种不同的策略，但大多数方法都有一定的风险，因为你在更改核心功能。

如果你或你的亲人想实现方法交换，可能需要考虑使用一个经过充分测试的包装包，如 JRSwizzle.^(3) Apple 可能会拒绝看起来以危险方式使用方法交换的应用。

### 结束语

总体而言，Objective-C 和 Cocoa API 是相当高级的，避免了许多 C 语言中的经典安全问题。尽管仍然存在一些破坏内存管理和对象操作的方法，但在现代代码中，这些方法大多数情况下会导致服务拒绝（Denial of Service，DoS）。如果你是开发者，尽可能依赖 Cocoa，而不是修补 C 或 C++ 代码。

然而，Objective-C 确实包含一些机制，如类别或方法交换，这些机制可能导致意外行为，并且可能广泛影响你的代码库。当你在应用程序评估中看到这些技术时，一定要仔细调查它们，因为它们可能会导致一些严重的安全问题。
