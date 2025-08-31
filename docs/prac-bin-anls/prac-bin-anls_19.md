## 附录：B

使用`libelf`实现`PT_NOTE`覆盖

在第七章中，你学会了如何通过覆盖`PT_NOTE`段以高层次方式注入代码段。在这里，你将看到虚拟机中的`elfinject`工具如何实现这一技术。在描述`elfinject`源代码的过程中，你还将了解`libelf`，这是一个流行的开源库，用于操作 ELF 二进制文件的内容。

我将重点介绍实现图 7-2（第 170 页）中的步骤的代码部分，这些步骤使用了`libelf`，并省略一些直观且不涉及`libelf`的代码部分。如需了解更多，可以在虚拟机上的代码目录中找到`elfinject`的其余源代码，位于第七章。

在阅读本附录之前，请务必阅读第 7.3.2 节，因为了解`elfinject`期望的输入和输出将使代码更易于理解。

在本讨论中，我只会使用`elfinject`所使用的`libelf`API 部分，以帮助你理解`libelf`的基本要点。欲了解更多细节，请参考优秀的`libelf`文档，或参考 Joseph Koshy 的《`libelf`实例》一书。^(1)

### B.1 必需的头文件

为了解析 ELF 文件，`elfinject`使用了流行的开源库`libelf`，该库已经预装在虚拟机中，并且大多数 Linux 发行版都有这个包。要使用`libelf`，你需要包含一些头文件，如清单 B-1 所示。你还需要通过向链接器提供`-lelf`选项来链接`libelf`。

*清单 B-1:* elfinject.c*:* libelf *头文件*

```
➊ #include <libelf.h>
➋ #include <gelf.h>
```

为了简洁起见，清单 B-1 未显示`elfinject`使用的所有标准 C/C++头文件，而仅显示了两个与`libelf`相关的头文件。主要的头文件是*libelf.h* ➊，它提供了对`libelf`所有数据结构和 API 函数的访问。另一个是*gelf.h* ➋，它提供了对`GElf`的访问，`GElf`是一个辅助 API，简化了对`libelf`某些功能的访问。`GElf`使你能够透明地访问 ELF 文件，而无需关心文件的 ELF 类和位宽（32 位与 64 位）。这种方式的好处将在你看到更多`elfinject`代码时变得更为明显。

### B.2 `elfinject`中使用的数据结构

清单 B-2 展示了两个在`elfinject`中核心使用的数据结构。其余代码使用这些数据结构来操作 ELF 文件及注入的代码。

*清单 B-2:* elfinject.c*:* elfinject *数据结构*

```
➊ typedef struct {
     int fd;          /* file descriptor */
     Elf *e;          /* main elf descriptor */
     int bits;        /* 32-bit or 64-bit */
     GElf_Ehdr ehdr;  /* executable header */
   } elf_data_t;

➋ typedef struct {
     size_t pidx;     /* index of program header to overwrite */
     GElf_Phdr phdr;  /* program header to overwrite */
     size_t sidx;     /* index of section header to overwrite */
     Elf_Scn *scn;    /* section to overwrite */
     GElf_Shdr shdr;  /* section header to overwrite */
     off_t shstroff;  /* offset to section name to overwrite */
     char *code;      /* code to inject */
     size_t len;      /* number of code bytes */
     long entry;      /* code buffer offset to entry point (-1 for none) */
     off_t off;       /* file offset to injected code */
     size_t secaddr;  /* section address for injected code */
     char *secname;   /* section name for injected code */
   } inject_data_t;
```

第一个数据结构`elf_data_t`➊跟踪在注入新代码段的 ELF 二进制文件中需要操作的数据。它包含一个指向 ELF 文件的文件描述符（`fd`）、一个指向文件的`libelf`句柄、一个表示二进制文件位宽的整数（`bits`），以及指向二进制文件可执行头的`GElf`句柄。我将省略打开`fd`的标准 C 代码，因此从这一点开始，假设`fd`已经被打开用于读写。我稍后会展示打开`libelf`和`GElf`句柄的代码。

`inject_data_t`结构体➋跟踪有关要注入的代码以及如何在二进制文件中注入这些代码的信息。首先，它包含有关需要修改二进制文件哪些部分来注入新代码的数据。这些数据包括要覆盖的`PT_NOTE`程序头的索引（`pidx`）和`GElf`句柄（`phdr`）。它还包括要覆盖的段的索引（`sidx`）以及`libelf`和`GElf`句柄（分别是`scn`和`shdr`），以及指向该段名称在字符串表中的文件偏移量（`shstroff`），以便将其更改为一个新名称，比如`.injected`。

然后是实际注入的代码，以缓冲区（`code`）和描述该缓冲区长度的整数（`len`）的形式给出。这段代码由`elfinject`用户提供，因此从这一点开始，假设`code`和`len`已经被设置。`entry`字段是`code`缓冲区内的一个偏移量，指向应该成为二进制文件新入口点的代码位置。如果没有新的入口点，那么`entry`被设置为`-1`来表示这一点。

`off`字段是二进制文件中应注入新代码的文件偏移量。这个偏移量将指向二进制文件的末尾，因为`elfinject`会将新代码放置在此位置，如图 7-2 所示。最后，`secaddr`是新代码段的加载地址，`secname`是被注入段的名称。你可以认为从`entry`到`secname`的所有字段都已经被设置，因为这些都是用户指定的，除了`off`，它是`elfinject`在加载二进制文件时计算的。

### B.3 初始化 libelf

