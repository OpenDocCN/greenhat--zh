## 附录：D

进一步阅读

本附录包含了二进制分析的参考资料和进一步阅读的建议。我将这些建议分为标准和参考资料、论文与文章以及书籍。尽管这份清单并不详尽无遗，但它应当为深入探索二进制分析世界提供了一个良好的起点。

### D.1 标准和参考资料

• *DWARF 调试信息格式版本 4*。可在 *[`www.dwarfstd.org/doc/DWARF4.pdf`](http://www.dwarfstd.org/doc/DWARF4.pdf)* 获取。

DWARF v4 调试格式规范。

• *可执行与可链接格式 (ELF)*。可在 *[`www.skyfree.org/linux/references/ELF_Format.pdf`](http://www.skyfree.org/linux/references/ELF_Format.pdf)* 获取。

ELF 二进制格式规范。

• *Intel 64 和 IA-32 架构软件开发手册*。可在 *[`software.intel.com/en-us/articles/intel-sdm`](https://software.intel.com/en-us/articles/intel-sdm)* 获取。

《Intel x86/x64 手册》。包含了整个指令集的详细描述。

• *PDB 文件格式*。可在 *[`llvm.org/docs/PDB/index.html`](https://llvm.org/docs/PDB/index.html)* 获取。

LLVM 项目的 PDB 调试格式非官方文档（基于微软在 *[`github.com/Microsoft/microsoft-pdb`](https://github.com/Microsoft/microsoft-pdb)* 发布的信息）。

• *PE 格式规范*。可在 *[`msdn.microsoft.com/en-us/library/windows/desktop/ms680547(v=vs.85).aspx`](https://msdn.microsoft.com/en-us/library/windows/desktop/ms680547(v=vs.85).aspx)* 获取。

关于 PE 格式的 MSDN 规范。

• *System V 应用程序二进制接口*。可在 *[`software.intel.com/sites/default/files/article/402129/mpx-linux64-abi.pdf`](https://software.intel.com/sites/default/files/article/402129/mpx-linux64-abi.pdf)* 获取。

x64 System V ABI 规范。

### D.2 论文与文章

• Baldoni, R., Coppa, E., D’Elia, D. C., Demetrescu, C., 和 Finocchi, I. (2017)。符号执行技术综述。可在 *[`arxiv.org/pdf/1610.00502.pdf`](https://arxiv.org/pdf/1610.00502.pdf)* 获取。

关于符号执行技术的综述论文。

• Barrett, C., Sebastiani, R., Seshia, S. A., 和 Tinelli, C. (2008)。模理论可满足性。在 *可满足性手册* 第十二章。IOS 出版社。可在 *[`people.eecs.berkeley.edu/~sseshia/pubdir/SMT-BookChapter.pdf`](https://people.eecs.berkeley.edu/~sseshia/pubdir/SMT-BookChapter.pdf)* 获取。

关于可满足性模理论（SMT）的书籍章节。

• Cha, S. K., Avgerinos, T., Rebert, A., 和 Brumley, D. (2012)。在二进制代码上释放混乱。在 *IEEE 安全与隐私研讨会论文集*，SP’12。可在 *[`users.ece.cmu.edu/~dbrumley/pdf/Cha%20et%20al._2012_Unleashing%20Mayhem%20on%20Binary%20Code.pdf`](https://users.ece.cmu.edu/~dbrumley/pdf/Cha%20et%20al._2012_Unleashing%20Mayhem%20on%20Binary%20Code.pdf)* 获取。

使用符号执行生成去除符号的二进制文件漏洞的自动利用代码。

• Dullien, T. 和 Porst, S. (2009). REIL: 一种用于静态代码分析的独立平台中间表示格式。载于 *《CanSecWest 会议论文集》*。可在 *[`www.researchgate.net/publication/228958277`](https://www.researchgate.net/publication/228958277)* 获得。

一篇关于 REIL 中间语言的论文。

• Kemerlis, V. P., Portokalidis, G., Jee, K., 和 Keromytis, A. D. (2012). libdft: 面向商用系统的实用动态数据流追踪。载于 *《虚拟执行环境会议论文集》*, VEE’12。可在 *[`nsl.cs.columbia.edu/papers/2012/libdft.vee12.pdf`](http://nsl.cs.columbia.edu/papers/2012/libdft.vee12.pdf)* 获得。

`libdft` 动态污点分析库的原始论文。

• Kolsek, M. (2017). 微软是否刚刚手动修补了他们的方程式编辑器可执行文件？是的，确实如此。（CVE-2017-11882）。可在 *[`blog.0patch.com/2017/11/did-microsoft-just-manually-patch-their.html`](https://blog.0patch.com/2017/11/did-microsoft-just-manually-patch-their.html)* 获得。

一篇描述微软如何修复软件漏洞，可能通过手写二进制补丁的文章。

• 链接时间优化（`gcc` 维基条目）。可在 *[`gcc.gnu.org/wiki/LinkTimeOptimization`](https://gcc.gnu.org/wiki/LinkTimeOptimization)* 获得。

一篇关于 `gcc` 维基中链接时间优化（LTO）的文章。包含指向其他相关 LTO 文章的链接。

• LLVM 链接时间优化：设计与实现。可在 *[`llvm.org/docs/LinkTimeOptimization.html`](https://llvm.org/docs/LinkTimeOptimization.html)* 获得。

一篇关于 LLVM 项目中的 LTO 的文章。

• Luk, C.-K., Cohn, R., Muth, R., Patil, H., Klauser, A., Lowney, G., Wallace, S., Reddi, V. J., 和 Hazelwood, K. (2005). Pin: 使用动态插桩构建定制化程序分析工具。载于 *《编程语言设计与实现会议论文集》*, PLDI’05。可在 *[`gram.eng.uci.edu/students/swallace/papers_wallace/pdf/PLDI-05-Pin.pdf`](http://gram.eng.uci.edu/students/swallace/papers_wallace/pdf/PLDI-05-Pin.pdf)* 获得。

Intel Pin 的原始论文。

• Pietrek, M. (1994). 深入了解 PE：Win32 可移植执行文件格式的探秘。可在 *[`msdn.microsoft.com/en-us/library/ms809762.aspx`](https://msdn.microsoft.com/en-us/library/ms809762.aspx)* 获得。

一篇关于 PE 格式复杂性的详细（尽管已过时）文章。

• Rolles, R. (2016). Synesthesia: 一种现代的 shellcode 生成方法。可在 *[`www.msreverseengineering.com/blog/2016/11/8/synesthesia-modern-shellcode-synthesis-ekoparty-2016-talk/`](http://www.msreverseengineering.com/blog/2016/11/8/synesthesia-modern-shellcode-synthesis-ekoparty-2016-talk/)* 获得。

一种基于符号执行的自动生成 shellcode 的方法。

• Schwartz, E. J., Avgerinos, T., 和 Brumley, D. (2010). 《你可能害怕问但却一直想知道的动态污点分析和前向符号执行》。收录于 *IEEE 安全与隐私研讨会论文集*，SP’10。可在 *[`users.ece.cmu.edu/~aavgerin/papers/Oakland10.pdf`](https://users.ece.cmu.edu/~aavgerin/papers/Oakland10.pdf)* 获取。

一篇关于动态污点分析和符号执行的实现细节及陷阱的深入论文。

• Slowinska, A., Stancescu, T., 和 Bos, H. (2011). Howard: 一个动态挖掘工具，用于逆向工程数据结构。收录于 *网络与分布式系统安全研讨会论文集*，NDSS’11。可在 *[`www.isoc.org/isoc/conferences/ndss/11/pdf/5_1.pdf`](https://www.isoc.org/isoc/conferences/ndss/11/pdf/5_1.pdf)* 获取。

一篇描述自动逆向工程数据结构的方法的论文。

• Yason, M. V. (2007). 解包艺术。收录于 *BlackHat USA*。可在 *[`www.blackhat.com/presentations/bh-usa-07/Yason/Whitepaper/bh-usa-07-yason-WP.pdf`](https://www.blackhat.com/presentations/bh-usa-07/Yason/Whitepaper/bh-usa-07-yason-WP.pdf)* 获取。

二进制解包技术介绍。

### D.3 书籍

• Collberg, C. 和 Nagra, J. (2009). *隐蔽软件：软件保护的混淆、水印和防篡改技术*。Addison-Wesley Professional。

一篇深入概述软件（去）混淆、水印和防篡改技术的文章。

• Eagle, C. (2011). *IDA Pro 手册：世界上最受欢迎的反汇编工具非官方指南（第 2 版）*。No Starch Press。

一本专门讲解使用 IDA Pro 反汇编二进制文件的完整书籍。

• Eilam, E. (2005). *逆向工程：逆向工程的秘密*。John Wiley & Sons, Inc.

手动逆向二进制文件的介绍（聚焦于 Windows）。

• Sikorski, M. 和 Honig, A. (2012). *实用恶意软件分析：恶意软件剖析实战指南*。No Starch Press。

一本关于恶意软件分析的全面介绍。
