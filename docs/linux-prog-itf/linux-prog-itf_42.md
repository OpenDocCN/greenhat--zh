## 第四十二章：共享库的高级特性

上一章讲解了共享库的基础知识。本章描述了共享库的若干高级特性，包括以下内容：

+   动态加载共享库；

+   控制共享库中定义的符号的可见性；

+   使用链接器脚本创建版本化符号；

+   使用初始化和终结函数在库加载和卸载时自动执行代码；

+   共享库预加载；以及

+   使用`LD_DEBUG`来监视动态链接器的操作。

## 动态加载的共享库

当可执行文件启动时，动态链接器会加载程序的所有共享库依赖列表。然而，有时可以在稍后的时间加载库。例如，插件只有在需要时才会加载。这种功能由动态链接器的 API 提供。这个 API 通常称为*dlopen* API，最初出现在 Solaris 中，现在许多部分都在 SUSv3 中进行了规范。

*dlopen* API 使得程序能够在运行时打开共享库，在该库中按名称查找函数，然后调用该函数。以这种方式在运行时加载的共享库通常被称为*动态加载的库*，并且与其他共享库一样创建。

核心的*dlopen* API 包含以下函数（所有函数都在 SUSv3 中进行了规范）：

+   *dlopen()*函数打开一个共享库，返回一个句柄，用于后续调用。

+   *dlsym()*函数在库中查找一个符号（包含函数或变量名称的字符串），并返回其地址。

+   *dlclose()*函数关闭通过*dlopen()*打开的库。

+   *dlerror()*函数返回一个错误消息字符串，并在前面函数返回失败时使用。

*glibc*的实现还包括若干相关函数，下面我们将描述其中一些。

在 Linux 上构建使用*dlopen* API 的程序时，我们必须指定* -ldl*选项，以便链接*libdl*库。

### 打开共享库：*dlopen()*

*dlopen()*函数将名为*libfilename*的共享库加载到调用进程的虚拟地址空间，并增加对该库的打开引用计数。

```
#include <dlfcn.h>

void *`dlopen`(const char **libfilename*, int *flags*);
```

### 注意

成功时返回库句柄，出错时返回`NULL`

如果*libfilename*包含斜杠（`/`），*dlopen()*将其解释为绝对路径或相对路径。否则，动态链接器将使用运行时查找共享库中描述的规则来查找共享库。

在成功时，*dlopen()*返回一个句柄，后续可以使用该句柄调用*dlopen* API 中的函数。如果发生错误（例如，无法找到库），*dlopen()*返回`NULL`。

如果*libfilename*指定的共享库依赖于其他共享库，*dlopen()*也会自动加载这些库。如果需要，加载过程会递归进行。我们将这些加载的库称为该库的*依赖树*。

可以多次调用*dlopen()*加载同一个库文件。该库只会被加载到内存一次（通过初始调用），所有调用都会返回相同的*handle*值。然而，*dlopen* API 会为每个库句柄维护一个引用计数。每次调用*dlopen()*时，计数会递增，每次调用*dlclose()*时，计数会递减；只有当计数为 0 时，*dlclose()*才会将库从内存中卸载。

*flags*参数是一个位掩码，必须包括`RTLD_LAZY`或`RTLD_NOW`中的一个常量，且具有以下含义：

`RTLD_LAZY`

库中的未定义函数符号应仅在代码执行时解析。如果一段代码未执行某个符号的需求，该符号就不会被解析。懒加载只会对函数引用进行解析；对变量的引用始终会立即解析。指定`RTLD_LAZY`标志提供的行为与动态链接器在加载可执行文件的动态依赖列表中标识的共享库时的正常操作相对应。

`RTLD_NOW`

库中的所有未定义符号应该在*dlopen()*完成之前立即解析，无论它们是否会被需要。因此，打开库的速度较慢，但任何潜在的未定义函数符号错误会立即被检测到，而不是在某个后续的时间点。这对于调试应用程序时很有用，或者简单地确保应用程序在遇到未解析的符号时会立即失败，而不是在长时间运行后才发生错误。

### 注意

通过将环境变量`LD_BIND_NOW`设置为非空字符串，我们可以强制动态链接器在加载可执行文件的动态依赖列表中标识的共享库时立即解析所有符号（即，像`RTLD_NOW`一样）。此环境变量在*glibc* 2.1.1 及以后版本中有效。设置`LD_BIND_NOW`会覆盖*dlopen()*中`RTLD_LAZY`标志的效果。

也可以在*flags*中包含更多的值。以下是 SUSv3 中指定的标志：

`RTLD_GLOBAL`

该库及其依赖树中的符号可以用于解析由该进程加载的其他库中的引用，并且也可以通过*dlsym()*进行查找。

`RTLD_LOCAL`

