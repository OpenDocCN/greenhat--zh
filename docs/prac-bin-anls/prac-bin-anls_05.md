# 第四章：使用 LIBBFD 构建二进制加载器

现在，你已经通过前几章对二进制文件有了扎实的理解，准备开始构建自己的分析工具了。在本书中，你将经常构建自己的工具来操作二进制文件。由于几乎所有这些工具都需要解析并（静态地）加载二进制文件，因此拥有一个提供此功能的通用框架是非常有意义的。在这一章中，我们将使用`libbfd`来设计和实现这样的框架，以加深你对二进制格式的理解。

在本书的第三部分中，你将再次看到二进制加载框架，该部分涵盖了构建你自己二进制分析工具的高级技术。在设计框架之前，我将简要介绍`libbfd`。

### 4.1 什么是 libbfd？

二进制文件描述符库^(1)（`libbfd`）提供了一个通用接口，用于读取和解析所有流行的二进制格式，并为各种架构编译。这包括针对 x86 和 x86-64 机器的 ELF 和 PE 文件。通过将二进制加载器基于`libbfd`，你可以自动支持所有这些格式，而无需实现任何格式特定的支持。

BFD 库是 GNU 项目的一部分，并被`binutils`套件中的许多应用程序使用，包括`objdump`、`readelf`和`gdb`。它提供了对所有常见二进制格式组件的通用抽象，例如描述二进制目标和属性的头文件、节列表、重定位集合、符号表等。在 Ubuntu 中，`libbfd`是`binutils-dev`包的一部分。

你可以在*/usr/include/bfd.h*中找到核心的`libbfd` API。^(2) 不幸的是，`libbfd`的使用可能有些笨重，因此我们不打算在这里解释它的 API，而是直接深入探索 API，同时实现二进制加载框架。

### 4.2 一个简单的二进制加载接口

在实现二进制加载器之前，让我们先设计一个易于使用的接口。毕竟，二进制加载器的整个目的是使加载二进制文件的过程尽可能简单，以便后续所有你将在本书中实现的二进制分析工具都能使用。它主要用于静态分析工具。请注意，这与操作系统提供的动态加载器完全不同，后者的工作是将二进制文件加载到内存中以执行，如第一章中讨论的那样。

让我们使二进制加载接口与底层实现无关，这意味着它不会暴露任何`libbfd`函数或数据结构。为了简化，我们还将保持接口尽可能基础，仅暴露你在后续章节中经常使用的二进制部分。例如，接口将省略如重定位之类的组件，这些通常与二进制分析工具无关。

清单 4-1 显示了描述二进制加载器将公开的基本 API 的 C++ 头文件。请注意，它位于 VM 上的 *inc* 目录中，而不是包含本章其他代码的 *chapter4* 目录中。原因是加载器在本书的所有章节中是共享的。

*清单 4-1:* inc/loader.h

```
   #ifndef LOADER_H
   #define LOADER_H

   #include <stdint.h>
   #include <string>
   #include <vector>

   class Binary;
   class Section;
   class Symbol;

➊ class Symbol {
   public:
     enum SymbolType {
       SYM_TYPE_UKN = 0,
       SYM_TYPE_FUNC = 1
     };

     Symbol() : type(SYM_TYPE_UKN), name(), addr(0) {}

     SymbolType type;
     std::string name;
     uint64_t    addr;
   };

➋ class Section {
   public:
     enum SectionType {
       SEC_TYPE_NONE = 0,
       SEC_TYPE_CODE = 1,
       SEC_TYPE_DATA = 2
     };

     Section() : binary(NULL), type(SEC_TYPE_NONE),
                 vma(0), size(0), bytes(NULL) {}

     bool contains(uint64_t addr) { return (addr >= vma) && (addr-vma < size); }

     Binary         *binary;
     std::string     name;
     SectionType     type;
     uint64_t        vma;
     uint64_t        size;
     uint8_t         *bytes;
   };

➌ class Binary {
   public:
     enum BinaryType {
       BIN_TYPE_AUTO = 0,
       BIN_TYPE_ELF  = 1,
       BIN_TYPE_PE   = 2
     };
     enum BinaryArch {
       ARCH_NONE = 0,
       ARCH_X86 = 1
     };

     Binary() : type(BIN_TYPE_AUTO), arch(ARCH_NONE), bits(0), entry(0) {}

     Section *get_text_section()
       { for(auto &s : sections) if(s.name == ".text") return &s; return NULL; }

     std::string            filename;
     BinaryType             type;
     std::string            type_str;
     BinaryArch             arch;
     std::string            arch_str;
     unsigned               bits;
     uint64_t               entry;
     std::vector<Section>   sections;
     std::vector<Symbol>    symbols;
   };

➍ int load_binary(std::string &fname, Binary *bin, Binary::BinaryType type);
➎ void unload_binary(Binary *bin);

 #endif /* LOADER_H */
```

