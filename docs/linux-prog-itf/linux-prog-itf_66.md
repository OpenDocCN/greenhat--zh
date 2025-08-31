## 附录 B. 解析命令行选项

一个典型的 UNIX 命令行具有以下形式：

### 注意

```
*command* [ *options* ] *arguments*
```

一个选项的形式是一个连字符（`-`）后跟一个唯一字符来标识选项，后面可能跟一个选项的参数。一个选项如果有参数，可以选择与参数之间用空格分开。多个选项可以在一个连字符后组合在一起，且组合中的最后一个选项可能是需要参数的选项。根据这些规则，以下命令都是等效的：

```
$ `grep -l -i -f patterns *.c`
$ `grep -lif patterns *.c`
$ `grep -lifpatterns *.c`
```

在上述命令中，*–l* 和 *–i* 选项没有参数，而 *–f* 选项将字符串 *patterns* 作为其参数。

由于许多程序（包括本书中的一些示例程序）需要按上述格式解析选项，因此提供此功能的标准库函数为 *getopt()*。

```
#include <unistd.h>

extern int `optind`, `opterr`, `optopt`;
extern char *`optarg`;

int `getopt`(int *argc*, char *const *argv*[], const char **optstring*);
```

### 注意

参见正文中的描述，了解返回值的详细信息。

*getopt()* 函数解析在 *argc* 和 *argv* 中给定的一组命令行参数，这些参数通常来自于传递给 *main()* 的同名参数。*optstring* 参数指定了 *getopt()* 在 *argv* 中应查找的选项集合。该参数由一系列字符组成，每个字符表示一个选项。SUSv3 指定 *getopt()* 至少应允许 `[a-zA-Z0-9]` 字符集中的字符作为选项。大多数实现还允许其他字符，除了 `:`, `?`, 和 `-`，这些字符对 *getopt()* 有特殊意义。每个选项字符后面可以跟一个冒号（`:`），表示该选项需要一个参数。

我们通过重复调用 *getopt()* 来解析命令行。每次调用返回有关下一个未处理选项的信息。如果找到了选项，则返回选项字符作为函数结果。如果已达到选项列表的末尾，*getopt()* 返回 -1。如果选项有参数，*getopt()* 会将全局变量 *optarg* 设置为指向该参数的地址。

注意，*getopt()* 的函数返回值是 *int* 类型。我们不能将 *getopt()* 的结果赋值给 *char* 类型的变量，因为在 *char* 为无符号类型的系统上，*char* 变量与 -1 的比较是无效的。

### 注意

如果选项没有参数，则 *glibc getopt()* 实现（与大多数其他实现一样）将 *optarg* 设置为 `NULL`。然而，SUSv3 并未指定此行为，因此应用程序不能依赖此行为（通常也不需要）。

SUSv3 指定了（并且 *glibc* 实现了）一个相关函数，*getsubopt()*，该函数解析由一个或多个用逗号分隔的字符串组成的选项参数，格式为 *name[=value]*。详细信息请参见 *getsubopt(3)* 手册页。

在每次调用 *getopt()* 时，全局变量 *optind* 会更新为包含 *argv* 中下一个未处理元素的索引（当多个选项在一个词中组合时，*getopt()* 会进行一些内部记录，以跟踪下一个要处理的部分）。*optind* 变量在第一次调用 *getopt()* 之前会自动设置为 1。我们可能会在以下两种情况下使用此变量：

+   如果 *getopt()* 返回 -1，表示没有更多选项，并且 *optind* 小于 *argc*，那么 *argv[optind]* 就是命令行中下一个非选项词的位置。

+   如果我们正在处理多个命令行向量或重新扫描相同的命令行，则必须显式地将 *optind* 重置为 1。

当以下情况发生时，*getopt()* 函数返回 -1，表示选项列表的结束：

+   到达由 *argc* 和 *argv* 描述的列表末尾（即 *argv[optind]* 为 `NULL`）。

+   *argv* 中下一个未处理的词不以选项分隔符开头（即 *argv[optind][0]* 不是一个连字符）。

+   *argv* 中下一个未处理的词是一个单独的连字符（即 *argv[optind]* 是 -）。某些命令会将这样的词理解为具有特殊意义的参数，如第 5.11 节所述。

+   *argv* 中下一个未处理的词由两个连字符 (`--`) 组成。在这种情况下，*getopt()* 会默默地消耗这两个连字符，并且 *optind* 会调整为指向双连字符后面的下一个词。这种语法使得用户能够指示命令的选项结束，即使命令行上的下一个词（双连字符后的词）看起来像是一个选项（即以连字符开头）。例如，如果我们想用 *grep* 搜索文件中的字符串 *-k*，那么我们可以写作 *grep -- -k myfile*。

