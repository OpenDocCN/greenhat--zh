# 第十一章：外部函数接口

![](img/chapterart.png)

不是所有的代码都是用 Rust 编写的。这是令人震惊的，我知道。时不时，你需要与其他语言编写的代码进行交互，可能是通过从 Rust 调用这些代码，或者允许这些代码调用你的 Rust 代码。你可以通过*外部函数接口（FFI）*来实现这一点。

在本章中，我们将首先了解 Rust 为 FFI 提供的主要机制：`extern`关键字。我们将看到如何使用`extern`将 Rust 函数和静态变量暴露给其他语言，以及如何让 Rust 访问外部提供的函数和静态变量。然后，我们将介绍如何将 Rust 类型与其他语言中定义的类型对齐，并探讨一些让数据流经 FFI 边界的复杂性。最后，我们将讨论一些在进行大量 FFI 操作时，你可能会用到的工具。

## 跨越边界与 extern

FFI 的核心目标是访问来自你应用程序 Rust 代码之外的字节。为此，Rust 提供了两个主要构建块：*符号*，它们是分配给你二进制文件中某一段特定地址的名称，允许你在外部来源与 Rust 代码之间共享内存（无论是数据还是代码）；以及*调用约定*，它们提供了一种共同的理解，说明如何调用存储在这些共享内存中的函数。我们将依次了解这两个构建块。

### 符号

编译器从你的代码生成的任何二进制文件都会充满符号——你定义的每个函数或静态变量都有一个符号，指向它在编译后二进制文件中的位置。通用函数甚至可能有多个符号，每个符号对应编译器生成的该函数的不同单态化版本！

通常，你不需要考虑符号——它们是由编译器内部使用，用来传递二进制文件中函数或静态变量的最终地址。这就是编译器知道在生成最终机器代码时，每个函数调用应该定位到内存中的哪个位置，或者当你的代码访问静态变量时应该从哪里读取。由于你通常不会在代码中直接引用符号，编译器默认会为它们选择半随机名称——你可能在代码的不同部分有两个名为`foo`的函数，但编译器会为它们生成不同的符号，从而避免混淆。

然而，当你想调用一个函数或访问一个未同时编译的静态变量时，使用随机名称的符号是行不通的，例如那些用不同语言编写并因此由不同编译器编译的代码。如果符号的名称是半随机的并且不断变化，你无法将一个 C 语言中定义的静态变量告诉 Rust。反之，如果你无法为 Rust 函数提供一个稳定的名称，你也无法通过 Python 的 FFI 接口来访问 Rust 函数。

为了使用具有外部来源的符号，我们还需要某种方式告诉 Rust 该变量或函数的存在，以便编译器会查找在其他地方定义的相同符号，而不是定义自己的符号（稍后我们会讨论这种查找是如何进行的）。否则，我们最终会得到两个相同的符号用于那个函数或静态变量，且无法共享。实际上，很可能编译会失败，因为任何引用该符号的代码都不知道该使用哪个定义（即，哪个地址）。

#### 编译与链接的简要说明

编译器速成课程！了解将代码转换为可运行二进制文件的复杂过程的粗略概念，将有助于你更好地理解 FFI。你看，编译器并不是一个庞大的程序，而是（通常）被拆分成几个小程序，每个程序执行不同的任务，并按顺序运行。从高层次来看，编译有三个不同的阶段——*编译*、*代码生成*和*链接*——由三个不同的组件来处理。

第一个阶段由大多数人认为是“编译器”的程序执行；它处理类型检查、借用检查、单态化以及我们与某个编程语言相关联的其他特性。这个阶段不会生成机器代码，而是生成一个使用大量注解的抽象机器操作的低级表示。然后，这个低级表示被传递给代码生成工具，后者会生成可以在特定 CPU 上运行的机器代码。

这两个操作，合起来并不需要在一次大规模的过程中对整个代码库进行处理。相反，代码库可以被切分成更小的块，然后并行地进行编译。例如，Rust 通常会独立且并行地编译不同的 crate，只要它们之间没有依赖关系。它还可以分别调用代码生成工具来并行处理独立的 crate。Rust 甚至经常可以分别编译同一个 crate 的多个小片段！