这是`RTLD_GLOBAL`的相反，且如果没有指定任何常量时为默认值。它指定该库及其依赖树中的符号不可用于解析随后的加载库中的引用。

如果没有指定`RTLD_GLOBAL`或`RTLD_LOCAL`，SUSv3 并未规定默认值。大多数 UNIX 实现假设与 Linux 相同的默认值（`RTLD_LOCAL`），但也有少数实现假设默认值为`RTLD_GLOBAL`。

Linux 还支持一些在 SUSv3 中未指定的标志：

`RTLD_NODELETE`（自*glibc* 2.2 起）

在*dlclose()*时不要卸载库，即使引用计数降为 0。这意味着如果库稍后通过*dlopen()*重新加载，库的静态变量不会被重新初始化。（对于由动态链接器自动加载的库，我们可以通过在创建库时指定*gcc -Wl,-znodelete*选项来实现类似效果。）

`RTLD_NOLOAD`（自*glibc* 2.2 起）

不加载库。这样做有两个目的。首先，我们可以使用这个标志检查某个特定库是否已作为进程地址空间的一部分被加载。如果加载了，*dlopen()* 会返回该库的句柄；如果没有，*dlopen()* 会返回`NULL`。其次，我们可以使用这个标志来“提升”已加载库的*flags*。例如，当对一个之前使用`RTLD_LOCAL`打开的库调用*dlopen()*时，我们可以在*flags*中指定`RTLD_NOLOAD | RTLD_GLOBAL`。

`RTLD_DEEPBIND`（自*glibc* 2.3.4 起）

在解析此库做出的符号引用时，首先在库中查找定义，再去查找已经加载的其他库中的定义。这允许库自包含，优先使用其自己的符号定义，而不是其他已加载共享库中具有相同名称的全局符号定义。（这类似于运行时符号解析中描述的*-Bsymbolic*链接选项的效果。）

`RTLD_NODELETE`和`RTLD_NOLOAD`标志也在 Solaris 的*dlopen* API 中实现，但在其他一些 UNIX 实现中很少可用。`RTLD_DEEPBIND`标志是 Linux 特有的。

作为特例，我们可以将*libfilename*指定为`NULL`。这会导致*dlopen()*返回主程序的句柄。（SUSv3 将其称为“全局符号对象”的句柄。）在随后的*dlopen()*调用中指定此句柄会使请求的符号首先在主程序中查找，然后在程序启动时加载的所有共享库中查找，最后在使用`RTLD_GLOBAL`标志动态加载的所有库中查找。

### 错误诊断：*dlerror()*

如果我们从*dlopen()*或*dlopen* API 中的其他函数接收到错误返回值，我们可以使用*dlerror()*获取指向字符串的指针，以指示错误的原因。

```
#include <dlfcn.h>

const char **dlerror*(void);
```

### 注意

返回指向错误诊断字符串的指针，或者如果自上次调用*dlerror()*以来没有发生错误，则返回`NULL`。

如果自上次调用*dlerror()*以来没有发生错误，*dlerror()*函数会返回`NULL`。我们将在下一节看到这如何有用。

### 获取符号的地址：*dlsym()*

*dlsym()* 函数在由 *handle* 引用的库以及该库的依赖树中的库中查找命名的 *symbol*（一个函数或变量）。

```
#include <dlfcn.h>

void *`dlsym`(void **handle*, char **symbol*);
```

### 注意

返回 *symbol* 的地址，如果 *symbol* 没有找到，则返回 `NULL`。

如果找到了 *symbol*，*dlsym()* 返回其地址；否则，*dlsym()* 返回 `NULL`。*handle* 参数通常是通过之前调用 *dlopen()* 返回的库句柄。或者，它可能是下面描述的所谓伪句柄之一。

### 注意

相关的函数 *dlvsym(handle, symbol, version)* 与 *dlsym()* 类似，但它可以用于在符号版本库中查找符号定义，并确保其版本与 *version* 中指定的字符串匹配。（我们在符号版本控制中描述了符号版本控制。）必须定义 `_GNU_SOURCE` 特性测试宏，才能从 `<dlfcn.h>` 获取该函数的声明。

*dlsym()* 返回的符号值可能是 `NULL`，这与“符号未找到”的返回值无法区分。为了区分这两种情况，我们必须事先调用 *dlerror()*（以确保清除任何之前的错误字符串），然后，如果在调用 *dlsym()* 后，*dlerror()* 返回一个非 `NULL` 的值，就说明发生了错误。

如果 *symbol* 是一个变量的名称，那么我们可以将 *dlsym()* 的返回值赋给一个适当的指针类型，并通过解引用该指针来获取变量的值：

```
int *ip;

ip = (int *) dlsym(symbol, "myvar");
if (ip != NULL)
    printf("Value is %d\n", *ip);
```

