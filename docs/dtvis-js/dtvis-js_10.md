## 第十章：构建数据驱动的 Web 应用程序：第二部分

在第九章中，我们搭建了 web 应用程序的框架，并展示了每个视图将要显示的可视化内容。但是在我们的 web 应用程序完成之前，我们还有一些其他的细节需要处理。首先，我们必须使 web 应用程序与 Nike+ 服务进行通信，并处理该服务特有的一些问题。接下来，我们将着手让我们的应用程序更加易于导航。在本章中，我们将讨论以下内容：

+   如何将应用程序模型与外部 REST API 连接

+   如何在单页应用程序中支持 Web 浏览器约定

## 连接到 Nike+ 服务

尽管我们的示例应用程序依赖于 Nike+ 服务来获取数据，但我们并没有查看该服务接口的具体细节。正如我提到的，Nike+ 并没有完全遵循像 Backbone.js 这样的应用库所期望的常见 REST API 约定。但是在这方面，Nike+ 并不算非常特殊。实际上，REST API 并没有一个真正的 *标准*，许多其他服务也采取了类似于 Nike+ 的方法。幸运的是，Backbone.js 已经预见到了这种变化。正如我们将在接下来的步骤中看到的，扩展 Backbone.js 以支持 REST API 的变化并不像想象中那样困难。

### 步骤 1：授权用户

正如你可能预料到的，Nike+ 并不允许互联网的任何人获取任何用户的跑步详情。用户期望至少对这些信息保持一定的隐私。因此，在我们的应用程序能够获取任何跑步信息之前，它需要用户的许可。我们在这里不详细讨论这个过程，但其结果将是一个 `authorization_token`。这个对象是一个任意字符串，我们的应用程序必须将其包含在每个 Nike+ 请求中。如果令牌缺失或无效，Nike+ 将拒绝我们的应用程序访问数据。

到目前为止，我们一直让 Backbone.js 处理 REST API 的所有细节。接下来，我们需要修改 Backbone.js 构造 AJAX 请求的方式。幸运的是，这并不像听起来那么复杂。我们只需要在我们的 Runs 集合中添加一个 `sync()` 方法。当集合中存在 `sync()` 方法时，每当 Backbone.js 发起 AJAX 请求时，都会调用它。（如果集合中没有这个方法，Backbone.js 会调用其主要的 `Backbone.sync()` 方法。）我们将在集合中直接定义这个新方法。

```
Running.Collections.Runs = Backbone.Collection.extend({

    sync: **function**(method, collection, options) {
        *// Handle the AJAX request*
    }
```

如你所见，`sync()` 被传递了一个 `method`（如 `GET`、`POST` 等）、相关的集合以及一个包含请求选项的对象。为了将授权令牌发送到 Nike+，我们可以通过这个 `options` 对象将其作为参数添加进去。

```
sync: **function**(method, collection, options) {
    options = options || {};
    _(options).extend({
        data: { authorization_token: **this**.settings.authorization_token }
    });
    Backbone.sync(method, collection, options);
}
```

方法中的第一行确保 `options` 参数存在。如果调用者没有提供值，我们将其设置为空对象（`{}`）。接下来的语句使用来自 Underscore.js 的 `extend()` 工具方法，向 `options` 对象添加一个 `data` 属性。`data` 属性本身是一个对象，我们在其中存储授权令牌。接下来我们将看看如何做到这一点，但首先让我们完成 `sync()` 方法。一旦添加了令牌，我们的请求就是一个标准的 AJAX 请求，所以我们可以让 Backbone.js 通过调用 `Backbone.sync()` 来继续处理。

现在我们可以将注意力转向从 `sync()` 方法中获取授权令牌的 `settings` 对象。我们使用这个对象来保存与集合相关的属性，类似于模型的属性。Backbone.js 不会自动为我们创建这个对象，但我们可以轻松地自己创建。我们将在集合的 `initialize()` 方法中创建它。该方法接受两个参数：一个是集合的模型数组，另一个是任何集合选项。

```
Running.Collections.Runs = Backbone.Collection.extend({

    initialize: **function**(models, options) {
        **this**.settings = { authorization_token: "" };
        options = options || {};
        _(**this**.settings).extend(_(options)
            .pick(_(**this**.settings).keys()));
    },
```

`initialize()` 方法中的第一条语句定义了一个 `settings` 对象用于集合，并为该对象设置了默认值。由于没有合适的默认值用于授权令牌，我们将使用一个空字符串。

接下来的语句确保 `options` 对象存在。如果没有作为参数传递，我们至少会有一个空对象。

最后一条语句提取 `settings` 中的所有键，查找 `options` 对象中具有相同键的任何值，并通过使用新的键值扩展 `settings` 对象。再次利用了 Underscore.js 的一些工具：`extend()` 和 `pick()`。

当我们首次创建 Runs 集合时，可以将授权令牌作为参数传入。我们将一个空数组作为第一个参数，因为目前我们没有任何模型用于集合。这些模型将来自 Nike+。在以下的代码片段中，我们使用一个虚拟值作为授权令牌。真实的应用程序会使用 Nike 提供的代码来获取真实的值。

```
**var** runs = **new** Running.Collections.Runs([], {
    authorization_token: "authorize me"
});
```

只需少量额外的代码，我们就将授权令牌添加到了向 Nike+ 发送的 AJAX 请求中。

### 第 2 步：接受 Nike+ 响应

当我们的集合查询 Nike+ 获取用户活动列表时，Backbone.js 已准备好接收特定格式的响应。更具体地说，Backbone.js 期望响应是一个简单的模型数组。

```
[
  { "activityId": "2126456911", */* Data continues... */* },
  { "activityId": "2125290225", */* Data continues... */* },
  { "activityId": "2124784253", */* Data continues... */* },
  *// Data set continues...*
]
```

然而，实际上，Nike+ 将其响应返回为一个对象。活动数组是该对象的一个属性。

```
{
  "data": [
    { "activityId": "2126456911", */* Data continues... */* },
    { "activityId": "2125290225", */* Data continues... */* },
    { "activityId": "2124784253", */* Data continues... */* },
    *// Data set continues...*
  ],
  *// Response continues...*
}
```

为了帮助 Backbone.js 处理这个响应，我们在集合中添加了一个 `parse()` 方法。这个方法的工作是接收服务器提供的响应，并返回 Backbone.js 所期望的响应格式。

```
Running.Collections.Runs = Backbone.Collection.extend({

    parse: **function**(response) {
        **return** response.data;
    },
```

在我们的案例中，我们只是返回响应中的 `data` 属性。

### 第 3 步：分页集合

