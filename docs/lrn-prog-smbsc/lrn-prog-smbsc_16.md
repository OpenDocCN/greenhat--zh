## 第十七章：16

**使用关联数组存储数据**

![image](img/common01.jpg)

在像 Facebook 和 LinkedIn 这样的社交网站上，人们会在文本框中输入信息，比如他们的名字、关系状态，甚至是定期向朋友更新（比如，“哦不！！我刚踩到了一只虫子，我觉得我得了虫子中毒！”）。需要搜索或过滤这些数据的程序可能会使用关联数组来存储文本的各个部分。

除了在第十五章中使用的索引数组外，Small Basic 还支持其他类型的数组，这些数组可以简化许多编程任务。在本章中，你将从学习*关联数组*开始。接着，你将学习`Array`对象，使用它创建一些有趣的应用，甚至将你的计算机变成一位诗人！

### 关联数组

在前一章中，你学习了如何使用整数索引来访问数组的元素。但在 Small Basic 中，数组的索引也可以是字符串。由字符串索引的数组被称为*关联数组*、*映射*或*字典*。在本书中，我们称它们为关联数组。就像索引数组一样，关联数组可以存储任何类型的值。你可以使用关联数组在一组*键*（字符串索引）和一组值之间创建关联，这就是创建键值对*映射*。

以下代码展示了关联数组在实际应用中的一个简单例子。这是一个由两位字母缩写键控的州列表：

```
state["CA"] = "California"
state["MI"] = "Michigan"
state["OH"] = "Ohio"
' ... and so on
```

要显示一个州的名称，你只需要使用其对应的键和正确的语法。例如，要显示`Michigan`，你可以写下这个语句：

```
TextWindow.WriteLine(state["MI"])
```

通过写出数组的名称，后跟用方括号括起来的键，你可以访问对应的项。关联数组就像一个*查找表*，将键映射到值；如果你知道键，就可以非常快速地找到其对应的值。

要学习如何使用关联数组，让我们编写一个程序，通过名字追踪你朋友的年龄。在清单 16-1 中输入该程序。

```
1 ' AssociativeArray.sb
2 age["Bert"] = 17
3 age["Ernie"] = 16
4 age["Zoe"] = 16
5 age["Elmo"] = 17
6 TextWindow.Write("Enter the name of your friend: ")
7 name = TextWindow.Read()
8 TextWindow.Write(name + " is [")
9 TextWindow.WriteLine(age[name] + "] years old.")
```

*清单 16-1：使用关联数组*

第 2 行到第 5 行创建了一个名为`age`的关联数组，其中包含四个元素。如果你愿意，可以添加更多元素，或者你可以更改数组来存储你自己朋友的年龄。第 6 行让你输入一个朋友的名字，第 7 行将其读取到`name`变量中。在第 9 行，`age[name]`查找该朋友的年龄。

让我们看一下这个程序的一些示例运行：

```
Enter the name of your friend: Ernie
Ernie is [16] years old.

Enter the name of your friend: ernie
ernie is [16] years old.
```

请注意，键是大小写不敏感的：无论你输入`age["Ernie"]`、`age["ernie"]`，还是`age["ERNIE"]`，都没关系。如果数组包含名为`Ernie`的键，无论其大小写如何，Small Basic 都会返回该键的值。

假设你忘记了在数组中存储了哪些朋友的名字，并且你试图访问一个你忘记包括的朋友的年龄：

```
Enter the name of your friend: Grover
Grover is [] years old.
```

如果数组中不包含某个键，Small Basic 会返回一个空字符串，这就是为什么`age["Grover"]`是空的原因。

**关联数组与 IF/ELSEIF 阶梯**

在编程中，通常有多种不同的方式来解决特定的问题。这里有另一种写法，类似于 Listing 16-1 中的程序：

