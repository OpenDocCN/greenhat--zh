## 第十八章：创建优秀项目的技巧和可重用解决方案目录

*经验是一个严厉的老师，因为她先给测试，然后才教课。

—Vernon Sanders Law^(1)*

![Image](img/common.jpg)

本章最初是作为一个可重用解决方案的目录——可以说是预设宏。但当我完成了之前的章节后，我意识到我需要拓宽我对 *预设解决方案* 的定义。与其仅仅列出有趣的宏，本章列出了几个不相关但对创建优秀项目非常重要的技巧。其中一些与 GNU Autotools 相关，但其他的只是涉及开源和自由软件项目的良好编程实践。

### 项目 1：将私有细节与公共接口隔离

有时，我遇到过设计不良的库接口，其中一个项目的 *config.h* 文件是该项目公共头文件所必须的。当消费者需要多个这样的库时，就会出现问题。应该包含哪个 *config.h* 文件呢？它们的名字相同，而且很可能提供了相似或相同的定义。

当你仔细考虑 *config.h* 的目的时，你会发现将其暴露在库的公共接口中（通过将其包含在任何公共头文件中）没有多大意义，因为它的目的是为库的特定构建提供平台特定的定义。另一方面，便携库的公共接口定义上是平台无关的。

接口设计是计算机科学中的一个相当通用的话题。这个项目专注于如何避免在公共接口中包含 *config.h*，并通过这一点，确保你永远不要安装 *config.h*。

在为其他项目设计库时，你有责任避免将无用的垃圾从你的头文件污染到消费者的符号空间。我曾经参与过一个项目，它使用了来自另一个团队的库接口。该团队提供了 Windows 和 Unix 版本的库，头文件在这两个平台之间是可移植的。不幸的是，他们没有理解干净接口的定义。在他们的公共头文件中，某个地方有一段代码看起来像是 清单 18-1。

```
#ifdef _WIN32
# include <windows.h>
#else
typedef void * HANDLE;
#endif
```

*清单 18-1：一个设计不良的公共头文件，暴露了平台特定的头文件*

哎呀！他们真的需要仅仅为了定义 `HANDLE` 就包含 *windows.h* 吗？不需要，他们可能应该为其公共接口中的句柄对象使用一个不同的名称，因为 `HANDLE` 太过通用，容易与其他许多库的接口冲突。像 `XYZ_HANDLE` 或更具体的与 *XYZ* 库相关的名称会是更好的选择。

为了正确设计一个库，首先设计公共接口，尽可能少地暴露库的内部实现。现在，你需要确定*合理*的定义是什么，但这通常会在抽象和性能之间进行折衷。

在设计 API 时，从你想要暴露的库功能开始；设计能最大化易用性的函数。如果你发现自己在选择更简单的实现和更简单的用户体验之间犹豫不决，始终倾向于为消费者提供更易用的体验。他们会通过实际使用你的库来感谢你。当然，如果接口已经由软件标准定义，那么你的工作大部分已经完成。通常情况并非如此，你将不得不做出这些决策。

接下来，尝试抽象掉内部细节。不幸的是，C 语言并不容易做到这一点，因为你通常需要在公共 API 中传递包含内部实现细节的结构引用，而消费者不需要看到这些细节。（C++在这方面实际上更糟：C++类在同一个类定义中定义公共接口和私有实现细节。）

#### *C 中的解决方案*

在 C 语言中，解决这个问题的常见方法是为私有结构定义一个公共别名，通常是通过通用（`void`）指针。许多开发者不喜欢这种方法，因为它降低了接口的类型安全性，但类型安全性的丧失被接口抽象化的提升大大抵消了，正如示例 18-2 和 18-3 所示。

```
#include <abc_pub.h>

# include <config.h>

typedef struct
{
    /* private details */
} abc_impl;

int abc_func(abc * p)
{
    abc_impl * ip = (abc_impl *)p;
    /* use 'p' through 'ip' */
}
```

*示例 18-2：一个私有 C 语言源文件的示例*

```
typedef void abc;
int abc_func(abc * p);
```

*示例 18-3：* abc_pub.h：*一个描述公共接口（API）的公共头文件*

请注意，抽象化方便地减轻了在库的公共接口中包含大量非常私有定义的需求。^(2)

但有一种方法可以更好地利用语言语法。在 C 语言中，有一个鲜为人知、且使用更少的概念，叫做*前向声明*，它允许你在公共头文件中仅仅声明类型，而不必在那里实际定义它。示例 18-4 提供了一个使用前向声明来定义函数声明中使用的类型的库公共头文件的示例。

```
struct abc;
--snip--
int abc_func(struct abc * p);
```

*示例 18-4：在公共头文件中使用前向声明*

当然，使用`struct abc`假设你的公共接口中的其他函数返回该类型对象的指针，你可以将其传递给`abc_func`。如果用户需要在传递其地址之前填写结构体，那么这个机制显然不适用。相反，这里的使用仅仅是为了隐藏`struct abc`的内部实现。

#### *C++中的解决方案*

在 C++ 中也可以使用前向声明，但方式不同。在 C++ 中，前向声明更多用于最小化编译时头文件之间的相互依赖，而不是在公共接口中隐藏实现细节。然而，我们可以使用其他技术来将实现细节隐藏在用户之外。

在 C++ 中，使用接口抽象来隐藏实现细节可以通过几种不同的方式完成，其中包括使用虚拟接口和 *PIMPL（私有实现）*模式。

##### PIMPL 模式

在 PIMPL 模式中，实现细节通过指向一个私有实现类的指针隐藏，该指针作为公共接口类中的私有数据存储，正如 清单 18-5 和 18-6 中所示。

```
#include <abc_pub.h>

# include <config.h>

class abc_impl
{
    /* private details */
};

int abc::func(void)
{
    /* use 'pimpl' pointer */
}
```

*清单 18-5：一个私有的 C++ 语言源文件，展示了 PIMPL 模式的正确使用*

```
➊ class abc_impl;

   class abc {
   ➋ abc_impl * pimpl;
   public:
     int func(void);
};
```

*清单 18-6:* abc_pub.h: *公共头文件通过 PIMPL 模式仅暴露少量私有细节。*

如前所述，C++ 语言也允许使用前向声明（如 ➊ 处的声明），用于任何仅通过引用或指针使用的类型（如 ➋ 所示），但在公共接口中从未实际取消引用。因此，私有实现类的定义不需要暴露在公共接口中，因为编译器可以愉快地编译公共接口头文件，而不需要私有实现类的定义。

这里的性能权衡通常涉及动态分配一个私有实现类的实例，然后通过这个指针间接访问类的数据，而不是直接在公共结构体中访问。请注意，所有内部细节现在都方便地隐藏了，因此公共接口不需要这些细节。

##### C++ 虚拟接口

使用 C++ 的另一种方法是定义一个公共的 *接口* 类，其方法被声明为 *纯虚拟*，接口由库内部实现。要访问该类的对象，消费者调用一个公共的 *工厂* 函数，该函数返回指向实现类的指针，并以接口定义的形式表示。清单 18-7 和 18-8 展示了 C++ 虚拟接口的概念。

```
#include <abc_pub.h>

# include <config.h>

class abc_impl : public abc {
    virtual int func(void) {
        int rv;
        // implementation goes here
        return rv;
   }
};
```

*清单 18-7：一个私有的 C++ 语言源文件，实施了一个纯虚拟接口*

```
   #define xyz_interface class

   xyz_interface abc {
   public:
     virtual int func(void) = 0;
   };

➊ abc * abc_instantiate( /* abc_impl ctor params */ );
```

*清单 18-8:* abc_pub.h: *一个公共的 C++ 语言头文件，仅提供接口定义*

在这里，我使用 C++ 预处理器定义了一个新的关键字 `xyz_interface`。根据定义，`xyz_interface` 与 `class` 同义，因此可以互换使用。这里的理念是接口不会向消费者暴露任何实现细节。公共的 *工厂* 函数 `abc_instantiate` 在 ➊ 返回一个指向 `abc_impl` 类型新对象的指针，除了 `abc` 之外。因此，公共头文件中不需要显示任何内部内容给调用者。

可能看起来虚拟接口类的方法比 PIMPL 方法更高效，但实际上，大多数编译器将虚拟函数调用实现为由隐藏的 *vptr* 地址引用的函数指针表，存在于实现类中。因此，你最终还是会通过指针间接调用所有公共方法。你选择隐藏实现细节的技术更多是个人偏好的问题，而不是性能问题。^(3)

当我设计一个库时，我首先设计一个最小的，但完整的，功能接口，并尽可能多地将我的内部实现抽象化。我尽量在函数原型中只使用标准库的基本类型，然后仅包含那些由这些类型和定义所需的 C 或 C++ 标准头文件。这种技巧是我发现的创建高可移植性和可维护性接口的最快方法。

如果你仍然看不出这条建议的价值，那么让我给你再提供一个情景来思考。考虑一下当一个 Linux 发行版的打包者决定为你的库创建一个 *devel* 包时会发生什么——即，一个包含静态库和头文件的包，设计用于安装到目标系统的 */usr/lib* 和 */usr/include* 目录中。你的库所需的每一个头文件必须安装到 */usr/include* 目录中。如果你的库的公共接口需要包含你的 *config.h* 文件，那么扩展来说，你的 *config.h* 文件必须安装到 */usr/include* 目录中。现在考虑一下当多个此类库需要安装时会发生什么。哪一份 *config.h* 文件会被保留？在 */usr/include* 目录中只能存在一个 *config.h* 文件。

我在 Autotools 邮件列表上看到过一些讨论，支持在公共接口中发布 *config.h* 的必要性，并提供一些为包特定方式命名 *config.h* 的技巧。这些技巧通常涉及对该文件进行某种形式的后处理，以重命名其宏，以避免与其他包安装的 *config.h* 定义冲突。虽然这样做是可行的，并且在某些情况下有一些合理的理由（通常是涉及到一个无法修改的广泛使用的遗留代码库，修改它会破坏大量现有代码），但这些情况应该被视为例外，而不是常规，因为一个设计良好的项目不应该需要在其公共接口中暴露平台和项目特定的定义。

如果你的项目在公共接口中无法没有*config.h*，可以探索`AC_CONFIG_HEADERS`宏的细节。像所有实例化宏一样，这个宏接受一个输入文件列表。`autoheader`工具只会写入列表中的第一个输入文件，因此你可以手动创建第二个输入文件，包含你认为必须包含在公共接口中的定义。记得命名你的公共输入文件，以减少与其他包公共接口的冲突。

**注意**

*此外，还可以探索 Autoconf 宏库中的*`AX_PREFIX_CONFIG_H`*宏（见“项目 8：使用 Autoconf 宏库”在第 528 页），它会为 config.h 中的所有项添加自定义前缀。*

### 项目 2：实现递归扩展目标

*扩展目标*是你为实现某个构建目标而编写的`make`目标，Automake 并不会自动支持该目标。*递归扩展目标*是那种遍历你的项目目录结构，访问 Autotools 构建系统中的每个*Makefile.am*文件，并在扩展目标被创建时为每个文件提供执行操作的机会。