如你所见，API 暴露了表示二进制不同组件的多个类。`Binary` 类是“根”类，表示整个二进制的抽象 ➌。除此之外，它还包含一个 `Section` 对象的 `vector` 和一个 `Symbol` 对象的 `vector`。`Section` 类 ➋ 和 `Symbol` 类 ➊ 分别表示二进制文件中包含的节和符号。

从核心来看，整个 API 仅围绕两个函数展开。第一个是 `load_binary` 函数 ➍，它接受一个二进制文件的名称（`fname`）、一个指向 `Binary` 对象的指针用于存储加载的二进制文件（`bin`），以及一个二进制类型的描述符（`type`）。它将请求的二进制文件加载到 `bin` 参数中，并在加载成功时返回 0，若加载失败则返回小于 0 的值。第二个函数是 `unload_binary` ➎，它只是接受一个指向先前加载的 `Binary` 对象的指针并将其卸载。

现在你已经熟悉了二进制加载器的 API，接下来我们来看看它是如何实现的。我将从讨论 `Binary` 类的实现开始。

#### *4.2.1 Binary 类*

正如其名称所示，`Binary` 类是一个完整二进制文件的抽象。它包含二进制文件的文件名、类型、架构、位宽、入口点地址，以及节和符号。二进制类型具有双重表示：`type` 成员包含一个数字类型标识符，而 `type_str` 包含二进制类型的字符串表示。同样的双重表示也用于架构。

有效的二进制类型在 `enum BinaryType` 中列举，包括 ELF（`BIN_TYPE_ELF`）和 PE（`BIN_TYPE_PE`）。还有一个 `BIN_TYPE_AUTO`，你可以将其传递给 `load_binary` 函数，要求它自动判断二进制文件是 ELF 还是 PE 文件。类似地，有效的架构在 `enum BinaryArch` 中列举。对于这些目的，唯一有效的架构是 `ARCH_X86`。这包括 x86 和 x86-64；两者之间的区别由 `Binary` 类的 `bits` 成员表示，x86 设置为 32 位，x86-64 设置为 64 位。

通常，你可以通过分别迭代 `Binary` 类中的 `sections` 和 `symbols` 向量来访问节和符号。由于二进制分析通常关注 `.text` 节中的代码，因此还有一个名为 `get_text_section` 的便捷函数，顾名思义，它会自动查找并返回该节。

#### *4.2.2 Section 类*

段由`Section`类型的对象表示。`Section`类是一个简单的包装器，用于表示段的主要属性，包括段的名称、类型、起始地址（`vma`成员）、大小（以字节为单位）以及该段包含的原始字节。为了方便，还提供了一个指向包含`Section`对象的`Binary`的指针。段类型由`enum SectionType`值表示，指示该段是包含代码（`SEC_TYPE_CODE`）还是数据（`SEC_TYPE_DATA`）。

在分析过程中，你通常需要检查特定的指令或数据片段属于哪个段。因此，`Section`类有一个名为`contains`的函数，它接受一个代码或数据地址，并返回一个`bool`值，指示该地址是否属于该段。

#### *4.2.3 符号类*

如你所知，二进制文件包含许多类型的符号，包括本地和全局变量、函数、重定位表达式、对象等。为了简化，加载器接口只暴露了一种符号类型：函数符号。它们特别有用，因为当函数符号可用时，它们使得你可以轻松地实现函数级别的二进制分析工具。

加载器使用`Symbol`类来表示符号。该类包含一个符号类型，表示为`enum SymbolType`，其唯一有效值为`SYM_TYPE_FUNC`。此外，类还包含符号描述的函数的符号名称和起始地址。

### 4.3 实现二进制加载器

现在二进制加载器有了明确的接口，我们开始实现它吧！这就是`libbfd`发挥作用的地方。由于完整的加载器代码较长，我会将其分成几个部分，一一讨论。在以下代码中，你可以通过`bfd_`前缀识别`libbfd`的 API 函数（也有一些以`_bfd`结尾的函数，但它们是加载器定义的函数）。

首先，你当然需要包含所有需要的头文件。我不会提及加载器使用的所有标准 C/C++ 头文件，因为这些内容在这里不重要（如果你真的需要，可以在虚拟机上查看加载器的源码）。需要特别提到的是，所有使用`libbfd`的程序都必须包含*bfd.h*，如 Listing 4-2 所示，并通过指定链接器标志`-lbfd`来链接`libbfd`。除了*bfd.h*之外，加载器还包含了前一部分中创建的接口所在的头文件。

*Listing 4-2:* inc/loader.cc

```
#include <bfd.h>
#include "loader.h"
```

说到这，接下来要看的代码部分是`load_binary`和`unload_binary`，这是加载器接口暴露的两个入口函数。Listing 4-3 展示了这两个函数的实现。

*Listing 4-3:* inc/loader.cc *(续)*

