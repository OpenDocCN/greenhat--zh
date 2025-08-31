# 第十二章：阅读、写作与 Ruby 魔法

# 文件输入与输出

Ruben 环顾四周，叹了口气。“如果货运电梯这么*慢*，我们为什么要跑到这里来？”他问道。

“你知道，”国王一边揉着胡须下巴，一边说道，“我真的不知道。但我想它随时都会到达！”

就在国王刚开口时，货运电梯便带着一声巨大的*铿锵*声到达了。门滑开，露出一个巨大的金属电梯车厢。

“都上车！”Rusty 说道，他们纷纷爬了上去。Rusty 按下一个标有“装货码头”的圆形红色按钮，随着另一声*铿锵*，电梯车开始缓慢下降，驶向 Refactory 的心脏。

“我们很快就到，”Rusty 说道。

“一个*慢*的瞬间，”Scarlet 说道。Ruben 憋住了笑。

![没有标题的图片](img/httpatomoreillycomsourcenostarchimages2160067.png.jpg)

“别担心，”Rusty 说道。“Refactory 里面的每个工人都在那儿，所以那些坏蛋根本不可能逃脱！”

国王在电梯车里走来走去。“我等不及要审问那些无赖了，”他说道。“他们造成了这么多麻烦！我真想知道是什么让他们这么做的。”

“我敢打赌他们是邪恶的忍者巫师！”Ruben 说道。

“更像是邪恶的机器人海盗，”Scarlet 说道。

“无论他们是谁，他们都得为此负责，”女王说道。“不过我们很快就会知道。我们很接近——我能感觉到！”

“我们确实快到了，”Rusty 说道。“下一站：装货码头！”

一会儿后，货运电梯的门发出呻吟声打开，国王、女王、Scarlet、Ruben 和 Rusty 走上了 Refactory 装货码头那片巨大的繁忙的地面。

“Foreman 在这儿！”Rusty 对着一群戴着安全帽的男女喊道，同时带领大家走上金属走道，走到那间巨大的房间中心一个大平台上。“我们得到了什么？”

“先生！”Marshall 一边爬上走道，一边说道，“我赶在你们前面下来，试图评估情况。看起来我们有四个闯入者藏在其中一个装货码头里。”

“哪一个？”Rusty 问道。

Marshall 摇了摇头。“我们不知道！他们在我们看到他们去哪之前就藏了起来。我们只知道，当他们消失时，我们已经包围了码头，所以他们一定还在这里某个地方。”

Rusty 点了点头，轻轻摸了摸胡须。 “嗯，”他终于说道，“最好还是去找他们。” 他走到平台边缘，用靴子踩上一个大圆形凹陷处。随着一阵蒸汽的喷出，一根柱子从平台上升起。面向 Foreman 的一侧闪烁着一个计算装置屏幕特有的光辉。

“每个码头都由一个 Ruby 程序控制，”Rusty 说道，国王、女王、Ruben 和 Scarlet 围在他身边。“Ruby 把每个码头都当作一个*文件*来处理。如果我们能打开每个文件，就能找到我们失踪的罪犯！”

“一个文件？你是说像普通的计算机文件？”Scarlet 问道。

“正是！”Rusty 说道。“Ruby 几乎可以打开你能想到的任何文件：Ruby 程序、文本文件、图片，统统能打开！”

女王笑了。“我对文件了如指掌！”她说道。“我很乐意帮忙打开这些档口，找到我们的罪魁祸首。”她甩了甩手指。“有多少个文件？”她问道。

Rusty 指着远处的墙，那面墙上挂满了数百个沉重的金属门。

![没有标题的图片](img/httpatomoreillycomsourcenostarchimages2160069.png.jpg)

“哦我的天，”女王说道。“那么！我们最好开始了。”她转向 Scarlet 和 Ruben。“为了做这个，我们需要使用 Ruby 的*文件输入/输出*方法，”她解释道。“I/O 部分代表‘输入/输出’。*输入*是你放入文件中的内容，*输出*是文件中出来的内容。”

“就像你写文本文件或保存图片一样？”Scarlet 问道。