当你向构建系统中添加新的顶级目标时，你必须将其绑定到现有的 Automake 目标，或者将你自己的`make`代码添加到所需的目标中，来遍历 Automake 在你的构建系统中提供的子目录结构。

`SUBDIRS`变量用于递归遍历当前目录的所有子目录，并将请求的构建命令传递到这些目录中的 makefile。这对于那些必须基于配置选项构建的目标非常有效，因为在配置之后，`SUBDIRS`变量仅包含那些注定要被构建的目录。

然而，如果你需要在*所有*子目录中执行新的递归目标，不管`SUBDIRS`中是否排除了某些目录，你可以使用`DIST_SUBDIRS`变量。

有多种方法可以遍历构建层次结构，包括 GNU `make`特有语法提供的一些非常简单的一行命令。但最具可移植性的方法是使用 Automake 本身使用的技术，如清单 18-9 所示。

```
my-recursive-target:
      ➊ $(preorder_commands)
        for dir in $(SUBDIRS); do \
          ($(am__cd) $$dir && $(MAKE) $(AM_MAKEFLAGS) $@) || exit 1; \
        done
      ➋ $(postorder_commands)

.PHONY: my-recursive-target
```

*清单 18-9：具有递归目标的 makefile（警告：`SUBDIRS`中不支持“`.`”）*

在层次结构的某个点，你需要做一些有用的事情，而不仅仅是向下调用更低层次。➊处的`preorder_commands`宏可以用于在递归进入更低层次的目录之前做一些必要的操作。➋处的`postorder_commands`宏同样可以在从更低层次的目录返回后执行额外的操作。只需在需要进行某些前序或后序处理的 makefile 中定义这两个宏中的任何一个或两个，即可为`my-recursive-target`执行相应操作。

例如，如果你想生成一些文档，你可能会有一个名为`doxygen`的特殊目标。即使你可以在顶层目录中构建文档，仍然可能需要将文档的生成分发到项目层次结构中的各个目录。你可能会在项目中的每个*Makefile.am*文件中使用类似于清单 18-10 中所示的代码。

```
   # uncomment if doxyfile exists in this directory
➊ # postorder_commands = $(DOXYGEN) $(DOXYFLAGS) doxyfile

   doxygen:
           $(preorder_commands)
        ➋ for dir in $(SUBDIRS); do \
         ➌ ($(am__cd) $$dir && $(MAKE) $(AM_MAKEFLAGS) $@) || exit 1; \
          done
          $(postorder_commands)

.PHONY: doxygen
```

*清单 18-10：为`doxygen`目录实现*`postorder_commands`**

对于没有*doxyfile*的目录，你可以注释掉（或者更好的是，干脆省略）在➊处的`postorder_commands`宏定义。在这种情况下，`doxygen`目标将通过➋处的三行 Shell 代码无害地传播到构建树中的下一级。

在➌处的`exit`语句确保当低级 makefile 在递归目标上失败时，构建会终止，并将 Shell 错误代码（1）传播回每个父 makefile，直到到达顶层 Shell。这一点非常重要；如果没有它，构建可能会在失败后继续，直到遇到另一个错误。

**注意**

*我选择不使用有些不太便携的*`-C make`*命令行选项来在运行子*`make`*操作之前更改目录。我还使用了一个名为*`am__cd`*的 Automake 宏来更改目录。这个宏被定义为考虑*`CDPATH`*环境变量的内容，以减少构建过程中不必要的输出噪音。你可以用*`cd`*（或*`chdir`*）来替代它。检查一个由 Automake 生成的 makefile，看看 Automake 是如何定义这个宏的。*

如果你选择以这种方式实现一个完全递归的全局目标，你必须在项目中的每个*Makefile.am*文件中包含清单 18-10，即使该 makefile 与文档生成无关。如果不这样做，`make`将在该 makefile 上失败，因为其中不存在`doxygen`目标。命令可能什么都不做，但目标必须存在。

如果你想做一些更简单的事情，比如将一个目标传递到顶层目录下的单个子目录（例如，紧邻顶层的*doc*目录），那么生活就变得更轻松了。只需实现清单 18-11 和 18-12 中所示的代码。

```
doxygen:
     ➊ $(am__cd) doc && $(MAKE) $(AM_MAKEFLAGS) $@

.PHONY: doxygen
```

*清单 18-11：一个将目标传播到单个子目录的顶层 makefile*

```
doxygen:
        $(DOXYGEN) $(DOXYFLAGS) doxyfile

.PHONY: doxygen
```

*清单 18-12:* doc/Makefile.am: *处理新目标的代码*

在清单 18-11 的顶层 makefile 中的➊处，Shell 语句只是将目标（`doxygen`）传递到目标目录（`doc`）。

**注意**

*假设变量*`DOXYGEN`*和*`DOXYFLAGS`*已经存在，它们是通过某些宏或 Shell 代码在*`configure`*脚本中执行的。*

Automake 的递归目标更为复杂，因为它们还支持`make`的`-k`命令行选项，以便在发生错误后继续构建。此外，Automake 的递归目标实现支持在`SUBDIRS`变量中使用点（`.`），它代表当前目录。你也可以支持这些功能，但如果支持的话，你的标准递归`make` shell 代码将变得更为混乱。为了完整起见，清单 18-13 展示了支持这些功能的实现。将这个清单与清单 18-9 进行对比，突出显示的 shell 代码展示了这两者之间的差异。

```
my-recursive-target:
        $(preorder_commands)
        @failcom='exit 1'; \
        for f in x $$MAKEFLAGS; do \
          case $$f in \
            *=* | --[!k]*);; \
         ➊ *k*) failcom='fail=yes';; \
          esac; \
        done; \
        for dir in $(SUBDIRS); do \
 ➋ if test "$$dir" != .; then \
             ($(am__cd) $$dir && $(MAKE) $(AM_MAKEFLAGS) $@) || eval \
                $$failcom; \
           fi; \
         done
         $(postorder_commands)

.PHONY: my-recursive-target
```

*清单 18-13：添加`make -k`和检查当前目录*

在➊，`case`语句检查`MAKEFLAGS`环境变量中是否存在`-k`选项，并在找到该选项时将`failcom` shell 变量设置为一些无害的 shell 代码。如果未找到该选项，则`failcom`保持其默认值`exit 1`，然后在错误发生时插入该命令以退出。➋处的`if`语句仅跳过`SUBDIRS`中的点条目递归调用。如前所述，针对当前目录，递归目标的功能完全包含在`$(preorder_commands)`和`$(postorder_commands)`宏扩展中。

我尝试在这个项目中向你展示，你可以根据自己的需要做更多或更少的递归目标实现。大多数实现其实只是命令中的 shell 代码。

### 项目 3：在包版本中使用仓库修订版本号

版本控制是每个项目中非常重要的一部分。它不仅保护知识产权，还允许开发者在经历了一系列错误后进行备份并重新开始。像 Git 和 Subversion 这样的版本控制系统的一个优势是，系统为每次更改分配一个唯一的修订版本号。这意味着任何项目源代码的发布都可以逻辑上与特定的仓库修订版本号相关联。本节介绍了一种技术，允许你将仓库修订版本号自动插入到包的 Autoconf 版本字符串中。

Autoconf 的`AC_INIT`宏的参数必须是静态文本。也就是说，它们不能是 shell 变量，Autoconf 会将尝试在这些参数中使用 shell 变量的行为标记为错误。除非你在配置过程中想要计算包版本号的某部分，否则这一点完全没有问题。

我曾经尝试在 `AC_INIT` 的 `VERSION` 参数中使用 shell 变量，以便在执行 `configure` 时将我的 Subversion 修订号替换到 `VERSION` 参数中。我花了几天时间试图弄清楚如何欺骗 Autoconf 让我在软件包的版本号中使用 shell 变量作为 *修订* 字段。最终，我发现了清单 18-14 中展示的技巧，并在我的 *configure.ac* 文件和顶层 *Makefile.am* 文件中实现了该技巧。

```
➊ SVNREV=`LC_ALL=C svnversion $srcdir 2>/dev/null`
➋ if ! svnversion || case $SVNREV in Unver*) true;; *) false;; esac;
  ➌ then SVNREV=`cat $srcdir/SVNREV`
  ➍ else echo $SVNREV>$srcdir/SVNREV
    fi
➎ AC_SUBST(SVNREV)
```

*清单 18-14:* configure.ac: *作为软件包版本的一部分实现动态修订号*

在这里，shell 变量 `SVNREV` 在 ➊ 处被设置为 `svnversion` 命令的输出，该命令在项目顶层目录中执行。输出是一个原始的 Subversion 修订号——也就是说，*如果*代码在一个真实的 Subversion 工作区中执行，但这并不总是如此。

当用户从分发归档中执行此 `configure` 脚本时，Subversion 可能甚至没有安装在他的工作站上。即使安装了，顶层项目目录也来自归档，而不是 Subversion 仓库。为了处理这些情况，➋ 处的行会检查 `svnversion` 是否能执行，或者第一行的输出是否以 *Unversioned directory* 这几个字母开头，这是在非工作区目录上执行 `svnversion` 工具时的结果。

如果这两种情况之一为真，`SVNREV` 变量将在 ➌ 处从名为 *SVNREV* 的文件的内容中填充。该项目应配置为随分发归档一起发送 *SVNREV* 文件，该归档包含了清单 18-14 中的配置代码。必须这样做，因为如果 `svnversion` 生成了一个真实的 Subversion 仓库修订号，该值会立即通过 ➍ 处的 `if` 语句的 `else` 子句写入 *SVNREV* 文件。

最后，➎ 处对 `AC_SUBST` 的调用将 `SVNREV` 变量替换到模板文件中，包括项目的 makefile 文件。

在顶层的 *Makefile.am* 文件中，我通过将 *SVNREV* 文件添加到 `EXTRA_DIST` 列表中，确保它成为分发归档的一部分。因此，当维护者创建并发布分发归档时，它将包含一个 *SVNREV* 文件，其中记录了用于从该源代码生成归档的源树修订号。*SVNREV* 文件中的值也会在从该 tarball 中的源代码生成归档时使用（通过 `make dist`）。这是准确的，因为原始归档实际上是从这个特定修订的 Subversion 仓库生成的。

通常来说，项目的分发归档能否生成正确的分发归档并不特别重要，但一个由 Automake 生成的归档可以做到这一点，而无需进行此修改，因此它在*有*此修改的情况下也应该能够做到。清单 18-15 突出了对顶层 *Makefile.am* 文件的相关更改。

```
EXTRA_DIST = SVNREV
distdir = $(PACKAGE)-$(VERSION).$(SVNREV)
```

*第 18-15 节:* Makefile.am: *配置为 SVN 修订号的顶层 makefile*

在 第 18-15 节 中，`distdir` 变量控制由 Automake 生成的分发目录名称和归档文件名。在顶层的 *Makefile.am* 文件中设置此变量会影响分发归档的生成，因为该 *Makefile.am* 文件是最终生成的 *Makefile* 中包含此功能的位置。

**注意**