接下来我们要处理的 Nike+ API 的一个方面是分页。当我们请求某个用户的活动时，服务通常不会返回*所有*的活动。用户可能在 Nike+ 中存储了成千上万的活动，一次性返回所有活动可能会使应用程序不堪重负。这样做肯定会带来明显的延迟，因为应用程序必须等待整个响应返回才能进行处理。为了避免这个问题，Nike+ 将用户活动分为多个页面，并一次返回一页活动。我们需要调整应用程序以适应这种行为，但我们将获得更加流畅的用户体验。

我们要进行的第一个调整是在请求中。我们可以向请求中添加参数，表示我们准备接受多少个活动响应。这两个参数是 `offset` 和 `count`。`offset` 告诉 Nike+ 在响应中返回哪个活动作为第一个，而 `count` 则表示 Nike+ 应该返回多少个活动。例如，如果我们想要获取前 20 个活动，可以将 `offset` 设置为 `1`，将 `count` 设置为 `20`。然后，要获取接下来的 20 个活动，我们将 `offset` 设置为 `21`（并保持 `count` 为 `20`）。

我们像添加授权令牌一样，向请求中添加这些参数——在 `sync()` 方法中。

```
sync: **function**(method, collection, options) {
    options = options || {};
    _(options).extend({
        data: {
            authorization_token: **this**.settings.authorization_token,
            count: **this**.settings.count,
            offset: **this**.settings.offset
        }
    });
    Backbone.sync(method, collection, options);
}
```

我们还需要在初始化期间为这些设置提供默认值。

```
initialize: **function**(models, options) {
    **this**.settings = {
        authorization_token: "",
        count: 25,
        offset: 1
    };
```

这些值将返回前 25 个活动，但这只是一个开始。我们的用户可能希望查看所有的跑步记录，而不仅仅是前 25 个活动。为了获取更多的活动，我们需要向服务器发出更多的请求。一旦我们获得了前 25 个活动，就可以请求接下来的 25 个活动。当这些活动返回后，我们可以再请求 25 个。我们将继续这样做，直到达到某个合理的限制，或者服务器没有更多的活动可供返回。

首先，我们将合理的限制定义为另一个设置值。在下面的代码中，我们将使用 `10000` 作为这个限制。

```
initialize: **function**(models, options) {
    **this**.settings = {
        authorization_token: "",
        count: 25,
        offset: 1,
        max: 10000
    };
```

接下来，我们需要修改集合的 `fetch()` 方法，因为标准的 Backbone.js `fetch()` 无法处理分页。在我们实现此方法时，有三个步骤：

1.  保存 Backbone.js 在请求中使用的所有选项的副本。

1.  通过添加一个回调函数来扩展这些选项，当请求成功时调用这个回调。

1.  将控制权交给正常的 Backbone.js `fetch()` 方法处理集合。

这些步骤在以下实现中对应一行代码。最后一个步骤看起来可能有点复杂，但如果一步一步地理解，实际上是很有道理的。表达式 `Backbone.Collection.prototype.fetch` 指的是 Backbone.js 集合的正常 `fetch()` 方法。我们使用 `.call()` 来执行这个方法，这样我们可以为方法设置上下文，使其成为我们的集合。这是 `call()` 的第一个 `this` 参数。第二个参数包含 `fetch()` 的选项，这些选项就是我们在第 2 步中创建的扩展选项。

```
Running.Collections.Runs = Backbone.Collection.extend({

    fetch: **function**(options) {
        **this**.fetchoptions = options = options || {};
        _(**this**.fetchoptions).extend({ success: **this**.fetchMore });
        **return** Backbone.Collection.prototype.fetch.call(
            **this**, **this**.fetchoptions
        );
    },
```

通过为 AJAX 请求添加`success`回调，我们请求在请求完成时收到通知。实际上，我们已经指定希望调用`this.fetchMore()`函数。现在是时候编写这个函数了；它也是 Runs 集合的方法。这个函数会检查是否还有更多活动。如果有，它会像前面的代码一样，执行另一次对 Backbone.js 常规集合`fetch()`的调用。

```
fetchMore: **function**() {
    **if** (**this**.settings.offset < **this**.settings.max) {
        Backbone.Collection.prototype.fetch.call(**this**, **this**.fetchoptions);
    }
}
```

由于`fetchMore()`是通过查看设置来决定何时停止的，我们需要更新这些值。因为我们已经有了一个`parse()`方法，并且 Backbone 会在每次响应时调用这个方法，所以更新操作放在这里是很方便的。我们在`return`语句之前添加一些代码。如果服务器返回的活动数量少于我们请求的数量，那么我们已经用尽了活动列表。我们将`offset`设置为`max`，以便`fetchMore()`知道停止。否则，我们会将`offset`增加活动的数量。

```
parse: **function**(response) {
    **if** (response.data.length < **this**.settings.count) {
        **this**.settings.offset = **this**.settings.max;
    } **else** {
        **this**.settings.offset += **this**.settings.count;
    }
    **return** response.data;
}
```

到目前为止，我们编写的代码几乎完成，但它有一个问题。当 Backbone.js 获取一个集合时，它假设是在获取整个集合。因此，默认情况下，每次获取的响应都会将集合中已经存在的模型替换为响应中的模型。这种行为在第一次调用`fetch()`时没问题，但对于`fetchMore()`来说就不合适了，因为`fetchMore()`是为了将数据添加到集合中，而不是替换它。幸运的是，我们可以通过设置`remove`选项轻松地调整这种行为。

在我们的`fetch()`方法中，我们将这个选项设置为`true`，这样 Backbone.js 就会开始一个新的集合。

```
fetch: **function**(options) {
    **this**.fetchoptions = options = options || {};
    _(**this**.fetchoptions).extend({
        success: **this**.fetchMore,
        remove: **true**
     });
    **return** Backbone.Collection.prototype.fetch.call(**this**,
        **this**.fetchoptions
    );
}
```

现在，在`fetchMore()`方法中，我们可以将这个选项重置为`false`，这样 Backbone.js 就会将模型添加到集合中，而不是替换它们。

```
fetchMore: **function**() {
    **this**.fetchoptions.remove = **false**;
    **if** (**this**.settings.offset < **this**.settings.max) {
        Backbone.Collection.prototype.fetch.call(**this**, **this**.fetchoptions);
    }
}
```

`fetchMore()`方法仍然存在一个小问题。该代码引用了集合的属性（`this.fetchoptions`和`this.settings`），但是该方法将在 AJAX 请求完成时异步调用。当那时发生时，集合将不在上下文中，所以`this`不会指向集合。为了解决这个问题，我们可以在初始化时将`fetchMore()`绑定到集合。再次地，Underscore.js 的一个实用函数派上了用场。

