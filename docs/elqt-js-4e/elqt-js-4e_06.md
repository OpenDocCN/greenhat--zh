# `第六章`：`高阶函数`

一个大型程序是一个高成本的程序，这不仅仅是因为构建它所需的时间。大小几乎总是意味着复杂性，而复杂性会使程序员感到困惑。困惑的程序员又会在程序中引入错误（`bug`）。因此，大型程序提供了大量隐藏这些错误的空间，使它们难以被发现。

让我们简要回顾一下引言中的最后两个示例程序。第一个是自包含的，长六行。

```js
let total = 0, count = 1;
while (count <= 10) {
  total += count;
  count += 1;
}
console.log(total);
```

第二个依赖于两个外部函数，且只有一行长。

```js
console.log(sum(range(1, 10)));
```

哪一个更可能包含错误？

如果我们计算求和和范围的定义的大小，第二个程序也很大——甚至比第一个还要大。但我仍然认为，它更可能是正确的。

这是因为解决方案使用了与待解决问题相对应的词汇。对一系列数字求和并不是关于循环和计数器，而是关于范围和总和。

这个词汇的定义（函数`sum`和`range`）仍然会涉及循环、计数器和其他附带的细节。但因为它们表达的概念比整个程序更简单，所以更容易正确实现。

### `抽象`

在编程的上下文中，这类词汇通常被称为`抽象`。`抽象`使我们能够在更高（或更抽象）的层次上讨论问题，而不被无趣的细节分散注意力。

作为类比，比较这两种豌豆汤的食谱。第一个是这样的：

每人放`1`杯干豌豆到容器中。加水直到豌豆完全浸没。将豌豆在水中浸泡至少`12`小时。将豌豆从水中取出，放入烹饪锅中。每人加`4`杯水。盖上锅盖，让豌豆慢炖`2`小时。每人取半个洋葱。用刀切成小块，加入豌豆。每人取一根芹菜。用刀切成小块，加入豌豆。每人取一根胡萝卜。切成小块。用刀！加入豌豆。再煮`10`分钟。

而这就是第二个食谱：

每人：`1`杯干豌豆、`4`杯水、半个切碎的洋葱、一根芹菜和一根胡萝卜。

将豌豆浸泡`12`小时。慢炖`2`小时。切碎并添加蔬菜。再煮`10`分钟。

第二个更简短，更容易理解。但你确实需要理解一些与烹饪相关的词汇，比如`浸泡`、`慢炖`、`切碎`，还有，我想，`蔬菜`。

在编程时，我们不能依赖于所有需要的词汇都在字典中等待我们。因此，我们可能会陷入第一个食谱的模式——逐步确定计算机必须执行的精确步骤，而忽视了它们所表达的更高层次的概念。

在编程中，注意到自己在过低的`抽象`层次工作是一项有用的技能。

### `抽象重复`

到目前为止，我们看到的普通函数是构建`抽象`的好方法。但有时它们并不够。

程序执行某个操作若干次是很常见的。你可以为此编写一个`for`循环，如下所示：

```js
for (let i = 0; i < 10; i++) {
  console.log(i);
}
```

我们能否将“做某事`N`次”抽象为一个函数？实际上，编写一个调用`console.log` `N`次的函数是很简单的。

```js
function repeatLog(n) {
  for (let i = 0; i < n; i++) {
    console.log(i);
  }
}
```

但如果我们想做些其他事情，而不是记录数字呢？由于“做某事”可以表示为一个函数，而函数只是值，我们可以将我们的操作作为函数值传递。

```js
function repeat(n, action) {
  for (let i = 0; i < n; i++) {
    action(i);
  }
}

repeat(3, console.log);
// → 0
// → 1
// → 2
```

我们不必传递一个预定义的函数来重复。通常，现场创建一个函数值更为简单。

```js
let labels = [];
repeat(5, i => {
  labels.push(`Unit ${i + 1}`);
});
console.log(labels);
// → ["Unit 1", "Unit 2", "Unit 3", "Unit 4", "Unit 5"]
```

这在结构上有点像`for`循环——它首先描述循环的类型，然后提供一个主体。然而，主体现在写成了一个函数值，它被包裹在调用`repeat`的括号中。这就是为什么它必须用结束大括号`和`结束括号来关闭。在这种情况下，主体是一个单独的小表达式，你也可以省略大括号，将循环写在一行上。

