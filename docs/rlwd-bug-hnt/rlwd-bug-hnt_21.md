## 第二十一章：**A

TOOLS**

![Image](img/common.jpg)

本附录包含了一份黑客工具清单。这些工具中的一些允许你自动化侦察过程，而其他工具则帮助你发现可攻击的应用程序。这份清单并不意味着详尽无遗，它仅反映了我常用的工具，或是我知道其他黑客常用的工具。同时请记住，这些工具都不应替代观察力或直觉思考。HackerOne 的联合创始人 Michiel Prins 值得赞扬，他帮助开发了这个清单的初始版本，并在我开始进行黑客攻击时提供了如何有效使用这些工具的建议。

### Web Proxies

Web 代理可以捕获你的网络流量，方便你分析发送的请求和接收到的响应。许多此类工具是免费的，尽管专业版工具具有额外功能。

**Burp Suite**

Burp Suite (*[`portswigger.net/burp/`](https://portswigger.net/burp/)*) 是一个集成的安全测试平台。平台中最有用的工具，也是我使用频率最高的工具，是 Burp 的 Web 代理。回想一下书中的漏洞报告，代理功能让你能够监控你的流量、实时拦截请求、修改它们并转发。Burp 拥有丰富的工具集，但以下是我认为最值得注意的几个：

+   一个应用感知的蜘蛛工具，用于爬取内容和功能（可以是被动的也可以是主动的）

+   用于自动化漏洞检测的 Web 扫描器

+   一个用于操作和重新发送单个请求的重复器

+   用于在平台上构建额外功能的扩展

Burp 提供免费版本，但工具的使用有一定限制，你也可以通过年度订阅购买 Pro 版。我建议你先从免费版开始，直到你掌握了如何使用它。当你稳定地发现漏洞时，可以购买 Pro 版以便更轻松地工作。

**Charles**

Charles (*[`www.charlesproxy.com/`](https://www.charlesproxy.com/)*) 是一款 HTTP 代理工具、HTTP 监视器和反向代理工具，允许开发者查看 HTTP 和 SSL/HTTPS 流量。使用它，你可以查看请求、响应和 HTTP 头（其中包含了 cookies 和缓存信息）。

**Fiddler**

Fiddler (*[`www.telerik.com/fiddler/`](https://www.telerik.com/fiddler/)*) 是另一款轻量级代理工具，可以用来监控你的流量，但其稳定版本仅支持 Windows。Mac 和 Linux 版本在写本文时仍处于测试阶段。

**Wireshark**

Wireshark (*[`www.wireshark.org/`](https://www.wireshark.org/)*) 是一款网络协议分析工具，可以让你详细查看网络中的发生情况。当你需要监控无法通过 Burp 或 ZAP 代理的流量时，Wireshark 非常有用。如果你刚开始使用，且网站仅通过 HTTP/HTTPS 进行通信，使用 Burp Suite 可能是最好的选择。

**ZAP Proxy**

OWASP Zed Attack Proxy（ZAP）是一个免费的、社区驱动的开源平台，类似于 Burp。它可以通过 *[`www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project`](https://www.owasp.org/index.php/OWASP_Zed_Attack_Proxy_Project)* 获得。它还具有各种工具，包括代理、重放器、扫描器、目录/文件暴力破解工具等。此外，它支持插件，因此如果你有需要，可以创建额外的功能。该网站提供了一些有用的信息，可以帮助你入门。

### 子域名枚举

网站通常有一些子域名，人工工作很难发现。暴力破解子域名可以帮助你识别程序的额外攻击面。

**Amass**

OWASP Amass 工具 (*[`github.com/OWASP/Amass`](https://github.com/OWASP/Amass)*) 通过抓取数据源、使用递归暴力破解、爬取网页归档、排列或修改名称，并利用反向 DNS 扫描来获取子域名。Amass 还利用在解析过程中获得的 IP 地址来发现关联的网络块和自治系统号码（ASNs）。然后，它利用这些信息来构建目标网络的地图。

**[crt.sh](http://crt.sh)**

[crt.sh](http://crt.sh) 网站 (*[`crt.sh/`](https://crt.sh/)*) 允许你浏览证书透明日志，以便查找与证书关联的子域名。证书注册可以揭示站点使用的其他子域名。你可以直接使用该网站，或者使用工具 SubFinder，它解析来自 [crt.sh](http://crt.sh) 的结果。

**Knockpy**

Knockpy (*[`github.com/guelfoweb/knock/`](https://github.com/guelfoweb/knock/)*) 是一款 Python 工具，旨在通过遍历词汇列表来识别公司的网站子域名。识别子域名可以为你提供更大的可测试面，并增加找到成功漏洞的机会。

**SubFinder**

SubFinder (*[`github.com/subfinder/subfinder/`](https://github.com/subfinder/subfinder/)*) 是一款用 Go 编写的子域名发现工具，通过使用被动在线来源来发现有效的子域名。它具有简单的模块化架构，旨在替代类似的工具 Sublist3r。SubFinder 使用被动来源、搜索引擎、代码粘贴板、互联网档案等来查找子域名。当它找到子域名时，它使用一个灵感来自工具 altdns 的排列模块来生成排列，并使用强大的暴力破解引擎来解析它们。如果需要，它还可以执行纯暴力破解。该工具具有高度可定制性，代码采用模块化方式构建，易于添加功能和修复错误。

### 发现

当你识别出一个程序的攻击面后，下一步是枚举文件和目录。这样做可以帮助你发现隐藏的功能、敏感文件、凭证等。

**Gobuster**

Gobuster (*[`github.com/OJ/gobuster/`](https://github.com/OJ/gobuster/)*) 是一个可以用来暴力破解 URI（目录和文件）以及 DNS 子域名的工具，支持通配符。它非常快速、可定制，并且易于使用。

**SecLists**

虽然严格来说，SecLists (*[`github.com/danielmiessler/SecLists/`](https://github.com/danielmiessler/SecLists/)*) 本身不是一个工具，但它是一个可以在黑客攻击时使用的词汇表集合。这些词汇表包括用户名、密码、URL、模糊测试字符串、常见目录/文件/子域名等等。

**Wfuzz**

Wfuzz (*[`github.com/xmendez/wfuzz/`](https://github.com/xmendez/wfuzz/)*) 允许你在 HTTP 请求的任何字段中注入任何输入。通过使用 Wfuzz，你可以对 Web 应用程序的不同组件（如参数、身份验证、表单、目录或文件、头部等）执行复杂的攻击。你还可以在支持插件的情况下，将 Wfuzz 作为漏洞扫描器使用。

### 截图工具

在某些情况下，你的攻击面可能过大，无法测试其每个方面。当你需要检查一个很长的网站或子域名列表时，可以使用自动截图工具。这些工具允许你在不访问每个网站的情况下，直观地检查网站内容。

**EyeWitness**

EyeWitness (*[`github.com/FortyNorthSecurity/EyeWitness/`](https://github.com/FortyNorthSecurity/EyeWitness/)*) 旨在截取网站截图，提供服务器头信息，并在可能的情况下识别默认凭证。它是一个非常适合用来检测哪些服务在常见 HTTP 和 HTTPS 端口上运行的工具，你可以结合其他工具（如 Nmap）一起使用，以快速枚举黑客目标。

**Gowitness**

Gowitness (*[`github.com/sensepost/gowitness/`](https://github.com/sensepost/gowitness/)*) 是一个用 Go 编写的网站截图工具。它使用 Chrome Headless 生成网页界面的截图，并支持命令行操作。该项目受 EyeWitness 工具的启发。

**HTTPScreenShot**

HTTPScreenShot (*[`github.com/breenmachine/httpscreenshot/`](https://github.com/breenmachine/httpscreenshot/)*) 是一个用于截取大量网站屏幕截图和 HTML 的工具。HTTPScreenShot 接受 IP 列表作为 URL 来进行截图。它还可以暴力破解子域名，将其添加到待截图的 URL 列表中，并对结果进行聚类以便更容易审查。

### 端口扫描

除了查找 URL 和子域名外，你还需要弄清楚哪些端口是开放的，以及服务器上运行了哪些应用程序。

**Masscan**

Masscan (*[`github.com/robertdavidgraham/masscan/`](https://github.com/robertdavidgraham/masscan/)*) 宣称是世界上最快的互联网端口扫描器。它可以在不到六分钟的时间内扫描整个互联网，每秒传输 1000 万个数据包。其结果与 Nmap 类似，唯一的区别是速度更快。此外，Masscan 允许你扫描任意的地址范围和端口范围。

**Nmap**

Nmap (*[`nmap.org/`](https://nmap.org/)*)是一个免费的开源工具，用于网络发现和安全审计。Nmap 使用原始 IP 数据包来确定：

+   网络上有哪些主机可用

+   这些主机提供哪些服务（包括应用程序名称和版本）

+   他们运行的是哪些操作系统（及版本）

+   正在使用的是什么类型的数据包过滤器或防火墙

Nmap 网站提供了适用于 Windows、Mac 和 Linux 的完整安装说明。除了端口扫描外，Nmap 还包括一些脚本来构建附加功能。我常用的一个脚本是`http-enum`，它可以在端口扫描完服务器后列举出服务器上的文件和目录。

### Reconnaissance

在你找到网站的 URI、子域名和端口后，你需要了解它们使用的技术以及它们连接的互联网的其他部分。以下工具将帮助你做到这一点。

**BuiltWith**

BuiltWith (*[`builtwith.com/`](http://builtwith.com/)*)帮助你识别目标上使用的不同技术。根据其网站的介绍，它可以检查超过 18,000 种互联网技术，包括分析工具、托管服务、CMS 类型等。

**Censys**

Censys (*[`censys.io/`](https://censys.io/)*)通过每天对 IPv4 地址空间进行 ZMap 和 ZGrab 扫描，收集有关主机和网站的数据。它维护着一个关于主机和网站如何配置的数据库。不幸的是，Censys 最近实施了付费模式，对于大规模黑客攻击来说费用较高，但免费的层级仍然有帮助。

**Google Dorks**

Google Dorking (*[`www.exploit-db.com/google-hacking-database/`](https://www.exploit-db.com/google-hacking-database/)*)是指使用 Google 提供的高级语法来查找在手动浏览网站时无法轻易获取的信息。这些信息可能包括查找易受攻击的文件、外部资源加载的机会以及其他攻击面。

**Shodan**

Shodan (*[`www.shodan.io/`](https://www.shodan.io/)*)是一个物联网的搜索引擎。Shodan 可以帮助你发现哪些设备连接到互联网，它们的位置以及谁在使用它们。当你探索潜在目标并尽可能多地了解目标的基础设施时，这尤其有帮助。

**What CMS**

What CMS (*[`www.whatcms.org/`](http://www.whatcms.org/)*)允许你输入 URL，并返回该网站最有可能使用的内容管理系统（CMS）。找出网站使用的 CMS 类型非常有帮助，因为：

+   了解网站使用的 CMS 能够帮助你洞察该网站代码的结构。

+   如果 CMS 是开源的，你可以浏览代码寻找漏洞，并在网站上进行测试。

+   该网站可能已过时，并且容易受到已披露的安全漏洞攻击。

### Hacking Tools

使用黑客工具，你不仅可以自动化发现和枚举过程，还可以自动化查找漏洞的过程。

**Bucket Finder**

Bucket Finder (*[`digi.ninja/files/bucket_finder_1.1.tar.bz2`](https://digi.ninja/files/bucket_finder_1.1.tar.bz2)*) 用于查找可读取的桶并列出其中的所有文件。它还可以快速找到存在但无法列出文件的桶。当你发现这类桶时，可以尝试使用 AWS CLI，详见“ HackerOne S3 Buckets Open”中的漏洞报告，参见第 223 页。

**CyberChef**

CyberChef (*[`gchq.github.io/CyberChef/`](https://gchq.github.io/CyberChef/)*) 是一款多功能的编码和解码工具。

**Gitrob**

Gitrob (*[`github.com/michenriksen/gitrob/`](https://github.com/michenriksen/gitrob/)*) 帮助你查找可能已被推送到 GitHub 公共仓库中的敏感文件。Gitrob 会克隆用户或组织的仓库，直到可配置的深度，并遍历提交历史，标记出符合敏感文件签名的文件。它通过 Web 界面展示结果，便于浏览和分析。

**Online Hash Crack**

Online Hash Crack (*[`www.onlinehashcrack.com/`](https://www.onlinehashcrack.com/)*) 尝试恢复以哈希形式存储的密码、WPA 转储以及 MS Office 加密文件。它支持识别超过 250 种哈希类型，当你想要识别一个网站使用的哈希类型时非常有用。

**sqlmap**

你可以使用开源渗透工具 sqlmap (*[`sqlmap.org/`](http://sqlmap.org/)*) 来自动化检测和利用 SQL 注入漏洞的过程。该网站列出了其功能，包括支持以下内容：

+   支持多种数据库类型，例如 MySQL、Oracle、PostgreSQL、MS SQL Server 等。

+   六种 SQL 注入技术

+   用户、密码哈希、权限、角色、数据库、表和列枚举

**XSSHunter**

XSSHunter (*[`xsshunter.com/`](https://xsshunter.com/)*) 帮助你发现盲目 XSS 漏洞。在注册 XSSHunter 后，你将获得一个 *xss.ht* 短域名，用于标识你的 XSS 并托管你的有效载荷。当 XSS 触发时，它会自动收集发生位置的信息，并向你发送电子邮件通知。

**Ysoserial**

Ysoserial (*[`github.com/frohoff/ysoserial/`](https://github.com/frohoff/ysoserial/)*) 是一个概念验证工具，用于生成利用不安全 Java 对象反序列化漏洞的有效载荷。

### 移动端

尽管本书中的大多数漏洞是通过网页浏览器发现的，但在某些情况下，你需要分析移动应用程序作为测试的一部分。能够拆解并分析应用程序的组件，将帮助你了解它们是如何工作的，以及它们可能的漏洞所在。

**dex2jar**

dex2jar (*[`sourceforge.net/projects/dex2jar/`](https://sourceforge.net/projects/dex2jar/)*) 是一套移动黑客工具，可以将 dalvik 可执行文件（*.dex* 文件）转换为 Java *.jar* 文件，这使得审计 Android APK 更加轻松。

**Hopper**

Hopper (*[`www.hopperapp.com/`](https://www.hopperapp.com/)*) 是一个逆向工程工具，让你能够反汇编、反编译和调试应用程序。它对于审核 iOS 应用程序非常有用。

**JD-GUI**

JD-GUI (*[`github.com/java-decompiler/jd-gui/`](https://github.com/java-decompiler/jd-gui/)*) 帮助你探索 Android 应用程序。它是一个独立的图形化工具，可以从 *CLASS* 文件中显示 Java 源代码。

### Browser Plug-Ins

Firefox 有几个浏览器插件，你可以将它们与其他工具结合使用。虽然这里只介绍了 Firefox 版本的工具，但在其他浏览器上可能也有类似的工具可供使用。

**FoxyProxy**

FoxyProxy 是一个高级代理管理插件，适用于 Firefox。它增强了 Firefox 内建的代理功能。

**User Agent Switcher**

User Agent Switcher 在 Firefox 浏览器中添加了一个菜单和工具栏按钮，允许你切换用户代理。你可以使用此功能在执行一些攻击时伪装浏览器。

**Wappalyzer**

Wappalyzer 帮助你识别一个网站使用的技术，如 CloudFlare、框架、JavaScript 库等。