“非常像那个，”女王说道。“Ruby 可以将输入写入文件，就像用键盘输入然后点击保存一样。它也可以从文件中读取输出，就像双击文件并打开它一样！”

女王转向 Rusty。“我可以使用一个测试文件来展示它是如何工作的么？”她问道。

Rusty 点点头。“试试*lunch.txt*，”他说。“我想它里面只是有‘ONE KAT-MAN-BLEU BURGER, PLEASE’这段文字。”

“什么是 Kat-Man-Bleu 汉堡？”鲁本问道。

“这是 Refactory 自助餐厅的星期三午餐特餐！”Rusty 说道。“这里的食物没有 Hashery 的食物好，但也还行。那个文件里只是包含了最新的午餐订单。”

# 用 Ruby 打开文件

“非常好！”女王说道。“现在，如果你有一个名为*lunch.txt*的文件，里面只包含‘ONE KAT-MAN-BLEU BURGER, PLEASE’这段文字，你可以这样访问它！”她开始输入：

```

>> file = File.open('lunch.txt', 'r')
=> #<File:lunch.txt>

>> file.read
=> "ONE KAT-MAN-BLEU BURGER, PLEASE\n"

```

“这完全就像你双击*lunch.txt*文件一样，只不过我们可以直接在 Ruby 里读取文件的内容！`PLEASE`后面的`\n`是 Ruby 表示‘换行’的方式。如果你打开文件，它只会是‘ONE KAT-MAN-BLEU BURGER, PLEASE’这段文字，下面会有一行空白。”

女王想了想。“让我再多解释一点。`File.open`告诉 Ruby 根据一个名为*lunch.txt*的文件创建一个文件对象。”

“`'r'`是什么？”鲁本问道。

“那叫做*模式*，”女王说道，“它告诉 Ruby 以什么*模式*打开文件。`'r'`表示我们现在只是读取文件，而不是修改它。”

“好的，”Scarlet 说道，“那么，我们有一个存储在`file`中的文件对象。调用`read`方法会做什么？”

“完全是你想的那样！”女王说道。“它读取文件的内容并展示给我们看。”她停顿了一下。

“虽然通常我们是用一个块来打开文件，就像这样。”她继续输入：

```

>> File.open('lunch.txt', 'r') { |file| file.read }
=> "ONE KAT-MAN-BLEU BURGER, PLEASE"

```

“再次，我们用`File.open`，然后传入我们要打开的文件名作为字符串，后面跟着第二个字符串，告诉我们以什么模式打开文件。在这个例子中，我们使用了`'r'`表示‘读取’。”

“到目前为止明白了，”国王说道。

“我们不再像之前那样将文件对象保存到`file`变量中，然后调用`read`，”皇后继续说道，“而是将一个代码块传递给`File.open`。我们把`file`传递给代码块，然后在代码块中调用`file.read`！”

“用代码块打开文件和不使用代码块打开文件有什么区别吗？”斯卡雷特问。

“这是一个非常重要的区别！”皇后说，“当你用代码块打开文件时，文件会在代码块执行完毕后立即关闭。但是如果你*不*使用代码块打开文件，它不会自动关闭。明白了吗？”她输入了：

```

>> file = File.open('lunch.txt', 'r')
=> #<File:lunch.txt>
>> file.closed?
=> false

```

“如果你*没有*用代码块打开文件，怎么关闭它呢？”鲁本问。

“通过使用`close`方法，就像这样！”皇后说，边输入：

```

>> file = File.open('lunch.txt', 'r')
=> #<File:lunch.txt>

>> file.read
=> "ONE KAT-MAN-BLEU BURGER, PLEASE"

>> file.close
=> nil

```

“这看起来很简单，”国王说，“但是我们为什么一开始就需要关闭文件呢？”

“Ruby 会追踪我们打开的所有文件，而我们运行 Ruby 的计算机只允许我们同时打开有限数量的文件，”皇后解释道，“如果我们试图打开太多文件而不关闭它们，可能会导致计算机崩溃！”

“甜蜜的放风筝的豪猪！”国王说，“我们当然不想要*那样*的事情。”