*注意* SVNREV *文件名和*`SVNREV make`* 变量 *`[$(SVNREV)]`* 在 第 18-15 节 中的相似性。尽管它们看起来相同，添加到 *`EXTRA_DIST`* 行的文本指的是顶层项目目录中的 *SVNREV* 文件，而添加到 *`distdir`* 变量的文本则指向一个 *`make`* 变量。

对于大多数目的，在顶层 *Makefile.am* 文件中设置 `distdir` 应该足够。然而，如果你需要在项目中的另一个 *Makefile.am* 文件中正确格式化 `distdir`，只需在该文件中也设置它即可。

本项中介绍的技术并不会在你提交新更改时自动重新配置项目以生成新的 *SVNREV* 文件（因此更改构建中使用的 Subversion 修订号）。我本可以通过一些恰当的 `make` 规则添加此功能，但那样会迫使每次构建时检查是否有提交^(4)。

第 18-16 节 显示了与 第 18-14 节 中的代码类似的代码，区别在于此代码适用于 Git，而非 Subversion。

```
GITREV=`git -C $srcdir rev-parse --short HEAD`
if [ -z "$GITREV" ];
  then GITREV=`cat $srcdir/GITREV`
  else echo $GITREV>$srcdir/GITREV
fi
AC_SUBST(GITREV)
```

*第 18-16 节:* configure.ac: *实现一个 Git 动态修订号*

这个版本对我来说似乎更直观，因为 `git` 工具更好地利用了错误条件的正确输出通道——如果当前工作目录不是 Git 仓库，命令的输出会被发送到 `stderr`。

当然，你还应该修改 第 18-15 节 中的代码，引用 *GITREV* 文件而不是 *SVNREV* 文件。

如果你已经在使用 Gnulib，另一个很好的选择是使用该库中的 *version-gen* 模块。此模块提供了许多与将版本号融入构建过程相关的好功能。

### 第 4 项: 确保你的分发包是干净的

你是否曾经下载并解压了一个开源包，然后尝试运行 **`./configure && make`**，结果在这些步骤中的某一环节失败了？当你深入研究问题时，或许发现了归档中的缺失文件。可惜的是，这种情况发生在 Autotools 项目中，尽管 Autotools 使得确保这一点不发生变得非常简单。

为了确保你的分发归档始终干净和完整，请在新创建的归档上运行`distcheck`目标。不要满足于你*相信*你包的状态。让 Automake 来运行分发单元测试。我将这些测试称为*单元测试*，因为它们为分发包提供了与常规单元测试为源代码提供的相同测试功能。

你绝不会在没有运行单元测试的情况下进行代码更改并发布软件包，对吧？（如果是这样，那么你可以放心跳过这一部分。）同样，不要在没有运行构建系统单元测试的情况下发布归档——在发布新归档之前，先在你的项目上运行**`make distcheck`**。如果`distcheck`目标失败了，找出原因并修复它。这样做的回报是值得的。

### 第 5 项：黑客攻关 Autoconf 宏

有时你需要一个 Autoconf 并未完全提供的宏。这时候，知道如何复制和修改现有的 Autoconf 宏就很有帮助。^(5)

例如，以下是一个解决常见 Autoconf 邮件列表问题的方案。一个用户想使用`AC_CHECK_LIB`来捕获所需的库到`LIBS`变量中。问题在于这个库导出了 C++ 函数，而不是 C 函数。`AC_CHECK_LIB`在处理 C++ 时并不太友好，主要是因为`AC_CHECK_LIB`对符号的假设是基于 C 语言的链接方式，而这些假设并不适用于 C++ 符号。

例如，广为人知的（并且是标准化的）C 链接规则指出，导出的 C 链接符号（在 Intel 系统上也称为`cdecl`调用约定）是区分大小写的，并且会加上前导下划线，^(6)而使用 C++ 链接导出的符号则是通过非标准的、厂商定义的规则进行*改编*的。这些修饰符是基于函数签名——具体来说，是参数的数量和类型，以及函数所属的类和/或命名空间。但确切的规则并未在 C++ 标准中定义。

现在，停下来思考一下，在什么情况下你可能会有符号从库中导出，并使用 C++ 链接方式。导出 C++ 符号有两种方式。第一种是（无论是故意还是不小心）导出*全局*函数而没有在函数原型中使用`extern "C"`链接说明。第二种是导出整个类——包括公共和受保护的方法以及类数据。

如果你不小心忘记在全局函数中使用`extern "C"`，那么，请停止这种做法。如果你故意这么做，那我想知道为什么？我能想到的唯一理由是你想导出一个给定函数名的多个重载。这似乎是一个相当微不足道的理由，阻止你的 C 开发者使用你的库。

如果你导出的是类，那就是另一个问题了。在这种情况下，你专门为 C++ 用户提供支持，而这给`AC_CHECK_LIB`带来了一个实际问题。

Autoconf 为`AC_CHECK_LIB`的定义提供了一个框架，允许在 C 和 C++之间进行差异处理。如果在调用`AC_CHECK_LIB`之前使用`AC_LANG([C++])`宏，你将生成一个特定于 C++的测试程序版本。但不要抱太大希望；当前的 C++版本实现仅仅是 C 版本的复制。我预计设计一个通用的 C++实现充其量是非常困难的。

但并非一切都失去希望。虽然实现一个*通用*的版本可能很困难，但作为项目的维护者，你可以轻松地使用`AC_CHECK_LIB`的测试代码编写一个特定于项目的版本。

首先，我们需要找到`AC_CHECK_LIB`宏的定义。对 Autoconf 宏目录（通常是*/usr/(local/)share/autoconf/autoconf*）进行`grep`搜索，应该很快就能在名为*libs.m4*的文件中找到`AC_CHECK_LIB`的定义。因为大多数宏定义都以注释头开始，头部包含一个井号和宏的名称及一个空格，因此以下方法应该有效：

```
$ cd /usr/share/autoconf/autoconf
$ grep "^# AC_CHECK_LIB" *.m4
libs.m4:# AC_CHECK_LIB(LIBRARY, FUNCTION,
$
```

`AC_CHECK_LIB`的定义如清单 18-17 所示。^(7)

```
# AC_CHECK_LIB(LIBRARY, FUNCTION,
#             [ACTION-IF-FOUND], [ACTION-IF-NOT-FOUND],
#             [OTHER-LIBRARIES])
--snip--
# freedom.
AC_DEFUN([AC_CHECK_LIB],
[m4_ifval([$3], , [AH_CHECK_LIB([$1])])dnl
AS_LITERAL_IF([$1], [AS_VAR_PUSHDEF([ac_Lib], [ac_cv_lib_$1_$2])],
    [AS_VAR_PUSHDEF([ac_Lib], [ac_cv_lib_$1''_$2])])dnl
AC_CACHE_CHECK([for $2 in -l$1], [ac_Lib],
    [ac_check_lib_save_LIBS=$LIBS
    LIBS="-l$1 $5 $LIBS"
  ➊ AC_LINK_IFELSE([AC_LANG_CALL([], [$2])],
         [AS_VAR_SET([ac_Lib], [yes])],
         [AS_VAR_SET([ac_Lib], [no])])
     LIBS=$ac_check_lib_save_LIBS])
     AS_VAR_IF([ac_Lib], [yes],
         [m4_default([$3], [AC_DEFINE_UNQUOTED(AS_TR_CPP(HAVE_LIB$1))
         LIBS="-l$1 $LIBS"
     ])],
 [$4])dnl
AS_VAR_POPDEF([ac_Lib])dnl
])# AC_CHECK_LIB
```

*清单 18-17：`AC_CHECK_LIB`的定义，如在*libs.m4 中找到的

这个明显的难题通过一点分析就能轻松解决。宏似乎接受最多五个参数（如注释头部所示），其中前两个是必需的。突出显示的部分是宏的定义——我们将把它复制到我们的*configure.ac*文件中并修改，以便与我们的 C++导出功能兼容。

回想一下第十六章，M4 宏定义参数的占位符类似于 Shell 脚本的占位符：一个美元符号后跟一个数字。第一个参数用`$1`表示，第二个用`$2`表示，依此类推。我们需要确定哪些参数对我们重要，哪些可以忽略。我们知道，大多数对`AC_CHECK_LIB`的调用只传递前两个参数。第三和第四个参数是可选的，仅仅存在于你希望根据是否在指定的库中找到所需的函数来更改宏的默认行为时。第五个参数允许你提供一个额外的链接器命令行参数列表（通常是附加的库和库目录引用），这些是正确链接所需库的必要条件，以确保测试程序不会因为多余的原因失败。

假设我们有一个 C++库，导出了一个类的公共数据和方法。我们的库名为*fancy*，类名为`Fancy`，我们感兴趣的方法名为`execute`——特别是接受两个整数参数的`execute`方法。因此，它的签名将是

```
Fancy::execute(int, int)
```

当使用 C 链接导出时，这样的函数仅以`_execute`（或在某些平台上直接为`execute`，没有前导下划线）呈现给链接器，但当它使用 C++链接导出时，由于供应商特定的名称修饰，情况就复杂了。

让链接器找到这个符号的唯一方法是使用完全相同的签名在编译后的源代码中声明它，但我们没有向`AC_CHECK_LIB`提供足够的信息，以便在测试代码中正确声明函数签名。以下是所需的声明，告诉编译器如何正确地修饰此方法的名称：

```
class Fancy { public: void execute(int,int); };
```

假设我们正在寻找一个具有 C 连接性的名为`execute`的函数，`AC_CHECK_LIB`宏会生成一个像清单 18-18 中所示的小型测试程序。我已经突出显示了我们的函数名，以便你可以轻松看到宏如何将其插入到生成的测试代码中。

```
/* confdefs.h. */
#define PACKAGE_NAME ""
#define PACKAGE_TARNAME ""
#define PACKAGE_VERSION ""
#define PACKAGE_STRING ""
#define PACKAGE_BUGREPORT ""
/* end confdefs.h. */

/* Override any GCC internal prototype to avoid an error.
   Use char because int might match the return type of a GCC
   builtin and then its argument prototype would still apply. */
#ifdef __cplusplus
extern "C"
#endif

char execute ();
int
main ()
{
return execute ();
    ;
    return 0;
}
```

*清单 18-18：一个由 Autoconf 生成的检查全局 C 语言 `execute` 函数的示例*

除了这两处使用指定函数名的地方外，所有对`AC_CHECK_LIB`的调用生成的测试程序都是相同的。这个宏为所有函数创建一个通用的原型，以便所有函数都以相同的方式处理。然而，显然并不是所有函数都不接受参数并返回一个字符，正如这段代码中所定义的那样。`AC_CHECK_LIB`实际上是在向编译器谎报函数的真实性质。测试只关心测试程序是否能够成功链接；它永远不会尝试执行它（在大多数情况下，这一操作会以灾难性的方式失败）。

对于 C++ 符号，我们需要生成一个不同的测试程序——一个不对我们导出的符号的签名做任何假设的程序。

回顾一下清单 18-17 中的➊，看起来`AC_LANG_CALL`宏与清单 18-18 中测试代码的生成有关系，因为`AC_LANG_CALL`的输出会直接生成到对`AC_LINK_IFELSE`的调用的第一个参数中；其第一个参数是要用链接器测试的源代码。事实证明，这个宏也是另一个宏`AC_LANG_PROGRAM`的高级封装。清单 18-19 展示了这两个宏的定义。^(8)

