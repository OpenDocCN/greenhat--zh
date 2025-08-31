## 第十一章：国际化

*像所有伟大的旅行者一样，我看到的比记得的多，记得的比看到的多。

—本杰明·迪斯雷利，《维维安·格雷》*

![Image](img/common.jpg)

当谈到将软件提供到其他语言时，母语为英语的人往往有些傲慢——但谁能怪我们呢？从几乎童年起，我们通过在这个行业中所经历的每一件事，都被教导英语是唯一重要的语言——以至于我们现在甚至不再去思考这个问题。所有计算机科学相关的研究和学术讨论，任何有影响力的社区都是用英语进行的。就连我们的编程语言都有英语关键词。

有人可能反驳说，这是因为大多数编程语言都是由英语使用者发明的。其实并不完全是这样！例如，来自瑞士的尼古拉斯·维尔特（Niklaus Wirth），一位德语为母语的人，他发明或参与了多个重要编程语言的发明，包括欧拉（Euler）、帕斯卡尔（Pascal）和莫杜拉（Modula）。觉得这些不够有名吗？丹麦出生的比雅尼·斯特劳斯特鲁普（Bjarne Stroustrup）发明了 C++。出生并成长于荷兰的吉多·范罗苏姆（Guido van Rossum）发明了 Python。出生在丹麦，后来移居加拿大的拉斯穆斯·勒多夫（Rasmus Lerdorf）编写了 PHP。Ruby 则是由日本的松本行弘（Yukihiro Matsumoto）编写的。

我的观点是，开发者——即使是非英语母语的开发者——从未考虑过发明使用德语关键词的编程语言。为什么不呢？可能是因为如果真的有人这么做，几乎没有人会使用它们——甚至连德国人也不会。新的编程语言往往是在学术或企业研究环境中构思出来的，促进这些发明的优缺点讨论的行业期刊、论坛和标准化组织几乎都是用英语书写或管理的——当然，这也是出于实际考虑。没有人在说英语是最好的语言。而是我们需要一个共同的媒介来发布信息，英语作为地球上最广泛使用的语言之一，就自然而然地扮演了这个角色。

由于这种只讲英语的态度，我们错过了一个重要的方面，那就是有整个非英语使用者的社区，他们在理解完全用英语编写的应用程序时感到困难。对他们来说，使用这些应用程序就像对只懂英语的人来说，去看一个中文或俄文网页一样不舒服。

大公司通常会提供语言包，以便这些社区能够在其母语中使用软件产品。部分商业本地化产品非常全面，甚至支持更为复杂的阿拉伯语和亚洲语言。^(1)然而，大多数较小的商业和开源软件包作者甚至都不尝试，因为他们认为成本过高、困难重重，或者这些对于他们的社区或市场来说并不重要。第一个论点*在企业界*可能有一定的道理。让我们讨论一下我们解决这些问题的选项，也就是如何扩大我们的社区。

### 强制性免责声明

在深入讨论这个话题之前，我先明确声明，关于软件国际化和本地化的多卷作品完全可以（也应该）被写出来。这个话题实在是太庞大了。我不可能在几章内容里涵盖所有内容。我的目标是为一个看起来可能令人生畏的主题提供一个介绍。如果你已经熟悉这些概念，可能会对我没有涉及的材料感到厌烦。请理解，这些章节并不是为你们准备的，虽然你们或许能从中找到一些有价值的想法。实际上，这些章节是为那些在这个领域经验较少的初学者准备的。

在这一章中，我将涵盖 C 标准中的内容以及与 UTF-8 编码集兼容的部分，还会稍微超出一些。我还将涵盖*GNU gettext*库的主要部分，因为将*gettext*集成到 Autotools 项目中实际上是本章的重点，但我不会涉及第三方库和解决方案，尽管在适当的地方我会提到它们。我也不会讨论宽字符字符串操作和多字节到宽字符（以及反之）的转换；有很多资源详细讨论了这些话题。

我刚才提到，我将涵盖*gettext*的*主要部分*，这意味着有些部分我将跳过，因为它们只在特定条件下使用。一旦你掌握了基础，随时可以从手册中获取其余内容。

说到手册，像许多软件手册一样，*gettext* 手册更多的是作为参考资料，而非为初学者设计的教程。你可能曾经尝试阅读 *gettext* 手册，打算通过这个渠道熟悉国际化和本地化，但读完后你可能会想，“要么这是一本糟糕的手册，要么我根本无法理解。”我曾经也有过这种感觉。如果是这样，你在某种程度上是对的。首先，很明显这本手册是由非英语母语的人编写的。难道这很奇怪吗？我们已经决定，一般来说，英语母语的人并不太关心这个话题。手册中使用的一些习语对英语使用者来说根本不熟悉，而且一些表达方式显然是外来的。不过，不谈出处，这本手册的组织结构也并不有利于那些想要熟悉这个话题的人。当我为这一章做研究时，我发现了几篇在线教程，它们对那些只想弄清楚从哪里入手的程序员来说，要比手册更有帮助。

那么，让我们首先从一些定义开始。

### 国际化 (I18n)

*国际化*，在文献中有时被称为 *i18n*，因为这样写更简便，2 是准备将软件包发布到其他语言或文化中的过程。这项准备工作包括以一种方式编写（或重构）软件，使其能够轻松配置以显示其他语言的人类可读文本，或者符合其他文化习俗和标准。我在这里提到的文本包括字符串、数字、日期和时间、货币值、邮政地址、称呼和问候、纸张大小、度量衡以及你能想到的任何其他在人类交流中可能会因语言和文化差异而有所不同的方面。

国际化特别强调*不是*将嵌入的文本从一种语言转换成另一种语言。而是通过为你的软件做准备，使得静态和生成的文本可以轻松地以目标语言显示，或者以符合目标文化规范的格式显示。例如，英国文化中的人们期望看到日期、小数数字和本地货币的显示方式与美国人不同，尽管两者都讲英语。所以国际化不仅仅包括语言支持，还包括一般的文化支持。

为了明确，这个准备工作不是为西班牙语使用者专门构建一个版本的应用程序。例如，这个话题留待 第十二章讨论，在那里我将讨论 *本地化* 的概念。而国际化是指设计或修改你的软件，使其 *能够* 被西班牙语使用者轻松使用。这意味着首先找到并标记出软件中应该翻译的字符串，找出代码中显示格式化时间、日期、货币、数字和其他区域特定内容的地方。然后，你需要使这些静态文本和文本生成代码可以根据全球或指定的区域设置进行配置。当然，这也意味着配置你的软件，使其能够识别当前的系统区域设置，并自动切换到该设置。

软件国际化有两个足够不同的领域，我们应该将它们分开讨论：

+   动态、运行时生成的文本消息

+   硬编码到应用中的静态文本消息

让我们先讨论生成的消息，因为在这方面，我们通常会从编程语言标准库中获得一些帮助。大多数此类库都提供某种形式的区域设置管理支持，C 语言也不例外。C++ 提供了同样的功能，只不过是面向对象的方式。^(3) 一旦你理解了 C 中可用的内容，C++ 版本就很容易自学，所以我们将在这里介绍 C 标准库提供的功能。

我还将向你介绍 POSIX 2008 和 X/Open 标准提供的扩展接口，因为正如我们将看到的，标准 C 库提供的功能虽然可以使用，但有些薄弱，而 POSIX 和 X/Open 标准的功能则相当广泛可用。最后，GNU 对 C 标准的扩展可以让你的应用在其他文化中脱颖而出，只要你愿意稍微偏离标准。

#### *为动态消息加 Instrumentation 的源代码*

标准 C 库提供了 `setlocale` 和 `localeconv` 函数，这些函数由 *locale.h* 头文件公开，如 清单 11-1 中所示。

```
#include <locale.h>

char *setlocale(int category, const char *locale);
struct lconv *localeconv(void);
```

*清单 11-1：标准 C 库 `setlocale` 和 `localeconv` 函数的概要*

`setlocale` 的任务是告诉标准 C 库在给定的库功能类别中使用哪个区域设置。此函数接受一个 *`category`*—一个枚举值，表示库中应从当前区域设置切换到新的目标 *`locale`* 的区域特定功能段。可用的标准类别枚举值如下。

**`LC_ALL`**

`LC_ALL` 代表所有类别。更改此类别的值将所有可用类别设置为指定的区域设置。这是最常见且推荐使用的值，除非你有非常具体的理由不将所有类别设置为相同的区域设置。

**`LC_COLLATE`**

更改`LC_COLLATE`会影响诸如`strcoll`和`strxfrm`等排序函数的工作方式。不同的语言和文化依据不同的字符或字形顺序规则进行排序。设置排序区域会改变库中排序函数使用的规则。

**`LC_CTYPE`**

更改`LC_CTYPE`会影响在*ctype.h*中定义的字符属性函数的工作方式（`isdigit`和`isxdigit`除外）。它还会影响这些函数的多字节和宽字符版本。

**`LC_MONETARY`**

更改`LC_MONETARY`会影响`localeconv`返回的货币格式信息（稍后在本节中讨论），以及由 X/Open 标准和 POSIX 扩展`strfmon`返回的结果字符串。

**`LC_NUMERIC`**

更改`LC_NUMERIC`会影响格式化输入和输出操作中使用的十进制点字符（如`printf`和`scanf`函数）以及由`localeconv`返回的与十进制格式相关的值，以及由 X/Open 标准和 POSIX 扩展`strfmon`返回的结果字符串。

**`LC_TIME`**

更改`LC_TIME`会影响`strftime`格式化时间和日期字符串的方式。

`setlocale`的返回值是一个表示先前区域设置的字符串，或者如果所有类别的区域设置不相同，则是一个区域设置集合。如果你只对确定当前区域设置感兴趣，可以在*`locale`*参数中传递`NULL`，这样`setlocale`就不会更改任何内容。如果你已将某些类别独立设置为不同的区域值，则在传递`LC_ALL`时返回的字符串格式是由实现定义的，因此不像预期的那样有用。尽管如此，大多数实现会允许你将这个字符串传回`setlocale`，并使用`LC_ALL`将类别特定的区域重置为先前获取的状态。