一旦应用程序的每个部分的机器码都生成完毕，这些部分就可以被组合在一起。这是在链接阶段完成的，毫不奇怪的是，由链接器来进行。链接器的主要工作是将代码生成过程中产生的所有二进制文件（称为*目标文件*）拼接成一个单一的文件，然后将每个符号的引用替换为该符号的最终内存地址。这就是你如何在一个 crate 中定义一个函数并在另一个 crate 中调用它，同时仍然能分别编译这两个 crate 的原因。

链接器使 FFI 工作。它不关心每个输入目标文件是如何构建的；它只是忠实地将所有目标文件链接在一起，然后解析任何共享的符号。一个目标文件可能最初是 Rust 代码，一个可能是 C 代码，还有一个可能是从互联网下载的二进制文件；只要它们使用相同的符号名称，链接器就会确保生成的机器代码为任何共享符号使用正确的交叉引用地址。

符号可以通过*静态*或*动态*方式进行链接。静态链接是最简单的，因为每个对符号的引用仅仅被该符号定义的地址所替代。另一方面，动态链接将每个对符号的引用与生成的代码绑定，该代码会在程序*运行时*尝试找到符号的定义。稍后我们将进一步讨论这些链接模式。Rust 通常默认对 Rust 代码使用静态链接，对 FFI 使用动态链接。

#### 使用 extern

`extern` 关键字是让我们声明一个符号存在于外部接口中的机制。具体来说，它声明了一个在其他地方定义的符号的存在。在清单 11-1 中，我们在 Rust 中定义了一个静态变量 `RS_DEBUG`，并通过 FFI 使其对其他代码可用。我们还声明了一个静态变量 `FOREIGN_DEBUG`，它的定义未指定，但将在链接时解决。

```
#[no_mangle]
pub static RS_DEBUG: bool = true;

extern {
    static FOREIGN_DEBUG: bool;
}
```

清单 11-1：通过 FFI 暴露一个 Rust 静态变量，并访问在其他地方声明的静态变量

`#[no_mangle]` 属性确保 `RS_DEBUG` 在编译期间保持该名称，而不是让编译器为其分配另一个符号名称，例如，用以区分程序中其他地方的（非 FFI）`RS_DEBUG` 静态变量。由于它是 crate 的公共 API 一部分，因此该变量也声明为 `pub`，尽管对于标记为 `#[no_mangle]` 的项，这个注解并不是严格必要的。请注意，我们没有为 `RS_DEBUG` 使用 `extern`，因为它是在此处定义的。它仍然可以通过其他语言进行链接访问。

包围 `FOREIGN_DEBUG` 静态变量的 `extern` 块表示该声明引用了一个 Rust 会在链接时基于同一符号定义所在位置来学习的地方。由于它在其他地方定义，因此我们没有给它初始化值，只给了一个类型，这个类型应该与定义位置使用的类型匹配。因为 Rust 不知道定义静态变量的代码，所以无法检查你是否为符号声明了正确的类型，因此 `FOREIGN_DEBUG` 只能在 `unsafe` 块内访问。

声明 FFI 函数的过程非常类似。在清单 11-2 中，我们使 `hello_rust` 可供非 Rust 代码访问，并引入外部的 `hello_foreign` 函数。

```
#[no_mangle]
pub extern fn hello_rust(i: i32) { ... }

extern {
    fn hello_foreign(i: i32);
}
```

清单 11-2：通过 FFI 暴露一个 Rust 函数，并访问在其他地方定义的函数

构建模块与 Listing 11-1 中的完全相同，唯一不同的是 Rust 函数使用 `extern fn` 声明，我们将在下一节中探讨这一点。