```
   # AC_LANG_CALL(C)(PROLOGUE, FUNCTION)
   # -----------------------------------
   # Avoid conflicting decl of main.
   m4_define([AC_LANG_CALL(C)],
➊ [AC_LANG_PROGRAM([$1
 m4_if([$2], [main], ,
   [/* Override any GCC internal prototype to avoid an error.
       Use char because int might match the return type of a GCC
       builtin and then its argument prototype would still apply. */
   #ifdef __cplusplus
   extern "C"
   #endif
➋ char $2 ();])], [return $2 ();])])

   # AC_LANG_PROGRAM(C)([PROLOGUE], [BODY])
   # --------------------------------------
   m4_define([AC_LANG_PROGRAM(C)],
➌ [$1
   m4_ifdef([_AC_LANG_PROGRAM_C_F77_HOOKS], [_AC_LANG_PROGRAM_C_F77_HOOKS])[]dnl
   m4_ifdef([_AC_LANG_PROGRAM_C_FC_HOOKS], [_AC_LANG_PROGRAM_C_FC_HOOKS])[]dnl
   int
   main ()
   {
   dnl Do *not* indent the following line: there may be CPP directives.
   dnl Don't move the `;' right after for the same reason.
➍ $2
     ;
     return 0;
   }])
```

*清单 18-19：`AC_LANG_CALL`和`AC_LANG_PROGRAM`的定义*

在➊处，`AC_LANG_CALL(C)`会生成对`AC_LANG_PROGRAM`的调用，将`PROLOGUE`参数作为第一个参数传入。在➌处，这个序言（以`$1`的形式）会立即发送到输出流。如果传递给`AC_LANG_CALL(C)`的第二个参数（`FUNCTION`）不是`main`，则会为该函数生成一个 C 风格的函数原型。在➋处，文本`return $2 ();`作为`BODY`参数传递给`AC_LANG_PROGRAM`，该宏在 ➍ 处使用这段文本生成对函数的调用。（记住，这段代码只会被链接，而不会执行。）

对于 C++，我们需要能够定义更多的测试程序，以使其不对我们导出的符号的原型做任何假设，而`AC_LANG_CALL`对 C 来说过于具体，因此我们将使用更底层的宏`AC_LANG_PROGRAM`。示例 18-20 展示了如何改写`AC_CHECK_LIB`来处理来自名为*fancy*的库中的函数`Fancy::execute(int, int)`。我已在第 512 页的示例 18-17 中标出了我修改原始宏定义的地方。

```
   AC_PREREQ([2.59])
   AC_INIT([test], [1.0])

   AC_LANG([C++])

   # --- A modified version of AC_CHECK_LIB
   m4_ifval([], , [AH_CHECK_LIB([fancy])])dnl
➊ AS_VAR_PUSHDEF([ac_Lib], [ac_cv_lib_fancy_execute])dnl
➋ AC_CACHE_CHECK([whether -lfancy exports Fancy::execute(int,int)], [ac_Lib],
   [ac_check_lib_save_LIBS=$LIBS
 LIBS="-lfancy $LIBS"
➌ AC_LINK_IFELSE([AC_LANG_PROGRAM(
   [[class Fancy {
       public: void execute(int i, int j);
   };]],
   [[ MyClass test;
     test.execute(1, 1);]])],
     [AS_VAR_SET([ac_Lib], [yes])],
     [AS_VAR_SET([ac_Lib], [no])])
   LIBS=$ac_check_lib_save_LIBS])
   AS_VAR_IF([ac_Lib], [yes],
     [AC_DEFINE_UNQUOTED(AS_TR_CPP(HAVE_LIBFANCY))
     LIBS="-lfancy $LIBS"
   ],
   [])dnl
   AS_VAR_POPDEF([ac_Lib])dnl
   # --- End of modified version of AC_CHECK_LIB

   AC_OUTPUT
```

*示例 18-20：将修改版的`AC_CHECK_LIB`黑客化到 configure.ac 中*

在示例 18-20 中，我在➊和➋处分别用库名和函数名替换了参数占位符，并在➌处添加了由`AC_LANG_PROGRAM`生成的程序的前言和主体。我还删除了一些与`AC_CHECK_LIB`的可选参数相关的冗余文本，这些参数在我的版本中不需要。

这段代码比简单的`AC_CHECK_LIB`调用要长得多，理解起来也更困难，因此它迫切需要被转化为一个宏。我会把这个任务留给你作为练习。读完第十六章后，你应该能够轻松完成这项任务。还要注意，这个宏有很多优化的空间。随着你对 M4 的熟练度提升，你无疑会找到减少此宏大小和复杂性的方式，同时保持所需的功能。

#### *提供特定库的 Autoconf 宏*

这一条是关于在需要标准宏未提供的特殊功能时，修改 Autoconf 宏的内容。我使用的示例特别是关于在库中寻找特定函数的情况。这是一个更普遍问题的特殊案例：查找提供所需功能的库。

如果你是库的开发者，可以考虑提供可下载的 Autoconf 宏，用来测试库的存在性，或许还可以测试其中的特定版本功能。通过这样做，你可以使用户更容易确保他们的用户能够正确访问你的库。

这些宏不必具有通用性，因为它们是为特定库量身定制的。特定库的宏更容易编写，并且在测试库功能时可以更为彻底。作为作者，你更有可能理解库的各个版本的细微差别，因此你的宏可以准确地确定用户可能需要区分的库特征。

### 第 6 项：交叉编译

交叉编译发生在*构建系统*（即构建二进制文件的系统）和*主机系统*（即这些二进制文件将要运行的系统）类型不同时。例如，当我们在典型的 Intel x86 平台上运行 GNU/Linux 时为嵌入式系统构建 Motorola 68000 二进制文件，或者在运行 Solaris 的 DEC Alpha 系统上构建 Sparc 二进制文件时，我们就是在进行交叉编译。一个更常见的场景是使用 Linux 系统为旨在运行在嵌入式微控制器上的软件进行构建。

如果你正在构建的软件（如编译器或链接器）能够生成软件，情况会变得更加复杂。在这种情况下，*目标系统*代表了你的编译器或链接器最终将生成代码的系统。当这样的构建系统涉及三种不同的架构时，通常称之为*加拿大交叉编译*^(9)。在这种情况下，编译器或链接器是在架构 A 上构建的，运行在架构 B 上，并为架构 C 生成代码。另一种三系统构建类型，称为*交叉到本地*构建，涉及在架构 A 上构建编译器*以在*架构 B 上运行。在这种情况下，涉及三种架构，但主机架构和目标架构相同。一旦掌握了双系统交叉编译的概念，转向使用三系统交叉编译模式就相对简单了。

Autoconf 会生成配置脚本，试图猜测构建系统类型，并假定主机系统类型与其相同。除非通过命令行选项另行指定，否则`configure`假定处于非交叉编译模式。当没有指定构建或主机系统类型的命令行选项时，Autoconf 生成的配置脚本通常能够准确确定系统类型及其特征。

**注意**

*GNU Autoconf 手册*的第十四部分“手动配置”讨论了如何将 Autoconf 设置为交叉编译模式。不幸的是，编写适用于交叉编译的*configure.ac*文件所需的信息散布在手册中的各个部分。每个与交叉编译相关的宏都包含一段描述交叉编译模式对该宏影响的文字。可以通过搜索手册中的“cross-comp”来查找所有相关内容。

系统类型在*GNU Autoconf 手册*中通过一种包含 CPU、厂商和操作系统的三部分规范命名方式来定义，形式为`cpu-vendor-os`。但是，`os`部分本身可以是一个包含内核和系统类型的对（`kernel-system`）。如果你知道某个系统的规范名称，你可以在`configure`的三个参数中分别指定它，如下所示：

+   `--build=build-type`

+   `--host=host-type`

+   `--target=target-type`

这些`configure`命令行选项，配合正确的规范系统类型名称，允许你定义构建、主机和目标系统类型。（将主机系统类型定义为与你的构建系统类型相同是多余的，因为这是`configure`的默认情况。）

使用这些选项时，最具挑战性（也是文档最少）的一方面是确定在这些命令行选项中使用的适当规范系统名称。在*GNU Autoconf 手册*中，你不会找到任何告诉你如何构造一个合适规范名称的语句，因为规范名称并不是每种系统类型唯一的。例如，在大多数有效的交叉编译配置中，规范名称中的`vendor`部分会被忽略，因此可以设置为任何值。

当你在*configure.ac*文件中早期使用`AC_CANONICAL_SYSTEM`宏时，你会发现两个新的 Autoconf 助手脚本被添加到你的项目目录中（由`automake --add-missing`添加，`autoreconf --install`也会执行此操作）。具体来说，这些助手脚本是`config.guess`和`config.sub`。`config.guess`的任务是通过启发式方法确定用户系统的规范名称——构建系统。你可以自己执行此程序来确定适合自己构建系统的规范名称。例如，在我的 64 位 Intel GNU/Linux 系统上，运行`config.guess`会得到以下输出：

```
$ /usr/share/automake-1.15/config.guess
x86_64-pc-linux-gnu
$
```

如你所见，`config.guess`不需要命令行选项，尽管有一些可用选项。（使用`--help`选项可以查看它们。）它的任务是猜测你的系统类型，主要基于`uname`工具的输出。这个猜测会作为默认的系统类型，但用户可以在`configure`命令行中覆盖它。在交叉编译时，你可以在`--build`命令行选项中使用这个值。^(10)

`config.sub`程序的任务是接受一个输入字符串，作为你所寻找的系统类型的别名，然后将其转换为适当的 Autoconf 规范名称。那么，什么是有效的别名呢？你可以在`config.sub`中搜索“Decode aliases”以获取一些线索。你可能会在某段代码上方找到一个注释，解释它的任务是解码某些`CPU-COMPANY`组合的别名。以下是我系统中执行的一些示例，你应该在你的系统上找到相同的结果：

```
$ /usr/share/automake-1.15/config.sub i386
i386-pc-none
$ /usr/share/automake-1.15/config.sub i386-linux
i386-pc-linux-gnu
$ /usr/share/automake-1.15/config.sub m68k
m68k-unknown-none
$ /usr/share/automake-1.15/config.sub m68k-sun
m68k-sun-sunos4.1.1
$ /usr/share/automake-1.15/config.sub alpha
alpha-unknown-none
$ /usr/share/automake-1.15/config.sub alpha-dec
alpha-dec-ultrix4.2
$ /usr/share/automake-1.15/config.sub sparc
sparc-sun-sunos4.1.1
$ /usr/share/automake-1.15/config.sub sparc-sun
sparc-sun-sunos4.1.1
$ /usr/share/automake-1.15/config.sub mips
mips-unknown-elf
$
```

如你所见，单独的 CPU 名称通常不足以让`config.sub`正确地确定所需主机系统的有用规范名称。

另外需要注意的是，有一些通用的关键字，有时可以提供足够的信息进行交叉编译，而不需要提供真实的厂商或操作系统名称。例如，`unknown` 可以作为厂商名称的替代项，而 `none` 有时适用于操作系统名称。显然，`elf` 也是一个有效的系统名称，并且在某些情况下，对于 `configure` 来说，仅凭它就足以决定使用哪种工具链。然而，通过简单地将合适的厂商名称附加到 CPU 上，您可以让 `config.sub` 相当准确地推测出该 CPU 和厂商组合最可能的操作系统，从而生成有用的标准系统类型名称。

最终，确定合适的标准系统类型名称的最佳方法是检查 `config.sub`，找到与您认为应该用于 CPU 和厂商名称的内容相近的条目，然后直接询问它。虽然这看起来像是在瞎猜，但如果您已经进入编写需要交叉编译的程序构建系统的阶段，您可能已经非常熟悉主机 CPU、厂商和操作系统的名称了。

在进行交叉编译时，您很可能会使用一些不同于通常在系统中使用的工具，或者至少会在您的常用工具中添加额外的命令行选项。这些工具通常作为软件包一起安装。另一个确定合适主机系统标准名称的线索是这些工具名称的前缀。Autoconf 处理交叉编译的方式并没有什么神秘之处。主机系统标准名称直接用于在系统路径中通过名称定位正确的工具。因此，您使用的主机系统标准名称必须与工具名称的前缀匹配。

现在让我们来看一个常见场景：在相同 CPU 架构的 64 位机器上构建 32 位代码。从技术上讲，这是一种交叉编译形式，而且通常比为完全不同的机器架构进行交叉编译更简单。许多 GNU/Linux 系统支持 32 位和 64 位执行。在这些系统上，您通常可以使用构建系统的工具链，通过特定的命令行选项来执行此任务。例如，要在 64 位 Intel 系统上为 32 位 Intel 系统构建 C 源代码，您只需使用以下 `configure` 命令行（我已突出显示与交叉编译相关的行）：^(11)

```
   $ ./configure CPPFLAGS=-m32 LDFLAGS=-m32
   checking for a BSD-compatible install... /usr/bin/install -c
   checking whether build environment is sane... yes
   checking for a thread-safe mkdir -p... /bin/mkdir -p
   checking for gawk... gawk
   checking whether make sets $(MAKE)... yes
