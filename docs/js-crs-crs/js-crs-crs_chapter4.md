

## 第十一章：4 条件语句与循环



![](img/opener.png)

*条件语句* 和 *循环* 是编程中的基本元素。它们通过允许你的代码根据特定条件做出决策，为程序添加了逻辑和结构。条件语句和循环一起被称为 *控制结构*，因为它们让你控制代码的执行时机和频率。通过条件语句，你可以仅在某个条件为真时才运行特定的代码。同时，循环允许你在某个条件为真时反复执行一段代码。

在本章中，你将学习如何使用 if 语句有条件地执行代码，以及如何使用 while 和 for 语句进行代码循环。你还将学习如何在复合数据类型的元素上进行循环。这在你需要对数组或对象的每个元素执行操作时尤其有用。

当我们开始使用控制结构时，我们将编写更复杂的脚本，这些脚本直接在控制台输入时并不实用，因为每个语句一输入就会立即执行。因此，在本章中，我们将切换到将 JavaScript 代码嵌入到 HTML 文件中，然后在浏览器中打开这些文件。这让你可以一次运行整个程序，并且能够轻松地进行修改并重新运行整个程序。要复习如何操作，请参阅 第一章中的“使用文本编辑器”部分。

### 使用条件语句做决策

条件语句允许你在设定的条件为真时运行一段代码。例如，你可能只希望在银行账户余额低于某个阈值时显示警告消息，或者在游戏中，当玩家被敌人击中时失去一条生命。你通常使用比较运算符，如 === 和 >，来创建这些条件，这些我们在 第二章 中已经讨论过。你还可以使用逻辑运算符，如 && 和 ||，将多个条件组合在一起。关键是，整体条件必须评估为真或假。

条件语句有两种主要类型：if 语句和 if…else 语句。我们将依次讨论这两种类型。

#### if 语句

如果某个条件为真，if 语句会执行代码；如果条件为假，它会跳过该段代码。例如，我们可以创建一个程序，当某个值大于指定阈值时，将消息记录到控制台。打开 VS Code，创建一个名为 *if.html* 的新文件，并输入 列表 4-1 的内容。

```
<html><body><script>
let speed = 30;
console.log(`Your current speed is ${speed} mph.`);
❶ if (speed > 25) {
  console.log("Slow down!");
}
</script></body></html> 
```

列表 4-1：一个 if 语句

这段代码开始和结束时使用了第一章中用于嵌入 JavaScript 代码到 HTML 文件的相同标签。JavaScript 本身首先将 speed 变量初始化为 30，并使用 console.log 将该值打印到控制台。然后，我们使用 if 语句 ❶检查 speed 的值，并在其大于 25 时打印另一条消息。

if 语句以 if 关键字开始，主要由两部分组成：*条件*，即括号内的内容；以及当条件为真时执行的代码，称为*语句体*，它被包含在一对大括号内。这里，条件是 speed > 25，如果条件为真，执行的代码是 console.log("慢下来！")。由于我们设置了 speed 大于 25，条件为真，因此语句体中的代码会执行。因此，当你在浏览器中打开*if.html*时，你应该在 JavaScript 控制台中看到以下输出：

```
Your current speed is 30 mph.
Slow down! 
```

我们的条件通过了，因此“慢下来！”的消息被记录到控制台中。然而，如果条件为假，if 语句体中的代码将不会执行。为了亲自验证这一点，尝试将*if.html*中的 speed 值初始化为 20 而不是 30。然后重新保存文件并重新加载页面。这一次，你应该只看到以下输出：

```
Your current speed is 20 mph.
```

因为 speed > 25 现在为假，括号内的代码没有执行。然而，if 语句体外的代码仍然执行，因此我们仍然能看到 speed 的值，这要归功于第一次的 console.log 调用。

#### if…else 语句

当条件为真时，你可能希望运行一段代码，当条件为假时运行另一段代码。为此，我们使用 if…else 语句。试着创建一个名为*ifElse.html*的新文件，并输入列表 4-2 的内容。

```
<html><body><script>
let speed = 20;
console.log(`Your current speed is ${speed} mph.`);
if (speed > 25) {
❶ console.log("Slow down!");
} else {
❷ console.log("You're obeying the speed limit.");
}
</script></body></html> 
```

列表 4-2：一个 if…else 语句

