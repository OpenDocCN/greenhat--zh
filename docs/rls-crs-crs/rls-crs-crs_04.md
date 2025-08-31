# 第三章 模型

在 Rails 中，模型表示应用程序中的数据以及操作这些数据的规则。模型管理应用程序与相应数据库表之间的交互。应用程序的大部分业务逻辑也应该放在模型中。

本章介绍了活跃记录，这是 Rails 组件之一，提供模型持久化（即，将数据存储在数据库中），以及数据验证、数据库迁移和模型关联。*验证*是确保仅有效数据存储在数据库中的规则。你创建数据库*迁移*来更改数据库的架构，*关联*是你应用程序中多个模型之间的关系。

# 帖子模型

在上一章中，我们使用 Rails 脚手架生成器构建了一个简单的博客，包含用于博客帖子的模型、视图和控制器。通过在你喜欢的文本编辑器中打开文件*app/models/post.rb*，查看脚手架生成器创建的帖子模型。

```
class Post < ActiveRecord::Base
end
```

这里没有太多内容。目前，这个文件仅告诉我们`Post`类继承自`ActiveRecord::Base`。在我讲解你可以实际做什么之前，让我们从活跃记录开始讨论。

# 活跃记录

*活跃记录*是对象关系映射（ORM）模式的一种实现，Martin Fowler 在其《企业应用架构模式》（Addison-Wesley Professional, 2002）中使用相同的名称描述了这一模式。它是类与表之间、属性与列之间的自动映射。

数据库中的每个表都由应用程序中的一个类表示。该表的每一行由关联类的实例（或对象）表示，行中的每一列由该对象的一个属性表示。表 3-1 中的示例演示了这种结构。如果你能查看你的数据库，看到的就是这样的内容。

表 3-1. 帖子表

| id | 标题 | 内容 | 创建时间 | 更新时间 |
| --- | --- | --- | --- | --- |
| 1 | 你好，世界 | 欢迎来到我的博客... | ... | ... |
| 2 | 我的猫 | 最可爱的猫咪... | ... | ... |
| 3 | 太忙了 | 抱歉，我没有更新... | ... | ... |

表 3-1 包含了三个示例博客帖子。这个表由`Post`类表示。`id`为 1 的帖子可以由一个`Post`对象表示。我们把这个对象叫做`post`。

你可以通过调用对象的属性方法来访问与单个列相关的数据。例如，要查看帖子的标题，调用`post.title`。通过在对象上调用属性方法访问和更改数据库值的能力被称为*直接操作*。

# 创建、读取、更新和删除

让我们通过在 Rails 控制台中输入一些命令进一步探索 Active Record。Rails 控制台就是你在第一章中使用的 IRB，只不过加载了 Rails 应用程序的环境。

要启动 Rails 控制台，进入你的*blog*目录并输入**`bin/rails console`**。你可能会注意到控制台启动时比 IRB 稍微慢一些。在这短暂的暂停期间，应用程序的环境正在加载。

与 IRB 一样，当你完成操作时，你可以输入`exit`退出控制台。

数据库应用程序的四个主要功能是*创建*、*读取*、*更新*和*删除*，通常缩写为*CRUD*。一旦你掌握了这四个操作，你就能构建任何类型的应用程序。

Rails 使这些操作变得非常简单。在大多数情况下，你可以通过一行代码完成每个操作。现在让我们使用它们来处理我们博客上的帖子。

## 创建

我们将从向数据库中添加一些记录开始。在完成本节内容时，请在 Rails 控制台中输入这些命令。本章接下来的示例将使用这些记录。

在 Rails 中创建记录最简单的方法是使用命名恰当的`create`方法，如下所示：

```
  2.1.0 :001 > **Post.create title: "First Post"**
➊    (0.1ms) begin transaction
    SQL (0.4ms) INSERT INTO "posts" ("created_at"...
     (1.9ms) commit transaction
   => #<Post id: 1, title: "First Post", ...>
```

Rails 控制台会在执行命令时显示发送到数据库的 SQL 语句 ➊。为了简洁起见，接下来的示例中我将省略这些 SQL 语句。

`create`方法接受一组属性-值对，并将记录插入到数据库中，使用适当的值。在这种情况下，它将`title`属性设置为`"First Post"`的值。当你运行这个示例时，`id`、`created_at`和`updated_at`的值会自动为你设置。`id`列是数据库中的自增值，而`created_at`和`updated_at`是 Rails 为你设置的时间戳。由于没有为`body`列传入值，因此它被设置为 NULL。

