## 第十三章：使用 Gnulib 达到最大可移植性

*没有任何创作是由两个人完成的。在艺术、音乐、诗歌、数学、哲学中没有良好的合作。创作的奇迹一旦发生，群体可以建设和扩展它，但群体永远不会发明任何东西。*

——约翰·斯坦贝克，《伊甸园之东》

![Image](img/common.jpg)

你知道那些你过去十年左右一直在使用的酷炫脚本语言吗——Python、PHP、Perl、JavaScript、Ruby 等等？这些语言的最酷功能之一，甚至一些像 Java 这样的编译语言，也具备通过 pip 和 maven 等工具访问社区提供的库功能的能力，来自 PEAR、RubyGems、CPAN 和 Maven Central 等仓库。

难道你不希望能在 C 和 C++ 中做类似的事情吗？你可以在 C 中体验这种感觉，使用 *GNU 可移植性库 (Gnulib)*^(1)，以及它的伴随命令行工具 `gnulib-tool`。Gnulib 是一个旨在广泛可移植的源代码库，甚至可以移植到像 Windows 这样的平台，使用本地编译和基于 Cygwin 的编译方式（尽管 Gnulib 在 Cygwin 上的测试稍多于在本地 Windows 构建中的测试）。

Gnulib 中有数百个便携的工具函数，它们的设计目标只有一个——便于移植到许多不同的平台。本章将介绍如何开始使用 Gnulib，并如何充分发挥它的优势。

### 许可证警告

在我继续之前，我应该提到，Gnulib 的大部分源代码是根据 GPLv3+ 或 LGPLv3+ 许可证发布的。然而，一些 Gnulib 的源代码是根据 LGPLv2+ 许可证发布的，这可能会使得相关功能稍微更易接受。可以合理用于库中的 Gnulib 函数是根据 LGPLv2+ 或 LGPLv3+ 许可证发布的；其他的则是根据 GPLv3+ 许可证发布，或者是某种混合的“LGPLv3+ 和 GPLv2”许可（从最终的兼容性上看，它与 GPLv2 比与 LGPLv2 更兼容）。如果这让你感到困扰，你可以跳过这一章，但在完全放弃 Gnulib 之前，考虑检查你希望使用的功能的许可证，看看你的项目是否能够适应它。

由于 Gnulib 是以源代码格式分发的，并且设计上是要以这种格式纳入应用程序和库中，使用 Gnulib 就意味着将 GPL 和 LGPL 源代码直接纳入到你的源代码库中。至少，这意味着你需要使用 GPL 和 LGPL 许可证对部分代码进行授权。这也许能解释为什么 Gnulib 并不是特别流行，除了其他 GNU 软件包的维护者。

另一方面，如果你正在编写一个已经根据 GPL 许可证发布的开源程序，或者一个已经使用 LGPL 的开源库，那么你的项目非常适合使用 Gnulib。继续阅读。

### 开始使用

如前所述，Gnulib 以源代码格式发布。虽然你可以随时访问 Savannah git 仓库，在线浏览并下载单个文件，但更简单的方法是将 Gnulib 仓库克隆到本地工作区。Gnulib 仓库在根目录中提供了`gnulib-tool`工具，你可以用它将所需的源模块、伴随的 Autoconf 宏和构建脚本直接复制到你的项目中。

`gnulib-tool`工具可以直接在仓库的根目录下运行。为了方便访问，可以在你的`PATH`中某个位置创建到这个程序的软链接；这样你就可以在项目目录中运行`gnulib-tool`，将 Gnulib 模块添加到基于 Autotools 的项目中：

```
$ git clone https://git.savannah.gnu.org/git/gnulib.git
--snip--
$ ln -s $PWD/gnulib/gnulib-tool $HOME/bin/gnulib-tool
$
```

这就是使 Gnulib 在你的系统上以最有效的方式可用所需的一切。

**注意**

*上游的 Gnulib 项目不做发布，而是直接将更改和修复直接合并到主分支中。本章中的编程示例是使用 Savannah Gnulib git 仓库中提交号为 f876e0946c730fbd7848cf185fc0dcc712e13e69 的 Gnulib 源代码编写的。如果你在构建本章代码时遇到问题，可能是因为自本书编写以来，Gnulib 源代码有所变化。尝试退回到这个 Gnulib 提交版本。*