```
initialize: **function**(models, options) {
    _.bindAll(**this**, "fetchMore");
```

在这一步的最后部分，我们可以使我们的集合对使用它的代码更友好。为了继续获取额外的页面，我们已经为`fetch()`的选项设置了`success`回调。那么，如果使用我们集合的代码有自己的回调怎么办呢？不幸的是，我们已经删除了那个回调，替换成了我们自己的回调。更好的做法是简单地暂时保留一个现有的回调函数，然后在我们完成获取整个集合之后恢复它。我们将在`fetch()`方法中首先执行这一操作。以下是该方法的完整代码：

```
fetch: **function**(options) {
    **this**.fetchoptions = options = options || {};
    **this**.fetchsuccess = options.success;
    _(**this**.fetchoptions).extend({
        success: **this**.fetchMore,
        remove: **true**
        });
    **return** Backbone.Collection.prototype.fetch.call(**this**,
        **this**.fetchoptions
    );
}
```

这是`fetchMore()`的代码：

```
fetchMore: **function**() {
    **this**.fetchoptions.remove = **false**;
    **if** (**this**.settings.offset < **this**.settings.max) {
        Backbone.Collection.prototype.fetch.call(**this**, **this**.fetchoptions);
    } **else** **if** (**this**.fetchsuccess) {
        **this**.fetchsuccess();
    }
}
```

现在，当我们用尽服务器返回的列表时，可以在`fetchMore()`中执行回调。

### 第 4 步：动态更新视图

通过分批获取运行记录集合，我们使得应用程序变得更加响应迅速。即使在等待从服务器检索其余用户运行记录时，我们也可以开始显示前 25 条运行记录的汇总数据。然而，要有效地做到这一点，我们需要对我们的汇总视图进行一些小改动。目前，视图正在监听集合的任何变化。当发生变化时，视图会从头开始重新渲染。

```
initialize: **function** () {
    **this**.listenTo(**this**.collection, "change", **this**.render);
    **return** **this**;
}
```

每次我们获取新的运行记录页面时，集合会发生变化，代码会重新渲染整个视图。这几乎肯定会让我们的用户感到烦扰，因为每次获取页面时，浏览器会暂时清空页面然后再重新填充内容。相反，我们希望只渲染新添加模型的视图，保持现有模型视图不变。为了做到这一点，我们可以监听 `"add"` 事件而不是 `"change"` 事件。当该事件触发时，我们只需要渲染该模型的视图。我们已经实现了为单个 Run 模型创建并渲染视图的代码：`renderRun()` 方法。因此，我们可以按照如下方式修改我们的汇总视图：

```
initialize: **function** () {
    **this**.listenTo(**this**.collection, "add", **this**.renderRun);
    **return** **this**;
}
```

现在，当我们的集合从服务器获取新的运行模型时，它们会被添加到集合中，触发一个 `"add"` 事件，视图会捕获该事件。然后，视图会在页面上渲染每一条运行记录。

### 步骤 5：过滤集合

尽管我们的应用程序只关注跑步，Nike+ 服务支持多种体育活动。当我们的集合从服务端获取数据时，响应将包含这些其他活动。为了避免将它们包含在我们的应用中，我们可以将它们从响应中过滤掉。

我们可以手动过滤响应，检查每项活动并移除非跑步的记录。然而，这样做需要大量的工作，而 Backbone.js 提供了一个更简便的方法。为了利用 Backbone.js，我们将首先向我们的 Run 模型添加一个 `validate()` 方法。这个方法接收潜在模型的属性和创建或修改时使用的任何选项作为参数。在我们的案例中，我们只关心属性。我们会检查确保 `activityType` 等于 `"RUN"`。

```
Running.Models.Run = Backbone.Model.extend({
    validate: **function**(attributes, options) {
        **if** (attributes.activityType.toUpperCase() !== "RUN") {
            **return** "Not a run";
        }
    },
```

从这段代码中，你可以看到 `validate()` 函数的行为。如果模型中有错误，`validate()` 会返回一个值。只要 JavaScript 将其视为 `true`，返回值的具体内容并不重要。如果没有错误，`validate()` 就不需要返回任何值。

现在我们的模型已经有了 `validate()` 方法，我们需要确保 Backbone.js 会调用它。Backbone.js 会在模型被创建或修改时自动检查 `validate()`，但通常不会验证来自服务器的响应。然而，在我们的情况下，我们确实需要验证来自服务器的响应。这就要求我们在 Runs 集合的 `fetch()` 选项中设置 `validate()` 属性。以下是包括此更改的完整 `fetch()` 方法。

```
Running.Collections.Runs = Backbone.Collection.extend({
    fetch: **function**(options) {
        **this**.fetchoptions = options = options || {};
        **this**.fetchsuccess = options.success;
        _(**this**.fetchoptions).extend({
            success: **this**.fetchMore,
            remove: **true**,
            validate: **true**
          });
        **return** Backbone.Collection.prototype.fetch.call(**this**,
          **this**.fetchoptions
        );
    },
```

现在，当 Backbone.js 接收到服务器响应时，它会将响应中的所有模型都传递给模型的`validate()`方法。任何未通过验证的模型都会从集合中移除，我们的应用也不必处理那些不是跑步记录的活动。

### 第 6 步：解析响应

既然我们正在为 Run 模型添加代码，那么还有另一个改动会让 Backbone.js 感到高兴。Backbone.js 要求模型具有一个属性，使每个对象具有唯一性；它可以利用这个标识符来区分不同的跑步记录。默认情况下，Backbone.js 期望该属性为`id`，因为这是一个常见的约定。然而，Nike+的跑步记录没有`id`属性。相反，该服务使用`activityId`属性。我们可以通过在模型中添加一个额外的属性，告诉 Backbone.js 这一点。

```
Running.Models.Run = Backbone.Model.extend({
    idAttribute: "activityId",
```

这个属性让 Backbone.js 知道，对于我们的跑步记录，`activityId`属性是唯一的标识符。

### 第 7 步：检索详情

到目前为止，我们一直依赖集合的`fetch()`方法来获取运行数据。该方法从服务器获取一组跑步记录。然而，当 Nike+返回活动列表时，它并不包含每个活动的完整细节。它返回的是摘要信息，但省略了详细的度量数组和任何 GPS 数据。获取这些详细信息需要额外的请求，因此我们需要对我们的 Backbone.js 应用做出一个小的调整。