`create`方法是一个快捷方式，用于实例化一个新的`Post`对象、分配值并将其保存到数据库。如果你不想使用快捷方式，你也可以为每个操作写单独的代码行：

```
2.1.0 :002 > **post = Post.new**
 => #<Post id: nil, title: nil, ...>
2.1.0 :003 > **post.title = "Second Post"**
 => "Second Post"
2.1.0 :004 > **post.save**
 => true
```

这次我们使用了多个命令，但就像之前一样，我们创建了一个全新的`Post`对象。现在数据库中存储了两个帖子。在这两个示例中，我们仅为帖子的`title`属性分配了值，但你可以通过相同的方式为帖子的`body`属性分配值。Rails 会自动为你分配`id`、`created_at`和`updated_at`的值。你不应该修改这些值。

## 阅读

一旦你的数据库中有了一些帖子，你可能会希望将它们读出并显示。首先，让我们使用`all`方法查看数据库中的所有帖子：

```
2.1.0 :005 > **posts = Post.all**
 => #<ActiveRecord::Relation [#<Post id: 1, ...>, #<Post id: 2, ...>]>
```

这会返回一个 Active Record *关系*，它包含一个数据库中所有帖子的数组，并将其存储在`posts`中。你可以将其他方法链式调用到这个关系上，Active Record 会将它们合并成一个查询。

Active Record 还实现了 `first` 和 `last` 方法，它们返回数组中的第一个和最后一个条目。Active Record 版本的这些方法只会返回数据库表中的第一个或最后一个记录。这比先获取表中所有记录，再在数组上调用 `first` 或 `last` 要高效得多。让我们试着从数据库中获取几篇帖子：

```
2.1.0 :006 > **Post.first**
 => #<Post id: 1, title: "First Post", ...>
2.1.0 :007 > **Post.last**
 => #<Post id: 2, title: "Second Post", ...>
```

这个例子返回的是按照 `id` 排序的第一篇和最后一篇帖子。你将在下一个章节学习如何按其他字段对记录进行排序。然而，有时你可能确切知道想要哪个记录，而它可能不是第一个或最后一个。在这种情况下，你可以使用 `find` 方法通过 `id` 获取记录。

```
2.1.0 :008 > **post = Post.find 2**
 => #<Post id: 2, title: "Second Post", ...>
```

只是不要请求 `find` 去获取一个不存在的记录。如果数据库中没有指定 `id` 的记录，Active Record 将抛出 `ActiveRecord::RecordNotFound` 异常。当你知道某个特定记录存在但不知道它的 `id` 时，可以使用 `where` 方法指定你已知的某个属性：

```
2.1.0 :009 > **post = Post.where(title: "First Post").first**
 => #<Post id: 1, title: "First Post", ...>
```

`where` 方法也返回一个关系。如果有多个记录匹配，你可以在 `where` 后链式调用 `all` 方法，告诉 Rails 按需获取所有匹配的记录。

如果你知道数据库中只有一个匹配的记录，你可以在 `where` 后链式调用 `first` 方法来获取这个特定记录，就像在之前的例子中一样。这个模式非常常见，因此 Active Record 还提供了 `find_by` 方法作为快捷方式：

```
2.1.0 :010 > **post = Post.find_by title: "First Post"**
 => #<Post id: 1, title: "First Post", ...>
```

这个方法接受一个属性-值对的哈希，并返回第一个匹配的记录。

## 更新

更新记录就像读取它到一个变量、通过直接操作改变值，然后将其保存回数据库一样简单：

```
2.1.0 :011 > **post = Post.find 2**
 => #<Post id: 2, title: "Second Post", ...>
2.1.0 :012 > **post.title = "2nd Post"**
 => "2nd Post"
2.1.0 :013 > **post.save**
 => true
```

Rails 还提供了 `update` 方法，它接受一个属性-值对的哈希，更新记录，并在一行中保存到数据库：

```
2.1.0 :014 > **post = Post.find 2**
 => #<Post id: 2, title: "2nd Post", ...>
2.1.0 :015 > **post.update title: "Second Post"**
 => true
```

`update` 方法与 `save` 方法类似，成功时返回 `true`，如果保存记录时出现问题，则返回 `false`。

## 删除

