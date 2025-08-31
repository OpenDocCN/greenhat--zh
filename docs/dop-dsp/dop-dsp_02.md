# 第一章：设置虚拟机

![](img/chapterart.png)

*配置*（即设置）虚拟机（VM）是为特定目的配置虚拟机的过程。这个目的可以是运行应用程序、在不同平台上测试软件，或应用更新。

设置虚拟机需要两个步骤：创建和配置。在这个示例中，您将使用 Vagrant 和 Ansible 来构建和配置虚拟机。Vagrant 自动化了虚拟机的创建过程，而 Ansible 在虚拟机运行后对其进行配置。您将在本地的 VirtualBox 上设置并测试虚拟机。这个过程类似于在云中创建和配置服务器。您现在设置的虚拟机将是本书第一部分所有示例的基础。

## 为什么要使用代码来构建基础设施？

使用代码来构建和配置基础设施使您能够始终如一、快速且高效地管理和部署应用程序。这使得您的基础设施和服务可以扩展。它还可以降低运营成本，减少灾难恢复时间，并最小化配置错误的机会。

将基础设施视为代码的另一个好处是易于部署。应用程序在交付流水线中以相同的方式构建和测试。例如，像 Docker 镜像这样的工件会被一致地创建和部署，使用相同版本的库和程序。将基础设施视为代码使您能够构建可重用的组件，使用测试框架，并应用标准的软件工程最佳实践。

然而，有时将基础设施视为代码可能会显得过于复杂。例如，如果您只需要建立一个虚拟机或运行一个简单的 Bash 脚本，可能不值得花费时间和精力来创建所有基础设施和 CM 代码，以完成一个您可以在五分钟内完成的任务。做决策时请根据具体情况作出最佳判断。

## 使用 Vagrant 入门

*Vagrant* 是一个框架，使得创建和管理虚拟机变得简单。它支持多种操作系统（OS），可以运行在多个平台上。Vagrant 使用一个名为 *Vagrantfile* 的配置文件来以代码描述虚拟环境。您将使用这个文件来创建您的本地基础设施。

### 安装

