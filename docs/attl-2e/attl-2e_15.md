## 第十五章：FLAIM 第二部分：突破极限

*我们在大学里所做的，就是克服我们的小气思想。教育——要得到它，你必须呆在那儿，直到你能理解。*

—罗伯特·李·弗罗斯特*^(1)

![图片](img/common.jpg)

有一个被广泛理解的原则是，无论你读了多少书，参加了多少讲座，或者在邮件列表上提了多少问题，你仍然会有一些没有答案的问题。据估计，今天全球约有一半人口可以上网。^(2) 从你的桌面上可以获取成千上万的千兆字节信息。然而，似乎每个项目都有一两个问题，足够与其他问题有所不同，即使是互联网搜索也常常无济于事。

为了减少学习 Autotools 可能带来的挫败感，本章继续进行 FLAIM 构建系统转换项目，通过解决 FLAIM 构建系统要求中一些不常见的功能来继续进行。我希望通过展示一些不常见问题的解决方案，您将能熟悉 Autotools 提供的底层框架。这样的熟悉将为您提供洞察力，使您能够将 Autotools 灵活地应用于您的独特需求。

*xflaim* 库提供 Java 和 C# 语言绑定。Automake 提供了构建 Java 源代码的基础支持，但目前没有内置支持构建 C# 源代码。在本章中，我将向您展示如何使用 Automake 内置的 Java 支持来构建 xflaim 中的 Java 语言绑定，然后我将向您展示如何为 C# 语言绑定编写您自己的 `make` 规则。

本章将通过讨论使用本地编译器选项、构建生成的文档，以及添加您自己的顶层递归 `make` 目标，来完成本章内容，并结束 FLAIM 转换项目。

### 使用 Autotools 构建 Java 源代码

*GNU Automake 手册* 介绍了两种构建 Java 源代码的方法。第一种是传统的、被广泛理解的方法，即将 Java 源代码编译成 Java 字节码，然后在 Java 虚拟机（JVM）中执行。第二种方法是较少人知的，通过使用 GNU 编译器工具套件的 GNU Java 编译器前端（`gcj`）将 Java 源代码直接编译成本地机器码。包含这些机器码的目标文件可以通过标准的 GNU 链接器链接成本地可执行程序。由于缺乏兴趣，并且由于 JVM 在多年来得到了极大的改进，GCJ 项目已经不再维护。因此，很可能所有对这一机制的支持很快会完全从 Autotools 中删除。

在本章中，我将重点讲解前者——使用 Automake 内置的 `JAVA` 主体从 Java 源文件构建 Java 类文件。我们还将探讨构建和安装 *.jar* 文件所需的扩展。

#### *Autotools Java 支持*

Autoconf 对 Java 的内置支持几乎没有，甚至可以说是没有。例如，它没有提供任何宏来定位终端用户环境中的 Java 工具。^(3) Automake 对构建 Java 类的内置支持非常有限，但如果你愿意花些时间去深入了解，使其工作并不难。最大的问题更多是概念性的，而非功能性的。你需要付出一些努力，将你对 Java 构建过程的理解与 Automake 设计者的理解对齐。

Automake 提供了一个内置的主体（`JAVA`）来构建 Java 源文件，但它并没有提供任何预配置的安装位置前缀来安装 Java 类。然而，通常安装 Java 类和 *.jar* 文件的位置是 `$(datadir)`*/java* 目录，因此，创建一个正确的前缀和使用 Automake 的前缀扩展机制，定义一个以 *dir* 结尾的变量就可以了，具体可以参见 Listing 15-1。

```
--snip--
javadir = $(datadir)/java
java_JAVA = file_a.java file_b.java ...
--snip--
```

*Listing 15-1：在* Makefile.am *文件中定义 Java 安装目录*

现在，你通常不希望安装 Java 源文件，而是希望安装 *.class* 文件，或者更可能是包含所有 *.class* 文件的 *.jar* 文件。这通常更有用，因此，定义 `JAVA` 主体时使用 `noinst` 前缀会更合适。此外，`JAVA` 主体列表中的文件默认不会分发，所以你甚至可能想使用 `dist` 超前缀，如 Listing 15-2 所示。

```
dist_noinst_JAVA = file_a.java file_b.java...
```

*Listing 15-2：定义一个非安装的 Java 文件列表，这些文件会被分发*

当你在一个包含 `JAVA` 主体的变量中定义 Java 源文件列表时，Automake 会生成一个 `make` 规则，该规则在一个命令中构建该文件列表，使用 Listing 15-3 中显示的语法。^(4)

```
--snip--
JAVAROOT = .
JAVAC = javac
CLASSPATH_ENV = CLASSPATH=$(JAVAROOT):$(srcdir)/$(JAVAROOT):\
  $${CLASSPATH:+":$$CLASSPATH"}
--snip--
all: all-am
--snip--
all-am: Makefile classnoinst.stamp $(DATA) all-local
--snip--
classnoinst.stamp: $(am__java_sources)
        @list1='$?'; list2=; if test -n "$$list1"; then \
        for p in $$list1; do \
          if test -f $$p; then d=; else d="$(srcdir)/"; fi; \
          list2="$$list2 $$d$$p"; \
        done; \
 ➊ echo '$(CLASSPATH_ENV) $(JAVAC) -d $(JAVAROOT) \
          $(AM_JAVACFLAGS) $(JAVACFLAGS) '"$$list2"; \
        $(CLASSPATH_ENV) $(JAVAC) -d $(JAVAROOT) \
          $(AM_JAVACFLAGS) $(JAVACFLAGS) $$list2; \
        else :; fi
     ➋ echo timestamp > $@
--snip--
```

*Listing 15-3：这个长 shell 命令来自 Automake 生成的* Makefile *文件。*

在这些命令中，你看到的大部分代码仅仅是为了将 `$(srcdir)` 前缀加到用户指定的 Java 源文件列表中的每个文件，以便正确支持 `VPATH` 构建。该代码使用 shell 的 `for` 语句将列表拆分为单独的文件，添加 `$(srcdir)` 前缀，然后重新组装列表。^(5)

实际上完成构建 Java 源文件工作的部分出现在底部附近的两行（实际上是四行折叠的行）^(6) ➊。

