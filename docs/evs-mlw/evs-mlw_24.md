

# 第二十一章：C 进一步阅读与资源



![](img/opener.jpg)

本附录列出了进一步阅读的书籍和资源，它们可以成为本书的极佳补充。有些提供了对本书中仅仅提到的内容的深入探讨，而其他的则补充了书中的讨论，或专门聚焦于某个特定的工具或方法。在本附录的末尾，你会找到一份在线沙箱和恶意软件来源的列表，帮助你开始练习书中展示的技巧。

## 第一部分（第 1–3 章）

+   Eagle, Chris, *《IDA Pro 书籍：世界上最流行的反汇编工具非官方指南》，第 2 版*。旧金山：No Starch Press，2011 年。这是一本关于 IDA Pro 反汇编工具和调试器的终极指南，涵盖了从浏览 IDA Pro 界面到使用插件及其一些高级功能的所有内容。

+   Eagle, Chris, 和 Kara Nance. *《Ghidra 书籍：权威指南》*。旧金山：No Starch Press，2020 年。Eagle 和 Nance 介绍了使用 Ghidra 反汇编工具的基础知识，以及该工具的一些高级使用方法。

+   Kleymenov, Alexey, 和 Amr Thabet. *《恶意软件分析精通：恶意软件分析师的实用指南，打击恶意软件、APT、网络犯罪和物联网攻击》*。伯明翰，英国：Packt，2022 年。这是一本现代、全面的恶意软件分析概念和技术书籍，甚至涵盖了 Linux 和物联网恶意软件以及 macOS 和 iOS 威胁等领域。

+   Sikorski, Michael, 和 Andrew Honig. *《实用恶意软件分析：动手解剖恶意软件指南》*。旧金山：No Starch Press，2012 年。这本书是最早的全面恶意软件分析指南之一。即使这本书已经出版了十多年，书中讨论的许多技术至今仍然非常具有相关性。

+   Yosifovich, Pavel, Mark E. Russinovich, Alex Ionescu, 和 David A. Solomon. *《Windows 内部原理，第一部分：系统架构、进程、线程、内存管理等》，第 7 版*。雷德蒙德，WA：Microsoft Press，2017 年；Allievi, Andrea, Alex Ionescu, David A. Solomon, 和 Mark E. Russinovich. *《Windows 内部原理，第二部分》，第 7 版*。雷德蒙德，WA：Microsoft Press，2021 年。两本 *《Windows 内部原理》* 书籍提供了关于 Windows 工作原理的极为详细的解析。如果你想深入了解 Windows 架构，这两本书是最佳选择。

## 第二至四部分（第 4–17 章）

+   Andriesse, Dennis. *《实用二进制分析：构建你自己的 Linux 工具进行二进制插桩、分析和反汇编》*。旧金山：No Starch Press，2018 年。本书探讨了分析各种类型二进制文件的技术，涵盖了一些我在这里只简要介绍的概念，比如二进制插桩。

+   Hand, Matt. *规避 EDR：终端检测系统击败终极指南*。旧金山：No Starch Press，2023 年。Hand 涵盖了大量关于终端检测与响应（EDR）系统的信息，包括其一般架构和规避策略。

+   Ligh, Michael Hale, Andrew Case, Jamie Levy, 和 Aaron Walters. *内存取证艺术：检测 Windows、Linux 和 Mac 内存中的恶意软件和威胁*。霍博肯，新泽西州：Wiley，2014 年。作者详细解释了内存和内存取证这一复杂话题——这是本书中我仅简要提到的概念。

+   Matrosov, Alex, Eugene Rodionov, 和 Sergey Bratus. *Rootkits 和 Bootkits：逆向现代恶意软件和下一代威胁*。旧金山：No Starch Press，2019 年。本书深入探讨了现代 Rootkits 和 Bootkits，以及如何从恶意软件分析和取证的角度进行调查。