如果 *symbol* 是一个函数的名称，那么 *dlsym()* 返回的指针可以用来调用该函数。我们可以将 *dlsym()* 返回的值存储在适当类型的指针中，例如如下：

```
int (*funcp)(int);              /* Pointer to a function taking an integer
                                   argument and returning an integer */
```

然而，我们不能直接将 *dlsym()* 的结果赋值给这样的指针，如下所示的示例：

```
funcp = dlsym(handle, symbol);
```

原因是 C99 标准禁止在函数指针和 *void ** 之间进行赋值。解决方法是使用以下（稍显笨拙的）强制转换：

```
*(void **) (&funcp) = dlsym(handle, symbol);
```

使用 *dlsym()* 获取函数指针后，我们可以使用标准的 C 语法来解引用该函数指针，调用该函数：

```
res = (*funcp)(somearg);
```

替代上面所示的 **(void **)* 语法，当赋值 *dlsym()* 的返回值时，可以考虑使用以下看似等效的代码：

```
(void *) funcp = dlsym(handle, symbol);
```

然而，对于这段代码，*gcc -pedantic* 警告：“ANSI C 禁止将强制转换表达式用作左值。” **(void **)* 语法不会触发这个警告，因为我们是在为赋值的左值指向的地址 *赋值*。

在许多 UNIX 实现中，我们可以使用如下的强制转换来消除 C 编译器的警告：

```
funcp = (int (*) (int)) dlsym(handle, symbol);
```

然而，SUSv3 *技术修正第 1 号* 中对 *dlsym()* 的规范指出，C99 标准仍然要求编译器为这种转换生成警告，并提出了上述 **(void **)* 语法。

### 注意

SUSv3 TC1 指出，由于需要 **(void **)* 语法，标准的未来版本可能会定义单独的 *dlsym()* 类似 API 来处理数据和函数指针。然而，SUSv4 在这一点上没有任何变化。

#### 使用库伪句柄与 *dlsym()*

可以指定以下任一 *伪句柄* 作为 *dlsym()* 的 *handle* 参数，而不是指定由 *dlopen()* 调用返回的库句柄：

`RTLD_DEFAULT`

从主程序开始搜索 *symbol*，然后按照顺序搜索加载的所有共享库，包括通过带有 `RTLD_GLOBAL` 标志的 *dlopen()* 动态加载的库。这对应于动态链接器使用的默认搜索模型。

`RTLD_NEXT`

在调用 *dlsym()* 的库之后，搜索 *symbol*。这在创建与其他地方定义的函数同名的包装函数时非常有用。例如，在我们的主程序中，我们可能会定义自己的 *malloc()* 版本（可能会执行一些内存分配的记账），然后这个函数可以通过首先通过调用 *func = dlsym(RTLD_NEXT, “malloc”)* 获取真实的 *malloc()* 地址来调用实际的 *malloc()*。

上面列出的伪句柄值不是 SUSv3 所要求的（尽管它为未来使用保留了这些值），并且并非所有 UNIX 实现都支持它们。为了从 `<dlfcn.h>` 获取这些常量的定义，我们必须定义 `_GNU_SOURCE` 功能测试宏。

#### 示例程序

示例 42-1 演示了如何使用 *dlopen* API。该程序接受两个命令行参数：要加载的共享库的名称和在该库中要执行的函数的名称。以下示例展示了如何使用此程序：

```
$ `./dynload ./libdemo.so.1 x1`
Called mod1-x1
$ `LD_LIBRARY_PATH=. ./dynload libdemo.so.1 x1`
Called mod1-x1
```

在上述第一个命令中，*dlopen()* 注意到库路径包含斜杠，因此将其解释为相对路径名（在这种情况下，是指向当前工作目录中的库）。在第二个命令中，我们指定了 `LD_LIBRARY_PATH` 中的库搜索路径。这个搜索路径会根据动态链接器的常规规则进行解释（在这种情况下，同样是为了找到当前工作目录中的库）。

示例 42-1. 使用 *dlopen* API

```
`shlibs/dynload.c`
#include <dlfcn.h>
#include "tlpi_hdr.h"

int
main(int argc, char *argv[])
{
    void *libHandle;            /* Handle for shared library */
    void (*funcp)(void);        /* Pointer to function with no arguments */
    const char *err;

    if (argc != 3 || strcmp(argv[1], "--help") == 0)
        usageErr("%s lib-path func-name\n", argv[0]);

    /* Load the shared library and get a handle for later use */

    libHandle = dlopen(argv[1], RTLD_LAZY);
    if (libHandle == NULL)
        fatal("dlopen: %s", dlerror());

    /* Search library for symbol named in argv[2] */

    (void) dlerror();                           /* Clear dlerror() */
    *(void **) (&funcp) = dlsym(libHandle, argv[2]);
    err = dlerror();
    if (err != NULL)
        fatal("dlsym: %s", err);

    /* If the address returned by dlsym() is non-NULL, try calling it
       as a function that takes no arguments */

    if (funcp == NULL)
        printf("%s is NULL\n", argv[2]);
    else
        (*funcp)();

    dlclose(libHandle);                         /* Close the library */

    exit(EXIT_SUCCESS);
}

      `shlibs/dynload.c`
```