“另外，如果你不关闭文件，”皇后继续说道，“Ruby 就不知道你已经完成了操作，如果你在没有正确关闭文件的情况下再次使用它，可能会发生一些意外情况。你甚至可能会不小心删除文件中的所有内容！”

“好的，我们会确保关闭我们打开的所有文件，”鲁本说，“听起来，用代码块打开文件是最简单的方式。”

“除了`'r'`，我们还能传递什么给`open`方法呢？”国王边挠着他那小小的皇冠边问。“我们除了读取文件，还能做些什么呢？”

# 写入和向文件添加内容

“当然可以，亲爱的。”皇后说，“你看，Ruby 会按照你告诉它的方式做事，这意味着你必须非常精确地告诉它你要它做什么。当你`open`一个文件时，你传给`open`方法的第一个参数是文件名，第二个参数告诉 Ruby 你希望它对文件做什么。你可以用`open`做很多事——比如，`open 'r'`告诉 Ruby 打开一个文件，但*仅仅*是为了读取文件，从文件的开始位置读取。”

“还有哪些其他模式呢？”斯卡雷特问。

“嗯，你可以使用`open 'w'`来写入文件，”皇后说，“使用`'w'`模式会告诉 Ruby 创建一个你指定名称的新文件，或者完全覆盖任何已存在的同名文件。”

“覆盖！”斯卡雷特说。“你是说它会用你给定的文本替换掉已有文件中的所有内容？”

“没错，”皇后说。

“如果你想*添加*到现有的文件中呢？”鲁本问。

“为此，你可以使用`'a'`模式，”皇后说，“这种模式仍然会告诉 Ruby 创建一个你指定名称的新文件（如果文件尚不存在），但如果该文件*已*存在，Ruby 会从文件末尾开始写入，这样就不会丢失文件中已有的内容。”

“读取、写入和添加，”Scarlet 说。“我想这就是我们想做的所有操作。但如果你使用了一种模式告诉 Ruby 你要做一件事，但又尝试做另一件事，会发生什么呢？”她问。

“我来给你演示！”女王说。她在计算装置上打字：

```

>> file = File.open('lunch.txt', 'w')
=> #<File:lunch.txt>
>> file.read
IOError: not opened for reading

```

“一个错误！”Ruben 说。“那我们在打开文件时必须小心使用正确的模式了。”

“正是如此，”女王说。“记住：Ruby 会精确地执行你告诉它的操作。如果你使用`'w'`模式告诉 Ruby 你只想写入文件，然后试图从文件中读取，Ruby 就会迷惑并产生错误。”

“如果你既想读文件又想写文件呢？”国王问，他正在忙着检查粘在胡子上的一团粉红色的绒毛。

“那么我们需要传递一个稍微不同的模式给`File.open`，”女王说道。她转向 Rusty，“今天食堂有什么特色菜？”她问。

“烤奶酪三明治！”Rusty 说。女王点点头，在计算装置上打字：

```

>> file = File.open('lunch.txt', 'w+')
=> #<File:lunch.txt>

>> file.puts('THE MELTIEST OF GRILLED CHEESES')
=> nil

```

“哇，那是什么？”Ruben 说。“我不知道你可以用`puts`来写入文件！”

“是的，你可以这么做，”女王说。“`puts`和`write`的唯一区别是，`puts`会在你输入的文本后加上一行空白行，Ruby 通过`\n`表示这一空白行（记住，这代表‘换行’）。如果你打开文件，它就会是‘THE MELTIEST OF GRILLED CHEESES’这段文字，下面会有一行空白行！”

“现在，我们试着把午餐文本读回来，”女王说，“但看看第一次我们尝试时发生了什么！”

```

>> file.read
=> ""

>> file.rewind
=> 0

>> file.read
=> "THE MELTIEST OF GRILLED CHEESES\n"

```

“哇！”Scarlet 说。“第一次调用`file.read`时，我们什么也没得到，只有一个空字符串，但在你调用了`file.rewind`之后，我们就能读取到* lunch.txt *中的内容。`rewind`是做什么的？”

“就像你按下遥控器上的 REWIND 按钮将电影送回开头一样，Ruby 使用`rewind`方法将你送回文件的开头。如果你不`rewind`，然后在写入文件后试图直接读取，你只会得到一个空字符串！”女王回答。

