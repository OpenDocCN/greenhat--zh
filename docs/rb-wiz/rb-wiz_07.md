# 第八章 一切都是对象（几乎）

# 我们故事的主题是一个对象

斯嘉丽跑到计算装置前。“你知道那个王国中每个人的目录名吗？”她对国王喊道。“它是一个哈希表，将每个人的名字和地址关联起来。”

“让我们看看，”国王说。“啊，是的！我很确定它叫做`citizens`。”

斯嘉丽点点头，开始在 IRB 中输入。当她按下回车键时，她看到的是：

```

>> **citizens**
=> {
  :aaron_a_aardvark => 'A van down by the river',
  :alice_b_abracadabra => 'The green house with two chimneys',
  :trady_blix => 'Mal Abochny',
  # ...and so on and so forth

```

国王从她肩膀上探头看。“就是它！”他说。“不过哇！王国里肯定有成千上万的人！我们怎么找到 Wherefore？”

斯嘉丽继续敲打键盘：

```

>> **citizens.size**
=> 24042

```

“是的，哈希表肯定太大，不能手动查找，”斯嘉丽说，“不过我敢打赌我们可以写一个方法来找到他！”

鲁本仔细研究了`citizens`哈希表。“记得我们可以通过输入哈希名，然后在方括号里加上键，来获取哈希值吗？”他问。

“是的，”斯嘉丽说。

“嗯，”鲁本说，“如果我们写一个方法，传入一个人的名字和`citizens`哈希表，然后试图在哈希表中查找这个名字，怎么样？”

“鲁本，你真是个天才！”斯嘉丽说。她迅速打字：

```

>> **def find_person(name, people)**
>>   **if people[name]**
>>     **puts people[name]**
>>   **else**
>>     **puts 'Not found!'**
>>   **end**
>> **end**

```

“等等，等等。这是什么？”国王问。“只是我随便写的一个方法，”斯嘉丽说。“看吧？它叫做`find_person`，它接受一个人的`name`作为符号和一个`people`的哈希表作为参数。如果它在哈希表中找到了名字，就打印出来；否则，它就说名字没找到！”她继续输入：

```

>> **find_person(:wherefore, citizens)**
=> One half mile due east!

```

“找到了！”斯嘉丽说。“它在`citizens`哈希表中找到了`:wherefore`键。”

“往东走半英里！”褪色骑士说。“应该只要几分钟，东边就在那儿。走吧！”

达格龙挺直了身子，瞬间遮住了太阳。“我也去，”她说，“我和 Wherefore 是老朋友，我们已经有一段时间没见面了。能再次见到他真好。”

“那好，”国王说，“带路吧！”

褪色骑士和达格龙转身，朝着上午晚些时候的太阳走去，国王、斯嘉丽和鲁本紧随其后。他们走着走着，树木越来越高，彼此也越来越密集，几分钟后，太阳只从卡尔梅松松树的树梢间透出一抹温暖的红光。

“等一下，”鲁本说，他停下脚步，转过头去。“你听到了吗？”

他们都停了下来。国王捂住耳朵，摇了摇头，把小指头插进耳朵里转了转，然后又捂住耳朵。“我什么也听不见，”他说。

“我也听到了，”达格龙说。“是——”

“音乐！”鲁本喊道，“它是从那边传来的！”他指向他们之前前进方向的右侧。

“走吧！”斯嘉丽说道，大家继续向松树林走去。

音乐声渐渐增大，在穿过一片特别密集的树林后，大家发现自己站在一片小草地的边缘。在草地中央，树桩上坐着一个身穿红色长袍、戴着带有长白羽毛的弓箭手帽的男人。他正在弹奏一把粉色的曼陀林，偶尔停下来用羽毛笔在一卷长纸上急促地涂写，而这支羽毛笔与他帽子上的白羽毛一模一样。

“为什么！”达格龙轰然回应。

站在树桩上的男人停下了涂鸦，抬起头来看。他的脸上露出了灿烂的笑容。“达格龙！”他喊道，“真高兴见到你！进来，进来，进来。”

![没有标题的图片](img/httpatomoreillycomsourcenostarchimages2160029.png.jpg)

在达格龙的带领下，大家穿过草地，围绕着为什么转了一圈。为什么灵活地跳下树桩，脱下帽子，深深地鞠了一躬。

“朋友们，”他说，“欢迎来到我的森林根据地！”他指着树桩。“现在看起来不怎么样，但我一直对修缮旧物有一种情结。而我，”他说，“是你们谦卑的主人，流浪的吟游诗人为什么。”为什么把帽子戴回了头上。“我当然认识达格龙，我之前也见过那位灰白骑士。”他看着国王，双手合十。“陛下，”他说，“我之前没有荣幸见过您，但现在*确实*是荣幸。”

“同样的，”国王说道，“我们听说过很多关于你的事情！”

为什么转向鲁本和斯卡利特。“那就剩下你们这些可爱的无赖了。你们叫什么名字？”

“我是斯卡利特，”斯卡利特说道，“这是鲁本。”

“嗨！”鲁本说道。

“你好，你好！”为什么说道，“很高兴见到你。不过恐怕你们来得不是时候。”他叹了口气。“我整个上午都在写一首民谣，但才写了一半。如果晚上之前想写完，我得立刻回去继续。”

“一首民谣？”斯卡利特说道。

“哦，是的，”为什么说道，“你看，我算是个商人。我经营一个小型的民谣配送服务，拥有几十个客户。唯一的难题是，”他说，“这意味着我确实有*几十*个客户，而每首民谣都需要我花上好几个小时才能完成。我简直忙不过来了！”他从长袍口袋里拿出一条手帕，擦了擦额头上的汗水。

达格龙若有所思地哼着，吐出几缕烟雾。“你知道，”她说道，“我想我可以帮上忙。”她环顾四周，几乎空旷的草地。“不过我需要一点鲁比魔法。你们附近有计算装置吗？”

为什么笑了。“*我*有计算装置！”他说着，踩上了树桩上最大的根部。树桩震动了一下，然后从地面上升起几英尺。它缓慢旋转着升起，露出了熟悉的计算装置屏幕的光芒！

# 类与对象

“太好了，”Dagron 说着，绕过树墩，紧挨着屏幕靠了过去。“那么！你 Ruby 程序中的每个对象都有一个唯一的 ID 号，”她说。“你会发现，你创建的对象通常比 Ruby 创建的对象有更高的 ID 号。看？”她用爪子触摸了计算设备的屏幕，说道：“Ruby 有一些非常熟悉的对象，比如`0`或`true`。Ruby 中的每个对象都有自己的 ID 号，这就是 Ruby 用来追踪它们的方式。看看！”

```

>> **0.object_id**
=> 1
>> **true.object_id**
=> 20

```

“像这样的内建 Ruby 对象在 IRB 启动或脚本加载时，Ruby 会自动分配 ID 号，”Dagron 继续说道。“Ruby 也会给我们在程序中创建的 Ruby 对象分配 ID 号，但这些 ID 号通常是非常高的。这是因为 Ruby 为我们提供了很多内建对象！”她再次触摸计算设备，屏幕上出现了更多文字：

```

>> **:baloney.object_id**
=> 1238088
>> **"The Ballad of Wherefore the Wand'ring Minstrel".object_id**
=> 2174481360

```

“她是怎么做到的？”Ruben 悄声对 Off-White 骑士说道。“她甚至什么都没打！”

“她不需要，”骑士低声回应。“龙是神奇的生物，而 Dagron 是所有龙中最具魔力的一个。”