### 关闭共享库：*dlclose()*

*dlclose()* 函数用于关闭一个库。

```
#include <dlfcn.h>

int `dlclose`(void **handle*);
```

### 注意

成功时返回 0，错误时返回 -1

*dlclose()* 函数会递减系统对由 *handle* 引用的库的打开引用计数。如果该引用计数降为 0，并且该库中的符号不再被其他库需要，则该库将被卸载。此过程也会（递归地）对该库依赖树中的库进行处理。进程终止时，会隐式执行所有库的 *dlclose()* 操作。

### 注意

从 *glibc* 2.2.3 开始，共享库中的函数可以使用 *atexit()*（或 *on_exit()*）来建立一个在库被卸载时自动调用的函数。

### 获取已加载符号的信息：*dladdr()*

给定 *addr* 中的一个地址（通常是通过之前调用 *dlsym()* 获取的地址），*dladdr()* 会返回一个包含该地址信息的结构体。

```
#define _GNU_SOURCE
#include <dlfcn.h>

int `dladdr`(const void **addr*, Dl_info **info*);
```

### 注意

如果 *addr* 在共享库中找到了，则返回非零值，否则返回 0。

*info* 参数是指向调用者分配的结构体的指针，该结构体具有以下形式：

```
typedef struct {
    const char *dli_fname;          /* Pathname of shared library
                                       containing 'addr' */
    void       *dli_fbase;          /* Base address at which shared
                                       library is loaded */
    const char *dli_sname;          /* Name of nearest run-time symbol
                                       with an address <= 'addr' */
    void       *dli_saddr;          /* Actual value of the symbol
                                       returned in 'dli_sname' */
} Dl_info;
```

*Dl_info* 结构的前两个字段指定了包含 *addr* 指定地址的共享库的路径名和运行时基地址。最后两个字段返回该地址的相关信息。假设 *addr* 指向共享库中符号的准确地址，那么 *dli_saddr* 返回的值与传入的 *addr* 相同。

SUSv3 并没有规定 *dladdr()*，并且并非所有 UNIX 实现都提供这个函数。

### 访问主程序中的符号

假设我们使用 *dlopen()* 动态加载一个共享库，使用 *dlsym()* 获取该库中函数 *x()* 的地址，然后调用 *x()*。如果 *x()* 又调用了函数 *y()*，那么 *y()* 通常会在程序加载的某个共享库中被寻找。

有时，我们希望 *x()* 调用的是主程序中实现的 *y()*。这类似于回调机制。为了实现这一点，我们必须通过使用 *--export-dynamic* 链接器选项将主程序中的（全局作用域）符号提供给动态链接器：

```
$ `gcc -Wl,--export-dynamic main.c`     *(plus further options and arguments)*
```

等效地，我们可以写出以下代码：

```
$ `gcc -export-dynamic main.c`
```

使用这些选项中的任何一个，都可以让动态加载的库访问主程序中的全局符号。

### 注意

*gcc -rdynamic* 选项和 *gcc -Wl,-E* 选项是 *-Wl,--export-dynamic* 的其他同义词。

## 控制符号的可见性

一个设计良好的共享库应该只暴露那些构成其指定应用程序二进制接口（ABI）的一部分的符号（函数和变量）。其原因如下：

+   如果共享库设计者不小心导出了未指定的接口，那么使用该库的应用程序作者可能会选择使用这些接口。这就为共享库的未来升级带来了兼容性问题。库开发者希望能够更改或删除任何不属于文档化 ABI 的接口，而库用户希望继续使用他们当前使用的相同接口（保持相同语义）。

+   在运行时符号解析过程中，任何由共享库导出的符号可能会插入由其他共享库提供的定义（运行时符号解析）。

+   导出不必要的符号会增加动态符号表的大小，从而导致在运行时必须加载更多的内容。

如果库的设计者确保只导出库指定 ABI 所需的符号，所有这些问题都可以最小化甚至完全避免。可以使用以下技术来控制符号的导出：

+   在 C 程序中，我们可以使用`static`关键字将符号设为源代码模块的私有，从而使其无法被其他目标文件绑定。

### 注意

除了将符号设为源代码模块的私有外，`static`关键字还有一个相反的效果。如果一个符号被标记为`static`，那么同一源文件中对该符号的所有引用将绑定到该符号的定义。因此，这些引用不会受到其他共享库定义的运行时插入（如在运行时符号解析中所描述的方式）。`static`关键字的这个效果类似于运行时符号解析中描述的*-Bsymbolic*链接选项，区别在于`static`关键字只影响单个源文件中的单个符号。