➊ checking build system type... x86_64-pc-linux-gnu
➋ checking host system type... x86_64-pc-linux-gnu
   checking for style of include used by make... GNU
   checking for gcc... gcc
   checking for C compiler default output file name... a.out
   checking whether the C compiler works... yes
➌ checking whether we are cross compiling... no
   checking for suffix of executables...
   checking for suffix of object files... o
   --snip--
```

注意 ➌，就 `configure` 来说，我们并没有进行交叉编译，因为我们没有给 `configure` 提供任何命令行选项，指示它使用不同于常规的工具链。如您在 ➊ 和 ➋ 中所看到的，构建系统和主机系统类型都是您期望的 64 位 GNU/Linux 系统类型。此外，由于我的系统是双模式系统，它可以执行使用这些标志编译的测试程序。它们将在 64 位 CPU 的 32 位模式下正常运行。

**注意**

*许多系统要求你在`gcc`能够识别`-m32`标志之前安装 32 位工具。例如，Fedora 系统要求安装`glibc-devel.i686`包，而我的 Linux Mint（基于 Ubuntu）系统则要求我安装`gcc-multilib`包。*

为了更确保在 Linux 系统上正确构建，你还可以使用`linux32`工具将你的 64 位系统的特性更改为 32 位系统，像这样：

```
$ linux32 ./configure CPPFLAGS=-m32 LDFLAGS=-m32
--snip--
checking whether we are cross compiling... no
--snip--
checking build system type... i686-pc-linux-gnu
checking host system type... i686-pc-linux-gnu
--snip--
```

我们在这里使用`linux32`，因为`configure`执行的一些子脚本可能会检查`uname -m`来确定构建机器的架构。`linux32`工具确保这些脚本能正确识别为 32 位 Linux 系统。你可以通过在`linux32`下运行`uname`来自己测试这一点：

```
$ uname -m
x86_64
$ linux32 uname -m
i686
$
```

为了让这种交叉编译在 Linux 双模式系统上工作，通常需要安装一个或多个 32 位开发包，如前所述。如果你的项目使用其他系统级服务，如图形桌面，那么你还需要这些库的 32 位版本。

现在让我们用更传统的方式（敢说是*标准*吗？）来做。我们不再将`-m32`添加到`CPPFLAGS`和`LDFLAGS`变量中，而是手动在`configure`命令行中设置构建和主机系统类型，然后看看会发生什么。再次说明，我已经突出显示了与交叉编译相关的输出行：

```
   $ ./configure --build=x86_64-pc-linux-gnu --host=i686-pc-linux-gnu
   checking for a BSD-compatible install... /usr/bin/install -c
   checking whether build environment is sane... yes
   checking for a thread-safe mkdir -p... /bin/mkdir -p
   checking for gawk... gawk
   checking whether make sets $(MAKE)... yes
➊ checking for i686-pc-linux-gnu-strip... no
   checking for strip... strip
➋ configure: WARNING: using cross tools not prefixed with host triplet
   checking build system type... x86_64-pc-linux-gnu
   checking host system type... i686-pc-linux-gnu
   checking for style of include used by make... GNU
➌ checking for i686-pc-linux-gnu-gcc... no
   checking for gcc... gcc
   checking for C compiler default output file name... a.out
   checking whether the C compiler works... yes
   checking whether we are cross compiling... yes
   checking for suffix of executables...
   checking for suffix of object files... o
   --snip--
```

这个示例中的几行关键内容表明，就`configure`而言，我们正在进行交叉编译。交叉编译的构建环境是`x86_64-pc-linux-gnu`，而主机系统是`i686-pc-linux-gnu`。

但是注意看➋处的`WARNING`文本。我的系统没有专门用于构建 32 位 Intel 二进制文件的工具链。这样的工具链包含了构建我的产品 64 位版本所需的所有相同工具，但 32 位版本的工具会以主机系统的标准系统名称作为前缀。如果你没有安装并在系统路径中可用的正确前缀工具链，`configure`将默认使用构建系统工具——那些没有前缀的工具。如果你的构建系统工具可以通过正确的命令行选项交叉编译到主机系统，并且你在`CPPFLAGS`和`LDFLAGS`中也指定了这些选项，那么这样是可以正常工作的。

通常，你需要安装一个设计用于构建正确类型二进制文件的工具链。在这个例子中，可以通过创建软链接和简单的 shell 脚本来提供这些工具的版本，脚本会传递额外所需的标志。根据➊和➌处`configure`脚本的输出，我需要提供`i686-pc-linux-gnu-`前缀版本的`strip`和`gcc`。

通常，这些外部工具链会安装到一个辅助目录中，这意味着你需要将该目录添加到你的系统 `PATH` 变量中，以便 `configure` 可以找到它们。对于这个例子，我将它们创建在*~/bin*中。^(12) 我再次强调了与跨平台编译相关的输出文本：

```
   $ ln -s /usr/bin/strip ~/bin/i686-pc-linux-gnu-strip
   $ echo '#!/bin/sh
   > gcc -m32 "$@"' > ~/bin/i686-pc-linux-gnu-gcc
   $ chmod +x ~/bin/i686-pc-linux-gnu-gcc
   $ ./configure --build=x86_64-pc-linux-gnu --host=i686-pc-linux-gnu
   checking for a BSD-compatible install... /usr/bin/install -c
   checking whether build environment is sane... yes
   checking for a thread-safe mkdir -p... /bin/mkdir -p
   checking for gawk... gawk
   checking whether make sets $(MAKE)... yes
   checking for i686-pc-linux-gnu-strip... i686-pc-linux-gnu-strip
   checking build system type... x86_64-pc-linux-gnu
   checking host system type... i686-pc-linux-gnu
   checking for style of include used by make... GNU
   checking for i686-pc-linux-gnu-gcc... i686-pc-linux-gnu-gcc
   checking for C compiler default output file name... a.out
   checking whether the C compiler works... yes
   checking whether we are cross compiling... yes
   checking for suffix of executables...
   checking for suffix of object files... o
   checking whether we are using the GNU C compiler... yes
   checking whether i686-pc-linux-gnu-gcc accepts -g... yes
   --snip--
   $ make
   --snip--
➊ libtool: compile: i686-pc-linux-gnu-gcc -DHAVE_CONFIG_H -I. -I.. -g -O2
    -MT print.lo -MD -MP -MF .deps/print.Tpo -c print.c -fPIC -DPIC -o
   --snip--
   $
```

这次，`configure` 能够找到正确的工具。注意，➊ 处的编译器命令不再包含 `-m32` 标志。它仍然存在，但已隐藏在 `i686-pc-linux-gnu-gcc` 脚本中。就 Autotools 而言，`i686-pc-linux-gnu-gcc` 已经知道如何在 64 位系统上构建 32 位二进制文件。

跨平台编译并不适合普通的终端用户。作为开源软件开发者，我们使用像 Autotools 这样的工具包，确保我们的终端用户在构建和安装我们的软件包时，不需要成为软件开发专家。但跨平台编译需要一定的系统配置，这超出了 Autotools 通常对终端用户的预期。此外，跨平台编译通常应用于一些专业领域，例如工具链或嵌入式系统开发。这些领域的终端用户通常*是*软件开发专家。

在一些地方，跨平台编译可以，并且可能应该，提供给普通终端用户。然而，我强烈建议你在你提供给用户的*README*和*INSTALL*文档中，详细并明确地说明这些指令。

### 第 7 项：模拟 Autoconf 文本替换技术

假设你的项目构建了一个在启动时通过配置文本文件中的值进行配置的守护进程。守护进程如何在启动时知道在哪里找到这个文件？一种方法是简单地假设它位于*/etc*目录中，但一个写得好的程序会允许用户在构建软件时配置该位置。系统配置目录的路径是一个可变位置，其值可以在 `configure`、`make all` 或 `make install` 命令行中指定，如下例所示：

```
$ ./configure --sysconfdir=/etc
--snip--
$ make all sysconfdir=/usr/mypkg/etc
--snip--
$ sudo make install sysconfdir=/usr/local/mypkg/etc
--snip--
```

所有这些例子都利用了 Autotools 构建系统提供的命令行功能，因此在创建项目和项目构建源文件时，必须仔细考虑它们。让我们看一些例子，来说明如何做到这一点。

现在，有些情况是根本无法工作的。例如，你不能在构建程序时通过 makefile 将系统配置目录路径传递到 C 源代码中，然后期望它在你更改配置文件安装位置并在 `make install` 命令行上运行时仍然正常工作。大多数终端用户不会在命令行上传递任何内容，但你仍然需要确保他们可以通过 `configure` 和 `make` 命令行设置前缀目录。

本条目重点讨论的是如何将命令行前缀变量覆盖信息放置到代码和安装数据文件的适当位置，以尽可能晚地在构建过程中进行处理。

Autoconf 会在配置时将 `AC_SUBST` 变量中的文本替换为那些变量在 `configure` 中定义的值，但它不会将文本替换为原始值。在一个 Autotools 项目中，如果你用特定的 `datadir` 执行 `configure`，你会得到如下结果：

```
   $ ./configure --datadir=/usr/share
   --snip--
   $ grep "datadir =" Makefile
   pkgdatadir = $(datadir)/b64