“但是这些对象都来自哪里？”Dagron 问道。Wherefore 盘腿坐在地上，期待地仰望着她。

“来自*类*，”Dagron 说，回答了她自己提的问题。“你可以把 Ruby 类看作是制造特定类型对象的小机器，每个 Ruby 对象都知道自己属于哪个类。我们可以使用`class`方法来询问对象属于哪个类。首先，Ruby 的数字来自`Fixnum`类。看！”她说着，屏幕上出现了更多代码：

```

>> **7.class**
=> Fixnum

```

“一个字符串的类自然是……`String`！”她继续说道：

```

>> **'Odelay!'.class**
=> String

```

“知道这些很不错，”国王插话道，“但是这对我们有什么*好处*呢？”

“我正要说这个，”Dagron 说道。“当你知道一个 Ruby 对象属于哪个类时，你可以使用`new`方法来创建该类的一个新对象。你以前见过这个，对吧？”她指着屏幕上的新代码说道：

```

>> **greeting = 'Hello!'**

```

“是的！”Ruben 说道。

“好吧，现在你可以做*这个*！”Dagron 说着，再次触摸了计算设备。

```

>> **greeting = String.new('Hello!')**
=> "Hello!"

>> **greeting**
=> "Hello!"

>> **greeting.class**
=> String

```

“你看到了吗？”Dagron 说，折叠起她的爪子。“Ruby 中的每个对象都有一个类，我们可以用`class`方法找到它。更重要的是，每个对象都是通过类的`new`方法创建的，类的工作就是生成特定类型的对象！”

“所以这个类就像一个饼干模具，压制出特定种类的饼干，”Wherefore 说着，用闭拳拍打着掌心做出压制的动作。“姜饼人、巧克力碎片饼干、雪花形状的糖饼干。而对象就是这些饼干！”

“这是一个非常好的思考方式，”Dagron 说。

“什么时候吃午餐？”Wherefore 问道。

![没有标题的图片](img/httpatomoreillycomsourcenostarchimages2160031.png.jpg)

“恐怕我还是不太明白，”国王打断道。“我仍然有点困惑，类到底有什么重要的？”

“我想我可以帮忙解释这个，”斯嘉丽说道。“当我们处理数字或字符串时，类所做的有用事情可能不太明显。但是如果我们要创建我们*自己的*对象，并且有*自己的*新类，类就成了从模板创建一堆对象的方式。例如，如果我们有一个`Minstrel`类，我们就可以创造一堆吟游诗人！”

“怎么做？”国王问道。

# 创建我们的第一个类：Minstrel

“很高兴你问了！我们来试试吧，”达格龙说。她触摸了计算装置，更多代码出现在屏幕上。

### 注

*对于这些较长的代码示例，我们将编写 Ruby 脚本！每当你看到代码上方的文件名以斜体显示时，比如接下来例子中的* minstrel.rb *，这意味着你可以将代码输入到文本编辑器中，并将其保存为给定名称的文件。*

minstrel.rb

```

class Minstrel
  def initialize(name)
    @name = name
  end

  def introduce
    puts "My name is #{@name}!"
  end

  def sing
    puts 'Tralala!'
  end
end

```

“那么，”达格龙清了清嗓子，说道，“我们来看看。`class`关键字告诉 Ruby 你想创建一个新类，”她说。“就像你使用`def`和`end`告诉 Ruby 你在定义一个新方法一样，你使用`class`和`end`告诉 Ruby 你想创建一个新类。”

“在`class`之后，你输入类的名称，可以是你喜欢的任何名字，”达格龙解释道。“不过，类名*总是*以大写字母开头，比如`Minstrel`。”Wherefore 已经把羊皮纸翻过来，正在尽快地在他的歌谣背面做笔记。“我们正在创建`Minstrel`类，这样我们就可以创建很多新的吟游诗人。”

“在`class`和最后的`end`之间，你可以添加任何你想要的方法，就像在类外定义方法一样，”达格龙继续说道。“在`Minstrel`类中，我定义了三个方法：`initialize`、`introduce`和`sing`。”

鲁本凑近计算装置的屏幕。“为什么那个`@name`变量前面有一个`@`呢？”他问。

“一切都在合适的时候，”达格龙说道。

### 注

*为了跟随达格龙，我们需要将她的脚本加载到 IRB 中。当我们想从文件中在 IRB 中使用代码时，只需在包含 Ruby 脚本的文件夹中启动 IRB，然后使用`load`命令加载文件。像这样加载* minstrel.rb *：*

```

>> **load 'minstrel.rb'**
=> true

```

*现在让我们试试达格龙的代码！*

“首先，让我们看看`Minstrel`类的`initialize`方法。每当我们使用`new`方法创建类的新实例时，这个方法就会被调用。看看！”达格龙在屏幕上添加了更多代码。

```

>> **wherefore = Minstrel.new('Wherefore')**
=> #<Minstrel:0x000001052c77b0 @name="Wherefore">

```

“当我们调用`Minstrel.new`时，我们创建了一个新的吟游诗人。因为`initialize`方法只接受一个参数`name`，所以我们在调用`new`方法时传入了一个名字。你看到`@name="Wherefore"`那部分了吗？这意味着`wherefore`的名字是`'Wherefore'！`”达格龙深思了一下，“所以如果你想在创建类的新实例时立刻执行某些代码，就把它放在类的`initialize`方法定义中。”

“明白了，”国王说。

“现在所有的 `Proc.new` 相关内容更有意义了！”Ruben 说道。“我们只是每次调用 `new` 时创建新的 proc！”

“没错！”Off-White Knight 说道。“`Proc` 是一个内置的 Ruby 类，每当我们调用 `new` 时，就会创建一个新的实例。我们基本上有一个小工厂，每当我们想要时就生成新的 proc。类就是这个：小工厂，生产对象！”

“正是如此，”Dagron 喘着气说，她几乎露出了笑容。

“你添加的另外两个方法怎么样？”Scarlet 问道。

“啊，是的，”Dagron 说道。“我们的 `wherefore` 是一个 `Minstrel`，因此他可以自动访问那些方法。”

```

>> **wherefore.introduce**
My name is Wherefore!
=> nil

```

“看到了吗？”她说。“`introduce` 方法打印一个包含乐师名字的字符串，在这个例子中是 Wherefore。而且他不仅可以自我介绍，还能唱歌！”

```

>> **wherefore.sing**
Tralala!
=> nil

```

“我们已经讨论过类是如何生成某种类型的对象的，”Dagron 说道，“但我们其实还没有真正提到对象*是什么*。其实很简单：对象就是一小堆值！你可以把它们想象成信息的容器——可能包含一个名字、一个大小，或者一个颜色。每个对象从它的类那里继承方法，允许我们访问它的名字、大小或颜色，这就构成了我们的 Ruby 代码。”

“好的，”国王说道，“现在我明白为什么类如此重要了：它们让你可以重用代码来处理多个对象，而不需要每次都重写所有的信息和方法，Ruby 代码就是由对象组成的。但让我们回到 Ruben 的问题——我们看到的那个奇怪的螺旋形图案在 `wherefore` 的 `@name` 上是怎么回事？”

“@符号 (`@`) 只是告诉 Ruby 这是一个特殊类型的变量——一种描述对象值的变量，像是对象的名称、大小或颜色！我稍后会详细解释一下。让我们通过使用巫师（weezards）来试试这个例子，”Wherefore 说道。

“你是说巫师，”Scarlet 说道。

“不，是 weezards，”Wherefore 说道。“短巫师。小巫师。Wee 的东西。Weezards。”

![无标题图片](img/httpatomoreillycomsourcenostarchimages2160033.png.jpg)