```
  int
➊ load_binary(std::string &fname, Binary *bin, Binary::BinaryType type)
  {
    return ➋load_binary_bfd(fname, bin, type);
  }

  void
➌ unload_binary(Binary *bin)
  {
    size_t i;
    Section *sec;

➍ for(i = 0; i < bin->sections.size(); i++) {
     sec = &bin->sections[i];
     if(sec->bytes) {
➎      free(sec->bytes);
     }
    }
   }
```

`load_binary` ➊ 的工作是解析由文件名指定的二进制文件，并将其加载到传入的 `Binary` 对象中。这是一个有点繁琐的过程，因此 `load_binary` 明智地将这项工作推迟给另一个函数，叫做 `load_binary_bfd` ➋。稍后我会讨论这个函数。

首先，让我们看一下 `unload_binary` ➌。和许多事情一样，销毁一个 `Binary` 对象要比创建一个容易得多。为了卸载 `Binary` 对象，加载器必须释放（使用 `free`）所有 `Binary` 的动态分配组件。幸运的是，这些组件并不多：只有每个 `Section` 的 `bytes` 成员是动态分配的（使用 `malloc`）。因此，`unload_binary` 只需遍历所有 `Section` 对象 ➍，并为它们逐个释放 `bytes` 数组 ➎。现在你已经了解了卸载二进制文件的工作原理，让我们更详细地看看如何使用 `libbfd` 实现加载过程。

#### *4.3.1 初始化 libbfd 并打开二进制文件*

在上一节中，我承诺会向你展示 `load_binary_bfd`，这个函数使用 `libbfd` 来处理加载二进制文件的所有工作。在此之前，我得先处理一个先决条件。也就是说，要解析并加载二进制文件，你首先必须打开它。打开二进制文件的代码实现于一个名为 `open_bfd` 的函数中，具体代码见 Listing 4-4。

*Listing 4-4:* inc/loader.cc *(续)*

```
   static bfd*
   open_bfd(std::string &fname)
   {
     static int bfd_inited = 0;
     bfd *bfd_h;

     if(!bfd_inited) {
➊      bfd_init();
        bfd_inited = 1;
     }

➋   bfd_h = bfd_openr(fname.c_str(), NULL);
     if(!bfd_h) {
       fprintf(stderr, "failed to open binary '%s' (%s)\n",
               fname.c_str(), ➌bfd_errmsg(bfd_get_error()));
       return NULL;
     }
➍   if(!bfd_check_format(bfd_h, bfd_object)) {
       fprintf(stderr, "file '%s' does not look like an executable (%s)\n",
               fname.c_str(), bfd_errmsg(bfd_get_error()));
       return NULL;
     }

     /* Some versions of bfd_check_format pessimistically set a wrong_format
     * error before detecting the format and then neglect to unset it once
     * the format has been detected. We unset it manually to prevent problems.
     */
➎  bfd_set_error(bfd_error_no_error);

➏  if(bfd_get_flavour(bfd_h) == bfd_target_unknown_flavour) {
      fprintf(stderr, "unrecognized format for binary '%s' (%s)\n",
             fname.c_str(), bfd_errmsg(bfd_get_error()));
      return NULL;
    }

    return bfd_h;
  }
```

`open_bfd` 函数使用 `libbfd` 来确定由文件名（`fname` 参数）指定的二进制文件的属性，打开它，然后返回一个指向该二进制文件的句柄。在使用 `libbfd` 之前，你必须调用 `bfd_init` ➊ 来初始化 `libbfd` 的内部状态（或者像文档中所说的那样，初始化“神奇的内部数据结构”）。由于这只需要做一次，`open_bfd` 使用静态变量来跟踪初始化是否已经完成。

在初始化 `libbfd` 后，你调用 `bfd_openr` 函数，通过文件名打开二进制文件 ➋。`bfd_openr` 的第二个参数允许你指定目标（二进制文件的类型），但在本例中，我将其设置为 `NULL`，这样 `libbfd` 会自动确定二进制文件的类型。`bfd_openr` 的返回值是一个指向类型为 `bfd` 的文件句柄的指针；这是 `libbfd` 的根数据结构，你可以将其传递给所有其他 `libbfd` 函数来对二进制文件执行操作。如果发生错误，`bfd_openr` 会返回 `NULL`。

一般来说，每当发生错误时，你可以通过调用`bfd_get_error`来找到最近的错误类型。该函数返回一个`bfd_error_type`类型的对象，你可以将其与预定义的错误标识符进行比较，比如`bfd_error_no_memory`或`bfd_error_invalid_target`，从而判断如何处理该错误。通常，你可能只想退出并显示错误信息。为此，`bfd_errmsg`函数可以将`bfd_error_type`转换为描述错误的字符串，供你打印到屏幕上➌。