此时，我们跳过`elfinject`的初始化代码，假设所有初始化都已成功：用户参数已经解析，主机二进制文件的文件描述符已打开，注入文件已加载到`struct inject_data_t`中的代码缓冲区。所有这些初始化工作都在`elfinject`的`main`函数中进行。

之后，`main`将控制权传递给一个名为`inject_code`的函数，这是实际代码注入的起点。让我们来看一下清单 B-3，其中展示了`inject_code`的一部分，负责在`libelf`中打开给定的 ELF 二进制文件。请记住，函数名称以`elf_`开头的是`libelf`函数，而以`gelf_`开头的是`GElf`函数。

*清单 B-3:* elfinject.c*:* inject_code *函数*

```
   int
   inject_code(int fd, inject_data_t *inject)
   {
➊  elf_data_t elf;
    int ret;
    size_t n;

    elf.fd = fd;
    elf.e = NULL;

➋   if(elf_version(EV_CURRENT) == EV_NONE) {
      fprintf(stderr, "Failed to initialize libelf\n");
      goto fail;
    }

     /* Use libelf to read the file, but do writes manually */
➌   elf.e = elf_begin(elf.fd, ELF_C_READ, NULL);
     if(!elf.e) {
       fprintf(stderr, "Failed to open ELF file\n");
       goto fail;
     }

➍   if(elf_kind(elf.e) != ELF_K_ELF) {
        fprintf(stderr, "Not an ELF executable\n");
        goto fail;
     }

➎   ret = gelf_getclass(elf.e);
     switch(ret) {
     case ELFCLASSNONE:
       fprintf(stderr, "Unknown ELF class\n");
       goto fail;
     case ELFCLASS32:
       elf.bits = 32;
       break;
     default:
       elf.bits = 64;
       break;
     }

   ...
```

`inject_code` 函数中的一个重要局部变量，`elf` ➊ 是先前定义的 `elf_data_t` 结构类型的一个实例，用于存储加载的 ELF 二进制文件的所有重要信息，并将其传递给其他函数。

在使用任何其他 `libelf` API 函数之前，必须调用 `elf_version` ➋，该函数接受一个 ELF 规范的版本号作为唯一参数。如果版本不受支持，`libelf` 会通过返回常量 `EV_NONE` 来报告问题，在这种情况下，`inject_code` 会放弃并报告初始化 `libelf` 时出错。如果 `libelf` 没有报告问题，说明请求的 ELF 版本是受支持的，接下来可以安全地进行其他 `libelf` 调用，以加载和解析二进制文件。

目前，所有标准 ELF 二进制文件都是根据规范的主版本 1 格式进行格式化的，因此这是你可以传递给 `elf_version` 的唯一合法值。按照约定，除了直接传递字面量的“1”给 `elf_version`，你还可以传递常量值 `EV_CURRENT`。`EV_NONE` 和 `EV_CURRENT` 都在 *elf.h* 中进行了定义，而不是在 *libelf.h* 中。若 ELF 格式有重大修订，`EV_CURRENT` 将在使用新 ELF 版本的系统上递增为下一个版本。

在 `elf_version` 成功返回后，可以开始加载并解析二进制文件，以便将新代码注入其中。第一步是调用 `elf_begin` ➌，该函数打开 ELF 文件并返回一个类型为 `Elf*` 的句柄。你可以将该句柄传递给其他 `libelf` 函数，执行对 ELF 文件的操作。

`elf_begin` 函数接受三个参数：用于打开 ELF 文件的文件描述符，一个常量，表示是否以读或写模式打开文件，以及指向 `Elf` 句柄的指针。在这种情况下，文件描述符为 `fd`，而 `inject_code` 传递常量 `ELF_C_READ`，表示它仅仅对使用 `libelf` 读取 ELF 二进制文件感兴趣。对于最后一个参数（`Elf` 句柄），`inject_code` 传递 `NULL`，以便 `libelf` 自动分配并返回一个句柄。

你也可以传递 `ELF_C_WRITE` 或 `ELF_C_RDWR`，以表示希望使用 `libelf` 向 ELF 二进制文件写入修改，或者进行读写操作的组合。为了简化，`elfinject` 仅使用 `libelf` 来解析 ELF 文件。为了将任何修改写回，它绕过 `libelf`，直接使用文件描述符 `fd`。

在使用 `libelf` 打开 ELF 文件后，通常会将打开的 `Elf` 句柄传递给 `elf_kind`，以确定所处理的 ELF 类型 ➍。在这种情况下，`inject_code` 将 `elf_kind` 的返回值与常量 `ELF_K_ELF` 进行比较，验证 ELF 文件是否为可执行文件。其他可能的返回值为 `ELF_K_AR`（表示 ELF 存档文件）或 `ELF_K_NULL`（表示发生错误）。在这两种情况下，`inject_code` 无法执行代码注入，因此会返回错误。

接下来，`inject_code` 使用一个名为 `gelf_getclass` 的 `GElf` 函数来获取 ELF 二进制文件的“类” ➎。这表示 ELF 文件是 32 位（`ELFCLASS32`）还是 64 位（`ELFCLASS64`）。如果发生错误，`gelf_getclass` 会返回 `ELFCLASSNONE`。`ELFCLASS*` 常量在 *elf.h* 中定义。目前，`inject_code` 只将二进制文件的位宽（32 位或 64 位）存储在 `elf` 结构的 `bits` 字段中。了解位宽在解析 ELF 二进制文件时是必要的。

以上就是初始化 `libelf` 并获取二进制文件基本信息的过程。接下来我们来看 `inject_code` 函数的其他部分，参见清单 B-4。