在 *getopt()* 处理选项列表时，可能会发生两种类型的错误。一种错误发生在遇到未在 *optstring* 中指定的选项时。另一种错误发生在期望有参数的选项没有提供参数时（即该选项出现在命令行的末尾）。*getopt()* 处理和报告这些错误的规则如下：

+   默认情况下，*getopt()* 会在标准错误输出上打印适当的错误信息，并返回字符`?`作为其函数结果。在这种情况下，全球变量 *optopt* 返回错误的选项字符（即未识别的选项字符或缺少参数的选项字符）。

+   全局变量 *opterr* 可用于抑制 *getopt()* 打印的错误信息。默认情况下，该变量设置为 1。如果将其设置为 0，那么 *getopt()* 不会打印错误信息，但在其他方面的行为与前述相同。程序可以通过 `?` 函数结果来检测错误，并显示自定义的错误信息。

+   另外，我们可以通过在 *optstring* 中指定冒号（`:`）作为第一个字符来抑制错误信息（这样会覆盖将 *opterr* 设置为 0 的效果）。在这种情况下，错误报告的方式与将 *opterr* 设置为 0 相同，唯一的区别是，缺少参数的选项会通过返回 `:` 作为函数结果来报告。这种返回值的差异使我们能够区分两种类型的错误（未识别的选项和缺少选项参数），如果我们需要区分它们的话。

上述错误报告替代方法在 表 B-1 错误报告行为") 中进行了总结。

表 B-1. *getopt()* 错误报告行为

| 错误报告方法 | *getopt()* 是否显示错误信息？ | 对于未识别选项的返回值 | 对于缺少参数的返回值 |
| --- | --- | --- | --- |
| 默认（*opterr == 1*） | Y | `?` | `?` |
| *opterr == 0* | N | `?` | `?` |
| *optstring* 开头的 `:` | N | `?` | `:` |

## 示例程序

示例 B-1") 演示了使用 *getopt()* 来解析命令行中的两个选项：* -x * 选项，不需要参数，以及 *-p* 选项，需要一个参数。该程序通过在 *optstring* 中指定冒号（`:`）作为第一个字符，来抑制 *getopt()* 的错误信息。

为了观察 *getopt()* 的操作，我们在程序中加入了一些 *printf()* 调用，以显示每次 *getopt()* 调用返回的信息。程序完成后，会打印一些关于指定选项的总结信息，并且如果命令行上有下一个非选项单词，还会显示出来。以下是运行该程序时使用不同命令行参数的结果：

```
$ `./t_getopt -x -p hello world`
opt =120 (x); optind = 2
opt =112 (p); optind = 4
-x was specified (count=1)
-p was specified with the value "hello"
First nonoption argument is "world" at argv[4]
$ `./t_getopt -p`
opt = 58 (:); optind = 2; optopt =112 (p)
Missing argument (-p)
Usage: ./t_getopt [-p arg] [-x]
$ `./t_getopt -a`
opt = 63 (?); optind = 2; optopt = 97 (a)
Unrecognized option (-a)
Usage: ./t_getopt [-p arg] [-x]
$ `./t_getopt -p str -- -x`
opt =112 (p); optind = 3
-p was specified with the value "str"
First nonoption argument is "-x" at argv[4]
$ `./t_getopt -p -x`
opt =112 (p); optind = 3
-p was specified with the value "-x"
```

请注意，在上面的最后一个示例中，字符串 *-x* 被解释为 *-p* 选项的一个参数，而不是作为一个选项。

示例 B-1. 使用 *getopt()*

```
`getopt/t_getopt.c`
#include <ctype.h>
#include "tlpi_hdr.h"

#define printable(ch) (isprint((unsigned char) ch) ? ch : '#')

static void             /* Print "usage" message and exit */
usageError(char *progName, char *msg, int opt)
{
    if (msg != NULL && opt != 0)
        fprintf(stderr, "%s (-%c)\n", msg, printable(opt));
    fprintf(stderr, "Usage: %s [-p arg] [-x]\n", progName);
    exit(EXIT_FAILURE);
}

int
main(int argc, char *argv[])
{
    int opt, xfnd;
    char *pstr;

    xfnd = 0;
    pstr = NULL;

    while ((opt = getopt(argc, argv, ":p:x")) != -1) {
        printf("opt =%3d (%c); optind = %d", opt, printable(opt), optind);
        if (opt == '?' || opt == ':')
            printf("; optopt =%3d (%c)", optopt, printable(optopt));
        printf("\n");

        switch (opt) {
        case 'p': pstr = optarg;        break;
        case 'x': xfnd++;               break;
        case ':': usageError(argv[0], "Missing argument", optopt);
        case '?': usageError(argv[0], "Unrecognized option", optopt);
        default:  fatal("Unexpected case in switch()");
        }
    }

    if (xfnd != 0)
        printf("-x was specified (count=%d)\n", xfnd);
    if (pstr != NULL)
        printf("-p was specified with the value \"%s\"\n", pstr);
    if (optind < argc)
        printf("First nonoption argument is \"%s\" at argv[%d]\n",
                argv[optind], optind);
    exit(EXIT_SUCCESS);
}
      `getopt/t_getopt.c`
```

