# 附录 C. 资源

本附录是一个关于 GIMP 及其组件、用户和相关事项的网站列表，并附有评论。像任何此类列表一样，它在本书印刷时是最新的，但六个月后，一些网站可能已经消失，新的可能已经创建。接下来的所有网址中，最初的 *http://* 部分已被省略。

# C.1 官方 GIMP 页面

下面是官方 GIMP 网站的主要部分。

*[www.gimp.org](http://www.gimp.org)* 是官方的 GIMP 网站。如果你只知道一个网站，那必须是这个，尤其是因为它包含了指向许多其他重要网站的链接。在首页，你会找到有关最新发布的新闻以及下载页面的链接。网站的其他部分包含了本节提到的主要页面。

*[wiki.gimp.org](http://wiki.gimp.org)* 是 GIMP 开发者维基，一个关于 GIMP 的协作网站。它是为开发者设计的，但对任何想了解开发者项目的人都很有帮助。

*[bugs.gimp.org](http://bugs.gimp.org)* 是一个有些深奥的页面。它并没有解释如何报告错误，而是列出了不同版本的 GIMP 中当前的错误列表。这个页面让你查看开发者的工作，并了解谁在做什么。要报告错误，请查看 *[www.gimp.org/bugs/](http://www.gimp.org/bugs/)*。

*[developer.gimp.org](http://developer.gimp.org)* 是关于 GIMP 开发的页面。现在这个网站并没有什么更新，它的主要有趣内容是关于编写插件的教程，地址是 *[developer.gimp.org/writing-a-plug-in/1/](http://developer.gimp.org/writing-a-plug-in/1/)*。

*[gui.gimp.org](http://gui.gimp.org)* 是一个关于 GIMP 图形用户界面（GUI）的工作维基。这个网站解释了关于 GUI 的工作是如何进行的，以及它的进展方向。

*[docs.gimp.org](http://docs.gimp.org)*，令人惊讶的是，并不等同于 *[www.gimp.org/docs/](http://www.gimp.org/docs/)*。实际上，第一个链接指向的页面包含了先前版本的手册链接，并且现在的版本仍在进行中。第二个链接指向的是当前（完成的）文档，可以直接从 GIMP 本身访问。直到版本 2.8 的文档完成之前，它将指向版本 2.6 的文档。目前，这些文档可以在英语、荷兰语、法语、德语、意大利语、挪威语、韩语、俄语、西班牙语和瑞典语中找到。对于 2.8 版本，还将包括希腊语和日语。

*[registry.gimp.org](http://registry.gimp.org)* 是 GIMP 插件注册表，所有由 GIMP 用户构建的插件和脚本都可以在这里找到并下载。GIMP 插件注册表以博客形式呈现，最新的条目会出现在首页。你还可以在这里找到论坛，阅读和留下评论，最有用的部分是一个搜索引擎，用来寻找特定的插件。这个网站可能是继 *[www.gimp.org](http://www.gimp.org)* 之后，第二个最有用的网站。

# C.2 相关官方站点

在这里，我们提到了一些处理软件项目的网站或类似的网站，它们作为 GIMP 的框架或其构建的基础。

*[www.gnu.org](http://www.gnu.org)* 是 GNU 项目的官方网站。GNU 项目是世界上最古老的自由软件项目，实际上，这个项目创立了自由软件的概念。该项目由 Richard M. Stallman 于 1983 年发起，他希望拥有一个类 Unix 操作系统，且在使用、修改和分发方面没有任何限制。该项目开始时构建了所有必要的组件，除了内核。这个最后的组件后来由 Linux 内核提供。计划中的 GNU 内核，名为 Hurd，仍处于 alpha 阶段。记住，GIMP 是 GNU 图像处理程序（GNU Image Manipulation Program）的缩写。

*[www.gtk.org](http://www.gtk.org)* 是 GTK+ 项目的官方网站。这个软件最初是 GIMP 工具包，用于实现 GIMP 的小部件工具包。从开始至今，它已经发展成为 GNU/Linux 中最重要的工具包之一。例如，它是 GNOME 桌面环境的基础，并且现在是面向对象的（因此得名 GTK+），还被移植到了 Windows 和 Mac OS X 等操作系统。

*[www.gimp.org/about/COPYING](http://www.gimp.org/about/COPYING)* 是 GNU 通用公共许可证（GPL），这是自由软件领域最广泛使用的许可证。使用 GPL 授权的软件产品提供四项基本自由：自由地运行程序以满足任何目的，自由地研究程序如何工作并根据自己的需要进行修改，自由地重新分发副本以帮助他人，以及自由地改进程序并将改进成果发布给公众，使整个社区受益。重要的是，这些自由不会被重新分发程序的人所剥夺。

*[www.gegl.org](http://www.gegl.org)* 是通用图形库（Generic Graphic Library）的官方网站，这是一个基于图形的图像处理框架。在当前版本的 GIMP 中，图像被表示为像素数组。当你编辑图像时，你会改变像素，除非打开先前保存的副本，否则无法返回到未修改的图像：这就是所谓的*破坏性编辑*。使用 GEGL 时，图像被表示为图形，其中边是图像组件，节点是这些组件上的操作。通过改变图形的结构，你可以进行非破坏性编辑。此外，这种表示方式还支持表示高于当前 8 位深度的图像。GEGL 操作已经可以在 GIMP 2.8 中使用，但只有 GIMP 2.10 或 3.0 才能完全受益于 GEGL 的整合。

# C.3 教程

许多 GIMP 教程网站可供使用，但它们的质量参差不齐。而且，许多网站过于陈旧，无法提供有用的信息，因为它们讨论的 GIMP 版本早于 2.4。找到相关教程通常靠运气。以下是一些有趣且通常有用的网站。

*[www.gimp.org/tutorials/](http://www.gimp.org/tutorials/)* 提供了一系列有趣的教程，尽管大多数已经有几年历史，且没有最近的更新。

*[www.ghuj.com](http://www.ghuj.com)* 使用起来相当慢且复杂，但包含许多教程。

*[www.pixel2life.com/tutorials/gimp/](http://www.pixel2life.com/tutorials/gimp/)* 也较慢且复杂，但它被认为是网络上最大规模的 GIMP 教程集合。

*[www.gimpusers.com/tutorials/](http://www.gimpusers.com/tutorials/)* 提供了许多教程，按类别分类。

*gimps.de* 提供英语和德语版本，包含教程等内容。*[gimpology.com](http://gimpology.com)* 是一个大型教程网站，经常更新。

*[www.tutorialized.com/tutorials/Gimp/](http://www.tutorialized.com/tutorials/Gimp/)* 拥有数量可观且分类明确的教程，尤其是关于照片效果的内容。

*[gimpguru.org](http://gimpguru.org)* 包含一些虽然旧但很好的教程。

*[meetthegimp.org](http://meetthegimp.org)* 提供视频教程。

# C.4 社区与博客

尽管几位 GIMP 开发者曾经有博客，但目前只有一个仍在“活跃”，以每年九篇文章的更新速度进行：*[www.chromecode.com](http://www.chromecode.com)* 是马丁·诺德霍尔特（Martin Nordholts）的博客，他花在博客上的时间比 GIMP 少。这个博客展示了他为 GIMP 图形界面所做的一些最惊人的改变。

其他有趣的社区网站包括：

*[libregraphicsworld.org](http://libregraphicsworld.org)* 主要由其维护者 Alexandre Prokoudine 撰写，形式为博客。它是一个非常有趣的网站，涉及所有自由软件领域中的图形应用程序，包含公告、简短教程、评测等内容。无论如何，你应该把它加入书签！

*[www.gimpusers.com](http://www.gimpusers.com)* 是一个内容丰富的网站，提供新闻、教程和许多其他功能。然而，要小心它的论坛，因为大多数论坛内容只是对应邮件列表的转录（见后文）。例如，如果你想发送和阅读 GIMP 用户列表上的消息，最好直接使用邮件列表，避免使用这个网站上的伪论坛。

*[gimp-brainstorm.blogspot.com](http://gimp-brainstorm.blogspot.com)* 是一个相当特殊的网站。每个人都可以提出关于 GIMP 用户界面的想法，但贡献只能是图形类的。你必须保持沉默，并且必须保持匿名。这些想法可能会被 GIMP 用户界面重设计团队采用，或者完全被忽视。

*[www.gimptalk.com](http://www.gimptalk.com)* 是另一个内容丰富的网站，提供新闻、教程、论坛以及其他资源，如刷子、照片和插件。

*[www.gimpgallery.net](http://www.gimpgallery.net)* 的目的和前一个网站差不多，但它的使用略微复杂一些。

*[www.graphics-muse.org](http://www.graphics-muse.org)* 是 Michael J. Hammel 的博客，他是优秀书籍 *The Artist’s Guide to GIMP, 2nd Edition*（No Starch Press）的作者。

*[blog.mmiworks.net/](http://blog.mmiworks.net/)* 是 Peter Sikking 的博客，他是 m+mi works 的首席交互架构师，负责 GIMP 用户界面重设计。

*[www.ramonmiranda.com](http://www.ramonmiranda.com)* 是 Ramón Miranda 的博客。他是 GPS（Gimp Paint Studio）的创始人，并且是 GIMP 2.8 中新预设的贡献者。他的博客用英语和西班牙语书写。

*[groups.google.com/group/gimp-brushmakers-guild](http://groups.google.com/group/gimp-brushmakers-guild)* 是 GIMP Brushmakers Guild 的网站。尽管这个小组活动不多，但它正在尝试为 GIMP 实现一个非常重要的目标。

*[gimper.net](http://gimper.net)* 是另一个社区网站，提供新闻、帮助、论坛、资源等。

*[gimpmagazine.org](http://gimpmagazine.org)* 是一本全新的 GIMP 杂志，在发布后的前 24 小时内就被下载了十万次。第一期的表现非常有前景。

# C.5 刷子和插件

很多人通过各种方式为 GIMP 做出贡献。这里我们只包括贡献，而不包括偏离主题的内容。

## 刷子集

*[code.google.com/p/gps-gimp-paint-studio/](http://code.google.com/p/gps-gimp-paint-studio/)* 是 GIMP Paint Studio 的网站，这是一个巧妙构建的刷子集合，配合一组预定义的工具预设。详情请见 第十五章。

*[www.pgd-design.com/gimp/](http://www.pgd-design.com/gimp/)* 提供了一系列 GIMP 画笔和图案。

*[ljfhutch.blogspot.com.au](http://ljfhutch.blogspot.com.au)* 是 L.J.F. Hutch 的博客，他是一位专业的 2D 画家。该网站提供了大量的画笔资源。

## 插件集

以下网站包含重要的特定插件或插件集：

+   *ftp://ftp.gimp.org/pub/gimp/plug-ins/v2.6/gap/* 不是一个网站，而是一个用于 GAP 插件的 FTP 站点。有关 GAP 的更多信息，请查看 GIMP 文档或阅读本书的第六章和第十八章。

+   *[gmic.sourceforge.net](http://gmic.sourceforge.net)* 是 G'MIC 的官方网站。

+   *[liquidrescale.wikidot.com](http://liquidrescale.wikidot.com)* 是 Liquid Rescale GIMP 插件的官方网站。

+   *[sites.google.com/site/elsamuko/](http://sites.google.com/site/elsamuko/)* 是 Elsamuko 插件集的官方网站。

# C.6 邮件列表和 IRC 频道

GIMP 有五个官方邮件列表，由*[lists.xcf.berkeley.edu](http://lists.xcf.berkeley.edu)* 托管（不要直接访问此网站）。使用这些邮件列表，而不是它们在*[www.gimpusers.com](http://www.gimpusers.com)*上的对应论坛。如果你不熟悉邮件列表的礼仪，先看看*[www.gimp.org/mail_lists.html](http://www.gimp.org/mail_lists.html)*。

+   GIMP User (*[`mail.gnome.org/mailman/listinfo/gimp-user-list`](https://mail.gnome.org/mailman/listinfo/gimp-user-list)*) 是 GIMP 用户的一个活跃邮件列表。尽管官方站点这样说明，但它并非特别针对 Unix 系统。你可以提出任何问题，但请先查看文档和列表归档。如果你的问题表述清晰，通常会收到多个有用的回答。

+   GIMP Developer (*[`mail.gnome.org/mailman/listinfo/gimp-developer-list`](https://mail.gnome.org/mailman/listinfo/gimp-developer-list)*) 也是一个活跃的邮件列表。尽管这个名字是给开发者的，但它不仅限于开发者。它主要面向开发者，所以不要用这个列表提问简单的用户问题。可以用它来提出你认为对大家有兴趣的建议，或者准备一个错误报告。

+   GIMP Web (*[`mail.gnome.org/mailman/listinfo/gimp-web-list`](https://mail.gnome.org/mailman/listinfo/gimp-web-list)*) 处理 GIMP 网站的事务，最近有些不活跃。

+   GIMP Docs (*[`mail.gnome.org/mailman/listinfo/gimp-docs-list`](https://mail.gnome.org/mailman/listinfo/gimp-docs-list)*) 处理 GIMP 文档的相关事务。负责构建和翻译文档的人是主要用户，但你也可以用它来指出错误等问题。

有两个 IRC 频道，分别是#gimp 和#gimp-users。如果你不习惯使用 IRC，最好避免使用它们。这些频道有时完全不活跃，有时又非常活跃。

# C.7 其他图形应用程序

当然，在这里我们提到的仅是自由软件应用程序。

*[www.imagemagick.org](http://www.imagemagick.org)* 是 ImageMagick 网站。该应用程序在附录 F 中有所描述。

*[krita.org](http://krita.org)* 提供了 Krita，这是一个与 GIMP 目标相似的应用程序。它嵌入在基于 KDE 的 Koffice 套件中，但也可以在 GNOME 下使用。它相对于 GIMP 的主要优势在于支持 16 位色深和多个色彩空间。但在其他方面，它远不如 GIMP 发达，这一优势将在 GIMP 下一个版本发布时消失。

*[www.inkscape.org](http://www.inkscape.org)* 提供了 Inkscape，这是一个矢量图形编辑器。它的目标与 GIMP 的目标不同，后者是一个位图图形编辑器。这两个应用程序实际上可以被视为互补的。

*[www.blender.org](http://www.blender.org)* 是 Blender 的官方网站，Blender 是一款极其强大的 3D 图形应用程序，具有建模、仿真、动画等多种功能。使用 GIMP 构建的位图图像经常被加载到 Blender 中，作为新项目的一部分。

*[al.chemy.org](http://al.chemy.org)* 提供了 Alchemy，这是一个开放的绘图项目。Alchemy 是一个简单的应用程序，旨在探索新的素描和绘画方式。使用 Alchemy 构建的第一个草图可以加载到 GIMP 中进行进一步开发。

*[mypaint.intilinux.com](http://mypaint.intilinux.com)* 提供了 MyPaint，这是一个适用于 GNU/Linux 和 Windows 的绘图应用程序。它最显著的特点是能够定义许多新的画笔，这些画笔仅能与图形绘图板一起使用。

# C.8 相关的图形软件项目

在这里，我们再次提到的都是自由软件应用程序。

*[hugin.sourceforge.net](http://hugin.sourceforge.net)* 提供了 Hugin 全景照片拼接工具，它可以用来制作比 GIMP 能够制作的更大的全景图。

*[www.sane-project.org](http://www.sane-project.org)* 是 Scanner Access Now Easy (SANE) 项目的官方网站，该项目提供了对大多数扫描仪的标准化访问。

*[ufraw.sourceforge.net](http://ufraw.sourceforge.net)* 是 UFRaw 网站。该项目旨在读取和处理数字相机的 RAW 图像。我们推荐通过相应的 GIMP 插件使用它，但也可以作为独立的应用程序使用。

*[www.darktable.org](http://www.darktable.org)* 介绍了 Darktable，这是一款摄影工作流应用程序和 RAW 开发工具，具有比 UFRaw 更多的功能。它可以在 GNU/Linux 和 Mac OS X 下运行。

*[www.mplayerhq.hu](http://www.mplayerhq.hu)* 介绍了 MPlayer，这是一个通用的电影播放器，也可以作为批处理命令用于转换不同的动画格式。它的库被用于一些 GAP 命令中。

*[cinelerra.org](http://cinelerra.org)* 提供了 Cinelerra，这是一个视频编辑和合成应用程序。一个类似的应用程序是 PiTiVi (*[www.pitivi.org](http://www.pitivi.org)*)。

# C.9 其他图形网站

以下网站是有趣的计算机图形示例来源。

*[www.cgsociety.org](http://www.cgsociety.org)* 是计算机图形学会（CGSociety）的网站。即使你不是会员，也可以浏览画廊和作品集，全面了解当前计算机艺术家的作品。

*[art.gnome.org](http://art.gnome.org)* 具有不同的目的。它是一个图标、背景和其他艺术作品的画廊，用于更改并且希望能改善你的 GNOME 桌面的视觉外观。
