# 第五章：E

故障排除部署

![](img/chapterart.png)

部署应用程序在成功时是非常令人满意的，尤其是如果你以前从未做过。但是，在部署过程中可能会出现许多障碍，不幸的是，这些问题中的一些可能难以识别和解决。本附录将帮助你理解现代部署方法，并在出现问题时提供具体的故障排除方法。

如果本附录中的额外信息不足以帮助你顺利完成部署过程，请查看[`ehmatthes.github.io/pcc_3e`](https://ehmatthes.github.io/pcc_3e)上的在线资源；那里更新的内容几乎肯定会帮助你成功完成部署。

## 理解部署

当你试图故障排除特定的部署尝试时，清楚地理解一个典型的部署过程是很有帮助的。*部署*是指将一个在本地系统上正常工作的项目复制到远程服务器的过程，以便它可以响应来自互联网上任何用户的请求。远程环境与典型的本地系统在多个重要方面有所不同：它可能不是你正在使用的相同操作系统（OS），而且它很可能是单一物理服务器上的多个虚拟服务器之一。

当你部署一个项目，或将其*推送*到远程服务器时，需要执行以下步骤：

+   在数据中心的物理机器上创建一个虚拟服务器。

+   建立本地系统和远程服务器之间的连接。

+   将项目的代码复制到远程服务器。

+   确定项目的所有依赖项，并将它们安装在远程服务器上。

+   设置数据库并运行现有的迁移。

+   将静态文件（CSS、JavaScript 文件和媒体文件）复制到可以高效提供服务的位置。

+   启动一个服务器来处理传入的请求。

+   一旦项目准备好处理请求，开始将传入的请求路由到该项目。

当你考虑到部署过程中涉及的所有步骤时，部署失败并不令人意外。幸运的是，一旦你理解了部署应该发生的过程，你就更有可能找出出错的地方。如果你能识别出问题所在，或许能够找到一个解决方法，让下一次的部署尝试成功。

你可以在一种操作系统上进行本地开发，然后将其推送到运行不同操作系统的服务器上。了解你推送到的系统类型很重要，因为这可以指导你的一些故障排除工作。在本文写作时，Platform.sh 上的基础远程服务器运行 Debian Linux；大多数远程服务器都是基于 Linux 的系统。

## 基础故障排除

一些故障排除步骤是特定于每个操作系统的，但我们稍后会讨论这些内容。首先，让我们考虑在故障排除部署时每个人都应该尝试的步骤。

你最好的资源就是在尝试推送时生成的输出。这个输出可能看起来令人畏惧；如果你是应用部署的新手，它可能看起来非常技术化，而且通常会有很多内容。好消息是，你不需要理解输出中的所有内容。你在浏览日志输出时应该有两个目标：识别任何成功的部署步骤，并识别任何失败的步骤。如果你能做到这一点，你也许能够找出需要在项目或部署过程中更改的地方，从而使下一次推送成功。

### 按照屏幕上的建议操作

有时候，你推送的平台会生成一条消息，给出如何解决问题的明确建议。例如，如果你在初始化 Git 仓库之前创建了一个 Platform.sh 项目，然后尝试推送该项目，你将看到以下消息：

```py
$ **platform push**
❶ Enter a number to choose a project:
  [0] ll_project (votohz445ljyg)
 > **0**

❷   [RootNotFoundException]
  Project root not found. This can only be run from inside a project directory.

❸  To set the project for this Git repository, run:
    platform project:set-remote [id]
```

我们正在尝试推送一个项目，但本地项目尚未与远程项目关联。所以，Platform.sh CLI 会询问我们想要推送到哪个远程项目❶。我们输入`0`，选择唯一列出的项目。但接下来，我们看到一个`RootNotFoundException` ❷。发生这种情况是因为 Platform.sh 在检查本地项目时会查找 *.git* 目录，以便弄清楚如何将本地项目与远程项目连接起来。在这种情况下，由于创建远程项目时没有 *.git* 目录，因此该连接从未建立。CLI 提供了一个修复建议❸；它告诉我们可以使用 `project:set-remote` 命令指定应与此本地项目关联的远程项目。

让我们尝试这个建议：

```py
$ **platform project:set-remote votohz445ljyg**
Setting the remote project for this repository to: ll_project (votohz445ljyg)

The remote project for this repository is
    now set to: ll_project (votohz445ljyg)
```

在之前的输出中，CLI 显示了该远程项目的 ID，`votohz4451jyg`。所以我们运行了建议的命令，使用这个 ID，CLI 成功地将本地项目与远程项目连接起来。

现在，让我们再次尝试推送项目：

```py
$ **platform push**
Are you sure you want to push to the main (production) branch? [Y/n] **y**
Pushing HEAD to the existing environment main
`--snip--`
```

这是一次成功的推送；按照屏幕上的建议操作有效。

你应该小心运行你不完全理解的命令。然而，如果你有充分的理由相信某个命令的危害很小，并且你信任推荐的来源，那么尝试使用工具提供的建议可能是合理的。

### 阅读日志输出

如前所述，运行类似 `platform push` 的命令时，日志输出既可以提供信息，也可能让人感到畏惧。请阅读以下日志输出片段，这是另一次使用 `platform push` 的尝试，看看你能否找出问题所在：

```py
`--snip--`
Collecting soupsieve==2.3.2.post1
  Using cached soupsieve-2.3.2.post1-py3-none-any.whl (37 kB)
Collecting sqlparse==0.4.2
  Using cached sqlparse-0.4.2-py3-none-any.whl (42 kB)
Installing collected packages: platformshconfig, sqlparse,...
Successfully installed Django-4.1 asgiref-3.5.2 beautifulsoup4-4.11.1...
W: ERROR: Could not find a version that satisfies the requirement gunicorrn
W: ERROR: No matching distribution found for gunicorrn

130 static files copied to '/app/static'.

Executing pre-flight checks...
`--snip--`
```

当部署尝试失败时，一个好的策略是查看日志输出，看看是否能发现任何类似警告或错误的信息。警告是相当常见的；它们通常是关于项目依赖项即将发生变化的消息，帮助开发人员在问题导致实际故障之前解决问题。

成功的推送可能会有警告，但不应包含任何错误。在这种情况下，Platform.sh 无法找到安装要求 `gunicorrn` 的方法。这是 *requirements_remote.txt* 文件中的一个拼写错误，原本应该包含 `gunicorn`（少一个 *r*）。在日志输出中，通常不容易直接找到根本问题，尤其是当问题导致一系列级联错误和警告时。就像在本地系统上查看回溯信息一样，最好仔细查看列出的前几个错误和最后几个错误。中间的大多数错误往往是内部包在抱怨某些东西出了问题，并将错误信息传递给其他内部包。我们通常可以解决的实际错误，通常是列出的第一个或最后一个错误。

有时候，你可以很容易发现错误，但有时候，你根本不明白输出的含义。这绝对值得一试，利用日志输出成功诊断错误是一种非常满足的感觉。当你花更多时间查看日志输出时，你会变得更擅长识别对你最有意义的信息。

## 操作系统特定的故障排除

你可以在任何你喜欢的操作系统上进行开发，并推送到任何你喜欢的主机上。推送项目的工具已经足够成熟，它们会根据需要修改你的项目，以确保能够在远程系统上正确运行。然而，仍然可能会出现一些特定操作系统的问题。

在 Platform.sh 的部署过程中，最可能遇到的困难之一是安装 CLI。下面是安装命令：

```py
$ **curl -fsS https://platform.sh/cli/installer | php**
```

该命令以 `curl` 开头，`curl` 是一个让你在终端中通过 URL 请求远程资源的工具。这里，它用于从 Platform.sh 服务器下载 CLI 安装程序。命令中的 `-fsS` 部分是一个标志集，用来修改 `curl` 的运行方式。`f` 标志告诉 `curl` 抑制大多数错误消息，这样 CLI 安装程序可以处理这些错误，而不是把所有错误报告给你。`s` 标志告诉 `curl` 静默运行；它允许 CLI 安装程序决定在终端中显示什么信息。`S` 标志告诉 `curl` 如果整体命令失败，则显示错误消息。命令结尾的 `| php` 告诉系统使用 PHP 解释器运行下载的安装程序文件，因为 Platform.sh 的 CLI 是用 PHP 编写的。

这意味着你的系统需要安装 `curl` 和 PHP 才能安装 Platform.sh CLI。要使用该 CLI，你还需要 Git 和能够运行 Bash 命令的终端。*Bash* 是一种大多数服务器环境中都可以使用的语言。大多数现代系统都有足够的空间安装这些工具。

以下章节将帮助你解决适用于你操作系统的相关需求。如果你尚未安装 Git，请参考附录 D 中第 484 页的 Git 安装说明，然后查看适用于你操作系统的部分。

### 从 Windows 部署

近年来，Windows 在程序员中重新获得了广泛的关注。Windows 集成了许多其他操作系统的不同元素，提供了多种方式让用户进行本地开发工作并与远程系统交互。

从 Windows 部署的最大困难之一是，核心的 Windows 操作系统与 Linux 基础的远程服务器所使用的系统不同。一个基础的 Windows 系统拥有与基础 Linux 系统不同的工具和语言，因此，要从 Windows 执行部署工作，你需要选择如何将基于 Linux 的工具集集成到本地环境中。

#### Windows Subsystem for Linux

一种流行的方法是使用 *Windows Subsystem for Linux* (*WSL*)，它是一个允许 Linux 在 Windows 上直接运行的环境。如果你已经设置好 WSL，那么在 Windows 上使用 Platform.sh CLI 就和在 Linux 上一样简单。CLI 并不会知道它是在 Windows 上运行，它只会看到你正在使用的 Linux 环境。

设置 WSL 是一个两步过程：首先安装 WSL，然后选择一个 Linux 发行版并将其安装到 WSL 环境中。设置 WSL 环境的过程较为复杂，无法在这里详细描述；如果你对这种方法感兴趣并且尚未设置，可以查看 [`docs.microsoft.com/en-us/windows/wsl/about`](https://docs.microsoft.com/en-us/windows/wsl/about) 上的文档。一旦你设置好了 WSL，就可以按照本附录中 Linux 部分的说明继续部署工作。

#### Git Bash

另一种构建本地环境的方法是使用 *Git Bash*，它是一个与 Bash 兼容的终端环境，但可以在 Windows 上运行。当你从 [`git-scm.com`](https://git-scm.com) 使用安装程序时，Git Bash 会和 Git 一起安装。这个方法是可行的，但没有 WSL 那么简化。在这种方法中，你需要在一些步骤中使用 Windows 终端，而在其他步骤中则使用 Git Bash 终端。

首先，你需要安装 PHP。你可以通过 *XAMPP* 来完成这一点，它是一个将 PHP 和其他一些开发者工具捆绑在一起的套件。访问 [`apachefriends.org`](https://apachefriends.org)，点击按钮下载适用于 Windows 的 XAMPP。打开安装程序并运行它；如果看到有关用户帐户控制（UAC）限制的警告，点击 **OK**。接受所有安装程序的默认设置。

当安装程序运行完成后，你需要将 PHP 添加到系统的路径中；这将告诉 Windows 在你想运行 PHP 时该去哪里查找。在开始菜单中输入`path`，然后点击**编辑系统环境变量**；点击标记为**环境变量**的按钮。你应该看到变量`Path`被高亮显示；在此面板下点击**编辑**。点击**新建**，将一个新的路径添加到当前的路径列表中。假设你在运行 XAMPP 安装程序时保持了默认设置，在出现的框中添加`C:\xampp\php`，然后点击**确定**。完成后，关闭所有仍然打开的系统对话框。

在处理好这些需求之后，你可以安装 Platform.sh CLI。你需要使用具有管理员权限的 Windows 终端；在开始菜单中输入`command`，然后在命令提示符应用下，点击**以管理员身份运行**。在弹出的终端中，输入以下命令：

```py
> **curl -fsS https://platform.sh/cli/installer | php**
```

这将安装 Platform.sh CLI，正如前面所描述的那样。

最后，你将在 Git Bash 中进行操作。要打开 Git Bash 终端，进入开始菜单并搜索`git bash`。点击出现的**Git Bash 应用程序**，你应该会看到一个终端窗口打开。在这个终端中，你可以使用传统的基于 Linux 的命令，如`ls`，以及基于 Windows 的命令，如`dir`。为了确保安装成功，执行`platform list`命令。你应该看到 Platform.sh CLI 中所有命令的列表。从此以后，所有的部署工作都将在 Git Bash 终端窗口内使用 Platform.sh CLI 完成。

### 从 macOS 部署

macOS 操作系统并不是基于 Linux，但它们都遵循相似的原则开发。实际上，这意味着你在 macOS 上使用的许多命令和工作流程也可以在远程服务器环境中使用。你可能需要安装一些面向开发者的资源，才能在本地的 macOS 环境中使用这些工具。如果在工作中任何时候出现提示要求安装*命令行开发工具*，点击**安装**以批准安装。

安装 Platform.sh CLI 时最可能遇到的困难是确保 PHP 已安装。如果你看到提示`php`命令未找到，你需要安装 PHP。安装 PHP 的一种最简单方法是使用*Homebrew*包管理器，它简化了程序员所依赖的各种软件包的安装。如果你还没有安装 Homebrew，可以访问[`brew.sh`](https://brew.sh)并按照指示安装。

一旦 Homebrew 安装完成，使用以下命令来安装 PHP：

```py
$ **brew install php**
```

这可能需要一段时间才能完成，但完成后，你应该能够成功安装 Platform.sh CLI。

### 从 Linux 部署

因为大多数服务器环境都是基于 Linux 的，所以你应该很容易安装和使用 Platform.sh CLI。如果你尝试在新安装的 Ubuntu 系统上安装 CLI，它会告诉你需要哪些软件包：

```py
$ **curl -fsS https://platform.sh/cli/installer | php**
Command 'curl' not found, but can be installed with:
sudo apt  install curl
Command 'php' not found, but can be installed with:
sudo apt install php-cli
```

实际输出会提供更多关于其他一些可用软件包的信息，并包含一些版本信息。以下命令将安装 curl 和 PHP：

```py
$ **sudo apt install curl php-cli**
```

运行此命令后，Platform.sh CLI 安装命令应该会成功运行。由于你的本地环境与大多数基于 Linux 的托管环境非常相似，因此你在终端中学习的许多内容也将适用于远程环境。

## 其他部署方法

如果 Platform.sh 不适合你，或者你想尝试不同的方法，市面上有许多托管平台可以选择。有些平台的工作方式与第二十章中描述的过程类似，而有些则采用与本附录开头描述的步骤完全不同的方法：

+   Platform.sh 允许你使用浏览器来执行我们使用 CLI 进行的步骤。如果你更喜欢基于浏览器的界面，而不是终端工作流，可能会更喜欢这种方法。

+   还有许多其他托管服务提供商，提供 CLI 和基于浏览器的方法。有些提供商在其浏览器内提供终端，因此你无需在系统上安装任何东西。

+   一些提供商允许你将项目推送到像 GitHub 这样的远程代码托管站点，然后将你的 GitHub 仓库连接到托管站点。主机会从 GitHub 拉取代码，而不是要求你将代码从本地系统直接推送到主机。Platform.sh 也支持这种工作流程。

+   一些提供商提供一系列服务供你选择，以便组建适合你项目的基础设施。这通常需要你对部署过程有更深入的了解，以及远程服务器为服务项目所需的内容。这些主机包括 Amazon Web Services（AWS）和微软的 Azure 平台。在这些平台上，追踪费用可能更困难，因为每项服务都可能独立产生费用。

+   很多人将他们的项目托管在虚拟私人服务器（VPS）上。在这种方法中，你租用一个虚拟服务器，它就像一台远程计算机，你可以登录到服务器，安装运行项目所需的软件，复制代码，设置正确的连接，并允许服务器开始接收请求。

新的托管平台和方法定期出现；找到一个对你有吸引力的，并投入时间学习该提供商的部署流程。保持你的项目足够长的时间，直到你了解什么方法适合你的提供商，什么方法不适合。没有任何托管平台是完美的；你需要持续评估你目前使用的提供商是否足够适合你的使用场景。

我将对选择部署平台和整体部署方法给出最后的警告。有些人会热衷于引导你选择过于复杂的部署方法和服务，这些方法和服务旨在让你的项目具有高度可靠性，并能够同时为数百万用户提供服务。许多程序员花费大量时间、金钱和精力来构建复杂的部署策略，结果却发现几乎没有人使用他们的项目。大多数 Django 项目可以在一个小型托管计划上设置，并调整到每分钟处理成千上万的请求。如果你的项目的流量低于这个水平，在投资那些适用于全球最大网站的基础设施之前，先花时间配置一个能在最小平台上顺利运行的部署。

部署有时极具挑战性，但当你的实时项目运作顺利时，也同样令人满足。享受这个挑战，遇到困难时，及时寻求帮助。
