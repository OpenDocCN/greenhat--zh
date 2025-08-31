## 附录：C

二进制分析工具列表

在第六章中，我使用 IDA Pro 进行了递归反汇编示例，使用 `objdump` 进行了线性反汇编，但你可能更喜欢其他工具。附录列出了你可能觉得有用的流行反汇编器和二进制分析工具，包括用于逆向工程的交互式反汇编器以及能够执行跟踪的反汇编 API 和调试器。

### C.1 反汇编器

**IDA Pro**（Windows，Linux，macOS；*[www.hex-rays.com](http://www.hex-rays.com)*）

这是业界标准的递归反汇编器。它是交互式的，包含 Python 和 IDC 脚本 API 以及反编译器。它是最好的反汇编器之一，但也是最昂贵的（最基础版售价 $700）。一个旧版本（v7）是免费的，但仅支持 x86-64 且不包括反编译器。

**Hopper**（Linux，macOS；*[www.hopperapp.com](http://www.hopperapp.com)*）

这是一个比 IDA Pro 更简单且便宜的替代工具。它共享了许多 IDA 的功能，包括 Python 脚本和反编译，尽管这些功能的开发不如 IDA 完善。

**ODA**（任何平台；*[onlinedisassembler.com](http://onlinedisassembler.com)*）

在线反汇编器是一个免费的、轻量级的在线递归反汇编工具，非常适合快速实验。你可以上传二进制文件或在控制台输入字节。

**Binary Ninja**（Windows，Linux，macOS；*binary.ninja*）

一款有前途的新兴工具，Binary Ninja 提供了一个交互式递归反汇编器，支持多种架构，并且为 C、C++ 和 Python 提供了广泛的脚本支持。反编译功能是计划中的特性。Binary Ninja 不是免费的，但个人版对于一个功能完备的逆向平台来说相对便宜，售价为 $149。也有一个有限的演示版可用。

**Relyze**（Windows；*[www.relyze.com](http://www.relyze.com)*）

Relyze 是一个交互式递归反汇编器，提供二进制差异功能和 Ruby 脚本支持。它是商业软件，但比 IDA Pro 便宜。

**Medusa**（Windows，Linux；*[github.com/wisk/medusa/](http://github.com/wisk/medusa/)*）

Medusa 是一个交互式、多架构、递归反汇编器，具有 Python 脚本功能。与大多数同类反汇编器不同，它是完全免费的开源工具。

**radare**（Windows，Linux，macOS；*[www.radare.org](http://www.radare.org)*）

这是一个极其灵活的以命令行为导向的逆向工程框架。与其他反汇编器的不同之处在于，它被设计为一组工具，而不是一个统一的界面。通过命令行任意组合这些工具使得 radare 非常灵活。它提供了线性和递归反汇编模式，可以交互使用，也可以完全脚本化。它的主要应用领域是逆向工程、取证和黑客攻击。这个工具集是免费且开源的。

**objdump**（Linux，macOS；*[www.gnu.org/software/binutils/](http://www.gnu.org/software/binutils/)*）

这是本书中使用的著名线性反汇编器。它是免费的并且开源。GNU 版本是 GNU binutils 的一部分，已为所有 Linux 发行版预打包。它也可以在 macOS 上使用（如果安装了 Cygwin^(1))。

### C.2 调试器

**gdb**（Linux；*[www.gnu.org/software/gdb/](http://www.gnu.org/software/gdb/)*）

GNU 调试器是 Linux 系统上的标准调试器，主要用于交互式调试，也支持远程调试。虽然您也可以使用`gdb`进行执行追踪，第九章显示其他工具，如 Pin，更适合自动化地执行这项任务。

**OllyDbg**（Windows；*[www.ollydbg.de](http://www.ollydbg.de)*）

这是一个功能强大的 Windows 调试器，具有内置的执行追踪功能和用于解包混淆二进制文件的高级功能。它是免费的，但不是开源的。虽然没有直接的脚本功能，但有用于开发插件的接口。

**windbg**（Windows；*[`docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugger-download-tools`](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugger-download-tools)*) 这是微软发布的 Windows 调试器，能够调试用户模式和内核模式代码，并分析崩溃转储。

**Bochs**（Windows，Linux，macOS；*[`bochs.sourceforge.net`](http://bochs.sourceforge.net)*）

这是一个便携式 PC 模拟器，支持大多数平台，您还可以用它调试模拟的代码。Bochs 是开源的，按照 GNU LGPL 协议发布。

### C.3 反汇编框架

**Capstone**（Windows，Linux，macOS；*[www.capstone-engine.org](http://www.capstone-engine.org)*）

Capstone 不是一个独立的反汇编器，而是一个免费的开源反汇编引擎，您可以用它来构建自己的反汇编工具。它提供了一个轻量级的多架构 API，并且支持 C/C++、Python、Ruby、Lua 等多种语言的绑定。该 API 允许对反汇编指令的属性进行详细检查，这对于构建自定义工具非常有用。第八章完全讲解了如何使用 Capstone 构建自定义反汇编工具。

**distorm3**（Windows，Linux，macOS；*[github.com/gdabah/distorm/](http://github.com/gdabah/distorm/)*）

这是一个开源的 x86 代码反汇编 API，旨在快速反汇编。它提供了多个语言的绑定，包括 C、Ruby 和 Python。

**udis86**（Linux，macOS；*[github.com/vmt/udis86/](http://github.com/vmt/udis86/)*）

这是一个简单、干净、极简的开源反汇编库，专为 x86 代码设计，您可以使用它在 C 语言中构建自己的反汇编工具。

### C.4 二进制分析框架

**angr**（Windows，Linux，macOS；*angr.io*）

Angr 是一个面向 Python 的逆向工程平台，作为构建自己二进制分析工具的 API。它提供了许多高级功能，包括反向切片和符号执行（在第十二章中讨论）。它主要是一个研究平台，但在积极开发中，并且文档相当完善（且不断改进）。Angr 是免费的且开源的。

**Pin**（Windows、Linux、macOS；* [www.intel.com/software/pintool/](http://www.intel.com/software/pintool/) *）

Pin 是一个动态二进制插桩引擎，允许你构建自己的工具，在运行时添加或修改二进制文件的行为。（有关动态二进制插桩的更多内容，请参见第九章）。Pin 是免费的，但不是开源的。它由英特尔开发，仅支持英特尔 CPU 架构，包括 x86。

**Dyninst**（Windows、Linux；* [www.dyninst.org](http://www.dyninst.org) *）

像 Pin 一样，Dyninst 也是一个动态二进制插桩 API，尽管你也可以用它进行反汇编。Dyninst 是免费的且开源的，更多偏向于研究用途，而不是像 Pin 那样侧重于工具开发。

**Unicorn**（Windows、Linux、macOS；* [www.unicorn-engine.org](http://www.unicorn-engine.org) *）

Unicorn 是一个轻量级的 CPU 模拟器，支持多种平台和架构，包括 ARM、MIPS 和 x86。由 Capstone 作者维护，Unicorn 支持多种语言绑定，包括 C 和 Python。Unicorn 不是一个反汇编器，而是一个用于构建基于仿真的分析工具的框架。

**libdft**（Linux；* [www.cs.columbia.edu/~vpk/research/libdft/](http://www.cs.columbia.edu/~vpk/research/libdft/) *）

这是一个免费的、开源的动态污点分析库，用于第十一章中的所有污点分析示例。`libdft` 设计上追求快速和易用，提供了两种变体，支持字节粒度的影像内存，并提供一种或八种污点颜色。

**Triton**（Windows、Linux、macOS；* [triton.quarkslab.com](http://triton.quarkslab.com) *）

Triton 是一个动态二进制分析框架，支持符号执行和污点分析等功能。你可以在第十三章中看到它的符号执行能力。Triton 是免费且开源的。
