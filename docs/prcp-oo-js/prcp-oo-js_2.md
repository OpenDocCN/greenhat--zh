## 第二章 函数

如 第一章 中所讨论的，函数实际上是 JavaScript 中的对象。函数的定义特征——将其与其他对象区分开来——是存在一个名为 `[[Call]]` 的 *内部属性*。内部属性无法通过代码访问，而是定义了代码执行时的行为。ECMAScript 为 JavaScript 中的对象定义了多个内部属性，这些内部属性通过双中括号表示。

`[[Call]]` 属性是函数特有的，表示该对象可以被执行。因为只有函数具有这个属性，所以 ECMAScript 定义了 `typeof` 运算符，对于任何具有 `[[Call]]` 属性的对象，`typeof` 返回 `"function"`。这曾导致一些混淆，因为某些浏览器也为正则表达式添加了 `[[Call]]` 属性，因此它们被错误地识别为函数。现在所有浏览器的行为一致，所以 `typeof` 不再将正则表达式识别为函数。

本章讨论了在 JavaScript 中定义和执行函数的各种方式。由于函数是对象，它们的行为与其他语言中的函数不同，这种行为对于深入理解 JavaScript 至关重要。

## 声明与表达式

实际上有两种字面量形式的函数。第一种是 *函数声明*，它以 `function` 关键字开头，并紧接着是函数的名称。函数的内容被包含在大括号中，如下面的声明所示：

```
`function` add(num1, num2) {
    `return` num1 `+` num2;
}
```

第二种形式是 *函数表达式*，它不需要在 `function` 后面指定名称。这些函数被视为匿名函数，因为函数对象本身没有名称。相反，函数表达式通常通过变量或属性进行引用，如以下表达式所示：

```
`var` add `=` `function`(num1, num2) {
    `return` num1 `+` num2;
};
```

这段代码实际上将一个函数值赋给了变量 `add`。函数表达式与函数声明几乎相同，唯一不同的是缺少名称和结尾的分号。赋值表达式通常以分号结束，就像你在赋值任何其他值时一样。

虽然这两种形式非常相似，但它们在一个非常重要的方面有所不同。函数声明会在代码执行时被 *提升* 到上下文的顶部（无论是函数内部还是全局作用域）。这意味着你实际上可以在代码中使用函数之后再定义它，而不会产生错误。例如：

```
`var` result `=` add(5, 5);

`function` add(num1, num2) {
    `return` num1 `+` num2;
}
```

这段代码看起来像是会导致错误，但实际上它是可以正常运行的。这是因为 JavaScript 引擎将函数声明提升到顶部，并实际上按照如下方式执行代码：

```
`// how the JavaScript engine interprets the code`
`function` add(num1, num2) {
    `return` num1 `+` num2;
}

`var` result `=` add(5, 5);
```

函数提升只会发生在函数声明中，因为函数名在使用之前是已知的。另一方面，函数表达式不能提升，因为这些函数只能通过变量来引用。因此，以下代码会导致错误：

```
`// error!`
`var` result `=` add(5, 5);

`var` add `=` `function`(num1, num2) {
    `return` num1 `+` num2;
};
```

只要你始终在使用函数之前定义它们，就可以使用函数声明或函数表达式。

## 函数作为值

由于 JavaScript 具有一等函数，你可以像使用其他对象一样使用它们。你可以将它们赋值给变量，添加到对象中，将它们作为参数传递给其他函数，甚至从函数中返回。基本上，你可以在任何需要引用值的地方使用函数。这使得 JavaScript 的函数功能极其强大。考虑以下示例：

```
❶ `function` sayHi() {
      console.log(`"Hi!"`);
  }

  sayHi();        `// outputs "Hi!"`

❷ `var` sayHi2 `=` sayHi;

  sayHi2();       `// outputs "Hi!"`
```

在这段代码中，`sayHi` ❶ 是一个函数声明。接着，创建了一个名为 `sayHi2` 的变量，并将 `sayHi` 的值赋给它 ❷。此时，`sayHi` 和 `sayHi2` 都指向同一个函数，这意味着两者都可以执行，且结果相同。为了理解为什么会这样，请看一下重写后的代码，使用了 `Function` 构造器：

```
`var` sayHi `=` `new` `Function`(`"console.log(\"Hi!\");"`);

sayHi();        `// outputs "Hi!"`