“很好，”Dagron 说道。“但为了讲明白这个问题，我需要解释一下 Ruby 中四种不同类型的变量。”

“四种！”国王惊呼道。“我以为只有一种！”

“你通常看到的变量叫做*局部变量*，”Dagron 说道。“它们非常适合创建你很快就会用到的变量。但一旦我们开始编写自己的方法和类时，就需要创建一些可以在这些方法和类的定义内定义，但会在稍后使用的变量——例如，当我们最终调用一个方法或创建一个类的实例时。”

“另外三种变量，”Dagron 接着说，“分别是*全局变量*、*类变量*和*实例变量*。虽然在不同地方使用不同类型的变量可能让人感到困惑，但一旦你掌握了其中的规律，就会发现其实非常简单。”

“你说的不同地方是什么意思？”Scarlet 问道。

“*作用域*，”Dagron 说道。

# 变量作用域

哇哦，这越来越有意思了。我们正在进入语言的真正核心！*作用域*是 Ruby 中一个非常重要的概念，我激动得不行，简直无法抑制自己的兴奋。我希望你不介意我在 Dagron 向我们的勇敢英雄们解释作用域时，借此机会也给你简单解释一下作用域。只需要一分钟。

这可能让你感到惊讶，但并不是所有变量都可以在 Ruby 程序中的任何时候随便使用。在程序中，有时候即使你定义了一个变量，如果你尝试使用它，Ruby 会抱怨并表示它不存在！这可能意味着什么呢？

这意味着什么呢：在程序的任何给定时刻，只有*某些*你定义的变量和方法可以被访问。程序中任何时刻可以访问的变量和方法集合定义了当前的作用域；你可以使用作用域内的任何内容，而无法使用作用域外的任何东西。

那么，是什么决定了 Ruby 中变量的作用域呢？目前，这里有一个很好的经验法则：新的作用域是在方法定义、类定义和代码块内部创建的。所以，如果你使用的是我们一直在使用的普通局部变量，这样做完全没问题：

```

>> **regular_old_variable = 'Hello!'**
=> "Hello!"

```

我们只是将一个`regular_old_variable`设置为字符串`'Hello!'`。这很标准。

接下来，我们将在方法内部定义一个变量：

```

>> **def fancy_greeting**
>>   **greeting = 'Salutations!'**
>> **end**
=> nil

```

在这里，我们在名为 `fancy_greeting` 的方法内部定义了一个名为 `greeting` 的变量。你之前已经见过方法定义，所以这里也没有什么新鲜的东西！

接下来，我们将重新回顾代码块：

```

>> **3.times { |number| puts number }**
0
1
2
=> 3

```

到这个阶段，你已经是一个块的专家了，所以你也掌握了这一点：我们在数字 `3` 上调用了 `times` 方法，并传递了一个代码块。在块内部，我们使用变量 `number` 来跟踪当前数字，并依次打印出 0 到 2 的每个数字。（别忘了，计算机从 0 开始计数，而不是从 1。）

## 这些变量错误将会让你震惊和惊讶！

不过，可能让你感到惊讶的是，这些代码中的某些部分会导致 Ruby 抛出错误！让我们一一看看。在下面的代码中，我们从定义一个变量开始。但这个`regular_old_variable`存在于 `FancyThings` 类定义之外（在外部*作用域*中），因此它在类定义内部*不存在*！

```

>> **regular_old_variable = 'Hello!'**
=> "Hello!"

>> **class FancyThings**
>>   **puts regular_old_variable**
>> **end**
NameError: undefined local variable or method `regular_old_variable'
for FancyThings:Class

```

在类定义内部，你会获得一组全新的局部变量（你一直以来看到的那种变量），因此 Ruby 正确地告诉你，在类内部，你还没有一个叫做 `regular_old_variable` 的变量。

方法定义也是如此：它们也会获得自己的局部变量集，因此当你在方法内部定义 `regular_old_variable` 时，它在方法外部是不存在的：

```

>> **def fancy_greeting**
>>   **puts regular_old_variable**
>> **end**

>> **fancy_greeting**
NameError: undefined local variable or method `regular_old_variable'
for main:Object

```

又一个错误！

而且，正如你可能已经猜到的那样，我们在块示例中的 `number` 变量是*局部的*，它在块结束后立即停止存在，所以如果我们在块结束后再次尝试使用它，就会出现错误！

```

>> **3.times { |number| puts number }**
0
1
2
=> 3

>> **puts number**
NameError: undefined local variable or method `number' for
main:Object

```

在这里，对于从 0 到 3 的每个数字，Ruby `puts`将传入块的`number`打印出来。现在，块变得有趣了：就像方法或类一样，在块中定义的变量在块结束时会停止存在。不过，*不同于*方法和类，块可以访问它们外部的变量和信息！在这种情况下，我们的块知道数字 3，因此知道变量`number`应该取 0 到 3 之间的每个数字。然而，一旦块结束，Ruby 就不再关心`number`，所以如果我们试图再次使用它，就会导致错误。

当我第一次了解到 Ruby 可以在程序的某些部分看到变量，而在其他部分看不到时，我好好挠了挠头，我相信你现在一定在问自己我当时问自己的一样问题：“如果是真的，那我到底怎么才能在程序的其他地方使用我在类或方法中创建的变量呢？”好吧，幸运的是，Dagron 就要告诉我们答案了！

## 全局变量

“让我们从*全局变量*开始，它可以在程序的任何地方被访问。举个例子可能会有帮助，”Dagron 说，她用爪子触碰了计算装置的屏幕：

```

>> **$location = 'The Carmine Pines!'**

>> **def where_are_we?**
>>   **puts $location**
>> **end**

>> **where_are_we?**
The Carmine Pines!
=> nil

```

“这里，”Dagron 说，“我们创建了一个名为`$location`的变量，它的值是字符串`'The Carmine Pines!'`。然后我们创建了一个方法，`where_are_we?`，它尝试访问`$location`。通常情况下，这不会起作用，但因为`$location`是一个全局变量，我们在调用`where_are_we?`方法时会得到`'The Carmine Pines!'`！”

“啊哈！我以前见过这种变量，”Off-White Knight 说。“我能通过它前面的美元符号认出来！全局变量可以很有用，因为它们可以在 Ruby 程序的任何地方被访问。你可以在方法外定义全局变量，在方法内定义，在类中定义，随便你想在哪里定义，而且如果你在程序的其他地方使用它，它也能正常工作。但，”她举起一根手指说，“如果变量可以在程序的任何地方被访问，它也可以在程序的任何地方被*更改*，而且你不总是能明确知道何时或者如何发生了这种变化。”

Scarlet 点点头。“没错！”她说。“记得我们发现有东西正在改变 Flowmatic Something-or-Other 中的变量吗？想象一下，如果我们所有的变量都能在程序的任何地方随时被更改，那会有多糟糕！”

“想都别想！”国王打了个冷战说。“我们当然不想要*那个*。好吧，那如果可以避免，我们就不使用全局变量！那我们可以使用其他类型的变量吗？”

## 类变量

“明智的选择，陛下，”达格龙说道。“我们还可以使用另一种类型的变量，那就是*类变量*，它非常有用，特别是当我们希望一个类保存一些关于自己的信息时。就像所有全局变量都以`$`开头一样，所有类变量都以`@@`开头，且一个类可以有任意多个类变量。类变量可以被类内部和类的任何实例访问；所有实例共享同一个类变量。现在，韦尔福，我们来用你的巫师例子。”她对着计算装置吹了个烟圈，屏幕上出现了这段代码：

weezard.rb

```