一旦设置了所需的区域设置，可以调用`localeconv`函数，返回一个指向结构的指针，结构中包含当前区域的一些属性。为什么不是所有的属性？因为这个 API 的设计者——按理说是聪明的人——在创建它时可能正在服用止痛药。说真的，*GNU C 库*手册对此有一些解释：

与`setlocale`函数一起，ISO C 的人发明了`localeconv`函数。这是一个设计极差的杰作。它使用起来代价高昂，无法扩展，并且由于它只能提供与`LC_MONETARY`和`LC_NUMERIC`相关的信息，因此通常不易使用。然而，如果在特定情况下适用，仍然应该使用它，因为它非常便捷。^(4)

除了这些批评外，我还要补充一点：它不是线程安全的；在你访问它的同时，结构体的内容可能会被另一个线程（通过调用`setlocale`）修改。不过，规则明确规定了它如何被修改——只有通过传递非`NULL`的*`locale`*参数值的`setlocale`调用，才会修改它——因此它是可以使用的，但既不优雅也不完整。正如前面的摘录所示，如果你的应用程序不需要额外的信息，你应该尽量使用`localeconv`，因为它是 C 标准的一部分，因此具有极高的可移植性。

公正地说，`localeconv`返回的结构体中的字段是那些需要程序员直接干预才能正确使用的字段，考虑到 C 标准库提供的功能。例如，`printf`系列函数没有为特定地区的数字和货币值提供特别的格式说明符，因此与`LC_NUMERIC`和`LC_MONETARY`类别相关的信息必须以某种方式提供给开发者，才能在设计用于以特定地区格式打印数字和货币金额的程序中正确使用这些类别。当然，这也意味着，没有第三方库或 C 标准扩展，你将不得不编写一些繁琐的文本格式化函数，根据`localeconv`返回的规则变化其输出。

另一方面，`LC_COLLATE`、`LC_TIME`和`LC_CTYPE`类别直接影响现有的标准库功能，因此程序员可能不需要直接访问这些库函数使用的地区信息属性。^(5)

##### 设置和使用地区信息

C 和 C++ 标准要求所有标准库的实现都必须在每个进程中初始化为默认的“C”地区信息，这样所有没有明确选择地区信息的程序将以可预测和一致的方式运行。因此，国际化软件的第一步就是改变地区信息。最简单且一致的方法是在程序开始时的某个地方调用`setlocale`，并将*`category`*值设置为`LC_ALL`。但是，我们应该传递什么字符串作为*`locale`*参数呢？这就是这个函数的妙处——你根本不需要传递任何特定的地区字符串。传递一个空字符串将禁用*默认地区信息*，允许库选择当前主机上有效的*环境地区信息*。这使得用户可以决定你的程序如何显示时间和日期、十进制数字和货币值，以及如何进行排序和字符集管理。

示例 11-2 显示了一个程序的代码，该程序配置标准 C 库以使用主机环境的区域设置，并将从 `localeconv` 获取的标准区域设置属性显示到控制台。

**注意**