➊ datadir = /usr/share
   $
```

如➊所示，你可以看到 `configure` 中 shell 变量 `datadir` 的值会根据 `make` 变量 `datadir` 在 *Makefile* 中的命令行指令被精确替换。这里不显而易见的是，`datadir` 的默认值，无论是在 `configure` 脚本中，还是在替换后的 makefile 中，都是相对于构建系统中的其他变量的。通过不在 `configure` 命令行上覆盖 `datadir`，我们可以看到 makefile 中默认值包含了未展开的 shell 变量引用：

```
$ ./configure
--snip--
$ cat Makefile
--snip--
datadir = ${datarootdir}
datarootdir = ${prefix}/share
--snip--
prefix = /usr/local
--snip--
$
```

在第三章（参见示例 3-36）中，我们看到我们可以将命令行选项传递给预处理器，以允许我们在源代码中使用这些路径值。示例 18-21 通过在 `CPPFLAGS` 变量中传递 C 预处理器定义，展示了这一点，假设程序名为 `myprog`。^(13)

```
myprog_CPPFLAGS = -DSYSCONFDIR="\"@sysconfdir@\""
```

*示例 18-21：将前缀变量推送到 C 源代码中的* Makefile.am *或* Makefile.in

一个 C 源文件可能会包含示例 18-22 中显示的代码。

```
--snip--
#ifndef SYSCONFDIR
# define SYSCONFDIR "/etc"
#endif
--snip--
const char * sysconfdir = SYSCONFDIR;
--snip--
```

*示例 18-22：在 C 源代码中使用预处理器定义的变量*

Automake 对示例 18-21 中 *Makefile.am* 和 *Makefile.in* 之间的行没有做特殊处理，但 `configure` 脚本会将 *Makefile.in* 中的这一行转换为在示例 18-23 中显示的 *Makefile* 行。

```
myprog_CPPFLAGS = -DSYSCONFDIR="\"${prefix}/etc\""
```

*示例 18-23：`configure` 替换 `@sysconfdir@` 后的 Makefile 行*

当 `make` 在编译器命令行中传递这个选项时，它会解引用这些变量，从而生成以下输出命令行（这里只展示了部分）：

```
libtool: compile: gcc ... -DSYSCONFDIR=\"/usr/local/etc\" ...
```

这种方法有一些问题。首先，在 `configure` 和 `make` 之间，你会丢失 `sysconfdir` 变量的解析，因为 `configure` 用 `${prefix}`*/etc* 来替换 `@sysconfdir@`，而不是用 `${sysconfdir}`。问题在于，你不能再在 `make` 命令行上设置 `sysconfdir` 的值。为了解决这个问题，可以直接在 `CPPFLAGS` 变量中使用 `${sysconfdir} make` 变量，如示例 18-24 所示，而不是使用 Autoconf 的 `@sysconfdir@` 替代变量。

```
myprog_CPPFLAGS = -DSYSCONFDIR="\"${sysconfdir}\""
```

*示例 18-24：在 `CPPFLAGS` 中使用 `make` 变量，而不是 Autoconf 替代变量*

你可以使用这种方法在`configure`和`make`命令行上都指定`sysconfdir`的值。在`configure`命令行上设置变量会在*Makefile.in*（以及随后生成的*Makefile*）中定义一个默认值，然后可以在`make`命令行中覆盖这个值。

在`make all`和`make install`命令行上使用不同的值所带来的问题稍微微妙一些。考虑一下如果你做了以下操作会发生什么：

```
$ make sysconfdir=/usr/local/myprog/etc
--snip--
$ sudo make install sysconfdir=/etc
--snip--
$
```

在这里，你基本上是在欺骗编译器，当你告诉它你的配置文件将在构建期间安装到*/usr/local/myprog/etc*时。编译器会高兴地生成清单 18-22 中的代码，让它引用这个路径；然后第二个命令行会将你的配置文件安装到*/etc*，而你的程序将包含一个硬编码的错误路径。不幸的是，你几乎无法纠正这个问题，因为你允许用户在任何地方定义这些变量，并且*GNU 编码标准*指出`make install`不应该重新编译任何内容。

**注意**

*有时，构建和安装过程会故意指定不同的安装路径。回想一下在《将项目集成到 Linux 发行版》中关于*`DESTDIR`*的讨论（参见第 67 页），其中 RPM 包会在临时目录中构建和安装，以便之后将构建的产品打包成 RPM，并安装到正确的位置。*

尽管存在潜在的陷阱，但能够在`make`命令行上指定安装位置是一种强大的技术，然而这种方法仅在 makefile 中有效，因为它在 makefile 中的编译器命令行内 heavily 依赖于`make`变量替换。

如果你想替换一个已安装数据文件中的值，而这个数据文件没有经过`make`命令处理，你可以将数据文件转换为一个 Autoconf 模板，然后在该文件中简单地引用 Autoconf 替换变量。

事实上，我们在第十五章为 FLAIM 项目创建的*doxyfile.in*模板中做的就是这件事。然而，这只在 Doxygen 输入文件中有效，因为这些模板中使用的变量类始终由`configure`定义为完整的绝对或相对路径。也就是说，`@srcdir@`和`@top_srcdir@`的值不包含任何额外的 shell 变量。这些变量不是安装目录（前缀）变量，除了`prefix`本身，其他前缀变量总是相对于其他前缀变量定义的。

你也可以在 makefile 中*模拟*Autoconf 替换变量的过程，从而允许在已安装的数据文件中使用替换变量。清单 18-25 展示了一个模板，其中你可能希望用在构建过程中通常在标准前缀变量中找到的路径信息替换变量。

```
# Configuration file for myprog
logdir = @localstatedir@/log
--snip--
```

*清单 18-25：myprog 的配置文件模板示例，将被安装到`$(sysconfdir)`中*

这个模板用于程序配置文件，通常会安装在系统配置目录中。我们希望程序日志文件的位置，按照该配置文件中的指定，在安装时通过`@localstatedir@`的值来确定。不幸的是，`configure`会将这个变量替换成至少包含`${prefix}`的字符串，这在程序配置文件中并没有用处。清单 18-26 展示了一个*Makefile.am*文件，里面有一个额外的`make`脚本，通过对*myprog.cfg.in*中的变量进行替换，生成*myprog.cfg*文件。

```
   EXTRA_DIST = myprog.cfg.in
➊ sysconf_DATA = myprog.cfg

➋ edit = sed -e 's|@localstatedir[@]|$(localstatedir)|g'
➌ myprog.cfg: myprog.cfg.in Makefile
           $(edit) $(srcdir)/$@.in > $@.tmp
           mv $@.tmp $@

   CLEANFILES = myprog.cfg
```

*清单 18-26：在 makefile 中使用`sed`将`make`变量替换到数据文件中*

在这个*Makefile.am*文件中，我在➌定义了一个自定义的`make`目标，用来构建*myprog.cfg*数据文件。我还在➋定义了一个名为`edit`的`make`变量，这个变量解析为一个部分的`sed`命令，用来将模板文件(`$(srcdir)/`*myprog.cfg.in*)中的所有`@localstatedir@`替换为`$(localstatedir)`变量的值。由于`make`会递归地处理变量替换，直到所有的变量引用都被解析，因此以这种方式使用`make`可以确保最终输出中不会留下任何变量引用。在这个命令中，`sed`的输出被重定向到输出文件(*myprog.cfg*)。^(14)

这个示例中唯一不明显的代码是`sed`表达式中在尾随的`@`符号周围使用方括号，它们表示正则表达式语法，指示任何包含的字符都应该被匹配。由于只有一个包含的字符，这看起来像是一个无意义的复杂化，但这些方括号的目的是防止`configure`在对这个 makefile 进行 Autoconf 变量替换时，替换掉`edit`变量中的`@localstatedir@`。我们希望`make`使用这个变量，而不是`configure`。

我在➊将*myprog.cfg*分配给`sysconf_DATA`变量，以将这一新规则的执行与 Automake 提供的框架绑定。Automake 会在构建文件之后（如果需要）将其安装到系统配置目录中。

`DATA`主文件中的文件作为依赖项被添加到`all`目标中，经过内部的`all-am`目标。如果*myprog.cfg*不存在，`make`会查找构建它的规则。由于我有这样的规则，`make`会在我构建`all`目标时简单地执行这个规则。

我将模板文件名*myprog.cfg.in*添加到了 Listing 18-26 顶部的`EXTRA_DIST`变量中，因为 Autoconf 和 Automake 都不了解这个文件。另外，我将生成的文件*myprog.cfg*添加到了列表底部的`CLEANFILES`变量中，因为在 Automake 看来，*myprog.cfg*是一个分发数据文件，不应该被`make clean`自动删除。

**注意**

*这个例子展示了 Automake 不自动分发*`DATA`*主文件的一个充分理由。有时候这些文件是以这种方式构建的。如果自动分发构建数据文件，*`distcheck`*目标将失败，因为在构建之前，myprog.cfg 并不可以用于分发。*

在这个例子中，我将*myprog.cfg*的构建过程与安装过程结合，通过将其添加到`sysconf_DATA`变量中，然后我在*mydata.cfg.in*和*mydata.cfg*之间建立了一个依赖关系^(15)，以确保在执行`make all`时正确构建安装文件。你还可以通过使用适当的`-hook`或自定义目标，将其与标准或自定义构建或安装目标结合。

讨论这个话题时，如果不提到 Gnulib 的*configmake*模块，那就不完整。如果你已经在使用 Gnulib，并且需要做一些像我在这一项目中讨论的事情，可以考虑使用*configmake*，它会创建一个*configmake.h*头文件，可以被你的源文件包含，以便通过 C 预处理器宏访问所有标准目录变量。它仅对 C 代码有用，因此对于非 C 源代码的使用场景（比如需要引用前缀变量路径的已安装配置文件），你仍然需要我在这里展示的技巧。

### 项目 8：使用 Autoconf 档案项目

在第 511 页的“项目 5：黑客 Autoconf 宏”中，我演示了一种黑客 Autoconf 宏的技术，提供了接近但并不完全相同于原始宏功能的功能。当你需要一个 Autoconf 没有提供的宏时，你可以自己编写，或者寻找别人已经编写的。这一项目讲的是第二个选项，开始寻找的一个完美地方是 Autoconf 档案项目。

截至目前，Autoconf Archive 源项目托管在 GNU Savannah 上。^(16) 原始的 ac-archive 项目是两个较旧项目合并的结果：一个由 Guido Draheim（位于*[`ac-archive.sourceforge.net/`](http://ac-archive.sourceforge.net/)*)创建，另一个由 Peter Simon（位于*http://auto-archive.cryp.to*）创建。第一个网站至今仍在运行，尽管它显示了一个巨大的红色警告框，提示你应将更新提交到 GNU Autoconf 宏存档（位于 Savannah）；第二个网站已被关闭。这两个项目之间有很长的历史，也有不少邮件列表上的激烈争论。最终，每个项目都将对方的大部分内容纳入了自己的项目中，但 Peter Simon 的项目最终被迁移到了 Savannah 仓库，当前主页位于*[`www.gnu.org/software/autoconf-archive/`](https://www.gnu.org/software/autoconf-archive/)*。^(17)

存档中的价值在于，私有宏变为公开宏，而公开宏则被许多用户逐步改进。

截至目前，宏存档包含超过 500 个不随 Autoconf 分发的宏，包括在《正确使用线程》一文中讨论的`AX_PTHREAD`宏，见第 384 页。存档的最新版本可以从项目的 Savannah git 站点进行查看。该站点按类别、作者和开源许可证索引宏，允许你根据特定标准选择宏。你还可以通过名称搜索宏，或输入宏头部注释中可能出现的任何文本。

如果你发现自己需要一个 Autoconf 似乎没有提供的宏，可以查看 Autoconf Archive。

### 项目 9：使用增量安装技术

一些人要求`make install`足够智能，只安装那些尚未安装或比已安装版本更新的文件。

默认情况下，用户可以通过向`install-sh`传递`-C`命令行选项来使用此功能。最终用户可以通过在执行`make install`时使用以下语法直接启用此功能：

```
$ make install "INSTALL=/path/to/install-sh -C"
```

如果你认为你的用户会受益于此选项，可以考虑在项目随附的*INSTALL*文件中添加一些关于如何正确使用此功能的信息。你不觉得这种不需要你实现的功能很棒吗？

### 项目 10：使用生成的源代码

Automake 要求在项目的*Makefile.am*文件中静态定义所有源文件，但有时源文件的内容需要在构建时生成。

有两种方法可以处理项目中的生成源文件（更具体地说，是生成的头文件）。第一种方法是使用 Automake 提供的“拐杖”，供那些不关心`make`细节的开发人员使用。第二种方法是编写适当的依赖规则，让`make`理解源文件与产品之间的关系。我将首先讲解“拐杖”方法，然后我们再深入探讨如何在*Makefile.am*文件中进行适当的依赖管理。

#### *使用 BUILT_SOURCES 变量*

当你有一个作为构建过程一部分生成的头文件时，可以告诉 Automake 生成规则，确保在尝试构建产品之前总是先创建该文件。为此，请将头文件添加到 Automake 的`BUILT_SOURCES`变量中，如清单 18-27 所示。

```
bin_PROGRAMS = program
program_SOURCES = program.c program.h
nodist_program_SOURCES = generated.h
BUILT_SOURCES = generated.h
CLEANFILES = generated.h
generated.h: Makefile
        echo "#define generated 1" > $@
