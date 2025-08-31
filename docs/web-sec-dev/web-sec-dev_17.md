# 第十五章：**XML 攻击**

![image](img/common01.jpg)

随着 90 年代互联网的爆炸式增长，各组织开始通过 Web 共享数据。计算机之间共享数据意味着必须达成一个共享的数据格式。在 Web 上的人类可读文档使用超文本标记语言（HTML）进行标记。机器可读的文件通常存储在一种类似的数据格式中，称为 *可扩展标记语言（XML）*。

XML 可以被看作是 HTML 的一种更通用的实现：在这种标记方式中，标签和属性名称可以由文档作者选择，而不是像 HTML 规范中那样固定。在 列表 15-1 中，你可以看到一个 XML 文件，描述了一本书的目录，使用了 `<catalog>`、`<book>` 和 `<author>` 等标签。

```
<?xml version="1.0"?>
<catalog>
   <book id="7991728882998">
      <author>Sponden, Phillis</author>
      <title>The Evil Horse That Knew Karate</title>
      <genre>Young Adult Fiction</genre>
      <description>Three teenagers with very different personalities
team up to defeat a surprising villain.</description>
   </book>
   <book id="28299171927772">
      <author>Chenoworth, Dr. Sebastian</author>
      <title>Medical Encyclopedia of Elbows, 12th Edition</title>
      <genre>Medical</genre>
      <description>The world's foremost forearm expert gives detailed diagnostic
and clinical advice on maintaining everyone's favorite joint.</description>
   </book>
</catalog>
```

*列表 15-1：描述书籍目录的 XML 文档*

这种数据格式的流行，尤其是在 Web 初期，意味着 XML *解析*——将 XML 文件转换为内存中的代码对象的过程——在过去几十年的每个浏览器和 Web 服务器中都有实现。不幸的是，XML 解析器是黑客攻击的常见目标。即使你的网站设计上不处理 XML，Web 服务器也可能默认解析该数据格式。本章将展示 XML 解析器如何受到攻击以及如何化解这些攻击。

### XML 的应用

与 HTML 类似，XML 将数据项封装在标签之间，并允许标签嵌套在彼此之间。XML 文档的作者可以选择具有语义意义的标签名称，使得 XML 文档自描述。由于 XML 非常易读，这种数据格式被广泛采用，用于编码供其他应用程序使用的数据。

XML 的应用非常广泛。允许客户端软件通过互联网调用函数的应用程序接口（API）通常使用 XML 来接收和响应数据。网页中的 JavaScript 代码在与服务器进行异步通信时，常常使用 XML。许多类型的应用程序——包括 Web 服务器——都使用基于 XML 的配置文件。

在过去的十年中，一些应用开始使用比 XML 更适合、更简洁的数据格式。例如，JSON 是在 JavaScript 和其他脚本语言中编码数据的更自然方法。YAML 语言通过有意义的缩进，使其成为配置文件的更简单格式。尽管如此，每个 Web 服务器都以某种方式实现了 XML 解析，并且需要防范 XML 攻击。

XML 漏洞通常发生在验证过程中。让我们花一点时间讨论在解析 XML 文档的上下文中，验证的含义。

### 验证 XML

由于 XML 文件的作者能够选择文档中使用的标签名称，任何读取数据的应用程序都需要知道预期的标签名称及其出现顺序。XML 文档的预期结构通常通过正式语法来描述，文档可以根据该语法进行*验证*。

*语法*文件规定了哪些字符序列是语言中有效的表达式。例如，编程语言的语法可能会规定，变量名只能包含字母数字字符，而某些运算符如`+`需要两个输入。

XML 有两种主要方式来描述 XML 文档的预期结构。*文档类型定义（DTD）*文件类似于*巴科斯–诺尔范式（BNF）*符号，经常用于描述编程语言的语法。*XML Schema 定义（XSD）*文件是更现代、更具表现力的替代方案，能够描述更广泛的 XML 文档；在这种情况下，语法本身是通过 XML 文件来描述的。这两种 XML 验证方法都被 XML 解析器广泛支持。然而，DTD 包含一些可能暴露解析器于攻击的特性，因此我们将重点讨论这一点。