*清单 B-4:* elfinject.c*:* inject_code *函数（续）*

```
   ...

➊  if(!gelf_getehdr(elf.e, &elf.ehdr)) {
      fprintf(stderr, "Failed to get executable header\n");
      goto fail;
   }

   /* Find a rewritable program header */
➋  if(find_rewritable_segment(&elf, inject) < 0) {
     goto fail;
   }

   /* Write the injected code to the binary */
➌  if(write_code(&elf, inject) < 0) {
     goto fail;
   }

   /* Align code address so it's congruent to the file offset modulo 4096 */
➍  n = (inject->off % 4096) - (inject->secaddr % 4096);
   inject->secaddr += n;

   /* Rewrite a section for the injected code */
➎  if((rewrite_code_section(&elf, inject) < 0)
        || ➏(rewrite_section_name(&elf, inject) < 0)) {
       goto fail;
   }

   /* Rewrite a segment for the added code section */
➐  if(rewrite_code_segment(&elf, inject) < 0) {
       goto fail;
   }

   /* Rewrite entry point if requested */
➑  if((inject->entry >= 0) && (rewrite_entry_point(&elf, inject) < 0)) {
     goto fail;
   }

   ret = 0;
   goto cleanup;
 fail:
     ret = -1;

   cleanup:
     if(elf.e) {
➒      elf_end(elf.e);
     }

     return ret;
   }
```

如你所见，`inject_code` 函数的其余部分包括几个主要步骤，这些步骤对应于图 7-2 中列出的步骤，以及一些图中未显示的额外低级步骤：

• 获取二进制可执行文件的头部 ➊，后续需要用来调整入口点。

• 查找 `PT_NOTE` 段 ➋ 进行覆盖，如果没有合适的段则失败。

• 将注入的代码写入二进制文件的末尾 ➌。

• 调整注入部分的加载地址，以满足对齐要求 ➍。

• 用新注入部分的头部覆盖 `.note.ABI-tag` 节头 ➎。

• 更新被覆盖的部分头部的节名称 ➏。

• 覆盖 `PT_NOTE` 程序头 ➐。

• 如果用户要求，调整二进制文件的入口点 ➑。

• 通过调用 `elf_end` 清理 `Elf` 句柄 ➒。

接下来我将更详细地讲解这些步骤。

### B.4 获取可执行文件头

在清单 B-4 的步骤 ➊ 中，`elfinject` 获取了二进制可执行文件的头部。回想一下第二章，可执行文件头包含了这些表格的文件偏移量和大小。可执行文件头还包含了二进制文件的入口点地址，`elfinject` 会根据用户的需求修改这个入口点地址。

要获取 ELF 可执行文件头，`elfinject` 使用 `gelf_getehdr` 函数。这是一个 `GElf` 函数，它返回一个与 ELF 类无关的可执行文件头表示。32 位和 64 位二进制文件的可执行文件头格式略有不同，但 `GElf` 隐藏了这些差异，因此你无需担心这些问题。也可以仅使用纯 `libelf` 获取可执行文件头，而不使用 `GElf`。但是，在这种情况下，你必须根据 ELF 类手动调用 `elf32_getehdr` 或 `elf64_getehdr`。

`gelf_getehdr` 函数接受两个参数：`Elf` 句柄和一个指向 `GElf_Ehdr` 结构的指针，`GElf` 可以在其中存储可执行文件头。如果一切正常，`gelf_getehdr` 返回非零值。如果发生错误，它返回 0 并设置 `elf_errno`，这是一个错误代码，你可以通过调用 `libelf` 的 `elf_errno` 函数来读取该错误代码。此行为是所有 `GElf` 函数的标准行为。

要将 `elf_errno` 转换为人类可读的错误消息，你可以使用 `elf_errmsg` 函数，但 `elfinject` 并没有这么做。`elf_errmsg` 函数接受 `elf_errno` 的返回值作为输入，并返回一个指向适当错误字符串的 `const char*`。

### B.5 查找 PT_NOTE 段

在获取可执行文件头后，`elfinject` 会遍历二进制文件中的所有程序头，检查是否存在一个可以安全覆盖的 `PT_NOTE` 段（清单 B-4 中的步骤 ➋）。所有这些功能都在一个名为 `find_rewritable_segment` 的单独函数中实现，见清单 B-5。

*清单 B-5:* elfinject.c*: 查找* PT_NOTE *程序头*

```
   int
   find_rewritable_segment(elf_data_t *elf, inject_data_t *inject)
   {
     int ret;
     size_t i, n;

➊  ret = elf_getphdrnum(elf->e, &n);
    if(ret != 0) {
       fprintf(stderr, "Cannot find any program headers\n");
       return -1;
   }

➋  for(i = 0; i < n; i++) {
➌    if(!gelf_getphdr(elf->e, i, &inject->phdr)) {
        fprintf(stderr, "Failed to get program header\n");
        return -1;
    }

➍  switch(inject->phdr.p_type) {
    case ➎PT_NOTE:
      ➏inject->pidx = i;
      return 0;
    default:
      break;
    }
   }
➐ fprintf(stderr, "Cannot find segment to rewrite\n");
   return -1;
  }
```

如清单 B-5 所示，`find_rewritable_segment` 接受两个参数：一个名为 `elf` 的 `elf_data_t*` 和一个名为 `inject` 的 `inject_data_t*`。回忆一下，这些是自定义数据类型，在清单 B-2 中定义，包含有关 ELF 二进制文件和注入的所有相关信息。