Automake 在➋处生成一个印记文件，因为单个`$(JAVAC)`命令会从*.java*文件生成多个*.class*文件。Automake 不会随机选择其中一个文件，而是生成并使用印记文件作为规则的目标，这会导致`make`忽略单个*.class*文件与其对应的*.java*文件之间的关系。也就是说，如果删除一个*.class*文件，makefile 中的规则不会导致其被重新构建。唯一能导致重新执行`$(JAVAC)`命令的方法是修改一个或多个*.java*文件，从而使它们的时间戳变得比印记文件更新，或者完全删除印记文件。

在构建环境和命令行中使用的变量包括`JAVAROOT`、`JAVAC`、`JAVACFLAGS`、`AM_JAVACFLAGS`和`CLASSPATH_ENV`。每个变量可以在*Makefile.am*文件中指定。^(7) 如果未指定某个变量，则使用列表 15-3 中显示的默认值。

在`JAVA`主变量中指定的所有*.java*文件将通过单个命令行编译，这在命令行长度有限的系统上可能会导致问题。如果遇到此类问题，您可以将 Java 项目拆分为多个 Java 源目录，或者编写自己的`make`规则来构建 Java 类。（在我讨论如何在“构建 C#源代码”一节中构建 C#代码时，第 418 页展示了如何编写这些自定义规则。）

`CLASSPATH_ENV`变量设置 Java `CLASSPATH`环境变量，使其包含`$(JAVAROOT)`、`$(srcdir)/$(JAVAROOT)`，以及任何可能由最终用户在环境中配置的类路径。

`JAVAROOT`变量用于指定项目构建树中项目的 Java 根目录的位置，Java 编译器将会在该位置找到生成的包目录层级的起点。

`JAVAC`变量默认包含`javac`，假设`javac`可以在系统路径中找到。`AM_JAVACFLAGS`变量可以在*Makefile.am*中设置，尽管该变量的非 Automake 版本（`JAVACFLAGS`）被视为用户变量，因此不应在 makefile 中设置。

这在一定程度上是可行的，但远远不够。在这个相对简单的 Java 项目中，我们仍然需要使用`javah`工具生成 Java 本地接口（JNI）头文件，并从 Java 源代码构建的*.class*文件生成*.jar*文件。不幸的是，Automake 提供的 Java 支持甚至无法处理这些任务，因此我们必须通过手动编写`make`规则来完成剩余工作。我们将首先使用 Autoconf 宏来确保我们拥有一个良好的 Java 构建环境。

#### *使用 ac-archive 宏*

GNU Autoconf Archive 提供了社区贡献的 Autoconf 宏，接近我们需要的功能，以确保我们拥有一个良好的 Java 开发环境。在这种情况下，我下载了最新的源代码包，并将我需要的*.m4*文件手动安装到*xflaim/m4*目录中。^(8)

然后我修改了这些文件（包括它们的名称），使其像我的`FLM_PROG_TRY_DOXYGEN`宏那样工作。我想定位任何现有的 Java 工具，但如果需要，我也希望能够继续没有这些工具的工作。虽然在过去的 10 年里情况有了很大的改善，但考虑到围绕 Java 工具在 Linux 发行版中的存在所涉及的政治问题，这种做法可能是明智的。

我在相应的 Java 相关的*.m4*文件中创建了以下宏：

+   `FLM_PROG_TRY_JAVAC`定义在*flm_prog_try_javac.m4*中。

+   `FLM_PROG_TRY_JAVAH`定义在*flm_prog_try_javah.m4*中。

+   `FLM_PROG_TRY_JAVADOC`定义在*flm_prog_try_javadoc.m4*中。

+   `FLM_PROG_TRY_JAR`定义在*flm_prog_try_jar.m4*中。

+   `FLM_PROG_TRY_JNI`定义在*flm_prog_try_jni.m4*中。

通过稍微多一点的努力，我也能够创建 C#宏，以完成相同的任务，处理 C#语言绑定：

+   `FLM_PROG_TRY_CSC`定义在*flm_prog_try_csc.m4*中。

+   `FLM_PROG_TRY_CSVM`定义在*flm_prog_try_csvm.m4*中。

列表 15-4 展示了 xflaim *configure.ac*文件中的一部分，调用了这些 Java 和 C#宏。

```
--snip--
# Checks for optional programs.
FLM_PROG_TRY_CSC
FLM_PROG_TRY_CSVM
FLM_PROG_TRY_JNI
FLM_PROG_TRY_JAVADOC
--snip--
# Automake conditionals.
AM_CONDITIONAL([HAVE_JAVA], [test "x$flm_prog_have_jni" = xyes])
AM_CONDITIONAL([HAVE_CSHARP], [test -n "$CSC"])
--snip--
```

*列表 15-4:* xflaim/configure.ac: *该文件中的一部分，负责查找 Java 和 C#工具*

这些宏将`CSC`、`CSVM`、`JAVAC`、`JAVAH`、`JAVADOC`和`JAR`变量设置为它们各自的 C#和 Java 工具的位置，然后使用`AC_SUBST`将它们替换到 xflaim 项目的*Makefile.in*模板中。如果在执行`configure`脚本时用户的环境中已经设置了这些变量，它们的值将保持不变，从而允许用户覆盖宏原本会设置的值。

我在第十六章中讨论了这些宏的内部操作。

#### *标准系统信息*

使用来自 GNU Autoconf Archive 的宏时，你需要了解的唯一一个不太显眼的信息是，许多宏依赖于内置的 Autoconf 宏`AC_CANONICAL_HOST`。Autoconf 提供了一种方法，可以在宏定义之前自动扩展宏内部使用的任何宏，从而使所需的宏立即可用。然而，如果在某些宏（包括`LT_INIT`）之前没有使用`AC_CANONICAL_HOST`，`autoreconf`将生成大约十几个警告消息。

为了消除这些警告，我在我的 xflaim 级别的*configure.ac*文件中添加了`AC_CANONICAL_TARGET`，并将其置于`AC_INIT`调用之后。`AC_CANONICAL_SYSTEM`宏，以及它调用的宏（`AC_CANONICAL_BUILD`、`AC_CANONICAL_HOST`和`AC_CANONICAL_TARGET`），旨在确保`configure`定义了适当的值，描述用户的构建、主机和目标系统，分别保存在`$build`、`$host`和`$target`环境变量中。由于我在这个构建系统中没有进行交叉编译，因此我只需要调用`AC_CANONICAL_TARGET`。

