# 第六章 部署

现在你已经构建了一个应用程序，我们来把它放到网上供大家查看。Rails 应用程序可以通过多种方式部署。Rails 可以运行在从简单的共享主机到专用服务器，再到云端虚拟服务器等各种环境中。

被称为 Heroku 的云应用平台是部署应用程序最简单的方法之一，我将在本章中讲解它。Heroku 使用 Git 版本控制系统来部署应用程序，所以我们需要先讨论版本控制系统。

# 版本控制

*版本控制系统 (VCS)* 会记录文件随时间变化的情况，因此你可以轻松地回到某个特定版本。*版本库*是一个数据结构，通常存储在服务器上，保存了 VCS 中文件的副本和这些文件的历史变化列表。使用 VCS 时，你可以在修改源代码时，知道自己始终可以回到最后一个有效版本。

最初，版本控制系统是*集中式*的。也就是说，源代码库存储在单一的服务器上。开发者可以连接到该服务器并检出文件以对代码进行修改。但集中式系统也存在单点故障的问题。集中式版本控制系统的例子包括并发版本系统 (CVS) 和 Subversion。

当今最流行的版本控制系统类型是*分布式*的。在分布式版本控制系统中，每个客户端都会存储源代码库的完整副本。这样，如果某个客户端出现故障，其他人仍然可以继续工作且不会丢失数据。

在分布式系统中，通常仍会使用中央服务器。开发者将他们的更改*推送*到该服务器，并*拉取*其他开发者所做的更改。流行的分布式版本控制系统包括 Git 和 Mercurial。由于 Heroku 使用 Git 部署应用程序，所以我将重点讲解 Git。

# Git

Git 最初由 Linus Torvalds 于 2005 年为 Linux 内核开发。*git* 这个词是英国俚语，指的是一个可鄙的人。Torvalds 曾开玩笑说，他将所有的项目都以自己命名。

Git 很快便传播到 Linux 社区之外，现在大多数 Ruby 项目都使用 Git，包括 Ruby on Rails。如果你还没有安装 Git，可以在 Ruby, Rails, and Git 中找到安装说明。

## 设置

在开始使用 Git 之前，设置你的名字和电子邮件地址。打开一个终端窗口并输入以下命令来设置你的名字：

```
$ **git config --global user.name "** ***Your Name***"
```

`--global` 标志告诉 Git 将此更改应用于全局配置。没有此标志时，更改仅会应用于当前版本库。同时，设置你的电子邮件地址：

```
$ **git config --global user.email "** ***you@example.com*** **"**
```

现在每次提交更改时，你的名字和电子邮件地址都会被包含在内，这样在团队协作时就能轻松看到是谁在什么时候做了哪些更改。

## 入门

现在你已经准备好为博客创建一个版本库了。进入你的*code/blog* 目录并输入以下命令：

```
$ **git init**
Initialized empty Git repository in /Users/tony/code/blog/.git/
```

这会在隐藏的 *.git* 子目录中初始化一个空的 Git 仓库。接下来，让我们将应用程序的所有文件添加到仓库中：

```
$ **git add .**
```

`add` 命令接受文件名或目录路径，并将其添加到 Git 的暂存区。暂存区中的文件准备好提交到仓库。当你执行提交时，Git 会拍摄项目当前状态的快照，并将其存储在仓库中。命令中的点表示当前目录。因此，运行此命令后，当前目录及其任何子目录中的所有文件都准备好提交。

现在将所有已暂存的文件提交到仓库：

```
➊ $ **git commit -m "Initial commit"**
  [master (root-commit) e393590] Initial commit
   85 files changed, 1289 insertions(+)
   create mode 100644 .gitignore
   create mode 100644 Gemfile
  --*snip*--
   create mode 100644 test/test_helper.rb
   create mode 100644 vendor/assets/javascripts/.keep
   create mode 100644 vendor/assets/stylesheets/.keep
```

请注意，我通过 `-m` 标志指定了提交信息 `"Initial commit"` ➊。如果不加上此标志，Git 会打开你的默认编辑器，以便你输入提交信息。如果你没有输入提交信息，提交会失败。

如果你想查看当前仓库的提交历史，输入 `git log` 命令。列表按从最新到最旧的顺序显示之前的提交。每个条目都包括提交者和时间，以及提交信息。

```
  $ **git log**
➊ commit e3935901a2562bf8c04c480b3c5681c102985a4e
  Author: Your Name <you@example.com>
  Date:   Wed Apr 2 16:41:24 2014 -0500

      Initial commit
```

每次提交都会由一个独特的 40 字符十六进制哈希值表示 ➊。这些哈希值可以缩写为前七个字符——在这个例子中是 e393590——如果你需要再次引用这个特定的提交。

## 基本用法