这段代码使用 if…else 语句检查 speed 是否大于 25。与列表 4-1 中的代码一样，条件以 if 关键字开头，后面跟着括号中的条件。然而，不同于列表 4-1，if…else 语句有两个代码块，而不是只有一个，else 关键字位于它们之间。如果条件为真，第一个代码块 ❶会执行；如果条件为假，第二个代码块 ❷会执行。每个代码块都被一对大括号包围。在这个例子中，由于 speed 为 20，条件计算结果为假，因此第二个代码块会执行。当你在 Chrome 中打开文件时，你应该看到以下输出：

```
Your current speed is 20 mph.
You're obeying the speed limit. 
```

else 语句体中的消息已经被记录到控制台中。然而，如果你尝试将 speed 设置为更高的值，比如 30，那么 if 语句体中的消息将会被记录。

#### 更复杂的条件

可以通过结合逻辑运算符，使用更复杂的布尔表达式作为条件。例如，假设你只想在上学时间检查驾驶员的速度。假设你有一个包含当前小时数的小时变量（使用 24 小时制），你可以做如下操作：

```
if (speed > 25 && hour > 7 && hour < 16) {
```

只有当速度大于 25 且小时数大于 7 但小于 16 时，这个 if 语句的主体才会执行。换句话说，如果在上学时间外，即使速度超过 25，也不会执行 if 语句的主体。

如果你的条件变得太复杂，可能会使 if 语句难以阅读。在这种情况下，通常最好将布尔表达式单独写出来，并将其赋值给一个新变量。然后，你可以将这个变量作为 if 语句的条件。例如，之前的条件可以重写为：

```
let tooFastForSchool = speed > 25 && hour > 7 && hour < 16;
if (tooFastForSchool) { 
```

在这里，我们将相同的复杂布尔表达式赋值给 tooFastForSchool 变量，然后将该变量提供给 if 语句。由于变量名具有意义，现在的条件几乎就像一句话：“如果太快而不适合上学，[做某事]。”

如果把速度和小时的测试放到一个布尔变量中显得有些奇怪，那么一个折中的方法是把小时检查单独放入一个变量中，像这样：

```
let schoolHours = hour > 7 && hour < 16;
if (speed > 25 && schoolHours) { 
```

现在，schoolHours 变量根据是否在上学时间内存储真或假，而 if 语句将该变量与速度测试结合。最终，你选择的方法归结为一个主观问题：你觉得这段代码容易阅读吗？

#### 链式 if…else 语句

如果你需要让代码在三个或更多可能的分支之间做出选择，可以将多个 if…else 语句链接在一起。例如，你可以使用这种技巧根据速度变量的值记录三个可能的消息之一。创建一个新文件，命名为*ifElseIf.html*，并使用清单 4-3 中的代码。

```
<html><body><script>
let speed = 20;
console.log(`Your current speed is ${speed} mph.`);
if (speed > 25) {
  console.log("Slow down!");
} else if (speed > 15) {
  console.log("You're driving at a good speed.");
} else {
  console.log("You're driving too slowly.");
}
</script></body></html> 
```

清单 4-3：一个带有三个主体的链式 if…else 语句

这个脚本与清单 4-2 中的 if…else 语句非常相似，不同之处在于现在有三个部分，每个部分都有自己的主体：if、else if 和 else。只有一个主体——第一个条件为真的主体——会运行。下面是它的工作原理：

1. 首先，我们使用 if 来检查速度是否大于 25。如果是，首先的主体将运行，将“减速！”记录到控制台，其余的条件将被跳过。

2.  接下来，我们使用 else if 添加第二个条件，测试速度是否大于 15，并在符合条件时记录不同的信息。如果代码执行到这一点，说明 speed > 25 的条件已经被判断为假，因此实际上 speed > 15 是在测试 speed 是否介于 15 和 25 之间。我们可以通过写 else if (speed > 15 && speed <= 25) 来明确这一点，但因为我们已经知道 speed 不会大于 25，所以不需要指定 && speed <= 25 部分。

3.  最后，我们使用 else 记录第三个可能的消息，如果前两个条件都不成立的话。

在这个例子中，我们将 speed 设置为 20，因此只有 else if 分支会运行，产生以下输出：

```
Your current speed is 20 mph.
You're driving at a good speed. 
```

尝试使用不同的 speed 值，触发 if 和 else 分支。

你可以在初始 if 和最终 else 之间链式添加任意数量的 else if 子句，如 清单 4-4 所示，从而在条件结构中创建任意数量的分支。