`var` sayHi2 `=` sayHi;

sayHi2();       `// outputs "Hi!"`
```

`Function` 构造器使得 `sayHi` 可以像其他对象一样被传递得更加明确。当你记住函数也是对象时，许多行为就开始变得有意义。

例如，你可以将一个函数作为参数传递给另一个函数。JavaScript 数组上的 `sort()` 方法接受一个可选的比较函数作为参数。每当数组中的两个值需要进行比较时，都会调用该比较函数。如果第一个值小于第二个值，比较函数必须返回一个负数。如果第一个值大于第二个值，函数必须返回一个正数。如果两个值相等，函数应该返回零。

默认情况下，`sort()` 会将数组中的每个项转换为字符串，然后进行比较。这意味着在没有指定比较函数的情况下，你无法准确地对数字数组进行排序。例如，你需要提供一个比较函数来准确排序数字数组，例如：

```
  `var` numbers `=` [ 1, 5, 8, 4, 7, 10, 2, 6 ];
❶ numbers.sort(`function`(first, second) {
      `return` first - second;
  });

  console.log(numbers);       `// "[1, 2, 4, 5, 6, 7, 8, 10]"`

❷ numbers.sort();
  console.log(numbers);       `// "[1, 10, 2, 4, 5, 6, 7, 8]"`
```

在这个例子中，传入 `sort()` 的比较函数 ❶ 实际上是一个函数表达式。注意，这个函数没有名称，它只是作为引用传递给另一个函数（因此它是一个*匿名函数*）。通过减去两个值，可以从比较函数中返回正确的结果。

将其与第二次调用 `sort()` ❷ 进行比较，后者没有使用比较函数。数组的顺序与预期不同，因为 1 后面跟着 10。这是因为默认的比较会在比较之前将所有值转换为字符串。

## 参数

JavaScript 函数的另一个独特之处在于，你可以向任何函数传递任意数量的参数，而不会导致错误。这是因为函数的参数实际上是以类似数组的结构 `arguments` 存储的。就像常规的 JavaScript 数组一样，`arguments` 可以增长以包含任意数量的值。可以通过数字索引引用这些值，并且有一个 `length` 属性来确定有多少个值存在。

`arguments` 对象在任何函数内部都是自动可用的。这意味着函数中的命名参数主要是为了方便，而并不会真正限制函数可以接受的参数数量。

### 注意

*`arguments` 对象不是 Array 的实例，因此没有与数组相同的方法；Array.isArray(arguments) 总是返回 false。*

另一方面，JavaScript 并不会忽略函数的命名参数。函数期望的参数个数存储在函数的 `length` 属性中。记住，函数实际上只是一个对象，因此它可以拥有属性。`length` 属性表示函数的*参数个数*，即它期望的参数数量。在 JavaScript 中，了解函数的参数个数很重要，因为如果传入的参数过多或过少，函数并不会抛出错误。

下面是一个使用 `arguments` 和函数参数个数的简单示例；请注意，传递给函数的参数个数不会影响报告的参数个数：

```
`function` reflect(value) {
    `return` value;
}

console.log(reflect(`"Hi!"`));        `// "Hi!"`
console.log(reflect(`"Hi!"`, `25`));    `// "Hi!"`
console.log(reflect.length);        `// 1`

reflect `=` `function`() {
    `return` arguments[0];
};

console.log(reflect(`"Hi!"`));        `// "Hi!"`
console.log(reflect(`"Hi!"`, `25`));    `// "Hi!"`
console.log(reflect.length);        `// 0`
```

这个示例首先使用单个命名参数定义了 `reflect()` 函数，但当传入第二个参数时并不会报错。此外，`length` 属性为 `1`，因为有一个命名参数。然后，`reflect()` 函数被重新定义为没有命名参数；它返回 `arguments[0]`，即传入的第一个参数。这个新版本的函数与之前的版本完全相同，但它的 `length` 为 `0`。

`reflect()` 的第一次实现更容易理解，因为它使用了命名参数（就像在其他语言中一样）。使用 `arguments` 对象的版本可能会让人困惑，因为没有命名参数，必须阅读函数体才能确定是否使用了参数。这就是为什么许多开发人员除非必要，否则倾向于避免使用 `arguments`。

然而，有时使用 `arguments` 实际上比命名参数更有效。例如，假设你想创建一个函数，接受任意数量的参数并返回它们的和。你无法使用命名参数，因为你不知道需要多少个参数，因此在这种情况下，使用 `arguments` 是最佳选择。

```
`function` sum() {

    `var` result `=` `0`,
        i `=` `0`,
        len `=` arguments.length;

    `while` (i `<` len) {
        result `+=` arguments[i];
        i`++`;
    }

    `return` result;
}