在使用 Git 开发项目时，遵循这个基本的工作流：

1.  根据需要编辑本地文件。

1.  使用 `git add` 命令将文件暂存，以便提交。

1.  使用 `git commit` 命令将更改提交到仓库。

你可以根据需要频繁提交更改，但我发现将与单一简单功能或 bug 修复相关的更改一起提交会更有帮助。这样，所有更改都与一个提交绑定，若需要回滚或移除某个功能时会更容易。结束一个工作会话时，提交所有未完成的更改也是个好主意。

### 其他有用的命令

Git 包含许多额外的命令；输入 `git --help` 查看最常用的命令列表。你已经见过 `init`、`add`、`commit` 和 `log` 命令，但这里还有一些在使用 Git 时特别有用的命令。

`git status` 命令显示已更改和新增文件的列表：

```
$ **git status**
On branch master
nothing to commit, working directory clean
```

在这种情况下，没有任何更改。编辑项目中的文件，例如 *README.rdoc*，然后再次输入 `git status` 命令：

```
  **$ git status**
  On branch master
  Changes not staged for commit:
    (use "git add <file>..." to update what will be committed)
    (use "git checkout -- <file>..." to discard changes...)

➊   modified: README.rdoc

   no changes added to commit (use "git add" and/or "git commit -a")
```

`git status` 命令显示当前工作目录和暂存区的状态。在这里，它列出了所有已暂存以待提交的文件，以及那些未暂存但有更改的文件 ➊。

`git diff` 命令显示文件的详细更改：

```
  $ **git diff**
  diff --git a/README.rdoc b/README.rdoc
  index dd4e97e..c7fabfa 100644
  --- a/README.rdoc
  +++ b/README.rdoc
  @@ -1,4 +1,4 @@
➊ -== README
  +== Blog

   This README would normally document whatever steps are necessary to get the
   application up and running.
```

在这里，我将文件第一行中的 *README* 改为 *Blog* ➊。使用此命令可以在执行 `git add` 之前查看将要提交的具体更改。如果只关心单个文件的更改，也可以将文件名传递给此命令。

`git checkout` 命令可以撤销文件的更改：

```
➊ $ **git checkout -- README.rdoc**
  $ **git status**
  On branch master
  nothing to commit, working directory clean
```

在这里，我通过使用 `git checkout` 后跟两个破折号和文件名 ➊ 来丢弃对 *README.rdoc* 文件的更改。此命令没有产生任何输出。然后，我使用 `git status` 确认更改已被丢弃。

`git clone` 命令会创建一个远程仓库的本地副本：

```
$ **git clone** *url*
```

远程仓库由 *`<url>`* 表示。Git 是一个非常适合协作的工具，许多开源项目都在使用它。这个命令使得这一切成为可能。在你开始处理一个现有项目之前，你需要 *克隆* 该仓库的副本到你的电脑上。

### 分支

你可能已经注意到 `git status` 命令中包含了“On branch master”这一短语。在 Git 中，*分支* 是一组命名的更改。默认分支被称为 *master*，它代表了开发的主线。到目前为止，我所做的所有更改都已提交到主分支。

如果你正在处理一个可能需要较长时间完成的大功能，你可以创建一个单独的分支来存储正在进行的更改，而不会影响主分支。这样，你可以在自己的分支上工作，而不会影响团队的其他成员。一旦新功能完成，你将 *合并* 你的新分支到主分支。

使用 `git branch` 命令后跟你选择的分支名称来创建一个新分支。在这个示例中，我将我的分支命名为 *testing*：

```
$ **git branch testing**
```

输入 `git branch` 命令而不指定分支名称，可以查看当前仓库中已存在的所有分支：

```
$ **git branch**
* master
 testing
```

星号显示的是当前选中的分支。我创建了一个新分支，但我仍然在查看主分支。要切换到另一个分支，使用 `git checkout` 命令：

```
$ **git checkout testing**
Switched to branch 'testing'
```

现在我在 testing 分支上。在这里提交的更改不会影响主分支。完成更改后，`checkout` 主分支并将你的更改合并到主分支中：

```
$ **git checkout master**
Switched to branch 'master' $
**git merge testing**
Already up-to-date.
```

来自 testing 分支的所有更改现在也出现在主分支上。你可以使用 `git log` 命令确认这一点。现在你已经完成了 testing 分支的工作，使用 `-d` 标志将 `git branch` 命令删除它：

```
$ **git branch -d testing**
Deleted branch testing (was e393590).
```

在分支合并后你不必删除分支，但删除它们可以保持分支列表的清晰。

### 远程