在获得二进制文件的句柄后，你应该使用`bfd_check_format`函数检查二进制文件的格式 ➍。该函数接受一个`bfd`句柄和一个`bfd_format`值，后者可以设置为`bfd_object`、`bfd_archive`或`bfd_core`。在这种情况下，加载器将其设置为`bfd_object`，以验证打开的文件是否确实是一个对象，在`libbfd`术语中，这意味着可执行文件、可重定位对象或共享库。

在确认处理的是`bfd_object`之后，加载器手动将`libbfd`的错误状态设置为`bfd_error_no_error`➎。这是对一些版本的`libbfd`中的一个问题的变通方法，这些版本在检测格式之前就设置了`bfd_error_wrong_format`错误，并且即使格式检测没有问题，也会保留该错误状态。

最后，加载器通过使用`bfd_get_flavour`函数检查二进制文件是否具有已知的“风味”➏。该函数返回一个`bfd_flavour`对象，表示二进制文件的类型（如 ELF、PE 等）。有效的`bfd_flavour`值包括`bfd_target_msdos_flavour`、`bfd_target_coff_flavour`和`bfd_target_elf_flavour`。如果二进制格式未知或发生错误，`get_bfd_flavour`将返回`bfd_target_unknown_flavour`，在这种情况下，`open_bfd`会打印错误并返回`NULL`。

如果所有检查都通过，说明你已成功打开一个有效的二进制文件，并准备开始加载其内容！`open_bfd`函数返回它所打开的`bfd`句柄，供你在后续的`libbfd` API 调用中使用，如下几个清单所示。

#### *4.3.2 解析基本二进制属性*

现在你已经看过了打开二进制文件所需的代码，是时候看一下`load_binary_bfd`函数了，见清单 4-5。回想一下，这是处理所有实际解析和加载工作的函数，代表`load_binary`函数。在本节中，目的是将有关二进制文件的所有有趣细节加载到由`bin`参数指向的`Binary`对象中。

*清单 4-5:* inc/loader.cc *(续)*

```
   static int
   load_binary_bfd(std::string &fname, Binary *bin, Binary::BinaryType type)
   {
     int ret;
     bfd *bfd_h;
     const bfd_arch_info_type *bfd_info;

     bfd_h = NULL;
➊   bfd_h = open_bfd(fname);
     if(!bfd_h) {
       goto fail;
     }

     bin->filename = std::string(fname);
➋   bin->entry    = bfd_get_start_address(bfd_h);

➌   bin->type_str = std::string(bfd_h->xvec->name);
➍   switch(bfd_h->xvec->flavour) {
     case bfd_target_elf_flavour:
       bin->type = Binary::BIN_TYPE_ELF;
       break;
    case bfd_target_coff_flavour:
      bin->type = Binary::BIN_TYPE_PE;
      break;
    case bfd_target_unknown_flavour:
    default:
      fprintf(stderr, "unsupported binary type (%s)\n", bfd_h->xvec->name);
      goto fail;
    }

➎     bfd_info = bfd_get_arch_info(bfd_h);
➏     bin->arch_str = std::string(bfd_info->printable_name);
➐     switch(bfd_info->mach) {
      case bfd_mach_i386_i386:
        bin->arch = Binary::ARCH_X86;
        bin->bits = 32;
        break;
      case bfd_mach_x86_64:
        bin->arch = Binary::ARCH_X86;
        bin->bits = 64;
        break;
      default:
        fprintf(stderr, "unsupported architecture (%s)\n",
                bfd_info->printable_name);
        goto fail;
      }

      /* Symbol handling is best-effort only (they may not even be present) */
➑    load_symbols_bfd(bfd_h, bin);
➒    load_dynsym_bfd(bfd_h, bin);

      if(load_sections_bfd(bfd_h, bin) < 0) goto fail;

      ret = 0;
      goto cleanup;

    fail:
      ret = -1;

    cleanup:
➓    if(bfd_h) bfd_close(bfd_h);

      return ret;
   }
```

`load_binary_bfd`函数首先使用刚刚实现的`open_bfd`函数打开`fname`参数指定的二进制文件，并获取一个指向该二进制文件的`bfd`句柄➊。然后，`load_binary_bfd`设置一些`bin`的基本属性。它首先复制二进制文件的名称，并使用`libbfd`查找并复制入口点地址➋。

要获取二进制文件的入口点地址，可以使用`bfd_get_start_address`，它简单地返回`bfd`对象中`start_address`字段的值。起始地址是一个`bfd_vma`，本质上就是一个 64 位无符号整数。

接下来，加载器收集有关二进制类型的信息：它是 ELF、PE 格式，还是其他不受支持的类型？你可以在`libbfd`维护的`bfd_target`结构中找到这些信息。要获取指向这个数据结构的指针，只需要访问`bfd`句柄中的`xvec`字段。换句话说，`bfd_h->xvec`给你一个指向`bfd_target`结构的指针。

