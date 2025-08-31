

# 第四章：4 侦察



![](img/opener.jpg)

每次黑客攻击活动都始于某种形式的信息收集。在本章中，我们将通过编写 bash 脚本来对目标进行侦察，运行各种黑客工具。你将学习如何使用 bash 自动化任务，并将多个工具串联成一个工作流。

在这个过程中，你将发展出一项重要的 bash 脚本技能：解析各种工具的输出，提取你需要的信息。你的脚本将与工具进行交互，确定哪些主机在线，哪些主机的端口是开放的，运行了哪些服务，然后以你要求的格式将这些信息传递给你。

在 第三章 中设置的易受攻击的网络上，所有黑客活动都应在 Kali 环境中进行。

## 创建可重用的目标列表

*范围* 是你被允许攻击的系统或资源列表。在渗透测试或漏洞挖掘活动中，目标公司可能会向你提供各种类型的范围：

+   独立的 IP 地址，如 172.16.10.1 和 172.16.10.2

+   网络，如 172.16.10.0/24 和 172.16.10.1–172.16.10.254

+   独立的域名，如 *lab.example.com*

+   一个父域名及其所有子域名，如 **.example.com*

当使用如端口和漏洞扫描器等工具时，你通常需要对你范围内的所有主机进行相同类型的扫描。然而，由于每个工具使用不同的语法，这可能很难高效完成。例如，一个工具可能允许你指定一个包含目标列表的输入文件，而其他工具可能需要单独的地址。

当使用一些不允许你提供广泛目标范围的工具时，你可以使用 bash 来自动化这个过程。在本节中，我们将使用 bash 创建基于 IP 和 DNS 的目标列表，供扫描工具使用。

### 连续的 IP 地址

假设你需要创建一个包含从 172.16.10.1 到 172.16.10.254 的 IP 地址列表的文件。虽然你可以手动写下所有 254 个地址，但这会非常耗时。让我们使用 bash 来自动化这个任务！我们将考虑三种策略：在 for 循环中使用 seq 命令，使用 echo 的花括号扩展，以及使用 printf 的花括号扩展。

在 列表 4-1 中显示的 for 循环中，我们使用 seq 来遍历从 1 到 254 的数字，并将每个数字分配给 ip 变量。每次迭代后，我们使用 echo 将 IP 地址写入磁盘上的专用文件 *172-16-10-hosts.txt*。

```
#!/bin/bash

# Generate IP addresses from a given range.
for ip in $(seq 1 254); do
  echo "172.16.10.${ip}" >> 172-16-10-hosts.txt
done 
```

列表 4-1：使用 seq 命令和 for 循环创建 IP 地址列表

你可以直接从命令行运行这段代码，或者将其保存在脚本中再运行。生成的文件应该如下所示：

```
$ **cat 172-16-10-hosts.txt**
172.16.10.1
172.16.10.2
172.16.10.3
172.16.10.4
172.16.10.5
`--snip--` 
```

与大多数情况一样，您可以在 bash 中使用多种方法来完成相同的任务。我们可以使用简单的 echo 命令生成 IP 地址列表，而不需要运行任何循环。在 列表 4-2 中，我们使用 echo 与大括号展开生成字符串。

```
$ **echo 10.1.0.{1..254}**

10.1.0.1 10.1.0.2 10.1.0.3 10.1.0.4 ... 
```

列表 4-2：使用 echo 执行大括号展开

您会注意到，此命令在单行上输出以空格分隔的 IP 地址列表。这并不理想，因为我们真正想要的是每个 IP 地址单独占据一行。在 列表 4-3 中，我们使用 sed 将空格替换为换行符（\n）。

```
$ **echo 10.1.0.{1..254} | sed 's/ /\n/g'**

10.1.0.1
10.1.0.2
10.1.0.3
`--snip--` 
```

列表 4-3：使用 echo 和 sed 生成 IP 地址列表

或者，您可以使用 printf 命令生成相同的列表。使用 printf 不需要管道到 sed，产生更干净的输出：

```
$ **printf "10.1.0.%d\n" {1..254}**
```

`%d` 是整数占位符，将与大括号展开中定义的数字交换，以生成从 10.1.0.1 到 10.1.0.254 的 IP 地址列表。您可以将输出重定向到新文件，然后将其用作输入文件。

### 可能的子域

假设您正在对公司的父域 *example.com* 进行渗透测试。在此次参与中，您不受限于任何特定的 IP 地址或域名，这意味着您在信息收集阶段发现的该父域上的任何资产都被视为在范围内。

公司倾向于将其服务和应用程序托管在专用子域上。这些子域可以是任何内容，但通常情况下，公司使用对人类有意义且易于输入到 Web 浏览器中的名称。例如，您可能会在 *helpdesk.example.com* 找到帮助台门户，*monitoring.example.com* 上的监控系统，*jenkins.example.com* 上的持续集成系统，*mail.example.com* 上的电子邮件服务器以及 *ftp.example.com* 上的文件传输服务器。

我们如何为目标生成可能的子域列表？Bash 让这变得很容易。首先，我们需要一个常见子域的列表。您可以在 Kali 中找到这样的列表，位于 */usr/share/wordlists/amass/subdomains-top1mil-110000.txt* 或 */usr/share/wordlists/amass/bitquark_subdomains_top100K.txt*。要查找互联网上的单词列表，您可以使用以下 Google 搜索查询来搜索由社区成员提供的 GitHub 文件：**subdomain wordlist site:gist.github.com**。这将搜索 GitHub 上包含单词 *subdomain wordlist* 的代码片段（也称为 *gists*）。

出于本示例的目的，我们将使用*subdomains-1000.txt*，它包含在本章文件中，并且存储在书籍的 GitHub 存储库中。下载这个子域列表并将其保存在你的主目录中。该文件每行包含一个子域名，但没有关联的父域名。你需要将每个子域名与目标的父域名连接起来形成一个完全限定的域名。与前一节一样，我们将展示多种完成此任务的策略：使用 while 循环和使用 sed。

> 注意