class Weezard
  @@spells = 5

  def initialize(name, power='Flight')
    @name = name
    @power = power
  end

  def cast_spell(name)
    if @@spells > 0
      @@spells -= 1
      puts "Cast #{name}! Spells left: #{@@spells}."
    else
      puts 'No more spells!'
    end
  end
end

```

“我们定义了一个`Weezard`类，其中有一个类变量`@@spells`，”达格龙说道，“还有两个方法：`initialize`，它为特定的巫师设置名字和能力；`cast_spell`，任何巫师都可以使用。现在，我们使用`new`来创建两个具有特殊能力的新巫师。别忘了先`load`你刚刚写的代码！”

```

>> **load 'weezard.rb'**
=> true
>> **merlin = Weezard.new('Merlin', 'Sees the future')**
=> #<Weezard:0x00000104949260 @name="Merlin", @power="Sees the
future">
>> **fumblesnore = Weezard.new('Fumblesnore', 'Naps')**
=> #<Weezard:0x0000010494c500 @name="Fumblesnore", @power="Naps">

```

“这就是我们这些巫师有趣的地方，”达格龙继续说道。“即便是`Merlin`和`Fumblesnore`有不同的能力，它们却在操作同一个变量`@@spells`！每当它们使用`cast_spell`时，法术变量就会减少一。看看这个。”

```

>> **merlin.cast_spell('Prophecy')**
Cast Prophecy! Spells left: 4.
=> nil

>> **fumblesnore.cast_spell('Nap')**
Cast Nap! Spells left: 3.
=> nil

```

“所以当你创建一个类变量时，整个类只有一个副本，而你创建的所有实例都共享这个类变量？”鲁本问道。

“没错，”达格龙说道。

“所有巫师共享固定的法术组，听起来有点奇怪，不是吗？”韦尔福问道。“是不是每个巫师都有自己的一套法术更合理？”

## 实例变量

达格龙点了点头。“有时候，创建对象的类需要跟踪某些信息，但并不是每次都这样，”她说。“因此，我们在 Ruby 中并不常使用类变量；我们更多使用的是*实例*变量和*局部*变量。事实上，通过实例变量，我们可以为每个巫师提供她自己的法术集，”达格龙继续说道，屏幕上出现了更多代码。“实例变量可以被类内部以及类的任何实例访问，就像类变量一样。大区别在于，每个实例都有自己独立的变量副本！”

weezard_2.rb

```

class Weezard
  def initialize(name, power='Flight')
    @name = name
    @power = power
    @spells = 5
  end

  def cast_spell(name)
    if @spells > 0
      @spells -= 1
      puts "Cast #{name}! Spells left: #{@spells}."
    else
      puts 'No more spells!'
    end
  end
end

```

“看看我们是如何将`@@spells`变量从一个属于类的变量移到`initialize`方法中的`@spells`实例变量的吗？”达格龙问道。“以`@`开头的变量是*实例变量*。之所以称为实例变量，是因为每个*实例*，也就是 Ruby 中由类创建的对象，都有自己的副本。”

“所以当我们用`new`方法创建`Weezard`类的实例时，每个实例都会分配到自己的`@spells`变量吗？”斯嘉丽问道。

“正是如此，”达格龙说道。“事实上，我们现在就来做这个。我们将像之前一样创建我们的巫师。”

```

>> **load 'weezard_2.rb'**
=> true
>> **merlin = Weezard.new('Merlin', 'Sees the future')**
=> #<Weezard:0x0000010459e160 @name="Merlin", @power="Sees the
future", @spells=5>
>> **fumblesnore = Weezard.new('Fumblesnore', 'Naps')**
=> #<Weezard:0x000001045a13d8 @name="Fumblesnore", @power="Naps",
@spells=5>

```

“这看起来就像上次我们创建巫师时的样子！”国王抱怨道。

“非常相似，”达格龙承认，“但*确实*有一个重要的区别。看看每个巫师施法时发生了什么！”

```

>> **merlin.cast_spell('Prophecy')**
Cast Prophecy! Spells left: 4.
=> nil

>> **fumblesnore.cast_spell('Nap')**
Cast Nap! Spells left: 4.
=> nil

```

“它们每个都有自己的`@spells`变量！”斯嘉丽说。“这就是为什么`fumblesnore`的法术次数在`merlin`施法时没有受到影响。”

“完全正确，”达格龙说。“尽管它们的`@spells`变量有相同的名字，每个实例都有自己的一份，所以它们不会互相冲突。不仅如此，因为类的实例总是可以访问它们的实例变量，所以我们在类的`initialize`方法中定义的任何实例变量都可以被新创建的对象使用。”

“这就是为什么我们在`initialize`方法定义中做像`@name = name`这样的事，”象牙白骑士说。“它确保当我们传入`name`参数时，每个实例都会在`@name`中保存一份。”

## 局部变量

“说到局部变量，”达格龙说，“我们来看看这些吧，好吗？它们应该很熟悉，但值得再看一眼。*局部变量*只能在它当前的作用域内看到，这意味着它只能在定义它的方法或类内看到。”

计算机装置屏幕上出现了新的代码：

```

>> **class YeOldeClass**
>>   **local_variable = 'I only exist inside the class!'**
>> **end**

>> **puts local_variable**
NameError: undefined local variable or method `local_variable' for
main:Object

>> **def yet_another_method**
>>   **another_local = 'I only exist inside this method!'**
>> **end**

>> **puts another_local**
NameError: undefined local variable or method `another_local' for
main:Object

```

“所以实际上，局部变量只能在它们定义的类或方法内部看到，或者我们可以在*所有*类和方法定义之外使用它们，”斯嘉丽说道。

“没错，”达格龙说。“Ruby 中有一个特殊的作用域，叫做*顶级作用域*，所以如果你在*任何*方法或类定义之外定义局部变量，Ruby 就能看到它们。看看这个！”

```

>> **local_variable = "I'm the top-level local variable!"**

>> **def local_in_method**
>>   **local_variable = "I'm the local variable in the method!"**
>>   **puts local_variable**
>> **end**

>> **puts local_variable**
I'm the top-level local variable!
=> nil

>> **local_in_method**
I'm the local variable in the method!
=> nil

```

“你看到了吗？”达格龙说。“局部变量甚至可以有完全相同的变量名，只要它们在不同的作用域中！Ruby 知道方法定义会有自己的一组局部变量，所以它不会抱怨有两个同名的变量。”

“所以局部变量只能在我们定义它们的类或方法中，或者在这个特殊的顶级作用域中看到，”国王说。“但全局变量可以在任何地方看到，而且如果我们创建了一个类的实例，实例可以看到我们在定义类时创建的任何实例变量。”

“正是如此，”达格龙说。

“而类可以看到它自己的类变量，”国王继续说道。

“正确！”达格龙说。“事实上，不仅实例可以有像`initialize`、`introduce`和`sing`这样的的方法；*类*也可以有它们自己的方法！”

“就在我开始理解的时候！”国王抱怨道。“这是怎么可能的？”

“因为，”达格龙回答道，“Ruby 类*也是对象*！”

“我需要坐下了，”国王说道。

“你*确实*坐下了，”在场的问道。

“是的，”国王说道，他盘腿坐在象牙白骑士和漫游歌手之间。“继续吧，达格龙女士，”他说。“我们如何能直接将一个方法添加到类本身，而不仅仅是类的一个实例呢？”

# 对象和 self

“嗯，”达格龙说道，“Ruby 始终保持一个名为`self`的特殊内置变量，而`self`指代的是我们当前谈论的 Ruby 对象。”她开始快速讲解，嘴里冒出小小的火花。“所以我们所需要做的就是使用`self`来定义类中的方法，而不是将该方法添加到实例上，而是将它添加到类本身。”

“也许举个例子会更清楚些，”白色骑士说。她伸手过去，开始在计算机装置上打字：

monkey.rb

```

