# 附录 A. 解决方案

# 第一章

1.  练习 1 是学习如何读取文件，并使用文件内容探索数组方法。我期望在完成练习后，控制台会显示类似以下内容：

    ```
    irb(main):001:0> **file = File.read("test.txt")**
     => "Call me Ishmael..."
    irb(main):002:0> **puts file.split**
    Call
    me
    Ishmael
    --*snip*--
     => nil
    irb(main):003:0> **puts file.split.length**
     => 198
    irb(main):004:0> **puts file.split.uniq.length**
     => 140
    ```

    输出取决于你使用的文本。

1.  第二个练习需要写一些代码。以下示例仅使用到目前为止介绍的方法来解决问题：

    ```
    file = File.read("test.txt")
    counts = {}
    file.split.each do |word|
      if counts[word]
        counts[word] = counts[word] + 1
      else
        counts[word] = 1
      end
    end
    puts counts
    ```

    这个解决方案应该打印类似以下内容：

    ```
    => {"Call"=>1, "me"=>3, "Ishmael."=>1, ...
    ```

    词语 “Call” 在段落中出现了一次；词语 “me” 出现了三次；以此类推。

1.  使用练习 3 中提供的示例代码，完整的解决方案如下所示：

    ```
    class WordCounter
      def initialize(file_name)
        @file = File.read(file_name)
      end

      def count
        @file.split.length
      end

      def uniq_count
        @file.split.uniq.length
      end

      def frequency
        counts = {}
        @file.split.each do |w|
          if counts[w]
            counts[w] = counts[w] + 1
          else
            counts[w] = 1
          end
        end
      end
    end
    ```

这将前两个练习的解决方案结合起来，并将它们封装在一个 Ruby 类中。

# 第二章

1.  第一个练习是熟悉一个简单的 Rails 应用程序以及默认提供的功能。主页的地址是*http://localhost:3000/posts*。随着你在应用中移动，这个地址会发生变化。新帖子的表单在 */posts/new*；第一篇帖子在 */posts/1*；编辑第一篇帖子的表单在 */posts/1/edit*。这些路径及其含义在第四章中讲解。

1.  如果你以前从未在大型应用程序上工作过，那么典型的 Rails 应用程序中的文件数量可能会让你感到畏惧。大多数编辑器都包含某种类型的项目列表用于打开文件，并且提供快捷键来快速通过文件名搜索文件。这些功能在处理较大项目时非常宝贵。

# 第三章

1.  以下命令生成并运行迁移，以向评论添加电子邮件地址：

    ```
    $ **bin/rails g migration add_email_to_comments email:string**
        invoke  active_record
        create    db/migrate/20140404225418_add_email_to_comments.rb
    $ **bin/rake db:migrate**
    == 20140404225418 AddEmailToComments: migrating...
    --*snip*--
    ```

    然后你可以启动一个 Rails 控制台，使用 `bin/rails console` 并创建一个带有电子邮件地址的新评论。

1.  打开*app/models/comment.rb*并添加如下所示的验证：

    ```
    class Comment < ActiveRecord::Base
      belongs_to :post
      **validates :author, :body, presence: true**
    end
    ```

    注意，我将两个字段的验证合并在一行中。你也可以通过对 `validates` 方法进行两次调用来完成此操作。

1.  你不能写一个查询来确定每个帖子的评论数量，但你可以遍历所有帖子并计算评论数量。在 Rails 控制台输入类似以下内容：

    ```
    2.1.0 :001 > **Post.all.each do |post|**
    2.1.0 :002 *   **puts post.comments.count**
    2.1.0 :003 > **end**
    ```

这段代码首先找到所有的帖子，然后对每个帖子在评论表上执行一个计数查询。

# 第四章