### `高阶函数`

操作其他函数的函数，或者通过将它们作为参数传递，或通过返回它们，被称为`高阶函数`。既然我们已经看到函数是常规值，那么这样的函数存在并没有什么特别之处。这个术语源于数学，在数学中，函数与其他值之间的区别更为重要。

高阶函数使我们能够对`动作`进行`抽象`，而不仅仅是值。它们有几种形式。例如，我们可以有创建新函数的函数。

```js
function greaterThan(n) {
  return m => m > n;
}
let greaterThan10 = greaterThan(10);
console.log(greaterThan10(11));
// → true
```

我们还可以有改变其他函数的函数。

```js
function noisy(f) {
  return (...args) => {
    console.log("calling with", args);
    let result = f(...args);
    console.log("called with", args, ", returned", result);
    return result;
  };
}
noisy(Math.min)(3, 2, 1);
// → calling with [3, 2, 1]
// → called with [3, 2, 1] , returned 1
```

我们甚至可以编写提供新类型控制流的函数。

```js
function unless(test, then) {
  if (!test) then();
}

repeat(3, n => {
  unless(n % 2 == 1, () => {
    console.log(n, "is even");
  });
});
// → 0 is even
// → 2 is even
```

有一个内置的数组方法`forEach`，它提供类似于`for/of`循环的高阶函数。

```js
["A", "B"].forEach(l => console.log(l));
// → A
// → B
```

### `脚本数据集`

`高阶函数`在数据处理方面表现出色。要处理数据，我们需要一些实际的示例数据。本章将使用一个关于脚本的数据集——如拉丁字母、西里尔字母或阿拉伯字母等书写系统。

记得`Unicode`吗？它是将数字分配给书写语言中每个字符的系统，来自`第一章`？这些字符大多与特定的脚本相关。标准包含`140`种不同的脚本，其中`81`种仍在使用中，`59`种是历史脚本。

尽管我只能流利地阅读拉丁字符，但我很欣赏人们用至少`80`种其他书写系统书写文本的事实，其中许多我甚至无法识别。例如，这里有一段泰米尔文的手写样本：

![图像](img/f0085-01.jpg)

