# 附录 进一步阅读

## 第一章

+   示例中使用的简化投射运动方程可以在 Wikipedia 上的[*https://<wbr>en<wbr>.wikipedia<wbr>.org<wbr>/wiki<wbr>/Projectile<wbr>_motion*](https://en.wikipedia.org/wiki/Projectile_motion)中找到详细解释。

+   Frances Buontempo 的*《程序员的遗传算法与机器学习》*（Pragmatic Bookshelf, 2019）专门有一章讨论弹道学。本章的初始示例来源于该书，并为教学目的做了适当的修改。

+   你可以在这里了解更多关于原始痴迷问题及其解决方法：

    +   “原始痴迷”wiki 页面，[*https://<wbr>wiki<wbr>.c2<wbr>.com<wbr>/<wbr>?PrimitiveObsession*](https://wiki.c2.com/?PrimitiveObsession)

    +   Refactoring Guru 网站，[*https://<wbr>refactoring<wbr>.guru<wbr>/smells<wbr>/primitive<wbr>-obsession*](https://refactoring.guru/smells/primitive-obsession)

    +   Fit wiki 页面，[*http://<wbr>fit<wbr>.c2<wbr>.com<wbr>/wiki<wbr>.cgi?w<WholeValue*](http://fit.c2.com/wiki.cgi?WholeValue)

    +   “信息完整性检查模式语言”，Ward Cunningham 著，[*http://<wbr>c2<wbr>.com<wbr>/ppr<wbr>/checks<wbr>.xhtml*](http://c2.com/ppr/checks.xhtml)

+   数量模式在 Martin Fowler 的*《分析模式：可重用的对象模型》*（Addison-Wesley Professional, 1996）中有详细描述。原始痴迷代码异味在他的书《重构：改善现有代码的设计》*（Addison-Wesley Professional, 2018）*中被指出。

+   在计算中混淆值的单位可能会产生严重后果；参见 Ajay Harish 的文章《NASA 由于度量数学错误丧失了一艘航天器》：[*https://<wbr>www<wbr>.simscale<wbr>.com<wbr>/blog<wbr>/2017<wbr>/12<wbr>/nasa<wbr>-mars<wbr>-climate<wbr>-orbiter<wbr>-metric<wbr>/*](https://www.simscale.com/blog/2017/12/nasa-mars-climate-orbiter-metric/)。

+   Kevlin Henney 的书籍*《程序员必知的 97 件事》*（O'Reilly, 2010）提供了许多关于这一主题以及其他多个主题的宝贵建议。领域概念在代码中的表示在第十一章由 Dan North 巧妙地阐述，在第六十五章由 Einar Landre 进行了更深入的探讨。

+   Henney 还在《Java 中的模式》一书中探讨了值对象、整体值、类工厂方法和其他值模式，这些模式同样适用于 C#：[*https://<wbr>www<wbr>.slideshare<wbr>.net<wbr>/Kevlin<wbr>/value<wbr>-added<wbr>-43542768*](https://www.slideshare.net/Kevlin/value-added-43542768)。还可以参考他在 2003 年 VikingPLoP（程序模式语言）大会上的论文《工厂和处置方法：互补且对称的模式对》，链接：[*https://<wbr>www<wbr>.researchgate<wbr>.net<wbr>/publication<wbr>/238075361*](https://www.researchgate.net/publication/238075361)。

+   长期以来，人们已认识到保持对象简洁有助于创建易于理解的程序。单一责任原则——即 SOLID 中的*S*——可能是最著名的指导原则：[*https://<wbr>en<wbr>.wikipedia<wbr>.org<wbr>/wiki<wbr>/SOLID*](https://en.wikipedia.org/wiki/SOLID)。

+   然而，关于分离职责的好处早在 1970 年代就已被认识到，甚至更早，当时埃兹格·迪克斯特拉就曾写过关于分离关注点的文章：

    +   “逻辑系统的有效排列，” [*https://<wbr>www<wbr>.cs<wbr>.utexas<wbr>.edu<wbr>/users<wbr>/EWD<wbr>/transcriptions<wbr>/EWD05xx<wbr>/EWD562<wbr>.xhtml*](https://www.cs.utexas.edu/users/EWD/transcriptions/EWD05xx/EWD562.xhtml)

    +   “科学思维的作用，” [*https://<wbr>www<wbr>.cs<wbr>.utexas<wbr>.edu<wbr>/users<wbr>/EWD<wbr>/transcriptions<wbr>/EWD04xx<wbr>/EWD447<wbr>.xhtml*](https://www.cs.utexas.edu/users/EWD/transcriptions/EWD04xx/EWD447.xhtml)

## 第二章

+   公共类型系统在以下微软文档中有总结：

    +   [*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/dotnet<wbr>/csharp<wbr>/programming<wbr>-guide<wbr>/types<wbr>/#the<wbr>-common<wbr>-type<wbr>-system*](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/types/#the-common-type-system)

    +   [*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/dotnet<wbr>/standard<wbr>/base<wbr>-types<wbr>/common<wbr>-type<wbr>-system*](https://docs.microsoft.com/en-us/dotnet/standard/base-types/common-type-system)

+   微软关于结构类型语言规则的文档可以在 [*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/dotnet<wbr>/csharp<wbr>/language<wbr>-reference<wbr>/language<wbr>-specification<wbr>/structs*](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/structs) 中找到。

+   关于 System.Threading.Monitor 在锁定对象方面的行为的更多信息，可以在 [*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/dotnet<wbr>/api<wbr>/system<wbr>.threading<wbr>.monitor<wbr>?view<wbr>=net<wbr>-6<wbr>.0#Lock*](https://docs.microsoft.com/en-us/dotnet/api/system.threading.monitor?view=net-6.0#Lock) 中找到。

+   关于可选参数的特定重载解析规则，请参见 [*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/dotnet<wbr>/csharp<wbr>/programming<wbr>-guide<wbr>/classes<wbr>-and<wbr>-structs<wbr>/named<wbr>-and<wbr>-optional<wbr>-arguments#overload<wbr>-resolution*](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/named-and-optional-arguments#overload-resolution)。

+   Eric Lippert 解释了 *为什么* 只读字段和构造函数的初始化器按它们的顺序执行，详情请见 [*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-gb<wbr>/archive<wbr>/blogs<wbr>/ericlippert<wbr>/why<wbr>-do<wbr>-initializers<wbr>-run<wbr>-in<wbr>-the<wbr>-opposite<wbr>-order<wbr>-as<wbr>-constructors<wbr>-part<wbr>-one*](https://docs.microsoft.com/en-gb/archive/blogs/ericlippert/why-do-initializers-run-in-the-opposite-order-as-constructors-part-one)。

+   Lippert 已经广泛撰写了关于 C# 中值类型的相关内容，包括以下内容：

    +   “关于值类型的真相，” [*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-gb<wbr>/archive<wbr>/blogs<wbr>/ericlippert<wbr>/the<wbr>-truth<wbr>-about<wbr>-value<wbr>-types*](https://docs.microsoft.com/en-gb/archive/blogs/ericlippert/the-truth-about-value-types)

    +   “栈是实现细节，第一部分，” [*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-gb<wbr>/archive<wbr>/blogs<wbr>/ericlippert<wbr>/the<wbr>-stack<wbr>-is<wbr>-an<wbr>-implementation<wbr>-detail<wbr>-part<wbr>-one*](https://docs.microsoft.com/en-gb/archive/blogs/ericlippert/the-stack-is-an-implementation-detail-part-one)

    +   “栈是实现细节，第二部分，” [*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-gb<wbr>/archive<wbr>/blogs<wbr>/ericlippert<wbr>/the<wbr>-stack<wbr>-is<wbr>-an<wbr>-implementation<wbr>-detail<wbr>-part<wbr>-two*](https://docs.microsoft.com/en-gb/archive/blogs/ericlippert/the-stack-is-an-implementation-detail-part-two)

+   可空引用类型的文档由 Microsoft 发布，详情请见 [*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/dotnet<wbr>/csharp<wbr>/nullable<wbr>-references*](https://docs.microsoft.com/en-us/dotnet/csharp/nullable-references)。

+   此外，Jon Skeet 在他的博客中分享了他关于可空引用类型的早期经验，访问 [*https://<wbr>codeblog<wbr>.jonskeet<wbr>.uk<wbr>/2018<wbr>/04<wbr>/21<wbr>/first<wbr>-steps<wbr>-with<wbr>-nullable<wbr>-reference<wbr>-types<wbr>/*](https://codeblog.jonskeet.uk/2018/04/21/first-steps-with-nullable-reference-types/)。

+   Tony Hoare 在 2009 年的 QCon 大会上为 null 引用问题做出了著名的道歉。摘要可在 [*https://<wbr>qconlondon<wbr>.com<wbr>/london<wbr>-2009<wbr>/qconlondon<wbr>.com<wbr>/london<wbr>-2009<wbr>/presentation<wbr>/Null%2bReferences<wbr>_%2bThe%2bBillion%2bDollar%2bMistake<wbr>.xhtml*](https://qconlondon.com/london-2009/qconlondon.com/london-2009/presentation/Null%2bReferences_%2bThe%2bBillion%2bDollar%2bMistake.xhtml) 查阅。

## 第三章

+   C#语言规范中描述了变量类别，网址为：[*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/dotnet<wbr>/csharp<wbr>/language<wbr>-reference<wbr>/language<wbr>-specification<wbr>/variables*](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/variables)。

+   关于确定性赋值的规则，也可以在 C#语言参考中找到，地址为：[*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/dotnet<wbr>/csharp<wbr>/language<wbr>-reference<wbr>/language<wbr>-specification<wbr>/variables#94<wbr>-definite<wbr>-assignment*](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/variables#94-definite-assignment)。

+   Jon Skeet 在他的博客中探讨了按引用和按值传递参数的方式：[*https://<wbr>jonskeet<wbr>.uk<wbr>/csharp<wbr>/parameters<wbr>.xhtml*](https://jonskeet.uk/csharp/parameters.xhtml)。

+   Jon Skeet 的《*C#深度剖析*》（Manning，2014 年）第五章和第十六章详细分析了闭包。

+   闭包在 C#中并不新鲜，但它们的行为在某些方面发生了变化；捕获循环变量就是一个例子。Eric Lippert 的博客中有一篇很好的文章，解释了捕获循环变量（在函数对象中）的*旧*行为（C# v5 之前的行为）的背后原因：[*https://<wbr>ericlippert<wbr>.com<wbr>/2009<wbr>/11<wbr>/12<wbr>/closing<wbr>-over<wbr>-the<wbr>-loop<wbr>-variable<wbr>-considered-harmful<wbr>-part<wbr>-one<wbr>/*](https://ericlippert.com/2009/11/12/closing-over-the-loop-variable-considered-harmful-part-one/)。

+   关于高效代码的几种 C#特性，包括只读结构体和in参数的概述，可以参考：[*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/dotnet<wbr>/csharp<wbr>/write<wbr>-safe<wbr>-efficient<wbr>-code*](https://docs.microsoft.com/en-us/dotnet/csharp/write-safe-efficient-code)。

+   C#编程指南在[*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/dotnet<wbr>/csharp<wbr>/programming<wbr>-guide<wbr>/classes<wbr>-and<wbr>-structs<wbr>/ref<wbr>-returns*](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/ref-returns)中描述了 ref 返回值和 ref 局部变量。

+   尽管 ref 返回值和 ref 局部变量直到 C# v7.0 才被引入，但这种思想早已有之，正如 Eric Lippert 在[*https://<wbr>ericlippert<wbr>.com<wbr>/2011<wbr>/06<wbr>/23<wbr>/ref<wbr>-returns<wbr>-and<wbr>-ref<wbr>-locals<wbr>/*](https://ericlippert.com/2011/06/23/ref-returns-and-ref-locals/)中所解释的那样。

+   Vladimir Sadov 在[*http://<wbr>mustoverride<wbr>.com<wbr>/safe<wbr>-to<wbr>-return<wbr>/*](http://mustoverride.com/safe-to-return/)中研究了 ref 局部变量是否可以安全返回的规则。

+   垃圾回收器的运作是一个复杂的话题，但一个很好的起点是微软文档中的[*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/dotnet<wbr>/standard<wbr>/garbage<wbr>-collection<wbr>/fundamentals*](https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/fundamentals)。

+   Andrew Hunter 也在他的博客中描述了垃圾回收：[ *https://<wbr>www<wbr>.red<wbr>-gate<wbr>.com<wbr>/simple<wbr>-talk<wbr>/development<wbr>/dotnet<wbr>-development<wbr>/understanding<wbr>-garbage<wbr>-collection<wbr>-in<wbr>-net<wbr>/*](https://www.red-gate.com/simple-talk/development/dotnet-development/understanding-garbage-collection-in-net/)。

+   微软关于 C# 7.0 及以后的值元组支持的文档在 [*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/dotnet<wbr>/csharp<wbr>/language<wbr>-reference<wbr>/builtin<wbr>-types<wbr>/value<wbr>-tuples*](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-tuples)。

## 第四章

+   Eric Lippert 曾广泛撰写关于值类型的内容，并在 [*https://<wbr>ericlippert<wbr>.com<wbr>/2008<wbr>/05<wbr>/14<wbr>/mutating<wbr>-readonly<wbr>-structs<wbr>/*](https://ericlippert.com/2008/05/14/mutating-readonly-structs/) 中讨论了修改返回值的情况。

+   Lippert 在这篇文章中探讨了值类型的构建和使用临时中间实例的情况：[ *https://<wbr>ericlippert<wbr>.com<wbr>/2010<wbr>/10<wbr>/11<wbr>/debunking<wbr>-another<wbr>-myth<wbr>-about<wbr>-value<wbr>-types<wbr>/*](https://ericlippert.com/2010/10/11/debunking-another-myth-about-value-types/)。

+   C# 语言规范中关于对象创建的内容可以在 [*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/dotnet<wbr>/csharp<wbr>/language<wbr>-reference<wbr>/language<wbr>-specification<wbr>/expressions#117152<wbr>-object<wbr>-creation<wbr>-expressions*](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#117152-object-creation-expressions) 上在线阅读。对象初始化器的内容可以在 [*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/dotnet<wbr>/csharp<wbr>/language<wbr>-reference<wbr>/language<wbr>-specification<wbr>/expressions#117153<wbr>-object<wbr>-initializers*](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#117153-object-initializers) 上找到。

+   微软关于类型转换的文档，包括用户定义的转换方法的链接，可以在 [*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/dotnet<wbr>/csharp<wbr>/programming<wbr>-guide<wbr>/types<wbr>/casting<wbr>-and<wbr>-type<wbr>-conversions#implicit<wbr>-conversions*](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/types/casting-and-type-conversions#implicit-conversions) 中查看。

+   Jon Skeet 在他的博客中研究了只读字段，地址是 [*https://<wbr>codeblog<wbr>.jonskeet<wbr>.uk<wbr>/2014<wbr>/07<wbr>/16<wbr>/micro<wbr>-optimization<wbr>-the<wbr>-surprising<wbr>-inefficiency<wbr>-of<wbr>-readonly<wbr>-fields<wbr>/*](https://codeblog.jonskeet.uk/2014/07/16/micro-optimization-the-surprising-inefficiency-of-readonly-fields/)。

+   微软关于 in 参数的文档（[*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/dotnet<wbr>/csharp<wbr>/language<wbr>-reference<wbr>/keywords<wbr>/in<wbr>-parameter<wbr>-modifier*](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/in-parameter-modifier)）以及 ref readonly 返回值和局部变量的文档（[*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/dotnet<wbr>/csharp<wbr>/language<wbr>-reference<wbr>/keywords<wbr>/ref#reference<wbr>-return<wbr>-values*](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/ref#reference-return-values)）提供了关于正确使用它们的注意事项和规则。

+   Sergey Tepliakov 的博客上也有一些关于 in 和 ref readonly 性能的启发性文章，链接如下：[*https://<wbr>devblogs<wbr>.microsoft<wbr>.com<wbr>/premier<wbr>-developer<wbr>/the<wbr>-in<wbr>-modifier<wbr>-and<wbr>-the<wbr>-readonly<wbr>-structs<wbr>-in<wbr>-c<wbr>/*](https://devblogs.microsoft.com/premier-developer/the-in-modifier-and-the-readonly-structs-in-c/) 和 [*https://<wbr>devblogs<wbr>.microsoft<wbr>.com<wbr>/premier<wbr>-developer<wbr>/performance<wbr>-traps<wbr>-of<wbr>-ref<wbr>-locals<wbr>-and<wbr>-ref<wbr>-returns<wbr>-in<wbr>-c<wbr>/*](https://devblogs.microsoft.com/premier-developer/performance-traps-of-ref-locals-and-ref-returns-in-c/)。

+   Donald Knuth 的名言出自他 1974 年的 ACM 图灵奖演讲。他更为著名的补充是：“过早的优化是万恶之源。”完整的文本可以在[*https://<wbr>dl<wbr>.acm<wbr>.org<wbr>/doi<wbr>/10<wbr>.1145<wbr>/1283920<wbr>.1283929*](https://dl.acm.org/doi/10.1145/1283920.1283929)找到。

## 第五章

+   Eric Lippert 关于 null 主题的博客文章非常有启发性：[*https://<wbr>ericlippert<wbr>.com<wbr>/2013<wbr>/07<wbr>/25<wbr>/what<wbr>-is<wbr>-the<wbr>-type<wbr>-of<wbr>-the<wbr>-null<wbr>-literal<wbr>/*](https://ericlippert.com/2013/07/25/what-is-the-type-of-the-null-literal/)。

+   字符串驻留由微软在[*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/dotnet<wbr>/api<wbr>/system<wbr>.string<wbr>.intern<wbr>?view<wbr>=net<wbr>-5<wbr>.0*](https://docs.microsoft.com/en-us/dotnet/api/system.string.intern?view=net-5.0)中进行了文档化。

+   有多篇文章讨论了浮点数的表示和可能出现的陷阱。弗朗西斯·博恩坦波（Frances Buontempo）在她的*Overload*文章《浮点数的趣味与滑稽》中提供了概述，并为那些想深入研究的人提供了例子和更多参考，网址为 [*https://<wbr>accu<wbr>.org<wbr>/journals<wbr>/overload<wbr>/17<wbr>/91<wbr>/buontempo<wbr>_1558*](https://accu.org/journals/overload/17/91/buontempo_1558)。

+   理查德·哈里斯（Richard Harris）对浮点数比较和算术有广泛的写作。这系列的*Overload*文章探讨了 IEEE-754 浮点数的常见替代方案：

    +   “你将不得不思考！”， [*https://<wbr>accu<wbr>.org<wbr>/journals<wbr>/overload<wbr>/18<wbr>/99<wbr>/harris<wbr>_1702*](https://accu.org/journals/overload/18/99/harris_1702)

    +   “为什么定点数不能治愈你的浮点数烦恼，” [*https://<wbr>accu<wbr>.org<wbr>/journals<wbr>/overload<wbr>/18<wbr>/100<wbr>/harris<wbr>_1717*](https://accu.org/journals/overload/18/100/harris_1717)

    +   “为什么有理数不能治愈你的浮点数烦恼，” [*https://<wbr>accu<wbr>.org<wbr>/journals<wbr>/overload<wbr>/19<wbr>/101<wbr>/harris<wbr>_1986*](https://accu.org/journals/overload/19/101/harris_1986)

    +   “为什么计算机代数不能治愈你的浮点数烦恼，” [*https://<wbr>accu<wbr>.org<wbr>/journals<wbr>/overload<wbr>/19<wbr>/102<wbr>/harris<wbr>_1979*](https://accu.org/journals/overload/19/102/harris_1979)

    +   “为什么区间算术不能治愈你的浮点数烦恼，” [*https://<wbr>accu<wbr>.org<wbr>/journals<wbr>/overload<wbr>/19<wbr>/103<wbr>/harris<wbr>_1974*](https://accu.org/journals/overload/19/103/harris_1974)

+   微软文档中有关于 C#浮点类型的比较，网址为 [*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/dotnet<wbr>/csharp<wbr>/language<wbr>-reference<wbr>/language<wbr>-specification<wbr>/types#floating<wbr>-point<wbr>-types*](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/types#floating-point-types)。

+   微软文档中提供了常量模式的概述，网址为 [*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/dotnet<wbr>/csharp<wbr>/language<wbr>-reference<wbr>/operators<wbr>/patterns#constant<wbr>-pattern*](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#constant-pattern)。

+   声明模式在微软文档中有描述，网址为 [*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/dotnet<wbr>/csharp<wbr>/language<wbr>-reference<wbr>/operators<wbr>/patterns#declaration<wbr>-and<wbr>-type<wbr>-patterns*](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#declaration-and-type-patterns)。

+   可空值类型操作符重载在 C# 语言规范中有所描述，其中简短提到 operator==，可参考 [*https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-value-types#lifted-operators*](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-value-types#lifted-operators)。

+   C# 语言规范中还有一部分内容讨论了可空引用类型，详见 [*https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-reference-types*](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-reference-types)。

+   Lippert 有一系列关于可空值类型的精彩博客，从这里开始：[*https://ericlippert.com/2012/12/20/nullable-micro-optimizations-part-one/*](https://ericlippert.com/2012/12/20/nullable-micro-optimizations-part-one/)。

+   Lippert 在 [*https://docs.microsoft.com/en-us/archive/blogs/ericlippert/what-exactly-does-lifted-mean*](https://docs.microsoft.com/en-us/archive/blogs/ericlippert/what-exactly-does-lifted-mean) 中探讨了提升操作符的概念。

+   *MSDN Magazine* 探讨了为何值元组不是不可变的，详见 [*https://docs.microsoft.com/en-us/archive/msdn-magazine/2018/june/csharp-tuple-trouble-why-csharp-tuples-get-to-break-the-guidelines*](https://docs.microsoft.com/en-us/archive/msdn-magazine/2018/june/csharp-tuple-trouble-why-csharp-tuples-get-to-break-the-guidelines)。

## 第六章

+   若要深入了解建模复杂系统中的值类型，请参见 Dirk Bäumer 等人著作的《面向对象系统中的值》[*Ubilab 技术报告*](https://riehle.org/computer-science/research/1998/ubilab-tr-1998-10-1.pdf)。

+   Kevlin Henney 讲解了对象类型的分类法，并提供了关于 C++ 和 C# 中对象比较的其他有价值的见解，见 [*https://www.slideshare.net/Kevlin/objects-of-value*](https://www.slideshare.net/Kevlin/objects-of-value)。

+   Martin Fowler 在 [*https://www.martinfowler.com/bliki/AnemicDomainModel.xhtml*](https://www.martinfowler.com/bliki/AnemicDomainModel.xhtml) 中描述了贫血领域模型。

+   Fowler 在 [*https://www.martinfowler.com/bliki/AliasingBug.xhtml*](https://www.martinfowler.com/bliki/AliasingBug.xhtml) 中描述了对象别名引发的错误。

+   别名（Aliasing）也不是一个新思想，正如你可以在 Eric S. Raymond 的《行话文件》中看到的：[*http://www.catb.org/jargon/html/A/aliasing-bug.xhtml*](http://www.catb.org/jargon/html/A/aliasing-bug.xhtml)。

+   Scott Stanchfield 关于使用没有值传递概念的语言的风险的文章虽然已经很老，且主要关注那个时代的 Java，但仍然充满启发性：[*http://www.javadude.com/articles/passbyvalue.htm*](http://www.javadude.com/articles/passbyvalue.htm)。

+   IComparable实现的契约在 Microsoft 文档中描述，参见：[*https://docs.microsoft.com/en-us/dotnet/api/system.icomparable-1.compareto?view=net-5.0#notes-to-implementers*](https://docs.microsoft.com/en-us/dotnet/api/system.icomparable-1.compareto?view=net-5.0#notes-to-implementers)。

+   Henney 在这篇会议论文中讨论了一些值类型的模式，包括对称性：[*https://www.researchgate.net/publication/244405850_The_Good_the_Bad_and_the_Koyaanisqatsi_Consideration_of_Some_Patterns_for_Value_Objects*](https://www.researchgate.net/publication/244405850_The_Good_the_Bad_and_the_Koyaanisqatsi_Consideration_of_Some_Patterns_for_Value_Objects)。

+   Scott Meyers 写了许多经典著作，C++程序员会立刻认出它们，但他有许多对任何语言的程序员都很有意义的见解。特别是《Effective C++》第三版（Addison-Wesley, 2005 年）和《More Effective C++》（Addison-Wesley, 1996 年）探讨了如何使接口既容易正确使用，又难以错误使用，以及将函数移出类的好处。

+   不可传递或非传递骰子是探索和挑战“小于”和内在排序思想的一种有趣方式；请参阅 Rosetta Code 网站：[*https://rosettacode.org/wiki/Non-transitive_dice*](https://rosettacode.org/wiki/Non-transitive_dice)。

## 第七章

+   关于 Microsoft 开发者网络（MSDN）关于重写值的Equals方法的建议，参见：[*https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/how-to-define-value-equality-for-a-type*](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/statements-expressions-operators/how-to-define-value-equality-for-a-type)。

+   这篇 2005 年关于 CLR 如何管理对象实例的文章，虽然现在显得有些过时，但仍然具有启发性，链接：[*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/archive<wbr>/msdn<wbr>-magazine<wbr>/2005<wbr>/may<wbr>/net<wbr>-framework<wbr>-internals<wbr>-how<wbr>-the<wbr>-clr<wbr>-creates<wbr>-runtime<wbr>-objects*](https://docs.microsoft.com/en-us/archive/msdn-magazine/2005/may/net-framework-internals-how-the-clr-creates-runtime-objects)。关于 Adam Sitnik 的更近期分析，请参见：[*https://<wbr>adamsitnik<wbr>.com<wbr>/Value<wbr>-Types<wbr>-vs<wbr>-Reference<wbr>-Types<wbr>/*](https://adamsitnik.com/Value-Types-vs-Reference-Types/)。

+   有关多态性种类的详细分析，请参阅 Luca Cardelli 和 Peter Wegner 在《*Computing Surveys*》中发表的《理解类型、数据抽象和多态性》（On Understanding Types, Data Abstraction, and Polymorphism）一文，链接：[*http://<wbr>lucacardelli<wbr>.name<wbr>/Papers<wbr>/OnUnderstanding<wbr>.A4<wbr>.pdf*](http://lucacardelli.name/Papers/OnUnderstanding.A4.pdf)。

+   关于子类型的正式定义，请参阅 Barbara H. Liskov 和 Jeannette M. Wing 在《*ACM Transactions on Programming Languages and Systems*》上发表的《子类型的行为概念》（A Behavioral Notion of Subtyping），链接：[*https://<wbr>dl<wbr>.acm<wbr>.org<wbr>/doi<wbr>/10<wbr>.1145<wbr>/197320<wbr>.197383*](https://dl.acm.org/doi/10.1145/197320.197383)。

+   Eric Lippert 在一系列文章中更为全面地讨论了 Liskov 替代原则，系列文章从这里开始：[*https://<wbr>ericlippert<wbr>.com<wbr>/2015<wbr>/04<wbr>/27<wbr>/wizards<wbr>-and<wbr>-warriors<wbr>-part<wbr>-one<wbr>/*](https://ericlippert.com/2015/04/27/wizards-and-warriors-part-one/)。

+   Lippert 认为，所有的相等性都可以简单地通过符合要求的<code>CompareTo</code>实现来推导，链接：[*https://<wbr>www<wbr>.informit<wbr>.com<wbr>/articles<wbr>/article<wbr>.aspx<wbr>?p<wbr>=2425867*](https://www.informit.com/articles/article.aspx?p=2425867)。

+   Kevlin Henney 关于字符串和值类型的这篇文章基于 C++的字符串，但许多观察对 C#同样适用，链接：[*https://<wbr>www<wbr>.slideshare<wbr>.net<wbr>/Kevlin<wbr>/highly<wbr>-strung*](https://www.slideshare.net/Kevlin/highly-strung)。

+   *seam*（缝隙）这个术语通常归功于 Michael Feathers，他在其著作《*与遗留代码有效合作*》（Working Effectively with Legacy Code，Pearson，2004）中进行了阐述。相关章节可以在线访问，链接：[*https://<wbr>www<wbr>.informit<wbr>.com<wbr>/articles<wbr>/article<wbr>.aspx<wbr>?p<wbr>=359417&seqNum<wbr>=2*](https://www.informit.com/articles/article.aspx?p=359417&seqNum=2)。

+   模拟对象（Mock objects）作为面向对象单元测试的一个重要特性，已经存在很长时间，Wikipedia 上有一个很好的概述，链接：[*https://<wbr>en<wbr>.wikipedia<wbr>.org<wbr>/wiki<wbr>/Mock<wbr>_object*](https://en.wikipedia.org/wiki/Mock_object)。

+   Gerard Meszaros 在他的书《*xUnit 测试模式*》（Addison-Wesley，2007 年）中，以及在网上的 [*http://<wbr>xunitpatterns<wbr>.com<wbr>/Test%20Double<wbr>.xhtml*](http://xunitpatterns.com/Test%20Double.xhtml) 中描述了测试双重体（test double）的更一般概念。

+   Henney 在 [*https://<wbr>kevlinhenney<wbr>.medium<wbr>.com<wbr>/simplicity<wbr>-before<wbr>-generality<wbr>-use<wbr>-before<wbr>-reuse<wbr>-722a8f967eb9*](https://kevlinhenney.medium.com/simplicity-before-generality-use-before-reuse-722a8f967eb9) 中对术语 *reuse* 提出了反对意见。

+   关于 C# v8.0 范围的总结，包括与通用的 index 运算符相关的规范，请参见微软文档 [*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/dotnet<wbr>/csharp<wbr>/language<wbr>-reference<wbr>/proposals<wbr>/csharp<wbr>-8<wbr>.0<wbr>/ranges*](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-8.0/ranges)。

+   相对亮度的计算大约来自国际电信联盟无线电通信部门（ITU-R）推荐的 Wikipedia 页面 [*https://<wbr>en<wbr>.wikipedia<wbr>.org<wbr>/wiki<wbr>/Luma<wbr>_(video)*](https://en.wikipedia.org/wiki/Luma_(video))，但本章仅用于演示隐式转换。

+   C# 有几种方式表示类型之间的转换。Lippert 对 is 和 as 的描述可参见 [*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/archive<wbr>/blogs<wbr>/ericlippert<wbr>/is<wbr>-is<wbr>-as<wbr>-or<wbr>-is<wbr>-as<wbr>-is*](https://docs.microsoft.com/en-us/archive/blogs/ericlippert/is-is-as-or-is-as-is) 和 [*https://<wbr>ericlippert<wbr>.com<wbr>/2013<wbr>/05<wbr>/30<wbr>/what<wbr>-the<wbr>-meaning<wbr>-of<wbr>-is<wbr>-is<wbr>/*](https://ericlippert.com/2013/05/30/what-the-meaning-of-is-is/)。

+   Henney 的这些文章主要面向 C++ 程序员，但其中的原则广泛适用于任何面向对象的编程语言，包括 C#： [*https://<wbr>www<wbr>.slideshare<wbr>.net<wbr>/Kevlin<wbr>/promoting<wbr>-polymorphism*](https://www.slideshare.net/Kevlin/promoting-polymorphism) 和 [*https://<wbr>www<wbr>.slideshare<wbr>.net<wbr>/Kevlin<wbr>/substitutability*](https://www.slideshare.net/Kevlin/substitutability)。

+   重载解析的规则可以在 C# 语言规范中找到，链接：[*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/dotnet<wbr>/csharp<wbr>/language<wbr>-reference<wbr>/language<wbr>-specification<wbr>/expressions#1164<wbr>-overload<wbr>-resolution*](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/language-specification/expressions#1164-overload-resolution)。

+   Jon Skeet 在他的博客中介绍了重载的内容，访问地址为 [*https://<wbr>csharpindepth<wbr>.com<wbr>/articles<wbr>/Overloading*](https://csharpindepth.com/articles/Overloading)。

+   Lippert 讨论了关于重载解析的一些有趣问题，相关内容可以在以下链接中找到：[*https://<wbr>ericlippert<wbr>.com<wbr>/2006<wbr>/04<wbr>/05<wbr>/odious<wbr>-ambiguous<wbr>-overloads<wbr>-part<wbr>-one<wbr>/*](https://ericlippert.com/2006/04/05/odious-ambiguous-overloads-part-one/) 和 [*https://<wbr>ericlippert<wbr>.com<wbr>/2006<wbr>/04<wbr>/06<wbr>/odious<wbr>-ambiguous<wbr>-overloads<wbr>-part<wbr>-two<wbr>/*](https://ericlippert.com/2006/04/06/odious-ambiguous-overloads-part-two/)。

+   有关 C# 9.0 中记录类型的高级描述，请参阅微软文档：[*https://<wbr>docs<wbr>.microsoft<wbr>.com<wbr>/en<wbr>-us<wbr>/dotnet<wbr>/csharp<wbr>/whats<wbr>-new<wbr>/csharp<wbr>-9#record<wbr>-types*](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-9#record-types)。

+   类型构建器是工厂模式的一种变体。详见 Erich Gamma 等人所著的 *Design Patterns: Elements of Reusable Object-Oriented Software*（Addison-Wesley, 1995）。

+   Henney 在他的论文《The Good, the Bad, and the Koyaanisqatsi: Consideration of Some Patterns for Value Objects》中描述了可变伴随对象及其他值对象模式，该论文发表于 2003 年 VikingPLoP（程序模式语言）会议，论文链接为 [*https://<wbr>www<wbr>.researchgate<wbr>.net<wbr>/publication<wbr>/244405850*](https://www.researchgate.net/publication/244405850)。

## 第八章

+   一个有意设计为慢速的算法是 bogosort，通常用作一种简单方法来故意保持 CPU 占用；维基百科有相关详细信息，链接为 [*https://<wbr>en<wbr>.wikipedia<wbr>.org<wbr>/wiki<wbr>/Bogosort*](https://en.wikipedia.org/wiki/Bogosort)。

+   一个用于基准测试性能的流行库是 BenchmarkDotNet，相关介绍请见 [*https://<wbr>benchmarkdotnet<wbr>.org*](https://benchmarkdotnet.org)。

+   Joe Duffy 关于性能与优化的博客已经有十多年历史，但仍然发人深省，且其原则至今仍具有相关性，链接为：[*http://<wbr>joeduffyblog<wbr>.com<wbr>/2010<wbr>/09<wbr>/06<wbr>/the<wbr>-premature<wbr>-optimization<wbr>-is<wbr>-evil<wbr>-myth<wbr>/*](http://joeduffyblog.com/2010/09/06/the-premature-optimization-is-evil-myth/)。

+   关于 ValueType 重写的 Equals 方法，微软文档可在 [*https://docs.microsoft.com/en-us/dotnet/api/system.valuetype.equals?view=net-6.0*](https://docs.microsoft.com/en-us/dotnet/api/system.valuetype.equals?view=net-6.0) 查阅。

+   Sergey Tepliakov 的博客提供了大量关于为何重写Equals方法如此重要的信息，并给出了一些关于ValueType.GetHashCode的良好建议和有趣的背景知识：[*https://<wbr>devblogs<wbr>.microsoft<wbr>.com<wbr>/premier<wbr>-developer<wbr>/performance<wbr>-implications<wbr>-of<wbr>-default<wbr>-struct<wbr>-equality<wbr>-in<wbr>-c<wbr>/*](https://devblogs.microsoft.com/premier-developer/performance-implications-of-default-struct-equality-in-c/)。

+   有关.NET 6 中默认GetHashCode方法的实现，请参见[*https://<wbr>github<wbr>.com<wbr>/dotnet<wbr>/runtime<wbr>/blob<wbr>/release<wbr>/6<wbr>.0<wbr>/src<wbr>/coreclr<wbr>/vm<wbr>/comutilnative<wbr>.cpp#L1878*](https://github.com/dotnet/runtime/blob/release/6.0/src/coreclr/vm/comutilnative.cpp#L1878)。

+   结构体的默认相等性在ValueType中定义，详见[*https://<wbr>github<wbr>.com<wbr>/dotnet<wbr>/runtime<wbr>/blob<wbr>/release<wbr>/6<wbr>.0<wbr>/src<wbr>/coreclr<wbr>/System<wbr>.Private<wbr>.CoreLib<wbr>/src<wbr>/System<wbr>/ValueType<wbr>.cs#L21*](https://github.com/dotnet/runtime/blob/release/6.0/src/coreclr/System.Private.CoreLib/src/System/ValueType.cs#L21)。

+   Niklaus Wirth 的 Project Oberon 文档可以在[*https://<wbr>people<wbr>.inf<wbr>.ethz<wbr>.ch<wbr>/wirth<wbr>/ProjectOberon<wbr>/PO<wbr>.System<wbr>.pdf*](https://people.inf.ethz.ch/wirth/ProjectOberon/PO.System.pdf)找到。