除其他外，这个结构提供了一个包含目标类型名称的字符串。加载器将这个字符串复制到`Binary`对象中 ➌。接下来，它通过`switch`语句检查`bfd_h->xvec->flavour`字段，并根据该字段设置`Binary`的类型 ➍。加载器仅支持 ELF 和 PE 格式，因此如果`bfd_h->xvec->flavour`表示任何其他类型的二进制文件，它将产生错误。

现在你已经知道二进制文件是 ELF 还是 PE 格式，但还不知道它的架构。要找出这一点，可以使用`libbfd`的`bfd_get_arch_info`函数 ➎。顾名思义，这个函数返回一个指向数据结构的指针，该结构提供有关二进制架构的信息。这个数据结构被称为`bfd_arch_info_type`。它提供了一个方便的可打印字符串，描述了架构，加载器将这个字符串复制到`Binary`对象中 ➏。

`bfd_arch_info_type`数据结构还包含一个名为`mach`的字段 ➐，它只是一个表示架构的整数标识符（在`libbfd`术语中称为*machine*）。这种架构的整数表示允许使用方便的`switch`语句来实现特定架构的处理。如果`mach`等于`bfd_mach_i386_i386`，则表示它是一个 32 位 x86 二进制文件，加载器将相应地设置`Binary`中的字段。如果`mach`为`bfd_mach_x86_64`，则它是一个 x86-64 二进制文件，加载器再次设置相应的字段。任何其他类型都不受支持，并会导致错误。

现在你已经了解了如何解析有关二进制类型和架构的基本信息，是时候进行实际的工作了：加载二进制文件中包含的符号和段。正如你想象的那样，这并不像你到目前为止看到的那么简单，因此加载器将必要的工作推迟到专门的函数中，这些函数将在接下来的章节中描述。加载器用来加载符号的两个函数分别称为`load_symbols_bfd`和`load_dynsym_bfd` ➑。正如接下来章节所述，它们分别从静态和动态符号表中加载符号。加载器还实现了`load_sections_bfd`，这是一个专门用于加载二进制文件段的函数 ➒。我将在第 4.3.4 节中详细讨论它。

在加载完符号和段之后，你将把所有感兴趣的信息复制到你自己的`Binary`对象中，这意味着你已经完成了对`libbfd`的使用。因为`bfd`句柄不再需要，所以加载器使用`bfd_close` ➓关闭它。如果在完全加载二进制之前发生任何错误，它也会关闭句柄。

#### *4.3.3 加载符号*

清单 4-6 显示了`load_symbols_bfd`函数的代码，用于加载静态符号表。

*清单 4-6:* inc/loader.cc *(续)*

```
   static int
   load_symbols_bfd(bfd *bfd_h, Binary *bin)
   {
     int ret;
     long n, nsyms, i;
➊   asymbol **bfd_symtab;
     Symbol *sym;

     bfd_symtab = NULL;

➋    n = bfd_get_symtab_upper_bound(bfd_h);
     if(n < 0) {
       fprintf(stderr, "failed to read symtab (%s)\n",
               bfd_errmsg(bfd_get_error()));
       goto fail;
     } else if(n) {
➌      bfd_symtab = (asymbol**)malloc(n);
       if(!bfd_symtab) {
         fprintf(stderr, "out of memory\n");
        goto fail;
       }
➍     nsyms = bfd_canonicalize_symtab(bfd_h, bfd_symtab);
       if(nsyms < 0) {
         fprintf(stderr, "failed to read symtab (%s)\n",
                bfd_errmsg(bfd_get_error()));
         goto fail;
       }
➎     for(i = 0; i < nsyms; i++) {
➏       if(bfd_symtab[i]->flags & BSF_FUNCTION) {
           bin->symbols.push_back(Symbol());
           sym = &bin->symbols.back();
➐         sym->type = Symbol::SYM_TYPE_FUNC;
➑         sym->name = std::string(bfd_symtab[i]->name);
➒         sym->addr = bfd_asymbol_value(bfd_symtab[i]);
         }
       }
     }
     ret = 0;
     goto cleanup;

   fail:
     ret = -1;

   cleanup:
➓   if(bfd_symtab) free(bfd_symtab);

     return ret;

  }
```

在`libbfd`中，符号通过`asymbol`结构表示，实际上它是`struct bfd_symbol`的简称。反过来，符号表只是一个`asymbol**`，意味着一个指向符号的指针数组。因此，`load_symbols_bfd`的工作是填充在➊声明的`asymbol`指针数组，然后将感兴趣的信息复制到`Binary`对象中。

`load_symbols_bfd`的输入参数是一个`bfd`句柄和一个用于存储符号信息的`Binary`对象。在加载任何符号指针之前，你需要分配足够的空间来存储它们。`bfd_get_symtab_upper_bound`函数 ➋会告诉你为此分配多少字节。如果出现错误，字节数为负；如果为零，则表示没有符号表。如果没有符号表，`load_symbols_bfd`就会完成并直接返回。

