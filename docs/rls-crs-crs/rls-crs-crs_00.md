# 前言

Ruby on Rails 框架强调开发者的生产力，使得曾经需要数月才能完成的网站，如今可以在几周甚至几天内实现！感谢 Ruby 编程语言以及诸如*约定优于配置*和*避免重复自己*等原则，Rails 开发者可以花更少的时间来配置应用程序，更多的时间用于编写代码。

Ruby on Rails 还是一个*全栈*网页框架，这意味着它处理从访问数据库中的数据到在浏览器中渲染网页的所有内容。作为一个全栈框架，Rails 由看似无尽的不同组件组成，例如 Active Record、资产管道、CoffeeScript、Sass、jQuery、turbolinks 以及各种测试框架。

本书旨在简化内容，准确解释你需要了解的所有知识，帮助你开发自己的 Ruby on Rails 应用程序。在你掌握 Rails 基础知识后，我将根据需要介绍和解释框架的新组件。

到最后，你将学会如何从零开始构建自己的 Rails 应用程序。你将添加测试来确保功能按预期工作，保护你的应用程序和用户免受安全漏洞的威胁，优化应用程序的性能，并最终将应用程序部署到自己的服务器上。

# 本书适用人群

我假设你在开始本书之前已经有一定的网页开发经验。你应该熟悉 HTML 和 CSS。你应该知道什么是 `H1` 元素，以及如何将图像和链接添加到网页中。了解面向对象编程的一些知识是有帮助的，但不是必需的。

你将使用计算机的终端（或命令提示符）输入命令，但你无需有太多终端命令的经验也能跟随示例进行操作。除了终端，你还需要一个文本编辑器来编写 Ruby 代码。许多 Rails 开发者使用复古编辑器，如 Vim 或 Emacs。