“就像试图在电影已经放完时按下播放按钮！”Ruben 说。

“正是如此，”女王说。

“这都说得通，”Scarlet 说，“但我们用了`'w+'`模式，这意味着我们覆盖了原来的* lunch.txt *文件！”

“我们做到了，”女王说。“让我们把它放回去！我在操作的时候会教你几个新技巧。”她开始打字：

```

>> file = File.open('lunch.txt', 'a+')
=> #<File:lunch.txt>

>> file.write('ONE KAT-MAN-BLEU BURGER, PLEASE')
=> 31

>> file.rewind
=> 0

>> file.readlines
=> ["THE MELTIEST OF GRILLED CHEESES\n", "ONE KAT-MAN-BLEU BURGER,
PLEASE"]

```

![没有标题的图片](img/httpatomoreillycomsourcenostarchimages2160071.png.jpg)

“首先，我们使用`File.open`重新打开* lunch.txt *文件进行写入，使用`'a+'`模式，”女王解释道。“这告诉 Ruby 我们想把新文本添加到文件的末尾，而不是替换文件中已有的文本。接下来，我们调用`file.write`并传入我们想要添加到* lunch.txt *末尾的新文本。”

“为什么我们调用`file.write`时 Ruby 会返回`31`？”Ruben 问。

“一个很好的问题！”女王说道。“Ruby 正在告诉我们，它成功地将 31 个字符添加到了 *lunch.txt* 文件的末尾。”

“我明白了，”Ruben 说道。“所以 `'a+'` 模式一定意味着我们向文件中添加内容——这样我们就不会删除已经存在的内容——而 `+` 部分意味着我们既可以添加内容 *也* 可以读取文件！”

“正确！”女王说道。“你还会看到，由于添加文本让我们的位置一直到了文件的末尾，所以我们调用 `file.rewind` 将位置‘倒带’回文件的开始。这就是为什么 `file.rewind` 返回 `0`：我们已经回到了文件的最开始！”

“但那个 `readlines` 方法是做什么的？”Ruben 问道。“它只是给我们返回一个包含文件中所有行的数组吗？”

“说得对，”女王说道。“因为我使用 `puts` 添加了第一行，所以 ONE KAT-MAN-BLEU BURGER, PLEASE 被单独添加在了一行上。`readlines` 方法会读取文件，创建一个数组，每个数组元素就是文件中的一行文本。所以我们这里有一个包含两项的数组。”

“惊人！”国王说道，透过妻子的肩膀往下看。

“不是吗？”她问道。“还有一个 `readline` 方法，它一次只返回一行。看？”她继续输入：

```

>> file.rewind
=> 0

>> file.readline
=> "THE MELTIEST OF GRILLED CHEESES\n"

>> file.readline
=> "ONE KAT-MAN-BLEU BURGER, PLEASE"

```

“我们甚至可以用 `readlines` 和 `each` 一次性打印出所有行！”女王说道，打字速度更快了：

```

>> file.rewind
=> 0

>> file.readlines.each { |line| puts line }

THE MELTIEST OF GRILLED CHEESES
ONE KAT-MAN-BLEU BURGER, PLEASE
=> ["THE MELTIEST OF GRILLED CHEESES\n", "ONE KAT-MAN-BLEU BURGER,
PLEASE"]

```

“太厉害了！”Ruben 说道。

# 在处理文件时避免错误

“我想我现在开始理解文件输入输出了。但是，如果我尝试使用一个不存在的文件，会发生什么？”Ruben 问道，他伸手到计算机装置的键盘上，输入了：

```

>> File.open('imaginary.txt', 'r')
Errno::ENOENT: No such file or directory - imaginary.txt

```

“出错了！”Scarlet 说道。“这有道理。有没有办法在我们尝试使用文件之前先检查它是否存在 *before*？”

“好问题！”女王说道。“如果我们不确定一个文件是否存在，可以使用 Ruby 内建的 `File.exist?` 方法来检查。”她输入了：

```

>> File.exist? 'lunch.txt'
=> true

>> File.exist? 'imaginary.txt'
=> false

```