这些变量包含构建、主机和目标 CPU、供应商及操作系统的规范值。类似这样的值对于扩展宏非常有用。如果宏可以假设这些变量已经正确设置，那么它可以节省在宏定义中重复的代码。

这些变量的值是通过辅助脚本`config.guess`和`config.sub`计算得出的，这些脚本与 Autoconf 一起分发。`config.guess`脚本通过一系列`uname`命令来探测构建系统的信息，然后使用这些信息推导出 CPU、供应商和操作系统的规范值。`config.sub`脚本用于将用户在`configure`命令行上指定的构建、主机和目标信息重新格式化为规范值。除非你通过`configure`的命令行选项覆盖它们，否则主机和目标值默认为构建的值。这样的覆盖可能用于交叉编译。（有关 Autotools 框架中交叉编译的更详细说明，请参见第 517 页的“第 6 项：交叉编译”）

#### *xflaim/java 目录结构*

原始的 xflaim 源代码布局将 Java JNI 和 C#本地源代码放置在*xflaim/src*之外的目录结构中。JNI 源代码位于*xflaim/java/jni*，C#本地源代码位于*xflaim/csharp/xflaim*。虽然 Automake 可以生成规则来访问当前目录层次结构之外的文件，但将这些文件放置得离它们唯一真正属于的库如此遥远似乎是没有必要的。因此，在这种情况下，我打破了自己不重新安排文件的规则，并将这两个目录的内容移到了*xflaim/src*下。我将 JNI 目录命名为*xflaim/src/java*，将 C#本地源代码目录命名为*xflaim/src/cs*。以下图表展示了这一新的目录层次结构：

```
flaim
  xflaim
    src
      cs
        wrapper
      java
        wrapper
          xflaim
```

如你所见，我还在*java*目录下添加了一个*wrapper*目录，并在其中根植了 xflaim 包装器包层次结构。由于 Java xflaim 包装器类是 Java xflaim 包的一部分，它们必须位于名为*xflaim*的目录中。然而，构建过程发生在 wrapper 目录中。在*wrapper/xflaim*目录或其下的任何目录中都没有找到构建文件。

**注意**

*无论你的包层次结构多深，你仍然将在*wrapper* *目录中构建 Java 类，该目录就是该项目的*`JAVAROOT`* 目录。Autotools Java 项目认为 *`JAVAROOT`* 目录是 Java 包的构建目录。*

#### *xflaim/src/Makefile.am 文件*

此时，*configure.ac* 文件尽其所能确保我拥有良好的 Java 构建环境，在这种情况下，我的构建系统将能够生成 JNI 包装类和头文件，并构建我的 C++ JNI 源文件。如果我的最终用户的系统未提供这些工具，他们将无法在该主机上构建或链接 JNI 语言绑定到 *xflaim* 库。

看一看 列表 15-5 中展示的 *xflaim/src/Makefile.am* 文件，检查与构建 Java 和 C# 语言绑定相关的部分。

```
if HAVE_JAVA
  JAVADIR = java
  JNI_LIBADD = java/libxfjni.la
endif

if HAVE_CSHARP
  CSDIR = cs
  CSI_LIBADD = cs/libxfcsi.la
endif

SUBDIRS = $(JAVADIR) $(CSDIR)
--snip--
libxflaim_la_LIBADD = $(JNI_LIBADD) $(CSI_LIBADD) $(FTK_LTLIB)
--snip--
```

*列表 15-5*：xflaim/src/Makefile.am：*这个 makefile 中构建 Java 和 C# 源文件的部分*

我已经解释了使用条件语句来确保只有在满足适当条件时，*java* 和 *cs* 目录才会被构建。你现在可以看到这如何融入我到目前为止所创建的构建系统中。

请注意，我在条件中定义了两个新的库变量。如果我能构建 Java 语言绑定，*java* 子目录将被构建，`JNI_LIBADD` 变量将引用在 *java* 目录中构建的库。如果我能构建 C# 语言绑定，*cs* 子目录将被构建，`CSI_LIBADD` 变量将引用在 *cs* 目录中构建的库。在这两种情况下，如果 `configure` 没有找到所需的工具，对应的变量将保持未定义。当引用一个未定义的 `make` 变量时，它展开为空，因此在 `libxflaim_la_LIBADD` 中使用它没有问题。

#### *构建 JNI C++ 源文件*

现在请注意 *xflaim/src/java/Makefile.am* 文件，见 列表 15-6。

```
SUBDIRS = wrapper

XFLAIM_INCLUDE = -I$(srcdir)/..

noinst_LTLIBRARIES = libxfjni.la

libxfjni_la_SOURCES = \
 jbackup.cpp \
 jdatavector.cpp \
 jdb.cpp \
 jdbsystem.cpp \
 jdomnode.cpp \
 jistream.cpp \
 jniftk.cpp \
 jniftk.h \
 jnirestore.cpp \
 jnirestore.h \
 jnistatus.cpp \
 jnistatus.h \
 jostream.cpp \
 jquery.cpp

libxfjni_la_CPPFLAGS = $(XFLAIM_INCLUDE) $(FTK_INCLUDE)
```

*列表 15-6*：xflaim/src/java/Makefile.am：*这个 makefile 构建 JNI 源文件。*

再次强调，我希望 *wrapper* 目录在 *xflaim* 库之前先被构建（`SUBDIRS` 列表末尾的点是隐含的），因为 *wrapper* 目录将构建 JNI 便利库源文件所需的类文件和 JNI 头文件。构建这个目录不是条件性的。如果我已经到达构建层次结构的这一部分，我知道我拥有所有需要的 Java 工具。这个 *Makefile.am* 文件仅构建一个包含我的 JNI C++ 接口函数的便利库。

由于 Libtool 以相同的源代码构建共享库和静态库，因此这个便利库将成为 *xflaim* 共享库和静态库的一部分。原始的构建系统 makefile 已通过仅将 JNI 和 C# 本地接口对象链接到共享库中（在那里它们是有意义的）来考虑这一点。

**注意**

*这些库被添加到共享和静态*xflaim*库中并不是一个真正的问题。只要这些对象中的函数和数据没有被引用，静态库中的对象在应用程序或链接到静态库的库中就不会被使用，尽管这在我新的构建系统中有些瑕疵。*

#### *Java 包装类和 JNI 头文件*