+   GNU C 编译器，*gcc*，提供了一个编译器特定的属性声明，它执行的任务与`static`关键字类似：

    ```
    void
    __attribute__ ((visibility("hidden")))
    func(void) {
        /* Code */
    }
    ```

    而`static`关键字将符号的可见性限制在单一的源代码文件内，`hidden`属性则使符号在构成共享库的所有源代码文件中可用，但防止它在库外部可见。

    ### 注意

    与`static`关键字类似，`hidden`属性也有防止符号在运行时插入的相反效果。

+   版本脚本（链接器版本脚本）可以用来精确控制符号的可见性，并选择引用所绑定的符号版本。

+   在动态加载共享库时（打开共享库：*dlopen()*")），可以使用*dlopen()*的`RTLD_GLOBAL`标志来指定库定义的符号应当对后续加载的库可用，*—export-dynamic*链接选项（访问主程序中的符号）可以用来使主程序的全局符号在动态加载的库中可用。

关于符号可见性主题的更多详细信息，请参见[Drepper, 2004 (b)]。

## 链接器版本脚本

*版本脚本*是一个包含链接器*ld*指令的文本文件。为了使用版本脚本，我们必须指定*—version-script*链接选项：

```
$ `gcc -Wl,--version-script,```*`myscriptfile.map`*`` `...`

```

Version scripts are commonly (but not universally) identified using the extension `.map`.

The following sections describe some uses of version scripts.

### Controlling Symbol Visibility with Version Scripts

One use of version scripts is to control the visibility of symbols that might otherwise accidentally be made global (i.e., visible to applications linking against the library). As a simple example, suppose that we are building a shared library from the three source files `vis_comm.c`, `vis_f1.c`, and `vis_f2.c`, which respectively define the functions *vis_comm()*, *vis_f1()*, and *vis_f2()*. The *vis_comm()* function is called by *vis_f1()* and *vis_f2()*, but is not intended for direct use by applications linked against the library. Suppose we build the shared library in the usual way:

```

$ `gcc -g -c -fPIC -Wall vis_comm.c vis_f1.c vis_f2.c`

$ `gcc -g -shared -o vis.so vis_comm.o vis_f1.o vis_f2.o`

```

If we use the following *readelf* command to list the dynamic symbols exported by the library, we see the following:

```

$ `readelf --syms --use-dynamic vis.so | grep vis_`

30  12: 00000790    59    FUNC GLOBAL DEFAULT  10 vis_f1

25  13: 000007d0    73    FUNC GLOBAL DEFAULT  10 vis_f2

27  16: 00000770    20    FUNC GLOBAL DEFAULT  10 vis_comm

```

This shared library exported three symbols: *vis_comm()*, *vis_f1()*, and *vis_f2()*. However, we would like to ensure that only the symbols *vis_f1()* and *vis_f2()* are exported by the library. We can achieve this result using the following version script:

```

$ `cat vis.map`

VER_1 {

    global:

        vis_f1;

        vis_f2;

    local:

        *;

};

```

The identifier *VER_1* is an example of a *version tag*. As we’ll see in the discussion of symbol versioning in Symbol Versioning, a version script may contain multiple *version nodes*, each grouped within braces (`{}`) and prefixed with a unique version tag. If we are using a version script only for the purpose of controlling symbol visibility, then the version tag is redundant; nevertheless, older versions of *ld* required it. Modern versions of *ld* allow the version tag to be omitted; in this case, the version node is said to have an anonymous version tag, and no other version nodes may be present in the script.

Within the version node, the `global` keyword begins a semicolon-separated list of symbols that are made visible outside the library. The local keyword begins a list of symbols that are to be hidden from the outside world. The asterisk (*) here illustrates the fact that we can use wildcard patterns in these symbol specifications. The wildcard characters are the same as those used for shell filename matching—for example, `*` and `?`. (See the *glob(7)* manual page for further details.) In this example, using an asterisk for the `local` specification says that everything that wasn’t explicitly declared `global` is hidden. If we did not say this, then *vis_comm()* would still be visible, since the default is to make C global symbols visible outside the shared library.

We can then build our shared library using the version script as follows:

```

$ `gcc -g -c -fPIC -Wall vis_comm.c vis_f1.c vis_f2.c`

$ `gcc -g -shared -o vis.so vis_comm.o vis_f1.o vis_f2.o \`

        `-Wl,--version-script,vis.map`

```

Using *readelf* once more shows that *vis_comm()* is no longer externally visible:

