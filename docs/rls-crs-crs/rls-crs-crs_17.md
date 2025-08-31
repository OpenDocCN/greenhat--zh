# 第十五章。定制部署

将你的完成的应用程序投入生产并使其可供用户访问，需要做出许多选择。你可以选择各种各样的网络托管服务提供商、Rails 应用程序服务器、数据库和自动化部署系统。在 第六章中，你学习了 Heroku，一种使用 Git 进行部署的托管服务。

大多数大型公司都有一个运营团队来配置服务器并部署应用程序。但作为一名初学者 Rails 程序员，你可能没有专门的运营团队来部署你的应用程序。

在本章中，你将设置一个服务器来托管你的应用程序，配置应用程序的生产环境，将应用程序推送到 GitHub，最后使用 Capistrano 部署到服务器。

# 虚拟私人服务器

*虚拟私人服务器（VPS）* 是由网站托管服务提供商销售的一种虚拟机。一个物理服务器可以运行多个虚拟私人服务器。每个 VPS 通常被称为 *实例*。

当你购买 VPS 时，你获得了一个更大物理服务器的一部分处理能力、内存和磁盘空间。你可以完全访问服务器的这部分，包括选择操作系统的能力。因此，你可以自由安装所需的软件，并按自己喜欢的方式配置服务器。不幸的是，你也需要对服务器上的任何安装和配置错误负责。

许多不同的托管服务提供商提供 VPS 服务。通过快速的 Google 搜索可以找到数百家竞争的提供商。亚马逊 Web 服务（AWS）是创业公司和成熟企业中常见的选择。

### 注意

*本章其余部分将使用 AWS 设置服务器并部署应用程序，但说明并非特定于 AWS。如果你更愿意使用其他服务，可以创建一个运行 Ubuntu Linux 14.04 LTS 的实例，应该可以顺利跟随操作。Ubuntu Linux 14.04 LTS 是一个长期支持版本，保证支持至 2019 年 4 月。*

## 亚马逊 AWS 设置