要安装 Vagrant，请访问 Vagrant 的官方网站 [`www.vagrantup.com/downloads.html`](https://www.vagrantup.com/downloads.html)。选择适合您的主机操作系统和架构的版本。完成安装后，下载二进制文件并按照您的操作系统特定的说明进行安装。例如，由于我使用的是 Mac，所以我会选择 macOS 64 位的链接下载最新版本。

当你的虚拟机启动时，你还需要确保它安装了 VirtualBox 的来宾增强功能。（在跟随本书引言的过程中，你应该已经安装了 VirtualBox。）*来宾增强功能*提供更好的驱动支持、端口转发和仅主机网络功能。它们帮助你的虚拟机运行得更快，并提供更多可用选项。安装 Vagrant 后，在终端输入以下命令来安装 Vagrant 的来宾增强插件：

```
$ **vagrant plugin install vagrant-vbguest**
Installing the 'vagrant-vbguest' plugin. This can take a few minutes...
Fetching vagrant-vbguest-0.30.0.gem
Installed the plugin 'vagrant-vbguest (0.30.0)'!
```

上面的输出显示了成功安装 Vagrant 的 `vbguest` 插件。你的插件版本很可能会有所不同，因为新版本会定期发布。每次升级 Vagrant 和 VirtualBox 时，最好更新这个插件。

### Vagrantfile 的结构

Vagrantfile 描述了如何构建和配置虚拟机（VM）。最佳实践是每个项目使用一个 Vagrantfile，这样你可以将配置文件添加到项目的版本控制中并与团队共享。配置文件的语法是 Ruby 编程语言，但你只需要理解一些基本原理即可开始使用。

本书提供的 Vagrantfile 包含文档和合理的选项，可以为你节省时间。这个文件太大，无法在这里包含，因此我只会讨论我从 Vagrant 默认设置中更改的部分。你将从文件的顶部开始，一直到文件的底部，所以可以随时打开文件并跟着做。该文件位于你从本书引言中克隆的仓库中的 *vagrant/* 目录下。在本章稍后的部分，你将使用这个文件来创建你的虚拟机。

#### 操作系统

Vagrant 默认支持许多操作系统基础镜像，称为 *boxes*。你可以在 [`app.vagrantup.com/boxes/search/`](https://app.vagrantup.com/boxes/search/) 查找 Vagrant 支持的 boxes 列表。一旦找到你想要的，使用 `vm.box` 选项将其设置在 Vagrantfile 的顶部，如下所示：

```
config.**vm.box** = **"ubuntu/focal64"**
```

在这个例子中，我将 `vm.box` 标识符设置为 `ubuntu/focal64`。

#### 网络配置

你可以为不同的网络场景配置虚拟机的网络选项，如 *静态 IP* 或 *动态主机配置协议（DHCP）*。为此，请在文件中部修改 `vm.network` 选项：

```
config.**vm.network** **"private_network"**,type:**"dhcp"**
```

对于这个例子，你希望虚拟机通过 DHCP 从私有网络获取 IP 地址。这样，你就可以轻松地从本地主机访问虚拟机上的资源，如 Web 服务器。

#### 提供者

*提供者*是一个插件，它知道如何创建和管理虚拟机。Vagrant 支持多个提供者来管理不同类型的机器。每个提供者都有类似 CPU、磁盘和内存的常见选项。Vagrant 将使用提供者的应用编程接口（API）或命令行选项来创建虚拟机。你可以在 [`www.vagrantup.com/docs/providers/`](https://www.vagrantup.com/docs/providers/) 找到支持的提供者列表。提供者设置在文件的底部，看起来像这样：

```
 config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.name = "dftd"
    `--snip--`
  end
```

### 基本的 Vagrant 命令

现在你知道了 Vagrantfile 的布局，让我们来看看一些基本的 Vagrant 命令。你最常用的四个命令是`vagrant up`、`vagrant destroy`、`vagrant status`和`vagrant ssh`：

1.  `vagrant up` 使用 Vagrantfile 作为指南创建虚拟机

1.  `vagrant destroy` 销毁正在运行的虚拟机

1.  `vagrant status` 检查虚拟机的运行状态

1.  `vagrant ssh` 通过安全外壳（Secure Shell）访问虚拟机

这些命令都有额外的选项。要查看它们，输入命令后添加`--help`标志以获取更多信息。要了解更多关于 Vagrant 功能的信息，请访问[`www.vagrantup.com/docs/`](https://www.vagrantup.com/docs/)上的文档。

一旦通过运行`vagrant up`创建了虚拟机，你将得到一个核心的 Linux 系统，包含所有操作系统的默认设置。接下来，让我们来看一下如何通过配置管理应用你自己的设置。

## 入门指南：Ansible

*Ansible* 是一个配置管理（CM）工具，可以协调虚拟机等基础设施的配置。Ansible 使用*声明式配置风格*，这意味着它允许你描述基础设施的期望状态。这与*命令式配置风格*不同，后者要求你提供关于期望状态的所有细节。由于采用声明式风格，Ansible 是一个非常适合不太懂系统管理的开发人员的工具。Ansible 还是开源软件，并且免费使用。

Ansible 是用 Python 编写的，但你不需要理解 Python 就能使用它。你需要理解的唯一依赖项是*Yet Another Markup Language (YAML)*，它是一种数据序列化语言，Ansible 用它来描述复杂的数据结构和任务。通过查看一些基本示例，它很容易上手，稍后我在讲解 Ansible 的 playbook 和任务时会提供一些示例。这里有两个重要的要点需要注意：YAML 使用缩进来组织元素，像 Python 一样，并且它是区分大小写的。你可以在[`docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html`](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html)上阅读更多关于 YAML 的内容。

Ansible 通过*安全外壳（SSH）*应用配置更改，SSH 是一种与远程主机通信的安全协议。SSH 最常见的用途是获得远程主机上的命令行访问权限，但用户也可以使用它转发网络流量并安全地复制文件。通过使用 SSH，Ansible 可以通过网络对单个主机或一组主机进行配置。

### 安装

现在，你应该安装 Ansible，以便 Vagrant 可以用它进行配置。访问 Ansible 的文档 [`docs.ansible.com/ansible/latest/installation_guide/intro_installation.html`](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)。查找适合你操作系统的文档，并按照步骤安装 Ansible。例如，我使用的是 macOS，安装 Ansible 的推荐方式是使用 *pip*，这是一个用于安装应用程序和依赖项的 Python 包管理器。我是在安装 Ansible 的 macOS 链接下找到这些信息的，最终将我引导到通过安装 pip 安装 Ansible 的链接。由于 Ansible 是用 Python 编写的，使用 pip 是安装最新版本的有效方法。

### Ansible 关键概念

现在你已经安装了 Ansible，你需要了解这些术语和概念，以便快速让它运行：

1.  Playbook A *playbook* 是一系列有序的任务或角色，您可以使用它来配置主机。

1.  控制节点 A *控制节点* 是任何安装了 Ansible 的 Unix 机器。你将从控制节点运行你的 playbook 或命令，并且可以有任意数量的控制节点。

1.  库存 A *库存* 是一个文件，包含 Ansible 可以通信的主机或主机组的列表。

1.  模块 A *模块* 封装了如何在不同操作系统中执行某些操作的细节，比如如何安装软件包。Ansible 自带了许多模块。

1.  任务 A *任务* 是在管理主机上执行的命令或操作（例如安装软件或添加用户）。

1.  角色 A *角色* 是一组任务和变量，组织在一个标准化的目录结构中，定义了服务器的特定用途，并且可以与其他用户共享以达成共同目标。一个典型的角色可能会配置主机为数据库服务器。这个角色将包括安装数据库应用程序、配置用户权限和应用种子数据所需的所有文件和说明。

### Ansible Playbook

要配置虚拟机，你将使用我提供的 Ansible playbook。这个名为 *site.yml* 的文件位于你从介绍中克隆的 *ansible/* 目录下。把 playbook 当作一本关于如何组装主机的说明书。现在，看看 playbook 文件本身。导航到 *ansible/* 目录，并在你的编辑器中打开 *site.yml* 文件。

你可以将 playbook 文件拆分为不同的部分。第一部分充当头部，这里是设置全局变量的好地方，可以在整个 playbook 中使用。在头部，你将设置 `name`（play 名称）、`hosts`、`remote_user` 和特权升级方法等内容：

```
---
- name: Provision VM
  hosts: all
  become: yes
  become_method: sudo
  remote_user: ubuntu
`--snip--`
```

这些设置大多是样板代码，但我们先集中讨论一些要点。务必为每个剧本（play）指定一个`name`，这样如果出现问题，能更容易定位和调试。上述示例中的剧本（play）的`name`被设置为`Provision VM`。你可以在单个剧本中包含多个剧本，但对于此示例，只需要一个。接下来，`hosts`选项设置为`all`，以匹配任何由 Vagrant 构建的虚拟机，因为 Vagrant 会动态生成 Ansible 清单文件。一些主机上的操作可能需要提升的权限，因此 Ansible 允许你为特定用户*提升*权限或激活权限提升。由于你使用的是 Ubuntu，默认的具有提升权限的用户是`ubuntu`。你还可以设置不同的授权方法，在此示例中，你将使用`sudo`。

下一部分是列出主机的所有任务。在这里，实际的工作将被执行。如果你把剧本当作一本说明书，那么*任务*就像说明书中的每个独立步骤。`tasks`部分的格式如下：

```
`--snip--`
  tasks:
   **#- import_tasks: chapter2/pam_pwquality.yml**
   **#- import_tasks: chapter2/user_and_group.yml**
`--snip--`
```

内置的 Ansible `import_tasks`函数从两个独立的文件中加载任务：*pam_pwquality.yml* 和 *user_and_group.yml*。`import_tasks`函数可以更好地组织任务，避免产生一个庞大杂乱的剧本。这些文件每个可以包含一个或多个独立的任务。我将在未来的章节中讨论任务和剧本的其他部分。现在，请注意，这些任务已经用井号（`#`）符号注释掉，直到取消注释它们之前，它们不会改变任何东西。

### 基本的 Ansible 命令

Ansible 应用程序带有多个命令，但你通常只会使用这两个命令：`ansible` 和 `ansible-playbook`。

你主要使用`ansible`命令来运行临时或一次性命令，这些命令通常是从命令行执行的。例如，要指示一组 web 服务器重启 Nginx，你可以输入以下命令：

```
$ **ansible** **webservers-m service -a "name=nginx state=restarted" --become**
```

这条指令让 Ansible 在一个名为*webservers*的主机组上重启 Nginx。请注意，*webservers*组的映射将在清单文件中找到。Ansible 的`service`模块与操作系统交互以执行重启操作。`service`模块需要一些额外的参数，这些参数通过`-a`标志传递。在此情况下，既指定了`service`的名称（`nginx`），也指定了它应该重启。你需要*root*权限才能重启`service`，因此你将使用`--become`标志来请求提升权限。

`ansible-playbook`命令用于运行剧本。事实上，这是 Vagrant 在配置阶段使用的命令。为了让`ansible-playbook`命令对名为*dockerhosts*的主机组执行*aws-cloudwatch.yml*剧本，你需要在终端中输入以下命令：

```
$ **ansible-playbook -l dockerhosts aws-cloudwatch.yml**
```

`dockerhosts`需要在库存文件中列出，才能使命令成功运行。请注意，如果你没有使用`-l`标志提供主机子集，Ansible 会默认认为你希望在库存文件中的所有主机上运行这个 playbook。

## 创建一个 Ubuntu 虚拟机

到目前为止，我们一直在讨论概念和配置文件。现在，让我们将这些知识付诸实践，站起来并配置一些基础设施。要创建 Ubuntu 虚拟机，请确保你在与 Vagrantfile 相同的目录中。这是因为 Vagrant 在创建虚拟机时需要引用配置文件。你将使用`vagrant up`命令来创建虚拟机，但在运行命令之前，你应该知道它会产生大量的输出并且可能需要几分钟时间。因此，我在这里只关注相关部分。在终端中输入以下命令：

```
$ **vagrant up**
```

输出的第一部分是 Vagrant 下载基础镜像：

```
`--snip--`
Bringing machine 'default' up with 'virtualbox' provider...
==> default: **Importing base box 'ubuntu/focal64'**...
`--snip--`
```

在这里，Vagrant 正在下载`ubuntu`镜像，正如预期的那样。镜像的下载可能需要几分钟时间，具体取决于你的网络连接。

接下来，Vagrant 将配置一个公钥/私钥对，以提供 SSH 访问虚拟机的权限。（我们将在第三章详细讨论密钥对。）

```
`--snip--` 
default: Vagrant insecure key detected. Vagrant will automatically replace
default: this with a newly generated keypair for better security.
default:
default: Inserting generated public key within guest...
default: Removing insecure key from the guest if it's present...
default: Key inserted! Disconnecting and reconnecting using new SSH key...
`--snip--`
```

Vagrant 会将私钥保存在本地主机(*.vagrant/*)中，然后将公钥添加到虚拟机的*~/.ssh/authorized_keys*文件中。如果没有这些密钥，你将无法通过 SSH 连接到虚拟机。

默认情况下，Vagrant 和 VirtualBox 会在虚拟机内挂载一个共享目录。这个共享目录将允许你从虚拟机内访问主机的目录：

```
`--snip--`
==> default: Mounting shared folders...
   default: /vagrant => /Users/bradleyd/devops_for_the_desperate/vagrant
`--snip--`
```

你可以看到，我的本地主机目录*Users/bradleyd/devops_for_the_desperate/*已挂载在虚拟机内部的*vagrant/*目录下。你的目录会有所不同。你可以使用这个共享目录在主机和虚拟机之间传输文件，比如源代码。如果你不需要共享目录，Vagrant 提供了一个配置选项来关闭它。详情请参阅 Vagrant 的文档。

最后，以下是 Ansible `provisioner`的输出：

```
`--snip--`
==> default: Running provisioner: ansible...
    default: Running 1ansible-playbook...

PLAY [Provision VM] *******************************
2 TASK [Gathering Facts] ****************************
3 ok:  [default]

PLAY RECAP *******************************
`--snip--`
default	   : ok=1    4changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

这表明 Ansible `provisioner`正在使用`ansible-playbook` 1 命令运行。Ansible 会记录每个`TASK` 2 及其是否在主机 3 上有所更改。在这种情况下，所有`tasks`都被注释掉了，因此虚拟机上没有任何`changed` 4。这是判断成功与否时需要查看的第一个输出。

让我们进行一个简单的检查，看看虚拟机是否实际在运行。在终端中输入以下命令，查看虚拟机的当前状态：

```
$ **vagrant status**
Current machine states:
default    running (virtualbox)
`--snip--`
```

在这里，你可以看到虚拟机的状态是`running`。这意味着你已经创建了虚拟机，并且它应该可以通过 SSH 访问。

如果你的输出与预期不同，请确保在继续之前，`vagrant up` 命令没有错误。如果需要更多信息，请向 `up` 命令添加 `debug` 标志，以增加 Vagrant 输出的详细程度：`vagrant up --debug`。此时你需要确保已成功完成配置，否则在接下来的章节中会很难跟上。

## 总结

在本章中，你安装了 Vagrant 和 Ansible 来创建和配置虚拟机（VM）。你学习了如何通过 Vagrantfile 配置 Vagrant，并掌握了使用 Ansible playbooks 和任务来配置虚拟机的基本知识。现在，你已经理解了这些基本概念，应该能够创建并配置任何类型的基础设施，而不仅仅是虚拟机。

在下一章中，你将使用提供的两个 Ansible 任务来创建一个用户和组。在配置主机时，你需要具备用户和组管理的基础知识。
