## 第六章 对象模式

JavaScript 有许多创建对象的模式，通常有不止一种方法可以完成相同的任务。你可以在任何时候定义自己的自定义类型或通用对象。你可以使用继承来共享对象之间的行为，或者你可以采用其他技术，如 mixins（混入）。你还可以利用 JavaScript 的高级特性，防止对象结构被修改。本章讨论的模式为你提供了强大的对象管理和创建方式，所有这些方式都是基于你的使用场景。

## 私有和特权成员

JavaScript 中的所有对象属性都是公共的，且没有显式的方式来指示某个属性不应该从外部访问。然而，在某些情况下，你可能不希望数据是公开的。例如，当一个对象使用某个值来确定某种状态时，如果在没有对象知情的情况下修改该数据，会导致状态管理过程陷入混乱。一种避免这种情况的方法是使用命名约定。例如，当属性不打算公开时，常常会使用下划线前缀（如 `this._name`）。然而，也有一些不依赖约定的隐藏数据的方法，因此在防止私密信息被修改方面更具“防弹性”。

### 模块模式

*模块模式* 是一种对象创建模式，旨在创建具有私有数据的单例对象。基本方法是使用一个*立即调用函数表达式（IIFE）*，该表达式返回一个对象。IIFE 是一种函数表达式，它被定义后立即调用以产生结果。该函数表达式可以包含任何数量的局部变量，这些变量无法从外部访问。因为返回的对象是在该函数内定义的，所以该对象的方法可以访问这些数据。（所有在 IIFE 内定义的对象都可以访问相同的局部变量。）以这种方式访问私有数据的方法被称为 *特权* 方法。以下是模块模式的基本格式：

```
  `var` yourObject `=` (`function`() {

      `// private data variables`

      `return` {
          `// public methods and properties`
      };
❶ }());
```

在这种模式中，创建并立即执行一个匿名函数。（注意函数末尾的额外括号❶。你可以使用这种语法立即执行匿名函数。）这意味着该函数存在的时间仅为片刻，执行后立即销毁。IIFE（立即调用的函数表达式）是 JavaScript 中非常流行的模式，部分原因在于它们在模块模式中的使用。

模块模式允许你将常规变量用作实际上不公开的对象属性。你通过创建*闭包*函数作为对象方法来实现这一点。闭包只是访问其外部作用域数据的函数。例如，每当你在函数中访问一个全局对象（例如在网页浏览器中的`window`）时，该函数就是在访问一个外部作用域的变量。与模块函数的不同之处在于，变量是在 IIFE 内部声明的，而在 IIFE 内部声明的函数则访问这些变量。例如：

```
  `var` person `=` (`function`() {

❶     `var` age `=` `25`;

      `return` {
          name: `"Nicholas"`,

❷         getAge: `function`() {
              `return` age;
          },

❸         growOlder: `function`() {
              age++;
          }
      };

  }());

  console.log(person.name);       `// "Nicholas"`
  console.log(person.getAge());   `// 25`

  person.age `=` `100`;
  console.log(person.getAge());   `// 25`

  person.growOlder();
  console.log(person.getAge());   `// 26`
```

这段代码使用模块模式创建了`person`对象。`age`变量❶充当该对象的私有属性。它不能直接从对象外部访问，但可以被对象方法使用。对象上有两个特权方法：`getAge()`❷，用于读取`age`变量的值，和`growOlder()`❸，用于增加`age`的值。这两个方法都可以直接访问`age`变量，因为它是在它们定义的外部函数中声明的。

有一种模块模式的变体叫做*揭示模块模式*，它将所有变量和方法放置在 IIFE 的顶部，并简单地将它们分配给返回的对象。你可以像下面这样使用揭示模块模式来编写前面的例子：

```
  `var` person `=` (`function`() {

      `var` age `=` `25`;

      `function` getAge() {
          `return` age;
      }

      `function` growOlder() {
          age++;
      }

      `return` {
          name: `"Nicholas"`,
❶         getAge: getAge,
          growOlder: growOlder
      };

  }());