为了找到 `PT_NOTE` 段，`elfinject` 首先查找二进制文件中包含的程序头数量 ➊。通过使用一个名为 `elf_getphdrnum` 的 `libelf` 函数来实现，它接受两个参数：`Elf` 句柄和一个指向 `size_t` 类型整数的指针，用于存储程序头的数量。如果返回值非零，则表示发生了错误，`elfinject` 会放弃，因为它无法访问程序头表。如果没有错误，`elf_getphdrnum` 会将程序头的数量存储在清单 B-5 中的 `size_t` 类型变量 `n` 中。

现在`elfinject`知道程序头的数量`n`，它遍历每个程序头以查找类型为`PT_NOTE`的头部➋。为了访问每个程序头，`elfinject`使用`gelf_getphdr`函数➌，该函数允许以与 ELF 类无关的方式访问程序头。它的参数是`Elf`句柄、要获取的程序头的索引号`i`以及一个指向`GElf_Phdr`结构体的指针（在本例中为`inject->phdr`），用于存储程序头。像`GElf`函数一样，非零返回值表示成功，而返回值 0 表示失败。

在此步骤完成后，`inject->phdr`包含第`i`个程序头。剩下的就是检查程序头的`p_type`字段➍，并检查其类型是否为`PT_NOTE`➎。如果是，`elfinject`将程序头索引存储在`inject->pidx`字段中➏，并且`find_rewritable_segment`函数成功返回。

如果在遍历所有程序头后，`elfinject`未能找到类型为`PT_NOTE`的头部，它会报告错误➐并退出，而不修改二进制文件。

### B.6 注入代码字节

在定位到可覆盖的`PT_NOTE`段后，就可以将注入的代码追加到二进制文件中（列表 B-4 中的步骤➌）。让我们来看一下执行实际注入操作的函数，它叫做`write_code`，如列表 B-6 所示。

*列表 B-6:* elfinject.c*: 将注入的代码追加到二进制文件中*

```
   int
   write_code(elf_data_t *elf, inject_data_t *inject)
   {
     off_t off;
     size_t n;
➊   off = lseek(elf->fd, 0, SEEK_END);
     if(off < 0) {
       fprintf(stderr, "lseek failed\n");
       return -1;
   }

➋  n = write(elf->fd, inject->code, inject->len);
    if(n != inject->len) {
      fprintf(stderr, "Failed to inject code bytes\n");
      return -1;
   }
➌  inject->off = off;

    return 0;
  }
```

就像你在前一节中看到的`find_rewritable_segment`函数一样，`write_code`函数将`elf_data_t*`类型的`elf`和`inject_data_t*`类型的`inject`作为参数。`write_code`函数不涉及`libelf`，它仅在打开的 ELF 二进制文件的文件描述符`elf->fd`上使用标准的 C 文件操作。

首先，`write_code`将光标定位到二进制文件的末尾➊。然后，它在那里追加注入的代码字节➋，并将代码字节写入的字节偏移量保存到`inject->off`字段中➌。

现在代码注入已经完成，剩下的就是更新一个段头和程序头（可选地更新二进制的入口点），以描述新注入的代码段，并确保在二进制执行时加载它。

### B.7 对注入段的加载地址进行对齐

随着注入的代码字节被追加到二进制文件的末尾，现在几乎可以覆盖一个段头以指向这些注入的字节。ELF 规范对可加载段的地址以及它们包含的段提出了一些要求。具体来说，ELF 标准要求对于每个可加载段，`p_vaddr`与`p_offset`在页大小（4,096 字节）上的模运算结果必须相等。以下公式总结了这一要求：

(*p*_*vaddr* mod 4096) =  (*p*_*offset* mod 4096)

类似地，ELF 标准要求 `p_vaddr` 与 `p_offset` 在 `p_align` 模数下是同余的。因此，在覆盖节头之前，`elfinject` 会调整用户指定的注入部分内存地址，使其满足这些要求。清单 B-7 显示了对齐地址的代码，这与清单 B-4 中步骤 ➍ 所示的代码相同。

*清单 B-7:* elfinject.c*: 对注入部分的加载地址进行对齐*

```
    /* Align code address so it's congruent to the file offset modulo 4096 */
➊  n = (inject->off % 4096) - (inject->secaddr % 4096);
➋  inject->secaddr += n;
```

清单 B-7 中的对齐代码包括两个步骤。首先，它计算注入代码的文件偏移量模 4096 与节地址模 4096 之间的差值 `n` ➊。ELF 规范要求偏移量和地址在模 4096 下是同余的，此时 `n` 将为零。为了确保正确的对齐，`elfinject` 将 `n` 加到节地址中，以便文件偏移量与节地址之间的差值在模 4096 下变为零（如果还没有的话）➋。

### B.8 覆盖 `.note.ABI-tag` 节头

现在已经知道了注入部分的地址，`elfinject` 继续覆盖节头。回想一下，它覆盖了 `.note.ABI-tag` 节头，该节头是 `PT_NOTE` 段的一部分。清单 B-8 显示了处理覆盖的函数，名为 `rewrite_code_section`。它在清单 B-4 的步骤 ➎ 中被调用。

*清单 B-8:* elfinject.c*: 覆盖 * .note.ABI-tag * 节头*