## GNU 特有行为

默认情况下，*glibc* 实现的 *getopt()* 实现了一个非标准功能：它允许选项和非选项交替出现。例如，以下两种方式是等效的：

```
$ `ls -l file`
$ `ls file -l`
```

在处理第二种形式的命令行时，*getopt()* 会重新排列 *argv* 的内容，使所有选项移到数组的开头，所有非选项移到数组的末尾。（如果 *argv* 包含一个指向 `--` 的元素，那么只有该元素之前的元素会被重新排列并解释为选项。）换句话说，之前所示 *getopt()* 原型中 *argv* 的 `const` 声明在 *glibc* 中并不完全成立。

*argv*的内容重新排列不被 SUSv3（或 SUSv4）允许。我们可以通过将环境变量`POSIXLY_CORRECT`设置为任何值，强制*getopt()*提供符合标准的行为（即遵循前面列出的确定选项列表结束的规则）。这可以通过两种方式完成：

+   在程序内部，我们可以调用*putenv()*或*setenv()*。这样做的好处是用户无需进行任何操作。其缺点是需要修改程序源代码，并且只会改变该程序的行为。

+   我们可以在执行程序之前从 shell 中定义变量：

    ```
    $ `export POSIXLY_CORRECT=y`
    ```

    这种方法的好处是它会影响所有使用*getopt()*的程序。然而，它也有一些缺点。`POSIXLY_CORRECT`会导致 Linux 工具的其他行为变化。此外，设置这个变量需要用户的明确干预（很可能是通过在 shell 启动文件中设置该变量）。

防止*getopt()*重新排列命令行参数的另一种方法是使*optstring*的第一个字符为加号（`+`）。(如果我们还希望抑制*getopt()*的错误信息，如上所述，那么*optstring*的前两个字符应该是`+:`，按此顺序。）与使用*putenv()*或*setenv()*一样，这种方法的缺点是需要修改程序代码。有关详细信息，请参见*getopt(3)*手册页。

### 注意

SUSv4 的未来技术更正很可能会添加一个规范，要求在*optstring*中使用加号来防止命令行参数的重新排列。

请注意，*glibc getopt()*的重新排列行为会影响我们编写 shell 脚本的方式。（这会影响从其他系统移植 shell 脚本到 Linux 的开发者。）假设我们有一个 shell 脚本，它对目录中的所有文件执行以下命令：

```
chmod 644 *
```

如果其中一个文件名以连字符开头，那么*glibc getopt()*的重新排列行为会导致该文件名被解释为*chmod*的一个选项。这在其他 UNIX 实现中不会发生，因为第一个非选项（`644`）的出现确保了*getopt()*不再继续查找命令行中的选项。对于大多数命令，（如果我们没有设置`POSIXLY_CORRECT`，那么）在必须在 Linux 上运行的 shell 脚本中处理这种可能性的方法是，在第一个非选项参数之前放置字符串`--`。因此，我们会将上述行重写为：

```
chmod -- 644 *
```

在这个特定的例子中，使用文件名生成，我们可以改为这样写：

```
chmod 644 ./*
```

尽管我们上面用了文件名模式匹配（通配符）的示例，但类似的场景也可能由于其他 shell 处理（例如命令替换和参数扩展）而发生，处理方式也可以类似，通过使用`--`字符串来分隔选项和参数。

## GNU 扩展

GNU C 库提供了对*getopt()*的多个扩展。我们简要说明如下：

+   SUSv3 规范允许选项仅具有必需的参数。在 GNU 版本的 *getopt()* 中，我们可以在 *optstring* 中的选项字符后放置两个冒号，以表示其参数是可选的。此类选项的参数必须与选项本身在同一个单词中（即，选项与其参数之间不能有空格）。如果没有提供参数，则从 *getopt()* 返回时，*optarg* 被设置为 `NULL`。

+   许多 GNU 命令支持一种长选项语法。长选项以两个连字符开头，选项本身通过一个单词而非单个字符来标识，如以下示例所示：

    ```
    $ `gcc --version`
    ```

    *glibc* 函数 *getopt_long()* 可用于解析此类选项。

+   GNU C 库提供了一个更为复杂（但不便携）的 API 来解析命令行，称为 *argp*。该 API 在 *glibc* 手册中有描述。