```

$ `readelf --syms --use-dynamic vis.so | grep vis_`

25   0: 00000730    73    FUNC GLOBAL DEFAULT  11 vis_f2

29  16: 000006f0    59    FUNC GLOBAL DEFAULT  11 vis_f1

```

### Symbol Versioning

Symbol versioning allows a single shared library to provide multiple versions of the same function. Each program uses the version of the function that was current when the program was (statically) linked against the shared library. As a result, we can make an incompatible change to a shared library without needing to increase the library’s major version number. Carried to an extreme, symbol versioning can replace the traditional shared library major and minor versioning scheme. Symbol versioning is used in this manner in *glibc* 2.1 and later, so that all versions of *glibc* from 2.0 onward are supported within a single major library version (`libc.so.6`).

We demonstrate the use of symbol versioning with a simple example. We begin by creating the first version of a shared library using a version script:

```

$ `cat sv_lib_v1.c`

#include <stdio.h>

void xyz(void) { printf("v1 xyz\n"); }

$ `cat sv_v1.map`

VER_1 {

        global: xyz;

        local:  *;      # 隐藏所有其他符号

};

$ `gcc -g -c -fPIC -Wall sv_lib_v1.c`

$ `gcc -g -shared -o libsv.so sv_lib_v1.o -Wl,--version-script,sv_v1.map`

```

### Note

Within a version script, the hash character (`#`) starts a comment.

(To keep the example simple, we avoid the use of explicit library sonames and library major version numbers.)

At this stage, our version script, `sv_v1.map`, serves only to control the visibility of the shared library’s symbols; *xyz()* is exported, but all other symbols (of which there are none in this small example) are hidden. Next, we create a program, *p1*, which makes use of this library:

```

$ `cat sv_prog.c`

#include <stdlib.h>

int

main(int argc, char *argv[])

{

    void xyz(void);

    xyz();

    exit(EXIT_SUCCESS);

}

$ `gcc -g -o p1 sv_prog.c libsv.so`

```

When we run this program, we see the expected result:

```

$ `LD_LIBRARY_PATH=. ./p1`

v1 xyz

```

Now, suppose that we want to modify the definition of *xyz()* within our library, while still ensuring that program *p1* continues to use the old version of this function. To do this, we must define two versions of *xyz()* within our library:

```

$ `cat sv_lib_v2.c`

#include <stdio.h>

__asm__(".symver xyz_old,xyz@VER_1");

__asm__(".symver xyz_new,xyz@@VER_2");

void xyz_old(void) { printf("v1 xyz\n"); }

void xyz_new(void) { printf("v2 xyz\n"); }

void pqr(void) { printf("v2 pqr\n"); }

```

Our two versions of *xyz()* are provided by the functions *xyz_old()* and *xyz_new()*. The *xyz_old()* function corresponds to our original definition of *xyz()*, which is the one that should continue to be used by program *p1*. The *xyz_new()* function provides the definition of *xyz()* to be used by programs linking against the new version of the library.

The two `.symver` assembler directives are the glue that ties these two functions to different version tags in the modified version script (shown in a moment) that we use to create the new version of the shared library. The first of these directives says that *xyz_old()* is the implementation of *xyz()* to be used for applications linked against version tag *VER_1* (i.e., program *p1* in our example), and that *xyz_new()* is the implementation of *xyz()* to be used by applications linked against version tag *VER_2*.

The use of `@@` rather than `@` in the second `.symver` directive indicates that this is the default definition of *xyz()* to which applications should bind when statically linked against this shared library. Exactly one of the `.symver` directives for a symbol should be marked using `@@`.

The corresponding version script for our modified library is as follows:

```

$ `cat sv_v2.map`

VER_1 {

        global: xyz;

        local:  *;      # 隐藏所有其他符号

};

VER_2 {

        global: pqr;

} VER_1;

```

This version script provides a new version tag, *VER_2*, which depends on the tag *VER_1*. This dependency is indicated by the following line:

```

} VER_1;

```

Version tag dependencies indicate the relationships between successive library versions. Semantically, the only effect of version tag dependencies on Linux is that a version node inherits `global` and `local` specifications from the version node upon which it depends.

Dependencies can be chained, so that we could have another version node tagged *VER_3*, which depended on *VER_2*, and so on.

The version tag names have no meanings in themselves. Their relationship with one another is determined only by the specified version dependencies, and we chose the names *VER_1* and *VER_2* merely to be suggestive of these relationships. To assist maintenance, recommended practice is to use version tags that include the package name and a version number. For example, *glibc* uses version tags with names such as *GLIBC_2.0*, *GLIBC_2.1*, and so on.

The *VER_2* version tag also specifies that the new function *pqr()* is to be exported by the library and bound to the *VER_2* version tag. If we didn’t declare *pqr()* in this manner, then the `local` specification that *VER_2* version tag inherited from the *VER_1* version tag would make *pqr()* invisible outside the library. Note also that if we omitted the `local` specification altogether, then the symbols *xyz_old()* and *xyz_new()* would also be exported by the library (which is typically not what we want).