```
TextWindow.Write("Enter the name of your friend: ")
name = TextWindow.Read()
If (name = "Bert") Then
  age = 17
ElseIf (name = "Ernie") Then
  age = 16
ElseIf (name = "Zoe") Then
  age = 16
ElseIf (name = "Elmo") Then
  age = 17
Else
  age = ""
EndIf
TextWindow.WriteLine(name + " is [" + age + "] years old.")
```

尽管这个程序看起来与 Listing 16-1 中的程序类似，但两者有一个重要区别：在这里，字符串比较是区分大小写的。如果你输入`ernie`（小写的*e*），程序将显示如下输出：

```
ernie is [] years old.
```

表达式`If("ernie" = "Ernie")`为假。这个版本的程序也更难以阅读和编写。当你需要在一组键和值之间进行映射时，最好使用关联数组，这样你就不必担心大小写问题。

### 使用关联数组

现在你已经理解了关联数组的基础知识，让我们看几个程序示例，展示如何使用它们。

#### *法语中的星期

第一个示例将星期几从英语翻译成法语。这个程序提示用户输入一个英文的星期几名称，并输出该名称的法语翻译。请在 Listing 16-2 中输入代码。

```
 1 ' FrenchDays.sb
 2 day["Sunday"] = "Dimanche"
 3 day["Monday"] = "Lundi"
 4 day["Tuesday"] = "Mardi"
 5 day["Wednesday"] = "Mercredi"
 6 day["Thursday"] = "Jeudi"
 7 day["Friday"] = "Vendredi"
 8 day["Saturday"] = "Samedi"
 9
10 TextWindow.Write("Enter the name of a day: ")
11 name = TextWindow.Read()
12 TextWindow.WriteLine(name + " in French is " + day[name])
```

*Listing 16-2: 一个英法翻译程序*

`day`数组存储了星期几的法语名称（第 2-8 行）。数组中的每个键是该天的英文名称。程序提示用户输入一个英文的星期几名称（第 10 行），并将用户的输入存储在`name`变量中（第 11 行）。然后，程序使用用户的输入作为键，通过语法`day[name]`查找对应的法语名称，并显示它（第 12 行）。以下是一次示例运行的输出：

```
Enter the name of a day: Monday
Monday in French is Lundi
```

你会其他语言吗？修改程序，帮助你的朋友们学习如何用一种新语言说出星期几。想要调皮一下吗？你甚至可以编造一个自己的秘密语言！

**尝试一下 16-1**

如果用户输入一个无效的星期名称（比如*Windsday*），Listing 16-2 的输出会是什么？当发生这种情况时，更新程序以显示错误信息。使用如下的`If`语句：

```
If (day[name] = "") Then
  ' Tell the user they entered a wrong name
Else
  ' Show the French translation
EndIf
```

#### *存储记录*

生意兴隆，你所在城镇的本地草坪修剪服务公司 Moe Mows 雇佣你编写一个程序，用于显示其客户的联系信息。当公司输入客户的姓名时，程序需要显示客户的家庭地址、电话号码和电子邮件地址。请在 Listing 16-3 中输入该程序。

```
 1 ' MoeMows.sb
 2 address["Natasha"] = "3215 Romanoff Rd"
 3 phone["Natasha"] = "(321) 555 8745"
 4 email["Natasha"] = "blackwidow64@shield.com"
 5
 6 address["Tony"] = "8251 Stark St"
 7 phone["Tony"] = "(321) 555 4362"
 8 email["Tony"] = "ironman63@shield.com"
 9
10 TextWindow.Write("Name of customer: ")
11 name = TextWindow.Read()
12 TextWindow.WriteLine("Address...: " + address[name])
13 TextWindow.WriteLine("Phone.....: " + phone[name])
14 TextWindow.WriteLine("Email.....: " + email[name])
```

*Listing 16-3: 构建一个简单的数据库*

该程序使用了三个关联数组：`address`、`phone` 和 `email`。这三个数组都以客户的名字作为键，数组共同用来存储客户的记录。*记录* 是一组相关的数据项。在这个例子中，每个客户的记录有三个字段：地址、电话和电子邮件。无论程序有两个记录还是 1,000 条记录，搜索的方式都是一样的。例如，第 12 行的语句 `address[name]` 返回与 `address` 数组中 `name` 键关联的值。我们不需要自己去搜索 `address` 数组；Small Basic 会为我们做这一切，完全免费！