```

在揭示模块模式中，`age`、`getAge()`和`growOlder()`都被定义为 IIFE 的局部变量。然后，`getAge()`和`growOlder()`函数被分配给返回的对象❶，有效地“揭示”了它们，让它们可以在 IIFE 外部访问。这段代码本质上与之前使用传统模块模式的例子相同；然而，一些人更喜欢这种模式，因为它将所有变量和函数声明集中在一起。

### 构造函数的私有成员

模块模式非常适合定义具有私有属性的独立对象，但如果是需要私有属性的自定义类型呢？你可以在构造函数内部使用类似于模块模式的模式，来创建实例特定的私有数据。例如：

```
  `function` Person(name) {

      `// define a variable only accessible inside of the Person constructor`
      `var` age `=` `25`;

      `this`.name `=` name;

❶     `this`.getAge `=` function() {
          `return` age;
      };

❷     `this`.growOlder `=` `function`() {
          age++;
      };
   }

   `var` person `=` `new` Person(`"Nicholas"`);

   console.log(person.name);       `// "Nicholas"`
   console.log(person.getAge());   `// 25`

   person.age `=` `100`;
   console.log(person.getAge());   `// 25`

   person.growOlder();
   console.log(person.getAge());   `// 26`
```

在这段代码中，`Person`构造函数有一个局部变量`age`。该变量作为`getAge()`❶和`growOlder()`❷方法的一部分。当你创建`Person`的实例时，该实例将获得自己的`age`变量、`getAge()`方法和`growOlder()`方法。从许多方面来看，这类似于模块模式，其中构造函数创建了一个局部作用域并返回`this`对象。如在第四章中讨论的那样，将方法放置在对象实例上比放置在原型上效率低，但当你需要私有的、实例特定的数据时，这是唯一可行的做法。

如果你想让私有数据在所有实例之间共享（就像它在原型上），你可以使用一种混合方法，这种方法看起来像模块模式，但使用构造函数：

```
  `var` Person `=` (`function`() {

      `// everyone shares the same age`
❶     `var` age `=` `25`;

❷     `function` InnerPerson(name) {
          `this`.name `=` name;
      }

      InnerPerson.prototype.getAge `=` `function`() {
          `return` age;
      };

      InnerPerson.prototype.growOlder `=` `function`() {
          age++;
      };

      `return` InnerPerson;

  }());
  `var` person1 `=` `new` Person(`"Nicholas"`);
  `var` person2 `=` `new` Person(`"Greg"`);

  console.log(person1.name);      `// "Nicholas"`
  console.log(person1.getAge());  `// 25`

  console.log(person2.name);      `// "Greg"`
  console.log(person2.getAge());  `// 25`

  person1.growOlder();
  console.log(person1.getAge());  `// 26`
  console.log(person2.getAge());  `// 26`
```

在这段代码中，`InnerPerson` 构造函数❷被定义在一个立即调用的函数表达式（IIFE）内。变量 `age` ❶ 定义在构造函数外部，但被两个原型方法使用。然后返回 `InnerPerson` 构造函数，并成为全局作用域中的 `Person` 构造函数。所有 `Person` 的实例最终都共享 `age` 变量，因此通过一个实例更改其值会自动影响到其他实例。

## 混入（Mixins）

尽管伪经典继承和原型继承在 JavaScript 中使用频繁，但也有一种通过混入实现的伪继承类型。*混入*发生在一个对象获取另一个对象的属性时，而无需修改原型链。第一个对象（*接收者*）实际上是通过直接复制第二个对象（*提供者*）的属性来接收这些属性的。传统上，你可以使用这样的函数来创建混入：

```
`function` mixin(receiver, supplier) {
    `for` (`var` property `in` supplier) {
        `if` (supplier.hasOwnProperty(property)) {
            receiver[property] `=` supplier[property]
        }
    }

    `return` receiver;
}
```

`mixin()` 函数接受两个参数：接收者和提供者。该函数的目标是将所有可枚举的属性从提供者复制到接收者。你通过使用 `for-in` 循环遍历 `supplier` 中的属性，然后将该属性的值赋给接收者上同名的属性来实现这一点。请记住，这是一个浅拷贝，所以如果某个属性包含对象，那么提供者和接收者都会指向同一个对象。这个模式通常用于为已经存在于其他对象上的 JavaScript 对象添加新行为。

例如，你可以通过混入而非继承向一个对象添加事件支持。首先，假设你已经定义了一个用于事件的自定义类型：

```
  `function` EventTarget(){
  }

  EventTarget.prototype `=` {

      constructor: EventTarget,

❶     addListener: `function`(type, listener){

          `// create an array if it doesn't exist`
          `if` (`!``this`.hasOwnProperty(`"_listeners"`)) {
              `this`._listeners `=` [];
          }

          `if` (`typeof` `this`._listeners[type] `==` `"undefined"`){
              `this`._listeners[type] `=` [];
          }

          `this`._listeners[type].push(listener);
      },

❷     fire: `function`(event){

          `if` (`!`event.target){
              event.target `=` `this`;
          }

          `if` (`!`event.type){ `// falsy`
              `throw` `new` `Error`(`"Event object missing 'type' property."`);
          }

          `if` (`this`._listeners `&&` `this`._listeners[event.type] `instanceof` `Array`){
              `var` listeners `=` `this`._listeners[event.type];
              `for` (`var` `i``=``0`, len`=`listeners.length; i `<` len; i`++`){
                  listeners[i].call(this, event);
              }
          }
      },
❸     removeListener: `function`(type, listener){
          `if` (`this`._listeners `&&` `this`._listeners[type] `instanceof` `Array`){
              `var` listeners `=` `this`._listeners[type];
              `for` (`var` `i``=``0`, len`=`listeners.length; i `<` len; `i``++`){
                   `if` (listeners[`i`] `===` listener){
                       listeners.splice(i, `1`);
                       `break`;
                   }
              }
          }
      }
 };