We now build the new version of our library in the usual way:

```

$ `gcc -g -c -fPIC -Wall sv_lib_v2.c`

$ `gcc -g -shared -o libsv.so sv_lib_v2.o -Wl,--version-script,sv_v2.map`

```

Now we can create a new program, *p2*, which uses the new definition of *xyz()*, while program *p1* uses the old version of *xyz()*.

```

$ `gcc -g -o p2 sv_prog.c libsv.so`

$ `LD_LIBRARY_PATH=. ./p2`

v2 xyz                                        *使用* *xyz@VER_2*

$ `LD_LIBRARY_PATH=. ./p1`

v1 xyz                                        *使用* *xyz@VER_1*

```

The version tag dependencies of an executable are recorded at static link time. We can use *objdump -t* to display the symbol tables of each executable, thus showing the different version tag dependencies of each program:

```

$ `objdump -t p1 | grep xyz`

08048380       F *UND*  0000002e              xyz@@VER_1

$ `objdump -t p2 | grep xyz`

080483a0       F *UND*  0000002e              xyz@@VER_2

```

We can also use *readelf -s* to obtain similar information.

### Note

Further information about symbol versioning can be found using the command *info ld scripts version* and at [`people.redhat.com/drepper/symbol-versioning`](http://people.redhat.com/drepper/symbol-versioning).

## Initialization and Finalization Functions

It is possible to define one or more functions that are executed automatically when a shared library is loaded and unloaded. This allows us to perform initialization and finalization actions when working with shared libraries. Initialization and finalization functions are executed regardless of whether the library is loaded automatically or loaded explicitly using the *dlopen* interface (Dynamically Loaded Libraries).

Initialization and finalization functions are defined using the *gcc* `constructor` and `destructor` attributes. Each function that is to be executed when the library is loaded should be defined as follows:

```

void __attribute__ ((constructor)) some_name_load(void)

{

    /* 初始化代码 */

}

```

Unload functions are similarly defined:

```

void __attribute__ ((destructor)) some_name_unload(void)

{

    /* 终结代码 */

}

```

The function names *some_name_load()* and *some_name_unload()* can be replaced by any desired names.

### Note

It is also possible to use the *gcc* `constructor` and `destructor` attributes to create initialization and finalization functions in a main program.

#### The *_init()* and *_fini()* functions

An older technique for shared library initialization and finalization is to create two functions, *_init()* and *_fini()*, as part of the library. The *void _init(void)* function contains code that is to executed when the library is first loaded by a process. The *void _fini(void)* function contains code that is to be executed when the library is unloaded.

If we create *_init()* and *_fini()* functions, then we must specify the *gcc -nostartfiles* option when building the shared library, in order to prevent the linker from including default versions of these functions. (Using the *-Wl,-init* and *-Wl,-fini* linker options, we can choose alternative names for these two functions if desired.)

Use of *_init()* and *_fini()* is now considered obsolete in favor of the *gcc* `constructor` and `destructor` attributes, which, among other advantages, allow us to define multiple initialization and finalization functions.

## Preloading Shared Libraries

For testing purposes, it can sometimes be useful to selectively override functions (and other symbols) that would normally be found by the dynamic linker using the rules described in Finding Shared Libraries at Run Time. To do this, we can define the environment variable `LD_PRELOAD` as a string consisting of space-separated or colon-separated names of shared libraries that should be loaded before any other shared libraries. Since these libraries are loaded first, any functions they define will automatically be used if required by the executable, thus overriding any other functions of the same name that the dynamic linker would otherwise have searched for. For example, suppose that we have a program that calls functions *x1()* and *x2()*, defined in our *libdemo* library. When we run this program, we see the following output:

```

$ `./prog`

调用 mod1-x1 DEMO

调用 mod2-x2 DEMO

```

(In this example, we assume that the shared library is in one of the standard directories, and thus we don’t need to use the `LD_LIBRARY_PATH` environment variable.)

We could selectively override the function *x1()* by creating another shared library, `libalt.so`, which contains a different definition of *x1()*. Preloading this library when running the program would result in the following:

```

$ `LD_PRELOAD=libalt.so ./prog`

调用 mod1-x1 ALT

调用 mod2-x2 DEMO

```

Here, we see that the version of *x1()* defined in `libalt.so` is invoked, but that the call to *x2()*, for which no definition is provided in `libalt.so`, results in the invocation of the *x2()* function defined in `libdemo.so`.

The `LD_PRELOAD` environment variable controls preloading on a per-process basis. Alternatively, the file `/etc/ld.so.preload`, which lists libraries separated by white space, can be used to perform the same task on a system-wide basis. (Libraries specified by `LD_PRELOAD` are loaded before those specified in `/etc/ld.so.preload`.)

For security reasons, set-user-ID and set-group-ID programs ignore `LD_PRELOAD`.

## Monitoring the Dynamic Linker: `LD_DEBUG`

Sometimes, it is useful to monitor the operation of the dynamic linker in order to know, for example, where it is searching for libraries. We can use the `LD_DEBUG` environment variable to do this. By setting this variable to one (or more) of a set of standard keywords, we can obtain various kinds of tracing information from the dynamic linker.

If we assign the value *help* to `LD_DEBUG`, the dynamic linker displays help information about `LD_DEBUG`, and the specified command is *not* executed:

```

$ `LD_DEBUG=help date`

LD_DEBUG 环境变量的有效选项包括：

libs       显示库搜索路径

reloc      显示重定位处理

files      显示输入文件的进度

symbols    显示符号表处理

bindings   显示符号绑定信息

versions   显示版本依赖关系

all        所有之前的选项组合

statistics 显示重定位统计信息

unused     确定未使用的 DSO

help       显示此帮助信息并退出

将调试输出定向到文件而非标准输出

可以使用 LD_DEBUG_OUTPUT 环境变量指定文件名。

```

The following example shows an abridged version of the output provided when we request tracing of information about library searches:

```

$ `LD_DEBUG=libs date`

    10687:     找到库=librt.so.1 [0]; 正在搜索

    10687:      搜索缓存=/etc/ld.so.cache

    10687:       尝试文件=/lib/librt.so.1

    10687:     找到库=libc.so.6 [0]; 正在搜索

    10687:      搜索缓存=/etc/ld.so.cache

    10687:       尝试文件=/lib/libc.so.6

    10687:     找到库=libpthread.so.0 [0]; 正在搜索

    10687:      搜索缓存=/etc/ld.so.cache

    10687:       尝试文件=/lib/libpthread.so.0

    10687:     调用 init: /lib/libpthread.so.0

    10687:     调用 init: /lib/libc.so.6

    10687:     调用 init: /lib/librt.so.1

    10687:     初始化程序：date

    10687:     转交控制：date

2010 年 12 月 28 日 星期二 17:26:56 CEST

    10687:     调用 fini: date [0]

    10687:     调用 fini: /lib/librt.so.1 [0]

    10687:     调用 fini: /lib/libpthread.so.0 [0]

    10687:     调用 fini: /lib/libc.so.6 [0]

```

每行开头显示的值 10687 是被追踪进程的进程 ID。如果我们在监控多个进程（例如父进程和子进程）时，这个信息非常有用。

默认情况下，`LD_DEBUG`的输出写入标准错误，但我们可以通过将路径名分配给`LD_DEBUG_OUTPUT`环境变量将其重定向到其他地方。

如果需要，我们可以通过逗号分隔（不要有空格）为`LD_DEBUG`分配多个选项。*symbols*选项（跟踪动态链接器的符号解析）的输出特别庞大。

`LD_DEBUG`对于动态链接器隐式加载的库和通过*dlopen()*动态加载的库都有效。

出于安全原因，`LD_DEBUG`（自*glibc* 2.2.5 起）在设置了用户 ID 和组 ID 的程序中会被忽略。

## 总结

动态链接器提供了*dlopen* API，它允许程序在运行时显式地加载额外的共享库。这使得程序能够实现插件功能。

共享库设计的一个重要方面是控制符号可见性，以便库只导出那些程序实际使用的符号（函数和变量）。我们研究了可以用来控制符号可见性的各种技术。其中一种技术是使用版本脚本，它提供了精细的符号可见性控制。

我们还展示了如何使用版本脚本实现一个方案，使得单一共享库可以为不同链接到该库的应用程序导出多个符号定义。（每个应用程序使用在静态链接时该应用程序所使用的当前符号定义。）这种技术提供了一个替代传统的库版本控制方法，即在共享库的实际名称中使用主版本号和次版本号。

在共享库中定义初始化和终结函数使我们能够在库被加载和卸载时自动执行代码。

`LD_PRELOAD` 环境变量允许我们预加载共享库。通过使用这一机制，我们可以有选择性地覆盖动态链接器通常会在其他共享库中找到的函数和其他符号。

我们可以为`LD_DEBUG`环境变量赋予不同的值，以监控动态链接器的操作。

#### 进一步信息

请参阅摘要中列出的进一步信息来源。

## 练习

1.  编写一个程序来验证，如果一个库通过*dlclose()*关闭，当其符号被另一个库使用时，该库不会被卸载。

1.  向示例 42-1（`dynload.c`）中的程序添加一个*dladdr()*调用，以便检索由*dlsym()*返回的地址的信息。打印出返回的*Dl_info*结构体字段的值，并验证它们是否符合预期。