这是这个程序示例运行的输出：

```
Name of customer: Tony
Address...: 8251 Stark St
Phone.....: (321) 555 4362
Email.....: ironman63@shield.com
```

**动手实践 16-2**

更新 列表 16-3 中的程序，将一些朋友的联系信息存储在其中（但不是你所有 500 个 Facebook 朋友的信息）。再添加一个数组，用来存储每个朋友的生日。你再也不会忘记生日了！

### 数组对象

Small Basic 库中的 `Array` 对象可以帮助你找到程序中数组的重要信息。在本节中，我们将详细探讨这个对象，并查看一些如何使用它的示例。要探索 `Array` 对象，让我们首先输入以下代码：

```
name = "Bart"         ' An ordinary variable
age["Homer"] = 18     ' An associative array with two elements
age["Marge"] = 17
score[1] = 90         ' An indexed array with one element
```

这段代码定义了一个普通变量 `name`，一个名为 `age` 的关联数组，包含两个元素，还有一个名为 `score` 的索引数组，包含一个元素。你将在接下来的例子中使用这些数组。`Array` 对象能告诉你什么？让我们来看看！

#### *它是一个数组吗？*

你认为 Small Basic 知道 `name` 是一个普通变量，而 `age` 和 `score` 是数组吗？运行 列表 16-4 中的程序来找出答案。

```
1 ' IsArray.sb
2 name = "Bart"
3 age["Homer"] = 18
4 age["Marge"] = 17
5 score[1] = 90
6 ans1 = Array.IsArray(name)       ' Returns "False"
7 ans2 = Array.IsArray(age)        ' Returns "True"
8 ans3 = Array.IsArray(score)      ' Returns "True"
9 TextWindow.WriteLine(ans1 + ", " + ans2 + ", " + ans3)
```

*列表 16-4：演示* `IsArray()` *方法*

这段代码使用了 `Array` 对象的 `IsArray()` 方法。如果变量是数组，该方法返回 `"True"`；否则返回 `"False"`。这个方法表明变量 `age` 和 `score` 是数组，但变量 `name` 不是数组。`IsArray()` 方法可以帮助你确保程序中的变量是数组。

#### *数组有多大？*

`Array` 对象还可以告诉你数组中存储了多少元素。运行 列表 16-5 中的程序。

```
1 ' GetItemCount.sb
2 name = "Bart"
3 age["Homer"] = 18
4 age["Marge"] = 17
5 score[1] = 90
6 ans1 = Array.GetItemCount(name)       ' Returns: 0
7 ans2 = Array.GetItemCount(age)        ' Returns: 2
8 ans3 = Array.GetItemCount(score)      ' Returns: 1
9 TextWindow.WriteLine(ans1 + ", " + ans2 + ", " + ans3)
```

*列表 16-5：演示* `GetItemCount()` *方法*

`GetItemCount()` 方法返回指定数组中的项目数量。注意 `GetItemCount(name)` 返回 0，因为 `name` 不是一个数组。其他两个调用返回每个数组中的元素数量。使用 `GetItemCount()` 来跟踪你在数组中存储了多少项。你可能会在一个允许玩家将物品存入背包的游戏中使用此方法，并且你希望检查他们捡到了多少物品。

#### *它有特定的索引吗？*

你还可以使用 `Array` 对象来检查你的数组是否包含某个特定的索引。要了解如何操作，请运行 清单 16-6 中的程序。