```

`EventTarget` 类型为任何对象提供了基本的事件处理功能。你可以在对象上直接添加❶和移除❸监听器，并触发❷事件。事件监听器存储在 `_listeners` 属性中，只有在第一次调用 `addListener()` 时才会创建这个属性（这使得混入变得更容易）。你可以像这样使用 `EventTarget` 的实例：

```
`var` target `=` `new` EventTarget();
target.addListener(`"message"`, `function`(event) {
    console.log(`"Message is "` `+` event.data);
})

target.fire({
    type: `"message"`,
    data: `"Hello world!"`
});
```

对象支持事件对于 JavaScript 来说非常有用。如果你想要一个支持事件的不同类型对象，你有几个选择。首先，你可以创建一个 `EventTarget` 的新实例，然后添加你需要的属性：

```
`var` person `=` `new` EventTarget();
person.name `=` `"Nicholas"`;
person.sayName `=` `function`() {
    console.log(`this`.name);
    `this`.fire({ type`:` `"namesaid"`, name`:` name });
};
```

在这段代码中，创建了一个名为 `person` 的新变量，作为 `EventTarget` 的实例，并添加了与 `person` 相关的属性。不幸的是，这意味着 `person` 实际上是 `EventTarget` 的一个实例，而不是 `Object` 或自定义类型。你还需要手动添加一堆新属性，这带来了额外的开销。更好的方法是有一种更有组织的方式来实现这一点。

解决这个问题的第二种方法是使用伪经典继承：

```
  `function` Person(name) {
      `this`.name `=` name;
  }

❶ Person.prototype `=` `Object`.create(EventTarget.prototype);
  Person.prototype.constructor `=` Person;

  Person.prototype.sayName = `function`() {
      console.log(`this`.name);
      `this`.fire({ type`:` `"namesaid"`, name`:` name });
  };

  `var` person `=` `new` Person(`"Nicholas"`);

  console.log(person `instanceof` Person);      `// true`
  console.log(person `instanceof` EventTarget); `// true`
```

在这种情况下，有一个新的 `Person` 类型，它继承自 `EventTarget` ❶。你可以之后在 `Person` 的原型上添加任何需要的方法。然而，这并不像它本来应该的那样简洁，你可以认为这种关系并没有意义：一个人是事件目标的一种类型？通过使用混合器，你可以减少分配这些新属性到原型所需的代码量：

```
  `function` Person(name) {
      `this`.name `=` name;
  }
❶ mixin(Person.prototype, `new` EventTarget());
  mixin(Person.prototype, {
      constructor: Person,

      sayName: function() {
          console.log(`this`.name);
          this.fire({ type: "namesaid", name: name });
      }
  });

  `var` person `=` `new` Person(`"Nicholas"`);

  console.log(person `instanceof` Person);      `// true`
  console.log(person `instanceof` EventTarget); `// false`
```

在这里，`Person.prototype` 与一个新的 `EventTarget` 实例混合 ❶ 以获得事件行为。然后，`Person.prototype` 与 `constructor` 和 `sayName()` 混合，以完成原型的组合。在这个示例中，`Person` 的实例不是 `EventTarget` 的实例，因为没有继承。

当然，你可能决定，尽管你确实想使用对象的属性，但你根本不想使用伪类继承的构造函数。在这种情况下，你可以在创建新对象时直接使用混合器：

```
`var` person `=` mixin(`new` EventTarget(), {

    name: `"Nicholas"`,

    sayName: `function`() {
        console.log(`this`.name);
        `this`.fire({ type`:` `"namesaid"`, name`:` name });
    }
});
```

在这个示例中，一个新的 `EventTarget` 实例与一些新的属性混合，创建了 `person` 对象，而不影响 `person` 的原型链。

使用这种方式的混合器时，需要记住的一件事是，供应者上的访问器属性会变成接收者上的数据属性，这意味着如果不小心，你可以覆盖它们。这是因为接收者属性是通过赋值而不是 `Object.defineProperty()` 创建的，这意味着会先读取供应者属性的当前值，然后将其赋给接收者上同名的属性。例如：

```
  `var` person `=` mixin(`new` EventTarget(), {

❶     get name() {
          `return` `"Nicholas"`
      },

      sayName: `function`() {
          console.log(`this`.name);
          `this`.fire({ type`:` `"namesaid"`, name`:` name });
      }
  });

  console.log(person.name);        `// "Nicholas"`