除了是一个流行的选择外，亚马逊还为新用户提供了 AWS 免费使用层。你可以通过 *[`aws.amazon.com/free/`](http://aws.amazon.com/free/)* 阅读更多关于免费使用层的信息，看看自己是否符合条件。即使你不符合免费使用层的条件，你仍然可以以每小时几美分的价格获得一个 AWS 微型实例。

亚马逊将他们的 VPS 服务称为 *亚马逊弹性计算云（Amazon EC2）*。为了避免在这里详细介绍如何设置亚马逊账户，请参考亚马逊 EC2 文档，链接见 *[`aws.amazon.com/documentation/ec2/`](http://aws.amazon.com/documentation/ec2/)*。

点击 **User Guide** 链接，并按照从设置开始的说明进行操作。本节将指导你完成注册 AWS、在 AWS 身份与访问管理（IAM）系统中创建用户帐户、创建密钥对和创建安全组的过程。请务必存储你的 IAM 凭据和私钥 —— 你将在本章中需要它们。

然后继续进行“入门”部分。在本节中，你应该启动一个 EC2 实例，连接到你的实例，添加一个存储卷，最后清理你的实例和卷。EC2 用户指南使用了一个 Amazon Linux 机器镜像，我们不会再次使用，因此在完成本节后，请确保按照用户指南中的清理说明进行操作。

一旦你熟悉了 Amazon EC2，你就可以按照本节所述设置生产服务器。我推荐使用 Ubuntu Linux，因此以下的指令是针对 Ubuntu 的。从 EC2 管理控制台，点击 **Launch Instance** 按钮以创建一个新的服务器实例，并在“快速启动”部分选择 Ubuntu Server 14.04 LTS (PV) Amazon 机器镜像。因为这是一个 Web 服务器，你需要配置安全组以允许 HTTP 流量。点击 **Next** 按钮，直到到达步骤 6：配置安全组。现在点击 **Add Rule** 按钮，选择 **HTTP** 从类型下拉菜单中，然后点击 **Review and Launch** 按钮。最后，点击 **Launch** 按钮。

一旦实例启动，记下在 EC2 管理控制台中显示的公共 DNS 名称，然后通过终端窗口使用 SSH 连接到实例。使用以下命令，将 `your_key_file` 替换为在 EC2 用户指南的“设置”部分中创建的私钥文件的完整路径，将 `your_instance_name` 替换为实例的公共 DNS 名称：

```
$ **ssh -i** your_key_file **ubuntu@**your_instance_name
Welcome to Ubuntu 14.04 LTS...
*--snip--*
```

Ubuntu AMI 上的默认用户帐户名为 *ubuntu*。因此，这个命令会连接到你实例上的名为 ubuntu 的用户。

## Ubuntu Linux 设置

一旦你连接成功，就可以配置实例以托管 Ruby on Rails 应用程序。在 SSH 连接上执行本节中的所有命令。

Ubuntu 使用一个名为 `apt-get` 的系统来从在线仓库安装软件。你需要的第一件事是 Ruby。不幸的是，默认的仓库通常包含较旧版本的 Ruby，但你有解决办法。

### 安装 Ruby

一家名为 Brightbox 的托管公司开发人员创建了自己的 Ubuntu 仓库，提供最新版本的 Ruby，并将其公开提供。这个仓库被称为 *个人软件包存档（PPA）*。你可以通过这些命令将该仓库添加到你的实例中，并获取最新版本的 Ruby：

```
$ **sudo apt-get install python-software-properties**
Reading package lists... Done
*--snip--*
Setting up python-software-properties (0.92.36) ...
$ **sudo apt-add-repository ppa:brightbox/ruby-ng**
Next generation Ubuntu packages for Ruby ...
*--snip--*

http://brightbox.com
More info: https://launchpad.net/~brightbox/+archive/ruby-ng
Press [ENTER] to continue or ctrl-c to cancel adding it
```

当提示时按 ENTER，然后等待 `OK` 显示出来。在添加 Brightbox 仓库后，更新 `apt-get` 包列表，以便它能够找到更新版本的 Ruby 包。

```
$ **sudo apt-get update**
Ign http://us-east-1.ec2.archive.ubuntu.com trusty ...
*--snip--*
Fetched 13.7 MB in 9s (1,471 kB/s)
Reading package lists... Done
```

现在安装 Ruby 2.1 版本。以下命令将同时安装 Ruby 解释器和编译额外 gem 所需的开发头文件：

```
$ **sudo apt-get install ruby2.1 ruby2.1-dev**
Reading package lists... Done
--*snip*-
Do you want to continue? [Y/n]
```

按下 ENTER 继续。安装完成后，检查 Ruby 版本。

```
$ **ruby -v**
ruby 2.1.1p76 (2014-02-24 revision 45161) [x86_64-linux-gnu]
```

由于 Ruby 经常更新，你可能会看到比此处显示的版本号更新的版本。现在 Ruby 已安装，你需要一个 web 服务器来支持 Ruby on Rails 应用。

### 安装 Apache 和 Passenger

目前有多种 web 服务器可供选择。最流行的 web 服务器是 Apache，我们将使用它。使用以下命令安装 Apache HTTP Server 2 版本：

```
$ **sudo apt-get install apache2**
Reading package lists... Done
*--snip--*
Do you want to continue? [Y/n]
```

按下 ENTER 继续。

完成后，打开浏览器，访问你实例的公共 DNS 名称，以查看默认的 Ubuntu 网站。虽然此时还看不到你的应用，但你已经在取得进展。

Apache 是用于提供网页的优秀选择，但你需要一个应用服务器来运行你的 Ruby on Rails 应用。与 Apache 集成的一个流行应用服务器是 Phusion Passenger。

Phusion 通过自己的 `apt-get` 仓库提供 Passenger 应用服务器。与之前使用的 Brightbox 仓库不同，它不是 PPA，因此设置过程会多一些步骤。

首先，输入 `apt-key` 命令，将 Phusion 的 RSA 密钥导入到 Ubuntu 密钥服务器：

```
$ **sudo apt-key adv --keyserver keyserver.ubuntu.com \**
                   **--recv-keys 561F9B9CAC40B2F7**
Executing: gpg --ignore-time-conflict ...
*--snip--*
gpg:               imported: 1 (RSA: 1)
```

`apt-get` 程序使用这个密钥来确保你安装的软件包确实来自 Phusion。Phusion 的仓库使用加密的 HTTP 连接（HTTPS）与实例进行通信。

首先，你需要将 Phusion Passenger 仓库添加到你的实例中。输入以下命令，在你的实例上用 `nano` 编辑器打开一个新文件。（或者，如果你更喜欢使用其他命令行编辑器，可以使用其他编辑器。）

```
$ **sudo nano /etc/apt/sources.list.d/passenger.list**
```

在第一行输入 **`deb https://oss-binaries.phusionpassenger.com/apt/passenger trusty main`**，将 Phusion Passenger 仓库的地址添加到你的实例中。然后，如果你使用的是 `nano`，按 CTRL-O 然后按 ENTER 保存文件，按 CTRL-X 退出编辑器。

现在再次更新 `apt-get` 包列表：

```
$ **sudo apt-get update**
Ign http://us-east-1.ec2.archive.ubuntu.com trusty InRelease
*--snip--*
Reading package lists... Done
```

然后安装 Apache 2 Phusion Passenger 模块：

```
$ **sudo apt-get install libapache2-mod-passenger**
Reading package lists... Done
*--snip--*
Do you want to continue? [Y/n]
```

按下 ENTER 继续。安装完成后，你的实例应该已经配置好，可以提供标准网页和 Ruby on Rails 应用的服务。

安装好 web 服务器后，为你的应用创建一个目录。常规 HTML 网页的默认目录是 */var/www/html*。因为你正在部署 Ruby on Rails 应用，所以需要使用以下命令创建一个单独的目录。

```
$ **sudo mkdir /var/www/social**
$ **sudo chown ubuntu /var/www/social**
$ **sudo chgrp ubuntu /var/www/social**
```

第一个命令创建一个名为 */var/www/social* 的目录。接下来的两个命令将该目录的所有权分配给你的 ubuntu 用户和组，允许你根据需要向该目录写入文件。

现在你需要为你的应用安装并配置一个数据库。

### 安装 PostgreSQL

本章使用了 PostgreSQL 数据库，但你选择哪款数据库软件主要取决于你。MySQL 是另一个你可以考虑的流行开源选项。

使用以下命令安装 PostgreSQL：

```
$ **sudo apt-get install postgresql postgresql-contrib**
Reading package lists... Done
--*snip*-
Do you want to continue? [Y/n]
```

按 ENTER 键继续。现在数据库软件已安装，我们来添加一个用户账户并创建一些数据库。PostgreSQL 的默认用户账户名为*postgres*，所以你需要使用 `sudo -u postgres` 命令作为 `postgres` 用户执行 `createuser` 命令：

```
$ **sudo -u postgres createuser --superuser ubuntu**
```

这个命令创建了一个名为*ubuntu*的新用户，该用户具有对数据库的超级用户访问权限。该用户可以完全访问所有数据库命令。在 Ubuntu 中，PostgreSQL 配置了一个名为*ident sameuser*的身份验证系统，默认情况下，如果你的 Ubuntu 用户名与 PostgreSQL 用户名匹配，你可以无需密码直接连接。

既然你已经为自己创建了 PostgreSQL 账户，接下来添加一个数据库，看看能否成功连接：

```
$ **createdb ubuntu**
$ **psql**
psql (9.3.4)
Type "help" for help.

ubuntu=# help
You are using psql, the command-line interface to PostgreSQL.
Type: \copyright for distribution terms
      \h for help with SQL commands
      \? for help with psql commands
      \g or terminate with semicolon to execute query
      \q to quit
ubuntu=#
```

现在你的账户可以登录 PostgreSQL 并运行命令。输入**`\q`**退出。接下来，通过输入以下命令为你的社交应用程序添加一个生产数据库：

```
$ **createdb social_production**
```

你在实例上不需要输入其他 PostgreSQL 命令。既然你已经创建了生产数据库，应用程序中的迁移会创建应用程序所需的表。在部署到实例之前，你将配置应用程序以使用这个数据库。

### 安装构建工具

你的实例几乎准备就绪！不过，在你部署应用程序之前，你需要再安装一些工具。你的应用程序使用的一些 gems 需要被编译，为此你需要像 C 编译器这样的构建工具。你还需要 Git 来从代码仓库中获取代码，并且需要 PostgreSQL 的头文件来编译 PostgreSQL 数据库 gem。

幸运的是，这个单一命令应该会安装你所需的所有构建工具：

```
$ **sudo apt-get install build-essential git libpq-dev**
Reading package lists... Done
*--snip--*
Do you want to continue? [Y/n]
```

`build-essential` 包是一组常见的构建工具，许多不同类型的软件在编译时都需要它们。你已经在第六章中熟悉了 Git。`libpq-dev` 包是编译 PostgreSQL 客户端应用程序（如 pg gem）所必需的。

### 安装 Gems

最后一步设置是安装你的应用程序所需的 gems。正如你将在下一节中学习的，`bundle` 命令在你部署时会自动运行，但在连接到服务器时安装 gems 有助于验证一切是否正常工作。

Gems 在安装时通常会生成文档。在服务器上，这些文档只是占用空间并减慢安装速度。你可以通过在 *.gemrc* 文件中添加 `gem: --no-document` 来告诉 `gem` 命令不要生成文档：

```
$ **echo "gem: --no-document" >> ~/.gemrc**
```

既然你已经关闭了 gem 文档生成，现在可以安装 Rails：

```
$ **sudo gem install rails**
Fetching: thread_safe-0.3.3.gem (100%)
Successfully installed thread_safe-0.3.3
Fetching: minitest-5.3.3.gem (100%)
Successfully installed minitest-5.3.3
*--snip--*
```

因为你正在使用 PostgreSQL 数据库，所以还需要安装 pg gem。这个 gem 的部分内容是用 C 语言编写的，安装时会自动编译。

```
$ **sudo gem install pg**
Building native extensions. This could take a while...
Successfully installed pg-0.17.1
1 gem installed
```

最后，你需要一个叫做 therubyracer 的 gem。这个 gem 将 Google 的 V8 JavaScript 解释器嵌入到 Ruby 中。Rails 使用这个 gem 在服务器上编译资产。这个 gem 的部分内容也需要进行编译。

```
$ **sudo gem install therubyracer**
Building native extensions. This could take a while...
Successfully installed therubyracer-0.12.1
1 gem installed
```

配置好这些 gems 后，你的实例就可以运行 Rails 应用程序了。现在 VPS 设置已完成，让我们了解 Capistrano 以及你需要对应用程序进行的更改，以便将其部署并在生产环境中运行。

# Capistrano

Capistrano 是一个开源工具，用于自动化通过 SSH 连接在远程服务器上运行脚本和部署应用程序的过程。Capistrano 扩展了你已经使用过的 `rake` 工具。就像 `rake` 一样，Capistrano 使用一个简单的 DSL 来定义 *任务*，这些任务会根据不同的 *角色* 应用到不同的服务器上。

任务包括从 Git 仓库拉取代码、运行 `bundle install` 或通过 `rake` 运行数据库迁移等。角色是不同类型的服务器，如 Web 服务器、应用服务器或数据库服务器。目前这些服务器都在同一台服务器上，但当应用程序变得过大而无法仅依赖一台服务器时，Capistrano 使得将工作分配到多台服务器上变得更加简单。

Capistrano 还支持将应用程序部署到不同的*阶段*。Capistrano 阶段是服务器的集合，例如预发布服务器和生产服务器。这两台服务器都在生产环境中运行你的 Rails 应用程序，但预发布服务器可能仅用于测试，而生产服务器则是用户可以访问的。

## 入门

退出 VPS 上的 SSH 会话，或者在你的本地计算机上打开另一个终端窗口来设置 Capistrano。由于 Capistrano 是一个 gem，你首先需要更新应用程序的 *Gemfile*。Capistrano 已经出现在文件中，但它被注释掉了。删除 capistrano-rails gem 前面的井号，以便安装 Capistrano 和你需要的 Rails 特定任务。

在编辑 *Gemfile* 时，还需要做出适应生产环境运行的更改：

```
  *--snip--*

  # Use sqlite3 as the database for Active Record
1 gem 'sqlite3'**, group: [:development, :test]**

  *--snip--*

  # See https://github.com/sstephenson/execjs#readme...
2 **gem 'therubyracer', platforms: :ruby, group: :production**

  *--snip--*

  # Use Capistrano for deployment
3 **gem 'capistrano-rails', group: :development**

4 **# Use PostgreSQL in production**
  **gem 'pg', group: :production**

  # Use debugger
  gem 'byebug', group: [:development, :test]
```

这些更改首先指定了 SQLite gem 仅在 `development` 和测试环境中需要 ➊。接下来，therubyracer gem 在生产环境中需要用于编译资产 ➋，如上一节所述。capistrano-rails gem 仅在开发环境中需要 ➌。最后，在生产环境中，你还需要 PostgreSQL gem ➍。

现在更新你计算机上安装的 gems：

```
$ **bin/bundle install --binstubs --without production**
Fetching gem metadata from https://rubygems.org/........
Fetching additional metadata from https://rubygems.org/..
Resolving dependencies...
--*snip*--
```

--binstubs 选项告诉 bundler 还要将可执行文件安装到 *bin/* 目录中。例如，Capistrano 包含你将用来部署应用程序的 cap 命令，你将从 *bin/* 目录运行该命令。--without production 选项告诉 bundler 仅安装开发和测试环境所需的 gems。

接下来，你需要在应用程序中安装 Capistrano：

```
$ **bin/cap install**
mkdir -p config/deploy
create config/deploy.rb
create config/deploy/staging.rb
create config/deploy/production.rb
mkdir -p lib/capistrano/tasks
Capified
```

这个过程生成了你配置 Capistrano 部署应用程序所需的文件。接下来我们来详细了解这些内容。

## 配置

现在您的应用程序已经被 Capified，您可能会注意到一些新文件。第一个文件名为*Capfile*，位于应用程序的根目录。您需要对该文件进行一个小的修改：

```
  # Load DSL and Setup Up Stages
  require 'capistrano/setup'

  # Includes default deployment tasks
  require 'capistrano/deploy'

➊ **# Include all Rails tasks**
  **require 'capistrano/rails'**

*--snip--*
```

正如注释所述，新的`require`行将 Capistrano 的 Rails 特定任务包含到您的应用程序中 ➊。保存该文件后，您可以通过在终端中输入`bin/cap -T`命令来查看 Capistrano 任务列表。

接下来，您需要编辑文件*config/deploy.rb*。该文件包含所有部署阶段共享的配置，例如您的应用程序名称和 Git 仓库地址。

```
  # config valid only for Capistrano 3.1
  lock '3.2.1'

➊ **set :application, 'social'**
  **set :repo_url, 'https://github.com/**yourname**/social.git'**

  # Default branch is :master
  # ask :branch, proc { `git rev-parse --abbrev-ref HEAD`.chomp }.call

  # Default deploy_to directory is /var/www/my_app
➋ **set :deploy_to, '/var/www/social'**

  *--snip--*

  namespace :deploy do
  desc 'Restart application'
  task :restart do
    on roles(:app), in: :sequence, wait: 5 do
      # Your restart mechanism here, for example:
➌     **execute :touch, release_path.join('tmp/restart.txt')**
    end
  end

  after :publishing, :restart

  *--snip--*

end
```

首先，将您的应用程序名称设置为`social`，并指定您的 Git 仓库的 URL ➊。将`yourname`替换为您的 GitHub 用户名。接下来，将*deploy*目录设置为您在实例上创建的*/var/www/social*目录 ➋。最后，在`restart`任务中取消注释`execute`行 ➌。此行会执行`touch tmp/restart.txt`命令。部署后，此命令用于重新启动 Passenger 应用服务器。

现在共享设置已经更新，请编辑*config/deploy/production.rb*文件。该文件包含 Capistrano `production`阶段特定的设置。将该文件中的现有代码替换为以下代码：

```
   **server '**your_instance_name**',**
➊            **user: 'ubuntu', roles: %w{web app db}**
➋  **set :ssh_options, {**
        **keys: '**your_key_file
   **' }**
```

首先，Capistrano 需要您服务器的地址，以及每台服务器的用户名和角色 ➊。您的实例执行所有三个角色，用户名是`ubuntu`。将`your_instance_name`替换为服务器的公共 DNS 名称。接下来，指定连接到实例所需的 SSH 选项 ➋。Capistrano 需要私钥的路径来进行连接。将`your_key_file`替换为您的私钥文件的完整路径。

## 数据库设置

接下来，配置您的应用程序以使用您之前创建的 PostgreSQL 数据库。数据库配置位于文件*config/database.yml*中。更新`production`部分，如下所示：

```
  *--snip--*

  production:
    **adapter: postgresql**
    **encoding: unicode**
➊   **database: social_production**
    **pool: 5**
➋   **username: ubuntu**
➌   **password:**
```

这段代码告诉 Rails 在`production`环境中使用名为`social_production`的 PostgreSQL 数据库 ➊。Rails 将使用用户名`ubuntu` ➋并且没有密码 ➌，这要归功于之前提到的 Ubuntu 的 ident sameuser 身份验证设置。

## 密钥设置

您需要设置的最后一件事是用于签署应用程序 cookie 的密钥。该值存储在文件*config/secrets.yml*中。此文件还可以用于存储其他秘密信息，如应用程序所需的密码或 API 密钥。

```
  *--snip--*
  development:
    secret_key_base: 242ba1d...

  test:
    secret_key_base: 92d581d...

➊ # Do not keep production secrets in the repository,
  # instead read values from the environment.
  production:
➋   secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>
```

如注释中所述，您不应将生产环境的密钥保存在此文件中 ➊。如果您的应用程序代码存储在公共 Git 仓库中，那么这些密钥将变得公开可用。相反，该文件使用 ERB 标签读取`SECRET_KEY_BASE`环境变量的值 ➋。

在您可以在服务器上设置此环境变量之前，请使用以下命令生成一个值：

```
$ **bin/rake secret**
a3467dbd655679241a41d44b8245...
```

复制此命令输出的值，并将其保存在安全的地方。稍后在本章设置应用的虚拟主机时你还会用到它。

## 添加到 Git

配置好 Capistrano 并配置好数据库后，你就可以为你的应用创建 Git 仓库并将代码推送到 GitHub 了。Capistrano 会在你的实例上运行 `git` 命令，在部署过程中从 GitHub 拉取你应用的更改。

首先在你的本地计算机上使用以下命令创建 Git 仓库。如果需要复习 Git，可以参考 第六章。

```
$ **git init**
Initialized empty Git repository in ...
$ **git add .**
$ **git commit -m "Initial commit"**
[master (root-commit) 1928798] Initial commit
123 files changed, 1826 insertions(+)
*--snip--*
```

现在登录到你的 GitHub 账户，并创建一个名为 *social* 的新公共仓库。创建仓库后，向你刚创建的本地仓库添加一个远程仓库，并将代码推送到 GitHub。

```
$ **git remote add origin https://github.com/**yourname**/social.git**
$ **git push -u origin master**
Counting objects: 141, done.
--*snip*--
Branch master set up to track remote branch master from origin.
```

一旦 Capistrano 配置完成且你的应用已上传至 GitHub，你就可以进行部署了。

## 部署

首先，测试与实例的连接，并检查实例是否准备好接收来自 Capistrano 的部署。`deploy:check` 任务确保实例上的所有设置正确：

```
$ **bin/cap production deploy:check**
 INFO [722a06ac] Running /usr/bin/env ...
*--snip--*
 INFO [5d3c6d3e] Finished ... exit status 0 (successful).
```

请注意，我在命令中指定了 `production` 阶段。每次执行 Capistrano 命令时都必须包括阶段。

如果 `deploy:check` 任务成功完成，你就可以第一次部署你的应用了：

```
$ **bin/cap production deploy**
 INFO [e6d54911] Running /usr/bin/env ...
*--snip--*
 INFO [3cb59e26] Finished ... exit status 0 (successful).
```

`deploy` 任务不仅会从 GitHub 检出最新的代码，还会运行 `bundle install` 更新已安装的 gems，编译应用的资产，并迁移数据库。然而，即使你的应用已经安装并在实例上运行，你仍然需要进行最后一次配置修改，才能在互联网上访问到你的应用。

## 添加虚拟主机

*虚拟主机* 是一种在同一服务器或实例上托管多个站点的方式。Apache Web 服务器允许你在同一物理服务器上设置多个不同的站点。它根据每个站点的 DNS 名称，来为每个传入请求提供正确的站点。你当前的实例上只运行了一个站点，但你仍然需要将其设置为虚拟主机。

这一步只需执行一次。除非你决定在同一服务器上添加另一个站点，否则以后无需再次执行此步骤。由于你接下来要指定的目录名称在之前并不存在，所以你需要等到应用部署完成后再进行。

首先，使用 SSH 连接到实例，然后在 */etc/apache2/sites-available* 目录中为社交应用创建配置文件：

```
$ **sudo nano /etc/apache2/sites-available/social.conf**
```

上述命令会在 `nano` 编辑器中打开新文件。在新文件中输入以下 Apache 配置代码：

```
➊ **<VirtualHost *:80>**
➋   **ServerName** ***your_instance_name***
➌   **DocumentRoot /var/www/social/current/public**
➍   **SetEnv SECRET_KEY_BASE** ***a3467dbd65...***
➎   **<Directory /var/www/social/current/public>**
      **Allow from all**
      **Options -MultiViews**
    **</Directory>**
  **</VirtualHost>**
```

第一行表示此虚拟主机响应所有请求（通过星号表示）并监听 80 端口 ➊。接下来，指定此虚拟主机的服务器名称 ➋。将 ***`your_instance_name`*** 替换为你实例的公共 DNS 名称。

然后为这个虚拟主机设置文档根目录 ➌。文档根目录通常是网站的 HTML 文件所在的位置，但在这里，你将其设置为你的应用程序的公共目录。此配置特定于 Passenger 应用程序服务器。

下一行设置了`SECRET_KEY_BASE`环境变量 ➍。将此处显示的部分密钥替换为你之前输入的`bin/rake secret`命令生成的完整 128 位密钥。

最后，为文档根目录设置选项 ➎。`Allow from all`这一行意味着所有主机和 IP 地址都可以访问此目录中的文件。`Options -MultiViews`这一行关闭了 Apache 中的 MultiViews 功能。该功能使用自动内容协商，可能会导致 Apache 向客户端提供文件，即使文件扩展名未指定，这是你不希望发生的。

按 CTRL-O 然后按 ENTER 保存文件，再按 CTRL-X 退出编辑器。

现在，新的站点已经在 Apache 中配置完成，你需要禁用 Apache 自带的默认站点并启用社交网站：

```
$ **sudo a2dissite 000-default**
Site 000-default disabled.
To activate the new configuration, you need to run:
  service apache2 reload
$ **sudo a2ensite social**
Enabling site social.
To activate the new configuration, you need to run:
  service apache2 reload
```

完成此操作后，重新加载 Apache 以激活更改：

```
$ **sudo service apache2 reload**
 * Reloading web server apache2
 *
```

现在打开你的网络浏览器，访问你实例的公共 DNS 名称。你的应用程序应该可以在互联网上访问，并在你自己的虚拟私人服务器上以生产模式运行。

# 总结

在本章中，你学习了如何为托管 Rails 应用程序设置 Linux 服务器。你安装并配置了 Apache Web 服务器、Phusion Passenger 应用程序服务器和 PostgreSQL 数据库服务器。

你还学习了如何将远程服务器自动化工具 Capistrano 集成到你的 Rails 应用程序中。你为生产环境配置了 Rails 应用程序，并使用 Capistrano 将其部署到你的实例中。

完成这些后，你已经在成为一名专业的 Rails 开发者的道路上迈出了重要的一步！

# 练习

| Q: | 1\. 对你的应用程序进行一些小改动，例如更新每个页面的标题。将更改提交到本地 Git 仓库，推送更改到 GitHub，然后将更改部署到你的实例。 |
| --- | --- |
| Q: | 2\. 了解其他可以用来轻松为你的 Rails 应用程序添加功能的 gem。例如，你可能希望允许用户将图片上传到你的网站，而不是使用第三方图片托管服务。数百个开源项目可以为你的应用程序添加此类功能。找到一个你喜欢的并试试。如果你发现 bug，修复它并向开发者发送 GitHub 上的 pull request。 |
| Q: | 3\. 了解 Ruby on Rails 社区并参与其中。在 GitHub 上关注 Rails 开发。查看官方 Ruby on Rails 网站和博客。了解 Ruby 和 Rails 的会议，并尝试参加；在你当地的 Ruby 或 Rails 用户小组中让自己出名。 |