到目前为止，我们的所有更改都存储在本地，但你应该在另一服务器上存储一份仓库的备份副本，并且便于其他人克隆你的仓库。为此，你需要设置一个远程仓库。*远程* 只是另一个仓库在特定 URL 上的昵称。使用 `git remote add` 命令将昵称与 URL 关联起来：

```
git remote add *name url*
```

一旦你添加了远程仓库，使用 `git push` 命令将更改发送到远程仓库，使用 `git pull` 命令来获取远程的更改。在下一节中，你将看到这个操作的实际示例。

# Heroku

Heroku 是一个云应用程序平台，用于部署 Web 应用程序。这种平台有时被称为*平台即服务（PaaS）*，意味着 Heroku 负责服务器配置和管理，这样你就可以专注于应用程序开发。该服务还包括一系列丰富的附加组件。开始使用是免费的，但需要更多处理器资源和内存的大型应用程序可能会变得非常昂贵。

完成初步设置后，你可以使用`git push`命令部署你的应用程序，并在网页上访问它。

## 开始使用

首先，在*【http://www.heroku.com】(http://www.heroku.com)*注册一个免费账户。记住你选择的密码；你需要它来登录。

接下来，如果你还没有安装 Heroku Toolbelt，请安装它（有关说明，请参见*【http://toolbelt.heroku.com/】(http://toolbelt.heroku.com/)*）。Toolbelt 是 Heroku 提供的一组工具，用于将你的应用程序部署到 Heroku 平台。

现在，打开终端窗口，导航到你的博客目录，并登录 Heroku：

```
$ **heroku login**
Enter your Heroku credentials.
Email: you@example.com
Password (typing will be hidden):
Authentication successful.
```

该命令会提示你输入你的电子邮件地址和之前创建的密码，然后它会检查你电脑上是否存在有效的安全外壳（SSH）公钥。你的公钥是用于通过 SSH 身份验证的公私钥对的一部分。当你尝试登录时，你的私钥会用于生成一个加密数字签名。Heroku 随后使用你的公钥验证该数字签名，从而确认你的身份。

如果你还没有公钥，在提示时按**Y**以创建一个。公钥创建后会自动上传到 Heroku。Heroku 使用你的公钥进行身份验证，这样你每次部署应用程序时就不需要输入密码。

现在你已经登录到 Heroku，你需要准备好你的应用程序进行部署。

## 更新你的 Gemfile

无论你正在构建什么类型的应用程序，你都需要安装某些 gem 来与 Heroku 交互并部署你的应用程序。在这一部分，我们将讨论你需要添加到应用程序的*Gemfile*中的两个 gem。

Heroku 的服务器使用 PostgreSQL 数据库服务器。我们在本地使用 SQLite 进行开发，而不是安装 PostgreSQL。你需要确保在生产环境中安装名为 pg 的 PostgreSQL gem。

Heroku 还需要 rails_12factor gem，它确保 Heroku 的服务器可以提供你的应用程序的资产，并确保你的应用程序的日志文件被发送到正确的位置。

打开位于 Rails 应用程序根目录下的文件*Gemfile*，找到`gem 'sqlite3'`这一行。你将在生产环境中使用 PostgreSQL gem，但你仍然需要 SQLite gem 用于开发和测试，因此更新这一行，按照如下所示添加`group: [:development, :test]`：

```
gem 'sqlite3'**, group: [:development, :test]**
```

这条指令告诉`bundle`命令仅在开发和测试环境中安装该 gem。

现在你需要安装前面提到的 `pg` 和 `rails_12factor` gem。你只需要在生产环境中使用这些 gem，因此在你刚刚更新的那一行下面添加以下几行：

```
**# gems required by Heroku**
**gem 'pg', group: :production**
**gem 'rails_12factor', group: :production**
```

在你做完这些更改后，保存并关闭 *Gemfile*。因为你更改了应用的 *Gemfile*，需要重新运行 `bundle` 命令来更新依赖。

```
$ **bin/bundle install --without production**
```

因为你是在本地运行此命令，进行应用的开发和测试，所以你不需要安装生产环境的 gem，因此需要添加 `--without production` 标志。Bundler 会记住传递给 `bundle install` 的标志，因此从现在开始每次运行该命令时，都会默认使用 `--without production`。

最后，你需要将这些更改添加并提交到你的 Git 仓库。输入以下命令来更新 Git：

```
$ **git add .**
$ **git commit -m "Update Gemfile for Heroku"**
[master 0338fc6] Update Gemfile for Heroku
2 files changed, 13 insertions(+), 1 deletion(-)
```

你可以在 *Update Gemfile for Heroku* 位置输入任何消息，但提交信息在描述你所做更改时会更有帮助。

现在，你的账户已经设置好了，应用也几乎准备好部署。最后一步是创建一个 Heroku 应用：