如果存在多个定义的外部符号，例如 `FOREIGN_DEBUG` 或 `hello_foreign`，你可以使用 `#[link]` 属性明确指定符号应该链接到哪个库。如果不指定，链接器会报错，表示它找到了该符号的多个定义。例如，如果你在 `extern` 块前加上 `#[link(name = "crypto")]`，你是在告诉链接器将任何符号（无论是静态的还是函数）与名为 “crypto” 的库进行链接。你还可以通过在 Rust 代码中为声明加上 `#[link_name = "<actual_symbol_name>"]` 注解来重命名外部静态或函数，这样该项就会链接到你希望的任何名称。类似地，你可以使用 `#[export_name = "<export_symbol_name>"]` 来重命名 Rust 项目的导出名称。

#### 链接类型

`#[link]` 也接受 `kind` 参数，该参数决定块中的项应如何链接。默认情况下，该参数为 `"dylib"`，表示 C 兼容的动态链接。另一个 `kind` 值是 `"static"`，表示块中的项应该在编译时完全链接（即静态链接）。这基本上意味着外部代码直接被嵌入编译器生成的二进制文件中，因此在运行时不需要存在。还有一些其他的 `kind` 值，但它们不太常见，且超出了本书的范围。

静态链接和动态链接之间有几个权衡，但主要考虑因素是安全性、二进制文件大小和分发。首先，动态链接通常更安全，因为它使得独立升级库变得更加容易。动态链接允许部署包含你代码的二进制文件的人在不需要重新编译代码的情况下升级代码所依赖的库。如果比如说 `libcrypto` 得到了安全更新，用户可以在主机上更新加密库并重启二进制文件，更新后的库代码将自动使用。静态编译则是将库的代码直接硬编码到二进制文件中，因此用户必须重新编译你的代码，以适应升级后的库版本。

动态链接通常会产生更小的二进制文件。由于静态编译会将所有链接的代码包含到最终的二进制输出中，并且任何代码所拉入的代码也会被包括进来，从而产生更大的二进制文件。而动态链接则使每个外部项仅包含少量的包装代码，在运行时加载指定的库，并转发访问。

到目前为止，静态链接可能看起来不太吸引人，但它相比动态链接有一个大优势：分发的便捷性。使用动态链接时，任何想要运行包含你代码的二进制文件的人，*还需要*拥有你的代码所依赖的所有库。更重要的是，他们必须确保拥有的每个库的版本与代码期望的版本兼容。像 `glibc` 或 OpenSSL 这样的库在大多数系统上都有，可能不会造成问题，但对于一些较为冷门的库来说就会产生问题。用户需要知道自己应该安装这些库，并且必须去寻找它们才能运行你的代码！而静态链接将库的代码直接嵌入到二进制文件中，因此用户不需要自己安装它。

最终，静态链接和动态链接之间没有一个*正确*的选择。动态链接通常是一个很好的默认选项，但对于特别受限的部署环境，或者非常小型或特殊的库依赖，静态编译可能是更好的选择。使用你的最佳判断！

### 调用约定

符号决定了*某个*函数或变量定义的位置，但这还不足以跨越 FFI 边界进行函数调用。要调用任何语言中的外部函数，编译器还需要知道它的*调用约定*，这决定了用什么样的汇编代码来调用该函数。我们这里不会涉及每个调用约定的实际技术细节，但作为一个概述，调用约定规定了：

+   调用的栈帧是如何设置的

+   参数是如何传递的（是在栈上还是寄存器中，顺序还是反向顺序）

+   函数返回时如何告知它跳回哪里

+   在函数完成后，如何恢复各种 CPU 状态，如寄存器等

Rust 有自己独特的调用约定，这个约定并没有标准化，并且允许编译器随时间变化进行调整。只要所有的函数定义和调用都由同一个 Rust 编译器编译，这样的约定就没有问题，但如果你希望与外部代码互操作，就会出现问题，因为外部代码不知道 Rust 的调用约定。

如果你没有声明任何其他内容，所有的 Rust 函数都会隐式声明为 `extern "Rust"`。单独使用 `extern`，就像在示例 11-2 中那样，是 `extern "C"` 的简写，意思是“使用标准的 C 调用约定”。之所以使用简写，是因为 C 调用约定几乎是所有 FFI 情况下的首选。