```
if (speed > 25) {
  console.log("Slow down!");
} else if (speed > 20) {
  console.log("You're driving at a good speed.");
} else if (speed > 15) {
  console.log("You're driving a little bit too slowly.");
} else if (speed > 10) {
  console.log("You're driving too slowly.");
} else {
  console.log("You're driving far too slowly!");
} 
```

清单 4-4：一个带有五个分支的链式 if…else 语句

这个链式 if…else 语句有五种可能的分支，取决于 speed 是否大于 25、20、15、10，或者这些都不是。与前面的例子一样，条件的顺序在这里很重要。按照从大到小的顺序进行比较，使我们能够定义速度的五个可能值范围，而无需显式定义范围的上限。例如，我们可以写成 else if (speed > 15)，而不是 else if (speed > 15 && speed <= 20)，因为到那时我们已经确认 speed 不大于 20。表 4-1 展示了每个分支的完整条件，清单 4-4 中提供了详细信息。

| 表 4-1： 清单 4-4 中的完整条件和输出 |
| --- |
| 条件 | 输出 |
| speed > 25 | 减速！ |
| speed > 20 && speed <= 25 | 你开得速度刚好。 |
| speed > 15 && speed <= 20 | 你开得有点慢。 |
| speed > 10 && speed <= 15 | 你开得太慢了。 |
| speed <= 10 | 你开得太慢了！ |

请注意，我们可以反转条件和分支的顺序，最终得到相同的效果。反转后，条件将是 speed <= 10、speed <= 15、speed <= 20 和 speed <= 25。speed > 25 的情况会在 else 块中处理。需要注意的是，条件是逐一检查的，因此检查第二个条件意味着第一个条件为假。同时，注意 > 的相反是 <=（想想如果 speed 正好是 10 时，应该触发哪个条件）。

### 使用循环重复代码

循环是 JavaScript 中另一种控制结构，允许你根据需要多次重复执行相同的代码。例如，你可以使用循环打印购物清单中的每一项。如果没有循环，这将是不可能的，因为你事先并不一定知道清单上有多少项。循环在你希望一直运行相同的代码直到某个条件成立时也非常有用；例如，反复要求用户输入他们的出生日期，直到他们提供有效的日期。

在本章中，你将学习四种循环：`while` 循环、`for` 循环、`for…in` 循环和 `for…of` 循环。我们从 `while` 循环开始。

#### while 循环

类似于 `if` 语句，`while` 循环依赖于条件测试。就像 `if` 语句一样，如果条件最初被发现为假，`while` 循环将完全跳过执行其代码。然而，与 `if` 语句不同的是，`while` 循环会在条件为真时继续运行其代码块，在每次新的一轮执行前重新检查条件。换句话说，它会在某个条件为真时重复执行一段代码。这个特性在你需要多次执行某段代码时非常有用，它使得程序可以在需要时一直运行，而不是只执行一次然后停止。

要查看 `while` 循环如何工作，创建一个名为 *while.html* 的新文件，并输入 Listing 4-5 的内容。

```
<html><body><script>
let speed = 30;
❶ while (speed > 25) {
  console.log(`Your current speed is ${speed} mph.`);
  speed--;
}
❷ console.log(`Now your speed is ${speed} mph.`);
</script></body></html> 
```

Listing 4-5: 一个 while 循环

这个脚本将速度设置为 30，然后使用 `while` 循环 ❶ 将速度控制在限制范围内。我们使用 `while` 关键字编写 `while` 循环，后面跟着括号中的条件和大括号中的代码块，类似于 `if` 语句。在这里，我们的条件检查速度是否大于 25。代码块将速度值打印到控制台，然后使用递减运算符（--）将速度减少 1。这为我们提供了一个新的速度值，以便在下一次循环中测试。`while` 循环会不断重复代码块，直到条件为假，输出如下：

```
Your current speed is 30 mph.
Your current speed is 29 mph.
Your current speed is 28 mph.
Your current speed is 27 mph.
Your current speed is 26 mph.
Now your speed is 25 mph. 
```

让我们来思考当这段代码运行时会发生什么。第一次进入 while 循环时，speed 是 30，因此条件（speed > 25）为真。这意味着 while 循环的主体执行一次，输出“当前速度是 30 英里每小时”，并将 speed 从 30 减少到 29。循环主体结束时，我们回到起始位置，重新检查条件。由于 speed 现在是 29，条件仍然为真，因此我们再次执行循环主体，打印“当前速度是 29 英里每小时”，并将 speed 减少到 28。然后我们再次回到起始位置，继续检查条件，依此类推。最终，在第五次循环时，speed 从 26 减少到 25。当我们第六次检查条件时，它的值为假（25 不大于 25）。这使得 JavaScript 停止循环，并跳到 while 循环之后的第一行代码 ❷，输出最后一行文本。