```
  int
  rewrite_code_section(elf_data_t *elf, inject_data_t *inject)
  {
    Elf_Scn *scn;
    GElf_Shdr shdr;
    char *s;
    size_t shstrndx;

➊   if(elf_getshdrstrndx(elf->e, &shstrndx) < 0) {
      fprintf(stderr, "Failed to get string table section index\n");
      return -1;
    }

    scn = NULL;
➋   while((scn = elf_nextscn(elf->e, scn))) {
➌     if(!gelf_getshdr(scn, &shdr)) {
        fprintf(stderr, "Failed to get section header\n");
        return -1;
       }
➍     s = elf_strptr(elf->e, shstrndx, shdr.sh_name);
      if(!s) {
        fprintf(stderr, "Failed to get section name\n");
        return -1;
      }
 ➎     if(!strcmp(s, ".note.ABI-tag")) {
➏       shdr.sh_name      = shdr.sh_name;              /* offset into string table */
        shdr.sh_type      = SHT_PROGBITS;               /* type */
        shdr.sh_flags     = SHF_ALLOC | SHF_EXECINSTR;  /* flags */
        shdr.sh_addr      = inject->secaddr;            /* address to load section at */
        shdr.sh_offset    = inject->off;                /* file offset to start of section */
        shdr.sh_size      = inject->len;                /* size in bytes */
        shdr.sh_link      = 0;                          /* not used for code section */
        shdr.sh_info      = 0;                          /* not used for code section */
        shdr.sh_addralign = 16;                         /* memory alignment */
        shdr.sh_entsize   = 0                           /* not used for code section */

➐       inject->sidx = elf_ndxscn(scn);
        inject->scn = scn;
        memcpy(&inject->shdr, &shdr, sizeof(shdr));

➑       if(write_shdr(elf, scn, &shdr, elf_ndxscn(scn)) < 0) {
             return -1;
        }

➒       if(reorder_shdrs(elf, inject) < 0) {
             return -1;
        }

        break;
      }
    }
➓   if(!scn) {
      fprintf(stderr, "Cannot find section to rewrite\n");
      return -1;
     }

     return 0;
   }
```

为了找到需要覆盖的 `.note.ABI-tag` 节头，`rewrite_code_section` 会循环遍历所有节头并检查节名称。回想一下，在第二章中提到过，节名称存储在一个名为 `.shstrtab` 的特殊节中。为了读取节名称，`rewrite_code_section` 首先需要获取描述 `.shstrtab` 节的节头的索引号。要获取这个索引，可以读取可执行文件头的 `e_shstrndx` 字段，或者可以使用 `libelf` 提供的 `elf_getshdrstrndx` 函数。清单 B-8 使用了后一种选项 ➊。

`elf_getshdrstrndx` 函数接受两个参数：一个 `Elf` 句柄和一个指向 `size_t` 类型整数的指针，用于存储节索引。该函数在成功时返回 0，失败时设置 `elf_errno` 并返回 -1。

获取`.shstrtab`的索引后，`rewrite_code_section`会循环遍历所有节头，逐一检查。在循环遍历节头时，它使用`elf_nextscn`函数 ➋，该函数接受`Elf`句柄（`elf->e`）和`Elf_Scn*`（`scn`）作为输入。`Elf_Scn`是由`libelf`定义的结构，描述了一个 ELF 节。最初，`scn`为`NULL`，这导致`elf_nextscn`返回指向节头表中索引 1 的第一个节头的指针。^(2) 这个指针成为`scn`的新值，并在循环体中处理。在下一次循环迭代中，`elf_nextscn`接受现有的`scn`并返回指向索引 2 的节的指针，依此类推。通过这种方式，你可以使用`elf_nextscn`遍历所有节，直到它返回`NULL`，表示没有下一个节。

循环体处理由`elf_nextscn`返回的每个节`scn`。对每个节执行的第一件事是使用`gelf_getshdr`函数 ➌获取该节的与 ELF 类无关的表示。它的工作方式与第 B.5 节中学习的`gelf_getphdr`类似，只是`gelf_getshdr`接受`Elf_Scn*`和`GElf_Shdr*`作为输入。如果一切顺利，`gelf_getshdr`将用给定`Elf_Scn`的节头填充给定的`GElf_Shdr`并返回指向该头的指针。如果出现问题，它将返回`NULL`。

使用存储在`elf->e`中的`Elf`句柄、`.shstrtab`节的索引`shstrndx`以及当前节名称在字符串表中的索引`shdr.sh_name`，`elfinject`现在获取指向描述当前节名称的字符串的指针。为此，它将所有必需的信息传递给`elf_strptr`函数 ➍，该函数返回指针，如果发生错误，则返回`NULL`。

接下来，`elfinject`将刚获得的节名称与字符串`".note.ABI-tag"` ➎进行比较。如果匹配，则表示当前节是`.note.ABI-tag`节，`elfinject`会按照接下来的描述覆盖该节，然后跳出循环并从`rewrite_code_section`成功返回。如果节名称不匹配，循环将进入下一次迭代，检查下一个节是否匹配。

如果当前节的名称是`.note.ABI-tag`，`rewrite_code_section`将覆盖节头中的字段，将其转变为描述注入节的头 ➏。正如在图 7-2 中的高级概述所提到的，这涉及将节类型设置为`SHT_PROGBITS`；将节标记为可执行；并填写适当的节地址、文件偏移、大小和对齐方式。

接下来，`rewrite_code_section`将覆盖的节头的索引、指向`Elf_Scn`结构的指针以及`GElf_Shdr`的副本保存在`inject`结构中 ➐。为了获取节的索引，它使用`elf_ndxscn`函数，该函数以`Elf_Scn*`为输入，并返回该节的索引。

一旦头部修改完成，`rewrite_code_section`使用另一个名为`write_shdr` ➑的`elfinject`函数将修改后的节头写回 ELF 二进制文件，然后按节地址重新排序节头 ➒。接下来我将讨论`write_shdr`函数，跳过对`reorder_shdrs`函数的描述，后者负责排序节，因为它对于理解`PT_NOTE`覆盖技术并不是核心内容。