1.  打开文件*app/controllers/comments_controller.rb*，找到 `create` 方法。

    ```
    class CommentsController < ApplicationController
      def create
        @post = Post.find(params[:post_id])

        if @post.comments.create(comment_params) ➊
          redirect_to @post,
                      notice: 'Comment was successfully created.'
        else
          redirect_to @post,
                      alert: 'Error creating comment.'
        end
      end
    --*snip*--
    ```

    注意，目前它使用 @post.comments.create(comment_params) ➊ 来初始化并保存新评论，作为 if 语句的一部分。你需要将新评论存储在一个变量中，这样当保存失败时，你可以使用 errors 方法获取错误列表。根据下面的示例更新 create 方法：

    ```
    class CommentsController < ApplicationController
      def create
        @post = Post.find(params[:post_id])
        **@comment = @post.comments.build(comment_params)**

        if **@comment.save**
          redirect_to @post,
                      notice: 'Comment was successfully created.'
        else
          redirect_to @post,
                      alert: 'Error creating comment. ' **+**
                        **@comment.errors.full_messages.to_sentence** ➊
        end
      end
    --*snip*--
    ```

    这段代码将错误添加到现有的警告中。注意，我使用了 `to_sentence` 方法 ➊ 将错误消息的数组转换为类似这样的句子：“Author can’t be blank 和 Body can’t be blank”。

1.  编辑*app/controllers/comments_controller.rb*，找到 comment_params 方法。将 :email 添加到对 permit 方法的调用中：

    ```
    class CommentsController < ApplicationController
    --*snip*--

      private
      def comment_params
        params.require(:comment).permit(:author, :body**, :email**)
      end
    end
    ```

现在，如果用户在添加新评论时输入电子邮件地址，地址应该被存储到数据库中。如果没有这个更改，`email`字段将被忽略。

# 第五章

1.  从*app/views/posts/index.html.erb*中删除`h1`元素，并更新*app/views/layouts/application.html.erb*，如这里所示：

    ```
    --*snip*--
    <body>
    **<h1>Listing posts</h1>**

    <%= yield %>

    </body>
    </html>
    ```

    还需要将*app/views/posts/new.html.erb*和*app/views/posts/edit.html.erb*中的标题更改为`h2`标题：

    ```
    **<h2>New post</h2>**

    <%= render 'form' %>

    <%= link_to 'Back', posts_path %>
    ```

1.  首先，在*app/views/posts/_form.html.erb*部分中添加`：author`的标签和文本字段：

    ```
    --*snip*--
      <div class="field">
        <%= f.label :title %><br>
        <%= f.text_field :title %>
      </div>
      **<div class="field">**
        **<%= f.label :author %><br>**
        **<%= f.text_field :author %>**
      **</div>**
      <div class="field">
        <%= f.label :body %><br>
        <%= f.text_area :body %>
      </div>
    --*snip*--
    ```

    然后，在*app/controllers/posts_controller.rb*底部的`post_params`方法中，将`：author`添加到允许的参数列表中：

    ```
    --*snip*--
        def post_params
          params.require(:post).permit(:title, **:author,** :body)
        end
    end
    ```

1.  按照问题中描述的内容，修改*config/routes.rb*和*app/views/comments/_comment.html.erb*。这是我在*app/controllers/comments_controller.rb*中编写`destroy`操作的方法：

    ```
    --*snip*--
      **def destroy**
        **@post = Post.find(params[:post_id])**
        **@comment = @post.comments.find(params[:id])**

        **@comment.destroy**
        **respond_to do |format|**
          **format.html { redirect_to @post }**
          **format.json { head :no_content }**
        **end**
      **end**
    --*snip*--
    ```

# 第六章

1.  在应用程序中编辑文件后，使用**`git add .`**暂存你的更改，然后使用**`git commit -m "`** ***`提交信息"`***提交这些更改，最后使用**`git push heroku master`**将更改推送到 Heroku。