```
 1 ' ContainsIndex.sb
 2 age["Homer"] = 18
 3 age["Marge"] = 17
 4 score[1] = 90
 5 ans1 = Array.ContainsIndex(age, 1)       ' Returns "False"
 6 ans2 = Array.ContainsIndex(age, "homer") ' Returns "True"
 7 ans3 = Array.ContainsIndex(age, "Lisa")  ' Returns "False"
 8 TextWindow.WriteLine(ans1 + ", " + ans2 + ", " + ans3)
 9
10 ans1 = Array.ContainsIndex(score, "1")   ' Returns "True"
11 ans2 = Array.ContainsIndex(score, 1)     ' Returns "True"
12 ans3 = Array.ContainsIndex(score, 2)     ' Returns "False"
13 TextWindow.WriteLine(ans1 + ", " + ans2 + ", " + ans3)
```

*清单 16-6：演示* `ContainsIndex()` *方法*

`ContainsIndex()` 方法接受两个参数。第一个参数是数组的名称，第二个参数是你要检查的索引。该方法会根据索引是否存在于数组中返回 `"True"` 或 `"False"`。

第 6 行显示了搜索索引时是不区分大小写的，这就是为什么搜索索引 `homer` 返回 `"True"`。此外，搜索 `score` 数组中的索引 `"1"`（作为字符串）或索引 `1`（作为数字）都返回了 `"True"`。

如果你不确定某个数组是否包含特定的索引，可以使用 `ContainsIndex()` 方法来查找。这个方法对于处理非常长的数组特别有用。

#### *它是否具有特定的值？*

`Array` 对象还提供了一种方法，用于检查数组是否包含某个特定的值。运行 清单 16-7 中的程序，了解 `ContainsValue()` 方法是如何工作的。

```
1 ' ContainsValue.sb
2 age["Homer"] = 18
3 age["Marge"] = 17
4 score[1] = 90
5 ans1 = Array.ContainsValue(age, 18)   ' Returns "True"
6 ans2 = Array.ContainsValue(age, 20)   ' Returns "False"
7 ans3 = Array.ContainsValue(score, 90) ' Returns "True"
8 TextWindow.WriteLine(ans1 + ", " + ans2 + ", " + ans3)
```

*清单 16-7：演示* `ContainsValue()` *方法*

`ContainsValue()` 方法根据检查的值是否存在于数组中，返回 `"True"` 或 `"False"`。

**注意**

*与* `ContainsIndex()` *方法不同，* `ContainsValue()` *方法是区分大小写的。所以最好保持大小写一致！*

#### *给我所有的索引*

`Array` 对象的另一个有用方法是 `GetAllIndices()`。该方法返回一个包含给定数组所有索引的数组。返回数组的第一个元素的索引为 1。要理解这个方法是如何工作的，请运行 清单 16-8 中的程序。

```
1 ' GetAllIndices.sb
2 age["Homer"] = 18
3 age["Marge"] = 17
4 names = Array.GetAllIndices(age)
5 TextWindow.WriteLine("Indices of the age array:")
6 For N = 1 To Array.GetItemCount(names)
7   TextWindow.WriteLine("Index" + N + " = " + names[N])
8 EndFor
```

*清单 16-8：演示* `GetAllIndices()` *方法*

第 4 行调用 `GetAllIndices()` 来查找 `age` 数组的所有索引。该方法返回一个数组，并将其保存在 `names` 标识符中。接着代码开始一个循环，从 `names` 中的第一个元素运行到最后一个元素。注意代码是如何使用 `GetItemCount()` 方法来计算这个值的。以下是这段代码的输出：

```
Indices of the age array:
Index1 = Homer
Index2 = Marge
```

现在让我们将你学到的方法好好利用一下。你觉得你的电脑足够聪明，能够写诗吗？好吧，我们来看看！

**动手实践 16-3**

打开本章文件夹中的 *AnimalSpeed.sb* 文件。这个游戏会考察玩家不同动物的最高速度（单位为英里每小时）。程序包含一个关联数组，类似于这样：

```
speed["cheetah"] = 70
speed["antelope"] = 60
speed["lion"] = 50
' ... and so on
```

运行这个游戏看看它是如何工作的。这个游戏使用了哪些`Array`对象方法？解释一下游戏的工作原理，然后想一些点子让游戏更有趣。确保完成所有任务。别像猎豹一样偷懒！