如前所述，如果`elfinject`成功找到并覆盖了`.note.ABI-tag`节头，它会从遍历所有节头的主循环中跳出，并成功返回。另一方面，如果循环完成而没有找到可以覆盖的节头，则注入过程无法继续，`rewrite_code_section`会以错误 ➓ 返回。

列表 B-9 展示了`write_shdr`的代码，这是负责将修改后的节头写回 ELF 文件的函数。

*列表 B-9：* elfinject.c*：将修改后的节头写回二进制文件*

```
  int
  write_shdr(elf_data_t *elf, Elf_Scn *scn, GElf_Shdr *shdr, size_t sidx)
  {
    off_t off;
    size_t n, shdr_size;
    void *shdr_buf;

➊   if(!gelf_update_shdr(scn, shdr)) {
      fprintf(stderr, "Failed to update section header\n");
      return -1;
    }

➋   if(elf->bits == 32) {
➌     shdr_buf = elf32_getshdr(scn);
       shdr_size = sizeof(Elf32_Shdr);
    } else {
➍     shdr_buf = elf64_getshdr(scn);
      shdr_size = sizeof(Elf64_Shdr);
    }

    if(!shdr_buf) {
      fprintf(stderr, "Failed to get section header\n");
      return -1;
    }

➎   off = lseek(elf->fd, elf->ehdr.e_shoff + sidx*elf->ehdr.e_shentsize, SEEK_SET);
    if(off < 0) {
      fprintf(stderr, "lseek failed\n");
      return -1;
    }

➏   n = write(elf->fd, shdr_buf, shdr_size);
    if(n != shdr_size) {
      fprintf(stderr, "Failed to write section header\n");
      return -1;
    }

    return 0;
  }
```

`write_shdr`函数接受三个参数：存储读取和写入 ELF 二进制所需所有重要信息的`elf_data_t`结构，名为`elf`；一个`Elf_Scn*`（`scn`）和一个`GElf_Shdr*`（`shdr`），它们对应于需要覆盖的节；以及该节在节头表中的索引（`sidx`）。

首先，`write_shdr`调用`gelf_update_shdr` ➊。回顾一下，`shdr`包含所有头字段中的新覆盖值。由于`shdr`是一个与 ELF 类无关的`GElf_Shdr`结构，它是`GElf` API 的一部分，写入它并不会自动更新底层的 ELF 数据结构（`Elf32_Shdr`或`Elf64_Shdr`，具体取决于 ELF 类）。然而，正是这些底层数据结构是`elfinject`写入 ELF 二进制文件的目标，因此必须确保它们被更新。`gelf_update_shdr`函数接受一个`Elf_Scn*`和一个`GElf_Shdr*`作为输入，并将对`GElf_Shdr`所做的任何更改写回到底层的数据结构，这些数据结构是`Elf_Scn`结构的一部分。`elfinject`写入底层数据结构而不是`GElf`数据结构的原因在于，`GElf`数据结构内部使用的内存布局与文件中数据结构的布局不匹配，因此写入`GElf`数据结构会破坏 ELF 文件。

现在，`GElf` 已将所有待处理的更新写回到底层的本机 ELF 数据结构中，`write_shdr` 获取更新后的节头的本机表示，并将其写入 ELF 文件，覆盖旧的 `.note.ABI-tag` 节头。首先，`write_shdr` 检查二进制文件的位宽 ➋。如果是 32 位，那么 `write_shdr` 调用 `libelf` 的 `elf32_getshdr` 函数（并传递 `scn`）来获取指向修改后的头部的 `Elf32_Shdr` 表示的指针 ➌。对于 64 位二进制文件，则使用 `elf64_getshdr` ➍，而不是 `elf32_getshdr`。

接下来，`write_shdr` 将 ELF 文件描述符（`elf->fd`）定位到 ELF 文件中要写入更新后头部的偏移量 ➎。请记住，执行文件头中的 `e_shoff` 字段包含节头表开始的文件偏移量，`sidx` 是要覆盖的头部的索引，`e_shentsize` 字段包含节头表中每个条目的字节大小。因此，以下公式计算出写入更新后的节头的文件偏移量：

*e*_*shoff* + *sidx* × *e*_*shentsize*

在定位到此文件偏移量后，`write_shdr` 将更新后的节头写入 ELF 文件 ➏，用描述注入节的新节头覆盖旧的 `.note.ABI-tag` 节头。此时，新的代码字节已经被注入到 ELF 二进制文件的末尾，并且有一个新的代码节包含这些字节，但该节在字符串表中还没有一个有意义的名称。下一节将解释 `elfinject` 如何更新节名称。

### B.9 设置注入节的名称

列表 B-10 显示了将被覆盖的节 `.note.ABI-tag` 的名称更改为更有意义的名称，例如 `.injected` 的函数。这是 列表 B-4 中的步骤 ➏。

*列表 B-10:* elfinject.c*: 设置注入节的名称*

