## <samp class="SANS_Futura_Std_Bold_Condensed_B_11">4</samp> <samp class="SANS_Dogma_OT_Bold_B_11">第一个 Dash 应用</samp>

![](img/opener-img.png)

在本章中，你将构建你的第一个 Dash 应用。我们将分析自 2011 年以来 16 位明星收到的 Twitter 点赞数。你可以通过本书的资源下载数据，网址是 [*https://<wbr>nostarch<wbr>.com<wbr>/python<wbr>-dash*](https://nostarch.com/python-dash)。我们进行的这种分析在社交媒体分析领域非常常见，通常用于更好地理解受众行为、帖子效果以及账户的整体表现。

第一个仪表盘将绘制每条推文的点赞数。一旦你掌握了使用 Dash 绘制这种简单图表的过程，你就能将你的技能扩展到其他更大、更复杂的数据领域：Instagram 帖子的浏览量、Facebook 个人主页的访问量、LinkedIn 帖子的点击率，以及 YouTube 视频的表现。

本章将为你提供足够的 Dash 知识，以便你创建自己的仪表盘应用。你将学习如何将数据集成到应用中，管理多个仪表盘组件，构建基本图表（如折线图），并通过回调装饰器为仪表盘添加交互功能。首先，让我们下载代码并运行应用，看看它的功能。

### <samp class="SANS_Futura_Std_Bold_B_11">设置项目</samp>

打开 PyCharm，创建一个新项目，并命名为 *my-first-app*（项目名称应该是新建项目对话框中 Location 字段最后一个反斜杠后的后缀文本）。使用标准的 Virtualenv 设置你的虚拟环境。

<samp class="SANS_Dogma_OT_Bold_B_15">注意</samp>

*本章的代码假设你正在使用 Python IDE，如 PyCharm。如果你没有安装 IDE 或者没有设置虚拟环境，请回到第二章并完成 Python 环境的设置。如果你使用的是其他编程环境，只需根据你的环境调整本章的说明即可。本章的代码还要求 Python 3.6 或更高版本。*

接下来，你需要将本章的仪表盘应用文件下载到你的项目文件夹中。不同于我们在第二章中使用 Git 克隆仓库的做法，这次我们将直接下载 ZIP 文件。尝试不同的项目设置方式是值得的，因为你可能会遇到一些不能直接作为 Git 仓库提供的项目。要使用 ZIP 文件，访问 GitHub 仓库 [*https://<wbr>github<wbr>.com<wbr>/DashBookProject<wbr>/Plotly<wbr>-Dash*](https://github.com/DashBookProject/Plotly-Dash)，点击 **Code**，然后点击 **Download ZIP**，如图 4-1 所示。

![](img/Figure4-1.png)

<samp class="SANS_Futura_Std_Book_Oblique_I_11">图 4-1：从 GitHub 下载应用代码</samp>

一旦你将 *Plotly-Dash-master.zip* 文件下载到电脑中，打开它并进入 *Chapter-4* 文件夹。将该文件夹中的所有文件复制到你最近创建的 *my-first-app* 项目文件夹中。项目文件夹应具有以下结构：

<samp class="SANS_TheSansMonoCd_W5Regular_11">– my-first-app</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">|––assets</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      ––mystyles.css</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">|––tweets.csv</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">|––twitter_app.py</samp>

*assets* 文件夹将保存 CSS 脚本。*tweets.csv* 文件保存我们将使用的数据，*twitter_app.py* 是我们用来运行应用的主要文件。

现在，我们将在虚拟环境中安装必要的库。转到 PyCharm 窗口底部的 Terminal 标签，见图 4-2。

![](img/Figure4-2.png)

<samp class="SANS_Futura_Std_Book_Oblique_I_11">图 4-2：在 PyCharm 中打开终端</samp>

输入并执行以下代码行以安装 pandas 和 Dash（Plotly 包会随 Dash 自动安装，因此无需单独安装 Plotly，NumPy 包也会随 pandas 自动安装）：

<samp class="SANS_TheSansMonoCd_W5Regular_11">$</samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">pip install pandas</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">$</samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">pip install dash</samp>

要检查库是否正确安装，请输入：

<samp class="SANS_TheSansMonoCd_W5Regular_11">$</samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">pip list</samp>

这将创建一个列出当前虚拟环境中所有 Python 包的清单。如果所有包都列出，你可以继续。请注意，pandas 和 Dash 的所有依赖项也会被列出，所以你可能会看到比你安装的两个包更多的库。

接下来，在 PyCharm 中打开 *twitter_app.py* 并运行脚本。你应该看到以下信息：

<samp class="SANS_TheSansMonoCd_W5Regular_11">* Serving Flask app "twitter_app" (lazy loading)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">* Environment: production</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">WARNING: This is a development server. Do not use it in a production deployment.</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">Use a production WSGI server instead.</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">* Debug mode: on</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">Dash is running on</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">http://127.0.0.1:8050/</samp>

这个警告只是提醒我们应用正在开发服务器上运行，这是完全正常的。要打开你的应用，点击 HTTP 链接或将其复制并粘贴到浏览器的地址栏中。

恭喜！你现在应该能看到你的第一个 Dash 应用，它应该像图 4-3 那样。

![](img/Figure4-3.png)

<samp class="SANS_Futura_Std_Book_Oblique_I_11">图 4-3：Twitter 点赞分析应用</samp>

玩得开心！在你的仪表盘应用中随便玩玩。改变下拉框的值，点击链接，点击图表图例，并按住鼠标左键拖动鼠标来缩放到某个日期范围。看看你能推导出什么信息。

现在让我们来看一下应用的代码。大多数 Dash 应用程序的代码布局是类似的：

1.  导入必要的 Python 库。

2.  读取数据。

3.  分配样式表以描述应用的显示方式。

4.  构建应用布局，定义如何显示所有元素。

5.  创建回调函数，以实现应用组件之间的交互性。

由于 Dash 应用大多遵循这一大纲，我们将按此顺序来浏览代码。

<samp class="SANS_Futura_Std_Bold_Condensed_Oblique_I_11">导入库</samp>

首先看一下我们将要使用的库，见列表 4-1。

<samp class="SANS_TheSansMonoCd_W5Regular_11">import pandas as pd</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">import plotly.express as px</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">from dash import Dash, dcc, html, Input, Output</samp>

<samp class="SANS_Futura_Std_Book_Oblique_I_11">列表 4-1：来自</samp> <samp class="SANS_Futura_Std_Book_11">twitter_app.py 的导入部分</samp>

我们首先导入 pandas 来处理数据。然后导入 Plotly，这是一个流行的 Python 可视化库。在 Plotly 中有两种主要的创建图表方式。我们使用的是*Plotly Express*，这是一个用于通过单次函数调用快速创建图表的高级接口，只需很少的代码行。它有足够的功能，能够让你无缝地快速构建图表，并且是两者中用于简单应用最容易使用的。

另一种选择是*Plotly Graph Objects*，这是一个从底层构建图表的低级接口。使用 Graph Objects 时，你需要定义数据、布局以及有时的帧，这些都使得图表构建过程更加复杂。然而，它的完整功能集允许你以多种方式自定义图表，从而增加图表的丰富性，所以一旦你掌握了 Dash 的基础，并且需要构建更复杂的图表时，你可能会选择使用 Plotly Graph Objects。我们在大多数情况下会使用 Plotly Express，只有在更复杂的情况下才会回退到 Graph Objects。

接下来，我们导入一些 Dash 库来处理组件和依赖项。*组件*是可以组合在一起为用户创建丰富复杂界面的构建模块，例如下拉菜单、范围滑块和单选按钮。Dash 附带了两个由 Plotly 维护的关键组件库：dash-html-components（HTML）和 dash-core-components（DCC）。*dash-html-components* 库包含诸如标题和分隔符等结构性元素，用于设置页面上元素的样式和位置，而 *dash-core-components* 提供应用程序的核心功能，如用户输入字段和图形。

<samp class="SANS_Futura_Std_Bold_Condensed_Oblique_I_11">数据管理</samp>

在这个应用程序中，我们使用 CSV 电子表格作为数据源。为了使用数据，我们需要通过 pandas 将其读取到内存中，但在此之前，我们必须 *清洗* 数据。这意味着通过执行标准化字符串的大小写、时间格式、去除空格以及为缺失值添加空值等操作，来准备数据以供分析和绘制图表。当数据是 *脏的* 时，通常它是无序的，并且可能包含缺失值。如果你尝试使用脏数据，图表可能无法正常工作，分析可能不准确，而且你会发现过滤数据很困难。清洗数据确保数据可读、可展示并且可以绘图。

列表 4-2 显示了代码中的数据管理部分。

❶ <samp class="SANS_TheSansMonoCd_W5Regular_11">df =</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">pd.read_csv("tweets.csv")</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">df["name"] = pd.Series(df["name"]).str.lower()</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">df["date_time"] = pd.to_datetime(df["date_time"])</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">df = (</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   df.groupby([df["date_time"].dt.date, "name"])[</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      ["number_of_likes", "number_of_shares"]</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   ]</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   .mean()</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   .astype(int)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">df = df.reset_index()</samp>

<samp class="SANS_Futura_Std_Book_Oblique_I_11">列表 4-2：数据管理部分来自</samp> <samp class="SANS_Futura_Std_Book_11">twitter_app.py</samp>

在 ❶ 处，我们获取 CSV 电子表格并将其读取到一个名为 <samp class="SANS_TheSansMonoCd_W5Regular_11">df</samp> 的 pandas DataFrame 中。Dash 应用程序开始时的 DataFrame 通常被称为 *全局 DataFrame*，而数据则是一个 *全局变量*（*全局*意味着对象在函数外部声明，意味着它可以在应用程序中随处访问）。

为了清理数据，我们将名人 <samp class="SANS_TheSansMonoCd_W5Regular_11">name</samp> 列的字符串转换为小写，以便于比较。我们将 <samp class="SANS_TheSansMonoCd_W5Regular_11">date_time</samp> 列转换为 pandas 可识别的日期格式，并根据 <samp class="SANS_TheSansMonoCd_W5Regular_11">date_time</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">name</samp> 对数据进行分组，以确保每一行都有唯一的日期和名称。如果不这样分组数据，就会导致多个相同日期和名称的行，最终生成一个杂乱无章、无法阅读的折线图。

要检查数据，在脚本中添加以下代码行，紧接在 <samp class="SANS_TheSansMonoCd_W5Regular_11">df</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">=</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">df.reset_index()</samp> 之后：

<samp class="SANS_TheSansMonoCd_W5Regular_11">print(df.head())</samp>

重新运行脚本后，你应该在 Python 终端中看到类似以下的内容：

|  | <samp class="SANS_TheSansMonoCd_W5Regular_11">date_time</samp> | <samp class="SANS_TheSansMonoCd_W5Regular_11">name</samp> | <samp class="SANS_TheSansMonoCd_W5Regular_11">number_of_likes</samp> | <samp class="SANS_TheSansMonoCd_W5Regular_11">number_of_shares</samp> |
| --- | --- | --- | --- | --- |
| <samp class="SANS_TheSansMonoCd_W5Regular_11">0</samp> | <samp class="SANS_TheSansMonoCd_W5Regular_11">2010-01-06</samp> | <samp class="SANS_TheSansMonoCd_W5Regular_11">selenagomez</samp> | <samp class="SANS_TheSansMonoCd_W5Regular_11">278</samp> | <samp class="SANS_TheSansMonoCd_W5Regular_11">695</samp> |
| <samp class="SANS_TheSansMonoCd_W5Regular_11">1</samp> | <samp class="SANS_TheSansMonoCd_W5Regular_11">2010-01-07</samp> | <samp class="SANS_TheSansMonoCd_W5Regular_11">jtimberlake</samp> | <samp class="SANS_TheSansMonoCd_W5Regular_11">62</samp> | <samp class="SANS_TheSansMonoCd_W5Regular_11">189</samp> |
| <samp class="SANS_TheSansMonoCd_W5Regular_11">2</samp> | <samp class="SANS_TheSansMonoCd_W5Regular_11">2010-01-07</samp> | <samp class="SANS_TheSansMonoCd_W5Regular_11">selenagomez</samp> | <samp class="SANS_TheSansMonoCd_W5Regular_11">201</samp> | <samp class="SANS_TheSansMonoCd_W5Regular_11">630</samp> |
| <samp class="SANS_TheSansMonoCd_W5Regular_11">3</samp> | <samp class="SANS_TheSansMonoCd_W5Regular_11">2010-01-08</samp> | <samp class="SANS_TheSansMonoCd_W5Regular_11">jtimberlake</samp> | <samp class="SANS_TheSansMonoCd_W5Regular_11">27</samp> | <samp class="SANS_TheSansMonoCd_W5Regular_11">107</samp> |
| <samp class="SANS_TheSansMonoCd_W5Regular_11">4</samp> | <samp class="SANS_TheSansMonoCd_W5Regular_11">2010-01-08</samp> | <samp class="SANS_TheSansMonoCd_W5Regular_11">selenagomez</samp> | <samp class="SANS_TheSansMonoCd_W5Regular_11">349</samp> | <samp class="SANS_TheSansMonoCd_W5Regular_11">935</samp> |

如你所见，结果是一个整洁的 pandas DataFrame，其中的每一行数据表示每个名人在每天的点赞和分享的平均数。

在应用的开始阶段读取和准备数据总是一个好的做法，因为读取数据可能是一个消耗内存的任务；通过在开始时插入数据，你可以确保应用只会将数据加载到内存中一次，而不会在每次用户与仪表盘交互时重复此过程。

<samp class="SANS_Futura_Std_Bold_Condensed_Oblique_I_11">布局和样式</samp>

下一步是管理应用组件的布局和样式，比如标题、图表和下拉菜单。我们将在本章稍后的“Dash 组件”部分进一步了解这些组件；在这里，我们将重点关注布局部分。

在 Dash 应用中，*布局* 指的是组件在应用中的排列方式。*样式* 指的是元素的外观，比如颜色、大小、间距等属性（在 Dash 中被称为 *props*）。对应用进行样式设计可以实现更加定制化和专业化的展示。如果没有样式设置，你可能会得到像 图 4-4 中展示的那样的应用，其中标题没有居中，下拉框字段覆盖了整个页面，并且链接与其上方的下拉框之间没有间距。

![](img/Figure4-4.png)

<samp class="SANS_Futura_Std_Book_Oblique_I_11">图 4-4：没有适当布局和样式的 Twitter 点赞分析应用</samp>

<samp class="SANS_Futura_Std_Bold_Condensed_B_11">对齐</samp>

Dash 应用是基于 Web 的，因此它使用网页的标准语言：HTML（超文本标记语言）。幸运的是，Dash 包含了 Dash HTML 组件模块，它将 Python 转换为 HTML，这意味着我们可以使用 Python 来编写 HTML。

HTML 最基本的组件之一是 <samp class="SANS_TheSansMonoCd_W5Regular_11">Div</samp>，即 *division*（区块）的缩写，它只是一个用于容纳其他元素的容器，并且是将元素分组的一种方式。我们在 Dash 应用中使用的每个组件都将被包含在一个 <samp class="SANS_TheSansMonoCd_W5Regular_11">Div</samp> 中，且一个 <samp class="SANS_TheSansMonoCd_W5Regular_11">Div</samp> 可以包含多个组件。我们构建 <samp class="SANS_TheSansMonoCd_W5Regular_11">Div</samp>，然后对其进行样式设置，以告诉网页浏览器该组件应该放置在哪里，以及应该占据多少空间。

假设我们正在创建一个包含三个下拉菜单的仪表盘应用，下拉菜单由关键字 <samp class="SANS_TheSansMonoCd_W5Regular_11">Dropdown</samp> 表示，如 清单 4-3 中所示。

<samp class="SANS_TheSansMonoCd_W5Regular_11">app.layout =</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">html.Div([</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   html.Div(dcc.Dropdown()),</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   html.Div(dcc.Dropdown()),</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   html.Div(dcc.Dropdown()),</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">])</samp>

<samp class="SANS_Futura_Std_Book_Oblique_I_11">Listing 4-3：示例 Div 代码（非主应用部分）</samp>

这一行 <samp class="SANS_TheSansMonoCd_W5Regular_11">app.layout</samp> 为这个 Dash 应用创建了一个布局。所有与布局相关的内容都必须放在 <samp class="SANS_TheSansMonoCd_W5Regular_11">app.layout</samp> 中。然后，我们创建了一个包含三个下拉菜单的 <samp class="SANS_TheSansMonoCd_W5Regular_11">Div</samp>。

默认情况下，<samp class="SANS_TheSansMonoCd_W5Regular_11">Div</samp> 会占据父容器的整个宽度，这意味着它被认为是一个占满整个页面宽度的大单元格。这样，第一个 <samp class="SANS_TheSansMonoCd_W5Regular_11">Dropdown</samp> 会出现在左上角，并从左到右填满整个页面。第二个 <samp class="SANS_TheSansMonoCd_W5Regular_11">Dropdown</samp> 会出现在第一个 <samp class="SANS_TheSansMonoCd_W5Regular_11">Dropdown</samp> 的正下方，并同样占满整个页面的宽度，第三个 <samp class="SANS_TheSansMonoCd_W5Regular_11">Dropdown</samp> 依此类推。换句话说，每个 <samp class="SANS_TheSansMonoCd_W5Regular_11">Div</samp> 都会占据页面的整个宽度，并将相邻的元素强制移到新的一行。

为了最好地控制一个 <samp class="SANS_TheSansMonoCd_W5Regular_11">Div</samp> 被分配的空间，我们应该将网页定义为一个由行和列组成的网格，并将每个 <samp class="SANS_TheSansMonoCd_W5Regular_11">Div</samp> 放置在网格中的特定单元格内。我们可以使用预制的 CSS 样式表快速定义行和列。CSS（层叠样式表）是另一种用于定义页面如何显示的网页语言。我们将样式表放在外部文件中，或从在线目录中调用一个样式表到我们的应用程序中。我们使用的是来自 [*https://<wbr>codepen<wbr>.io*](https://codepen.io) 的样式表。该样式表由 Plotly Dash 的创始人 Chris Parmer 编写，内容全面，适合用于基本的 Dash 应用程序。在 Listing 4-4 中，我们导入了 CSS。我们还告诉 *twitter_app.py* 从网络获取 CSS 样式表并将其集成到应用程序中，然后用 <samp class="SANS_TheSansMonoCd_W5Regular_11">Dash</samp> 实例化我们的应用程序。

<samp class="SANS_TheSansMonoCd_W5Regular_11">stylesheets = ['</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">https://codepen.io/chriddyp/pen/bWLwgP.css']</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">app = Dash(__name__, external_stylesheets=stylesheets)</samp>

<samp class="SANS_Futura_Std_Book_Oblique_I_11">Listing 4-4：将样式表导入</samp> <samp class="SANS_Futura_Std_Book_11">twitter_app.py</samp>

我们的 CSS 样式表使用 CSS 类描述页面上列和行的宽度和高度。我们只需要在 Dash 代码中引用这些类，就能将 <samp class="SANS_TheSansMonoCd_W5Regular_11">Div</samp> 内容放入网格中的特定单元格。

首先，我们必须分配行，因为列应该由行包裹。为此，我们将字符串值 <samp class="SANS_TheSansMonoCd_W5Regular_11">"row"</samp> 设置为 <samp class="SANS_TheSansMonoCd_W5Regular_11">className</samp>。让我们基于 Listing 4-3 中的 <samp class="SANS_TheSansMonoCd_W5Regular_11">Div</samp> 示例进行扩展，假设此代码已导入自定义样式表；新代码部分已加粗（参见 Listing 4-5）。

<samp class="SANS_TheSansMonoCd_W5Regular_11">app.layout = html.Div([</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   html.Div(dcc.Dropdown()),</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   html.Div(dcc.Dropdown()),</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   html.Div(dcc.Dropdown()),</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">],</samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">className="row"</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">)</samp>

<samp class="SANS_Futura_Std_Book_Oblique_I_11">Listing 4-5: 带有 className 的示例 Div 代码</samp> <samp class="SANS_Futura_Std_Book_Oblique_I_11">(不是应用程序的主要部分)</samp>

在这里，我们为包含所有三个下拉框的 <samp class="SANS_TheSansMonoCd_W5Regular_11">html.Div</samp> 分配了一行，因此所有这些下拉框将显示在页面的同一行中（Figure 4-5）。<samp class="SANS_TheSansMonoCd_W5Regular_11">className</samp> 是一个可以从 CSS 样式表分配类的属性，用于告诉 Dash 如何样式化元素。在这里，我们为其分配了 <samp class="SANS_TheSansMonoCd_W5Regular_11">row</samp> 类，这告诉应用程序该 <samp class="SANS_TheSansMonoCd_W5Regular_11">Div</samp> 中的所有组件应该位于同一行中。每个 Dash 组件都有一个 <samp class="SANS_TheSansMonoCd_W5Regular_11">className</samp>，通常用于样式化和定义布局。我们使用 <samp class="SANS_TheSansMonoCd_W5Regular_11">html.Div</samp> 的 <samp class="SANS_TheSansMonoCd_W5Regular_11">className</samp> 属性来描述每个 <samp class="SANS_TheSansMonoCd_W5Regular_11">Div</samp> 的行和列布局。

定义行之后，我们需要定义列的宽度，以便 Dash 知道在该行中为每个组件分配多少列空间。我们为行中包含的每个 <samp class="SANS_TheSansMonoCd_W5Regular_11">html.Div</samp> 执行此操作，如 Listing 4-6 中加粗部分所示。

<samp class="SANS_TheSansMonoCd_W5Regular_11">app.layout =</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">html.Div([</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   html.Div(dcc.Dropdown(),</samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">className="four columns"</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">),</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   html.Div(dcc.Dropdown(),</samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">className="four columns"</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">),</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   html.Div(dcc.Dropdown(),</samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">className="four columns"</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">),</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">], className=</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">"row")</samp>

<samp class="SANS_Futura_Std_Book_Oblique_I_11">清单 4-6：设置列宽（不是主应用的一部分）</samp>

我们为每个 <samp class="SANS_TheSansMonoCd_W5Regular_11">Div</samp> 组件设置它应占用的列数空间，通过设置 <samp class="SANS_TheSansMonoCd_W5Regular_11">className</samp> 为字符串值，并格式化为 <samp class="SANS_TheSansMonoCd_W5Regular_11">"one column"</samp> 或 <samp class="SANS_TheSansMonoCd_W5Regular_11">"two columns"</samp> 等等。大多数网页的列数最多为 12 列（且行数可能是无限的），这意味着组件的列宽总和不能超过 12，所以我们在这里将它们设置为每个占 4 列。请注意，我们不需要填满所有 12 列。

图 4-5 显示了这个简单页面的显示方式。

![](img/Figure4-5.png)

<samp class="SANS_Futura_Std_Book_Oblique_I_11">图 4-5：一行三个下拉框的演示</samp>

考虑到这些，让我们看看清单 4-7，我们*twitter_app.py* 文件中的 <samp class="SANS_TheSansMonoCd_W5Regular_11">html.Div</samp> 部分，它包含不到 12 列。

<samp class="SANS_TheSansMonoCd_W5Regular_11">html.Div(</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   [</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   </samp>❶ <samp class="SANS_TheSansMonoCd_W5Regular_11">html.Div(</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">        dcc.Dropdown(</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">           id="my-dropdown",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">           multi=True,</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">           options=</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">[</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">              {"label": x, "value": x}</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">              for x in sorted(df["name"].unique())</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">           ],</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   </samp><samp class="SANS_TheSansMonoCd_W5Regular_11">        value=["taylorswift13", "cristiano", "jtimberlake"],</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">        ),</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">        className="three columns",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">     ),</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   </samp>❷ <samp class="SANS_TheSansMonoCd_W5Regular_11">html.Div(</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">        html.A(</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">           id="my-link",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">           children="点击这里访问 Twitter",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">           href="</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">https://twitter.com/explore",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">           target="_blank",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">        ),</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">        className="two columns",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">    ),</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   ],</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   className="row",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">),</samp>

<samp class="SANS_Futura_Std_Book_Oblique_I_11">清单 4-7：Dropdown</samp> <samp class="SANS_Futura_Std_Book_Oblique_I_11">部分</samp> <samp class="SANS_Futura_Std_Book_11">twitter_app.py</samp>

我们可以看到，这一行包含两个<samp class="SANS_TheSansMonoCd_W5Regular_11">Div</samp>：一个提供多个名人选择的<samp class="SANS_TheSansMonoCd_W5Regular_11">Dropdown</samp> ❶ 和一个用户点击的链接 ❷。这两个<samp class="SANS_TheSansMonoCd_W5Regular_11">Div</samp>的列数总和为五列，意味着它们在页面上是左对齐的，如图 4-6 所示。

![](img/Figure4-6.png)

<samp class="SANS_Futura_Std_Book_Oblique_I_11">图 4-6：宽度为五列的组件</samp>

请注意，一些样式表，包括我们正在使用的这个样式表，要求我们首先创建父级的<samp class="SANS_TheSansMonoCd_W5Regular_11">Div</samp>并为其指定一个行（row）。然后，在父级<samp class="SANS_TheSansMonoCd_W5Regular_11">Div</samp>的子元素中，我们定义每个内部<samp class="SANS_TheSansMonoCd_W5Regular_11">Div</samp>的列宽。

<samp class="SANS_Futura_Std_Bold_Condensed_Oblique_I_11">样式：美化你的应用</samp>

样式是赋予应用程序生命的元素。我们可以添加颜色、改变文本的字体和大小、加下划线，等等。改变应用程序样式的主要方式有两种：第一种是在 Dash HTML 组件中使用<samp class="SANS_TheSansMonoCd_W5Regular_11">style</samp>属性。这允许用户指定将直接应用于该组件的 CSS 样式声明。

第二种方法是引用一个 CSS 样式表，类似于我们创建行和列的方式。我们将向你展示如何将额外的样式表 *mystyles.css* 集成到应用中；如果你按照本章前面“设置项目”的说明下载了文件，它应该位于你的 *assets* 文件夹中。我们首先来看一下如何使用 <samp class="SANS_TheSansMonoCd_W5Regular_11">style</samp> 属性来修改应用。

<samp class="SANS_Futura_Std_Bold_Condensed_B_11">使用 style 属性</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">style</samp> 属性期望一个 Python 字典，其中的键指定我们想要修改的方面，值设置样式。在我们的 *twitter_app.py* 文件中，我们将通过在用于添加 URL 链接的 <samp class="SANS_TheSansMonoCd_W5Regular_11">html.A</samp> 组件中定义 <samp class="SANS_TheSansMonoCd_W5Regular_11">style</samp> 属性，将链接的文本颜色修改为红色，正如在 列表 4-8 中所示。

<samp class="SANS_TheSansMonoCd_W5Regular_11">html.Div(</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   html.A(id="my-link", children="点击这里访问 Twitter",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      href="</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">https://twitter.com/explore",</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">target="_blank",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   </samp>❶   <samp class="SANS_TheSansMonoCd_W5Regular_11">style={"color": "red"}),</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   className="two columns")</samp>

<samp class="SANS_Futura_Std_Book_Oblique_I_11">列表 4-8：为</samp> <samp class="SANS_Futura_Std_Book_11">twitter_app.py</samp> <samp class="SANS_Futura_Std_Book_Oblique_I_11">中的 HTML 元素设置样式</samp>

在 ❶ 处，我们将一个字典赋值给 <samp class="SANS_TheSansMonoCd_W5Regular_11">style</samp> 属性，其中键为 <samp class="SANS_TheSansMonoCd_W5Regular_11">color</samp>，值为 <samp class="SANS_TheSansMonoCd_W5Regular_11">red</samp>。这告诉浏览器将该链接的文本呈现为红色。

现在，我们通过向字典中添加另一个键值对，为同一个链接添加黄色背景色：

<samp class="SANS_TheSansMonoCd_W5Regular_11">style={"color": "red",</samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">"backgroundColor": "yellow"</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">}</samp>

请注意，字典的键是驼峰式命名的字符串。在 Dash 中，<samp class="SANS_TheSansMonoCd_W5Regular_11">style</samp> 字典中的键应该始终使用驼峰式命名。

最后，我们将把链接的字体大小修改为 40 像素：

<samp class="SANS_TheSansMonoCd_W5Regular_11">style={"color": "red", "backgroundColor": "yellow",</samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">"fontSize": "40px"</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">}</samp>

Dash 的一个美妙之处在于，样式不仅限于 HTML 组件；我们还可以为核心组件添加样式，例如 <samp class="SANS_TheSansMonoCd_W5Regular_11">Dropdown</samp>。例如，要将下拉选项的文本颜色更改为绿色，我们需要在 <samp class="SANS_TheSansMonoCd_W5Regular_11">dcc.Dropdown</samp> 中添加 <samp class="SANS_TheSansMonoCd_W5Regular_11">style</samp> 属性，正如 清单 4-9 所示。

<samp class="SANS_TheSansMonoCd_W5Regular_11">html.Div(</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   dcc.Dropdown(id="my-dropdown", multi=True,</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">           options=[{"label": x, "value": x}</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">                 for x in sorted(df["name"].unique())],</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">           value=["taylorswift13", "cristiano", "jtimberlake"],</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">           </samp><samp class="SANS_TheSansMonoCd_W7Bold_B_11">style={"color": "green"}</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">),</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   className="three columns"),</samp>

<samp class="SANS_Futura_Std_Book_Oblique_I_11">清单 4-9：在</samp> <samp class="SANS_Futura_Std_Book_11">twitter_app.py</samp> <samp class="SANS_Futura_Std_Book_Oblique_I_11">中样式化核心组件</samp>

如 图 4-7 左下角所示，下拉选项现在将显示为绿色，而不是黑色。

![](img/Figure4-7.png)

<samp class="SANS_Futura_Std_Book_Oblique_I_11">图 4-7：运行在您计算机上的下拉选项，显示为绿色</samp>

<samp class="SANS_Futura_Std_Bold_Condensed_B_11">使用样式表</samp>

样式化应用组件的第二种方法是通过元素或类定义样式。通常，当样式需要大量代码时，我们会使用这种方法。为了减少应用中样式代码的数量，我们将样式代码放在外部的 CSS 样式表中。CSS 样式表也是可重用的；您可以定义一个特定的类，并将其应用于多个组件。

我们将使用的 CSS 样式表是 *mystyles.css*，它应该已经在您与本书资源一起下载的 *assets* 文件夹中。通过双击在 PyCharm 或您首选的文本编辑器中打开该 CSS 样式表，您应该会看到以下代码：

<samp class="SANS_TheSansMonoCd_W5Regular_11">/*</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">h1 {font-size: 8.6rem; line-height: 1.35; letter-spacing: -.08rem;</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">margin-bottom: 1.2rem; margin-top: 1.2rem;}</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">*/</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">/*</samp> 是注释语法，因此，要启用样式，删除 CSS 代码上下的 <samp class="SANS_TheSansMonoCd_W5Regular_11">/*</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">*/</samp> 符号。在这里，<samp class="SANS_TheSansMonoCd_W5Regular_11">h1</samp> 是 *选择器*，它指定我们要应用样式的元素；在这种情况下，它是所有的 <samp class="SANS_TheSansMonoCd_W5Regular_11">h1</samp> 元素。在大括号内部，我们声明属性和属性值，这些属性将设置应用内的各种样式。在这个例子中，我们将元素的字体大小设置为 8.6，行高设置为 1.35，字母间距设置为 -0.08，上下边距设置为 1.2。

清单 4-10 展示了我们应用中的 <samp class="SANS_TheSansMonoCd_W5Regular_11">H1</samp> 标题组件是如何使用这个 CSS 样式表的。

<samp class="SANS_TheSansMonoCd_W5Regular_11">html.Div(html.H1("Twitter 名人点赞分析",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">                 style={"textAlign": "center"}),</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">         className="row"),</samp>

<samp class="SANS_Futura_Std_Book_Oblique_I_11">清单 4-10：在</samp> <samp class="SANS_Futura_Std_Book_11">twitter_app.py</samp> 中的 html.H1 组件

<samp class="SANS_TheSansMonoCd_W5Regular_11">html.H1</samp> 到 <samp class="SANS_TheSansMonoCd_W5Regular_11">html.H6</samp> 组件用于定义标题，其中 <samp class="SANS_TheSansMonoCd_W5Regular_11">H1</samp> 代表最高级别的标题，而 <samp class="SANS_TheSansMonoCd_W5Regular_11">H6</samp> 代表最低级别的标题。图 4-8 展示了这种标题样式的效果。

![](img/Figure4-8.png)

<samp class="SANS_Futura_Std_Book_Oblique_I_11">图 4-8：使用 CSS 样式的应用标题</samp>

如你所见，如果将 图 4-8 与 图 4-6 进行对比，你会发现应用标题的字体大小变大了，标题上下边距增多，字母间距减少。如果应用的标题没有改变大小，请重启应用以查看效果。

如果你想将标题的字体大小恢复到较小的尺寸，只需通过重新插入 <samp class="SANS_TheSansMonoCd_W5Regular_11">/*</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">*/</samp> 符号来注释掉 CSS 代码，如下所示：

<samp class="SANS_TheSansMonoCd_W5Regular_11">/*</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">h1 {font-size: 8.6rem; line-height: 1.35; letter-spacing: -.08rem;</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">margin-bottom: 1.2rem; margin-top: 1.2rem;}</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">*/</samp>

你已经学会了如何通过纯 Python 操作应用程序的样式和布局。然而，这仅仅是开始。在第五章中，我们将深入探讨 dash-bootstrap-components，它将使仪表板应用程序的布局设计和样式变得更加简单和多样化。

### <samp class="SANS_Futura_Std_Bold_B_11">Dash 组件</samp>

在这里，我们将概述 Dash 中一些常用的组件，这些组件由 dash-html-components 和 dash-core-components 库提供。还有许多其他组件库，甚至可以编写自己的组件！但是 dash-html-components 和 dash-core-components 包含了我们所需的大部分基本功能。HTML 组件通常用于构建网页布局，包括 <samp class="SANS_TheSansMonoCd_W5Regular_11">Div</samp>、<samp class="SANS_TheSansMonoCd_W5Regular_11">Button</samp>、<samp class="SANS_TheSansMonoCd_W5Regular_11">H1</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">Form</samp> 等组件。Core 组件——如 <samp class="SANS_TheSansMonoCd_W5Regular_11">Dropdown</samp>、<samp class="SANS_TheSansMonoCd_W5Regular_11">Checklist</samp>、<samp class="SANS_TheSansMonoCd_W5Regular_11">RangeSlider</samp> 等——用于创建交互式体验。所有 HTML 和 Core 组件都有属性，可以增强其功能。有关这些属性及其组件的完整列表，请访问 Dash 文档中有关 HTML 和 Core 组件的页面：[*https://<wbr>dash<wbr>.plotly<wbr>.com<wbr>/dash<wbr>-core<wbr>-components*](https://dash.plotly.com/dash-core-components).

<samp class="SANS_Futura_Std_Bold_Condensed_Oblique_I_11">HTML 组件</samp>

Dash HTML 组件是用 Python 编写的，并会自动转换为 HTML，因此使用 Dash 应用程序时无需成为 HTML 或 CSS 的专家。以下这行 Python 代码

<samp class="SANS_TheSansMonoCd_W5Regular_11"><h1> 名人推特点赞分析 </h1></samp>

大致等同于以下这行由网页浏览器读取的 HTML：

<samp class="SANS_TheSansMonoCd_W5Regular_11">html.H1("名人推特点赞分析")</samp>

现在可以用纯 Python 编写完整的仪表板应用程序：Python 永远！

要创建一个 HTML 组件，你需要使用点符号（dot notation）来连接 <samp class="SANS_TheSansMonoCd_W5Regular_11">html</samp> 关键字和组件名称。例如，对于 <samp class="SANS_TheSansMonoCd_W5Regular_11">Div</samp> 组件，你会使用 <samp class="SANS_TheSansMonoCd_W5Regular_11">html.Div</samp>，正如我们之前所见。我们还看到了两个额外的 HTML 组件：<samp class="SANS_TheSansMonoCd_W5Regular_11">html.H1</samp>，它创建一个顶级标题，和 <samp class="SANS_TheSansMonoCd_W5Regular_11">html.A</samp>，它创建一个超链接。让我们更仔细地看看如何使用 <samp class="SANS_TheSansMonoCd_W5Regular_11">html.H1</samp> 来表示页面的标题，标题本身以字符串的形式编写，如下所示：

<samp class="SANS_TheSansMonoCd_W5Regular_11">html.H1("Twitter 喜欢分析：名人")</samp>

这将字符串赋值给 <samp class="SANS_TheSansMonoCd_W5Regular_11">children</samp> 属性，这通常是接受子组件的任何组件的第一个位置参数。在这个上下文中，<samp class="SANS_TheSansMonoCd_W5Regular_11">children</samp> 是一个属性，它将一个组件或元素（例如文本标签）放置在另一个组件内。完整地写出，前一行看起来像这样：

<samp class="SANS_TheSansMonoCd_W5Regular_11">html.H1(children="Twitter 喜欢分析：名人")</samp>

在以下代码的前三个示例中，<samp class="SANS_TheSansMonoCd_W5Regular_11">children</samp> 属性向页面添加文本。在最后一个示例中，使用 <samp class="SANS_TheSansMonoCd_W5Regular_11">html.Div</samp> 时，<samp class="SANS_TheSansMonoCd_W5Regular_11">children</samp> 属性将 <samp class="SANS_TheSansMonoCd_W5Regular_11">html.H1</samp> 组件添加到页面中，该组件也包含文本。<samp class="SANS_TheSansMonoCd_W5Regular_11">children</samp> 属性可以接受整数、字符串、Dash 组件或这些的列表。所有这些示例都是可能的：

<samp class="SANS_TheSansMonoCd_W5Regular_11">html.H1(children=2),</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">html.H1(children=["Twitter 喜欢分析：名人"]),</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">html.H1(children=["Twitter 喜欢分析：名人"]),</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">html.Div(children=</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">[</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   html.H1("Twitter 喜欢分析：名人"),</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   html.H2("Twitter 喜欢分析：名人")</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">])</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">html.A</samp> 组件创建了一个 <samp class="SANS_TheSansMonoCd_W5Regular_11"><a></samp> HTML5 元素，用于创建超链接。在此组件中，如 清单 4-11 所示，我们使用了四个属性：<samp class="SANS_TheSansMonoCd_W5Regular_11">id</samp>、<samp class="SANS_TheSansMonoCd_W5Regular_11">children</samp>、<samp class="SANS_TheSansMonoCd_W5Regular_11">href</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">target</samp>。

<samp class="SANS_TheSansMonoCd_W5Regular_11">html.A(id="my-link", children=</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">"点击这里访问 Twitter",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   href="</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">https://twitter.com/explore",</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">target="_blank")</samp>

<samp class="SANS_Futura_Std_Book_Oblique_I_11">清单 4-11：HTML 链接组件在</samp> <samp class="SANS_Futura_Std_Book_11">twitter_app.py</samp>

我们赋值给 <samp class="SANS_TheSansMonoCd_W5Regular_11">href</samp> 的值是完整的链接目标地址，用户点击链接后将跳转到该位置。<samp class="SANS_TheSansMonoCd_W5Regular_11">target</samp> 属性指示链接将在何处打开：如果其赋值为 <samp class="SANS_TheSansMonoCd_W5Regular_11">_self</samp>，链接将在当前浏览器标签页中打开；如果赋值为 <samp class="SANS_TheSansMonoCd_W5Regular_11">_blank</samp>，链接将在新的浏览器标签页中打开。<samp class="SANS_TheSansMonoCd_W5Regular_11">children</samp> 属性定义了组件的内容，在这里是一个表示链接文本的字符串，用户在页面上看到的就是这些文本。

<samp class="SANS_TheSansMonoCd_W5Regular_11">id</samp> 属性非常重要，因为 Dash 组件使用 <samp class="SANS_TheSansMonoCd_W5Regular_11">id</samp> 来相互识别和交互，这赋予了仪表板应用其交互能力。稍后在本章的“回调装饰器”部分，我们将详细介绍这一点。目前，只需注意，赋值给 <samp class="SANS_TheSansMonoCd_W5Regular_11">id</samp> 的值必须是唯一的字符串，以便用于识别该组件。

<samp class="SANS_Futura_Std_Bold_Condensed_Oblique_I_11">核心组件</samp>

Dash 核心组件是 Dash 库中的预构建组件，允许用户以直观的方式与应用程序进行交互。在这个应用程序中，我们使用了两个核心组件：<samp class="SANS_TheSansMonoCd_W5Regular_11">Graph</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">Dropdown</samp>。要构建或访问特定的核心组件，我们使用 <samp class="SANS_TheSansMonoCd_W5Regular_11">dcc</samp> 关键字并使用点号语法来指定组件名称，例如 <samp class="SANS_TheSansMonoCd_W5Regular_11">dcc.Dropdown</samp>。

<samp class="SANS_Futura_Std_Bold_Condensed_B_11">Graph 组件</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">Graph</samp> 组件允许你将数据可视化图表、图形和图像以 Plotly 的形式嵌入到你的应用中。它是核心组件中最受欢迎的之一，你很可能会在每个分析仪表板应用中看到它。

一个 <samp class="SANS_TheSansMonoCd_W5Regular_11">Graph</samp> 组件有两个主要的属性：<samp class="SANS_TheSansMonoCd_W5Regular_11">id</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">figure</samp>。以下是定义 <samp class="SANS_TheSansMonoCd_W5Regular_11">Graph</samp> 组件的模板：

<samp class="SANS_TheSansMonoCd_W5Regular_11">dcc.Graph(id="line-chart", figure={})</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">id</samp> 属性为 <samp class="SANS_TheSansMonoCd_W5Regular_11">Graph</samp> 组件提供了一个唯一的 ID。<samp class="SANS_TheSansMonoCd_W5Regular_11">figure</samp> 属性是 Plotly 图表的占位符。一旦创建了 Plotly 图表，我们会将其分配给 <samp class="SANS_TheSansMonoCd_W5Regular_11">figure</samp> 属性，替代空字典。例如，在我们的应用中，我们使用 列表 4-12 中的代码创建了一个 Plotly 线图。

<samp class="SANS_TheSansMonoCd_W5Regular_11">import plotly.express as px</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">--</samp><samp class="SANS_TheSansMonoCd_W5Regular_Italic_I_11">省略--</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">fig = px.line(data_frame=df_filtered, x="date_time", y="number_of_likes",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   color="name", log_y=True)</samp>

<samp class="SANS_Futura_Std_Book_Oblique_I_11">列表 4-12：在</samp> <samp class="SANS_Futura_Std_Book_11">twitter_app.py</samp> 中创建 Plotly 图表

我们将在本章的“Plotly Express 线图”部分详细介绍 Plotly 图表。现在，这一行代码简单描述了图表的外观，并将其分配给 <samp class="SANS_TheSansMonoCd_W5Regular_11">fig</samp> 对象，从而将其转化为一个 Plotly 图形。然后，我们可以将 <samp class="SANS_TheSansMonoCd_W5Regular_11">fig</samp> 插入到 <samp class="SANS_TheSansMonoCd_W5Regular_11">dcc.Graph</samp> 的 <samp class="SANS_TheSansMonoCd_W5Regular_11">figure</samp> 属性中，以在页面上显示线图。列表 4-13 显示了来自 *twitter_app.py* 文件的代码，它就是这样做的，并将图表分配给 <samp class="SANS_TheSansMonoCd_W5Regular_11">app.layout</samp>。

<samp class="SANS_TheSansMonoCd_W5Regular_11">html.Div(dcc.Graph(id="line-chart", figure=fig), className="row")</samp>

<samp class="SANS_Futura_Std_Book_Oblique_I_11">列表 4-13：将图表引入 Graph 组件中，位于</samp> <samp class="SANS_Futura_Std_Book_11">twitter_app.py</samp>

我们将 <samp class="SANS_TheSansMonoCd_W5Regular_11">Graph</samp> 组件放入 <samp class="SANS_TheSansMonoCd_W5Regular_11">Div</samp> 组件中，并将其分配给页面上的单个行。一旦完整的应用脚本激活，折线图应该会显示在页面上。

要观看关于 Dash <samp class="SANS_TheSansMonoCd_W5Regular_11">Graph</samp> 组件及其用法的完整视频教程，请访问视频 “关于 Graph 组件的一切” 在 [*https://<wbr>learnplotlydash<wbr>.com*](https://learnplotlydash.com)。

<samp class="SANS_Futura_Std_Bold_Condensed_B_11">Dropdown 组件</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">Dropdown</samp> 组件允许用户从下拉菜单中选择选项，以动态过滤数据并更新图表。我们通过为四个属性提供值来定义 <samp class="SANS_TheSansMonoCd_W5Regular_11">Dropdown</samp> 组件：<samp class="SANS_TheSansMonoCd_W5Regular_11">id</samp>、<samp class="SANS_TheSansMonoCd_W5Regular_11">multi</samp>、<samp class="SANS_TheSansMonoCd_W5Regular_11">options</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">value</samp>，如 列表 4-14 所示。此菜单显示在 图 4-9 中。

<samp class="SANS_TheSansMonoCd_W5Regular_11">dcc.Dropdown(id="my-dropdown", multi=True,</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">              options=[{"label": x, "value": x}</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">                 for x in sorted(df["name"].unique())],</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">              value=["taylorswift13", "cristiano", "jtimberlake"])</samp>

<samp class="SANS_Futura_Std_Book_Oblique_I_11">列表 4-14：在</samp> <samp class="SANS_Futura_Std_Book_11">twitter_app.py</samp> <samp class="SANS_Futura_Std_Book_Oblique_I_11">中创建一个 Dropdown 组件</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">multi</samp> 属性允许我们选择用户是否可以一次选择多个值或一次只能选择一个值。当此属性设置为 <samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp> 时，应用用户可以选择多个值。当设置为 <samp class="SANS_TheSansMonoCd_W5Regular_11">False</samp> 时，应用用户一次只能选择一个值。

<samp class="SANS_TheSansMonoCd_W5Regular_11">options</samp> 属性表示用户在点击 <samp class="SANS_TheSansMonoCd_W5Regular_11">Dropdown</samp> 时可以选择的值。我们为其分配一个字典列表，每个字典包含 <samp class="SANS_TheSansMonoCd_W5Regular_11">label</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">value</samp> 键，其中每个字典表示一个菜单选项。<samp class="SANS_TheSansMonoCd_W5Regular_11">label</samp> 是用户看到的选项名称，而 <samp class="SANS_TheSansMonoCd_W5Regular_11">value</samp> 是应用程序读取的实际数据。

![](img/Figure4-9.png)

<samp class="SANS_Futura_Std_Book_Oblique_I_11">图 4-9：应用程序下拉选项</samp>

在清单 4-14 中，我们使用列表推导为我们的 pandas DataFrame 的<samp class="SANS_TheSansMonoCd_W5Regular_11">name</samp>列中的每个唯一值创建一个包含<samp class="SANS_TheSansMonoCd_W5Regular_11">label</samp>和<samp class="SANS_TheSansMonoCd_W5Regular_11">value</samp>键的字典。

如果我们只有几个值，可能更容易逐个编写每个字典，而不是使用列表推导。例如，在清单 4-15 中，我们构建了一个仅包含两个值的<samp class="SANS_TheSansMonoCd_W5Regular_11">Dropdown</samp>：<samp class="SANS_TheSansMonoCd_W5Regular_11">taylorswift13</samp>和<samp class="SANS_TheSansMonoCd_W5Regular_11">cristiano</samp>。

<samp class="SANS_TheSansMonoCd_W5Regular_11">dcc.Dropdown(id="my-dropdown", multi=True,</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">             options=[{"label": "Taylor", "value": "taylorswift13"},</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">                      {"label": "Ronaldo", "value": "cristiano"}]</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">)</samp>

<samp class="SANS_Futura_Std_Book_Oblique_I_11">清单 4-15：不在</samp> <samp class="SANS_Futura_Std_Book_11">twitter_app.py</samp> 的一个下拉示例

在这里，我们使用 DataFrame 中的值，以便更容易进行过滤。但是，我们可以选择一种用户友好的表示方式，将<samp class="SANS_TheSansMonoCd_W5Regular_11">label</samp>键使其对用户更易识别。当用户点击下拉框时，他们将看到两个选项*Taylor*和*Ronaldo*，这些选项分别被应用程序读取为<samp class="SANS_TheSansMonoCd_W5Regular_11">taylorswift13</samp>和<samp class="SANS_TheSansMonoCd_W5Regular_11">cristiano</samp>。

最后一个<samp class="SANS_TheSansMonoCd_W5Regular_11">Dropdown</samp>属性是<samp class="SANS_TheSansMonoCd_W5Regular_11">value</samp>（不要与字典中的<samp class="SANS_TheSansMonoCd_W5Regular_11">value</samp>键混淆），它包含了用户启动应用程序时<samp class="SANS_TheSansMonoCd_W5Regular_11">Dropdown</samp>将采用的默认值。由于我们有一个多值<samp class="SANS_TheSansMonoCd_W5Regular_11">Dropdown</samp>，我们使用来自 DataFrame 的<samp class="SANS_TheSansMonoCd_W5Regular_11">name</samp>列的三个字符串作为初始值：<samp class="SANS_TheSansMonoCd_W5Regular_11">taylorswift13</samp>，<samp class="SANS_TheSansMonoCd_W5Regular_11">cristiano</samp>和<samp class="SANS_TheSansMonoCd_W5Regular_11">jtimberlake</samp>。

这些字符串对应于在列表 4-14 中`options`属性生成的值。这些字符串是预加载的，因此在用户点击下拉菜单之前，这三个值会自动被选择。一旦用户在下拉菜单中选择了不同的值，这些值也会相应地变化。

关于 Dash <samp class="SANS_TheSansMonoCd_W5Regular_11">Dropdown</samp> 组件及其用法的完整视频教程，请查看视频“Dropdown 选择器”在[*https://<wbr>learnplotlydash<wbr>.com*](https://learnplotlydash.com)。

### <samp class="SANS_Futura_Std_Bold_B_11">Dash 回调</samp>

一个 *Dash 回调* 使用户能够在仪表板应用中进行交互；它是将 Dash 组件彼此连接的机制，以便执行某个操作会导致其他事情发生。当应用用户选择下拉值时，图表会更新；当用户点击按钮时，应用标题的颜色会改变或另一个图表会被添加到页面。Dash 组件之间的可能交互是无限的，没有回调，应用程序将是静态的，用户无法修改任何内容。

Dash 回调有两个部分，一个是回调装饰器，用于识别相关组件，该部分在布局部分定义：

<samp class="SANS_TheSansMonoCd_W5Regular_11">@app.callback()</samp>

和回调函数，它定义了 Dash 组件之间如何交互：

<samp class="SANS_TheSansMonoCd_W5Regular_11">def function_name(y):</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   return x</samp>

这个简单的应用程序只有一个回调，虽然更复杂的应用程序会有很多个回调。

<samp class="SANS_Futura_Std_Bold_Condensed_Oblique_I_11">回调装饰器</samp>

回调装饰器将回调函数注册到你的 Dash 应用中，告诉它何时调用该函数，以及如何使用函数的返回值来更新应用程序。（我们在第一章中讨论了装饰器。）

回调装饰器应该放置在回调函数的正上方，装饰器和函数之间不能有空格。装饰器有两个主要参数：<samp class="SANS_TheSansMonoCd_W5Regular_11">Output</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">Input</samp>，它们分别表示在用户对另一个组件进行操作时，应该改变的组件（<samp class="SANS_TheSansMonoCd_W5Regular_11">Output</samp>）和用户操作的组件（<samp class="SANS_TheSansMonoCd_W5Regular_11">Input</samp>）。例如，输出可能是折线图，其变化应根据用户在 <samp class="SANS_TheSansMonoCd_W5Regular_11">Dropdown</samp> 组件中的输入而变化，如列表 4-16 所示。

<samp class="SANS_TheSansMonoCd_W5Regular_11">@app.callback(</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   Output(component_id="line-chart", component_property="figure"),</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   [Input(component_id="my-dropdown", component_property="value")],</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">)</samp>

<samp class="SANS_Futura_Std_Book_Oblique_I_11">Listing 4-16: 来自</samp> <samp class="SANS_Futura_Std_Book_11">twitter_app.py</samp> 的回调装饰器

<samp class="SANS_TheSansMonoCd_W5Regular_11">Output</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">Input</samp> 都接受两个参数：<samp class="SANS_TheSansMonoCd_W5Regular_11">component_id</samp>，它应该对应于特定 Dash 组件的 <samp class="SANS_TheSansMonoCd_W5Regular_11">id</samp>，以及 <samp class="SANS_TheSansMonoCd_W5Regular_11">component_property</samp>，它应该对应于该组件的特定属性。在 Listing 4-16 中，<samp class="SANS_TheSansMonoCd_W5Regular_11">Input</samp> 的 <samp class="SANS_TheSansMonoCd_W5Regular_11">component_id</samp> 指的是我们之前定义的 <samp class="SANS_TheSansMonoCd_W5Regular_11">my-dropdown Dropdown</samp>。<samp class="SANS_TheSansMonoCd_W5Regular_11">component_property</samp> 具体指的是 <samp class="SANS_TheSansMonoCd_W5Regular_11">my-dropdown</samp> 的 <samp class="SANS_TheSansMonoCd_W5Regular_11">value</samp> 属性，它表示要显示的 Twitter 用户数据，初始值为 <samp class="SANS_TheSansMonoCd_W5Regular_11">["taylorswift13", "cristiano", "jtimberlake"]</samp>，如 Listing 4-14 中所示。

在 <samp class="SANS_TheSansMonoCd_W5Regular_11">Output</samp> 中，我们引用了 <samp class="SANS_TheSansMonoCd_W5Regular_11">dcc.Graph</samp> 的 <samp class="SANS_TheSansMonoCd_W5Regular_11">figure</samp> 属性，我们在布局中也之前定义了它，如 Listing 4-17 中所示。

<samp class="SANS_TheSansMonoCd_W5Regular_11">dcc.Graph(id="line-chart", figure={})</samp>

<samp class="SANS_Futura_Std_Book_Oblique_I_11">Listing 4-17: 在</samp> <samp class="SANS_Futura_Std_Book_11">twitter_app.py</samp> 中的布局部分的 Graph 组件

在这里，<samp class="SANS_TheSansMonoCd_W5Regular_11">figure</samp> 属性目前是一个空字典，因为回调函数将根据输入值创建一个折线图并将其分配给 <samp class="SANS_TheSansMonoCd_W5Regular_11">figure</samp>。让我们深入了解回调函数，全面理解这一过程是如何发生的。

<samp class="SANS_Futura_Std_Bold_Condensed_Oblique_I_11">回调函数</samp>

我们应用的回调函数名为 <samp class="SANS_TheSansMonoCd_W5Regular_11">update_graph()</samp>，它包含一系列的 <samp class="SANS_TheSansMonoCd_W5Regular_11">if-else</samp> 语句，用于筛选 DataFrame <samp class="SANS_TheSansMonoCd_W5Regular_11">df</samp> 并根据选择的输入值创建折线图。Listing 4-18 显示了我们应用中的回调函数。

<samp class="SANS_TheSansMonoCd_W5Regular_11">def update_graph(chosen_value):</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   print(f"用户选择的值: {chosen_value}")</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   if len(chosen_value) == 0:</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      return {}</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      else:</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">     df_filtered = df[df["name"].isin(chosen_value)]</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">     fig = px.line(</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">        data_frame=df_filtered,</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">        x="date_time",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">        y="number_of_likes",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">        color="name",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">        log_y=</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">True,</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">        labels={</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">           "number_of_likes": "点赞数",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">           "date_time": "日期",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">           "name": "名人",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">        },</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      )</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      return fig</samp>

<samp class="SANS_Futura_Std_Book_Oblique_I_11">清单 4-18: twitter_app.py 中的回调函数</samp> <samp class="SANS_Futura_Std_Book_11">twitter_app.py</samp>

我们稍后会逐行讲解这里的逻辑。但首先，让我们讨论一下这个函数的作用。执行时，<samp class="SANS_TheSansMonoCd_W5Regular_11">update_graph()</samp> 会返回一个名为 <samp class="SANS_TheSansMonoCd_W5Regular_11">fig</samp> 的对象，在本例中，它包含了 Plotly Express 的折线图。对象 <samp class="SANS_TheSansMonoCd_W5Regular_11">fig</samp> 被返回到我们在回调装饰器中指定的组件和属性的 <samp class="SANS_TheSansMonoCd_W5Regular_11">Output</samp> 中。正如我们所知道的，回调装饰器引用了布局中的 Dash 组件。这里，<samp class="SANS_TheSansMonoCd_W5Regular_11">fig</samp> 被赋值给布局部分中的 <samp class="SANS_TheSansMonoCd_W5Regular_11">Graph</samp> 组件的 <samp class="SANS_TheSansMonoCd_W5Regular_11">figure</samp> 属性，因此回调函数告诉应用程序显示折线图。以下是回调函数 <samp class="SANS_TheSansMonoCd_W5Regular_11">update_graph()</samp> 执行后，<samp class="SANS_TheSansMonoCd_W5Regular_11">Graph</samp> 组件的样子：

<samp class="SANS_TheSansMonoCd_W5Regular_11">dcc.Graph(id="line-chart", figure=fig)</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">figure</samp> 属性现在被分配给了对象 <samp class="SANS_TheSansMonoCd_W5Regular_11">fig</samp>，而不是我们在 F 显示 4-17 中看到的空字典。

我们将总结这一点，因为这个过程非常重要！一旦用户输入激活回调函数，它会返回一个与回调装饰器中<sup><samp class="SANS_TheSansMonoCd_W5Regular_11">Output</samp></sup> 的<sup><samp class="SANS_TheSansMonoCd_W5Regular_11">component_property</samp></sup>相关联的对象。考虑到组件属性代表应用布局中某个组件的实际属性，结果就是一个通过用户交互不断更新的应用。

想了解 Dash 回调装饰器及其使用的完整视频教程，请参阅视频“Dash 回调——输入、输出、状态等”：[*https://<wbr>learnplotlydash<wbr>.com*](https://learnplotlydash.com)。

<samp class="SANS_Futura_Std_Bold_Condensed_B_11">激活回调</samp>

要激活回调，用户必须与回调装饰器中指定的 <samp class="SANS_TheSansMonoCd_W5Regular_11">Input</samp> 组件进行交互。在这个应用中，组件的属性表示 <samp class="SANS_TheSansMonoCd_W5Regular_11">Dropdown</samp> 的值，因此每当应用用户选择不同的下拉值（如 Twitter 账号）时，回调函数就会被触发。

如果回调装饰器有三个 <samp class="SANS_TheSansMonoCd_W5Regular_11">Input</samp>，用户需要提供三个参数来触发回调函数。而在我们的例子中，回调装饰器只有一个 <samp class="SANS_TheSansMonoCd_W5Regular_11">Input</samp>；因此，回调函数只需要一个参数：<samp class="SANS_TheSansMonoCd_W5Regular_11">chosen_value</samp>。

<samp class="SANS_Futura_Std_Bold_Condensed_B_11">函数如何工作</samp>

让我们查看一下 F 显示 4-19，它展示了应用回调函数内部发生的情况。

❶  <samp class="SANS_TheSansMonoCd_W5Regular_11">def update_graph(chosen_value):</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">        print(f"用户选择的值：{chosen_value}")</samp>

❷ <samp class="SANS_TheSansMonoCd_W5Regular_11">if len(chosen_value) == 0:</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">     return {}</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   else:</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      df_filtered = df[df["name"].isin(chosen_value)]</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      fig =</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">px.line(</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">        data_frame=df_filtered,</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">        x="date_time",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">        y="number_of_likes",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">        color="name",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">        log_y=True,</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">        labels={</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">           "number_of_likes": "点赞数",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">           "date_time": "日期时间",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">           "name": "名人",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">        },</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      )</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      return fig</samp>

<samp class="SANS_Futura_Std_Book_Oblique_I_11">清单 4-19: 回调函数用于</samp> <samp class="SANS_Futura_Std_Book_11">twitter_app.py</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">chosen_value</samp> 参数 ❶ 指的是 <samp class="SANS_TheSansMonoCd_W5Regular_11">dcc.Dropdown</samp> 的值，这是一个 Twitter 用户名的列表。每当用户选择新选项时，函数会被激活。用户可以选择任意数量的可用名人，<samp class="SANS_TheSansMonoCd_W5Regular_11">chosen_value</samp> 列表中的项目数量将相应增加或减少。它可能是一个包含 3 个值、10 个值，甚至没有值的列表。因此，我们检查 <samp class="SANS_TheSansMonoCd_W5Regular_11">chosen_value</samp> 列表的长度 ❷。如果它等于零，意味着是一个空列表，函数将返回一个空字典，并且返回的 <samp class="SANS_TheSansMonoCd_W5Regular_11">fig</samp> 对象会显示一个空图表。

如果 <samp class="SANS_TheSansMonoCd_W5Regular_11">chosen_value</samp> 列表的长度不等于零，在 <samp class="SANS_TheSansMonoCd_W5Regular_11">else</samp> 分支中，我们使用 pandas 过滤 DataFrame，只保留包含选定 Twitter 用户名的行。过滤后的 DataFrame 被保存为 <samp class="SANS_TheSansMonoCd_W5Regular_11">df_filtered</samp>，然后用作创建折线图的数据，该图被保存为 <samp class="SANS_TheSansMonoCd_W5Regular_11">fig</samp> 对象。<samp class="SANS_TheSansMonoCd_W5Regular_11">fig</samp> 对象被返回，用于在应用页面上显示折线图。

关于这些函数的一个重要提示：如果原始的 DataFrame 被以任何方式修改，你应该始终创建原始 DataFrame 的副本，正如我们在创建<samp class="SANS_TheSansMonoCd_W5Regular_11">df_filtered</samp>时所做的那样。应用开始时定义的原始 DataFrame，在清单 4-2 中，是被视为全局变量的。全局变量不应被修改，因为修改它会影响到应用中其他用户所看到的变量。例如，如果某个用户在一个财务仪表板应用中修改了全局变量<samp class="SANS_TheSansMonoCd_W5Regular_11">price_values</samp>，所有用户都将看到这些变化的价格。这可能会造成严重的损害和混乱。

<samp class="SANS_Futura_Std_Bold_Condensed_Oblique_I_11">回调图</samp>

Dash 有一个强大的回调图工具，可以显示回调的结构，并清晰地描述元素之间的联系。在定义回调时，特别是当回调具有多个<samp class="SANS_TheSansMonoCd_W5Regular_11">输入</samp>和<samp class="SANS_TheSansMonoCd_W5Regular_11">输出</samp>时，使用此工具非常有帮助，因为这时很难掌握回调结构。要打开回调图，请点击应用页面右下角的蓝色按钮，如图 4-10 所示。

![](img/Figure4-10.png)

<samp class="SANS_Futura_Std_Book_Oblique_I_11">图 4-10：点击右下角的按钮以打开菜单。</samp>

然后点击灰色的**回调**按钮，如图 4-11 所示。

![](img/Figure4-11.png)

<samp class="SANS_Futura_Std_Book_Oblique_I_11">图 4-11：点击回调按钮查看回调图。</samp>

结果应如图 4-12 所示。

![](img/Figure4-12.png)

<samp class="SANS_Futura_Std_Book_Oblique_I_11">图 4-12：回调图展示了</samp> <samp class="SANS_Futura_Std_Book_11">twitter_app.py</samp>

左侧的元素是<samp class="SANS_TheSansMonoCd_W5Regular_11">输入</samp>的组件属性。中间的元素描述了回调在本次会话中被触发的次数（在本例中为一次），以及回调完全执行所需的时间（614 毫秒）。右侧的元素是<samp class="SANS_TheSansMonoCd_W5Regular_11">输出</samp>的组件属性。该图帮助清晰地展示了<samp class="SANS_TheSansMonoCd_W5Regular_11">下拉</samp>值（<samp class="SANS_TheSansMonoCd_W5Regular_11">输入</samp>）如何影响折线图的图形（<samp class="SANS_TheSansMonoCd_W5Regular_11">输出</samp>）。

继续通过更改主应用页面上的 <samp class="SANS_TheSansMonoCd_W5Regular_11">Dropdown</samp> 名人名字来触发回调。你看到中间的绿色元素发生变化了吗？通过点击左侧和右侧的元素来探索这个图表；你应该会看到每个元素中的额外信息。

在将应用部署到网络之前，确保通过将 <samp class="SANS_TheSansMonoCd_W5Regular_11">debug</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">=</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">False</samp> 关闭调试模式，以便关闭图表。否则，最终用户也可以访问该图表。

### <samp class="SANS_Futura_Std_Bold_B_11">Plotly Express 折线图</samp>

在这里，我们将回顾如何创建 Plotly 图表。我们将专注于折线图，因为这是我们在这个应用中使用的图表类型，其他类型的图表将在未来的章节中回顾。

Plotly Express 是一个用于快速直观创建图表的高级接口。它包含了数十种图表类型供选择，涵盖了从科学、统计、金融图表到 3D 图表和地图等各种图表。每个图表都有许多属性，允许你根据用户的需求自定义图表。以下是适用于 Plotly Express 折线图的所有属性的完整列表，目前都设置为 <samp class="SANS_TheSansMonoCd_W5Regular_11">None</samp>：

<samp class="SANS_TheSansMonoCd_W5Regular_11">plotly.express.line(</samp><samp class="SANS_TheSansMonoCd_W7Bold_B_11">data_frame=None, x</samp><samp class="SANS_TheSansMonoCd_W7Bold_B_11">=None, y=None</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">, line_group=None, color=None, line_dash=None, hover_name=None, hover_data=None, custom_data=</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">None, text=None, facet_row=None, facet_col=None, facet_col_wrap=0, facet_row_spacing=None, facet_col_spacing=None, error_x=None, error_x_minus=None, error_y=None, error_y_minus=</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">None, animation_frame=None, animation_group=None, category_orders={}, labels={}, orientation=None, color_discrete_sequence=None, color_discrete_map={}, line_dash_sequence=None, line_dash_map={}, log_x=</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">False, log_y=False, range_x=None, range_y=None, line_shape=None, render_mode='auto', title=None, template=None,width=None, height=None)</samp>

Plotly Express 的美妙之处在于，在大多数情况下，创建图表所需了解的仅仅是前三个属性：<samp class="SANS_TheSansMonoCd_W5Regular_11">data_frame</samp>、<samp class="SANS_TheSansMonoCd_W5Regular_11">x</samp> 和 <samp class="SANS_TheSansMonoCd_W5Regular_11">y</samp>，在示例中以粗体显示。这些分别表示数据框、用于 x 轴的数据列和用于 y 轴的数据列。下面是我们绘制的一个非常简单的折线图：

<samp class="SANS_TheSansMonoCd_W5Regular_11">import plotly.express as px</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">px.line(data_frame=df, x="some_xaxis_data", y="some_yaxis_data")</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">fig.show()</samp>

这创建了最基本的折线图，绘制了两列数据之间的关系，类似于图 4-13。

![](img/Figure4-13.png)

<samp class="SANS_Futura_Std_Book_Oblique_I_11">图 4-13：最简单的折线图</samp>

随着你越来越熟悉 Plotly Express，你会发现自己不断向图表添加更多的属性。例如，为了通过颜色区分不同的数据组，我们添加了<samp class="SANS_TheSansMonoCd_W5Regular_11">color</samp>属性，并将其分配给假设 DataFrame 中的一列：

<samp class="SANS_TheSansMonoCd_W5Regular_11">px.line(data_frame=df, x="some_ xaxis _data", y="some_yaxis_data",</samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">color</samp><samp class="SANS_TheSansMonoCd_W7Bold_B_11">="some_data"</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">)</samp>

结果，我们会看到类似于图 4-14 的内容。

![](img/Figure4-14.png)

<samp class="SANS_Futura_Std_Book_Oblique_I_11">图 4-14：向简单图表添加颜色属性</samp>

要改变图形的高度，我们添加了<samp class="SANS_TheSansMonoCd_W5Regular_11">height</samp>属性，并为其分配了一个像素值：

<samp class="SANS_TheSansMonoCd_W5Regular_11">px.line(data_frame=df, x="some_xaxis_data", y='some_yaxis_data',</samp> <samp class="SANS_TheSansMonoCd_W7Bold_B_11">height=300</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">)</samp>

这里我们将整个图形的高度设置为 300 像素。

在我们的 Twitter 点赞分析应用中，折线图包括了<data_frame>、<samp class="SANS_TheSansMonoCd_W5Regular_11">x</samp>、<samp class="SANS_TheSansMonoCd_W5Regular_11">y</samp>、<samp class="SANS_TheSansMonoCd_W5Regular_11">color</samp>属性，以及<samp class="SANS_TheSansMonoCd_W5Regular_11">labels</samp>和<samp class="SANS_TheSansMonoCd_W5Regular_11">log_y</samp>属性。清单 4-20 展示了我们的 Plotly 图表代码。

<samp class="SANS_TheSansMonoCd_W5Regular_11">fig = px.line(</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   data_frame=df_filtered,</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   x="date_time",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   y=</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">"number_of_likes",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   color="name",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   log_y=True,</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   labels={</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      "number_of_likes": "Likes",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      "date_time": "Date",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">      "name": "Celebrity",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">   },</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">)</samp>

<samp class="SANS_Futura_Std_Book_Oblique_I_11">清单 4-20：Plotly 折线图用于</samp> <samp class="SANS_Futura_Std_Book_11">twitter_app.py</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">log_y</samp> 属性告诉应用使用对数刻度来显示 y 轴数据。当图表中的一些数据点比大部分数据点要大或小得多时，推荐使用对数刻度，因为它可以提供更清晰的可视化。我们不会在这里详细讨论对数刻度，但可以尝试将该属性从 <samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp> 更改为 <samp class="SANS_TheSansMonoCd_W5Regular_11">False</samp>，然后刷新应用查看更新后的图表。你更喜欢哪一种？

`<samp class="SANS_TheSansMonoCd_W5Regular_11">labels</samp>` 属性更改了应用用户所看到的轴标签。用于绘制折线图的三列是 <samp class="SANS_TheSansMonoCd_W5Regular_11">date_time</samp>（x 轴）、<samp class="SANS_TheSansMonoCd_W5Regular_11">number_of_likes</samp>（y 轴）和 <samp class="SANS_TheSansMonoCd_W5Regular_11">name</samp>（颜色）。这些是 pandas DataFrame 中列的名称，我们必须保持它们的格式和拼写以确保与正确的列匹配。通过 <samp class="SANS_TheSansMonoCd_W5Regular_11">labels</samp> 属性，我们改变用户在应用页面上看到的内容，使其更加用户友好，*例如* <samp class="SANS_TheSansMonoCd_W5Regular_11">number_of_likes</samp> 变成了 <samp class="SANS_TheSansMonoCd_W5Regular_11">Likes</samp>。

每个属性在 Plotly 文档中都有详细描述，网址为 [*https://<wbr>plotly<wbr>.com<wbr>/python<wbr>-api<wbr>-reference*](https://plotly.com/python-api-reference)。花时间阅读这些描述是值得的，因为它将帮助你了解如何自定义折线图和其他类型的图形。

有关带有 <samp class="SANS_TheSansMonoCd_W5Regular_11">Dropdown</samp> 的 Plotly Express 折线图的完整视频教程，请观看视频“Line Plot (Dropdown)”在 [*https://<wbr>learnplotlydash<wbr>.com*](https://learnplotlydash.com)。

<samp class="SANS_Dogma_OT_Bold_B_11">工具提示</samp>

<samp class="SANS_Futura_Std_Book_11">有一个我们在应用中没有使用，但足够常见值得在这里提到的属性：</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">hover_data</samp><samp class="SANS_Futura_Std_Book_11">，它允许你在用户用鼠标悬停在图表的特定元素上时，在工具提示中提供额外的信息。你可以将分配给</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">hover_data</samp> <samp class="SANS_Futura_Std_Book_11">的值放在列表或字典中。</samp>

<samp class="SANS_Futura_Std_Book_11">当你使用列表时，图表的悬停工具提示将包含列表中的值。例如，如果我们使用</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">number_of_shares</samp> <samp class="SANS_Futura_Std_Book_11">列作为</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">hover_data</samp> <samp class="SANS_Futura_Std_Book_11">列表，悬停工具提示将在用户悬停在图表的线条上时包含这些数据。要尝试此方法，请做出以下更改并重新运行应用：</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">fig = px.line(data_frame=df_filtered, x="date_time", y="number_of_likes",</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">              </samp><samp class="SANS_TheSansMonoCd_W5Regular_11">color="name", hover_data=["number_of_shares"])</samp>

<samp class="SANS_Futura_Std_Book_11">下图展示了悬停信息的区别。</samp>

![](img/pg69.png)

<samp class="SANS_Futura_Std_Book_Oblique_I_11">示例工具提示，包含“分享次数”在悬停数据中</samp>

<samp class="SANS_Futura_Std_Book_11">完成后，请确保删除此更改。</samp>

<samp class="SANS_Futura_Std_Book_11">当你使用字典而不是列表时，键是数据框（DataFrame）的列，值是布尔值，用于控制是否在悬停工具提示中显示数据（</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">True</samp><samp class="SANS_Futura_Std_Book_11">）或不显示（</samp><samp class="SANS_TheSansMonoCd_W5Regular_11">False</samp><samp class="SANS_Futura_Std_Book_11">）。例如，如果你将</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">number_of_likes</samp> <samp class="SANS_Futura_Std_Book_11">列作为字典的键，并将</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">False</samp> <samp class="SANS_Futura_Std_Book_11">作为字典的值，那么代表每个名人喜欢数量的数据将不再显示在悬停工具提示中：</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">hover_data={"number_of_likes": False}</samp>

<samp class="SANS_Futura_Std_Book_11">我们还可以使用</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">hover_data</samp> <samp class="SANS_Futura_Std_Book_11">字典来格式化在工具提示中显示的悬停数据。例如，默认情况下，</samp> <samp class="SANS_TheSansMonoCd_W5Regular_11">number_of_likes</samp> <samp class="SANS_Futura_Std_Book_11">用字母“k”表示 10,000（200,000 会写作 200k）。然而，如果我们更喜欢显示带有逗号作为千位分隔符的完整数字（200,000），我们可以使用：</samp>

<samp class="SANS_TheSansMonoCd_W5Regular_11">hover_data={"number_of_likes": ':,'}</samp>

### <samp class="SANS_Futura_Std_Bold_B_11">总结</samp>

本章向你介绍了一个基础 Dash 应用的基本元素：编程应用所需的 Python 库、使用的数据、Dash HTML 和 Core 组件、使用布局部分将应用组件定位到页面上、使用回调函数将组件连接在一起并创建交互性，以及 Plotly Express 绘图库。在下一章，我们将基于这里学到的技能，开发更复杂的 Dash 应用。