#### for 循环

for 循环是另一种更结构化的 JavaScript 循环方式。像 while 循环一样，for 循环会在某个条件为真时重复执行。但与 while 循环不同，在 for 循环中，管理重复执行的代码出现在循环的开始部分，与循环主体分开。

循环通常有一个特定的*循环变量*，用于跟踪循环的状态。一个常见的模式是将循环变量设置为一个初始值，某种方式更新它，并根据循环变量检查某个条件，决定是否停止重复执行。例如，我们在清单 4-5 中的 while 循环遵循了这个模式，speed 作为循环变量。我们在进入循环之前将 speed 设置为 30，每次通过循环时将 speed 减少，并一直循环，直到 speed 不再大于 25。

for 循环只是写出这个模式的一种更便捷方式。使用 for 循环时，我们将设置和更新循环变量的代码移动到循环的第一行，放在与循环条件相同的括号内。为了说明这一点，让我们将之前的例子重写，使用 for 循环代替 while 循环。将清单 4-6 的内容保存到*for.html*中。

```
<html><body><script>
for (let speed = 30; speed > 25; speed--) {
  console.log(`Your current speed is ${speed} mph.`);
}
</script></body></html> 
```

清单 4-6：一个 for 循环

我们使用 for 关键字声明 for 循环，后面跟着一对括号，其中包含三个组成部分，每个部分都有各自的循环管理任务：

1.  初始化循环变量（let speed = 30）。

2.  设置循环条件（speed > 25）。

3.  更新循环变量（speed--）。更新将在每次循环后进行。

这三个部分用分号隔开。

在循环体内，我们有一条语句将`speed`的值记录到控制台。注意，我们不再需要像在`while`循环中那样在循环体内递减`speed`；这一部分由循环管理代码的第三部分（括号内）来处理。同样，我们不再需要在声明循环之前初始化`speed`；这一点也在括号内处理。

运行这个脚本将产生与列表 4-5 中的`while`循环大致相同的输出：

```
Your current speed is 30 mph.
Your current speed is 29 mph.
Your current speed is 28 mph.
Your current speed is 27 mph.
Your current speed is 26 mph. 
```

唯一的区别是，我们不能像在`while`循环中那样在循环结束后记录最终速度。这是因为`speed`变量作为循环本身的一部分被声明，而不是在循环之前声明。因此，`speed`被限制在循环的*作用域*内，这意味着循环外的代码无法访问它。实际上，这正是`for`循环的一个优势：循环变量仅存在于循环内，无法在代码的其他部分意外地使用或修改。

使用`for`循环，你可以完成所有`while`循环能做的事情，但大多数程序员发现`for`循环比等效的`while`循环更容易阅读，因为所有循环逻辑都集中在一个地方。

#### for…of 循环

`for…of` 循环遍历数组中的项。与`while`循环或`for`循环在某个条件为真时一直循环不同，`for…of`循环逐一遍历数组中的每一项，直到没有剩余的项为止。这非常有用，因为通常需要对数组中的每个成员应用相同的操作。例如，如果你有一个数字数组，你可以通过遍历这些数字并为每个数字绘制一个矩形，使用数字来设置矩形的高度（单位为像素）来创建一个柱状图。类似地，如果你有一个关于电影的对象数组，你可以遍历这些电影并打印它们的标题。

让我们看看一个实际的`for…of`循环。创建一个名为*forOf.html*的新文件，内容参考列表 4-7。

```
<html><body><script>
let colors = ["Red", "Green", "Blue"];

for (let color of colors) {
  console.log(`${color} is a color.`);
}
</script></body></html> 
```

列表 4-7：使用 for…of 循环遍历数组

这段代码会为数组`colors`中的每个颜色记录一条句子，然后停止。我们首先创建了一个包含字符串“Red”、“Green”和“Blue”的数组。然后我们使用语句`for (let color of colors)`将循环变量`color`依次设置为`colors`中的每个元素。第一次执行时，`color`会被设置为“Red”。第二次时，`color`会被设置为“Green”。最后，第三次时，`color`会被设置为“Blue”。当数组中的项用完时，循环结束。该脚本应该输出如下内容：

```
Red is a color.
Green is a color.
Blue is a color. 
```

也可以使用常规的`for`循环来遍历数组中的项，详见列表 4-8。