```
  $ **heroku create**
➊ Creating glacial-journey-3029... done, stack is cedar
  http://glacial-journey-3029.herokuapp.com/ | git@he...
➋ Git remote heroku added
```

这个命令 ➊ 在 Heroku 的服务器上创建一个具有随机生成名称的新应用。你本可以在 `create` 命令后指定一个名称，但该名称必须是唯一的。如果需要，你可以稍后更改名称。`create` 命令还 ➋ 为你自动设置了一个名为 `heroku` 的 Git 远程仓库。

## 部署你的应用

一切准备就绪，现在你可以最终部署你的应用了。使用 `git push` 命令将主分支的当前状态推送到 Heroku：

```
$ **git push heroku master**
Initializing repository, done.
Counting objects: 102, done.
Delta compression using up to 8 threads.
--*snip*
------> Launching... done, v6
        http://glacial-journey-3029.herokuapp.com/ deployed to Heroku

To git@heroku.com:glacial-journey-3029.git
 * [new branch]      master -> master
```

Heroku 会识别这个 `git push` 命令，并自动检测到正在部署一个 Ruby on Rails 应用，安装 `Gemfile` 中指定的生产环境 gem，更新应用的数据库配置，预编译应用的资源，并启动应用。

当你第一次部署应用时，还需要运行数据库迁移，以便在 Heroku 的数据库服务器中创建应用所需的数据库表。使用 `heroku run` 命令在 Heroku 服务器上执行 `rake db:migrate` 命令：

```
$ **heroku run rake db:migrate**
Running `rake db:migrate` attached to terminal... up, run.1833
Migrating to CreatePosts (20140315004352)
--*snip*--
```

如果你对应用进行了更多的数据库更改，记得将更改提交到 Git 的主分支，推送主分支到 Heroku，并再次运行此命令。

现在你可以打开网页浏览器，访问 Heroku 为你创建的 URL，或者通过输入以下命令让 Heroku 为你处理：

```
$ **heroku open**
```

你的默认网页浏览器应该会自动打开并加载你的博客应用程序。

现在你的应用已经在 Heroku 上设置好了，你可以随时通过提交更改到 Git 仓库并将更改推送到 Heroku 来进行部署。

Github

在 Rails 书籍中讨论 Git 时，如果没有至少提到 GitHub，那将是不完整的。*GitHub* 是全球最大的源代码托管平台。GitHub 提供项目管理功能，如维基、问题追踪和通过拉取请求进行代码审查。

Rails 社区已将 GitHub 作为协作开源软件的最佳平台。Rails 本身托管在 GitHub 上，地址是 *[`github.com/rails/rails/`](https://github.com/rails/rails/)*。如果你还没有 GitHub 账户，快去注册一个免费账户，加入这个社区吧！

# 总结

你的博客现在已安全地存储在 Git 分布式版本控制系统中。对源代码的更改正在被跟踪，并且可以轻松撤销。你的博客也通过 Heroku 对全世界可用。现在，你可以通过 `git push` 命令来部署新功能。

# 第一部分备注

本章标志着本书第一部分的结束。我们已经覆盖了 Ruby 和 Rails 的基础知识。模型代表你应用的数据；视图是你应用的用户界面；控制器是将它们连接在一起的粘合剂。你将使用这些概念来构建任何你想要的应用。

查看你在第一部分构建的应用，你会发现有很多可以改进的地方。例如，任何人都可以编辑甚至删除你博客上的帖子。而且，如果你写了成千上万的帖子，会发生什么呢？索引页面可能会在显示所有帖子之前就超时！虽然你现在可能还没有足够的工具来解决这些问题，但一旦你深入学习第二部分，情况就会有所不同。

在本书的下一部分，我们将构建一个新的社交网络应用，并讨论更高级的话题，如更复杂的数据建模、身份验证、测试、安全性、性能优化和调试。

学习了这些概念后，你将能够解决博客中的这些问题，并构建各种其他应用。

# 练习

| 问： | 1\. 练习对你的应用进行修改，添加并提交这些更改到你的本地 Git 仓库，然后将更改推送到 Heroku。许多 Rails 开发者每天会进行多次部署，因此请熟悉这个过程。 |
| --- | --- |
| 问： | 2\. 在 GitHub 上创建一个账户，学习如何在其服务器上创建一个新的仓库，并将你的应用推送到 GitHub。GitHub 提供了一个在线帮助区域，如果你遇到任何问题，可以参考这个帮助来完成操作。另外，使用 GitHub 的 Explore 功能来查看其服务器上流行项目的仓库。 |
| 问： | 3\. 最后，看看你能否“解决你自己的痛点”。基于你的兴趣，创建一个简单的 Rails 应用。比如，创建一个你最喜欢书籍的目录，或者一个用来追踪你黑胶唱片收藏的应用。 |