Rust 还支持多种调用约定，你可以通过在 `extern` 关键字后添加字符串来指定（在 `fn` 和块上下文中均适用）。例如，`extern "system"` 表示使用操作系统标准库接口的调用约定，在写作时，这个约定在除 Win32 以外的地方与 `"C"` 一致，而 Win32 使用的是 `"stdcall"` 调用约定。一般来说，除非你处理的是特别平台特定或高度优化的外部接口，否则很少需要显式地提供调用约定，因此仅使用 `extern`（即 `extern "C"`）就可以了。

## 跨语言边界的类型

在 FFI 中，类型布局至关重要；如果一种语言以某种方式布局共享数据的内存，而 FFI 边界另一端的语言期望它以不同的方式布局，那么两端将不一致地解释数据。在这一节中，我们将讨论如何在 FFI 中使类型匹配，并在跨语言边界时需要注意的其他类型方面。

### 类型匹配

类型在 FFI 边界上不共享。当你在 Rust 中声明一个类型时，编译后该类型的信息会完全丢失。传递到另一端的只是构成该类型值的位。因此，你需要在边界的两端都声明这些位的类型。当你声明 Rust 版本的类型时，必须首先确保类型内包含的原始类型匹配。例如，如果另一端使用 C，并且 C 类型使用 `int`，那么 Rust 代码最好使用完全相同的 Rust 等价类型：`i32`。为了减少这一过程中的猜测，对于使用类似 C 类型的接口，Rust 标准库通过 `std::os::raw` 模块为你提供正确的 C 类型，其中定义了 `type c_int = i32`，`type c_char = i8/u8`（取决于 `char` 是否有符号），`type c_long = i32/i64`（取决于目标指针宽度）等。

对于更复杂的类型，如向量和字符串，通常需要手动进行映射。例如，由于 C 通常将字符串表示为以 0 字节终止的字节序列，而不是以 UTF-8 编码的字符串并将长度单独存储，因此一般不能通过 FFI 使用 Rust 的字符串类型。相反，假设另一端使用 C 风格的字符串表示，你应该分别使用 `std::ffi::CStr` 和 `std::ffi::CString` 类型来处理借用和拥有的字符串。对于向量，你可能需要使用指向第一个元素的原始指针，然后单独传递长度—`Vec::into_raw_parts` 方法在这方面可能会派上用场。

对于包含其他类型的类型，例如结构体和联合体，你还需要处理布局和对齐。如我们在第二章讨论的那样，Rust 默认情况下以未定义的方式布局类型，因此至少你需要使用`#[repr(C)]`来确保该类型具有确定的布局和对齐方式，这种方式与 FFI 边界上可能使用的（并且希望使用的）方式一致。如果接口还指定了该类型的其他配置，例如手动设置其对齐方式或删除填充，你需要相应地调整你的`#[repr]`。

Rust 枚举有多种可能的 C 样式表示，取决于枚举是否包含数据。考虑一个没有数据的枚举，如下所示：

```
enum Foo { Bar, Baz }
```

使用`#[repr(C)]`，类型`Foo`仅使用一个与 C 编译器为具有相同数量变体的枚举选择的整数大小相同的整数进行编码。第一个变体的值为`0`，第二个变体的值为`1`，以此类推。你也可以手动为每个变体分配值，如列表 11-3 所示。

```
#[repr(C)]
enum Foo {
    Bar = 1,
    Baz = 2,
}
```

列表 11-3：为无数据枚举定义显式变体值