示例数据集包含有关`Unicode`中定义的`140`种脚本的一些信息。它在本章节的编码沙箱中可用（*[`eloquentjavascript.net/code#5`](https://eloquentjavascript.net/code#5)），作为`SCRIPTS`绑定。该绑定包含一个对象数组，每个对象描述一种脚本。

```js
{
  name: "Coptic",
  ranges: [[994, 1008], [11392, 11508], [11513, 11520]],
  direction: "ltr",
  year: -200,
  living: false,
  link: "https://en.wikipedia.org/wiki/Coptic_alphabet"
}
```

这样的对象告诉我们脚本的名称、分配给它的Unicode范围、书写方向、（大致的）起源时间、是否仍在使用以及更多信息的链接。书写方向可以是“ltr”表示从左到右，“rtl”表示从右到左（阿拉伯语和希伯来语文本的书写方式），或者“ttb”表示从上到下（如蒙古文书写）。

`ranges`属性包含一个Unicode字符范围的数组，每个范围都是一个包含下限和上限的两个元素数组。这些范围内的任何字符代码都被分配给该脚本。下限是包含的（代码`994`是一个科普特字符），而上限是不包含的（代码`1008`不是）。

### 过滤数组

如果我们想找到数据集中仍在使用的脚本，下面的函数可能会有所帮助。它过滤掉数组中未通过测试的元素。

```js
function filter(array, test) {
  let passed = [];
  for (let element of array) {
    if (test(element)) {
      passed.push(element);
    }
  }
  return passed;
}

console.log(filter(SCRIPTS, script => script.living));
// → [{name: "Adlam", ...}, ...]
```

该函数使用名为`test`的参数，一个函数值，以填补计算中的“空缺”——决定收集哪些元素的过程。

注意`filter`函数如何不是从现有数组中删除元素，而是构建一个新数组，仅包含通过测试的元素。这个函数是`纯粹`的。它不会修改传入的数组。

像`forEach`一样，`filter`是一个标准数组方法。这个例子仅定义了该函数，以展示它在内部的工作原理。从现在开始，我们将这样使用它：

```js
console.log(SCRIPTS.filter(s => s.direction == "ttb"));
// → [{name: "Mongolian", ...}, ...]
```

### 使用`map`进行转换

假设我们有一个表示脚本的对象数组，这些对象是通过某种方式过滤`SCRIPTS`数组而生成的。我们希望得到一个名称的数组，这样更容易检查。

`map`方法通过对数组的所有元素应用函数并从返回值构建一个新数组来转换数组。新数组将具有与输入数组相同的长度，但其内容将通过函数`映射`到新形式。

```js
function map(array, transform) {
  let mapped = [];
  for (let element of array) {
    mapped.push(transform(element));
 }
  return mapped;
}

let rtlScripts = SCRIPTS.filter(s => s.direction == "rtl");
console.log(map(rtlScripts, s => s.name));
// → ["Adlam", "Arabic", "Imperial Aramaic", ...]
```

像`forEach`和`filter`一样，`map`也是一个标准数组方法。

### 使用`reduce`进行汇总

对数组进行的另一个常见操作是从中计算出单个值。我们反复提到的例子，求和一组数字，就是这个操作的一个实例。另一个例子是找出字符最多的脚本。

表示此模式的高阶操作称为`reduce`（有时也称为`fold`）。它通过重复从数组中提取单个元素并将其与当前值结合来构建一个值。在求和时，你将从零开始，并对每个元素将其加到总和中。

`reduce`的参数除了数组外，还有一个组合函数和一个起始值。这个函数比过滤和映射稍微复杂一些，所以请仔细看看：

```js
function reduce(array, combine, start) {
  let current = start;
  for (let element of array) {
    current = combine(current, element);
  }
  return current;
}

console.log(reduce([1, 2, 3, 4], (a, b) => a + b, 0));
// → 10
```

标准数组方法`reduce`，自然对应这个函数，具有额外的便利性。如果你的数组至少包含一个元素，你可以省略起始参数。该方法会将数组的第一个元素作为起始值，并从第二个元素开始减少。

```js
console.log([1, 2, 3, 4].reduce((a, b) => a + b));
// → 10
```

要使用`reduce`（两次）来找到字符最多的脚本，我们可以写如下内容：

```js
function characterCount(script) {
  return script.ranges.reduce((count, [from, to]) => {
 return count + (to - from);
  }, 0);
}

console.log(SCRIPTS.reduce((a, b) => {
  return characterCount(a) < characterCount(b) ? b : a;
}));
// → {name: "Han", ...}
```

`characterCount`函数通过对分配给脚本的范围进行求和来减少其大小。注意在`reducer`函数的参数列表中使用了解构。第二次调用`reduce`则利用这个功能通过不断比较两个脚本并返回较大的一个来找到最大的脚本。

汉字在 Unicode 标准中分配了超过 89,000 个字符，使其成为数据集中最大的书写系统。汉字有时用于中文、日文和韩文。这些语言共享许多字符，尽管它们通常以不同的方式书写。美国的 Unicode 联盟决定将它们视为一个书写系统，以节省字符编码。这被称为`汉字统一`，并且仍然让一些人感到非常愤怒。

### 可组合性

想想如果没有高阶函数，我们将如何编写之前的示例（寻找最大的脚本）。代码并没有变得差很多。

```js
let biggest = null;
for (let script of SCRIPTS) {
  if (biggest == null ||
      characterCount(biggest) < characterCount(script)) {
    biggest = script;
  }
}
console.log(biggest);
// → {name: "Han", ...}
```

还有几个绑定，程序多了四行，但仍然非常可读。

这些函数提供的抽象在你需要`组合`操作时表现得尤为出色。例如，让我们编写代码来找到数据集中存活和已亡脚本的平均来源年份。

```js
function average(array) {
  return array.reduce((a, b) => a + b) / array.length;
}

console.log(Math.round(average(
  SCRIPTS.filter(s => s.living).map(s => s.year))));
// → 1165
console.log(Math.round(average(
  SCRIPTS.filter(s => !s.living).map(s => s.year))));
// → 204
```

如你所见，Unicode中的已亡脚本平均来说比存活的脚本要古老。这并不是一个特别有意义或令人惊讶的统计数据。但我希望你会同意，用来计算它的代码并不难以阅读。你可以把它看作一个管道：我们从所有脚本开始，过滤出存活（或已亡）的脚本，从中取出年份，取平均值，然后四舍五入结果。

你当然也可以将这段计算写成一个大循环。

```js
let total = 0, count = 0;
for (let script of SCRIPTS) {
  if (script.living) {
    total += script.year;
    count += 1;
  }
}
console.log(Math.round(total / count));
// → 1165
```

然而，很难看出计算的内容及其方式。而且，由于中间结果并未作为一致的值表示，要将像平均数这样的内容提取到单独的函数中将会更加复杂。

在计算机实际执行的方面，这两种方法也相当不同。第一种方法在运行过滤和映射时会建立新的数组，而第二种只计算一些数字，工作量更少。通常你可以选择可读性更高的方法，但如果你在处理巨大的数组并且多次执行，那么较少抽象的风格可能值得额外的速度。

### 字符串和字符编码

这个数据集的一个有趣用法是确定一段文本使用了什么脚本。让我们看看一个实现这一点的程序。

记住，每种脚本都有一个与之相关的字符代码范围数组。给定一个字符代码，我们可以使用这样的函数来查找对应的脚本（如果有的话）：

```js
function characterScript(code) {
  for (let script of SCRIPTS) {
    if (script.ranges.some(([from, to]) => {
      return code >= from && code < to;
    })) {
      return script;
    }
  }
  return null;
}

console.log(characterScript(121));
// → {name: "Latin", ...}
```

`some`方法是另一个高阶函数。它接受一个测试函数，并告诉你该函数是否对数组中的任何元素返回`true`。

但我们如何获取字符串中的字符代码呢？

在第一章中，我提到 JavaScript 字符串被编码为一系列 16 位数字。这些被称为`代码单元`。最初，Unicode 字符代码应该适合这样的单元（这给你提供了稍微超过 65,000 个字符）。当变得清楚这不够时，许多人对每个字符需要使用更多内存感到抵触。为了应对这些担忧，`UTF-16`这种格式（JavaScript 字符串也使用该格式）被发明。它使用单个 16 位代码单元描述大多数常见字符，但对于其他字符则使用一对这样的单元。

今天，`UTF-16`通常被认为是一个糟糕的主意。它似乎几乎是故意设计来引发错误的。编写假装代码单元和字符是同一事物的程序非常简单。如果你的语言不使用两个单元的字符，这看起来似乎工作得很好。但一旦有人尝试用这样一个程序处理一些不太常见的汉字，就会出现问题。幸运的是，随着表情符号的出现，大家都开始使用两个单元的字符，这样处理这些问题的负担更公平地分配了。

不幸的是，对 JavaScript 字符串的明显操作，如通过`length`属性获取它们的长度和使用方括号访问它们的内容，只处理代码单元。

```js
// Two emoji characters, horse and shoe
let horseShoe = "";
console.log(horseShoe.length);
// → 4
console.log(horseShoe[0]);
// → (Invalid half-character)
console.log(horseShoe.charCodeAt(0));
// → 55357 (Code of the half-character)
console.log(horseShoe.codePointAt(0));
// → 128052 (Actual code for horse emoji)
```

JavaScript的`charCodeAt`方法返回一个代码单元，而不是完整的字符代码。稍后添加的`codePointAt`方法确实给出了完整的 Unicode 字符，因此我们可以用它从字符串中获取字符。但传递给`codePointAt`的参数仍然是代码单元序列中的一个索引。要遍历字符串中的所有字符，我们仍然需要处理一个字符占用一个或两个代码单元的问题。

在第四章中，我提到可以在字符串上使用`for/of`循环。与`codePointAt`类似，这种循环在人们对`UTF-16`存在的问题有深刻认识时被引入。当你用它遍历字符串时，它会给你真实的字符，而不是代码单元。

```js
let roseDragon = "";
for (let char of roseDragon) {
  console.log(char);
}
// → 
// → 
```

如果你有一个字符（它将是一个由一个或两个代码单元组成的字符串），你可以使用`codePointAt(0)`来获取它的代码。

### 识别文本

我们有一个`characterScript`函数和一种正确遍历字符的方法。下一步是计算每种脚本所包含的字符数量。以下计数抽象将在这里很有用：

```js
function countBy(items, groupName) {
  let counts = [];
  for (let item of items) {
    let name = groupName(item);
    let known = counts.find(c => c.name == name);
    if (!known) {
      counts.push({name, count: 1});
    } else {
      known.count++;
    }
  }
  return counts;
}

console.log(countBy([1, 2, 3, 4, 5], n => n > 2));
// → [{name: false, count: 2}, {name: true, count: 3}]
```

`countBy`函数期望一个集合（任何可以用`for/of`循环遍历的内容）和一个为给定元素计算组名的函数。它返回一个对象数组，每个对象命名一个组，并告诉你在该组中找到的元素数量。

它使用另一个数组方法`find`，该方法遍历数组中的元素并返回第一个函数返回`true`的元素。如果没有找到这样的元素，它将返回`undefined`。

使用`countBy`，我们可以编写一个函数来告诉我们在一段文本中使用了哪些脚本。

```js
function textScripts(text) {
  let scripts = countBy(text, char => {
    let script = characterScript(char.codePointAt(0));
 return script ? script.name : "none";
  }).filter(({name}) => name != "none");

  let total = scripts.reduce((n, {count}) => n + count, 0);
  if (total == 0) return "No scripts found";

  return scripts.map(({name, count}) => {
    return `${Math.round(count * 100 / total)}% ${name}`;
  }).join(", ");
}

console.log(textScripts('英国的狗说"woof", 俄罗斯的狗说"тяв"'));
// → 61% Han, 22% Latin, 17% Cyrillic
```

该函数首先通过名称计数字符，使用`characterScript`为它们分配名称，并在字符不属于任何脚本时回退到字符串`"none"`。`filter`调用从结果数组中删除`"none"`的条目，因为我们对这些字符不感兴趣。

为了能够计算百分比，我们首先需要计算属于某个脚本的字符总数，可以通过`reduce`来计算。如果没有找到这样的字符，函数将返回一个特定字符串。否则，它会通过`map`将计数条目转换为可读字符串，然后用`join`将它们组合起来。

### 总结

能够将函数值传递给其他函数是 JavaScript一个非常有用的特性。它允许我们编写带有“缺口”的计算模型函数。调用这些函数的代码可以通过提供函数值来填补这些缺口。

数组提供了许多有用的高阶方法。你可以使用`forEach`遍历数组中的元素。`filter`方法返回一个新数组，只包含通过谓词函数的元素。你可以通过使用`map`将每个元素放入一个函数来转换数组。你可以使用`reduce`将数组中的所有元素组合成一个单一值。`some`方法测试是否有任何元素与给定的谓词函数匹配，而`find`则找到第一个匹配谓词的元素。

### 练习

#### `扁平化`

使用`reduce`方法与`concat`方法结合，将一个数组的数组“扁平化”为一个包含所有原始数组元素的单一数组。

#### `你自己的循环`

编写一个高阶函数`loop`，提供类似于`for`循环语句的功能。它应该接受一个值、一个测试函数、一个更新函数和一个主体函数。每次迭代时，它应该首先在当前循环值上运行测试函数，如果返回`false`，则停止。然后调用主体函数，传递当前值，最后调用更新函数以生成新值并重新开始。

在定义函数时，你可以使用常规循环来实际执行循环。

#### `一切`

数组有一个`every`方法，与`some`方法类似。当给定的函数对数组中的`每个`元素返回`true`时，该方法返回`true`。从某种意义上说，`some`是作用于数组的`||`运算符的一个版本，而`every`则像是`&&`运算符。

将每个实现作为一个函数，该函数接受一个数组和一个谓词函数作为参数。写两个版本，一个使用循环，另一个使用`some`方法。

#### `主导书写方向`

编写一个函数，用于计算文本字符串中的主导书写方向。记住，每个脚本对象都有一个`direction`属性，可能是“`ltr`”（从左到右）、“`rtl`”（从右到左）或“`ttb`”（从上到下）。

主导方向是指大多数具有相关脚本的字符的方向。前面章节中定义的`characterScript`和`countBy`函数在这里可能会很有用。

`抽象数据类型通过编写一种特殊类型的程序来实现，该程序定义了可以在其上执行的操作类型。`

—芭芭拉·利斯科夫，`使用抽象数据类型编程`

![图像](img/f0094-01.jpg)