#### *文档类型定义*

DTD 文件通过指定预期文档中标签、子标签和数据类型来描述 XML 文件的结构。列表 15-2 展示了一个 DTD 文件，描述了列表 15-1 中`<catalog>`和`<book>`标签的预期结构。

```
<!DOCTYPE catalog [
  <!ELEMENT catalog     (book+)>
  <!ELEMENT book        (author,title,genre,description)>
  <!ENTITY  author      (#PCDATA)>
  <!ENTITY  title       (#PCDATA)>
  <!ENTITY  genre       (#PCDATA)>
  <!ENTITY  description (#PCDATA)>
  <!ATTLIST book id CDATA>
]>
```

*列表 15-2：描述列表 15-1 中 XML 格式的 DTD 文件*

这个 DTD 描述了顶层的`<catalog>`标签预计包含零个或多个`<book>`标签（数量由`+`符号表示），并且每个`<book>`标签预计包含描述`author`、`title`、`genre`和`description`的标签，以及一个`id`属性。标签和属性预计包含解析后的字符数据（`#PCDATA`）或字符数据（`CDATA`）——即文本而非标签。

DTD 可以包含在 XML 文档中，使文档自我验证。然而，支持这种*内联* DTD 的解析器容易受到攻击——因为上传此类 XML 文档的恶意用户可以控制 DTD 的内容，而不是由解析器本身提供。黑客利用内联 DTD 使文档在解析过程中消耗的服务器内存成倍增加（XML 炸弹），并访问服务器上的其他文件（XML 外部实体攻击）。让我们看看这些攻击是如何工作的。

### XML 炸弹

*XML 炸弹*使用内联 DTD 来爆炸 XML 解析器的内存使用量。这会通过耗尽服务器可用的所有内存，导致其崩溃，从而使 Web 服务器停机。

XML 炸弹利用了 DTD 可以指定简单的字符串替换宏，并在解析时展开，这些宏被称为*内部实体声明*。如果一段文本在 XML 文件中经常使用，你可以在 DTD 中将其声明为内部实体。这样，你就不必每次都输入它，只需在文档中键入实体名称作为简写。在列表 15-3 中，包含员工记录的 XML 文件通过使用内部实体声明在 DTD 中指定公司名称。

```
<?xml version="1.0"?>
<!DOCTYPE employees [
  <!ELEMENT employees (employee)*>
  <!ELEMENT employee (#PCDATA)>
  <!ENTITY company "Rock and Gravel Company"❶>
]>
<employees>
  <employee>
    Fred Flintstone, &company;❷
  </employee>
  <employee>
    Barney Rubble, &company;❸
  </employee>
</employees>
```

*列表 15-3：内部实体声明*

字符串 `&company;` ❷ ❸ 充当 `Rock and Gravel Company` ❶ 的占位符。当文档被解析时，解析器将所有 `&company;` 的实例替换为 `Rock and Gravel Company`，并生成最终的文档，如列表 15-4 所示。

```
<?xml version="1.0"?>
<employees>
  <employee>
    Fred Flintstone, Rock and Gravel Company
  </employee>
  <employee>
    Barney Rubble, Rock and Gravel Company
  </employee>
</employees>
```

*列表 15-4：解析器处理 DTD 后的 XML 文档*

内部实体声明有其用途，尽管很少使用。问题出现在内部实体声明引用其他内部实体声明时。列表 15-5 显示了一系列嵌套的实体声明，构成了一个 XML 炸弹。

```
<?xml version="1.0"?>
<!DOCTYPE lolz [
  <!ENTITY lol "lol">
  <!ENTITY lol2 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
  <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
  <!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
  <!ENTITY lol5 "&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;">
  <!ENTITY lol6 "&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;">
  <!ENTITY lol7 "&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;">
  <!ENTITY lol8 "&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;">
  <!ENTITY lol9 "&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;">
]>
<lolz>&lol9;</lolz>
```

