# 第十三章. 调试

我曾听说，并不是所有的开发者像你我一样完美。我们在代码中从不犯错，但有时*其他*开发者会犯错，我们需要清理这些错误。当这种情况发生时，Rails 中内建的调试功能就派上用场了。本章将介绍这些内建的调试功能，从 `debug` 辅助方法开始，它能帮助你更轻松地查看应用视图中的变量值。

在前几章中，我们花了一些时间查看了 Rails 日志。在本章中，你还将学习如何向该日志添加自己的消息。最后，使用调试器 gem，你可以在应用运行时进入应用内部，追踪那些非常棘手的 bug。

# 调试辅助方法

Rails 包含一个名为 `debug` 的视图辅助方法，你可以用它来显示 Rails 视图中可用的实例变量或方法调用的值。这个辅助方法会将输出包裹在 `<pre>` 标签中，使其更容易阅读。

比如，看看随着你在应用中移动，`current_user` 方法的输出如何变化。首先编辑 *app/views/layouts/application.html.erb* 文件，并在 `yield` 方法下方添加对 `debug` 辅助方法的调用，如下所示：

```
<!DOCTYPE html>
<html>
--*snip*--

    <%= yield %>

    **<%= debug current_user %>**
  </div>
</body>
</html>
```

现在启动 Rails 服务器（如果尚未启动），并在浏览器中访问 *http://localhost:3000/login*。你应该能看到 `debug` 辅助方法的输出，显示在“登录”按钮下方，如 图 13-1 所示。

![调试 current_user](img/httpatomoreillycomsourcenostarchimages2169108.png.jpg)

图 13-1. 调试 `current_user`

此时，输出只是三条破折号排成一行，接着是三点省略号排成另一行。`debug` 辅助方法正在使用 YAML 格式化其输出。YAML 是一种在 Rails 项目中常用的数据序列化语言。例如，Rails 数据库配置文件 (*config/database.yml*) 就是 YAML 格式的。你在 第十章 中也使用了 YAML 来定义为测试提供默认数据的夹具。

在 YAML 中，三条破折号表示文档的开始。三点省略号表示 YAML 文档的结束。换句话说，这是一个空的 YAML 文档。在登录页面，`current_user` 为 `nil`，空的 YAML 文档正是反映了这一点。

现在登录到你的应用程序，滚动到帖子索引页面的底部，看看 `current_user` 的输出是如何变化的。

```
➊ --- !ruby/object:User
➋ attributes:
    id: 1
    name: Alice
    email: alice@example.com
    created_at: 2014-02-26 ...
    updated_at: 2014-02-26 ...
    password_digest: "$2a$10$7..."
```

现在，YAML 输出稍微更完整了一些。第一行以三条破折号开始，接着是 `!ruby/object:User` ➊，它表示正在显示的对象类型。在这个例子中，对象是一个 Ruby `User` 类的对象。`attributes` ➋ 代表对象属性及其值的开始。在它下面，你会看到 `User` 模型的属性：`id`、`name`、`email`、`created_at`、`updated_at` 和 `password_digest`。

显示这些信息是监控应用程序运行状态的好方法。不幸的是，使用`debug`助手会限制你只能查看当前会话中的值，如果应用程序在浏览器窗口中没有渲染任何内容，你将无法看到任何值。在这种情况下，你可以依靠 Rails 日志来追踪错误。

# Rails 日志记录器

在本书中，我谈到了 Rails 服务器的输出。当 Rails 服务器运行时，它会显示开发日志的副本。即使服务器没有运行，你也可以在编辑器中打开文件*log/development.log*来查看该日志。

这个文件可能会很大，具体取决于你使用应用程序的频率。你可以使用`bin/rake log:clear`命令来清除应用程序的日志文件。

## 日志级别

Rails 日志记录器使用名为`:debug`、`:info`、`:warn`、`:error`、`:fatal`和`:unknown`的级别。这些级别表示日志消息的严重性。级别是由开发者在记录消息时分配的。

如果日志级别等于或高于当前环境配置的日志级别，则该消息会添加到相应的日志文件中。开发和测试环境中的默认日志级别是`:debug`及以上，而生产环境中的默认日志级别是`:info`及以上。

因为生产环境中的默认日志级别不显示`:debug`级别的消息，你可以放心地将这些调试消息留在代码中，而不必担心在应用程序部署并运行时会使日志变得杂乱。