然而，在将 C 中类似枚举的类型映射到 Rust 时，你需要小心，因为只有已定义变体的值对枚举类型的实例是有效的。这通常会导致你遇到问题，特别是与 C 风格的枚举有关，C 风格的枚举往往更像是位集合，变体可以按位或组合成一个值，从而同时封装多个变体。例如，在列表 11-3 中的例子中，取`Bar | Baz`产生的值`3`在 Rust 中对于`Foo`是无效的！如果你需要模拟一个 C API，该 API 使用枚举表示一组可以单独设置和取消设置的位标志，请考虑使用围绕整数类型的新类型包装器，并为每个变体定义关联常量，另外实现各种`Bit*`特征以提高可操作性。或者使用`bitflags` crate。

对于包含数据的枚举，`#[repr(C)]`属性使得枚举被表示为*标记联合体*。也就是说，它在内存中通过一个`#[repr(C)]`结构体表示，结构体有两个字段，第一个字段是判别符，如果没有变体包含字段时，它将按该方式进行编码，第二个字段是每个变体数据结构的联合体。具体示例，请参见列表 11-4 中的枚举及其相关表示。

```
#[repr(C)]
enum Foo {
    Bar(i32),
    Baz { a: bool, b: f64 }
}
// is represented as
#[repr(C)]
enum FooTag { Bar, Baz }
#[repr(C)]
struct FooBar(i32);
#[repr(C)]
struct FooBaz{ a: bool, b: f64 }
#[repr(C)]
union FooData {
  bar: FooBar,
  baz: FooBaz,
}
#[repr(C)]
struct Foo {
    tag: FooTag,
    data: FooData
}
```

列表 11-4：带有`#[repr(C)]`的 Rust 枚举被表示为标记联合体。

### 分配

当你分配内存时，该内存的分配归属于它的分配器，只有该分配器才能释放这块内存。如果你在 Rust 中使用多个分配器，或者如果你在 Rust 中分配内存并使用 FFI 边界另一侧的某个分配器进行内存分配，情况也是如此。你可以自由地跨越边界发送指针并随意访问这块内存，但当再次释放内存时，必须将其返回给相应的分配器。

大多数 FFI 接口将有两种内存分配处理配置：要么是调用者提供数据指针指向内存块，要么是接口暴露专门的释放方法，任何分配的资源在不再需要时应返回到这些方法中。示例 11-5 展示了来自 OpenSSL 库的一些 Rust 声明示例，这些声明使用了实现管理的内存。

```
// One function allocates memory for a new object.
extern fn ECDSA_SIG_new() -> *mut ECDSA_SIG;

// And another accepts a pointer created by new
// and deallocates it when the caller is done with it.
extern fn ECDSA_SIG_free(sig: *mut ECDSA_SIG);
```

示例 11-5：实现管理的内存接口

函数 `ECDSA_SIG_new` 和 `ECDSA_SIG_free` 组成一对，调用者应先调用 `new` 函数，在需要时使用返回的指针（可能会将其传递给其他函数），然后在完成引用的资源后将指针传递给 `free` 函数。可以推测，实现会在 `new` 函数中分配内存，在 `free` 函数中释放内存。如果这些函数在 Rust 中定义，`new` 函数可能会使用 `Box::new`，而 `free` 函数则会调用 `Box::from_raw`，然后通过 `drop` 执行析构。

示例 11-6 展示了调用者管理的内存示例。

```
// An example of caller-managed memory.
// The caller provides a pointer to a chunk of memory,
// which the implementation then uses to instantiate its own types.
// No free function is provided, as that happens in the caller.
extern fn BIO_new_mem_buf(buf: *const c_void, len: c_int) -> *mut BIO
```

示例 11-6：调用者管理的内存接口

在这里，`BIO_new_mem_buf` 函数要求调用者提供后备内存。调用者可以选择在堆上分配内存，或者使用任何其他合适的机制来获取所需的内存，并将其传递给库。之后，责任在于调用者，确保内存在不再被 FFI 实现使用时被释放！

你可以在 FFI API 中使用这些方法中的任意一种，甚至可以根据需要将它们混合使用。一般来说，当可能时，允许调用者传入内存，因为这样可以让调用者在管理内存时拥有更多自由。例如，调用者可能在某个定制的操作系统上使用高度专业化的分配器，可能不希望被迫使用你实现中将使用的标准分配器。如果调用者可以传入内存，它甚至可以避免分配内存，若它能够使用堆栈内存或重用已分配的内存。然而，记住调用者管理的接口通常更复杂，因为调用者必须完成所有工作，计算要分配多少内存，并在调用库之前进行设置。