❷ person.name `=` `"Greg"`;
  console.log(person.name);        `// "Greg"`
```

在这段代码中，`name` 被定义为只有 getter 的访问器属性 ❶。这意味着给这个属性赋值应该没有效果。然而，由于访问器属性变成了 `person` 对象上的数据属性，因此可以用新值覆盖 `name` ❷。在调用 `mixin()` 时，`name` 的值从供应者读取并赋给接收者上名为 `name` 的属性。在这个过程中，始终没有定义新的访问器，因此接收者上的 `name` 属性成为了一个数据属性。

如果你希望访问器属性作为访问器属性被复制过来，你需要一个不同的 `mixin()` 函数，比如：

```
  `function` mixin(receiver, supplier) {
❶     `Object`.keys(supplier).forEach(`function`(property) {
          `var` descriptor `=` `Object`.getOwnPropertyDescriptor(supplier, property);
❷         `Object`.defineProperty(receiver, property, descriptor);
      });

      `return` receiver;
  }

  `var` person `=` mixin(`new` EventTarget(), {

      get name() {
          `return` `"Nicholas"`
      },

      sayName: `function`() {
          console.log(`this`.name);
          `this`.fire({ type`:` `"namesaid"`, name`:` name });
      }
  });

  console.log(person.name);       `// "Nicholas"`

  person.name `=` `"Greg"`;
  console.log(person.name);       `// "Nicholas"`
```

这个版本的 `mixin()` 使用 `Object.keys()` ❶ 获取 `supplier` 上所有可枚举的自有属性的数组。然后使用 `forEach()` 方法遍历这些属性。每个属性的属性描述符会被检索出来，并通过 `Object.defineProperty()` ❷ 添加到 `receiver` 上。这确保了所有相关的属性信息都被转移到 `receiver`，而不仅仅是值。这意味着 `person` 对象有一个叫做 `name` 的访问器属性，因此它不能被覆盖。

当然，这个版本的 `mixin()` 仅在 ECMAScript 5 的 JavaScript 引擎中有效。如果你的代码需要在较旧的引擎上运行，你应该将这两种 `mixin()` 方法合并成一个单一的函数：

```
  `function` mixin(receiver, supplier) {

❶     `if` (`Object`.getOwnPropertyDescriptor) {

          `Object`.keys(supplier).forEach(`function`(property) {
              `var` descriptor `=` `Object`.getOwnPropertyDescriptor(supplier, property);
              `Object`.defineProperty(receiver, property, descriptor);
          });

      } `else` {

❷         `for` (`var` property `in` supplier) {
              `if` (supplier.hasOwnProperty(property)) {
                  receiver[property] `=` supplier[property]
              }
          }
      }

      `return` receiver;
  }
```

在这里，`mixin()` 检查 `Object.getOwnPropertyDescriptor()` ❶ 是否存在，以确定 JavaScript 引擎是否支持 ECMAScript 5。如果支持，它就使用 ECMAScript 5 版本。否则，使用 ECMAScript 3 版本 ❷。这个函数在现代和遗留的 JavaScript 引擎中都可以安全使用，因为它们会应用最合适的混合策略。

### 注意

*请记住，Object.keys() 仅返回可枚举的属性。如果你想要复制不可枚举的属性，请改用 `Object.getOwnPropertyNames()`。*

## 范围安全构造函数

因为所有构造函数本质上都是函数，你可以在不使用 `new` 操作符的情况下调用它们，从而影响 `this` 的值。这样做可能会导致意想不到的结果，因为在非严格模式下，`this` 会被强制转换为全局对象，或者在严格模式下构造函数会抛出错误。在第四章中，你遇到过这个例子：

```
  `function` Person(name) {
      `this`.name `=` name;
  }

  Person.prototype.sayName `=` `function`() {
      console.log(`this`.name);
  };