“太棒了，太棒了！”国王拍了拍手说。“有了这些精彩的 Ruby 工具，我毫不怀疑我们能很快抓住这些坏蛋。”

“没错！”女王说道。她转向 Rusty。“Ruby 程序里有代表所有装卸港口的东西吗？”她问道。

Rusty 点了点头。“有一个数组，`loading_docks`，它是一个文件数组。每个文件代表一个装卸港口门，所以如果你打开并读取所有文件，所有门就应该打开！”

女王思考了一会儿，手指悬停在键盘上方。然后她在计算机装置上输入：

```

loading_docks.each do |dock|
  current_dock = File.open(dock, 'r')
  puts current_dock.read
  current_dock.close
end

```

一个接一个的，装卸港口的门缓缓打开，稍微停留了一会儿，然后滑动关上。每个港口的内容描述开始填满计算机装置的屏幕。

“Ruby 代码... Ruby 代码... Key-a-ma-Jiggers 装运... *在那儿!*”Rusty 一边喊，一边指向远墙中央的一扇门。

四个身影从靠近墙壁左下角的装卸港口跃出，就在门开始再次滑动关闭时。

![没有标题的图片](img/httpatomoreillycomsourcenostarchimages2160073.png.jpg)

“停下！”国王大喊道。“我们包围了你们！”

那四个人的动作出奇地迅速，几乎把几位 Refactory 的工人撞倒，他们正试图找到最近的出口。

“阻止他们！” Rusty 大声喊道，他们五个人正沿着金属走道跑向装卸码头的地面。

几个 Refactory 工人同入侵者进行了搏斗，但他们太快，太灵活了。不到几秒钟，他们就一路跑到了出口！

“让开，让开！”王后喊道，他们五个人刚好在那群身影逃出门的同时，赶到了 Refactory 的出口。国王、王后、Ruby、Scarlet 和 Rusty 没有减速，一头冲过门口，进入了通向他们原路的小走廊。

“他们朝货梯走去了吗？”Ruben 一边跑一边喘气。

“更糟糕！”Rusty 说道。“他们正朝 WEBrick 路奔去！”

国王和王后一起倒吸了一口气。“WEBrick 路！”王后说道。“那条路直通出王国！如果他们从王国大门逃出去，我们就*永远*抓不住他们了！”

“那我们就得确保这种情况不会发生，”Rusty 说道。他转身并大声喊道：“大家，跟上他们！”说完，Refactory 的每个人都冲向小小的亮绿色出口标志，国王、王后、Scarlet、Ruben 和 Rusty 领头。

# 所有装卸码头，集合！

我们差点就抓到这些罪犯了！天啊，这种悬念快把我逼疯了。他们*到底是谁*？国王、王后、Ruben、Scarlet 和 Rusty 能及时抓住他们吗？明天 Refactory 食堂的午餐菜单是什么？这些问题肯定值得永远思考——至少，直到本章结束。与此同时，我们再多练习一点从文件中读取和写入数据吧。

我们从创建一个新的文件 `loading_docks.rb` 开始，输入以下代码。这是一个简单的小程序，它会为每个装卸码头创建一个文本文件，写入一些文本内容，然后再读取出来。

loading_docks.rb

```

  def create_loading_docks(➌docks=3)
➊   loading_docks = []

➋   (1..docks).each do |number|
➍     file_name = "dock_#{number}.txt"
      loading_docks << file_name

➎     file = File.open(file_name, 'w+')
      file.write("Loading dock no. #{number}, reporting for duty!")
      file.close
    end

    loading_docks
  end

➏ def open_loading_docks(docks)
➐   docks.each do |dock|
      file = File.open(dock, 'r')
      puts file.read
      file.close
    end
  end

➑ all_docks = create_loading_docks(5)
➒ open_loading_docks(all_docks)

```

虽然有一些来自前面章节的代码出现，但这里没有什么新鲜的内容需要担心。我们来逐行分析一下代码吧。

首先，我们创建一个名为 `loading_docks` 的空数组 ➊，用来存储我们将要创建的所有装卸码头文件的名字（这样我们可以稍后读取它们）。接下来，我们使用 `(1..docks)` 范围来创建和 `create_loading_docks` 方法要求的装卸码头数量一样多的文件 ➋（如果没有传递数字，默认是 `3` ➌）。