```

*清单 18-27：使用`BUILT_SOURCES`处理生成的源文件*

`nodist_program_SOURCES`变量确保 Automake 不会生成尝试分发此文件的规则；我们希望它在最终用户运行`make`时构建，而不是在分发包中提供。

如果没有用户提供的线索，Automake 生成的 makefile 无法知道*generated.h*的规则应该在编译*program.c*之前执行。我称`BUILT_SOURCES`为“拐杖”，因为它只是强制执行用于生成列出的文件的规则，并且仅在用户执行`all`或`check`目标时执行。即使直接尝试构建`program`目标，如果没有执行`BUILT_SOURCES`规则，也不会执行这些规则。话虽如此，让我们看看背后发生了什么。

#### *依赖管理*

在 C 或 C++项目中，源文件分为两类：一种是明确在 makefile 中定义为依赖项的文件，另一种是通过例如预处理器包含间接引用的文件。

你可以将所有这些依赖直接硬编码到你的 makefile 中。例如，如果*program.c*包含*program.h*，并且*program.h*包含*console.h*和*print.h*，那么*program.o*实际上依赖于所有这些文件，而不仅仅是*program.c*。然而，普通的手工编写的 makefile 只会显式定义*.c*文件与程序之间的关系。为了实现真正准确的构建，`make`需要通过类似清单 18-28 中所示的规则来了解所有这些关系。

```
   program: program.o
           $(CC) $(CFLAGS) $(LDFLAGS) -o $@ program.o

➊ program.o: program.c program.h console.h print.h
           $(CC) -c $(CPPFLAGS) $(CFLAGS) -o $@ program.c
```

*清单 18-28：描述文件之间完整关系的规则*

*program.o*与*program.c*之间的关系通常由一个*隐式*规则定义，因此清单 18-28 中➊的规则通常会被拆分为两个独立的规则，如清单 18-29 所示。

```
   program: program.o
           $(CC) $(CFLAGS) $(LDFLAGS) -o $@ program.o

➊ %.o: %.c
 $(CC) -c $(CPPFLAGS) $(CFLAGS) -o $@ $<

➋ program.o: program.h console.h print.h
```

*清单 18-29：C 源文件的隐式规则，定义为 GNU `make`模式规则*

在清单 18-29 中，GNU `make` 特定的 *模式规则* 在 ➊ 处告诉 `make` 关联的命令可以从一个以 *.c* 结尾、基名相同的文件生成一个以 *.o* 结尾的文件。^(18) 因此，每当 `make` 需要找到一个规则来生成作为某个规则依赖项列出的以 *.o* 结尾的文件时，它会搜索一个基名相同的 *.c* 文件。如果找到了，它会应用这个规则，从相应的 *.c* 文件重新构建 *.o* 文件，前提是 *.c* 文件的时间戳比现有的 *.o* 文件更新，或者 *.o* 文件丢失。

`make` 中有一套文档化的隐式模式规则，因此你通常不需要编写这样的规则。然而，你仍然需要以某种方式告诉 `make` *.o* 文件与任何包含的 *.h* 文件之间的间接^(19) 依赖关系。这些依赖关系不能仅通过内建规则来推断，因为这些文件之间没有基于文件命名约定的隐式关系，例如 *.c* 和 *.o* 文件之间的关系。这些关系是手动编码到源文件和头文件中的包含关系。

正如我在第三章中提到的，编写这样的规则是繁琐且容易出错的，因为在开发过程中（甚至在维护过程中，虽然程度较轻），源文件和头文件之间的种种关系可能随时发生变化，每次更改时都必须小心地更新规则以保持构建的准确性。C 预处理器更适合自动为你编写和维护这些规则。

##### 两遍系统

有两种方法可以使用预处理器来管理依赖关系。第一种方法是创建一个两遍系统，其中第一遍仅构建依赖关系，第二遍则根据这些依赖关系编译源代码。这是通过定义使用某些预处理器命令生成 `make` 依赖规则的规则来完成的，如清单 18-30 所示。^(20)

```
 program: program.o
           $(CC) $(CFLAGS) $(LDFLAGS) -o $@ program.o

   %.o: %.c
           $(CC) $(CPPFLAGS) -c $(CFLAGS) -o $@ $<

➊ %.d: %.c
           $(CC) -M $(CPPFLAGS) $< >$@

➋ sinclude program.d
```

*清单 18-30：直接构建自动依赖关系*

在清单 18-30 中，➊ 处的模式规则指定了与 *.d* 和 *.c* 文件之间的关系，正如清单 18-29 中 ➊ 处所示的 *.o* 和 *.c* 文件之间的关系。这里的 `sinclude` 语句在 ➋ 处告诉 `make` 包含另一个 makefile，并且 GNU `make` 足够聪明，不仅确保在分析主依赖关系图之前包含所有 makefile，还会查找构建它们的规则。^(21) 运行 `make` 时会产生如下输出：

```
   $ make
   cc -M program.c >program.d
   cc -c -o program.o program.c
   cc -o program program.o
   $
   $ cat program.d
   program.o: program.c /usr/include/stdio.h /usr/include/features.h \
   /usr/include/sys/cdefs.h /usr/include/bits/wordsize.h \
➊ --snip--
   /usr/include/bits/pthreadtypes.h /usr/include/alloca.h program.h \
   console.h print.h
   $
   $ touch console.h && make
   cc -c -o program.o program.c
   cc -o program program.o
   $
```

如你所见，生成*program.d*的规则首先被执行，因为`make`尝试包含该文件。➊位置省略的部分指的是在递归扫描包含的头文件集时遍历的许多系统头文件。该文件包含了类似于我们在列表 18-29 中的➋位置编写的依赖规则^(22)。 （我们手动编写的规则依赖列表中缺少对*program.c*的引用，因为它是多余的，尽管这样并不会造成问题。）从控制台示例中，你也可以看到，现在触碰这些包含的文件之一会正确地导致*program.c*源文件重新构建。

列表 18-30 中概述的机制存在的问题包括：整个源代码树必须被遍历两次：第一次是检查并可能生成依赖文件，然后再次编译任何修改过的源文件。

另一个问题是，如果一个头文件包含了另一个头文件，并且第二个头文件被修改了，目标文件会更新，但`make`包含的依赖文件却没有更新。下次修改第二级头文件时，既不会更新目标文件，也不会更新依赖文件。已删除的头文件也会引发问题：构建系统无法识别已删除的文件是被故意移除的，因此它会抱怨现有依赖关系中引用的文件丢失。

##### 一次性完成

处理自动依赖关系的更高效方式是将依赖文件作为编译的副作用生成。列表 18-31 展示了如何通过使用非便携的`-MMD` GNU 扩展编译器选项（在列表中突出显示）来完成这一操作。

```
   program: program.o
           $(CC) $(CFLAGS) $(LDFLAGS) -o $@ program.o

   %.o: %.c
➊         $(CC) -MMD $(CPPFLAGS) -c $(CFLAGS) -o $@ $<

➋ sinclude program.d
```

*列表 18-31：将生成依赖作为编译的副作用*

在这里，我已删除第二个模式规则（原本在列表 18-30 中的➊位置），并在列表 18-31 中的➊位置为编译器命令行添加了`-MMD`选项。此选项告诉预处理器生成一个与当前编译的*.c*文件具有相同基本名称的*.d*文件。当`make`在干净的工作区执行时，➋位置的`sinclude`语句会静默地失败，未能包含缺失的*program.d*文件，但这并不重要，因为所有的目标文件第一次都会被构建。在随后的增量构建过程中，之前构建的*program.d*文件会被包含，并且它的依赖规则会在这些构建中生效。

#### *正确构建的源文件*

上述描述的一次性方法大致是 Automake 在可能的情况下用来管理自动依赖的方式。这种方法的问题通常出现在处理生成的源文件时，包括*.c*文件和*.h*文件。例如，让我们扩展清单 18-31 中的示例，增加一个名为*generated.h*的生成的头文件，该文件被*program.h*包含。清单 18-32 展示了这一修改的第一次尝试。清单 18-31 中的新增内容在此清单中有所高亮。

```
program: program.o
        $(CC) $(CFLAGS) $(LDFLAGS) -o $@ program.o

%.o: %.c
        $(CC) -MMD $(CPPFLAGS) -c $(CFLAGS) -o $@ $<

generated.h: Makefile
        echo "#define generated" >$@