最后，*xflaim/src/java/wrapper/Makefile.am*带我们进入了问题的核心。我尝试过许多不同的配置来构建 Java JNI 包装器，而这个配置总是表现得最为出色。列表 15-7 展示了*包装*目录的 Automake 输入文件。

```
   JAVAROOT = .

➊ jarfile = $(PACKAGE)jni-$(VERSION).jar
➋ jardir = $(datadir)/java
   pkgpath = xflaim
   jhdrout = ..

   $(jarfile): $(dist_noinst_JAVA)
           $(JAR) cvf $(JARFLAGS) $@ $(pkgpath)/*.class

➌ jar_DATA = $(jarfile)

   java-headers.stamp: $(classdist_noinst.stamp)
           @list=`echo $(dist_noinst_JAVA) | sed -e 's|\.java||g' -e 's|/|.|g'`;\
             echo "$(JAVAH) -cp . -jni -d $(jhdrout) $(JAVAHFLAGS) $$list"; \
             $(JAVAH) -cp . -jni -d $(jhdrout) $(JAVAHFLAGS) $$list
        ➍ @echo "JNI headers generated" > java-headers.stamp

➎ all-local: java-headers.stamp

➏ CLEANFILES = $(jarfile) $(pkgpath)/*.class java-headers.stamp\
    $(jhdrout)/xflaim_*.h

   dist_noinst_JAVA = \
    $(pkgpath)/BackupClient.java \
    $(pkgpath)/Backup.java \
    --snip--
    $(pkgpath)/XFlaimException.java \
    $(pkgpath)/XPathAxis.java
```

*列表 15-7*：xflaim/src/java/wrapper/Makefile.am：*包装目录的*Makefile.am*文件*

在文件的顶部，我将`JAVAROOT`变量设置为点（`.`），因为我希望 Automake 能够告诉 Java 编译器这里是包层次结构的起始点。`JAVAROOT`的默认值是`$(top_builddir)`，这将错误地使包装类属于*xflaim.src.java.wrapper.xflaim*包。

我在➊创建了一个名为`jarfile`的变量，其值来自`$(PACKAGE``_TARNAME)`和`$(PACKAGE_VERSION)`。 （回想一下第三章中，`distdir`变量的值也是这样得出的，从中得到了 tarball 的名称。）一个`make`规则指明了*.jar*文件应该如何构建。这里，我使用的是`JAR`变量，其值由`configure`脚本中的`FLM_PROG_TRY_JNI`宏计算得出。

我在➋定义了一个新的安装变量`jardir`，用于指定*.jar*文件的安装位置，并在➌处将该变量用作`DATA`主项的前缀。Automake 认为符合 Automake *`where_HOW`*模式（带有定义的*`where`*`dir`）的文件，要么是架构独立的数据文件，要么是平台特定的可执行文件。以`bin`、`sbin`、`libexec`、`sysconf`、`localstate`、`lib`、`pkglib`开头，或包含“exec”字符串的安装位置变量（以`dir`结尾）被视为平台特定的可执行文件，并在执行`install-exec`目标时安装。Automake 认为安装在其他位置的文件是数据文件，并在执行`install-data`目标时进行安装。诸如*bindir*、*sbindir*等常见的安装位置已经被占用，但如果你想安装自定义的依赖架构的可执行文件，只需确保你的自定义安装位置变量包含“exec”字符串，如`myspecialexecdir`。

我在➍使用另一个时间戳文件，在规则中从*.class*文件生成 JNI 头文件，原因与 Automake 在规则中使用时间戳文件来从*.java*源文件生成*.class*文件相同。

这是这个 Makefile 中最复杂的部分，所以我将其拆分成更小的部分。

该规则声明，stamp 文件依赖于`dist_noinst_JAVA`变量中列出的源文件。该命令是一个复杂的 Shell 脚本，它会从文件列表中剥离*.java*扩展名，并将所有的斜杠字符转换为点号。这样做的原因是`javah`工具需要的是类名列表，而不是文件名列表。`$(JAVAH)`命令接受这个完整的列表作为输入，以生成相应的 JNI 头文件列表。当然，最后一行会生成 stamp 文件。

最后，在➎处，我将`java-headers.stamp`目标与`all`目标挂钩，通过将它作为依赖项添加到`all-local`目标中。当在此 makefile 中执行`all`目标（所有 Automake 生成的 makefile 的默认目标）时，*java-headers.stamp*将与 JNI 头文件一起构建。

**注意**

*最好将自定义规则目标作为依赖项添加到 Automake 提供的钩子和本地目标中，而不是直接将命令与这些钩子和本地目标关联。这样，单个任务的命令将保持独立，从而更易于维护。*

我将*.jar*文件、所有*.class*文件、*java-headers.stamp*文件以及所有生成的 JNI 头文件添加到`CLEANFILES`变量中（➏），以便在执行`make clean`时，Automake 会将它们清理掉。再次强调，我可以在这里使用`CLEANFILES`变量，因为我并不打算删除任何目录。

编写任何自定义代码的最后一步是确保`distcheck`目标仍然有效，因为当我们生成自己的产品时，必须确保`clean`目标能够正确地删除它们。

最后，我应该提到，构建*.jar*文件的规则（在列表 15-7 顶部附近）依赖于通配符来选取*xflaim*目录中的所有*.class*文件。Autotools 故意避免使用此类通配符，原因有很多，其中一个非常合理的原因是你可能会无意中选中那些由先前构建生成、但在更改后不再与项目相关的文件，因为这些源文件已从项目中删除。对于 Java，指定应放入*.jar*文件中的确切*.class*文件的唯一方法是解析所有*.java*文件，并生成一个由这些源文件构建出的*.class*文件的列表。我在这里做出了一个判断，决定使用通配符值得冒着可能会引起的问题。我还在列表 15-7 底部的`CLEANFILES`变量中使用了通配符。当然，这里也存在相同的潜在问题——你可能会删除一个文件，而这个文件目前存在，但已不再与构建相关。

#### *关于使用 JAVA 主目标的警告*