如果一切正常，且符号表包含正字节数，你会分配足够的空间来存储所有的`asymbol`指针 ➌。如果`malloc`成功，你就可以准备好让`libbfd`来填充你的符号表！你可以通过`bfd_canonicalize_symtab`函数 ➍来实现，这个函数接受你的`bfd`句柄和你要填充的符号表（即你的`asymbol**`）作为输入。按照要求，`libbfd`将正确填充你的符号表，并返回它在表中放置的符号数量（如果该数字为负，则说明出现了问题）。

现在你已经有了填充的符号表，你可以遍历它包含的所有符号 ➎。回想一下，对于二进制加载器，你只对函数符号感兴趣。因此，对于每个符号，你检查是否设置了`BSF_FUNCTION`标志，这表示它是一个函数符号 ➏。若是这样，你就为`Binary`对象中的`Symbol`（回想一下，这是加载器自己用来存储符号的类）预留空间，通过向包含所有已加载符号的`vector`中添加条目来实现。你将新创建的`Symbol`标记为函数符号 ➐，复制符号名称 ➑，并设置`Symbol`的地址 ➒。要获取函数符号的值，即函数的起始地址，你可以使用`libbfd`提供的`bfd_asymbol_value`函数。

现在，所有有趣的符号都已被复制到`Symbol`对象中，加载器不再需要`libbfd`的表示。因此，当`load_symbols_bfd`完成时，它会释放为存储`libbfd`符号所保留的空间➓。之后，它返回，符号加载过程完成。

这就是如何通过`libbfd`从静态符号表加载符号的过程。那么，动态符号表是如何完成的呢？幸运的是，过程几乎完全相同，正如你在 Listing 4-7 中看到的那样。

*Listing 4-7:* inc/loader.cc *(续)*

```
   static int
   load_dynsym_bfd(bfd *bfd_h, Binary *bin)
   {
     int ret;
     long n, nsyms, i;
➊   asymbol **bfd_dynsym;
     Symbol *sym;

     bfd_dynsym = NULL;

➋   n = bfd_get_dynamic_symtab_upper_bound(bfd_h);
     if(n < 0) {
       fprintf(stderr, "failed to read dynamic symtab (%s)\n",
               bfd_errmsg(bfd_get_error()));
       goto fail;
     } else if(n) {
       bfd_dynsym = (asymbol**)malloc(n);
       if(!bfd_dynsym) {
         fprintf(stderr, "out of memory\n");
         goto fail;
      }
➌    nsyms = bfd_canonicalize_dynamic_symtab(bfd_h, bfd_dynsym);
      if(nsyms < 0) {
        fprintf(stderr, "failed to read dynamic symtab (%s)\n",
                bfd_errmsg(bfd_get_error()));
       goto fail;
     }
     for(i = 0; i < nsyms; i++) {
       if(bfd_dynsym[i]->flags & BSF_FUNCTION) {
         bin->symbols.push_back(Symbol());
         sym = &bin->symbols.back();
         sym->type = Symbol::SYM_TYPE_FUNC;
         sym->name = std::string(bfd_dynsym[i]->name);
         sym->addr = bfd_asymbol_value(bfd_dynsym[i]);
       }
      }
     }

     ret = 0;
     goto cleanup;

   fail:
     ret = -1;

   cleanup:
     if(bfd_dynsym) free(bfd_dynsym);

     return ret;
   }
```

在 Listing 4-7 中展示的从动态符号表加载符号的函数被恰当地命名为`load_dynsym_bfd`。如你所见，`libbfd`使用相同的数据结构（`asymbol`）来表示静态和动态符号➊。与之前展示的`load_symbols_bfd`函数的唯一区别如下。首先，为了找到你需要为符号指针保留的字节数，你调用`bfd_get_dynamic_symtab_upper_bound` ➋，而不是`bfd_get_symtab_upper_bound`。其次，为了填充符号表，你使用`bfd_canonicalize_dynamic_symtab` ➌，而不是`bfd_canonicalize_symtab`。就这些！其余的动态符号加载过程与静态符号的加载过程相同。

#### *4.3.4 加载节*

加载符号后，剩下的事情只有一件，尽管这可能是最重要的一步：加载二进制文件的节。Listing 4-8 展示了`load_sections_bfd`是如何实现这一功能的。

*Listing 4-8:* inc/loader.cc *(续)*