1.  如果你还没有 GitHub 账号，访问* [`github.com/`](https://github.com/)*并填写注册表单。接下来，你需要选择一个计划。免费计划包括无限的公共仓库。一旦完成注册过程，你应该会看到 GitHub Bootcamp 屏幕。按照屏幕上的说明创建一个仓库并上传你的应用程序。

1.  在第二章中创建你新的应用程序，而不是在*blog*目录内。使用`rails new`命令，后面跟上你新应用程序的名称。例如，要创建一个跟踪你的唱片收藏的应用程序，输入以下命令：

    ```
    $ **rails new vinyl**
    ```

    接下来，考虑一下应用程序所需的模型。在这种情况下，你可能需要一个`Record`或`Album`模型。模型需要如`title`、`artist`和`release_date`等字段。进入*vinyl*目录，并使用`rails scaffold`命令生成一些代码以开始：

    ```
    $ **cd vinyl**
    $ **bin/rails generate scaffold Album title artist release_date:datetime**
    ```

现在启动 Rails 服务器，并开始使用你的新应用程序。

# 第七章

1.  在我的 Rails 版本中，`Post`类有 58 个祖先。

    ```
    irb(main):001:0> Post.ancestors.count
    => 58
    ```

    使用 Ruby 的漂亮打印方法（`pp`），你可以将每个祖先列出在单独的行中：

    ```
    irb(main):012:0> pp Post.ancestors
    [Post(id: integer, title: string, body: text, created_at: datetime,
    updated_at: datetime, author: string),
     Post::GeneratedFeatureMethods,
     #<Module:0x007fabc21bafd8>,
     ActiveRecord::Base,
     --*snip*--
     ActiveRecord::Validations,
     --*snip*--
     Kernel,
     BasicObject]
    ```

    当你浏览祖先列表时，应该会看到一些你熟悉的名字，比如`ActiveRecord::Associations`和`ActiveRecord::Validations`。同时，注意到`Post`类继承自`BasicObject`，就像 Ruby 中的其他所有类一样。

1.  `cannot_`*`feature`*`!`方法应该与`can_`*`feature`*`!`方法相同，唯一的区别是它将`false`赋值给`@features[f]`，而不是`true`。

    ```
    class User
      FEATURES = ['create', 'update', 'delete']

      FEATURES.each do |f|
        define_method "can_#{f}!" do
          @features[f] = true
        end

        **define_method "cannot_#{f}!" do**
          **@features[f] = false**
        **end**

        define_method "can_#{f}?" do
          !!@features[f]
        end
      end

      def initialize
        @features = {}
      end
    end
    ```

    添加这个方法后，创建另一个`User`类的实例，并确保新方法按预期工作：

    ```
    irb(main):001:0> **user = User.new**
     => #<User:0x007fc01b95abe0 @features={}>
    irb(main):002:0> **user.can_create!**
     => true
    irb(main):003:0> **user.can_create?**
     => true
    irb(main):004:0> **user.cannot_create!**
     => false
    irb(main):005:0> **user.can_create?**
     => false
    ```

1.  首先，查看`Element`类定义的实例方法：

    ```
    irb(main):001:0> **Element.instance_methods(false)**
     => [:name, :name=]
    ```

    `name`和`name=`方法如预期所定义。现在重新打开`Element`类并添加对`accessor :symbol:`的调用。

    ```
    irb(main):002:0> **class Element**
    irb(main):003:1> **accessor :symbol**
    irb(main):004:1> **end**
     => :symbol=
    ```

    这应该创建两个新方法，分别命名为`symbol`和`symbol=`。你可以通过再次调用`instance_methods`来验证方法是否已创建：

    ```
    irb(main):005:0> **Element.instance_methods(false)**
     => [:name, :name=, :symbol, :symbol=]
    ```

你可以通过创建`Element`类的实例并使用`e.symbol = "Au"`来验证方法是否按预期工作。

# 第八章

1.  在`belongs_to`关联的一方指定`dependent: :destroy`会导致父模型在任何子模型被销毁时一并销毁。在这个例子中，销毁任何`Post`也会销毁关联的`User`。这个错误比较常见。

1.  完整的`Comment`模型应该是这样的：

    ```
    class Comment < ActiveRecord::Base
      belongs_to :post
      belongs_to :user

      **validates :post_id, presence: true**
      **validates :user_id, presence: true**
    end
    ```

    Rails 生成器会自动添加`belongs_to`关联，但不会添加验证。

1.  使用**`bin/rails console`**启动 Rails 控制台。创建一个新的`User`、`TextPost`和`Comment`。验证所有模型是否已创建。然后对新创建的`User`调用`destroy`，并验证关联的`TextPost`和`Comment`记录是否也被销毁。

    ```
    irb(main):001:0> **carol = User.create name: "Carol"**
     => #<User id: 3, name: "Carol", ...>
    irb(main):002:0> **post = TextPost.create user: carol, body: "Testing"**
     => #<TextPost id: 3, body: "Testing", ...>
    irb(main):003:0> **comment = Comment.create post: post, user: carol, \**
                                    **body: "Hello"**
     => #<Comment id: 1, body: "Hello", ...>
    irb(main):004:0> **carol.posts.count**
     => 1
    irb(main):005:0> **carol.comments.count**
     => 1
    irb(main):006:0> **carol.destroy** ➊
    --*snip*--
     => #<User id: 3, name: "Carol", ...>
    irb(main):007:0> **carol.posts.count**
     => 0
    irb(main):008:0> **carol.comments.count**
     => 0
    irb(main):009:0> **carol.reload** ➋
    ActiveRecord::RecordNotFound: Couldn't find User with id=3
    --*snip*--
    ```

注意，调用`destroy`方法并不会从内存中删除模型➊。即使模型已经从数据库中删除，变量`carol`仍然引用该模型。尝试从数据库重新加载模型时，会抛出`ActiveRecord::RecordNotFound`异常，因为 carol 的记录已被删除➋。

# 第九章

1.  首先，编辑位于*app/views/text_posts/_text_post.html.erb*的文本帖子部分， 如下所示：

    ```
    <div class="panel panel-default">
      <div class="panel-heading">
        <h3 class="panel-title">
          <%= text_post.title %>
        </h3>
        **<%= link_to(**
              **"#{time_ago_in_words text_post.created_at} ago",**
              **post_path(text_post)) %>**
      </div>
    --*snip*--
    ```

    这会创建一个指向 text_post 的链接，链接中显示类似“5 天前”之类的时间。按照类似的修改方式编辑位于*app/views/image_posts/_image_post.html.erb*的图片帖子部分。

    ```
    --*snip*--
      </h3>
      **<%= link_to "#{time_ago_in_words image_post.created_at} ago",**
            **post_path(image_post) %>**
    </div>
    --*snip*--
    ```

    唯一的区别在于单词 text_post 被替换为 image_post。现在加载帖子索引页面，确保链接正常工作。

1.  这个练习最重要的部分是限制控制器访问仅限认证用户。在*app/controllers/comments_controller.rb*中添加`before_action :authenticate_user!`，如下所示：

    ```
    class CommentsController < ApplicationController
      **before_action :authenticate_user!**

      --*snip*--
    end
    ```

    位于*app/views/comments/_comment.html.erb*的评论部分展示了创建评论的用户的`name`和评论的`body`。

    ```
    **<p><em><%= comment.user.name %> said:</em></p>**
    **<p><%= comment.body %></p>**
    ```

    这个部分模板在`post`的`show`视图中通过`render @post.comments`为每个评论渲染一次。

1.  首先，使用**`bin/rails console`**启动 Rails 控制台，查看用户的**`password_digest`**。

    ```
    irb(main):001:0> **alice = User.find 1**
      User Load ...
     => #<User id: 1, name: "Alice", ...>
    irb(main):002:0> **alice.password_digest**
     => "$2a$10$NBjrpHtfLJN14c6kVjG7sety1N4ifyuto7GD5qX7xHdVmbtweL1Ny"
    ```

你看到的`alice.password_digest`的值会有所不同。Bcrypt 在生成哈希摘要之前会自动为密码添加盐。通过查看该值，我无法知道`alice`的密码。Bcrypt 看起来相当安全！

你可以通过查看浏览器开发者工具或页面信息中的资源来看到网站的 cookie。根据 Chrome 开发者工具，我当前的`_social_session` cookie 是 465 字节的字母数字字符串，类似于这个`"M2xkVmNTaGpVaFd..."`。不过，我无法解读这些信息。

# 第十章

1.  打开*app/views/text_posts/_text_post.html.erb*中的`TextPost`部分。它已经显示了用户的`name`。在`text_post.user.name`之前添加对`link_to`帮助方法的调用，并将`text_post.user`传递给该帮助方法：

    ```
    --*snip*--
    <div class="panel-body">
      <p><em>By **<%= link_to text_post.user.name, text_post.user %>**</em></p>

      <%= text_post.body %>
    </div>
    --*snip*--
    ```

    然后更新*app/views/image_posts/_image_post.html.erb*中的`ImagePost`部分：

    ```
    --*snip*--
    <div class="panel-body">
      <p><em>By **<%= link_to image_post.user.name, image_post.user %>**</em></p>

      <%= image_tag image_post.url, class: "img-responsive" %>

      <%= image_post.body %>
    </div>
    --*snip*--
    ```

    最后，更新*app/views/layouts/application.html.erb*中的应用程序布局：

    ```
    --*snip*--
    <div class="pull-right">
      <% if current_user %>
        **<%= link_to 'Profile', current_user %>**
        <%= link_to 'Log Out', logout_path %>
      <% else %>
    --*snip*--
    ```

    应用程序布局中已经有对`current_user`的检查。将*Profile*链接放在这个条件语句内。

1.  打开`UsersController`位于*app/controllers/users_controller.rb*。在 follow 操作之前要求认证是通过使用你在第九章中编写的`authenticate_user!`方法进行的一行代码修改。

    ```
    class UsersController < ApplicationController
      **before_action :authenticate_user!, only: :follow**

    --*snip*--
    ```

    唯一的`:follow`选项意味着匿名用户仍然可以访问`show`、`new`和`create`操作。现在更新*app/views/users/show.html.erb*中的用户`show`视图。我使用了两个 if 语句，首先验证`current_user`不是 nil，然后验证`current_user`不等于或尚未关注正在显示的用户。

    ```
    --*snip*--
    <p class="lead"><%= @user.name %></p>

    **<% if current_user %>**
      **<% if current_user != @user && !current_user.following?(@user) %>**
        <%= link_to "Follow", follow_user_path(@user),
                 class: "btn btn-default" %>
      **<% end %>**
    **<% end %>**

    <h3>Posts</h3>
    --*snip*--
    ```

    你也可以通过结合所有三个条件语句来使用单个 if 语句完成此操作。

1.  首先，打开*app/controllers/image_posts_controller.rb*，并为新建和创建操作以及私有的 image_post_params 方法添加方法。这些方法类似于 TextPostsController 中的相应方法。

    ```
    class ImagePostsController < ApplicationController
      **def new**
        **@image_post = ImagePost.new**
      **end**
      **def create**
        **@image_post = current_user.image_posts.build(image_post_params)**
        **if @image_post.save**
          **redirect_to post_path(@image_post),**
                        **notice: "Post created!"**
        **else**
          **render :new, alert: "Error creating post."**
        **end**
      **end**

      **private**

      **def image_post_params**
        **params.require(:image_post).permit(:title, :url, :body)**
      **end**
    end
    ```

    接下来，在*app/views/image_posts/new.html.erb*中添加新的视图：

    ```
    <div class="page-header">
      <h1>New Image Post</h1>
    </div>

    <%= render 'form' %>
    ```

    然后在*app/views/image_posts/_form.html.erb*中添加表单部分：

    ```
    <%= form_for @image_post do |f| %>
      <div class="form-group">
        <%= f.label :title %>
        <%= f.text_field :title, class: "form-control" %>
      </div>
      <div class="form-group">
        <%= f.label :url %>
        <%= f.text_field :url, class: "form-control" %>
      </div>
      <div class="form-group">
        <%= f.label :body %>
        <%= f.text_area :body, class: "form-control" %>
      </div>

      <%= f.submit class: "btn btn-primary" %>
      <%= link_to 'Cancel', :back, class: "btn btn-default" %>
    <% end %>
    ```

    最后，在*app/views/posts/index.html.erb*的主页上添加一个按钮，链接到新的图片帖子表单：

    ```
    --*snip*--
    <p>
      <%= link_to "New Text Post", new_text_post_path,
            class: "btn btn-default" %>
      **<%= link_to "New Image Post", new_image_post_path,**
            **class: "btn btn-default" %>**
    </p>
    --*snip*--
    ```

如果你对这些操作或视图有任何问题，请回顾创建帖子。

# 第十一章

1.  首先，在*app/controllers/image_posts_controller.rb*中为`edit`和`update`操作添加方法，如下所示：

    ```
    --*snip*--

    **def edit**

      **@image_post = current_user.image_posts.find(params[:id])**
    **end**

      **def update**
        **@image_post = current_user.image_posts.find(params[:id])**
        **if @image_post.update(image_post_params)**
          **redirect_to post_path(@image_post), notice: "Post updated!"**
        **else**
          **render :edit, alert: "Error updating post."**
        **end**
      **end**

      private

      def image_post_params
        params.require(:image_post).permit(:title, :body, :url)
      end
    end
    ```

    接下来，在*app/views/image_posts/edit.html.erb*中创建`edit`视图：

    ```
    <div class="page-header">
      <h1>Edit Image Post</h1> </div>
    <%= render 'form' %>
    ```

    这个视图使用你在第十章中创建的表单部分。最后，在*app/views/image_posts/_image_post.html.erb*中的`ImagePost`部分添加指向`edit`操作的链接：

    ```
    --*snip*--
    <%= image_post.body %>
        **<% if image_post.user == current_user %>**
          **<p>**
          **<%= link_to 'Edit', edit_image_post_path(image_post),**
                **class: "btn btn-default" %>**
          **</p>**
        **<% end %>**
      </div>
    </div>
    ```

    这个链接被包装在一个条件语句中，只有当该图片帖子是当前用户创建时才会显示。

1.  更新*app/controllers/posts_controller.rb*中的`PostsController`，如问题所示。

    ```
      --*snip*--

      def show
        @post = Post.find(params[:id])
        **@can_moderate = (current_user == @post.user)**
      end
    end
    ```

    现在编辑*app/views/comments/_comment.html.erb*中的评论部分，并在`@can_moderate`实例变量为`true`时添加删除评论的链接：

    ```
    <p><em><%= comment.user.name %> said:</em></p>
    <p><%= comment.body %></p>
    <% if @can_moderate %>
      **<p>**
      **<%= link_to 'Destroy', comment_path(comment),**
            **method: :delete, class: "btn btn-default" %>**
      **</p>**
    **<% end %>**
    ```

    确保在链接中添加`method: :delete`，以便调用`destroy`操作。最后，在*app/controllers/comments_controller.rb*中添加`destroy`操作：

    ```
    --*snip*--

    **def destroy**
      **@comment = Comment.find(params[:id])**

      **if @comment.destroy**
        **redirect_to post_path(@comment.post_id),**
                    **notice: 'Comment successfully destroyed.'**
      **else**
        **redirect_to post_path(@comment.post_id),**
                    **alert: 'Error destroying comment.'**
      **end**
    **end**
      private

      def comment_params
        params.require(:comment).permit(:body, :post_id)
      end
    end
    ```

    这个方法查找评论，调用`destroy`，并重定向回帖子，显示成功或失败的消息。

1.  打开*config/routes.rb*中的路由文件，并编辑`logout`路由：

    ```
      --*snip*--
      get 'login', to: 'sessions#new', as: 'login'
      **delete** 'logout', to: 'sessions#destroy', as: 'logout'

      root 'posts#index'
    end
    ```

    编辑位于*app/views/layouts/application.html.erb*的应用布局，并向*Log Out*链接添加`method: :delete`。

    ```
    --*snip*--

    <div class="pull-right">
      <% if current_user %>
        <%= link_to 'Profile', current_user %>
        <%= link_to 'Log Out', logout_path**, method: :delete** %>
      <% else %>
    --*snip*--
    ```

现在该链接会发出 DELETE 请求以注销应用。

# 第十二章

1.  显示页面加载评论集合以进行渲染，然后在渲染评论时逐个加载每个评论的所有者。您可以通过在`PostsController`中的`show`方法里添加`includes(comments: [:user])`来预加载一个帖子的评论和所有者，位置在*app/controllers/posts_controller.rb*：

    ```
    --*snip*--

      def show
        @post = Post**.includes(comments: [:user])**.find(params[:id]) ➊
        @can_moderate = (current_user == @post.user)
      end
    end
    ```

    添加`includes(comments: [:user])`会告诉 Rails 预加载该帖子的所有评论及其关联的所有用户。

1.  打开位于*app/views/comments/_comment.html.erb*的`Comment`部分，并添加缓存块：

    ```
    <% cache [comment, @can_moderate] do %> ➊
      <p><em><%= comment.user.name %> said:</em></p>
      <p><%= comment.body %></p>
      <% if @can_moderate %>
        <p>
          <%= link_to 'Destroy', comment_path(comment),
                method: :delete, class: "btn btn-default" %>
        </p>
      <% end %>
    **<% end %>**
    ```

    将一个数组传递给`cache`方法会创建一个缓存键，该键结合了数组中的元素➊。在这种情况下，缓存键包含了评论的`id`和`updated_at`字段的值，以及`@can_moderate`的值，可能为 true 或 false。

1.  打开位于*app/views/posts/show.html.erb*的显示页面，并添加`cache`块。

    ```
    --*snip*--

    <h3>Comments</h3>
    **<% cache [@post, 'comments', @can_moderate] do %>** ➊
      <%= render @post.comments %>
    **<% end %>**

    *--snip--*
    ```

    这会创建一个缓存键，它是`@post`的缓存键、单词“comments”和`@can_moderate`的值的组合➊。现在，评论集合在从缓存中读取一次后就会显示出来。

# 第十三章

1.  您需要更新此练习中两种类型帖子的视图部分。首先，编辑文件*app/views/text_posts/_text_post.html.erb*并在底部附近添加一个`debug`调用，如下所示：

    ```
    <div class="panel panel-default">
      --*snip*--

        <%= debug text_post %>
      </div>
    </div>
    ```

    然后编辑*app/views/link_posts/_link_post.html.erb*并在底部附近添加一个`debug`调用：

    ```
    <div class="panel panel-default">
      --*snip*--

        <%= debug link_post %>
      </div>
    </div>
    ```

1.  将每个帖子的 id 和类型添加到日志的最简单方法是遍历`@posts`实例变量的内容。编辑*app/controllers/posts_controller.rb*并更新`index`动作。

    ```
    class PostsController < ApplicationController
      before_action :authenticate_user!

      def index
        user_ids = current_user.timeline_user_ids
        @posts = Post.includes(:user).where(user_id: user_ids)
                   .paginate(page: params[:page], per_page: 5)
                   .order("created_at DESC")

        **@posts.each do |post|**
          **logger.debug "Post #{post.id} is a #{post.type}"**
        **end**
      end
    --*snip*--
    ```

    现在当您刷新帖子索引页面时，应该能在日志中看到类似于“Post 5 is a TextPost”的五行记录。

1.  为了调试用户登录应用时发生的情况，您需要在*app/controllers/sessions_controller.rb*中的 create 动作里添加一个`debugger`调用：

    ```
    class SessionsController < ApplicationController
      --*snip*--

      def create
        **debugger**
        user = User.find_by(email: params[:email])
        if user && user.authenticate(params[:password])
          session[:user_id] = user.id
          redirect_to root_url, :notice => "Logged in!"
        else
          flash.now.alert = "Invalid email or password"
          render "new"
        end
      end

      --*snip*--
    ```

添加这行代码后，您可以检查发送到此动作的`params`，当前`session`的内容，以及在此动作中执行时`user`的值。

# 第十四章

1.  这个`curl`命令与您之前用来创建新帖子的命令相同，只是我将*token*替换成了`fake`。

    ```
    $ **curl -i \**
           **-d '{"text_post":{"title":"Test","body":"Hello"}}' \**
           **-H "Content-Type: application/json" \**
           **-H "Authorization: Token fake" \**
           **http://localhost:3000/api/text_posts**

    HTTP/1.1 401 Unauthorized
    --*snip*--

    HTTP Token: Access denied.
    ```

    请注意，状态码是*401 Unauthorized*，且响应体包含文本`"HTTP Token: Access denied."`

1.  文本帖子验证正文是否存在，因此使用`curl`尝试创建一个没有指定正文的文本帖子。

    ```
    $ **curl -i \**
           **-d '{"text_post":{"title":"Test"}}' \**
           **-H "Content-Type: application/json" \**
           **-H "Authorization: Token *token"* \**
           **http://localhost:3000/api/text_posts**
    HTTP/1.1 422 Unprocessable Entity
    --*snip*--

    {"errors":{"body":["can't be blank"]}}
    ```

    请注意，状态码是*422 Unprocessable Entity*，且响应体包含错误的 JSON 表示。

1.  向*app/controllers/api/posts_controller.rb*添加`show`方法：

    ```
    module Api
      class PostsController < ApplicationController
        respond_to :json

        --*snip*--

        **def show**
          **@post = Post.find(params[:id])**
          **respond_with @post**
        **end**
      end
    end
    ```

    该方法查找请求的帖子，并将其分配给`@post`实例变量，然后返回该帖子。以下`curl`命令验证此动作是否有效：

    ```
    $ **curl http://localhost:3000/api/posts/1**
    {
      "id":1,
      "title":"First Post",
      "body":"Hello, World!",
      "url":null,
      "user_id":1,
      "created_at":"2014-04-22T00:56:48.188Z",
      "updated_at":"2014-04-22T00:56:48.188Z"
    }
    ```

因为你没有为此操作创建 jbuilder 视图，所以返回的是帖子默认的 JSON 表示形式。

# 第十五章

1.  编辑文件 *app/views/layouts/application.html.erb* 以更改每个页面的标题：

    ```
    <!DOCTYPE html>
    <html>
    <head>
      <title>**My Awesome Site**</title>
      --*snip*--
    ```

    在保存此更改后，将其添加到本地 Git 仓库的暂存区，然后使用合适的 `commit` 消息提交更改。

    ```
    $ **git add .**
    $ **git commit -m "Update title"**
    ```

    现在，通过在终端中输入 **`bin/cap production deploy`** 来部署你的更改。

1.  Ruby 工具箱在 *[`www.ruby-toolbox.com/`](https://www.ruby-toolbox.com/)* 上列出了数百个宝石，你可以用来为你的应用添加功能。例如，你可以让用户向你的应用上传文件。查看 Rails 文件上传类别，找到多个选项，包括 Paperclip 和 CarrierWave。在这里，你可以访问网站，阅读文档，并查看每个项目的源代码。

1.  访问 *[`github.com/rails/rails/`](https://github.com/rails/rails/)* 参与讨论开放问题和拉取请求，并查看以前的提交记录。Ruby on Rails 也有一个页面 *[`rubyonrails.org/community/`](http://rubyonrails.org/community/)*，供那些希望在线参与的人。你可以在 *[`rubyconf.org/`](http://rubyconf.org/)* 和 *[`railsconf.com,/`](http://railsconf.com,/)* 分别了解即将举行的 Ruby 和 Rails 大会。希望在那里见到你！