```
  int
  rewrite_section_name(elf_data_t *elf, inject_data_t *inject)
  {
    Elf_Scn *scn;
    GElf_Shdr shdr;
    char *s;
    size_t shstrndx, stroff, strbase;

➊   if(strlen(inject->secname) > strlen(".note.ABI-tag")) {
       fprintf(stderr, "Section name too long\n");
       return -1;
    }

➋   if(elf_getshdrstrndx(elf->e, &shstrndx) < 0) {
       fprintf(stderr, "Failed to get string table section index\n");
       return -1;
    }

    stroff = 0;
    strbase = 0;
    scn = NULL;
➌   while((scn = elf_nextscn(elf->e, scn))) {
➍     if(!gelf_getshdr(scn, &shdr)) {
         fprintf(stderr, "Failed to get section header\n");
         return -1;
      }
➎     s = elf_strptr(elf->e, shstrndx, shdr.sh_name);
       if(!s) {
         fprintf(stderr, "Failed to get section name\n");
         return -1;
      }

➏     if(!strcmp(s, ".note.ABI-tag")) {
         stroff = shdr.sh_name;    /* offset into shstrtab */
➐      } else if(!strcmp(s, ".shstrtab")) {
         strbase = shdr.sh_offset; /* offset to start of shstrtab */
       }
    }
 ➑   if(stroff == 0) {
      fprintf(stderr, "Cannot find shstrtab entry for injected section\n");
      return -1;
    } else if(strbase == 0) {
      fprintf(stderr, "Cannot find shstrtab\n");
      return -1;
    }

➒   inject->shstroff = strbase + stroff;

➓   if(write_secname(elf, inject) < 0) {
       return -1;
    }

    return 0;
  }
```

用于覆盖节名称的函数名为 `rewrite_section_name`。这个注入节的新名称不能比旧名称 `.note.ABI-tag` 更长，因为字符串表中的所有字符串紧密打包在一起，没有多余的空间来容纳额外的字符。因此，`rewrite_section_name` 首先会检查存储在 `inject->secname` 字段中的新节名称是否适合 ➊。如果不适合，`rewrite_section_name` 会返回错误。

接下来的步骤与我之前讨论的 `rewrite_code_section` 函数中的相应步骤相同，见 列表 B-8：获取字符串表节的索引 ➋，然后遍历所有节 ➌ 并检查每个节的节头 ➍，使用节头中的 `sh_name` 字段来获取指向节名称的字符串指针 ➎。有关这些步骤的详细信息，请参阅 B.8 节。

覆盖旧的 `.note.ABI-tag` 段名需要两个信息：`.shstrtab` 段（字符串表）开始的文件偏移量，以及 `.note.ABI-tag` 段名在字符串表中的偏移量。给定这两个偏移量，`rewrite_section_name` 就知道在哪里在文件中写入新的段名字符串。`.note.ABI-tag` 段名在字符串表中的偏移量保存在 `.note.ABI-tag` 段头的 `sh_name` 字段中 ➏。类似地，段头中的 `sh_offset` 字段包含 `.shstrtab` 段的起始位置 ➐。

如果一切顺利，循环会定位到两个所需的偏移量 ➑。如果没有，`rewrite_section_name` 会报告错误并放弃。

最后，`rewrite_section_name` 计算写入新段名的文件偏移量，并将其保存在 `inject->shstroff` 字段中 ➒。然后，它调用另一个名为 `write_secname` 的函数，将新段名写入 ELF 二进制文件中，写入位置是刚刚计算出的偏移量 ➓。写入段名到文件是直接的，只需要标准的 C 文件 I/O 函数，因此我在这里省略了对 `write_secname` 函数的描述。

回顾一下，ELF 二进制文件现在包含了注入的代码、被覆盖的段头，以及为注入段设置的正确名称。下一步是覆盖 `PT_NOTE` 程序头，创建一个包含注入段的可加载段。

### B.10 覆盖 PT_NOTE 程序头

如你所记得，列表 B-5 展示了定位并保存 `PT_NOTE` 程序头以进行覆盖的代码。剩下的工作就是覆盖相关的程序头字段，并将更新后的程序头保存到文件中。列表 B-11 展示了更新并保存程序头的函数 `rewrite_code_segment`。这个函数在 列表 B-4 中的步骤 ➐ 被调用。

*列表 B-11:* elfinject.c*：覆盖* PT_NOTE *程序头*

```
  int
  rewrite_code_segment(elf_data_t *elf, inject_data_t *inject)
  {
➊   inject->phdr.p_type   = PT_LOAD;          /* type */
➋   inject->phdr.p_offset = inject->off;      /* file offset to start of segment */
    inject->phdr.p_vaddr   = inject->secaddr;  /* virtual address to load segment at */
    inject->phdr.p_paddr   = inject->secaddr;  /* physical address to load segment at */
    inject->phdr.p_filesz  = inject->len;      /* byte size in file */
    inject->phdr.p_memsz   = inject->len;      /* byte size in memory */
➌   inject->phdr.p_flags  = PF_R | PF_X;      /* flags */
➍   inject->phdr.p_align  = 0x1000;           /* alignment in memory and file */

➎   if(write_phdr(elf, inject) < 0) {
       return -1;
    }

    return 0;
  }
```

请记住，之前定位的 `PT_NOTE` 程序头保存在 `inject->phdr` 字段中。因此，`rewrite_code_segment` 首先更新此程序头中的必要字段：通过将 `p_type` 设置为 `PT_LOAD` ➊ 使其可加载；设置注入代码段的文件偏移量、内存地址和大小 ➋；使段可读并可执行 ➌；并设置正确的对齐方式 ➍。这些修改与在 图 7-2 中展示的高级概述相同。

在进行必要的修改后，`rewrite_code_segment` 调用另一个名为 `write_phdr` 的函数，将修改后的程序头写回 ELF 二进制文件 ➎。列表 B-12 展示了 `write_phdr` 的代码。该代码与 `write_shdr` 函数类似，后者是将修改后的段头写入文件的函数，你已经在 列表 B-9 中看过，因此我将重点介绍 `write_phdr` 和 `write_shdr` 之间的重要区别。