class Monkey
  @@number_of_monkeys = 0

  def initialize
    @@number_of_monkeys += 1
  end

  def self.number_of_monkeys
    @@number_of_monkeys
  end
end

```

“这里我创建了一个`Monkey`类，”骑士说道。“它有一个`@@number_of_monkeys`类变量，用来跟踪我们创建了多少个猴子实例，还有我们在之前的类中看到的`initialize`方法。当我们对`Monkey`调用`new`来创建一个新猴子时，它会把`@@number_of_monkeys`加 1。”

“那那个`self.number_of_monkeys`方法呢？”鲁本问道。

“那是一个类方法！”骑士说道。“这是`Monkey`类本身的方法，当我们调用它时，它将返回`@@number_of_monkeys`。我们来看看吧！首先，我们加载那个脚本，然后创建几个猴子。”

```

>> **load 'monkey.rb'**
=> true
>> **monkey_1 = Monkey.new**
=> #<Monkey:0x000001048fccf8>
>> **monkey_2 = Monkey.new**
=> #<Monkey:0x00000104902310>
>> **monkey_3 = Monkey.new**
=> #<Monkey:0x00000104907900>

```

“很好！”白色骑士说。“现在我们有了猴子，让我们问问`Monkey`类有多少只猴子。”她在计算机装置上打字：

```

>> **Monkey.number_of_monkeys**
=> 3

```

“太棒了！”Wherefore 说道。“但是为什么不直接问一个猴子有多少只猴子呢？”

“嗯，”骑士说道，“首先，问一个猴子实例有多少其他实例是没有意义的——那是类的事情，而不是实例的！但更重要的是，因为我们在定义`number_of_monkeys`方法时用了`self`，它仅仅是类的方法，而不是实例的方法！看见了吗？”她继续打字：

```

>> **monkey_1.number_of_monkeys**
NoMethodError: undefined method `number_of_monkeys' for
#<Monkey:0x000001048fccf8>

```

“看！现在`Monkey`类有了自己的`number_of_monkeys`方法，但它只属于类本身；猴子实例没有这个方法。”

![没有标题的图片](img/httpatomoreillycomsourcenostarchimages2160035.png.jpg)

“事实上，”骑士说道，“向类添加方法是很常见的，Ruby 为此提供了更简洁的语法。它看起来像这样！”她继续打字：

monkey_2.rb

```

class Monkey
  @@number_of_monkeys = 0

  def initialize
    @@number_of_monkeys += 1
  end

  class << self
    def number_of_monkeys
      @@number_of_monkeys
    end
  end
end

```

“看到吗？”她问道。“我没有在类中通过`self.number_of_monkeys`来定义`number_of_monkeys`方法，而是使用了`class << self`来告诉 Ruby：‘嘿！我定义的每个方法，直到我说`end`为止，都是类的方法，而不是实例的方法。’看看当我在`Monkey`上调用这个方法而没有创建任何实例时会发生什么。”

```

>> **load 'monkey_2.rb'**
=> true
>> **Monkey.number_of_monkeys**
=> 0

```

“现在看看，如果我创建一个实例并再次调用这个方法会发生什么，”骑士说道。

```

>> **monkey = Monkey.new**
=> #<Monkey:0x0000010490af60>
>> **Monkey.number_of_monkeys**
=> 1

```

“看到了吗？这就像使用`self.number_of_monkeys`一样，”白色骑士说，露出灿烂的笑容。

“真有趣，”达格龙说道。“我从没见过`class << self`。”

“真的？”Wherefore 问道。

“没人知道所有的事情，”达格龙说。“连我也不行！”

“许多人觉得`def self.method_name`语法更容易理解，”骑士说道，“所以每当你需要为一个类添加方法时，使用这个语法是完全没问题的。”

“当然，”斯嘉丽说，“现在`self`对我来说好多了！它只是指 Ruby 程序‘正在谈论’的对象。而在这种情况下，`self`就是我们所在的类！”

# 方法和实例变量

“完全正确，”达格龙说道。“有了这个，我还有一个技巧要展示给你们。你们看，虽然为我们的实例创建实例变量非常容易，但想要访问它们并不总是那么简单。明白我的意思了吗？”她说道，在她说话时，新的代码开始填满屏幕：

```

>> **class Minstrel**
>>   **def initialize(name)**
>>     **@name = name**
>>   **end**
>> **end**

```

“我重新创建了我们之前的`Minstrel`类，但只包含一个`initialize`方法，”达格龙说道。“没有`introduce`或`sing`方法！让我们像之前一样创建一个实例。”

```

>> **wherefore = Minstrel.new('Wherefore')**
=> #<Minstrel:0x000001049637c8 @name="Wherefore">

```

“现在，”达格龙说道，“看我们的吟游诗人实例是如何拥有‘Wherefore’这个名字的？（你可以通过`@name="Wherefore"`这一部分看出来。）让我们试着去访问它。”

```

>> **wherefore.name**
NoMethodError: undefined method `name' for
#<Minstrel:0x000001049637c8 @name="Wherefore">

```

“你看，”达格龙说道，“虽然`wherefore`有一个`@name`实例变量，但它没有`name`*方法*。在 Ruby 中，所有重要的是方法。为了让`wherefore.name`真正起作用，我们需要写一个方法来访问`@name`实例变量。”

“那是不是意味着我们需要在`Minstrel`类中定义一个叫做`name`的方法？”斯嘉丽问。

“完全正确，”达格龙说道，屏幕上的代码在她的爪子下发生了变化：

another_minstrel.rb

```

class Minstrel
  def initialize(name)
    @name = name
  end

  def name
    @name
  end
end

```

“现在我们有了一个返回`@name`实例变量的`name`方法，”达格龙说道。“让我们看看当我们创建一个带有这个`name`方法的新吟游诗人并尝试使用它时会发生什么！”

```

>> **load 'another_minstrel.rb'**
=> true
>> **wherefore = Minstrel.new('Wherefore')**
=> #<Minstrel:0x000001049637c8 @name="Wherefore">
>> **wherefore.name**
=> "Wherefore"

```

“万岁！”国王喊道。“我们成功了！我们通过`name`方法改变了吟游诗人的名字。”

“真是太棒了，”Wherefore 说道，“但是如果我们想把吟游诗人的名字改成别的呢？”

“好吧，”达格龙说道，“让我们看看能不能用现在的代码做到这一点。”她在计算装置的发光屏幕上添加了更多代码：

```

>> **wherefore.name = 'Stinky Pete'**
NoMethodError: undefined method `name=' for
#<Minstrel:0x000001049637c8 @name="Wherefore">

```

“我们可以*获取*名字，”达格龙说道，“但我们不能*改变*它；Ruby 抱怨我们的实例没有方法可以改变名字。它在寻找一个我们还没写的方法！”

鲁本仔细研究了屏幕。“又是那个`NoMethodError`，”他说。“看起来 Ruby 想让`Minstrel`类有一个叫做`name=`的方法！”

达格龙点了点头。“如果我们想要*改变*`@name`，我们需要写一个名为`name=`的特殊方法，”她说。“如果你在方法名后面加上等号，Ruby 会理解为：‘我想让这个方法改变某个东西的值。’所以为了改变`@name`，”她补充道，“我们需要添加一些额外的代码。”