```
  static int
  load_sections_bfd(bfd *bfd_h, Binary *bin)
  {
    int bfd_flags;
    uint64_t vma, size;
    const char *secname;
➊  asection* bfd_sec;
    Section *sec;
    Section::SectionType sectype;

➋  for(bfd_sec = bfd_h->sections; bfd_sec; bfd_sec = bfd_sec->next) {
➌    bfd_flags = bfd_get_section_flags(bfd_h, bfd_sec);

      sectype = Section::SEC_TYPE_NONE;
➍    if(bfd_flags & SEC_CODE) {
        sectype = Section::SEC_TYPE_CODE;
      } else if(bfd_flags & SEC_DATA) {
        sectype = Section::SEC_TYPE_DATA;
      } else {
        continue;
      }
➎    vma     = bfd_section_vma(bfd_h, bfd_sec);
➏    size    = bfd_section_size(bfd_h, bfd_sec);
➐    secname = bfd_section_name(bfd_h, bfd_sec);
     if(!secname) secname = "<unnamed>";

➑    bin->sections.push_back(Section());
      sec = &bin->sections.back();

      sec->binary = bin;
      sec->name   = std::string(secname);
      sec->type   = sectype;
      sec->vma    = vma;
      sec->size   = size;
➒    sec->bytes  = (uint8_t*)malloc(size);
      if(!sec->bytes) {
        fprintf(stderr, "out of memory\n");
        return -1;
     }

➓   if(!bfd_get_section_contents(bfd_h, bfd_sec, sec->bytes, 0, size)) {
       fprintf(stderr, "failed to read section '%s' (%s)\n",
              secname, bfd_errmsg(bfd_get_error()));
       return -1;
     }
   }

   return 0;
 }
```

为了存储节，`libbfd`使用一种叫做`asection`的数据结构，也称为`struct bfd_section`。在内部，`libbfd`保持一个`asection`结构的链表来表示所有节。加载器保留一个`asection*`来遍历这个列表➊。

要遍历所有的节，你需要从第一个节开始（由`bfd_h->sections`指向，这是`libbfd`的节列表头），然后跟随每个`asection`对象中包含的`next`指针➋。当`next`指针为`NULL`时，你就到达了列表的末尾。

对于每个节，加载器首先检查是否应该加载它。由于加载器只加载代码和数据节，它首先获取节的标志来检查节的类型。为了获取标志，它使用`bfd_get_section_flags` ➌。然后，它检查是否设置了`SEC_CODE`或`SEC_DATA`标志 ➍。如果没有，它就跳过该节，继续处理下一个。如果设置了其中任一标志，则加载器为相应的`Section`对象设置节类型，并继续加载该节。

除了节类型，加载器还会复制每个代码或数据节的虚拟地址、大小（以字节为单位）、名称和原始字节。要找到`libbfd`节的虚拟基地址，可以使用`bfd_section_vma` ➎。类似地，可以使用`bfd_section_size` ➏和`bfd_section_name` ➐分别获取节的大小和名称。如果节没有名称，`bfd_section_name`将返回`NULL`。

现在，加载器将节的实际内容复制到`Section`对象中。为此，它在`Binary` ➑中保留一个`Section`，并复制它刚刚读取的所有字段。然后，它在`Section`的`bytes`成员中分配足够的空间来容纳节中的所有字节 ➒。如果`malloc`成功，它会使用`bfd_get_section_contents`函数 ➓将所有节字节从`libbfd`节对象复制到`Section`中。它所接受的参数包括`bfd`句柄、指向相关`asection`对象的指针、用于存储节内容的目标数组、复制的起始偏移量以及要复制的字节数。为了复制所有字节，起始偏移量为 0，复制字节的数量等于节的大小。如果复制成功，`bfd_get_section_contents`返回`true`；否则返回`false`。如果一切顺利，加载过程就完成了！

### 4.4 测试二进制加载器

让我们创建一个简单的程序来测试新的二进制加载器。该程序将接受一个二进制文件名作为输入，使用加载器加载该二进制文件，然后显示关于加载内容的一些诊断信息。清单 4-9 展示了测试程序的代码。

*清单 4-9:* loader_demo.cc

```
     #include <stdio.h>
     #include <stdint.h>
     #include <string>
     #include "../inc/loader.h"

     int
     main(int argc, char *argv[])
     {
       size_t i;
       Binary bin;
       Section *sec;
       Symbol *sym;
       std::string fname;

       if(argc < 2) {
         printf("Usage: %s <binary>\n", argv[0]);
         return 1;
     }

     fname.assign(argv[1]);
➊   if(load_binary(fname, &bin, Binary::BIN_TYPE_AUTO) < 0) {
       return 1;
     }

➋   printf("loaded binary '%s' %s/%s (%u bits) entry@0x%016jx\n",
           bin.filename.c_str(),
           bin.type_str.c_str(), bin.arch_str.c_str(),
           bin.bits, bin.entry);

➌   for(i = 0; i < bin.sections.size(); i++) {
       sec = &bin.sections[i];
       printf(" 0x%016jx %-8ju %-20s %s\n",
              sec->vma, sec->size, sec->name.c_str(),
              sec->type == Section::SEC_TYPE_CODE ? "CODE" : "DATA");
     }

➍   if(bin.symbols.size() > 0) {
       printf("scanned symbol tables\n");
       for(i = 0; i < bin.symbols.size(); i++) {
         sym = &bin.symbols[i];
         printf(" %-40s 0x%016jx %s\n",
                sym->name.c_str(), sym->addr,
                (sym->type & Symbol::SYM_TYPE_FUNC) ? "FUNC" : "");
       }
     }

➎   unload_binary(&bin);

     return 0;
    }
```