### 你的电脑是诗人

现在，让我们运用所学的关联数组知识，编写一个生成诗歌的程序。这个人工诗人从五个列表（`article`，`adjective`，`noun`，`verb` 和 `preposition`）中随机选择单词，并将它们组合成固定的模式。为了给诗歌赋予一个中心主题，这些列表中的所有单词都与爱与自然相关。当然，我们可能还是会得到一些傻乎乎的诗歌，但那也一样有趣！

**注意**

*这个程序的灵感来自于丹尼尔·瓦特的《使用 Logo 学习》（McGraw-Hill, 1983）。*

图 16-1 显示了该应用程序的用户界面。

![image](img/f16-01.jpg)

*图 16-1：Poet.sb 的用户界面*

每次点击“New”按钮时，诗人都会朗诵一首新诗。每首诗包含三行，遵循以下模式：

• 第 1 行：冠词，形容词，名词

• 第 2 行：冠词，名词，动词，介词，冠词，形容词，名词

• 第 3 行：形容词，形容词，名词

接下来的部分将指导你创建这个程序。

#### *步骤 1：打开启动文件*

打开本章文件夹中的 *Poet_Incomplete.sb* 文件。该文件包含一个名为 `CreateLists()` 的子例程，用于创建程序所需的五个列表。添加这个子例程是为了让你不必输入一堆单词。它的内容如下：

```
Sub CreateLists
  article = "1=a;2=the;...;5=every;"
  adjective = "1=beautiful;2=blue;...;72=young;"
  noun = "1=baby;2=bird;...;100=winter;"
  verb = "1=admires;2=amuses;...;92=whispers;"
  prepos = "1=about;2=above;...;37=without;"
EndSub
```

省略号（`...`）表示缺失的数组元素，但当你打开文件时，你可以看到所有这些元素。请注意，article 数组还包括其他限定词，如 one、each 和 every。

#### *步骤 2：设置图形用户界面*

将 清单 16-9 中的代码添加到程序文件的开头，以设置图形用户界面（GUI）并注册按钮的事件处理程序。

```
 1 GraphicsWindow.Title = "The Poet"
 2 GraphicsWindow.CanResize = "False"
 3 GraphicsWindow.Width = 480
 4 GraphicsWindow.Height = 360
 5 GraphicsWindow.FontBold = "False"
 6 GraphicsWindow.FontItalic = "True"
 7 GraphicsWindow.FontSize = 16
 8
 9 path = Program.Directory
10 GraphicsWindow.DrawImage(path + "\Background.png", 0, 0)
11 Controls.AddButton("New", 10, 10)
12
13 CreateLists()
14
15 artCount = Array.GetItemCount(article)
16 adjCount = Array.GetItemCount(adjective)
17 nounCount = Array.GetItemCount(noun)
18 verbCount = Array.GetItemCount(verb)
19 prepCount = Array.GetItemCount(prepos)
20
21 Controls.ButtonClicked = OnButtonClicked
22 OnButtonClicked()
```

*清单 16-9：设置图形用户界面*

程序通过初始化图形窗口（第 1-7 行）、绘制背景图像（第 9-10 行）和创建“New”按钮（第 11 行）开始。接下来，它调用 `CreateLists()` 子例程来初始化五个索引数组（第 13 行）。然后，程序使用 `Array` 对象获取每个数组中的项数，并将这些值保存在第 15-19 行。这样，你就可以在不影响程序其余部分的情况下，向这些数组的末尾添加更多元素。例如，如果你想添加第 73 个形容词，可以在 `CreateLists()` 子例程中的 `adjectives` 数组行末尾加上 `73=callipygous;`。因为第 16 行在 清单 16-9 中获取该数组的元素数量，所以你添加的新元素会自动被计数并随机选入诗歌，就像其他元素一样。

最后，程序为 `ButtonClicked` 事件注册了一个处理程序（第 21 行），并调用该处理程序子例程来显示第一首诗（第 22 行）。