### 将 Gnulib 模块添加到项目

为了帮助你理解如何使用 Gnulib，让我们创建一个有实际用途的项目。我们将编写一个程序，用于将数据转换为 base64 字符串并反向转换，这在今天被广泛使用，而 Gnulib 提供了一个可移植的 base64 转换功能库。我们将从创建一个仅包含`main`函数的小程序开始，这个程序将作为驱动程序，稍后我们将添加 Gnulib 的 base64 转换功能。

**注意**

*该项目的源代码在 NSP-Autotools GitHub 仓库中的* b64 *目录，地址为* [`github.com/NSP-Autotools/b64/`](https://github.com/NSP-Autotools/b64/)。

Git 标签：13.0

```
$ mkdir -p b64/src
$ cd b64
$
```

编辑*src/b64.c*并添加 Listing 13-1 中显示的内容。

```
#include "config.h"
#include <stdio.h>

int main(void)
{
    printf("b64 - convert data to and from base64 strings.\n");
    return 0;
}
```

*Listing 13-1*：src/b64.c：*驱动程序主源文件的初始内容*

现在让我们运行`autoscan`以提供一个基础的*configure.ac*文件，将新的*configure.scan*文件重命名为*configure.ac*，然后为我们的项目创建一个*Makefile.am*文件。请注意，我在这里创建的是一个非递归的 Automake 项目，将单个源文件*src/b64.c*直接添加到顶层*Makefile.am*文件中。

由于我们没有创建“外部”项目，因此我们还需要添加标准的 GNU 文本文件（但如果你希望，可以在*configure.ac*中的`AM_INIT_AUTOMAKE`宏参数列表中添加`foreign`，以避免做这些修改）：

```
$ autoscan
$ mv configure.scan configure.ac
$ echo "bin_PROGRAMS = src/b64
src_b64_SOURCES = src/b64.c" >Makefile.am
$ touch NEWS README AUTHORS ChangeLog
$
```

编辑新的*configure.ac*文件，并按照 Listing 13-2 中的更改进行修改。

```
#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.69])
AC_INIT([b64], [1.0], [b 64-bugs@example.org])
AM_INIT_AUTOMAKE([subdir-objects])
AC_CONFIG_SRCDIR([src/b64.c])
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_MACRO_DIRS([m4])
--snip--
AC_CONFIG_FILES([Makefile])

AC_OUTPUT
```

*Listing 13-2*：configure.ac：*`autoscan`生成的* configure.ac *文件需要的更改*

我已经在 `AM_INIT_AUTOMAKE` 宏中添加了 `subdir-objects` 选项，以创建一个非递归的 Automake 构建系统。我还添加了 `AC_CONFIG_MACRO_DIRS` 宏以保持系统的清晰。^(2)

此时，我们应该能够运行 `autoreconf -i`，然后执行 `configure` 和 `make`，以构建项目：

```
$ autoreconf -i
aclocal: warning: couldn't open directory 'm4': No such file or directory
configure.ac:12: installing './compile'
configure.ac:6: installing './install-sh'
configure.ac:6: installing './missing'
Makefile.am: installing './INSTALL'
Makefile.am: installing './COPYING' using GNU General Public License v3 file
Makefile.am:     Consider adding the COPYING file to the version control
system
Makefile.am:     for your code, to avoid questions about which license your
project uses
Makefile.am: installing './depcomp'
$
$ ./configure && make
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /bin/mkdir -p
--snip--
config.status: creating Makefile
config.status: creating config.h
config.status: executing depfiles commands
make  all-am
make[1]: Entering directory '/.../b64'
depbase=`echo src/b64.o | sed 's|[^/]*$|.deps/&|;s|\.o$||'`;\
gcc -DHAVE_CONFIG_H -I.         -g -O2 -MT src/b64.o -MD -MP -MF $depbase.Tpo -c
-o src/b64.o src/b64.c &&\
mv -f $depbase.Tpo $depbase.Po
gcc    -g -O2 -o src/b64 src/b64.o
make[1]:   Leaving directory '/.../b64'
$
$ src/b64
$ b64 - convert data to and from base64 strings.
$
```

现在我们可以开始将 Gnulib 功能添加到该项目中。我们需要做的第一件事是使用`gnulib-tool`将 base64 模块导入到我们的项目中。假设你已经正确克隆了 Gnulib git 项目，并将 `gnulib-tool` 的软链接添加到你的 `PATH` 中的某个目录（如果该目录在你的 `PATH` 中，可以是 *$HOME/bin*），请从 *b64* 项目目录结构的根目录执行以下命令：

Git 标签 13.1

```
$ gnulib-tool --import base64
Module list with included dependencies (indented):
    absolute-header
  base64
    extensions
    extern-inline
    include_next
    memchr
    snippet/arg-nonnull
    snippet/c++defs
    snippet/warn-on-use
    stdbool
    stddef
    string
File list:
  lib/arg-nonnull.h
  lib/base64.c
  lib/base64.h
  --snip--
  m4/string_h.m4
  m4/warn-on-use.m4
  m4/wchar_t.m4
Creating directory ./lib
Creating directory ./m4
Copying file lib/arg-nonnull.h
Copying file lib/base64.c
Copying file lib/base64.h
--snip--
Copying file m4/string_h.m4
Copying file m4/warn-on-use.m4
Copying file m4/wchar_t.m4
Creating lib/Makefile.am
Creating m4/gnulib-cache.m4
Creating m4/gnulib-comp.m4
Creating ./lib/.gitignore
Creating ./m4/.gitignore
Finished.

You may need to add #include directives for the following .h files.
  #include "base64.h"

Don't forget to
  - add "lib/Makefile" to AC_CONFIG_FILES in ./configure.ac,
  - mention "lib" in SUBDIRS in Makefile.am,
  - mention "-I m4" in ACLOCAL_AMFLAGS in Makefile.am,
  - mention "m4/gnulib-cache.m4" in EXTRA_DIST in Makefile.am,
  - invoke gl_EARLY in ./configure.ac, right after AC_PROG_CC,
  - invoke gl_INIT in ./configure.ac.
$
```

在这个控制台示例中省略的列表，当使用一个有很多依赖于其他 Gnulib 模块的模块时，可能会变得相当长。*base64* 模块仅直接依赖于 *stdbool* 和 *memchr* 模块；然而，依赖关系列表显示了其他的传递性依赖关系。你可以通过检查模块在 *MODULES* 页面上的依赖列表，或者通过阅读你克隆的 Gnulib 仓库中的 *modules/base64* 文件，在决定是否使用该模块之前查看其直接依赖项。此页面可以在 *[gnu.org](http://gnu.org)* 找到。^(3)

base64 模块所需的一些传递性依赖项包括一些模块，旨在使 base64 更加可移植，适用于多种平台。例如，*string* 模块提供了一个包装器，用于你系统中的 *string.h* 头文件，提供了额外的常用字符串功能，或者修复了一些平台上的 bug。

从输出中可以看到，创建了两个目录——*m4* 和 *lib*——然后一些支持的 M4 宏文件被添加到 *m4* 目录中，一些源代码和构建文件被添加到 *lib* 目录中。

**注意**

*如果你在一个 git 仓库中工作，*`gnulib-tool`* 会向* m4 *和* lib *目录中添加 .gitignore 文件，这样当你运行类似* `git add -A` *的命令时，可以重新生成或重新复制的文件就不会被自动检查进去了。*相反，你会看到只会添加* lib/.gitignore, m4/.gitignore *和* m4/gnulib-cache.m4 *这几个文件。*所有其他文件可以在你使用所需的 Gnulib 模块配置项目后重新生成（或重新复制）。*

最后，在输出的末尾，`gnulib-tool` 会为你提供一些简洁的说明，告诉你如何使用你添加的 base64 模块。首先，根据这些说明，我们需要将 *lib/Makefile* 添加到我们在 *configure.ac* 中的 `AC_CONFIG_FILES` 列表中。稍后在同一列表中，我们会找到关于 *configure.ac* 更多一般修改的说明。*示例 13-3* 显示了我们应该根据这些说明对 *configure.ac* 进行的所有更改。

```
--snip--
# Checks for programs.
AC_PROG_CC
gl_EARLY

# Checks for libraries.

# Checks for header files.

# Initialize Gnulib.
gl_INIT

# Checks for typedefs, structures, and compiler characteristics.

# Checks for library functions.

AC_CONFIG_FILES([Makefile lib/Makefile])

AC_OUTPUT
```

*示例 13-3*：configure.ac：*Gnulib 需要的更改*

一些指令还指示了我们项目中顶层*Makefile.am*文件所需的更改。列表 13-4 突出了这些更改。

```
ACLOCAL_AMFLAGS = -I m4
EXTRA_DIST = m4/gnulib-cache.m4
SUBDIRS = lib

bin_PROGRAMS = src/b64
src_b64_SOURCES = src/b64.c
```

*列表 13-4*：Makefile.am：*Gnulib 所需的更改*

在做出这些更改之后，你的项目应该继续构建。我们需要运行`autoreconf -i`，以包括现在由我们添加到*configure.ac*中的 Gnulib 宏所要求的额外文件。

当我们导入 base64 模块时，`gnulib-tool`的输出指示我们可能需要添加一个*base64.h*的包含指令。目前，我们不需要这样的指令，因为我们的代码实际上并未使用 base64 的任何功能。我们即将进行更改，但每个模块都有自己的包含指令集，因此我接下来将展示的步骤仅与 base64 模块相关。其他模块也有类似的步骤，但会具体针对你选择使用的模块。每个模块的文档会告诉你如何访问该模块的公共接口——也就是说，应该包含哪些头文件。

虽然文档对此点的说明不是特别清晰，但实际上你不需要将任何模块特定的库链接到你的项目中，因为*lib/Makefile.am*文件会构建所有导入模块的源文件，并将结果对象添加到一个名为*libgnu.a*的静态库中。这是一个定制版的 Gnulib 库，仅包含你拉入项目中的模块。由于 Gnulib 是一个源代码库，因此不需要项目消耗 Gnulib 功能的二进制文件（除了*lib*目录中构建的那个）。因此，链接 Gnulib 功能的过程对于所有 Gnulib 模块都是相同的。

让我们将一些 base64 的功能添加到我们的项目中，看看实际使用该模块涉及哪些内容。根据列表 13-5 中的更改，对你的*src/b64.c*文件进行修改。

Git 标签 13.2

```
#include "config.h"
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>
#include <errno.h>
#include <unistd.h>

#include "base64.h"

#define BUF_GROW 1024

static void exit_with_help(int status)
{
    printf("b64 – convert data TO and FROM base64 (default: TO).\n");
    printf("Usage: b64 [options]\n");
    printf("Options:\n");
    printf("  -d   base64-decode stdin to stdout.\n");
    printf("  -h   print this help screen and exit.\n");
    exit(status);
}

static char *read_input(FILE *f, size_t *psize)
{
    int c;
    size_t insize, sz = 0;
    char *bp = NULL, *cp = NULL, *ep = NULL;

    while ((c = fgetc(f)) != EOF)
    {
        if (cp >= ep)
        {
            size_t nsz = sz == 0 ? BUF_GROW : sz * 2;
            char *np = realloc(bp, nsz);
            if (np == NULL)
            {
                perror("readin realloc");
                exit(1);
            }
 cp = np + (cp - bp);
            bp = np;
            ep = np + nsz;
            sz = nsz;
        }
        *cp++ = (char) c;
    }
    *psize = cp - bp;
    return bp;
}

static int encode(FILE *f)
{
    size_t insize;
    char *outbuf, *inbuf = read_input(f, &insize);
    size_t outsize = base64_encode_alloc(inbuf, insize, &outbuf);
    if (outbuf == NULL)
    {
        if (outsize == 0 && insize != 0)
        {
            fprintf(stderr, "encode: input too long\n");
            return 1;
        }
        fprintf(stderr, "encode: allocation failure\n");
    }
    fwrite(outbuf, outsize, 1, stdout);
    free(inbuf);
    free(outbuf);
    return 0;
}

static int decode(FILE *f)
{
    size_t outsize, insize;
    char *outbuf, *inbuf = read_input(f, &insize);
    bool ok = base64_decode_alloc(inbuf, insize, &outbuf, &outsize);
    if (!ok)
    {
        fprintf(stderr, "decode: input not base64\n");
        return 1;
    }
    if (outbuf == NULL)
    {
        fprintf(stderr, "decode: allocation failure\n");
        return 1;
    }
    fwrite(outbuf, outsize, 1, stdout);
    free(inbuf);
    free(outbuf);
    return 0;
}
int main(int argc, char *argv[])
{
 int c;
    bool tob64 = true;

    while ((c = getopt(argc, argv, "dh")) != -1)
    {
        switch (c)
        {
            case 'd':
                tob64 = false;
                break;
            case 'h':
            default:                exit_with_help(c == 'h' ? 0 : 1);
        }
    }
    return tob64 ? encode(stdin) : decode(stdin);
}
```

*列表 13-5*：src/b64.c：*集成 base64 功能所需的更改*

我已在列表 13-5 中提供了整个文件，因为原始代码中只剩下几行。这程序旨在作为 Unix 过滤器，读取`stdin`中的输入数据并将输出数据写入`stdout`。要从文件读取和写入，只需使用命令行重定向。

我应该提到一些关于这个程序的重要事项。首先，它在`read_input`函数中使用了一个缓冲区增长算法。这个代码的大部分可以通过调用另一个 Gnulib 模块函数`x2nrealloc`来替代。在线文档对该方法的使用，甚至对它的存在，描述得很少——可能是因为 xalloc 接口已经以不同形式存在了很多年。你可以在 Gnulib 源代码的*lib*目录下找到*xalloc.h*头文件，里面有很多长注释，包含了许多函数的示例用法，包括`x2nrealloc`函数。

使用 xalloc 功能来满足所有内存分配需求的另一个优点是，它的分配函数会自动检查`NULL`返回值，并在内存分配失败时通过适当的错误信息中止程序。如果你希望对中止过程有更多控制，可以向代码中添加一个名为`xalloc_die`的函数（无参数，无返回值），如果它存在，xalloc 函数会调用它。你可以使用这个钩子在程序退出前执行任何必要的清理工作。为什么不让你来决定是否退出呢？你内存不足——你究竟能做什么？在今天多 TB 地址空间的世界中，这种内存不足的情况不常发生，但仍然需要进行检查。xalloc 函数使得进行这种检查变得不那么痛苦。

最后，与许多过滤器不同，如果你向这个程序输入一个包含 1GB 数据的文件，它可能会崩溃，因为它会将整个输入缓冲到一个已分配的内存块中，并在读取`stdin`数据时调整其大小。原因是默认的 base64 模块使用方式并不设计为处理流式数据。它要求事先准备好整个缓冲区。然而，有一个`base64_encode_alloc_ctx`方法，允许你以迭代方式编码输入文本的小块。我将这个任务留给你，读者，让你修改这个程序以使用这种 base64 模块的形式。

为了使这段代码正确构建，你需要按照示例 13-6 所示更改*Makefile.am*。

```
ACLOCAL_AMFLAGS = -I m4
EXTRA_DIST = m4/gnulib-cache.m4
SUBDIRS = lib

bin_PROGRAMS = src/b64
src_b64_SOURCES = src/b64.c
src_b64_CPPFLAGS = -I$(top_builddir)/lib -I$(top_srcdir)/lib
src_b64_LDADD = lib/libgnu.a
```

*示例 13-6*：Makefile.am：*在源代码中使用 base64 模块所需的更改*

`src_b64_CPPFLAGS`指令将目录添加到编译器的包含搜索路径中，以便它能够找到通过选定的 Gnulib 模块添加的任何头文件。`src_b64_LDADD`指令将*lib/libgnu.a*追加到链接器命令行中。这两个指令到现在为止应该已经很熟悉了。

让我们构建并运行`b64`程序。正如我之前提到的，首先你需要运行`autoreconf -i`，以应用 Gnulib 对项目所做的任何更改。

```
$ autoreconf -i
--snip--
$ ./configure && make
--snip--
$ echo hi | src/b64
aGkK$ echo -n aGkK | src/b64 -d
hi
$
```

我使用`echo`将一些文本通过管道传递到`b64`过滤器，后者输出该文本的 base64 等效：“`aGkK`”。注意输出的末尾没有换行符。`b64`过滤器只输出输入数据的 base64 文本版本。然后，我使用`echo -n`将 base64 文本重新传入过滤器，使用`-d`标志解码回原始输入数据。输出是原始文本，包括一个终止的换行符。默认情况下，`echo`会将换行符附加到你输入的任何文本末尾；因此，原始的编码文本包括一个终止的换行符。`-n`选项告诉`echo`抑制换行符。如果不使用`-n`，解码将失败，并出现错误，提示输入数据不是有效的 base64 文本，因为`echo`附加了一个换行符，而这不是 base64 文本的一部分。

从 Gnulib 文档中并不清楚的一点是，按照“从不提交可以轻松再生的文件或数据”的一般原则，Gnulib 的 *.gitignore* 文件会阻止导入的模块源代码被提交到你的仓库。这样做有几个原因。首先，Gnulib 源代码已经存在于一个仓库中——那就是 Gnulib 本身的仓库。没有必要通过将它存储在每个使用它的仓库中来使 Gnulib 源代码在互联网上大量传播。

不将其存储在你的项目仓库中的另一个原因是，用户和维护者总是提供修复补丁。每次更新你的 Gnulib 工作区并构建项目时，你可能会得到你正在使用的模块的更好版本。

假设你今天的工作已经完成，并且你想将工作区恢复到一个干净的状态。你输入`git clean -xfd`，然后清除所有未暂存或未提交的内容。第二天，你回来并输入`autoreconf -i`，接着是`configure && make`，但你发现项目无法构建；*m4*和*lib*目录中似乎有一些重要的文件丢失了。事实上，你发现，只有*m4/gnulib-cache.m4*文件作为一个微妙的提醒，告诉你项目曾经与 Gnulib 有关。

实际上，那个*gnulib-cache.m4*文件就是你真正需要的。它告诉`gnulib-tool`你已经导入了哪些模块。要重新获取所有内容，只需使用`--update`选项执行`gnulib-tool`。这会让`gnulib-tool`将所有相关的 Gnulib 文件的当前版本重新复制到你的项目中。

**注意**

*使用 *`--update`* 选项与 *`gnulib-tool`* 并不会从远程仓库更新你的 Gnulib 工作区。相反，它仅仅更新你项目中使用的 Gnulib 模块，并用当前存在于 Gnulib 工作区中的文件替换这些模块。如果你真的想使用某个过去版本的 Gnulib 模块，你可以从过去检出一个 Gnulib 仓库的修订版本，然后运行 *`gnulib-tool --update`* 来从 Gnulib 工作区中拉取当前的文件集。*

`--update` 选项也可以在你使用 git 更新了 Gnulib 工作区后，用来复制更新后的文件版本。

为了帮助你记住在使用 Gnulib 的项目中使用 `gnulib-tool --update`，Gnulib 手册建议你创建一个 `bootstrap.sh` 脚本（并标记为可执行），脚本中至少包含 列表 13-7 中显示的行。

Git 标签 13.3

```
#!/bin/sh
gnulib-tool --update
autoreconf -i
```

*列表 13-7*：`bootstrap.sh`：*b64 的项目引导脚本*

如果 `autoreconf` 足够智能，能够注意到你使用了 Gnulib 模块并自动为你调用 `gnulib-tool --update`，那该有多好。我怀疑这是 Autoconf 未来版本中的一项功能。当前而言，然而，你需要记得在将项目仓库克隆到新的工作区或在你要求 git 将当前工作区恢复为干净状态之后，手动运行此命令来拉取 Gnulib 文件。

### 摘要

在本章中，我讨论了如何将 Gnulib 模块添加到基于 Autotools 的项目中。我相信我已经给了你足够的 Gnulib 资源，让你对它产生兴趣。只要你掌握了基础，Gnulib 手册写得很好，容易理解（虽然文档不是特别全面）。

下一步是你去 Gnulib 模块页面，浏览可用的功能。模块的头文件和源代码也可以从该页面以及仓库中的 *modules* 和 *lib* 目录中查看。随时可以查看它们。

维护者总是可以在文档方面获得帮助。 一旦你使用了一个模块并且变得熟悉它，看看它的文档是否需要更新，并考虑成为贡献者。你可以使用 Gnulib 邮件列表^(5)作为资源，无论是关于使用 Gnulib 的问题，还是文档和源代码的补丁^(6)。