console.log(sum(`1`, `2`));         `// 3`
console.log(sum(`3`, `4`, `5`, `6`));   `// 18`
console.log(sum(`50`));           `// 50`
console.log(sum());             `// 0`
```

`sum()` 函数接受任意数量的参数，并通过使用 `while` 循环遍历 `arguments` 中的值将它们加在一起。这与你必须将一组数字相加是完全相同的。即使没有传入参数，该函数也能正常工作，因为 `result` 初始化为 `0`。

## 重载

大多数面向对象的语言支持 *函数重载*，即单个函数可以拥有多个 *签名*。函数签名由函数名称加上函数期望的参数数量和类型组成。因此，一个函数可以有一个接受单个字符串参数的签名，也可以有一个接受两个数字参数的签名。语言会根据传入的参数来确定调用哪个版本的函数。

如前所述，JavaScript 函数可以接受任意数量的参数，并且函数接受的参数类型根本没有指定。这意味着 JavaScript 函数实际上没有签名。没有函数签名也意味着没有函数重载。看看当你尝试声明两个同名函数时会发生什么：

```
`function` sayMessage(message) {
    console.log(message);
}

`function` sayMessage() {
    console.log("D`efault` `message``"``);`
}

sayMessage(`"Hello!"`);       `// outputs "Default message"`
```

如果这是另一种语言，`sayMessage("Hello!")` 的输出可能会是 `"Hello!"`。然而，在 JavaScript 中，当你定义多个同名函数时，最后一个出现在代码中的函数会“获胜”。之前的函数声明会完全被删除，最后的那个函数会被使用。再次强调，使用对象来理解这种情况会很有帮助：

```
`var` sayMessage `=` `new` `Function`(`"message"`, `"console.log(message);"`);

sayMessage `=` new Function(`"console.log(\"Default message\");"`);

sayMessage(`"Hello!"`);       `// outputs "Default message"`
```

这样看代码可以清楚地说明为什么之前的代码没有工作。一个函数对象被连续两次赋值给 `sayMessage`，所以可以理解第一个函数对象会被丢失。

JavaScript 中函数没有签名并不意味着你不能模拟函数重载。你可以使用 `arguments` 对象来获取传入的参数数量，并利用这些信息来决定要做什么。例如：

```
`function` sayMessage(message) {

    `if` (arguments.length `===` `0`) {
        message `=` `"Default message"`;
    }

    console.log(message);
}

sayMessage(`"Hello!"`);       `// outputs "Hello!"`
```

在这个例子中，`sayMessage()` 函数根据传入的参数数量行为不同。如果没有传入参数（`arguments.length === 0`），则使用默认消息。否则，使用第一个参数作为消息。这比其他语言中的函数重载稍微复杂一些，但最终结果是相同的。如果你真的想检查不同的数据类型，可以使用 `typeof` 和 `instanceof`。

### 注意

*在实践中，检查命名参数是否为 undefined 比依赖 `arguments.length` 更常见。*

## 对象方法

如在第一章中提到的，你可以随时向对象添加和删除属性。当属性值实际上是一个函数时，该属性就被视为方法。你可以像添加属性一样向对象添加方法。例如，在以下代码中，`person` 变量被赋值为一个包含 `name` 属性和一个名为 `sayName` 的方法的对象字面量。

```
`var` person `=` {
    name: `"Nicholas"`,
    sayName: `function`() {
        console.log(person.name);
    }
};

person.sayName();       `// outputs "Nicholas"`
```

注意，数据属性和方法的语法是完全相同的——一个标识符后跟冒号和值。在 `sayName` 的情况下，值恰好是一个函数。你可以像 `person.sayName("Nicholas")` 这样直接从对象调用该方法。

### `this` 对象

你可能在前面的示例中注意到了一些奇怪的地方。`sayName()` 方法直接引用了 `person.name`，这导致了方法和对象之间的紧密耦合。这样存在一些问题。首先，如果你更改了变量名，你还需要记得在方法中也更改对该名称的引用。其次，这种紧密耦合使得同一个函数难以在不同对象之间复用。幸运的是，JavaScript 提供了一种解决这个问题的方法。

JavaScript 中的每个作用域都有一个 `this` 对象，表示调用该函数的对象。在全局作用域中，`this` 代表全局对象（在网页浏览器中为 `window`）。当函数与对象绑定时，`this` 的值默认等于该对象。因此，你可以在方法中使用 `this` 来代替直接引用对象。例如，你可以将前一个示例中的代码重写为使用 `this`：

```
`var` person `=` {
    name: `"Nicholas"`,
    sayName: `function`() {
        console.log(`this`.name);
    }
};

person.sayName();      `// outputs "Nicholas"`
```

这段代码的运行方式与早期版本相同，但这次 `sayName()` 引用了 `this` 而不是 `person`。这意味着你可以轻松地更改变量名，甚至在不同的对象上重用该函数。

```
`function` sayNameForAll() {
    console.log(`this`.name);
}

`var` person1 `=` {
    name: `"Nicholas"`,
    sayName: sayNameForAll
};

`var` person2 `=` {
    name: `"Greg"`,
    sayName: sayNameForAll
};

`var` name `=` `"Michael"`;

person1.sayName();      `// outputs "Nicholas"`
person2.sayName();      `// outputs "Greg"`

sayNameForAll();        `// outputs "Michael"`
```

在这个示例中，首先定义了一个名为 `sayName` 的函数。然后，创建了两个对象字面量，并将 `sayName` 赋值为 `sayNameForAll` 函数。函数本身只是引用值，因此你可以将它们作为属性值分配给任意数量的对象。当在 `person1` 上调用 `sayName()` 时，它输出 `"Nicholas"`；而在 `person2` 上调用时，输出 `"Greg"`。这是因为当函数被调用时，`this` 被设置，因此 `this.name` 是准确的。

这个示例的最后一部分定义了一个名为 `name` 的全局变量。当直接调用 `sayNameForAll()` 时，它输出 `"Michael"`，因为全局变量被认为是全局对象的一个属性。

### 修改 `this`

使用和操作函数的`this`值是 JavaScript 中良好的面向对象编程的关键。函数可以在许多不同的上下文中使用，并且需要能够在每种情况下都能正常工作。尽管`this`通常会自动分配，你仍然可以改变它的值以实现不同的目标。有三种函数方法可以让你改变`this`的值。（记住，函数是对象，而对象可以有方法，所以函数也可以有方法。）

#### `call()`方法

操作`this`的第一个函数方法是`call()`，它以特定的`this`值和具体的参数执行函数。`call()`的第一个参数是执行函数时`this`应该等于的值。所有后续的参数是应该传入函数的参数。例如，假设你更新了`sayNameForAll()`来接受一个参数：

```
`function` sayNameForAll(label) {
    console.log(label `+` `":"` `+` `this`.name);
}

`var` person1 `=` {
    name: `"Nicholas"`
};

`var` person2 `=` {
    name: `"Greg"`
};

`var` name `=` `"Michael"`;

sayNameForAll.call(`this`, `"global"`);        `// outputs "global:Michael"`
sayNameForAll.call(person1, `"person1"`);    `// outputs "person1:Nicholas"`
sayNameForAll.call(person2, `"person2"`);    `// outputs "person2:Greg"`
```

在这个例子中，`sayNameForAll()`接受一个参数，这个参数作为输出值的标签。然后，这个函数被调用了三次。请注意，在函数名称后没有括号，因为它作为对象被访问，而不是作为代码来执行。第一次函数调用使用了全局的`this`，并传入参数`"global"`来输出`"global:Michael"`。同样的函数被再调用了两次，分别用于`person1`和`person2`。因为使用的是`call()`方法，所以你不需要直接将函数添加到每个对象上——你显式指定了`this`的值，而不是让 JavaScript 引擎自动处理。

#### `apply()`方法

你可以用来操作`this`的第二种函数方法是`apply()`。`apply()`方法的工作原理与`call()`完全相同，只是它只接受两个参数：`this`的值和要传递给函数的参数数组或类数组对象（这意味着你可以使用`arguments`对象作为第二个参数）。因此，与你使用`call()`时逐个命名每个参数不同，你可以轻松地将数组作为第二个参数传递给`apply()`。否则，`call()`和`apply()`的行为是完全相同的。这个例子展示了`apply()`方法的使用：

```
`function` sayNameForAll(label) {
    console.log(label `+` `":"` `+` `this`.name);
}

`var` person1 `=` {
    name: `"Nicholas"`
};

`var` person2 `=` {
    name: `"Greg"`
};

`var` name `=` `"Michael"`;

sayNameForAll.apply(`this`, [`"global"`]);      `// outputs "global:Michael"`
sayNameForAll.apply(person1, [`"person1"`]);  `// outputs "person1:Nicholas"`
sayNameForAll.apply(person2, [`"person2"`]);  `// outputs "person2:Greg"`
```

这段代码将前面的例子中的`call()`替换为`apply()`；结果完全相同。你通常使用哪种方法取决于你拥有的数据类型。如果你已经有了一个数组数据，使用`apply()`；如果只有单独的变量，使用`call()`。

#### `bind()`方法

改变`this`的第三种方法是`bind()`。该方法在 ECMAScript 5 中被添加，并且它的行为与其他两种方法有很大不同。`bind()`的第一个参数是新函数的`this`值。所有其他参数表示应该在新函数中永久设置的命名参数。你仍然可以在稍后传入未永久设置的任何参数。

以下代码展示了两个使用 `bind()` 的例子。你通过将 `this` 的值绑定到 `person1` 来创建 `sayNameForPerson1()` 函数，而 `sayNameForPerson2()` 则将 `this` 绑定到 `person2` 并将第一个参数绑定为 `"person2"`。

```
  `function` sayNameForAll(label) {
      console.log(label `+` `":"` `+` `this`.name);
  }

  `var` person1 `=` {
      name: `"Nicholas"`
  };

  `var` person2 `=` {
      name: `"Greg"`
  };

  `// create a function just for person1`
❶ `var` sayNameForPerson1 `=` sayNameForAll.bind(person1);
  sayNameForPerson1(`"person1"`);       `// outputs "person1:Nicholas"`

  `// create a function just for person2`
❷ `var` sayNameForPerson2 `=` sayNameForAll.bind(person2, `"person2"`);
  sayNameForPerson2();                `// outputs "person2:Greg"`

  `// attaching a method to an object doesn't change 'this'`
❸ person2.sayName `=` sayNameForPerson1;
  person2.sayName(`"person2"`);         `// outputs "person2:Nicholas"`
```

`sayNameForPerson1()` ❶ 没有绑定任何参数，所以你仍然需要传入输出的标签。函数 `sayNameForPerson2()` 不仅将 `this` 绑定到 `person2`，还将第一个参数绑定为 `"person2"` ❷。这意味着你可以调用 `sayNameForPerson2()` 而不需要传入任何额外的参数。这个例子的最后部分将 `sayNameForPerson1()` 添加到 `person2` 上，命名为 `sayName` ❸。该函数已被绑定，因此即使 `sayNameForPerson1` 现在是 `person2` 上的一个函数，`this` 的值也不会改变。该方法仍然输出 `person1.name` 的值。

## 总结

JavaScript 函数的独特之处在于它们本身也是对象，这意味着它们可以像任何其他对象一样被访问、复制、覆盖和处理。JavaScript 函数与其他对象之间最大的区别是一个特殊的内部属性 `[[Call]]`，它包含函数的执行指令。`typeof` 运算符会在对象上查找这个内部属性，如果找到了，就会返回 `"function"`。

函数字面量有两种形式：声明和表达式。函数声明包含在 `function` 关键字右侧的函数名，并且会被提升到定义它们的上下文的顶部。函数表达式用于可以使用其他值的地方，例如赋值表达式、函数参数或另一个函数的返回值。

因为函数是对象，所以有一个 `Function` 构造函数。你可以通过 `Function` 构造函数创建新的函数，但通常不推荐这样做，因为这可能会使代码更难理解，调试也变得更加困难。尽管如此，在某些情况下你可能会遇到它的使用，这些情况通常是在函数的真实形式直到运行时才会知道。

你需要很好地掌握函数的使用，才能理解 JavaScript 中面向对象编程的工作原理。由于 JavaScript 没有类的概念，因此函数和其他对象是你实现聚合和继承的唯一工具。