#### *步骤 3：响应按钮点击*

现在你需要添加 `OnButtonClicked()` 子程序，如 列表 16-10 所示。

```
 1 Sub OnButtonClicked
 2   GraphicsWindow.DrawImage(path + "\Background.png", 0, 0)
 3
 4   MakeLine1()  ' Constructs poemLine1
 5   MakeLine2()  ' Constructs poemLine2
 6   MakeLine3()  ' Constructs poemLine3
 7
 8   GraphicsWindow.DrawText(180, 140, poemLine1)
 9   GraphicsWindow.DrawText(100, 165, poemLine2)
10   GraphicsWindow.DrawText(180, 190, poemLine3)
11 EndSub
```

*列表 16-10：* `OnButtonClicked()` *子程序*

这个子程序重新绘制背景图像以清除图形窗口（第 2 行）。接着，它调用三个子程序来生成诗歌的三行内容（第 4-6 行），并将这些行绘制到图形窗口中（第 8-10 行）。接下来，你将添加三个缺失的子程序。

#### *第 4 步：编写诗歌的第一行*

诗歌的第一行采用以下形式：冠词、形容词、名词。添加 列表 16-11 中的子程序，该子程序创建诗歌的第一行并将其赋值给 `poemLine1` 变量。

```
1 Sub MakeLine1
2   art1 = article[Math.GetRandomNumber(artCount)]
3   adj1 = adjective[Math.GetRandomNumber(adjCount)]
4   noun1 = noun[Math.GetRandomNumber(nounCount)]
5   poemLine1 = art1 + " " + adj1 + " " + noun1
6 EndSub
```

*列表 16-11：* `MakeLine1()` *子程序*

`MakeLine1()` 子程序从 `article`、`adjective` 和 `noun` 数组中随机选择三个单词，并将其存储在 `art1`、`adj1` 和 `noun1` 中（第 2-4 行）。然后，它通过在这些变量之间添加空格来填充 `poemLine1`（第 5 行）。

#### *第 5 步：编写诗歌的第二行和第三行*

`MakeLine2()` 和 `MakeLine3()` 子程序与 `MakeLine1()` 子程序非常相似。第二行的形式是：冠词、名词、动词、介词、冠词、形容词、名词。第三行的形式是：形容词、形容词、名词。自己创建这些子程序。如果遇到困难，可以打开文件 *Poet.sb* 查看我们如何编写这些子程序。完成后，把你最喜欢的诗歌输出背诵给家人或朋友听，看看他们是否认为是你写的！

**尝试 16-4**

多次运行你的诗人程序，看看机器诗人能创作出什么样的作品。设计不同的诗歌模式，并教这个诗人如何创作它们。然后，将单词更改为你想要的任何单词（以及任意数量的单词）！前往 *[`tiny.cc/sbpoet/`](http://tiny.cc/sbpoet/)* 与社区分享你的诗歌程序，并看看其他人创作了什么。

**注意**

*`Array`* *对象包括三个创建不同类型数组的方法：* `SetValue()`、`GetValue()` *和* `RemoveValue()`*。尽管这些方法效果很好，但数组的方括号形式在编程语言中更为通用，这也是本书专注于这种形式的原因。*

### 编程挑战

如果遇到困难，请查看 *[`nostarch.com/smallbasic/`](http://nostarch.com/smallbasic/)* 以获取解决方案、更多资源和教师及学生的复习问题。

1.  编写一个程序，记录你朋友的电话号码。使用一个关联数组，将你朋友的名字作为键；例如，`phone["Yoda"] = "555-1138"`。

1.  编写一个程序，保存书籍信息。书籍的关键是 ISBN。对于每本书，你需要知道书名、作者和出版年份。使用三个关联数组：`title[ISBN]`、`author[ISBN]` 和 `year[ISBN]`。

1.  打开本章文件夹中的*VirtualPiano.sb*文件。该程序使用键盘实现了一个虚拟钢琴。解释一下该程序是如何工作的。