+   MITRE ATT&CK ([*https://<wbr>attack<wbr>.mitre<wbr>.org*](https://attack.mitre.org)) 是一个记录威胁行为者使用技术的知识库，是本书几乎所有主题的绝佳伴随资源。

+   Unprotect ([*https://<wbr>unprotect<wbr>.it*](https://unprotect.it)) 是由研究人员 Jean-Pierre Lesueur 和 Thomas Roccia 维护的一个项目，旨在对恶意软件规避技术进行分类。这是本书的另一个优秀伴随资源。

+   Yason, Mark Vincent. “解包艺术。”亚特兰大：IBM 互联网安全系统，2011 年。[*https://<wbr>www<wbr>.blackhat<wbr>.com<wbr>/presentations<wbr>/bh<wbr>-usa<wbr>-07<wbr>/Yason<wbr>/Whitepaper<wbr>/bh<wbr>-usa<wbr>-07<wbr>-yason<wbr>-WP<wbr>.pdf*](https://www.blackhat.com/presentations/bh-usa-07/Yason/Whitepaper/bh-usa-07-yason-WP.pdf)。虽然现在有些过时，但这篇研究论文包含了大量关于解包恶意软件的信息。它还深入探讨了反分析和调试器攻击等话题。

+   Yehoshua, Nir 和 Uriel Kosayev. *绕过杀毒软件的技巧：学习实际技巧与战术来对抗、绕过和规避杀毒软件*。伯明翰，英国：Packt，2021 年。本书包含了关于绕过反恶意软件防御的专业知识。虽然主要面向“进攻性”研究人员，但书中的技巧可以帮助恶意软件分析师理解恶意软件如何绕过这些防御。

## 在线恶意软件沙箱

这是一些免费（或部分免费的）沙箱列表。你可以通过这些服务提交并分析恶意软件，但它们的免费层可能功能有限。还要注意，这些沙箱中的一些可能会将数据共享给未知方，样本也可能会向更广泛的观众开放。仅提交不包含敏感数据的文件，且请自行承担风险！

+   Any.Run: [*https://<wbr>any<wbr>.run*](https://any.run)

+   Cuckoo: [*https://<wbr>cuckoo<wbr>.cert<wbr>.ee*](https://cuckoo.cert.ee)

+   Hybrid Analysis: [*https://<wbr>hybrid<wbr>-analysis<wbr>.com*](https://hybrid-analysis.com)

+   Joe Sandbox: [*https://<wbr>joesandbox<wbr>.com*](https://joesandbox.com)

+   Triage: [*https://<wbr>tria<wbr>.ge*](https://tria.ge)

+   UnpacMe: [*https://<wbr>unpac<wbr>.me<wbr>/*](https://unpac.me/)

+   VirusTotal: [*https://<wbr>www<wbr>.virustotal<wbr>.com*](https://www.virustotal.com)

+   Yomi: [*https://<wbr>yomi<wbr>.yoroi<wbr>.company<wbr>/*](https://yomi.yoroi.company/)

## 恶意软件来源

如果你的日常工作不涉及研究新型恶意软件，你可能需要一些恶意软件样本来进行实践。以下来源是免费的（有些需要简单注册），但 VirusTotal 除外。

+   MalShare: [*https://<wbr>malshare<wbr>.com*](https://malshare.com)

+   MalwareBazaar: [*https://<wbr>bazaar<wbr>.abuse<wbr>.ch*](https://bazaar.abuse.ch)

+   Malware Traffic Analysis: [*https://<wbr>www<wbr>.malware<wbr>-traffic<wbr>-analysis<wbr>.net*](https://www.malware-traffic-analysis.net)

+   VirusShare: [*https://<wbr>virusshare<wbr>.com*](https://virusshare.com)

+   VirusSign: [*https://<wbr>www<wbr>.virussign<wbr>.com*](https://www.virussign.com)

+   VirusTotal: [*https://<wbr>www<wbr>.virustotal<wbr>.com*](https://www.virustotal.com)

+   vx-underground: [*https://<wbr>www<wbr>.vx<wbr>-underground<wbr>.org*](https://www.vx-underground.org)