❶ `var` person1 `=` Person(`"Nicholas"`);           `// note: missing "new"`

  console.log(person1 `instanceof` Person);     `// false`
  console.log(`typeof` person1);                `// "undefined"`
  console.log(name);                          `// "Nicholas"`
```

在这种情况下，`name` 被创建为全局变量，因为 `Person` 构造函数是没有 `new` 的情况下调用的 ❶。请记住，这段代码是在非严格模式下运行的，因为如果在严格模式下遗漏 `new` 会抛出错误。构造函数以大写字母开头通常表示它应该以 `new` 开头，但如果你希望允许这种用法并且让函数在没有 `new` 的情况下工作呢？许多内建的构造函数，例如 `Array` 和 `RegExp`，也能在没有 `new` 的情况下工作，因为它们是 *范围安全的*。一个范围安全的构造函数可以带 `new` 或不带 `new` 调用，并且无论哪种方式都会返回相同类型的对象。

当 `new` 与函数一起调用时，`this` 表示的新创建的对象已经是由构造函数表示的自定义类型的实例。所以你可以使用 `instanceof` 来判断函数调用中是否使用了 `new`：

```
`function` Person(name) {
    `if` (`this` `instanceof` Person) {
        `// called with "new"`
    } `else` {
        `// called without "new"`
    }
}
```

使用这样的模式可以让你根据函数是带 `new` 还是不带 `new` 调用来控制函数的行为。你可能希望根据不同的情况处理不同的逻辑，但通常你希望函数的行为相同（经常是为了防止遗漏 `new`）。一个范围安全的 `Person` 构造函数如下所示：

```
`function` Person(name) {
    `if` (`this` `instanceof` Person) {
        `this`.name `=` name;
    } `else` {
        `return` `new` Person(name);
    }
}
```

对于这个构造函数，当使用 `new` 时，`name` 属性总是会被分配。如果没有使用 `new`，构造函数会通过 `new` 递归调用，以创建该对象的正确实例。这样，以下两者是等效的：

```
`var` person1 `=` `new` Person(`"Nicholas"`);
`var` person2 `=` Person(`"Nicholas"`);