一旦你从数据库中读取了一个记录，你可以通过 `destroy` 方法将其删除。但这次不要输入这些命令，你可不想删除你之前创建的帖子！

```
2.1.0 :016 > **post = Post.find 2**
 => #<Post id: 2, title: "Second Post", ...>
2.1.0 :017 > **post.destroy**
 => #<Post id: 2, title: "Second Post", ...>
```

`destroy` 方法也可以在类上调用，通过 `id` 删除记录，这与先将记录读取到变量中的效果相同：

```
2.1.0 :018 > **Post.destroy 2**
 => #<Post id: 2, title: "Second Post", ...>
```

你还可以根据关系删除记录：

```
2.1.0 :019 > **Post.where(title: "First Post").destroy_all**
 => [#<Post id: 1, title: "First Post", ...>]
```

这个例子删除了所有标题为 `"First Post"` 的记录。然而，使用 `destroy_all` 方法时要小心。如果在没有 `where` 条件的情况下调用它，你会删除指定类的所有记录！

# 更多 Active Record 方法

如果你熟悉 SQL 或其他访问数据库记录的方法，你会知道操作数据库不仅仅是简单的 CRUD。Active Record 提供了更多数据库操作的方法，如排序、限制、计数和其他计算。

## 查询条件

除了你到目前为止看到的简单 `where` 条件，Active Record 还提供了几个方法来帮助你优化查询。`order` 方法指定返回记录的顺序；`limit` 指定返回多少条记录；`offset` 指定从列表中返回的第一条记录。

`limit` 和 `offset` 方法通常一起用于分页。例如，如果你想每页显示 10 篇博客文章，你可以这样读取第一页的文章：

```
2.1.0 :020 > **posts = Post.limit(10)**
 => #<ActiveRecord::Relation [#<Post id: 1, ...>, #<Post id: 2, ...>]>
```

要读取网站第二页的文章，你需要跳过前 10 篇文章：

```
2.1.0 :021 > **posts = Post.limit(10).offset(10)**
 => #<ActiveRecord::Relation []>
```

输入这个会返回一个空集，因为我们的数据库中只有两篇文章。当你以这种方式将 `offset` 和 `limit` 结合使用时，你可以将 `offset` 设置为 `limit` 的倍数，查看博客的不同页面。

你还可以更改关联中条目的排序方式。使用 `limit` 时，返回的记录顺序是未定义的，所以你需要指定排序方式。使用 `order` 方法，你可以为返回的记录集指定不同的排序方式：

```
2.1.0 :022 > **posts = Post.limit(10).order "created_at DESC"**
 => #<ActiveRecord::Relation [#<Post id: 2, ...>, #<Post id: 1, ...>]>
```

使用 `DESC` 告诉 `order` 返回从最新到最旧的文章。你也可以使用 `ASC` 按相反的顺序排列。如果你更愿意按标题的字母顺序查看文章，可以将 `"created_at DESC"` 替换为 `"title ASC"`。如果不指定 `ASC` 或 `DESC`，`order` 方法默认按升序排列，但我总是指定一个排序，以便明确我的意图。

## 计算

数据库还提供了对记录执行计算的方法。我们可以在 Ruby 中读取记录并执行这些操作，但内置的数据库方法通常已优化为更快并且使用更少的内存。

`count` 方法返回匹配给定条件的记录数：

```
2.1.0 :023 > **count = Post.count**
 => 2
```

如果你没有指定条件，`count` 默认会计算所有记录，如此示例所示。

`sum`、`average`、`minimum` 和 `maximum` 方法在某个字段上执行请求的功能。例如，这行代码会查找并返回最新博客文章的日期：

```
2.1.0 :024 > **date = Post.maximum :created_at**
 => 2014-03-12 04:10:08 UTC
```

你看到的最大 `created_at` 日期应该与最新博客文章的日期匹配，而不一定是示例中显示的日期。

# 迁移

*数据库迁移* 用于每次需要更改数据库结构时。当我们使用脚手架生成器创建博客文章时，它为我们生成了迁移文件，但你也可以自己创建迁移文件。随着你构建应用程序，数据库迁移将包含对数据库所做更改的完整记录。