## 日志记录

每个日志级别都有一个相应的方法用于打印消息。例如，你可以调用`logger.debug "`*`Message`*`"`来将一个`debug`级别的消息添加到日志中。

你已经看过如何使用`debug`助手在视图中显示值。Rails 的日志消息通常用于模型和控制器中。

让我们将`current_user`的值添加到日志中，并将其与浏览器中显示的内容进行比较。打开你的编辑器中的文件*app/controllers/posts_controller.rb*，并将这里显示的日志语句添加到`PostsController`中：

```
  class PostsController < ApplicationController
    before_action :authenticate_user!

    def index
➊     **logger.debug current_user**

      user_ids = current_user.timeline_user_ids
  --*snip*--
```

这一行 ➊每次调用`posts`的`index`操作时，都将`current_user`的输出添加到开发日志中。刷新浏览器中的页面，并检查终端中的日志输出：

```
  Started GET "/" for 127.0.0.1 at 2014-04-05 19:34:03 -0500
  Processing by PostsController#index as HTML
    User Load (0.1ms) SELECT "users".* FROM "users"
      WHERE "users"."id" = ? LIMIT 1 [["id", 1]]
➊ #<User:0x007fd3c94d4e10>
     (0.1ms) SELECT "users".id FROM "users" ...
  --*snip*--
    Rendered posts/index.html.erb within layouts/application (27.1ms)
  Completed 200 OK in 61ms (Views: 35.9ms | ActiveRecord: 1.7ms)
```

`logger.debug`将`current_user`方法的值转换为字符串，并将其作为`#<User:0x007fd3c94d4e10>` ➊添加到日志中。不幸的是，当像`current_user`这样的 Ruby 对象被转换为字符串时，默认的表示方式是对象的`class`后跟其`object_id`。

你要做的是`inspect`这个对象。`inspect`方法在 Rails 模型上调用时会显示属性和值。将你刚才添加到`PostsController`中的`current_user`调用更改为`current_user.inspect`，然后再次刷新浏览器中的页面。

```
 Started GET "/" for 127.0.0.1 at 2014-04-05 19:34:27 -0500
 Processing by PostsController#index as HTML
   User Load (0.1ms) SELECT "users".* FROM "users"
     WHERE "users"."id" = ? LIMIT 1 [["id", 1]]
➊ #<User id: 1, name: "User One", ...>
     (0.1ms) SELECT "users".id FROM "users" ...
  --*snip*--
    Rendered posts/index.html.erb within layouts/application (27.1ms)
  Completed 200 OK in 63ms (Views: 40.9ms | ActiveRecord: 1.7ms)
```

这个输出要好得多。`current_user` 的值显示 ➊，包含所有属性，就像在 Rails 控制台中看到的一样。Rails 日志记录器会显示你发送给它的任何字符串。有时候，我会为日志中的数据加上标签，并添加像星号这样的字符，以便数据更突出：

```
class PostsController < ApplicationController
  before_action :authenticate_user!

  def index
    **logger.debug "** current_user = "**
    logger.debug current_user.inspect

    user_ids = current_user.timeline_user_ids
--*snip*--
```

你之前可能在输出中很难找到 `current_user` 的值，但现在有了人类可读的标签，找起来就容易多了。

# 调试器

有时候，单纯查看变量的值并不足以帮助调试问题。Ruby 调试器让你能够在应用程序运行时逐步进入。你可以在调试器中看到代码的执行过程，检查变量的值，甚至修改值。

首先，编辑你的应用程序的*Gemfile*，添加调试器 gem。对于 Ruby 版本 2.0 或更高版本，应该使用 byebug gem。对于旧版本的 Ruby，则应使用 debugger gem。

```
--*snip*--

# Use debugger
**gem 'byebug', group: [:development, :test]**
```

适用于你 Ruby 版本的正确 gem 已在 *Gemfile* 底部注释掉。去掉行首的 `#` 并保存文件。生产环境不需要调试器，所以这行代码只会将其添加到开发和测试组。

由于你修改了 *Gemfile*，记得使用 `bin/bundle install` 命令来更新已安装的 gems。你还需要重新启动 Rails 服务器：

```
$ **bin/rails server**
```

现在你已经安装了调试器，接下来让我们看看它能做什么。

## 进入调试器