在某些情况下，调用者甚至无法提前知道需要分配多少内存——例如，如果您的库类型是不可见的（因此调用者无法知晓），或者这些类型会随着时间变化，调用者就无法预测分配的大小。类似地，如果您的代码在运行时需要分配更多内存，比如在动态构建图时，所需的内存量可能会在运行时动态变化。在这种情况下，您将需要使用由实现管理的内存。

当您必须做出权衡时，对于任何*较大*或*频繁*的内存分配，最好使用调用者分配内存。在这些情况下，调用者可能最关心的是控制内存分配。对于其他情况，您的代码分配内存并为每个相关类型暴露析构函数可能是可以接受的。

### 回调

您可以跨 FFI 边界传递函数指针，并通过这些指针调用引用的函数，只要函数指针的类型具有与函数调用约定匹配的 `extern` 注解。也就是说，您可以在 Rust 中定义一个 `extern "C" fn(c_int) -> c_int` 函数，然后将该函数的引用传递给 C 代码，作为回调，C 代码最终会调用它。

使用回调时要小心恐慌，因为如果恐慌在函数结束后发生，而该函数的类型不是 `extern "Rust"`，则行为是未定义的。目前，Rust 编译器在检测到这种恐慌时会自动中止，但这可能不是您希望的行为。相反，您可能希望使用 `std::panic::catch_unwind` 来检测任何标记为 `extern` 的函数中的恐慌，然后将恐慌转化为一个 FFI 兼容的错误。

### 安全性

当您编写 Rust FFI 绑定时，大部分实际与 FFI 接口交互的代码都会是 `unsafe` 的，主要涉及原始指针。然而，您的目标应该是在 FFI 上方最终呈现一个*安全*的 Rust 接口。这样做主要是通过仔细阅读您正在包装的 `unsafe` 接口的不变量，并确保通过 Rust 类型系统在安全接口中保持这些不变量。安全封装外部接口的三个最重要元素是准确捕获 `&` 与 `&mut`，适当地实现 `Send` 和 `Sync`，并确保指针不会被意外混淆。接下来，我将详细介绍如何执行这些操作。

#### 引用和生命周期

如果外部代码有可能修改给定指针指向的数据，确保安全的 Rust 接口通过 `&mut` 拥有对相关数据的独占引用。否则，您的安全封装的用户可能会意外读取正在被外部代码同时修改的内存，后果不堪设想！

你还需要充分利用 Rust 生命周期来确保所有指针的生命周期与 FFI 要求的一致。例如，假设有一个外部接口，允许你创建一个`Context`，然后从该`Context`创建一个`Device`，并要求`Context`在`Device`的生命周期内保持有效。在这种情况下，任何安全的接口包装器都应该通过使`Device`持有与`Context`的借用相关联的生命周期来在类型系统中强制执行这一要求。

#### Send 和 Sync

除非外部库明确记录其类型是线程安全的，否则不要为外部库中的类型实现`Send`和`Sync`！确保安全 Rust 代码*不能*违反外部代码的不变式，从而触发未定义行为，这是安全 Rust 包装器的职责。

有时，你甚至可能想要引入虚拟类型来强制执行外部不变式。例如，假设你有一个事件循环库，其接口如示例 11-7 所示。

```
extern fn start_main_loop();
extern fn next_event() -> *mut Event;
```

示例 11-7：一个期望单线程使用的库

假设外部库的文档说明`next_event`只能由调用`start_main_loop`的同一线程调用。然而，在这里我们没有可以避免为其实现`Send`的类型！相反，我们可以借鉴第三章的思路，介绍额外的标记状态来强制执行不变式，如示例 11-8 所示。