sinclude program.d
```

*清单 18-32：一个与生成的头文件依赖关系一起工作的 makefile*

在这种情况下，当我们执行`make`时，我们发现缺乏初始依赖文件对我们不利：

```
$ make
cc -MMD -c -o program.o program.c
In file included from program.c:4:
program.h:3:23: error: generated.h: No such file or directory
make: *** [program.o] Error 1
$
```

因为没有初始的二级依赖信息，`make`不知道它需要执行*generated.h*规则的命令，因为*generated.h*只依赖于*Makefile*，而*Makefile*没有变化。为了解决在*Makefile.am*文件中的这个问题，我们可以像在清单 18-27 中第 531 页那样，将*generated.h*列入`BUILT_SOURCES`变量中。这样会将*generated.h*添加为`all`和`check`目标的第一个依赖项，从而强制它们首先构建，以应对用户输入`make`、`make all`或`make check`的可能性。^(23)

处理这个问题的正确方法非常简单，并且在 makefile 和*Makefile.am*文件中每次都能奏效：在*program.o*和*generated.h*之间编写一个依赖规则，如清单 18-33 中更新后的 makefile 所示。高亮的行包含了额外的规则。

```
program: program.o
        $(CC) $(CFLAGS) $(LDFLAGS) -o $@ program.o

%.o: %.c
        $(CC) -MMD $(CPPFLAGS) -c $(CFLAGS) -o $@ $<

program.o: generated.h
generated.h: Makefile
        echo "#define generated" >$@

sinclude program.d
```

*清单 18-33：为生成的头文件添加硬编码依赖规则*

新的规则告知`make`*program.o*与*generated.h*之间的关系：

```
   $ make
   echo "#define generated" >generated.h
   cc -MMD -c -o program.o program.c
   cc -o program program.o
   $
   $ make
   make: 'program' is up-to-date.
   $
➊ $ touch generated.h && make
   cc -MMD -c -o program.o program.c
   cc -o program program.o
   $
➋ $ touch Makefile && make
   echo "#define generated" >generated.h
   cc -MMD -c -o program.o program.c
   cc -o program program.o
   $
```

在这里，修改*generated.h*（在➊处）会导致`program`被更新。修改*Makefile*（在➋处）会首先重新创建*generated.h*。

要在 Automake 的*Makefile.am*文件中实现清单 18-33 中显示的依赖规则，您将使用清单 18-34 中高亮的规则。

```
bin_PROGRAMS = program
program_SOURCES = program.c program.h
nodist_program_SOURCES = generated.h
program.$(OBJEXT): generated.h
CLEANFILES = generated.h
generated.h: Makefile
        echo "#define generated 1" > $@
```

*清单 18-34：用一个合适的依赖规则替换`BUILT_SOURCES`*

这与之前在清单 18-27 中第 531 页展示的代码完全相同，唯一的区别是我们将`BUILT_SOURCES`变量替换为了一个合适的依赖规则。此方法的优点在于它始终按预期工作；无论用户指定什么目标，*generated.h*都会在需要的时候被构建。^(24)

如果你尝试生成一个 C 源文件而不是头文件，你会发现其实你根本不需要额外的依赖规则，因为 *.o* 文件隐式地依赖于它们的 *.c* 文件。然而，你仍然必须在 `nodist_program_SOURCES` 变量中列出你生成的 *.c* 文件，以防止 Automake 尝试分发它。

**注意**

*当你定义自己的规则时，你会抑制 Automake 可能为该产品生成的任何规则。在特定的目标文件的情况下，这通常不会成为问题，但在定义规则时请记住这个 Automake 的特性。*

如你所见，正确管理生成的源文件所需要的仅仅是正确编写的依赖规则集以及适当的 `nodist_*_SOURCES` 变量。`make` 工具和 Autotools 提供了所需的框架，通过内建的 `make` 功能、宏和变量。你只需要将它们正确地组合在一起。例如，在 *GNU Automake 手册* 中，参见第 8.1.2 节，它讨论了程序链接。^(25) 本节提到 `EXTRA_prog_DEPENDENCIES` 变量，作为扩展 Automake 生成的特定目标的依赖图的一种机制。

### 项目 11：禁用不需要的目标

有时候，Autotools 为你做了太多的事情。以下是来自 Automake 邮件列表的一个例子：

我在我的一个项目中使用 automake 和 texinfo。该项目的文档充满了图片。正如你可能知道的，`` `make pdf' `` 会将 JPG 和 PNG 文件制作成 PDF 文档，而 `` `make dvi' `` 则需要 EPS 文件。然而，EPS 图像非常大（在这个案例中比 JPG 大 15 倍）。

问题是运行 `` `make distcheck' `` 会导致错误，因为应该存在的 EPS 图像并不存在，而 `` `make distcheck' `` 会尝试在所有地方运行 `make dvi'`。我希望能运行 `` `make pdf' ``，或者至少禁用构建 DVI。有办法做到吗？

首先是一些背景信息：Automake 的 `TEXINFOS` 主要使多个文档目标可供最终用户使用，包括 `info`、`dvi`、`ps`、`pdf` 和 `html`。它还提供了多个安装目标，包括 `install-info`、`install-dvi`、`install-ps`、`install-pdf` 和 `install-html`。在这些目标中，只有 `info` 会在执行 `make` 或 `make all` 时自动构建，只有 `install-info` 会在执行 `make install` 时执行。^(26)

然而，似乎 `distcheck` 目标也会构建至少 `dvi` 目标。刚才提到的问题是，海报没有提供构建 DVI 文档所需的封装 PostScript（EPS）图形文件，因此 `distcheck` 目标失败，因为它无法构建海报本不打算支持的文档。

要解决这个问题，你只需要提供你自己的版本的目标，它什么也不做，如清单 18-35 所示。

```
   --snip--
   info_TEXINFOS = zardoz.texi
➊ dvi: # do nothing for make dvi
```

*清单 18-35：在 Makefile.am 中禁用 `dvi` 目标，该文件指定了 `TEXINFOS` 主项*

在 ➊ 处添加一行代码后，`make distcheck` 又恢复了工作。现在，当它构建 `dvi` 目标时，它成功了，因为什么都没有做。

其他 Automake 主项也提供了多个附加目标。如果你只想支持这些目标中的一部分，你可以通过提供你自己的目标来有效地禁用不需要的目标。如果你希望在禁用重写时更加直言不讳，可以简单地包括一个 `echo` 语句，告诉用户你的包不提供 DVI 文档，但要小心不要执行任何可能失败的操作，否则用户会再次遇到相同的问题。

### 项目 12：注意制表符字符！

在过渡到 Automake 后，你不再使用原始的 makefile，因此为什么你还需要关注制表符字符呢？记住，*Makefile.am* 文件只是样式化的 makefile。最终，*Makefile.am* 文件中的每一行都会被 Automake 直接处理，然后转化为真正的 `make` 语法，或者直接复制到最终的 makefile 中。这意味着，在 *Makefile.am* 文件中，制表符字符是重要的。

参考这个来自 Automake 邮件列表的例子：

```
lib_LTLIBRARIES = libfoo.la
libfoo_la_SOURCES = foo.cpp
if WANT_BAR
  ➊ libfoo_la_SOURCES += a.cpp
else
  ➋ libfoo_la_SOURCES += b.cpp
endif

AM_CPPFLAGS = -I${top_srcdir}/include
libfoo_la_LDFLAGS = -version-info 0:0:0
```

我已经阅读了 autoconf 和 automake 手册，据我所见，上面的做法应该是可行的。然而，文件（a.cpp 或 b.cpp）[始终] 被添加到生成的 Makefile 的底部，因此在编译时未被使用。不管我尝试什么，我无法让上述代码生成正确的 makefile，但显然我做错了什么。

另一位发布者提供的答案简单而准确，尽管有些简洁到极点：

移除缩进。

这里的问题是，Automake 条件语句中的两行在 ➊ 和 ➋ 处用制表符字符进行了缩进。

你可能还记得在 第 380 页中“Automake 配置特性”部分，我讨论了 Automake 条件语句的实现，其中条件语句中的文本前缀是一个 Autoconf 替换变量，最终会转化为空字符串或哈希符号。这里的含义是，这些行在最终的 makefile 中基本上是保留原样或被注释掉。被注释的行对我们不重要，但你可以清楚地看到，如果 makefile 中未注释的行以制表符字符开头，Automake 会将它们视为命令，而不是定义，并将它们在最终的 makefile 中进行排序。当 `make` 处理生成的 makefile 时，它会试图将这些行解释为孤立命令。

**注意**

*如果原始发布者使用空格来缩进条件语句，就不会遇到问题了。*

故事的教训：注意制表符字符！

### 项目 13：打包选项

包维护者的最终目标是使最终用户的使用变得简单。系统级包通常没有这个问题，因为它们不依赖于任何不是操作系统核心部分的东西。但更高层次的包常常依赖于多个子包，其中有些比其他的更加普遍。

例如，考虑 Subversion 项目。如果你从 Subversion 项目网站下载最新的源代码包，你会发现它有两种版本。第一个包只包含 Subversion 源代码，但如果你解压并构建该项目，你会发现你需要下载并安装 Apache 运行时和运行时工具（*apr*和*apr-utils*）包、*zlib-devel*包和*sqlite-devel*包。这时，你可以构建 Subversion，但为了通过 HTTPS 启用对仓库的安全访问，你还需要*neon*或*serf*和*openssl*。

Subversion 项目的维护者认为，社区采用 Subversion 足够重要，值得不遗余力地推进。所以，为了帮助你构建一个功能完备的 Subversion 包，他们提供了一个名为*subversion-deps*的第二个包，其中包含了 Subversion 一些重要依赖的源代码分发包^(27)。只需将*subversion-deps*源包解压到与你解压*subversion*源包相同的目录中。*subversion-deps*包的根目录仅包含子目录——每个子目录对应一个源级别的依赖。

你可以选择以相同的方式将源代码包添加到你的项目构建系统中。当然，如果你使用的是 Automake，过程会简单得多。你只需要调用`AC_CONFIG_SUBDIRS`来为构建树中包含附加项目的子目录配置子项目。`AC_CONFIG_SUBDIRS`会默默忽略缺失的子项目目录。我在第十四章中向你展示了这个过程的一个例子，当 FLAIM 工具包作为任何更高层次 FLAIM 项目目录中的子目录存在时，我将它构建为一个子项目。

你应该随你的包一起发布哪些包？关键在于确定哪些包是你的消费者最不可能自行找到的。

### 总结

希望这些解决方案——实际上，这本书——对你在开源项目中创建出色用户体验的探索有所帮助。我在本书开头提到，人们常常因为不了解 Autotools 的目的而讨厌它们。到现在为止，你应该已经对这个目的有了相当清晰的认识。如果你之前不愿意使用 Autotools，那么希望我已经给你足够的理由去重新考虑。

回忆一下阿尔伯特·爱因斯坦那句常被误引的名言：“一切事物都应该尽可能简单，但不能过于简单。”^(28) 并不是所有事物都能简化到任何人都能在少量培训下掌握的程度。尤其是当涉及到那些旨在为他人简化生活的过程时，这一点尤为明显。Autotools 提供了一个让专家—程序员和软件工程师—能够让开源软件更加易于终端用户访问的能力。说实话，这个过程并不简单，但 Autotools 力图将其简化到尽可能的程度。