如果你在代码中调用 `debugger` 方法，当应用程序执行到该调用时，它会暂停并启动调试器。例如，去掉你之前在 *app/controllers/posts_controller.rb* 中 posts `index` 动作里添加的日志语句，改用调试器：

```
class PostsController < ApplicationController
  before_action :authenticate_user!

  def index
    user_ids = current_user.timeline_user_ids

    **debugger**

    @posts = Post.includes(:user).where(user_id: user_ids)
               .paginate(page: params[:page], per_page: 5)
               .order("created_at DESC")
  end
--*snip*--
```

当调用 `index` 动作时，执行会在 `debugger` 语句处暂停，调试器也会启动。刷新浏览器中的 posts 索引页面。页面应该不会加载完成。检查终端中的服务器输出，你应该能看到调试器提示符：

```
➊ .../social/app/controllers/posts_controller.rb:9
  @posts = Post.includes(:user).where(user_id: user_ids)

  [4, 13] in .../social/app/controllers/posts_controller.rb
➋    4    def index
     5      user_ids = current_user.timeline_user_ids
     6
     7      debugger
     8
  => 9   @posts = Post.includes(:user).where(user_id: user_ids)
     10                 .paginate(page: params[:page], per_page: 5)
     11                 .order("created_at DESC")
     12    end
     13
➌ (rdb:2)
```

在正常的服务器输出中，你应该能看到一行指示当前代码位置的提示 ➊。在这个例子中，执行暂停在 *app/controllers/posts_controller.rb* 文件的第 9 行。接着，输出 ➋ 会显示你在代码中的位置。你应该会看到以第 9 行为中心的 10 行代码。最后，调试器提示符 ➌ 等待你的输入。

## 调试器命令

调试器接受多种命令来操作你的应用程序代码。本节涵盖了最常用的命令。除非另有说明，每个命令都可以使用其名称的首字母缩写。

首先输入 `help` 命令：

```
(rdb:2) **help**
ruby-debug help v1.6.6
Type 'help <command-name>' for help on a specific command

Available commands:
backtrace  break    catch    condition
continue   delete   disable  display
down       edit     enable   eval
exit       finish   frame    help
info       irb      jump     kill
list       method   next     p
pp         ps       putl     quit
reload     restart  save     set
show       skip     source   start
step       thread   tmate    trace
undisplay  up       var      where

(rdb:2)
```

`help` 命令会显示所有可用的调试器命令列表。你也可以在 `help` 后面跟上其他命令的名称，以获取关于特定命令的信息。

当你进入调试器时，系统会显示当前位置周围的 10 行代码。`list` 命令会在调试器中显示接下来的 10 行代码。

```
(rdb:2) **list**
[14, 18] in /Users/tony/code/social/app/controllers/posts_controller.rb
   14    def show
   15      @post = Post.find(params[:id])
   16      @can_moderate = (current_user == @post.user)
   17    end
   18  end
(rdb:2)
```

每次你输入`list`命令时，都会显示另外 10 行代码。在这种情况下，当前文件只剩下五行代码，因此显示这五行。输入`list-`可以查看前 10 行代码，输入`list=`则可以显示当前行周围的代码：

```
(rdb:2) **list=**
[4, 13] in /Users/tony/code/social/app/controllers/posts_controller.rb
   4    def index
   5      user_ids = current_user.timeline_user_ids
   6
   7      debugger
   8
=> 9   @posts = Post.includes(:user).where(user_id: user_ids)
   10                  .paginate(page: params[:page], per_page: 5)
   11                  .order("created_at DESC")
   12    end
   13
(rdb:2)
```

现在你知道了自己在代码中的位置，你可能想查看一些变量的值。`var`命令会显示当前已定义的变量及其内容。要查看局部变量，输入`var local`命令：

```
(rdb:2) **var local**
self = #<PostsController:0x007ffbfeb21018>
user_ids = [2, 1]
(rdb:2)
```

这里仅定义了两个局部变量。变量`self`表示你当前在`PostsController`内。变量`user_ids`在之前的代码的第 5 行接收了它的内容。

使用`var instance`命令列出实例变量及其值：