*本章中的示例程序可以在名为* NSP-Autotools/gettext *的在线 GitHub 仓库中找到，地址为* [`github.com/NSP-Autotools/gettext/`](https://github.com/NSP-Autotools/gettext/)。* 本章中呈现的小型实用程序位于该仓库中的* small-utils *目录，并且提供了一个 makefile，默认情况下会构建它们。使用类似*`make lc`*的命令，例如，只构建 示例 11-2 中呈现的 *`lc`* 程序。*

Git 标签 11.0

```
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <string.h>
#include <limits.h>
#include <locale.h>

static void print_grouping(const char *prefix, const char *grouping)
{
    const char *cg;
    printf("%s", prefix);
    for (cg = grouping; *cg && *cg != CHAR_MAX; cg++)
        printf("%c %d", cg == grouping ? ':' : ',', *cg);
    printf("%s\n", *cg == 0 ? " (repeated)" : "");
}

static void print_monetary(bool p_cs_precedes, bool p_sep_by_space,
        bool n_cs_precedes, bool n_sep_by_space,
        int p_sign_posn, int n_sign_posn)
{
    static const char * const sp_str[] =
    {
        "surround symbol and quantity with parentheses",
        "before quantity and symbol",
        "after quantity and symbol",
        "right before symbol",
        "right after symbol"
    };
 printf("     Symbol comes %s a positive (or zero) amount\n",
             p_cs_precedes ? "BEFORE" : "AFTER");
    printf("     Symbol %s separated from a positive (or zero) amount by a space\n",
             p_sep_by_space ? "IS" : "is NOT");
    printf("     Symbol comes %s a negative amount\n",
             n_cs_precedes ? "BEFORE" : "AFTER");
    printf("     Symbol %s separated from a negative amount by a space\n",
             n_sep_by_space ? "IS" : "is NOT");
    printf("     Positive (or zero) amount sign position: %s\n",
             sp_str[p_sign_posn == CHAR_MAX? 4: p_sign_posn]);
    printf("     Negative amount sign position: %s\n",
             sp_str[n_sign_posn == CHAR_MAX? 4: n_sign_posn]);
}

int main(void)
{
    struct lconv *lc;
    char *isym;

    setlocale(LC_ALL, "");    // enable environment locale
    lc = localeconv();        // obtain locale attributes

    printf("Numeric:\n");
    printf("  Decimal point: [%s]\n", lc->decimal_point);
    printf("  Thousands separator: [%s]\n", lc->thousands_sep);

    print_grouping("    Grouping", lc->grouping);

    printf("\nMonetary:\n");
    printf("  Decimal point: [%s]\n", lc->mon_decimal_point);
    printf("  Thousands separator: [%s]\n", lc->mon_thousands_sep);

    print_grouping("    Grouping", lc->mon_grouping);

    printf("    Positive amount sign: [%s]\n", lc->positive_sign);
    printf("    Negative amount sign: [%s]\n", lc->negative_sign);
    printf("    Local:\n");
    printf("      Symbol: [%s]\n", lc->currency_symbol);
    printf("      Fractional digits: %d\n", (int)lc->frac_digits);

    print_monetary(lc->p_cs_precedes, lc->p_sep_by_space,
            lc->n_cs_precedes, lc->n_sep_by_space,
            lc->p_sign_posn, lc->n_sign_posn);

    printf("  International:\n");
    isym = lc->int_curr_symbol;
    printf("    Symbol (ISO 4217): [%3.3s], separator: [%s]\n",
            isym, strlen(isym) > 3 ? isym + 3 : "");
    printf("    Fractional digits: %d\n", (int)lc->int_frac_digits);

#ifdef __USE_ISOC99
    print_monetary(lc->int_p_cs_precedes, lc->int_p_sep_by_space,
            lc->int_n_cs_precedes, lc->int_n_sep_by_space,
            lc->int_p_sign_posn, lc->int_n_sign_posn);
#endif
    return 0;
}
```

*示例 11-2:* lc.c: *一个程序，用于显示从 `localeconv` 获取的所有区域设置属性*

`struct lconv` 结构包含 `char *` 和 `char` 字段。`char *` 字段大多是指字符串，其值由当前区域设置决定。一些 `char` 字段用于表示布尔值，而其他则设计为小整数值。示例代码 示例 11-2 应该清楚地指示哪些是布尔值，哪些是小整数值。您编译器的标准库文档也应该能明确说明这一点。

唯一奇怪的是 `grouping` 和 `mon_grouping` 字段，它们分别表示数字和货币值的分组方式，分组之间由相应的*千位分隔符*字符串分隔。`grouping` 和 `mon_grouping` 字段是 `char *` 类型的字段，设计时并非作为字符串读取，而是作为小整数数组读取。它们以零或 `CHAR_MAX`（在 *limits.h* 中定义）为终止符。如果它们以零终止，最后的分组值将永远重复；否则，最后的分组将包含值中剩余的数字。

最后，注意对内部 `print_monetary` 例程的调用，该调用被包装在一个 `__USE_ISOC99` 检查中（在示例底部附近）。这些货币属性的国际化形式是通过 C99 标准加入的。现在每个人都应该使用 C99，因此通常不成问题。我添加了条件编译检查，因为对于这个实用程序来说，这样做是可能且合适的。对于一个试图使用这些字段的应用程序，您应该要求 C99 标准是构建该应用程序的必要条件。

从美国英语的 Linux 系统构建并执行此程序会生成以下控制台输出：

```
$ gcc lc.c -o lc
$ ./lc
Numeric:
  Decimal point: [.]
  Thousands separator: [,]
  Grouping: 3, 3 (repeated)

Monetary:
  Decimal point: [.]
  Thousands separator: [,]
  Grouping: 3, 3 (repeated)
  Positive amount sign: []
  Negative amount sign: [-]
  Local:
    Symbol: [$]
    Fractional digits: 2
    Symbol comes BEFORE a positive (or zero) amount
    Symbol is NOT separated from a positive (or zero) amount by a space
    Symbol comes BEFORE a negative amount
 Symbol is NOT separated from a negative amount by a space
    Positive (or zero) amount sign position: before quantity and symbol
    Negative amount sign position: before quantity and symbol
  International:
    Symbol (ISO 4217): [USD], separator: [ ]
    Fractional digits: 2
    Symbol comes BEFORE a positive (or zero) amount
    Symbol IS separated from a positive amount by a space
    Symbol comes BEFORE a negative amount
    Symbol IS separated from a negative amount by a space
    Positive (or zero) amount sign position: before quantity and symbol
    Negative amount sign position: before quantity and symbol
$
```

要更改环境区域设置，请将 `LC_ALL` 环境变量设置为您想使用的区域设置名称。您可以使用的值是系统中生成并安装的区域设置。

**注意**

*你也可以使用与类别名称相同的环境变量来设置单独的 locale 类别。例如，要将 locale 更改为西班牙语（西班牙），但仅针对*`LC_TIME`*类别，你可以将*`LC_TIME`*环境变量设置为*`es_ES.utf8`*。这对所有前面定义的标准类别有效。^(6)*

要查看可用的 locale，运行`locale`工具并使用`-a`选项，如下所示：

```
$ locale -a
C
C.UTF-8
en_AG
en_AG.utf8
en_AU.utf8
en_BW.utf8
en_CA.utf8
en_DK.utf8
en_GB.utf8
en_HK.utf8
en_US.utf8
en_ZA.utf8
en_ZM
en_ZM.utf8
en_ZW.utf8
ja_JP.utf8
POSIX
sv_SE.utf8
$
```

**注意**

*我的示例控制台列表是在基于 Debian 的系统上执行的。如果你使用的是基于 Fedora 的发行版，例如，你应该预期会看到不同的结果，因为 Fedora 在安装语言包和*`locale`*工具的工作方式上有显著不同的默认功能。我将在本章稍后讨论与 Red Hat 相关的具体情况，只有在真正需要时才会涉及。*

通常，Linux 的美国英语安装会配置多个以`en`开头的 locale。我在我的基于 Debian 的系统上还生成了瑞典语（`sv_SE.utf8`）和日语（`ja_JP.utf8`）locale，以展示当环境配置为非英语语言和文化时输出的示例。

**注意**

*我在本章后面也使用了法语（*`fr_FR.utf8`*）locale。你可能希望通过你发行版提供的机制预先构建或预安装所有这些 locale，以便在你的系统上更容易跟随我的示例。当然，如果你不是以英语为母语的人，你可能已经默认使用了不同的 locale。在这种情况下，你可能还需要构建或安装*`en_US.utf8`* locale——尽管不出所料，即使在非美国制造或销售的系统上，这个 locale 通常也是预安装的。*

你可能已经注意到前面列表中的`C`、`C.UTF-8`和`POSIX` locales。正如之前提到的，`C` locale 是未显式设置 locale 的程序的默认 locale。`POSIX` locale 目前被定义为`C` locale 的别名。

##### 生成和安装 Locales

生成和安装 locale 的过程通常与发行版密切相关，但也有一些常见的实现方式。例如，在基于 Debian 或 Ubuntu 的系统上，你可以查看*/usr/share/i18n/SUPPORTED*文件，查看可以从系统上的源生成并安装的 locale：

```
$ cat /usr/share/i18n/SUPPORTED
aa_DJ.UTF-8 UTF-8
aa_DJ ISO-8859-1
aa_ER UTF-8
--snip--
zh_TW BIG5
zu_ZA.UTF-8 UTF-8
zu_ZA ISO-8859-1
$
```

在我的 Linux Mint 系统上，这个文件中有 480 个 locale 名称。locale 名称的一般格式，如 X/Open 标准所定义，如下所示：

```
language[_territory][.codeset][@modifier]
```

一个区域设置名称最多包含四个部分。第一部分，*`language`*，是必需的。其余部分，*`territory`*、*`codeset`* 和 *`modifier`*，是可选的。例如，使用 UTF-8 字符集的美国英语的区域设置名称是 `en_US.utf8`。*`language`* 以两位字母的 ISO 639 语言代码表示。^(7) 例如，`en` 指的是英语，可以是美式英语、加式英语、英式英语或其他英语方言。

*`territory`* 部分表示语言的地区，采用两位字母的 ISO 3166 国家代码表示。^(8) 例如，`US` 代表美国，`CA` 代表加拿大，`GB` 代表英国。

点（`.`）后的部分表示 *`codeset`* 或字符编码，格式为标准 ISO 字符编码名称，如 UTF-8 或 ISO-8859-1。^(9) 最常见的字符编码是 UTF-8（在区域设置名称中表示为 `utf8`），因为它可以表示世界上所有字符。然而，它并不是高效地表示所有字符；一些语言不使用 `utf8`，因为在这种编码中它们需要多个字节来表示每个字符。

*`modifier`* 部分并不常用。^(10) 其中一个可能的用途是生成一个仅在大小写敏感性或其他不是标准区域设置属性的属性上有所不同的区域设置。例如，当设置 `LC_MESSAGES=en@``boldquot` 时，你会得到一个英语消息集，区别在于引用的文本是加粗的。另一个历史上常见的例子是 `en_IE@eu``ro` 区域设置，仅通过使用不同的货币符号来区分。可以说，使用特定修改符的区域设置应用的差异是为非常特殊的用例设计的。

要在基于 Debian 或 Ubuntu 的系统上生成并安装特定的区域设置，你可以在 */var/lib/locales/supported.d* 目录下添加一个文件，文件中包含来自 *SUPPORTED* 的表示你要添加的区域设置的行。添加到 *supported.d* 目录中的文件名并不特别重要，尽管我建议不要使用与该目录结构中已有文件名称相差太远的名称。唯一重要的是该目录下存在一个文件，并且文件内容完全与 *SUPPORTED* 中的相应行一致。

例如，要添加 `sv_SE.utf8`，我会找到 *SUPPORTED* 中表示此语言的行，将该行添加到 *supported.d* 中的一个文件里，然后运行 `locale-gen` 程序，步骤如下：

```
$ cat /usr/share/i18n/SUPPORTED | grep sv_SE
sv_SE.UTF-8 UTF-8
sv_SE ISO-8859-1
sv_SE.ISO-8859-15 ISO-8859-15
$
$ echo "sv_SE.UTF-8 UTF-8" | sudo tee -a /var/lib/locales/supported.d/sv
[sudo] password for jcalcote: *****
sv_SE.UTF-8 UTF-8
$ sudo locale-gen
Generating locales (this might take a while)...
  en_AG.UTF-8... done
--snip--
  en_ZW.UTF-8... done
  a_JP.UTF-8... done
  sv_SE.UTF-8... done
Generation complete.
$
$ locale -a
C
C.UTF-8
en_AG
--snip--
ja_JP.utf8
POSIX
sv_SE.utf8
$
```

*SUPPORTED* 中的每一行包含一个语言环境数据库条目名称，后跟字符集名称。对于瑞典语，我们关注的条目是 `sv_SE.UTF-8`，字符集是 `UTF-8`。我选择添加一个名为 *sv* 的文件到 */var/lib /locales/supported.d* 中。你可以向文件中添加任意多的行；每一行将被作为单独的语言环境处理。由于 */var/lib/locale* 中的文件属于 root 用户，因此你需要具有 root 权限才能创建或写入它们。我使用了一个常见的小技巧，通过 `tee` 和 `echo` 命令将我想要的行添加到 *supported.d/sv* 中，作为 root 用户。^(11) 当然，你也可以直接使用带 `sudo` 的文本编辑器。

要在基于 Red Hat 或 CentOS 的系统上生成语言环境，你可以以这种方式使用 `localedef` 工具：

```
$ localedef --list-archive
aa_DJ
aa_DJ.iso88591
aa_DJ.utf8
--snip--
sv_SE.utf8
--snip--
zu_ZA
zu_ZA.iso88591
zu_ZA.utf8
$
$ sudo localedef -i sv_SE -f UTF-8 sv_SE.UTF-8
$
$ locale -a | grep sv_SE.utf8
sv_SE.utf8
$
```

`-i` 选项在 `localedef` 命令行中表示输入文件，该文件来自 `localedef --list-archive` 命令的输出。`-f` 选项表示使用的字符集。

**注意**

*我发现最近的 Red Hat（因此 CentOS）系统通常预装了许多语言环境。你可能会发现，通过使用 *`locale -a`*，你不需要生成任何语言环境。任何在 *`locale -a`* 中显示的内容都可以立即作为 *`LANG`* 和 *`LC_*`* 环境变量中的语言环境使用。而 Fedora 系统则需要安装特定语言的语言包，即使该语言环境已显示在 *`locale -a`* 列表中。例如，瑞典语需要安装 glibc-langpack-sv。此外，Fedora 上似乎没有安装语言源。因此，*`localedef`* 命令在该平台上无法使用，但安装语言包后会提供语言环境的预编译版本。*

现在我们已经可以使用瑞典语语言环境了，让我们看看当我们使用该语言环境时执行来自 Listing 11-2 中代码的 `lc` 程序时会显示什么：

```
$ LC_ALL=sv_SE.utf8 ./lc
Numeric:
  Decimal point: [,]
  Thousands separator: [ ]
  Grouping: 3, 3 (repeated)

Monetary:
  Decimal point: [,]
  Thousands separator: [ ]
  Grouping: 3, 3 (repeated)
  Positive amount sign: []
  Negative amount sign: [-]
  Local:
    Symbol: [kr]
    Fractional digits: 2
    Symbol comes AFTER a positive (or zero) amount
    Symbol IS separated from positive (or zero) amount by a space
    Symbol comes AFTER a negative value
    Symbol IS separated from negative value by a space
    Positive (or zero) amount sign position: before quantity and symbol
    Negative amount sign position: before quantity and symbol
  International:
    Symbol (ISO 4217): [SEK], separator: [ ]
    Fractional digits: 2
    Symbol comes AFTER a positive value
    Symbol IS separated from positive value by a space
    Symbol comes AFTER a negative value
    Symbol IS separated from negative value by a space
    Positive (or zero) amount sign position: before quantity and symbol
    Negative amount sign position: before quantity and symbol
$
```

不幸的是，正如我之前提到的，`localeconv` 只返回关于数字（`LC_NUMERIC`）和货币（`LC_MONETARY`）类别的信息，虽然听起来有点糟糕，但实际上其他类别几乎由库自动处理。无论如何，还有其他方式可以访问完整的语言环境属性，我们将在本章后面讨论。

##### 格式化时间和日期以供显示

标准 C 库在后台安静地处理时间和日期，具体取决于你在传递给 `strftime` 的格式字符串中使用的格式说明符。以下是 `strftime` 的原型：

```
#include <time.h>

size_t strftime(char *s, size_t max, const char *format, const struct tm *tm);
```

简单来说，`strftime`函数将最多*`max`*字节放入由*`s`*指向的缓冲区。内容由*`format`*中的文本和格式说明符决定。在*`format`*中只能指定一个时间值格式，其值从*`tm`*中获得。由于这是一个标准库函数，你可以参考任何标准 C 库手册，了解格式说明符在此函数中的使用方式。

清单 11-3 提供了一个小程序的源代码，该程序以某种形式输出当前时间和日期，这种格式被所有语言和地区支持。^(12)

```
#include <stdio.h>
#include <locale.h>
#include <time.h>

int main(void)
{
    time_t t = time(0);
    char buf[128];

    setlocale(LC_ALL, "");  // enable environmental locale

    strftime(buf, sizeof buf, "%c", gmtime(&t));
    printf("Calendar time: %s\n", buf);
    return 0;
}
```

*清单 11-3:* td.c: *一个小程序，用于在环境语言环境中打印日历日期和时间*

构建并执行这个程序会在控制台显示类似以下的输出；你的时间和日期可能与我的不同：

```
$ gcc td.c -o td
$ LC_ALL=C ./td
Calendar time: Tue Jul    2 03:57:56 2019
$ ./td
Calendar time: Tue 02 Jul 2019 03:57:58 AM GMT
$ LC_ALL=sv_SE.utf8 ./td
Calendar time: tis    2 jul 2019 03:57:59
$
```

我在第一次执行时设置了`LC_ALL=C`，以展示如何使用默认的 C 语言环境执行本地化程序。这对于测试你的国际化软件来说是一个很有用的调试工具。

**注意**

*C 语言环境并不是“美国”语言环境。它被称为最简语言环境。如果你使用*`LC_ALL=C`*执行*`lc`*程序，你会发现许多选项为空。标准库期望并以适当方式处理这些空选项。*

比较英文和瑞典文输出。日期和月份名称使用的是当地语言。对于七月，英文和瑞典文的月份名称恰好是一样的。然而，注意到日期和月份名称的大小写差异。在英文中，名称首字母大写，而在瑞典文中则不是。另一个区别是英文使用 12 小时制的 AM/PM 时间格式，而瑞典文使用 24 小时制时间格式。瑞典和 C 语言省略了日期前的零，而美国语言环境则没有。最后，美国时间后会跟随格林威治标准时间区（`GMT`）的名称，而瑞典只有一个时区——中欧时间（`CET`）——这一点反映在瑞典标准时间和日期格式的简洁性上。

所有这些差异都由环境语言环境定义，但快速浏览清单 11-3 中的代码，可以看到我仅在调用`strftime`时使用了`%c`格式说明符。有效的语言环境使得这个格式说明符根据具体语言环境输出通用的时间和日期信息。

但是，并非所有`strftime`接受的格式说明符都是如此有用。例如，使用像`"%X %D"`这样的格式字符串看似是一个不错的方法，但在所有区域设置中，它并不能产生正确的结果。`%X`说明符以特定区域设置的方式格式化时间，但`%D`则以非常美国英语的方式格式化日期。此外，完整的时间日期字符串在不同的区域设置中会有所不同，时间和日期部分的顺序也会不同。在本章后面，我将向你展示如何使用`nl_langinfo`来解决这些问题。

##### 排序和字符类

现在让我们考虑那些不那么显而易见的类别——那些不会在`struct lconv`中返回的信息：`LC_COLLATE`和`LC_CTYPE`。

`LC_COLLATE`影响`strcoll`和`strxfrm`函数的工作方式。对于英语使用者来说，这些函数的内部运作更难以理解，因为在英语中，区域设置特定的字符比较恰好与它们在*ASCII*表中的字典顺序一致。

**注意**

*原始的* 美国信息交换标准代码（ASCII） *由* 美国标准协会（ASA） *于 1963 年发明。* *最初，它只包括美国英语的大写字母和数字。1967 年，它被修订为包括控制字符和小写字母。由于标准将代码长度限制为 7 位，它只包含 128 个字符，使用 0 到 127 的代码。这个 7 位限制是因为每个字节的第八位通常用于数据传输中的错误校正。1981 年，IBM 将 ASCII 代码纳入一个 8 位、256 字符的代码的下半部分，并将其命名为*代码 页面 437*，并将这个代码纳入了其 IBM PC 系列个人计算机的固件中。在本章中，当我提到*ASCII 表*时，实际上是指*代码 页面 437。* *从技术上讲，ASCII 仍然只限于 128 个字符。*

许多其他语言并非如此。例如，在英语和西班牙语中，带重音的元音会正确地排在其没有重音的对应元音之后，而在日语中，既没有元音也没有重音元音，因此它们按 ASCII 表中的序号值进行排序。由于所有带重音的元音位于 ASCII 表的上半部分，而所有没有重音的元音位于下半部分，因此应该很清楚，使用英语或西班牙语区域设置时，西班牙单词列表的排序顺序与任何基于不包含拉丁字母的语言的区域设置不同。

清单 11-4 包含一个简短的程序，使用 C 语言的`qsort`函数通过不同的比较例程对西班牙语单词列表进行排序。

```
#include <stdio.h>
#include <stdlib.h>
#include <locale.h>
#include <string.h>

#define ECOUNT(x) (sizeof(x)/sizeof(*(x)))

int lex_count = 0;
int loc_count = 0;

static int compare_lex(const void *a, const void *b)
{
    lex_count++;
    return strcmp(*(const char **)a, *(const char **)b);
}
static int compare_loc(const void *a, const void *b)
{
    loc_count++;
    return strcoll(*(const char **)a, *(const char **)b);
}

static void print_list(const char * const *list, size_t sz)
{
    for (int i = 0; i < sz; i++)
        printf("%s%s", i ? ", " : "", list[i]);
    printf("\n");
}

int main()
{
    const char *words[] = {"rana", "rastrillo", "radio", "rápido", "ráfaga"};

    setlocale(LC_ALL, "");    // enable environment locale

    printf("Unsorted                : ");
    print_list(words, ECOUNT(words));

    qsort(words, ECOUNT(words), sizeof *words, &compare_lex);

    printf("Lex (strcmp)        : ");
    print_list(words, ECOUNT(words));

    qsort(words, ECOUNT(words), sizeof *words, &compare_loc);

    printf("Locale (strcoll): ");
    print_list(words, ECOUNT(words));

    return 0;
}
```

*清单 11-4:* sc.c: *一个简短的程序，演示了不同区域设置下的排序顺序差异*

首先，将未排序的`words`列表打印到控制台；接着，使用`compare_lex`函数通过`qsort`对`words`列表中的指针进行排序，`compare_lex`函数使用`strcmp`来确定每对单词中字母的排序顺序。`strcmp`函数不了解任何区域设置，它只是使用单词中字母在 ASCII 表中的顺序。然后，将排序后的列表打印到控制台。

接下来，再次对`words`调用`qsort`——这次使用`compare_loc`，该函数使用`strcoll`来确定单词对的排序顺序。`strcoll`函数使用当前区域设置来确定被比较单词中字母的相对顺序。然后，将重新排序的列表打印到控制台。

使用不同的区域设置构建并执行该程序会显示以下输出：

```
$ gcc sc.c -o sc
$ ./sc
Unsorted        : rana, rastrillo, radio, rápido, ráfaga,
Lex (strcmp)    : radio, rana, rastrillo, ráfaga, rápido,
Locale (strcoll): radio, ráfaga, rana, rápido, rastrillo,
$ LC_ALL=es_ES.utf8 ./sc
Unsorted        : rana, rastrillo, radio, rápido, ráfaga,
Lex (strcmp)    : radio, rana, rastrillo, ráfaga, rápido,
Locale (strcoll): radio, ráfaga, rana, rápido, rastrillo,
$ LC_ALL=ja_JP.utf8 ./sc
Unsorted        : rana, rastrillo, radio, rápido, ráfaga,
Lex (strcmp)    : radio, rana, rastrillo, ráfaga, rápido,
Locale (strcoll): radio, rana, rastrillo, ráfaga, rápido,
$
```

英语和西班牙语的重音元音排序方式相同。`C`区域设置，由使用`strcmp`获得的结果表示，始终严格按照 ASCII 表排序。然而，日语的排序方式与拉丁语言不同，因为日语没有假设如何排序其字母表中未包含的字符（无论是否带有重音）。

在内部，`strcoll`使用一种算法将比较字符串中的字符转换为数字值，这些数字值在当前区域设置下自然排序；然后，它使用`strcmp`函数比较这些字节数组。`strcoll`使用的算法可能相当复杂，因为对于每一对它比较的字符串，它会将这些字符串对中的区域特定的多字节字符序列转换为可以按字典顺序比较的字节序列，通过字符集的顺序值，然后使用`strcmp`内部比较这些字节序列。

如果你知道自己将比较相同的字符串或字符串集，那么先使用`strxfrm`函数可能会更高效，它暴露了`strcoll`在内部使用的转换算法。然后，你可以简单地对这些转换后的字符串使用`strcmp`，以获得与对未转换字符串使用`strcoll`时相同的排序结果。

Listing 11-5 通过将 Listing 11-4 中的内容转换为使用`strxfrm`处理`words`数组中的单词，演示了这一过程，并将转换后的单词写入一个足够大的二维数组，以容纳转换后的字符串。

```
   #include <stdio.h>
   #include <stdlib.h>
   #include <locale.h>
   #include <string.h>

   #define ECOUNT(x) (sizeof(x)/sizeof(*(x)))

➊ typedef struct element
   {
       const char *input;
 const char *xfrmd;
   } element;

   static int compare(const void *a, const void *b)
   {
       const element *e1 = a;
       const element *e2 = b;
    ➋ return strcmp(e1->xfrmd, e2->xfrmd);
   }

   static void print_list(const element *list, size_t sz)
   {
       for (int i = 0; i < sz; i++)
        ➌ printf("%s, ", list[i].input);
       printf("\n");
   }

   int main()
   {
       element words[] =
       {
           {"rana"}, {"rastrillo"}, {"radio"}, {"rápido"}, {"ráfaga"}
       };

       setlocale(LC_ALL, "");   // enable environment locale

       // point each xfrmd field at corresponding input field
       for (int i = 0; i < ECOUNT(words); i++)
        ➍ words[i].xfrmd = words[i].input;

       printf("Unsorted            : ");
       print_list(words, ECOUNT(words));

       qsort(words, ECOUNT(words), sizeof *words, &compare);

       printf("Lex (strcmp)        : ");
       print_list(words, ECOUNT(words));

       for (int i = 0; i < ECOUNT(words); i++)
       {
           char buf[128];
           strxfrm(buf, words[i].input, sizeof buf);
        ➎ words[i].xfrmd = strdup(buf);
       }

       qsort(words, ECOUNT(words), sizeof *words, &compare);

       printf("Locale (strxfrm/cmp): ");
       print_list(words, ECOUNT(words));

       return 0;
}
```

*Listing 11-5:* sx.c: *重写后的`sc`程序，使用`strxfrm`*

这里有几个需要注意的地方。`strxfrm`函数返回一个以零终止的字节缓冲区，它看起来和行为像一个普通的 C 字符串。里面没有内置的空字符；它可以被标准 C 库中的其他字符串函数操作，但从人类可读性的角度来看，它不一定是易于理解的。由于这种奇怪的特性，转换缓冲区的内容只能在排序时用于比较目的。原始输入值必须用于显示。因此，我们需要跟踪并作为对进行排序，输入缓冲区和转换缓冲区的每对单词。`element`结构在 ➊ 处为我们管理这一点。

由于我们不再需要使用`strcoll`，我已经移除了`compare_loc`函数，并将`compare_lex`重命名为`compare`，同时修改了代码以比较传入的`element`结构中的`xfrmd`字段（在 ➋ 处）。但是需要注意的是，`print_list`函数仍然打印元素的`input`字段（在 ➌ 处）。之所以可行，是因为`words`数组已被转换为一个对数组，其中每个元素包含原始和转换后的单词。

为了使这段代码与*sc.c*中原始的`main`流程兼容，在设置本地化后，*sx.c*会遍历`words`（在 ➍ 处），将每个元素的`xfrmd`指针设置为与其`input`指针相同的值。这让我们能够看到在第一次调用`qsort`时使用`strcmp`对未转换字符串进行比较时发生了什么。

在 ➎ 处，在打印完第一次排序操作的结果后，程序再次遍历`words`，这次对每个输入字符串调用`strxfrm`，并将相应的`xfrmd`字段指向转换缓冲区`buf`的`strdup`副本。^(13)

构建并执行示例 11-5 中的代码应该能显示出与我们运行示例 11-4 中的代码时相同的输出：

```
$ gcc sx.c -o sx
$ ./sx
Unsorted            : rana, rastrillo, radio, rápido, ráfaga,
Lex (strcmp)        : radio, rana, rastrillo, ráfaga, rápido,
Locale (strxfrm/cmp): radio, ráfaga, rana, rápido, rastrillo,
$ LC_ALL=es_ES.utf8 ./sx
Unsorted            : rana, rastrillo, radio, rápido, ráfaga,
Lex (strcmp)        : radio, rana, rastrillo, ráfaga, rápido,
Locale (strxfrm/cmp): radio, ráfaga, rana, rápido, rastrillo,
$ LC_ALL=ja_JP.utf8 ./sx
Unsorted            : rana, rastrillo, radio, rápido, ráfaga,
Lex (strcmp)        : radio, rana, rastrillo, ráfaga, rápido,
Locale (strxfrm/cmp): radio, rana, rastrillo, ráfaga, rápido,
$
```

它稍微复杂一点——这种版本的价值在排序五个单词时不立即显现出来，但在排序数百个字符串时，尽管有分配和释放转换缓冲区的开销，相比在`strcoll`内转换字符串，节省的时间是显著的。

**注意**

*此示例通过捷径突出显示*`strxfrm`*的重要点。一个真实的程序会检查*`strxfrm`*的结果，它返回转换所需的字节数（不包括终止的空字符）。如果返回值大于指定的缓冲区大小，程序应该重新分配内存并再次调用*`strxfrm`*。没有合理的方式可以预先确定任何给定区域和字符集所需的缓冲区大小。我将缓冲区设置得足够大，以应对几乎所有的可能性，因此为了代码可读性我跳过了此检查，但这不是推荐的做法。*

现在让我们关注`LC_CTYPE`区域设置类别。更改此区域设置类别会影响大多数字符分类函数在*ctype.h*中的工作方式，包括`isalnum`、`isalpha`、`isctrl`、`isgraph`、`islower`、`isprint`、`ispunct`、`isspace`和`isupper`（但特别不包括`isdigit`或`isxdigit`）。它还会影响`toupper`和`tolower`的工作方式——有点像。事实上，*ctype.h*中的函数在国际化方面存在许多问题。问题在于它们依赖算法机制来转换字符大小写，当你坚持使用 ASCII 表时，这些机制工作得很好。然而，一旦你离开这个熟悉的领域，情况就变得不确定了。有时它们工作正常，有时则不然。让它们正常工作的最一致方法是使用宽字符，因为这些函数的宽字符版本在 C 和 C++标准中是较新的，UTF-16 和 UTF-32 字符集也允许类似的算法转换，以支持更广泛的字符集。然而，即使使用宽字符，仍然有一些情况算法方法无法正确转换，因为某些语言有三种形式的双字母（digraphs）：小写、大写和标题式。对于这些情况，根本没有算法可以正确处理。

清单 11-6 中的源代码展示了如何正确地将西班牙单词从大写转换为小写的一种方法。

```
#include <stdio.h>
#include <locale.h>
#include <wctype.h>
#include <wchar.h>

int main()
{
    const wchar_t *orig = L"BAÑO";
    wchar_t xfrm[64];

    setlocale(LC_ALL, "");  // enable environment locale

    int i = 0;
    while (i < wcslen(orig))
    {
        xfrm[i] = towlower(orig[i]);
        i++;
    }
 xfrm[i] = 0;
        printf("orig: %ls, xfrm: %ls\n", orig, xfrm);

        return 0;
}
```

*清单 11-6:* ct.c: *使用宽字符转换西班牙单词*

输出如下：

```
$ gcc ct.c -o ct
$ ./ct
orig: BAÑO, xfrm: baño
$
```

如果你将缓冲区更改为`char`类型并使用 UTF-8，这个程序就无法正常工作。使用宽字符时，几乎无法正常工作。如果你设置`LC_ALL=C`，它只会打印`orig:`，因为如果我们检查清单 11-6 中的`printf`返回值（我们应该这样做——尤其是在处理字符集转换时），我们会看到它返回`-1`，这是它在无法使用`%ls`将宽字符字符串转换为多字节字符串时的返回值。

与其详细讨论`LC_CTYPE`类别中哪些有效、哪些无效，不如直接说，如果你需要做大量这种类型的转换和字符分类，我强烈推荐使用像 IBM 的*Unicode 国际组件（ICU）*^(14)或 GNU libunistring^(15)这样的第三方库（简而言之，它们在所有情况下都会做正确的事情）。ICU 是一个庞大的库，有一定的学习曲线，但如果你需要它，这个努力是值得的。GNU libunistring 稍微容易理解一些，但它仍然提供了很多新的功能。如果你在使用 C++，也可以使用像*Boost::locale*^(16)这样的包装库，它使得访问 ICU 变得更简单，尽管*Boost::locale*本身比较复杂。

##### X/Open 和 POSIX 标准扩展

很遗憾，目前没有一个标准的 C 库函数能够像`strftime`根据区域设置格式化时间和日期那样格式化数字和货币金额。然而，X/Open 和 POSIX 标准提供了一个扩展，并在 GNU C 库中实现了——`strfmon`函数，其原型如下：

```
#include <monetary.h>

ssize_t strfmon(char *s, size_t max, const char *format, ...);
```

它的工作方式与`strftime`非常相似，将格式化后的值字符串放入由`s`指向的*`max`*大小的缓冲区中。*`format`*字符串的作用类似于`printf`系列函数中的格式字符串，以及`strftime`中的格式字符串。格式说明符是特定于此函数的，但与其他函数的格式说明符一样，都是以百分号（`%`）开始，并以格式字符结束。可以在百分号和格式字符之间使用几个支持的修饰符字符。两个有效的格式字符是`i`用于国际格式，`n`用于本地格式。

这个函数旨在格式化货币金额，并遵循所有由`localeconv`提供的`LC_CURRENCY`规则，但它也可以根据`localeconv`提供的`LC_NUMERIC`规则来格式化十进制数字。列表 11-7 提供了不使用任何特殊修饰符来格式化本地和国际货币格式以及格式化十进制数字的示例代码。与`strftime`不同，`strfmon`可以格式化多个值。

```
#include <stdio.h>
#include <locale.h>
#include <monetary.h>

int main()
{
    double amount = 12654.376;
    char buf[256];

    setlocale(LC_ALL, "");  // enable environment locale

    strfmon(buf, sizeof buf, "Local: %n, Int'l: %i, Decimal: %!6.2n",
            amount, amount, amount);
    printf("%s\n", buf);
    return 0;
}
```

*列表 11-7:* amount.c: *调用`strfmon`格式化货币和十进制数值的示例*

让我们构建并执行这个程序，看看使用不同区域设置时显示的内容：

```
$ gcc amount.c -o amount
$ LC_ALL=C ./amount
Local: 12654.38, Int'l: 12654.38, Decimal: 12654.38
$ ./amount
Local: $12,654.38, Int'l: USD 12,654.38, Decimal: 12,654.38
$ LC_ALL=sv_SE.utf8 ./amount
Local: 12 654,38 kr, Int'l: 12 654,38 SEK, Decimal: 12 654,38
$ LC_ALL=ja_JP.utf8 ./amount
Local: ¥12,654, Int'l: JPY 12,654, Decimal: 12,654.38
$
```

所有由`lc`程序在列表 11-2 中显示的货币和数字类别的特性，都被`strfmon`以与标准`strftime`函数处理时间和日期特性相同的方式考虑。例如，在英语和日语中，货币符号显示在数值前面，而瑞典的货币符号`kr`和`SEK`则显示在数值后面。在瑞典（以及许多其他欧洲区域设置）中，小数分隔符是逗号，而日元值不显示小数部分。

十进制格式说明符中的感叹号（`!`）修饰符用于抑制货币符号的显示。通过显式指定格式精度，我们可以覆盖默认的日语区域设置特性，该特性指示货币值不应有小数部分。`strfmon`函数显然是为格式化货币值设计的，但正如我们在这里看到的，它同样可以用来格式化普通的十进制和整数数值。

##### 克服 localeconv 的局限性

X/Open 和 POSIX 标准还提供了一个更好、更具功能性的`localeconv`版本，称为`nl_langinfo`。以下是该函数的原型：

```
#include <langinfo.h>

char *nl_langinfo(nl_item item);
```

这个接口相较于标准库接口有许多优势。首先，它更加高效，只在需要时获取和返回你请求的字段，而不是每次请求时都填充并返回整个区域设置属性结构。`nl_langinfo`函数用于获取由*`item`*指定的全局环境区域设置的单一属性。

如果你的应用程序需要同时管理多个区域设置，可以查看 POSIX 接口，用于在同一应用程序中管理多个离散的区域设置。我在这里不会详细讲解，因为它们管理的是与我已经展示给你的接口相同的区域设置类别。相反，请参阅 POSIX 2008 标准，了解与`nl_langinfo_l`函数相关的`newlocale`、`duplocale`、`uselocale`和`freelocale`函数的信息，其中`nl_langinfo_l`函数接受一个类型为`locale_t`的第二个参数，这个参数是由`newlocale`返回的。我会提到，`uselocale`函数可用于设置当前线程的区域设置。到目前为止，我提到的所有函数都由 GNU C 库实现。

GNU C 库还提供对额外类别的区域设置信息的支持，包括`LC_MESSAGES`、`LC_PAPER`、`LC_NAME`、`LC_ADDRESS`、`LC_TELEPHONE`、`LC_MEASUREMENT`和`LC_IDENTIFICATION`。`LC_MESSAGES`类别已经通过 POSIX 标准化，是*gettext*的基础，我稍后会讨论。其他类别在 C 或 POSIX 中并未标准化，但它们已经被纳入 Linux 的许多方面，包括 X 窗口系统的 Linux 移植版，已经很难想象它们在可预见的未来会被替换或移除。因此，如果你不打算将软件移植到 GNU 工具以外的地方，我建议使用它们。

这些额外的类别无法通过`localeconv`和`struct lconv`结构访问。相反，你需要使用`nl_langinfo`来访问与这些类别相关的区域设置值。

示例 11-8 是与示例 11-2 相同的程序，不同之处在于这个版本使用`nl_langinfo`来显示通过该接口提供的区域设置信息。它的组织方式刻意与两种接口显示相同内容，并保持完全相同的格式。

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>
#include <limits.h>
#include <stdint.h>
#include <locale.h>
#include <langinfo.h>

static void print_grouping(const char *prefix, const char *grouping)
{
    const char *cg;
    printf("%s", prefix);
    for (cg = grouping; *cg && *cg != CHAR_MAX; cg++)
        printf("%c %d", cg == grouping ? ':' : ',', *cg);
    printf("%s\n", *cg == 0 ? " (repeated)" : "");
}

static void print_monetary(bool p_cs_precedes, bool p_sep_by_space,
        bool n_cs_precedes, bool n_sep_by_space,
        int p_sign_posn, int n_sign_posn)
{
    static const char * const sp_str[] =
    {
        "surround symbol and quantity with parentheses",
        "before quantity and symbol",
        "after quantity and symbol",
        "right before symbol",
        "right after symbol"
    };
    printf("    Symbol comes %s a positive (or zero) amount\n",
            p_cs_precedes ? "BEFORE" : "AFTER");
    printf("    Symbol %s separated from a positive (or zero) amount by a space\n",
            p_sep_by_space ? "IS" : "is NOT");
    printf("    Symbol comes %s a negative amount\n",
            n_cs_precedes ? "BEFORE" : "AFTER");
    printf("    Symbol %s separated from a negative amount by a space\n",
            n_sep_by_space ? "IS" : "is NOT");
    printf("    Positive (or zero) amount sign position: %s\n",
            sp_str[p_sign_posn == CHAR_MAX? 4: p_sign_posn]);
    printf("    Negative amount sign position: %s\n",
            sp_str[n_sign_posn == CHAR_MAX? 4: n_sign_posn]);
}
#ifdef OUTER_LIMITS

#define ECOUNT(x) (sizeof(x)/sizeof(*(x)))

static const char *_get_measurement_system(int system_id)
{
    static const char * const measurement_systems[] = { "Metric", "English" };
    int idx = system_id - 1;
    return idx < ECOUNT(measurement_systems)
            ? measurement_systems[idx] : "unknown";
}

#endif

int main(void)
{
    char *isym;

    setlocale(LC_ALL, "");

    printf("Numeric\n");
    printf("  Decimal: [%s]\n", nl_langinfo(DECIMAL_POINT));
    printf("  Thousands separator: [%s]\n", nl_langinfo(THOUSANDS_SEP));

    print_grouping("  Grouping", nl_langinfo(GROUPING));

    printf("\nMonetary\n");
    printf("  Decimal point: [%s]\n", nl_langinfo(MON_DECIMAL_POINT));
    printf("  Thousands separator: [%s]\n", nl_langinfo(MON_THOUSANDS_SEP));
    printf("  Grouping");

    print_grouping("  Grouping", nl_langinfo(MON_GROUPING));

    printf("  Positive amount sign: [%s]\n", nl_langinfo(POSITIVE_SIGN));
    printf("  Negative amount sign: [%s]\n", nl_langinfo(NEGATIVE_SIGN));
    printf("  Local:\n");
    printf("    Symbol: [%s]\n", nl_langinfo(CURRENCY_SYMBOL));
    printf("    Fractional digits: %d\n", *nl_langinfo(FRAC_DIGITS));

    print_monetary(*nl_langinfo(P_CS_PRECEDES), *nl_langinfo(P_SEP_BY_SPACE),
            *nl_langinfo(N_CS_PRECEDES), *nl_langinfo(N_SEP_BY_SPACE),
            *nl_langinfo(P_SIGN_POSN), *nl_langinfo(N_SIGN_POSN));

    printf("  International:\n");
    isym = nl_langinfo(INT_CURR_SYMBOL);
    printf("    Symbol (ISO 4217): [%3.3s], separator: [%s]\n",
           isym, strlen(isym) > 3 ? isym + 3 : "");
    printf("    Fractional digits: %d\n", *nl_langinfo(INT_FRAC_DIGITS));

    print_monetary(*nl_langinfo(INT_P_CS_PRECEDES), *nl_langinfo(INT_P_SEP_BY_SPACE),
            *nl_langinfo(INT_N_CS_PRECEDES), *nl_langinfo(INT_N_SEP_BY_SPACE),
            *nl_langinfo(INT_P_SIGN_POSN), *nl_langinfo(INT_N_SIGN_POSN));
 printf("\nTime\n");
    printf("  AM: [%s]\n", nl_langinfo(AM_STR));
    printf("  PM: [%s]\n", nl_langinfo(PM_STR));
    printf("  Date & time format: [%s]\n", nl_langinfo(D_T_FMT));
    printf("  Date format: [%s]\n", nl_langinfo(D_FMT));
    printf("  Time format: [%s]\n", nl_langinfo(T_FMT));
    printf("  Time format (AM/PM): [%s]\n", nl_langinfo(T_FMT_AMPM));
    printf("  Era: [%s]\n", nl_langinfo(ERA));
    printf("  Year (era): [%s]\n", nl_langinfo(ERA_YEAR));
    printf("  Date & time format (era): [%s]\n", nl_langinfo(ERA_D_T_FMT));
    printf("  Date format (era): [%s]\n", nl_langinfo(ERA_D_FMT));
    printf("  Time format (era): [%s]\n", nl_langinfo(ERA_T_FMT));
    printf("  Alt digits: [%s]\n", nl_langinfo(ALT_DIGITS));

    printf("   Days (abbr)");
    for (int i = 0; i < 7; i++)
        printf("%c %s", i == 0 ? ':' : ',', nl_langinfo(ABDAY_1 + i));
    printf("\n");

    printf("  Days (full)");
    for (int i = 0; i < 7; i++)
        printf("%c %s", i == 0 ? ':' : ',', nl_langinfo(DAY_1 + i));
    printf("\n");

    printf("  Months (abbr)");
    for (int i = 0; i < 12; i++)
        printf("%c %s", i == 0 ? ':' : ',', nl_langinfo(ABMON_1 + i));
    printf("\n");

    printf("  Months (full)");
    for (int i = 0; i < 12; i++)
        printf("%c %s", i == 0 ? ':' : ',', nl_langinfo(MON_1 + i));
    printf("\n");

    printf("\nMessages\n");
    printf("  Codeset: %s\n", nl_langinfo(CODESET));

#ifdef OUTER_LIMITS

    printf("\nQueries\n");
    printf("  YES expression: %s\n", nl_langinfo(YESEXPR));
    printf("  NO expression:  %s\n", nl_langinfo(NOEXPR));

    printf("\nPaper\n");
    printf("  Height:  %dmm\n", (int)(intptr_t)nl_langinfo(_NL_PAPER_HEIGHT));
    printf("  Width:   %dmm\n", (int)(intptr_t)nl_langinfo(_NL_PAPER_WIDTH));
    printf("  Codeset: %s\n", nl_langinfo(_NL_PAPER_CODESET));

    printf("\nName\n");
    printf("  Format: %s\n", nl_langinfo(_NL_NAME_NAME_FMT));
    printf("  Gen:    %s\n", nl_langinfo(_NL_NAME_NAME_GEN));
    printf("  Mr:     %s\n", nl_langinfo(_NL_NAME_NAME_MR));
    printf("  Mrs:    %s\n", nl_langinfo(_NL_NAME_NAME_MRS));
    printf("  Miss:   %s\n", nl_langinfo(_NL_NAME_NAME_MISS));
    printf("  Ms:     %s\n", nl_langinfo(_NL_NAME_NAME_MS));
 printf("\nAddress\n");
    printf("  Country name:   %s\n", nl_langinfo(_NL_ADDRESS_COUNTRY_NAME));
    printf("  Country post:   %s\n", nl_langinfo(_NL_ADDRESS_COUNTRY_POST));
    printf("  Country abbr2:  %s\n", nl_langinfo(_NL_ADDRESS_COUNTRY_AB2));
    printf("  Country abbr3:  %s\n", nl_langinfo(_NL_ADDRESS_COUNTRY_AB3));
    printf("  Country num:    %d\n",
            (int)(intptr_t)nl_langinfo(_NL_ADDRESS_COUNTRY_NUM));
    printf("  Country ISBN:   %s\n", nl_langinfo(_NL_ADDRESS_COUNTRY_ISBN));
    printf("  Language name:  %s\n", nl_langinfo(_NL_ADDRESS_LANG_NAME));
    printf("  Language abbr:  %s\n", nl_langinfo(_NL_ADDRESS_LANG_AB));
    printf("  Language term:  %s\n", nl_langinfo(_NL_ADDRESS_LANG_TERM));
    printf("  Language lib:   %s\n", nl_langinfo(_NL_ADDRESS_LANG_LIB));
    printf("  Codeset:        %s\n", nl_langinfo(_NL_ADDRESS_CODESET));

    printf("\nTelephone\n");
    printf("  Int'l format:    %s\n", nl_langinfo(_NL_TELEPHONE_TEL_INT_FMT));
    printf("  Domestic format: %s\n", nl_langinfo(_NL_TELEPHONE_TEL_DOM_FMT));
    printf("  Int'l select:    %s\n", nl_langinfo(_NL_TELEPHONE_INT_SELECT));
    printf("  Int'l prefix:    %s\n", nl_langinfo(_NL_TELEPHONE_INT_PREFIX));
    printf("  Codeset:         %s\n", nl_langinfo(_NL_TELEPHONE_CODESET));

   printf("\nMeasurement\n");
   printf("  System:  %s\n",_get_measurement_system(
           (int)*nl_langinfo(_NL_MEASUREMENT_MEASUREMENT)));
   printf("  Codeset: %s\n", nl_langinfo(_NL_MEASUREMENT_CODESET));

   printf("\nIdentification\n");
   printf("  Title:       %s\n", nl_langinfo(_NL_IDENTIFICATION_TITLE));
   printf("  Source:      %s\n", nl_langinfo(_NL_IDENTIFICATION_SOURCE));
   printf("  Address:     %s\n", nl_langinfo(_NL_IDENTIFICATION_ADDRESS));
   printf("  Contact:     %s\n", nl_langinfo(_NL_IDENTIFICATION_CONTACT));
   printf("  Email:       %s\n", nl_langinfo(_NL_IDENTIFICATION_EMAIL));
   printf("  Telephone:   %s\n", nl_langinfo(_NL_IDENTIFICATION_TEL));
   printf("  Language:    %s\n", nl_langinfo(_NL_IDENTIFICATION_LANGUAGE));
   printf("  Territory:   %s\n", nl_langinfo(_NL_IDENTIFICATION_TERRITORY));
   printf("  Audience:    %s\n", nl_langinfo(_NL_IDENTIFICATION_AUDIENCE));
   printf("  Application: %s\n", nl_langinfo(_NL_IDENTIFICATION_APPLICATION));
   printf("  Abbr:        %s\n", nl_langinfo(_NL_IDENTIFICATION_ABBREVIATION));
   printf("  Revision:    %s\n", nl_langinfo(_NL_IDENTIFICATION_REVISION));
   printf("  Date:        %s\n", nl_langinfo(_NL_IDENTIFICATION_DATE));
   printf("  Category:    %s\n", nl_langinfo(_NL_IDENTIFICATION_CATEGORY));
   printf("  Codeset:     %s\n", nl_langinfo(_NL_IDENTIFICATION_CODESET));

#endif // OUTER_LIMITS

    return 0;
}
```

*示例 11-8:* nl.c: *使用 `nl_langinfo` 显示可用的区域设置信息*

要构建这个代码，你需要在命令行中添加几个定义：`_GNU_SOURCE`和`OUTER_LIMITS`。第一个定义属于 GNU C 库，允许*nl.c*访问 C99 之前 C 标准中没有的扩展国际货币字段。第二个是我自己的发明，允许你在没有 GNU C 库提供的扩展类别的情况下构建程序：

```
$ gcc -D_GNU_SOURCE -DOUTER_LIMITS nl.c -o nl
$ ./nl
Numeric
  Decimal: [.]
  Thousands separator: [,]
  Grouping: 3, 3 (repeated)

Monetary
  Decimal point: [.]
  Thousands separator: [,]
  Grouping    Grouping: 3, 3 (repeated)
  Positive amount sign: []
  Negative amount sign: [-]
  Local:
    Symbol: [$]
    Fractional digits: 2
    Symbol comes BEFORE a positive (or zero) amount
    Symbol is NOT separated from a positive (or zero) amount by a space
    Symbol comes BEFORE a negative amount
    Symbol is NOT separated from a negative amount by a space
    Positive (or zero) amount sign position: before quantity and symbol
    Negative amount sign position: before quantity and symbol
  International:
    Symbol (ISO 4217): [USD], separator: [ ]
    Fractional digits: 2
    Symbol comes BEFORE a positive (or zero) amount
    Symbol IS separated from a positive (or zero) amount by a space
    Symbol comes BEFORE a negative amount
    Symbol IS separated from a negative amount by a space
    Positive (or zero) amount sign position: before quantity and symbol
    Negative amount sign position: before quantity and symbol

Time
  AM: [AM]
  PM: [PM]
  Date & time format: [%a %d %b %Y %r %Z]
  Date format: [%m/%d/%Y]
  Time format: [%r]
  Time format (AM/PM): [%I:%M:%S %p]
  Era: []
  Year (era): []
  Date & time format (era): []
  Date format (era): []
  Time format (era): []
  Alt digits: []
  Days (abbr): Sun, Mon, Tue, Wed, Thu, Fri, Sat
  Days (full): Sunday, Monday, Tuesday, Wednesday, Thursday, Friday, Saturday
  Months (abbr): Jan, Feb, Mar, Apr, May, Jun, Jul, Aug, Sep, Oct, Nov, Dec
  Months (full): January, February, March, April, May, June, July, August,
September, October, November, December
Messages
  Codeset: UTF-8

Queries
  YES expression: ^[yY].*
  NO expression:  ^[nN].*

Paper
  Height:  279mm
  Width:   216mm
  Codeset: UTF-8

Name
  Format: %d%t%g%t%m%t%f
  Gen:
  Mr:       Mr.
  Mrs:      Mrs.
  Miss:     Miss.
  Ms:       Ms.

Address
  Country name:  USA
  Country post:  USA
  Country abbr2: US
  Country abbr3: USA
  Country num:   840
  Country ISBN:  0
  Language name: English
  Language abbr: en
  Language term: eng
  Language lib:  eng
  Codeset:       UTF-8

Telephone
  Int'l format:    +%c (%a) %l
  Domestic format: (%a) %l
  Int'l select:    11
  Int'l prefix:    1
  Codeset:         UTF-8

Measurement
  System:  English
  Codeset: UTF-8

Identification
  Title:         English locale for the USA
  Source:        Free Software Foundation, Inc.
  Address:       http://www.gnu.org/software/libc/
  Contact:
  Email:         bug-glibc-locales@gnu.org
  Telephone:
  Language:      English
  Territory:     USA
  Audience:
  Application:
 Abbr:
  Revision:        1.0
  Date:            2000-06-24
  Category:        en_US:2000
  Codeset:         UTF-8
$
```

上述输出中突出显示的部分显示了`nl`输出中的一部分，超出了 Listing 11-2 中的`lc`程序。额外的区域类别定义如下。

**`LC_MESSAGES`**

此类别提供一个额外的项目值，`CODESET`，它定义了该区域使用的字符集。该项目被归类为“消息”，因为它旨在帮助翻译应用程序代码中的静态文本消息。该值还可以作为环境变量在 Linux 系统上使用，以帮助选择要使用的静态消息目录。

**`LC_PAPER`**

纸张类别提供两个项目，`_NL_PAPER_HEIGHT`和`_NL_PAPER_WIDTH`，它们返回该区域最常用打印纸张的纸张尺寸值（以毫米为单位）。这在格式化打印输出或自动选择纸张尺寸时非常有用——例如*letter*和*A04*。请注意，从这些项目枚举值返回的指针值应像本地字大小整数值一样对待，而不是实际的指针。有关详细信息，请参阅 Listing 11-8 中的*nl.c*代码。

**`LC_NAME`**

名称类别提供关于格式化称呼（如先生、女士、小姐和女士）在该区域的相关信息。此类别中的项目允许你的软件自动选择如何以当前语言和地区陈述这些称呼。

**`LC_ADDRESS`**

地址类别提供返回地理信息的项目，例如国家名称、邮政编码以及两字母和三字母的国家名称缩写。它还返回该区域使用的语言名称和库。

**`LC_TELEPHONE`**

电话类别提供格式化说明符字符串，可以在*printf*系列函数中使用，以在当前区域常见的样式中显示电话号码。

**`LC_MEASUREMENT`**

测量类别提供一个项目，用于返回当前区域使用的度量系统。`_NL_MEASUREMENT_MEASUREMENT`项目返回一个字符串，其第一个字符是一个短整数值：`0`表示公制，`1`表示英制。

**`LC_IDENTIFICATION`**

标识类别实际上是区域元数据。也就是说，该类别的字段返回关于领土、作者以及创建当前区域所使用的过程的信息（例如，区域作者的姓名、电子邮件地址、电话号码等）。它还返回有关区域的版本信息。请注意，从`_NL_ADDRESS_COUNTRY_NUM`返回的指针值应被视为本地字大小整数值，而非指针。有关详细信息，请参阅 Listing 11-8 中的*nl.c*代码。

你可以通过 Linux 发行版中预装的`locale`命令行程序，使用`-k`选项访问相同的信息，如下所示：

```
$ locale -k LC_PAPER
height=279
width=216
paper-codeset="UTF-8"
$
```

您可以查询 GNU C 库的`nl_langinfo`函数，以获取当前区域设置下的各类时间和日期格式属性，如 AM 和 PM 字符串、各种更细粒度的格式说明符字符串以及一周的完整和缩写日期和月份。

GNU C 库的`nl_langinfo`实现甚至返回了正则表达式，用于匹配查询响应。`YESEXPR`和`NOEXPR`项枚举值返回的正则表达式可以用来匹配软件提示的问题的*是*或*否*答案。

#### *静态消息源代码仪表化*

在源代码中对区域设置特定的静态文本消息进行仪表化，也是软件国际化过程的一部分，因此我们将在此讨论静态文本显示消息的仪表化。然后我们将继续探讨如何在第十二章中生成和使用语言包，在那里我将讨论本地化。

到现在为止，应该很清楚，我们需要处理“*`greeting`*，来自*`progname`*！”这段静态文本（例如从 Jupiter 打印的文本）。我不会继续讨论 Jupiter，但它提供了一个简洁的示例，说明当区域设置改变时，我们的程序中需要做出变化。为翻译静态显示消息而对源代码进行仪表化的过程，涉及扫描源代码中所有可能在程序执行期间显示给用户的字符串文字，然后做一些操作，使得程序能够使用专门针对当前区域设置的版本来显示这些字符串。

有一些开源（以及几个第三方商业）库可以用来完成这项任务，但我们将专注于 GNU *gettext* 库。从软件的角度来看，*gettext* 库非常简单。它的最简单形式包含一个用于标记需要翻译的消息的函数和两个用于选择显示消息目录的函数。标记函数名为`gettext`，其原型显示在列表 11-9 中。

```
#include <libintl.h>

char *gettext(const char *msgid);
```

*列表 11-9：`gettext`函数的原型*

该函数接受一个消息标识符作为*`msgid`*参数，并将显示消息返回给用户。消息标识符可以是任何字符串，但通常是显示消息本身，使用 US-ASCII 编码。这样做的原因是，如果找不到消息目录，`gettext`会返回*`msgid`*值本身，程序将以相同的方式使用该值，就像如果找到了翻译后的消息一样。因此，`gettext`函数不会以任何可能导致程序在任何合理条件下无法正常工作的方式失败。

这种约定使得为现有程序添加国际化支持以及编写使用基于语言环境的消息目录的新程序变得非常简单。你只需要找到程序源文件中的所有静态文本消息，并将它们包装在 `gettext` 调用中。

有时，需要向翻译员提供比单纯的字符串更多的上下文信息。例如，当为菜单项提供消息 ID 时，例如 `File` 菜单中的 `Open` 子菜单选项，程序员可能已经告诉翻译员，他们已将整个菜单层级以 `|File|Open` 这样的格式提供给翻译员。当翻译员看到这个时，他们会知道只需要翻译最后一个竖线符号后面的部分。但如果当前语言环境没有翻译，消息 ID 将会是完整的字符串。在这种情况下，程序员必须编写代码来检查是否存在前导竖线。如果找到，只有最后一个竖线后的部分才应该显示。

Listing 11-10 中的代码展示了一个非常简短（且有些熟悉）的示例程序，使用了 `gettext`。

```
#include <stdio.h>
#include <libintl.h>

#define _(x) gettext(x)

int main()
{
     printf(_("Hello, world!\n"));
     return 0;
}
```

*Listing 11-10:* gt.c: *一个简短的程序，演示了如何使用 gettext 库*

`printf` 函数将 `gettext` 的返回值发送到 `stdout`。`gettext` 函数由 GNU C 库导出，因此使用时无需额外的库。当不使用 GNU C 时，只需链接 *intl* 库（共享对象或静态库）。

我们可以直接在 `printf` 中调用 `gettext`，但下划线（`_`）宏在国际化软件时是常用的惯用法，原因有二：首先，它减少了在现有代码库中使用 *gettext* 时的视觉干扰；其次，如果我们选择通过额外功能包装 `gettext`，或者决定使用 `gettext` 的其他变体（例如 `dgettext` 和 `dcgettext`），它允许我们在一个地方进行替换。我在这里没有讨论这些变体，但你可以在 *GNU C Library* 手册中找到更多信息。^(17)

##### 消息目录选择

消息目录的选择分为两个阶段：程序员阶段和用户阶段。程序员阶段由 `textdomain` 和 `bindtextdomain` 函数处理。这些函数的原型（也由 GNU C 库导出）见 Listing 11-11。

```
#include <libintl.h>

char *textdomain(const char *domainname);
char *bindtextdomain(const char *domainname, const char *dirname);
```

*Listing 11-11: `textdomain` 和 `bindtextdomain` 的原型*

`textdomain` 函数允许软件作者在程序的任何给定点确定正在使用的消息目录域。该域表示包含程序中部分消息的特定消息目录。从源代码中提取的所有属于特定域的字符串最终都会进入该域的消息目录。

一个包可能有多个域。域之间的典型边界，因此也就是消息目录之间的边界，是可执行模块——无论是程序还是库。例如，*curl*包安装了命令行`curl`程序和*libcurl.so*共享库。*curl*库设计为供`curl`程序和其他第三方程序及库使用。如果*curl*包进行了国际化，包的作者可能会决定为`curl`程序使用*curl*域，为库使用*libcurl*域，这样使用*libcurl*的第三方应用就不需要安装`curl`消息目录。

*GNU C 库*手册中使用的示例^(18)是*libc*本身使用`libc`作为域名，但使用*libc*的程序会使用自己的域名。简单来说，这些函数中的*`domainname`*参数直接对应一个消息目录文件名。

`bindtextdomain`中的`dirname`参数用于指定一个基础目录，在该目录中搜索已定义的消息目录结构，我稍后会讨论这个结构。通常，传递给此参数的值是 Automake `datadir`变量中的绝对路径，并以*/locale*结尾。回想一下，`datadir`默认包含`$(prefix)`*/share*，而`prefix`包含*/usr/local*，所以这里使用的完整路径将是*/usr/local/share/locale*。对于由发行版提供的包，`prefix`通常只是*/usr*，因此完整路径将变成*/usr/share/locale*。因此，维护者需要确保在软件中可以使用`datadir`（使用第三章中讨论的技术），并在传递给该参数的参数中引用它。

示例 11-12 展示了如何添加必要的代码来根据当前地区选择合适的消息目录。当然，我们必须先以通常的方式通过调用`setlocale`让程序了解当前地区。

```
#include <stdio.h>
#include <locale.h>
#include <libintl.h>

#ifndef LOCALE_DIR
# define LOCALE_DIR "/usr/local/share/locale"
#endif

#ifdef TEST_L10N
# include <stdlib.h>
# undef LOCALE_DIR
# define LOCALE_DIR getenv("PWD")
#endif

#define _(x) gettext(x)

int main()
{
     const char *localedir = LOCALE_DIR;

     setlocale(LC_ALL, "");
     bindtextdomain("gt", localedir);
     textdomain("gt");

     printf(_("Hello, world!\n"));

     return 0;
}
```

*示例 11-12:* gt.c: *启用当前地区并选择消息目录的增强功能*

我在这里使用*gt*作为域名，因为这是程序的名称。如果这个程序是某个包的一部分，而包中的所有组件都使用相同的域名，那么包名可能是一个更好的选择。

传递给`bindtextdomain`第二个参数的目录名来源于将来*config.h*的包含。我们稍后会在将该程序集成到 Autotools 构建系统时添加它。如果在编译器命令行中定义了`TEST_L10N`，目录名将解析为`PWD`环境变量的值，从而允许我们在任何包含地区目录结构的位置测试程序。（我们将在第十二章中用更符合 Autotool 的机制替换这个临时方案。）

这就是为你的代码添加消息目录查找功能的全部内容。在下一部分，我将讨论如何生成和构建消息目录，这也是本地化软件包过程的一部分。我还将讲解*gettext*库的内部工作原理，该库允许用户在用户阶段选择应由其环境变量设置选择的消息目录。

### 总结

在本章中，我的目标是为你提供足够的背景知识，让你能够轻松地继续学习如何使你的软件项目国际化。我已经涵盖了 C 标准库中帮助你进行软件国际化的功能。

在下一章中，我们将继续探索这个话题，深入研究本地化。我们还将发现如何将这一切与 Autotools 结合，以便通过 Automake 生成的`make`命令构建和安装语言包。