迁移文件存储在 *db/migrate* 目录中，并以时间戳开头，表示它们的创建时间。例如，你可以通过编辑文件 *db/migrate/*_create_posts.rb* 来查看脚手架生成器创建的迁移文件。（由于你文件上的时间戳肯定与我的不同，从现在开始我将使用星号来表示文件名中的日期部分。）现在我们来看看这个文件：

```
  class CreatePosts < ActiveRecord::Migration
➊   def change
      create_table :posts do |t|
        t.string :title
        t.text :body

        t.timestamps
      end
    end
  end
```

数据库迁移实际上是 Ruby 类。当迁移运行时，调用`change`方法 ➊。在这个例子中，该方法创建一个名为`posts`的表，并包含`title`、`body`和`timestamps`字段。`timestamps`字段指的是`created_at`和`updated_at`字段。Rails 还会自动添加`id`列。

你可以使用`rake`命令将迁移作为任务运行。例如，输入`bin/rake db:migrate`来运行所有待处理的迁移，并使你的数据库保持最新。

Rails 通过在名为`schema_migrations`的数据库表中存储时间戳，跟踪哪些迁移已经运行。

如果在数据库迁移中犯了错误，可以使用`db:rollback`任务来撤销它。纠正迁移后，使用`db:migrate`重新运行它。

## 模式

除了单独的迁移文件外，Rails 还存储了你数据库的当前状态。你可以通过打开文件*db/schema.rb*来查看。忽略文件顶部的注释块，应该像这样：

```
--*snip*--
ActiveRecord::Schema.define(version: 20130523013959) do

  create_table "posts", force: true do |t|
    t.string   "title"
    t.text     "body"
    t.datetime "created_at"
    t.datetime "updated_at"
  end

end
```

每次运行数据库迁移时，此文件都会更新。你不应手动编辑它。如果你将应用程序迁移到新电脑，并且想要一次性创建一个新的空数据库，而不是通过运行单独的迁移，你可以使用`db:schema:load rake`任务来实现：

```
$ **bin/rake db:schema:load**
```

运行此命令会重置数据库结构，并在此过程中移除所有数据。

## 添加列

现在你对迁移有了更多了解，接下来让我们创建一个并运行它。当我们创建博客文章模型时，忘记了文章需要作者。通过生成一个新的迁移，向文章表添加一个字符串列：

```
$ **bin/rails g migration add_author_to_posts author:string**
```

Rails 生成器（`g`是`generate`的缩写）查看迁移的名称，在这个例子中是`add_author_to_posts`，并尝试推断你想做什么。这是约定优于配置的另一个例子：按照`add_`*`ColumnName`*`_to_`*`TableName`*的格式命名迁移，Rails 会解析这些信息并自动添加所需内容。根据名称，我们显然想要将名为`author`的列添加到文章表中。我们还指定了`author`是一个字符串，因此 Rails 拥有创建迁移所需的所有信息。

### 注意

*你可以为迁移命名任何你想要的名称，但你应该遵循约定，这样就不需要手动编辑迁移文件。*

输入**`bin/rake db:migrate`**来运行迁移并向数据库中添加`author`列。如果你仍然打开了 Rails 控制台，你需要**`exit`**并重新启动，使用**`bin/rails console`**才能使更改生效。你也可以查看*db/schema.rb*文件，以查看文章表中新添加的列。

## 在作者迁移中

你刚生成的添加列的代码很简单。编辑文件*db/migrate/*_add_author_to_posts.rb*来查看它是如何工作的。

```
class AddAuthorToPosts < ActiveRecord::Migration
  def change
    add_column :posts, :author, :string
  end
end
```

像 **_create_posts.rb**，这个迁移是一个包含 `change` 方法的类。调用 `add_column` 方法并传入表名、列名和列类型。如果你想添加多个列，你可以为每个列创建单独的迁移，或者可以多次调用这个方法。

Active Record 迁移还提供了 `rename_column` 方法用于更改列名，`remove_column` 方法用于从表中删除列，以及 `change_column` 方法用于更改列的类型或其他选项，如默认值。

# 验证

记住，模型有用于操作应用数据的规则。Active Record *验证* 是一组规则，旨在保护你的数据。添加验证规则以确保只有有效数据被写入你的数据库。

## 添加验证

让我们看一个例子。因为我们在做一个博客，所以我们应该确保所有帖子都有标题，以免读者感到困惑，我们可以通过验证规则来做到这一点。

验证在 Rails 中作为类方法实现。打开你的帖子模型（*app/models/post.rb*）并添加这一行：

```
class Post < ActiveRecord::Base
  **validates :title, :presence => true**
end
```

这会验证 `title` 字段中是否有文本。如果尝试创建没有标题的博客文章，现在应该会出现错误。

其他常见验证

除了 `:presence` 验证，Rails 还提供了各种其他验证。例如，你可以使用 `:uniqueness` 验证，确保没有两篇帖子有相同的标题。

`:length` 验证接受一个选项哈希，以确认值的长度是否正确。将这一行添加到你的帖子模型中，可以确保所有标题至少有五个字符：

```
**validates :title, :length => { :minimum => 5 }**
```

你还可以指定 `:maximum` 值来代替 `:minimum`，或者使用 `:is` 设置一个精确值。

`:exclusion` 验证确保值不属于给定的值集合。例如，添加此验证会禁止标题为 *Title* 的博客帖子：

```
**validates :title, :exclusion => { :in => [ "Title" ] }**
```

你可以将 `:exclusion` 看作是不允许的值的黑名单。Rails 还提供了 `:inclusion` 验证，用于指定一个接受的值的白名单。

## 测试数据

验证会在数据保存到数据库之前自动运行。如果尝试保存无效数据，`save` 会返回 `false`。你也可以手动测试模型，使用 `valid?` 方法：

```
2.1.0 :025 > **post = Post.new**
 => #<Post id: nil, title: nil, ...>
2.1.0 :026 > **post.valid?**
 => false
2.1.0 :027 > **post.errors.full_messages**
 => ["Title can't be blank"]
```

在这个例子中，`valid?` 方法应该返回 `false`，因为你没有为标题设置值。验证失败会将消息添加到一个名为 `errors` 的数组中，调用 `errors` 数组上的 `full_messages` 方法应该返回一个由 Active Record 根据你的验证生成的错误消息列表。

自由使用验证规则以防止无效数据进入你的数据库，但在创建这些验证时，也要考虑到用户。清楚地指出哪些值是有效的，并在提供无效数据时显示错误消息，方便用户纠正错误。

# 关联

只有最简单的应用程序才包含一个单一模型。随着你的应用程序的增长，你会需要更多的模型，随着你添加更多模型，你需要描述它们之间的关系。Active Record *关联*描述了模型之间的关系。例如，让我们为博客帖子添加评论。

帖子和评论是相关联的。每个帖子*有许多*评论，每个评论*属于*一个帖子。这种*一对多*关系是最常用的关联之一，我们将在这里探讨它。

## 生成模型

一个博客评论应该有一个作者、一个内容和一个指向帖子的引用。你可以轻松地使用这些信息生成一个模型：

```
$ **bin/rails g model Comment author:string body:text post:references**
```

### 注意

*记得在生成新模型后运行数据库迁移！*

`post:references` 选项告诉 Rails 生成器在评论数据库表中添加一个外键。在这种情况下，外键名为 `post_id`，因为它指向一个帖子。`post_id` 字段包含该评论对应帖子的 `id`。迁移创建了我们在数据库中需要的列，现在我们需要编辑模型来完成关联设置。

## 添加关联

首先，再次打开 *app/model/post.rb* 来添加评论关联。之前我提到过每个帖子有许多评论，这正是我们在这里需要的关联：

```
class Post < ActiveRecord::Base
  validates :title, :presence => true
  **has_many :comments**
end
```

Rails 使用一个叫做 `has_many` 的类方法以可读的方式创建这个关联。现在，编辑 *app/model/comment.rb*，你会看到 Rails 生成器已经自动为你添加了匹配的 `belongs_to` 语句：

```
class Comment < ActiveRecord::Base
  belongs_to :post
end
```

现在，帖子到评论的关联应该能按预期工作。如果你的 Rails 控制台在你做这些更改时仍在运行，你需要重启它才能看到效果。

## 使用关联

当你在模型中创建一个关联时，Rails 会自动为该模型定义几个方法。使用这些方法，你就不需要担心保持 `post_id` 更新了。它们会自动维护这个关系。

### has_many 方法

你在 `Post` 中看到的 `has_many :comments` 语句定义了几个方法：

+   ****`comments`****。返回一个 Active Record 关系，表示该帖子的评论数组。

+   ****`comments<`****。将现有的评论添加到该帖子中。

+   ****`comments=`****。用给定的评论数组替换该帖子的现有评论数组。

+   ****`comment_ids`****。返回与该帖子相关联的评论 ID 数组。

+   ****`comment_ids=`****。用给定的 ID 数组中的评论替换该帖子的现有评论数组。

因为 `comments` 方法返回的是一个关系，它通常与其他方法一起使用。例如，你可以使用 `post.comments.build` 创建与某个帖子相关的新评论，它会为该帖子构建一个新评论，或者使用 `post.comments.create` 创建并保存一个新评论到数据库中。每个方法都会自动为新创建的评论分配 `post_id`。此示例为你的第一个帖子创建了一个新评论。你应该能在 `post.comments` 的输出中看到新评论：

```
2.1.0 :028 > **post = Post.first**
 => #<Post id: 1, title: "First Post", ...>
2.1.0 :029 > **post.comments.create :author => "Tony", :body => "Test comment"**
 => #<Comment id: 1, author: "Tony", ...>
2.1.0 :030 > **post.comments**
 => #<ActiveRecord::Relation [#<Comment id: 1, author: "Tony", ...>]>
```

如果你想检查是否有评论与某个帖子相关联，可以使用 `comments.empty?`，如果没有评论，则返回 `true`。你也许还会发现，知道某个帖子有多少个评论是很有用的；在这种情况下，你可以使用 `comments.size`：

```
2.1.0 :031 > **post.comments.empty?**
 => false
2.1.0 :032 > **post.comments.size**
 => 1
```

当你知道某个帖子有评论与之相关时，你可以通过传递评论 ID 给 `post.comments.find` 来查找特定评论。如果找不到与该帖子关联的匹配评论，该方法将抛出 `ActiveRecord::RecordNotFound` 异常。如果你不想抛出异常，可以使用 `post.comments.where`，如果没有找到匹配的评论，该方法会返回一个空的关系。

### 属于 `belongs_to` 方法

`Comment` 模型中的 `belongs_to :post` 语句定义了五个方法。由于 `belongs_to` 是单一关联（一个评论只能属于一个帖子），因此所有这些方法的名称都是单数形式：

+   ****`post`****。返回此评论所属帖子的实例

+   ****`post=`****。将此评论分配给另一个帖子

+   ****`build_post`****。为此评论构建一个新帖子

+   ****`create_post`****。为此评论创建一个新帖子并保存到数据库

+   ****`create_post!`**** 为此评论创建一个新帖子，但如果帖子无效，则会抛出 `ActiveRecord::RecordInvalid` 异常

这些方法是 `Post` 模型中定义的方法的逆操作。当你有一个评论并希望操作其关联的帖子时，可以使用它们。例如，下面我们来获取与我们第一个评论相关的帖子：

```
2.1.0 :033 > **comment = Comment.first**
 => #<Comment id: 1, author: "Tony", ...>
2.1.0 :034 > **comment.post**
 => #<Post id: 1, title: "First Post", ...>
```

在第一个评论上调用 `post`，它也是我们目前唯一的评论，应该会返回我们的第一个帖子。这证明了关联是双向有效的。如果你数据库中仍然有多个帖子，你也可以将此评论分配给另一个帖子：

```
2.1.0 :035 > **comment.post = Post.last**
 => #<Post id: 2, title: "Second Post", ...>
2.1.0 :036 > **comment.save**
 => true
```

将评论分配给另一个帖子会更新评论的 `post_id`，但不会写入数据库。更新 `post_id` 后别忘了调用 `save`！如果你犯了这个常见错误，评论的 `post_id` 实际上是不会改变的。

# 总结

本章对 Active Record 进行了快速概览，因此在控制台中练习，直到你对这些概念感到熟悉为止。增加更多的帖子，更新现有帖子的正文，并为这些帖子创建评论。特别关注 CRUD 操作和关联方法。这些方法在所有 Rails 应用程序中都很常用。

下一章将介绍 Rails 控制器。在那里，你将看到所有这些方法在实际操作中是如何使用的，随着你一步步完成各种控制器动作。

# 练习

| 问题： | 1\. 或许我们想联系一下那些在我们博客上留言的人。生成一个新的迁移，为评论表添加一个字符串列，用来存储电子邮件地址。运行这个迁移，并使用 Rails 控制台验证你现在是否可以为评论添加电子邮件地址。 |
| --- | --- |
| 问题： | 2\. 我们需要确保用户在创建评论时实际上输入了一些文字。为评论模型中的`author`和`body`字段添加验证。 |
| 问题： | 3\. 写一个查询来确定每个帖子拥有的评论数量。你不能通过一个查询来完成，但你应该能够通过遍历帖子集合（就像它是一个数组一样）来找到答案。 |