使用 `JAVA` 主变量时，有一个重要的警告，那就是每个 *Makefile.am* 文件中只能定义一个 `JAVA` 主变量。其原因在于，一个 *.java* 文件可能会生成多个类，而要知道哪些类是由哪个 *.java* 文件生成的，唯一的办法就是让 Automake 解析 *.java* 文件（这显然不现实，并且也是像 *Apache Ant* 和 *Maven* 这样的构建工具被开发出来的主要原因）。为了避免这样做，Automake 只允许每个文件定义一个 `JAVA` 主变量，因此所有在给定构建目录中生成的 *.class* 文件都会安装到由单个 `JAVA` 主变量前缀指定的位置。^(10)

**注意**

*我设计的系统在这种情况下能很好地工作，但好在我不需要安装我的 JNI 头文件，因为我无法从我的 Makefile.am 文件中知道它们叫什么！*

到现在为止，你应该能看到 Autotools 在处理 Java 时遇到的问题。事实上，这些问题更多是与 Java 语言本身的设计问题相关，而不是 Autotools 设计中的问题，正如你将在下一节中看到的。

### 构建 C# 源代码

回到 *xflaim/src/cs* 目录，我们将讨论如何为 Automake 不支持的语言构建源代码：C#。清单 15-8 显示了我为 *cs* 目录编写的 *Makefile.am* 文件。

```
SUBDIRS = wrapper

XFLAIM_INCLUDE = -I$(srcdir)/..

noinst_LTLIBRARIES = libxfcsi.la

libxfcsi_la_SOURCES = \
 Backup.cpp \
 DataVector.cpp \
 Db.cpp \
 DbInfo.cpp \
 DbSystem.cpp \
 DbSystemStats.cpp \
 DOMNode.cpp \
 IStream.cpp \
 OStream.cpp \
 Query.cpp

libxfcsi_la_CPPFLAGS = $(XFLAIM_INCLUDE) $(FTK_INCLUDE)
```

*清单 15-8*：xflaim/src/cs/Makefile.am：*cs 目录的 Automake 输入文件内容*

毫不奇怪，这看起来与 *xflaim/src/java* 目录中的 *Makefile.am* 文件几乎相同，因为我正在从该目录中的 C++ 源文件构建一个简单的便捷库，正如我在 *java* 目录中所做的那样。与 Java 版本一样，这个 makefile 首先会构建一个名为 *wrapper* 的子目录。

清单 15-9 显示了 *wrapper/Makefile.am* 文件的完整内容。

```
   EXTRA_DIST = xflaim cstest sample xflaim.ndoc

   xfcs_sources = \
    xflaim/BackupClient.cs \
    xflaim/Backup.cs \
    --snip--
    xflaim/RestoreClient.cs \
    xflaim/RestoreStatus.cs

   cstest_sources = \
    cstest/BackupDbTest.cs \
    cstest/CacheTests.cs \
    --snip--
    cstest/StreamTests.cs \
    cstest/VectorTests.cs

   TESTS = cstest_script

   AM_CSCFLAGS = -d:mono -nologo -warn:4 -warnaserror+ -optimize+
   #AM_CSCFLAGS += -debug+ -debug:full -define:FLM_DEBUG

➊ all-local: xflaim_csharp.dll

   clean-local:
           rm -f xflaim_csharp.dll xflaim_csharp.xml cstest_script\
             cstest.exe libxflaim.so
           rm -f Output_Stream
           rm -rf abc backup test.*

   install-exec-local:
           test -z "$(libdir)" || $(MKDIR_P) "$(DESTDIR)$(libdir)"
           $(INSTALL_PROGRAM) xflaim_csharp.dll "$(DESTDIR)$(libdir)"

   install-data-local:
           test -z "$(docdir)" || $(MKDIR_P) "$(DESTDIR)$(docdir)"
           $(INSTALL_DATA) xflaim_csharp.xml "$(DESTDIR)$(docdir)"

   uninstall-local:
           rm -f "$(DESTDIR)$(libdir)/xflaim_csharp.dll"
           rm -f "$(DESTDIR)$(docdir)/xflaim_csharp.xml"

➋ xflaim_csharp.dll: $(xfcs_sources)
           @list1='$(xfcs_sources)'; list2=; if test -n "$$list1"; then \
             for p in $$list1; do \
               if test -f $$p; then d=; else d="$(srcdir)/"; fi; \
               list2="$$list2 $$d$$p"; \
             done; \
             echo '$(CSC) -target:library $(AM_CSCFLAGS) $(CSCFLAGS) -out:$@\
               -doc:$(@:.dll=.xml) '"$$list2";\
             $(CSC) -target:library $(AM_CSCFLAGS) $(CSCFLAGS) \
               -out:$@ -doc:$(@:.dll=.xml) $$list2; \
           else :; fi

   check_SCRIPTS = cstest.exe cstest_script

➌ cstest.exe: xflaim_csharp.dll $(cstest_sources)
          @list1='$(cstest_sources)'; list2=; if test -n "$$list1"; then \
             for p in $$list1; do \
               if test -f $$p; then d=; else d="$(srcdir)/"; fi; \
               list2="$$list2 $$d$$p"; \
             done; \
             echo '$(CSC) $(AM_CSCFLAGS) $(CSCFLAGS) -out:$@ '"$$list2"'\
               -reference:xflaim_csharp.dll'; \
             $(CSC) $(AM_CSCFLAGS) $(CSCFLAGS) -out:$@ $$list2 \
               -reference:xflaim_csharp.dll; \
          else :; fi

➍ cstest_script: cstest.exe
           echo "#!/bin/sh" > cstest_script
           echo "$(top_builddir)/libtool --mode=execute \
           ➎ -dlopen=../../libxflaim.la $(CSVM) cstest.exe" >> cstest_script
           chmod 0755 cstest_script
```

*清单 15-9*：xflaim/src/cs/wrapper/Makefile.am：*C# makefile 的完整内容*

*Makefile.am* 的默认目标是 `all`，与普通的非-Automake makefile 相同。同样，我通过实现 `all-local` 目标将我的代码挂钩到 `all` 目标，该目标依赖于名为 *xflaim_csharp.dll* 的文件。^(11)

C# 源文件通过 ➋ 处的 *xflaim_csharp.dll* 目标下的命令进行构建，而 *xflaim_csharp.dll* 二进制文件依赖于 `xfcs_sources` 变量中指定的 C# 源文件列表。此规则中的命令是从 Automake 生成的 *java/wrapper/Makefile* 中复制的，并经过稍微修改，以便从 C# 源文件构建 C# 二进制文件（如清单中所示）。这里的重点不是讲解如何构建 C# 源文件；重点在于通过在 ➊ 处创建 `all-local` 目标与您自己目标之间的依赖关系，默认目标会被自动构建。

