# 附录 E. 安装 GIMP

在计算机上安装 GIMP 不再困难。因此，本附录将相对简短，如果你遇到问题，可以在附录 C 中提到的网站找到帮助。官方的 GIMP 网站（* [`www.gimp.org/`](http://www.gimp.org/) *）是获取 GIMP 及其组件下载和安装说明的最佳途径。

尽管 GPL 许可证并不禁止销售 GIMP 发行版，但购买 GIMP 的商业发行版是没有意义的，甚至可能有害：除了某些精美的包装外，它没有任何附加价值，而且有些商业发行版已知包含病毒。请勿从非官方 GIMP 网站或非官方链接下载 GIMP。

# E.1 GNU/Linux 和 Unix

GIMP 最初是为 GNU/Linux 设计的，且大多数开发者在这个操作系统上工作。因此，在该系统上安装 GIMP 尤其容易。此外，主要的 GNU/Linux 发行版提供自动化工具来安装 GIMP（如果它还没有默认安装的话）。当然，如果你的发行版几个月前安装，并且你没有定期更新，那么很可能你系统上的 GIMP 版本不是最新的（即 2.8 版本），因此你需要更新软件。在大多数情况下，简单地更新你的安装就足够了。

## Debian

对于 Debian，使用简单的命令

```
apt-get install gimp
```

以`root`用户身份运行安装 GIMP 及其所有依赖项。但安装一些额外的软件包是有帮助的。以下是一些主要的软件包：

+   `gimp-gap`是 GIMP 动画包，详见第十八章。

+   `gimp-plugin-registry`包含许多有用的插件和脚本，详见第二十一章。

+   `gimp-gmic`是 G'MIC 插件集，详见第二十一章。

+   `gnome-xcf-thumbnailer`允许 GNOME 桌面环境显示 XCF 文件的缩略图。

+   `gtkam-gimp`将 GIMP 与`Gtkam`连接，以便访问数码相机中的照片。

+   `gimp-gutenprint`链接到`Gutenprint`，提供一个强大且灵活的接口，用于许多打印机。

+   `gimp-ufraw`将 GIMP 与`Ufraw`连接，以处理大多数数码相机拍摄的 RAW 照片。

+   `gimp-data-extras`提供额外的画笔、调色板和渐变集。

使用`synaptic`图形工具安装这些软件包非常简单。

## Ubuntu

由于 Ubuntu 是 Debian 的衍生版，因此安装说明是相同的。唯一的真正区别是，默认情况下，Ubuntu 没有`root`用户，安装命令是

```
sudo apt-get install gimp
```

从 Ubuntu 10.04 版本开始，GIMP 不再是安装光盘的一部分，因此你需要单独安装它。

## Mint

Mint 是一个相对较新的 GNU/Linux 发行版，提供两个版本。Linux Mint 12 基于 Ubuntu，Linux Mint Debian 基于 Debian。因此，安装 GIMP 在 Mint 上的步骤与这两个发行版的步骤相同。

## Fedora

在 Fedora 中，概念类似，但命令不同。运行

```
yum install gimp
```

以 `root` 用户身份运行来安装 GIMP。Fedora 包数据库还包含以下包：

+   `gimp-data-extras` 在 Debian 中起到相同的作用。

+   `gimpfx-foundry` 提供了一组额外的插件。

+   `gimp-help` 让你可以在一个独立的包中访问 GIMP 帮助。

其他插件，包括 GAP，必须通过 GIMP 插件注册表手动安装。

## OpenSUSE

在 OpenSUSE 中，运行

```
zypper install gimp
```

以 `root` 用户身份运行（如果你正在使用 OpenSUSE 的最新版本）。你必须手动安装所有附加数据或插件。

## Mandriva

在 Mandriva 中，运行

```
urpmi gimp
```

以 `root` 用户身份运行。附加的包包括以下内容：

+   `gimp-data-extras` 在 Debian 中起到相同的作用。

+   `gimp2-gap` 是 GAP 插件集。

+   `gimpfx-foundry` 提供了一组额外的插件。

+   `gimp-help` 让你可以访问 GIMP 帮助的独立包，取决于语言。

## 其他类 Unix 操作系统

对于其他类 Unix 操作系统，如 BSD 版本或一些不太知名的 GNU/Linux 发行版，已经编译的包不可用，唯一的解决方案是从源代码编译 GIMP 和所有需要的库。这个过程可能会很痛苦，尽管所有这些软件都是免费的，源代码也可以自由获取。请访问 *[`www.gimp.org/downloads/`](http://www.gimp.org/downloads/)* 并按照说明操作。选择一个靠近你所在位置的镜像站点：这样可以避免主网站的过载，并提高下载速度。

# E.2 Windows

尽管大多数 GIMP 开发者都是 GNU/Linux 的用户，但他们知道外部世界仍然由各种版本的 Windows 操作系统主导。如在 *[`www.gimp.org/windows/`](http://www.gimp.org/windows/)* 所述，Jernej Simončič 为 Windows 提供了一个 GIMP 安装程序，并为每个新版本进行更新。你可以从 *[`gimp-win.sourceforge.net/stable.html`](http://gimp-win.sourceforge.net/stable.html)* 网站下载该安装包并立即安装。它包含了你所需的一切，并可以顺利安装在 Windows XP SP2 及更高版本上。安装过程类似于大多数应用程序，此处不再详细说明。安装完成后，从 GIMP 插件注册表下载并安装任何附加插件 (*[`registry.gimp.org/`](http://registry.gimp.org/)*).

但是，请注意，在线帮助是一个单独的包，提供多种语言版本。我们还建议，如果你已经安装了旧版本的 GIMP，最好先卸载它，再尝试安装 GIMP 2.8。

如果正确执行这些简单的操作，你应该不会遇到问题。当然，Windows 桌面环境与各种可用的 GNU/Linux 环境不同，通常在使用 GIMP 时不太方便。在 Windows 中，你无法享受多个工作区的优势，这意味着所有窗口都显示在同一个屏幕上，处理多个应用程序，甚至是一个应用程序的多个窗口，会显得有些麻烦。

Windows 用户可能更喜欢单窗口模式的 GIMP 界面，如 第九章 所描述。这并不意味着多窗口界面不可用，但 Windows 用户需要改变一些习惯来使用它。此外，只有在能够改变窗口管理器的行为，使得鼠标指针一接触窗口，窗口就变为活动窗口时，这种界面才有用。每次从工具箱切换到图像窗口时都需要点击两次（一次点击激活窗口，第二次点击触发预期的操作）可能会非常烦人。

不要尝试在 Windows XP SP2 之前的版本上使用 GIMP。而且不要忘了，向你的电脑中安装 GNU/Linux 发行版（与 Windows 并存）非常简单，并且可以在两个操作系统之间共享数据。更多细节请参见 *[`en.wikipedia.org/wiki/Multiboot`](http://en.wikipedia.org/wiki/Multiboot)*。

# E.3 Mac OS X

使用 OS X 的 GIMP 开发者数量仍然少于使用 Windows 的数量。但网站 *ftp://ftp.gimp.org/pub/gimp/v2.8/osx/gimp-2.8.2-dmg-2.dmg* 和 *[`gimp.lisanet.de/`](http://gimp.lisanet.de/)* 为 OS X 用户提供了与 Windows 一样便捷的 GIMP 安装工具。现在不再需要先安装 X11 环境。不要在较旧版本的 Mac OS 上安装 GIMP。

与 Windows 一样，GIMP 帮助文档是一个单独的软件包，但仅提供英文、德文、西班牙文和法文版本。

由于 Mac 界面的问题，其他界面问题最近都已经解决，GIMP 在 Mac 上的表现现在与其他操作系统完全相同。甚至菜单栏也采用了 GIMP 风格，而不是 Mac 风格，因此屏幕顶部的栏位将不会非常有用。Mac 用户可能会像 Windows 用户一样，更喜欢单窗口模式界面。