```
pub struct EventLoop(std::marker::PhantomData<*const ()>);
pub fn start() -> EventLoop {
    unsafe { ffi::start_main_loop() };
    EventLoop(std::marker::PhantomData)
}
impl EventLoop {
    pub fn next_event(&self) -> Option<Event> {
        let e = unsafe { ffi::next_event() };
        // ...
    }
}
```

示例 11-8：通过引入辅助类型来强制执行 FFI 不变式

空类型`EventLoop`实际上并未与底层外部接口的任何内容连接，而是强制要求在调用`start_main_loop`后，并且仅在同一线程上调用`next_event`。你通过使`EventLoop`既不是`Send`也不是`Sync`来强制执行“同一线程”这一部分，通过让它持有一个虚拟的原始指针（该指针本身既不是`Send`也不是`Sync`）。

````Using `PhantomData<*const ()>` to “undo” the `Send` and `Sync` auto-traits as we do here is a bit ugly and indirect. Rust does have an unstable compiler feature that enables negative trait implementations like `impl !Send for EventLoop {}`, but it’s surprisingly difficult to get its implementation right, and it likely won’t stabilize for some time.    You may have noticed that nothing prevents the caller from invoking `start_main_loop` multiple times, either from the same thread or from another thread. How you’d handle that would depend on the semantics of the library in question, so I’ll leave it to you as an exercise.    #### Pointer Confusion    In many FFI APIs, you don’t necessarily want the caller to know the internal representation for each and every chunk of memory you give it pointers to. The type might have internal state that the caller shouldn’t fiddle with, or the state might be difficult to express in a cross-language-compatible way. For these kinds of situations, C-style APIs usually expose *void pointers*, written out as the C type `void*`, which is equivalent to `*mut std::ffi::c_void` in Rust. A type-erased pointer like this is, effectively, *just* a pointer, and does not convey anything about the thing it points to. For that reason, these kinds of pointers are often referred to as *opaque*.    Opaque pointers effectively serve the role of visibility modifiers for types across FFI boundaries—since the method signature does not say what’s being pointed to, the caller has no option but to pass around the pointer as is and use any available FFI methods to provide visibility into the referenced data. Unfortunately, since one `*mut c_void` is indistinguishable from another, there’s nothing stopping a user from taking an opaque pointer as is returned from one FFI method and supplying it to a method that expects a pointer to a *different* opaque type.    We can do better than this in Rust. To mitigate this kind of pointer type confusion, we can avoid using `*mut c_void` directly for opaque pointers in FFI, even if the actual interface calls for a `void*`, and instead construct different empty types for each distinct opaque type. For example, in Listing 11-9 I use two distinct opaque pointer types that cannot be confused.    ``` #[non_exhaustive] #[repr(transparent)] pub struct Foo(c_void); #[non_exhaustive] #[repr(transparent)] pub struct Bar(c_void); extern {     pub fn foo() -> *mut Foo;     pub fn take_foo(arg: *mut Foo);     pub fn take_bar(arg: *mut Bar); } ```    Listing 11-9: Opaque pointer types that cannot be confused    Since `Foo` and `Bar` are both zero-sized types, they can be used in place of `()` in the `extern` method signatures. Even better, since they are now distinct types, Rust won’t let you use one where the other is required, so it’s now impossible to call `take_bar` with a pointer you got back from `foo`. Adding the `#[non_exhaustive]` annotation ensures that the `Foo` and `Bar` types cannot be constructed outside of this crate.    ## bindgen and Build Scripts    Mapping out the Rust types and `extern`s for a larger external library can be quite a chore. Big libraries tend to have a large enough number of type and method signatures to match up that writing out all the Rust equivalents is time-consuming. They also have enough corner cases and C oddities that some patterns are bound to require more careful thought to translate.    Luckily, the Rust community has developed a tool called `bindgen` that significantly simplifies this process as long as you have C header files available for the library you want to interface with. `bindgen` essentially encodes all the rules and best practices we’ve discussed in this chapter, plus a number of others, and wraps them up in a configurable code generator that takes in C header files and spits out appropriate Rust equivalents.    `bindgen` provides a stand-alone binary that generates the Rust code for C headers once, which is convenient when you want to check in the bindings. This process allows you to hand-tune the generated bindings, should that be necessary. If, on the other hand, you want to generate the bindings automatically on every build and just include the C header files in your source code, `bindgen` also ships as a library that you can invoke in a custom *build script* for your package.    You declare a build script by adding `build = "``<some-file.rs>``"` to the `[package]` section of your *Cargo.toml*. This tells Cargo that, before compiling your crate, it should compile *<some-file.rs>* as a stand-alone Rust program and run it; only then should it compile the source code of your crate. The build script also gets its own dependencies, which you declare in the `[build-dependencies]` section of your *Cargo.toml*.    Build scripts come in very handy with FFI—they can compile a bundled C library from source, dynamically discover and declare additional build flags to be passed to the compiler, declare additional files that Cargo should check for changes for the purposes of recompilation, and, you guessed it, generate additional source files on the fly!    Though build scripts are very versatile, beware of making them too aware of the environment they run in. While you can use a build script to detect if the Rust compiler version is a prime or if it’s going to rain in Istanbul tomorrow, making your compilation dependent on such conditions may make builds fail unexpectedly for other developers, which leads to a poor development experience.    The build script can write files to a special directory supplied through the `OUT_DIR` environment variable. The same directory and environment variable are also accessible in the Rust source code at compile time so that it can pick up files generated by the build script. To generate and use Rust types from a C header, you first have your build script use the library version of `bindgen` to read in a *.h* file and turn it into a file called, say, *bindings.rs* inside `OUT_DIR`. You then add the following line to any Rust file in your crate to include *bindings.rs* at compilation time:    ``` include!(concat!(env!("OUT_DIR"), "/bindings.rs")); ```    Since the code in *bindings.rs* is autogenerated, it’s generally best practice to place the bindings in their own crate and give the crate the same name as the library the bindings are for, with the suffix `-sys` (for example, `openssl-sys`). If you don’t follow this practice, releasing new versions of your library will be much more painful, as it is illegal for two crates that link against the same external library through the `links` key in *Cargo.toml* to coexist in a given build. You would essentially have to upgrade the entire ecosystem to the new major version of your library all at once. Separating just the bindings into their own crate allows you to issue new major versions of the wrapper crate that can be adopted incrementally. The separation also allows you to cut a breaking release of the crate with those bindings if the Rust bindings change—say, if the header files themselves are upgraded or a `bindgen` upgrade causes the generated Rust code to change slightly—without *also* having to cut a breaking release of the crate that safely wraps the FFI bindings.    If your crate instead produces a library file that you intend others to use through FFI, you should also publish a C header file for its interface to make it easier to generate native bindings to your library from other languages. However, that C header file then needs to be kept up to date as your crate changes, which can become cumbersome as your library grows in size. Fortunately, the Rust community has also developed a tool to automate this task: `cbindgen`. Like `bindgen`, `cbindgen` is a build tool, and it also comes as both a binary and a library for use in build scripts. Instead of taking in a C header file and producing Rust, it takes Rust in and produces a C header file. Since the C header file represents the main computer-readable description of your crate’s FFI, I recommend manually looking it over to make sure the autogenerated C code isn’t too unwieldy, though in general `cbindgen` tends to produce fairly reasonable code. If it doesn’t, file a bug!    ## Summary    In this chapter, we’ve covered how to use the `extern` keyword to call out of Rust into external code, as well as how to use it to make Rust code accessible to external code. We’ve also discussed how to align Rust types with types on the other side of the FFI boundary, and some of the common pitfalls in trying to get code written in two different languages to mesh well. Finally, we talked about the `bindgen` and `cbindgen` tools, which make the experience of keeping FFI bindings up to date much more pleasant. In the next chapter, we’ll look at how to use Rust in more restricted environments, like embedded devices, where the standard library may not be available and where even a simple operation like allocating memory may not be possible.````