console.log(person1 `instanceof` Person);     `// true`
console.log(person2 `instanceof` Person);     `// true`
```

不使用 `new` 操作符创建新对象的做法越来越普遍，这是为了减少因遗漏 `new` 导致的错误。JavaScript 本身有多个带有范围安全构造函数的引用类型，例如 `Object`、`Array`、`RegExp` 和 `Error`。

## 总结

在 JavaScript 中，有许多不同的方式来创建和组合对象。虽然 JavaScript 没有正式的私有属性概念，但你可以创建仅在对象内部可访问的数据或函数。对于单例对象，可以使用模块模式来隐藏外部世界的数据。你可以使用立即调用的函数表达式（IIFE）来定义仅新创建的对象可访问的局部变量和函数。特权方法是指那些能够访问私有数据的对象方法。你还可以通过在构造函数中定义变量或使用 IIFE 创建共享给所有实例的私有数据来创建具有私有数据的构造函数。

Mixins 是一种强大的方法，可以在不继承的情况下为对象添加功能。Mixin 将一个对象的属性复制到另一个对象，从而使接收对象获得功能而无需继承提供对象。与继承不同，mixins 不允许你在对象创建后识别功能的来源。因此，mixins 最适合用于数据属性或小段功能。当你需要获得更多功能并且需要知道这些功能的来源时，继承仍然是首选。

范围安全构造函数是指你可以使用或不使用 `new` 来调用的构造函数，用于创建新的对象实例。该模式利用了 `this` 在构造函数开始执行时即为自定义类型实例的事实，这使得你可以根据是否使用 `new` 操作符来改变构造函数的行为。

*面向对象的 JavaScript 原则* 使用了 New Baskerville、Futura、TheSansMono Condensed 和 Dogma 字体。此书由位于伊利诺伊州梅尔罗斯公园的 Lake Book Manufacturing 印刷和装订。纸张采用 60# Husky Opaque Offset Smooth，且通过了可持续森林计划（SFI）认证。

## 索引

### 数字索引的说明

索引条目中的链接显示为该条目所在章节的标题。由于某些章节包含多个索引标记，因此一个条目可能会有多个指向同一章节的链接。点击任何一个链接都会直接带你到文本中标记出现的地方。

### 符号

==（双等号运算符）， 识别原始类型

===（三等号运算符）， 识别原始类型

[ ]（中括号）

对于数组字面量，对象和数组字面量

对于属性访问，函数字面量

[[ ]]（双中括号表示法）， 函数

[[Call]] 属性，函数

[[Configurable]] 属性， 常见属性，访问器属性

对于密封对象， 密封对象

[[Delete]] 操作，针对对象属性，检测属性

[[Enumerable]] 属性属性，常见属性，访问器属性属性

[[Extensible]] 属性，防止对象修改

[[Get]] 属性，数据属性属性

[[Prototype]] 属性，原型，使用构造函数的原型

[[Put]] 方法，定义属性

对于数据属性，枚举

[[Set]] 属性，定义属性，数据属性属性

[[Value]] 属性，常见属性

[[Writable]] 属性，常见属性

_（下划线），在属性名前缀，属性类型，私有与特权成员

_proto_ 属性，[[[Prototype]] 属性](ch04.html#iddle1288)

{ }（花括号）

和对象属性，实例化内建类型

对于函数内容，函数

### A

访问器属性，枚举

属性，数据属性属性

创建，访问器属性属性

添加属性，解除引用对象

匿名函数，作为值的函数，私有与特权成员

apply() 方法，call() 方法，构造函数继承

arguments 对象，作为值的函数

arguments，作为函数，作为值的函数

参数个数，作为值的函数

Array 内建类型，解除引用对象

数组字面量，实例化内建类型

Array.isArray() 方法，识别引用类型

Array.prototype，修改，更改原型

数组

识别，识别引用类型

传递给 apply()，call() 方法

赋值表达式，函数

属性的属性，属性类型

访问器属性，数据属性属性

数据属性，常见属性

自动装箱，识别数组

### B

bind() 方法，bind() 方法

布尔对象，原始类型，原始包装类型

布尔原始包装类型，识别数组

花括号 ({ })

和对象属性，实例化内建类型

对于函数内容，函数

中括号表示法，用于属性访问，函数字面量

内建对象原型，更改原型

内建类型，实例化，解引用对象

### C

call() 方法，this 对象，构造函数继承

构造函数名称的大小写，构造函数

capitalize() 方法，用于字符串，更改原型

charAt() 方法，原始方法

类，JavaScript 对类的缺乏支持，原始类型和引用类型

闭包函数，模块模式

比较函数，函数作为值

无强制转换的比较，识别原始类型

console.log 函数，构造函数

constructor 属性，构造函数

通过对象字面量表示法进行更改，使用构造函数与原型

构造函数，原始方法，构造函数与原型

继承，对象继承

Object.defineProperty() 方法内部，构造函数

私有成员，模块模式

使用原型与，使用构造函数与原型

目的，构造函数

范围安全，混入

偷窃，构造函数继承

子类型，对象继承，构造函数继承

超类型，对象继承，构造函数继承

create() 方法，对象继承

创建

访问器属性，访问器属性特性

对象，原始方法

属性，在临时对象上，原始包装类型

克罗克福德，道格拉斯，修改 Object.prototype

### D

数据

共享私有，构造函数的私有成员

存储在原型上，使用构造函数与原型

类型（见原始类型；引用类型；类型）

数据属性，枚举

属性，常见属性

从混入中，混入

内建的 Date 类型，解引用对象

Date 对象，valueOf() 方法，valueOf()

声明与表达式，函数

defineProperties() 方法，访问器属性特性

defineProperty() 方法，常见属性，构造函数

删除运算符，检测属性，总结

解引用, 对象, 创建对象

检测属性, 定义属性

点表示法, 用于访问属性, 函数字面量

双等号运算符 (==), 识别基本类型

双中括号表示法 ([[ ]]), 函数

### E

可枚举属性

添加到 Object.prototype, 修改 Object.prototype

供应商与接收者之间的复制, 构造函数的私有成员

枚举, 枚举

等于运算符，双等号 (==) 和三等号 (===), 识别基本类型

错误内建类型, 实例化内建类型

错误

对于基本包装对象, 基本包装类型

严格模式下的构造函数, 构造函数

事件支持, 添加到对象, 混入

表达式, 与声明的对比, 函数

防止对象扩展, 防止对象修改

### F

假值, 定义属性

一等函数, 基本类型和引用类型

JavaScript 的灵活性, 基本类型和引用类型

for-in 循环, 枚举, 修改 Object.prototype, 构造函数的私有成员

网页框架, 在不同框架间传递值, 识别引用类型

freeze() 方法, 冻结对象, 修改原型

冻结对象, 冻结对象

冻结对象，原型修改与之关系, 修改原型

Function 构造器, 实例化内建类型, 对象和数组字面量, 函数作为值

function 关键字, 函数

函数字面量, 对象和数组字面量

函数, 基本类型和引用类型, 函数

作为值, 声明与表达式

声明与表达式, 函数

变量提升, 函数

函数重载, 重载

参数, 函数作为值

### G

垃圾回收语言，作为 JavaScript， 创建对象

getOwnPropertyDescriptor() 方法, 定义多个属性

getPrototypeOf() 方法, [[[原型]] 属性](ch04.html#iddle1134)

getter 函数, 枚举

全局对象, 使用 this 表示, this 对象

### H

哈希映射, JavaScript 对象作为, 总结

hasOwnProperty() 方法，检测属性，构造函数，原型链和 Object.prototype，修改 Object.prototype

提升函数，函数

### I

if 条件，定义属性

立即调用函数表达式 (IIFE)，私有和特权成员

in 操作符，构造函数

测试属性实例，定义属性

继承，继承

对象之间，修改 Object.prototype

构造函数，对象继承

来自 Object.prototype 的方法，原型链和 Object.prototype

原型链，继承

伪经典，构造函数窃取，混入

instanceof 操作符，属性访问

临时对象，原始包装类型

实例，原始与引用类型

（另见对象）

检查类型，构造函数

引用类型，原始方法

原型链与构造函数，使用构造函数与原型

实例化

内建类型，取消引用对象

对象，原始方法

原始包装器，原始包装类型

函数的内部属性，函数

isArray() 方法，识别引用类型

isExtensible() 方法，防止对象修改，封闭对象

isFrozen() 方法，冻结对象

isPrototypeOf() 方法，[[[Prototype]] 属性](ch04.html#iddle1168)，原型链和 Object.prototype

isSealed() 方法，封闭对象

### K

键/值对，总结

keys() 方法，枚举，混入

### L

函数的 length 属性，函数作为值

字面量，原始类型，实例化内建类型

数组，对象和数组字面量

函数，对象和数组字面量

对象，实例化内建类型

正则表达式，函数字面量

### M

内存位置，指针指向，创建对象

方法，原始方法，重载

添加到数组，改变原型

对于超类型，访问，访问超类型方法

原始，原始方法

特权的, 私有和特权成员

定义原型, 使用原型与构造函数

混入, 构造函数的私有成员

数据属性来自, 混入

模块模式, 私有和特权成员

### N

名称

对于构造函数, 大写, 构造函数

对于属性, 私有和特权成员

使用相同的多个函数, 重载

new 操作符, 原始方法, 混入

构造函数和, 构造函数与原型, 构造函数, 构造函数

使用引用类型实例化, 实例化内建类型

使用该对象创建的, 构造函数

null 值, 原始类型

确定一个值是否为, 识别原始类型

将对象变量设置为, 创建对象

设置属性为, 检测属性

typeof 操作符和, 识别原始类型

Number 原始包装类型, 识别数组

数字类型, 原始类型

### O

Object 内建类型, 实例化内建类型

Object 构造函数, 定义属性

对象字面量, 实例化内建类型

对象模式, 对象模式

私有和特权成员, 私有和特权成员

Object.create() 方法, 对象继承

Object.defineProperties() 方法, 访问器属性特性

Object.defineProperty() 方法, 常见属性, 构造函数

Object.freeze() 方法, 冻结对象, 改变原型

Object.getOwnPropertyDescriptor() 方法, 定义多个属性

Object.getPrototypeOf() 方法, [[[Prototype]] 属性](ch04.html#iddle1218)

Object.isExtensible() 方法, 防止对象修改, 密封对象

Object.isFrozen() 方法, 冻结对象

Object.isSealed() 方法, 密封对象

Object.keys() 方法, 枚举, 混入

Object.preventExtensions() 方法, 防止对象修改

Object.prototype 原型

继承的方法, 原型链和 Object.prototype

修改, toString()

Object.prototype.isPrototypeOf() 方法， [[[Prototype]] 属性](ch04.html#iddle1229)， 原型链和 Object.prototype

Object.seal() 方法， 防止对象修改， 修改原型

对象， 原始和引用类型， 原始方法， 理解对象

创建， 原始方法

取消引用， 创建对象

冻结， 冻结对象

继承， 修改 Object.prototype

方法， 重载

修改，防止， 防止对象修改

属性，定义， 定义属性

从原型继承属性， 继承

引用类型作为， 原始和引用类型

密封， 防止对象修改

重载函数， 重载

自有属性

确定存在性， 原型链和 Object.prototype

确定是否可枚举， 原型链和 Object.prototype

对象的， 定义属性

使用 in 操作符检查， 检测属性

与原型属性比较， [[[Prototype]] 属性](ch04.html#iddle1253)

### P

参数， 作为值的函数

人物对象，模块模式创建， 模块模式

指向内存位置， 创建对象

preventExtensions() 方法， 防止对象修改

防止对象修改， 防止对象修改

原始方法， 原始方法

原始类型， 原始和引用类型， 原始类型

原始包装类型， 识别数组

私有数据， 共享， 构造函数的私有成员

私有成员， 私有和特权成员

构造函数， 模块模式

特权成员， 私有和特权成员

属性， 原始方法， 函数字面量， 私有和特权成员

添加或移除， 取消引用对象

复制可枚举的，在接收方和提供方之间， 构造函数的私有成员

在临时对象上创建， 原始包装类型

定义， 定义属性

定义多个， 访问器属性特性

检测， 定义属性

可枚举，添加到 Object.prototype，修改 Object.prototype

枚举，枚举

在原型上识别，原型

移除，检测属性

用于名称的字符串字面量，实例化内建类型

类型，枚举

属性特性，属性类型

更改，常见属性

检索，定义多个属性

propertyIsEnumerable() 方法，枚举，常见属性，原型链和 Object.prototype

原型链，继承，对象继承，构造函数继承

没有对象，对象继承

覆盖，构造函数继承

原型属性

识别，原型

与自身属性对比，[[[Prototype]] 属性](ch04.html#iddle1295)

函数的原型属性，构造函数，对象继承

原型，构造函数

内建对象，更改原型

更改，使用构造函数的原型

识别属性，原型

覆盖，使用构造函数的原型

从继承中继承属性

与构造函数一起使用，使用构造函数的原型

伪经典继承，构造函数偷窃，混入

伪继承，混入用于，构造函数的私有成员

### R

只读属性，属性类型

接收者，复制可枚举属性到供应者和接收者之间，构造函数的私有成员

矩形构造函数，构造函数继承

引用类型，基本类型和引用类型，基本方法

识别，属性访问

引用值，存储在原型上，使用构造函数的原型

正则表达式内建类型，实例化内建类型

正则表达式构造函数，函数字面量

正则表达式字面量，函数字面量

移除属性，解除引用对象，检测属性

检索属性特性，定义多个属性

揭示模块模式，模块模式

### S

范围安全构造函数，混入

seal() 方法, 防止对象修改, 改变原型

密封对象, 原型修改与, 改变原型

密封对象, 防止对象修改

设置器函数, 枚举

共享私有数据, 构造函数的私有成员

签名, 带有多个参数的函数, 重载

sort() 方法, 函数作为值

方括号 ([ ])

对于数组字面量, 对象和数组字面量

属性访问, 函数字面量

Square 构造函数, 构造函数继承

窃取构造函数, 构造函数继承

严格模式

对于不可扩展的对象, 防止对象修改

对于密封对象, 密封对象

字符串字面量, 作为属性名, 实例化内建类型

字符串原始包装类型, 识别数组

字符串类型, 原始类型

字符串

capitalize() 方法, 改变原型

将值转换为用于比较, 函数作为值

方法, 原始方法

substring() 方法, 原始方法

子类型构造函数, 对象继承, 构造函数继承

sum() 函数, 函数作为值

超类型

构造函数, 对象继承, 构造函数继承

方法, 访问, 访问超类型方法

供应商, 在接收者与供应商之间复制可枚举属性, 构造函数的私有成员

### T

临时对象, 在其上创建属性, 原始包装类型

this 对象, this 对象

改变值, this 对象

使用 new 创建, 构造函数

创建长宽属性, 构造函数窃取

toFixed() 方法, 原始方法

toLowerCase() 方法, 原始方法

toString() 方法, 原始方法, 检测属性, 原型链与 Object.prototype, valueOf()

三等号操作符 (===), 识别原始类型

真值, 定义属性

typeof 操作符, 原始类型, 属性访问

类型, 原始类型和引用类型

（另见原始类型；引用类型）

检查不同, 重载

检查实例，构造函数

实例化内置，解除引用对象

### U

undefined 类型， 原始类型

下划线 (_) 在属性名前缀中，属性类型, 私有和特权成员

### V

valueOf() 方法, 原型链和 Object.prototype, valueOf()

值

函数作为，声明与表达式

传递，在网页框架之间，识别引用类型

变量对象，原始类型和引用类型

变量，对于原始类型， 原始类型

### W

网页，传递值在框架之间，识别引用类型

包装类型，原始，识别数组

只写属性，属性类型
