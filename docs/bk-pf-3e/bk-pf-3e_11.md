## 附录 A. 资源

![资源](img/httpatomoreillycomsourcenostarchimages2127149.png.jpg)

虽然我希望能涵盖 PF 配置的所有细节，但在这些页面中，覆盖所有可能的配置细节证明是不可能的。我希望这里列出的资源能补充一些细节，或者提供略有不同的视角。它们中的一些甚至因为自身的趣味性而值得一读。

## 互联网上的一般网络与 BSD 资源

以下是本书中引用的常见网络资源。值得查看各个 BSD 项目的官方网站，以获取最新的信息。

+   对于 OpenBSD 用户来说，特别值得关注的是在线的 *OpenBSD Journal ([`undeadly.org/`](http://undeadly.org/))*. 它提供关于 OpenBSD 和相关问题的新闻和文章。

+   OpenBSD 的官网 *[`www.openbsd.org/`](http://www.openbsd.org/)* 是获取 OpenBSD 信息的主要参考。如果你在使用 OpenBSD，你会不时访问这个网站。

+   你可以在 *[`www.openbsd.org/papers/`](http://www.openbsd.org/papers/)* 找到 OpenBSD 开发者的演讲和论文合集。这个网站是了解 OpenBSD 持续发展的一个良好信息来源。

+   *OpenBSD 常见问题解答 ([`www.openbsd.org/faq/index.html`](http://www.openbsd.org/faq/index.html))* 更像是一本用户指南，而非传统的问答文档。在这里，你将找到大量背景信息以及如何设置和运行 OpenBSD 系统的逐步说明。

+   Henning Brauer 的演讲“更快的包—网络栈和 PF 的性能调优” *([`quigon.bsws.de/papers/2009/eurobsdcon-faster_packets/`](http://quigon.bsws.de/papers/2009/eurobsdcon-faster_packets/))* 是当前主要 PF 开发者概述了最近 OpenBSD 版本中为提高网络性能所做的工作，PF 是其中的核心组件。

+   *PF: OpenBSD 数据包过滤器 ([`www.openbsd.org/faq/pf/index.html`](http://www.openbsd.org/faq/pf/index.html))*, 也称为 *PF 用户指南* 或 *PF 常见问题解答*，是 OpenBSD 团队维护的官方 PF 文档。本指南会根据每个版本进行更新，是 PF 从业者极为宝贵的参考资源。

+   Bob Beck 的“pf. 它不仅仅是防火墙” *([`www.ualberta.ca/~beck/nycbug06/pf/`](http://www.ualberta.ca/~beck/nycbug06/pf/))* 是一场 NYCBUG 2006 的演讲，讲解了 PF 的冗余性和可靠性特性，并通过来自阿尔伯塔大学网络的实际案例进行了说明。

+   Daniel Hartmeier 的 PF 页面 *([`www.benzedrine.cx/pf.html`](http://www.benzedrine.cx/pf.html))* 是他整理的 PF 相关资料，并提供了指向网络上各种资源的链接。

+   Daniel Hartmeier 的《OpenBSD 有状态包过滤器（pf）的设计与性能》*([`www.benzedrine.cx/pf-paper.html`](http://www.benzedrine.cx/pf-paper.html))*是他在 Usenix 2002 年会议上发表的论文。它描述了 PF 的初步设计和实现。

+   Daniel Hartmeier 的三部分*undeadly.org* PF 系列包括“PF：防火墙规则集优化”*([`undeadly.org/cgi?action=article&sid=20060927091645`](http://undeadly.org/cgi?action=article&sid=20060927091645))*, “PF：测试你的防火墙”*([`undeadly.org/cgi?action=article&sid=20060928081238`](http://undeadly.org/cgi?action=article&sid=20060928081238))*, 和“PF：防火墙管理”*([`undeadly.org/cgi?action=article&sid=20060929080943`](http://undeadly.org/cgi?action=article&sid=20060929080943))*。这三篇文章详细讲解了各自的主题，同时又保持了较高的可读性。

+   RFC 1631，《IP 网络地址转换器（NAT）》，1994 年 5 月（*[`www.ietf.org/rfc/rfc1631.txt`](http://www.ietf.org/rfc/rfc1631.txt)*，由 K. Egevang 和 P. Francis 编写）是 NAT 规范的第一部分，它的生命力比作者们最初的预期要长久。虽然它仍然是理解 NAT 的重要资源，但它已经在很大程度上被更新后的 RFC 3022（*[`www.ietf.org/rfc/rfc3022.txt`](http://www.ietf.org/rfc/rfc3022.txt)*，由 P. Srisuresh 和 K. Egevang 编写），该文档于 2001 年 1 月发布，所取代。

+   RFC 1918，《私人互联网地址分配》，1996 年 2 月（*[`www.ietf.org/rfc/rfc1918.txt`](http://www.ietf.org/rfc/rfc1918.txt)*，由 Y. Rebhter、B. Moskowitz、D. Karrenberg、G.J. de Groot 和 E. Lear 编写）是 NAT 和私人地址空间难题的第二部分。该 RFC 描述了分配私有、不可路由地址空间的动机，并定义了地址范围。RFC 1918 被指定为当前最佳实践。

+   如果你正在寻找一本全面详细地介绍网络协议的书，且明确倾向于 TCP/IP 视角，那么 Charles M. Kozierok 的*《TCP/IP 指南》*（No Starch Press，2005 年 10 月），并且有在线更新的版本，地址为*[`www.tcpipguide.com/`](http://www.tcpipguide.com/)*，几乎没有其他书籍能与之匹敌。它有超过 1600 页，虽然不算是口袋书，但在你的桌面上或浏览器窗口中非常有用，可以帮助你澄清在其他书籍中解释不清的任何网络术语。

## 示例配置和相关思考

许多人慷慨地分享了他们的经验，并在网上提供了示例配置。以下是我最喜欢的一些。

+   Marcus Ranum 的《计算机安全中的六个最愚蠢的想法》*([`www.ranum.com/security/computer_security/editorials/dumb/index.html`](http://www.ranum.com/security/computer_security/editorials/dumb/index.html))*, 2005 年 9 月 1 日，是我长期以来的最爱。本文探讨了关于安全的一些常见误解及其对现实世界安全工作产生的不幸影响。

+   Randal L. Schwartz 的《使用 OpenBSD 的包过滤器监控网络流量》*([`www.stonehenge.com/merlyn/UnixReview/col51.html`](http://www.stonehenge.com/merlyn/UnixReview/col51.html))*展示了流量监控和使用标签进行计费的实际示例。尽管在这几年里 PF 和标签的一些细节发生了变化，但这篇文章依然可读，并且很好地呈现了几个重要概念。

+   瑞典用户组 Unix.se 的*Brandvägg med OpenBSD* ([`unix.se/Brandv%E4gg_med_OpenBSD`](http://unix.se/Brandv%E4gg_med_OpenBSD))及其示例配置，例如基本的 ALTQ 配置，在我初期使用时非常有用。该网站很好地提醒我们，像本地用户组这样的志愿者努力可以成为宝贵的信息来源。

+   *#pf* IRC 频道维基(*[`www.probsd.net/pf/`](http://www.probsd.net/pf/)*)是由*#pf* IRC 频道讨论参与者维护的文档、示例配置和其他 PF 信息的集合。这是一个非常值得关注的志愿者努力的例子。

+   来自意大利的 OpenBSD 爱好者 Daniele Mazzocchio 维护着 Kernel Panic 网站，该网站收录了关于各种 OpenBSD 主题的有用文章和教程类文档，网址为*[`www.kernel-panic.it/openbsd.html`](http://www.kernel-panic.it/openbsd.html)*（英文和意大利文）。对于从一个似乎致力于保持材料与最新稳定版 OpenBSD 同步更新的人那里获取关于各种有趣话题的新视角，访问该站点非常值得。

+   Kenjiro Cho 的《使用 ALTQ 管理流量》*([`www.usenix.org/publications/library/proceedings/usenix99/cho.html`](http://www.usenix.org/publications/library/proceedings/usenix99/cho.html))* 是描述 ALTQ 设计及其在 FreeBSD 上早期实现的原始论文。

+   Jason Dixon 的《使用 OpenBSD 和 CARP 的故障转移防火墙》，发表于 2005 年 5 月的*SysAdmin Magazine* ([`planet.admon.org/howto/failover-firewalls-with-openbsd-and-carp/`](http://planet.admon.org/howto/failover-firewalls-with-openbsd-and-carp/))，概述了 CARP 和 pfsync，并提供了一些实际示例。

+   Theo de Raadt 的 OpenCON 2006 演讲“硬件的开放文档：为什么硬件文档如此重要，以及为什么它如此难以获得” *([`openbsd.org/papers/opencon06-docs/index.html`](http://openbsd.org/papers/opencon06-docs/index.html))*，是附录 B 中关于自由操作系统硬件支持的一条重要灵感来源，特别是针对 OpenBSD。

## 其他 BSD 系统上的 PF

PF 已经从 OpenBSD 移植到其他 BSD 系统，尽管这些努力的目标自然是尽可能与 OpenBSD 最新版本的 PF 保持同步，但追踪其他 BSD 系统中的 PF 项目仍然是有用的。

+   FreeBSD 数据包过滤器（pf）主页 *([`pf4freebsd.love2party.net/`](http://pf4freebsd.love2party.net/))* 描述了在 FreeBSD 上与 PF 相关的早期工作及其项目目标。目前该页面并未完全更新最新进展，但一旦 Max Laier 注意到他在印刷版书籍中的引用，页面可能会再次活跃起来。

+   NetBSD 项目在 *[`www.netbsd.org/docs/network/pf.html`](http://www.netbsd.org/docs/network/pf.html)* 上维护着关于 PF 的页面，你可以在这里找到关于 NetBSD 上 PF 的最新信息。

## BSD 和网络书籍

除了似乎不断扩展的在线资源外，几本书籍也可以作为本书的伴随或补充读物。

+   Michael W. Lucas, *Absolute OpenBSD*, 第 2 版（No Starch Press，2013）。这本书提供了 OpenBSD 的全面指南，内容丰富且实践性强。

+   Michael W. Lucas, *Network Flow Analysis*（No Starch Press，2010）。这是少数几本使用免费的 NetFlow 工具进行网络分析和管理的书籍之一，本书展示了如何使用这些工具和方法，发现你网络中真实发生的事情。

+   Brandon Palmer 和 Jose Nazario, *Secure Architectures with OpenBSD*（Addison-Wesley，2004）。本书概述了 OpenBSD 的特性，并着重介绍了构建安全和可靠系统的方法。书中参考了当时最新版本的 OpenBSD 3.4。

+   Douglas R. Mauro 和 Kevin J. Schmidt, *Essential SNMP*, 第 2 版（O'Reilly Media，2005）。正如书名所示，这是一本关于 SNMP 的基础参考书。

+   Jeremy C. Reed（编辑），*The OpenBSD PF Packet Filter Book*（Reed Media Services，2006）。本书基于 *PF 用户指南*，扩展到涵盖 FreeBSD、NetBSD 和 DragonFly BSD 上的 PF，并包括一些关于与 PF 互操作的第三方工具的附加内容。

+   Christopher M. Buechler 和 Jim Pingle, *pfSense: The Definitive Guide*（Reed Media Services，2009）。这本约 515 页的书是一本全面的指南，介绍了基于 FreeBSD 和 PF 的防火墙设备分发版本。根据写作时的计划，修订版预计于 2014 年出版。

## 无线网络资源

Kjell Jørgen Hole 的 Wi-Fi 课程材料 *([`www.nowires.org/`](http://www.nowires.org/))* 是理解无线网络的一个极好的资源。该课程材料主要面向参加 Hole 教授课程的卑尔根大学学生，但它是免费的，值得一读。

## spamd 和灰名单相关资源

如果处理电子邮件是你生活的一部分（或者未来可能成为你生活的一部分），你可能已经喜欢了本书中对`spamd`、陷阱邮件和灰名单的描述。如果你想要比相关 RFC 中找到的更多背景信息，以下文档和网络资源将提供相关内容。

+   Greylisting.org (*[`www.greylisting.org/`](http://www.greylisting.org/)*）收集了有用的灰名单相关文章和关于灰名单及 SMTP 的一般信息。

+   Evan Harris 的《垃圾邮件控制战争中的下一步：灰名单》*([`greylisting.org/articles/whitepaper.shtml`](http://greylisting.org/articles/whitepaper.shtml))* 是最初的灰名单论文。

+   Bob Beck 的《OpenBSD spamd——灰名单及其他》*([`www.ualberta.ca/~beck/nycbug06/spamd/`](http://www.ualberta.ca/~beck/nycbug06/spamd/))* 是一场 NYCBUG 演讲，解释了`spamd`的工作原理，并描述了`spamd`在阿尔伯塔大学基础设施中的作用。（请注意，演讲中提到的许多“未来工作”已经实施。）

+   《有效的垃圾邮件和恶意软件对策》*([`bsdly.blogspot.com/2014/02/effective-spam-and-malware.html`](http://bsdly.blogspot.com/2014/02/effective-spam-and-malware.html))*, 最初是我在 BSDCan 2007 的论文并进行了更新，包含了如何使用灰名单、`spamd`以及各种其他免费工具和 OpenBSD 来成功打击你网络中的垃圾邮件和恶意软件的最佳实践描述。

+   一个有前景的新进展是 Peter Hessler 的*BGP-spamd*项目，该项目略微滥用 BGP 路由协议，将`spamd`数据分发到参与的主机之间。更多信息请访问该项目的官方网站 *[`bgp-spamd.net/`](http://bgp-spamd.net/)*。

## 与书籍相关的网络资源

有关本书的新闻和更新，请访问 No Starch Press 网站上的书籍主页 *([`www.nostarch.com/pf3/`](http://www.nostarch.com/pf3/))*。该页面包含指向我的个人网页空间的链接，在那里将会发布各种更新和书籍相关资源。关于本书的新闻和更新也会发布在 *[`www.bsdly.net/bookofpf/`](http://www.bsdly.net/bookofpf/)*。与本书相关的公告可能也会通过我的博客 *[`bsdly.blogspot.com/`](http://bsdly.blogspot.com/)* 出现。

我维护着教程手稿《使用 OpenBSD 的 PF 数据包过滤器进行防火墙设置》，它是本书的前身。我的政策是在合适的时候进行更新，通常是在我了解 PF 和相关软件的变化或新特性时，以及准备参加会议时。该教程手稿以 BSD 许可协议发布，可以从我的网页空间下载，格式有多种，地址是 *[`home.nuug.no/~peter/pf/`](http://home.nuug.no/~peter/pf/)*。更新版本会不定期出现在该网址，通常是在活动之间的自然调整过程中。

## 购买 OpenBSD 光盘并捐赠！

如果你喜欢这本书或觉得它有用，请访问 OpenBSD.org 订购页面 *[`www.openbsd.org/orders.html`](http://www.openbsd.org/orders.html)* 购买光盘套装，或者你也可以访问捐赠页面 *[`www.openbsd.org/donations.html`](http://www.openbsd.org/donations.html)*，通过金钱捐赠来支持 OpenBSD 项目的进一步开发。

如果你更倾向于向公司捐赠，你可以联系 OpenBSD 基金会，这是一家于 2007 年成立的加拿大非营利公司，专门用于此目的。有关更多信息，请访问 OpenBSD 基金会网站 *[`www.openbsdfoundation.org/`](http://www.openbsdfoundation.org/)*。

如果你在会议上找到了这本书，附近可能会有 OpenBSD 展位，你可以购买光盘、T 恤和其他物品。

请记住，即使是免费的软件，也需要真实的工作和资金来开发和维护。