*列表 B-12:* elfinject.c*：将覆盖的程序头写回 ELF 文件*

```
   int
   write_phdr(elf_data_t *elf, inject_data_t *inject)
   {
     off_t off;
     size_t n, phdr_size;
     Elf32_Phdr *phdr_list32;
     Elf64_Phdr *phdr_list64;
     void *phdr_buf;

➊   if(!gelf_update_phdr(elf->e, inject->pidx, &inject->phdr)) {
       fprintf(stderr, "Failed to update program header\n");
       return -1;
     }

     phdr_buf = NULL;
➋   if(elf->bits == 32) {
➌     phdr_list32 = elf32_getphdr(elf->e);
       if(phdr_list32) {
➍        phdr_buf = &phdr_list32[inject->pidx];
          phdr_size = sizeof(Elf32_Phdr);
       }
     } else {
       phdr_list64 = elf64_getphdr(elf->e);
       if(phdr_list64) {
         phdr_buf = &phdr_list64[inject->pidx];
         phdr_size = sizeof(Elf64_Phdr);
       }
     }
     if(!phdr_buf) {
       fprintf(stderr, "Failed to get program header\n");
       return -1;
     }

➎   off = lseek(elf->fd, elf->ehdr.e_phoff + inject->pidx*elf->ehdr.e_phentsize, SEEK_SET);
     if(off < 0) {
       fprintf(stderr, "lseek failed\n");
       return -1;
    }

➏  n = write(elf->fd, phdr_buf, phdr_size);
    if(n != phdr_size) {
      fprintf(stderr, "Failed to write program header\n");
      return -1;
    }

    return 0;
  }
```

与 `write_shdr` 函数类似，`write_phdr` 首先确保将对程序头的 `GElf` 表示所做的所有修改写回到底层的本地 `Elf32_Phdr` 或 `Elf64_Phdr` 数据结构 ➊。为此，`write_phdr` 调用 `gelf_update_phdr` 函数，以便将更改刷新到底层数据结构中。此函数接受一个 ELF 句柄、修改的程序头的索引和指向更新后的 `GElf_Phdr` 程序头表示的指针。像往常一样，对于 `GElf` 函数，成功时返回非零值，失败时返回 0。

接下来，`write_phdr` 获取对相关程序头本地表示的引用（根据 ELF 类，可能是 `Elf32_Phdr` 或 `Elf64_Phdr` 结构），并将其写入文件 ➋。这个过程类似于你在 `write_shdr` 函数中看到的，唯一不同的是 `libelf` 不允许你直接获取特定程序头的指针。相反，你必须先获取指向程序头表开始位置的指针 ➌，然后通过索引获取指向更新后的程序头的指针 ➍。要获取指向程序头表的指针，可以根据 ELF 类使用 `elf32_getphdr` 或 `elf64_getphdr` 函数。它们在成功时返回指针，在失败时返回 `NULL`。

鉴于被覆盖的 ELF 程序头的本地表示，现在所需要做的就是寻找到正确的文件偏移量 ➎ 并将更新后的程序头写入该位置 ➏。这完成了将新代码段注入 ELF 二进制文件中的所有强制步骤！剩下的唯一步骤是可选的：修改 ELF 入口点，使其指向注入的代码。

### B.11 修改入口点

列表 B-13 显示了 `rewrite_entry_point` 函数，该函数负责修改 ELF 入口点。只有在 列表 B-4 的第 ➑ 步用户请求时，才会调用此函数。

*列表 B-13:* elfinject.c*：修改 ELF 入口点*

```
   int
   rewrite_entry_point(elf_data_t *elf, inject_data_t *inject)
   {
➊   elf->ehdr.e_entry = inject->phdr.p_vaddr + inject->entry;
➋   return write_ehdr(elf);
  }
```

回想一下，`elfinject` 允许用户通过命令行参数指定一个新的二进制文件入口点，该参数包含指向注入代码的偏移量。用户指定的偏移量保存在 `inject->entry` 字段中。如果偏移量为负，则表示入口点不应改变，在这种情况下，`rewrite_entry_point` 永远不会被调用。因此，如果调用了 `rewrite_entry_point`，则可以保证 `inject->entry` 为非负值。

`rewrite_entry_point` 做的第一件事是更新 ELF 可执行文件头中的 `e_entry` 字段 ➊，该字段之前已加载到 `elf->ehdr` 字段中。接下来，它通过将注入代码的相对偏移量（`inject->entry`）加到包含注入代码的可加载段的基地址（`inject->phdr.p_vaddr`）来计算新的入口点地址。然后，`rewrite_entry_point` 调用专用函数 `write_ehdr` ➋，该函数将修改后的可执行文件头写回 ELF 文件。

`write_ehdr` 的代码与 清单 B-9 中展示的 `write_shdr` 函数类似。唯一的区别是，它使用 `gelf_update_ehdr` 代替了 `gelf_update_shdr`，并且使用 `elf32_getehdr`/`elf64_getehdr` 代替了 `elf32_getshdr`/`elf64_getshdr`。

现在你已经知道如何使用 `libelf` 将代码注入二进制文件、覆盖一个节区和程序头以容纳新代码，并修改 ELF 入口点，以便在加载二进制时跳转到注入的代码！修改入口点是可选的，你可能并不总是希望在二进制启动时立即使用注入的代码。有时候，你会希望因不同的原因使用注入的代码，例如替代现有函数的实现。第 7.4 节讨论了除了修改 ELF 入口点之外，一些将控制权转移到注入代码的技巧。