*列表 15-5：一种 XML 炸弹，被称为* 十亿笑声攻击

当此 XML 文件被解析时，`&lol9;` 字符串将被替换为 10 次 `&lol8;` 字符串。然后，*每一个* `&lol8;` 实例将被替换为 10 次 `&lol7;` 字符串。XML 文件的最终形式包含一个 `<lolz>` 标签，其中有超过 *十亿* 次 `lol` 字符串的出现。这个简单的 XML 文件在 DTD 完全展开后将占用超过 3GB 的内存，足以使 XML 解析器崩溃！

消耗 XML 解析器的可用内存将使你的 Web 服务器离线，这使得 XML 炸弹成为黑客发起拒绝服务攻击的有效手段。攻击者所需要做的就是找到一个接受 XML 上传的 URL，他们只需点击一下按钮就能让你的网站离线。

接受内联 DTD 的 XML 解析器也容易受到一种更隐蔽类型攻击的威胁，这种攻击以不同的方式利用实体定义。

### XML 外部实体攻击

DTD 可以包含来自外部文件的内容。如果 XML 解析器配置为处理内联 DTD，攻击者可以利用这些*外部实体声明*来探索本地文件系统或触发来自 Web 服务器本身的网络请求。

一个典型的外部实体如列表 15-6 所示。

```
<?xml version="1.0" standalone="no"?>
<!DOCTYPE copyright [
  <!ELEMENT copyright (#PCDATA)>
  <!ENTITY copy PUBLIC "http://www.w3.org/xmlspec/copyright.xml"❶>
]>
<copyright>&copy;❷ </copyright>
```

*列表 15-6：使用外部实体在 XML 文件中包含标准版权文本*

根据 XML 1.0 规范，解析器应读取外部实体中指定文件的内容，并在 XML 文档中每次引用该实体时插入该数据。在这个例子中，托管在 *http://www.w3.org/xmlspec/copyright.xml* ❶ 上的数据将被插入到 XML 文档中，每当文本 `&copy;` ❷ 出现时。

外部实体声明所引用的 URL 可以使用不同的网络协议，具体取决于前缀。我们示例中的 DTD 使用 *http://* 前缀，这将导致解析器发起 HTTP 请求。XML 规范还支持使用 *file://* 前缀读取本地磁盘文件。因此，外部实体定义是一个安全的 *灾难*。

#### *黑客如何利用外部实体*

当 XML 解析器抛出错误时，错误信息通常会包括正在解析的 XML 文档的内容。黑客正是利用这一点，通过外部实体声明来读取服务器上的文件。例如，一个恶意构造的 XML 文件可能会包含对 Linux 系统中类似 *file://etc/passwd* 文件的引用。当这个外部文件被解析器插入到 XML 文档中时，XML 就会变得不合法——因此解析失败。解析器会忠实地将文件内容包含在错误响应中，允许黑客查看引用文件中的敏感数据。通过这种技术，黑客可以读取易受攻击的 web 服务器上的敏感文件，这些文件包含密码和其他机密信息。

外部实体也可以用于发起 *服务器端请求伪造（SSRF）* 攻击，攻击者通过这种方式从你的服务器触发恶意 HTTP 请求。一个配置不当的 XML 解析器会在遇到带有网络协议前缀的外部实体 URL 时发起网络请求。能够将你的 web 服务器欺骗成根据攻击者选择的 URL 发起网络请求，对攻击者来说无疑是一个巨大的好处！黑客已经利用这一特性来探测内部网络、发起针对第三方的拒绝服务攻击，并伪装恶意的 URL 请求。你将在下一章中了解更多关于 SSRF 攻击的风险。

### 保护你的 XML 解析器

这是一个简单的修复，能够保护你的解析器免受 XML 攻击：在你的配置中禁用内联 DTD 的处理。DTD 是一种过时的技术，内联 DTD 本身就是一个糟糕的主意。事实上，许多现代 XML 解析器默认已经经过硬化，这意味着它们在开箱即用时会禁用那些可能使解析器受到攻击的功能，所以你可能已经得到保护。如果你不确定，应该检查你使用的 XML 解析技术（如果有的话）。