她将`name=`方法添加到剩余的代码中，大家都看到了：

another_minstrel_2.rb

```

class Minstrel
  def initialize(name)
    @name = name
  end

  def name
    @name
  end

  def name=(new_name)
    @name = new_name
  end
end

```

“现在我们有了一个新的方法，`name=`，它接受一个参数，`new_name`，”达戈龙说道。“这应该告诉 Ruby，允许我们通过调用`wherefore.name = '`*`some new name`*`'`来更改名字！我们来试试。首先，我们创建一个新的吟游诗人。”

```

>> **load 'another_minstrel_2.rb'**
=> true
>> **wherefore = Minstrel.new('Wherefore')**
=> #<Minstrel:0x000001049637c8 @name="Wherefore">
>> **wherefore.name**
=> "Wherefore"

```

“接下来，我们将尝试更改它的名字。”

```

>> **wherefore.name = 'Stinky Pete'**
=> "Stinky Pete"

>> **wherefore.name**
=> "Stinky Pete"

```

“太棒了！”鲁本说道。“不过写这些方法来获取和设置实例变量真是辛苦。有没有更快的方法呢？”

达戈龙点了点头。“其实是有的，”她说。“有三种内置的快捷方法来读取和写入实例变量：`attr_reader`、`attr_writer`和`attr_accessor`。它们是这样工作的。”她用爪子触碰了计算装置，出现了这些文字：

another_minstrel_3.rb

```

class Minstrel
  attr_accessor :name
  attr_reader :ballad

  def initialize(name)
    @name = name
    @ballad = 'The Ballad of Chucky Jim'
  end
end

```

“举个例子，如果你将符号`:name`传递给`attr_reader`，它会自动创建一个叫做`name`的方法，用来读取实例变量`@name`。`attr_writer`会自动创建一个叫做`name=`的方法，用来改变`@name`的值，而`attr_accessor`则会同时创建`name`和`name=`这两个方法。”达戈龙点击了她的爪子。“在这个例子中，我用`:name`调用了`attr_accessor`，用`:ballad`调用了`attr_reader`，这意味着我可以既获取又更改吟游诗人的名字，但只能读取他的`ballad`，而不能修改它。让我们创建一个新的吟游诗人来测试一下。”

```

>> **load 'another_minstrel_3.rb'**
=> true
>> **wherefore = Minstrel.new('Wherefore')**
=> #<Minstrel:0x0000010413c0e0 @name="Wherefore", @ballad="The
Ballad of Chucky Jim">

```

“太完美了，”达戈龙说道。“让我们看看`attr_accessor`能不能让我们像之前一样获取和更改那位吟游诗人的`name`。”

```

>> **wherefore.name**
=> "Wherefore"

>> **wherefore.name = 'Wherefive'**
=> "Wherefive"

>> **wherefore**
=> #<Minstrel:0x0000010413c0e0 @name="Wherefive", @ballad="The
Ballad of Chucky Jim">

```

“现在让我们看看是否能读取吟游诗人的`ballad`，但不改变它；这就是`attr_reader`应该做的事情，”达戈龙说道。她在计算装置上填入了更多代码：

```

>> **wherefore.ballad**
=> "The Ballad of Chucky Jim"

>> **wherefore.ballad = 'A Song of Mice and Friars'**
NoMethodError: undefined method `ballad=' for
#<Minstrel:0x0000010413c0e0>

```

Wherefore 震惊地摇了摇头。“太不可思议了！”他说。“有了这些 Ruby 工具，我马上就能写出歌曲来。”

“这是 Ruby 中最神奇的部分之一，”Off-White 骑士说道。“当我们围绕对象设计程序时，我们就在做一种叫做*面向对象编程*的事情，它让我们能够编写描述现实世界事物（如吟游诗人和歌曲）的程序。一切都变得轻松了千倍！”

“这太棒了，真是太棒了，”Wherefore 说道。“我真是不知道该怎么感谢你们。怎么才能报答你们呢？”

“嗯，”斯嘉丽说，“其实，我们是来找你问问，是否注意到王国里发生了什么异常的事情。王国各地的 Ruby 系统整整一天都在崩溃，我们开始觉得这些问题可能不是偶然的。”

“把鳞片给他看！”鲁本说。

“哦，太棒了！”斯嘉丽说着，从口袋里拿出了那片闪闪发光的绿色鳞片。“你见过这样的东西吗？我们一开始以为它可能属于达戈龙，但我们检查过了，她并没有少一片。”

“嗯，”Wherefore 说，“真是个难题。不，我想我从没见过有什么生物有像这样的鳞片，但我*确实*在一小时前看到过一些奇怪的东西，在松树林里。”

国王、鲁本和斯卡利特交换了吃惊的眼神。

“是什么？”斯卡利特问。

“嗯，”威尔福说，“我只听到了一小段对话，不过是几个人低声在那片灌木丛后说话。我去看看发生了什么，但当我靠近时，他们就跑了——三个人，可能四个，”他说道。

![image with no caption](img/httpatomoreillycomsourcenostarchimages2160037.png.jpg)

“他们是谁？”国王问。

“我没看清，”威尔福说，“但我听到的那部分确实相当卑鄙。他们说什么没造成足够的影响，打算去找王后谈谈他们正在做的事情。我敢打赌，当他们逃跑时，肯定是直奔城堡去了！”

“城堡！王后！”国王喊道。“天哪，天哪！如果这些就是我们的破坏者，王后可能处于极大的危险之中！”

“我们必须尽快回去！”斯卡利特说道。“白骑士，达格龙，你们能帮我们吗？”

骑士若有所思地皱起了眉头。“我有责任留在松林中，帮助任何迷路的人，”她说，“但我的职责同样属于国王和王后。我可以尽快传达消息，告诉大家麻烦已经出现，并派遣尽可能多的朋友去城堡！”

“请说！”斯卡利特说道，“那你呢，达格龙？”

达格龙摇了摇头。“魔法和智慧都有代价，”她说，“我不能离开红松林。但是，*有*一条通往城堡的捷径。”

“哪里？”鲁本问。

“地下通道！”威尔福说。“是的，我知道。跟我来，我带你们去！”

国王、斯卡利特和鲁本感谢了白骑士和达格龙，挥手告别后急忙赶上已经走到草地半程的威尔福。它们都飞快地沿着一条弯曲的小路冲过去，路旁是树根和交织的树干，几分钟气喘吁吁的奔跑后，他们来到了那棵巨大的红松树，比周围的任何一棵都大，远远望去一眼难见尽头。

“下去了！”威尔福喊道，并且在树干上敲了三下。随着一声愉快的*叮*，一扇门在树干侧面打开，露出了一个狭小的电梯厢。

“乘电梯下到次次地下室，”他说着，把他们三个塞了进去。“你会看到一条狭长的通道，通向西方。走到尽头后，找一个大黑管子。上面会写着——”

“—*神秘的管道!*”鲁本和斯卡利特异口同声地喊道。“我们今天早些时候看到哈尔多消失在城堡的下层，这条通道肯定是通向同一个地方！”

“那你就知道路了！”威尔福说。“再见，祝好运——同时，我会帮白骑士和达格龙尽快把援助送到你们那儿。”话音刚落，电梯门紧闭，国王、斯卡利特和鲁本开始向地下深处下降。

# 《拨打旋律》，或者说是“吟游诗人的快递服务”

现在我们已经向《四处流浪的歌手》讲解了 Ruby 中的对象和类，是时候帮助他创建他自己的`Ballad`（歌曲）了！否则，他就不算是一个真正的歌手了。不过别担心——既然你已经了解了类及其工作原理，创建一个简单的类来帮助《四处流浪的歌手》创作更快、更好的歌曲将不再是难题。

让我们从创建一个新的文件 ballad.rb 并输入以下代码开始。

ballad.rb

```

  class Ballad