如果你还没有自己偏好的文本编辑器，我推荐 Sublime Text。你可以在 *[`www.sublimetext.com/`](http://www.sublimetext.com/)* 上找到 Sublime Text 的免费试用版。免费试用版没有到期限制，但偶尔会提示你购买许可证。

# 概述

本书分为两部分。第一部分介绍 Ruby 语言和 Ruby on Rails 框架的基础知识。第二部分则介绍 Ruby 和 Ruby on Rails 中的高级主题。每章末尾都有练习题，答案会在本书末尾提供。

**第一章**介绍了 Ruby 的基础知识，包括数据类型、控制流、方法和类。

**第二章**介绍了 Ruby on Rails 的基础知识。内容包括 Rails 原则、Rails 应用程序使用的目录结构和常见的 Rails 命令。在本章结束时，你将创建你的第一个 Rails 应用程序！

**第三章**、**第四章** 和 **第五章** 讲解了 Rails 使用的模型-视图-控制器架构的三个部分。

**第六章** 讲解了如何创建 Git 仓库来存储你的应用程序，并使用 Heroku 将应用程序部署到网络上。

一旦你掌握了 Ruby 和 Ruby on Rails 的基础知识，你就可以进入更高级的话题。

**第七章** 讲解了 Ruby 模块、Ruby 对象模型，甚至还有一些元编程内容。

**第八章** 介绍了更高级的 Active Record 关联。在本章结束时，你还将构建一个新应用程序的数据模型。

**第九章** 讲解了你新应用程序使用的认证系统。该系统允许用户注册账户、登录应用程序并登出。

**第十章** 讲解了使用 Ruby 随附的 MiniTest 框架对应用程序的各个部分进行自动化测试。本章还讨论了测试驱动开发。

**第十一章** 介绍了常见的 web 应用程序安全漏洞，并解释了如何确保你的应用程序是安全的。

**第十二章** 介绍了 Rails 应用程序的性能优化。内容包括 Rails 内置的优化功能、SQL 查询优化和缓存。

**第十三章** 介绍了几种追踪 bug 的方法。学习如何添加应用程序生成的日志文件，并如何使用交互式调试器来解决真正棘手的 bug。

**第十四章** 解释了如何使用 GitHub API，并讲解了为你的应用程序创建自己的 API 的过程。

最后，**第十五章** 解释了如何在 Amazon 云上设置自己的服务器，并使用 Capistrano 部署你的应用程序。

# 安装

要跟随本书中的示例并完成练习，你需要 Ruby 编程语言、Ruby on Rails 框架、Git 版本控制系统以及 Heroku Toolbelt。

Ruby 语言官网提供了安装说明，链接在 *[`www.ruby-lang.org/en/installation/`](https://www.ruby-lang.org/en/installation/)*。Rails 被分发为一组 Ruby gems，你可以通过单个命令下载和安装，具体取决于你的操作系统。（Ruby on Rails 官网也提供了安装说明，链接在 *[`rubyonrails.org/download/`](http://rubyonrails.org/download/)*。）你可以在 *[`git-scm.com/downloads/`](http://git-scm.com/downloads/)* 下载 Git。

安装完 Ruby、Rails 和 Git 后，安装最新版本的 Heroku Toolbelt，用于将你的应用程序部署到 Heroku。下载 Heroku Toolbelt 安装程序，链接在* [`toolbelt.heroku.com/`](https://toolbelt.heroku.com/)*，然后按照那里提供的指示完成安装。

## Ruby、Rails 和 Git

以下部分包含了在 Mac OS X、Linux 和 Windows 上安装 Ruby、Rails 和 Git 的详细说明。如果你使用的是 Mac OS X 或 Linux，还可以参考多个 Ruby 版本，这是一种替代的安装 Ruby 的方式。Windows 上有一个名为 pik 的工具可以管理多个 Ruby 版本，但自 2012 年以来该工具未再更新，因此我在此不作介绍。

### Mac OS X

通过`ruby --version`检查你当前的 Ruby 版本。如果你使用的是 Mac OS X Mavericks，你应该已经安装了 Ruby 2.0.0 版本。否则，你需要安装一个更新的版本。

即使你已经有了 Ruby 2.0.0，我仍然推荐在 Mac OS X 上使用 Homebrew 包管理器。Homebrew 是一种在 Mac OS X 上安装和更新常用开发工具的简便方式。有关下载和安装 Homebrew 的说明可以在* [`brew.sh/`](http://brew.sh/)*上找到。安装 Homebrew 后，打开终端并输入命令**`brew install ruby`**来安装最新版本的 Ruby。

接下来，使用命令**`gem install rails`**安装 Ruby on Rails。然后再次使用 Homebrew 安装 Git，输入命令**`brew install git`**。

### Linux

Linux 的安装说明根据你使用的 Linux 发行版有所不同。首先，检查你的包管理器，它可能已经有最新版本的 Ruby。如果有，只需像安装其他软件包一样安装它。

如果没有，你需要从源代码安装 Ruby。从* [`www.ruby-lang.org/en/downloads/`](https://www.ruby-lang.org/en/downloads/)*下载当前稳定版本。解压文件后，在终端中输入以下命令：

```
$ **./configure**
$ **make**
$ **sudo make install**
```

安装完成后，通过输入命令**`sudo gem install rails`**来安装 Ruby on Rails。

每个 Linux 发行版都包含 Git。如果你的系统中没有安装 Git，可以使用包管理器安装它。

### Windows

你将使用 RubyInstaller 来安装 Ruby。下载 RubyInstaller 和相应的开发工具包，链接在* [`rubyinstaller.org/downloads/`](http://rubyinstaller.org/downloads/)*。

首先，点击 RubyInstaller 下载页面上的最新 Ruby 版本进行下载；在写这篇文章时，是 *2.1.5*。然后向下滚动到“Development Kit”部分，点击你所选 Ruby 版本下的链接下载开发工具包。以 Ruby 2.1 为例，你需要选择 *DevKit-mingw64-32-4.7.2-20130224-1151-sfx.exe*。如果你使用的是 64 位版本的 Windows，则需要下载 64 位版本的安装程序和相应的 64 位开发工具包，当前版本是 *DevKit-mingw64-64-4.7.2-20130224-1151-sfx.exe*。

下载完成后，双击 RubyInstaller 文件，然后按照屏幕上的提示完成 Ruby 的安装。确保勾选 *Add Ruby executables to your PATH* 选项。完成后，双击 *DevKit* 文件并输入路径 *C:\DevKit* 来解压文件。现在打开命令提示符并输入以下命令以安装开发工具包：

```
$ cd C:\DevKit
$ ruby dk.rb init
$ ruby dk.rb install
```

有些用户在尝试安装 gems 时会遇到 SSL 错误。更新到最新版本的 `gem` 命令可以修复这些错误。输入以下命令来更新你的 `gem` 版本：

```
$ gem update --system –-clear-sources –-source http://rubygems.org
```

安装 Ruby 和开发工具包后，通过输入 **`gem install rails`** 来安装 Rails。这将连接到 RubyGems 服务器，下载并安装构成 Ruby on Rails 框架的各种包。

最后，下载最新版本的 Git 并双击文件完成安装。

### 注意

注意：在本书中，我要求你输入一些命令，如 `bin/rake` 和 `bin/rails`。这些命令在 Windows 上不起作用。Windows 用户请在这些命令前添加 `ruby`。例如，你需要输入 `ruby bin/rake` 和 `ruby bin/rails`。

## 多个 Ruby 版本

存在一些第三方工具，可以方便地在单台计算机上安装和管理多个版本的 Ruby。如果你维护多个不同的应用程序，或者想在不同版本的 Ruby 上测试一个应用程序，这将非常有用。

Ruby on Rails 网站推荐使用 `rbenv` 和 `ruby-build` 插件来管理 Ruby 的安装。`rbenv` 命令用于在 Ruby 版本之间切换，而 `ruby-build` 提供了 `rbenv install` 命令，用于安装不同版本的 Ruby。

### 安装 rbenv

如果你使用的是 Mac OS X，`rbenv` 和 `ruby-build` 都可以通过 Homebrew 安装。有关安装 Homebrew 的说明，请访问 *[`brew.sh/`](http://brew.sh/)*。

打开终端，输入 **`brew install rbenv ruby-build`**，然后跳转到 安装 Ruby。

在 Linux 上，通过从 GitHub 克隆代码来安装 `rbenv` 和 `ruby-build`，如下所示。完整的安装说明可以在线查看，地址为 *[`github.com/sstephenson/rbenv/`](https://github.com/sstephenson/rbenv/)*。

首先，确保你已安装适当的开发工具。`ruby-build`的 wiki 页面* [`github.com/sstephenson/ruby-build/wiki/`](https://github.com/sstephenson/ruby-build/wiki/)*包含了大多数流行 Linux 发行版的建议构建环境。例如，在 Ubuntu 上，输入以下命令来安装编译 Ruby 所需的一切。

```
$ **sudo apt-get install autoconf bison build-essential git \**
                       **libssl-dev libyaml-dev libreadline6 \**
                       **libreadline6-dev zlib1g zlib1g-dev**
Reading package lists... Done
Building dependency tree
--*snip*--
Do you want to continue? [Y/n]
```

输入字母**`y`**来安装这些包，然后按回车。其他 Linux 发行版所需的包可以在上面的 wiki 页面中找到。

接下来，输入以下命令将`rbenv`的 git 仓库克隆到你的主目录。

```
$ **git clone https://github.com/sstephenson/rbenv.git ~/.rbenv**
Cloning into '/home/ubuntu/.rbenv'...
--*snip*--
Checking connectivity... done.
```

然后，将*~/.rbenv/bin*目录添加到你的`$PATH`中，并在*.bashrc*文件中添加一行，以便每次登录时初始化`rbenv`。

```
$ **echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc**
$ **echo 'eval "$(rbenv init -)"' >> ~/.bashrc**
$ **source ~/.bashrc**
```

最后，通过将`rbenv`插件目录克隆到`ruby-build`的 git 仓库来安装`ruby-build`，可以使用以下命令。

```
$ **git clone https://github.com/sstephenson/ruby-build.git \**
            **~/.rbenv/plugins/ruby-build**
Cloning into '/home/ubuntu/.rbenv/plugins/ruby-build'...
--*snip*--
Checking connectivity... done.
```

一旦安装了`rbenv`和`ruby-build`，你就可以安装 Ruby 了。

### 安装 Ruby

输入命令**`rbenv install -l`**来列出当前可用的 Ruby 版本。

```
$ **rbenv install -l**
Available versions:
  1.8.6-p383
  1.8.6-p420
  1.8.7-p249
  1.8.7-p302
  --*snip*--
```

忽略以*jruby*、*rbx*和*ree*等词开头的版本。现在只需关注版本号。截止本文写作时，最新版本是 2.1.1。如果在你安装 rbenv 时有更新的版本，请将下面命令中的 2.1.1 替换为正确的版本号。

```
$ **rbenv install 2.1.1**
Downloading yaml-0.1.6.tar.gz...
--*snip*--
Installed ruby-2.1.1 to /home/ubuntu/.rbenv/versions/2.1.1
```

一旦完成，输入**`rbenv global 2.1.1`**来设置系统的全局默认 Ruby 版本。现在通过输入**`gem install rails`**安装 Ruby on Rails。最后，输入**`rbenv rehash`**来更新`rbenv`。你可以在`rbenv`官网* [`github.com/sstephenson/rbenv/`](https://github.com/sstephenson/rbenv/)*了解更多关于`rbenv`如何切换 Ruby 版本的内容。