这个*Makefile.am*文件还构建了一套用于评估 C#语言绑定的单元测试。该规则的目标是*cstest.exe*（➌），最终成为一个 C#可执行文件。规则说明，*cstest.exe*依赖于*xflaim_csharp.dll*和源文件。我再次复制了构建*xflaim_csharp.dll*的规则中的命令（如高亮显示），并对其进行了修改以构建 C#程序。

最终，在构建`check`目标时，Automake 生成的 makefile 将尝试执行`TESTS`变量中列出的脚本或可执行文件。这里的目的是确保在执行这些文件之前，所有必要的组件都已经构建完成。我通过定义`check-local`并使其依赖于我的测试代码目标，来将其与`check`目标关联起来。

➎处的`cstest_script`是一个仅用于在 C#虚拟机中执行* cstest.exe*二进制文件的 Shell 脚本。C#虚拟机位于`CSVM`变量中，该变量由`FLM_PROG_TRY_CSVM`宏生成的`configure`代码定义。

`cstest_script`仅依赖于`cstest.exe`程序。然而，*xflaim*库要么必须存在于当前目录中，要么必须在系统库搜索路径中。在这里，我们通过使用 Libtool 的*execute*模式，在执行 C#虚拟机之前将*xflaim*库添加到系统库搜索路径中，从而实现最大的可移植性。

#### *手动安装*

由于在这个示例中我自己完成所有工作，因此我必须编写自己的安装规则。清单 15-10 仅复制了清单 15-9 中的*Makefile.am*文件中的安装规则。

```
--snip--
install-exec-local:
        test -z "$(libdir)" || $(MKDIR_P) "$(DESTDIR)$(libdir)"
        $(INSTALL_PROGRAM) xflaim_csharp.dll "$(DESTDIR)$(libdir)"

install-data-local:
        test -z "$(docdir)" || $(MKDIR_P) "$(DESTDIR)$(docdir)"
        $(INSTALL_DATA) xflaim_csharp.xml "$(DESTDIR)$(docdir)"

uninstall-local:
        rm -f "$(DESTDIR)$(libdir)/xflaim_csharp.dll"
        rm -f "$(DESTDIR)$(docdir)/xflaim_csharp.xml"
--snip--
```

*清单 15-10*：xflaim/src/cs/wrapper/Makefile.am：*该 makefile 的安装规则*

根据*GNU 编码标准*中定义的规则，安装目标不依赖于它们所安装的二进制文件，因此，如果二进制文件尚未构建，我可能需要退出*root*账户，切换到我的用户账户并先使用`make all`构建二进制文件。

Automake 区分安装程序和安装数据。然而，只有一个`uninstall`目标。其基本原理似乎是，你可能希望在网络中的每台系统上执行`install-exec`操作，但只需进行一次共享的`install-data`操作。卸载产品时不需要这种区分，因为卸载数据多次通常是无害的。

#### *再次清理*

和往常一样，必须正确清理文件，以便使分发检查通过。`clean-local`目标很好地处理了这一点，如清单 15-11 所示。

```
--snip--
clean-local:
        rm -f xflaim_csharp.dll xflaim_csharp.xml cstest_script \
          cstest.exe libxflaim.so
        rm -f Output_Stream
        rm -rf abc backup test.*
--snip--
```

*清单 15-11*：xflaim/src/cs/wrapper/Makefile.am：*该 makefile 中定义的清理规则*

### 配置编译器选项

原始的 GNU Make 构建系统提供了许多命令行构建选项。通过在`make`命令行中指定一系列辅助目标，用户可以指示他们想要调试或发布构建、在 64 位系统上强制进行 32 位构建、在 Solaris 系统上生成通用的 SPARC 代码等等。这是一种即插即用的构建系统方法，在商业代码中非常常见。

在开源项目中，尤其是在基于 Autotools 的构建系统中，更常见的做法是省略这些固定的框架，允许用户在标准用户变量中设置自己的选项：`CC`、`CPP`、`CXX`、`CFLAGS`、`CXXFLAGS`、`CPPFLAGS`等。^(12)

可能对于 Autotools 方法管理选项最有力的论点是，它是政策驱动的，商业软件供应商使用的固定框架可以很容易地通过更加灵活的政策驱动的 Autotools 框架来实现。例如，*config.site*文件可以用于为特定站点上进行的所有基于 Autotools 的构建提供站点范围的选项。在调用`configure`之前，可以使用一个简单的脚本来配置各种基于环境的选项，或者这些选项甚至可以在这样的脚本中直接传递给`configure`或`make`。Autotools 的政策驱动方法提供了灵活性，可以根据开发人员的需求进行配置，或者根据管理层的要求进行严格限制。

最终，我们希望 FLAIM 项目选项符合 Autotools 政策驱动的方法；然而，我不想失去确定原始 makefile 中硬编码本地编译器选项所涉及的研究工作。为此，我已经将*部分*选项添加回了*configure.ac*文件，这些选项是原始构建系统所支持的，但我也有一些选项没有添加。 清单 15-12 显示了这些努力的最终结果。此代码根据一些用户变量的内容，按需启用各种本地编译器选项、优化和调试功能。

```
   --snip--
   # Configure supported platforms' compiler and linker flags
➊ case $host in
     sparc-*-solaris*)
       LDFLAGS="$LDFLAGS -R /usr/lib/lwp"
       case $CXX in
         *g++*) ;;
         *)
           if "x$debug" = xno; then
             CXXFLAGS="$CXXFLAGS -xO3"
           fi
           SUN_STUDIO=`$CXX -V | grep "Sun C++"`
           if "x$SUN_STUDIO" = "xSun C++"; then
             CXXFLAGS="$CXXFLAGS -errwarn=%all -errtags\
               -erroff=hidef,inllargeuse,doubunder"
           fi ;;
     esac ;;

   *-apple-darwin*)
     AC_DEFINE([OSX], [1], [Define if building on Apple OSX.]) ;;

   *-*-aix*)
     case $CXX in
       *g++*) ;;
       *) CXXFLAGS="$CXXFLAGS -qstrict" ;;
     esac ;;

   *-*-hpux*)
     case $CXX in
       *g++*) ;;
       *)
         # Disable "Placement operator delete
 # invocation is not yet implemented" warning
         CXXFLAGS="$CXXFLAGS +W930" ;;
     esac ;;
  esac
  --snip--
```