```
(rdb:2) **var instance**
@_action_has_layout = true
@_action_name = "index"
@_config = {}
@_env = {"GATEWAY_INTERFACE"=>"CGI/1.1", "P...
@_headers = {"Content-Type"=>"text/html"}
@_lookup_context = #<ActionView::LookupCont...
@_prefixes = ["posts", "application"]
@_request = #<ActionDispatch::Request:0x007...
@_response = #<ActionDispatch::Response:0x0...
@_response_body = nil
@_routes = nil
@_status = 200
@current_user = #<User id: 1, name: "User O...
@marked_for_same_origin_verification = true
(rdb:2)
```

到目前为止，已经定义了相当多的实例变量。这段代码唯一设置的实例变量是`@current_user`。这个实例变量是在`ApplicationController`的`current_user`方法中定义的。其他变量是由 Rails 定义的。请注意，`@posts`尚未定义。你当前的位置在第 9 行，这一行定义了`@posts`，但这一行代码尚未执行。

`display`命令将一个变量添加到调试器中的显示列表。如果你特别关心`user_ids`的值，可以输入`display user_ids`命令将其添加到显示列表，如下所示：

```
(rdb:2) **display user_ids**
1: user_ids = [2, 1]
(rdb:2)
```

你也可以使用`display`命令（简写为`disp`）显示显示列表的内容及其值：

```
(rdb:2) **disp**
1: user_ids = [2, 1]
(rdb:2)
```

要从显示列表中移除一个变量，使用`undisplay`命令后跟列表中对应变量的编号。例如，`undisplay 1`会将`user_ids`从显示列表中移除。

使用`eval`命令可以评估你喜欢的任何 Ruby 代码，并打印其值。这个命令的简写是`p`，就像 print。例如，你可能想打印`user_ids`数组的长度或`current_user`方法的输出。

```
(rdb:2) **eval user_ids.length**
2
(rdb:2) **p current_user**
#<User id: 1, name: "User One", email: "user...
(rdb:2)
```

调试器是一个 Ruby shell，因此你也可以通过在提示符下直接输入命令来评估 Ruby 代码。甚至不需要使用`eval`命令。例如，通过在调试器提示符下输入以下语句，将`user_ids`的值设置为空数组：

```
(rdb:2) **user_ids = []**
[]
(rdb:2)
```

这将打印表达式`user_ids = []`的返回值，就像你在 Rails 控制台中输入它一样。

调试器提供了几个命令，用于在调试过程中执行应用程序的代码。最常用的命令是`next`，它执行下一行代码。`next`命令会执行下一行代码中的方法，但不会进入方法内部。

`step`命令与之类似，但它还会显示每一行在方法调用内部执行的情况。`step`命令会逐行执行你的应用程序及其依赖项的代码。你可以用它来查找 Rails 框架或应用程序中使用的其他 gem 中的错误。

当你完成了代码中的导航后，使用 `continue` 命令恢复执行并完成当前请求。如果你跟随本节内容进行操作，你可能会记得你将 `user_ids` 的值设置为空数组。当你 `continue` 执行并且帖子索引页面最终渲染时，不会显示任何帖子。因为你将 `user_ids` 设置为空数组，`@posts` 实例变量也为空，`index` 视图中的 `render @posts` 语句不会渲染任何内容。

Ruby 调试器可能不是你每天都会使用的工具，某些开发者甚至从不使用它。但如果你遇到一个非常难以发现的 bug，调试器将是无价的。

# 摘要

本章描述了几种调试技术。使用 `debug` 辅助方法在应用程序视图中显示值，或通过 `logger` 语句将数据添加到日志文件中，可以帮助你追踪大多数 bug。交互式调试器提供了对应用程序的完全控制，允许你逐步执行代码并定位那些特别难以发现的 bug。

下一章将介绍 Web 应用程序编程接口（API）。我们将讨论如何使用其他应用程序的 API 并创建你自己的 API。

# 练习

| 问题: | 1\. 使用 `debug` 辅助方法，在帖子索引页面渲染时显示每个帖子的内容。在每种类型的帖子部分内部添加一个 `debug` 调用。 |
| --- | --- |
| 问题: | 2\. 使用 `logger.debug` 在 *app/controllers/posts_controller.rb* 的 `index` 动作中，将 `@posts` 实例变量中每个帖子的 `id` 和 `type` 添加到日志中。 |
| 问题: | 3\. 练习使用调试器探索你应用程序的代码。使用调试器中的 `next` 命令查看用户登录应用程序时会发生什么。 |