以下部分描述了如何在一些主要的 Web 编程语言中保护你的 XML 解析器。即使你认为你的代码并不解析 XML，你使用的第三方依赖项很可能以某种形式使用 XML。确保你分析整个依赖树，以查看当你的 web 服务器启动时，哪些库被加载到内存中。

#### *Python*

`defusedxml` 库明确拒绝内联 DTD，并且是 Python 标准 XML 解析库的直接替代品。使用此模块来替代 Python 的标准库。

#### *Ruby*

Ruby 中解析 XML 的事实标准是 `Nokogiri` 库。自版本 1.5.4 以来，该库已经加固了 XML 攻击的防护，因此请确保你的代码使用该版本或更高版本进行解析。

#### *Node.js*

Node.js 有多种解析 XML 的模块，包括 `xml2js`、`parse-xml` 和 `node-xml`。大多数模块默认省略 DTD 的处理，因此请确保查阅你所使用的解析器的文档。

#### *Java*

Java 有多种解析 XML 的方法。符合 Java 规范的解析器通常通过 `javax.xml.parsers.DocumentBuilderFactory` 类启动解析。Listing 15-7 说明了如何在该类的任何实例化位置配置安全的 XML 解析，使用 `XMLConstants.FEATURE_SECURE_PROCESSING` 特性。

```
DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
factory.setFeature(XMLConstants.FEATURE_SECURE_PROCESSING, true);
```

*Listing 15-7: 安全配置 Java XML 解析库*

#### *.NET*

.NET 有多种解析 XML 的方法，所有这些方法都包含在 `System.Xml` 命名空间中。`XmlDictionaryReader`、`XmlNodeReader` 和 `XmlReader` 默认是安全的，`System.Xml.Linq.XElement` 和 `System.Xml.Linq.XDocument` 也是如此。`System.Xml.XmlDocument`、`System.Xml.XmlTextReader` 和 `System.Xml.XPath.XPathNavigator` 从 .NET 版本 4.5.2 起已被加固。如果你使用的是早期版本的 .NET，应该切换到安全的解析器，或禁用内联 DTD 的处理。Listing 15-8 展示了如何通过设置 `ProhibitDtd` 属性标志来实现这一点。

```
XmlTextReader reader = new XmlTextReader(stream);
reader.ProhibitDtd = true;
```

*Listing 15-8: 禁用 .NET 中的内联 DTD 处理*

### 其他考虑事项

外部实体攻击的威胁说明了遵循 *最小权限原则* 的重要性，该原则指出，软件组件和进程应该仅获得执行其任务所需的最小权限。XML 解析器几乎没有正当理由进行外发网络请求：考虑将 Web 服务器的外发网络请求限制到最小。如果确实需要外发网络访问——例如，如果你的服务器代码调用了第三方 API——你应该在防火墙规则中白名单这些 API 的域名。

同样，限制 Web 服务器可以访问的磁盘目录也很重要。在 Linux 操作系统中，可以通过将 Web 服务器进程运行在 `chroot` 监狱中来实现，这样可以忽略运行进程尝试更改根目录的任何操作。在 Windows 操作系统中，应该手动白名单 Web 服务器可以访问的目录。

### 总结

可扩展标记语言（XML）是一种灵活的数据格式，广泛用于在互联网上交换机器可读的数据。如果你的 XML 解析器配置为接受并处理内联文档类型定义（DTD），它可能会受到攻击。XML 爆炸文件使用内联 DTD 来爆炸解析器的内存使用，可能导致你的 Web 服务器崩溃。XML 外部实体攻击引用本地文件或网络地址，并可以用于欺骗解析器泄露敏感信息或发起恶意网络请求。确保你使用一个加强版的 XML 解析器，该解析器禁用了内联 DTD 解析。

下一章将扩展本章提到的一个概念：黑客如何利用你 Web 服务器中的安全漏洞，对第三方发起攻击。即使你不是直接受害者，作为一个合格的互联网公民，阻止利用你的系统进行的攻击也是非常重要的。