对于范围中的每个数字，我们调用一个块，这个块会创建一个包含该数字的文件（比如 *dock_1.txt*），并将该文件名添加到 `loading_docks` 数组 ➍ 中。接着，我们打开文件，写入一串文本，再关闭文件 ➎。

最后，在 `open_loading_docks` 方法 ➏ 中，我们简单地获取包含码头名称的数组（它看起来像 `["dock_1.txt", "dock_2.txt"...]`，以此类推），对于每个文件名，我们打开文件进行读取，读取其内容，然后关闭它 ➐。所以当我们运行这个脚本时，使用 `all_docks = create_loading_docks(5)` ➑ 和 `open_loading_docks(all_docks)` ➒ 在最后，我们最终会创建 *dock_1.txt* 到 *dock_5.txt*，每个文件都包含其独立的编号和 `"reporting for duty!"` 字符串。

相当不错吧？

一如既往，你可以通过在命令行中输入 **`ruby loading_docks.rb`** 来运行完成的脚本。运行时，你会看到以下内容：

```

Loading dock no. 1, reporting for duty!
Loading dock no. 2, reporting for duty!
Loading dock no. 3, reporting for duty!
Loading dock no. 4, reporting for duty!
Loading dock no. 5, reporting for duty!

```

如果你查看运行了*loading_docks.rb*的目录，你还会看到每个码头的 *.txt* 文件，里面包含了我们的脚本输出的文本！

但我相信你现在已经开始思考如何改进这个简单的小脚本了。例如，我们可以将创建文件的数量从 5 个改成 1 个、3 个、10 个，或者任何我们选择的数字！不过要小心——创建太多文件不仅会填满你的文件夹，还可能会导致计算机崩溃。（这就是为什么我们默认创建 3 个文件，并且在示例中只创建了 5 个的原因。）

你可能已经注意到，我们是使用 `'w+'` 模式来写入文件的，这意味着如果我们再次运行脚本，它将用新内容覆盖文件。那么，如果我们想在文件中添加内容呢？（提示：`'a+'` 模式可能会涉及到。）

那么，如果我们想写入比普通文本文件更复杂的内容呢？如果我们想写入一个 *另一个 Ruby 文件* 呢？这不仅是可能的，而且是专业程序员每天都会做的事情。试着写入一个包含少量 Ruby 代码的文件——像 `puts 'Written by Ruby!'` 这样简单的内容。（确保你将文件以 *.rb* 结尾，而不是 *.txt*，这样 Ruby 才能运行它。）