*清单 15-12*：xflaim/configure.ac：*此文件中启用特定编译器选项的部分*

请记住，这段代码依赖于之前使用的`AC_CANONICAL_SYSTEM`（或`AC_CANONICAL_TARGET`）宏，该宏将`build`、`host`和`target`环境变量设置为规范的字符串值，以指示 CPU、供应商和操作系统。

在 清单 15-12 中，我在 ➊ 的 `case` 语句中使用了 `host` 变量来确定我为其构建的系统类型。这个 `case` 语句通过查找 `host` 中的子字符串来判断用户是否在 Solaris、Apple Darwin、AIX 或 HP-UX 上构建，这些子字符串在这些平台的所有变体中都是共同的。`config.guess` 和 `config.sub` 文件是你在这里的好帮手。如果你需要为你的项目编写类似的代码，可以检查这些文件，找出你希望为其设置各种编译器和链接器选项的进程和系统的共同特征。

**注意**

*在这些情况下（除了在 Apple Darwin 系统上定义 *`OSX`* 预处理器变量的情况），我实际上只是为本地编译器设置标志。GNU 编译器工具似乎能够处理任何代码，而无需额外的编译器选项。在这里值得再次强调的是，Autotools 的特性-存在方法设置选项再次胜出。当你不必为一个不断增长的支持主机和工具集列表支持大量的 *`case`* 语句时，维护工作会大大减少。*

### 将 Doxygen 集成到构建过程中

我希望在构建过程中生成文档，如果可能的话。也就是说，如果用户已经安装了 `doxygen`，构建系统将使用它作为 `make all` 过程的一部分来生成 Doxygen 文档。

原始的构建系统同时有静态文档和生成的文档。静态文档应始终安装，但只有在主机上可用 `doxygen` 程序时，Doxygen 文档才可以构建。因此，我始终构建 *docs* 目录，但我使用 `AM_CONDITIONAL` 宏来有条件地构建 *docs/doxygen* 目录。

Doxygen 使用配置文件（通常称为 *doxyfile*）来配置数百个 Doxygen 选项。这个配置文件包含一些配置脚本已知的信息。这听起来像是一个使用 Autoconf 生成的文件的绝佳机会。为此，我编写了一个名为 *doxyfile.in* 的 Autoconf 模板文件，包含了一个正常的 Doxygen 输入文件会包含的大部分内容，并且有一些 Autoconf 替换变量引用。此文件中的相关行如 清单 15-13 所示。

```
--snip--
PROJECT_NAME                = @PACKAGE_NAME@
--snip--
PROJECT_NUMBER              = @PACKAGE_VERSION@
--snip--
STRIP_FROM_PATH             = @top_srcdir@
--snip--
INPUT                       = @top_srcdir@/src/xflaim.h
--snip--
```

*清单 15-13*：xflaim/docs/doxygen/doxyfile.in：*此文件中包含 Autoconf 变量的行*

这个文件中还有很多其他行，但它们与输出文件完全相同，所以为了节省空间和提高清晰度，我省略了它们。这里的关键是 `config.status` 会将这些替换变量替换为它们在 *configure.ac* 中定义的值，并由 Autoconf 本身定义。如果这些值在 *configure.ac* 中发生变化，生成的文件将会使用新值重新写入。我在 xflaim 的 *configure.ac* 文件中为 *xflaim/docs/doxygen/doxyfile* 添加了一个条件引用到 `AC_CONFIG_FILES` 列表中。就这么简单。

清单 15-14 显示了*xflaim/docs/doxygen/Makefile.am*文件。

```
➊ docpkg = $(PACKAGE_TARNAME)-doxy-$(PACKAGE_VERSION).tar.gz

➋ doc_DATA = $(docpkg)

➌ $(docpkg): doxygen.stamp
           tar chof - html | gzip -9 -c >$@

   doxygen.stamp: doxyfile
           $(DOXYGEN) $(DOXYFLAGS) $<
           echo Timestamp > $@

➍ install-data-hook:
           cd $(DESTDIR)$(docdir) && tar xf $(docpkg)

   uninstall-data-hook:
           cd $(DESTDIR)$(docdir) && rm -rf html

➎ CLEANFILES = doxywarn.txt doxygen.stamp $(docpkg)

   clean-local:
           rm -rf html
```

*清单 15-14*：xflaim/docs/doxygen/Makefile.am：*此 Makefile 的完整内容*

在这里，我在➊为包含 Doxygen 文档文件的 tar 包创建了一个包名称。这基本上与 xflaim 项目的分发 tar 包相同，只不过包名称后包含`-doxy`。

我在➋定义了一个`doc_DATA`变量，该变量包含 Doxygen tar 包的名称。该文件将安装到`$(docdir)`目录，默认情况下是`$(datarootdir)`*/doc/*`$(PACKAGE_TARNAME)`，而`$(datarootdir)`由 Automake 配置为`$(prefix)`*/share*，默认情况下如此。

**注意**

*The* `DATA` *主目标带来了显著的 Automake 功能——安装由系统自动管理。虽然我必须构建 Doxygen 文档包，但* `DATA` *主目标自动为我挂钩`all`目标，这样当用户执行`make`或`make all`时，我的包就会被构建。

我在➌使用另一个印章文件，因为 Doxygen 会从我的项目中的源文件生成数百个*.html*文件。与其尝试找出一个合理的方法来分配依赖关系，我选择生成一个印章文件，然后使用它来判断文档是否过时。^(13)

我还决定，将文档归档解压到包的*doc*目录会很不错。如果仅依赖 Automake，tar 包会在安装时被放入正确的目录，但仅此而已。我需要能够挂钩安装过程来完成这一操作，而这正是 Automake `-hook`目标的完美应用。我在➍使用`install-data-hook`目标，因为`-hook`目标允许你在被挂钩操作完成后执行额外的用户定义的 Shell 命令。同样，我使用`uninstall-hook`来删除在安装过程中提取*.tar*文件时创建的*html*目录。（卸载平台特定文件和平台无关文件之间没有区别，因此卸载文件时只有一个钩子。）

为了清理我生成的文件，我使用了➎处的`CLEANFILES`变量和一个`clean-local`规则，只是为了演示它可以被完成。