➊   attr_accessor :title
    attr_accessor :lyrics

➋   @@number_of_ballads = 0

➌   def initialize(title, lyrics='Tralala!')
      @title = title
      @lyrics = lyrics
      @@number_of_ballads += 1
    end

➍   def self.number_of_ballads
      @@number_of_ballads
    end
  end

➎ ballad = Ballad.new('The Ballad of Chucky Jim')

➏ puts "Number of ballads: #{Ballad.number_of_ballads}"
  puts "Ballad object ID: #{ballad.object_id}"
  puts "Ballad title: #{ballad.title}"
  puts "Ballad object ID again!: #{ballad.object_id}"
  puts "Ballad lyrics: #{ballad.lyrics}"

```

难以置信的是，现在你已经学会了这么多 Ruby，实际上这里没有什么新内容！你以前见过这些东西：创建类和类的实例，使用`attr_accessor`，使用类和实例变量，给类和实例添加方法，所有这些。让我们逐行查看并看看输出。

首先，我们在 ➊ 创建一个`Ballad`类，拥有一个`title`（标题）和`lyrics`（歌词），我们可以读取并修改它们（感谢`attr_accessor`）。

接下来，在 ➋，我们设置了一个类变量`@@number_of_ballads`，用来跟踪我们的类创建了多少首歌曲，而我们的`initialize`方法在 ➌ 同时设置歌曲的名称和歌词，并将`@@number_of_ballads`加 1。

对于类定义的最后部分，我们在 ➍ 为`Ballad`类本身添加了一个`number_of_ballads`方法，这将让我们稍后访问`@@number_of_ballads`。

最后，我们在 ➎ 使用`Ballad.new`创建一首新的歌曲，然后在 ➏ 打印出一些关于我们歌曲的有趣事实。

你可以通过使用终端进入保存*ballad.rb*文件的文件夹，然后在命令行输入**`ruby ballad.rb`**来运行文件中的代码。

你的对象 ID 会和我的稍微不同，但你应该能看到类似这样的内容：

```

Number of ballads: 1
Ballad object ID: #<Ballad:0x0000010413e0e0>
Ballad title: The Ballad of Chucky Jim
Ballad object ID again!: #<Ballad:0x0000010413e0e0>
Ballad lyrics: Tralala!
=> nil

```

我们刚刚证明了`self.number_of_ballads`方法有效，我们的对象 ID 在创建对象后不会改变，并且通过`attr_accessor`的魔力，我们可以访问我们在歌曲中存储的所有信息。

这些都没问题，但*真正*有趣的部分是如何进一步拓展它！例如，你可以从小处开始，编写代码来修改你创建的歌曲的标题，或者在创建后更新其歌词。（你觉得这会改变对象 ID 吗？）

你还可以添加更多的`attr_reader`、`attr_writer`或`attr_accessor`。你可以添加更多的方法（比如创建一个`playing_time`方法来返回歌曲的时长是多少分钟？）。你还可以添加类方法或创建额外的歌曲。

你甚至可以迎接最大的挑战：实际写出《查基·吉姆的歌》！世界是你的牡蛎。（如果你不喜欢牡蛎，那世界就是你的杯子蛋糕。）

# 你知道这些！

你在这一章学到了不少内容，但远远不及你在学习方法时那样充实！了解对象和类几乎就像度假一样轻松！即便如此，我们还是花点时间再回顾一遍，确保你已经完全掌握了。

## 对象和类

你已经知道几乎所有的东西在 Ruby 中都是对象，但在这一章中，你学习了更多关于对象的内容，并仔细查看了对象 ID。对象的 ID 号就像指纹：每个对象都有自己独一无二的 ID，两个对象不会有完全相同的 ID。一般来说，Ruby 创建的对象 ID 号比你创建的对象 ID 号要低：

```

>> **0.object_id**
=> 1
>> **:minstrel.object_id**
=> 465608

```

我们还看到，*类*是我们创建一堆具有相似特征的对象的方式。我们通过`class`关键字创建类，如下所示：

```

>> **class Monkey**
>>   **# Class magicks go here!**
>> **end**

```

创建类本身很好，但类在我们*实例化*（创建）该类的对象之前，并不会为我们做太多事情。你可以把类想象成饼干模具，把它们创建的对象想象成饼干：饼干模具（类）做了一堆非常相似的东西，但我们最感兴趣的其实是饼干本身（对象）。

比如，我们可以用`class`关键字定义一个`Monkey`类，然后通过调用`Monkey.new`来实例化它——也就是从我们的`Monkey`类饼干模具中做出一个*特定*的猴子：

monkey_review.rb

```

class Monkey
  @@bananas = 5

  def initialize(name)
    @name = name
  end

  def eat_banana
    @@bananas -= 1
    puts "Ate a banana! #{@@bananas} left."
  end
end

```

很好！到目前为止，我们已经有了一个`Monkey`类，里面有两个方法和一个类变量。类变量`@@bananas`跟踪所有猴子实例的香蕉数量，`initialize`方法在调用`Monkey.new`时设置猴子的名字，`eat_banana`方法将`@@bananas`减少 1。

接下来，让我们创建几只猴子：

```

>> **load 'monkey_review.rb'**
=> true

>> **socks = Monkey.new('Socks')**
=> #<Monkey:0x000001052c77b0 @name="Socks">

>> **stevie = Monkey.new('Stevie')**
=> #<Monkey:0x00000104ca38e8 @name="Stevie">

```

现在我们可以让每只猴子吃个香蕉，看看会发生什么：

```

>> **socks.eat_banana**
Ate a banana! 4 left.
=> nil
>> **stevie.eat_banana**
Ate a banana! 3 left.
=> nil

```

你注意到每次*任何*猴子实例吃香蕉时，我们的`Monkey`类的`@@bananas`类变量都会减少吗？记住，这是因为类变量是该类所有实例共享的。

我们可以在类中结合使用局部变量、实例变量、类变量和全局变量，如下所示：

monkey_review_2.rb

```

class Monkey
  $home = 'the jungle'
  @@number_of_monkeys = 0

  def initialize(type)
    @type = type
    @@number_of_monkeys += 1
    puts "Made a new monkey! Now
  end
end

```

在这里，我们已经把`Monkey`类修改为拥有一个全局的`$home`变量（`'the jungle'`），一个`@@number_of_monkeys`类变量，用来跟踪`Monkey`类创建了多少个实例，还有一个`@type`实例变量，让每只猴子都有不同的类型。

```

>> **load 'monkey_review_2.rb'**
=> true

>> **blue = Monkey.new('blue monkey')**
Made a new monkey! Now there's 1.
=> #<Monkey:0x00000104aafb40 @type="blue monkey">

>> **silver = Monkey.new('silver monkey')**
Made a new monkey! Now there's 2.
=> #<Monkey:0x00000104ab3b28 @type="silver monkey">
>> **gold = Monkey.new('golden monkey')**
Made a new monkey! Now there's 3.
=> #<Monkey:0x00000104ab7c00 @type="golden monkey">

```

看看每个`@type`是如何对每只猴子独一无二的，但它们都在改变同一个`@@number_of_monkeys`变量吗？

最后，由于`$home`是全局变量，程序的每个部分也都可以访问它：

```

>> **puts "Our monkeys live in #{$home}."**
Our monkeys live in the jungle.
=> nil