```
for (let index = 0; index < colors.length; index++) {
  console.log(`${colors[index]} is a color.`);
} 
```

清单 4-8：使用 for 循环代替 for…of 循环遍历数组

在这里，循环变量 index 代表数组中每个项的索引。我们的循环设置代码将 index 初始化为 0，并逐渐增加，直到它不再小于 colors 数组的长度（记住，长度为 *N* 的数组中，最高的索引是 *N* - 1）。在循环体内，我们使用 colors[index] 访问当前的颜色。

很长一段时间，这种 for 循环风格是 JavaScript 中遍历数组的唯一方式。能够识别它是值得的，因为你可能会在许多旧代码中看到它。如今，for…of 风格更为常见。然而，旧的 for 循环技巧的一个优点是它可以让你访问数组的索引。这很有用，因为有时候知道你当前正在处理数组中的哪个元素是很重要的。例如，你可能希望对偶数和奇数元素做不同的处理，或者你可能只是想在输出元素值的同时打印出索引，以便生成一个编号列表。你也可以使用 for…of 循环，通过对数组使用 entries 方法来实现这一点。要查看它是如何工作的，创建一个新的 *forOfEntries.html* 文件，并输入 清单 4-9 的内容。

```
<html><body><script>
let colors = ["Red", "Green", "Blue"];
for (let [index, item] of colors.entries()) {
  console.log(`${index}: ${item} is a color.`);
}
</script></body></html> 
```

清单 4-9：使用 for…of 循环与 entries 方法访问数组中的索引

在上一章中，你看到将 Object.entries 方法应用于对象时，会得到一个包含数组的数组，其中每个内层数组包含对象的一个键及其关联的值。在这里，对 colors 数组调用 entries 方法做了类似的操作，得到数组 [[0, "Red"], [1, "Green"], [2, "Blue"]]。语法 let [index, item] 被称为 *解构赋值*。它将 colors.entries 中的每个两元素数组（例如 [0, "Red"]）拆分成两个独立的变量，index 用于索引号，item 用于对应的值。通过这种方式，我们可以将索引包含进日志消息中，生成如下输出：

```
0: Red is a color.
1: Green is a color.
2: Blue is a color. 
```

请注意，解构赋值也可以在常规赋值语句中使用，在 for…of 循环之外，将数组拆分成独立的变量。例如，你可以像这样将表示 RGB 颜色值的三个数字数组转换为单独的 r、g 和 b 变量：

```
let rgbcolor = [125, 100, 0];
let [r, g, b] = rgbcolor; 
```

由于解构赋值，r 现在的值为 125，g 的值为 100，b 的值为 0。我们在本书中不会频繁使用这种语法，但能够识别它是有帮助的。

#### for…in 循环

for…in 循环遍历对象中的键。它的工作方式类似于 for…of 循环，依次取出每个键，并在没有更多键时停止。不同之处在于，for…in 循环适用于对象，而非数组，遍历的是键，而非值。保存 清单 4-10 的内容为 *forIn.html* 来进行尝试。

```
<html><body><script>
let me = {
  "first name": "Nick",
  "last name": "Morgan",
  "age": 39
};

for (let key in me) {
  console.log(`My ${key} is ${me[key]}.`);
}
</script></body></html> 
```

清单 4-10：使用 for…in 循环遍历对象中的键

在这里，我们创建了一个包含三个键值对的 me 对象（可以随意填写你自己的名字和年龄）。然后我们使用 for…in 循环遍历这些键。类似于 for…of 循环语法，写 `for (let key in me)` 会创建一个循环变量 key，并将其设置为 me 对象中的每个键，逐个进行。第一次循环时，key 被设置为 "first name"（名字），第二次时被设置为 "last name"（姓氏），依此类推。在循环体内，我们使用表示法 me[key] 来访问与当前键相关联的值，并将其与键一起嵌入到消息中。输出应类似于以下内容：

```
My first name is Nick.
My last name is Morgan.
My age is 39. 
```

我们本可以使用 Object.entries(me) 来获取一个包含键值对的数组，然后用 for…of 循环遍历这些键值对。像往常一样，这种选择主要是个人偏好。

### 总结

本章展示了如何使用条件语句和循环为代码添加逻辑和结构。这些控制结构让你能够决定代码何时以及多少次执行。像 if 和 if…else 这样的条件语句根据某个条件是否成立来决定是否执行代码。某些循环，如 while 和 for，会重复执行相同的代码，直到满足某个条件。而像 for…of 和 for…in 这样的循环则是用来遍历数组或对象的元素。