这个测试程序加载作为第一个参数传递给它的二进制文件 ➊，然后显示一些关于该二进制文件的基本信息，如文件名、类型、架构和入口点 ➋。接着，它会打印每个节的基地址、大小、名称和类型 ➌，最后显示所有找到的符号 ➍。然后，它会卸载二进制文件并返回 ➎。尝试在虚拟机中运行`loader_demo`程序！你应该看到类似于清单 4-10 的输出。

*清单 4-10: 加载器测试程序的示例输出*

```
$ loader_demo /bin/ls

loaded binary '/bin/ls' elf64-x86-64/i386:x86-64 (64 bits) entry@0x4049a0
  0x0000000000400238 28     .interp                DATA
  0x0000000000400254 32     .note.ABI-tag          DATA
  0x0000000000400274 36     .note.gnu.build-id     DATA
  0x0000000000400298 192    .gnu.hash              DATA
  0x0000000000400358 3288   .dynsym                DATA
  0x0000000000401030 1500   .dynstr                DATA
  0x000000000040160c 274    .gnu.version           DATA
  0x0000000000401720 112    .gnu.version_r         DATA
  0x0000000000401790 168    .rela.dyn              DATA
  0x0000000000401838 2688   .rela.plt              DATA
  0x00000000004022b8 26     .init                  CODE
  0x00000000004022e0 1808   .plt                   CODE
  0x00000000004029f0 8      .plt.got               CODE
  0x0000000000402a00 70281  .text                  CODE
  0x0000000000413c8c 9      .fini                  CODE
  0x0000000000413ca0 27060  .rodata                DATA
  0x000000000041a654 2060   .eh_frame_hdr          DATA
  0x000000000041ae60 11396  .eh_frame              DATA
  0x000000000061de00 8      .init_array            DATA
  0x000000000061de08 8      .fini_array            DATA
  0x000000000061de10 8      .jcr                   DATA
  0x000000000061de18 480    .dynamic               DATA
  0x000000000061dff8 8      .got                   DATA
  0x000000000061e000 920    .got.plt               DATA
  0x000000000061e3a0 608    .data                  DATA
scanned symbol tables
...
  _fini                     0x0000000000413c8c     FUNC
  _init                     0x00000000004022b8     FUNC
  free                      0x0000000000402340     FUNC
  _obstack_memory_used      0x0000000000412960     FUNC
  _obstack_begin            0x0000000000412780     FUNC
  _obstack_free             0x00000000004128f0     FUNC
  localtime_r               0x00000000004023a0     FUNC
  _obstack_allocated_p      0x00000000004128c0     FUNC
  _obstack_begin_1          0x00000000004127a0     FUNC
  _obstack_newchunk         0x00000000004127c0     FUNC
  malloc                    0x0000000000402790     FUNC
```

### 4.5 总结

在第一章到第三章中，你学习了有关二进制格式的所有内容。在本章中，你学习了如何加载这些二进制文件，为后续的二进制分析做准备。在这个过程中，你还了解了`libbfd`，这是一个常用的二进制加载库。现在你已经拥有了一个功能齐全的二进制加载器，准备继续学习二进制分析技术。在本书的第二部分中，你将学习一些基本的二进制分析技术，在第三部分中，你将使用加载器来实现自己的二进制分析工具。

习题

1\. 转储节内容

为了简洁，当前版本的`loader_demo`程序没有显示段内容。扩展程序，使其能够接受一个二进制文件和一个段名作为输入，然后以十六进制格式将该段的内容转储到屏幕上。

2\. 覆盖弱符号

有些符号是*弱的*，这意味着它们的值可能会被另一个非弱符号覆盖。目前，二进制加载器没有考虑这一点，而是简单地存储所有符号。扩展二进制加载器，使其在弱符号被其他符号覆盖时，仅保留最新版本。查看*/usr/include/bfd.h*以找出需要检查的标志。

3\. 打印数据符号

扩展二进制加载器和`loader_demo`程序，使它们能够处理本地和全局数据符号以及函数符号。你需要在加载器中添加数据符号的处理，向`Symbol`类中添加一个新的`SymbolType`，并在`loader_demo`程序中添加代码，以将数据符号打印到屏幕上。务必在一个未剥离的二进制文件上测试你的修改，以确保数据符号的存在。请注意，数据项在符号术语中被称为*对象*。如果你对输出的正确性有疑问，可以使用`readelf`来验证。