### 添加非标准目标

添加一个新的非标准目标与挂钩现有目标略有不同。首先，你不需要使用 `AM_CONDITIONAL` 和其他 Autoconf 测试来检查是否拥有所需的工具。相反，你可以直接从 *Makefile.am* 文件中进行所有条件测试，因为你控制与目标相关的整个命令集，尽管这并不是推荐的做法。（最好从 `configure` 脚本确保构建环境配置正确。）在某些情况下，如果 `make` 目标只能在特定条件下或特定平台上工作，最好在目标中提供检查，确保请求的操作实际上可以执行。

一开始，我在每个项目的根目录下创建了一个名为 *obs* 的目录，用来存放构建 RPM 包文件的 *Makefile.am* 文件。（*OBS* 是 *openSUSE Build Service* 的缩写，一个在线包构建服务。）^(14)

构建 RPM 包文件是通过一个配置文件完成的，这个配置文件叫做 *spec* 文件，它非常类似于用于为特定项目配置 Doxygen 的 *doxyfile*。与 *doxyfile* 一样，RPM spec 文件引用了 `configure` 知道的包信息。因此，我编写了一个 *xflaim.spec.in* 文件，在适当的地方添加了替换变量，然后将另一个文件引用添加到 `AC_CONFIG_FILES` 宏中。这使得 `configure` 可以将项目的信息替换到 spec 文件中。清单 15-15 显示了 *xflaim.spec.in* 文件中的相关部分。

```
Name: @PACKAGE_TARNAME@
BuildRequires: gcc-c++ libstdc++-devel flaimtk-devel gcc-java gjdoc fastjar
mono-core doxygen
Requires: libstdc++ flaimtk mono-core java >= 1.4.2
Summary: XFLAIM is an XML database library.
URL: http://sourceforge.net/projects/flaim/
Version: @PACKAGE_VERSION@
Release: 1
License: GPL
Vendor: Novell, Inc.
Group: Development/Libraries/C and C++
Source: %{name}-%{version}.tar.gz
BuildRoot: %{_tmppath}/%{name}-%{version}-build
--snip--
```

*清单 15-15*：x flaim/obs/xflaim.spec.in：*此文件中展示了如何使用 Autoconf 变量的部分*

请注意在这个清单中使用了变量 `@PACKAGE_TARNAME@` 和 `@PACKAGE_VERSION@`。虽然在这个项目的生命周期中，tar 文件名不太可能发生变化，但版本号会经常变化。如果没有 Autoconf 替换机制，每当我更新 *configure.ac* 文件中的版本时，我必须记得更新这个版本号。清单 15-16 展示了 *xflaim/obs/Makefile.am* 文件，它实际上完成了构建 RPM 的工作。

```
   rpmspec = $(PACKAGE_TARNAME).spec

   rpmmacros =\
    --define="_rpmdir $${PWD}"\
    --define="_srcrpmdir $${PWD}"\
    --define="_sourcedir $${PWD}/.."\
    --define="_specdir $${PWD}"\
    --define="_builddir $${PWD}"
   RPMBUILD = rpmbuild
   RPMFLAGS = --nodeps --buildroot="$${PWD}/_rpm"

➊ rpmcheck:
           if ! ($(RPMBUILD) --version) >/dev/null 2>&1; then \
             echo "*** This make target requires an rpm-based Linux
   distribution."; \
             (exit 1); exit 1; \
           fi

   srcrpm: rpmcheck $(rpmspec)
           $(RPMBUILD) $(RPMFLAGS) -bs $(rpmmacros) $(rpmspec)

   rpms: rpmcheck $(rpmspec)
           $(RPMBUILD) $(RPMFLAGS) -ba $(rpmmacros) $(rpmspec)

   .PHONY: rpmcheck srcrpm rpms
```

*清单 15-16*：xflaim/obs/Makefile.am：*该 makefile 的完整内容*

构建 RPM 包非常简单，正如你所看到的。这个 makefile 提供的目标包括 `srcrpm` 和 `rpms`。➊ 处的 `rpmcheck` 目标在内部使用，用于验证 RPM 是否可以在最终用户的环境中构建。

要查明低级别 *Makefile.am* 文件中哪些目标被顶级构建支持，请查看顶级 *Makefile.am* 文件。如 清单 15-17 所示，如果目标没有传递下来，那么该目标必须仅用于内部，在低级目录中。

```
--snip--
RPM = rpm

rpms srcrpm: dist
     ➊ (cd obs && $(MAKE) $(AM_MAKEFLAGS) $@) || exit 1
        rpmarch=`$(RPM) --showrc | grep "^build arch" | \
          sed 's/\(.*: \)\(.*\)/\2/'`; \
        test -z "obs/$$rpmarch" || \
          ( mv obs/$$rpmarch/* . && rm -rf /obs/$$rpmarch )
        rm -rf obs/$(distdir)
--snip--
.PHONY: srcrpm rpms
```

*清单 15-17*：xflaim/Makefile.am：*如果目标没有传递下来，它就是一个内部目标。*

正如你从清单 15-17 中➊处的命令可以看到，当用户从顶级构建目录中选择`rpms`或`srcrpm`时，命令会递归传递给*obs/Makefile*。其余的命令则只是删除 RPM 构建过程中留下的垃圾文件，这些垃圾文件在这一层次上更容易清除。（有机会构建一次 RPM 包，你就会明白我是什么意思！）

还要注意，这两个顶级 makefile 目标都依赖于`dist`目标，因为 RPM 构建过程需要分发的 tarball。将 tarball 添加为`rpms`目标的依赖项，可以确保在`rpmbuild`工具需要它时，分发 tarball 已经存在。

### 总结

在使用 Autotools 时，你需要管理许多细节——大多数细节，如开源软件界所说，*可以等到下一次发布再处理*！即使我将这段代码提交到 FLAIM 项目的代码库时，我也注意到有一些细节是可以改进的。这里的关键教训是，构建系统永远不可能完成。它应该随着时间推移而逐步改进，利用你时间表中的空闲时间进行优化。而且，做到这一点是很有回报的。

我已经向你展示了许多本书前面章节没有涉及到的新特性，当然还有许多更多的特性是本书无法涵盖的。要真正精通，建议你阅读 Autotools 的手册。到现在为止，你应该能够轻松地自己去获取这些额外的信息。