*你可以从* [`github.com/dolevf/Black-Hat-Bash/blob/master/ch04`](https://github.com/dolevf/Black-Hat-Bash/blob/master/ch04) *下载本章的资源*。

列表 4-4 接受用户提供的父域名和一个单词列表，然后通过使用你之前下载的单词列表打印出一个完全限定子域名列表。

```
#!/bin/bash
DOMAIN="${1}"
FILE="${2}"

# Read the file from standard input and echo the full domain.
while read -r subdomain; do
  echo "${subdomain}.${DOMAIN}"
done < "${FILE}" 
```

列表 4-4：使用 while 循环生成子域名列表

该脚本使用 while 循环读取文件，并依次将每行赋值给变量 subdomain。然后 echo 命令将这两个字符串连接在一起以形成完整的域名。将此脚本保存为*generate_subdomains.sh*并向其提供两个参数：

```
$ **./generate_subdomains.sh example.com subdomains-1000.txt**

www.example.com
mail.example.com
ftp.example.com
localhost.example.com
webmail.example.com
`--snip--` 
```

第一个参数是父域名，第二个参数是包含所有可能子域名的文件的路径。

我们可以使用 sed 将内容写入文件每一行的末尾。在列表 4-5 中，该命令使用$符号来找到行尾，然后用目标域名前缀加上一个点(*.example.com*)来完成域名。

```
$ **sed 's/$/.example.com/g' subdomains-1000.txt**

relay.example.com
files.example.com
newsletter.example.com 
```

列表 4-5：使用 sed 生成子域名列表

sed 中参数的开头的 s 代表*substitute*，g 代表 sed 将在文件中替换所有匹配项，而不仅仅是第一个匹配项。因此，简单来说，我们用*.example.com*替换文件中每一行的末尾。如果你将此代码保存为脚本，输出应与前面的示例相同。

## 主机发现

当测试一系列地址时，你可能首先想要做的事情之一是获取有关它们的信息。它们是否有任何开放端口？这些端口后面有什么服务？它们是否容易受到安全漏洞的影响？手动回答这些问题是可能的，但如果你需要为数百甚至数千个主机执行此操作，这可能会很具挑战性。让我们使用 bash 来自动化网络枚举任务。

一种识别活动主机的方法是尝试发送网络数据包并等待它们返回响应。在本节中，我们将使用 bash 和其他网络实用程序来执行主机发现。

### ping

在其最基本形式中，ping 命令接受一个参数：目标 IP 地址或域名。运行以下命令来查看其输出：

```
$ **ping 172.16.10.10**

PING 172.16.10.10 (172.16.10.10) 56(84) bytes of data.
64 bytes from 172.16.10.10: icmp_seq=1 ttl=64 time=0.024 ms
64 bytes from 172.16.10.10: icmp_seq=2 ttl=64 time=0.029 ms
64 bytes from 172.16.10.10: icmp_seq=3 ttl=64 time=0.029 ms 
```

ping 命令将永远运行，因此按 CTRL-C 停止其执行。

如果你查看 ping 的手册页面（通过运行 man ping），你会注意到没有办法一次性对多个主机运行该命令。但是通过 bash，我们可以非常轻松地做到这一点。示例 4-6 对网络 172.16.10.0/24 上的所有主机进行 ping 测试。

```
#!/bin/bash
FILE="${1}"

❶ while read -r host; do
   ❷ if ping -c 1 -W 1 -w 1 "${host}" &> /dev/null; then
    echo "${host} is up."
  fi
❸ done < "${FILE}" 
```

示例 4-6：使用 while 循环 ping 多个主机

在❶处，我们运行一个 while 循环，从命令行传递给脚本的文件中读取内容。该文件被赋值给变量 FILE。我们从文件中读取每一行并将其赋值给 host 变量。然后我们运行 ping 命令，使用-c 参数并设置值为 1，在❷处告诉 ping 只发送一次 ping 请求并退出。默认情况下，在 Linux 上，ping 会无限期地发送 ping 请求，直到你手动通过发送 SIGHUP 信号（CTRL-C）停止它。

我们还使用了参数-W 1（设置超时时间，单位为秒）和-w 1（设置截止时间，单位为秒），以限制 ping 等待响应的时间。这非常重要，因为我们不希望 ping 在遇到无响应的 IP 地址时卡住；我们希望它继续从文件中读取，直到所有 254 个主机都经过测试。

最后，我们使用标准输入流读取文件，并将文件内容“传递”给 while 循环 ❸。

将此代码保存为*multi_host_ping.sh*并在传入*hosts*文件时运行。你应该会看到代码检测到一些活跃主机：

```
$ **./multi_host_ping.sh 172-16-10-hosts.txt**

172.16.10.1 is up.
172.16.10.10 is up.
172.16.10.11 is up.
172.16.10.12 is up.
172.16.10.13 is up. 
```

这种主机发现方法的一个警告是，某些主机，尤其是经过强化的主机，可能根本不会响应 ping 命令。因此，如果我们仅依赖这种方法进行发现，可能会错过网络上的活跃主机。

还需要注意的是，默认情况下会无限运行的命令（如 ping）在集成到 bash 脚本时可能会带来挑战。在本示例中，我们明确设置了一些特殊标志，以确保我们的 bash 脚本在执行 ping 时不会挂起。因此，在将命令集成到脚本中之前，先在终端中测试命令非常重要。通常，工具会提供一些特殊选项，确保它们不会永远执行，例如超时选项。

对于没有提供超时选项的工具，timeout 命令允许你在经过一段时间后自动退出命令。你可以将 timeout 加到任何 Linux 工具之前，并传递一个时间间隔（以*秒，分钟，小时*的格式）—例如，timeout 5s ping 8.8.8.8。在时间到达后，整个命令会退出。

### Nmap

Nmap 端口扫描器有一个特别的选项叫做-sn，它执行一个*ping 扫描*。这种简单的技术通过向网络上的主机发送 ping 命令并等待积极响应（称为*ping 响应*）来查找活跃主机。由于许多操作系统默认会响应 ping，因此这种技术被证明是非常有价值的。Nmap 中的 ping 扫描基本上会使 Nmap 通过网络发送互联网控制消息协议（ICMP）数据包，以发现正在运行的主机：

```
$ **nmap -sn 172.16.10.0/24**

Nmap scan report for 172.16.10.1
Host is up (0.00093s latency).
Nmap scan report for 172.16.10.10
Host is up (0.00020s latency).
Nmap scan report for 172.16.10.11
Host is up (0.00076s latency).
`--snip--` 
```

这个输出包含很多文本。通过一些 bash 魔法，我们可以使用 grep 和 awk 命令提取仅识别为活动的 IP 地址，以获得更清晰的输出。

```
$ **nmap -sn 172.16.10.0/24 | grep "Nmap scan" | awk -F'report for ' '{print $2}'**

172.16.10.1
172.16.10.10
`--snip--` 
```

图 4-7：解析 Nmap 的 ping 扫描输出

使用 Nmap 的内置 ping 扫描可能比手动使用 bash 包装 ping 实用程序更有用，因为您不必担心检查命令是否成功的条件。此外，在渗透测试中，您可能会在多种操作系统上放置 Nmap 二进制文件，并且相同的语法将始终在 ping 实用程序存在与否的情况下一致工作。

### arp-scan

我们可以远程执行渗透测试，从不同的网络或与目标相同的网络内部执行。在本节中，我们将强调使用 arp-scan 作为在本地进行测试时发现网络中主机的方法。

arp-scan 实用程序向网络上的主机发送地址解析协议（ARP）数据包，并显示它收到的任何响应。*ARP* 映射了分配给网络设备的唯一的 12 位十六进制地址（称为媒体访问控制（MAC）地址）到 IP 地址。因为 ARP 是 OSI 模型中的第二层协议，在本地网络上才有用；ARP 不能用于通过互联网执行远程扫描。

请注意，arp-scan 需要 root 权限才能运行；这是因为它使用需要提升权限的函数来读取和写入数据包。在其最基本的形式中，您可以通过执行 arp-scan 命令并将单个 IP 地址作为参数传递来运行它：

```
$ **sudo arp-scan 172.16.10.10 -I br_public**
```

我们还需要告诉 arp-scan 在哪个网络接口上发送数据包，因为 Kali 有几个网络接口。为此，我们使用 -I 参数。br_public 接口对应于实验室中的 172.16.10.0/24 网络。

要扫描整个网络，您可以传递 arp-scan 一个 CIDR 范围，例如 */24*。例如，以下命令扫描从 172.16.10.1 到 172.16.10.254 的所有 IP 地址：

```
$ **sudo arp-scan 172.16.10.0/24 -I br_public**
```

最后，您可以将在“连续 IP 地址”中创建的 hosts 文件作为 arp-scan 的输入：

```
$ **sudo arp-scan -f 172-16-10-hosts.txt -I br_public**
```

arp-scan 生成的输出应如下所示：

```
172.16.10.10  02:42:ac:10:0a:0a     (Unknown: locally administered)
172.16.10.11  02:42:ac:10:0a:0b     (Unknown: locally administered)
172.16.10.12  02:42:ac:10:0a:0c     (Unknown: locally administered)
172.16.10.13  02:42:ac:10:0a:0d     (Unknown: locally administered) 
```

此输出由三个字段组成：IP 地址、MAC 地址和供应商详细信息，由 MAC 地址的前三个八位组标识。在此扫描中，工具识别出网络上四个响应 ARP 数据包的主机。

练习 3：接收关于新主机的警报

假设你希望在网络上出现新主机时收到通知。例如，也许你想知道新的笔记本电脑或 IT 资产何时连接。如果你在测试不同时区的目标，当你在线时设备用户可能不在线，这可能很有用。

你可以使用 bash 在脚本发现新资产时向自己发送电子邮件。Listing 4-9 运行一个持续扫描，识别新的在线主机，将它们添加到“连续 IP 地址”中创建的 *172-16-10-hosts.txt* 文件，并通知你发现的结果。

因为这个脚本比之前的更复杂，我们将通过一个示例解决方案（Listing 4-8）来进行讲解，然后讨论如何自行改进它。

host_monitor _notification.sh

```
#!/bin/bash

# Sends a notification upon new host discovery
KNOWN_HOSTS="172-16-10-hosts.txt"
NETWORK="172.16.10.0/24"
INTERFACE="br_public"
FROM_ADDR="kali@blackhatbash.com"
TO_ADDR="security@blackhatbash.com"

❶ while true; do
  echo "Performing an ARP scan against ${NETWORK}..."

❷ sudo arp-scan -x -I ${INTERFACE} ${NETWORK} | while read -r line; do
  ❸ host=$(echo "${line}" | awk '{print $1}')
  ❹ if ! grep -q "${host}" "${KNOWN_HOSTS}"; then
      echo "Found a new host: ${host}!"
    ❺ echo "${host}" >> "${KNOWN_HOSTS}"
    ❻ sendemail -f "${FROM_ADDR}" \
        -t "${TO_ADDR}" \
        -u "ARP Scan Notification" \
        -m "A new host was found: ${host}"
    fi
  done

  sleep 10
done 
```

Listing 4-8：使用 sendemail 接收关于新 arp-scan 发现的通知

首先，我们设置一些变量。我们将包含要查找的主机的文件 *172-16-10-hosts.txt* 赋值给 KNOWN_HOSTS 变量，将目标网络 172.16.10.0/24 赋值给 NETWORK 变量。我们还设置了 FROM_ADDR 和 TO_ADDR 变量，用于发送通知邮件。

然后，我们使用 while 运行一个无限循环 ❶。除非我们主动退出，否则这个循环不会结束。在循环内部，我们使用 arp-scan 命令，并使用 -x 选项以显示纯文本输出（以便更容易解析），使用 -I 选项定义网络接口 br_public ❷。在同一行中，我们使用 while read 循环遍历 arp-scan 输出的内容。我们使用 awk 解析输出中的每个 IP 地址，并将其赋值给 host 变量 ❸。

在 ❹ 处，我们使用 if 条件判断主机变量（代表由 arp-scan 发现的主机）是否存在于我们的 *hosts* 文件中。如果存在，我们什么也不做；如果不存在，我们将其写入文件 ❺ 并通过 sendemail 命令发送电子邮件通知 ❻。注意，sendemail 命令中的每一行都以反斜杠（\）结尾。当行较长时，bash 允许我们以这种方式分隔它们，同时仍将其视为单个命令。将长代码行分开可以使其更易读。在此过程的最后，我们使用 sleep 10 等待 10 秒钟，然后再运行此发现过程。

如果运行此脚本，每当发现新主机时，你应该会收到一封电子邮件。为了正确发送邮件，你需要在系统上配置邮件传输代理，例如 Postfix。有关如何配置的更多信息，请参阅文档 *[`www.postfix.org/documentation.html`](https://www.postfix.org/documentation.html)*。

请注意，脚本执行的持续网络探测并不隐秘。若要更隐秘地探测网络，尝试以以下方式修改脚本：

+   降低探测速度，使其每隔几个小时或任意分钟数触发一次。你甚至可以将此间隔随机化，以使其更难预测。

+   如果你在受损的网络中运行脚本，尝试将结果写入内存，而不是通过网络发送通知。

+   将结果上传到看似无害的第三方网站。Living Off Trusted Sites（LOTS）项目在*[`lots-project.com`](https://lots-project.com)*维护着一个合法网站清单，企业网络通常允许这些网站。攻击者通常利用这些网站执行诸如数据外泄等活动，使其流量混入其他合法流量中，使分析师更难以发现。

现在您知道 172.16.10.0/24 网络上可用的主机，我们建议从*172-16-10-hosts.txt*文件中删除任何无响应的 IP 地址，以使您未来的扫描更快。

要进一步探索，我们鼓励您尝试其他通知传递方法，如 Slack、Discord、Microsoft Teams 或您日常使用的任何其他消息系统。例如，像 Slack 这样的平台使用*webhook*，它允许脚本向特定统一资源定位符（URL）发出 HTTP POST 请求，以向所选频道发送自定义消息。

## 端口扫描

一旦您发现了网络上的主机，您可以运行端口扫描程序来查找它们的开放端口和正在运行的服务。让我们通过使用三个工具来探索端口扫描：Nmap、RustScan 和 Netcat。

### Nmap

Nmap 允许我们针对单个目标或多个目标同时执行端口扫描。在下面的示例中，我们使用 Nmap 对域名*scanme.nmap.org*执行端口扫描：

```
$ **nmap scanme.nmap.org**
```

Nmap 还接受 IP 地址，如下所示：

```
$ **nmap 172.16.10.1**
```

当我们在命令行上未提供 Nmap 的任何特殊选项时，它将使用以下默认设置：

**执行 SYN 扫描 **Nmap 将使用同步（SYN）扫描来发现目标上的开放端口。也称为*半开放扫描*，*SYN 扫描*涉及发送一个 SYN 包并等待响应。Nmap 不会完成完整的 TCP 握手（也就是不会发送 ACK），这就是为什么我们称这种扫描为*半开放*。

**扫描前 1000 个端口 **Nmap 只会扫描那些经常使用的热门端口，比如 TCP 端口 21、22、80 和 443。它不会扫描整个端口范围 0 至 65,534，以节约资源。

**扫描 TCP 端口 **Nmap 只会扫描 TCP 端口，不会扫描用户数据报协议（UDP）端口。

Nmap 允许您通过命令行传递多个目标进行扫描。在下面的示例中，我们同时扫描*localhost*和*scanme.nmap.org*：

```
$ **nmap localhost scanme.nmap.org**
```

当传递-iL 选项时，Nmap 还可以从给定文件中读取目标。目标必须以新行分隔。让我们使用*Nmap 扫描多个目标的 172-16-10-hosts.txt*文件：

```
$ **nmap -sV -iL 172-16-10-hosts.txt**

`--snip--`
Nmap scan report for 172.16.10.1
Host is up (0.00028s latency).
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.0p1 Debian 1+b2 (protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
`--snip--`

Nmap scan report for 172.16.10.10
Host is up (0.00029s latency).
PORT     STATE SERVICE          VERSION
8081/tcp open  blackice-icecap?
`--snip--` 
```

由于使用了-sV 选项，该扫描可能需要一些时间来完成，该选项用于检测每个端口的服务版本。正如您所见，Nmap 返回了一些 IP 地址及其开放端口，包括它们的服务甚至与主机上运行的操作系统相关的信息。如果我们想要过滤，比如只想看到开放的端口，我们可以使用 grep：

```
$ **nmap -sV -iL 172-16-10-hosts.txt | grep open**

22/tcp open  ssh
8081/tcp open  blackice-icecap
21/tcp open  ftp
80/tcp open  http
80/tcp open  http
22/tcp open  ssh
`--snip--` 
```

Nmap 能够识别几个开放的 TCP 端口上的服务，例如端口 21 上的文件传输协议 (FTP)，端口 22 上的安全外壳协议 (SSH)，以及端口 80 上的 HTTP。在本章后面，我们将更详细地查看每个服务。

Nmap 也允许您在命令行上使用 --open 标志，仅显示找到的开放端口：

```
$ **nmap -sV -iL 172-16-10-hosts.txt --open**
```

Kali 的本机接口 IP（172.16.10.1）将在此端口扫描中被捕获，因为它在 hosts 文件中。您可以使用 Nmap 的 --exclude 选项在执行网络范围扫描时排除此特定 IP：--exclude 172.16.10.1。您还可以从文件中手动删除它以方便操作。

使用 man nmap 命令了解更多关于 Nmap 扫描和过滤功能的信息。

### RustScan

RustScan 在漏洞赏金和渗透测试领域越来越受欢迎，因为其速度和可扩展性。以下是 rustscan 命令执行的端口扫描。-a（地址）参数接受单个地址或地址范围：

```
$ **rustscan -a 172.16.10.0/24**

Open 172.16.10.11:21
Open 172.16.10.1:22
Open 172.16.10.13:22
`--snip--` 
```

RustScan 的输出非常适合用 bash 解析。以 Open 开头的行表明在特定 IP 地址上找到了开放端口。这些行后面跟着 IP 地址和端口，用冒号分隔。

当您运行 RustScan 时，您可能会注意到初始输出包含横幅、作者信用和与扫描结果无直接关系的其他信息。使用 -g（grepable）选项仅显示扫描信息。以下命令使用 grepable 输出模式扫描 172.16.10.0/24 的前 1024 个端口（也称为 *特权端口*）：

```
$ **rustscan -g -a 172.16.10.0/24 -r 1-1024**

172.16.10.11 -> [80]
172.16.10.12 -> [80] 
```

现在输出更适合用 grep。要解析它，我们只需传递分隔符 ->，用 awk 分隔 IP 地址和端口：

```
$ **rustscan -g -a 172.16.10.0/24 -r 1-1024 | awk -F'->' '{print $1,$2}'**
```

此命令输出两个字段：IP 地址和端口。为了去掉周围的 [] 括号，我们使用 tr 命令和 -d（删除）参数后跟要删除的字符：

```
$ **rustscan -g -a 172.16.10.0/24 -r 1-1024 | awk -F'->' '{print $1,$2}' | tr -d '[]'**
```

这应该返回更清晰的输出。

> 警告

*请记住，在激进模式下运行端口扫描程序会增加被发现的风险，特别是如果目标使用入侵检测系统或终端响应系统。此外，如果您以快速的速度扫描，可能会因网络洪水而导致拒绝服务。*

### Netcat

您也可以使用 Netcat 进行端口扫描活动。当人们想要检查单个端口的状态（如是否开放或关闭）时，通常会使用此工具，但 Netcat 也可以让您用单个命令扫描多个端口。让我们看看如何实现这一点。

运行以下命令扫描 172.16.10.11 上的 TCP 端口 1–1024：

```
$ **nc -zv 172.16.10.11 1-1024**

`--snip--`

(UNKNOWN) [172.16.10.11] 80 (http) open
(UNKNOWN) [172.16.10.11] 21 (ftp) open 
```

我们使用 nc 命令，带有 -z 标志（零输入/输出模式，不会发送任何数据）和 -v 标志（详细模式），然后是目标 IP 和用连字符（-）分隔的端口范围。如输出所示，找到两个开放端口。

练习 4：整理扫描结果

将扫描结果按感兴趣的类别进行分类通常非常有用。例如，你可以将每个 IP 地址的结果导出到一个专门的文件中，或根据发现的软件版本来整理结果。在本练习中，你将根据端口号来组织扫描结果。编写一个脚本，实现以下功能：

1.  对文件中的主机运行 Nmap

2.  使用 bash 创建以开放端口为文件名的单独文件

3.  在每个文件中，写入对应端口开放时的 IP 地址

在本练习的最后，你应该会有一堆文件，例如 *port-22.txt*、*port-80.txt* 和 *port-8080.txt*，在每个文件中，你应该能看到一个或多个 IP 地址，表示该端口被发现开放。当你有大量目标主机时，这可以非常有用，尤其是当你希望通过针对与特定端口相关的协议进行群体攻击时。

为了帮助你入门，清单 4-9 显示了一个示例解决方案。

nmap_to_portfiles.sh

```
#!/bin/bash
HOSTS_FILE="172-16-10-hosts.txt"
❶ RESULT=$(nmap -iL ${HOSTS_FILE} --open | grep "Nmap scan report\|tcp open")

# Read the nmap output line by line.
while read -r line; do
❷ if echo "${line}" | grep -q "report for"; then
    ip=$(echo "${line}" | awk -F'for ' '{print $2}')
  else
  ❸ port=$(echo "${line}" | grep open | awk -F'/' '{print $1}')
  ❹ file="port-${port}.txt"
  ❺ echo "${ip}" >> "${file}"
  fi
done <<< "${RESULT}" 
```

清单 4-9：使用 bash 根据端口组织扫描结果

我们将 nmap 命令的输出赋值给变量 NMAP_RESULT ❶。在这个命令中，我们还会过滤出包含“Nmap scan report”或“tcp open”字样的特定行。这些行是 Nmap 标准端口扫描输出的一部分，表示在某个 IP 地址上发现了开放端口。

我们使用 while 循环逐行读取 NMAP_RESULT，检查每一行是否包含字符串“report for ❷”。这一行将包含发现端口开放的 IP 地址。如果找到这样的行，我们将其赋值给 ip 变量。然后，我们解析该行，提取发现开放的端口 ❸。在 ❹ 处，我们创建文件变量，用于保存我们将创建的磁盘文件，命名方案为 *port-NUMBER.txt*。最后，我们将 IP 地址附加到文件 ❺。

将脚本保存为 *nmap_to_portfiles.sh* 文件并运行。接下来，运行 ls -l 查看哪些文件已创建，并使用 cat 查看它们的内容：

```
$ **ls -l**

total 24
-rw-r--r-- 1 kali kali 3448 Mar  6 22:18 172-16-10-hosts.txt
-rw-r--r-- 1 kali kali   13 Mar  8 22:34 port-21.txt
-rw-r--r-- 1 kali kali   25 Mar  8 22:34 port-22.txt
`--snip--`

$ **cat port-21.txt**

172.16.10.11 
```

如你所见，Nmap 的标准输出格式稍微有些难以解析，但并非不可能。

为了改进此脚本，可以考虑使用 Nmap 的其他输出格式选项，这些选项可以简化解析，尤其是在脚本编写时。其中一个选项是 -oG 标志，它用于生成便于 grep 和 awk 使用的可搜索输出格式：

```
$ **nmap -iL 172-16-10-hosts.txt --open -oG -**

Host: 172.16.10.1 ()    Status: Up
Host: 172.16.10.1 ()    Ports: 22/open/tcp//ssh///  Ignored State: closed (999)
Host: 172.16.10.10 ()   Status: Up
Host: 172.16.10.10 ()   Ports: 8081/open/tcp//blackice-icecap///  Ignored State: closed (999)
`--snip--` 
```

输出现在会在同一行打印 IP 地址及其开放的端口。

你还可以通过使用 -oX 选项让 Nmap 生成可扩展标记语言（XML）输出。XML 格式的 Nmap 输出中的开放端口如下所示：

```
$ **nmap -iL 172-16-10-hosts.txt --open -oX -**

`--snip--`
<port protocol="tcp" portid="22"><state state="open" reason="syn-ack" reason_ttl="0"/><service
name="ssh" method="table" conf="3"/></port>
`--snip--` 
```

作为额外挑战，尝试编写一个单行的 bash 脚本，从 XML 输出中提取开放端口。

## 检测新的开放端口

如果你想监视某台主机，直到它开放某个特定端口怎么办？如果你正在测试一个主机频繁上线和下线的环境，你可能会发现这非常有用。我们可以通过 while 循环轻松实现这一点。

在清单 4-10 中，我们持续检查端口是否打开，每次执行之间间隔五秒。一旦我们找到一个开放端口，我们将这些信息传递给 Nmap 进行服务发现，并将结果写入文件。

port _watchdog.sh

```
#!/bin/bash
LOG_FILE="watchdog.log"
IP_ADDRESS="${1}"
WATCHED_PORT="${2}"

service_discovery(){
  local host
  local port
  host="${1}"
  port="${2}"

❶ nmap -sV -p "${port}" "${host}" >> "${LOG_FILE}"
}

❷ while true; do
❸ port_scan=$(docker run --network=host -it --rm \
             --name rustscan rustscan/rustscan:2.1.1 \
             -a "${IP_ADDRESS}" -g -p "${WATCHED_PORT}")
❹ if [[-n "${port_scan}"]]; then
    echo "${IP_ADDRESS} has started responding on port ${WATCHED_PORT}!"
    echo "Performing a service discovery..."
  ❺ if service_discovery "${IP_ADDRESS}" "${WATCHED_PORT}"; then
      echo "Wrote port scan data to ${LOG_FILE}"
      break
    fi
  else
    echo "Port is not yet open, sleeping for 5 seconds..."
  ❻ sleep 5
  fi
done 
```

清单 4-10：新开放端口的看门狗脚本

在❷，我们启动一个无限循环。该循环运行 RustScan，并传递给它包含我们在命令行中收到的 IP 地址的-a（地址）参数❸。我们还传递 RustScan -g（greppable）选项，生成适合 grep 的格式，并传递端口选项（-p）以扫描特定端口，这个端口同样是我们从命令行接收的，结果被分配给 port_scan 变量。

我们检查扫描结果❹。如果结果不为空，我们将 IP 地址和端口传递给 service_discovery 函数❺，该函数执行 Nmap 服务版本发现扫描（-sV），并将结果写入日志文件*watchdog.log*❶。如果端口扫描失败，意味着端口已关闭，我们将暂停五秒❻。因此，整个过程将每五秒重复一次，直到找到开放的端口。

保存脚本，然后使用以下参数运行它：

```
$ **./port_watchdog.sh 127.0.0.1 3337**
```

由于本地主机的此端口不应有任何服务在运行，脚本将永远运行。我们可以通过使用 Python 的内置*http.server*模块来模拟端口打开事件，该模块启动一个简单的 HTTP 服务器：

```
$ **python3 -m http.server 3337**
```

现在，*port_watchdog.sh*脚本应显示以下内容：

```
Port is not yet open, sleeping for 5 seconds...
127.0.0.1 has started responding on port 3337!
Performing a service discovery...
Wrote port scan data to watchdog.log 
```

你可以通过打开*watchdog.log*文件来查看扫描结果：

```
$ **cat watchdog.log**
Starting Nmap (https://nmap.org)
Nmap scan report for 172.16.10.10
Host is up (0.000099s latency).

PORT     STATE SERVICE          VERSION
3337/tcp open  SimpleHTTPServer
`--snip--` 
```

使用这个脚本，你应该能够在网络上识别出四个具有开放端口的 IP 地址：172.16.10.10（属于*p-web-01*机器）运行 8081/TCP；172.16.10.11（属于*p-ftp-01*机器）同时运行 21/TCP 和 80/TCP；172.16.10.12（属于*p-web-02*机器）运行 80/TCP；以及 172.16.10.13（属于*p-jumpbox-01*机器）运行 22/TCP。

## 横幅抓取

了解远程服务器上运行的软件是渗透测试中的关键步骤。在本章的其余部分，我们将探讨如何识别端口和服务背后的内容——例如，端口 8081 上运行的是什么 Web 服务器，它使用哪些技术为客户端提供内容？

*Banner grabbing*（横幅抓取）是指提取远程网络服务在建立连接时发布的信息的过程。服务通常会传输这些横幅以“迎接”客户端，客户端可以以多种方式利用提供的信息，例如确保它们连接到正确的目标。横幅还可能包含系统管理员的每日消息或服务的特定运行版本。

*被动横幅抓取*使用第三方网站查找横幅信息。例如，像 Shodan (*[`shodan.io`](https://shodan.io)*)、ZoomEye (*[`zoomeye.org`](https://zoomeye.org)*)、Censys (*[`censys.io`](https://censys.io)*) 这样的网站执行扫描，映射互联网，抓取横幅、版本、网页和端口，然后利用这些数据创建清单。我们可以使用这些网站查找横幅信息，而无需与目标服务器直接交互。

*主动横幅抓取*则恰恰相反；它建立与服务器的连接并直接与其交互，以接收其横幅信息。通过横幅进行自我宣传的网络服务包括 Web 服务器、SSH 服务器、FTP 服务器、Telnet 服务器、网络打印机、物联网设备和消息队列等。

请记住，横幅通常是自由格式的文本字段，可以更改以误导客户端。例如，Apache Web 服务器可能会伪装成另一种类型的 Web 服务器，如 nginx。一些组织甚至会创建 *蜜罐服务器* 来引诱威胁行为者（或渗透测试者）。蜜罐利用欺骗技术伪装成脆弱的服务器，但它们的真正目的是检测和分析攻击者的活动。然而，更多时候，横幅传输的是系统管理员没有更改的默认设置。

### 使用主动横幅抓取

为了展示什么是主动横幅抓取，我们将使用以下 Netcat 命令连接到 IP 地址 172.16.10.11 上运行的端口 21（FTP）（*p-ftp-01*）：

```
$ **nc 172.16.10.11 -v 21**

172.16.10.11: inverse host lookup failed: Unknown host
(UNKNOWN) [172.16.10.11] 21 (ftp) open
220 (**vsFTPd 3.0.5**) 
```

如你所见，172.16.10.11 正在运行 FTP 服务器 vsFTPd 版本 3.0.5。这些信息可能会发生变化，具体取决于 vsFTPd 版本的升级或降级，或者系统管理员是否决定在 FTP 服务器的配置中完全禁用横幅广告。

Netcat 是一个很好的例子，说明了一个工具本身并不支持探测多个 IP 地址。因此，了解一些 bash 脚本编程可以帮助我们解决这个问题。Listing 4-11 将使用 Netcat 从一个文件中保存的多个主机抓取端口 21 上的横幅。

netcat_banner _grab.sh

```
#!/bin/bash
FILE="${1}"
PORT="${2}"

❶ if [["$#" -ne 2]]; then
  echo "Usage: ${0} <file> <port>"
  exit 1
fi

❷ if [[! -f "${FILE}"]]; then
  echo "File: ${FILE} was not found."
  exit 1
fi

❸ if [[! "${PORT}" =~ ^[0-9]+$]]; then
  echo "${PORT} must be a number."
  exit 1
fi

❹ while read -r ip; do
  echo "Running netcat on ${ip}:${PORT}"
  result=$(echo -e "\n" | nc -v "${ip}" -w 1 "${PORT}" 2> /dev/null)
❺ if [[-n "${result}"]]; then
    echo "==================="
    echo "+ IP Address: ${ip}"
    echo "+ Banner: ${result}"
    echo "==================="
  fi
done < "${FILE}" 
```

Listing 4-11：使用 Netcat 进行横幅抓取

这个脚本在命令行接受两个参数：FILE 和 PORT。我们使用 if 条件语句检查是否确实传入了两个参数 ❶；如果没有，我们以状态码 1（失败）退出，并打印一个使用提示，说明如何运行该脚本。然后我们使用另一个 if 条件语句，并通过 -f 测试检查用户提供的文件是否确实存在于磁盘上 ❷。

在 ❸ 处，我们检查用户提供的端口是否为数字。任何非数字的输入都将失败。然后，我们逐行读取主机文件并在每个 ❹ 处运行给定端口上的 nc (Netcat) 命令。我们使用另一个 if 条件来检查命令结果是否不为空 ❺，这意味着发现了开放的端口，并打印从服务器返回的 IP 地址和数据。

### 检测 HTTP 响应

生产系统上经常会出现流行的 curl HTTP 客户端。为了对 HTTP 响应执行横幅抓取，我们可以使用 curl 发送 HTTP 请求，使用 HEAD 方法。HEAD 方法允许我们读取响应头，而无需从 Web 服务器获取整个响应载荷。

Web 服务器通常通过将 Server HTTP 响应头设置为其名称来进行自我宣传。有时您可能还会在那里遇到宣传的运行版本。以下 curl 命令向 *p-web-01* 机器（172.16.10.10:8081）发送 HTTP HEAD 请求：

```
$ **curl --head 172.16.10.10:8081**

HTTP/1.1 200 OK
Server: Werkzeug/2.2.3 Python/3.11.1
`--snip--`
Content-Length: 7176
Connection: close 
```

正如你所见，服务器在响应中返回了一堆头信息，其中之一是 Server 头信息。这个头信息显示远程服务器正在运行一个名为 Werkzeug 版本 2.2.3 的基于 Python 的 Web 框架，由 Python 版本 3.11.1 驱动。

列表 4-12 将这个 curl 命令整合到一个更大的脚本中，使用 bash read 命令提示用户输入信息，然后向用户呈现一个横幅。

curl_banner _grab.sh

```
#!/bin/bash
DEFAULT_PORT="80"

❶ read -r -p "Type a target IP address: " ip
❷ read -r -p "Type a target port (default: 80): " port

❸ if [[-z "${ip}"]]; then
  echo "You must provide an IP address."
  exit 1
fi

❹ if [[-z "${port}"]]; then
  echo "You did not provide a specific port, defaulting to ${DEFAULT_PORT}"
❺ port="${DEFAULT_PORT}"
fi

echo "Attempting to grab the Server header of ${ip}..."

❻ result=$(curl -s --head "${ip}:${port}" | grep Server | awk -F':' \
        '{print $2}')

echo "Server header for ${ip} on port ${port} is: ${result}" 
```

列表 4-12：从 Web 服务器提取服务器响应头

这个交互式脚本要求用户在命令行中提供有关目标的详细信息。首先，我们使用 read 命令提示用户输入 IP 地址，并将这个值分配给 ip_address 变量 ❶。然后，我们要求用户输入所需的端口号，并将其保存到 port 变量 ❷ 中。

在 ❸ 处，我们通过使用 -z 测试检查 ip_address 变量的长度是否为零，并在此条件为真时退出。接下来，我们对端口变量 ❹ 进行相同的检查。这次，如果用户没有提供端口，我们使用默认的 HTTP 端口 80 ❺。在 ❻ 处，我们将输出存储到 result 变量中。我们使用 grep 和 awk 解析 curl 的结果并提取 Server 头信息。

运行脚本，并在提示时提供 IP 地址 172.16.10.10 和端口 8081：

```
$ **./curl_banner_grab**

Type a target IP address: **172.16.10.10**
Type a target port (default: 80): **8081**
Attempting to grab the Server header of 172.16.10.10...
Server header for 172.16.10.10 on port 8081 is: Werkzeug/2.2.3 Python/3.11.1 
```

正如你所见，脚本返回了从目标 IP 地址和端口获取的正确信息。如果我们在终端中没有指定端口，它会默认使用端口 80\. 请注意，我们也可以使用 Netcat 发送 HTTP HEAD 请求，但了解多种实现给定任务的方法是很有用的。

### 使用 Nmap 脚本

Nmap 不仅仅是一个端口扫描器；我们可以将其转化为一个功能齐全的漏洞评估工具。*Nmap 脚本引擎 (NSE)* 允许渗透测试人员使用 Lua 语言编写脚本来扩展 Nmap 的功能。Nmap 预装了一些 Lua 脚本，正如你在这里所见：

```
$ **ls -l /usr/share/nmap/scripts**

-rw-r--r-- 1 root root  3901 Oct  6 10:43 acarsd-info.nse
-rw-r--r-- 1 root root  8749 Oct  6 10:43 address-info.nse
-rw-r--r-- 1 root root  3345 Oct  6 10:43 afp-brute.nse
-rw-r--r-- 1 root root  6463 Oct  6 10:43 afp-ls.nse
-rw-r--r-- 1 root root  3345 Oct  6 10:43 afp-brute.nse
-rw-r--r-- 1 root root  6463 Oct  6 10:43 afp-ls.nse
`--snip--` 
```

*banner.nse* 脚本位于 */usr/share/nmap/scripts* 文件夹中，允许你同时从多个主机抓取 banner。以下 bash 命令使用此脚本执行 banner 抓取和服务发现（-sV）：

```
$ **nmap -sV --script=banner.nse -iL 172-16-10-hosts.txt**

Nmap scan report for 172.16.10.12
`--snip--`
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.54 ((Debian))
|_http-server-header: Apache/2.4.54 (Debian)
`--snip--` 
```

当 banner 抓取脚本找到一个 banner 时，包含该 banner 的输出行将以特殊的字符序列（|_）开头。我们可以过滤这个序列来提取 banner 信息，方法如下：

```
$ **nmap -sV --script=banner.nse -iL 172-16-10-hosts.txt | grep "|_banner\||_http-server-header"**
```

你可能注意到，在 172.16.10.10 的 8081 端口（*p-web-01* 机器）的情况下，Nmap 会做出如下回应：

```
PORT     STATE SERVICE          VERSION
8081/tcp open  blackice-icecap?
| fingerprint-strings:
`--snip--` 
```

blackice-icecap? 值表示 Nmap 无法明确发现服务的身份。但如果你仔细查看指纹 -strings 转储，你会看到一些与 HTTP 相关的信息，揭示了我们手动使用 curl 进行 banner 抓取时发现的相同响应头。具体来说，请注意 Werkzeug Web 服务器的 banner。稍微 Google 一下，你会发现这个服务器运行在 Flask 上，这是一个基于 Python 的 Web 框架。

### 操作系统检测

Nmap 还可以通过使用 *TCP/IP 指纹识别* 来猜测目标服务器的操作系统，这也是其操作系统检测扫描的一部分。此技术通过以不同方式构造数据包并分析返回的响应，识别操作系统 TCP/IP 堆栈的实现。每个操作系统（如 Linux、Windows 和 macOS）对 TCP/IP 堆栈的实现略有不同，Nmap 分析这些微妙的差异来识别运行的系统。在某些情况下，Nmap 还可能能够识别运行的内核版本。

要运行操作系统检测扫描，请在 Nmap 中使用 -O 标志。请注意，此扫描需要 sudo 权限：

```
$ **sudo nmap -O -iL 172-16-10-hosts.txt**

`--snip--`
21/tcp open  ftp
80/tcp open  http
MAC Address: 02:42:AC:10:0A:0B (Unknown)
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.6
Network Distance: 1 hop 
```

让我们创建一个 bash 脚本，解析此输出并按 IP 地址和操作系统进行排序（Listing 4-13）。

os_detection.sh

```
#!/bin/bash
HOSTS="$*"

❶ if [["${EUID}" -ne 0]]; then
  echo "The Nmap OS detection scan type (-O) requires root privileges."
  exit 1
fi

❷ if [["$#" -eq 0]]; then
  echo "You must pass an IP or an IP range"
  exit 1
fi

echo "Running an OS Detection Scan against ${HOSTS}..."

❸ nmap_scan=$(sudo nmap -O ${HOSTS} -oG -)
❹ while read -r line; do
  ip=$(echo "${line}" | awk '{print $2}')
  os=$(echo "${line}" | awk -F'OS: ' '{print $2}' | sed 's/Seq.*//g')

❺ if [[-n "${ip}"]] && [[-n "${os}"]]; then
    echo "IP: ${ip} OS: ${os}"
  fi
done <<< "${nmap_scan}" 
```

Listing 4-13: 解析操作系统检测扫描

因为此扫描需要 root 权限，我们会检查有效用户的 ID ❶。如果用户 ID 不等于零，则退出，因为如果用户没有使用 root 权限，继续进行没有意义。然后我们检查用户是否在命令行中传递了目标主机 ❷。在 ❸，我们对这些目标运行 Nmap 操作系统检测扫描，并将其分配给 HOSTS 变量。

我们使用一个 while 循环 ❹ 遍历扫描结果，解析每一行，并将输出中的 IP 地址赋值给 ip 变量。然后我们再次解析该行，提取 Nmap 的操作系统信息。我们使用 sed 清理输出，使其仅显示操作系统信息，删除 Seq 字样后的所有内容。接下来，我们检查 ip 和 os 变量是否都已设置 ❺。如果都已设置，意味着我们已正确解析了输出，可以通过打印 IP 地址和操作系统类型来完成脚本。

为了理解我们为什么以这种方式解析输出，使用 grep、awk 和 sed，可以在一个单独的终端中运行以下命令：

```
$ **sudo nmap -O 172.16.10.0/24 -oG -**

`--snip--`
Host: 172.16.10.10 () Ports: 8081/open/tcp//blackice-icecap/// Ignored State: closed (999) OS:
Linux 4.15 - 5.6   Seq Index: 258   IP ID Seq: All zeros
`--snip--` 
```

如你所见，输出是由空格分隔的。IP 地址位于第一个空格之后，操作系统类型位于 OS: 之后，但在 Seq 之前，这就是为什么我们需要提取这两个词之间的文本。你也可以用其他方法进行解析，比如使用正则表达式；这只是实现任务的一种方法。

使用以下命令保存并运行脚本：

```
$ **sudo ./os_detection.sh 172.16.10.0/24**

Running an OS Detection Scan against 172.16.10.0/24...
IP: 172.16.10.10 OS: Linux 4.15 - 5.6
IP: 172.16.10.11 OS: Linux 4.15 - 5.6
IP: 172.16.10.12 OS: Linux 4.15 - 5.6
IP: 172.16.10.13 OS: Linux 4.15 - 5.6
IP: 172.16.10.1 OS: Linux 2.6.32 
```

到此为止，我们已经识别出几个 HTTP 服务器、一个 FTP 服务器和一个 SSH 服务器。接下来，我们仔细看看 HTTP 服务器。

### 网站分析与 JSON

让我们使用 WhatWeb 查看 172.16.10.0/24 网络中运行的 Web 应用程序的服务。我们将首先查看 172.16.10.10（*p-web-01*）上的 8081 端口：

```
$ **whatweb 172.16.10.10:8081**

http://172.16.10.10:8081 [200 OK] Country[RESERVED][ZZ], HTML5,
HTTPServer[Werkzeug/2.3.7 Python/3.11.4], IP[172.16.10.10],
Python[3.11.4], Title[Menu], Werkzeug[2.3.7], X-UA-Compatible[ie=edge]
`--snip--` 
```

WhatWeb 的输出默认打印到标准输出，内容由空格和逗号分隔。如你所见，它找到了关于此 Web 服务器运行技术的一些信息。

我们可以使用 awk 和 grep 等工具轻松解析此输出，但为了向你介绍一些新技巧，我们将探索如何解析 *JavaScript 对象表示法（JSON）* 输出。JSON 是由键和值组成的数据格式。为了解析它，使用像 jq 这样的工具来遍历 JSON 结构并提取我们需要的信息会非常有帮助。

WhatWeb 可以通过 --log-json 参数将输出格式化为 JSON，该参数需要传入一个文件名作为值。但如果我们想将输出发送到屏幕而不写入磁盘怎么办？我们可以将参数指定为 */dev/stdout* 文件，强制它将输出发送到标准输出：

```
$ **whatweb 172.16.10.10:8081 --log-json=/dev/stdout --quiet | jq**

[
  {
`--snip--`
    "plugins": {
      "Country": {
        "string": [
          "RESERVED"
        ],
        "module": [
          "ZZ"
        ]
      },
      "HTML5": {},
      "HTTPServer": {
        "string": [
          "Werkzeug/2.3.7 Python/3.11.4"
        ]
      },
      "IP": {
        "string": [
          "172.16.10.10"
        ]
      },
 "Python": {
        "version": [
          "3.11.4"
        ]
      },
      "Title": {
        "string": [
          "Menu"
        ]
      },
      "Werkzeug": {
        "version": [
          "2.3.7"
        ]
      },
      "X-UA-Compatible": {
        "string": [
          "ie=edge"
        ]
      }
    }
  }
]
`--snip--` 
```

现在，输出被打印到标准输出，并以 JSON 格式显示。如你所见，我们获得的信息与运行基础的 whatweb 命令时得到的信息相同，只是没有特殊的格式。

输出是一个对象数组，我们可以使用像 jq 这样的工具来提取相关信息。例如，让我们提取 HTTPServer 的值：

```
$ **whatweb 172.16.10.10:8081 --log-json=/dev/stdout --quiet |**
**jq '.[0].plugins.HTTPServer.string[0]'**

"Werkzeug/2.3.7 Python/3.11.4" 
```

jq 语法刚开始可能看起来有点奇怪，我们来逐步解析它。我们将提取模式放在两个单引号（'）之间。在这里，我们选择数组中的第一个元素（.[0]），该元素包含由键和值组成的各种对象。接着，我们选择 plugins 键，再选择 HTTPServer 键。在 HTTPServer 键内，有一个名为 string 的键，它是一个数组。我们使用 string[0] 来选择该数组中的第一个元素，它包含的值是 Werkzeug/2.3.7 Python/3.11.4。

同样地，我们可以提取 IP 地址。只需将 HTTPServer 键替换为 IP 键：

```
$ **whatweb 172.16.10.10:8081 --log-json=/dev/stdout --quiet | jq '.[0].plugins.IP.string[0]'**

"172.16.10.10" 
```

继续运行 WhatWeb，检查我们已识别的每个 Web 服务器，查看它们运行的技术。

## 总结

在本章中，我们以多种方式运用了 bash。我们创建了动态目标主机列表；使用多种工具执行主机发现、端口扫描和横幅获取；创建了一个自动化脚本来通知我们新发现的主机；并解析了各种工具的结果。在下一章中，我们将对这些目标运行漏洞扫描器和模糊测试工具。