```

## 变量和作用域

这一切可能有点难以理清楚，所以我创建了下面这张方便的表格来帮助你记住局部、全局、实例和类变量之间的区别。

| 变量类型 | 形式 | 能在哪里看到？ |
| --- | --- | --- |
| 局部 | `odelay` | 在定义它的顶级作用域、方法或类内部。 |
| 全局 | `$odelay` | 任何地方！ |
| 实例 | `@odelay` | 在定义它的类内部或该类的任何实例中。每个实例都有自己的副本。 |
| 类别 | `@@odelay` | 在类内部或者类的任何实例中。每个实例与所有其他实例共享相同的类变量。 |

请记住，通常不建议使用全局变量，因为它们不仅在程序中的任何地方都是可见的，而且还可以在程序中的任何地方被*修改*。当变量可以在许多地方被修改时，若发生意外情况，可能很难弄清楚是程序中的哪个部分做出了修改。我展示给你全局变量，是为了让你了解它们是什么以及如何工作，但在几乎所有情况下，它们带来的麻烦远大于它们的价值。

正如你在上一个示例中看到的，我们可以从类定义外部访问`$home`变量，因为它被定义为全局变量（全局变量以`$`开头）。我们只有在变量处于正确的作用域中时，才能访问它。让我们回顾一下本章早些时候的一些示例：

```

>> **local_variable = 'Local here!'**
=> "Local here!"

```

我们的`local_variable`存在于这个外部作用域中，但它在类定义内部并不存在：

```

>> **class OutOfTowner**
>>   **puts local_variable**
>> **end**
NameError: undefined local variable or method `local_variable' for
OutOfTowner:Class

```

`local_variable`在方法定义内部也不存在！

```

>> **def tourist**
>>   **puts "Can you take our picture, #{local_variable}?"**
>> **end**

>> **tourist**
NameError: undefined local variable or method `local_variable' for
main:Object

```

我们的变量`number`存在于块内，但一旦块的代码执行完，它就消失了：

```

>> **3.times { |number| puts number }**
0
1
2
=> 3
>> **puts number**
NameError: undefined local variable or method `number' for
main:Object

```

我们发现 Ruby 有一个内建变量`self`，它指代方法将要调用的对象，我们可以使用`self`直接向类中添加方法（而不仅仅是向它们创建的对象添加方法），如下所示：

monkey_review_3.rb

```

class Monkey
  @@number_of_monkeys = 0

  def initialize
    @@number_of_monkeys += 1
  end

  def self.number_of_monkeys
    @@number_of_monkeys
  end
end

```

你之前见过这个！这是我们的`Monkey`类，它有一个`@@number_of_monkeys`类变量，一个`initialize`方法，每次我们创建一个新的猴子时都会增加这个变量，还有一个`self.number_of_monkeys`方法，这意味着我们可以调用`Monkey.number_of_monkeys`来查看我们到目前为止创建了多少个猴子：

```

>> **load 'monkey_review_3.rb'**
=> true
>> **Monkey.number_of_monkeys**
=> 0

```

它现在是`0`，但是如果我们创建一个猴子，我们会看到这个数字上升！

```

>> **monkey = Monkey.new**
=> #<Monkey:0x0000010490af60>
>> **Monkey.number_of_monkeys**
=> 1

```

如果你不确定程序中特定部分的`self`值是什么，你可以随时使用`puts self`来查看它是什么。

我们还学到，如果一个对象有一个实例变量，我们想查看或修改它，我们必须编写方法来实现。我们*可以*像下面这样自己编写这些方法：

minstrel_review.rb

```

class Minstrel
  def initialize(name)
    @name = name
  end

  def name
    @name
  end

  def name=(new_name)
    @name = new_name
  end
end

```

在这里，我们在`initialize`方法中设置了`@name`，这意味着每次我们调用`Minstrel.new`时，我们都会传入该游吟诗人的名字。`name`方法会获取这个`@name`变量，而`name=`方法允许我们为游吟诗人分配一个`new_name` . . .

. . . 但我们也可以使用快捷方式`attr_reader`（用于读取实例变量）、`attr_writer`（用于修改实例变量）和`attr_accessor`（同时做两者）。我们所做的就是将实例变量名作为符号传递，例如：

minstrel_review_2.rb

```

class Minstrel
  attr_accessor :name
  attr_reader :ballad

  def initialize(name)
    @name = name
    @ballad = 'The Ballad of Chucky Jim'
  end
end

```

在这里，我们使用了`attr_accessor`并传入`:name`符号，它会自动为我们创建`name`和`name=`方法；我们用`attr_reader`和`:ballad`调用，所以我们只会得到一个读取`@ballad`实例变量的`ballad`方法。看看如果我们尝试更改我们的歌谣会发生什么！

```

>> **load 'minstrel_review_2.rb'**
=> true

>> **wherefore = Minstrel.new('Wherefore')**
=> #<Minstrel:0x0000010413c0e0 @name="Wherefore", @ballad="The
Ballad of Chucky Jim">

>> **wherefore.ballad**
=> "The Ballad of Chucky Jim"

>> **wherefore.name**
=> "Wherefore"

>> **wherefore.name = 'Wherefive'**
=> "Wherefive"

>> **wherefore.ballad = 'A Song of Mice and Friars'**
NoMethodError: undefined method `ballad=' for
#<Minstrel:0x0000010413c0e0>

```

## 面向对象编程

最后，我们学到，编写围绕类和对象展开的程序叫做*面向对象编程*（*OOP*）。我们的`minstrel`是一个很好的对象示例：一段行为像现实世界中的某个东西的代码！它有属性（关于它自己的事实）以及行为，后者就是指对象知道如何使用的方法。我们可以用`Minstrel`类来定义任何吟游诗人的行为，如下所示。

minstrel_review_3.rb

```

class Minstrel
  attr_reader :name

  @@number_of_minstrels = 0

  def initialize(name)
    @name = name
    @@number_of_minstrels += 1
  end

  def sing(song_name)
    puts "Time to sing a song called: #{song_name}!"
    puts 'Tralala!'
  end

  def self.number_of_minstrels
    @@number_of_minstrels
  end
end

```

我们的类有一个`attr_reader`，用于`：name`（这意味着我们可以读取名称，但不能修改它），还有一个`@@number_of_minstrels`类变量，用于跟踪我们创建了多少实例，以及一个`initialize`方法，给我们的吟游诗人起名字并增加`@@number_of_minstrels`。

它还有两个方法：一个是`sing`，是一个吟游诗人实例的方法，演唱一首小歌；另一个是`self.number_of_minstrels`，是`Minstrel`类的方法，告诉我们到目前为止我们创建了多少个吟游诗人。

让我们看看它们的实际应用吧！

```

>> **load 'minstrel_review_3.rb'**
=> true

>> **wherefore = Minstrel.new('Wherefore')**
=> #<Minstrel:0x000001031eac68 @name="Wherefore">
>> **Minstrel.number_of_minstrels**
=> 1

wherefore.sing('A Tail of Two Foxes')
Time to sing a song called: A Tail of Two Foxes!
Tralala!
=> nil

```

看！我们可以创建一个新的吟游诗人，调用`Minstrel.number_of_minstrels`查看我们创建了一个，然后调用我们的吟游诗人实例（`wherefore`）的`sing`方法，听他唱他的“狐狸的故事”。

事情开始变得有些悬疑了，所以我要去拿一包爆米花—马上回来。与此同时，去看看国王、Scarlet 和 Ruben 回到城堡后会发现什么，并准备好迎接更多面向对象的 Ruby 魔法！