我们将首先请求作为 Charts 视图基础的详细度量数据。当 Runs 集合从服务器获取跑步记录列表时，每个 Run 模型最初会有一个空的`metrics`数组。为了获取该数组的详细信息，我们必须再次向服务器发送请求，并在请求的 URL 中包含活动标识符。例如，如果获取跑步记录列表的 URL 是*[`api.nike.com/v1/me/sport/activities/`](https://api.nike.com/v1/me/sport/activities/)*，那么获取特定跑步记录的详细信息，包括其度量数据的 URL 是*[`api.nike.com/v1/me/sport/activities/2126456911/`](https://api.nike.com/v1/me/sport/activities/2126456911/)*。这个 URL 末尾的数字*2126456911*就是该跑步记录的`activityId`。

多亏了我们在本节之前所做的步骤，在 Backbone.js 中获取这些细节变得非常容易。我们所要做的就是`fetch()`模型。

```
run.fetch();
```

Backbone.js 知道 URL 的根路径，因为我们在 Runs 集合中设置了它（而且我们的模型是该集合的一部分）。Backbone.js 也知道每个跑步记录的唯一标识符是`activityId`，因为我们在前一步中设置了该属性。而且，幸运的是，Backbone.js 足够聪明，能够将这些信息结合起来并发起请求。

然而，我们在某些方面必须帮助 Backbone.js。Nike+应用对所有请求都需要一个授权令牌，到目前为止，我们只在集合中为该令牌添加了代码。我们还需要将相同的代码添加到模型中。此代码几乎与本节第一步中的代码相同：

```
   Running.Models.Run = Backbone.Model.extend({
       sync: **function**(method, model, options) {
           options = options || {};
           _(options).extend({
               data: {
                   authorization_token:
➊                     **this**.collection.settings.authorization_token
               }
           });
           Backbone.sync(method, model, options);
       },
```

我们首先确保`options`对象存在，然后通过添加授权令牌来扩展它。最后，我们调用常规的 Backbone.js `sync()`方法。在 ➊ 处，我们直接从集合中获取令牌的值。我们可以在这里使用`this.collection`，因为 Backbone.js 会将模型的`collection`属性设置为引用它所属的集合。

现在我们必须决定何时以及在哪里调用模型的`fetch()`方法。实际上，我们并不需要在应用主页面上的 Summary 视图中获取度量详情；只有在创建 Details 视图时，我们才需要去获取这些数据。我们可以方便地在视图的`initialize()`方法中执行此操作。

```
Running.Views.Details = Backbone.View.extend({
    initialize: **function** () {
        **if** (!**this**.model.get("metrics") ||
            **this**.model.get("metrics").length === 0) {
            **this**.model.fetch();
        }
    },
```

你可能认为请求的异步特性可能会给我们的视图带来问题。毕竟，我们在渲染新创建的视图时试图绘制图表。是不是会在服务器响应之前（即在我们还没有任何图表数据之前）就绘制了图表？实际上，几乎可以肯定的是，我们的视图会在数据可用之前就尝试绘制图表。尽管如此，由于我们构建视图的方式，根本没有问题。

魔法就在我们 Charts 视图的`initialize()`方法中的一行语句。

```
Running.Views.Charts = Backbone.View.extend({
    initialize: **function** () {
        **this**.listenTo(**this**.model,
            "change:metrics change:gps", **this**.render);
        *// Code continues...*
```

该语句告诉 Backbone.js，当关联模型的`metrics`（或`gps`）属性发生变化时，我们的视图希望知道。服务器响应`fetch()`并更新该属性后，Backbone.js 会调用视图的`render()`方法，并尝试（再次）绘制图表。

这个过程中涉及了很多内容，所以一步步来看可能会更有帮助。

1.  应用程序调用了 Runs 集合的`fetch()`方法。

1.  Backbone.js 向服务器发送请求，获取活动列表。

1.  服务器的响应包括每个活动的摘要信息，Backbone.js 使用这些信息来创建初始的 Run 模型。

1.  应用程序为特定的 Run 模型创建一个 Details 视图。

1.  该视图的`initialize()`方法调用了特定模型的`fetch()`方法。

1.  Backbone.js 向服务器发送请求，获取该活动的详细信息。

1.  与此同时，应用程序渲染它刚刚创建的 Details 视图。

1.  Details 视图创建了一个 Charts 视图并渲染了该视图。

1.  因为没有图表的数据，Charts 视图实际上并没有向页面添加任何内容，而是在等待接收任何与模型相关的变化。

1.  最终，服务器在第 6 步中对请求做出响应，并提供了活动的详细信息。

1.  Backbone.js 使用新的详细信息更新模型，并注意到`metrics`属性因此发生了变化。

1.  Backbone.js 触发了 Charts 视图一直在监听的变更事件。

1.  Charts 视图接收到事件触发器，并再次渲染自己。

1.  由于图表数据现在已经可用，`render()`方法能够创建图表并将它们添加到页面上。

呼！幸好 Backbone.js 处理了所有这些复杂性。

到此为止，我们已经成功地检索到跑步的详细指标，但还没有添加任何 GPS 数据。Nike+ 需要一个额外的请求来获取这些数据，所以我们将使用类似的过程。然而，在这种情况下，我们不能依赖 Backbone.js，因为 GPS 请求的 URL 是 Nike+ 独有的。该 URL 是通过将单个活动的 URL 和 `/gps` 拼接在一起形成的——例如，* [`api.nike.com/v1/me/sport/activities/2126456911/gps/`](https://api.nike.com/v1/me/sport/activities/2126456911/gps/)*。

为了发出额外的请求，我们可以在常规的 `fetch()` 方法中添加一些代码。我们将在 Backbone.js 请求指标详细信息的同时请求 GPS 数据。基本方法如以下代码片段所示，非常简单。我们首先检查活动是否有任何 GPS 数据。我们可以通过检查服务器提供的活动摘要中的 `isGpsActivity` 属性来做到这一点。如果有，我们就可以请求该数据。无论如何，我们还需要执行模型的常规 `fetch()` 过程。我们通过获取模型的标准 `fetch()` 方法的引用（`Backbone.Model.prototype.fetch`），然后调用该方法来实现。我们将相同的 `options` 参数传递给它。

```
Running.Models.Run = Backbone.Model.extend({
    fetch: **function**(options) {
        **if** (**this**.get("isGpsActivity")) {
            *// Request GPS details from the server*
        }
        **return** Backbone.Model.prototype.fetch.call(**this**, options);
     },
```

接下来，为了向 Nike+ 发出请求，我们可以使用 jQuery 的 AJAX 函数。由于我们请求的是 JavaScript 对象（JSON 数据），因此 `$.getJSON()` 函数是最合适的。首先，我们通过将 `this` 赋值给局部变量 `model` 来保留对跑步的引用。我们需要这个变量，因为在 jQuery 执行我们的回调时，`this` 将不再指向模型。然后，我们调用 `$.getJSON()` 并传递三个参数。第一个参数是请求的 URL。我们通过调用模型的 `url()` 方法并附加尾部的 `/gps` 来从 Backbone.js 获取这个 URL。第二个参数是请求时要包含的数据值。像往常一样，我们需要包含一个授权令牌。就像之前一样，我们可以从集合中获取该令牌的值。最后一个参数是回调函数，当 jQuery 收到服务器响应时执行。在我们的例子中，函数会将模型的 `gps` 属性设置为响应数据。

```
**if** (**this**.get("isGpsActivity")) {
    **var** model = **this**;
    $.getJSON(
        **this**.url() + "/gps",
        { authorization_token:
          **this**.collection.settings.authorization_token },
        **function**(data) { model.set("gps", data); }
    );
}
```

不出所料，检索 GPS 数据的过程与检索详细指标的过程相同。最初，我们的地图视图没有创建跑步地图所需的数据。然而，由于它在监听模型的 `gps` 属性的变化，因此当数据可用时，它会收到通知。此时，它可以完成 `render` 函数，用户将能够查看跑步的漂亮地图。

## 将一切整合起来

在本章的这一部分，我们已经具备了构建一个简单数据驱动型 Web 应用的所有组件。现在，我们将这些组件组装成应用程序。在这一节结束时，我们将拥有一个完整的应用程序。用户通过访问网页启动应用程序，而我们的 JavaScript 代码则从这里开始运行。结果是一个*单页应用*（SPA）。SPA 之所以流行，是因为 JavaScript 代码能够立即响应用户的操作，这比传统网站通过服务器与位于遥远网络另一端的服务器通信要快得多。用户通常会对这种迅速且响应灵敏的结果感到满意。

尽管我们的应用程序只在一个网页上执行，但用户仍然期望浏览器能提供某些行为。比如，他们希望能够书签保存页面、与朋友分享，或使用浏览器的前进和后退按钮进行导航。传统的网站可以依赖浏览器来支持这些行为，但单页应用却不能。如我们接下来的步骤所示，我们必须编写一些额外的代码，以实现用户期望的行为。

### 步骤 1：创建 Backbone.js 路由器

到目前为止，我们已经看过了三个 Backbone.js 组件——模型、集合和视图——这些都可以在任何 JavaScript 应用程序中派上用场。第四个组件是*路由器*，它对于单页应用特别有用。你不会惊讶地发现，我们可以使用 Yeoman 来创建路由器的脚手架。

```
$ **yo** backbone:router app
   **create** app/scripts/routes/app.js
   **invoke**   backbone-mocha:router
   **create**     test/routers/app.spec.js
```

请注意，我们将路由器命名为`app`。正如你从这个名字中可能会预期的那样，我们将这个路由器作为应用程序的主要控制器。这个方法有优缺点。一些开发者认为路由器应该严格限于路由功能，而另一些开发者则认为路由器是协调整个应用程序的自然位置。对于像我们这样简单的示例，将一些额外的代码添加到路由器中来控制应用程序其实并不会带来太大问题。然而，在复杂的应用程序中，最好还是将路由功能与应用程序控制分开。Backbone.js 的一个优点就是它能够支持这两种方法。

在脚手架搭建完成后，我们可以开始将路由器代码添加到*app.js*文件中。我们将定义的第一个属性是`routes`。这个属性是一个对象，其键是 URL 片段，值是路由器的方法。下面是我们的起点。

```
Running.Routers.App = Backbone.Router.extend({
    routes: {
        "":         "summary",
        "runs/:id": "details"
    },
});
```

第一个路由的 URL 片段为空（`""`）。当用户访问我们的页面且没有指定路径时，路由器将调用其`summary()`方法。例如，如果我们使用*greatrunningapp.com*域名托管我们的应用程序，那么用户在浏览器中输入*[`greatrunningapp.com/`](http://greatrunningapp.com/)*时就会触发这个路由。在我们查看第二个路由之前，让我们看看`summary()`方法的作用。

代码和我们之前看到的一样。`summary()` 方法创建一个新的 Runs 集合，获取该集合，创建该集合的 Summary 视图，并将该视图渲染到页面上。访问我们应用程序首页的用户将看到他们的运行摘要。

```
summary: **function**() {
    **this**.runs = **new** Running.Collections.Runs([],
        {authorizationToken: "authorize me"});
    **this**.runs.fetch();
    **this**.summaryView = **new** Running.Views.Summary({collection: **this**.runs});
    $("body").html(**this**.summaryView.render().el);
},
```

现在我们可以考虑第二条路由。它的 URL 片段是 *runs/:id*。其中 *runs/* 是标准的 URL 路径，而 *:id* 是 Backbone.js 用来标识任意变量的方式。通过这条路由，我们告诉 Backbone.js 寻找一个以 *[`greatrunningapp.com/runs/`](http://greatrunningapp.com/runs/)* 开头的 URL，并将其后面的部分视为 `id` 参数的值。我们将在路由器的 `details()` 方法中使用这个参数。下面是我们开始开发该方法的方式：

```
details: **function**(id) {
    **this**.run = **new** Running.Models.Run();
    **this**.run.id = id;
    **this**.run.fetch();
    **this**.detailsView = **new** Running.Views.Details({model: **this**.run});
    $("body").html(**this**.detailsView.render().el);
    },
```

如你所见，代码几乎与 `summary()` 方法相同，只是我们这里只显示了单个运行，而不是整个集合。我们创建了一个新的 Run 模型，将其 `id` 设置为 URL 中的值，从服务器获取该模型，创建一个 Details 视图，并将该视图渲染到页面上。

路由器让用户可以直接访问单个运行，通过使用合适的 URL。例如，URL *[`greatrunningapp.com/runs/2126456911`](http://greatrunningapp.com/runs/2126456911)* 将获取并显示 `activityId` 等于 `2126456911` 的运行的详细信息。注意，路由器无需关心是什么特定的属性定义了模型的唯一标识符。它使用通用的 `id` 属性。只有模型本身需要知道服务器使用的实际属性名称。

有了路由器，我们的单页面应用程序可以支持多个 URL。一个 URL 显示所有运行的摘要，而其他 URL 显示特定运行的详细信息。由于 URL 是独立的，用户可以像浏览不同的网页一样处理它们。他们可以将其添加到书签，发送电子邮件，或者在社交网络上分享。每当他们或他们的朋友返回某个 URL 时，它将显示与之前相同的内容。这正是用户对 Web 的期望行为。

然而，用户还期望有另一种行为，我们尚未支持。用户希望能使用浏览器的后退和前进按钮在浏览历史中进行导航。幸运的是，Backbone.js 提供了一个工具来处理这个功能。它就是 *history* 功能，我们可以在应用路由器初始化时启用它。

```
Running.Routers.App = Backbone.Router.extend({
    initialize: **function**() {
        Backbone.history.start({pushState: **true**});
    },
```

对于我们简单的应用程序，这就是我们处理浏览历史的全部工作。其余的由 Backbone.js 负责。

### 注意

**支持多个 URL 可能需要对您的 Web 服务器进行一些配置。更具体地说，您需要让服务器将所有 URL 映射到同一个 *index.html* 文件。此配置的细节取决于 Web 服务器技术。对于开源的 Apache 服务器，** *.htaccess* **文件可以定义这种映射。**

### 步骤 2：支持不属于任何集合的 Run 模型

不幸的是，如果我们尝试在现有的 Run 模型中使用上述代码，我们将遇到一些问题。首先是我们的 Run 模型依赖于它的父集合。例如，它使用`this.collection.settings.authorization_token`来找到授权令牌。然而，当浏览器直接访问某个特定 run 的 URL 时，不会有集合。以下代码对这个问题做了调整：

```
    Running.Routers.App = Backbone.Router.extend({
       routes: {
           "":         "summary",
           "runs/:id": "details"
       },
       initialize: **function**(options) {
           **this**.options = options;
           Backbone.history.start({pushState: **true**});
       },
       summary: **function**() {
           **this**.runs = **new** Running.Collections.Runs([],
➊             {authorizationToken: **this**.options.token});
           **this**.runs.fetch();
           **this**.summaryView = **new** Running.Views.Summary({
               collection: **this**.runs});
           $("body").html(**this**.summaryView.render().el);
       },
       details: **function**(id) {
           **this**.run = **new** Running.Models.Run({},
➋             {authorizationToken: **this**.options.token});
           **this**.run.id = id;
           **this**.run.fetch();
           **this**.detailsView = **new** Running.Views.Details({
               model: **this**.run});
           $("body").html(**this**.detailsView.render().el);
   });
```

现在我们在创建 Run 模型时提供授权令牌 ➋。我们还将其值作为选项传递给在创建时的集合 ➊。

接下来，我们需要修改 Run 模型，以使用这个新参数。我们将像在 Runs 集合中一样处理令牌。

```
Running.Models.Run = Backbone.Model.extend({
    initialize: **function**(attrs, options) {
        **this**.settings = { authorization_token: "" };
        options = options || {};
        **if** (**this**.collection) {
            _(**this**.settings).extend(_(**this**.collection.settings)
                .pick(_(**this**.settings).keys()));
        }
        _(**this**.settings).extend(_(options)
            .pick(_(**this**.settings).keys()));
},
```

我们首先为所有设置定义默认值。与集合不同，我们的模型所需的唯一设置是`authorization_token`。接下来，我们确保我们有一个`options`对象。如果没有提供，我们就创建一个空对象。在第三步，我们通过查看`this.collection`来检查模型是否属于一个集合。如果该属性存在，我们就从集合中获取任何设置并覆盖默认值。最后一步将用任何作为选项传递给构造函数的设置来覆盖结果。当像前面的代码一样，我们的路由提供了一个`authorization_token`值时，模型将使用这个值。当模型是集合的一部分时，模型没有与之关联的特定令牌。在这种情况下，我们会回退到集合的令牌。

现在我们有了授权令牌，可以将其添加到模型的 AJAX 请求中。代码与我们在 Runs 集合中的代码几乎相同。我们需要一个属性来指定 REST 服务的 URL，并且我们需要覆盖常规的`sync()`方法，将令牌添加到所有请求中。

```
urlRoot: "https://api.nike.com/v1/me/sport/activities",

sync: **function**(method, model, options) {
    options = options || {};
    _(options).extend({
        data: { authorization_token: **this**.settings.authorization_token }
    });
    Backbone.sync(method, model, options);
},
```

这段额外的代码处理了授权，但我们的模型仍然存在问题。在前一部分中，Run 模型仅作为 Runs 集合的一部分存在，获取该集合的操作会为每个模型填充摘要属性，例如`isGpsActivity`。每当我们尝试获取模型的详细信息时，模型可以安全地检查该属性，并在适当时同时发起 GPS 数据请求。然而，现在我们单独创建一个 Run 模型，没有集合的帮助。当我们获取模型时，我们唯一知道的属性是唯一标识符。因此，在服务器响应获取请求之前，我们无法决定是否请求 GPS 数据。

为了将 GPS 数据请求与一般的获取请求分开，我们可以将该请求移到一个独立的方法中。代码与之前相同（当然，唯一的不同是我们从本地设置中获取授权令牌）。

```
fetchGps: **function**() {
    **if** (**this**.get("isGpsActivity") && !**this**.get("gps")) {
        **var** model = **this**;
        $.getJSON(
            **this**.url() + "/gps",
            { authorization_token: **this**.settings.authorization_token },
            **function**(data) { model.set("gps", data); }
        );
    }
}
```

为了触发这个方法，我们将告诉 Backbone.js，每当模型发生变化时，它应该调用`fetchGps()`方法。

```
initialize: **function**(attrs, options) {
    **this**.on("change", **this**.fetchGps, **this**);
```

当`fetch()`响应到达时，Backbone.js 会检测到这种变化，并填充模型，此时我们的代码可以安全地检查`isGpsActivity()`并发出额外的请求。

### 步骤 3：让用户切换视图

现在我们的应用程序能够正确显示两种不同的视图，是时候让用户也参与其中了。在这一步中，我们将为他们提供一种轻松的方式在视图之间切换。首先让我们考虑一下`Summary`视图。如果用户能够点击表格中出现的任何一行并立即跳转到该行的详细视图，那就太好了。

我们的第一个决定是将监听点击事件的代码放在哪里。起初，可能会觉得`SummaryRow`视图是放置这段代码的自然位置。该视图负责渲染行，因此让该视图处理与行相关的事件似乎是合乎逻辑的。如果我们想这么做，Backbone.js 使得这一切变得非常简单；我们只需要在视图中添加一个额外的属性和一个额外的方法。它们可能看起来像下面这样：

```
Running.Views.SummaryRow = Backbone.View.extend({
    events: {
        "click": "clicked"
    },
    clicked: **function**() {
        *// Do something to show the Details view for this.model*
    },
```

`events`属性是一个对象，列出了我们视图感兴趣的事件。在这种情况下，只有一个事件：`click`事件。该值——在此情况下是`clicked`——标识了当事件发生时 Backbone.js 应调用的方法。我们暂时跳过了该方法的细节。

从技术上讲，这种方法没有问题，如果我们继续实现它，可能会很好地工作。然而，它非常低效。想象一个用户在 Nike+上存储了数百条跑步记录。摘要表格将有数百行，每一行都将有自己监听`click`事件的函数。这些事件处理程序可能会占用大量内存和其他浏览器资源，导致我们的应用变得迟缓。幸运的是，还有一种不同的方法，对浏览器的压力要小得多。

与其为每一行都设置数百个监听`click`事件的事件处理程序，我们不如使用一个事件处理程序监听所有表格行的点击事件。由于`Summary`视图负责所有这些行，它是添加该处理程序的自然位置。我们仍然可以利用 Backbone.js 使实现变得简单，只需向视图中添加一个`events`对象。然而，我们可以做得更好一些。我们不关心表头的`click`事件，只有表格主体中的行才是我们关心的。通过在事件名称后添加类似 jQuery 的选择器，我们可以将处理程序限制为匹配该选择器的元素。

```
Running.Views.Summary = Backbone.View.extend({
    events: {
        "click tbody": "clicked"
    },
```

上述代码要求 Backbone.js 监听我们视图中的`<tbody>`元素中的`click`事件。当事件发生时，Backbone.js 将调用我们视图的`clicked()`方法。

在我们为`clicked()`方法编写任何代码之前，我们需要一种方法让它弄清楚用户选择了哪个特定的运行模型。事件处理器能够判断用户点击的是哪一行，但它怎么知道那一行代表的是哪个模型呢？为了让处理器能够轻松获取答案，我们可以直接在行的标记中嵌入必要的信息。这需要对我们之前创建的`renderRun()`方法做一些小调整。

修改后的方法仍然为每个模型创建一个 SummaryRow 视图，渲染该视图并将结果添加到表格主体中。不过，现在我们会在将行添加到页面之前，增加一个额外的步骤。我们为该行添加一个特殊的属性`data-id`，并将其值设置为模型的唯一标识符。我们使用`data-id`是因为 HTML5 标准允许任何以`data-`开头的属性名。这种形式的自定义属性不会违反标准，也不会导致浏览器错误。

```
renderRun: **function** (run) {
    **var** row = **new** Running.Views.SummaryRow({ model: run });
    row.render();
    row.$el.attr("data-id", run.id);
    **this**.$("tbody").append(row.$el);
},
```

对于标识符为`2126456911`的运行，生成的标记大致如下所示：

```
**<tr** data-id="2126456911"**>**
    **<td>**04/09/2013**</td>**
    **<td>**0:22:39**</td>**
    **<td>**2.33 Miles**</td>**
    **<td>**240**</td>**
    **<td>**9:43**</td>**
**</tr>**
```

一旦我们确保页面中的标记与 Run 模型有回溯引用，我们就可以在`clicked`事件处理器中利用这些标记。当 Backbone.js 调用该处理器时，它会传递一个事件对象。从这个对象中，我们可以找到事件的目标。在`click`事件的情况下，目标就是用户点击的 HTML 元素。

```
clicked: **function** (ev) {
    **var** $target = $(ev.target)
```

从前面的标记可以看出，表格行大部分是由表格单元格（`<td>`元素）组成的，因此表格单元格很可能是`click`事件的目标。我们可以使用 jQuery 的`parents()`函数来查找点击目标的父表格行。

```
clicked: **function** (ev) {
    **var** $target = $(ev.target)
    **var** id = $target.attr("data-id") ||
             $target.parents("[data-id]").attr("data-id");
```

一旦我们找到了那个父行，我们就提取`data-id`属性的值。为了安全起见，我们还要处理用户不小心点击表格行本身而不是单个表格单元格的情况。

在获取了属性值之后，我们的视图知道了用户选择了哪个运行模型；现在它需要对这些信息进行处理。可能会有一种冲动让 Summary 视图直接渲染运行模型的 Details 视图，但这样做并不合适。Backbone.js 视图应该只负责自己和它包含的任何子视图。这种做法使得视图可以在各种上下文中安全地复用。例如，我们的 Summary 视图可能会在一个没有 Details 视图的上下文中使用。在这种情况下，直接切换到 Details 视图，充其量会引发错误。

由于总结视图本身不能响应用户点击表格行的操作，它应该遵循应用的层次结构，实际上将信息“传递给上层”。Backbone.js 为这种类型的通信提供了一个方便的机制：自定义事件。总结视图不会直接响应用户点击，而是触发一个自定义事件。其他部分可以监听这个事件并作出相应的响应。如果没有其他代码在监听这个事件，那么什么也不会发生，但至少总结视图可以说它已经完成了自己的工作。

这是我们如何在视图中生成自定义事件：

```
clicked: **function** (ev) {
    **var** $target = $(ev.target)
    **var** id = $target.attr("data-id") ||
             $target.parents("[data-id]").attr("data-id");
    **this**.trigger("select", id);
}
```

我们将事件命名为 `select`，以表明用户选择了一个特定的运行，并将该运行的标识符作为与事件相关的参数传递。到此为止，总结视图已经完成。

应该响应这个自定义事件的组件是最初创建总结视图的组件：我们的应用路由器。我们首先需要监听这个事件。我们可以在 `summary()` 方法中创建它之后立即进行监听。

```
Running.Routers.App = Backbone.Router.extend({
    summary: **function**() {
        **this**.runs = **new** Running.Collections.Runs([],
            {authorizationToken: **this**.options.token});
        **this**.runs.fetch();
        **this**.summaryView = **new** Running.Views.Summary({
            collection: **this**.runs});
    $("body").html(**this**.summaryView.render().el);
    **this**.summaryView.on("select", **this**.selected, **this**);
},
```

当用户从总结视图中选择一个特定的运行时，Backbone.js 会调用我们路由器的 `selected()` 方法，并将任何事件数据作为参数传入。在我们的例子中，事件数据是唯一标识符，所以它成为该方法的参数。

```
Running.Routers.App = Backbone.Router.extend({
    selected: **function**(id) {
        **this**.navigate("runs/" + id, { trigger: **true** });
    }
```

如你所见，事件处理器的代码非常简单。它构造一个对应详细信息视图的 URL（`"runs/" + id`），并将该 URL 传递给路由器的 `navigate()` 方法。该方法更新浏览器的导航历史记录。第二个参数（`{ trigger: true }`）告诉 Backbone.js，假如用户实际导航到该 URL，也要执行相应的操作。由于我们已经设置了 `details()` 方法来响应 *runs/:id* 格式的 URL，Backbone.js 会调用 `details()`，然后我们的路由器将显示所选运行的详细信息。

当用户查看详细信息视图时，我们还希望提供一个按钮，让他们能够轻松地导航到总结视图。与总结视图一样，我们可以为按钮添加一个事件处理器，当用户点击按钮时触发一个自定义事件。

```
Running.Views.Details = Backbone.View.extend({
    events: {
        "click button": "clicked"
    },
    clicked: **function** () {
        **this**.trigger("summarize");
    }
```

当然，我们需要在路由器中监听这个自定义事件。

```
Running.Routers.App = Backbone.Router.extend({
    details: **function**(id) {
        *// Set up the Details view*
        *// Code continues...*
        **this**.detailsView.on("summarize", **this**.summarize, **this**);
    },
    summarize: **function**() {
        **this**.navigate("", { trigger: **true** });
    },
```

我们再次通过构造适当的 URL 并触发导航来响应用户。

你可能会想，为什么我们必须显式触发导航变化？难道这不是默认行为吗？虽然这看起来合理，但在大多数情况下，这并不合适。我们的应用足够简单，触发路由就能正常工作。然而，更复杂的应用可能希望根据用户是执行了某个操作还是直接导航到特定 URL 来采取不同的行动。处理每种情况的代码最好分开写。在第一种情况下，应用仍然希望更新浏览器的历史记录，但不希望触发完全的导航操作。

### 第四步：精细调整应用程序

到目前为止，我们的应用程序已经完全功能化。用户可以查看他们的总结，收藏并分享特定运行的详情，并通过浏览器的前进和后退按钮来导航应用程序。然而，在我们宣布它完成之前，还有最后一项清理工作要做。应用程序的性能不是最优的，而且更为关键的是，它存在*内存泄漏*，即占用了浏览器的小部分内存，却从未释放。

最明显的问题出现在路由器的`summary()`方法中，具体代码如下：

```
Running.Routers.App = Backbone.Router.extend({
    summary: **function**() {
        **this**.runs = **new** Running.Collections.Runs([],
            {authorizationToken: **this**.options.token});
        **this**.runs.fetch();
        **this**.summaryView = **new** Running.Views.Summary({
            collection: **this**.runs});
        $("body").html(**this**.summaryView.render().el);
        **this**.summaryView.on("select", **this**.selected, **this**);
    },
```

每次执行该方法时，它都会创建一个新的集合，获取该集合，并为该集合渲染一个 Summary 视图。显然，第一次执行该方法时，我们必须经过这些步骤，但之后就不需要重复执行了。如果用户选择了特定的运行记录并返回到总结视图，集合或其视图不会发生变化。我们可以在方法中添加一个检查，只有在视图不存在时才执行这些步骤。

```
summary: **function**() {
    **if** (!**this**.summaryView) {
        **this**.runs = **new** Running.Collections.Runs([],
            {authorizationToken: **this**.options.token});
        **this**.runs.fetch();
        **this**.summaryView = **new** Running.Views.Summary({
            collection: **this**.runs});
        **this**.summaryView.render();
        **this**.summaryView.on("select", **this**.selected, **this**);
    }
    $("body").html(**this**.summaryView.el);
},
```

我们还可以在`details()`方法中添加一个检查。当该方法执行并且 Summary 视图存在时，我们可以使用 jQuery 的`detach()`函数“搁置”Summary 视图的标记。这将保留标记及其事件处理程序，以便用户返回到总结页面时，可以快速重新插入。

```
details: **function**(id) {
    **if** (**this**.summaryView) {
        **this**.summaryView.$el.detach();
    }
    **this**.run = **new** Running.Models.Run({},
        {authorizationToken: **this**.options.token});
    **this**.run.id = id;
    **this**.run.fetch();
    $("body").html(**this**.detailsView.render().el);
    **this**.detailsView.on("summarize", **this**.summarize, **this**);
},
```

这些更改使得在 Summary 视图之间的切换更加高效。我们还可以对 Details 视图做类似的优化。在`details()`方法中，如果运行记录已经存在于集合中，我们就不必重新获取它。我们可以添加一个检查，如果该运行的数据已经可用，就不再进行获取操作。

```
details: **function**(id) {
    **if** (!**this**.runs || !(**this**.run = **this**.runs.get(id))) {
        **this**.run = **new** Running.Models.Run({},
            {authorizationToken: **this**.options.token});
        **this**.run.id = id;
        **this**.run.fetch();
    }
    **if** (**this**.summaryView) {
        **this**.summaryView.$el.detach();
    }
    **this**.detailsView = **new** Running.Views.Details({model: **this**.run});
    $("body").html(**this**.detailsView.render().el);
    **this**.detailsView.on("summarize", **this**.summarize, **this**);
},
```

在`summary()`方法中，我们不希望像处理 Summary 视图时那样简单地将 Details 视图搁置一旁。这是因为，如果用户开始查看所有可用的运行记录，可能会有成百上千个 Details 视图存在。因此，我们希望干净地删除 Details 视图，这样浏览器就可以释放该视图占用的内存。

如下代码所示，我们将分三个步骤进行操作。

1.  移除我们之前为捕捉`summarize`事件而添加到 Details 视图的事件处理程序。

1.  调用视图的`remove()`方法，以释放它所占用的内存。

1.  将`this.detailsView`设置为`null`，表示该视图不再存在。

```
summary: **function**() {
    **if** (**this**.detailsView) {
        **this**.detailsView.off("summarize");
        **this**.detailsView.remove();
        **this**.detailsView = **null**;
    }
    **if** (!**this**.summaryView) {
        **this**.runs = **new** Running.Collections.Runs([],
            {authorizationToken: **this**.options.token});
        **this**.runs.fetch();
        **this**.summaryView = **new** Running.Views.Summary({
            collection: **this**.runs});
        **this**.summaryView.render();
        **this**.summaryView.on("select", **this**.selected, **this**);
    }
    $("body").html(**this**.summaryView.el);
},
```

这样一来，我们的应用程序就完成了！你可以在书籍的源代码中查看最终结果 (*[`jsDataV.is/source/`](http://jsDataV.is/source/)*)。

## 总结

在本章中，我们完成了一个数据驱动的 Web 应用程序。首先，我们看到 Backbone.js 如何让我们灵活地与不完全遵循常规的 REST API 进行交互。接着，我们使用了 Backbone.js 路由器，确保我们的单页面应用像一个完整的网站一样运行，这样我们的用户可以像期望的那样与之交互。