最后，你打算如何使用我们看到的文件方法，比如 `exist?`、`rewind` 或 `puts` 呢？Ruby 文档中的 *[`ruby-doc.org/core-1.9.3/File.html`](http://ruby-doc.org/core-1.9.3/File.html)* 中是否还有其他可能很酷的文件方法可以使用？记得在上网前向你的本地成年人请教！

# 你知道这个！

你可以读取！你可以写入！好吧，事实上你已经知道如何做这些事情了，但现在你知道了如何 *使用 Ruby* 来做这些事。我不怀疑你现在已经是一个完整的 Ruby 大师了，但为了确保你对我们刚才讲解的 Ruby 魔法没有任何疑问，让我们花点时间回顾一下。

你已经看到，Ruby 可以创建、读取、写入并理解 *文件*，这些文件就像你已经熟悉的计算机文件：文本文件、图片、Ruby 脚本等等。Ruby 可以使用 `open` 方法打开已经存在的文件：

```

>> file = File.open('alien_greeting.txt', 'r')
=> #<File:alien_greeting.txt>

```

它可以使用 `read` 方法来读取文件：

```

>> file.read
=> "GREETINGS HUMAN!"

```

当我们使用完文件时，应该使用 `close` 方法将其关闭：

```

>> file.close
=> nil

```

结果我们发现，如果一次性打开太多文件，我们可能会不小心让电脑崩溃，所以打开的文件最好及时关闭。幸运的是，如果我们使用块打开文件，Ruby 会自动为我们关闭文件：

```

>> File.open('alien_greeting.txt', 'r') { |file| file.read }
=> "GREETINGS HUMAN!"

```

Ruby 对于被告知该做什么很挑剔，所以我们必须使用不同的*模式*来告诉 Ruby 应该使用哪种输入和输出*模式*。当我们使用`'r'`时，我们告诉 Ruby 我们只希望它读取文件，当我们使用`'w'`时，我们告诉它我们只希望它写入文件。若我们希望 Ruby 同时读取*和*写入文件，可以使用`'w+'`模式：

```

>> new_file = File.new('brand_new.txt', 'w+')
=> #<File:brand_new.txt>

>> new_file.write("I'm a brand-new file!")
=> 21

>> new_file.close
=> nil

>> File.open('brand_new.txt', 'r') { |file| file.read }
=> "I'm a brand-new file!"

```

你发现`'w+'`会覆盖一个文件——也就是说，它会将现有文件中的*所有*内容替换成我们告诉 Ruby 写入的字符串。如果我们只想*添加*内容到文件，而不是完全替换它，我们可以使用`'a'`模式（如果我们还想从文件中读取，可以使用`'a+'`模式）：

```

>> file = File.open('breakfast.txt', 'a+')
=> #<File:breakfast.txt>

>> file.write('Chunky ')
=> 7

>> file.write('bacon!')
=> 6

>> file.rewind
=> 0

>> file.read
=> "Chunky bacon!"

```

说到我们的朋友`rewind`，你看到我们可以用它将文件指针倒回到文件开头，从而读取整个文件：

```

>> file = File.open('dinner.txt', 'a+')
=> #<File:dinner.txt>

>> file.write('A festive ham!')
=> 14

>> file.read
=> ""

>> file.rewind
=> 0

>> file.read
=> "A festive ham!"

```

在第一次调用`file.read`时，字符串为空，因为我们已经到达文件的末尾。不过，当我们调用`rewind`时，我们回到了文件的开始位置，再次调用`file.read`时，文本就会显示出来。

你发现如果我们想在一行文本后添加一个空行，我们可以使用文件的`puts`方法而不是`write`。当我们重新读取文件时，Ruby 会把空行显示为一个反斜杠和字母*n*（`\n`）：

```

>> file.puts('A sprig of fresh parsley!')
=> nil

>> file.rewind
=> 0

>> file.read
=> "A festive ham!A sprig of fresh parsley!\n"

```

事实上，你看到我们可以使用`readline`和`readlines`方法逐行读取文件。`readline`一次读取文件的一行，反复调用它就能一行一行地读取：

```

>> file = File.new('dessert.txt', 'a+')
=> #<File:dessert.txt>

>> file.puts('A gooseberry pie')
=> nil

>> file.puts('A small sack of muffins')
=> nil

>> file.rewind
=> 0

>> file.readline
=> "A gooseberry pie\n"

>> file.readline
=> "A small sack of muffins\n"

```

如果我们想一次性读取文件中的所有行，可以使用`file.readlines`结合`each`方法和一个代码块：

```

>> file.rewind
=> 0

>> file.readlines.each { |line| puts line }
A gooseberry pie
A small sack of muffins
=> ["A gooseberry pie\n", "A small sack of muffins\n"]

```

最后，你看到我们可以使用`exist?`方法来检查一个文件是否存在：

```

>> File.exist? 'breakfast.txt'
=> true

>> File.exist? 'fancy_snack.txt'
=> false

```

文件和文件的输入/输出现在对你来说可能没什么大不了的（尤其是因为你已经了解了它们的工作原理），但它们是计算机完成工作的一个重要部分。不要犹豫，去尝试在你的电脑上创建和修改文件，另外——如果得到许可——在互联网上搜索更多关于文件的资料，了解它们如何工作，以及你可以运行的任何有趣的 Ruby 代码来加深理解。但我不再啰嗦了：我们的英雄们正在紧追那些整天在王国里捣乱的骗子们，我们就快要揭开他们的面目，看看他们想要什么，以及国王、王后、鲁本、斯卡利特和重构号的船员们是否能一劳永逸地阻止他们！
